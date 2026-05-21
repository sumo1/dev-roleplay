# Todo 标签过滤 — 任务专项审查规则

> 仅针对本任务（step1 / step2）的额外审查规则。
> 与项目工程标准 `docs/engineering/conventions.md` 组合使用。
> **任务完成后本文归档。**

## 1. 已冻结的边界（任意 diff 触碰即高风险）

### API 边界

- 公开响应 `data` 数组的字段集 = `{ id, title, tags, created_at }`，**不增不减不改名**
- `code: 3` (ILLEGAL_INPUT) → HTTP 400，不变
- `next_cursor` 字段类型恒为 `string | null`，不混合其他形态

### 命名约定

- 响应 JSON 全部 `snake_case`——`created_at` 不能写成 `createdAt`、`next_cursor` 不能写成 `nextCursor`
- service / repo 内部 TS 类型保持 `camelCase`，**只在 mapper 层做转换**

### 数据库边界

- Todo 实体字段不改（迁移期被冻结，原因见 [`background/existing-todo-api-survey.md`](../background/existing-todo-api-survey.md)）
- `migrations/` 文件名格式 `{YYYYMMDD}-{kebab}.sql`，一个变更一个文件

## 2. 专项 checklist

code-reviewer 每次审查本任务相关 diff 时必过：

- [ ] **是否引入了"未来可能用"的抽象**？（PaginationHelper / Cursor 类等）禁止——见 [`memory/SUMMARY.md`](../memory/SUMMARY.md) 主题 4
- [ ] **是否顺手改了 step 范围外的 todo handler**？（drive-by refactor）禁止
- [ ] **是否在响应里多塞了 `recipe_prompt` / `internal_*` 字段**？公开 API 不能透出内部字段
- [ ] **正则校验是否与已沉淀的决策一致**？`^[a-zA-Z0-9_-]{1,50}$`，见 SUMMARY 主题 3
- [ ] **是否为多标签过滤预留了接口但未实现**？禁止——见 SUMMARY 主题"多标签推迟"

## 3. 已知噪声（低风险，不阻塞）

- `tags` 字段在 `select` 列表里出现两次（join 时）——已知，TS 类型层无问题，下次重构时统一
- handler 里的 query schema 与 step1/step2 有 ~10 行重复——容忍，不抽公共

## 4. 与跨任务知识的引用

- HTTP 空结果语义遵循 [`docs/knowledge/principles/empty-result-vs-404.md`](../../../knowledge/principles/empty-result-vs-404.md)
