# X 上 wayfinder 真实讨论（浏览器直查）

调研时间：2026-08-20。范围只取 2026-06-01 之后的 X 帖子；游戏、职业规划、加密项目等同名内容已排除。日期按浏览器中 X 中文界面显示。先按要求搜 `from:mattpocock wayfinder`，因账号写错而无结果；改用 Matt 的实际账号 `@mattpocockuk` 后，在 Live 和 Top 都取得结果。另搜了 `wayfinder skill claude`、`"wayfinder" "decision map"`、`wayfinder map`、`wayfinder (shipped OR built OR finished)`。

## 一、Matt 原帖

- 2026-07-02｜https://x.com/mattpocockuk/status/2072599827540578664 —— 首次公开定名 `/wayfinder`，说明它识别“决策前沿”和未知区域，并按问题选择 prototype、research 或 grilling；X 显示约 7 万次查看。
- 2026-07-03｜https://x.com/mattpocockuk/status/2072716979195326905 —— Matt 称自己连续四天用它规划整门课程，进行了近 100 次 grilling、prototype、research 会话，并汇总到一张持续增减的地图。
- 2026-07-06｜https://x.com/mattpocockuk/status/2073811512938868814 —— 展示一份由多日 wayfinder 会话生成的 PRD，强调每个结论都可追溯到做决定的会话。
- 2026-07-08｜https://x.com/mattpocockuk/status/2074860312423997800 —— v1.1 发布帖正式推出 `/wayfinder`，并给出 wayfinder、to-spec、to-tickets、implement、code-review 的完整链路；约 25.3 万次查看。
- 2026-07-11｜https://x.com/mattpocockuk/status/2075856898142740821 —— 澄清大任务应走 `/wayfinder → /to-spec → /to-tickets → /implement`；承认有人直接用 map 做到交付，但他在编码任务中更偏好先转成 spec；约 8.2 万次查看。
- 2026-07-12｜https://x.com/mattpocockuk/status/2076297916336013516 —— 看到第三方可视化后，提出做 wayfinder CLI，以 GitHub、GitLab 或本地 map 编号作为入口。
- 2026-07-13｜https://x.com/mattpocockuk/status/2076679479435337784 —— 发布 wayfinder 直播演示入口，邀请用户现场提问；Live 搜索补出了上次遗漏的这条。
- 2026-07-14｜https://x.com/mattpocockuk/status/2076996976525054246 —— 公开回应真实缺陷：许多用户认为 wayfinder 的“共享语言”工作不如 grill-with-docs，Matt 提议在画 map 前先走 domain-modeling。
- 2026-07-15｜https://x.com/mattpocockuk/status/2077081524474630227 —— 发起 3,212 人投票询问“构建新东西时会用哪个 skill”；wayfinder 得票 25.9%，低于 grill-with-docs 的 32.9%，但略高于 grill-me 的 23%。
- 2026-07-16｜https://x.com/mattpocockuk/status/2077743625639714850 —— 推荐 Will Ness 的用法：把 wayfinder 当作自定义 skill 的编排器，并给 UI 原型分阶段开票；约 18.1 万次查看。
- 2026-07-17｜https://x.com/mattpocockuk/status/2078031590337245518 —— 分享自己的实际入口：先 grill-with-docs，发现工作超出单会话后，再让 wayfinder 建 map；约 7.2 万次查看。
- 2026-07-29｜https://x.com/mattpocockuk/status/2082456028717654525 —— 展示另一份从完整 wayfinder map 转出的“大规格”，用于说明 map 应在实现前收敛为 spec；约 7.2 万次查看。
- 2026-07-30｜https://x.com/mattpocockuk/status/2082774006189449355 —— 主推讲解帖：wayfinder 会找出当前可决定的前沿、边走边揭示路线、组织研究和原型，并把 map 存进 GitHub/Linear；约 12.2 万次查看。
- 2026-07-31｜https://x.com/mattpocockuk/status/2082918384606314949 —— Matt 在回复中称已有团队成功共用 map 分派工单，也有人把一张 wayfinder map 当作另一张 map 的票；这是作者转述，不是第三方复盘。

## 二、第三方讨论

- 2026-07-03｜https://x.com/albertgao/status/2072874339523756394 —— Albert Gao 用同一 wayfinder 提示比较 Devin/Fable 5 与 Codex/GPT-5.5，认为 Fable 更早识别关键问题、深度更合适，但提问偏封闭；这是有正有负的实测评价。
- 2026-07-11｜https://x.com/ahmetbilicanxyz/status/2075915507568451928 —— Ahmet Bilican 称 wayfinder + Claude 规划、GPT-5.6 Sol 实现是自己近期找到的最佳流程，能并行管理原本难以独立处理的大块工作，团队反馈也很好。
- 2026-07-11｜https://x.com/hampsonw/status/2075940748877316255 —— 一位连续使用两天的用户指出：多会话 map 的知识不一定留在当前上下文，`to-spec` 的完成信号和上下文交接存在缺口。
- 2026-07-12｜https://x.com/johnrwgoh/status/2076009954604151147 —— johnrwgoh 明确说“loving” wayfinder，并做了 planetary view 来可视化过程；约 11.9 万次查看。
- 2026-07-16｜https://x.com/WillNessAI/status/2077740031561642398 —— Will Ness 把 frontend grilling/prototype 变体接入 wayfinder，让 map 自动创建引用该 skill 的票，评价这是“非常强的规划模式”；约 25 万次查看。
- 2026-07-20｜https://x.com/CopperFernCo/status/2078941700454506758 —— 用户称自己在 23 个仓库运行 wayfinder，规划从“猜测”变成逐个解决决策票；内容具体，但互动很低。
- 2026-07-29｜https://x.com/CyberDelanooch/status/2082153238527435154 —— 用户花约一周为大项目生成 50 多张 issue，当时已实现一半；说明它能支撑长项目，但 map 本身也可能很重。
- 2026-08-10｜https://x.com/Harshvikram14/status/2086773301842792604 —— 用户说原以为一天的工作被 wayfinder map 拉长到四天；仍喜欢它，但明确认为需要调整，属于“过度展开”反馈。
- 2026-08-11｜https://x.com/marcadx/status/2087086001386569982 —— 用户认为 wayfinder 开新票前没有充分读取旧票，容易拆得过细；他的补救方式是要求 agent 重读 map 并合并过度细化的票。
- 2026-08-12｜https://x.com/Bryce58831457/status/2087373307704398205 —— 用户称模糊项目中反复运行会膨胀到约 60 张票，还丢掉重要备注和一半想法，体验甚至不如原生 Claude；这是最直接的负面实测。
- 2026-08-12｜https://x.com/lucidedev/status/2087558457314734255 —— 用户质疑“过期地图”问题：早期决定改变后，wayfinder 是否会标出失效的下游票，还是仍要人工发现。
- 2026-08-16｜https://x.com/rizvancodes/status/2088664909487698082 —— 用户称 wayfinder 是 game changer，因为它补上了“需求收集之前先把模糊想法画成 map 并澄清”的步骤。
- 2026-08-17｜https://x.com/leeked/status/2089127652368003126 —— 用户把 wayfinder map 交给 Grok、Claude、Codex 等组成的 panel 审查，找出 10 多个重大问题；说明 map 可被复用审计，也说明单靠 map 仍可能漏掉重要问题。
- 2026-08-17｜https://x.com/alexdupler/status/2089021398320873781 —— 用户指出在 Copilot CLI 中按 map 为每张开放 issue 建 child session 时，会碰到 user-invoked/model-invoked 的触发兼容问题。
- 2026-08-19｜https://x.com/alexkelerman/status/2090058524399112672 —— Alex Keler 公开完整流程：3 小时做 wayfinder/grilling，随后转 spec、tickets，并用多 agent/worktree 隔夜实现；这是目前最完整的第三方成功复盘。

## 三、完成项目经验

- 2026-08-19｜https://x.com/alexkelerman/status/2090058524399112672 —— **确认完成**：用 wayfinder map 规划并在 24 小时内重建一个数据收集应用，含前端、Supabase 后端、PostHog 分析、Resend 邮件和 VPS 部署；作者认为最关键的是前 3 小时的 map 与 grilling。
- 2026-07-29｜https://x.com/CyberDelanooch/status/2082153238527435154 —— **进行中，不算完成**：一周生成 50 多张 issue，发帖时实现约一半，能证明持续执行但不能证明最终交付。
- 2026-07-20｜https://x.com/CopperFernCo/status/2078941700454506758 —— **长期使用，不算完成复盘**：声称 23 个仓库都在运行 wayfinder，但没有列出具体完成项目或最终结果。
- 2026-07-03｜https://x.com/mattpocockuk/status/2072716979195326905 —— **作者自己的真实使用**：四天、近 100 次会话规划整门课程；证明 map 能承载超大任务，但原帖没有确认课程在发帖时已完成。
- 2026-07-06｜https://x.com/mattpocockuk/status/2073811512938868814 —— **作者完成规划产物，不等于完成项目**：多日 map 已转成可追溯 PRD，但帖子只展示了规划终点。

热度结论：**有**——Matt 本人持续高频推广，核心帖多在 3.7 万至 25.3 万次查看；第三方已不止上次找到的 2 条，存在多项真实使用、改造和负面反馈，但讨论仍集中在 AI coding 小圈层。X 访问方式：**直连**（已登录 Chrome 的 X 标签页；独立无登录窗口的搜索撞登录墙，relay 尝试超时）。
