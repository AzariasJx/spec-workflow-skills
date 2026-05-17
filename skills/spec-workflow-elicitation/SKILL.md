---
name: spec-workflow-elicitation
description: "对模糊想法做需求挖掘（发散-收敛对话），产出 requirement-brief.md 供后续 /spec-workflow-change --from 消费。触发：有个想法 / 我想做 X / 需求不清晰 / 先聊聊 / 挖掘需求 / elicitation / /spec-workflow-elicitation。NOT FOR: 已明确要改哪个模块的变更（直接走 /spec-workflow-change）；bug 修复（走 /spec-workflow-fix）；新模块初始化（走 /spec-workflow-module-init）。"
---

# spec-workflow-elicitation — 需求挖掘

用户带着模糊想法来，先不做文档，先聊透。通过发散-收敛对话把模糊意图结构化为 requirement-brief.md，供后续 `/spec-workflow-change <module> --from <proposal-id>` 消费。

## 核心理念

1. **先发散再收敛**：不急着结构化——先让用户把脑子里所有东西倒出来，再由 AI 提炼维度、补问空白
2. **不做 change**：本 skill 不产出 change.md，不写 Delta，不碰 spec。产出是轻量级 requirement-brief
3. **暴露不确定性**：宁可多问三轮把边界说清楚，也不要揣测后塞进文档。没想清楚的地方标 `[待定]`
4. **持久化但不锁死**：requirement-brief.md 是持久工件，但 status 保持 `eliciting` 直到收敛完成。用户随时可以回来改
5. **路由优先于产出**：如果对话中发现这不是新需求（是 bug / 已有模块 / 不需要 elicitation），立即引导用户走正确的 skill，不硬产出

## 前置条件

- **必须能找到 `.eo-project.json`**。找不到 → 报错退出，提示运行 `/spec-workflow-project-init`。`eo-doc/` 路径通过 `doc_root` 字段解析
- 用户有一个模糊想法（不要求知道具体模块名）

## 工作流程

### 第零步：快速路由判断

在开始 elicitation 对话之前，做一次 30 秒路由检查：

| 信号 | 路由 |
|------|------|
| 用户说"修个 bug" / "这里不对" / "应该是 X 但实际是 Y" | → `/spec-workflow-fix` |
| 用户说"我要一个全新的 XXX 模块"且该模块不存在 | → `/spec-workflow-module-init` |
| 用户说"我要改 YYY 的 ZZZ 功能"且模块已存在、意图清晰 | → `/spec-workflow-change` |
| 用户说"有个想法" / "我想做 X 但还没想清楚" / "先聊聊" | → 继续本 skill（第一步） |

**路由不硬判断**：如果信号模糊，问一句话确认，不要替用户决定。

### 第一步：发散阶段 — 让用户自由表达

**目标**：让用户把脑子里所有相关想法倒出来，不加约束。

1. 向用户说一段引导语（不是冷冰冰的"请描述需求"）：

   > 跟我说说你想做什么？随便聊，不需要组织语言。想到什么说什么——痛点、场景、灵感、犹豫的点都可以。

2. 用户自由表达（可能是口语化的、跳跃的、不完整的）
3. AI 只做**积极倾听**：用简短回应确认理解，不打断，不纠正，不提前结构化
4. 如果用户一次说得不够（<3 句话），追问一两个开放式问题：
   - "你自己在什么场景下会用到这个？"
   - "现在没有这个功能的时候，你怎么做的？"
   - "有没有什么你特别担心或犹豫的点？"
5. **发散不超过 3 轮**：3 轮自由对话后进入结构化，避免无限发散

**AI 绝不做的事**：不列举功能清单、不画架构图、不提技术方案、不说"这很简单/很难"。

退出条件（满足任一即进入第二步）：
- 用户已表达超过 5 个不同的点/想法
- 用户自然停顿并问"你觉得呢"
- 已完成 3 轮对话

### 第二步：结构化 — AI 提炼维度

**目标**：从发散阶段的原始输入中提炼出需求维度，找出已知和未知。

1. 根据发散阶段的对话，提炼出以下维度（不是所有维度都有内容，没内容的标 `_未涉及_`）：

   | 维度 | 说明 | 典型问题 |
   |------|------|---------|
   | **D1 业务背景** | 为什么想做这个？解决什么问题？ | 业务动机、触发场景 |
   | **D2 目标用户** | 给谁用的？ | 角色、画像、使用频率 |
   | **D3 核心场景** | 用户在什么情况下做什么事？ | 完整的使用故事 |
   | **D4 期望行为** | 做完之后系统应该是什么样？ | 功能期望、交互期望 |
   | **D5 涉及范围** | 大概涉及哪些模块/功能点？ | 模块猜测、边界感知 |
   | **D6 约束与偏好** | 有什么限制？什么不能做？ | 技术偏好、时间约束、合规要求 |
   | **D7 非目标** | 明确不做什么 | 排除项、v2 再考虑的 |
   | **D8 成功标准** | 怎么算做成了？ | 验收直觉、关键指标 |

2. 将提炼结果呈现给用户，格式如下：

```
我听到你想做的是：{一句话总结}

我把你说的梳理成几个维度，你看看准不准：

**✅ 已经清楚的**：
- D1 业务背景：{摘要}
- D3 核心场景：{摘要}
...

**❓ 还需要聊清楚的**：
- D2 目标用户：你提到"用户"，具体是内部员工还是外部客户？
- D4 期望行为：你说"自动处理"，是完全自动还是需要人工确认？
...

**🤷 没提到的**：
- D6 约束与偏好：_未涉及_
- D8 成功标准：_未涉及_
```

**分类标准**：如果一个维度有足够信息写出一句话摘要，算"已清楚"。如果只能猜，算"需要追问"。

### 第三步：收敛阶段 — 靶向问答填补空白

**目标**：针对第二步中标 `❓` 和 `🤷` 的维度，向用户提出精准问题。

1. **只问空白维度**：已经清楚的不重复问
2. **每次最多 3 个问题**：不要一口气列出 10 个问题，分轮次问
3. **问题必须具体**：不给选择题列表让用户填表，而是用对话方式问：

   **好的问法**（具体、场景化）：
   - "你说的'通知'是发邮件还是短信还是都要？"
   - "现在没有这个功能时，用户是怎么解决这个问题的？"
   - "你说'尽快'，大概期望多长时间内？"

   **差的问法**（太泛、术语化、像填表）：
   - "请描述完整的业务流程"
   - "非功能需求有什么要求？"
   - "请列出所有约束条件"

4. **允许用户说"不知道"**：用户说不知道的维度标 `[待定]`，不强求填满
5. **最多 3 轮问答**：3 轮后还没收敛的维度标 `[待定]`，在 brief 里标记为待决策项
6. 收敛完成的标志：
   - 所有维度至少有内容（包括 `[待定]`）
   - 用户说"差不多" / "就这样" / "可以了"
   - 或者已达 3 轮上限

### 第三步点五：模块归属建议

**目标**：在收敛完成后，帮用户确定需求归属哪个模块。

1. 扫描 `eo-doc/dev/` 下所有模块的 `spec.md` frontmatter（title / module_name / tags / summary）
2. 根据需求内容语义匹配 1-3 个候选模块
3. 向用户建议：

   > 这个需求看起来和以下模块相关：
   > - `ruoyi-workflow` — 工作流引擎，你说的审批流程可能归属这里
   > - `ruoyi-system` — 如果主要是权限/角色相关的
   >
   > 你觉得放哪个模块合适？如果都不合适，可能需要先 `/spec-workflow-module-init` 建一个新模块。

4. 如果用户无法确定模块：
   - 在 brief 里标 `module: _待定_`
   - 后续 `/spec-workflow-change --from <proposal-id>` 启动时会重新判定模块
5. 如果需求涉及多个模块：在 brief 里列出所有候选，后续 change 阶段按跨模块规则拆分

### 第四步：分配 proposal-id

1. 用户指定或由 AI 建议一个目标模块（允许 `_待定_`）
2. 若模块已确定：`cd eo-doc/dev/<module>/proposals/` 扫描现有子目录（不存在则创建）
3. 取最大数字前缀 + 1，3 位补零（如 `001` / `002`）
4. 用户给语义名（小写 kebab-case），拼接为 `<NNN>-<kebab-id>`
5. 若模块待定：暂时放在 `eo-doc/proposals-pending/<NNN>-<kebab-id>/`，待模块确定后移动
6. 示例：`001-employee-feedback`、`002-smart-notification`

### 第五步：撰写 requirement-brief.md

创建目录并按下方"固定模板"写入 `requirement-brief.md`。

### 第六步：用户确认

交付用户确认。根据反馈修订到用户满意，然后改 `status: elicited`。

### 第七步：更新索引

1. 若模块已确定：更新 `eo-doc/dev/<module>/proposals/INDEX.md`（不存在则创建）
2. 若模块待定：更新 `eo-doc/proposals-pending/INDEX.md`（不存在则创建）

### 第八步：提示后续流程

根据 brief 的完整度给出不同后续建议：

**Brief 完整度高**（无 `[待定]` 或仅 1 个）：

> 需求梳理完成。下一步：
> 1. `/spec-workflow-change <module> --from <proposal-id>` — 基于 requirement-brief 发起变更
> 2. 如果还没确定模块 → 先 `/spec-workflow-module-init <module-name>` 建模块

**Brief 有较多待定项**（2+ 个 `[待定]`）：

> 需求已初步梳理，还有 N 个待定项。你可以：
> 1. 回来继续聊，把待定项补上（直接告诉我"继续聊 <proposal-id>"）
> 2. 带 `[待定]` 先进 change，在 change 的澄清阶段解决
> 3. 先把想法记到 backlog：`/spec-workflow-backlog`

**Brief 揭示这其实不是新需求**：

> 聊下来发现这可能是：
> - **一个 bug** → `/spec-workflow-fix` 做诊断路由
> - **一个已有模块的小调整** → 直接 `/spec-workflow-change <module>`，不需要 elicitation
> - **一个全新的独立模块** → `/spec-workflow-module-init <module-name>` 先建模块

---

## 固定模板 — requirement-brief.md

```markdown
---
name: <kebab-id>
module: <module-name> | _待定_
status: eliciting | elicited | consumed | aborted
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [tag1, tag2]
summary: >
  一句话概括需求意图（从发散阶段提炼的核心命题）。
related_change: null | <module>/changes/<NNN-xxx>
---

# <需求简述标题>

> 本文档由 `/spec-workflow-elicitation` 产出，供 `/spec-workflow-change <module> --from <proposal-id>` 消费。
> 不替代 change.md，是 change 的前置输入。

## D1 业务背景

{为什么想做这个？解决什么问题？触发场景是什么？}

## D2 目标用户

{给谁用的？角色、画像、使用频率。}

## D3 核心场景

{用户在什么情况下做什么事？用故事方式描述，1-3 个场景。}

### 场景 1：{场景名}

{谁、在什么情况下、想做什么、期望结果}

## D4 期望行为

{做完之后系统应该是什么样？功能期望、交互期望。}

## D5 涉及范围

### 候选模块

- `<module-a>` — {为什么相关}
- `<module-b>` — {为什么相关}

### 已有能力（初步判断）

{读 spec frontmatter 后，初步判断哪些模块已有相关能力，哪些没有。}

## D6 约束与偏好

{有什么限制？技术偏好？时间约束？合规要求？}

## D7 非目标（明确不做）

{排除项、v2 再考虑的。}

## D8 成功标准

{怎么算做成了？验收直觉、关键指标。}

## 待决策项

| 编号 | 维度 | 问题 | 状态 |
|------|------|------|------|
| Q1 | D2 | {具体问题} | [待定] |
| Q2 | D6 | {具体问题} | [待定] |

## 对话摘要

{发散-收敛过程中用户的关键原话或决策，2-5 条，供后续 change 阶段参考。}

- "{用户原话 1}"
- "{用户原话 2}"
```

---

## 固定模板 — proposals/INDEX.md

**模块内**（位于 `eo-doc/dev/<module>/proposals/INDEX.md`）：

```markdown
# <module-name> 需求提案时间线

| 编号 | 标题 | 状态 | 日期 | 摘要 | 关联 Change |
|------|------|------|------|------|-------------|
| [001-xxx](001-xxx/requirement-brief.md) | ... | elicited | YYYY-MM-DD | ... | — |
```

**待定**（位于 `eo-doc/proposals-pending/INDEX.md`）：

```markdown
# 待定模块的需求提案

| 编号 | 标题 | 状态 | 日期 | 摘要 | 模块归属 |
|------|------|------|------|------|---------|
| [001-xxx](001-xxx/requirement-brief.md) | ... | elicited | YYYY-MM-DD | ... | _待定_ |
```

---

## 与 spec-workflow-change 的衔接

当用户执行 `/spec-workflow-change <module> --from <proposal-id>` 时：

1. spec-workflow-change 读取 `eo-doc/dev/<module>/proposals/<proposal-id>/requirement-brief.md`
2. 将 brief 的各维度内容**预填充**到 change.md 的对应章节：
   - D1 业务背景 → §2.1 现状与问题
   - D2 + D3 + D4 → §2.2 变更目标
   - D5 → 辅助模块匹配（仍需执行第一步验证）
   - D7 → §5 Out of Scope
   - D8 → §5 验收标准初始参考
3. change 的第三步（澄清）只需处理 brief 中标 `[待定]` 的项和 change 特有的技术细节（Delta、TODO）
4. 对话摘要中的关键原话作为口径约束，不得矛盾
5. 若 brief 的 `module` 为 `_待定_`：按正常流程判定模块（第一步），并将确定的模块回写到 brief 的 frontmatter
6. 完成第四步（分配 change-id）后，回写 requirement-brief.md 的 frontmatter：
   - `related_change: <module>/changes/<NNN-xxx>`
   - `status: consumed`
7. 若 brief 在 `proposals-pending/`：将整个目录移动到确定模块的 `proposals/` 下，更新两边的 INDEX.md

---

## 状态流转

```
eliciting ──(收敛完成)──> elicited ──(被 change 消费)──> consumed
    │                        │
    │                        └──(用户回来补充)──> eliciting
    │
    └──(发现是 bug/路由错误)──> aborted
```

- `eliciting`：对话进行中，brief 正在撰写
- `elicited`：收敛完成，brief 可供 change 消费
- `consumed`：已被 change 引用，brief 进入只读历史状态
- `aborted`：对话中发现走错了 skill，brief 作废

---

## 续聊恢复

当用户说"继续聊 <proposal-id>"时：

1. 定位 requirement-brief.md（按模块内 → proposals-pending/ 顺序查找）
2. 读取 frontmatter 的 `status` 和 `待决策项` 章节
3. `status: eliciting` → 从待定问题处恢复收敛阶段
4. `status: elicited` → 问用户是想补充更多内容还是直接进 change
5. `status: consumed` → 告知已关联到 change，建议继续该 change 流程
6. `status: aborted` → 告知已作废，建议重新开始或走对应 skill

---

## 多想法分流

发散阶段如果用户同时描述了多个不相关主题：

1. 识别并告知用户："A 和 B 看起来是完全不同的需求，建议分开处理。"
2. 问用户想先聚焦哪一个
3. 为选定主题完成本 skill 流程
4. 对其余主题建议："想现在聊另一个还是先记到 backlog？`/spec-workflow-backlog`"

---

## 关键约束

| 约束 | 说明 |
|------|------|
| 不产出 change.md | 本 skill 的产出是 requirement-brief.md，不是 change.md |
| 不写 Delta / TODO | 这些是 change 阶段的产物，elicitation 只梳理需求 |
| 发散有上限 | 发散阶段不超过 3 轮，收敛阶段不超过 3 轮，总对话不超过 6 轮 |
| 路由优先 | 发现走错 skill 时立即引导，不硬产出 |
| brief 不替代 change | brief 是 change 的前置输入，不是 change 的替代品 |
| proposal-id 命名 | `NNN-kebab-name`，NNN 按模块内（或 proposals-pending/ 内）现有最大编号 +1，3 位补零 |
| 待定标记诚实 | 没想清楚就标 `[待定]`，不要 AI 编造内容 |
| 模块待定不阻塞 | 用户不知道放哪个模块也能完成 elicitation |
| 路由错误时标 aborted | 发现是 bug 或已有功能调整时，若 brief 已创建则标 `status: aborted` 并注明原因 |
