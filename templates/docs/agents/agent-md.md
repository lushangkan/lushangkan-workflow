# AGENTS.md 规范

`AGENTS.md` 是 Agent 的项目入口，不是项目知识库。

## 职责

根 `AGENTS.md` 只放：

- 高优先级行为规则。
- 语言策略。
- Git / GitHub 基本约束。
- OpenSpec 与项目级文档的边界。
- 关键文档入口。
- 明确禁止事项。

根 `AGENTS.md` 不放：

- 长篇项目愿景。
- 完整 PRD。
- 架构长文。
- GitHub Issue / PR 模板全文。
- OpenSpec schema 细节。
- 大量子目录规则。

## 长度

根 `AGENTS.md` 应保持短小。默认目标是 60 行以内；如果规则开始变长，应拆到 `docs/agents/` 或专门 skill。

## 分层

推荐分层：

```txt
AGENTS.md                 # 短入口
docs/agents/github.md     # GitHub 协作规则
docs/agents/openspec.md   # OpenSpec 使用边界
docs/agents/<topic>.md    # 其他稳定规则
```

## 中文规则

- 默认使用简体中文。
- Issue、PR、review comment 默认使用简体中文。
- 代码 comment 默认使用简体中文。
- API、字段名、错误码、命令、文件名、库名和协议名保留英文。

## 修改规则

如果一个规则只是某个工具的操作细节，应写进 skill，而不是写进根 `AGENTS.md`。

如果一个规则是所有 Agent 都必须知道的硬约束，才写进根 `AGENTS.md`。
