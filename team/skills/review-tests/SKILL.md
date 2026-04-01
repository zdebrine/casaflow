---
name: review-tests
description: >
  Use when auditing test quality after implementation. Runs four phases:
  coverage audit, false positive detection via mutation testing, quality
  rubric evaluation, and letter-grade summary with action items. Invoked
  by /review-tests and automatically during Stage 4 (Polish) approve-gate.
tier: workflow
alwaysApply: false
---

# Skill: Review Tests

## Purpose

This skill evaluates whether tests actually prove what they claim to prove.
A test suite that always passes is not the same as a test suite that catches
real bugs. The difference is the key insight:

**The best way to prove a test matters is to intentionally break the code
it covers and confirm the test fails.** If breaking the production code
breaks the test, the test is real. If the test still passes, it's a false
positive — and false positives are worse than no tests, because they provide
false confidence.

---

## When this skill is active

- Invoked by `/casaflow:review-tests`
- Invoked automatically at the start of Stage 4 (Polish) in the approve-gate
  before the comprehension questions

---

## Phase 1: Coverage audit

Claude reads the test files and the feature spec side by side.

For each acceptance criterion in the spec, answer:
- Is there a test for the happy path? (yes / no / partial)
- Is there a test for the failure path? (yes / no / partial)
- Is the test verifying the right thing, or an implementation detail that
  could change without the feature breaking?

Produce a coverage table:

| Criterion | Happy path | Failure path | Notes |
|-----------|-----------|--------------|-------|
| [criterion text] | ✓ / ✗ / ~ | ✓ / ✗ / ~ | [gap or concern] |

Where ✓ = covered, ✗ = missing, ~ = present but weak.

---

## Phase 2: False positive check (the break test)

For every test marked ✓ or ~, propose a specific mutation — a small change
to the production code that *should* break the feature but *might not* break
the test.

Format each mutation:

> **Test**: `[test name or describe block]`
> **File to mutate**: `[file path]`
> **Mutation**: Change `[specific line or expression]` from `[X]` to `[Y]`
> **Expected**: The test should fail because `[specific reason]`
> **Risk**: If the test does NOT fail after this mutation, it is a false positive

Instruct the developer to make each mutation, run the affected test, and
report the result. After each:
- Test fails as expected → ✓ confirmed real
- Test still passes → false positive detected, flagged for fix

**Prioritize the top 3 highest-risk mutations.** Focus on:
1. Auth and permission checks
2. Data-modifying operations (writes, deletes, updates)
3. Error handling paths

The developer does not need to run every mutation. Prioritization is
Claude's responsibility.

---

## Phase 3: Test quality rubric

Evaluate each test file against five dimensions. Note violations with
file and line references.

**Isolation**: Does this test depend on state left by another test?
Tests that must run in order to pass are coupled. Coupled tests produce
false failures in isolation and hide real bugs.

**Specificity**: Does the test assert what it claims to assert?
A test named `should reject invalid input` that only checks a 400 status
code is not testing the error message, format, or rejected field. Weak
test dressed as a strong one.

**Realistic inputs**: Does the test use realistic data?
Tests using `id: 1` and `name: "test"` miss bugs that appear only with
real-world inputs: long strings, special characters, zero, negative numbers,
concurrent requests, unicode, max-length values.

**Edge cases**: Are the boring edge cases tested?
Empty arrays, zero values, maximum values, missing optional fields, null
responses. These are the inputs that break real apps in production.

**Behavior over implementation**: If an internal function is renamed, do
the tests still make sense?
Tests tied to internal function names break during refactoring and produce
false failures. Tests should describe what the system does, not how.

---

## Phase 4: Summary and action items

**Test suite grade** (A / B / C / D):

| Grade | Criteria |
|-------|---------|
| A | All criteria covered, no false positives found, rubric clean |
| B | Minor coverage gaps, no false positives, small rubric issues |
| C | Coverage gaps or false positives found, rubric violations present |
| D | Significant coverage gaps and/or multiple false positives |

**Action items**: Numbered list ordered by severity. Each item includes:
- What to fix
- File and line reference
- Why it matters (what bug it would miss in production)

**What's solid**: Explicitly call out tests that are working well. This
is not filler — developers need to know what not to change during the fix
pass. A good test is an asset and should be named as one.

---

## After the review

Ask:

> "Do you want me to fix any of the action items, or would you prefer to
> tackle them yourself first? If you fix them yourself, run
> `/casaflow:review-tests` again and I'll re-evaluate and update the grade."

Giving the developer the option to fix things themselves is important.
The goal is their understanding of the test suite, not just a passing grade.

---

## What Claude never does

- Never says "tests look good" without completing all four phases
- Never skips Phase 2 because "the tests seem thorough"
- Never treats line coverage percentage as a proxy for test quality
- Never flags a false positive based on reading alone — the mutation must
  be run by the developer
- Never compresses Phase 2 by proposing mutations without instructing
  the developer to run them
