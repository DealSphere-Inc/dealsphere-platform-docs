# DealSphere Phase 1 — Functional Test Cases (PRD-Aligned)

## Conventions
- Roles: Admin, GP, LP(A), LP(B), Auditor, AI Assistant (virtual)
- Classes: A and B configured with distinct rules
- PRD references use "PRD x.y" matching docs/product/Phase1_PRD.md

## Section 4.1 Platform & Security (RBAC, class segregation, audit)
- **TC-SEC-001: Role matrix enforcement (PRD 4.1)**
  - Preconditions: Users exist for each role; classes A/B configured
  - Steps: Attempt read/write actions per role across classes
  - Expected: Admin full; GP per-fund; LP only own class; Auditor read-only; AI restricted to caller's scope
  - Negative: LP(B) reading Class A docs or capital calls returns 403
- **TC-SEC-002: Class isolation in GraphQL/REST (PRD 4.1)**
  - Steps: Query lists and nested relations
  - Expected: Results filtered to user's class, totals reflect filtered set
- **TC-SEC-003: Token/session authorization (PRD 4.1)**
  - Steps: Use valid token with mismatched class scope
  - Expected: 403, audit log with actor/class/resource
- **TC-SEC-004: On-ledger access metadata reflection (PRD 4.1, 4.8)**
  - Steps: Change class membership/permissions; call API
  - Expected: Authorization decisions reflect updated on-ledger state within expected delay
- **TC-SEC-005: Audit trail completeness (PRD 4.1)**
  - Steps: Perform read/write/denied attempts
  - Expected: Logs include actor, class, resource, action, timestamp; immutable reference on-ledger

## Section 4.2 Document Management (versioning, integrity, logs)
- **TC-DOC-001: Upload with on-ledger metadata (PRD 4.2)**
  - Steps: Upload v1; verify metadata on-ledger, binary encrypted off-ledger
- **TC-DOC-002: Versioning workflow v1→v2 (PRD 4.2)**
  - Steps: Upload v2 of same document
  - Expected: Version increments; history lists v1/v2 with authors and timestamps; v1 remains accessible read-only
- **TC-DOC-003: Version access control by class/role (PRD 4.1, 4.2)**
  - Steps: LP(A) accesses A doc; LP(B) attempts access
  - Expected: LP(B) denied; GP/Admin allowed; Auditor read-only
- **TC-DOC-004: Content hash verification (PRD 4.2)**
  - Steps: Compute client hash after upload; compare to stored hash; simulate tamper
  - Expected: Hash match normally; tamper detected; alert/audit logged
- **TC-DOC-005: Access logs for view/download/version (PRD 4.2, 6)**
  - Expected: Accurate entries for each action
- **TC-DOC-006: Smart search and OCR classification (initial) (PRD 4.2, 4.7)**
  - Steps: Upload scanned PDF; search text; check class label
  - Expected: Results only within user's class; OCR assigns correct class; misclassification never leaks across classes

## Section 4.3 Capital Calls (class rules, notices, tracking)
- **TC-CALL-001: Configure class rules (PRD 4.3)**
  - Steps: Set Class A 10%, Class B 5%
  - Expected: Rules persist; reflected in issuance
- **TC-CALL-002: Issue capital call per class (PRD 4.3)**
  - Steps: Issue; verify per-LP obligations and notices by class
- **TC-CALL-003: Payment status lifecycle (PRD 4.3, 6)**
  - Steps: Mark Paid/Partial; simulate overdue
  - Expected: Per-LP statuses correct; class rollups update
- **TC-CALL-004: Reminders/escalations (PRD 4.3, 4.5)**
  - Steps: Trigger reminder schedule; verify escalation to GP on breach
- **TC-CALL-005: Guardrails and audit (PRD 4.3, 4.1)**
  - Steps: Attempt wrong-class payment/overpay
  - Expected: Rejected with audit trail

## Section 4.4 Waterfall Calculations (EU/US, priority, determinism)
- **TC-WF-001: European model per class (PRD 4.4, 6)**
  - Steps: Run with provided inputs; compare outputs to vectors
  - Expected: Pref, catch-up, carry applied per class; matches vectors
- **TC-WF-002: American model with clawback (PRD 4.4)**
  - Steps: Over-distribute then realize; verify clawback
- **TC-WF-003: Inter-class priority switching (PRD 4.4)**
  - Steps: Toggle priority; recompute
  - Expected: Ordering and outputs change as expected
- **TC-WF-004: Determinism (PRD 4.4)**
  - Steps: Run multiple times with same inputs
  - Expected: Identical outputs or flagged divergence

## Section 4.5 Workflow Automation (approvals, reminders, AI routing)
- **TC-WFLO-001: Class-specific approval chains (PRD 4.5)**
  - Steps: Require approvers for Class A/B capital calls
  - Expected: Cannot proceed without required sign-offs
- **TC-WFLO-002: Reminders/escalations (PRD 4.5)**
  - Steps: Validate reminders at intervals; escalation on breach
- **TC-WFLO-003: Concurrent class workflows isolation (PRD 4.5, 6)**
  - Steps: Run Class A and B workflows in parallel
  - Expected: No cross-impact; independent statuses
- **TC-WFLO-004: AI-assisted routing within class (PRD 4.5, 4.7)**
  - Steps: Suggested approvers list
  - Expected: Only approvers for that class; accepted suggestions produce valid routing

## Section 4.6 Basic Analytics (class filters, exports)
- **TC-ANL-001: Class-filtered committed vs deployed (PRD 4.6)**
  - Steps: Switch class filters; validate numbers
- **TC-ANL-002: Portfolio breakdown by class (PRD 4.6, 4.10)**
- **TC-ANL-003: Export PDF/Excel scoped to role/class (PRD 4.6)**
  - Expected: Exports match on-screen and contain only authorized data

## Section 4.7 AI Integration (initial)
- **TC-AI-001: Class-filtered queries (PRD 4.7)**
- **TC-AI-002: Draft capital call with class parameters (PRD 4.7)**
  - Expected: Draft requires manual approval via workflows
- **TC-AI-003: OCR + class assignment constraints (PRD 4.2, 4.7)**

## Section 4.8 Architecture Design (enforcement checks)
- **TC-ARCH-001: API gateway enforces auth and class segregation (PRD 4.8)**
- **TC-ARCH-002: No cross-tenant/class leakage in GraphQL/REST (PRD 4.8)**

## Section 4.9 Fund Accounting
- **TC-ACCT-001: Class-attributed GL entries (PRD 4.9)**
- **TC-ACCT-002: NAV per class and combined; date-based recompute (PRD 4.9, 6)**
- **TC-ACCT-003: Role-based visibility (LP class-only; GP/Admin all) (PRD 4.9)**

## Section 4.10 Portfolio Tracking
- **TC-PORT-001: Company profiles with class-split contributions/distributions (PRD 4.10)**
- **TC-PORT-002: IRR/MOIC per class and consolidated; date filters (PRD 4.10)**
- **TC-PORT-003: Role-based visibility (PRD 4.10)**

## Cross-cutting API/GraphQL functional checks
- **TC-API-001: Pagination/sorting within class scope (4.1, 4.8)**
- **TC-API-002: Mutations blocked across classes (4.1)**
- **TC-API-003: Error messages are user-friendly, non-revealing (4.1)**

## Appendix: Sample test data
- Classes: A (10% call), B (5% call)
- LPs: 2 per class with different commitments
- Deals: 2–3 investments with staged cashflows
- Waterfall vectors: Place under docs/qa/data/waterfalls.json
- Documents: 3 per class including a scanned PDF for OCR
