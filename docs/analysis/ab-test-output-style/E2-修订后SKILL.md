建议：**保留 OpenSpec，但把 OpenSpec 降为文件格式和校验适配层；用 3 个 user-invoked skill 编排；把 gate 和复用规则改成 model-invoked skill。**

# 摘要

- **现在的问题**：OpenSpec schema 同时控制文件格式、执行顺序和 gate。修改任何一部分，都可能影响整条流水线。
- **目标结构**：user-invoked skill 只决定先做什么；model-invoked skill 保存写作、审查、Git 操作规则；OpenSpec adapter 只负责读写和校验文件。
- **推荐迁移方式**：先迁移 `proposal-review`，让新旧结构同时工作。验证通过后，再处理 `implementation-mode`、`execution-plan` 和 `apply`。

最关键的一条：**artifact、skill、workflow 是三种东西，不要按一个 artifact 对应一个 skill 来拆。**

# 目标结构：入口、规则、OpenSpec adapter

“编排”指决定先做什么、跳过什么、调用什么。

“纪律”指执行任务时必须遵守的规则和检查标准。

“适配层”指把通用输入转换成 OpenSpec 文件，并执行 schema 校验。

| 部分 | 推荐职责 | 禁止承担的职责 |
|---|---|---|
| user-invoked skill | 接收用户目标，选择步骤，设置停点 | 不保存完整 review 标准和 Git 操作细节 |
| model-invoked skill | 提供单项写作、审查或操作规则 | 不决定下一阶段，不启动完整流水线 |
| OpenSpec adapter | 读写 artifact，执行 schema 校验，兼容旧格式 | 不判断是否需要 design，不调用下一个 skill |

## 推荐保留 3 个用户入口

| 用户入口 | 负责什么 | 可能调用的 model-invoked skill |
|---|---|---|
| `plan-change` | 从请求生成足够实施的计划 | proposal、spec、design、task 的写作和审查 skill |
| `implement-change` | 准备工作区、执行 tasks、验证结果 | worktree、执行、测试、代码审查 skill |
| `ship-change` | commit、PR、GitHub Project 更新 | Git、GitHub、交付检查 skill |

入口只写三类内容：

1. **路由条件**：什么时候需要 specs 或 design。
2. **调用顺序**：当前命令需要调用哪些 skill。
3. **停止条件**：什么时候询问用户，什么时候终止。

写作规范、审查标准和 Git 安全规则放到 model-invoked skill。

## 怎么判断一个内容应该放在哪里

| if | then |
|---|---|
| 用户直接发出一个完整目标，例如“实施这个变更” | 放 user-invoked skill |
| 多个入口都会复用同一套规则 | 放 model-invoked skill |
| 输入到输出完全确定，不需要模型判断 | 放脚本或 adapter |
| 内容只描述 OpenSpec 路径、字段或 schema | 放 OpenSpec adapter |
| 一个 skill 生成文件后还决定下一阶段 | 把下一阶段的决定移到 user-invoked skill |

例如，`write-spec` 可以规定 spec 怎么写。

`write-spec` 不应包含“写完后调用 design，再调用 tasks”。

## model-invoked skill 的统一结构

每个 model-invoked skill 建议只保留五项：

| 项目 | 内容 |
|---|---|
| Trigger | 什么时候调用 |
| Inputs | 执行所需的信息，不绑定文件名 |
| Rules | 必须遵守的规则 |
| Output | 返回什么内容 |
| Boundaries | 不负责什么，什么时候停止 |

例如，`review-spec` 的输入应该是“行为目标、验收标准、限制条件和待审查 spec”。

不要把输入写成“必须先存在 `proposal-review.md`”。OpenSpec adapter 可以从 proposal、Issue 或用户消息中取得需要的信息。

# 现有内容怎么改

## 8 个 artifact 分成内容、过程和动作

### 保留为长期内容

| 当前 artifact | 推荐处理 | 原因 |
|---|---|---|
| `proposal` | 保留，但允许引用已有 Issue | proposal 保存目标、范围和原因 |
| `specs` | 条件生成 | 只有行为或接口发生变化时才需要 |
| `design` | 条件生成 | 只有实现决策需要被记录时才需要 |
| `tasks` | 保留 | tasks 记录需要完成什么 |

### 从固定 artifact 改成过程信息

| 当前 artifact | 推荐处理 | 原因 |
|---|---|---|
| `proposal-review` | 改成统一 review 结果；adapter 可继续写旧文件 | review 是一次判断，不是业务文档 |
| `implementation-mode` | 改成 `implement-change` 的路由结果 | mode 是运行时选择，不需要独立阶段 |
| `execution-plan` | 默认在执行期间生成；需要多人或分批执行时才保存 | tasks 已经记录工作内容，执行顺序可以动态调整 |
| `apply` | 改成 user-invoked command | apply 是动作，不是 artifact |

这样处理后，OpenSpec 不再把“文件存在”当成“流程已完成”。

## 用条件代替静态 `requires`

推荐的条件如下：

| if | then |
|---|---|
| Issue 已经包含明确目标、范围和验收标准 | 直接引用 Issue，不重复写完整 proposal |
| 变更会修改用户可见行为、API 或数据格式 | 必须生成 specs |
| 变更涉及组件边界、数据迁移、安全或难以撤销的选择 | 必须生成 design |
| tasks 已能表达执行顺序 | 不生成独立 execution-plan |
| 需要多个执行者、多个批次或明确交接 | 保存 execution-plan |
| 开始实施 | 必须有可执行 tasks 和验收标准 |
| 完成实施 | 必须执行测试和实现审查 |

旧依赖可以这样替换：

| 旧依赖 | 新要求 |
|---|---|
| `specs requires proposal-review` | 写 specs 前必须有已确认的目标、范围和验收标准 |
| `design requires specs` | 写 design 前必须有行为目标和技术限制 |
| `tasks requires design` | tasks 必须包含实施所需的决策；没有设计决策时可跳过 design |
| `apply requires execution-plan` | 实施前必须有可执行 tasks；只有复杂执行才需要 execution-plan |

核心 skill 只检查信息是否齐全。OpenSpec adapter 负责从现有 artifact 中取得信息。

## 5 道 gate 改成独立检查

gate 负责回答一个问题：**当前内容是否可以进入下一步？**

子代理仍然可以保留。子代理只是执行 review 的方式，不再决定流程结构。

每道 gate 使用相同输出格式：

| 字段 | 内容 |
|---|---|
| `status` | `pass`、`revise` 或 `blocked` |
| `findings` | 每项包含位置、问题和必须修改的内容 |
| `evidence` | 已检查的文件、命令结果或依据 |

user-invoked skill 根据结果处理：

| status | user-invoked skill 的动作 |
|---|---|
| `pass` | 继续当前命令 |
| `revise` | 返回当前产物的生成步骤 |
| `blocked` | 停止，并向用户索取缺少的信息或决定 |

review skill 不调用下一个 skill，也不直接改被审查文件。这样可以保持审查独立。

另外 4 道 gate 的名称没有提供。迁移时不用先改名。按每道 gate 的检查对象逐个处理即可。

### 可由脚本判断的 gate 不要调用子代理

| 检查内容 | 推荐实现 |
|---|---|
| 必填字段是否存在 | schema 或脚本 |
| 文件格式是否正确 | OpenSpec validator |
| 测试命令是否成功 | 脚本或 CI |
| proposal 是否清楚 | model-invoked review skill |
| design 取舍是否合理 | model-invoked review skill |
| 实现是否满足验收标准 | model-invoked review skill 加测试结果 |

固定规则放代码。需要判断的标准放 skill。

## OpenSpec 只保留三项职责

OpenSpec adapter 负责：

1. 从当前目录和 artifact 中读取目标、限制、决策和 tasks。
2. 把 skill 输出写成 OpenSpec 文件。
3. 执行 schema 校验，并生成旧格式的 review 记录。

核心 skill 不应知道：

- OpenSpec artifact 的目录结构。
- `requires` 字段。
- `proposal-review` 的具体文件名。
- 当前使用的是 OpenSpec 还是别的格式。

迁移期间只保留一份正式内容。不要同时维护“通用文档”和“OpenSpec 文档”两份副本。

## 5 个独立 skill 的处理方式

| 当前 skill | 推荐角色 | 调整方式 |
|---|---|---|
| `git-master` | model-invoked | 检查内部是否混合 commit、历史改写和故障恢复；触发条件不同就拆分 |
| `using-git-worktrees` | model-invoked | 保存创建、选择、清理 worktree 的规则；是否使用 worktree 由入口决定 |
| `github-project-flow` | user-invoked | 可以保留为交付入口；Git 操作规则移到 Git skill |
| `project-context-docs` | model-invoked | 如果读取和更新是两个独立触发条件，拆成 `read-project-context` 和 `update-project-context` |
| `project-init-workflow` | user-invoked | 保留编排职责；调用 context、Git 和 GitHub skill |

`git-master` 不要为了原子化立刻拆成许多文件。使用下面的判据：

| if | then |
|---|---|
| 几组规则总是同时使用 | 保留一个 skill |
| commit、rebase、恢复操作会被独立调用 | 按触发原因拆分 |
| 拆分后每个 skill 仍需要读取另一个 skill 才能执行 | 拆分过度，合并回来 |

如果运行器支持调用权限 metadata：

- user-invoked skill 只允许用户直接调用。
- model-invoked skill 不出现在用户命令列表。
- 不要只依赖目录名区分调用方式。

# 迁移顺序和验收方式

## 选择渐进迁移

| 方案 | 代价和风险 | 推荐 |
|---|---|---|
| 渐进迁移 | 会暂时保留 adapter 和旧 artifact，但每一步都能回退 | **推荐** |
| 一次重写 | 最终目录更快变干净，但很难判断失败来自 schema、skill 还是路由 | 不推荐作为第一步 |

## 必做的 5 步

### 1. 固定现有行为

先建立三个测试场景：

| 场景 | 预期 |
|---|---|
| 局部修复，不改外部行为 | 可以跳过 design 和独立 execution-plan |
| 修改公开 API，并改变组件边界 | 生成 specs 和 design，并执行对应 review |
| 用户提供已确认的 tasks | 直接进入实施，不重新生成 proposal |

验收结果：每个场景都能列出实际生成的 artifact、执行的 gate 和跳过原因。

### 2. 统一 gate 输出

先从 `proposal-review` 开始。

让 reviewer 输出统一的 `status`、`findings` 和 `evidence`。OpenSpec adapter 再把结果写成现有 `proposal-review` artifact。

验收结果：

- 旧流水线继续运行。
- reviewer 不调用 specs。
- review 失败时只返回问题，不改 proposal。
- `proposal-review` 文件由 adapter 生成。

### 3. 隔离 OpenSpec

把路径查找、schema 字段和文件写入移到 adapter。

model-invoked skill 改为接收内容字段，例如目标、验收标准和限制条件。

验收结果：

- model-invoked skill 中不再出现 OpenSpec 路径和 `requires`。
- adapter 之外的代码不直接写 artifact 文件。
- 现有 OpenSpec validator 仍能通过。

### 4. 加入三个用户入口

实现 `plan-change`、`implement-change` 和 `ship-change`。

迁移期间可以让旧 `apply` 指向 `implement-change`，避免立即破坏现有调用方式。

验收结果：

- 用户可以从 proposal、Issue、specs 或 tasks 中任意一个位置开始。
- `plan-change` 能根据条件跳过 design。
- `implement-change` 不会重新生成已经确认的规划内容。
- `ship-change` 复用 Git、worktree 和 GitHub skill。

### 5. 切换默认入口

新入口稳定后，停止使用 OpenSpec `requires` 驱动流程。

保留 schema 校验和文件格式。删除已经没有调用者的 `implementation-mode` 阶段和 `apply` artifact。

验收结果：

- OpenSpec 不再决定下一步。
- 任意 model-invoked skill 可以单独测试。
- 同一个 skill 可以被多个用户入口复用。
- 每项内容只有一个正式来源。

## 避免三个常见问题

1. **不要把静态链换成 skill 链。**  
   `proposal-skill → review-skill → spec-skill` 仍然是固定流水线，只是换了文件位置。

2. **不要让 review skill 顺手修文件。**  
   reviewer 只给判断和修改要求。生成内容的 skill 负责修改。

3. **不要维护两份正式文档。**  
   skill 产生内容，OpenSpec adapter 负责保存。OpenSpec 文件可以继续作为正式来源。

## 当前状态和下一步

- **已完成**：目标结构、artifact 分类、gate 合同和迁移顺序已经明确。
- **当前步骤**：迁移 `proposal-review`，暂时不改变固定流水线。
- **后续步骤**：隔离 OpenSpec，再引入三个用户入口。

**现在先做第 1 个改造 PR：让 proposal reviewer 输出统一 review 结果，再由 OpenSpec adapter 写回旧 `proposal-review` 格式；其余流水线暂时保持不变。**