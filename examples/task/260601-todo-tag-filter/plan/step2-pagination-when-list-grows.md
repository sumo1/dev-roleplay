# step2-pagination-when-list-grows

## 背景

step1 加了 `GET /todos?tag={name}` 但未分页。本步骤补 cursor 分页，避免标签下 todo 过多时一次性返回过大响应。

上下游：

- 上游：依赖 step1 已经合并（端点存在 + 索引就位）
- 下游：无

**与 step1 并行性说明**：本步骤 **依赖** step1，不能并行。文件范围与 step1 重叠（都改 `src/http/todo.ts`），所以只能串行。

## 施工契约

### 范围

- **可改文件**：
  - `src/http/todo.ts`（改 handler，加 query string 校验）
  - `src/service/todo-service.ts`（改 `findByTag` 签名）
  - `src/repository/todo-repo.ts`（改 SQL，加 `WHERE created_at < cursor` 条件）
- **不可改文件**：
  - 已有的 `listTodos` / `createTodo` 等其他端点（不顺手加分页）
  - `migrations/`（不需要新 DDL）
- **不可新增的抽象**：
  - 不新增通用 PaginationHelper / Cursor 类——只在 todo 模块内联

### 产出清单

- `GET /todos?tag={name}&cursor={iso8601}&limit={int}`
  - `cursor`：可选，传上一页最后一条 `created_at`
  - `limit`：可选，默认 20，最大 100，取值非法 → 400
  - 响应新增字段：`next_cursor: string | null`（`null` 表示已到末尾）
- service 方法签名：`findByTag(tag: string, opts: { cursor?: Date, limit: number }): Promise<{ items: Todo[], nextCursor: Date | null }>`

### 约束（已冻结的边界）

- 沿用 step1 的字段命名 `snake_case`、错误码 `code: 3` for ILLEGAL_INPUT
- **不改** `data` 数组的字段结构——`next_cursor` 与 `data` 同级
- 不破坏 step1 的现有调用方（不传 cursor/limit 时行为与 step1 等价）

### 复用的现有模式

- `cursor` 用 `created_at` 而非 `id`：见 [`memory/2026-06-03-cursor-vs-offset-pagination.md`](../memory/2026-06-03-cursor-vs-offset-pagination.md)
- query string 校验沿用 step1 的 zod schema 写法

### 依赖的前置子任务

- **step1-tag-filter-endpoint** 必须已合并（端点已存在 + GIN 索引就位）

## 验收契约

### 代码结构验证

- [ ] handler 接受 `cursor` 和 `limit` query 参数
- [ ] service `findByTag` 签名包含 `opts: { cursor?, limit }`
- [ ] repository SQL 包含 `WHERE created_at < $cursor ORDER BY created_at DESC LIMIT $limit`

### 命令验收

| 命令 | 通过标准 |
|------|---------|
| `npm run type-check` | exit 0 |
| `npm run lint` | exit 0 |
| `npm test -- src/service/todo-service.test.ts` | 新增至少 4 个测试（首页 / 翻页 / 末页 / 非法 limit），全部通过 |
| `npm test -- src/http/todo.test.ts` | 新增至少 3 个测试（默认 limit / 自定义 limit / cursor 翻页），全部通过 |

### 数据 / 字段验收

- [ ] 响应顶层包含字段 `next_cursor`，类型 `string | null`
- [ ] 当返回数据条数 < limit 时 `next_cursor === null`
- [ ] 当返回数据条数 === limit 时 `next_cursor` 等于最后一条的 `created_at`

### 负面用例

- [ ] `?tag=foo&limit=0` → 400 + `code: 3`
- [ ] `?tag=foo&limit=101` → 400 + `code: 3`
- [ ] `?tag=foo&cursor=not-a-date` → 400 + `code: 3`
- [ ] `?tag=foo`（不传 cursor/limit）→ 200，行为与 step1 等价（限默认 20 条）

### 剩余风险

- 跨多页时 `created_at` 相同的两条记录会重复或漏掉——本次不解决（数据量小不触发，未来需要再加 `id` 二级排序）

## 施工尝试记录

> 调度者按轮次追加。首次生成 plan 时为空。
