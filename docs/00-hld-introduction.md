# NeoBank Digital Leap — HLD Introduction

## Abstract

NeoBank Digital Leap is a high-level architecture design for the digital transformation of a traditional retail bank. The proposed system introduces new digital banking capabilities for mobile users, web users, and Open Banking third-party providers while preserving the existing Core Banking System as the authoritative system of record for financial transactions.

The MVP covers account information, balance display, bank transfers, real-time fraud detection, offline fraud analytics, monthly income and expense reporting, Open Banking APIs, and an AI-based financial advisory service. The architecture separates money-moving operations from read-oriented digital experiences: all financial transactions are executed through the existing CBS transaction flow, while scalable read models and digital services are introduced to improve customer experience, reduce load on legacy systems, and support faster delivery of new channels and services.

The design is driven by the project's key quality attributes and constraints: 99.999% uptime, no data loss, secure access, GDPR compliance, regional data processing, monitoring for both technical and business insights, support for 100,000 digital users within the first year, and scalability toward 1 million digital users within three years.

## About This Document

The purpose of this document is to provide a clear and concise understanding of the proposed system architecture before detailed design and implementation work begins. It describes the most important system components, their relationships, their responsibilities, and the way they interact with each other across on-premises, cloud, legacy, and external environments.

This document is intended to communicate the architecture to multiple stakeholder groups:

- **Business and product stakeholders**, who need to understand how the proposed architecture supports the digital transformation objectives, customer experience, regulatory expectations, and time-to-market goals.
- **Engineering teams**, who need a shared technical blueprint for services, APIs, data flows, integration points, deployment boundaries, and implementation responsibilities.
- **Security, compliance, and risk stakeholders**, who need to understand how customer data, financial transactions, auditability, fraud detection, and GDPR requirements are addressed.
- **Operations and infrastructure teams**, who need visibility into deployment topology, availability, monitoring, recovery, scalability, and operational ownership.

The document focuses on high-level design decisions and intentionally avoids unnecessary implementation detail. It defines the architecture boundaries, assumptions, functional and non-functional requirements, major components, high-level flows, data movement, persistence strategy, external dependencies, risks, limitations, and open issues. Detailed component-level design, source code structure, database schemas, and vendor-specific implementation details may be documented separately during later design phases.

## Document Scope

This HLD covers:

- Digital channels and public API access.
- Hybrid on-premises and regional cloud deployment boundaries.
- CBS integration and transaction execution boundaries.
- Read models and operational data stores.
- Fraud detection modes.
- Open Banking access and consent boundaries.
- AI financial advisor placement and data minimization rules.
- Observability, auditability, and operational concerns.

This HLD does not cover:

- Full low-level implementation design.
- Detailed database schema definitions.
- Source code structure.
- Final vendor procurement decisions.
- Complete cost model; only high-level cost drivers and sizing assumptions are included at this stage.
