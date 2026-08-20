# R3 调研：市面"异步 coding agent 派活"产品扫描

调研日期：2026-08-19。调研方法：web 搜索 + 官方文档，每条结论附来源链接。查不到的明确标注。

## 结论先行

1. "issue 提需求 → 后台 agent 干活 → PR 交回"这个形态，2025–2026 已经是主流产品类目，云端 SaaS 一大排（Copilot coding agent、Cursor Cloud Agents、Devin、Jules、Codex cloud、Codegen、Factory、Sweep）。
2. 满足"本机跑 + 自选模型 + 代码不离本机"的现成产品很少，主要四个：Multica、OpenHands、Claude Code Action（配 self-hosted runner）、raft.build；另有几个极轻量的开源小项目（kraken、cavil-loop）。
3. 对照用户需求（本机、异步、GitHub 中心、轻量）：没有完全命中的现成产品。最接近的是 Multica（功能全但确实重）和 cavil-loop / kraken（够轻但都是个人小项目，成熟度低）。OpenHands 居中。

## 对比总表

| 产品 | 开源/商用 | 本机还是云端 | 怎么领任务 | PR 工作流 | 代码出不离开本机 |
|---|---|---|---|---|---|
| Multica | 开源（自托管） | 本机 daemon | 自家 issue 系统（可接 GitHub/GitLab/Gitea） | agent 完成后回 issue 交 review-ready 结果 | 不离本机（官方明示） |
| OpenHands | 开源 | 本机/云端都行 | GitHub/GitLab issue 打 `fix-me` 标签或 @openhands-agent | 自动开 draft PR 或推分支 | 可不离本机（本地跑） |
| Claude Code GitHub Action | 开源 action + 商用模型 | 本机（self-hosted runner） | issue/PR 里 @claude 或打标签 | 自动建分支、开 PR、跑 lint/test | 代码留在 runner 上，但模型请求走 Anthropic API |
| raft.build | 商用 SaaS（开源状态未确认） | 本机 daemon | 自家频道/频道对话，不是 issue 中心 | 有工程团队用例，PR 流程细节未确认 | 官方宣称不离本机 |
| kraken | 开源（个人项目） | 本机 | GitHub Issues 当任务队列，worker 认领 | 开 draft PR | 不离本机 |
| cavil-loop | 开源（个人项目） | 本机 | GitHub label（pending/agent）触发 | 每 issue 一个 worktree 分支，完成开 PR | 不离本机（官方明示） |
| GitHub Copilot coding agent | 商用（付费订阅） | 云端（GitHub Actions） | 直接把 issue assign 给 Copilot | 自动开 draft PR、@copilot 评论迭代 | 离开（在 GitHub 云端跑） |
| Cursor Cloud Agents | 商用 | 云端 VM | Cursor 里发起 / Slack / issue | 完成开 PR | 离开 |
| Devin | 商用 | 云端 | Slack / 自家 IDE / issue | 直接产 PR | 离开 |
| OpenAI Codex cloud | 商用 | 云端 | GitHub / Linear / Slack 发起 | 完成后开 PR | 离开 |
| Google Jules | 商用（有免费档） | 云端（Google VM） | issue 打 `jules` 标签 | 自动产 diff、开 PR | 离开 |
| Codegen | 商用 | 云端沙箱 | GitHub @codegen-agent / Slack / Linear / Jira | 自动开 PR + 自动修 CI | 离开 |
| Factory Droids | 商用 | 云端为主，企业档有 on-prem/self-hosted | Linear / Jira / GitHub Issues 指派 | issue → 带代码、测试、文档的 PR | 默认离开；企业档可不离开 |
| Sweep | 商用 | 云端 | GitHub 内 assign 给 Sweep | 多文件编辑后开 PR | 离开 |
| Tembo | 商用 | 云端沙箱 | GitHub/GitLab/Sentry/Linear/Slack 事件或 webhook 触发 | 产 merge-ready PR | 离开 |

## 各产品备注

### Multica（multica-ai/multica）

- 开源，可整套自托管（Docker Compose 或 Helm）；支持 GitHub、GitLab、Gitea、Forgejo。来源：https://github.com/multica-ai/multica
- 架构：Next.js 前端 + Go 后端 + PostgreSQL 17（带 pgvector）+ 本机 daemon。daemon 自动探测本机已装的 20+ 种 agent CLI（Claude Code、Codex、Cursor、OpenCode、Qwen 等），每 3 秒轮询服务器领任务，在隔离 workspace 里跑，结果流式回传。来源：https://github.com/multica-ai/multica/blob/main/CLI_AND_DAEMON.md、https://multica.ai/docs/daemon-runtimes
- 官方明示：本机工具和凭证留在本机，Multica 不执行命令、不上传整个工作目录。来源：https://multica.ai/docs/daemon-runtimes
- 明显缺点：技术栈重（要养一套 Next.js + Go + Postgres），用户已判断"太重"，此调研与该判断一致。另外它有自己的一套 issue/工作台体系，不是纯 GitHub 中心。来源：https://github.com/multica-ai/multica

### OpenHands（OpenHands/OpenHands）

- 开源，自托管优先；官方带 resolver：GitHub/GitLab/Bitbucket/Azure DevOps 的 issue 打 `fix-me` 标签或评论 `@openhands-agent` 即触发，agent 分析后开 draft PR（可选 branch/draft/ready 三种产出）。来源：https://github.com/OpenHands/OpenHands/blob/main/openhands/resolver/README.md
- 模型自选：通过 `LLM_MODEL`/`LLM_API_KEY`/`LLM_BASE_URL` 环境变量接任意 OpenAI 兼容端点（官方推荐 Claude/GPT-4 级）。来源：同上
- 本机跑可行：resolver 可以本地命令行直接跑（`python -m openhands.resolver.resolve_issue`），不一定要起整套平台。来源：同上
- 明显缺点：整套平台（Agent Canvas + 多后端）本身也不轻；resolver 单用可以，但 issue→PR 的成功率社区评价参差（本次未找到权威横评数据，标注为未核实）。来源：https://github.com/OpenHands/OpenHands/

### Claude Code GitHub Action（anthropics/claude-code-action）

- 官方开源 action。触发方式：@claude 提及、issue assign、打标签。跑在 GitHub Actions 上；配 self-hosted runner 时代码就在自己机器上跑，模型请求走 Anthropic API（或 AWS Bedrock / Google Vertex / Microsoft Foundry）。来源：https://github.com/anthropics/claude-code-action、https://code.claude.com/docs/en/github-actions
- PR 工作流现成模板：打 `plan` 标签 → Claude 回实施计划评论；打 `do` 标签 → 建分支、跑 lint/type-check、开 PR（带 Closes #N、Conventional Commits）。来源：https://storyie.com/blog/claude-code-action-github-issues
- 明显缺点：模型锁 Anthropic 一家（Bedrock/Vertex 本质也是 Claude）；不能配 PI 或本地模型；需要维护 self-hosted runner 的安全（runner 能拿到 repo 写权限 token）。来源：https://github.com/anthropics/claude-code-action

### raft.build

- 商业产品（Botiverse 公司，2025 年成立；原名 Slock）。核心形态是"人类 + agent 在持久频道里实时协作"，agent 有名字、记忆、身份，跑在你自己电脑的轻量 daemon 上，官方宣称数据不离开你的基础设施。来源：https://raft.build/
- 支持多 runtime（Claude、Codex、Kimi 等，按 agent 各选各的）。来源：https://raft.build/
- 有工程团队用例（PM、工程师、reviewer 组队，issue→PR），但领任务入口是频道对话，不是 GitHub issue 中心；PR 流程细节公开文档里查不到。来源：https://raft.build/resources/use-cases/engineering-team/
- 是否开源：官网和 GitHub org（github.com/botiverse）未找到开源仓库声明，标注为未确认，倾向"闭源 SaaS + 本机 daemon"。来源：https://raft.build/
- 明显缺点：形态是"聊天室里的队友"，和用户要的"issue 提需求→PR 交回"的 GitHub 中心形态不重合；闭源 + 早期公司，锁定风险。

### kraken（rafael-adcp/kraken）

- 开源个人项目。GitHub Issues 当任务队列：issue 即任务，worker（Claude Code、Copilot CLI 或任何实现协议的工具）原子认领，在自备环境里干活，开 draft PR 交回；issue timeline 即审计记录。来源：https://github.com/rafael-adcp/kraken
- 零基础设施：kraken 只提供协议和 skill 脚本，不带服务器；环境、沙箱、密钥全部自备。来源：https://github.com/rafael-adcp/kraken/blob/main/README.md
- 明显缺点：个人项目、早期；headless worker（无人值守）还是 open issue（#57）。来源：https://github.com/rafael-adcp/kraken/issues/57

### cavil-loop（GigleAI/cavil-loop）

- 开源个人项目。本机 60 秒轮询 GitHub label（如 `pending/agent`），命中后在本机 git worktree 里起 agent（Claude Code、OpenCode、Codex 均可），设计→写码→测试→开 PR，全程代码不出本机。来源：https://github.com/GigleAI/cavil-loop
- 形态和用户设想几乎逐字重合：GitHub label 当接口、worktree 隔离、本机跑、并行每 issue 一个 worker。来源：同上
- 明显缺点：个人项目、成熟度低、未见过大规模使用反馈；等于"用户想自建的最简版别人已经写了个雏形"，可当参考实现。来源：同上

### GitHub Copilot coding agent

- 商用，已 GA（2025-09-25），所有付费 Copilot 订阅可用。assign issue 给 Copilot → 在 GitHub Actions 云端环境干活 → 自动开 draft PR，@copilot 评论迭代；消耗 Actions 分钟数 + premium requests。来源：https://github.blog/changelog/2025-09-25-copilot-coding-agent-is-now-generally-available/、https://github.blog/ai-and-ml/github-copilot/assigning-and-completing-issues-with-coding-agent-in-github-copilot/
- 明显缺点：纯云端，代码在 GitHub 环境跑；模型不可自选（用 Copilot 自带模型链）；与 PI 体系无关。来源：同上

### Cursor Cloud Agents（原 Background Agents）

- 商用，Cursor Pro（$20/月）起。云端 VM 跑任务、开 PR；模型在 Cursor 精选集里选；计费按模型 API 价另算，需设 spend limit。来源：https://cursor.com/docs/cloud-agent、https://forum.cursor.com/t/what-is-the-pricing-structure-for-using-cloud-agents/156843
- 明显缺点：云端、绑 Cursor 生态（触发入口在 Cursor IDE/Web/Slack）；不自托管。来源：https://cursor.com/docs/cloud-agent

### Devin（Cognition）

- 商用，Devin 2.0 起价 $20/月，按量计费（约 $2.25/ACU）。Slack 指派任务 / 自家云端 IDE，产 PR。来源：https://venturebeat.com/technology/devin-2-0-is-here-cognition-slashes-price-of-ai-software-engineer-to-20-per-month-from-500、https://cognition.ai/blog/new-self-serve-plans-for-devin
- 明显缺点：云端、封闭、定价模型复杂（ACU + 超额计费）；公开口碑两极（本次调研未逐条核实差评，标注为一般印象）。来源：同上

### OpenAI Codex cloud

- 商用，含在 ChatGPT 各档订阅里（Plus 起额度够用）。从 GitHub PR、Linear、Slack 发起云端任务，在隔离云容器里跑，完成后开 PR。来源：https://learn.chatgpt.com/docs/cloud、https://chatgpt.com/codex/pricing/
- 明显缺点：云端、模型锁 GPT 系。来源：同上

### Google Jules

- 商用，免费档 15 任务/天，Pro $19.99/月，Ultra $124.99/月。GitHub issue 打 `jules` 标签触发，克隆到 Google 云 VM，自动产 diff 并开 PR；模型用 Gemini 系。来源：https://jules.google/、https://sdd.sh/2026/04/agentic-coding-agent-comparison-2026/
- 明显缺点：云端、Google 绑定、模型不可自选。来源：同上

### Codegen

- 商用平台（"OS for Code Agents"）。GitHub 评论 @codegen-agent、Slack、Linear、Jira 等触发；每 context 一个专属 agent 在云端沙箱跑，自动开 PR、自动修 CI、自动 PR review。来源：https://docs.codegen.com/capabilities/triggering-codegen、https://docs.codegen.com/introduction/overview
- 明显缺点：云端 SaaS、面向团队场景，个人用偏重。来源：同上

### Factory Droids（factory.ai）

- 商用，Pro $20/月起，Plus $100/月才有 Factory 托管云电脑；企业档才有 on-prem/self-hosted。Linear/Jira/GitHub Issues 指派 → 产带代码、测试、文档的 PR；官方宣称模型无关（Claude、Gemini、GPT、自定义端点）。来源：https://factory.ai/pricing、https://factory.ai/industries/saas
- 明显缺点：本机/自托管只在企业档（价格未公开）；个人档全是云端。来源：https://factory.ai/pricing

### Sweep（sweep.dev）

- 商用（YC W23），GitHub 原生：assign issue 给 Sweep → 多文件编辑 → 开 PR；计费在 LLM 成本上加 5% 服务费。来源：https://delv.tools/agents/sweep、https://docs.sweep.dev/pricing
- 明显缺点：云端、GitHub-only、个人项目信息更新不活跃（本次未核实其 2026 维护状态，标注未确认）。来源：同上

### Tembo（tembo.io）

- 商用自动化平台。事件驱动（GitHub/GitLab/Sentry/Linear/Slack/webhook），agent 在云端沙箱跑，产 merge-ready PR；每个 agent 可选 harness + 模型（Claude Code+Sonnet、Codex+GPT、Cursor 等）。来源：https://docs.tembo.io/features/agents
- 明显缺点：云端、偏企业自动化场景。来源：同上

## 对照用户需求：谁最接近

用户四条硬需求：本机跑、异步、GitHub 中心、轻量。

| 需求 | 命中产品 |
|---|---|
| 四条全中 | 无 |
| 中三条半 | Multica（本机✓ 异步✓ 轻量✗；GitHub 中心半中——它有自家工作台，GitHub 只是接入口之一） |
| 中三条 | cavil-loop、kraken（本机✓ 异步✓ GitHub 中心✓；轻量✓ 但代价是个人项目成熟度） |
| 中三条 | OpenHands resolver 单用（本机✓ 异步✓ GitHub 中心✓；轻量勉强——比 Multica 轻，但仍是一套框架） |
| 中两条半 | Claude Code Action + self-hosted runner（本机✓ 异步✓ GitHub 中心✓；但模型锁 Anthropic，且要养 runner） |
| 只中"异步" | Copilot coding agent、Jules、Codex cloud、Cursor、Devin、Codegen、Factory、Sweep、Tembo（全部云端 SaaS） |
| 形态不重合 | raft.build（本机✓ 但以聊天频道为中心，不是 issue→PR） |

三条判断：

1. **"完全现成且全命中"不存在。** 云端 SaaS 阵营（Copilot、Jules、Devin 等）功能最成熟但全部不满足"代码不离本机"；本机阵营里功能最全的 Multica 恰恰是用户嫌重的那个。
2. **cavil-loop 和 kraken 证明"最简自建"路线可行且已有人走过。** 它们的形态（label 轮询 + worktree + 本机 agent CLI + draft PR）与 D4 决策里"cron + PI + sandbox 最简自建"几乎同构，可直接当参考实现读代码，不必从零想。来源：https://github.com/GigleAI/cavil-loop、https://github.com/rafael-adcp/kraken
3. **推荐候选顺序：先精读 cavil-loop/kraken 当自建蓝本；若想直接拿现成，OpenHands resolver 单用是"重"和"自建"之间的折中；Multica 维持"备选不动"（与 D6/D9 已决一致）。**
