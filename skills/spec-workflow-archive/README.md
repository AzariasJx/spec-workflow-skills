# spec-workflow-archive

**变更归档。** 把审查通过的 change 的 Spec Delta 机械合并回模块 spec.md，是 change 生命周期的最后一步。

## 什么时候用

- 代码审查通过后
- companion change（被依赖声明）创建后直接归档

## 命令

```
/spec-workflow-archive <module-name> <change-id>
```

## 三种归档模式

| 模式 | 触发条件 | 行为 |
|------|---------|------|
| **Delta 合并** | feature / enhance / refactor | ADDED/MODIFIED/REMOVED 落到 spec |
| **bootstrap** | change_type: bootstrap | 跳过 Delta 合并，只更新元信息 |
| **companion** | companion: true | Delta 合并 + 跳过 implement/test/review 校验 |

## 产出

- spec.md 更新（Delta 合并 + 元信息）
- change.md status → `archived`
- INDEX.md 状态更新

## 核心原则

- **机械合并**：不重新理解业务，只按 Delta 声明合并
- **冲突必须停下**：Delta 与现有 spec 冲突时让用户裁决
- **归档不可逆**

## 前置条件

- change.md status: done（companion 放宽为 approved）
- review.md 通过（companion 豁免）
- test.md 通过（companion 豁免）

## 相关 skill

- 前置：`/spec-workflow-review`
- 归档后：spec.md 已更新，可继续 `/spec-workflow-change`
