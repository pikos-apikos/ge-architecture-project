# 6. Risks and Mitigations

| ID | Risk | Likelihood / impact / rating | Mitigation and evidence | Owner / gate |
|---|---|---|---|---|
| R-001 | CBS integration becomes a delivery or runtime bottleneck. | Medium / High / High | Gateway isolation, idempotency, rate protection, contract tests, thin slice and capacity evidence. | Mainframe and Payments |
| R-002 | Stale or conflicting account data reduces customer trust. | Medium / High / High | Freshness metadata, recent-write overlay, account quarantine and CBS-led reconciliation. | Data Platform and Product |
| R-003 | Fraud policy causes false positives or misses. | Medium / High / High | Governed policy/model versions, reason codes, tuning evidence and controlled investigation. | Fraud and Risk |
| R-004 | Cloud/on-premises latency or link failure degrades journeys. | Medium / High / High | BFF aggregation, resilient links, capacity headroom, circuit isolation and degradation tests. | Infrastructure and SRE |
| R-005 | AI advice or product promotion violates compliance obligations. | Medium / High / High | Bank-rule eligibility, minimized context, disclosures, revalidation and audit evidence. | Product, Legal and Compliance |
| R-006 | ODS is treated as a second system of record. | Low / Critical / High | CBS authority rule, service authorization, reconciliation and architecture conformance review. | Architecture and DBA |
| R-007 | Region, regulator or bank policy rejects external AI inference. | Medium / Medium / Medium | Governed phased rollout, 2–6-week review assumption and graceful no-AI operation. | Legal, Security and AI Platform |
| R-008 | Enterprise ODS choice creates license or vendor lock-in. | Medium / Medium / Medium | Phase-0 option evidence, portable canonical model and formal license position. | Enterprise Architecture and DBA |
| R-009 | Five-person COBOL pool constrains critical-path work. | High / High / High | Minimize CBS change, reserve three specialists, thin-slice early and escalate capacity conflicts. | Mainframe leadership |
| R-010 | Open Banking regulation or standards change. | Medium / High / High | Versioned boundary, consent isolation, regulatory evidence and deprecation policy. | Open Banking and Compliance |
| R-011 | Infrastructure or software TCO exceeds plan. | Medium / High / High | Low/base/high bands, 20% contingency, FinOps recalculation and procurement quotes. | Finance and Architecture |
| R-012 | Delivery estimate misses approval, dependency or bootstrap work. | Medium / High / High | P75 plan, reserve, external scenarios, WS12.1 and Phase-0 re-estimation. | Program leadership |
| R-013 | Credential, endpoint, session or authorization compromise enables an authenticated malicious action. | Medium / High / High | MFA, token validation, distributed authorization, fraud controls and time-bound acceptance. | IAM, Channels and CISO |
| R-014 | Edge/WAF/DDoS or cloud control-plane failure exposes or denies public ingress. | Low / High / Medium | Provider controls, configuration evidence, capacity and DR tests. | Infrastructure, SRE and CISO |
| R-015 | Consent revocation delay or TPP credential compromise permits excess access. | Low / High / Medium | Short-lived verified context, exact scope/account checks, revocation SLO and certificate lifecycle. | Consent, Open Banking and CISO |
| R-016 | Legacy CBS protocol, credential, encryption or unknown-outcome limits weaken modern controls. | Medium / High / High | Gateway controls, canonical contracts, PROCESSING state, reconciliation and recovery evidence. | Mainframe, Payments and CISO |
| R-017 | CDC mapping, poison event, replay or data leakage corrupts a projection or exposes legacy data. | Low / High / Medium | Canonical allowlist, versioning, account quarantine, replay tests, DLP and schema evidence. | Data Platform and Privacy |
| R-018 | Prompt injection, provider behavior or context leakage produces unsafe advice or discloses data. | Medium / Medium / Medium | Advisor Context-only route, egress controls, output validation, model red-team and audit. | AI Platform, Product and Privacy |
| R-019 | Privileged insider, PAM compromise or key-recovery failure defeats controls or blocks recovery. | Low / Critical / High | JIT privilege, dual control, HSM/KMS custody, session evidence and break-glass tests. | Security Operations and CISO |

Unaccepted High risks R-001, R-002, R-003, R-004, R-005, R-006, R-009, R-010, R-011, R-012, R-013, R-016 and R-019 block production readiness until mitigated or formally accepted by the named accountable authorities with review and expiry.

