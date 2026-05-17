# spec-workflow-module-init

**模块初始化。** 为新模块建立 spec 活文档并通过 spec-review 审查。一个模块只跑一次。

## 什么时候用

- 需要新建一个业务模块（如 `user-service`、`order-service`）
- 模块目录在 `eo-doc/dev/` 下不存在时

## 命令

```
/spec-workflow-module-init <module-name>
```

## 产出

| 文件 | 说明 |
|------|------|
| `eo-doc/dev/<module>/spec.md` | 模块能力说明书（活文档） |
| `eo-doc/dev/<module>/spec-review.md` | spec 审查报告 |
| `eo-doc/dev/<module>/changes/INDEX.md` | 变更时间线索引 |

## 前置条件

- 已运行 `/spec-workflow-project-init`
- 模块名使用小写 kebab-case（如 `user-service`）

## 流程

1. 调用 `/spec-workflow-spec` 撰写 spec（与用户反复澄清）
2. 用户确认 → `status: confirmed`
3. 自动触发 `/spec-workflow-spec-review` 审查
4. 通过后建立 `changes/` 骨架

## 相关 skill

- 被调用：`/spec-workflow-spec`、`/spec-workflow-spec-review`
- 后续：`/spec-workflow-change`（发 bootstrap change 落地为代码）
