# GitHub 协作规则

GitHub 协作规则由 `github-project-flow` skill 执行，本文档只描述稳定边界。

## 默认策略

- 微小维护：Issue 推荐，不强制。
- 普通变更：默认需要 Issue + PR。
- 架构变更：必须需要 Issue + PR，并视情况补充 ADR、C4 或领域文档。
- 默认分支使用 `main`。
- 不直接在默认分支提交普通变更或架构变更，除非用户明确要求。

## 语言

- Issue 标题和正文默认使用简体中文。
- PR 标题和正文默认使用简体中文。
- Review comment 和 issue comment 默认使用简体中文。
- 英文代码标识符、文件名、命令、字段名、错误码和协议名保留英文。

## 标签

标签真值维护在 `skills/github-project-flow/SKILL.md`。

不要在项目文档里维护另一套标签表，避免漂移。

## 失败处理

如果 GitHub 操作失败：

- 认证失败：要求用户登录或修复 `gh auth`。
- remote 缺失：要求用户提供 owner/repo 或先设置 remote。
- 权限不足：停止并报告需要的权限。
- 发现重复 Issue/PR：更新已有对象，不创建重复对象。

不要在失败后反复尝试不同创建方式。
