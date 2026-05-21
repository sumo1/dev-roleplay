# Roles

角色骨架。每个文件可以独立阅读，7 份共同服务于知识在时间轴上的三段流动（详见 [doctrine/00 — 双 SSOT 与三个时间方向](../doctrine/00-dual-ssot.md)）。

## 清单

| 角色 | 时间方向 | 定位 |
|------|---------|------|
| [task-designer](./task-designer.md) | 过去 → 现在 | 读历史沉淀，拆任务、开契约、生成任务文档结构 |
| [coder](./coder.md) | 现在 | 按契约施工、本地自验 |
| [evaluator](./evaluator.md) | 现在 | 复跑验收命令、独立判定是否达标 |
| [code-reviewer](./code-reviewer.md) | 现在 | 独立看 diff、风险分级 |
| [doc-refresher](./doc-refresher.md) | **现在 → 未来（保鲜引擎）** | 防止 SSOT 文档腐坏，让下次任务起点新鲜 |
| [dreamer](./dreamer.md) | **现在 → 未来（沉淀引擎）** | 把当前任务的业务判断上浮回知识库 |
| [git-push](./git-push.md) | 流程闸口 | 前置检查 → 审查 → 提交 → 推送 |

## 结构约定

每个角色文件遵循同一套结构：

- **Soul** — 角色的性格与基本态度
- **输入契约** — 调用方必须提供的信息
- **工作流程** — 具体步骤
- **产出** — 交付物形态
- **硬边界** — 能做什么、不能做什么
- **禁止事项** — 行为红线

## 默认路径约定

角色之间靠共享的文件路径交接。约定的默认布局：

```
docs/task/{task-id}/         任务文档根
  ├── progress.md            任务入口
  ├── plan/{step}.md         子任务契约（施工契约 + 验收契约同文件）
  ├── memory/*.md            按日沉淀
  └── task-reviewer/         任务专项审查规则（可选）
docs/knowledge/              跨任务知识（dreamer 上浮目标）
docs/engineering/            项目规范（引用锚点）
src/                         源代码根
```

fork 后按自己项目的实际路径调整即可，保持 7 个角色内部引用同一套就能正常交接。
