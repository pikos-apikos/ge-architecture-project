# Appendix C — Sizing, Capacity and TCO Evidence

## C.1 Workload derivation

All values are planning assumptions until replaced by dated production telemetry, data profiling or representative load tests. Registered-user targets are known programme inputs; the remaining multipliers are explicit assumptions.

| Measure | Year 1 | Year 3 | Derivation |
|---|---:|---:|---|
| Registered digital users | 100,000 | 1,000,000 | Programme target |
| Peak concurrent sessions | 5,000 | 50,000 | 5% of registered users |
| Steady peak edge rate | 250 RPS | 2,500 RPS | One request per active session per 20 seconds |
| Five-minute burst | 750 RPS | 7,500 RPS | 3× steady peak |
| Internal service traffic | 500 RPS | 5,000 RPS | 2× edge requests, excluding the original request and database calls |
| CBS-bound steady rate | 12.5 TPS | 125 TPS | Transfer and PIS commands; 5% of edge traffic |
| CBS-bound burst | 37.5 TPS | 375 TPS | 3× steady CBS-bound rate |
| Durable events, steady | 200/s | 2,000/s | Command, CDC and lifecycle event model |
| Durable events, burst | 600/s | 6,000/s | 3× steady event rate |

The approved peak edge mix is 70% account/balance/history reads, 15% authentication/session, 4% transfer commands, 5% Open Banking AIS, 1% consent, 1% PIS, 2% reports, 1% AI Advisor and 1% other controlled APIs. Each CBS-bound command has a ten-event allowance; CDC is allowed at twice the digital CBS-bound rate; other lifecycle events equal 10% of edge traffic.

Capacity rules:

- steady peak must consume no more than 60% of usable active capacity;
- stateless services start with at least 1.67× steady demand and scale to the 3× burst within two minutes;
- critical tiers retain N+1 capacity in the active failure domain;
- stateful IOPS, connections and queues must sustain the five-minute burst without SLO breach;
- CBS admission control uses the verified throughput ceiling and never converts rejection into accepted success;
- replay uses separate quotas and cannot consume live RC0/RC1 capacity.

## C.2 Daily volume and retention assumptions

The approved peak-to-average ratio is 10:1. Capacity uses peak and burst; storage and usage cost use the 24-hour average.

| Daily workload | Year 1 | Year 3 |
|---|---:|---:|
| Edge requests | 2.16 million | 21.6 million |
| CBS-bound commands | 108,000 | 1.08 million |
| Durable events | 1.728 million | 17.28 million |

Durable events average 2 KB, have a p99 planning size of 16 KB and a 256 KB hard limit. Larger evidence or documents go to immutable object storage and the event carries the reference, content type, size and checksum. Raw event growth is approximately 3.46 GB/day in Year 1 and 34.56 GB/day in Year 3 before replication, indexes, compression and filesystem overhead.

| Data family | Hot/online rule | Protected retention rule |
|---|---|---|
| Event Bus | 7 days live | 90 days replay archive; longer evidence retention by classification |
| ODS transactions | 24 months queryable | Governed archive thereafter |
| Monthly statements | Reconciled immutable snapshots | Legal retention policy; WORM where required |
| Audit evidence | Search tier by operational need | Canonical receipt plus immutable evidence plane |
| Backups | Daily/weekly/monthly rotations | Product and legal policy, tested restore and expiry |
| Observability | Hot tier sized by journey/SLO need | Tiered metrics, logs and traces; security evidence exported separately |

ODS write arrival includes transaction changes, balance updates, account state and projection metadata. Index, replica and temporal-history factors must be measured. Reconciliation and replay must preserve account ordering, idempotency and quarantine boundaries.

## C.3 Workload-specific allowances

Statement sizing assumes an average of 1.5 statement-bearing accounts per customer. Monthly statements are generated from a reconciled snapshot at T+1 and must not compete with RC0/RC1 workload. Report and AI batch windows use quotas and may be paused under platform stress.

The approved AI workload is 15.77B input and 3.15B output tokens in Year 1, increasing to 157.68B input and 31.54B output tokens in Year 3. At least 90% uses the cost-efficient route and no more than 10% premium escalation. Product eligibility remains governed by bank rules; the model ranks and explains only eligible candidates.

Telemetry capacity is based on the eight approved end-to-end journeys, controlled span sampling, SLO metrics, security evidence export and bounded high-cardinality labels. Debug sampling can be raised temporarily through a time-bounded change. Observability retention must not become an uncontrolled copy of customer data.

## C.4 Physical capacity baseline

| Tier | Year 1 nodes | Year 3 nodes | Architectural role |
|---|---:|---:|---|
| Application compute | 8 | 14 | Stateless Digital Core, gateways and workers with N+2 active capacity |
| ODS | 6 | 12 | Local HA, governed replication and NVMe/SAN-backed data paths |
| Event Bus | 6 | 12 | Odd quorum, partitioned durable events and isolated replay capacity |
| Object archive/evidence/backup | 8 | 16 | Erasure-coded immutable archive, evidence and backup |
| Hot observability | 6 | 12 | Metrics, logs, traces and SIEM export separated from workloads |
| **Total** | **34** | **66** | Primary and recovery-site topology by recovery class |

Application hosts use redundant x86 compute, memory, NIC and storage paths. ODS, Event Bus and observability use workload-appropriate NVMe tiers. Network paths must provide redundant cloud-to-datacentre connectivity and independent management access. Warm DR reserve follows RC0–RC3 recovery classes; the model does not assume active-active writes across sites.

Installed hardware capital is estimated at USD 1.0M–1.6M for Year 1 and USD 2.4M–4.0M for Year 3. Procurement must replace these architecture bands with comparable vendor quotations.

## C.5 Annual operating-cost envelope

Pricing date is 2026-08-27. Values are USD millions per year, use AWS Europe public/on-demand anchors, five-year straight-line hardware amortization and a 20% uncertainty contingency. Taxes, negotiated discounts, implementation labour and project delivery cost are excluded.

| Cost category | Y1 low | Y1 base | Y1 high | Y3 low | Y3 base | Y3 high |
|---|---:|---:|---:|---:|---:|---:|
| Cloud, connectivity, support and AI | 0.30 | 0.45 | 0.70 | 0.75 | 1.15 | 1.90 |
| Hardware amortization and support | 0.35 | 0.50 | 0.65 | 0.83 | 1.10 | 1.49 |
| ODS RDBMS license and support | 0.15 | 0.35 | 1.00 | 0.30 | 0.70 | 2.00 |
| Platform software and support | 0.25 | 0.45 | 0.75 | 0.50 | 0.90 | 1.50 |
| Power, cooling, floor and network | 0.18 | 0.27 | 0.43 | 0.30 | 0.48 | 0.70 |
| Subtotal | 1.23 | 2.02 | 3.53 | 2.68 | 4.33 | 7.59 |
| 20% contingency | 0.25 | 0.40 | 0.71 | 0.54 | 0.87 | 1.52 |
| **Infrastructure TCO** | **1.48** | **2.42** | **4.24** | **3.22** | **5.20** | **9.11** |

Operational staffing is separate: Y1 USD 0.7M/1.2M/1.8M and Y3 USD 1.0M/1.8M/3.0M. Infrastructure plus staffing is therefore Y1 USD 2.2M/3.6M/6.0M and Y3 USD 4.2M/7.0M/12.1M.

The AI allowance inside the cloud row is approximately USD 0.003M/0.006M/0.025M in Year 1 and USD 0.030M/0.060M/0.250M in Year 3. It must remain visible by model route, token budget and business capability. ODS licensing is the largest on-premises uncertainty: the low case assumes an existing agreement or lower-cost approved topology; the high case allows for Oracle Enterprise options, replication, diagnostics and support. Facilities assume PUE 1.6 and electricity at USD 0.15–0.30/kWh.

## C.6 Validation and stop conditions

Phase 0 must validate provider and region, data residency, ODS product and license position, CBS throughput, CDC and row volumes, event and telemetry sizes, retention, DR reserve, hardware quotes, facilities, connectivity and support plans. FinOps must rerun the cloud model with contracted rates. Load and soak tests must validate capacity units, scaling, queue limits, partition distribution and replay isolation. A result outside the approved low-to-high band triggers a sizing and cost review. This architecture estimate is not purchasing authority.
