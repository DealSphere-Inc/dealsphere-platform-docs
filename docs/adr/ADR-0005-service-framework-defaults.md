# ADR-0007: Service Framework Default Selection

**Author:** @MysterTech  
**Status:** Accepted  
**Date:** 2025-08-13  
**Deciders:** @MysterTech  
**Technical Story:** [optional link to ticket/issue]  
**Tags:** framework, dgs, graphql, services  

## Context

- Services are being bootstrapped with divergent patterns (project structure, configuration, security, telemetry), leading to inconsistent realization of core Phase 1 capabilities (permissions, documents, capital calls, waterfalls, workflows, analytics, accounting, portfolio, AI), and increasing integration risk.
- Cross-cutting functional needs mandated by the PRD—strict role/class-based access, encrypted storage with hash verification, workflow SLAs and escalations, deterministic financial calculations, auditability, and API-first integrations are re-implemented differently across services, causing uneven behavior and slowing end-to-end validation.
- Inconsistent build/run conventions impede CI/CD policy enforcement required to meet PRD acceptance criteria (e.g., schema governance, security scans, performance/SLO checks, deterministic outputs for waterfalls, class segregation in AI and analytics).
- Developer onboarding and delivery velocity suffer because contributors must relearn patterns per service, delaying PRD milestones and increasing review/defect cycles.
- Phase 1 requires predictable, testable, and operable services across multiple domains and sprints; without shared defaults and guardrails, class-segregated flows (RBAC, documents, capital calls, waterfalls, workflows) and their acceptance criteria are harder to deliver reliably.

Therefore, this ADR proposes establishing minimal, opinionated defaults and starter templates focused on PRD needs:

- Security & Access: uniform authentication/authorization with strict class-level segregation and auditability.
- Data & Documents: standardized configuration for encrypted storage, on-ledger metadata, content hash verification, and versioning/audit logs.
- Observability: consistent logs/metrics/traces with baseline dashboards/alerts to validate SLAs, workflow latencies, and compliance signals.
- Resiliency: timeouts, retries, circuit breakers, backoff, idempotency patterns to support workflow reliability and external integrations.
- Deterministic Calculations: conventions for numeric precision and validation harnesses to meet waterfall test vectors and accounting accuracy.
- CI/CD & Quality: consistent build/test conventions, static analysis, coverage, schema checks, supply-chain security, and performance gates aligned to acceptance criteria.
- Interfaces & Contracts: API/schema standards and error contracts to support API-first integrations and PRD traceability.
- Developer Workflow: coherent local dev, testing, and documentation to reduce time-to-first-commit and hit sprint timelines.

## Decision

Adopt Netflix DGS (GraphQL Java with DGS) as the default service framework for Phase 1 services, with standardized starter configurations and common patterns.

## Rationale

#### Why Netflix DGS (Service Framework Default Selection)

**Stability and Support**

- GraphQL-first schema-driven development reduces boilerplate and accelerates service scaffolding
- Rich ecosystem of DGS features covers common requirements (code generation, schema registry, federation)
- Built-in observability (GraphQL instrumentation, metrics, health checks, logging)
- Native GraphQL federation support enables distributed schema composition and seamless gateway integration

**Ecosystem Readiness**

- Consistent GraphQL patterns across services reduce context switching and onboarding time
- Mature tooling integration (schema linting, breaking-change detection, IDE support)
- Performance optimizations through data loader patterns and query cost analysis

**Team Proficiency and Velocity**

- Standardized GraphQL development patterns across all microservices
- Reduced configuration drift through DGS code generation and schema registry integration
- Clear upgrade paths and long-term support from Netflix

**Modern Features without migration burden**

- Security at GraphQL layer through authorization directives and middlewares
- Established patterns for query/mutation/subscription handling and federation/gateway compatibility
- Strong community support and comprehensive documentation

**Risk Management**

- Accelerated development velocity for Phase 1 delivery
- GraphQL query cost limits and depth/complexity analysis for performance tuning
- Proven track record in similar microservices architectures at Netflix scale

### Rejected Alternatives

- **Spring Boot**
  - Reason for rejection: Rich ecosystem and team expertise, but adopting DGS focuses service delivery on GraphQL-first architecture, federation, and schema-driven contracts without additional REST overhead.
  - Reconsider if: REST-centric needs or gaps in DGS ecosystem for critical features arise after Phase 1.

- **Quarkus**
  - Reason for rejection: Fast startup and native compilation attractive, but extra learning curve and ecosystem risk during initial delivery; DGS better aligns with immediate GraphQL federation priorities.
  - Reconsider if: Post-Phase 1, performance profiling shows that native compilation or Quarkus features bring clear, material benefits.

- **Micronaut**
  - Reason for rejection: Lightweight and fast but adds framework diversity without decisive advantages for GraphQL and federation over DGS, reducing standardization.
  - Reconsider if: Team expertise or service requirements shift in Micronaut's favor, or unique features are required later.

## Consequences

**Positive:**

- Standardized GraphQL development patterns across all microservices
- Reduced configuration drift through DGS code generation and schema registry integration
- Leveraged GraphQL ecosystem for query/mutation/subscription patterns and federation compatibility
- Accelerated development velocity for Phase 1 delivery through schema-driven development
- Consistent GraphQL instrumentation and observability across services
- Enhanced API contracts through GraphQL schema-first approach
- Improved pagination and caching patterns through GraphQL best practices
- Performance tuning capabilities with query cost limits and depth/complexity analysis

**Negative:**

- Potential vendor lock-in to Netflix DGS ecosystem
- May not be optimal for non-GraphQL or high-throughput REST use cases
- Framework upgrade dependencies across multiple services
- Learning curve for teams unfamiliar with GraphQL patterns

**Mitigation Strategies:**

- Maintain service interface contracts independent of framework implementation
- Use GraphQL schema-first approaches to reduce coupling to specific implementations
- Document migration patterns for future framework transitions
- Implement CI checks for schema linting and breaking-change detection
- Establish federation/gateway compatibility for gradual framework updates

## Revisit Triggers

- Performance bottlenecks identified in Phase 1 services
- Team expertise shifts significantly toward alternative frameworks
- Major Netflix DGS security vulnerabilities or end-of-life announcements
- GraphQL ecosystem shifts requiring different tooling approaches

## Target Sprint for Formal Decision

Target Sprint: Sprint 1

## Guardrails

### Must

- All new services must use approved Netflix DGS starter templates
- Framework deviations require architecture review and explicit approval
- GraphQL schema contracts must follow federation/gateway compatibility standards
- Performance benchmarks must meet Phase 1 SLA requirements with query cost limits
- CI checks must include schema linting and breaking-change detection

### Should

- Follow Netflix DGS best practices for GraphQL schema design and code generation
- Implement standardized GraphQL instrumentation and observability patterns
- Use authorization directives and middlewares for security at GraphQL layer
- Apply data loader patterns for efficient data fetching
- Implement proper pagination and caching patterns

### Won't

- Custom GraphQL implementations without architectural approval
- Framework mixing within individual services
- Bypassing established Netflix DGS patterns without justification
- Schema changes without proper versioning and breaking-change analysis

## Approvals

| Review | Reviewer | Date (YYYY-MM-DD) | Status | Notes |
|--------|----------|-------------------|-----------|-------|
| Architectural Review | @MysterTech | 2025-08-14 | Approved | |
| Security Review | @MysterTech | 2025-08-14 | Approved | |
| SRE Review | @MysterTech | 2025-08-14 | Approved | |

## Links

**Review Before Deciding:**
- Netflix DGS Documentation: [Developer docs](https://netflix.github.io/dgs/)

**Technology Landscape:** [Tech Landscape Document](../tech/tech-landscape.md)

**Product/PRD:** [Phase 1 PRD - Service Framework](../product/Phase1_PRD.md#service-framework)

**Sprint Plan:** [Link to sprint planning documentation]

**Related ADRs:**
- [ADR-0001: Microservices Architecture](ADR-0001-microservices-architecture.md) - Establishes distributed system foundation
- [ADR-0002: API-First Design](ADR-0002-api-first-design.md) - Defines contract-driven development approach
- [ADR-0003: Technology Selection Criteria](ADR-0003-technology-selection-criteria.md) - Framework evaluation principles
- [ADR-0004: Data Layer Strategy](ADR-0004-data-layer-strategy.md) - Database and persistence patterns
- [ADR-0005: Service Templates](ADR-0005-service-templates.md) - Standardized service scaffolding
- [ADR-0006: Deployment Patterns](ADR-0006-deployment-patterns.md) - Container and orchestration standards
- [ADR-0008: Authentication Strategy](ADR-0008-authentication-strategy.md) - Identity and access management
- [ADR-0009: Monitoring and Observability](ADR-0009-monitoring-observability.md) - System visibility patterns
- [ADR-0010: Data Synchronization](ADR-0010-data-synchronization.md) - Inter-service data consistency
- [ADR-0011: Error Handling](ADR-0011-error-handling.md) - Exception and failure management
- [ADR-0012: Performance Requirements](ADR-0012-performance-requirements.md) - SLA and benchmark definitions
- [ADR-0013: Security Framework](ADR-0013-security-framework.md) - Comprehensive security approach
- [ADR-0000-template](ADR-0000-template.md) - ADR template structure
- [Microservices Architecture Guidelines](../architecture/microservices-guidelines.md)
