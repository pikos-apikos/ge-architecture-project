# 3.6 Sizing

## 3.6.1 Executive sizing summary

The sizing model is derived from registered users, peak concurrency, traffic mix, synchronous amplification, durable event volume, ODS record arrival, retention, telemetry, monthly statements, and AI context workload. Appendix C contains the complete formulas and evidence.

| Scale measure | Year 1 | Year 3 |
|---|---:|---:|
| Registered digital users | 100,000 | 1,000,000 |
| Peak concurrent sessions | 5,000 | 50,000 |
| Peak session assumption | 5% | 5% |
| Application compute physical nodes | 8 | 14 |
| ODS physical nodes | 6 | 12 |
| Event Bus physical nodes | 6 | 12 |
| Object archive/evidence/backup nodes | 8 | 16 |
| Hot observability physical nodes | 6 | 12 |
| Total approved physical nodes | 34 | 66 |

Stateless services use a standard compute unit and maintain at least 30% headroom under the approved base peak. The platform applies quotas, backpressure, bounded queues, and CBS protection rather than scaling an unsafe dependency without limit.

## 3.6.2 On-premises hardware and software

| Layer | Hardware baseline | Software baseline | HA/DR rule |
|---|---|---|---|
| Application platform | x86 hosts with redundant CPU, memory, NIC, boot and data paths | Enterprise Linux plus Kubernetes/OpenShift-class platform, service mesh and secrets integration | N+2 primary capacity; warm recovery-site reserve by recovery class |
| ODS | Enterprise x86 database nodes with NVMe tier and SAN/object backup integration | Oracle, SQL Server, or Db2 Enterprise; product selected in Phase 0 | Synchronous/local HA and governed cross-site replication |
| Event Bus | Dedicated broker nodes with NVMe and 25-GbE-class networking | Kafka-compatible enterprise distribution | Odd-sized quorum in each active cluster; replicated recovery capacity |
| Archive/evidence/backup | Capacity-optimized object nodes with erasure coding and immutable storage capability | S3-compatible object platform, backup catalog and WORM/retention controls | Site copy plus protected backup; evidence integrity verification |
| Observability | Compute/storage nodes sized for hot search and metric cardinality | Metrics, logs, traces, SIEM integration and evidence exporters | Failure-domain separation from monitored workloads |
| Mainframe integration | Gateway HA pair on separate hosts/zones | Canonical gateway, protocol adapter, HSM/KMS client and reconciliation worker | Only digital path to CBS; no channel or ODS bypass |

## 3.6.3 Performance and Capacity View

Diagram: `docs/diagrams/17-performance-capacity.mmd`.

The diagram traces the calculation from users to sessions, edge requests, internal amplification, compute, events, storage, the CBS protection envelope, and TCO. Every input is a planning assumption until Phase-0 measurement or load-test evidence confirms it.

## 3.6.4 Annual operational cost

All values are USD millions per year, priced on 2026-08-27 using AWS Europe public/on-demand anchors and enterprise hardware/software bands. Taxes, discounts, implementation labor, and project delivery cost are excluded. Hardware is amortized over five years. A 20% contingency is included.

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

Operational staffing is separate: Y1 USD 0.7M/1.2M/1.8M and Y3 USD 1.0M/1.8M/3.0M for low/base/high. Infrastructure plus staffing is therefore Y1 USD 2.2M/3.6M/6.0M and Y3 USD 4.2M/7.0M/12.1M.

## 3.6.5 AI inference allowance

The approved workload is 15.77B input and 3.15B output tokens in Year 1, growing to 157.68B input and 31.54B output tokens in Year 3. At least 90% uses a cost-efficient route; no more than 10% uses premium escalation. The allowance is approximately USD 0.003M/0.006M/0.025M in Y1 and USD 0.030M/0.060M/0.250M in Y3 for low/base/high.

## 3.6.6 Validation gates

Phase 0 MUST validate cloud provider and regions, ODS product and licenses, CBS throughput, CDC volume, row/event/telemetry sizes, load-test capacity, retention, hardware quotes, facilities inputs, connectivity, and support plans. A result outside the approved low-to-high band triggers re-sizing. This estimate is not purchasing authority.

