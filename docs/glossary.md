# NeoBank HLD — Glossary

| Term | Meaning |
|---|---|
| ADABAS | Legacy database used for customer information in the current NeoBank estate. |
| Advisor Context Service | On-premises data-minimization boundary between the cloud AI Financial Advisor and governed projections. Checks consent and purpose scope, returns only authorized minimized customer context, applies bank product-eligibility rules, and records the data-minimization decision in the audit log (D-008). |
| AI Financial Advisor | Cloud-allowed digital service that provides financial guidance based on authorized customer data. |
| API Gateway | Managed entry point for public APIs, responsible for routing, throttling, request validation, and policy enforcement. |
| AIS | Account Information Service under Open Banking. |
| BFF | Backend for Frontend. A client-oriented API layer optimized for mobile and web application needs. |
| CBS | Core Banking System. The authoritative system for financial transactions and ledger consistency. |
| CBS Transaction Gateway | Controlled integration boundary between new digital services and the legacy CBS transaction interface. |
| CDC | Change Data Capture. A technique for detecting and propagating database changes to downstream systems. |
| CQRS | Command Query Responsibility Segregation. A design style that separates write operations from read operations. |
| Db2 | Legacy transaction database used by the current CBS estate. |
| Digital Core | New on-premises service platform hosting regulated, CBS-adjacent, and latency-sensitive workloads. |
| Digital Edge | Cloud-facing platform exposing digital banking APIs and customer-facing integration. |
| DDoS | Distributed Denial of Service. A type of attack that attempts to overwhelm public systems. |
| GDPR | General Data Protection Regulation. European data protection regulation. |
| HSM | Hardware Security Module. Tamper-resistant custody and cryptographic-operation boundary for high-value keys. |
| IAM | Identity and Access Management. Authentication, authorization, token, and session management capability. |
| MVP | Minimum Viable Product. The first production launch scope for the one-year delivery target. |
| OIDC | OpenID Connect. Identity layer used with OAuth 2.0 for authenticated user and client context. |
| NGFW | Next-Generation Firewall. On-premises firewall inspecting and filtering traffic between the cloud link, the DMZ, and internal zones. |
| ODS | Operational Data Store. A read-optimized data store used for digital queries and operational projections. |
| Open Banking | Regulated API access pattern that allows third-party providers to access customer banking data with consent. |
| PAM | Privileged Access Management. Brokered, approved, time-bound and recorded administrator access. |
| PIS | Payment Initiation Service under Open Banking. |
| PSU | Payment Service User. The customer represented in an Open Banking consent. |
| RC0–RC3 | Recovery classes ranging from financial authority/evidence (RC0) to degradable derived capabilities (RC3). |
| RDBMS | Relational Database Management System. Example candidates for the ODS include Oracle, SQL Server, and Db2. |
| RPO | Recovery Point Objective. Maximum acceptable amount of data loss measured in time. |
| RTO | Recovery Time Objective. Maximum acceptable recovery duration after a failure. |
| TPP | Third-Party Provider. External Open Banking consumer authorized by the customer. |
| SCA | Strong Customer Authentication. Required assurance for applicable Open Banking or protected customer actions. |
| SBOM | Software Bill of Materials. Signed inventory of components included in a release artifact. |
| SLI | Service Level Indicator. Measured value used to evaluate an SLO. |
| SLO | Service Level Objective. Target for an end-to-end journey or capability. |
| TCO | Total Cost of Ownership. The annual infrastructure, software, facilities and separately reported staffing cost model. |
| WAF | Web Application Firewall. Public security layer before the API Gateway. |
| z/OS | IBM mainframe operating system used by the legacy CBS environment. |
