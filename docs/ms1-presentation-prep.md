# MS1 Presentation Prep — NeoBank Digital Leap

> For the Week 7 MS1 talk. Two parts:
> 1. A precise draw.io cleanup spec for the C4 Container diagram (`docs/diagrams/neobank-digital-leap-c4.drawio`).
> 2. A slide outline with sources, timings, and speaker notes.

**Status:** Part 1 (C4 cleanup) has been **applied to the .drawio** in the same commit that added this doc. The Mermaid mirror (`docs/diagrams/07-c4-container.mmd`) was later hardened so that TPPs do not bypass WAF/API Gateway and the AI Advisor does not access the ODS directly. Keep the draw.io version aligned with the Mermaid source before exporting the final MS1 image.

## Part 1 — C4 diagram cleanup spec

### Why this is needed
The current draw.io C4 Container diagram is the right direction (proper C4 notation, AWS icons, legend), but must preserve the architectural boundaries an instructor or peer will check first: public traffic enters through WAF, money movement reaches CBS only through the CBS Transaction Gateway, ODS is not a second system of record, and the AI Advisor receives only minimized context.

### Fix 1 — Remove the duplicate Read Models / ODS boundary
- **Delete** `n0soMGVSXSvGg1exUm9i-21` (the lower copy at y=2020, h=360, w=679).
- **Keep** `n0soMGVSXSvGg1exUm9i-19` (y=1620). This is the canonical ODS container boundary.

### Fix 2 — Move Person shapes outside the system scope
In C4, Persons are external actors — they sit **outside** the system boundary. Right now `Mobile Banking` (`WFdSSb0orDeSZbyKRULI-83`) and `Web Banking` (`WFdSSb0orDeSZbyKRULI-82`) are inside the Digital Channels SystemScopeBoundary.
- **Move** Mobile Banking to x=20, y=1600.
- **Move** Web Banking to x=20, y=1845.
- Keep their `mxgraph.c4.person2` shape (they're correctly C4 Person types).

### Fix 3 — Move Open Banking TPPs out of the Digital Channels scope
TPPs are an **External Software System**, not part of "Digital Channels". The current position puts them inside our boundary.
- **Move** `WFdSSb0orDeSZbyKRULI-81` (Open Banking TPPs) to x=20, y=2080. C4 style: draw external systems outside the main boundary with a labeled arrow crossing in.
- TPPs must point to **WAF / DDoS** only. Do **not** draw a direct TPP → Consent Service edge; consent traffic still enters through WAF → API Gateway → BFF → Consent Service.

### Fix 4 — Fix the FE-DMZ sub-boundary
`WFdSSb0orDeSZbyKRULI-101` (FE-DMZ, x=680, y=1560, w=440, h=220) is positioned exactly where its parent `Regional Cloud Environment` (`WFdSSb0orDeSZbyKRULI-84`, x=680, y=1560, w=440, h=880) starts. It visually collides with the parent.
- **Recommended: Delete** the FE-DMZ sub-boundary. The parent Regional Cloud Environment is enough; WAF and API Gateway are the only "DMZ" components and they sit at the top of the cloud scope.
- Alternative: resize FE-DMZ to h=180 and nudge it down by 20px so it sits inside the parent and encloses only WAF and API Gateway icons.

### Fix 5 — Add the missing on-prem containers
The C4 diagram omits **Audit / Compliance Store**, **On-Prem Observability**, and the **Advisor Context API**. These are first-class architectural boundaries.
- **Add** an `Audit / Compliance Store` Container inside the New On-Premises boundary (e.g. x=2600, y=1700, technology `Append-only log / object storage`).
- **Add** an `On-Prem Observability` Container inside the New On-Premises boundary (e.g. x=2600, y=1860, technology `Prometheus + Loki + Tempo`).
- **Add** an `Advisor Context API` Container inside the New On-Premises boundary. This service authorizes and minimizes customer context before the cloud AI Advisor can use it.
- These containers belong inside the New On-Premises boundary (`n0soMGVSXSvGg1exUm9i-108`, x=1202, y=1560, w=2149, h=880).

### Fix 6 — Add / correct the missing edges
A complete C4 needs the following edges. Source/target IDs are from the .drawio where known; draw.io will route them.

| # | Source | Target | Label | Technology |
|---|---|---|---|---|
| 1 | `n0soMGVSXSvGg1exUm9i-5` (DB2) | `n0soMGVSXSvGg1exUm9i-16` (DB2 Adapter) | CDC / replication | internal |
| 2 | `n0soMGVSXSvGg1exUm9i-6` (ADABAS) | `n0soMGVSXSvGg1exUm9i-17` (ADABAS Adapter) | CDC / replication | internal |
| 3 | `n0soMGVSXSvGg1exUm9i-16` (DB2 Adapter) | `n0soMGVSXSvGg1exUm9i-18` (Event Bus) | publishes | Kafka |
| 4 | `n0soMGVSXSvGg1exUm9i-17` (ADABAS Adapter) | `n0soMGVSXSvGg1exUm9i-18` (Event Bus) | publishes | Kafka |
| 5 | `n0soMGVSXSvGg1exUm9i-18` (Event Bus) | `n0soMGVSXSvGg1exUm9i-20` (Data Stores / ODS) | projects | Kafka consumer |
| 6 | `n0soMGVSXSvGg1exUm9i-9` (Account Info) | `n0soMGVSXSvGg1exUm9i-20` (Data Stores) | reads | SQL |
| 7 | `n0soMGVSXSvGg1exUm9i-13` (Report) | `n0soMGVSXSvGg1exUm9i-20` (Data Stores) | reads | SQL |
| 8 | `n0soMGVSXSvGg1exUm9i-14` (Consent) | `n0soMGVSXSvGg1exUm9i-20` (Data Stores) | reads | SQL |
| 9 | `WFdSSb0orDeSZbyKRULI-106` (AI Advisor) | `Advisor Context API` (new) | requests minimized context | mTLS |
| 10 | `Advisor Context API` (new) | `n0soMGVSXSvGg1exUm9i-14` (Consent) | checks authorization scope | mTLS |
| 11 | `Advisor Context API` (new) | `n0soMGVSXSvGg1exUm9i-20` (Data Stores) | reads authorized projections | SQL |
| 12 | `n0soMGVSXSvGg1exUm9i-12` (Offline Fraud) | `n0soMGVSXSvGg1exUm9i-20` (Data Stores) | reads | SQL |
| 13 | `n0soMGVSXSvGg1exUm9i-12` (Offline Fraud) | `n0soMGVSXSvGg1exUm9i-11` (Real-Time Fraud) | publishes rules | internal |
| 14 | `n0soMGVSXSvGg1exUm9i-15` (CBS Gateway) | (Audit — new) | writes | append-only |
| 15 | `n0soMGVSXSvGg1exUm9i-10` (Transfer) | (Audit — new) | writes | append-only |
| 16 | `n0soMGVSXSvGg1exUm9i-11` (Real-Time Fraud) | (Audit — new) | writes | append-only |
| 17 | All on-prem Containers | (On-Prem Observability — new) | emits | OTel |
| 18 | (On-Prem Observability — new) | `WFdSSb0orDeSZbyKRULI-99` (Cloud Observability) | ships | mTLS |
| 19 | `n0soMGVSXSvGg1exUm9i-18` (Event Bus) | `n0soMGVSXSvGg1exUm9i-12` (Offline Fraud) | feeds | Kafka |

Do **not** add:

```text
TPPs -> Consent Service
AI Advisor -> ODS
```

Both are boundary leaks. TPPs enter through WAF/API Gateway, and the AI Advisor must request minimized context through the Advisor Context API.

### Fix 7 — Add DR / HA markers
99.999% is one of the headline quality attributes (NFR-AR-010). Make it visible in one glance.
- Add a `‖` or `(replica)` annotation to: `n0soMGVSXSvGg1exUm9i-15` (CBS Gateway), `n0soMGVSXSvGg1exUm9i-18` (Event Bus), `n0soMGVSXSvGg1exUm9i-20` (Data Stores). Or draw dashed replica Containers beside each.
- A short `DR Site` note in the top-right of the C4 view ties it to the LLM top diagram's Disaster Recovery site.

### Style notes for presentation export
- Export to **PNG at 1920×1080** for slide embedding (File → Export As → PNG, zoom 200%). The C4 with all edges reads well at that size.
- Keep the AWS icon style — it matches the team's Java+AWS background (deck p.15; AGENTS.md course constraints).
- Keep the C4 legend visible; it saves explaining notation on stage.

## Part 2 — Slide outline (target: 6–8 slides, ~10 min)

| # | Title | What to show | Source | Time | Speaker notes |
|---|---|---|---|---|---|
| 1 | **Title** | Project name, your name, date, course tag | — | 0:30 | "I'm [name], this is the MS1 preliminary solution for NeoBank Digital Leap, the Software Architecture course final project." |
| 2 | **Context & drivers** | The bank (1M customers, founded 1905, mainframe CBS); the disruption (fintech, Open Banking, regulator-forced credit-card spinoff); the 3 business + 6 technical objectives | `../Software Architecture - Lesson 1B Deck - Final Project - NeoBank.pdf` p.6–11 | 1:30 | "The bank has a working COBOL CBS that they cannot replace. They need to add digital channels without disturbing the core ledger. Cost pressure is real — every CBS call is expensive." |
| 3 | **Goals & quality attributes** | 99.999%, 100K→1M users, 1-year MVP, GDPR regional, Java+AWS team | `00-hld-introduction.md` Abstract; `01-ms1-preliminary-solution.md` §2; deck p.15 | 1:00 | "These are the non-negotiables. The architecture is shaped by them — particularly 99.999% and the 1-year deadline, which together rule out any big-bang CBS replacement." |
| 4 | **Architecture style** | Three decisions, each with a one-line rationale: Strangler Fig + CQRS + hybrid on-prem/cloud. The 7 accepted decisions (D-001..D-007). | `ADR-001`, `ADR-002`; `01-ms1-preliminary-solution.md` §3; `project-status.md` Current Decisions | 1:30 | "Strangler Fig means we build around the CBS, not against it. CQRS means reads don't touch CBS — we use read models. Hybrid means regulated data stays on-prem, elastic services run in the cloud." |
| 5 | **C4 Container view** | The main C4 diagram after the cleanup spec. Walk left-to-right: people, WAF, APIGW, BFF, AI, Advisor Context API, on-prem services, CBS, data stores. | `docs/diagrams/neobank-digital-leap-c4.drawio`, `docs/diagrams/07-c4-container.mmd` | 3:00 | "Three things to notice: WAF is the only public entry point; the CBS Transaction Gateway is the only path to the mainframe; the AI advisor reads through a minimized context boundary, never from CBS or directly from the ODS. Those three rules protect the bank." |
| 6 | **Key flows** | Three small flow diagrams: account info, bank transfer, AI advisor. 30 sec each. | `docs/diagrams/02-account-information-flow.mmd`, `03-bank-transfer-flow.mmd`, `05-ai-financial-advisor-flow.mmd` | 1:30 | "These are the three use cases the deck calls out. Each has a one-sentence design rule. Account info: read from the ODS with freshness metadata. Transfer: real-time fraud, then CBS Gateway. AI: cloud-only, data-minimized." |
| 7 | **Decisions & risks** | 7 accepted decisions; top 3 risks (CBS integration, data freshness, ODS-as-second-SoR); on-prem ODS shortlist (Oracle/SQL Server/Db2 — vendor deferred to MS2). | `project-status.md` Current Decisions; `07-risks-and-mitigations.md`; `02-ods-database-recommendation.md` | 1:30 | "We've made the major decisions. The two open ones are the ODS vendor — Oracle, SQL Server, or Db2, all viable for a conservative bank — and the cloud region for the AI advisor." |
| 8 | **Next steps & questions** | MS2 plan (security, network topology, performance, data flow diagrams); 11 open questions; submission timeline (MS2 Week 13, Final end of course). | `project-status.md` Next Work; `08-open-issues.md`; deck p.17 | 0:30 | "For MS2 we add four diagrams and resolve the open issues. Final HLD by end of course. Questions?" |

Total: ~10 min, 8 slides. Trim to 6 by combining 2+3 and 7+8 if running short.

### Backup / Q&A material
- **Why not Aurora / PostgreSQL for the ODS?** → `02-ods-database-recommendation.md`. Aurora is cloud-managed; PostgreSQL fights the conservative bank culture.
- **Why is the AI advisor in the cloud?** → Deck p.14: "only this service can run on the cloud and not on-prem"; Advisor Context API and data minimization ensure it never touches CBS and never reads the ODS directly (FN-120).
- **What's the 24-hour account staleness?** → `NFR-PC-060`; deck p.15. Allows read models to be eventually consistent within that bound.
- **What's the disaster recovery story?** → 99.999% maps to active-active or active-passive on critical layers, durable event bus with replay, idempotent transfer commands (NFR-DI-040).
- **How does GDPR work?** → NFR-GDPR-010 (regional processing), NFR-GDPR-020 (right-to-delete), NFR-GDPR-030/040 (financial records retained, personal data deletable).
- **Where do network diagrams fit?** → `docs/diagrams/06-network-topology.mmd` is the MS2 scaffold (teacher-mandated). MS1 doesn't require it.
