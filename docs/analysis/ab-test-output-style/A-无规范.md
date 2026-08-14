你现在真正需要拆的，不是“把 8 个 artifact 分别包装成 8 个 skill”，而是把以下四件目前混在一起的东西解耦：

1. **流程编排**：这次变更要走哪些步骤、先后顺序是什么。
2. **模型纪律**：怎样写需求、怎样做设计、怎样评审、怎样实施。
3. **状态与产物**：哪些信息需要长期保存，哪些只是临时执行状态。
4. **存储与工具实现**：这些内容是不是写进 OpenSpec、GitHub、Markdown 文件或别的系统。

最适合你的目标形态是：

> **用户调用的 skill 负责选择和编排流程；模型调用的 skill 负责单一、可复用的判断纪律；OpenSpec 降级为 artifact backend/adapter，而不是流程本身。**

---

# 一、先诊断当前设计的问题

你目前的模型大致是：

```text
OpenSpec schema
  └─ proposal
      └─ proposal-review
          └─ specs
              └─ design
                  └─ implementation-mode
                      └─ tasks
                          └─ execution-plan
                              └─ apply
```

并在若干节点调用子代理 gate。

这个设计的问题不在于步骤多，而在于它同时编码了四种不同概念。

## 1. artifact 被当成了流程状态

`proposal-review`、`execution-plan`、`apply` 并不都是同一种东西：

- `proposal`、`specs`、`design` 是知识型产物。
- `proposal-review` 是对某个版本的评估结果。
- `implementation-mode` 是策略决定。
- `execution-plan` 更像运行时计划。
- `apply` 是动作，不是 artifact。

把这些都放入同一条 `requires` 链，会导致“文件存在”被误认为“工作已经准备好”。

## 2. OpenSpec 同时拥有了存储和控制流

理想情况下，OpenSpec 应该回答：

- 某个变更有哪些 artifact？
- 它们存在哪里？
- 是否符合结构约束？
- artifact 之间有什么语义关系？

但现在它还回答了：

- 下一步必须是什么？
- 哪个 gate 必须在什么时候运行？
- 哪些变更都必须走完整流程？

于是所有 skill 都必须理解 OpenSpec 的文件名、目录、schema 和状态机。

## 3. 所有变更被迫走同一条路径

以下变更显然不应走完全相同的流程：

- 改一个拼写错误。
- 修一个局部 bug。
- 加一个普通 feature。
- 做数据库迁移。
- 改认证或权限体系。
- 进行跨模块重构。

固定链条通常会出现两个结果：

- 简单变更为了合规产生大量低价值 artifact。
- 复杂变更看似完成所有 artifact，实际风险仍未被覆盖。

## 4. gate 与阶段绑定，而不是与风险或质量属性绑定

例如，“proposal review”听起来是在审查某个文档阶段；但真正想检查的可能是：

- 问题是否清楚？
- 范围是否过大？
- 是否存在未确认的假设？
- 需求是否可验证？
- 是否遗漏安全、兼容性或迁移风险？

这些检查不一定只在 proposal 之后发生，也可能在 design 或 implementation 阶段重新触发。

---

# 二、目标架构：四层分离

建议改成下面的结构：

```text
用户请求
   │
   ▼
User-invoked orchestrators
/change-plan
/change-implement
/change-review
/change-ship
/project-init
   │
   ├── 选择流程深度和检查点
   ├── 调用 model-invoked skills
   └── 决定如何处理 gate 结果
   │
   ▼
Model-invoked disciplines
frame-change
specify-behavior
design-solution
decompose-work
execute-work-item
verify-change
review-*
   │
   ▼
Semantic contracts / ports
intent
requirements
design
work-items
review-result
execution-evidence
   │
   ▼
Adapters
OpenSpec
Markdown/filesystem
Git
GitHub Projects
```

关键点是：

- **编排层知道“什么时候调用谁”**。
- **原子 skill 只知道“如何做好一件事”**。
- **核心 skill 使用语义输入输出，不直接依赖 OpenSpec 路径**。
- **adapter 负责把语义 artifact 映射到 OpenSpec。**

---

# 三、重新分类现有 8 个 artifact

不建议立刻删除现有 artifact。第一步是先改变它们的语义定位。

| 当前 artifact | 实际性质 | 建议的新定位 |
|---|---|---|
| proposal | 持久知识 | `change-intent`：问题、目标、范围、约束 |
| proposal-review | 评审结果 | `review-result`，绑定被评审 artifact 的具体 revision |
| specs | 持久知识 | `requirements`：行为、验收、边界条件 |
| design | 持久知识，但并非所有变更都需要 | `solution-design`，按风险生成 |
| implementation-mode | 策略决定 | `implementation-strategy`，通常作为计划元数据，而非必需独立文档 |
| tasks | 工作分解 | `work-items`，复杂或跨会话工作才需要持久化 |
| execution-plan | 运行时计划 | 默认临时生成；只有长周期、多代理任务才持久化 |
| apply | 动作 | 拆成 `execute-work-item`、`verify-change` 等执行 skill，不再作为 artifact |

## 一个重要变化

不要再把：

```text
proposal-review requires proposal
specs requires proposal-review
```

理解为全局流程。

应该改成：

```text
review-result:
  subject: proposal@revision-123
  verdict: pass | conditional | fail | blocked
```

然后由编排器决定：

- 这个 review 是否是当前 profile 的强制 gate。
- `conditional` 是否允许继续。
- 失败后回到哪个能力修正。
- artifact 修改后是否需要重新 review。

评审结果必须绑定具体 revision 或内容 hash，否则 proposal 修改后，旧的通过结果仍可能被误用。

---

# 四、用“语义前置条件”替换全局 `requires`

原子 skill 可以有前置条件，但不应该拥有完整流程。

建议把依赖分成三类。

## 1. 硬前置条件

没有就无法完成任务。

例如 `execute-work-item` 至少需要：

- 一个边界明确的工作项；
- 允许修改的代码范围；
- 验收或验证方法；
- 用户已经授权实施。

## 2. 可选上下文

有最好，没有也可以根据现场推导。

例如 `design-solution` 可以使用：

- 结构化 requirements；
- issue 描述；
- change intent；
- 现有代码和约束。

它不应该硬编码：

> 必须读取 `proposal.md` 和 `specs/*.md`，否则停止。

## 3. 就绪条件

不要检查“前一个文件是否存在”，而是检查信息是否足够。

例如：

```text
intent ready:
- 目标明确
- 范围或非目标明确
- 关键约束已知
- 未解决问题已显式记录

requirements ready:
- 可观察行为明确
- 验收条件可验证
- 关键错误和边界情况已覆盖

execution ready:
- 工作项边界明确
- 依赖已满足
- 验证方式明确
- 风险和回滚要求已知
```

这样，某些信息可以来自 OpenSpec，也可以来自 GitHub issue、用户对话或已有设计文档。

---

# 五、用户调用 skill 和模型调用 skill 怎么拆

## 1. 用户调用的 skill：薄编排器

建议公开少量稳定入口，而不是把所有内部 primitive 暴露给用户。

### `/change-plan`

职责：

1. 收集请求与项目上下文。
2. 判断变更类型、风险和不确定性。
3. 选择 `patch / standard / rigorous` 等流程 profile。
4. 调用必要的模型 skill。
5. 调用配置的 gate。
6. 将需要持久化的结果写入当前 artifact backend。

它不应该自己包含：

- 怎样写好需求的长篇规则。
- 怎样做架构设计。
- 怎样拆任务。
- 怎样审查安全性。

这些纪律应由模型调用 skill 持有。

### `/change-implement`

职责：

1. 定位要实施的变更或工作项。
2. 检查实施就绪性。
3. 必要时创建 worktree。
4. 逐个调用 `execute-work-item`。
5. 调用验证纪律。
6. 更新工作项和执行证据。

### `/change-review`

职责：

- 对现有 change、PR 或当前 diff 运行指定 gate 集合。
- 不必重新生成 proposal/spec/design。
- 可独立用于外部代码或别人创建的 PR。

### `/change-ship`

职责：

- 最终验证。
- 检查未提交修改、提交结构和分支状态。
- 创建或更新 PR。
- 更新 GitHub Project。
- 更新稳定的项目上下文。
- 关闭或归档 change。

### `/project-init`

保留为用户调用的编排器，但把内部能力拆出去。

## 2. 模型调用的 skill：持有单一纪律

可以考虑以下能力集：

```text
frame-change
specify-behavior
design-solution
select-implementation-strategy
decompose-work
execute-work-item
verify-change
maintain-traceability
review-intent
review-requirements
review-design
review-work-plan
review-implementation
```

每个 skill 应做到：

- 有独立触发条件。
- 只负责一种可复用判断或行为。
- 接收语义输入，不要求特定前置文件名。
- 返回标准化结果。
- 不擅自决定完整流程。
- 不在末尾写“然后必须调用下一个 skill”。
- 明确副作用与停止条件。

例如 `specify-behavior` 的职责是把意图转成可验证行为，不应该同时设计实现、拆任务和创建 worktree。

## 3. 对执行型模型 skill 增加授权边界

像 `execute-work-item`、`rewrite-git-history` 会产生副作用。即便是 model-invoked，也应该要求调用者传入明确授权：

```yaml
authorization:
  may_modify_files: true
  allowed_scope:
    - src/auth/**
    - tests/auth/**
  may_commit: false
  may_push: false
```

“model-invoked”不等于“模型可以随时自动修改”。

---

# 六、不要让每个小动作都变成 skill

原子化不等于 skill 越多越好。

可以按以下原则划分：

| 类型 | 适合放在哪里 |
|---|---|
| 需要模型判断、权衡、审查的行为 | skill |
| 确定性的读写、转换、校验 | script/tool |
| 输入输出结构 | contract/schema |
| 调用顺序和异常路由 | user-invoked orchestrator/recipe |
| OpenSpec/GitHub 特有操作 | adapter |
| 团队或项目的强制规则 | policy |

例如：

- “判断需求是否可测试”是 skill。
- “把 Markdown 写到 OpenSpec 的指定目录”是 adapter script。
- “解析 Git 状态”优先是脚本或工具。
- “根据 Git 状态判断是否允许安全 rebase”才是模型纪律。

---

# 七、把 5 道 gate 改成可组合 policy

子代理可以保留。问题不在于“用了子代理”，而在于 gate 是否被静态嵌入 artifact 链。

## 1. 给所有 gate 统一输出契约

例如：

```yaml
subject:
  role: requirements
  ref: artifact://change-123/requirements
  revision: sha256:abc123

verdict: pass | conditional | fail | blocked

findings:
  - id: REQ-001
    severity: blocking | major | minor
    category: testability
    claim: "错误状态没有可观察的验收结果"
    evidence:
      - "Requirement R-4 only describes the happy path"
    recommendation: "补充超时和无权限时的响应与测试要求"

checks_run:
  - clarity
  - consistency
  - testability
  - scope

uncertainties:
  - "无法确认旧客户端兼容要求"
```

gate 最好遵循：

- 默认只评审，不直接修改被评审内容。
- finding 必须有证据。
- 区分 blocking 和非 blocking。
- `blocked` 表示上下文不足，不等于质量失败。
- 绑定具体 revision。
- 由编排器决定怎样处理 verdict。

## 2. 从“阶段 gate”逐步转向“质量属性 gate”

如果你现有 5 个 gate 大致对应 proposal/spec/design/tasks/apply，可以先保留兼容名称。但长期更适合按关注点拆分，例如：

- intent clarity / scope
- requirement consistency / testability
- design feasibility / risk
- traceability / plan executability
- implementation correctness / evidence
- security / privacy
- compatibility / migration
- release readiness

不是所有 gate 都需要在每次变更执行。

## 3. 让流程 profile 选择 gate

例如：

| Profile | 适用场景 | 主要产物 | Gate |
|---|---|---|---|
| patch | 拼写、局部低风险修复 | intent 摘要、执行证据 | implementation verification |
| standard | 普通 bug/feature | intent、requirements、可选 design、work-items | requirements、implementation |
| rigorous | 安全、迁移、跨模块、公共 API | 完整 intent、requirements、design、strategy、work-items | 多阶段强制 gate |

还可以用覆盖规则：

```text
涉及认证、授权、密钥：
  强制 security gate

涉及数据库 schema：
  强制 migration/rollback design

涉及公共 API：
  强制 compatibility gate

涉及多仓库或长周期实施：
  强制持久化 work-items 和 execution state
```

这样 gate 由风险驱动，而不是由文件顺序驱动。

---

# 八、OpenSpec 应该怎么降级为 adapter

最重要的规则是：

> 除 OpenSpec adapter 和 legacy wrapper 外，其他 skill 不应知道 OpenSpec 的目录、命令、artifact 名称或 `requires`。

## 1. 核心层只使用语义 role

例如：

```text
change-intent
requirements
solution-design
implementation-strategy
work-items
review-result
execution-evidence
```

而不是：

```text
openspec/changes/foo/proposal.md
openspec/changes/foo/specs/...
```

## 2. 定义一个很薄的 artifact port

不必做成大型框架，只需要覆盖现有需求：

```text
resolve(change_id, role) -> ArtifactRef[]
read(ref) -> content + metadata
write(change_id, role, content, metadata) -> ArtifactRef
validate(ref) -> ValidationResult
link(from, relation, to)
```

其中：

```yaml
ArtifactRef:
  role: requirements
  uri: openspec://changes/auth-refresh/specs
  revision: sha256:...
  media_type: text/markdown
```

核心 skill 只读写 role 和 content contract，adapter 决定实际路径。

## 3. artifact 之间使用语义关系，而不是完整流程依赖

可以保留这类关系：

```text
requirements derives_from intent
design addresses requirements
work-item implements requirement
evidence verifies requirement
review-result assesses artifact@revision
```

这些关系支持追踪，但不规定：

> design 之前必须生成 proposal-review 文件。

## 4. 保留一个 legacy profile

迁移期间可以继续支持旧流程：

```text
legacy-openspec-v1:
proposal → review → specs → design → mode → tasks → plan → apply
```

但它应该成为一个用户调用编排器或 recipe，而不是所有核心 skill 的默认假设。

如果 OpenSpec schema 无法表达可选 artifact 和分支流程，可以：

1. 冻结旧 schema，作为兼容模式。
2. 新建一个更轻的 schema，只描述 artifact 类型和结构。
3. 或者只在 `rigorous` profile 中使用 OpenSpec 完整 change。
4. 对 patch/standard 流程使用轻量 Markdown 或 issue backend。

不要为了让 OpenSpec 继续控制一切，再开发一套复杂的通用 workflow DSL。那只会从“被 OpenSpec 捆绑”变成“被自研引擎捆绑”。

---

# 九、现有 5 个独立 skill 怎么处理

## 1. `git-master`

这个名字和职责通常过宽，容易混入：

- 查看状态。
- 创建提交。
- 交互式 rebase。
- 拆分提交。
- 冲突解决。
- 恢复误操作。
- 分支与推送策略。

建议拆成模型纪律：

```text
inspect-git-state
create-focused-commit
rewrite-history-safely
resolve-git-conflict
recover-git-state
prepare-branch-for-review
```

确定性操作尽量由脚本完成，模型 skill 负责安全判断。

可以保留 `/git-master` 作为用户调用的兼容 dispatcher，但它本身不再持有所有细节。

## 2. `using-git-worktrees`

拆成两部分：

- `worktree-isolation-policy`：模型判断何时需要 worktree、如何避免污染现有工作。
- `prepare-worktree`：原子副作用能力，创建并验证 worktree。

由 `/change-implement` 或 `/start-isolated-work` 编排调用。

需要明确：

- 不覆盖已有目录。
- 不删除含未提交修改的 worktree。
- 分支冲突时停止。
- 创建完成后验证仓库根目录、分支和状态。

## 3. `github-project-flow`

它很可能混合了流程编排和 GitHub adapter。

建议拆成：

```text
GitHub adapter:
- resolve-github-issue
- update-project-item
- create-or-update-pr
- link-change-to-issue

用户编排:
- /start-issue
- /update-change-status
- /ship-change
```

业务 skill 不直接写死 GitHub Project 字段 ID 或状态名，应通过 adapter 映射。

## 4. `project-context-docs`

拆成：

- `read-project-context`
- `extract-stable-project-knowledge`
- `update-project-context`

其中最重要的纪律是：

> 只把长期稳定、跨任务可复用的信息写入项目上下文，不要把当前 change 的临时状态、猜测和执行日志写进去。

文件路径、格式和 section 更新由 adapter/script 处理。

## 5. `project-init-workflow`

它天然适合作为 user-invoked orchestrator，保留即可，但内部应改成组合：

```text
/project-init
  ├─ inspect repository
  ├─ detect stack
  ├─ initialize or validate git
  ├─ optionally configure worktree policy
  ├─ create project context
  ├─ optionally configure OpenSpec adapter
  └─ optionally configure GitHub adapter
```

初始化 skill 自己不应持有 Git、OpenSpec、GitHub 的全部纪律。

---

# 十、推荐的仓库结构

不一定要照搬目录名，但边界最好明确：

```text
skills/
  user/
    change-plan/
    change-implement/
    change-review/
    change-ship/
    project-init/

  model/
    frame-change/
    specify-behavior/
    design-solution/
    select-implementation-strategy/
    decompose-work/
    execute-work-item/
    verify-change/
    maintain-traceability/

    review-intent/
    review-requirements/
    review-design/
    review-work-plan/
    review-implementation/

    inspect-git-state/
    create-focused-commit/
    rewrite-history-safely/
    update-project-context/

contracts/
  change-intent.md
  requirements.md
  solution-design.md
  work-items.md
  review-result.md
  execution-evidence.md

recipes/
  patch.md
  standard-change.md
  rigorous-change.md
  legacy-openspec-v1.md

policies/
  risk-classification.md
  gate-policy.md
  side-effect-policy.md
  git-safety.md

adapters/
  openspec/
    README.md
    resolve-artifact.*
    read-artifact.*
    write-artifact.*
    validate-artifact.*

  github/
  project-context/
  filesystem/

tests/
  skills/
  contracts/
  adapters/
  recipes/
  fixtures/
```

如果你的 skill 系统要求所有 skill 平铺，也可以平铺；重点是职责边界，不是目录本身。

---

# 十一、每个原子 skill 的推荐模板

可以统一采用类似结构：

```markdown
---
name: specify-behavior
invocation: model
side_effects: none
---

# Purpose

把已知变更意图转化为明确、可验证的行为要求。

# Use when

- 用户描述了目标，但验收行为不完整
- issue 需要转成可实施需求
- 实现前需要补齐边界和错误行为

# Do not use when

- 仅需要检查已有 requirements
- 需要决定技术架构
- 用户尚未明确核心目标

# Inputs

- change intent，或等价的用户请求/issue
- relevant project context
- existing behavior and constraints
- optional prior requirements

# Discipline

- 区分目标、需求和实现方案
- 每条要求必须描述可观察行为
- 明确正常路径、失败路径和边界情况
- 不擅自填补高影响业务决策
- 对未知信息记录 assumption 或 open question

# Output

- status: ready | needs-input | blocked
- requirements
- assumptions
- open questions
- source references

# Stop conditions

- 核心业务结果不明确
- 两个来源存在无法消解的冲突
- 缺少必须由用户决定的策略

# Non-goals

- 不选择实现技术
- 不拆任务
- 不创建 OpenSpec 文件
- 不决定下一个 workflow step
```

核心检查点：

- skill 内不要写 OpenSpec 路径。
- 不要内置完整前后步骤。
- 不要一边评审一边静默重写。
- 不要输出“下一步必须调用某 skill”。
- 可以输出 `gaps` 和 `readiness`，由编排器决定下一步。

---

# 十二、建议的迁移顺序

不要大爆炸重写。最安全的路线是“先保行为，再切边界”。

## 阶段 1：盘点并建立语义词汇

为现有每个 artifact 和 gate 记录：

- 它的真实目的。
- 输入是什么。
- 输出是什么。
- 是否需要持久化。
- 是否有副作用。
- 哪些部分依赖 OpenSpec。
- 是否隐含调用了其他流程。
- 简单变更是否也必须需要它。

产出一份旧模型到新 role 的映射表。

## 阶段 2：先定义 contracts

优先定义：

1. `change-intent`
2. `requirements`
3. `solution-design`
4. `work-items`
5. `review-result`
6. `execution-evidence`

先统一“输出是什么”，再拆 skill。否则拆完后每个 skill 仍然输出不同格式。

## 阶段 3：给 OpenSpec 包一层 adapter

暂时不改旧流程，只做：

- 核心代码不再直接拼 OpenSpec 路径。
- 所有读写通过 adapter。
- adapter 继续产生完全相同的旧目录结构。

这是最重要的解耦点，也最容易回归验证。

## 阶段 4：提取模型纪律

从现有 8 段 prompt 中抽出：

- authoring 纪律；
- review 纪律；
- implementation 纪律；
- verification 纪律。

每个 skill 先在旧流程里调用，确保行为不退化。

## 阶段 5：把旧流水线改成兼容 orchestrator

创建：

```text
legacy-openspec-v1
```

它显式执行原来的完整顺序。此时固定流程仍然存在，但它不再分散在各 skill 和 schema 假设中。

## 阶段 6：改造 gate

- 统一 `review-result`。
- gate 绑定 revision。
- gate 默认只读。
- 由 orchestrator 解释 verdict。
- 将 gate 选择移入 profile/policy。

## 阶段 7：增加 patch 和 standard profile

先支持三条路径：

```text
patch
standard
rigorous/legacy
```

不要一开始做十几个 profile。

### Patch

```text
frame intent briefly
→ identify bounded change
→ implement
→ verify
```

### Standard

```text
frame intent
→ specify behavior
→ optional design
→ decompose if needed
→ implement
→ verify
```

### Rigorous

```text
intent
→ requirements
→ design
→ strategy
→ work-items
→ gated execution
→ final verification
```

## 阶段 8：重构 Git、worktree、GitHub 和 context skill

优先从最宽泛、最容易越权的 `git-master` 开始。

## 阶段 9：逐步弃用旧名称和路径假设

保留一段时间的兼容 wrapper，并在日志或输出中提示：

```text
legacy artifact "implementation-mode" is now represented as
implementation-strategy metadata
```

---

# 十三、如何验证改造成功

建议设置以下验收标准。

## 解耦指标

- 除 `adapters/openspec` 和 `legacy-openspec-v1` 外，代码和 skill 中不出现 OpenSpec 专有路径。
- 模型 skill 不检查某个固定 predecessor 文件是否存在。
- 更换 filesystem fixture 后，核心 skill 仍可运行。
- OpenSpec schema 变化不要求修改所有 skill。

## 原子性指标

每个 model-invoked skill：

- 只有一个主要 outcome。
- 不拥有完整生命周期。
- 不擅自调用下一阶段。
- 有明确输入、输出、停止条件和副作用。
- 可以被至少两个不同 orchestrator 或场景复用。

但不要把“可复用两次”机械当成门槛；核心是职责独立。

## 流程指标

- 低风险 patch 可以不生成 design、tasks 和 execution-plan。
- 高风险变更可以强制完整 gate。
- gate 失败可以回到相应能力修正，而不是整条链重跑。
- 可以单独 review 一个已有 change 或 PR。
- 可以从 GitHub issue、用户对话或 OpenSpec 任一来源启动流程。

## 评审正确性指标

- review 结果绑定 subject revision。
- artifact 更新后旧 review 自动视为 stale。
- blocking finding 有证据和明确修复条件。
- reviewer 不静默修改 subject。

## 回归测试

建议至少建立：

- contract fixture 测试。
- adapter round-trip 测试。
- 各 skill 的 prompt/eval 测试。
- gate verdict golden cases。
- patch/standard/rigorous 三条端到端测试。
- Git/worktree 副作用 sandbox 测试。

---

# 十四、几个容易踩的坑

## 1. 不要从 OpenSpec schema 换成自研万能 workflow engine

如果你为了动态编排开始设计：

- DSL；
- DAG runtime；
- event bus；
- 状态机；
- plugin registry；
- 多 backend ORM；

很可能只是创造了新的强耦合。

初期只需要：

- 几个 Markdown recipe；
- 一个风险分类 policy；
- 一个很薄的 artifact adapter；
- 标准化 skill contract。

## 2. 不要把“8 个 artifact”机械变成“8 个 skill”

`apply` 是动作，不是知识产物；`proposal-review` 是评估；`execution-plan` 多数时候是临时状态。按真实职责拆，而不是按现有文件名拆。

## 3. 不要让 orchestrator 重新变成巨型 prompt

编排器只应描述：

- 如何分类；
- 何时调用哪些能力；
- 如何处理结果；
- 哪些 checkpoint 需要用户确认；
- 怎样持久化。

具体的需求、设计、Git、安全和验证纪律全部下沉。

## 4. 不要把所有 review 合并成一个万能 reviewer

统一的是输出协议，不一定是 reviewer 本身。

需求可测试性、架构风险、安全、迁移和代码正确性需要不同的检查视角。可以并行调用多个小 gate，再由 orchestrator 汇总。

## 5. 不要为了原子性丢掉项目上下文

原子 skill 可以读取丰富上下文；原子性指的是职责边界，不是输入必须极少。设计 skill 读取 intent、requirements、代码和项目约束是合理的，只要它不顺便负责整个 lifecycle。

---

# 十五、最值得优先做的第一版

如果只做一轮改造，我建议先完成下面六件事：

1. **把现有流水线冻结为 `legacy-openspec-v1`。**
2. **定义六个语义 contract：intent、requirements、design、work-items、review-result、execution-evidence。**
3. **建立 OpenSpec adapter，禁止核心 skill 直接引用 OpenSpec 路径。**
4. **把 `proposal-review` 从强制 predecessor 改成 revision-bound gate result。**
5. **把 `apply` 从 artifact 改成 `execute-work-item + verify-change`。**
6. **新增 `/change-plan` 和 `/change-implement` 两个薄编排器，先支持 `standard` 和 `legacy` 两种 profile。**

这一版完成后，你的系统就会从：

```text
OpenSpec schema 决定流程，skill 填充节点
```

转变为：

```text
用户编排器决定流程
→ 原子 skill 提供纪律
→ policy 决定 gate
→ OpenSpec 负责持久化和校验
```

这才是最核心的解耦。之后无论你换成普通 Markdown、GitHub Issues，还是保留 OpenSpec 作为严谨变更流程，都不需要重写需求、设计、评审和实施能力。