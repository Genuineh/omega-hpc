---
name: review
description: Review Principles and Execution Standard. Defines how to review code changes for correctness, regressions, security, maintainability, and missing validation with a consistent severity bar.
version: 1.0.0
---

# Review

Use this skill when reviewing a diff, PR, staged change, or completed task and the goal is to assess quality with a consistent standard.

This is not an agent role. It is a set of review principles and reporting standards for evaluation work.

---

## Review Principles

- Review for real risk, not personal style preferences.
- Prioritize correctness, regressions, security, and data integrity ahead of elegance.
- Evaluate changed code in context, including call sites and related behavior.
- Prefer fewer high-confidence findings over many speculative comments.
- Keep findings actionable and tied to impact.

---

## Execution Standard

1. Gather the diff and enough surrounding context to understand what changed.
2. Inspect the changed behavior before judging implementation details in isolation.
3. Review in severity order: correctness, security, data loss, regression risk, missing tests, maintainability.
4. Report only findings that are concrete and defensible.
5. If no actionable issues are found, state that explicitly and note residual risk or test gaps.

---

## Guardrails

- Do not block on style unless it conflicts with established project rules.
- Do not report theoretical concerns without a plausible failure mode.
- Do not repeat the same issue across multiple findings when one consolidated comment is stronger.
- Do not hide uncertainty; label assumptions or open questions separately from findings.
- Do not mix implementation work into the review unless the task explicitly asks for both.

---

## Severity Guide

- `critical`: security issue, data loss, or broken core behavior
- `high`: likely bug, regression, or major maintainability risk
- `medium`: weaker correctness risk, performance issue, or notable gap
- `low`: minor robustness or clarity issue worth fixing

---

## Output Expectations

When finishing work with this skill:

- list findings first, ordered by severity
- include file and line references for each finding when available
- keep summaries brief and secondary
- state explicitly if no findings were identified
