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
│   ├── c4-model.md
│   ├── glossary.md
│   ├── requirements/
│   │   └── requirements-draft.md
│   ├── diagrams/
│   │   ├── 01-high-level-solution.mmd
│   │   ├── 02-account-information-flow.mmd
│   │   ├── 03-bank-transfer-flow.mmd
│   │   ├── 04-data-ingestion-read-model-flow.mmd
│   │   └── 05-ai-financial-advisor-flow.mmd
│   └── decisions/
│       ├── ADR-001-hybrid-strangler-cqrs.md
│       └── ADR-002-ods-enterprise-rdbms.md
└── project-status.md
```

## Milestones

| Milestone | Scope | Status |
|---|---|---|
| MS1 | Preliminary solution diagram and architecture narrative | In progress |
| MS2 | HLD + data + security + performance diagrams | Not started |
| Final | Full High Level Design document | Not started |

## Working Rule

All official project text is written in **English** and maintained as **Markdown** so it can later be converted into the final Word HLD submission.

## Author

Yiannis Miliaresis
