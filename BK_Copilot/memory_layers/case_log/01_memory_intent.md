# Case Log - Memory Intent

## 1. 为什么这层 memory 必须存在

它解决的问题是：

> 为 future case-based judgment（未来基于案例判断）保存按 entity（实体）组织的 completed-case precedent（已完成案例先例），让系统能复用过往有证据条件、例外上下文和纠正引用的记账判断，而不用把学习记忆混进身份权威、确定性规则或最终审计记录。

如果删除它，会失去：

- entity-indexed case memory（按实体索引的案例记忆）：系统看到同一 stable entity（稳定实体）时，无法读取过去的案例、例外和纠正记录。
- evidence-conditioned precedent（有证据条件的先例）：系统无法知道某个历史记账判断是在什么 receipt、invoice、bank descriptor、role gap 或 exception context 下成立。
- correction learning（纠正学习）：accountant correction（会计师纠正）无法沉淀为未来判断时可见的案例约束。
- rule promotion candidate（规则升级候选）：重复案例只能停留在 isolated transaction history（孤立交易历史），无法形成受控 rule promotion review（规则升级审核）依据。
- automation risk / entity risk evidence（自动化风险 / 实体风险依据）：历史案例显示的 mixed-use risk（混用风险）、classification instability（分类不稳定）或 automation risk（自动化风险）无法进入治理候选。

这些能力不能由 runtime handoff（运行时交接）或现有 store（现有存储）覆盖，因为：

- runtime handoff 只描述当前交易，不能保存未来可复用的 completed-case learning memory。
- `Transaction Log`（交易日志）是 audit-facing final transaction record，不参与 runtime judgment，也不是未来 learning source。
- `Entity Log`（实体日志）保存 identity authority，不保存 classification memory、case precedent 或 accounting treatment pattern。
- `Alias Log`（别名日志）只保存 confirmed transaction surface text -> stable entity 的身份复用关系，不保存 case precedent。
- `Rule Log`（规则日志）保存 approved deterministic rules（已批准确定性规则），不保存尚未升级为规则的案例判断。
- `Knowledge Summary`（知识摘要）是 readable context（可读上下文），不能替代 source authority（来源权威）。

## 2. 保存什么

这层 memory 保存：

- 从 completed transaction（已完成交易）中抽取的 reusable bookkeeping precedent（可复用记账先例）。
- 该 precedent 成立所依赖的 evidence condition（证据条件）。
- exception context（例外上下文），例如 role gap（角色缺口）、mixed-use risk、identity ambiguity（身份歧义）或其他导致不能简单泛化的上下文。
- correction reference（纠正引用），用于说明历史案例是否来自 accountant correction 或最终确认。
- `transaction_log_ref` 或等价 finalization proof，用来证明该案例来自已完成交易。
- 与 stable entity 或 candidate entity 相关的 case reference；两者 authority 不同。
- case-derived candidate signal（案例衍生候选信号），例如 rule promotion candidate、automation risk review candidate、entity risk update candidate。

其中可以成为 reusable authority（可复用权威）的是：

- stable-linked completed case precedent：在记录的 evidence condition 和 exception context 范围内，可作为 future case-based judgment 的案例依据。
- accountant-corrected or accountant-confirmed case reference：可作为未来避免重复错误或解释例外的案例依据。
- case history pattern：只能作为 case-based judgment 或 governance candidate 的依据，不能直接成为 deterministic rule。

只作为 trace / context / explanation（追溯 / 上下文 / 解释）的是：

- `transaction_log_ref` 或 finalization proof 本身。
- evidence refs（证据引用）和 evidence condition 的可追溯说明。
- exception explanation（例外说明）。
- correction explanation（纠正说明）。
- candidate-linked case（候选实体关联案例）：默认只能作为 weak context 或 governance evidence。
- case-derived risk / policy / rule promotion signal：只能作为 candidate，不是 durable mutation（长期变更）本身。
- readable summary（可读摘要）或 Knowledge Compilation 输出。

## 3. 绝不保存什么

这层 memory 绝不保存：

- transaction process log（交易处理过程日志）。
- final transaction audit record（最终交易审计记录）。
- 完整 processing path（处理路径）作为学习依据。
- entity identity authority（实体身份权威）。
- Alias relationship（别名关系）。
- confirmed role（已确认角色）。
- `entity_status`（实体生命周期状态）。
- `automation_policy`（自动化策略）。
- active rule payload（生效规则内容）。
- raw evidence blob（原始证据正文）。
- unapproved AI reasoning（未经批准的 AI 推理）作为 future authority。
- unknown entity（未知实体）作为 durable identity handle。

它也不能替代：

- `Evidence Log`（证据日志）：raw evidence 和 evidence refs 的 source of truth。
- `Transaction Log`（交易日志）：每笔交易 final outcome、review trace 和 audit trail 的 source of truth。
- `Entity Log`（实体日志）：stable entity identity、Alias、confirmed role、status 和 automation policy 的 authority store。
- `Alias Log`（别名日志）：confirmed transaction surface text -> stable entity 的 identity reuse memory。
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
- 让 case-based judgment 读取的不是孤立结论，而是带 evidence condition、exception context 和 correction reference 的先例。
- 让 accountant correction 进入未来判断上下文，但不自动污染 Rule Log 或 Entity Log。
- 让 Post-Batch Lint、Review 和 Governance Review 可以用案例历史识别 rule promotion、automation risk 或 entity risk candidate。
- 通过要求 finalization proof 和 traceable evidence，避免未完成交易、模型推理或非权威摘要进入长期案例记忆。
- 通过区分 stable-linked case、candidate-linked case 和 unknown entity，防止身份不稳定时产生强学习记忆。

## 5. 已知约束

- `entity_id` 是 Case Log 的主要索引。
- Case Log 只保存 completed-case learning memory，不保存 transaction process log 或 final audit record。
- Case Log 必须能引用 `transaction_log_ref` 或等价 finalization proof；它不能替代或重写 Transaction Log。
- Stable entity 是强 case reuse 的正常身份基础。
- Candidate entity 可以被 Case Log 引用，但 candidate-linked case 默认只能作为 weak context 或 governance evidence。
- Unknown entity 不写入 Entity Log，也不能被 Case Log 当作 identity handle 引用。
- Case Log 可以提供 entity-level risk 或 automation policy candidate 的依据，但不能直接修改 Entity Log。
- Case Log 可以提供 rule promotion candidate 的依据，但不能直接创建、升级、修改、删除或降级 active rule。
- Repeated outcome 不能自动升级为 approved rule。
- Case Log 不保存 unapproved AI reasoning 作为 future authority。

## 6. 未决定问题

- 哪些 completed transaction 具备 case write eligibility。
- duplicate / corrected / reversed / split transaction 如何影响 case supersession。
- candidate-linked case 在 candidate 后来升级为 stable entity 后是否自动变成 strong precedent。
- `new_entity_candidate` 或 candidate identity signal 的持久化位置、namespace 和 Case Log 引用方式。
- `Case Log` 与 `Transaction Log` 的 exact trigger order。
- 交易完成分类后的多 log 统一写入机制。
- 哪些 case-derived signals 可以进入 governance candidate，以及具体审批路径。
- Case Log 是否进入 M3 data contract；在字段、消费者和 mutation authority 稳定前不得进入。
