# JE Generation Node — 设计摘要

> 由原 Stage 1/2/3 合并瘦身而来；保留定位、边界、contract 字段和开放问题，删除阶段说明、examples、self-review、历史读档记录和重复解释。

## 1. 定位与职责

Stage 1 已确定以下内容：
- `JE Generation Node` 是 pure journal-entry computation node。
- 它位于 finalizable current outcome 之后、`Transaction Logging Node` 之前。
- 它把已具备当前交易落账 authority 的 accounting outcome 转换为 double-entry journal entry。
- 它必须保持借贷平衡和 tax treatment 一致性。
- 它不选择 COA 科目。
- 它不判断 HST/GST 是否适用。
- 它不替 accountant 作最终业务决定。
- 它不生成 accountant approval。
- 它不写 `Transaction Log`。
- 它不写或修改 Entity / Case / Rule / Governance memory。
- 它不执行 rule promotion、entity governance、profile update 或长期客户知识沉淀。
- 如果当前交易尚不能 finalization，它应暴露不能生成 JE 的原因，而不是自行猜测。

## 2. 逻辑与边界

### Trigger Boundary

`JE Generation Node` 在当前交易具备 finalizable accounting outcome，且需要转换为 journal entry 时触发。

概念触发条件是：
- 当前交易已经完成 evidence intake、transaction identity 和必要的上游 classification / review workflow；并且
- 当前交易已有可追溯、可绑定到当前交易的 accounting outcome；并且
- 当前 outcome 的 authority 足以支持本笔交易进入 JE finalization；并且
- 当前交易尚未完成 final transaction logging。
典型触发来源包括：
- profile-backed structural path 已形成可落账结果。
- deterministic rule-handled path 已形成可落账结果。
- case-supported path 在当前 authority 边界内已形成可继续结果。
- pending clarification 或 review 已回收 accountant-provided context，并形成可继续结果。
- accountant 已在 `Review Node` 中批准或修正当前交易 outcome。
以下情况不能触发正常 JE generation：
- 当前交易仍是 pending、still-pending、not-approved 或 unresolved。
- 当前交易存在 unresolved identity、ambiguous evidence、conflicting evidence、authority block、governance block 或 review-required condition。
- 当前分类、COA、tax treatment、amount basis 或 direction basis 不足以生成一致分录。
- 当前 outcome 只是 candidate signal、review draft、governance candidate、case memory candidate 或 rule candidate。

### Input Categories

Stage 2 按判断作用组织 input categories，不列字段清单。

#### Objective transaction basis

说明当前交易的客观金额、方向、日期、账户语境和交易身份基础。

#### Finalizable accounting outcome basis

说明当前交易最终或当前可 finalization 的会计处理结果是什么，以及来自哪条上游 authority path。

核心边界：
- 可 finalization 的 outcome 可以支持 JE。
- 未批准、仍 pending、review-required、candidate-only 或 governance-needed context 不能支持 JE。
- 本节点不能把上游 rationale 重新解释成新的 accounting outcome。

#### COA and account-mapping basis

说明当前 outcome 应落到哪些已存在或已授权的会计科目语境。

#### Tax treatment basis

说明当前交易的 GST/HST 处理是否已由上游 outcome、客户 tax config、approved rule、structural logic 或 accountant decision 给出。

本节点不能自行判断：
- 当前交易是否 taxable、exempt、zero-rated 或 out-of-scope
- 应使用哪个 tax rate
- 应落入 HST/GST Receivable、HST/GST Payable 或其他 tax control treatment
- receipt / invoice 是否足以支持 ITC 或 tax claim

#### Authority and review basis

说明当前 outcome 为什么可以进入 JE，或为什么不能进入 JE。

#### Audit-support context

说明后续 `Transaction Logging Node` 需要知道本次 JE 来自哪个 outcome、哪些 evidence reference、哪些 review / intervention context 和哪些 upstream rationale。

### Output Categories

Stage 2 只定义 conceptual output categories，不冻结 routing enum 或对象形状。

#### Journal entry result

含义：当前交易的 finalizable accounting outcome 已被转换成借贷平衡、tax treatment 一致、可供 final logging 和输出使用的 journal entry。

边界：
- 这是当前交易的分录计算结果。
- 它不等于 accountant review approval。
- 它不等于 `Transaction Log` write。
- 它不等于 Case Log memory write。
- 它不等于 rule、entity、alias、role 或 automation-policy approval。

#### JE-blocked / not-finalizable handoff

含义：当前交易尚不能生成 journal entry，因为缺少或冲突的是 finalization 必需条件。

典型原因包括：
- accounting outcome 尚未 finalizable
- account basis 缺失或冲突
- tax treatment basis 缺失或冲突
- amount / direction basis 不一致
- authority / review basis 不足
- governance block 或 review-required condition 尚未解决
边界：
- Blocked handoff 是安全停止，不是分类判断。
- 本节点不能用 fallback 科目、默认税务处理或 confidence 语言绕过阻断。
- 后续应由 Review、Coordinator、上游 workflow 或 governance workflow 处理。

#### Consistency issue signal

含义：本节点在分录计算前后发现 current outcome、objective transaction basis、tax treatment、account basis 或 authority trace 之间存在不一致。

边界：
- 该 signal 只说明 JE 不能安全 finalization。
- 它不自行修正 accounting outcome。
- 它不自行创建 governance event。
- 它不写长期 memory。

#### Audit-support handoff

含义：本节点可以把 JE result、JE-blocked reason、calculation rationale 和 upstream authority trace 交给 `Transaction Logging Node` 或输出 flow。

边界：
- Handoff 不等于 durable audit log。
- Final audit-facing write 属于 `Transaction Logging Node`。
- Handoff 不能被 `Case Log`、`Rule Log`、Entity / Governance memory 直接当成 durable authority。

#### Candidate signal

含义：如果 JE generation 暴露出长期配置或治理问题，本节点可以提出候选信号。

典型候选包括：
- COA / account-mapping review candidate
- tax configuration review candidate
- rule condition gap candidate
- structural-profile issue candidate
- review / governance candidate
边界：
- Candidate signal 只表示后续应评估。
- 它不修改 COA、Profile、Rule Log、Entity Log、Case Log 或 Governance Log。
- 它不批准 governance event。

### Deterministic Code vs LLM Semantic Judgment

Stage 2 的核心边界是：

#### Deterministic code responsibility

Deterministic code 负责：
- 判断本节点是否被触发：当前交易是否已有 finalizable accounting outcome，且尚未 final logging。
- 检查当前 outcome 是否具备 JE generation 所需 authority。
- 检查 pending、review-required、governance-needed、candidate-only 或 not-approved context 不会进入 JE。
- 使用 objective transaction basis、approved account basis 和 approved tax basis 计算借贷分录。
- 保持金额方向语义一致，避免重复 sign 解释。
- 保持借贷平衡。
- 保持 GST/HST treatment 与 approved outcome 和客户 tax config 一致。
- 识别 tax、amount、direction、account basis 或 authority trace 的缺失 / 冲突。
- 区分 JE result、JE-blocked handoff、consistency issue signal 和 candidate signal。
- 防止本节点写入 `Transaction Log` 或长期 memory。
- 防止 `Transaction Log`、Knowledge Summary、case precedent 或 candidate signal 被当作当前 JE authority。

#### LLM semantic judgment responsibility

LLM 通常不应参与 JE 生成本身。

如果后续阶段允许 LLM 辅助，它最多可以在受限边界内帮助：
- 把 JE-blocked reason 总结成 accountant / reviewer 可读说明。
- 解释分录计算为什么不能继续。
- 将多个 consistency issue 合并成更清晰的 review / pending explanation。
- 生成候选信号的自然语言说明。

#### Hard boundary

LLM 不能：
- 选择 COA 科目
- 判断 HST/GST tax treatment
- 决定 current outcome 是否 finalizable
- 把 pending 或 review-required 交易推进到 JE
- 修正 amount、direction 或 tax basis
- 为了平衡分录而发明调整行
- 使用 fallback account 或默认 tax treatment 补缺口
- 替 accountant approve current outcome
- 创建或修改 rule
- 修改 Profile、Entity Log、Case Log、Rule Log 或 Governance Log
- 写入 `Transaction Log`

### Accountant Authority Boundary

Accountant 仍然拥有最终 accounting decision 和 durable authority。

本节点不能：
- 选择或修正当前交易分类
- 选择或修正 COA account
- 选择或修正 GST/HST treatment
- 把 accountant 模糊回答解释为批准
- 把系统高置信建议解释为 accountant approval
- 把 JE 成功生成解释为 accountant 已审核
- 把当前交易 JE 结果变成长期客户政策

### Governance Authority Boundary

Governance-level changes 不属于 `JE Generation Node` authority。

本节点不能：
- 修改 `Profile`
- 创建、升级、修改、删除或降级 active rule
- approve / reject alias
- confirm role
- create stable entity authority
- merge / split entity
- archive / reactivate entity
- change automation policy
- 修改 tax configuration 或 account-mapping authority
- 批准 governance event
- invalidate durable memory

### Memory / Log Boundary

Stage 2 采用三层边界：read / consume、candidate-only、no direct mutation。

#### Read / consume boundary

`JE Generation Node` 可以读取或消费以下 conceptual context：
- objective transaction basis、stable transaction identity 和 current evidence references
- finalizable accounting outcome from upstream workflow
- approved account basis、customer COA / accounting configuration 和 structural accounting context
- approved tax treatment basis 和客户 tax config
- review / intervention context that establishes current outcome authority
- upstream rationale and authority trace needed for audit-support handoff

#### Candidate-only boundary

本节点只能作为候选或 handoff signal 表达：
- account-mapping review candidate
- tax configuration review candidate
- profile structural issue candidate
- rule condition gap or rule conflict candidate
- review-needed or governance-needed candidate
- evidence / amount / direction inconsistency candidate

#### No direct mutation boundary

本节点绝不能：
- 写入 `Transaction Log`
- 写入或修改 `Case Log`
- 写入或修改 `Rule Log`
- 写入或修改 stable `Entity Log` authority
- 写入或批准 `Governance Log` mutation
- 修改 `Profile`
- 修改 COA / tax config / automation policy
- 创建 stable case memory
- 创建 stable rule
- 批准 alias、role、entity 或 governance event
- 把 JE result 或 JE-blocked reason 当作长期 authority

### Insufficient / Ambiguous / Conflicting Evidence Behavior

Stage 2 采用优先级边界：authority first、accounting basis second、calculation consistency third。

#### Authority first

如果当前 outcome 尚未具备 JE finalization authority，本节点不能生成 journal entry。

典型情况：
- transaction outcome still pending
- review not approved
- accountant correction ambiguous
- unresolved / ambiguous identity 仍影响 current outcome
- review-required policy 尚未处理
- governance block 存在
- 当前内容只是 candidate signal

#### Accounting basis second

如果 authority 允许继续，但 account basis、tax basis、amount basis 或 direction basis 不足，本节点仍不能生成 journal entry。

典型情况：
- COA account 未确定或存在冲突
- GST/HST treatment 未确定或与 evidence / config 冲突
- tax control treatment 不明确
- 金额、方向或退款 / reversal 语义不一致
- 当前交易需要 split 或 multi-line 处理，但上游 outcome 未给出足够会计 basis

#### Calculation consistency third

如果输入 basis 看似足够，但计算结果不能保持借贷平衡、tax treatment 一致或 traceability 完整，本节点应阻止 finalization。

典型情况：
- debit / credit 不平衡
- tax line 与 approved tax basis 不一致
- gross / net / tax 关系不一致
- 交易方向与分录方向矛盾
- JE result 无法绑定到当前 transaction identity 或 authority trace

#### Conflict preservation

如果 current outcome 与 raw evidence、Profile、COA / tax config、review decision、rule result、case rationale 或 accountant correction 冲突，本节点不能选择一个版本继续。

#### Hard boundary

- JE generation 是 pure computation，不是 hidden classification layer。
- 能算出平衡分录不等于该 outcome 有 accountant authority。
- 不能为了 batch completion 使用 suspense / fallback / default tax treatment，除非该处理本身已由上游 authority 明确允许。
- `Transaction Log` 是 audit-facing final log，不参与 runtime JE decision。
- 模糊、冲突或缺失 basis 不能被包装成 confidence。

## 3. Contract 字段摘要

### Contract Position in Workflow

`JE Generation Node` 位于：

#### Upstream handoff consumed

本节点消费一个 runtime `je_generation_request`。

上游必须已经完成或明确给出：
- objective transaction basis；
- stable `transaction_id`；
- 当前交易的 accounting outcome；
- account basis；
- tax treatment basis；
- authority / review basis；
- audit-support references。

#### Downstream handoff produced

本节点输出一个 runtime `je_generation_result`。

下游按结果类别消费：
- `Transaction Logging Node`：消费 `journal_entry_result`、JE rationale 和 authority trace，用于 final audit logging；它才负责写 `Transaction Log`。
- `Review Node` / `Coordinator / Pending Node`：消费 `je_blocked_handoff` 或 `consistency_issue_signal`，处理缺失、冲突、review-required 或 accountant clarification。
- `Governance Review Node`：只可消费 candidate-only governance / configuration signal。
- output flow：可消费已生成的 journal entry result；不能把 output artifact 当作 durable memory。

#### Logs / memory stores read

本节点可以读取或消费：
- `Evidence Log` references；
- objective transaction record / transaction identity；
- upstream finalizable outcome；
- approved account basis / customer COA context；
- approved tax treatment basis / customer tax config；
- review / intervention context that establishes current authority；
- upstream rationale and authority trace；
- `Profile` 中与 bank account、tax config 或 structural transaction 有关的稳定配置引用。

#### Logs / memory stores written or candidate-only

本节点不直接写入任何 durable log / memory store。

它不能写：
- `Transaction Log`
- `Case Log`
- `Rule Log`
- `Entity Log`
- `Governance Log`
- `Knowledge Log / Summary Log`
- `Profile`
- COA / tax config / automation policy
本节点只可以输出 candidate-only signals，例如：
- `account_mapping_review_candidate`
- `tax_configuration_review_candidate`
- `profile_structural_issue_candidate`
- `rule_condition_gap_candidate`
- `governance_review_candidate`

### Input Contracts

#### `je_generation_request`

Required fields：
- `transaction_basis`
- `finalizable_accounting_outcome`
- `account_mapping_basis`
- `tax_treatment_basis`
- `finalization_authority_basis`
- `audit_support_context`
Optional fields：
- `complex_treatment_basis`
- `effective_governance_constraints`
- `request_trace`
Validation / rejection rules：
- 缺少 `transaction_basis.transaction_id` 时，input invalid。
- 缺少 `finalizable_accounting_outcome.outcome_status` 时，input invalid。
- `finalization_authority_basis.finalization_status != finalizable` 时，不得生成 JE。
- 如果 request 只包含 candidate signal、review draft、pending context、governance candidate 或 unapproved proposal，不得生成 JE。
- `request_trace` 只能用于 runtime diagnostics，不能成为 account、tax 或 authority source。
Runtime-only vs durable references：
- `je_generation_request` 是 runtime-only envelope。
- `transaction_id`、`evidence_refs`、`rule_id`、`entity_id`、review refs、governance refs、COA refs、tax config refs 是 durable references；它们不表示本节点拥有写入权。

#### `transaction_basis`

Required fields：
- `transaction_id`：稳定交易 ID。
- `transaction_date`
- `amount_abs`：绝对值金额。
- `direction`；allowed values: `inflow`, `outflow`。
- `bank_account`
- `currency`
- `evidence_refs`：至少一个 evidence reference。
Optional fields：
- `raw_description`
- `description`
- `pattern_source`
- `receipt_refs`
- `cheque_info_refs`
- `import_batch_id`
- `bank_account_control_account_ref`
Validation / rejection rules：
- `amount_abs` 必须为正数；金额正负号不能重复表达 direction。
- `direction` 必须显式给出，不能从 signed amount 推断。
- `currency` 缺失时不得生成 final JE，除非上游 shared transaction contract 已明确默认货币。
- `evidence_refs` 不能为空。
- `description = null` 和 `pattern_source = null` 是有效状态；本节点不能要求 canonical description。
- 如果 `bank_account_control_account_ref` 缺失，必须能从 approved Profile / COA context 追溯到客户侧银行账户对应的会计科目，否则 JE blocked。
Runtime-only vs durable references：
- 客观字段来自 current transaction record。
- `evidence_refs` 是 durable evidence references。
- `bank_account_control_account_ref` 是 COA / Profile reference；本节点不能创建或修改该 mapping。

#### `finalizable_accounting_outcome`

Required fields：
- `outcome_id`
- `outcome_source_type`
- `outcome_status`
- `accounting_treatment_ref`
- `primary_account_ref`
- `classification_summary`
- `tax_treatment_ref`
- `source_evidence_refs`
Allowed `outcome_source_type` values：
- `profile_structural_result`
- `approved_rule_result`
- `case_supported_result`
- `pending_clarification_resolved`
- `accountant_review_approved`
- `accountant_review_corrected`
Allowed `outcome_status` values：
- `finalizable`
- `pending`
- `review_required`
- `not_approved`
- `governance_blocked`
- `conflicting`
- `candidate_only`
Optional fields：
- `entity_ref`
- `rule_ref`
- `case_refs_used`
- `review_decision_ref`
- `intervention_refs`
- `accountant_correction_ref`
- `upstream_rationale`
- `confidence_label`
- `classification_basis`
Validation / rejection rules：
- 只有 `outcome_status = finalizable` 可以支持 normal JE generation。
- `case_supported_result` 只有在 `finalization_authority_basis` 明确允许当前交易 operational finalization 时才可支持 JE；本节点不能把 Case Judgment recommendation 当作 accountant approval。
- `candidate_only`、`review_required`、`pending`、`not_approved`、`governance_blocked` 或 `conflicting` 不能支持 JE。
- `primary_account_ref` 必须来自 approved accounting outcome、approved rule、accountant correction、structural mapping 或 approved COA context；不能由本节点推断。
- `classification_summary` 只是 human-readable summary，不能替代 `primary_account_ref` 或 `accounting_treatment_ref`。
Runtime-only vs durable references：
- 该对象是 runtime handoff。
- 其中的 refs 可以指向 durable sources，但本节点不能把 outcome 写入 `Case Log`、`Rule Log`、`Entity Log` 或 `Transaction Log`。

#### `account_mapping_basis`

Required fields：
- `primary_account_ref`
- `bank_account_control_account_ref`
- `account_mapping_authority`
- `account_refs_source`
Optional fields：
- `receivable_or_payable_account_ref`
- `clearing_account_ref`
- `contra_account_ref`
- `split_account_refs`
- `account_mapping_notes`
Allowed `account_mapping_authority` values：
- `profile_structural_mapping`
- `approved_rule_mapping`
- `accountant_review_mapping`
- `accountant_correction_mapping`
- `approved_coa_config`
Validation / rejection rules：
- `primary_account_ref` 与 `finalizable_accounting_outcome.primary_account_ref` 必须一致，或必须有 accountant correction / approved mapping 解释差异。
- `bank_account_control_account_ref` 必须能绑定到 `transaction_basis.bank_account`。
- 如果 outcome 需要 split / multi-line classification，必须提供 `split_account_refs` 或等价 approved account basis；否则 JE blocked。
- 本节点不能创建 COA account、选择 fallback account、合并 account 或把 unmatched account name 当作 approved account reference。
Runtime-only vs durable references：
- Account refs 指向 durable COA / Profile / rule / review authority。
- 本节点只读这些 references；不能修改 account mapping。

#### `tax_treatment_basis`

Required fields：
- `tax_treatment_status`
- `tax_authority_source`
- `tax_amount_basis`
- `tax_control_treatment`
Allowed `tax_treatment_status` values：
- `not_applicable`
- `taxable`
- `exempt`
- `zero_rated`
- `out_of_scope`
- `reverse_charge_or_special`
- `unresolved`
- `conflicting`
Required fields inside `tax_amount_basis`：
- `basis_type`; allowed values: `tax_included`, `tax_excluded`, `explicit_tax_amount`, `no_tax`
- `gross_amount`
- `net_amount`
- `tax_amount`
- `tax_rate_ref`
Allowed `tax_control_treatment` values：
- `none`
- `HST_GST_Receivable`
- `HST_GST_Payable`
- `approved_special_tax_control`
Optional fields：
- `jurisdiction`
- `tax_config_ref`
- `receipt_tax_refs`
- `tax_explanation`
- `tax_review_requirement`
Validation / rejection rules：
- `tax_treatment_status = unresolved` 或 `conflicting` 时不得生成 JE。
- `tax_amount_basis` 的 `gross_amount` 必须与 `transaction_basis.amount_abs` 一致，除非 `complex_treatment_basis` 明确解释 split / partial-tax / foreign-currency basis。
- `net_amount + tax_amount` 必须等于 `gross_amount`，允许的 rounding policy 留给后续 shared contract；本节点不能静默吸收 material difference。
- `tax_control_treatment = HST_GST_Receivable` 或 `HST_GST_Payable` 时，必须能追溯到 approved tax config / accountant decision / approved rule。
- 本节点不能自行判断 taxable、exempt、zero-rated、ITC eligibility、tax rate 或 tax control account。
Runtime-only vs durable references：
- Tax basis 是 runtime handoff + durable tax config / evidence references。
- 本节点不能修改 tax config，也不能把 tax calculation outcome 写成长期 tax policy。

#### `finalization_authority_basis`

Required fields：
- `finalization_status`
- `authority_source_type`
- `authority_refs`
- `review_requirement_status`
- `governance_block_status`
Allowed `finalization_status` values：
- `finalizable`
- `not_finalizable`
- `requires_review`
- `requires_pending_clarification`
- `governance_blocked`
- `invalid`
Allowed `authority_source_type` values：
- `structural_authority`
- `approved_rule_authority`
- `case_operational_authority`
- `accountant_review_approval`
- `accountant_correction`
- `resolved_pending_context`
Allowed `review_requirement_status` values：
- `not_required_by_current_authority`
- `already_approved`
- `required_not_completed`
- `rejected`
- `ambiguous`
Allowed `governance_block_status` values：
- `none`
- `active_block`
- `approval_required`
- `conflict_unresolved`
Optional fields：
- `automation_policy_snapshot`
- `rule_lifecycle_snapshot`
- `review_decision_ref`
- `intervention_resolution_ref`
- `governance_constraint_refs`
- `authority_notes`
Validation / rejection rules：
- `finalization_status` 必须为 `finalizable` 才能生成 JE。
- `review_requirement_status = required_not_completed`、`rejected` 或 `ambiguous` 时不得生成 JE。
- `governance_block_status != none` 时不得生成 JE。
- `case_operational_authority` 不能自动等同 accountant approval；是否可进入 JE 必须由 upstream authority context 明确表达。
- 模糊 accountant answer 不能作为 `accountant_review_approval` 或 `accountant_correction`。
Runtime-only vs durable references：
- Authority envelope 是 runtime-only。
- Accountant decision、rule approval、governance event 等 refs 指向 durable authority source；本节点不创建或修改它们。

#### `audit_support_context`

Required fields：
- `transaction_id`
- `source_evidence_refs`
- `upstream_path_trace`
- `authority_trace_refs`
Optional fields：
- `entity_refs`
- `rule_refs`
- `case_refs`
- `review_refs`
- `intervention_refs`
- `governance_refs`
- `policy_trace_refs`
- `audit_note_seed`
Validation / rejection rules：
- `audit_support_context.transaction_id` 必须等于 `transaction_basis.transaction_id`。
- `source_evidence_refs` 不能为空，且必须覆盖生成 outcome 所依赖的关键 evidence。
- `upstream_path_trace` 只能说明路径，不得成为新的 accounting authority。
- 不能把 `Transaction Log` history 作为当前 JE authority trace。
Runtime-only vs durable references：
- 该对象是 runtime audit handoff。
- 它可被 `Transaction Logging Node` 用于 durable audit record，但本节点不写 durable log。

#### `complex_treatment_basis`

Required when applicable：
- `complex_treatment_type`
- `approved_line_basis_refs`
- `amount_allocation_basis`
- `authority_refs`
Allowed `complex_treatment_type` values：
- `split`
- `refund`
- `reversal`
- `multi_line`
- `partial_tax`
- `foreign_currency`
- `special_tax`
Optional fields：
- `original_transaction_ref`
- `fx_rate_basis`
- `allocation_notes`
- `rounding_basis`
Validation / rejection rules：
- 如果当前交易显然需要复杂处理，但缺少 approved `complex_treatment_basis`，不得生成 JE。
- `refund` / `reversal` 必须能引用原始交易或 approved reversal basis；不能仅凭负数金额推断。
- `foreign_currency` 必须有 approved currency / FX basis；本节点不能自行选择汇率。
- `split` / `partial_tax` 必须有 approved amount allocation；本节点不能用 LLM 或 fallback 比例补齐。
Runtime-only vs durable references：
- `complex_treatment_basis` 是 runtime handoff。
- 其中的 references 可指向 durable transaction、review、rule 或 evidence records；本节点不修改这些 records。

### Output Contracts

#### `je_generation_result`

Required fields：
- `transaction_id`
- `status`
- `result_reason`
- `memory_write_effect`
Allowed `status` values：
- `journal_entry_generated`
- `je_blocked`
- `consistency_issue`
- `invalid_input`
Conditionally required fields：
- `journal_entry_result`：当 `status = journal_entry_generated` 时 required。
- `je_blocked_handoff`：当 `status = je_blocked` 时 required。
- `consistency_issue_signal`：当 `status = consistency_issue` 时 required。
- `invalid_input_errors`：当 `status = invalid_input` 时 required。
Optional fields：
- `candidate_signals`
- `audit_support_handoff`
- `runtime_trace`
Allowed `memory_write_effect` value：
- `none`
Validation rules：
- `transaction_id` 必须等于 input transaction_id。
- `journal_entry_generated` 必须有 balanced `journal_entry_result`。
- `je_blocked` 和 `consistency_issue` 不得同时携带 finalized journal entry。
- `candidate_signals` 不能改变当前 status，也不能作为 JE authority。
- `memory_write_effect` 必须为 `none`。
Durable memory effect：
- runtime-only output。
- 本节点不写 `Transaction Log`；下游可记录其结果作为 audit input。

#### `journal_entry_result`

Required fields：
- `journal_entry_id`
- `transaction_id`
- `entry_date`
- `currency`
- `source_outcome_ref`
- `finalization_authority_refs`
- `je_lines`
- `debit_total`
- `credit_total`
- `balance_status`
- `tax_summary`
Allowed `balance_status` values：
- `balanced`
Validation rules：
- `debit_total` 必须等于 `credit_total`。
- `balance_status` 只能在实际借贷平衡时为 `balanced`。
- `entry_date` 默认应等于 `transaction_basis.transaction_date`，除非上游 approved accounting treatment 明确另有 posting date basis。
- `source_outcome_ref` 必须绑定到 `finalizable_accounting_outcome.outcome_id`。
- `finalization_authority_refs` 不能为空。
- `je_lines` 至少两行；每行只能有 debit 或 credit 一边有金额。
Durable memory effect：
- runtime-only JE result。
- 只有 Transaction Logging Node 或后续 output / accounting export flow 可以消费；本节点不落盘。

#### `je_line`

Required fields per line：
- `line_id`
- `account_ref`
- `debit_credit`; allowed values: `debit`, `credit`。
- `amount`
- `line_role`
- `source_basis_ref`
Allowed `line_role` values：
- `bank_control`
- `primary_accounting`
- `tax_control`
- `receivable_or_payable`
- `clearing_or_special`
Optional fields per line：
- `account_display_name`
- `tax_component`
- `allocation_ref`
- `line_note`
Validation rules：
- `amount` 必须为正数。
- `account_ref` 必须来自 approved account basis；不能是 free-text account name。
- `tax_control` line 必须对应 `tax_treatment_basis.tax_control_treatment`。
- `clearing_or_special` line 只能在 upstream approved `complex_treatment_basis` 或 approved special accounting outcome 中出现。
- 本节点不能为了平衡分录而发明 `clearing_or_special` line。
Durable memory effect：
- JE line 是 runtime computed output。
- 它可以被 audit logging 或 export 保存，但本节点不直接写 durable store。

#### `tax_summary`

Required fields：
- `tax_treatment_status`
- `tax_control_treatment`
- `gross_amount`
- `net_amount`
- `tax_amount`
- `tax_line_ids`
Optional fields：
- `tax_rate_ref`
- `tax_config_ref`
- `rounding_note`
Validation rules：
- `tax_summary` 必须与 input `tax_treatment_basis` 一致。
- `tax_line_ids` 必须引用实际存在的 `je_lines`；`tax_control_treatment = none` 时应为空。
- `tax_amount = 0` 时不得生成非零 tax control line。
- `HST_GST_Receivable` / `HST_GST_Payable` 必须分别对应 approved tax control account basis。
Durable memory effect：
- runtime-only calculation summary。
- 不能成为 future tax policy 或 rule source。

#### `je_blocked_handoff`

Required fields：
- `blocked_reason`
- `blocking_basis`
- `required_resolution_path`
- `why_je_generation_cannot_override`
Allowed `blocked_reason` values：
- `outcome_not_finalizable`
- `authority_insufficient`
- `review_required_unresolved`
- `governance_block`
- `missing_account_basis`
- `conflicting_account_basis`
- `missing_tax_basis`
- `conflicting_tax_basis`
- `missing_amount_direction_basis`
- `unsupported_complex_treatment`
- `traceability_missing`
Allowed `required_resolution_path` values：
- `upstream_rework`
- `coordinator_pending`
- `review_node`
- `governance_review_node`
- `tax_or_coa_configuration_review`
Optional fields：
- `missing_fields`
- `conflicting_refs`
- `suggested_review_focus`
- `candidate_signal_refs`
Validation rules：
- Blocked handoff 不能包含 generated JE。
- 如果原因是 authority / governance，不得伪装成 ordinary missing field。
- `suggested_review_focus` 只能聚焦问题，不能替 accountant 作答。
Durable memory effect：
- runtime-only handoff。
- 它可以被后续记录为 audit / intervention context，但本节点不写 log。

#### `consistency_issue_signal`

Required fields：
- `issue_type`
- `issue_summary`
- `affected_refs`
- `required_resolution_path`
Allowed `issue_type` values：
- `debit_credit_not_balanced`
- `gross_net_tax_mismatch`
- `direction_vs_entry_conflict`
- `account_basis_conflict`
- `tax_basis_conflict`
- `authority_trace_conflict`
- `transaction_binding_conflict`
Optional fields：
- `calculation_snapshot`
- `candidate_signals`
- `review_note`
Validation rules：
- Consistency issue 不能自行修正 upstream outcome。
- 不能用 adjustment line、rounding line、fallback account 或 fallback tax treatment 消除 material conflict，除非这些处理已由 upstream authority 明确批准。
- `calculation_snapshot` 只能解释问题，不能作为 durable accounting result。
Durable memory effect：
- runtime-only issue signal。
- 若暗示长期配置问题，只能形成 candidate signal，不能直接写 memory。

#### `candidate_signals`

Required fields per signal：
- `signal_type`
- `signal_reason`
- `source_refs`
- `requires_authority_path`
Allowed `signal_type` values：
- `account_mapping_review_candidate`
- `tax_configuration_review_candidate`
- `profile_structural_issue_candidate`
- `rule_condition_gap_candidate`
- `governance_review_candidate`
- `upstream_contract_gap_candidate`
Allowed `requires_authority_path` values：
- `review_node`
- `governance_review_node`
- `profile_or_coa_owner_workflow`
- `tax_config_owner_workflow`
- `rule_governance_workflow`
Optional fields：
- `related_account_refs`
- `related_tax_refs`
- `related_rule_refs`
- `related_profile_refs`
- `suggested_review_focus`
Validation rules：
- signal 不能包含 `approval_status = approved`。
- signal 不能改变 `Profile`、COA、tax config、Rule Log、Entity Log、Case Log 或 Governance Log。
- signal 不能支持当前交易 JE generation。
Durable memory effect：
- candidate-only。
- 长期化只能通过 accountant / governance approval 或对应 owner workflow。

### Field Authority and Memory Boundary

#### Source of truth for important fields

- `transaction_id`：Transaction Identity Node / transaction identity layer。
- `amount_abs`、`direction`、`transaction_date`、`bank_account`、`currency`：Evidence Intake / Preprocessing 后的 transaction record；`direction` 是方向 source of truth，金额保持 absolute value。
- `evidence_refs`：Evidence Log。
- `bank_account_control_account_ref`：Profile / approved COA account mapping。
- `primary_account_ref`：approved accounting outcome、approved rule、accountant correction、structural mapping 或 approved COA context。
- `tax_treatment_status`、`tax_rate_ref`、`tax_control_treatment`：approved accounting outcome、customer tax config、approved rule、structural logic 或 accountant decision。
- `HST_GST_Receivable` / `HST_GST_Payable`：当前 canonical GST/HST control-account treatment names；实际 account ref 必须来自 approved COA / tax config mapping。
- `finalization_authority_basis`：upstream authority trace、review decision、rule approval、governance restriction 或 accountant correction。
- `journal_entry_result`：本节点 runtime computation output。
- `Transaction Log`：audit-facing final log；只写和查询，不参与本节点 runtime decision。

#### Fields that can never become durable memory by this node

以下字段或对象不能由本节点直接成为 durable memory：
- `je_generation_request`
- `je_generation_result`
- `journal_entry_result`
- `je_lines`
- `tax_summary`
- `je_blocked_handoff`
- `consistency_issue_signal`
- `candidate_signals`
- `runtime_trace`
- `calculation_snapshot`
- 任何 LLM / natural-language explanation

#### Fields that can become durable only after accountant / governance approval

以下信息只有经过对应 authority path 才可能长期化：
- account mapping change：必须经 accountant / COA owner workflow / governance approval。
- tax config or control-account treatment change：必须经 accountant / tax config owner workflow / governance approval。
- rule condition gap or rule correction：必须经 accountant / governance approval 后才可改变 `Rule Log`。
- Profile structural issue：必须经 Profile owner workflow 和必要 accountant confirmation。
- current transaction final audit record：只能由 `Transaction Logging Node` 写入 `Transaction Log`。
- completed case memory：只能由 `Case Memory Update Node` 在完成交易且具备 learning authority 后写入 `Case Log`。

#### Audit vs learning / logging boundary

`journal_entry_result` 可以成为 audit-support input，但它不是 durable audit log。

### Validation Rules

#### Contract-level validation rules

- 每个 input / output 必须绑定同一个 `transaction_id`。
- 只有 `outcome_status = finalizable` 且 `finalization_status = finalizable` 的 request 可以输出 `journal_entry_generated`。
- `review_requirement_status`、`governance_block_status` 和 effective governance constraints 不能被 JE computation 覆盖。
- 金额必须使用 `amount_abs + direction` 语义，不能混用 signed amount。
- JE output 必须借贷平衡。
- Tax line 必须与 approved tax basis 一致。
- Account refs 必须来自 approved account basis；不能使用 free-text fallback。
- `Transaction Log` 不能作为 runtime JE decision source。
- `memory_write_effect` 必须为 `none`。

#### Conditions that make the input invalid

- 缺少 `transaction_id`。
- 缺少 `amount_abs`、`direction`、`currency`、`bank_account` 或必要 evidence refs。
- `amount_abs <= 0`，且没有上游 approved zero / reversal treatment。
- `direction` 不是 `inflow` 或 `outflow`。
- 缺少 `finalizable_accounting_outcome.outcome_status`。
- 缺少 approved `primary_account_ref` 或 bank account control account mapping。
- tax basis unresolved / conflicting。
- request 要求本节点选择 COA、判断 tax treatment、approve review、写 Transaction Log 或修改 long-term memory。
- 把 candidate signal、review draft、pending context、governance candidate 或 Transaction Log history 当作 finalization authority。

#### Conditions that make the output invalid

- `status = journal_entry_generated` 但缺少 `journal_entry_result`。
- `journal_entry_result` 借贷不平衡。
- JE line 使用 free-text account name 或未授权 account ref。
- Tax line 与 approved tax treatment 不一致。
- JE result 在 pending、review-required、not-approved、governance-blocked 或 conflicting context 下生成。
- `je_blocked_handoff` 同时携带 finalized JE。
- candidate signal 声称已 approved、confirmed、promoted、applied 或 written。
- 输出声称本节点已写入 `Transaction Log`、`Case Log`、`Rule Log`、`Entity Log`、`Governance Log`、`Knowledge Log`、Profile、COA 或 tax config。

#### Stop / ask conditions for unresolved contract authority

后续 Stage 或实现中如遇到以下情况，应 stop and ask，不得自行补 product authority：
- 需要决定哪些 high-confidence structural / rule / case-supported result 可以不经逐笔 accountant review 直接进入 JE。
- 需要冻结全局 shared accounting treatment / JE-ready schema。
- 需要定义 split、refund、reversal、partial-tax、foreign-currency 或 special tax 的完整 algorithm。
- 需要允许 fallback / suspense / clearing account 用于缺失 account 或 tax basis。
- 需要决定 JE-blocked 是否也形成 terminal audit record，以及由哪个节点写。
- 需要改变 `Transaction Log` 是否参与 runtime decision。
- 需要允许 JE Generation 写 long-term memory、Profile、COA、tax config、Governance Log 或 Transaction Log。

## 4. Open Boundaries

### Stage 1: Open Boundaries

以下问题留到后续阶段，不在 Stage 1 冻结：
- 哪些 upstream outcome 可以被视为 `finalizable current transaction outcome`。
- `Review Node` 是否覆盖所有 JE 前交易，还是只覆盖 review-required / sampled / governance-candidate items。
- 高可信 structural / rule / case-supported result 是否可以不经逐笔 accountant review 直接触发 JE。
- accountant approval、correction、rejection、still-pending 与 JE Generation 的 exact handoff contract。
- JE-blocked reason 应返回哪个上游 workflow，或进入哪个 review / pending flow。
- split、refund、reversal、multi-line、tax-included / tax-excluded、zero-tax、partial-tax 等复杂分录类别的精确边界。
- exact input / output schema、字段名、对象结构、routing enum、执行算法、测试矩阵和 coding-agent task contract。

### Stage 2: Stage 2 Open Boundaries

以下内容留到后续阶段，不在 Stage 2 冻结：
- exact input / output schema
- field names and object shape
- exact routing enum
- threshold numbers
- storage paths
- execution algorithm
- test matrix
- coding-agent task contract
- exact prompt structure or tool interface
- 哪些 upstream outcome 可被视为 `finalizable accounting outcome`
- Review Node 是否覆盖所有 JE 前交易，还是只覆盖 review-required / sampled / governance-candidate items
- high-confidence structural / rule / case-supported result 是否可以不经逐笔 review 直接触发 JE
- accountant approval、correction、still-pending 与 JE Generation 的 exact handoff contract
- JE-blocked reason 与 Coordinator / Review / upstream rework / Governance Review 的 exact routing
- split、refund、reversal、multi-line、tax-included / tax-excluded、zero-tax、partial-tax、foreign currency 等复杂分录类别的 exact behavior
- account-mapping、tax configuration 和 COA governance 的 precise ownership split

### Stage 3: Open Contract Boundaries

- 哪些 high-confidence structural / rule / case-supported result 可以不经逐笔 accountant review 直接进入 JE，当前仍未由 active docs 冻结。本文件只要求 upstream 明确提供 `finalization_authority_basis`。
- `approved_accounting_treatment` / `accounting_treatment_ref` 的全局 shared schema 仍需与 Rule Match、Case Judgment、Review、Transaction Logging 后续 contracts 对齐。
- split、refund、reversal、multi-line、partial-tax、foreign-currency、special tax 的完整 JE algorithm 和 rounding policy 尚未冻结；本文件只定义这类处理需要 approved `complex_treatment_basis`。
- JE-blocked handoff 的 exact routing 尚未冻结：可能回到 Coordinator、Review、upstream rework、tax / COA configuration review 或 Governance Review。
- JE-blocked / not-finalizable 是否需要 terminal audit record，以及是否由 Transaction Logging Node 记录，留给后续阶段。
- Canonical `journal_entry_id` 格式、line id 格式、amount decimal precision 和 rounding tolerance 尚未由 active docs 冻结。
- `tax_control_treatment` 的 account-role enum 是否应成为全局 tax contract，尚未最终决定。
