# 空结果返回 200 + `data: []`，不返回 404

> 日期：2026-06-02
> 作者：task-designer（评审 step1 契约时确认）
> **已上浮**：[`docs/knowledge/principles/empty-result-vs-404.md`](../../../knowledge/principles/empty-result-vs-404.md)

## 背景

step1 草稿里曾考虑：当 `GET /todos?tag=nonexistent` 找不到匹配时返回 HTTP 404。直觉是"资源不存在"。

## 决策

**返回 HTTP 200 + `data: []`。** 不用 404。

## 理由

404 表达的是"**这个 URL 指向的资源不存在**"——例如 `GET /todos/abc123` 这种 ID 找不到的情况。
列表查询找不到匹配项是**"查询的结果集为空"**，不是"资源不存在"——URL 指向的是"todos 集合"，集合本身永远存在。

混用会让消费方写出错的客户端逻辑：

- 用 404 处理空结果 → 调用方不得不区分"业务空结果"和"真正的 endpoint 不存在 / 路由错"，错误处理被污染
- 用 200 + `data: []` → 调用方一次 `if (data.length === 0)` 解决

## 被否决的方案

- **404 + 错误码**：增加调用方区分负担
- **204 No Content**：HTTP 语义上更接近"成功无内容"，但和现有 API（成功永远返回 `{ code: 0, data: ... }`）不一致

## 证据

- 项目其他列表端点（`GET /todos` 不带过滤）空表也返回 200 + `[]`
- HTTP RFC 9110 § 15.5.5：404 用于 origin server 不识别的资源映射，不是空查询结果

## 相关提交

（规划阶段记录）
