# 3.4 Managed API and Event Contracts

This HLD manages fourteen logical contracts. The main body defines ownership, purpose, delivery, compatibility, and trust boundaries. Appendix B contains the complete field tables and six JSON examples. This is intentionally not a full OpenAPI or AsyncAPI specification.

## 3.4.1 Common REST convention

- API major versions appear in the URI, for example `/v1`.
- Requests carry `X-Correlation-ID` and `traceparent`.
- Money-moving commands carry `Idempotency-Key`; queries do not require it.
- Errors use RFC 9457 `application/problem+json` with stable bank extensions.
- Monetary values use `{ "value": "<decimal-string>", "currency": "<ISO-4217>" }`.
- Additive, backward-compatible changes are allowed inside a major version.
- Breaking changes require a new major version and a concurrent deprecation window.
- Producers do not repurpose fields; consumers tolerate unknown optional fields.

## 3.4.2 Common event convention

Events use an envelope containing `eventId`, `eventType`, `schemaVersion`, `occurredAt`, `correlationId`, `causationId`, `producer`, `sourceSystem`, `partitionKey`, `sourcePosition`, and `payload`.

Delivery is at least once. Consumers deduplicate by `eventId` and enforce the applicable aggregate version or source position. `partitionKey` identifies the approved ordering scope. `sourcePosition` is mandatory for legacy CDC and may be null for native digital events.

## 3.4.3 Contract catalog

| # | Contract | Boundary and owner | Architectural purpose |
|---:|---|---|---|
| 1 | `TransferCommand` | Public/internal caller to Transfer Service | Unified internal/IBAN command with trusted identity outside the body |
| 2 | `TransferStatusResult` | Transfer Service to channels | Independent execution and settlement status with CBS-confirmed references |
| 3 | `FraudDecision` | Fraud Engine to Transfer Service | Binary APPROVE/DECLINE business decision; transport failure is separate |
| 4 | `CbsTransactionCommandResult` | Transfer Service and CBS Gateway | Canonical anti-corruption command/result; raw CBS protocol stays in Gateway |
| 5 | `AccountInformationResponse` | Account Information Service to channels | Summary and paged history with freshness and write capability |
| 6 | `ConsentContext` | Consent Service to downstream services | Short-lived verified TPP/PSU/scope/account/SCA context |
| 7 | `AdvisorContextResponse` | Advisor Context Service to AI Advisor | Curated advisory snapshot; bank rules own eligibility |
| 8 | `LegacyDataChangeEvent` | Mainframe CDC to Event Bus | Canonical ACCOUNT/BALANCE/TRANSACTION state change |
| 9 | `TransferOutcomeEvent` | Transfer Service to Event Bus | Complete current-state snapshot after every persisted transfer transition |
| 10 | `ProjectionStatusEvent` | Projection Service to Event Bus | Account-scoped applied/quarantined/reconciling status snapshot |
| 11 | `MonthlyStatementSnapshot` | Reporting Service | Immutable, versioned and reconciled monthly statement |
| 12 | `AuditEvidenceEvent` | Regulated action owner | Canonical evidence receipt with references and integrity metadata |
| 13 | `ConsentChangedEvent` | Consent Service | Complete current consent state for creation, scope change, suspension, revocation or expiry |
| 14 | `CacheInvalidationEvent` | Authoritative projection/transfer owner | Version-aware invalidation for account-scoped cached views |

## 3.4.4 Authority rules

- CBS owns transfer commit, rejection, and returned balances.
- Transfer Service owns the durable transfer workflow state.
- Fraud Engine owns fraud policy decisions.
- Consent Service owns consent validity and scope.
- ODS Projection Service owns projection version, freshness, and quarantine status, but not business balances.
- Reporting Service owns the immutable statement artifact after reconciliation.
- Bank eligibility rules own product eligibility; AI owns only ranking and explanation.
- Audit evidence records the owning system's outcome and integrity references; it does not become a second authority.

## 3.4.5 Publication and failure rules

`TransferOutcomeEvent` is published from a transactional outbox after every durable state transition. `ProjectionStatusEvent` is published after status persistence. `ConsentChangedEvent` is published after a durable consent transition. Cache invalidation is version-aware and cannot make an older view appear current.

If publication is delayed, the durable source state remains authoritative and the outbox retries. If a consumer fails, delivery resumes at least once and the consumer deduplicates. A timeout at the Fraud Engine or CBS Gateway does not create a false business decision.

## 3.4.6 Documentation depth

Appendix B includes a field table for all fourteen contracts with field, logical type, requirement status, authority/source, meaning, and rule. Full JSON examples are included for:

1. `TransferCommand`
2. `TransferStatusResult`
3. `FraudDecision`
4. `CbsTransactionCommandResult`
5. `AccountInformationResponse`
6. `LegacyDataChangeEvent`

`OfflineFraudFindingEvent` is outside the approved catalog. Open Banking reuses the managed account, transfer, and consent schemas.
