---
name: tester
description: Testing Principles and Execution Standard. Defines how to design, write, and validate tests that protect behavior, cover edge cases, and provide credible regression safety.
version: 1.0.0
---

# Tester

Use this skill when creating, updating, or evaluating automated tests and the task needs a clear testing standard.

This is not an agent role. It is a set of testing principles and execution rules for validation work.

---

## Testing Principles

- Test behavior that matters to users, callers, or contracts.
- Favor deterministic tests over broad but flaky coverage.
- Protect regressions in the changed path before chasing coverage numbers.
- Prefer realistic boundaries and inputs over brittle implementation-coupled assertions.
- Use mocks only at true external boundaries.

---

## Execution Standard

1. Read the implementation and existing tests before adding new cases.
2. Enumerate happy path, expected failures, boundaries, and regressions.
3. Add or update tests that assert externally visible outcomes.
4. Follow local test conventions for structure, fixtures, and naming.
5. Run the narrowest meaningful test command for the affected scope.
6. Report what was validated, what remains untested, and any blockers found.

---

## Guardrails

- Do not weaken assertions just to make tests pass.
- Do not over-mock internal logic that should be exercised directly.
- Do not rely on implementation details when observable behavior can be asserted instead.
- Do not leave flaky, order-dependent, or time-sensitive tests unexplained.
- Do not treat test count as a substitute for meaningful coverage.

If testing reveals a product bug, missing requirement, or unstable test seam, state it directly.

---

## Quality Bar

- Happy path is covered.
- Expected failures and edge cases are covered.
- Test names describe behavior and condition.
- Assertions verify outcomes, not internal mechanics.
- Added tests increase confidence in the changed behavior.
- The affected tests were run when feasible, or the limitation was stated.

---

## Output Expectations

When finishing work with this skill, report:

- what behavior was tested
- which test files changed
- what command(s) ran and whether they passed
- any uncovered gaps, flaky areas, or production issues found
