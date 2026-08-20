# lushangkan-workflow

个人 AI 工作流的家：规范、设计文档、脚本。Skill 本体在 fork 仓库。

## 两个仓库的分工

| 仓库 | 装什么 |
|---|---|
| 本仓库 | AGENTS.md 规范、`docs/`（workflow v2 总稿 + 调研档案）、`scripts/workflow-next`、`skills/output-style` |
| [lushangkan/skills](https://github.com/lushangkan/skills)（fork 自 mattpocock/skills） | 全部 skill 本体：Matt 开发链 + 个人层改造 |

Skill 激活：fork 里的 `scripts/link-skills.sh` 把 skill 符号链接进 `~/.agents/skills/` 和 `~/.claude/skills/`，omp 和 Claude Code 同时生效。

## 票在哪

Multica（workspace Soulhub）：里程碑 = Project，票 = Issue（SOUL-N），GitHub 只管代码和 PR。每天开工跑 `scripts/workflow-next`。

## 写给人看的内容

一律遵守 `skills/output-style/SKILL.md`。这是无障碍要求，不是风格偏好。
