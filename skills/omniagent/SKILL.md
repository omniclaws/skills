---
name: omniagent
description: "A harness engineering practice implementing a three-layer Planner→Generator→Evaluator architecture for software projects. 5 roles: Lead (plan & decompose), PM (write PRD), RD (write code), QA (write tests), Reviewer (review any deliverable). Invoke with a role: '/omniagent lead', '/omniagent pm', '/omniagent rd', '/omniagent qa', '/omniagent reviewer'. Also triggers on: 'project planning', 'task breakdown', 'write PRD', 'implement', 'write tests', 'review this', '项目规划', '任务拆解', '写PRD', '写代码', '写测试', '评审', '验收'."
license: MIT
---

# OmniAgent — Harness Engineering for Software Teams

OmniAgent encodes a **three-layer architecture** into structured roles. Each role has clear boundaries, workflows, and output conventions. The harness enforces process discipline so the model focuses on the actual work.

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

## How to Use

This skill is invoked with a **role name**. On activation:

1. Determine which role the user wants based on their input
2. Read the corresponding reference file from `references/`
3. Follow that role's workflow exactly

### Role Routing

| User Input | Role | Reference File |
|-----------|------|---------------|
| `lead`, `plan`, `project planning`, `task breakdown`, `架构设计`, `任务拆解` | Lead | `references/lead.md` |
| `pm`, `PRD`, `product requirement`, `产品需求`, `写PRD` | PM | `references/pm.md` |
| `rd`, `develop`, `implement`, `code`, `写代码`, `编码` | RD | `references/rd.md` |
| `qa`, `test`, `write tests`, `写测试`, `测试` | QA | `references/qa.md` |
| `reviewer`, `review`, `check quality`, `评审`, `验收`, `检查` | Reviewer | `references/reviewer.md` |

**After routing, read the reference file and follow its instructions.** The reference file contains the full role definition: principles, workflow steps, output format, and red flags.

### The Loop

```
1. Plan     lead      →  docs/plan/*.md + TaskList
2. Review   reviewer  →  docs/plan/review/*.md
3. Generate pm        →  docs/prd/*.md
            rd        →  code commits
            qa        →  test files + reports
4. Review   reviewer  →  feedback into each role's docs/
5. Iterate  Roles pick up feedback automatically on next activation
```

The Reviewer can enter at **any** point. The earlier you review, the cheaper it is to fix.

### Feedback Flow

The Reviewer writes feedback **into the reviewed role's own doc directory**:

| Reviewed Role | Feedback Location |
|--------------|-------------------|
| Lead | `docs/plan/review/` |
| PM | `docs/prd/review/` |
| RD | `docs/rd/review/` |
| QA | `docs/test-report/review/` |

### Output Artifacts

| Role | Output Location |
|------|----------------|
| Lead | `docs/plan/YYYY-MM-DD-<topic>.md` + TaskList |
| PM | `docs/prd/YYYY-MM-DD-<topic>.md` |
| RD | Code commits |
| QA | Test files + `docs/test-report/YYYY-MM-DD-<topic>.md` |
| Reviewer | `docs/<role>/review/YYYY-MM-DD-<topic>.md` |

### Templates

- Project plan template: `references/plan-template.md`
- PRD template: `references/prd-template.md`
