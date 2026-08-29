# 5. Limitations

| ID | Limitation | Consequence / boundary |
|---|---|---|
| L-001 | The HLD is not a low-level design or implementation specification. | Component internals, complete OpenAPI/AsyncAPI, physical schemas, code and product configuration are deferred. |
| L-002 | CBS and legacy protocols remain in place for the MVP. | Gateway and reconciliation controls reduce coupling but do not remove mainframe cost or change risk. |
| L-003 | ODS is non-authoritative. | Digital reads can be stale; money movement and authoritative balance decisions remain CBS-led. |
| L-004 | The 24-hour staleness tolerance is narrow. | It applies only to incoming external or out-of-app activity and does not weaken digital transfer read-your-writes. |
| L-005 | ODS vendor is not selected. | Oracle, SQL Server, and Db2 remain the approved shortlist until Phase-0 evidence selects one. |
| L-006 | Exact network design is not fixed. | CIDRs, ports, carrier, bandwidth, firewall objects and routing require detailed design. |
| L-007 | AI advice is non-binding. | AI cannot decide eligibility, terms, disclosures, or execute a sale. |
| L-008 | AI availability depends on external approval and provider service. | Core banking continues without AI; the delivery scenario may extend by external lead time. |
| L-009 | Cost values are planning bands. | Procurement quotes, licenses, discounts, taxes and actual facilities rates can materially change TCO. |
| L-010 | Effort values are reference-class estimates. | Phase 0 must recalibrate team capacity, three-point inputs, specialist availability and dependencies. |
| L-011 | AI coding-agent uplift is not funded as a saving. | Productivity benefit is recognized only after a measurable governed pilot. |
| L-012 | Non-production cost saving uses scheduled shutdown. | Justified tests or support work require an approved, time-bound override. |
| L-013 | Recovery promotion is human-governed for RC0. | Financial-authority DR cannot be fully automatic. |
| L-014 | Offline fraud findings do not trigger automatic reversal. | A controlled case, human decision and governed compensation path are required. |
| L-015 | Final regulatory and retention parameters are unresolved. | Phase-0 legal/compliance evidence gates production readiness. |

