# spec-workflow-project-lesson

**记录经验教训。** 捕获踩坑、最佳实践、意外收获，自动判断是项目特有还是全局通用。

## 什么时候用

- 踩坑了、学到了、发现了更好的做法
- 对话中出现"太慢了"、"踩坑"、"不应该"、"下次注意"等信号

## 命令

```
/spec-workflow-project-lesson <经验内容>
```

## 路由规则（踩坑类）

```
这是项目特有？还是全局通用？
  ├── 项目特有 → 仅写 .spec-workflow/lessons/
  └── 全局通用 → 写 .spec-workflow/lessons/ + 写 wiki/pitfalls/P{0-2}/
```

## 产出

| 文件 | 说明 |
|------|------|
| `.spec-workflow/lessons/YYYY-MM-DD-<slug>.md` | 项目级经验（必建） |
| `wiki/pitfalls/P{N}/<slug>.md` | 全局踩坑（仅全局通用时） |

## 经验类别

| 类别 | 说明 | 路由 |
|------|------|------|
| `pitfall` | 踩坑 | 自动判断项目/全局 |
| `best-practice` | 最佳实践 | 仅项目级 |
| `surprise` | 意外收获 | 仅项目级 |

## 前置条件

- 已运行 `/spec-workflow-project-init`

## 相关 skill

- 全局知识查询：`/spec-workflow-wiki`
