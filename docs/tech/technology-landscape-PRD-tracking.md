# DealSphere Technology Landscape (Phase 1)

**This document applies only to Phase 1 implementation as defined in PRD 4.1–4.10 and Acceptance Criteria §6.**

## Agreed Technology Decisions (Phase 1, per PRD 4.1–4.10 and AC §6)

- **R3 Corda 5** (as blockchain/DLT)
- **Java 17** (backend runtime)
- **React + TypeScript** (frontend)
- **Open source only** (no proprietary dependencies)
- **Microservices from the start** (no monolith allowed)
- **gRPC** for service-to-service communication
- **GraphQL** for external API, implemented using Apollo Router/Gateway
- **DevOps orchestration**: Docker Compose for all environments now; Kubernetes is explicitly deferred/out-of-scope

## Phase 1 Feature Mapping (PRD 4.1–4.10)

- **4.1 Platform & Security** (RBAC, class segregation, DLT audit)
- **4.2 User & Company Management** (profile creation, authentication)
- **4.3 Deal Creation & Lifecycle** (create, update, status management)
- **4.4 Deal Discovery & Search** (search, filters, recommendations)
- **4.5 Deal Details & Viewing** (detailed views, documentation)
- **4.6 Notifications & Alerts** (system notifications, user preferences)
- **4.7 User Dashboard** (personal deal overview, activity)
- **4.8 Company Dashboard** (company-level analytics, management)
- **4.9 Administrative Functions** (system admin, user management)
- **4.10 System Configuration** (settings, integrations, maintenance)

## PRD ↔ Tech Crosswalk (Phase 1)

- **[4.1 Platform & Security](../../product/Phase1_PRD.md#41-platform--security)**
  - Responsible Service(s): Security Service, Audit Service
  - Key Tech: Corda 5 (DLT audit trail), Java 17 services, gRPC inter-service auth, GraphQL via Apollo Router (gateway security), Postgres (RBAC data)
  - Acceptance Criteria: [§6.1 Security Requirements](../../product/Phase1_PRD.md#61-security-requirements)

- **[4.2 User & Company Management](../../product/Phase1_PRD.md#42-user--company-management)**
  - Responsible Service(s): User Management Service, Company Service
  - Key Tech: Java 17 services, gRPC (service communication), GraphQL via Apollo Router (API), Postgres (user/company data)
  - Acceptance Criteria: [§6.2 User Management Requirements](../../product/Phase1_PRD.md#62-user-management-requirements)

- **[4.3 Deal Creation & Lifecycle](../../product/Phase1_PRD.md#43-deal-creation--lifecycle)**
  - Responsible Service(s): Deal Service, Workflow Service
  - Key Tech: Corda 5 (deal state management), Java 17 services, gRPC, GraphQL via Apollo Router, Postgres (deal metadata), MinIO (deal documents)
  - Acceptance Criteria: [§6.3 Deal Management Requirements](../../product/Phase1_PRD.md#63-deal-management-requirements)

- **[4.4 Deal Discovery & Search](../../product/Phase1_PRD.md#44-deal-discovery--search)**
  - Responsible Service(s): Search Service, Recommendation Service
  - Key Tech: Java 17 services, gRPC, GraphQL via Apollo Router, Postgres (search indices)
  - Acceptance Criteria: [§6.4 Search & Discovery Requirements](../../product/Phase1_PRD.md#64-search--discovery-requirements)

- **[4.5 Deal Details & Viewing](../../product/Phase1_PRD.md#45-deal-details--viewing)**
  - Responsible Service(s): Deal Service, Document Service
  - Key Tech: Java 17 services, gRPC, GraphQL via Apollo Router, Postgres (deal data), MinIO (document storage)
  - Acceptance Criteria: [§6.5 Deal Viewing Requirements](../../product/Phase1_PRD.md#65-deal-viewing-requirements)

- **[4.6 Notifications & Alerts](../../product/Phase1_PRD.md#46-notifications--alerts)**
  - Responsible Service(s): Notification Service, Event Service
  - Key Tech: Java 17 services, gRPC (event distribution), GraphQL via Apollo Router, Postgres (notification preferences)
  - Acceptance Criteria: [§6.6 Notification Requirements](../../product/Phase1_PRD.md#66-notification-requirements)

- **[4.7 User Dashboard](../../product/Phase1_PRD.md#47-user-dashboard)**
  - Responsible Service(s): Dashboard Service, Analytics Service
  - Key Tech: Java 17 services, gRPC, GraphQL via Apollo Router, Postgres (dashboard data)
  - Acceptance Criteria: [§6.7 User Dashboard Requirements](../../product/Phase1_PRD.md#67-user-dashboard-requirements)

- **[4.8 Company Dashboard](../../product/Phase1_PRD.md#48-company-dashboard)**
  - Responsible Service(s): Dashboard Service, Analytics Service, Company Service
  - Key Tech: Java 17 services, gRPC, GraphQL via Apollo Router, Postgres (company analytics)
  - Acceptance Criteria: [§6.8 Company Dashboard Requirements](../../product/Phase1_PRD.md#68-company-dashboard-requirements)

- **[4.9 Administrative Functions](../../product/Phase1_PRD.md#49-administrative-functions)**
  - Responsible Service(s): Admin Service, User Management Service
  - Key Tech: Java 17 services, gRPC, GraphQL via Apollo Router, Postgres (admin data)
  - Acceptance Criteria: [§6.9 Administrative Requirements](../../product/Phase1_PRD.md#69-administrative-requirements)

- **[4.10 System Configuration](../../product/Phase1_PRD.md#410-system-configuration)**
  - Responsible Service(s): Configuration Service, Integration Service
  - Key Tech: Java 17 services, gRPC, GraphQL via Apollo Router, Postgres (config data)
  - Acceptance Criteria: [§6.10 Configuration Requirements](../../product/Phase1_PRD.md#610-configuration-requirements)

## Architecture Implementation

### Service Architecture

- Microservices pattern with domain-driven boundaries
- gRPC for internal service-to-service communication
- GraphQL API gateway using Apollo Router for external clients
- Database per service pattern with PostgreSQL
- R3 Corda 5 for blockchain/DLT functionality

### Frontend Architecture

- Component-based React application with TypeScript
- GraphQL client integration for API consumption
- Responsive design for cross-device compatibility
- Open source UI libraries and components only

### Development & Operations

- **Local Development**: Docker Compose environment setup
- **Runtime**: Java 17 for backend services
- **Database**: PostgreSQL with service-specific schemas
- **Version Control**: Git-based workflow with feature branching
- **Testing**: Unit and integration testing frameworks
- **Documentation**: API documentation, setup guides, ADRs

## Acceptance Criteria Alignment (PRD Section 6)

All technical implementations must satisfy the acceptance criteria defined in PRD Section 6, ensuring:

- Functional requirements are met for each feature (4.1–4.10)
- Non-functional requirements (performance, security, usability) are addressed
- User experience standards are maintained
- System reliability and availability targets are achieved
- Open source compliance and microservices architecture maintained

## Decide-as-you-build (Phase 1 operating model)

- Technology decisions are made incrementally during implementation
- Each feature team has autonomy within agreed architectural boundaries
- Decision points are documented as Architecture Decision Records (ADRs)
- Regular architecture reviews ensure consistency across services
- Rapid prototyping and proof-of-concepts inform technology choices
- Cross-team collaboration on shared infrastructure and patterns
- Emphasis on pragmatic solutions that deliver Phase 1 requirements

## Open Decisions (Phase 1)

### Database Schema Migration Strategy

- **Current choice**: Flyway migration scripts
- **Deferred alternative(s)**: Liquibase, custom migration tooling
- **Revisit trigger**: Integration complexity with microservices architecture

### Frontend State Management

- **Current choice**: React Context API + useReducer
- **Deferred alternative(s)**: Redux Toolkit, Zustand, Jotai
- **Revisit trigger**: State complexity exceeds Context API capabilities

### API Rate Limiting & Throttling

- **Current choice**: Application-level rate limiting in Apollo Router
- **Deferred alternative(s)**: Redis-based distributed rate limiting, API gateway solutions
- **Revisit trigger**: Performance bottlenecks or security requirements

### Container Orchestration

- **Current choice**: Docker Compose (explicitly deferred: Kubernetes)
- **Deferred alternative(s)**: Kubernetes, Docker Swarm, cloud-native services
- **Revisit trigger**: Scaling beyond single-node deployments

### Logging & Monitoring Strategy

- **Current choice**: Structured logging with JSON format, basic metrics collection
- **Deferred alternative(s)**: ELK stack, Prometheus/Grafana, cloud monitoring solutions
- **Revisit trigger**: Production deployment readiness requirements

### Authentication & Authorization Implementation

- **Current choice**: JWT tokens with role-based access control
- **Deferred alternative(s)**: OAuth 2.0 integration, SAML, external identity providers
- **Revisit trigger**: Enterprise customer requirements or security audit findings

---

*This document is a living specification focused on Phase 1 implementation. All architectural decisions should be documented as ADRs in the `docs/adr/` directory.*
