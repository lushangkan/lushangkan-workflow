# Agent 规范

本项目默认使用简体中文协作。

## 文档范围

- 项目级描述、产品简介、整体愿景和跨多个切片的大范围需求放在 OpenSpec 外部，例如 `README.md`、`PROJECT.md` 或 `docs/`。
- 项目长期上下文文档由 `project-context-docs` skill 按需读取和懒创建。
- OpenSpec change 只用于更小的垂直切片；每个切片都必须能独立设计、实现、验证和归档。
- 不要把整个项目描述、产品愿景或平台级路线图塞进一个 OpenSpec change。

## 输出规范

面向用户的输出（会话回复、README、PROJECT、docs、OpenSpec artifact、Issue、PR、review comment）必须遵守项目安装的 `output-style` skill。这是无障碍要求，不是风格偏好。核心 10 条：

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

代码 comment 默认使用简体中文。给子代理的指令和内部记录不受本节约束。

## Git 与 GitHub

- 默认使用 Git 管理项目。
- 默认分支使用 `main`。
- 普通变更和架构变更默认使用 GitHub Issue + PR。
- 涉及 Git 提交、rebase 或历史调查时必须使用项目安装的 `git-master` skill。
- Git commit message 必须使用 `fix:`、`feat:`、`docs:` 等 Conventional Commits 前缀。
- GitHub Issue / PR 标签、正文和状态同步规则见项目安装的 `github-project-flow` skill。

## OpenSpec

- 本项目使用 OpenSpec 管理垂直切片变更。
- 进入实现前，先确认本次工作是否是微小维护、普通变更还是架构变更。
- 普通变更和架构变更应先形成有边界的 OpenSpec change，再实现。
