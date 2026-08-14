# Agent 规范

本仓库维护 lushangkan 的个人 Agent workflow 资产。所有产出物默认使用简体中文。

## 工作边界

- 本仓库用于维护 workflow、skills、OpenSpec schema、Agent 规范和安装模板。
- 不在本仓库实现具体业务项目功能。
- 新项目初始化规则应写入 `skills/project-init-workflow/SKILL.md` 或模板文件。
- 项目长期上下文文档规则应写入 `skills/project-context-docs/SKILL.md`。
- GitHub Issue / PR 协作规则应写入 `skills/github-project-flow/SKILL.md`。
- 面向用户的输出规范应写入 `skills/output-style/SKILL.md`。
- OpenSpec 切片流程应写入 `schemas/ai-solo-workflow/`。

## 输出规范

面向用户的输出（会话回复、README、docs、Issue、PR、review comment）必须遵守 `skills/output-style/SKILL.md`。这是无障碍要求，不是风格偏好。核心 10 条：

1. 默认简体中文；API、字段名、错误码、库名、框架名、协议名等固定技术术语保留英文原词。
2. 一次只让读者同时持有 3 项以内的新概念。
3. 判断或答案放第一行。
4. 信息写在用它的那一刻，不写"如前所述"。
5. 每轮开头给位置：刚做完什么、现在在哪、下一步做什么。
6. 待决策事项 2 个以内，每个带推荐值。
7. 用日常词；新概念首次出现时用一句人话定义。
8. 一句一意，主谓宾直陈。
9. 判断直述，不写"我认为""换句话说""需要注意的是"。
10. 映射关系用表格；代码块只装能运行的代码或命令。

代码 comment 默认使用简体中文。给子代理的指令和内部记录不受本节约束，写给模型读的文档用 `writing-for-agents`。

## GitHub 协作

- GitHub Issue / PR 的创建、更新、标签和正文格式由 `skills/github-project-flow/SKILL.md` 维护。
- 普通变更和架构变更默认需要 Issue + PR。
- 微小维护可不创建 Issue，但需要在最终说明中注明。
- 默认分支使用 `main`。
- 涉及 Git 提交、rebase 或历史调查时必须使用 `skills/git-master/SKILL.md`。
- Git commit message 必须使用 `fix:`、`feat:`、`docs:` 等 Conventional Commits 前缀。

## OpenSpec 边界

- 项目级描述、产品简介、整体愿景和跨多个切片的大范围需求放在 OpenSpec 外部。
- OpenSpec change 只用于垂直切片。
- 每个切片必须能独立设计、实现、验证和归档。

## 修改规则

- 修改 schema 后，同步检查 `README.md` 和相关 skill 是否仍然一致。
- 修改 skill 后，检查对应模板和安装说明是否需要更新。
- 不要把长篇项目管理规范塞进根 `AGENTS.md`；根文件只保留高优先级规则和入口。
