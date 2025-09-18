# Phase 1 Epics: Weekly Tech Deliverables Guide

This document provides a week-by-week breakdown of all core tech deliverables for Phase 1 of DealSphere, ensuring that every business-critical capability from the PRD and all functional test cases are tracked and completed.  
Each Epic summarizes a vertical slice (major user-facing/integrated feature set) and links every deliverable to specific functional test cases.  
Use this guide as the master reference for sprint planning, tracking, and acceptance—each Story or Deliverable is “Done” when all linked test cases are passing in CI.

---

| Epic (Weeks)                    | Story / Deliverable                                     | Linked Functional Test Cases                |
|----------------------------------|--------------------------------------------------------|---------------------------------------------|
| **Core Framework & Auth (1–2)**      | Repo, CI pipeline, Docker Compose setup                | --                                          |
|                                  | User authentication (Login/Logout)                     | 1.1.1, 1.1.2, 1.1.5, 1.2.5                  |
|                                  | Role/permission model setup, TDD tests                 | 1.1.3, 1.1.4, 1.1.6, 1.1.7                 |
|                                  | Basic negative/abuse auth tests                        | 1.2.1, 1.2.2, 1.2.3                        |
| **User Mgmt & Documents (3–4)**      | User CRUD (API & UI), RBAC enforcement                 | 1.3.2                                      |
|                                  | Document upload/download, S3/MinIO integration         | 2.1.1, 2.1.2, 2.1.4                        |
|                                  | Document metadata, versioning, audit log               | 2.1.3, 2.2.1–2.2.4                         |
|                                  | Doc access restriction tests                           | 2.1.4                                      |
|                                  | Basic doc AI/categorization test stub                  | 2.3.1–2.3.3                                |
| **Capital Call & Waterfall (5–6)**   | Capital call create/view/update                        | 3.1.1–3.1.4, 3.2.1–3.2.4                   |
|                                  | Notification integration (email/in-app)                | 3.1.3, 3.2.3                               |
|                                  | Waterfall model scaffolding                            | 4.1.1–4.1.5, 4.2.1–4.2.3                   |
|                                  | Waterfall calculation, unit allocation                 | 4.3.1–4.3.4                                |
|                                  | Capital call/Waterfall negative/edge case tests        | 3.3.1–3.3.3, 4.2.4                         |
| **Workflow Automation & Integrations (7–8)** | Approval, reminder, SLA scheduler               | 5.1.1–5.1.4, 5.2.1–5.2.4, 5.3.1–5.3.3     |
|                                  | AI-assisted capital call, doc query iteration          | 7.1.1–7.1.3, 7.2.1–7.2.3, 7.3.1–7.3.3    |
|                                  | R3 Corda stub integration                             | 1.3.1–1.3.3                                |
|                                  | Integration/edge tests for AI/Corda                    | 8.2.1–8.2.3                                |
| **Accounting, Analytics, Portfolio (9–10)** | Fund ledger, NAV, report UI & export      | 6.1.1–6.1.3, 6.3.1–6.3.4, 9.1.1–9.1.3, 9.2.1–9.2.4 |
|                                  | Analytics dashboards and role-based views              | 6.2.1–6.2.3, 10.1.1–10.1.3                 |
|                                  | Portfolio and company tracking                         | 10.2.1–10.2.4                              |
| **E2E, Security, Launch-Ready (11–12)** | End-to-end UI flow automation                   | ALL major functional test cases             |
|                                  | Security/negative tests (abuse, failover, RBAC)        | 1.2.4, 8.1.1–8.1.3                         |
|                                  | Load/smoke/restore tests, backup-restore               | Performance and reliability testing          |
|                                  | Final non-functional and regression suite              | ALL                                        |

---

**How to use this document:**

- **Sprint Planning:** Assign stories/deliverables week by week, referencing test cases to ensure test-driven development and coverage.
- **Tracking Progress:** Check off epics and stories as their corresponding test cases pass—all features are “Done” only when verified by passing tests.
- **Review & Adjust:** Re-align deliverables or epics as priorities shift in the MVP build, ensuring that every functional and technical goal remains traceable.
- **Traceability:** If new features/test cases are added, extend the table and epics so the plan always aligns with current project reality.

**For questions, proposed changes, or refinement, update this doc and communicate with the DealSphere tech and QA teams.**
