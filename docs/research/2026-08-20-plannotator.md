# 1）它是什么、干什么、谁做的

**结论：权威仓库确认为 `backnotprop/plannotator`。** GitHub 搜索结果中，该仓库不是 fork，描述与产品官网一致；仓库 README 也把官网指向 `plannotator.ai`。它是一个本地运行、浏览器展示的人工 review 界面，主要用于批注 coding agent 生成的计划、spec、Markdown/HTML，以及 review 代码 diff。来源：[仓库](https://github.com/backnotprop/plannotator)、[README](https://github.com/backnotprop/plannotator/blob/main/README.md)、[官网](https://plannotator.ai/)

**作者是 Michael Ramos，GitHub 用户名 `backnotprop`。** 仓库 package 元数据和版权声明使用 `backnotprop`；其 GitHub 公开资料显示姓名为 Michael Ramos。项目采用 MIT 或 Apache-2.0 双许可证。来源：[作者资料](https://github.com/backnotprop)、[package.json](https://github.com/backnotprop/plannotator/blob/main/package.json)、[README License 段](https://github.com/backnotprop/plannotator/blob/main/README.md#license)

它不是只给 Claude Code 的小插件。当前 README 明列 Amp、Claude Code、Codex、Copilot CLI、Droid、Gemini CLI、Kiro、OpenCode 和 Pi，并包含独立 CLI、VS Code、Obsidian、GitHub/GitLab 等入口。来源：[README](https://github.com/backnotprop/plannotator/blob/main/README.md)

# 2）怎么工作：输入、网页交互、输出

**直接输入以文件为主，不是剪贴板。** 通用命令是 `plannotator annotate <目标>`；目标可为 Markdown、若干纯文本格式、HTML、URL 或文件夹。独立 CLI 会读取内容、启动本地 HTTP server、打开浏览器，并阻塞等待用户提交。官方命令文档没有 stdin 或“从剪贴板新建正文”的入口，因此剪贴板正文输入标为**未确认/未提供**；可先把内容写入临时 `.md` 文件。来源：[Annotate 命令文档](https://github.com/backnotprop/plannotator/blob/main/apps/marketing/src/content/docs/commands/annotate.md)、[CLI help 实现](https://github.com/backnotprop/plannotator/blob/main/apps/hook/server/cli.ts)

**也可以直接 review agent 的最后一条消息。** `plannotator annotate-last` / `/plannotator-last` 会从受支持 harness 的会话中取得最后一条回复；Pi 还暴露 `plan-review: { planContent, planFilePath? }` 事件接口，可直接传 Markdown 字符串，但这是 Pi extension 内的接口，不是通用 CLI 的 stdin。来源：[README Commands](https://github.com/backnotprop/plannotator/blob/main/README.md#commands)、[Pi extension README](https://github.com/backnotprop/plannotator/blob/main/apps/pi-extension/README.md#shared-plannotator-event-api)

**网页支持逐段、逐句批注。** 用户可选中文字后做删除、替换/评论、全局评论、快捷标签、“looks good”，也可附图片。前端把 Markdown 解析成块，选区批注最后导出为结构化 Markdown。来源：[Markdown File Annotation](https://github.com/backnotprop/plannotator/blob/main/apps/portal/ANNOTATE.md)、[Annotate 命令文档](https://github.com/backnotprop/plannotator/blob/main/apps/marketing/src/content/docs/commands/annotate.md)

**反馈回传有三种稳定方式。** 默认模式把结构化 Markdown 打到 stdout；`--gate --json` 输出单行 `{ "decision": "approved|annotated|dismissed", "feedback": "..." }`；`--hook` 在要求修改时输出 Claude Code/Codex 原生的 `{ "decision":"block", "reason":"..." }`。严格 gate 还可用 `--require-approval` 和 `--result-file`，以退出码区分批准、拒绝和环境错误。来源：[Annotate Gates and JSON Responses](https://github.com/backnotprop/plannotator/blob/main/apps/marketing/src/content/docs/guides/annotate-gates-and-json-responses.md)

在原生 plan-mode 集成中，agent 提交计划后网页打开；批准则继续，拒绝则把批注送回同一会话；再次提交时会显示 plan diff。来源：[README How it works](https://github.com/backnotprop/plannotator/blob/main/README.md#how-it-works)、[Pi extension README](https://github.com/backnotprop/plannotator/blob/main/apps/pi-extension/README.md#plan-mode)

# 3）安装和接入：通用 CLI、Pi 与 OMP

**安装有完整 installer，也可只装 CLI。** macOS/Linux/WSL 使用官方 shell installer，Windows 使用 PowerShell installer；`--minimal` 只安装 `plannotator` 二进制到 `~/.local/bin`，不改任何 agent 配置。installer 从 GitHub Releases 下载 binary，支持固定版本；发行物有 SHA256，v0.17.2 起有 SLSA provenance。来源：[README Install](https://github.com/backnotprop/plannotator/blob/main/README.md#install)、[README Security](https://github.com/backnotprop/plannotator/blob/main/README.md#security)

**支持非 Claude Code 环境。** 除多种官方插件外，独立 CLI 本身就是通用边界：任何 workflow 只要能准备一个文件、启动阻塞命令并读取 stdout/JSON，就能接入 `plannotator annotate plan.md --gate --json`。对有生命周期 hook 的 agent，可用 `plannotator annotate <file> --hook`。来源：[Standalone CLI](https://github.com/backnotprop/plannotator/blob/main/apps/marketing/src/content/docs/commands/annotate.md#standalone-cli-outside-an-agent-session)、[Hook Integration](https://github.com/backnotprop/plannotator/blob/main/apps/marketing/src/content/docs/guides/hook-integration.md)

**Pi 有官方原生 extension。** 安装命令是 `pi install npm:@plannotator/pi-extension`；它提供 `--plan`、`/plannotator-plan-mode`、`plannotator_submit_plan`、计划/执行状态机、反馈注入，以及批准后交给外部编排器的 `executionMode: external`。来源：[Pi extension README](https://github.com/backnotprop/plannotator/blob/main/apps/pi-extension/README.md)

**OMP 没有官方专用集成，是否能直接加载 Pi extension 为“未确认”。** 官方支持列表、仓库 topic 和安装表都只写 Pi，没有写 Oh My Pi/OMP；仓库 issue 搜索也没有找到 OMP 集成说明。因此不能把“Pi 支持”直接等同于“OMP 原生支持”。来源：[README Integrations/Install](https://github.com/backnotprop/plannotator/blob/main/README.md#install)、[仓库 topics](https://github.com/backnotprop/plannotator)

**OMP 的稳妥接法是薄 CLI 适配，而不是赌 Pi extension 兼容。** workflow 把计划或临时文字写到稳定 `.md`/临时文件，运行 `plannotator annotate "$file" --gate --json --require-approval`，解析单行 JSON，再把 `feedback` 注入 agent；若 OMP 有兼容 Stop/PostToolUse hook，可改用 `--hook`。这是基于官方 CLI 协议的接入推断，不是官方 OMP 配方。来源：[结构化 gate 协议](https://github.com/backnotprop/plannotator/blob/main/apps/marketing/src/content/docs/guides/annotate-gates-and-json-responses.md)、[Hook Integration](https://github.com/backnotprop/plannotator/blob/main/apps/marketing/src/content/docs/guides/hook-integration.md)

# 4）活跃度和成熟度

**活跃度高。** 截至 2026-08-20，仓库约 7,914 stars、585 forks、133 个 open issues，未归档；GitHub 显示仓库当天仍有更新。8 月 15–18 日可见多笔功能、修复和 Pi 集成提交，8 月 17 日版本升至 `0.27.4`。来源：[仓库](https://github.com/backnotprop/plannotator)、[v0.27.4 提交](https://github.com/backnotprop/plannotator/commit/2a22e5805aac6296ce925a3aa2e629d8f01c9d76)、[近期 Pi 修复](https://github.com/backnotprop/plannotator/commit/8d735cb7dd2bd7b61074b36bbc5871b3dc994f1b)

**工程成熟度高于普通个人插件，但仍在快速变化期。** 有跨平台 installer、卸载/清理机制、版本固定、SHA256、SLSA provenance、SBOM、多个 agent 的专门 README、远程/SSH 配置和结构化自动化协议。来源：[README](https://github.com/backnotprop/plannotator/blob/main/README.md)、[package.json](https://github.com/backnotprop/plannotator/blob/main/package.json)

**文档总体完整，但存在“代码仓库文档与新官网文档分离”的风险。** issue #1086 说明公开 docs 已迁到单独的私有源仓库，旧仓库内的 marketing 文档不一定就是线上最新版；该 issue 也确认文件批注会保存历史副本，而相关隐私说明当时尚未补齐。来源：[issue #1086](https://github.com/backnotprop/plannotator/issues/1086)

成熟度判断：**可用于真实个人 workflow，但应固定版本并做一次 OMP 端到端试跑。** 依据是已有稳定 CLI/JSON 契约，同时官方仍把 OpenCode 2 标为 experimental，部分宿主兼容性仍在复核。来源：[OpenCode README](https://github.com/backnotprop/plannotator/blob/main/apps/opencode-plugin/README.md)、[issue #904](https://github.com/backnotprop/plannotator/issues/904)

# 5）与“网页逐段 review 计划/文字”的匹配度和短板

**核心需求高度匹配。** 它原生展示 Markdown，允许用户对具体选区做删除、评论和标签，支持全局意见；提交后输出可直接交给 agent 的结构化 Markdown/JSON。`--gate` 又把“批准、要求修改、关闭”变成明确决策，因此不仅能批注，也能充当 workflow gate。来源：[Annotate 命令文档](https://github.com/backnotprop/plannotator/blob/main/apps/marketing/src/content/docs/commands/annotate.md)、[Gate 文档](https://github.com/backnotprop/plannotator/blob/main/apps/marketing/src/content/docs/guides/annotate-gates-and-json-responses.md)

**它也适合“不进 PR 的产出”。** 输入可以是任意本地 Markdown/纯文本文件、agent 最后一条消息或整个文档目录，不依赖 Git diff 或 PR；官方还明确把 spec-kit、Kiro、OpenSpec 产生的 `spec.md`、`plan.md`、`tasks.md` 等列为主要用例。来源：[Gate 文档 Primary use cases](https://github.com/backnotprop/plannotator/blob/main/apps/marketing/src/content/docs/guides/annotate-gates-and-json-responses.md#primary-use-cases)

已知短板：

1. **OMP 要自己接回传。** 没有官方 OMP plugin；CLI 能产出反馈，但仍需 workflow/skill 负责写文件、启动命令、解析决定、重投给 agent。来源：[支持列表](https://github.com/backnotprop/plannotator/blob/main/README.md#install)、[CLI JSON 协议](https://github.com/backnotprop/plannotator/blob/main/apps/marketing/src/content/docs/guides/annotate-gates-and-json-responses.md)
2. **没有已记录的剪贴板/标准输入正文入口。** 临时文字需要先落成文件，或由受支持 harness 的 `annotate-last` 读取会话。来源：[Annotate 输入表](https://github.com/backnotprop/plannotator/blob/main/apps/marketing/src/content/docs/commands/annotate.md#what-you-can-annotate)
3. **浏览器与远程/多会话可达性仍有缺口。** open issue 记录了无 TTY、SSH、多 session、浏览器未正常打开时的 URL/路由问题；本地重投还可能换随机端口。固定 `PLANNOTATOR_PORT` 是当前 workaround。来源：[issue #899](https://github.com/backnotprop/plannotator/issues/899)、[issue #1134](https://github.com/backnotprop/plannotator/issues/1134)
4. **本地历史不是完全无状态。** eligible 的本地文件会在打开时复制到 `~/.plannotator/history/` 并默认长期保存；敏感临时产出要设置 `PLANNOTATOR_ANNOTATE_HISTORY=0`，且该行为的公开文档曾不完整。来源：[issue #1086](https://github.com/backnotprop/plannotator/issues/1086)
5. **跨人异步分享不是本需求下最稳的部分。** README 称旧 link sharing 正走向 deprecated、Workspaces 是主方向；另有 open bug 指出 short link 可能不包含新批注。单人本地 review 不受此问题影响。来源：[README Link sharing](https://github.com/backnotprop/plannotator/blob/main/README.md#link-sharing)、[issue #798](https://github.com/backnotprop/plannotator/issues/798)

**能否用作通用 plan review 工具：部分能——通用 CLI、网页逐段批注和结构化 JSON 已满足核心需求，但 OMP 仍需一个薄适配层来落文件并把反馈送回 agent。**
