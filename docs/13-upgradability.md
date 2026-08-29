# 3.5 Upgradability, Release and Secure SDLC

NeoBank uses immutable-artifact promotion, compatibility-first change, progressive delivery, and evidence-based production admission. Non-production environments use scheduled auto-shutdown outside working hours unless a justified, time-bound override is approved.

The authoritative visual sources are:

- `docs/diagrams/15-release-promotion-evidence.mmd`
- `docs/diagrams/16-zero-downtime-upgrade-recovery.mmd`

## Final release, upgrade, and Secure SDLC matrix — approved

**DECISION OWNER: HUMAN**

This matrix compiles the approved decisions. The ticket MUST remain open until the user approves the complete model.

### Environment and data controls

| Area | Approved control |
|---|---|
| Environments | DEV, INTEGRATION, PRE-PROD, and PROD MUST use separate identities, secrets, network boundaries, and data stores. PRE-PROD MUST provide production-like release, performance, resilience, and recovery rehearsal. |
| Artifact promotion | The delivery process MUST build once and promote the same immutable signed artifact digest. Configuration and secrets MUST remain external. |
| Non-production cost | Non-production compute MUST stop automatically outside its configured working window. An override MUST record owner, reason, scope, start, and expiry. |
| Test data | Non-production MUST use synthetic data by default. Approved masked or tokenized data MAY enter only after transformation, Security and Data Protection approval, bounded access, and scheduled deletion. Raw production PII and secrets are prohibited. |

### Source, supply-chain, and Secure SDLC controls

- Protected release branches MUST reject direct writes.
- A standard change MUST receive one independent approval.
- A high-risk security, money-movement, CBS, contract, data, or infrastructure change MUST receive two independent approvals, including the applicable control-domain owner.
- The author MUST NOT self-approve or become the only source and production authority.
- Each artifact MUST have a digest, signature, provenance, and digest-bound SBOM.
- Production admission MUST fail closed if the digest, signature, provenance, SBOM reference, or approval is absent or inconsistent.
- SAST, SCA, license, and secret checks MUST run before merge.
- Artifact, malware, vulnerability, configuration, IaC, network-policy, and privilege checks MUST run where applicable.
- An unresolved Critical, exploitable High, confirmed secret exposure, or prohibited policy result MUST block promotion.
- Risk acceptance MUST be artifact-specific, approved by Security and the risk owner, time-bound, and auditable.

### Progressive promotion

| Gate | Minimum evidence |
|---|---|
| Merge | Review, automated tests, architecture rules, source and dependency security checks |
| Integration | API/event/database/cache/CBS compatibility, integration tests, migration and replay safety, artifact checks |
| Pre-production | Affected journeys from the approved eight-journey catalog, performance, resilience, recovery, reconciliation, deployment rehearsal, dashboards, alerts, and runbooks |
| Production | Signed artifact, provenance, SBOM, all required results and approvals, change classification, recovery plan, canary queries, and accountable owner |

A failed required gate MUST stop promotion.

### Compatibility and versioning

- APIs and events MUST remain additive within a major version.
- A breaking change MUST use a new major version and parallel support.
- Database changes MUST use expand, migrate, and contract.
- Cache changes MUST use versioned keys or namespaces when representations are incompatible.
- CBS Gateway changes MUST use parallel contract versions and joint Digital/Mainframe activation.
- A destructive change MUST stop while any supported consumer, replay path, recovery site, or rollback dependency requires the old representation.
- Public and Open Banking API retirement MUST provide at least 12 months of notice and parallel support.
- Internal API/event retirement MUST provide at least 90 days and two normal production release cycles, whichever is longer.
- CBS Gateway retirement MUST provide at least six months of parallel support and joint approval.

### Deployment by recovery class

| Class | Deployment behavior |
|---|---|
| RC0 | Small canary, surge capacity, zero downtime, one failure domain at a time, no concurrent primary/DR upgrade, zero-loss and reconciliation gates |
| RC1 | Canary followed by zone- or fault-domain-based rolling deployment with healthy capacity and prior-version availability |
| RC2 | Rolling consumer/worker replacement with durable checkpoints, pause/restart/replay, lag, quarantine, convergence, and reconciliation gates |
| RC3 | Rolling or scheduled replacement; approved temporary degradation MAY occur within the RC3 target; incomplete statements and unverified AI advice MUST NOT publish |

Stateful migration MUST remain separate from application rollout.

### Canary and recovery

- Traffic MUST progress through canary, limited, and full stages.
- Each stage MUST evaluate journey SLOs, technical health, security signals, business invariants, CBS outcomes, idempotency, event lag, consent, audit, and reconciliation as applicable.
- A material authority, financial, security, consent, or evidence discrepancy MUST abort the canary.
- Missing required telemetry MUST stop advancement.
- Stateless rollback MAY restore the prior signed artifact only while all active state and contracts remain compatible.
- A committed CBS transaction or durable event MUST NOT be deleted or reversed by application rollback.
- Externally visible or irreversible state MUST use roll-forward, approved compensation, replay, and reconciliation.
- An irreversible change MUST declare its point of no return, checkpoint, recovery procedure, owners, approvals, and reconciliation criteria before execution.

### CBS and Mainframe coordination

- Digital and mainframe artifacts MAY deploy in separate windows.
- Both sides MUST remain compatible until controlled activation.
- Contract, idempotency, timeout, unknown-outcome, throughput, admission, audit, and reconciliation tests MUST pass before activation.
- Digital Release Owner and Mainframe Authority MUST approve activation and retirement.
- A coupled release window MAY be used only when technical decoupling is impossible and the exception records the dependency and recovery plan.

### Operational governance

- Feature flags MUST have owner, purpose, safe default, scope, telemetry, expiry, and removal work.
- Feature flags MUST NOT bypass authentication, consent, fraud, CBS authority, audit, signing, admission, or data controls.
- Emergency changes MUST use an audited break-glass fast path approved by Incident Commander and the applicable Service, Security, or Mainframe authority.
- Emergency changes MUST retain signing, provenance, secret checks, admission, JIT access, audit evidence, and a recovery plan.
- Normal production deployment MUST use the delivery workload identity.
- Engineers MUST NOT retain standing production write access.
- Exceptional access MUST use individual JIT PAM grants with scope, reason, approval, expiry, attribution, and session evidence.
- Shared production accounts and untracked direct changes are prohibited.

### Evidence and retention

- Each production release MUST create one immutable signed canonical release receipt.
- The receipt MUST bind source, artifact, provenance, SBOM, gates, exceptions, approvals, migrations, flags, canary results, access, recovery actions, and final outcome.
- The canonical receipt MUST remain in WORM storage for seven years.
- Detailed pipeline, scanner, test, canary, and session evidence MUST remain for at least 13 months.
- Incident, investigation, regulatory, or legal-hold evidence MUST remain until the hold closes.
- Evidence MUST minimize customer data and MUST NOT contain plaintext secrets.

### Required HLD visuals

1. Release Promotion and Evidence Flow: four environments, one artifact digest, gates, signing, promotion, canary, receipt, and non-production schedule.
2. Zero-Downtime Upgrade and Recovery Sequence: version coexistence, expand–migrate–contract, CBS activation, point of no return, abort, rollback-safe and roll-forward branches.
3. RC0 through RC3 deployment-control table.

### Explicit validation items

The HLD MUST preserve the controls above and MUST validate:

- available source-control, CI/CD, signing, provenance, SBOM, scanner, PAM, session-recording, and WORM capabilities;
- bank-approved non-production schedules and override authority;
- emergency retrospective SLA;
- mainframe test environment, throughput evidence, and release calendar;
- exact canary cohort sizes, observation windows, and thresholds;
- signing-key custody and workload-identity operation;
- regulatory or scheme requirements that extend public API support periods.

If a selected tool or organizational process cannot satisfy an approved control, the gap MUST become a production-readiness risk. Validation MUST NOT silently weaken this matrix.

**Human decision recorded for validation posture:** “Συμφωνώ” in response to mandatory controls with explicit validation items.

**FINAL HUMAN GATE:** Approve, reject, defer, or request a change to this complete release, upgrade, and Secure SDLC model.

---


