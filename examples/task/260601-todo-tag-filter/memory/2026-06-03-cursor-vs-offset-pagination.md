# step2 分页选 cursor (created_at) 而非 offset

> 日期：2026-06-03
> 作者：task-designer（规划 step2 时决定）

## 背景

step2 要给 `GET /todos?tag=` 加分页。两种主流选择：

1. **offset 分页**：`?offset=20&limit=20`
2. **cursor 分页**：`?cursor={created_at}&limit=20`

## 决策

选 **cursor 分页，cursor 用 `created_at`**。

## 理由

- offset 在数据持续写入时会出现"翻页时漏看 / 重复看"——第二页时插入了新数据，offset=20 实际跳过了不同的记录
- cursor 用时间戳更稳定：`WHERE created_at < $cursor` 永远指向"那条记录之前的"
- GIN 索引在 step1 已经建在 `tags` 上；`created_at` 排序由现有的隐式 b-tree 主键覆盖（`id` 是 ULID 时间有序）

## 被否决的方案

- **offset 分页**：翻页一致性问题
- **cursor 用 `id`**：ULID 时间有序所以 `id` 和 `created_at` 等价，但 `created_at` 对调用方更自解释（不需要懂 ULID 内部结构）

## 已知遗留风险

`created_at` 相同精度（毫秒）的两条记录跨页时可能重复或漏掉。当前数据量下不触发；未来需要时加二级排序 `(created_at, id)`。已写在 step2 的"剩余风险"段，留给下一任务。

## 相关提交

step2 同批次落地。
