# Rule Match Node — 设计摘要

> 由原 Stage 1/2/3 合并瘦身而来；保留定位、边界、contract 字段和开放问题，删除阶段说明、examples、self-review、历史读档记录和重复解释。

## 1. 定位与职责

Stage 1 已确定以下内容：
- `Rule Match Node` 是 approved-rule deterministic execution gate。
- 它位于 `Entity Resolution Node` 之后、`Case Judgment Node` 之前。
- 它只处理未被结构性路径完成、且已有 entity resolution output 的非结构性交易。
- 它只在 entity、alias、role/context、automation policy、active rule 和 rule 条件全部允许时完成 deterministic rule-handled path。
- `new_entity_candidate` 不能支持 rule match。
- candidate alias 不能支持 rule match；approved alias 才能支持 rule match。
- 未确认 role/context 不能支持需要该 role/context 的 rule match。
- Rule match 是确定性路径，不是 LLM 语义判断路径。
- 它不执行 entity resolution、case judgment、COA / HST 语义判断或 journal entry generation。
- 它不写 `Transaction Log`。
- 它不创建、升级、修改、删除、降级或批准 rules。
- Rule promotion 和 active rule changes 必须经过 accountant / governance approval。

## 2. 逻辑与边界

### Trigger Boundary

`Rule Match Node` 在主 workflow 中，`Entity Resolution Node` 已经为当前非结构性交易给出 entity-resolution context 之后触发。

概念触发条件是：
- 当前交易已有可追溯 evidence foundation、客观交易基础和稳定交易身份；并且
- 当前交易没有被 profile-backed structural path 完成；并且
- 当前交易已完成 entity resolution handoff；并且
- 后续 workflow 需要判断是否存在可以直接执行的 approved active rule。
典型触发场景包括：
- 当前交易已安全识别到 stable entity，且 alias / role / automation-policy 可能允许 rule match。
- 当前交易识别到 stable entity，但 role/context、alias 或 policy 可能阻断 rule match。
- 当前交易是 `new_entity_candidate`、ambiguous 或 unresolved，需要明确 deterministic rule path 不可用。
- 当前交易可能存在 active rule，但当前方向、金额、context 或其他已批准条件未满足。
- 当前交易存在 rule governance restriction、非 active rule、需要 review 的 rule context 或 automation policy block。

### Input Categories

Stage 2 按判断作用组织 input categories，不列字段清单。

#### Entity resolution basis

说明当前交易的 entity-resolution 状态及其 authority 语境。

核心边界：
- stable active entity 可以成为 rule eligibility 的必要基础。
- `new_entity_candidate` 不能支持 rule match。
- ambiguous entity candidates 不能支持 rule match。
- unresolved identity 不能支持 rule match。
- candidate alias 不能支持 rule match。
- rejected alias 不能被 rule match 绕过。
- unconfirmed role/context 不能支持需要该 role/context 的 rule。

#### Automation-policy basis

说明当前实体或语境是否允许 rule-based automation。

核心边界：
- 允许 case-based judgment 不等于允许 rule match。
- `rule_required` 表示只有 approved rule 可以自动分类；没有适用 approved rule 时必须离开 deterministic path。
- `review_required` 或 `disabled` 不能被 rule match 降级为自动处理。
- automation policy 的升级或放宽必须经过 accountant approval。

#### Approved rule basis

说明是否存在可用于当前 entity / role / context 的 active rule。

核心边界：
- 只有 active、approved、未被治理限制的 rule 可以支持 deterministic match。
- rule candidate、case memory、Knowledge Summary、repeated outcome 或 lint suggestion 不能作为 active rule source。
- 非 active、未批准、被拒绝、等待审批、处于 review 限制或 governance-blocked 的 rule 不能被当作 active rule 执行。

#### Rule condition basis

说明当前交易是否满足 rule 自身已经批准的适用条件。

#### Upstream workflow basis

说明上游 evidence、identity、profile / structural 和 entity-resolution nodes 已经给出的限制。

#### Governance / intervention basis

说明是否存在 rule、entity、alias、role、automation policy 或近期 accountant intervention 相关的治理限制。

### Output Categories

Stage 2 只定义 conceptual output categories，不冻结 routing enum 或对象形状。

#### Deterministic rule-handled path

含义：当前交易满足 rule match 的全部前置 authority 和 approved rule condition，可以进入 deterministic rule-handled path。

边界：
- 这是当前交易的 runtime deterministic path。
- 它不等于 accountant final review approval。
- 它不创建或修改 rule。
- 它不创建或修改 entity / alias / role / automation policy。
- 它不写 `Transaction Log`。

#### Rule miss path

含义：当前 entity / context 允许尝试 rule matching，但不存在适用 active rule，或当前交易不满足任何 active rule 的已批准适用条件。

边界：
- Rule miss 不是 case judgment。
- Rule miss 不是 permission to guess。
- Rule miss 不自动创建 rule candidate。
- 下游 `Case Judgment Node` 可以读取 miss reason 和当前 context 继续判断。

#### Rule blocked / ineligible path

含义：当前不是单纯没有命中 rule，而是 rule path 缺少必要 authority 或被 policy / governance 限制。

典型原因包括：
- entity 不是 stable active entity
- `new_entity_candidate`
- ambiguous / unresolved identity
- alias 不是 approved alias 或存在 rejected-alias conflict
- role/context 未确认
- automation policy 不允许 rule-based automation
- rule lifecycle 或 governance state 不允许执行
边界：
- Blocked / ineligible reason 是约束，不是 Case Judgment 或 LLM 可以自行解除的软建议。
- 下游应根据原因进入 pending、review-required、governance candidate 或 case-allowed path；exact routing 留到后续阶段。

#### Review-required / governance-needed signal

含义：当前 rule path 不能完成，因为需要 accountant review、role/alias/entity confirmation、rule governance、automation policy decision 或 rule conflict resolution。

边界：
- 本节点只暴露 review / governance need。
- 它不提问 accountant。
- 它不批准治理事件。
- 它不改变 active rule 或 automation policy。

#### Candidate signals

含义：本节点可以指出后续可能需要评估的候选信号，例如：
- rule candidate
- rule conflict review candidate
- rule stale / unstable candidate
- rule condition gap candidate
- entity / alias / role confirmation candidate
- automation-policy / governance candidate
边界：
- Candidate signal 只供后续节点评估。
- 它不是 durable approval。
- 它不进入 active rule set。
- 它不改变 Entity Log、Rule Log 或 Governance Log authority。

### Deterministic Code vs LLM Semantic Judgment

Stage 2 的核心边界是：

#### Deterministic code responsibility

Deterministic code 负责：
- 判断本节点是否被触发：上游 evidence intake、transaction identity、profile / structural gate 和 entity resolution 已完成，且当前交易需要 ordinary rule eligibility check。
- 读取并汇总 entity resolution basis、alias authority、role/context authority、automation policy、approved rule basis、rule lifecycle 和 upstream issue context。
- 判断 entity 是否具备 rule eligibility：stable active entity、approved alias、confirmed required role/context。
- 判断 automation policy 是否允许 rule-based automation。
- 判断是否存在 active approved rule。
- 判断当前 objective transaction basis 是否满足 rule 自身已批准条件。
- 区分 deterministic rule-handled、rule miss、rule blocked / ineligible、review / governance-needed 和 candidate signal。
- 防止 `new_entity_candidate`、candidate alias、unconfirmed role、ambiguous / unresolved identity 支持 rule match。
- 防止 case memory、Knowledge Summary、Transaction Log、lint suggestion 或 repeated outcomes 被当作 active rule。
- 防止 LLM 语义解释扩大、缩小或改写 approved rule 条件。
- 限制 memory / governance actions：哪些只能作为 candidate，哪些绝不能直接修改。

#### LLM semantic judgment responsibility

LLM 通常不应参与 rule match 本身。

如果后续阶段允许 LLM 辅助，它最多可以在受限边界内帮助：
- 把 rule miss、blocked reason 或 governance need 解释成可读说明。
- 总结为什么当前交易未满足某个已批准 rule 条件。
- 帮助生成 pending / review explanation。
- 帮助整理 rule candidate 或 conflict candidate 的自然语言理由。

#### Hard boundary

LLM 不能：
- 决定 rule 是否命中
- 放宽 rule 条件
- 创造临时 rule condition
- 把 candidate rule 当作 active rule
- 把 case memory 当作 rule
- 把 `new_entity_candidate` 当作 rule-eligible entity
- 把 candidate alias 当作 approved alias
- 确认 role/context
- 忽略 automation policy 或 governance block
- promote、modify、delete 或 downgrade rule
- 选择 COA / HST 处理来替代 rule match
- 批准 accountant 或 governance decision
- 写入 `Transaction Log`

### Accountant Authority Boundary

Accountant 仍然拥有最终 accounting decision 和 durable authority。

本节点不能：
- 把 repeated runtime outcomes 自动升级为 rule
- 把 case precedent 自动升级为 rule
- 批准 rule candidate
- 修改 rule condition
- 确认 role/context
- 批准 alias
- 放宽 automation policy
- 替 accountant 解决 rule conflict 的业务含义
- 替 accountant 批准当前交易最终 review outcome

### Governance Authority Boundary

Governance-level changes 不属于 Rule Match authority。

本节点不能：
- create / promote active rule
- modify rule condition
- delete / downgrade active rule
- approve / reject rule candidate
- approve / reject alias
- confirm role
- create stable entity authority
- merge / split entity
- upgrade or relax automation policy
- approve governance event
- invalidate durable memory

### Memory / Log Boundary

Stage 2 采用三层边界：read / consume、candidate-only、no direct mutation。

#### Read / consume boundary

`Rule Match Node` 可以读取或消费以下 conceptual context：
- runtime evidence foundation
- objective transaction basis
- stable transaction identity
- non-structural handoff context from `Profile / Structural Match Node`
- Entity Resolution output
- `Entity Log` 中的 stable entity、approved aliases、confirmed roles、status、authority、risk flags 和 automation-policy context
- `Rule Log` 中的 approved active rules、rule lifecycle 和 governance status
- `Governance Log` 或 governance context 中已生效的 alias、role、rule、policy 限制
- `Intervention Log` 中已经转化为治理状态或 review-required constraint 的相关语境

#### Candidate-only boundary

本节点只能作为候选或 issue signal 表达：
- rule candidate
- rule condition gap candidate
- rule conflict candidate
- rule stale / unstable candidate
- missing role/context confirmation issue
- alias / entity authority issue
- automation-policy / governance candidate

#### No direct mutation boundary

本节点绝不能：
- 写入 `Transaction Log`
- 写入或修改 `Entity Log`
- 写入或修改 `Case Log`
- 写入或修改 `Rule Log`
- 写入或批准 `Governance Log`
- 创建、升级、修改、删除或降级 active rule
- 批准 alias
- 确认 role/context
- 创建 stable entity
- 修改 automation policy
- 把 case memory 或 repeated outcome 直接变成 rule
- 把 runtime rule miss 直接变成 rule promotion

### Insufficient / Ambiguous / Conflicting Evidence Behavior

Stage 2 采用优先级边界：authority first、deterministic fit second、safe handoff third。

#### Authority first

如果 entity、alias、role/context、automation policy、rule lifecycle 或 governance context 缺少必要 authority，本节点不能完成 rule-handled path。

典型情况：
- 当前是 `new_entity_candidate`。
- 当前 entity ambiguous 或 unresolved。
- matched alias 只是 candidate alias。
- matched alias 与 rejected alias 或治理历史冲突。
- required role/context 尚未 accountant-confirmed。
- automation policy 不允许 rule-based automation。
- rule 不是 active approved rule。
- rule 正在等待审批、处于 review 限制、被治理限制或需要人工确认。

#### Deterministic fit second

即使 authority 完整，当前交易也必须满足 active rule 自身已批准条件。

#### Safe handoff third

如果没有适用 active rule，且不存在必须先处理的 rule authority block，应把 deterministic path 未完成原因交给 `Case Judgment Node`。

#### Conflict behavior

如果 current evidence、entity-resolution context、Rule Log、automation policy、governance history 或 accountant context 冲突，本节点应保守处理。

典型边界：
- 冲突只是缺一个具体事实确认：输出 pending / review context。
- 冲突涉及 rule 是否仍应 active：输出 governance candidate。
- 冲突影响当前 rule 是否能确定性适用：不能输出 rule-handled path。

#### Hard boundary

- `new_entity_candidate` 不支持 rule match。
- Candidate alias 不支持 rule match。
- Candidate role 不等于 confirmed role。
- Case memory 不等于 rule。
- Knowledge Summary 不等于 deterministic rule source。
- Transaction Log 不参与 runtime rule decision。
- Lint warning 和 recent intervention 只有通过治理改变 active authority 后，才影响 rule eligibility。
- Rule miss 不等于 rule candidate approval。
- Rule match confidence 不应替代 approved authority。

## 3. Contract 字段摘要

### Contract Position in Workflow

`Rule Match Node` 位于：

#### Upstream handoff consumed

本节点消费一个 runtime `rule_match_request`，由上游 workflow 在以下前提满足后传入：
- evidence intake / preprocessing 已形成客观交易基础；
- transaction identity 已分配稳定 `transaction_id`；
- profile / structural path 已确认没有完成本笔交易；
- Entity Resolution Node 已输出 `entity_resolution_output`；
- workflow 需要判断是否存在可直接执行的 approved active rule。

#### Downstream handoff produced

本节点输出一个 runtime `rule_match_result`。

下游读取该结果来判断：
- 是否已经完成 deterministic `rule_handled` path；
- 如果没有完成，原因是 `rule_miss`、`rule_blocked`、`review_required`、`governance_needed` 还是 `rule_conflict`；
- Case Judgment Node 是否可以继续 case-based judgment；
- Coordinator / Review / Governance Review 需要哪些聚焦 context。

#### Logs / memory stores read

本节点可以读取或消费：
- runtime evidence references；
- objective transaction basis；
- Entity Resolution output；
- `Entity Log` 中的 stable entity、approved aliases、confirmed roles、status、authority、automation_policy 和已生效 governance context；
- `Rule Log` 中的 approved active deterministic rules；
- `Governance Log` 中已经生效的 alias、role、rule、policy 限制；
- `Intervention Log` 中已经转化为治理状态或 review-required constraint 的相关语境。

#### Logs / memory stores written or candidate-only

本节点不直接写入任何 durable log / memory store。

它只可以输出 runtime candidate / issue signals，例如：
- `rule_candidate_signal`
- `rule_condition_gap_signal`
- `rule_conflict_signal`
- `rule_stale_or_unstable_signal`
- `entity_alias_role_authority_signal`
- `automation_policy_governance_signal`

### Input Contracts

#### `rule_match_request`

Required fields：
- `transaction_basis`
- `upstream_workflow_context`
- `entity_resolution_output`
- `entity_authority_snapshot`
- `automation_policy_snapshot`
- `rule_log_snapshot`
Optional fields：
- `effective_governance_constraints`
- `effective_intervention_constraints`
- `request_trace`
Validation / rejection rules：
- 缺少 `transaction_basis.transaction_id` 时，input invalid。
- 缺少 `entity_resolution_output.status` 时，input invalid。
- 如果 `upstream_workflow_context.structural_path_status = handled`，input invalid；本节点不能重跑已由 structural path 完成的交易。
- 如果 `rule_log_snapshot` 无法表明候选 rule 的 approval / active authority，不能输出 `rule_handled`。
- `request_trace` 只能用于 runtime diagnostics，不能作为 rule authority。
Runtime-only vs durable references：
- `rule_match_request` 是 runtime-only。
- 其中的 `transaction_id`、`evidence_refs`、`entity_id`、`rule_id`、governance references 是 durable objects 的引用，不表示本节点拥有写入权。

#### `transaction_basis`

Required fields：
- `transaction_id`：稳定交易 ID。
- `amount_abs`：绝对值金额。
- `direction`：交易方向；allowed values: `inflow`, `outflow`。
- `transaction_date`
- `bank_account`
- `evidence_refs`：至少一个 evidence reference。
Optional fields：
- `raw_description`
- `description`
- `pattern_source`
- `receipt_refs`
- `cheque_info_refs`
- `counterparty_surface_text`
- `currency`
- `objective_tags`
Validation / rejection rules：
- `amount_abs` 必须为正数；金额符号不能与 `direction` 重复表达。
- `direction` 只能使用允许值；不能用 signed amount 推断 direction。
- `evidence_refs` 不能为空。
- `description = null` 是有效状态；本节点不能要求所有交易都有 canonical description。
- `pattern_source = null` 是有效状态；不能把 null 当作 fallback rule source。
Runtime-only vs durable references：
- 客观字段来自 durable evidence / transaction identity。
- `objective_tags` 若存在，只是 runtime helper，不能成为 rule authority，除非其来源是 approved rule condition 或客观 evidence fact。

#### `upstream_workflow_context`

Required fields：
- `evidence_status`；allowed values: `ready`, `blocked`。
- `transaction_identity_status`；allowed values: `assigned`, `duplicate_blocked`, `blocked`。
- `structural_path_status`；allowed values: `not_structural`, `not_handled`, `handled`, `blocked`。
- `entity_resolution_completed`：boolean。
Optional fields：
- `upstream_blocking_reasons`
- `upstream_issue_signals`
- `profile_structural_reference`
Validation / rejection rules：
- `evidence_status != ready` 时，本节点不能输出 `rule_handled`。
- `transaction_identity_status != assigned` 时，本节点不能输出 `rule_handled`。
- `structural_path_status = handled` 时，本节点输入 invalid。
- `entity_resolution_completed != true` 时，本节点输入 invalid。
- `upstream_blocking_reasons` 不能被包装成 ordinary `rule_miss` 后继续自动化。
Runtime-only vs durable references：
- 该对象是 runtime handoff。
- `profile_structural_reference` 如存在，只能说明 structural gate 的结果或限制，不给本节点 profile mutation authority。

#### `entity_resolution_output`

Required fields：
- `status`；allowed values: `resolved_entity`, `resolved_entity_with_unconfirmed_role`, `new_entity_candidate`, `ambiguous_entity_candidates`, `unresolved`。
- `confidence`
- `reason`
- `evidence_used`
Conditionally required fields：
- `entity_id`：当 `status = resolved_entity` 或 `resolved_entity_with_unconfirmed_role` 时 required。
- `matched_alias`：当 rule eligibility 依赖 alias 时 required。
- `alias_status`：当 `matched_alias` 存在时 required；allowed values: `candidate_alias`, `approved_alias`, `rejected_alias`。
- `candidate_entities`：当 `status = ambiguous_entity_candidates` 时 required。
- `blocking_reason`：当 status 不能支持 deterministic automation 时 required。
Optional fields：
- `candidate_role`
- `matched_alias_evidence_ref`
- `identity_risk_flags`
Validation / rejection rules：
- `new_entity_candidate`、`ambiguous_entity_candidates`、`unresolved` 不能支持 `rule_handled`。
- `resolved_entity_with_unconfirmed_role` 不能支持需要 confirmed role/context 的 rule。
- `alias_status = candidate_alias` 不能支持 rule match。
- `alias_status = rejected_alias` 必须输出 blocked / review / governance context，不能输出 ordinary miss。
- `confidence` 只表示 entity resolution confidence，不能替代 alias、role、rule 或 accounting authority。
Runtime-only vs durable references：
- `entity_resolution_output` 是 runtime-only。
- `candidate_role` 是 runtime-only，不能由本节点写入 durable entity roles。
- `entity_id` 和 approved alias references 指向 `Entity Log`，但本节点没有修改权。

#### `entity_authority_snapshot`

Required fields：
- `entity_id`
- `entity_status`；allowed values: `candidate`, `active`, `merged`, `archived`。
- `approved_aliases`
- `confirmed_roles`
- `authority`
- `automation_policy`
Optional fields：
- `display_name`
- `risk_flags`
- `governance_notes`
- `effective_governance_refs`
- `merged_into_entity_id`
Validation / rejection rules：
- `entity_status != active` 不能支持 `rule_handled`。
- `approved_aliases` 不包含当前 `matched_alias` 时，不能支持 `rule_handled`。
- rule 要求的 role/context 不在 `confirmed_roles` 中时，不能支持该 rule。
- `merged` entity 不能直接支持 rule match；必须经治理确认后的 active entity / rule authority。
- `authority` 不能缺失来源；无 authority metadata 的 entity snapshot 不能支持 `rule_handled`。
Runtime-only vs durable references：
- snapshot 是 runtime-only view。
- snapshot 字段来自 durable `Entity Log`，但本节点不能写入或修正它。

#### `automation_policy_snapshot`

Required fields：
- `entity_id`
- `policy`；allowed values: `eligible`, `case_allowed_but_no_promotion`, `rule_required`, `review_required`, `disabled`。
- `policy_source`
Optional fields：
- `effective_from`
- `reason`
- `governance_ref`
Validation / rejection rules：
- `eligible` 允许 rule-based automation。
- `rule_required` 允许 approved active rule 自动处理；没有适用 rule 时必须离开 deterministic path。
- `case_allowed_but_no_promotion` 是否允许 rule-based automation 不是由名称本身完全冻结；本节点必须按 effective policy metadata 判断。如果 metadata 未明确允许 rule-based automation，不能输出 `rule_handled`。
- `review_required` 或 `disabled` 不能输出 `rule_handled`。
- policy upgrade / relaxation 只能来自 accountant-approved governance，不能由本节点推断。
Runtime-only vs durable references：
- snapshot 是 runtime-only。
- `policy_source` 和 `governance_ref` 是 durable authority references。

#### `rule_log_snapshot`

Required fields：
- `rules`：rule-facing records list；可以为空。
- `snapshot_source`
Each `rule_record` required fields：
- `rule_id`
- `entity_id`
- `rule_status`；node-facing allowed values: `active`, `inactive`, `pending_approval`, `rejected`, `governance_blocked`, `review_required`。
- `approval_status`；allowed values: `approved`, `pending`, `rejected`。
- `authority_source`
- `applicability`
- `approved_condition_set`
- `approved_accounting_treatment`
Each `rule_record` optional fields：
- `rule_name`
- `version`
- `effective_from`
- `effective_until`
- `governance_refs`
- `review_notes`
`applicability` required fields：
- `entity_id`
`applicability` optional fields：
- `required_aliases`
- `required_roles`
- `required_context_tags`
`approved_condition_set` optional fields：
- `direction`
- `amount_range`
- `currency`
- `required_evidence_types`
- `excluded_evidence_types`
- `approved_exception_conditions`
`approved_accounting_treatment` required meaning：
- approved rule-owned accounting outcome payload to be forwarded when the rule is matched.
- It must include enough approved accounting authority for downstream classification / JE workflow, but the exact JE line schema belongs to JE Generation Stage 3.
Validation / rejection rules：
- 只有 `rule_status = active` 且 `approval_status = approved` 的 rule 可以支持 `rule_handled`。
- `authority_source` 缺失时，不能执行 rule。
- `applicability.entity_id` 必须与当前 active entity 一致。
- `required_aliases` 若存在，当前 `matched_alias` 必须是 approved alias 且包含在 required aliases 内。
- `required_roles` 若存在，当前 context 必须由 confirmed role 满足。
- `approved_condition_set` 只能使用 rule 已批准条件和 objective transaction facts；不能由本节点临时扩展。
- `approved_accounting_treatment` 不能由本节点重写。
Runtime-only vs durable references：
- `rule_log_snapshot` 是 runtime read view。
- `rule_record` 来自 durable `Rule Log`。
- 本节点不能新增、修改、降级、删除或 approve rule。

#### `effective_governance_constraints`

Required fields when present：
- `constraint_id`
- `constraint_type`；allowed values: `alias_restriction`, `role_restriction`, `rule_restriction`, `automation_policy_restriction`, `entity_lifecycle_restriction`。
- `applies_to`
- `effect`；allowed values: `block_rule_match`, `require_review`, `limit_rule_scope`。
- `authority_ref`
Optional fields：
- `reason`
- `effective_from`
- `expires_at`
Validation / rejection rules：
- `effect = block_rule_match` 时不能输出 `rule_handled`。
- `effect = require_review` 时不能输出 automatic final handling；可以输出 review-required context。
- pending / rejected governance proposal 不能作为 effective constraint，除非它已经通过其他 authority 改变了 rule、entity 或 automation-policy status。
Runtime-only vs durable references：
- constraints 是 runtime view of durable governance authority。
- 本节点不能写入 `Governance Log`。

#### `effective_intervention_constraints`

Required fields when present：
- `intervention_ref`
- `constraint_type`；allowed values: `review_required`, `rule_suspended`, `policy_changed`, `role_or_alias_disputed`。
- `authority_ref`
Optional fields：
- `reason`
- `related_rule_ids`
- `related_entity_ids`
Validation / rejection rules：
- raw recent intervention 或 lint warning 不能直接改变 rule eligibility。
- 只有已经转化为 `rule_status`、`automation_policy`、effective governance constraint 或 review-required constraint 的 intervention 才能影响本节点输出。
Runtime-only vs durable references：
- 该对象是 runtime view。
- intervention 原文不成为 rule source。

### Output Contracts

#### `rule_match_result`

Required fields：
- `transaction_id`
- `status`
- `authority_summary`
- `result_reason`
Allowed `status` values：
- `rule_handled`
- `rule_miss`
- `rule_blocked`
- `review_required`
- `governance_needed`
- `rule_conflict`
- `invalid_input`
Conditionally required fields：
- `matched_rule`：当 `status = rule_handled` 时 required。
- `rule_output_payload`：当 `status = rule_handled` 时 required。
- `miss_context`：当 `status = rule_miss` 时 required。
- `blocked_context`：当 `status = rule_blocked` 时 required。
- `review_context`：当 `status = review_required` 时 required。
- `governance_context`：当 `status = governance_needed` 时 required。
- `conflict_context`：当 `status = rule_conflict` 时 required。
- `invalid_input_errors`：当 `status = invalid_input` 时 required。
Optional fields：
- `candidate_signals`
- `downstream_handoff_hint`
- `runtime_trace`
Validation rules：
- `status = rule_handled` 时必须能追溯到 active approved rule、active entity、approved alias、confirmed required role/context 和允许 rule-based automation 的 policy。
- `status = rule_miss` 不能包含 authority block；如果存在 authority block，应使用 `rule_blocked`、`review_required` 或 `governance_needed`。
- `runtime_trace` 不能包含 raw prompt trace 或未授权内部实现细节；它也不能成为 durable authority。
Durable memory boundary：
- `rule_match_result` 是 runtime output。
- 它可以被 downstream Transaction Logging Node 作为 audit input 保存，但本节点不写 `Transaction Log`。
- `candidate_signals` 只提出候选，不写 durable memory。

#### `matched_rule`

Required fields：
- `rule_id`
- `rule_version`
- `entity_id`
- `authority_source`
- `matched_conditions`
Optional fields：
- `rule_name`
- `governance_refs`
Validation rules：
- `rule_id` 必须来自 `Rule Log`。
- `authority_source` 必须表示 accountant / governance approved authority。
- `matched_conditions` 只能记录实际满足的 approved rule conditions。
Memory boundary：
- `matched_rule` 是对 durable rule 的 runtime reference。
- 本节点不改变 rule record。

#### `rule_output_payload`

Required fields：
- `source_rule_id`
- `approved_accounting_treatment`
- `classification_authority`；allowed value for this node: `approved_rule`
Optional fields：
- `review_requirement`
- `accounting_notes`
- `evidence_refs_used`
Validation rules：
- `approved_accounting_treatment` 必须原样来自 matched approved rule，不能由本节点改写。
- 如果 matched rule 本身要求 review，本节点不能输出纯 automatic `rule_handled`；应输出 `review_required` 或在 `review_requirement` 中明确下游必须 review，具体 routing 见 Open Contract Boundaries。
Memory boundary：
- runtime-only handoff。
- 下游可将其用于 JE Generation、review context 或 Transaction Log audit input。

#### `miss_context`

Required fields：
- `miss_reason`；allowed values: `no_active_rule_for_entity_context`, `condition_not_satisfied`, `no_rule_after_policy_allowed_attempt`。
- `entity_id`
Optional fields：
- `evaluated_rule_ids`
- `condition_mismatch_summary`
- `case_judgment_allowed`：boolean
Validation rules：
- `miss_context` 不能用于 authority 不足的情况。
- `case_judgment_allowed` 不能覆盖 entity / policy / governance hard block；它只表示没有 rule hit 后是否可以进入 Case Judgment。
Memory boundary：
- runtime-only。
- 不能自动创建 rule candidate 或 rule promotion。

#### `blocked_context`

Required fields：
- `blocked_reason`；allowed values: `entity_not_active`, `new_entity_candidate`, `ambiguous_entity`, `unresolved_entity`, `alias_not_approved`, `alias_rejected`, `role_unconfirmed`, `automation_policy_block`, `rule_not_approved_or_not_active`, `governance_block`。
Optional fields：
- `entity_id`
- `matched_alias`
- `alias_status`
- `candidate_role`
- `related_rule_ids`
- `authority_refs`
Validation rules：
- blocked reason 是 hard authority limit，不能被 LLM 或 Case Judgment 自行解除。
- `candidate_role` 只能解释 block，不能写入 durable role。
Memory boundary：
- runtime-only。
- 可生成 candidate signal，但不写 durable memory。

#### `review_context`

Required fields：
- `review_reason`；allowed values: `rule_requires_review`, `policy_requires_review`, `alias_or_role_dispute`, `condition_requires_accountant_confirmation`, `upstream_authority_issue`。
Optional fields：
- `question_focus`
- `related_entity_ids`
- `related_rule_ids`
- `evidence_refs`
Validation rules：
- review context 不能表达 approval。
- `question_focus` 只能帮助下游提问，不能替 accountant 作答。
Memory boundary：
- runtime-only。
- accountant 的回答若要成为 durable authority，必须走 Review / Governance authority path。

#### `governance_context`

Required fields：
- `governance_need_type`；allowed values: `rule_approval_needed`, `rule_conflict_resolution_needed`, `rule_lifecycle_review_needed`, `alias_approval_needed`, `role_confirmation_needed`, `automation_policy_decision_needed`, `entity_lifecycle_review_needed`。
Optional fields：
- `candidate_signal_refs`
- `related_entity_ids`
- `related_rule_ids`
- `reason`
Validation rules：
- governance context 不是 `Governance Log` write。
- 本节点不能批准、拒绝或应用 governance changes。
Memory boundary：
- candidate-only。
- 只有 Review / Governance Review / memory update workflow 可以把它转成 durable governance event。

#### `conflict_context`

Required fields：
- `conflict_type`；allowed values: `multiple_active_rules`, `rule_vs_policy`, `rule_vs_governance_constraint`, `entity_alias_role_conflict`, `rule_condition_overlap`。
- `conflicting_refs`
- `reason`
Optional fields：
- `recommended_downstream_review_scope`
Validation rules：
- 存在 unresolved conflict 时，不能输出 `rule_handled`。
- 本节点不能用 LLM、confidence 或 fallback preference 选择一个 rule 覆盖冲突。
Memory boundary：
- runtime-only conflict signal。
- 是否形成 durable governance event 由后续 workflow 决定。

#### `candidate_signals`

Allowed `signal_type` values：
- `rule_candidate`
- `rule_condition_gap`
- `rule_conflict_review`
- `rule_stale_or_unstable`
- `alias_confirmation_needed`
- `role_confirmation_needed`
- `automation_policy_review`
Required fields per signal：
- `signal_type`
- `reason`
- `related_refs`
Optional fields：
- `evidence_refs`
- `suggested_review_focus`
Validation rules：
- signal 不能包含 `approval_status = approved`。
- signal 不能直接写入 `Rule Log`、`Entity Log` 或 `Governance Log`。
- signal 不能支持当前交易的 rule match。
Memory boundary：
- candidate-only。
- 长期化只能通过 accountant / governance approval。

### Field Authority and Memory Boundary

#### Source of truth

- `transaction_id` 的 source of truth 是 Transaction Identity layer。
- `amount_abs`、`direction`、`transaction_date`、`bank_account`、evidence references 的 source of truth 是 Evidence Intake / Preprocessing 和 Evidence Log。
- `entity_id`、approved aliases、confirmed roles、entity status、automation policy 的 source of truth 是 `Entity Log` 加已生效 governance events。
- active approved rules、rule status、approved condition set、approved accounting treatment 的 source of truth 是 `Rule Log`。
- effective governance restrictions 的 source of truth 是 `Governance Log`。
- accountant intervention 只有在转化为 review-required constraint、policy change、rule status 或 governance event 后，才影响本节点 authority。

#### Fields that can never become durable memory by this node

本节点不能把以下字段直接写成 durable memory：
- `candidate_role`
- `candidate_signals`
- `miss_context`
- `blocked_context`
- `review_context`
- `governance_context`
- `conflict_context`
- `runtime_trace`
- rule miss / condition mismatch summary
- LLM-readable explanation

#### Fields that can become durable only after accountant / governance approval

以下内容只有经过 accountant / governance approval 才可能长期化：
- new rule / rule promotion
- rule condition change
- rule lifecycle change
- alias approval / rejection
- role confirmation
- automation policy upgrade or relaxation
- entity lifecycle change
- governance event resolution

#### Audit vs learning / logging boundary

`Transaction Log` 是 audit-facing final record，不参与 runtime rule decision。

### Validation Rules

#### Contract-level validation rules

- Input 必须有稳定 `transaction_id`、objective transaction basis、completed entity-resolution handoff 和 rule authority snapshot。
- `rule_handled` 必须同时满足：active entity、approved alias、confirmed required role/context、policy 允许 rule-based automation、active approved rule、approved rule conditions satisfied。
- 所有 authority-bearing fields 必须有 source reference。
- Candidate-only fields 不能出现在 authority position。
- Runtime-only trace / explanation 不能成为 durable decision source。

#### Conditions that make input invalid

- 缺少 `transaction_id`。
- 缺少 `entity_resolution_output.status`。
- `entity_resolution_completed != true`。
- `structural_path_status = handled`。
- `transaction_identity_status` 不是 `assigned`。
- `amount_abs <= 0`。
- `direction` 不在 `inflow` / `outflow`。
- `evidence_refs` 为空。
- rule records 缺少 `rule_id`、`rule_status`、`approval_status` 或 `authority_source`，但请求要求本节点判断 rule match。

#### Conditions that make output invalid

- `status = rule_handled` 但没有 `matched_rule` 或 `rule_output_payload`。
- `status = rule_handled` 但 rule 不是 active approved。
- `status = rule_handled` 但 entity 不是 active。
- `status = rule_handled` 但 alias 是 candidate / rejected / missing required approved alias。
- `status = rule_handled` 但 required role/context 未 confirmed。
- `status = rule_handled` 但 automation policy 是 `review_required` 或 `disabled`。
- `status = rule_miss` 但实际原因是 authority block。
- `candidate_signals` 被标记为 approved 或 durable write。
- output 暗示本节点已经写入 `Transaction Log`、`Entity Log`、`Rule Log` 或 `Governance Log`。

#### Stop / ask conditions for unresolved contract authority

如果后续设计要求本节点直接决定以下事项，应停止并要求 product decision：
- `case_allowed_but_no_promotion` 是否一律允许 rule-based automation，而不是依赖 effective policy metadata。
- Rule Log 的完整 lifecycle enum 是否要在 Stage 3 全局冻结。
- `approved_accounting_treatment` 的 exact JE-ready schema 是否由 Rule Match Stage 3 定义，还是由 JE Generation / shared contract 定义。
- matched rule 自带 review requirement 时，`rule_handled + review_requirement` 与 `review_required` 的 routing 语义如何统一。
- 多条 active rules 同时满足时是否存在 deterministic priority，而不是一律 `rule_conflict`。

## 4. Open Boundaries

### Stage 1: Open Boundaries

以下问题留到后续阶段，不在 Stage 1 冻结：
- rule 条件的 exact input / output schema、字段名和对象结构。
- exact routing enum：rule hit、rule miss、rule blocked、review-required 等状态如何命名。
- role/context requirement 与 automation policy 的精确组合规则。
- 多条 active rules 同时可能适用时的 exact conflict handling。
- 非 active rule、rule stability concern、recent intervention 或 lint warning 如何通过治理状态影响 future match。
- rule miss、rule blocked 和 rule candidate signal 的 exact downstream handoff contract。
- rule-handled result 与 JE generation、review 和 transaction logging 的 exact contract。
- accountant 补充确认或 governance approval 后是否重新触发本节点。
- execution algorithm、storage path、test matrix 和 coding-agent task contract。

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
- role/context requirement 与 rule applicability 的 exact contract
- 多条 active rules 同时适用时的 exact conflict handling
- rule lifecycle states 与 governance state 的 exact contract
- rule miss、rule blocked、rule conflict、rule candidate signal 的 exact downstream routing
- accountant 补充确认后是否重新触发 Rule Match
- deterministic rule-handled result 与 JE generation、review、transaction logging 的 exact handoff contract

### Stage 3: Open Contract Boundaries

- `Rule Log` 的完整全局 lifecycle enum 仍未在 active docs 中冻结；本文件只定义 Rule Match Node 必须理解的 node-facing statuses。
- `approved_accounting_treatment` 的 exact shared accounting payload / JE-ready schema 需要与 JE Generation、Review、Transaction Logging 的 Stage 3 contracts 对齐。
- `case_allowed_but_no_promotion` 是否默认允许 rule-based automation 仍需更明确的 policy contract；本文件保守要求 effective metadata 明确允许。
- 多条 active approved rules 同时满足时，当前 contract 默认输出 `rule_conflict`；是否存在 deterministic priority / specificity resolution 留待 Stage 4 或 shared rule contract 决定。
- matched rule 自带 review requirement 时，`review_required` 与 `rule_handled` 加 downstream review flag 的关系未完全冻结。
- Accountant 补充确认 alias / role / rule 后是否自动 retrigger Rule Match，不在 Stage 3 冻结。
