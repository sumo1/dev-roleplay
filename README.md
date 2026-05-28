# dev-roleplay-harness

一套用于研发提效的多角色扮演与协作的Harness。

## 核心判断

当 AI 让 Coding 本身不再是最大瓶颈时，真正昂贵的东西变成了：

**业务知识如何获取、保存、更新、复用，并且不腐烂。**

作为业务开发，我们不只是写代码。我们还在持续理解业务、更新判断、积累约束。这些判断是经验池，也是工程价值所在。

过去，业务知识通常活在三个地方：

**沉淀在代码里**：代码能告诉你系统现在怎么跑，但说不出“为什么不那么跑”。被否决的方案、当年的权衡、字段命名背后的约束、这层抽象为什么没加，很多都活在代码的负空间里。

**沉淀在外部文档里**：Confluence、Notion、飞书里的文档脱离了工程纪律：不会被 review、不会被 diff、不会被回溯。代码变了，文档没人同步，过两个月就没人信了。

**沉淀在人脑里**：人会忘，人会走，交流成本会越来越贵。AI 时代，真正的瓶颈经常不是人和机器，而是人和人之间的认知同步。

这三个地方各有缺陷，单独任何一个都不能成为业务知识的 SSOT。

dev-roleplay-harness 的立场很简单：

**代码是工程仓库的一等公民。**  
**业务知识也应该是工程仓库的一等公民。**

两者一起进入 Git，一起 review，一起 diff，一起演进，构成软件工程的双 SSOT：

- 代码保存 **what**
- 文档保存 **why / why not / 当时的判断依据**

这套 Harness 的目标，是让 AI Coding 过程中产生的大量判断、决策、踩坑、否决方案，不再留在一次性会话上下文里，而是回流进仓库，成为下一次任务的起点。

Thinking is EVERYTHING.

## 什么时候用

适合：

- 多步骤、跨模块、错一点就很贵的工程任务
- 数据一致性、API 契约、服务迁移、长期演进的产品代码
- 需要多人或多 agent 反复回看的任务
- 业务判断比代码行数更重要的任务

不适合：

- 一次性小改
- 探索性 prototyping
- 单文件脚本
- 团队不接受文档进 Git，仍坚持把业务知识放在外部工具里

最后一种不是工具问题，是哲学不匹配。文档保鲜和知识沉淀这两台引擎没有落点，自进化回路就断了。

详细讨论见 [`doctrine/06-anti-patterns`](./doctrine/06-anti-patterns.md)。

## 设计哲学

### 1. 双 SSOT

代码和业务知识都应该被工程化管理。

代码告诉你系统如何运行，业务知识告诉你为什么这样运行、为什么没选另一条路、未来改动时哪些边界不能踩。

只保留代码，三个月后只能读到结果，读不到判断。

只写外部文档，文档会脱离代码腐坏。

所以 dev-roleplay-harness 要求两者同仓、同审、同提交。

### 2. 角色独立

一个 agent 自己写、自己验、自己审，容易得到一份自洽但漂移的交付物。

dev-roleplay-harness 把规划、施工、验收、审查、文档保鲜、知识沉淀拆成不同角色。每个角色只看产出物和契约，不继承上一个角色的心理辩护。

这不是为了复杂，而是为了减少视角污染。

### 3. 沉淀有边界

沉淀不是越多越好。

真正值得留下的知识，必须满足一个标准：

> 去掉这条 memory，未来某个决策点的分支会不会改变？

不会改变，就合并、归档或删除。  
会改变，才值得保留。

dev-roleplay-harness 反对“让 agent 总结一下最近学到什么”这种虚胖式沉淀。听起来正确但没有证据、不能改变未来判断的内容，不应该污染知识库。

### 4. 工程纪律约束知识

业务知识一旦进入仓库，就必须接受工程纪律：

- 可 review
- 可 diff
- 可回溯
- 可更新
- 可被下一个 agent 主动读取

这就是文档从“写给人看的说明”变成“工程系统的一部分”的关键。

## 角色清单

| 角色 | 核心职责 |
|---|---|
| [task-designer](./roles/task-designer.md) | 读历史沉淀，拆任务、开契约 |
| [coder](./roles/coder.md) | 按契约施工、本地自验 |
| [evaluator](./roles/evaluator.md) | 复跑验收命令，独立判定 |
| [code-reviewer](./roles/code-reviewer.md) | 看 diff、风险分级 |
| [doc-refresher](./roles/doc-refresher.md) | 保持代码和文档的信息一致性 |
| [dreamer](./roles/dreamer.md) | 把流水记忆酿造成长期知识 |
| [git-push](./roles/git-push.md) | 检查 → review → 提交 → 推送 |

## 五个关键特性

通过多角色协作，业务知识不再只是“写在某个地方”，而是具备五个工程性质：

- **持久化**：业务知识可记录、可追溯，不会随着一次会话结束而蒸发
- **自进化**：每次任务产生的新判断会回流进仓库，让下一次任务站在更高起点
- **自反思**：任务过程中的踩坑、否决方案和权衡会被总结沉淀，而不是只留下最终代码
- **保持新鲜**：`doc-refresher` 持续检查文档和代码的一致性，维持 Fresh SSOT
- **对抗式协作**：`coder`、`evaluator`、`code-reviewer` 独立判断，用角色隔离对抗漂移

这五个词是整套 Harness 的压缩版：防失忆、防腐坏、防自嗨。

## 运转模型

整套 Harness 聚焦在一条循环：

```text
   前置                  迭代                    沉淀回去
─────────────────────────────────────────────────────
 历史沉淀  ─→   当前任务的拆解 + 执行  ─→   入仓回流

knowledge/       task-designer              dreamer
engineering/     coder + evaluator             ↓
   ↑             code-reviewer           knowledge/
   │             doc-refresher  ←─保鲜   engineering/
   │                 ↓                       ↑
   └──────  下一次任务从新鲜起点出发  ─────────┘
```

### 前置知识

已经沉淀的项目背景、跨任务原则、编码规范、审查标准，放在：

- `docs/knowledge/`
- `docs/engineering/`
- `docs/review/`

开工前，task-designer 和 coder 应该主动读取这些内容。

doc-refresher 守这一段：每次代码变更后，检查哪些文档可能腐坏。

### 当前任务

每个任务有独立目录：

```text
docs/task/{task-id}/
```

其中保存：

- PRD / 背景调查
- progress
- 双契约 plan
- 按日 memory
- 被否决方案 archive
- 任务专项审查规则

coder 负责施工，evaluator 负责独立验收，code-reviewer 负责看 diff 和风险。三者上下文独立，互相不污染判断。

### 回流沉淀

任务过程中产生的判断、踩坑、原则和被否决方案，先进入 task memory。

任务结束后，dreamer 把流水记录蒸馏成：

- 任务级 SUMMARY
- 可复用的跨任务原则
- 需要上浮到 `knowledge/` 或 `engineering/` 的长期知识

doc-refresher 和 dreamer 是这套系统的两台引擎：

- doc-refresher 守外部一致性，防文档腐坏
- dreamer 守内部信息密度，防沉淀腐坏

## 三个硬协议

### 双契约

每个独立子任务的 plan 文件里，同时写：

- 施工契约：给 coder，看怎么做
- 验收契约：给 evaluator，看怎么判定完成

两段配对，避免“写的人说做完了，验的人不知道该验什么”。

详见 [`doctrine/03-dual-contract`](./doctrine/03-dual-contract.md)。

### 打回循环

evaluator 必须独立复跑验收命令，不认 coder 自报。

复跑结果和 coder 报告不一致，以复跑为准。

同一契约项连续多轮打回时，升级人类裁决，避免两个 agent 越修越乱。

详见 [`doctrine/04-refusal-loop`](./doctrine/04-refusal-loop.md)。

### 三层记忆

memory 分三层：

```text
按日沉淀 → 任务汇总 → 跨任务知识
```

写入、整理、上浮由不同角色负责，避免流水记录直接污染长期知识库。

详见 [`doctrine/05-memory-layering`](./doctrine/05-memory-layering.md)。

## Quick Start

### 1. 选择集成方式

dev-roleplay-harness 面向支持独立 subagent 原语的 Coding Agent 工具，例如 Claude Code。

角色骨架与语言、框架、技术栈无关；具体路径、命令、规范由使用者在自己项目内定义。

接入方式见 [`integration/claude-code/`](./integration/claude-code/)。

### 2. 复制角色骨架

把 [`roles/`](./roles/) 下的角色定义接入你的 agent 系统。

Claude Code 场景下，可以复制到：

```text
.claude/agents/
```

### 3. 建立项目知识目录

推荐在业务项目中建立：

```text
docs/
  ├── task/{task-id}/
  ├── knowledge/
  ├── engineering/
  └── review/
```

### 4. 从 task-designer 开始

给 task-designer 一个真实需求，让它先读历史沉淀，再生成任务目录、背景调查和双契约 plan。

### 5. 按角色闭环执行

推荐顺序：

```text
task-designer
  → coder
  → evaluator
  → code-reviewer
  → doc-refresher
  → dreamer
  → git-push
```

如果 evaluator 打回，回到 coder；如果连续多轮争议，升级人类裁决。

## 目录

- [`doctrine/`](./doctrine/) — 设计思想，先读 [`00-dual-ssot`](./doctrine/00-dual-ssot.md)
- [`roles/`](./roles/) — 7 个角色骨架
- [`integration/`](./integration/) — 针对具体 Harness 的接入指南
- [`examples/`](./examples/) — 真实项目下的填充样例

## 示例

脱敏后的完整任务示例在 [`examples/`](./examples/)。

它展示了一个任务从 background 探索、双契约 plan、按日 memory，到 dreamer 蒸馏 SUMMARY，再上浮一条原则到 `knowledge/principles/` 的全过程。

建议阅读顺序见 [`examples/README.md`](./examples/README.md)。

## 这套东西真正交付什么

三个月后回看一段代码时，仓库里应该能直接读到：

- **what**：代码怎么写的
- **why**：当时为什么这么选
- **why not**：哪些方案被否决，证据是什么
- **同类怎么做**：同模块过去的判断
- **跨任务原则**：哪些约束不只对这次有效

不需要找当年的人问，不需要翻聊天记录，不需要拼凑 commit message。

业务的 why 在代码隔壁，和代码一起保持新鲜。

这才是 dev-roleplay-harness 真正想交付的东西。

## 起源

这套骨架不是先有理论再写示例，而是从真实的跨多仓库服务迁移项目里长出来的。

多 coder 并行施工逼出了双契约和文件范围互斥；反复看到“用户点破同一类问题”沉淀成了“先质疑问题是否成立”的原则；跨仓库切流的漂移代价催生了三层记忆流动；文档与代码不断脱节，让“提交前扫一遍文档新鲜度”成了固定流程。

理论是后来总结的，骨架是从事故边缘活下来的。

## License

[MIT](./LICENSE)
