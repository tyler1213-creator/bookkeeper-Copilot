# Post-Batch Lint Node — 设计摘要

> 由原 Stage 1/2/3 合并瘦身而来；保留定位、边界、contract 字段和开放问题，删除阶段说明、examples、self-review、历史读档记录和重复解释。

## 1. 定位与职责

Stage 1 已确定以下内容：
- 这个节点是 batch-level health check，不是 runtime classification。
- 它在批次结束后运行，检查跨交易和跨记忆层风险。
- 它识别 entity merge / split risk、rule instability、case-to-rule candidate 和 automation risk。
- 它可以产生 review / governance / knowledge follow-up 语境。
- 它不能重写 `Transaction Log`。
- 它不能把 `Transaction Log` 变成 runtime decision source 或 learning layer。
- 它不能自动改变 active rule。
- 它不能自动批准 entity、alias、role、merge 或 split。
- 它不能 upgrade 或 relax automation policy。
- Active baseline 允许它在受控边界内自动降低 entity automation policy，并把该风险降低动作暴露给 review / governance。

## 2. 逻辑与边界

### Trigger Boundary

`Post-Batch Lint Node` 在 batch-level boundary 下触发。

概念触发条件是：
- 一个批次已经形成足够的完成交易、review、intervention、case、rule、entity 或 governance 语境；并且
- 系统需要检查跨交易、跨记忆层或跨治理事项的自动化健康风险。
典型触发语境包括：
- 批次处理结束后的常规健康检查。
- Review / intervention 中出现重复 correction、反复 pending、rule conflict 或 automation-risk signal。
- Case Memory Update 暴露 case-to-rule candidate、case conflict、entity / alias / role candidate 或 automation-policy concern。
- Governance Review 产生已批准、拒绝、暂缓、限制或 auto-applied downgrade 相关语境，需要未来监控。
- Knowledge Compilation 暴露 summary mismatch、stale knowledge 或 unresolved boundary，需要回到 source memory 检查。

### Input Categories

Stage 2 按判断作用组织 input categories。这里不是字段清单。

#### Completed batch / audit-history basis

说明本批次实际完成了哪些交易、哪些交易被 review / correction / intervention 影响、哪些候选信号随 finalization 被保留下来。

边界：
- `Transaction Log` 只能作为已完成历史和 audit trace 被查询。
- 它不参与未来 runtime classification、rule match 或 case judgment。
- 它不是 learning layer，也不是 rule source。

#### Entity consistency basis

说明 entity memory 是否出现拆合、别名、角色、状态或自动化权限相关风险。

关键判断包括：
- 是否可能存在同一对象被拆成多个 entity。
- 是否可能存在一个 entity 混入不同业务角色或不同对象。
- alias / role candidate 是否反复出现但未治理。
- `new_entity_candidate` 是否积累出需要治理的 entity / alias / role 候选。
边界：
- Lint finding 不能自动 merge / split entity。
- Lint finding 不能把 candidate alias 或 candidate role 变成 approved / confirmed authority。
- `new_entity_candidate` 不因批次重复出现而自动变成 stable entity。

#### Rule health basis

说明 active rule 是否仍然安全、稳定、适用。

关键判断包括：
- active rule 是否反复被 accountant 修正。
- rule 条件是否过宽、过窄或与实际案例冲突。
- rule 是否依赖已经变化的 entity、alias、role 或 automation policy。
- rule miss / blocked pattern 是否暴露 rule candidate 或 rule review need。
边界：
- 本节点不能 create、promote、modify、delete 或 downgrade active rule。
- rule candidate、lint suggestion、repeated outcome、Case Log 或 Knowledge Summary 不能替代 `Rule Log` 中的 approved active rule。

#### Case-to-rule basis

说明一组 completed cases 是否表现出可能进入 rule governance 的稳定性。

关键判断包括：
- completed cases 是否在相同 entity / role / context 下表现出稳定处理。
- 是否存在明显例外条件，使 rule promotion 不安全。
- 是否存在 mixed-use、金额异常、tax treatment 差异、role ambiguity 或 review history，使规则化应被限制。
边界：
- case-to-rule 只能是 candidate。
- 它不等于 rule promotion。
- 它不能绕过 accountant / governance approval。

#### Review / intervention basis

说明 accountant 在本批次或近期介入中反复修正了什么、确认了什么、拒绝了什么、暂缓了什么。

边界：
- Accountant response 可以解释 lint finding，但不自动变成 durable governance mutation。
- 模糊回答不能被 lint pass 升级成 stable entity、active rule 或放宽后的 automation policy。

#### Governance / automation-policy basis

说明现有 governance history、automation policy、rule lifecycle 或 entity authority 是否限制未来自动化。

关键判断包括：
- 是否需要把 automation policy 降到更保守状态。
- 是否需要把 automation policy 升级或放宽候选交给 accountant。
- 是否需要把 unresolved governance risk 继续保留为 review / lint / knowledge follow-up。
边界：
- 自动降级 automation policy 是 active baseline 允许的风险降低动作。
- 升级或放宽 automation policy 必须 accountant approval。
- rule lifecycle 变化必须 accountant / governance approval。

#### Knowledge / summary basis

说明客户知识摘要是否暴露 stale、conflicting 或 missing context。

边界：
- Knowledge Summary 是可读背景，不是 deterministic rule source。
- 摘要与 source memory 冲突时，source memory / governance history 优先。
- 本节点不能通过修改摘要来改变 entity、case、rule 或 governance authority。

### Output Categories

Stage 2 只定义 conceptual output categories，不冻结 routing enum。

#### No-material-finding result

含义：本批次没有发现需要 review、governance、knowledge follow-up 或 automation-policy action 的 material issue。

边界：
- 这不是对未来自动化永久安全的保证。
- 它不改变任何 durable authority。

#### Review-facing lint finding

含义：发现需要 accountant 可见的批次级风险、趋势或未解决问题。

边界：
- Finding 不是 `Log`。
- Finding 不是 accountant approval。
- Finding 不是 durable memory mutation。

#### Governance candidate handoff

含义：发现可能需要 `Governance Review Node` 处理的高权限事项。

可覆盖语境包括：
- entity merge / split candidate
- alias approval / rejection candidate
- role confirmation candidate
- rule candidate or rule-health candidate
- automation-policy review candidate
- unresolved governance risk candidate
边界：
- Candidate 只表示后续治理应评估。
- 它不批准治理变化。
- 它不改变 active rule、stable entity authority 或 automation policy，除受控自动降级例外外。

#### Case-to-rule candidate

含义：completed cases 可能已稳定到值得进入 rule governance review。

边界：
- 它不是 rule。
- 它不进入 rule match。
- 它不能作为 `Rule Log` authority。

#### Rule-health restriction candidate

含义：active rule、rule condition、rule applicability 或 rule lifecycle 暴露风险，需要 review / governance 评估。

边界：
- 本节点不能直接修改、删除或降级 active rule。
- 如果需要改变 active rule，必须进入 governance / accountant approval。

#### Limited auto-applied automation-policy downgrade

含义：在 active baseline 允许的受控边界内，系统可以对 entity automation policy 做风险降低方向的自动降级，并把该动作暴露给 Review / Governance visibility。

边界：
- 只能是更保守的自动化权限。
- 不能放宽权限。
- 不能升级权限。
- 不能改变 active rule。
- 不能批准 entity、alias、role、merge 或 split。
- 不能替 accountant 做会计判断。
- 必须保持后续 accountant visibility 和 governance traceability。

#### Knowledge compilation follow-up

含义：lint finding、unresolved risk、stale summary 或 rejected / deferred governance context 需要进入未来客户知识摘要或监控背景。

边界：
- Knowledge follow-up 不能变成 deterministic rule source。
- Summary wording 不能把 candidate 写成 approved authority。

### Deterministic Code vs LLM Semantic Judgment

Stage 2 的核心边界是：

#### Deterministic code responsibility

Deterministic code 负责：
- 判断 Post-Batch Lint 是否可触发：是否存在完成批次或等价的 batch-level review / memory / governance context。
- 区分 source categories：Transaction audit history、Entity、Case、Rule、Intervention、Governance、Knowledge。
- 检查 source authority：approved、completed、candidate-only、rejected、deferred、auto-applied downgrade、unresolved、conflicting。
- 防止 transient handoff、queue、draft、candidate 或 report artifact 被称为 `Log`。
- 防止 `Transaction Log` 被用于 runtime decision、learning、rule source 或 governance approval。
- 判断哪些 finding 只能 review-facing，哪些可以进入 governance candidate，哪些可以进入 knowledge follow-up。
- 执行 active rule no-mutation boundary。
- 执行 entity merge / split no-mutation boundary。
- 执行 automation policy boundary：只能在允许范围内风险降低；升级或放宽必须 accountant approval。
- 防止 `new_entity_candidate`、candidate alias、candidate role 或 repeated cases 获得 stable authority。
- 防止 Knowledge Summary 或 lint finding 替代 source memory authority。

#### LLM semantic judgment responsibility

LLM 可以负责：
- 比较多个 cases 是否语义上属于同一业务场景。
- 解释 rule instability、case conflict、entity split / merge risk 或 automation-risk trend。
- 帮助识别候选事项的业务含义和 accountant-facing wording。
- 总结为什么某个 rule candidate 仍有例外风险。
- 总结为什么某个 entity 可能需要 merge、split、role confirmation 或 automation-policy restriction。

#### Hard boundary

LLM 不能：
- 扩大 automation authority。
- 放宽 automation policy。
- create / promote / modify / delete / downgrade active rule。
- merge / split entity。
- approve alias or role。
- 把 `new_entity_candidate` 变成 stable entity。
- 把 repeated cases 变成 active rule。
- 把 lint finding 写成 durable governance approval。
- 重写 historical `Transaction Log`。

### Accountant Authority Boundary

Accountant 仍然拥有最终 accounting decision 和 durable governance authority。

`Post-Batch Lint Node` 不能：
- 替 accountant 批准会计分类。
- 替 accountant 批准 rule promotion。
- 替 accountant 批准 entity merge / split。
- 替 accountant 批准 alias、role 或 stable entity authority。
- 替 accountant 放宽 automation policy。
- 把 accountant 模糊回答解释成长期 governance approval。

### Governance Authority Boundary

Governance-level changes 不属于本节点的自由裁量。

除此之外，它不能：
- approve / reject alias
- confirm role
- create stable entity from candidate
- merge / split entity
- create / promote / modify / delete / downgrade active rule
- upgrade or relax automation policy
- approve governance event
- invalidate durable memory
- rewrite historical audit history

### Memory / Log Boundary

Stage 2 采用四层边界：read / consume、candidate-only、limited mutation、no unauthorized mutation。

#### Read / consume boundary

`Post-Batch Lint Node` 可以读取或消费以下 conceptual context：
- `Transaction Log` 中的 finalized audit history，只作 batch-level historical context、traceability 和 risk analysis。
- `Intervention Log` 中的 accountant questions、answers、corrections、confirmations 和 unresolved intervention context。
- `Entity Log` 中的 entity、alias、role、status、authority、risk flags、automation policy 和 evidence links。
- `Case Log` 中的 completed cases、final classifications、evidence、exceptions、accountant corrections 和 review context。
- `Rule Log` 中的 approved active rules、rule lifecycle、historical rule authority 和 rule-health context。
- `Governance Log` 中的 approvals、rejections、deferrals、restrictions 和 auto-applied downgrade history。
- `Knowledge Log / Summary Log` 中的人类和 agent 可读摘要，但只能作为辅助背景。
- Profile context 中与 entity / rule / automation risk 有关的结构事实。

#### Candidate-only boundary

本节点可以提出候选信号，但不能执行对应高权限变化：
- entity merge / split candidate
- alias approval / rejection candidate
- role confirmation candidate
- new entity governance candidate
- case-to-rule candidate
- rule conflict / instability / stale rule candidate
- automation-policy upgrade / relaxation review candidate
- knowledge compilation follow-up candidate
- unresolved governance risk candidate

#### Limited mutation boundary

本节点唯一在 active baseline 中已知的受限 durable action 是：
- 对 entity automation policy 做风险降低方向的自动降级。
该边界必须同时满足：
- 只降低自动化权限，不放宽。
- 不改变 active rule。
- 不批准 entity / alias / role / merge / split。
- 不改变当前交易 accounting outcome。
- 不重写 historical `Transaction Log`。
- 对 accountant / governance review 保持可见。

#### No unauthorized mutation boundary

本节点绝不能：
- 写入或修改 `Transaction Log`
- 把 `Transaction Log` 变成 learning layer 或 runtime decision source
- 写入或修改 `Case Log`
- 写入或修改 `Rule Log`
- 修改 active rule
- 写入或修改 stable `Entity Log` authority，除受控 automation-policy downgrade 外
- approve alias、role、entity、merge 或 split
- upgrade or relax automation policy
- 写入或批准 governance approval
- 把 candidate、queue、review draft、lint finding 或 report draft 称为 `Log`
- 把 Knowledge Summary 当作 source authority

### Insufficient / Ambiguous / Conflicting Evidence Behavior

Stage 2 采用优先级边界：authority first、source sufficiency second、conflict caution third。

#### Authority first

如果事项涉及 active rule、entity authority、alias / role approval、merge / split、automation-policy upgrade / relaxation 或 governance approval，本节点不能自行批准。

#### Source sufficiency second

如果 finding 只来自未绑定 evidence 的自然语言说明、draft、queue item、单笔 runtime rationale 或未完成交易，本节点不能把它升级成 lint conclusion。

#### Conflict caution third

如果 source memory、Transaction Log audit history、Case Log、Rule Log、Entity Log、Governance history、Knowledge Summary 或 accountant correction 互相冲突，本节点应保守处理。

典型边界：
- Transaction Log 与后续 governance state 不一致：不重写历史，通过治理历史解释时点差异。
- Case Log 与 Rule Log 表达不同处理边界：区分 case precedent 与 active rule，不能用 case 直接覆盖 rule。
- Knowledge Summary 与 source memory 冲突：source memory / governance history 优先，摘要进入 follow-up。
- Lint finding 与 approved authority 冲突：保持 warning / candidate，不直接改变 authority。
- Automation-policy downgrade evidence ambiguous：不要静默降级，进入 review / governance candidate。

#### Hard boundary

- 批次级趋势不等于 governance approval。
- 重复案例不等于 active rule。
- Lint warning 不等于 runtime rule override。
- `new_entity_candidate` 不天然阻断当前交易分类，但不创造 stable entity、approved alias、confirmed role 或 rule authority。
- `Transaction Log` 是 audit-facing，不参与 runtime decision，也不是 learning layer。
- 风险降低可以保守；权限放宽必须审批。

## 3. Contract 字段摘要

### Contract Position in Workflow

#### Upstream handoff consumed

本节点消费 `post_batch_lint_request`。

典型上游语境来自：
- completed batch / audit-history basis
- Case Memory Update Node 的 completed case、candidate signal 或 consistency signal
- Review Node / Intervention Log 的 accountant correction、confirmation、rejection、deferred 或 unresolved context
- Governance Review Node 的 approval、rejection、deferral、restriction、auto-applied downgrade 或 pending governance context
- Entity Log、Rule Log、Case Log、Knowledge Log / Summary Log 的 source-memory context
- Knowledge Compilation Node 暴露的 stale summary、summary mismatch 或 unresolved boundary

#### Downstream handoff produced

本节点输出 `post_batch_lint_result`。

下游消费者包括：
- `Review Node`：消费 accountant-facing lint finding、batch-level risk summary 和 auto-downgrade visibility item。
- `Governance Review Node`：消费 entity merge / split、alias / role、rule health、case-to-rule、automation-policy 或 unresolved governance candidate。
- `Knowledge Compilation Node`：消费 knowledge follow-up，后续把 unresolved risk、monitoring concern 或治理结果编译进客户知识摘要。
- 未来 `Entity Resolution Node`、`Rule Match Node` 和 `Case Judgment Node`：只能通过已经生效的 durable memory / governance changes 间接受影响，不能读取 raw lint finding 作为 runtime authority。

#### Logs / memory stores read

本节点可以读取或消费以下 source-memory context：
- `Transaction Log`：finalized audit history、final outcome trace、review / correction trace；只作批次级历史和审计参考，不参与 runtime decision，也不是 learning layer。
- `Intervention Log`：accountant questions、answers、corrections、confirmations、objections、unresolved intervention context。
- `Entity Log`：entity、alias、role、status、authority、risk flags、automation_policy、evidence links。
- `Case Log`：completed cases、final classifications、evidence、exceptions、accountant corrections、review context。
- `Rule Log`：approved active rules、rule lifecycle、rule authority、rule-health context。
- `Governance Log`：approvals、rejections、deferrals、restrictions、auto-applied downgrade history、pending governance context。
- `Knowledge Log / Summary Log`：人和 agent 可读摘要；只能作为辅助背景，不能作为 deterministic rule source。
- `Profile`：与 entity / rule / automation risk 有关的客户结构事实。

#### Logs / memory stores written or candidate-only

本节点默认输出 runtime handoff 和 candidate signals，不默认创建新的 durable `Log`。

本节点可以 candidate-only 输出：
- entity merge / split candidate
- alias approval / rejection candidate
- role confirmation candidate
- new entity governance candidate
- case-to-rule candidate
- rule conflict / rule instability / stale rule candidate
- automation-policy upgrade / relaxation review candidate
- knowledge compilation follow-up candidate
- unresolved governance risk candidate
本节点唯一在 active baseline 中已知的受限 durable action 是：
- 对 entity `automation_policy` 做风险降低方向的自动降级。
该动作必须通过 governance-event authority path 表达：
- 写出或触发 `entity_governance_event` 语义记录；
- `event_type = change_automation_policy`；
- `source = lint_pass`；
- `requires_accountant_approval = false`；
- `approval_status = auto_applied_downgrade`；
- accountant / governance review 保持可见。
除此之外，本节点不得写入或修改：
- `Transaction Log`
- `Case Log`
- `Rule Log`
- stable `Entity Log` authority
- approved alias / confirmed role / stable entity status
- entity merge / split outcome
- approved governance mutation
- current transaction accounting outcome 或 JE

### Input Contracts

#### `post_batch_lint_request`

Allowed `lint_scope` values：
- `completed_batch`
- `recent_completed_batches`
- `governance_follow_up_scope`
- `knowledge_mismatch_scope`
- `targeted_entity_or_rule_scope`
- `mixed_batch_scope`
Allowed `trigger_reasons` values：
- `routine_post_batch_check`
- `repeated_review_correction`
- `repeated_intervention_pattern`
- `case_to_rule_signal`
- `entity_consistency_signal`
- `rule_health_signal`
- `automation_policy_risk_signal`
- `governance_follow_up`
- `knowledge_summary_mismatch`
- `accountant_requested_lint`
Validation / rejection rules：
- `request_id`、`client_id`、`batch_id`、`lint_scope`、`trigger_reasons`、`completed_batch_basis`、`source_authority_context` 缺失时，input invalid。
- `trigger_reasons` 必须至少包含一个 allowed value。
- `completed_batch_basis` 必须证明存在 completed transaction、completed review / intervention context、durable memory context 或 governance follow-up context；只有未完成 runtime suggestion 时 input invalid。
- `Transaction Log` 只能作为 finalized audit history 查询语境，不得被声明为 learning source、rule source、runtime decision source 或 governance approval source。
- `request_trace`、draft、queue item、report draft 或自然语言 note 不能替代 source memory refs、evidence refs、review refs 或 governance refs。

#### `completed_batch_basis`

Allowed `batch_completion_status` values：
- `completed_batch_closed`
- `completed_partial_scope_closed`
- `governance_follow_up_scope_ready`
- `knowledge_mismatch_scope_ready`
- `invalid_not_completed`
Optional fields：
- `final_transaction_log_refs`：Transaction Log refs；audit-facing only。
- `review_decision_refs`：本批次相关 review decision / intervention refs。
- `je_refs`：completed JE refs；本节点不读取 JE line 来重判会计处理。
- `excluded_transaction_refs`：因未完成、pending、invalid 或 out-of-scope 被排除的交易 refs。
- `prior_batch_refs`：近期批次 ref，用于趋势分析；不是无限历史扫描授权。
Validation / rejection rules：
- `batch_completion_status = invalid_not_completed` 时，不得输出 material lint conclusion，只能输出 blocked / invalid result。
- `completed_transaction_refs` 不能为空，除非 `lint_scope` 是 `governance_follow_up_scope` 或 `knowledge_mismatch_scope` 且相关 durable source refs 足够。
- `final_transaction_log_refs` 不能作为 future runtime classification、rule promotion 或 learning authority。
- `excluded_transaction_refs` 中的 pending / not-finalized 交易不能被计入 rule instability、case-to-rule 或 auto-downgrade 依据。

#### `source_authority_context`

Allowed `source_category` values：
- `transaction_audit_history`
- `intervention_history`
- `entity_memory`
- `case_memory`
- `rule_memory`
- `governance_history`
- `knowledge_summary`
- `profile_context`
- `runtime_candidate_signal`
Allowed `authority_state` values：
- `approved_authority`
- `completed_history`
- `accountant_confirmed`
- `candidate_only`
- `rejected`
- `deferred`
- `pending_governance`
- `auto_applied_downgrade`
- `unresolved`
- `conflicting`
- `summary_only`
Allowed `allowed_use` values：
- `audit_context_only`
- `risk_analysis_context`
- `candidate_generation_allowed`
- `governance_candidate_allowed`
- `knowledge_follow_up_allowed`
- `auto_downgrade_basis_allowed`
- `not_authoritative`
Optional fields：
- `authority_notes`：简短说明；不能替代 refs。
- `conflict_refs`：发生 source conflict 时的相关 refs。
- `source_freshness`：source 是否 stale；仅作 validation / risk context。
Validation / rejection rules：
- `source_category = transaction_audit_history` 时，`allowed_use` 不能是 `auto_downgrade_basis_allowed` 的唯一依据，也不能是 rule source / learning source。
- `authority_state = candidate_only`、`rejected`、`deferred`、`unresolved`、`summary_only` 时，不得作为 durable mutation authority。
- `Knowledge Summary` 与 source memory 冲突时，`source_category = knowledge_summary` 必须被标记为 `summary_only` 或 `conflicting`，并只能产生 knowledge follow-up。
- 若 `hard_authority_blocks` 影响 active rule、entity authority、merge/split、alias/role approval 或 automation-policy relaxation，输出必须保持 candidate / review-needed / governance-needed 语义。

#### `entity_consistency_basis`

Optional fields：
- `new_entity_candidate_refs`
- `candidate_alias_refs`
- `candidate_role_context`
- `merge_candidate_inputs`
- `split_candidate_inputs`
- `recent_correction_refs`
- `automation_policy_context`
- `prior_governance_refs`
Validation / rejection rules：
- 没有 `entity_refs` 或 candidate refs 时，不能输出 entity-specific finding。
- `new_entity_candidate_refs` 不能被视为 stable entity authority，不能支持 rule match 或 rule promotion。
- `candidate_alias_refs`、`candidate_role_context` 不能被写成 approved alias 或 confirmed role。
- merge / split 风险只能输出 candidate；不得改变 entity lifecycle status 或历史 case 归属。
- automation-policy downgrade 必须引用 stable active entity；不能对 unresolved、candidate-only 或 ambiguous entity 静默降级。

#### `rule_health_basis`

Optional fields：
- `rule_blocked_or_ineligible_refs`
- `rule_miss_pattern_context`
- `conflicting_case_refs`
- `prior_rule_health_candidate_refs`
- `governance_restriction_refs`
Validation / rejection rules：
- `rule_refs` 必须引用 approved / durable rule context；candidate rule 只能进入 `case_to_rule_basis` 或 governance candidate context。
- 本节点不能 create、promote、modify、delete 或 downgrade active rule。
- `rule_correction_refs` 可支持 rule-health finding，但不能自动改变 rule lifecycle。
- rule instability 如果只来自 unfinalized transaction、summary wording 或 unsupported natural language note，不能形成 material rule-health conclusion。

#### `case_to_rule_basis`

Optional fields：
- `entity_ref`
- `confirmed_role_refs`
- `candidate_role_context`
- `supporting_evidence_refs`
- `conflicting_case_refs`
- `review_correction_refs`
- `automation_policy_context`
- `knowledge_summary_refs`
Validation / rejection rules：
- `case_refs` 必须是 completed cases；pending、draft 或 not-finalized cases 不能计入 promotion candidate。
- `new_entity_candidate`、candidate alias 或 candidate role 可以作为 risk/context，但不能支持 rule promotion authority。
- `case_to_rule_basis` 只能输出 candidate；不得写 `Rule Log` 或创建 active rule。
- `Knowledge Summary` 只能帮助 wording / context，不得替代 `Case Log` refs。

#### `review_intervention_basis`

Allowed `accountant_action_state` values：
- `explicit_correction`
- `explicit_confirmation`
- `explicit_rejection`
- `explicit_deferral`
- `still_pending`
- `ambiguous_response`
- `unresolved_objection`
Optional fields：
- `correction_pattern_summary`
- `related_rule_refs`
- `related_entity_refs`
- `related_case_refs`
- `prior_pending_question_refs`
- `governance_candidate_refs`
Validation / rejection rules：
- Accountant 模糊回答不能被解释成 stable entity、active rule、alias approval、role confirmation、merge/split approval 或 automation-policy relaxation。
- Explicit correction 可以支持 review-facing finding、rule-health candidate、case conflict context 或 automation-policy risk context，但不能自动写 active rule。
- `still_pending` 或 `unresolved_objection` 只能支持 review / governance needed 语义，不能支持 auto-downgrade，除非还有独立 durable evidence 满足降级边界。

#### `governance_automation_basis`

Allowed `automation_policy` values：
- `eligible`
- `case_allowed_but_no_promotion`
- `rule_required`
- `review_required`
- `disabled`
Allowed `approval_status_context` values：
- `pending`
- `approved`
- `rejected`
- `deferred`
- `auto_applied_downgrade`
- `unresolved`
- `conflicting`
Optional fields：
- `prior_auto_downgrade_event_refs`
- `restriction_refs`
- `contested_downgrade_refs`
- `relaxation_or_upgrade_candidate_refs`
- `governance_notes`
Validation / rejection rules：
- automation-policy downgrade 只能从较宽松状态转向更保守状态。
- automation-policy upgrade / relaxation 必须输出 review / governance candidate，不能自动生效。
- rule lifecycle 变化必须 accountant / governance approval；本节点不能直接改变 active rule。
- rejected / deferred governance decision 不得被 lint 重新包装成 approved authority。

#### `knowledge_summary_basis`

Allowed `summary_issue_type` values：
- `stale_summary`
- `summary_source_conflict`
- `missing_unresolved_boundary`
- `overstated_candidate_as_authority`
- `missing_governance_result`
- `insufficient_source_refs`
Optional fields：
- `affected_entity_refs`
- `affected_rule_refs`
- `affected_case_refs`
- `affected_governance_refs`
- `suggested_follow_up_scope`
Validation / rejection rules：
- Knowledge Summary 不能作为 deterministic rule source。
- Summary 与 source memory 冲突时，source memory / governance history 优先，输出应为 knowledge follow-up，不得直接改 entity、case、rule 或 governance authority。
- 如果 summary 缺少 source refs，不能用 summary wording 支持 material lint conclusion。

### Output Contracts

#### `post_batch_lint_result`

Allowed `lint_result_status` values：
- `no_material_finding`
- `findings_emitted`
- `candidates_emitted`
- `auto_downgrade_applied`
- `mixed_result`
- `blocked_invalid_input`
- `blocked_unresolved_authority`
Optional fields：
- `review_facing_findings`
- `governance_candidate_handoffs`
- `case_to_rule_candidates`
- `rule_health_restriction_candidates`
- `auto_applied_automation_policy_downgrades`
- `knowledge_compilation_follow_ups`
- `blocked_reason`
- `result_trace`
Validation rules：
- `lint_result_status = no_material_finding` 时，不得同时包含 material findings、governance candidates 或 auto-applied downgrades。
- `lint_result_status = auto_downgrade_applied` 或 `mixed_result` 且包含 downgrade 时，必须包含对应 `auto_applied_automation_policy_downgrades`。
- `blocked_invalid_input` 或 `blocked_unresolved_authority` 时，不得输出 durable mutation payload。
- `authority_boundary_summary` 必须明确 raw lint finding 不可作为 runtime classification、rule source 或 governance approval。

#### `review_facing_lint_finding`

Allowed `finding_type` values：
- `entity_merge_risk`
- `entity_split_risk`
- `alias_role_governance_need`
- `new_entity_governance_need`
- `rule_instability`
- `case_to_rule_opportunity`
- `rule_health_restriction_need`
- `automation_policy_risk`
- `stale_knowledge_or_summary_conflict`
- `repeated_intervention_pattern`
- `unresolved_governance_risk`
- `insufficient_evidence_monitoring`
Allowed `severity` values：
- `info`
- `monitor`
- `review_needed`
- `governance_needed`
- `high_risk`
Optional fields：
- `related_candidate_ids`
- `related_auto_downgrade_event_id`
- `source_conflict_summary`
- `llm_semantic_summary`
Validation rules：
- `subject_refs` 和 `evidence_refs` 不能同时为空。
- `finding_summary` 不能把 candidate 写成 approved authority。
- `recommended_review_action` 不能要求 accountant 批准 `authority_limit` 禁止的结果。
- `llm_semantic_summary` 只能解释风险，不能成为 mutation authority。

#### `governance_candidate_handoff`

Allowed `candidate_type` values：
- `entity_merge_candidate`
- `entity_split_candidate`
- `alias_approval_candidate`
- `alias_rejection_candidate`
- `role_confirmation_candidate`
- `new_entity_governance_candidate`
- `rule_candidate`
- `rule_conflict_candidate`
- `rule_instability_candidate`
- `automation_policy_review_candidate`
- `automation_policy_upgrade_or_relaxation_candidate`
- `unresolved_governance_risk_candidate`
- `knowledge_authority_cleanup_candidate`
Allowed `required_authority` values：
- `accountant_approval_required`
- `governance_review_required`
- `review_visibility_required`
- `knowledge_compilation_required`
Allowed `candidate_status` values：
- `candidate_open`
- `candidate_deferred`
- `candidate_blocked_insufficient_evidence`
- `candidate_superseded`
Optional fields：
- `risk_flags`
- `conflict_refs`
- `related_review_finding_id`
- `proposed_event_preview`
Validation rules：
- Candidate 必须绑定具体 `subject_refs`；不能只有自然语言建议。
- `proposed_event_preview` 只能作为 preview，不是 executed governance event。
- `automation_policy_upgrade_or_relaxation_candidate` 永远不能自动生效。
- merge / split、alias approval / rejection、role confirmation、new entity stabilization、rule lifecycle change 都必须保持 candidate 语义。

#### `case_to_rule_candidate`

Optional fields：
- `confirmed_role_refs`
- `candidate_role_context`
- `supporting_evidence_refs`
- `conflicting_case_refs`
- `review_correction_refs`
- `automation_policy_context`
- `related_governance_candidate_id`
Validation rules：
- `case_refs` 必须全部为 completed cases。
- `entity_ref` 必须引用 stable entity；如果只有 `new_entity_candidate`，只能输出 governance candidate，不得输出 case-to-rule candidate。
- 有 candidate alias / candidate role 时，必须放入 `exception_risk_context`，不得把它们当作 approved rule precondition。
- `promotion_boundary` 必须说明 accountant / governance approval 是 rule promotion 的唯一 authority path。

#### `rule_health_restriction_candidate`

Allowed `rule_health_issue_type` values：
- `frequent_accountant_correction`
- `overbroad_rule_condition`
- `overnarrow_rule_condition`
- `entity_or_alias_authority_changed`
- `role_context_conflict`
- `automation_policy_conflict`
- `case_conflict_with_rule`
- `stale_rule_context`
- `insufficient_evidence_for_action`
Allowed `required_follow_up` values：
- `review_visibility_only`
- `governance_review_required`
- `monitor_next_batch`
- `blocked_insufficient_evidence`
Optional fields：
- `related_case_refs`
- `related_intervention_refs`
- `related_entity_refs`
- `related_governance_refs`
- `suggested_rule_review_question`
Validation rules：
- `rule_ref` 必须引用 durable Rule Log 中的 rule；candidate rule 不适用本对象。
- `no_mutation_boundary` 必须存在。
- `supporting_source_refs` 不能只包含 Knowledge Summary 或 unbound natural language note。
- 输出该 candidate 不得改变 rule lifecycle、rule condition 或 active status。

#### `auto_applied_automation_policy_downgrade`

Allowed downgrade direction：
- `eligible` → `case_allowed_but_no_promotion`
- `eligible` → `rule_required`
- `eligible` → `review_required`
- `eligible` → `disabled`
- `case_allowed_but_no_promotion` → `rule_required`
- `case_allowed_but_no_promotion` → `review_required`
- `case_allowed_but_no_promotion` → `disabled`
- `rule_required` → `review_required`
- `rule_required` → `disabled`
- `review_required` → `disabled`
Optional fields：
- `related_review_finding_id`
- `related_rule_health_candidate_id`
- `prior_policy_history_refs`
- `contest_or_reversal_guidance`
Validation rules：
- `entity_id` 必须引用 stable entity；candidate、ambiguous、merged-away 或 unresolved entity 不得静默降级。
- `new_automation_policy` 必须比 `old_automation_policy` 更保守；任何 upgrade / relaxation invalid。
- `supporting_source_refs` 不能只包含 Transaction Log refs、Knowledge Summary refs、draft、queue item 或 unbound note。
- 降级不能改变 active rule、alias approval、role confirmation、entity merge/split、current transaction accounting outcome 或 historical Transaction Log。
- 必须有 Review / Governance visibility；不能静默 durable mutation。

#### `knowledge_compilation_follow_up`

Allowed `follow_up_type` values：
- `stale_summary_refresh`
- `summary_source_conflict_review`
- `candidate_authority_wording_cleanup`
- `unresolved_risk_monitoring_note`
- `governance_result_summary_update`
Optional fields：
- `related_review_finding_id`
- `related_governance_candidate_id`
- `suggested_compilation_scope`
Validation rules：
- `source_memory_refs` 不能为空。
- `summary_boundary_note` 不能把 candidate 或 lint finding 写成 approved authority。
- 该 follow-up 不得改变 entity、case、rule、governance 或 Transaction Log authority。

#### `no_material_finding_result`

Optional fields：
- `monitor_next_batch_subject_refs`
- `insufficient_data_notes`
Validation rules：
- `no_finding_statement` 不能写成未来自动化永久安全保证。
- 如果存在 unresolved authority conflict、invalid input 或 blocked mutation claim，不能输出 no-material result。

### Field Authority and Memory Boundary

#### Source of truth for important fields

| Field / concept | Source of truth |
| --- | --- |
| `client_id`、`batch_id`、`request_id`、`result_id` | workflow runtime / orchestration context |
| `transaction_id` | Transaction Identity Node / durable transaction identity layer |
| completed transaction history | finalization context and audit-facing `Transaction Log` query; audit-only |
| `entity_id`、entity lifecycle、approved alias、confirmed role、automation_policy | `Entity Log` plus governance-event history |
| completed case precedent | `Case Log` |
| active deterministic rule | `Rule Log` only |
| accountant correction / approval / objection / pending answer | `Intervention Log` / Review Node durable interaction record |
| governance approval / rejection / deferral / restriction / auto-applied downgrade | `Governance Log` / governance-event semantics |
| knowledge summary wording | `Knowledge Log / Summary Log`; never deterministic rule source |
| profile structural facts | `Profile` |

#### Fields that can never become durable memory by this node

本节点不能把以下字段或对象直接写成 durable authority：
- `review_facing_lint_finding`
- `governance_candidate_handoff`
- `case_to_rule_candidate`
- `rule_health_restriction_candidate`
- `knowledge_compilation_follow_up`
- `llm_semantic_summary`
- `finding_summary`
- `recommended_review_action`
- `candidate_role_context`
- `new_entity_candidate_refs`
- `candidate_alias_refs`
- `request_trace` / `result_trace`
- draft、queue、report draft、unbound natural language note

#### Fields that can become durable only after accountant / governance approval

以下事项必须经 accountant / governance approval 才能成为 durable authority：
- entity merge / split outcome
- stable entity creation from candidate
- approved alias / rejected alias
- confirmed role
- rule creation / promotion / modification / deletion / downgrade
- automation-policy upgrade or relaxation
- governance approval / rejection / deferral decision
- source-memory cleanup that changes entity、case、rule 或 governance authority

#### Limited durable action: automation-policy downgrade

本节点唯一可直接触发的 durable action 是更保守方向的 entity `automation_policy` downgrade。

该动作必须同时满足：
- stable entity 可识别；
- `new_automation_policy` 比 `old_automation_policy` 更保守；
- 支撑依据来自 durable source refs，而不是 draft / summary-only / unbound note；
- 写入 `entity_governance_event` 语义记录，`approval_status = auto_applied_downgrade`；
- 对 Review / Governance 可见；
- 不改变 active rule、alias、role、merge/split、current accounting outcome 或 historical `Transaction Log`。

#### Audit vs learning / logging boundary

`Transaction Log` 是 audit-facing history。它可以被查询来理解完成交易和 review/correction trace，但：
- 不参与 runtime decision；
- 不是 learning layer；
- 不是 rule source；
- 不是 governance approval source；
- 不被本节点重写。

### Validation Rules

#### Contract-level validation rules

- 所有 output 必须能追溯到 `post_batch_lint_request.request_id`、`client_id` 和 `batch_id`。
- 所有 material finding / candidate / downgrade 必须有 `subject_refs` 或等价 durable refs。
- Candidate、finding、summary 和 LLM wording 不能替代 source memory refs。
- `Transaction Log` refs 必须标记为 audit-facing / completed history，不得作为 runtime decision、rule source、learning authority 或 governance approval。
- Knowledge Summary 与 source memory 冲突时，source memory / governance history 优先。
- 本节点不能输出任何会导致 active rule 直接变化的 payload。
- 本节点不能输出任何 entity merge / split、alias approval、role confirmation 或 stable entity creation 的 executed payload。
- automation-policy downgrade 之外的 durable mutation claim invalid。

#### Conditions that make the input invalid

Input invalid when：
- `post_batch_lint_request` required fields 缺失。
- `trigger_reasons` 不在 allowed values。
- `completed_batch_basis` 不能证明 batch-level lint boundary 已达到。
- 输入只有单笔未完成 runtime rationale、pending、draft、queue item、candidate note 或 unbound natural language warning。
- source authority 把 candidate-only、rejected、deferred、unresolved 或 summary-only source 标成 approved authority。
- Transaction Log 被声明为 learning source、rule source、runtime decision source 或 governance approval source。
- active source memory 与 summary 冲突但未标记 conflict / source priority。

#### Conditions that make the output invalid

Output invalid when：
- `post_batch_lint_result` required fields 缺失。
- `no_material_finding` 与 material finding / candidate / downgrade 同时出现。
- finding / candidate 没有 durable subject refs 或 source refs。
- candidate wording 把 proposed / repeated / likely 写成 approved / active / confirmed。
- case-to-rule candidate 被写成 active rule 或 Rule Match source。
- rule-health candidate 直接修改 active rule lifecycle 或 condition。
- merge / split、alias、role、new entity candidate 被写成 executed governance outcome。
- automation-policy output 放宽权限、升级权限，或对 non-stable entity 降级。
- auto-downgrade 没有 governance-event payload、supporting refs 或 review visibility。
- output 声称重写或修正 historical `Transaction Log`。

#### Stop / ask conditions for unresolved contract authority

后续 Stage 或实现前必须 stop / ask 的情况：
- 需要把 lint findings 持久保存为新的 durable monitoring artifact / `Log`。
- 需要冻结 auto-downgrade exact eligibility threshold、contest / reversal contract 或 persistence mechanics。
- 需要让 Post-Batch Lint 直接改变 active rule、alias、role、merge/split 或 stable entity status。
- 需要把 Knowledge Summary 或 Transaction Log 作为 future runtime classification / rule source。
- 需要决定 Post-Batch Lint 与 Knowledge Compilation 的 exact ordering。

## 4. Open Boundaries

### Stage 1: 留到 Stage 2 的问题

以下问题有意不在 Stage 1 解决：
- 哪些条件触发 post-batch lint。
- 它消费哪些 input categories。
- 它输出哪些 conceptual categories。
- 哪些检查属于 deterministic code。
- 哪些比较或解释可以使用 LLM semantic judgment。
- 自动降级 automation policy 的 authority 边界。
- governance candidate 与 review-facing finding 的边界。
- memory/log read、candidate-only、limited mutation 和 no-mutation 边界。
- evidence insufficient、ambiguous 或 conflicting 时如何处理。

### Stage 2: Stage 2 Open Boundaries

以下内容仍留到后续 stage 或用户决策：
- Post-Batch Lint 与 Knowledge Compilation 的 exact ordering。
- lint finding、review package、governance candidate queue 的 exact handoff contract。
- automation-policy auto-downgrade 的 exact eligibility、persistence、visibility 和 contest / reversal contract。
- case-to-rule candidate 的 exact eligibility contract。
- rule instability、frequent correction、entity split / merge risk 的 exact thresholds。
- rejected / deferred governance decision 对未来 lint monitoring 和 Knowledge Summary 的 exact expression。
- Post-Batch Lint 是否生成可持久保存的 monitoring artifact；若需要，必须先决定它是否是 durable `Log`，不能默认把 handoff / queue 称为 `Log`。

### Stage 3: Open Contract Boundaries

- Post-Batch Lint 是否需要生成可持久保存的 monitoring artifact 仍未冻结；本 Stage 3 不把 finding / candidate / handoff 默认命名为新的 `Log`。
- Post-Batch Lint 与 Knowledge Compilation 的 exact ordering 未冻结。
- automation-policy auto-downgrade 的 exact eligibility threshold、contest / reversal contract、dedupe / idempotency contract 和 persistence mechanics 未冻结。
- case-to-rule candidate 的 exact eligibility threshold 未冻结。
- rule instability、frequent correction、entity split / merge risk 的 exact thresholds 未冻结。
- rejected / deferred governance decision 对未来 lint monitoring 和 Knowledge Summary 的 exact expression 未冻结。
