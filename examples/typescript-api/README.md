# Example — TypeScript API

展示一个 TypeScript + Node.js API 项目里，dev-roleplay 的实际填充形态。

## 项目设定

- 框架：TypeScript + Node.js + 某 HTTP 框架 + ORM（具体框架不重要，示例里用通用语法）
- 现有能力：Todo 管理（增删改查）
- 新需求：给 Todo 服务加一个 `GET /todos?tag={name}` 接口，按标签过滤

## 文件总览

```
typescript-api/
└── docs/
    ├── engineering/
    │   └── conventions.md        # 项目规范示例
    ├── knowledge/
    │   └── (空，本示例未展示 dreamer 上浮)
    └── task/
        └── 260601-todo-tag-filter/
            ├── progress.md
            ├── plan/
            │   └── step1-tag-filter-endpoint.md
            └── memory/
                └── 2026-06-01-tag-index-decision.md
```

## 阅读顺序

1. 先读 [`docs/engineering/conventions.md`](./docs/engineering/conventions.md) 了解项目规范基线
2. 再读 [`docs/task/260601-todo-tag-filter/progress.md`](./docs/task/260601-todo-tag-filter/progress.md) 看任务入口
3. 重点读 [`docs/task/260601-todo-tag-filter/plan/step1-tag-filter-endpoint.md`](./docs/task/260601-todo-tag-filter/plan/step1-tag-filter-endpoint.md)——**这是施工契约 + 验收契约双段的真实样本**
4. 最后读 [`docs/task/260601-todo-tag-filter/memory/2026-06-01-tag-index-decision.md`](./docs/task/260601-todo-tag-filter/memory/2026-06-01-tag-index-decision.md) 看一条按日沉淀的样例

## 这个示例展示了什么

- task-designer 如何生成 `progress.md` + `plan/{step}.md`
- 双契约的两段如何配对
- coder 在这个上下文里会怎么读、怎么施工（未展示实际代码）
- evaluator 会跑哪些命令验收
- memory 条目的记录形态

## 这个示例没展示什么

- 可运行的业务代码（不是示例的重点）
- dreamer 的上浮流程（需要多条 memory 累积后才有意义，本示例只展示单条 memory）
- git-push 的提交流程（本示例不展示 git 交互）
- code-reviewer / doc-refresher 的报告样本（主要是角色通用能力，不需要多个技术栈各来一份）
