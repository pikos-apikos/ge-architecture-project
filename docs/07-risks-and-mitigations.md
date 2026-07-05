# NeoBank HLD — Risks and Mitigations (HLD §6)

> Required by the HLD template. MS1 seed content is reproduced from
> [`01-ms1-preliminary-solution.md`](01-ms1-preliminary-solution.md) §10;
> expand during MS2.

## Risk register

| ID | Risk | Impact | Likelihood | Mitigation | Owner |
|---|---|---|---|---|---|
| R-001 | CBS integration bottleneck | MVP delay and runtime instability | Medium | Isolate CBS calls through CBS Transaction Gateway; idempotency, throttling, contract testing. | Mainframe / Architecture |
| R-002 | Data freshness confusion | Customer trust issues | High | Display freshness metadata; clearly define stale-data rules; NFR-PC-060 (24h). | Product / UX |
| R-003 | Fraud false positives | Customer friction and support cost | Medium | Rule tuning, offline analytics, explainability, manual review paths. | Fraud / Risk |
| R-004 | Cloud / on-prem latency | Poor UX | Medium | BFF aggregation, caching, regional deployment, colocation of on-prem services for critical flows. | Architecture / SRE |
| R-005 | Regulatory constraints on AI advisor | Compliance risk | Medium | Data minimization, consent, regional processing, audit trails, advice disclaimers. | Legal / Compliance |
| R-006 | ODS becomes a second system of record | Financial correctness risk | Medium | Enforce design rule: CBS authoritative for money movement; ODS is read-only projection. ADR-001. | Architecture / DBA |
| R-007 | Cloud region / regulator rejection | Cannot launch AI advisor in cloud | Low | Fall back to on-prem placement for AI advisor if regional approval fails; design data minimization so re-platforming is feasible. | Legal / Compliance |
| R-008 | Vendor lock-in (Oracle / SQL Server / Db2) | Long-term cost and migration risk | Low | Keep ODS schemas portable; avoid vendor-specific features in the projection layer. ADR-002. | Architecture / DBA |
| R-009 | Team skill gap (COBOL, 5 devs) | Slow CBS change cycles | Medium | Minimize CBS changes; isolate legacy integration behind adapters; ADR-001 design rule. | Mainframe / Architecture |
| R-010 | Open Banking regulatory change | API contract change | Low | API versioning (NFR-DEP-040, NFR-BC-010, NFR-BC-020); consent service as a stable boundary. | Product / Compliance |
| R-011 | Cost overrun on cloud / on-prem | Budget pressure | Medium | Right-size during MS2; cap cloud egress; prefer managed services that match team skills; revisit sizing annually. | Finance / Architecture |
| R-012 | Operational cost calculation is wrong | Submission rejection | Low | Use the AWS and Azure pricing calculators as a sanity check; show assumptions explicitly. Deck page 15. | Architecture |

## Notes

- Risk IDs are stable; do not renumber after first revision. Add between intervals (e.g. R-011a).
- Cross-reference ADRs, NFRs, and Open Questions where relevant.
- The template's Risks and Mitigations table is the source of truth for the final Word HLD.
