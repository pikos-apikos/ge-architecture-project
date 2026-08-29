# 3.2 Design Rules and Cross-Cutting Architecture

This section is normative for detailed design and implementation. The CBS remains the sole financial authority. ODS, caches, reports and AI datasets are governed projections and must never authorize money movement.

## 3.2.1 Consistency and freshness

| Data path | Authority and contract | Failure, reconciliation and customer behavior |
|---|---|---|
| Transfer → Fraud → CBS | Success only after CBS commit and stable transaction ID. Fraud approval precedes CBS submission. | Fraud timeout uses one initial call plus three retries, then `NOT_SUBMITTED`. Unknown CBS outcome remains `PROCESSING` until reconciled. External settlement is a separate status axis. |
| Completed transfer → account view | CBS response supplies transaction ID and confirmed available/current balances; ODS later converges. | Apply a read-your-writes overlay until the matching projection version appears. The Digital Core never calculates a replacement balance. |
| DB2/ADABAS → CDC/Event Bus | Source log is authoritative. Delivery is at least once and monotonically ordered per account. | Stable event ID, source position, transaction reference and aggregate version enable replay and deduplication. Global ordering is unnecessary. |
| CDC → ODS projection | ODS is non-authoritative. Freshness SLO is p95 ≤ 5 s and p99 ≤ 30 s. | Bounded retry, then account-scoped quarantine. Other accounts continue. Beyond 60 s, show `Data temporarily delayed` and `Last updated`. |
| Quarantined account | CBS remains authoritative; last complete ODS view remains readable. | Hold later events for that account, block balance-dependent writes before Fraud/CBS, and resume only after ordered replay and successful comparison. |
| Account-data cache | ODS projection version governs version-aware cache-aside; TTL ≤ 30 s. | ODS events invalidate/update. Cache failure falls back to ODS, never CBS. Quarantine and freshness state take precedence. |
| External/non-app activity | CBS is authoritative. | A source-specific delay up to 24 h is permitted and must be timestamped. This allowance does not apply to NeoBank transfers or ordinary CDC lag. |
| Mobile/Web and Open Banking AIS | ODS, cache and CBS overlay under the same freshness contract. | Responses expose timestamps/freshness; quarantined accounts are readable but write-blocked. |
| Open Banking PIS | Common Transfer path with the same consent, fraud, idempotency, CBS and outcome rules. | Unknown results remain `PROCESSING`; quarantined accounts are rejected before submission. |
| Monthly statement | Reconciled month-end snapshot by T+1 day; issued version is immutable. | A correction creates a new version. Every artifact exposes period, `As of` and version. |
| AI Advisor | Pseudonymized derived context refreshed at least daily; maximum age 24 h. | It cannot authorize transactions. Current-balance questions use the Account Information path and time-sensitive answers show `As of`. |
| CBS–ODS mismatch | CBS always wins. | Preserve evidence, quarantine, rebuild from the last trusted checkpoint and compare again. Never patch rows to force convergence. |

Cross-path invariants are no loss of transaction commands, integration events or audit evidence; no duplicate business effect; no interpretation of a timeout as proof; stable idempotency identifiers; explicit stale-state disclosure; and immutable/versioned evidence for every persisted transfer transition.

## 3.2.2 Security model

Every zone crossing is default-deny, explicitly authorized, encrypted and auditable. Network location is not identity, authentication is not authorization, and request-body identity, channel, role or consent data is untrusted.

| ID | Trust zone | Principal assets and posture |
|---|---|---|
| Z1 | External Actors | Mobile/Web clients and TPPs; untrusted input and no direct internal route |
| Z2 | Cloud Public Edge | WAF/DDoS filtering, rate controls and attack absorption |
| Z3 | Cloud Private Applications | API Gateway, Customer IAM, BFF and AI Advisor |
| Z4 | Hybrid Interconnect / DMZ | Private-link/VPN termination, NGFW and reverse proxy |
| Z5 | Digital Core Services | Account, Transfer, Consent, Report, Fraud, Advisor Context, CBS Gateway and adapters |
| Z6 | Regulated Data & Integration | ODS, Event Bus and Audit Store with least-privilege data-plane access |
| Z7 | Mainframe Zone | CBS, DB2 and ADABAS; isolated authority identities |
| Z8 | Security & Management Plane | PAM, KMS/HSM, monitoring and admin endpoints under separate identities |

### Zone-crossing rules

- Z1 reaches internal services only through Z2 and the managed ingress chain.
- Z2→Z3 re-encrypts traffic and validates customer OIDC/MFA or TPP OAuth+mTLS.
- Z3→Z4 and Z4→Z5 require workload identity, mTLS, short-lived audience/scope tokens, allowlists and correlation evidence.
- AI can call only the Advisor Context API. It has no direct route to ODS, Event Bus or CBS.
- Z5→Z6 uses dedicated operation-scoped identities for read, publish, consume or append-only audit.
- Only the CBS Gateway can execute canonical money-moving commands in Z7; raw legacy codes remain inside the gateway/operations boundary.
- DB2/ADABAS CDC reaches Z6 only through isolated adapters and canonical mapping with source-position evidence.
- Z8 administration requires a managed route, dedicated privileged identity, phishing-resistant MFA, JIT PAM, session recording and automatic expiry.
- Cloud workloads cannot bypass Z4; the DMZ cannot access Z6/Z7 directly; ordinary customer, corporate and application networks provide no administrative route.

### Identity, authorization and data protection

| Identity | Authentication | Authorization boundary |
|---|---|---|
| Customer | OAuth 2.0/OIDC and MFA when required | Gateway coarse policy plus owning-service resource/business checks |
| TPP | OAuth client identity, mTLS and verified short-lived ConsentContext | Gateway scope plus Consent/Account/Transfer ownership and operation checks |
| Workload | Cryptographic identity, mTLS and short-lived signed token | Validate issuer, signature, audience, expiry and scope at every boundary |
| CBS Gateway | Dedicated isolated legacy identity | Canonical-command policy and mainframe entitlement |
| Administrator | Separate privileged identity and managed device | Target-specific JIT role, approval and recorded session; no standing privilege |

Every governed artifact carries one sensitivity class (`PUBLIC`, `INTERNAL`, `CONFIDENTIAL`, `RESTRICTED`) and one retention/legal class (`DELETABLE_PERSONAL`, `OPERATIONAL`, `REGULATED_FINANCIAL`, `SECURITY_AUDIT`). Restricted data is encrypted in transport and at rest; high-risk fields use tokenization, field encryption or non-reversible hashing as appropriate. Cloud, on-premises and mainframe root/key-encryption keys remain in separate trust domains; production and non-production keys are separate. Root, recovery, destructive rotation and export operations require dual control and immutable evidence. Secrets never appear in source, ordinary configuration, events, audit payloads or logs.

Transfer, consent, TPP, fraud approval, CBS execution, cryptographic validation, privileged access and required durable evidence fail closed. A read-only path may continue only with valid identity/policy and an approved, non-quarantined freshness state. Reports and AI may degrade; they cannot bypass controls.

### Threat-flow controls

| Flow | Dominant STRIDE concerns | Required controls and evidence |
|---|---|---|
| Public ingress/session | Identity theft, replay/tampering, disclosure, abuse and scope elevation | OIDC/MFA, TPP mTLS, signed-token validation, schema/idempotency, WAF/rate controls, sanitized errors and policy-version evidence |
| TPP consent/AIS/PIS | Forged or expired consent, account overreach and repudiation | Server-issued ConsentContext, SCA reference, permitted-account/scope checks, expiry/revocation, canonical consent evidence |
| Transfer/Fraud/CBS | Duplicate or altered command, spoofed decision, unknown result and privileged bypass | Trusted authenticated context, idempotency, binary fraud decision, canonical gateway, `PROCESSING` reconciliation and immutable transition receipts |
| CDC/Event Bus/ODS | Source spoofing, reordering, duplicate effects, poison events and data leakage | Isolated publisher, source position, per-account order, idempotent projection, quarantine, replay evidence and payload minimization |
| AI Advisor | Prompt/data leakage, eligibility overreach and untraceable recommendation | Curated context, bank-rule candidates, purpose scopes, provenance/model version, output controls and no direct data-plane access |
| Privileged/key/telemetry | Account takeover, evidence tampering, secret exposure and denial of control | JIT PAM, dual control, HSM/KMS separation, session recording, append-only receipts, canaries and independent security monitoring |

Security evidence includes identity/token outcome; policy-version authorization; consent context; cross-zone correlation; transfer/fraud/CBS references; certificate/key lifecycle; privileged approval/session/command; data export/deletion/retention; edge and firewall events; and fail-closed or break-glass outcomes.

## 3.2.3 Observability model

The platform measures eight end-to-end journeys from managed edge to authoritative outcome. Correct business declines are not technical failures; timeouts, 5xx, incorrect outcomes and undisclosed stale data are failures. An HTTP 2xx/202 alone is never proof of transfer success.

| # | Journey / tier | Authoritative SLI and target | Primary owner |
|---:|---|---|---|
| 1 | Authentication / T0 | Correct authn/authz outcome; 99.999% availability | Identity & Edge |
| 2 | Account/balance read / T0 | Correct authority/freshness; 99.999%, server p95 ≤ 200 ms | Account Information |
| 3 | Transfer/Fraud/CBS/settlement / T0 | Separate CBS outcome, `PROCESSING` age, settlement and reconciliation; 99.999%; latency target open | Payments |
| 4 | Open Banking consent/AIS/PIS / T0 | Correct verified-consent outcome; 99.999% | Open Banking |
| 5 | CDC/Event Bus/ODS/cache / T1 | Ordered, deduplicated projection; p95 ≤ 5 s, p99 ≤ 30 s | Data Platform |
| 6 | Statement/report / T2 | Immutable reconciled artifact by T+1 | Reporting |
| 7 | AI Advisor / T2 | Eligible recommendation with provenance; dataset age ≤ 24 h | AI Advisor Product |
| 8 | Privileged/security evidence / T1 | Complete durable canonical receipt | Security Operations |

Metrics and business-outcome counters cover 100% of eligible operations. The correlation spine uses `traceparent`, `correlationId`, `eventId`, `causationId` and opaque domain identifiers. Every cloud, on-premises, Event Bus and CBS boundary preserves or explicitly translates it. Cross-zone telemetry uses buffered asynchronous delivery; a delivery gap is observable and never changes an authoritative business outcome or bypasses fail-secure evidence.

Adaptive sampling is allowed for normal successful traces. Errors, slow operations, unknown transfers, mismatches, quarantines, security incidents and required audit evidence retain complete approved evidence. Minimization occurs before persistence; telemetry rejects secrets, credentials, raw IBANs, raw CBS records and unnecessary PII.

| Evidence class | Retention baseline |
|---|---|
| High-resolution metrics | 90 days |
| Aggregated SLI/business metrics | 13 months |
| Normal traces | 30 days |
| Operational logs and exceptional traces | 90 days |
| Incident packages | 1 year after closure |
| Security telemetry | 1 year searchable, then policy archive |
| Audit evidence | Legal/Compliance policy; legal hold overrides deletion |

The platform provides one Bank Health dashboard, eight journey dashboards, cross-zone diagnostics and separated security/audit views. Alerting is journey-first: P1 acknowledges within 5 minutes using 14.4× burn in 5-minute and 1-hour windows; P2 within 15 minutes using 6× in 30-minute and 6-hour windows; P3 by next business day using 1× in 6-hour and 3-day windows. Financial-integrity, audit-gap and critical-security signals bypass burn windows and open P1.

Production canaries run at most every minute for authentication/account reads, every 15 minutes for controlled Transfer/PIS writes, every 5 minutes for CDC freshness, and after each scheduled statement/AI refresh. They use synthetic identities and controlled accounts and do not contaminate business KPIs, fraud policy or model training. P1/P2 incidents create an evidence bundle and link an executable runbook tested quarterly and after material changes.

## 3.2.4 Resilience and disaster recovery

| Class | Maximum RTO | Maximum RPO | Typical state |
|---|---:|---:|---|
| RC0 — authority/evidence | 15 min | Zero permanent loss | Commands, outcomes, idempotency, consent, integration checkpoints and audit receipts |
| RC1 — customer-critical access | 30 min | 5 min for reconstructable state; RC0 stays zero-loss | Auth, account serving and API layer |
| RC2 — replayable projections | 4 h | Zero permanent loss after replay | ODS projections and complete convergence |
| RC3 — degradable derived | 24 h | 24 h | Statements under generation and reproducible AI/report datasets |

Tier-0 99.999% availability uses automatic local HA. Catastrophic DR remains SLO evidence and may exhaust the error budget; the DR objective is not a waiver.

The primary cloud region and on-premises Digital Core each use at least two local failure domains. A second cloud failure domain and geographically separate on-premises site provide warm standby. Two provider/PoP-diverse private links plus encrypted VPN fallback connect cloud and bank; routing convergence target is 60 seconds and every path preserves NGFW, allowlists, mTLS, correlation and default deny. The MVP prohibits active-active money movement: CBS remains the single writer.

RC0 acknowledgement occurs only after approved zero-loss recoverability. Stateful systems use continuous logs/PITR, daily encrypted backup and immutable isolated cross-site copy; replication is not backup. Event recovery preserves durable log and checkpoints. ODS/cache restore may accelerate recovery, but authoritative replay and CBS-led reconciliation prove correctness. Configuration, schemas, certificates and keys remain recoverable under separate custody.

Local HA may fail over automatically. Site/region DR may prepare automatically but cannot enable writes without dual authority. Digital promotion requires Incident Commander + Service Owner; data promotion adds Data Owner; CBS promotion requires Mainframe Authority; Security may block. Recovery fences the primary, proves replication position and single-writer state, validates idempotency, keys, controls, evidence continuity and connectivity, then opens traffic through canary → limited → full gates. Failback is manual after resynchronization and reconciliation. If single-writer authority is uncertain, money-moving writes remain disabled.

| Recovery wave | Scope |
|---:|---|
| 0 | Security/management plane and evidence verification |
| 1 | RC0 transfer, consent, event and audit state; CBS authority |
| 2 | RC1 auth, account and API serving |
| 3 | RC2 projection convergence and RC3 reports/AI |

Backup restore is tested monthly; component/failure-domain failover quarterly; end-to-end RC0/RC1 every six months; full multi-domain failover and failback annually; and targeted retest follows material change. Each exercise records achieved RTO/RPO, zero-loss result, fencing, degraded behavior, reconciliation, ownership and canonical evidence. A failed test is a production-readiness risk.

## 3.2.5 Open parameters

The following remain explicit: Product/Risk must approve numeric transfer latency; Legal/Compliance owns statutory audit retention; Security owns the privileged-operation canary interval; Infrastructure/Compliance must select cloud failure domains and DR site; Mainframe/Operations must prove CBS meets RC0; and product teams must select database, broker, backup and recovery tooling without weakening these contracts.
