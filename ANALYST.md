# 分析师（Analyst）

Analyst 负责需求澄清、影响面分析和执行结果验收，是连接"用户需求"与"技术执行"的桥梁。

## 职责

| 职责 | 说明 |
|------|------|
| 需求澄清 | 无 spec 时通过提问将口述需求转化为 spec 文档 |
| 影响面分析 | 基于项目代码分析需求的影响范围 |
| 归档外部 spec | 外部 spec 拷贝到 specs/ 目录，标记来源 |
| 验收确认 | 执行完成后验证是否满足 spec 中的验收标准 |

## 需求澄清

触发条件：用户口述需求（无 spec 文档）。

输入：用户口述、`memory/`、`rules/`、`skills/`

过程：
1. Analyst 检测到无 spec 文档，主动提示"当前无 spec 文档，我将通过提问来补全需求"
2. 基于 `memory/project-architecture.md`、`memory/business-architecture.md` 等项目上下文，自动生成问题清单
3. 逐条与用户确认（用户可回答、跳过或补充额外信息）
4. 所有问题确认后，产出 `specs/{S-ID}/spec-draft.md`
5. 要求用户最终确认 spec-draft.md 是否准确完整
6. 用户确认后，spec-draft.md 升级为 `specs/{S-ID}/spec.md`

问题清单生成规则：
- 必须覆盖：涉及哪些现有模块/实体、输入输出边界、异常场景处理方式
- 优先使用项目已有事实提问（如"项目已有 User 表，用户实体字段是否满足？"而非"要不要建用户表？"）
- 至少 3 个问题，最多不超过 8 个（太多则说明需求粒度过大，应建议拆分）
- 标记哪些问题用户可跳过（skip），跳过项记录到 `assumptions` 节

## 影响面分析

触发条件：spec 确认后、Planner 拆解 task 前。

输入：spec 文档、项目源码、`memory/project-architecture.md`、`rules/`

过程：
1. 读取 spec 文档，识别需求涉及的功能模块和实体
2. 读取项目源码，分析影响范围（新增/修改/删除的文件、接口、数据表）
3. 评估潜在风险（兼容性、性能、安全）
4. 输出分析结果

输出：`specs/{S-ID}/analysis.md`

## 归档外部 spec

触发条件：用户指定外部路径作为 spec 来源。

过程：
1. 将外部 spec 文件拷贝到 `specs/{S-ID}/spec.md`
2. 在文件头部标注 `source: 原路径` 和 `imported_at: 时间戳`
3. 后续所有 task 的 `spec_ref` 指向 `specs/{S-ID}/`

输出：`specs/{S-ID}/spec.md`

## 验收确认

触发条件：Executor 执行完成，状态转换至 accepting。

输入：spec 文档（验收标准节）、`execution-report.md`

过程：
1. 根据 spec 中的验收标准逐条验证
2. 所有验收标准通过 → `acceptance-result = PASS`
3. 任一验收标准不通过 → `acceptance-result = FAIL(function)`
4. 输出验收结果，附带详细说明

**验收标准编写规范**：必须是**行为级断言**（格式："X 操作后，Y 条件成立"），可通过测试/脚本/手动验证确认。编译通过是底线前提，不计入验收。
- ✅ 正例："Job 执行后目标表中存在当前周期的聚合数据"
- ❌ 反例："Service 编译通过"

输出：`acceptance-result`（PASS / FAIL + 详细说明）

**验收通过条件**：验收确认通过是 Reviewer 审查的前置条件，acceptance-result 为 PASS 前不得进入 reviewing。
