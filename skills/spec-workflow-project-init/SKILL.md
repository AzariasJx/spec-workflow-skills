---
name: spec-workflow-project-init
description: "spec-workflow 在当前仓库的总入口：生成 .eo-project.json、初始化项目管理侧（roadmap/log/backlog）和代码侧最小骨架（eo-doc/），以及 agent 配置注入。触发：启动项目 / 初始化项目 / 新建项目 / /spec-workflow-project-init。"
---

# spec-workflow-project-init

## 定位

**所有 spec-workflow-* skill 的总入口**。其它 skill（spec-workflow-change / spec-workflow-implement / spec-workflow-doc-manager / ...）都依赖 `.eo-project.json`；未运行过本 skill 的项目无法使用其它 spec-workflow-* skill。

一次 init 完成三件事：
1. 生成 `.eo-project.json`（项目级配置，所有 skill 读它）
2. 初始化**项目管理侧**（local 模式）——最小骨架
3. 初始化**代码侧** `eo-doc/` 最小骨架（内部调用 `spec-workflow-doc-manager` init 的子流程）


## 输入

用户提供以下之一：
- **PRD/MVP 文档路径**
- **口头描述**：项目名称 + 要做什么 + 大致阶段
- **仅项目名**：快速创建空骨架（后续再补充）

可选：
- 代码仓库路径（当前 cwd 不是代码仓库时）
- 技术栈信息（语言/框架/工具）

## 执行步骤

### 1. 检查 cwd 是否已有 `.eo-project.json`

- 已有 → 走「更新/修复」分支（见 §6）
- 未有 → 继续 §2

### 2. 解析项目信息

从输入中提取：
- **项目名称**（`project_name`）
- **项目目标**：一句话描述（`description`）
- **技术栈**（`tech_stack`）：用户提到的语言/框架/工具，数组形式
- **初始状态**（`status`）：`active` / `researching`，默认 `active`
- **当前日期**（`created_at`）

### 3. 计算 `project_root`

固定规则：
- `project_root` = `<repo>/.spec-workflow/`

检查 `project_root` 是否已存在：
- 存在且含 `roadmap.md` → 询问：1) 只建代码侧关联 2) 更新 roadmap 3) 重建（需确认）
- 存在但无 `roadmap.md` → 异常，提示补全后进入拆解
- 不存在 → 正常创建

### 4. 创建项目管理侧骨架（最小）

```
<repo>/.spec-workflow/
├── roadmap.md     # 必建
├── log.md         # 必建
└── backlog.md     # 必建
```

**按需目录一律不预建**（phases / decisions / lessons / brainstorm / docs），等对应 skill 首次写入时由那个 skill 创建。

写入 `roadmap.md`：

```markdown
---
type: roadmap
project: "{{project_name}}"
status: "{{status}}"
created: "{{date}}"
updated: "{{date}}"
---

# {{project_name}}

## 目标

{{project_goal}}

## 阶段概览

| 阶段 | 目标 | 交付物 | 状态 |
|------|------|--------|------|
| _待拆解_ | | | |

## 备注

```

写入 `log.md`：

```markdown
# {{project_name}} 操作日志

## [{{date}}] init | 项目启动
- 来源：{{source}}
- 阶段数：{{phase_count}}
- 初始状态：{{status}}
```

写入 `backlog.md`：

```markdown
---
type: backlog
project: "{{project_name}}"
updated: "{{date}}"
---

# {{project_name}} Backlog

> 待办池 + 未接入的未来规划。

## 待办

## 灵感 & 以后再说

## 未接入（等 skill 支持再接入）
```

### 5. Roadmap 拆解（可选）

如果用户提供了 PRD/MVP 或愿意拆解：

1. **明确终态**：这个项目做完，用户能做到什么之前做不到的事？
2. **逆推里程碑**：从终态往回推，找出必须经过的中间状态
3. **里程碑 → Phase**：每个里程碑对应一个 Phase，控制在 2-5 个
4. **Phase 内任务拆解**：每个任务 0.5-2 天粒度，遵循 MECE 原则
5. 用户确认后，**lazy 创建** `phases/` 目录，每个阶段一个文件：

```markdown
---
type: phase
project: "{{project_name}}"
phase: {{phase_number}}
status: "{{phase_status}}"
created: "{{date}}"
updated: "{{date}}"
---

# Phase {{phase_number}}：{{phase_name}}

## 目标

{{phase_goal}}

## 任务

{{task_checklist}}

## 交付物

{{deliverables}}
```

6. 更新 `roadmap.md` 的阶段概览表

仅"快速创建空骨架"时可跳过此步。

拆解对话不超过 5 轮，避免过度讨论——先落地，再迭代。

### 6. 初始化代码侧 `eo-doc/`（内部调用 spec-workflow-doc-manager init 子流程）

在代码仓库根目录创建**最小骨架**：

```
eo-doc/
├── agent-handbook/INDEX.md   # 骨架
├── dev/INDEX.md              # 骨架
└── templates/                # 空目录
```

**不创建** `state/`（首次 `/spec-workflow-doc-manager sync` 时按需建）。
**不创建** `design/` / `research/` / `knowledgebase/`。

额外：
- 初始化 `eo-doc/.sync-cursor`（当前 HEAD 作为首次基线）
- 将 `eo-doc/.sync-cursor` 追加到 `.gitignore`
- CLAUDE.md 注入（详见 §9）

**注意**：如果用户本次只想要项目管理侧（例如纯规划项目，没代码），可用 `--skip-code-side` 跳过本步。此时 `doc_root` 字段仍写入配置，留待将来补建。

### 7. 生成 `.eo-project.json`

在**代码仓库根目录**写入：

```json
{
  "project_name": "{{project_name}}",
  "project_root": "{{repo_root}}/.spec-workflow/",
  "doc_root": "eo-doc",
  "created_at": "{{date}}",
  "status": "{{status}}",
  "tech_stack": ["{{tech_1}}", "{{tech_2}}"],
  "description": "{{project_description}}"
}
```

所有路径使用绝对路径。

### 8. 处理 `.spec-workflow/` 的 gitignore

默认追加到 `.gitignore`：

```
# spec-workflow local management side
.spec-workflow/
```

若用户明确想让管理侧随仓库提交，当场询问后跳过 gitignore 追加。

### 9. Agent 配置注入

检测代码仓库使用的 agent 配置文件（顺序）：
1. `CLAUDE.md`
2. `AGENTS.md`
3. `COPILOT.md`
4. `CURSOR.md`
5. 都不存在 → 询问用户创建哪个

使用 `<!-- spec-workflow:start -->` / `<!-- spec-workflow:end -->` 标记段落幂等注入：

```markdown
<!-- spec-workflow:start -->
## Spec-Workflow

本项目通过 `.eo-project.json` 关联到项目管理侧：`.spec-workflow/`

- 项目管理侧（roadmap / backlog / decisions / lessons 等）：`.spec-workflow/`
- 代码侧文档：`eo-doc/`

### 待办提醒

当对话中出现以下任一信号时，主动提示：

- **显性表达**："以后要做"、"TODO"、"先跳过"、"回头处理"、"以后再说"、"暂不处理"
- **隐性行为**：用户做了 workaround、临时方案、手动操作、跳过了某个步骤

> 检测到待办事项：「{内容}」。要加入项目 backlog 吗？

用户确认后**必须调用 `/spec-workflow-backlog` skill** 追加到 `.spec-workflow/backlog.md`。禁止手动写文件。

### 决策同步

当对话中出现关键技术决策时，主动提示。信号包括：

- **显性表达**：选型结论、架构方案确定、取舍决定、"用 X 不用 Y"
- **隐性行为**：用户明确否定了某个方案、对比后做了选择、改变了原有设计

> 这是一个关键决策。要记录到 decisions/ 吗？

用户确认后，在 `.spec-workflow/decisions/` 创建决策记录（首次时 lazy 建目录）。

### 经验教训

当对话中出现以下任一信号时，主动提示：

- **负面体验**："太慢了"、"不好用"、"很卡"、"效率低"、"不方便"、"浪费时间"
- **后悔/反思**："踩坑"、"下次不这么干"、"学到了"、"不应该"、"早知道"
- **改进建议**："应该用 X"、"不如 X"、"建议 X"、"以后要 X"、"下次注意"
- **隐性信号**：用户对某流程表达了不满、主动提出优化方案、纠正了某个做法

> 要记录到 lessons/ 吗？

用户确认后**必须调用 `/spec-workflow-project-lesson` skill**，由 skill 处理路由判断（项目级 vs 全局 wiki/pitfalls/）和文件创建。禁止手动写文件，禁止跳过路由判断。
<!-- spec-workflow:end -->
```

### 10. 处理 `.eo-project.json` 的 gitignore

将 `.eo-project.json` 提交到仓库（团队共享配置），**不加入** `.gitignore`。

### 11. 处理已有配置（更新/修复分支）

当 §1 检测到 `.eo-project.json` 已存在时：
1. 读取现有配置，展示给用户
2. 询问意图：修复损坏的骨架 / 更新项目信息 / 重建
3. 按用户选择执行对应操作，保持幂等

### 12. 同步更新 `log.md`

在 `log.md` 追加本次操作记录：

```markdown
## [{{date}}] init | 项目初始化完成
- 模式：local
- 项目管理侧：.spec-workflow/
- 代码侧骨架：eo-doc/
- 阶段数：{{phase_count}}
```

### 13. 验证完整性

检查所有必建文件是否成功创建：
- `.eo-project.json` 存在且可解析
- `.spec-workflow/roadmap.md` 存在
- `.spec-workflow/log.md` 存在
- `.spec-workflow/backlog.md` 存在
- `eo-doc/agent-handbook/INDEX.md` 存在（未跳过代码侧时）
- `eo-doc/dev/INDEX.md` 存在（未跳过代码侧时）
- `eo-doc/templates/` 目录存在（未跳过代码侧时）
- agent 配置文件中包含 `<!-- spec-workflow:start -->` 标记

如有缺失，当场补建。

### 14. 输出摘要

展示：
- 项目名称、状态、描述
- `.eo-project.json` 路径和内容
- 项目管理侧骨架结构（`.spec-workflow/`）
- 代码侧骨架结构（`eo-doc/`）
- `.gitignore` 变更
- agent 配置注入状态
- 可选的 phases 拆解结果
- 提示后续可用 skill：
  - `/spec-workflow-doc-manager` — 管理代码侧文档
  - `/spec-workflow-backlog` — 管理 backlog
  - `/spec-workflow-change` — 发起变更
  - `/spec-workflow-implement` — 实现代码

## 输出

- **代码仓库**：`.eo-project.json` + `eo-doc/` 最小骨架 + agent 配置注入
- **项目管理侧**：`.spec-workflow/` 含 `roadmap.md` / `log.md` / `backlog.md`（+ 可选 `phases/`）

## 约束

- **`.eo-project.json` 是所有 spec-workflow-* skill 的启动前置**。本 skill 的核心产出
- 固定 local 模式，`project_root` 始终为 `<repo>/.spec-workflow/`
- `.eo-project.json` schema 使用 `project_name` / `project_root` / `doc_root` / `created_at` / `status` / `tech_stack` / `description` 字段，不含 `mode` / `kanban_path`
- 按需目录（phases / decisions / lessons / brainstorm / docs）**init 时不预建**，由对应 skill 首次写入时 lazy 创建
- 项目名用用户给的原始名称，不转换
- 原始 PRD/MVP 若提供，存到 `.spec-workflow/docs/`（lazy 建）
- `.spec-workflow/` 默认进 `.gitignore`；用户可当场覆盖
- `.eo-project.json` 提交到仓库，不进 `.gitignore`
- agent 配置注入使用 `<!-- spec-workflow:start/end -->` 标记，幂等可重复执行
- 所有 skill 引用使用 `spec-workflow-` 前缀

## Skill 启动时的配置解析

**除本 skill 外的所有 spec-workflow-* skill 启动时：**

1. 从 cwd 向上查找 `.eo-project.json`（到文件系统根为止）
2. 找不到 → 报错并退出：
   ```
   未找到 .eo-project.json
   请先运行 /spec-workflow-project-init 初始化项目。
   ```
3. 找到 → 解析其内容，后续一律用其中的路径

## 目录结构参考

### 项目管理侧（`<repo>/.spec-workflow/`）

```
.spec-workflow/
├── roadmap.md     # 必建
├── log.md         # 必建
├── backlog.md     # 必建（待办池 + 未接入的未来规划）
├── phases/        # 按需，roadmap 拆解后生成
├── decisions/     # 按需，首次记录决策时建
├── lessons/       # 按需，首次记录经验时建
├── brainstorm/    # 按需，spec-workflow-brainstorming 首次产出时建
└── docs/          # 按需，原始 PRD / 设计 / 规划
```

### 代码侧（仓库内 `eo-doc/`）

```
eo-doc/
├── agent-handbook/   # 必建，代码架构（AI）
├── dev/              # 必建，spec/change/review 流
├── templates/        # 必建（空），spec-workflow-* 扩展点
└── state/            # 按需，系统当前状态描述（sync 时首建）
```
