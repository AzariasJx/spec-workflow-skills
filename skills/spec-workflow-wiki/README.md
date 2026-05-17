# spec-workflow-wiki

**知识库集成查询与写入。** 统一检索全局 wiki 知识库和本地项目经验，供其他 skill 在执行前做前置知识检查。

## 什么时候用

- 其他 skill 执行前的前置知识检查
- 搜索踩坑记录、设计模式、技术决策
- 写入轻量知识到 wiki

## 命令

```
/spec-workflow-wiki <command> <args>
```

## 命令

| 命令 | 用途 | 依赖 |
|------|------|------|
| `check <topic>` | 综合检查所有知识源 | 工作区根 |
| `search <keyword>` | 关键词搜索 | 工作区根 |
| `read <path>` | 读取指定文档 | 工作区根 |
| `write <location> <content>` | 写入 wiki 或项目经验 | 视位置而定 |
| `daily` | 生成每日总结 | 工作区根 |

## 搜索范围（按优先级）

1. `wiki/pitfalls/P0-P2/` — 全局踩坑
2. `wiki/learning/` — 学习笔记
3. `wiki/patterns/` — 设计模式
4. `.spec-workflow/lessons/` — 项目级经验

## 核心原则

- 不全文 grep：用 INDEX + frontmatter 匹配
- 可选集成 llmwiki-cli 提升搜索精度
- wiki/ 目录本身就是 Obsidian vault

## 被其他 skill 调用

- `/spec-workflow-spec` 写 spec 前 → `check <topic>`
- `/spec-workflow-change` 写 change 前 → `check <topic>`
- `/spec-workflow-implement` 写代码前 → `check <topic>`

## 相关 skill

- 经验捕获（完整路由）：`/spec-workflow-project-lesson`
