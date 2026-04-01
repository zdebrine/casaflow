---
name: explain
description: >
  Use when the developer wants a deep technical explanation of code just
  written. Covers approach and tradeoffs, failure modes, change surface, what
  to test, and what to refactor. Can be invoked mid-stage, at a gate, or after
  the fact. Invoked by /explain.
tier: workflow
alwaysApply: false
---

# Skill: Explain

## Purpose

This skill exists to transfer mental models, not to summarize code. The
developer can read the code. What they need is:

- Why this approach over alternatives
- Where this code will fail in production
- What files to touch if requirements change
- What tests would catch real bugs
- What to clean up before this is truly production-ready

**Claude never produces a code summary.** Every section must answer "why"
or "what would happen if" — not "what does this code do."

---

## When this skill is active

Invoked by `/casaflow:explain` with an optional argument:
- No argument → explains the most recent code increment
- File path → explains that file
- Function/class name → explains that symbol
- Stage number → explains everything from that build stage

---

## Five sections (all required, in order)

### 1. Approach and tradeoffs

Why was this specific approach taken?

- Name at least two alternatives that were considered (or that Claude
  chose not to use)
- For each alternative: one sentence on why it was not chosen
- For the chosen approach: what is its main downside?

Do not say "we could have used X but this approach is better." Name why X
was rejected and acknowledge the cost of what was chosen.

### 2. Failure modes

The three most likely ways this code fails in production. For each:

- **What triggers it**: the specific condition that causes the failure
- **What it looks like from outside**: the symptom a user or monitor sees
- **Where to look in code**: the exact file and function where diagnosis
  starts

Do not describe hypothetical edge cases. Describe failure modes with the
highest probability in a real production environment given this codebase.

### 3. Change surface

If the behavior described in the explanation target needed to change, which
files would you touch and in what order?

- List each file
- One sentence on what changes in that file
- What breaks first if the order is wrong?

This maps the dependency graph from the developer's perspective, not the
compiler's.

### 4. What to test

**If no tests exist yet**: What are the three most important tests to write?
For each: what specific bug would it catch if the production code were broken?
A test that doesn't map to a named failure mode is not important.

**If tests exist**: Which existing test is most likely a false positive?
Name the test, describe the mutation that would expose it, and explain why
the test might pass even when it shouldn't.

### 5. What to refactor if time permits

Name one specific thing in the code just written that you would change if
this were going to production tomorrow.

Must include:
- File path
- Line range or function name
- The pattern you'd replace
- What you'd replace it with and why

Do not say "this could be improved." Say exactly what, where, and how.

---

## Format requirements

- Use the section headers above (`## 1. Approach and tradeoffs`, etc.)
- Use file:line references wherever possible (`auth/middleware.ts:47`)
- Use concrete examples — do not describe behavior in the abstract
- Avoid "this is a good approach" — developer can judge quality, they need
  understanding

---

## Length guidance

Each section: 3–8 sentences or equivalent. Explain is not a spec — it is
a targeted transfer of the specific mental model the developer needs to own
this code. More is not better.
