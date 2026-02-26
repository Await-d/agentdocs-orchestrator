# Agentdocs Orchestrator

一个高级任务编排系统，与 `.agentdocs/` 知识管理框架深度集成，专为复杂多步骤任务的分解、并行执行和状态同步而设计。

## 🎯 核心特性

### 1. 智能任务规划
- **自动创建工作流文档** - 为复杂任务自动生成结构化的规划文档
- **任务分解** - 将复杂请求拆解为独立的原子任务
- **依赖分析** - 自动识别任务间的依赖关系，构建 DAG 执行图

### 2. 多代理并行执行
- **三种执行模式** - OpenCode 原生 `task()` / 顺序执行 / Claude CLI
- **并行任务调度** - 自动识别可并行执行的任务组
- **状态实时追踪** - 实时更新任务执行状态（待执行、运行中、已完成、失败）

### 3. 运行时隔离
- **任务级隔离** - 每个任务拥有独立的运行时目录
- **并发任务支持** - 多个任务可同时运行而互不干扰
- **清晰的职责分离** - 规划文档（持久）与执行运行时（临时）分离

### 4. 状态同步机制
- **自动状态更新** - 任务完成后自动同步到工作流文档
- **进度可视化** - 使用状态图标清晰展示任务进度
- **完成后归档** - 已完成任务自动移至归档目录

---

## 📁 目录结构

```
.agentdocs/
├── index.md                      # 索引文档（知识入口）
├── workflow/                     # 任务规划文档（持久化）
│   ├── YYMMDD-task-name.md      # 任务分析、设计、待办事项
│   └── done/                     # 已完成任务归档
└── runtime/                      # 分布式执行运行时（临时）
    ├── YYMMDD-task-name/         # 每个任务拥有独立运行时
    │   ├── master_plan.md        # 该任务的编排计划
    │   ├── agent_tasks/          # 代理任务文件
    │   │   ├── agent-01.md
    │   │   └── ...
    │   └── results/              # 执行结果
    │       ├── agent-01-result.md
    │       └── ...
    └── YYMMDD-another-task/      # 另一个并发任务
        └── ...
```

**关键区别：**
- `workflow/` - 任务**规划与思考**（持久化，决策记录，应 commit 到 git）
- `runtime/<task-id>/` - 任务**分布式执行**（临时，每个任务隔离，应加入 `.gitignore`）

## 🗂️ Git 集成策略

`.agentdocs/` 的两个子目录有不同的 git 生命周期：

```bash
# 推荐的 .gitignore 配置
echo ".agentdocs/runtime/" >> .gitignore
```

| 目录 | git 策略 | 原因 |
|------|----------|------|
| `workflow/` | **提交** | 持久化决策记录，是项目文档的一部分 |
| `workflow/done/` | 按需保留 | 历史存档，可选是否保留 |
| `runtime/` | **忽略** | 临时执行协调空间，任务完成后删除 |
| `index.md` | **提交** | 知识入口，需要持久化 |

---

## 🚀 快速开始

### 步骤 1：初始化 agentdocs 结构

```bash
mkdir -p .agentdocs/workflow/done
mkdir -p .agentdocs/runtime
echo ".agentdocs/runtime/" >> .gitignore
```

如果 `.agentdocs/index.md` 不存在，创建一个空文件作为知识入口。

### 步骤 2：创建工作流文档

任务 ID 格式为 `YYMMDD-task-name`（例如：`260112-fix-audio-player`）。

```markdown
# .agentdocs/workflow/YYMMDD-task-name.md

## 任务概述
[任务的简要描述]

## 当前分析
[问题分析、约束条件和考虑因素]

## 解决方案设计
[高层次方法和关键决策]

## 实施计划

### 阶段 1：[阶段名称]
- [ ] T-01: [任务描述]
- [ ] T-02: [任务描述]

## 备注
[执行过程中的重要观察、阻塞或决策]
```

### 步骤 3：在索引中注册

```markdown
## 当前任务文档
`workflow/YYMMDD-task-name.md` - [任务简要描述]
```

---

## 🔄 核心五阶段工作流

| 阶段 | 关键动作 |
|------|---------|
| 1️⃣ 分析与规划 | 检查 index.md → 分析意图 → 创建工作流文档 → 拆解原子任务 |
| 2️⃣ 代理分配 | 创建 `runtime/<task-id>/` → 生成 master_plan.md → 分配 Agent |
| 3️⃣ 并行执行 | OpenCode `task()` / 顺序执行 / CLI 三选一 |
| 4️⃣ 结果聚合 | 收集 results/ → 按依赖顺序合并 → 生成 final_output.md |
| 5️⃣ 同步与清理 | 更新工作流 TODOs → 归档 → 清理 runtime/ |

**状态图标：** 🟡 待执行 | 🔵 运行中 | ✅ 已完成 | ❌ 失败 | ⏸️ 等待依赖

> 详细工作流说明参见 [SKILL.md](agentdocs-orchestrator/SKILL.md)

---

## 🎯 使用场景

在以下情况下激活此编排系统：

- ✅ 复杂的多步骤任务（3+ 步骤，存在可并行部分）
- ✅ 用户提到"并行"、"并发"、"子任务"、"代理"
- ✅ 需要任务分解、跨步骤状态追踪
- ✅ 需要通过 Claude CLI 执行子任务

**不需要激活：**

- ❌ ≤2 步的顺序任务（直接执行更高效）
- ❌ 单文件修改或简单重构

---

## 📚 依赖关系类型

```
串行依赖:    T-01 → T-02 → T-03

并行独立:    T-01 ─┬─→ T-04
             T-02 ─┤
             T-03 ─┘

DAG 依赖:    T-01 ───→ T-03
                 ╲   ╱
             T-02 ───→ T-04
```

---

## 📖 相关文档

| 文件 | 用途 |
|------|------|
| [SKILL.md](agentdocs-orchestrator/SKILL.md) | **主入口** — 触发条件、决策树、执行模式、完整行为规范 |
| [RULES.md](agentdocs-orchestrator/RULES.md) | 强制执行规则（方案先行、TDD、分段改动）|
| [workflow.md](agentdocs-orchestrator/workflow.md) | 详细工作流程说明（含调度算法）|
| [templates.md](agentdocs-orchestrator/templates.md) | 完整模板集合（master_plan、agent task、result、final output）|
| [cli-integration.md](agentdocs-orchestrator/cli-integration.md) | Claude CLI 深度集成（Mode C）|
| [examples.md](agentdocs-orchestrator/examples.md) | 实际使用示例（代码审查、翻译、API 测试、TDD Bug 修复）|

---

## 🔗 与 agentdocs 的集成

此编排器与更广泛的 `.agentdocs/` 知识系统集成：

- **执行前**：检查 `index.md` 查找相关架构文档、约束条件
- **执行期间**：引用现有文档获取上下文
- **执行后**：将新模式、决策或记忆更新到文档

---

## 📝 许可证

本项目是 agentdocs 知识管理框架的一部分。

---

**版本**: 1.1.0
**最后更新**: 2026-02-26
