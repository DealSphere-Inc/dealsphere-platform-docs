# Acceptance Criteria Validation — Phase 1 PRD

## Method
- For each PRD section 6 criterion, list validating test cases, datasets, and pass/fail rules.

## Criteria and validation

### Permissions: "LPs see only their class; adjustable without redeploy"
- Tests: TC-SEC-001, TC-SEC-002, TC-SEC-004
- Data: LP(A)/LP(B), membership change scenario
- Pass: Unauthorized access denied; membership change reflected in auth decisions

### Documents: "Versioning works; access logs accurate; hash verification consistent"
- Tests: TC-DOC-002, TC-DOC-005, TC-DOC-004
- Data: One doc per class; v1/v2 uploads; client hash
- Pass: Version history correct; logs complete; hash matches; tamper detected

### Capital Calls: "Issued per class rules; LP payment statuses update correctly"
- Tests: TC-CALL-001, TC-CALL-002, TC-CALL-003
- Data: Class A 10%, Class B 5%; 2 LPs each
- Pass: Obligations computed per rules; statuses roll up

### Waterfalls: "Distributions per class match vectors; switching preserves class logic"
- Tests: TC-WF-001, TC-WF-003
- Data: Test vector JSON; priority toggle
- Pass: Within specified tolerance vs vectors; switching maintains rules

### Workflows: "Class A and B flows run concurrently; reminders as configured"
- Tests: TC-WFLO-003, TC-WFLO-002
- Pass: No cross-impact; reminders/escalations fire as configured

### Analytics: "Reports filterable by class; exports work"
- Tests: TC-ANL-001, TC-ANL-003
- Pass: Filters correct; export totals match UI; scope restricted

### AI: "Filters/outputs adhere to class constraints; drafts match class parameters"
- Tests: TC-AI-001, TC-AI-002, TC-DOC-006
- Pass: No leakage across classes; draft uses class rule params; OCR class assignment correct

### Accounting: "NAV computed separately by class and combined"
- Tests: TC-ACCT-002
- Pass: NAV per class and combined computed correctly on target dates

### Portfolio: "Class-specific contributions and returns displayed"
- Tests: TC-PORT-001, TC-PORT-002
- Pass: Contributions/returns per class visible to authorized roles only

## Execution and evidence
- Record actuals (inputs/outputs), screenshots, exports, and audit log entries for each criterion
- Store waterfall vectors and result diffs under docs/qa/data
- Mark each criterion Passed/Failed with links to evidence commits or artifacts
