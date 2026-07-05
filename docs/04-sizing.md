# NeoBank HLD — Sizing (HLD §3.6)

> Required by the HLD template. Covers on-premises hardware and software
> sizing, cloud sizing, units of scale, HA/DR topology, and an operational
> cost estimate (deck page 15: "present your calculation of operational costs",
> with the AWS and Azure pricing calculators as references).
> Status: **scaffold — to be filled during MS2.**

## 1. On-Premises Digital Core — Hardware and Software

> Deck page 13 is explicit: "You need to define the hardware and software
> aspects of" the new on-premises deployment. This section is therefore
> in-scope, not optional.

| Layer | Technology / Spec | Quantity | Notes |
|---|---|---|---|
| Compute — API / BFF tier | _to be sized_ | | VM count, vCPU, RAM, OS |
| Compute — Application services (Account, Transfer, Fraud RT, Report, Consent) | _to be sized_ | | |
| Compute — CBS Transaction Gateway | _to be sized_ | | Latency-sensitive, near-CBS |
| Compute — DB2 / ADABAS adapters + event workers | _to be sized_ | | |
| Storage — ODS (Enterprise RDBMS) | Oracle / SQL Server / Db2 — see [`02-ods-database-recommendation.md`](02-ods-database-recommendation.md) | | Vendor deferred to MS2 |
| Storage — Audit / Compliance Store | _to be sized_ | | Append-only, retention per regulator |
| Messaging — Durable Event Bus / Streaming | _to be sized_ | | Throughput target, partition count |
| Network — Internal segmentation | _to be sized_ | | Segregation between regulated and DMZ tiers |
| HA / DR topology | Active-passive / active-active per tier | | RTO / RPO per NFR-AR-020 |

## 2. Regional Cloud Digital Edge

| Layer | Technology / Spec | Quantity | Notes |
|---|---|---|---|
| WAF / DDoS Protection | _to be sized_ | | Public ingress — see ADR-001 design rule |
| API Gateway | _to be sized_ | | Routing, throttling, OAuth2 / OIDC |
| Customer IAM | _to be sized_ | | MFA per NFR-SEC-020 |
| BFF / Digital API | _to be sized_ | | Client-oriented aggregation |
| AI Financial Advisor | _to be sized_ | | Cloud-only; data minimization per FN-120 |
| Cloud observability | _to be sized_ | | Metrics, logs, traces, dashboards |

## 3. Units of Scale

| Unit | Scale | Trigger to scale out |
|---|---|---|
| Customer (registered) | 100K year 1 → 1M year 3 (NFR-PC-010 / NFR-PC-020) | |
| Concurrent session | _to be sized_ | |
| Transactions / second | _to be sized_ | |
| Read-model records | _to be sized_ | |
| Event-bus throughput | _to be sized_ | |

## 4. HA / DR Targets

Per NFR-AR-010 (99.999% uptime) and NFR-AR-020 (RTO / RPO per component by MS2):

| Component | RTO | RPO | Pattern |
|---|---|---|---|
| CBS | _to be sized_ | _to be sized_ | |
| CBS Transaction Gateway | _to be sized_ | _to be sized_ | |
| Account / Transfer / Report services | _to be sized_ | _to be sized_ | |
| ODS (RDBMS) | _to be sized_ | _to be sized_ | |
| Event bus | _to be sized_ | _to be sized_ | |
| Cloud API Gateway / BFF | _to be sized_ | _to be sized_ | |

## 5. Operational Cost Estimate

Use https://calculator.aws/#/ or https://azure.microsoft.com/en-us/pricing/calculator/ to size the cloud portion. On-prem costs (licensing, support, hardware amortization) should be sized separately.

| Category | Annual cost (USD) | Source / notes |
|---|---|---|
| Cloud compute | _to be calculated_ | |
| Cloud managed services (DB, messaging, IAM) | _to be calculated_ | |
| Cloud network egress | _to be calculated_ | |
| On-prem hardware amortization | _to be calculated_ | |
| On-prem software licensing (incl. ODS) | _to be calculated_ | Oracle / SQL Server / Db2 |
| On-prem operations (power, cooling, floor) | _to be calculated_ | |
| **Total annual** | _sum_ | |

## 6. Notes

- Anchors the deck's cost calculation requirement (deck page 15).
- The 24-hour account-info staleness tolerance (NFR-PC-060) relaxes read-model freshness targets and therefore the storage / replication sizing — call this out explicitly in the final HLD.
- Reuse the sizing approach when the high-level solution is later stress-tested.
