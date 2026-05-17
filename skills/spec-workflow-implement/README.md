# spec-workflow-implement

**代码实现。** 按 change.md 的 TODO 逐项写代码，或根据 test/review 反馈修复缺陷。

## 什么时候用

- change 批准后开始写代码
- 测试/审查发现问题需要修复（fix 循环）
- bug 修复（不开新 change，在当前 change 内循环）

## 命令

```
/spec-workflow-implement <change-path>
```

## 两种模式

| 模式 | 触发条件 | 行为 |
|------|---------|------|
| **首次实施** | change status: approved | 按 TODO 逐项编码 |
| **fix 循环** | test.md 有 FAIL / review.md 有 P0/P1 | 按优先级修复，不改 change §3 |

## 核心原则

- 严格按 change TODO 实现，不擅自扩展
- 完成每个 TODO 立即在 change.md 中勾选 `- [x]`
- 遇到 change 未覆盖的技术细节先问用户
- bug 修复不升格为新 change

## 前置条件

- change.md 存在且 status 为 `approved` 或 `implementing`

## 相关 skill

- 前置：`/spec-workflow-change`
- 后续：`/spec-workflow-test` → `/spec-workflow-review`
