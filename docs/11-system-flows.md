# 3.3 High Level System Flows

The eight journeys below are the shared unit for architecture, SLOs, security, resilience, audit, and release verification. Every journey carries `X-Correlation-ID` and `traceparent`; money-moving commands also carry `Idempotency-Key`.

## 3.3.1 Authentication and Public Ingress

Diagram: `docs/diagrams/08-authentication-public-ingress.mmd`.

1. The customer establishes TLS through the public edge.
2. WAF/DDoS controls apply bot, threat, normalization, and rate policies.
3. The API Gateway validates route, client, token, quota, and correlation policy.
4. IAM validates identity, assurance, token issuer, audience, expiry, and MFA state.
5. The Channel BFF receives trusted identity context and performs only channel composition.
6. The owning domain service repeats resource and business authorization.

The journey fails closed when trusted identity or required assurance cannot be established. A sanitized RFC 9457 problem response is returned without tokens, raw policy details, or internal topology.

## 3.3.2 Account Information Read

Diagram: `docs/diagrams/02-account-information-flow.mmd`.

The Account Information Service reads the account-scoped ODS projection and returns `AccountInformationResponse` with `projectionVersion`, `dataAsOf`, `freshnessStatus`, `lastUpdated`, and `writeCapability`. Immediately after a completed transfer, a CBS-confirmed recent-write overlay may replace an older projected balance. The overlay MUST carry the authoritative CBS transaction identifier and balances.

The service exposes account summary and transaction history as separate operations. History uses an opaque cursor and bounded limit. A quarantined account remains readable with a visible warning, but transfer initiation is blocked until reconciliation succeeds. The general 24-hour tolerance applies only to the project brief's external incoming transactions and out-of-app payments; it does not weaken read-your-writes for completed digital transfers.

## 3.3.3 Bank Transfer

Diagram: `docs/diagrams/03-bank-transfer-flow.mmd`.

The unified `POST /v1/transfers` contract supports an internal account or IBAN creditor. Customer, channel, and consent identity come from trusted context, not the body.

The Transfer Service persists every transition, calls the Fraud Engine for a binary business decision, and calls the CBS only through the Gateway. A Fraud Engine timeout creates no decline; the command remains held while a bounded retry budget is applied. A CBS timeout creates no fabricated result; the transfer enters `PROCESSING` until reconciliation determines the authoritative CBS outcome.

Execution and settlement are independent axes. `executionStatus=COMPLETED` means the CBS committed. For an external transfer, `settlementStatus=PENDING` means beneficiary receipt is not yet confirmed.

## 3.3.4 Open Banking AIS and PIS

Diagram: `docs/diagrams/09-open-banking-flow.mmd`.

TPPs use the same public ingress chain as bank channels. The Consent Service validates TPP identity, PSU, permitted accounts, scopes, status, expiry, and SCA. It issues a short-lived verified `ConsentContext`. Domain services trust only this server-issued context and enforce resource-level authorization. AIS reuses account-information schemas; PIS reuses transfer schemas. Open Banking does not create duplicate business models.

## 3.3.5 Mainframe CDC and ODS Projection

Diagram: `docs/diagrams/04-data-ingestion-read-model-flow.mmd`.

The CDC boundary publishes only canonical `ACCOUNT`, `BALANCE`, and `TRANSACTION` state changes. Delivery is at least once. Consumers deduplicate by `eventId`, order by account-scoped partition and source position, and reject stale entity versions.

A poison event or mapping failure quarantines only the affected account stream. Healthy accounts continue to project. Reconciliation is CBS-led. A rebuild clears affected projection state, replays controlled evidence, compares against CBS, and reopens writes only after an auditable match.

## 3.3.6 Monthly Statement

Diagram: `docs/diagrams/10-monthly-statement-flow.mmd`.

At month end, the Reporting Service selects the reconciled reporting period, generates an immutable, versioned `MonthlyStatementSnapshot`, and reconciles statement totals against CBS/ODS evidence. A successful snapshot is published by T+1. A failed reconciliation is held from customers and creates operational evidence; it is not silently corrected in place.

## 3.3.7 AI Financial Advisor

Diagram: `docs/diagrams/05-ai-financial-advisor-flow.mmd`.

The Advisor Context Service builds a curated snapshot with holdings, balance and cash-flow summaries, transaction aggregates, freshness, provenance, and bank-rule-approved product candidates. Governed bank rules own eligibility. The AI may rank candidates and explain relevance, but it cannot change eligibility, terms, disclosures, or execute a sale. A binding offer MUST revalidate eligibility.

The AI dataset target is at most 24 hours old. The Advisor degrades independently when the provider, model, or legal approval is unavailable. Core banking remains unaffected.

## 3.3.8 Privileged Operation and Audit

Diagram: `docs/diagrams/11-privileged-operation-audit-flow.mmd`.

Workforce administrators use a dedicated identity, phishing-resistant MFA, managed device, PAM mediation, JIT scoped privilege, explicit ticket/approval, and a recorded session. Each operation produces a canonical evidence receipt with actor, approval, target, action category, outcome, timestamp, correlation, integrity reference, and retention class. Break-glass access expires automatically and requires post-use review.

## 3.3.9 Journey SLO summary

| Journey | Tier | Primary success authority | Degradation rule |
|---|---|---|---|
| Authentication and public ingress | Tier 0 | API Gateway/IAM accepted session | Fail closed for invalid or unverifiable identity |
| Account information | Tier 0 | Successful response satisfying freshness contract | Read with warning; block writes for quarantined account |
| Bank transfer | Tier 0 / RC0 | Durable Transfer Service state plus CBS result | Hold, retry, or PROCESSING; never guess |
| Open Banking | Tier 0 | Consent and owning service outcome | Fail closed on missing/invalid consent |
| CDC and projection | Tier 1 / RC2 | Durable applied projection and reconciliation | Account-scoped quarantine and controlled replay |
| Monthly statement | Tier 2 / RC3 | Reconciled immutable snapshot | Hold publication |
| AI advisor | Tier 2 / RC3 | Valid curated context and advisory response | Graceful unavailability |
| Privileged operation/audit | Tier 1 / RC0 | PAM outcome plus evidence receipt | Deny if required evidence cannot be produced |

## 3.3.10 State and evidence invariants

- Every persisted transfer transition creates a current-state `TransferOutcomeEvent`.
- Every durable projection-status change creates an account-scoped `ProjectionStatusEvent`.
- Monthly statements are immutable, versioned, and reconciled.
- Privileged and regulated actions create canonical evidence receipts.
- No success response may precede the durability point defined by the journey.
- Retries use the same idempotency identity and MUST NOT create a second money-moving transaction.
- Audit evidence records authoritative references; it MUST NOT duplicate secrets or unrestricted payloads.
