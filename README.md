<p>
  <img src="assets/jig.png" alt="CasaFlow" width="100">
</p>

**The CasaPerks AI engineering workflow — spec-first, comprehension-gated, Jira-native.**

CasaFlow is built on the [Jig](https://github.com/duronext/jig) framework and tuned for CasaPerks engineering culture. It guides AI agents through a structured pipeline — from Jira ticket to post-mortem — with quality gates at every stage that keep developers in comprehension of what they're building.

**Four priorities**: Speed · Dev comprehension · Code robustness · Developer education

---

## Why CasaFlow

Most AI dev tools optimize for speed. CasaFlow optimizes for **comprehension first, speed second** — because an engineer who doesn't understand their own codebase will accumulate debt faster than any AI can generate it.

Every feature flows through a structured pipeline with two hard gates: a **spec gate** (no code without a spec) and **approval gates** (three comprehension questions after every build stage). These gates are intentionally uncomfortable. The discomfort is the learning.

CasaFlow's workflow is designed so that cutting corners on comprehension is harder than just doing the work. Lean into it.

---

## Installation

### Step 1: Add the settings file to your repo

Create `.claude/settings.json` in your project root and commit it. This way every engineer who clones the repo gets the marketplace declaration automatically.

```json
{
  "enabledPlugins": {
    "casaflow@casaperks-casaflow": true
  },
  "extraKnownMarketplaces": {
    "casaperks-casaflow": {
      "source": {
        "source": "github",
        "repo": "CasaPerks/casaflow"
      }
    }
  }
}
```

### Step 2: Install the plugin from the terminal

Open a terminal, `cd` into your project, and launch the Claude Code CLI:

```bash
cd /path/to/your-project
claude
```

Run these two commands inside the Claude Code REPL:

```
/plugin marketplace add https://github.com/CasaPerks/casaflow.git
/plugin install casaflow@casaperks-casaflow --scope project
```

Then restart your Claude session (`/exit` and run `claude` again).

**Three things that will trip you up if you skip them:**

1. **Use the terminal CLI, not VS Code.** The `/plugin` commands must be run in the terminal (`claude`). They don't work from the VS Code chat panel. After installing from the terminal, VS Code picks up the plugin automatically.
2. **Use the full HTTPS URL.** The shorthand `CasaPerks/casaflow` tries SSH and will fail unless you have SSH keys configured for GitHub. Always use `https://github.com/CasaPerks/casaflow.git`.
3. **Restart the session after install.** Commands won't appear until you start a new Claude session. In VS Code, reload the window (Cmd+Shift+P → "Reload Window").

### Step 3: Verify

Type `/casaflow:` — you should see the full list of commands autocomplete. You can also run `/plugins` to confirm CasaFlow is listed and enabled.

---

## Your First Feature

### 1. Write the spec

```
/casaflow:spec
```

Claude walks you through six sections: feature summary, acceptance criteria, non-goals, test spec, architecture sketch, and open questions. You write the spec — Claude asks questions and pushes back until each section is solid. At the end, you answer a comprehension check to confirm you understand the most complex acceptance criterion. The spec saves to `specs/<feature-slug>.md`.

### 2. Kick off the pipeline

```
/casaflow:kickoff
```

The pipeline orchestrator. It classifies your work type (bug/feature/improvement/task), checks for the spec (features and improvements are blocked without one), and walks through each stage: discover, brainstorm, plan, execute, review, ship, learn. You don't need to run each stage manually — kickoff orchestrates the handoffs.

### 3. Build

```
/casaflow:build
```

If you have an existing plan and want to jump straight to building. Build finds your most recent plan in `docs/plans/`, analyzes the task graph, and decides whether to run tasks in parallel or serial. After each stage, you'll face the approval gate — three comprehension questions you must answer before the next stage begins.

### 4. Review and ship

```
/casaflow:review
/casaflow:pr-create
```

Review dispatches the specialist swarm (security, dead-code, error-handling, async-safety, performance) in parallel, scores findings mechanically, and produces a unified report. Fix any Critical or Major findings, then pr-create writes the PR with a structured description and test plan.

---

## Command Reference

### Core Pipeline

| Command | What It Does |
|---------|-------------|
| `/casaflow:kickoff` | Start the full pipeline. Classifies work, enforces spec gate, orchestrates all stages. |
| `/casaflow:spec` | Write a feature spec (6 sections). Claude asks questions — doesn't write it for you. |
| `/casaflow:brainstorm` | Collaborative design exploration. Hard gate: no code until design is approved. |
| `/casaflow:plan` | Turn a design into an implementation plan with bite-sized TDD tasks. |
| `/casaflow:build` | Execute a plan. Auto-selects parallel or serial strategy. |
| `/casaflow:review` | Dispatch the specialist review swarm. Produces scored report. |
| `/casaflow:pr-create` | Create a PR with voice/tone standards and test plan. |
| `/casaflow:pr-respond` | Handle PR feedback: analyze, fix, commit, push, reply, resolve. |
| `/casaflow:finish` | Branch completion — merge, PR, keep, or discard. |

### Quality Gates

| Command | What It Does |
|---------|-------------|
| `/casaflow:approve` | Stage report + 3 comprehension questions (structural, failure mode, change impact). |
| `/casaflow:review-tests` | 4-phase test audit: coverage, mutation testing, rubric, letter grade. |
| `/casaflow:verify` | Evidence before assertions — run it before claiming it works. |

### Education

| Command | What It Does |
|---------|-------------|
| `/casaflow:explain` | 5-section explanation: approach, failure modes, change surface, tests, refactor. |
| `/casaflow:retro` | Post-feature retrospective with pattern detection across retros. |
| `/casaflow:postmortem` | Post-merge analysis with specialist/logic reviewer diagnosis. |

### Development Practices

| Command | What It Does |
|---------|-------------|
| `/casaflow:debug` | Systematic debugging — root cause before fixes, always. |
| `/casaflow:tdd` | Red-green-refactor discipline. |
| `/casaflow:prd` | PRD creation with enforceable acceptance checklists. |
| `/casaflow:ticket` | Create or look up a ticket. |

### Engineering Standards

| Command | What It Does |
|---------|-------------|
| `/casaflow:eng-copywriting` | Sentence case standards for UI copy. |
| `/casaflow:eng-logging` | Log level guidance. |
| `/casaflow:eng-testing` | Test strategy guidance. |

---

## How the Pipeline Works

```
[SPEC] → DISCOVER → BRAINSTORM → PLAN → EXECUTE → REVIEW → SHIP → LEARN
  ↑                                          ↑
  Hard gate for features              Approve-gate after
  (no code without a spec)            every build stage
```

Work type determines depth:

| Work Type | Spec Gate | Brainstorm | Review | Learn |
|-----------|-----------|------------|--------|-------|
| Feature | Required | Full (with concerns checklist) | Full swarm | Always |
| Improvement | Required | Medium (2-3 approaches) | Full swarm | Always |
| Bug | Skipped | Light (root cause + fix) | Full swarm | Optional |
| Task/Chore | Skipped | Skipped | Light | Skipped |

---

## Configuration

CasaFlow reads settings from `casaflow.config.md` in your project root. If the file doesn't exist, sensible defaults are used.

Create one from the template:

```bash
cp .claude-plugin-source/scaffold/casaflow.config.md casaflow.config.md
```

Or create it manually with the essentials:

```markdown
## Team
name: your-team-name
platform: claude
git-host: github
ticket-system: jira

## Jira
project-key: CASA

## Branching
format: "{username}/{ticket-id}-{kebab-title}"
main-branch: main
```

See [scaffold/casaflow.config.md](scaffold/casaflow.config.md) for all available options.

### Jira Sync (Optional)

CasaFlow can sync specs to Jira, create tickets, update statuses, and post PR URLs as comments. Each developer configures this in their **personal** Claude settings (`~/.claude/settings.json`):

```json
{
  "mcpServers": {
    "atlassian": {
      "command": "npx",
      "args": ["-y", "@atlassian/mcp-atlassian"],
      "env": {
        "ATLASSIAN_URL": "https://your-org.atlassian.net",
        "ATLASSIAN_EMAIL": "you@casaperks.com",
        "ATLASSIAN_API_TOKEN": "your-api-token"
      }
    }
  }
}
```

Get your API token at https://id.atlassian.com/manage-profile/security/api-tokens. If Jira isn't configured, CasaFlow skips sync gracefully — it never blocks the pipeline.

---

## Updating

When a new version of CasaFlow is pushed, run in the terminal CLI:

```
/plugin marketplace update casaperks-casaflow
/plugin install casaflow@casaperks-casaflow --scope project
```

---

## Troubleshooting

**Commands don't show up when I type `/casaflow:`**

Make sure you've run the plugin install from the **terminal CLI** (not VS Code) — see Installation above. The `.claude/settings.json` file alone doesn't install the plugin. After installing, restart your Claude session. In VS Code, reload the window (Cmd+Shift+P → "Reload Window").

**SSH authentication error when adding marketplace**

Use the full HTTPS URL: `/plugin marketplace add https://github.com/CasaPerks/casaflow.git`. The shorthand form uses SSH by default.

**Claude doesn't follow the command workflow**

Try running the command again in a fresh conversation. Long conversations can dilute instruction adherence.

**Build chooses serial when I expected parallel**

Build auto-selects based on the task graph. It uses serial when tasks share files or there aren't enough independent tasks. Check `parallel-threshold` in `casaflow.config.md` (default: 3). Also verify agent teams are enabled: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS: "1"` in Claude Code settings.

**Spec gate blocks me but I don't want to write a spec**

The spec gate only applies to features and improvements. Classify your work as a bug or task during kickoff and the gate is skipped. For features, the spec is non-negotiable.

---

## Guides

| Guide | What It Covers |
|-------|---------------|
| [Getting the Most Out of CasaFlow](docs/getting-the-most-out-of-casaflow.md) | Philosophy, full pipeline walkthrough, and how to engage every stage for maximum value |

## Framework Reference

| Document | What It Covers |
|----------|---------------|
| [framework/PIPELINE.md](framework/PIPELINE.md) | The 7-stage development pipeline |
| [framework/DISCOVERY.md](framework/DISCOVERY.md) | How skills are found and loaded |
| [framework/SKILL_SCHEMA.md](framework/SKILL_SCHEMA.md) | Frontmatter spec for writing skills |
| [framework/TIER_SYSTEM.md](framework/TIER_SYSTEM.md) | How tiers control skill activation |
| [scaffold/casaflow.config.md](scaffold/casaflow.config.md) | Config template |

---

## License

MIT
