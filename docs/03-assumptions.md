# NeoBank HLD — Assumptions

> **Project working file.** The HLD template does not define a standalone
> Assumptions top-level section — it places assumptions inside §2 Requirements.
> This file is a working convenience for the MVP: assumptions are consolidated
> into the Requirements section during final HLD assembly. Seed assumptions
> already exist in [`01-ms1-preliminary-solution.md`](01-ms1-preliminary-solution.md)
> §9 and are reproduced below; consolidate or update IDs as the HLD evolves.

## Assumptions

| ID | Assumption | Source |
|---|---|---|
| A-001 | CBS remains the system of record for financial transactions during MVP. | MS1 §9 |
| A-002 | The bank can deploy a new on-premises digital platform close to CBS. | MS1 §9 |
| A-003 | The cloud region is legally acceptable for selected digital services and AI advisor workloads. | MS1 §9 |
| A-004 | DB2 and ADABAS data can be ingested through adapters, CDC, batch replication, or controlled export mechanisms. | MS1 §9 |
| A-005 | Some account information can be stale up to 24 hours for incoming transactions from other banks or payments not made through the app. | MS1 §9 / deck p.15 / NFR-PC-060 |
| A-006 | The bank already has enterprise database operational standards for Oracle, SQL Server, or Db2. | MS1 §9 / ADR-002 |
| A-007 | The MVP can avoid major COBOL changes by introducing a CBS Transaction Gateway and adapter layer. | MS1 §9 |
| A-008 | Team profile is ~60 Java + AWS developers with 5 COBOL developers; CBS remains COBOL. | deck p.15 |
| A-009 | MVP must launch within one year (deck p.13 / p.15). | deck |
| A-010 | All 1M target users are in the same geographic region within three years (NFR-GDPR-010). | deck p.15 |
| A-011 | Peak concurrency is ~5% of registered digital users (5K concurrent sessions at 100K users, 50K at 1M). Drives NFR-PC-015 and the sizing in HLD §3.6. | working estimate — validate in MS2 |

## Notes

- New assumptions should use the next free ID (`A-NNN`).
- When the assumption is also captured as a requirement, cross-reference the requirement ID.
- When the assumption is invalidated, mark it `Superseded` and link to the replacement.
