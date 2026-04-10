---
name: docs-prds
description: PRD (Product Requirements Document) Writer. Guides creation of product requirement documents for features and projects. Use when creating new feature proposals or plans.
version: 1.0.0
---

# Docs-PRDs Skill

Skill for creating Product Requirements Documents that clearly define what to build and why.

## Purpose

PRD documents the feature or project from a product perspective:
- What problem are we solving?
- Who is the user?
- What exactly should be built?
- What are the success criteria?

## When to Use

- Creating new feature proposals
- Planning significant changes
- Defining new projects
- Documenting product requirements

---

## PRD Structure

### Required Sections

```markdown
# [Feature Name]

## Summary
[One paragraph describing the feature]

## Problem
[What problem does this solve?]

## Users
[Who is this for?]

## Requirements
### Must Have
- [Requirement 1]
- [Requirement 2]

### Should Have
- [Requirement 1]
- [Requirement 2]

### Nice to Have
- [Requirement 1]

## User Stories
- As a [user], I want to [action] so that [benefit]

## Success Metrics
- [Metric 1]
- [Metric 2]

## Timeline
- [Phase 1]: [Date]
- [Phase 2]: [Date]

## Open Questions
- [Question 1]
- [Question 2]
```

---

## Section Guidelines

### Summary

- 1-2 sentences
- What and why
- Clear value proposition

### Problem

- Describe the pain point
- Quantify if possible
- Current workaround if any

### Users

- Define target audience
- User personas if relevant
- User expertise level

### Requirements

**Must Have (P0)**
- Core functionality
- Without this, feature is useless

**Should Have (P1)**
- Important but not critical
- Can ship without

**Nice to Have (P2)**
- Enhancement
- Can defer to future

### User Stories

Format: As a [user], I want to [action], so that [benefit]

Example:
- As a developer, I want to run tests with one command, so that I can verify my changes quickly

### Success Metrics

- Measurable outcomes
- KPIs for the feature
- How to measure adoption

### Timeline

- Milestones
- Key dates
- Dependencies

---

## Writing Tips

### DO

- Be specific about requirements
- Focus on outcomes, not implementation
- Include edge cases
- Define acceptance criteria
- Link to related docs

### DO NOT

- Write implementation details
- Use vague language
- Skip user stories
- Ignore error cases

---

## Frontmatter (Required)

```yaml
---
status: draft
owner: [name or team]
created: YYYY-MM-DD
updated: YYYY-MM-DD
related_issue: [issue-number]
version: 1.0
---
```

---

## Acceptance Criteria

Always include specific acceptance criteria:

```markdown
### Acceptance Criteria

- [ ] User can [action]
- [ ] System returns [expected result]
- [ ] Error message shows when [condition]
- [ ] Performance is under [threshold]
```

---

## Review Checklist

Before finalizing PRD:

- [ ] Problem clearly stated
- [ ] Target users defined
- [ ] All requirements listed with priority
- [ ] User stories cover main scenarios
- [ ] Success metrics defined
- [ ] Acceptance criteria specific and testable
- [ ] Open questions identified
- [ ] Related docs linked

---

## Example

```markdown
# User Authentication System

## Summary
Implement a secure authentication system allowing users to sign up, log in, and manage their accounts.

## Problem
Users currently have no way to create accounts or log in to the platform. All access is anonymous.

## Users
- End users who need account access
- Admin users who manage users

## Requirements
### Must Have
- User registration with email
- Login with email/password
- Password reset flow
- Logout functionality

### Should Have
- Remember me checkbox
- Session management

### Nice to Have
- Social login (Google, GitHub)
- Two-factor authentication

## User Stories
- As a new user, I want to register with my email so that I can access my account
- As a returning user, I want to log in so that I can access my data
- As a user, I want to reset my password so that I can recover my account

## Success Metrics
- 80% successful registration rate
- Average login time under 2 seconds
- Zero security incidents

## Timeline
- Phase 1 (MVP): 2 weeks
- Phase 2 (Enhancements): 1 week
```

---

## Self-Check / Validation

After creating PRD, run these validation commands to ensure the document meets basic requirements.

### Validation Commands

```bash
# 1. Check file exists
ls -la docs/prds/

# 2. Check required sections exist
FILE="docs/prds/feature-name.md"
rg "^# " "$FILE"                    # Title
rg "^## Summary" "$FILE"             # Summary
rg "^## Problem" "$FILE"            # Problem
rg "^## Users" "$FILE"              # Users
rg "^## Requirements" "$FILE"        # Requirements

# 3. Check frontmatter
rg "^---" "$FILE"                   # Frontmatter markers
rg "^status:" "$FILE"               # Status field

# 4. Check requirements have priorities
rg "^### Must Have" "$FILE"          # P0
rg "^### Should Have" "$FILE"        # P1
rg "^### Nice to Have" "$FILE"      # P2

# 5. Count user stories
rg "^## User Stories" "$FILE" -A 5
```

### Validation Checklist

- [ ] Title present (H1)
- [ ] Summary section exists
- [ ] Problem clearly stated
- [ ] Target users defined
- [ ] Requirements section with priorities
- [ ] User stories present
- [ ] Success metrics defined
- [ ] Frontmatter with status
- [ ] Owner defined

### Quick Validation Script

```bash
FILE="docs/prds/new-feature.md"
echo "Validating $FILE..."

# Check required sections
for section in "Summary" "Problem" "Users" "Requirements" "User Stories"; do
  rg "^## $section" "$FILE" && echo "✓ $section found"
done

# Check frontmatter
rg "^---" "$FILE" && echo "✓ Frontmatter found"
rg "^status:" "$FILE" && echo "✓ Status found"

# Word count
wc -w "$FILE"
```

### Common Issues to Fix

| Issue | Fix |
|-------|-----|
| Missing summary | Add ## Summary section |
| No user stories | Add ## User Stories with at least 3 stories |
| No priorities | Add Must Have, Should Have, Nice to Have |
| No success metrics | Add ## Success Metrics |
| Missing frontmatter | Add frontmatter with status and owner |
