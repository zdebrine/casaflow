---
name: approve-gate
description: >
  Use when a build stage has just completed. Enforces the CasaPerks
  comprehension gate — stage report plus three mandatory questions the
  developer must answer before the next stage begins. Invoked automatically
  at the end of every build stage and by /approve.
tier: workflow
alwaysApply: false
---

# Skill: Approve Gate

## Purpose

This skill governs what happens at the end of every build stage. The gate
exists to keep the developer in comprehension of the codebase as it grows.
Claude pauses here and does not continue until both parts are complete.

**The gate has two parts:**
1. Claude produces a structured stage report
2. Developer passes three comprehension questions

Both are required. The gate cannot be skipped. This discomfort is the
learning mechanism — if a developer can't answer these questions, the next
stage will be harder to debug.

---

## When this skill is active

- Automatically invoked at the end of every `/casaflow:build` stage
- Invoked when the developer runs `/casaflow:approve`

---

## Part 1: Stage Report (Claude produces this automatically)

Before asking any questions, Claude produces the following report:

### Files changed
An exact list of every file created or modified. For each file:
- File path
- One sentence: what it does and why it exists
- Coupling: what breaks if this file is removed or significantly changed?

### How to run
Step-by-step commands to start the app and exercise what was just built.
Not "run the app and test it" — specific commands, specific URLs, specific
interactions to perform. Number every step.

### What to test manually
A checklist derived from the feature spec acceptance criteria:
```
- [ ] [user action] → [expected result]
- [ ] [user action] → [expected result]
```

### What was not done
An honest list of deferred items with brief reasons. This prevents the
developer from assuming the current stage is more complete than it is.

### Tradeoffs made
For every significant implementation decision in this stage:
- What was chosen and why
- What the main alternative was and why it wasn't chosen
- What the downside of the chosen approach is

This section is not optional. If no significant decisions were made, say
"No significant tradeoffs in this stage." But that is rare.

---

## Part 2: Comprehension Check (developer must pass)

After the stage report, Claude asks exactly three questions.

**Question 1 — Structural**
> "Walk me through what happens when [key interaction from this stage].
> Start from the user action and trace it to the data layer."

**Question 2 — Failure mode**
> "What happens in the code if [a realistic failure condition for this
> stage]? Where would you look to debug it?"

**Question 3 — Change impact**
> "If we needed to change [a specific thing Claude just built], what other
> files would you need to touch and why?"

Claude chooses questions that are specific to what was built in this stage.
Generic questions that could apply to any stage are not acceptable.

### Evaluating answers

**Acceptable**: Answer traces the correct code path, names the right files,
identifies the right failure point, or correctly maps the change surface.
Minor gaps are fine if the core understanding is there.

**Not acceptable**: Vague answers ("it would just error"), circular
definitions ("it would fail because it fails"), or responses that reveal
the developer hasn't read the code that was just written.

If an answer is not acceptable:
1. Explain specifically what was misunderstood (name the file, the function,
   the behavior)
2. Ask the question again with different framing
3. Do not proceed until the answer is acceptable

If the developer runs `/approve` without answering the comprehension
questions, respond:

> "Before I continue, I need you to answer the three comprehension questions
> above. This isn't a formality — if you can't answer these, we'll hit
> problems in the next stage that are harder to debug."

---

## Gate outcomes

**Pass** → Confirm stage complete. Open the next stage with:
- One sentence describing what the next stage will cover
- Which spec acceptance criteria it addresses
- Any concerns from the current stage that carry forward

**Redirect** → Developer specifies a change. Claude adjusts scope or
approach. Small changes (a different variable name, a different endpoint
path) do not require re-running comprehension questions. Large redirects
(architecture changes, new data flows, additional acceptance criteria) require
re-running all three questions.

**Pause** → Developer needs time. Claude summarizes:
- Current stage number and what was completed
- The three unanswered comprehension questions
- What the next stage covers
So context can be resumed cleanly in a new session.

---

## Default stage definitions

Teams override these in `casaflow.config.md`. These are the defaults:

| Stage | Scope |
|-------|-------|
| 1 — Scaffold | Project structure, dependencies, config, CI skeleton |
| 2 — Backend | API endpoints, data layer, auth, validation, security |
| 3 — Frontend | UI components, API integration, error handling, UX |
| 4 — Polish | Tests, docs, edge cases, performance, accessibility |

For large features, Claude may propose splitting a stage. This should happen
at plan time, not mid-stage.

**Stage 4 (Polish) special behavior**: Before the comprehension questions,
invoke `review-tests` for the four-phase test audit. Passing the gate
requires both the test review and the comprehension questions.

---

## What Claude never does at a gate

- Never says "looks good, moving on" without running the comprehension check
- Never skips the tradeoffs section
- Never marks a stage complete if tests for that stage are missing without
  an explicit deferral reason documented in "What was not done"
- Never continues if the developer's answers reveal a fundamental
  misunderstanding of the architecture
- Never compresses the three questions into one question
