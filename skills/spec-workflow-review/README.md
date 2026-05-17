# spec-workflow-review

**代码审查。** 对已实施的代码进行结构化审查，对照 spec 和 change 检查功能完整性、逻辑正确性、架构合规性。

## 什么时候用

- 代码实现完成 + 测试通过后
- **不用于**：审查 change 方案（走 `/spec-workflow-change-review`）

## 命令

```
/spec-workflow-review <change-path>
```

## 产出

- `eo-doc/dev/<module>/changes/<change-id>/review.md` — P0/P1/P2 分级审查报告

## 审查维度

1. **功能完整性**：TODO 和 AC 是否全部满足
2. **逻辑正确性**：核心业务逻辑、异常处理、边界条件
3. **架构合规性**：分层、模块边界、依赖方向
4. **代码规范**：命名、类型、重复代码
5. **安全与性能**：注入、越权、性能瓶颈

## 前置条件

- 代码已实现（change status 为 `implementing` 或 `done`）
- 不接受 status: draft 或 approved（代码还没写）

## 相关 skill

- 前置：`/spec-workflow-implement`、`/spec-workflow-test`
- 后续：`/spec-workflow-archive`
- 区别：本 skill 审代码，`/spec-workflow-change-review` 审方案，`/spec-workflow-spec-review` 审 spec
