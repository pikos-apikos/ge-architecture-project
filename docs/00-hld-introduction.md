# NeoBank Digital Leap — High Level Design

> Single-source HLD stand-in for the final Word submission. Section order
> follows the Lesson 1B HLD template (`../Software Architecture - Lesson 1B - Architecture_HLD_Template.docx.md`).
> Each top-level section either contains content or points to the dedicated
> file in `docs/`.

## 1. General

### 1.1 Introduction

#### Abstract

NeoBank Digital Leap is a high-level architecture design for the digital transformation of a traditional retail bank. The proposed system introduces new digital banking capabilities for mobile users, web users, and Open Banking third-party providers while preserving the existing Core Banking System as the authoritative system of record for financial transactions.

The MVP covers account information, balance display, bank transfers, real-time fraud detection, offline fraud analytics, monthly income and expense reporting, Open Banking APIs, and an AI-based financial advisory service. The architecture separates money-moving operations from read-oriented digital experiences: all financial transactions are executed through the existing CBS transaction flow, while scalable read models and digital services are introduced to improve customer experience, reduce load on legacy systems, and support faster delivery of new channels and services.

The design is driven by the project's key quality attributes and constraints: 99.999% uptime, no data loss, secure access, GDPR compliance, regional data processing, monitoring for both technical and business insights, support for 100,000 digital users within the first year, and scalability toward 1 million digital users within three years.

#### About this document

The purpose of this document is to provide a clear and concise understanding of the proposed system architecture before detailed design and implementation work begins. It describes the most important system components, their relationships, their responsibilities, and the way they interact with each other across on-premises, cloud, legacy, and external environments.

This document is intended to communicate the architecture to multiple stakeholder groups:

- **Business and product stakeholders**, who need to understand how the proposed architecture supports the digital transformation objectives, customer experience, regulatory expectations, and time-to-market goals.
- **Engineering teams**, who need a shared technical blueprint for services, APIs, data flows, integration points, deployment boundaries, and implementation responsibilities.
- **Security, compliance, and risk stakeholders**, who need to understand how customer data, financial transactions, auditability, fraud detection, and GDPR requirements are addressed.
- **Operations and infrastructure teams**, who need visibility into deployment topology, availability, monitoring, recovery, scalability, and operational ownership.

The document focuses on high-level design decisions and intentionally avoids unnecessary implementation detail. It defines the architecture boundaries, assumptions, functional and non-functional requirements, major components, high-level flows, data movement, persistence strategy, external dependencies, risks, limitations, and open issues. Detailed component-level design, source code structure, database schemas, and vendor-specific implementation details may be documented separately during later design phases.

### 1.2 Glossary

See [`docs/glossary.md`](glossary.md) for the authoritative terminology used throughout this HLD (CBS, BFF, ODS, TPP, RPO/RTO, ADABAS, etc.). New acronyms must be added to the glossary on first use; do not redefine them inline in body text.

## 2. Requirements

See [`docs/requirements/requirements-draft.md`](requirements/requirements-draft.md) for the authoritative functional requirements (`FN-NNN`) and non-functional requirements (`NFR-<CAT>-NNN`).

Existing NFR categories in this project: **AR** (Availability and Recovery), **DI** (Data Integrity and Consistency), **PC** (Performance and Capacity), **SEC** (Security), **BC** (Backward Compatibility), **GDPR**, **OBS** (Monitoring and Debugging), **DEP** (Deployment and Upgradability). Each requirement is testable and uses the wording "the system shall …".

> **Note (project working file):** architectural assumptions are tracked separately in [`docs/03-assumptions.md`](03-assumptions.md). The HLD template places assumptions within the Requirements section; the standalone assumptions file is a working convenience for the MVP and is consolidated into §2 during final HLD assembly.

## 3. High Level Design

### 3.1 High Level System Diagram

The MS1 high-level solution is the source of truth for component layout. Render the Mermaid source for the final Word submission; do not edit rendered images.

- Mermaid source: [`docs/diagrams/01-high-level-solution.mmd`](diagrams/01-high-level-solution.mmd)
- C4 views: [`docs/c4-model.md`](c4-model.md)

**Fixed ingress rule** (from the diagram and `01-ms1-preliminary-solution.md` §4): `Mobile App / Web App / Open Banking TPPs → WAF / DDoS Protection → API Gateway`. Channels must not bypass the WAF and must not point directly to the API Gateway.

### 3.2 Design Rules and Principles

Design rules are captured as Architecture Decision Records in [`docs/decisions/`](decisions/):

- [`ADR-001 — Hybrid Strangler Fig + CQRS`](decisions/ADR-001-hybrid-strangler-cqrs.md): CBS stays the system of record; new digital capabilities are introduced around it; ODS is a projection, never a second system of record.
- [`ADR-002 — Enterprise RDBMS ODS`](decisions/ADR-002-ods-enterprise-rdbms.md): On-prem ODS shortlist is Oracle / SQL Server / Db2; vendor deferred to MS2.

Operational design rules (no fluff):

- CBS is authoritative for balances and transfers. The CBS Transaction Gateway is the only digital component allowed to call CBS.
- All money-moving commands are idempotent and auditable, with correlation IDs spanning API Gateway → BFF → services → event bus → CBS gateway → audit log (NFR-OBS-040).
- AI Financial Advisor runs in the regional cloud and never accesses CBS, DB2, or ADABAS directly (FN-120).
- Offline Fraud Analytics is asynchronous / batch risk scoring, not a synchronous transfer-path service.

### 3.3 High Level System Flows

Component-level narrative and flow-level design decisions live in [`docs/01-ms1-preliminary-solution.md`](01-ms1-preliminary-solution.md) §5 (Main Components) and §6 (Key System Flows). Per-flow Mermaid sources:

- Account information flow — [`02-account-information-flow.mmd`](diagrams/02-account-information-flow.mmd)
- Bank transfer flow — [`03-bank-transfer-flow.mmd`](diagrams/03-bank-transfer-flow.mmd)
- Data ingestion and read-model flow — [`04-data-ingestion-read-model-flow.mmd`](diagrams/04-data-ingestion-read-model-flow.mmd)
- AI financial advisor flow — [`05-ai-financial-advisor-flow.mmd`](diagrams/05-ai-financial-advisor-flow.mmd)

### 3.4 Message Schemas

Status: **to be defined in MS2** (see `01-ms1-preliminary-solution.md` §12 "MS2 Work Items"). The template (§3.4) requires the inter-service APIs and event-bus message schemas to be managed inside the HLD to enable integrations.

Placeholder structure for the final HLD:

| API / Event | Producer | Consumer | Schema location | Versioning |
|---|---|---|---|---|
| _to be filled in MS2_ | | | | |

### 3.5 Upgradability

Status: **to be defined in MS2**. Required by the template; current coverage in [`docs/requirements/requirements-draft.md`](requirements/requirements-draft.md) §NFR-DEP-020 (rolling deployment) and §NFR-DEP-040 (API versioning).

Placeholder structure for the final HLD:

| Component | Upgrade pattern | Downtime | Rollback |
|---|---|---|---|
| _to be filled in MS2_ | | | |

### 3.6 Sizing

See [`docs/04-sizing.md`](04-sizing.md) for on-premises hardware and software sizing, including HA/DR topology, units of scale, and an operational cost estimate derived from the AWS / Azure pricing calculators (deck page 15, lesson requirements).

## 4. Time Estimation

See [`docs/05-time-estimation.md`](05-time-estimation.md) for the per-subsystem / per-team work estimate. Required by the template; the team is ~60 Java + AWS developers with 5 COBOL developers (deck page 15).

## 5. Limitations

See [`docs/06-limitations.md`](06-limitations.md) for explicit out-of-scope and architectural limitations. Document Scope below lists the MVP-level limitations; this section will expand to cover implementation / operational limitations in MS2.

## 6. Risks and Mitigations

See [`docs/07-risks-and-mitigations.md`](07-risks-and-mitigations.md) for the risk register with mitigations. MS1 seed content is in `01-ms1-preliminary-solution.md` §10 (Preliminary Risks).

## 7. Open Issues

See [`docs/08-open-issues.md`](08-open-issues.md) for open questions tracked into MS2. MS1 seed content is in `01-ms1-preliminary-solution.md` §11 (Open Questions for MS2).

## Document Scope

This HLD covers:

- Digital channels and public API access.
- Hybrid on-premises and regional cloud deployment boundaries.
- CBS integration and transaction execution boundaries.
- Read models and operational data stores.
- Fraud detection modes.
- Open Banking access and consent boundaries.
- AI financial advisor placement and data minimization rules.
- Observability, auditability, and operational concerns.

This HLD does not cover:

- Full low-level implementation design.
- Detailed database schema definitions.
- Source code structure.
- Final vendor procurement decisions.
- Complete cost model; only high-level cost drivers and sizing assumptions are included at this stage.
