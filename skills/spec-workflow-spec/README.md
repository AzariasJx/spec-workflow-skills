# spec-workflow-spec

**撰写模块 spec 活文档。** 把模糊需求转化为清晰、无歧义的 spec.md。通常被 `/spec-workflow-module-init` 自动调用。

## 什么时候用

- 被 module-init 自动调用，一般不直接用
- 需要为全新模块写第一版 spec 时

## 命令

```
/spec-workflow-spec <module-name>
```

## 核心原则

- **不猜测**：模糊点必须向用户提问
- **不写代码**：只用自然语言描述需求行为
- **活文档**：spec 会随后续 change 的 Delta 持续演化

## 产出

- `eo-doc/dev/<module>/spec.md` — 包含 §1-§10 固定章节（概述、用户故事、功能需求、非功能需求、边界、验收标准等）

## 前置条件

- 已运行 `/spec-workflow-project-init`
- 仅用于新模块；已有模块的修改走 `/spec-workflow-change`

## 相关 skill

- 调用方：`/spec-workflow-module-init`
- 后续：`/spec-workflow-spec-review`、`/spec-workflow-change`
