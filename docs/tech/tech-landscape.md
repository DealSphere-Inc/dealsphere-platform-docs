# Minimal Tech Landscape for DealSphere MVP

## 1. Architectural Pattern

**Monolithic Application**

- Fast to develop and deploy for small teams and frequent iteration.
- Simplifies testing, transactions, and deployments.
- Ideal when domain boundaries are not fully volatile or discovered.
- All core modules (UI, business logic, data handling, integrations) live in a single deployable app.
- Use layered separation (Controller/Service/Repository) for maintainability and future migration to microservices, if needed.

## 2. Core Tech Stack Choices

| Layer                | Minimal Option (Best-fit)                                                    |
|----------------------|------------------------------------------------------------------------------|
| Web Frontend         | React (with Vite/CRA for speed, optional MUI/AntD for quick UI)              |
| Backend API/Logic    | Spring Boot (Java) **or** Node.js (Express)                                  |
| Database             | PostgreSQL (transactional & mature; easy migration)                          |
| File/Blob Storage    | S3-compatible service (e.g., AWS S3, MinIO local for MVP/dev)                |
| AuthN & AuthZ        | JWT-based auth with Spring Security/Express middleware                       |
| Document Management  | Local doc store + hash tracking in DB; integrate with S3 for scale           |
| AI Integration       | API-based calls to OpenAI/Claude (proxy for future custom models)            |
| R3 Corda             | As separate service; interface via HTTP/REST with mock for MVP               |
| Infra/CI             | Docker Compose (for local, simple), GitHub Actions for builds/tests          |

## 3. Optional/Pluggable

- Admin UI built into main React app with feature flags.
- Basic monitoring: logs + health check endpoint.
- Email: SMTP or 3rd-party service (SendGrid/Mailgun) from backend.

## 4. Fit with ADRs

- Stack above is aligned with accepted ADRs:
    - Layered separation (ADR: Modular boundaries)
    - S3 or compatible (ADR: Object store)
    - AI as external/sidecar (ADR: Cloud LLM API call)
    - RBAC as API middleware (ADR: Simple Auth Middleware)
    - Corda as remote or integrated API (ADR: Corda data plane)
    - Single DB, pluggable storage (ADR: DB/storage split)

## 5. Migration Path

- Partition logic/services internally for easy “lift-and-split” to microservices as needed.
- Wrap integrations/external calls in adapters/interfaces.

## 6. Monolithic Viability

- Monolithic application is strongly recommended for MVP:
    - PRD scope fits: user management, documents, workflows, analytics, integrations.
    - Evolve to microservices if required for scale/organization.

## 7. Tech Landscape Diagram (Text)        

[React Frontend]    
       |    
[Rest API (SpringBoot/Node)]    
       |    
-----------------------------    
|     |     |      |    |    |    
DB  S3/Blob Email  AI  Corda            
(Postgres) (S3)   (SMTP) (API)

