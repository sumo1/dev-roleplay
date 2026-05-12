# dev-roleplay

**在 AI 协作里重建软件工程团队的分工。**

一个大 agent 从设计到验收全包，你得到的是一份自信但漂移的交付物。
这个项目定义了一组有分工、有边界、有纪律的角色，让 AI 协作像一个工程团队而不是一个独角戏。

## 为什么不是一个大 agent

人类团队几十年前就搞明白了：一个人既写代码又自己 review 自己不靠谱。AI 协作同样不靠谱——上下文里留下的东西就是 LLM 的"情感"，自己评自己的产出时会被偏置牵着走。

解法不是让 agent 更聪明，是把它拆成**几个独立视角**，用**契约**把它们串起来。

## 7 个角色

| 角色 | 人类对照 | 核心职责 | 硬边界 |
|------|---------|---------|--------|
| task-designer | Tech Lead | 拆任务、开双契约、生成任务目录 | 不写代码 |
| coder | Engineer | 按契约施工、本地自验 | 不做 drive-by refactor、不碰契约外文件 |
| evaluator | QA | 复跑验收命令、3 轮打回升级 | 不改代码、不补契约 |
| code-reviewer | Staff Reviewer | 独立看 diff、风险分级 | 不改代码、不做 QA 的活 |
| doc-refresher | Tech Writer | 代码 vs 文档一致性 | 不改文档、只标记 |
| dreamer | Knowledge Curator | 按天 memory → 任务 SUMMARY → 跨任务 knowledge | 不发明、不覆盖原始条目 |
| git-push | Release Manager | 前置检查 → review → 提交 → 推送 | 不用 --no-verify / --force / --amend |

## 4 条协议

- **Soul**：每个角色以"性格"而非"任务描述"开头，对抗 LLM 漂移
- **Dual Contract**：同一 plan 文件里切出"施工契约"和"验收契约"，让 coder 和 evaluator 在同一张纸上对齐
- **Refusal Loop**：evaluator 必须复跑、不认 coder 自报；连续 3 轮打回升级到主会话
- **Memory Layering**：写入 / 整理 / 上浮 分给不同角色，避免 agent 自己管自己的记忆

详见 [doctrine/](./doctrine/)。

## 适用 / 不适用

**适用**：
- 多步骤、跨模块的工程任务
- 有数据一致性、API 契约、迁移等"错一点就很贵"的改动
- 需要把决策和教训沉淀下来的长期项目

**不适用**：
- 一次性小改、typo 修复
- 探索性 prototyping
- 单文件脚本

有反例才显真实。

## 怎么用

这个项目当前在 **Claude Code** 下验证过。其他 harness（Codex / Cursor / Aider 等）也有 subagent 原语，理论上能跑，但我们没做适配——使用者自行承担。

### Claude Code 用户

1. 把 [`roles/`](./roles/) 下 7 个角色文件拷到你项目的 `.claude/agents/` 下（或自建一份薄引用）
2. 参考 [`integration/claude-code/`](./integration/claude-code/) 填入你项目的具体约定（目录结构、验收命令、规范文档锚点）
3. 在主会话里按需 spawn 角色

### 其他 harness

[`roles/`](./roles/) 里是语言/工具无关的 prompt 骨架。你需要自己写编排代码（什么时候 spawn 谁、怎么维护 3 轮计数、如何把 plan 文件内容喂给 subagent）。

## 这个项目不是什么

- 不是 LangGraph / Autogen 的替代品——那是编排框架，这是**角色设计模式**
- 不是通用多 agent framework——只讲"软件工程"这一个具体场景
- 不是开箱即用的 SaaS——要求你读懂 7 个角色 md，然后融进你项目的工作流

## License

[MIT](./LICENSE)
