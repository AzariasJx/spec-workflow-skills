---
name: spec-workflow-project-update
description: "更新项目进度、阶段状态、决策记录，并同步项目日志。通过 .eo-project.json 定位项目管理侧（.spec-workflow/）。触发：更新项目 / 同步进展 / 记录决策 / 阶段完成 / /spec-workflow-project-update。"
---

# spec-workflow-project-update

## 功能

更新项目的进度、阶段状态、决策记录，同步刷新项目日志。操作目标为项目管理侧（`.spec-workflow/`）文件：`roadmap.md`、`phases/`、`decisions/`、`log.md`。

## 前置

**必须**能找到 `.eo-project.json`（cwd 或父目录）。找不到时报错退出，提示运行 `/spec-workflow-project-init`。

## 输入

- **进度更新**："D6 完成了"、"任务 X 搞定"
- **阶段切换**："phase 1 完成，进入 phase 2"
- **决策记录**："决定用 Vue 而不是 React"
- **阻塞标记**："被 XX 卡住了"
- **自由更新**：其它情况追加 log

## 路径解析

从 `.eo-project.json` 读取：
- `project_root` — 项目管理侧绝对路径（即 `<repo>/.spec-workflow/`），所有项目侧文件均在此目录下

所有 skill 内部操作一律通过 `project_root` 定位，不硬编码。

## 执行步骤

### 1. 解析更新类型

| 类型 | 信号词 | 操作 |
|------|--------|------|
| 任务完成 | "完成"、"搞定"、"done" | 勾选 phase 中的任务 |
| 阶段切换 | "阶段完成"、"进入下一阶段" | 切换 phase status |
| 决策记录 | "决定"、"选择"、"决策" | 创建 `decisions/` 文件（目录不存在时 lazy 建） |
| 阻塞标记 | "卡住"、"blocked"、"阻塞" | 记录阻塞原因到 log |
| 自由更新 | 其他 | 追加 log |

### 2. 执行对应操作

#### 任务完成

1. 读取当前活跃 phase（`<project_root>/phases/phase-N-*.md` 中 `status: active`）
2. 找到匹配的 `- [ ]` 改为 `- [x]`
3. 更新 phase frontmatter 的 `updated`

#### 阶段切换

1. 当前 phase `status: done`
2. 下一个 phase `status: active`
3. 更新 `<project_root>/roadmap.md` 阶段概览
4. 若全部完成，提示是否归档

#### 决策记录

1. `<project_root>/decisions/` 若不存在则 lazy 创建
2. 读取最大编号 +1，创建 `decisions/{NNN}-{决策简述}.md`：

```markdown
---
type: decision
project: "项目名"
date: YYYY-MM-DD
status: accepted
---

# {决策简述}

## 背景

{为什么要做这个决策}

## 选项

1. **{选项A}** — {优劣}
2. **{选项B}** — {优劣}

## 决定

选择 {选项X}，因为 {理由}。

## 后果

{预期影响}
```

3. 从用户输入提炼填充，推断不了的标 "待补充"

#### 阻塞标记

记录阻塞原因到 log.md。阻塞信息直接写入日志条目，不依赖看板。

### 3. 追加项目日志

在 `<project_root>/log.md` 顶部追加：

```markdown
## [YYYY-MM-DD] {update_type} | {简述}
- {具体变更}
```

### 4. 输出摘要

向用户展示：
- 更新了什么
- 当前项目状态（阶段 + 进度 + 下一步）

## 输出

- Phase 文件更新：`<project_root>/phases/phase-N-*.md`
- 决策文件（按需）：`<project_root>/decisions/NNN-*.md`
- 项目日志：`<project_root>/log.md`

## 约束

- 不修改 `roadmap.md` 的目标和阶段划分（只更新状态标记）
- 决策记录一旦 `accepted` 不可修改内容，只能新建覆盖
- 所有路径通过 `.eo-project.json` 解析，不硬编码
- `project_root` 不可达 → 报错提示检查配置
- 不涉及看板操作（无 kanban）
- lazy 创建 `decisions/`（本 skill 首次写决策时建）
