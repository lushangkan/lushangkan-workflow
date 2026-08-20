# Workflow v2 总稿

> 状态：设计已定稿（2026-08-20）。下一步：逐项实施（clone fork、装引用项、改 skill）。
> 新会话接手读这份 + `docs/research/` 即可，不用重翻历史。

## 1. 底座与原则

- **fork**：`lushangkan/skills`（fork 自 mattpocock/skills，已建）。包管理器 lock 指向 fork；上游更新 = 显式 merge。
- **改法**：所有改动直接做在 fork 的 Matt skill 文件里。不建并列的第二套流程（双源 = 税，否决过）。
- **够用的工具**：每个环节必须能回答"没有它冲刺会慢吗"，答不上就砍。
- **无障碍**：output-style 是全流程输出纪律（阅读障碍 + ADHD 适配，非偏好）。
- **harness 中立**：编排层用 Multica（daemon 探测 20+ agent CLI，omp/PI/Claude Code 官方支持）；票和状态以 Multica 为唯一真相源；harness 差异收进 Agent 配置。

## 2. 开发链与里程碑（Multica 版）

开发链（Matt 原样）：`grill-with-docs → to-spec → to-tickets → implement → code-review 双轴 → PR（Closes MUL-N）→ 人工合并`。

**权威分工**：Multica 管 Project（里程碑）、Issue（薄片票）、Task（运行记录）；GitHub 只管代码、PR、CI、人工 review、merge；本机 daemon + omp/PI/Claude Code 实际执行。

**里程碑映射**（改 fork 里的 to-spec / to-tickets，规则写进 AGENTS.md）：

| 动作 | 做法 |
|---|---|
| 愿景 | 写进当前 Project 的说明，不单建文件；Project 说明会自动进该项目下 agent 的执行上下文 |
| 建里程碑 | 一次冲刺/交付目标 = 一个 Multica Project；当前 Project 展开成薄片 Issue，后续 Project 只留标题 + 一句目标（滚动式） |
| 薄片票 | Project 内建 Issue（MUL-N）；批次顺序用父子 + stage（无原生依赖图，不造第三套概念） |
| 每天开工 | 一条薄包装命令 `workflow next`：list 当前 Project 的 todo 票 → 取未指派、优先级最高的一张 → assign 给预配置 Agent（Multica 无原子 claim-next，包装 = 两次 CLI 调用） |
| 里程碑完成 | Project 进度自动算（done+cancelled ÷ 总数）；清空后拆下一个 Project |
| 方向变更 | 三步：①改当前 Project 说明，追加一行变更记录 ②未开工的过期票设 cancelled ③只重拆当前 Project，后续不动 |

**概念澄清写死**（AGENTS.md + 相关 skill 开头）：里程碑 = Multica **Project**；票 = Multica **Issue**（MUL-N）；GitHub 上没有票，只有代码和 PR。找票用 `multica issue` 命令，不要去 GitHub issue 里找。（用户实测：模型默认以为里程碑在 GitHub issue 里，会找错地方。）

**Agent 配置**：建几个稳定 Agent（如 OMP Implementer、PI Researcher、Claude Reviewer），各自固定 runtime + 模型 + skills；派票 = 指派给 Agent，不临时选 CLI 和模型。

**交付契约**：Agent 在每 Task 独立的 git worktree 干活，完成用本机 `gh` push + `gh pr create`；PR body 写 `Closes MUL-N`、分支名带 `mul-n`；人 review 合并后 Multica 自动把票设 done，Project 进度自己走。

**冲刺第一天**：grill-with-docs 盘问平台本身——全仓库没有一个字写平台是什么，不盘问 agent 只能瞎编。

## 3. 个人层处置（调研结论：6 项全有现成实现）

| 项 | 处置 | 来源 |
|---|---|---|
| 输出无障碍 | 已有 output-style（2026-08-20 融合 attention-control：新增 B8 禁语清单、D8 收尾动作，A4 加查证命令）；fork 里相关 skill 和 AGENTS.md 引用它 | 本仓库 |
| 计划/文字网页批注 | 已装好，接线到 to-spec / 计划交付后当 review 门 | plannotator（8-16 已装） |
| 黑盒验收 | **引用 + 外层编排**：behavior-validator 的协议比你旧版更完整（行为契约模板、反作弊探测、逐条报告）；但外层必须补三条你旧版的硬保证——每次派**全新零上下文 agent**、契约从原始需求**冻结**（不许验收者自己改写）、**非微小变更强制触发** | openclaw/agent-skills，MIT |
| HTML 报告 | **引用** effective-html（plannotator 官方，skills.sh 收录，`npx skills add plannotator/effective-html`）；分工：output-style 管内容（短句、结论先行），effective-html 管呈现；调用时固定要求低装饰、正文优先、高可读字体、关键信息不许藏进 hover | plannotator/effective-html，MIT |
| ADR + 词汇表 + C4/mermaid | **组合+改造**：格式用 adr-scribe（MADR）+ c4designer + Matt 的词汇表；触发方式改成**无感知自动**——你一做架构决策它就当场写入，不用你开口；每份 ADR 必须含"考虑过哪些方案、为什么排除" | muthub-ai/c4-skills，MIT |
| 反向拷问 | **移植** plan-understanding-quiz（计划后两道题考用户，答错先解释再决定去留），封装成小 skill（UditAkhourii/adhd 查过：无此方案） | PolyArch/humanize |
| 反完美主义 | **引用** governor（投入分档、决策点给默认值、尊重用户否决）+ 已耗时间先做软提醒（Linux 计时 hook 无现成；adhd 仓库的"按风险定预算、强制收敛 shortlist"两条机制并入设计备注） | cwinvestments/memstack，MIT |
| 多 agent 编排 + issue 管理 | **Multica 原生**（2026-08-20 用户拍板）：Project/Issue/Agent/daemon 全包，官方支持 omp/pi/claude；milestone-workflow 移植取消 | multica-ai/multica，v0.4.31 pre-1.0 |
| 结对质疑 | AGENTS.md 规则，不是 skill | — |

**一个保留意见**：用户说黑盒验收可替代 Matt 的 tdd。两者阶段不同——tdd 是写代码时的纪律（先写测试），黑盒验收是交付时的门。我的推荐：都留，implement 内驱 tdd + 交付前 behavior-validator 黑盒门。用户拍板替代也行，implement 就不再先写测试。

### 社区验证后追加的规则（Reddit r/ADHD_Programmers，2026-08-20 调研）

| 规则 | 落点 |
|---|---|
| 回场通知契约：每条通知只答四件事——哪张票完成或阻塞、agent 做了什么、验证结果在哪、你现在只需做哪一个动作 | 异步基建设计 + AGENTS.md |
| review 带宽限流：一张小票 = 一个 worktree = 一个小 PR = 一个可观察行为；待 review 到上限就停领新票（并行度由 review 能力定，不由 agent 数定） | AGENTS.md + 领票命令处 |
| dopamine 分区：人保留架构、关键取舍、有趣的难题、最终 review；agent 接机械活（重复代码、查资料、测试/文档初稿、小实现） | AGENTS.md |
| 票边界：每张票写明允许改哪些行为、预期 diff 大小、禁止新增哪些抽象；票外发现只记 backlog，不顺手改 | fork 的 to-tickets |

依据：`docs/research/2026-08-20-reddit-adhd.md`。高热证据（516 赞"AI 夺走解题乐趣"、2,895 赞 i-have-adhd 帖）支持这些方向。

## 4. 冲刺计划与清理

- **每日决策成本 = 1**：这张票现在做不做。开工一条命令；implement 内驱纪律；收工 PR 写 Closes MUL-N，Project 进度自己走。
- **异步基建 = Multica**（Cloud 协调面 + 本机 daemon）：自建 cron+PI+sandbox 方案取消。两条安全注意：worktree 是并发隔离不是沙箱，daemon 用专用 OS 用户或容器跑；Cloud 的数据区域和保留周期未公开，敏感资料放入前找官方确认。冲刺期间固定一个已验证版本，不追它的日更。
- **清理**：删 `Things.md`、`docs/workflow-skills.md`（过期，误导 agent）。
- **调研档案**：`docs/research/` 共 14 份。
