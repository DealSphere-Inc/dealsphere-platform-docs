# DealSphere MVP: Minimal Technical Landscape

## 1. Architecture Overview
- Monolithic web application
    - Modular project structure for later extraction/migration
    - ADR-driven decisions documented ([see ADR-0001-0007])
- Single cloud environment (e.g., AWS, Azure, GCP) with standard VPC
- CI/CD pipeline performing automated build, test, and deploy (GitHub Actions or similar)

## 2. Core Technology Stack

| Layer                  | MVP Choice                          | Rationale                                               |
|------------------------|-------------------------------------|---------------------------------------------------------|
| UI/Frontend            | React (TypeScript)                  | Widely supported, fast MVP UI, aligns with ADRs         |
| Backend/API            | Java (Spring Boot, minimal modules) | Robust, PRD-compliant, supports later modularization    |
| Database               | PostgreSQL                          | Relational, open-source, easy migration/scaling         |
| AuthN/AuthZ            | JWT, Role-based Access (RBAC)       | Satisfies security/privacy ADRs and PRD                 |
| Infrastructure as Code | Terraform or CloudFormation (basic) | Repeatable, tracks config changes (ADR-0005/6 aligned)  |
| Documentation          | Markdown in repo (docs/), OpenAPI   | Lightweight, supports ADR-0007 (public APIs/comments)   |

## 3. Key Integrations
- Document Storage: Cloud object storage (e.g., S3 Bucket)
- AI Service: External API (OpenAI or Azure OpenAI)
- Payment/Bank: Mocked or stubbed for MVP, upgradeable
- Blockchain/Corda: Integrated as a soft adapter or simulation layer (minimal live nodes for demo only, upgrade path clear per ADR)

## 4. Testing
- Unit/integration tests: JUnit (backend), React Testing Library (frontend)
- Smoke/UI tests: Cypress or Playwright (core flows only)
- Documentation checks: ADRs tracked and auto-validated (adr-tools)

## 5. DevOps & Deployment
- Single environment: Prod and staging from unified codebase, IaC driven
- Persistence: Automated DB migrations, basic backup
- Monitoring: App + error logs only for MVP (CloudWatch/Stackdriver)

## 6. Security & Compliance
- Encryption: TLS in transit, DB encryption at rest
- Basic audit logging: User activity written to DB log table (searchable)
- No explicit multi-tenant isolation for MVP
- Public APIs: Documented and commented inline per ADR-0007

## 7. Documentation
- ADRs: Markdown files, numbered, in docs/adr/
- API: OpenAPI spec (docs/api/)
- PRD Mapping: README and mapping tables in docs/

---

### Open/Deferred for post-MVP
- Full Kubernetes orchestration
- End-to-end automated compliance suite
- Multi-region/cloud active-active deployment
- Advanced observability, tracing

---

*This MVP tech stack and approach:*
- Ensures all ADR-0001–7 are directly referenced in the repo and visible to developers
- Delivers all PRD requirements in the simplest, fastest-to-demo way, with all critical upgrade paths and doc tracks established
