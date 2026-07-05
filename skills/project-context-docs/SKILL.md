---
name: project-context-docs
description: "Use when work needs project context documents: reading PROJECT.md, CONTEXT.md, CONTEXT-MAP.md, docs/adr, docs/agents before work, or preserving durable project context after design, implementation, debugging, architecture decisions, domain vocabulary, OpenSpec memory promotion, or agent collaboration rules."
---

# Project Context Docs

管理项目长期上下文文档的读取与沉淀。这个 skill 不只在初始化时使用；任何设计、实现、排障、架构调整、OpenSpec 归档或 PR 收尾，都可以触发它。

核心原则：**先读已有上下文，后按需沉淀新上下文；不预先创建空文档。**

## Scope

本 skill 负责：

- 开工前判断并读取相关项目上下文文档。
- 判断当前任务产生的信息是否值得长期保存。
- 将长期上下文写入合适位置。
- 保持 `PROJECT.md`、`CONTEXT.md`、ADR、`docs/agents/*.md` 的职责边界。

本 skill 不负责：

- 不替代 OpenSpec change；垂直切片仍放在 `openspec/changes/`。
- 不管理 GitHub Issue / PR；相关协作由 `github-project-flow` 负责。
- 不批量生成项目文档骨架。
- 不在每个目录下预创建 ADR。
- 不把实现细节、临时 scratch 或切片 spec 塞进 `CONTEXT.md`。

## 1. Read Before Work

开始涉及项目语义、架构、历史决策或长期规则的任务前，先按需读取：

- `PROJECT.md`：项目目标、边界、非目标、长期约束。
- `CONTEXT.md`：领域词汇、业务概念、统一语言。
- `CONTEXT-MAP.md`：多上下文项目中各 context 的入口。
- `docs/adr/`：系统级架构决策。
- `src/<context>/CONTEXT.md` 和 `src/<context>/docs/adr/`：多上下文项目中的局部领域词汇和决策。
- `docs/agents/*.md`：Agent 协作与长期消费规则。
- 当前 OpenSpec change：本次切片内的 proposal、design、tasks、memory 或验证记录。

读取规则：

- 只读与当前任务相关的上下文，不做全量文档巡检。
- 文件不存在时安静继续；不要把缺失本身当成问题。
- 如果 `CONTEXT-MAP.md` 存在，优先按 map 找相关 context，再读局部 `CONTEXT.md` 和 ADR。
- 如果当前任务明显会违反已有 ADR，先显式指出冲突，再继续讨论是否重开决策。

完成标准：能说明当前任务依赖了哪些上下文、哪些上下文不存在但不影响继续。

## 2. Decide What To Preserve

只有满足“未来读者或未来 Agent 会再次需要”的信息，才提升为长期上下文。

保存位置按信息类型决定：

| 信息类型 | 保存位置 |
|---|---|
| 项目目标、边界、非目标、长期约束 | `PROJECT.md` |
| 领域词汇、业务概念、统一语言 | `CONTEXT.md` 或 context 局部 `CONTEXT.md` |
| 难以回退、有真实取舍、未来读者会疑惑的技术决策 | `docs/adr/NNNN-slug.md` 或 context 局部 ADR |
| Agent 长期协作规则、工具消费规则 | `docs/agents/*.md` |
| 只对当前切片有用的发现、验证记录、实现备注 | 当前 OpenSpec change 或 `.memory/` |

不要保存：

- 已经能从代码直接看出的普通实现细节。
- 临时调试过程。
- 没有稳定下来的猜测。
- 只服务当前对话、未来不会复用的信息。

完成标准：每条待保存信息都有明确归属；不值得保存的信息留在当前工作产物中。

## 3. Write Lazily

写入规则：

- 不预先创建空文档。
- 只有当前任务产生了明确需要保存的上下文时，才创建或更新对应文件。
- 修改已有文件时保留原结构，只追加或更新相关小节。
- 根 `AGENTS.md` 只放入口和高优先级规则；稳定细节放进 `docs/agents/` 或 skill。
- 默认使用简体中文；固定技术术语保留英文。

各类文档约束：

- `PROJECT.md` 只记录项目级背景、目标、边界和长期约束，不记录单个切片的详细方案。
- `CONTEXT.md` 只记录领域词汇和业务概念，不记录实现方案或技术决策。
- ADR 只记录重要技术决策，不记录普通任务笔记。
- `docs/agents/*.md` 只记录 Agent 需要反复遵守的消费规则，不记录一次性执行日志。

完成标准：新上下文已写入最小合适位置；没有创建空文档或无关文档。

## ADR Gate

只有同时满足以下条件，才写 ADR：

- 决策难以回退，或未来回退成本高。
- 存在真实取舍，而不是显而易见的实现选择。
- 未来读者离开当前对话后会疑惑“为什么这么做”。

ADR 文件命名：

```txt
docs/adr/NNNN-slug.md
```

编号规则：扫描目标 ADR 目录中已有最大编号，加一。不要复用编号。

## CONTEXT Gate

当一个领域词汇、业务概念或边界语言已经稳定，并且后续 issue、PR、设计或测试会反复使用它时，写入 `CONTEXT.md`。

建议格式：

```md
**术语**：一两句话定义它在本项目里的含义。
_Avoid_：不要使用的近义词或误导性叫法。
```

如果项目是多 context，优先写入最贴近该领域的局部 `CONTEXT.md`；只有跨 context 的概念才写根目录 `CONTEXT.md`。

## Promotion Check

在 OpenSpec change 归档、PR 收尾或重要实现完成后，做一次轻量提升检查：

- 是否产生了新的稳定领域词汇？如果是，更新 `CONTEXT.md`。
- 是否做了重要技术决策？如果是，补 ADR。
- 是否澄清了项目目标、边界或非目标？如果是，更新 `PROJECT.md`。
- 是否形成了新的 Agent 长期协作规则？如果是，更新 `docs/agents/*.md` 或相关 skill。

如果答案都是否，明确记录“不需要提升”，不要为了完成流程而写文档。
