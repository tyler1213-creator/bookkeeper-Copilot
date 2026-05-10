# Profile / Structural Match Node — 设计摘要

> 由原 Stage 1/2/3 合并瘦身而来；保留定位、边界、contract 字段和开放问题，删除阶段说明、examples、self-review、历史读档记录和重复解释。

## 1. 定位与职责

Stage 1 已确定以下内容：
- `Profile / Structural Match Node` 是 profile-backed structural transaction gate。
- 它位于 `Transaction Identity Node` 之后、`Entity Resolution Node` 之前。
- 它先处理内部转账、已知贷款还款和其他可由稳定客户结构事实确定的交易。
- 它只基于具备 authority 的 `Profile` 结构事实完成结构性处理。
- profile candidate、未确认关系或模糊结构信号不能被当成稳定 profile truth。
- 内部转账识别属于 bank-raw-signal 语义，不应被强行压成普通 canonical description / entity matching 问题。
- 它不做普通 entity resolution、rule match 或 case judgment。
- 它不做普通交易的 COA / HST / JE 业务判断。
- 它不写 `Transaction Log`。
- 它不直接修改 `Profile`、Entity / Case / Rule / Governance 等长期记忆。

## 2. 逻辑与边界

### Trigger Boundary

`Profile / Structural Match Node` 在主 workflow 中，`Transaction Identity Node` 已经为当前交易建立或复用稳定交易身份之后触发。

概念触发条件是：
- 当前交易已经具备可消费的 objective transaction basis、evidence references 和稳定交易身份；并且
- 后续 workflow 必须先判断它是否属于 profile-backed structural transaction；并且
- 交易尚未进入普通 entity resolution、rule match 或 case judgment。
典型触发场景包括：
- 银行账户之间可能存在内部转账。
- 当前交易可能对应已知贷款还款。
- 当前交易可能被客户 `Profile` 中的稳定结构事实直接解释。
- 当前交易暴露出 profile 缺失、未确认关系或结构事实冲突，需要在进入 entity-first workflow 前显露。

### Input Categories

Stage 2 按判断作用组织 input categories，不列字段清单。

#### Objective transaction basis

说明当前交易中可客观确定的结构事实，例如方向、金额、账户来源、交易日期和其他非会计判断型交易事实。

#### Evidence reference basis

说明当前结构性判断能回到哪些原始 evidence。

#### Stable profile structural basis

说明当前客户 `Profile` 中已经具备 authority 的结构事实。

核心边界：
- 只有 stable profile truth 可以支持结构性完成路径。
- profile candidate、onboarding 推断、runtime 疑似关系或未确认 accountant 说明，不能被本节点当作稳定结构事实。
- `Profile` 是客户结构事实层，不是普通 entity、case、rule 或 transaction audit layer。

#### Bank raw signal basis

说明当前交易是否包含银行原始信号，足以支持内部转账或类似结构关系判断。

#### Structural candidate / issue basis

说明当前交易是否暴露出疑似结构关系、profile 缺失、profile 候选、未确认贷款关系、账户关系不完整或结构事实冲突。

#### Existing workflow context basis

说明上游是否已经暴露 evidence issue、identity issue、duplicate candidate 或补充材料语境。

### Output Categories

Stage 2 只定义 conceptual output categories，不冻结 routing enum 或对象形状。

#### Structural handled path

含义：当前交易可由 stable profile structural basis 确定为结构性交易，并进入结构性处理路径。

边界：
- 这是当前交易的 runtime structural path。
- 它不等于普通 entity classification。
- 它不创建或修改 profile facts。
- 它不批准 governance change。
- 它不写 `Transaction Log`。

#### Non-structural handoff path

含义：当前交易没有被 profile-backed structural facts 确定，应继续进入 `Entity Resolution Node`。

边界：
- 未命中结构性路径不等于实体已知或未知。
- 未命中结构性路径不等于 rule miss。
- 未命中结构性路径不等于 case judgment。
- 本节点不应为了避免下游不确定性而扩张结构性解释。

#### Structural pending / review-needed path

含义：当前交易可能是结构性交易，但缺少关键 profile fact、accountant confirmation、evidence clarification 或 identity/evidence stability，因此不能安全完成结构性处理。

边界：
- 如果问题是可补信息，后续由 `Coordinator / Pending Node` 聚焦提问。
- 如果问题是 profile authority、治理或结构事实冲突，后续应进入 review / governance 相关路径。
- 本节点不自行提问，不自行批准回答。

#### Structural issue signals

含义：本节点可以输出结构性问题语境，例如疑似内部转账未配置、疑似贷款关系未确认、已知 profile relation 与当前 evidence 冲突、结构性 evidence 不足等。

边界：
- issue signal 不是 accountant decision。
- issue signal 不是 governance approval。
- issue signal 不改变 profile。

#### Candidate signals

含义：本节点可以提出后续可能需要评估的候选信号，例如：
- profile fact candidate
- internal transfer relationship candidate
- loan relationship candidate
- profile conflict review candidate
- governance candidate
边界：
- candidate 只供后续节点评估。
- candidate 不写入 stable `Profile`。
- candidate 不支持本笔交易的 structural handled path。
- candidate 不创建 rule、case 或 entity authority。

### Deterministic Code vs LLM Semantic Judgment

Stage 2 的核心边界是：

#### Deterministic code responsibility

Deterministic code 负责：
- 判断本节点是否被触发：上游 evidence intake 和 transaction identity 已完成，且交易尚未进入 entity-first workflow。
- 读取当前客户 stable profile structural basis。
- 判断 profile authority 是否足以支持 structural handled path。
- 执行结构性匹配，例如内部转账、已知贷款还款或其他已确认结构关系。
- 保持 bank-raw-signal 语义与 ordinary canonical description / entity matching 语义分离。
- 区分 structural handled、non-structural handoff、structural pending / review-needed、issue signal 和 candidate signal。
- 防止 profile candidate、未确认关系或模糊信号被当作 stable profile truth。
- 防止结构性交易错误穿透到普通 entity/rule/case 路径。
- 防止普通 vendor/entity 交易被错误截断为结构性交易。
- 防止本节点写入 `Transaction Log` 或修改 Entity / Case / Rule / Governance memory。

#### LLM semantic judgment responsibility

LLM 通常不应拥有结构性匹配 authority。

如果后续阶段允许 LLM 辅助，它最多可以在受限边界内帮助：
- 解释 messy evidence 中的人类可读线索。
- 总结为什么当前交易看起来可能是结构性交易。
- 生成 pending / review explanation。
- 帮助整理 profile issue 或 candidate signal 的可读说明。

#### Hard boundary

LLM 不能：
- 确认内部转账关系
- 确认贷款关系
- 把 profile candidate 升级为 stable profile truth
- 创建或修改 `Profile`
- 把未确认银行原始信号当作结构性完成依据
- 判断普通 vendor/entity identity
- 执行 rule match 或 case judgment
- 选择普通交易 COA / HST 处理
- 生成 journal entry
- 批准 accountant 或 governance decision

### Accountant Authority Boundary

Accountant 仍然拥有最终 accounting decision 和 durable profile authority。

本节点不能：
- 替 accountant 批准新增银行账户关系
- 替 accountant 确认内部转账关系
- 替 accountant 确认贷款关系
- 把 client-provided 或系统推断的结构说明直接写成 stable profile truth
- 把结构性疑似交易当作已确认结构性交易处理
- 替 accountant 解决 profile conflict 的业务含义

### Governance Authority Boundary

Governance-level changes 不属于本节点最终批准 authority。

本节点不能：
- 创建、修改或删除 stable profile facts
- 批准 profile governance event
- approve / reject alias
- confirm role
- create stable entity
- merge / split entity
- promote / modify / delete / downgrade active rule
- upgrade or relax automation policy
- invalidate durable memory

### Memory / Log Boundary

Stage 2 采用三层边界：read / consume、candidate-only、no direct mutation。

#### Read / consume boundary

`Profile / Structural Match Node` 可以读取或消费以下 conceptual context：
- runtime evidence foundation
- objective transaction basis
- evidence references
- stable transaction identity
- current customer `Profile`
- profile authority / confirmation context
- current batch issue context from upstream evidence or identity nodes

#### Candidate-only boundary

本节点只能作为候选或 issue signal 表达：
- profile fact candidate
- internal transfer relationship candidate
- loan relationship candidate
- bank account relationship issue
- profile conflict issue
- missing profile fact issue
- governance / review candidate

#### No direct mutation boundary

本节点绝不能：
- 写入 `Transaction Log`
- 写入或修改 `Profile`
- 写入或修改 `Entity Log`
- 写入或修改 `Case Log`
- 写入或修改 `Rule Log`
- 写入或批准 `Governance Log`
- 创建 stable entity
- 批准 alias、role、rule、automation policy 或 entity governance change
- 把 profile candidate 当作 stable profile truth
- 把 structural candidate 当作 handled transaction

### Insufficient / Ambiguous / Conflicting Evidence Behavior

Stage 2 采用优先级边界：profile authority first、structural certainty second、safe handoff third。

#### Profile authority first

如果缺少已确认 profile fact，本节点不能完成 structural handled path。

典型情况：
- 疑似内部转账，但 profile 中没有已确认关系。
- 疑似贷款还款，但贷款关系或还款语境未确认。
- onboarding 只产生了 profile candidate，尚未获得 authority。
- client 或系统说明尚未被 accountant / governance 确认。

#### Structural certainty second

即使存在 stable profile fact，当前交易证据也必须足以落入该结构关系。

#### Safe handoff third

如果当前交易无法被结构性确定，且不存在必须先处理的 profile authority 问题，应交给 `Entity Resolution Node`。

#### Conflict behavior

如果 current evidence 与 stable profile facts 冲突，本节点应保守处理。

典型边界：
- 冲突可能只是 evidence 缺失或配对不清：输出 pending context。
- 冲突可能说明 profile 已过期或错误：输出 review / governance candidate。
- 冲突影响当前交易是否结构性：不能完成 structural handled path。

#### Hard boundary

- Profile ambiguity 不等于可以猜。
- Profile candidate 不等于 stable profile truth。
- Bank raw signal 不等于 profile authority。
- 结构性疑似不等于结构性完成。
- 当前交易的结构性处理不能创建长期 profile authority。
- `Transaction Log` 不参与 runtime structural decision。

## 3. Contract 字段摘要

### Contract Position in Workflow

`Profile / Structural Match Node` 位于：

#### Upstream Handoff Consumed

本节点消费 runtime `profile_structural_match_request`。

上游必须已经完成或明确给出：
- objective transaction basis；
- evidence references；
- stable `transaction_id`；
- current batch / client context；
- identity / duplicate 状态；
- 可供读取的 current `Profile` structural snapshot。

#### Downstream Handoff Produced

本节点输出 runtime `profile_structural_match_result`。

下游按结果类别消费：
- `Entity Resolution Node`：只消费 `non_structural_entity_resolution_handoff`，表示当前交易未被结构性路径完成。
- `Coordinator / Pending Node`：消费 `structural_pending_handoff` 和聚焦问题语境。
- `Review Node` / `Governance Review Node`：消费 `structural_issue_signal` 或 `profile_governance_candidate_signal`；这些 signal 不是 approval。
- `JE Generation Node`：可消费 `structural_handled_result` 中的 approved structural accounting basis；具体 journal entry 仍由 JE Generation Node 计算。
- `Transaction Logging Node`：不直接消费本节点作为 runtime decision source；最终审计记录由后续 final outcome / JE / review flow 进入 Transaction Logging Node。

#### Logs / Memory Stores Read

本节点可以读取或消费：
- `Evidence Log` references：用于追溯 current transaction evidence，不作为业务结论。
- stable transaction identity：来自 `Transaction Identity Node`。
- `Profile`：客户结构事实，例如 bank accounts、internal transfer relationships、loan relationships、tax config 或其他已确认结构 facts。
- profile authority / confirmation context：只用于判断某个 profile fact 是否具备 stable authority。
- upstream evidence / identity issue context：用于限制结构性判断。
本节点不读取：
- `Transaction Log` 做 runtime structural decision；
- `Entity Log` 做普通 entity identity；
- `Case Log` 做 case precedent；
- `Rule Log` 做 deterministic rule match；
- `Knowledge Log / Summary Log` 作为 stable profile authority。

#### Logs / Memory Stores Written or Candidate-Only

本节点不直接写入任何 durable log / memory store。

它不能写：
- `Transaction Log`
- `Profile`
- `Entity Log`
- `Case Log`
- `Rule Log`
- `Governance Log`
- `Knowledge Log / Summary Log`
- approved profile fact、stable entity、approved alias、confirmed role、active rule 或 automation policy
本节点只可以输出 candidate-only / issue signals，例如：
- `profile_fact_candidate_signal`
- `internal_transfer_relationship_candidate`
- `loan_relationship_candidate`
- `bank_account_relationship_issue`
- `profile_conflict_issue`
- `missing_profile_fact_issue`
- `profile_governance_candidate_signal`

### Input Contracts

#### `profile_structural_match_request`

Optional fields：
- `bank_raw_signal_basis`：银行原始信号视图，用于 internal-transfer 或类似 structural relation 的 raw-signal matching。
- `structural_candidate_issue_context`：上游或当前批次暴露的 profile candidate、missing-profile、conflict 或 accountant-context signal。
- `request_trace`：runtime diagnostics；不能作为 profile authority。
Validation / rejection rules：
- 缺少 `transaction_basis.transaction_id`、`transaction_basis.client_id`、`transaction_basis.evidence_refs` 或 `profile_structural_snapshot.client_id` 时，input invalid。
- `transaction_basis.client_id` 必须与 `profile_structural_snapshot.client_id` 一致。
- `upstream_workflow_context.evidence_status` 必须表示 evidence foundation ready。
- `upstream_workflow_context.transaction_identity_status` 必须表示 stable identity assigned。
- 如果上游已标记 duplicate blocked、identity blocked 或 evidence blocked，本节点不能输出 `structural_handled`。
- `request_trace`、candidate context、LLM explanation 或 raw bank signal 不能替代 stable profile authority。
Runtime-only vs durable references：
- `profile_structural_match_request` 是 runtime-only envelope。
- `transaction_id`、`client_id`、`evidence_refs`、profile fact refs 是 durable references。
- 本节点收到 request 不表示拥有任何 durable write authority。

#### `transaction_basis`

Allowed `direction` values：
- `inflow`
- `outflow`
Optional fields：
- `raw_description`：银行原始描述；可用于 bank raw signal，但不是普通 entity identity authority。
- `description`：兼容或展示用 canonical description；本节点不能依赖它替代 bank raw signal。
- `pattern_source`：兼容字段；可为 `null`。
- `currency`
- `receipt_refs`
- `cheque_info_refs`
- `statement_line_ref`
- `counterparty_surface_text`
- `objective_tags`
Validation / rejection rules：
- `amount_abs` 必须为正数；不得用 signed amount 重复表达 direction。
- `direction` 必须显式给出，不能从金额正负号推断。
- `evidence_refs` 不能为空。
- `bank_account` 必须存在；否则本节点不能判断 bank-account structural relation。
- `description = null` 和 `pattern_source = null` 是有效状态；本节点不能要求 canonical description。
- `raw_description` 缺失时，涉及 bank-raw-signal 的结构匹配不能完成，但非 bank-raw-signal 的 stable profile relation 仍可在证据足够时被评估。
Runtime-only vs durable references：
- 客观字段来自 current transaction record。
- `transaction_id` 和 `evidence_refs` 是 durable references。
- `raw_description` 是 evidence surface，不是 durable profile fact。

#### `upstream_workflow_context`

Allowed `evidence_status` values：
- `ready`
- `blocked`
Allowed `transaction_identity_status` values：
- `assigned`
- `blocked`
Allowed `duplicate_status` values：
- `not_duplicate`
- `duplicate_blocked`
- `duplicate_warning`
Optional fields：
- `upstream_issue_flags`
- `identity_issue_refs`
- `evidence_issue_refs`
- `accountant_context_refs`
- `profile_context_refs`
Validation / rejection rules：
- `evidence_status != ready` 时，本节点不能输出 `structural_handled`。
- `transaction_identity_status != assigned` 时，本节点输入无效。
- `duplicate_status = duplicate_blocked` 时，本节点不能继续当前交易 structural handling。
- `already_entered_entity_workflow = true` 时，本节点输入无效；本节点不能在普通 entity / rule / case path 后回头吸收下游职责。
- `upstream_issue_flags` 可以限制输出，但不能单独支持 structural handled。
Runtime-only vs durable references：
- 该对象是 runtime handoff。
- issue refs 如存在，是 durable logs 或 current batch issue 的引用；本节点不能修改它们。

#### `profile_structural_snapshot`

Allowed `account_status` values：
- `active`
- `inactive`
- `archived`
Allowed `structural_type` values：
- `internal_transfer_relationship`
- `known_loan_repayment_relationship`
- `confirmed_bank_account_relationship`
- `other_confirmed_profile_structural_fact`
Allowed `authority_status` values：
- `accountant_confirmed`
- `governance_approved`
- `approved_onboarding_import`
- `candidate`
- `unconfirmed`
- `rejected`
Allowed `effective_status` values：
- `active`
- `inactive`
- `archived`
- `conflicted`
Optional fields：
- `tax_config_refs`
- `loan_account_refs`
- `related_bank_account_refs`
- `profile_risk_flags`
- `governance_constraint_refs`
- `profile_notes_refs`
Validation / rejection rules：
- Only `authority_status in {accountant_confirmed, governance_approved, approved_onboarding_import}` and `effective_status = active` can support `structural_handled`.
- `candidate`、`unconfirmed`、`rejected`、`conflicted`、`inactive` 或 `archived` facts 不能支持 `structural_handled`。
- `other_confirmed_profile_structural_fact` 只能在 `Profile` 已经保存 approved structural accounting basis 时使用；不能作为普通 vendor/entity 分类的兜底桶。
- `structural_accounting_basis` 缺失时，不能输出 finalizable `structural_handled_result`，但可以输出 review / pending / issue context。
- `Knowledge Log / Summary Log` 摘要不能填充 `authority_context`。
Runtime-only vs durable references：
- snapshot 是 runtime read view。
- profile facts、profile version、evidence refs 和 governance refs 是 durable references。
- 本节点不能修改 snapshot 或将 candidate fact 写回 `Profile`。

#### `bank_raw_signal_basis`

Optional fields：
- `bank_counterparty_text`
- `bank_reference_number`
- `bank_trace_id`
- `paired_statement_line_refs`
- `raw_direction_marker`
- `raw_account_marker`
- `raw_signal_quality`
Allowed `raw_signal_quality` values：
- `strong`
- `usable`
- `weak`
- `conflicting`
- `missing`
Validation / rejection rules：
- `bank_raw_signal_basis` 只能支持 stable profile fact 已授权的 structural relation matching。
- `raw_description` 或 paired raw markers 不能单独创建 internal transfer relationship。
- `description` / canonical pattern 不能替代 `raw_description` 作为 bank-raw-signal authority。
- `raw_signal_quality in {weak, conflicting, missing}` 时，不能单独支持 `structural_handled`。
Runtime-only vs durable references：
- raw signal object 是 runtime-only。
- `source_evidence_ref` 和 paired evidence refs 是 durable references。
- raw signal 可被 candidate signal 引用，但不能由本节点写入 stable profile truth。

#### `structural_candidate_issue_context`

Optional fields：
- `profile_fact_candidates`
- `missing_profile_fact_signals`
- `profile_conflict_signals`
- `unconfirmed_accountant_context_refs`
- `onboarding_candidate_refs`
- `prior_profile_issue_refs`
Allowed `signal_type` values：
- `internal_transfer_relationship_candidate`
- `loan_relationship_candidate`
- `missing_bank_account_relationship`
- `profile_fact_conflict`
- `profile_fact_missing`
- `evidence_clarification_needed`
- `governance_review_candidate`
Validation / rejection rules：
- Candidate / issue context 可以支持 `structural_pending`、`structural_review_needed` 或 candidate signal。
- Candidate / issue context 不能支持 `structural_handled`，除非同一 profile fact 已在 `profile_structural_snapshot` 中以 approved authority 出现。
- accountant context ref 若未被 review / governance 确认，不能被当作 stable profile truth。
Runtime-only vs durable references：
- issue context 是 runtime-only。
- evidence refs、review refs、onboarding refs 如存在，是 durable references。
- 本节点只能转发或组织 signal，不能把 signal 变成 durable approval。

### Output Contracts

#### `profile_structural_match_result`

Allowed `structural_match_status` values：
- `structural_handled`
- `non_structural_handoff`
- `structural_pending`
- `structural_review_needed`
- `input_invalid`
Required by status：
- `structural_handled` requires `structural_handled_result`。
- `non_structural_handoff` requires `non_structural_entity_resolution_handoff`。
- `structural_pending` requires `structural_pending_handoff`。
- `structural_review_needed` requires at least one `structural_issue_signal` or `profile_governance_candidate_signal`。
- `input_invalid` requires `invalid_reason` and must not produce downstream automation handoff。
Optional fields：
- `structural_issue_signals`
- `profile_governance_candidate_signals`
- `request_trace`
- `diagnostic_notes`
Validation rules：
- Exactly one primary status must be present.
- `structural_handled` cannot coexist with `non_structural_entity_resolution_handoff` as an active route.
- `structural_pending` / `structural_review_needed` cannot be disguised as ordinary `non_structural_handoff` if the blocking issue is profile authority or structural conflict.
- `input_invalid` is a contract failure status, not a normal business route.
- `authority_trace` must not cite `Transaction Log`, `Case Log`, `Rule Log` or `Knowledge Log / Summary Log` as runtime structural authority.
Memory boundary：
- Entire result is runtime-only unless downstream Transaction Logging / Review / Governance workflow later records a final result or approved governance event.
- 本节点不能持久化该 result。

#### `structural_handled_result`

Allowed `structural_type` values：
- `internal_transfer`
- `known_loan_repayment`
- `confirmed_bank_account_structural_transaction`
- `other_confirmed_profile_structural_transaction`
Allowed `matched_profile_fact_authority` values：
- `accountant_confirmed`
- `governance_approved`
- `approved_onboarding_import`
Optional fields：
- `related_bank_account_refs`
- `loan_account_refs`
- `counterparty_bank_account_ref`
- `tax_config_refs`
- `structural_limit_notes`
- `review_visibility_hint`
Validation rules：
- `matched_profile_fact_id` must refer to an `active` stable profile fact in `profile_structural_snapshot`。
- `matched_profile_fact_authority` cannot be `candidate`、`unconfirmed`、`rejected`、`conflicted`、`inactive` 或 `archived`。
- `matched_evidence_refs` must include evidence supporting this transaction, not only the historical evidence that created the profile fact。
- `structural_accounting_basis` must be an approved reference, not a newly inferred ordinary classification。
- `other_confirmed_profile_structural_transaction` must not be used to bypass Entity Resolution for normal vendor / payee / counterparty transactions。
- This output must not contain final JE lines; JE computation belongs to JE Generation Node。
Durable memory / candidate boundary：
- Runtime-only structural outcome.
- Does not write `Transaction Log`.
- Does not update `Profile`.
- May be included later in final Transaction Log only through Transaction Logging Node after downstream finalization.

#### `non_structural_entity_resolution_handoff`

Optional fields：
- `upstream_issue_flags`
- `accountant_context_refs`
- `profile_context_refs`
- `non_blocking_structural_notes`
Validation rules：
- Cannot be produced if a stable profile fact was matched strongly enough for `structural_handled`。
- Cannot be produced if the current blocker is missing profile authority, profile conflict, or evidence needed to decide a likely structural transaction; those must use `structural_pending` or `structural_review_needed`。
- Must preserve evidence refs and objective transaction basis; cannot pass only a summary.
- Must not include entity_id, rule_id, case outcome, COA classification, HST/GST treatment, or JE lines.
Memory boundary：
- Runtime-only handoff.
- Does not write durable memory.

#### `structural_pending_handoff`

Allowed `pending_category` values：
- `missing_profile_confirmation`
- `missing_evidence`
- `unclear_internal_transfer_pairing`
- `unconfirmed_loan_relationship`
- `bank_account_relationship_missing`
- `identity_or_duplicate_dependency`
Optional fields：
- `candidate_profile_fact_refs`
- `suggested_accountant_question`
- `blocking_profile_fields`
- `related_transaction_refs`
- `profile_context_refs`
Validation rules：
- `structural_pending_handoff` cannot contain a stable profile mutation instruction。
- `suggested_accountant_question` is advisory;本节点不自行提问、不自行批准回答。
- If the issue is a durable profile conflict rather than missing information, use `structural_review_needed` instead。
- Candidate refs cannot be used by this node to complete the current transaction。
Memory boundary：
- Runtime-only pending handoff.
- Accountant answer may later become durable only through Review / Governance / Profile update authority path.

#### `structural_issue_signal`

Allowed `signal_type` values：
- `missing_profile_fact_issue`
- `profile_conflict_issue`
- `bank_account_relationship_issue`
- `internal_transfer_pairing_issue`
- `loan_relationship_issue`
- `profile_authority_issue`
- `evidence_structural_conflict`
Allowed `severity` values：
- `info`
- `warning`
- `blocking`
Optional fields：
- `related_profile_fact_ids`
- `candidate_profile_fact_refs`
- `related_transaction_refs`
- `recommended_downstream_owner`
Allowed `recommended_downstream_owner` values：
- `coordinator_pending`
- `review_node`
- `governance_review`
- `post_batch_lint`
Validation rules：
- Blocking issue cannot be paired with active `non_structural_handoff` unless the blocker is explicitly non-structural and safe to ignore for entity workflow。
- Signal cannot update Profile or Governance Log by itself。
- Signal must include evidence or authority refs sufficient for downstream review; pure model assertion is invalid。
Memory boundary：
- Runtime-only issue signal.
- May become durable only if later written by the appropriate downstream log / governance process.

#### `profile_governance_candidate_signal`

Allowed `candidate_type` values：
- `profile_fact_candidate`
- `internal_transfer_relationship_candidate`
- `loan_relationship_candidate`
- `bank_account_relationship_candidate`
- `profile_fact_correction_candidate`
- `profile_conflict_review_candidate`
Optional fields：
- `proposed_profile_fact_shape`
- `related_profile_fact_ids`
- `related_transaction_refs`
- `authority_gap`
- `risk_notes`
Validation rules：
- `requires_accountant_or_governance_approval` must be `true`。
- `proposed_profile_fact_shape` is candidate-only and cannot be treated as stable profile schema or write instruction。
- Candidate cannot support `structural_handled` for the current transaction。
- Candidate cannot create Entity / Case / Rule authority。
Memory boundary：
- Runtime-only candidate.
- Can become durable only after accountant / governance approval through the proper Profile / Governance authority path.

### Field Authority and Memory Boundary

#### Source of Truth for Important Fields

- `transaction_id`：source of truth 是 `Transaction Identity Node` / identity registry；本节点只引用。
- `amount_abs`、`direction`、`transaction_date`、`bank_account`、`raw_description`、`evidence_refs`：source of truth 是 Evidence Intake / Evidence Log；本节点只消费。
- `client_id`、`batch_id`：source of truth 是 workflow / client context。
- `Profile` structural facts、bank accounts、loan relationships、internal transfer relationships、tax/profile config：source of truth 是 `Profile`，并受 accountant / governance authority 控制。
- `profile_fact_id`、`authority_status`、`effective_status`、`structural_accounting_basis`：source of truth 是 `Profile` projection over approved Review / Governance / onboarding-import authority。
- `structural_match_status`、`reason`、runtime issue / candidate signals：source of truth 是本节点的 runtime result；它们不是 durable memory。

#### Fields That Can Never Become Durable Memory by This Node

本节点绝不能把以下字段直接写成长记忆：
- `structural_match_status`
- `reason`
- `request_trace`
- `diagnostic_notes`
- `bank_raw_signal_basis`
- `structural_pending_handoff`
- `structural_issue_signal`
- `profile_governance_candidate_signal`
- `proposed_profile_fact_shape`
- any LLM explanation or messy-evidence interpretation
- any candidate relationship inferred from current transaction

#### Fields That Can Become Durable Only After Accountant / Governance Approval

以下内容只有经过 accountant / governance / approved Profile update path 后，才可能成为 durable `Profile` 或 `Governance Log` authority：
- 新 bank account relationship
- internal transfer relationship
- loan relationship
- profile fact correction
- profile conflict resolution
- structural matching condition change
- structural accounting basis change
- tax/profile config change

#### Audit vs Learning / Logging Boundary

- `Transaction Log` 是 audit-facing final record，不参与本节点 runtime decision。
- 本节点不写 `Transaction Log`；后续 finalization / JE / logging flow 才决定如何记录最终处理结果。
- Issue / candidate signals 不是 learning layer，不能直接进入 `Case Log`、`Rule Log` 或 `Profile`。
- 当前交易若由 structural path 完成，其 final audit trace 可以 later include matched profile fact and evidence refs，但这不是本节点直接写日志。
- Runtime structural candidate 不得污染 rule promotion、case memory 或 entity authority。

### Validation Rules

#### Contract-Level Validation Rules

- 输入必须绑定同一 `client_id`、稳定 `transaction_id` 和至少一个 `evidence_ref`。
- `amount_abs` 必须是绝对值；`direction` 必须显式使用 `inflow` / `outflow`。
- `Profile` snapshot 只能作为 read-only view。
- `structural_handled` 只能基于 active stable profile fact。
- Candidate、unconfirmed、conflicted、inactive、archived 或 rejected profile facts 不能支持 `structural_handled`。
- Bank raw signal 只能在 stable profile fact 允许的结构关系内使用。
- `Transaction Log`、`Case Log`、`Rule Log`、`Entity Log`、`Knowledge Log / Summary Log` 不能成为本节点 structural authority。
- Output 必须只有一个 active primary route。

#### Conditions That Make the Input Invalid

输入无效条件包括：
- 缺少 `transaction_id`、`client_id`、`transaction_basis`、`evidence_refs` 或 `profile_structural_snapshot`。
- `transaction_basis.client_id` 与 `profile_structural_snapshot.client_id` 不一致。
- evidence intake 未 ready。
- transaction identity 未 assigned。
- duplicate 已 blocked。
- 当前交易已经进入 entity-first workflow 后又回到本节点。
- profile snapshot 不能表明相关 structural facts 的 authority / effective status。
- request 试图把 candidate context、raw bank signal、LLM explanation 或 summary log 当成 stable profile authority。

#### Conditions That Make the Output Invalid

输出无效条件包括：
- `structural_handled` 缺少 `structural_handled_result`。
- `structural_handled_result` 引用 candidate / unconfirmed / conflicted / inactive / archived profile fact。
- `structural_handled_result` 缺少 current transaction evidence refs。
- `structural_handled_result` 包含 final JE lines 或普通 vendor/entity classification。
- `non_structural_entity_resolution_handoff` 在存在 profile-authority blocker 或 structural conflict 时被输出。
- `structural_pending_handoff` 包含 profile write instruction 或 accountant approval。
- `profile_governance_candidate_signal.requires_accountant_or_governance_approval != true`。
- result 同时声明 active `structural_handled` 和 active `non_structural_handoff`。
- output 直接写入或声称写入 `Profile`、`Transaction Log`、`Entity Log`、`Case Log`、`Rule Log` 或 `Governance Log`。

#### Stop / Ask Conditions for Unresolved Contract Authority

后续设计或实现遇到以下情况必须停下问产品 / spec authority，不能自行补：
- 需要新增 stable profile fact authority category。
- 需要把 `other_confirmed_profile_structural_fact` 扩展成新的 structural transaction type。
- 需要决定 profile candidate 到 stable profile truth 的具体审批 workflow。
- 需要决定 accountant 回答后是否重触发本节点，以及重触发范围。
- 需要冻结 structural handled path 与 JE Generation 的完整 accounting outcome schema。
- 需要让本节点读取 Transaction Log、Case Log、Rule Log 或 Knowledge Summary 做 runtime decision。

## 4. Open Boundaries

### Stage 1: Open Boundaries

以下问题留到后续阶段，不在 Stage 1 冻结：
- 哪些 profile facts 足以支持结构性确定处理。
- profile candidate 到 stable profile truth 的确认 workflow。
- 结构性未命中、结构性疑似、profile 缺失和 profile 冲突的精确下游路径。
- 结构性处理结果与后续 journal entry generation 的 exact contract。
- 本节点可以提出哪些 profile / governance candidate signals。
- 补充 evidence 或 accountant 回答后是否重新触发本节点，以及如何限制重跑边界。
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
- stable profile fact 的 exact authority categories
- structural handled path 与 JE generation 的 exact handoff contract
- profile candidate 到 stable profile truth 的 confirmation workflow
- accountant 补充确认后是否重新触发本节点
- profile conflict 的人工处理与 governance workflow 边界

### Stage 3: Open Contract Boundaries

- Stable profile fact 的 exact authority categories 仍未由 active docs 全局冻结。本文件只为本节点定义最小可用 authority labels；未来若 Profile 全局 contract 收敛，应以全局 contract 为准并回填本文件。
- `other_confirmed_profile_structural_fact` 的具体子类型未冻结。它只能表示已经由 `Profile` 明确保存 approved structural accounting basis 的结构事实，不能被实现层扩展成普通交易分类兜底。
- `structural_handled_result` 与 JE Generation Node 的完整 accounting outcome handoff 仍需后续跨节点收敛。本文件只要求提供 approved structural accounting basis，不定义 JE lines 或完整 finalizable outcome schema。
- profile candidate 到 stable profile truth 的 confirmation workflow 未冻结；本节点只输出 candidate signal。
- accountant 补充确认后是否重新触发本节点、如何限制重跑边界，仍未冻结。
- profile conflict 的人工处理与 Governance Review 的 exact event schema 未在本节点冻结。
