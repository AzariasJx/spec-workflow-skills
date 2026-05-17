# spec-workflow-fix

**Bug 诊断路由。** 发现 bug 但不确定是代码错了、change 方案偏了、还是 spec 基线本身错了。先诊断再分叉。

## 什么时候用

- 发现 bug 但不确定根因在哪一层
- **不用于**：已定位的代码 bug（直接走 `/spec-workflow-implement` fix 模式）
- **不用于**：明确的业务变更（走 `/spec-workflow-change` enhance）

## 命令

```
/spec-workflow-fix <问题描述>
```

## 诊断流程

1. **搜集现象**：期望行为 vs 实际行为
2. **轻量定位**：靠 INDEX + frontmatter 找到相关 spec/change/state
3. **三方对比**：凑齐 F-spec / F-change / F-code 三份事实
4. **诊断分叉**：

| 根因 | 建议路径 |
|------|---------|
| 代码没按 change 实现 | `/spec-workflow-implement` fix |
| change 方案写偏了 | 内联改 change → `/spec-workflow-change-review` |
| spec 基线错了（业务变更） | `/spec-workflow-change` enhance |

## 产出

- 诊断报告（内联输出，非文件）

## 核心原则

- **不动手改**：只出诊断 + 建议，由用户拍板后派给对应 skill
- **先索引后全文**：用 INDEX + frontmatter 收敛，不全局 grep

## 相关 skill

- 分叉到：`/spec-workflow-implement`、`/spec-workflow-change`、`/spec-workflow-change-review`
