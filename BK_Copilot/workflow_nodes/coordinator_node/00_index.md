# Coordinator Node

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
- Last reviewed:
- Owner / reviewer:

## 当前标准文件

- `01_functional_intent.md`
- `02_logic_and_boundaries.md`

## 当前未冻结边界

- L3：CJ Pending handoff / `pending_request_context` exact schema、pending 子类型 enum、Coordinator question 模板、accountant 回答结构、情况 X / Y 落点字段、带选项 pending 问题结构、身份确认 provenance、Case Log 学习记录字段 / enum、各类 governance / review candidate schema、转 JE 所需格式 contract。
- L4 / seam：Entity Log + Alias Log 统一写入执行机制、Case Log 学习记录 exact writer / trigger order、多 log finalization、stable entity 下分类问题聚合呈现与逐笔确认机制、多表排序 / 批起始快照、受限重跑边界、提问话术 / 编排、身份缺口后续 triage、人机对话通道与多节点提问呈现。
- L2·外阻：JE Generation、Transaction Log、Intervention Log、Governance（授权确认 = Human Review + Finalization）、Profile / Structural Match、Knowledge Summary、Human Review Node、interaction_agent、Onboarding 等圈外对象正式边界。
- 以上延后项详情见 `02_logic_and_boundaries.md` §12。延后项的 `缺口地图.md` Coordinator section 由用户单独维护；本节点正式草案不写回缺口地图。

## 进入下一阶段前必须解决

- JE Generation、Transaction Log、Intervention Log 三个圈外下游完成足以支撑 Coordinator 契约收口的正式落文。
- L3 字段 / enum / 阈值完成联合收口，尤其 Pending handoff、问题 / 回答结构、转 JE 所需格式、Case Log 学习记录语义和身份确认 provenance。
- L4 / seam 机制完成联合收口，尤其写入机制、聚合呈现与逐笔确认、扇出边界、多表排序和多 log finalization。
- 上述前置未完成前，本节点不得进入 Stage 3 data contract、Stage 4 execution algorithm 或 implementation。
