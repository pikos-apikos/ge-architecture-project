# NeoBank HLD — Open Issues (HLD §7)

> Required by the HLD template. MS1 seed content is reproduced from
> [`01-ms1-preliminary-solution.md`](01-ms1-preliminary-solution.md) §11
> "Open Questions for MS2"; expand and resolve during MS2.

## Open issues

| ID | Issue / Question | Owner / Area | Status | Target resolution |
|---|---|---|---|---|
| OI-001 | Which cloud provider and region are approved by the bank and regulator? | Infrastructure / Compliance | Open | MS2 |
| OI-002 | Which enterprise RDBMS is already approved for on-premises ODS workloads? (Oracle / SQL Server / Db2) | DBA / Enterprise Architecture | Open | MS2 — ADR-002 vendor selection |
| OI-003 | What are the existing CBS integration mechanisms and throughput limits? | Mainframe Team | Open | MS2 |
| OI-004 | What is the allowed maximum latency for same-bank and external bank transfers? | Product / Risk | Open | MS2 |
| OI-005 | What Open Banking standard version must be supported? | Product / Compliance | Open | MS2 |
| OI-006 | What customer data may be used by the AI financial advisor? | Legal / Data Protection | Open | MS2 |
| OI-007 | What is the required RTO / RPO for each major component? (drives NFR-AR-020 and the sizing section) | Operations / Architecture | Open | MS2 |
| OI-008 | What is the CBS transaction contract version, and what is the deprecation / versioning policy for the CBS Transaction Gateway? (NFR-BC-030) | Mainframe / Architecture | Open | MS2 |
| OI-009 | What is the regulatory approval status of the regional cloud for AI advisor workloads? (A-003) | Legal / Compliance | Open | MS2 |
| OI-010 | What is the migration / cutover plan for the first 100K digital users? | Operations / Product | Open | MS2 |
| OI-011 | Is there an existing bank WAF / DDoS service to reuse, or do we provision a new one? | Infrastructure / Security | Open | MS2 |

## Notes

- Issue IDs are stable; do not renumber after first revision. Add between intervals.
- When resolved, mark `Resolved` and link to the decision (ADR, NFR, or sizing entry).
- The template's Open Issues table is the source of truth for the final Word HLD.
