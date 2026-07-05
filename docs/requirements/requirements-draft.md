# NeoBank HLD — Requirements Draft

## Functional Requirements

| ID | Requirement |
|---|---|
| FN-010 | The system shall allow authenticated customers to view account information and balances through the mobile app and web app. |
| FN-020 | The system shall allow authenticated customers to perform bank transfers to another account in the same bank. |
| FN-030 | The system shall allow authenticated customers to perform bank transfers to another bank. |
| FN-040 | The system shall execute all money-moving transactions through a CBS transaction. |
| FN-050 | The system shall evaluate transfer requests using a real-time fraud detection engine before transaction execution. |
| FN-060 | The system shall cancel a transaction when fraud is detected and inform the customer in the application. |
| FN-070 | The system shall support offline fraud analytics using larger historical datasets and longer processing windows. |
| FN-080 | The system shall show a monthly income and expenses report to the customer. |
| FN-090 | The system shall expose Open Banking APIs over REST. |
| FN-100 | The system shall manage customer consent for Open Banking third-party providers. |
| FN-110 | The system shall provide an AI financial advisor based on authorized customer data. |
| FN-120 | The system shall prevent the AI financial advisor from directly accessing CBS, DB2, or ADABAS. |
| FN-130 | The system shall ingest transaction data from DB2 into digital read models. |
| FN-140 | The system shall ingest customer information from ADABAS into digital read models. |
| FN-150 | The system shall provide audit records for customer actions, transaction decisions, fraud decisions, consent changes, and administrative operations. |

## Non-Functional Requirements

### Availability and Recovery

| ID | Requirement |
|---|---|
| NFR-AR-010 | The system shall be designed for 99.999% uptime for critical digital banking capabilities. |
| NFR-AR-020 | The system shall define RTO and RPO targets per major component before MS2. |
| NFR-AR-030 | The system shall support disaster recovery for the on-premises Digital Core and the regional cloud environment. |
| NFR-AR-040 | The system shall support graceful degradation for non-critical services such as AI advice and monthly reports. |

### Data Integrity and Consistency

| ID | Requirement |
|---|---|
| NFR-DI-010 | The system shall prevent data loss for transaction commands, audit events, and integration events. |
| NFR-DI-020 | The system shall treat CBS as the authoritative system of record for balances and transfers. |
| NFR-DI-030 | The system shall treat ODS/read models as read-optimized projections, not as transaction authorities. |
| NFR-DI-040 | The system shall support idempotency for transfer requests and CBS Transaction Gateway calls. |
| NFR-DI-050 | The system shall support reconciliation between CBS records, integration events, read models, and audit records. |

### Performance and Capacity

| ID | Requirement |
|---|---|
| NFR-PC-010 | The system shall support 100,000 digital users within one year. |
| NFR-PC-020 | The system shall support architectural scalability toward 1,000,000 digital users within three years. |
| NFR-PC-030 | The system shall provide low-latency responses for read-heavy digital use cases through read models and caching. |
| NFR-PC-040 | The system shall protect the CBS from direct high-volume digital read traffic. |
| NFR-PC-050 | The system shall monitor event ingestion lag and read model freshness. |
| NFR-PC-060 | The system shall display account information that may be up to 24 hours stale for incoming transactions from other banks or payments not made through the app, with visible freshness metadata. |

### Security

| ID | Requirement |
|---|---|
| NFR-SEC-010 | The system shall use secure customer authentication and authorization. |
| NFR-SEC-020 | The system shall support MFA for customer authentication where required. |
| NFR-SEC-030 | Public traffic shall pass through WAF/DDoS protection before reaching the API Gateway. |
| NFR-SEC-040 | The system shall enforce customer consent boundaries for Open Banking access. |
| NFR-SEC-050 | The system shall restrict AI advisor access to minimized, authorized, regionally compliant customer data. |
| NFR-SEC-060 | The system shall encrypt sensitive data in transit and at rest. |

### Backward Compatibility

| ID | Requirement |
|---|---|
| NFR-BC-010 | The system shall evolve public, internal, and event-bus message schemas in a backward-compatible manner (additive only within a major version). |
| NFR-BC-020 | The system shall support multiple supported API versions concurrently during deprecation windows. |
| NFR-BC-030 | The system shall avoid breaking CBS transaction contracts; any change to the CBS Transaction Gateway contract shall be versioned and coordinated with the mainframe team. |

### GDPR and Compliance

| ID | Requirement |
|---|---|
| NFR-GDPR-010 | The system shall process customer data in the approved geographic region. |
| NFR-GDPR-020 | The system shall support right-to-delete or anonymization workflows where legally permitted. |
| NFR-GDPR-030 | The system shall retain mandatory financial and audit records according to regulatory requirements. |
| NFR-GDPR-040 | The system shall document the difference between deletable personal data and legally retained financial records. |

### Monitoring and Debugging

| ID | Requirement |
|---|---|
| NFR-OBS-010 | The system shall provide technical metrics, logs, traces, dashboards, and alerts. |
| NFR-OBS-020 | The system shall provide business metrics for transactions, fraud decisions, Open Banking usage, and digital adoption. |
| NFR-OBS-030 | The system shall provide operational visibility across cloud, on-premises, and legacy integration boundaries. |
| NFR-OBS-040 | The system shall support correlation IDs across API Gateway, BFF, services, event bus, CBS gateway, and audit logs. |

### Deployment and Upgradability

| ID | Requirement |
|---|---|
| NFR-DEP-010 | The system shall separate on-premises regulated workloads from cloud-allowed workloads. |
| NFR-DEP-020 | The system shall support rolling deployment for new digital services where possible. |
| NFR-DEP-030 | The system shall avoid requiring major CBS changes for the MVP. |
| NFR-DEP-040 | The system shall support API versioning for external and internal APIs. |
