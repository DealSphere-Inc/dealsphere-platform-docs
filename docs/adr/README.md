# Architecture Decision Records (ADRs)

This directory contains Architecture Decision Records for the DealSphere platform. ADRs document important architectural decisions made during platform development.

## Index

| ADR | Title | Status | Tags | Date |
|---|---|---|---|---|
| [ADR-0001](ADR-0001-java-17-runtime.md) | Java 17 as Language/Runtime | Accepted | `language`, `runtime`, `java` | 2025-08-13 |
| [ADR-0002](ADR-0002-microservices-architecture.md) | Microservices Architecture | Accepted | `architecture`, `microservices` | 2025-08-13 |
| [ADR-0003](ADR-0003-http-inter-service-communication.md) | HTTP for Inter-Service Communication | Accepted | `communication`, `http`, `microservices` | 2025-08-13 |
| [ADR-0004](ADR-0004-graphql-apollo-gateway-external-api.md) | GraphQL via Apollo Gateway for External API | Accepted | `api`, `graphql`, `apollo` | 2025-08-13 |
| [ADR-0005](ADR-0005-r3-corda-5-dlt-platform.md) | R3 Corda 5 as DLT Platform | Accepted | `dlt`, `corda`, `blockchain` | 2025-08-13 |
| [ADR-0006](ADR-0006-docker-compose-phase1-orchestration.md) | Docker Compose for Phase 1 Orchestration | Accepted | `orchestration`, `docker`, `compose` | 2025-08-13 |
| [ADR-0007](ADR-0007-service-framework-defaults.md) | Service Framework Default Selection | Open | `framework`, `spring`, `services` | 2025-08-13 |
| [ADR-0008](ADR-0008-database-selection.md) | Database Selection | Open | `database`, `postgresql`, `storage` | 2025-08-13 |
| [ADR-0009](ADR-0009-schema-migration-strategy.md) | Schema Migration Strategy | Open | `migrations`, `flyway`, `database` | 2025-08-13 |
| [ADR-0010](ADR-0010-identity-provider-and-claims-strategy.md) | Identity Provider and Claims Strategy | Open | `identity`, `authentication`, `claims` | 2025-08-13 |
| [ADR-0011](ADR-0011-authorization-strategy.md) | Authorization Strategy | Open | `authorization`, `security`, `rbac` | 2025-08-13 |
| [ADR-0012](ADR-0012-secrets-management.md) | Secrets Management | Open | `secrets`, `security`, `vault` | 2025-08-13 |
| [ADR-0013](ADR-0013-observability-stack.md) | Observability Stack | Open | `observability`, `monitoring`, `logging` | 2025-08-13 |
| [ADR-0014](ADR-0014-rate-limiting-strategy.md) | Rate Limiting Strategy | Open | `rate-limiting`, `api`, `throttling` | 2025-08-13 |
| [ADR-0015](ADR-0015-feature-flags-system.md) | Feature Flags System | Open | `feature-flags`, `configuration`, `deployment` | 2025-08-13 |
| [ADR-0016](ADR-0016-caching-strategy.md) | Caching Strategy | Open | `caching`, `performance`, `redis` | 2025-08-13 |
| [ADR-0017](ADR-0017-search-implementation.md) | Search Implementation | Open | `search`, `elasticsearch`, `indexing` | 2025-08-13 |
| [ADR-0018](ADR-0018-on-ledger-dlt-implementation-details.md) | On-Ledger DLT Implementation Details | Open | `dlt`, `ledger`, `transactions` | 2025-08-13 |
| [ADR-0019](ADR-0019-dlt-access-patterns.md) | DLT Access Patterns | Open | `dlt`, `access`, `integration` | 2025-08-13 |
| [ADR-0020](ADR-0020-graphql-federation-conventions.md) | GraphQL Federation Conventions | Open | `graphql`, `federation`, `schema` | 2025-08-13 |
| [ADR-0021](ADR-0021-protobuf-idl-versioning-strategy.md) | Protobuf IDL Versioning Strategy | Accepted | `protobuf`, `versioning`, `idl` | 2025-08-13 |
| [ADR-0022](ADR-0022-connection-pool-budget-management.md) | Connection Pool Budget Management | Open | `connections`, `pooling`, `resources` | 2025-08-13 |
| [ADR-0023](ADR-0023-persisted-queries-and-batching.md) | Persisted Queries and Batching | Open | `graphql`, `queries`, `performance` | 2025-08-13 |
| [ADR-0024](ADR-0024-multi-tenant-baseline-strategy.md) | Multi-Tenant Baseline Strategy | Open | `multi-tenancy`, `isolation`, `security` | 2025-08-13 |
| [ADR-0025](ADR-0025-event-streaming-platform.md) | Event Streaming Platform | Open | `events`, `streaming`, `kafka` | 2025-08-13 |

## ADR Process

### Status Definitions

- **Accepted**: Decision has been finalized and is in actual use
- **Open**: Decision is under consideration or deferred; current choice documented with alternatives and revisit triggers
- **Deprecated**: Decision is no longer applicable
- **Superseded**: Decision has been replaced by a newer ADR

### Creating ADRs

#### Step-by-Step Procedure

1. **Copy template**: Copy `docs/adr/ADR-0000-template.md` to new file `ADR-XXXX-<kebab-short-title>.md`
2. **Fill required sections**: Complete all required sections, maintaining **bolding conventions** from template
3. **Format Related ADRs**: Use number-only link text followed by short title (e.g., `[0001](ADR-0001-java-17-runtime.md) Java 17 Runtime`)
4. **Add to ADR index**: Update this README.md index table if applicable
5. **Open PR**: Submit pull request with completed ADR for review

#### General Guidelines

1. Use chronological numbering (ADR-XXXX)
2. Include relevant tags for easy lookup
3. For Open ADRs, document:
   - Current choice ("for now" selection)
   - Deferred alternative(s)
   - Revisit trigger conditions
   - Target sprint for formal decision
   - Guardrails for keeping choices reversible

### Tags

Common tags for categorization:
- `architecture`: Overall system architecture decisions
- `database`: Data storage and persistence
- `api`: API design and protocols
- `security`: Security-related decisions
- `performance`: Performance optimization
- `monitoring`: Observability and monitoring
- `deployment`: Deployment and infrastructure
- `integration`: System integration patterns
