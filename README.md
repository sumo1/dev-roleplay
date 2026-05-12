# dev-roleplay

在 AI 协作里重建软件工程团队的分工。

一个大 agent 从设计到验收全包，得到的是一份自信但漂移的交付物。dev-roleplay 主张把工作拆成一组独立的角色，每个角色有自己的性格、职责、硬边界，角色之间靠显式契约交接。

## 为什么不是一个大 agent

人类团队几十年前就搞清楚了一件事：写代码的人不适合验自己写的代码。不是因为能力不够，而是因为**上下文已经把判断力污染了**——你知道你为什么这么写，于是看不见别人会在哪里摔倒。

LLM 有同样的问题。上下文里留下的东西就是它的偏置——它推崇过的方案、它跳过的边界情况、它自己安慰自己"没事"的那几句话，都会在回看产出时替它辩护。让同一个 agent 既写又验，得到的是一套**自洽但漂移**的交付物。

解法不是让 agent 更聪明，是把工作拆成**几个独立视角**，用**契约**把它们串起来。

## 7 个角色

| 角色 | 人类对照 | 核心职责 | 硬边界 |
|------|---------|---------|--------|
| [task-designer](./roles/task-designer.md) | Tech Lead | 拆任务、开契约、生成任务文档结构 | 不写代码 |
| [coder](./roles/coder.md) | Engineer | 按契约施工、本地自验 | 不做 drive-by refactor、不碰契约外文件 |
| [evaluator](./roles/evaluator.md) | QA | 复跑验收命令、独立判定是否达标 | 不改代码、不补契约 |
| [code-reviewer](./roles/code-reviewer.md) | Staff Reviewer | 独立看 diff、风险分级 | 不改代码、不做 QA 的活 |
| [doc-refresher](./roles/doc-refresher.md) | Tech Writer | 代码 vs 文档一致性 | 不改文档，只标记过时项 |
| [dreamer](./roles/dreamer.md) | Knowledge Curator | 把流水式记忆沉淀为长期知识 | 不发明、不覆盖原始条目 |
| [git-push](./roles/git-push.md) | Release Manager | 前置检查 → review → 提交 → 推送 | 不跳过钩子、不强推、不改写历史 |

## 4 条协议

- **[Soul](./doctrine/02-soul-section.md)**：每个角色以"性格"而非"任务描述"开头，用风格化的约束对抗 LLM 在长上下文里的行为漂移
- **[Dual Contract](./doctrine/03-dual-contract.md)**：同一份 plan 文件里同时写"施工契约"和"验收契约"，让施工者和验收者在同一张纸上对齐
- **[Refusal Loop](./doctrine/04-refusal-loop.md)**：验收者必须复跑，不认施工者自报；连续多轮无法达标时升级给主调度者，避免施工者 / 验收者陷入死循环
- **[Memory Layering](./doctrine/05-memory-layering.md)**：记忆分"按日沉淀 / 任务汇总 / 跨任务知识"三层，写入、整理、上浮由不同角色负责，避免一个 agent 自己管自己的记忆

更多思想推导见 [`doctrine/`](./doctrine/)。

## 适用与不适用

**适用**：

- 多步骤、跨模块的工程任务
- 数据一致性、API 契约、迁移等"错一点就很贵"的改动
- 需要把决策和教训沉淀下来的长期项目

**不适用**：

- 一次性小改、typo 修复
- 探索性 prototyping
- 单文件脚本

## 范围

dev-roleplay 面向支持独立 subagent 原语的 coding agent 工具（如 Claude Code）。
角色骨架与语言、框架、技术栈无关；引用的路径、命令、规范文档都由使用者在项目内自行定义。

具体接入方式见 [`integration/claude-code/`](./integration/claude-code/)。
真实项目下的填充形态见 [`examples/`](./examples/)。

## 目录

- [`roles/`](./roles/) — 7 个角色骨架
- [`doctrine/`](./doctrine/) — 设计思想与反模式
- [`integration/`](./integration/) — 针对具体 harness 的接入指南
- [`examples/`](./examples/) — 真实项目下的填充样例

## License

[MIT](./LICENSE)
