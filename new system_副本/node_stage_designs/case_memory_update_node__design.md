# Case Memory Update Node — 设计摘要

> 由原 Stage 1/2/3 合并瘦身而来；保留定位、边界、contract 字段和开放问题，删除阶段说明、examples、self-review、历史读档记录和重复解释。

## 1. 定位与职责

Stage 1 已确定以下内容：
- `Case Memory Update Node` 是 completed-transaction case-learning node。
- 它在完成交易具备可学习 authority 后运行。
- 它把已完成交易沉淀为 `Case Log` 中的 case memory。
- `Case Log` 保存真实历史案例、最终分类、证据、例外和 accountant correction。
- 它可以提出 entity / alias / role / rule / automation-policy / governance 候选。
- 候选不等于 durable authority。
- 它不写 `Transaction Log`；final audit logging 属于 `Transaction Logging Node`。
- 它不执行 entity governance、rule promotion 或 automation-policy 放宽。
- 它不把 `Transaction Log` 变成 runtime decision source 或学习层。
- `new_entity_candidate` 可以支持候选实体和 case-memory 语境，但不能获得稳定实体或规则权限。

## 2. 逻辑与边界

### Trigger Boundary

`Case Memory Update Node` 在当前交易已经完成，并且具备可学习 authority 时触发。

概念触发条件是：
- 当前交易已有稳定 transaction identity 和可追溯 evidence foundation；并且
- 当前交易 outcome 已完成必要 review、correction、JE、final logging 或等价 finalization boundary；并且
- 当前交易不是 pending、still-pending、not-approved、JE-blocked、not-finalizable、unresolved、conflicting 或 governance-blocked；并且
- 当前完成交易尚未被安全沉淀为 case memory 或候选 handoff。
典型触发来源包括：
- structural path、rule path 或 case-supported path 已形成可学习的完成交易。
- pending / review 回收 accountant context 后，当前交易已完成并可沉淀。
- accountant correction 已明确绑定到当前交易，且后续 JE / logging boundary 已满足。
- final audit logging 后暴露出需要 Case Log 或治理候选处理的完成交易语境。
以下情况不能触发正常 case memory write：
- 当前交易仍缺 final outcome authority。
- 当前交易仍需 accountant clarification 或 review approval。
- 当前交易存在 unresolved identity、ambiguous evidence、conflicting evidence、JE-blocked、governance block 或 review-required condition。
- 当前内容只是 runtime judgment、candidate signal、review draft、report draft、lint finding 或 unapproved governance proposal。

### Input Categories

Stage 2 按判断作用组织 input categories，不列字段清单。

#### Completed transaction basis

说明当前要沉淀的是哪一笔已完成交易，以及它的客观交易基础、稳定身份和完成状态。

#### Final outcome and authority basis

说明当前交易最终怎样处理，以及该结果为什么具备可学习 authority。

核心边界：
- 只有已完成、可追溯、具备 authority 的 outcome 可以沉淀为 case。
- 未批准、仍 pending、仍冲突或仅为系统建议的内容不能写成完成案例。
- 当前交易 outcome 可以成为 case precedent，但不自动成为 rule 或 governance authority。

#### Evidence and exception basis

说明这笔完成交易为什么具有未来案例价值。

#### Entity and identity basis

说明完成交易当时指向哪个稳定实体、候选实体、alias、role 或 identity risk。

边界：
- 稳定 entity / approved alias / confirmed role 可以作为 case context 被引用。
- `new_entity_candidate` 可以随完成交易形成待治理 case 语境或候选信号。
- `new_entity_candidate` 不能因此变成 stable entity。
- candidate alias / candidate role 不能因此变成 approved alias 或 confirmed role。

#### Candidate and governance basis

说明这笔完成交易是否暴露出后续治理或规则候选。

#### Prior memory and duplication basis

说明这笔完成交易是否已经存在 case memory，或与既有 case / entity / rule / governance history 发生重复、冲突或 supersession risk。

### Output Categories

Stage 2 只定义 conceptual output categories，不冻结 routing enum 或对象形状。

#### Case memory write

含义：当前已完成交易被沉淀为 `Case Log` 中的真实历史案例。

边界：
- 这是 case-learning memory，不是 audit-facing `Transaction Log`。
- 它保存完成交易的 outcome、证据语境、例外语境和 correction / review context。
- 它支持未来 `Case Judgment Node` 的 case-based judgment。
- 它不参与 deterministic rule match。
- 它不等于 accountant 批准 rule。
- 它不等于 entity / alias / role / automation-policy governance approval。

#### Case memory skipped / blocked handoff

含义：当前交易不能安全沉淀为 case memory，因为完成状态、authority、证据、身份、review 或 consistency basis 不足。

边界：
- Blocked handoff 是安全停止，不是用低可信 case 填补学习层。
- 本节点不能用“先记成案例以后再修”绕过 authority 或 traceability。
- 后续应返回 Review、Coordinator、Transaction Logging、上游修正或 governance workflow。

#### Entity / alias / role candidate handoff

含义：完成交易暴露出可能需要新增实体、确认 alias、确认 role、合并 / 拆分实体或修正身份记忆的候选。

边界：
- 候选只表示后续应评估。
- 它不创建 stable entity。
- 它不 approve / reject alias。
- 它不 confirm role。
- 它不执行 merge / split。
- 它不改变 automation policy。

#### Rule / automation-policy candidate handoff

含义：完成交易暴露出可能需要 rule promotion、rule review、rule conflict handling、rule downgrade、automation-policy downgrade / upgrade review 或 no-promotion restriction 的候选。

边界：
- 本节点可以提出 rule 或 automation-policy 候选。
- 它不能 create / promote / modify / delete / downgrade active rule。
- 它不能 upgrade or relax automation policy。
- 系统自动降级 automation policy 的正式边界属于 lint / governance 相关 workflow，不在本节点内冻结。
- `case_allowed_but_no_promotion` 的实体不得被本节点推进为 rule promotion 候选。

#### Knowledge compilation handoff

含义：新 case 或候选信号可能使客户知识摘要需要更新。

边界：
- 该 handoff 只提示 `Knowledge Compilation Node` 后续编译。
- 它不是客户知识摘要本身。
- 摘要不能反过来成为 deterministic rule source。

#### Consistency issue signal

含义：本节点发现完成交易、case memory、entity context、rule context、review correction、audit history 或 governance history 之间存在不一致。

边界：
- 该 signal 不自行修正历史。
- 它不重写 `Transaction Log`。
- 它不修改 active memory。
- 它应进入 review、governance 或 lint 语境。

### Deterministic Code vs LLM Semantic Judgment

Stage 2 的核心边界是：

#### Deterministic code responsibility

Deterministic code 负责：
- 判断本节点是否被触发：当前交易是否已完成并具备可学习 authority。
- 检查 transaction identity、final outcome、JE / finalization context、review / correction trace 和 evidence foundation 是否足以支持 case memory write。
- 阻止 pending、not-approved、review-required、governance-needed、candidate-only、JE-blocked、not-finalizable 或 conflicting context 写入 Case Log。
- 区分 Case Log write、candidate handoff、consistency issue 和 blocked handoff。
- 防止同一完成交易重复写入 case memory，或在 reprocessing / correction 语境中产生互相矛盾的案例。
- 执行 Case Log 的 durable write discipline。
- 防止 `Transaction Log` 被当作 runtime decision source 或学习层。
- 防止本节点修改 Profile、stable Entity Log authority、Rule Log 或 Governance Log。
- 防止 candidate entity / alias / role / rule / automation-policy 被误写为 durable authority。
- 执行 `new_entity_candidate` 边界：可以形成候选和 case 语境，但不能支持 rule match、稳定实体创建或 rule promotion。

#### LLM semantic judgment responsibility

LLM 可以在 code 允许的边界内辅助：
- 总结完成交易中对未来案例有用的业务语境、证据语境和例外语境。
- 解释 accountant correction 或 review rationale 对 case precedent 的意义。
- 比较当前完成交易与既有 case memory 的语义关系，提示是否存在重复、例外或冲突风险。
- 识别可能的 entity / alias / role / rule / governance candidate signal，并生成解释。
- 将 case memory 的人类可读说明写得清楚、可追溯、不过度概括。

#### Hard boundary

LLM 不能：
- 决定当前交易是否具备可学习 authority
- 把 pending / not-approved / conflicting 交易写成完成案例
- 选择 COA 科目或 HST/GST treatment
- 生成或修正 journal entry
- 替 accountant approve current outcome
- 把 runtime rationale 当作 accountant correction
- 把 `new_entity_candidate` 变成 stable entity
- approve / reject alias
- confirm role
- merge / split entity
- create / promote / modify / delete / downgrade active rule
- upgrade or relax automation policy
- 批准 governance event
- 重写 `Transaction Log` 或既有 durable memory

### Accountant Authority Boundary

Accountant 仍然拥有最终 accounting decision 和 durable authority。

本节点不能：
- 把系统高置信结果解释为 accountant approval。
- 把 accountant silence、模糊回答或未完成 review 解释为 approval。
- 把当前交易 correction 自动变成长期客户政策。
- 把 repeated completed outcomes 自动升级成 active rule。
- 把 alias / role / entity clarification 自动批准为 durable authority。
- 把 case memory write 解释为 accountant 已批准后续治理候选。

### Governance Authority Boundary

Governance-level changes 不属于 `Case Memory Update Node` 的直接 mutation authority。

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
- rewrite historical cases to follow later governance state without approved process

### Memory / Log Boundary

Stage 2 采用四层边界：read / consume、write allowed、candidate-only、no direct mutation。

#### Read / consume boundary

`Case Memory Update Node` 可以读取或消费以下 conceptual context：
- current transaction identity、objective transaction basis 和 evidence references
- completed outcome from structural / rule / case / pending / review workflow
- JE completion / finalization context，只用于确认交易已完成和 case traceability
- final audit context from `Transaction Log` 或等价 finalization handoff
- upstream rationale from Profile / Structural Match、Entity Resolution、Rule Match、Case Judgment、Coordinator / Pending Node、Review Node 和 JE Generation Node
- `Intervention Log` 中与当前交易相关的 questions、answers、corrections、confirmations 和 review context
- Entity / Case / Rule / Governance context 中与当前 case 或候选相关的 authority、risk、history 和 restriction
- `Knowledge Log / Summary Log` 中的人类可读客户知识摘要，但只能作为 case wording 或 candidate context 的背景，不能作为 deterministic rule source

#### Write allowed boundary

本节点可以执行的 durable write 只限于 `Case Log` 语义：
- 把已完成交易沉淀为真实历史案例。
- 保留支持该案例的 evidence、outcome、exception、review / correction 和 authority context。
- 标记该案例在未来只提供 case precedent，而不是 deterministic rule authority。

#### Candidate-only boundary

本节点只能作为候选或 handoff signal 表达：
- entity creation / merge / split candidate
- alias approval / rejection candidate
- role confirmation candidate
- rule candidate、rule conflict candidate、rule-health candidate 或 case-to-rule candidate
- automation-policy downgrade / upgrade-review candidate
- profile / tax config / account-mapping review candidate
- knowledge compilation candidate
- post-batch lint candidate
- consistency issue signal

#### No direct mutation boundary

本节点绝不能：
- 写入 `Transaction Log`
- 生成或修改 journal entry
- 修改 `Profile`
- 写入或修改 stable `Entity Log` authority
- 写入或修改 `Rule Log`
- 直接写入或批准 `Governance Log` mutation
- 创建 stable entity
- approve / reject alias as durable authority
- confirm role as durable authority
- merge / split entity
- 修改 automation policy
- 创建、升级、修改、删除或降级 active rule
- 把 candidate、summary、review draft 或 lint finding 称为 `Log`
- 把 Case Log 或 Transaction Log 变成 deterministic rule source

### Insufficient / Ambiguous / Conflicting Evidence Behavior

Stage 2 采用优先级边界：completed authority first、case traceability second、governance caution third。

#### Completed authority first

如果当前交易尚未完成或不具备可学习 authority，本节点不能写正常 case memory。

典型情况：
- transaction outcome still pending
- review not approved
- accountant correction ambiguous
- JE generation not-finalizable
- final logging / finalization boundary 未满足
- unresolved / ambiguous identity 仍影响 current outcome
- 当前内容只是 candidate signal、runtime judgment 或 review draft
- governance block 存在

#### Case traceability second

如果 authority 允许继续，但交易身份、evidence references、review / correction trace、upstream rationale 或 final outcome 无法可靠绑定，本节点仍不能正常写 Case Log。

#### Governance caution third

如果完成交易暗示 entity、alias、role、rule、automation policy、Profile 或 tax config 需要长期变化，本节点应拆分：
- 当前完成交易可以作为 case memory 的部分。
- 长期变化只能作为 candidate 或 governance handoff。

#### Conflict behavior

如果完成交易 outcome 与 raw evidence、review correction、Entity Log、Case Log、Rule Log、Governance history、prior intervention 或 Transaction Log 发生冲突，本节点应保守处理。

典型边界：
- 冲突影响当前交易完成结果：阻止 case memory write，返回 review / upstream correction。
- 冲突只影响长期记忆：写入当前 case 的能力取决于 completed authority；长期变化只形成 candidate。
- 冲突影响 entity safety、rule authority 或 automation policy：形成 governance / lint candidate，不直接 mutation。
- 发现重复或 reprocessed transaction：不得产生互相矛盾的并列案例，具体 supersession 行为留到后续阶段。

#### Hard boundary

- Case memory 是学习层，不是审计日志。
- Transaction Log 是 audit-facing final log，不参与 runtime decision，也不是学习层。
- Case Log 可以支持未来 Case Judgment，但不能支持 deterministic rule match。
- Active rule 只能来自 accountant / governance approved Rule Log。
- `new_entity_candidate` 不天然阻断当前交易分类，但不创造 durable authority。
- Completed transaction outcome 可以成为 case precedent，但不能自动成为 rule、stable entity、approved alias、confirmed role 或 automation-policy change。
- 模糊、冲突、缺失或未批准状态不能被包装成可学习案例。

## 3. Contract 字段摘要

### Contract Position in Workflow

#### Upstream handoff consumed

本节点消费 `case_memory_update_request`。

该 request 必须由 workflow orchestration 在交易已经经过 finalization boundary 后形成，来源可以包括：
- completed structural / rule / case-supported outcome
- Review Node 的 accountant-approved 或 accountant-corrected current outcome handoff
- JE Generation completion context
- Transaction Logging Node 或等价 finalization handoff
- 与当前交易绑定的 Intervention Log / review / correction context
- 上游 entity、case、rule、governance candidate signals

#### Downstream handoff produced

本节点输出 `case_memory_update_result`。

下游消费者包括：
- `Case Judgment Node`：未来读取 `Case Log` 中的 case precedent；不读取本节点 runtime result 作为直接判断依据。
- `Governance Review Node`：消费 entity / alias / role / rule / automation-policy / governance candidate handoff。
- `Knowledge Compilation Node`：消费 knowledge compilation handoff，后续重新编译客户知识摘要。
- `Post-Batch Lint Node`：消费 consistency / rule-health / entity-risk signal。
- `Review Node` 或 `Coordinator / Pending Node`：在 blocked / invalid / unresolved authority 时接收回退信号。

#### Logs / memory stores read

本节点可以读取或消费引用：
- `Evidence Log`：原始证据和 evidence references。
- `Transaction Log` 或等价 finalization handoff：只用于确认当前交易已完成和审计 trace 可追溯；不作为未来 runtime decision source。
- `Intervention Log`：与当前交易绑定的 accountant question、answer、approval、correction、objection、review context。
- `Entity Log`：当前 entity / alias / role / automation policy 的 authority context。
- `Case Log`：用于 duplication、supersession、conflict risk 检查；不是让本节点重判当前交易。
- `Rule Log`：用于识别 rule conflict / rule candidate context；不执行 rule match。
- `Governance Log`：用于识别已生效 restriction 或 pending governance risk。
- `Knowledge Log / Summary Log`：仅作 case wording 或 candidate context 背景；不作为 deterministic rule source。

#### Logs / memory stores written or candidate-only

本节点唯一可直接 durable write 的语义目标是 `Case Log`：
- 写入真实 completed transaction case。
- 保留 final outcome、evidence、exception、review / correction、authority context。
- 明确该 case 只提供 future case precedent，不提供 deterministic rule authority。
本节点只能 candidate-only 输出：
- new entity / entity merge / entity split candidate
- alias approval / rejection candidate
- role confirmation candidate
- rule candidate / rule conflict / rule-health candidate
- automation-policy review candidate
- profile / tax config / account-mapping review candidate
- knowledge compilation handoff
- consistency issue signal
本节点不得写入或修改：
- `Transaction Log`
- `Profile`
- stable `Entity Log` authority
- `Rule Log`
- approved `Governance Log` mutation
- JE lines 或 accounting finalization record

### Input Contracts

#### `case_memory_update_request`

Validation / rejection rules：
- `transaction_id`、`completed_transaction_basis`、`learning_authority_context`、`final_outcome_context`、`evidence_context`、`traceability_context` 缺失时，input invalid。
- `learning_authority_context.learning_authority_status != eligible_for_case_write` 时，不能写正常 `case_log_record`。
- 如果 `completed_transaction_basis.completion_status` 不是已完成状态，本节点必须输出 blocked / invalid result。
- `final_outcome_context` 不能只有 runtime suggestion、review draft、candidate signal 或 LLM rationale。
- `transaction_logging_context` 如存在，只能作为 audit/finalization proof，不得作为 future learning source 或 runtime decision basis。
- `request_trace` 不能替代 evidence refs、review refs、finalization refs 或 accountant authority。

#### `completed_transaction_basis`

Allowed `completion_status` values：
- `completed_finalized`
- `completed_logged`
- `completed_with_equivalent_finalization`
- `blocked_not_completed`
- `invalid_completion_claim`
Allowed `completion_path` values：
- `structural_path`
- `approved_rule_path`
- `case_supported_path`
- `accountant_approved_review_path`
- `accountant_corrected_review_path`
- `manual_finalization_path`
Optional fields：
- `final_transaction_log_ref`：final Transaction Log ref；audit-facing only。
- `je_ref`：相关 JE ref；本节点不读取 JE line 来重判会计处理。
- `review_decision_ref`：review decision / intervention ref。
- `reprocessing_context`：如果是 reprocess / correction 后完成，说明原交易与 supersession risk。
Validation / rejection rules：
- `completion_status` 为 `blocked_not_completed` 或 `invalid_completion_claim` 时，不能写 Case Log。
- `completion_path = approved_rule_path` 时，必须能追溯到 approved active rule authority，但本节点不重新执行 rule match。
- `completion_path = accountant_corrected_review_path` 时，必须有明确 accountant correction ref。
- `reprocessing_context` 存在时，必须进入 duplication / supersession validation；不得并列写出互相冲突 case。

#### `learning_authority_context`

Allowed `learning_authority_status` values：
- `eligible_for_case_write`
- `candidate_handoff_only`
- `blocked_pending_or_review_required`
- `blocked_not_finalized`
- `blocked_authority_conflict`
- `blocked_traceability_gap`
- `invalid_authority_claim`
Allowed `authority_source` values：
- `explicit_accountant_approval`
- `explicit_accountant_correction`
- `approved_rule_finalization`
- `structural_finalization`
- `case_supported_finalization`
- `manual_finalization`
- `equivalent_finalization_boundary`
- `none`
Optional fields：
- `accountant_actor_ref`：如果 authority 来自 accountant action。
- `review_decision_ref`：review package / decision item ref。
- `governance_refs`：影响学习 authority 的 governance refs。
- `authority_notes`：简短说明；不能替代 refs。
Validation / rejection rules：
- `learning_authority_status = eligible_for_case_write` 时，`authority_source` 不能是 `none`，`authority_refs` 不能为空。
- 有 `hard_blocks` 时不能写 Case Log，除非 block 只影响长期治理候选且不影响当前 completed outcome authority；这种区分必须在 `hard_blocks` 或 `authority_notes` 中明确。
- accountant silence、bulk acknowledgement、模糊自然语言、system confidence 或 runtime recommendation 不能成为 `explicit_accountant_approval`。
- `candidate_handoff_only` 可以输出 candidate signals，但不得输出 `case_log_record`。

#### `final_outcome_context`

Allowed `outcome_type` values：
- `structural_result`
- `approved_rule_result`
- `case_supported_result`
- `accountant_approved_result`
- `accountant_corrected_result`
- `manual_result`
Optional fields：
- `classification_basis`：例如 structural、approved rule、case-supported、accountant correction。
- `coa_account_ref`：最终使用的 COA account reference；source authority 属于 upstream approved result / accountant correction。
- `hst_gst_treatment`：最终 tax treatment；source authority 属于 upstream approved result / accountant correction。
- `amount_allocation`：如有 split / allocation 的最终摘要；完整 JE schema 不在本节点定义。
- `case_refs_used`：如果 outcome 由 case-supported path 形成。
- `rule_ref_used`：如果 outcome 由 approved rule path 形成。
- `exception_summary`：与未来案例有关的例外语境。
Validation / rejection rules：
- `accounting_outcome` 不能为空；但本 Stage 不冻结共享 accounting schema。
- `coa_account_ref`、`hst_gst_treatment` 如存在，本节点只能复制为 case context，不能修正。
- `approved_rule_result` 必须引用 approved active rule；pending / rejected rule 不能支持 final outcome。
- `case_supported_result` 必须已越过后续 finalization / review boundary；不能把 Case Judgment runtime proposal 直接写成 final outcome。
- `accountant_corrected_result` 必须有明确 correction binding ref。

#### `evidence_context`

Allowed `evidence_sufficiency_status` values：
- `sufficient_for_case_write`
- `limited_but_authorized`
- `insufficient_for_case_write`
- `conflicting_evidence`
- `unbound_evidence`
Optional fields：
- `raw_description_ref`
- `receipt_refs`
- `cheque_info_refs`
- `accountant_context_refs`
- `evidence_limitations`
- `evidence_exception_notes`
Validation / rejection rules：
- `evidence_refs` 不能为空，除非 `learning_authority_context` 明确允许 `limited_but_authorized` 且有 authority ref。
- `conflicting_evidence` 或 `unbound_evidence` 不能写正常 case。
- `evidence_summary` 不能只有“模型认为相似”；必须绑定 evidence refs 或明确 evidence limitations。

#### `entity_identity_context`

Allowed `entity_resolution_status` values：
- `resolved_entity`
- `resolved_entity_with_unconfirmed_role`
- `new_entity_candidate`
- `ambiguous_entity_candidates`
- `unresolved`
Allowed `identity_case_binding_status` values：
- `stable_entity_context`
- `candidate_entity_context`
- `identity_resolved_by_review`
- `identity_limited_case_context`
- `identity_conflict_blocks_case_write`
Optional fields：
- `entity_id`：仅在 stable entity 安全识别时填写。
- `display_name_snapshot`：case 当时显示名称快照。
- `matched_alias`
- `alias_status`；allowed values: `candidate_alias`, `approved_alias`, `rejected_alias`。
- `confirmed_role_refs`
- `candidate_role`
- `candidate_entities`
- `identity_risk_flags`
- `related_governance_refs`
Validation / rejection rules：
- `entity_resolution_status = resolved_entity` 时，`entity_id` 必须引用 stable entity authority。
- `new_entity_candidate` 可以写入 Case Log 的 candidate identity context，但不得携带 stable `entity_id` authority，也不得支持 rule promotion。
- `candidate_role` 只能作为当前 case context 或 candidate signal，不能写入 stable Entity Log role。
- `alias_status = candidate_alias` 可作为 case context，但不能被写成 approved alias。
- `alias_status = rejected_alias` 如仍影响当前 outcome safety，应 block case write 或输出 consistency issue。
- `identity_case_binding_status = identity_conflict_blocks_case_write` 时不能写正常 case。

#### `traceability_context`

Allowed `binding_status` values：
- `all_required_refs_bound`
- `non_blocking_refs_missing`
- `blocking_refs_missing`
- `conflicting_refs`
Optional fields：
- `transaction_log_ref_check`
- `review_ref_check`
- `intervention_ref_check`
- `evidence_ref_check`
- `je_ref_check`
- `prior_case_duplication_check`
- `governance_ref_check`
Validation / rejection rules：
- `binding_status = blocking_refs_missing` 或 `conflicting_refs` 时不能写 normal case。
- `non_blocking_refs_missing` 只有在缺失项不影响 completed outcome authority 和 evidence traceability 时才可继续，并必须记录 limitation。
- prior duplicate / reprocessing / correction risk 不能被自然语言摘要绕过。

#### `candidate_signal_inputs`

Allowed `signal_type` values：
- `new_entity_candidate`
- `entity_merge_candidate`
- `entity_split_candidate`
- `alias_approval_candidate`
- `alias_rejection_candidate`
- `role_confirmation_candidate`
- `rule_candidate`
- `rule_conflict_candidate`
- `rule_health_candidate`
- `automation_policy_review_candidate`
- `profile_or_tax_config_review_candidate`
- `knowledge_compilation_candidate`
- `post_batch_lint_candidate`
- `case_consistency_issue`
Allowed `required_authority_path` values：
- `case_memory_update_only`
- `review_node`
- `governance_review`
- `post_batch_lint`
- `knowledge_compilation`
- `coordinator_pending`
- `not_eligible`
Optional fields：
- `proposed_candidate_payload`
- `source_node`
- `source_result_ref`
- `related_case_refs`
- `risk_level`
Validation / rejection rules：
- `subject_refs` 不能为空。
- candidate signal 不得声称 approved alias、confirmed role、stable entity、active rule、automation-policy upgrade 或 governance event 已生效。
- `new_entity_candidate` 不得输出 `rule_candidate` / rule promotion support；如有 repeated case signal，只能是治理/体检风险或 future review item。
- `case_allowed_but_no_promotion` context 不得生成 rule promotion candidate，除非 signal 明确表达 no-promotion restriction / risk。

### Output Contracts

#### `case_memory_update_result`

Allowed `result_status` values：
- `case_written`
- `case_written_with_candidate_handoffs`
- `candidate_handoff_only`
- `skipped_duplicate_or_superseded`
- `blocked_not_completed`
- `blocked_authority_gap`
- `blocked_traceability_gap`
- `blocked_conflict`
- `invalid_input`
Allowed `case_write_status` values：
- `written`
- `not_written_candidate_only`
- `not_written_duplicate`
- `not_written_blocked`
- `not_written_invalid`
Allowed `candidate_handoff_status` values：
- `none`
- `candidate_signals_forwarded`
- `candidate_signals_blocked`
- `candidate_signals_not_allowed`
Optional fields：
- `case_log_record`：当 `case_write_status = written` 时 required。
- `candidate_handoffs`：当 candidate signals 被转交时 required。
- `knowledge_compilation_handoff`
- `consistency_issue_signal`
- `blocked_context`
- `runtime_trace`
Validation rules：
- `result_status = case_written` 或 `case_written_with_candidate_handoffs` 时，必须有 `case_log_record`。
- `case_write_status = written` 时，input `learning_authority_status` 必须是 `eligible_for_case_write`。
- output 不能包含 Transaction Log write、Rule Log write、stable Entity Log mutation、Governance approval 或 JE write。
- `candidate_handoff_only` 不能被 downstream 当作 Case Log write。

#### `case_log_record`

Allowed `case_status` values：
- `active_case`
- `limited_authority_case`
- `superseded_case`
- `blocked_from_automation_use`
Allowed `case_authority.authority_type` values：
- `accountant_approved_case`
- `accountant_corrected_case`
- `approved_rule_finalized_case`
- `structural_finalized_case`
- `case_supported_finalized_case`
- `manual_finalized_case`
Allowed `case_precedent_scope.use_level` values：
- `case_judgment_context_only`
- `case_supported_classification_context`
- `exception_context_only`
- `negative_or_conflict_context`
Optional fields：
- `rule_ref_used`
- `case_refs_used`
- `exception_context`
- `risk_flags`
- `amount_context`
- `direction`
- `bank_account_ref`
- `receipt_refs`
- `cheque_info_refs`
- `accountant_notes_summary`
- `candidate_signal_refs`
- `supersedes_case_ids`
- `superseded_by_case_id`
- `limitations`
Validation rules：
- `case_id` 必须唯一。
- `transaction_id` 必须与 input 一致。
- `final_outcome_snapshot` 必须来自 approved / finalized source，不能来自 runtime suggestion。
- `evidence_refs` 必须可追溯到 Evidence Log。
- `review_correction_refs` 中的 accountant action 必须绑定当前 transaction / review item。
- `case_precedent_scope` 不得声称该 case 可支持 deterministic rule match。
- `new_entity_candidate` case 必须设置 `use_level` 为 `case_judgment_context_only`、`exception_context_only` 或更保守语义，且 `limitations` 必须说明不能支持 rule promotion / stable entity authority。

#### `entity_identity_snapshot`

Optional fields：
- `entity_id`
- `display_name_snapshot`
- `matched_alias`
- `alias_status`
- `confirmed_role_refs`
- `candidate_role`
- `candidate_entities`
- `identity_risk_flags`
- `related_governance_refs`
Validation rules：
- `entity_id` 只能在 stable entity authority 存在时出现。
- `alias_status = candidate_alias` 不得被重写为 approved。
- `candidate_role` 不得进入 stable roles。
- `new_entity_candidate` 不得声称 `entity_id` 是 stable authority。

#### `candidate_handoff`

Allowed `candidate_type` values：
- `new_entity_candidate`
- `entity_merge_candidate`
- `entity_split_candidate`
- `alias_approval_candidate`
- `alias_rejection_candidate`
- `role_confirmation_candidate`
- `rule_candidate`
- `rule_conflict_candidate`
- `rule_health_candidate`
- `automation_policy_review_candidate`
- `profile_or_tax_config_review_candidate`
- `knowledge_compilation_candidate`
- `post_batch_lint_candidate`
- `case_consistency_issue`
Allowed `required_authority_path` values：
- `governance_review`
- `post_batch_lint`
- `knowledge_compilation`
- `review_node`
- `coordinator_pending`
- `not_eligible`
Allowed `candidate_status` values：
- `proposed_candidate`
- `forwarded_candidate`
- `blocked_candidate`
- `not_eligible_candidate`
Optional fields：
- `case_id`
- `source_signal_ids`
- `proposed_candidate_payload`
- `risk_level`
- `blocking_reason`
- `related_governance_refs`
Validation rules：
- `subject_refs` 不能为空。
- `required_authority_path = governance_review` 的 candidate 不得被标记为 approved / applied。
- `candidate_type = rule_candidate` 时不得来自 `new_entity_candidate` 或 `case_allowed_but_no_promotion` context，除非明确是 no-promotion / rule-risk signal。
- `automation_policy_review_candidate` 不得改变 automation policy；升级或放宽必须 accountant / governance approval。

#### `knowledge_compilation_handoff`

Allowed `compilation_reason` values：
- `new_case_added`
- `case_exception_added`
- `case_correction_added`
- `entity_or_rule_candidate_added`
- `consistency_issue_detected`
Optional fields：
- `suggested_summary_focus`
- `affected_entity_ids`
- `affected_case_ids`
Validation rules：
- 该 handoff 不能包含已编译 knowledge summary。
- 不能声明 summary 已成为 deterministic rule source。

#### `blocked_context`

Allowed `blocked_reason` values：
- `transaction_not_completed`
- `learning_authority_missing`
- `accountant_approval_missing`
- `accountant_correction_ambiguous`
- `traceability_gap`
- `evidence_insufficient_or_conflicting`
- `identity_conflict`
- `governance_block`
- `duplicate_or_supersession_unresolved`
- `invalid_input_contract`
- `candidate_handoff_not_allowed`
Allowed `suggested_return_path` values：
- `review_node`
- `coordinator_pending`
- `transaction_logging_or_finalization`
- `evidence_reprocessing`
- `governance_review`
- `post_batch_lint`
- `manual_investigation`
Optional fields：
- `blocking_summary`
- `missing_fields`
- `conflicting_field_refs`
- `candidate_preservation_note`
Validation rules：
- blocked output must not include `case_log_record`.
- blocked output must not silently drop candidate signals if preserving them is authority-safe; if dropped, explain why.

### Field Authority and Memory Boundary

#### Source of truth for important fields

- `transaction_id`：Transaction Identity Node / transaction identity layer。
- raw evidence、receipt、cheque、accountant context refs：Evidence Log / Intervention Log。
- final accounting outcome：upstream finalized result、explicit accountant approval / correction、approved rule finalization、structural finalization 或 equivalent finalization boundary。
- `coa_account_ref`、`hst_gst_treatment`、amount allocation：上游 approved result / accountant correction / finalization authority；本节点只保存快照。
- `entity_id`、approved alias、confirmed role、entity status、automation policy：Entity Log + approved Governance authority。
- approved active rule：Rule Log + accountant / governance approval。
- governance restriction / event status：Governance Log。
- case precedent：Case Log，由本节点在 authority 合格时写入。
- Transaction Log：audit-facing final log，只写和查询，不参与 runtime decision，也不是 learning layer。

#### Fields that can never become durable memory by this node

本节点绝不能把以下内容写成长期 authority：
- stable entity creation / approval
- alias approval or rejection
- role confirmation
- entity merge / split / archive / reactivate
- active rule creation / promotion / modification / deletion / downgrade
- automation policy upgrade / relaxation / durable change
- Profile / tax config / account mapping mutation
- JE lines or final accounting computation
- Transaction Log entry
- Governance approval event
- LLM prompt trace、model confidence、candidate-only rationale

#### Fields that can become durable only after accountant / governance approval

以下内容可以由本节点提出候选，但必须经后续 authority path 才能成为长期 memory：
- `new_entity_candidate` → stable entity：Governance Review / approved entity governance。
- `alias_approval_candidate` / `alias_rejection_candidate` → approved / rejected alias：Governance Review / accountant approval。
- `role_confirmation_candidate` → confirmed role：Governance Review / accountant approval。
- `rule_candidate` / `rule_conflict_candidate` → active rule or rule change：Governance Review / accountant approval。
- `automation_policy_review_candidate` → automation policy change：governance workflow；升级或放宽必须 accountant approval。
- profile / tax config / account mapping review candidate → Profile durable change：appropriate review/governance owner。

#### Audit vs learning / logging boundary

`Transaction Log` 记录 final处理结果和审计轨迹；本节点可以读取它或等价 finalization handoff 来验证完成状态，但不能把 Transaction Log 当作未来 runtime learning source。

### Validation Rules

#### Contract-level validation rules

- 每个 request 必须绑定一个稳定 `transaction_id`。
- Case write 必须同时满足 completed transaction、learning authority、final outcome、evidence traceability、identity binding 和 duplication/supersession safety。
- 本节点只能写 `Case Log`；任何 Entity / Rule / Governance / Profile / Transaction Log mutation 字段都使 output invalid。
- `new_entity_candidate` 可以形成 case context 和 candidate handoff，但不能支持 rule match、rule promotion 或 stable entity authority。
- `candidate_alias`、`candidate_role` 只能作为 candidate/context，不能变成 approved alias / confirmed role。
- `Transaction Log` refs 只能用于 audit/finalization proof，不能作为 future runtime decision source。

#### Conditions that make input invalid

- 缺少 `transaction_id`、`learning_authority_context`、`final_outcome_context`、`evidence_context` 或 `traceability_context`。
- 当前交易仍 pending、review-required、not-approved、JE-blocked、not-finalized、identity unresolved in a blocking way、evidence conflicting 或 governance-blocked。
- outcome 只是 Case Judgment / LLM runtime proposal，没有 finalization authority。
- accountant response 未绑定当前交易或语义模糊，却被包装成 approval / correction。
- evidence refs 缺失、不可追溯或与 outcome 冲突。
- reprocessing / correction / duplicate risk 未解决。
- input 声称本节点应写 Rule Log、Entity Log、Governance Log、Profile、Transaction Log 或 JE。

#### Conditions that make output invalid

- `case_write_status = written` 但没有 `case_log_record`。
- `case_log_record.final_outcome_snapshot` 来自 runtime suggestion 或 candidate signal。
- `case_log_record.case_precedent_scope` 声称可以支持 deterministic rule match。
- candidate handoff 声称 approved / confirmed / promoted / applied。
- `new_entity_candidate` case 或 candidate 声称有 stable entity authority 或 rule promotion support。
- output 包含 Transaction Log write、Rule Log write、stable Entity Log mutation、Governance approval、Profile mutation 或 JE generation result。
- blocked output 同时包含正常 `case_log_record`。

#### Stop / ask conditions for unresolved contract authority

后续 Stage 或实现中如遇到以下问题，应 stop and ask，不得自行补 product authority：
- 需要决定哪些 high-confidence non-review outcomes 可直接获得 `eligible_for_case_write`。
- 需要冻结 shared `accounting_outcome` / JE-ready schema。
- 需要定义 durable candidate queue / governance event storage schema。
- 需要决定 duplicate / reprocessed / corrected / reversed / split transaction 的 exact supersession behavior。
- 需要允许本节点修改 Entity Log、Rule Log、Governance Log、Profile 或 Transaction Log。
- 需要改变 `Transaction Log` 不参与 runtime decision / learning 的边界。

## 4. Open Boundaries

### Stage 1: Open Boundaries

以下问题留到后续阶段，不在 Stage 1 冻结：
- completed-transaction learning authority 的 exact boundary。
- Case Memory Update 是否必须等 final `Transaction Log` 完成后运行，还是可消费等价 finalization handoff。
- 哪些 high-confidence system outcomes 可直接形成 case，哪些必须先经 accountant review。
- accountant correction、review approval、intervention context 与 case memory 的 exact handoff contract。
- `new_entity_candidate` 相关完成交易应如何在 case memory 中表达待治理身份语境。
- case memory write 与 governance candidate handoff 的 exact separation。
- case-to-rule candidate 何时由本节点提出，何时由 `Post-Batch Lint Node` 提出。
- exact input / output schema、字段名、对象结构、routing enum、存储路径、执行算法、测试矩阵和 coding-agent task contract。

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
- completed-transaction learning authority 的 exact contract
- Case Memory Update 与 final `Transaction Log` 的 exact trigger order
- high-confidence structural / rule / case-supported outcome 是否可不经逐笔 accountant review 直接形成 case
- accountant approval、correction、rejection、still-pending 与 Case Log write 的 exact handoff contract
- `new_entity_candidate` 相关完成交易在 Case Log 中的 exact authority expression
- duplicate / reprocessed / corrected / reversed / split transaction 对 case memory 的 exact behavior
- case-to-rule candidate 由本节点、Post-Batch Lint Node、Governance Review Node 还是组合流程提出

### Stage 3: Open Contract Boundaries

以下问题未被 live docs 完全冻结，但不阻塞本 Stage 3，因为本文件用 conservative authority envelope 把它们隔离为上游/后续决策：
- `completed-transaction learning authority` 的 exact upstream granting rule：哪些 high-confidence structural / rule / case-supported outcomes 可不经逐笔 accountant review 直接 `eligible_for_case_write`。
- Case Memory Update 与 final `Transaction Log` 的 exact trigger order：必须严格后置于 Transaction Logging Node，还是可以消费等价 finalization handoff。
- shared `accounting_outcome` / `final_outcome_snapshot` 的全局字段结构，尤其 COA、HST/GST、split、allocation 与 JE-ready payload 的边界。
- duplicate / reprocessed / corrected / reversed / split transaction 的 exact Case Log supersession behavior。
- candidate handoff 是否写入独立 durable queue、Governance Log pending event、Review package，或只作为 workflow handoff。
- case-to-rule candidate 的 primary owner：本节点可以提出保守候选，但 Post-Batch Lint / Governance Review 的分工还未完全冻结。
- `case_allowed_but_no_promotion` 下 rule-related signal 的 exact allowed shape：本文件只允许 no-promotion / risk signal，不允许 promotion candidate。
