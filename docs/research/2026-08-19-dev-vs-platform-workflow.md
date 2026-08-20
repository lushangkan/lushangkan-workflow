# 调研：开发链之外要不要一层"阶段拆解"——外部讨论汇总

调研日期：2026-08-19。范围：Reddit、Hacker News、个人博客、GitHub 讨论区、框架官方文档。

**一句话结论：单人用 AI 做大项目时，"缺一层高于单次改动的规划"是最常被吐槽、最有翻车实录的问题；社区共识解法不是上重量级平台层，而是加一层很薄的里程碑规划。**

---

## 一、缺"阶段拆解"层会撞什么问题（真实案例）

这一问的反面证据最多：大量翻车实录的共同根因，就是只有"功能级开发链"、没有"整个项目怎么拆阶段"的视角。

1. **架构漂移直到推倒重写。** 开发者 k10s 用 AI 辅助写了 7 个月，功能出得快，但架构逐步漂移、上下文最终"崩塌"，最后宣布回归手写。教训原文摘要："让人来设计架构，让 AI 干活"（let a human design the architecture）。
   来源：https://news.ycombinator.com/item?id=46765460 （原博文 https://blog.k10s.dev/im-going-back-to-writing-code-by-hand/ 已 403，HN 讨论仍可查）

2. **失去对代码库的理解 → 多次从头重写。** HN 长帖 "Vibe coding is mad depressing" 多人自述：纯 vibe coding 前期快，但一旦失去对代码的理解，大项目变得无法调试维护，"有些项目因为上下文丢失不得不从头重写多次"。
   来源：https://news.ycombinator.com/item?id=46227422

3. **规划与执行搅在一个会话里 → 信任崩塌。** 一位单人开发者自述"对 AI 执行失去信任"，根因是规划和执行缠在同一会话；他的解法是另建一层协调层（SQLite 任务系统），把"排阶段"从"写代码"里剥出来。
   来源：https://arturastutkus.substack.com/p/i-kept-losing-trust-in-ai-execution

4. **单次 spec 喂不饱大改动 → agent 瞎猜。** HN 上一个 2 万行 TypeScript 个人项目的开发者：工具处理"小而有边界的改动"没问题，一遇到跨文件大改动就开始编造不存在的名字和约定。ClaudeGuide 把"拆成多会话、每会话一个自包含模块"称为结构性约束，"不是提示词问题"。
   来源：https://news.ycombinator.com/item?id=43683927 、https://claudeguide.io/claude-code-large-codebase

5. **AI 看不见改动的影响面。** HN "Mikk" 帖：单人开发者的 AI 在大型项目上失败，因为"AI 不知道什么会被改坏"（your AI wrote the code but doesn't know what breaks）——缺的不是更好的模型，是架构级的全局视角。
   来源：https://news.ycombinator.com/item?id=47387807

6. **反面约束：25 万行也能成，但靠的就是阶段纪律。** 一个单人 6 个月、25 万行的项目复盘：AI 能扛大项目，前提是作者自己维持了规划与验证的分层。摘要原文："exposing new risks at scale"。
   来源：https://medium.com/@ibare/what-i-learned-building-a-250k-line-system-with-ai-over-six-months-92b785f7374e

小结：吐槽集中在四点——架构漂移、上下文丢失导致的重写、agent 对影响面瞎猜、规划与执行不分。这四点全部是"项目级/阶段级"问题，不是"某个功能没写好"的问题。

---

## 二、PRD 粒度之争：一次改动一个 PRD vs 整个产品一个 PRD

这不是非此即彼的争论。社区较一致的立场是：**两种粒度服务不同读者，缺哪层都疼；争议点在"产品级那层要多重"。**

| 立场 | 主张 | 理由 | 来源 |
|---|---|---|---|
| 功能级 spec（Spec Kit 风格） | 每次改动一份 spec，直接喂给 agent 执行 | spec 是"AI 要执行的东西"，必须边界清晰、可验证；一个 PR 都审不完就说明 spec 切错了 | https://tekk.coach/spec-driven-development/how-granular-should-a-spec-be-one-per-feature-subsystem-or-milestone/ |
| 产品级 PRD（BMAD 风格） | 整个产品一份 PRD，再拆成 epic/story | 没有产品级意图文档，功能级 spec 各自为政，架构无法被约束 | https://github.com/bmad-code-org/BMAD-METHOD/blob/819d373e/docs/reference/workflow-map.md |
| 折中（Spec Kit 社区讨论 #815） | PRD 作为独立的前置产物，影响但不替代 feature spec | 用户明确想要"overarching PRD"；社区建议 PRD 要短、面向结构和方法而非商业套话，存进版本库 | https://github.com/github/spec-kit/discussions/815 |
| 分工论 | PRD 回答"该不该做"（给人看），spec 回答"做成什么样"（给 AI 执行） | "PRD 是对齐人的思考工具，spec 是 AI 的执行件；单人开发通常需要更多 spec、更少 PRD" | https://tekk.coach/spec-driven-development/are-specs-better-than-pr-ds/ 、https://paelladoc.com/blog/spec-vs-prd/ |

值得注意的反方声音：Spec Kit 讨论区 #1784 有人抱怨"SpecKit 制造了工作的幻觉，生成一大堆文字"——重量即成本，小改动走全套流程是过度工程。
来源：https://github.com/github/spec-kit/discussions/1784

---

## 三、被多人验证过的"最薄阶段规划"做法

有三种做法有多个独立来源背书，按厚度从薄到厚：

1. **一份长期存活的 markdown 计划文件（最薄）。** Boris Tane 的做法：不用工具内置 plan mode，自己在仓库里维护一份计划文档，含任务拆解、文件路径、实现细节，迭代 1–6 轮才允许写代码；实现时按计划逐项勾掉。摘要原文："the plan is a real artifact you edit and persist in the project"。
   来源：https://boristane.com/blog/how-i-use-claude-code/

2. **里程碑 → 竖切薄片两级拆解（中薄，GSD 的核心）。** GSD（Get Shit Done，Claude Code 上很火的框架）把项目拆成 milestone（如"核心配方平台"），每个里程碑拆成可演示的竖切薄片（slice），slice 再拆成单个上下文窗口能完成的任务；每个里程碑有审计和完成关卡。竖切薄片本身也有独立验证：groundwork、Addy Osmani 的 agent-skills 都把它写成 skill——"每片端到端、可独立测试、可独立回滚"。
   来源：https://getshitdone.help/user-guide/developing-with-gsd/ 、https://github.com/etr/groundwork/blob/main/skills/vertical-slice/SKILL.md 、https://github.com/addyosmani/agent-skills/blob/main/skills/incremental-implementation/SKILL.md

3. **任务以"agent 能一次做完"为粒度单位。** taim.io 的实操规则：拆成 1–3 个文件、一个行为改动、约 100–150 行的小块，每块有四段式 spec。
   来源：https://www.taim.io/agentic-coding/planning-a-task-the-agent-can-actually-finish

关于"决策地图"（decision map）：除 Matt Pocock 自家的 wayfinder 外，未在社区找到多人验证过的同类公开实践。**未确认。**

---

## 四、BMAD、Spec Kit、GSD 怎么处理"平台 → 阶段 → 功能"层级

| 方法 | 层级 | 平台级产物 | 阶段级产物 | 功能级产物 | 来源 |
|---|---|---|---|---|---|
| BMAD | 四层：Analysis → Planning → Solutioning → Implementation | brief.md + prd.md（FR/NFR 编号） | architecture.md（ADR）+ epic 拆分 | story（约一个 dev-day，声明独占文件范围，可并行成 wave） | https://deepwiki.com/bmadcode/BMAD-METHOD/6.2-four-phase-development-lifecycle 、https://aj-geddes.github.io/claude-code-bmad-skills/examples/ |
| Spec Kit | 两层：项目宪法 + 每功能 | constitution（原则，不是路线图） | **没有内置阶段层**——这正是讨论 #815 里用户在要的东西 | 每个 feature 一份 spec → plan → tasks | https://github.com/github/spec-kit/discussions/815 、https://konabos.com/blog/choosing-an-ai-development-framework-a-practical-guide-for-individuals-teams-and-organizations |
| GSD | 三层：milestone → slice → task | PROJECT/REQUIREMENTS（new-project 时产出） | 每里程碑 ROADMAP + CONTEXT + RESEARCH，完成前过审计关卡 | slice 内的 task，单上下文窗口可完成 | https://getshitdone.help/user-guide/developing-with-gsd/ 、https://github.com/gsd-build/get-shit-done/blob/main/docs/USER-GUIDE.md |

横向对比的几点共识（多个独立对比文互相印证）：

- BMAD 最重，适合真正的绿地大项目；对单人日常维护"仪式感过重"，不适合小改动。来源：https://arceapps.com/blog/sdd-frameworks-analysis-spec-kit-openspec-bmad/ 、https://tekk.coach/spec-driven-development/lightweight-sdd-frameworks-compared/
- Spec Kit 最轻、生态最大，但"阶段门禁偏硬"，且没有产品到阶段的拆解层。来源：https://konabos.com/blog/choosing-an-ai-development-framework-a-practical-guide-for-individuals-teams-and-organizations
- 三家的功能级产物（spec/story/task）高度同构；差异几乎全在"功能之上还有几层、每层多重"。

---

## 五、对本决策的三条启示

1. **平台层该有，但要薄。** 翻车实录反复指向同一根因：只有功能级开发链，没有"整个项目拆成哪几个阶段"的那张纸。但成功案例（Boris Tane、25 万行复盘、GSD 用户）用的都是"一份路线图/里程碑清单"，没有人靠 BMAD 全套重量级产物活下来。结论：加一层，厚度对齐 GSD 的 milestone 级，不对齐 BMAD 的 PRD+architecture+epic 三级。

2. **产品级和功能级文档不是二选一，是两层各管一件事。** 社区共识：功能级 spec 是喂给 agent 的执行件；产品级产物只负责"为什么做、边界在哪、分几个里程碑"，一两页就够（Spec Kit #815 的建议：短、面向结构、进版本库）。这与本仓库 D10"开发文档分类"的既有方向兼容。

3. **新层可以直接插在现有开发链的入口上。** GSD 的模式可照搬：里程碑 ROADMAP 里的每个 slice 就是一次"进入开发链"的门票——进链之后照走 to-spec/to-tickets/implement，平台层不需要发明新的功能级流程。决策地图（wayfinder 式）做法无社区验证，若要用属于自建实验，风险自担。
