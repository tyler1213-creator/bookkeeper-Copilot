# Entity Resolution Node

## 文档状态

- Current standard status: draft
- Covered stages:
  - [x] Stage 1: Functional Intent
  - [x] Stage 2: Logic and Boundaries
  - [ ] Stage 3: Data Contract
  - [ ] Stage 4: Execution Algorithm
  - [ ] Stage 5: Technical Implementation
  - [ ] Stage 6: Tests and Fixtures
  - [ ] Stage 7: Agent Task Contract
  - [ ] Stage 8: Maintenance
- Last reviewed: 2026-05-10
- Owner / reviewer: system audit session / user review required

## 当前标准文件

- `01_functional_intent.md`
- `02_logic_and_boundaries.md`

## 当前未冻结边界

- stable identity（稳定身份）的语义判据已冻结；如后续需要 exact numeric threshold（精确数字门槛）、confidence cutoff（置信度切线）或评分规则，属于 L3，不在本阶段冻结。
- `entity_resolution_output`（实体识别运行时输出）的 exact field schema（精确字段结构）尚未冻结。
- `new_stable_entity` 最小创建 provenance 的 exact field schema（精确字段结构）尚未冻结。
- external evidence reference（外部证据引用）的 exact field schema（例如 URL / source title / retrieved_at / snippet 等）尚未冻结。
- ER 判定新建 stable entity 后，Entity Log + Alias Log 同步 finalization 的实际写入执行者、调用方式和写入顺序属于 L4 / seam，尚未冻结。
- `knowledge_summary_conflict_repair`（Knowledge Summary 与 Entity Log / Governance Log 冲突时的修复流程）尚未冻结。

## 当前已确认边界

- Entity Resolution 对外只输出两种 identity state（身份状态）：`stable` 与 `unknown`。ambiguous、conflict、unresolved、missing evidence、alias issue 等只能作为 `unknown` 的 reason / context，不能构成第三态、candidate entity、Case Log handle 或 Rule Match basis。
- stable identity（稳定身份）的 L2 语义判据是：当前所有可追溯 evidence 能够直接、清晰且无歧义地说明交易主体是谁。该判断只限 identity，不包含 COA、HST / GST、业务用途、Rule 或 automation permission。
- stable 输出必须带可审计 identity reason，说明依据哪些 evidence points 判定主体明确；reason 不是会计分类、Rule authority、Case authority、accountant approval 或 governance approval。
- Alias（别名）是过去已经确认过的 transaction surface text 和 stable entity（稳定实体）的对应关系。
- 当前只确认两类信息可能成为 Alias：bank statement 中每笔交易的 description / descriptor / raw bank surface text；以及当 bank description 本身没有明确身份意义时，其他可能重复出现并能指向交易主体的字段，例如 cheque payee。
- 当前确认需要一个可被 Entity Resolution 查询的 Alias 库，用于从当前交易的 surface text 反查过去已经确认过的 entity；Alias 库具体技术形态尚未冻结。
- exact Alias match 是确定性身份复用路径：当前 transaction surface text 完全命中 Alias Log / Alias 库中既有 Alias 时，Entity Resolution 可以直接输出该 Alias 指向的 `stable` entity。
- Entity Resolution 可以使用 AI 联网搜索辅助 identity 判断；搜索结果列表和搜索摘要只能作为 clue。由搜索打开并可追溯的外部来源，在与当前交易 evidence 共同指向清晰、唯一、无实质竞争的业务对象时，可以作为 stable reason 的 evidence point；该 evidence point 只支持 identity，不支持会计分类、Rule 或自动化许可。
- Entity Resolution 判定新建 stable entity 时，state 仍为 `stable`，provenance 语义为新建。ER 自主决定、无需 governance approval，并必须在当前交易进入下游 judgment / pending 前发起同步 Entity Log + Alias Log finalization：Entity Log 创建 stable entity 本体、最小创建 provenance 和初始 entity-centered Alias surface，Alias Log 写入该 confirmed surface text -> stable entity 的反查 projection；实际由 ER 亲手写入还是同步调用专门写入 / 存储机制，留 L4 / seam。
- `unknown` 输出后由 accountant 明确确认 identity 时，交易不重新进入 Entity Resolution；accountant confirmation（会计师确认）替代本节点的身份判断，后续创建 stable entity 和分类完成由下游路径处理。
- `unknown` 输出可以携带 identity clues、ambiguity reason、missing evidence reason、搜索线索和 evidence refs 作为 runtime context，但这些信息不是 stable entity 或 identity authority。
- 本节点对外输出收敛为身份结果本身：identity state（`stable` / `unknown`）、stable 时的 entity 标识、identity reason、evidence refs、`unknown` 的 reason / context；**不再输出 `candidate_signal` / `merge_split_candidate` / `alias_conflict_issue` / `identity_governance_issue` / `blocking_reason` / `identity_risk_flags` 等并行候选 / 风险通道（2026-06-25 收口删除）**。理由：这些输出无下游消费者、与 `unknown` reason 重复、或超出本节点能力 / 职责。merge / split 归会计师人发起 Human Review + Entity Log；identity risk flags 归 Entity Log（本节点只读）；身份卡点与冲突一律经 `unknown` + reason 表达。

## 进入下一阶段前必须解决

- 决定 `entity_resolution_output`（实体识别运行时输出）中哪些字段属于 identity fact（身份事实），哪些只属于 downstream derived decision（下游派生判断）。
- 本节点除新建 stable entity 的同步 Entity Log + Alias Log finalization 外，不产生任何 durable write，也不产生与身份状态并行的候选 / 风险通道；身份卡点只经 `unknown` + reason 表达（原 `candidate_signal` 等已删除，见上）。
- 决定是否需要 Stage 3: Data Contract。如果需要，必须先移除或下沉 `rule_eligibility`（规则匹配资格）、`case_context_eligibility`（案例上下文资格）和 `downstream_authority_summary`（下游权限摘要）这类下游派生字段。
