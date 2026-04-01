---
name: retro
description: >
  Use when a feature has shipped and the developer wants to capture what was
  learned. Runs a five-question conversational retrospective and saves the
  output as a team artifact. Invoked by /retro and prompted automatically
  after finish/merge.
tier: workflow
alwaysApply: false
---

# Skill: Retro

## Purpose

The retro exists to make developers better, not just faster. Every feature
built on a codebase teaches something — about the code, about AI-assisted
development, about how to spec work. Without a deliberate capture step, that
learning evaporates.

The retro output is a team artifact. Over time, `retros/` becomes a living
record of how the team's understanding of their codebase and their AI
workflow is evolving.

---

## When this skill is active

- Developer runs `/casaflow:retro [feature-name]`
- Automatically prompted by `finish` after a merge is confirmed

---

## Format: Conversation, not a form

Ask the five questions one at a time. Wait for a real answer before moving
to the next. Do not present them all at once. Do not rush.

If an answer is thin ("it went fine," "nothing surprised me"), push once:
"What was the part you were most uncertain about going into this?"

---

## The five questions

**Question 1 — What surprised you?**

> "What was the one thing in this feature that turned out to be more complex
> than you expected when you wrote the spec? Why was it harder?"

**Question 2 — What would you spec differently?**

> "Looking at the spec you wrote before we started — which acceptance
> criterion turned out to be wrong, incomplete, or too vague? How would
> you write it now?"

**Question 3 — What did you learn about the codebase?**

> "What did you discover about how the existing code is structured that you
> didn't know before this feature? Would you have built something differently
> if you'd known it at the start?"

**Question 4 — What did you learn about AI-assisted development?**

> "Was there a moment in this feature where my output was wrong, misleading,
> or subtly off? How did you catch it? What does that tell you about where
> to pay close attention in the next feature?"

**Question 5 — What's your one rule for next time?**

> "Finish this sentence: 'Next time I build a feature like this, I will
> always ___.' It should be specific enough that you could check whether
> you followed it."

---

## Output format

After all five answers, save the retro to `retros/<feature-name>.md`.

If the developer provided a feature name as an argument, use it. If not,
ask: "What should we call this retro? (I'll save it as
`retros/<your-answer>.md`)"

```markdown
# Retro: [feature name]

Date: YYYY-MM-DD

## What surprised us
[developer's answer]

## Spec lesson
[developer's answer, plus Claude's observation on what this pattern
suggests about spec writing in general for this codebase]

## Codebase discovery
[developer's answer]

## AI collaboration lesson
[developer's answer, plus Claude's observation — be honest if the
mistake was Claude's output]

## Rule for next time
[developer's one rule, verbatim]

## Patterns
[See below]
```

---

## Pattern detection

After saving the retro, scan the `retros/` directory for other retro files.

If a recurring theme appears across two or more retros, add a **Patterns**
section to the new retro:

> "I noticed this connects to a pattern from earlier retros:
> - [previous retro name]: [the shared theme]
> - [previous retro name]: [the shared theme]
>
> Recurring surprises become process improvements. Worth discussing as a
> team whether this points to a spec template change, a codebase refactor,
> or an updated concerns checklist."

Patterns to watch for:
- The same file or module surprising developers repeatedly
- The same type of acceptance criterion turning out to be wrong
- The same failure mode of AI output (hallucinated APIs, wrong framework
  patterns, missed auth requirements)
- The same codebase coupling discovered unexpectedly

If no patterns are detected, omit the Patterns section entirely. Do not add
filler.

---

## What Claude never does

- Never rushes through all five questions at once
- Never accepts "everything went fine" as a complete answer without one
  follow-up push
- Never writes the developer's answers for them
- Never skips the pattern scan — it is the mechanism by which individual
  retros become team intelligence
