# Review Node — 设计摘要

> 由原 Stage 1/2/3 合并瘦身而来；保留定位、边界、contract 字段和开放问题，删除阶段说明、examples、self-review、历史读档记录和重复解释。

## 1. 定位与职责

Stage 1 已确定以下内容：
- `Review Node` 是 accountant-facing review and decision-capture gate。
- 它面向当前交易或批次中需要 accountant authority 的系统结果、人工澄清结果、风险说明和治理候选。
- 它捕捉 accountant 的批准、修正、拒绝或继续待处理决定。
- 它不重新执行 evidence intake、transaction identity、structural match、entity resolution、rule match、case judgment 或 pending question generation。
- 它不替 accountant 作最终业务决定。
- 它不生成 journal entries。
- 它不写 `Transaction Log`。
- 它不直接修改 `Profile`、Entity / Case / Rule / Governance 等长期业务记忆。
- 它可以把长期记忆或治理相关事项作为候选或 accountant-facing review context 交给后续 workflow。
- 高权限长期 memory / governance mutation 必须由 accountant / governance approval 和后续治理 workflow 处理。

## 2. 逻辑与边界

### Trigger Boundary

`Review Node` 在当前交易、批次或治理候选存在 accountant-facing review need 时触发。

概念触发条件是：
- 上游 workflow 已形成可追溯 evidence、稳定交易身份、系统处理结果、pending clarification result、review-required context 或 candidate signal；并且
- 当前事项需要 accountant 审核、批准、修正、拒绝或继续保持 pending；并且
- 该事项尚未进入 JE finalization、final transaction logging 或 durable governance mutation execution。
典型触发来源包括：
- upstream deterministic 或 case-supported result 需要 accountant review。
- `Coordinator / Pending Node` 回收了 accountant clarification、correction 或 supplemental context，需要决定当前交易是否足以继续。
- 上游节点暴露 review-required、governance-needed、conflicting evidence 或 authority-risk context。
- `Case Judgment Node` 产生 runtime rationale 或 candidate signals，需要 accountant-facing review context。
- `Rule Match Node`、`Entity Resolution Node` 或 `Post-Batch Lint Node` 产生 rule / alias / role / entity / automation-policy 候选或风险说明。
- 系统需要在 JE generation 和 final audit logging 之前捕捉 accountant approval / correction。

### Input Categories

Stage 2 按判断作用组织 input categories，不列字段清单。

#### Reviewable transaction result context

说明当前交易已经形成了什么系统处理结果、来自哪条 workflow path、为什么认为可审核。

#### Evidence and audit-support context

说明 accountant 审核当前结果需要看到哪些 evidence foundation、evidence references、objective transaction basis、upstream rationale、risk reason 和 unresolved issue。

#### Accountant intervention and correction context

说明 accountant 已经通过 pending、review 或人工修正提供过哪些 clarification、correction、confirmation 或 objection。

#### Candidate and governance context

说明当前 review package 中有哪些 memory / governance 候选或风险：
- case memory update candidate
- entity / alias / role candidate
- rule candidate、rule conflict 或 rule-health concern
- automation-policy candidate
- merge / split candidate
- auto-applied downgrade needing accountant visibility

#### Authority and policy context

说明当前 review item 最多允许产生什么后续结果：
- approve current transaction outcome
- apply accountant correction to current transaction flow
- keep pending or return for clarification
- create candidate handoff
- enter governance review
- block finalization

#### Batch-level review context

说明一组交易或候选是否需要聚合给 accountant 审核，例如重复风险、同类 correction、case-to-rule candidate、entity split / merge concern 或 automation-risk trend。

### Output Categories

Stage 2 只定义 conceptual output categories，不冻结 routing enum 或对象形状。

#### Accountant-approved current outcome handoff

含义：accountant 已批准当前交易的处理结果，后续可以进入 JE generation、final completion 和 audit logging flow。

边界：
- 这是当前交易的 accountant-facing approval。
- 它不等于 rule promotion。
- 它不等于 case memory write。
- 它不等于 entity / alias / role / automation-policy approval。
- 它不写 `Transaction Log`；final audit logging 属于 `Transaction Logging Node`。

#### Accountant-corrected current outcome handoff

含义：accountant 修正了当前交易处理结果或相关业务语境，后续 workflow 应基于修正语境继续处理、生成 JE 或形成候选信号。

边界：
- Correction 可以改变当前交易 outcome。
- Correction 不自动改变长期 memory。
- Correction 不自动创建 active rule。
- Correction 不自动批准 alias、role、entity 或 automation policy。
- 如果 correction 暗示长期记忆错误，应形成 candidate / governance handoff。

#### Rejected / not-approved / still-pending handoff

含义：accountant 未批准当前系统结果，或当前 review package 仍缺关键 evidence、clarification、authority 或 conflict resolution。

边界：
- 未批准不能被系统解释成“选择最接近的系统建议”。
- still-pending 应返回补信息或上游修正语境。
- conflict unresolved 时不能进入 JE generation 或 final logging。

#### Review intervention record

含义：accountant 在 review 中的 approval、correction、rejection、confirmation、objection 或说明被保留为 intervention / review 语境。

边界：
- 该记录保存 accountant interaction 和 review rationale。
- 它不等于 final `Transaction Log`。
- 它不等于 `Case Log`、`Rule Log`、stable `Entity Log` 或 `Governance Log` mutation。
- 它不把 transient review draft、queue 或 handoff 称为 `Log`。

#### Memory / governance candidate handoff

含义：Review Node 可以把 accountant review 中暴露的长期记忆或治理事项整理为后续 workflow 候选。

典型候选包括：
- case memory update candidate
- entity / alias / role candidate
- rule candidate 或 rule conflict candidate
- automation-policy candidate
- merge / split candidate
- governance review candidate
边界：
- Candidate handoff 只表示“后续应评估”。
- 它不写长期 memory。
- 它不批准 governance event。
- 它不改变 active rule 或 automation policy。

#### Batch review summary handoff

含义：对于一组交易或候选，Review Node 可以输出 accountant-facing review summary、已处理事项、未处理事项、仍需澄清事项和治理候选分组。

边界：
- Summary 是审核辅助，不是 deterministic rule source。
- Summary 不替代 `Knowledge Log / Summary Log` 的客户知识编译职责。
- Summary 不能直接作为 future runtime authority。

### Deterministic Code vs LLM Semantic Judgment

Stage 2 的核心边界是：

#### Deterministic code responsibility

Deterministic code 负责：
- 判断本节点是否被触发：存在 accountant-facing review need，且尚未进入 JE finalization、final logging 或 governance mutation execution。
- 汇总上游 result、rationale、evidence references、intervention context、candidate signals 和 authority limits。
- 区分 current transaction approval、current transaction correction、still-pending、review-required、governance-needed 和 candidate-only context。
- 保持 review package 与具体交易、批次、evidence 或候选事项绑定。
- 防止未经 accountant 明确决定的系统建议进入 approved outcome。
- 防止 review correction 直接写入 `Profile`、Entity / Case / Rule / Governance memory。
- 防止本节点写入 `Transaction Log` 或生成 journal entry。
- 标记哪些 accountant decision 只影响当前交易，哪些只能形成 memory / governance candidate。
- 执行 hard blocks：unresolved identity、ambiguous evidence、review-required policy、governance block、rule-required condition、disabled automation 或 conflicting evidence 不能被 review summary 语言绕过。

#### LLM semantic judgment responsibility

LLM 可以在 code 允许的边界内辅助：
- 将上游 rationale、pending history 和 evidence context 总结成 accountant 可读的 review explanation。
- 对多个候选或风险按业务含义聚合展示。
- 解释 accountant 自然语言 correction 的可能含义，并标记仍需确认的歧义。
- 总结 system result 与 accountant correction 的差异。
- 生成 governance candidate explanation、still-pending explanation 或 unresolved-review explanation。

#### Hard boundary

LLM 不能：
- 替 accountant approve current outcome
- 把模糊回答当作明确批准
- 选择 COA / HST 处理来替代 accountant correction
- 扩大 automation authority
- 把 candidate 变成 durable memory
- approve / reject alias
- confirm role
- create stable entity
- merge / split entity
- create / promote / modify / delete / downgrade active rule
- upgrade or relax automation policy
- 批准 governance event
- 生成 journal entry
- 写入 `Transaction Log`

### Accountant Authority Boundary

Accountant 仍然拥有最终 accounting decision 和 durable authority。

本节点不能：
- 替 accountant 作最终决定
- 把未确认 review package 变成 approved outcome
- 把当前交易 correction 直接变成长期客户政策
- 把 repeated correction 自动升级成 active rule
- 把 candidate alias / role / entity 直接批准
- 把 automation policy upgrade or relaxation 直接生效

### Governance Authority Boundary

Governance-level changes 不属于 `Review Node` 的直接 mutation authority。

但本节点不能直接：
- approve / reject alias as durable authority
- confirm role as durable authority
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

`Review Node` 可以读取或消费以下 conceptual context：
- current evidence foundation、objective transaction basis 和 stable transaction identity
- upstream results and rationale from Profile / Structural Match、Entity Resolution、Rule Match、Case Judgment 和 Coordinator / Pending Node
- `Intervention Log` 中与当前交易、批次或客户相关的 questions、answers、corrections、confirmations 和 prior review context
- `Profile`、Entity / Case / Rule / Governance context 中与当前 review item 相关的 authority limits、risk flags、automation policy、rule lifecycle 或 candidate state
- `Knowledge Log / Summary Log` 中可读的客户知识摘要，但只能作为 review context，不能替代 deterministic rule source 或 durable authority
- `Transaction Log` 中既有 finalized audit history，但不能作为当前 runtime decision、rule source、case memory 或 learning layer

#### Write allowed boundary

`Review Node` 可以在 review / intervention 语境内执行有限 durable write 或形成等价 handoff：
- 记录 accountant review action。
- 记录 accountant approval、correction、rejection、confirmation、objection 或 still-pending explanation。
- 记录 review package 为什么被批准、修正、拒绝或退回。

#### Candidate-only boundary

本节点只能作为候选或 handoff signal 表达：
- supplemental evidence / reprocessing candidate
- profile change or profile confirmation candidate
- entity / alias / role confirmation candidate
- case memory update candidate
- rule candidate、rule conflict candidate 或 rule-health candidate
- automation-policy / governance candidate
- merge / split candidate
- still-pending or unresolved-review signal

#### No direct mutation boundary

本节点绝不能：
- 写入 `Transaction Log`
- 生成 journal entry
- 修改 `Profile`
- 写入或修改 stable `Entity Log` authority
- 写入或修改 `Case Log`
- 写入或修改 `Rule Log`
- 直接写入或批准 `Governance Log` mutation
- 创建 stable entity
- approve / reject alias as durable authority
- confirm role as durable authority
- merge / split entity
- 修改 automation policy
- 创建、升级、修改、删除或降级 active rule
- 把 review summary、candidate 或 accountant 模糊回答变成 durable authority

### Insufficient / Ambiguous / Conflicting Evidence Behavior

Stage 2 采用优先级边界：do not approve incomplete review、preserve conflict、separate current outcome from durable authority。

#### Do not approve incomplete review

如果 review package 缺少关键 evidence、upstream rationale、transaction identity、accountant decision 或 authority basis，本节点不能输出 approved current outcome。

典型情况：
- evidence reference 不足以支持 accountant 审核。
- system result 没有说明来源 path 或 rationale。
- pending answer 未绑定到具体交易或问题。
- accountant response 不完整、指代不明或没有表达明确 approval / correction。
- governance candidate 缺少业务含义或证据说明。

#### Preserve conflict

如果 accountant correction 与 raw evidence、Profile、Entity Log、Case Log、Rule Log、Governance context、prior intervention 或 system rationale 冲突，本节点应保留冲突并阻止静默 finalization。

典型边界：
- 冲突只缺一个具体事实：返回 pending / clarification。
- 冲突影响当前交易处理：保持 not-approved / review-required，不能进入 JE generation。
- 冲突暗示长期 memory 错误：形成 governance / memory candidate。
- 冲突影响 evidence authenticity、transaction identity 或 entity safety：不能继续 finalization。

#### Separate current outcome from durable authority

Accountant 可以批准或修正当前交易 outcome，但这不自动改变长期 memory 或 automation behavior。

典型边界：
- 当前交易 approval 不等于 case memory write。
- 当前交易 correction 不等于 active rule change。
- repeated correction 不等于 rule promotion。
- alias / role / entity clarification 不等于 durable authority。
- automation policy upgrade or relaxation 必须进入 governance approval。

#### Governance caution

如果治理候选证据不足、业务含义不清、影响范围不明、或 accountant response 模糊，本节点不能把候选转成 approved governance event。

#### Hard boundary

- Review 是 accountant authority interface，不是 LLM final decision interface。
- Accountant approval 必须明确、可追溯、可绑定到具体交易或候选事项。
- Transaction Log 是 audit-facing final log，不参与 runtime review decision。
- Intervention / review record 不等于 Transaction Log、Case Log、Rule Log、Entity Log 或 Governance Log mutation。
- Governance-needed 不能被降级成普通 current-transaction approval。
- 模糊、冲突或缺失 review 不能被包装成 confidence。

## 3. Contract 字段摘要

### Contract Position in Workflow

`Review Node` 位于 accountant-facing review gate：

#### Upstream handoff consumed

本节点消费 runtime `review_package_request`。

该 handoff 表示：
- upstream workflow 已形成可追溯 evidence、稳定 `transaction_id`、system result、pending clarification result、review-required context 或 governance / memory candidate；
- 当前事项需要 accountant 审核、批准、修正、拒绝或继续 pending；
- 当前事项尚未进入 JE finalization、final Transaction Log write 或 durable governance mutation execution。

#### Downstream handoff produced

本节点输出 `review_decision_package`，供下游 workflow 消费：
- `JE Generation Node`：只消费 accountant-approved 或 accountant-corrected current outcome handoff。
- `Transaction Logging Node`：消费 finalizable current outcome 与 review trace reference；本节点不写 `Transaction Log`。
- `Case Memory Update Node`：消费 accountant-confirmed outcome、correction context 和候选信号；本节点不写 `Case Log`。
- `Governance Review Node`：消费 entity / alias / role / rule / automation-policy / merge-split 等候选和 review context；本节点不批准或执行 governance mutation。
- `Coordinator / Pending Node`：消费 still-pending、clarification-needed 或 unresolved-review handoff。

#### Logs / memory stores read

本节点可以读取或消费：
- `Evidence Log` references 和 current evidence basis；
- stable transaction identity references；
- upstream outputs and rationale from Profile / Structural Match、Entity Resolution、Rule Match、Case Judgment、Coordinator / Pending、Post-Batch Lint；
- `Profile`、`Entity Log`、`Case Log`、`Rule Log`、`Governance Log` 中与当前 review item 有关的 authority limits、risk flags、policy constraints、candidate state 和已生效治理 context；
- `Intervention Log` 中与当前交易、批次或候选事项有关的 accountant questions、answers、corrections、confirmations、objections 和 prior review context；
- `Knowledge Log / Summary Log` 中可读摘要，但只能作为 review context；
- 既有 finalized `Transaction Log` history，但只能作为 audit-facing 历史参考，不能作为当前 runtime decision、rule source、case memory 或 learning authority。

#### Logs / memory stores written or candidate-only

本节点可以写入或形成等价 durable handoff 的唯一长期记录类别是 review / intervention 语义记录：
- accountant review action
- accountant approval / correction / rejection / confirmation / objection / still-pending explanation
- review package 被批准、修正、拒绝或退回的 reason / trace
本节点只能 candidate-only 输出：
- supplemental evidence / reprocessing candidate
- profile change or profile confirmation candidate
- entity / alias / role candidate
- case memory update candidate
- rule candidate / rule conflict / rule-health candidate
- automation-policy / governance candidate
- merge / split candidate
- still-pending or unresolved-review signal
本节点绝不能直接写入或修改：
- `Transaction Log`
- `Profile`
- stable `Entity Log` authority
- `Case Log`
- `Rule Log`
- approved `Governance Log` mutation
- journal entry

### Input Contracts

#### `review_package_request`

Allowed `review_scope` values：
- `single_transaction`
- `transaction_batch`
- `governance_candidate_batch`
- `mixed_review_package`
Allowed `trigger_reasons` values：
- `current_outcome_requires_review`
- `pending_clarification_returned`
- `accountant_requested_review`
- `review_required_policy`
- `conflicting_evidence`
- `authority_risk`
- `governance_candidate_visibility`
- `post_batch_lint_candidate`
- `batch_review_policy`
Optional fields：
- `package_summary_context`：给 accountant 的 package-level 摘要 context；不是 approval。
- `related_intervention_refs`：与本 package 有关的 prior intervention references。
- `request_trace`：runtime diagnostics / orchestration trace。
Validation / rejection rules：
- `review_package_id`、`client_id`、`batch_id`、`review_scope`、`trigger_reasons`、`review_items`、`authority_limit_context` 必须存在。
- `review_items` 不能为空。
- `trigger_reasons` 必须至少有一个 allowed value。
- 如果 package 声称某 item 已经 final logged 或 governance mutation 已执行，本节点输入 invalid；Review Node 不能作为 runtime decision source 重开最终记录。
- `request_trace` 不能作为 accountant approval、accounting authority、rule authority 或 governance authority。
Runtime-only vs durable references：
- `review_package_request` 是 runtime-only。
- `client_id`、`batch_id`、`transaction_id`、`evidence_refs`、memory refs、governance refs 是 durable references。
- 收到该 package 不给 Review Node 任何长期 memory mutation authority。

#### `review_item`

Allowed `item_type` values：
- `current_transaction_outcome`
- `transaction_correction_review`
- `still_pending_review`
- `evidence_conflict_review`
- `case_memory_candidate_review`
- `entity_candidate_review`
- `alias_candidate_review`
- `role_candidate_review`
- `rule_candidate_review`
- `automation_policy_candidate_review`
- `merge_split_candidate_review`
- `batch_review_summary`
Optional fields：
- `transaction_outcome_context`
- `evidence_review_context`
- `intervention_context`
- `candidate_governance_context`
- `batch_review_context`
- `risk_flags`
- `display_group_key`：review presentation grouping helper；不是 authority。
Validation / rejection rules：
- 每个 `review_item` 必须能绑定到具体 `transaction_id`、candidate id、entity/rule ref、evidence ref 或 batch ref；不能只有自然语言摘要。
- `review_question` 不能要求 accountant 批准一个 authority_limits 明确禁止的结果。
- `item_type` 与附带 context 必须匹配；例如 `current_transaction_outcome` 必须带 `transaction_outcome_context`。
- `display_group_key` 只能用于展示分组，不能作为去重、approval 或 governance authority。
Runtime-only vs durable references：
- `review_item` 是 runtime-only。
- `item_subject_refs` 中的 durable refs 只能被引用，不表示本节点拥有写入权。

#### `transaction_outcome_context`

Allowed `outcome_source_path` values：
- `profile_structural_match`
- `rule_match`
- `case_judgment`
- `coordinator_pending_return`
- `manual_review_context`
- `post_batch_rework`
Allowed `outcome_review_state` values：
- `ready_for_accountant_review`
- `review_required_by_policy`
- `pending_clarification_returned`
- `conflict_requires_review`
- `not_approved_prior_result`
- `blocked_from_finalization`
Required fields inside `current_outcome_payload`：
- `accounting_outcome_type`：当前结果类型；allowed values: `classification_result`, `structural_result`, `correction_result`, `no_final_outcome_yet`。
- `accounting_outcome_summary`：给 accountant 和 downstream handoff 使用的简短结果摘要。
- `finalization_readiness`：该 outcome 在当前 authority 下是否理论上可被 accountant approval 推入下游；allowed values: `finalizable_if_approved`, `requires_correction`, `not_finalizable`, `governance_only`。
- `source_authority`：说明该 payload 来自哪个 upstream result、accountant clarification 或 policy context。
Optional fields inside `current_outcome_payload`：
- `coa_account_ref`
- `hst_gst_treatment`
- `amount_allocation`
- `business_context`
- `entity_ref`
- `rule_ref`
- `case_refs`
- `policy_or_risk_trace_refs`
- `unresolved_fields`
Validation / rejection rules：
- 缺少 `transaction_id`、`current_outcome_payload`、`upstream_rationale_refs` 或 `evidence_refs` 时，不能输出 `approved_current_outcome_handoff`。
- `current_outcome_payload.finalization_readiness != finalizable_if_approved` 时，不能输出 approved current outcome；只能 correction、still-pending、rejection 或 candidate handoff。
- `accounting_outcome_type = no_final_outcome_yet` 时，不能被 accountant approval 包装成 final outcome。
- `coa_account_ref`、`hst_gst_treatment` 等 accounting fields 如存在，其 authority 属于 upstream result 或 accountant correction；Review Node 只能捕捉 accountant 对其批准或修正。
- `Transaction Log` history 不得出现在 `source_authority` 中作为当前 runtime outcome authority。
Runtime-only vs durable references：
- `transaction_outcome_context` 是 runtime-only。
- `transaction_id`、`evidence_refs`、entity/rule/case refs 是 durable references。
- `current_outcome_payload` 经 accountant approval / correction 后可成为下游 JE 和 final logging 的输入，但不是由 Review Node 写成 `Transaction Log`。

#### `evidence_review_context`

Allowed `evidence_sufficiency_status` values：
- `sufficient_for_review`
- `missing_key_evidence`
- `conflicting_evidence`
- `low_quality_evidence`
- `unbound_evidence`
Optional fields：
- `missing_evidence_types`
- `conflict_descriptions`
- `supporting_document_refs`
- `identity_or_dedupe_issue_refs`
- `evidence_authenticity_flags`
Validation / rejection rules：
- `evidence_refs` 不能为空。
- `evidence_sufficiency_status != sufficient_for_review` 时，不能输出 approved current outcome。
- `unbound_evidence` 表示 evidence 未绑定到明确交易或候选事项，不能被 review summary 语言绕过。
- `evidence_explanation` 不能替代 evidence refs，也不能覆盖 raw evidence。
Runtime-only vs durable references：
- context object 是 runtime-only。
- evidence references 是 durable refs。
- 本节点不能因 evidence review 而修改 `Evidence Log`；只能输出 supplemental evidence / reprocessing candidate。

#### `intervention_context`

Allowed `binding_status` values：
- `bound_to_item`
- `partially_bound`
- `unbound`
- `ambiguous_reference`
Optional fields：
- `raw_response_refs`
- `clarification_payload`
- `correction_payload`
- `confirmation_payload`
- `objection_payload`
- `unresolved_questions`
Validation / rejection rules：
- `binding_status != bound_to_item` 时，不能把 accountant response 当作 approval 或 correction authority。
- `actor_context` 必须能区分 accountant action、client note、system-generated summary 和 imported context。
- accountant 自然语言如果指代不明，必须保留为 ambiguous / still-pending；LLM 不能替 accountant 选择含义。
- `correction_payload` 只可影响 current transaction handoff 或生成 candidate；不能直接变成 durable profile/entity/rule/governance mutation。
Runtime-only vs durable references：
- prior `intervention_refs` 是 durable references。
- current review interaction 在输出 `review_intervention_record` 后可成为 durable intervention / review record。
- 本节点写入的 review/intervention 记录不等于 `Transaction Log`、`Case Log`、`Rule Log`、`Entity Log` 或 `Governance Log` mutation。

#### `candidate_governance_context`

Allowed `candidate_type` values：
- `supplemental_evidence_candidate`
- `profile_change_candidate`
- `profile_confirmation_candidate`
- `new_entity_candidate`
- `alias_candidate`
- `role_confirmation_candidate`
- `case_memory_update_candidate`
- `rule_candidate`
- `rule_conflict_candidate`
- `rule_health_candidate`
- `automation_policy_candidate`
- `merge_candidate`
- `split_candidate`
- `auto_applied_downgrade_visibility`
Allowed `approval_requirement` values：
- `current_transaction_only`
- `candidate_only`
- `accountant_approval_required`
- `governance_review_required`
- `auto_applied_visibility_only`
- `not_eligible_for_approval`
Allowed `current_authority_status` values：
- `no_durable_authority`
- `candidate_exists`
- `approved_authority_exists`
- `rejected_or_disabled`
- `conflicting_authority`
- `unknown_authority`
Optional fields：
- `suggested_downstream_workflow`；allowed values: `coordinator_pending`, `case_memory_update`, `governance_review`, `post_batch_lint`, `evidence_reprocessing`, `no_downstream_action`。
- `risk_flags`
- `affected_transactions`
- `related_intervention_refs`
- `related_governance_refs`
- `accountant_visibility_note`
Validation / rejection rules：
- `candidate_subject_refs` 不能为空。
- candidate context 不能声称 approved alias、confirmed role、stable entity、active rule 或 automation-policy upgrade 已经由 Review Node 生效。
- `auto_applied_downgrade_visibility` 可以说明 downgrade 已按系统 lint policy 生效并需要展示，但 Review Node 不能把它包装成 accountant-approved upgrade / relaxation。
- `approval_requirement = governance_review_required` 的 item 不能输出为普通 current transaction approval 后绕过 governance。
Runtime-only vs durable references：
- 该 context 在 Review Node 内是 runtime-only / candidate-only。
- 后续是否写入 `Governance Log`、`Entity Log`、`Rule Log`、`Case Log` 或 `Profile`，由下游 workflow 和 accountant/governance approval 决定。

#### `authority_limit_context`

Allowed `allowed_decision_types` values：
- `approve_current_outcome`
- `correct_current_outcome`
- `reject_current_outcome`
- `keep_still_pending`
- `return_for_clarification`
- `forward_candidate_only`
- `acknowledge_visibility_only`
- `block_finalization`
Optional fields：
- `policy_refs`
- `governance_refs`
- `automation_policy_refs`
- `rule_lifecycle_refs`
- `entity_authority_refs`
- `review_required_reasons`
Validation / rejection rules：
- Output `decision_type` 必须属于 `allowed_decision_types`。
- `finalization_blocks` 非空时，不能输出 finalizable approved handoff，除非 block 被 accountant correction 明确解决且 evidence/context 可追溯。
- `memory_mutation_blocks` 不能被 Review Node 输出覆盖；只能产生 candidate handoff。
- 如果 `requires_explicit_accountant_action = true`，accountant silence、bulk acknowledgement、模糊自然语言或系统建议不能算 approval。
Runtime-only vs durable references：
- `authority_limit_context` 是 runtime view。
- policy/governance/entity/rule refs 是 durable references。
- 本节点不得修改这些 authority refs。

#### `batch_review_context`

Allowed `batch_review_reason` values：
- `batch_policy_review`
- `repeated_correction_pattern`
- `rule_health_concern`
- `entity_merge_split_concern`
- `automation_risk_trend`
- `unresolved_item_summary`
- `accountant_requested_batch_view`
Optional fields：
- `affected_transaction_ids`
- `candidate_context_ids`
- `risk_summary`
- `partial_decision_allowed`：boolean。
Validation / rejection rules：
- `batch_review_context` 不能直接成为 future runtime authority。
- repeated correction 或 trend 不能自动升级为 rule、entity authority 或 automation-policy change。
- 批次级 acknowledgement 不能替代 item-level explicit approval，除非 `authority_limit_context` 明确允许且每个 item 绑定清楚。
Runtime-only vs durable references：
- batch context 是 runtime-only review aid。
- 其中的 transaction、candidate、intervention refs 可以被下游引用。
- 不属于 `Knowledge Log / Summary Log`，也不替代 Knowledge Compilation。

### Output Contracts

#### `review_decision_package`

Allowed `package_status` values：
- `all_items_decided`
- `partially_decided`
- `still_pending`
- `blocked_by_invalid_review`
- `candidate_only_forwarded`
- `no_finalization_allowed`
Optional fields：
- `package_level_summary`
- `review_intervention_record_refs`
- `downstream_handoff_refs`
- `unresolved_item_refs`
Validation rules：
- `decision_items` 不能为空。
- `review_actor` 必须能证明是 accountant 或被允许的 review actor；system actor 不能批准 accountant-only decision。
- `package_status` 必须由 item statuses 支撑；不能全部 item still-pending 但 package 标为 all_items_decided。
- output 不能包含 JE lines、Transaction Log entries、Case Log records、Rule Log records、stable Entity mutation 或 approved Governance mutation。
Durable boundary：
- package envelope 本身可以作为 review/intervention trace 被引用。
- 它不是 final audit log，不是 learning layer，不是 governance approval record。

#### `review_decision_item`

Allowed `decision_type` values：
- `approved_current_outcome`
- `corrected_current_outcome`
- `rejected_current_outcome`
- `still_pending`
- `returned_for_clarification`
- `candidate_forwarded`
- `visibility_acknowledged`
- `finalization_blocked`
Allowed `decision_authority` values：
- `explicit_accountant_approval`
- `explicit_accountant_correction`
- `explicit_accountant_rejection`
- `explicit_accountant_pending_decision`
- `accountant_visibility_acknowledgement`
- `system_validation_block`
Allowed `downstream_route` values：
- `je_generation`
- `transaction_logging_after_finalization`
- `case_memory_update_candidate`
- `governance_review`
- `coordinator_pending`
- `evidence_reprocessing`
- `upstream_rework`
- `no_downstream_action`
Optional fields：
- `approved_current_outcome_handoff`
- `corrected_current_outcome_handoff`
- `rejected_or_pending_handoff`
- `memory_governance_candidate_handoff`
- `review_intervention_record`
- `conflict_preservation_note`
Validation rules：
- `decision_type = approved_current_outcome` 必须使用 `decision_authority = explicit_accountant_approval`，且不得存在 unresolved finalization block。
- `decision_type = corrected_current_outcome` 必须使用 `decision_authority = explicit_accountant_correction`，并带 `corrected_current_outcome_handoff`。
- `candidate_forwarded` 不能被 downstream 当作 approved governance event。
- `visibility_acknowledged` 只能说明 accountant 已看到事项；不能变成 approval、correction、rule change 或 policy change。
- `system_validation_block` 只能产生 blocked / pending / clarification route，不能产生 accountant approval。
Durable boundary：
- decision item 可以写成 review/intervention record。
- 它不直接写入最终 transaction audit、case memory、rule memory、entity authority 或 governance mutation。

#### `approved_current_outcome_handoff`

Optional fields：
- `upstream_outcome_context_id`
- `approved_with_conditions`
- `condition_notes`
- `case_memory_candidate_hint`
Validation rules：
- 必须有 explicit accountant approval。
- `approved_outcome_payload.finalization_readiness` 必须为 `finalizable_if_approved` 或等价的 accountant-corrected finalizable state。
- `approval_evidence_refs` 不能为空。
- 如果存在 unresolved evidence conflict、identity conflict、governance block 或 review-required policy block，output invalid。
- 该 handoff 不等于 case memory write、rule promotion、entity / alias / role approval 或 automation-policy approval。
Durable boundary：
- runtime handoff can feed JE / final logging。
- Review Node 不写 JE，不写 Transaction Log。
- `case_memory_candidate_hint` 只是 candidate，不是 `Case Log` write。

#### `corrected_current_outcome_handoff`

Allowed `correction_scope` values：
- `current_transaction_only`
- `current_transaction_plus_candidate_signal`
- `requires_upstream_rework`
- `requires_governance_review`
- `not_finalizable_yet`
Optional fields：
- `corrected_outcome_payload`
- `unresolved_followup_questions`
- `memory_governance_candidate_handoff`
- `review_trace_ref`
Validation rules：
- 必须有 explicit accountant correction。
- `correction_payload` 必须绑定到明确 `transaction_id` 和原始 review item。
- 如果 correction 仍缺关键 COA / HST / evidence / identity 信息，不能 route 到 `je_generation`，应 route 到 pending、upstream rework 或 governance review。
- `correction_scope` 不能被扩大为 durable memory mutation。
- 如果 correction 暗示 Profile、Entity、Rule、Case 或 automation policy 需要变化，只能输出 candidate handoff。
Durable boundary：
- correction 可成为 current transaction downstream authority。
- correction 不自动写长期 memory，不自动创建 active rule，不自动批准 alias / role / entity / automation policy。

#### `rejected_or_pending_handoff`

Allowed `blocking_category` values：
- `missing_evidence`
- `ambiguous_accountant_response`
- `unresolved_identity`
- `conflicting_evidence`
- `authority_block`
- `governance_needed`
- `accounting_outcome_rejected`
- `insufficient_review_context`
Optional fields：
- `suggested_downstream_route`
- `related_intervention_refs`
- `candidate_handoff_refs`
Validation rules：
- rejected / still-pending item 不能进入 JE generation。
- `ambiguous_accountant_response` 不能被 LLM 解释成 approval。
- `governance_needed` 不能被降级为普通 current transaction approval。
Durable boundary：
- 可写 review/intervention trace。
- 不写 Transaction Log final outcome。

#### `review_intervention_record`

Allowed `actor_role` values：
- `accountant`
- `authorized_reviewer`
- `system_validation`
Allowed `action_type` values：
- `approve_current_outcome`
- `correct_current_outcome`
- `reject_current_outcome`
- `keep_still_pending`
- `request_clarification`
- `confirm_context_for_current_transaction`
- `object_to_system_result`
- `acknowledge_candidate_visibility`
- `system_blocked_invalid_review`
Optional fields：
- `rationale_text`
- `correction_payload`
- `visibility_candidate_refs`
- `conflict_notes`
- `downstream_handoff_refs`
Validation rules：
- `system_validation` actor_role 不能产生 accountant approval 或 correction。
- `raw_action_ref` 必须保留或能追溯 accountant 原话 / 操作；不能只保存 LLM paraphrase。
- `normalized_action_summary` 不能比 raw action 拥有更高 authority。
- record 不能包含 final Transaction Log entry、JE lines、Case Log write、Rule Log write、Entity authority mutation 或 Governance approval mutation。
Durable boundary：
- 这是 Review Node 唯一允许写入或形成 durable-equivalent handoff 的记录类别。
- 它是 intervention / review interaction record，不是 audit-facing final transaction log，不是 learning layer。

#### `memory_governance_candidate_handoff`

Allowed `required_authority_path` values：
- `case_memory_update_review`
- `governance_review_required`
- `accountant_approval_required_downstream`
- `evidence_reprocessing_required`
- `coordinator_clarification_required`
- `post_batch_lint_followup`
- `candidate_rejected_no_action`
Optional fields：
- `suggested_event_type`
- `affected_transactions`
- `proposed_value`
- `old_value_context`
- `risk_flags`
- `blocking_reasons`
Validation rules：
- `candidate_handoff_id` 不能被当成 durable memory id。
- handoff 不能声明 alias approved、role confirmed、entity stable、rule active、automation policy upgraded / relaxed 或 governance event approved。
- `proposed_value` 只是 proposal；不能直接执行。
- `required_authority_path = governance_review_required` 时，consumer 必须进入 Governance Review 或等价 governance workflow。
Durable boundary：
- candidate-only。
- 不写长期 memory，不批准治理，不改变 automation behavior。

#### `batch_review_summary_handoff`

Allowed `summary_status` values：
- `batch_review_complete`
- `batch_review_partial`
- `batch_review_blocked`
- `governance_candidates_forwarded`
- `still_pending_items_exist`
Optional fields：
- `accountant_visible_summary`
- `risk_summary`
- `next_review_focus`
Validation rules：
- summary 不能替代 item-level review decision。
- summary 不能作为 deterministic rule source。
- summary 不属于 `Knowledge Log / Summary Log`，不能直接作为 future runtime memory。
Durable boundary：
- 可作为 review trace 被保存或引用。
- 不写 final Transaction Log，不写长期 memory。

### Field Authority and Memory Boundary

#### Source of truth for important fields

- `transaction_id`：source of truth 是 Transaction Identity Node / transaction identity layer。
- `client_id`、`batch_id`：source of truth 是 client / workflow runtime context。
- `evidence_refs`：source of truth 是 `Evidence Log`；Review Node 只能引用。
- `current_outcome_payload`：source of truth 是 upstream result 或 accountant correction；Review Node 只捕捉 accountant 对其 approval / correction。
- `review_actor`、`actor_id`、`raw_action_ref`：source of truth 是 accountant review interaction capture。
- `authority_limit_context`：source of truth 是 active policy、Profile / Entity / Rule / Governance authority、upstream hard blocks。
- `candidate_governance_context`：source of truth 是 upstream candidate signal、review correction 中暴露的 candidate，或 Post-Batch Lint context；不是 approved authority。
- `review_intervention_record`：source of truth 是 Review Node 捕捉到的 review action，但 authority 仅限 intervention / review interaction。

#### Fields that can never become durable memory by this node

Review Node 永远不能把以下字段直接写成长久业务记忆或治理状态：
- `display_group_key`
- `package_summary_context`
- `review_question`
- `evidence_explanation`
- `interaction_summary`
- `normalized_action_summary`
- `package_level_summary`
- `batch_review_summary_handoff.accountant_visible_summary`
- `candidate_reason`
- `proposed_value`
- `risk_summary`
- LLM-generated explanation / paraphrase
- accountant silence、ambiguous response、bulk acknowledgement

#### Fields that can become durable only after accountant / governance approval

以下内容最多由 Review Node 输出为 candidate 或 current-transaction correction context；只有经过相应 downstream authority path 后才可能变成 durable memory：
- profile change / profile confirmation
- new entity / entity merge / split
- alias approval / rejection
- role confirmation
- case memory update
- rule creation / promotion / modification / deletion / downgrade
- automation policy upgrade / relaxation / governance change
- accountant correction that implies long-term client policy

#### Audit vs learning / logging boundary

- `Transaction Log` 是 audit-facing final log，不参与 runtime review decision；Review Node 不写它。
- `review_intervention_record` 是 accountant interaction / review rationale record，不是 final audit log。
- `Case Log` 是 completed transaction 的 case memory；Review Node 不写它。
- `Rule Log` 只保存 accountant / governance approved deterministic rules；Review Node 不写 active rule。
- `Governance Log` 保存高权限长期记忆变化及审批状态；Review Node 只能转交 candidate，不批准或执行 mutation。
- `Knowledge Log / Summary Log` 是编译摘要；Review Node 的 batch summary 不等于 customer knowledge summary。

### Validation Rules

#### Contract-level validation rules

- 每个 review decision 必须绑定到明确 `review_item_id` 和 subject refs。
- Accountant approval / correction 必须有明确 actor、raw action ref、timestamp、evidence refs 和 item binding。
- Runtime summary、LLM explanation、display grouping、candidate signal 不能提升 authority。
- Output decision 必须尊重 input `authority_limit_context.allowed_decision_types`、`finalization_blocks` 和 `memory_mutation_blocks`。
- Current transaction finalization 与 long-term memory / governance candidate 必须分开输出。
- Review Node output 不能包含 JE lines、final Transaction Log entries、Case Log records、Rule Log records、stable Entity mutation、Profile mutation 或 approved Governance mutation。

#### Conditions that make the input invalid

Input invalid when：
- 缺少 `review_package_id`、`client_id`、`batch_id`、`review_items` 或 `authority_limit_context`。
- `review_items` 为空。
- item 没有明确 subject refs。
- current transaction item 缺少 `transaction_id`、`current_outcome_payload`、`upstream_rationale_refs` 或 `evidence_refs`。
- evidence refs 不可追溯，或 evidence 未绑定到 item。
- pending / intervention context 没有绑定到明确交易、问题或候选事项。
- package 声称 item 已 final logged 或 governance mutation 已执行。
- input 把 `Transaction Log` history 当成当前 runtime decision authority。
- input 把 candidate alias / role / entity / rule / policy 当成 approved durable authority。

#### Conditions that make the output invalid

Output invalid when：
- `approved_current_outcome` 缺少 explicit accountant approval。
- `corrected_current_outcome` 缺少 explicit accountant correction 或 correction payload。
- ambiguous accountant response、silence、bulk acknowledgement 或 LLM interpretation 被当作 approval。
- unresolved evidence、identity、authority 或 governance block 仍存在，却 route 到 JE generation。
- rejected / still-pending item 被 route 到 JE generation。
- governance-needed item 被输出为普通 approved current outcome 以绕过 Governance Review。
- candidate handoff 声称已经写入 long-term memory 或 approved governance event。
- review intervention record 包含 final Transaction Log write、JE lines、Case Log write、Rule Log write、Entity authority mutation 或 Profile mutation。
- batch summary 被当作 item-level approval 或 deterministic rule source。

#### Stop / ask conditions for unresolved contract authority

后续阶段或实现前必须 stop / ask when：
- Review Node 与 Governance Review Node 的 split 被要求承担 durable governance approval / execution，而 live docs 未冻结该 authority。
- 下游要求 Review Node 直接写 `Transaction Log`、`Case Log`、`Rule Log`、stable `Entity Log`、`Profile` 或 approved `Governance Log`。
- 需要决定 Review Node 是否审核所有交易、只审核 review-required items、还是支持 sampling / policy-based batch review 的产品范围。
- exact accounting payload schema 被要求冻结为 Review Node authority，而 upstream / JE contract 尚未定义。
- accountant bulk approval 是否能覆盖多个 item 缺少明确 item binding 或 authority policy。

## 4. Open Boundaries

### Stage 1: Open Boundaries

以下问题留到后续阶段，不在 Stage 1 冻结：
- Review Node 是否覆盖每一笔交易的批次审核，还是只覆盖 review-required / sampled / governance-candidate items。
- accountant approval、correction、rejection 和 still-pending 的 exact routing contract。
- Review Node 与 `Governance Review Node` 对治理候选的精确分工：哪些只展示和捕捉意向，哪些进入正式治理审批。
- Review decision 如何影响 JE Generation 的触发边界。
- Review decision 与 `Intervention Log`、`Transaction Log`、`Governance Log` 的 exact persistence boundary。
- 多笔交易和多个治理候选如何聚合、排序、去重和展示。
- exact input / output schema、字段名、对象结构、routing enum 和执行算法。

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
- Review Node 是否覆盖所有交易、只覆盖 review-required items、还是以 batch review / sampling / policy 组合触发
- accountant approval、correction、rejection、still-pending 与 JE Generation 的 exact handoff contract
- Review Node 与 `Governance Review Node` 在 accountant governance approval capture、governance event approval 和 durable mutation execution 上的 exact split
- review / intervention record 与 `Intervention Log` 的 exact contract
- review trace 如何被 `Transaction Logging Node` 纳入 final audit trail
- 多交易、多候选的聚合、去重、排序、展示和部分批准边界

### Stage 3: Open Contract Boundaries

以下问题仍未由 live docs 完全冻结，但不阻塞本 Stage 3 contract：
- Review Node 是否覆盖所有交易、只覆盖 review-required items、还是支持 sampling / policy-based batch review 的 exact product scope。
- Review Node 与 `Governance Review Node` 在 accountant governance approval capture、governance event approval 和 durable mutation execution 上的 exact split。
- `review_intervention_record` 与未来 `Intervention Log` implementation schema 的 exact persistence contract。
- `review_trace_ref` 如何被 `Transaction Logging Node` 纳入 final audit trail 的 exact downstream schema。
- 多交易、多候选 package 的 aggregation、dedup、sorting、partial approval 和 bulk action 规则。
- `current_outcome_payload` 中 COA / HST / amount allocation 的完整 accounting schema；本文件只规定 Review Node 如何消费和转交，不把 accounting payload source of truth 放到 Review Node。
