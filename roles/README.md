# Roles — 角色骨架

7 个角色的 prompt 骨架。语言和工具无关。

## 篇目

- `task-designer.md`
- `coder.md`
- `evaluator.md`
- `code-reviewer.md`
- `doc-refresher.md`
- `dreamer.md`
- `git-push.md`

## 使用

这些文件**不是开箱即用**。角色里引用的路径、命令、规范文档，都是你项目里自己定义的。

推荐做法：

1. 把文件拷贝到你项目的 `.claude/agents/` 下（或对应 harness 的 agent 目录）
2. 在你项目根建 `integration.md`，填入该项目的具体约定（见 [`../integration/claude-code/`](../integration/claude-code/)）
3. 按需增删角色——7 个是推荐起点，不是定数

## 目录约定（框架唯一的强约定）

角色之间靠共享的文件路径交接。推荐值：

```
docs/task/{task-id}/         # 任务文档根
  ├── progress.md            # 任务入口
  ├── plan/{step}.md         # 子任务契约（双契约都在这）
  └── memory/*.md            # 按天沉淀
docs/knowledge/              # 跨任务知识（dreamer 上浮目标）
docs/engineering/            # 项目规范（引用锚点）
```

路径可以改，但 7 个角色之间**必须用同一套**，否则交接会断。
