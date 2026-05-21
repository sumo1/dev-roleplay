# step1-tag-filter-endpoint

## 背景

给 `src/http/todo.ts` 加一个新端点 `GET /todos?tag={name}`，按标签过滤返回的 Todo 列表。数据层的 `tags` 字段已存在但无索引，本步骤同时加 GIN 索引支持查询。

上下游：

- 上游：无前置子任务
- 下游：下次迭代会加"多标签过滤"，本次不涉及

## 施工契约

### 范围

- **可改文件**：
  - `src/http/todo.ts`（新增端点处理函数）
  - `src/service/todo-service.ts`（新增 `findByTag` 方法）
  - `src/repository/todo-repo.ts`（新增 `findByTag` 方法）
  - `migrations/20260601-add-todo-tags-gin-index.sql`（新文件）
- **不可改文件**：
  - `src/http/todo.ts` 的已有端点（不允许顺手重构）
  - Todo 实体定义（本次不改字段）
- **不可新增的抽象**：
  - 不新增通用 Filter/Query Builder 类
  - 不改当前分层结构

### 产出清单

- 端点：`GET /todos?tag={name}`
  - 入参：query string `tag`，必填，长度 1-50，仅允许 `a-zA-Z0-9-_`
  - 响应：`{ code: 0, data: [ { id, title, tags, created_at } ] }`
  - 空结果返回 `data: []`，不返回 404
- service 方法：`TodoService.findByTag(tag: string): Promise<Todo[]>`
- repository 方法：`TodoRepository.findByTag(tag: string): Promise<TodoRow[]>`
- migration：在 `tags` 字段上创建 GIN 索引

### 约束（已冻结的边界）

- 响应字段命名 `snake_case`（`created_at` 不能写成 `createdAt`）
- 错误响应使用 `AppError` + 项目统一 code 体系
- 业务错误码沿用现有体系，不新增
- 参数校验失败返回 HTTP 400 + `code: 3`（ILLEGAL_INPUT）
- 不改已有 Todo 列表端点的响应结构

### 复用的现有模式

- 端点注册：参见 `src/http/todo.ts` 里 `GET /todos` 的写法
- Service→Repo 层调用：参见 `src/service/todo-service.ts` 里 `list()` 的写法
- Migration 命名规范：`{YYYYMMDD}-{description}.sql`，参见 `migrations/` 历史文件

### 依赖的前置子任务

无。

## 验收契约

### 代码结构验证

- [ ] `src/http/todo.ts` 中新增路由注册 `GET /todos?tag=`，handler 函数签名正确
- [ ] `src/service/todo-service.ts` 中新增方法 `findByTag(tag: string): Promise<Todo[]>`
- [ ] `src/repository/todo-repo.ts` 中新增方法 `findByTag(tag: string): Promise<TodoRow[]>`
- [ ] `migrations/20260601-add-todo-tags-gin-index.sql` 存在，包含 `CREATE INDEX ... USING GIN (tags)` 语句

### 命令验收

| 命令 | 通过标准 |
|------|---------|
| `npm run type-check` | exit 0，无新增类型错误 |
| `npm run lint` | exit 0，无新增 lint 错误 |
| `npm test -- src/service/todo-service.test.ts` | 新增至少 3 个测试用例（查到结果 / 查不到结果 / 非法输入），全部通过 |
| `npm test -- src/http/todo.test.ts` | 新增至少 2 个 HTTP 层测试（成功 + 400），全部通过 |

### 数据 / 字段验收

- [ ] 接口响应的每个元素包含字段：`id`、`title`、`tags`（数组）、`created_at`
- [ ] 字段命名全部 `snake_case`
- [ ] `tags` 字段类型为 `string[]`
- [ ] migration 文件里的索引类型为 `USING GIN`

### 负面用例

- [ ] `GET /todos?tag=` （空值）→ 返回 HTTP 400，`code: 3`
- [ ] `GET /todos?tag=<script>` （非法字符）→ 返回 HTTP 400，`code: 3`
- [ ] `GET /todos?tag=` 长度 51 字符 → 返回 HTTP 400，`code: 3`
- [ ] `GET /todos?tag=nonexistent-tag` → 返回 HTTP 200，`data: []`

### 剩余风险（仅提示 code-reviewer / doc-refresher 关注）

- GIN 索引在百万级数据量下的查询性能——本次没 benchmark，上线后观察
- 多标签过滤的未来扩展——需要重构 repo 方法签名，届时可能破坏兼容

## 施工尝试记录（打回循环中累积）

> 本段由调度者在每轮 coder / evaluator 结束后追加。首次生成 plan 时为空。
