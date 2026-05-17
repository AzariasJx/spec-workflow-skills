---
name: spec-workflow-wiki
description: "wiki 知识库与项目经验的集成查询/写入技能。支持全局 wiki/ 和本地项目经验的搜索、读取、写入，供 spec/change/implement 等技能调用。触发：查 wiki / 搜经验 / 写踩坑 / wiki search / /spec-workflow-wiki。"
---

# spec-workflow-wiki — 知识库集成查询与写入

wiki/ 全局知识库与本地项目经验的统一集成技能。提供跨层知识检索、结构化写入和每日汇总能力，供其他 spec-workflow 技能（spec / change / implement 等）在执行前调用，也支持用户直接使用。

## 定位

| 对比 | 本技能 | spec-workflow-project-lesson |
|------|--------|------------------------------|
| 核心动作 | **检索**已有知识 + 路由写入 | 捕获经验教训 |
| 调用方 | 其他技能在执行前调用做前置知识检查 | 用户遇到坑主动记录 |
| 输入 | 关键词 / 路径 / 主题 | 经验内容 |
| 产出 | 知识摘要 / 写入确认 | 经验文件 + 可选全局路由 |

**关键差异**：本技能侧重"查"和"轻量写"，project-lesson 侧重"结构化捕获 + 路由判断"。写入踩坑时，若需要完整的路由判断流程，应委托给 `/spec-workflow-project-lesson`。

## 前置

- `check` / `search` / `read` 命令：不依赖 `.eo-project.json`，只要有工作区根目录（含 `wiki/`）即可运行
- `write` 命令写入 `.spec-workflow/lessons/` 时：需要 `.eo-project.json`
- `daily` 命令：需要工作区根目录

### 可选：llmwiki-cli

[llmwiki-cli](https://www.npmjs.com/package/llmwiki-cli)（`llmwiki-cli@1.0.1`）是 wiki 知识库的命令行工具，提供 `wiki search/read/write/lint` 等命令。

```bash
# 安装（仅依赖 Node.js）
npm install -g llmwiki-cli

# 验证
wiki --version
```

**安装时的行为**：搜索优先使用 `wiki search` CLI，索引匹配更精准。

**未安装时的行为**：自动回退到 INDEX + frontmatter 匹配模式，功能不受影响，仅搜索效率略低。

### 可选：Obsidian

`wiki/` 目录本身就是 Obsidian vault。用 Obsidian 打开 `wiki/` 目录即可获得：

- **图谱视图**：可视化 wikilinks 关联
- **反向链接**：查看哪些页面引用了当前页面
- **每日笔记**：Obsidian 的 Daily Notes 插件可直接打开 `wiki/daily/` 下的文件
- **搜索**：全文搜索所有 wiki 内容

Claude Code 通过文件系统直接读写 `wiki/` 目录（本技能的 write 命令），Obsidian 通过其文件监听自动感知变更。两者无需额外配置即可共存。

**非项目文档记录**：直接在 Obsidian 中创建和编辑 `wiki/` 下的文件即可。学习笔记、资源收集、日常想法等非项目相关内容最适合用 Obsidian 手动管理。Claude Code 在后续执行 `check` / `search` 时会自动发现这些内容。

## 命令总览

| 命令 | 说明 | 依赖 |
|------|------|------|
| `search <keyword>` | 按关键词搜索 wiki/ + 本地项目经验 | 工作区根 |
| `read <path>` | 读取指定 wiki 或项目经验文档 | 工作区根 |
| `write <location> <content>` | 写入 wiki 或本地项目经验 | 视位置而定 |
| `check <topic>` | 综合检查：搜索所有知识源，返回结构化摘要 | 工作区根 |
| `daily` | 生成每日总结到 wiki/daily/YYYY-MM-DD.md | 工作区根 |

---

## 命令详细说明

### `check <topic>` — 综合知识检查

**最重要的命令**。其他技能（spec / change / implement）在开始工作前调用此命令，获取相关历史知识。

#### 搜索范围（按优先级）

1. `wiki/pitfalls/P0/` — 致命全局踩坑（最高优先级）
2. `wiki/pitfalls/P1/` — 严重全局踩坑
3. `wiki/pitfalls/P2/` — 一般全局踩坑
4. `wiki/learning/` — 学习笔记
5. `wiki/patterns/` — 设计模式
6. `wiki/decisions/` — 技术决策
7. `<project>/.spec-workflow/lessons/` — 项目级经验（若在项目目录下）
8. `eo-doc/dev/<module>/pitfalls/` — 模块级踩坑（若在模块目录下）

#### 搜索策略

优先使用高效搜索，避免全文扫描：

1. **优先尝试 LLMWiki CLI**：`wiki search <keyword>`
   - 可用时直接使用，CLI 内部已优化搜索
   - 支持 `--dir` 限定目录范围
2. **回退到 INDEX + frontmatter 匹配**：
   - 读取各子目录的 `INDEX.md` 或 `index.md`
   - 匹配 frontmatter 中的 `tags` / `title` / `summary` 字段
   - 仅对匹配的文件读取摘要段落（前 10 行）
3. **禁止全文 grep**：不使用 `grep -r` 全文扫描。知识库的检索入口是 INDEX 文件和 frontmatter

#### 输出格式

返回结构化摘要：

```markdown
## 知识检查结果：<topic>

### 全局踩坑（P0/P1）

- **[P1] JWT Token 过期处理不当** (`wiki/pitfalls/P1/jwt-token-expiry.md`)
  - 摘要：Token 过期后未刷新导致用户被踢出
  - 相关度：高（涉及认证流程）

### 全局踩坑（P2）

- **无相关记录**

### 学习笔记

- **React Hooks 闭包陷阱** (`wiki/learning/react-hooks-closure.md`)
  - 摘要：useEffect 依赖数组遗漏的常见模式

### 设计模式

- **无相关记录**

### 技术决策

- **无相关记录**

### 项目经验

- **Snake Game 渲染闪烁** (`.spec-workflow/lessons/2026-05-16-canvas-flicker.md`)
  - 摘要：特定分辨率下 canvas 重绘问题

### 模块级踩坑

- **无相关记录**

### 建议

- 在实现认证模块时参考 jwt-token-expiry 的预防措施
- React 状态管理注意闭包陷阱
```

#### 执行步骤

1. **解析上下文**：确定当前是否在项目目录下（查找 `.eo-project.json`），若是则获取项目路径和当前模块名
2. **构建搜索关键词**：从 `<topic>` 提取核心关键词（技术名、组件名、功能名）
3. **尝试 LLMWiki CLI**：运行 `wiki search <keyword>`，若成功则解析结果
4. **回退到 INDEX 匹配**（若 CLI 不可用或结果不足）：
   - 读取 `wiki/pitfalls/index.md` 匹配相关条目
   - 读取 `wiki/learning/index.md` 匹配相关条目
   - 读取各优先级目录下的文件 frontmatter
5. **搜索项目级经验**（若在项目目录下）：
   - 读取 `.spec-workflow/lessons/` 下的 INDEX 或文件列表
   - 匹配相关经验
6. **搜索模块级踩坑**（若在模块目录下）：
   - 读取 `eo-doc/dev/<module>/pitfalls/` 下的文件
7. **汇总输出**：按输出格式组织，标注相关度（高/中/低）

---

### `search <keyword>` — 关键词搜索

简化版搜索，只返回匹配结果列表。

#### 执行步骤

1. 尝试 `wiki search <keyword>`
2. 若不可用，读取 INDEX 文件做 frontmatter 匹配
3. 返回匹配文件列表（路径 + 标题 + 摘要）

#### 输出格式

```
搜索结果 "<keyword>"：

全局知识库：
1. wiki/pitfalls/P1/jwt-token-expiry.md — JWT Token 过期处理
2. wiki/learning/react-hooks-closure.md — React Hooks 闭包陷阱

项目经验（当前项目）：
3. .spec-workflow/lessons/2026-05-16-canvas-flicker.md — Canvas 渲染闪烁
```

---

### `read <path>` — 读取文档

读取指定的 wiki 或项目经验文档，返回完整内容。

#### 路径解析规则

| 输入格式 | 解析结果 |
|----------|----------|
| `pitfalls/P1/jwt-token-expiry` | `wiki/pitfalls/P1/jwt-token-expiry.md` |
| `learning/react-hooks` | `wiki/learning/react-hooks.md` |
| `lessons/canvas-flicker` | `.spec-workflow/lessons/YYYY-MM-DD-canvas-flicker.md` |
| 完整路径 | 直接读取 |

#### 执行步骤

1. 解析路径（补全 `.md` 后缀，按规则补全前缀）
2. 读取文件内容
3. 返回内容（含 frontmatter 元信息）

---

### `write <location> <content>` — 写入文档

将内容写入指定位置，同时更新相关索引。

#### 写入位置路由

| 位置标识 | 目标路径 | 说明 |
|----------|----------|------|
| `pitfalls/P0` | `wiki/pitfalls/P0/<slug>.md` | 致命全局踩坑 |
| `pitfalls/P1` | `wiki/pitfalls/P1/<slug>.md` | 严重全局踩坑 |
| `pitfalls/P2` | `wiki/pitfalls/P2/<slug>.md` | 一般全局踩坑 |
| `learning` | `wiki/learning/<slug>.md` | 学习笔记 |
| `daily` | `wiki/daily/YYYY-MM-DD.md` | 每日总结 |
| `decisions` | `wiki/decisions/<slug>.md` | 技术决策 |
| `patterns` | `wiki/patterns/<slug>.md` | 设计模式 |
| `lessons` | `.spec-workflow/lessons/<date>-<slug>.md` | 项目级经验（需要 .eo-project.json） |

#### 执行步骤

1. **解析位置和内容**：从用户输入提取目标位置、标题、内容
2. **确定目标路径**：按路由表解析完整路径
3. **生成文件内容**：
   - 自动填充 frontmatter（date / tags 从内容推断）
   - 按目标位置选择对应模板（见 wiki/SCHEMA.md 和 wiki/pitfalls/README.md）
4. **写入文件**：lazy 创建目标目录
5. **更新索引**：
   - wiki 类：更新 `wiki/pitfalls/index.md` 或 `wiki/learning/index.md` 等对应索引
   - lessons 类：更新 `.spec-workflow/log.md`
6. **确认输出**：返回写入路径和摘要

#### 与 project-lesson 的边界

- **本技能的 `write`**：轻量写入，适用于用户明确知道要写什么、写到哪的场景
- **`/spec-workflow-project-lesson`**：完整的经验捕获流程，含路由判断（项目特有 vs 全局通用）、优先级评估、向用户确认。需要完整踩坑路由判断时，委托给 project-lesson

---

### `daily` — 每日总结

生成当天的每日总结文件到 `wiki/daily/YYYY-MM-DD.md`。

#### 执行步骤

1. **确定日期**：当前日期，格式 `YYYY-MM-DD`
2. **扫描今日 git 提交**：
   - `git log --since="YYYY-MM-DD 00:00:00" --until="YYYY-MM-DD 23:59:59" --oneline`
   - 提取提交信息、涉及的项目/模块
3. **扫描今日 handoff 文件**：
   - 搜索 `tmp/*-handoff.md` 中今天创建/修改的文件
   - 提取关键状态和决策
4. **扫描今日经验记录**：
   - 搜索 `.spec-workflow/lessons/` 中今天创建的文件
   - 搜索 `wiki/pitfalls/` 中今天创建的文件
5. **汇总生成**：

```markdown
---
type: daily
date: YYYY-MM-DD
tags: [daily]
---

# YYYY-MM-DD 每日总结

## 今日工作

### <项目名>

- <提交/变更描述>

## 关键决策

- <决策内容>

## 经验记录

- <踩坑/学习摘要> — [[wiki/pitfalls/P1/xxx]]

## 学习收获

- <新学到的内容>

## 明日计划

- <待办事项>

## 关联文档

- [[spec 路径]]
- [[change 路径]]
- [[handoff 路径]]
```

6. **写入文件**：若已存在今日文件则追加，不存在则创建
7. **更新索引**：更新 `wiki/learning/index.md` 或相关索引

---

## 被其他技能调用的场景

### spec 技能调用

```
/spec-workflow-spec 在撰写新模块 spec 前：
→ 调用 check <module-topic>
→ 获取相关踩坑、设计模式、技术决策
→ 在 spec §4 非功能需求和 §5 边界中参考已有知识
```

### change 技能调用

```
/spec-workflow-change 在发起变更前：
→ 调用 check <change-topic>
→ 获取相关经验教训
→ 在 change.md 的技术方案中规避已知问题
```

### implement 技能调用

```
/spec-workflow-implement 在写代码前：
→ 调用 check <implementation-topic>
→ 获取相关技术踩坑
→ 在实现中应用预防措施
```

---

## 路径解析规则

### 工作区根定位

从当前目录向上查找包含 `wiki/` 目录的祖先，即为工作区根。

### 项目根定位

从当前目录向上查找包含 `.eo-project.json` 的目录。找不到时仅跳过项目级搜索，不报错。

### 模块定位

从 `.eo-project.json` 的 `doc_root` 字段解析 `eo-doc/` 路径，结合当前上下文确定模块名。

---

## 搜索策略详细说明

### 优先级策略

1. **LLMWiki CLI 优先**：若 `wiki` 命令可用，使用 CLI 搜索（内部已优化索引匹配）
2. **INDEX 文件次之**：读取各目录的 `INDEX.md` / `index.md`，匹配标题和摘要
3. **Frontmatter 匹配最后**：遍历目录文件，读取 frontmatter 的 `tags` / `title` 字段

### 关键词扩展

对用户输入的 `<keyword>` / `<topic>` 进行语义扩展：

- 技术名扩展：`JWT` → 同时搜索 `token` / `auth` / `认证`
- 框架名扩展：`React` → 同时搜索 `hooks` / `component` / `jsx`
- 功能名扩展：`登录` → 同时搜索 `auth` / `login` / `session`

扩展后的关键词逐一搜索，去重合并结果。

### 相关度评估

| 级别 | 标准 |
|------|------|
| **高** | 标题完全匹配、或 tags 中包含精确关键词 |
| **中** | 摘要中包含关键词、或 tags 中包含扩展关键词 |
| **低** | 内容中包含关键词但不在标题/tags/摘要中 |

---

## 输出

| 命令 | 产出 |
|------|------|
| `check` | 结构化知识摘要（内联输出，非文件） |
| `search` | 匹配文件列表（路径 + 标题 + 摘要） |
| `read` | 文档完整内容 |
| `write` | 新文件 + 索引更新确认 |
| `daily` | `wiki/daily/YYYY-MM-DD.md` 文件 |

## 约束

- **不全文 grep**：搜索入口是 INDEX 文件和 frontmatter，不是全文扫描
- **不覆盖已有文件**：write 命令遇到已存在文件时报错提示，不静默覆盖（daily 命令除外，采用追加模式）
- **不替代 project-lesson**：需要完整踩坑路由判断时，委托给 `/spec-workflow-project-lesson`
- **不依赖 .eo-project.json**（check / search / read / daily）：无项目配置时跳过项目级搜索，不报错
- **中文优先**：本技能所有输出和写入内容默认使用中文
- **内部引用统一前缀**：所有对其他 spec-workflow 技能的引用使用 `spec-workflow-` 前缀（如 `spec-workflow-change`、`spec-workflow-implement`）
- **索引同步**：写入后必须更新对应的 INDEX.md，保持索引与文件一致
- **lazy 创建目录**：目标目录不存在时在首次写入时创建，不预建
