# OmniAgent

一个 Claude Code 插件，实现了 **规划者 → 执行者 → 评审者（Planner → Generator → Evaluator）** 三层架构。一个 skill，五个角色，按需加载。

## 这是什么？

OmniAgent 是基于 Claude Code 的一套 **Harness Engineering（驾驭工程）** 实践。它不依赖单个 prompt 包揽一切，而是将流程规范——角色边界、检查清单、工作流步骤、产物约定——编码到结构化的 harness 中，让模型专注于实际工作本身。

### 三层架构

```
┌─────────────────────────────────────────────────┐
│  规划层 Planner                                  │
│  ┌───────────────────────────────────────────┐  │
│  │                  Lead                     │  │
│  │     产品专家 + 技术架构师 + 测试策略师       │  │
│  └───────────────────────────────────────────┘  │
└──────────────────────┬──────────────────────────┘
                       │ 规划 & 拆解任务
                       ▼
┌─────────────────────────────────────────────────┐
│  执行层 Generators                               │
│  ┌─────────────┐ ┌─────────────┐ ┌───────────┐ │
│  │     PM      │ │     RD      │ │    QA     │ │
│  │   撰写 PRD  │ │   编写代码   │ │  编写测试  │ │
│  └─────────────┘ └─────────────┘ └───────────┘ │
└──────────────────────┬──────────────────────────┘
                       │ 交付产物
                       ▼
┌─────────────────────────────────────────────────┐
│  评审层 Evaluator                                │
│  ┌───────────────────────────────────────────┐  │
│  │                Reviewer                   │  │
│  │    评审所有层级的产出：规划 + 交付物          │  │
│  └───────────────────────────────────────────┘  │
└──────────────────────┬──────────────────────────┘
                       │ 反馈
                       ▼
              ┌─────────────────┐
              │   回流至规划层    │
              │   或执行层       │
              └─────────────────┘
```

**Lead（规划者）**—— PM、RD、QA 的团队负责人，集产品专家、技术架构师、测试策略师于一身。

**PM / RD / QA（执行者）**—— 执行规划。PM 撰写 PRD，RD 编写代码，QA 编写测试。

**Reviewer（评审者）**—— 独立专家，可评审任意层级的产出（包括 Lead 的规划），并将反馈写入对应角色的文档目录，供下一轮自动读取。

### 为什么要把 Lead 和 Reviewer 分开？

做规划的人不应该自己评判规划的好坏。将规划和评审拆分为平级角色，避免"自己批改自己的作业"，确保质量关口的客观性。

## 工作原理

OmniAgent 是一个**单一 skill**，各角色的指令以 references 文件形式按需加载。激活时，skill 根据用户输入路由到对应角色，读取相应的 reference 文件，然后按该角色的工作流执行。

| 角色 | 触发词 | Reference 文件 |
|------|--------|---------------|
| **Lead** | `lead`、`plan`、`project planning`、`task breakdown`、`规划`、`拆任务` | `references/lead.md` |
| **PM** | `pm`、`PRD`、`product requirement`、`产品需求`、`写PRD` | `references/pm.md` |
| **RD** | `rd`、`develop`、`implement`、`code`、`写代码`、`编码` | `references/rd.md` |
| **QA** | `qa`、`test`、`write tests`、`写测试`、`测试` | `references/qa.md` |
| **Reviewer** | `reviewer`、`review`、`check quality`、`评审`、`验收` | `references/reviewer.md` |

## 安装

```bash
# 注册 marketplace
claude plugin marketplace add omniclaws/skills

# 安装插件
claude plugin install omniagent@omniclaws
```

## 更新

```bash
# 第一步：更新 marketplace 索引
claude plugin marketplace update omniclaws/skills

# 第二步：更新插件
claude plugin update omniagent@omniclaws
```

## 使用方法

```bash
# Lead —— 规划项目、拆解任务
/omniagent lead 规划任务管理器的完整开发方案

# PM —— 撰写产品需求文档
/omniagent pm 设计一个面向远程团队的任务管理应用

# RD —— 编写代码
/omniagent rd 实现用户认证模块

# QA —— 编写和运行测试
/omniagent qa 为认证模块编写测试

# Reviewer —— 评审任意阶段的产出
/omniagent reviewer 评审项目规划
/omniagent reviewer 评审 PRD
/omniagent reviewer 评审最近的代码变更
/omniagent reviewer 评审测试覆盖率
```

### 迭代循环

```
1. 规划     /omniagent lead      →  docs/plan/*.md + TaskList
2. 评审     /omniagent reviewer  →  docs/plan/review/*.md
3. 修订     /omniagent lead      →  吸收反馈，更新规划
4. 执行     /omniagent pm        →  docs/prd/*.md
            /omniagent rd        →  代码提交
            /omniagent qa        →  测试文件 + 报告
5. 评审     /omniagent reviewer  →  反馈写入各角色的 docs/ 目录
6. 迭代     各角色下次激活时自动读取反馈
```

Reviewer 可以在**任意节点**介入。越早评审，修复成本越低。

### 反馈流向

Reviewer 将反馈写入被评审角色自己的文档目录：

| 被评审角色 | 反馈位置 | Reviewer 评审视角 |
|-----------|---------|-----------------|
| Lead | `docs/plan/review/` | 范围完整性、任务均衡性、执行顺序 |
| PM | `docs/prd/review/` | 需求清晰度、边界场景、可测试性 |
| RD | `docs/rd/review/` | 性能、可维护性、错误处理 |
| QA | `docs/test-report/review/` | 覆盖度缺口、边界条件、测试稳定性 |

### 产出物

| 角色 | 产出位置 |
|------|---------|
| Lead | `docs/plan/YYYY-MM-DD-<topic>.md` + TaskList |
| PM | `docs/prd/YYYY-MM-DD-<topic>.md` |
| RD | 代码提交 |
| QA | 测试文件 + `docs/test-report/YYYY-MM-DD-<topic>.md` |
| Reviewer | `docs/<role>/review/YYYY-MM-DD-<topic>.md` |

## 项目结构

```
.claude-plugin/
├── marketplace.json
└── plugin.json
skills/
└── omniagent/
    ├── SKILL.md                    # 路由 + 架构概览
    └── references/
        ├── lead.md                 # Lead 角色（规划者）
        ├── pm.md                   # PM 角色（执行者）
        ├── rd.md                   # RD 角色（执行者）
        ├── qa.md                   # QA 角色（执行者）
        ├── reviewer.md             # Reviewer 角色（评审者）
        ├── plan-template.md        # 项目规划模板
        └── prd-template.md         # PRD 模板
```

## 许可证

MIT
