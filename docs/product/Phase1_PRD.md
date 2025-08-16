# DealSphere Phase 1 (MVP) — Product Requirements Document

> **Related Documents:**
> - [Sprint Plan Phase 1](../tech/sprint-plan-phase-1.md) - Implementation timeline and sprint breakdown
> - [Phase 1 Functional Test Cases](../qa/phase1-functional-test-cases.md) - Comprehensive test scenarios
> - [Sprint→Functional Test Mapping](../qa/sprint-functional-test-mapping-phase-1.md) - Test coverage mapping

## 1. Product Overview
- Product Name: DealSphere
- Description: Secure, DLT-backed fund management platform for PE/VC with class-specific workflows and automated waterfalls, built on R3 Corda.
- Objective: Deliver an MVP that supports multi-class fund operations (permissions, capital calls, workflows, waterfalls), basic analytics, fund accounting, and portfolio tracking with AI-assisted experiences and a finalized architecture for scale.

---
## 2. Target Users
- General Partners (GP), Limited Partners (LP) by class (A, B, etc.)
- Fund Managers and Administrators
- Auditors and Compliance users
- Investment/Portfolio Analysts
- Investors (read-only class-specific views)
- AI Assistant (virtual role for automated tasks)

---
## 3. Core Use Cases
- Role-based, class-segregated access and views
- Document management with on-ledger metadata, versioning, and audit logs
- Capital call lifecycle per class (rules, notices, tracking)
- Multi-class European and American waterfalls with prioritization and clawbacks
- Class-specific workflow automation (approvals, reminders, escalations)
- Basic analytics by class (committed vs. deployed, portfolio breakdown)
- Fund accounting and NAV/P&L at class and combined levels
- Portfolio tracking with class-based contributions and returns
- AI-assisted class-specific queries and drafting

---
## 4. Feature Requirements (Phase 1)
### 4.1 Platform & Security
- R3 Corda for on-ledger access control and metadata
- Encryption at rest and in transit; content hash verification for documents
- Role-based access with class-level segregation (Admin, GP, LP by class, Auditor, AI Assistant)
- API gateway and secure node topology

### 4.2 Document Management
- On-ledger metadata; encrypted off-ledger storage
- Version control, audit logging, and access logs
- Content hash verification for integrity
- Smart search and OCR-based classification (initial AI)

### 4.3 Capital Calls
- Class-specific capital call rules (percentages, schedules)
- Smart contract templates for notices and enforcement
- Automated payment tracking per LP and per class
- Status updates per LP; automated reminders/escalations

### 4.4 Waterfall Calculations (Multi-Class)
- European: whole-of-fund with class-specific pref returns, catch-up, carry
- American: deal-by-deal with class-specific clawback logic
- Configurable inter-class priority and model switching
- Deterministic outputs per class with expected results validation

### 4.5 Workflow Automation (Per Class)
- Approvals for capital calls, distributions, and major document approvals
- Class-specific SLAs, reminder schedules, and escalations
- AI-assisted routing to reduce approval latency

### 4.6 Basic Analytics
- Class-level committed vs. deployed capital
- Portfolio breakdown and class contribution/return
- Export to PDF/Excel

### 4.7 AI Integration (Initial)
- Class-filtered queries (e.g., "Show returns for Class B LPs")
- AI-assisted capital call drafting with class parameters
- OCR + classification for documents with class awareness

### 4.8 Architecture Design
- Corda node topology for scale and security
- API gateway for integrations and external services
- Security model for strict class-based segregation and audit

### 4.9 Fund Accounting
- Multi-class general ledger
- NAV and P&L per class and combined

### 4.10 Portfolio Tracking
- Company profiles with investment history split by class
- Performance metrics per class and consolidated views

---
## 5. Non-Functional Requirements
- Mobile-responsive web app
- High security: DLT auditability, encryption, RBAC with class segregation
- GDPR-ready and regional compliance placeholders
- Cloud deployment (AWS/Azure/GCP)
- API-first for integrations and modular scale

---
## 6. Acceptance Criteria (Key)
- Permissions: LPs see only their class; adjustable without contract redeploy
- Documents: Versioning works; access logs accurate; hash verification consistent
- Capital Calls: Issued per class rules; LP payment statuses update correctly
- Waterfalls: Distributions per class match test vectors; switching preserves class logic
- Workflows: Class A and B flows run concurrently without conflict; reminders as configured
- Analytics: Reports filterable by class; exports work
- AI: Filters/outputs adhere to class constraints; drafts match class parameters
- Accounting: NAV computed separately by class and combined
- Portfolio: Class-specific contributions and returns displayed

---
## 7. Success Metrics
- Funds onboarded leveraging multi-class features
- Volume of class-specific capital calls and distributions
- Workflow automation throughput and SLA adherence
- Accuracy of waterfall outputs vs. expected vectors
- Frequency of AI-assisted actions
- Analytics/export usage by class

---
## 8. Risks & Mitigation
- Multi-class complexity: Test vectors and simulation harnesses
- Security/segregation: Pen testing, rigorous data access validation, on-ledger audit
- Scope creep: Phased deliveries with strict change control
- Compliance variance: Region placeholders and pilot validation

---
## 9. Timeline (Sprint Breakdown)
> **📋 For detailed implementation plan, see:** [Sprint Plan Phase 1](../tech/sprint-plan-phase-1.md)

- Sprint 1: Architecture, Corda setup, role-based permissions, document metadata + versioning
- Sprint 2: Capital calls (multi-class rules), basic class workflows, payment tracking
- Sprint 3: Waterfalls (EU/US) with class logic and prioritization; advanced workflows
- Sprint 4: Basic analytics, fund accounting (class NAV/P&L), portfolio tracking
- Sprint 5: AI-assisted workflows (queries, drafting, OCR), integration testing, QA/security, pilot onboarding

---
## 10. QA Documentation
> **🧪 For comprehensive testing coverage, see:** [Sprint→Functional Test Mapping](../qa/sprint-functional-test-mapping-phase-1.md)

For comprehensive quality assurance and testing information, please refer to the following documentation:

- [QA Overview](../qa/README.md) - Complete guide to our QA processes and methodologies
- [Phase 1 Functional Test Cases](../qa/phase1-functional-test-cases.md) - Detailed test scenarios for all Phase 1 features
- [PRD Traceability Matrix](../qa/prd-traceability-phase1.md) - Mapping of requirements to test cases and validation criteria
- [Acceptance Criteria Validation](../qa/acceptance-criteria-validation.md) - Framework for validating acceptance criteria and sign-off procedures

### Verification Links
- [Sprint Plan Phase 1](../tech/sprint-plan-phase-1.md) - Detailed sprint planning and timeline for Phase 1 implementation
- [Sprint→Functional Test Mapping](../qa/sprint-functional-test-mapping-phase-1.md) - Mapping of sprint deliverables to functional test coverage

---
**Summary:** Phase 1 delivers a secure, multi-class MVP across permissions, documents, capital calls, waterfalls, workflows, analytics, fund accounting, and portfolio tracking, with AI assist and an architecture ready for scale.
