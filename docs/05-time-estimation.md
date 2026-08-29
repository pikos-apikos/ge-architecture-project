# 4. Time Estimation

## 4.1 Planning basis

The production-ready Year-1 MVP is planned through 12 workstreams and 56 work packages. Estimates use productive person-days, three-point PERT expected effort, a conservative uncertainty calibration, and separate schedule logic. Appendix D contains every work package and calculation.

| Measure | Person-days |
|---|---:|
| Best Case | 3,820 |
| Most Likely | 5,780 |
| Worst Case | 8,990 |
| P50 | 5,988 |
| P75 planning effort | 6,217 |
| P90 sensitivity | 6,425 |
| 10% central management reserve | 622 |
| **P75 funding envelope** | **6,839** |

The funding envelope equals 310.9 presentation person-months at 22 person-days per month. Effective delivery capacity is 70%, or 154 productive person-days per FTE-year.

## 4.2 Staffing

The plan uses approximately 46 average and 55 peak FTE, including three of the five available COBOL specialists. Specialist assignment, actual team allocation, organizational overhead, and peak feasibility are Phase-0 gates.

## 4.3 Delivery roadmap

Diagram: `docs/diagrams/18-delivery-roadmap.mmd`.

| Phase | P75 window | Primary exit evidence |
|---|---|---|
| Phase 0 — Mobilization and validation | Weeks 1–4 | Authority, dependencies, vendor and estimate assumptions validated |
| Phase 1 — Foundations and thin slices | Weeks 3–14 | Account and Transfer end-to-end thin slices prove the critical boundaries |
| Phase 2 — Parallel capability build | Weeks 9–32 | Core journeys, data, security and platform features integrated |
| Phase 3 — Integration, migration and hardening | Weeks 24–42 | Performance, resilience, migration and control evidence pass |
| Phase 4 — Pilot, certification and production readiness | Weeks 38–48 | Regulatory, operational and launch gates pass |
| Phase 5 — Cutover, hypercare and handover | Weeks 47–52 | Controlled cutover and initial hypercare complete |

P50 elapsed sensitivity is 46 weeks; P75 is 52 weeks; P90 is 60 weeks. The target cutover is approximately Week 48 with hypercare in Weeks 49–52.

## 4.4 External dependencies

| Scenario | Elapsed duration | Rule |
|---|---:|---|
| On-time | 52 weeks | Approvals and procurement complete inside planned windows |
| Moderate critical delay | 60 weeks | One material external dependency reaches the critical path |
| Major critical delay | 68–76 weeks | Multiple or long regulatory/procurement/vendor delays affect critical path |

External waiting time is not counted as delivery effort. Review of an external AI inference tool is assumed to require two to six elapsed weeks.

## 4.5 Agentic engineering enablement

Agentic engineering adds bootstrap work before it can produce measurable benefit. Work package WS12.1 (80/140/240 person-days) runs in parallel during approximately Weeks 1–10. The architect defines the governed workflow, repositories/instructions, security and data controls, measurement, review standards, and team onboarding. It blocks AI-assisted work but does not block conventional delivery.

The Year-1 AI tooling and inference envelope is EUR 150,000. The pilot cap is EUR 20,000; a financial review occurs at 70%; spend stops at 100% unless a documented human override is approved. Baseline productivity uplift is zero. Sensitivity cases of 5%, 10%, and 20% apply only to a provisional 3,000-person-day eligible pool after evidence from the pilot.

## 4.6 Critical paths and reserve

Platform, Mainframe, and Data are three converging critical paths. Risk-first Account and Transfer thin slices prove their convergence. The 10% reserve is centrally governed and is not distributed as hidden padding. Phase 0 recalibrates three-point inputs and may reforecast the plan when evidence changes.

