# 项目初始化手册

当 `AGENTS.md` 被放入全新项目时，读取本文档完成首次初始化。
初始化完成后 `agent-entry/` 目录就绪，后续开发走 `AGENTS.md` 定义的功能开发流程。

---

## 初始化管线

### Step 0：自检目录

检测 `agent-entry/` 是否存在：

- **不存在** → 按 `AGENTS.md` §一 的目录结构定义，创建以下全量目录和文件：

  ```
  agent-entry/
  ├── rules/
  │   ├── framework/      ← 从 AGENTS.md §一 获取
  │   ├── custom/
  │   ├── generated/
  │   ├── go/
  │   ├── python/
  │   └── node/
  ├── specs/
  ├── memory/
  ├── tasks/
  │   ├── current-task.md
  │   └── backlog.md
  ├── outputs/
  ├── skills/
  ├── snapshots/
  ├── other/
  └── metrics/
  ```

- **存在** → 跳过，确认是否需要重新初始化

### Step 1：检测技术栈

扫描项目根目录，识别以下特征文件：

| 特征文件 | 识别结果 |
|----------|----------|
| `pom.xml` / `build.gradle` / `settings.gradle` | Java |
| `go.mod` | Go |
| `package.json` | Node.js |
| `Cargo.toml` | Rust |
| `requirements.txt` / `pyproject.toml` | Python |
| 无法识别任一 | → blocked，提示用户提供 |

**完成标志**：明确输出项目所用技术栈。

### Step 2：整理 rules/ 目录

基于 Step 1 的检测结果：

- 保留匹配的技术栈目录（如检测到 Java 则保留 `rules/java/`）
- 删除其余所有不匹配的技术栈目录（`rules/go/`、`rules/python/`、`rules/node/` 等）
- 创建 `rules/custom/`（可插拔规则入口，初始为空）
- 创建 `rules/generated/`（项目专属规则输出目录）

**完成标志**：`rules/` 下仅存在 `framework/`、匹配的技术栈目录、`custom/`、`generated/`。

### Step 3：分析项目结构

采集以下项目事实：

- 构建工具及其版本
- 源码目录、测试目录、配置目录
- 模块列表（多模块项目的各子模块）
- 主要框架和中间件
- 包命名约定、分层约定
- 配置文件位置
- 日志框架和日志目录

**完成标志**：输出完整的项目结构事实清单。

### Step 4：生成 rules/generated/

基于 Step 3 的采集结果，生成以下文件：

**`rules/generated/project-structure.md`**：
- 模块清单、目录结构快照
- 构建命令
- 主要技术栈版本摘要
- 配置文件和日志位置

**`rules/generated/verification-cmds.md`**：
- 全量编译命令
- 最小单元测试命令
- 定向模块测试命令（多模块项目）
- 集成测试命令
- 静态检查命令

**完成标志**：生成的文件可被后续任务的 Planner 和 Executor 引用。

### Step 5：填充 memory/

- `memory/project-architecture.md`：写入项目事实，bootstrap 状态设为 `completed`
- `memory/business-architecture.md`：占位，标注"待业务任务补充"，bootstrap 状态设为 `completed`
- `memory/index.md`：更新索引，指向更新后的文件

**完成标志**：`memory/` 下所有文件一致且状态为 `completed`。

### Step 6：输出报告

输出初始化报告（直接输出或写入 `agent-entry/outputs/`）：
- 检测结果、目录变更、生成文件清单
- 残余风险（如：无法识别的模块、缺失的验证命令）

**完成标志**：可明确回答项目技术栈、构建工具、模块结构、主要框架、验证命令。

---

## 初始化检查清单

- [ ] `agent-entry/` 目录结构完整
- [ ] `rules/` 已整理为 4 层结构（framework/ + 技术栈/ + custom/ + generated/）
- [ ] `rules/generated/project-structure.md` 已生成
- [ ] `rules/generated/verification-cmds.md` 已生成
- [ ] `memory/project-architecture.md` 状态为 `completed`
- [ ] `memory/business-architecture.md` 状态为 `completed`
- [ ] 可回答：技术栈、构建工具、验证命令
