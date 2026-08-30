# NeoBank Digital Leap — Final High Level Design

## 1. General

### 1.1 Introduction

NeoBank Digital Leap defines the Year-1 MVP architecture for a publicly traded retail bank serving one million customers through an existing COBOL/z/OS Core Banking System. The program introduces mobile, web, Open Banking, fraud, reporting, and AI-assisted advisory capabilities while preserving the CBS as the sole financial authority.

The design uses a hybrid Strangler Fig architecture. Money movement remains strongly controlled through the CBS Transaction Gateway. Scalable read journeys use a reconciled on-premises Enterprise RDBMS ODS populated by canonical CDC events. The regional cloud provides the protected public edge, elastic channel services, and the AI Financial Advisor. Regulatory, identity, data, network, and evidence boundaries remain explicit across cloud, on-premises, and mainframe zones.

The MVP covers account information and balances, internal and external transfers, real-time and offline fraud modes, reconciled monthly statements, Open Banking AIS/PIS, and an AI financial advisor that can recommend eligible bank products. Bank rules decide eligibility; the AI ranks and explains. The AI has no direct access to the CBS, Db2, ADABAS, or ODS.

The principal quality targets are 99.999% availability for the customer-critical aggregate, no loss of accepted financial commands or mandatory evidence, immediate read-your-writes for completed digital transfers, up to 24 hours of visible staleness only for the external activity allowed by the project brief, support for 100,000 users in Year 1 and one million in Year 3, regional GDPR processing, and measurable technical and business operations.

### 1.1.1 Purpose and audience

This HLD is the shared architecture contract for business sponsors, product, engineering, mainframe, security, privacy, compliance, operations, SRE, data, procurement, FinOps, and delivery leadership. It describes system boundaries, major components, flows, managed contracts, trust zones, recovery, observability, sizing, cost, delivery effort, limitations, risks, and unresolved Phase-0 validations.

The document intentionally remains at high level. It does not define complete product configurations, physical database schemas, low-level component internals, source code, full OpenAPI/AsyncAPI specifications, packet-level firewall rules, or purchasing authority.

### 1.1.2 Business outcomes

- Provide remarkable, immediate digital experiences without exposing legacy platforms directly.
- Introduce regulated digital services within one year and create a path to one million users.
- Reduce mainframe read cost and change pressure through projections and anti-corruption boundaries.
- Make money movement, consent, fraud, privileged operations, and release decisions auditable.
- Support Open Banking competition and personalized product recommendations under bank governance.
- Create a measurable delivery model that includes the bootstrap and inference cost of agentic engineering.

### 1.1.3 Document structure

The document follows the Lesson 1B template:

1. General
2. Requirements, including assumptions
3. High Level Design
4. Time Estimation
5. Limitations
6. Risks and Mitigations
7. Open Issues

Exactly four appendices provide traceability, the managed contract catalog, sizing/TCO evidence, and delivery-estimation evidence. Exactly eighteen embedded diagrams are generated from the canonical Mermaid sources under `docs/diagrams`.

### 1.2 Glossary

The authoritative glossary is `docs/glossary.md`. Terms are used consistently throughout the HLD.

