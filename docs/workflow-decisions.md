# Workflow 决策记录

> 模型主动记,不等提醒。每条:决定了什么 + 为什么。
> 2026-08-15 起。

## D1 一切皆 skill,去掉 OpenSpec

- 决定:workflow 全部拆成 skill,不用 OpenSpec。
- 为什么:OpenSpec 和 skill 职责重叠,反复卡在它上面。planning 改用 spec + 小票。

## D2 骨架用 Matt Pocock skills v1.2.3

- 决定:直接拿 Matt 的骨架当地基,在上面加自己的东西。
- 为什么:成熟、原子化、按调用者分层。不重造。

## D3 先搭 workflow 基建,不是先交平台

- 决定:虽然有一周半的紧急平台任务,仍先搭基建。
- 为什么:一天只有 6 小时清醒工时,同步编码搭不完。后台 agent 一直跑能把墙钟时间撑开。
- 风险:基建本身可能吃掉比省下的更多时间。对策见 D4——只搭最短路径。

## D4 第一块基建 = 异步 agent 领 GitHub 任务、提 PR

- 决定:基建先只搭这一条——后台 agent 从 GitHub 领任务、干活、提 PR。这是加速引擎本身。
- 为什么:守护进程和 PI 迁移是优化,先手动顶替;这条不跑起来,后台 agent 无从谈起。
- 架构已明确是**本机**:GitHub 当任务队列和 PR 界面。容器和守护进程原为设想,已被 D5 推翻,见 D5。

## D5 只隔离 workflow,不做多容器隔离

- 决定:砍掉多容器隔离(Docker development container + 守护进程编排器 + 按需启容器)。任务隔离用 git worktree 就够。
- 为什么:多容器是想法箱里最重的基建,死线前净亏。worktree 隔离是现成资产(_backup/skills/using-git-worktrees)。
- 影响 D4:引擎从"后台 agent + Docker 容器 + 守护进程"收缩为"后台 agent + git worktree 隔离"。不需要容器,不需要守护进程。

## D6 评估 Multica 作为基建底座

- 决定:先实测开源平台 Multica(multica-ai/multica),不自己从零搭守护进程/容器。
- 为什么:它已现成实现想法箱里的基建——异步 agent 领 GitHub issue、本机 daemon 编排、worktree 隔离(D5)、原生支持 omp runtime、代码不离开本机。
- 硬约束:必须验证**能给每个 agent/任务指派模型**。这是切 PI 的原始动因;若 Multica 满足,可能不用切 harness。
- 已知风险(真实差评 issue #1579):完成状态靠 agent 自报不靠验证;多 agent 复杂任务结果不可信。→ 编排用 Multica,质量仍靠自建 skill(TDD、code-review、黑盒验收)。
- 待验证清单:见 docs/multica-eval.md。

## D7 harness 用 omp,放弃 PI

- 决定:留在 omp(Oh My Pi),不迁 PI。
- 为什么:切 PI 的唯一动因是"主 agent 给 sub agent 指定模型",omp 已原生支持(见 D8)。PI 的插件碎片、劫持 edit 工具冲突问题因此都不用面对。
- 附加收获:omp 的 task 工具自带 worktree 隔离(合 D5)、后台 async job(合异步工作模式)、agent hub 群聊式协作(合想 raft.build 的诉求)。

## D8 模型指派 = 声明式 agent + modelRoles

- 决定:每个 sub agent 的模型靠"自定义 agent 绑角色 + config.yml 里 modelRoles 映射到具体模型"实现。主 agent 按 agent 名字派活,模型随之确定。
- 机制:agent frontmatter 写 `model: "@review"`;config.yml 写 `modelRoles.review: 具体模型`。优先级:task.agentModelOverrides → agent frontmatter → 父 session。
- 边界:声明式,不是运行时任意点名。主 agent 只能派"已定义过的带模型 agent",不能临时指定一个没定义过的模型。对本工作流够用。
- 合并想法箱第 3 节:"模型能力清单"skill = 这批 agent 定义 + modelRoles,两件事合成一件,不再单独做清单。

## D9 Multica 是任务台,GitHub 只托管代码

- 决定:Multica 的 issue 是它自己的系统(编号如 MUL-123),不是 GitHub issue。需求在 Multica 提,不在 GitHub。
- 数据边界:Multica 存 issue/评论/状态/agent 配置/task 记录;本机存代码/凭证/实际改动;GitHub 只存 PR 和 CI。
- 协作链:Multica 建 issue → 派给本机 omp agent → agent 改代码提 PR 到 GitHub → PR 分支名/标题带 MUL-123 → Multica 自动关联 → PR 带 Closes 合并后 Multica 自动标 done。
- 纠正旧设想:不是"全押 GitHub"。GitHub 只承担代码托管那一半。

## D10 文档分两类:Spec 和 Context

- 决定:平台文档按业界成熟结构分两类,不混在一起。
- Spec(这次要做什么,可归档):docs/prd/ 放平台级整体愿景;specs/<功能>/ 放拆出的每个功能。
- Context(项目长期为真):AGENTS.md 放 agent 每次读的规则;CONTEXT.md 放领域术语和架构;docs/adr/ 放架构决策。
- 依据:Spec Kit + Kiro + Diataxis 的成熟做法(Martin Fowler:spec 是"这次做什么",context 是"代码库一直是什么样")。
- 落地:write-prd 写 docs/prd/;to-tickets 拆 specs/;domain-modeling + project-context-docs 维护 CONTEXT.md 和 ADR。
