# behavior-validator 调研报告

## 机制详解

### 1. 定位与触发方式

`behavior-validator` 是源码盲（source-blind）的行为验收 skill。它只判断运行中的 Web、CLI、API 或生成物是否满足预先写好的行为契约。它与 `autoreview` 分工：后者看变更包，前者看用户可观察行为。

它没有 slash command，也没有规定必须绑定某个开发阶段。触发依赖 SKILL frontmatter 的 description，被 agent 或编排器在需要“对预写契约做源码盲行为验证”时调用。修复发现后，只重跑受影响的契约条款及邻近回归探测。

来源：

- https://github.com/openclaw/agent-skills/blob/main/skills/behavior-validator/SKILL.md

### 2. 输入契约

验收前必须先读 behavior contract。若没有现成契约，validator 会根据用户请求临时写一份短契约。官方模板要求包含：

1. 用户可见目标。
2. 目标类型与访问入口，如 URL、命令、endpoint 或 artifact 路径。
3. 允许使用的 fixture，以及凭据来自哪个 secret tool 或环境变量；只写变量名，不写值。
4. 真实用户任务与对应的可观察结果，包括错误行为。
5. anti-cheat probes、证据要求和明确排除项。

契约完成标准是：每个用户任务和 anti-cheat probe 都有预期可观察结果及证据类型。运行时输入可包括目标 URL、CLI 命令、API endpoint、fixture、凭据来源或生成物路径。

这是一份“可执行行为契约”，不只是验收标准列表。不过，“缺少契约时由 validator 自己从用户请求生成”会让验收者同时解释 oracle，隔离程度弱于使用预先冻结的独立契约。

来源：

- https://github.com/openclaw/agent-skills/blob/main/skills/behavior-validator/SKILL.md
- https://github.com/openclaw/agent-skills/blob/main/skills/behavior-validator/references/contract-template.md

### 3. 隔离方式

它要求 validator 始终保持源码盲，只通过用户或运维人员可见的表面操作：浏览器、CLI、API、生成文件、公开日志、截图、accessibility tree 或文档化运行输出。

推荐做法是创建权限为 `700` 的随机临时目录。目录内只保留：

- behavior contract；
- 允许的 fixture；
- 已脱敏证据。

凭据通过批准的 secret tool 或精确指定的环境变量注入，不复制到目录、报告、截图或日志。若应用只能从源码 checkout 启动，应在另一终端启动，validator 不读取 checkout。固定共享目录也被禁止，避免不同验收任务互相污染。

发现类似实现细节的证据时，应视为上下文污染。若继续验收必须读取源码，则停止并报告 `blocked_source_required`。

需要注意：SKILL 写的是“Prefer a source-blind workspace”，并未要求必须创建一个全新的零上下文 subagent。因此，它提供了较完整的工作区隔离操作，但没有从机制上保证会话上下文本身是干净的。

来源：

- https://github.com/openclaw/agent-skills/blob/main/skills/behavior-validator/SKILL.md

### 4. 反作弊探测

它把 anti-cheat 明确列为标准步骤，而不是只跑 happy path：

- 改变 fixture 或输入，确认输出随真实数据变化；
- refresh、retry 或 reopen，验证持久化或预期重置；
- 测 empty、invalid 与 boundary input；
- 检查生成物的实际内容；
- 确认按钮或命令确实完成工作，而非只显示成功文案。

以下情况应判 fail：行为违反契约、用户任务无法完成、状态是 fake/static，或证据不足以支持 pass。缺 runtime、凭据、fixture、网络或工具才判 blocked。只有契约明确排除，或依赖用户产品决策，才可判 out of scope。审美、代码质量和实现风格不属于本 skill。

来源：

- https://github.com/openclaw/agent-skills/blob/main/skills/behavior-validator/SKILL.md
- https://github.com/openclaw/agent-skills/blob/main/skills/behavior-validator/references/contract-template.md

### 5. 报告格式

最终报告必须包含：

- 实际操作的 target；
- 使用的契约文件或 inline contract；
- pass、fail、blocked、out-of-scope 汇总；
- 按契约条款列出的 finding、复现步骤和证据；
- 已运行的 anti-cheat probes；
- 剩余 blocker。

当编排器需要机器可读结果时，官方 JSON schema 使用这些主要字段：

- `overall_behavior`：`satisfies_contract`、`violates_contract` 或 `blocked`；
- `overall_confidence`；
- `target`；
- `checks[]`：契约条款、状态、失败严重度、证据、复现步骤、置信度；
- `anti_cheat_probes[]`；
- `blockers[]`。

每个相关契约条款必须被归类为 pass、fail、blocked 或 out of scope，不能用模糊总评跳过条款。所有证据必须去除凭据、token、cookie、私人数据和无关日志。

来源：

- https://github.com/openclaw/agent-skills/blob/main/skills/behavior-validator/SKILL.md
- https://github.com/openclaw/agent-skills/blob/main/skills/behavior-validator/references/report-schema.md

## 与旧 workflow 的对比

旧版规则位于 `ai-solo-workflow` 的 apply 第 10 步：非微小变更在 Final Gate 后必须派一个无上下文验收代理。该代理不能读源码、`proposal.md`、`design.md`、`tasks.md`、测试实现或内部实现说明。它只能拿到用户原始任务、运行入口、必要凭据、测试数据、明确成功标准和禁止事项。Web 项目必须从真实用户入口使用 `agent-browser`。旧版 GitHub skill 只明确要求把黑盒验收的 `passed/failed/path` 写入 PR，并在通过或失败时更新 Issue。

`_backup/docs/agents/` 三份文档分别约束 GitHub、OpenSpec 和 AGENTS.md 边界，没有补充黑盒验收规则。`docs/workflow-decisions.md` 的 D6 只说明 Multica 自报完成不可信，因此质量仍需自建黑盒验收；具体协议仍以上述旧 schema 为准。

旧版来源：

- https://github.com/lushangkan/lushangkan-workflow/blob/main/_backup/schemas/ai-solo-workflow/schema.yaml#L245-L270
- https://github.com/lushangkan/lushangkan-workflow/blob/main/_backup/skills/github-project-flow/SKILL.md#L148-L221
- https://github.com/lushangkan/lushangkan-workflow/blob/main/docs/workflow-decisions.md#L34-L40
- https://github.com/lushangkan/lushangkan-workflow/tree/main/_backup/docs/agents

| 对比项 | 用户旧 workflow | `behavior-validator` | 多了什么 / 少了什么 |
|---|---|---|---|
| 上下文隔离 | **必须派无上下文验收代理**；以新 agent 的会话边界隔离实现上下文 | 要求 source-blind；推荐随机私有临时目录，只放契约、fixture、脱敏证据；未强制新 agent | **多了**可执行的工作区、secret、证据隔离规范及污染处理。**少了**“全新零上下文 agent”这一硬保证；已有会话即使不再读源码，也可能携带实现知识。 |
| 输入 | 只能获得**用户原始任务**、运行入口、必要凭据、测试数据、明确成功标准、禁止事项 | 读取预写 behavior contract；包含目标、入口、任务、预期行为、anti-cheat、证据、排除项。无契约时可自行从用户请求生成 | **多了**统一、可执行的契约结构、证据要求和 out-of-scope。**少了**“必须直接以用户原始任务为 oracle”的限制；自行改写契约可能引入解释漂移。 |
| 禁读源码的强制力 | 强制禁止源码，并点名禁止 proposal、design、tasks、测试实现和内部说明 | 禁止源码、diff、tests、git history、实现说明、build internals、review bundles；若源码成为继续条件，停止并报 `blocked_source_required` | **多了**更广的禁读清单、污染判定和明确停止状态。**少了**独立 agent 的结构性强制；临时工作区也只是“prefer”。两者组合才最强。 |
| 报告形式 | 核心 schema 未规定逐条报告；GitHub 层只固定 `passed/failed/path`，并同步 Issue/PR | 必须逐条归类；记录 target、契约、复现步骤、证据、anti-cheat 和 blocker；可输出带置信度的 JSON | **明显多了**可审计、可机器消费的报告协议。**少了**旧流程自带的 Issue/PR 状态同步；该 skill 自身只产报告，不负责项目流转。 |
| 触发时机 | 确定性 gate：**非微小变更**在 Final Gate 后、GitHub Tracking 前必须执行 | 原子能力；由 description 或外部编排器按需调用。修复后支持局部重验及邻近回归探测 | **多了**可独立复用和低成本局部重验。**少了**“所有非微小变更必跑”的生命周期保证，也没有定义微小/非微小边界。 |
| 反作弊 | 旧规则只要求从真实入口完成任务，没有独立 probe 协议 | 内建数据变化、刷新/重试、异常与边界输入、持久化、生成物和假成功探测 | **新增**系统化 anti-cheat，能发现静态页面、假按钮和只写成功文案的伪实现。 |
| Web 入口 | 明确要求使用 `agent-browser` 从真实用户入口操作 | 允许 browser 与 accessibility tree，但不绑定具体浏览器工具 | **多了**跨 harness 可移植性。**少了**旧环境中明确的工具选择；需由 adapter 或编排器指定实际浏览器。 |

## 结论

`behavior-validator` 的执行协议比旧版完整：它补上了行为契约模板、反作弊、证据脱敏、逐条状态和机器可读报告。但它不能单独保留旧版最重要的两项结构性保证：**全新零上下文 agent**，以及**所有非微小变更在固定阶段强制验收**。若要迁移，应由外层 workflow 强制用新 agent 调用它，并传入从用户原始需求冻结的契约；不要允许 validator 在正式验收时自行改写 oracle。

能否完整替代旧版：**不能直接完整替代；加上“新零上下文 agent + 原始需求冻结契约 + 非微小变更强制触发”的外层编排后，可以替代且能力更强。**
