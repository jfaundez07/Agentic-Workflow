# Agentic Workflow Specification

## Overview

This document defines the agentic software development workflow. The system
consists of 7 specialized agents coordinated by the **Tech Lead**, who manages
execution via a shared state file (`resources/tech-leader/workflow-state-template.json`). Artifacts are passed
between agents exclusively through `.md` files — no agent-to-agent messaging.

The Tech Lead first ensures deep project analysis is available (delegating to the
**Analyst** if needed), then drives the Inception phase — exploring the existing
project, understanding the user's request, producing the specification — and
delegates implementation to the sub-agents (Developer, QA, Docs Writer) who work
in parallel.

## Agent Roster

| # | Role | Responsibility | Produces | Gate | Parallel |
|---|------|--------------|----------|------|----------|
| 1 | **Tech Lead** (orchestrator) | Explore existing project, understand requirements, write spec, delegate to sub-agents, manage state | `docs/spec.md` (or uses provided file), updates `resources/tech-leader/workflow-state-template.json` | Spec approval | No |
| 2 | **Analyst** | Deep technical analysis of existing project: architecture, tech stack, dependencies, security, technical debt | `.opencode/analysis.md` | — | No (runs before INCEPTION) |
| 3 | **Developer** | Feature implementation per spec, bug fixes, unit tests, linting | Project source code and configuration | — | Yes (with QA, DOC) |
| 4 | **Code Reviewer** | Code quality, correctness, security scanning, style compliance | Review comments (inline) | Merge approval | No |
| 5 | **QA Engineer** | Test strategy, test cases, integration/E2E tests, regression, validation | `docs/test-plan.md`, `docs/test-report.md` | — | Yes (with DEV, DOC) |
| 6 | **Documentation Writer** | API docs, README, user guide, changelog, inline docs | `docs/guide.md`, `docs/changelog.md`, `README.md` | — | Yes (with DEV, QA) |
| 7 | **Commiter** | Analyze workspace git diff, generate Conventional Commit message | Commit message | — | No |

## Shared State File

**File:** `workflow-state.json`

The Tech Lead is the sole writer of this file. All other agents read it to
determine:

- Current phase
- Which artifacts are available (especially `artifacts.spec` — the spec file path)
- Gate statuses (pending / approved / rejected)
- Active agent assignments
- Workflow history

All agents **must** read `workflow-state.json` before starting their work
to confirm their phase is active and their inputs are ready.

Sub-agents find the spec file by reading `workflow-state.json` → `artifacts.spec`.
This can be `"docs/spec.md"` (written by Tech Lead) or a user-provided `.md` file path.

## File-Based Artifact Protocol

Every handoff happens through the filesystem. No agent sends messages to another.

| Step | Agent | Reads | Writes | Gate |
|------|-------|-------|--------|------|
| 0 | TL | Existing project, user prompt or `.md` file, `docs/analysis.md` | `docs/spec.md` (if gaps exist) | **CLI: Approve spec?** |
| 1 | ANALYST | Existing project (full codebase) | `docs/analysis.md` | — |
| 2a | DEV | `artifacts.spec` (from state), `docs/analysis.md` | Project source code | — |
| 2b | QA | `artifacts.spec` | `docs/test-plan.md` | — |
| 2c | DOC | `artifacts.spec` | `docs/guide.md` | — |
| 3 | REVIEWER | Project source code, `artifacts.spec`, `docs/analysis.md` | Review report | **CLI: Approve merge?** |
| 4 | QA | Project source code, `docs/test-plan.md` | `docs/test-report.md` | — |
| 5 | DOC | Project source code, `artifacts.spec`, `docs/guide.md` | `docs/changelog.md`, `README.md` | — |
| 6 | COMMITER | Project source code, git diff | Commit message | — |

## Workflow Phases

### Phase 0: ANALYSIS (Analyst — optional, before INCEPTION)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  TL checks if .opencode/analysis.md already exists                            │
│                                                                               │
│  If NOT:                                                                      │
│  ├── TL delegates to Analyst subagent                                         │
│  ├── Analyst explores the full codebase                                       │
│  ├── Analyst writes .opencode/analysis.md with:                               │
│  │   ├── Technology stack                                                     │
│  │   ├── Architecture patterns                                                │
│  │   ├── APIs and endpoints                                                   │
│  │   ├── Database schema (if applicable)                                      │
│  │   ├── Dependencies and their status                                        │
│  │   ├── Authentication & security analysis                                   │
│  │   ├── Deployment & DevOps setup                                            │
│  │   └── Technical debt identification                                        │
│  └── TL proceeds to INCEPTION                                                 │
│                                                                               │
│  If YES: skip analysis, proceed to INCEPTION                                  │
└──────────────────────────────────────────────────────────────────────────────┘
```

- **Input:** Existing project codebase
- **Output:** `docs/analysis.md` (deep technical report)

### Phase 1: INCEPTION (Tech Lead)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  TL receives user input — either a direct message or a .md file path         │
│                                                                               │
│  1. Explore the existing project (read key files, understand context)         │
│     Optionally read .opencode/analysis.md for deeper context                  │
│                                                                               │
│  2. If .md file path:                                                         │
│  ├── Read the file                                                            │
│  ├── Does it have clear task description + acceptance criteria + context?     │
│  │   ├── YES → use file directly as spec                                     │
│  │   └── NO  → ask clarifying questions → write docs/spec.md                 │
│  └── Present for approval → "Approve this spec? (y/n)"                       │
│                                                                               │
│  3. If direct message:                                                        │
│  ├── Ask 2-3 clarifying questions                                             │
│  ├── Explore project for context                                              │
│  ├── Write docs/spec.md — focus on specific change, not full system          │
│  └── Present for approval → "Approve this spec? (y/n)"                       │
│                                                                               │
│  TL updates resources/tech-leader/workflow-state-template.json → phase: BUILD                                │
└──────────────────────────────────────────────────────────────────────────────┘
```

- **Input:** Human-provided requirements (conversation) or `.md` file, optionally `docs/analysis.md`
- **Output:** Approved spec (either `docs/spec.md` or the provided file)

### Phase 2: BUILD (Parallel Execution)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  ┌── DEV: reads artifacts.spec → implements code                             │
│  ├── QA: reads artifacts.spec → writes test-plan.md                          │
│  └── DOC: reads artifacts.spec → drafts guide.md                             │
│                                                                               │
│  TL monitors progress via workflow-state.json                                 │
└──────────────────────────────────────────────────────────────────────────────┘
```

- **Input:** The approved spec file (from `artifacts.spec`)
- **Output:** Source code, `docs/test-plan.md`, `docs/guide.md`

### Phase 3: REVIEW & VALIDATION

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  1. REVIEWER reads code + spec                                                │
│  2. CLI prompts: "Approve merge? (y/n/reason)"                               │
│  3. If rejected → loop back to DEV for fixes                                 │
│  4. QA runs tests against approved code                                      │
│  5. QA writes test-report.md                                                 │
│  6. DOC finalizes docs with actual API details                               │
│  7. DOC writes changelog.md + README.md                                      │
└──────────────────────────────────────────────────────────────────────────────┘
```

- **Input:** Source code, `docs/test-plan.md`, `docs/guide.md`
- **Output:** Approved code, `docs/test-report.md`, final docs

### Phase 4: COMMIT

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  1. COMMITER reads resources/tech-leader/workflow-state-template.json — confirm phase is COMMIT              │
│  2. COMMITER reads project source code, runs `git diff`                        │
│  3. COMMITER analyzes workspace changes and generates Conventional Commit     │
│  4. COMMITER presents commit message for copy-pasting                         │
│  5. TL marks workflow complete in resources/tech-leader/workflow-state-template.json                         │
└──────────────────────────────────────────────────────────────────────────────┘
```

- **Input:** Project source code, git diff
- **Output:** Conventional Commit message

## CLI Gate Protocol

Each gate follows this pattern:

```
1. Agent presents the artifact to the human (reads file content)
2. Agent asks: "Approve [artifact description]? (y/n/reason)"
3. Human responds:
   - "y" → proceed
   - "n" → agent asks for reason, then loops back to revise
   - Specific feedback → agent incorporates and re-presents
4. Agent updates resources/tech-leader/workflow-state-template.json with gate result
```

## Error / Loop Handling

| Condition | Action |
|-----------|--------|
| Spec rejected | TL revises using more questions → re-presents for approval |
| Code review fails | Reviewer provides reason → Developer fixes → re-review |
| Tests fail | QA documents failures in test-report.md → Developer fixes |
| Spec changes mid-workflow | TL updates spec → re-approval |
| Any gate rejected | TL resets workflow-state.json phase → appropriate prior phase |

## Parallel Work Coordination

When DEV, QA, and DOC run in parallel:

1. TL writes `workflow-state.json` → `activeAgents: ["DEV", "QA", "DOC"]`
2. Each agent reads the state file to confirm it should start
3. Each agent reads `workflow-state.json` → `artifacts.spec` to find the spec file
4. Each agent monitors the state file for blocking conditions:
   - DEV blocks if `artifacts.spec` is null
   - QA blocks if `artifacts.spec` is null
   - DOC blocks if `artifacts.spec` is null
5. When all parallel agents complete, TL advances the phase

## File Structure

```
agentic-workflow-old/
├── agents/
│   ├── tech-lead.md           (this agent — orchestrator)
│   ├── analyst.md             (sub-agent — deep technical analysis)
│   ├── developer.md           (sub-agent — implements code)
│   ├── reviewer.md            (sub-agent — code review)
│   ├── qa.md                  (sub-agent — test planning + execution)
│   ├── docs-writer.md         (sub-agent — documentation)
│   └── commiter.md            (sub-agent — commit message generation)
├── docs/
│   └── agentic-workflow-spec.md
├── resources/
│   └── tech-leader/
│       ├── spec-template.md              (template for TL to use when writing the spec)
│       └── workflow-state-template.json  (template — shared state managed by TL)
└── README.md                      (written by DOC)
```

Additional `.md` files may exist at the project root or in `docs/` if the user
provided them as input. These become the spec via `artifacts.spec` reference.

## Agent-Specific Instructions

### Analyst (Phase 0 — Analysis)

1. Read the existing project codebase thoroughly
2. Write `docs/analysis.md` with the following sections:
   - **Technology Stack** — runtime, framework, database, auth, critical libraries, build tools
   - **Architecture** — patterns, folder structure, layer separation, entry point
   - **APIs and Endpoints** — HTTP methods, routes
   - **Database** — type, schema, relationships, indexes, constraints
   - **Dependencies** — table with versions and status (up-to-date, outdated, deprecated)
   - **Authentication & Security** — mechanisms, password hashing, CORS
   - **Deployment & DevOps** — Docker, environment configs, CI/CD
   - **Technical Debt** — location, severity, description, impact, root cause
3. Deliver the file to `docs/analysis.md` for all other agents to reference

### Tech Lead (Phase 0–1 — Analysis + Inception)

1. Read `resources/tech-leader/workflow-state-template.json` — confirm `phase` is `INCEPTION`
2. **Explore the existing project** before writing the spec:
   - Read key project files (`package.json`, config files, existing source)
   - Inspect directory structure to understand the codebase
   - Identify relevant files, current behavior, tech stack
2a. **Run deep analysis (if needed):**
   - Check if `docs/analysis.md` already exists
   - If it does **not** exist, delegate to the **Analyst** subagent to produce it
   - Wait for the Analyst to complete before proceeding
3. Receive user input:
   - **Direct message** → ask 2-3 clarifying questions, explore project context, then write `docs/spec.md` focused on the specific change
   - **`.md` file path** → read the file, check for task description + acceptance criteria + context:
     - **Complete** → skip `docs/spec.md`, use file directly as spec
     - **Incomplete** → ask clarifying questions, write `docs/spec.md`
4. Set `workflow-state.json` → `artifacts.spec` to the correct file path
5. Present spec for CLI approval
6. If approved, advance to BUILD phase and set active agents

### Developer (Phase 2 — Build)

1. Read `workflow-state.json` — confirm `phase` is `BUILD`
2. Read the file at `workflow-state.json` → `artifacts.spec` — this is your primary guide
3. Read `docs/analysis.md` if it exists — provides a deep technical overview of the project's architecture, tech stack, dependencies, and conventions
4. **Explore the existing project** — read key files, understand the codebase before implementing
5. Look for these sections in the spec:
```markdown
- ## Task — what exactly needs to change
- ## Context — current project state and relevant files
```
   - Requirements, acceptance criteria
   - **Build Order** — if present, follow it exactly
6. If a **Build Order** exists, follow it step by step
7. Otherwise, implement directly from requirements and acceptance criteria
8. Before marking complete: run linter, run tests, verify the app builds

### Code Reviewer (Phase 3 — Review)

1. Read `workflow-state.json` — confirm `phase` is `REVIEW`
2. Read the file at `workflow-state.json` → `artifacts.spec` — understand the task and acceptance criteria
3. Read `docs/analysis.md` if it exists — provides deeper architectural context and identified tech debt to watch for
5. **Explore the existing project** — understand codebase conventions before reviewing
6. Read all project source files and evaluate for:
   - **Quality** — clean, readable, no duplication
   - **Correctness** — matches spec, handles edge cases and error states
   - **Security** — input validation, secrets exposure, auth checks, parameterized queries
   - **Style** — follows project conventions, clean imports, consistent formatting
7. Categorize issues as **BLOCKER** (must fix), **WARNING** (should fix), or **NIT** (optional)
8. If BLOCKER items exist → reject the merge
9. If only WARNING/NIT items → approve with comments
10. **CLI prompt:** "Approve merge? (y/n/reason)"

### QA Engineer (Phase 2-3 — Test)

**Phase: Test Planning (Parallel with DEV):**
1. Read `workflow-state.json` — confirm `phase` is `BUILD`
2. Read the file at `workflow-state.json` → `artifacts.spec` — understand the task and acceptance criteria
3. Read `docs/analysis.md` if it exists — provides deeper understanding of codebase structure, risk areas, and dependencies
5. **Explore the existing project** — read key files to understand codebase context before planning tests
6. Write `docs/test-plan.md` with: scope, test levels, test cases per user story, environment, test data, risk areas

**Phase: Test Execution (After Code is Approved):**
1. Read `workflow-state.json` — confirm phase
2. Read the spec at `workflow-state.json` → `artifacts.spec` for acceptance criteria
3. Read approved source code
4. Run the test suite
5. Write `docs/test-report.md` with: summary, failed tests, acceptance criteria check, bug list, recommendation

### Documentation Writer (Phase 2-3 — Documentation)

**Phase: Drafting (Parallel with DEV and QA):**
1. Read `workflow-state.json` — confirm `phase` is `BUILD`
2. Read the file at `workflow-state.json` → `artifacts.spec` — understand the task and project context
3. Read `docs/analysis.md` if it exists — provides a technical overview to inform documentation structure
4. **Explore the existing project** — read key files to understand the codebase before writing docs
4. Write `docs/guide.md` with: overview, installation, configuration, usage, API reference, troubleshooting

**Phase: Finalization (After Code is Approved):**
1. Read the approved project source code for actual API details
2. Read the spec at `workflow-state.json` → `artifacts.spec` for project context
3. Finalize `docs/guide.md` with real code examples
4. Write `docs/changelog.md` (version, date, Added/Changed/Fixed sections)
5. Write `README.md` at the project root with: project name, quick start, tech stack, available scripts, links

### Commiter (Phase 4 — Commit)

1. Read `workflow-state.json` — confirm `phase` is `COMMIT`
2. Load the `conventional-commit` skill
3. Analyze the git diff — what files were modified, added, or deleted
4. Determine the intent and scope of the changes
5. Select the appropriate Conventional Commit type (`feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `perf`)
6. Draft a subject line: `<type>(<scope>): <description>`
7. Generate a detailed body explaining what changed and why
8. Present the commit message (with 2-3 alternative suggestions) for copy-pasting
9. Update `resources/tech-leader/workflow-state-template.json` — `phase: "DONE"`
