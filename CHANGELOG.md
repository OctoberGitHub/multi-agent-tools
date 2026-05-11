# CHANGELOG

> 按时间倒序排列。每个版本标注 SHA1 和核心变更，重大版本附详细清单。

---

## v3.6.0

- **SHA1**: `4d8e65c`（含未提交工作树变更）

**主题**：Analyst 角色独立 + Bootstrap 初始化管线 + 状态机 accepting 状态

引入 Analyst 角色接管需求端和验收端，新增 Bootstrap 初始化流程，在 implementing→reviewing 之间插入 accepting 状态，使流程从"Planner→Executor→Reviewer"三角色演变为"Analyst→Planner→Executor→Reviewer"四角色。

**关键变更：**

| # | 变更项 | 说明 |
|---|--------|------|
| 1 | **新增 Analyst 角色** | 独立角色定义（ANALYST.md），职责涵盖需求澄清、影响面分析、外部 spec 归档、验收确认 |
| 2 | **新增 accepting 状态** | 在 implementing→reviewing 之间插入 Analyst 验收环节，验收通过（PASS）方可进入 reviewing |
| 3 | **新增 Bootstrap 初始化** | INIT.md 定义 6 步初始化管线：自检目录→检测技术栈→整理 rules→分析项目结构→生成规则→填充 memory |
| 4 | **TL;DR 重写** | 加入初始化流程、需求形成流程、Analyst 验收环节、FAIL(function/code/design) 三类回退 |
| 5 | **章节重组（3 章→9 章）** | 新增一、项目初始化；四、分析师；六、Bug-fix 独立章节；七、任务拆分原则；八、最小可用栈；九、质量度量 |
| 6 | **目录结构细化** | rules/ 拆为 4 层（framework/ + 技术栈/ + custom/ + generated/），specs/ 加入子目录结构 |
| 7 | **全局原则扩充** | 规则内核优先级链（custom > generated > 技术栈 > framework）、跨域优先级链（rules > memory > skill）、流程跳过需确认原则 |
| 8 | **工作流新增步骤 0** | Spec 形成环节归入 Analyst，含归档和需求澄清两条路径 |
| 9 | **backlog 条目规范增强** | 新增 spec_ref、priority 字段，task_id 和 spec_ref 统一命名格式 |
| 10 | **plan.md 规范细化** | 验收标准由 Analyst 在 spec 中定义，Planner 直接引用不重复编写；新增业务架构沉淀清单 |
| 11 | **状态转换规则重构** | 清除冗余守卫条件（如 planning→blocked 由通用阻塞规则覆盖），新增上游依赖验收前置检查 |
| 12 | **范围漂移处理收缩** | 从 15 行→5 行（保留三方角色分工步骤），与 v3.5.0 文档精简方向一致 |

**涉及文件**：
- AGENTS.md：216 行变更（+283 / -83），全篇章节目录重编
- ANALYST.md：新增独立角色定义文件
- INIT.md：新增 6 步初始化管线

---

## v3.5.0

- **SHA1**: `407092aeee183db2d0d5be10ab485ffbbcf96740`

**主题**：任务拆分新增依赖层级一致性原则 + 文档精简

解决两个问题：（1）Planner 在归类跨层级行为时凭直觉合并，导致 backlog 依赖声明与 plan 实际依赖硬伤不一致（典型案例：T-005 将「仅依赖 T-001 的分销商统计」与「依赖 T-001~T-004 的手动 Job」合并）；（2）文档存在重复信息、格式瑕疵和技术债。

**关键变更 — 新增依赖层级一致性原则：**

| # | 变更项 | 说明 |
|---|--------|------|
| 1 | 新增 §5.2 依赖层级一致性原则 | 定义依赖层级 = 前置 task 集合；同一 task 内所有行为层级必须一致 |
| 2 | 提供反例与正例对照 | 反例：跨层合并；正例：按 L0/L1/L2 分层，同层可并行 |
| 3 | 骨架合并例外 | 允许骨架搭建与首个最简行为合并（T-001 模式），骨架视为零层级 |
| 4 | Planner 项目级规划自检清单 | 输出 backlog 前必须通过 4 项自检（前置集合一致性、跨层检查、依赖拓扑图、跨层合并理由） |
| 5 | Planner 项目级规划约束强化 | 拆解完成后需画出依赖拓扑图（标注每个 task 所在层级），与 backlog 一起提交用户确认 |
| 6 | 5.3 拆分自检新增一条 | 「每个 task 内所有行为的依赖层级是否一致」 |
| 7 | 章节编号调整 | 原「核心原则」升级为 §5.1，新增 §5.2，原「拆分自检」降为 §5.3 |

**关键变更 — 文档精简（B 档）：**

| # | 变更项 | 说明 |
|---|--------|------|
| 8 | 修复格式 bug | §5.2 "禁止"条目加上 `-` 前缀，列表对齐 |
| 9 | 删除 blocked 触发冗余引用框 | 状态转换表已完整覆盖，删除重复说明 |
| 10 | 删除两处"适用场景"行 | 章节标题自解释 |
| 11 | 验收标准规范压缩 | 6 行 → 3 行（保留 1 正 1 反例） |
| 12 | 范围漂移处理压缩 | 15 行 → 5 行（保留三方角色分工步骤） |
| 13 | 向后兼容说明迁移此处 | AGENTS.md 中移除，历史留档附在下方 |

综合效果：406 行 → ≈375 行，规则条款零损失。

**原「向后兼容」说明（历史留档）**：

> 旧 task 无 `spec_sha1` / `auto_check_spec` 字段时，`spec_sha1` 视为空（跳过 SHA1 检查），`auto_check_spec` 视为 `false`，`spec_mode` 为空时视为 `strict`。行为等同于旧版流程。

---

## v3.4.0

- **SHA1**: `fe634494e389a4b5cffa440dbe521bfed031f918`

**主题**：Planner 两级规划模式

解决「spec → backlog」的拆解步骤无明确归属问题。

**关键变更：**

| # | 变更项 | 说明 |
|---|--------|------|
| 1 | Planner 项目级规划模式 | 输入 spec → 输出 backlog.md 中多个 task 条目（含 task_id/summary/scope/depends_on/verification） |
| 2 | backlog 条目规范 | 每个 task 必须包含行为目标、范围边界、验证方式；禁止"创建 XX 文件"式描述 |
| 3 | 工作流补充步骤 0 | 用户提交 spec → Planner 拆解 → 用户确认 backlog → 进入执行流程 |
| 4 | TL;DR 更新 | 反映完整的两级规划流程 |
| 5 | 未来展望补充 | 记录多 Agent 场景下独立 Decomposer 角色方案（方案 C） |

---

## v3.3.0

- **SHA1**: `2584608858a20958798edc95f88a379bb957d1f6`

**主题**：强化 Plan 质量、Executor 边界约束与任务拆分规范

解决 plan 过于简单导致 Executor 仅以编译通过为目标、任务依赖间无法检测产出缺失、按层拆分导致无法端到端验证的问题。

**关键变更：**

| # | 变更项 | 说明 |
|---|--------|------|
| 1 | Planner 输出契约增强 | plan.md 新增 4 项必含内容：接口契约、上游依赖清单、stub/TODO 范围、数据流描述 |
| 2 | 验收标准规范化 | 禁止仅以"编译通过"作为验收条件，必须是行为级断言（含正/反例） |
| 3 | 状态机守卫条件加强 | planning→implementing 新增接口契约/依赖清单/stub范围/数据流检查；implementing→reviewing 新增"非 stub 逻辑无 TODO + snapshot 已更新" |
| 4 | Plan 自检清单 | Planner 提交前必须通过 5 项自检 |
| 5 | Executor 协作协议补充 | 必须实现的逻辑不得留 TODO；执行完成前自检接口契约和上游依赖 |
| 6 | Reviewer 审查增强 | 新增依赖完整性审查、TODO 合规审查、snapshot 完整性检查 |
| 7 | snapshot 机制强化 | 从"按需"升级为"必备"，纳入状态机守卫和 Reviewer 审查 |
| 8 | 新增「任务拆分原则」章节 | 明确"按行为切不按层切"原则，含正反例对比和拆分自检清单 |
| 9 | 质量度量补充 | 新增 Plan 维度（自检通过率、因 plan 不足导致的 rework） |
| 10 | 一致性修复 | reviewing 入口动作补全 plan.md；snapshots 目录标注从"按需"改为"必备" |

---

## v3.2.0

- **SHA1**: `92bbea135002ebe1059ee88d9b8209d28e94abac`

优化 spec 协作流程：
- `spec_mode` 默认 `strict`，生成 task 时预填充，取消 planning 前每次确认的阻塞点
- 新增 spec SHA1 追踪机制：`spec_sha1` 和 `auto_check_spec` 字段
- 状态机 `pending→planning` / `pending→blocked` 守卫条件更新

---

## v3.1.0

- **SHA1**: `d1ca0a03365c9a41464ad28c0bb815b5c5173310`

补全任务分流机制：
- TL;DR 后加入 `task_type` 字段说明（`feature`/`bugfix`）
- 目录结构展开 `rules/` 子文件（review-rules.md + business-rules.md）

---

## v3.0.0

- **SHA1**: `90031c7650d571f12b36b2bf01aa9e01bb4c7a08`

架构重构：模式决策点前置。
- 文档拆为「三、功能开发」和「四、Bug-fix」两大独立章节，各自闭环
- 全局原则提至二节，作为共享上下文

---

## v2.3.0

- **SHA1**: `2bc426831835f846122cce6a73aec551cb6800bd`

重构工作流章节：
- 拆为「功能开发流程」和「Bug-Fix 流程」两条明确路径
- 断点恢复、阶段切换信号、范围漂移处理归属功能开发流程

---

## v2.2.0

- **SHA1**: `17e4755b392824bc348d4211f45de9e000ee9412`

新增 Bug-fix 快捷路径：
- Bug-fix 跳过 planning 和 reviewing，走 `pending→implementing→completed` 简化流
- 新增 Bug-Fix 执行流程（scope 确认→修复→验证→沉淀）

---

## v2.1.0

- **SHA1**: `d90c06ece872b8a5aa4166ede2ab65b55de17080`

新增范围漂移处理机制：
- Executor 发现产出偏离 plan 范围时主动标记 blocked
- Planner 评估后调整 backlog.md 再解除 blocked

---

## v2.0.0

- **SHA1**: `4068a56d82b9c0794632f087459cafe275254a5a`

精简：
- 删除 reworking 状态（合并回 implementing）
- Memory Manager 合并到 Reviewer
- 消除 3 处信息重复
- acceptance.md 改为推荐而非强制
- 5 阶段拆解改为推荐而非强制

---

## v1.1.0

- **SHA1**: `1128f8f91104ba9ef98d37bea6523a888dcfa5de`

Planner 输出契约增加：
- 5 阶段拆解（skeleton/mvp/feature/test/deliver）
- acceptance.md
- 新增六、最小可用栈

---

## v1.0.0

- **SHA1**: N/A（历史版本未记录）

增加 7 状态（pending/planning/implementing/reviewing/reworking/completed/blocked），4 角色（Planner/Executor/Reviewer/Memory Manager）。

---

## v0.1.0

- **SHA1**: N/A（历史版本未记录）

初始原型。3 状态（planning/implementing/reviewing），3 角色（Planner/Executor/Reviewer）。

---

## 未来展望

> 三大方向：文档优化 → 部署架构 → 角色演进（触发门槛依次升高）

### 一、文档优化

#### 1. 按需拆分

将三、功能开发和四、Bug-fix 各自独立为 `rules/feature-workflow.md` 和 `rules/bugfix-workflow.md`，AGENTS.md 缩减为 ~50 行的入口文件。

| 场景 | 当前加载 | 优化后加载 | 节省 |
|------|---------|-----------|------|
| feature 任务 | AGENTS.md 全量 | AGENTS.md（50 行）+ feature-workflow（120 行） | ~170 行（-35%） |
| bugfix 任务 | AGENTS.md 全量 | AGENTS.md（50 行）+ bugfix-workflow（50 行） | ~100 行（-62%） |

**触发条件**：AGENTS.md 超过 500 行，或出现第三条工作流路径，或单次任务仅需文档 20% 内容的不对称超过 5:1。

**不提前拆的理由**：AGENTS.md 是每 session 一次的固定成本，真正吃掉上下文的是代码库、对话历史、工具输出。

---

### 二、部署架构

#### 2. 多 Agent 协作

当前 AGENTS.md 定义角色与工作流，不绑定"谁来执行"。多 Agent 场景下角色分配在用户级配置中定义。

| 层级 | 职责 | 配置位置 |
|------|------|---------|
| 项目级 | 角色、状态机、工作流、输入/输出契约 | AGENTS.md |
| 用户级 | 角色到 agent 的映射关系（部署拓扑） | `~/CLAUDE.md`、`agent-entry/rules/` |

**触发条件**：实际部署了 ≥2 个独立 Agent 实例。

---

### 三、角色演进

#### 3. Decomposer 角色独立

当前"拆解"由 Planner 的项目级规划承担（方案 B）。多 Agent 场景下应独立为 **Decomposer** 角色（方案 C）：

```text
用户提交 spec → Decomposer → backlog.md → Planner → plan.md
```

| 维度 | 方案 B（Planner 两级规划） | 方案 C（独立 Decomposer） |
|------|------------------------|------------------------|
| 职责 | Planner 兼顾项目级+任务级 | Decomposer 专精拆解，Planner 专精执行规划 |
| 审查 | 自我审查 | 天然独立审查点 |
| 适用 | 单 Agent | 多 Agent |
| 代价 | 职责膨胀 | 上下文需显式传递 |

**触发条件**：实际部署 ≥2 个独立 Agent 实例，或项目级规划拆解质量反复出问题，或 backlog 超过 10 个 task 且依赖关系复杂。

**迁移路径**：将 Planner 的"项目级规划"模式抽取为独立的 Decomposer 角色定义，原有 backlog 输出规范直接复用。

---

## 阶段性评估存档

> 评估时点：v3.5.0 ｜ 视角：OpenAI / Anthropic 工程师 ｜ AGENTS.md 当前状态：385 行

### 整体评分

| 视角 | 评分 | 侧重点 |
|------|------|--------|
| OpenAI 工程师 | **8.0 / 10** | 可执行性与通用性 |
| Anthropic 工程师 | **8.2 / 10** | 指令遵循精度与可解释性 |

### 一、优点

| # | 优点 | 评价 |
|---|------|------|
| 1 | **状态机严谨，守卫条件具体** | `planning → implementing` 的守卫条件列出 6 项前置（spec_mode + 验收标准 + 接口契约 + 上游依赖清单 + stub 范围 + 数据流），spec SHA1 变更检测解决了 spec 漂移 |
| 2 | **Plan 自检清单是行业级亮点** | 5 条自检把「plan 质量」从模糊主观评价变为可勾选清单，与 Anthropic pre-flight checks 思路契合且更严格 |
| 3 | **依赖层级一致性原则** | 市面上首次看到对「任务拆分」做形式化定义（`依赖层级 = 前置 task 集合`），工程级严谨 |
| 4 | **blocked 优先的反脆弱设计** | 与 Claude 4 "prefer pausing over hallucinating" 原则一致 |
| 5 | **5 阶段切换信号表** | 把开发直觉转化为 Agent 可判定规则 |
| 6 | **沉淀闭环（gotchas ≥3 → rule_proposal）** | 具备自举改进能力，大部分协议只记录不升级 |

### 二、不足

#### P0 严重缺失

| # | 问题 | 后果 |
|---|------|------|
| 1 | 缺少可观测性/审计层 | 没有事件日志，出问题无法回溯「为什么这个 task 卡 3 小时」 |
| 2 | 缺少 Cost/Timeout Budget | Agent 可能在 reviewing 无限 FAIL → implementing 循环 |
| 3 | Memory 检索机制未定义 | gotchas 越积越多会变成几 MB 文本，Agent 无法精确定位 |

#### P1 重要缺失

| # | 问题 | 后果 |
|---|------|------|
| 4 | Skill 使用契约模糊 | 何时发现、何时调用、skill 之间如何组合均未定义 |
| 5 | 多 Agent 并发语义缺失 | 同层 task 并行时的协调/锁/合并策略未定 |
| 6 | 产物 Schema 未形式化 | plan/execution-report 是自由 Markdown，Reviewer 靠肉眼 + 正则 |
| 7 | 人机交互协议弱 | blocked 后用户看什么、如何 reply、timeout 怎么办均未定 |
| 8 | 错误严重级缺失 | 只有 code_issue / design_issue 二分，缺 severity 和 category |

#### P2 细节改进

| # | 问题 |
|---|------|
| 9 | 元规则缺失：AGENTS.md 自己的修改流程未定义 |
| 10 | 安全边界单薄：只提了 git reset，缺 secrets/生产数据/外部 API 隔离 |
| 11 | §6 最小可用栈与实际脱节：提到 git，但本项目禁用 git 命令 |
| 12 | Bug-fix 工作流过于单薄：根因定位方法论（5 Whys、二分定位）完全缺失 |

### 三、进化方向（Roadmap）

#### 阶段一：工程化（v4.x）

| 方向 | 说明 | 价值 |
|------|------|------|
| 事件日志层 | 每次状态转换产生 JSONL：`{ts, taskID, from, to, actor, reason}` | 可观测、可审计、可重放 |
| 产物 Schema 化 | plan.md/execution-report.md 加 YAML frontmatter，定义必填字段 | Reviewer 可自动化校验 |
| Budget 机制 | task 定义 max_tokens / max_duration / max_iterations | 防止死循环失控 |
| 错误分类体系 | severity × category 矩阵，对应不同 rework 策略 | FAIL 后处理更精准 |

#### 阶段二：多 Agent 协作（v5.x）

| 方向 | 说明 | 价值 |
|------|------|------|
| 并发锁/分支策略 | memory 的更新走 CRDT 或事务 | 支持真正并行 |
| Agent 协商协议 | 同层 task 冲突时的投票/仲裁机制 | 减少 user 介入 |
| Role specialization | Planner/Executor/Reviewer 可挂载不同模型 | 成本与质量平衡 |

#### 阶段三：自适应进化（v6.x）

| 方向 | 说明 | 价值 |
|------|------|------|
| Memory 向量化检索 | gotchas/decisions 建索引，Agent 按相似度检索 | memory 规模可扩展至 GB |
| metrics 闭环自动优化 | `一次通过率 < 60%` → 自动触发规则审视 | 自举改进 |
| Meta-rules | AGENTS.md 的修改本身遵循一套规则 | 防止协议自毁 |
| A/B 并行执行 | 同一 task 让 2 个 Agent 跑，Reviewer 比较结果 | 提升产出质量 |

#### 阶段四：Agent 操作系统（v7+）

| 方向 | 说明 |
|------|------|
| Capability-based permissions | 替代「git reset 可回滚」的粗粒度安全，做到文件级/API 级权限 |
| Skill 市场 | Skill 具有签名、版本、依赖，像 npm 一样可分发 |
| Spec DSL | spec 从自然语言升级为半形式化 DSL，直接编译为 backlog |
| 自愈能力 | 发现 rules 冲突时自动提议修改而非 blocked |

### 四、一句话总结

> 这份 AGENTS.md 已经超过 90% 开源社区可见的同类文档，在「状态机严谨度」「任务拆分理论」「自检清单」三项上达到工业级水准。核心短板在**运行时层缺失**（可观测性/Budget/Schema），这是从「写得好的协议」走向「能跑在生产环境的 Agent OS」的必经之路。
