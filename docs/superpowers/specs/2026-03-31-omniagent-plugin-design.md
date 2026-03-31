# OmniAgent Plugin Design Spec

## Overview

A Claude Code marketplace + plugin that provides 4 professional role-based skills for software development lifecycle:

- **PM** — Product Manager: market research, user analysis, PRD generation
- **RD-Leader** — Tech Lead: architecture design, task decomposition
- **RD** — Developer: code implementation
- **QA** — Tester: unit tests and E2E tests

## Installation Commands

```bash
# Step 1: Register marketplace
claude plugin marketplace add omniclaws/skills

# Step 2: Install plugin
claude plugin install omniagent@omniclaws
```

## Usage Commands

```
/omniagent:pm <requirement>         # Generate PRD
/omniagent:rd-leader <requirement>  # Design architecture + task breakdown
/omniagent:rd <task>                # Code implementation
/omniagent:qa <scope>               # Write and run tests
```

---

## Repository Structure

```
omniclaws/skills (GitHub repo = marketplace)
├── .claude-plugin/
│   ├── marketplace.json      # Marketplace definition
│   └── plugin.json           # Plugin metadata
├── skills/
│   ├── pm/
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── prd-template.md
│   ├── rd-leader/
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── design-template.md
│   ├── rd/
│   │   └── SKILL.md
│   └── qa/
│       └── SKILL.md
└── README.md
```

---

## Marketplace Configuration

**`.claude-plugin/marketplace.json`**:

```json
{
  "name": "omniclaws",
  "description": "OmniClaws skill marketplace — professional AI agent roles for full SDLC coverage",
  "owner": {
    "name": "omniclaws"
  },
  "plugins": [
    {
      "name": "omniagent",
      "description": "4 professional role-based skills (PM, RD-Leader, RD, QA) for full software development lifecycle",
      "source": "./",
      "category": "development"
    }
  ]
}
```

**`.claude-plugin/plugin.json`**:

```json
{
  "name": "omniagent",
  "version": "1.0.0",
  "description": "Professional role-based skills for software development: PM, Tech Lead, Developer, QA",
  "author": {
    "name": "omniclaws"
  },
  "repository": "https://github.com/omniclaws/skills",
  "license": "MIT",
  "keywords": ["pm", "rd", "qa", "sdlc", "product-manager", "tech-lead", "developer", "tester"]
}
```

---

## Skill Definitions

### 1. PM Skill (`skills/pm/SKILL.md`)

**Role**: Professional Product Manager

**Trigger**: `/omniagent:pm`, product design, PRD, requirement analysis

**Workflow**:
1. Understand the requirement (ask clarifying questions if ambiguous)
2. Market and competitive research (via WebSearch)
3. Define user personas and user stories
4. Enumerate functional requirements with priority (P0/P1/P2)
5. Define non-functional requirements (performance, security, compatibility)
6. Generate structured PRD document

**Output**: `docs/prd/YYYY-MM-DD-<topic>.md`

**PRD Structure** (see `references/prd-template.md`):
- Background & Problem Statement
- Goals & Success Metrics
- User Personas
- User Stories
- Functional Requirements (prioritized)
- Non-Functional Requirements
- Information Architecture
- Milestones & Timeline
- Open Questions

**Tools Used**: WebSearch, Read, Glob, Write

### 2. RD-Leader Skill (`skills/rd-leader/SKILL.md`)

**Role**: Technical Lead / Architect

**Trigger**: `/omniagent:rd-leader`, technical design, architecture, task decomposition

**Workflow**:
1. Read PRD if available (`docs/prd/` directory auto-scan)
2. Explore existing codebase (Glob/Grep/Read)
3. Technology selection with trade-off analysis
4. Architecture design (modules, data flow, interfaces)
5. API/interface contract definition
6. Task decomposition with estimates, priorities, and dependencies
7. Generate design doc + task list

**Output**:
- `docs/design/YYYY-MM-DD-<topic>.md` — Technical design document
- TaskCreate entries — Structured task breakdown

**Design Doc Structure** (see `references/design-template.md`):
- Technical Context
- Architecture Overview
- Module Breakdown
- Data Model
- API Contracts
- Error Handling Strategy
- Task Breakdown (with estimates and dependencies)
- Risks & Mitigations

**Upstream Detection**: On activation, scans `docs/prd/` for the most recent PRD and auto-incorporates it.

**Tools Used**: Read, Glob, Grep, Write, TaskCreate

### 3. RD Skill (`skills/rd/SKILL.md`)

**Role**: Software Developer

**Trigger**: `/omniagent:rd`, develop, code, implement

**Workflow**:
1. Read design doc if available (`docs/design/` auto-scan)
2. Check TaskList for assigned tasks
3. Analyze impact scope of the change
4. Implement code following design constraints
5. Self-test the implementation (run existing tests)
6. Self-review checklist:
   - Does it match the design spec?
   - Are edge cases handled?
   - Is error handling complete?
   - Are there any hardcoded values that should be configurable?

**Output**: Code implementation + git commits

**Upstream Detection**: Reads `docs/design/` for latest design doc, checks TaskList for assigned work.

**Tools Used**: All tools (Read, Write, Edit, Bash, Glob, Grep, etc.)

### 4. QA Skill (`skills/qa/SKILL.md`)

**Role**: Quality Assurance Engineer

**Trigger**: `/omniagent:qa`, test, unit test, E2E, QA

**Workflow**:
1. Read PRD for acceptance criteria (if available)
2. Analyze codebase to understand what to test
3. Define test strategy (unit vs integration vs E2E coverage)
4. Write unit tests (framework auto-detection: Jest, Vitest, pytest, Go test, etc.)
5. Write E2E tests (Playwright/Cypress if web, otherwise integration tests)
6. Execute all tests
7. Generate test report with coverage

**Output**:
- Test code files (co-located or in `__tests__`/`tests/` following project convention)
- `docs/test-report/YYYY-MM-DD-<topic>.md` — Test report

**Test Report Structure**:
- Test Strategy Summary
- Coverage Metrics
- Test Results (pass/fail breakdown)
- Edge Cases Covered
- Known Gaps / TODO

**Upstream Detection**: Reads `docs/prd/` for acceptance criteria, scans code changes since last commit.

**Tools Used**: Read, Glob, Grep, Write, Bash (test execution)

---

## Role Collaboration Model

Each skill is independently usable. When upstream artifacts exist, they are auto-detected:

```
PM (PRD)
  ↓ docs/prd/*.md
RD-Leader (Design + Tasks)
  ↓ docs/design/*.md + TaskList
RD (Code)
  ↓ code changes
QA (Tests)
```

**Detection Logic**: Each skill on activation scans known output directories. If found, the skill announces it's using the upstream artifact and incorporates it. If not found, the skill proceeds independently and asks the user for context.

---

## Design Decisions

1. **Single plugin, multi-skill**: Follows pua/superpowers pattern. One install, four skills, shared references.
2. **Markdown-based artifacts**: All outputs are markdown files in `docs/` subdirectories, making them version-controllable and human-readable.
3. **Auto-detection over explicit wiring**: Skills detect upstream artifacts by convention (`docs/prd/`, `docs/design/`) rather than requiring explicit parameters.
4. **Framework-agnostic**: QA skill auto-detects the project's test framework rather than prescribing one.
5. **Reference files for templates**: PRD and design templates are in `references/` directories, keeping SKILL.md focused on workflow and behavior.
