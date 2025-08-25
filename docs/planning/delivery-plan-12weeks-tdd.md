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

**Tips:**
- Each week: deliver 1-2 demo-able user flows.
- Retros and plan adjust weekly; track test coverage and build status in CI.
- Prefer vertical slice delivery—working functionality over isolated modules.
