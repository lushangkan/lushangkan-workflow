# Effective HTML 调研

## 1. 它是什么、谁维护

**结论：它是 Plannotator 官方维护的一组 Agent Skills；与 Plannotator 属于同一维护方，核心提交者也相同。**

- Effective HTML 不是 Markdown 转 HTML 的固定转换器，也不是模板库。它是一组指导 coding agent 生成 HTML artifact 的 skill，覆盖报告、解释页、计划、图表、wireframe 和交互 prototype。
- 仓库归 `plannotator` GitHub 组织所有。官方站直接写明 “Effective HTML is by Plannotator”。插件清单中的 owner 也是 `plannotator`。
- Plannotator 主仓库仍在个人账号 `backnotprop/plannotator` 下。两个仓库使用同一官网 `plannotator.ai`。Effective HTML 的近期主体提交者是 `@backnotprop`，也是 Plannotator 主仓库的核心提交者。因此可以判断：**是同一作者团队，不是无关的第三方项目。**

来源：
- https://github.com/plannotator/effective-html
- https://github.com/plannotator/effective-html/blob/main/README.md
- https://github.com/plannotator/effective-html/blob/main/.claude-plugin/marketplace.json
- https://www.effectivehtml.com/
- https://github.com/backnotprop/plannotator
- https://github.com/plannotator/effective-html/commits/main/?author=backnotprop

## 2. 它怎么把内容做成 HTML

**结论：它让 agent 按内容重新设计一个单文件 HTML，不套固定模板。**

工作方式如下：

1. `html` 是总入口。它先判断内容属于报告、计划、图表、diagram、wireframe 还是 prototype，再读取相应的 specialist skill。
2. 生成一个独立 `.html` 文件。必要的 CSS 和 JavaScript 都内联；文件可直接用浏览器打开，不需要 build。除非用户允许，默认也不依赖网络资源。
3. 它**没有固定 house style 或通用模板**。颜色、字体、布局和交互由当前主题、读者、用途及项目已有设计系统决定。skill 甚至明确要求不要复用上次的 palette、字体栈、卡片系统或布局。
4. 交互是可选的。筛选、展开、缩放、键盘导航和动画只有在能帮助理解时才加入；删掉不损失含义的动画应被删除。重要事实不能只藏在 hover 中。
5. 输出契约要求 semantic HTML、响应式布局、可访问对比度、可见键盘焦点、`prefers-reduced-motion`、横向溢出控制，并要求在宽、窄 viewport 中用浏览器检查成品。

针对报告，它另有明确规则：开头快速交代“这是什么、为什么重要、哪里值得注意”；随后按读者需要排列证据和细节；区分 observation、interpretation、recommendation 与 uncertainty；表格、时间线、代码和图表可以使用更宽区域。

这意味着它比 readout 一类固定渲染器更灵活，但结果质量更依赖模型是否忠实执行 skill 和输入内容是否清楚。

来源：
- https://github.com/plannotator/effective-html/blob/main/skills/html/SKILL.md
- https://github.com/plannotator/effective-html/blob/main/skills/html/references/documents-and-presentations.md
- https://github.com/plannotator/effective-html/blob/main/skills/html/references/charts-and-data.md
- https://github.com/plannotator/effective-html/blob/main/skills/design-artifact/SKILL.md
- https://www.effectivehtml.com/catalog

## 3. 安装方式、skills.sh 收录情况

**结论：已被 skills.sh 收录，可以装整套，也可以只装一个 skill。**

整套安装：

```bash
npx skills add plannotator/effective-html
```

查看列表或只装某个 skill：

```bash
npx skills add plannotator/effective-html --list
npx skills add plannotator/effective-html --skill html
npx skills add plannotator/effective-html --skill design-artifact
```

还提供 Claude Code 和 Codex plugin 安装方式。skills.sh 页面当前列出 6 个 skills：`html`、`design-artifact`、`html-wireframe`、`html-prototype`、`html-plan`、`html-diagram`；页面显示总安装量约 8.0K。复杂报告的直接入口是 `html`，它再按需组合 `design-artifact` 和报告 reference。

来源：
- https://github.com/plannotator/effective-html/blob/main/README.md
- https://www.skills.sh/plannotator/effective-html

## 4. 活跃度和许可证

**结论：项目很新，近期开发密集，但还没有长期稳定性记录；许可证宽松。**

- 仓库未归档。查询时约有 1.7K stars、132 forks。
- 近两个月可查到 49 次提交。2026-07-29 至 2026-08-03 有一轮密集开发，加入 wireframe、prototype、网站、设计 skill 和文档；同期有 16 个 PR 合并。
- 最近一批可确认的代码/文档提交在 2026-08-03。仓库页面仍有更新，但目前看更像刚发布后快速成形的项目，不能据此证明半年以上的维护稳定性。
- 贡献高度集中在 `@backnotprop`，另有少量外部贡献。单一核心维护者风险存在。
- 许可证为 MIT，Copyright 标为 `plannotator`，允许使用、修改、分发和商用。

来源：
- https://github.com/plannotator/effective-html
- https://github.com/plannotator/effective-html/commits/main/
- https://github.com/plannotator/effective-html/pulls?q=is%3Apr
- https://github.com/plannotator/effective-html/blob/main/LICENSE

## 5. 对 ADHD／阅读障碍用户看复杂报告的匹配度

**结论：基础方向很匹配，认知负担通常低于长篇 Markdown；但它不是专门的 ADHD／阅读障碍无障碍规范，不能裸用。**

有利点：

- 报告先给方向和重点，再给证据、结构和细节，适合先扫结论再深入。
- 明确要求真实层级、可读行宽、留白、对齐比较、宽表单独滚动，而不是把内容压小。
- 重要信息不能依赖 hover；动画必须有解释价值，并支持 reduced motion。
- 示例周报把关键指标、Highlights、Shipped、Velocity 和 Carryover 分区；事故报告先给 TL;DR，再给 Timeline、Root cause、Impact 和 Action items。两者都比连续文字墙更容易定位。
- 支持把复杂关系改成图、时间线或对齐表格，能降低工作记忆负担。

风险：

- skill 没有明确写入 ADHD、dyslexia 或 WCAG 的专项验收标准。
- `design-artifact` 鼓励为主题选择两种以上字体和鲜明视觉方向。若模型追求“editorial”效果，可能使用不利于阅读障碍的展示字体、过多视觉装饰或密度过高的 dashboard。
- 每次都按主题自由设计，排版质量会波动；它不像固定模板那样天然一致。
- HTML 只能改善信息呈现，不能自动修复原文中的长句、术语堆叠、层级过深和结论滞后。

建议把它放在 `output-style` 之后：先由 `output-style` 控制短句、结论先行、列表长度和术语；再由 Effective HTML 负责视觉层级、表格、图和交互。调用时还应明确要求“workmanlike register、低装饰、正文优先、禁止把关键信息藏进交互、正文使用高可读字体”。这样可以保留 HTML 的扫读优势，同时限制它的创意自由度。

示例来源：
- https://thariqs.github.io/html-effectiveness/11-status-report.html
- https://thariqs.github.io/html-effectiveness/12-incident-report.html
- https://github.com/plannotator/effective-html/blob/main/skills/html/references/documents-and-presentations.md
- https://github.com/plannotator/effective-html/blob/main/skills/design-artifact/SKILL.md

**最终判断：能作为「复杂报告渲染成 HTML」的默认 skill，但必须让 `output-style` 作为上位内容约束，并默认采用低装饰、正文优先的报告模式。**