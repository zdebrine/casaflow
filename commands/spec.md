---
description: "Write a feature spec before any code is written. Guides you through feature summary, acceptance criteria, non-goals, test spec, architecture sketch, and open questions."
---

# Skill: Feature Spec

## Purpose

This skill governs how Claude helps a developer write a feature spec before
any code is written. The spec is not bureaucracy — it is the mechanism by
which the developer proves to themselves (and to Claude) that they understand
what they're building before delegating implementation to AI.

A developer who cannot write a spec cannot evaluate whether Claude's output
is correct. The spec is the developer's intellectual stake in the feature.

**Claude does NOT write the spec for the developer.** Claude asks questions
and reflects answers back until each section is solid.

---

## When this skill is active

Invoked by:
- Developer running `/casaflow:spec`
- `kickoff` skill when no spec file exists for a feature/improvement

---

## Spec sections (developer writes all of these)

### 1. Feature summary
One paragraph. What is this feature? Who uses it? What problem does it solve?

**Claude's job**: Ask "why does this feature exist?" until the answer is
concrete and not generic. Push back on "we need this for users" — that's not
a reason, that's a category.

### 2. Acceptance criteria
A numbered list of concrete, testable statements. Format: "Given X, when Y,
then Z." Not vague goals — specific observable outcomes.

**Claude's job**: Push back on any criterion that can't be tested. "The UI
looks good" is not a criterion. "The modal closes within 300ms of clicking
confirm" is a criterion. Any criterion using the word "should" instead of
"will" needs to be rewritten.

Require at least three criteria. One criterion features are under-specified.

### 3. Explicit non-goals
What is this feature intentionally NOT doing?

**Claude's job**: Prompt for at least two non-goals. Developers almost always
forget this section. Non-goals prevent scope creep and help the developer
communicate constraints to the AI implementer.

### 4. Test spec
For each acceptance criterion, define:
- Happy path test: what does success look like in a test?
- Failure path test: what does failure look like? What should the system do?
- "How to break this test" check: describe one way to make this test a false
  positive. Confirm it can't happen accidentally.

**Claude's job**: For each acceptance criterion, Claude **proposes** all three
paths (happy, failure, false positive) based on the criterion and architecture
context. The developer then reviews, edits, or approves each suggestion.
Developers should not be expected to generate these from scratch — Claude
drafts, the developer refines. A spec without failure tests is still
incomplete, but the developer's job is curation and correction, not invention.

### 5. Architecture sketch
Before implementation: what files will change? What new files will be created?
What data flows through this feature? Draw it in words or pseudocode.

**Claude's job**: Ask "where does this data come from and where does it go?"
If the developer can't answer, they don't understand the feature yet. Do not
proceed to `/build` until this is answered.

### 6. Open questions
What does the developer not know yet? What assumptions are they making?

**Claude's job**: Surface assumptions the developer hasn't made explicit.
Common ones: "I'm assuming the API already exists," "I'm assuming auth is
handled upstream," "I'm assuming this data is already in the database."

If the developer says they have no open questions, push: "What would surprise
you during implementation? What are you most uncertain about?"

---

## Red flags (Claude watches for these)

Stop and surface explicitly before moving on:

- **Developer can't describe failure cases** — spec is not ready
- **Criteria use "should" instead of "will"** — too vague, rewrite required
- **No non-goals listed** — scope will creep
- **Architecture sketch is missing** — developer is guessing at implementation
- **Zero open questions** — developer is overconfident; probe harder

---

## Completion

When all six sections are solid, Claude produces the spec document.

**Save location**: Read `vault-path` and `project-name` from
`casaflow.config.md`. Create a feature directory and save the spec:

```
~/Documents/<project-name>/<feature-slug>/spec.md
```

Feature slug: lowercase, hyphens, derived from the feature name
(e.g., "Add checkout flow" → `~/Documents/casaflow/add-checkout-flow/spec.md`).

Create the feature directory if it does not exist.

**Mandatory final comprehension check** before handing off:

> "Spec complete. Before we start building: can you describe in your own
> words what happens when [most complex acceptance criterion]? I want to
> make sure the spec reflects your mental model before we write any code."

This check is not optional. Claude does not skip it and does not move on if
the answer is vague.

After the comprehension check passes, invoke the `jira-sync` skill if
configured. Then prompt the developer to start the full pipeline:

> "Spec saved. Run `/casaflow:kickoff` to start the development pipeline —
> it will set up your ticket, branch, design, plan, and guide you through
> the full build."

Do **not** hand off directly to `/build` — the developer needs the full
pipeline (discover → brainstorm → plan → execute → review → ship → learn).

---

## Spec file format

```markdown
# Spec: [Feature Name]

Date: YYYY-MM-DD

## Feature Summary
[One paragraph]

## Acceptance Criteria
1. Given [context], when [action], then [outcome].
2. Given [context], when [action], then [outcome].
...

## Non-Goals
- [What this explicitly does not do]
- [What this explicitly does not do]

## Test Spec

### Criterion 1: [criterion text]
- Happy path: [what success looks like in a test]
- Failure path: [what failure looks like; what the system should do]
- False positive check: [one way this test could pass when it shouldn't]

...

## Architecture Sketch
[Files that change, data flow, pseudocode]

## Open Questions
- [Assumption or unknown]
- [Assumption or unknown]
```
