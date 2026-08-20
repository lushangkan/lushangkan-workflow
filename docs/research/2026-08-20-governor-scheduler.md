# 调研报告：governor 规则 + 软提醒/计时 hook

调研时间：2026-08-20。全部结论来自源码或官方文档，行号基于克隆时的 HEAD。

## 结论先行

1. `github.com/alirezarezvani/claude-governor` **不存在**（404，无重定向）。本项目自己的 `docs/workflow-v2.md` 记录 governor 的真实来源是 **cwinvestments/memstack 的 governor skill**（MIT）——「投入分档、决策点给默认值、尊重用户否决」三条规则原文都在那里。
2. `warpdotdev/commons-skills`（真名 `common-skills`，无 s）25 个 skill **没有**软提醒/计时类。真正的「软提醒/计时」skill 是 **warpdotdev/oz-skills 的 scheduler**（MIT）：不是 hook，是给 agent 的指令型 skill，落地靠 OS 原生调度器（Linux 用 cron/systemd timer + notify-send）。
3. scheduler 不是 Claude Code 专用，**可直接移植到 omp**；且 omp 自己就有完整 hooks 子系统（ExtensionAPI），连 claude-governor 式 5 小时协议都有等价移植路径。

---

## 第 1 节 claude-governor 是什么

### 1.1 目标 URL 不存在

- `api.github.com/repos/alirezarezvani/claude-governor` → HTTP 404；`github.com/alirezarezvani/claude-governor` → 404 且无 301 重定向（GitHub 对改名仓库会重定向，所以该仓库要么从未存在，要么已被删除）。
- alirezarezvani 名下 33 个公开仓库（claude-skills、claude-code-mastery 等），无任何 governor 命名仓库；claude-skills 内搜 "governor" 只有无关内容（git 治理命令、EU AI Act 合规）。
- Wayback Machine 当天离线，历史存在性**未确认**。

### 1.2 项目自有记录指向的真实来源：cwinvestments/memstack 的 governor skill

`docs/workflow-v2.md` 第 49 行（2026-08-20 当天记录）：

> 反完美主义 | **引用** governor（投入分档、决策点给默认值、尊重用户否决）+ 已耗时间先做软提醒（Linux 计时 hook 无现成；…） | cwinvestments/memstack，MIT

即主 agent 要的「governor 规则」原文在 `cwinvestments/memstack/skills/governor/SKILL.md`（skill version 1.0.0，来源 MemStack v3.2，2026-02）。三条规则的具体写法：

**① 投入分档**（Tier 表，L49–53）：

| Tier | 描述 | Effort Allocation |
|---|---|---|
| Prototype | 探索想法，可能丢弃 | Minimal——只写能跑的代码 |
| MVP | 想法已验证，给第一批用户 | Moderate——基础质量门 |
| Production | 服务真实用户 | Full——完整质量栈 |

每档配 Allowed / NOT Allowed 清单与一句判据（L72/86/100）：
- Prototype：「If it works in a demo, ship it.」
- MVP：「If the first 10 users can use it reliably, ship it.」
- Production：「If it breaks at 3 AM, someone gets paged.」

**② 决策点默认值**（L55）：

> If tier is unclear, default to **Prototype** and escalate only when evidence suggests otherwise.

（档位不明 → 默认 Prototype；只有出现证据才升档。）

**③ 尊重用户否决**（两处）：

- Context Guard（L28）：「User explicitly overrides ("I know, do it anyway") — DORMANT — user has authority」
- Protocol（L124）：「Always defer to the user if they override. Governor advises, doesn't block.」

**附带内容**（反完美主义的实质）：
- Anti-Rationalization 对照表（L30–41）：六条典型的范围蔓延借口逐条反驳（"Adding tests is always good practice" → 原型的测试是浪费；"Let me add CI/CD while I'm at it" → 原型 CI/CD 是镀金；"This should be configurable" → 先硬编码，有人要再配置化……）。
- 违规标记模板（L113–124）：「You're proposing {X}, but this is a {Tier} project. … Want to proceed anyway, or skip it for now?」——只提醒、不拦截的完整句式。

license：**MIT**（仓库 LICENSE：Copyright 2026 CW Affiliate Investments LLC）。

### 1.3 同名「claude-governor」仓库（若主 agent 确实指 GitHub 上的同名项目）

| 仓库 | license | 定位 | 与本任务关系 |
|---|---|---|---|
| SamSamala/claude-governor | MIT | Claude Code 用量 governor：GOVERNOR.md 规则文件 + settings.json 深合并 + 3 个 hook + statusline | 规则写法可借鉴（同意门/否决/模型分档），机制是 Claude Code 专属 |
| prathameshkadam130404/claude-governor | MIT | 预算感知 agent 循环：statusline + 8 个 hook 注入 [governor] 预算行，四档 CRUISE/ECONOMY/WIND-DOWN/CHECKPOINT | 与"软提醒"概念接近，但仍是 Claude Code 专属 |
| jedarden/claude-governor | Apache-2.0 | Rust 写的 CI 侧 Sonnet worker 容量 governor | 无关 |

SamSamala 版 GOVERNOR.md 里与 output-style 同构的交互写法（可借鉴，不用抄机制）：
- 同意门 + 单决策点（L43–55）：「One consent gate, up front — then run everything automatically. … Wait for a yes. … it's not a second decision point for the user.」
- 否决即停（L46–49）：「If they say no, or don't clearly agree, stop here. Do not train anything… This is required, not optional.」
- 模型分档路由（L147–151）：「routes subagents by task — haiku for simple work, sonnet for feature work, opus for hard reasoning.」
- 5 小时协议（L177–197）：statusline 每秒读 rate_limits → 70% 静默提醒、85% 触发 PostToolBatch 写 HANDOFF.md → 下个会话 SessionStart 载入。

---

## 第 2 节 与 output-style 的重叠度（只列增量）

output-style 现有（已核对 SKILL.md 原文）：
- **A2**（L74）：待决策事项 2 个以内，每个带推荐值；不以"还有一个选择"追加开放选项。
- **让位条款**（L22–29）：用户要求详细解释时可写长；破坏性操作先确认；debug 三轮没进展就停下；规则与任务冲突时任务赢。
- 项目跨领域约定：只提醒不 override，最终决定权在用户。

memstack governor 里 output-style **没有**的增量：

| # | 增量 | 说明 |
|---|---|---|
| 1 | 投入分档（三档 + 每档 Allowed/NOT Allowed 清单） | output-style 只管"怎么说"，完全不管"做什么复杂度"。纯增量 |
| 2 | 决策点默认值 + 显式升级门槛 | A2 要求"带推荐值"，但没有"不明时默认保守档、有证据才升档" |
| 3 | 用户 override 的显式状态机写法 | 项目有"最终决定权在用户"约定，但没有"用户说 do it anyway 后 governor 转 DORMANT 不再提醒"的写法 |
| 4 | 反合理化对照表（6 条借口逐条反驳） | output-style 无反完美主义内容。与项目"预防不拦截"约定同构 |
| 5 | 违规标记模板（"想继续还是跳过？"） | 只提醒不拦截的现成句式 |

**落地时的坑**：memstack 的 Prototype 档写 "NOT Allowed: unit tests"，与本项目 TDD 纪律直接冲突——落地到根 AGENTS.md 时必须改写（如"原型期不追求完整测试栈，保留冒烟验证"）。

---

## 第 3 节 warpdotdev 的软提醒/计时 skill

### 3.1 common-skills 里没有

`warpdotdev/common-skills`（注意：无 s；warpdotdev 组织下另有 `oz-skills`）共 25 个 SKILL.md：brandalf、check-impl-against-spec、complain、council、create-pr、cross-critique、diagnose-ci-failures、fix-errors、implement-specs、pr-walkthrough、readout、reproduce-bug-report、research、resolve-merge-conflicts、respond-to-pr-comments-in-blocklist、review-pr、saga、scan-new-specs、spec-driven-implementation、suggestion-box、update-skill、validate-changes-match-specs、write-feature-docs、write-product-spec、write-tech-spec。**无 timer/reminder/notify 类**（全文搜索确认；"notify/remind" 命中只在 complain/suggestion-box 的 Slack webhook 文本里）。license：MIT（Copyright 2026 Denver Technologies, Inc.）。

### 3.2 真正的软提醒/计时 skill：warpdotdev/oz-skills 的 scheduler

路径 `.agents/skills/scheduler/SKILL.md`。frontmatter（L1–4）：

```
name: scheduler
description: Schedule on-device reminders and local actions only. Use this skill to set personal
reminders or run lightweight, local tasks at a specific time or interval (e.g., notifications, local
scripts), on the user's computer or with platforms like Slack. Do NOT use for scheduling cloud
agents, background agentic jobs, or Oz-managed workflows.
license: MIT
```

**机制：不是 hook、不是脚本、不是 systemd timer 本体**，而是一个指令型 skill（Markdown 指令），由 agent 按用户 OS 选原生调度器落地：

| OS | 调度 | 通知 |
|---|---|---|
| Linux | cron 或 systemd timer（user 级优先） | notify-send（无桌面通知则退回终端输出） |
| macOS | launchd（免管理员权限） | osascript → Notification Center |
| Windows | 任务计划程序（user 级优先） | Toast 通知 |

行为要点（原文）：不假设默认投递方式，不明时先问（L17–18、L42）；reminder 与 task 二选一，不明默认 reminder（L42）；生成稳定 kebab-case 名、重名加数字后缀；落地后必须给确认摘要（名称/动作/时间/投递方式，L217+）；支持列出/暂停/恢复/删除；能力不可用时不静默降级，先说明再问（L123 附近）。license：**MIT**（仓库 LICENSE Copyright 2026 Warp；SKILL.md frontmatter 同标）。

---

## 第 4 节 omp 能不能原样用

### 4.1 scheduler 不是 Claude Code 专用 → 直接移植，不需要"替代"

- 机制 = agent 指令 + OS 原生调度命令，与 harness 无关；omp 的 agent 有 bash 工具，能执行 systemd/cron/notify-send。
- omp 的 skill 发现：`<skills-root>/<skill-name>/SKILL.md` 非递归布局；`.agent[s]/skills/` 是 omp 原生位置（agents provider）；frontmatter 支持 name/description 等，多余 key（license 等）保留为元数据 → scheduler 的 SKILL.md 结构完全兼容，放进 `skills/scheduler/SKILL.md` 即可被发现。
- 移植建议：删掉 Warp 专属的 Slack 段落（可选）；保留"不明时先问"行为；description 触发词改为中文适配（"提醒我、X 分钟后、每天"）。

### 4.2 顺带澄清：omp 有完整 hooks 子系统（hooks 机制并非不明）

- 两种 API：`HookAPI`（legacy）与 `ExtensionAPI`（新，推荐，支持 hook 事件 + 扩展专属事件）。TS 工厂模块 `export default function hook(pi) { pi.on(event, handler) }`，发现路径如 `.omp/hooks/pre/*.ts`。
- 关键事件：session_start、turn_start/turn_end、agent_start/end、tool_call（可 block/改参数）、tool_result（可改输出）、context（每次 LLM 调用前改消息）、session.compacting、todo_reminder。
- UI：`ctx.ui.notify(message, type?)`（应用内通知）、`ctx.ui.setStatus(key, text)`（状态栏，等价 Claude Code statusline）、confirm/select/input。
- 含义：若未来要把 SamSamala 式"5 小时协议"（70% 静默提醒、85% handoff、SessionStart 载入）搬进 omp，有等价事件：statusline → setStatus；PostToolBatch → turn_end/tool_result；SessionStart → session_start；handoff 注入 → context/session.compacting。当前需求（软提醒）不需要走这条路。

### 4.3 Linux 最小落地（纯计时软提醒，不写 skill）

systemd --user timer + notify-send（示例）：

```ini
# ~/.config/systemd/user/remind-stretch.service
[Unit]
Description=Soft reminder

[Service]
Type=oneshot
ExecStart=/usr/bin/notify-send "休息一下" "站起来拉伸 2 分钟"

# ~/.config/systemd/user/remind-stretch.timer
[Unit]
Description=Soft reminder timer

[Timer]
OnActiveSec=30min
OnUnitActiveSec=30min
Unit=remind-stretch.service

[Install]
WantedBy=timers.target
```

```bash
systemctl --user daemon-reload
systemctl --user start remind-stretch.timer   # 启动
systemctl --user enable --now remind-stretch.timer  # 开机自启
systemctl --user list-timers                 # 查看
```

一次性提醒用 cron 更简（或用 `systemd-run --user --on-calendar=...`）。注意：notify-send 依赖桌面通知守护进程（KDE/GNOME/Hyprland+dunst/mako 自带或补装）；无桌面环境时退回终端输出（scheduler skill 原话）。

---

## 第 5 节 最小落地建议

1. **governor 规则进根 AGENTS.md**：取第 2 节 5 个增量，压缩成 2–3 条规则 + 一张档位表。必改点：Prototype 档的"不写测试"要改写成与 TDD 纪律兼容的表述；档位内容按本项目语境改写（不是通用软件工程清单）。
2. **软提醒落地**：以 oz-skills scheduler 为蓝本写一个小 skill（或最小版：根 AGENTS.md 一行规则"用户说'提醒我 X'时用 systemd --user timer + notify-send 建一次性提醒"）。不用 Claude Code hook，不走 omp hooks，机制与 harness 解耦。
3. **不采用**：SamSamala/claude-governor 的 settings.json 深合并 + 3 hook 方案（Claude Code 专属、会改用户配置）；只借鉴其"同意门 + 单决策点 + 否决即停"的交互写法，与 output-style A2/让位条款互补。
4. **待主 agent 确认**：若 1.3 中某个同名 claude-governor 才是真正目标，需主 agent 指认；本报告按项目自有记录（workflow-v2.md）以 memstack governor 为准。

## 未确认项

- alirezarezvani/claude-governor 的历史存在性（Wayback 离线，当前 404 无重定向）。
- omp `todo_reminder` 事件的触发来源细节（事件存在但未深挖；不影响上述结论）。
