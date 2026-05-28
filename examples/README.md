# Examples

dev-roleplay-harness 在真实项目里**填充后**的样子——含完整任务沉淀的全部产物。

骨架（`roles/`、`doctrine/`）告诉你形态和原理，这里告诉你它落进 git 之后长什么样。

## 项目设定

示例假设的项目是个 TypeScript + Node.js API 服务，技术栈细节不重要——
路径和命名是通用形态，换成 Python / Go 项目沿用同一套结构。

- 现有能力：Todo 管理（增删改查）
- 本次任务：给 Todo 服务加按标签过滤接口；后续补分页

## 目录形态

```
examples/
├── engineering/                                    # 工程规范——告诉 coder 怎么写
│   ├── conventions.md                              # 通用基线：命名 / 错误 / 日志 / API
│   └── tracing.md                                  # 主题分册示例：可观测性规范
├── review/                                         # 审查标准——告诉 reviewer 怎么审
│   └── code-check.md                               # 工程层面的固定 checklist
├── knowledge/                                      # 跨任务知识库（dreamer 上浮目标）
│   └── principles/
│       ├── README.md
│       └── empty-result-vs-404.md                  # 从本任务上浮的一条原则
└── task/
    └── 260601-todo-tag-filter/                     # 本示例的具体任务
        ├── README.md                               # 任务入口
        ├── progress.md                             # 步骤、决策记录
        ├── background/
        │   ├── README.md
        │   └── existing-todo-api-survey.md         # 开工前的代码探索
        ├── plan/
        │   ├── step1-tag-filter-endpoint.md        # 双契约（施工 + 验收）
        │   ├── step2-pagination-when-list-grows.md # 第二个子任务
        │   └── archive/
        │       └── README.md                       # plan 归档机制说明
        ├── memory/
        │   ├── 2026-06-01-tag-index-decision.md    # 5 条按日沉淀
        │   ├── 2026-06-02-empty-result-not-404.md
        │   ├── 2026-06-02-tag-validation-regex.md
        │   ├── 2026-06-03-defer-multi-tag-filter.md
        │   ├── 2026-06-03-cursor-vs-offset-pagination.md
        │   ├── archive/
        │   │   └── 2026-06-01-considered-fulltext-search.md  # 被推翻的归档
        │   └── SUMMARY.md                          # dreamer 蒸馏后的任务级汇总
        └── task-reviewer/                          # 任务专项审查规则
            ├── code-review.md
            └── history/
                └── architecture-review-2026-06-01.md
```

真实项目里，这三个一级目录通常落在仓库根的 `docs/` 下。示例把它们直接放在 `examples/` 下是为了少一层包装，本质布局相同。

## 阅读顺序（推荐）

按"信息生产顺序"走一遍——从开工前的探索，到双契约，到日常沉淀，到蒸馏，到上浮：

1. **基线（双 SSOT）**
   - [`engineering/conventions.md`](./engineering/conventions.md)：编码 / 命名 / 错误 / 日志规范——**告诉 coder 怎么写**
   - [`engineering/tracing.md`](./engineering/tracing.md)：演示 engineering 可按主题拆分（可观测性专项）
   - [`review/code-check.md`](./review/code-check.md)：审查 checklist 与风险分级——**告诉 reviewer 怎么审**

2. **任务入口** — [`task/260601-todo-tag-filter/README.md`](./task/260601-todo-tag-filter/README.md) → [`progress.md`](./task/260601-todo-tag-filter/progress.md)

3. **开工前探索** — [`background/existing-todo-api-survey.md`](./task/260601-todo-tag-filter/background/existing-todo-api-survey.md)：task-designer 的代码考古

4. **双契约真实样本** — [`plan/step1-tag-filter-endpoint.md`](./task/260601-todo-tag-filter/plan/step1-tag-filter-endpoint.md)（必看）+ [`plan/step2-pagination-when-list-grows.md`](./task/260601-todo-tag-filter/plan/step2-pagination-when-list-grows.md)：施工契约 + 验收契约同文件配对

5. **日常沉淀** — [`memory/`](./task/260601-todo-tag-filter/memory/) 下 5 条按日决策记录 + 1 条归档

6. **蒸馏** — [`memory/SUMMARY.md`](./task/260601-todo-tag-filter/memory/SUMMARY.md)：dreamer 把 6 条原始 memory 整理成 5 个主题

7. **上浮**（双 SSOT 闭环的关键）— [`knowledge/principles/empty-result-vs-404.md`](./knowledge/principles/empty-result-vs-404.md)：从 memory 抽出的跨任务原则，下次任何"列表查询"任务直接复用

8. **任务专项审查** — [`task-reviewer/code-review.md`](./task/260601-todo-tag-filter/task-reviewer/code-review.md)：本任务期间 code-reviewer 在工程标准之外额外应用的规则

## 这个示例展示了什么

- **engineering / review 分离**：编码规范和审查标准是两个独立 SSOT——前者告诉 coder 怎么写，后者告诉 reviewer 怎么审
- **engineering 可按主题分册**：conventions（基线）+ tracing（专项）演示按需拆分
- **task-designer** 的产出形态：progress / plan / 双契约 / background
- **双契约** 两段如何配对（施工契约的每一项映射到验收契约的某一条）
- **memory 三种典型形态**：决策记录、被否决方案的归档、跨任务原则的上浮
- **dreamer 的蒸馏**：从 6 条原始 memory 到 5 个主题汇总到 1 条上浮
- **双 SSOT 闭环**：本任务产生的 `empty-result-vs-404` 原则成为下一个任务可直接复用的项目级资产
- **task-reviewer 槽位**：迁移期 / 重构期 / 契约冻结期的任务专项审查规则

## 这个示例没展示什么

- 可运行的业务代码（不是示例的重点）
- coder / evaluator 的实际报告样本（角色通用能力，不需要多个技术栈各来一份）
- doc-refresher 的输出报告样本（同上）
- git-push 的提交流程（涉及 git 交互，本示例不演示）
- 多任务长期演进后 `knowledge/` 的丰富形态（本示例只演示 1 条上浮）

## 注：这只是一个示例任务

真实任务的复杂度可能远超此处。复杂任务会有：

- `integration-test/` — 多服务集成测试场景集
- `replay-baseline/` — 迁移类任务的对比基线
- 更多并行 step + 更深的 plan/archive
- 几十条 memory 经过多轮 dreamer 蒸馏

这些**按需扩展**，不预建。本示例只展示**必要骨架 + 双 SSOT 闭环**。
