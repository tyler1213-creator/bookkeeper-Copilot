# Coordinator / Pending Node — 设计摘要

> 由原 Stage 1/2/3 合并瘦身而来；保留定位、边界、contract 字段和开放问题，删除阶段说明、examples、self-review、历史读档记录和重复解释。

## 1. 定位与职责

Stage 1 已确定以下内容：
- `Coordinator / Pending Node` 是 runtime clarification and intervention bridge。
- 它在交易因证据、身份、结构、rule、case 或 authority 问题不能安全继续时介入。
- 它的核心职责是向 accountant 提出聚焦问题，并保留回答语境。
- 它不重新执行 evidence intake、transaction identity、structural match、entity resolution、rule match 或 case judgment。
- 它不做最终会计分类、HST/GST 判断或 journal entry generation。
- 它不写 `Transaction Log`。
- 它不直接修改 `Profile`、Entity / Case / Rule / Governance 等长期业务记忆。
- 它可以捕捉 profile、entity、alias、role、rule、case 或 governance 相关候选信号，但不能批准或落地这些变化。
- Accountant authority 和 durable governance changes 必须留给 Review / Governance Review / memory update workflow。

## 2. 逻辑与边界

### Trigger Boundary

`Coordinator / Pending Node` 在当前交易或当前 batch 出现 clarification-needed / pending-needed / review-needed context 时触发。

概念触发条件是：
- 上游 workflow 已经整理出可追溯 evidence、稳定交易身份和当前处理语境；并且
- 至少一个上游节点说明当前交易不能安全继续自动完成；并且
- 该卡点可能通过 accountant clarification、补充 evidence、业务用途说明、身份/角色确认或 review context 得到推进；并且
- 当前问题尚未进入最终 review approval、journal entry generation、transaction logging 或 governance approval。
典型触发来源包括：
- Evidence Intake 暴露 missing evidence、conflicting evidence、unmatched evidence 或低质量 evidence。
- Transaction Identity 暴露 same-transaction / duplicate candidate、identity conflict 或补交 evidence 归属不清。
- Profile / Structural Match 暴露 profile fact 缺失、结构关系未确认或 profile conflict。
- Entity Resolution 暴露 ambiguous entity、unresolved identity、new entity candidate 需要 accountant context、unconfirmed role 或 alias authority 问题。
- Rule Match 暴露 rule blocked、rule ineligible、rule conflict、role / alias / policy authority gap 或 governance-needed context。
- Case Judgment 暴露 evidence insufficiency、case precedent 不适用、current evidence 与历史先例冲突、或判断不稳。

### Input Categories

Stage 2 按判断作用组织 input categories，不列字段清单。

#### Current transaction and evidence context

说明当前 pending 对象是哪笔交易、有哪些 evidence、哪些 objective transaction basis 已确定、哪些 evidence issue 仍存在。

#### Upstream pending / review context

说明上游节点为什么不能继续处理。

#### Accountant clarification basis

说明 accountant 需要补充或确认的内容类别，例如：
- 缺失 evidence 或附件归属
- 当前交易业务用途
- counterparty / vendor / payee 身份
- role / context
- profile structural fact
- rule exception 或 rule conflict
- case precedent 是否适用
- 当前交易是否应人工 review

#### Prior intervention context

说明同一交易、同一 batch 或同一客户近期是否已经发生过 accountant 提问、回答、修正或确认。

#### Authority and policy context

说明当前最多允许哪种后续动作：
- 继续补信息
- 进入 review
- 继续受限 runtime judgment
- 形成 candidate signal
- 进入 governance review

#### Accountant response context

说明 accountant 已经提供的回答、补充 evidence、业务说明、修正或确认。

### Output Categories

Stage 2 只定义 conceptual output categories，不冻结 routing enum 或对象形状。

#### Focused accountant clarification

含义：本节点把上游卡点转化为 accountant 可以回答的聚焦问题。

边界：
- 问题必须服务于已知卡点。
- 问题不应要求 accountant 重新设计系统内部流程。
- 问题不应把未确认候选包装成事实。
- 问题本身不是 accountant decision。

#### Intervention record / answer context

含义：accountant 的提问、回答、修正、确认或补充 context 被保留为 intervention 语境。

边界：
- 这是 accountant interaction 的可追溯记录。
- 它不等于最终会计分类。
- 它不等于 durable memory mutation。
- 它不等于 governance approval。

#### Supplemental evidence / context handoff

含义：如果 accountant 提供新 evidence、附件归属说明或当前交易上下文，本节点可以把它作为后续 workflow 的补充 handoff。

边界：
- 补交原始 evidence 仍必须保持 evidence-first 语义。
- evidence amendment、re-intake 或 reprocessing 的 exact workflow 留到后续阶段。
- 本节点不能用自然语言回答覆盖原始 evidence。

#### Runtime continuation handoff

含义：accountant clarification 可能让交易回到受限的上游或下游 workflow，继续 evidence intake、identity handling、structural match、entity resolution、rule match、case judgment、review 或 completion flow。

边界：
- 继续处理不等于本节点完成分类。
- 继续处理不等于自动批准 review outcome。
- 是否重新触发哪个节点、以什么边界重跑，留到后续阶段。

#### Review / governance candidate handoff

含义：如果 accountant 回答暴露出长期 profile、entity、alias、role、rule、automation policy 或 case-memory 风险，本节点可以形成候选信号交给后续 review / governance workflow。

边界：
- Candidate handoff 只表示“后续应评估”。
- 它不写长期 memory。
- 它不批准治理事件。
- 它不改变 active rule 或 automation policy。

#### Still-pending / unresolved handoff

含义：如果 accountant 没有回答、回答不足、回答模糊、回答冲突或仍缺关键 evidence，本节点应保持 pending / review-needed 语义。

边界：
- 未解决不确定性不能被 confidence 语言掩盖。
- 本节点不能为了推动 workflow 而猜测。
- 下游应看到仍然缺什么、为什么不能继续。

### Deterministic Code vs LLM Semantic Judgment

Stage 2 的核心边界是：

#### Deterministic code responsibility

Deterministic code 负责：
- 判断本节点是否被触发：存在 pending / clarification-needed / review-needed context，且尚未进入最终 approval 或 logging。
- 汇总上游卡点、来源节点、交易身份、evidence references、authority limits 和 prior intervention context。
- 区分 ordinary pending、review-needed、governance-needed、supplemental evidence needed 和 still-unresolved context。
- 保持问题与上游卡点绑定，避免泛化或重复提问。
- 防止 accountant answer 被直接写入 `Profile`、Entity / Case / Rule / Governance memory。
- 防止本节点写入 `Transaction Log`。
- 防止 `Transaction Log` 被用作 runtime pending decision source。
- 记录 accountant interaction 的 intervention 语境，但不把 interaction record 当作 durable business memory approval。
- 标记哪些回答只能作为 candidate，哪些必须交给 Review / Governance Review。
- 执行 hard blocks：policy、governance、unresolved identity、ambiguous entity、unconfirmed role、rule-required condition 或 review-required condition 不能被本节点通过提问自行解除。

#### LLM semantic judgment responsibility

LLM 可以在 code 允许的边界内辅助：
- 把上游 pending reason 转成 accountant 能理解的简洁问题。
- 对多个 pending reasons 做可读合并，避免重复问同一事实。
- 解释 accountant 自然语言回答的可能含义。
- 总结回答是否补足了原卡点，或仍缺哪些信息。
- 生成 review explanation、governance candidate explanation 或 unresolved explanation。

#### Hard boundary

LLM 不能：
- 扩大 automation authority
- 把 accountant 模糊回答当作明确确认
- 把当前交易回答变成长期 profile truth
- approve / reject alias
- confirm role
- create stable entity
- merge / split entity
- promote、modify、delete 或 downgrade active rule
- upgrade or relax automation policy
- 批准 governance event
- 选择 COA / HST 处理来替代上游 judgment
- 生成 journal entry
- 写入 `Transaction Log`

### Accountant Authority Boundary

Accountant 仍然拥有最终 accounting decision 和 durable authority。

本节点不能：
- 替 accountant 作最终 review approval
- 把 accountant 当前交易回答直接变成长期客户政策
- 把 profile change signal 直接写入 `Profile`
- 把 entity / alias / role clarification 直接写成 stable Entity Log authority
- 把 case clarification 直接写成 confirmed Case Log memory
- 把 repeated answer 或当前回答直接升级成 active rule
- 把 accountant clarification 直接变成 governance approval

### Governance Authority Boundary

Governance-level changes 不属于 `Coordinator / Pending Node` authority。

本节点不能：
- approve / reject alias
- confirm role
- create stable entity authority
- merge / split entity
- archive / reactivate entity
- change automation policy
- create / promote / modify / delete / downgrade active rule
- approve governance event
- invalidate durable memory

### Memory / Log Boundary

Stage 2 采用四层边界：read / consume、write allowed、candidate-only、no direct mutation。

#### Read / consume boundary

`Coordinator / Pending Node` 可以读取或消费以下 conceptual context：
- runtime evidence foundation、objective transaction basis 和 stable transaction identity
- upstream pending / review context from Evidence Intake、Transaction Identity、Profile / Structural Match、Entity Resolution、Rule Match 和 Case Judgment
- current evidence issue、identity issue、profile issue、entity ambiguity、rule blocked / miss reason、case judgment rationale 和 risk / policy context
- `Intervention Log` 中与当前交易、batch 或客户相关的 prior questions、answers、corrections 和 confirmations
- `Profile`、Entity / Case / Rule / Governance context 中由上游节点已带入的 authority limits 或 review-needed context

#### Write allowed boundary

`Coordinator / Pending Node` 可以在 intervention 语境内执行有限 durable write：
- 记录 accountant-facing question。
- 记录 accountant answer、clarification、correction 或 confirmation 的交互语境。
- 保留本次 pending 为什么发生、回答解决了什么、仍缺什么的介入语境。

#### Candidate-only boundary

本节点只能作为候选或 handoff signal 表达：
- supplemental evidence / re-intake candidate
- profile change signal or profile confirmation candidate
- entity / alias / role confirmation candidate
- case memory update candidate
- rule candidate or rule conflict candidate
- automation-policy / governance candidate
- review-needed or unresolved-pending signal

#### No direct mutation boundary

本节点绝不能：
- 写入 `Transaction Log`
- 修改 `Profile`
- 写入或修改 `Entity Log`
- 写入或修改 `Case Log`
- 写入或修改 `Rule Log`
- 写入或批准 `Governance Log`
- 创建 stable entity
- approve / reject alias
- confirm role
- merge / split entity
- 修改 automation policy
- 创建、升级、修改、删除或降级 active rule
- 生成 journal entry
- 把 accountant answer 直接变成 final accounting result

### Insufficient / Ambiguous / Conflicting Evidence Behavior

Stage 2 采用优先级边界：ask only for repairable gaps、preserve authority limits、escalate conflicts。

#### Ask only for repairable gaps

如果当前卡点可以通过 accountant 补充一个具体事实推进，本节点应提出聚焦问题。

典型情况：
- 缺 receipt、invoice、contract、cheque context 或附件归属说明。
- 业务用途不明。
- counterparty / vendor / payee 身份需要确认。
- role/context 需要 accountant 说明。
- profile structural fact 需要确认。
- case precedent 是否适用缺少一个具体条件。

#### Preserve authority limits

Accountant 回答可以补充当前交易 context，但不能让本节点绕过已知 authority limits。

典型边界：
- `new_entity_candidate` 不因一次回答获得 stable entity authority。
- candidate alias 不因一次 pending 回答直接成为 approved alias。
- candidate role 不因本节点解释直接成为 confirmed role。
- profile candidate 不因本节点记录直接成为 stable profile truth。
- rule candidate 不因 accountant 当前回答直接成为 active rule。
- automation policy upgrade or relaxation 必须走 approval。

#### Escalate conflicts

如果 accountant response 与 raw evidence、profile、Entity Log、Case Log、Rule Log、Governance context 或 prior intervention 冲突，本节点应保守处理。

典型边界：
- 冲突只是缺少一个可补事实：保持 pending，并提出更具体 clarification。
- 冲突影响当前交易分类：进入 Review Node 语境，而不是自动选择一方。
- 冲突暗示长期 memory 错误：形成 governance / memory candidate。
- 冲突影响 evidence authenticity 或 identity safety：不能继续 operational path。

#### Ambiguous or missing answer

如果 accountant 未回答、回答不完整、回答指代不明或回答无法绑定到具体交易 / evidence / issue，本节点不能自行填补。

#### Hard boundary

- Pending 是补信息机制，不是治理批准机制。
- Accountant answer 是高价值 context，但不是本节点的 durable memory mutation 权限。
- Intervention Log 记录介入过程，不等于 Case Log、Rule Log、Entity Log 或 Transaction Log。
- Transaction Log 不参与 runtime pending decision。
- Review-required 不能被本节点降级成普通自动化路径。
- Governance-needed 不能被本节点降级成普通 pending completion。
- 模糊、冲突或缺失回答不能被包装成 confidence。

## 3. Contract 字段摘要

### Contract Position in Workflow

#### Upstream Handoff Consumed

本节点消费 `pending_coordination_request`。

该 request 可由以下 upstream nodes 或 workflow context 触发：
- `Evidence Intake / Preprocessing Node`
- `Transaction Identity Node`
- `Profile / Structural Match Node`
- `Entity Resolution Node`
- `Rule Match Node`
- `Case Judgment Node`
- workflow runtime 中已经存在的 review-needed / clarification-needed context
该 handoff 必须说明：
- 当前 pending 绑定到哪笔稳定交易、哪个 batch、哪个 client；
- 上游哪个节点说明不能安全继续；
- 缺的是 evidence、identity、profile structure、entity / alias / role authority、rule authority、case judgment context、review context 还是 governance candidate context；
- 当前最多允许哪些后续动作。

#### Downstream Handoff Produced

本节点可产生以下 output categories：
- `accountant_clarification_request`：面向 accountant 的聚焦问题。
- `intervention_log_record`：写入 `Intervention Log` 的问答 / 修正 / 确认语境。
- `supplemental_context_handoff`：交给后续 workflow 的补充 evidence 或 context。
- `runtime_continuation_handoff`：说明当前交易可以在受限边界内继续后续 judgment / review flow。
- `review_or_governance_candidate_handoff`：只表达后续应评估的候选信号。
- `still_pending_handoff`：说明回答缺失、不足、冲突或仍不能继续。

#### Logs / Memory Stores Read

本节点可以读取或消费：
- runtime evidence foundation、objective transaction basis 和 `transaction_id`
- 上游节点带入的 pending / review / governance-needed context
- `Intervention Log` 中同一 transaction、batch、client 或相关 issue 的 prior questions / answers / corrections / confirmations
- `Profile`、`Entity Log`、`Case Log`、`Rule Log`、`Governance Log` 中由上游 handoff 带入的 effective authority limits 或 review-needed context
- `Knowledge Log / Summary Log` 的辅助摘要，但不能把它当作 deterministic rule source 或 accountant approval

#### Logs / Memory Stores Written or Candidate-Only

本节点唯一允许的 durable write 是 `Intervention Log` 语义下的 `intervention_log_record`。

该 write 只能记录：
- accountant-facing question
- accountant answer、clarification、correction、confirmation
- pending 为什么发生、回答解决了什么、仍缺什么
- 当前 interaction 与 transaction / evidence / upstream issue 的引用关系
本节点只能 candidate-only 输出：
- supplemental evidence / re-intake candidate
- profile change signal
- entity / alias / role confirmation candidate
- case memory update candidate
- rule candidate / rule conflict candidate
- automation-policy / governance candidate
- review-needed or unresolved-pending signal
本节点绝不能直接写入或修改：
- `Transaction Log`
- `Profile`
- `Entity Log`
- `Case Log`
- `Rule Log`
- approved `Governance Log`
- final JE / accounting result

### Input Contracts

#### `pending_coordination_request`

Allowed `trigger_source_node` values：
- `evidence_intake_preprocessing`
- `transaction_identity`
- `profile_structural_match`
- `entity_resolution`
- `rule_match`
- `case_judgment`
- `review_context`
- `workflow_orchestrator`
Optional fields：
- `prior_intervention_context`：同一 transaction / batch / issue 相关的既有 intervention context。
- `suggested_question_targets`：上游建议本节点询问的事实类别；只能作为 question target，不能作为 authority。
- `request_trace`：runtime diagnostics；不得作为业务结论。
Validation / rejection rules：
- `transaction_id`、`client_id`、`batch_id`、`trigger_source_node`、`pending_issue_context`、`authority_limit_context` 缺失时 input invalid。
- `pending_issue_context` 为空时 input invalid；本节点不能主动创造 pending reason。
- 如果当前交易已经被 deterministic path、approved review path、JE generation 或 Transaction Logging 完成，input invalid。
- `trigger_source_node` 必须能与至少一个 `pending_issue_context.source_node` 对齐。
- `request_trace` 不能被用作 accountant question 的事实来源。
Runtime-only vs durable references：
- envelope 本身是 runtime-only。
- `transaction_id`、`client_id`、evidence refs、memory refs 是 durable references。
- 接收该 request 不授予本节点任何 long-term memory mutation authority。

#### `transaction_context`

Required `objective_transaction_basis` fields：
- `transaction_date`
- `amount_abs`
- `direction`; allowed values: `inflow`, `outflow`
- `bank_account`
Optional fields：
- `raw_description`
- `description`
- `pattern_source`
- `receipt_refs`
- `cheque_info_refs`
- `invoice_refs`
- `contract_refs`
- `counterparty_surface_text`
- `currency`
Allowed `evidence_quality_status` values：
- `ready`
- `missing_evidence`
- `conflicting_evidence`
- `unmatched_evidence`
- `low_quality_evidence`
- `identity_or_attachment_unclear`
Validation / rejection rules：
- `amount_abs` 必须为正数；金额方向只能由 `direction` 表达，不能由 signed amount 暗示。
- `direction` 必须属于允许值。
- `evidence_refs` 不能为空，除非 pending issue 本身是 “missing evidence for known transaction”；这种情况必须在 `pending_issue_context` 中明确说明缺什么 evidence。
- `description = null` 和 `pattern_source = null` 都是有效状态；本节点不能要求 canonical description 存在。
- accountant-facing question 不能引用没有 `evidence_ref` 支撑的模型总结作为事实。
Runtime-only vs durable references：
- `transaction_context` 是 runtime handoff。
- 其中的 objective fields 来自 durable evidence / transaction identity。
- 本节点不能用自然语言 clarification 覆盖原始 evidence。

#### `pending_issue_context`

Allowed `source_node` values：
- `evidence_intake_preprocessing`
- `transaction_identity`
- `profile_structural_match`
- `entity_resolution`
- `rule_match`
- `case_judgment`
- `review_context`
- `workflow_orchestrator`
Allowed `issue_type` values：
- `missing_evidence`
- `conflicting_evidence`
- `unmatched_evidence`
- `transaction_identity_unclear`
- `duplicate_or_same_transaction_unclear`
- `profile_structural_fact_missing`
- `profile_structural_conflict`
- `entity_ambiguous`
- `entity_unresolved`
- `new_entity_context_needed`
- `alias_authority_gap`
- `role_unconfirmed`
- `rule_blocked`
- `rule_conflict`
- `automation_policy_block`
- `case_precedent_uncertain`
- `case_evidence_insufficient`
- `review_required`
- `governance_candidate`
- `other_upstream_pending_issue`
Allowed `question_target` values：
- `evidence_attachment`
- `business_purpose`
- `counterparty_identity`
- `role_or_context`
- `profile_structural_fact`
- `rule_exception_or_conflict`
- `case_precedent_applicability`
- `review_confirmation`
- `governance_signal_context`
- `unresolved_issue_explanation`
Optional fields：
- `candidate_entity_refs`
- `candidate_alias_text`
- `candidate_role`
- `candidate_rule_refs`
- `case_memory_refs`
- `risk_or_policy_flags`
- `upstream_confidence`
- `upstream_policy_trace`
Validation / rejection rules：
- 每个 issue 必须有 `source_node`、`issue_type`、`blocking_reason` 和至少一个 `question_target`。
- `issue_type = new_entity_context_needed` 不得被解释为自动阻断 current operational classification；它只表示不能产生 durable entity / rule authority，且是否能继续分类取决于后续 evidence / judgment authority。
- `candidate_alias_text` 不能被当作 approved alias。
- `candidate_role` 不能被当作 confirmed role。
- `upstream_confidence` 不能替代 accountant clarification、review approval 或 governance approval。
- `review_required` 和 `governance_candidate` 不能被降级成普通 clarification completion。
Runtime-only vs durable references：
- issue object 是 runtime-only。
- `supporting_refs` 指向 durable evidence / memory / intervention records。
- issue 中的 candidate fields 永远不是 durable authority。

#### `authority_limit_context`

Allowed `allowed_next_actions` values：
- `ask_accountant`
- `record_intervention`
- `handoff_supplemental_context`
- `continue_runtime_judgment`
- `route_to_review`
- `propose_governance_candidate`
- `remain_pending`
Allowed `hard_blocks` values：
- `missing_stable_transaction_identity`
- `evidence_authenticity_or_attachment_conflict`
- `ambiguous_entity`
- `unresolved_entity`
- `unconfirmed_role`
- `candidate_or_rejected_alias`
- `rule_required_without_active_rule`
- `review_required_policy`
- `disabled_automation_policy`
- `governance_approval_required`
- `final_processing_already_completed`
Required `memory_mutation_policy` value for this node：
- `intervention_log_only_no_business_memory_mutation`
Optional fields：
- `effective_entity_policy`
- `effective_rule_lifecycle_status`
- `effective_review_requirement`
- `effective_governance_refs`
- `authority_notes`
Validation / rejection rules：
- 如果 `memory_mutation_policy` 允许本节点直接修改 Profile、Entity、Case、Rule、Governance 或 Transaction Log，则 input invalid。
- `allowed_next_actions` 不能包含本节点无权执行的 final accounting classification、JE generation、Transaction Log write 或 governance approval。
- `hard_blocks` 必须被 output 保留或升级，不能被 omission 消除。
Runtime-only vs durable references：
- 该对象是 runtime projection。
- governance / entity / rule references 可以指向 durable records，但本节点只有读取 / candidate handoff 权限。

#### `prior_intervention_context`

Allowed `relation_scope` values：
- `same_transaction`
- `same_batch`
- `same_entity_or_candidate`
- `same_issue_type`
- `same_client_recent`
Optional fields：
- `prior_question_summary`
- `prior_answer_summary`
- `prior_resolution_status`
- `prior_conflict_flags`
Validation / rejection rules：
- prior intervention 可以提示重复或冲突，但不能替代 `Entity Log`、`Case Log`、`Rule Log`、`Governance Log` 或 `Profile` authority。
- 未经 review / governance 转化的 prior answer 不能作为 current durable business truth。
- 如果 prior answer 与 current evidence 或 memory authority 冲突，本节点必须保留 conflict signal。
Runtime-only vs durable references：
- summaries 是 runtime projection。
- `intervention_refs` 是 durable references。
- prior intervention 本身只属于 intervention history，不是 deterministic automation authority。

#### `accountant_response_payload`

Allowed `response_type` values：
- `answer`
- `correction`
- `confirmation`
- `cannot_answer`
- `supplemental_evidence_provided`
- `needs_review`
- `governance_context_provided`
Optional fields：
- `attached_evidence_refs`
- `target_transaction_id`
- `target_issue_ids`
- `structured_response_claims`
- `accountant_confidence_note`
- `response_language`
Validation / rejection rules：
- `clarification_request_id` 必须能对应当前或 prior unresolved question。
- `target_transaction_id` 如存在，必须等于 request 的 `transaction_id`；否则必须作为 identity conflict 处理。
- `response_text` 不能为空，除非 `response_type = supplemental_evidence_provided` 且 `attached_evidence_refs` 非空。
- `structured_response_claims` 只能表示 accountant 说了什么，不能自动变成 Profile truth、approved alias、confirmed role、active rule 或 governance approval。
- `response_type = governance_context_provided` 只产生 governance candidate context，不等于 approved governance event。
Runtime-only vs durable references：
- payload 本身在写入前是 runtime input。
- 被接受后可写入 `Intervention Log`。
- attached raw evidence 若需要保存或重新处理，仍属于 Evidence Intake / Evidence Log 边界。

### Output Contracts

#### `accountant_clarification_request`

Allowed `question_status` values：
- `drafted`
- `sent`
- `answered`
- `superseded`
- `cancelled`
Optional fields：
- `answer_format_hint`：例如 yes/no、选择 candidate、补充一句业务用途、上传附件。
- `grouped_issue_ids`：多个 issue 被合并为一个问题时使用。
- `do_not_imply_facts`：列出不能包装成事实的 candidate / conflict。
Validation rules：
- `question_text` 必须绑定至少一个 `source_issue_id`。
- `question_text` 不得要求 accountant 批准内部系统流程。
- `question_text` 不得把 candidate entity、candidate alias、candidate role、candidate rule 包装为已确认事实。
- `authority_warning` 必须覆盖相关 issue 的 no-mutation / governance boundary。
Durable / candidate / runtime：
- question object 是 runtime output。
- 发送或记录后，其内容可以作为 `Intervention Log` record 的一部分 durable 保存。
- 它不写任何 business memory。

#### `intervention_log_record`

Allowed `intervention_type` values：
- `question_asked`
- `answer_received`
- `correction_received`
- `confirmation_received`
- `supplemental_evidence_received`
- `unresolved_followup`
- `review_context_captured`
- `governance_candidate_context_captured`
Allowed `resolution_status` values：
- `open`
- `partially_answered`
- `answered_for_current_context`
- `needs_followup`
- `needs_review`
- `governance_candidate_only`
- `unresolved`
- `conflicting`
Optional fields：
- `attached_evidence_refs`
- `resolved_issue_ids`
- `remaining_issue_ids`
- `conflict_refs`
- `candidate_signal_refs`
- `supersedes_intervention_id`
- `language_or_translation_note`
Validation rules：
- `intervention_id`、`transaction_id`、`source_issue_ids`、`intervention_type`、`authority_boundary` 必须存在。
- `question_asked` 必须有 `question_text`。
- `answer_received` / `correction_received` / `confirmation_received` 必须有 `accountant_response_text` 或可追溯 `attached_evidence_refs`。
- `resolution_status = answered_for_current_context` 只表示当前 context 已被回答，不表示 final classification、durable memory update 或 governance approval。
- `governance_candidate_only` 不能写成 approved governance result。
Durable / candidate / runtime：
- 该对象是本节点唯一允许 durable write 的类别。
- 它属于 `Intervention Log`，不是 `Transaction Log`、`Case Log`、`Rule Log`、`Entity Log` 或 `Governance Log`。
- 它可以成为后续 workflow 的 evidence/context reference，但不直接成为 deterministic authority。

#### `supplemental_context_handoff`

Allowed `handoff_type` values：
- `supplemental_evidence`
- `evidence_attachment_clarification`
- `business_context`
- `identity_context`
- `profile_context`
- `case_context`
- `review_context`
Optional fields：
- `recommended_downstream_consumer`
- `requires_re_intake`：boolean
- `attached_evidence_refs`
- `remaining_authority_limits`
Validation rules：
- `source_intervention_id` 必须存在。
- `supplemental_evidence` 必须携带 `attached_evidence_refs` 或明确说明 evidence 缺失。
- `business_context` / `identity_context` / `profile_context` 不能覆盖 raw evidence。
- `requires_re_intake = true` 时，本 handoff 仍不等于 Evidence Log write；是否 intake 属于 Evidence Intake boundary。
Durable / candidate / runtime：
- 该 handoff 是 runtime-only。
- `source_intervention_id` 和 evidence refs 是 durable references。
- 它不写 long-term memory。

#### `runtime_continuation_handoff`

Allowed `continuation_status` values：
- `context_ready_for_case_judgment`
- `context_ready_for_review`
- `context_ready_for_limited_reprocessing`
- `still_requires_review`
Optional fields：
- `suggested_reprocessing_scope`
- `supplemental_context_handoff_ids`
- `review_note`
Validation rules：
- `continuation_status` 不是最终 routing enum；它只说明 data contract 上的 handoff readiness。
- 若存在 hard block，例如 `governance_approval_required`、`review_required_policy`、`disabled_automation_policy`，不得输出会暗示自动 completion 的 status。
- `resolved_issue_ids` 不能包含实际未被 accountant 明确回答或 evidence 修复的 issue。
Durable / candidate / runtime：
- 该 handoff 是 runtime-only。
- 它不写 durable memory。
- 它不批准 classification、review outcome 或 governance event。

#### `review_or_governance_candidate_handoff`

Allowed `candidate_type` values：
- `profile_change_signal`
- `entity_candidate_context`
- `alias_confirmation_candidate`
- `role_confirmation_candidate`
- `case_memory_update_candidate`
- `rule_candidate`
- `rule_conflict_candidate`
- `automation_policy_candidate`
- `governance_review_candidate`
- `review_required_signal`
Allowed `required_authority_path` values：
- `review_node`
- `case_memory_update_node`
- `governance_review_node`
- `evidence_intake_reprocessing`
- `post_batch_lint_node`
Optional fields：
- `affected_entity_refs`
- `affected_alias_text`
- `affected_role`
- `affected_rule_refs`
- `affected_profile_field_hint`
- `risk_flags`
- `conflict_refs`
Validation rules：
- candidate handoff 必须写明 `required_authority_path`。
- `alias_confirmation_candidate` 不能写成 `approved_alias`。
- `role_confirmation_candidate` 不能写成 confirmed role。
- `rule_candidate` 不能写成 active rule。
- `automation_policy_candidate` 不能放宽或升级 automation policy。
- `governance_review_candidate` 不能包含 `approval_status = approved`。
Durable / candidate / runtime：
- 该 handoff 是 candidate-only runtime output。
- 只有后续 Review / Governance / Case Memory workflow 才能把它转化为 durable memory 或 approved governance event。

#### `still_pending_handoff`

Allowed `pending_status` values：
- `awaiting_accountant_response`
- `answer_incomplete`
- `answer_ambiguous`
- `answer_conflicting`
- `evidence_still_missing`
- `authority_still_blocked`
- `requires_review_instead_of_pending`
- `requires_governance_instead_of_pending`
Optional fields：
- `conflict_refs`
- `prior_intervention_refs`
- `suggested_followup_question`
- `review_note`
Validation rules：
- `remaining_issue_ids` 不能为空。
- `unresolved_reason` 必须具体说明缺什么或冲突在哪里。
- 如果问题实际需要 review / governance，必须使用对应 status，不能伪装成普通 missing answer。
- 不能使用 confidence 语言掩盖 unresolved state。
Durable / candidate / runtime：
- handoff 是 runtime-only。
- 相关 unanswered / conflicting interaction 可以另行写入 `Intervention Log`。

### Field Authority and Memory Boundary

`transaction_id` 的 source of truth 是 `Transaction Identity Node`。本节点只能引用，不能分配、合并、重写或去重。

以下字段或对象永远不能由本节点变成 durable memory authority：
- `candidate_alias_text`
- `candidate_role`
- `candidate_entity_refs`
- `candidate_rule_refs`
- `structured_response_claims`
- `suggested_question_targets`
- `request_trace`
- `upstream_confidence`
- `upstream_policy_trace`
- `accountant_confidence_note`
以下内容只能在后续 authority path 批准后成为 durable memory：
- profile structural fact：只能经 Review / profile authority path 确认后写入 `Profile`。
- entity / alias / role：只能经 Review / Governance Review 后写入 `Entity Log` 或 effective governance projection。
- case memory：只能经交易完成后的 `Case Memory Update Node` 形成，不由本节点直接写入。
- active rule：只能经 accountant / governance approval 后写入 `Rule Log`。
- automation policy change：放宽或升级必须 accountant / governance approval；自动降级若由 lint pass 触发，也不属于本节点。
- governance event：只能由 `Governance Review Node` 批准或记录。
Audit vs learning / logging boundary：
- `Transaction Log` 是 audit-facing final processing record，只写和查询，不参与本节点 runtime decision。
- `Intervention Log` 保存 pending intervention 过程，可供 review / governance / future coordinator context 使用，但不是 learning layer 本身。
- `Log` 在本设计中表示 durable long-term meaningful information storage；临时 question queue、candidate queue、runtime handoff、report draft 不能因为是列表就称为 `Log`。

### Validation Rules

Contract-level validation rules：
- 每次 coordination 必须绑定 `client_id`、`batch_id`、`transaction_id` 和至少一个 upstream issue。
- 每个 accountant question 必须能追溯到具体 issue 和 supporting refs。
- 每个 output 必须保留相关 authority boundary，尤其是 no direct business memory mutation。
- 本节点只能写 `Intervention Log`；所有其他 output 都是 runtime-only 或 candidate-only。
- `new_entity_candidate` 可以支持当前交易后续 case judgment context，但不能支持 rule match、stable entity authority 或 rule promotion。
- Case Judgment 的 candidate signals 可以作为 input context，但不能被本节点写成 long-term memory 或 governance approval。
Input invalid conditions：
- 缺少 stable `transaction_id`。
- 缺少 `pending_issue_context` 或上游卡点不能绑定具体 transaction / evidence / issue。
- 当前交易已经 final processed、JE generated 或 written to `Transaction Log`。
- request 要求本节点执行 final classification、JE、Transaction Log write、Profile / Entity / Case / Rule / Governance mutation。
- `authority_limit_context.memory_mutation_policy` 不是 `intervention_log_only_no_business_memory_mutation`。
Output invalid conditions：
- question 把候选对象包装为 confirmed fact。
- output 将 accountant answer 直接写成 Profile truth、approved alias、confirmed role、stable entity、case memory、active rule 或 approved governance event。
- output 使用 `Transaction Log` 作为 runtime pending decision source。
- still-unresolved / conflicting state 被标成可以自动 completion。
- review-required 或 governance-needed 被降级为 ordinary pending completion。
- `intervention_log_record` 缺少 authority boundary 或 source issue references。
Stop / ask conditions for unresolved contract authority：
- 如果后续设计要求本节点直接接收并保存 raw supplemental evidence，而不经过 Evidence Intake / Evidence Log 边界，应停止并确认 evidence re-intake contract。
- 如果 workflow 需要把 accountant clarification 直接作为 current-batch final classification approval，应停止并确认 Review Node 边界。
- 如果产品想让本节点创建 / approve durable entity、alias、role、rule、profile 或 governance changes，应停止并确认治理 authority 是否改变。
- 如果 exact routing enum 要统一 across Coordinator、Review、Case Judgment、Governance Review，应在跨节点 contract freeze 中决定，不应由本节点单独发明。

## 4. Open Boundaries

### Stage 1: Open Boundaries

以下问题留到后续阶段，不在 Stage 1 冻结：
- accountant 回答后是否重新触发哪个上游节点，以及如何限制重跑边界。
- 哪些 accountant clarification 足以支持当前交易继续处理，哪些必须进入 Review Node。
- `Intervention Log` 的 exact contract 和与 Evidence Log 的边界。
- 补交 evidence 应由本节点直接接收、还是重新进入 Evidence Intake / Preprocessing Node。
- 多笔 pending 交易的问题聚合、去重和展示边界。
- review-required、governance-needed 和 ordinary pending 的 exact routing contract。
- exact input / output schema、字段名、routing enum 和执行算法。

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
- accountant 回答后是否重新触发哪个上游节点，以及重跑边界
- `Intervention Log` exact contract，以及它与 Evidence Log、Review Node、Governance Log 的交接边界
- 补交 evidence 是否由本节点接收后转交，还是必须重新进入 Evidence Intake / Preprocessing
- ordinary pending、review-required、governance-needed 和 still-unresolved 的 exact routing contract
- accountant response 何时足以支持当前交易继续处理，何时必须进入 Review Node
- 多笔 pending 交易的问题聚合、去重、排序和展示边界

### Stage 3: Open Contract Boundaries

- `Intervention Log` 的全局 schema 尚未在 active docs 中完整冻结；本文件只定义本节点最小可写 record category。
- accountant 补交 raw evidence 后，是由本节点先捕捉再转交，还是必须由 Evidence Intake 重新接收，仍需跨节点 contract 决定；本文件用 `requires_re_intake` 保留边界。
- accountant 回答后 exact reprocessing target 和 rerun boundary 尚未冻结；本文件只定义 handoff readiness，不定义执行算法。
- ordinary pending、review-required、governance-needed、still-unresolved 的全局 routing enum 尚未跨节点冻结；本文件中的 status 是 Coordinator data-contract labels，不是最终 workflow routing enum。
- 多笔 pending 交易的问题聚合、去重、排序和展示方式仍未冻结；本文件只允许 `grouped_issue_ids` 表示问题合并，不定义 UI 或 batching algorithm。
- accountant response 何时足以支持 current operational classification、何时必须进入 Review Node，还需要 Case Judgment / Review Node Stage 3 共同收敛。
