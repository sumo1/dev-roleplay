# Todo 按标签过滤

## 目标

Todo 服务支持 `GET /todos?tag={name}[&cursor=&limit=]` 接口，按标签过滤返回的 Todo 列表，支持 cursor 分页，接口契约、性能、数据一致性均满足项目规范。

## 背景

- 用户反馈列表展开后找不到特定主题的 todo
- 已有数据：Todo 实体已有 `tags: string[]` 字段，但无索引
- 详细代码现状见 [`background/existing-todo-api-survey.md`](./background/existing-todo-api-survey.md)

## 步骤

1. [x] **step1-tag-filter-endpoint** — 新增 `GET /todos?tag={name}` 接口，加 GIN 索引
2. [ ] **step2-pagination-when-list-grows** — 给 step1 端点补 cursor 分页

step2 依赖 step1（文件范围重叠），**串行**执行。

## 决策记录

| 决策 | 日期 | 链接 |
|------|------|------|
| 不在本任务里支持"多标签 AND/OR 过滤" | 2026-06-03 | [`memory/2026-06-03-defer-multi-tag-filter.md`](./memory/2026-06-03-defer-multi-tag-filter.md) |
| 为 `tags` 字段加 GIN 索引而不是全文索引 | 2026-06-01 | [`memory/2026-06-01-tag-index-decision.md`](./memory/2026-06-01-tag-index-decision.md) |
| 空结果返回 `data: []` 而非 404 | 2026-06-02 | [`memory/2026-06-02-empty-result-not-404.md`](./memory/2026-06-02-empty-result-not-404.md) → 已上浮到 [`knowledge/principles/`](../../knowledge/principles/empty-result-vs-404.md) |
| tag 校验正则 `^[a-zA-Z0-9_-]{1,50}$` | 2026-06-02 | [`memory/2026-06-02-tag-validation-regex.md`](./memory/2026-06-02-tag-validation-regex.md) |
| 分页用 cursor (created_at) 不用 offset | 2026-06-03 | [`memory/2026-06-03-cursor-vs-offset-pagination.md`](./memory/2026-06-03-cursor-vs-offset-pagination.md) |

## 沉淀产出

- 任务级汇总：[`memory/SUMMARY.md`](./memory/SUMMARY.md)（dreamer 于 2026-06-04 整理）
- 上浮到跨任务知识：[`knowledge/principles/empty-result-vs-404.md`](../../knowledge/principles/empty-result-vs-404.md)
- 归档：[`memory/archive/2026-06-01-considered-fulltext-search.md`](./memory/archive/2026-06-01-considered-fulltext-search.md)

## 任务专项审查规则

[`task-reviewer/code-review.md`](./task-reviewer/code-review.md) — 本任务运行期间，code-reviewer 在工程标准之外额外应用此规则；任务收尾后归档。
