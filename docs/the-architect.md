## The Architect

For building a project from scratch, use the**blueprint** — a detailed plan that what and how to build it.

**The Architect does this for software.**

You open this project in an AI agent (e.g. OpenCode, Claude Code), tell it what you want to build (a website, an app, a SaaS, an API — anything), and The Architect:

1. **Asks you smart questions** about your idea
2. **Designs the entire system** — database, API, frontend, auth, payments, everything
3. **Generates a blueprint file** (`.md`) so detailed that another AI agent instance can build the whole thing — step by step, without asking you anything

Think of it like this:

```
You: "I want to build a project management app like Trello"
         ↓
The Architect: asks questions, designs everything
         ↓
Blueprint.md: complete plan with tech stack, database schema,
               API routes, component hierarchy, build order...
          ↓
Your AI agent: reads the blueprint → builds the entire app
```

**You describe the idea. The Architect creates the plan. Your AI agent builds it.**

---

## How It Works

The Architect follows a 4-phase workflow:

### Phase 1: Discovery
> "What are you building? Who is it for? How big should it be?"

The Architect asks 2-3 questions to understand your idea. Based on your answers, it classifies your project into one of **6 archetypes** (project types).

### Phase 2: Deep Dive
> "Do you need user accounts? Payments? Real-time features?"

Now it asks questions specific to YOUR type of project. It also researches best practices using skills like `/deep-research`.

### Phase 3: Architecture
> "Here's what I'd build: Next.js + Supabase + Clerk + Stripe on Vercel..."

The Architect presents the complete tech stack and architecture with reasons for every decision. You confirm or adjust. For frontend projects, it designs a full visual system (colors, fonts, spacing) using `/ui-ux-pro-max`.

### Phase 4: Generate
The Architect produces the final blueprint — a single `.md` file with **16 sections** covering everything the build agent needs to build your project from zero to deployed.

---

## The Blueprint: 16 Sections

Every blueprint The Architect generates includes:

| # | Section | What It Contains |
|---|---------|-----------------|
| 1 | Project Overview | Vision, goals, success metrics |
| 2 | Tech Stack | Every technology with rationale |
| 3 | Directory Structure | Full file tree with explanations |
| 4 | Data Model | Entities, fields, relationships, SQL schema |
| 5 | API Design | Routes, endpoints, request/response shapes |
| 6 | Frontend Architecture | Pages, components, state management |
| 7 | Design System | Colors, fonts, spacing, component style |
| 8 | Auth & Authorization | Login flow, roles, permissions |
| 9 | **Build Order** | Step-by-step: what to build first, second, third... |
| 10 | Environment Setup | Prerequisites, env vars, initial commands |
| 11 | Dependencies | Every package with its purpose |
| 12 | Deployment | Hosting, CI/CD, domains |
| 13 | Testing | What to test, which tools, when |
| 14 | Skills to Use | Which agent skills help during the build |
| 15 | **AGENTS.md** | Complete instructions for the builder — ready to paste |
| 16 | Rules | Non-negotiable constraints for the build |

**Section 9 (Build Order) is the most important.** It tells the build agent exactly what to build and in what order — so it can work autonomously without asking you questions.

---

### Usage

**Step 1:** Tell The Architect what you want to build:

```
You: I want to build a SaaS for managing restaurant reservations
     with team accounts, Stripe payments, and a customer-facing booking page.
```

**Step 2:** Answer the questions (2-3 at a time, conversational).

**Step 3:** Review the proposed architecture. Confirm or adjust.

**Step 4:** The Architect generates your blueprint at `output/<project-name>-blueprint.md`.

**Step 5:** Use the blueprint to build your project:

```bash
# Create your new project
mkdir ~/my-restaurant-saas
cp output/restaurant-saas-blueprint.md ~/my-restaurant-saas/AGENTS.md

# Open it in your AI agent — it builds everything from the blueprint
cd ~/my-restaurant-saas
opencode
```

### Fast-Track Mode

Don't want to answer many questions? Say:

```
You: Build me a SaaS for restaurant reservations. Just build it.
```

The Architect asks only 3 essential questions and uses smart defaults for everything else.

---

## Project Structure

```
the-architect/
├── architect.md                          # The brain — makes your AI agent become The Architect
├── knowledge/
│   ├── archetypes/                    # 6 project type templates
│   │   ├── saas-webapp.md             #   SaaS apps
│   │   ├── marketing-site.md          #   Landing pages
│   │   ├── mobile-app.md              #   Mobile apps
│   │   ├── api-backend.md             #   APIs & backends
│   │   ├── internal-tool.md           #   Admin panels
│   │   └── content-platform.md        #   Blogs & CMS
│   ├── building-blocks/               # 8 cross-cutting decision guides
│   │   ├── auth-patterns.md           #   Authentication options
│   │   ├── database-patterns.md       #   Database & ORM choices
│   │   ├── deployment-patterns.md     #   Hosting & CI/CD
│   │   ├── api-design-patterns.md     #   REST vs tRPC vs GraphQL
│   │   ├── frontend-stacks.md         #   Framework comparisons
│   │   ├── testing-patterns.md        #   Testing strategies
│   │   ├── styling-systems.md         #   CSS & design tokens
│   │   └── state-management.md        #   State & real-time patterns
│   └── skills-registry.md             # Maps agent skills to blueprint sections
├── templates/
│   ├── blueprint-template.md          # The 16-section output skeleton
│   └── agent-md-template.md           # Template for the target project's AGENTS.md
├── questions/
│   ├── phase-1-discovery.md           # Universal discovery questions
│   ├── phase-2-branches.md            # Archetype-specific deep dives
│   └── phase-3-confirmation.md        # Architecture confirmation
└── output/                            # Generated blueprints go here
```

---

## Skills Integration

The Architect leverages agent skills during the design process:

| Skill | Used For |
|-------|----------|
| `/deep-research` | Researching technologies and best practices |
| `/ui-ux-pro-max` | Designing visual systems (colors, fonts, spacing) |
| `/find-skills` | Discovering skills to recommend for the build phase |
| `/playwright-cli` | Analyzing reference sites |

And recommends skills for the **build phase** in the blueprint:

| Skill | Recommended For |
|-------|----------------|
| `/frontend-design` | Building production-grade UIs |
| `/shadcn-ui` | Setting up component libraries |
| `/seo-audit` | SEO auditing after build |
| `/humanizer` | Making content sound natural |

---

## Contributing

Contributions are welcome! You can help by:

- **Adding new archetypes** — new project types in `knowledge/archetypes/`
- **Improving building blocks** — better decision matrices, newer tech options
- **Enhancing the blueprint template** — more sections, better structure
- **Translating** — help make The Architect accessible in more languages
