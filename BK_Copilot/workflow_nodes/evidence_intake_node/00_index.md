# Evidence Intake Node

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
- Last reviewed: 2026-06-14
- Owner / reviewer: system audit session / user-reviewed L2 proposal

## 当前标准文件

- `01_functional_intent.md`
- `02_logic_and_boundaries.md`

## 当前未冻结边界

- Evidence Intake 的未冻结 L3 / L4 / seam / L2 external blocker / DEFERRED 项唯一权威池是根目录 `缺口地图.md` 的 Evidence Intake section；本文档不复制完整清单。

## 进入下一阶段前必须解决

- 字段级 schema 未冻结：`EvidenceFoundationHandoff`、`ObjectiveNormalizedTransactionBasis`、`evidence_refs`、`EvidenceQualityIssueSignal`、duplicate material reference、transaction-ready validation matrix 等 exact fields / required-optional / refs 形态必须先完成 L3。
- `direction` enum 必须与 Rule Match / JE 等下游统一。
- `source_channel` / `source_type` / `evidence_type` / `source_actor_type` enum 必须跨 runtime / onboarding / supplemental 来源对齐。
- `evidence_condition` schema 与 evidence sufficiency 语义归 Case / Rule 侧完成联合 L3；Evidence Intake 只输出证据存在性事实。
- Evidence Log 写入时点、写入执行者、finalization、失败回滚、snapshot / retrieval / hash / blob storage 机制尚未冻结。
- Evidence Log / Coordinator-Pending / Profile-Structural Match / Governance 等圈外对象的正式边界尚未冻结；本文仅挂起 consumer 引用，不替这些对象冻结 spec。
- EI 内部处理步骤的 exact 执行顺序 / routing sequence 与 parsing / OCR / LLM extraction algorithm 属 Stage 4-7，尚未冻结。
