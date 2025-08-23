# ADR-0002: GraphQL for External API

**Author:** @MysterTech  
**Status:** Accepted  
**Date:** 2025-08-13  
**Deciders:** @MysterTech  
**Technical Story:** [optional link to ticket/issue]  
**Tags:** api, graphql

## Context

- The DealSphere platform requires a unified external API that can efficiently serve multiple client applications (web, mobile, partners) while maintaining strong type safety, query optimization, and schema evolution capabilities.
- The PRD requires contract-first governance (schema linting, composition checks, breaking-change detection), persisted queries/safelisting, rate limiting, caching, centralized authn/z with tenant/class enforcement, and SLO-grade observability.
- Internal services implement GraphQL subgraphs using Netflix DGS on Java 17 (ADR-0007), adopting Federation v2 for clear ownership and composition.
- The architecture must support Federation v2 features including entity references, shared types, and advanced composition patterns between Node.js gateway and Java DGS subgraphs.
- **PRD Alignment:** The Phase 1 PRD requires a single governed GraphQL endpoint with defined SLOs (latency and error budgets), enforced via CI composition checks and safe rollouts. This ADR meets that by composing DGS subgraphs under Federation v2 and enforcing persisted queries, query limits, and centralized authn/z.

## Decision

Adopt Apollo Gateway (Node.js, Federation v2) as the primary external API gateway to federate DGS subgraphs running on Java 17 for the DealSphere platform.

## Rationale

#### Why Apollo Gateway for External API?

**Federation and Contract Governance**
- Federation v2 aligns with bounded contexts and clear entity ownership.
- CI-enforced schema linting, composition, and breaking-change checks ensure safe evolution.

**Protocol Boundary: GraphQL over HTTP(S)**
- Gateway ↔ subgraphs communicate via GraphQL over HTTPS, matching federation standards and tooling.
- Keeps observability, auth, caching, and rate limiting consistent at the HTTP layer.

**Event-Driven Readiness (Protobuf)**
- Protobuf schemas maintained alongside GraphQL SDL define internal message contracts.
- Enables future Kafka-based eventing without coupling core logic to a single transport.

**Transport-Agnostic Service Design**
- Subgraphs keep business logic behind interfaces; resolvers stay thin.
- Adapters (e.g., Kafka producers/consumers) can be added later without changing external contracts.

**Security and Multi-Tenancy**
- Centralized JWT/OIDC at the gateway; authorization via directives/middleware.
- Tenant/class enforcement and audit logging applied uniformly across subgraphs.

**Operability and Scale**
- Standard metrics, traces, logs with correlation IDs; dashboards and alerts for SLOs.
- Independent scaling of gateway and subgraphs; persisted queries and query limits protect backends.

### Deferred Alternatives

- **New transport for inter-service calls (e.g., gRPC)**
  - Reason for rejection: Not required for Phase 1; GraphQL over HTTPS suffices for federation and keeps stack simpler.
  - Reconsider if: Internal service-to-service latency or throughput demands justify adding gRPC alongside HTTP.

- **Event backbone via Kafka (as primary internal integration)**
  - Reason for rejection: Not needed for initial external API delivery; adds operational complexity up front.
  - Reconsider if: Domain events and async workflows become central; throughput or decoupling needs increase.

### Rejected Alternatives

- **Monolithic GraphQL server**
  - Reason for rejection: Conflicts with bounded-context ownership and independent evolution of teams.
  - Reconsider if: Domain scope shrinks significantly and organizational structure centralizes.

- **REST/OpenAPI-only gateway**
  - Reason for rejection: Lacks GraphQL composition benefits and flexible client querying across domains.
  - Reconsider if: External consumers require strictly RESTful contracts with no GraphQL adoption path.

## Consequences

**Positive**
- Unified API endpoint simplifies client integration and governance.
- Consistent contracts and schema evolution through federation.
- Standardized observability, auth, and policies across all services.

**Negative**
- Added operational overhead for gateway and schema registry.
- Federation complexity can cause logic creep or performance risks.
- Teams must support both Node.js and Java service runtimes.

**Mitigation Strategies**
- Automated CI checks and schema validation at each deploy.
- Strict separation of concerns: thin gateway, minimal custom logic.
- Performance/load testing and cross-team documentation to maintain standards.

## Rollout Plan

- **Staging Phase:** Deploy Apollo Gateway (Federation v2) in staging with connected DGS subgraphs; enable CI checks for schema linting, composition, and breaking-change detection; configure persisted queries, baseline query limits, and centralized authn/z; set up metrics, tracing, and logs.
- **Canary Deployment:** Route limited client and operation sets through the gateway; validate SLOs and core policies (authn/z, tenant/class checks, caching, rate limiting); monitor performance, error rates, and throughput.
- **Cutover:** Gradually ramp traffic to 100% through Gateway; enforce query safelisting, rate limits, and production caching; monitor latency, errors, and throttling; deprecate any legacy endpoints per migration timelines.
- **Post-Cutover Operations:** Tune limits, caching strategies, and autoscaling based on telemetry; keep CI gates strict for ongoing schema changes; finalize incident runbooks and SLO dashboards for operation continuity.

## Revisit Triggers

- Sustained latency/cost/regression attributable to the gateway or federation model.
- Domain/team boundaries shift, making current subgraph ownership/composition misaligned.
- Material ecosystem/security changes (tooling, vulnerabilities) affecting gateway/subgraphs or governance.

## Target Sprint for Formal Decision

Target Sprint: Sprint 1

## Guardrails

### Must
- All DGS subgraphs implement Federation v2 directives correctly.
- Schema changes pass CI compatibility validation; authn/z enforced at gateway.
- Query limits, centralized error handling, and distributed tracing across all boundaries.

### Should
- Use GraphQL and DGS best practices; DataLoader for efficiency.
- Apply consistent naming, proper caching, and persisted queries in production.
- Maintain comprehensive API documentation and dashboards.

### Won't
- Allow direct DB access from resolvers; no bypass of federation/approval processes.
- Release schema changes without impact analysis or approval.
- Implement custom federation outside Apollo/DGS standards.

## Approvals

| Review | Reviewer | Date (YYYY-MM-DD) | Status | Notes |
|--------|----------|-------------------|--------|---------|
| Architectural Review | @MysterTech | 2025-08-14 | Approved | |
| Security Review | @MysterTech | 2025-08-14 | Approved | |
| SRE Review | @MysterTech | 2025-08-14 | Approved | |

## Links

**Review Before Deciding**

- Apollo Federation Documentation: [Apollo Federation v2](https://www.apollographql.com/docs/federation/)
- Netflix DGS Documentation: [DGS Framework](https://netflix.github.io/dgs/)

**Technology Landscape:** [Tech Landscape Document](../tech/tech-landscape.md)

**Product/PRD:** [Phase 1 PRD - External API](../product/Phase1_PRD.md#external-api)

**Sprint Plan:** [Link to sprint planning documentation]

**Related ADRs:**

- [ADR-0001: Architecture Principles](../ADR-0001-architecture-principles.md)
- [ADR-0002: Runtime/Platform Baseline](../ADR-0002-runtime-platform-baseline.md)
- [ADR-0003: API-First Strategy](../ADR-0003-api-first-strategy.md)
- [ADR-0005: Infrastructure Baseline](../ADR-0005-infrastructure-baseline.md)
- [ADR-0006: Security Baseline](../ADR-0006-security-baseline.md)
- [ADR-0007: Service Framework Selection (Netflix DGS, Java 17)](../ADR-0007-service-framework-selection-dgs-java17.md)
- [ADR-0008: API/Schema Governance & Federation](../ADR-0008-api-schema-governance-federation.md)
- [ADR-0011: RBAC & Authorization Policies](../ADR-0011-rbac-authorization-policies.md)
- [ADR-0012: Secrets Management](../ADR-0012-secrets-management.md)
- [ADR-0013: Observability Standards (Metrics/Traces/Logs/SLOs)](../ADR-0013-observability-standards.md)
