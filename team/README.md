# CasaFlow Team Overrides

This directory contains CasaPerks-specific customizations layered on top of
the Jig framework core. Skills here take priority over `core/` and `packs/`
by the Jig discovery system.

## What lives here

| Skill | Overrides | What it adds |
|-------|-----------|-------------|
| `kickoff` | `core/skills/kickoff` | Spec-first gate for features/improvements |
| `spec` | *(new)* | Guides developer through writing a feature spec |
| `approve-gate` | *(new)* | Comprehension check after each build stage |
| `explain` | *(new)* | 5-section education-focused code explanation |
| `review-tests` | *(new)* | 4-phase test audit with mutation testing |
| `retro` | *(new)* | Post-feature retrospective and learning capture |

## Philosophy

CasaFlow optimizes for four things:

1. **Speed** — Jig's parallel execution, task graphs, and PR automation
2. **Dev comprehension** — Approval gates with 3 mandatory comprehension
   questions per build stage
3. **Code robustness** — Review swarm (5 specialists), TDD discipline,
   mutation testing
4. **Developer education** — Explain command, failure-mode docs, retro
   artifacts

## Adding team overrides

Create `team/skills/{name}/SKILL.md`. If `{name}` matches a core or pack
skill name, the team version replaces it. Otherwise it's a new addition.

See `framework/SKILL_SCHEMA.md` for the frontmatter spec.
