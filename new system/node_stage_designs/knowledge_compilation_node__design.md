# Knowledge Compilation Node — 设计摘要

> 由原 Stage 1/2/3 合并瘦身而来；保留定位、边界、contract 字段和开放问题，删除阶段说明、examples、self-review、历史读档记录和重复解释。

## 1. 定位与职责

Stage 1 已确定以下内容：
- `Knowledge Compilation Node` 是 durable-knowledge summarization node。
- 它把 entity、case、rule、governance 历史及必要审计背景编译成人和 agent 可读的客户知识摘要。
- 它的 durable 输出是 `Knowledge Log / Summary Log` 语义。
- 摘要帮助后续 humans 和 agents 理解常见模式、例外、风险和开放边界。
- 摘要不作为 deterministic rule source。
- 摘要不替代 `Entity Log`、`Case Log`、`Rule Log`、`Governance Log`、`Evidence Log` 或 `Transaction Log` 的 source authority。
- 它不处理当前交易分类、JE、pending、review approval 或 transaction logging。
- 它不执行 case memory write。
- 它不批准 entity、alias、role、rule、automation policy 或 governance 变化。
- `Transaction Log` 可以作为已完成交易审计背景来源，但不因此变成 runtime decision source 或学习层。

## 2. 逻辑与边界

### Trigger Boundary

`Knowledge Compilation Node` 在客户长期记忆、治理历史或完成交易背景发生足以影响客户知识摘要的变化时触发。

概念触发条件是：
- `Entity Log`、`Case Log`、`Rule Log`、`Governance Log` 或相关 completed audit history 已有可追溯变化；并且
- 这些变化可能影响 humans 或 agents 对客户常见模式、例外、风险、自动化边界或未解决问题的理解；并且
- 上游变化已经进入各自 durable source 或 formal candidate / handoff 语境；并且
- 当前任务是更新可读知识摘要，而不是处理当前交易、批准治理或执行 rule。
典型触发来源包括：
- onboarding 形成初始 entity、historical case、rule / governance candidates 或 customer knowledge seed。
- `Case Memory Update Node` 写入新完成案例或产生 knowledge-compilation handoff。
- `Governance Review Node` 记录批准、拒绝、暂缓、限制或 auto-applied downgrade 相关治理结果。
- `Transaction Logging Node`、Review、Intervention 或 audit history 暴露出对客户知识有解释价值的完成交易背景。
- `Post-Batch Lint Node` 暴露需要进入未来摘要或监控背景的风险、候选或 unresolved issue。
以下情况不能触发本节点去生成权威结论：
- 当前交易仍 pending、review-required、not-approved、JE-blocked、unresolved、conflicting 或 governance-blocked。
- 输入只是 runtime rationale、review draft、report draft、unapproved candidate、queue item 或未绑定 evidence 的自然语言说明。
- 需要 accountant / governance approval 的事项尚未获得相应 authority，却被要求写成稳定客户知识。

### Input Categories

Stage 2 按判断作用组织 input categories，不列字段清单。

#### Entity knowledge basis

说明客户有哪些稳定或受限 entity、alias、role、status、authority、risk flags 和 automation-policy context。

边界：
- Entity summary 只能概括 `Entity Log` 和已授权治理结果。
- candidate alias、candidate role、`new_entity_candidate` 或 merge / split candidate 可以作为未解决边界描述，但不能被写成 stable authority。
- 摘要不能替代未来 Entity Resolution 对 `Entity Log` 的读取。

#### Case knowledge basis

说明过去真实完成案例如何处理、有哪些条件、例外、correction、review context 或风险模式。

边界：
- Case summary 是历史先例的可读压缩。
- 它不是 simple majority rule。
- 它不能把多个相似案例自动升级成 deterministic rule。
- 它不能把未完成、未批准或冲突交易写成稳定案例知识。

#### Rule knowledge basis

说明哪些 deterministic rules 已被 accountant / governance 批准，哪些 rule 处于限制、冲突、拒绝、暂缓或需要 review 的状态。

边界：
- Rule summary 只能解释 active / historical rule authority。
- 它不能创建、修改、删除、降级或 promote rule。
- 它不能把 summary wording 变成 rule condition。
- Rule Match 仍必须读取 `Rule Log`，不能读取摘要来执行 rule。

#### Governance knowledge basis

说明长期记忆和自动化权威为什么发生变化，以及哪些事项被批准、拒绝、暂缓、限制或自动降级。

边界：
- Governance summary 是解释层，不是 approval。
- rejected / deferred decisions 可以作为风险或限制背景，但不能变成 future authority。
- 未解决治理风险应被表达为开放边界，不应被总结成稳定事实。

#### Completed audit and intervention basis

说明哪些已完成交易审计轨迹、intervention、review correction 或 accountant confirmation 对客户知识摘要有解释价值。

边界：
- `Transaction Log` 只作历史审计背景和 traceability 来源。
- 它不参与 runtime decision，也不是学习层。
- Knowledge Compilation 不能用 Transaction Log 直接推导 active rule、stable entity 或 accounting authority。

#### Lint / monitoring basis

说明批后检查或监控语境中有哪些 entity 拆合风险、rule instability、case-to-rule 候选、automation risk 或 unresolved issue 值得 humans / agents 后续看见。

边界：
- lint / monitoring finding 可以进入摘要作为风险或待处理背景。
- 它不是 governance approval。
- 它不能直接改变 active memory 或 automation authority。

### Output Categories

Stage 2 只定义 conceptual output categories，不冻结 routing enum 或对象形状。

#### Customer knowledge summary write

含义：把可追溯的客户长期语境编译成 durable `Knowledge Log / Summary Log` 摘要。

摘要可覆盖：
- 常见 entity / vendor / payee 语境
- 历史 case patterns 与例外条件
- 已批准 rules 的人类可读解释
- governance history 和 authority changes 的解释
- automation-policy 风险和限制
- unresolved boundaries、review cautions、lint-relevant issues
边界：
- 这是可读编译视图。
- 它不是 deterministic rule source。
- 它不是 current transaction accounting outcome。
- 它不是 accountant approval。
- 它不是 source evidence、case、entity、rule 或 governance record 的替代品。

#### Source-conflict / stale-summary warning

含义：摘要无法安全更新，或既有摘要与 source memory / governance history 不一致，需要标记 stale、conflict、supersession 或 review-needed。

边界：
- Warning 不修正 source logs。
- Warning 不自行选择冲突版本。
- Warning 不把不一致隐藏在自然语言里。

#### Knowledge update skipped / blocked handoff

含义：当前输入不足以编译可靠摘要，或候选事项尚未获得 source authority。

边界：
- Blocked handoff 是安全停止。
- 本节点不能用“先写摘要以后再修”绕过 source authority。
- 后续应返回 source memory workflow、Review、Governance Review、Post-Batch Lint 或人工澄清。

#### Candidate / follow-up signal

含义：编译过程中发现可能需要后续处理的候选或问题。

可能包括：
- stale entity / alias / role explanation candidate
- case memory consistency issue
- rule-health or rule-summary mismatch signal
- unresolved governance risk
- post-batch lint follow-up candidate
- review-facing clarification candidate
边界：
- Candidate signal 只供后续 workflow 评估。
- 它不是 durable approval。
- 它不修改 source memory。
- 它不改变 automation policy。

### Deterministic Code vs LLM Semantic Judgment

Stage 2 的核心边界是：

#### Deterministic code responsibility

Deterministic code 负责：
- 判断本节点是否被触发：是否存在需要编译或刷新客户知识摘要的 durable source change / handoff。
- 汇总并区分 source categories：Entity、Case、Rule、Governance、completed audit、intervention、lint / monitoring context。
- 检查每类输入的 authority：approved、completed、candidate-only、rejected、deferred、stale、conflicting、unresolved。
- 防止 candidate、draft、lint finding、runtime rationale 或 ambiguous accountant response 被写成稳定客户知识。
- 维护 source precedence：摘要必须服从 durable source logs / memory stores，而不是反过来覆盖它们。
- 标记 stale / conflict / supersession / unresolved 状态，防止过期摘要误导下游。
- 限制 durable write：本节点只能写 `Knowledge Log / Summary Log` 语义，不能写或修改 source authority stores。
- 防止摘要被下游误当作 deterministic rule source、stable entity authority、case authority、governance approval 或 Transaction Log。

#### LLM semantic judgment responsibility

LLM 可以在 code 允许的边界内辅助：
- 把多个 source-grounded cases 总结成可读的常见模式、例外条件和风险提示。
- 将 governance decisions、rule lifecycle、automation-policy context 转写成清楚的人类可读解释。
- 比较摘要草案与 source context，提示可能遗漏、过度概括、矛盾或 stale 的地方。
- 将 unresolved boundaries 写成明确警示，而不是模糊信心语言。
- 为 future agents 生成 compact context，使其知道哪些内容必须回查 source memory。

#### Hard boundary

LLM 不能：
- 生成 source logs 中不存在的客户事实
- 把 candidate 写成 approved fact
- 把 repeated cases 自动升级成 rule
- 把 summary wording 变成 rule condition
- 扩大 entity / alias / role authority
- 确认 role
- approve / reject alias
- 创建 stable entity
- merge / split entity
- 修改 automation policy
- create / promote / modify / delete / downgrade active rule
- 批准 governance event
- 选择 COA / HST treatment
- 生成或修正 journal entry
- 重写 Transaction Log、Case Log、Rule Log、Entity Log 或 Governance Log

### Accountant Authority Boundary

Accountant 仍然拥有最终 accounting decision 和 durable authority。

本节点不能：
- 把摘要写法解释为 accountant approval。
- 把 accountant silence、模糊回答或未完成 review 总结成稳定客户政策。
- 把当前交易 correction 自动变成长期治理事实。
- 把 repeated completed outcomes 自动升级成 active rule。
- 把 candidate alias、candidate role、`new_entity_candidate` 或 rule candidate 总结成已批准状态。
- 把未解决风险淡化为稳定自动化许可。

### Governance Authority Boundary

Governance-level changes 不属于 `Knowledge Compilation Node` 的 mutation authority。

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
- rewrite historical cases or transaction logs to match later summary wording

### Memory / Log Boundary

Stage 2 采用四层边界：read / consume、write allowed、candidate-only、no direct mutation。

#### Read / consume boundary

`Knowledge Compilation Node` 可以读取或消费以下 conceptual context：
- `Entity Log` 中的 entity、alias、role、status、authority、risk flags、automation policy 和 evidence links。
- `Case Log` 中的 completed cases、final classifications、evidence、exceptions、accountant corrections 和 review context。
- `Rule Log` 中的 approved active rules、historical rule authority、rule lifecycle 和 governance status。
- `Governance Log` 中的 approvals、rejections、deferrals、restrictions、auto-applied downgrades 和 authority rationale。
- `Transaction Log` 中的 finalized audit history，只作可读背景、traceability 和 completed-history context。
- `Intervention Log` 中的 accountant questions、answers、corrections、confirmations 和 unresolved intervention context。
- `Evidence Log` references，只用于保持摘要可追溯，不用于覆盖 source memory authority。
- `Profile` 中与客户结构、tax config 或 automation boundary 有关的稳定背景或待确认限制。
- `Post-Batch Lint Node` / monitoring handoff 中需要进入摘要或后续关注的风险语境。

#### Write allowed boundary

本节点可以执行的 durable write 只限于 `Knowledge Log / Summary Log` 语义：
- 写入或刷新 source-grounded customer knowledge summary。
- 标记摘要的风险、开放边界、stale / conflict / supersession 状态。
- 保存让 humans 和 agents 能回查 source memory 的摘要语境。
这些写入不等于：
- `Transaction Log` write
- `Case Log` write
- `Rule Log` mutation
- `Entity Log` authority mutation
- `Governance Log` approval
- `Profile` update
- report draft 或 review package

#### Candidate-only boundary

本节点只能作为候选或 issue signal 表达：
- source conflict / stale summary issue
- entity / alias / role summary mismatch candidate
- case memory consistency candidate
- rule-health / rule-summary mismatch candidate
- unresolved governance risk candidate
- automation-policy review candidate
- post-batch lint follow-up candidate
- review-facing clarification candidate

#### No direct mutation boundary

本节点绝不能：
- 写入或修改 `Transaction Log`
- 写入或修改 `Case Log`
- 写入或修改 `Rule Log`
- 写入或修改 stable `Entity Log` authority
- 写入或批准 `Governance Log` mutation
- 修改 `Profile`
- 生成或修改 journal entry
- 创建 stable case memory
- 创建 stable entity
- approve / reject alias
- confirm role
- merge / split entity
- 修改 automation policy
- 创建、升级、修改、删除或降级 active rule
- 把 candidate、queue、lint finding、review draft 或 report draft 称为 `Log`
- 把 Knowledge Summary 变成 runtime decision source、deterministic rule source 或 accountant approval

### Insufficient / Ambiguous / Conflicting Evidence Behavior

Stage 2 采用优先级边界：source authority first、traceability second、conflict preservation third、summary humility fourth。

#### Source authority first

如果输入缺少 durable source authority，本节点不能把它写成稳定客户知识。

典型情况：
- alias / role / entity 仍是 candidate。
- rule 只是 candidate、lint suggestion 或 repeated outcome。
- governance decision 仍 pending、deferred 或 authority 不足。
- accountant response 模糊，无法区分当前交易 correction 与长期政策。
- 当前交易仍未完成或未批准。

#### Traceability second

摘要必须能回到 source memory、evidence reference、case、transaction audit history、intervention 或 governance decision。

#### Conflict preservation third

如果 source memories 之间冲突，本节点不能用 LLM 选择一个“更合理”的版本写成摘要事实。

典型边界：
- Entity Log 与 governance history 冲突：标记 conflict，返回治理或 memory consistency workflow。
- Case Log 与 Rule Log 表达不同处理边界：摘要必须区分 case precedent 与 active rule。
- Transaction Log 历史处理与后续 governance state 不一致：不重写历史，通过治理历史解释时点差异。
- lint finding 与 approved authority 冲突：保持 warning / candidate，不改变 authority。

#### Summary humility fourth

摘要应压缩复杂语境，但不能过度概括。

本节点应避免：
- 把“通常如此”写成“必须如此”。
- 把少数例外隐藏在概括句里。
- 把未解决风险写成已解决结论。
- 把 stale summary 留给下游当作当前 authority。
- 用 confidence 语言掩盖 evidence、authority 或治理缺口。

#### Hard boundary

- Knowledge Summary 是编译视图，不是 source authority。
- Knowledge Summary 不等于 deterministic rule source。
- Knowledge Summary 不等于 stable entity authority。
- Knowledge Summary 不等于 case memory write。
- Knowledge Summary 不等于 governance approval。
- Transaction Log 是 audit-facing，不参与 runtime decision，也不是学习层。
- `new_entity_candidate` 不天然阻断当前交易分类，但不能在摘要中获得 stable entity、approved alias、confirmed role 或 rule authority。
- Active rule 变化必须经过 accountant / governance approval。
- Automation policy 升级或放宽必须 accountant approval。
- 模糊、冲突、缺失、过期或未批准状态不能被包装成稳定客户知识。

## 3. Contract 字段摘要

### Contract Position in Workflow

#### Upstream handoff consumed

`Knowledge Compilation Node` 消费 `knowledge_compilation_request`。

该 request 只应在客户长期记忆、治理历史、完成交易背景或 post-batch monitoring context 已经形成可追溯变化后传入。典型 upstream source 包括：
- `Onboarding Node`：初始 entity、historical case、rule / governance candidate 或 customer knowledge seed。
- `Case Memory Update Node`：已完成案例写入后，或产生需要刷新摘要的 handoff。
- `Governance Review Node`：已记录 approval、rejection、deferral、restriction 或 auto-applied downgrade 后。
- `Transaction Logging Node` / `Review Node` / `Intervention Log` context：已完成交易、accountant correction、confirmation 或解释性审计背景。
- `Post-Batch Lint Node`：风险、候选、unresolved issue 或 monitoring context 需要未来 humans / agents 看见。

#### Downstream handoff produced

本节点可产生以下 output categories：
- `knowledge_summary_record`：写入 durable `Knowledge Log / Summary Log` 的客户知识摘要。
- `knowledge_summary_write_receipt`：runtime-only 写入结果引用。
- `summary_warning_handoff`：source conflict、stale、supersession 或 review-needed warning。
- `knowledge_update_blocked_handoff`：当前输入不足以可靠编译摘要，或 source authority 不足。
- `knowledge_follow_up_signal`：candidate-only 后续处理信号。

#### Logs / memory stores read

本节点可以读取或消费：
- `Entity Log`：entity、alias、role、status、authority、risk flags、automation policy、evidence links。
- `Case Log`：completed cases、final classifications、evidence、exceptions、accountant corrections、review context。
- `Rule Log`：approved active rules、historical rule authority、rule lifecycle、governance status。
- `Governance Log`：approvals、rejections、deferrals、restrictions、auto-applied downgrades、authority rationale。
- `Evidence Log` references：仅用于 traceability，不用于覆盖 source memory authority。
- `Transaction Log`：finalized audit history only；它是 audit-facing，不参与 runtime decision，也不是学习层。
- `Intervention Log`：accountant questions、answers、corrections、confirmations、unresolved intervention context。
- `Profile`：客户结构、tax config 或 automation boundary 的稳定背景或待确认限制。
- `Post-Batch Lint Node` / monitoring handoff：需要进入摘要或后续关注的风险语境。
- 既有 `Knowledge Log / Summary Log`：用于 staleness、conflict、supersession 或 refresh context。

#### Logs / memory stores written or candidate-only

本节点唯一允许的 durable write 是 `Knowledge Log / Summary Log` 语义下的 `knowledge_summary_record`。

该 write 只能保存：
- source-grounded customer knowledge summary
- source references、authority labels、scope、version / supersession metadata
- unresolved boundary、risk、warning、stale / conflict / review-needed status
本节点只能 candidate-only 输出：
- entity / alias / role summary mismatch candidate
- case memory consistency issue
- rule-health / rule-summary mismatch signal
- unresolved governance risk
- automation-policy review candidate
- post-batch lint follow-up candidate
- review-facing clarification candidate
本节点绝不能直接写入或修改：
- `Transaction Log`
- `Case Log`
- `Rule Log`
- stable `Entity Log` authority
- approved `Governance Log`
- `Profile`
- journal entry
- stable entity、approved alias、confirmed role、active rule、automation policy 或 accountant approval

### Input Contracts

#### `knowledge_compilation_request`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `compilation_request_id` | 本次 compilation request 引用 | workflow runtime | runtime |
| `client_id` | 当前客户标识 | client context | durable reference |
| `trigger_context` | 为什么需要编译或刷新摘要 | upstream workflow | runtime object with durable refs |
| `source_basis_bundle` | 本次可用于编译的 source-grounded context | source logs / memory stores | runtime object with durable refs |
| `authority_filter_context` | 哪些 source 可写成稳定摘要，哪些只能 candidate / unresolved | upstream authority + source stores | runtime projection |

**Optional fields**

- `existing_summary_context`：既有 summary 的 current / stale / superseded / conflict 状态。
- `requested_summary_scope`：上游建议的摘要范围；不能越过 source authority。
- `lint_or_monitoring_context`：post-batch 风险或 unresolved issue。
- `runtime_notes`：runtime diagnostics；不得作为业务事实或 authority。

**Allowed `trigger_context.trigger_source` values**

- `onboarding`
- `case_memory_update`
- `governance_review`
- `transaction_logging`
- `review_node`
- `intervention_context`
- `post_batch_lint`
- `manual_refresh`
- `workflow_orchestrator`

#### `trigger_context`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `trigger_source` | 触发来源 | upstream workflow | runtime |
| `trigger_reason` | 触发原因 | upstream workflow | runtime |
| `changed_source_refs` | 发生变化或需要重读的 source refs | source logs / memory stores | durable references |

**Optional fields**

- `batch_id`
- `transaction_ids`
- `entity_ids`
- `case_ids`
- `rule_ids`
- `governance_event_ids`
- `intervention_ids`
- `lint_finding_ids`
- `requested_refresh_mode`

**Allowed `trigger_reason` values**

- `initial_knowledge_seed`
- `new_completed_case`
- `governance_decision_recorded`
- `governance_risk_unresolved`
- `finalized_audit_context_available`
- `accountant_correction_or_confirmation`
- `lint_or_monitoring_issue`
- `source_conflict_detected`
- `summary_stale_or_superseded`
- `manual_refresh_requested`

#### `source_basis_bundle`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `basis_scope` | 本次 source bundle 覆盖的客户 / entity / case / rule / governance 范围 | workflow runtime + source refs | runtime |
| `entity_knowledge_basis` | entity / alias / role / automation policy context | `Entity Log` + `Governance Log` | runtime object with durable refs |
| `case_knowledge_basis` | completed case context | `Case Log` + completed workflow refs | runtime object with durable refs |
| `rule_knowledge_basis` | approved / historical rule context | `Rule Log` + governance refs | runtime object with durable refs |
| `governance_knowledge_basis` | authority change history and unresolved governance context | `Governance Log` | runtime object with durable refs |

**Optional fields**

- `completed_audit_basis`
- `intervention_basis`
- `profile_basis`
- `lint_monitoring_basis`
- `evidence_trace_basis`
- `existing_summary_refs`

#### `entity_knowledge_basis`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `entity_refs` | 本次摘要相关 entity references | `Entity Log` | durable references |
| `entity_authority_status` | entity context 的 authority 状态 | `Entity Log` / governance | runtime projection |
| `source_refs` | 支撑 entity summary 的 source refs | source logs | durable references |

**Optional fields**

- `display_names`
- `approved_aliases`
- `candidate_aliases`
- `rejected_aliases`
- `confirmed_roles`
- `candidate_roles`
- `entity_lifecycle_statuses`
- `automation_policies`
- `risk_flags`
- `merge_split_context`
- `governance_notes_refs`
- `new_entity_candidate_refs`

**Allowed authority / status values**

`entity_authority_status` 允许：
- `stable_authority`
- `candidate_only`
- `mixed_authority`
- `conflicting`
- `stale`
- `unresolved`

#### `case_knowledge_basis`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `case_refs` | 本次摘要相关 completed case refs | `Case Log` | durable references |
| `case_authority_status` | case context 是否可作为已完成历史先例 | `Case Log` / Review | runtime projection |
| `source_refs` | supporting evidence / review / case refs | source logs | durable references |

**Optional fields**

- `entity_refs`
- `final_classification_refs`
- `evidence_condition_refs`
- `exception_markers`
- `accountant_correction_refs`
- `review_context_refs`
- `case_pattern_candidates`
- `case_conflict_notes`

**Allowed `case_authority_status` values**

- `completed_authority`
- `candidate_only`
- `mixed_authority`
- `conflicting`
- `stale`
- `unresolved`

#### `rule_knowledge_basis`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `rule_refs` | 本次摘要相关 rule refs | `Rule Log` | durable references |
| `rule_authority_status` | rule context 的 authority 状态 | `Rule Log` / governance | runtime projection |
| `source_refs` | rule / governance supporting refs | source logs | durable references |

**Optional fields**

- `active_rule_refs`
- `historical_rule_refs`
- `restricted_rule_refs`
- `rejected_rule_candidate_refs`
- `deferred_rule_candidate_refs`
- `rule_health_notes`
- `rule_conflict_refs`
- `related_entity_refs`

**Allowed `rule_authority_status` values**

- `approved_active`
- `historical_or_superseded`
- `restricted`
- `candidate_only`
- `rejected`
- `deferred`
- `conflicting`
- `stale`

#### `governance_knowledge_basis`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `governance_event_refs` | 本次摘要相关 governance events | `Governance Log` | durable references |
| `governance_authority_status` | governance context 的 authority 状态 | `Governance Log` | runtime projection |
| `source_refs` | supporting governance / evidence refs | source logs | durable references |

**Optional fields**

- `approved_event_refs`
- `rejected_event_refs`
- `deferred_event_refs`
- `auto_applied_downgrade_refs`
- `pending_event_refs`
- `restriction_refs`
- `affected_entity_refs`
- `affected_rule_refs`
- `authority_rationale_refs`

**Allowed `governance_authority_status` values**

- `approved_or_applied`
- `rejected`
- `deferred`
- `pending`
- `auto_applied_downgrade`
- `mixed_authority`
- `conflicting`
- `unresolved`

#### `completed_audit_basis`

**Required fields when present**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `completed_transaction_refs` | 已完成交易引用 | `Transaction Log` / final workflow | durable references |
| `audit_context_status` | audit context 是否 finalized / usable as background | Transaction Logging / Review | runtime projection |
| `source_refs` | supporting audit / intervention refs | source logs | durable references |

**Optional fields**

- `review_decision_refs`
- `intervention_refs`
- `accountant_confirmation_refs`
- `accountant_correction_refs`
- `final_outcome_refs`
- `audit_trace_notes`

**Allowed `audit_context_status` values**

- `finalized_background`
- `intervention_confirmed`
- `correction_recorded`
- `candidate_only`
- `conflicting`
- `unresolved`

#### `lint_monitoring_basis`

**Required fields when present**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `finding_refs` | lint / monitoring finding refs | Post-Batch Lint / monitoring workflow | durable or runtime refs |
| `finding_authority_status` | finding 是否已治理、candidate-only 或 unresolved | lint / governance context | runtime projection |
| `source_refs` | supporting source refs | source logs / lint context | durable references |

**Optional fields**

- `entity_merge_candidate_refs`
- `entity_split_candidate_refs`
- `rule_instability_refs`
- `case_to_rule_candidate_refs`
- `automation_risk_refs`
- `memory_consistency_issue_refs`
- `review_follow_up_refs`

**Allowed `finding_authority_status` values**

- `candidate_only`
- `governance_needed`
- `review_needed`
- `approved_or_applied`
- `auto_applied_downgrade`
- `rejected`
- `deferred`
- `unresolved`

#### `authority_filter_context`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `stable_summary_allowed_refs` | 可写成稳定摘要事实的 source refs | source authority | runtime projection |
| `candidate_only_refs` | 只能作为 candidate / follow-up 的 refs | source authority | runtime projection |
| `unresolved_boundary_refs` | 必须保留 unresolved / review-needed / governance-needed 的 refs | source authority | runtime projection |
| `excluded_refs` | 不可进入摘要的 refs 及原因 | source authority / validation | runtime projection |

**Optional fields**

- `conflict_refs`
- `stale_refs`
- `superseded_refs`
- `source_precedence_notes`
- `authority_warnings`

#### `existing_summary_context`

**Required fields when present**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `summary_refs` | 既有摘要引用 | `Knowledge Log / Summary Log` | durable references |
| `existing_summary_status` | 既有摘要状态 | Knowledge Log + source comparison | runtime projection |

**Optional fields**

- `superseded_by_refs`
- `stale_reason`
- `conflict_reason`
- `source_snapshot_refs`
- `last_compiled_at`

**Allowed `existing_summary_status` values**

- `current`
- `stale`
- `superseded`
- `conflicting`
- `review_needed`
- `unknown`

### Output Contracts

#### `knowledge_summary_record`

**Required fields**

| Field | Meaning | Authority | Runtime / Durable |
| --- | --- | --- | --- |
| `summary_id` | 摘要记录唯一引用 | Knowledge Log write | durable |
| `client_id` | 当前客户标识 | client context | durable reference |
| `summary_scope` | 摘要覆盖范围 | compilation request + source refs | durable metadata |
| `summary_status` | 摘要当前可用状态 | compilation validation | durable metadata |
| `summary_sections` | 结构化可读摘要内容 | source-grounded compilation | durable compiled view |
| `source_refs` | 支撑摘要的 source refs | source logs / memory stores | durable references |
| `authority_labels` | 每段摘要的 authority / candidate / unresolved 标记 | authority filter | durable metadata |
| `compiled_from_request_id` | 来源 request 引用 | workflow runtime | durable metadata |
| `compiled_at` | 编译时间 | workflow runtime | durable metadata |

**Optional fields**

- `supersedes_summary_refs`
- `superseded_by_summary_ref`
- `stale_reason`
- `conflict_notes`
- `open_boundaries`
- `risk_warnings`
- `downstream_usage_limits`
- `follow_up_signal_refs`
- `review_needed_reason`

**Allowed `summary_scope.scope_type` values**

- `client_level`
- `entity_level`
- `case_pattern_level`
- `rule_level`
- `governance_level`
- `mixed_scope`

**Allowed `summary_status` values**

- `current`
- `current_with_open_boundaries`
- `stale`
- `superseded`
- `conflicting`
- `review_needed`
- `blocked_not_written`

#### `knowledge_summary_write_receipt`

**Required fields**

- `compilation_request_id`
- `write_status`
- `summary_refs`
- `written_at`

**Optional fields**

- `warning_refs`
- `blocked_handoff_ref`
- `follow_up_signal_refs`
- `runtime_notes`

**Allowed `write_status` values**

- `written`
- `refreshed`
- `superseded_previous`
- `warning_only`
- `blocked`
- `no_write_needed`

#### `summary_warning_handoff`

**Required fields**

- `warning_id`
- `client_id`
- `warning_type`
- `affected_summary_refs`
- `affected_source_refs`
- `warning_reason`

**Optional fields**

- `suggested_follow_up_target`
- `severity`
- `related_entity_refs`
- `related_case_refs`
- `related_rule_refs`
- `related_governance_event_refs`

**Allowed `warning_type` values**

- `source_conflict`
- `summary_stale`
- `summary_superseded`
- `authority_gap`
- `traceability_gap`
- `memory_consistency_issue`
- `review_needed`
- `governance_needed`

**Allowed `severity` values**

- `info`
- `caution`
- `blocks_summary_write`
- `blocks_downstream_use`

#### `knowledge_update_blocked_handoff`

**Required fields**

- `blocked_handoff_id`
- `compilation_request_id`
- `client_id`
- `blocked_reason`
- `blocking_refs`
- `required_resolution_owner`

**Optional fields**

- `safe_partial_summary_allowed`
- `warning_refs`
- `follow_up_signal_refs`
- `runtime_notes`

**Allowed `blocked_reason` values**

- `no_traceable_source_refs`
- `source_authority_insufficient`
- `source_conflict_unresolved`
- `candidate_only_input`
- `current_transaction_not_final`
- `governance_approval_missing`
- `review_approval_missing`
- `transaction_log_misused_as_learning_source`
- `requested_authority_out_of_scope`

**Allowed `required_resolution_owner` values**

- `source_memory_workflow`
- `review_node`
- `governance_review_node`
- `post_batch_lint_node`
- `case_memory_update_node`
- `workflow_orchestrator`
- `human_product_decision`

#### `knowledge_follow_up_signal`

**Required fields**

- `signal_id`
- `client_id`
- `signal_type`
- `signal_summary`
- `supporting_refs`
- `target_workflow`
- `authority_status`

**Optional fields**

- `related_summary_refs`
- `related_entity_refs`
- `related_case_refs`
- `related_rule_refs`
- `related_governance_event_refs`
- `priority_hint`

**Allowed `signal_type` values**

- `entity_summary_mismatch`
- `alias_or_role_review_candidate`
- `case_memory_consistency_issue`
- `rule_health_or_summary_mismatch`
- `unresolved_governance_risk`
- `automation_policy_review_candidate`
- `post_batch_lint_follow_up`
- `review_clarification_candidate`

**Allowed `target_workflow` values**

- `review_node`
- `governance_review_node`
- `post_batch_lint_node`
- `case_memory_update_node`
- `entity_memory_workflow`
- `rule_memory_workflow`
- `workflow_orchestrator`

**Allowed `authority_status` values**

- `candidate_only`
- `review_needed`
- `governance_needed`
- `unresolved`

### Field Authority and Memory Boundary

#### Source of truth

- `client_id`：client / batch context 是引用 authority；本节点不创建客户身份。
- `entity_id`、alias、role、entity status、automation policy：`Entity Log` + approved / applied `Governance Log` 是 source of truth。
- completed cases、final classification history、case exceptions、accountant correction context：`Case Log`、Review context 和 completed workflow refs 是 source of truth。
- active / historical / restricted rules：`Rule Log` + governance decisions 是 source of truth。
- approval、rejection、deferral、restriction、auto-applied downgrade：`Governance Log` 是 source of truth。
- finalized audit history：`Transaction Log` 是 audit-facing source of truth，但不参与 runtime decision，也不是 learning layer。
- accountant questions / answers / confirmations / corrections：`Intervention Log` / Review context 是 intervention source。
- raw evidence and evidence references：`Evidence Log` 是 source of truth。
- compiled customer knowledge wording：`Knowledge Log / Summary Log` 是 compiled-view source for the summary text only，不是 upstream facts 的 authority。

#### Fields that can never become durable memory by this node

本节点绝不能把以下内容写成 source memory 或 authority mutation：
- stable entity creation
- approved alias
- confirmed role
- entity merge / split / archive / reactivate
- automation policy upgrade or relaxation
- active rule create / promote / modify / delete / downgrade
- stable case memory write
- Transaction Log final record
- Profile update
- COA / HST / JE decision
- accountant approval or governance approval
- current transaction final classification

#### Fields that can become durable only through accountant / governance approval

以下字段或信号如果需要成为长期 authority，必须经对应 accountant / governance path，而不是本节点：
- `candidate_aliases` → 只能经 alias approval path 变成 approved alias。
- `candidate_roles` → 只能经 role confirmation path 变成 confirmed role。
- `new_entity_candidate_refs` → 只能经 entity governance / approved memory path 变成 stable entity。
- `case_pattern_candidates` → 只能经 completed case write 或 rule governance path 影响 source memory。
- `rule_health_or_summary_mismatch` / `case_to_rule_candidate_refs` → 只能经 rule governance path 影响 Rule Log。
- `automation_policy_review_candidate` → 降级必须符合 active baseline；升级或放宽必须 accountant approval。
- `governance_needed` signals → 只能经 Governance Review 形成 approved / rejected / deferred governance events。

#### Audit vs learning / summary boundary

- `Transaction Log` 保存最终处理结果和审计轨迹；本节点只能把 finalized audit context 当作解释背景引用。
- `Knowledge Log / Summary Log` 保存 compiled view；它帮助 humans / agents 理解客户语境，但不直接学习、执行 rule、批准 governance 或处理当前交易。
- `Log` 在本节点语境中仍表示 durable long-term meaningful information storage；candidate queues、transient handoffs、review draft、report draft 或 runtime notes 不能被称为 Log。

### Validation Rules

#### Contract-level validation rules

- Every stable summary fact must have traceable `source_refs`.
- Every source ref must have an authority label: stable, candidate-only, unresolved, rejected, deferred, stale, superseded, conflicting, or excluded.
- Summary text must preserve source authority distinctions; candidate / unresolved content must not be phrased as settled fact.
- Conflicts must be explicit. The node cannot use LLM judgment to silently choose a source version.
- If existing summary is stale or conflicting, new output must mark staleness / conflict / supersession.
- Summary output must include downstream usage limits when there is any risk it may be mistaken for rule or authority.
- The node may write only `Knowledge Log / Summary Log` semantics.

#### Conditions that make the input invalid

Input is invalid or blocked when:

- required envelope fields are missing;
- no traceable source refs exist;
- current transaction is not completed but is presented as stable case / audit basis;
- governance-required change lacks approval but is presented as authority;
- candidate alias / role / entity / rule is presented as approved;
- source conflicts are present but conflict metadata is omitted;
- `Transaction Log` is presented as runtime decision or learning source;
- request asks this node to classify a transaction, generate JE, approve governance, mutate Profile, or modify source logs.

#### Conditions that make the output invalid

Output is invalid when:

- `knowledge_summary_record.summary_sections` contains facts without source refs;
- `summary_status = current` while known conflicts or stale source refs are unresolved;
- candidate-only content is written under `authority_labels = stable_fact`;
- summary text creates new accounting treatment, COA, HST, JE, rule condition, entity authority, alias approval, role confirmation, or automation permission;
- `knowledge_follow_up_signal.authority_status` is anything stronger than candidate / review-needed / governance-needed / unresolved;
- `knowledge_summary_write_receipt.write_status = written` without summary refs;
- warning or blocked handoff lacks affected / blocking refs and reason.

#### Stop / ask conditions for unresolved contract authority

Stop and ask for product decision before freezing a narrower contract if:

- downstream design requires summary to become a runtime decision source;
- exact `Knowledge Log / Summary Log` granularity must be frozen as one canonical storage shape;
- source precedence between source logs and summaries is proposed to change;
- unresolved governance or lint findings are proposed to become stable policy by summary wording;
- Transaction Log is proposed as direct learning input for runtime nodes;
- summary lifecycle rules would mutate source memory or authority automatically.

## 4. Open Boundaries

### Stage 1: Open Boundaries

以下问题留到后续阶段，不在 Stage 1 冻结：
- Knowledge Compilation 的 exact trigger：onboarding seed、批后定期编译、durable memory mutation 后增量编译，或组合模式。
- `Knowledge Log / Summary Log` 的 exact scope：客户级总摘要、entity-level 摘要、case-pattern 摘要、governance summary 是否分层。
- 摘要 staleness、supersession、versioning 和 refresh 的 exact behavior。
- 摘要与 source logs 冲突时的 exact source-precedence 和 warning contract。
- rejected / deferred governance decision、unresolved risk、lint finding 是否以及如何进入摘要。
- Knowledge Compilation 与 `Post-Batch Lint Node` 对风险摘要、监控发现和候选提示的 exact split。
- downstream nodes 读取摘要时必须同时读取哪些 source authority。
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
- Knowledge Compilation 的 full-refresh、incremental-refresh、batch-refresh、governance-triggered refresh 边界
- `Knowledge Log / Summary Log` 的 summary granularity：client-level、entity-level、case-pattern-level、rule-level、governance-level 是否分层
- stale、superseded、conflict、review-needed summary 的 exact lifecycle
- source logs 与 summary 发生冲突时的 exact precedence and repair workflow
- Onboarding seed summary 与后续 runtime / batch summary 的 merge or replacement behavior
- rejected / deferred governance decision、unresolved lint finding 和 monitoring issue 的 exact summary wording boundary
- Knowledge Compilation 与 Post-Batch Lint 对 risk summary、monitoring finding、candidate follow-up 的 exact split
- downstream runtime nodes 读取 summary 时必须同步读取哪些 source memory 的 exact contract

### Stage 3: Open Contract Boundaries

- Exact `Knowledge Log / Summary Log` granularity 仍未冻结：client-level、entity-level、case-pattern-level、rule-level、governance-level 是否作为独立 durable record，还是以 `summary_scope` 区分，留到后续阶段。
- Exact lifecycle 仍未冻结：full-refresh、incremental-refresh、batch-refresh、governance-triggered refresh、supersession behavior 和 retention policy 留到后续阶段。
- `summary_status` 与 source log conflict 的 repair workflow 未冻结；本 Stage 3 只要求显式 warning / blocked / stale / conflict。
- Downstream runtime nodes 读取 summary 时必须同步读取哪些 source memory 的 exact contract 未冻结；当前只冻结“summary 不可替代 source authority”。
- Knowledge Compilation 与 `Post-Batch Lint Node` 对 risk summary、monitoring finding、candidate follow-up 的 exact split 仍未完全冻结。
- rejected / deferred governance decision、unresolved lint finding 和 monitoring issue 的 exact human-readable wording style 未冻结；当前只冻结 authority label 和 boundary。
- Onboarding seed summary 与后续 runtime / batch summary 的 merge or replacement behavior 未冻结。
