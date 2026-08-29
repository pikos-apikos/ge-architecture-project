# ADR-002 — Use an Enterprise RDBMS for the On-Premises ODS

## Status

Accepted for MS1 draft. Vendor selection deferred to MS2.

## Context

The on-premises Digital Core requires read models and operational data stores for account information, customer views, reporting, Open Banking reads, fraud investigation, support views, and AI advisor context generation.

The ODS must support a conservative banking environment, high availability expectations, strict operations, auditability, and mature support contracts. The ODS is not the system of record for money movement. CBS remains authoritative.

## Decision

Use an **on-premises Enterprise RDBMS ODS** as the baseline architecture choice.

Candidate technologies:

- Oracle Database Enterprise Edition
- Microsoft SQL Server Enterprise Edition
- IBM Db2 LUW / Db2 pureScale

The final product selection will be decided during MS2 based on:

- Existing bank database standards
- Current licenses and support contracts
- DBA skillset
- HA/DR requirements
- Operational cost
- Integration with existing monitoring, backup, and security tooling

## Rationale

Aurora is not a good default for the on-premises Digital Core because it is an AWS-managed cloud database service. PostgreSQL may be technically valid, but it may be less politically and operationally acceptable in a conservative mainframe-based retail bank unless the bank already has strong PostgreSQL production standards.

Oracle, SQL Server, and Db2 are more natural enterprise shortlist options for an on-premises banking ODS.

## Consequences

### Positive

- Better stakeholder acceptance in a conservative bank.
- Mature HA/DR and backup patterns.
- Strong DBA tooling and vendor support options.
- Better fit for complex relational read models and operational queries.

### Negative

- Higher licensing and operational cost.
- Risk of vendor lock-in.
- Potentially slower environment provisioning than managed cloud alternatives.
- Requires clear boundaries to prevent the ODS from becoming a second system of record.

## Design Rules

- ODS stores read-optimized projections only.
- CBS remains authoritative for transaction execution and balances.
- Cache is optional and short-lived.
- Search/indexing is optional and used for investigation, not for authoritative banking state.
- ODS data freshness must be observable and visible to dependent services.

