建议把 **OpenSpec 降级为可替换的存储适配器**：3 个 user-invoked skill 负责编排，model-invoked 原子 skill 持有质量规则，artifact 只记录变更状态和审查证据。

**摘要**

- 固定的 `proposal → ... → apply` 链改成**条件驱动流程**。小修复可以跳过 specs 和 design；行为变化才要求 specs；架构变化才要求 design。
- 8 个 artifact 收敛为一个工具无关的 **Change Record**。Change Record 只包含 3 类信息：**意图、计划、证据**。OpenSpec 负责把 Change Record 写成现有文件。
- 5 道 gate 保留审查标准，但拆开“何时审查、怎么审查、是否放行”。编排 skill 决定是否放行，子代理只返回审查结果。
- 现有 5 个 skill 按调用者重新归类。`project-init-workflow` 和 `github-project-flow` 更适合 user-invoked；Git、worktree、上下文维护更适合 model-invoked。
- 迁移时先加适配层，再改流程。不要一次删除 OpenSpec，也不要同时维护两份可编辑的状态。

## 1. 目标架构：编排、纪律、存储分成三层

你现在的耦合来自一个所有权问题：OpenSpec schema 同时决定了**保存什么、先做什么、何时审查**。

目标架构把三个责任拆开。

| 层 | 推荐内容 | 负责什么 | 不负责什么 |
|---|---|---|---|
| user-invoked 编排层 | `start-change`、`implement-change`、`ship-change` | 根据目标、风险和当前状态选择下一步 | 不保存 OpenSpec 路径和 schema 细节 |
| model-invoked 纪律层 | scope、spec、design、planning、execution、verification | 规定每类工作怎样才算合格 | 不决定整个流程，不自动进入下一阶段 |
| Change Record 与适配器层 | Change Record、OpenSpec adapter、Markdown adapter | 读写状态，转换文件格式 | 不决定是否需要 specs、design 或 gate |

### Change Record 只保留三个概念

**Change Record** 是工具无关的变更数据接口。Change Record 不要求新增一个大文件，也不要求替换现有 Markdown。

| 部分 | 保存内容 |
|---|---|
| 意图 | 目标、范围、不做什么、约束 |
| 计划 | 验收条件、可选 specs、可选 design、任务、执行方式 |
| 证据 | 审查结果、测试结果、提交和 PR 链接 |

OpenSpec adapter 可以继续把这些内容写进现有 artifact。以后换成普通 Markdown，只需要更换 adapter。

### 推荐的 user-invoked skill

| skill | 用户什么时候调用 | 编排责任 |
|---|---|---|
| `start-change` | 开始一个变更 | 收集意图，判断是否需要 specs、design 和提前审查 |
| `implement-change` | 开始或继续实现 | 生成或读取任务，逐项执行，持续验证 |
| `ship-change` | 准备提交或创建 PR | 运行最终验证，检查证据，调用 Git 和 GitHub 能力 |

三个 skill 都可以调用 model-invoked skill。三个 skill 不直接依赖 `proposal-review` 之类的固定前置 artifact。

### 推荐的 model-invoked skill

按职责分组，避免创建一长串阶段 skill。

**定义变更**

- `change-scoping`：约束目标、范围和不做什么。
- `behavior-specification`：把行为变化写成可验证的要求。
- `design-decisions`：记录关键技术选择、代价和风险。

**实施变更**

- `implementation-planning`：把要求拆成可执行任务。
- `task-execution`：一次实现一个任务，并保存局部验证结果。
- `change-verification`：对照验收条件检查最终结果。

**审查协议**

- `review-reporting`：统一子代理的审查输出格式。

每个 model-invoked skill 都应满足三个判据：

1. 删除 OpenSpec 路径和 artifact 名称后，skill 仍然完整。
2. 单独调用 skill 时，skill 能产生有用结果。
3. skill 不包含“完成后进入下一阶段”之类的编排指令。

### 推荐的仓库边界

| 位置 | 内容 |
|---|---|
| `skills/user/` | 三个 user-invoked 编排 skill |
| `skills/model/` | scope、spec、design、planning、execution、verification |
| `workflow/policies/` | 条件 gate 和完成条件 |
| `contracts/change-record/` | 工具无关的字段定义 |
| `adapters/` | OpenSpec、Markdown、GitHub 等格式或平台转换 |

## 2. 把 8 个 artifact 改成状态、可选文档和操作

不要把 8 个 artifact 机械地改成 8 个 skill。artifact 是保存结果的文件；skill 是模型工作时必须遵守的规则。两者不需要一一对应。

### 核心 artifact 的处理方式

| 现有 artifact | 推荐处理 | 原因 |
|---|---|---|
| `proposal` | 映射到 Change Record 的“意图” | 变更目标仍然需要长期保存 |
| `proposal-review` | 改成附在意图上的审查证据 | 审查结果不是流程阶段 |
| `tasks` | 保留为计划中的任务列表 | 任务是可执行单位 |
| `execution-plan` | 合并进任务列表 | 顺序、依赖、检查方法可以放在每个任务中 |
| `apply` | 改成操作，不再作为 artifact | apply 会产生代码、测试结果和执行证据 |

### 条件 artifact 的处理方式

| 现有 artifact | 推荐处理 | 什么时候存在 |
|---|---|---|
| `specs` | 保留为可选行为契约 | API、用户行为或业务规则发生变化时 |
| `design` | 保留为可选设计记录 | 跨模块、数据、安全或依赖选择需要解释时 |
| `implementation-mode` | 改成计划中的一个字段 | 执行方式确实会影响任务拆分时 |

`implementation-mode` 单独成为 artifact 的收益通常很低。调用者只需要知道执行方式，以及执行方式怎样影响任务和验证。

### 静态 `requires` 改成三类规则

| 规则类型 | 例子 |
|---|---|
| 硬前置条件 | 执行任务前，必须有任务目标和验收方法 |
| 条件要求 | 如果外部行为变化，那么必须有 specs |
| 完成条件 | 关闭变更前，必须有验证证据 |

这样仍然有纪律，但不会强迫每个变更经过相同的 8 个文件。

### 三种变更会走不同路径

| 变更类型 | 实际路径 |
|---|---|
| 小型内部修复 | 意图 → 简短计划 → 实现 → 验证 |
| 外部行为变化 | 意图 → specs 和审查 → 计划 → 实现 → 验证 |
| 架构或数据变化 | 意图 → design 和审查 → 计划 → 实现 → 验证 |

`start-change` 负责选择路径。OpenSpec adapter 只负责保存选择结果。

## 3. Gate 与现有 skill 的原子化方式

### Gate 拆成三个组成部分

每道 gate 都使用相同结构：

1. **Policy 决定是否触发。**
2. **子代理执行审查。**
3. **编排 skill 根据结果决定是否继续。**

子代理不拥有 gate。子代理只返回结构化结果。

推荐让审查结果包含以下内容：

| 字段 | 含义 |
|---|---|
| `verdict` | `pass`、`revise` 或 `block` |
| `findings` | 问题位置、原因和修改方法 |
| `reviewed_revision` | 审查的是哪个文件版本或内容版本 |
| `evidence` | 支持判断的文本、测试或 diff |

`reviewed_revision` 很重要。artifact 修改后，旧的 `pass` 不能继续作为有效 gate。

### 五类条件 gate

你可以先保留现有五类审查标准，只改变触发方式。下面是一套可直接落地的触发模型。

| if 条件 | then 动作 |
|---|---|
| 目标、范围或不做什么不清楚 | 运行 scope 审查；通过前不进入实现 |
| API、用户行为或业务规则变化 | 生成并审查 specs |
| 跨模块、数据、安全或依赖选择发生变化 | 生成并审查 design |
| 任务缺少验收方法或执行顺序 | 运行计划审查 |
| 实现准备提交或创建 PR | 对照验收条件运行最终验证 |

这种设计保留了五道质量检查，同时允许简单变更跳过不适用的审查。

### 一个 gate 怎样组合原子 skill

以 specs 审查为例：

- `behavior-specification` 提供领域规则，例如行为是否可观察、验收条件是否可测试。
- `review-reporting` 提供统一的审查结果格式。
- 子代理组合两个 skill，输出 `verdict` 和 `findings`。
- `start-change` 或 `implement-change` 决定是否允许继续。

这样不需要一个拥有完整流程的 `proposal-to-specs-reviewer`。

### 每个原子 skill 的固定结构

| 部分 | 必须回答的问题 |
|---|---|
| 触发条件 | 模型在什么情况下加载 skill |
| 输入 | 运行 skill 最少需要什么 |
| 纪律 | 模型必须检查或执行什么 |
| 输出 | 调用者会拿到什么 |
| 停止条件 | skill 明确不做什么，什么时候返回调用者 |

例如，`behavior-specification` 可以规定：

- 外部行为发生变化时加载。
- 输入是变更意图和现有行为。
- 每条要求必须可观察、可验证。
- 输出行为要求和未解决问题。
- 不创建任务，不选择实现方式，不修改 Git 状态。

### 现有 5 个 skill 的推荐归类

| 现有 skill | 推荐归类 | 推荐调整 |
|---|---|---|
| `git-master` | model-invoked | 如果 commit、rebase、历史重写和故障恢复有不同触发条件，拆成独立纪律；如果内容只是一套统一 Git 安全规则，可以保留 |
| `using-git-worktrees` | model-invoked | skill 负责何时隔离工作区和安全规则；创建、删除 worktree 交给脚本或工具 |
| `github-project-flow` | user-invoked | 保留 Issue、分支、PR 的流程编排；把命名和链接规则提取为 model-invoked 纪律 |
| `project-context-docs` | model-invoked | 在架构决定、命令或约束变化时触发；不要让 skill 主动启动完整文档流程 |
| `project-init-workflow` | user-invoked | 继续负责项目初始化编排，并按需调用 Git、worktree 和上下文 skill |

`git-master` 不需要为了追求文件数量而强制拆分。拆分判据是**触发条件是否独立**。如果模型会在完全不同的情境下加载不同部分，就应该拆分。

## 4. 按五步迁移，并用典型变更验证

开始迁移前，准备这些材料：

- OpenSpec schema 和 8 个 artifact 模板。
- 5 道 gate 的子代理 prompt。
- 现有 5 个 `SKILL.md`。
- 至少 3 个历史变更样本：小修复、行为变化、架构变化。

当前状态是固定 OpenSpec 流水线。下一步先建立边界，不要先删 artifact。

### 第 1 步：列出每个 artifact 真正提供的信息

为 8 个 artifact 分别记录：

- 谁生成。
- 谁读取。
- 哪些字段属于长期状态。
- 哪些内容只是阶段控制。
- 哪些内容直接引用 OpenSpec 路径或命令。

**完成标准：**每段信息都能归入意图、计划或证据。无法归类的流程控制信息交给 user-invoked 编排层。

### 第 2 步：定义 Change Record 接口和 OpenSpec adapter

先让 OpenSpec 继续作为唯一存储来源。

OpenSpec adapter 负责：

- 从现有 8 个 artifact 读取 Change Record。
- 把 Change Record 的更新写回现有 artifact。
- 验证 OpenSpec 文件格式。
- 在兼容期生成现有状态。

编排 skill 只读写 Change Record，不直接访问 `proposal.md`、`tasks.md` 之类的路径。

**完成标准：**把 OpenSpec adapter 替换成测试 adapter 后，编排逻辑仍然能运行。

### 第 3 步：从 gate prompt 提取 model-invoked 纪律

先提取审查标准，不改 gate 数量。

推荐顺序：

1. 提取 `review-reporting`，统一所有子代理输出。
2. 提取 specs、design 和 planning 的质量规则。
3. 删除 prompt 中的固定前后阶段和 OpenSpec 路径。
4. 给每份审查结果绑定 `reviewed_revision`。

**完成标准：**每个 gate 都能针对手工提供的输入单独运行。

### 第 4 步：加入三个 user-invoked 编排 skill

先实现 `start-change`，因为路径选择发生在最前面。

`start-change` 应按这个顺序工作：

1. 读取或创建变更意图。
2. 判断行为、架构和风险影响。
3. 选择需要的可选文档和 gate。
4. 调用对应 model-invoked skill。
5. 把结果交给 OpenSpec adapter 保存。

然后实现 `implement-change` 和 `ship-change`。

**完成标准：**小修复可以在没有 specs 和 design 的情况下完成，同时仍然有验收和验证证据。

### 第 5 步：删除静态链，并增加 Markdown adapter

先运行回归场景，再删除 `requires`。

| 测试场景 | 预期结果 |
|---|---|
| 一行内部修复 | 不生成 specs 和 design |
| API 行为变化 | 必须生成并审查 specs |
| 跨模块设计变化 | 必须生成并审查 design |
| 实现后修改了已审查文件 | 旧 gate 失效，需要重新审查 |
| 不使用 OpenSpec 的仓库 | 更换 Markdown adapter 后可以运行相同编排 |

全部场景通过后，再把静态 `requires` 从流程真相改成兼容校验，最后删除不再使用的链式定义。

### 改造完成的验收标准

- 小型变更可以合法跳过 specs 和 design，不需要 bypass。
- model-invoked skill 中没有 OpenSpec 文件路径和阶段编号。
- 每道 gate 都能针对单独输入运行。
- 替换 OpenSpec adapter 时，不需要改三个 user-invoked skill。
- Git、worktree 和项目上下文 skill 可以脱离变更流水线独立使用。

最大的风险是把 OpenSpec 耦合替换成一个更大的自研 schema。Change Record 应保持为最小接口，只保存**意图、计划、证据**；具体文件布局继续交给 adapter。

先做第 1 步：用现有 schema、8 个模板和 5 个 gate prompt 建立耦合清单，再定义最小 Change Record。