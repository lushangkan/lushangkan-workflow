# 调研：aaddrick/attention-control 与 output-style 的融合评估

调研日期：2026-08-20。结论均带来源 URL。查不到的内容标「未确认」。

## 1. 它是什么

**一个面向 coding agent 的"输出风格"规则集，专为 ADHD 读者写。** 不是 hook、不是工具、不是插件逻辑，是纯文本规则（系统提示级别的输出规范），通过插件/技能/规则文件分发到各 harness。

- 仓库：https://github.com/aaddrick/attention-control （作者 aaddrick，2026-08-02 创建）
- 定位（README 原文）："Air traffic control discipline for agent output. Written for a reader with ADHD."——航空管制纪律应用于 agent 输出。
- 解决什么问题：agent 的回复对 ADHD 读者不可读。航管解决"负载下分心的人类会听错指令"用两招——受控词汇（一词一义）+ 固定消息形状（指令在前、背景在后）。该风格把这两招套到 coding agent 输出上：**开头给可执行动作，每句话一词一义**。
- 双层结构：
  - **Shape 层**（11 条规则）：决定说什么、什么顺序。来源 `ayghri/i-have-adhd`（MIT）。
  - **Language 层**：决定每句话怎么读，是 ASD-STE100 受控英语（Issue 9）的浓缩。来源 L1nefeed 的 asd-ste100 gist（https://gist.github.com/L1nefeed/4164ecaaf77879e76dca3c06f142f1c2）。
- 形态：唯一权威源是 `output-styles/attention-control.md`，`scripts/sync_style.py` 生成各平台副本——Claude Code 插件（唯一有原生 outputStyle 槽）、Codex 插件、Cursor rules/skill、Gemini extension、Zed、AGENTS.md 块。
- 附带评测管线（`evals/`）：24 个 case（16 dev + 8 holdout）、6 维评分、盲评、双 pass 翻转率、发布门、comparability ledger。两轮已冻结 run 都过门（加权 +0.449 / +0.491）。

## 2. 核心机制逐条

### Shape 层 11 条（防什么行为）

| # | 规则 | 防止的行为 |
|---|---|---|
| 1 | 首行是读者当下能做的动作；是事实题就首行给事实 | 铺垫吞掉答案，读者找不到入口 |
| 2 | 自己能做完的活绝不推回给读者（此规则压过所有规则） | 把 5 步里自己能做 4 步的活变成读者跑 5 步 |
| 3 | 多步工作编号，一步一个动作，一步内不出现两次 "and then" | 步骤糊成一团，读者中途迷路 |
| 4 | 收尾给一个具体下一步动作，两分钟内能做，"打开文件"也算 | 回复结束在"完成"上，读者不知道下一步 |
| 5 | 压制离题。做完整件事，第二个话题单独作为问题提出 | 一个回复开多个线程，注意力被撕开 |
| 6 | 每轮重述状态："第 3/5 步完成，我改了 schema。下一步：跑 backfill" | 工作记忆装不下跨消息的进度 |
| 7 | 时间估算给具体单位（"约 15 分钟；没测试覆盖就是一下午"） | "有点工作"和"几小时"在读者心里一样重 |
| 8 | 改动后说出现在什么能用了（"登录用 magic link 能用了"） | 埋没的成果不产生多巴胺反馈 |
| 9 | 报错给位置、原因、修法，前面不写"哎呀" | 恐慌词和真信息抢注意力 |
| 10 | 列表 ≤5 项，超了拆"现在做/以后做"或"必须/可选" | 一屏超过 5 个选项读者停摆 |
| 11 | 无开场白、无回顾、无结束语 | 寒暄消耗本已稀缺的注意力 |

设计依据（README 原文）：ADHD 阅读五个事实——工作记忆小（屏幕外即消失）、知道答案≠做出答案、启动最难、时间感失真、多巴胺稀缺。每条 shape 规则对应一个事实。

### Language 层（受控英语）

- 一词一义，一个动作一个动词，不轮换同义词；标准动词表（check/make sure/start/stop/use/show/find/change/remove/need，各配禁用的近义词）。
- 主动语态、点名行动者；只用简单时态，禁用完成时和助动词堆叠；祈使句下指令。
- 指令句 ≤20 词，说明句 ≤25 词；一句一个指令；名词堆叠 ≤3 词。
- 结构：一段一个话题，段 ≤6 句；3 步以上用编号列表，3 项以上并列用项目符号。

### Precedence（冲突仲裁）与 Pre-send check

- 冲突仲裁 4 条：任务题首行动、事实题首结果、不知道时首"缺口"；删整句不删冠词主谓（可运行性是底线）；**删套话、留不确定**——"也许/可能"删掉，不确定就直说"我没看过你的 schema"，**绝不编造版本号/日期/行号来填空**，改用能查证的命令；列表 3 项起用、5 项上限。
- 发送前检查 6 删 2 查：删开场宣告句、删"还要别的吗"收尾、删 by the way、删无信息副词和编造的具体值、删习语比喻（"circle back"）、删完成时/被动/长名词堆叠；最后查两件事——只读首行末行能否知道该做什么和刚发生什么、每词是否一词一义。

## 3. 活跃度和许可证

| 项目 | 数据 | 来源 |
|---|---|---|
| 创建 | 2026-08-02 | https://api.github.com/repos/aaddrick/attention-control |
| 最后 push | 2026-08-09（10 天密集开发后停更） | 同上（pushed_at） |
| Stars / Forks | 83 / 2 | 同上 |
| Open issues | 1 | 同上 |
| 许可证 | MIT（注明派生自 ayghri/i-have-adhd，MIT；ASD 原文零复制，见 NOTICE.md） | https://github.com/aaddrick/attention-control/blob/main/LICENSE |
| 评测 | 2 轮冻结 run，均过发布门 | https://github.com/aaddrick/attention-control/blob/main/evals/results/LEDGER.md |

判断：**活跃度中低**——项目不到 3 周、单维护者、已停更约 11 天，但 18 天 83 star 说明口碑不错，评测工程水准高。许可证干净（MIT），与后续融合无冲突。

## 4. 与 output-style 的融合点

**关键洞察：attention-control 的两个来源（i-have-adhd + ASD-STE100）正是 output-style 的 sources.md 里已引用的参考对象。** 它不是新大陆，而是"同一批来源针对 coding agent 逐轮输出的英文化再合成"，且多了一套评测。

### 层面分工

| 维度 | output-style（41 条，中文） | attention-control（英文） |
|---|---|---|
| 读者 | 人（会话回复 + 文档 + Issue/PR） | coding agent 的逐轮输出 |
| 骨架 | ISO 24495-1 四原则（relevant/findable/understandable/usable） | 航管两纪律（受控词汇 + 固定形状） |
| 强度 | 更严：新概念 ≤3 项、待决策 ≤2 个 | 较松：列表 ≤5 项 |
| 语言 | 中文；不移植英文时态/词性 | 英文 STE（标准动词表、时态规则） |

### 重叠（已覆盖，无需动作）

B1↔规则1（首行可执行）、B7↔规则5（第二话题单独问）、D1↔规则3（多步编号）、D2↔规则6（显示进度）、M5↔规则6（每轮给位置）、D5↔规则9（报错平述）、D6↔规则8（说清怎么验证）、C8↔一词一义、C14↔名词堆叠（中文版两层）、B5↔警告前置、C3/A4↔删套话留不确定、让位条款 1/2/3/4↔When to break 1/2/3/5。

### 有意差异（保持，不合并）

- **A3 时间估算**：attention-control 要求"永远给具体单位"；output-style 选"有依据才给，没依据说不知道"（sources.md 已记录这一取舍，防编造估算）。保持 A3。
- **决策点上限**：output-style 的 3 项/2 决策比 attention-control 的 5 项更严，无需回退。

### 真缺口（attention-control 有、output-style 没有）

1. **收尾动作规则**（shape 规则 4）：单条回复结尾给一个 2 分钟内能做的具体动作。output-style 的 D2 只管多步任务，pre-send 第 7 条只隐含没成规则。这是对 ADHD 用户价值最大的一条。
2. **显式禁语清单**（shape 规则 11）：点名禁"Great question / Let me… / Hope this helps / Let me know if you need anything else"。output-style 只规定"第一行是判断不是铺垫"，没有点名中文禁语（好的！/没问题！/让我看看/希望能帮到你/有问题随时说）。
3. **防编造具体值**（Precedence 3）：不知道时"用能查证的命令代替编造的版本号/日期/行号"。A4 有"数字只写来源里有的"，但缺"给查证命令"这个动作。
4. **（可选）删习语比喻**（pre-send 5）：C10 覆盖"等等/一些"含糊词，未点名中文习语（事半功倍、双刃剑）。

### 不并进 output-style、另放的东西

1. **shape 规则 2"Do the work you own"**：这是 agent 行为规则（自主性），不是文本规则，应放 omp 主 agent 指令层/AGENTS.md。注意评测显示这条规则本身没解决 agent 否认能力的问题（agent-owned-edit 24 个 case 全灭），行为层要单独治。
2. **STE 英语语法层**（标准动词表、时态、词数上限）：英文专属，中文不适配；output-style 已有中文适配（C8/C9/C14/C1）。
3. **评测管线**（盲评、holdout、双 pass 翻转率、comparability ledger）：不是规则是方法。output-style 已做过四档回归；若要更严，可借鉴这套结构。**重要诚实点**：两轮评测加权增益的 66–75% 来自 language/concision 两维，而这两维的评分细则正是风格自己的规则（有循环论证风险）；correctness/autonomy/safety 三轴无显著变化——即该风格改善"散文控制"，不改善正确性。

## 5. 融合方案建议

**动小手术，不整体移植**（同源已覆盖约九成）：

| 动作 | 内容 |
|---|---|
| 并进 output-style（3 条） | ① usable 组新增：回复收尾给一个两分钟内能做的具体动作；② findable 组新增禁语清单（开场禁"好的！/没问题！/让我看看"，结尾禁"希望能帮到你/有问题随时说"）；③ 增强 A4：不知道具体值时给能查证的命令，不编造版本号、日期、行号 |
| 增强（可选） | C10 加一句：删习语和比喻，用字面说法 |
| 另放 | shape 规则 2（自主性）→ omp 主 agent 指令层；STE 英语语法 → 不搬；评测管线 → 将来做回归时作方法论参照，另立 |
| 保持差异 | A3 估算保守化（已有记录）；3 项/2 决策上限不回退 |

许可证：MIT 且与已引用来源同源（i-have-adhd、asd-ste100 均已写在 sources.md），中文改写融合不产生新义务；若整段搬运英文原文需保留版权声明。

---

**融合价值：中** —— 两者同源（i-have-adhd + STE），output-style 41 条已覆盖其约九成机制且中文适配更严，真正增量只有 3 条小规则加一套可借鉴的评测方法论，值得吸收但不构成体系性融合。
