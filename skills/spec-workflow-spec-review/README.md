# spec-workflow-spec-review

**审查模块 spec 基线。** 检查 spec 的业务自洽性、需求完整性、AC 可测性。module-init 时自动跑一次。

## 什么时候用

- module-init 时自动触发
- 手动检查已有 spec 的质量
- 归档多个 MODIFIED/REMOVED Delta 后验证新基线

## 命令

```
/spec-workflow-spec-review <module-name>
```

## 产出

- `eo-doc/dev/<module>/spec-review.md` — 审查报告，含 P0/P1/P2 问题

## 审查维度

1. 业务自洽性（需求之间有无矛盾）
2. 需求完整性（边界、异常路径）
3. AC 可测性（验收标准是否可验证）
4. 结构合规性（章节完整性）

## 前置条件

- `eo-doc/dev/<module>/spec.md` 存在

## 相关 skill

- 被调用：`/spec-workflow-module-init`（步骤四）
- 区别：本 skill 审 spec，`/spec-workflow-change-review` 审 change，`/spec-workflow-review` 审代码
