DealSphere Phase 1 QA Suite — PRD-Aligned Testing Framework
==========================================================

Overview
--------
This QA suite is strictly aligned to the Phase 1 PRD at docs/product/Phase1_PRD.md.
It provides full functional coverage and traceability for PRD sections 4.1–4.10 and acceptance criteria in section 6.
Execution is organized by the PRD Timeline (Sprint Breakdown) in section 9.

Scope (Phase 1)
- Platform & Security (4.1): RBAC, class segregation, audit, on-ledger access metadata
- Document Management (4.2): on-ledger metadata, encrypted storage, versioning, access logs, content hash verification, smart search and OCR classification (initial)
- Capital Calls (4.3): class rules, notices, payment tracking, statuses, reminders/escalations
- Waterfalls (4.4): European, American, inter-class priority, switching, determinism
- Workflow Automation (4.5): approvals for calls/distros/docs, class SLAs, reminders/escalations, AI-assisted routing
- Basic Analytics (4.6): class-level committed vs deployed, portfolio breakdown, export PDF/Excel
- AI Integration (4.7): class-filtered queries, drafting assist, OCR+classification with class awareness
- Architecture Design (4.8): gateway enforcement and strict segregation checks
- Fund Accounting (4.9): multi-class GL, NAV/P&L per class and combined
- Portfolio Tracking (4.10): class-split histories and metrics

What's included in this folder
- phase1-functional-test-cases.md: Complete functional test cases grouped by PRD section
- prd-traceability-phase1.md: Bidirectional matrix mapping PRD requirements to test cases and sprints
- acceptance-criteria-validation.md: How each acceptance criterion in PRD section 6 is validated (tests, data, pass/fail)

Sprint-based execution (from PRD section 9)
- Sprint 1: Platform & Security (4.1), Document metadata + versioning (4.2)
- Sprint 2: Capital Calls (4.3), basic Workflow Automation (4.5)
- Sprint 3: Waterfalls (4.4), advanced Workflow Automation (4.5)
- Sprint 4: Basic Analytics (4.6), Fund Accounting (4.9), Portfolio Tracking (4.10)
- Sprint 5: AI Integration (4.7), end-to-end integration and acceptance criteria validation

Quality gates
- 100% coverage of functional requirements (PRD 4.1–4.10)
- All acceptance criteria (PRD 6) validated with passing tests
- Class segregation verified across APIs/UI/exports
- Deterministic waterfall results matching test vectors
- Full document versioning and hash integrity verified

Execution notes
- Each test case includes: ID, Title, PRD Reference, Preconditions, Test Data, Steps, Expected Result, Negative/Edge variants, Postconditions.
- Use representative class data: GP, Admin, Auditor, LPs in Class A and Class B; at least two LPs per class; 2–3 deals; sample documents (PDF/DOCX).
- Store any test vectors for waterfalls under docs/qa/data if needed.
