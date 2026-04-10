---
name: docs-specs
description: Technical Specification Writer. Guides creation of technical specification documents for systems and components. Use when documenting technical implementation details.
version: 1.0.0
---

# Docs-Specs Skill

Skill for creating Technical Specifications that define how to implement a feature or system.

## Purpose

Technical specs document the implementation details:
- How will it be built?
- What are the technical decisions?
- What are the interfaces and contracts?
- What is the architecture?

## When to Use

- Documenting system architecture
- Planning implementation details
- Defining APIs and interfaces
- Making technical decisions
- Creating formal specifications

---

## Specification Structure

### Required Sections

```markdown
# [System/Component Name] Specification

## Overview
[High-level description of what this system does]

## Goals
- [Goal 1]
- [Goal 2]

## Non-Goals
- [What this system will NOT do]

## Architecture

### Components
- [Component 1]: [Description]
- [Component 2]: [Description]

### Data Flow
[Description or diagram of how data flows]

## API Specification

### [Endpoint/Method]
- **Input**: [Parameters]
- **Output**: [Return value]
- **Errors**: [Error cases]

## Data Models

### [Model Name]
| Field | Type | Description |
|-------|------|-------------|
| name | string | User name |

## Technical Decisions

| Decision | Choice | Rationale |
|---------|--------|-----------|
| [Topic] | [Choice] | [Why] |

## Security Considerations
- [Security point 1]
- [Security point 2]

## Performance Requirements
- [Performance requirement 1]
- [Performance requirement 2]

## Testing Strategy
- Unit tests for [component]
- Integration tests for [flow]
```

---

## Section Guidelines

### Overview

- 2-3 sentences
- What the system does
- Why it exists

### Goals

- Specific objectives
- Measurable where possible

### Non-Goals

- Scope boundaries
- What is explicitly out of scope

### Architecture

**Components**
- List all major components
- Describe responsibility of each

**Data Flow**
- How data moves through system
- Key integrations

### API Specification

For each endpoint/function:
- Input parameters with types
- Output/return value
- Error cases
- Examples if helpful

### Data Models

For each data structure:
- Field name
- Type
- Description
- Constraints

### Technical Decisions

Document important decisions:
- The decision made
- Alternatives considered
- Rationale for choice

### Security Considerations

- Authentication
- Authorization
- Data protection
- Input validation

### Performance Requirements

- Latency
- Throughput
- Resource usage

---

## Writing Tips

### DO

- Be specific about types and interfaces
- Include examples
- Document edge cases
- Consider error scenarios
- Reference related specs/prds

### DO NOT

- Include implementation code
- Be too verbose
- Skip error handling
- Ignore security

---

## Frontmatter (Required)

```yaml
---
status: draft
owner: [team or developer]
created: YYYY-MM-DD
updated: YYYY-MM-DD
version: 1.0
supersedes: [previous spec if any]
related_prds: [list of related PRD files]
---
```

---

## Status Values

| Status | Meaning |
|--------|---------|
| `draft` | Work in progress |
| `review` | Under review |
| `approved` | Finalized, ready to implement |
| `deprecated` | Outdated, superseded |
| `implemented` | Fully implemented |

---

## Example

```markdown
# User Service Specification

## Overview
Microservice handling user management including registration, authentication, and profile management.

## Goals
- Provide secure user authentication
- Manage user profiles
- Support multiple authentication methods

## Non-Goals
- Not responsible for authorization (handled by permission service)
- Not handling billing (separate service)

## Architecture

### Components
- **Auth Module**: Handles login, logout, token management
- **Profile Module**: Manages user profile data
- **Repository**: Data access layer

### Data Flow
User -> API Gateway -> User Service -> Database

## API Specification

### POST /users/register
- **Input**: { email, password, name }
- **Output**: { user, token }
- **Errors**: EMAIL_EXISTS, INVALID_INPUT

### POST /users/login
- **Input**: { email, password }
- **Output**: { token, user }
- **Errors**: INVALID_CREDENTIALS, ACCOUNT_LOCKED

## Technical Decisions

| Decision | Choice | Rationale |
|---------|--------|-----------|
| Auth Method | JWT | Stateless, scales well |
| Password Hash | bcrypt | Industry standard |
| Database | PostgreSQL | ACID compliance needed |

## Security Considerations
- Passwords hashed with bcrypt (cost factor 12)
- JWT tokens expire in 24 hours
- Rate limiting on login endpoint

## Performance Requirements
- 99th percentile latency < 200ms
- Support 10,000 concurrent users
```

---

## Self-Check / Validation

After creating Technical Specification, run these validation commands to ensure the document meets basic requirements.

### Validation Commands

```bash
# 1. Check file exists
ls -la docs/specs/

# 2. Check required sections exist
FILE="docs/specs/component-name.md"
rg "^# " "$FILE"                      # Title
rg "^## Overview" "$FILE"             # Overview
rg "^## Goals" "$FILE"                # Goals
rg "^## Architecture" "$FILE"         # Architecture
rg "^## API Specification" "$FILE"     # API section
rg "^## Data Models" "$FILE"         # Data Models
rg "^## Technical Decisions" "$FILE"   # Technical Decisions

# 3. Check frontmatter
rg "^---" "$FILE"                     # Frontmatter markers
rg "^status:" "$FILE"                 # Status field
rg "^owner:" "$FILE"                  # Owner field

# 4. Check for API endpoints
rg "### POST|GET|PUT|DELETE" "$FILE"  # API methods

# 5. Check for data models
rg "^### .*Model" "$FILE"              # Data model headings
```

### Validation Checklist

- [ ] Title present (H1)
- [ ] Overview section exists
- [ ] Goals defined
- [ ] Architecture section present
- [ ] API specification present (if applicable)
- [ ] Data models defined (if applicable)
- [ ] Technical decisions documented
- [ ] Security considerations
- [ ] Frontmatter with status and owner

### Quick Validation Script

```bash
FILE="docs/specs/new-spec.md"
echo "Validating $FILE..."

# Check required sections
for section in "Overview" "Goals" "Architecture" "Technical Decisions"; do
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
| Missing overview | Add ## Overview section |
| No architecture | Add ## Architecture with components |
| No API spec | Add ## API Specification with endpoints |
| No technical decisions | Add ## Technical Decisions table |
| Missing security | Add ## Security Considerations |
| Missing frontmatter | Add frontmatter with status, owner, version |
