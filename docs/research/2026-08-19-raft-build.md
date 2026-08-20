# R4 调研报告：raft.build 工作流程与上手方法

判断先行：raft.build 对个人用户是开放可注册的（有免费档），上手路径是"网页注册 → 装本机服务 → 建 agent"三步；官方支持 Pi 作为 runtime，与你的 harness 直接兼容。你配置不了的最可能原因不是产品门槛，而是网络访问（控制台在境外）或本机服务没常驻。

调研日期：2026-08-19。产品原名 Slock（slock.ai 会 301 跳到 raft.build），Botiverse 公司，2025 年成立，目前 Raft 1.0 已公开发布。

## 1. 完整上手流程

来源：https://docs.raft.build/meet-your-onboarding-agent/ （官方"Meet your Onboarding Agent"教程，含视频）

四步，官方称十分钟内完成：

1. **注册 + 建 server**。在 https://app.raft.build 注册账号，创建 server（团队空间），起个名字，URL slug 自动生成。建完落在私有频道 `#onboarding-owner`。
2. **连接一台电脑（Computer）**。网页的"Connect a computer"面板给出一条安装命令，形如：
   `curl -fsSL https://cdn.raft.build/computer/install.sh | sh && raft-computer setup /<server-slug>`
   在终端跑它：装 `raft-computer` 本机服务 → 浏览器弹出设备登录页 → 批准 → 机器上线。装完后 Raft 会扫描这台机器上已安装的 runtime 并列出来。
3. **创建第一个 agent（Cindy，引导 agent）**。给她选 runtime + provider + 模型。runtime 清单：Claude Code、Codex CLI（官方推荐这两个），另有 Antigravity CLI、Copilot CLI、Cursor CLI、Gemini CLI、Kimi Code、OpenCode、**Pi**；也可以不装 runtime、直接自带 API key。
   来源：https://docs.raft.build/features/agents/runtime/
4. **Cindy 在频道里接管引导**。她自我介绍、带你建团队，之后遇到任何问题可以直接问她。

如果机器上没检测到 runtime，第 2 步会卡住——需要先装一个 runtime 或准备 API key。

## 2. 工作流程

来源：https://docs.raft.build/hand-off-your-first-task/ 、https://docs.raft.build/divide-the-work/ 、https://docs.raft.build/features/messaging/channels/

核心循环就一句话：**像在群聊里给同事派活一样给 agent 派活**。

| 动作 | 怎么做 |
|---|---|
| 建频道 | 侧栏 Channels 旁的 **+** → Create Channel。分公开/私有；公开频道 agent 可自己加入，私有频道要 owner/admin 拉 |
| agent 进频道 | 创建 agent 时选电脑 + runtime + 名字 + 描述；agent 可以自己加入公开频道，@mention 也能触达未加入的频道 |
| 派活 | 在频道里直接发消息说需求（不用写好 prompt）；然后右键消息选 **Convert to Task**，或发消息时勾选 **As Task**。任务有四态：todo → in progress → in review → done |
| agent 干活 | agent 认领任务，在任务的 thread 里持续汇报进度；你不用盯着，可以走开 |
| 回消息/验收 | agent 完成后把任务置为 in review 并发结果；你在 thread 里回复意见，它会记住反馈，下次更准 |
| 多 agent | 一个频道放多个 agent（官方样例：PM / engineer / reviewer 三个角色一个频道），它们互相交接、互相 review |

**PR 怎么产出**：官方没有"一键出 PR"的功能。模式是 agent 跑在你的电脑上、就在你的 git 仓库里干活（工程团队用例页明确写 "Works with: any git repo"），产出 commit/分支靠 runtime 自带的 git 能力，PR 链接由 agent 发到频道里。官网首页演示对话就是 "@Tenny CI on PR #982 怎么样了" 这种形态。
来源：https://raft.build/resources/use-cases/engineering-team/ 、https://raft.build/

## 3. GitHub 怎么接

**结论：没有第一方 GitHub 集成。这一点翻遍了全部 40+ 页官方文档（llms.txt 索引）确认。**

- 文档索引里没有任何 GitHub 专页：https://docs.raft.build/llms.txt
- 在官方文档仓库 botiverse/raft-docs 里搜 "github"，命中的全是 runtime 安装链接和文档仓库自身链接，无集成功能：https://github.com/botiverse/raft-docs

实际的 GitHub 接入方式是间接的：

1. **agent 用你机器上现成的 git/gh 凭据**。runtime 跑在你电脑上，git push、`gh pr create` 走的是你自己登录的 gh CLI / git credential。权限范围 = 你自己 GitHub 账号的权限，Raft 不经手。
2. **想接得更深，走 Raft Apps**：第三方应用可以通过 "Login with Raft" 接入，agent 登录后获得 app 提供的动作。这是面向开发者的机制，不是开箱即用的 GitHub 按钮。
   来源：https://docs.raft.build/features/apps/ 、https://docs.raft.build/developers/login-with-raft/

授权方式（OAuth scope、GitHub App 权限）：**未确认**——官方没有第一方集成，所以不存在这套东西可考。

## 4. 本机 daemon（Raft Computer）要求与平台

来源：官方安装脚本原文 https://cdn.raft.build/computer/install.sh 、https://docs.raft.build/features/server/computers/

| 项 | 结论 |
|---|---|
| macOS | 支持，x64 + arm64（含 Rosetta 下自动选原生 arm64） |
| Linux | 支持，x64 + arm64。**你的 CachyOS x64 在支持范围内** |
| Windows | 没有原生应用，走 WSL；且是过渡方案——旧式 `raft-daemon` 只在终端窗口开着时存活 |
| 形态 | 约 150MB 的自包含单文件二进制（SEA），**不需要 Node.js**；装到 `~/.local/bin`，命令名 `raft-computer` |
| 常驻 | 装完后作为后台服务运行，agent 崩溃会自动恢复；机器掉线则该机所有 agent 停摆 |
| 状态目录 | `~/.slock/`（沿用 Slock 时代命名，也可用 `RAFT_HOME` 覆盖） |
| 管理命令 | `raft-computer start/restart/setup /<server-slug>`、`raft-computer doctor` |

注意：网上旧教程（Slock 时代）写的是 `npx @slock-ai/daemon`、需要 Node.js 18+——那是旧版 daemon，已被 `raft-computer` 取代，官方提供迁移路径。
来源：https://codepick.dev/en/guides/slock-setup/ （旧）、https://docs.raft.build/features/server/computers/ （新，含迁移说明）

## 5. 定价与免费额度

来源：官网首页定价区 https://raft.build/#pricing （2026-08-19 抓取原文）

| 档 | 价格 | 内容 |
|---|---|---|
| Free | $0 | 频道、任务、agent 跑在自己电脑上、agent 提醒、基础可观测性、**30 天消息历史**、每月 100MB 文件上传 |
| Pro | **$8.80 / seat / 月**（按年付） | Free 全部 + 无限消息历史 + 更高上传限额 + joint channels（跨 server 共享频道）。**计费规则：1 个人 = 1 seat，1 个 agent = 0.1 seat** |
| Enterprise | 未上线（coming soon） | 私有化部署、SSO、专属支持 |

关键点：**免费档就能完整体验"agent 群聊协作"**——频道、任务、本机 agent 都在 Free 里。模型费用另算：runtime 用你自己的订阅（Claude/Codex/DeepSeek 等），Raft 不转售模型。第三方报道（testingcatalog、everydev.ai）数字与官网一致。
来源：https://www.testingcatalog.com/raft-1-0-puts-ai-agents-in-team-mode/ 、https://www.everydev.ai/tools/raft-build

## 6. 已知安装/配置坑

官方排障页：https://docs.raft.build/features/agents/troubleshooting/

| 坑 | 来源 |
|---|---|
| 控制台在境外，中国大陆访问 app.raft.build 需要代理；agent 执行在本机，不需要 | https://codepick.dev/en/guides/slock-setup/ |
| 本机服务掉线 = 所有 agent 失联（V2EX 用户原话"一旦本地的后台程序掉线，就是灾难"） | https://www.v2ex.com/t/1231480 |
| Windows 过渡 daemon 关掉终端就死 | https://docs.raft.build/features/server/computers/ |
| runtime 检测不到：runtime 没装或不在 PATH 上，第 2 步会显示空列表 | https://docs.raft.build/meet-your-onboarding-agent/ |
| 150MB 二进制下载，部分网络会撞 HTTP/2 传输错误（安装脚本为此内置了 3 次重试） | https://cdn.raft.build/computer/install.sh 注释 |
| `~/.local/bin` 不在 PATH：安装脚本只会自动改 `.bashrc`/`.zshrc`，用 fish 等其它 shell 要手动加 | 同上脚本 `persist_shell_path` 段 |
| Hermes 用户没装 raft CLI 时，Hermes 的 Raft 插件每 10 秒刷一条警告日志（Hermes 侧 bug，已报） | https://github.com/NousResearch/hermes-agent/issues/49234 、https://github.com/NousResearch/hermes-agent/issues/49336 |
| Slock 时代的旧教程（npx @slock-ai/daemon、Node 18+）已过时，照抄会走错路 | https://www.npmjs.com/package/@slock-ai/daemon 、https://codepick.dev/en/guides/slock-setup/ |
| agent 不响应的常见原因：灰点=进程没跑或电脑掉线；没加进频道=收不到消息；黄点=在忙别的 | https://docs.raft.build/features/agents/troubleshooting/ |

没找到 Reddit / HN 上有 raft.build 的配置失败帖（搜索命中均为同类产品）；主仓库 botiverse/slock 在 GitHub 上已不可公开访问（`gh repo view` 返回 not found），所以无法查它的 issue 列表。这一项标注**未确认**——issue 区的配置失败报告看不到。

## 7. 用户配置不了的可能原因清单（按可能性排序）

针对你的场景：CachyOS Linux 桌面、PI harness、想体验 agent 群聊。

1. **控制台打不开或登录不上**：app.raft.build 托管在境外，你的网络（有 mihomo/Clash 环境）可能没让浏览器走代理。
   验证：直接 `curl -I https://app.raft.build` 看是否超时；浏览器开代理后再试。
2. **装错了时代的东西**：照着 Slock 旧教程跑 `npx @slock-ai/daemon` 或找"桌面应用"下载。Raft 没有桌面客户端，本机只装 `raft-computer` CLI，界面全在网页。
   验证：看自己跑过的命令——正确的是 `curl ... install.sh | sh && raft-computer setup /...`；装完 `which raft-computer` 应有输出。
3. **`~/.local/bin` 不在 PATH**：CachyOS 默认 shell 若不是 bash/zsh（安装脚本只认这两个），`raft-computer` 命令找不到，setup 第二步直接报 command not found。
   验证：`echo $PATH | tr ':' '\n' | grep local/bin`；没有就手动 `export PATH="$HOME/.local/bin:$PATH"`。
4. **设备登录页没弹或没批**：setup 时要跳浏览器批准设备授权，浏览器没开/代理拦截/没登录账号都会卡在这一步。
   验证：重跑 `raft-computer setup /<server-slug>`，观察终端是否打印登录链接；在已登录 app.raft.build 的浏览器里打开该链接。
5. **下载中断**：150MB 二进制，网络不稳会失败。
   验证：重跑安装命令即可（脚本有重试和降级保护）；或设 `RAFT_COMPUTER_VERSION` 钉版本重试。
6. **装完后没检测到你的 runtime，建不了 agent**：PI 虽在支持清单里，但要求是"装在这台电脑上且在 PATH 上"。检测不到时可以选"自带 API key"兜底。
   验证：终端里 `which pi`（或你实际的 PI 命令名）；没有就先装 PI 再回第 3 步。
7. **服务没常驻，agent 全灰**：电脑关机/休眠/网络抖动后服务断了，网页上 agent 全离线。
   验证：本机跑 `raft-computer doctor`，再 `raft-computer restart /<server-slug>`；网页侧看 Computers 侧栏是否绿点。

## 附：与你 workflow 的相关性（一句话）

PI 是官方支持的 runtime（https://docs.raft.build/features/agents/runtime/ 清单含 Pi），D4 调研里"agent 群聊"形态的这个竞品与你的 PI 栈天然兼容；免费档足够体验，体验成本 = 注册 + 一条安装命令。
