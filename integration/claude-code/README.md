# Claude Code 集成

把 [`../../roles/`](../../roles/) 下的骨架接进 Claude Code 的做法。

包含：

- `.claude/agents/` 下文件布局
- `tools:` 字段的推荐白名单（按角色分）
- 主会话如何 spawn subagent 的示例
- 3 轮打回循环的轮次维护方式
- plan 文件如何被 subagent 读到

## 填空清单

使用前需要在项目里填的占位符：

| 占位符 | 说明 | 示例 |
|--------|------|------|
| `{task-doc-root}` | 任务文档根目录 | `docs/task/` |
| `{knowledge-root}` | 跨任务知识库 | `docs/knowledge/` |
| `{engineering-root}` | 项目规范目录 | `docs/engineering/` |
| `{verification-spec}` | 本地自验清单锚点 | 项目 CONTRIBUTING.md 中的段落 |
| `{sensitive-boundary-spec}` | 高风险/迁移期专项规则（如有） | 项目规范文档中的相应段落 |

**不填具体命令**——自验怎么跑、用什么工具、哪些文件触发哪些检查，由项目在 `{verification-spec}` 里自定义。框架不规定。
