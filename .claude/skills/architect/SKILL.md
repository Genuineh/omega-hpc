---
name: architect
description: Architecture Rules Enforcer. Validates architecture designs against core principles, flags anti-patterns, and provides compliance scoring. Use when designing systems, reviewing architecture, or making architectural decisions.
version: 1.0.0
---

# Architecture Rules

Use this skill when designing systems, reviewing architecture decisions, or validating architectural designs.

---

## Core Architectural Principles

### 1. Single Responsibility + High Cohesion, Low Coupling
- Every module/class/service does exactly one thing
- Related logic stays together, unrelated logic stays apart
- Minimize dependencies between components

### 2. Clear Interfaces & Separation of Concerns
- Explicit interfaces only (no implicit contracts)
- Business logic, UI, data, and infrastructure must remain completely separate
- Each layer has clear responsibilities and boundaries

### 3. Scalability
- Prefer stateless design for horizontal scalability
- Design for growth - consider load patterns early
- Avoid bottlenecks and single points of failure

### 4. Testability
- All business logic must be fully independent of UI, database, or external frameworks
- Business logic should be unit-testable without mocks for infrastructure
- Separate domain logic from infrastructure concerns

### 5. Maintainability & Security by Design
- Easy to understand, modify, and extend
- Security built in from the start, not added later
- Clear code ownership and documentation

---

## Red Flags (High Risk)

Flag these immediately as architectural anti-patterns:

| Anti-Pattern | Impact | Example |
|--------------|--------|---------|
| **God Objects** | Unmaintainable, hard to test, single point of failure | A class handling auth, caching, logging, and business logic |
| **Circular Dependencies** | Brittle, hard to test, deployment issues | A → B → C → A |
| **Shotgun Surgery** | High coupling, fear of change | One change requires updates across many files |
| **Tight Coupling** | Hard to test, replace, or scale | Direct instantiation of concrete classes |
| **Anemic Domain Model** | No business logic encapsulation | Objects with only getters/setters, no behavior |
| **Premature Abstraction** | Unnecessary complexity, over-engineering | Abstracting before understanding the domain |
| **Big Ball of Mud** | No clear boundaries, spaghetti code | Everything connected to everything |
| **Hidden Globals / Singletons** | Hidden dependencies, hard to test | Global state accessed everywhere |
| **Layer Violations** | Tight coupling, unclear responsibilities | Business logic in controllers, UI code in services |

---

## Compliance Scoring

Rate architecture against principles:

| Score | Meaning | Action |
|-------|---------|--------|
| 9-10 | Excellent | Proceed with confidence |
| 7-8 | Good | Minor improvements needed |
| 5-6 | Fair | Significant refactoring required |
| 3-4 | Poor | Major redesign needed |
| 1-2 | Critical | Stop and redesign |

### Scoring Criteria

For each principle, rate 1-2:
- **Single Responsibility**: Does each component have one clear purpose?
- **Separation of Concerns**: Are layers properly separated?
- **Scalability**: Is the design stateless and horizontally scalable?
- **Testability**: Can business logic be unit tested independently?
- **Maintainability**: Is the code easy to understand and modify?

**Total Score** = Sum of all ratings (max 10)

---

## Usage

1. Before making architectural decisions → validate against principles
2. When reviewing code → check for red flags
3. When designing new systems → use principles as guidelines
4. When debugging → check if issues are caused by anti-patterns

---

## Self-Check

- [ ] Does each component have a single, clear responsibility?
- [ ] Are there clear interfaces between layers?
- [ ] Is the design stateless where possible?
- [ ] Can business logic be tested without infrastructure?
- [ ] Are there hidden globals or singletons?
- [ ] Are there circular dependencies?
- [ ] Is there proper separation of concerns?
