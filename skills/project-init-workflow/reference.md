# 项目初始化参考

`project-init-workflow` 的参考资料。只加载当前步骤需要的章节。

## 决策问题

按以下顺序询问选项。除非用户明确覆盖，否则保留默认值。

### 语言策略

- 文档是否默认使用简体中文？默认：是。
- GitHub Issue 是否使用简体中文？默认：是。
- PR title/body 是否使用简体中文？默认：是。
- Review comment / issue comment 是否使用简体中文？默认：是。
- 代码 comment 是否使用简体中文？默认：是。
- Commit message 是否使用 Conventional Commits 前缀？默认：是，例如 `fix:`、`feat:`、`docs:`。
- API、字段名、错误码、协议名、库名等固定技术标识是否保留英文？默认：是。

### Git 和 GitHub 策略

- 是否使用 Git 管理这个项目？默认：是。
- 如果仓库未初始化，是否现在初始化？默认：是。
- 默认分支是否使用 `main`？默认：是。
- Git 提交、rebase 和历史调查是否使用 `git-master` skill？默认：是。
- 是否使用 GitHub Issues？默认：如果 remote 指向 GitHub，则是；否则询问。
- 是否使用 PR 流程？默认：普通变更和架构变更使用 PR。

### OpenSpec 用法

- 是否使用 OpenSpec 管理垂直切片？默认：需要持续开发的项目使用。
- 是否安装 `ai-solo-workflow`？默认：使用 OpenSpec 时安装。
- 是否安装 `github-project-flow` skill？默认：使用 GitHub Issues/PR 时安装。

## 初始化文件

只创建当前立即有用的内容：

- `AGENTS.md`
- 用户现在需要项目描述时，创建 `PROJECT.md`
- `docs/agents/github.md`
- `docs/agents/openspec.md`
- `docs/agents/agent-md.md`
- 使用 GitHub 时，创建 `.opencode/skills/github-project-flow/SKILL.md`
- 使用 Git 时，创建 `.opencode/skills/git-master/SKILL.md`
- `.opencode/skills/project-context-docs/SKILL.md`
- `.opencode/skills/project-init-workflow/SKILL.md`
- 使用 OpenSpec 时，创建 `openspec/schemas/ai-solo-workflow/`

## 项目上下文文档

项目上下文文档由 `project-context-docs` skill 管理。初始化时安装该 skill，但不要预先创建空的项目管理文档。

- 开工前需要读取项目长期上下文时，调用 `project-context-docs`。
- 任务中需要保存项目描述、领域词汇、架构决策或 Agent 长期规则时，调用 `project-context-docs`。
- 如果 `PROJECT.md`、`CONTEXT.md`、ADR 或相关 `docs/agents/*.md` 不存在且当前任务不需要它们，安静继续，不要把缺失本身当成问题。

## Agents 区块

在根目录 `AGENTS.md` 中添加或更新这个区块：

```md
## lushangkan workflow

- 默认使用简体中文产出文档、Issue、PR 和 review comment。
- 项目级描述放在 OpenSpec 外部，例如 `README.md`、`PROJECT.md` 或 `docs/`。
- 项目上下文文档由 `project-context-docs` skill 按需读取和懒创建。
- OpenSpec change 只用于可独立设计、实现、验证和归档的垂直切片。
- Git 提交、rebase 和历史调查必须使用 `git-master` skill。
- Git commit message 必须使用 `fix:`、`feat:`、`docs:` 等 Conventional Commits 前缀。
- GitHub Issue / PR 协作规则见 `docs/agents/github.md` 和 `github-project-flow` skill。
- OpenSpec 使用规则见 `docs/agents/openspec.md`。
```

## OpenSpec 安装结果

安装 OpenSpec 后，目标项目应包含：

```txt
openspec/config.yaml
openspec/specs/
openspec/changes/archive/
openspec/schemas/ai-solo-workflow/
```

`openspec/config.yaml` 应包含：

```yaml
schema: ai-solo-workflow
```
