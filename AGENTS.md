# AGENTS 操作手册（Project Agent Entry）

## TL;DR

```text
初始化：    读取 INIT.md → 自动创建 agent-entry/ → 检测技术栈 → 准备就绪

需求形成：  口述需求 → Analyst 需求澄清 → spec → 用户确认
           外部 spec → Analyst 归档 → specs/{S-ID}/
功能开发：  spec → Analyst 影响面分析 → Planner 拆解为 Tasks（用户确认）
           → 拾取任务 → Planner 出 plan → Executor 执行
           → Analyst 验收（功能正确？）→ Reviewer 审查（代码质量？）→ completed
           → FAIL(function): Executor 返工 | FAIL(code): Executor 返工 | FAIL(design): Planner 重规划
           → 输入缺失: blocked

Bug-fix：  拾取任务 → 确认 scope → 定位修复 → 验证
          → 修复生效: completed + 沉淀 | 修复不生效: blocked
```

任何角色遇到"输入契约缺失"或"需用户确认"均进入 `blocked`，不自主推进。

> **任务分流**：task 通过 `backlog.md` 中的 `task_type` 字段分流——`feature` 走 [五、功能开发](#五功能开发)，`bugfix` 走 [六、Bug-fix](#六bug-fix)。

---

## 一、项目初始化（Bootstrap）

首次使用：读取 [INIT.md](INIT.md) 按步骤执行初始化。`agent-entry/` 目录由 INIT.md 自动创建，无需手动准备。

初始化完成后，项目进入可工作状态，后续功能开发和 Bug-fix 按以下章节执行。

---

## 二、目录结构说明

```text
agent-entry/
├── rules/       # 强约束（必备）
│   ├── framework/    # 框架流程规则（模板自带，始终存在，不可变）
│   │   ├── workflow-rules.md   # 工作流规则
│   │   └── git-rules.md        # Git 操作规则
│   ├── java/         # 技术栈默认规则（bootstrap 检测匹配后保留）
│   │   ├── implementation-rules.md  # 实现规范
│   │   └── test-rules.md            # 测试规范
│   ├── custom/       # 开发者可插拔规则（按需添加，优先级最高）
│   ├── generated/    # bootstrap 生成的项目专属规则（自动生成）
│   │   ├── project-structure.md
│   │   └── verification-cmds.md
│   ├── go/           # 其他技术栈规则占位（bootstrap 不匹配则删除）
│   ├── python/
│   └── node/
├── specs/       # spec 文档（必备）
│   └── {S-ID}/                  # S-ID 命名格式：S-{YYYYMMDD}-{序号}，如 S-20260509-001
│       ├── spec-draft.md       # Analyst 产出（待确认）
│       ├── spec.md             # 正式 spec 文档（含验收标准）
│       └── analysis.md         # 影响面分析（按需）
├── memory/      # 经验沉淀（必备）
│   ├── business-architecture.md # 业务架构基线
│   ├── gotchas.md               # 踩坑经验
│   ├── decisions.md             # 架构决策
│   ├── conventions.md           # 项目约定
│   └── index.md                 # 记忆索引
├── tasks/       # 任务队列（必备）
│   ├── current-task.md
│   └── backlog.md
├── outputs/     # 任务产出（必备）
│   └── {taskID}/
│       ├── plan.md              # Planner 输出（含验收标准节）
│       ├── acceptance.md        # 复杂任务独立验收文件（按需）
│       ├── execution-report.md  # Executor 输出
│       └── review-report.md     # Reviewer 输出
├── skills/      # 可复用技能（按需）
├── snapshots/   # 断点恢复快照（必备）
├── other/       # Bug-fix 总结等（按需）
└── metrics/     # 质量度量（按需）
```

---

## 三、全局原则

- 执行前必须先读取 rules
- 优先使用已有规范，不重复造轮子
- 重要决策必须写入 memory
- 业务架构变更必须写入 `memory/business-architecture.md`
- 临时猜测不能当长期规则
- 优先级链（规则内）：`custom/ > generated/ > {技术栈}/ > framework/`，冲突时高优先级覆盖
  - framework/ 管流程，{技术栈}/ 管实现规范，custom/ 管团队偏好，generated/ 管项目事实
- 优先级链（跨域）：`rules > memory > skill`，冲突时遵循高优先级并记录
- 任务结束后必须输出沉淀总结：新坑（gotchas）、可复用经验（conventions）、架构决策（decisions）、业务架构沉淀、是否需要新增 rule/skill
- **流程跳过需确认**：若执行者判断当前步骤（spec 产出、task 拆解等）粒度过小、无需执行，不得自行跳过，必须向用户说明理由并询问确认。用户同意后方可省略对应产出物。
- **不可擅自跳过任何步骤**：所有步骤必须按流程执行，即使是简单任务。如执行者认为某步骤可以跳过，必须先向用户说明理由并取得同意。

---

## 四、分析师（Analyst）

> Analyst 角色完整定义见 [ANALYST.md](ANALYST.md)。职责：需求澄清、影响面分析、外部 spec 归档、验收确认（功能正确性验证，是 Reviewer 审查的前置条件）。

---

## 五、功能开发

### 5.1 状态机

| 状态 | 说明 | 入口动作 | 出口动作 |
|------|------|----------|----------|
| `pending` | 任务在 backlog 中排队 | 记录入队时间 | — |
| `planning` | Planner 拆解任务 | 读取 rules/skills/memory | 输出 plan.md |
| `implementing` | Executor 执行代码 | 读取 plan.md | 更新 snapshot |
| `accepting` | Analyst 验收 | 读取 spec + execution-report | 输出 acceptance-result |
| `reviewing` | Reviewer 审查 + 沉淀 | 读取 plan.md + acceptance-result + diff | 输出 review-report + 更新 memory |
| `completed` | 任务完成 | — | 归档 snapshot |
| `blocked` | 暂停，等待输入或用户确认 | 记录阻塞原因 | 解除后回到源状态 |

**通用阻塞规则**：任何状态在以下情况发生时 → `blocked`：输入缺失、需用户确认、外部依赖/环境不可用。解除后回到原状态。

**状态转换规则**（仅列出非阻塞的核心转换；阻塞规则见上方通用规则）：

| 源状态 | 目标状态 | 触发条件 | 守卫条件 |
|---|---|---|---|
| pending | planning | 任务被拾取 | 优先级最高 且 无未满足依赖 且 spec_mode 已预填 且（auto_check_spec 未启用 或 spec SHA1 一致 或 用户已确认变更） |
| pending | blocked | spec SHA1 变化且 auto_check_spec 启用 | 等待用户确认是否采用最新 spec |
| planning | implementing | plan.md 生成 | plan.md 含 spec_mode + 接口契约 + 上游依赖清单 + stub范围 + 数据流 且 无未解决 assumption 且 所有上游依赖 task 的输出已通过验收 |
| implementing | accepting | 执行完成 | 编译通过 且 plan 中所有非 stub 标记的逻辑已实现（无遗留 TODO）且 snapshot 已更新至最终步骤 |
| implementing | blocked | 实际产出偏离 plan 范围（多/少实现），可能影响下游 task | 标注受影响的下游 task，等待 Planner 调整 backlog |
| accepting | reviewing | 验收通过 | acceptance-result = PASS |
| accepting | implementing | 验收不通过 | acceptance-result = FAIL(function) |
| reviewing | completed | 审查通过 | Analyst 已验收通过 且 review-report 无 blocking issue |
| reviewing | implementing | 审查有代码问题 | review-report 含 code_issue → Executor 返工 |
| reviewing | planning | 审查有设计问题 | review-report 含 design_issue → Planner 重规划 |
| blocked | 原状态 | 阻塞原因解除 | `current-task.md` 中标注 resolved |

### 5.2 角色定义

#### 规划者（Planner）

Planner 在两个粒度上工作，核心逻辑相同：分析输入 → 拆解为可执行步骤 → 输出结构化文档。

| 粒度 | 触发条件 | 输入 | 输出 |
|------|---------|------|------|
| **Spec → Tasks** | 用户提交新 spec 指示拆解，或 backlog 为空且存在未拆解的 spec | spec、`rules/`、`skills/`、`memory/` | `tasks/backlog.md` 中的 task 条目 |
| **Task → Plan** | 任务从 backlog 拾取进入 planning 状态 | `tasks/current-task.md`、`rules/`、`skills/`、`memory/` | `outputs/{taskID}/plan.md` |

**backlog.md 中每个 task 条目必须包含**：
- `task_id`、`task_type`、`priority`、`spec_mode`、`spec_ref`
  - `task_id` 命名格式：`T-{YYYYMMDD}-{序号}`，如 T-20260509-001
  - `spec_ref` 命名格式：`S-{YYYYMMDD}-{序号}`，如 S-20260509-001
- `summary`：一句话描述该 task 的行为目标（不是”创建 XX 文件”，而是”实现 XX 行为”）
- `depends_on`：上游 task ID 列表
- `scope`：本 task 负责的行为边界（做什么 + 不做什么）
- `verification`：如何验证该 task 完成（必须是可运行的验证方式）

**plan.md 必须包含**：
- spec 协作确认结果（`spec_mode: strict | collaborative`）
- 任务拆解步骤（含依赖）。**复杂任务建议按 5 阶段拆**：skeleton（搭骨架）→ mvp（最小可运行）→ feature（补全能力）→ test（修边界）→ deliver（整理交付），每阶段定义切换信号
- **接口契约**：每个对外暴露的方法必须定义签名、参数语义、返回值结构、异常处理策略
- **上游依赖清单**：具体到方法/字段级别
- **stub/TODO 允许范围**：明确哪些逻辑本 task 必须实现、哪些允许留 stub 并标注后续哪个 task 负责填充
- **数据流描述**：从入口到出口，数据如何流转、谁调谁
- 业务架构沉淀清单、风险评估与缓解
- `outputs/{taskID}/acceptance.md`（复杂任务推荐独立文件，小任务可写在 plan.md 验收标准节）

> 验收标准由 Analyst 在 spec 中定义（见 [ANALYST.md](ANALYST.md)），Planner 在 plan.md 中直接引用，不重复编写。

**约束**：
- 必须遵循「七、任务拆分原则」，禁止按层切
- 每个 task 必须是可独立验证的行为单元
- **同一 task 内所有行为的依赖层级必须一致**（见 §7.2），骨架合并为唯一例外
- Spec → Tasks 拆解完成后需画出依赖拓扑图，与 backlog 一起提交用户确认

**协作协议**：
- `spec_mode` 默认为 `strict`，生成 task 时已在 backlog 中预填充
- strict 模式：严格按 spec 生成 plan，不自行发散；如需发散，用户修改 task 的 spec_mode 即可
- collaborative 模式：允许发散，但与 spec 不一致处须再次确认
- 不确定事项写入 `assumptions` 节并确认，不确定项 ≥2 个自动触发协作模式

**Plan 自检清单**（提交 plan.md 前必须通过）：
- [ ] 验收标准引用是否指向 spec 中定义的标准？
- [ ] 能否仅凭 plan 写出集成测试？
- [ ] 上游依赖是否具体到方法/字段？
- [ ] stub/TODO 范围是否明确标注？
- [ ] Reviewer 能否据此判断”做没做完”？

#### 执行者（Executor）

**输入**：`plan.md`、`acceptance.md`（如有）、`rules/`、`skills/`

**输出**：
- 代码实现
- 每完成一个子步骤必须更新 `snapshots/{taskID}-progress.md`（含已完成步骤列表、当前进度、下一步计划）
- `outputs/{taskID}/execution-report.md`：已完成步骤、与 plan 偏差说明、业务架构变更记录、编译验证结果

**协作协议**：
- 严格按 plan 执行，偏差必须记录
- **发现实际产出偏离 plan 范围（多实现或少实现），立即标记 blocked，在 execution-report 中列出受影响的下游 task**
- strict 模式下不得实现超出 spec 的内容
- 遇到 plan 未覆盖情况，暂停并标记 blocked
- **plan 中标记为"必须实现"的逻辑不得留 TODO**，仅 plan 中明确标注为 stub 的部分允许留 TODO
- 执行完成前必须自检：plan 中列出的接口契约是否全部满足、上游依赖是否全部对接

#### 审查者（Reviewer）

**输入**：`execution-report.md`、`acceptance.md`（如有）、`plan.md`（核验接口契约和依赖清单）、`rules/review-rules.md`、`rules/business-rules.md`、`memory/business-architecture.md`、代码 diff

**输出**：
- `outputs/{taskID}/review-report.md`：审查结论（PASS / FAIL）、问题清单（severity / category / description / suggestion）、可沉淀的经验
- code_issue → 回到 implementing；design_issue → 回到 planning
- 审查必须包含自动化验证（编译、测试）
- **依赖完整性审查**：校验 plan 中列出的上游依赖是否全部对接（如 Mapper 方法是否存在、SQL 是否实现）；校验本 task 承诺给下游的接口是否全部就绪
- **TODO 合规审查**：仅 plan 中标注为 stub 的部分允许存在 TODO，其余 TODO 视为 blocking issue
- **snapshot 完整性检查**：确认 `snapshots/{taskID}-progress.md` 存在且最终状态为 done，步骤列表与 execution-report 一致

**沉淀职责**：
- PASS 后，将本次经验按分类写入 `memory/`：
  - gotchas → `memory/gotchas.md`
  - 架构决策 → `memory/decisions.md`
  - 项目约定 → `memory/conventions.md`
  - 业务架构变更 → `memory/business-architecture.md`
- 更新 `memory/index.md` 索引
- **自动规则提议**：同一类 gotchas 出现 ≥3 次，在 `memory/index.md` 标记 `rule_proposal:pending`

### 5.3 工作流

```text
0. [Spec 形成] 以 Analyst 为中心形成 spec 文档：
   - 有 spec 文档（外部传入或已有）→ Analyst 归档到 specs/{S-ID}/ + 影响面分析
   - 无 spec 文档 → Analyst 需求澄清 → 产出 spec-draft → 用户确认 → 正式 spec

1. [项目级规划] spec → Planner 拆解为多个 task → 输出 backlog.md → 用户确认
   - backlog 中每个 task 标注 spec_ref 指向 specs/{S-ID}/

2. 拾取最高优先级任务 → 写入 current-task.md

3. 检查 spec 变更：若 `auto_check_spec` 启用，对比当前 spec SHA1 与 task 记录的 `spec_sha1`，不一致则需用户确认是否采用最新 spec

4. Planner 拆解任务 → 输出 plan.md（验收标准引用 spec 定义，不重复编写）

5. 检查上游依赖：若 task 有 `depends_on`，验证所有被依赖 task 的输出已通过验收

6. Executor 按 plan 执行 → 更新 snapshot → 输出 execution-report

7. [Analyst 验收] 验收执行结果 → 输出 acceptance-result
   - PASS → 走步骤 8
   - FAIL(function) → 回到步骤 6

8. [Reviewer 审查] 审查代码质量 → 输出 review-report
   - PASS → 归档
   - FAIL(code) → 回到步骤 6
   - FAIL(design) → 回到步骤 4

9. 回到步骤 2，拾取下一任务
```

**断点恢复**：
- 读取 `snapshots/{taskID}-progress.md`，从最近完成的步骤继续
- 快照不存在则从 plan.md 第一步重新开始

**snapshot 格式规范**（`snapshots/{taskID}-progress.md`）：
```markdown
# {taskID} 进度快照

## 基本信息
- task_id: T-XXX
- plan_steps_total: N
- current_step: M
- last_updated: YYYY-MM-DD HH:mm

## 已完成步骤
- [x] step 1: 描述（产出物/关键文件）
- [x] step 2: 描述
- [ ] step 3: 描述（← 下一步）

## 当前状态
- status: in_progress | blocked | done
- blocking_reason: （如有）
- notes: 执行中发现的问题/备注
```

**阶段切换信号**（复杂任务适用）：

Planner 定义每阶段切换信号，Reviewer 据此判断执行质量。**中后段（test/deliver 阶段）禁止新增功能模块**，需求变更须经 Planner 重新规划。

| 阶段 | 完成标志 | 违规信号（触发 design_issue） |
|------|---------|------------------------------|
| skeleton | 可编译，目录结构就绪 | 还在纠结技术选型 |
| mvp | 主流程可运行 | 主流程未跑通就开始铺功能 |
| feature | 核心功能逐个补齐 | 同时展开 3 个以上模块 |
| test | 单测通过，边界已覆盖 | 还在新增功能 |
| deliver | README 可复现，依赖完整 | 文档缺失但还在修代码 |

**范围漂移处理**：长程任务中某 task 可能多/少实现，导致下游范围需动态调整。

1. [Executor] 发现偏离 → blocked，在 execution-report 列出：多/少实现项 + 受影响下游 task ID
2. [Planner] 评估下游影响（缩减/扩展/删除） → 更新 backlog.md → 解除 blocked
3. Executor 不得自行调整下游范围；调整后的 task 需重新确认 spec_mode；经验写入 `memory/gotchas.md`

---

## 六、Bug-fix

### 6.1 状态机

| 状态 | 说明 | 入口动作 | 出口动作 |
|------|------|----------|----------|
| `pending` | Bug-fix 任务在 backlog 中排队 | 记录入队时间 | — |
| `implementing` | 定位根因 + 修复 + 验证 | 读取 `current-task.md` | 输出修复 + execution-report |
| `completed` | 修复生效，经验已沉淀 | 更新 memory | 归档 |
| `blocked` | 修复不生效或引入新问题 | 记录阻塞原因 | 解除后回到 implementing |

**状态转换规则**：

| 源状态 | 目标状态 | 触发条件 | 守卫条件 |
|---|---|---|---|
| pending | implementing | 任务被拾取 | `current-task.md` 已写明 bug 现象、复现步骤、预期修复后行为 |
| pending | blocked | scope 未确认 | `current-task.md` 缺少 bug 现象 / 复现步骤 / 预期行为 |
| implementing | completed | 修复生效且未引入新问题 | 复现脚本通过 |
| implementing | blocked | 修复不生效或引入新问题 | 当前方案不行，需重新定位 |
| blocked | implementing | 阻塞原因解除 | 新的定位方案已确认 |

### 6.2 工作流

**执行**：

```text
1. [确认 scope] 在 current-task.md 写明 bug 现象、复现步骤、预期修复后行为
2. [implementing] 定位根因 → 修复 → 跑复现脚本确认修复生效
3. [判断]
   - 修复生效且未引入新问题 → completed，进入反馈沉淀
   - 修复不生效或引入新问题 → blocked（重新定位）
```

**反馈沉淀**：

```text
1. bug fix 总结 → agent-entry/other/
2. 按维度分类写入 memory/：
   - gotchas → memory/gotchas.md
   - 架构决策 → memory/decisions.md
   - 项目约定 → memory/conventions.md
   - 业务架构变更 → memory/business-architecture.md
3. 更新 memory/index.md 索引
4. 检查：是否导致已有 decisions 过时？（标记 superseded_by）
         是否暴露 rules 不足？（更新 rules）
         是否重复出现？（触发自动规则提议，同类 gotchas ≥3 次 → rule_proposal:pending）
```

---

## 七、任务拆分原则

### 7.1 核心原则：按行为切，不按层切

每个 task 应该是一个**可独立验证的行为单元**，而非一个技术层的空壳。

| 拆分方式 | 示例 | 问题 | 推荐度 |
|----------|------|------|--------|
| ❌ 按层切 | T-001: Entity+Mapper, T-002: Service, T-003: Controller | 每个 task 只产出空壳，无法端到端验证 | 禁止 |
| ✅ 按行为切 | T-001: "写入目标存储的批处理行为", T-002: "对外查询行为" | 每个 task 自包含，可独立跑通 | 推荐 |
| ✅ 混合切 | T-001: skeleton(全层骨架+主流程跑通), T-002: 补充查询能力 | 骨架阶段允许跨层，但必须跑通主流程 | 允许 |

### 7.2 依赖层级一致性原则

**定义**：同一 task 内所有行为的**依赖层级必须一致**。

- **依赖层级** = 一个行为能启动所需的前置 task 集合
- 两个行为层级"一致" ⇔ 它们所需的前置 task 集合完全相同
- **禁止**：将「只依赖 T-01」的行为与「依赖 T-01~T-04」的行为合并到同一 task

**反例（违反规则）**：

```text
❌ T-0X：行为A + 行为B
   ├─ 行为A → 仅依赖 T-01 骨架             (L1)
   └─ 行为B → 依赖 T-01~T-04 全部           (L4)
   问题：backlog 若声明 depends_on=T-01，则行为B 会因上游缺失而 blocked
         若声明 depends_on=T-01~T-04，则行为A 被不必要地延后
```

**正例（符合规则）**：

```text
✅ 按层级拆分
L0: T-01 骨架 + 首个独立行为
L1: T-02, T-03, T-04, T-05  （同构独立行为，均仅依赖 T-01，可并行）
L2: T-06 聚合行为, T-07 对外接口行为  （均依赖 T-01~T-05 全部，可并行）
```

**例外：骨架合并**

允许骨架搭建与首个最简行为合并到同一 task（如 T-001 模式）。此时骨架视为"零层级"不计入依赖判定，首个行为的层级等同于该 task 的层级。

**Planner 项目级规划自检**（输出 backlog.md 前必须通过）：

- [ ] 每个 task 内的所有行为，前置 task 集合是否完全相同？
- [ ] 是否存在跨层级合并的 task？（有 → 拆开，除非是骨架合并例外）
- [ ] 画出依赖拓扑图：每个 task 位于哪一层？同层是否可真正并行？
- [ ] 若合并跨层行为，plan 中是否明确标注"跨层级合并理由"？

**违反后果**：
- backlog 依赖声明与 plan 实际依赖出现硬伤不一致
- Executor 拾取任务时可能因上游未就绪而 blocked
- 声称的并行性不可信

### 7.3 拆分自检

- 每个 task 是否有可运行的验证方式？（不能只靠编译）
- 删掉某个 task 后，其他 task 是否仍可独立验证？（是 → 耦合度合理）
- 是否存在"只产出空壳，无行为可验证"的 task？（有 → 合并到下游 task）
- 每个 task 内所有行为的依赖层级是否一致？（不一致 → 拆开）

---

## 八、最小可用栈

| 层级 | 组件 | 就绪标准 |
|------|------|----------|
| 模型 | 核心执行者 | 能完成"读文件→改代码→跑命令"闭环 |
| 执行器 | loop 或调度器 | 能连续运行 ≥3 轮不丢失状态 |
| 状态文件 | `tasks/` + `memory/` + `snapshots/` | 任一角色读取状态文件可独立恢复执行 |
| 工具 | shell、git、文件读写 | 能完成"创建→编译→git commit"全链路 |
| 验证 | lint、test、run script | 至少有一个可执行的验证脚本 |
| 安全 | 权限控制、git 回滚 | `git reset --hard` 可恢复，敏感操作有拦截 |

---

## 九、质量度量

- **任务维度**：完成率、一次通过率、rework 次数、阶段切换违规次数
- **Plan 维度**：自检清单通过率、因 plan 不足导致的 rework 次数
- **Skill 维度**：使用频次、关联 rework 次数
- **Memory 维度**：近 N 个任务中被引用次数、自动规则提议数与采纳率

详细数据维护在 `metrics/` 目录下。
