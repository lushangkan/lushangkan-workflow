# Agent 规范

本项目默认使用简体中文协作。

## 文档范围

- 项目级描述、产品简介、整体愿景和跨多个切片的大范围需求放在 OpenSpec 外部，例如 `README.md`、`PROJECT.md` 或 `docs/`。
- 项目长期上下文文档由 `project-context-docs` skill 按需读取和懒创建。
- OpenSpec change 只用于更小的垂直切片；每个切片都必须能独立设计、实现、验证和归档。
- 不要把整个项目描述、产品愿景或平台级路线图塞进一个 OpenSpec change。

## 中文写作

- README、PROJECT、docs、OpenSpec artifact、代码审查意见、Issue 内容、PR 标题和 PR 描述默认使用简体中文。
- 代码 comment 默认使用简体中文。
- API、REST、provider、OpenSpec、webhook、job、status、字段名、错误码、库名、框架名、协议名和其他固定技术术语保留英文。
- PR title、PR body、review comment 和 issue comment 必须使用自然中文表达。

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
