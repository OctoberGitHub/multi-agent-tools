# agent-update

AI Agent 协作协议 — 定义 Agent 间如何协作完成软件开发任务。

## 概述

`agent-update` 是一套形式化的 Agent 协作协议，定义 Agent 在软件开发中的角色分工、状态转换、工作流程和质量标准。不同于非结构化 prompt，本协议通过**状态机守卫条件**、**输入/输出契约**和**自检清单**约束 Agent 行为，目标是让 Agent 产出的质量和过程可预测、可审计。

## 四角色模型

| 角色 | 职责 | 核心产出 |
|------|------|----------|
| **Analyst** | 需求澄清、影响面分析、验收确认 | spec 文档、acceptance-result |
| **Planner** | 将 spec 拆解为 task，为每个 task 制定执行计划 | backlog.md、plan.md |
| **Executor** | 按 plan 编码实现 | 代码、execution-report.md |
| **Reviewer** | 审查代码质量、沉淀经验 | review-report.md、memory 更新 |

## 状态机

feature 任务走 7 状态流：

```
pending → planning → implementing → accepting → reviewing → completed
                                                                
                    ┌─ FAIL(function) → implementing              │
                    │  FAIL(code)     → implementing              │
                    └─ FAIL(design)   → planning                  │
                                                                  
任何状态 → blocked（输入缺失/需确认/外部依赖不可用）→ 解除后回原状态
```

bug-fix 任务走简化流：`pending → implementing → completed`。

## 快速开始

1. 读取 [INIT.md](INIT.md) 完成项目初始化（检测技术栈、创建 `agent-entry/` 目录结构）
2. 通过 Analyst 形成 spec 文档
3. Analyst 影响面分析 → Planner 拆解为 task → 用户确认 backlog
4. 逐 task 执行：Plan → Implement → Accept → Review → 沉淀

## 目录结构

```
agent-entry/
├── rules/        # 约束规则（framework/ + 技术栈/ + custom/ + generated/）
├── specs/        # spec 文档
├── memory/       # 经验沉淀（gotchas / decisions / conventions）
├── tasks/        # 任务队列（backlog.md + current-task.md）
├── outputs/      # 任务产出（plan / execution-report / review-report）
├── snapshots/    # 断点恢复快照
└── metrics/      # 质量度量
```

## 核心原则

- **按行为切，不按层切**：每个 task 是可独立验证的行为单元，禁止按技术层拆分
- **依赖层级一致性**：同一 task 内所有行为的依赖层级必须一致
- **契约即守卫**：状态转换由输入/输出契约完整性守卫，不满足则 blocked
- **沉淀闭环**：同类 gotchas ≥3 次自动触发规则提议

## 关键文档

| 文件 | 说明 |
|------|------|
| [AGENTS.md](AGENTS.md) | 完整协作协议（角色、状态机、工作流、约束） |
| [ANALYST.md](ANALYST.md) | 分析师角色详述 |
| [INIT.md](INIT.md) | 项目初始化管线 |
| [CHANGELOG.md](CHANGELOG.md) | 版本变更历史（v0.1.0 → v3.6.0） |
