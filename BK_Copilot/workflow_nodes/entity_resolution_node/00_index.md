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
- `same_batch_retrigger`（会计师补充确认后是否在同一批次重跑本节点）尚未冻结。
- `unconfirmed_role_routing`（角色/上下文未确认时由下游如何路由）尚未冻结。
- `knowledge_summary_conflict_repair`（Knowledge Summary 与 Entity Log / Governance Log 冲突时的修复流程）尚未冻结。

## 进入下一阶段前必须解决

- 决定 `entity_resolution_output`（实体识别运行时输出）中哪些字段属于 identity fact（身份事实），哪些只属于 downstream derived decision（下游派生判断）。
- 确认本节点不直接持久化 `candidate_signal`（候选信号），只通过 runtime handoff（运行时交接）交给下游。
- 决定是否需要 Stage 3: Data Contract。如果需要，必须先移除或下沉 `rule_eligibility`（规则匹配资格）、`case_context_eligibility`（案例上下文资格）和 `downstream_authority_summary`（下游权限摘要）这类下游派生字段。
