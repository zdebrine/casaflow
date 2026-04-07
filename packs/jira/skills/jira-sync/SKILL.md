---
name: jira-sync
description: >
  Use when syncing a feature spec to Jira after it is saved. Handles four
  states — no epic/ticket, epic only, both exist, or dev opts out. Invoked
  automatically by the spec skill after the spec file is saved. Also invoked
  after stage approvals to update ticket status.
tier: workflow
alwaysApply: false
---

# Skill: Jira Sync

## Purpose

This skill runs after a spec is saved. It syncs the spec content to Jira
by creating or updating epics and tickets using the Atlassian MCP server.
It requires explicit developer approval before any Jira action is taken.

Jira sync is always optional and failure is always recoverable. It must
never block the developer from reaching `/build`.

---

## Prerequisites

Before any MCP call, confirm the Atlassian MCP server is available. If
Jira MCP tools are not configured, say:

> "The Atlassian MCP server is not configured. To enable Jira sync, add
> the Atlassian MCP server to your Claude Code settings.json under
> `mcpServers`. Skipping Jira sync and continuing to `/build`."

Then proceed to `/build` with no Jira action.

---

## Step 1 — Ask whether to sync

> "Spec saved. Do you want to sync this spec to Jira? (yes / skip)"

- **skip** → "Skipping Jira sync. Returning to the calling skill." Stop here.
- **yes** → Continue to Step 2.

---

## Step 2 — Gather Jira context

Ask in a single message:

> "Two quick questions:
> 1. What is the Jira project key? (e.g. CASA, ENG, PLAT)
> 2. Do you have an existing epic key, a ticket key, both, or neither?
>    - Neither → I'll create both
>    - Epic only → provide the epic key, I'll create the ticket
>    - Both → provide both keys, I'll update the ticket"

Parse the response to determine which state applies (A, B, or C below).

---

## Step 3 — Generate summary

Generate a concise summary from the saved spec:

- Feature name (one line)
- What it does and who uses it (2–3 sentences)
- Acceptance criteria as a numbered list (one line each)
- Non-goals as a bulleted list (one line each)

This summary becomes the Jira description. The full spec markdown becomes
a comment on the ticket.

---

## Step 4 — Show preview and get approval

Present before any MCP call:

> "Here is what I will push to Jira:
>
> **Action:** [create epic + ticket / create ticket under [EPIC-KEY] /
> update [TICKET-KEY]]
> **Project:** [PROJECT-KEY]
>
> **Description:**
> [generated summary]
>
> **Comment:** Full contents of `~/Documents/<project-name>/<feature-slug>/spec.md` added as a comment.
>
> Approve this push? (yes / skip)"

- **skip** → "Skipping Jira sync. Returning to the calling skill." Stop here.
- **yes** → Continue to Step 5.

---

## Step 5 — Execute Jira action

### State A — Neither epic nor ticket exists

1. Create epic: project key, feature name as summary, type Epic,
   description: generated summary
2. Create ticket: same project, feature name, type Story (or Task if Story
   unavailable), description: generated summary, parent: epic key from step 1
3. Add comment to ticket: full spec markdown

### State B — Epic exists, no ticket

1. Create ticket: project key, feature name, type Story/Task, description:
   generated summary, parent: developer-provided epic key
2. Add comment to ticket: full spec markdown

### State C — Both exist

1. Update ticket description: generated summary
2. Add comment to ticket: full spec markdown

---

## Step 6 — Verify

After each MCP create or update call, read back the affected issue by key.

Confirm independently:
- Description matches generated summary
- Comment exists and contains the spec markdown

**Description correct, comment missing:**
> "The ticket description was synced successfully, but the comment could
> not be confirmed. Paste the contents of `~/Documents/<project-name>/<feature-slug>/spec.md` into the
> ticket comment manually to stay consistent with other tickets."
Proceed to Step 7 for the description success.

**Description missing or wrong:** Fall through to the failure path.

**Both correct:** Proceed to Step 7.

---

## Step 7 — Confirm and hand off

Output only the keys that were created or updated:

> "Jira sync complete. [Epic: EPIC-KEY / ] Ticket: TICKET-KEY"

Then immediately invoke `/build` without waiting for further input.

---

## Stage update behavior (post-spec)

After each approved build stage, optionally update the Jira ticket status.
Check `casaflow.config.md` for `jira.auto-update-on-stage`.

If enabled:
- Stage 1 complete → transition to "In Progress"
- Stage 4 complete → transition to "In Review"
- PR created → add PR URL as comment on ticket
- Merge confirmed → transition to "Done" (if `jira.done-on-merge: true`)

Status transitions use the MCP `transitionIssue` tool. If the transition
fails, surface the failure but do not block the pipeline.

---

## Failure path

If any MCP call fails:

1. State clearly what failed:
   > "The Jira sync failed — [epic / ticket / comment] was not created
   > or updated successfully."

2. Output the summary for manual entry:
   > "Here is the summary to paste into the Jira description manually:"
   > [generated summary]

3. Remind to add the spec as a comment:
   > "For the ticket comment, paste the contents of `~/Documents/<project-name>/<feature-slug>/spec.md`."

4. Return control to the calling skill without waiting for further input.

---

## Edge case handling

**Empty or missing spec file**: Confirm the spec file exists and is non-empty
before Step 3. If missing: skip Jira sync, proceed to `/build`.

**Invalid project key**: If the MCP returns a key-not-found error, allow
one re-entry. If the second attempt also fails, fall through to failure path.

**Special characters in feature name**: Pass as-is — the MCP handles
encoding. If the MCP rejects the name, surface the error and use failure path.

---

## What this skill never does

- Never populates Jira fields beyond description and comment (no assignee,
  priority, due date, labels, sprint, story points)
- Never reads from or writes to Jira without explicit dev approval (Step 4)
- Never retries a failed MCP call — surface and move on
- Never blocks the developer from reaching `/build`
