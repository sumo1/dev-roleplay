# dev-roleplay

让 AI 写的代码，和它该理解的业务知识，一起沉淀进同一个仓库。

## 它在解决什么

dev-roleplay 是一套面向 AI coding agent 的协作骨架——七个角色、几条协议、一份目录约定。最直观的用法是把"让一个大 agent 把活全包"换成"一组各司其职的角色按契约协作"，用独立视角对抗单 agent 的"自洽但漂移"。

但角色分工只是表层。这套骨架真正想解决的是另一个问题：

> AI 写一次代码很容易，让它每写一次都比上次更懂这个项目，难。

每次任务结束后，agent 学到的判断、踩过的坑、被否决的方案，全在那次会话的上下文里。会话一关，全部蒸发。下次新 agent 来，从零开始论证已经决定过的事，从零开始踩已经踩过的坑。

dev-roleplay 给的解法是把**业务知识**作为和**代码**同等重要的项目资产——放进 git、和代码一起 review、一起 commit、一起持续维护。这样后续来的人和 agent 都能站在前人的沉淀上做事，而不是每次重启炉灶。

## 双 SSOT：代码和文档都是一等公民

代码表达系统**怎么跑**。
文档表达系统**为什么这么跑**——业务意图、边界来源、被否决的方案。

后面这一半，是项目工程除了代码之外的另一条**业务表达**——SSOT 的另一条腿。代码无法表达"我们曾经考虑过 A，因为 X 否掉了"，无法表达"这个字段为什么用 snake_case"，无法表达"这层抽象为什么没加"。这些活在代码"负空间"里的判断，只能交给文档承担。

两个 SSOT 必须始终一致，否则任何一边都会退化成装饰。这是为什么文档必须放进 git，和代码用同一套纪律维护——只有当文档会被 review、被 diff、被回溯，它才会被认真对待。Confluence、Notion、飞书都不行，它们让文档脱离工程纪律，文档很快就和代码脱节。

哲学的完整推导在 [`doctrine/00-dual-ssot.md`](./doctrine/00-dual-ssot.md)。

## 七个角色服务三个时间方向

整套 agents 不是一条横向流水线，而是知识在时间轴上的三段流动：

```
   过去                 现在                  未来
─────────────────────────────────────────────────────
 历史沉淀  ─→   当前任务的拆解 + 执行  ─→   沉淀回去

knowledge/       task-designer              dreamer
engineering/     coder + evaluator             ↓
   ↑             code-reviewer           knowledge/
   │             doc-refresher  ←─保鲜   engineering/
   │                 ↓                       ↑
   └──────  下一次任务从新鲜起点出发  ─────────┘
```

- **过去 → 现在**：task-designer 强制读历史沉淀，让上次任务的判断真正被这次用上
- **现在**：coder / evaluator / code-reviewer 三个独立视角拦截不同方向的偏差
- **现在 → 未来**：doc-refresher 把可能腐坏的文档标出来（保鲜），dreamer 把这次任务里的判断和教训上浮回知识库（沉淀）

doc-refresher 和 dreamer 是体系的两台引擎——一台防止过去→现在的链路腐坏，一台保障现在→未来的链路畅通。少任何一台，业务知识每次任务结束就流失掉，整套 agents 体系退化成"几个干活的工种"。

## 角色清单

| 角色 | 时间方向 | 核心职责 |
|---|---|---|
| [task-designer](./roles/task-designer.md) | 过去 → 现在 | 读历史沉淀，拆任务、开契约 |
| [coder](./roles/coder.md) | 现在 | 按契约施工、本地自验 |
| [evaluator](./roles/evaluator.md) | 现在 | 复跑验收命令，独立判定 |
| [code-reviewer](./roles/code-reviewer.md) | 现在 | 看 diff、风险分级 |
| [doc-refresher](./roles/doc-refresher.md) | 现在 → 未来（保鲜引擎） | 把可能腐坏的文档标出 |
| [dreamer](./roles/dreamer.md) | 现在 → 未来（沉淀引擎） | 把流水记忆酿造为长期知识 |
| [git-push](./roles/git-push.md) | 流程闸口 | 检查 → review → 提交 → 推送 |

每个角色文件按同一套结构写：性格 → 输入契约 → 工作流程 → 产出 → 硬边界 → 禁止事项。性格段（"Soul"）放最前——它不是装饰，是角色行为的核心约束。详见 [`doctrine/02-soul-section.md`](./doctrine/02-soul-section.md)。

## 几条协议

让七个角色真正协作起来的，是仓库里几个跨角色的硬约定：

- **双契约**——每个独立子任务的 plan 文件里同时写施工契约（给 coder）和验收契约（给 evaluator），两段配对、互相对应。详见 [`doctrine/03-dual-contract.md`](./doctrine/03-dual-contract.md)
- **打回循环**——evaluator 必须独立复跑命令，不认 coder 自报；同一条契约项连续多轮打回时升级给人类裁决，避免两个 agent 越修越乱。详见 [`doctrine/04-refusal-loop.md`](./doctrine/04-refusal-loop.md)
- **三层记忆**——按日沉淀 → 任务汇总 → 跨任务知识，写入、整理、上浮分别由不同角色负责，避免 agent 自己管自己的记忆。详见 [`doctrine/05-memory-layering.md`](./doctrine/05-memory-layering.md)

## 起源

这套骨架不是先有理论再写示例，是从一个真实的跨多仓库服务迁移项目里活下来的——多 coder 并行施工逼出了双契约和文件范围互斥；反复看到"用户点破同一类问题"沉淀成了"先质疑问题是否成立"原则；跨仓库切流的漂移代价催生了三层记忆流动；文档与代码不断脱节让"提交前扫一遍文档新鲜度"成了流水线的固定一站。

脱敏后的真实形态在 [`examples/typescript-api/`](./examples/typescript-api/)——一个完整任务从 background 探索、双契约 plan、按日 memory，到 dreamer 蒸馏出 SUMMARY、再上浮一条原则到 `knowledge/principles/` 的全程。

## 用与不用

**适合**：多步骤、跨模块、错一点就很贵、会被多次回看的工程任务——数据一致性、API 契约、服务迁移、长期演进的产品代码。

**不适合**：一次性小改、探索性 prototyping、单文件脚本。还有一种情况要特别提：团队**不接受文档进 git**，文档继续放外部工具——这种情况 doc-refresher 和 dreamer 失去落点，整套体系会退化成"七个干活的工种"，没有意义。

详细的反模式讨论在 [`doctrine/06-anti-patterns.md`](./doctrine/06-anti-patterns.md)。

## 范围

dev-roleplay 面向支持独立 subagent 原语的 coding agent 工具（如 Claude Code）。
角色骨架与语言、框架、技术栈无关；具体路径、命令、规范由使用者在自己项目内定义。

具体接入方式见 [`integration/claude-code/`](./integration/claude-code/)。

## 目录

- [`doctrine/`](./doctrine/) — 设计思想与反模式（先读 [`00-dual-ssot`](./doctrine/00-dual-ssot.md)）
- [`roles/`](./roles/) — 7 个角色骨架
- [`integration/`](./integration/) — 针对具体 harness 的接入指南
- [`examples/`](./examples/) — 真实项目下的填充样例

## License

[MIT](./LICENSE)
