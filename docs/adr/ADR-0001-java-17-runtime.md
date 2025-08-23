# ADR-0001: Java 17 as Language/Runtime

**Author:** @MysterTech  
**Status:** Accepted  
**Date:** 2025-08-13  
**Deciders:** @MysterTech  
**Technical Story:** [optional link to ticket/issue]  
**Tags:** contracts, protobuf, versioning

## Context

- We need a stable, well-supported runtime to deliver Phase 1 reliably; mixed Java versions and ad-hoc JDK choices across services are causing build inconsistencies, flaky CI pipelines, and environment drift between local/dev/prod.
- Several core frameworks (e.g., Spring Boot 3.x) and observability/tooling baselines are officially validated on Java 17; targeting newer LTS versions prematurely risks dependency incompatibilities and unplanned refactors during critical delivery windows.
- Teams are losing time troubleshooting version-specific quirks (toolchain, plugins, container images) due to lack of a single runtime standard, slowing onboarding and cross-service collaboration.
- Security updates and support timelines are harder to manage with heterogeneous runtimes; we need predictable LTS maintenance to meet operational and compliance expectations.
- Phase 1 emphasizes reliability and maintainability over adopting the newest features, so a proven LTS (Java 17) minimizes delivery risk while keeping a clear path for future upgrades.

## Decision

Adopt Java 17 as the standard language/runtime for all backend services in Phase 1.

## Rationale

### Why Java 17 (Accepted)

**Stability and Support (LTS)**
- Predictable maintenance and security updates aligned with enterprise expectations.
- Reduced risk window during Phase 1 due to longer support horizon.
- Vendor ecosystem (JDK vendors, container images) provides mature, stable distributions.

**Ecosystem Readiness**
- Frameworks
  - Spring Boot 3.x and related starters are widely validated on Java 17.
  - Strong ecosystem of testing libraries (JUnit 5, Testcontainers) supports Java 17.
- Tooling
  - Build tools (Gradle/Maven) natively support Java 17 toolchains.
  - Observability agents (e.g., OpenTelemetry Java agent) and profilers are Java 17–compatible.
- CI/CD
  - Cloud runners and base images for Java 17 are standard and maintained.

**Team Proficiency and Velocity**
- Familiarity with JVM reduces onboarding and avoids context switching.
- Established patterns for debugging, profiling, and tuning on the JVM.
- Minimizes polyglot operational overhead in Phase 1.

**Modern JVM Features (without near-term migration burden)**
- Language features
  - Records for concise immutable data carriers.
  - Sealed classes for controlled inheritance and domain modeling.
- Runtime improvements
  - Mature GC options (e.g., G1/ZGC) for latency-sensitive services.
  - JVM performance optimizations benefiting microservices footprints.

**Risk Management for Phase 1**
- Avoids introducing a new LTS upgrade path mid‑phase (e.g., Java 21) before dependency readiness.
- Minimizes integration churn across multiple services and pipelines.
- Keeps focus on delivering PRD-defined core flows rather than platform migrations.

**Upgrade Path Considerations**
- Structured path to Java 21 (or newer) when:
  - Critical dependencies are certified and performance gains are tangible.
  - A test window is scheduled to run regression/performance suites.
- Build toolchain configured to ease future LTS bumps (centralized toolchain configuration).

### Alternatives Considered

**Java 21 LTS**
- Pros: Newer LTS with incremental JVM improvements.
- Cons: Some dependencies/tooling may lag certification; introducing an upgrade during Phase 1 could add risk and testing overhead.

**Kotlin-first on JVM**
- Pros: Language ergonomics (null-safety, coroutines, data classes).
- Cons: Baseline language shift not required to meet PRD Phase 1 goals; can be introduced selectively later without changing runtime.

**Node/TypeScript**
- Pros: Rapid iteration for certain I/O-heavy services.
- Cons: Polyglot complexity, diverges from JVM stack and team strengths; not necessary to meet Phase 1 scope.

## Consequences

**Positive:**
- Consistent build and runtime across services, simplifying CI/CD pipelines and operational practices.
- Access to modern JVM features (e.g., records, sealed classes, improved GC), improving code clarity and potential performance.
- Strong ecosystem support accelerates development and minimizes integration risks.
- Team proficiency with JVM enables faster onboarding and debugging capabilities.
- Predictable LTS maintenance window aligns with Phase 1 delivery timelines.

**Negative:**
- Future upgrade path to next LTS (e.g., Java 21) will require dependency validation and performance testing.
- Potential missed opportunities from newer language features in Java 21.
- Larger memory footprint compared to more lightweight runtime alternatives.

**Mitigation Strategies:**
- Maintain compatibility matrix for critical libraries to track next LTS readiness.
- Configure centralized build toolchains to ease future LTS upgrades.
- Schedule regular evaluation of Java 21 adoption benefits and ecosystem readiness.
- Implement performance benchmarks to measure upgrade impact.

## Revisit Trigger and Target Sprint

- Revisit Trigger:
  - All critical dependencies and tooling are certified on the next LTS (e.g., Java 21) and there is a measurable benefit (performance, maintainability, security).
  - A scheduled upgrade window is available with capacity for comprehensive regression testing.
- Target Sprint:
  - Sprint 1 (adoption finalized in Phase 1 foundations)

## Guardrails

### Must
- Use only LTS (Long Term Support) releases for JVM in production; never mix runtimes across services.
- Centralize all build (Gradle/Maven) configurations and container images; keep them version-locked and reviewed in CI.
- Track and document LTS readiness of all critical dependencies for upgrade planning.
- Require performance, regression, and compatibility testing for every runtime upgrade or change.
- Follow a standardized migration playbook for major Java upgrades, including rollback and validation steps.

### Should
- Review ecosystem and dependency readiness for new Java releases at least quarterly.
- Document upgrade blockers and mitigation strategies before proposing LTS changes.
- Maintain uniform JVM and build tooling across environments (local/dev/CI/prod).
- Periodically benchmark services to assess future upgrade benefits and risks.

### Won't
- Allow ad-hoc runtime or toolchain changes without ADR-backed review and CI validation.
- Mix non-LTS Java or alternative JVM languages into the baseline without architectural approval.
- Perform major upgrades during critical delivery windows or without rollback planning.

## Approvals

| Review | Reviewer | Date (YYYY-MM-DD) | Status | Notes |
|--------|----------|-------------------|-----------|-------|
| Architectural Review | @MysterTech | 2025-08-14 | Approved | |
| Security Review | @MysterTech | 2025-08-14 | Approved | |
| SRE Review | @MysterTech | 2025-08-14 | Approved | |

## Links

- 