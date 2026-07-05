---
name: github-project-flow
description: Use when an OpenSpec change needs GitHub Issues, labels, PR creation, PR body updates, or syncing proposal/spec/design/tasks into GitHub tracking. Keeps Issue/PR workflow, Chinese labels, and artifact-derived templates out of the main OpenSpec prompt.
---

# GitHub Project Flow

Use this skill whenever an `ai-solo-workflow` change needs GitHub Issue or PR work.

The goal is to make GitHub tracking predictable: fixed labels, stable templates, no repeated trial-and-error with `gh`, and no ad hoc Issue/PR bodies that drift from OpenSpec artifacts.

## Scope

This skill handles:

- GitHub label vocabulary for Issues and PRs.
- Creating or updating the tracking Issue for an OpenSpec change.
- Creating or updating the implementation PR.
- Building Issue and PR bodies from `proposal.md`, `proposal-review.md`, specs, `design.md`, `implementation-mode.md`, `tasks.md`, black-box acceptance, and verification results.
- Linking Issue, PR, OpenSpec change, ADR, C4, and relevant docs.

This skill does **not** design the feature, write specs, implement code, or decide architecture.

## Preconditions

Before creating or updating GitHub objects:

1. Resolve the OpenSpec change with `openspec status --change "<name>" --json`.
2. Use the returned `changeRoot`, `artifactPaths`, and `planningHome`; do not guess paths.
3. Read only the artifact files needed for the current operation.
4. If GitHub repository owner/name is unclear, derive it from `git remote -v`; if still unclear, ask the user once.
5. Never create duplicate Issues or PRs. Search existing Issues/PRs by title, branch, and OpenSpec change name first.

## Label Vocabulary

Labels are Chinese and intentionally small.

Create missing labels before applying them. If the repository already has similar labels, prefer reusing exact existing labels only when the meaning matches.

| Label | Color | Use |
|---|---|---|
| `类型：微小维护` | `d0d7de` | 文档错字、配置小修、无行为变化 |
| `类型：普通变更` | `1f883d` | 新功能、小修复、已有行为调整 |
| `类型：架构变更` | `8250df` | 技术栈、数据模型、外部依赖、安全、部署、核心流程 |
| `状态：提案中` | `fbca04` | Scope/Behavior/Solution/Execution Gate 尚未完成 |
| `状态：待实现` | `0e8a16` | artifacts 完成，等待 apply |
| `状态：实现中` | `0969da` | apply 正在进行 |
| `状态：待验收` | `bf3989` | 实现完成，等待黑盒验收或 PR review |
| `状态：阻塞` | `cf222e` | 需要用户决策、artifact 修正或外部依赖 |
| `范围：垂直切片` | `54aeef` | Issue/PR 聚焦一个端到端可验证切片 |
| `需要：C4` | `5319e7` | 需要或已经产出 C4 文档 |
| `需要：ADR` | `b083f0` | 需要或已经产出 ADR |
| `需要：领域建模` | `c297ff` | 需要或已经更新 CONTEXT.md / 领域术语 |
| `执行：AI主导` | `6f42c1` | AI 主导，人工辅助 |
| `执行：多代理` | `a371f7` | 多子代理并行实现 |

Use one `类型：...` label exactly.

Use one current `状态：...` label exactly; remove stale status labels when changing state.

PRs should receive the same type/execution/need labels as the linked Issue, plus the current status label.

## Issue Creation Timing

For `ai-solo-workflow`:

- 微小维护：Issue 推荐，不强制。
- 普通变更：Issue 必须在 Scope Gate 后存在。
- 架构变更：Issue 必须在 Scope Gate 后存在，并在 Solution Gate 后更新设计摘要。

If a user explicitly asks not to create an Issue, record that exception in `proposal.md` and do not create one.

## Issue Title

Use this format:

```txt
[<变更等级>] <一个端到端结果>
```

Examples:

```txt
[普通变更] 用户可以提交论文并看到提交结果
[架构变更] 论文处理流程采用可审计的异步任务架构
```

The title must describe the vertical slice outcome, not a layer such as “实现后端 API”.

## Issue Body Template

Build the body from OpenSpec artifacts.

```md
## OpenSpec

- Change: `<change-name>`
- Change Root: `<changeRoot>`
- Workflow: `ai-solo-workflow`

## 问题范围

<from proposal.md: 问题范围>

## 用户意图

<from proposal.md: 用户意图>

## 垂直切片

<from proposal.md: 垂直切片>

## 行为规格

<summarize specs: requirements and scenario titles only>

## 非目标

<from proposal.md: 非目标>

## 流程触发

- C4 建模：<from proposal.md / design.md>
- 领域建模：<from proposal.md / design.md>
- ADR：<from proposal.md / design.md>

## 关联文档

- Proposal: `<path>`
- Proposal Review: `<path>`
- Specs: `<paths>`
- Design: `<path or pending>`
- Tasks: `<path or pending>`
- C4: `<paths or none>`
- ADR: `<paths or none>`

## 当前 Gate

- [ ] A. Scope Gate
- [ ] B. Behavior Gate
- [ ] C. Solution Gate
- [ ] D. Execution Gate
- [ ] E. Acceptance Gate
```

Keep the Issue body concise. Link artifact paths instead of pasting full files.

## Issue Update Rules

Update the existing Issue when:

- proposal-review changes the scope or split decision.
- specs add or rename scenario titles.
- design selects a technical direction or approves new libraries.
- tasks become ready for apply.
- implementation is blocked.
- black-box acceptance passes or fails.

Do not close the Issue until the PR is merged or the user explicitly closes the change as not planned.

## PR Creation Timing

Create the PR when there is an implementation branch with meaningful changes.

For AI-led or multi-agent modes, create the PR no later than after the first completed vertical slice, unless the user prefers a single final PR.

Never create a PR from `main` or `master` without explicit user consent.

## PR Title

Use the same outcome language as the Issue:

```txt
<one vertical slice outcome>
```

If the PR closes the Issue, include `Closes #<issue>` in the PR body, not necessarily the title.

## PR Body Template

```md
## Summary

<1-3 bullets describing what changed, derived from proposal/design/tasks>

## OpenSpec

- Change: `<change-name>`
- Proposal: `<path>`
- Specs: `<paths>`
- Design: `<path>`
- Tasks: `<path>`

## Linked Issue

Closes #<issue-number>

## Behavior Covered

<scenario titles from specs, not full spec text>

## Design Notes

<selected design direction, new libraries approved, C4/ADR/domain references>

## Execution Mode

<from implementation-mode.md>

## Verification

- Tests: `<commands and result summary>`
- Black-box acceptance: `<passed/failed/path>`
- Oracle reviews: `<proposal/design/tasks/code review summaries>`

## Risk

<remaining risk or “无已知阻塞风险”>
```

Update the PR body when verification, black-box acceptance, ADR, or C4 links change.

## Command Discipline

Prefer `gh` when available.

Recommended sequence:

1. Inspect repository and auth:
   - `git remote -v`
   - `gh auth status`
   - `gh repo view --json owner,name,url`
2. Ensure labels exist before applying them.
3. Search before create:
   - Issues: title and OpenSpec change name.
   - PRs: current branch and linked Issue.
4. Create or update exactly one Issue per OpenSpec change unless the proposal was explicitly split.
5. Create or update exactly one PR per implementation branch.

Do not keep retrying different GitHub approaches. If `gh` fails because auth, repo, or permissions are missing, stop and report the exact blocker.

## Failure Handling

If GitHub operations fail:

- Auth missing: ask user to authenticate `gh`.
- Repo remote missing: ask user to provide owner/repo or initialize remote.
- Labels unavailable due permissions: continue without label changes only after telling the user.
- Duplicate Issue/PR detected: update the existing one instead of creating a new one.

Record unresolved GitHub blockers in the OpenSpec artifact that triggered the operation.
