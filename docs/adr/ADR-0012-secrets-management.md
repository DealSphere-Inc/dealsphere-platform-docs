# ADR-0012: Secrets Management

**Permalink: ADR-0012: Secrets Management**

• **Author:** @dealsphere-team
• **Status:** Open
• **Date:** 2025-08-13
• **Deciders:** @name1, @name2, @name3
• **Technical Story:** [optional link to ticket/issue]
• **Tags:** secrets, security, vault

## Context

The DealSphere platform requires a secure and scalable approach to manage sensitive configuration data, API keys, database credentials, certificates, and other secrets across all microservices and environments.

PRD Alignment:
- [Secrets Management Requirements](docs/product/Phase1_PRD.md#secrets-management) — Essential for secure credential handling across the platform
- [Security Requirements](docs/product/Phase1_PRD.md#security-requirements) — Foundational security controls for Phase 1 deployment

## Decision

What is the change that we're proposing and/or doing?

Use **HashiCorp Vault** as the centralized secrets manager with application-level secret retrieval via SDK integration for Sprint 1 needs.

**For now:**
• Central secrets store with encryption at rest and in transit
• Application-level retrieval using Vault SDK (Java client libraries)
• Basic secret rotation capabilities with manual triggers

## Rationale

Why is this the right decision given our architectural pillars?

### Rationale Pillars

#### Why Secrets Management (HashiCorp Vault)

• **Security:** 
  ◦ Eliminates hardcoded credentials and reduces attack surface
  ◦ Provides encryption at rest and in transit for all secrets

• **Compliance:**
  ◦ Provides audit trails and access controls required for enterprise deployment
  ◦ Enables fine-grained access control policies

• **Operational Excellence:**
  ◦ Enables secret rotation without application restarts
  ◦ Centralized management across multiple microservices and environments

• **Team Proficiency and Velocity:**
  ◦ Consistent secret access patterns across all services
  ◦ Well-documented Java SDK integration paths

• **Modern Features without migration burden:**
  ◦ Docker Compose integration for local development
  ◦ Supports multiple authentication methods

• **Risk Management:**
  ◦ Reduces risk of credential exposure in configuration files
  ◦ Provides audit logging for all secret access operations

## Deferred Alternatives

What alternatives are we deferring for future consideration?

**Alternative: Vault Agent Sidecar Pattern**
• **Trigger:** When container orchestration matures beyond Docker Compose
• **Timeline:** Sprint 5+ when Kubernetes adoption begins
• **Reason for deferral:** Adds complexity to current Docker Compose setup

**Alternative: Cloud-native KMS Integration (AWS/Azure)**
• **Trigger:** Phase 2 cloud migration requirements
• **Timeline:** Phase 2 cloud deployment timeline
• **Reason for deferral:** Not aligned with Phase 1 on-premise requirements

#### Rejected Alternatives

• **Environment Variables in CI/CD:** No secret rotation capabilities, limited audit capabilities, secrets visible in process lists
• **KMS-only without Vault:** No centralized secret storage, complex application integration, no built-in rotation mechanisms

## Consequences

What becomes easier or more difficult to do because of this change?

**Positive:**
• Enhanced security posture with encrypted secret storage
• Audit trail for all secret access operations
• Simplified secret rotation and management
• Consistent secret access patterns across services
• Support for multiple authentication methods

**Negative:**
• Additional infrastructure component to manage
• Network dependency for secret retrieval
• Initial setup complexity for SDK integration
• Potential single point of failure (mitigated by HA deployment)

**Mitigation Strategies:**
• Services require modification for Vault integration
• Development environment setup includes Vault configuration
• Implement graceful fallback mechanisms for service startup

## Revisit Triggers

• Performance issues with secret retrieval latency (>100ms p95)
• Compliance requirements change significantly
• Alternative secrets management solutions mature substantially
• Phase 2 cloud-native requirements conflict with current approach
• Sprint 1 services exceed 5 concurrent secret requests per service

## Target Sprint for Formal Decision

• **Target Sprint:** Sprint 3 (Phase 1) for Vault infrastructure setup and core integration

## Guardrails

### Must

• All secrets MUST be stored in Vault, never in environment variables or config files
• Secret access MUST be logged and auditable
• Development environments MUST use Vault dev mode with local Docker instance
• Initial services (User Service, Auth Service) MUST integrate via Vault Java SDK
• Vault policies MUST follow principle of least privilege for Sprint 1 services

### Should

• Applications SHOULD handle Vault unavailability gracefully with cached secrets (15-minute TTL)
• Secret rotation SHOULD be implemented for database credentials
• Backup procedures SHOULD be established for Vault data

### Won't

• Advanced HA deployment (single instance acceptable for Sprint 1)
• Automated secret rotation (manual acceptable for Sprint 1)
• Multi-datacenter replication

## Approvals

| **Review** | **Reviewer** | **Date (YYYY-MM-DD)** | **Status** | **Notes** |
|------------|--------------|----------------------|------------|----------|
| Architectural Review | | | Pending | |
| Security Review | | | Pending | |
| SRE Review | | | Pending | |

## Links

**Review Before Deciding:**
• Performance benchmarks: /docs/benchmarks/api-performance.md
• Security guidelines: /docs/security/security-guidelines.md
• Architecture principles: /docs/architecture/principles.md
• Technology radar: /docs/tech-radar/technology-radar.md
• Cost analysis framework: /docs/cost-analysis/framework.md
• Scalability patterns: /docs/patterns/scalability-patterns.md
• Monitoring standards: /docs/monitoring/standards.md
• Compliance requirements: /docs/compliance/requirements.md

**Technology Landscape:** [Secrets Management](docs/tech/technology-landscape.md#secrets)

**Product/PRD:** [Secrets Management](docs/product/Phase1_PRD.md#secrets-management) and [Security Requirements](docs/product/Phase1_PRD.md#security-requirements)

**Sprint Plan:** [Phase 1 Sprint Plan](docs/tech/sprint-plan.md#phase-1-sprint-plan)

**Related ADRs:**
• [ADR-0010: Identity Provider and Claims Strategy](ADR-0010-identity-provider-and-claims-strategy.md)
• [ADR-0011: Authorization Strategy](ADR-0011-authorization-strategy.md)
