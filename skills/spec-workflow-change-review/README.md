# spec-workflow-change-review

**审查变更方案。** 在 implement 之前检查 change.md 的 Delta 合规性、TODO 完整性、AC 覆盖度。

## 什么时候用

- 大规模/高风险 change 写完后、写代码之前
- change-review 返工后的复审

## 命令

```
/spec-workflow-change-review <change-path>
```

## 产出

- `eo-doc/dev/<module>/changes/<change-id>/change-review.md`

## 审查维度

1. §3 合规性（Delta 是否指向正确章节、是否真的是增量）
2. §3 ↔ TODO 一致性（方案是否完整覆盖 Delta）
3. TODO 拆解完整性
4. AC 覆盖度
5. 范围与风险

## 前置条件

- change.md 存在且 status 为 `draft` 或 `approved`

## 注意

- **这是方案审查，不审代码**。代码审查用 `/spec-workflow-review`。
- 可选环节：小型 change 可跳过直接 implement。

## 相关 skill

- 区别：本 skill 审 change 方案，`/spec-workflow-spec-review` 审 spec，`/spec-workflow-review` 审代码
