## 执行模式

<!-- 多子代理并行实现 | 不适用 -->

## 全局约束

<!-- 从 proposal/spec/design 复制硬约束，不重新解释。 -->

## Memory Authority

- Authority Worktree: <!-- 主 Agent 所在 worktree -->
- Authority Change Root: <!-- 权威 <changeRoot> -->
- 主 Agent: <!-- 负责维护 ledger/handoff 的 Agent -->
- 合并规则: <!-- 其他 worktree 或子代理只写独立 report，由主 Agent 汇总 -->

## Work Unit 列表

| Work Unit | 所属垂直切片 | 推荐代理 | 写入范围 | 读取范围 | 文件所有权 | Review Owner | Blocked By | Blocks | 可并行 | 验收输出 |
|---|---|---|---|---|---|---|---|---|---|---|

## 子代理交付契约

每个子代理必须返回：

- **Status:** `DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT`
- 变更摘要
- 测试摘要
- TDD 证据
- 修改文件
- 报告文件路径

报告路径命名规则：`<changeRoot>/.memory/apply/agent-reports/<work-unit-id>.md`。

Review 报告路径命名规则：`<changeRoot>/.memory/apply/agent-reports/<work-unit-id>-review.md`。

子代理只写自己的 report；handoff 接收方是维护 Memory Authority 的主 Agent。

## Review 规则

每个 Work Unit 完成后，必须先做 **Spec Compliance Review**，再做 **Code Quality Review**。

Critical / Important 问题必须修复并 re-review。

Minor 问题记录到 ledger，可在最终 review 中统一处理。

## Progress Ledger

执行时必须维护 `<changeRoot>/.memory/ledger.md`。

每个 Work Unit 完成并通过 review 后，记录：

- Work Unit ID
- 完成状态
- 修改范围
- 测试结果
- review 结论
- blocker 或 concern
