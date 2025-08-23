# Architecture Decision Records (ADRs)

This directory contains Architecture Decision Records for the DealSphere platform. ADRs document important architectural decisions made during platform development.

## Index

| ADR | Title | Status | Tags | Date |
|---|---|---|---|---|


## ADR Process

### Status Definitions

- **Accepted**: Decision has been finalized and is in actual use
- **Open**: Decision is under consideration or deferred; current choice documented with alternatives and revisit triggers
- **Deprecated**: Decision is no longer applicable
- **Superseded**: Decision has been replaced by a newer ADR

### Creating ADRs

#### Step-by-Step Procedure

1. **Copy template**: Copy `docs/adr/ADR-0000-template.md` to new file `ADR-XXXX-<kebab-short-title>.md`
2. **Fill required sections**: Complete all required sections, maintaining **bolding conventions** from template
3. **Format Related ADRs**: Use number-only link text followed by short title (e.g., `[0001](ADR-0001-java-17-runtime.md) Java 17 Runtime`)
4. **Add to ADR index**: Update this README.md index table if applicable
5. **Open PR**: Submit pull request with completed ADR for review

#### General Guidelines

1. Use chronological numbering (ADR-XXXX)
2. Include relevant tags for easy lookup
3. For Open ADRs, document:
   - Current choice ("for now" selection)
   - Deferred alternative(s)
   - Revisit trigger conditions
   - Target sprint for formal decision
   - Guardrails for keeping choices reversible

### Tags

Common tags for categorization:
- `architecture`: Overall system architecture decisions
- `database`: Data storage and persistence
- `api`: API design and protocols
- `security`: Security-related decisions
- `performance`: Performance optimization
- `monitoring`: Observability and monitoring
- `deployment`: Deployment and infrastructure
- `integration`: System integration patterns
