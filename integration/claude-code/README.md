# Claude Code 集成

把 [`../../roles/`](../../roles/) 下的角色接入 Claude Code 的做法。

## 目录布局

Claude Code 在项目根下读取 `.claude/agents/` 作为 subagent 定义目录，每个 `.md` 文件对应一个可被 spawn 的 subagent。

推荐布局：

```
your-project/
├── .claude/
│   └── agents/
│       ├── task-designer.md
│       ├── coder.md
│       ├── evaluator.md
│       ├── code-reviewer.md
│       ├── doc-refresher.md
│       ├── dreamer.md
│       └── git-push.md
├── agents/                      # 可选：角色内容的 SSOT
│   └── {role-name}/
│       └── {role-name}.md
└── docs/
    ├── task/{task-id}/
    ├── knowledge/
    └── engineering/
```

**两种组织方式**：

- **简单方式**：直接把 `roles/*.md` 的内容复制到 `.claude/agents/` 下，每个文件顶部加 Claude Code frontmatter。
- **SSOT 方式**：把完整角色内容保存在 `agents/{role-name}/` 下，`.claude/agents/` 下的文件只做**薄引用**（十来行 frontmatter + 一句"参见 `agents/{role-name}/`"）。这样避免跨目录维护同一份内容。

## Claude Code Frontmatter

Claude Code 要求每个 `.claude/agents/*.md` 文件以 YAML frontmatter 开头：

```yaml
---
name: coder
description: 子任务实现者。收到 task-designer 预先规划好的单个独立子任务，读取工程规范与任务专项规则后实现代码，并跑本地验证自证。
tools: ...
---
```

关键字段：

- `name`：角色名，主会话通过它 spawn subagent
- `description`：**非常重要**——主会话通过这段决定什么时候 spawn 这个 subagent。写清楚触发条件和适用场景
- `tools`：白名单，限制这个 subagent 可用的工具

## tools 白名单建议

按角色职责分发工具，最小权限原则：

| 角色 | 推荐白名单 |
|------|-----------|
| task-designer | 全部工具（需要读代码、读规范、写 plan 文件） |
| coder | 代码编辑 + 读取 + 执行命令（跑测试）+ grep/find |
| evaluator | 读取 + 执行命令。**不给代码编辑工具** |
| code-reviewer | 读取 + grep/find。**不给任何写入工具和执行工具** |
| doc-refresher | 读取 + grep/find。**不给任何写入工具** |
| dreamer | 读取 + 写入（只写 memory / knowledge 相关路径）|
| git-push | Bash（执行 git 和前置检查） + 子 agent 调用权 |

通过 `tools:` 字段硬性剥夺写权限，比靠 prompt 里的"禁止"更可靠。prompt 约束可能被上下文冲散，工具白名单不会。

## 主会话如何 spawn subagent

Claude Code 主会话使用 `Agent` 工具 spawn subagent，格式类似：

```
Agent({
  subagent_type: "coder",
  description: "实现 step1-data-layer 子任务",
  prompt: "任务 ID: 260512-user-auth\n子任务: plan/step1-data-layer.md\n请按施工契约开工。"
})
```

关键点：

- `subagent_type` 对应 `.claude/agents/` 下文件里的 `name` 字段
- `prompt` 必须**自包含**——subagent 看不到主会话上下文，所有它需要的信息都要在 prompt 里
- subagent 产出返回给主会话后，subagent 上下文销毁——这是独立视角的技术保证

## 主调度者的职责

主会话充当"调度者"角色，负责：

### 1. 角色编排

按任务阶段依次 spawn 角色：

```
task-designer  →  coder  →  evaluator  →  code-reviewer  →  doc-refresher  →  git-push
                              ↑_________|（打回循环）
```

### 2. 轮次计数维护

打回循环的"连续 3 轮"计数由主会话维护。每次 spawn evaluator 时，prompt 里传入"本轮是第 N 轮"：

```
Agent({
  subagent_type: "evaluator",
  prompt: "任务 ID: ...\n子任务: ...\n本轮轮次: 2\n上轮 evaluator 打回原因: ..."
})
```

到第 3 轮仍未通过时，主会话直接升级为人类裁决，不再 spawn coder。

### 3. 争议识别

evaluator 或 coder 的报告里出现"【争议】"、"⚠️ 契约问题"、"🛑 需调用方裁决"等标记时，主会话不再 spawn 下一个角色，停下来等人类决策。

### 4. 上下文传递

plan 文件、memory 条目、工程规范等**不在 subagent 的初始上下文里**，必须：

- 在 prompt 里给出路径（subagent 自己读）
- 或者把内容直接贴进 prompt（小文件场景）

前者更常见，因为 Claude Code 的 subagent 有独立的文件读取能力。

## 推荐的 description 字段写法

`description` 字段决定主会话何时 spawn 这个 subagent。写法建议：

```yaml
description: 任务规划者。接收需求后，理解工程上下文，拆解为可执行的步骤计划，并在 docs/task/ 下生成标准化的任务目录结构。 只做规划和目录生成，不直接编写业务代码。
```

关键要素：

- **定位**（一句话说角色是什么）
- **触发场景**（什么情况下应该 spawn 它）
- **边界**（不做什么——防止主会话误用）

## 角色与 skill 的关系

Claude Code 还有 `.claude/skills/` 作为用户可主动触发的 skill 定义。两者的分工：

- **Subagent**（`.claude/agents/`）：被主会话自动 spawn，完成封闭的子任务
- **Skill**（`.claude/skills/`）：用户主动触发（通过 `/{skill-name}`），通常是一条完整的工作流

dev-roleplay 的 7 个角色里，**git-push** 最适合同时作为 skill 存在——用户输入 "提交" / "/git-push" 时触发，内部再 spawn code-reviewer + doc-refresher 两个 subagent。

其他 6 个角色主要作为 subagent 存在，由主会话按需调度。

## 常见坑

### 1. subagent 上下文里没有主会话历史

subagent 被 spawn 时看到的是**独立的空白上下文**，不知道主会话之前聊了什么。所有必要信息必须通过 prompt 显式传递。

### 2. 工具白名单不到位时 subagent 会"偷跑"

如果 evaluator 的 `tools:` 里留了代码编辑工具，它可能在发现"一个明显的笔误"时忍不住直接改——这就违反了独立视角。硬性剥夺权限比 prompt 约束更可靠。

### 3. description 太模糊导致不被 spawn

`description: "审查代码"` 会让主会话不知道什么时候该 spawn 这个 subagent。要写得足够具体，让触发条件明确。

### 4. 多轮打回时 coder 会带着上轮污染

coder 被第二轮 spawn 时**上下文是全新的**，它不知道上轮发生了什么。主会话需要在 prompt 里明确告知：

```
本轮是第 2 轮施工。
上轮 evaluator 打回原因：{原文}
请只修打回的具体项，不要扩大改动范围。
```

## 版本兼容性

本集成指南基于 Claude Code 的 subagent 机制编写。Claude Code 的 subagent 行为和 frontmatter 字段可能随版本演进——使用者在引入时请以当前版本的官方文档为准，以下内容可能需要适配：

- `tools:` 字段的可选值
- subagent 与主会话之间的上下文传递机制
- `Agent` / `Task` 工具的 API 形态
