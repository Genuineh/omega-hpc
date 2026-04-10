---
name: docs-adr
description: ADR (Architecture Decision Record) Writer. Guides creation and maintenance of ADRs for documenting architectural decisions. Use when documenting significant technical decisions.
version: 1.0.0
---

# ADR Writing Skill

Skill for creating and maintaining Architecture Decision Records (ADRs) that document important technical decisions.

## Purpose

ADRs (Architecture Decision Records) document significant architectural decisions:
- What was the decision?
- What were the alternatives?
- Why was this choice made?
- What are the consequences?

## When to Use

- Documenting a new architectural decision
- Making a significant technical choice
- Recording a design pattern adoption
- Justifying a technology selection
- Tracking evolution of architectural choices

---

## ADR Structure

### Required Sections

```markdown
# [Decision Title]

## Status
[Proposed | Accepted | Deprecated | Superseded]

## Context
[Description of the issue motivating this decision]

## Decision
[What was decided? What will be done?]

## Consequences
### Positive
- [Positive outcome 1]
- [Positive outcome 2]

### Negative
- [Negative outcome 1]
- [Negative outcome 2]

---

## Alternatives Considered

### Alternative 1: [Name]
**Pros**: [Pros]
**Cons**: [Cons]
**Why Rejected**: [Reason]

### Alternative 2: [Name]
**Pros**: [Pros]
**Cons**: [Cons]
**Why Rejected**: [Reason]

## Notes
[Any additional notes, links, or references]
```

---

## When to Create an ADR

| Trigger | Example |
|---------|---------|
| New technology adoption | Choosing React over Vue |
| Architecture pattern | Implementing microservices |
| Database selection | PostgreSQL vs MongoDB |
| API design choice | REST vs GraphQL |
| Infrastructure decision | Kubernetes vs serverless |
| Significant refactoring | Moving to event-driven |

---

## ADR Metadata (Required)

```yaml
---
adr_number: 001
date: 2024-01-15
status: accepted
author: [name]
reviewed_by: [names]
---
```

### Numbering Convention

- Use sequential numbers: 001, 002, 003...
- Never reuse numbers
- Keep a master index of all ADRs

---

## Status Meanings

| Status | Meaning |
|--------|---------|
| `proposed` | Under consideration |
| `accepted` | Approved and implemented |
| `deprecated` | No longer recommended |
| `superseded` | Replaced by another ADR |
| `rejected` | Decision not to proceed |

---

## Best Practices

### DO

- Keep ADRs focused on single decisions
- Write in clear, concise language
- Include specific pros/cons
- Link to related ADRs
- Update status when decision changes

### DO NOT

- Make ADRs too long (1-2 pages max)
- Skip alternatives section
- Forget to update superseded ADRs
- Write vague decisions

---

## ADR Index

Maintain an index of all ADRs:

```markdown
# Architecture Decision Records Index

| ID | Title | Status | Date |
|----|-------|--------|------|
| 001 | Use PostgreSQL for primary database | accepted | 2024-01-15 |
| 002 | Implement REST API | accepted | 2024-01-20 |
| 003 | Adopt microservices | deprecated | 2024-02-01 |
```

---

## Example

```markdown
# 001: Use PostgreSQL as Primary Database

## Status
Accepted

## Context
We need to choose a primary database for the application. The system requires:
- Relational data model
- ACID compliance
- Complex queries
- High write throughput

## Decision
We will use PostgreSQL as the primary database.

## Consequences
### Positive
- Strong ACID guarantees
- Excellent query optimizer
- Rich JSON support
- Active community

### Negative
- Horizontal scaling more complex
- Higher resource requirements than NoSQL

## Alternatives Considered

### Alternative 1: MySQL
**Pros**: Familiar to team, good replication
**Cons**: Less feature-rich, older architecture
**Why Rejected**: PostgreSQL has better JSON support and CTE

### Alternative 2: MongoDB
**Pros**: Easy horizontal scaling, flexible schema
**Cons**: No ACID transactions (at the time)
**Why Rejected**: We need strong ACID guarantees

## Notes
- PostgreSQL 15+ recommended
- Consider read replicas for scaling
- Related: ADR 005 (Caching Strategy)
```

---

## Integration with Other Docs

ADRs should reference:
- PRDs (`docs-prds`) - for product requirements
- Specs (`docs-specs`) - for technical specifications
- Guides (`docs-guide`) - for implementation guides

---

## Self-Check / Validation

### Validation Commands

```bash
# 1. Check file exists
ls -la docs/decisions/

# 2. Check required sections
FILE="docs/decisions/001-database-choice.md"
rg "^# " "$FILE"                    # Title
rg "^## Status" "$FILE"              # Status
rg "^## Context" "$FILE"             # Context
rg "^## Decision" "$FILE"           # Decision
rg "^## Consequences" "$FILE"       # Consequences

# 3. Check frontmatter
rg "^---" "$FILE"                   # Frontmatter markers
rg "^status:" "$FILE"               # Status field
rg "^adr_number:" "$FILE"           # ADR number

# 4. Check for alternatives
rg "^## Alternatives" "$FILE"        # Alternatives section
```

### Validation Checklist

- [ ] Title present with ADR number
- [ ] Status field valid (proposed/accepted/deprecated/superseded/rejected)
- [ ] Context clearly describes the problem
- [ ] Decision is specific and actionable
- [ ] Consequences include both positive and negative
- [ ] At least one alternative considered
- [ ] Frontmatter with adr_number, date, author
- [ ] Linked to related ADRs

### Quick Validation Script

```bash
FILE="docs/decisions/001-new-decision.md"
echo "Validating $FILE..."

# Check required sections
for section in "Status" "Context" "Decision" "Consequences"; do
  rg "^## $section" "$FILE" && echo "✓ $section found"
done

# Check frontmatter
rg "^---" "$FILE" && echo "✓ Frontmatter found"
rg "^status:" "$FILE" && echo "✓ Status found"
rg "^adr_number:" "$FILE" && echo "✓ ADR number found"

# Check alternatives
rg "^## Alternatives" "$FILE" && echo "✓ Alternatives found"
```

### Common Issues to Fix

| Issue | Fix |
|-------|-----|
| Missing status | Add ## Status with proposed/accepted/etc. |
| No alternatives | Add ## Alternatives Considered section |
| Vague decision | Make decision specific and actionable |
| Missing consequences | Add both positive and negative consequences |
| No ADR number | Add adr_number in frontmatter |
