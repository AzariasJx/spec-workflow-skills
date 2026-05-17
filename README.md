# Spec-Workflow Skills

一套用于 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 的 **文档驱动开发技能包**。让 AI 编码助手像高级工程师一样工作：先写需求文档、再规划方案、最后才写代码——全程有据可查、可回溯。

## 这是什么？

你有没有遇到过这些情况：

- 让 AI 写了个功能，过两天改需求时它完全不记得之前怎么设计的
- 多个模块互相调用，改了一个模块却不知道会不会影响其他模块
- AI 直接上手写代码，写到一半发现方向错了，前面全白做

**spec-workflow** 就是用来解决这些问题的。它给 AI 加了一套"工程纪律"：

1. **先写文档，再写代码** — 每个模块有一份 spec（能力说明书），每次变更有一份 change（变更方案）
2. **文档永远是最新的** — 变更完成后自动把改动合并回 spec，不会过时
3. **跨模块关系可追踪** — A 模块用了 B 模块的接口，B 的文档里会自动记录"谁在用我"

## 快速开始

### 安装

**前提**：已安装 [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)。

将你需要的 skill 目录复制（或创建 symlink）到 Claude Code 的 skills 目录：

```bash
# skills 目录位置
# macOS/Linux: ~/.claude/skills/
# Windows:     C:\Users\<你的用户名>\.claude\skills\

# 方式一：直接复制
cp -r spec-workflow-* ~/.claude/skills/

# 方式二：创建 symlink（推荐，方便更新）
# Linux/macOS
ln -s /path/to/spec-workflow-skills/spec-workflow-* ~/.claude/skills/

# Windows (PowerShell，管理员权限)
# 假设技能包在 E:\work-space\spec-workflow-skills\
Get-ChildItem "E:\work-space\spec-workflow-skills\spec-workflow-*" | ForEach-Object {
    New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\skills\$($_.Name)" -Target $_.FullName
}
```

安装后在 Claude Code 对话中直接输入斜杠命令即可使用。

### 第一个项目

```
你: /spec-workflow-project-init 我的项目名称是 online-shop，一个在线商城，技术栈是 Spring Boot + Vue
```

这一条命令会自动创建项目骨架：

```
你的项目/
├── .eo-project.json          # 项目配置（所有 skill 都读它）
├── .spec-workflow/            # 项目管理侧（默认不提交到 git）
│   ├── roadmap.md             # 项目路线图
│   ├── log.md                 # 操作日志
│   └── backlog.md             # 待办池
├── eo-doc/                    # 代码侧文档（提交到 git）
│   ├── dev/INDEX.md           # 模块索引
│   ├── agent-handbook/INDEX.md
│   └── templates/             # 模板目录
└── CLAUDE.md                  # 自动注入 spec-workflow 提示词
```

### 创建第一个模块

```
你: /spec-workflow-module-init user-service

AI: （会和你对话澄清需求，然后产出 spec.md）
    用户管理模块的 spec 已就绪，请确认...
你: 看起来没问题

AI: 模块 user-service 初始化完成。
    接下来可以用 bootstrap 类型的 change 把 spec 落地为代码：
    /spec-workflow-change user-service
```

### 发起第一次变更

```
你: /spec-workflow-change user-service 我想给用户加一个头像上传功能

AI: （分析需求、匹配模块、和你澄清细节，然后产出 change.md）
    变更文档已就绪。
    接下来：
    1. 改 status 为 approved
    2. /spec-workflow-implement <change-path>  — 写代码
    3. /spec-workflow-review <change-path>     — 代码审查
    4. /spec-workflow-archive user-service 001  — 归档，自动更新 spec
```

## 核心流程

所有工作都围绕下面这个生命周期展开：

```
spec（模块能力说明书）
  │
  ├── /spec-workflow-module-init    ← 新建模块时走这一步
  │     写出第一版 spec.md
  │
  └── 之后每次变更都走这个循环：
        │
        ▼
      /spec-workflow-change          ← 写变更方案（不改代码）
        │  产出 change.md（Delta + TODO）
        │
        ▼
      /spec-workflow-implement       ← 按方案写代码
        │  勾选 TODO，写业务代码
        │
        ▼
      /spec-workflow-test            ← 测试
        │  产出 test.md
        │
        ▼
      /spec-workflow-review          ← 代码审查
        │  产出 review.md（P0/P1/P2）
        │
        ▼
      /spec-workflow-archive         ← 归档
           把 Delta 合并回 spec.md
           spec 永远是最新的 ✓
```

**关键理念**：spec 是活文档。你不手动改 spec，而是通过 change 声明"我要加什么/改什么/删什么"（叫 Delta），归档时自动合并。这样 spec 永远和代码保持一致。

## 全部技能一览

### 必用（日常开发核心）

| 命令 | 用途 | 什么时候用 |
|------|------|-----------|
| `/spec-workflow-project-init` | 初始化项目 | 新项目刚开始，只跑一次 |
| `/spec-workflow-module-init` | 初始化模块 | 新增一个业务模块，只跑一次 |
| `/spec-workflow-change` | 发起变更 | 要加功能/改功能/重构时 |
| `/spec-workflow-implement` | 写代码 | change 批准后，按 TODO 写代码 |
| `/spec-workflow-test` | 测试 | 代码写完后跑测试 |
| `/spec-workflow-review` | 代码审查 | 测试通过后审查代码质量 |
| `/spec-workflow-archive` | 归档 | 审查通过后，把变更合并回 spec |

### 按需使用

| 命令 | 用途 | 什么时候用 |
|------|------|-----------|
| `/spec-workflow-spec` | 写模块 spec | 被 module-init 自动调用，一般不直接用 |
| `/spec-workflow-spec-review` | 审查 spec | module-init 时自动跑；也可手动检查 spec 质量 |
| `/spec-workflow-change-review` | 审查变更方案 | 大变更写完方案后、写代码之前，先审查方案本身 |
| `/spec-workflow-fix` | 修 bug | 发现 bug 时，在现有 change 内修复（不建新 change） |
| `/spec-workflow-doc-manager` | 文档管理 | 管理 eo-doc/ 目录下的各类文档 |

### 项目管理

| 命令 | 用途 | 什么时候用 |
|------|------|-----------|
| `/spec-workflow-backlog` | 管理待办 | 往 backlog 里加/改待办事项 |
| `/spec-workflow-project-lesson` | 记录经验教训 | 踩坑了、学到了，记录下来 |
| `/spec-workflow-project-update` | 更新项目状态 | 更新 roadmap、阶段进度 |
| `/spec-workflow-wiki` | 知识库 | 全局经验、设计模式、常见坑 |
| `/spec-workflow-handoff` | 交接上下文 | 换人接手或换个对话继续 |

### 特殊场景

| 命令 | 用途 |
|------|------|
| `/spec-workflow-miniapp-ideation` | 小程序/小工具的快速创意到方案 |

## 跨模块依赖追踪

当一个功能涉及多个模块时（比如工单模块调用了用户模块的接口），spec-workflow 会自动建立双向追踪：

```
ruoyi-workflow（消费方）          ruoyi-system（提供方）
  changes/                         spec.md
    002-smart-ticketing/           ├─ §3.11 工单功能
      change.md                    └─ §8. 被依赖声明
        §2.3 依赖:                    │ ruoyi-workflow 使用了
        - ruoyi-system              │ RemoteDeptService
        - ruoyi-resource            │ RemoteUserService
                                    │
                  companion change ──┘
                  (001-companion-smart-ticketing)
```

**效果**：下次有人要改 `ruoyi-system` 时，打开 spec 一眼就能看到"ruoyi-workflow 在用我的 RemoteDeptService"，不会无感知地改坏下游。

## 变更类型

每次发起 change 时，AI 会自动判断类型：

| 类型 | 含义 | spec 变化 |
|------|------|----------|
| `bootstrap` | spec 写好了但代码还没写，第一次实现 | spec 不变，只写代码 |
| `feature` | 全新功能，spec 里没有的 | spec 新增章节 |
| `enhance` | 改进已有功能 | spec 修改章节 |
| `refactor` | 内部重构，对外能力不变 | spec 可能微调内部描述 |

注意：**修 bug 不是新 change**。bug 修复在当前 change 的 implement 循环内完成。

## 目录结构说明

一个使用 spec-workflow 的项目，文档分两层：

### 代码侧 `eo-doc/`（提交到 git，团队共享）

```
eo-doc/
├── dev/
│   ├── INDEX.md                          # 所有模块的索引
│   ├── user-service/
│   │   ├── spec.md                       # 模块能力说明书（活文档）
│   │   └── changes/
│   │       ├── INDEX.md                  # 变更时间线
│   │       └── 001-add-avatar/
│   │           ├── change.md             # 变更方案（Delta + TODO）
│   │           ├── review.md             # 代码审查报告
│   │           └── test.md               # 测试报告
│   └── order-service/
│       └── ...
└── templates/                             # 项目级模板
```

### 管理侧 `.spec-workflow/`（默认不提交，本地管理）

```
.spec-workflow/
├── roadmap.md          # 项目路线图和阶段规划
├── backlog.md          # 待办池
├── log.md              # 操作日志
├── phases/             # 阶段详情（按需创建）
├── decisions/          # 技术决策记录（按需创建）
└── lessons/            # 经验教训（按需创建）
```

## 文件状态流转

### change.md 的状态

```
draft → approved → implementing → done → archived
  │                                          │
  │  写方案、澄清需求                         │  Delta 合并回 spec
  │  /spec-workflow-change                   │  /spec-workflow-archive
  │                                          │
  └── /spec-workflow-change-review ──────────┘
      （可选，大规模变更建议跑）

implementing ←─── fix 循环 ───→ implementing
  │                                  │
  /spec-workflow-implement           /spec-workflow-test
                                → /spec-workflow-review
                                → /spec-workflow-implement（修 bug）
```

### spec.md 的状态

```
draft → confirmed
  │
  module-init 时写完，用户确认后变为 confirmed
  之后不再手动改——所有变更通过 archive 自动合并
```

## 常见问题

### Q: 我的项目已经写了很多代码了，还能用吗？

可以。用 `/spec-workflow-project-init` 初始化项目，然后对已有模块用 `/spec-workflow-module-init` 补写 spec（spec 记录的是模块"当前"的能力）。之后再做任何改动就走正常的 change 流程了。

### Q: 小改动也要走完整流程吗？

小改动（比如改个文案、加个字段）可以直接走简化路线：

```
/spec-workflow-change → 改 status 为 approved → /spec-workflow-implement → /spec-workflow-archive
```

跳过 change-review、test、review 都可以。流程是灵活的，只有 archive 是必须的最后一步。

### Q: bug 修复怎么处理？

在当前正在实施的 change 里直接修，不需要建新 change。如果 change 已经归档了，用 `/spec-workflow-implement` 直接修代码（它会自动进入 fix 模式）。

只有当 bug 的根因是"spec 本身写错了"时，才需要发一个新的 `enhance` 类型 change。

### Q: companion change 是什么？

当你的功能调用了其他模块的接口时，系统会在对方模块里自动创建一个"伴随变更"（companion change），把"谁在用我的哪个接口"记录到对方的 spec 里。这是纯文档操作，不涉及代码，归档后就完成了。

### Q: 它和直接让 AI 写代码有什么区别？

| 直接写代码 | 用 spec-workflow |
|-----------|-----------------|
| 需求在对话里，对话结束就没了 | 需求在 spec.md 里，永久保存 |
| 改了什么只能看 git log | 每次 change 有完整的方案和审查记录 |
| 模块间关系靠人记 | spec 里自动记录"谁依赖谁" |
| AI 每次从零理解项目 | AI 读 spec 就能理解全貌 |

## 技术要求

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Git 仓库（推荐）
- 无额外依赖，纯 Markdown 文档驱动

## License

MIT
