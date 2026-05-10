# Onboarding Node — 设计摘要

> 由原 Stage 1/2/3 合并瘦身而来；保留定位、边界、contract 字段和开放问题，删除阶段说明、examples、self-review、历史读档记录和重复解释。

## 1. 定位与职责

Stage 1 已确定以下内容：
- `Onboarding Node` 是初始化节点，不是主 workflow 的常规 runtime 分类节点。
- 它处理历史材料，建立初始 evidence、profile 候选、entity、historical case memory 和 rule / governance 候选。
- 它的核心方向是 evidence-first：先保留证据，再建立 entity 和 case，最后才筛选 rule 候选或初始 rule 路径。
- 它可以使用 accountant 已处理过的历史账本作为高 authority 来源，但必须保留清楚的 source / authority 语境。
- 它不能把缺少 authority 的推断直接写成稳定客户事实。
- 它不能绕过 accountant authority 或 governance approval。
- 它不生成 journal entries。
- 它不写新交易的 `Transaction Log`。
- 它不把 transient handoff、候选队列或草稿称为 `Log`，除非该信息确实进入 durable storage。

## 2. 逻辑与边界

### Trigger Boundary

`Onboarding Node` 在新客户初始化时触发。

它的 conceptual trigger 是：
- 有一批客户历史材料需要进入新系统；并且
- 这些材料需要被转化为 evidence-first 的初始记忆基础；并且
- 主 workflow 尚不能安全依赖空白 memory 直接运行，或需要先获得初始化 context。
典型触发材料包括：
- 历史银行流水
- accountant 已处理过的历史账本
- 小票、支票、合同或其他交易证据
- accountant 对客户结构、业务背景或特殊处理的说明

### Input Categories

Stage 2 按判断作用组织 input categories，不列字段清单。

#### Historical evidence basis

说明历史材料中有哪些原始证据可以保留和引用。

#### Accountant-processed historical basis

说明哪些历史交易已经被 accountant 或 accountant-prepared books 处理过。

#### Customer structural basis

说明历史材料是否暗示客户结构事实，例如银行账户、内部转账关系、贷款、员工、tax config、owner 使用公司账户等。

#### Entity basis

说明历史材料中出现了哪些 counterparty、vendor、payee、alias、简称或表面写法。

#### Case precedent basis

说明历史交易如何被处理，以及处理条件、证据和例外是什么。

#### Rule stability basis

说明是否存在少量稳定对象，历史分类长期一致，例外边界清楚，且适合进入 initial rule review 或 approval path。

#### Risk / conflict basis

说明初始化材料中是否存在混用风险、身份歧义、账本和证据不一致、分类例外、近期人工修正、role 未确认或治理风险。

### Output Categories

Stage 2 只定义 conceptual output categories，不冻结 routing enum 或对象形状。

#### Initial evidence foundation

含义：历史原始材料被保留为可追溯 evidence，并可被后续 entity、case、review 和 audit 语境引用。

边界：
- evidence 保存原始材料和引用。
- evidence 不保存业务结论。
- evidence 不等于 classification authority。

#### Profile candidates

含义：从历史材料中识别出的客户结构事实候选。

边界：
- candidate 可以帮助 Review / accountant 聚焦确认。
- candidate 不能在缺少 authority 时变成稳定 profile truth。
- profile truth 的确认边界留给 accountant authority 或后续 governance/review 流程。

#### Initial entity foundation

含义：历史材料中的 counterparty / vendor / payee 被会合为初始 entity context，或被保留为 entity candidates / alias candidates / role candidates。

边界：
- 稳定 entity 初始化必须有足够 evidence 和 source authority。
- candidate alias 可以作为后续语境，但不能支持 rule match。
- role 如果不是 accountant-confirmed 或明确 accountant-derived，不能写成稳定 role。
- entity merge / split 只能作为 governance candidate，不能由 Onboarding 无审批执行。

#### Historical case foundation

含义：accountant 已处理过的历史交易可以成为 historical case memory，使后续 Case Judgment 读取真实案例、分类结果、证据和例外。

边界：
- historical case 依赖可追溯 accountant-processed basis。
- 没有可靠 authority 的历史推断不能成为已确认 case。
- case memory 是案例和条件，不是统计压缩，也不是 deterministic rule。

#### Customer knowledge summary seed

含义：从 entity、case 和风险语境中生成客户专属知识摘要，帮助后续 humans 和 agents 理解常见模式、例外和风险。

边界：
- summary 是可读语境。
- summary 不是 deterministic rule source。
- summary 不能替代 raw evidence、case memory 或 accountant approval。

#### Rule / governance candidates

含义：少量稳定对象可能被提出为 initial rule candidate、automation policy candidate、alias / role governance candidate 或 entity governance candidate。

边界：
- candidate 只表示后续 review/governance 应评估。
- candidate 不改变 active rule。
- candidate 不放宽 automation policy。
- candidate 不批准 alias、role、entity merge/split 或 rule promotion。

### Deterministic Code vs LLM Semantic Judgment

Stage 2 的核心边界是：

#### Deterministic code responsibility

Deterministic code 负责：
- 判断输入属于 onboarding / historical initialization，而不是 runtime transaction processing。
- 对客观结构信息做标准化。
- 为历史交易建立稳定身份并处理去重。
- 保留原始 evidence 和 evidence references。
- 标记材料来源与 authority category。
- 区分 evidence、candidate、accountant-derived historical case、rule candidate 和 governance candidate。
- 执行已知 authority limits：没有 accountant authority 的 profile、role、rule 或 governance 变化不能被写成最终 authority。
- 防止 `Transaction Log` 被误用为 onboarding 学习层。
- 防止 transient handoff、候选队列、报告草稿被命名或处理成 durable `Log`。

#### LLM semantic judgment responsibility

LLM 可以辅助：
- 从历史材料中识别可能的 vendor、payee、counterparty、简称和 alias。
- 判断多个表面写法是否可能指向同一 entity。
- 从 accountant-prepared historical books 中抽取 historical case meaning。
- 总结客户专属知识、常见处理方式、例外条件和风险提示。
- 提出 profile、entity、case、rule 或 governance candidates 的解释。

#### Hard boundary

LLM 不能：
- 把 raw evidence 直接升级为 accountant-confirmed conclusion
- 把 candidate profile fact 写成稳定 profile truth
- 把 candidate alias 支持 rule match
- 把 candidate role 写成稳定 role
- 把 rule candidate 变成 active rule
- 放宽 automation policy
- 批准 entity merge / split
- 用 summary 替代 evidence、case 或 approval
- 为缺少 accountant authority 的历史材料发明会计结论

### Accountant Authority Boundary

Accountant 仍然拥有最终 accounting decision 和 durable authority。

Onboarding 不能：
- 替 accountant 批准未确认的 profile fact
- 替 accountant 确认 role / alias / entity governance change
- 把不稳定历史模式升级成 active deterministic rule
- 把历史材料中的歧义解释成最终客户政策
- 把缺证据或冲突材料写成稳定客户知识

### Governance Authority Boundary

Governance-level changes 不属于 Onboarding 的最终批准 authority。

Onboarding 可以提出治理候选，例如：
- alias approval candidate
- role confirmation candidate
- entity merge / split candidate
- automation policy candidate
- rule promotion / initial rule candidate
Onboarding 不能直接批准：
- entity merge / split
- alias approval / rejection
- role confirmation
- active rule promotion / modification / deletion / downgrade
- automation policy upgrade or relaxation
- governance event approval

### Memory / Log Boundary

Stage 2 采用四层边界：read / consume、write allowed、candidate-only、no direct mutation。

#### Read / consume boundary

Onboarding 可以读取或消费以下 conceptual context：
- 历史 raw evidence：银行流水、小票、支票、历史账本文本、accountant 原话。
- accountant-processed historical records：已由 accountant 处理过的历史分类材料。
- existing customer context：如果存在迁移资料或客户说明，可以作为初始化参考。
- existing governance / intervention context：如果初始化发生在迁移或补材料场景，可作为风险语境；该场景是否常规支持仍是 Open Boundary。

#### Write allowed boundary

Onboarding 可以建立以下初始化基础，但必须保留 authority/source 语境：
- durable evidence foundation
- accountant-derived historical case foundation
- sufficiently supported initial entity foundation
- customer knowledge summary seed

#### Candidate-only boundary

Onboarding 只能作为候选提出：
- profile facts that require confirmation
- entity / alias / role changes that require governance
- rule candidates or initial rule approval candidates
- automation policy changes
- entity merge / split candidates
- unresolved conflict notes requiring accountant review

#### No direct mutation boundary

Onboarding 绝不能：
- 写入新交易的 `Transaction Log`
- 把候选 rule 写成 active `Rule Log` authority
- 把 candidate alias 当作 approved alias 使用
- 把 candidate role 写入稳定 entity role
- 执行 entity merge / split
- 放宽 automation policy
- 把 customer knowledge summary 当作 deterministic rule source
- 把不可靠历史材料沉淀为 confirmed case memory

### Insufficient / Ambiguous / Conflicting Evidence Behavior

Stage 2 采用优先级边界：source authority first、evidence traceability second、conflict caution third。

#### Source authority first

如果历史分类来自 accountant-prepared books 或明确 accountant instruction，它可以支持 historical case foundation。

#### Evidence traceability second

任何初始化结论都必须能回到原始材料或 accountant source。

#### Conflict caution third

如果历史账本、receipt、bank description、cheque information 或 accountant notes 互相冲突，Onboarding 不应自行选择最顺眼的版本作为稳定事实。

冲突行为边界：
- 可以保留冲突证据。
- 可以提出 review / governance candidate。
- 可以生成面向 accountant 的冲突说明。
- 不能把冲突隐藏在 summary 中。
- 不能把冲突材料升级为 deterministic rule。

#### Ambiguity handling

如果多个 entity、alias、role 或客户结构解释都可能成立，Onboarding 应保持候选或歧义状态。

## 3. Contract 字段摘要

### Contract Position in Workflow

`Onboarding Node` 位于新客户主 workflow 之前。

#### Upstream handoff consumed

本节点消费 `onboarding_initialization_request`。

该 request 表示：
- 当前客户处于初始化、迁移、历史材料导入或等价 bootstrap context；
- 输入材料是历史材料，不是主 workflow 中的新交易 runtime handoff；
- 材料需要被保留为可追溯 evidence，并被解析为 profile、entity、case、knowledge、rule / governance candidate 的初始化基础；
- request 本身不创造 accountant approval、governance approval 或 durable accounting conclusion。
典型 upstream source 包括：
- client onboarding / migration workflow
- accountant 提供的历史账本、说明、确认或补充材料
- historical bank statement / receipt / cheque / contract package
- prior customer context import

#### Downstream handoff produced

本节点输出 `onboarding_initialization_result`，供后续 workflow 使用。

主要 downstream consumers：
- `Evidence Intake / Preprocessing Node`：读取初始化阶段形成的 evidence foundation 或 historical transaction basis；不把 Onboarding summary 当作 raw evidence。
- `Transaction Identity Node`：读取 historical transaction identity seed 或 dedupe context；不把 Onboarding 输出当作新交易的最终处理结果。
- `Profile / Structural Match Node`：读取已确认 profile facts 或 profile candidates 的 review status；未确认 candidate 不得作为稳定 structural truth。
- `Entity Resolution Node`：读取 initial entity foundation、candidate aliases、candidate roles 和 ambiguity notes；candidate alias / role 不能支持 rule match。
- `Rule Match Node`：只读取 accountant / governance approved active rules；不得消费 rule candidate 作为 deterministic rule。
- `Case Judgment Node`：读取 accountant-derived historical case foundation 和 customer knowledge summary；summary 不能替代 case evidence 或 rule authority。
- `Review Node` / `Governance Review Node`：消费 profile、entity、alias、role、rule、automation-policy、merge/split 和 conflict candidates。
- `Knowledge Compilation Node`：后续可重新编译或刷新 customer knowledge summary seed。

#### Logs / memory stores read

本节点可以读取或消费：
- historical raw evidence files / records：银行流水、小票、支票、合同、发票、历史账本原文、accountant 原话。
- existing `Evidence Log` references：如果客户迁移或补材料场景中已有 evidence records。
- existing `Profile` context：已知客户结构事实、tax config、bank accounts、loan / employee / owner-use context。
- existing `Entity Log` context：迁移已有 entity、alias、role、status、authority、automation_policy 和 risk flags。
- existing `Case Log` context：迁移已有 historical cases 或 prior accountant-corrected cases。
- existing `Rule Log` context：仅用于避免重复 rule candidate 或识别 authority conflict，不用于自行执行 rule match。
- existing `Governance Log` / `Intervention Log` context：已批准、已拒绝、待处理或近期 accountant 说明。
- `Knowledge Log / Summary Log`：只作为辅助背景，不能作为 deterministic authority。

#### Logs / memory stores written or candidate-only

本节点可以直接建立的 durable foundation 只限于 Stage 1/2 已允许的初始化语义：
- `Evidence Log` foundation：保存原始材料、source metadata 和 evidence references；不保存业务结论。
- historical transaction identity seed / basis：为历史材料中的交易提供稳定引用或 dedupe basis；不等于当前 runtime transaction finalization。
- accountant-derived historical `Case Log` foundation：只在历史处理结果来自 accountant-prepared books、accountant instruction 或等价高 authority source 时成立。
- sufficiently supported initial `Entity Log` foundation：只回答“是谁”和 role/context authority，不保存 COA / HST / JE 结论。
- `Knowledge Log / Summary Log` seed：可读客户知识摘要；不能作为 deterministic rule source。
本节点只能 candidate-only 输出：
- profile facts that require confirmation
- entity / alias / role governance candidates
- rule candidates 或 initial rule approval candidates
- automation policy candidates
- entity merge / split candidates
- conflict notes requiring accountant review
- evidence quality / traceability / authority issues
本节点不得直接写入或修改：
- 新交易的 `Transaction Log`
- active `Rule Log` authority
- approved alias / rejected alias authority
- stable role confirmation，除非 active docs 明确允许的 `accountant_derived_role` 且保留 source / authority metadata
- entity merge / split mutation
- automation policy upgrade or relaxation
- stable `Profile` truth，除非已有明确 accountant authority 或 active docs 已支持该 truth
- journal entry、review-facing report 或 final accounting output

### Input Contracts

#### `onboarding_initialization_request`

Allowed `onboarding_scope` values：
- `new_client_initialization`
- `client_migration`
- `historical_backfill`
- `re_onboarding_candidate`
- `incremental_historical_import_candidate`
Allowed `requested_outputs` values：
- `evidence_foundation`
- `historical_transaction_basis`
- `profile_candidates`
- `entity_foundation`
- `historical_case_foundation`
- `knowledge_summary_seed`
- `rule_governance_candidates`
- `conflict_review_package`
Validation / rejection rules：
- `onboarding_request_id`、`client_id`、`onboarding_scope`、`material_package_refs`、`source_authority_context` 缺失时，input invalid。
- `material_package_refs` 不能为空；空材料不能生成初始化记忆，只能返回 invalid / blocked result。
- `onboarding_scope` 不能表示普通 runtime transaction classification；若输入是新交易 runtime handoff，应拒绝并交由主 workflow。
- `source_authority_context` 必须能区分 raw evidence、accountant-processed historical basis、accountant instruction、client statement、system inference 和 migrated context。
- `requested_outputs` 不能要求 Onboarding 直接批准 active rule、automation policy upgrade、entity merge/split、unconfirmed profile truth 或 final journal entry。
Runtime-only vs durable references：
- request envelope 是 runtime-only。
- `client_id`、material refs、existing memory refs 和 accountant refs 是 durable references。
- 本节点不能仅凭 request metadata 产生 durable business conclusion；所有 durable output 必须绑定 material evidence 和 source authority。

#### `historical_material_package`

Allowed `material_type` values：
- `bank_statement`
- `historical_ledger`
- `receipt`
- `cheque`
- `invoice`
- `contract`
- `accountant_note`
- `client_note`
- `prior_system_export`
- `other_historical_evidence`
Allowed `source_actor_type` values：
- `accountant`
- `client`
- `firm_staff`
- `system_migration`
- `unknown`
Allowed `source_authority_category` values：
- `raw_evidence`
- `accountant_processed_books`
- `accountant_instruction`
- `client_statement`
- `system_migrated_context`
- `system_inferred`
- `unknown_authority`
Allowed `traceability_status` values：
- `traceable_to_original`
- `traceable_to_accountant_source`
- `derived_with_source_refs`
- `summary_only_insufficient_trace`
- `missing_source`
Validation / rejection rules：
- `material_ref` 必须可回到原始材料或 accountant source；summary-only 材料不能支持 stable entity、confirmed case 或 rule candidate authority。
- `source_authority_category = system_inferred` 的材料不能单独支持 accountant-derived case、stable role 或 active rule candidate。
- `source_actor_type = unknown` 或 `source_authority_category = unknown_authority` 时，只能作为 weak evidence 或 review context。
- `traceability_status = missing_source` 时，不得产生 durable memory write；最多产生 invalid / evidence gap issue。
Runtime-only vs durable references：
- `historical_material_package` 的分类 envelope 是 runtime-only。
- `material_ref`、linked refs、hash / fingerprint 和 persisted source metadata 可以成为 durable evidence references。

#### `historical_transaction_basis`

Allowed `direction` values：
- `inflow`
- `outflow`
- `transfer_unknown_direction`
- `unknown`
Validation / rejection rules：
- `amount` 必须采用 absolute amount + explicit `direction`；不得用正负号隐含方向。
- `transaction_date`、`amount`、`currency`、`bank_account_ref` 缺失时，不能形成正常 historical transaction basis；可以保留 evidence issue。
- `direction = unknown` 时不能支持 structural profile truth、stable case conclusion 或 rule candidate，除非 accountant source 明确补足。
- `accountant_classification_context` 只有在 source authority 是 `accountant_processed_books` 或 `accountant_instruction` 时，才能支持 historical case foundation。
- historical transaction basis 不得写入新交易 `Transaction Log`。
Runtime-only vs durable references：
- historical transaction basis envelope 是 runtime-only。
- accepted evidence refs 和 identity seed 可以成为 durable references。
- 本节点不能把历史交易 basis 变成 final current transaction processing record。

#### `accountant_processed_historical_basis`

Allowed `authority_category` values：
- `accountant_prepared_books`
- `explicit_accountant_instruction`
- `accountant_correction`
- `firm_migration_approved`
- `uncertain_accountant_source`
Validation / rejection rules：
- `authority_category = uncertain_accountant_source` 不能支持 confirmed historical case write；只能进入 candidate / review package。
- `accounting_treatment_summary` 不能只有 LLM 总结，必须绑定 ledger / accountant source refs。
- `coa_account_ref` 或 tax treatment context 如来自旧 COA / historical books，只是 historical case context，不自动创建 active rule 或 current COA mapping authority。
- 如果 accountant-processed result 与 raw evidence 冲突，必须保留 conflict issue；不得隐藏在 summary 中。
Runtime-only vs durable references：
- basis envelope 是 runtime-only。
- `basis_ref`、source docs、ledger refs、accountant note refs 可以作为 durable evidence / case references。

#### `customer_structural_basis`

Allowed `structural_fact_type` values：
- `bank_account`
- `internal_transfer_relationship`
- `loan`
- `employee_presence`
- `payroll_context`
- `tax_config`
- `owner_uses_company_account`
- `related_party_context`
- `other_profile_fact`
Allowed `confirmation_status` values：
- `already_confirmed_profile_truth`
- `accountant_confirmed_in_onboarding_source`
- `candidate_requires_confirmation`
- `conflicting_requires_review`
- `insufficient_evidence`
Validation / rejection rules：
- `confirmation_status = candidate_requires_confirmation`、`conflicting_requires_review` 或 `insufficient_evidence` 时，不得写成 stable Profile truth。
- client statement 可以支持 candidate 或 review question，但不能单独支持高 authority profile mutation，除非 accountant 后续确认。
- profile candidate 不得支持 current structural match，直到被确认为 stable Profile truth。
- `tax_config`、loan、employee、owner personal use 等高影响事实必须保留 source / authority 和 risk context。
Runtime-only vs durable references：
- structural basis envelope 是 runtime-only。
- confirmed Profile refs 和 evidence refs 是 durable references。
- candidate profile fact 不是 durable Profile truth；它可以作为 review / governance candidate 被持久追踪，但不能被 Profile / Structural Match 当作已确认事实。

#### `entity_onboarding_basis`

Allowed `signal_type` values：
- `raw_bank_text`
- `historical_ledger_name`
- `receipt_vendor`
- `cheque_payee`
- `invoice_party`
- `contract_party`
- `accountant_context`
- `existing_alias`
- `other_historical_identity_signal`
Allowed `source_priority` values：
- `primary`
- `supporting`
- `weak`
- `conflicting`
Allowed `entity_initialization_status` values：
- `stable_initial_entity_supported`
- `entity_candidate_only`
- `ambiguous_entity_group`
- `merge_split_candidate_only`
- `insufficient_identity_evidence`
- `conflicting_identity_evidence`
Validation / rejection rules：
- `surface_signals` 不能为空；没有 identity signal 时不能生成 entity foundation。
- `stable_initial_entity_supported` 必须有足够 traceable evidence，且不得依赖不可追溯 summary。
- `proposed_aliases` 默认是 candidate alias；不得支持 rule match，除非后续 governance / accountant approval 使其成为 approved alias。
- `proposed_role_context` 默认是 candidate role；只有 `role_authority_category = accountant_derived_from_processed_books` 或等价 explicit accountant authority，并保留 source metadata，才可作为受控例外进入 stable entity role context。
- entity basis 不得保存 COA、HST、JE、classification conclusion 或 deterministic rule authority。
- merge / split 只能输出 candidate；不得由 Onboarding 执行。
Allowed `role_authority_category` values：
- `accountant_derived_from_processed_books`
- `explicit_accountant_instruction`
- `system_inferred_candidate`
- `client_statement_candidate`
- `unknown_role_authority`
Runtime-only vs durable references：
- entity basis envelope 和 explanation 是 runtime-only。
- stable entity foundation 如被写入 `Entity Log`，必须只持有 identity / alias / role authority metadata 和 evidence links。
- candidate aliases、candidate roles、merge/split candidates 可以进入 governance review package，但不等于 durable approval。

#### `case_precedent_basis`

Allowed `case_authority_category` values：
- `accountant_prepared_books`
- `explicit_accountant_instruction`
- `accountant_correction`
- `firm_migration_approved`
- `insufficient_accounting_authority`
Allowed `case_write_eligibility` values：
- `eligible_for_accountant_derived_case`
- `candidate_case_requires_review`
- `blocked_by_missing_authority`
- `blocked_by_traceability_gap`
- `blocked_by_conflict`
Validation / rejection rules：
- `case_write_eligibility = eligible_for_accountant_derived_case` 必须绑定 accountant authority 和 traceable evidence。
- 缺少 accountant authority 的 historical transaction 不能写入 confirmed Case Log；只能作为 candidate case 或 evidence context。
- entity unresolved 不必自动阻止 case candidate，但会限制 confirmed case memory 的 entity binding；不得伪装成 resolved entity case。
- historical case 是 precedent，不是 deterministic rule；不能支持 Rule Match。
- Transaction Log 不参与 case write authority。
Runtime-only vs durable references：
- case precedent basis envelope 是 runtime-only。
- accepted `case_log_record` 可以成为 durable `Case Log` memory。
- candidate case 或 conflict package 只是 review / governance input，不是 confirmed case memory。

#### `rule_stability_basis`

Allowed `rule_candidate_status` values：
- `initial_rule_candidate`
- `rule_candidate_requires_accountant_review`
- `blocked_by_entity_authority`
- `blocked_by_alias_or_role_authority`
- `blocked_by_case_instability`
- `blocked_by_conflict_or_exception`
- `not_rule_candidate`
Validation / rejection rules：
- 本 input 只能支持 rule candidate；不能产生 active Rule Log authority。
- `supporting_case_refs` 如不是 confirmed accountant-derived cases，只能支持 weak candidate 或 review question。
- `entity_ref` 若只是 candidate entity、alias 未 approved 或 role 未 confirmed，必须阻断 active rule path。
- automation policy upgrade / relaxation 只能作为 candidate，不能由 Onboarding 直接生效。
- rule candidate 必须保留 exception / conflict boundary；不得把 mixed-use 或近期修正隐藏为稳定 rule。
Runtime-only vs durable references：
- rule stability basis 是 runtime-only candidate evaluation。
- rule candidate 可进入 governance package；active rule 只能由 accountant / governance approval 后产生。

#### `onboarding_authority_context`

Allowed `allowed_direct_write_categories` values：
- `evidence_foundation`
- `historical_transaction_identity_seed`
- `accountant_derived_historical_case`
- `sufficiently_supported_initial_entity`
- `knowledge_summary_seed`
Allowed `candidate_only_categories` values：
- `profile_fact_candidate`
- `alias_candidate`
- `role_confirmation_candidate`
- `entity_merge_split_candidate`
- `rule_candidate`
- `automation_policy_candidate`
- `conflict_review_candidate`
- `case_requires_review_candidate`
Allowed `blocked_categories` values：
- `current_transaction_log_write`
- `journal_entry_generation`
- `active_rule_creation_without_approval`
- `approved_alias_without_approval`
- `stable_role_without_authority`
- `entity_merge_split_without_governance`
- `automation_policy_relaxation_without_approval`
- `unconfirmed_profile_truth_write`
- `classification_conclusion_from_raw_evidence_only`
Optional fields：
- `governance_restriction_refs`：已生效或 pending governance restriction。
- `accountant_instruction_refs`：明确 accountant instruction refs。
- `authority_notes`：用于 explainability 的人类可读 authority 说明。
Validation / rejection rules：
- `allowed_direct_write_categories` 不得包含 active rule、current Transaction Log、JE、automation relaxation、merge/split approval 或 unconfirmed Profile truth。
- 若 `authority_context_ref` 缺失或与 input source authority 冲突，必须按更保守的 candidate-only / blocked path 输出。
- candidate-only category 不得被下游解释为 approval。
Runtime-only vs durable references：
- authority context envelope 是 runtime-only。
- governance / accountant instruction refs 是 durable references。

### Output Contracts

#### `onboarding_initialization_result`

Allowed `result_status` values：
- `completed_with_foundation`
- `completed_with_candidates_only`
- `partial_foundation_with_review_needed`
- `blocked_invalid_input`
- `blocked_by_authority_gap`
- `blocked_by_traceability_gap`
- `blocked_by_conflict`
Optional fields：
- `evidence_foundation_refs`
- `historical_transaction_identity_refs`
- `initial_entity_refs`
- `historical_case_refs`
- `knowledge_summary_refs`
- `profile_candidate_refs`
- `governance_review_request_refs`
- `quality_warnings`
- `open_issue_summary`
- `created_at`
Validation rules：
- `result_status = completed_with_foundation` 必须至少包含一个 allowed direct durable foundation ref。
- candidate-only run 不得伪装为 foundation write；`completed_with_candidates_only` 必须明确没有 stable durable authority change。
- `blocked_invalid_input`、`blocked_by_authority_gap`、`blocked_by_traceability_gap`、`blocked_by_conflict` 必须列出 `blocked_or_invalid_items`。
- `durable_foundation_refs` 不得包含 active rule、current Transaction Log、JE、automation upgrade、merge/split approval 或 unconfirmed Profile truth。
Durable / candidate / runtime：
- result envelope 是 runtime-only。
- listed durable refs 指向各自 memory store 中已允许的 foundation records。
- candidate package refs 指向 review / governance candidate tracking，不等于 approval。

#### `initial_evidence_record`

Allowed `evidence_type` values：
- `bank_statement_line`
- `historical_ledger_entry`
- `receipt_record`
- `cheque_record`
- `invoice_record`
- `contract_record`
- `accountant_note`
- `client_note`
- `migration_record`
- `other_evidence`
Optional fields：
- `covered_period`
- `linked_evidence_ids`
- `parse_quality_flags`
- `account_refs`
- `hash_or_fingerprint`
- `language_or_format_note`
Validation rules：
- evidence record 不得保存 COA conclusion、HST conclusion、rule result 或 JE。
- `content_ref_or_excerpt` 必须可追溯；不能只有 LLM paraphrase。
- `source_authority_category` 必须保留，不得把 accountant source 和 raw evidence source 抹平。
Durable / candidate / runtime：
- 这是 durable evidence foundation。
- 它不能单独成为 classification authority。

#### `historical_transaction_identity_seed`

Allowed `identity_status` values：
- `stable_historical_identity_seed`
- `possible_duplicate`
- `split_or_combined_transaction_candidate`
- `insufficient_identity_basis`
- `conflicting_identity_basis`
Optional fields：
- `linked_historical_transaction_refs`
- `source_file_refs`
- `bank_account_ref`
- `ledger_entry_refs`
- `quality_flags`
Validation rules：
- identity seed 不得包含会计分类结论。
- identity seed 不得写入 `Transaction Log`。
- duplicate / split / combined candidates 必须保持候选状态，不能强制合并历史 cases。
Durable / candidate / runtime：
- accepted identity seed 可以成为 durable identity / dedupe reference。
- ambiguous identity signals 是 candidate / issue context。

#### `profile_candidate_record`

Allowed `confirmation_status` values：
- `candidate_requires_confirmation`
- `accountant_confirmed_in_source`
- `already_confirmed_profile_truth`
- `conflicting_requires_review`
- `insufficient_evidence`
Optional fields：
- `existing_profile_ref`
- `affected_account_refs`
- `effective_period`
- `risk_flags`
- `suggested_review_question`
- `governance_candidate_ref`
Validation rules：
- 只有 `already_confirmed_profile_truth` 或 `accountant_confirmed_in_source` 可被后续流程评估为 stable Profile write；其具体 confirmation workflow 未在本 Stage 冻结。
- `candidate_requires_confirmation`、`conflicting_requires_review`、`insufficient_evidence` 不得被 Profile / Structural Match Node 当作 confirmed structural fact。
- `suggested_review_question` 只是 runtime helper，不是 evidence 或 authority。
Durable / candidate / runtime：
- profile candidate 可被持久追踪为 candidate package。
- 它不是 stable Profile truth，除非后续 accountant / governance authority 完成确认。

#### `initial_entity_record_or_candidate`

Allowed `entity_initialization_status` values：
- `stable_initial_entity_supported`
- `entity_candidate_only`
- `ambiguous_entity_group`
- `merge_split_candidate_only`
- `insufficient_identity_evidence`
- `conflicting_identity_evidence`
Allowed `entity_type` values：
- `vendor`
- `payee`
- `customer`
- `financial_institution`
- `government_agency`
- `employee_or_contractor`
- `related_party`
- `unknown_or_other`
Allowed stable `status` values：
- `candidate`
- `active`
- `merged`
- `archived`
Allowed alias statuses in this output：
- `candidate_alias`
- `approved_alias`
- `rejected_alias`
Validation rules：
- `stable_initial_entity_supported` 不等于可以自动分类；automation 仍由 rule / case / policy authority 控制。
- `candidate_alias` 不得支持 rule match。
- `candidate_role` 不得写成 stable role。
- `approved_alias` 或 `rejected_alias` 必须来自 existing governance / accountant authority，不得由 Onboarding 直接发明。
- `roles` 只有在 accountant-confirmed、existing Entity Log 或受控 `accountant_derived_from_processed_books` source metadata 支持时，才能成为 durable role。
- entity record 不得包含 COA、HST、JE 或 final classification conclusion。
- merge / split candidate 不得执行 mutation。
Durable / candidate / runtime：
- stable entity foundation 可写入 `Entity Log`，但只在 entity memory 职责内有效。
- candidate alias、candidate role、merge/split、automation policy candidate 只能进入 review / governance path。

#### `historical_case_record_or_candidate`

Allowed `case_write_status` values：
- `written_accountant_derived_case`
- `candidate_case_requires_review`
- `blocked_by_missing_accountant_authority`
- `blocked_by_traceability_gap`
- `blocked_by_identity_or_entity_conflict`
- `blocked_by_accounting_conflict`
Optional fields：
- `case_id`
- `coa_account_ref`
- `tax_treatment_context`
- `exception_conditions`
- `receipt_or_supporting_evidence_refs`
- `accountant_note_refs`
- `review_candidate_ref`
- `case_reason`
Validation rules：
- `written_accountant_derived_case` 必须有 accountant authority 和 traceable evidence。
- raw bank evidence、receipt、cheque 或 LLM inference 不能单独支持 confirmed historical case。
- historical case 不等于 deterministic rule。
- unresolved entity 可允许 candidate case，但 confirmed case 的 entity binding 必须如实标注 unresolved / candidate / stable context。
- case record 不得写入 `Transaction Log`，不得生成 JE。
Durable / candidate / runtime：
- `written_accountant_derived_case` 是 durable `Case Log` foundation。
- candidate / blocked statuses 是 review or issue context，不是 confirmed case memory。

#### `customer_knowledge_summary_seed`

Allowed `summary_scope` values：
- `client_initial_knowledge`
- `entity_patterns_overview`
- `case_precedent_overview`
- `risk_and_exception_overview`
- `conflict_and_review_needed_overview`
Optional fields：
- `related_entity_refs`
- `related_case_refs`
- `related_profile_candidate_refs`
- `related_governance_candidate_refs`
- `summary_quality_flags`
- `expires_or_refresh_reason`
Validation rules：
- `summary_text` 必须绑定 source refs；不能是不可追溯的自由发挥。
- summary 不能包含“已批准 rule”“已确认 Profile fact”“已合并 entity”等超出 source authority 的结论。
- summary 不能替代 Evidence Log、Case Log、Rule Log 或 accountant approval。
Durable / candidate / runtime：
- 可以作为 durable Knowledge / Summary seed。
- 只能作为 context，不得参与 deterministic rule match。

#### `rule_and_governance_candidate_package`

Allowed `package_status` values：
- `ready_for_review`
- `needs_evidence_refinement`
- `blocked_by_authority_gap`
- `blocked_by_conflict`
- `deferred_open_boundary`
Allowed `candidate_type` values：
- `profile_fact_confirmation`
- `alias_approval`
- `alias_rejection`
- `role_confirmation`
- `entity_merge_candidate`
- `entity_split_candidate`
- `rule_candidate`
- `automation_policy_candidate`
- `case_review_candidate`
- `conflict_review_candidate`
- `traceability_issue`
Allowed `approval_status` values：
- `not_submitted`
- `pending_review`
- `approved`
- `rejected`
- `deferred`
- `blocked`
Optional fields：
- `related_entity_refs`
- `related_case_refs`
- `related_profile_refs`
- `related_rule_refs`
- `proposed_old_value`
- `proposed_new_value`
- `risk_flags`
- `review_question_hint`
- `candidate_reason`
Validation rules：
- Onboarding-created candidate item cannot start as `approved` unless it references an existing prior approval; otherwise `approval_status` must be `not_submitted` or `pending_review`.
- `rule_candidate` cannot be consumed by Rule Match Node until governance / accountant approval creates active Rule Log authority.
- `automation_policy_candidate` cannot relax automation policy without approval; restrictive suggestions also need proper governance visibility if persisted.
- merge/split candidate cannot mutate Entity Log.
- candidate package is not a durable Log unless and until represented as a proper Governance Log event or approved memory mutation record.
Durable / candidate / runtime：
- package is candidate-only.
- It may create Governance Review request refs, but approval belongs downstream.

#### `onboarding_blocked_issue`

Allowed `issue_type` values：
- `invalid_onboarding_scope`
- `missing_required_material`
- `missing_traceable_source`
- `missing_accountant_authority`
- `objective_transaction_basis_invalid`
- `identity_conflict`
- `entity_ambiguity`
- `accounting_treatment_conflict`
- `profile_authority_gap`
- `rule_authority_gap`
- `governance_boundary_block`
Allowed `required_resolution_path` values：
- `request_correct_material`
- `request_accountant_confirmation`
- `send_to_review_node`
- `send_to_governance_review_node`
- `defer_until_open_boundary_resolved`
- `exclude_from_durable_memory`
Optional fields：
- `suggested_question`
- `conflicting_refs`
- `severity`
- `created_at`
Validation rules：
- blocked issue must not be hidden only inside knowledge summary.
- `exclude_from_durable_memory` must not delete evidence; it only means no durable business memory write.
- issue reason must distinguish traceability gap, accountant authority gap, identity ambiguity, entity ambiguity, and accounting conflict when known.
Durable / candidate / runtime：
- issue is runtime / review handoff.
- It can be persisted as review/governance context if downstream workflow chooses, but is not itself `Log` authority.

### Field Authority and Memory Boundary

#### Source of truth for important fields

| Field / Concept | Source of truth | Onboarding authority |
| --- | --- | --- |
| `client_id` | client / onboarding workflow | reference only |
| source material content | original material / Evidence Log | preserve and reference |
| objective transaction fields | bank / ledger / evidence source, normalized by deterministic intake | normalize within evidence bounds |
| `direction` | explicit evidence-derived direction | must be explicit; no sign-based ambiguity |
| accountant historical classification | accountant-prepared books / instruction / correction | may support accountant-derived case |
| Profile truth | Profile + accountant / governance confirmation | candidate unless authority already supports truth |
| entity identity | Entity Log / traceable evidence / accountant source | may create sufficiently supported initial entity |
| alias approval / rejection | Governance / accountant approval | candidate only unless pre-approved source exists |
| stable role | accountant-confirmed role or controlled accountant-derived role | no system-inferred stable role |
| historical case memory | accountant-derived historical treatment with evidence refs | may write only when eligible |
| active rule | Rule Log after accountant / governance approval | candidate only |
| automation policy relaxation | accountant / governance approval | candidate only |
| automation policy downgrade | governed policy / lint / governance rules | Onboarding may propose; direct downgrade authority not settled here |
| customer knowledge summary | source-bound synthesis over evidence/entity/case/governance refs | context only |
| Transaction Log | Transaction Logging Node final audit write | no write, no runtime learning |

#### Fields that can never become durable memory by this node

以下字段或信息不得由 Onboarding 直接变成 durable authority：
- LLM-only inference without source refs
- raw evidence interpreted as accounting conclusion
- candidate profile fact as stable Profile truth
- candidate alias as approved alias
- candidate role as stable role
- rule candidate as active Rule Log record
- automation policy upgrade / relaxation
- entity merge / split execution
- current transaction final outcome
- journal entry
- current transaction `Transaction Log` entry
- review-facing report output
- prompt trace、temporary candidate queue、debug trace

#### Fields that can become durable only after accountant / governance approval

以下字段可以由 Onboarding 提出 candidate，但必须经 accountant / governance authority：
- profile facts requiring confirmation
- alias approval / rejection
- role confirmation，除 active docs 允许的 accountant-derived role 受控例外外
- entity merge / split
- rule promotion / initial rule creation
- rule modification / deletion / downgrade
- automation policy upgrade or relaxation
- profile tax config、loan、payroll、owner-use 等高影响结构事实
- unresolved conflict resolution

#### Audit vs learning / logging boundary

- `Evidence Log` 保存原始证据和证据引用，不保存业务结论。
- `Case Log` 可以保存 accountant-derived historical case，供 future case precedent 使用；它不是 deterministic rule。
- `Rule Log` 只保存 accountant / governance approved deterministic rules；Onboarding 只能提出 rule candidate。
- `Transaction Log` 只保存每笔交易最终处理结果和审计轨迹；它不参与 runtime decision，也不是 learning layer。Onboarding 不写新交易 `Transaction Log`。
- `Knowledge Log / Summary Log` 是可读摘要，不是 rule source。
- Candidate package、review queue、runtime handoff、trace、summary draft 不能被命名或处理成 durable `Log`，除非它们进入 active baseline 定义的 durable store。

### Validation Rules

#### Contract-level validation rules

- 所有 durable outputs 必须绑定 `client_id`、source authority metadata 和 evidence/source refs。
- 所有 accounting treatment、case precedent、role、rule、profile fact、automation policy 相关字段必须携带 authority category。
- evidence、candidate、case、rule、governance、summary 语义必须分层，不能用一个 summary 字段混装。
- amount 必须是 absolute amount，direction 必须显式。
- `Transaction Log` 不能被用作 Onboarding learning layer。
- candidate-only output 必须明确 candidate type、approval status 和 downstream authority path。
- conflicting evidence 必须作为 issue / conflict candidate 暴露，不能被 summary 掩盖。
- `Knowledge Summary` 不能缺少 source refs，不能表达超出 source authority 的结论。

#### Conditions that make the input invalid

输入在以下情况下 invalid 或必须被阻断：
- `onboarding_initialization_request` 缺少 `client_id`、scope、material refs 或 authority context。
- scope 是普通 runtime transaction classification。
- material package 为空。
- material 无法追溯到原始 source 或 accountant source，却请求 durable memory write。
- historical transaction 缺少日期、金额、currency、bank account 或 explicit direction，且没有 accountant source 补足。
- source authority 无法区分 raw evidence、accountant-processed basis、accountant instruction、client statement 和 system inference。
- requested output 要求 Onboarding 直接写 active rule、Transaction Log、JE、automation relaxation、merge/split approval 或 unconfirmed Profile truth。

#### Conditions that make the output invalid

输出在以下情况下 invalid：
- durable foundation ref 指向 active rule、current Transaction Log、JE 或 review-facing report。
- profile candidate 被标记为 stable Profile truth 但没有 accountant / governance authority。
- candidate alias 被标记为 approved alias 但没有 prior approval reference。
- system-inferred role 被写入 stable role。
- historical case 缺少 accountant authority 或 traceable evidence。
- rule candidate 被输出为 active deterministic rule。
- automation policy candidate 被输出为已放宽或已升级。
- merge/split candidate 执行了 Entity Log mutation。
- knowledge summary 缺少 source refs 或替代了 evidence/case/rule authority。
- blocked conflict 只出现在 summary 中，没有结构化 issue record。

#### Stop / ask conditions for unresolved contract authority

后续 Stage 或 implementation task 遇到以下情况必须 stop / ask，不得自行补产品 authority：
- 需要决定 re-onboarding / incremental historical import 是否完整复用本节点 trigger boundary。
- 需要决定 profile candidate 到 stable Profile truth 的 exact approval workflow。
- 需要决定 initial rule candidate 到 active Rule Log 的 exact approval workflow。
- 需要决定初始化部分失败时主 workflow 是否可启动及启动限制。
- 需要为 stable entity、accountant-derived role、historical case write 设置 threshold number。
- 需要决定 Onboarding 是否可直接执行任何 automation policy downgrade。
- 需要把 candidate package 技术持久化为某种 queue / store / log。

## 4. Open Boundaries

### Stage 1: Open Boundaries

以下问题留到后续阶段或治理设计，不在 Stage 1 冻结：
- 哪些历史材料足以支持稳定 entity 初始化，哪些只能形成 entity candidate。
- 哪些 accountant-derived role 可以在 Onboarding 中进入稳定 entity context。
- 初始 profile 候选怎样获得 accountant confirmation。
- 初始 rule 候选在什么审批边界后才能成为 active deterministic rule。
- Onboarding 产生的 governance candidates 由哪个后续节点聚合、展示和批准。
- re-onboarding 或增量历史导入是否复用同一节点边界。
- Onboarding 失败、部分完成或材料冲突时，主 workflow 是否允许启动，以及以什么限制启动。

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
- incremental onboarding / re-onboarding 是否复用同一 trigger boundary
- 初始 rule candidate 到 active rule 的具体 approval workflow
- profile candidate 到 stable profile truth 的具体 confirmation workflow
- 初始化部分失败时主 workflow 的启动限制

### Stage 3: Open Contract Boundaries

以下问题未被 live docs 冻结，但不阻塞本 Stage 3 contract：
- `re_onboarding_candidate` 和 `incremental_historical_import_candidate` 是否完整复用本节点，还是需要独立 trigger / authority boundary。
- profile candidate 到 stable Profile truth 的 exact confirmation workflow。
- initial rule candidate 到 active Rule Log 的 exact approval workflow。
- 初始化部分失败时，主 workflow 是否允许启动，以及以什么 restrictions 启动。
- `stable_initial_entity_supported`、`accountant_derived_role`、`eligible_for_accountant_derived_case` 的具体 evidence threshold / scoring。
- Onboarding 是否可直接执行任何 automation policy downgrade，还是只能提出 candidate。
- historical transaction identity seed 的全局 registry owner 与 Transaction Identity Node 的 exact handoff。
- candidate package 是否需要独立持久化 store；若持久化，不能误称为 `Log`，除非 active baseline 明确定义。
- historical COA / tax treatment 与当前客户 COA / tax config 的 mapping contract 尚未冻结；Onboarding 只能保留 historical context 和 candidate。
