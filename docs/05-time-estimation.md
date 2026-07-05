# NeoBank HLD — Time Estimation (HLD §4)

> Required by the HLD template. Per-subsystem / per-team workdays for the
> one-year MVP. Team profile (deck page 15): ~60 developers, mostly Java +
> AWS; 5 COBOL developers. Status: **scaffold — to be filled during MS2.**

## Team profile

| Skill | Headcount | Notes |
|---|---|---|
| Java + AWS | ~55 | Primary digital service build |
| Frontend / mobile | _to be sized_ | Channel team |
| Security / IAM | _to be sized_ | |
| DevOps / SRE | _to be sized_ | |
| DBA | _to be sized_ | On-prem ODS support |
| Mainframe / COBOL | 5 | CBS integration only |
| Product / Design / QA | _to be sized_ | |

## Subsystem workdays (MVP, 1 year)

| Subsystem / team | Workdays | Headcount | Calendar duration | Notes |
|---|---|---|---|---|
| Mobile app | | | | |
| Web app | | | | |
| API Gateway + BFF | | | | |
| Customer IAM | | | | |
| Account Information Service | | | | |
| Transfer Service | | | | |
| Real-Time Fraud Engine | | | | |
| Offline Fraud Analytics | | | | |
| Income / Expense Report Service | | | | |
| Open Banking API + Consent | | | | |
| AI Financial Advisor | | | | |
| CBS Transaction Gateway | | | | Critical path — coordinates with COBOL team |
| DB2 / ADABAS adapters + event bus | | | | |
| ODS implementation (RDBMS) | | | | Vendor decision blocks this |
| Audit / Compliance Store | | | | |
| Observability (cloud + on-prem) | | | | |
| Security baseline (WAF, mTLS, secrets) | | | | |
| On-prem infra (HW + VM + OS) | | | | |
| Migration / cutover / DR drills | | | | |
| Documentation, training, handover | | | | |

## Critical path

The CBS Transaction Gateway and the ODS platform decision are the two items most likely to gate downstream work. Track them explicitly.

## Notes

- All workdays assume 1 year of calendar time, with team sizes from the team profile table.
- The template uses Workdays as a unit; the final HLD can also include dependency arrows or a Gantt if the Word tool supports it.
- Update this table as the MS2 work-breakdown structure stabilizes.
