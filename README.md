<p align="center">
  <img alt="OpenCode" src="https://img.shields.io/badge/OpenCode-agentic--workflow-6C47FF?style=flat-square">
  <img alt="Status" src="https://img.shields.io/badge/status-active-success?style=flat-square">
</p>

<h1 align="center">Agentic Workflow Setup</h1>
<p align="center"><strong>Personal agentic workflow for building software with OpenCode — a config/doc repo of agent definitions and architecture blueprints.</strong></p>

This repository defines a team of AI agents that collaborate to plan, build, verify, review, and commit software projects. It is a **configuration and documentation repository** — no application code, no build scripts, no test runners. Everything is markdown with YAML frontmatter that OpenCode loads as executable agent definitions.

## Table of Contents

- [Installation](#installation)
- [Architecture](#architecture)
- [Directory Structure](#directory-structure)
- [The Architect System](#the-architect-system)
- [Usage](#usage)
- [Key Details](#key-details)
- [Relationship to AGENTS.md](#relationship-to-agentsmd)

---

## Installation

Clone this repository and copy the agent definitions and resources to your OpenCode config directory:

```bash
git clone https://github.com/jfaundez07/agentic-workflow.git
cp -r agentic-workflow/agents ~/.config/opencode/agents
cp -r agentic-workflow/resources ~/.config/opencode/resources
```

> **Note:** Paths in agent instructions reference `~/.config/opencode/...` at runtime, so copying these directories to that location ensures everything resolves correctly.

---

## Architecture

The workflow is organized into two categories of agents: **primary agents** (run standalone) and **sub-agents** (dispatched by primaries).

### Primary Agents

| Agent | File | Role |
|-------|------|------|
| **Tech Lead** | `agents/tech-lead.md` | Orchestrator — communicates with user, delegates to sub-agents, verifies output. Dispatches in workflow steps. |
| **The Architect** | `agents/the-architect.md` | System designer — generates buildable blueprints from scratch. Does NOT write code. |

### Sub-Agents

| Agent | File | Responsibility |
|-------|------|----------------|
| **Analyst** | `agents/analyst.md` | Deep technical analysis of any project — tech stack, architecture, APIs, dependencies, security, DevOps, and technical debt identification. Produces `.opencode/docs/analysis.md`. |
| **Designer** | `agents/designer.md` | Creates detailed implementation plans (requirements, acceptance criteria, build order) at `.opencode/docs/plan.md`. |
| **Developer** | `agents/developer.md` | Implements code per specification — writes source files, tests, configurations. |
| **QA** | `agents/qa.md` | Designs the test strategy and plan — identifies risk areas, maps acceptance criteria to test cases, and produces `.opencode/docs/test-plan.md`. Does **not** write or execute tests. |
| **Reviewer** | `agents/reviewer.md` | Reviews code for correctness, security, performance, style, and adherence to spec. Pass/fail gate with BLOCKER/WARNING/NIT ratings. |
| **Commiter** | `agents/commiter.md` | Analyzes workspace diff and generates a Conventional Commit message with alternatives. |

### Pipeline Flow

The Tech Lead orchestrates sub-agents sequentially. Users compose their own workflow by selecting from **5 composable steps** in logical order:

| Step | Responsibility | Output |
|------|---------------|--------|
| **Designer** | Explores the project, defines requirements, acceptance criteria, and build order | `.opencode/docs/plan.md` |
| **Developer** | Implements the plan — writes all source code and tests | Working implementation |
| **QA** | Designs the test strategy — scope, test cases per AC, risk areas, edge cases | `.opencode/docs/test-plan.md` |
| **Reviewer** | Audits code quality with pass/fail gate (BLOCKER/WARNING/NIT) | Review report |
| **Commiter** | Generates a conventional commit message for the user | Commit message |

Steps must follow the logical order (Designer → Developer → QA → Reviewer → Commiter). The user may pick any subset — e.g., Designer + Developer only, or skip Commiter. A step cannot be included without its prerequisites (Developer requires Designer; QA and Reviewer require Developer).

After each step, the Tech Lead verifies the output and passes structured context to the next agent. The pipeline stops early if any stage fails verification. If the same sub-agent fails twice consecutively, the Tech Lead escalates to the user.

---

## Directory Structure

```
.
├── README.md                    # This file — user-facing introduction
├── agents/                      # Agent definitions (8 agents)
│   ├── analyst.md
│   ├── commiter.md
│   ├── designer.md
│   ├── developer.md
│   ├── qa.md
│   ├── reviewer.md
│   ├── tech-lead.md
│   └── the-architect.md
├── docs/                        # Extended documentation
│   └── the-architect.md
└── resources/                   # Knowledge base for The Architect
    └── the-architect/
        ├── knowledge/
        │   ├── archetypes/      # 6 project-type templates
        │   ├── building-blocks/ # 8 decision-guide resources
        │   └── skills-registry.md
        ├── questions/           # Discovery phase questionnaires
        └── templates/           # Blueprint & AGENTS.md templates

```

| Path | Purpose |
|------|---------|
| `agents/` | Agent definitions — 8 markdown files with YAML frontmatter (analyst, designer, developer, qa, reviewer, commiter, tech-lead, the-architect) |
| `docs/` | Documentation — notably `docs/the-architect.md` for the blueprint system |
| `resources/the-architect/` | Knowledge base for The Architect: archetypes, building blocks, templates, and discovery questions |
| `resources/the-architect/knowledge/archetypes/` | 6 project-type templates: saas, marketing, mobile, api, internal-tool, content |
| `resources/the-architect/knowledge/building-blocks/` | Decision guides: auth, db, deployment, api design, frontend, testing, styling, state |
| `resources/the-architect/templates/` | Blueprint output templates and AGENTS.md generation templates |

---

## The Architect System

**IMPORTANT:** This agent is an OpenCode implementation of [The Architect](https://github.com/Hainrixz/the-architect.git), designed by [Hainrixz](https://github.com/Hainrixz)

The Architect (`agents/the-architect.md`, `docs/the-architect.md`) is a standalone primary mode that designs complete software systems and produces buildable blueprints. It follows a **4-phase workflow**:

1. **Discovery** — Asks targeted questions to understand project goals, constraints, and context using the knowledge resources under `resources/the-architect/`.
2. **Deep Dive** — Explores the domain thoroughly, researching technical decisions and documenting trade-offs.
3. **Architecture** — Synthesizes findings into a coherent system design: component hierarchy, data flow, API surface, deployment model.
4. **Generate** — Produces a structured blueprint `.md` file that can be handed to the Tech Lead for implementation.

The Architect draws on **6 archetypes** (project-type templates like SaaS, API, mobile) and **8 building-block decision guides** (auth, database, deployment, api design, frontend, testing, styling, state management) to accelerate design.

---

## Usage

Using this workflow requires [OpenCode](https://opencode.ai) with this repo as the working directory.

1. **Describe your idea** — Tell the Tech Lead agent what you want to build (a new feature, a bug fix, or an entire project).
2. **Compose your workflow** — The Tech Lead presents 5 available steps (Designer, Developer, QA, Reviewer, Commiter). Pick the ones you need — they must follow logical order (Designer → Developer → QA → Reviewer → Commiter), but you can skip any steps you don't need.
3. **Plan generation** — The Tech Lead dispatches the **Designer** subagent to explore the project and produce a plan at `.opencode/docs/plan.md` with requirements, acceptance criteria, and build order. You review and approve the plan before work begins.
4. **Pipeline runs** — The Tech Lead dispatches each chosen sub-agent sequentially per your workflow. Each agent reads the plan, performs its role, and passes results to the next.
5. **Review output** — The final deliverable depends on your chosen workflow: a plan document, implementation files, a test plan, a review report, or a commit message.

For new projects from scratch, use **The Architect** instead — it generates a complete blueprint that the Tech Lead can then implement.

---

## Key Details

1. **Path resolution for The Architect** — Knowledge files are referenced as `~/.config/opencode/resources/the-architect/...` in agent instructions. These resolve at runtime from the user's home config, not the repo. Do not rewrite these to local paths.
2. **`.gitignore` exclusions** — `AGENTS.md` and `.opencode/` are gitignored as generated artifacts. They will not be committed if created or modified.
3. **Agent mode matters** — `mode: primary` agents (tech-lead, the-architect) run standalone. `mode: subagent` agents (analyst, designer, developer, qa, reviewer, commiter) are dispatched by primaries via the `task` tool.
4. **No application code** — Every file in this repo is documentation or configuration. There is no `package.json`, `node_modules`, or dev server. All agent definitions are markdown with YAML frontmatter that OpenCode parses at load time.
5. **Plan-driven pipeline** — The Tech Lead dispatches the **Designer** subagent to create `.opencode/docs/plan.md` during the plan phase. All downstream sub-agents read this file before starting their work. The plan is presented to the user for explicit approval before any implementation begins.
6. **Composable workflow, not fixed scopes** — Unlike traditional fixed-scope pipelines, users pick individual steps to form their workflow. The Tech Lead validates that the chosen order respects logical dependencies.

---

## Relationship to AGENTS.md

This README is a **user-facing** introduction to the project. `AGENTS.md` is an **internal agent-facing** reference — it contains instructions that agents read at runtime to understand the codebase. The two files complement each other: use the README to understand what the project is, and `AGENTS.md` to understand how agents should operate within it.
