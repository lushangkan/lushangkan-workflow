# 规则出处

## 摘要

`SKILL.md` 的 41 条规则来自 **4 个来源**，每条都能追到具体条款或实证。

| 来源 | 性质 | 贡献规则 |
|---|---|---|
| **W3C COGA** | 认知障碍无障碍指南 | 工作记忆上限、用词、指令拆分、内容量 |
| **ISO 24495-1:2023** | 首个国际平易语言标准 | 四原则骨架、信息顺序、标题、验收方法 |
| **WCAG 2.2 Readable** | 无障碍规范 AAA 级 | 术语定义、缩写、阅读难度上限 |
| **实证分析** | GPT 与 Claude 同题对比 | 元话语、句式、自造词、决策点收敛 |

另有两个参考对象提供表述方式：ASD-STE100 受控语言（一词一义、一句一事、警告先于步骤）经 `Fenng/tech-doc-style-chinese` 适配到中文；`ayghri/i-have-adhd` 提供输出塑形和发送前检查。

## W3C COGA

文档：<https://www.w3.org/TR/coga-usable/>

| 条款 | 原文要点 | 对应规则 |
|---|---|---|
| 2.2 背景 | People with impaired working memory may only be able to hold **one to three items** in their memory at the same time | M1、B6 |
| 3.5.1 Distractions | 失焦后需要 reminders of what I was doing、where I am、what I just did | M5 |
| 3.6.1 Previous Steps | 流程不能依赖记忆，需要能看到前面步骤输入的信息 | M2、M3 |
| 3.7.5 Task Management | 需要知道怎么开始、涉及哪些步骤、要多少工夫、选项的优劣 | A3、D4、D7 |
| 4.2.4 Make Each Step Clear | 多步流程要显示已完成、当前、待办和关键选择 | D2 |
| 4.4.1 Use Clear Words | 用常见词；删含糊词如 and so forth；**Do not invent new words or give words new meanings** | C1、C10 |
| 4.4.3 Avoid Double Negatives | 不用双重否定表达肯定；不用从句套从句 | C7 |
| 4.4.4 Use Literal Language | 用具体可感知的词；比喻必须附解释 | C5 |
| 4.4.5 Keep Text Succinct | 段落一事、主旨前置、一句一意、用列表、短标题 | C7、B2 |
| 4.4.8 Provide Summary | 长文档给简短摘要，关键词加粗；**不到 300 字时标题即摘要** | B3 |
| 4.4.9 Separate Each Instruction | 每步分开，包含"显而易见"的步骤，复杂条件用 if/then 表 | D1、B4 |
| 4.4.13 Numerical Concepts | 给数字概念替代表达（认知障碍者易误解百分比） | C12、M6 |
| 4.6.1 Limit Interruptions | 避免打扰 | B7 |
| 4.6.2 Short Critical Paths | 只保留必需步骤，可选步骤分离且不强制 | D3 |
| 4.6.3 Avoid Too Much Content | 每屏 5 个以内主要选项 | B6 |
| 4.7.5 Do Not Rely on Memory | 每步必须自带所需信息，**must not rely on memory from prior steps**；不要求计算、抄写、记字符串；说明和标签放在动作之前 | M3、M4、M6 |
| 5.5.6 测试 | 在疲惫或有压力的状态下检验能否完成 | 验收方法 |
| 6.7.2 Maria 场景 | 信息必须出现在**使用它的那一刻**（at the exact point of use） | M2 |
| 6.6.5 Kwame 场景 | 复杂呈现会让认知直接停摆 | M1 |

### 不采纳的条款

| 条款 | 内容 | 原因 |
|---|---|---|
| 4.4.2 | 主动语态、直接称呼读者 | 中文输出默认如此，写了不改变行为 |
| 4.4.7 | 元音和变音符号 | 阿拉伯语、希伯来语专用 |
| 4.4.10、4.4.11 | 留白、前景背景对比 | 视觉排版，Markdown 已解决 |
| 4.4.12 | 解释讽刺、玩笑、表情 | 规则已禁止这类表达，不需要解释机制 |
| 4.8.1、4.8.4 到 4.8.7 | 人工帮助、表单帮助、反馈、提醒 | 产品功能，不是文本输出 |

## ISO 24495-1:2023

标准页：<https://www.iso.org/standard/78907.html>

四条治理原则：relevant（给读者需要的）、findable（容易找到）、understandable（容易读懂）、usable（容易使用）。`SKILL.md` 的规则分组直接用这四条。

| 条款 | 原文要点 | 对应规则 |
|---|---|---|
| 5.1.6 | 选读者需要的内容，删掉不需要的 | A1 |
| 5.2.2 a | 最重要的信息放在读者容易找到的位置，通常是开头 | B1 |
| 5.2.2 c | 操作类内容按实际执行顺序排列 | D1 |
| 5.2.2 e | 不遵守流程会造成损害时，警告放在指令之前 | B5 |
| 5.2.4 | 用标题帮读者预测下文 | B2 |
| 5.3.2 | 选熟悉的词 | C1 |
| 5.3.3、5.3.4 | 写清晰句子、写简洁句子 | C7 |
| 5.4.2 到 5.4.4 | 持续用真实读者检验 | 验收方法：读者看不懂就是违规 |

## WCAG 2.2 Readable（准则 3.1）

文档：<https://www.w3.org/TR/WCAG22/#readable>

| 条款 | 级别 | 原文要点 | 对应规则 |
|---|---|---|---|
| 3.1.3 Unusual Words | AAA | 对以不寻常或受限方式使用的词（含习语和行话）提供定义机制 | C1 |
| 3.1.4 Abbreviations | AAA | 提供缩写全称机制 | M6 |
| 3.1.5 Reading Level | AAA | 文本要求的阅读能力超过初中水平时，提供补充内容或不超过该水平的版本 | C11 |

## 实证分析

素材：`docs/analysis/2026-07-25-gpt-vs-claude/`（同一会话里 GPT-5.6 答 21 条、Claude Opus 4.8 答 53 条，含同题对比）

| 观测 | GPT | Claude | 对应规则 |
|---|---|---|---|
| 引号自造词 | 50 处 | 16 处 | C1 |
| "不是 X 而是 Y" | 9 次 | 0 次 | C4 |
| "我认为 / 我建议" | 9 次 | 1 次 | C3 |
| 代码块 | 114 个 | 12 个 | B4 |
| 结尾追加开放选项 | 有 | 无 | A2 |
| 开头建分类法 | 有（4 项） | 无 | M1 |

A/B 实验：`docs/analysis/ab-test-output-style/`

| 注入内容 | 字数 | 自造词 | 代码块 | 表格行 |
|---|---|---|---|---|
| 无规范 | 16859 | 23 | 56 | 23 |
| 只给用词规则 | 13024 | 9 | 42 | 10 |
| 全套规则 | 9570 | 3 | 0 | 114 |

结论：只给用词规则治不了结构问题，两层都要。

## 参考对象

- **ASD-STE100**：航空工业受控语言，约 900 个批准词加 53 条规则，核心是一词一义、一句一事、主动语态、警告先于步骤。中文适配见 `Fenng/tech-doc-style-chinese` 的受控中文技术写作模块，它明确说明只取可迁移方法，不移植英文词性和时态规则。贡献 C8、C9、A4。
- **asd-ste100-skill**（`danyuchn/asd-ste100-skill`）：STE 面向 agent 输出的实现。三条机制被本 skill 采用：
  - 名词堆叠上限（英语 ≤3 词），中文版即 C14 定语最多两层。
  - 领域术语合法：STE 允许项目级词汇表，专业名词照用、首次定义即可；简化针对句子结构和普通词汇，不针对术语。C11 据此明确边界。
  - 不为简短牺牲清晰：目标是消除歧义，不是字数最短。压缩过头反而浪费读者时间。
- **ayghri/i-have-adhd**：输出塑形 skill。贡献 B1、B7、D2、D5、D6 和发送前检查的做法。它要求给具体时间估算，本 skill 改为 A3（有依据才给），与编造估算的风险取舍后选择保守做法。
