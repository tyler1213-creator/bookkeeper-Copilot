# Governance Review Node — 设计摘要

> 由原 Stage 1/2/3 合并瘦身而来；保留定位、边界、contract 字段和开放问题，删除阶段说明、examples、self-review、历史读档记录和重复解释。

## 1. 定位与职责

Stage 1 已确定以下内容：
- `Governance Review Node` 是 durable-memory governance gate。
- 它处理高权限长期记忆变化候选。
- 典型治理对象包括 entity merge / split、alias approval / rejection、role confirmation、rule promotion / modification / deletion / downgrade、automation policy change。
- accountant authority 仍然是长期治理变化的最终 authority。
- 候选信号不等于 durable authority。
- Case Judgment、Review、Transaction Logging 和 Case Memory Update 都不能直接批准治理变化。
- `new_entity_candidate` 不天然阻断当前交易分类，但不能未经治理成为稳定实体、支持 rule match 或自动创建 / 升级 rule。
- rule promotion 和 active rule 变化必须经过治理 / accountant approval。
- automation policy 升级或放宽必须 accountant approval；系统自动降级可以在受控边界内生效，但必须作为治理历史和 review visibility 处理。
- `Transaction Log` 是 audit-facing，不参与 runtime decision，也不因治理变化被重写。
- `Governance Log` 保存高权限长期记忆变化及其审批状态。

## 2. 逻辑与边界

### Trigger Boundary

`Governance Review Node` 在存在高权限长期记忆变化候选，且该候选需要治理审查或 accountant / governance decision 时触发。

概念触发条件是：
- 上游 workflow 已产生可追溯的治理候选、风险信号、accountant instruction、review correction、lint finding 或 memory consistency issue；并且
- 候选事项可能影响 entity authority、alias authority、role authority、rule lifecycle、automation policy 或其他长期自动化权威；并且
- 当前事项不是普通 runtime classification、pending clarification、JE generation、final transaction logging 或 case memory write；并且
- 该事项尚未被正式批准、拒绝、退回或记录为已处理治理结果。
典型触发来源包括：
- `Case Judgment Node`、`Review Node` 或 `Case Memory Update Node` 提出的 entity / alias / role / rule / automation-policy 候选。
- `Transaction Logging Node` 暴露的 candidate relation、logging consistency issue 或治理相关审计语境。
- `Post-Batch Lint Node` 发现的 entity merge / split 风险、rule instability、case-to-rule 候选或 automation risk。
- accountant 在 review、pending、intervention 或 onboarding 语境中提出的长期记忆修正意图。
- 系统自动降级 automation policy 后，需要 durable governance history 和 accountant visibility 的事项。
以下情况不能触发正式 governance approval：
- 当前交易 outcome 仍 pending、not-approved、JE-blocked、unresolved 或 conflicting。
- 候选缺少 evidence foundation、affected memory context 或 authority basis。
- accountant response 模糊，无法区分当前交易 correction 与长期治理指令。
- 事项只是 runtime rationale、review draft、report draft、candidate-only note 或未绑定证据的 lint warning。

### Input Categories

Stage 2 按判断作用组织 input categories，不列字段清单。

#### Governance candidate basis

说明当前候选是什么，以及它可能改变哪类长期 authority。

#### Evidence and traceability basis

说明候选依据来自哪些可追溯 evidence、交易、case、review、intervention、audit history 或 lint finding。

#### Current memory state basis

说明当前长期记忆处于什么状态，以及候选变化会影响什么。

核心边界：
- Entity Log 回答“是谁、别名、角色、状态、authority、automation policy”。
- Case Log 回答“过去真实完成案例怎样处理”。
- Rule Log 回答“哪些 deterministic rules 已被批准”。
- Governance Log 回答“高权限长期变化及审批状态”。
- Knowledge Summary 是可读摘要，不是 deterministic rule source。

#### Authority and approval basis

说明当前候选最多允许什么治理结果，以及谁拥有最终 authority。

#### Impact and downstream basis

说明候选若被批准，会影响哪些后续 workflow 和历史解释。

本节点必须区分：
- 可以影响未来 runtime authority 的长期变化。
- 只能作为历史解释保留的治理记录。
- 不能回写或改写历史 `Transaction Log` 的后续变化。

#### Conflict and risk basis

说明候选是否与 evidence、current memory、prior governance、accountant correction、case history、rule history、automation policy 或 audit history 冲突。

### Output Categories

Stage 2 只定义 conceptual output categories，不冻结 routing enum 或对象形状。

#### Governance approval / rejection / deferral record

含义：治理候选被正式记录为批准、拒绝、退回、暂缓或已通过受控系统降级生效。

边界：
- 这是 governance history。
- 它必须保留 approval / rejection / deferral 的 authority 和 reason。
- 它不等于当前交易 accounting approval。
- 它不重写历史 `Transaction Log`。

#### Authorized durable memory change

含义：在 accountant / governance authority 明确允许后，长期记忆或自动化权威可以发生变化。

典型变化类别包括：
- alias approval / rejection
- role confirmation
- entity merge / split / lifecycle handling
- rule creation / promotion / modification / deletion / downgrade
- automation policy change
边界：
- 只有被授权的治理变化可以影响 future runtime authority。
- exact mutation contract 留到后续阶段冻结。
- 历史交易日志不随治理变化被重写。
- `new_entity_candidate` 只有经过治理批准后，才可能获得稳定实体或相关 authority。

#### Governance blocked / insufficient handoff

含义：候选不能被批准，因为 evidence、authority、impact、conflict resolution 或 accountant decision 不足。

边界：
- Blocked handoff 是安全停止，不是低可信治理记录补丁。
- 本节点不能用“先批准以后再修”绕过 durable authority。
- 后续应返回 Review、Coordinator、Case Memory Update、Post-Batch Lint、上游 correction 或人工治理。

#### Candidate refinement / grouping handoff

含义：多个候选需要合并、拆分、去重、补证据或按业务含义重组后再进入正式 approval。

边界：
- Refinement 不是 approval。
- grouping 不是 evidence。
- 去重或聚合不能把多个弱信号自动变成强 authority。

#### Knowledge / lint handoff

含义：治理结果或未解决风险需要交给 `Knowledge Compilation Node` 或 `Post-Batch Lint Node`，用于未来摘要、监控或再检查。

边界：
- Knowledge handoff 不是客户知识摘要本身。
- Lint handoff 不是治理审批。
- 摘要和 lint finding 不能成为 deterministic rule source。

### Deterministic Code vs LLM Semantic Judgment

Stage 2 的核心边界是：

#### Deterministic code responsibility

Deterministic code 负责：
- 判断本节点是否被触发：是否存在长期 authority 相关治理候选，且尚未被正式处理。
- 区分 current transaction correction、case memory candidate、audit logging context、lint finding 和 formal governance candidate。
- 汇总候选与 evidence、case、transaction、intervention、review、memory state 和 prior governance 的绑定关系。
- 判断候选是否具备进入治理审查的最低 authority / traceability basis。
- 执行 hard blocks：未完成当前交易、缺少证据、accountant decision 模糊、impact 不明、冲突未解决、governance restriction 存在时不得批准 durable change。
- 防止 candidate alias / role / entity / rule / automation-policy 被误写为 active authority。
- 防止 `new_entity_candidate` 在未治理前支持 rule match、stable entity authority 或 rule promotion。
- 防止 rule creation、promotion、modification、deletion 或 downgrade 绕过 accountant / governance approval。
- 防止 automation policy upgrade or relaxation 绕过 accountant approval。
- 保持 Governance Log 与 durable memory mutation、rejection、deferral 或 auto-applied downgrade visibility 的边界清晰。

#### LLM semantic judgment responsibility

LLM 可以在 code 允许的边界内辅助：
- 总结候选的业务含义、证据链和影响范围。
- 比较候选与既有 Entity Log、Case Log、Rule Log、Governance history 的语义冲突。
- 将多个候选按业务含义聚合，提示可能的重复、拆分、合并或冲突。
- 解释 accountant natural-language instruction 是否看起来像当前交易 correction、长期治理意图，或仍需确认的模糊表达。
- 生成 approval / rejection / deferral explanation 的人类可读草案。

#### Hard boundary

LLM 不能：
- 批准 governance event
- 替 accountant 作长期治理决定
- 把模糊 accountant response 解释成 durable approval
- 把候选变成 stable entity、approved alias、confirmed role 或 active rule
- merge / split entity
- create / promote / modify / delete / downgrade active rule
- upgrade or relax automation policy
- 把 repeated cases 自动升级成 rule
- 改写 historical Transaction Log
- 选择 COA 科目或 HST/GST treatment
- 生成或修正 journal entry
- 把 governance approval 当成当前交易 accounting approval

### Accountant Authority Boundary

Accountant 仍然拥有最终 accounting decision 和 durable governance authority。

本节点不能：
- 把 accountant silence 解释为 approval。
- 把模糊自然语言解释为长期治理批准。
- 把当前交易 correction 自动变成 stable policy。
- 把 repeated completed outcomes 自动升级成 active rule。
- 把 candidate alias、candidate role 或 `new_entity_candidate` 自动批准。
- 把系统 lint finding 自动升级成放宽 automation policy 的依据。

### Governance Authority Boundary

Governance-level changes 是本节点的核心处理对象，但不是无条件 mutation 权限。

本节点可以在明确 authority 边界内处理：
- alias approval / rejection
- role confirmation
- entity merge / split / lifecycle handling
- rule creation / promotion / modification / deletion / downgrade
- automation policy downgrade / upgrade / relaxation review
- governance rejection、deferral、restriction 或 no-promotion decision
但本节点必须保持三条边界：
- 候选不等于 approval。
- approval 不等于执行算法或字段契约已经冻结。
- approved governance change 只影响 future authority；历史 `Transaction Log` 不被重写。

### Memory / Log Boundary

Stage 2 采用四层边界：read / consume、write allowed、candidate-only、no unauthorized mutation。

#### Read / consume boundary

`Governance Review Node` 可以读取或消费以下 conceptual context：
- Evidence Log references and objective evidence context
- Entity Log authority、alias、role、status、risk 和 automation-policy context
- Case Log completed-case precedent、exception 和 accountant correction context
- Rule Log active / historical rule authority 和 rule-health context
- Transaction Log finalized audit history，只作历史审计语境和 candidate grounding
- Intervention Log accountant question / answer / correction / confirmation context
- Governance Log prior approvals、rejections、deferrals、restrictions 和 auto-applied downgrade history
- Knowledge Log / Summary Log 的客户知识摘要，只作可读背景
- Profile context 中与 governance candidate 有关的结构事实或限制

#### Write allowed boundary

本节点可以写入或推动写入的 durable 内容只限于治理语义：
- 记录治理候选的正式处理结果。
- 记录 accountant / governance approval、rejection、deferral、restriction 或 auto-applied downgrade visibility。
- 在明确授权边界内推动对应长期记忆或自动化权威变化。
这些写入或 mutation 不等于：
- 当前交易 final accounting outcome
- `Transaction Log` write
- `Case Log` completed-case write
- JE generation
- report output

#### Candidate-only boundary

本节点仍只能作为候选或 handoff 表达：
- evidence re-check candidate
- current-transaction review correction candidate
- case memory correction / supersession candidate
- profile / tax config / account-mapping review candidate
- knowledge compilation candidate
- post-batch lint follow-up candidate
- unresolved governance risk candidate

#### No unauthorized mutation boundary

本节点绝不能：
- 在 accountant / governance authority 不足时修改 Entity Log、Rule Log、Governance Log、Profile 或 automation policy。
- 把 candidate alias 写成 approved alias。
- 把 candidate role 写成 confirmed role。
- 把 `new_entity_candidate` 写成 stable entity。
- 在未批准时 merge / split entity。
- 在未批准时 create / promote / modify / delete / downgrade active rule。
- 在未批准时 upgrade or relax automation policy。
- 把 governance note、review draft、lint finding、queue 或 candidate registry 称为 `Log`。
- 把 Transaction Log、Case Log 或 Knowledge Summary 变成 deterministic rule source。
- 重写 historical Transaction Log 来追随后续治理变化。

### Insufficient / Ambiguous / Conflicting Evidence Behavior

Stage 2 采用优先级边界：authority first、traceability second、impact clarity third、conflict preservation fourth。

#### Authority first

如果候选缺少明确 accountant / governance authority，本节点不能批准 durable change。

典型情况：
- accountant response 模糊或沉默。
- review correction 只影响当前交易，没有表达长期治理意图。
- candidate 来自 runtime judgment 或 lint finding，但未经过必要 review。
- automation policy upgrade / relaxation 缺少 accountant approval。
- rule promotion / active rule change 缺少 accountant approval。

#### Traceability second

如果候选无法绑定到 evidence、case、transaction、intervention、review decision、current memory state 或 prior governance context，本节点不能批准 durable change。

#### Impact clarity third

如果候选会影响 future entity resolution、rule match、case judgment、automation policy 或历史解释，但影响范围不清，本节点不能直接批准。

典型情况：
- merge / split 会影响多个实体、alias、rules 或 cases，但影响边界不清。
- rule promotion 的适用条件不清。
- role confirmation 会改变未来 rule eligibility，但 accountant intent 不清。
- automation policy change 会放宽自动化，但风险边界不清。

#### Conflict preservation fourth

如果候选与 raw evidence、completed cases、active rules、prior governance、accountant correction、Transaction Log audit history 或 Entity Log authority 冲突，本节点应保守处理。

典型边界：
- 冲突影响当前交易 final outcome：返回 Review / Coordinator / upstream correction，不做治理批准。
- 冲突影响长期记忆：保持治理候选或 deferral，直到冲突被解释或 accountant 明确决策。
- 冲突影响 historical audit explanation：不重写 Transaction Log，通过 governance history 解释后续变化。
- 冲突影响 rule authority 或 automation policy：不能用 LLM 选择一个版本直接生效。

#### Hard boundary

- Governance Review 是 durable authority gate，不是 runtime classifier。
- 候选不等于批准。
- accountant 当前交易 correction 不自动等于长期治理指令。
- `new_entity_candidate` 不天然阻断当前交易分类，但不创造 stable entity、approved alias、confirmed role 或 rule authority。
- Active rule 变化必须经过 accountant / governance approval。
- Automation policy 升级或放宽必须 accountant approval。
- Transaction Log 是 audit-facing，不参与 runtime decision，也不因治理变化被重写。
- 模糊、冲突、缺失或未批准状态不能被包装成 durable governance approval。

## 3. Contract 字段摘要

### Contract Position in Workflow

`Governance Review Node` 位于普通交易处理结果、review / intervention 语境、case learning、lint findings 与长期记忆 authority 变化之间：

#### Upstream handoff consumed

本节点消费 `governance_review_request`。

该 request 表示：
- 上游 workflow 已提出可能影响长期 authority 的治理候选；
- 候选已绑定 evidence、transaction、case、review、intervention、memory state、lint finding 或 accountant instruction 中至少一种可追溯 basis；
- 候选事项不是当前交易分类、pending clarification、JE generation、final Transaction Log write 或 completed-case write；
- 候选尚未被正式批准、拒绝、暂缓、退回、阻断或记录为已处理治理结果。
典型上游来源包括：
- `Entity Resolution Node`
- `Rule Match Node`
- `Case Judgment Node`
- `Coordinator / Pending Node`
- `Review Node`
- `Transaction Logging Node`
- `Case Memory Update Node`
- `Post-Batch Lint Node`
- `Onboarding Node`
- accountant instruction / intervention context

#### Downstream handoff produced

本节点可产生以下 output categories：
- `governance_decision_record`：写入或推动写入 `Governance Log` 的 durable governance history。
- `authorized_memory_change_handoff`：在 approval authority 足够时，表达允许对应长期记忆或 automation authority 改变的授权 handoff。
- `governance_blocked_handoff`：runtime-only 安全停止，说明候选不能被批准。
- `candidate_refinement_handoff`：runtime-only 候选重组、补证据、拆分、合并或澄清 handoff。
- `knowledge_lint_handoff`：runtime-only 下游交接，供 `Knowledge Compilation Node` 或 `Post-Batch Lint Node` 使用。

#### Logs / memory stores read

本节点可以读取或消费：
- `Evidence Log` references 和 objective evidence context；
- `Entity Log` 中 entity、alias、role、status、authority、risk flags、automation_policy 和 evidence links；
- `Case Log` 中 completed-case precedent、exception、accountant correction 和 case authority context；
- `Rule Log` 中 active / historical rule authority、rule lifecycle、rule conflicts 和 rule-health context；
- `Transaction Log` finalized audit history，但只能作为 audit-facing historical context，不能参与 runtime decision、learning 或 rule source；
- `Intervention Log` 中 accountant questions、answers、corrections、confirmations、objections 和 review context；
- `Governance Log` 中 prior approvals、rejections、deferrals、restrictions、auto-applied downgrades 和 authority rationale；
- `Knowledge Log / Summary Log` 中可读摘要，但不能作为 deterministic rule source；
- `Profile` 中与候选影响有关的结构事实或限制。

#### Logs / memory stores written or candidate-only

本节点可以写入或推动写入的 durable 内容只限治理语义：
- `Governance Log` / governance event history；
- accountant / governance approval、rejection、deferral、restriction 或 auto-applied downgrade visibility；
- approved governance event 对应的 authorized durable memory change handoff。
本节点不能写入：
- `Transaction Log`
- `Case Log`
- journal entry
- current transaction accounting outcome
- report output

### Input Contracts

#### `governance_review_request`

Allowed `trigger_source` values：
- `entity_resolution_node`
- `rule_match_node`
- `case_judgment_node`
- `coordinator_pending_node`
- `review_node`
- `transaction_logging_node`
- `case_memory_update_node`
- `post_batch_lint_node`
- `onboarding_node`
- `accountant_instruction`
- `workflow_orchestrator`
Allowed `trigger_reason` values：
- `entity_authority_candidate`
- `alias_authority_candidate`
- `role_authority_candidate`
- `entity_merge_split_candidate`
- `rule_lifecycle_candidate`
- `rule_conflict_or_health_issue`
- `automation_policy_candidate`
- `auto_applied_downgrade_visibility`
- `memory_consistency_issue`
- `accountant_governance_instruction`
- `post_batch_lint_governance_finding`
- `profile_tax_or_account_mapping_review_candidate`
Optional fields：
- `request_summary`：人类可读摘要；不是 authority。
- `related_review_package_refs`：相关 Review Node package / decision refs。
- `related_intervention_refs`：相关 accountant intervention refs。
- `related_transaction_log_refs`：相关 finalized audit history refs；只作 audit context。
- `runtime_trace`：orchestration diagnostics；不能作为 approval 或 evidence。
Validation / rejection rules：
- `governance_review_request_id`、`client_id`、`trigger_source`、`trigger_reason`、`candidate_items`、`authority_limit_context` 必须存在。
- `candidate_items` 不能为空。
- 每个 candidate 必须能绑定至少一种可追溯 durable reference：evidence、transaction、case、entity、rule、review、intervention、governance event、lint finding 或 onboarding source。
- 如果 request 要求本节点重新分类当前交易、生成 JE、写 Transaction Log 或写 completed Case Log，输入 invalid。
- 如果 request 声称 accountant silence、LLM explanation、frequency、similarity 或 lint warning 已经等于 governance approval，输入 invalid。
- `Transaction Log` refs 只能作为 audit-facing history，不能作为 runtime rule source 或 learning authority。
Runtime-only vs durable references：
- `governance_review_request` 是 runtime-only envelope。
- `client_id`、`transaction_id`、`evidence_refs`、`entity_ids`、`rule_ids`、`case_refs`、`review_refs`、`intervention_refs`、`governance_event_refs` 是 durable references。
- 接收该 request 不授予本节点未批准 mutation authority。

#### `governance_candidate_item`

Allowed `candidate_type` values：
- `approve_alias`
- `reject_alias`
- `confirm_role`
- `create_entity_from_candidate`
- `merge_entity`
- `split_entity`
- `archive_entity`
- `change_entity_status`
- `change_automation_policy`
- `record_auto_applied_downgrade`
- `create_rule`
- `promote_rule`
- `modify_rule`
- `downgrade_rule`
- `delete_rule`
- `rule_no_promotion_restriction`
- `profile_tax_or_account_mapping_review`
- `unresolved_governance_risk`
Allowed `candidate_status` values：
- `candidate_only`
- `requires_accountant_confirmation`
- `requires_governance_review`
- `pending_governance_decision`
- `previously_deferred`
- `previously_rejected`
- `auto_applied_downgrade_already_effective`
Optional fields：
- `candidate_summary`
- `conflict_basis`
- `risk_flags`
- `prior_governance_refs`
- `grouping_key`：展示或处理分组 helper；不是 evidence 或 approval。
- `supersedes_candidate_refs`
- `related_candidate_refs`
Validation / rejection rules：
- `candidate_status` 不能是 `approved`。已批准事项应作为 prior governance context 输入，而不是待批准 candidate。
- `candidate_type` 必须与 `proposed_change` 和 `subject_refs` 匹配。
- `approve_alias` / `reject_alias` 必须包含 affected alias surface text 和相关 `entity_id` 或 candidate entity ref。
- `confirm_role` 必须包含 proposed role/context 和相关 entity ref；runtime `candidate_role` 不能直接写入 stable role。
- `merge_entity` / `split_entity` 必须列出 affected entity ids 和影响范围；不能暗示 active rules、approved aliases 或 cases 会自动迁移。
- `create_rule` / `promote_rule` / `modify_rule` / `downgrade_rule` / `delete_rule` 必须绑定 rule candidate 或 existing rule ref、applicability basis 和 accountant / governance approval requirement。
- `change_automation_policy` 如果是 upgrade 或 relaxation，必须要求 accountant approval；系统不能自动放宽。
- `record_auto_applied_downgrade` 只能表示受控自动降级已生效并需要 governance history / Review visibility，不能表达 policy upgrade。
- `profile_tax_or_account_mapping_review` 当前只可作为 review candidate，除非后续 authority 明确纳入本节点治理 mutation scope。
Runtime-only vs durable references：
- `governance_candidate_item` 本身是 runtime / candidate handoff。
- `subject_refs` 和 evidence refs 指向 durable sources。
- 候选只有在 `governance_decision_record` 记录为 approved 或 auto-applied downgrade visibility 后，才可能影响长期 authority。

#### `evidence_traceability_basis`

Allowed `source_categories` values：
- `evidence_log_ref`
- `transaction_log_ref`
- `case_log_ref`
- `intervention_log_ref`
- `review_decision_ref`
- `entity_log_ref`
- `rule_log_ref`
- `governance_log_ref`
- `post_batch_lint_finding_ref`
- `onboarding_source_ref`
- `accountant_instruction_ref`
Allowed `traceability_status` values：
- `sufficient`
- `insufficient`
- `conflicting`
- `missing`
Optional fields：
- `contradicting_source_refs`
- `accountant_quote_refs`
- `transaction_ids`
- `case_refs`
- `lint_finding_refs`
- `source_quality_notes`
Validation / rejection rules：
- `supporting_source_refs` 不能为空，除非候选被明确输出为 blocked due to missing traceability。
- `evidence_summary` 不能替代 `supporting_source_refs`。
- `Transaction Log` refs 只能说明历史 audit context，不能作为 runtime decision source 或 rule source。
- `traceability_status != sufficient` 时不得输出 approved durable memory change。
Runtime-only vs durable references：
- 该对象是 runtime evidence packaging。
- refs 指向 durable records；summary 不成为 durable authority。

#### `current_memory_state_basis`

Allowed `memory_conflict_status` values：
- `none`
- `potential_conflict`
- `confirmed_conflict`
- `unknown`
Optional fields：
- `entity_state_snapshot`
- `alias_state_snapshot`
- `role_state_snapshot`
- `rule_state_snapshot`
- `automation_policy_snapshot`
- `profile_constraint_snapshot`
- `knowledge_summary_refs`
- `prior_governance_refs`
Validation / rejection rules：
- Knowledge Summary 与 source memory / Governance Log 冲突时，source memory / Governance Log authority wins。
- pending governance event 不能被当作 approved authority。
- rejected governance event 不能作为正向 approval basis 重用。
- 如果 current memory state 缺失到无法判断影响范围，应输出 blocked 或 refinement handoff，不得批准。
Runtime-only vs durable references：
- snapshot 和 summary 是 runtime packaging。
- Entity / Rule / Case / Governance / Profile refs 是 durable authority source。

#### `authority_basis`

Allowed `required_authority_level` values：
- `accountant_confirmation`
- `governance_approval`
- `system_controlled_downgrade_visibility`
- `owner_workflow_review`
- `not_approvable_from_current_context`
Allowed `authority_status` values：
- `sufficient_for_approval`
- `sufficient_for_rejection`
- `sufficient_for_auto_applied_downgrade_record`
- `insufficient`
- `ambiguous`
- `blocked_by_prior_governance`
Allowed `approval_actor_requirement` values：
- `accountant`
- `governance_owner`
- `system_controlled_downgrade`
- `profile_or_tax_or_account_mapping_owner`
- `none_candidate_only`
Optional fields：
- `accountant_id`
- `accountant_decision_ref`
- `approval_timestamp`
- `authority_notes`
- `prior_restriction_refs`
Validation / rejection rules：
- Accountant silence 不能成为 approval。
- 模糊 accountant natural-language response 不能被解释成 durable approval。
- 当前交易 correction 不自动等于长期治理 instruction。
- Rule creation / promotion / modification / deletion / downgrade 必须有 accountant / governance approval。
- Automation policy upgrade 或 relaxation 必须有 accountant approval。
- `authority_status != sufficient_for_approval` 时不得输出 approved durable memory change。
Runtime-only vs durable references：
- `authority_basis` 是 runtime authority projection。
- approval / restriction refs 指向 durable review、intervention 或 governance records。

#### `impact_and_conflict_basis`

Allowed `affected_workflow_areas` values：
- `future_entity_resolution`
- `future_rule_match`
- `future_case_judgment`
- `case_to_rule_eligibility`
- `automation_policy`
- `knowledge_compilation`
- `post_batch_lint_monitoring`
- `review_display`
- `audit_explanation`
- `profile_tax_or_account_mapping_review`
Allowed `impact_clarity_status` values：
- `clear`
- `partial`
- `unclear`
Allowed `conflict_status` values：
- `none`
- `potential_conflict`
- `unresolved_conflict`
- `resolved_by_accountant_decision`
- `blocked_by_conflict`
Optional fields：
- `conflicting_evidence_refs`
- `conflicting_case_refs`
- `conflicting_rule_refs`
- `conflicting_transaction_log_refs`
- `conflicting_governance_refs`
- `suggested_refinement_focus`
Validation / rejection rules：
- `impact_clarity_status != clear` 时不得批准 entity merge/split、rule lifecycle change 或 automation policy relaxation。
- `conflict_status` 为 `unresolved_conflict` 或 `blocked_by_conflict` 时不得批准 durable memory change。
- 冲突影响 historical audit explanation 时，不得重写 historical `Transaction Log`，只能通过 governance history 解释后续变化。
Runtime-only vs durable references：
- 该对象是 runtime impact / conflict packaging。
- conflict refs 指向 durable records，但不改变这些 records。

### Output Contracts

#### `governance_decision_record`

Allowed `event_type` values：
- `approve_alias`
- `reject_alias`
- `confirm_role`
- `create_entity_from_candidate`
- `merge_entity`
- `split_entity`
- `archive_entity`
- `change_entity_status`
- `change_automation_policy`
- `record_auto_applied_downgrade`
- `create_rule`
- `promote_rule`
- `modify_rule`
- `downgrade_rule`
- `delete_rule`
- `rule_no_promotion_restriction`
- `profile_tax_or_account_mapping_review`
- `unresolved_governance_risk`
Allowed `decision_status` values：
- `approved`
- `rejected`
- `deferred`
- `returned_for_clarification`
- `blocked`
- `auto_applied_downgrade_recorded`
Allowed `approval_status` values：
- `pending`
- `approved`
- `rejected`
- `auto_applied_downgrade`
Optional fields：
- `approval_status`：active `entity_governance_event` 使用的审批状态；entity-related event 必填。
- `resolved_at`
- `accountant_id`
- `entity_ids`
- `rule_ids`
- `case_refs`
- `transaction_ids`
- `affected_aliases`
- `affected_roles`
- `old_value`
- `new_value`
- `policy_change_direction`
- `restriction_scope`
- `supersedes_event_refs`
- `related_event_refs`
- `downstream_visibility`
Allowed `policy_change_direction` values：
- `downgrade`
- `upgrade`
- `relaxation`
- `restriction`
- `no_change_visibility_only`
Validation rules：
- `event_id` 是治理动作主键；`entity_ids`、`rule_ids` 或 `transaction_ids` 只能作为查询索引或 affected refs。
- Entity-related `governance_decision_record` 必须包含 `approval_status`，以对齐 active baseline 的 `entity_governance_event` 最小字段。
- `approved` 必须有 `authority_refs` 且 `authority_status = sufficient_for_approval`。
- `rejected` / `deferred` / `returned_for_clarification` / `blocked` 必须保留 reason。
- `auto_applied_downgrade_recorded` 只能用于系统受控降级已生效的 visibility / history；不能用于 upgrade 或 relaxation。
- Entity-related event 至少应能按 `event_id`、`entity_id`、`decision_status`、`event_type` 查询；若关联 transaction / case，也应能按对应引用查询。
- `merge_entity` / `split_entity` 的 record 不能声明 active rules、approved aliases 或 historical cases 已自动迁移，除非对应迁移也有明确 accountant / governance authority。
- `decision_status = approved` 不等于当前交易 accounting approval，也不重写 historical `Transaction Log`。
Durable memory effect：
- durable governance history。
- 只有 `approved` 或 `auto_applied_downgrade_recorded` 可以影响 future authority，但具体 memory mutation execution locus 尚未冻结。
- `rejected`、`deferred`、`returned_for_clarification` 和 `blocked` 只能作为治理历史、future warning、knowledge / lint context 或再审 context。

#### `authorized_memory_change_handoff`

Allowed `mutation_target` values：
- `entity_log`
- `rule_log`
- `governance_log_projection`
- `profile_or_tax_or_account_mapping_owner_workflow`
Allowed `authorized_change_type` values：
- `approve_alias`
- `reject_alias`
- `confirm_role`
- `create_stable_entity`
- `merge_entity`
- `split_entity`
- `archive_entity`
- `change_entity_status`
- `change_automation_policy`
- `create_rule`
- `promote_rule`
- `modify_rule`
- `downgrade_rule`
- `delete_rule`
- `record_restriction`
Optional fields：
- `effective_at`
- `migration_constraints`
- `do_not_migrate_refs`
- `post_change_review_visibility`
- `knowledge_compilation_hint`
- `post_batch_lint_monitoring_hint`
Validation rules：
- 必须引用 `decision_status = approved` 的 `governance_event_id`，除非是 `record_auto_applied_downgrade` 对应的已生效降级记录。
- `change_automation_policy` 如果方向是 `upgrade` 或 `relaxation`，必须有 accountant approval refs。
- Rule lifecycle change 必须有 accountant / governance approval refs。
- `new_entity_candidate` 只有通过 `create_stable_entity` 授权后，才可能成为 stable entity authority。
- `candidate_alias` 只有通过 `approve_alias` 授权后，才可能支持 future rule match。
- `candidate_role` 只有通过 `confirm_role` 授权后，才可能成为 stable role。
- `merge_entity` 不得删除旧 `entity_id`；旧 entity 应进入 governed lifecycle state，并通过治理事件解释后续关系。
- `split_entity` 不得自动全量迁移 historical cases；只能迁移 accountant 明确确认的案例。
Durable memory effect：
- 这是 authorized durable-memory change handoff。
- 本 Stage 3 不冻结本节点是直接执行 mutation，还是把该 handoff 交给专门 memory update workflow。

#### `governance_blocked_handoff`

Required fields：
- `blocked_handoff_id`
- `source_candidate_id`
- `block_reason`
- `blocked_refs`
- `required_resolution_path`
Allowed `block_reason` values：
- `missing_evidence_traceability`
- `insufficient_authority`
- `ambiguous_accountant_instruction`
- `current_transaction_not_finalized`
- `impact_scope_unclear`
- `unresolved_conflict`
- `blocked_by_prior_governance`
- `candidate_type_out_of_scope`
- `profile_tax_or_account_mapping_owner_required`
Allowed `required_resolution_path` values：
- `return_to_review_node`
- `return_to_coordinator_pending_node`
- `return_to_case_memory_update_node`
- `return_to_post_batch_lint_node`
- `return_to_upstream_owner_workflow`
- `request_accountant_governance_decision`
- `defer_until_more_evidence`
Optional fields：
- `suggested_clarification_question`
- `missing_refs`
- `conflict_summary`
- `risk_flags`
Validation rules：
- Blocked handoff 不能包含 approved event、authorized mutation 或 durable memory write claim。
- 如果原因是 authority / governance，不得伪装成 ordinary missing field。
- `suggested_clarification_question` 只能聚焦问题，不能替 accountant 作答。
Durable memory effect：
- runtime-only handoff。
- 可被下游记录为 intervention / review / governance context，但本对象本身不是 approval。

#### `candidate_refinement_handoff`

Required fields：
- `refinement_handoff_id`
- `source_candidate_ids`
- `refinement_need`
- `affected_refs`
- `required_next_review_scope`
Allowed `refinement_need` values：
- `group_candidates_by_business_meaning`
- `split_candidate_into_separate_events`
- `deduplicate_candidate_refs`
- `attach_missing_evidence_refs`
- `clarify_accountant_intent`
- `clarify_impact_scope`
- `resolve_memory_conflict`
Optional fields：
- `suggested_grouping_key`
- `candidate_relationship_summary`
- `proposed_refinement_notes`
Validation rules：
- Grouping、dedup 或 LLM semantic summary 不能把多个弱信号自动变成 strong authority。
- `suggested_grouping_key` 不能作为 evidence、approval 或 mutation key。
- refinement handoff 不能包含 `decision_status = approved`。
Durable memory effect：
- runtime-only / candidate-only。
- 是否持久保存为 candidate queue item 尚未冻结；即使持久保存，也不是 `Governance Log` approval。

#### `knowledge_lint_handoff`

Required fields：
- `handoff_id`
- `handoff_type`
- `source_governance_event_refs`
- `summary_for_downstream`
- `authority_effect`
Allowed `handoff_type` values：
- `approved_governance_summary`
- `rejected_governance_summary`
- `deferred_governance_summary`
- `unresolved_governance_risk`
- `auto_applied_downgrade_visibility`
- `post_change_monitoring_hint`
Allowed `authority_effect` values：
- `future_runtime_authority_changed`
- `history_only_no_runtime_authority`
- `candidate_only_no_authority`
- `monitoring_required`
Optional fields：
- `affected_entity_ids`
- `affected_rule_ids`
- `affected_case_refs`
- `review_visibility_note`
- `lint_monitoring_focus`
Validation rules：
- Knowledge handoff 不是 Knowledge Summary 本身。
- Lint handoff 不是 governance approval。
- Knowledge Summary 和 lint finding 不能成为 deterministic rule source。
- `authority_effect = future_runtime_authority_changed` 必须引用 approved governance event 或 auto-applied downgrade record。
Durable memory effect：
- runtime-only downstream handoff。
- 下游是否写 `Knowledge Log / Summary Log` 或 lint follow-up 由对应节点 contract 决定。

### Field Authority and Memory Boundary

#### Source of truth for important fields

- `transaction_id`：Transaction Identity Node / transaction identity layer。
- `evidence_refs`：`Evidence Log`。
- `entity_id`、alias state、role state、entity lifecycle status、automation_policy：`Entity Log` as governed by approved `Governance Log` events。
- `case_refs`、completed-case precedent、accountant correction context：`Case Log` / review-confirmed case memory authority。
- `rule_id`、active rule、rule lifecycle、rule conditions：`Rule Log` as governed by accountant / governance approval。
- `Transaction Log` refs：audit-facing finalized history only，不参与 runtime decision 或 learning。
- `Intervention Log` / Review refs：accountant question、answer、correction、confirmation、approval 或 objection 的 source。
- `Governance Log` / `governance_decision_record`：高权限长期记忆变化及审批状态的 source。
- `Knowledge Log / Summary Log`：可读摘要 source，不是 deterministic rule source。
- Profile / tax config / account-mapping context：对应 owner workflow / Profile / tax config authority；本节点是否可直接治理 mutation 尚未冻结。

#### Fields that can never become durable memory by this node

以下内容不能由本节点直接变成 stable Entity / Case / Rule / Profile memory：
- `governance_review_request`
- `governance_candidate_item`
- `request_summary`
- `candidate_summary`
- `evidence_summary`
- `memory_state_summary`
- `impact_basis` summary
- `conflict_basis` summary
- `grouping_key`
- `suggested_grouping_key`
- `runtime_trace`
- LLM-generated explanation
- lint finding
- review draft
- current transaction correction without explicit long-term governance instruction

#### Fields that can become durable only after accountant / governance approval

以下内容可以成为 long-term authority，但只能通过 accountant / governance-approved path：
- `new_entity_candidate` → stable entity。
- `candidate_alias` → approved alias。
- `candidate_role` → confirmed stable role。
- merge / split candidate → governed entity merge / split event。
- rule candidate → active rule。
- rule modification / downgrade / deletion → changed rule lifecycle。
- automation policy upgrade / relaxation → changed automation authority。
- no-promotion restriction or automation restriction → durable governance restriction。

#### Audit vs learning / logging boundary

`Transaction Log` 是 audit-facing final history，只写和查询，不参与本节点 runtime decision，也不是 learning layer 或 rule source。

### Validation Rules

#### Contract-level validation rules

- 每个 governance candidate 必须有 `candidate_type`、`subject_refs`、`proposed_change`、`evidence_traceability_basis`、`current_memory_state_basis`、`authority_basis` 和 `impact_basis`。
- Candidate 不得用 `Log` 命名，除非它已经写入 active baseline 定义的 durable `Governance Log`。
- `authority_basis` 必须先于 durable change validation；authority 不足时不得批准。
- `traceability_status` 必须足够；不能用 confidence、frequency、similarity 或 natural-language summary 填补证据链。
- `impact_clarity_status` 必须足够；影响范围不明时不能批准会影响 future runtime authority 的变化。
- conflict 必须保留；LLM 不得替 accountant 在冲突中选择 durable authority。
- 所有 approved rule lifecycle change 必须引用 accountant / governance approval。
- 所有 automation policy upgrade / relaxation 必须引用 accountant approval。
- 所有 entity merge / split 必须通过治理流程，并保留 governance event。

#### Conditions that make the input invalid

输入 invalid，如果：
- 请求本节点执行当前交易分类、COA 选择、HST/GST treatment 判断、JE generation 或 Transaction Log final write。
- 候选没有 subject refs 或 evidence / review / intervention / memory refs。
- accountant response 模糊、沉默或只表达当前交易 correction，却被标为 long-term approval。
- `candidate_status = approved` 却作为待审 candidate 输入。
- request 把 Transaction Log history 当成 runtime decision source、learning source 或 rule source。
- request 把 `new_entity_candidate`、candidate alias、candidate role、candidate rule 或 lint finding 当成 durable authority。

#### Conditions that make the output invalid

输出 invalid，如果：
- `governance_decision_record.decision_status = approved` 但缺少 authority refs。
- approved event 缺少 evidence links 或 reason。
- `authorized_memory_change_handoff` 未引用 approved governance event。
- `authorized_memory_change_handoff` 尝试改变 target 外的 durable store，或把 profile / tax / account-mapping change 当作本节点已冻结 mutation scope。
- merge / split 输出声称自动迁移 active rules、approved aliases 或 all historical cases。
- output 声称重写 historical `Transaction Log`。
- output 声称 governance approval 已批准当前交易 accounting outcome。
- candidate refinement 或 blocked handoff 包含 approved mutation。

#### Stop / ask conditions for unresolved contract authority

必须 stop / ask，而不是写成 final contract，如果当前任务要求：
- 冻结 Governance Review 是否直接执行 durable memory mutation。
- 冻结 Review Node 与 Governance Review Node 在 accountant approval capture 上的 exact split。
- 冻结 profile / tax config / account-mapping change 是否属于本节点直接治理 mutation scope。
- 冻结 rule promotion eligibility threshold、queue ranking、dedup、expiry、reopen 或 supersession behavior。
- 冻结 entity merge / split 后 cases、aliases、roles、rules 的完整 migration algorithm。

## 4. Open Boundaries

### Stage 1: Open Boundaries

以下问题留到后续阶段，不在 Stage 1 冻结：
- Governance Review 与 Review Node 对 accountant approval capture、formal governance approval 和 durable mutation execution 的 exact split。
- Governance Review 是否直接执行 approved durable memory mutation，还是只生成授权 handoff 给专门 memory update workflow。
- 哪些 profile / tax config / account-mapping change 属于本节点治理范围，哪些属于其他 profile governance workflow。
- 系统自动降级 automation policy 与 accountant 审批型治理事件的 exact boundary。
- rule candidate 由 Case Memory Update、Post-Batch Lint、Governance Review 还是组合流程提出的 exact division。
- rejected / deferred governance candidate 后续如何保留、再审或过期。
- governance decision 与 Knowledge Compilation、Post-Batch Lint、future runtime authority 的 exact handoff contract。
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
- Governance Review 与 Review Node 对 accountant approval capture、formal governance approval 和 durable mutation execution 的 exact split
- approved governance change 是由本节点直接执行，还是由专门 memory update workflow 执行
- governance candidate queue、grouping、dedup、expiry、reopen 和 supersession 的 exact behavior
- profile / tax config / account-mapping candidate 是否属于本节点治理范围
- auto-applied automation-policy downgrade 与 approval-required policy change 的 exact contract
- entity merge / split 后 cases、aliases、roles、rules 和 future matching 的 exact migration behavior
- rule promotion / modification / downgrade 的 exact eligibility contract
- rejected / deferred governance decision 对 future runtime warning、Knowledge Summary 和 Post-Batch Lint 的 exact handoff

### Stage 3: Open Contract Boundaries

- Governance Review 与 Review Node 对 accountant approval capture、formal governance approval 和 durable mutation execution 的 exact split 仍未冻结。
- Approved governance change 是由本节点直接执行 durable mutation，还是由专门 memory update workflow 执行，仍未冻结。
- Governance candidate queue、grouping、dedup、expiry、reopen、supersession 和 retry behavior 仍未冻结。
- Profile / tax config / account-mapping candidate 是否属于本节点直接治理 mutation scope 仍未冻结；当前 contract 只允许 candidate / owner workflow review handoff。
- Rule promotion / modification / downgrade 的 exact eligibility threshold、condition schema 和 rule-health contract 仍未冻结。
- Entity merge / split 后 cases、aliases、roles、rules 和 future matching 的完整 migration algorithm 仍未冻结。
- Rejected / deferred governance decision 对 future runtime warning、Knowledge Summary 和 Post-Batch Lint 的 exact expression 仍未冻结。
