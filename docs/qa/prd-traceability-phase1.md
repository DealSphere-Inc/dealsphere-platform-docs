# PRD Traceability Matrix — Phase 1

## Purpose

- Bidirectional mapping between PRD requirements (Sections 4.1–4.10 and 6) and test cases.
- Aligns tests to PRD sprints (Section 9) based on confirmed sprint plan and test demarcation logic.
- Maps PRD §4.1–4.10 and AC §6.x to existing test IDs and their planned sprint(s).

## Matrix

| PRD Reference           | Requirement Feature                                          | Test Case IDs                                  | Sprint |
|-------------------------|-------------------------------------------------------------|------------------------------------------------|--------|
| 4.1 Platform & Security | RBAC/class segregation                                      | TC-SEC-001, TC-SEC-002                        | 1      |
|                         | On-ledger access reflection                                | TC-SEC-004                                     | 1      |
|                         | Security hardening                                         | TC-SEC-003                                     | 2      |
| 4.2 Document Mgmt       | On-ledger metadata, encrypted storage                       | TC-DOC-001                                     | 2      |
|                         | Versioning lifecycle                                       | TC-DOC-002                                     | 2      |
|                         | Access control/logs                                        | TC-DOC-LOG-001                                 | 2      |
| 4.3 Capital Calls       | Rules/issuance                                              | TC-CALL-001, TC-CALL-002                      | 4      |
|                         | Payments/status/reminders                                  | TC-NOTIF-001, TC-NOTIF-002                    | 4      |
| 4.4 Waterfalls          | European/American models                                   | TC-DEAL-001, TC-DEAL-002                      | 3      |
|                         | Priority/switching                                         | TC-WF-001                                      | 3      |
| 4.5 Workflow Automation | Approvals                                                  | TC-WF-001                                      | 3      |
|                         | Reminders/escalations                                      | TC-NOTIF-001, TC-NOTIF-002                    | 4      |
| 4.6 Analytics           | Class filters, portfolio, export                           | TC-ANL-001, TC-EXP-001                        | 5      |
| 4.7 AI Integration      | Class-filtered queries/drafts/OCR                          | TC-AI-001, TC-AI-002, TC-AI-003                | Deferred |
| 4.8 Architecture Design | Gateway segregation                                        | TC-ARCH-001, TC-ARCH-002                      | 1      |
| 4.9 Fund Accounting     | GL/NAV/P&L/visibility                                      | TC-ACCT-001, TC-ACCT-002, TC-ACCT-003          | 5      |
| 4.10 Portfolio Tracking | Profiles/metrics/visibility                                | TC-PERF-001                                    | 5      |
| DLT Infrastructure      | Audit plumbing and reference implementation               | TC-DLT-REF-001                                 | 1      |
|                         | DLT marking (if required)                                  | TC-DLT-MARK-001                                | 4      |
| User/Company Management | User CRUD operations                                       | TC-USER-001                                    | 2      |
|                         | Company management                                         | TC-COMP-001                                    | 2      |
| Dashboard & UI          | User dashboards                                            | TC-DASH-USER-001                               | 3      |
|                         | Company dashboards                                         | TC-DASH-CO-001                                 | 3      |
| Performance & Caching   | Cache optimization (if Redis used)                        | TC-CACHE-OPT-001                               | 5      |
| E2E & Validation        | End-to-end testing                                         | TC-E2E-001                                     | 6      |
|                         | Security hardening validation                              | TC-SEC-HARD-001                                | 6      |
|                         | Disaster Recovery                                          | TC-DR-001                                      | 6      |

### PRD Section 6 Acceptance Criteria Sprint Mapping

| Acceptance Criteria                                                    | Test Cases                                         | Sprint |
|----------------------------------------------------------------------- |----------------------------------------------------|--------|
| Permissions: LPs see only their class; adjustable without redeploy    | TC-SEC-001, TC-SEC-002, TC-SEC-004                | 1, 2   |
| Documents: Versioning works; access logs accurate; hash consistent    | TC-DOC-002, TC-DOC-LOG-001, TC-DOC-004           | 2      |
| Capital Calls: Issued per class; payment statuses update             | TC-CALL-001, TC-CALL-002, TC-NOTIF-001           | 4      |
| Waterfalls: Distributions match vectors; switching preserves logic   | TC-DEAL-001, TC-WF-001                           | 3      |
| Workflows: Class A & B flows run concurrently; reminders as configured| TC-WF-001, TC-NOTIF-002                         | 3, 4   |
| Analytics: Reports filterable by class; exports work                  | TC-ANL-001, TC-EXP-001                           | 5      |
| AI: Outputs/drafts adhere to class constraints                        | TC-AI-001, TC-AI-002, TC-DOC-006                | Deferred|
| Accounting: NAV by class and combined                                 | TC-ACCT-002                                      | 5      |
| Portfolio: Class contributions and returns displayed                  | TC-PERF-001                                      | 5      |

### Sprint Alignment Summary (Confirmed Sprint Plan)

**Sprint 1: Security/RBAC core and DLT audit plumbing**
- Test Cases: TC-SEC-001, TC-SEC-002, TC-SEC-LOG-001, TC-DLT-REF-001
- PRD Coverage: 4.1 (RBAC core), 4.8 (Architecture), DLT infrastructure

**Sprint 2: Documents, User/Company CRUD, resolver enforcement**
- Test Cases: TC-DOC-001, TC-DOC-002, TC-DOC-LOG-001, TC-USER-001, TC-COMP-001, TC-SEC-003
- PRD Coverage: 4.2 (Documents), User/Company management, Security enforcement

**Sprint 3: Deal lifecycle, workflow basics, dashboards**
- Test Cases: TC-DEAL-001, TC-DEAL-002, TC-WF-001, TC-DASH-USER-001, TC-DASH-CO-001
- PRD Coverage: 4.4 (Waterfalls), 4.5 (Workflow basics), Dashboard UI

**Sprint 4: Capital calls MVP, notifications/reminders**
- Test Cases: TC-CALL-001, TC-CALL-002, TC-NOTIF-001, TC-NOTIF-002, TC-DLT-MARK-001 (if required)
- PRD Coverage: 4.3 (Capital Calls), Notification system

**Sprint 5: Analytics/portfolio, exports, performance basics**
- Test Cases: TC-ANL-001, TC-EXP-001, TC-PERF-001, TC-CACHE-OPT-001 (if Redis used)
- PRD Coverage: 4.6 (Analytics), 4.9 (Fund Accounting), 4.10 (Portfolio), Performance optimization

**Sprint 6: Full AC validation, E2E, security hardening, DR/backups**
- Test Cases: TC-E2E-001, TC-SEC-HARD-001, TC-DR-001, crosswalk closure validation
- PRD Coverage: End-to-end validation, Security hardening, Disaster Recovery

**Deferred/Post-Phase 1:**
- 4.7 AI Integration: TC-AI-001, TC-AI-002, TC-AI-003
- Advanced AI features requiring additional infrastructure

## Change Control

- Update this matrix within 24 hours of any PRD change.
- Reflect added/modified tests and sprint coverage immediately.
- Highlight any PRD items not covered by functional tests for future action.

## Notes

- Sprint mapping covers PRD §4.1–4.10 and AC §6.x as confirmed in sprint plan
- Test case alignment follows DLT-first approach with 6 sprint structure
- All acceptance criteria have corresponding test validation except deferred AI features
- E2E testing in Sprint 6 validates cross-sprint integration and final acceptance criteria
