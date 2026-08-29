# Appendix D — Delivery Estimation Evidence

## D.1 Estimation contract

The estimate covers the production-ready Year-1 MVP for approximately 100,000 registered users. It uses 56 work packages across 12 workstreams. Values are productive person-days and exclude external waiting time.

| Rule | Approved value |
|---|---|
| Expected task effort | `(Best + 4 × Most Likely + Worst) / 6` |
| Task standard deviation | `(Worst - Best) / 2.1` |
| Confidence factors | P75 `0.67`; P90 `1.28` |
| Presentation conversion | 22 person-days per person-month |
| Effective annual capacity | 70%, or 154 productive days per FTE-year |
| Planning baseline | P75 effort plus a separately governed 10% central reserve |
| Evidence rule | Phase 0 recalibrates material inputs and records variance |

The calculation aggregates independent task variances before applying the confidence factors. Correlated programme risks, external approvals and supplier lead times are represented in schedule scenarios and the risk register rather than hidden inside task effort.

## D.2 Detailed three-point inputs

| ID | Work package | Best | Most Likely | Worst |
|---|---|---:|---:|---:|
| WS1.1 | Mobilization and governance | 50 | 70 | 100 |
| WS1.2 | Product scope and journeys | 60 | 90 | 130 |
| WS1.3 | Detailed architecture and traceability | 80 | 120 | 170 |
| WS1.4 | Regulatory and privacy approvals | 45 | 70 | 110 |
| WS1.5 | Enterprise and procurement coordination | 90 | 130 | 190 |
| WS2.1 | Mobile application | 110 | 160 | 230 |
| WS2.2 | Web application | 90 | 130 | 190 |
| WS2.3 | WAF and API Gateway | 45 | 70 | 110 |
| WS2.4 | BFF and channel aggregation | 70 | 105 | 155 |
| WS2.5 | Channel hardening and release | 55 | 85 | 130 |
| WS3.1 | Customer IAM and MFA | 90 | 130 | 190 |
| WS3.2 | Customer-account authorization | 60 | 90 | 140 |
| WS3.3 | Consent and SCA | 70 | 105 | 160 |
| WS3.4 | AIS/PIS and TPP sandbox | 85 | 125 | 190 |
| WS3.5 | Open Banking certification | 35 | 50 | 85 |
| WS4.1 | Account API | 65 | 90 | 130 |
| WS4.2 | Transfer orchestration | 100 | 150 | 230 |
| WS4.3 | Transfer state and idempotency | 70 | 100 | 160 |
| WS4.4 | Real-time fraud | 70 | 100 | 150 |
| WS4.5 | Offline fraud and compensation | 55 | 80 | 130 |
| WS4.6 | Service hardening and evidence integration | 85 | 130 | 200 |
| WS5.1 | CBS discovery and mapping | 45 | 70 | 120 |
| WS5.2 | Canonical Gateway | 70 | 110 | 180 |
| WS5.3 | Unknown-result controls | 55 | 85 | 145 |
| WS5.4 | Contract and capacity tests | 55 | 85 | 140 |
| WS5.5 | Version and DR coordination | 45 | 70 | 120 |
| WS6.1 | ODS platform | 90 | 130 | 200 |
| WS6.2 | DB2 CDC | 70 | 105 | 170 |
| WS6.3 | ADABAS ingestion | 55 | 85 | 145 |
| WS6.4 | Event schemas and governance | 60 | 90 | 135 |
| WS6.5 | Projections | 80 | 120 | 180 |
| WS6.6 | Cache | 35 | 55 | 90 |
| WS6.7 | Replay and reconciliation | 75 | 115 | 190 |
| WS7.1 | Monthly statements | 50 | 75 | 115 |
| WS7.2 | Income and expense reporting | 45 | 65 | 100 |
| WS7.3 | Advisor Context and eligibility | 55 | 85 | 130 |
| WS7.4 | AI guardrails and evaluation | 45 | 75 | 125 |
| WS8.1 | Cloud primary and DR | 80 | 120 | 180 |
| WS8.2 | On-premises primary and DR | 85 | 130 | 210 |
| WS8.3 | Hybrid network | 65 | 100 | 165 |
| WS8.4 | Four environments | 60 | 90 | 140 |
| WS8.5 | Runtime platforms | 75 | 110 | 170 |
| WS8.6 | Capacity and admission controls | 45 | 70 | 110 |
| WS9.1 | PKI and encryption | 60 | 90 | 145 |
| WS9.2 | Security and PAM controls | 75 | 115 | 180 |
| WS9.3 | Audit and WORM evidence | 65 | 100 | 155 |
| WS9.4 | Observability and SLOs | 80 | 120 | 180 |
| WS9.5 | Secure SDLC | 60 | 95 | 150 |
| WS10.1 | Automated and contract tests | 90 | 140 | 210 |
| WS10.2 | Eight end-to-end journeys | 100 | 150 | 230 |
| WS10.3 | Performance and soak tests | 80 | 120 | 190 |
| WS10.4 | HA, DR, backup and failback exercises | 120 | 190 | 300 |
| WS11.1 | Pilot, migration and cutover | 90 | 140 | 220 |
| WS11.2 | Training, runbooks and handover | 45 | 70 | 110 |
| WS11.3 | Hypercare and stabilization | 55 | 90 | 140 |
| WS12.1 | Bank-grade Agentic Engineering Enablement | 80 | 140 | 240 |

## D.3 Aggregate effort and staffing

| Measure | Person-days | Person-months |
|---|---:|---:|
| Best Case | 3,820 | 173.6 |
| Most Likely | 5,780 | 262.7 |
| Worst Case | 8,990 | 408.6 |
| P50 | 5,988 | 272.2 |
| **P75 planning effort** | **6,217** | **282.6** |
| P90 sensitivity | 6,425 | 292.0 |
| 10% central management reserve | 622 | 28.3 |
| **P75 funding envelope** | **6,839** | **310.9** |

The delivery envelope assumes approximately 46 average FTE, 55 peak FTE and three of the five available COBOL specialists. Effective annual capacity is `46 × 154 = 7,084` person-days. P75 effort uses approximately 87.8% of that capacity; the funding envelope including reserve uses approximately 96.5%. The reserve is not preallocated and requires governed drawdown.

## D.4 Overlapping delivery plan

| Phase | P75 window | Exit evidence |
|---|---|---|
| 0 — Mobilization and validation | Weeks 1–4 | Baselines, allocations, vendor and dependency evidence |
| 1 — Foundations and thin slices | Weeks 3–14 | Account and Transfer thin slices across real boundaries |
| 2 — Parallel capability build | Weeks 9–32 | Feature-complete services and platform capabilities |
| 3 — Integration, migration and hardening | Weeks 24–42 | Integrated journeys, performance, security and reconciliation |
| 4 — Pilot, certification and readiness | Weeks 38–48 | Pilot acceptance, certification and production-readiness gates |
| 5 — Cutover, hypercare and handover | Weeks 47–52 | Controlled cutover, operational ownership and stabilization |

The three converging critical paths are Platform, Mainframe and Data. P50 elapsed sensitivity is 46 weeks, the P75 planning duration is 52 weeks and P90 sensitivity is 60 weeks. The plan targets cutover around Week 48 and initial hypercare in Weeks 49–52.

| External-dependency scenario | Elapsed duration | Treatment |
|---|---:|---|
| On-time | 52 weeks | Approved P75 plan |
| Moderate critical delay | 60 weeks | Re-sequence unaffected work; preserve blocked dependency evidence |
| Major critical delay | 68–76 weeks | Rebaseline governance and release commitments |

External waiting time is not delivery effort. Regulatory review, procurement, certification, network circuits, hardware, ODS licensing and organizational approvals remain explicit dependencies. External AI-tool legal/security approval is assumed to require 2–6 elapsed weeks and must be validated in Phase 0.

## D.5 Agentic engineering enablement

WS12.1 recognizes that AI coding agents do not create free productivity. The architect and platform teams must first establish a measurable, bank-grade workflow.

| Enablement component | Best | Most Likely | Worst |
|---|---:|---:|---:|
| Workflow architecture and agent contracts | 25 | 40 | 60 |
| Secure integration and access boundaries | 20 | 35 | 60 |
| Repository instructions, templates and evaluations | 20 | 35 | 60 |
| Telemetry, cost controls and auditability | 10 | 20 | 35 |
| Pilot onboarding and operating guidance | 5 | 10 | 25 |
| **Total** | **80** | **140** | **240** |

WS12.1 runs as a parallel enabler during approximately Weeks 1–10. It gates AI-assisted work but does not block conventional development. The Year-1 tooling and inference envelope is EUR 150,000, with a EUR 20,000 pilot cap, review at 70% spend and stop or documented human override at 100%.

The baseline assumes zero AI productivity uplift. After the pilot, 5%, 10% and 20% sensitivity cases may be applied only to a confirmed pool of remaining eligible work, provisionally capped at 3,000 person-days. Legal approval, model access, license and inference charges remain separate from person-day effort.

## D.6 Mandatory recalibration gates

Phase 0 must validate team allocation, specialist availability, the 70% capacity factor, all material three-point inputs, vendor and network lead times, external approval durations, the 2–6-week AI-tool review assumption, AI commercial and data-processing terms, the eligible-work pool, critical-path concurrency and reserve governance. A material variance triggers a controlled reforecast rather than silent substitution.
