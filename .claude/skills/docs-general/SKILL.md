---
name: docs-general
description: Repository Documentation Manager. Manages documentation structure, archives old docs, and maintains documentation hygiene. Use when working with documentation or need to organize docs.
version: 1.0.0
---

# Docs-General Skill

Skill for managing repository documentation according to repository guidelines.

## Documentation Structure

Maintain this structure:

```
docs/
├── TODO.md              # Current work and priority order
├── prds/                # Plans, architecture, design details
├── guide/               # Usage guides and contributor workflows
├── specs/               # Formal specs, contracts, repository rules
├── whitepapers/         # Long-form rationale and vision
├── decisions/           # Architecture decisions (ADRs)
└── archive/             # Retired, superseded documents
```

Additional files:
- `README.md` - Top-level index
- `CHANGELOG.md` - Notable changes after shipping starts
- `LICENSE` - At repository root

## When to Use

- Creating new documentation
- Archiving old documents
- Moving or renaming documents
- Updating TODO.md after work completion
- Checking documentation structure
- Code changes that require docs updates
- Managing architecture decisions (use `docs-adr` skill)

## Related Skills

- `docs-todo` - TODO management
- `docs-prds` - Product requirements
- `docs-specs` - Technical specifications
- `docs-guide` - User guides
- `docs-adr` - Architecture decision records

---

## Common Tasks

### 1. Archive a Document

When a document is superseded, completed, or outdated:

1. Move file to `docs/archive/`
2. Add archive note at top:
   ```
   ---
   archived: true
   archived_date: YYYY-MM-DD
   replaced_by: [new document or N/A]
   reason: [why archived]
   ---
   ```
3. Update related files:
   - `README.md` - Remove/update links
   - `docs/TODO.md` - Mark completed/replaced
   - `docs/prds/` - Update references

### 2. Update TODO.md

After development work:

```markdown
### [Feature Name]
- **Status**: [In Progress/Completed/Replaced]
- **Date**: YYYY-MM-DD
- **Notes**: What was done
- **Issue**: #[issue-number]
- **PR**: [pr-link]
```

### 3. Check Documentation Health

```bash
# Review current priorities
sed -n '1,120p' docs/TODO.md

# Inspect tracked documentation
rg --files docs

# Verify doc moves and links
git diff -- README.md docs/
```

### 4. Create New Document

1. Choose correct directory based on type:
   - `prds/` - Plans and designs
   - `guide/` - Usage guides
   - `specs/` - Formal specs
   - `decisions/` - Architecture decisions
2. Use descriptive filename
3. Add to README.md if it is a main document
4. Add frontmatter with metadata

---

## Frontmatter Metadata (REQUIRED)

Every new document MUST include frontmatter:

```yaml
---
status: [draft | active | deprecated | superseded]
last_verified_commit: [commit-hash or N/A]
owner: [owner or team]
version: [v1.0]  # Optional: for versioned docs
related_issue: [issue-number]  # Optional
related_pr: [pr-number]  # Optional
---
```

### Status Meanings

| Status | Meaning |
|--------|---------|
| `draft` | Work in progress |
| `active` | Currently valid and used |
| `deprecated` | Outdated but kept for reference |
| `superseded` | Replaced by another document |

---

## Sync Check (IMPORTANT)

When code changes, check if docs need updates:

Before completing any code change, verify:

1. **Affected specs/prds**: Which documents relate to changed code?
2. **Update needed**: Do those docs still reflect current implementation?
3. **Sync update**: If implementation changed, update related docs in same commit

**Rule**: Code change + doc update = same commit

If docs are outdated:
- Mark issue in TODO.md
- Update docs before marking task complete
- Use Verifier Agent to check sync status

---

## Consolidation Process

When to consolidate:
- Feature is complete and stable
- Multiple small PRDs exist for one feature
- Fragmented PRDs need merging

Consolidation steps:

1. **Identify** all related prds for one feature
2. **Extract** core logic into specs/ or guide/
3. **Update** prds to reference the consolidated doc
4. **Archive** the fragmented prds
5. **Update** README.md index

Example:
```
Before:
docs/prds/feature-login-flow.md
docs/prds/feature-auth-api.md
docs/prds/feature-auth-db.md

After:
docs/specs/auth-system.md  (consolidated)
docs/prds/feature-auth.md  (points to specs, archived)
```

---

## Versioning Strategy

For multi-version projects:

- Use version in filename: `auth-api-v1.md`, `auth-api-v2.md`
- Or use versioned directories: `docs/v1/`, `docs/v2/`
- Mark old versions as `deprecated` in frontmatter

Version naming convention:
- Major versions: `v1/`, `v2/`
- Or filename: `feature-name-v1.md`

---

## Change Tracking (REQUIRED)

Every doc update MUST reference change reason:

Add to document footer or TODO.md:

```markdown
---

### Change Log
- YYYY-MM-DD: [Brief change description] - Issue #[n] / PR #[n]
```

Required for:
- Spec changes
- PRD modifications
- Architecture decision updates

---

## Archive Rules

Archive when:
- Document is superseded by newer version
- Plan is finished and no longer active
- Guide/draft does not match current structure
- Kept only for historical context

Archive process:
1. Move to `docs/archive/`
2. Add archive metadata at top
3. Update all references
4. Keep filename descriptive

Always update in same change:
- README.md links
- TODO.md status
- Related prds/guides

---

## Style Guidelines

- Keep docs short, direct, actionable
- Use stable filenames
- Use consistent structure
- Prefer explicit names over generic ones
- Follow project established conventions
- Include frontmatter in all docs

---

## Important Commands

```bash
# Check current priorities
sed -n '1,120p' docs/TODO.md

# List all docs
rg --files docs

# Check for broken links
git diff -- README.md docs/

# Find docs needing verification
rg "last_verified_commit: N/A" docs/
```

---

## Self-Check / Validation

After completing any docs task, run these validation commands to ensure the document meets basic requirements.

### Validation Commands

```bash
# 1. Check frontmatter exists and is valid
rg "^---" docs/
rg "^status:" docs/

# 2. Verify required fields in frontmatter
rg "status:" docs/ | rg -v "draft|active|deprecated|superseded"

# 3. Check for common issues
rg "^# " docs/ | head -20  # Verify headings exist

# 4. Check for broken links (if link checker available)
# rg "http" docs/  # Verify external links

# 5. Verify document is not empty
wc -l docs/*.md
```

### Validation Checklist

Run these checks before marking task complete:

- [ ] Frontmatter present with required fields
- [ ] Status field is valid value
- [ ] Document has headings
- [ ] Document has content (not empty)
- [ ] All links are valid (if applicable)
- [ ] File in correct directory

### Quick Validation Script

```bash
# Check single file
FILE="docs/prds/new-feature.md"
echo "Checking $FILE..."
rg "^---" "$FILE" && echo "✓ Frontmatter found"
rg "^status:" "$FILE" && echo "✓ Status found"
rg "^# " "$FILE" && echo "✓ Headings found"
wc -l "$FILE"
```

### Common Issues to Fix

| Issue | Fix |
|-------|-----|
| Missing frontmatter | Add frontmatter with status, owner |
| Invalid status value | Use: draft, active, deprecated, superseded |
| Empty document | Add meaningful content |
| Wrong directory | Move to correct docs/ subdirectory |
