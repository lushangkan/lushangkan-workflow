# Workflow Skill 清单

> 这份文件只列清单：每个 skill 一行,名字 + 一句话职责 + 来源。
> 内部规则不在这里写,是下一步的事。

## 原则

- 一切皆 skill。没有 OpenSpec,没有外部流程工具。
- 骨架用 Matt Pocock skills v1.2.3,在上面加自己的东西。
- planning 走 skill:盘问 → spec → 小票 → 实现。spec 和小票都是普通 markdown。
- 分两层:编排层(打字触发,管流程),纪律层(模型自动触发,管专业方法)。

## 编排层:管流程

| skill | 职责 | 来源 |
|---|---|---|
| ask-workflow | 路由:根据任务类型告诉你从哪个 skill 开始 | 改 Matt ask-matt |
| project-init | 新建或接手项目:装规则、定目录、装 skill | 自建 |
| grill-with-docs | 盘问想法,顺手更新 CONTEXT.md 和 ADR | Matt |
| to-spec | 把聊清楚的东西写成 spec | Matt |
| to-tickets | 把 spec 拆成可独立验证的小票,标依赖 | Matt |
| implement | 按票实现,内部驱动 tdd,收尾跑 review | Matt |
| github-flow | Issue / PR 协作:标签、正文、状态同步 | 自建(保留) |
| wayfinder | 超大模糊、装不进一个 session 的活,画决策地图 | Matt |

## 纪律层:管专业方法

| skill | 职责 | 来源 |
|---|---|---|
| output-style | 中文输出无障碍规范(已落地,41 条) | 自有,保留 |
| grilling | 盘问的底层循环 | Matt |
| domain-modeling | 维护领域术语表和 ADR | Matt |
| project-context-docs | 读写 PROJECT / CONTEXT / ADR 等长期上下文 | 自有,保留 |
| tdd | red-green 循环 | Matt |
| code-review | 两轴审查:标准 + 是否合规 | Matt |
| black-box-acceptance | 黑盒验收:不读源码,从真实入口验证 | 自建(独有资产) |
| codebase-design | 深模块设计词汇 | Matt |
| diagnosing-bugs | 难 bug 诊断循环 | Matt |
| prototype | 一次性原型,回答设计问题 | Matt |
| research | 后台查资料,给带引用的报告 | Matt |
| git-master | 提交、rebase、历史调查 | 自有,保留 |
| using-git-worktrees | 隔离 worktree 准备 | 自有,保留 |
| resolving-merge-conflicts | 解合并冲突 | Matt |
| wizard | 生成 bash 向导,带人做只有人能做的手动步骤 | Matt |

## 辅助层:跨场景

| skill | 职责 | 来源 |
|---|---|---|
| handoff | 跨 session 交接 | Matt |
| writing-for-agents | 写给模型看的文档规范 | Matt |

## 砍掉

| 砍掉的东西 | 原因 |
|---|---|
| OpenSpec schema + 8 artifact 流水线 | 与 skill 职责重叠;planning 改用 spec + 小票 |
| ai-solo-apply-execution | 太重;拆成 implement + tdd + code-review + black-box-acceptance |
| proposal / proposal-review / implementation-mode / execution-plan 模板 | 随 OpenSpec 一起去掉 |

## 待定

- planning 产物(spec、小票)放哪个目录、什么格式
- black-box-acceptance 和 implement 怎么衔接
- 非编码任务(如写简历)怎么接入这套骨架
- 迁移顺序:从 _backup 搬回哪些、重写哪些
