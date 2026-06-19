# Project Status

## Current Layer

**Software Architecture course → NeoBank Digital Leap → High Level Design artifact.**

This repository currently focuses on MS1 and early HLD structure. No implementation code is required for the course submission.

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
| `docs/00-hld-introduction.md` | HLD abstract and document introduction. |
| `docs/01-ms1-preliminary-solution.md` | MS1 preliminary architecture narrative. |
| `docs/02-ods-database-recommendation.md` | ODS technology recommendation for on-premises bank context. |
| `docs/requirements/requirements-draft.md` | Initial functional and non-functional requirements. |
| `docs/glossary.md` | Common project terminology. |
| `docs/diagrams/*.mmd` | Mermaid diagram sources. |
| `docs/decisions/*.md` | Architecture Decision Records. |

## Next Work

1. Review and refine the MS1 high-level solution diagram.
2. Convert Mermaid diagrams into Word-ready rendered diagrams when needed.
3. Start MS2 security boundary diagram.
4. Start MS2 data flow diagram.
5. Start MS2 performance and scaling diagram.
6. Expand HLD assumptions, limitations, risks, and open issues.
7. Convert selected Markdown files into the final Word HLD structure.

## Notes

The repository is intentionally Markdown-first. The final course submission can be assembled into a Word document after the architecture narrative and diagrams stabilize.
