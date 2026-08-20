# Reddit ADHD 社区：AI coding skills、工具与做法

## 结论

最值得吸收的不是“让 agent 多写代码”，而是把 AI 当作**外置执行功能**：记住当前目标、压缩输出、提醒回场、处理机械步骤。

社区对全自动 agent 明显分裂。高热讨论反复指出：大量并行生成会把开发变成枯燥的 PR 审阅，并制造难以承受的上下文负担。对单人、ADHD、每天约 6 小时的工作方式，更合适的是：**小票异步派活 + 自动通知 + 小 diff + 人保留设计和有趣部分**。

> 调研方法：2026-08-20 通过用户本机 Chrome 搜索 Reddit。覆盖 r/ADHD、r/ADHD_Programmers，并补充一条直接讨论 ADHD skill 的 r/ClaudeAI 高热帖。赞数会被 Reddit 模糊处理，以下为检索时快照。

## 大家喜欢的东西

| 名称 / 做法 | 热度 | 一句话为什么喜欢 | 链接 |
|---|---:|---|---|
| `i-have-adhd` skill | **约 2,895 赞 / 450 评论**；r/ADHD_Programmers 复帖仅 0 赞 / 10 评论 | 让 Claude 停止长篇漫谈，先给下一步；编号、限制列表长度、重复当前状态、显示进度。高热帖认可“方向”，但有人认为太长、会失效，也有人偏爱详细答案，必须允许个性化。 | [高热主帖](https://old.reddit.com/r/ClaudeAI/comments/1v8o1jn/whoever_created_the_adhd_skill_god_bless_you/) · [ADHD 程序员社区帖](https://old.reddit.com/r/ADHD_Programmers/comments/1vit75d/anyone_used_the_ihaveadhd_agentic_skill/) · [原 skill](https://github.com/ayghri/i-have-adhd) |
| Claude skills：工作日志、每日查 ticket、跟踪 MR、Slack 提醒 | **54 赞 / 57 评论**；支持性评论 11 赞 | 充当“外置执行功能”：跑偏后能恢复原目标，不再忘记已开但搁置的 MR；临期和逾期状态可视化。 | [Work has fully embraced AI and I love it](https://old.reddit.com/r/ADHD_Programmers/comments/1t7a2k8/work_has_fully_embraced_ai_and_i_love_it/) |
| 完成 / 等待输入时发 OS 声音或桌面通知 | 两帖分别 **26 赞 / 33 评论**、**34 赞 / 35 评论**；相关顶评 45、33 赞 | 30 秒到 10 分钟的等待最容易触发刷网页；通知把人拉回任务，不必一直盯 agent。 | [30–60 秒等待讨论](https://old.reddit.com/r/ADHD_Programmers/comments/1ulgnsj/what_do_you_do_in_the_3060s_while_claude_code/) · [5–10 分钟等待讨论](https://old.reddit.com/r/ADHD_Programmers/comments/1sribex/how_to_deal_with_wait_time_when_working_with_ai/) |
| AI 只做窄范围任务：函数、重复代码、测试草稿、文档、解释、找 bug | **180 赞 / 102 评论**；另一工作流帖 10 赞 / 33 评论 | 保留人类的解题乐趣和系统理解，把无聊、重复、查资料的部分交给 AI；比整项目生成更容易审查。 | [AI coding removes dopamine](https://old.reddit.com/r/ADHD_Programmers/comments/1u3s334/anyone_else_find_ai_coding_removes_your_source_of/) · [Successful workflow?](https://old.reddit.com/r/ADHD_Programmers/comments/1umgwgy/had_anyone_successfully_worked_ai_into_their/) |
| 小步 plan → 单个 agent → 逐步验证；“短牵引绳” | **22 赞 / 17 评论**；顶评 31 赞 | 一次生成 9 PR / 55 文件会造成“速度陷阱”；社区更认可一项一项做、边写边看、每个行为一个小改动。 | [AI velocity trap](https://old.reddit.com/r/ADHD_Programmers/comments/1t6yucw/ai_velocity_is_becoming_a_trap_for_me_adhd_huge/) |
| 先写功能规格 / Gherkin，再实现并跑功能测试 | 所在帖 **10 赞 / 33 评论**；建议评论 5 赞 | `Given / When / Then` 先固定行为，减少 agent 偷带假设；最后以功能测试闭环。属于低热但方向明确的工程建议。 | [相关评论](https://old.reddit.com/r/ADHD_Programmers/comments/1umgwgy/had_anyone_successfully_worked_ai_into_their/ovc7czj/) |
| AI 作为互动式编程导师：先给大图，再给短解释和多个代码例子 | r/ADHD **23 赞 / 11 评论**；支持评论 5、7 赞 | 比通读文档更容易启动；能快速试多个例子，降低新技术的入门摩擦。社区也提醒模型可能给错信息。 | [Learning has gotten way easier recently with AI](https://old.reddit.com/r/ADHD/comments/1jeu2an/learning_has_gotten_way_easier_recently_with_ai/) |
| GitHub 事件触发专用 agent；任务运行期间离开，回来收 PR | 所在帖 **10 赞 / 33 评论**；相关评论 1–2 赞 | 真正异步，不把 30–60 秒等待塞进清醒工时；成熟仓库、自动 lint / test、清晰规则是前提。证据热度低，属于小众实践，不代表社区共识。 | [GitHub 编排评论](https://old.reddit.com/r/ADHD_Programmers/comments/1umgwgy/had_anyone_successfully_worked_ai_into_their/oveg6i0/) · [异步交付 PR 评论](https://old.reddit.com/r/ADHD_Programmers/comments/1umgwgy/had_anyone_successfully_worked_ai_into_their/ovcdusw/) |

### 工具名出现情况

- **Claude / Claude Code**：出现最频繁。被喜欢的不是品牌本身，而是 skills、项目上下文、长任务和终端集成。
- **GitHub Copilot**：有人用来先讨论架构，再生成 DTO 等重复代码；也有人只保留 autocomplete。相关支持评论约 9 赞。[来源](https://old.reddit.com/r/ADHD_Programmers/comments/1u3s334/anyone_else_find_ai_coding_removes_your_source_of/or7m1fp/)
- **Cursor**：常与 Claude 一起出现，但评价混合。最大抱怨是大任务产生过量代码和 review 负担，不是缺少更强模型。[来源](https://old.reddit.com/r/ADHD_Programmers/comments/1t6yucw/ai_velocity_is_becoming_a_trap_for_me_adhd_huge/)
- **ChatGPT**：r/ADHD 中更常作为解释、拆任务和学习助手，而非自主 coding agent。[来源](https://old.reddit.com/r/ADHD/comments/1jeu2an/learning_has_gotten_way_easier_recently_with_ai/)
- **Obsidian + Claude Code**：有人喜欢用 Claude 降低搭建个人知识库的门槛，但仅 **6 赞 / 5 评论**，采用度未确认。[来源](https://old.reddit.com/r/ADHD_Programmers/comments/1u7d1ly/claude_code_obsidian_is_a_great_way_to_build_your/)

## 反复出现的痛点与民间解法

| 反复出现的痛点 | 证据与热度 | 民间解法 | 对我们的判断 |
|---|---|---|---|
| **等待把注意力切断**：30 秒到 10 分钟足以开始刷 Reddit；回来时忘了原任务 | [30–60 秒讨论](https://old.reddit.com/r/ADHD_Programmers/comments/1ulgnsj/what_do_you_do_in_the_3060s_while_claude_code/) 26/33；[5–10 分钟讨论](https://old.reddit.com/r/ADHD_Programmers/comments/1sribex/how_to_deal_with_wait_time_when_working_with_ai/) 34/35；通知建议分别获 45、33 赞 | 完成 / 等待输入时发 OS 通知；等待时只做纸上笔记、伸展、查看同一任务的 preview，不打开无限滚动内容 | **应吸收。** 异步任务不要让人盯；所有需要人接手的状态统一通知，并附“刚才在做什么 / 下一步”。 |
| **上下文丢失与工作记忆超载**：多终端后只剩“这个在干什么”；handoff 越写越长 | [多终端讨论](https://old.reddit.com/r/ADHD_Programmers/comments/1ulgnsj/what_do_you_do_in_the_3060s_while_claude_code/)中相关评论 29 赞；[55 文件案例](https://old.reddit.com/r/ADHD_Programmers/comments/1t6yucw/ai_velocity_is_becoming_a_trap_for_me_adhd_huge/) 22/17 | 工作日志、当前 ticket / MR 清单、嵌套短清单、明确 current step；agent 输出短状态摘要；无关发现进 backlog，不在当前会话展开 | **应吸收。** 任务状态由系统保存，不依赖记忆；交接只写目标、已完成、下一步、阻塞和验证证据。 |
| **AI 夺走解题的 dopamine**：工作变成 prompt + 等待 + review 循环 | [AI has sapped dopamine](https://old.reddit.com/r/ADHD_Programmers/comments/1v2hcis/ai_has_sapped_all_the_dopamine_out_of_programming/) 516/117；[相似讨论](https://old.reddit.com/r/ADHD_Programmers/comments/1u3s334/anyone_else_find_ai_coding_removes_your_source_of/) 180/102；[AI Burnout](https://old.reddit.com/r/ADHD_Programmers/comments/1vmyscu/ai_burnout/) 146/70 | 人保留架构、关键决策和想写的代码；AI 做重复代码、测试草稿、文档、解释和小 bug；必要时退回 function-completion / pair 模式 | **应设边界。** “尽量让 agent 写”不是目标。异步只用于清楚、机械、可验收的小票。 |
| **生成速度超过 review 带宽**：冗余逻辑、过度抽象、55 文件 diff、第二天重写 | [Velocity trap](https://old.reddit.com/r/ADHD_Programmers/comments/1t6yucw/ai_velocity_is_becoming_a_trap_for_me_adhd_huge/) 22/17，顶评 31 赞；[AI code sucks](https://old.reddit.com/r/ADHD_Programmers/comments/1mcx6xb/ai_code_sucks/) 103/105 | 一次一个小行为；限制文件数；先 plan 后执行；每步验证；要求先用现有工具，不准无请求新增抽象；测试 / contract 先行 | **强制吸收。** 用“小 diff 可审查”控制吞吐。并行度由 review 能力决定，不由 agent 数决定。 |
| **输出太长、假设和幻觉太多**：spec 变成数页文字，规则在长上下文中失效 | [AI Burnout](https://old.reddit.com/r/ADHD_Programmers/comments/1vmyscu/ai_burnout/) 146/70；[`i-have-adhd` 讨论](https://old.reddit.com/r/ClaudeAI/comments/1v8o1jn/whoever_created_the_adhd_skill_god_bless_you/) 2,895/450，但评论明确“会忘、需提醒” | 先给下一步；短编号；一次一个决定；重复当前状态；功能规格 / 测试作为外部真相源；新 session 重新注入必要规则 | **应吸收但不照搬。** output-style 已覆盖主要价值；保留个性化开关，避免把 ADHD 简化成“所有人都要极短”。 |
| **项目和切线爆炸**：几十个半成品、五个新问题替代原任务 | [等待讨论](https://old.reddit.com/r/ADHD_Programmers/comments/1sribex/how_to_deal_with_wait_time_when_working_with_ai/)中有人报告“30 个 vibe-coded 项目”；[个人 agent 规则帖](https://old.reddit.com/r/ADHD_Programmers/comments/1va188u/i_originate_most_of_my_work_from_a_coding_agent/)提到阻止调 shell、反复抛光 | 将无关发现记入分级 backlog；系统提示阻止“继续抛光而不发布”；每次只给一个可执行下一步；完成前不自动扩 scope | **应吸收。** agent 发现额外问题只记录，不顺手改；默认把当前小票送到可交付状态。 |

## 值得我们吸收的

### 1. 把 `output-style` 保持为常驻无障碍层

`i-have-adhd` 的高热反馈验证了已有方向：结论 / 动作先行、短编号、外置状态、抑制切线、显示进度。

不建议直接并入对方 skill 全文。原因有三点：

1. 我们已有更完整、经过回归测试的 `output-style`。
2. 高热帖中有人明确偏爱详细答案；应按用户画像固定，而不是声称适合所有 ADHD 人群。
3. Reddit 反复报告长 skill 会在多轮后失效。核心规则应放常驻层，细则按需加载。

### 2. 给异步 agent 增加“回场通知契约”

每个通知只回答四件事：

- 哪张票完成或被阻塞；
- agent 做了什么；
- 验证结果在哪里；
- 人现在只需做哪一个动作。

不要让用户在 30–60 秒的碎片等待里管理多个终端。真正异步：派活后离开，完成、失败或需要决策时再通知。

### 3. 用 review 带宽限制并行度

推荐默认：**一张小票 = 一个 worktree = 一个小 PR = 一个可观察行为**。

可以后台并发，但不要一次把 9 个 PR / 55 个文件倾倒给用户。若待 review 数达到上限，停止领新票。这个约束比增加 agent 数更适合每天约 6 小时的清醒工时。

### 4. 保留人的 dopamine 区，agent 接机械区

推荐分工：

| 人保留 | Agent 优先接手 |
|---|---|
| 产品判断、架构、关键取舍、有趣的难题、最终 review | 重复代码、资料查找、测试 / 文档初稿、小而明确的实现、流水线监控 |

这不是降低自动化，而是避免把唯一有动力的部分也变成被动审阅。

### 5. 将“防过度工程”写进任务边界和审查

每张票明确：允许改哪些行为、预期 diff 大小、禁止新增哪些抽象、必须复用哪些现有模式。agent 遇到票外问题只登记 backlog。

验收顺序建议保持：行为 contract / spec → 小步实现 → 自动验证 → 双轴 review → 黑盒验收。Reddit 的 Gherkin、短牵引绳和自动测试建议都支持这个方向，但具体流程仍以我们的现有纪律为准。

## 采用判断

| 项目 | 判断 |
|---|---|
| `i-have-adhd` 的表达原则 | **融合思想，不复制 skill。** 现有 `output-style` 已是更合适的权威实现。 |
| 工作日志、ticket / MR 跟踪、下一步提醒 | **采用。** 直接缓解遗忘、跑偏和恢复成本。 |
| OS / 桌面完成通知 | **采用。** 是热度和重复度最高的具体民间解法之一。 |
| 多 agent 并行 | **有限采用。** 后台可并行；用户侧只暴露有上限的 review 队列。 |
| 全项目自主生成 | **不设为默认。** 高热负面反馈集中在 dopamine 丢失、上下文过载和 review 债务。 |
| 小票、contract、测试、短 diff | **强化采用。** 与社区最可靠的正向实践一致。 |
