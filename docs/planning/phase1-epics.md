| Epic (Weeks)                    | Story / Deliverable                                     | Linked Functional Test Cases                |
|----------------------------------|--------------------------------------------------------|---------------------------------------------|
| **Core Framework & Auth (1–2)**      | Repo, CI pipeline, Docker Compose setup                | --                                          |
|                                  | User authentication (Login/Logout)                     | 1.1.1, 1.1.2, 1.1.5, 1.2.5                  |
|                                  | Role/permission model setup, TDD tests                 | 1.1.3, 1.1.6, 1.1.7                        |
|                                  | Basic negative/abuse auth tests                        | 1.2.1, 1.2.2, 1.2.3                        |
| **User Mgmt & Documents (3–4)**      | User CRUD (API & UI), RBAC enforcement                 | 1.1.x, 1.3.2                               |
|                                  | Document upload/download, S3/MinIO integration         | 2.1.1, 2.1.2, 2.1.4                        |
|                                  | Document metadata, versioning, audit log               | 2.1.3, 2.2.1–2.2.4                         |
|                                  | Doc access restriction tests                           | 2.1.4                                      |
|                                  | Basic doc AI/categorization test stub                  | 2.3.1–2.3.3                                |
| **Capital Call & Waterfall (5–6)**   | Capital call create/view/update                        | 3.1.1–3.1.4, 3.2.1–3.2.3                   |
|                                  | Notification integration (email/in-app)                | 3.1.3, 3.2.3                               |
|                                  | Waterfall model scaffolding                            | 4.1.1–4.1.3, 4.2.1–4.2.3                   |
|                                  | Waterfall calculation, unit allocation                 | 4.3.1–4.3.4                                |
|                                  | Capital call/Waterfall negative/edge case tests        | 3.3.1–3.3.3, 4.2.4                         |
| **Workflow Automation & Integrations (7–8)** | Approval, reminder, SLA scheduler               | 5.1.1–5.1.3, 5.2.1–5.2.2                   |
|                                  | AI-assisted capital call, doc query iteration          | 7.1.1–7.1.3, 2.3.1–2.3.3                   |
|                                  | R3 Corda stub integration                             | 1.3.1–1.3.3                                |
|                                  | Integration/edge tests for AI/Corda                    | 8.2.1–8.2.3                                |
| **Accounting, Analytics, Portfolio (9–10)** | Fund ledger, NAV, report UI & export      | 6.1.1–6.1.4, 9.1.1–9.1.4                   |
|                                  | Analytics dashboards and role-based views              | 6.2.1–6.2.3, 10.1.1–10.1.3                 |
|                                  | Portfolio and company tracking                         | 10.2.1–10.2.3, 10.3.1                      |
| **E2E, Security, Launch-Ready (11–12)** | End-to-end UI flow automation                   | ALL major functional test cases             |
|                                  | Security/negative tests (abuse, failover, RBAC)        | 1.2.4, 1.3.3, 8.1.1–8.1.3                  |
|                                  | Load/smoke/restore tests, backup-restore               | 8.3.1, 8.3.2, 8.3.3, 8.3.4                 |
|                                  | Final non-functional and regression suite              | ALL                                        |
