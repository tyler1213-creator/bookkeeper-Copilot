# Rule Match Node

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

- automation_policy 中“哪种取值 = 放行 rule-based automation”的 exact 取值集合 / 语义尚未冻结；归 L3，需 RuleLog / EntityLog 联合对齐 Governance。
- RuleMatch 输入 handoff 的 exact 字段 schema 尚未冻结；归 L3。handoff 由谁组装属于 L4 / seam。
- RuleLog reader 的调用机制属于 L4 / seam。RuleLog 必须支持按 `entity_id` 索引，且每条 rule 承载 applicability；这是对 RuleLog 的前置约束，待 RuleLog 设计落定。
- rule 资格判据的最终归档之家尚未冻结，取决于 RuleLog M1-M2；本节点只声明自己消费该资格语义标准。
- rule 集合 overlap-validation 算法属于 L4 / seam；catch-all pattern 是否作为 schema 语法糖属于 L3。
- entity-level / pattern-level rule 的 exact schema、condition enum、accounting treatment（judgment-free 完整）schema、资格阈值具体数字尚未冻结，归 L3 / JE Generator。
- JE Generator 接口尚未冻结；treatment “judgment-free 完整”的判据依赖 JE Generator 需要什么，属于跨节点 seam。

## 进入下一阶段前必须解决

- RuleLog M1-M2 落定：资格判据之家、`entity_id` 索引语义、rule payload authority 和 rule lifecycle 边界。
- automation_policy 取值集合和放行 rule-based automation 的 exact 语义完成联合 L3，并与 Governance 对齐。
- RuleMatch handoff exact schema 完成 L3。
- JE Generator 接口口径完成联合收口，说明 approved accounting treatment 的可执行完整性需要哪些语义。
