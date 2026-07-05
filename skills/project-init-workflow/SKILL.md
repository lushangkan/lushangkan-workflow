---
name: project-init-workflow
description: Project init workflow for starting a new repo or retrofitting an existing repo with lushangkan's Agent rules, Chinese output policy, Git/GitHub defaults, and OpenSpec assets.
---

# Project Init Workflow

Run a tight project init loop: **probe -> decide -> draft -> write -> verify**.

Default to the lightest useful setup. Project context documents are managed by `project-context-docs`; initialization should install or point to that skill, not create empty docs upfront.

For exact questions, initialization files, and reusable blocks, load [reference.md](reference.md) when you reach the matching step.

## 1. Probe

Inspect before asking or writing:

- Git: `git status --short`, current branch, remotes.
- Agent files: `AGENTS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`, `.cursor/rules/`.
- Project docs: `README.md`, `PROJECT.md`, `docs/`, `openspec/`.
- Skills: `.opencode/skills/`.

Completion criterion: you can state what already exists, what would be newly created, and which existing files might be edited.

## 2. Decide

Ask only for choices that affect files you will write now.

Use [reference.md](reference.md#decision-questions) for the decision questions:

- Language policy.
- Git and GitHub policy.
- OpenSpec usage.

Do not ask about future documentation structure unless it affects files you will write now.

Completion criterion: every file to create or edit has a known policy source; unresolved choices are either answered or marked as blockers.

## 3. Draft

Present the intended file list and the root `AGENTS.md` workflow block before writing.

Use [reference.md](reference.md#初始化文件) to keep the initial file list minimal. Use [reference.md](reference.md#agents-区块) for the reusable block.

Completion criterion: the user has approved the draft, or the user explicitly requested non-interactive setup.

## 4. Write

Write only the approved setup.

Rules:

- Do not overwrite existing files silently.
- Update an existing `AGENTS.md` block in place instead of appending duplicates.
- Keep root `AGENTS.md` short; put stable detail in `docs/agents/` or a skill.
- Do not create empty project docs upfront; use `project-context-docs` when the current task needs `PROJECT.md`, `CONTEXT.md`, ADRs, or `docs/agents/*.md` persisted.
- If initializing Git, use non-interactive commands and rename the default branch to `main` when chosen.
- If using OpenSpec, install `ai-solo-workflow` and keep project-level PRD or roadmap outside OpenSpec.

Completion criterion: every approved file exists or has been intentionally skipped with a stated reason.

## 5. Verify

Run the smallest checks that prove the setup works:

- `bunx skills list --json` when project skills were installed through `bunx skills`.
- `openspec schema validate ai-solo-workflow` when OpenSpec assets were installed.
- `openspec doctor` when an OpenSpec root was created or changed.
- `git status --short` to report resulting changes.

Completion criterion: verification commands passed, or each failure has a concrete blocker and next action.

## Hard Stops

Ask before:

- Creating broad project docs that are not needed by the current task.
- Broadly rewriting an existing `AGENTS.md`.
- Initializing Git in a non-empty directory.
- Creating GitHub labels, Issues, or PRs remotely.
