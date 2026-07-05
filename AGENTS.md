# Agent 规范

本仓库维护 lushangkan 的个人 Agent workflow 资产。所有产出物默认使用简体中文。

## 工作边界

- 本仓库用于维护 workflow、skills、OpenSpec schema、Agent 规范和安装模板。
- 不在本仓库实现具体业务项目功能。
- 新项目初始化规则应写入 `skills/project-init-workflow/SKILL.md` 或模板文件。
- 项目长期上下文文档规则应写入 `skills/project-context-docs/SKILL.md`。
- GitHub Issue / PR 协作规则应写入 `skills/github-project-flow/SKILL.md`。
- OpenSpec 切片流程应写入 `schemas/ai-solo-workflow/`。

## 中文写作

- README、docs、skills、OpenSpec artifact、Issue、PR、review comment 默认使用简体中文。
- 代码 comment 默认使用简体中文。
- API、REST、provider、OpenSpec、webhook、job、status、字段名、错误码、库名、框架名、协议名和其他固定技术术语保留英文。
- PR title、PR body、review comment 和 issue comment 使用自然中文表达，不使用英文模板腔。

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
