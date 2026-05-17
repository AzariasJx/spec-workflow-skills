# spec-workflow-change

**发起模块变更。** 产出 change.md（spec Delta + 技术方案 + TODO），是整个工作流的核心输入。

## 什么时候用

- 要加功能（feature）
- 要改已有功能（enhance）
- 要重构（refactor）
- 要把 spec 落地为代码（bootstrap）
- **不用于**：修 bug（走 `/spec-workflow-implement`）

## 命令

```
/spec-workflow-change <module-name> <变更描述>
```

## 产出

- `eo-doc/dev/<module>/changes/<NNN>-<name>/change.md`
- 若涉及其他模块：自动创建 companion change（被依赖声明）

## change 类型

| 类型 | 含义 |
|------|------|
| `bootstrap` | spec 已有但代码还没写 |
| `feature` | spec 里没有的全新能力 |
| `enhance` | 调整已有能力 |
| `refactor` | 内部重构，对外能力不变 |

## 跨模块规则

- 调用了其他模块的接口 → 自动为每个被依赖模块创建 companion change
- 改了多个模块的 spec → 拆成多个 change，用 `depends_on` 关联

## 前置条件

- 已运行 `/spec-workflow-project-init` 和 `/spec-workflow-module-init`
- 模块 spec 状态为 `confirmed`

## 相关 skill

- 后续：`/spec-workflow-change-review` → `/spec-workflow-implement`
