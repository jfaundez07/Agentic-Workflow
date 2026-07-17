<p align="center">
  <img alt="OpenCode" src="https://img.shields.io/badge/OpenCode-agentic--workflow-6C47FF?style=flat-square">
  <img alt="Status" src="https://img.shields.io/badge/status-active-success?style=flat-square">
</p>

<h1 align="center">Agentic Workflow Setup</h1>
<p align="center"><strong>Personal agentic workflow for building software with OpenCode — a config/doc repo of agent definitions.</strong></p>

This repository defines a team of AI agents (8 total — 1 primary, 7 sub-agents) that collaborate to plan, build, document, verify, review, and commit software projects. It is a **configuration repository** — no application code, no build scripts, no test runners. Everything is markdown with YAML frontmatter that OpenCode loads as executable agent definitions.

## Table of Contents

- [Installation](#installation)
- [Architecture](#architecture)
- [Directory Structure](#directory-structure)
- [Usage](#usage)
- [Key Details](#key-details)
- [Relationship to AGENTS.md](#relationship-to-agentsmd)

---

## Installation

Clone this repository and copy the agent definitions to your OpenCode config directory:

```bash
git clone https://github.com/jfaundez07/agentic-workflow.git
cp -r agentic-workflow/agents ~/.config/opencode/agents
```

> **Note:** Paths in agent instructions reference `~/.config/opencode/...` at runtime, so copying to that location ensures everything resolves correctly.

---

## Architecture

The workflow has one primary agent and seven sub-agents dispatched by it.

### Primary Agent

| Agent | File | Role |
|-------|------|------|
| **Tech Lead** | `agents/tech-lead.md` | Orchestrator — communicates with user, dispatches to sub-agents, verifies output, manages handoffs. |

### Sub-Agents

| Agent | File | Responsibility |
|-------|------|----------------|
| **Analyst** | `agents/analyst.md` | Deep technical analysis of any project — tech stack, architecture, APIs, dependencies, security, DevOps, and technical debt identification. Runs independently to produce `docs/analysis.md`. |
| **Planner** | `agents/planner.md` | Creates detailed implementation plans (requirements, acceptance criteria, build order) at `docs/plans/plan-<id>.md`. |
| **Developer** | `agents/developer.md` | Implements code per specification — writes source files, tests, configurations. |
| **Code Documenter** | `agents/code-documenter.md` | Adds inline documentation (docstrings, comments) to all written or modified code. Reads the plan for context, never changes logic. |
| **Tester** | `agents/tester.md` | Designs and implements unit tests — presents a test plan then implements and runs tests to verify all acceptance criteria are covered. |
| **Reviewer** | `agents/reviewer.md` | Reviews code for correctness, security, performance, style, and adherence to spec. Pass/fail gate with BLOCKER/WARNING/NIT ratings. |
| **Commiter** | `agents/commiter.md` | Analyzes workspace diff and generates a Conventional Commit message with alternatives. Used as a fallback, not a pipeline step. |

### Pipeline Flow

The Tech Lead orchestrates and dispatches sub-agents in a plan-driven workflow:

| Step | Responsibility | Output |
|------|---------------|--------|
| **Intake** | Tech Lead gathers requirements from the user, asks clarifying questions, explores the existing project | Clear requirements |
| **Planner** | Explores the project, defines requirements, acceptance criteria, and build order | `docs/plans/plan-<id>.md` |
| **Developer** | Implements the plan — writes all source code | Working implementation |
| **Code Documenter** | Adds inline documentation to all written or modified code | Documented source files |
| **Tester** | Designs a test plan, then implements and runs tests | Test plan + passing tests |
| **Reviewer** | Audits code quality with pass/fail gate (BLOCKER/WARNING/NIT) | Review report |

After each step, the Tech Lead verifies the output and passes structured context to the next agent via a JSON handoff summary. The pipeline stops early if any stage fails verification. If the same sub-agent fails twice consecutively, the Tech Lead escalates to the user. The Tech Lead writes orchestration logs to `docs/workflow-log.json`.

---

## Directory Structure

```
.
├── README.md                    # This file — user-facing introduction
├── AGENTS.md                    # Internal agent-facing reference
├── .gitignore                   # Ignores .opencode/ runtime artifacts
└── agents/                      # Agent definitions (8 agents)
    ├── analyst.md
    ├── code-documenter.md
    ├── commiter.md
    ├── developer.md
    ├── planner.md
    ├── reviewer.md
    ├── tech-lead.md
    └── tester.md
```

| Path | Purpose |
|------|---------|
| `agents/` | Agent definitions — 8 markdown files with YAML frontmatter (tech-lead, analyst, planner, developer, tester, reviewer, commiter, code-documenter) |
| `.gitignore` | Excludes `.opencode/` runtime artifacts from version control |

---

## Usage

Using this workflow requires [OpenCode](https://opencode.ai) with this repo as the working directory.

1. **Describe your task** — Tell the Tech Lead agent what you want to build (a new feature, a bug fix, or an entire project).
2. **Intake & exploration** — The Tech Lead asks clarifying questions and explores the project structure.
3. **Plan generation** — The Tech Lead dispatches the **Planner** subagent to produce a plan at `docs/plans/plan-<id>.md` with requirements, acceptance criteria, and build order. You review and approve the plan before work begins.
4. **Pipeline runs** — The Tech Lead dispatches the sub-agents. Each agent reads the plan, performs its role, and passes results to the next.
5. **Review output** — The final deliverable depends on the pipeline stage: a plan document, implementation files, a test plan, a review report, or a commit message.

The **Analyst** subagent can also be dispatched independently to produce a `docs/analysis.md` document at any time.

---

## Key Details

1. **`.opencode/` is gitignored** — Runtime artifacts generated by agents (analysis, plans, logs) live under `.opencode/` and are excluded from version control.
2. **Agent mode matters** — `mode: primary` agents (tech-lead) run standalone. `mode: subagent` agents (analyst, code-documenter, commiter, developer, planner, reviewer, tester) are dispatched by primaries via the `task` tool.
3. **No application code** — Every file in this repo is documentation or configuration. There is no `package.json`, `node_modules`, or dev server. All agent definitions are markdown with YAML frontmatter that OpenCode parses at load time.
4. **No `opencode.json`** — Config lives entirely in markdown frontmatter. There is no separate OpenCode config file in this repo.
5. **Plan-driven pipeline** — The Tech Lead dispatches the **Planner** subagent to create `docs/plans/plan-<id>.md` during the plan phase. All downstream sub-agents read this file before starting their work. The plan is presented to the user for explicit approval before any implementation begins.
6. **Permission model** — The Tech Lead has restricted write/edit permissions (only `docs/plans/` and `docs/workflow-log.json`). The Reviewer has no shell access — the only sub-agent with `bash: false`.
7. **Commiter is a fallback** — It is not part of the standard pipeline. The Tech Lead can dispatch it on demand to generate commit messages from workspace diffs.

---

## Relationship to AGENTS.md

This README is a **user-facing** introduction to the project. `AGENTS.md` is an **internal agent-facing** reference — it contains instructions that agents read at runtime to understand the codebase. The two files complement each other: use the README to understand what the project is, and `AGENTS.md` to understand how agents should operate within it.
