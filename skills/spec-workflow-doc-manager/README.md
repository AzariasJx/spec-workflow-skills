# spec-workflow-doc-manager

**代码侧文档管理。** 管理 `eo-doc/` 下的文档体系：初始化、增量同步、全量重建、查询。

## 什么时候用

- 初始化项目的文档骨架
- 代码变更后同步更新文档
- 查询某方面的文档

## 命令

```
/spec-workflow-doc-manager <command>
```

## 命令路由

| 命令 | 用途 |
|------|------|
| `init` | 创建 eo-doc/ 最小骨架 |
| `sync` | 基于 git diff 增量同步文档 |
| `re-sync` | 全量扫描源码重建文档 |
| `modify` | 修改/创建文档 |
| `query` | 按关键词查文档 |

## 管理的目录

```
eo-doc/
├── agent-handbook/   # 给 AI 看的代码地图
├── dev/              # spec/change/review（由其他 skill 维护）
├── templates/        # spec-workflow 扩展点
└── state/            # 给人看的系统现状描述
```

## 前置条件

- `init` 之外的命令需要 `.eo-project.json`

## 相关 skill

- 被调用：`/spec-workflow-project-init`（init 子流程）
- `dev/` 目录由 spec-workflow-change 等开发 skill 维护
