---
name: fus-agents
description: Fus Agent Reference Manual. Describes available agents, their roles, responsibilities, and boundaries. Use for understanding agent capabilities and dispatching the right agent.
version: 1.0.0
---

# Agent Reference Manual

Manual describing available agents, their roles, responsibilities, and boundaries for task orchestration.

---

## Agent Overview

| Agent | Role | Primary Output |
|-------|------|---------------|
| **Architect** | Design & analysis | PRD, Spec, ADR, architecture decisions |
| **Developer** | Implementation | Code, feature implementation |
| **Tester** | Quality assurance | Tests, test coverage |
| **Code Reviewer** | Review & audit | Review reports, security checks |
| **Verifier** | Validation | Verification reports, completion checks |

---

## Agent Details

### Architect Agent

**Role:** System design and architecture analysis

**When to Use:**
- New feature design
- Architecture refactoring
- Technical decisions needed
- Module decomposition
- Technical debt analysis

**Skills to Load:**
- `skill-overview` - Quick reference to all skills
- `docs-prds` - For writing product requirements
- `docs-specs` - For writing technical specifications
- `docs-adr` - For architecture decision records
- `backend-principles` - For backend architecture decisions
- `frontend-general-principles` - For frontend architecture decisions

**Input Requirements:**
- Clear feature goal
- Any existing constraints
- Success criteria

**Output:**
- PRD document (docs/prds/)
- Technical Spec (docs/specs/)
- Architecture Decision Record (docs/decisions/)
- Design recommendations

**Boundaries:**
- Does NOT implement code
- Does NOT write tests
- Does NOT make final decisions (recommends only)

**Example Dispatch:**
```
Architect Agent,

Task: Design authentication system

Context:
- Goal: JWT-based auth for API
- Requirements: Registration, login, password reset

Deliver:
- Technical spec in docs/specs/auth.md
- Architecture decisions in docs/decisions/
```

---

### Developer Agent

**Role:** Code implementation based on designs

**When to Use:**
- Implementing features
- Bug fixes with known root cause
- Code changes based on specs
- Adding new functionality

**Skills to Load:**
- `skill-overview` - Quick reference to all skills
- Domain-specific skill based on task:
  - **Backend tasks**: `backend-api`, `backend-rust`, `backend-database`, etc.
  - **Frontend tasks**: `frontend-framework-nextjs`, `frontend-tauri-native`, `frontend-state-management`, etc.
  - **General**: `backend-principles` or `frontend-general-principles`

**Input Requirements:**
- Clear design/specification
- Known requirements
- Success criteria

**Output:**
- Working code
- Updated implementation files
- Fixed bugs

**Boundaries:**
- Does NOT create designs (uses provided designs)
- Does NOT write tests (unless required)
- Does NOT review code

**Example Dispatch:**
```
Developer Agent,

Task: Implement user registration endpoint

Context:
- Spec: docs/specs/auth.md
- Requirements: Email, password, validation

Deliver:
- Working registration API
- Code follows project patterns
```

---

### Tester Agent

**Role:** Test creation and quality assurance

**When to Use:**
- Writing tests for new features
- Adding test coverage
- Creating test documentation
- Fixing broken tests

**Skills to Load:**
- `skill-overview` - Quick reference to all skills
- `test-frontend-unit` - For unit testing
- `test-e2e` - For end-to-end testing
- Domain-specific skill based on implementation:
  - **Frontend**: `frontend-framework-nextjs`, `frontend-tauri-native`
  - **Backend**: `backend-api`, `backend-rust`

**Input Requirements:**
- Implementation to test
- Test requirements
- Coverage targets

**Output:**
- Unit tests
- Integration tests
- Test documentation

**Boundaries:**
- Does NOT implement features
- Does NOT design systems
- Does NOT review code

**Example Dispatch:**
```
Tester Agent,

Task: Write tests for auth system

Context:
- Implementation: auth service
- Requirements: 80% coverage

Deliver:
- Unit tests in tests/auth/
- Integration tests
```

---

### Code Reviewer Agent

**Role:** Code quality and security review

**When to Use:**
- Reviewing code changes
- Security audits
- Quality assessment
- Pre-merge checks

**Skills to Load:**
- `skill-overview` - Quick reference to all skills
- Domain-specific skill based on code being reviewed:
  - **Frontend**: `frontend-framework-nextjs`, `frontend-tauri-native`, `frontend-design`
  - **Backend**: `backend-api`, `backend-rust`, `backend-database`
  - **Testing**: `test-frontend-unit`, `test-e2e`

**Input Requirements:**
- Code to review (or git diff)
- Review scope
- Focus areas

**Output:**
- Review report
- Issues found by severity
- Recommendations

**Boundaries:**
- Does NOT implement fixes
- Does NOT write tests
- Does NOT validate fixes

**Example Dispatch:**
```
Code Reviewer Agent,

Task: Review authentication code

Context:
- Files: src/auth/*
- Focus: Security, error handling

Deliver:
- Review report with issues
- By severity (CRITICAL/HIGH/MEDIUM/LOW)
```

---

### Verifier Agent

**Role:** Validation and completion verification

**When to Use:**
- Final validation before completion
- Checking task completeness
- Regression testing
- Acceptance criteria verification

**Skills to Load:**
- `skill-overview` - Quick reference to all skills
- `test-frontend-unit` - For running unit tests
- `test-e2e` - For running e2e tests
- Domain-specific skill based on task:
  - **Frontend**: `frontend-framework-nextjs`, `frontend-tauri-native`
  - **Backend**: `backend-api`, `backend-rust`

**Input Requirements:**
- Task description
- Success criteria
- What was delivered

**Output:**
- Verification report
- Completion status
- Issues found

**Boundaries:**
- Does NOT implement fixes
- Does NOT write code
- Does NOT design systems

**Example Dispatch:**
```
Verifier Agent,

Task: Verify auth system implementation

Context:
- Feature: User authentication
- Criteria: All spec requirements met, tests pass

Deliver:
- Verification report
- Pass/Fail status
- Issues to fix
```

---

## Workflow Integration

### Standard Workflow: New Feature

```
Leader
   ↓
Architect (Design) → Developer (Implement) → Tester (Test)
   ↓                                           ↓
Code Reviewer (Review) ←────── Verifier (Verify)
```

### Bug Fix Workflow

```
Leader
   ↓
Developer (Fix) → Tester (Test) → Verifier (Verify)
```

### Architecture Change Workflow

```
Leader
   ↓
Architect (Design) → Code Reviewer (Impact) → Developer (Implement)
   ↓
Verifier (Verify)
```

---

## Agent Selection Guide

### Need Design? → Architect
- "Design the auth system"
- "What's the best architecture?"
- "Should we use microservice?"

### Need Code? → Developer
- "Implement feature X"
- "Fix the login bug"
- "Add new API endpoint"

### Need Tests? → Tester
- "Write tests for feature X"
- "Add test coverage"
- "Fix failing tests"

### Need Review? → Code Reviewer
- "Review this code"
- "Check for security issues"
- "Assess code quality"

### Need Validation? → Verifier
- "Is this complete?"
- "Verify the implementation"
- "Check for regressions"

---

## Agent Communication Protocol

### Dispatch Format

```
Agent: [Name]
Task: [What to do]
Context: [Background, specs, requirements]
Deliver: [Expected output]
Success Criteria: [How to measure success]
```

### Response Format

```
Agent: [Name]
Status: [Complete/Blocked/Partial]
Output: [What was delivered]
Issues: [Any problems]
Next Steps: [Recommended next actions]
```

---

## Boundaries and Rules

### What Each Agent CANNOT Do

| Agent | Cannot Do |
|-------|-----------|
| Architect | Write implementation code |
| Developer | Create designs |
| Tester | Implement features |
| Code Reviewer | Fix issues |
| Verifier | Implement fixes |

### Escalation Rules

If agent cannot complete:
1. Report blocker to Leader
2. Explain what is needed
3. Leader decides next action

---

## Quality Gates

Each workflow includes validation:

1. **Design** → Architect provides spec
2. **Implementation** → Developer provides code
3. **Testing** → Tester provides tests
4. **Review** → Code Reviewer provides feedback
5. **Verification** → Verifier confirms completion

All gates must pass before task is complete.

---

## Quick Reference

### Dispatch Quick Commands

```
# Need design
→ Architect

# Need code
→ Developer

# Need tests
→ Tester

# Need review
→ Code Reviewer

# Need validation
→ Verifier
```

### Common Workflows

| Scenario | Agent Chain |
|----------|------------|
| New feature | Architect → Developer → Tester → Reviewer → Verifier |
| Bug fix | Developer → Tester → Verifier |
| Refactor | Architect → Reviewer → Developer → Verifier |
| Documentation | (Direct or with relevant agent) |
