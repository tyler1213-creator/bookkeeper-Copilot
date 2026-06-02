# Entity Resolution Node — 设计摘要

> 由原 Stage 1/2/3 合并瘦身而来；保留定位、边界、contract 字段和开放问题，删除阶段说明、examples、self-review、历史读档记录和重复解释。

## 1. 定位与职责

Stage 1 已确定以下内容：
- `Entity Resolution Node` 是 runtime counterparty / vendor / payee identity gate。
- 它位于 `Profile / Structural Match Node` 之后、`Rule Match Node` 之前。
- 它只处理未被结构性路径完成的非结构性交易。
- 它使用当前 evidence、Entity Log、alias / role authority 和必要上下文识别对象。
- 它输出当前交易的实体识别状态，而不是会计分类结论。
- 它必须区分已安全识别、角色未确认、新实体候选、多实体歧义和无法识别等状态。
- `new_entity_candidate` 不天然阻断当前交易后续 case-based judgment，但不能支持 rule match，不能创建 stable entity authority，也不能自动创建或升级 rule。
- 它不执行 rule match、case judgment、COA / HST 判断或 journal entry generation。
- 它不写 `Transaction Log`。
- 它不批准 alias、role、entity merge/split、automation policy 或 rule / governance changes。

## 2. 逻辑与边界

### Trigger Boundary

`Entity Resolution Node` 在主 workflow 中，`Profile / Structural Match Node` 已确认当前交易未被结构性路径完成之后触发。

概念触发条件是：
- 当前交易已有可追溯 evidence foundation、客观交易基础和稳定交易身份；并且
- 当前交易没有被 stable profile structural facts 完成处理；并且
- 后续普通交易 workflow 需要知道当前 evidence 指向哪个 counterparty / vendor / payee，或为什么不能安全确认。
典型触发场景包括：
- 银行描述、小票、支票、invoice、contract 或 accountant context 暴露出 vendor / payee / counterparty 信号。
- 当前 evidence 可能匹配已知实体或 approved alias。
- 当前 evidence 可能指向已知实体，但 role/context 未确认。
- 当前 evidence 看起来是新对象。
- 当前 evidence 可能对应多个实体或角色。
- 当前 evidence 不足以识别对象。

### Input Categories

Stage 2 按判断作用组织 input categories，不列字段清单。

#### Current evidence identity signals

说明当前交易 evidence 中有哪些可用于识别对象的表面信号。

#### Existing entity basis

说明 `Entity Log` 中是否存在可能对应的 stable entity、candidate entity、archived / merged entity、known aliases、known roles、status、authority、risk flags 或 automation-policy context。

#### Alias authority basis

说明当前表面写法是否已被确认、仍是候选、已被拒绝，或与多个对象存在相似关系。

核心边界：
- approved alias 可以支持稳定 entity resolution 和后续 rule eligibility。
- candidate alias 可以作为 Case Judgment 的上下文，但不能支持 rule match。
- rejected alias 是负证据，不能被 LLM 语义相似度绕过。
- 未确认表面写法不能自动变成 approved alias。

#### Role / context basis

说明当前交易是否需要客户关系中的 role/context 才能安全解释对象。

#### Upstream workflow basis

说明上游 evidence、identity 和 profile / structural nodes 已经给出的约束。

#### Governance / intervention basis

说明是否存在已知治理限制、近期人工修正、rejected alias、merge / split history、automation policy 限制或身份风险。

### Output Categories

Stage 2 只定义 conceptual output categories，不冻结 routing enum 或对象形状。

#### Stable entity basis

含义：当前 evidence 可以安全指向一个已知稳定实体，并且当前 identity basis 足以供下游消费。

边界：
- 这是实体识别结论，不是会计分类结论。
- 它不等于 rule match。
- 它不等于 automation permission 已满足。
- 它不创建新的 alias、role 或 governance approval。

#### Stable entity with unconfirmed role/context

含义：对象本身可识别，但当前交易所需 role/context 尚未获得 accountant-confirmed authority。

边界：
- 该状态可以支持 Case Judgment 的上下文理解。
- 未确认 role 通常不能支持 rule match。
- 本节点不能把 candidate role 写成 stable role。
- 是否 pending、review 或继续 case-based path，取决于下游 authority boundary。

#### New entity candidate

含义：当前 evidence 更像一个尚未稳定存在的新对象，而不是既有实体的安全匹配。

边界：
- `new_entity_candidate` 不天然阻断当前交易后续分类。
- 如果 current evidence 足够强且 authority 允许，后续 Case Judgment 可以把它作为当前交易上下文。
- 它不能支持 rule match。
- 它不能自动创建 stable entity authority。
- 它不能自动批准 alias、role、rule 或 automation policy。
- 它不能自动创建或升级 rule。

#### Ambiguous entity candidates

含义：当前 evidence 可能对应多个已知实体、alias 或 role/context，不能安全归属。

边界：
- 歧义不是让 LLM 选择一个看起来最像对象的许可。
- 该状态通常限制 deterministic rule 和 case-based automation。
- 下游应得到清楚的 ambiguity reason，以便 pending、review 或 governance flow 处理。

#### Unresolved identity

含义：当前 evidence 不足以判断 counterparty / vendor / payee 是谁。

边界：
- unresolved 不是会计分类失败的总称，而是身份识别不足。
- 本节点不应为了让 workflow 继续而伪造 entity。
- 下游应得到缺失或不足原因。

#### Candidate and issue signals

含义：本节点可以指出后续可能需要评估的候选或问题信号，例如：
- alias candidate
- role confirmation candidate
- new entity candidate
- merge / split candidate
- rejected-alias conflict issue
- identity ambiguity issue
- automation-policy / governance candidate
边界：
- Candidate signal 只供后续节点评估。
- 它不是 durable approval。
- 它不改变 stable Entity Log authority。
- 它不改变 active rule 或 automation policy。

### Deterministic Code vs LLM Semantic Judgment

Stage 2 的核心边界是：

#### Deterministic code responsibility

Deterministic code 负责：
- 判断本节点是否被触发：上游 evidence intake、transaction identity 和 profile / structural gate 已完成，且当前交易需要普通 entity-first workflow。
- 汇总当前 evidence identity signals、existing entity basis、alias authority、role authority、automation-policy context 和 upstream issue context。
- 执行 authority checks：approved alias、candidate alias、rejected alias、confirmed role、candidate role、entity lifecycle status、automation-policy 限制。
- 防止结构性交易穿透后被当作普通 entity 交易处理。
- 防止 candidate alias 支持 rule match。
- 防止 rejected alias 被语义相似度绕过。
- 防止 unconfirmed role 被当作 stable role。
- 防止 `new_entity_candidate` 获得 stable entity、rule 或 governance authority。
- 防止 ambiguous / unresolved identity 被自动升级为 stable entity basis。
- 标记下游 eligibility 上限：可进入 rule eligibility、只能作为 case context、需要 pending / review、或只能产生 governance candidate。
- 限制 memory / governance actions：哪些只能作为 candidate，哪些绝不能直接修改。

#### LLM semantic judgment responsibility

LLM 可以在 code 允许的边界内辅助：
- 比较 messy evidence 中的 vendor / payee / counterparty 表面写法。
- 判断 current evidence 是否语义上更像已知实体、全新对象，还是多实体歧义。
- 解释小票、支票、invoice、contract 或 accountant note 中的人类可读 identity clues。
- 总结 identity rationale、ambiguity reason、missing-evidence reason 或 candidate explanation。

#### Hard boundary

LLM 不能：
- 扩大 entity / alias / role authority
- 把 candidate alias 当作 approved alias
- 忽略 rejected alias
- 确认 role
- 创建 stable entity
- merge / split entity
- 修改 automation policy
- 执行 rule match
- 判断 case precedent 是否足以分类
- 选择 COA / HST 处理
- 批准 governance event
- 写入 `Transaction Log`

### Accountant Authority Boundary

Accountant 仍然拥有最终 accounting decision 和 durable authority。

`Entity Resolution Node` 不能：
- 替 accountant 确认 role/context
- 替 accountant approve / reject alias
- 替 accountant 判断两个对象是否应长期 merge / split
- 把 runtime identity guess 变成 accountant-confirmed entity memory
- 把 client-provided context 直接升级成 accountant-approved authority
- 替 accountant 批准当前交易最终分类或 review outcome

### Governance Authority Boundary

Governance-level changes 不属于 Entity Resolution authority。

本节点不能：
- approve / reject alias
- confirm role
- create stable entity authority
- merge / split entity
- archive / reactivate entity
- upgrade or relax automation policy
- promote / modify / delete / downgrade active rule
- approve governance event
- invalidate durable memory

### Memory / Log Boundary

Stage 2 采用三层边界：read / consume、candidate-only、no direct mutation。

#### Read / consume boundary

`Entity Resolution Node` 可以读取或消费以下 conceptual context：
- runtime evidence foundation
- objective transaction basis
- evidence references
- stable transaction identity
- non-structural handoff context from `Profile / Structural Match Node`
- `Entity Log` 中的 known entities、aliases、roles、status、authority、risk flags 和 automation-policy context
- `Governance Log` 或 governance context 中已生效的 alias、role、merge/split、policy 限制
- `Intervention Log` 中与 identity correction 有关的近期人工介入语境
- `Knowledge Log / Summary Log` 中可读的客户知识摘要，但只能作为辅助上下文，不能替代 Entity Log authority

#### Candidate-only boundary

本节点只能作为候选或 issue signal 表达：
- new entity candidate
- alias candidate
- role confirmation candidate
- entity ambiguity issue
- unresolved identity issue
- rejected-alias conflict issue
- merge / split candidate
- automation-policy / governance candidate

#### No direct mutation boundary

本节点绝不能：
- 写入 `Transaction Log`
- 修改 stable `Entity Log` authority
- 批准 alias
- 拒绝 alias
- 确认 role
- merge / split entity
- 修改 entity lifecycle status 为 stable authority
- 修改 automation policy
- 写入或修改 `Case Log`
- 写入或修改 `Rule Log`
- 写入或批准 `Governance Log`
- 创建、升级、修改、删除或降级 active rule
- 把 runtime identity result 直接变成 accountant decision

### Insufficient / Ambiguous / Conflicting Evidence Behavior

Stage 2 采用优先级边界：authority first、identity safety second、candidate clarity third。

#### Authority first

如果 alias、role、entity lifecycle、automation policy 或 governance context 已经限制当前 identity basis，本节点不能用语义相似度绕过。

典型情况：
- 表面写法对应 rejected alias。
- 当前 alias 只是 candidate alias。
- 当前 role/context 需要确认但尚未确认。
- known entity 存在治理限制。
- automation policy 限制下游自动化。
- merge / split history 显示旧实体引用需要治理解释。

#### Identity safety second

如果 current evidence 可以安全指向一个已知实体，本节点应输出 stable identity basis。
如果证据指向新对象，应保持 `new_entity_candidate` 语义。
如果证据不足或多个对象都合理，应保持 ambiguous 或 unresolved。

#### Candidate clarity third

当 evidence 缺失、模糊或冲突时，本节点应清楚说明卡点属于哪类问题：
- 缺少可识别对象信号
- 表面写法相似但 authority 不足
- 可能是新对象
- 多个实体或 role/context 竞争
- rejected alias 或治理历史冲突
- evidence source 之间互相矛盾

#### Conflict behavior

如果 current evidence 与 Entity Log、alias authority、role authority、governance history 或 accountant context 冲突，本节点应保守处理。

典型边界：
- 冲突只是缺一个具体事实确认：输出 pending context。
- 冲突涉及长期 entity memory 是否错误：输出 review / governance candidate。
- 冲突影响当前是否能安全识别对象：不能输出 stable identity basis。

#### Hard boundary

- 相似名称不等于同一实体。
- Candidate alias 不等于 approved alias。
- Candidate role 不等于 confirmed role。
- `new_entity_candidate` 不天然 hard block，但也不提供 durable authority。
- Ambiguity 不等于可以猜。
- Entity resolution confidence 只表示身份识别信心，不表示会计分类信心。
- `Transaction Log` 不参与 runtime identity decision。

## 3. Contract 字段摘要

### Contract Position in Workflow

#### Upstream Handoff Consumed

本节点消费 `Profile / Structural Match Node` 之后的 `non_structural_entity_resolution_handoff`。

该 handoff 表示：
- evidence intake 已形成可追溯 evidence foundation；
- transaction identity 已稳定；
- profile / structural path 未完成当前交易处理；
- 当前交易需要进入普通 `entity -> rule -> case` workflow。

#### Downstream Handoff Produced

本节点输出 `entity_resolution_output`，供以下 downstream nodes 消费：
- `Rule Match Node`：只在 stable entity、approved alias、confirmed role/context 和 automation policy 都允许时使用。
- `Case Judgment Node`：使用 entity basis、new entity candidate、role uncertainty、ambiguity reason 或 unresolved reason 作为当前交易上下文。
- `Coordinator / Pending Node`：使用 blocking / issue reason 生成 accountant question。
- `Review Node` / `Governance Review Node`：使用 candidate signals，但这些 signal 不是 approval。
- `Case Memory Update Node`：在交易完成后，可结合最终结果消费 candidate signals；本节点输出本身不等于 long-term memory mutation。

#### Logs / Memory Stores Read

本节点可以读取或消费：
- `Evidence Log` references：只作为 evidence source，不作为业务结论。
- `Entity Log`：known entities、aliases、roles、status、authority、risk_flags、automation_policy、evidence_links。
- `Governance Log` / governance context：已生效的 alias、role、merge/split、policy 限制。
- `Intervention Log`：与 identity correction、accountant confirmation、recent conflict 有关的语境。
- `Knowledge Log / Summary Log`：只作为辅助上下文，不能替代 `Entity Log` authority。
- `Profile / Structural Match Node` 的 runtime handoff：用于确认当前交易不是已完成结构性交易。

#### Logs / Memory Stores Written or Candidate-Only

本节点不直接写入：
- `Transaction Log`
- stable `Entity Log` authority
- `Case Log`
- `Rule Log`
- approved `Governance Log` event
- `Profile`
本节点可以输出 candidate-only signals：
- `new_entity_candidate`
- `alias_candidate`
- `role_confirmation_candidate`
- `merge_split_candidate`
- `identity_ambiguity_issue`
- `unresolved_identity_issue`
- `rejected_alias_conflict_issue`
- `automation_policy_governance_candidate`

### Input Contracts

#### `non_structural_entity_resolution_handoff`

Optional fields：
- `upstream_issue_flags`：evidence quality、identity issue、profile issue、duplicate warning 等上游问题信号。
- `accountant_context_refs`：当前批次或当前交易可引用的 accountant-provided context。
- `profile_context_refs`：与本交易有关但未完成结构性处理的 profile context 引用。
Validation / rejection rules：
- `transaction_id`、`client_id`、`objective_transaction_basis`、`evidence_refs` 缺失时输入无效。
- `structural_match_status` 必须明确表示当前交易未被结构性路径完成；否则本节点必须拒绝该 handoff。
- `objective_transaction_basis.amount` 必须采用 absolute amount + explicit direction 的项目级语义；不得用正负号替代 `direction`。
- `evidence_refs` 必须能追溯到 evidence source；不能只传不可追溯摘要。
Runtime-only vs durable references：
- handoff envelope 是 runtime-only。
- `transaction_id`、`client_id`、`evidence_refs` 是 durable references。
- 本节点不能因为收到该 handoff 而创建 durable memory。

#### `current_evidence_identity_signals`

Allowed `signal_type` values：
- `raw_bank_text`
- `receipt_vendor`
- `cheque_payee`
- `invoice_party`
- `contract_party`
- `historical_ledger_name`
- `accountant_context`
- `other_current_evidence`
Allowed `source_priority` values：
- `primary`
- `supporting`
- `weak`
- `conflicting`
Optional fields per signal：
- `normalized_display_text`：仅用于 runtime comparison / display，不等于 approved alias。
- `language_or_script_note`：文本语言、缩写、拼写异常等说明。
- `extracted_role_hint`：当前 evidence 暗示的 role/context，只能是 candidate role hint。
Validation / rejection rules：
- `signals` 可以为空，但若为空，本节点只能输出 `unresolved` 或明确说明缺少 identity signal。
- `surface_text` 不能被替换成没有 evidence trace 的模型总结。
- `normalized_display_text` 不能作为 durable alias approval。
- `extracted_role_hint` 不能作为 confirmed role。
Runtime-only vs durable references：
- `signals` 是 runtime-only object。
- `evidence_ref` 是 durable reference。
- `surface_text` 可被后续 candidate signal 引用，但不能由本节点写入 approved alias。

#### `entity_authority_context`

Allowed `entity_type` values：
- `vendor`
- `payee`
- `counterparty`
- `person`
- `organization`
- `unknown`
Allowed `status` values：
- `candidate`
- `active`
- `merged`
- `archived`
Allowed alias statuses：
- `candidate_alias`
- `approved_alias`
- `rejected_alias`
Allowed automation policy values：
- `eligible`
- `case_allowed_but_no_promotion`
- `rule_required`
- `review_required`
- `disabled`
Optional fields：
- `risk_flags`
- `evidence_links`
- `governance_notes`
- `merge_split_history_refs`
- `recent_identity_intervention_refs`
- `source_retrieval_reason`
Validation / rejection rules：
- `candidate_entities` may be empty; empty candidate set does not invalidate input.
- `active` entity status alone cannot be treated as rule eligibility.
- `candidate` entity status cannot support stable entity authority.
- `merged` or `archived` entity references must be treated as constrained context, not active identity target unless governance context explicitly provides the active target.
- `candidate_alias` cannot support rule match.
- `rejected_alias` is negative evidence and cannot be bypassed by semantic similarity.
- `roles` must contain only accountant-confirmed or allowed onboarding accountant-derived roles with authority metadata.
Runtime-only vs durable references：
- `entity_authority_context` is a runtime view.
- Entity, alias, role, status, policy, governance refs are durable authority records.
- 本节点不能 mutate these records.

#### `identity_governance_and_intervention_context`

Optional fields：
- `recent_corrections`
- `pending_governance_events`
- `rejected_governance_events`
- `auto_applied_downgrades`
- `accountant_confirmation_refs`
Validation / rejection rules：
- pending governance event cannot be treated as approved authority.
- rejected governance event cannot be reused as positive evidence.
- auto-applied downgrade can restrict automation, but cannot upgrade or relax automation.
- accountant confirmation references must identify what was confirmed: alias、role、merge/split、policy 或 current transaction classification；不能泛化成所有 authority。
Runtime-only vs durable references：
- context envelope 是 runtime-only。
- intervention / governance references are durable records.
- 本节点只能 consume constraints or emit candidate signals.

#### `knowledge_summary_context`

Required fields：
- None；该输入整体 optional。
Optional fields：
- `summary_refs`
- `entity_summary_text`
- `identity_risk_notes`
- `known_ambiguity_notes`
Validation / rejection rules：
- Knowledge Summary 与 `Entity Log` authority 冲突时，`Entity Log` / `Governance Log` authority wins。
- Summary text cannot create approved alias, confirmed role, stable entity, rule eligibility, or automation permission。
Runtime-only vs durable references：
- summary context 是 runtime read context。
- summary refs may be durable references。
- 本节点不能把 summary 内容写回 durable entity authority。

### Output Contracts

#### `entity_resolution_output`

Consumer / downstream authority：
- `Rule Match Node` may consume only when output allows rule eligibility.
- `Case Judgment Node` may consume resolved, new candidate, role uncertainty, ambiguous or unresolved context within its own authority boundary.
- `Coordinator / Pending Node` consumes issue reasons.
- `Review Node` / `Governance Review Node` consumes candidate signals as proposals only.
Allowed `status` values：
- `resolved_entity`
- `resolved_entity_with_unconfirmed_role`
- `new_entity_candidate`
- `ambiguous_entity_candidates`
- `unresolved`
Optional fields：
- `entity_id`
- `matched_alias`
- `alias_status`
- `candidate_role`
- `blocking_reason`
- `candidate_entities`
- `governance_issue_refs`
- `intervention_context_refs`
- `identity_risk_flags`
Allowed `confidence` values：
- `high`
- `medium`
- `low`
- `unknown`
Allowed `rule_eligibility` values：
- `eligible_for_rule_match`
- `blocked_candidate_alias`
- `blocked_rejected_alias`
- `blocked_unconfirmed_role`
- `blocked_new_entity_candidate`
- `blocked_ambiguous_identity`
- `blocked_unresolved_identity`
- `blocked_entity_status`
- `blocked_automation_policy`
- `blocked_governance_constraint`
Allowed `case_context_eligibility` values：
- `eligible_for_case_context`
- `case_context_allowed_with_identity_caution`
- `case_context_allowed_but_no_rule_promotion`
- `case_context_requires_pending`
- `case_context_blocked_by_governance`
Allowed `memory_write_boundary` values：
- `runtime_only_no_memory_write`
- `candidate_signal_only`
Validation rules：
- If `status = resolved_entity`, `entity_id` is required and must refer to an active stable entity authority.
- If `status = resolved_entity`, `alias_status` must be `approved_alias` for `rule_eligibility = eligible_for_rule_match`.
- If `status = resolved_entity_with_unconfirmed_role`, `entity_id` is required and `candidate_role` or role issue context must be present.
- If `status = new_entity_candidate`, `entity_id` must be absent unless referring only to a candidate entity record whose authority is explicitly non-stable; `rule_eligibility` must be `blocked_new_entity_candidate`.
- If `status = ambiguous_entity_candidates`, `candidate_entities` must include at least two candidates or an explicit ambiguity reason.
- If `status = unresolved`, `blocking_reason` must explain missing or insufficient identity evidence.
- `confidence` cannot be interpreted as accounting classification confidence.
- `memory_write_boundary` cannot imply stable durable write.
Durable memory effect：
- This output does not write durable memory by itself.
- It can propose candidate signals.
- Any stable memory change requires accountant / governance authority through downstream workflow.

#### `candidate_signal`

Consumer / downstream authority：
- `Coordinator / Pending Node` may turn issue into accountant question.
- `Review Node` may display or collect confirmation.
- `Case Memory Update Node` may consider it after transaction completion.
- `Governance Review Node` may create or process governance event.
Allowed `signal_type` values：
- `new_entity_candidate`
- `alias_candidate`
- `role_confirmation_candidate`
- `merge_split_candidate`
- `identity_ambiguity_issue`
- `unresolved_identity_issue`
- `rejected_alias_conflict_issue`
- `automation_policy_governance_candidate`
Allowed `signal_status` values：
- `candidate_only`
- `requires_accountant_confirmation`
- `requires_governance_review`
Optional fields：
- `proposed_display_name`
- `proposed_alias_surface_text`
- `related_entity_ids`
- `proposed_candidate_role`
- `conflicting_evidence_refs`
- `governance_constraint_refs`
- `suggested_accountant_question`
Validation rules：
- `candidate_signal` cannot have `approved`, `confirmed`, `active`, or `auto_applied` as status.
- `supporting_evidence_refs` required unless `signal_type = unresolved_identity_issue` and the reason is explicitly no identity signal exists.
- `proposed_candidate_role` cannot be written as stable role by this node.
- `related_entity_ids` for merge/split candidate must not imply merge/split approval.
Durable memory effect：
- Candidate signal is not durable authority.
- Whether candidate signals are persisted immediately or only carried in runtime handoff is not frozen in this Stage 3; see Open Contract Boundaries.

#### `downstream_authority_summary`

Validation rules：
- `can_enter_rule_match = true` requires `status = resolved_entity`, `entity_id` present, `alias_status = approved_alias`, confirmed role/context where required, and automation policy allowing rule-based automation.
- `new_entity_candidate` may have `can_enter_case_judgment = true` when current evidence is strong and authority permits, but must have `can_enter_rule_match = false` and `can_support_rule_promotion = false`.
- `resolved_entity_with_unconfirmed_role` must have `can_enter_rule_match = false` unless a future approved contract defines a narrow exception.
- `ambiguous_entity_candidates` and `unresolved` must not support rule match or rule promotion.
Durable memory effect：
- Runtime-only summary.
- Cannot become durable memory by itself.

### Field Authority and Memory Boundary

#### Source of Truth for Important Fields

| Field / Concept | Source of Truth |
| --- | --- |
| `transaction_id` | `Transaction Identity Node` / transaction identity layer |
| objective transaction basis | Evidence Intake / Preprocessing output with Evidence Log trace |
| raw evidence / receipt / cheque / invoice references | `Evidence Log` |
| stable entity fields | `Entity Log` |
| approved / rejected alias | `Entity Log` as governed by accountant / governance approval |
| confirmed role/context | `Entity Log` with accountant-confirmed or allowed onboarding authority metadata |
| entity lifecycle status | `Entity Log` / approved governance projection |
| automation policy | `Entity Log` / `Governance Log`; upgrades or relaxations require accountant approval |
| governance event state | `Governance Log` |
| accountant intervention / confirmation process | `Intervention Log` |
| final transaction processing audit | `Transaction Log`, written downstream only |

#### Fields That Can Never Become Durable Memory by This Node

本节点不能把以下字段或判断直接写成 durable memory：
- `confidence`
- `reason`
- `blocking_reason`
- `candidate_role`
- `normalized_display_text`
- `suggested_accountant_question`
- `rule_eligibility`
- `case_context_eligibility`
- LLM semantic comparison rationale
- raw model prompt trace
- rejected candidate list not exposed in contract
- accounting classification implication

#### Fields That Can Become Durable Only After Accountant / Governance Approval

以下内容可以成为 long-term memory，但只能通过 accountant / governance-approved authority path：
- new stable entity
- approved alias
- rejected alias
- confirmed role/context
- entity merge / split result
- entity archive / reactivate result
- automation policy upgrade or relaxation
- active rule creation / modification / deletion / downgrade / promotion

#### Audit vs Learning / Logging Boundary

`Transaction Log` 是 audit-facing final record：
- 记录最终处理结果和审计轨迹；
- 不参与本节点 runtime decision；
- 不作为 Entity Resolution 的学习来源；
- 不由本节点写入。

### Validation Rules

#### Contract-Level Validation Rules

- Every output must preserve `transaction_id`.
- Every output must identify one and only one `status`.
- `confidence` must be scoped to identity confidence only.
- Every positive identity result must cite `evidence_used`.
- Every blocked or limited result must include `blocking_reason` or `authority_limit_reason`.
- Candidate-only fields must stay candidate-only unless downstream accountant / governance approval changes authority.
- `Transaction Log` must not appear as runtime input.
- Knowledge Summary cannot override Entity Log / Governance Log authority.

#### Conditions That Make Input Invalid

Input is invalid when:

- `transaction_id` is missing or unstable.
- `evidence_refs` are missing or non-traceable.
- objective transaction basis is missing.
- `direction` is absent or amount sign is used as direction authority.
- profile / structural status says current transaction was already completed structurally.
- candidate alias, pending governance event, or knowledge summary is labeled as approved authority.
- rejected alias conflict is omitted from authority context when known.

#### Conditions That Make Output Invalid

Output is invalid when:

- `resolved_entity` has no `entity_id`.
- `resolved_entity` uses a `candidate_alias` or `rejected_alias` while claiming `eligible_for_rule_match`.
- `new_entity_candidate` claims rule eligibility or rule promotion support.
- `resolved_entity_with_unconfirmed_role` writes or implies confirmed role.
- `ambiguous_entity_candidates` selects a single winner without resolving authority.
- `unresolved` lacks reason.
- output claims to write stable Entity Log, Rule Log, Case Log, Governance Log, Transaction Log, or Profile.
- output treats `confidence` as accounting classification confidence.

#### Stop / Ask Conditions for Unresolved Contract Authority

Future Stage 3 revisions or later stages must stop and ask if they need to decide:

- whether candidate entity / alias / role signals are persisted by this node or by downstream memory-update / governance workflow;
- exact threshold for sufficient evidence categories to support stable resolution;
- whether accountant confirmation should retrigger Entity Resolution in the same batch;
- exact rule/case routing for unconfirmed role exceptions;
- how to resolve direct conflict between Knowledge Summary text and current Entity Log authority beyond the rule that Entity Log / Governance Log wins.

## 4. Open Boundaries

### Stage 1: Open Boundaries

以下问题留到后续阶段，不在 Stage 1 冻结：
- 哪些 evidence categories 足以支持 stable entity resolution。
- alias status、role authority 和 automation policy 如何精确限制 downstream eligibility。
- `new_entity_candidate` 在 runtime 中只作为 handoff signal，还是可以创建待治理 candidate memory；exact write locus 尚未冻结。
- 多实体歧义、角色未确认、rejected alias、弱证据和冲突 evidence 的 precise downstream path。
- Entity Resolution 是否以及何时可触发补证据、人工确认或 governance candidate。
- entity resolution confidence 的语义边界和下游消费方式。
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
- stable entity resolution 所需 evidence categories 的精确门槛
- candidate entity 是否由 Entity Resolution 直接落盘，还是由 Case Memory Update / Governance workflow 落盘
- candidate alias / role / merge-split signal 的 exact memory operation contract
- accountant 补充确认后是否重新触发本节点
- role/context 未确认时下游 rule、case、pending、review 的 exact routing contract
- Knowledge Summary 与 Entity Log authority 冲突时的精确处理 contract

### Stage 3: Open Contract Boundaries

- Candidate entity / alias / role / merge-split signals 的 exact persistence locus 尚未冻结：本节点直接写 candidate queue，还是只在 runtime handoff 中返回并由 `Case Memory Update Node` / `Governance Review Node` 落盘。
- stable entity resolution 所需 evidence category threshold 尚未冻结；本 Stage 3 只要求 evidence traceability 和 authority consistency。
- accountant 回答 identity question 后是否 same-batch retrigger Entity Resolution 尚未冻结。
- `resolved_entity_with_unconfirmed_role` 在哪些窄场景可以进入 Case Judgment、Pending 或 Review 的 precise routing 仍属 downstream contract。
- Knowledge Summary 与 Entity Log / Governance Log 冲突时，当前 contract 只冻结 authority order，不冻结修复流程。
- `confidence` 的 exact threshold / calibration 尚未冻结；本 contract 只冻结 compact labels 和语义边界：identity confidence only, not accounting confidence。
