# OmniAgent

A Claude Code plugin that provides **5 professional role-based skills** covering the full software development lifecycle (SDLC) — from planning to execution to review.

## What Is This?

OmniAgent is a **harness engineering** practice on top of Claude Code. Instead of one prompt doing everything, it encodes a **Planner → Generator → Evaluator** architecture into structured, reusable skills.

The core idea of harness engineering: encode process discipline — role boundaries, checklists, workflow steps, artifact conventions — into the harness, so the model focuses on the actual work instead of figuring out how to work.

### The Three-Layer Architecture

```
┌─────────────────────────────────────────────────┐
│  Planner                                        │
│  ┌───────────────────────────────────────────┐  │
│  │                  Lead                     │  │
│  │ Product Expert + Architect + QA Strategist│  │
│  └───────────────────────────────────────────┘  │
└──────────────────────┬──────────────────────────┘
                       │ plan & decompose
                       ▼
┌─────────────────────────────────────────────────┐
│  Generators                                     │
│  ┌─────────────┐ ┌─────────────┐ ┌───────────┐  │
│  │     PM      │ │     RD      │ │    QA     │  │
│  │  Write PRD  │ │ Write Code  │ │Write Tests│  │
│  └─────────────┘ └─────────────┘ └───────────┘  │
└──────────────────────┬──────────────────────────┘
                       │ deliverables
                       ▼
┌─────────────────────────────────────────────────┐
│  Evaluator                                      │
│  ┌───────────────────────────────────────────┐  │
│  │                Reviewer                   │  │
│  │  Reviews ALL layers: Plan + Deliverables  │  │
│  └───────────────────────────────────────────┘  │
└──────────────────────┬──────────────────────────┘
                       │ feedback
                       ▼
              ┌─────────────────┐
              │   Loop back to  │
              │ Planner or      │
              │ Generators      │
              └─────────────────┘
```

**Lead (Planner)** — The team lead of PM, RD, and QA. A product expert, technical architect, and testing strategist rolled into one. Sets project direction, designs architecture, decomposes tasks across all roles.

**PM / RD / QA (Generators)** — Execute the plan. PM writes PRDs, RD writes code, QA writes tests. Each follows upstream artifacts and project conventions.

**Reviewer (Evaluator)** — The Lead's peer, not subordinate. An independent expert who reviews deliverables from ANY stage — including the Lead's own plan. Brings domain-specific expertise to each review: thinks like a project manager when reviewing plans, like an architect when reviewing code, like a QA lead when reviewing tests. Writes actionable feedback directly into the source role's doc directory, so it's automatically picked up on the next iteration.

### Why Separate Lead and Reviewer?

The person who made the plan shouldn't be the one who judges whether the plan is good. Separating planning and evaluation as peer roles prevents "reviewing your own homework" and creates an honest quality gate.

The Reviewer can push back on the Lead's plan — and should, if there are gaps in scope, lopsided task breakdowns, or unrealistic timelines. This tension is by design.

## Features

| Role | Skill | Architecture Role | Description |
|------|-------|-------------------|-------------|
| **Lead** | `/omniagent:lead` | Planner | Project planning, architecture design, cross-role task decomposition |
| **PM** | `/omniagent:pm` | Generator | Market research, competitive analysis, user personas, structured PRD |
| **RD** | `/omniagent:rd` | Generator | Code implementation following design specs, self-testing |
| **QA** | `/omniagent:qa` | Generator | Test strategy, unit tests, E2E tests, coverage reporting |
| **Reviewer** | `/omniagent:reviewer` | Evaluator | Independent expert review of any deliverable, feedback to source role |

## Installation

```bash
# Step 1: Register the marketplace
claude plugin marketplace add omniclaws/skills

# Step 2: Install the plugin
claude plugin install omniagent@omniclaws
```

## Usage

```bash
# Lead — plan the project and break down tasks
/omniagent:lead Plan the full development of the task manager

# PM — generate a PRD
/omniagent:pm Design a task management app for remote teams

# RD — implement code
/omniagent:rd Implement the user authentication module

# QA — write and run tests
/omniagent:qa Write tests for the authentication module

# Reviewer — review any stage
/omniagent:reviewer Review the project plan
/omniagent:reviewer Review the PRD
/omniagent:reviewer Review the latest code changes
/omniagent:reviewer Review the test coverage
```

### The Planner → Generator → Evaluator Loop

```
1. Plan     /omniagent:lead      →  docs/plan/*.md + TaskList
2. Review   /omniagent:reviewer  →  docs/plan/review/*.md (is the plan sound?)
3. Revise   /omniagent:lead      →  incorporates feedback, updates plan
4. Generate /omniagent:pm        →  docs/prd/*.md
            /omniagent:rd        →  code commits
            /omniagent:qa        →  test files + reports
5. Review   /omniagent:reviewer  →  feedback into each role's docs/
6. Iterate  Roles pick up feedback automatically on next activation
```

The Reviewer can enter at **any** point in this loop. Review the plan before generators start. Review the PRD before coding begins. Review the code before testing. Review the tests before shipping. The earlier you review, the cheaper it is to fix.

### Feedback Flow

The Reviewer writes feedback **into the reviewed role's own doc directory**, creating a natural feedback loop:

| Reviewed Role | Feedback Location | What Reviewer Brings |
|--------------|-------------------|---------------------|
| Lead | `docs/plan/review/` | Project management expertise — scope gaps, task balance, sequencing |
| PM | `docs/prd/review/` | Product expertise — requirement clarity, edge cases, testability |
| RD | `docs/rd/review/` | Architecture expertise — performance, maintainability, error handling |
| QA | `docs/test-report/review/` | QA expertise — coverage gaps, boundary conditions, flakiness risks |

### Output Artifacts

| Role | Output Location |
|------|----------------|
| Lead | `docs/plan/YYYY-MM-DD-<topic>.md` + TaskList |
| PM | `docs/prd/YYYY-MM-DD-<topic>.md` |
| RD | Code commits |
| QA | Test files + `docs/test-report/YYYY-MM-DD-<topic>.md` |
| Reviewer | `docs/<role>/review/YYYY-MM-DD-<topic>.md` |

## Project Structure

```
.claude-plugin/
├── marketplace.json
└── plugin.json
skills/
├── lead/
│   ├── SKILL.md             # Lead skill (Planner)
│   └── references/
│       └── design-template.md
├── pm/
│   ├── SKILL.md             # PM skill (Generator)
│   └── references/
│       └── prd-template.md
├── rd/
│   └── SKILL.md             # RD skill (Generator)
├── qa/
│   └── SKILL.md             # QA skill (Generator)
└── reviewer/
    └── SKILL.md             # Reviewer skill (Evaluator)
```

## License

MIT
