# Architecture Decision Records (ADRs)

This directory contains Architecture Decision Records for the DealSphere platform.  
ADRs document important architectural decisions made during platform development.  
Each ADR should be updated in this index with its number, title, status, tags, and date.

---

## ADR Index

| ADR           | Title                                 | Status    | Tags                       | Date       |
|---------------|---------------------------------------|-----------|----------------------------|------------|
| ADR-0001      | Java 17 Runtime                       | Accepted  | runtime, java, backend     | 2025-XX-XX |
| ADR-0002      | R3 Corda 5 DLT Platform               | Accepted  | blockchain, dlt, backend   | 2025-XX-XX |
| ADR-0003      | Docker Compose Phase 1 Orchestration  | Accepted  | deployment, orchestration  | 2025-XX-XX |
| ADR-0004      | Service Framework Defaults            | Accepted  | architecture, tech-stack   | 2025-XX-XX |
| ADR-000-template | ADR Authoring Template             | -         | meta, template             | 2025-XX-XX |

*Replace XX-XX in Date with actual decision dates.*

---

## Status Definitions

- **Accepted:** Decision finalized and in production use.
- **Open:** Still under discussion/trial; current choice and alternatives listed.
- **Deprecated:** No longer applies (historical only).
- **Superseded:** Explicitly replaced by a newer ADR.

---

## ADR Process

To create a new ADR:
1. Copy `ADR-000-template.md` to a new file with next sequential number and short title, e.g. `ADR-0005-storage-choice.md`
2. Fill all required sections using bolded headings.
3. Reference related ADRs by number and brief title (e.g. `[0001](ADR-0001-java-17-runtime.md) Java 17`)
4. Update this index table with the new ADR.
5. Submit a pull request for review by the core team.

**General Guidelines**
- Chronological numbering (ADR-XXXX).
- Use descriptive tags (architecture, api, security, performance, etc).
- For "Open" ADRs, document deferred alternatives and triggers for revisit.
- Keep this README as the authoritative ADR directory for the repo.

---

## Common Tags

- `architecture` – Platform/system architecture patterns
- `database` – Persistence and storage
- `api` – API design/standards
- `security` – Auth, privacy, encryption
- `performance` – Optimization, scalability
- `monitoring` – Logging, metrics, alerting
- `deployment` – CI/CD, infra, containerization
- `integration` – External/partner interfaces

---

_Last updated: August 2025 – update table and tags as new decisions are finalized._
