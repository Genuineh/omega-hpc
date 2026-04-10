---
name: plan
description: Task Planning and Decomposition Skill. Helps break down complex tasks into manageable subtasks, create execution plans, and structure TODO items. Use for task planning.
version: 1.0.0
---

# Task Planning Skill

Skill for decomposing complex tasks into manageable subtasks and creating execution plans.

## When to Use

- Breaking down new feature development
- Planning bug fixes
- Decomposing architecture changes
- Creating task lists for complex work
- Structuring TODO items

---

## Planning Process

### 1. Understand the Goal

Before planning, understand:
- What is the final outcome?
- What are the success criteria?
- What are the constraints?
- What is the timeline?

### 2. Decompose into Subtasks

Break big task into smaller pieces:

**Categories of subtasks:**
- **Research**: Things that need investigation
- **Design**: Planning and architecture
- **Implementation**: Code development
- **Testing**: Writing tests
- **Documentation**: Doc creation/updates
- **Review**: Code review, validation

### 3. Identify Dependencies

For each subtask:
- What must be done first?
- What can be done in parallel?
- What blocks what?

### 4. Estimate Complexity

Rate each subtask:
- **S**: Small (1-2 hours)
- **M**: Medium (half day)
- **L**: Large (1-2 days)
- **XL**: Extra Large (week+)

---

## Task Decomposition Template

### Phase 1: Initial Breakdown

```
## Task: [Task Name]

### Goal
[What we're trying to achieve]

### Subtasks

#### 1. [Subtask Name]
- **Type**: [Research/Design/Implementation/Test/Docs/Review]
- **Complexity**: [S/M/L/XL]
- **Dependencies**: [None / Task X]
- **Description**: [Brief description]

#### 2. [Subtask Name]
- **Type**: Implementation
- **Complexity**: M
- **Dependencies**: Task 1
- **Description**: [Brief description]
```

### Phase 2: Detailed Planning

For each subtask, define:
- Specific deliverables
- Acceptance criteria
- Potential blockers

---

## TODO Structure

### Create TODO Entry

```markdown
## [Feature/Bug Name]

### Overview
[Brief description]

### Subtasks

- [ ] **Task 1** (Type, Complexity)
  - Description: [What to do]
  - Deliverable: [What to deliver]
  - Dependencies: [Task X]

- [ ] **Task 2** (Type, Complexity)
  - Description: [What to do]
  - Deliverable: [What to deliver]
  - Dependencies: [Task 1]

### Timeline
- Start: [Date]
- Target: [Date]

### Risks
- [Risk 1]
- [Risk 2]
```

---

## Planning Principles

### DO

- Break into smallest meaningful units
- Identify clear dependencies
- Consider parallel execution
- Assign realistic complexity
- Define clear deliverables

### DO NOT

- Create tasks that take >2 days
- Leave dependencies unclear
- Skip testing tasks
- Forget documentation tasks
- Ignore review steps

---

## Priority Guidelines

| Priority | Criteria |
|----------|----------|
| **P0** | Critical path, blocks others |
| **P1** | Important, should do soon |
| **P2** | Nice to have, can defer |
| **P3** | Can do if time permits |

---

## Dependency Types

| Type | Meaning |
|------|---------|
| `None` | Can start immediately |
| `After X` | Must complete X first |
| `Parallel with X` | Can do alongside X |

---

## Example: Feature Development Plan

### Input
"Add user authentication system"

### Output

```markdown
# Plan: User Authentication System

## Goal
Implement JWT-based authentication with registration, login, and password reset.

## Subtasks

### Phase 1: Foundation
- [ ] **1.1 Design auth architecture** (Design, M)
  - Description: Create auth system design
  - Deliverable: docs/specs/auth-service.md
  - Dependencies: None

- [ ] **1.2 Create database schema** (Implementation, S)
  - Description: Design users table and migrations
  - Deliverable: Migration files
  - Dependencies: 1.1

### Phase 2: Core Implementation
- [ ] **2.1 Implement user registration** (Implementation, M)
  - Description: Create registration endpoint
  - Deliverable: Working registration API
  - Dependencies: 1.2

- [ ] **2.2 Implement login** (Implementation, M)
  - Description: Create login endpoint with JWT
  - Deliverable: Working login API
  - Dependencies: 2.1

- [ ] **2.3 Implement password reset** (Implementation, L)
  - Description: Password reset flow
  - Deliverable: Working password reset
  - Dependencies: 2.2

### Phase 3: Testing & Docs
- [ ] **3.1 Write unit tests** (Testing, M)
  - Description: Test auth functions
  - Deliverable: Test coverage >80%
  - Dependencies: 2.1, 2.2, 2.3

- [ ] **3.2 Write integration tests** (Testing, M)
  - Description: Test auth flows
  - Deliverable: Integration tests
  - Dependencies: 3.1

- [ ] **3.3 Update documentation** (Docs, S)
  - Description: Update API docs
  - Deliverable: docs/guide/auth.md
  - Dependencies: 2.1, 2.2, 2.3

### Phase 4: Review & Deploy
- [ ] **4.1 Code review** (Review, S)
  - Description: Review all auth code
  - Deliverable: Review report
  - Dependencies: Phase 3

- [ ] **4.2 Security audit** (Review, M)
  - Description: Security review
  - Deliverable: Security report
  - Dependencies: 4.1

## Timeline
- Start: [Date]
- Phase 1: 1 day
- Phase 2: 3 days
- Phase 3: 2 days
- Phase 4: 1 day
- Total: ~1 week

## Risks
- Third-party email service may have issues
- JWT token expiration needs tuning
```

---

## Validation Commands

```bash
# Check TODO structure
rg "^## " docs/TODO.md

# List all subtasks
rg "^- \[ \]" docs/TODO.md

# Find dependencies
rg "Dependencies" docs/TODO.md
```

---

## Quality Checklist

Before finalizing plan:

- [ ] Goal clearly stated
- [ ] All subtasks identified
- [ ] Dependencies mapped
- [ ] Complexity rated
- [ ] Deliverables defined
- [ ] Timeline estimated
- [ ] Risks identified
- [ ] No task > 2 days
