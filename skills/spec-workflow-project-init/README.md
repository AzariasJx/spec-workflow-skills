# spec-workflow-project-init

**项目初始化入口。** 一个项目只跑一次，生成 `.eo-project.json`、项目管理侧骨架（`.spec-workflow/`）和代码侧骨架（`eo-doc/`），以及 CLAUDE.md 注入。

## 什么时候用

- 新项目刚开始
- 已有项目首次接入 spec-workflow

## 命令

```
/spec-workflow-project-init <项目描述>
```

## 输入

- 项目名称 + 一句话目标
- 可选：PRD/MVP 文档路径、技术栈信息

## 产出

| 文件 | 说明 |
|------|------|
| `.eo-project.json` | 项目配置，所有其他 skill 都读它 |
| `.spec-workflow/roadmap.md` | 项目路线图 |
| `.spec-workflow/log.md` | 操作日志 |
| `.spec-workflow/backlog.md` | 待办池 |
| `eo-doc/` | 代码侧文档骨架 |
| CLAUDE.md 注入 | 自动添加 spec-workflow 提示词 |

## 前置条件

- 在 git 仓库根目录运行
- 只能运行一次（已有 `.eo-project.json` 时进入更新/修复模式）

## 相关 skill

- 后续：`/spec-workflow-module-init`、`/spec-workflow-change`
