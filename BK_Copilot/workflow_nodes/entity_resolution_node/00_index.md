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

- `stable_entity_resolution_threshold`（稳定实体识别所需证据门槛）尚未冻结。
- `entity_resolution_output`（实体识别运行时输出）的 exact field schema（精确字段结构）尚未冻结。
- `new_stable_entity` 最小创建 provenance 的 exact field schema（精确字段结构）尚未冻结。
- `knowledge_summary_conflict_repair`（Knowledge Summary 与 Entity Log / Governance Log 冲突时的修复流程）尚未冻结。

## 当前已确认边界

- Alias（别名）是过去已经确认过的 transaction surface text 和 stable entity（稳定实体）的对应关系。
- 当前只确认两类信息可能成为 Alias：bank statement 中每笔交易的 description / descriptor / raw bank surface text；以及当 bank description 本身没有明确身份意义时，其他可能重复出现并能指向交易主体的字段，例如 cheque payee。
- 当前确认需要一个可被 Entity Resolution 查询的 Alias 库，用于从当前交易的 surface text 反查过去已经确认过的 entity；Alias 库具体技术形态尚未冻结。
- Entity Resolution 可以使用 AI 联网搜索辅助 identity 判断；搜索结果只是 evidence 来源，不是 authority，也不用于会计分类判断。
- Entity Resolution 判断为 `new_stable_entity`（新稳定实体）时，可以同步写入 Entity Log，不需要 governance approval（治理批准）；写入内容限于 entity 本体和最小创建 provenance，不写 Alias（别名）、automation policy（自动化策略）或 rule（规则）。
- unknown entity 后由 accountant 明确确认 identity 时，交易不重新进入 Entity Resolution；accountant confirmation（会计师确认）替代本节点的身份判断，后续创建 stable entity 和分类完成由下游路径处理。
- unknown / unresolved identity 输出可以携带 identity clues、ambiguity reason、missing evidence reason、搜索线索和 evidence refs 作为 runtime context，但这些信息不是 stable entity、candidate entity 或 identity authority。

## 进入下一阶段前必须解决

- 决定 `entity_resolution_output`（实体识别运行时输出）中哪些字段属于 identity fact（身份事实），哪些只属于 downstream derived decision（下游派生判断）。
- 确认本节点除 `new_stable_entity` entity 本体同步写入外，不直接持久化 `candidate_signal`（候选信号），只通过 runtime handoff（运行时交接）交给下游。
- 决定是否需要 Stage 3: Data Contract。如果需要，必须先移除或下沉 `rule_eligibility`（规则匹配资格）、`case_context_eligibility`（案例上下文资格）和 `downstream_authority_summary`（下游权限摘要）这类下游派生字段。
