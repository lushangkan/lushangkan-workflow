# Agent 规范

本仓库维护 lushangkan 的个人 Agent workflow 资产:skills 和相关规范。不在本仓库实现具体业务功能。

## 输出规范

面向用户的输出(会话回复、README、docs、Issue、PR、review comment)必须遵守 `skills/output-style/SKILL.md`。这是无障碍要求,不是风格偏好。核心 10 条:

1. 默认简体中文;API、字段名、错误码、库名、框架名、协议名等固定技术术语保留英文原词。
2. 一次只让读者同时持有 3 项以内的新概念。
3. 判断或答案放第一行。
4. 信息写在用它的那一刻,不写"如前所述"。
5. 每轮开头给位置:刚做完什么、现在在哪、下一步做什么。
6. 待决策事项 2 个以内,每个带推荐值。
7. 用日常词;新概念首次出现时用一句人话定义。
8. 一句一意,主谓宾直陈。
9. 判断直述,不写"我认为""换句话说""需要注意的是"。
10. 映射关系用表格;代码块只装能运行的代码或命令。

代码 comment 默认使用简体中文。给子代理的指令和内部记录不受本节约束,写给模型读的文档用 `writing-for-agents`。

## Skill 入口

| skill | 何时用 |
|---|---|
| `output-style` | 写任何给人看的中文内容前 |
| `grill-with-docs` → `to-spec` → `to-tickets` → `implement` → `code-review` | 开发链,fork 版,依次调用 |
| `kiss-review` | 每张薄片票的轻筛:票边界、多余抽象、原子提交 |
| `plan-understanding-quiz` | 计划批准后、动手前,考用户确认理解 |
| `behavior-validator` | implement 交付前的黑盒验收 |
| `wayfinder` | 巨大模糊问题画地图;只在多 session 工作时用 |
| `governor` | 新项目定档、防过度工程;与 `tdd` 冲突时 `tdd` 赢 |
| `scheduler` | 定时提醒、定时本地任务(软提醒用它的 Linux 做法) |

fork 在 `~/Projects/skills`(上游 mattpocock/skills),通过符号链接激活:`~/.agents/skills/<名>` 指向 fork 内目录。改 fork 即生效;拉上游更新后检查本地改动是否冲突。

## 票在哪(Multica)

里程碑 = Multica **Project**;票 = Multica **Issue**(SOUL-N);GitHub 上没有票,只有代码、PR、CI、合并。

- 找票:`multica --profile desktop-api.multica.ai issue list`
- 开工领票:`scripts/workflow-next`
- 交付契约:分支名带 `soul-<n>`,PR body 写 `Closes SOUL-N`;合并后 Multica 自动把票设 done。

## Git 与 GitHub

- 默认分支 `main`。
- commit message 用 `fix:`、`feat:`、`docs:` 等 Conventional Commits 前缀。

## 修改规则

- 根 AGENTS.md 只放高优先级规则和 skill 入口。稳定细节放进对应 skill。
- 改 skill 后,检查本文件的入口表是否仍然一致。
- 自己能做完的活当场做完,不把能代劳的步骤列成清单推回给用户跑。
