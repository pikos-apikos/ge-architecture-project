# MS1 — Preliminary Solution Diagram

**Project:** NeoBank Digital Leap  
**Artifact:** Milestone 1 — Preliminary Solution Diagram  
**Author:** Yiannis Miliaresis  
**Status:** Draft v0.1  
**Purpose:** Provide the first high-level architecture direction for NeoBank's digital transformation MVP.

---

## 1. Executive Summary

NeoBank will launch a new digital banking platform while keeping the existing Core Banking System as the authoritative system of record for money movement and ledger consistency.

The proposed architecture follows a **Strangler Fig modernization approach**: new digital capabilities are introduced through a new integration and digital services layer, while the existing COBOL/z/OS CBS continues to execute all financial transactions.

The MVP introduces:

- A secure digital edge for mobile app, web app, and Open Banking APIs.
- A new on-premises digital core for regulated and latency-sensitive services.
- A regional cloud environment for scalable digital channels and the AI financial advisor.
- A CBS Transaction Gateway that protects the legacy CBS from direct digital-channel access.
- Read-optimized data stores for fast account, customer, report, and dashboard queries.
- Real-time fraud detection for synchronous transfer decisions.
- Offline Fraud Analytics / Batch Risk Scoring for historical analysis, model/rule tuning, and investigation.
- Event-driven data ingestion and replication between legacy systems, on-premises services, and cloud services.
- Centralized observability, audit, and security controls.

The design prioritizes **availability, no data loss, security, scalability, maintainability, and fast delivery within one year**.

---

## 2. Architecture Drivers

### 2.1 Business Drivers

| Driver | Architectural Implication |
|---|---|
| Deliver remarkable digital experiences | Use API-first services, low-latency read models, and scalable digital channels. |
| Support fast time-to-market | Use modular services, CI/CD, cloud-native deployment, and clear service boundaries. |
| Survive competitive pressure from fintech and card-company competitors | Build an extensible digital platform instead of isolated applications. |
| Reduce mainframe operational cost | Avoid unnecessary CBS calls by serving read-heavy use cases from replicated read models. |
| Support future growth to 1M digital users | Design for horizontal scalability and regional cloud/on-prem deployment. |

### 2.2 Technical Drivers

| Driver | Architectural Implication |
|---|---|
| All transactions must be executed by CBS | Use a CBS Transaction Gateway and do not bypass the core ledger. |
| 99.999% uptime | Design critical layers for redundancy, health checks, failover, and disaster recovery. |
| No data loss | Use durable messaging, idempotency, reconciliation, backups, and audit trails. |
| Account data may be stale up to 24h for external incoming/non-app transactions | Use read models with explicit freshness metadata and customer-facing freshness indicators. |
| 100K digital users in year 1, 1M in year 3 | Start with scalable foundations: API Gateway, container platform, autoscaling, caching, and partitioned data stores. |
| Java and AWS team experience | Prefer Java/Spring-based services and AWS-managed services where legally allowed. |
| Only 5 COBOL developers | Minimize CBS changes and isolate legacy integration behind adapters. |
| GDPR and regional data requirements | Keep data in-region, minimize personal data replication, support deletion/anonymization workflows where legally possible. |

---

## 3. Proposed Architecture Style

The recommended architecture style is a hybrid of:

1. **Strangler Fig Modernization**  
   New digital capabilities are built around the legacy CBS without replacing it during the MVP.

2. **API-First Digital Platform**  
   Mobile, web, and Open Banking clients consume stable APIs through WAF/DDoS protection, an API Gateway, and BFF/API services.

3. **CQRS for Banking Workloads**  
   Commands that move money go to CBS. Queries use read-optimized stores populated from data ingestion and events.

4. **Event-Driven Integration**  
   Data changes and domain events are propagated through durable messaging and streaming infrastructure.

5. **Hybrid On-Premises + Regional Cloud Deployment**  
   Regulated, CBS-adjacent, and legally restricted workloads run on-premises. Elastic customer-facing and AI workloads run in the regional cloud when allowed.

---

## 4. Preliminary High-Level Solution Diagram

The diagram source is maintained in:

```text
docs/diagrams/01-high-level-solution.mmd
```

Important ingress rule:

```text
Mobile App / Web App / Open Banking TPPs -> WAF / DDoS Protection -> API Gateway
```

The channels must not bypass the WAF and must not point directly to the API Gateway.

---

## 5. Main Components

### 5.1 Digital Channels

| Component | Responsibility |
|---|---|
| Mobile App | Customer-facing digital banking channel. |
| Web App | Browser-based digital banking channel. |
| Open Banking TPP Clients | Third-party providers consuming Open Banking APIs. |

### 5.2 Cloud Digital Edge

| Component | Responsibility |
|---|---|
| WAF / DDoS Protection | Protect public endpoints from common web attacks and denial-of-service attempts. |
| API Gateway | Public API entry point, routing, throttling, authentication enforcement, request validation, and API versioning. |
| Customer IAM | Authentication, authorization, MFA, OAuth2/OIDC, token issuance, and customer session management. |
| Digital API / BFF Layer | Client-oriented APIs for mobile/web. Aggregates data from on-premises services and cloud services. |
| AI Financial Advisor | Cloud-only service that provides financial advice based on authorized customer data. |
| Cloud Observability | Cloud metrics, logs, traces, dashboards, and alerts. |

### 5.3 On-Premises Digital Core

| Component | Responsibility |
|---|---|
| Account Information Service | Serves account and balance information from read models. |
| Transfer Service | Orchestrates bank transfers and ensures every transaction is executed through CBS. |
| Real-Time Fraud Engine | Evaluates transfers before execution. Cancels suspicious transactions and informs the customer. |
| Offline Fraud Analytics / Batch Risk Scoring | Performs deeper fraud analysis using larger historical datasets and longer processing windows. Produces risk findings, rules, and model feedback for the real-time fraud engine. |
| Income / Expense Report Service | Generates monthly income/expense reports from categorized transaction data. |
| Open Banking Consent Service | Manages customer consent and access boundaries for third-party providers. |
| Advisor Context Service | Data-minimization boundary for the cloud AI Financial Advisor (D-008). Checks consent scope with the Consent Service, reads only authorized projections from the ODS, and records data-minimization decisions in the audit log. The AI advisor never reads the ODS directly. |
| CBS Transaction Gateway | Single controlled interface for executing CBS transactions. Handles idempotency, validation, throttling, and audit. |
| Db2 Adapter | Ingests transactional data from Db2 into the event bus/read models. |
| ADABAS Adapter | Ingests customer data from ADABAS into the event bus/read models. |
| Durable Event Bus / Streaming | Reliable event backbone for data propagation, read model updates, audit, and analytics. |
| Read Models / Operational Data Stores | Query-optimized data stores for digital services. Implemented on an enterprise RDBMS ODS candidate platform: Oracle, SQL Server, or Db2. |
| Audit Log / Compliance Store | Immutable or append-only audit trail for security, financial, and regulatory traceability. |
| On-Prem Observability | Metrics, logs, traces, technical dashboards, business dashboards, and operational alerts. |

### 5.4 Existing Legacy Core

| Component | Responsibility |
|---|---|
| COBOL CBS on z/OS / IBM iSeries | Authoritative execution of financial transactions. |
| Db2 Transactions DB | System of record for transactions and ledger-related data. |
| ADABAS Customer DB | System of record for customer information. |

---

## 6. Key System Flows

### 6.1 Account Information Flow

The customer opens the mobile or web app and requests account information. The request enters through WAF/DDoS protection and the API Gateway, is authenticated by Customer IAM, and reaches the Digital API/BFF layer. The BFF calls the on-premises Account Information Service, which serves the response from read models and returns freshness metadata.

**Design decision:** Account queries should not call CBS directly unless a forced refresh is required. The read model must expose freshness metadata.

### 6.2 Bank Transfer Flow

The customer submits a bank transfer request. The request enters through WAF and API Gateway, then reaches the BFF and Transfer Service. The Transfer Service validates the request, calls the Real-Time Fraud Engine, and if approved, executes the transaction through the CBS Transaction Gateway. The gateway calls the CBS, records audit events, and returns the result. If fraud is detected, the transaction is cancelled and the customer is informed in the application.

**Design decision:** All money-moving transactions must go through CBS. The CBS Transaction Gateway is the only digital component allowed to execute CBS transactions.

### 6.3 Data Ingestion and Read Model Flow

Db2 and ADABAS adapters ingest legacy data changes into the durable event bus. Projection workers update the read models and operational data stores. Downstream services use the read models for account information, customer profile data, reports, Open Banking reads, and AI advisor context generation.

**Design decision:** Data ingestion should be durable, replayable where possible, and monitored for lag. Read models must be rebuilt from source data or event history if corruption occurs.

### 6.4 AI Financial Advisor Flow

The AI advisor runs in the regional cloud. It does not directly access CBS, Db2, ADABAS, or the ODS. It requests minimized customer context from the on-premises Advisor Context Service, which checks consent scope with the Consent Service, reads only authorized projections from the ODS, and returns an AI-safe, regionally compliant context.

**Design decision:** The AI advisor may run in the cloud, but it only receives minimized, authorized, regionally compliant customer data through the Advisor Context Service (D-008). The Advisor Context Service — not the AI service — writes the data-minimization audit record.

---

## 7. Preliminary Data Ownership Model

| Data / Operation | System of Record | Serving Model |
|---|---|---|
| Ledger transaction execution | CBS / Db2 | CBS Transaction Gateway |
| Account balance display | Db2 via replicated read model | Account Information Service |
| Customer profile | ADABAS via replicated read model | Customer Read Model |
| Bank transfer command | CBS | Transfer Service + CBS Transaction Gateway |
| Fraud real-time decision | Fraud Engine | Transfer Service |
| Fraud offline analysis | Fraud Analytics Dataset | Offline Fraud Analytics / Batch Risk Scoring |
| Monthly income/expense report | Derived from transaction data | Report Read Model |
| AI advice context | Minimized authorized customer view | Advisor Context Service |
| Consent | Consent Service | Consent Store |
| Audit trail | Audit / Compliance Store | Append-only audit queries |

---

## 8. MVP Scope

### In Scope for MVP

- Mobile and web API access through WAF and API Gateway.
- Customer authentication and authorization.
- Account and balance display.
- Bank transfers to same-bank and other-bank accounts through CBS.
- Real-time fraud detection for transfer flow.
- Offline fraud analytics for investigation, model/rule improvement, and batch risk scoring.
- Monthly income and expense reports.
- Open Banking REST API and consent management.
- AI financial advisor running in the regional cloud.
- Legacy data ingestion from Db2 and ADABAS.
- On-premises ODS/read models.
- Audit, monitoring, logging, and alerting.

### Out of Scope for MVP

- Full CBS replacement.
- Full data lake platform for all enterprise analytics.
- Multi-region active-active banking across different legal regions.
- Branch modernization.
- Internal employee portal redesign.
- Advanced AI autonomous decision-making for regulated financial actions.

---

## 9. Key Assumptions

| ID | Assumption |
|---|---|
| A-001 | CBS remains the system of record for financial transactions during MVP. |
| A-002 | The bank can deploy a new on-premises digital platform close to CBS. |
| A-003 | The cloud region is legally acceptable for selected digital services and AI advisor workloads. |
| A-004 | Db2 and ADABAS data can be ingested through adapters, CDC, batch replication, or controlled export mechanisms. |
| A-005 | Some account information can be stale up to 24 hours for incoming transactions from other banks or payments not made through the app. |
| A-006 | The bank already has enterprise database operational standards for Oracle, SQL Server, or Db2. |
| A-007 | The MVP can avoid major COBOL changes by introducing a CBS Transaction Gateway and adapter layer. |

---

## 10. Preliminary Risks

| Risk | Impact | Mitigation |
|---|---|---|
| CBS integration bottleneck | MVP delay and runtime instability | Isolate CBS calls through CBS Transaction Gateway, use idempotency, throttling, and contract testing. |
| Data freshness confusion | Customer trust issues | Display freshness metadata and clearly define stale-data rules. |
| Fraud false positives | Customer friction and support cost | Use rule tuning, offline analytics, explainability, and manual review paths. |
| Cloud/on-prem latency | Poor UX | Use BFF aggregation, caching, regional deployment, and colocated on-prem services for critical flows. |
| Regulatory constraints on AI advisor | Compliance risk | Use data minimization, consent, regional processing, audit trails, and clear advice disclaimers. |
| ODS becomes a second system of record | Financial correctness risk | Enforce design rule: CBS is authoritative for money movement; ODS is read-only projection. |

---

## 11. Open Questions for MS2

| ID | Question | Owner / Area |
|---|---|---|
| OQ-001 | Which cloud provider and region are approved by the bank and regulator? | Infrastructure / Compliance |
| OQ-002 | Which enterprise RDBMS is already approved for on-premises ODS workloads? | DBA / Enterprise Architecture |
| OQ-003 | What are the existing CBS integration mechanisms and throughput limits? | Mainframe Team |
| OQ-004 | What is the allowed maximum latency for same-bank and external bank transfers? | Product / Risk |
| OQ-005 | What Open Banking standard version must be supported? | Product / Compliance |
| OQ-006 | What customer data may be used by the AI financial advisor? | Legal / Data Protection |
| OQ-007 | What is the required RTO/RPO for each major component? | Operations / Architecture |

---

## 12. MS2 Work Items

- Convert preliminary diagrams into final HLD diagrams.
- Add data flow diagrams.
- Add security boundary diagrams.
- Add performance and scaling diagrams.
- Define ODS vendor decision criteria.
- Define message schemas and API contracts at high level.
- Define sizing assumptions and operational cost model.
- Expand risks and mitigations.
