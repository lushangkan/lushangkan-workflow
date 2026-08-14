修改：把 OpenSpec 降为可替换的 schema/导出 adapter；删除 8 个 artifact 的固定 `requires` 链；保留真正需要长期维护的文档；把 5 道 gate 改成可独立调用的审查 skill。

# 1. 逐项决定哪些保留、删除、修改

## 8 个 artifact

| 当前对象 | 判决 | 具体改法 | 依据 |
|---|---|---|---|
| `proposal` | **修改** | 保留为 `proposal.md`。只写目标、范围、非目标、约束和验收结果。路径由调用者传入。 | 它是有长期价值的变更说明，但不应依赖 OpenSpec 路径。 |
| `proposal-review` | **删除为必需 artifact** | 审查结果默认只返回给调用者。需要审计时，再保存到 `reviews/`。 | “审查动作”不等于“业务文档”。强制落盘会制造大量短期文件。 |
| `specs` | **修改** | 保留行为要求、场景、边界条件。允许多个文件。由调用者传入输入和输出路径。 | specs 有长期价值，但不应绑定固定前置 artifact。 |
| `design` | **修改为条件 artifact** | 涉及跨模块、数据结构、公共 API、安全、并发或迁移时才创建。 | 小修改强制写 design，只会增加重复内容。 |
| `implementation-mode` | **删除为 artifact** | 把其中的事实移到 `change.yaml` 或调用参数，例如风险项、是否允许直接实现、是否需要 worktree。 | “模式”是流程选择，不是需要长期维护的文档。 |
| `tasks` | **修改** | 保留任务 ID、依赖、验收条件、验证命令和状态。合并原 `execution-plan` 中的执行顺序。 | tasks 应同时回答“做什么、按什么顺序做、怎么确认完成”。 |
| `execution-plan` | **删除为固定 artifact** | 普通内容并入 `tasks.md`。部署、数据迁移、回滚步骤单独放到条件文件 `runbook.md`。 | 它通常与 tasks 重复。只有生产操作步骤值得单独维护。 |
| `apply` | **删除为 artifact，改成动作** | 改成 user-invoked 的 `apply-change`。每次只选择并实现一个 task，再验证。 | `apply` 是执行行为，不是文档。 |

最终长期保留的核心文档是：

- `proposal.md`
- `specs/`
- 条件性的 `design.md`
- `tasks.md`
- 条件性的 `runbook.md`

## OpenSpec、`requires` 和 5 道 gate

| 当前对象 | 判决 | 具体改法 |
|---|---|---|
| OpenSpec schema | **修改** | 不再定义完整流程。只用于兼容导出或内容验证。 |
| 8 个 artifact 的 `requires` | **删除** | 顺序、跳过条件和重试次数移到 user-invoked recipe。 |
| proposal gate | **修改** | 改成独立的 `review-scope`，检查目标、范围、非目标和验收结果。 |
| specs gate | **修改** | 改成 `review-specs`，检查可测试性、场景覆盖和与 proposal 的一致性。 |
| design gate | **修改** | 改成 `review-design`。没有 design 时直接跳过。 |
| tasks/execution gate | **修改** | 改成 `review-plan`，检查任务是否可执行、可验证、依赖完整。 |
| apply gate | **修改** | 改成 `review-implementation`，检查代码 diff、测试结果和规格符合性。 |

gate 不再由审查 skill 自己决定流程。

审查 skill 只提交 findings。user-invoked recipe 根据统一规则决定 `PASS`、`WARN` 或 `BLOCK`。

## 现有 5 个独立 skill

| 当前 skill | 判决 | 具体改法 |
|---|---|---|
| `git-master` | **拆分后修改** | 如果它同时覆盖 commit、历史分析、rebase/reset 等操作，拆成 `git-commit-discipline`、`git-history-analysis`、`git-rewrite-safety`。危险的 rewrite 只能显式调用。 |
| `using-git-worktrees` | **修改** | 保留为 model-invoked skill。只负责判断是否需要隔离、创建 worktree、返回路径和分支。不要自动进入实现流程。 |
| `github-project-flow` | **拆分后修改** | 把读取 issue、同步状态、创建 PR 分开。流程顺序放到 user-invoked recipe。 |
| `project-context-docs` | **拆分后修改** | 分成只读的 `read-project-context` 和显式写入的 `update-project-context`。避免模型为了读取上下文而意外修改文档。 |
| `project-init-workflow` | **修改并保留** | 明确设为 user-invoked。它可以负责编排初始化，但底层动作由独立 skill 完成。 |

`git-master` 不必按每条 Git 命令拆分。按权限和风险拆分即可：只读查询、普通写入、历史重写是三个不同边界。

# 2. 目标结构让 OpenSpec 不再控制流程

## 依赖方向

目标依赖方向是：

**用户 → user-invoked recipe → model-invoked skill / adapter → 文件和外部工具**

adapter 是只负责把同一份数据转换成某个工具所需格式的一小层代码。

| 层次 | 负责什么 | 可以知道什么 | 不应知道什么 |
|---|---|---|---|
| user-invoked recipe | 调用顺序、跳过条件、gate、重试、何时询问用户 | 完整变更目标和当前进度 | OpenSpec 内部格式细节 |
| model-invoked skill | 完成一个明确任务，例如写 specs、检查 design、创建 worktree | 本次输入、输出路径、检查标准 | 下一个 skill 是谁、整个流程有几步 |
| gate policy | 根据 findings 决定是否阻塞 | 严重程度、风险类型、目标文件 hash | 如何修复内容 |
| adapter | 转换路径、字段和工具命令 | OpenSpec、GitHub 等外部格式 | 变更流程和审批顺序 |
| artifact | 保存需要长期维护的信息 | 变更内容本身 | 谁调用了它、下一步是什么 |

原子化不等于“每个 prompt 都拆成一个文件”。

满足以下任一条件时才值得拆分：

- 调用者不同。
- 输入输出不同。
- 写入权限不同。
- 失败处理不同。
- 风险等级不同。

## 当前内容移到哪里

| 当前内容 | 目标位置 |
|---|---|
| OpenSpec artifact 内容规范 | 独立的 artifact contract |
| OpenSpec `requires` | `plan-change`、`apply-change` 等 user-invoked recipe |
| OpenSpec 路径规则 | `openspec-adapter` |
| proposal 审查 prompt | `review-scope` |
| specs 审查 prompt | `review-specs` |
| design 审查 prompt | `review-design` |
| tasks 和 execution-plan 审查 prompt | `review-plan` |
| apply 后代码审查 prompt | `review-implementation` |
| `implementation-mode` | `change.yaml` 中的事实字段，或 user-invoked 参数 |
| execution 顺序 | `tasks.md` 的依赖和排序字段 |
| 部署、迁移、回滚步骤 | 条件性的 `runbook.md` |
| worktree 创建 | `using-git-worktrees` |
| commit | `git-commit-discipline` |
| GitHub issue/PR 更新 | 独立 GitHub skill 或 adapter |

## 建议的文件位置

| 路径 | 用途 |
|---|---|
| `changes/<change-id>/change.yaml` | 变更 ID、标题、风险事实、外部链接、artifact 路径 |
| `changes/<change-id>/proposal.md` | 目标、范围、非目标、约束、验收结果 |
| `changes/<change-id>/specs/` | 行为要求和场景 |
| `changes/<change-id>/design.md` | 条件性的技术设计 |
| `changes/<change-id>/tasks.md` | 任务、依赖、验证方法和状态 |
| `changes/<change-id>/runbook.md` | 条件性的部署、迁移和回滚步骤 |
| `changes/<change-id>/reviews/` | 可选的审查记录 |
| `skills/user/` | user-invoked recipe |
| `skills/model/` | model-invoked 原子 skill |
| `adapters/openspec/` | OpenSpec 导入、导出和验证 |
| `policies/review-policy.md` | gate 的统一阻塞规则 |

`change.yaml` 不要重新实现 `requires`。

它只保存事实：

- change ID
- artifact 路径
- issue 或 PR 链接
- 是否涉及公共 API
- 是否涉及数据迁移
- 是否涉及安全或权限
- 是否需要部署和回滚

不要保存 `next_artifact`、`current_phase`、`must_run_after` 之类的流程字段。

## OpenSpec 的最终角色

建议采用单一事实来源：

1. `changes/<change-id>/` 是唯一可编辑版本。
2. OpenSpec adapter 从这里生成 OpenSpec 所需目录。
3. 生成出来的 OpenSpec 文件不手动修改。
4. OpenSpec 验证结果返回给调用者。
5. 没有 OpenSpec 时，核心计划和实现流程仍能运行。

如果 OpenSpec 无法在不使用固定 `requires` 的情况下提供价值，就只保留兼容导出，不要让新架构继续适配它的限制。

# 3. Skill 和 gate 应该怎样设计

## user-invoked recipe 只负责流程

建议保留 4 个入口。

| recipe | 负责什么 | 不负责什么 |
|---|---|---|
| `plan-change` | 读取请求、调用上下文 skill、生成必要 artifact、执行规划阶段 gate | 修改生产代码、commit、更新 GitHub |
| `apply-change` | 选择 task、准备 worktree、实现、验证、执行代码审查 | 重新定义需求、自动修改 proposal |
| `review-change` | 对已有变更独立执行适用的审查 | 自动修复问题 |
| `project-init` | 初始化项目文档、目录、配置和可选 GitHub 资源 | 长期接管项目工作流 |

`plan-change` 可以编排多个 model-invoked skill。被调用的 skill 不应知道自己处于第几步。

`apply-change` 建议一次处理一个 task。这样中断、回滚和重试都更清楚。

## model-invoked skill 按单一任务拆分

### 内容生成

| skill | 输入 | 输出 | 禁止行为 |
|---|---|---|---|
| `read-project-context` | repo 路径、请求 | 相关上下文和来源 | 修改文件 |
| `draft-proposal` | 用户请求、上下文、输出路径 | proposal | 生成 tasks、选择实现方式 |
| `write-specs` | proposal、上下文、输出目录 | specs | 修改 proposal、实现代码 |
| `write-design` | proposal、specs、技术上下文 | design | 决定是否必须经过 gate |
| `plan-tasks` | proposal、specs、可选 design | tasks | 实现 task、commit |

### 实现和验证

| skill | 输入 | 输出 | 禁止行为 |
|---|---|---|---|
| `implement-task` | 一个 task、相关 specs/design、工作目录 | 代码修改和实现说明 | 自动 commit、选择下一个 task、更新 GitHub |
| `verify-change` | 验收要求、验证命令、代码 diff | 验证结果和证据 | 为了通过验证而修改代码 |
| `using-git-worktrees` | repo、change ID、目标分支 | worktree 路径和分支 | 开始实现、commit |
| `git-commit-discipline` | 已确认的文件范围、commit 意图 | staging 建议或 commit 结果 | 混入无关文件、执行历史重写 |

验证 skill 保持只读很重要。验证失败后，由 `apply-change` 决定是否再次调用 `implement-task`。

### 审查

| skill | 主要检查对象 | 核心检查项 |
|---|---|---|
| `review-scope` | proposal | 目标是否明确、范围是否失控、验收结果是否可判断 |
| `review-specs` | proposal + specs | 场景覆盖、边界条件、可测试性、一致性 |
| `review-design` | specs + design | 可行性、风险、兼容性、迁移和回滚 |
| `review-plan` | 全部规划文档 + tasks | 任务完整性、依赖、验证方法、任务大小 |
| `review-implementation` | specs + tasks + code diff + 测试结果 | 规格符合性、缺陷、回归、未完成任务 |

每个审查 skill 应满足以下约束：

- 只读目标文件。
- 不直接修复问题。
- 不决定下一步调用谁。
- 不接收起草过程的完整对话。
- 每条 finding 都给出文件位置和证据。
- 审查结果绑定目标文件 hash，文件变化后结果失效。

## Skill 使用统一 contract

每个 `SKILL.md` 至少声明这些内容：

| 字段 | 要求 |
|---|---|
| Responsibility | 一句话说明唯一职责 |
| Invoked by | user、recipe 或 model |
| Inputs | 必需输入、可选输入、路径格式 |
| Outputs | 写入文件或结构化返回值 |
| Side effects | 会修改哪些文件、是否执行命令 |
| Must not | 明确禁止的行为 |
| Failure | `blocked`、`needs_input` 等失败结果 |
| Permissions | 只读、普通写入、Git 写入、危险操作 |

普通 skill 的结果只需要表达：

- `completed`
- `blocked`
- `needs_input`
- 写入了哪些文件
- 有哪些未解决问题

不要返回“下一步调用 `write-design`”。这属于 recipe 的职责。

## gate 根据 findings 判断，不由 reviewer 决定

审查 skill 返回：

| 字段 | 内容 |
|---|---|
| `review_id` | 本次审查标识 |
| `target_hash` | 被审查内容的 hash |
| `coverage` | 实际检查了哪些内容 |
| `findings` | 问题列表 |
| `severity` | `critical`、`high`、`medium`、`low` |
| `location` | 文件和具体位置 |
| `evidence` | 判定依据 |
| `suggested_fix` | 建议修复方向 |

recipe 再执行 gate policy。

推荐默认规则：

1. 未解决的 `critical` 和 `high` finding 阻塞。
2. `medium` 生成警告，并进入 tasks。
3. `low` 只记录，不阻塞。
4. 目标文件 hash 变化后，旧 gate 自动失效。
5. 自动修复最多重试 2 次，之后询问用户。

审查 subagent 最好只拿到目标文件、代码和检查标准。不要把起草过程的全部推理交给它。这样更容易发现前一个模型的假设错误。

## 不再使用固定路线

recipe 根据变更事实选择必要内容。

| 变更情况 | 必需 artifact | 必需审查 |
|---|---|---|
| 局部缺陷修复；不改变外部行为；不涉及安全、数据或公共 API | task、验证证据 | implementation review |
| 改变用户可见行为、CLI、API 或配置语义 | proposal、specs、tasks | scope、specs、plan、implementation |
| 跨模块改造、数据结构变化、并发、安全、权限 | proposal、specs、design、tasks | 5 类审查全部执行 |
| 部署、数据迁移或不可逆操作 | 上述内容加 `runbook.md` | design 和 implementation 审查必须检查回滚方案 |

这取代 `implementation-mode`。

recipe 根据具体事实选择路径，而不是先选一个含义模糊的模式名称。

# 4. 按低风险顺序迁移

**不要先删除旧 schema、旧 `requires` 或历史 artifact。先让新旧流程对同一批案例产生可比较结果。**

## 迁移步骤

1. **固定当前行为。**  
   准备三个测试案例：局部修复、普通行为变更、高风险结构变更。保存当前生成物和 gate 结果。

2. **定义中立 contract。**  
   为 proposal、specs、design、tasks 和 review findings 写清输入、输出和必需字段。

3. **先拆出 5 个 review skill。**  
   让它们可以直接审查任意路径，不引用 OpenSpec artifact 名称和 `requires`。

4. **创建 `plan-change` 和 `apply-change`。**  
   第一版仍可读取旧 OpenSpec 目录，但调用顺序只写在 recipe 中。

5. **拆出内容生成和执行 skill。**  
   每个 skill 接受显式输入路径、输出路径和约束，不读取固定的 OpenSpec 相对路径。

6. **建立 `changes/<change-id>/`。**  
   把它设为新的唯一可编辑位置，并加入最小化的 `change.yaml`。

7. **加入 OpenSpec adapter。**  
   adapter 负责从中立 artifact 生成 OpenSpec 格式，并运行可选验证。

8. **并行运行新旧流程。**  
   连续用至少 3 个真实变更比较文档质量、审查结果、耗时和失败恢复。

9. **切换默认入口。**  
   默认使用 `plan-change` 和 `apply-change`。旧 OpenSpec workflow 只作为兼容入口。

10. **删除固定流水线。**  
    新流程稳定后，再删除 `proposal-review`、`implementation-mode`、`execution-plan`、`apply` artifact 和静态 `requires`。

## 切换前必须通过

- OpenSpec 不可用时，`plan-change` 和 `apply-change` 仍可运行。
- 任意 model-invoked skill 都能处理调用者传入的路径。
- 只有 user-invoked recipe 知道 artifact 顺序。
- reviewer 不修改被审查内容。
- 没有 design 的变更可以正常跳过 design gate。
- OpenSpec 生成目录可以删除并完整重建。

## 待你决定的 2 项

1. **OpenSpec 的长期角色**  
   推荐值：**先保留为可选 adapter；运行 1～2 个月后，如果只剩格式转换用途，再删除运行时依赖。**

2. **审查结果是否提交到 Git**  
   推荐值：**普通审查不提交；只保存最终 `BLOCK`、人工批准和高风险变更的审查记录。**