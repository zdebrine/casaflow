<p>
  <img src="assets/jig.png" alt="CasaFlow" width="100">
</p>

**The CasaPerks AI engineering workflow — spec-first, comprehension-gated, Jira-native.**

CasaFlow is built on the [Jig](https://github.com/duronext/jig) framework and tuned for CasaPerks engineering culture. It guides AI agents through a structured pipeline — from Jira ticket to post-mortem — with quality gates at every stage that keep developers in comprehension of what they're building.

**Four priorities**: Speed · Dev comprehension · Code robustness · Developer education

---

## Prerequisites

Before installing CasaFlow, make sure you have:

1. **Claude Code** installed and working ([install guide](https://docs.claude.com)). You need to be on the CasaPerks Team plan.
2. **A CasaPerks repo** you want to use CasaFlow on (or this repo itself for contributing).
3. **Git**

Optional but recommended:

4. **Jira API token** — if your team uses Jira for ticket tracking. Get one at https://id.atlassian.com/manage-profile/security/api-tokens
5. **Agent teams enabled** — for parallel build execution, set `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS: "1"` in your Claude Code settings.

---

## Installation

### Option A: Project settings (recommended)

This is the preferred method. It means every engineer who clones the repo gets CasaFlow automatically — no manual setup.

Create or edit `.claude/settings.json` in your project root:

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

Commit that file. Done. The next time any teammate opens the project in Claude Code, they'll be prompted to enable the CasaFlow plugin. Once they accept, all `/casaflow:` commands are available.

### Option B: Manual install via Claude Code CLI

If you'd rather install per-developer instead of per-project:

```
/plugin marketplace add CasaPerks/casaflow
/plugin install casaflow@casaperks-casaflow --scope project
```

Commit that file. Every teammate who opens the project in Claude Code gets CasaFlow automatically — no manual setup, no plugin install.

### Verify it worked

Open Claude Code in your project and type `/casaflow:` — you should see the full list of commands autocomplete. If nothing shows up, run `/plugins` to check that CasaFlow is listed and enabled.

---

## Project Configuration

CasaFlow reads its settings from `casaflow.config.md` in your project root. If the file doesn't exist, CasaFlow uses sensible defaults, but you'll want to configure at least your team name and ticket system.

Copy the template:

```bash
cp .claude-plugin-source/scaffold/casaflow.config.md casaflow.config.md
```

Or create it manually with these essentials:

```markdown
## Team

name: your-team-name
platform: claude
git-host: github
ticket-system: jira # or "github" for GitHub Issues

## Jira

project-key: CASA # your Jira project key

## Branching

format: "{username}/{ticket-id}-{kebab-title}"
main-branch: main

## Execution

parallel-threshold: 3 # min independent tasks before parallel build
default-strategy: team-dev # team-dev (parallel) or sdd (serial)
```

See [scaffold/casaflow.config.md](scaffold/casaflow.config.md) for the full config reference with all available options.

---

## Jira Sync Setup (Optional)

CasaFlow can sync specs to Jira, create tickets, update statuses, and post PR URLs as comments. This requires the Atlassian MCP server.

Each developer configures this in their **personal** Claude settings (`~/.claude/settings.json`), not the project settings:

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

If Jira isn't configured, CasaFlow skips sync gracefully and continues the pipeline. It never blocks on a missing Jira connection.

---

## Your First Feature

Here's the typical flow for building a new feature with CasaFlow, start to finish.

### Step 1: Write the spec

```
/casaflow:spec
```

Claude will walk you through six sections: feature summary, acceptance criteria, non-goals, test spec, architecture sketch, and open questions. You write the spec — Claude asks questions and pushes back until each section is solid. At the end, you'll answer a comprehension check to confirm you understand the most complex acceptance criterion.

The spec saves to `specs/<feature-slug>.md`.

### Step 2: Kick off the pipeline

```
/casaflow:kickoff
```

This is the pipeline orchestrator. It classifies your work type (bug/feature/improvement/task), checks for the spec (features and improvements are blocked without one), and then walks through each stage: discover, brainstorm, plan, execute, review, ship, learn.

You don't need to run each stage manually — kickoff orchestrates the handoffs.

### Step 3: Build

If you want to jump straight to building from an existing plan:

```
/casaflow:build
```

Build finds your most recent plan in `docs/plans/`, analyzes the task graph, and automatically decides whether to run tasks in parallel (team-dev) or serial (sdd). After each stage, you'll face the approval gate — three comprehension questions you must answer before the next stage begins.

### Step 4: Review and ship

```
/casaflow:review
```

Dispatches the specialist swarm (security, dead-code, error-handling, async-safety, performance) in parallel, scores findings mechanically, and produces a unified report. Fix any Critical or Major findings, then:

```
/casaflow:pr-create
```

Creates the PR with a structured description and test plan.

---

## Command Reference

### Core Pipeline

| Command                | What It Does                                                                           |
| ---------------------- | -------------------------------------------------------------------------------------- |
| `/casaflow:kickoff`    | Start the full pipeline. Classifies work, enforces spec gate, orchestrates all stages. |
| `/casaflow:spec`       | Write a feature spec (6 sections). Claude asks questions — doesn't write it for you.   |
| `/casaflow:brainstorm` | Collaborative design exploration. Hard gate: no code until design is approved.         |
| `/casaflow:plan`       | Turn a design into an implementation plan with bite-sized TDD tasks.                   |
| `/casaflow:build`      | Execute a plan. Auto-selects parallel or serial strategy.                              |
| `/casaflow:review`     | Dispatch the specialist review swarm. Produces scored report.                          |
| `/casaflow:pr-create`  | Create a PR with voice/tone standards and test plan.                                   |
| `/casaflow:pr-respond` | Handle PR feedback: analyze, fix, commit, push, reply, resolve.                        |
| `/casaflow:finish`     | Branch completion — merge, PR, keep, or discard.                                       |

### Quality Gates

| Command                  | What It Does                                                                        |
| ------------------------ | ----------------------------------------------------------------------------------- |
| `/casaflow:approve`      | Stage report + 3 comprehension questions (structural, failure mode, change impact). |
| `/casaflow:review-tests` | 4-phase test audit: coverage, mutation testing, rubric, letter grade.               |
| `/casaflow:verify`       | Evidence before assertions — run it before claiming it works.                       |

### Education

| Command                | What It Does                                                                     |
| ---------------------- | -------------------------------------------------------------------------------- |
| `/casaflow:explain`    | 5-section explanation: approach, failure modes, change surface, tests, refactor. |
| `/casaflow:retro`      | Post-feature retrospective with pattern detection across retros.                 |
| `/casaflow:postmortem` | Post-merge analysis with specialist/logic reviewer diagnosis.                    |

### Development Practices

| Command            | What It Does                                            |
| ------------------ | ------------------------------------------------------- |
| `/casaflow:debug`  | Systematic debugging — root cause before fixes, always. |
| `/casaflow:tdd`    | Red-green-refactor discipline.                          |
| `/casaflow:prd`    | PRD creation with enforceable acceptance checklists.    |
| `/casaflow:ticket` | Create or look up a ticket.                             |

### Engineering Standards

| Command                     | What It Does                         |
| --------------------------- | ------------------------------------ |
| `/casaflow:eng-copywriting` | Sentence case standards for UI copy. |
| `/casaflow:eng-logging`     | Log level guidance.                  |
| `/casaflow:eng-testing`     | Test strategy guidance.              |

### Framework

| Command            | What It Does                                                 |
| ------------------ | ------------------------------------------------------------ |
| `/casaflow:extend` | Scaffold a new skill, specialist, or pack for the framework. |

---

## How the Pipeline Works

Every feature flows through these stages. The spec gate and approval gates are CasaFlow additions on top of the Jig core:

```
[SPEC] → DISCOVER → BRAINSTORM → PLAN → EXECUTE → REVIEW → SHIP → LEARN
  ↑                                          ↑
  Hard gate for features              Approve-gate after
  (no code without a spec)            every build stage
```

Work type determines depth:

| Work Type   | Spec Gate | Brainstorm                     | Review     | Learn    |
| ----------- | --------- | ------------------------------ | ---------- | -------- |
| Feature     | Required  | Full (with concerns checklist) | Full swarm | Always   |
| Improvement | Required  | Medium (2-3 approaches)        | Full swarm | Always   |
| Bug         | Skipped   | Light (root cause + fix)       | Full swarm | Optional |
| Task/Chore  | Skipped   | Skipped                        | Light      | Skipped  |

### The Approval Gate

Every build stage ends with two things:

1. **Stage report** — files changed (with coupling analysis), how to run the app, manual test checklist, what was deferred, and tradeoffs made.
2. **Three comprehension questions** — structural ("walk me through the interaction"), failure mode ("what happens if X breaks?"), and change impact ("if we changed Y, what else needs touching?"). You must answer all three before the next stage begins.

This is intentionally uncomfortable. The discomfort is the learning.

---

## Architecture

CasaFlow uses Jig's three-layer discovery system:

```
team/      ← CasaPerks overrides (spec gate, approval gates, education skills)
packs/     ← Jira pack, engineering pack
core/      ← Jig framework defaults (pipeline, review swarm, agents)
```

When a team skill has the same name as a core skill, the team version wins. CasaPerks overrides `kickoff` to add the spec gate — the Jig core stays untouched.

```
casaflow/
├── team/skills/           6 CasaPerks skills (spec, kickoff override, approve-gate, explain, review-tests, retro)
├── core/skills/           16 pipeline skills (brainstorm, plan, build, review, pr-create, etc.)
├── core/agents/           3 agents (commit, code-review, pr-review)
├── core/specialists/      5 review specialists (security, dead-code, error-handling, async-safety, performance)
├── packs/engineering/     Engineering standards (copywriting, logging, testing)
├── packs/jira/            Jira sync integration
├── commands/              Slash command definitions for all skills
├── framework/             How Jig works (pipeline, discovery, schema, tiers)
├── casaflow.config.md     Pipeline configuration
└── CLAUDE.md              Project context for Claude
```

---

## Updating CasaFlow

When a new version is pushed to the CasaFlow repo:

```
/plugin marketplace update casaperks-casaflow
/plugin install casaflow@casaperks-casaflow --scope project
```

If you used Option A (project settings), teammates get updates automatically the next time they open the project.

---

## Troubleshooting

**Commands don't show up when I type `/casaflow:`**

Run `/plugins` in Claude Code and check that CasaFlow is listed and enabled. If it's listed but disabled, run `/plugin install casaflow@casaperks-casaflow --scope project` to re-enable it.

**Claude doesn't follow the command workflow**

Each command contains detailed instructions that Claude must follow. If Claude freestyles instead of following the structured workflow, try running the command again in a fresh conversation. Long conversations can dilute instruction adherence.

**Jira sync fails or hangs**

Check that your Atlassian MCP server is configured in `~/.claude/settings.json` (not the project settings). Verify your API token is valid. CasaFlow will skip Jira sync if the MCP server isn't available — it never blocks the pipeline.

**Build chooses serial when I expected parallel**

Build auto-selects based on the task graph. It uses serial (sdd) when tasks share files or there aren't enough independent tasks to justify parallel overhead. Check `casaflow.config.md` for the `parallel-threshold` setting (default: 3). Also verify agent teams are enabled: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS: "1"` in Claude Code settings.

**Spec gate blocks me but I don't want to write a spec**

The spec gate only applies to features and improvements. If you're doing a bug fix or task, classify it accordingly during kickoff and the gate is skipped. For features, the spec is non-negotiable — it's the mechanism that keeps you in comprehension of what Claude builds.

---

## Guides

| Guide                                                                        | What It Covers                                                                         |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| [Getting the Most Out of CasaFlow](docs/getting-the-most-out-of-casaflow.md) | Philosophy, full pipeline walkthrough, and how to engage every stage for maximum value |

## Framework Reference

CasaFlow is built on Jig. The underlying framework docs:

| Document                                                   | What It Covers                      |
| ---------------------------------------------------------- | ----------------------------------- |
| [framework/PIPELINE.md](framework/PIPELINE.md)             | The 7-stage development pipeline    |
| [framework/DISCOVERY.md](framework/DISCOVERY.md)           | How skills are found and loaded     |
| [framework/SKILL_SCHEMA.md](framework/SKILL_SCHEMA.md)     | Frontmatter spec for writing skills |
| [framework/TIER_SYSTEM.md](framework/TIER_SYSTEM.md)       | How tiers control skill activation  |
| [team/README.md](team/README.md)                           | CasaFlow overrides guide            |
| [scaffold/casaflow.config.md](scaffold/casaflow.config.md) | Config template                     |

---

## License

MIT
