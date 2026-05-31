# Agentic Workflow — Agent Instructions

## Repo overview

This is a 100%-markdown meta-agent project. **The Architect** (`the-architect/`) is a Claude Code/OpenCode agent that interviews users about software ideas and produces self-contained build blueprints. There is no code to build, test, lint, or deploy.

- No `package.json`, `Makefile`, `Dockerfile`, CI config, or test framework.
- No package manager, no runtime dependencies.
- Everything is markdown — edit directly.

## Branch & git flow

- Default working branch is **`develop`** (HEAD points here, not `main`).
- Git Flow initialized: `bugfix/<name>` branches off `develop`, `release/<name>` branches off `main` → merged back to both.

## Directory structure

| Path | Purpose |
|------|---------|
| `the-architect/CLAUDE.md` | **Main entrypoint** — the instruction file that transforms an AI agent into "The Architect". Always read this first. |
| `the-architect/knowledge/archetypes/` | 6 project-type templates (saas-webapp, marketing-site, mobile-app, api-backend, internal-tool, content-platform) |
| `the-architect/knowledge/building-blocks/` | Decision guides (auth, database, deployment, API design, frontend stacks, testing, styling, state management) |
| `the-architect/knowledge/skills-registry.md` | Maps AI skills to blueprint sections |
| `the-architect/knowledge/stack-compatibility.md` | Tech stack compatibility reference |
| `the-architect/templates/` | Blueprint output skeleton + target-project CLAUDE.md template |
| `the-architect/questions/` | 3-phase interview question flows (phase-1-discovery, phase-2-branches, phase-3-confirmation) |
| `the-architect/output/` | Generated blueprints land here (`.gitkeep` in place) |

## Workflow (must follow in order)

1. **Phase 1: Discovery** — ask 2-3 questions, classify into archetype, read matching archetype file
2. **Phase 2: Deep Dive** — ask 3-5 archetype-specific questions, read building blocks as needed
3. **Phase 3: Architecture** — present opinionated tech stack + rationale, get confirmation before generating
4. **Phase 4: Generate** — read `templates/blueprint-template.md`, `templates/claude-md-template.md`, `knowledge/skills-registry.md`, then write to `output/<project-name>-blueprint.md`

## Key constraints (from CLAUDE.md)

- **Max 3 questions per message** — never dump all at once
- **Blueprint must be self-contained** — another agent with zero context must build from it
- **Must include numbered build order** (most critical section)
- **Must include a complete CLAUDE.md** for the target project inside the blueprint
- **Detect user language** from their first message; use it for all interaction
- **Be opinionated** — recommend one stack with rationale, not a menu of options
- **Fast-track mode**: if user says "just build it", ask only 3 essential questions + smart defaults

## Skills available for use

| Skill | When |
|-------|------|
| `/deep-research` | Comparing unfamiliar technologies |
| `/ui-ux-pro-max` | Designing visual system (frontend projects) |
| `/find-skills` | Discovering build-phase skills to recommend |
| `/playwright-cli` or `/chrome-bridge-automation` | Analyzing reference sites |

Do NOT use `/frontend-design` or `/shadcn-ui` during design — recommend them in the blueprint for the build phase.

## Generated output

- Always save to `the-architect/output/<project-name>-blueprint.md`
