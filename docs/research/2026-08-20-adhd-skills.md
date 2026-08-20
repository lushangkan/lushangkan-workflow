# UditAkhourii/adhd 调研

## 结论

这个仓库**不是“专门为 ADHD 开发者准备的一组 skills”**。截至本次读取的 `main` 分支，`skills/` 下只有一个 skill：`adhd`。它借用 ADHD 作为名称，解决的是 coding agent **过早收敛到第一个常规答案**的问题；它不是时间管理、注意力管理或执行功能辅助工具。

因此：

- **反向拷问 / 理解确认：没有。**
- **反完美主义 / 防过度投入：部分有。**它有成本门、适用范围、停止条件和陷阱筛选，但没有用户需要的完整机制。
- 最值得借鉴的是：把“发散”和“批判”机械隔离、按决策风险控制投入、强制收敛并给出第一步。

来源：

- 仓库：[UditAkhourii/adhd](https://github.com/UditAkhourii/adhd)
- 根目录：[main 分支](https://github.com/UditAkhourii/adhd/tree/main)
- skills 目录：[skills/](https://github.com/UditAkhourii/adhd/tree/main/skills)
- 唯一 skill 原文：[skills/adhd/SKILL.md](https://github.com/UditAkhourii/adhd/blob/main/skills/adhd/SKILL.md)

## Skill 清单

| Skill | 一句话说明 | 适用范围 |
|---|---|---|
| `adhd` | 让多个相互隔离的 agent 用不同认知视角并行发散，再由独立 critic 打分、聚类、排除陷阱，并深化最好的方案。 | 架构、API、命名、模糊故障、迁移、代码审查拓宽、策略等“存在多个可行答案”的高风险决策。 |

**完整性核对：**仓库根目录的 `skills/` 只含 `adhd/`，该目录只含 `SKILL.md`。没有第二个 skill，也没有隐藏的 ADHD 时间管理 skill 合集。仓库同时提供 TypeScript CLI / library、文档和评测，但它们是同一个方法的不同载体，不是额外 skills。

原文依据：

- `SKILL.md` 的 description 写明它是 “Parallel divergent ideation for coding agents”，并列出 architecture、naming、API/SDK、fuzzy-debugging 等触发场景：[原文](https://github.com/UditAkhourii/adhd/blob/main/skills/adhd/SKILL.md)
- 使用边界单列了 architecture、API、模糊 debugging、迁移、命名、code review widening 和 strategy：[When to use](https://github.com/UditAkhourii/adhd/blob/main/documentation/when-to-use.md)
- README 将它定义为解决 autoregressive reasoning “premature convergence”的架构方法，而不是 ADHD 人群的个人工作管理方案：[README](https://github.com/UditAkhourii/adhd/blob/main/README.md)

## 两个目标问题

### A. 反向拷问 / 理解确认

**判断：没有。**

目标机制是：agent 给出计划后，反过来考用户是否理解关键概念、取舍和下一步；必要时根据回答纠正理解。

`adhd` 没有以下行为：

- 不要求用户复述计划或做 teach-back。
- 不向用户出理解题，也不检查用户答案。
- 不设置“用户确认理解后才继续”的 gate。
- 不根据用户理解程度调整讲解或执行。

最接近的文字是输出格式中的 **Brief**：“用一两行确认问题和采用的 reframe”。但这是 **agent 向用户复述问题**，不是 agent 反向考用户。最后的 **Provocation** 也只是打开新方向的问题，不是理解验收。因此不能算命中。

来源：

- 输出格式与 Provocation：[SKILL.md — Output shape](https://github.com/UditAkhourii/adhd/blob/main/skills/adhd/SKILL.md#output-shape)
- 原始规格同样只有 Brief、wide set、converge、provocation，没有理解测验：[SOURCE-SPEC.md — Output shape](https://github.com/UditAkhourii/adhd/blob/main/SOURCE-SPEC.md#output-shape)

### B. 反完美主义 / 防过度投入

**判断：部分有，但不能直接替代我们的专门纪律。**

| 所需能力 | 判断 | 仓库已有内容 |
|---|---|---|
| 决策点默认值 | **部分有** | 默认运行规模是 5 个 frame × 6 个想法，再深化 3 个；按风险缩放。它提供的是“探索预算默认值”，不是产品决策的保守默认值。 |
| 耗时 / 成本提醒 | **有，调用前提醒；没有进行中的计时提醒** | Pre-flight 明说一次约 10 次 agent 调用、30–90 秒、单次回答的 5–10 倍；文档还提醒每个分支会重复加载基础上下文。 |
| 防镀金 / 防过度投入 | **部分有** | 只有开放、高风险、开放措辞三个条件都满足才自动启用；普通 side project、已知答案、lookup、已知根因 bug 不应使用。新候选开始重复时停止，不为凑数继续。 |
| 识别诱人但昂贵的方案 | **有** | critic 必须标记 hidden cost、false economy、不能扩展、premature abstraction 等“看起来诱人但其实是陷阱”的方案，并写一句原因。 |
| 强制收敛和选择 | **有** | 禁止只给一堆方案让用户自己选；必须聚类、筛掉 dead ideas、给 shortlist，并明确表达真正有希望的选择。 |
| 工时上限或持续提醒 | **没有** | 没有累计耗时、每日 6 小时预算、超时中止、阶段性提醒或时间盒状态。 |
| 用户可见的防完美主义干预 | **没有** | 不会在用户继续抛光时主动指出收益递减，也没有“够好了就交付”的明确规则。 |

关键原文要点：

1. **调用前成本门。**“This skill is expensive”，约 10 calls、30–90 秒、5–10 倍成本；不是开放且高风险的问题就中止。[SKILL.md — Pre-flight](https://github.com/UditAkhourii/adhd/blob/main/skills/adhd/SKILL.md#pre-flight-run-before-phase-1)
2. **低价值场景不投入。**高风险检查明确写了 “Side project at 11pm = no”；用户说 quick、standard、canonical、textbook、just、one-line 时也应直接回答。[同上](https://github.com/UditAkhourii/adhd/blob/main/skills/adhd/SKILL.md#pre-flight-run-before-phase-1)
3. **筛掉镀金式陷阱。**评分时标记 hidden cost、false economy、will not scale、premature abstraction。[SKILL.md — Phase 2](https://github.com/UditAkhourii/adhd/blob/main/skills/adhd/SKILL.md#phase-2--focus-critic-on)
4. **按风险缩放。**函数命名可用 3 × 4；产品定位可用 5 × 8；新候选重复已有形状时停止，“Do not pad to hit a number”。[SKILL.md — Calibration](https://github.com/UditAkhourii/adhd/blob/main/skills/adhd/SKILL.md#calibration)
5. **诚实计算真实成本。**文档指出成本主要来自每个隔离分支重复加载基础上下文，公式近似为 `N × (base_context + branch_output)`，不能只看调用次数。[When to use — Cost & speed](https://github.com/UditAkhourii/adhd/blob/main/documentation/when-to-use.md#cost--speed)

一个重要限制：用户显式输入 `/adhd` 时，skill 会**跳过其余 pre-flight**。所以成本门只能约束自动触发，不能在用户冲动投入时硬性保护。对 ADHD 用户来说，这恰好说明仍需独立的时间盒和反完美主义纪律。

## 值得拿来的其他东西

### 1. 把“发散”和“审查”机械隔离

它不靠一句“先别批评”来保证发散，而是用不同 agent 调用隔离上下文：generator 看不到其他分支，critic 在第二阶段才出现。这与我们已有的双轴 review 思路相容：**上下文隔离应由编排保证，而不是只靠提示词承诺。**

对异步工作流的用途：只在架构、公共 API、迁移方案等高影响 ticket 前开一次并行探索；产出的 shortlist 再进入正常 spec / ticket 链，不把发散塞进每张实现票。

来源：[SKILL.md — The loop](https://github.com/UditAkhourii/adhd/blob/main/skills/adhd/SKILL.md#the-loop)、[How it works](https://github.com/UditAkhourii/adhd/blob/main/documentation/how-it-works.md)

### 2. 用风险控制探索预算，而不是固定跑满流程

“函数命名 3 × 4，产品定位 5 × 8；候选开始重复就停”很适合每天只有 6 小时的约束。可抽成通用规则：**任务风险决定 agent 数和探索深度；信息增益消失就停止。**

这比“所有决策都开多 agent 圆桌”更省时间，也比只设固定 30 分钟更贴合问题价值。

来源：[SKILL.md — Calibration](https://github.com/UditAkhourii/adhd/blob/main/skills/adhd/SKILL.md#calibration)

### 3. 输出必须收敛，不能把选择负担还给用户

它把 “Here are 20 ideas, you decide” 定义为 cop-out。最终必须给 2–4 个 shortlist、说明理由，并标出最值得尝试的非显然方案。对 ADHD 用户，这能减少方案墙带来的决策瘫痪。

可拿来的规则：后台 agent 可以广泛探索，但交回给人时只保留少量经过判断的候选，并明确推荐项；完整 wide set 作为可折叠证据，不占主阅读路径。

来源：[SKILL.md — Anti-patterns](https://github.com/UditAkhourii/adhd/blob/main/skills/adhd/SKILL.md#anti-patterns)、[SOURCE-SPEC.md — Anti-patterns](https://github.com/UditAkhourii/adhd/blob/main/SOURCE-SPEC.md#anti-patterns)

### 4. 每个入选方案都要带“承重风险 + 第一具体步骤”

深化阶段不是继续写宏大方案，而是要求：4–8 句实现草图、load-bearing risk、builder 的 first concrete step，以及少量后续变体。对单人异步流程，这能让夜间 agent 产物在第二天直接转成决策或小票，而不是再做一轮“怎么开始”。

可拿来的格式：每个候选固定三栏——**怎么工作、最可能失败在哪里、下一步做什么**。

来源：[SKILL.md — Phase 2](https://github.com/UditAkhourii/adhd/blob/main/skills/adhd/SKILL.md#phase-2--focus-critic-on)

### 5. 三个可直接复用的约束视角

- **`$0 budget, 1 hour`：**强迫找出只保留承重价值的最小方案，直接对抗镀金。
- **`3am on-call`：**从维护负担而不是开发新鲜感判断设计。
- **`competitor trying to break it` / inversion：**主动发现常规方案的失败模式，再反推防线。

这些 frame 不必引入整套 10-call 流程。可以作为 wayfinder、grill 或 code review 中的可选检查视角，只有遇到高风险决策才启用。

来源：[SKILL.md — Frames](https://github.com/UditAkhourii/adhd/blob/main/skills/adhd/SKILL.md#frames)

## 采用建议

**不要把这个仓库当成 ADHD 个人工作管理合集整体引入。**只考虑抽取以下三条机制：

1. 高风险且多解的决策才发散；其他情况直接走常规链。
2. 发散分支与 critic 隔离；critic 必须标记诱人陷阱。
3. 给用户的结果强制收敛到少量候选，并附承重风险和第一步。

反向理解测验、每日工时预算、进行中耗时提醒、收益递减提醒和“够好就交付”仍需我们自己的 skill 或 workflow gate；这个仓库没有覆盖。
