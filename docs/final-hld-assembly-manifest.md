# Final HLD Assembly Manifest

## Deliverable

- Title: NeoBank Digital Leap — Final High Level Design
- Author: Yiannis Miliaresis
- Format: one Word document with embedded diagrams
- Body order: Lesson 1B sections 1–7
- Appendices: exactly four
- Diagrams: exactly eighteen
- Page target: 75–95 readable pages with no blank transition pages

## Main-body sources

1. General — `docs/00-hld-introduction.md` and `docs/glossary.md`
2. Requirements — `docs/requirements/requirements-draft.md` and `docs/03-assumptions.md`
3.1 System Architecture — `docs/09-system-architecture.md`
3.2 Design Rules and Cross-Cutting Architecture — `docs/10-design-rules-and-cross-cutting.md`
3.3 High Level System Flows — `docs/11-system-flows.md`
3.4 Managed API and Event Contracts — `docs/12-managed-contracts.md`
3.5 Upgradability — `docs/13-upgradability.md`
3.6 Sizing — `docs/04-sizing.md`
4. Time Estimation — `docs/05-time-estimation.md`
5. Limitations — `docs/06-limitations.md`
6. Risks and Mitigations — `docs/07-risks-and-mitigations.md`
7. Open Issues — `docs/08-open-issues.md`

## Appendix sources

- Appendix A — `docs/appendices/a-requirements-traceability.md`
- Appendix B — `docs/appendices/b-contract-catalog.md`
- Appendix C — `docs/appendices/c-sizing-capacity-tco.md`
- Appendix D — `docs/appendices/d-delivery-estimation.md`

## Diagram sources

All files `docs/diagrams/01-*.mmd` through `docs/diagrams/18-*.mmd` are embedded. Figure captions MUST use the document reading order below. Source filename numbers identify governed diagram assets; they MUST NOT determine the displayed figure number.

| Displayed figure | Governed source | Caption |
|---:|---|---|
| 1 | `01-high-level-solution.mmd` | High-Level Solution |
| 2 | `07-c4-container.mmd` | C4 Container View |
| 3 | `06-network-topology.mmd` | Network Topology |
| 4 | `12-security-trust-boundaries.mmd` | Security Trust Boundaries |
| 5 | `13-resilience-dr-topology.mmd` | Resilience and DR Topology |
| 6 | `14-observability-evidence.mmd` | Observability and Evidence View |
| 7 | `08-authentication-public-ingress.mmd` | Authentication and Public Ingress |
| 8 | `02-account-information-flow.mmd` | Account Information Flow |
| 9 | `03-bank-transfer-flow.mmd` | Bank Transfer Flow |
| 10 | `09-open-banking-flow.mmd` | Open Banking Flow |
| 11 | `04-data-ingestion-read-model-flow.mmd` | Data Ingestion and Read-Model Flow |
| 12 | `10-monthly-statement-flow.mmd` | Monthly Statement Flow |
| 13 | `05-ai-financial-advisor-flow.mmd` | AI Financial Advisor Flow |
| 14 | `11-privileged-operation-audit-flow.mmd` | Privileged Operation and Audit Flow |
| 15 | `15-release-promotion-evidence.mmd` | Release Promotion and Evidence Flow |
| 16 | `16-zero-downtime-upgrade-recovery.mmd` | Zero-Downtime Upgrade and Recovery |
| 17 | `17-performance-capacity.mmd` | Performance and Capacity View |
| 18 | `18-delivery-roadmap.mmd` | Delivery Roadmap |

## Supporting evidence only

The following sources inform the HLD but MUST NOT be assembled as duplicate chapters:

- `docs/01-ms1-preliminary-solution.md`
- `docs/02-ods-database-recommendation.md`
- `docs/c4-model.md`
- `docs/decisions/ADR-*.md`

## Rendering contract

- Render SVG first.
- Produce a verified 300-DPI PNG fallback.
- Place each diagram with a numbered caption and alternative text.
- Allocate one page to a diagram when necessary for approximately 9-point minimum visible labels.
- Inspect every final Word/PDF page for clipping, overlap, broken tables, missing glyphs and illegible labels.
