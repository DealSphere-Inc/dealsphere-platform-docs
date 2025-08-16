# ADR-0003: HTTP for Inter-Service Communication

**Author:** @MysterTech  
**Status:** Accepted  
**Date:** 2025-08-13  
**Deciders:** @MysterTech  
**Technical Story:** [optional link to ticket/issue]  
**Tags:** communication, https, microservices

## Context

The current challenge with inter-service communication in our microservices platform is a lack of simplicity, reliability, and easy debuggability. Previous approaches either relied on binary protocols (like gRPC) that complicate local testing and troubleshooting or added operational overhead using message brokers for synchronous flows. Teams face difficulties rapidly integrating new services, evolving contracts, and tracking request flows across domains—especially for simple request-response patterns. Our platform demands secure, type-safe, and easily observable communication without steep learning curves or operational friction.

As a result, we need a unified, standard HTTP-based approach for inter-service calls. This will:

- Simplify integration for new and existing services,
- Improve transparency and debuggability,
- Align with broader tooling and organizational expertise,
- And address PRD requirements for scalability and maintainability in real-time deal, document, and DLT workflows.

## Decision

Adopt HTTP as the primary protocol for inter-service communication within the DealSphere platform.

## Rationale

### Why HTTP for Inter-Service Communication?

- **Simplicity and Organizational Fit**  
  - Ubiquitous protocol with low onboarding cost and rich framework/tooling support.  
  - Easier local testing and debugging than binary RPC; straightforward polyglot integration.  
  - Aligns with existing infra, security, and observability practices.
- **Payload Flexibility**  
  - Supports JSON for readability and protobuf for efficiency via content negotiation.  
  - Allows gradual optimization (start with JSON, adopt protobuf where needed).  
  - Avoids lock-in to a single RPC framework while preserving strong contracts.
- **Operational Transparency**  
  - Clear request/response semantics, standard status codes, and mature tracing/logging.  
  - First-class TLS/JWT integration; consistent gateway/middlebox behavior.  
  - Simplifies SLO monitoring, rate limiting, and error handling across services.

### Deferred Alternatives

- **Kafka (or equivalent) as primary internal transport**  
  - Reason for deferral: Overhead for synchronous request/response; not necessary for initial scope.  
  - Reconsider if: Async/event-driven patterns become dominant or decoupling/throughput needs increase significantly.

### Rejected Alternatives

- **gRPC for synchronous inter-service calls**  
  - Reason for deferral: Adds ecosystem complexity and tighter coupling to RPC semantics not required for Phase 1.  
  - Reconsider if: Latency/throughput demands exceed HTTP+protobuf or streaming becomes a pervasive requirement.
- REST/HTTP JSON only (no binary encoding)  
  - Reason for rejection: Inefficient for high-volume paths; lacks a clear path to binary optimization.  
  - Reconsider if: Traffic levels remain low and developer ergonomics outweigh efficiency concerns.

## Consequences

**Positive**

- Service integration is faster and easier due to HTTP's universal support and familiar tooling.
- Debugging, monitoring, and tracing are straightforward for all teams and environments.
- Payload format flexibility enables optimization (protobuf) without losing readability or dev ergonomics.

**Negative**

- Lacks some streaming or advanced RPC features available in gRPC; may require more manual implementation for certain patterns.
- Binary protobuf payloads require developers to manage serialization, contracts, and compatibility discipline.
- Direct efficiency (latency, throughput) may be lower than binary RPC for very high-frequency, low-latency needs.

**Mitigation Strategies**

- Leverage content negotiation to support both JSON (for dev/onboarding/debug) and protobuf (for prod/high-throughput paths).
- Invest in automated contract validation, linting, and compatibility checks in CI.
- Document versioning, error handling, and back-compat best practices for payloads and HTTP APIs.

## Revisit Trigger and Target Sprint

- **Revisit Trigger:**  
  - HTTP struggles to meet latency or throughput needs for synchronous service calls under real-world load.  
  - Teams experience significant pain points in contract evolution, debugging, or developer ergonomics not resolved by available tooling.  
  - Event-driven or streaming use cases increase, making HTTP less fit for internal core workflows.  
  - Major improvements in protocol or framework support (e.g., gRPC, event brokers) change the cost/benefit landscape.  
  - PRD requirements evolve to emphasize patterns not easily supported via HTTP.
- **Target Sprint:**  
  - Sprint 1 (initial rollout and adoption complete as part of Phase 1 platform foundation).

## Guardrails

### Must

- Enforce TLS for all service-to-service HTTP calls; authenticate requests (e.g., JWT/OIDC) and authorize per service policy.
- Define explicit timeouts, retries, and circuit breakers for every endpoint; propagate correlation IDs for tracing.
- Version payload contracts; maintain backward compatibility and document deprecation timelines.

### Should

- Support both JSON (dev/debug) and protobuf (prod/high-throughput) via content negotiation.
- Standardize error envelopes and HTTP status code mapping; include machine-readable error codes.
- Instrument requests with metrics (latency, error rate, throughput) and distributed tracing; publish dashboards and SLOs.

### Won't

- Expose internal HTTP endpoints directly to external clients or bypass gateway/mesh policies.
- Share internal domain objects across service boundaries; no tight coupling to transport-specific types.
- Deploy breaking schema changes without CI compatibility checks and documented migration paths.

## Approvals

| Review | Reviewer | Date (YYYY-MM-DD) | Status | Notes |
|--------|----------|-------------------|--------|---------|
| Architectural Review | @MysterTech | 2025-08-14 | Approved | |
| Security Review | @MysterTech | 2025-08-14 | Approved | |
| SRE Review | @MysterTech | 2025-08-14 | Approved | |

## Links

- **Technology Landscape:** [Phase 1 Technical Plan](docs/tech/technology-landscape.md#phase-1-technical-plan)
- **Sprint Plan:** [Phase 1 Sprint Plan](docs/tech/sprint-plan-phase-1.md#phase-1-sprint-plan)
- **Related ADRs:**  
  
  - [ADR-0001: Architecture Principles](/docs/adr/ADR-0001-java-17-runtime.md)  
  - [ADR-0002: Microservices Architecture](/docs/adr/ADR-0002-microservices-architecture.md)  
  - [ADR-0004: GraphQL via Apollo Router for External API](/docs/adr/ADR-0004-graphql-apollo-gateway-external-api.md)  
  - [ADR-0005: Infrastructure Baseline](/docs/adr/ADR-0005-r3-corda-5-dlt-platform.md)  
  - [ADR-0006: Security Baseline](/docs/adr/ADR-0006-docker-compose-phase1-orchestration.md)  
  - [ADR-0007: Service Framework Selection](/docs/adr/ADR-0007-service-framework-defaults.md)  
  - [ADR-0008: API/Schema Governance & Federation](/docs/adr/ADR-0008-database-selection.md)  
  - [ADR-0011: RBAC & Authorization Policies](/docs/adr/ADR-0011-authorization-strategy.md)  
  - [ADR-0012: Secrets Management](/docs/adr/ADR-0012-secrets-management.md)  
  - [ADR-0013: Observability Standards](/docs/adr/ADR-0013-observability-stack.md)  
  - [ADR-0021: Protobuf IDL Versioning Strategy](/docs/adr/ADR-0021-grpc-idl-versioning-strategy.md)  
  - [ADR-0025: Event Streaming Platform](/docs/adr/ADR-0025-event-streaming-platform.md)
