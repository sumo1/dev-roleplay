# 集合查询无匹配返回 200，不返回 404

**一句话**：列表端点（按条件过滤）找不到匹配项时返回 HTTP 200 + 空数组，**不**返回 404。

## 为什么

HTTP 404 在语义上是"**这个 URL 指向的资源不存在**"——例如 `GET /todos/abc123` 这种 ID 找不到的情况。

列表查询找不到匹配项是**"查询的结果集为空"**——URL 指向的是"集合"，集合本身永远存在。

混用会污染调用方的错误处理：

| 反模式 | 正模式 |
|--------|--------|
| `GET /todos?tag=foo` 查不到 → 404 | `GET /todos?tag=foo` 查不到 → 200 + `data: []` |
| 调用方写 `if status == 404 && knownTagButNoData ... else if status == 404 && unknownEndpoint ...` | 调用方写 `if data.length === 0 ...` |
| 真正的"endpoint 不存在 / 路由错"的 404 信号被业务空结果淹没 | 404 永远只表达"路由 / 资源不存在"——信号不被污染 |

## 触发信号

看到以下情况立即对照本原则：

- 设计列表端点时考虑"查不到怎么办"
- 设计搜索 / 过滤 / 分页 endpoint 时
- 评审现有端点的错误响应表

## 应对动作

1. **资源单查**（`GET /resource/:id`）：找不到 → 404
2. **集合查询**（`GET /resource?filter=...`）：找不到 → 200 + 空数组（或 `{ data: [], next_cursor: null }` 等成功结构）
3. **不要混用**：同一端点不能在不同输入下既能 404 也能 200 + `[]`

## 边界情况

- 路径参数指向"不存在的父资源"——视为单查，仍可 404。例如 `GET /users/:userId/todos?tag=foo` 中 `userId` 不存在 → 404；存在但无匹配 todo → 200 + `[]`
- HTTP 204（No Content）：技术上也表达"成功无内容"，但与"成功永远返回 `{ code: 0, data: ... }`"的项目约定不一致——除非项目本来就用 204，否则不混用

## 来源案例

- [`docs/task/260601-todo-tag-filter/memory/2026-06-02-empty-result-not-404.md`](../../task/260601-todo-tag-filter/memory/2026-06-02-empty-result-not-404.md) — 为 `GET /todos?tag=` 端点确定空结果语义时的讨论
