# Phase 1 Functional Test Cases

## Overview

This document outlines the functional test cases for DealSphere Phase 1 (MVP) based on the requirements defined in Phase1_PRD.md. These test cases cover all core functional areas with class-specific multi-tenant functionality.

**Related Documents:**
- [Phase1_PRD.md](product/Phase1_PRD.md)

---

## 1. Platform & Security

### 1.1 Role-Based Access Control (RBAC)
- **Test Case 1.1.1**: Verify Admin users can access all fund classes and system configurations
- **Test Case 1.1.2**: Verify GP users can access all fund operations within their assigned funds
- **Test Case 1.1.3**: Verify LP Class A users can only access Class A-specific data and documents
- **Test Case 1.1.4**: Verify LP Class B users can only access Class B-specific data and documents
- **Test Case 1.1.5**: Verify Auditor users can access read-only views across all classes for audit purposes
- **Test Case 1.1.6**: Verify AI Assistant can access data according to class-specific permissions for automated tasks
- **Test Case 1.1.7**: Verify permission changes are enforced without contract redeployment

### 1.2 Security & Encryption
- **Test Case 1.2.1**: Verify all data is encrypted at rest using approved encryption standards
- **Test Case 1.2.2**: Verify all data transmission is encrypted in transit
- **Test Case 1.2.3**: Verify content hash verification works for document integrity
- **Test Case 1.2.4**: Verify secure node topology prevents unauthorized access
- **Test Case 1.2.5**: Verify API gateway enforces authentication and authorization

### 1.3 R3 Corda Integration
- **Test Case 1.3.1**: Verify on-ledger access control metadata is properly maintained
- **Test Case 1.3.2**: Verify Corda node handles multi-class transactions correctly
- **Test Case 1.3.3**: Verify ledger integrity and audit trail functionality

---

## 2. Document Management

### 2.1 Document Storage & Metadata
- **Test Case 2.1.1**: Verify documents are stored with encrypted off-ledger storage
- **Test Case 2.1.2**: Verify on-ledger metadata is correctly maintained for all documents
- **Test Case 2.1.3**: Verify content hash verification detects document tampering
- **Test Case 2.1.4**: Verify class-specific document access restrictions

### 2.2 Version Control & Audit Logging
- **Test Case 2.2.1**: Verify document version control tracks all changes with timestamps
- **Test Case 2.2.2**: Verify audit logging captures all document access events
- **Test Case 2.2.3**: Verify access logs show user identity, timestamp, and action performed
- **Test Case 2.2.4**: Verify version history is maintained across document updates

### 2.3 AI-Enhanced Document Features
- **Test Case 2.3.1**: Verify smart search returns relevant documents based on class permissions
- **Test Case 2.3.2**: Verify OCR-based document classification works accurately
- **Test Case 2.3.3**: Verify AI classification respects class-based document segregation

---

## 3. Capital Calls

### 3.1 Class-Specific Capital Call Rules
- **Test Case 3.1.1**: Verify capital call rules can be configured separately for each class
- **Test Case 3.1.2**: Verify capital call percentages are applied correctly per class
- **Test Case 3.1.3**: Verify capital call schedules work independently for different classes
- **Test Case 3.1.4**: Verify smart contract templates generate class-appropriate notices

### 3.2 Payment Tracking & Status Updates
- **Test Case 3.2.1**: Verify automated payment tracking works per LP and per class
- **Test Case 3.2.2**: Verify LP payment status updates are accurate and timely
- **Test Case 3.2.3**: Verify payment status is only visible to authorized class members
- **Test Case 3.2.4**: Verify automated reminders are sent according to class-specific schedules

### 3.3 Capital Call Lifecycle
- **Test Case 3.3.1**: Verify capital call notices are generated with correct class parameters
- **Test Case 3.3.2**: Verify enforcement mechanisms work per class rules
- **Test Case 3.3.3**: Verify escalation procedures follow class-specific SLAs

---

## 4. Waterfall Calculations

### 4.1 Multi-Class European Waterfall
- **Test Case 4.1.1**: Verify European waterfall calculates whole-of-fund distributions correctly
- **Test Case 4.1.2**: Verify class-specific preferred returns are applied accurately
- **Test Case 4.1.3**: Verify catch-up calculations work correctly for each class
- **Test Case 4.1.4**: Verify carry calculations are applied per class parameters
- **Test Case 4.1.5**: Verify inter-class priority is respected in distribution calculations

### 4.2 Multi-Class American Waterfall
- **Test Case 4.2.1**: Verify American waterfall processes deal-by-deal distributions correctly
- **Test Case 4.2.2**: Verify class-specific clawback logic is applied appropriately
- **Test Case 4.2.3**: Verify deal-level distributions respect class-specific parameters
- **Test Case 4.2.4**: Verify clawback calculations are accurate per class rules

### 4.3 Waterfall Configuration & Switching
- **Test Case 4.3.1**: Verify waterfall model switching preserves class-specific logic
- **Test Case 4.3.2**: Verify configurable inter-class priority settings work correctly
- **Test Case 4.3.3**: Verify deterministic outputs match expected results for each class
- **Test Case 4.3.4**: Verify waterfall calculations can be validated against test vectors

---

## 5. Workflow Automation

### 5.1 Class-Specific Approval Workflows
- **Test Case 5.1.1**: Verify approval workflows for capital calls work per class
- **Test Case 5.1.2**: Verify distribution approval workflows respect class boundaries
- **Test Case 5.1.3**: Verify document approval workflows are class-segregated
- **Test Case 5.1.4**: Verify Class A and Class B workflows run concurrently without conflict

### 5.2 SLAs, Reminders & Escalations
- **Test Case 5.2.1**: Verify class-specific SLAs are enforced correctly
- **Test Case 5.2.2**: Verify reminder schedules work according to class configuration
- **Test Case 5.2.3**: Verify escalation procedures follow class-specific rules
- **Test Case 5.2.4**: Verify reminders are sent as configured for each class

### 5.3 AI-Assisted Workflow Routing
- **Test Case 5.3.1**: Verify AI-assisted routing reduces approval latency
- **Test Case 5.3.2**: Verify AI routing respects class-based permissions
- **Test Case 5.3.3**: Verify intelligent workflow optimization works across classes

---

## 6. Basic Analytics

### 6.1 Class-Level Capital Analytics
- **Test Case 6.1.1**: Verify committed vs. deployed capital is tracked separately by class
- **Test Case 6.1.2**: Verify capital utilization reports are class-specific
- **Test Case 6.1.3**: Verify analytics data is only accessible to authorized class members

### 6.2 Portfolio Analytics
- **Test Case 6.2.1**: Verify portfolio breakdown shows class-specific contributions
- **Test Case 6.2.2**: Verify class-specific return calculations are accurate
- **Test Case 6.2.3**: Verify consolidated portfolio views work for authorized users

### 6.3 Reporting & Export
- **Test Case 6.3.1**: Verify reports are filterable by class
- **Test Case 6.3.2**: Verify PDF export functionality works for class-specific reports
- **Test Case 6.3.3**: Verify Excel export functionality works for class-specific data
- **Test Case 6.3.4**: Verify exported data maintains class-based security restrictions

---

## 7. AI Integration

### 7.1 Class-Filtered Queries
- **Test Case 7.1.1**: Verify AI queries respect class-based data access restrictions
- **Test Case 7.1.2**: Verify class-specific queries return accurate filtered results (e.g., "Show returns for Class B LPs")
- **Test Case 7.1.3**: Verify AI responses maintain class data segregation

### 7.2 AI-Assisted Drafting
- **Test Case 7.2.1**: Verify AI-assisted capital call drafting uses correct class parameters
- **Test Case 7.2.2**: Verify AI-generated documents respect class-specific requirements
- **Test Case 7.2.3**: Verify AI drafts match class-specific formatting and content rules

### 7.3 AI Document Processing
- **Test Case 7.3.1**: Verify OCR + classification works with class awareness
- **Test Case 7.3.2**: Verify AI document processing respects class-based access controls
- **Test Case 7.3.3**: Verify automated classification assigns appropriate class tags

---

## 8. Architecture Design

### 8.1 Scalability & Performance
- **Test Case 8.1.1**: Verify Corda node topology supports expected transaction volume
- **Test Case 8.1.2**: Verify API gateway handles concurrent multi-class requests
- **Test Case 8.1.3**: Verify system performance under multi-class load scenarios

### 8.2 Integration & Security Model
- **Test Case 8.2.1**: Verify API gateway integrations work with external services
- **Test Case 8.2.2**: Verify strict class-based segregation is maintained across all components
- **Test Case 8.2.3**: Verify audit capabilities work across the distributed architecture

---

## 9. Fund Accounting

### 9.1 Multi-Class General Ledger
- **Test Case 9.1.1**: Verify general ledger maintains separate accounting for each class
- **Test Case 9.1.2**: Verify cross-class transactions are properly recorded
- **Test Case 9.1.3**: Verify accounting entries are auditable and traceable by class

### 9.2 NAV and P&L Calculations
- **Test Case 9.2.1**: Verify NAV is calculated separately for each class
- **Test Case 9.2.2**: Verify combined NAV calculations are accurate
- **Test Case 9.2.3**: Verify P&L calculations work correctly per class
- **Test Case 9.2.4**: Verify consolidated P&L reports are accurate

---

## 10. Portfolio Tracking

### 10.1 Class-Based Investment Tracking
- **Test Case 10.1.1**: Verify company profiles show investment history split by class
- **Test Case 10.1.2**: Verify class-specific contributions are tracked accurately
- **Test Case 10.1.3**: Verify class-based returns are calculated correctly

### 10.2 Performance Metrics
- **Test Case 10.2.1**: Verify performance metrics are calculated separately per class
- **Test Case 10.2.2**: Verify consolidated portfolio views work for authorized users
- **Test Case 10.2.3**: Verify class-specific performance reports are accurate
- **Test Case 10.2.4**: Verify portfolio tracking data respects class-based access controls

---

## Test Execution Notes

### Prerequisites
- Multi-class test fund setup with Class A and Class B LPs
- Test users configured for each role (Admin, GP, LP Class A, LP Class B, Auditor, AI Assistant)
- Sample documents and transactions for each class
- Test vectors for waterfall calculations

### Success Criteria
- All functional areas maintain strict class-based segregation
- No cross-class data leakage in any scenario
- All automated processes respect class-specific configurations
- Performance meets specified requirements under multi-class load
- All audit trails and logging function correctly

### Test Data Requirements
- Multi-class fund structures
- Class-specific user permissions
- Sample capital calls for each class
- Test waterfall scenarios with expected outputs
- Sample documents with class-specific access requirements

---

*Last Updated: August 23, 2025*
*Version: 1.0*
*Related: Phase1_PRD.md*
