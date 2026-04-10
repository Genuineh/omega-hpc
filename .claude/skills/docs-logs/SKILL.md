---
name: docs-logs
description: Development Log Manager. Manages creation, updates, and archiving of development logs for traceability.
version: 1.0.0
---

# Docs Logs Manager

Manages the creation, updates, and archiving of development logs to ensure development history is traceable.

---

## Log Directory Structure

```
docs/logs/
├── README.md                    # Log index and documentation
├── YYYY-MM-DD-feature-name.md   # Log files named by date and feature
└── archive/                     # Archive directory (optional)
    └── old-logs.md
```

---

## Log File Naming Convention

| Type | Format | Example |
|------|--------|---------|
| New Feature | `YYYY-MM-DD-feature-name.md` | `2026-03-04-user-auth.md` |
| Bug Fix | `YYYY-MM-DD-bugfix-name.md` | `2026-03-04-login-fix.md` |
| Refactor | `YYYY-MM-DD-refactor-name.md` | `2026-03-04-api-cleanup.md` |
| Daily Update | `YYYY-MM-DD-update.md` | `2026-03-04-update.md` |

---

## Log Templates

### New Log Template

```markdown
---
name: [Feature Name]
description: [Brief description]
status: in_progress
created: [YYYY-MM-DD]
author: [Agent/User]
---

## Development Log

### [Feature Name]

**Status**: In Progress / Completed / Blocked

**Started**: [YYYY-MM-DD HH:MM]
**Updated**: [YYYY-MM-DD HH:MM]
**Owner**: [Agent]

#### Objective
[Brief description of what to implement]

#### Progress
- [ ] Task 1
- [ ] Task 2
- [ ] Task 3

#### Blockers
-

#### Records
| Date | Action | Description |
|------|--------|-------------|
| YYYY-MM-DD | Created | Initialize log |
| YYYY-MM-DD | Updated | [What was done] |
```

### Daily Update Template

```markdown
### [Date] - Update

**Time**: HH:MM

**Progress**:
- [Completed] Task A
- [In Progress] Task B
- [Pending] Task C

**Issues**:
- [Describe any issues encountered]

**Next Steps**:
- Continue Task B
- Start Task C
```

---

## Usage Scenarios

### 1. Starting New Development

When Developer Agent starts new feature development:

1. Create new log file `docs/logs/YYYY-MM-DD-feature-name.md`
2. Use New Log Template
3. Update `docs/logs/README.md` index

### 2. Updating Development Progress

During development, regularly update:

1. Open corresponding log file
2. Add new progress record
3. Update task status

### 3. Completing Task

When development is complete:

1. Update status to Completed
2. Add final summary
3. Move to archive/ if needed

---

## Maintenance Rules

### Log Index Maintenance

After creating or updating logs, always update `docs/logs/README.md`:

```markdown
## Development Log Index

| Date | Feature | Status | Owner |
|------|---------|--------|-------|
| 2026-03-04 | User Auth | In Progress | Developer |
| 2026-03-03 | API Refactor | Completed | Developer |
```

### Archiving Rules

- Completed logs older than 30 days can be moved to archive/
- Perform archive cleanup at end of each month
- Ensure all important information is recorded before archiving

### Cleanup Rules

- Remove duplicate log files
- Merge related short-term logs
- Delete obsolete temporary records

---

## Validation Checklist

When creating or updating logs, ensure:

- [ ] File name follows naming convention
- [ ] Contains required frontmatter
- [ ] Status field is correct (in_progress / completed / blocked)
- [ ] Updated timestamp is current
- [ ] README.md index is synced

---

## Command Reference

```bash
# View current logs
ls -la docs/logs/

# View specific log
cat docs/logs/YYYY-MM-DD-feature-name.md

# Update README index
cat docs/logs/README.md
```
