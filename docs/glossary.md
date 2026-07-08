# NeoBank HLD — Glossary

| Term | Meaning |
|---|---|
| ADABAS | Legacy database used for customer information in the current NeoBank estate. |
| Advisor Context API | On-premises data-minimization boundary between the cloud AI Financial Advisor and the ODS. Checks consent scope, returns only authorized minimized customer context, and records the data-minimization decision in the audit log (D-008). |
| AI Financial Advisor | Cloud-allowed digital service that provides financial guidance based on authorized customer data. |
| API Gateway | Managed entry point for public APIs, responsible for routing, throttling, request validation, and policy enforcement. |
| BFF | Backend for Frontend. A client-oriented API layer optimized for mobile and web application needs. |
| CBS | Core Banking System. The authoritative system for financial transactions and ledger consistency. |
| CBS Transaction Gateway | Controlled integration boundary between new digital services and the legacy CBS transaction interface. |
| CDC | Change Data Capture. A technique for detecting and propagating database changes to downstream systems. |
| CQRS | Command Query Responsibility Segregation. A design style that separates write operations from read operations. |
| DB2 | Legacy transaction database used by the current CBS estate. |
| Digital Core | New on-premises service platform hosting regulated, CBS-adjacent, and latency-sensitive workloads. |
| Digital Edge | Cloud-facing platform exposing digital banking APIs and customer-facing integration. |
| DDoS | Distributed Denial of Service. A type of attack that attempts to overwhelm public systems. |
| GDPR | General Data Protection Regulation. European data protection regulation. |
| IAM | Identity and Access Management. Authentication, authorization, token, and session management capability. |
| MVP | Minimum Viable Product. The first production launch scope for the one-year delivery target. |
| NGFW | Next-Generation Firewall. On-premises firewall inspecting and filtering traffic between the cloud link, the DMZ, and internal zones. |
| ODS | Operational Data Store. A read-optimized data store used for digital queries and operational projections. |
| Open Banking | Regulated API access pattern that allows third-party providers to access customer banking data with consent. |
| RDBMS | Relational Database Management System. Example candidates for the ODS include Oracle, SQL Server, and Db2. |
| RPO | Recovery Point Objective. Maximum acceptable amount of data loss measured in time. |
| RTO | Recovery Time Objective. Maximum acceptable recovery duration after a failure. |
| TPP | Third-Party Provider. External Open Banking consumer authorized by the customer. |
| WAF | Web Application Firewall. Public security layer before the API Gateway. |
| z/OS | IBM mainframe operating system used by the legacy CBS environment. |
