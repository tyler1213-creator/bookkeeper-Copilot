# Case Judgment Node — 设计摘要

> 由原 Stage 1/2/3 合并瘦身而来；保留定位、边界、contract 字段和开放问题，删除阶段说明、examples、self-review、历史读档记录和重复解释。

## 1. 定位与职责

Stage 1 已确定以下内容：
- 这个节点是 `case-based runtime judgment gate`。
- 它在结构性确定处理和 rule matching 都不能完成交易之后运行。
- 它使用当前 evidence、entity/profile context、case memory 和 customer knowledge 做 runtime judgment。
- 它判断交易可以进入 operational classification，还是必须进入 pending / review。
- 它不替代上游 matching、identity 或 rule responsibilities。
- 它不直接向 accountant 提问。
- 它不生成 journal entries。
- 它不写 `Transaction Log`。
- 它不直接修改长期 entity、case、rule 或 governance memory。
- 它不批准 governance-level changes。
- 它的下游影响是支持 operational classification、pending handling、review、audit trail formation 和后续 memory/governance workflows，但不拥有这些下游职责。

## 2. 逻辑与边界

### Trigger Boundary

`Case Judgment Node` 在以下 conceptual boundary 下触发：
- 当前交易不是已经由结构性确定路径完成的交易；并且
- deterministic path 未能完成当前交易。
因此，该节点接收所有 `非结构性交易 + deterministic path 未完成` 的情况，包括：
- 没有适用 deterministic rule
- rule path 因 automation authority、role、alias、entity、governance 或 review 要求而不能完成
- rule match 结果本身说明当前交易不能通过 deterministic path 自动处理

### Input Categories

Stage 2 按判断作用组织 input categories，并在概念层说明来源类别。这里不是字段清单。

#### Identity basis

说明交易当前被认为指向谁。

关键判断不是读取了哪个字段，而是：
- entity 是否足够支持当前 runtime judgment
- alias / role 是否足以支持自动化
- 当前 identity basis 是否允许 case-based judgment

#### Structural / profile basis

说明这笔交易是否受客户结构事实影响，例如 internal transfer、loan、tax config、employee context 等。

#### Case precedent basis

说明过去类似案例怎样处理，以及哪些条件下存在例外。

#### Current evidence basis

说明本次交易有哪些证据支持或推翻历史先例。

#### Risk and blocking basis

说明当前是否存在 mixed-use、unconfirmed role、new entity candidate、ambiguous / unresolved identity、policy block、rule block、recent intervention、governance restriction 等风险或阻断原因。

#### Authority basis

说明当前最多允许哪种层级的 runtime action：
- operational classification path
- pending / review path
- candidate signal
- governance candidate

### Output Categories

Stage 2 只定义 conceptual output categories，不冻结 routing enum。

#### Operational classification path

含义：当前 authority 允许 case-based automation，且 case precedent + current evidence 足够支持本笔交易进入 operational classification。

边界：
- 这是当前交易的 runtime path。
- 它不等于长期 case 写入。
- 它不等于 entity / alias / role approval。
- 它不等于 rule promotion。
- 它不等于 governance approval。

#### Pending path

含义：当前交易可能可判断，但缺少关键 accountant information、evidence clarification 或 context confirmation。

边界：
- Pending 是为了补信息。
- Pending 本身不是治理批准流程。
- Case Judgment 不应在可补证据缺失时自行猜测。

#### Review-required path

含义：当前问题不是单纯缺信息，而是 automation policy、required role confirmation、ambiguous / unresolved identity、rule-required condition、review-required condition、disabled automation、governance block 等 authority 原因要求人工确认或治理处理。

边界：
- Case Judgment 不能把 review-required path 降级成 operational classification path。
- Review-required 可能进入 Review Node、Governance Review Node 或 pending-review flow，但具体 routing 留到后续阶段。

#### Candidate signals

含义：Case Judgment 可以指出后续可能需要处理的候选信号，例如：
- case memory update candidate
- entity / alias / role candidate
- rule candidate
- automation policy / governance candidate
边界：
- Candidate signals 只供后续节点评估。
- 它们本身不写长期 memory。
- 它们不批准 governance。
- 它们不改变 active rule 或 automation policy。

### Deterministic Code vs LLM Semantic Judgment

Stage 2 的核心边界是：

#### Deterministic code responsibility

Deterministic code 负责：
- 判断 Case Judgment 是否被触发：非结构性交易，且 deterministic path 未完成。
- 汇总并标记上游 context：profile / structural result、entity resolution result、rule match result、blocking reason、automation policy、known governance constraints。
- 判断 authority / eligibility 上限：当前是否允许 operational classification，还是只能 pending、review-required 或 candidate signal。
- 执行 hard blocks：ambiguous / unresolved identity、rejected alias、required but unconfirmed role、review-required policy、disabled automation、rule-required condition without approved rule、governance block 等不能被 LLM 语义理由绕过。
- 对 `new_entity_candidate` 执行受限判断：它不能支持 rule match 或 durable memory authority；只有在 authority 允许且 current evidence 足够强时，才可能进入 operational classification path，否则进入 pending 或 review-required。
- 限制 memory / governance actions：哪些只能作为 candidate，哪些绝不能直接写入或批准。

#### LLM semantic judgment responsibility

LLM 可以负责：
- 比较 case precedent 与 current evidence 的语义匹配度。
- 判断当前证据是否足以支持本笔交易的 operational judgment。
- 解释历史先例的适用条件、例外条件，以及当前交易是否落在例外里。
- 生成 rationale、pending reason、review explanation、candidate signal explanation。

#### Hard boundary

LLM 可以在 code 允许的边界内判断“证据是否足够”。

LLM 不能：
- 扩大 automation authority
- 把 blocked reason 当作软建议
- 绕过 hard block
- 把 `new_entity_candidate` 变成 stable entity
- 批准长期记忆变化
- 批准治理变化

### Accountant Authority Boundary

Accountant 仍然拥有最终 accounting decision 和 durable authority。

Case Judgment 不能：
- 替 accountant 作最终 review approval
- 把 runtime operational judgment 变成 accountant-confirmed outcome
- 确认 role、alias、entity、rule 或 automation policy
- 把一次 runtime result 当作长期客户知识

### Governance Authority Boundary

Governance-level changes 不属于 Case Judgment authority。

Case Judgment 不能：
- approve / reject alias
- confirm role
- merge / split entity
- promote / modify / delete / downgrade active rule
- upgrade or relax automation policy
- approve governance event
- invalidate durable memory

### Memory / Log Boundary

Stage 2 采用三层边界：read、candidate-only、no direct mutation。

#### Read boundary

Case Judgment 可以读取或消费以下 conceptual context：
- Current Evidence / Evidence Log references：当前交易证据、receipt、cheque、accountant context 等。
- Profile / Structural context：结构性事实和结构性匹配结果。
- Entity context / Entity Log：已识别实体、candidate entity、alias / role 状态、automation policy、identity risk。
- Case Log / customer knowledge summary：历史案例、例外条件、客户专属知识摘要。
- Rule / blocking context / Rule Log：rule miss、rule blocked、deterministic path 未完成原因。
- Intervention / Governance context：近期人工干预、治理风险、已有治理限制。

#### Candidate-only boundary

Case Judgment 可以提出候选信号，但不能执行变化：
- case memory update candidate：本次处理之后可能值得沉淀为 case。
- entity / alias / role candidate：可能需要后续确认或治理。
- rule candidate：可能存在 rule promotion 或 rule review 价值。
- automation policy / governance candidate：可能需要降级、review 或治理动作。

#### No direct mutation boundary

Case Judgment 绝不能：
- 直接写 Entity Log、Case Log、Rule Log 或 Governance Log
- 直接 append / update / promote / invalidate 长期 memory
- 修改 active rule
- 批准 alias、role、entity、rule 或 automation policy
- 执行 entity merge / split
- 把 runtime judgment 当作长期 authority

### Insufficient / Ambiguous / Conflicting Evidence Behavior

Stage 2 采用优先级边界：authority block first、evidence repairability second、conflict caution third。

#### Authority block first

如果存在 policy、governance 或 identity hard block，Case Judgment 不能把它转成 operational classification。

例如：
- policy 要求 approved rule，但当前没有可用 approved rule
- policy 要求 review
- automation disabled
- required role 未确认且不能支持自动化
- alias 被拒绝或不能作为当前 identity basis
- ambiguous / unresolved entity 不能安全归属
- `new_entity_candidate` 缺少足够 current evidence 或 authority support
- governance block 存在

#### Evidence repairability second

如果 authority 允许 case-based judgment，但缺少关键证据或上下文，而 accountant 可以补充信息解决，则进入 pending path。

例如：
- receipt 缺失
- 用途不明
- 当前证据不足以判断历史先例是否适用
- `new_entity_candidate` 需要 accountant context 才能支持本笔判断
- accountant context 缺失

#### Conflict caution third

如果 current evidence 与 case precedent、customer knowledge、risk signal 或 policy context 发生冲突，Case Judgment 应保守处理。

#### Hard boundary

- Ambiguity 不等于可以猜。
- Conflicting evidence 不等于可以选择一个看起来更合理的版本。
- `new_entity_candidate` 不天然阻断当前交易分类，但也不提供 durable authority。
- Case Judgment 应输出最安全的 runtime path 和解释，而不是把不确定性隐藏在 confidence 语言里。

## 3. Contract 字段摘要

### Contract Position in Workflow

#### Upstream handoff consumed

`Case Judgment Node` 消费的是 `deterministic path 未完成` 后的 runtime handoff。

上游必须已经完成或明确未完成：
- `Evidence Intake / Preprocessing Node`
- `Transaction Identity Node`
- `Profile / Structural Match Node`
- `Entity Resolution Node`
- `Rule Match Node`

#### Downstream handoff produced

`Case Judgment Node` 输出 `case_judgment_result`。

下游消费方向包括：
- `Coordinator / Pending Node`：消费 `pending` reason、missing information 和 focused question context。
- `Review Node`：消费 runtime recommendation、rationale、review reason 和 candidate signals。
- `JE Generation Node`：只能在后续 authority 允许且 classification outcome 已达到该节点要求时消费 accounting treatment，不直接把 Case Judgment 当成 final approval。
- `Transaction Logging Node`：后续可把结果和审计轨迹写入 `Transaction Log`，但 Case Judgment 本身不写。
- `Case Memory Update Node`：只可在后续确认后消费 candidate signals 或 final outcome；不能把 runtime judgment 直接当作 durable case。
- `Governance Review Node`：只可消费 governance candidate signal；不能把该节点输出当作 approval。

#### Logs / memory stores read

该节点可读取或消费以下 runtime context / durable references：
- `Evidence Log` references：原始证据、小票、支票、accountant context 等证据引用。
- `Profile` / structural context：客户结构事实和上游 structural result。
- `Entity Log` context：entity、alias、role、status、authority、automation_policy、identity risk。
- `Case Log` / case memory pack：历史案例、最终分类、证据、例外和 accountant correction。
- `Rule Log` / rule path context：rule miss、blocked rule、rule lifecycle、rule-required condition。
- `Intervention Log` references：近期 accountant 介入、回答、修正、确认。
- `Governance Log` references：未解决或已生效的 governance restriction。
- `Knowledge Log / Summary Log`：由 entity、case、rule、governance 历史编译出的客户知识摘要；不作为 deterministic rule source。

#### Logs / memory stores written or candidate-only

`Case Judgment Node` 不直接写入任何 durable log / memory store。

它只能输出 candidate-only signals：
- `case_memory_update_candidate`
- `entity_candidate_signal`
- `alias_candidate_signal`
- `role_candidate_signal`
- `rule_candidate_signal`
- `automation_policy_or_governance_candidate`

### Input Contracts

#### `case_judgment_input`

**Required fields**

- `transaction_context`
- `deterministic_path_context`
- `entity_context`
- `case_memory_context`
- `current_evidence_context`
- `authority_context`

**Optional fields**

- `profile_structural_context`
- `rule_context`
- `intervention_context`
- `governance_context`
- `knowledge_summary_context`
- `runtime_notes`

#### `transaction_context`

**Required fields**

- `transaction_id`：稳定交易 ID。
- `date`：交易日期。
- `amount`：绝对金额。
- `direction`：资金方向；必须沿用共享交易 contract 的 direction 语义。
- `bank_account`：客户侧银行账户引用。
- `evidence_refs`：本笔交易可追溯证据引用列表。

**Optional fields**

- `raw_description`
- `description`
- `receipt_ref`
- `cheque_ref`
- `counterparty_surface_text`
- `import_batch_id`

#### `deterministic_path_context`

**Required fields**

- `path_completed`：必须为 `false` 才能进入本节点。
- `trigger_reason`
- `blocked_reasons`
- `structural_result_status`
- `rule_match_status`

**Optional fields**

- `matched_rule_id`
- `blocked_rule_ids`
- `rule_required_reason`
- `profile_structural_reason`
- `deterministic_trace_refs`

**Allowed values**

`trigger_reason` 允许：
- `no_applicable_rule`
- `rule_not_applicable`
- `rule_blocked_by_authority`
- `rule_required_but_missing`
- `review_required_by_policy`
- `deterministic_path_insufficient`
`rule_match_status` 允许：
- `not_attempted`
- `no_match`
- `matched_but_not_applicable`
- `blocked`
- `requires_review`

#### `entity_context`

**Required fields**

- `status`
- `confidence`
- `reason`
- `evidence_used`

**Optional fields**

- `entity_id`
- `display_name`
- `matched_alias`
- `alias_status`
- `candidate_role`
- `automation_policy`
- `entity_status`
- `blocking_reason`
- `candidate_entities`
- `risk_flags`

**Allowed values**

`status` 必须是 active baseline 已定义的 entity resolution status：
- `resolved_entity`
- `resolved_entity_with_unconfirmed_role`
- `new_entity_candidate`
- `ambiguous_entity_candidates`
- `unresolved`
`alias_status` 如存在，必须是：
- `candidate_alias`
- `approved_alias`
- `rejected_alias`
`automation_policy` 如存在，必须是：
- `eligible`
- `case_allowed_but_no_promotion`
- `rule_required`
- `review_required`
- `disabled`

#### `case_memory_context`

**Required fields**

- `case_refs`
- `case_summary`
- `precedent_fit_basis`

**Optional fields**

- `similar_case_count`
- `exception_patterns`
- `accountant_corrections`
- `negative_cases`
- `case_freshness`
- `source_summary_ref`

#### `current_evidence_context`

**Required fields**

- `available_evidence`
- `evidence_strength_summary`

**Optional fields**

- `receipt_details_summary`
- `cheque_details_summary`
- `accountant_context_refs`
- `missing_evidence`
- `conflicting_evidence`
- `mixed_use_indicators`
- `amount_or_context_anomaly`

#### `authority_context`

**Required fields**

- `max_allowed_action`
- `hard_blocks`
- `candidate_signal_allowed`

**Optional fields**

- `requires_accountant_confirmation`
- `requires_governance_review`
- `authority_reason`
- `policy_refs`

**Allowed values**

`max_allowed_action` 允许：
- `operational_classification_allowed`
- `pending_only`
- `review_required_only`
- `candidate_signal_only`

### Output Contracts

#### `case_judgment_result`

**Required fields**

- `transaction_id`
- `judgment_status`
- `authority_used`
- `rationale`
- `evidence_refs_used`
- `case_refs_used`
- `memory_write_effect`

**Optional fields**

- `accounting_treatment_proposal`
- `pending_request_context`
- `review_required_context`
- `candidate_signals`
- `confidence_label`
- `risk_flags`
- `blocked_reasons`
- `trace_summary`

**Allowed statuses**

`judgment_status` 允许：
- `operational_classification`
- `pending`
- `review_required`

#### `accounting_treatment_proposal`

**Required fields**

- `classification_basis`
- `proposed_accounting_treatment`
- `case_support_summary`
- `current_evidence_support_summary`

**Optional fields**

- `coa_account_ref`
- `hst_gst_treatment_ref`
- `business_use_context`
- `exception_handling_note`
- `confidence_label`

**Allowed values**

`classification_basis` 允许：
- `case_supported`
- `current_evidence_supported_new_entity`
- `case_supported_with_current_evidence_override`
`confidence_label` 如存在，允许：
- `high`
- `medium`
- `low`

#### `pending_request_context`

**Required fields**

- `pending_reason`
- `missing_or_unclear_information`
- `question_focus`
- `why_not_auto_classified`

**Optional fields**

- `suggested_evidence_to_request`
- `candidate_answer_options`
- `case_context_for_question`
- `review_caution`

**Allowed values**

`pending_reason` 允许：
- `missing_receipt_or_support`
- `unclear_business_purpose`
- `insufficient_current_evidence`
- `needs_accountant_context`
- `new_entity_needs_context`
- `case_precedent_fit_unclear`

#### `review_required_context`

**Required fields**

- `review_reason`
- `blocking_basis`
- `why_case_judgment_cannot_override`

**Optional fields**

- `governance_candidate_hint`
- `identity_risk_summary`
- `policy_refs`
- `conflict_summary`
- `suggested_review_focus`

**Allowed values**

`review_reason` 允许：
- `automation_policy_review_required`
- `automation_disabled`
- `rule_required_without_rule`
- `unconfirmed_required_role`
- `rejected_or_unsafe_alias`
- `ambiguous_or_unresolved_identity`
- `governance_block`
- `conflicting_evidence_requires_business_judgment`

#### `candidate_signals`

**Required fields per signal**

- `signal_type`
- `signal_reason`
- `source_evidence_refs`
- `requires_authority`

**Optional fields per signal**

- `related_entity_refs`
- `related_case_refs`
- `related_rule_refs`
- `proposed_candidate_payload`
- `risk_note`

**Allowed signal types**

- `case_memory_update_candidate`
- `entity_candidate_signal`
- `alias_candidate_signal`
- `role_candidate_signal`
- `rule_candidate_signal`
- `automation_policy_or_governance_candidate`

**Allowed `requires_authority` values**

- `case_memory_update_node`
- `accountant_review`
- `governance_review`

### Field Authority and Memory Boundary

#### Source of truth

- `transaction_id`：`Transaction Identity Node` / transaction identity layer。
- `amount`、`direction`、`date`、`bank_account`：preprocessing 后的 transaction record；`direction` 是资金方向 source of truth，金额保持 absolute value。
- `evidence_refs`：`Evidence Log`。
- `entity_id`、approved alias、confirmed role、entity status、automation policy：`Entity Log` 与对应 governance-approved state。
- `case_refs`、historical outcome、accountant correction：`Case Log`。
- active rule、rule lifecycle、rule-required condition：`Rule Log`。
- accountant intervention / answer / correction：`Intervention Log`。
- governance restriction / approval status：`Governance Log`。
- runtime judgment、rationale、pending reason、review reason：`Case Judgment Node` 本次运行输出，仅为 runtime authority。

#### Fields that can never become durable memory by this node

以下字段或对象不能由本节点直接成为 durable memory：
- `case_judgment_result`
- `accounting_treatment_proposal`
- `rationale`
- `confidence_label`
- `pending_request_context`
- `review_required_context`
- `candidate_signals`
- `candidate_role`
- `new_entity_candidate`
- `alias_candidate_signal`
- `rule_candidate_signal`
- 任何 LLM semantic explanation

#### Fields that can become durable only after approval / owner-node handling

以下信息只有通过后续 authority path 才可能进入长期记忆：
- final accountant-confirmed classification：经 review / confirmed workflow 后，才可由 `Case Memory Update Node` 写入 `Case Log`。
- new entity / alias / role：必须经 `Review Node` / `Governance Review Node` / approved entity governance path。
- rule candidate：必须经 accountant / governance approval 后才可写入 `Rule Log`。
- automation policy change：降级可由 lint pass 触发并记录，升级或放宽必须 accountant approval；Case Judgment 只能提出 signal。
- governance event：必须由 governance workflow 创建、审批、记录。

#### Audit vs learning / logging boundary

`Transaction Log` 保存最终处理结果和审计轨迹；它只写和查询，不参与 runtime decision，也不是 learning layer。

### Validation Rules

#### Contract-level validation rules

- 每个输入必须有稳定 `transaction_id`。
- 只有 `deterministic_path_context.path_completed = false` 的交易可以进入本节点。
- `authority_context.max_allowed_action` 是输出上限；LLM rationale 不能扩大它。
- `new_entity_candidate` 可以支持当前 operational classification，但只能在 authority 允许且 current evidence 足够强时；它不能支持 rule match、rule promotion 或 stable entity write。
- `Transaction Log` 不得出现在 runtime decision source 中。
- 所有 durable store references 必须可追溯；不能传入无来源的“历史通常如此”。
- 输出的 `memory_write_effect` 必须为 `none`。

#### Conditions that make the input invalid

- 缺少 `transaction_id`。
- preprocessing / identity 尚未完成。
- deterministic path 已完成。
- 缺少 entity resolution status。
- 缺少 authority context。
- 把 audit-only `Transaction Log` 当作 case memory 或 learning source。
- blocked rule / policy 状态缺少 blocked reason。
- 引用的 evidence、case、entity、rule、intervention 或 governance record 不可追溯。

#### Conditions that make the output invalid

- `operational_classification` 绕过 `review_required`、`disabled`、`rule_required` without rule、governance block、rejected alias 或 unsafe identity。
- `pending` 用于本质上需要 review / governance 的 authority block。
- `review_required` 缺少 blocking basis。
- `accounting_treatment_proposal` 没有 evidence / case support。
- 输出声称已写入 `Entity Log`、`Case Log`、`Rule Log`、`Governance Log` 或 `Transaction Log`。
- candidate signal 声称已 approved / confirmed / promoted / applied。
- `confidence_label = high` 但同时存在 unresolved material conflict 且未进入 review-required。

#### Stop / ask conditions for unresolved contract authority

后续 Stage 或实现中如遇到以下情况，应 stop and ask，不得自行补 product authority：
- 需要冻结全局 router enum 或跨节点状态机。
- 需要定义共享 accounting classification schema / JE-ready schema。
- 需要决定 `operational_classification` 是否可在无 accountant review 下直接进入 JE。
- 需要决定 candidate signal 的 durable queue schema。
- 需要改变 `Transaction Log` 是否参与 runtime decision。
- 需要允许 Case Judgment 直接写 Case Log、Rule Log、Entity Log 或 Governance Log。

## 4. Open Boundaries

### Stage 1: 留到 Stage 2 的问题

以下问题有意不在 Stage 1 解决：
- 哪些 evidence categories 可以支持 operational classification。
- 哪些情况必须进入 pending。
- runtime rationale 与 candidate signals 的边界。
- 这个节点可以读取哪些 logs / memory stores。
- 它可以把哪些 memory 或 governance changes 作为 candidates 提出。
- 哪些 memory 或 governance changes 它绝不能直接写入。
- accountant authority 的具体交接边界。
- governance authority 的具体交接边界。
- insufficient evidence、ambiguous evidence 或 conflicting evidence 下的行为。

### Stage 2: Stage 2 Open Boundaries

以下内容留到后续阶段，不在 Stage 2 冻结：
- exact input / output schema
- field names and object shape
- exact routing enum
- threshold numbers
- memory operation contract
- execution algorithm
- implementation files
- test matrix
- coding-agent task contract
- exact prompt structure or tool interface

### Stage 3: Open Contract Boundaries

- Final router enum / downstream state-machine 名称尚未由 active docs 冻结；本 Stage 3 只冻结 `judgment_status` 的 data contract 类别。
- 共享 `accounting_treatment_proposal` 与未来 JE-ready classification schema 的精确字段尚未冻结；本节点只能输出 proposal / reference，不定义 JE line contract。
- `confidence_label` 是否应成为全局共享枚举，还是仅作为本节点 runtime output，尚未最终决定。
- candidate signals 是否进入独立 durable queue、由哪个后续 node 先归集、以及 queue schema 如何设计，留给后续 Stage / node contract。
- `operational_classification` 在哪些客户或批次设置下可跳过 accountant review 直接进入 JE，未由当前 Stage 冻结；本文件只表达 Case Judgment 的 runtime recommendation authority。
- Timeout、retry、partial-result behavior 仍是 repo active risk，本 Stage 不定义。
