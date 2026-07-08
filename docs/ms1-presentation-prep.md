# MS1 Presentation Prep — NeoBank Digital Leap

> For the Week 7 MS1 talk. Two parts:
> 1. C4 diagram reference (the architectural rules to preserve in `docs/diagrams/neobank-digital-leap-c4.drawio`).
> 2. A full slide-by-slide setup with speaker notes — rewritten around Jukka's note: lead with **insights and aha-moments**, not context; the audience already knows the business goal, requirements, and system landscape.
>
> **Status (as of `1b79aaf`):** the .drawio was hardened by the 6 commits between `a7ab9ac` and `770f065` and the C4 cleanup spec below was applied. The current .drawio has all D-001..D-008 boundaries; the C4 Mermaid mirror (`docs/diagrams/07-c4-container.mmd`) matches it. Part 1 below is a **reference for the design rules**, not an open work item.

---

## Part 1 — C4 diagram reference (architectural rules to preserve)

### What the diagram must show
The C4 Container view has to make three architectural rules visually obvious on first glance. If any of these break, the architecture is wrong, regardless of how nice the diagram looks.

- **Rule 1 — Public ingress.** All public traffic (Mobile, Web, Open Banking TPPs) enters through the WAF / DDoS layer. No external actor bypasses the WAF. No actor points directly at the API Gateway, BFF, or any on-prem service.
- **Rule 2 — Mainframe path.** The CBS Transaction Gateway is the **only** path to the CBS. No on-prem service, no adapter, and no analytics service may call CBS / DB2 / ADABAS directly.
- **Rule 3 — AI data-minimization.** The AI Financial Advisor never reads the ODS directly. It goes through the **Advisor Context API**, which authorizes and minimizes customer context before returning it. The Advisor Context API sits inside the on-prem Digital Core.

### Forbidden edges (do not draw these)
```text
TPPs           -> Consent Service      # Rule 1 violation
TPPs           -> any on-prem service  # Rule 1 violation
AI Advisor     -> ODS / Data Stores    # Rule 3 violation
AI Advisor     -> CBS / DB2 / ADABAS   # Rule 3 violation
BFF            -> CBS                  # Rule 2 violation
Transfer Svc   -> DB2 / ADABAS         # Rule 2 violation
Account / Report / Consent -> CBS      # Rule 2 violation
On-Prem DMZ    -> ODS / Event Bus      # segmentation violation (no direct DMZ read)
```

### Required edges (must be present)
```text
mobile / web / tpp -> WAF -> API Gateway -> IAM -> BFF
BFF -> Account / Consent / Report / Transfer
Transfer -> Real-Time Fraud -> CBS Gateway -> CBS
CBS -> DB2; CBS -> ADABAS
DB2 / ADABAS -> Adapter -> Event Bus -> ODS / Offline Fraud / Audit
ODS <- Account, Report, Consent, Offline Fraud, Advisor Context
AI Advisor -> Advisor Context -> Consent (scope check) + ODS (authorized SQL)
Transfer / CBS Gateway / Real-Time Fraud -> Audit
On-Prem Observability -> Cloud Observability
```

### DR / HA markers
Mark the CBS Gateway, Event Bus, and Data Stores as HA pairs (e.g. `‖` or `(replica)`). The 99.999% target (NFR-AR-010) demands visible redundancy on the critical path.

### Style notes
- Export the .drawio to **PNG @ 1920×1080 (zoom 200%)** for slide embedding.
- Keep the 3 Rule callout boxes visible — they are the talk-track.

---

## Part 2 — Slide deck (10 slides, ~10 min)

> **The spine, in one sentence:** the four constraints (1-year MVP, 99.999% uptime, mainframe we can't replace, public APIs) force three architectural decisions (Strangler Fig, CQRS, hybrid on-prem+cloud), and three rules protect the bank (WAF, CBS Gateway, AI boundary). Everything in the C4 follows from that.

### Quick reference

| | |
|---|---|
| **Total time** | 10 min talk + Q&A |
| **Slides** | 10 |
| **Opening line** | "We were given four constraints. A one-year MVP. Ninety-nine point nine-nine-nine uptime. A COBOL mainframe we are not allowed to replace. And a public API surface. Pick any three of those four, and the architecture is forced. We picked Strangler Fig, CQRS, and hybrid on-prem plus cloud. Let me show you why — and what we'd do differently with more time." |
| **Closing line** | "If you take one thing away, it's this: in legacy modernization, the constraints are the design. The five COBOL developers, the twenty-four-hour staleness rule, the five-nines target — they're not problems to solve. They're the shape of the answer. Thank you." |
| **Headline message** | The mainframe isn't the obstacle; it's the foundation. The architecture crystallizes once you accept that. |
| **Image assets** | (1) C4 from `docs/diagrams/neobank-digital-leap-c4.drawio` (slide 6). (2) Account info, bank transfer, AI advisor flows from `02-` / `03-` / `05-.mmd` (slide 7). |

### Slides

#### Slide 1 — Title (0:30)
**Visual**
- Big project name: **NeoBank Digital Leap**
- Subtitle: *Software Architecture — MS1 Preliminary Solution*
- Below: your name, date, course tag

**Speaker**
> I'm [name]. This is the MS1 preliminary solution for NeoBank Digital Leap, the Software Architecture course final project. I won't spend time on the bank or the requirements — you know them. I'll spend the next ten minutes on the design decisions and what we learned.

**Source** — —

---

#### Slide 2 — The constraint that shaped everything (1:00)
**Visual**
- Four big stats at the top:
  - **1 year** — MVP delivery
  - **99.999%** — uptime (less than 6 min/year of downtime)
  - **120 years** — the COBOL mainframe, untouchable
  - **1M → users** — peak target
- Below: one line — *Pick any three of these. The architecture is forced.*

**Speaker**
> Four numbers, four constraints. The mainframe has been running since the 1980s. It's the system of record. We are not allowed to replace it. The bank needs to launch a digital platform in one year. The platform has to be up ninety-nine point nine-nine-nine percent of the time, which is less than six minutes of downtime per year. And it has to serve a million users inside the same geographic region within three years.
>
> Pick any three of those four, and the architecture is forced. The combination rules out most options. Big-bang replacement is out. All-cloud is out — the regulator won't approve a public cloud for the regulated workloads. An all-on-prem digital platform is out — a year isn't enough time to procure, install, and operate new on-prem capacity.
>
> What's left is a hybrid: regulated, mainframe-adjacent services on-prem; elastic, customer-facing services in the regional cloud. The mainframe stays. New services are added around it. The one-year MVP is the consequence of that single design decision — wrap, don't replace.

**Source** — `docs/00-hld-introduction.md`. `docs/01-ms1-preliminary-solution.md` §2. Deck p.15.

---

#### Slide 3 — Insight #1: Wrap, don't replace (1:00)
**Visual**
- Big headline: **Don't replace the mainframe. Design around it.**
- Two-column compare: "Replace (ruled out)" vs "Strangler Fig (chosen)"
- One line at the bottom: *The mainframe's contract is small: it's an authoritative ledger, not a feature platform. That's what makes wrapping possible.*

**Speaker**
> The first aha-moment for me was this: the mainframe isn't the obstacle, it's the foundation.
>
> Most teams, when they're handed a COBOL system, think "we need to modernize it". But the mainframe's contract is actually quite small. It's an authoritative ledger. It processes transactions. It produces audit trails. It does a few things very well, very fast, and very reliably — and we don't need it to do anything else.
>
> So instead of replacing it, we wrap it. The Strangler Fig pattern: new digital capabilities are added *around* the CBS, not against it. The CBS doesn't change in MVP. The COBOL team owns the contract. The Java + AWS team builds the wrapper.
>
> This single decision cascades into everything else. It forces CQRS, because reads from the new digital services can't go through the mainframe. It forces the event bus, because data has to flow out of the mainframe. It forces the ODS, because reads need a home. It forces the CBS Transaction Gateway, because there has to be exactly one way to talk to the mainframe.
>
> Once you stop trying to replace the mainframe, the rest of the architecture falls out almost mechanically.

**Source** — `docs/decisions/ADR-001-hybrid-strangler-cqrs.md`. `docs/01-ms1-preliminary-solution.md` §3.

---

#### Slide 4 — Insight #2: Reads and writes are different (1:00)
**Visual**
- Big headline: **CQRS isn't an optimization. It's a survival strategy.**
- A small table: *Mainframe: 5,000 tx/sec* vs *1M users × 1 balance check = 1M reads* vs *Result: 200× over budget*
- One line: *If reads went to the mainframe, the MVP doesn't launch.*

**Speaker**
> The second aha-moment was a number. The mainframe does about five thousand transactions per second. A million users, each checking their balance once a day, is roughly twelve reads per second on average — but during business hours it's a hundred-plus, and during a payroll run it's thousands. Across the whole day, we need to absorb a million balance checks.
>
> A million reads a day, all going through the mainframe, would crush it. Even if it didn't, every read costs the bank money — the mainframe bills by the operation. So the design has to separate reads from writes.
>
> That's CQRS. Commands — money-moving operations — go to the CBS. Queries — balance checks, account lookups, reports — go to replicated projections in the ODS, populated by change-data-capture from the mainframe.
>
> The ODS is not a second system of record. It's a projection, refreshed from the source of truth. The bank accepts a 24-hour staleness window on incoming external transactions — that's the only honest trade-off — and the rest of the read path is fully consistent.
>
> Without this split, the architecture is infeasible. CQRS isn't an optimization. It's the reason the MVP can launch at all.

**Source** — `docs/decisions/ADR-001-hybrid-strangler-cqrs.md`. `docs/02-ods-database-recommendation.md`. `docs/requirements/requirements-draft.md` NFR-PC-060.

---

#### Slide 5 — Insight #3: AI is a privacy problem, not an ML problem (1:00)
**Visual**
- Big headline: **The AI Advisor is a privacy boundary, not a feature.**
- A small before/after:
  - *Before (naive): AI Advisor → ODS — what does the AI need to see? Everything.*
  - *After (D-008): AI Advisor → Advisor Context API → Consent + authorized projection.*
- One line: *The hard question isn't "what can the model do?" It's "what does it have the right to see?"*

**Speaker**
> The third aha-moment was about the AI Advisor. I started this work assuming the AI Advisor was a machine-learning problem. Build a model, feed it data, get advice. That framing is wrong.
>
> The AI Advisor is a privacy problem. The hard question isn't "what can the model do?" — it's "what does the model have the right to see?"
>
> A cloud-hosted AI service that reads the full customer record is a non-starter. The customer record contains things the customer has not authorized for advice — payment history, fraud signals, sensitive personal data. The AI doesn't need any of that. It needs a minimal, authorized, regionally compliant context.
>
> So we added a new on-prem service — the Advisor Context API. It sits between the AI and the data. Every AI request goes: *what's the customer authorized for* (checked against the consent service) → *give me only the authorized projection* (read from the ODS) → *return the minimum*. The AI never sees the full record. The AI never crosses the on-prem zone directly.
>
> This isn't a feature. It's a boundary. And once we drew it, the architecture got simpler — not harder.

**Source** — `docs/project-status.md` D-008. `docs/diagrams/07-c4-container.mmd` (Advisor Context API edges). `docs/AGENTS.md` invariants.

---

#### Slide 6 — The C4 (2:00)
**Visual**
- Full C4 diagram, exported as PNG from `docs/diagrams/neobank-digital-leap-c4.drawio` (1920×1080, zoom 200%)
- The 3 Rule callouts visible (Rule 1 / Rule 2 / Rule 3)
- Optional: small annotations on the WAF, the CBS Gateway, and the Advisor Context API

**Speaker**
> Here's the architecture in one diagram. I'll walk it left to right, but I'll anchor each region back to one of the three insights.
>
> Far left: the three external actors — mobile, web, and Open Banking third-party providers. All three enter through the WAF. That's Insight #1's first consequence: there is exactly one public front door. Rule 1.
>
> The WAF forwards to the API Gateway, which authenticates and routes to the BFF. The BFF is the only thing the cloud and on-prem worlds share directly.
>
> Now into the on-prem Digital Core. Account, Consent, Report, Transfer, and the AI's advisor context — all reachable from the BFF, all serving the bank's regulated, mainframe-adjacent workloads. The Transfer Service is the one that touches the ledger: it goes through the Real-Time Fraud Engine, and if approved, through the CBS Transaction Gateway, to reach the CBS. That's Insight #1's second consequence: the mainframe has exactly one door, and we never bypass it. Rule 2.
>
> The AI Advisor sits in the cloud. But its data access is mediated by the Advisor Context API, which is on-prem. That's Insight #3: the AI is a privacy boundary, not a feature. Rule 3.
>
> On the data side: CDC from DB2 and ADABAS flows into adapters, into a durable event bus, into the ODS. Insight #2 in action: reads come from projections, not from the mainframe.
>
> Three boundaries, color-coded. Yellow for cloud, orange for on-prem, red for legacy. Three rules. The C4 is just the picture of the constraints and the consequences.

**Source** — `docs/diagrams/neobank-digital-leap-c4.drawio`. `docs/diagrams/07-c4-container.mmd`.

---

#### Slide 7 — The 3 flows (1:00)
**Visual**
- 3 panels, each with a small flow diagram (rendered from `02-` / `03-` / `05-.mmd`) and a one-line caption:
  - **Account info** — *BFF → Account Service → ODS. Read models with visible freshness metadata. 24h staleness tolerance.*
  - **Bank transfer** — *BFF → Transfer → Real-Time Fraud → CBS Gateway → CBS. Only path to the mainframe.*
  - **AI advisor** — *BFF → AI → Advisor Context → Consent + authorized ODS projection. Data-minimized.*

**Speaker**
> Three flows illustrate the insights in action. Twenty seconds each.
>
> Account info: the customer opens the app, the BFF calls the Account Information Service, which reads from the ODS. The read model may be up to 24 hours stale for incoming external transactions — that's a deliberate trade-off, captured as NFR-PC-060, and the customer sees the freshness on screen. Insight #2 in action: reads don't go to the mainframe.
>
> Bank transfer: the customer submits a transfer, the Transfer Service calls the Real-Time Fraud Engine, and if approved, the CBS Transaction Gateway executes the transaction. Insight #1 in action: there's exactly one way to the mainframe.
>
> AI advisor: the BFF requests advice, the AI Advisor asks the Advisor Context API for minimized context, the API checks the consent service for what the customer has authorized, reads the authorized projection, and returns a minimal dataset. Insight #3 in action: the AI never sees the full customer record.

**Source** — `docs/diagrams/02-account-information-flow.mmd`, `03-bank-transfer-flow.mmd`, `05-ai-financial-advisor-flow.mmd`.

---

#### Slide 8 — What we'd revisit (1:00)
**Visual**
- A 4-row table: risk / what it is / what we'd do differently
  - **24h staleness** / customers may see outdated balances for external transactions / visible freshness metadata + a "live refresh on transfer" pattern in MVP+1
  - **5 COBOL developers** / single point of capacity for any CBS change / pair-programming with the Java team on the gateway contract; documented runbooks
  - **No DR yet** / the C4 marks ‖ HA pairs but the topology isn't designed / full network topology + DR runbook in MS2 (scaffolded)
  - **ODS vendor not picked** / Oracle / SQL Server / Db2 shortlist; vendor decision deferred / MS2 decision; the architecture works for any of the three

**Speaker**
> Now the part the teacher asked for — what we'd do differently with more time. We don't have aha-moments without also having things we got wrong. Four.
>
> First, the 24-hour staleness tolerance. It's an honest trade-off, but it can hurt customer trust if a balance looks wrong. With more time, we'd add a "live refresh on transfer" pattern — any time a transfer touches a balance, the BFF forces a fresh read through a synchronous path. For MVP, we accepted the trade-off.
>
> Second, the five COBOL developers. We have five deep experts on the system that processes ninety-nine point nine-nine-nine percent of the bank's money. That's a feature, not a bug — but it's also a capacity risk. With more time, we'd pair-program the Java team with the COBOL team on the CBS Gateway contract, and document runbooks so the gateway is operable by more than five people.
>
> Third, no DR topology yet. The C4 shows HA markers on the CBS Gateway, the event bus, and the ODS — but those are aspirational. The full network topology and DR runbook is MS2 work; we have a scaffold.
>
> Fourth, the ODS vendor. Oracle, SQL Server, or Db2 — all viable. We deferred the decision to MS2. The architecture works for any of the three, so the decision is a procurement call, not a design call.
>
> We're not inventing rockets. We're building a working system, and these are the rough edges.

**Source** — `docs/06-limitations.md`. `docs/07-risks-and-mitigations.md`. `docs/02-ods-database-recommendation.md`.

---

#### Slide 9 — What's open (0:30)
**Visual**
- Horizontal timeline: **MS1 (Week 7)** · **MS2 (Week 13)** — security / network topology / performance / data flow diagrams + DR runbook + ODS vendor decision · **Final HLD (end of course)**
- Below: 3 biggest open questions highlighted

**Speaker**
> For MS2 in Week 13: we add the four diagrams — security boundary, network topology, performance, and data flow. We resolve the eleven open questions. We pick the ODS vendor. We design the DR topology. Final HLD is due at the end of the course. Any questions?

**Source** — `docs/project-status.md` Next Work. `docs/08-open-issues.md`. Deck p.17.

---

#### Slide 10 — Thank you / Q&A (open)
**Visual**
- "Thank you" + your name + contact info
- The three rules as a footer reminder, in small text

**Speaker**
> Thank you.

**Source** — —

---

### If running short
- Drop slide 1 to 30 seconds.
- Skip slide 9 ("What's open") — close after slide 8.
- Compress slide 6 (C4) to 90 seconds.

### If asked (Q&A backup)

**The audience is architects. Expect technical depth, not intro questions.**

**Q: Why is the AI Advisor in the cloud and not on-prem?**
> The deck is explicit (p.14): "only this service can run on the cloud and not on-prem". The AI Advisor is the one service the bank is comfortable running outside the data center. The Advisor Context API ensures it only sees minimized, authorized context. If the regulator refuses the cloud, the AI can move on-prem without changing the boundary — that's the point of the boundary.

**Q: Why an Advisor Context API and not direct AI → ODS?**
> Three reasons. Data minimization: the AI only sees what it needs for advice, not the full customer record. Authorization: every AI request is checked against the customer's consent. Auditability: the API writes to the audit log; the AI doesn't. The API is also the natural place to add rate limiting, abuse detection, and per-customer quota.

**Q: How do you test the AI boundary?**
> Two ways. Functional tests assert that the AI never receives fields it didn't ask for. Compliance tests assert that the audit log records every AI request with the data-minimization decision (which fields were returned, on what consent basis). Both are part of the test pyramid in MS2.

**Q: Why hybrid and not all-cloud?**
> Three things keep regulated workloads on-prem. Legal: the CBS-adjacent services can't legally run outside the bank's data center. Latency: services that talk to the mainframe need to be close to it — every millisecond compounds. Operational: the COBOL team needs to be able to see and operate the gateway that talks to their system. The cloud handles elastic, customer-facing work; the on-prem handles regulated, mainframe-adjacent work. We considered all-cloud with a private region — the regulatory approval risk made that non-viable for MVP.

**Q: How do you scale to a million users?**
> Three answers. 100K year 1 is absorbed by the read models and the cloud autoscaling. 1M year 3 is the same architecture, scaled — bigger ODS, more partitions, more replicas. The cloud edge scales horizontally; the on-prem core scales vertically (per bank convention). And the AI Advisor is stateless — it can scale horizontally as long as the Advisor Context API scales with it, which is a normal SQL read workload.

**Q: What's the migration path from current state?**
> Two-phase. Phase 1: the on-prem Digital Core goes live, but the existing channels (call center, branch) keep working. The first users opt into the digital app; the rest don't notice. Phase 2: cut over the default routing so new digital requests go to the BFF; legacy channels are sunset after a parallel-run period. The Strangler Fig is migration-friendly: we replace one channel at a time, behind the same public API surface.

**Q: What's the disaster recovery story for the CBS?**
> The CBS itself is the bank's existing mainframe; it has its own DR runbook (untouched by this work). The CBS Transaction Gateway is HA. The event bus and ODS are HA pairs (‖). Region failure is the open question — we're single-region for MVP, single-region for three years. Multi-region active-active banking is explicitly out of scope (`docs/06-limitations.md`).

**Q: Why Oracle / SQL Server / Db2 for the ODS?**
> It's a conservative bank. The three are mature, have strong HA/DR patterns, are already approved by the bank's enterprise architecture, and the DBAs know them. ADR-002 is explicit about why we're not using Aurora (cloud-managed, wrong fit) or PostgreSQL (fights the bank culture). The final vendor decision is MS2.

**Q: What's the cost?**
> The deck requires an operational cost calculation. Cloud costs go through the AWS or Azure pricing calculator. On-prem costs are sized separately (HW amortization, Oracle/SQL Server/Db2 licensing, ops). The Sizing section (`docs/04-sizing.md`) is scaffolded and gets filled in during MS2. I don't have a number for you today, but the architecture is built so the cost is dominated by cloud autoscaling — on-prem is a fixed cost.

**Q: The 24h staleness is a customer-trust risk. How do you mitigate it?**
> Two ways. First, visible freshness metadata on every balance display — the customer sees "as of 14 minutes ago" or "as of 23 hours ago". Second, the "live refresh on transfer" pattern I mentioned on slide 8: any time a transfer lands, we force a fresh read. For external incoming transactions, the 24h window is the trade-off; for everything else, freshness is sub-second.

---

### Speaker tips

- **Open with the constraint, not the context.** The teacher is right that the audience knows the business. The opening line is the forcing function. Say it, then pause.
- **The 3 insights are the talk.** Everything else is evidence. If you're running out of time, drop the flows (slide 7) and the open-questions slide. Never drop an insight.
- **Speak in your own voice, not the script.** The notes above are what I'd say, not what you must say. If a sentence doesn't sound like you, rewrite it. The teacher is listening for honesty, not polish.
- **The closing line is the take-home.** "The constraints are the design" — that's the sentence the room remembers. Say it slowly.
- **Time check.** At slide 6 (C4), you should be at ~5:30. At slide 8 (revisit), you should be at ~8:00. If you're past 6:30 by slide 6, skip the flows.
- **Honest ≠ apologetic.** "What we'd revisit" is a strength slide, not a weakness slide. It shows you've thought about trade-offs.

---

### Export checklist (do this the night before)

- [ ] Open `docs/diagrams/neobank-digital-leap-c4.drawio` in app.diagrams.net.
- [ ] File → Export As → PNG, zoom 200%, target 1920×1080 → save as `ms1-c4.png` next to the .drawio.
- [ ] Render the 3 Mermaid flows to PNG:
  ```bash
  mmdc -i docs/diagrams/02-account-information-flow.mmd -o /tmp/02.png -w 1200
  mmdc -i docs/diagrams/03-bank-transfer-flow.mmd -o /tmp/03.png -w 1200
  mmdc -i docs/diagrams/05-ai-financial-advisor-flow.mmd -o /tmp/05.png -w 1200
  ```
  (Use the Mermaid CLI — `npm install -g @mermaid-js/mermaid-cli` if not installed.)
- [ ] Re-read the opening and closing lines out loud once. Time yourself.
- [ ] Pick the 3 Q&A questions you think the teacher is most likely to ask, and rehearse 30-second answers in your own voice.
