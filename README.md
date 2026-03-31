# OmniAgent

A Claude Code plugin that provides **4 professional role-based skills** covering the full software development lifecycle (SDLC) — from product requirements to architecture design, coding, and testing.

## Features

| Role | Skill | Description |
|------|-------|-------------|
| **PM** | `/omniagent:pm` | Market research, competitive analysis, user personas, structured PRD generation |
| **RD-Leader** | `/omniagent:rd-leader` | Architecture design, technology selection, API contracts, task decomposition |
| **RD** | `/omniagent:rd` | Code implementation following design specs, self-testing, code self-review |
| **QA** | `/omniagent:qa` | Test strategy, unit tests, E2E tests, execution, and coverage reporting |

## Installation

```bash
# Step 1: Register the marketplace
claude plugin marketplace add omniclaws/skills

# Step 2: Install the plugin
claude plugin install omniagent@omniclaws
```

## Usage

Each role can be invoked independently via slash commands:

```bash
# Product Manager — generate a PRD
/omniagent:pm Design a task management app for remote teams

# Tech Lead — create architecture design + task breakdown
/omniagent:rd-leader Design the technical architecture for the task manager

# Developer — implement code following the design
/omniagent:rd Implement the user authentication module

# QA Engineer — write and run tests
/omniagent:qa Write tests for the authentication module
```

### Role Collaboration Pipeline

Roles can work independently, but when used together they form a pipeline. Each role auto-detects artifacts produced by upstream roles:

```
PM (PRD)
  ↓ docs/prd/*.md
RD-Leader (Design + Tasks)
  ↓ docs/design/*.md + TaskList
RD (Code)
  ↓ code changes
QA (Tests)
```

For example, when you invoke `/omniagent:rd-leader`, it automatically scans `docs/prd/` for the latest PRD and incorporates it into the technical design. No manual wiring needed.

### Output Artifacts

| Role | Output Location |
|------|----------------|
| PM | `docs/prd/YYYY-MM-DD-<topic>.md` |
| RD-Leader | `docs/design/YYYY-MM-DD-<topic>.md` + TaskList |
| RD | Code commits |
| QA | Test files + `docs/test-report/YYYY-MM-DD-<topic>.md` |

## Project Structure

```
.claude-plugin/
├── marketplace.json      # Marketplace definition
└── plugin.json           # Plugin metadata
skills/
├── pm/
│   ├── SKILL.md          # Product Manager skill
│   └── references/
│       └── prd-template.md
├── rd-leader/
│   ├── SKILL.md          # Tech Lead skill
│   └── references/
│       └── design-template.md
├── rd/
│   └── SKILL.md          # Developer skill
└── qa/
    └── SKILL.md          # QA Engineer skill
```

## License

MIT
