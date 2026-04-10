---
name: docs-todo
description: TODO Document Manager. Guides creation and maintenance of TODO documents for tracking work progress. Use when creating or updating TODO.md.
version: 1.0.0
---

# Docs-Todo Skill

Skill for creating and maintaining TODO documents that track work progress effectively.

## Purpose

TODO.md serves as the single source of truth for:
- Current work priorities
- Task status tracking
- Progress visibility
- Work coordination

## When to Use

- Creating new TODO.md
- Adding new tasks
- Updating task status
- Completing tasks
- Reviewing priorities

---

## TODO Structure

### Required Sections

```markdown
# TODO

## Current Priorities

[High-priority items in progress]

## Backlog

[Lower-priority items]

## Completed

[Finished items]

---

## Active Tasks

### [Task Name]
- **Status**: [In Progress]
- **Started**: YYYY-MM-DD
- **Priority**: [High/Medium/Low]
- **Owner**: [Name or Agent]
- **Description**: [Brief description]
- **Related**: [Issue #, PR #, Doc]

### [Task Name]
- **Status**: [Pending]
- **Priority**: [High/Medium/Low]
- **Description**: [Brief description]
```

---

## Task Status Values

| Status | Meaning |
|--------|---------|
| `Pending` | Not started, waiting |
| `In Progress` | Currently being worked on |
| `Blocked` | Stuck, needs help |
| `Review` | Waiting for review |
| `Completed` | Done |
| `Replaced` | Superseded by another task |

---

## Task Creation

### When Adding New Task

```markdown
### [Feature/Bug Name]
- **Status**: [Pending/In Progress]
- **Started**: YYYY-MM-DD (if in progress)
- **Priority**: [High/Medium/Low]
- **Owner**: [Name or Agent]
- **Description**: [What needs to be done - 1-2 sentences]
- **Related**: [Issue #, PR #, Doc path]
- **Blocked by**: [Task name if blocked]
- **Blocks**: [Task name if blocking others]
```

### Priority Guidelines

| Priority | When to Use |
|----------|-------------|
| `High` | Critical path, blocking others, deadline |
| `Medium` | Normal work items |
| `Low` | Nice to have, can wait |

---

## Updating Tasks

### When Starting Task

```markdown
### [Task Name]
- **Status**: In Progress
- **Started**: YYYY-MM-DD
```

### When Blocking

```markdown
### [Task Name]
- **Status**: Blocked
- **Blocked by**: [Issue/Reason]
- **Notes**: [What is needed to unblock]
```

### When Completing

Move to Completed section:

```markdown
## Completed

### [Task Name]
- **Status**: Completed
- **Completed**: YYYY-MM-DD
- **Summary**: [What was done]
- **Related**: [Issue #, PR #]
```

---

## Best Practices

### DO

- Keep TODO current - update at end of each session
- Use clear, actionable task names
- Include related Issue/PR links
- Set realistic priorities
- Break large tasks into smaller ones

### DO NOT

- Leave tasks in "In Progress" for days without update
- Create vague tasks like "work on X"
- Forget to link related documents
- Keep completed tasks in active section

---

## Review Checklist

Before marking task complete:

- [ ] Implementation done
- [ ] Tests written and passing
- [ ] Documentation updated
- [ ] Related Issue/PR linked
- [ ] Status changed to Completed

---

## Common Patterns

### Feature Development

```markdown
### Add User Authentication
- **Status**: In Progress
- **Priority**: High
- **Description**: Implement JWT-based auth for API endpoints
- **Related**: Issue #12, docs/prds/auth-design.md
- **Subtasks**:
  - [x] Design auth flow
  - [ ] Implement login endpoint
  - [ ] Add token refresh
  - [ ] Write tests
```

### Bug Fix

```markdown
### Fix Login Redirect Bug
- **Status**: Completed
- **Priority**: High
- **Description**: Users redirected to wrong page after login
- **Related**: Issue #45, PR #78
- **Root Cause**: Incorrect redirect URL in auth service
- **Solution**: Updated redirect logic to use user.role
```

---

## Important Rules

1. **One TODO per project** - Do not create multiple TODO files
2. **Update daily** - Refresh status at end of each work session
3. **Link everything** - Connect tasks to issues, PRs, and docs
4. **Be specific** - Task names should clearly describe what to do
5. **Complete the loop** - Move completed tasks to Completed section

---

## Automation Tips

Use scripts to help maintain TODO:

```bash
# Show tasks by status
rg "Status.*In Progress" docs/TODO.md

# Count pending tasks
rg -c "Status.*Pending" docs/TODO.md

# Find blocked tasks
rg "Status.*Blocked" docs/TODO.md
```

---

## Self-Check / Validation

After creating or updating TODO.md, run these validation commands to ensure the document meets basic requirements.

### Validation Commands

```bash
# 1. Check document exists and is readable
wc -l docs/TODO.md
head -20 docs/TODO.md

# 2. Check required sections exist
rg "^## " docs/TODO.md

# 3. Check for valid status values
rg "Status.*:" docs/TODO.md

# 4. Find stale tasks (not updated in 7+ days)
# Look for old dates
rg "Started.*202" docs/TODO.md

# 5. Count tasks by status
rg -c "Status.*Pending" docs/TODO.md
rg -c "Status.*In Progress" docs/TODO.md
rg -c "Status.*Completed" docs/TODO.md
```

### Validation Checklist

- [ ] Document exists at docs/TODO.md
- [ ] Required sections present (Current Priorities, Backlog, Completed)
- [ ] All tasks have valid status
- [ ] In-progress tasks have start dates
- [ ] Completed tasks have completion dates
- [ ] No tasks left stale (In Progress > 3 days without update)

### Quick Validation Script

```bash
FILE="docs/TODO.md"
echo "Validating $FILE..."

# Check file exists
test -f "$FILE" && echo "✓ File exists"

# Check sections
rg "^## " "$FILE" && echo "✓ Sections found"

# Check for tasks
rg "^### " "$FILE" | wc -l && echo "✓ Tasks found"

# Check status values
rg "Status.*:" "$FILE"
```

### Common Issues to Fix

| Issue | Fix |
|-------|-----|
| Missing sections | Add Current Priorities, Backlog, Completed |
| No tasks | Add at least one task |
| Stale tasks | Update status or move to completed |
| Missing dates | Add Started/Completed dates |
| Invalid status | Use: Pending, In Progress, Blocked, Review, Completed, Replaced |
