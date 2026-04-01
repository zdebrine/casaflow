# CasaFlow Configuration

## Team

```yaml
name: <your-team-name>
platform: claude
git-host: github
ticket-system: jira
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
  spec-required: false
task:
  skip: [brainstorm, learn]
  review: light
  spec-required: false
```

## Spec Gate

```yaml
# Work types that require a spec before kickoff proceeds
spec-required-for:
  - feature
  - improvement
spec-dir: specs/
```

## Approval Gates

```yaml
gates-enabled: true

# Default stage definitions (override per project)
stages:
  - name: scaffold
    scope: "Project structure, dependencies, config, CI skeleton"
  - name: backend
    scope: "API endpoints, data layer, auth, validation, security"
  - name: frontend
    scope: "UI components, API integration, error handling, UX"
  - name: polish
    scope: "Tests, docs, edge cases, performance, accessibility"

# Stage 4 (polish) automatically invokes review-tests before gate questions
review-tests-on-polish: true
```

## Jira

```yaml
project-key: <YOUR-PROJECT-KEY>    # e.g. CASA, ENG, PLAT
auto-sync-spec: true               # Invoke jira-sync after spec is saved
auto-update-on-stage: true         # Update ticket status after each approved stage
done-on-merge: true                # Transition ticket to Done on merge
```

## Branching

```yaml
# Include {ticket-id} to pull the key from the Jira ticket
format: "{username}/{ticket-id}-{kebab-title}"
main-branch: main
```

## Concerns Checklist

```yaml
# Skills or specialists invoked during brainstorm based on work type
# Values: path to skill/specialist, or "manual" for human review
error-handling: core/specialists/error-handling
security: core/specialists/security
test-strategy: packs/engineering/skills/eng-testing
# Add team-specific concerns below:
# accessibility: manual
# i18n: manual
# rate-limiting: manual
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
scopes: []                          # Fill in your team's scopes
require-ticket-reference: true      # Include Jira ticket key in commit footer
co-author: true
```

## Retro

```yaml
auto-prompt: true      # Prompt for retro after finish/merge
retro-dir: retros/
```
