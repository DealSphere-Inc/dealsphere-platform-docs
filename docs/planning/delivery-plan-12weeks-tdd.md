# 12-Week MVP Delivery Plan (TDD Approach)

This plan breaks the MVP delivery into six 2-week sprints. Each sprint is defined by an epic, a set of stories (major tasks), and explicit deliverables that must be achieved for a sprint to be considered “done.” All work must be validated by passing tests, demos, and CI/CD artefacts.

---

## Sprint 1 (Weeks 1–2): Core Framework & Auth

### Stories
- Project repo setup (backend, frontend, Docker Compose, DB, initial CI/CD pipeline)
- Code style, linting, and TDD test framework (Java/React)
- User authentication (Login/Logout)
- Role/permission model setup and TDD
- Negative/abuse authentication and permission tests
- Dev/staging environments and build artefact validation

### Deliverables
- CI pipeline running with all checks
- Docker Compose brings up all services in local/dev
- Passing unit/integration tests for authentication/RBAC
- Demo: User login/logout via UI and backend API
- "Green" RBAC/auth checks in test suite

---

## Sprint 2 (Weeks 3–4): User Mgmt & Documents

### Stories
- Complete User CRUD (API & UI) with full RBAC enforcement
- Document upload/download (S3/MinIO integration)
- Document metadata, versioning, and audit log
- Document access restrictions (class/role/ownership)
- Negative/edge document flows
- Doc AI/categorization prototype

### Deliverables
- User management UI/API running and tested
- Document workflow (upload, version, access, audit) demoable
- Document access strictly enforced by RBAC
- Unit/integration tests green for user/document modules
- Audit logs and doc AI/categorization prototype in dev
- Demoable user and document features

---

## Sprint 3 (Weeks 5–6): Capital Call & Waterfall

### Stories
- Capital call create/view/update (UI and API)
- Notification integration (email/in-app)
- Waterfall model design (algorithms, DB)
- Waterfall calculation/unit allocation features
- Negative/corner capital call and waterfall tests

### Deliverables
- Capital call workflow (from create to notification) demoable
- Waterfall logic for PRD scenarios covered and tested
- Notifications firing in all intended scenarios
- Passing end-to-end/negative tests for capital call & waterfall
- Demo of capital call + waterfall, start to finish

---

## Sprint 4 (Weeks 7–8): Workflow Automation & Integrations

### Stories
- Approval, reminders, SLA scheduler for workflow
- Workflow state transitions (approval → notification → escalation)
- AI-assisted capital call/doc query implementation
- R3 Corda integration stub/mock (main flows + error cases)
- Integration/edge tests for AI, Corda, bank API

### Deliverables
- Automated workflow demo (with SLAs/triggers)
- Integration stubs working for all major paths
- E2E, workflow & integration tests passing in CI
- Demo: approvals/escalation in test/stage

---

## Sprint 5 (Weeks 9–10): Accounting, Analytics, Portfolio

### Stories
- Fund ledger, NAV calculation, reports UI
- Analytics dashboards, role-based views, CSV export
- Portfolio/company/investment tracking screens
- Data integrity/reporting edge tests
- Cross-role analytics validation

### Deliverables
- Analytics/reporting UI shows real/test data
- Portfolio/accounting screens tested, working
- CSV export/bulk data tested for all grids
- Acceptance/unit/integration tests pass for metrics, portfolio
- Demo: analytics, reporting, and portfolio features

---

## Sprint 6 (Weeks 11–12): E2E, Security, Launch-Ready

### Stories
- Full end-to-end UI automation (Cypress/Playwright)
- Security, RBAC bypass, and abuse/edge case tests
- Load/smoke/restore test scripts & backup/restore process validation
- Performance & non-functional: failover and basic regression
- Polishing, bug fixes, final checks, and code freeze
- Freeze/publish all architecture/release/deployment docs

### Deliverables
- End-to-end automated test journeys “green” for all critical flows
- Security, load, failover, restore, and abuse tests pass
- All critical path test cases pass (functional, integration, security)
- Demo complete MVP user flows per persona
- Launch checklist and final documentation published

---

## Usage Notes

- Each sprint is “done” only when all stories are completed and deliverables demonstrably met.
- Always validate by demo and passing CI/CD tests.
- If features spill to the next sprint, update scope and priorities in this document.

_Last updated: August 2025_
