# NeoBank HLD — On-Premises ODS Database Recommendation

## Context

The **Read Models / Operational Data Stores** component is part of the new on-premises Digital Core. It is not the legal system of record for money movement. The Core Banking System remains the system of record, and all transactions must still be executed through a CBS transaction.

The ODS exists to serve fast digital read use cases, reporting projections, fraud analytics inputs, operational support views, and API response models without putting direct user traffic on the legacy mainframe databases.

## Recommendation for MS1 / HLD

For the MS1 diagram and the first HLD version, use the following technology label:

```text
Read Models / Operational Data Stores
Enterprise RDBMS ODS
Candidate technologies: Oracle Database, SQL Server, or IBM Db2 LUW
```

This keeps the architecture vendor-aware but not prematurely vendor-locked.

## Preferred HLD Position

Use an **Enterprise RDBMS ODS** rather than PostgreSQL/Aurora for the on-premises banking deployment.

For a conservative bank, the strongest shortlist is:

1. **Oracle Database Enterprise Edition** for the on-premises ODS, especially if the bank already has Oracle licensing, Oracle DBAs, or existing Oracle operational standards.
2. **Microsoft SQL Server Enterprise Edition** if the bank is strongly Microsoft-oriented and already operates SQL Server HA/DR platforms.
3. **IBM Db2 LUW / Db2 pureScale** if the bank wants closer alignment with the existing IBM/mainframe ecosystem and has IBM database operations experience.

For the HLD document, the final decision should be written as:

```text
The ODS will be implemented on an enterprise relational database platform deployed on-premises.
The preferred candidates are Oracle Database Enterprise Edition, Microsoft SQL Server Enterprise Edition, or IBM Db2 LUW.
The final product selection depends on existing bank licensing, DBA skillset, vendor support contracts, and operational standards.
```

## Why not Aurora for the on-premises ODS?

Aurora is a managed AWS cloud database service. It fits the cloud side of the architecture, but it is not the right default for the on-premises Digital Core if the requirement is a bank-controlled on-premises deployment.

If the architecture later allows managed cloud services for some read workloads, Aurora can be considered for cloud-side projections. It should not be used as the baseline on-premises ODS in this HLD.

## Why not PostgreSQL as the default HLD answer?

PostgreSQL can be technically strong, but the architecture exercise is about a regulated retail bank with mainframe systems, strict uptime, no data loss, GDPR, and conservative operational expectations.

For this context, PostgreSQL may create unnecessary stakeholder resistance unless the bank already has strong PostgreSQL standards, DBAs, support contracts, and production HA/DR experience.

The HLD should avoid spending political capital defending PostgreSQL when Oracle, SQL Server, or Db2 will be more immediately accepted by banking stakeholders.

## Option Analysis

### Option A — Oracle Database Enterprise Edition

**Good fit when:**

- The bank already operates Oracle databases.
- The bank expects a traditional enterprise database stack.
- Strong vendor support, mature HA/DR, and DBA familiarity matter more than cost.
- The ODS must support complex relational projections, reporting views, partitioning, and operational queries.

**Architecture fit:**

Oracle is a strong default for a conservative bank-grade ODS. It supports mature enterprise deployment models, high availability patterns, disaster recovery, partitioning, and operational tooling.

**Risks:**

- High licensing and operational cost.
- Complex operations.
- Risk of using Oracle-specific features too deeply, making future migration harder.

**HLD usage:**

```text
ODS: Oracle Database Enterprise Edition, deployed as an HA cluster with DR replication.
```

### Option B — Microsoft SQL Server Enterprise Edition

**Good fit when:**

- The bank already has a strong Microsoft platform team.
- SQL Server is already approved by enterprise architecture and security.
- The operations team is comfortable with SQL Server HA/DR, backup, patching, and monitoring.
- Reporting and operational analytics are expected to integrate with Microsoft tooling.

**Architecture fit:**

SQL Server is a credible enterprise ODS option for banks with Microsoft standards and existing DBA capacity.

**Risks:**

- Enterprise licensing cost.
- Less natural if the bank's engineering and operations stack is mostly Java, Linux, AWS, and IBM mainframe.
- Must avoid turning the ODS into a second system of record.

**HLD usage:**

```text
ODS: Microsoft SQL Server Enterprise Edition with enterprise HA/DR configuration.
```

### Option C — IBM Db2 LUW / Db2 pureScale

**Good fit when:**

- The bank wants technology alignment with its existing IBM ecosystem.
- Existing DB2/mainframe teams can support the operational model.
- The ODS receives data from DB2 and ADABAS through controlled replication/CDC pipelines.
- The bank values IBM vendor alignment and mainframe-adjacent support.

**Architecture fit:**

Db2 LUW is a strong candidate if the bank wants an IBM-aligned ODS. It can be positioned as a natural extension of the existing mainframe-adjacent data estate.

**Risks:**

- Smaller available talent pool compared with Oracle or SQL Server.
- May be less familiar to the 60 Java/AWS developers.
- Could increase dependency on IBM-specific operations.

**HLD usage:**

```text
ODS: IBM Db2 LUW, optionally using IBM clustering and HA/DR capabilities.
```

## Final HLD Recommendation

For MS1 and early HLD diagrams, use this neutral component label:

```text
Read Models / Operational Data Stores
Enterprise RDBMS ODS
```

For the written HLD, state the shortlist explicitly:

```text
The recommended ODS platform is an on-premises enterprise relational database.
Oracle Database Enterprise Edition is the preferred conservative banking option.
Microsoft SQL Server Enterprise Edition is acceptable if the bank is Microsoft-oriented.
IBM Db2 LUW is acceptable if the bank prefers IBM ecosystem alignment with the existing CBS estate.
The final selection will be validated during MS2 based on licensing, bank standards, DBA skillset, HA/DR design, and operational cost.
```

## Important Design Rule

The ODS is a read-optimized operational projection, not a transaction authority.

```text
CBS = legal system of record for balances and transfers
ODS = fast digital read models and operational projections
Cache = short-lived acceleration layer
Search = investigation and audit exploration layer
Data lake/object storage = historical analytics and immutable evidence
```

