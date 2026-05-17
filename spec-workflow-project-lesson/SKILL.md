---
name: spec-workflow-project-lesson
description: "捕获项目经验教训（踩坑、最佳实践、意外收获），写入当前项目的 lessons/ 目录。对踩坑类经验自动路由：项目特有写入项目级，全局通用同步写入 wiki/pitfalls/。通过 .eo-project.json 定位。触发：踩坑了 / 记录经验 / 教训 / lesson learned / /spec-workflow-project-lesson。"
---

# spec-workflow-project-lesson

## 功能

捕获项目经验教训，写入**项目级** `lessons/` 目录。对踩坑类经验实施**路由规则**：判断问题是项目特有还是全局通用，全局通用的问题同时写入工作区的 `wiki/pitfalls/P0-P2/` 目录。

配置与目录约定见 [spec-workflow-project-init/SKILL.md](../spec-workflow-project-init/SKILL.md)（project_root = `<repo>/.spec-workflow/`）。

## 前置

必须能找到 `.eo-project.json`。找不到 → 报错退出，提示运行 `/spec-workflow-project-init`。

## 输入

- **经验内容**：发生了什么、学到了什么
- 类别（可选，能推断就推断）：`pitfall` / `best-practice` / `surprise`

## 路径解析

从 `.eo-project.json` 读取：
- `project_root` — 项目管理侧根（`<repo>/.spec-workflow/`）
- `project_name` — 填入文件 frontmatter

lessons 目录：`<project_root>/lessons/`（**lazy 创建**，本 skill 首次运行时建）。

wiki pitfalls 目录：工作区根下的 `wiki/pitfalls/`（从 `.eo-project.json` 的 `project_root` 向上推导工作区根，即含 `wiki/` 目录的祖先）。

## 踩坑路由规则

**仅在类别为 `pitfall` 时执行路由判断。** `best-practice` 和 `surprise` 类别仅写入项目级 `lessons/`。

### 路由判断

| 判断 | 标准 | 目标 |
|------|------|------|
| **项目特有** | 仅当前项目会出现，涉及项目专属逻辑、UI、业务流程 | `<project_root>/lessons/` |
| **全局通用** | 可能在其他项目复现，属于通用技术问题 | `<project_root>/lessons/` **+** `wiki/pitfalls/P{0-2}/` |

### 判断依据

- **项目特有**示例：
  - "snake-game canvas 渲染在特定分辨率下闪烁" — 涉及具体游戏的渲染逻辑
  - "todo-app 拖拽排序在 Safari 触屏设备失效" — 特定项目的交互问题
  - "项目 X 的 API 网关配置导致超时" — 项目专属基础设施配置

- **全局通用**示例：
  - "JWT Token 过期处理不当" — 任何用 JWT 的项目都可能遇到
  - "React useEffect 依赖数组遗漏导致无限循环" — React 项目通用
  - "Git force push 丢失团队提交" — 任何 Git 项目都可能发生

### 优先级分级（全局通用踩坑）

| 优先级 | 条件 | 特征 | 目录 |
|--------|------|------|------|
| **P0** | 致命 | 数据丢失、安全漏洞、系统崩溃 | `wiki/pitfalls/P0/` |
| **P1** | 严重 | 核心功能受阻、性能严重下降 | `wiki/pitfalls/P1/` |
| **P2** | 一般 | 非核心功能、有 workaround | `wiki/pitfalls/P2/` |

优先级由问题的**潜在影响**决定，而非当前项目的严重程度——因为全局踩坑关注的是"其他项目遇到时的影响"。

## 执行步骤

### 1. 定位项目

读取 `.eo-project.json`，获取 `project_root` / `project_name`。

### 2. 提炼经验教训

从用户输入提取：
- **标题**：一句话概括
- **类别**：`pitfall` / `best-practice` / `surprise`
- **阶段**：从活跃 phase 文件读取（若有）
- **内容**：发生了什么 + 学到了什么 + 下次怎么做

### 3. 踩坑路由判断（仅 `pitfall` 类别）

当类别为 `pitfall` 时，执行路由判断：

1. 分析问题性质：是项目特有还是全局通用？
2. 向用户确认路由判断（"这个问题是否可能在其他项目中复现？"）
3. 若为全局通用：
   - 确定优先级（P0 / P1 / P2）
   - 向用户确认优先级
4. 路由结果记录在项目级经验的 frontmatter 中

### 4. 创建项目级经验文件

- 首次运行时 lazy 建 `<project_root>/lessons/` 目录
- 文件命名：`{YYYY-MM-DD}-{slug}.md`

```markdown
---
type: lesson
category: "pitfall" | "best-practice" | "surprise"
project: "项目名"
phase: "phase-N-阶段名"
date: YYYY-MM-DD
tags: []
routed: false | true
global_path: "" | "wiki/pitfalls/P1/xxx.md"
---

# {标题}

## 发生了什么

{背景和经过}

## 学到了什么

{核心教训}

## 下次怎么做

{可操作的改进方案}
```

- `routed: true` 且 `global_path` 非空表示已路由到全局 wiki
- 非 `pitfall` 类别 `routed` 固定为 `false`，`global_path` 为空

### 5. 创建全局踩坑文件（仅路由为全局通用时）

当路由判断为全局通用时，在 `wiki/pitfalls/` 写入全局踩坑文件。

确定工作区根：从 `project_root` 向上查找含 `wiki/pitfalls/` 目录的祖先。

- lazy 建对应优先级目录（`wiki/pitfalls/P0/` / `P1/` / `P2/`）
- 文件命名：`{slug}.md`

```markdown
---
priority: P0 | P1 | P2
date: YYYY-MM-DD
resolved: true | false
tags:
  - {tech-tag}
projects:
  - [[../../coding/projects/{project-name}]]
---

# {标题}

## 问题描述

{简明扼要描述问题}

## 问题代码

```{language}
// 导致问题的代码（如有）
```

## 问题分析

{问题原因分析}

## 解决方案

```{language}
// 正确的代码（如有）
```

## 预防措施

1. {预防建议}
2. {预防建议}

## 影响项目

- [[../../coding/projects/{project-name}]]
```

### 6. 更新全局踩坑索引（仅路由为全局通用时）

在 `wiki/pitfalls/index.md` 的对应优先级表格和"最近更新"表格中追加条目。

### 7. 追加项目日志

在 `<project_root>/log.md` 追加一条记录：

```markdown
## [{{date}}] lesson | {类别}：{标题}
- 文件：lessons/{date}-{slug}.md
- 路由：{未路由 | 已路由至 wiki/pitfalls/P{N}/{slug}.md}
```

### 8. 输出摘要

- 项目级经验文件路径
- 全局踩坑文件路径（如有路由）
- 经验摘要
- 当前项目累计经验数
- 路由结果说明

## 路由流程图

```
类别判断
    │
    ├── best-practice / surprise → 仅写项目级 lessons/
    │
    └── pitfall → 执行路由判断
                    │
                    ▼
              ┌─────────────────────────────┐
              │ 是否可能在其他项目中复现？   │
              └─────────────────────────────┘
                    │
                    ├── 否（项目特有）→ 仅写项目级 lessons/
                    │
                    └── 是（全局通用）→ 写项目级 lessons/ + 写 wiki/pitfalls/P{N}/
                                        │
                                        ▼
                                   ┌─────────────┐
                                   │ 优先级判断   │
                                   └─────────────┘
                                        │
                          ┌─────────┼─────────┐
                          │         │         │
                          ▼         ▼         ▼
                         P0        P1        P2
                        致命      严重       一般
```

## 输出

- 经验文件：`<project_root>/lessons/{date}-{slug}.md`
- 全局踩坑文件（可选）：`wiki/pitfalls/P{N}/{slug}.md`
- 全局索引更新（可选）：`wiki/pitfalls/index.md`
- 项目日志：追加记录

## 约束

- 经验文件一旦创建不可修改，只能新建补充
- 经验按项目隔离在 `lessons/`，全局踩坑按优先级隔离在 `wiki/pitfalls/P{0-2}/`
- 首次运行时 lazy 建 `lessons/` 目录
- 能推断的字段自动填充，路由判断需与用户确认
- 路径通过 `.eo-project.json` 解析，不硬编码
- 路由到全局时，项目级经验文件的 `routed: true` + `global_path` 标记确保可追溯
- 全局踩坑格式遵循 `wiki/pitfalls/README.md` 的模板
- 非 `pitfall` 类别不执行路由，直接写入项目级
- 向用户确认路由判断时提供判断依据，帮助用户做出正确决策
