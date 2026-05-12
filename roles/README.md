# Roles — 角色骨架

这里是 7 个角色的**泛化 prompt**。已去除项目专属内容（TS/Node 命令、medeo-market 目录、业务规范锚点），保留角色性格、职责、硬边界、协议契约。

## 篇目

_(以下文件将在后续提交中补齐——从 medeo-market 实战版本拔高改写)_

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
2. 在你项目根建 `integration.md`，填入该项目的具体约定（见 [`../integration/claude-code/`](../integration/claude-code/) 示例）
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
