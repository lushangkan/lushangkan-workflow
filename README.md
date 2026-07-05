# lushangkan-workflow

这是 lushangkan 的个人 Agent 开发 workflow 源仓库，用来集中维护：

- OpenSpec schema 与模板。
- 可复用 opencode skills。
- `AGENTS.md` 规范。
- GitHub Issue / PR 协作规则。
- 新项目初始化流程。

本仓库不是某个业务项目。业务项目通过 `bunx skills` 安装这里的 skills，并按需复制模板或 OpenSpec schema。

## 安装 skills

从目标项目根目录运行：

```bash
bunx skills add lushangkan/lushangkan-workflow --full-depth --skill github-project-flow git-master project-context-docs project-init-workflow
```

需要安装全部 skills 时，也可以使用：

```bash
bunx skills add lushangkan/lushangkan-workflow --full-depth --all
```

`templates/` 和 `schemas/` 是项目初始化与 OpenSpec 资产，不由 `bunx skills` 直接安装；需要时按项目规则复制或由 `project-init-workflow` 指导落地。

## 目录

```txt
schemas/
  ai-solo-workflow/        # OpenSpec schema 源
docs/
  agents/                  # AGENTS.md、GitHub、OpenSpec 边界源文档
skills/
  git-master/              # Git 提交、rebase 与历史调查 skill
  github-project-flow/     # GitHub Issue / PR 协作 skill
  project-context-docs/    # 项目长期上下文文档读写 skill
  project-init-workflow/   # 新项目初始化 skill
templates/
  AGENTS.md                # 新项目根 AGENTS.md 模板
  docs/agents/             # 与 docs/agents 同源的新项目规则模板
```

## 全局原则

- 默认使用简体中文写作。
- GitHub Issue、PR title、PR body、review comment 默认使用简体中文。
- 代码注释默认使用简体中文；API、字段名、错误码、协议名、库名等固定技术标识保留英文。
- 项目级描述放在 OpenSpec 外部。
- 项目长期上下文文档按需读取和懒创建，由 `project-context-docs` skill 负责。
- OpenSpec change 只描述一个可独立设计、实现、验证和归档的垂直切片。
- Git 提交、rebase 和历史调查使用 `git-master` skill；commit message 使用 `fix:`、`feat:`、`docs:` 等前缀。
- GitHub Issue / PR 细节由 `github-project-flow` skill 负责，不塞进主 OpenSpec prompt。
