# MS1 Presentation Prep — NeoBank Digital Leap

> For the Week 7 MS1 talk. Two parts:
> 1. C4 diagram spec (the architectural rules to preserve in `docs/diagrams/neobank-digital-leap-c4.drawio`).
> 2. A full slide-by-slide presentation setup with speaker notes — ready to drop into a deck tool.
>
> **Status (as of `1b79aaf`):** the .drawio was hardened by the 6 commits between `a7ab9ac` and `770f065` and the C4 cleanup spec below was applied. The current .drawio has all D-001..D-008 boundaries; the C4 Mermaid mirror (`docs/diagrams/07-c4-container.mmd`) matches it. Part 1 below is now a **reference for the design rules**, not an open work item.

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

### Quick reference

| | |
|---|---|
| **Total time** | 10 min talk + Q&A |
| **Slides** | 10 |
| **Opening line** | "Last year, Open Banking regulation required NeoBank to expose customer data to fintech challengers. Three years ago, the regulator forced the bank to spin off its credit-card arm — and that arm became a direct competitor overnight. The bank has a million customers and a COBOL mainframe that's been running since the 1980s. This is the architecture we propose to add a digital platform without disturbing the ledger." |
| **Closing line** | "Three rules protect the bank. WAF is the only public front door. The CBS Transaction Gateway is the only path to the ledger. The AI Advisor reads through a minimized context boundary, never from the ODS or the CBS. If you remember nothing else, remember those three rules. Thank you." |
| **Headline message** | We add a digital platform around the CBS, not against it — three rules keep it safe. |
| **Image assets to render before the talk** | (1) C4 from `docs/diagrams/neobank-digital-leap-c4.drawio` (slide 6). (2) Account info, bank transfer, AI advisor flows from `02-` / `03-` / `05-.mmd` (slide 7). |

### Slides

#### Slide 1 — Title (0:30)
**Visual**
- Big project name: **NeoBank Digital Leap**
- Subtitle: *Software Architecture — MS1 Preliminary Solution*
- Below: your name, date, course tag

**Speaker**
> I'm [name]. This is the MS1 preliminary solution for NeoBank Digital Leap, the Software Architecture course final project. Today I'll walk you through the architecture we propose to add a digital platform to a 120-year-old bank without disturbing its mainframe.

**Source** — —

---

#### Slide 2 — The bank and the moment (1:00)
**Visual**
- 3 stat cards: **1M** customers · **1905** founded · **120 years** on the mainframe
- One paragraph at the bottom: "Three years ago, the regulator forced the spin-off of the credit-card arm — and it became a direct competitor overnight. Last year, Open Banking regulation required the bank to expose customer data to fintechs. NeoBank now needs digital channels — fast — without touching the ledger."

**Speaker**
> NeoBank is a publicly traded retail bank. One million customers, ten thousand employees, two hundred branches, founded in 1905. The Core Banking System runs on COBOL on z/OS — the same technology the bank has operated for decades. It's the system of record. It's auditable. It processes thousands of transactions per second. And we are not allowed to touch it.
>
> Three years ago, the regulator forced the bank to sell its credit-card company. The buyer became a direct competitor overnight. Last year, Open Banking regulation required NeoBank to expose customer data to licensed third-party providers. And in the last two years, half a dozen fintechs launched competing services aimed at the same customers.
>
> So the bank needs to add digital channels fast. The architecture has to coexist with a mainframe it cannot replace, and ship within a year.

**Source** — `../Software Architecture - Lesson 1B Deck - Final Project - NeoBank.pdf` p.6–11 (Project Description + Competitive Landscape). `docs/01-ms1-preliminary-solution.md` §1 (Executive Summary).

---

#### Slide 3 — Goals & non-negotiables (1:00)
**Visual**
- 3 large number callouts:
  - **99.999%** — uptime
  - **100K → 1M** — digital users (year 1 → year 3)
  - **1 year** — MVP delivery
- Below: small line: *~60 Java + AWS developers · 5 COBOL developers · GDPR regional processing*

**Speaker**
> The non-negotiables are three numbers. Ninety-nine point nine-nine-nine uptime — five nines. That's less than six minutes of downtime a year for the digital platform. One hundred thousand digital users in year one, scaling to a million in year three. And a one-year MVP delivery.
>
> The team is roughly sixty developers, mostly Java and AWS, with five COBOL developers who own the mainframe. So the architecture has to be one a Java + AWS team can build and run, and one the COBOL team is barely aware of.
>
> These three numbers shape every architectural decision. Five nines and one year together rule out any big-bang CBS replacement.

**Source** — `docs/00-hld-introduction.md` (Abstract). `docs/01-ms1-preliminary-solution.md` §2. Deck p.15.

---

#### Slide 4 — Architecture style: three patterns (1:30)
**Visual**
- 3 panels side by side, each with a pattern name and a one-line rationale:
  - **Strangler Fig** — "Build around the CBS, not against it. CBS stays."
  - **CQRS** — "Commands go to CBS. Reads come from read models. The mainframe never sees a million balance checks."
  - **Hybrid on-prem + cloud** — "Regulated, latency-sensitive services on-prem. Elastic, customer-facing services in the regional cloud."

**Speaker**
> Three patterns make the goals achievable.
>
> First, Strangler Fig. We add new digital capabilities around the CBS, not against it. The CBS is the system of record. It stays. We don't try to replace it, we don't try to migrate away from it, and we don't ask the COBOL team to change anything in MVP. Every new capability routes around the existing core.
>
> Second, CQRS — Command Query Responsibility Segregation. Money-moving commands go to the CBS. Reads come from replicated projections, served by read models. So a million users checking their balance doesn't put a million queries on the mainframe. The mainframe sees a controlled, low-volume stream of legitimate money-moving commands. The read models absorb everything else.
>
> Third, hybrid deployment. Regulated, CBS-adjacent, and latency-sensitive services run on-premises, close to the mainframe. Elastic customer-facing services — the WAF, the API Gateway, the BFF, the AI Advisor — run in the regional cloud. The CDS-adjacent Digital Core is on-prem; the customer-facing Digital Edge is in the cloud.

**Source** — `docs/01-ms1-preliminary-solution.md` §3. `docs/decisions/ADR-001-hybrid-strangler-cqrs.md`.

---

#### Slide 5 — Three architectural rules (1:30)
**Visual**
- 3 large rule callouts (re-use the Rule 1 / Rule 2 / Rule 3 boxes from the .drawio):
  - **Rule 1** — WAF is the only public front door. Mobile, Web, and Open Banking TPPs all enter through it.
  - **Rule 2** — The CBS Transaction Gateway is the only path to the mainframe. Every money-moving command goes through it.
  - **Rule 3** — The AI Advisor reads through a minimized context boundary, never from the CBS or the ODS directly.

**Speaker**
> Three rules protect the bank. These aren't aspirations, they're the architectural invariants we will not violate.
>
> Rule one: WAF is the only public front door. Mobile, web, and Open Banking third parties all enter through the WAF. None of them can bypass it. None of them point at the API Gateway directly. The WAF is the choke point — and that means DDoS protection, rate limiting, and bot defense apply uniformly.
>
> Rule two: the CBS Transaction Gateway is the only path to the mainframe. Every money-moving command — every transfer, every authorization, every balance mutation — goes through the gateway. The gateway is the only place that knows how to talk to COBOL. The gateway handles idempotency, throttling, contract versioning, and audit. There is no other way to call the CBS.
>
> Rule three: the AI Financial Advisor sits in the cloud, but it never reads the ODS directly. It goes through the Advisor Context API — a new on-prem service that authorizes and minimizes customer context before returning it. The AI advisor gets only what it needs to render advice, not the full customer record. The ODS is never exposed to the cloud.
>
> These three rules are the spine of the architecture. Everything in the C4 diagram flows from them.

**Source** — `docs/diagrams/neobank-digital-leap-c4.drawio` (Rule 1/2/3 callout boxes). `docs/AGENTS.md` Architectural invariants. `docs/project-status.md` D-006, D-008.

---

#### Slide 6 — C4 Container view (2:00)
**Visual**
- Full C4 diagram, exported as PNG from `docs/diagrams/neobank-digital-leap-c4.drawio` (1920×1080, zoom 200%)
- Optional: small annotations overlaid (a circle on the WAF, a circle on the CBS GW, a circle on the Advisor Context API)

**Speaker**
> Here's the full C4 Container view. Walk left to right.
>
> Far left: three external actors — mobile, web, and Open Banking third-party providers. All three enter through the WAF. The WAF forwards to the API Gateway, which authenticates via Customer IAM and routes to the BFF.
>
> The BFF is the only thing the cloud and on-prem worlds share directly. It orchestrates calls to the on-prem services — account information, consent, reports, transfers, fraud. The Transfer Service goes through the Real-Time Fraud Engine, and if approved, through the CBS Transaction Gateway, to reach the CBS. That's the money path, and you'll see it's the only path.
>
> The AI Advisor sits in the cloud but goes through the Advisor Context API for any data access. The Advisor Context API sits on-prem, checks the consent service for the customer's authorization scope, and reads the authorized projection from the ODS. The AI never crosses the on-prem zone directly.
>
> On the data side: CDC from DB2 and ADABAS flows into adapters, into a durable event bus, into the read models in the ODS. The ODS is not a second system of record — it's a projection.
>
> Notice the three colored boundaries: yellow for cloud, orange for on-prem, red for legacy. Notice the single CBS Transaction Gateway box — that's the only path. Notice the Advisor Context API inside the orange on-prem boundary — that's the AI's only door.

**Source** — `docs/diagrams/neobank-digital-leap-c4.drawio` (the rebuilt C4). `docs/diagrams/07-c4-container.mmd` (Mermaid mirror, version-controlled).

---

#### Slide 7 — Key flows (1:30)
**Visual**
- 3 panels, each with a small flow diagram (rendered from `02-` / `03-` / `05-.mmd`) and a one-line caption:
  - **Account info** — "Read from ODS with freshness metadata. 24h staleness tolerance for external transactions."
  - **Bank transfer** — "Real-time fraud, then CBS Gateway. Only path to the mainframe."
  - **AI advisor** — "Cloud-only, data-minimized. AI → Advisor Context → Consent + ODS."

**Speaker**
> Three flows illustrate the design rules. Twenty seconds each.
>
> Account info: the customer opens the app, the BFF calls the Account Information Service, which reads from the ODS with freshness metadata. The read model may be up to 24 hours stale for incoming external transactions — that's a deliberate trade-off, captured as NFR-PC-060. It lets us absorb the read load without putting traffic on the mainframe.
>
> Bank transfer: the customer submits a transfer, the Transfer Service calls the Real-Time Fraud Engine, and if approved, the CBS Transaction Gateway executes the transaction on the CBS. The gateway is the only thing that knows how to talk to the mainframe. The COBOL team owns the contract. We don't bypass it.
>
> AI advisor: the BFF requests advice, the AI Advisor asks the Advisor Context API for minimized context, the API checks the consent service for what the customer has authorized, reads the authorized projection, and returns a minimal dataset — just enough to render advice. The AI never gets the full customer record.

**Source** — `docs/diagrams/02-account-information-flow.mmd`, `03-bank-transfer-flow.mmd`, `05-ai-financial-advisor-flow.mmd`. `docs/01-ms1-preliminary-solution.md` §6.

---

#### Slide 8 — Decisions & risks (1:00)
**Visual**
- 2 tables side by side, or stacked:
  - **Decisions (D-001..D-008)**: 8 rows, each one-line. Highlight D-002 (CBS GW), D-005/D-008 (AI boundary).
  - **Top 3 risks**: CBS integration bottleneck · Data freshness confusion · ODS becomes a second system of record. One-line mitigations next to each.

**Speaker**
> We've made the major decisions. Eight of them are accepted, captured in the project status document and the architecture decision records. The most important to call out: D-001, the CBS stays the system of record. D-002, the CBS Transaction Gateway is the only path. D-003, the ODS is a projection, never a second system of record. D-008, the AI Advisor reads through a minimized context boundary.
>
> The top three risks. CBS integration bottleneck: mitigated by isolating CBS calls behind the gateway, with idempotency, throttling, and contract testing. Data freshness confusion: mitigated by the 24-hour staleness tolerance and visible freshness metadata on every read. ODS becomes a second system of record: mitigated by the design rule that the ODS is read-only and CBS is authoritative.
>
> Two decisions remain open. The on-prem ODS vendor — Oracle, SQL Server, or Db2, all viable for a conservative bank; vendor decision deferred to MS2. And the cloud region for the AI advisor, pending regulatory approval.

**Source** — `docs/project-status.md` Current Decisions (D-001..D-008). `docs/07-risks-and-mitigations.md`. `docs/02-ods-database-recommendation.md`.

---

#### Slide 9 — MS2 plan & open questions (0:30)
**Visual**
- A horizontal timeline with three milestones: **MS1 (Week 7)** — done · **MS2 (Week 13)** — security / network topology / performance / data flow diagrams · **Final HLD (end of course)** — full HLD
- Below: 3–4 open questions highlighted, e.g. *ODS vendor · RTO/RPO per component · Cloud region for AI · Migration cutover plan*

**Speaker**
> For MS2 in Week 13, we add four diagrams — security boundary, network topology, performance, and data flow. The network topology one is teacher-mandated; we have a scaffold ready. We resolve the eleven open questions and pick the ODS vendor. Final HLD is due at the end of the course. Any questions?

**Source** — `docs/project-status.md` Next Work. `docs/08-open-issues.md`. Deck p.17.

---

#### Slide 10 — Thank you / Q&A (open)
**Visual**
- "Thank you" + your name + contact info
- The three rules as a footer reminder, in small text

**Speaker**
> Thank you. Happy to take questions.

**Source** — —

---

### If running short
- Combine 2+3 (context + goals, ~1:30)
- Combine 7+8 (flows + decisions, ~2:00)
- Total drops to 8 slides, ~7 min

### If asked (Q&A backup)

**Why is the AI Advisor in the cloud and not on-prem?**
> The deck is explicit (p.14): "only this service can run on the cloud and not on-prem". The AI Advisor is the one service the bank is comfortable running outside the data center — the Advisor Context API ensures it only sees minimized, authorized context. If the regulator refuses the cloud, the AI can move on-prem without changing the boundary.

**Why an Advisor Context API and not direct AI → ODS?**
> Three reasons. First, data minimization — the AI only sees what it needs for advice, not the full customer record. Second, authorization — every AI request is checked against the customer's consent. Third, auditability — the API writes to the audit log; the AI doesn't.

**Why not Aurora or PostgreSQL for the ODS?**
> `docs/02-ods-database-recommendation.md` covers this. Aurora is a cloud-managed service — wrong fit for a bank-controlled on-prem deployment. PostgreSQL is technically valid but fights the conservative bank culture; the DBA team, the procurement team, and the regulator are more comfortable with Oracle, SQL Server, or Db2.

**What's the disaster recovery story?**
> 99.999% maps to active-active or active-passive redundancy on the critical path — the CBS Transaction Gateway, the Event Bus, and the ODS are all marked as HA pairs. NFR-AR-020 requires RTO and RPO per component by MS2. NFR-DI-040 requires idempotent transfer commands, so retries are safe.

**How does GDPR work?**
> NFR-GDPR-010 — data processed in the approved geographic region, both for the on-prem Digital Core and the regional cloud. NFR-GDPR-020 — right-to-delete / anonymization workflows for personal data. NFR-GDPR-030 and 040 — financial and audit records are legally retained, but personal data is deletable; we document the split clearly.

**What about the 24-hour account staleness?**
> NFR-PC-060, taken from the deck (p.15). For incoming transactions from other banks and for payments not made through the app, account information may be up to 24 hours stale. This is a deliberate trade-off — it lets the ODS absorb the read load without putting traffic on the mainframe, and it lets us display visible freshness metadata so customers know what they're seeing.

**What's the cost?**
> The deck requires an operational cost calculation. Cloud costs go through the AWS or Azure pricing calculator. On-prem costs are sized separately. The Sizing section (`docs/04-sizing.md`) is scaffolded and gets filled in during MS2.

**Why hybrid and not all-cloud?**
> Three things keep regulated workloads on-prem. Legal: the CBS-adjacent services can't legally run outside the bank's data center. Latency: services that talk to the mainframe need to be close to it. Operational: the COBOL team needs to be able to see and operate the gateway that talks to their system. The cloud handles the elastic, customer-facing work; the on-prem handles the regulated, mainframe-adjacent work.

**How do you scale to a million users?**
> Three numbers. 100K year 1 is absorbed by the read models and the cloud autoscaling. 1M year 3 is the same architecture, scaled — bigger ODS, more partitions, more replicas. No re-architecture needed. The cloud edge scales horizontally; the on-prem core scales vertically (per bank convention).

---

### Speaker tips

- **Open slow.** Pause for two seconds after the title slide before the opening line. The room needs a beat to settle.
- **Eye contact on the rules.** When you say "three rules protect the bank", look at the audience, not the slide. Then turn to the slide for each rule.
- **Walk the C4.** Don't narrate the whole diagram. Stand on the left, point with your hand (or laser), move right as you go. Pause at each boundary: yellow for cloud, orange for on-prem, red for legacy.
- **Slow down on Rule 3.** Rule 3 (the AI boundary) is the new D-008 and the strongest differentiator. Take an extra breath before explaining it.
- **Don't apologize.** If the C4 looks busy, don't say "this is complex" — say "the architecture is shaped by three rules" and point at the Rule callouts.
- **Time check.** At slide 6, you should be at ~5:30. If you're past 6:00 by slide 6, skip the flow walkthrough and go straight to decisions.
- **Closing beat.** On slide 10, say the closing line exactly as written — "three rules protect the bank" is the take-home.

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
- [ ] Re-read the Q&A backup once. Pick the 3 questions you think the teacher is most likely to ask and rehearse 30-second answers.
