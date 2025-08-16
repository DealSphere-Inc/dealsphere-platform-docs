# ADR-0021: Protobuf IDL Versioning Strategy

**Author:** @MysterTech  
**Status:** Accepted  
**Date:** 2025-08-13  
**Deciders:** @MysterTech  
**Technical Story:** [optional link to ticket/issue]  
**Tags:** contracts, protobuf, versioning

## Context

As we launch a greenfield microservices platform, all service-to-service communication will use Protocol Buffers (protobuf) to define structured payloads over HTTP. The key issue driving this decision is the need to establish clear, consistent versioning and compatibility practices for protobuf contracts from the outset. Without disciplined guidelines, rapid team growth and parallel development could lead to accidental breaking changes or developer friction as services evolve.

Although we do not need to migrate or preserve legacy data, it is critical to set strong IDL (Interface Definition Language) management foundations so every future schema change remains safe, transparent, and well-governed. This proactive approach will enable autonomous teams to iterate quickly while maintaining reliability and cross-service compatibility as new features and services are added.

## Decision

Adopt a transport-agnostic protobuf IDL versioning strategy that mandates backward-compatible evolution by default, explicit deprecation timelines, and CI-enforced compatibility checks for all schema changes used over HTTP (and any other future transports).

## Rationale

### Why a Protobuf IDL Versioning Strategy?

**Prevent Breaking Changes Early**
- Even in a greenfield, multiple teams iterating in parallel can unintentionally introduce breaking schema changes.
- Clear evolution rules (additive-first, reserved fields/tags, explicit deprecations) protect downstream consumers and reduce coordination overhead.

**Strong Contracts with HTTP**
- Protobuf provides compact, strongly-typed contracts for HTTP payloads, enabling efficient S2S calls without coupling to gRPC semantics.
- A transport-agnostic IDL lets us evolve payloads while keeping HTTP concerns (status codes, headers, error envelopes) cleanly separated.

**Autonomy with Governance**
- Teams can iterate independently when contract rules, compatibility checks, and review workflows are standardized.
- Automated CI compatibility validation (backward/forward) reduces review burden and catches issues before merge.

**Observability and Debuggability**
- Consistent field evolution and error envelope conventions improve logs, traces, and on-call diagnostics.
- Predictable schema changes make it easier to analyze incidents and roll back safely.

### Future Direction

While HTTP with protobuf is our standard for synchronous service-to-service communication today, we anticipate migrating to a message broker architecture (e.g., Kafka) for asynchronous/event-driven workflows as platform scale and complexity grows. Protobuf contracts and disciplined versioning will remain foundational, ensuring a smooth transition to Kafka or similar brokers and unified schema evolution across both synchronous (HTTP) and asynchronous (message broker) channels.

### Deferred Alternatives

**Schema Registries and Central Brokers**
- Reason for deferral: Adds operational complexity not required for HTTP payloads at current scale.
- Revisit when: Cross-language schema discovery, runtime validation, or large-scale event streaming warrants registry-backed workflows.

**Full API Contract Federation (beyond protobuf)**
- Reason for deferral: Federation is more relevant to external API composition; internal S2S needs are met with protobuf + HTTP.
- Revisit when: Internal query composition across domains becomes a core requirement.

### Rejected Alternatives

**Ad-hoc JSON-only Contracts**
- Reason for rejection: Lacks strong typing, increases payload size, and raises risk of silent breaking changes.
- Reconsider only if: A service is strictly external-facing and human-readability/debuggability outweighs efficiency (not our S2S default).

**gRPC-Coupled Evolution Rules**
- Reason for rejection: Ties contract evolution to a transport we are not using for S2S; we need transport-agnostic guidance focused on HTTP payloads.
- Reconsider only if: We adopt gRPC broadly for S2S in the future and need its RPC-specific semantics.

## Consequences

**Positive**
- Consistent, strongly-typed contracts reduce accidental breaking changes and improve cross-team velocity.
- Smaller, faster payloads with protobuf improve internal latency and bandwidth efficiency over HTTP.
- Transport-agnostic IDL keeps options open for future Kafka/event-driven adoption with minimal rework.

**Negative**
- Additional governance overhead: PR reviews, CI compatibility checks, and documentation discipline are required.
- Binary payloads make ad-hoc debugging harder than JSON; teams must rely on tooling and generated stubs.
- Strict compatibility rules can slow down radical refactors without planned deprecation windows.

**Mitigation Strategies**
- Provide tooling (pre-commit hooks, CI jobs) for linting, backward/forward compatibility checks, and doc generation.
- Support content negotiation for JSON in dev/debug scenarios while defaulting to protobuf for S2S.
- Establish clear deprecation policies, change logs, and reserved field/tag practices to enable safe evolution.

## Rollout Plan

**Baseline Setup:**
- Establish a central, version-controlled repository for all protobuf contracts.
- Implement mandatory pull request reviews and CI jobs for schema linting and compatibility checks.
- Publish guidelines covering field/tags discipline, versioning, deprecation, and changelogs.

**Phase 1 Adoption:**
- Require all new services to source protobuf contracts from the central repo for HTTP payloads.
- Integrate automated CI enforcement for compatibility, additive changes, and reserved field checks.
- Document contract evolution and maintain history in the repo changelog.

**Continuous Improvement:**
- Periodically review contract evolution incidents and update guidelines as needed.
- Expand integration with automated contract documentation and monitoring dashboards.
- Revisit broker/event-driven adoption and schema registry needs as the number and complexity of asynchronous workflows grow.

**Future Broker Integration:**
- As event-driven patterns and Kafka adoption increase, plan for seamless schema management extension to broker channels—maintaining central repo discipline and considering schema registry adoption when justified by scale or polyglot scenarios.

## Revisit Trigger and Target Sprint

**Revisit Trigger:**  
- Compatibility checks become insufficient for real-world schemas, causing incidents.  
- Significant ecosystem/tooling improvements suggest an alternative IDL is materially better.  
- Widespread need for schema features protobuf cannot address cleanly.

**Target Sprint:**  
- Sprint 1 (baseline enforcement and documentation as part of Phase 1 foundations).

## Guardrails

### Must
- Define and manage all protobuf schemas in a central, version-controlled repository with mandatory PR reviews and CI compatibility checks.
- Use unique, never-reused field numbers; only add new optional fields or explicitly deprecate old ones.
- Maintain backward compatibility for all contract changes; deprecations must be documented with clear communication and established timelines.
- Enforce CI checks to lint, validate, and prevent tag reuse or breaking changes on every schema update.
- Contracts for both HTTP and future broker transports (e.g., Kafka) must follow the same versioning discipline and central governance.

### Should
- Prefer additive changes; avoid renaming fields and instead add new ones while deprecating old.
- Centralize common types and follow consistent naming and package conventions across all services.
- Provide JSON encoding for dev/debug scenarios but default to protobuf for service-to-service payloads.
- Review contract evolution incidents periodically and update guidelines as needed.

### Won’t
- Tie versioning, evolution, or schema governance to any single transport protocol (e.g., avoid gRPC-specific semantics).
- Reuse field numbers, remove fields without documentation or migration path, or bypass central schema repository processes.
- Merge business logic into serialization or transport layers.

## Approvals

| Review | Reviewer | Date (YYYY-MM-DD) | Status | Notes |
|--------|----------|-------------------|-----------|-------|
| Architectural Review | @MysterTech | 2025-08-14 | Approved | |
| Security Review | @MysterTech | 2025-08-14 | Approved | |
| SRE Review | @MysterTech | 2025-08-14 | Approved | |

## Links

- **Technology Landscape:** ../tech/technology-landscape.md#phase-1-technical-plan  
- **Sprint Plan:** ../tech/sprint-plan-phase-1.md#phase-1-sprint-plan

- **Related ADRs**
  - [ADR-0001: Java 17 as Language/Runtime](ADR-0001-java-17-runtime.md)
  - [ADR-0002: Microservices Architecture](ADR-0002-microservices-architecture.md)
  - [ADR-0003: HTTP for Inter-Service Communication](ADR-0003-http-inter-service-communication.md)
  - [ADR-0004: GraphQL via Apollo Gateway for External API](ADR-0004-graphql-apollo-gateway-external-api.md)
  - [ADR-0005: R3 Corda 5 as DLT Platform](ADR-0005-r3-corda-5-dlt-platform.md)
  - [ADR-0006: Docker Compose for Phase 1 Orchestration](ADR-0006-docker-compose-phase1-orchestration.md)
  - [ADR-0007: Service Framework Default Selection](ADR-0007-service-framework-defaults.md)
  - [ADR-0008: Database Selection](ADR-0008-database-selection.md)
  - [ADR-0011: Authorization Strategy](ADR-0011-authorization-strategy.md)
  - [ADR-0012: Secrets Management](ADR-0012-secrets-management.md)
  - [ADR-0013: Observability Stack](ADR-0013-observability-stack.md)
  - [ADR-0022: Connection Pool Budget Management](ADR-0022-connection-pool-budget-management.md)
  - [ADR-0023: Persisted Queries and Batching](ADR-0023-persisted-queries-and-batching.md)
  - [ADR-0025: Event Streaming Platform](ADR-0025-event-streaming-platform.md)
