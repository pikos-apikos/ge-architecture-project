# NeoBank HLD — Assumptions

## Assumptions

| ID | Assumption | Source |
|---|---|---|
| A-001 | CBS remains the system of record for financial transactions during MVP. | MS1 §9 |
| A-002 | The bank can deploy a new on-premises digital platform close to CBS. | MS1 §9 |
| A-003 | The cloud region is legally acceptable for selected digital services and AI advisor workloads. | MS1 §9 |
| A-004 | Db2 and ADABAS data can be ingested through adapters, CDC, batch replication, or controlled export mechanisms. | MS1 §9 |
| A-005 | Some account information can be stale up to 24 hours for incoming transactions from other banks or payments not made through the app. | MS1 §9 / deck p.15 / NFR-PC-060 |
| A-006 | The bank already has enterprise database operational standards for Oracle, SQL Server, or Db2. | MS1 §9 / ADR-002 |
| A-007 | The MVP can avoid major COBOL changes by introducing a CBS Transaction Gateway and adapter layer. | MS1 §9 |
| A-008 | Team profile is ~60 Java + AWS developers with 5 COBOL developers; CBS remains COBOL. | deck p.15 |
| A-009 | MVP must launch within one year (deck p.13 / p.15). | deck |
| A-010 | All 1M target users are in the same geographic region within three years (NFR-GDPR-010). | deck p.15 |
| A-011 | Peak concurrency is ~5% of registered digital users (5K concurrent sessions at 100K users, 50K at 1M). Drives NFR-PC-015 and the sizing in HLD §3.6. | working estimate — validate in MS2 |
| A-012 | Peak traffic, request amplification, event size, retention, storage, and compute-unit values in Appendix C are planning values and will be load-tested in Phase 0. | Issue #8 approved sizing model |
| A-013 | Direct Connect is the preferred cloud-to-on-premises link, with diverse circuits and VPN backup; exact carrier, facility, bandwidth, routing, and encryption design remain Phase-0 validation items. | Issue #7 / OI-004 |
| A-014 | The Enterprise RDBMS ODS vendor will be selected from Oracle, SQL Server, or Db2 using existing bank licenses, support, skills, CDC, HA/DR, capacity, and cost evidence. | ADR-002 / OI-002 |
| A-015 | A governed external AI inference service can be approved for regional use; legal, privacy, security, procurement, and model-risk review can take two to six weeks. | Issue #10 external-dependency scenario |
| A-016 | AI coding agents are introduced through a measurable, governed rollout after an architect creates the agentic workflow and evidence model during weeks 1–10. | Issue #10 work package WS12.1 |
| A-017 | AI coding-agent productivity uplift is zero in the funded baseline; 5%, 10%, and 20% are sensitivity cases applied only to eligible work after measured evidence. | Issue #10 |
| A-018 | Non-production environments can be stopped outside agreed working hours, except for a justified, approved, time-bound override. | Issue #9 |
| A-019 | The operational cost values are dated planning bands and exclude business-as-usual costs that the bank already carries unless the architecture creates an incremental charge. | Issue #8 |
| A-020 | The final delivery plan uses P75 effort plus a separate 10% management reserve; external approval delays are modeled as elapsed-time scenarios, not hidden person-day padding. | Issue #10 |
