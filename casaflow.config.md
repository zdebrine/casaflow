# CasaFlow Configuration

## Team

```yaml
name: casaflow
platform: claude
git-host: github
ticket-system: github
# ticket-prefix: CASAFLOW
```

## Pipeline

```yaml
stages:
  - discover
  - brainstorm
  - plan
  - execute
  - review
  - ship
  - learn
```

### Stage Overrides by Work Type

```yaml
bug:
  skip: [brainstorm-full, learn]
  brainstorm: light
task:
  skip: [brainstorm, learn]
  review: light
```

## Documentation

```yaml
vault-path: ~/Documents
project-name: casaflow
# Specs and plans are saved outside the repo in an Obsidian vault.
# Structure: ~/Documents/<project-name>/<feature-slug>/
#   spec.md, plan.md, design.md, prd.md
# This keeps markdown artifacts out of the codebase while remaining
# accessible to all team members via shared Obsidian vault.
```

## Branching

```yaml
format: "{username}/casaflow-{number}-{kebab-title}"
main-branch: main
```

## Concerns Checklist

```yaml
- skill-schema: team/specialists/skill-quality
- error-handling: core/specialists/error-handling
- security: core/specialists/security
- test-strategy: manual
```

## Review

```yaml
swarm-tiers:
  fast-pass: [security, dead-code, error-handling]
  full: all
deep-review-model: opus
specialist-model-default: haiku
```

## Execution

```yaml
parallel-threshold: 3
default-strategy: team-dev
teammate-mode: tmux
```

## Commit

```yaml
convention: conventional
format: "type(scope): message"
types: [feat, fix, docs, chore, refactor, test]
scopes: [core, framework, packs, adapters, scaffold, docs, agents, specialists]
require-ticket-reference: false
co-author: true
```
