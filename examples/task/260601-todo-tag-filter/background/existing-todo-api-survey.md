# 现有 Todo API 调研

> 日期：2026-06-01
> 探索者：task-designer

## 目的

在新增 `GET /todos?tag=` 之前，把现有 Todo API 摸清楚——确认边界，找到可复用模式。

## 现有端点

| 方法 | 路径 | 处理函数 | 响应字段 |
|------|------|---------|---------|
| GET | `/todos` | `src/http/todo.ts:listTodos` | `id`, `title`, `tags`, `created_at` |
| POST | `/todos` | `src/http/todo.ts:createTodo` | 同上 |
| GET | `/todos/:id` | `src/http/todo.ts:getTodo` | 同上 |
| PATCH | `/todos/:id` | `src/http/todo.ts:patchTodo` | 同上 |
| DELETE | `/todos/:id` | `src/http/todo.ts:deleteTodo` | `{ ok: true }` |

## 实体结构

`src/entity/todo.ts` 定义：

- `id: string`（ULID）
- `title: string`
- `tags: string[]`（已有字段，**无索引**）
- `created_at: Date`

API 响应使用 `snake_case`（项目规范，见 `docs/engineering/conventions.md`）。

## 可复用模式

- **路由注册**：`src/http/todo.ts` 里 `GET /todos` 的写法（参数校验 → service → mapper → 响应）
- **service → repo 调用链**：`src/service/todo-service.ts` 的 `list()` 是最近的同类模式
- **错误码**：项目用 `AppError + code` 体系，`code: 3` = `ILLEGAL_INPUT` (HTTP 400)
- **Migration 命名**：`migrations/{YYYYMMDD}-{kebab-description}.sql`

## 约束与注意点

- `tags` 字段无索引——要支持过滤必须加索引
- 现有 `GET /todos` 不分页（数据量小），但本任务可能引入分页需求（见 step2）
- Todo 实体在迁移期间不能改字段（已被另一个任务冻结）

## 没探索的部分

- 多标签 AND/OR 过滤——本任务不做
- 全文搜索——超出范围
