# Phase 1 Stakeholder Summary (MVP, Months 1–3)

## What we're delivering
- Multi-class foundations: class-based permissions, views, and governance
- Documents: on-ledger metadata anchors; encrypted off-ledger storage; versions; audit logs
- Capital calls: class-specific rules, smart contract notices, per-LP/class tracking, reminders
- Waterfalls: EU/US with class-specific terms, clawbacks, inter-class prioritization
- Workflows: class approvals, SLAs, reminders; AI-assisted routing (feature-flagged)
- Analytics and reporting: committed vs deployed by class; portfolio breakdown; export PDF/Excel
- Fund accounting: multi-class GL; NAV/P&L per class and combined
- Portfolio tracking: class contributions and returns; company profiles
- AI initial: class-filtered queries; capital call drafting; OCR/classification
- Architecture: Corda topology; API gateway; class-segregated security model

## How it maps
- Roadmap Phase 1 → covered by Sprint Plan (5 sprints × 2 weeks)
- PRD Sections → traced to sprints and ADRs in sprint-plan-phase-1.md (PRD Traceability)
- ADRs 0000–0006 → architecture baselines for runtime, identity, ledger, exports, API

## Milestones and acceptance
- Month 1 gate: RBAC, Docs v1, class rules, basic workflows (Staging live)
- Month 2 gate: EU/US waterfalls, analytics, accounting (vectors pass, parity met)
- Month 3 gate: AI initial, portfolio tracking, pilot readiness (pen test fixes, backup/restore)

## Risks and controls
- Multi-class complexity → test vectors/harness, segregation tests from Sprint 1
- Security posture → on-ledger audit, TLS/mTLS, pen test before Sprint 5 demo
- Scope creep → milestone gates and feature flags; strict change control

## Links
- Roadmap: docs/roadmap.md (placeholder or add as needed)
- PRD: docs/product/phase-1-prd.md (placeholder or add as needed)
- Sprint Plan: docs/tech/sprint-plan-phase-1.md
- Technical Plan: docs/tech/phase-1-technical-plan.md
- ADRs: docs/adr
