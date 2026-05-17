# spec-workflow-test

**测试与验证。** 根据 change 的验收标准编写和执行测试，产出结构化测试报告。

## 什么时候用

- 代码实现完成后
- review 修复后重新测试

## 命令

```
/spec-workflow-test <change-path>
```

## 产出

- `eo-doc/dev/<module>/changes/<change-id>/test.md`
- 测试代码文件（在项目测试目录下）

## 核心原则

- **验证驱动**：证明代码是否符合预期
- **禁止修改业务源码**：发现 bug 只记录，不自行修复
- 测试基于 change.md 的验收标准

## 阶段

1. **单元测试**：覆盖正常路径、边界条件、异常场景
2. **集成/场景验证**：端到端场景测试
3. **产出报告**：每个测试的执行证据

## 前置条件

- change 相关代码已实现
- change status 为 `implementing` 或 `done`

## 相关 skill

- 前置：`/spec-workflow-implement`
- 后续：`/spec-workflow-review`
- 修复：`/spec-workflow-implement`（fix 模式）
