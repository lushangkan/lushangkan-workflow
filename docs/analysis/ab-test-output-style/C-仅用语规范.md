## 先给结论

不要直接删掉 OpenSpec。

先把 OpenSpec 从“流程控制器”改成“文档存储和校验工具”。

目标可以概括成四点：

1. **user-invoked skill 决定流程。**  
   它决定先做什么、跳过什么、何时让子代理审查。

2. **model-invoked skill 保存具体规则。**  
   例如 requirements 应检查什么，design 应检查什么，完成实现前必须提供哪些验证证据。

3. **OpenSpec 只负责读写和校验。**  
   普通 skill 不直接访问 OpenSpec 路径、命令和 `requires`。

4. **文档、审查结果、决策、操作分开处理。**  
   `apply` 是操作，不是 artifact。`implementation-mode` 是决策，不是 artifact。

最重要的一步不是拆更多 skill，而是先把当前 8 个 artifact 的性质理清。

---

# 一、当前问题不只是“OpenSpec 绑定太深”

现在的 8 个 artifact 混合了四类东西。

| 当前 artifact | 实际性质 | 建议 |
|---|---|---|
| `proposal` | 长期保留的文档 | 保留 |
| `proposal-review` | 审查结果 | 改成 review record |
| `specs` | 长期保留的文档 | 保留，但按变化类型决定是否需要 |
| `design` | 长期保留的文档 | 保留，但不是所有改动都需要 |
| `implementation-mode` | 一次决策 | 改成 decision record |
| `tasks` | 实施计划的一部分 | 和 `execution-plan` 合并考虑 |
| `execution-plan` | 实施计划的一部分 | 和 `tasks` 合并考虑 |
| `apply` | 执行动作 | 改成 user-invoked skill |

这会带来三个问题。

### 1. 每个东西都被迫进入固定顺序

一个只改一处错误提示的任务，也要经过 proposal、specs、design、tasks、execution-plan。

流程成本不会根据改动内容变化。

### 2. 审查结果被当成正式文档

`proposal-review` 的价值是回答：

- 审查了哪个版本？
- 是否通过？
- 哪里有问题？
- 修改后是否需要重新审查？

它不应该和 proposal 处于同一种 artifact 类型。

### 3. OpenSpec 同时决定格式和流程

skill 会逐渐依赖这些内容：

- OpenSpec artifact 名称
- 文件路径
- `requires`
- OpenSpec CLI
- 固定的下一步名称

这时即使把 skill 拆小，skill 仍然是 OpenSpec 流程的一部分。

---

# 二、推荐的目标结构

建议分成四部分。

```text
用户
  │
  ▼
user-invoked workflow
  ├── 选择需要哪些文档
  ├── 选择需要哪些审查
  ├── 调用子代理
  ├── 决定是否继续
  └── 调用 change-store CLI
                    │
                    ▼
             OpenSpec adapter
                    │
                    ▼
               OpenSpec 文件
```

model-invoked skill 不负责推进流程。

它们只提供具体规则，例如：

- requirements 应该怎么写
- design 应该检查哪些风险
- plan 什么时候算可执行
- Git 操作有哪些安全要求
- 完成实现前必须提供什么验证结果

---

# 三、怎样判断一个 skill 是否足够原子化

原子化不等于“越小越好”。

一个 skill 满足下面这些条件，就可以认为边界比较合适：

1. 只有一个明确责任。
2. 单独调用时仍然有价值。
3. 输入和输出清楚。
4. 不决定整个流程的下一步。
5. 不依赖固定存储路径。
6. 不要求调用者必须使用 OpenSpec。
7. 不同时负责生成、审查、修复和批准。

例如：

### 合适的 model-invoked skill

`requirements-discipline`

负责：

- 检查行为是否明确
- 检查边界情况
- 检查错误处理
- 检查验收条件是否可验证

不负责：

- 创建 OpenSpec `specs`
- 决定下一步进入 `design`
- 把 change 状态改成 approved
- 自动启动实现

### 不合适的 model-invoked skill

`generate-specs-and-review-then-start-design`

它同时拥有生成、审查和流程推进。

它只是把原来的固定流水线搬进了一个 skill。

---

# 四、重新设计 artifact

## 1. 保留真正需要长期保存的文档

建议核心文档只保留这些语义类型：

- `proposal`
- `specs`
- `design`
- `plan`

其中：

- `proposal` 说明目标、范围和不做什么。
- `specs` 说明外部行为、接口和验收条件。
- `design` 说明技术选择、数据流、迁移和回滚。
- `plan` 说明修改位置、执行顺序和验证方式。

不是每次改动都要生成四份独立文件。

例如：

### 只修改内部实现

可能只需要：

- 简短 proposal
- plan

### 修改用户可见行为

可能需要：

- proposal
- specs
- plan

### 修改公共 API、数据库或部署方式

通常需要：

- proposal
- specs
- design
- plan

判断依据应该来自改动内容，不应该来自固定的 `requires`。

---

## 2. 合并 `tasks` 和 `execution-plan`

这两个 artifact 很容易重复。

建议统一成 `plan`。每个计划项至少包含：

```yaml
- id: T1
  change: 修改鉴权中间件
  files:
    - src/auth/middleware.ts
  depends_on: []
  verification:
    - 运行 auth 单元测试
    - 验证过期 token 返回 401
```

如果你确实需要分别展示任务列表和执行顺序，可以在同一份 `plan` 中使用两个 section：

```markdown
## Tasks

## Execution order
```

没有必要为这两部分建立两个强制阶段。

迁移期间也可以先保留旧文件。OpenSpec adapter 负责把统一的 `plan` 拆成旧的 `tasks` 和 `execution-plan`。

---

## 3. 把 `implementation-mode` 改成决策记录

它应该记录：

```yaml
name: implementation-mode
value: worktree
reason: 需要同时保留当前工作区中的未提交改动
decided_at: 2025-...
```

它不需要成为流程阶段。

可以按现有取值继续使用，例如：

- direct
- worktree
- delegated

关键点是：它只是当前 change 的一个选择。

---

## 4. 把 `apply` 改成 user-invoked skill

例如：

- `/change-plan`
- `/change-apply`
- `/change-review`

`/change-apply` 负责：

1. 读取已选定的文档版本。
2. 确认所需审查没有过期。
3. 选择 implementation mode。
4. 准备 worktree。
5. 执行 plan。
6. 运行验证。
7. 记录实现结果。
8. 返回完成、失败或需要用户决定。

这样 `apply` 不再伪装成 artifact。

---

# 五、把 5 道 gate 改成独立 review skill

不管当前 5 道 gate 叫什么，都建议使用统一输入输出。

可能的 review skill 包括：

- `scope-review`
- `requirements-review`
- `design-review`
- `plan-review`
- `implementation-review`

它们都遵守同一种结果格式。

```yaml
check: design-review
status: changes_requested

subjects:
  - kind: design
    revision: sha256:abc123
  - kind: specs
    revision: sha256:def456

findings:
  - severity: blocker
    criterion: rollback
    evidence: "设计只描述了迁移，没有描述失败后的回滚方式"
    action: "增加回滚条件和回滚步骤"

reviewer: design-reviewer
reviewed_at: 2025-...
```

`status` 可以限制为：

- `pass`
- `changes_requested`
- `blocked`

## review skill 不应该做这些事

- 不修改原文。
- 不把 change 自动推进到下一阶段。
- 不自行启动另一个 review。
- 不因为发现问题就直接实现修复。
- 不提交 Git commit。

review skill 只回答“当前内容是否符合这组标准”。

由 user-invoked workflow 决定：

- 是否修复
- 是否重新审查
- 是否允许带 warning 继续
- 是否交给用户处理

---

## 处理“文档修改后审查过期”

这是替代静态 `requires` 的重要部分。

每次 review 都记录它审查的文档版本。

如果 `design` 修改了，旧的 design review 自动失效。

如果 design review 同时参考了 `specs`，那么 `specs` 修改后，这次 design review 也应失效。

不需要写死：

```text
specs → design → design-review
```

只需要记录：

```text
这次 design-review 检查了 design revision A 和 specs revision B
```

这样依赖关系来自实际输入，不来自全局固定顺序。

---

# 六、让 user-invoked skill 真正拥有流程

这里的“拥有流程”指的是：

- 判断要生成哪些文档
- 判断要执行哪些审查
- 判断什么情况下可以跳过
- 判断失败后回到哪里
- 判断什么时候需要用户决定

## `/change-plan` 示例

```text
1. 读取用户请求和项目上下文。
2. 判断改动涉及哪些内容。
3. 记录本次需要的文档和审查。
4. 生成 proposal。
5. 按需生成 specs。
6. 按需生成 design。
7. 执行对应 review。
8. 生成 plan。
9. 执行 plan review。
10. 返回 ready 或 changes_requested。
```

第 3 步应根据明确条件判断。

### 只涉及单个模块的内部修改

如果不影响公共 API、数据格式和用户行为：

- 可以不生成独立 specs。
- 可以不生成 design。
- plan 可以很短。

### 影响用户可见行为或公共 API

需要：

- specs
- requirements review
- plan review

### 涉及数据迁移、安全、部署或不可逆操作

需要：

- design
- design review
- 回滚方案
- 实现后的独立验证

每次跳过都记录原因：

```yaml
skipped:
  design:
    reason: "修改只涉及单个函数，不改变模块边界、数据结构或公共 API"
```

不要只记录 `design: false`。

---

## `/change-apply` 示例

```text
1. 读取 plan 及其引用的文档版本。
2. 检查所需 review 是否通过。
3. 检查 review 是否过期。
4. 决定 direct 或 worktree。
5. 按 plan 实现。
6. 每完成一项就执行对应验证。
7. 执行最终验证。
8. 调用 implementation-review。
9. 记录结果。
10. 由用户决定是否 commit、push 或创建 PR。
```

建议最多自动执行两轮“修复后重新审查”。

两轮后仍有 blocker，就停止并交给用户。

这可以防止子代理在 review 和 fix 之间无限循环。

---

# 七、把 OpenSpec 隔离到 adapter

这是解除绑定的关键。

普通 skill 不应该出现这些内容：

- `.openspec/...`
- OpenSpec artifact 文件路径
- `openspec create`
- `openspec apply`
- `requires`
- “完成 specs 后进入 design”

skill 只调用统一的 CLI。

例如：

```bash
change-store document get CHANGE_ID --kind specs
change-store document put CHANGE_ID --kind design --file design.md
change-store decision record CHANGE_ID --file decision.yaml
change-store review record CHANGE_ID --file review.yaml
change-store validate CHANGE_ID
```

第一版只需要做 CLI，不需要做服务 API。

CLI 有几个好处：

- skill 容易调用。
- 可以用 JSON 作为输出。
- 可以用退出码表示失败。
- 测试简单。
- 以后可以增加其他存储方式。

## OpenSpec adapter 负责映射

例如：

```text
change-store document get --kind proposal
    ↓
读取 OpenSpec proposal artifact
```

```text
change-store review record
    ↓
写入原来的 proposal-review
或者写入 reviews/ 下的记录
```

```text
change-store document put --kind plan
    ↓
迁移期间拆成 tasks 和 execution-plan
```

以后如果你改用普通 Markdown、GitHub Issue 或其他系统，只需要实现另一套 `change-store`。

skill 不需要改。

---

## 不要一开始设计完整的通用数据模型

个人仓库不需要先做一个大型 workflow engine。

建议只定义三个最小对象。

### Document

```json
{
  "changeId": "add-oauth",
  "kind": "specs",
  "revision": "sha256:...",
  "content": "...",
  "references": [
    {
      "kind": "proposal",
      "revision": "sha256:..."
    }
  ]
}
```

### Decision

```json
{
  "changeId": "add-oauth",
  "name": "implementation-mode",
  "value": "worktree",
  "reason": "当前工作区有未提交内容"
}
```

### ReviewResult

```json
{
  "changeId": "add-oauth",
  "check": "design-review",
  "status": "pass",
  "subjects": [
    {
      "kind": "design",
      "revision": "sha256:..."
    }
  ],
  "findings": []
}
```

文档正文继续使用 Markdown。

不要把 OpenSpec 的每一个字段都复制到新的通用 schema。那样只会产生第二套 OpenSpec。

---

# 八、建议的目录结构

```text
skills/
  user/
    change-plan/
      SKILL.md
    change-apply/
      SKILL.md
    change-review/
      SKILL.md
    github-project-flow/
      SKILL.md
    project-init-workflow/
      SKILL.md

  model/
    requirements-discipline/
      SKILL.md
    design-discipline/
      SKILL.md
    planning-discipline/
      SKILL.md
    verification-discipline/
      SKILL.md
    git-master/
      SKILL.md
    using-git-worktrees/
      SKILL.md
    project-context-docs/
      SKILL.md

adapters/
  change-store/
    contract.md
    cli/
    openspec/
      read-document.*
      write-document.*
      record-review.*
      validate.*

references/
  review-result.schema.json
  decision.schema.json
  document.schema.json

profiles/
  default.yaml
  legacy-openspec.yaml

tests/
  adapter/
  workflows/
  skills/
  fixtures/
```

`references/` 中的内容不是可调用 skill。

它只保存多个 skill 共用的格式和检查表。

---

# 九、现有 5 个 skill 怎么处理

## 1. `github-project-flow`

保留为 user-invoked skill。

它负责：

- 从 Issue 创建工作
- 创建 branch 或 worktree
- 调用 change planning
- 调用 change apply
- 创建 PR
- 更新 Issue 或 Project 状态

它可以拥有整个 GitHub 流程。

但不要把 requirements、design 和 Git 安全规则全部复制进去。

---

## 2. `project-init-workflow`

保留为 user-invoked skill。

它负责：

- 检查仓库状态
- 初始化项目
- 创建项目上下文文档
- 初始化 Git 配置
- 按配置决定是否启用 OpenSpec

OpenSpec 应该是可选存储方式，不应成为该 skill 的固定前提。

---

## 3. `project-context-docs`

改成 model-invoked discipline。

它负责定义：

- 什么任务开始前必须读取项目上下文
- 哪类决策需要更新文档
- 怎样判断文档已经过期
- 更新后需要验证哪些引用

它不负责：

- 创建 branch
- commit
- 推进 change 状态
- 启动整个项目流程

---

## 4. `using-git-worktrees`

保留为 model-invoked skill。

它负责：

- 判断工作区是否适合直接修改
- 创建和清理 worktree
- 检查 branch 冲突
- 防止删除仍有未提交内容的 worktree

它不应该自动决定整个 GitHub 流程。

implementation mode 由调用者记录。该 skill 负责安全执行。

---

## 5. `git-master`

先检查它是否包含差异很大的操作。

如果它同时负责：

- 普通 commit
- 历史检查
- rebase
- reset
- force push
- 冲突恢复

它可能太宽。

可以按触发条件拆成：

- `git-commit-discipline`
- `git-history-inspection`
- `git-rewrite-safety`

不必为了数量而拆。

只有当这些操作的触发条件、风险和输出明显不同时再拆。

例如，普通 commit 和 force push 的安全规则差异很大，适合分开。

---

# 十、OpenSpec schema 怎么改

## 近期

先保留当前 schema 和 8 个 artifact。

但做两件事：

1. 所有 skill 改为通过 `change-store` 访问。
2. 5 道 gate 改为标准化 `ReviewResult`。

此时行为不变，只减少直接依赖。

## 中期

建立 OpenSpec schema v2。

v2 中只把真正的文档定义为 artifact：

- proposal
- specs
- design
- plan

其余内容改成记录：

- decisions
- reviews
- execution status
- verification evidence

`requires` 只用于局部数据要求。

例如某份 design 明确引用某个 specs 版本，可以记录引用。

不要再用 `requires` 表示全局流程：

```text
proposal 必须先于 specs
specs 必须先于 design
design 必须先于 tasks
```

如果 OpenSpec 无法表达条件式依赖，就不要再让它判断流程是否 ready。

由 `/change-plan` 和 `/change-apply` 判断。

## 兼容期

保留一个 `legacy-openspec` profile。

它继续要求：

- 8 个旧 artifact
- 5 道旧 gate
- 原来的顺序

新 workflow 默认使用按内容选择的流程。

这样你可以逐个项目迁移，而不是一次切断。

---

# 十一、建议按 6 个阶段迁移

## 阶段 1：建立现状测试

先准备几个固定场景：

1. 单文件错误修复。
2. 修改用户可见行为。
3. 修改公共 API。
4. 数据迁移。
5. 实现中发现 specs 有误。
6. review 后文档又被修改。

记录当前输出和失败方式。

这一步不改流程。

---

## 阶段 2：统一 gate 输出

让所有子代理返回同一种 `ReviewResult`。

先继续使用原来的固定流水线。

验收条件：

- 每次 review 都记录被审查版本。
- blocker、warning 和建议可以区分。
- review 不再自动推进状态。
- 文档修改后可以判断旧 review 已过期。

---

## 阶段 3：增加 `change-store`

把所有直接 OpenSpec 操作移动到 adapter。

验收条件：

- `skills/user` 和 `skills/model` 中不出现 `.openspec` 路径。
- 不直接调用 OpenSpec CLI。
- 替换存储方式时，skill 不需要改。
- OpenSpec adapter 有独立测试。

可以加一个简单的 CI 检查：

```bash
grep -R "\.openspec\|openspec " skills/
```

发现结果就失败。

---

## 阶段 4：增加新的 user-invoked workflow

先增加：

- `/change-plan`
- `/change-apply`
- `/change-review`

旧入口继续存在。

新入口开始根据改动内容选择文档和 review。

验收条件：

- 单文件内部修改可以跳过 design。
- 公共 API 修改必须有 specs。
- 数据迁移必须有 design 和回滚说明。
- 每次跳过都有原因。

---

## 阶段 5：建立 schema v2

开始移除这些 artifact：

- `proposal-review`
- `implementation-mode`
- `apply`

合并：

- `tasks`
- `execution-plan`

OpenSpec adapter 继续支持旧 schema。

不要同时修改 skill 结构、存储格式和所有命令名称。每次只改一类问题。

---

## 阶段 6：整理现有 5 个 skill

明确每个 skill 的调用者：

- 用户直接调用
- workflow 调用
- 模型按条件加载

删除同时服务两种调用者的模糊入口。

如果多个 skill 共享格式，把格式放进 `references/` 或脚本。

不要在多个 `SKILL.md` 中复制相同审查标准。

---

# 十二、建议加入的测试

## Adapter 测试

验证：

- 读取每种文档。
- 写入后 revision 变化。
- review 可以关联文档版本。
- OpenSpec 校验错误可以正确返回。
- `plan` 和旧 `tasks`、`execution-plan` 的转换不会丢内容。

## Workflow 场景测试

验证：

- 内部小改动不会强制 design。
- API 变化会要求 specs。
- migration 会要求 rollback。
- review 失败后不会进入 apply。
- 修改文档后旧 review 会过期。
- 没有验证证据时不能标记完成。

## Skill 边界测试

可以用静态检查约束：

- model-invoked skill 不调用下一个流程 skill。
- review skill 不写原文。
- 普通 skill 不访问 OpenSpec 路径。
- Git skill 不修改 change 审批状态。
- adapter 不包含 requirements 或 design 的质量规则。

---

# 十三、几个容易踩的坑

## 1. 按 8 个 artifact 建 8 个新 skill

这只是把 OpenSpec 流水线复制成 skill 流水线。

问题没有解决。

## 2. 让 model-invoked skill 自动调用下一步

例如 requirements skill 完成后自动调用 design skill。

这会把固定流程藏起来，后面更难修改。

## 3. 同时保留两套真实数据

不要让 OpenSpec 文件和新的通用目录都成为真实来源。

一个 change 只能有一个主要存储位置。

adapter 可以生成兼容文件，但要明确哪个版本是准的。

## 4. review 同时负责修改

审查者发现问题后直接改文档，会导致审查输入和输出混在一起。

建议流程是：

```text
review → changes_requested → author 修复 → 重新 review
```

## 5. 为了“通用”设计大型框架

你的仓库是个人工作流。

先实现：

- 三个数据对象
- 一个 CLI
- 一个 OpenSpec adapter
- 三个 user-invoked workflow

不需要先做插件系统、事件系统或完整状态机。

---

# 十四、最终验收标准

完成改造后，应该满足这些条件：

1. 删除 OpenSpec 后，核心 skill 仍然能描述完整工作方式。
2. 新增一种存储方式时，只需要增加 adapter。
3. `apply` 是操作，不是 artifact。
4. `implementation-mode` 是决策，不是阶段。
5. review 可以单独调用。
6. review 不会自动推进流程。
7. 文档修改后，相关 review 会失效。
8. 小改动可以跳过不需要的文档。
9. 公共 API、迁移和安全改动仍然受到严格检查。
10. 旧 OpenSpec 流程可以通过兼容 profile 继续运行。
11. skill 中没有 OpenSpec 路径和命令。
12. user-invoked skill 负责流程，model-invoked skill 负责规则。

如果只选一个最先实施的改动，我建议先做：

> **增加 `change-store` CLI，并禁止核心 skill 直接访问 OpenSpec。**

它不会立刻改变现有行为，但会切断最难处理的依赖。之后你才能安全调整 artifact、gate 和 workflow。