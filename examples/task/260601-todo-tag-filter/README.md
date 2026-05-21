# Todo 按标签过滤 — 任务文档总览

任务级目录形态示例。展示一个完整任务在 dev-roleplay 体系下沉淀出来的全部产物形态。

## 阅读顺序

1. **[`progress.md`](./progress.md)** — 任务入口：目标、步骤、决策记录
2. **[`background/`](./background/)** — 开工前的代码探索（task-designer 产出）
3. **[`plan/`](./plan/)** — 双契约的真实形态（施工契约 + 验收契约同文件）
4. **[`memory/`](./memory/)** — 按日沉淀的决策与踩坑记录
5. **[`memory/SUMMARY.md`](./memory/SUMMARY.md)** — dreamer 蒸馏后的任务级汇总
6. **[`task-reviewer/`](./task-reviewer/)** — 任务专项审查规则（迁移/重构期常用）

## 子目录是怎么长出来的

| 子目录 | 创建时机 | 创建者 |
|--------|---------|--------|
| `progress.md` | 任务一开始 | task-designer |
| `memory/` | 任务一开始（空目录） | task-designer 建空，所有角色按需追加 |
| `plan/` | 步骤 ≥ 3 个或有可并行子任务 | task-designer |
| `background/` | 需要调研、竞品分析或代码考古 | task-designer |
| `task-reviewer/` | 任务有专项审查规则（迁移、重构、契约冻结期） | task-designer |
| `memory/archive/` | dreamer 第一次归档时按需创建 | dreamer |
| `memory/SUMMARY.md` | dreamer 第一次蒸馏时创建 | dreamer |
| `plan/archive/` | 子任务被推翻或合并时按需创建 | task-designer |

**不预建空目录**——目录的存在本身就在告诉读者"这里发生过事情"。

## 闭合双 SSOT 闭环的位置

本任务沉淀过程产生了一条上浮到 `docs/knowledge/principles/` 的跨任务原则，形成 dreamer 引擎的完整工作闭环。详见 [`memory/SUMMARY.md`](./memory/SUMMARY.md) 的"已上浮"段。
