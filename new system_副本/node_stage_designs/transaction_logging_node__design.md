# Transaction Logging Node — 设计摘要

> 由原 Stage 1/2/3 合并瘦身而来；保留定位、边界、contract 字段和开放问题，删除阶段说明、examples、self-review、历史读档记录和重复解释。

## 1. 定位与职责

Stage 1 已确定以下内容：
- `Transaction Logging Node` 是 final audit-trail persistence node。
- 它写入 audit-facing `Transaction Log`。
- `Transaction Log` 保存每笔交易最终处理结果和审计轨迹。
- `Transaction Log` 只写和查询，不参与 runtime decision，也不是学习层。
- 它位于 finalizable outcome、必要 review 和 JE generation 之后。
- 它依赖稳定 `transaction_id`、final outcome、journal entry result、evidence references、review / intervention trace 和 upstream authority context。
- 它不做 evidence intake、identity、structural match、entity resolution、rule match、case judgment、pending clarification、review decision capture 或 JE generation。
- 它不替 accountant 作会计判断。
- 它不修改 Entity / Case / Rule / Governance memory。
- 它不执行 rule promotion、entity governance、profile update 或 automation-policy change。
- 它不把 transient handoff、queue、draft、candidate 或 report artifact 称为 `Log`。
- 它不重写历史日志来追随后续治理变化；后续解释应通过 governance event 或后续审计语境完成。

## 2. 逻辑与边界

### Trigger Boundary

`Transaction Logging Node` 在当前交易具备 final logging authority，且需要形成 audit-facing `Transaction Log` 记录时触发。

概念触发条件是：
- 当前交易已经完成 evidence intake、transaction identity 和必要的上游 classification / review workflow；并且
- 当前交易已有稳定 `transaction_id` 和可追溯 evidence foundation；并且
- 当前交易 outcome 已具备 finalization authority；并且
- 必要的 journal entry result 已由 `JE Generation Node` 形成，或后续阶段明确允许某类 terminal non-JE outcome 被最终记录；并且
- 当前交易尚未完成 final transaction logging。
典型触发来源包括：
- profile-backed structural path 已形成可落账结果，并完成必要 JE / review boundary。
- deterministic rule-handled path 已形成可落账结果，并完成必要 JE / review boundary。
- case-supported path 在当前 authority 边界内已形成可继续结果，并完成必要 JE / review boundary。
- pending clarification 或 review 已回收 accountant-provided context，形成可 final logging 的 outcome。
- accountant 已在 `Review Node` 中批准或修正当前交易 outcome，并且 `JE Generation Node` 已完成可用分录结果。
以下情况不能触发正常 final transaction logging：
- 当前交易仍 pending、still-pending、not-approved 或 unresolved。
- 当前交易存在 unresolved identity、ambiguous evidence、conflicting evidence、authority block、governance block 或 review-required condition。
- 当前交易尚未生成必要 journal entry result，或 JE generation 暴露 not-finalizable / consistency issue。
- 当前内容只是 candidate signal、review package、report draft、governance candidate、case memory candidate、rule candidate 或 lint finding。

### Input Categories

Stage 2 按判断作用组织 input categories，不列字段清单。

#### Transaction identity and objective basis

说明当前日志对象是哪一笔交易，以及这笔交易的客观基础是什么。

#### Final outcome basis

说明当前交易最终被怎样处理，以及该 outcome 来自哪条 authority path。

核心边界：
- final outcome 必须已经具备当前交易 finalization authority。
- pending、review-required、candidate-only、governance-needed 或 not-approved context 不能支持正常 final logging。
- 本节点不能把 upstream rationale 重新解释成新的 accounting outcome。

#### Journal entry completion basis

说明当前交易是否已有可供审计和输出使用的 journal entry result，以及分录生成是否通过必要一致性边界。

#### Authority and review basis

说明当前 outcome 为什么可以被最终记录，或为什么不能被最终记录。

#### Audit-support trace basis

说明最终日志需要保留哪些关键审计轨迹。

#### Downstream handoff basis

说明本次 final logging 后，哪些后续 workflow 可能需要读取已完成交易的审计语境。

### Output Categories

Stage 2 只定义 conceptual output categories，不冻结 routing enum 或对象形状。

#### Final transaction audit record

含义：当前交易的最终处理结果和关键审计轨迹被写入 audit-facing `Transaction Log`。

边界：
- 这是 durable audit record。
- 它记录当前交易当时的 final outcome、authority trace、evidence / review / intervention context 和 JE completion context。
- 它不参与未来 runtime decision。
- 它不是 Case Log memory write。
- 它不是 Rule Log、Entity Log、Governance Log 或 Profile mutation。
- 它不是 report draft、review package、handoff queue 或 candidate registry。

#### Final logging blocked handoff

含义：当前交易不能形成正常 final transaction audit record，因为 final logging 必需条件缺失、冲突或未获 authority。

典型原因包括：
- transaction identity 不稳定或无法绑定
- final outcome authority 不足
- review / accountant approval 缺失或冲突
- journal entry result 缺失、冲突或 not-finalizable
- evidence references 无法支撑 audit traceability
- governance block 或 unresolved conflict 存在
边界：
- Blocked handoff 是安全停止，不是审计记录补丁。
- 本节点不能用“先记下来再说”绕过 finalization authority。
- 后续应由 Review、Coordinator、JE Generation、上游 workflow 或 governance workflow 处理。

#### Audit-support query surface

含义：`Transaction Log` 可被后续节点、人类 review、输出 flow、knowledge compilation 或 post-batch lint 查询，用于理解已完成交易的历史处理依据。

边界：
- 查询是审计 / 分析用途，不是 runtime classification authority。
- 查询结果不能直接作为 rule source、case memory write 或 governance approval。
- 后续治理变化不应重写历史交易日志。

#### Candidate handoff preservation

含义：如果 final logging 过程中存在 case memory、entity / alias / role、rule、automation-policy、profile、tax config 或 governance 候选，本节点可以保留其与已完成交易的关系，供后续 workflow 评估。

边界：
- Candidate handoff 只表示“后续应评估”。
- 它不写长期 business memory。
- 它不批准 governance event。
- 它不改变 active rule、entity authority、Profile 或 automation policy。

#### Logging consistency issue signal

含义：本节点发现 final outcome、JE completion context、evidence references、review / intervention trace 或 transaction identity 之间存在不一致。

边界：
- 该 signal 只说明不能安全 final logging。
- 它不自行修正 upstream outcome。
- 它不自行重算 JE。
- 它不创建 governance event。

### Deterministic Code vs LLM Semantic Judgment

Stage 2 的核心边界是：

#### Deterministic code responsibility

Deterministic code 负责：
- 判断本节点是否被触发：当前交易是否具备 final logging authority，且尚未完成 final transaction logging。
- 检查当前交易是否有稳定 transaction identity 和可追溯 evidence foundation。
- 检查 final outcome、review / intervention trace、authority basis 和 JE completion context 是否足以支持 final logging。
- 阻止 pending、review-required、governance-needed、candidate-only、not-approved、not-finalizable 或 conflicting context 进入正常 final audit record。
- 执行 Transaction Log 的 durable write discipline。
- 保持日志记录与当前交易、evidence references、upstream path、review decision、JE result 和 authority trace 的绑定。
- 防止 Transaction Log 被用作 runtime decision、learning layer、rule source 或 governance approval。
- 防止 transient handoff、queue、draft、candidate 或 report artifact 被称为 `Log`。
- 识别 final logging 前的 traceability 缺失、重复落盘风险、authority conflict 或 JE consistency issue。
- 防止本节点修改 Profile、Entity Log、Case Log、Rule Log 或 Governance Log。

#### LLM semantic judgment responsibility

LLM 通常不应参与 final logging eligibility 或 durable write decision。

如果后续阶段允许 LLM 辅助，它最多可以在 code 允许的边界内帮助：
- 把上游 rationale、review / intervention context 和 JE explanation 总结成人类可读审计说明。
- 将多个 audit-support traces 合并成清晰 narrative。
- 解释为什么 final logging 被阻止。
- 生成候选信号的自然语言说明。

#### Hard boundary

LLM 不能：
- 决定当前交易是否具备 final logging authority
- 把 pending、not-approved 或 review-required 交易推进到 final logging
- 把 JE-blocked reason 改写成可落账结果
- 选择 COA 科目
- 判断 HST/GST tax treatment
- 生成或修正 journal entry
- 替 accountant approve current outcome
- 把 candidate 变成 durable memory
- approve / reject alias
- confirm role
- create stable entity
- merge / split entity
- create / promote / modify / delete / downgrade active rule
- upgrade or relax automation policy
- 批准 governance event
- 修改 `Profile`、Entity Log、Case Log、Rule Log 或 Governance Log

### Accountant Authority Boundary

Accountant 仍然拥有最终 accounting decision 和 durable authority。

本节点不能：
- 把系统高置信结果解释为 accountant approval
- 把 accountant silence、模糊回答或未完成 review 解释为 approval
- 把 review package 本身当作最终会计决定
- 把当前交易 correction 直接变成长期客户政策
- 把 repeated correction 自动升级成 active rule
- 把 alias / role / entity clarification 直接批准为 durable authority
- 把日志落盘解释为 accountant 已批准所有治理候选

### Governance Authority Boundary

Governance-level changes 不属于 `Transaction Logging Node` authority。

本节点不能：
- 修改 `Profile`
- approve / reject alias
- confirm role
- create stable entity authority
- merge / split entity
- archive / reactivate entity
- change automation policy
- create / promote / modify / delete / downgrade active rule
- approve governance event
- invalidate durable memory
- rewrite historical transaction logs to follow later governance changes

### Memory / Log Boundary

Stage 2 采用四层边界：read / consume、write allowed、candidate-only、no direct mutation。

#### Read / consume boundary

`Transaction Logging Node` 可以读取或消费以下 conceptual context：
- current transaction identity、objective transaction basis 和 evidence references
- final outcome from structural / rule / case / pending / review workflow
- journal entry result、JE rationale 和 JE consistency context
- upstream rationale and authority trace from Profile / Structural Match、Entity Resolution、Rule Match、Case Judgment、Coordinator / Pending Node 和 Review Node
- `Intervention Log` 中与当前交易相关的 questions、answers、corrections、confirmations 和 review context
- Entity / Case / Rule / Governance context 的引用或摘要，但只能作为 final audit trace 或 candidate context，不能重新做 runtime decision
- prior `Transaction Log` records only for duplicate-final-log prevention, audit continuity, historical display, or downstream analysis boundaries

#### Write allowed boundary

本节点可以执行的 durable write 只有 audit-facing `Transaction Log` 语义：
- 记录本笔交易最终处理结果。
- 记录 final outcome 的 workflow path 和 authority trace。
- 记录支持该结果的 evidence / review / intervention / JE 审计语境。
- 记录与后续 case memory、governance、knowledge compilation 或 lint 评估相关的候选关系。

#### Candidate-only boundary

本节点只能作为候选或 handoff signal 表达：
- case memory update candidate context
- entity / alias / role candidate context
- rule candidate、rule conflict 或 rule-health candidate context
- profile confirmation / profile change candidate context
- tax config or account-mapping review candidate context
- automation-policy / governance candidate context
- post-batch lint candidate context
- logging consistency issue signal

#### No direct mutation boundary

本节点绝不能：
- 修改 `Profile`
- 写入或修改 `Case Log`
- 写入或修改 `Rule Log`
- 写入或修改 stable `Entity Log` authority
- 写入或批准 `Governance Log` mutation
- 生成或修改 journal entry
- 创建 stable case memory
- 创建 stable rule
- 创建 stable entity
- approve / reject alias
- confirm role
- merge / split entity
- 修改 automation policy
- 把 review / intervention record 当作 final transaction log 的替代品
- 把 Transaction Log 变成 runtime decision source 或 learning layer
- 重写历史日志以追随后续治理变化

### Insufficient / Ambiguous / Conflicting Evidence Behavior

Stage 2 采用优先级边界：final authority first、traceability second、historical integrity third。

#### Final authority first

如果当前交易尚未具备 final logging authority，本节点不能写正常 final transaction audit record。

典型情况：
- transaction outcome still pending
- review not approved
- accountant correction ambiguous
- unresolved / ambiguous identity 仍影响 current outcome
- review-required policy 尚未处理
- governance block 存在
- 当前内容只是 candidate signal
- JE generation not-finalizable 或 consistency issue 未解决

#### Traceability second

如果 authority 允许继续，但交易身份、evidence references、upstream path、review / intervention trace 或 JE completion context 无法被可靠绑定，本节点仍不能正常 final logging。

典型情况：
- final outcome 无法绑定到稳定 `transaction_id`
- evidence references 缺失到无法支持审计解释
- review / accountant approval 无法绑定到具体交易或问题
- journal entry result 与当前交易或 authority trace 无法对应
- upstream rationale 与 final outcome 不一致

#### Historical integrity third

如果当前 final logging context 与既有 Transaction Log、governance history、entity / rule state 或 intervention history 发生冲突，本节点应保守处理。

典型边界：
- 同一交易疑似已完成 final logging：阻止重复正常落盘，输出 consistency issue。
- 后续 governance state 与历史交易当时引用不一致：不重写历史日志，通过 governance event 或后续解释处理。
- accountant correction 与 prior intervention 或 evidence 冲突：退回 Review / Coordinator，而不是选择一个版本落盘。
- rule / entity / alias / role authority 在处理过程中变化：记录或保留当时 authority trace；是否重新处理当前交易留到后续 workflow。

#### Blocked and not-finalizable caution

JE-blocked、not-finalizable、review-required、governance-needed 或 still-pending 状态不能被普通 final audit record 掩盖。

#### Hard boundary

- Transaction Logging 是 final audit persistence，不是 hidden review、classification、JE 或 governance layer。
- 能写日志不等于 outcome 有 accountant authority。
- Transaction Log 不参与 runtime decision，也不是学习层。
- Intervention Log 记录介入过程，不等于 final Transaction Log。
- Case Log 保存已完成交易可沉淀的案例，不等于 Transaction Log。
- Governance Log 保存高权限变化审批，不等于 Transaction Log。
- 模糊、冲突、缺失或未批准状态不能被包装成 final audit record。

## 3. Contract 字段摘要

### Contract Position in Workflow

#### Upstream handoff consumed

`Transaction Logging Node` 消费 `transaction_logging_request`。

该 request 只应在当前交易具备 final logging authority 时由 workflow 传入。上游必须已经完成或明确阻断：
- `Evidence Intake / Preprocessing Node`
- `Transaction Identity Node`
- `Profile / Structural Match Node`
- `Entity Resolution Node`、`Rule Match Node`、`Case Judgment Node` 中适用的 classification path
- `Coordinator / Pending Node` 或 `Review Node` 中必要的 clarification / approval / correction
- `JE Generation Node` 的 journal entry result，除非后续阶段明确允许某类 terminal non-JE outcome 进入单独 audit record

#### Downstream handoff produced

本节点可产生以下 output categories：
- `transaction_log_entry`：写入 audit-facing `Transaction Log` 的 durable final record。
- `transaction_log_write_receipt`：runtime-only 写入结果引用，供 workflow 继续交接。
- `final_logging_blocked_handoff`：runtime-only 安全停止 handoff，说明不能形成正常 final audit record。
- `logging_consistency_issue_signal`：runtime-only 一致性问题信号。
- `post_logging_handoff`：runtime-only 下游交接，引用已写入的 `transaction_log_entry` 和 candidate relationships。

#### Logs / memory stores read

本节点可以读取或消费：
- 当前交易的 objective transaction basis 和 `Evidence Log` references
- `Transaction Identity Node` 产生的稳定 `transaction_id`
- 上游 workflow path、classification result、authority trace 和 rationale references
- `JE Generation Node` 的 journal entry result、JE consistency status 和 calculation rationale reference
- `Intervention Log` / review context 中与当前交易绑定的 questions、answers、corrections、confirmations 和 approvals
- `Profile`、`Entity Log`、`Case Log`、`Rule Log`、`Governance Log` 的引用或摘要，但只能作为审计 trace、authority trace 或 candidate context
- 既有 `Transaction Log` 记录，但仅用于 duplicate-final-log prevention、audit continuity 或 historical display boundary

#### Logs / memory stores written or candidate-only

本节点唯一允许的 durable write 是 audit-facing `Transaction Log` 语义下的 `transaction_log_entry`。

本节点只能 candidate-only 保留：
- case memory update candidate context
- entity / alias / role candidate context
- rule candidate、rule conflict 或 rule-health candidate context
- profile change / tax config / account-mapping review candidate context
- automation-policy / governance candidate context
- post-batch lint candidate context

### Input Contracts

#### `transaction_logging_request`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `logging_request_id` | 本次 logging request 引用 | workflow runtime | runtime |
| `client_id` | 当前客户标识 | client / batch context | durable reference |
| `batch_id` | 当前处理批次 | workflow runtime | runtime |
| `transaction_context` | 当前交易客观基础 | Evidence Intake / Transaction Identity | runtime object with durable refs |
| `final_outcome_context` | 当前交易最终处理结果 | upstream classification / review path | runtime object with durable refs |
| `journal_entry_context` | JE result 或 JE blocking context | `JE Generation Node` | runtime object with durable refs |
| `authority_context` | final logging authority 与限制 | upstream workflow / accountant / governance context | runtime projection |
| `audit_trace_context` | 需要保留的审计轨迹 | upstream workflow | runtime object with durable refs |

**Optional fields**

- `candidate_context`：只保留候选关系，不执行 durable mutation。
- `existing_log_context`：既有 Transaction Log 状态，用于阻止重复 final logging。
- `runtime_notes`：runtime diagnostics；不能作为业务结论或 authority。

#### `transaction_context`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `transaction_id` | 稳定交易 ID | `Transaction Identity Node` | durable reference |
| `transaction_date` | 交易日期 | evidence / preprocessing | durable transaction fact |
| `amount_abs` | 绝对金额 | evidence / preprocessing | durable transaction fact |
| `direction` | 资金方向 | evidence / preprocessing | durable transaction fact |
| `bank_account` | 客户侧银行账户引用 | evidence / Profile context | durable reference |
| `evidence_refs` | 支撑当前交易的证据引用 | `Evidence Log` | durable references |

**Allowed values**

`direction` 允许：
- `inflow`
- `outflow`

**Optional fields**

- `raw_description`
- `description`
- `pattern_source`
- `receipt_refs`
- `cheque_info_refs`
- `counterparty_surface_text`
- `currency`
- `import_batch_id`

#### `final_outcome_context`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `outcome_status` | 当前 outcome 是否可最终记录 | upstream workflow | runtime |
| `authority_path` | outcome 的来源路径 | upstream workflow | runtime |
| `accounting_outcome_summary` | 当前交易最终会计处理摘要 | upstream classification / review path | runtime snapshot |
| `outcome_rationale_refs` | 支持 outcome 的 rationale / evidence / memory refs | upstream workflow | durable refs |

**Allowed values**

`outcome_status` 允许：
- `finalizable_system_outcome`
- `accountant_approved_outcome`
- `accountant_corrected_outcome`
- `pending`
- `review_required`
- `not_approved`
- `governance_blocked`
- `conflicting`
`authority_path` 允许：
- `profile_structural`
- `rule_handled`
- `case_supported`
- `pending_resolved`
- `accountant_approved`
- `accountant_corrected`

**Optional fields**

- `entity_ref`
- `rule_ref`
- `case_refs`
- `review_decision_ref`
- `intervention_refs`
- `policy_trace`
- `confidence_summary`
- `outcome_notes`

#### `journal_entry_context`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `je_status` | JE 是否已生成且可用于 final logging | `JE Generation Node` | runtime |
| `je_trace_refs` | JE 计算依据和结果引用 | `JE Generation Node` | durable / runtime refs |

**Allowed values**

`je_status` 允许：
- `generated`
- `blocked`
- `not_finalizable`
- `not_required_open_boundary`

**Conditionally required fields**

当 `je_status = generated` 时 required：
- `journal_entry_ref`
- `balanced`
- `je_total_debits`
- `je_total_credits`
- `je_result_summary`
当 `je_status = blocked` 或 `not_finalizable` 时 required：
- `je_blocked_reason`
- `blocked_by_node`
- `required_resolution_context`

**Optional fields**

- `tax_summary`
- `calculation_rationale_ref`
- `consistency_issue_refs`
- `je_notes`

#### `authority_context`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `final_logging_authority_status` | 当前是否具备 final logging authority | upstream workflow / accountant / governance context | runtime |
| `authority_basis_refs` | 支持该 authority status 的引用 | upstream workflow | durable refs |
| `blocked_reasons` | 阻断 normal final logging 的原因；authorized 时可为空 | upstream workflow / this node validation | runtime |

**Allowed values**

`final_logging_authority_status` 允许：
- `authorized`
- `pending`
- `review_required`
- `not_approved`
- `governance_blocked`
- `je_blocked`
- `traceability_blocked`
- `conflicting`

**Optional fields**

- `accountant_decision_ref`
- `review_decision_status`
- `automation_policy_snapshot`
- `rule_lifecycle_snapshot`
- `governance_restriction_refs`
- `authority_notes`

#### `audit_trace_context`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `workflow_path` | 本笔交易经过的主要 workflow path | workflow orchestrator | runtime |
| `evidence_refs` | 支撑当前处理结果的 evidence references | `Evidence Log` / upstream nodes | durable refs |
| `upstream_result_refs` | 上游 result / rationale references | upstream nodes | durable / runtime refs |
| `journal_entry_ref` | JE result 引用 | `JE Generation Node` | durable / runtime ref |

**Optional fields**

- `profile_structural_refs`
- `entity_refs`
- `rule_refs`
- `case_refs`
- `review_decision_refs`
- `intervention_refs`
- `governance_refs`
- `policy_trace`
- `audit_narrative`
- `trace_completeness_notes`

#### `candidate_context`

**Optional fields**

- `case_memory_candidate_refs`
- `entity_candidate_refs`
- `alias_candidate_refs`
- `role_candidate_refs`
- `rule_candidate_refs`
- `profile_change_candidate_refs`
- `tax_config_review_candidate_refs`
- `account_mapping_review_candidate_refs`
- `automation_policy_candidate_refs`
- `governance_candidate_refs`
- `post_batch_lint_candidate_refs`

#### `existing_log_context`

**Required fields when provided**

- `existing_final_log_status`

**Allowed values**

- `not_logged`
- `already_logged`
- `possible_duplicate_log`

**Optional fields**

- `existing_transaction_log_entry_refs`
- `prior_logging_notes`

### Output Contracts

#### `transaction_log_entry`

**Required fields**

| Field | Meaning | Authority | Durable / Runtime |
| --- | --- | --- | --- |
| `transaction_log_entry_id` | 本条 Transaction Log 记录引用 | Transaction Log write boundary | durable audit reference |
| `client_id` | 客户标识 | input request | durable reference |
| `batch_id` | 批次引用 | input request | durable / runtime reference |
| `transaction_id` | 稳定交易 ID | `Transaction Identity Node` | durable reference |
| `logged_at` | 写入时间 | Transaction Logging write event | durable audit metadata |
| `entry_status` | 本条记录状态 | Transaction Logging contract | durable audit field |
| `transaction_snapshot` | 当前交易客观事实快照 | `transaction_context` | durable audit snapshot |
| `final_outcome_snapshot` | 最终处理结果快照 | `final_outcome_context` | durable audit snapshot |
| `journal_entry_snapshot` | JE result summary / refs | `journal_entry_context` | durable audit snapshot |
| `authority_trace` | final logging authority refs | `authority_context` | durable audit snapshot |
| `audit_trace_refs` | evidence / workflow / review / intervention refs | `audit_trace_context` | durable references |

**Allowed values**

`entry_status` 当前只允许：
- `final_logged`

**Optional fields**

- `policy_trace`
- `candidate_relationships`
- `audit_narrative`
- `trace_completeness_notes`
- `source_node_versions`

#### `transaction_log_write_receipt`

**Required fields**

- `logging_request_id`
- `write_status`
- `transaction_id`

**Allowed values**

`write_status` 允许：
- `written`
- `blocked`
- `consistency_issue`

**Conditionally required fields**

- `transaction_log_entry_id`：当 `write_status = written` 时 required。
- `blocked_handoff_ref`：当 `write_status = blocked` 时 required。
- `consistency_issue_ref`：当 `write_status = consistency_issue` 时 required。

**Optional fields**

- `post_logging_handoff_ref`
- `runtime_notes`

#### `final_logging_blocked_handoff`

**Required fields**

- `blocked_handoff_id`
- `logging_request_id`
- `transaction_id`
- `blocked_reason_type`
- `blocked_reason_summary`
- `required_resolution_context`
- `source_refs`

**Allowed `blocked_reason_type` values**

- `missing_transaction_identity`
- `missing_evidence_trace`
- `outcome_not_finalizable`
- `review_or_approval_missing`
- `journal_entry_missing`
- `journal_entry_not_finalizable`
- `authority_block`
- `governance_block`
- `conflicting_context`
- `duplicate_final_log`

**Optional fields**

- `suggested_return_node`
- `affected_candidate_refs`
- `accountant_facing_summary`
- `runtime_notes`

#### `logging_consistency_issue_signal`

**Required fields**

- `consistency_issue_id`
- `logging_request_id`
- `transaction_id`
- `issue_type`
- `issue_summary`
- `conflicting_refs`
- `safe_next_boundary`

**Allowed `issue_type` values**

- `transaction_id_mismatch`
- `evidence_ref_missing_or_mismatched`
- `outcome_je_mismatch`
- `authority_trace_conflict`
- `review_intervention_conflict`
- `duplicate_or_prior_log_conflict`
- `candidate_conflicts_with_finalization`

**Optional fields**

- `severity`
- `accountant_facing_summary`
- `governance_candidate_refs`
- `runtime_notes`

#### `post_logging_handoff`

**Required fields**

- `transaction_log_entry_id`
- `transaction_id`
- `client_id`
- `batch_id`
- `available_audit_refs`

**Optional fields**

- `case_memory_candidate_refs`
- `governance_candidate_refs`
- `knowledge_compilation_refs`
- `post_batch_lint_refs`
- `output_flow_refs`

### Field Authority and Memory Boundary

#### Source of truth for important fields

- `transaction_id` 的 source of truth 是 `Transaction Identity Node` / transaction identity layer。
- `amount_abs`、`direction`、`transaction_date`、`bank_account` 和 raw evidence references 的 source of truth 是 evidence / preprocessing flow 和 `Evidence Log`。
- classification / accounting outcome 的 source of truth 是完成该 outcome 的 upstream workflow path 或 accountant review / correction。
- `journal_entry_ref`、balanced status、JE totals 和 JE rationale 的 source of truth 是 `JE Generation Node`。
- accountant approval / correction 的 source of truth 是 `Review Node` / `Intervention Log` 中可追溯 decision context。
- entity、alias、role、rule、automation policy、governance restriction 的 source of truth 分别是对应 durable store 和 approved governance context。
- `transaction_log_entry` 的 source of truth 仅限已写入的 audit-facing `Transaction Log` record。

#### Fields that can never become durable memory by this node

本节点永远不能把以下字段或对象写成长期业务 memory：
- `candidate_context` 中的任何 candidate refs
- `policy_trace` 中的 runtime AI activation / risk-pack trace
- `confidence_summary`
- `audit_narrative`
- `runtime_notes`
- `accountant_facing_summary`
- `logging_consistency_issue_signal`
- `final_logging_blocked_handoff`
- `post_logging_handoff`

#### Fields that can become durable only after accountant / governance approval

以下内容如果要进入长期 authority，必须经后续 accountant / governance approval path：
- new entity、alias、role、merge / split、entity status 或 automation policy 变化
- rule creation、promotion、modification、deletion、downgrade 或 conflict resolution
- profile change、tax config change、account mapping change
- case memory update 是否可沉淀为 `Case Log`
- repeated correction 是否支持 rule candidate 或 automation-policy change

#### Audit vs learning / logging boundary

`Transaction Log` 是 audit-facing durable record。它保存“当时为什么这样处理”和“结果如何形成”的 trace。

它不是：
- runtime classification source
- deterministic rule source
- Case Log memory
- Rule Log memory
- stable Entity Log authority
- Governance Log approval
- Profile truth
- report draft 或 review package

### Validation Rules

#### Contract-level validation rules

- 每个 normal `transaction_log_entry` 必须绑定一个稳定 `transaction_id`。
- `transaction_log_entry` 必须有可追溯 evidence refs、final outcome refs、JE refs 和 authority refs。
- `amount_abs` 必须为正数；direction 只能由 `direction` 字段表达。
- `journal_entry_context.je_status` 必须是 `generated`，且 JE 必须 balanced，才可 normal final logging。
- `authority_context.final_logging_authority_status` 必须是 `authorized`，才可 normal final logging。
- candidate-only 内容必须显式保留 candidate boundary。
- `Transaction Log` 历史记录不得被用于当前 runtime classification / learning authority。

#### Conditions that make the input invalid

输入 invalid 或必须停止，如果：
- 缺少 `transaction_id`
- 缺少足够 evidence refs 支持 audit traceability
- final outcome 仍 pending、review-required、not-approved、conflicting 或 governance-blocked
- JE 缺失、blocked、not-finalizable、unbalanced 或无法绑定同一 transaction
- accountant approval / correction 缺失但 request 声称 accountant authority
- candidate-only signal 被包装成 final outcome
- 既有 `Transaction Log` 表明该交易已 final logged，且后续 correction / supersession contract 未定义

#### Conditions that make the output invalid

输出 invalid，如果：
- `transaction_log_entry.entry_status` 不是 `final_logged`
- durable entry 中缺少 transaction / outcome / JE / authority / evidence trace 任一核心绑定
- durable entry 包含本节点新创造的 COA、tax treatment、classification 或 JE result
- durable entry 把 candidate relationship 表示为 approved memory
- blocked handoff 同时声称 normal final log 已写入
- consistency issue 被忽略并仍写入 normal final record
- post logging handoff 要求下游把 `Transaction Log` 当作 runtime decision source

#### Stop / ask conditions for unresolved contract authority

后续阶段或实现前必须 stop / ask，如果需要决定：
- JE-blocked / not-finalizable 是否需要 terminal audit-facing durable record。
- correction、reversal、superseded transaction、split transaction 或 reprocessing 如何影响既有 `Transaction Log`。
- 哪些 high-confidence system outcome 可以不经逐笔 review 直接进入 JE / final logging。
- Review Node 是否覆盖所有 final logging 前交易，还是只覆盖 review-required / sampled / governance-candidate items。
- `Transaction Log` 与 final output artifact 的 exact field-level boundary。

## 4. Open Boundaries

### Stage 1: Open Boundaries

以下问题留到后续阶段，不在 Stage 1 冻结：
- 哪些 upstream outcome 可被视为具备 final logging authority。
- `Review Node` 是否覆盖所有 final logging 前交易，还是只覆盖 review-required / sampled / governance-candidate items。
- high-confidence structural / rule / case-supported result 是否可以不经逐笔 accountant review 直接进入 JE 和 final logging。
- JE-blocked / not-finalizable handoff 是否能形成某种 terminal audit record，还是只能保留在 intervention / review / blocked workflow 语境中。
- accountant approval、correction、rejection、still-pending 与 final logging 的 exact handoff contract。
- review / intervention trace 与 `Transaction Log` 的 exact persistence boundary。
- final `Transaction Log` 与 `Case Memory Update Node`、`Governance Review Node`、`Knowledge Compilation Node`、`Post-Batch Lint Node` 的 exact handoff contract。
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
- 哪些 upstream outcome 可被视为具备 `final logging authority`
- Review Node 是否覆盖所有 final logging 前交易，还是只覆盖 review-required / sampled / governance-candidate items
- high-confidence structural / rule / case-supported result 是否可以不经逐笔 review 直接进入 JE 和 final logging
- accountant approval、correction、rejection、still-pending 与 final logging 的 exact handoff contract
- JE-blocked / not-finalizable handoff 是否需要 terminal audit record，以及它与正常 completed-transaction logging 的边界
- review / intervention record 与 final `Transaction Log` 的 exact persistence contract
- final logging blocked handoff 应返回 Coordinator、Review、JE Generation、upstream rework 还是 governance workflow
- Transaction Log 与 Case Memory Update、Governance Review、Knowledge Compilation、Post-Batch Lint、output flow 的 exact downstream handoff contract
- duplicate final logging prevention、correction / reversal / superseded transaction、split transaction 和 reprocessing 的 exact behavior

### Stage 3: Open Contract Boundaries

- JE-blocked / not-finalizable 是否需要单独 terminal audit-facing durable record，仍未由 active docs 决定。本 Stage 3 只允许它阻止 normal final logging。
- correction、reversal、superseded transaction、split transaction 和 reprocessing 如何影响既有 `Transaction Log`，仍未冻结。
- 哪些 high-confidence structural / rule / case-supported result 可以不经逐笔 accountant review 直接进入 JE / final logging，仍属于跨节点 authority boundary。
- Review Node 是否覆盖所有 final logging 前交易，还是只覆盖 review-required / sampled / governance-candidate items，仍未冻结。
- `Transaction Log` 与 final output report / export artifact 的 exact field-level boundary 仍未冻结。
- `journal_entry_context.je_result_summary` 的完整字段结构应由 JE Generation 的 Stage 3 / shared contract 冻结；本文件只要求可追溯引用和 minimum audit summary。
- duplicate final logging prevention 已定义为 contract boundary，但 idempotency key、correction flow 和 historical supersession behavior 留给后续阶段。
