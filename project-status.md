# Project Status

## Current Layer

**Software Architecture course → NeoBank Digital Leap → High Level Design artifact.**

This repository currently focuses on MS1 and early HLD structure. No implementation code is required for the course submission.

Submission timeline (deck page 17): **MS1 by Week 7**, **MS2 by Week 13**, **Final HLD by end of course**. Submission email subject: `Preliminary/Final Submission – SA Course [MM]-[YYYY] - [your full name]`.

## Current Decisions

| ID | Decision | Status |
|---|---|---|
| D-001 | Keep CBS as system of record for all financial transactions. | Accepted |
| D-002 | Use a CBS Transaction Gateway as the only controlled transaction execution boundary. | Accepted |
| D-003 | Use read models / ODS for digital read workloads. | Accepted |
| D-004 | Use an on-premises Enterprise RDBMS ODS shortlist: Oracle, SQL Server, Db2. | Accepted for MS1; vendor deferred |
| D-005 | Keep AI Financial Advisor in cloud-only scope but restrict it to minimized, authorized data. | Accepted |
| D-006 | Route all public client traffic through WAF/DDoS protection before API Gateway. | Accepted |
| D-007 | Treat Offline Fraud Analytics as asynchronous analytics/batch risk scoring, not as a synchronous transfer-path service. | Accepted |

## Current Files

| File | Purpose |
|---|---|
| `README.md` | Repository overview and architecture direction. |
| `project-status.md` | This file. Current decisions and milestone queue. |
| `AGENTS.md` | Onboarding for future OpenCode / agent sessions — repo rules, invariants, conventions. |
| `docs/00-hld-introduction.md` | HLD master, aligned to the Lesson 1B template section order. |
| `docs/01-ms1-preliminary-solution.md` | MS1 preliminary architecture narrative. |
| `docs/02-ods-database-recommendation.md` | ODS technology recommendation for on-premises bank context. |
| `docs/03-assumptions.md` | Project working file (HLD §2; template folds assumptions into Requirements). Seeded from MS1 §9. |
| `docs/04-sizing.md` | HLD §3.6 — on-prem hardware/software sizing + operational cost. |
| `docs/05-time-estimation.md` | HLD §4 — per-subsystem workdays. |
| `docs/06-limitations.md` | HLD §5 — out-of-scope and architectural limitations. |
| `docs/07-risks-and-mitigations.md` | HLD §6 — risk register. Seeded from MS1 §10. |
| `docs/08-open-issues.md` | HLD §7 — open questions. Seeded from MS1 §11. |
| `docs/c4-model.md` | C4 model layer description. |
| `docs/glossary.md` | Common project terminology. |
| `docs/requirements/requirements-draft.md` | Functional and non-functional requirements (`FN-NNN`, `NFR-<CAT>-NNN`). |
| `docs/diagrams/*.mmd` | Mermaid diagram sources. |
| `docs/decisions/*.md` | Architecture Decision Records. |
| `../Software Architecture - Lesson 1B - Architecture_HLD_Template.docx.md` | Course HLD template (Lesson 1B). Authoritative section order. |
| `../Software Architecture - Lesson 1B Deck - Final Project - NeoBank.pdf` | Course brief (Lesson 1B). Original requirements and submission rules. |

## Next Work

1. Review and refine the MS1 high-level solution diagram.
2. Convert Mermaid diagrams into Word-ready rendered images and embed in the final HLD Word file.
3. Start MS2 security boundary diagram.
4. **Start MS2 network topology diagram.** Scaffolded at `docs/diagrams/06-network-topology.mmd`. Required by Jukka Rohila (class guidance). Must include: cloud VPC public/private subnets, on-prem DMZ / regulated / mainframe zones, WAF and NGFW placements, cloud-to-on-prem link (VPN or Direct Connect — decision deferred), CIDR blocks, ports/protocols, and HA/DR topology per zone. Must show the CBS Transaction Gateway as the only path to the mainframe.
5. Start MS2 data flow diagram.
6. Start MS2 performance and scaling diagram.
7. Fill in the scaffolded HLD sections: `04-sizing.md` (HW/SW sizing + cost), `05-time-estimation.md` (workdays), `06-limitations.md`, `08-open-issues.md`. `07-risks-and-mitigations.md` is partly seeded from MS1 §10 and needs MS2 expansion.
8. Define message schemas and API contracts (HLD §3.4) and upgradability patterns (HLD §3.5).
9. Select the on-prem ODS vendor from the Oracle / SQL Server / Db2 shortlist (ADR-002).
10. Resolve open issues OI-001..OI-011.
11. Consolidate duplicate content: assumptions, risks, and open questions currently live in both MS1 §9/§10/§11 and `03..08` files — pick one location and deprecate the other.
12. Assemble the final Word HLD from the Markdown sources per the template's section order.

## Notes

The repository is intentionally Markdown-first. The final course submission is a **single Word file with diagrams embedded** (deck page 16) — Markdown here is authoring convenience. The Lesson 1B deck and HLD template (`../`) are authoritative: if any document in `docs/` contradicts the deck, the deck wins.

`AGENTS.md` is the onboarding file for future OpenCode / agent sessions working in this repo.
