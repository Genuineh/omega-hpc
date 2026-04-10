---
name: docs-guide
description: User Guide Writer. Guides creation of user guides, tutorials, and how-to documentation. Use when creating usage documentation.
version: 1.0.0
---

# Docs-Guide Skill

Skill for creating user guides and tutorials that help users accomplish tasks.

## Purpose

User guides help users:
- Understand how to use a feature
- Complete specific tasks
- Troubleshoot common issues
- Learn best practices

## When to Use

- Creating user documentation
- Writing tutorials
- Documenting workflows
- Creating how-to guides

---

## Guide Structure

### Basic Template

```markdown
# [Topic] Guide

## Overview
[Brief introduction - what is this and why use it]

## Prerequisites
- [Prerequisite 1]
- [Prerequisite 2]

## Getting Started
[Step-by-step instructions]

## Common Tasks

### [Task 1]
[Instructions for task]

### [Task 2]
[Instructions for task]

## Troubleshooting

### [Issue 1]
**Problem**: [Description]
**Solution**: [How to fix]

### [Issue 2]
**Problem**: [Description]
**Solution**: [How to fix]

## Best Practices
- [Practice 1]
- [Practice 2]

## Related Topics
- [Related doc 1]
- [Related doc 2]
```

---

## Section Guidelines

### Overview

- 1-2 paragraphs
- What the reader will learn
- What they can accomplish after reading

### Prerequisites

What user needs before starting:
- Account access
- Required tools
- Previous knowledge
- System requirements

### Getting Started

- Step-by-step instructions
- Numbered steps
- Include expected outcomes
- Screenshots if helpful

### Common Tasks

Each task:
- Clear title
- Prerequisites if any
- Step-by-step instructions
- Expected result

### Troubleshooting

For each issue:
- Clear problem description
- Likely cause
- Solution
- Prevention if applicable

### Best Practices

- Do and do not
- Tips from experience
- Common pitfalls to avoid

---

## Writing Style

### Tone

- Clear and concise
- Friendly but professional
- Second person (you)
- Active voice

### Formatting

- Use headings for structure
- Use lists for steps
- Use code blocks for commands
- Use tables for comparisons
- Use bold for emphasis

### Examples

Include practical examples:
```bash
# Example command
command --option value

# Example output
Expected output here
```

---

## Frontmatter (Required)

```yaml
---
status: draft
owner: [author]
created: YYYY-MM-DD
updated: YYYY-MM-DD
audience: [users/developers/admin]
level: [beginner/intermediate/advanced]
---
```

---

## Audience Levels

| Level | Description |
|-------|-------------|
| `beginner` | New to the topic |
| `intermediate` | Familiar with basics |
| `advanced` | Experienced user |

---

## Example

```markdown
# Getting Started Guide

## Overview
This guide will help you set up and run your first project. By the end, you will have a working local development environment.

## Prerequisites
- A computer running [OS]
- Administrator access
- Git installed
- Code editor (VS Code recommended)

## Getting Started

### Step 1: Clone the Repository
```bash
git clone https://github.com/example/project.git
cd project
```

### Step 2: Install Dependencies
```bash
npm install
```

### Step 3: Run the Application
```bash
npm start
```

You should see output indicating the server is running.

## Common Tasks

### Running Tests
```bash
npm test
```

### Building for Production
```bash
npm run build
```

## Troubleshooting

### Port Already in Use
**Problem**: Error message about port 3000
**Solution**: Stop other processes using port 3000 or change the port in config

### Dependencies Not Installing
**Problem**: npm install fails
**Solution**: Clear npm cache and try again:
```bash
npm cache clean --force
npm install
```

## Best Practices
- Always run tests before committing
- Keep dependencies updated
- Use feature branches for changes

## Related Topics
- Configuration Guide
- API Documentation
- Deployment Guide
```

---

## Review Checklist

Before publishing guide:

- [ ] Clear purpose stated
- [ ] Prerequisites listed
- [ ] Steps are accurate
- [ ] Code examples tested
- [ ] Troubleshooting section included
- [ ] Clear navigation with headings
- [ ] Links to related docs
- [ ] Appropriate for target audience

---

## Self-Check / Validation

After creating User Guide, run these validation commands to ensure the document meets basic requirements.

### Validation Commands

```bash
# 1. Check file exists
ls -la docs/guide/

# 2. Check required sections exist
FILE="docs/guide/topic-name.md"
rg "^# " "$FILE"                        # Title
rg "^## Overview" "$FILE"               # Overview
rg "^## Prerequisites" "$FILE"           # Prerequisites
rg "^## Getting Started" "$FILE"        # Getting Started

# 3. Check frontmatter
rg "^---" "$FILE"                       # Frontmatter markers
rg "^status:" "$FILE"                   # Status field
rg "^audience:" "$FILE"                 # Audience field

# 4. Check for code blocks
rg "```" "$FILE"                       # Code blocks

# 5. Check troubleshooting section
rg "^## Troubleshooting" "$FILE"         # Troubleshooting

# 6. Count steps in getting started
rg "^### Step" "$FILE" | wc -l
```

### Validation Checklist

- [ ] Title present (H1)
- [ ] Overview section exists
- [ ] Prerequisites listed
- [ ] Getting Started with steps
- [ ] Code examples present
- [ ] Troubleshooting section exists
- [ ] Frontmatter with status and audience
- [ ] Appropriate level for target audience
- [ ] At least one code example

### Quick Validation Script

```bash
FILE="docs/guide/new-guide.md"
echo "Validating $FILE..."

# Check required sections
for section in "Overview" "Prerequisites" "Getting Started"; do
  rg "^## $section" "$FILE" && echo "✓ $section found"
done

# Check troubleshooting
rg "^## Troubleshooting" "$FILE" && echo "✓ Troubleshooting found"

# Check frontmatter
rg "^---" "$FILE" && echo "✓ Frontmatter found"
rg "^status:" "$FILE" && echo "✓ Status found"

# Check for code examples
rg "```" "$FILE" | wc -l && echo " code blocks found"

# Word count
wc -w "$FILE"
```

### Common Issues to Fix

| Issue | Fix |
|-------|-----|
| Missing overview | Add ## Overview section |
| No prerequisites | Add ## Prerequisites |
| No steps | Add ## Getting Started with numbered steps |
| No code examples | Add at least one code block |
| No troubleshooting | Add ## Troubleshooting section |
| Missing frontmatter | Add frontmatter with status, audience, level |
| Wrong level | Adjust for beginner/intermediate/advanced |
