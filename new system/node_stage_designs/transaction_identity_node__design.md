# Transaction Identity Node — 设计摘要

> 由原 Stage 1/2/3 合并瘦身而来；保留定位、边界、contract 字段和开放问题，删除阶段说明、examples、self-review、历史读档记录和重复解释。

## 1. 定位与职责

Stage 1 已确定以下内容：
- `Transaction Identity Node` 是永久交易身份与去重节点。
- 它发生在 evidence intake 之后、结构性匹配和 entity resolution 之前。
- 它基于客观交易结构和 evidence references 分配或复用稳定 `transaction_id`。
- 它处理同一交易识别和重复导入问题。
- 永久交易身份需要 durable identity registry / ingestion registry；身份不能仅靠日期序号重建。
- 它不解析原始 evidence，不保存业务结论。
- 它不做 profile / structural match、entity resolution、rule match 或 case judgment。
- 它不做会计分类、HST/GST 判断或 journal entry generation。
- 它不写 `Transaction Log`。
- 它不修改 Entity / Case / Rule / Governance 等长期业务记忆。
- 它不把 transient handoff、重复候选、临时队列或 report draft 称为 `Log`。

## 2. 逻辑与边界

### Trigger Boundary

`Transaction Identity Node` 在主 workflow 中，`Evidence Intake / Preprocessing Node` 已经整理出可消费的 runtime evidence foundation 之后触发。

概念触发条件是：
- 当前材料已经具备客观交易结构和 evidence references；并且
- 系统需要判断这是否是一笔新交易、已存在交易、重复导入、或疑似同一交易；并且
- 后续 `Profile / Structural Match Node`、`Entity Resolution Node`、`Rule Match Node`、`Case Judgment Node` 或 review / logging flow 需要稳定交易身份才能继续。
典型触发场景包括：
- 新银行或支付账户交易进入主 workflow。
- 同一 batch 中出现可能重复的交易材料。
- 后续补交的小票、支票、invoice、contract 或 accountant note 可能属于已存在交易。
- 重新导入或 re-intake 材料可能与已登记交易重叠。

### Input Categories

Stage 2 按判断作用组织 input categories，不列字段清单。

#### Objective transaction basis

说明当前材料中可客观确定的交易事实。

#### Evidence reference basis

说明当前交易材料能回到哪些原始 evidence references。

#### Source / import context basis

说明材料来自哪个导入来源、批次、账户来源、补交渠道或历史初始化语境。

这一类输入帮助区分：
- 首次导入
- 重复导入
- 同源修正
- 跨来源补充 evidence
- historical onboarding 与 runtime processing 的潜在衔接

#### Association / pairing context basis

说明上游 evidence intake 已经提出哪些客观关联、候选关联、无法关联或重复材料信号。

#### Existing identity basis

说明 durable identity registry / ingestion registry 中是否已有可能对应的交易身份。

#### Evidence issue basis

说明当前材料是否存在缺失、模糊、冲突、解析不完整、来源不清或附件无法配对等问题。

### Output Categories

Stage 2 只定义 conceptual output categories，不冻结 routing enum 或对象形状。

#### New permanent transaction identity

含义：当前材料被判断为一笔尚未登记的新交易，应建立稳定 `transaction_id`。

边界：
- 这是交易对象身份，不是业务身份。
- 它不说明 counterparty / vendor / payee 是谁。
- 它不说明会计分类、HST/GST 或 automation path。

#### Existing transaction identity reuse

含义：当前材料属于已登记交易，应复用既有 `transaction_id`。

边界：
- 复用 identity 是为了保持同一交易连续性。
- 它不等于重写原始 evidence。
- 它不等于批准 review outcome 或重新分类。
- 如果当前材料是补充 evidence，后续 evidence amendment / reprocessing / review trigger 的具体边界留到后续阶段。

#### Duplicate import result

含义：当前材料看起来是已登记交易的重复导入，不应产生新的交易对象。

边界：
- duplicate import result 防止重复处理和重复落账。
- 它不删除原始 evidence。
- 它不写 `Transaction Log`。
- 它不决定是否需要 accountant review；若重复信号暴露异常，由下游 review / issue handling 处理。

#### Same-transaction / duplicate candidate

含义：当前材料可能对应已存在交易，但 evidence 不足、冲突或来源关系不清，不能安全确认。

边界：
- candidate 不能被当作 confirmed identity。
- candidate 不能自动复用 `transaction_id`。
- candidate 不能自动丢弃当前材料。
- 下游是否 pending、review 或暂停处理留给后续 workflow 边界。

#### Identity issue signal

含义：本节点可以暴露身份层问题，例如疑似重复、同一交易冲突、来源不清、补交 evidence 无法归属、或 identity registry 与当前材料不一致。

边界：
- issue signal 不是 accountant question 本身。
- issue signal 不等于 governance event。
- issue signal 不等于 final audit record。

### Deterministic Code vs LLM Semantic Judgment

Stage 2 的核心边界是：

#### Deterministic code responsibility

Deterministic code 负责：
- 判断本节点是否被触发：evidence intake 已完成，且后续 workflow 需要交易身份。
- 基于客观交易结构、source context、evidence references 和 existing identity basis 判断新建、复用、重复或候选。
- 为新交易分配永久 `transaction_id`。
- 在 durable identity registry / ingestion registry 中建立或复用交易身份。
- 防止日期序号、展示文本或可变描述成为唯一身份 authority。
- 防止同一交易重复产生多个交易对象。
- 防止 duplicate candidate 被当作 confirmed duplicate。
- 防止 transaction identity 被下游误读为 entity、case、rule 或 accounting authority。
- 防止 `Transaction Log` 被用于 runtime dedupe decision。
- 防止本节点写入 Entity / Case / Rule / Governance 等长期业务记忆。

#### LLM semantic judgment responsibility

LLM 通常不应拥有交易身份决策 authority。

如果后续阶段允许 LLM 辅助，它最多可以在受限边界内帮助：
- 解释 messy source context 的人类可读含义。
- 总结为什么 evidence 看起来可能属于同一交易。
- 生成 identity issue explanation，供 pending / review 理解。

#### Hard boundary

LLM 不能：
- 分配 `transaction_id`
- 确认 same-transaction identity
- 确认 duplicate import
- 决定丢弃或覆盖 evidence
- 从 vendor 名称、描述相似或业务语义推断永久交易身份
- 把 identity candidate 升级为 confirmed identity
- 修改 durable identity registry
- 写入 `Transaction Log`
- 批准 entity、case、rule 或 governance change

### Accountant Authority Boundary

Accountant 仍然拥有最终 accounting decision 和 durable governance authority。

本节点不能：
- 替 accountant 判断交易用途
- 替 accountant 确认分类结果
- 替 accountant 决定某个冲突 evidence 的业务解释
- 把 accountant 尚未确认的说明写成最终客户政策
- 把重复导入问题转化为会计结论

### Governance Authority Boundary

Governance-level changes 不属于 Transaction Identity authority。

本节点不能：
- approve / reject alias
- confirm role
- create stable entity
- merge / split entity
- promote / modify / delete / downgrade active rule
- upgrade or relax automation policy
- approve governance event
- invalidate durable business memory

### Memory / Log Boundary

Stage 2 采用四层边界：read / consume、write allowed、candidate-only、no direct mutation。

#### Read / consume boundary

Transaction Identity 可以读取或消费以下 conceptual context：
- runtime evidence foundation
- objective normalized transaction basis
- evidence references
- source / import context
- evidence association result 或 duplicate-looking raw material candidate
- durable identity registry / ingestion registry 中的既有交易身份
- 当前 batch identity context

#### Write allowed boundary

Transaction Identity 可以在交易身份层执行有限 durable write：
- 为新交易登记永久 `transaction_id`。
- 记录交易身份与 evidence references / source context 的可追溯关联。
- 标记重复导入或 identity issue 的身份层结果。

#### Candidate-only boundary

Transaction Identity 只能作为候选或 issue signal 表达：
- same-transaction candidate
- duplicate import candidate
- supplemental evidence candidate for existing transaction
- identity conflict issue
- source ambiguity issue
- registry inconsistency issue

#### No direct mutation boundary

Transaction Identity 绝不能：
- 写入 `Transaction Log`
- 写入或修改 `Entity Log`
- 写入或修改 `Case Log`
- 写入或修改 `Rule Log`
- 写入或批准 `Governance Log`
- 修改 `Profile`
- 创建 stable entity
- 批准 alias、role、rule、automation policy 或 entity governance change
- 删除、覆盖或丢弃原始 evidence
- 把 duplicate candidate 当作 confirmed duplicate 处理
- 把 transaction identity result 直接变成 accountant decision

### Insufficient / Ambiguous / Conflicting Evidence Behavior

Stage 2 采用优先级边界：preserve identity stability first、do not duplicate second、do not over-merge third。

#### Preserve identity stability first

如果材料足以确认属于已登记交易，本节点应复用既有 `transaction_id`，保持交易身份连续。

#### Do not duplicate second

如果材料明显是重复导入，本节点应阻止新交易对象产生。

#### Do not over-merge third

如果材料可能属于同一交易，但 evidence 不足、来源冲突或关键客观事实不一致，本节点不能强行复用既有 `transaction_id`。

#### Conflict behavior

如果 existing identity basis 与当前 objective transaction basis 冲突，本节点应保守处理。

典型边界：
- 可确认重复：复用或标记 duplicate import。
- 可确认新交易：分配新永久身份。
- 无法确认：输出 candidate / issue signal。
- 冲突影响 downstream safety：交给 pending / review / identity issue handling，而不是自行猜测。

#### Hard boundary

- 相似 vendor 或相似描述不等于同一交易。
- 同一金额不等于同一交易。
- 同一 evidence candidate 不等于 confirmed same transaction。
- `transaction_id` 稳定性优先于导入便利。
- 去重不能靠会计语义猜测完成。
- `Transaction Log` 不参与 runtime identity decision。

## 3. Contract 字段摘要

### Contract Position in Workflow

#### Upstream handoff consumed

本节点消费 `Evidence Intake / Preprocessing Node` 产出的 runtime evidence foundation：
- `EvidenceFoundationHandoff`
- `ObjectiveNormalizedTransactionBasis`
- `EvidenceAssociationResult`
- `EvidenceQualityIssueSignal`
- raw transaction source / supporting evidence references
- source / import / batch context

#### Downstream handoff produced

本节点输出 `transaction_identity_handoff` 给：
- `Profile / Structural Match Node`：当当前交易拥有稳定 `transaction_id` 且没有被 duplicate / identity block 截断时。
- `Coordinator / Pending Node` / `Review Node`：当存在 same-transaction candidate、duplicate candidate、source ambiguity、registry conflict 或 unresolved identity issue 时。
- 后续 audit / memory workflow：仅通过 durable references 追溯 identity decision、evidence refs 和 source context。

#### Logs / memory stores read

本节点可以读取或消费：
- current runtime evidence foundation
- current batch identity context
- `Evidence Log` references / evidence metadata
- durable transaction identity / ingestion registry 中已存在的 identity records
- source / import context 和 objective association signals
本节点不得读取：
- `Transaction Log` 作为 runtime dedupe decision source
- `Entity Log`、`Case Log`、`Rule Log` 或 `Knowledge Log / Summary Log` 来判断是否同一笔交易
- entity / vendor / business semantics 来替代客观交易身份判断

#### Logs / memory stores written or candidate-only

本节点唯一允许的 durable write 属于 transaction identity / ingestion registry 语义：
- 新建永久 `transaction_id`
- 将既有 `transaction_id` 与新的 evidence / source context 建立可追溯关联
- 记录 duplicate import、identity candidate 或 identity issue 的 identity-layer outcome
本节点只能 candidate-only 输出：
- same-transaction candidate
- duplicate import candidate
- supplemental evidence candidate for existing transaction
- source ambiguity issue
- registry conflict issue
- identity review-needed signal

### Input Contracts

#### `transaction_identity_request`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `identity_request_id` | 本次 identity request 引用 | workflow runtime | runtime-only |
| `client_id` | 当前客户标识 | client / batch context | durable reference |
| `batch_id` | 当前处理批次 | workflow runtime | runtime-only |
| `evidence_handoff_ref` | 上游 `EvidenceFoundationHandoff` 引用 | Evidence Intake / Preprocessing | runtime ref with durable evidence refs |
| `objective_identity_basis` | 用于 identity 判断的客观交易 basis | Evidence Intake / Preprocessing | runtime object with durable evidence refs |
| `evidence_refs` | 支撑当前 identity 判断的 evidence references | `Evidence Log` / evidence foundation | durable references |
| `source_import_context` | source、batch、account、import attempt 等上下文 | importer / workflow runtime | mixed |
| `existing_identity_basis` | registry lookup 后可能相关的既有交易身份语境 | transaction identity / ingestion registry | durable refs |

**Optional fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `association_context` | 上游材料关联、候选关联或 duplicate-looking signal | Evidence Intake / Preprocessing | runtime / evidence context |
| `evidence_issue_context` | evidence quality、missing、conflict、source unclear 等问题 | Evidence Intake / Preprocessing | runtime / evidence context |
| `supplemental_context` | 当前材料可能是后补 evidence 的语境 | workflow / source context | runtime-only unless referenced |
| `request_trace` | runtime diagnostics | workflow runtime | runtime-only |

#### `objective_identity_basis`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `objective_basis_id` | 对应上游 normalized basis | Evidence Intake / Preprocessing | runtime / evidence context |
| `source_material_id` | 当前主要 raw transaction material | Evidence Log / source material | durable evidence reference |
| `date` | 交易日期；必须带 source reference | objective evidence extraction | objective fact with evidence ref |
| `amount_abs` | 金额绝对值 | objective evidence extraction | objective fact |
| `direction` | 资金方向 | objective evidence extraction | objective fact |
| `source_account_ref` | 来源银行/支付账户引用；不是 COA | source evidence / Profile account context when available | durable or evidence reference |
| `raw_description_ref` | 原始描述 reference；可引用空描述状态 | Evidence Log | durable evidence reference |

**Allowed values**

`direction` 允许：
- `inflow`
- `outflow`
- `unknown`

**Optional fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `posted_at` | posted date/time | source evidence | objective fact when present |
| `currency` | currency | source evidence | objective fact when present |
| `source_row_ref` | file、page、row、payload entry 等定位 | source evidence | durable evidence reference |
| `external_transaction_ref` | source system 提供的交易引用 | source system metadata | evidence/source metadata |
| `cheque_number_ref` | cheque number reference when present | evidence source | evidence reference |
| `counterparty_surface_text_refs` | 表面描述文本引用 | evidence signal only | durable evidence refs |
| `normalization_issue_ids` | 解析问题引用 | Evidence Intake / Preprocessing | runtime / evidence context |

#### `source_import_context`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `source_context_id` | source context 引用 | workflow runtime | runtime / source metadata |
| `source_channel` | 材料进入渠道 | importer / upload context | source metadata |
| `import_attempt_id` | 本次 import / upload / re-intake attempt | workflow runtime | runtime ref; may be retained |
| `received_at` | 系统接收时间 | workflow / importer | source metadata |
| `source_account_ref` | 来源账户或支付账户引用 | source material / client account context | evidence/profile reference |

**Allowed `source_channel` values**

- `bank_import`
- `payment_account_import`
- `supporting_upload`
- `supplemental_evidence`
- `historical_onboarding`
- `manual_reintake`
- `source_correction`
- `external_import_context`

**Optional fields**

- `source_system_name`
- `source_file_ref`
- `source_batch_ref`
- `prior_import_attempt_refs`
- `related_runtime_transaction_ref`
- `actor_type`; allowed values: `client`, `accountant`, `system_import`, `third_party_source`, `unknown`
- `source_context_note`

#### `association_context`

**Required fields when present**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `association_result_ids` | 上游 association result 引用 | Evidence Intake / Preprocessing | runtime / evidence refs |
| `association_statuses` | 上游关联状态列表 | Evidence Intake / Preprocessing | runtime / evidence context |
| `basis_refs` | 支撑关联状态的 evidence refs | Evidence Log | durable refs |

**Allowed `association_statuses` values**

- `confirmed_objective_association`
- `candidate_association`
- `unassociated`
- `conflicting_association`
- `duplicate_material_candidate`

**Optional fields**

- `candidate_strength`
- `conflict_issue_ids`
- `material_ref_ids`
- `association_reason`

#### `existing_identity_basis`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `lookup_status` | registry lookup 状态 | identity registry lookup | runtime |
| `candidate_identity_records` | 可能相关的既有 identity records | identity registry | durable refs |

**Allowed `lookup_status` values**

- `no_candidate_found`
- `single_candidate_found`
- `multiple_candidates_found`
- `registry_unavailable`
- `registry_inconsistent`
Allowed `identity_record_status` values：
- `active`
- `duplicate_import_recorded`
- `identity_issue_open`
- `superseded_by_registry_correction`
Optional fields：
- `prior_import_attempt_refs`
- `linked_duplicate_refs`
- `open_identity_issue_refs`
- `registry_correction_refs`
- `identity_decision_reason_refs`

#### `evidence_issue_context`

**Required fields when present**

- `issue_ids`
- `issue_types`
- `severity`
- `affected_evidence_refs`
- `summary`
Allowed `severity` values：
- `blocks_objective_handoff`
- `allows_handoff_with_issue`
- `informational`

### Output Contracts

#### `transaction_identity_handoff`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `identity_handoff_id` | 本次 handoff 引用 | Transaction Identity Node | runtime-only |
| `client_id` | 当前客户 | input context | durable reference |
| `batch_id` | 当前批次 | workflow runtime | runtime-only |
| `transaction_identity_status` | 当前 identity gate 状态 | Transaction Identity Node | runtime state |
| `identity_decision_type` | 具体 identity outcome 类型 | Transaction Identity Node | runtime state |
| `evidence_refs` | 支撑 identity outcome 的 evidence refs | Evidence Log / input handoff | durable refs |
| `identity_rationale` | 简短 identity-layer reason | Transaction Identity Node | runtime / registry context |

**Allowed `transaction_identity_status` values**

- `assigned`
- `duplicate_blocked`
- `candidate_only`
- `blocked`

**Allowed `identity_decision_type` values**

- `new_transaction_identity`
- `existing_transaction_identity_reuse`
- `duplicate_import_confirmed`
- `same_transaction_candidate`
- `duplicate_import_candidate`
- `supplemental_evidence_candidate`
- `identity_issue`
- `invalid_identity_input`

**Optional fields**

- `objective_identity_snapshot`
- `source_import_context_ref`
- `association_context_refs`
- `evidence_issue_refs`
- `downstream_allowed_next_nodes`
- `runtime_notes`
Allowed `downstream_allowed_next_nodes` values：
- `profile_structural_match`
- `coordinator_pending`
- `review`
- `identity_issue_handling`
- `stop_no_downstream_processing`

#### `transaction_identity_registry_record`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `transaction_id` | 永久交易 ID；新建时采用 `txn_<ULID>` 语义 | Transaction Identity Node | durable primary identity |
| `client_id` | 所属客户 | input context | durable reference |
| `identity_record_status` | identity record 状态 | Transaction Identity Node / registry | durable state |
| `registered_at` | 登记时间 | system time | durable metadata |
| `registered_from_request_id` | 来源 identity request | workflow runtime | durable trace ref |
| `objective_identity_basis_refs` | 支撑 identity 的 objective basis refs | Evidence Intake / Preprocessing | durable refs |
| `evidence_refs` | 绑定到该 identity 的 evidence refs | Evidence Log | durable refs |
| `source_import_context_refs` | source / import context refs | importer / workflow | durable refs |
| `identity_reason` | 简短 identity-layer reason | Transaction Identity Node | durable identity metadata |

**Allowed `identity_record_status` values**

- `active`
- `duplicate_import_recorded`
- `identity_issue_open`
- `superseded_by_registry_correction`

**Optional fields**

- `duplicate_of_transaction_id`
- `linked_duplicate_import_refs`
- `supplemental_evidence_refs`
- `open_identity_issue_refs`
- `prior_identity_record_refs`
- `registry_correction_refs`
- `objective_identity_snapshot`
- `source_fingerprint_refs`

#### `identity_registry_write_receipt`

**Required fields**

- `registry_write_receipt_id`
- `write_status`
- `transaction_id` when applicable
- `registry_record_ref`
- `written_at`
- `write_summary`
Allowed `write_status` values：
- `created_new_identity`
- `reused_existing_identity`
- `recorded_duplicate_import`
- `recorded_identity_issue`
- `write_blocked`
Optional fields：
- `blocked_reason`
- `candidate_identity_refs`
- `issue_signal_refs`

#### `identity_issue_signal`

**Required fields**

Allowed `issue_type` values：
- `same_transaction_candidate`
- `duplicate_import_candidate`
- `supplemental_evidence_candidate`
- `source_ambiguity`
- `objective_fact_conflict`
- `registry_inconsistency`
- `registry_unavailable`
- `overmerge_risk`
- `invalid_identity_input`
Allowed `severity` values：
- `blocks_downstream_processing`
- `requires_identity_review`
- `allows_downstream_with_identity_caution`
- `informational`
Optional fields：
- `candidate_identity_refs`
- `conflicting_values`
- `source_context_refs`
- `recommended_downstream_owner`; allowed values: `coordinator_pending`, `review`, `identity_issue_handling`, `workflow_orchestrator`
- `human_readable_explanation`

### Field Authority and Memory Boundary

#### Source of truth for important fields

- `transaction_id` 的 source of truth 是 Transaction Identity / ingestion registry。新建永久 ID 采用 `txn_<ULID>` 语义。
- `client_id` 来自 client / batch context。
- `date`、`amount_abs`、`direction`、`source_account_ref` 来自 Evidence Intake / Preprocessing 的 objective extraction 和 source evidence。
- `evidence_refs` 的 source of truth 是 `Evidence Log` / evidence preservation layer。
- `source_import_context` 的 authority 来自 importer、upload context、batch runtime 或 onboarding/runtime context。
- `existing_identity_basis` 的 authority 来自 durable identity registry lookup。
- `identity_decision_type` 和 `transaction_identity_status` 的 authority 只限于 transaction identity / dedupe layer。

#### Fields that can never become durable business memory by this node

本节点不能把以下字段或结论写成 Entity / Case / Rule / Governance / Profile memory：
- counterparty / vendor / payee entity conclusion
- approved alias
- confirmed role/context
- COA account
- HST/GST treatment
- classification confidence
- case judgment rationale
- deterministic rule result
- JE result
- accountant approval / correction
- automation policy change
- governance approval

#### Fields that can become durable only through accountant/governance approval

如果 identity issue 暴露了可能的长期业务记忆变化，下列变化只能通过后续 accountant / governance authority path：
- entity merge / split
- alias approval / rejection
- role confirmation
- rule promotion / modification / deletion / downgrade
- automation policy upgrade / relaxation
- profile structural fact confirmation
- case memory write based on completed final outcome

#### Audit vs learning/logging boundary

- `Transaction Log` 是 final audit-facing record，不参与本节点 runtime dedupe。
- identity registry 是 runtime identity / ingestion discipline 的 source of truth，不是 final audit log。
- identity registry 可以支持 audit traceability，但不能替代 Transaction Logging Node 的 final processing record。
- duplicate import handling 防止重复交易对象，不等于删除、丢弃或覆盖 raw evidence。
- identity candidate / issue 可以进入 review context，但不能作为 learning layer 或 rule promotion evidence。

### Validation Rules

#### Contract-level validation rules

- 每个 normal downstream transaction item 必须绑定 stable `transaction_id`，或者被明确标记为 `duplicate_blocked` / `candidate_only` / `blocked`。
- `amount_abs` 必须是正数；direction 必须显式表达。
- `evidence_refs` 必须非空且可追溯。
- `Transaction Log`、`Entity Log`、`Case Log`、`Rule Log`、`Governance Log`、`Knowledge Log / Summary Log` 不能作为 identity decision source。
- LLM output、natural-language summary、vendor similarity 或 business semantics 不能创建 confirmed transaction identity。
- candidate-only signal 不能被当作 confirmed identity decision。

#### Conditions that make the input invalid

Input invalid if：
- 缺少 `client_id`、`batch_id`、`objective_identity_basis`、`evidence_refs`、`source_import_context` 或 `existing_identity_basis`。
- 上游 evidence handoff 已经 `rejected_invalid_input`。
- date、amount、direction 或 source account 的 objective basis 缺失到无法支持 identity assignment，且 request 仍要求 normal assignment。
- `existing_identity_basis.lookup_status = registry_inconsistent` 但 request 要求 normal reuse / new assignment。
- request 声称 entity、COA、rule、case、JE 或 accountant conclusion 是 identity authority。
- `evidence_refs` 只是不可追溯摘要。

#### Conditions that make the output invalid

Output invalid if：
- `transaction_identity_status = assigned` 但缺少 exactly one `transaction_id`。
- `assigned` 同时声明 `duplicate_of_transaction_id`。
- `duplicate_blocked` 仍允许普通 downstream classification path。
- `candidate_only` 输出被写成 confirmed reuse、confirmed duplicate 或 final discard。
- output 写入或修改 Entity / Case / Rule / Governance / Profile / Transaction Log。
- registry record 保存 business classification、HST/GST、JE 或 final review result。
- `blocked` / `identity_issue` 没有 reason 或 issue refs。

#### Stop / ask conditions for unresolved contract authority

后续 Stage 或 implementation 如果遇到以下情况，不能自行填补为产品结论：
- 需要冻结 exact same-transaction / duplicate threshold。
- 需要决定 supplemental evidence 对既有 transaction 的 amendment、reprocessing 或 review trigger。
- 需要决定 registry correction / merge / split transaction identity 的 authority path。
- 需要决定 historical onboarding identity 与 runtime identity 的 reconciliation order。
- 需要让 candidate-only identity 继续进入 operational classification 的条件。
- 需要改变 `Transaction Log` 不参与 runtime decision 的边界。

## 4. Open Boundaries

### Stage 1: Open Boundaries

以下问题留到后续阶段，不在 Stage 1 冻结：
- durable identity registry / ingestion registry 的 exact contract。
- 什么证据足以确认“同一交易”，什么只能作为 duplicate candidate。
- 重复导入、新附件补交、修正导入、re-intake 和 reprocessing 的 precise workflow boundary。
- 已存在 `transaction_id` 的交易收到新 evidence 时，是 identity update、evidence amendment、review trigger，还是下游补充处理。
- historical onboarding transaction identity 与 runtime transaction identity 的衔接边界。
- 交易身份冲突或疑似重复时，下游是否继续处理，以及以什么限制继续处理。

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
- durable identity registry / ingestion registry 的 exact contract
- duplicate candidate 到 confirmed duplicate 的 exact authority boundary
- supplemental evidence 归属已存在交易后的 reprocessing / review trigger
- identity conflict 的人工处理 workflow
- historical onboarding identity 与 runtime identity registry 的合并或衔接规则
- identity registry write 与 Evidence Log write 的精确事务边界

### Stage 3: Open Contract Boundaries

- Exact same-transaction / duplicate threshold 未冻结：本 Stage 只定义 contract states，不定义匹配算法、score 或阈值。
- Evidence amendment timing 未冻结：既有 `transaction_id` 收到 supplemental evidence 时，是 identity association、evidence amendment、review trigger 还是 reprocessing trigger，仍需后续阶段定义。
- Identity registry storage shape 未冻结：本文件定义 logical record，不定义 DB、index、migration 或 file layout。
- Historical onboarding identity 与 runtime identity 的 reconciliation order 未冻结。
- Registry correction / transaction identity merge-split authority path 未完整定义；当前只要求不能由本节点用 candidate signal 自动修正。
- `candidate_only` 是否允许某些受限下游继续处理未冻结；当前 contract 只要求不得当作 confirmed identity 或 duplicate。
- Evidence Log 写入时点与 identity registry 绑定时点未冻结；当前 contract 只要求所有 identity output 必须保留可追溯 evidence refs。
