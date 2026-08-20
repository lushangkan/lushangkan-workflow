# 现成 Agent Skill 调研

> 查询日期：2026-08-20。`star` 是查询时的仓库总星数，不是单个 skill 的星数。活跃度优先采用 GitHub `pushed_at`；无法取得精确日期时会明确标注。

## 搜索范围与判断标准

[skills.sh](https://www.skills.sh/) 确实存在。它是 Vercel 维护的 Agent Skills Directory，支持 Claude Code、Cursor、Codex 等 harness。技能页 URL 为 `https://www.skills.sh/<owner>/<repo>/<skill>`；网页可直接搜索，也可运行 `npx skills find <关键词>`。官方 `find-skills` 原文见 [vercel-labs/skills](https://github.com/vercel-labs/skills/blob/main/skills/find-skills/SKILL.md)。生产站的 [`/api/search?q=`](https://www.skills.sh/api/search?q=html-report) 也可检索；不要把自部署 `skills-api` 的 `/api/skills?query=` 当成生产站接口。

本次同时搜索了 skills.sh、GitHub 的 `SKILL.md`、Claude/Agent Skills 清单，以及 [obra/superpowers](https://github.com/obra/superpowers)。候选必须能读到实际 skill 或插件原文。普通工具、文章、只有搜索摘要但原文已删除的条目不算现成实现。

判断分三档：

- **可直接引用**：核心约束已经写进 skill，最多改路径和 harness 调用方式。
- **接近可改造**：有可靠实现，但缺少一项关键约束，或绑定另一套 harness。
- **没有只能自建**：没有可信的 skill 级实现。

## 1. HTML 报告

需求是把一次复杂分析渲染为便于阅读的 HTML，不是建设文档站。

| 候选 | 仓库 / 维护者 | 最近活跃度 | Stars / 许可证 | Fit |
|---|---|---:|---:|---|
| **readout** | [warpdotdev/common-skills](https://github.com/warpdotdev/common-skills/blob/main/.agents/skills/readout/SKILL.md) / Warp | 2026-08-18 push | 163 / MIT | **很高**。把当前对话或新调查做成单个、自包含 HTML；支持子 agent、代码引用、统一模板和本地索引。默认写到 `~/.readouts`，不会变成项目文档站。|
| **claude-html-report-skill** | [voidful/claude-html-report-skill](https://github.com/voidful/claude-html-report-skill) / voidful | 2026-05-09 push | 19 / MIT | **很高**。单文件交互报告，支持 Chart.js、Plotly、筛选表格、tabs、折叠区、Mermaid、KaTeX、暗色和打印样式。发布到 GitHub Pages 是可选附加流。|
| **Report-Skill** | [TheoRata/Report-Skill](https://github.com/TheoRata/Report-Skill) / TheoRata | 2026 年新仓库；12 commits | 5 / MIT | **高**。长报告转设计型单文件 HTML，带 sticky TOC、主题、callout、脚注、批注和导回 Markdown；尚未被 skills.sh 收录，维护规模小。|
| **a4-report** | [Pysamlam/a4-report](https://github.com/Pysamlam/a4-report) / Pysamlam | 2026 年；10 commits | 8 / MIT | **中高**。零依赖、自包含、咨询报告风格，打印可直接转 PDF；更偏 A4 商务报告。|

**判断：可直接引用。** 首选 Warp 的 `readout`：维护最活跃，恰好覆盖“把本次分析做成单页 HTML”。如果更看重图表和交互控件，选 voidful。

## 2. ADR + 领域词汇表 + C4 + Mermaid

没有发现一个 `SKILL.md` 同时覆盖 ADR、领域词汇表、C4 和 Mermaid。现成生态分成“架构记录/图”和“统一语言”两组。

| 候选 | 仓库 / 维护者 | 最近活跃度 | Stars / 许可证 | Fit |
|---|---|---:|---:|---|
| **c4designer + adr-scribe** | [muthub-ai/c4-skills](https://github.com/muthub-ai/c4-skills) / muthub-ai | 2026-04-26 push | 3 / MIT | **高但不完整**。两个标准 `SKILL.md` 分别负责 Mermaid C4 和 MADR ADR，含设计、反向记录、审查模式及验证脚本；缺领域词汇表。|
| **enterprise-architecture-skill** | [gauravs19/enterprise-architecture-skill](https://github.com/gauravs19/enterprise-architecture-skill) / gauravs19 | 2026 年活跃；有 changelog、release、eval | 7 / MIT | **高但不完整**。覆盖 C4、Structurizr、Mermaid/PlantUML、arc42、ADR/MADR，并有一致性检查；仍缺领域词汇表。|
| **architecture-decision-records 等组合** | [wshobson/agents](https://github.com/wshobson/agents) / W. Hobson | 2026 年持续维护 | 38,943 / MIT | **中高**。大型技能集合中有 ADR 与 C4 单项；skills.sh 的 [ADR 条目](https://www.skills.sh/wshobson/agents/architecture-decision-records) 安装量高，但不是四项一体。|
| **domain-modeling / ubiquitous-language** | [mattpocock/skills](https://github.com/mattpocock/skills) / Matt Pocock | 2026-08-19 push | 224,600 / MIT | **词汇表最佳补件**。`domain-modeling`/`ubiquitous-language` 负责领域术语和上下文，不负责 ADR 与 C4。skills.sh：[domain-modeling](https://www.skills.sh/mattpocock/skills/domain-modeling)。|

**判断：接近可改造。** 不必从零写：采用 `c4designer + adr-scribe`，再复用 Matt 的领域词汇规则即可。需要自建的只是一个很薄的编排 skill，负责把三份产物放到约定位置并保持链接；不要复制三个底层 skill 的规则。

## 3. 黑盒验收测试

这里严格区分“实施 agent 自己跑测试”和“source-blind 验收 agent”。

| 候选 | 仓库 / 维护者 | 最近活跃度 | Stars / 许可证 | Fit |
|---|---|---:|---:|---|
| **behavior-validator** | [openclaw/agent-skills](https://github.com/openclaw/agent-skills/blob/main/skills/behavior-validator/SKILL.md) / openclaw | 2026-08-18 push | 1,058 / MIT | **精确匹配**。明文禁止查看源码、diff、测试和 git 历史；只经 CLI、浏览器、API 或生成物验收；可从原始请求形成契约，并提供隔离工作区、反作弊探测及 pass/fail/blocked 报告。skills.sh：[条目](https://www.skills.sh/openclaw/agent-skills/behavior-validator)。|
| **agent-testing** | [lobehub/lobehub](https://github.com/lobehub/lobehub/blob/main/.agents/skills/agent-testing/SKILL.md) / LobeHub | 2026-08-20 push | 82k / 未识别标准许可证 | **中等**。通过 CLI、浏览器、Electron 做真实界面 E2E，并保留截图证据；但会读 `package.json`、README、CI 来建立适配器，不满足“不读源码/仓库”的硬约束。|
| **acceptance-testing** | [terraphim/opencode-skills](https://github.com/terraphim/opencode-skills/blob/main/skill/acceptance-testing/SKILL.md) / terraphim | 2026-01，约 7 个月未活跃 | 1 / Apache-2.0（skill 自述） | **低到中**。强调用户结果、Gherkin 和 sign-off，但没有 source-blind 门禁，且维护弱。|

[obra/superpowers 的 `verification-before-completion`](https://github.com/obra/superpowers/blob/main/skills/verification-before-completion/SKILL.md) 只是要求实施方用新鲜证据自证，不是零上下文黑盒验收，故不作为候选。

**判断：可直接引用。** `behavior-validator` 已经把最关键的“不许读源码”写成硬规则，只需把输入接口固定成“原始需求 + 运行入口 + 成功标准”。

## 4. 反向拷问 / 理解确认

搜索时排除了需求盘问、课程测验、rubber duck 和“解释代码”技能。目标是计划已经形成后，再考用户是否理解实现方案。

| 候选 | 仓库 / 维护者 | 最近活跃度 | Stars / 许可证 | Fit |
|---|---|---:|---:|---|
| **plan-understanding-quiz** | [PolyArch/humanize](https://github.com/PolyArch/humanize/blob/main/agents/plan-understanding-quiz.md) / PolyArch、Sihao Liu | 2026-04；相关提交约 2026-03 | 1.4k / **未见 LICENSE** | **很高，但不是 SKILL.md**。计划后生成两道多选题，检查核心机制、系统连接与约束；答错先解释，再让用户决定停止或继续。它是 Claude Code 插件内 agent。|
| **quiz-me** | [emanzurv/quiz-me-skill](https://github.com/emanzurv/quiz-me-skill) / emanzurv | 2026-08-13 push | 3 / MIT | **高但场景窄**。Claude 诊断 root cause 和 fix 后先考用户；答错换角度重讲，全对才允许写代码。定位偏 bug fix，而非任意计划。|
| **quiz-gate** | [nbbaier/agent-skills](https://github.com/nbbaier/agent-skills/blob/main/skills/quiz-gate/SKILL.md) / nbbaier | 2026-07 | 13 / 未指定 | **中**。在实现后、commit/merge 前考决策、失败模式、流程和边界；时点晚于目标，而且 skills.sh 条目已消失。|

Matt 的 `grilling`、Superpowers 的 brainstorming，以及常见 plan-interview 都是向用户收集需求，不符合本项。课程类 `check-understanding` 虽然会出题，也没有“针对刚形成的开发计划”这一契约。

**判断：接近可改造。** 社区已有几乎同构的行为，但最佳实现是 agent 文件而非独立 skill。建议抽取 `plan-understanding-quiz` 的两题门禁与答错处理，封装成自己的小 `SKILL.md`；不需要重做题型设计。

## 5. 反完美主义 / 防过度投入

完整定义包括四部分：识别决策点、给推荐默认值、提醒耗时、防 gold-plating 与 scope creep。

| 候选 | 仓库 / 维护者 | 最近活跃度 | Stars / 许可证 | Fit |
|---|---|---:|---:|---|
| **governor** | [cwinvestments/memstack](https://github.com/cwinvestments/memstack/blob/master/skills/governor/SKILL.md) / cwinvestments | 2026-08-04 push | 414 / MIT | **最高**。用 Prototype/MVP/Production tier 限制投入；范围越级时形成决策点并尊重用户 override；tier 不明默认 Prototype；有 anti-pattern 防镀金。缺 wall-clock 提醒。|
| **coding-discipline** | [viktorbezdek/skillstack](https://github.com/viktorbezdek/skillstack/blob/main/coding-discipline/README.md) / viktorbezdek | 2026-06-05 push | 10 / MIT | **中高**。Simplicity First、Surgical Changes、Goal-Driven Execution、iteration budget 和 stop-and-report；缺 wall-clock 提醒，也不是标准 `SKILL.md`。|
| **karpathy-guardrails** | [LucasDuys/forge](https://github.com/LucasDuys/forge/blob/main/skills/karpathy-guardrails/SKILL.md) / LucasDuys | 2026-07-15 push | 55 / MIT | **中高**。覆盖歧义决策、简单优先、范围追踪和目标驱动；绑定 Forge 流程，缺推荐默认与耗时提醒。|
| **agent-deadline-setup** | [kochetkov-ma/claude-brewcode](https://github.com/kochetkov-ma/claude-brewcode/blob/main/brewtools/skills/agent-deadline-setup/SKILL.md) / kochetkov-ma | 2026-08 持续发布 | 31 / MIT | **耗时补件**。hook 在预算 80% 提醒收尾、100% 禁止非收尾工具；默认 20 分钟。当前仅 macOS，Linux 尚在计划中。|

[avoid-scope-creep](https://www.skills.sh/aiming-lab/metaclaw/avoid-scope-creep) 只是一条“只改用户要求内容”的薄规则，不含决策点、默认值和耗时提醒，因此不是完整候选。Superpowers 虽贯彻 YAGNI，也没有本项专项 skill。

**判断：接近可改造。** `governor` 已覆盖三项中的大部分设计；社区真正的缺口是跨平台耗时提醒。可引用 governor 的 tier 和默认值，但 Linux 上的计时 hook 仍需自己做，或先只采用软提醒。

## 6. GitHub milestone 驱动任务流

目标是使用 GitHub 原生 milestone：issue 归属 milestone，再从 milestone 领票并交付。

| 候选 | 仓库 / 维护者 | 最近活跃度 | Stars / 许可证 | Fit |
|---|---|---:|---:|---|
| **milestone-workflow** | [richkuo/rk-skills](https://github.com/richkuo/rk-skills/blob/main/skills/milestone-workflow/SKILL.md) / richkuo | 2026-08-19 push | 46 / MIT | **很高**。读取 milestone issues，建依赖图，等用户批准执行计划，再逐 issue 实现、提 PR、审查、合并和 release。依赖 Claude Code Workflow 工具及特定 issue Execution block。|
| **milestone-driver / solve-milestone** | [kenmulford/milestone-driver](https://github.com/kenmulford/milestone-driver/blob/main/skills/solve-milestone/SKILL.md) / kenmulford | 2026-08-19 push | 2 / MIT | **很高**。按 milestone 名取 open issues，triage 后用 worktree 并行、test-first、review、CI 和 integration branch 完成交付；必装 obra/superpowers，外部消费者 wiring 仍在进行。|
| **milestone-management** | [troykelly/claude-skills](https://github.com/troykelly/claude-skills/blob/main/skills/milestone-management/SKILL.md) / troykelly | 2026-01-13 push | 11 / **无许可证** | **管理向**。支持创建、更新、关闭 milestone，把 issue 批量归属 milestone，以及进度和延期决策；不负责领票实现。|
| **github-project-management** | [ruvnet/agentic-flow](https://github.com/ruvnet/agentic-flow/blob/main/.claude/skills/github-project-management/SKILL.md) / ruvnet | 2026-07-30 push | 797 / **无许可证** | **中等**。支持 issue-to-swarm、milestone-track 和 board 同步，但重依赖 ruv-swarm/claude-flow MCP；milestone 主要是跟踪层。|

常见 `github-issue-to-pr`、issue-fix-flow 只会从指定 issue 开工；常见 milestone-management 又只做归属和统计。上表前两项是少数真正从整组 milestone 领票执行的 skill。

**判断：接近可改造。** `milestone-workflow` 的领域流程已完整，不需要重造；但它绑定 Claude Code Workflow。移植到 Oh My Pi 时，应保留 GitHub milestone、依赖图、逐 issue 交付这三部分，把调度调用改成 omp 的 `task`、worktree 和 PR 流程。

## 总结

六项能力中：

- **2 项可直接引用**：HTML 报告、黑盒验收。
- **4 项已有可靠实现，可小幅组合或移植**：架构记录、理解确认、防过度投入、milestone 任务流。
- **0 项需要完全从零自建**。

如果把“有现成”限定为“不改语义、直接安装”，答案是 **2/6**；如果把可移植的现成实现也算入 build-vs-adopt 的“adopt”，答案是 **6/6 都有可复用实现**。真正需要自己维护的只是四个薄层：架构产物编排、计划测验封装、Linux 计时提醒、omp 调度适配。