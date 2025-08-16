# Platform Development — Phase 1 Backlog (Epics and Stories)

## Sprint Mapping Overview

This document organizes Phase 1 epics and stories into 5 development sprints aligned with the technical plan:

- **Sprint 1**: Core Platform Foundation, Identity/Security, Secrets Management, Observability, CI/CD basics
- **Sprint 2**: Document Management, Capital Calls, Frontend/BFF Foundation  
- **Sprint 3**: Workflow Automation, Notifications
- **Sprint 4**: Integration Testing, Acceptance & E2E Tests
- **Sprint 5**: Production Readiness, ADR Formalization, Guardrails

Below is the complete epic and story list using the "Epic" naming format, aligned to Phase 1 execution and PRD/ADRs.

## Epic: Core Platform Foundation **[Sprint 1]**

-  Story: Initialize monorepo scaffolding (Gradle, module layout, quality gates, CI bootstrap)
-  Story: Corda 5 local/dev network bootstrap (Compose nodes, network map, certs, health checks)
-  Story: Service archetype templates (Spring Boot 3 starters, logging, error model)
-  Story: gRPC and GraphQL BFF baseline (Proto module, Apollo Router config, schema gateway)
-  Story: Environment configuration strategy (profiles, precedence, secret indirection)
-  Story: Repository governance (branching, CODEOWNERS, PR templates)
-  **Story: API gateway skeleton with auth & class segregation checks** **[Sprint 1]**
-  **Story: Multi-tenant user isolation tests** **[Sprint 1]**
-  **Story: Documentation structure** **[Sprint 1]**
-  **Story: Git workflow & branching** **[Sprint 1]**

## Epic: Identity, Security, and RBAC **[Sprint 1]**

-  Story: Keycloak realm, clients, roles, and OIDC config (scopes, claims)
-  Story: JWT validation middleware and security defaults (RS256, issuer/audience)
-  Story: Claims & class propagation end-to-end (Router → BFF → services)
-  Story: RBAC enforcement on APIs and flows (@PreAuthorize with resource:action)
-  Story: Security testing & negative cases (token tampering, scope/role failures)
-  Story: Session and token lifecycle settings (access/refresh, PKCE, rotation)

## Epic: Secrets and Configuration Management **[Sprint 1]**

-  Story: Vault dev deployment with base policies (Compose service, AppRole/dev auth)
-  Story: Vault SDK integration in services (DB creds, API keys, retry/backoff, cached TTL)
-  Story: Secret rotation runbook and manual procedure (DB creds)
-  Story: TLS baseline (self-signed, truststores, Router/BFF/services TLS)
-  Story: Secrets access auditing and alerts

## Epic: Observability (Metrics, Logs, Traces) **[Sprint 1]**

-  Story: Observability stack in Compose (Prometheus, Grafana, Loki, Promtail, Tempo)
-  Story: Metrics instrumentation and /metrics endpoints (Micrometer JVM/app metrics)
-  Story: Structured JSON logs and correlation IDs (Logback JSON)
-  Story: OpenTelemetry tracing propagation (headers, spans across layers)
-  Story: Baseline dashboards and alerts (service health, latency, error rate)

## Epic: CI/CD and Quality Gates **[Sprint 1]**

-  Story: GitHub Actions multi-workflow (Java/TS build/test, artifacts, cache)
-  Story: CodeQL (Java + TS) and dependency security scanning
-  Story: Coverage thresholds and reports (≥80% for core modules)
-  Story: Release versioning and changelogs (conventional commits, notes)
-  **Story: CI deployment to staging environment** **[Sprint 1 DoD]**

## Epic: Data and Storage **[Sprint 1]**

-  Story: PostgreSQL per-service setup (instances, roles, pooling)
-  Story: Flyway migrations per service (baseline V1__, checksums, startup execution)
-  Story: Protobuf and Buf conventions (lint, breaking checks, CI gates)
-  Story: MinIO SSE and lifecycle rules (buckets, encryption config)

## Epic: Document Management (On-ledger metadata, off-ledger storage) **[Sprint 2]**

-  Story: Document domain model and on-ledger metadata (hash, version, ACL ref)
-  Story: Off-ledger encrypted storage (MinIO SSE) with upload/download APIs
-  Story: Versioning and audit logs (history, access logs)
-  Story: Smart search baseline and tagging hooks (metadata, OCR stubs)
-  Story: E-signature integration stub (provider interface, mock flow)

## Epic: Capital Calls (Class-Specific) **[Sprint 2]**

-  Story: Capital call domain model (class parameters, schedules, rules)
-  Story: Capital call smart contract templates (deterministic notice generation)
-  Story: Capital call lifecycle flow (create, notify, per-LP/class tracking, reminders)
-  Story: Fiat/crypto payment tracking adapter (provider interface, on-chain logs stub)
-  Story: KYC/AML integration stub (provider adapter, approval hook)

## Epic: Frontend/BFF Foundation **[Sprint 2]**

-  Story: React app scaffolding with Apollo Client (routing, state, auth guards)
-  Story: GraphQL schemas and BFF resolvers (schema-first, federation directives)
-  Story: UI component library and theming (tokens, accessibility)
-  Story: UI RBAC guards (route and component-level visibility)

## Epic: Deal State and Basic Flows **[Sprint 2]**

-  Story: DealState contract and state (rules, participants, transitions)
-  Story: Basic deal creation flow (init state, participants, notarization)
-  Story: Participant invitation/onboarding flow (identity mapping, role assignment)
-  Story: Unit tests for DealState and flows (contract/flow tests)
-  Story: End-to-end deal creation test (PRD 4.1) and onboarding test (PRD 4.2)

## Epic: Domain Model Definition **[Sprint 2]**

-  Story: Define core domain models (Deal, Party, Roles, Milestones)
-  Story: Persistence mappings and repositories (JPA, transactional boundaries)
-  Story: Seed sample data for local development (migrations or bootstrap scripts)
-  Story: Validation rules and domain invariants (annotations, business checks)

## Epic: Workflow Automation (Per Class) **[Sprint 3]**

-  Story: Workflow engine v1 (approvals, reminders, escalations)
-  Story: Capital call approval flow (class-based approvers, SLAs)
-  Story: Document approval flow (multi-step, role-aware)
-  Story: AI-assisted routing stub (scoring interface, logging)

## Epic: Waterfalls (Multi-Class EU/US) **[Sprint 3]**

-  Story: Waterfall engine foundation (deterministic BigDecimal math utilities)
-  Story: European model (pref return, catch-up, carry) with class parameters
-  Story: American model (deal-by-deal, clawback) with class logic
-  Story: Model switching and inter-class priority (configurable rules)
-  Story: Validation harness and golden test vectors (scenarios, expected outputs)

## Epic: Analytics and Reporting (Initial) **[Sprint 3]**

-  Story: Data marts for core KPIs (schemas for committed vs. deployed, portfolio breakdown)
-  Story: KPIs APIs (committed vs. deployed by class)
-  Story: Portfolio breakdown and export (PDF/Excel templates)
-  Story: Basic dashboards (UI/BFF queries, RBAC filters)

## Epic: Fund Accounting (Multi-Class GL, NAV/P&L) **[Sprint 3]**

-  Story: Chart of accounts with class dimension
-  Story: Journal posting API and validations (double-entry)
-  Story: NAV/P&L calculations (class and combined)
-  Story: Reconciliation and tests (mismatch alerts)

## Epic: Portfolio Tracking **[Sprint 3]**

-  Story: Company profile model (investments, rounds, exits, class contributions)
-  Story: Class-based investments and returns (aggregations, filters, endpoints)
-  Story: Health score baseline and risk flags (rule-based, thresholds)

## Epic: User & Investor Experience **[Sprint 3]**

-  Story: Role-based dashboards (GP, LP, IB, Analyst)
-  Story: Onboarding flows and data import (CSV/XLS templates, idempotency)
-  Story: Alerts and quick actions (notification hub, class-aware filters)
-  Story: FAQ chatbot stub (intent registry, interfaces)

## Epic: IB & M&A Workflows **[Sprint 3]**

-  Story: Pipeline and mandate tracking (stages, ownership)
-  Story: Advisory process and counterparties (milestones, contacts, permissions)
-  Story: Virtual Data Room (permissioned doc sharing, audit logs)
-  Story: Transaction execution records (negotiations, agreements, immutable logs)

## Epic: Acceptance & E2E Tests (PRD/AC mapping) **[Sprint 4]**

-  Story: PRD 4.1 Basic deal creation E2E
-  Story: PRD 4.2 Participant onboarding E2E
-  Story: AC 6.1 Network operability checks
-  Story: AC 6.2 State persistence across nodes
-  Story: PRD 4.6 Deal state transitions E2E

## Epic: Ecosystem & Integrations **[Sprint 4]**

-  Story: External API (schema-first GraphQL, federated services)
-  Story: CRM/accounting connectors (adapter interfaces, mapping stubs)
-  Story: Webhooks and eventing (outbound hooks, retries, signatures)

## Epic: Compliance & Regulatory **[Sprint 4]**

-  Story: KYC/AML baseline integration (provider adapter, audit trail)
-  Story: Automated filing scaffolding (IN/US/EU templates, export pipeline)
-  Story: DLT compliance logging policy (on-ledger vs off-ledger with hashes)

## Epic: AI Enablement (Initial) **[Sprint 4]**

-  Story: Class-filtered query API (prompt/output guardrails)
-  Story: Capital call drafting helper (templates, class params)
-  Story: OCR + classification pipeline stub (queue/worker, metadata extraction)

## Epic: ADR Formalization and Follow-Through **[Sprint 5]**

-  Story: Finalize ADR-0007 (service framework defaults, starter templates)
-  Story: Finalize ADR-0008 (PostgreSQL benchmarks, pool budgets)
-  Story: Formalize ADR-0009 (migration governance, rollback templates)
-  Story: Finalize ADR-0010 (identity config, claims map, HA)
-  Story: Formalize ADR-0011 (RBAC policy matrix, SpEL guidelines)
-  Story: Finalize ADR-0012 (Vault policies, rotation schedule)
-  Story: Finalize ADR-0013 (dashboards, SLOs, alert rules)
-  Story: Revisit triggers monitor (metrics, owners, thresholds)

## Epic: Guardrails and Principles Enforcement **[Sprint 5]**

-  Story: BigDecimal math utilities and lint rules (no binary floating)
-  Story: GraphQL schema checks in CI (lint, breaking-change gate)
-  Story: Security guardrails (no PII on-ledger checks, secret scanning, TLS)
-  Story: Observability guardrails (/metrics, JSON logs, trace coverage checks)
