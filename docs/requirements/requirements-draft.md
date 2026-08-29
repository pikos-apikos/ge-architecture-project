# 2. Requirements

All requirements are testable HLD contracts. “The system” means the NeoBank Digital Leap solution within the approved MVP boundary.

## 2.1 Functional Requirements

| ID | Requirement |
|---|---|
| FN-010 | The system shall allow an authenticated customer to view account summaries, balances, and bounded transaction history through mobile and web channels. |
| FN-020 | The system shall allow an authenticated customer to initiate a transfer to another account in the same bank. |
| FN-030 | The system shall allow an authenticated customer to initiate a transfer to an IBAN at another bank. |
| FN-040 | The system shall execute every money-moving command as a CBS transaction through the CBS Transaction Gateway. |
| FN-050 | The system shall obtain a real-time fraud decision before submitting a transfer to the CBS. |
| FN-060 | The system shall prevent CBS submission and inform the customer when the real-time Fraud Engine returns DECLINE. |
| FN-070 | The system shall support asynchronous offline fraud analytics using a larger historical dataset and a longer processing window than real-time fraud. |
| FN-075 | The system shall create a controlled investigation case when offline fraud analytics flags an executed transfer; it shall require a human decision and a governed CBS/interbank compensation path before any reversal or recovery action. |
| FN-080 | The system shall provide an immutable, versioned, reconciled monthly income and expense statement by T+1. |
| FN-090 | The system shall expose versioned Open Banking AIS and PIS APIs over REST. |
| FN-100 | The system shall verify TPP identity, PSU, consent scope, permitted accounts, status, expiry, and SCA before Open Banking processing. |
| FN-110 | The system shall provide AI-generated financial guidance from an authorized, purpose-limited customer context. |
| FN-115 | The system shall allow governed bank rules to select eligible product candidates and shall allow the AI Advisor only to rank and explain those candidates. |
| FN-120 | The system shall prevent the AI Advisor from directly accessing the CBS, DB2, ADABAS, or the ODS. |
| FN-130 | The system shall ingest canonical ACCOUNT, BALANCE, and TRANSACTION changes from the legacy environment into digital read models. |
| FN-140 | The system shall provide account-scoped projection quarantine, replay, and CBS-led reconciliation. |
| FN-150 | The system shall provide canonical audit evidence for customer actions, transfer and fraud decisions, consent changes, release actions, and privileged operations. |
| FN-160 | The system shall expose visible freshness and write-capability metadata with account information. |
| FN-170 | The system shall provide a customer-visible transfer status with independent execution and settlement axes. |
| FN-180 | The system shall support a controlled case workflow for unknown CBS outcomes and post-transaction investigations. |

## 2.2 Availability and Recovery

| ID | Requirement |
|---|---|
| NFR-AR-010 | The system shall target 99.999% availability for the aggregate of customer-critical digital capabilities, subject to the approved journey SLO definitions. |
| NFR-AR-020 | The system shall classify data and capabilities into RC0, RC1, RC2, or RC3 recovery classes with explicit RTO and RPO. |
| NFR-AR-030 | The system shall provide high availability within the primary cloud, on-premises, data-platform, and mainframe failure domains. |
| NFR-AR-040 | The system shall provide disaster recovery across paired cloud regions and paired on-premises sites. |
| NFR-AR-050 | The system shall require dual-authority promotion for CBS and other RC0 financial-authority capabilities. |
| NFR-AR-060 | The system shall recover dependencies in approved waves: control plane, authority/integrity, customer-critical services, then derived capabilities. |
| NFR-AR-070 | The system shall degrade AI advice and monthly statements independently from account access and money movement. |
| NFR-AR-080 | The system shall produce immutable evidence for DR promotion, traffic opening, recovery tests, and failback. |

## 2.3 Data Integrity and Consistency

| ID | Requirement |
|---|---|
| NFR-DI-010 | The system shall prevent loss of accepted transfer commands, authoritative outcomes, integration events, and mandatory audit evidence. |
| NFR-DI-020 | The system shall treat the CBS as the sole authoritative system for balances, bookings, and money movement. |
| NFR-DI-030 | The system shall treat the ODS as a non-authoritative read projection. |
| NFR-DI-040 | The system shall make money-moving commands idempotent across channel, Transfer Service, CBS Gateway, and CBS retry boundaries. |
| NFR-DI-050 | The system shall reconcile CBS transactions, transfer state, events, projections, statements, and audit evidence. |
| NFR-DI-060 | The system shall implement read-your-writes after a completed digital transfer using CBS-confirmed overlay data until the ODS catches up. |
| NFR-DI-070 | The system shall deliver integration events at least once and shall require consumer deduplication. |
| NFR-DI-080 | The system shall preserve account-scoped event ordering using partition key, source position, and entity or aggregate version. |
| NFR-DI-090 | The system shall quarantine only the affected account when a CDC mapping or projection failure occurs. |
| NFR-DI-100 | The system shall represent an unknown CBS outcome as PROCESSING and shall not fabricate COMMITTED or REJECTED. |

## 2.4 Performance and Capacity

| ID | Requirement |
|---|---|
| NFR-PC-010 | The system shall support 100,000 registered digital users in Year 1. |
| NFR-PC-015 | The system shall support at least 5,000 concurrent sessions in Year 1 and 50,000 concurrent sessions in Year 3. |
| NFR-PC-020 | The system shall scale to 1,000,000 registered digital users in Year 3 within one geographic region. |
| NFR-PC-030 | The system shall target a p95 server-side response time of at most 200 ms for account-summary reads under the approved base workload. |
| NFR-PC-040 | The system shall protect the CBS from direct digital read traffic and from unbounded command bursts. |
| NFR-PC-050 | The system shall monitor edge throughput, internal amplification, durable event throughput, consumer lag, storage growth, and CBS-bound demand. |
| NFR-PC-060 | The system shall allow up to 24 hours of staleness only for incoming transactions from other banks or payments not made through the app and shall display freshness metadata. |
| NFR-PC-070 | The system shall meet the approved ODS freshness SLO for ordinary CDC projection and shall expose breach status. |
| NFR-PC-080 | The system shall provide elastic stateless capacity with at least 30% steady-state headroom and a controlled burst policy. |
| NFR-PC-090 | The system shall enforce bounded request, event, cursor, batch, and payload sizes. |

## 2.5 Security

| ID | Requirement |
|---|---|
| NFR-SEC-010 | The system shall authenticate customers, TPPs, workforce users, workloads, and administrators with identity controls appropriate to their trust zone. |
| NFR-SEC-020 | The system shall apply MFA or risk-based step-up to protected customer actions and phishing-resistant MFA to privileged access. |
| NFR-SEC-030 | The system shall route all public traffic through DDoS protection and WAF before the API Gateway. |
| NFR-SEC-040 | The system shall enforce consent, scope, account, audience, and purpose at both the public boundary and the owning service. |
| NFR-SEC-050 | The system shall provide the AI Advisor only minimized, authorized, regionally compliant data through the Advisor Context Service. |
| NFR-SEC-060 | The system shall encrypt sensitive data in transit and at rest using federated custody with bank-controlled roots. |
| NFR-SEC-070 | The system shall apply field-level tokenization or encryption to highly sensitive identifiers and secrets where classification requires it. |
| NFR-SEC-080 | The system shall provide JIT, scoped, expiring privileged access through PAM with recorded sessions and dual control for critical actions. |
| NFR-SEC-090 | The system shall fail closed when required identity, consent, authorization, key, or evidence controls cannot be established. |
| NFR-SEC-100 | The system shall retain only sanitized, classified, purpose-limited telemetry and evidence. |

## 2.6 Backward Compatibility

| ID | Requirement |
|---|---|
| NFR-BC-010 | The system shall allow only additive, backward-compatible API and event changes inside a major version. |
| NFR-BC-020 | The system shall run old and new supported major versions concurrently during a governed deprecation window. |
| NFR-BC-030 | The system shall version and coordinate CBS Gateway contract changes with the mainframe team. |
| NFR-BC-040 | The system shall require consumers to tolerate unknown optional fields and shall prohibit producers from repurposing existing fields. |
| NFR-BC-050 | The system shall use consumer-first expand–migrate–contract sequencing for APIs, events, databases, and cache state. |

## 2.7 GDPR and Compliance

| ID | Requirement |
|---|---|
| NFR-GDPR-010 | The system shall process customer data only in the approved geographic region. |
| NFR-GDPR-020 | The system shall support deletion or irreversible anonymization of personal data where legally permitted. |
| NFR-GDPR-030 | The system shall retain mandatory financial, security, consent, and audit records for governed retention periods. |
| NFR-GDPR-040 | The system shall distinguish deletable personal data from legally retained financial and evidence records. |
| NFR-GDPR-050 | The system shall record purpose, provenance, classification, and retention class for advisor datasets and regulated evidence. |

## 2.8 Observability and Debugging

| ID | Requirement |
|---|---|
| NFR-OBS-010 | The system shall provide metrics, logs, traces, dashboards, and alerts for eight approved end-to-end journeys. |
| NFR-OBS-020 | The system shall provide business outcome metrics for account access, transfers, fraud, Open Banking, statements, AI advice, and digital adoption. |
| NFR-OBS-030 | The system shall provide visibility across public edge, cloud, on-premises, data platform, mainframe, and evidence boundaries. |
| NFR-OBS-040 | The system shall propagate correlation and trace context through API, BFF, service, event, CBS Gateway, and evidence paths. |
| NFR-OBS-050 | The system shall measure journey success at the authoritative durable outcome, not only at the edge. |
| NFR-OBS-060 | The system shall use risk-aware sampling without sampling away failures, security signals, money movement, or mandatory evidence. |
| NFR-OBS-070 | The system shall use multi-window error-budget burn alerts with explicit journey ownership. |
| NFR-OBS-080 | The system shall produce a reproducible incident evidence bundle and executable runbook evidence. |

## 2.9 Deployment and Upgradability

| ID | Requirement |
|---|---|
| NFR-DEP-010 | The system shall separate regulated on-premises workloads from cloud-allowed workloads using explicit trust and network zones. |
| NFR-DEP-020 | The system shall support rolling, canary, or blue/green deployment for eligible stateless services. |
| NFR-DEP-030 | The system shall minimize CBS change and decouple digital and mainframe release trains through versioned contracts. |
| NFR-DEP-040 | The system shall promote the same signed immutable artifact through development, integration, pre-production, and production. |
| NFR-DEP-050 | The system shall require provenance, SBOM, security scan, segregation-of-duties, and release-receipt evidence before production admission. |
| NFR-DEP-060 | The system shall align rollback or roll-forward behavior to statefulness and recovery class. |
| NFR-DEP-070 | The system shall schedule non-production auto-shutdown outside working hours unless a justified, time-bound override is approved. |
| NFR-DEP-080 | The system shall use synthetic data by default outside production and shall govern any production-like data exception. |
| NFR-DEP-090 | The system shall restrict production deployment to the pipeline and shall treat direct access as a JIT audited exception. |
| NFR-DEP-100 | The system shall govern feature flags with ownership, scope, evidence, expiry, and emergency kill-switch controls. |

## 2.10 Assumptions and Constraints

The authoritative assumption register is `docs/03-assumptions.md`. Material assumptions MUST be validated in Phase 0 and MUST NOT silently override a requirement. The architecture is constrained by the one-year MVP target, 60-developer organization profile, five COBOL specialists, a conservative regulated-bank control environment, regional processing, and the requirement to preserve the CBS as the financial authority.

