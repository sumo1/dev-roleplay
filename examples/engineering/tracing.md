# 可观测性规范（示例）

> 工程规范的子文件——演示**`engineering/` 目录可以按主题拆分**，不是单文件容器。
>
> 本文不绑定具体 APM / OpenTelemetry / 厂商实现，只规定本仓库对**追踪、日志、指标**这三类信号的**通用规范**。具体技术栈实现作为代码 SSOT 在 `src/infra/` 下。

## 为什么单独拆这个主题

`engineering/conventions.md` 涵盖的是**所有改动都要遵守的基线**——命名、错误处理、API 格式等。
`engineering/tracing.md` 涵盖的是**只有改动到链路 / 埋点 / 日志切面时**才需要读的内容。

把高频常用的和低频专项的拆开，避免：

- coder 每改一行代码都要重读 200 行追踪规范
- 追踪相关的修改被埋在 conventions.md 第 13 节、容易漏看

按主题拆是 `engineering/` 目录的**通用做法**——其他可能的拆分主题（按项目需要选）：

- `engineering/security.md`——鉴权、加密、注入防御
- `engineering/migration.md`——数据迁移规范
- `engineering/i18n.md`——国际化与本地化
- `engineering/api-versioning.md`——API 版本兼容策略

---

## 1. 三类信号的职责

| 信号 | 用途 | 何时打 |
|------|------|------|
| **Trace** | 一次请求穿越多个服务/组件的完整链路 | 每个外部入站请求 + 每个跨服务调用 |
| **Log** | 离散事件、错误、业务关键点 | 关键决策、异常、外部依赖响应 |
| **Metric** | 可聚合的数值（QPS、延迟、错误率） | 系统健康观测 |

三类信号必须**互相交叉引用**——log 里带 `trace_id`，metric 按 service / endpoint 维度切，trace 里挂关键 log 摘要。任何一类信号孤立存在都不够。

---

## 2. Trace ID 规范

### 生成与传播

- 使用 W3C Trace Context（`traceparent` header）
- 入站请求：有 header 则继承 trace_id，无则新生成
- 出站调用：必须把当前 trace_id 透传到下游

### 透传纪律

跨服务调用时**必须**透传：

- `traceparent`（trace_id + span_id）
- `x-request-id`（业务请求 ID，与 trace_id 不同——用于业务侧关联）
- 任何项目特定的 traffic-routing header（染色泳道、灰度标记等）

不透传 = 链路在那个调用点断掉。这是经常被遗漏的事——**code-reviewer 审查跨服务调用时必查**。

---

## 3. 日志规范

### 结构化优先

禁止 `console.log` / `print` 入生产代码——使用项目约定的 logger。

每条日志至少包含：

- timestamp
- level（debug / info / warn / error）
- trace_id（自动注入）
- 业务上下文（user_id / request_id / tenant_id 等，按项目实际）
- 消息 + 结构化字段

### 等级使用

| 等级 | 何时用 | 何时不用 |
|------|------|------|
| `debug` | 开发期定位问题 | 生产环境默认关闭 |
| `info` | 业务关键节点 | 不要每个函数入口都打 |
| `warn` | 业务预期内的"不正常"（参数非法、找不到资源） | 不要给"可能值得关注"的事打 warn——会噪音淆没真正的 warn |
| `error` | 真实错误：不该发生的异常、外部依赖失败 | 业务校验失败不算 error |

**核心规则**：等级是**给运维报警用的**——`error` 应该可被监控规则直接触发告警。如果 `error` 里混入业务校验失败，告警系统就报废。

---

## 4. 指标规范

### 命名约定

- 用项目统一的 prefix（如 `myapp_`）
- 用 `_` 分词，不用 camelCase
- 单位放在名字最后：`http_request_duration_seconds`、`db_query_size_bytes`

### 标签（label）爆炸

不要把高基数维度作为 label——例如 `user_id`、`request_id`、`url` 完整路径——会让指标存储炸掉。

允许作为 label 的常见维度：

- `service` / `endpoint`（路由模板，非具体 URL）
- `method`（HTTP method 等枚举值）
- `status` / `error_type`（受控枚举）

把高基数 ID 留给 trace 和 log，**不要塞进 metric label**——这是 metric 系统的硬纪律。

---

## 5. 异步与切面

### 异步上下文传播

- 项目使用什么异步原语（`async/await` / Goroutine / `CompletableFuture`），文档里要写清楚 trace context 如何在该原语下传播
- 不传播的边界场景必须**显式列出**——例如某些定时任务、某些事件回调

### 切面日志（access log）

入口和出口必须有切面日志：

- 入口：method、path、headers 摘要、身份信息
- 出口：status、duration、关键响应字段
- 异常路径：异常类型、堆栈摘要、trace_id

切面日志由 framework hook 实现，**业务代码不重复打**——重复打就成了噪音。

---

## 6. 与代码 SSOT 的关系

本文是规范，**不是实现指南**。具体怎么初始化 OTel SDK、怎么配置 logger、怎么写 hook，看 `src/infra/` 下的代码——代码是真相，本文只解释为什么这么做。

代码改了但本文没跟上 → doc-refresher 检测到，列入"严重过时"。
