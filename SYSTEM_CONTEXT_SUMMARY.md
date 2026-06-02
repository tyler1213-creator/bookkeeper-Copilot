# 系统上下文速读

## 文件职责

`SYSTEM_CONTEXT_SUMMARY.md` 给新窗口快速理解当前 AI Bookkeeper 新旧系统差异。

它只保存已读材料后的高密度理解：

- 新系统交易主流程
- 旧系统交易主流程
- 两者关键差异
- 旧系统仍值得保留的产品约束
- 后续审计时应如何使用这些信息

它不证明当前新系统正确，不把旧系统当 baseline，也不替代 `new system_副本/` 或 `old_system_nodedesign/` 的源材料。

## 核心产品目标

系统要帮助会计师把客户历史账务和过往判断沉淀成可复用记忆，让新交易自动获得有证据支持的记账建议，并通过会计师纠正持续学习，在不牺牲审计性和控制权的前提下逐步提高自动化率。

## 新系统一句话

新系统不是先把交易压成 `description / pattern`，而是先保留可追溯 evidence，再认 `entity`，再查 `case memory`，最后判断能否通过 approved `rule` 或 case-based judgment 自动处理。

核心链条：

```text
raw evidence
-> transaction identity
-> profile structural check
-> entity resolution
-> rule match
-> case judgment / pending / review
-> JE generation
-> transaction logging
-> case memory / governance update
```

## 新系统主流程

1. `Onboarding Node` 处理客户历史材料，建立初始 evidence、profile 候选、entity 候选、case memory、rule 候选和 knowledge basis。它是客户初始化 / 历史学习入口，不是普通新交易 runtime 中间节点。
2. `Evidence Intake / Preprocessing Node` 接收银行流水、小票、支票、invoice、contract、accountant/client note，保留 evidence foundation，并标准化日期、金额、direction、账户、附件关联等客观字段。
3. `Transaction Identity Node` 分配或复用稳定 `transaction_id`，处理同一交易识别和重复导入，不做 vendor/entity 或会计分类判断。
4. `Profile / Structural Match Node` 先检查内部转账、已知贷款还款、已确认账户关系等 stable profile structural facts。只有 stable `Profile` 足以确定时才走结构性处理路径；candidate 或未确认结构信号不能强行处理。
5. `Entity Resolution Node` 处理普通非结构性交易的 counterparty / vendor / payee identity。它输出 `resolved_entity`、`resolved_entity_with_unconfirmed_role`、`new_entity_candidate`、`ambiguous_entity_candidates` 或 `unresolved`。它不做 COA/HST 判断，不创建 stable entity，不批准 alias/role，不 merge/split，不改 automation policy。
6. `Rule Match Node` 只在 stable active entity、approved alias、required role/context、automation policy、active approved rule、rule conditions 全部满足时执行 deterministic rule path。`new_entity_candidate`、candidate alias、unconfirmed role、ambiguous/unresolved identity 都不能支持 rule match。
7. `Case Judgment Node` 在 structural path 和 rule path 都不能完成时，基于 current evidence、entity/profile context、case memory 和 customer knowledge 做 runtime judgment。它可以输出 operational classification、pending、review-required 或 candidate signals，但不能把一次判断升级成 durable memory 或 governance authority。
8. `Coordinator / Pending Node` 把上游 blocking reason 转成 accountant 可回答的问题。它解决缺信息，不批准长期记忆。
9. `Review Node` 让 accountant 审核、批准、纠正系统结果和候选。它把 AI output 转成 accountant-controlled outcome。
10. `JE Generation Node` 把已确认或足够可信的分类结果转成 journal entry。它是纯计算层，不重新判断 entity/rule/case。
11. `Transaction Logging Node` 写入最终交易结果和 audit trail。`Transaction Log` 是只写 + 查询的审计层，不参与 runtime decision，也不是 learning layer。
12. `Case Memory Update Node` 在交易完成后沉淀 case memory，并提出 entity/alias/role/rule/governance candidates。candidate 不等于 approval。
13. `Governance Review Node` 处理长期高权限变化：alias approval/rejection、role confirmation、entity merge/split、automation policy change、rule promotion/modification/downgrade。
14. `Knowledge Compilation Node` 把 entity、case、rule、governance 历史编译成人和 agent 可读的客户知识摘要。摘要可辅助理解，但不能替代 Entity Log / Rule Log / Governance Log authority。
15. `Post-Batch Lint Node` 批次结束后检查 entity 拆合、rule 稳定性、case-to-rule 候选和 automation risk。系统可以自动降级风险权限；升级、放宽和 rule promotion 需要 accountant/governance approval。

## 新系统长期记忆层

- `Evidence Log`：保存原始证据和 evidence refs，不保存业务结论。
- `Entity Log`：保存 entity、alias、role、status、authority、risk flags、automation policy 和 evidence links，不保存 COA/HST/JE 结论。
- `Case Log`：保存真实历史案例、最终分类、证据、例外和 accountant correction；它替代旧 `Observations` 的主角色，但不只是聚合统计。
- `Rule Log`：保存 accountant/governance 已批准的 deterministic rules，不保存候选规则或一次性判断。
- `Transaction Log`：保存每笔交易最终处理结果和审计轨迹；不参与 runtime decision。
- `Intervention Log`：保存 accountant 介入、提问、回答、修正和确认过程。
- `Governance Log`：保存长期权威变化及其审批状态。
- `Knowledge Log / Summary Log`：保存编译摘要；不作为 deterministic rule source。
- `Profile`：保存客户结构档案，例如 bank accounts、internal transfer relationships、tax config、loans、employees。

## 旧系统一句话

旧系统是 pattern-centered automation：先把交易描述标准化成 `description / pattern`，再用 `Observations` 聚合历史分类事实，最后把稳定 pattern 升级成 `Rules`。

核心链条：

```text
raw files
-> parsed transaction data
-> transaction_id + canonical description/pattern
-> profile match
-> rules exact match
-> confidence classifier
-> coordinator / review
-> observations / rules / transaction log updates
```

## 旧系统主流程

1. `Data Preprocessing Agent` 接收 bank statement、receipt、cheque、attachments，解析、配对、去重，并生成标准交易数据。
2. 预处理阶段分配 `transaction_id`，并把可识别交易生成 canonical `description / pattern`。
3. `Node 1 / Profile Match` 用 `Profile.account_relationships` 等结构事实处理内部转账等确定性结构交易。
4. `Node 2 / Rules Match` 用 `transaction.description == rule.pattern` 做 exact deterministic match。
5. 命中 rule 后，系统生成 JE，写 `Transaction Log`。
6. 未命中 rule 的交易进入 `Node 3 / Confidence Classifier`。
7. Node 3 查询同 `(pattern, direction)` 的 `Observations`，结合 profile、COA、receipt、cheque、risk packs、domain packs，让 LLM 判断 account、HST 和 confidence。
8. 高置信度交易生成 JE，写 `Observations`，写 `Transaction Log`。
9. PENDING 交易交给 `Coordinator Agent` 向 accountant 提问。Accountant 确认后，Coordinator 生成 JE，写 `Observations` 和 `Transaction Log`。
10. 当前批次完成后，从 `Transaction Log` 生成 `report_draft`。
11. `Review Agent` 处理 accountant 审核：修正交易、处理 rule 修改/删除、observation 升级、pattern merge/rename/split、profile change、intervention log。
12. 最终 `.xlsx` 从 `Transaction Log` 导出。

## 新旧系统关键差异

| 维度 | 旧系统 | 新系统 |
| --- | --- | --- |
| 身份核心 | `description / pattern` | `entity + alias + role/context` |
| 历史记忆 | `Observations` 聚合统计 | `Case Log` 原子案例 + compiled knowledge |
| Rule 绑定对象 | `pattern + direction + amount_range` | `entity + approved alias + confirmed role/context + rule conditions` |
| 新交易第一问题 | 这串描述标准化成什么 pattern？ | 这笔 evidence 指向谁？ |
| AI 判断层 | Node 3 同时识别 vendor、选 COA、判 HST、判置信度 | Entity / Rule / Case Judgment / JE 分层拆开 |
| 学习方式 | pattern 聚合次数够了升级 rule | case memory 积累，稳定后经 governance/rule promotion |
| Authority 边界 | 集中在 Review Agent、字段标记和人工操作中 | 显式区分 candidate、approved、governance event、automation policy |
| 审计日志 | Transaction Log 清楚，但 AI reasoning 曾写入决策依据层 | 明确 AI summary 不能成为未来 authority |
| Governance | Review Agent 处理 rule/observation/profile/pattern 变更 | 独立 Governance Review Node + Governance Log |
| 批后体检 | observation 升级、review 修正、pattern 维护 | entity、rule、case、automation risk 的 post-batch lint |

最本质的变化：

旧系统把三个问题压在 `pattern` 上：

1. 这是谁？
2. 过去怎么分？
3. 能不能自动分？

新系统把它们拆开：

1. `Entity Resolution`：这是谁？
2. `Case Memory`：过去类似情况怎么处理？
3. `Rule / Automation Policy / Governance`：这次是否有 authority 自动处理？

## 旧系统仍值得保留的约束

这些不是因为旧系统存在所以正确，而是因为它们仍服务核心产品目标：

- `Transaction Log` 不参与 runtime decision，只做最终结果和审计追溯。
- `JE Generation` 应是纯确定性计算，不做 entity、rule、case 或会计语义判断。
- `Profile` 只存客户结构事实，不承担普通业务分类。
- Accountant 对最终分类、rule、profile、长期记忆变化拥有控制权。
- PENDING 必须能转成人能回答的问题，而不是把内部状态原样丢给 accountant。
- Review 阶段必须区分“这笔是例外”和“系统规则/记忆错了”。
- 人工纠正要进入可审计的 intervention / governance 记录，而不是只覆盖最终结果。

## 后续审计使用方式

审计新系统时，不要问“旧系统有没有对应物”，而要问：

> 这个 node / field / log / memory layer 是否解决了 pattern-centered 系统无法干净解决、但对记忆复用、证据支持、纠错学习、审计性或控制权真实重要的问题？

如果不能回答这个问题，该设计应被视为可疑，进入删除、合并、内联、重命名、收窄职责或暂缓讨论。