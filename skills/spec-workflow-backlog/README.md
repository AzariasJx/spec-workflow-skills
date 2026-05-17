# spec-workflow-backlog

**管理待办池。** 往项目的 `backlog.md` 追加待办、灵感或未接入的未来规划。

## 什么时候用

- 对话中出现"以后要做"、"先跳过"、"TODO"、"回头处理"等信号
- 想记录一个灵感或想法
- 暂时不做但不想忘的事

## 命令

```
/spec-workflow-backlog <内容>
```

## 自动分类

| 信号词 | 分类 |
|--------|------|
| TODO、待办、回头、先跳过 | **待办** |
| 以后、想法、先记一笔、idea | **灵感 & 以后再说** |
| 等 skill、未接入、暂时 | **未接入** |

## 产出

- `.spec-workflow/backlog.md` 追加一条条目

## 条目格式

```markdown
- [ ] 简述 — YYYY-MM-DD    # 待办
- 简述 — YYYY-MM-DD         # 灵感/未接入
```

## 前置条件

- 已运行 `/spec-workflow-project-init`

## 约束

- 只追加，不修改已有条目
- 不越界到 decisions/ 或 lessons/

## 相关 skill

- 决策记录：`/spec-workflow-project-update`
- 经验教训：`/spec-workflow-project-lesson`
