# ai-solo-workflow

`ai-solo-workflow` 是 lushangkan 的个人 OpenSpec schema，用于 AI 辅助单人开发、GitHub Issue/PR 管理、垂直切片、真实边界 TDD、子代理执行、黑盒验收和归档。

## 位置

本目录只维护 OpenSpec schema 与模板：

```txt
schema.yaml
templates/
```

GitHub Issue / PR 流程由仓库顶层 skill 维护：

```txt
skills/github-project-flow/SKILL.md
```

新项目初始化流程由仓库顶层 skill 维护：

```txt
skills/project-init-workflow/SKILL.md
```

## 安装

在目标项目中使用 `bunx skills` 安装相关 skills：

```bash
bunx skills add lushangkan/lushangkan-workflow --full-depth --skill github-project-flow git-master project-context-docs project-init-workflow
```

本 schema 是 OpenSpec 资产，不由 `bunx skills` 直接安装。需要使用时，将本目录复制到目标项目：

```txt
openspec/schemas/ai-solo-workflow/
```

目标项目还应包含：

```txt
openspec/config.yaml
openspec/specs/
openspec/changes/archive/
```

`openspec/config.yaml` 内容：

```yaml
schema: ai-solo-workflow
```

## 写作规则

- 所有 artifact 默认使用简体中文。
- OpenSpec 或其他工具依赖的英文标题、关键字、文件名或格式必须保留。
- 面向中文母语者编写，句子自然、清楚、直接。
- 每个段落只表达一个主要意思。
- 固定格式、路径、命令或模板使用代码块。
