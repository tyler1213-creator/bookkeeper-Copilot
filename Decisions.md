# Decisions

> 确定性决策记录。每条 = 做了什么 + 为什么 + 动了哪些文档。给新窗口快速对齐用，不展开论证。

## Entity Resolution（L3）

### D1 · 删除 ER 的并行候选 / 风险通道 — 2026-06-25
删除 `candidate_signal`、`merge_split_candidate`、`alias_conflict_issue`、`identity_governance_issue`、`blocking_reason`，以及作 ER **输出**的 `identity_risk_flags`。
为什么：无下游消费者（CJ 只读身份结果 + unknown reason；Human Review 从 Entity Log 读、且只接会计师人发起）；与 `unknown` reason 重复；merge/split 超出 ER 能力（单笔粒度看不到 split；merge 需被架构否决的语义相似）与职责（归 Entity Log + 会计师人发起 Human Review）。身份卡点一律经 `unknown` + reason 表达。`identity_risk_flags` 真值归 Entity Log，ER 只读。
文档：ER `00/01/02`。

### D2 · Alias 命中只认"是谁"，删"命中但不干净" — 2026-06-25
exact alias 命中 → 直接复用该 entity → `stable`，不再判该 entity 是否 active / 冲突 / merged。删 §5 的"冲突或多 entity 竞争"检测、§9 的"alias 库冲突 / 同一 surface 对多 entity → unknown"整段。
为什么：ER 只回答"是谁"；entity 干不干净是下游（Case Judgment 分类）与 Entity Log（supersession）的事，在身份这步无区别。高度相似 = 未完全命中，二者都落证据路。
文档：ER `02` §5、§9。

### D3 · §10 trace 瘦身 — 2026-06-25
ER 不设独立 trace 账。审计面 = 输出（`identity_state` / `entity_id`）+ `identity_reason`（stable）/ `unknown_reason_context`（unknown）+ `identity_evidence_refs`（外部来源作其 ref 变体）。删 `evidence_used`、`matched surface text`、`matched Alias surface text`、`Alias lookup basis`、`transaction_id`-作-trace；creation provenance 归 Entity Log；治理 / 介入 ref 折进 reason。
为什么：这些是"输出 / 落库内容 / 可重建值"的重复；ER 近乎无状态，output + reason 即审计。
文档：ER `02` §10。

### D4 · 保留两类冲突处理（非 alias 命中） — 2026-06-25
保留 §9 的"evidence vs Entity Log / Governance 冲突 → unknown"与"accountant context vs durable memory 冲突"。
为什么：它们管"认得对不对"（硬权威 > LLM 语义），不是 alias 命中的干净度，不在 D2 删除范围。
文档：ER `02` §9（未改，确认保留）。

### D5 · identity 审计材料落 Transaction Log — 2026-06-25
`identity_reason` + `identity_evidence_refs`（含外部来源 ref）是交易级审计，随交易在 finalization 时落 **Transaction Log**；ER 只产出、不写。新建实体的 creation provenance / 初始 alias 仍落 **Entity Log / Alias Log**。
为什么：身份的"为什么 + 凭什么"是交易作用域（同一 entity 各交易证据不同），Entity Log 是实体作用域、不存 per-transaction 推理（[entity_log 01:67](BK_Copilot/memory_layers/entity_log/01_memory_intent.md)）；ER 绝不写 Transaction Log（[ER 01:57](BK_Copilot/workflow_nodes/entity_resolution_node/01_functional_intent.md)），故由 finalization 落盘。"按 entity 查历史身份证据"是检索需求（Transaction Log 按 entity 建索引，L4/seam），不是复制进 Entity Log 的理由。
文档：ER `02` §10。

### D6 · ER 不预加工证据，下游读 raw — 2026-06-25
ER 不把证据预消化成共享产物给下游；Case Judgment 为会计分类**独立读 raw 证据**（经自身 evidence_refs 回 Evidence Log，与 ER 读的是同一份）。ER 的 `identity_evidence_refs` 是**指针（ref）**，非加工内容。
为什么：铁律"摘要不能替代 raw ref"（[case_judgment 02:52](BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md)、[ER 02:35](BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md)、[case_log 02:12](BK_Copilot/memory_layers/case_log/02_authority_lifecycle_and_boundaries.md)）禁止用不可追溯摘要替代 raw；让 ER 为会计预加工还会越其单一职责。重复读同一材料的成本被有意接受，换审计性 + 权威分离。"处理一次"只适用 EI 的客观交易结构（[Evidence Intake L2提案:45](L2/L2_proposals/Evidence%20Intake__L2提案.md)），不延伸到辅助证据。
文档：无改动（澄清既有不变量）；影响本轮 ER L3 = `identity_evidence_refs` 是 ref 非内容。

### D7 · ER 字段命名收敛（含 alias / raw_description / created_by / entity_id） — 2026-06-25
本轮 ER L3 锁定字段名：运行期表面文字全程 **`raw_description`**（沿用 EI 名，不漂移），无论 stable/unknown；**`alias` 不是运行期字段**，只是 stable·新建时 `raw_description` 写入 Alias Log / Entity Log 的存储记录名（alias 只配已 stable 的对象）。原 `creation_provenance` 简化为 **`created_by`**（`confirmed_by` 已被 Case Log 占用）。Entity 的输出值 = **`entity_id`**（stable 句柄，指向 Entity Log 记录，非复制本体）。`unknown_reason_context` 并入 **`identity_reason`**。
为什么："一个值一个名"——raw_description 不因状态改名；alias 作存储概念只在写 log 时出现，守"alias=身份 stable 之果"；命名能少则少。
文档：本轮 schema `L3_schema/ER_Schema/Entity_Resolution_L3_Schema.md` + `L3_schema/字段链路大表.md`（不改 ER `00/01/02`，冻结文档已与此一致）。
