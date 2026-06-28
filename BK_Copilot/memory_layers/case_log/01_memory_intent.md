# Case Log - Memory Intent

## 1. 为什么这层 memory 必须存在

它解决的问题是：

> 为 future case-based judgment（未来基于案例判断）保存按 stable entity（稳定实体）组织的 completed-case precedent（已完成案例先例），让系统能复用过往有证据条件、例外上下文、复用权限和最终会计处理快照的记账判断，而不用把学习记忆混进身份权威、确定性规则或最终审计记录。

如果删除它，会失去：

- entity-indexed case memory（按实体索引的案例记忆）：系统看到同一 stable entity（稳定实体）时，无法读取过去的案例、例外、纠正和复用权限。
- evidence-conditioned precedent（有证据条件的先例）：系统无法知道某个历史记账判断是在什么 receipt、invoice、bank descriptor 或 exception context 下成立。
- correction learning（纠正学习）：accountant correction（会计师纠正）无法沉淀为未来判断时可见的案例约束。
- rule promotion review evidence（规则升级审核依据）：重复案例只能停留在 isolated transaction history（孤立交易历史），无法形成受控 rule promotion review（规则升级审核）依据。
- automation 风险依据：历史案例显示的 mixed-use risk（混用风险）、classification instability（分类不稳定）等无法进入治理候选，供会计师人发起 force_pending / promotion_lock。

这些能力不能由 runtime handoff（运行时交接）或现有 store（现有存储）覆盖，因为：

- runtime handoff 只描述当前交易，不能保存未来可复用的 completed-case learning memory。
- `Transaction Log`（交易日志）是 audit-facing final transaction record；它可以通过索引支持审计 / 报表查询，但不作为主 workflow 的 entity-indexed learning layer。
- `Entity Log`（实体日志）保存 identity authority，不保存 classification memory、case precedent 或 accounting treatment pattern。
- `Alias Log`（别名日志）是面向 Entity Resolution 的 alias→entity 反查 projection / index，不保存 case precedent，也不持有 Alias authority。
- `Rule Log`（规则日志）保存 approved deterministic rules（已批准确定性规则），不保存尚未升级为规则的案例判断。
- `Knowledge Summary`（知识摘要）是 readable context（可读上下文），不能替代 source authority（来源权威）。

## 2. 保存什么

这层 memory 保存的是每笔 stable-linked finalized transaction（稳定实体关联且已完成交易）的 reusable case view（可复用案例视图）。字段尚未冻结；下表用预设字段说明 M1-M2 已确认需要保存的信息类型，exact 字段名 / enum / schema / validation 留 M3。

| # | 预设字段 | 保存的信息 / 作用 |
| --- | --- | --- |
| 1 | `case_id` | 案例唯一标识。用于 dedup（防同笔重复写入）与被引用的锚点。 |
| 2 | `entity_id` | 主索引：这是谁的案例。Case Log 按 entity 组织的前提。 |
| 3 | `transaction_log_ref` | finalization proof / 审计锚点。证明案例来自已完成交易；也作为通往修改历史、完整 JE、原文等审计留痕的桥。 |
| 4 | `alias` | Alias 的 authority 在 Entity Log；Alias Log 是 alias→entity 反查 projection / index，不持有 authority；本字段只是该交易的 alias 表面快照 / 引用，Case Log 不持有、不立 Alias authority。 |
| 5 | `direction` | 资金方向。改变会计含义，是模式区分的必要维度。 |
| 6 | `amount` | 交易金额绝对值。判断是否落在同一模式区间的基础。 |
| 7 | `date` | 交易日期。用于 recency 和读取层聚合的时间维度；period 可由它派生。 |
| 8 | `account` | 最终 COA 科目快照。被学习的答案本身；最终权威仍在 Transaction Log。 |
| 9 | `hst_gst_treatment` | 税务处理方式快照。答案的一部分；最终权威仍在 Transaction Log。 |
| 10 | `evidence_refs` | 证据引用，指向 Evidence Log，不存原文。用于回溯与下钻。 |
| 11 | `evidence_condition` | 当时有哪类证据支撑，例如有小票、invoice、支票或仅银行流水。决定该先例何时可被复用。 |
| 12 | `confirmed_by` | 记录结论的权威来源类型，用于决定该 case 的复用强度；exact enum 留 M3。若来源类型为 system 高置信度，只表示强辅助先例证据，不自动授予未来自动化放行；具体人员和完整审计留痕归 Transaction Log。 |
| 13 | `use_level` | 复用权限。表达这条案例未来允许被怎么用，并以失效取值承载最小 supersession 复用安全。 |
| 14 | `context_note` | 例外说明 + 针对这一笔的人工批注。解释为什么这笔不能简单泛化；entity / 类别级通则批注 / 快捷经验归 entity_level Knowledge Summary，Case Log 不存该批注基础层。 |

Case Log 不保存独立 pattern rollup（模式聚合）作为真值。某 entity 下的分布、次数、一致性、最近出现等聚合信息全部由逐笔 case 派生，由读取层聚合机制在需要时产出。

其中可以成为 reusable authority（可复用权威）的是：

- stable-linked completed case precedent：在记录的 evidence condition、context note 和 `use_level` 范围内，可作为 future case-based judgment 的案例依据。
- accountant-corrected or accountant-confirmed case reference：可作为未来避免重复错误或解释例外的案例依据。
- case history pattern：只能作为 case-based judgment 或 rule promotion / force_pending / promotion_lock review 的历史依据，不能直接成为 deterministic rule 或 durable mutation。

只作为 trace / context / explanation（追溯 / 上下文 / 解释）的是：

- `transaction_log_ref` 或 finalization proof 本身。
- evidence refs（证据引用）和 evidence condition 的可追溯说明。
- exception explanation（例外说明）。
- correction / exception explanation（纠正 / 例外说明）。
- rule promotion / force_pending / promotion_lock review 所需的历史依据；候选生成与审批归治理路径，候选信号本身不作为 Case Log 存储字段。
- readable summary（可读摘要）或 Knowledge Compilation 输出。

## 3. 绝不保存什么

这层 memory 绝不保存：

- transaction process log（交易处理过程日志）。
- final transaction audit record（最终交易审计记录）。
- 完整 processing path（处理路径）作为学习依据。
- 修改历史。
- 完整 JE 或 JE 生成过程。
- entity identity authority（实体身份权威）。
- Alias relationship（别名关系）。
- `entity_status`（实体生命周期状态）。
- `force_pending` / `promotion_lock`。
- active rule payload（生效规则内容）。
- raw evidence blob（原始证据正文）。
- unapproved AI reasoning（未经批准的 AI 推理）作为 future authority。
- unknown entity（未知实体）作为 durable identity handle。
- 以 description / 类别（而非 entity）为索引的学习记录。
- case-derived candidate signal refs（案例衍生候选信号引用）作为 durable storage 字段。
- entity / 类别级通则批注 / 快捷经验；归 entity_level Knowledge Summary，Case Log 不存该批注基础层。
- 独立 pattern rollup 或 pattern_ref 作为存储真值。

它也不能替代：

- `Evidence Log`（证据日志）：raw evidence 和 evidence refs 的 source of truth。
- `Transaction Log`（交易日志）：每笔交易 final outcome、review trace 和 audit trail 的 source of truth。
- `Entity Log`（实体日志）：stable entity identity、Alias、status 和 force_pending / promotion_lock 的 authority store。
- `Alias Log`（别名日志）：alias→entity 反查 projection / index；不持有 Alias authority。
- `Rule Log`（规则日志）：approved deterministic rules 的 source of truth。
- `Governance Log`（治理日志）：长期高权限 mutation、approval、rejection、downgrade、merge / split 的 audit source。
- `Knowledge Summary`（知识摘要）：可读摘要不能替代 Case Log source authority；反过来，Case Log 也不替代摘要层。

## 4. 对核心产品目标的贡献

这层 memory 必须清楚支持以下至少一项：

- [x] 记忆复用
- [x] 有证据支持的建议
- [x] accountant correction learning
- [x] 审计性
- [x] accountant control
- [x] 自动化率提升

具体贡献：

- 让同一 stable entity 下的历史案例可以被未来交易复用，而不是每笔交易从零判断。
- 让 case-based judgment 读取的不是孤立结论，而是带 evidence condition、context note、confirmed_by 和 use_level 的先例。
- 让 accountant correction 进入未来判断上下文，但不自动污染 Rule Log 或 Entity Log。
- 让 rule promotion / force_pending / promotion_lock review 可以使用案例历史作为依据，但候选生成与审批归治理路径。
- 通过要求 finalization proof 和 traceable evidence，避免未完成交易、模型推理或非权威摘要进入长期案例记忆。
- 通过区分 stable-linked case 和 unknown entity，防止身份不稳定时产生强学习记忆。

## 5. 已知约束

- `entity_id` 是 Case Log 的主要索引。
- Case Log 是 entity-indexed reusable-precedent layer，不是 Transaction Log 的 entity 副本。
- 所有已关联 stable entity 且已 finalized 的交易均具备 Case Log 写入资格；系统不在写入时预判学习价值。
- Case Log 的真值单位是逐笔已完成案例先例；聚合信息由逐笔 case 派生，不作为存储真值。
- Case Log 必须能引用 `transaction_log_ref` 或等价 finalization proof；它不能替代或重写 Transaction Log。
- Transaction Log 是 final 交易事实、完整 processing path、修改历史和 audit trail 的 source of truth；Case Log 只保存最新可学习状态。
- Stable entity 是强 case reuse 的正常身份基础。
- Unknown entity 不写入 Entity Log，也不能被 Case Log 当作 identity handle 引用。
- 一笔交易可以在 entity 未解析的情况下被 accountant 完成分类并 finalize；这类交易只 finalize 到 Transaction Log，不进入 Case Log（无 entity_id 索引键），也不创建 Entity Log 记录。
- 这类交易不得催生以 description / 类别为索引的学习记录；其身份缺口（情况 Y）是挂在 Transaction Log 上的不阻塞复查线索（标记落点已定为 Transaction Log，由 Transaction Log L2 用户拍板锁定；字段形态留 L3），不是 candidate entity，也不是 Case Log 可引用的 durable identity handle。情况 X / 情况 Y 的区分见 `BK_Copilot/workflow_nodes/coordinator_node/02_logic_and_boundaries.md`（原 `Coordinator Question.md` 已删除，内容已吸收）。
- 每条 case 必须具备 `use_level` 复用权限语义；exact enum 留 M3。
- `confirmed_by` = system 高置信度的 case 只是强辅助先例证据，不自动授予未来同类交易自动化放行；真正自动化仍需 accountant 深度参与；系统高置信度判断的限制由 Case Judgment 节点界定，Case Log 只引用、不重定义。
- 被纠正 / 冲销 / 治理限制后失效的旧 case 不得被未来节点静默当作有效正面先例复用；supersession 血缘链接和执行机制留后续。
- Case Log 可以提供 force_pending / promotion_lock review 或 rule promotion 的历史依据，但不能直接修改 Entity Log。
- Case Log 可以提供 rule promotion 评估依据，但不能直接创建、升级、修改、删除或降级 active rule。
- entity / 类别级通则批注 / 快捷经验的读取方向已定：按 `entity_id` 锁定后，按需加载该 entity 的 entity_level Knowledge Summary 注入给 Case Judgment；KS 记录自带 `authority_labels` / `downstream_usage_limits`，作为“建议不被当规则”的护栏。具体同步 co-read 哪些 source memory 仍留 KS / Case Judgment L2 seam。
- Repeated outcome 不能自动升级为 approved rule。
- Case Log 不保存 unapproved AI reasoning 作为 future authority。

## 6. 未决定问题

- Case Log record 的 exact 字段名 / enum / schema / validation。
- `use_level` 与 `confirmed_by` 的 exact enum 取值。
- 共享 accounting outcome（COA / HST / split / allocation）的全局字段结构。
- Case Judgment 读取 Case Log 时的 rollup / retrieval 机制。
- `Case Log` 与 `Transaction Log` 的 exact trigger order、exact writer 和多 log 统一 finalization 机制。
- supersession 的血缘链接字段与 corrected / reversed / split 后的重判执行机制。
- merge / split / archive 后旧 case 重新归属的治理执行机制。
- case-derived rule promotion / force_pending / promotion_lock candidate 进入治理的 exact contract。
- entity_level Knowledge Summary 注入 Case Judgment 时具体同步 co-read 哪些 source memory 的 exact contract。
