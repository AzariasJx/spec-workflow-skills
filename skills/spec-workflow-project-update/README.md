# spec-workflow-project-update

**更新项目进度。** 勾选任务、切换阶段、记录技术决策，同步刷新项目日志。

## 什么时候用

- 任务完成了
- 阶段切换
- 做了一个技术决策（选型、架构取舍）
- 被卡住了

## 命令

```
/spec-workflow-project-update <更新内容>
```

## 支持的更新类型

| 类型 | 信号词 | 操作 |
|------|--------|------|
| 任务完成 | 完成、搞定、done | 勾选 phase 中的任务 |
| 阶段切换 | 阶段完成、进入下一阶段 | 切换 phase status |
| 决策记录 | 决定、选择、决策 | 创建 decisions/ 文件 |
| 阻塞标记 | 卡住、blocked | 记录到 log |

## 产出

- Phase 文件更新
- 决策文件（按需）
- `.spec-workflow/log.md` 追加日志

## 前置条件

- 已运行 `/spec-workflow-project-init`

## 相关 skill

- backlog 管理：`/spec-workflow-backlog`
- 经验记录：`/spec-workflow-project-lesson`
