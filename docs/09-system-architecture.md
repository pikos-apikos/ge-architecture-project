# 3.1 High Level System Architecture

## 3.1.1 Architecture summary

NeoBank Digital Leap uses a hybrid Strangler Fig architecture with CQRS-style read models. The existing CBS on z/OS remains the sole system of record for balances, bookings, and money movement. New digital services provide customer journeys, orchestration, fraud checks, reporting, Open Banking, and advisory capabilities without turning the ODS into a competing financial authority.

The architecture separates four concerns:

1. Public channels and identity are protected by the fixed ingress chain: WAF/DDoS, API Gateway, IAM, and the applicable BFF.
2. Regulated digital services run in the on-premises Digital Core and use managed contracts.
3. Financial commands reach the CBS only through the CBS Transaction Gateway.
4. Read-heavy journeys use a reconciled Enterprise RDBMS ODS populated by canonical CDC events.

The regional cloud hosts the public edge, elastic channel services, and the AI Financial Advisor. The AI Advisor receives only purpose-limited data through the Advisor Context Service. It has no direct network or application route to the ODS, DB2, ADABAS, or the CBS.

## 3.1.2 High-Level Solution View

Authoritative source: `docs/diagrams/01-high-level-solution.mmd`.

The High-Level Solution View shows the end-to-end placement of channels, ingress, cloud services, on-premises services, the event and projection path, and the CBS authority boundary. The direction of the arrows is normative. Public callers MUST NOT bypass the WAF or API Gateway. Digital services MUST NOT bypass the CBS Transaction Gateway.

## 3.1.3 C4 Container View

Authoritative source: `docs/diagrams/07-c4-container.mmd`.

| Container | Placement | Primary responsibility | Authoritative dependency |
|---|---|---|---|
| Mobile and Web Apps | Customer devices | Customer presentation and secure session initiation | Channel BFFs |
| WAF and API Gateway | Regional cloud edge | Threat filtering, quotas, routing, API policy | IAM and BFFs |
| IAM | Regional cloud / bank identity plane | Customer, workforce, workload, and TPP identity | Bank identity sources |
| Channel BFFs | Regional cloud private subnets | Channel composition and response shaping | Domain APIs |
| Consent Service | Regulated boundary | TPP, PSU, scope, account and SCA consent verification | Consent store and IAM |
| Account Information Service | On-premises Digital Core | Account summaries and bounded history with freshness | ODS plus recent-write overlay |
| Transfer Service | On-premises Digital Core | Transfer state machine, idempotency, fraud and CBS orchestration | Fraud Engine and CBS Gateway |
| Real-Time Fraud Engine | On-premises Digital Core | Binary APPROVE or DECLINE decision | Governed fraud policy/model |
| CBS Transaction Gateway | Mainframe integration zone | Canonical anti-corruption boundary | CBS |
| CDC and Canonical Mapper | Mainframe integration zone | Canonical ACCOUNT, BALANCE and TRANSACTION changes | DB2/ADABAS logs |
| Event Bus | On-premises data platform | Durable at-least-once event delivery | Producer outboxes |
| ODS Projection Service | On-premises data platform | Account-scoped ordered projection and quarantine | Event Bus |
| Enterprise RDBMS ODS | On-premises data platform | Non-authoritative read projection | CBS-derived events |
| Reporting Service | On-premises Digital Core | Immutable monthly statements and report retrieval | Reconciled ODS |
| Advisor Context Service | On-premises Digital Core | Curated, minimized, eligible advisory context | Governed projections and bank rules |
| AI Financial Advisor | Regional cloud private subnets | Rank and explain eligible product candidates | Advisor Context Service only |
| Observability and Evidence | Segregated management plane | Telemetry, audit receipts, release and incident evidence | All approved journeys |

## 3.1.4 Network Topology View

Authoritative source: `docs/diagrams/06-network-topology.mmd`. This teacher-mandated view defines both hardware and software placement.

| Zone | Example address space | Required controls and protocols |
|---|---|---|
| Cloud public edge | `10.20.0.0/24` | Managed DDoS, WAF, API Gateway; inbound TLS 1.2+ on 443 only |
| Cloud private application | `10.20.16.0/20` | IAM, BFFs, AI Advisor; no public IPs; workload mTLS |
| Transit | Dedicated bank VRF | Two Direct Connect circuits in separate facilities, encrypted BGP sessions, site-to-site VPN backup |
| On-premises DMZ | `10.40.0.0/24` | HA NGFW pair and reverse-proxy ingress; explicit allowlist from cloud private subnets |
| Digital Core | `10.40.16.0/20` | Kubernetes/OpenShift worker and control nodes; mTLS east-west; namespace and network policy |
| Data platform | `10.40.32.0/20` | ODS, Event Bus, observability; database/broker ports only from approved workloads |
| Mainframe integration | `10.40.48.0/24` | CBS Gateway HA pair; dedicated workload identity; no direct channel route |
| Management and evidence | `10.40.64.0/20` | PAM, deployment agents, monitoring collectors and immutable evidence endpoints |

Exact CIDRs, ports other than public TLS 443, firewall objects, route tables, Direct Connect bandwidth, and vendor products remain Phase-0 validation items. Detailed design MUST preserve the zone relationships and prohibited routes shown here.

## 3.1.5 Security Trust-Boundary View

Authoritative source: `docs/diagrams/12-security-trust-boundaries.mmd`.

Eight trust zones are used: public/untrusted, edge security, channel and identity, regulated Digital Core, data and event, mainframe authority, regional AI cloud, and management/evidence. Every crossing MUST have an authenticated workload identity, encrypted transport, explicit authorization, schema validation, rate or capacity protection, correlation, and security/audit evidence appropriate to the data classification.

## 3.1.6 Resilience and DR Topology

Authoritative source: `docs/diagrams/13-resilience-dr-topology.mmd`.

High availability is provided inside each active failure domain. Disaster recovery uses a paired cloud region, paired on-premises site, and the bank's governed CBS recovery capability. Promotion of financial authority and opening customer traffic require two independent authorities and evidence that identity, keys, data integrity, connectivity, observability, and reconciliation are ready.

## 3.1.7 Observability and Evidence View

Authoritative source: `docs/diagrams/14-observability-evidence.mmd`.

The observability model follows eight end-to-end journeys rather than isolated services. Operational telemetry, security telemetry, and audit evidence are separate planes connected by correlation identifiers and authoritative outcome references. Dashboards, error-budget burn alerts, incident evidence bundles, and executable runbooks consume this correlated evidence.

## 3.1.8 Architectural authority matrix

| Information or action | Authority | Projection or consumer |
|---|---|---|
| Account identity and status | CBS / approved customer master source | ODS account projection |
| Current and available balance | CBS | ODS plus CBS-confirmed recent-write overlay |
| Transfer commit or rejection | CBS through CBS Gateway | Transfer Service and TransferOutcomeEvent |
| Transfer workflow state | Transfer Service | Channels, reporting and audit |
| Fraud decision | Real-Time Fraud Engine | Transfer Service and audit |
| Consent status and scope | Consent Service | BFFs and domain services through verified ConsentContext |
| Product eligibility | Governed bank rules | Advisor Context and AI Advisor |
| AI ranking and explanation | AI Advisor | Customer advisory response; never a binding sale |
| Projection freshness and quarantine | ODS Projection Service | Account APIs, operations and audit |
| Monthly statement snapshot | Reporting Service after reconciliation | Customer reporting API |
| Privileged and release evidence | Evidence plane | Audit, security, operations and compliance |

## 3.1.9 Explicitly prohibited architecture paths

- A mobile app, web app, or TPP MUST NOT call the API Gateway without passing through the WAF/DDoS boundary.
- A TPP MUST NOT call the Consent Service or a domain service directly.
- A digital service MUST NOT call the CBS, DB2, or ADABAS except through the approved gateway or CDC boundary.
- The ODS MUST NOT authorize a balance-changing transaction.
- The AI Advisor MUST NOT query the ODS or legacy stores directly.
- Offline fraud analytics MUST NOT sit on the synchronous transfer path.
- Administrative users MUST NOT obtain standing production privilege or bypass PAM.
- A recovery site MUST NOT accept customer traffic before the approved promotion evidence is complete.
