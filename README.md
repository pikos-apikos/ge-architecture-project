# NeoBank Digital Leap — Software Architecture Project

This repository contains the architecture work for the Software Architecture course final project: **NeoBank Digital Leap**.

The goal is to design a high-level architecture for a retail bank digital transformation initiative. NeoBank keeps its existing Core Banking System as the authoritative transaction system while introducing a new digital platform for mobile banking, web banking, Open Banking APIs, fraud detection, reporting, and an AI financial advisor.

## Current Architecture Direction

The proposed architecture uses a **hybrid on-premises and regional cloud model**:

- The existing CBS remains the system of record for financial transactions.
- A new on-premises Digital Core hosts regulated, CBS-adjacent, and latency-sensitive services.
- A regional cloud Digital Edge hosts public API access, customer-facing APIs, web/mobile integration, and the AI financial advisor where legally allowed.
- Read-heavy digital experiences use read models and operational data stores instead of calling the mainframe directly.
- Money-moving operations always pass through a controlled CBS Transaction Gateway.

## Key Architectural Patterns

- Strangler Fig modernization
- API-first digital platform
- CQRS-style separation of commands and queries
- Event-driven integration
- Hybrid deployment: on-premises + regional cloud
- Enterprise-grade availability, auditability, and security controls

## Repository Structure

```text
.
├── README.md
├── docs/
│   ├── 00-hld-introduction.md
│   ├── 01-ms1-preliminary-solution.md
│   ├── 02-ods-database-recommendation.md
│   ├── 03-assumptions.md
│   ├── 04-sizing.md
│   ├── 05-time-estimation.md
│   ├── 06-limitations.md
│   ├── 07-risks-and-mitigations.md
│   ├── 08-open-issues.md
│   ├── c4-model.md
│   ├── glossary.md
│   ├── ms1-presentation-prep.md
│   ├── requirements/
│   │   └── requirements-draft.md
│   ├── diagrams/
│   │   ├── 01-high-level-solution.mmd
│   │   ├── 02-account-information-flow.mmd
│   │   ├── 03-bank-transfer-flow.mmd
│   │   ├── 04-data-ingestion-read-model-flow.mmd
│   │   ├── 05-ai-financial-advisor-flow.mmd
│   │   ├── 06-network-topology.mmd
│   │   ├── 07-c4-container.mmd
│   │   └── neobank-digital-leap-c4.drawio
│   └── decisions/
│       ├── ADR-001-hybrid-strangler-cqrs.md
│       └── ADR-002-ods-enterprise-rdbms.md
└── project-status.md
```

## Milestones

| Milestone | Scope | Evidence | Status |
|---|---|---|---|
| MS1 | Preliminary solution, architecture narrative, and presentation | [MS1 source](docs/01-ms1-preliminary-solution.md) · [MS1 deck](docs/ms1-presentation/index.html) | Completed |
| MS2 | Expanded HLD with data, security, performance, resilience, and network-topology views | [Architecture source](docs/09-system-architecture.md) · [governed diagrams](docs/diagrams/) | Completed |
| Final HLD | Submission-ready HLD, managed contracts, traceability, sizing, cost, and delivery estimate | [Assembly manifest](docs/final-hld-assembly-manifest.md) · [appendices](docs/appendices/) | Completed |
| Final presentation | Fifteen-slide Final HLD presentation with speaker notes | [Final presentation](docs/final-presentation/index.html) | Completed |
| Workflow retrospective | Wayfinder adaptation and Agent Technical English workflow presentation | [Wayfinder + ATE presentation](docs/wayfinder-ate-presentation/index.html) | Completed |

## Working Rule

All official project text is written in **English** and maintained as **Markdown** so it can later be converted into the final Word HLD submission.

## Author

Yiannis Miliaresis

## License

Except for the third-party material identified below, this repository is licensed under the [Apache License 2.0](LICENSE).

The project-specific Wayfinder adaptation in `.codex/skills/wayfinder/SKILL.md` incorporates material from Matt Pocock's MIT-licensed Wayfinder. The original MIT notice is preserved in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

Copyright 2026 Yiannis Miliaresis.
