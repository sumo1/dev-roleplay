# dev-roleplay

在 AI 协作里重建软件工程团队的分工，并把**业务知识**和**代码**作为同等重要的双 SSOT 持续演进。

一个大 agent 从设计到验收全包，得到的是一份自信但漂移的交付物，而且这份交付物里**沉淀不下任何东西**——任务结束知识就蒸发了。dev-roleplay 主张两件事：

1. **横向上**——把工作拆成一组独立的角色，每个角色有自己的性格、职责、硬边界，靠显式契约交接，用独立视角对抗单 agent 的"自洽但漂移"
2. **纵向上**——让代码和文档成为**双 SSOT**，由两台专门的引擎（doc-refresher 保鲜 / dreamer 沉淀）把每次任务的业务知识闭环回 repo，让下次任务从更新鲜、更厚实的起点出发

它把 agent 工作流从"一次性 prompt"沉淀为**可复用、可审计、可版本化的项目资产**——角色定义、契约模板、记忆体系都是代码仓库里长期演化的一等公民。

## 设计哲学（先读这一节）

### 1. 代码与文档是双 SSOT

| SSOT | 表达什么 | 不可替代的原因 |
|------|---------|--------------|
| 代码 | 行为真相——系统实际怎么跑 | 表达不了"为什么不这么做"、被否决的方案、约束的来源 |
| 文档 | 业务知识真相——业务为什么这样、边界在哪、什么被否决过 | 业务意图常常活在**负空间**里——代码只能表达正空间 |

两者必须始终一致，否则任何一个都失去 SSOT 资格。这就是为什么文档必须在 git 里、和代码一起被 review、被 commit、被回溯——不能放 Confluence / Notion / 飞书。

详见 [doctrine/00 — 双 SSOT 与三个时间方向](./doctrine/00-dual-ssot.md)。

### 2. 七个角色服务三个时间方向

整套 agents 不是七个工种的横向流水线，而是**知识在时间轴上的三段流动**：

```
   过去                 现在                    未来
─────────────────────────────────────────────────────
 历史沉淀  ───→   当前任务的拆解 + 执行  ───→  沉淀回去

knowledge/        task-designer                dreamer
engineering/      coder + evaluator              ↓
   ↑              code-reviewer            knowledge/
   │              doc-refresher  ←─保鲜    engineering/
   │                  ↓                         ↑
   └────────  下一次任务从新鲜起点出发  ─────────┘
```

- **过去 → 现在**：task-designer 强制读 `engineering/` / `knowledge/` / 最近 `task/`，让历史沉淀真正被用上
- **现在**：coder / evaluator / code-reviewer 三个独立视角拦截不同方向的偏差
- **现在 → 未来**：doc-refresher 把变更后可能腐坏的 SSOT 标出来（保鲜），dreamer 把当前任务里的判断与教训上浮回知识库（沉淀）

doc-refresher 和 dreamer 不是"流程辅助工种"——它们是让整个体系成为**自我演进知识系统**的两台引擎：一台防止过去→现在的链路腐坏，一台保障现在→未来的链路畅通。少任何一台，业务知识每次任务结束就流失。

## 为什么不是一个大 agent

人类团队几十年前就搞清楚了一件事：写代码的人不适合验自己写的代码。不是因为能力不够，而是因为**上下文已经把判断力污染了**——你知道你为什么这么写，于是看不见别人会在哪里摔倒。

LLM 有同样的问题。上下文里留下的东西就是它的偏置——它推崇过的方案、它跳过的边界情况、它自己安慰自己"没事"的那几句话，都会在回看产出时替它辩护。让同一个 agent 既写又验，得到的是一套**自洽但漂移**的交付物。

解法不是让 agent 更聪明，是把工作拆成**几个独立视角**，用**契约**把它们串起来。详见 [doctrine/01 — 角色即剧班](./doctrine/01-roles-as-theater.md)。

## 7 个角色

| 角色 | 时间方向 | 人类对照 | 核心职责 | 硬边界 |
|------|---------|---------|---------|--------|
| [task-designer](./roles/task-designer.md) | 过去 → 现在 | Tech Lead | 读历史沉淀，拆任务、开契约、生成任务文档结构 | 不写代码 |
| [coder](./roles/coder.md) | 现在 | Engineer | 按契约施工、本地自验 | 不做 drive-by refactor、不碰契约外文件 |
| [evaluator](./roles/evaluator.md) | 现在 | QA | 复跑验收命令、独立判定是否达标 | 不改代码、不补契约 |
| [code-reviewer](./roles/code-reviewer.md) | 现在 | Staff Reviewer | 独立看 diff、风险分级 | 不改代码、不做 QA 的活 |
| [doc-refresher](./roles/doc-refresher.md) | **现在 → 未来（保鲜）** | Tech Writer | 代码 vs 文档一致性、SSOT 腐坏哨兵 | 不改文档，只标记过时项 |
| [dreamer](./roles/dreamer.md) | **现在 → 未来（沉淀）** | Knowledge Curator | 把流水式记忆酿造为长期业务知识 | 不发明、不覆盖原始条目 |
| [git-push](./roles/git-push.md) | 流程闸口 | Release Manager | 前置检查 → review → 提交 → 推送 | 不跳过钩子、不强推、不改写历史 |

## 5 条协议

- **[Dual SSOT](./doctrine/00-dual-ssot.md)**：代码和文档同为一等公民，文档在 git 里和代码同等纪律地维护——这是整套体系的前提
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
- 团队愿意把文档和代码一起放进 repo、一起 review、一起维护

**不适用**：

- 一次性小改、typo 修复
- 探索性 prototyping
- 单文件脚本
- 团队不接受双 SSOT 哲学（文档继续放外部工具里）——这种情况 doc-refresher 和 dreamer 会失去意义，整套体系会退化

详见 [doctrine/06 — 反模式](./doctrine/06-anti-patterns.md)。

## 起源

dev-roleplay 不是先有理论再写示例——它是从一个真实的、跨多仓库的服务迁移项目（Recipe 从 Director 服务迁到 Market 服务）中**沉淀出来的协作纪律**：

- 多个 coder 并行施工逼出了"双契约 + 文件范围互斥"
- 反复"用户点破同一类问题"的现象沉淀成了"3 轮升级 + question-the-problem-first 原则"
- 跨仓库切流的漂移代价催生了"task memory → SUMMARY → knowledge 三层流动"
- 文档与代码的不断脱节让"doc-refresher 哨兵"成为提交流水线的固定一站

dev-roleplay 把这些纪律脱敏后抽出来——保留骨架、砍掉项目特定词汇，做成可被任何项目 fork 的起点。

## 范围

dev-roleplay 面向支持独立 subagent 原语的 coding agent 工具（如 Claude Code）。
角色骨架与语言、框架、技术栈无关；引用的路径、命令、规范文档都由使用者在项目内自行定义。

具体接入方式见 [`integration/claude-code/`](./integration/claude-code/)。
真实项目下的填充形态见 [`examples/`](./examples/)。

## 目录

- [`roles/`](./roles/) — 7 个角色骨架
- [`doctrine/`](./doctrine/) — 设计思想与反模式（**先读 00 双 SSOT**）
- [`integration/`](./integration/) — 针对具体 harness 的接入指南
- [`examples/`](./examples/) — 真实项目下的填充样例

## License

[MIT](./LICENSE)
