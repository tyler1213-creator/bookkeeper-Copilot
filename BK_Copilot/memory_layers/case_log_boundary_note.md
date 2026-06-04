# Case Log Boundary Note

## 文件状态

- Status: superseded boundary note
- Last reviewed: 2026-05-11
- Scope: 记录已接受的 `Case Log`（案例日志）边界定义，作为正式 M1-M2 草案的来源之一。
- Current formal draft: `BK_Copilot/memory_layers/case_log/`
- Not a full spec: 本文档不是 `Case Log`（案例日志）的完整 memory layer spec，不冻结字段、生命周期、mutation path 或 storage contract。

## 已接受定义

`Case Log`（案例日志）是以 `entity_id`（实体唯一标识）为主要索引的 completed-case learning memory（已完成案例学习记忆）。

它从已完成交易中保存可复用的 bookkeeping precedent（记账先例）、evidence condition（证据条件）、exception context（例外上下文）和 correction reference（纠正引用），用于未来 case-based judgment（基于案例判断）、rule promotion candidate（规则升级候选）、automation risk review（自动化风险审核）和 entity risk update candidate（实体风险更新候选）。

`Case Log`（案例日志）不是 transaction process log（交易处理过程日志），也不是 audit-facing final transaction record（面向审计的最终交易记录）。

## 三层边界

`Entity Log`（实体日志）是 identity authority store（身份权威存储）。

- 回答：这个对象是谁，有哪些已确认 Alias（过去已确认的 transaction surface text 到 stable entity 的对应关系）、lifecycle state（生命周期状态）和 automation policy（自动化策略）。
- 不回答：这个对象过去的交易通常如何分类，或本次交易应如何记账。

`Transaction Log`（交易日志）是 audit-facing final transaction record（面向审计的最终交易记录）。

- 回答：某一笔 `transaction_id`（交易唯一标识）最终如何处理，processing path（处理路径）、final outcome（最终结果）、review trace（审核轨迹）和 audit trail（审计轨迹）是什么。
- 不回答：未来 runtime judgment（运行时判断）应直接复用哪个历史模式。

`Case Log`（案例日志）是 entity-indexed learning memory（按实体索引的学习记忆）。

- 回答：某个 `entity_id`（实体唯一标识）相关的已完成交易中，哪些可以作为 future case precedent（未来案例先例），它们在什么 evidence condition（证据条件）和 exception context（例外上下文）下成立。
- 不回答：entity identity authority（实体身份权威）、active rule authority（生效规则权威）或 final audit record（最终审计记录）。

## 与 Transaction Log 的区别

`Transaction Log`（交易日志）保存每一笔进入系统并完成处理的交易审计记录。

`Case Log`（案例日志）只保存从已完成交易中抽取出的可学习案例。它必须能引用 `transaction_log_ref`（交易日志引用）或等价 finalization proof（完成证明），但不能替代或重写 `Transaction Log`（交易日志）。

因此：

- 所有交易完成过程和最终审计轨迹属于 `Transaction Log`（交易日志）。
- 可用于未来 case-based judgment（基于案例判断）的 precedent（先例）属于 `Case Log`（案例日志）。
- `Case Log`（案例日志）不得把完整 processing path（处理路径）复制成学习依据。

## 与 Entity Log 的关系

`Case Log`（案例日志）可以按 `entity_id`（实体唯一标识）查询，也可以为 entity-level risk（实体级风险）提供依据。

但它不能直接修改：

- `risk_flags`（风险标记）。
- `automation_policy`（自动化策略）。
- Alias。
- `entity_status`（实体生命周期状态）。

如果历史案例显示某个 entity（实体）存在 mixed-use risk（混用风险）、unstable classification pattern（不稳定分类模式）或 automation risk（自动化风险），正确路径是：

```text
Case Log evidence
-> Post-Batch Lint / Review / Governance Review candidate
-> accountant / governance approval
-> Entity Log mutation or Rule Log mutation
-> Governance Log audit trail
```

`Case Log`（案例日志）只提供依据，不执行 mutation（变更）。

## 允许的读取者

- `Case Judgment Node`（案例判断节点）：读取 relevant case precedent（相关案例先例），辅助判断当前交易是否可走 case-based judgment（基于案例判断）。
- `Post-Batch Lint Node`（批后体检节点）：读取 entity-linked case history（实体关联案例历史），发现 rule promotion candidate（规则升级候选）、automation risk（自动化风险）或 entity risk candidate（实体风险候选）。
- `Review Node`（审核节点）：读取历史案例，帮助 accountant（会计师）理解当前 review item（审核项）的历史上下文。
- `Governance Review Node`（治理审核节点）：读取案例依据，评估 rule change（规则变化）、automation policy change（自动化策略变化）或 entity risk update（实体风险更新）。
- `Knowledge Compilation Node`（知识编译节点）：读取案例历史并生成 readable summary（可读摘要），但 summary（摘要）不能替代 source authority（来源权威）。

## 禁止边界

`Case Log`（案例日志）不能：

- 成为 deterministic rule source（确定性规则来源）。
- 直接创建、升级、修改、删除或降级 active rule（生效规则）。
- 直接修改 entity authority（实体权威）。
- 替代 `Transaction Log`（交易日志）的 final audit record（最终审计记录）。
- 保存 unapproved AI reasoning（未经批准的 AI 推理）作为 future authority（未来权威）。
- 把 repeated outcome（重复结果）自动升级为 approved rule（已批准规则）。

## 后续仍需正式讨论

正式 `Case Log`（案例日志）M1-M2 设计仍需决定：

- 哪些 completed transaction（已完成交易）具备 case write eligibility（案例写入资格）。
- duplicate / corrected / reversed / split transaction（重复 / 纠正 / 冲销 / 拆分交易）如何影响 case supersession（案例替代关系）。
- `Case Log`（案例日志）与 `Transaction Log`（交易日志）的 exact trigger order（精确触发顺序）。
- 哪些 case-derived signals（案例衍生信号）可以进入 governance candidate（治理候选）。
