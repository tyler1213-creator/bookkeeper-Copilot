# Evidence Intake / Preprocessing Node — 设计摘要

> 由原 Stage 1/2/3 合并瘦身而来；保留定位、边界、contract 字段和开放问题，删除阶段说明、examples、self-review、历史读档记录和重复解释。

## 1. 定位与职责

Stage 1 已确定以下内容：
- `Evidence Intake / Preprocessing Node` 是主 workflow 的 runtime evidence 入口节点。
- 它的核心职责是保留可追溯 evidence，并统一客观结构。
- 它可以建立 evidence foundation，但不能保存业务结论。
- 它不分配 `transaction_id`，不处理最终去重决策。
- 它不做 profile / structural match、entity resolution、rule match 或 case judgment。
- 它不做会计分类、HST/GST 判断或 journal entry generation。
- 它不写 `Transaction Log`。
- 它不修改 Entity / Case / Rule / Governance 等长期业务记忆。
- 它不把 transient handoff、临时配对候选、解析草稿或 review draft 称为 `Log`。

## 2. 逻辑与边界

### Trigger Boundary

`Evidence Intake / Preprocessing Node` 在主 workflow 接收新交易材料时触发。

概念触发条件是：
- 有新的 runtime transaction materials 进入系统；并且
- 这些材料需要被整理成可追溯 evidence foundation；并且
- 后续 `Transaction Identity Node`、结构性匹配、entity resolution、rule match 或 case judgment 尚不能安全消费未经整理的原始材料。
典型触发材料包括：
- 银行或支付账户导入材料
- 小票、支票、合同、发票或其他交易附件
- accountant 或 client 在运行中补交的当前交易证据
- 与当前 batch 或当前交易相关的原始说明

### Input Categories

Stage 2 按判断作用组织 input categories，不列字段清单。

#### Raw transaction source basis

说明新交易来自哪些原始交易来源，以及哪些内容是 source-provided fact。

#### Supporting evidence basis

说明当前交易是否有小票、支票、合同、发票、client note、accountant note 或其他支持性证据。

#### Pairing / association basis

说明多份原始材料之间是否可能属于同一笔交易或同一组交易证据。

#### Objective structure basis

说明原始材料中哪些客观交易事实可以被稳定解析和统一。

#### Evidence quality / completeness basis

说明当前 evidence 是否存在缺失、不可读、格式异常、来源不清、附件无法配对、内容互相冲突或解析不完整。

#### Existing runtime context basis

说明当前 batch 或当前客户是否已有可用的 intake context，例如已导入材料、已保存 evidence reference、或上游外部导入状态。

### Output Categories

Stage 2 只定义 conceptual output categories，不冻结 routing enum 或对象形状。

#### Runtime evidence foundation

含义：新交易原始材料被整理为后续节点可引用的 evidence foundation。

边界：
- foundation 保留原始材料和证据引用。
- foundation 不保存业务结论。
- foundation 不等于 identity、entity、case、rule 或 accounting authority。

#### Objective normalized transaction basis

含义：可稳定解析的客观交易结构被统一为后续 `Transaction Identity Node` 和结构性匹配可消费的基础。

边界：
- 只表达客观事实。
- 不冻结最终 schema。
- 不产生交易永久身份。
- 不产生会计分类。

#### Evidence association result

含义：本节点可以说明哪些原始材料已被客观关联，哪些只是候选关联，哪些无法安全关联。

边界：
- 关联结果帮助后续 identity、review 和 audit。
- 候选关联不能被当作已确认同一交易。
- 最终交易身份和去重不属于本节点。

#### Evidence quality / issue signals

含义：本节点可以暴露 evidence 缺失、冲突、解析失败、来源不清、附件不匹配或需要人工补充的信号。

边界：
- issue signal 不是 accountant question 本身。
- issue signal 不等于 pending routing 的最终决定。
- 下游 Coordinator / Pending Node 或 Review Node 负责把它转化为人工交互或审核动作。

#### Downstream handoff context

含义：本节点为后续 workflow nodes 提供同一组 evidence references、objective basis、quality context 和 association context。

边界：
- handoff 是运行时交接，不是 durable business memory。
- 不能把 handoff、临时队列、候选配对或草稿称为 `Log`，除非它们确实进入 durable log/store。

### Deterministic Code vs LLM Semantic Judgment

Stage 2 的核心边界是：

#### Deterministic code responsibility

Deterministic code 负责：
- 判断材料属于 runtime evidence intake，而不是 onboarding initialization、transaction identity、review 或 governance。
- 解析可客观确定的交易结构。
- 保留原始 evidence 与可追溯 references。
- 标记 evidence 来源类别和 intake 状态。
- 执行格式、完整性、一致性和可引用性检查。
- 区分已客观关联的 evidence、候选关联 evidence、无法关联 evidence。
- 防止本节点生成 transaction identity、entity identity、rule result、case judgment 或 accounting conclusion。
- 防止 `Transaction Log` 被误用为 runtime evidence source 或 learning layer。
- 防止 transient handoff、candidate association、临时解析结果或 report draft 被命名为 durable `Log`。

#### LLM semantic judgment responsibility

LLM 可以辅助：
- 从非结构化或半结构化材料中读取 human-visible text。
- 解释小票、支票、合同、invoice 或 note 中的自然语言内容。
- 帮助识别材料之间可能有关联的线索。
- 生成 evidence quality / missing-context explanation。
- 为下游 pending / review 提供可读的 evidence issue summary。

#### Hard boundary

LLM 不能：
- 把 messy evidence 解释成最终会计用途
- 判断稳定 entity
- 确认 alias 或 role
- 生成 deterministic rule result
- 判断 case precedent 是否足以分类
- 分配 transaction identity
- 写入 Transaction Log
- 修改 Entity / Case / Rule / Governance memory
- 把候选关联当作已确认事实
- 因为材料“看起来合理”而隐藏证据缺失或冲突

### Accountant Authority Boundary

Accountant 仍然拥有最终 accounting decision 和 durable authority。

`Evidence Intake / Preprocessing Node` 不能：
- 替 accountant 判断业务用途
- 替 accountant 确认缺失或冲突证据的真实含义
- 把 accountant 尚未确认的说明写成最终客户政策
- 把 client-provided context 直接升级成 accountant-approved conclusion
- 批准当前交易的 review outcome

### Governance Authority Boundary

Governance-level changes 不属于 Evidence Intake authority。

本节点不能：
- approve / reject alias
- confirm role
- create stable entity
- merge / split entity
- promote / modify / delete / downgrade active rule
- upgrade or relax automation policy
- approve governance event
- invalidate durable memory

### Memory / Log Boundary

Stage 2 采用四层边界：read / consume、write allowed、candidate-only、no direct mutation。

#### Read / consume boundary

Evidence Intake 可以读取或消费以下 conceptual context：
- 当前 runtime raw materials
- 当前 batch 的 intake context
- 已存在的 evidence references，用于避免重复保存或保留来源关系
- 外部导入状态或材料来源说明

#### Write allowed boundary

Evidence Intake 可以建立或追加 evidence foundation，但只能在 evidence 层职责内：
- 保存原始 evidence 或 evidence references
- 保存客观 intake / normalization context
- 保存 evidence quality、source clarity、association status 或 parsing issue context

#### Candidate-only boundary

Evidence Intake 只能作为候选或 issue signal 表达：
- evidence association candidate
- unmatched supporting evidence candidate
- duplicate-looking raw material candidate
- missing evidence issue
- conflicting evidence issue
- low-quality / unreadable evidence issue

#### No direct mutation boundary

Evidence Intake 绝不能：
- 写入 `Transaction Log`
- 写入或修改 `Entity Log`
- 写入或修改 `Case Log`
- 写入或修改 `Rule Log`
- 写入或批准 `Governance Log`
- 修改 `Profile`
- 创建 transaction identity
- 创建 stable entity
- 批准 alias、role、rule、automation policy 或 entity governance change
- 把 evidence issue signal 直接变成 accountant decision

### Insufficient / Ambiguous / Conflicting Evidence Behavior

Stage 2 采用优先级边界：preserve first、separate objective from uncertain second、surface issue third。

#### Preserve first

如果材料不完整、难读、格式异常或无法完全解析，本节点仍应尽可能保留原始 evidence 和来源关系。

#### Separate objective from uncertain second

如果一部分客观结构可确定，另一部分不确定，本节点应把可确定部分和不确定部分分开表达。

#### Surface issue third

如果 evidence 缺失、模糊或冲突，本节点应把问题显露给下游，而不是自行解决业务含义。

典型行为边界：
- 缺少支持性证据：输出 missing evidence issue。
- 附件无法配对：输出 unmatched / candidate association issue。
- 原始材料互相矛盾：输出 conflicting evidence issue。
- 材料来源不清：输出 source clarity issue。
- 解析失败或低质量：输出 parsing / quality issue。

#### Hard boundary

- Evidence ambiguity 不等于可以猜。
- Evidence conflict 不等于可以选择一个看起来更合理的版本。
- OCR / parser / LLM extraction 结果不能覆盖原始 evidence。
- 客观 intake issue 不能被静默转化为 business conclusion。
- 本节点应保留证据问题，而不是隐藏证据问题。

## 3. Contract 字段摘要

### Contract Position in Workflow

#### Upstream handoff consumed

本节点消费主 workflow 入口侧的新交易材料：
- bank / payment account import line
- receipt、cheque、invoice、contract 等 supporting evidence
- client / accountant 对当前交易或当前 batch 的原始说明
- 当前 batch 内已存在的 intake context 或 evidence reference

#### Downstream handoff produced

本节点输出给：
- `Transaction Identity Node`：objective transaction basis、evidence refs、association / duplicate-looking signals
- `Profile / Structural Match Node`：客观交易字段与 evidence refs
- `Entity Resolution Node`：raw counterparty / vendor / payee signals、supporting evidence refs、quality context
- `Coordinator / Pending Node` / `Review Node`：missing、conflicting、unmatched、low-quality evidence issue signals
- 后续 audit / memory workflow：可回溯 evidence references

#### Logs / memory stores read

本节点可以读取或消费：
- 当前 runtime raw materials
- 当前 batch intake context
- 已存在的 `Evidence Log` references，用于避免重复保存或保留 source relation
- 外部导入状态或材料来源说明

#### Logs / memory stores written or candidate-only

本节点可以写入或请求追加：
- `Evidence Log` record / evidence reference
- objective intake / normalization context
- evidence quality、source clarity、association status、parsing issue context
本节点只能 candidate-only 输出：
- evidence association candidate
- unmatched supporting evidence candidate
- duplicate-looking raw material candidate
- missing / conflicting / low-quality evidence issue
本节点绝不能写入或修改：
- `Transaction Log`
- `Entity Log`
- `Case Log`
- `Rule Log`
- `Governance Log`
- `Profile`

### Input Contracts

#### `RuntimeMaterialIntakeEnvelope`

Allowed `source_channel` values：
- `bank_import`
- `payment_account_import`
- `supporting_upload`
- `client_note`
- `accountant_note`
- `supplemental_evidence`
- `external_import_context`
Validation / rejection rules：
- `intake_batch_id`、`client_id`、`received_at`、`source_channel`、`material_refs` 必须存在。
- `source_channel` 必须属于允许值。
- `material_refs` 不能为空。
- envelope 不能声称 `transaction_id`、`entity_id`、`rule_id`、COA、HST/GST 或 JE 结论。

#### `RawTransactionSourceMaterial`

Allowed `source_type` values：
- `bank_statement_line`
- `payment_account_export_line`
- `processor_feed_line`
- `manual_transaction_source`
- `other_transaction_source`
Validation / rejection rules：
- `raw_evidence_pointer` 必须足以回到原始材料。
- `raw_amount_text` 和 `raw_date_text` 缺失时，材料仍可保存为 evidence，但不能产出 complete `ObjectiveNormalizedTransactionBasis`。
- `source_account_ref` 不得被解释为 COA account。
- `counterparty_surface_text` 不得被解释为稳定 `Entity`、approved alias 或 role。

#### `SupportingEvidenceMaterial`

Allowed `evidence_type` values：
- `receipt`
- `cheque`
- `invoice`
- `contract`
- `client_note`
- `accountant_note`
- `historical_attachment`
- `other_supporting_material`
Allowed `source_actor_type` values：
- `client`
- `accountant`
- `system_import`
- `third_party_source`
- `unknown`
Validation / rejection rules：
- `raw_evidence_pointer` 必须存在。
- `source_actor_type = accountant` 不等于 accountant approval；除非下游 Review / Governance flow 明确批准，本节点只能保存原话。
- `captured_text`、`visible_vendor_text`、`visible_date_text`、`visible_amount_text` 都是 extracted evidence signals，不能覆盖原始材料。

#### `AssociationInputHint`

Allowed `hint_type` values：
- `same_upload_group`
- `same_source_file`
- `client_claimed_related`
- `accountant_claimed_related`
- `filename_or_folder_hint`
- `source_system_link`
- `possible_duplicate_material`
Validation / rejection rules：
- hint 不能直接变成 confirmed association。
- `accountant_claimed_related` 表示 accountant 原话存在，不表示 review approval 已完成。
- `possible_duplicate_material` 只能交给下游 identity / review 评估，不能由本节点做 final dedupe。

#### `ExistingEvidenceReferenceContext`

Allowed `preservation_status` values：
- `preserved`
- `reference_only`
- `superseded_by_clearer_copy`
- `rejected_untraceable`
Validation / rejection rules：
- 只能用于 evidence preservation / reference discipline。
- 不能从既有 evidence reference 推断业务结论。
- 不能读取 `Transaction Log` 来补足 runtime decision。

### Output Contracts

#### `EvidenceLogRecord`

Allowed `evidence_type` values：
- `bank_statement_line`
- `payment_account_export_line`
- `processor_feed_line`
- `manual_transaction_source`
- `receipt`
- `cheque`
- `invoice`
- `contract`
- `client_note`
- `accountant_note`
- `historical_attachment`
- `other_supporting_material`
- `other_transaction_source`
Allowed `source_actor_type` values：
- `client`
- `accountant`
- `system_import`
- `third_party_source`
- `unknown`
Allowed `preservation_status` values：
- `preserved`
- `reference_only`
- `superseded_by_clearer_copy`
- `rejected_untraceable`
Validation rules：
- 必须能回到 raw evidence；只有模型摘要而无 `raw_evidence_pointer` 的输出无效。
- 不得包含 `transaction_id` assignment、`entity_id` conclusion、COA、HST/GST、JE、rule result 或 case judgment。
- `rejected_untraceable` 可以记录 rejection metadata，但不能作为后续自动化 evidence basis。

#### `ObjectiveNormalizedTransactionBasis`

Allowed `parse_status` values：
- `parsed`
- `partially_parsed`
- `unparseable`
Allowed `direction_basis.value` values when known：
- `money_in`
- `money_out`
- `unknown`
Validation rules：
- `parsed` 必须至少有 `date_basis`、`amount_absolute_basis`、`direction_basis`、`source_account_basis`。
- `partially_parsed` 必须显式说明缺失或冲突的 basis。
- `unparseable` 不得伪造客观字段；只能携带 source reference 和 issue。
- 本节点不得在此对象中分配 `transaction_id`。
- `amount_absolute_basis` 不得携带方向符号；direction 必须由 `direction_basis` 表达。

#### `EvidenceAssociationResult`

Allowed `association_status` values：
- `confirmed_objective_association`
- `candidate_association`
- `unassociated`
- `conflicting_association`
- `duplicate_material_candidate`
Allowed `candidate_strength` values when present：
- `strong`
- `weak`
- `unknown`
Validation rules：
- `confirmed_objective_association` 只能表示材料间客观配对成立，不能表示 same transaction identity 已确认。
- `candidate_association`、`duplicate_material_candidate` 不能支持 rule match、case promotion 或 durable business memory。
- `conflicting_association` 必须引用具体冲突来源。

#### `EvidenceQualityIssueSignal`

Allowed `issue_type` values：
- `missing_core_transaction_fact`
- `unreadable_material`
- `unsupported_format`
- `source_unclear`
- `parsing_failed`
- `partial_parse`
- `evidence_conflict`
- `unmatched_supporting_evidence`
- `candidate_association_only`
- `duplicate_material_candidate`
Allowed `severity` values：
- `blocks_objective_handoff`
- `allows_handoff_with_issue`
- `informational`
Validation rules：
- issue signal 不得自行生成 accountant-facing question。
- issue signal 不得决定 PENDING、review_required 或 classification path。
- `evidence_conflict` 必须列出冲突来源，不能只写“conflict”。

#### `EvidenceFoundationHandoff`

Allowed `handoff_status` values：
- `ready`
- `ready_with_issues`
- `rejected_invalid_input`
Validation rules：
- `ready` 至少需要一个 usable `ObjectiveNormalizedTransactionBasis`。
- `ready_with_issues` 必须引用至少一个 `EvidenceQualityIssueSignal` 或 non-confirmed `EvidenceAssociationResult`。
- `rejected_invalid_input` 必须说明 rejection issue，且不得输出伪造 objective basis。
- handoff 不得包含 final routing、classification、entity conclusion、rule result 或 JE。

### Field Authority and Memory Boundary

#### Source of truth for important fields

- Raw material content 的 source of truth 是原始 file、row、payload、image、note 或 external source record。
- `raw_evidence_pointer` / `source_location` 的 authority 来自 importer、upload context 或 evidence preservation layer。
- `date_basis`、`amount_absolute_basis`、`direction_basis`、`currency_basis` 的 authority 来自原始材料加本节点 objective extraction；它们不是业务判断。
- `counterparty_surface_text_refs`、`visible_vendor_text`、`visible_amount_text` 是 evidence signals，不是 stable entity、alias approval 或 role confirmation。
- `association_status` 的 authority 只限于 evidence association discipline，不延伸为 transaction identity / dedupe authority。
- `quality_issue` 的 authority 只说明 evidence condition，不说明 accounting treatment。

#### Fields that can never become durable memory by this node

本节点不能把以下字段或结论写成 durable business memory：
- `transaction_id` assignment
- stable `entity_id`
- approved alias
- confirmed role/context
- COA account
- HST/GST treatment
- journal entry
- deterministic rule result
- case judgment / precedent applicability
- accountant approval
- governance approval
- automation policy change

#### Fields that can become durable only after accountant / governance approval

以下内容即使从本节点材料中出现，也只能作为 evidence 或 candidate signal 传递；只有经过下游 accountant / governance authority path 后，才可能成为长期 authority：
- client/accountant note 中包含的客户结构事实
- alias、entity、role/context 候选
- rule candidate 或 automation policy candidate
- 对 evidence conflict 的业务解释
- 对 supporting evidence 与交易用途关系的确认

#### Audit vs learning / logging boundary

- `Evidence Log` 保存原始证据和证据引用，不保存业务结论。
- `Transaction Log` 保存最终处理结果和审计轨迹；本节点不写，也不把它作为 runtime decision source。
- Evidence quality / association issue 可以成为 audit context，但不能成为 learning layer 的 business conclusion。
- Candidate association、duplicate-looking signal、unmatched evidence 不是 durable learning。

### Validation Rules

#### Contract-level validation rules

- 所有 output 必须能追溯到 input material 或 existing evidence reference。
- 所有 extracted fields 必须保留 source reference；不能只有模型总结。
- Objective facts、evidence signals、candidate signals、business authority 必须分层表达。
- `Log` 只能指 durable long-term meaningful information storage；runtime handoff、candidate queue、temporary parse result 不能命名为 `Log`。
- `Transaction Log` 不参与本节点 runtime decision。

#### Conditions that make input invalid

Input invalid if：
- 没有 `client_id`。
- 没有任何可追溯 raw material reference。
- `source_channel`、`source_type`、`evidence_type` 或 `source_actor_type` 不属于允许值。
- 材料只提供摘要，没有 raw evidence pointer。
- 输入把 COA、HST/GST、entity、rule、case 或 transaction identity 当作本节点应接受的 asserted fact。
- accountant/client note 缺少 source actor metadata，却要求作为 authority 使用。

#### Conditions that make output invalid

Output invalid if：
- output 无法追溯到 raw evidence。
- output 用 OCR/parser/LLM extraction 覆盖原始 evidence。
- output 分配 `transaction_id`。
- output 生成 stable entity、approved alias、confirmed role、rule result、case judgment、COA、HST/GST 或 JE。
- candidate association 被写成 confirmed transaction identity。
- issue signal 被写成 accountant decision、PENDING decision 或 final routing。
- handoff 读取或引用 `Transaction Log` 作为 runtime decision authority。

#### Stop / ask conditions for unresolved contract authority

后续 Stage 如果遇到以下情况，应 stop / ask，而不是自行冻结：
- 需要决定 Evidence Log 写入是在 identity 前、identity 后，还是分为 pre-identity reference 与 post-identity attachment。
- 需要决定 supplemental evidence 对既有 transaction 的 re-intake / amendment 覆盖规则。
- 需要把 evidence association candidate 升级为 confirmed transaction association 的 authority。
- 需要让本节点读取 `Profile`、`Entity Log`、`Case Log`、`Rule Log` 或 `Transaction Log` 参与判断。
- 需要改变本节点不写 `Transaction Log` / business memory 的边界。

## 4. Open Boundaries

### Stage 1: Open Boundaries

以下问题留到后续阶段，不在 Stage 1 冻结：
- 原始 evidence 在本节点内何时被持久化，何时只作为待确认 intake result 传递。
- 新交易 evidence 与历史 onboarding evidence 的 exact linkage boundary。
- 支票、小票、合同或其他附件与交易的配对结果，哪些可以由本节点确定，哪些只能作为候选。
- 解析失败、部分解析、重复材料和补交材料的 precise workflow boundary。
- Evidence quality、conflict 和 missing evidence 由本节点表达到什么精度。
- 下游节点如何引用本节点产生的 evidence foundation。

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
- evidence write 与 runtime handoff 的精确时机
- re-intake / supplemental evidence 的触发和覆盖规则
- evidence association candidate 到 confirmed association 的 exact authority boundary
- duplicate-looking raw material 与 transaction identity dedupe 的 exact handoff contract

### Stage 3: Open Contract Boundaries

- Evidence Log 写入的 exact timing 未冻结：当前 contract 定义 logical `EvidenceLogRecord` 与 handoff reference，不决定是 pre-identity 立即持久化、identity 后绑定，还是两阶段引用。
- Supplemental evidence 的 re-intake / amendment workflow 未冻结：本节点可以表达 `supplemental_to_runtime_ref`，但不能决定覆盖既有 transaction 或重写 downstream outcome。
- `confirmed_objective_association` 的精确 authority threshold 未冻结：本 Stage 只要求它不得等同 transaction identity / dedupe。
- Evidence issue 如何转化为 accountant-facing question，属于 `Coordinator / Pending Node` 或 `Review Node` 后续 contract。
- `content_fingerprint_ref` 只作为 contract-level duplicate-material reference，不定义 hash / storage implementation。
