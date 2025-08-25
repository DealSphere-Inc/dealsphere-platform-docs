# 12-Week MVP Delivery Plan with TDD

## Weeks 1–2: Kickoff & Core Framework

- Project repo setup (backend, frontend, Docker Compose, DB, initial CI/CD pipeline)
- Code style, linting, and testing framework baseline for TDD (JUnit/pytest, React Testing Library, etc.)
- “Hello world” API with authentication mock, and first test: “User can log in/log out”
- Docker Compose integration for all services (DB, S3 mock, backend, frontend)
- TDD: Write base unit & integration tests for login, role switching, permissions
- Setup dev/staging environments

**Deliverables:**
- Repo scaffolds, basic README
- Demo: login/logout via UI and API, initial test passing in CI

---

## Weeks 3–4: User Management & Document Foundation

- User management API and UI (create/edit user, roles/RBAC)
- TDD: CRUD and permission tests for user data (including negative scenarios)
- Document upload, storage (integrate S3 or MinIO), download, metadata
- Initial audit log model and endpoints
- TDD: Document upload/download tests, hash check, access restriction

**Deliverables:**
- User CRUD UI/API with tests
- Document upload/download UI/API, tested and RBAC-secured
- Working audit logging basics

---

## Weeks 5–6: Capital Call Flows & Core Business Logic

- Capital call model, API, and basic UI
- Email/in-app notification integration for call creation
- TDD: Capital call creation/view, notification dispatch tests
- Implement core waterfall and multi-class logic (simplest case)
- TDD: Waterfall calculation/unit allocation business logic tests

**Deliverables:**
- End-to-end: admin creates capital call, users are notified, can view in UI
- Automated tests validate same

---

## Weeks 7–8: Workflow Automation & Integrations

- Workflow engine: approvals, reminders (scheduler, status change logic)
- AI integration (mock call, initial endpoint, e.g. OpenAI)
- R3 Corda: stub/mock integration flow for transactions/data push
- TDD: Tests for workflow state transitions and notification/approval logic
- Integration test: AI flows, Corda API call, test non-200/error scenarios

**Deliverables:**
- Users see/check workflow steps for capital call, approvals, escalations
- Basic AI/document extraction suggestion demo
- Automated workflow integration tests

---

## Weeks 9–10: Accounting, Analytics, Fine-Tuning

- Fund accounting flows (NAV, ledger entry, reporting)
- Analytics dashboards: basic reporting, CSV export, downloadable tables
- TDD: Fund/accounting tests (edge cases!), data integrity checks
- UI polish: dashboard, table/grid navigation, error states

**Deliverables:**
- Different user roles see relevant analytics/reports
- Accounting calculations tested
- Visual and code-level polish

---

## Weeks 11–12: End-to-End Coverage, Hardening, Non-Functionals

- Full E2E UI automation: Cypress/Playwright journeys for key roles and core flows
- Security tests: negative/abuse, RBAC/permission bypass, audit completeness
- Load/smoke tests; test backup/restore, failover (manual/automated w/Docker Compose)
- Fix bugs, perform usability/UX feedback rounds, add missing documentation

**Deliverables:**
- All functional tests pass; smoke/integration/acceptance pipelines green
- “Walking skeleton” demo-able to outside users
- Complete markdown and minimal architect/ops docs
- Go/no-go checklist for launch

---

## Detailed Sprint Plan

# 12-Week Sprint Plan: Epics, Stories, and Deliverables

---

## **Sprint 1 (Weeks 1–2)**
**Epic:** Core Framework & Auth  
**Stories:**
- Repo, CI pipeline, Docker Compose baseline setup
- Code style rules, linting, and testing framework initialization
- User authentication (Login/Logout)
- Role/permission model setup and TDD (unit/integration) tests
- Negative/abuse authentication and permission tests
- Dev/staging environment and build artifact validation

**Deliverables:**  
- Working codebase with basic CI pipeline
- Docker Compose environment working for local/dev
- Unit/integration tests passing for auth flows
- Demo: User log in/log out from UI and backend
- "Green" check for RBAC path in test suite

---

## **Sprint 2 (Weeks 3–4)**
**Epic:** User Mgmt & Documents  
**Stories:**
- User CRUD (create/edit/delete) via API & UI, full RBAC enforcement
- Document upload/download flow, S3/MinIO storage integration
- Metadata, file versioning, and audit log features for documents
- Document access restriction (class/role/ownership)
- Negative/edge document scenario tests
- Doc AI/categorization initial test/prototype

**Deliverables:**  
- User management UI/API fully functional
- Documents can be uploaded, downloaded, and versioned
- Document access strictly enforced by RBAC
- Unit/integration tests passing for user and document features
- Demo: Document workflow and audit log in UI
- Doc AI/categorization prototype working in dev

---

## **Sprint 3 (Weeks 5–6)**
**Epic:** Capital Call & Waterfall  
**Stories:**
- Capital call create/view/update UI and API
- Notification integration (at least email, in-app as stretch)
- Waterfall model scaffolding (core algorithms, database modeling)
- Waterfall calculation/unit allocation features
- Negative/corner capital call and waterfall tests

**Deliverables:**  
- Capital call create/view/update flows demoable
- Notifications for capital call events working (at least via email)
- End-to-end positive/negative test suite for capital call and waterfall logic
- Core waterfall calculation passes PRD scenarios

---

## **Sprint 4 (Weeks 7–8)**
**Epic:** Workflow Automation & Integrations  
**Stories:**
- Approval, reminder, SLA scheduler implementation
- Workflow state transitions (approval → notification → escalation)
- AI-assisted capital call, doc query iteration (first productionized flow)
- R3 Corda stub/mock integration—full mock flows, basic error handling
- Integration and edge tests (AI, Corda, bank API)

**Deliverables:**  
- Automated workflow with SLAs/triggers can be demoed
- Integration API stubs fully mocked and tested for main paths
- End-to-end workflow tested with passing artefacts and error handling
- Demo: Approval/escalation and AI flow in test/stage

---

## **Sprint 5 (Weeks 9–10)**
**Epic:** Accounting, Analytics, Portfolio  
**Stories:**
- Fund ledger design, NAV calculations, core reporting UI
- Analytics dashboards, role-based reporting, CSV export
- Portfolio view and company/investment tracking screens
- Data integrity, reporting edge tests, cross-role analytic validation

**Deliverables:**  
- Reports/dashboards generate correct results on real data
- Portfolio and accounting features pass acceptance/unit/integration tests
- CSV export works for all major grids/views
- Demo: Analytics and portfolio screens in UI

---

## **Sprint 6 (Weeks 11–12)**
**Epic:** E2E, Security, Launch-Ready  
**Stories:**
- Full end-to-end UI flow automation (Cypress/Playwright, etc.)
- Security, RBAC bypass, abuse/edge case tests
- Load/smoke/restore test scripts (including backup-restore drill)
- Final non-functional coverage: performance, basic failover
- Bug fix round, polishing, go-live checks
- Documentation freeze: architecture, release, deployment

**Deliverables:**  
- All functional, integration, and security test suites are “green”
- Load/smoke/failover/restore tests pass
- Launch checklist complete (prod/deployment readiness)
- Final demo: full user persona workflow E2E
- Documentation published and up to date

---

**Tips:**
- Each week: deliver 1-2 demo-able user flows.
- Retros and plan adjust weekly; track test coverage and build status in CI.
- Prefer vertical slice delivery—working functionality over isolated modules.
