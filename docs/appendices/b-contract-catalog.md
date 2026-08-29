# Appendix B — Managed API and Event Contract Catalog

## Comment 22

## B.1 API and value conventions

This comment compiles the approved decisions into HLD-level field tables. It does not define a complete OpenAPI specification.

### B.1.1 Common API and value conventions

- API major versions MUST appear in the URI, for example `/v1`.
- Requests MUST carry `X-Correlation-ID` and `traceparent`.
- Money-moving commands MUST carry `Idempotency-Key`.
- `Money` means `{ value: decimal-string, currency: ISO-4217-code }`.
- `Date` means an ISO 8601 calendar date.
- `Timestamp` means an RFC 3339 UTC timestamp.
- `Id` means an opaque string. Consumers MUST NOT parse business meaning from an identifier.
- REST errors MUST use RFC 9457 `application/problem+json` with the approved bank extensions.

### 1. `TransferCommand`

**Operation:** `POST /v1/transfers`  
**Authority:** Transfer Service validates the command; authenticated context owns customer, channel, and authorization identity.

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `debtorAccountId` | Id | Yes | Caller body, validated against authenticated context | Account to debit. Caller MUST be authorized for this account. |
| `creditor` | `Creditor` | Yes | Caller | Typed destination. `type` MUST be `INTERNAL_ACCOUNT` or `IBAN`. |
| `creditor.type` | enum | Yes | Caller | Selects exactly one destination shape. |
| `creditor.accountId` | Id | Conditional | Caller | REQUIRED only for `INTERNAL_ACCOUNT`; prohibited for `IBAN`. |
| `creditor.iban` | string | Conditional | Caller | REQUIRED only for `IBAN`; MUST pass IBAN validation. |
| `creditor.name` | string | Conditional | Caller | REQUIRED for `IBAN`; governed length and character rules apply. |
| `creditor.bic` | string | No | Caller | Optional for `IBAN`; MUST pass BIC validation when present. |
| `amount` | Money | Yes | Caller | `value` MUST be greater than zero; currency MUST be supported for the selected rail/account. |
| `requestedExecutionDate` | Date | Yes | Caller | Requested banking execution date; service validates supported scheduling window. |
| `remittanceInformation` | string | Yes | Caller | Payment reference; governed length and permitted-character rules apply. |

`Idempotency-Key` is a REQUIRED request header. Customer identity, channel identity, and Open Banking `consentId` MUST come from trusted authenticated or verified context, not from the request body.

### 2. `TransferStatusResult`

**Operations:** command response and `GET /v1/transfers/{transferId}`  
**Authority:** Transfer Service; CBS remains authoritative for execution commit and returned balances.

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `transferId` | Id | Yes | Transfer Service | Stable transfer identifier. |
| `executionStatus` | enum | Yes | Transfer Service from Fraud/CBS state | `RECEIVED`, `FRAUD_PENDING`, `FRAUD_REJECTED`, `CBS_PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`, or `NOT_SUBMITTED`. |
| `settlementStatus` | enum | Yes | Transfer/settlement integration | `NOT_APPLICABLE`, `PENDING`, `SETTLED`, `RETURNED`, or `REVERSED`. |
| `statusReasonCode` | stable code or null | Yes | Owning state transition | Machine-readable reason. Raw CBS codes MUST NOT appear. |
| `statusUpdatedAt` | Timestamp | Yes | Transfer Service | Time of the latest persisted transfer-state transition. |
| `cbsTransactionId` | Id | Conditional | CBS | REQUIRED when CBS committed; absent before a definitive commit. |
| `availableBalance` | Money | Conditional | CBS | Present only when CBS returns a confirmed balance. |
| `currentBalance` | Money | Conditional | CBS | Present only when CBS returns a confirmed balance. |

`PROCESSING` means that the CBS outcome is unknown and reconciliation is required. `COMPLETED` means CBS committed. For an external transfer, `COMPLETED` with `settlementStatus=PENDING` MUST NOT be described as confirmed beneficiary receipt.

### 3. `FraudDecision`

**Boundary:** Real-Time Fraud Engine response  
**Authority:** Fraud Engine owns approve/decline policy.

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `decisionId` | Id | Yes | Fraud Engine | Stable auditable decision identifier. |
| `decision` | enum | Yes | Fraud Engine | MUST be `APPROVE` or `DECLINE`; `REVIEW` is outside MVP scope. |
| `riskScore` | decimal | Yes | Fraud Engine | Governed scoring range; informational to callers, not an independent decision input. |
| `reasonCodes` | array of stable codes | Yes | Fraud Engine | Explains the decision; MAY be empty for an approval only if policy permits. |
| `policyVersion` | string | Yes | Fraud Engine | Exact policy/rules version used. |
| `modelVersion` | string | Conditional | Fraud Engine | REQUIRED when an ML model contributed. |
| `evaluatedAt` | Timestamp | Yes | Fraud Engine | Decision time. |
| `validUntil` | Timestamp | Yes | Fraud Engine | Decision reuse expiry; MUST be later than `evaluatedAt`. |

A timeout, connection failure, or 5xx MUST NOT create this contract and MUST NOT be converted to `DECLINE`. It activates the approved hold-and-retry policy.

### 4. `CbsTransactionCommandResult`

**Boundary:** Digital Core ↔ CBS Transaction Gateway  
**Authority:** Gateway maps the canonical contract; CBS owns the definitive transaction outcome and balances.

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `transferId` | Id | Yes | Transfer Service | Links command and result to the managed transfer. |
| `idempotencyKey` | string | Yes | Transfer Service | Same key MUST identify retries of the same canonical command. |
| `debtorAccountId` | Id | Yes | Transfer Service | Canonical debtor account identifier. |
| `creditor` | `Creditor` | Yes | Transfer Service | Same typed destination semantics as `TransferCommand`. |
| `amount` | Money | Yes | Transfer Service | Exact canonical amount sent for CBS execution. |
| `requestedExecutionDate` | Date | Yes | Transfer Service | Canonical requested execution date. |
| `fraudDecisionId` | Id | Yes | Fraud Engine via Transfer Service | MUST reference a valid `APPROVE` decision. |
| `outcome` | enum | Result only | CBS through Gateway | Definitive result MUST be `COMMITTED` or `REJECTED`. No result is fabricated after timeout. |
| `cbsTransactionId` | Id | Conditional | CBS | REQUIRED for `COMMITTED`; prohibited for a definitive rejection. |
| `bookedAt` | Timestamp | Conditional | CBS | REQUIRED for `COMMITTED`. |
| `availableBalance` | Money | Conditional | CBS | REQUIRED for `COMMITTED` when CBS returns it. |
| `currentBalance` | Money | Conditional | CBS | REQUIRED for `COMMITTED` when CBS returns it. |
| `reasonCode` | stable domain code | Conditional | Gateway mapping | REQUIRED for `REJECTED`; raw CBS codes remain inside the Gateway boundary. |

A lost connection or timeout produces no definitive `CbsTransactionCommandResult`; the Transfer Service moves to `PROCESSING` pending reconciliation.

### 5. `AccountInformationResponse`

**Operations:** `GET /v1/accounts/{accountId}` and `GET /v1/accounts/{accountId}/transactions?cursor=...&limit=...`  
**Authority:** CBS is authoritative; ODS supplies the non-authoritative read projection and freshness metadata.

### 5.1 Account summary

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `accountId` | Id | Yes | CBS identity via ODS | Stable account identifier. |
| `maskedAccountIdentifier` | string | Yes | Account projection | Masked account number or IBAN; full identifier MUST NOT be exposed unless the channel is authorized to receive it. |
| `accountType` | stable code | Yes | CBS via ODS | Managed account classification. |
| `currency` | ISO 4217 code | Yes | CBS via ODS | Account currency. |
| `availableBalance` | Money | Yes | CBS via ODS | Projected CBS balance; currency MUST match account currency. |
| `currentBalance` | Money | Yes | CBS via ODS | Projected CBS balance; currency MUST match account currency. |
| `projectionVersion` | monotonic version | Yes | ODS Projection Service | Account-scoped applied projection version. |
| `dataAsOf` | Timestamp | Yes | ODS Projection Service | Latest source-data time represented. |
| `freshnessStatus` | governed enum | Yes | ODS Projection Service | Exposes whether the read model satisfies the approved freshness contract. |
| `lastUpdated` | Timestamp | Yes | ODS Projection Service | Projection persistence time. |
| `writeCapability` | enum | Yes | Account/Transfer policy | `ENABLED` or `BLOCKED`; quarantine or unsafe freshness MUST block writes as approved. |

### 5.2 Transaction-history page

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `items` | array of `TransactionItem` | Yes | CBS via ODS | Bounded page; MUST NOT return unbounded history. |
| `items[].transactionId` | Id | Yes | Read model | Stable managed transaction identifier. |
| `items[].cbsTransactionId` | Id | No | CBS | Present when available. |
| `items[].bookingDate` | Date | Yes | CBS | CBS booking date. |
| `items[].valueDate` | Date | Yes | CBS | CBS value date. |
| `items[].amount` | Money | Yes | CBS | Absolute transaction amount. |
| `items[].direction` | enum | Yes | Canonical mapping | `CREDIT` or `DEBIT`. |
| `items[].counterparty` | object | No | CBS/canonical mapping | Data-minimized counterparty details. |
| `items[].remittanceInformation` | string | No | CBS/canonical mapping | Payment reference when applicable. |
| `items[].executionStatus` | enum | No | Transfer projection | Included for managed transfers when applicable. |
| `items[].settlementStatus` | enum | No | Settlement projection | Included when applicable. |
| `nextCursor` | opaque string | No | Account Information Service | Present only when another page exists. |
| `projectionVersion` | monotonic version | Yes | ODS Projection Service | Same account-scoped ordering contract as summary. |
| `dataAsOf` | Timestamp | Yes | ODS Projection Service | Latest source-data time represented. |
| `freshnessStatus` | governed enum | Yes | ODS Projection Service | Same freshness semantics as summary. |
| `lastUpdated` | Timestamp | Yes | ODS Projection Service | Projection persistence time. |

### 6. `ConsentContext`

**Boundary:** trusted server-issued context for Open Banking AIS/PIS  
**Authority:** Consent Service validates TPP, PSU, scope, accounts, validity, and SCA.

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `consentId` | Id | Yes | Consent Service | Validated consent identifier. |
| `tppId` | Id | Yes | Authenticated TPP identity | MUST match the authenticated TPP. |
| `psuSubjectId` | Id | Yes | Consent Service | Customer/PSU subject bound to consent. |
| `permittedAccountIds` | array of Id | Yes | Consent Service | Requested account MUST be in this set. |
| `scopes` | array of enum | Yes | Consent Service | Minimum supported: `AIS_READ_ACCOUNTS`, `AIS_READ_TRANSACTIONS`, `PIS_INITIATE_TRANSFER`. |
| `status` | enum | Yes | Consent Service | `ACTIVE`, `REVOKED`, `EXPIRED`, or `SUSPENDED`; downstream processing is permitted only for `ACTIVE`. |
| `validFrom` | Timestamp | Yes | Consent Service | Consent validity start. |
| `validUntil` | Timestamp | Yes | Consent Service | Consent validity end. |
| `authenticationStrength` | stable code | Yes | Authentication/SCA service | Verified assurance level. |
| `scaReference` | Id | Conditional | SCA service | REQUIRED when SCA applies. |
| `issuedAt` | Timestamp | Yes | Consent Service | Context issuance time. |
| `contextExpiresAt` | Timestamp | Yes | Consent Service | Short-lived context expiry; MUST NOT exceed consent validity. |

Public request bodies MUST NOT supply trusted consent attributes. A missing, expired, revoked, suspended, out-of-scope, or wrong-account context MUST be rejected before business processing. Tokens and SCA secrets MUST NOT appear.

### 7. `AdvisorContextResponse`

**Boundary:** curated AI-advisor data access  
**Authority:** governed bank rules decide eligibility; AI MAY rank and explain eligible candidates.

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `contextId` | Id | Yes | Advisor Context Service | Stable snapshot identifier for audit. |
| `customerScope` | object | Yes | Authenticated context / governed projection | Data-minimized customer and account scope. |
| `currentProductHoldings` | array | Yes | Product systems | Current holdings relevant to recommendation purpose. |
| `balanceSummary` | object of Money values | Yes | CBS-derived curated projection | Aggregated balance signals; MUST include provenance and freshness. |
| `cashFlowSummary` | object | Yes | Curated analytics | Purpose-limited income/outflow summary. |
| `transactionAggregates` | object | Yes | Curated analytics | Trends and aggregates only; unbounded raw transaction history is prohibited. |
| `preferencesOrGoals` | object | No | Customer-permitted sources | Included only when collected, permitted, and relevant. |
| `productCandidates` | array of eligible candidates | Yes | Governed bank rules | MUST contain only candidates currently marked eligible. |
| `productCandidates[].productId` | Id | Yes | Product catalog | Eligible bank product. |
| `productCandidates[].productName` | string | Yes | Product catalog | Display name. |
| `productCandidates[].eligibilityDecisionId` | Id | Yes | Eligibility rules service | Auditable eligibility decision. |
| `productCandidates[].eligibilityReasonCodes` | array | Yes | Eligibility rules service | Stable governed reason codes. |
| `productCandidates[].offerOrTermsReference` | reference | Yes | Product/offer service | AI MUST NOT alter the referenced terms. |
| `productCandidates[].disclosureReferences` | array of references | Yes | Compliance/product service | AI MUST NOT remove required disclosures. |
| `productCandidates[].validUntil` | Timestamp | Yes | Eligibility rules service | Eligibility MUST be revalidated before a binding offer or sale. |
| `dataAsOf` | Timestamp | Yes | Advisor Context Service | Latest represented data time. |
| `freshnessStatus` | governed enum | Yes | Advisor Context Service | Exposes dataset freshness. |
| `provenance` | array of source/version references | Yes | Advisor Context Service | Identifies contributing governed projections. |
| `contextVersion` | monotonic version | Yes | Advisor Context Service | Binds AI recommendation and audit to this snapshot. |

The AI MUST obtain banking data only through this API. It MUST NOT read CBS or ODS directly. It MUST NOT decide eligibility, create binding terms, or execute a sale.

---

## Comment 23

## B.2 Event contracts

### B.2.1 Common event envelope

All seven approved event contracts MUST use this envelope. Payload tables below define only the `payload` member.

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `eventId` | Id | Yes | Producer | Globally unique delivery identity used for deduplication. |
| `eventType` | stable string | Yes | Contract catalog | MUST identify one governed event type and MUST NOT be repurposed. |
| `schemaVersion` | major-compatible version | Yes | Schema governance | Additive changes only inside a major version; breaking change requires a new major. |
| `occurredAt` | Timestamp | Yes | Producer | Time the represented durable state change occurred. |
| `correlationId` | Id | Yes | Originating request/process | End-to-end correlation identifier. |
| `causationId` | Id or null | Yes | Producer | Identifier of the command/event that caused this event; null only when no cause exists. |
| `producer` | stable service code | Yes | Producer | Logical producing component, not an ephemeral instance name. |
| `sourceSystem` | stable system code | Yes | Producer | System that owns or originated the represented state. |
| `partitionKey` | string | Yes | Contract-specific authority | MUST identify the approved ordering scope. |
| `sourcePosition` | opaque string or null | Yes | Source/producer | REQUIRED for legacy CDC; MAY be null for native digital events without a legacy position. |
| `payload` | typed object | Yes | Event contract | MUST conform to the selected event payload schema version. |

Delivery is at least once. Consumers MUST deduplicate by `eventId`, tolerate unknown optional fields, and enforce the contract-specific entity version or source position.

### 8. `LegacyDataChangeEvent` payload

**Producer:** Mainframe CDC canonicalization boundary  
**Authority:** legacy CBS source data; the event is a canonical state-change representation, not a raw log record.

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `entityType` | enum | Yes | Approved CDC scope | MUST be `ACCOUNT`, `BALANCE`, or `TRANSACTION`. |
| `entityId` | Id | Yes | Canonical mapper | Stable canonical entity identity. |
| `accountId` | Id | Yes | Canonical mapper | Account ordering/quarantine scope; MUST equal envelope `partitionKey`. |
| `changeType` | enum | Yes | CDC mapper | `UPSERT` or `DELETE`. |
| `entityVersion` | opaque source version | Yes | Legacy source / mapper | Used with source position for stale-event rejection. |
| `changedFieldNames` | array of canonical field names | Yes | Canonical mapper | MUST name canonical fields, not copybook/database columns. |
| `state` | typed canonical object | Conditional | Canonical mapper | REQUIRED for `UPSERT`; MUST be the after-image. Prohibited for `DELETE`. |
| `deletedAt` | Timestamp | Conditional | Legacy source / mapper | REQUIRED for `DELETE`. |
| `deletionReasonCode` | stable code | No | Canonical mapper | Optional tombstone reason; raw legacy operation codes are prohibited. |

Raw records, copybook layouts, DB operation codes, partial mappings, and unsupported entity types MUST NOT be published. A mapping failure places only the affected account stream in quarantine.

### 9. `TransferOutcomeEvent` payload

**Producer:** Transfer Service after a durable transfer-state transition  
**Authority:** Transfer Service state; CBS remains authoritative for commit and balances.

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `transferId` | Id | Yes | Transfer Service | Aggregate identity; recommended event `partitionKey`. |
| `transferVersion` | monotonic integer | Yes | Transfer Service | Consumer MUST apply only a version greater than its last applied version. |
| `debtorAccountId` | Id | Yes | Transfer Service | Account affected by the transfer. |
| `executionStatus` | approved enum | Yes | Transfer Service | Same semantics as `TransferStatusResult`. |
| `settlementStatus` | approved enum | Yes | Transfer/settlement integration | Same semantics as `TransferStatusResult`. |
| `statusReasonCode` | stable code or null | Yes | Owning transition | Machine-readable reason without raw CBS codes. |
| `cbsTransactionId` | Id | No | CBS | Present after confirmed CBS commit. |
| `availableBalance` | Money | No | CBS | Present only when CBS confirmed it. |
| `currentBalance` | Money | No | CBS | Present only when CBS confirmed it. |
| `statusUpdatedAt` | Timestamp | Yes | Transfer Service | Time of the durable transition represented. |

Each event MUST be a complete current-state snapshot. It MUST NOT imply beneficiary settlement unless `settlementStatus=SETTLED`.

### 10. `ProjectionStatusEvent` payload

**Producer:** ODS Projection Service after durable projection/status persistence  
**Authority:** projection service for read-model state; CBS remains business-data authority.

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `accountId` | Id | Yes | Projection Service | Account scope; MUST equal envelope `partitionKey`. |
| `projectionVersion` | monotonic integer | Yes | Projection Service | Consumer MUST apply only a greater account version. |
| `lastAppliedEventId` | Id | Yes | Projection Service | Latest source event incorporated into the snapshot. |
| `sourcePosition` | opaque string | Yes | CDC source via projector | Latest applied source position; MUST agree with the envelope value when applicable. |
| `dataAsOf` | Timestamp | Yes | Projection Service | Latest source-data time represented. |
| `freshnessStatus` | governed enum | Yes | Projection Service | Account-specific freshness/health state. |
| `quarantineReasonCode` | stable code | Conditional | Projection Service | REQUIRED when the account projection is quarantined. |

A quarantined account MUST NOT stop other account streams. A global pipeline watermark MUST NOT replace this account-scoped event.

### 11. `MonthlyStatementSnapshot`

**Producer:** Reporting Service after CBS–ODS reconciliation  
**Authority:** immutable reconciled reporting artifact; not live-balance authority.

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `statementId` | Id | Yes | Reporting Service | Immutable statement-version identity. |
| `accountId` | Id | Yes | CBS identity | Account covered by the statement. |
| `statementPeriod` | `{startDate,endDate}` | Yes | Reporting Service | Closed monthly interval; dates MUST form one valid statement period. |
| `statementVersion` | monotonic integer | Yes | Reporting Service | Version within account and period. |
| `currency` | ISO 4217 code | Yes | CBS/reporting rules | Statement currency. |
| `openingBalance` | Money | Yes | Reconciled CBS data | Currency MUST match statement currency. |
| `closingBalance` | Money | Yes | Reconciled CBS data | Currency MUST match statement currency. |
| `lineItems` | ordered array | Yes | Reconciled transaction projection | Complete statement-period lines with transaction ID, dates, amount, direction, and description/reference. |
| `periodTotals` | typed Money totals | Yes | Reporting Service | Reconciled debit/credit totals; currencies MUST match statement currency. |
| `dataAsOf` | Timestamp | Yes | Reporting Service | Latest source data included. |
| `reconciledAt` | Timestamp | Yes | Reconciliation Service | Successful reconciliation time. |
| `generatedAt` | Timestamp | Yes | Reporting Service | Immutable artifact creation time; MUST meet T+1 SLO. |
| `supersedesStatementId` | Id | No | Reporting Service | REQUIRED for a correction; earlier version MUST remain retained. |

An issued statement MUST NOT be overwritten. A correction MUST create a new version and reference the superseded statement.

### 12. `AuditEvidenceEvent` payload

**Producer:** governed platform/service audit boundary  
**Authority:** append-only evidence ledger.

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `auditEvidenceId` | Id | Yes | Audit producer | Immutable evidence-receipt identity. |
| `actionType` | stable code | Yes | Governed audit catalog | Auditable action or decision type. |
| `actorReference` | typed reference | Yes | Authenticated/service context | Human, TPP, service, or system actor; data minimized. |
| `subjectReferences` | array of typed references | Yes | Business context | Affected customer/account/consent/transfer subjects as permitted. |
| `resourceType` | stable code | Yes | Business service | Type of affected resource. |
| `resourceId` | Id | Yes | Business service | Identifier of affected resource. |
| `outcome` | stable code | Yes | Business service | Auditable result; MUST preserve business meaning. |
| `policyVersions` | map of name→version | Yes | Decision services | Applicable policy, rule, and model versions; empty only when none apply. |
| `occurredAt` | Timestamp | Yes | Business service | Audited action time; MUST agree with envelope semantics. |
| `evidenceReferences` | array of protected references | Yes | Evidence store | References and cryptographic hashes; MAY be empty when no artifact exists. |
| `dataClassification` | stable code | Yes | Data governance | Classification governing access. |
| `retentionPolicyId` | Id | Yes | Records governance | Governing retention/disposal policy. |
| `correctsEvidenceId` | Id | No | Audit producer | A correction appends new evidence; earlier evidence remains immutable. |

The common audit stream MUST NOT copy complete request/response payloads or unnecessary PII. Protected artifacts remain behind governed references.

### 13. `ConsentChangedEvent` payload

**Producer:** Consent Service after a durable consent transition  
**Authority:** Consent Service.

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `consentId` | Id | Yes | Consent Service | Consent aggregate identity; MUST equal envelope `partitionKey`. |
| `consentVersion` | monotonic integer | Yes | Consent Service | Consumer MUST apply only a greater version. |
| `status` | enum | Yes | Consent Service | `ACTIVE`, `REVOKED`, `EXPIRED`, or `SUSPENDED`. |
| `tppId` | Id | Yes | Consent Service | Bound TPP identity. |
| `psuSubjectId` | Id | Yes | Consent Service | Bound PSU/customer subject. |
| `scopes` | array of enum | Yes | Consent Service | Complete current authorized scope set. |
| `permittedAccountIds` | array of Id | Yes | Consent Service | Complete current permitted-account set. |
| `validFrom` | Timestamp | Yes | Consent Service | Consent validity start. |
| `expiresAt` | Timestamp | Yes | Consent Service | Consent expiry. |
| `effectiveAt` | Timestamp | Yes | Consent Service | Time this state became effective. |
| `reasonCode` | stable code | Conditional | Consent Service | REQUIRED for revocation/suspension when a reason is available by policy. |

The event is a complete current-state snapshot and MUST NOT contain access tokens, refresh tokens, secrets, or SCA credentials. `REVOKED` and `SUSPENDED` MUST trigger related consent-cache invalidation.

### 14. `CacheInvalidationEvent` payload

**Producer:** service that durably advances the governed source version  
**Authority:** none for business data; performance/convergence hint only.

| Field | Logical type | Req. | Authority/source | Meaning and rule |
|---|---|---:|---|---|
| `resourceType` | stable code | Yes | Producer contract | Logical resource category; MUST NOT name a cache vendor or physical node. |
| `resourceId` | Id | Yes | Producer | Logical resource identity and recommended ordering key. |
| `affectedViews` | array of stable view codes | Yes | Producer contract | Logical read views to invalidate; MUST NOT contain physical cache keys. |
| `sourceVersion` | monotonic/opaque governed version | Yes | Authoritative producer/projection | Cached entries older than this version MUST be invalidated. |
| `reasonCode` | stable code | Yes | Producer | Explains the source change that requires invalidation. |
| `effectiveAt` | Timestamp | Yes | Producer | Time the newer source state became effective. |

The event MUST NOT carry replacement business data. Cache correctness MUST also use the approved TTL and freshness/version checks. This event MUST NOT become an authority for balances, transfers, consents, or product eligibility.

---

## Comment 24

## B.3 Contract examples

These examples are illustrative HLD contracts. Values are fictitious. They do not form a complete OpenAPI or AsyncAPI specification.

### JSON example 1 — `TransferCommand`

Required headers: `X-Correlation-ID`, `traceparent`, and `Idempotency-Key`.

```json
{
  "debtorAccountId": "acc_01J7A9K3P4M6Q8R2S5T0V1WXYZ",
  "creditor": {
    "type": "IBAN",
    "iban": "DE89370400440532013000",
    "name": "Example Supplier GmbH",
    "bic": "COBADEFFXXX"
  },
  "amount": {
    "value": "1250.40",
    "currency": "EUR"
  },
  "requestedExecutionDate": "2026-08-28",
  "remittanceInformation": "Invoice INV-2026-1042"
}
```

### JSON example 2 — `TransferStatusResult`

This example shows a CBS-committed external transfer whose beneficiary settlement is still pending.

```json
{
  "transferId": "trf_01J7AA1B2C3D4E5F6G7H8J9KLM",
  "executionStatus": "COMPLETED",
  "settlementStatus": "PENDING",
  "statusReasonCode": null,
  "statusUpdatedAt": "2026-08-27T19:42:18Z",
  "cbsTransactionId": "cbs_tx_849205771",
  "availableBalance": {
    "value": "8749.60",
    "currency": "EUR"
  },
  "currentBalance": {
    "value": "8749.60",
    "currency": "EUR"
  }
}
```

### JSON example 3 — `FraudDecision`

```json
{
  "decisionId": "frd_01J7AB2C3D4E5F6G7H8J9K0LMN",
  "decision": "APPROVE",
  "riskScore": 0.08,
  "reasonCodes": [
    "KNOWN_DEVICE",
    "USUAL_PAYMENT_PATTERN"
  ],
  "policyVersion": "transfer-policy-2026.08",
  "modelVersion": "rt-fraud-model-17",
  "evaluatedAt": "2026-08-27T19:42:14Z",
  "validUntil": "2026-08-27T19:47:14Z"
}
```

A technical failure returns no `FraudDecision`; it is not represented by a synthetic decline payload.

### JSON example 4 — `CbsTransactionCommandResult`

Canonical Gateway command:

```json
{
  "transferId": "trf_01J7AA1B2C3D4E5F6G7H8J9KLM",
  "idempotencyKey": "idem_01J7A9ZZYXWVUTSRQPONMLKJHG",
  "debtorAccountId": "acc_01J7A9K3P4M6Q8R2S5T0V1WXYZ",
  "creditor": {
    "type": "IBAN",
    "iban": "DE89370400440532013000",
    "name": "Example Supplier GmbH",
    "bic": "COBADEFFXXX"
  },
  "amount": {
    "value": "1250.40",
    "currency": "EUR"
  },
  "requestedExecutionDate": "2026-08-28",
  "fraudDecisionId": "frd_01J7AB2C3D4E5F6G7H8J9K0LMN"
}
```

Definitive committed result:

```json
{
  "transferId": "trf_01J7AA1B2C3D4E5F6G7H8J9KLM",
  "outcome": "COMMITTED",
  "cbsTransactionId": "cbs_tx_849205771",
  "bookedAt": "2026-08-27T19:42:18Z",
  "availableBalance": {
    "value": "8749.60",
    "currency": "EUR"
  },
  "currentBalance": {
    "value": "8749.60",
    "currency": "EUR"
  },
  "reasonCode": null
}
```

A timeout or lost connection returns no definitive result. The Transfer Service records `PROCESSING` until reconciliation resolves the outcome.

### JSON example 5 — `AccountInformationResponse` schema family

Account-summary response:

```json
{
  "accountId": "acc_01J7A9K3P4M6Q8R2S5T0V1WXYZ",
  "maskedAccountIdentifier": "GR••••••••••••••••••••••1234",
  "accountType": "CURRENT_ACCOUNT",
  "currency": "EUR",
  "availableBalance": {
    "value": "8749.60",
    "currency": "EUR"
  },
  "currentBalance": {
    "value": "8749.60",
    "currency": "EUR"
  },
  "projectionVersion": 41872,
  "dataAsOf": "2026-08-27T19:42:18Z",
  "freshnessStatus": "CURRENT",
  "lastUpdated": "2026-08-27T19:42:21Z",
  "writeCapability": "ENABLED"
}
```

Cursor-paginated transaction-history response:

```json
{
  "items": [
    {
      "transactionId": "txn_01J7AC3D4E5F6G7H8J9K0LMNPQ",
      "cbsTransactionId": "cbs_tx_849205771",
      "bookingDate": "2026-08-27",
      "valueDate": "2026-08-28",
      "amount": {
        "value": "1250.40",
        "currency": "EUR"
      },
      "direction": "DEBIT",
      "counterparty": {
        "name": "Example Supplier GmbH",
        "maskedAccountIdentifier": "DE••••••••••••••••••13000"
      },
      "remittanceInformation": "Invoice INV-2026-1042",
      "executionStatus": "COMPLETED",
      "settlementStatus": "PENDING"
    }
  ],
  "nextCursor": "cur_eyJ2Ijo0MTg3MiwicG9zIjoxfQ",
  "projectionVersion": 41872,
  "dataAsOf": "2026-08-27T19:42:18Z",
  "freshnessStatus": "CURRENT",
  "lastUpdated": "2026-08-27T19:42:21Z"
}
```

### JSON example 6 — `LegacyDataChangeEvent`

```json
{
  "eventId": "evt_01J7AD4E5F6G7H8J9K0LMNPQRS",
  "eventType": "LegacyDataChangeEvent",
  "schemaVersion": "1.0",
  "occurredAt": "2026-08-27T19:42:18Z",
  "correlationId": "cor_01J7AE5F6G7H8J9K0LMNPQRSTV",
  "causationId": "cbs_tx_849205771",
  "producer": "mainframe-cdc-canonicalizer",
  "sourceSystem": "CBS",
  "partitionKey": "acc_01J7A9K3P4M6Q8R2S5T0V1WXYZ",
  "sourcePosition": "db2:000000000004982771",
  "payload": {
    "entityType": "BALANCE",
    "entityId": "bal_01J7AF6G7H8J9K0LMNPQRSTVWX",
    "accountId": "acc_01J7A9K3P4M6Q8R2S5T0V1WXYZ",
    "changeType": "UPSERT",
    "entityVersion": "000000000004982771",
    "changedFieldNames": [
      "availableBalance",
      "currentBalance",
      "updatedAt"
    ],
    "state": {
      "availableBalance": {
        "value": "8749.60",
        "currency": "EUR"
      },
      "currentBalance": {
        "value": "8749.60",
        "currency": "EUR"
      },
      "updatedAt": "2026-08-27T19:42:18Z"
    }
  }
}
```

### B.4 Completion criteria

| Requirement | Result |
|---|---|
| Approved catalog | Exactly 14 logical contracts |
| Field documentation | Field tables for all 14 contracts |
| Full examples | Six approved critical contracts |
| API conventions | URI major version, correlation, tracing, idempotency |
| Event conventions | Common envelope, at-least-once, ordering, deduplication |
| Compatibility | Additive within major; parallel support for breaking major |
| Error model | RFC 9457 with governed bank extensions |
| Money | Decimal string plus ISO 4217 currency |
| Authority boundaries | CBS, Fraud Engine, Consent Service, bank eligibility rules preserved |
| Explicit exclusions | No raw CBS payloads/codes, no AI eligibility, no direct AI ODS/CBS access, no `OfflineFraudFindingEvent`, no full OpenAPI/AsyncAPI |




---
