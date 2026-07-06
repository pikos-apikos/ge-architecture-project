# AGENTS.md

## What this repo is
Documentation-only Software Architecture course final project: **NeoBank Digital Leap** — a high-level design for a retail bank's digital transformation MVP. Author: Yiannis Miliaresis. There is **no source code, no build, no tests, no lint, no package manager**. Everything is Markdown during authoring; **the final course submission is a single Word file with diagrams embedded** (deck, page 16: "Diagrams should be integrated in the Word file – please submit one file"). Course reference material lives one level up at `/Users/pikos/Projects/Training/` (HLD template `.docx.md` and the Lesson 1B PDF deck).

## Working rules
- English and Markdown only — see `README.md` "Working Rule".
- Do not introduce code, test runners, CI config, or `package.json` / `pom.xml` / `requirements.txt` etc. There is nothing to install or execute.
- Do not run `npm` / `pip` / `mvn` / `gradle` / `dotnet` — they will fail and are not needed.
- "Executable truth" here is the `.mmd` Mermaid diagrams in `docs/diagrams/`, the ADRs in `docs/decisions/`, and the **Lesson 1B deck + HLD template** at `../`. The deck is the original brief — if `docs/` contradicts the deck, the deck wins. If prose and a diagram disagree, the diagram is authoritative.
- Treat `project-status.md` as the source of truth for current decisions and next work.

## Course / submission constraints (from the Lesson 1B deck)
- **Single Word file deliverable** with diagrams embedded. Mermaid `.mmd` is authoring convenience only; before final submission, render diagrams (Mermaid CLI → PNG/SVG) and embed them in the Word file.
- **Milestone weeks**:
  - **Week 7** — MS1: preliminary solution diagram.
  - **Week 13** — MS2: HLD + Data + Security + Performance diagrams.
  - **End of course** — Final HLD.
- **Submission email subject** (deck page 17):
  `Preliminary/Final Submission – SA Course [MM]-[YYYY] - [your full name]`
  sent to the instructor's email (not yet filled in — see deck).
- **On-prem must be specified in hardware AND software terms** (deck page 13, explicit: "You need to define the hardware and software aspects of it"). Not just architecture boxes — VMs, OS, sizing, HA/DR topology.
- **Operational cost calculation is required.** Deck points at `https://calculator.aws/#/` and `https://azure.microsoft.com/en-us/pricing/calculator/`. Include a sized cost estimate in the HLD's Sizing / Time Estimation area.
- **Team profile** (deck page 15): ~60 developers, mostly **Java + AWS** background, 5 COBOL. Bias new digital services toward the Java/AWS ecosystem; CBS stays COBOL/z/OS.

## Architectural invariants — do not violate
These are accepted decisions (`project-status.md` D-001..D-008, ADR-001, ADR-002). Any edit must preserve them:

- **CBS (COBOL on z/OS) is the sole system of record** for balances and money movement. All transfers go: Transfer Service → CBS Transaction Gateway → CBS. No digital channel may call CBS, DB2, or ADABAS directly (ADR-001 design rule).
- **Hybrid Strangler Fig + CQRS**: read-heavy digital experiences read from replicated projections / ODS. **ODS is never a second system of record** (ADR-001).
- **Public ingress order is WAF → API Gateway → IAM / BFF.** The high-level diagram in `docs/diagrams/01-high-level-solution.mmd` explicitly fixes this — do not redraw channels pointing straight at the API Gateway.
- **Open Banking TPPs must not bypass WAF / API Gateway / BFF.** Do not draw a direct TPP → Consent Service edge; consent flows still enter through the public ingress chain.
- **AI Financial Advisor runs in the regional cloud** and may **not** access CBS, DB2, ADABAS, or the ODS directly (FN-120). It receives only minimized, authorized data through the Advisor Context API / data-minimization boundary.
- **On-prem Enterprise RDBMS ODS** is the baseline — shortlist is **Oracle / SQL Server / Db2**. Vendor selection is deferred to MS2. Do not propose Aurora or PostgreSQL as the default on-prem ODS; ADR-002 explicitly rejects that for a conservative mainframe-based retail bank context.
- **Offline Fraud Analytics is asynchronous / batch** risk scoring, not a synchronous transfer-path service.
- **All money-moving commands are idempotent and auditable**, with correlation IDs spanning API Gateway → BFF → services → event bus → CBS gateway → audit log (NFR-OBS-040).
- **Regional data processing** for GDPR; right-to-delete applies to personal data but not to legally retained financial/audit records (NFR-GDPR-030, 040).
- **Account-info staleness tolerance is up to 24 hours** for incoming transactions from other banks or payments not done through the app (deck page 15). Read models can therefore be eventually consistent within that bound. Captured as `NFR-PC-060` in `docs/requirements/requirements-draft.md`.

## Layout (load-bearing files only)
```
README.md
project-status.md                   # current decisions (D-001..) + milestone queue
AGENTS.md                           # this file
docs/00-hld-introduction.md         # HLD master, aligned to template section order
docs/01-ms1-preliminary-solution.md  # MS1 narrative
docs/02-ods-database-recommendation.md
docs/03-assumptions.md              # project working file (template folds these into §2)
docs/04-sizing.md                   # HLD §3.6 Sizing — on-prem HW/SW + cost
docs/05-time-estimation.md          # HLD §4
docs/06-limitations.md              # HLD §5
docs/07-risks-and-mitigations.md    # HLD §6
docs/08-open-issues.md              # HLD §7
docs/c4-model.md
docs/glossary.md                    # authoritative terminology
docs/requirements/requirements-draft.md   # FN-* and NFR-<CAT>-* IDs
docs/ms1-presentation-prep.md      # MS1 talk prep: C4 cleanup spec + slide outline
docs/diagrams/*.mmd                 # Mermaid sources — edit these, not images
                                    # 01 high-level, 02-05 use-case flows,
                                    # 06 network topology (MS2 scaffold),
                                    # 07 C4 Container view (MS1)
docs/diagrams/neobank-digital-leap-c4.drawio  # MS1 C4 Container view (drawio, for presentation)
docs/decisions/ADR-*.md             # ADRs
```

## Conventions
- **Diagrams**: edit `.mmd` sources only. Render to PNG/SVG with the Mermaid CLI when a Word-ready image is needed for the final submission. The deck shows both top-down and left-to-right C4-style layouts — pick per diagram; high-level is left-to-right, flows can be top-down.
- **Network topology diagram** (`docs/diagrams/06-network-topology.mmd`) is an **MS2 teacher-mandated deliverable** (Jukka Rohila, class guidance), not an optional nice-to-have. It must show zones (cloud public/private subnets, on-prem DMZ, regulated Digital Core, mainframe), WAF and NGFW placements, the cloud-to-on-prem link (VPN or Direct Connect — decision deferred), and the CBS Transaction Gateway as the only path to the mainframe. The diagram is currently a scaffold; CIDR blocks, ports/protocols, and HA/DR topology are MS2 work.
- **ADRs**: existing structure is Status, Context, Decision, Consequences, Design Rules. New ones go in `docs/decisions/ADR-NNN-kebab-title.md` with the next free number.
- **Requirements IDs are stable**: `FN-NNN` (functional), `NFR-<CAT>-NNN` (categories from the HLD template: **AR** Availability/Recovery, **PC** Performance/Capacity, **SEC** Security, **BC** Backward Compatibility, **GDPR**, **OBS** Observability, **DEP** Deployment/Upgradability). The repo currently uses AR, DI Data Integrity, PC, SEC, BC, GDPR, OBS, DEP — keep `DI` as a project-specific category. When adding, pick the next free number in that category. Template also allows DF-NNN (Data Flow) and UF-NNN (User Flow) sub-id schemes.
- **Requirement wording**: use "the system shall …" (per template). Each requirement must be testable. After the first revision is published, never renumber existing IDs — insert between intervals (e.g. add FN-015 between FN-010 and FN-020).
- **HLD section order** (from the Lesson 1B template, when assembling the final Word HLD): **1 General (Introduction + Glossary) → 2 Requirements (FN + NFR) → 3 HLD (High Level System Diagram → Design Rules and Principles → High Level System Flows → Message Schemas → Upgradability → Sizing) → 4 Time Estimation → 5 Limitations → 6 Risks and Mitigations → 7 Open Issues.** The template does NOT have a standalone top-level "Assumptions" section — assumptions belong inside §2 Requirements — but `docs/03-assumptions.md` exists as a project working file for the MVP and is consolidated into §2 at final-assembly time. `docs/00-hld-introduction.md` is now aligned to this order.
- **Glossary**: define new acronyms in `docs/glossary.md` on first use; do not redefine them inline in body text.
- **Milestones**: MS1 in progress, MS2 (security / data / **network topology** / performance diagrams) not started, Final HLD not started. See `project-status.md` → "Next Work" for the current queue.

## Common agent mistakes to avoid
- Scaffolding code, adding a test runner, or writing a CI workflow.
- Suggesting Aurora or PostgreSQL as the on-prem ODS.
- Drawing public channels that skip WAF or hit the API Gateway directly.
- Drawing Open Banking TPPs directly to the Consent Service.
- Letting the ODS be treated as authoritative for balances.
- Letting the AI Financial Advisor talk to CBS / DB2 / ADABAS or read the ODS directly.
- Putting offline fraud analytics on the synchronous transfer path.
- Mixing languages or writing in Greek — the project is English-only by working rule.
- Editing the rendered diagram images instead of the `.mmd` source.
- Submitting the folder of Markdown files as the final deliverable — it must be **one Word file with diagrams embedded**. Render `.mmd` → image first.
- Omitting hardware/software specs for the on-prem digital platform, or skipping the operational cost calculation.
- Renumbering existing FN-/NFR- IDs instead of inserting between intervals, or using "must" / "should" instead of "shall" in requirement wording.
- Treating `docs/00-hld-introduction.md` as the final Word HLD submission — it is the Markdown stand-in aligned to the template's section order, but the final deliverable is still a single Word file with embedded diagrams (see "Course / submission constraints" above).
- Treating the network topology diagram as optional. It is **teacher-mandated** for the MS2 submission. Do not finalize the MS2 / Final HLD without it.
