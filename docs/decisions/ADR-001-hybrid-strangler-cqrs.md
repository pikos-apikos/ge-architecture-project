# ADR-001 — Use Hybrid Strangler Fig Architecture with CQRS-style Read Models

## Status

Accepted for MS1 draft.

## Context

NeoBank must deliver new digital banking capabilities within one year while preserving the existing CBS/mainframe estate. The CBS is based on COBOL/z/OS and remains responsible for legal transaction execution, strong consistency, and ledger correctness.

The new digital services must support mobile, web, Open Banking, fraud detection, reporting, and AI advisory use cases. The architecture must also support 99.999% uptime, no data loss, GDPR compliance, and growth from 100K digital users in year 1 to 1M digital users within three years.

Replacing the CBS during the MVP would create unacceptable delivery, operational, regulatory, and data integrity risk.

## Decision

Use a **hybrid Strangler Fig modernization architecture** combined with **CQRS-style separation of commands and queries**.

- The CBS remains the system of record for financial transactions.
- All money-moving commands go through the Transfer Service and CBS Transaction Gateway.
- Read-heavy digital experiences use replicated read models and operational data stores.
- New digital capabilities are introduced around the existing CBS instead of replacing it.
- Regulated and CBS-adjacent components run on-premises.
- Cloud-allowed digital edge and AI workloads run in the regional cloud.

## Consequences

### Positive

- Reduces CBS change risk during MVP.
- Protects the mainframe from direct digital traffic.
- Enables faster delivery of new digital channels.
- Allows independent scaling of read-heavy workloads.
- Supports a gradual modernization roadmap.

### Negative

- Introduces data replication and freshness management complexity.
- Requires clear reconciliation between CBS, events, and read models.
- Requires strict design discipline to prevent ODS from becoming a second system of record.
- Adds operational complexity across cloud, on-premises, and legacy environments.

## Design Rules

- CBS is authoritative for balances and transfers.
- ODS/read models are projections, not transaction authorities.
- Customer-facing account screens must show freshness metadata where relevant.
- All transfer requests must be idempotent and auditable.
- No digital channel may call CBS, DB2, or ADABAS directly.
