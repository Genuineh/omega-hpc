---
name: developer
description: Development Principles and Execution Standard. Defines the principles, workflow, and quality bar for implementation work when turning clear requirements into maintainable code changes.
version: 1.0.0
---

# Developer

Use this skill when writing or modifying production code and the task needs a clear implementation standard.

This is not an agent role. It is a set of development principles and execution rules for implementation work.

---

## Development Principles

- Start from the real requirement, not assumptions inferred from code alone.
- Prefer the smallest change that fully solves the problem.
- Preserve existing architecture and conventions unless the task explicitly includes refactoring.
- Keep logic cohesive, dependencies explicit, and side effects controlled.
- Optimize for readability, maintainability, and testability over cleverness.

---

## Execution Standard

1. Read the requirement, touched area, and nearby call sites before editing.
2. Identify the minimal file set and the behavioral change being introduced.
3. Implement with existing project patterns for naming, interfaces, validation, and error handling.
4. Keep business logic separate from framework glue and I/O where practical.
5. Run the narrowest meaningful verification for the changed area.
6. Document any remaining limitation, follow-up, or risk if verification is incomplete.

---

## Guardrails

- Do not silently redesign requirements or architecture.
- Do not introduce broad abstractions without a concrete need in the current code path.
- Do not leave debug code, dead branches, or placeholder TODOs behind.
- Do not couple unrelated responsibilities into one file, function, or type.
- Do not claim completion without stating what was or was not verified.

If the task cannot be completed cleanly without changing requirements, surface the constraint explicitly.

---

## Quality Bar

- Requirement is clear enough to code without guessing core behavior.
- Existing patterns in the touched area were reviewed first.
- Error paths and edge cases were handled.
- The change remains understandable to another engineer reading it later.
- Debug code, dead branches, and placeholder TODOs were not left behind.
- Verification was run when feasible, or the gap was stated plainly.

---

## Output Expectations

When finishing work with this skill, report:

- what was implemented
- which files changed
- what verification ran
- any remaining risks, assumptions, or follow-up
