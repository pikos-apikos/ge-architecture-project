# NeoBank Digital Leap — C4 Model Layers

## Purpose

This document describes how the C4 model can be applied to the NeoBank Digital Leap architecture. The goal is to keep the architecture understandable at different levels of detail, starting from business context and moving down to deployable systems, internal components, and selected code-level design decisions.

For this project, the C4 model should be used as a communication tool, not as a mandatory documentation burden. The High Level Design should mainly include Level 1 and Level 2 diagrams. Level 3 diagrams should be added only for critical or complex containers. Level 4 should be used only when a specific implementation detail is important enough to affect architecture.

---

## Level 1 — System Context Diagram

The System Context diagram shows NeoBank Digital Platform as a single system and explains how it interacts with users, external actors, legacy systems, and regulatory or operational dependencies.

At this level, the audience is mainly business stakeholders, product owners, security stakeholders, operations, and senior engineering leadership. The diagram should answer the question: **who uses the system, what external systems does it depend on, and what is the main boundary of responsibility?**

For NeoBank, the system of interest is the new Digital Banking Platform. It provides mobile banking, web banking, Open Banking APIs, fraud-protected transfers, account information, monthly financial reports, and an AI financial advisor. It does not replace the existing Core Banking System in the MVP. Instead, it surrounds and integrates with the legacy CBS, which remains the source of truth for financial transactions.

Typical external actors and systems:

- Retail banking customers using the mobile app or web banking.
- Third Party Providers using the Open Banking API.
- Bank operations and support teams monitoring the digital platform.
- The legacy Core Banking System running on mainframe technology.
- Existing banking data stores such as DB2 and ADABAS.
- External payment networks or interbank transfer systems.
- Identity provider and authentication infrastructure.
- Regulatory and audit stakeholders.

Recommended NeoBank Level 1 diagram title:

```text
C4 Level 1 — NeoBank Digital Platform System Context
```

Recommended message:

```text
The Digital Banking Platform provides modern digital channels while keeping the legacy CBS as the transaction source of truth. It exposes secure customer and Open Banking APIs, integrates with the CBS for money movement, and uses replicated operational data stores for scalable read access.
```

---

## Level 2 — Container Diagram

The Container diagram decomposes the NeoBank Digital Platform into major deployable or runtime units. A container in C4 does not necessarily mean a Docker container. It can be a web application, API gateway, backend service, database, message broker, batch processor, or external runtime component.

At this level, the audience is architects, senior developers, infrastructure engineers, security engineers, and DevOps teams. The diagram should answer the question: **what are the major building blocks, where do they run, and how do they communicate?**

For NeoBank, this is the most important MS1 diagram. It should clearly show the split between cloud-facing digital edge, on-premises banking integration, operational read stores, and legacy CBS dependencies.

Recommended containers:

- Mobile App.
- Web Banking Application.
- Third Party Provider clients for Open Banking.
- WAF / DDoS Protection.
- API Gateway.
- Digital BFF / Channel API.
- Identity and Access Management integration.
- Consent and Open Banking Service.
- Account Information Service.
- Transfer Service.
- Real-Time Fraud Engine.
- Offline Fraud Analytics / Batch Risk Scoring.
- AI Financial Advisor Service.
- CBS Transaction Gateway.
- Legacy CBS / Mainframe integration adapters.
- Event Bus / Streaming Platform.
- Data Ingestion / CDC / Replication pipeline.
- Read Models / Operational Data Store.
- Cache layer.
- Audit and immutable evidence store.
- Observability platform.
- Disaster Recovery environment.

The important architectural rule at this level is that customer-facing traffic must enter through the WAF before reaching the API Gateway:

```text
Mobile App / Web Banking / TPP Clients
        -> WAF / DDoS Protection
        -> API Gateway
        -> Digital Platform Services
```

Recommended NeoBank Level 2 diagram title:

```text
C4 Level 2 — NeoBank Digital Platform Containers
```

Recommended message:

```text
The container architecture separates digital channel access, transactional command processing, read-optimized data access, fraud detection, AI advisory capabilities, and legacy CBS integration. Money-moving operations are routed through the CBS Transaction Gateway, while read-heavy use cases are served from operational read models.
```

---

## Level 3 — Component Diagram

The Component diagram opens one selected container and describes its internal responsibilities. This level should not be created for every service. It should be used only where the internal design is architecturally important, risky, or likely to be discussed in the HLD review.

At this level, the audience is mainly developers, tech leads, architects, QA engineers, and security reviewers. The diagram should answer the question: **what are the internal components of this container and how do they collaborate?**

For NeoBank, the best candidates for Level 3 diagrams are:

1. Transfer Service.
2. CBS Transaction Gateway.
3. Data Ingestion / Projection Builder.
4. Real-Time Fraud Engine.
5. Consent and Open Banking Service.
6. AI Financial Advisor Service.

### Example: Transfer Service components

A Level 3 diagram for the Transfer Service could include:

- Transfer API Controller.
- Request Validation component.
- Authentication and Authorization Guard.
- Idempotency Manager.
- Real-Time Fraud Client.
- CBS Transaction Client.
- Transfer Orchestrator / Application Service.
- Audit Event Publisher.
- Transfer Status Repository.
- Error and Reconciliation Handler.

Recommended message:

```text
The Transfer Service is responsible for validating transfer requests, enforcing idempotency, invoking real-time fraud checks, executing the transaction through the CBS Transaction Gateway, publishing audit events, and exposing the final transfer status to the customer channel.
```

### Example: CBS Transaction Gateway components

A Level 3 diagram for the CBS Transaction Gateway could include:

- CBS Command API.
- Protocol Adapter for mainframe communication.
- Transaction Mapper.
- Retry and Timeout Policy Manager.
- CBS Response Classifier.
- Reconciliation Queue Publisher.
- Audit Logger.
- Operational Metrics Exporter.

Recommended message:

```text
The CBS Transaction Gateway isolates the digital platform from mainframe-specific protocols, data formats, error semantics, and operational constraints. It provides a controlled and auditable boundary for all CBS transactions.
```

---

## Level 4 — Code Diagram

The Code diagram shows implementation-level details such as classes, interfaces, modules, packages, or important functions. This level is usually not required in a High Level Design document unless a specific implementation decision is central to the architecture.

At this level, the audience is developers and reviewers working directly on implementation. The diagram should answer the question: **how will this component be implemented in code?**

For NeoBank MS1, Level 4 should not be included. For MS2 or the final HLD, Level 4 may be useful only for selected critical paths, such as transfer orchestration, CBS gateway isolation, idempotency, or audit publishing.

A possible Level 4 example for the Transfer Service:

```text
TransferController
TransferApplicationService
TransferRequestValidator
IdempotencyService
FraudCheckPort
CbsTransactionPort
TransferStatusRepository
AuditEventPublisher
TransferErrorMapper
```

Recommended implementation style:

- Hexagonal architecture / ports and adapters for CBS and fraud integrations.
- Explicit application services for business orchestration.
- Domain events for audit and downstream processing.
- Idempotency keys for transfer commands.
- Clear separation between command path and read model updates.

Recommended message:

```text
Code-level diagrams should be limited to architecture-sensitive implementation areas. The purpose is not to document every class, but to make critical design decisions visible and reviewable before implementation.
```

---

## Recommended Scope for the Course Deliverables

### MS1 — Preliminary Solution Diagram

Include:

- C4 Level 1 System Context.
- C4 Level 2 Container diagram.
- Short narrative explaining the main architectural decisions.

Do not include:

- Detailed component diagrams for every service.
- Code-level diagrams.
- Full database schema.

### MS2 — HLD + Data + Security + Performance Diagrams

Include:

- Updated C4 Level 2 Container diagram.
- Selected C4 Level 3 Component diagrams for risky or critical containers.
- Data flow diagrams.
- Security flow diagrams.
- Performance and scalability view.

### Final HLD

Include:

- Stable Level 1 and Level 2 diagrams.
- Level 3 diagrams only where they clarify important design choices.
- Level 4 appendix only if it helps explain a critical implementation constraint.

---

## Summary

For this project, the C4 model should be applied pragmatically:

- **Level 1** explains the business and external system context.
- **Level 2** explains the main deployable containers and runtime architecture.
- **Level 3** explains the internals of selected critical services.
- **Level 4** explains code-level design only when it protects an important architectural decision.

The most important diagrams for NeoBank are Level 1 and Level 2. They show that the new digital platform improves customer experience and scalability while keeping the legacy CBS as the authoritative transaction system.

