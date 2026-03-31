# OmniAgent

A Claude Code plugin implementing a **Planner → Generator → Evaluator** architecture for software projects. One skill, five roles, loaded on demand.

## What Is This?

OmniAgent is a **harness engineering** practice on top of Claude Code. Instead of one prompt doing everything, it encodes process discipline — role boundaries, checklists, workflow steps, artifact conventions — into a structured harness, so the model focuses on the actual work.

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
│  ┌─────────────┐ ┌─────────────┐ ┌───────────┐ │
│  │     PM      │ │     RD      │ │    QA     │ │
│  │  Write PRD  │ │ Write Code  │ │Write Tests│ │
│  └─────────────┘ └─────────────┘ └───────────┘ │
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

**Lead (Planner)** — The team lead of PM, RD, and QA. A product expert, technical architect, and testing strategist rolled into one.

**PM / RD / QA (Generators)** — Execute the plan. PM writes PRDs, RD writes code, QA writes tests.

**Reviewer (Evaluator)** — Independent expert who reviews deliverables from ANY layer, including the Lead's plan. Writes feedback into the source role's doc directory for automatic pickup.

### Why Separate Lead and Reviewer?

The person who made the plan shouldn't judge whether the plan is good. Separating planning and evaluation prevents "reviewing your own homework" and creates an honest quality gate.

## How It Works

OmniAgent is a **single skill** with role-specific instructions loaded as references on demand. When activated, the skill routes to the right role based on your input, reads the corresponding reference file, and follows that role's workflow.

| Role | Trigger | Reference |
|------|---------|-----------|
| **Lead** | `lead`, `plan`, `project planning`, `task breakdown` | `references/lead.md` |
| **PM** | `pm`, `PRD`, `product requirement` | `references/pm.md` |
| **RD** | `rd`, `develop`, `implement`, `code` | `references/rd.md` |
| **QA** | `qa`, `test`, `write tests` | `references/qa.md` |
| **Reviewer** | `reviewer`, `review`, `check quality` | `references/reviewer.md` |

## Installation

```bash
# Register the marketplace
claude plugin marketplace add omniclaws/skills

# Install the plugin
claude plugin install omniagent@omniclaws
```

## Update

```bash
# Step 1: Update the marketplace index
claude plugin marketplace update omniclaws/skills

# Step 2: Update the plugin
claude plugin update omniagent@omniclaws
```

## Usage

```bash
# Lead — plan the project and break down tasks
/omniagent lead Plan the full development of the task manager

# PM — generate a PRD
/omniagent pm Design a task management app for remote teams

# RD — implement code
/omniagent rd Implement the user authentication module

# QA — write and run tests
/omniagent qa Write tests for the authentication module

# Reviewer — review any deliverable
/omniagent reviewer Review the project plan
/omniagent reviewer Review the PRD
/omniagent reviewer Review the latest code changes
/omniagent reviewer Review the test coverage
```

### The Loop

```
1. Plan     /omniagent lead      →  docs/plan/*.md + TaskList
2. Review   /omniagent reviewer  →  docs/plan/review/*.md
3. Revise   /omniagent lead      →  incorporates feedback, updates plan
4. Generate /omniagent pm        →  docs/prd/*.md
            /omniagent rd        →  code commits
            /omniagent qa        →  test files + reports
5. Review   /omniagent reviewer  →  feedback into each role's docs/
6. Iterate  Roles pick up feedback automatically on next activation
```

The Reviewer can enter at **any** point. The earlier you review, the cheaper it is to fix.

### Feedback Flow

The Reviewer writes feedback into the reviewed role's own doc directory:

| Reviewed Role | Feedback Location | Reviewer Expertise |
|--------------|-------------------|--------------------|
| Lead | `docs/plan/review/` | Scope, task balance, sequencing |
| PM | `docs/prd/review/` | Requirement clarity, edge cases, testability |
| RD | `docs/rd/review/` | Performance, maintainability, error handling |
| QA | `docs/test-report/review/` | Coverage gaps, boundary conditions, flakiness |

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
└── omniagent/
    ├── SKILL.md                    # Routing + architecture overview
    └── references/
        ├── lead.md                 # Lead role (Planner)
        ├── pm.md                   # PM role (Generator)
        ├── rd.md                   # RD role (Generator)
        ├── qa.md                   # QA role (Generator)
        ├── reviewer.md             # Reviewer role (Evaluator)
        ├── plan-template.md        # Project plan template
        └── prd-template.md         # PRD template
```

## License

MIT
