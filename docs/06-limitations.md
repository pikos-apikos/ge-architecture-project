# NeoBank HLD — Limitations (HLD §5)

> Required by the HLD template. Captures explicit out-of-scope items and
> architectural limitations of the MVP. Status: **scaffold — to be filled
> during MS2.**

## Out of scope (MVP)

| Limitation | Description | Implication |
|---|---|---|
| Full CBS replacement | CBS (COBOL on z/OS) remains the system of record. | The architecture depends on CBS availability; long-term modernization is not part of MVP. |
| Full enterprise data lake | Analytics is limited to ODS projections + offline fraud analytics. | Enterprise-wide historical analytics is deferred. |
| Multi-region active-active banking across legal regions | All 1M target users are in the same geographic region. | Cross-region DR is out of scope for MVP. |
| Branch modernization | Branches and branch systems are not in scope. | |
| Internal employee portal redesign | Out of scope. | |
| AI autonomous financial decisions | AI advisor is advice-only; no autonomous regulated actions. | Compliance scope is limited to advice and disclosure. |

## Architectural limitations (current HLD)

| Limitation | Description | Trigger to revisit |
|---|---|---|
| ODS vendor not selected | Shortlist is Oracle / SQL Server / Db2; final vendor deferred to MS2 (ADR-002). | MS2 vendor decision |
| Read-model freshness | Up to 24 hours stale for external bank transactions and non-app payments (NFR-PC-060). | Customer trust review / product decision |
| Offline fraud latency | Offline fraud analytics is asynchronous — does not block the synchronous transfer path. | If real-time blocking of suspected fraud becomes a regulatory requirement |
| AI advisor data scope | AI advisor only sees minimized, authorized, regionally compliant data (FN-120). | If product/legal expands advisor scope |
| API Gateway as single public ingress | Public traffic enters only through WAF → API Gateway; no direct service exposure. | If TPP or partner integration requires bypass — must go through the gateway |
| CBS throughput | CBS throughput is bounded by mainframe capacity and the CBS Transaction Gateway contract. | If digital volume exceeds CBS integration capacity, requires mainframe team engagement |

## Open limitations to confirm in MS2

- Exact CBS transaction throughput ceiling and any hard quota.
- Maximum allowed latency for same-bank and external bank transfers (OQ-004).
- Required RTO / RPO per major component (NFR-AR-020, OQ-007).
- Regulatory approval of cloud region for AI advisor data (A-003).
