# X 调研：模型、harness 与多 harness 工作流

检索范围：2026-06-01 至 2026-08-19。检索方式为直接打开 X 的 `search?...&f=live` 与 `&f=top` 页面，再进入相关帖子阅读全文。以下数字为检索时页面显示的查看量。

## 1. 同一模型在不同 CLI 的表现是否不同

- **2026-06-13** — [Max Katz：模型在原生 harness 中通常表现更好](https://x.com/maxktz/status/2065709361507209473)：作者从自定义 Pi 配置转回 Claude/Codex，认为原厂团队迭代更快、模型在原生 harness 中通常更好；同时建议把工作流押在可移植的提示、skills 和脚本上，而不是某个自定义 agent。（9.1 万查看）
- **2026-06-15** — [Dmitriy Kovalenko：同模型、同提示、同代码库的三 harness 对照](https://x.com/neogoose_btw/status/2066360746548842565)：作者用相同模型、effort、provider、代码库和提示，对照 Claude Code、Pi + fff、OpenCode，并明确反驳“模型在原生 harness 更好”；这条实测式反驳获得 61.3 万查看、2,725 喜欢。
- **2026-06-12** — [Kun Chen：相同模型下 Claude Code 落后于 OpenCode 和 Cursor CLI](https://x.com/kunchenguid/status/2065345999682568593)：作者引用 coding-agent benchmark，指出相同模型在 Claude Code 的成绩最差，并据此反对模型订阅锁死在自家工具内。（14.9 万查看）
- **2026-07-26** — [Omed：同模型、同提示，在不同 harness 中“智商完全不同”](https://x.com/OmedVibeCodes/status/2081066821696786472)：作者称严肃编码只信 Claude Code 与 Pi，不信 OpenCode、Codex CLI、Kimi Code 等，因为相同模型和提示在不同 harness 中输出差异明显。（1.6 万查看）
- **2026-08-19** — [derorian41：Kimi K3 在 Claude Code 与 Kimi CLI 的直接体感差异](https://x.com/derorian41/status/2090056454573277509)：作者称 Kimi K3 放进 Claude Code 后表现很差，结论是 Kimi K3 必须在 Kimi CLI 中使用；这是最贴近“模型在自家 CLI 更强”的用户吐槽，但传播量很小。（71 查看）
- **2026-08-19** — [Zain：原生测试套件没有稳定优势](https://x.com/zainhas/status/2089893513962119469)：作者对 GPT↔Codex、Opus/Sonnet↔Claude Code、Kimi K3↔Kimi CLI 的结果表示困惑，称未看到原生 harness 的一致优势；说明社区结论仍有明显分歧。（845 查看）
- **2026-07-08** — [Darren Shepherd：Claude Code 明显优于 Codex CLI](https://x.com/ibuildthecloud/status/2074627050951086151)：作者的主观体感是 Claude Code 产品打磨和投入远胜 Codex CLI，后者像小团队副项目；这比较的是完整产品体验，不是严格控制模型变量的实验。（4,237 查看）

## 2. harness 中立 skills 与一份能力多家复用

- **2026-06-13** — [Max Katz：工作流应押在可移植的提示、skills 和脚本上](https://x.com/maxktz/status/2065709361507209473)：即使作者赞成原生 harness，他给出的落点仍是“不要押注单一自定义 agent”，而应维护能跨 harness 带走的基本资产。（9.1 万查看）
- **2026-07-22** — [SurgePix skill 一次安装支持四家工具](https://x.com/Bangbang_cicily/status/2079938671193796911)：帖子给出 `npx skills add SurgePix/agent-skills`，并明确同一 skill 可在 Claude Code、Codex、Cursor、Gemini CLI 使用。（110 查看）
- **2026-07-26** — [Guney：Claude Code 与 Codex 共用 `.agents/skills`](https://x.com/guneysol/status/2081365120194674767)：作者把共享 skills 与可复用提示放进 `.agents/skills`，再链接到 `.claude/skills`，以一份源文件同时服务 Claude Code 和 Codex。（674 查看）
- **2026-07-29** — [Zane Chen：同一 skill 库覆盖 13 个 coding tools](https://x.com/chenzeling4/status/2082361235597828456)：作者认为 Claude、Codex、Cursor 用户都在重复编写相同 skills，并推广一套覆盖 13 个工具的共享库。（267 查看）
- **2026-08-10** — [AoYi：六个 harness 共用一套能力源文件](https://x.com/Aoyi21/status/2086704097999118382)：帖子介绍一个统一 marketplace，Claude Code、Codex CLI、Cursor、OpenCode、Gemini CLI、GitHub Copilot 可消费相同的 plugins、agents、skills 和 commands 源文件。（178 查看）
- **2026-08-19** — [EK：skills.sh 下载量与 Claude/Codex 安装支持](https://x.com/ekcheungAI/status/2089970642921660742)：帖子称 mattpocock/skills 在 skills.sh 累计 1,350 万次下载，并点出 Claude marketplace 与 Codex 均支持，视其为 agent 工作流标准化信号。（61 查看）
- **2026-07-25** — [transitions.dev：同一 UI 能力既供人用，也作为三家 agent skill](https://x.com/Jakubantalik/status/2081036752924352811)：同一套 UI transitions 可作为 CSS/React 代码，或通过 `npx skills add` 装给 Claude、Cursor、Codex。（8 万查看）

## 3. 按任务挑 harness 的多 harness 实践

- **2026-07-09** — [anupamrjp：Codex 构建，Claude 审查](https://x.com/anupamrjp/status/2074902861264326775)：作者认为最佳方案不是二选一，而是 Codex + Claude，一个负责构建、一个负责审查。（814 查看）
- **2026-07-24** — [pcshipp：同一项目中 Codex 写代码、Claude Code 调试](https://x.com/pcshipp/status/2080638718470504508)：作者让 Codex 编写代码，同时把 Claude Code 当同事负责调试，并称这种分工还能节省 Claude 每周额度。（1,493 查看）
- **2026-07-29** — [alon：三种 harness 按运行方式分工](https://x.com/alonzuman/status/2082493337378963918)：作者同时使用 Codex、Claude Code、Cursor，分别承担远程控制、补贴的云端 agent、异步工作。（452 查看）
- **2026-08-15** — [Vox：真实项目同时开 Claude 与 Codex](https://x.com/Voxyz_ai/status/2088596581587018158)：作者把 Claude 与 Codex 各放一个 pane，一个构建、一个审查，由中间的主 agent 派活；同时按任务难度选择 Sol Medium、Sol High 或 Luna，并说明 Codex 日常更顺手、Claude 的 hooks 和团队 agent 更成熟。（1.8 万查看）
- **2026-06-28** — [Sam：Codex 适合快速原型，Claude Code 适合讲究质量的实现](https://x.com/futurenomics/status/2071086359276671461)：高传播讨论把 Codex 描述为快速交付可运行原型，把 Claude Code 描述为先追问、后交付更优雅代码，评论区进一步概括为“Claude 做抽象推理，Codex 做纯效率”。（49 万查看）
- **2026-07-04** — [Rohil Agarwal：不愿再把工作流锁在一家模型公司](https://x.com/rohil_ag/status/2073218310129545553)：作者先后长期使用 Claude Code、Codex 后回到 Cursor，理由是希望自由选择模型生态；它是切换实践，不是同时并用，但直接支持 harness 中立的动机。（4 万查看）

热度结论：**“原生 harness 是否更强”热度高但无共识，正反帖子达到 9.1 万至 61.3 万查看；跨 harness skills 已形成明确安装与单一来源实践；真正按任务并用多 harness 的帖子数量较少、通常为数百至 1.8 万查看，但已有可复述的构建/调试/审查分工。实际访问方式：直连 X 已登录页面，未用 relay，未撞登录墙。**