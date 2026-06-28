# Rule Log

## 文档状态

- Current standard status: draft
- Covered stages:
  - [x] M1: Memory Intent
  - [x] M2: Authority, Lifecycle, and Boundaries
  - [ ] M3: Data Contract
  - [ ] M4: Storage and Operations
  - [ ] M5: Tests and Fixtures
- Last reviewed:
- Owner / reviewer:

## 一句话定义

Rule Log 是以 `entity_id` 为主索引的 executable rule authority layer：它保存 RuleMatch 可以执行的 approved deterministic rules，包括 rule applicability、approved accounting treatment、scope 类型、当前执行状态语义和来源 / 批准 provenance。每条 rule 带一个**稳定、唯一的 `rule_id`**（贯穿生命周期不变，作引用 / 审计 handle，见 Decisions D9），且只保存**当前可执行状态**、不存版本历史（变更历史归 Governance Log）。Rule Log 不保存 rule candidate、case evidence、transaction audit trail、EntityLog 控制状态字段（`force_pending` / `promotion_lock`）或非可执行会计建议。

## 当前标准文件

- `01_memory_intent.md`
- `02_authority_lifecycle_and_boundaries.md`

## 当前未冻结边界

- **L3**：rule record exact 字段名、`rule_id` 的 exact 字符串 / 格式、scope enum、current execution state 字段名 / enum、source / approval provenance refs 形态。（`rule_id` 的**存在与角色已定**，见 Decisions D9：稳定唯一、贯穿生命周期、作引用 / 审计 handle；此处仅 exact 格式留 M3。）
- **L3**：entity-level / pattern-level rule 的 exact schema、condition enum、客观条件表达、catch-all pattern 语法糖。
- **L3 / L4**：overlap-validation 算法、condition satisfiability 检查、amount range validation、冲突 repair path。
- **L3 / JE Generator**：approved accounting treatment 的 exact shape、COA / HST / GST / split / allocation schema、JE Generator contract 与 validation rule。
- **L3，EntityLog / RuleLog / Governance 联合**：force_pending 取值如何在上游 eligibility router 决定交易是否放行进入 rule-based automation / RuleMatch（命中即改道、不进 RuleMatch）。
- **L4 / seam**：RuleLog 执行面 query contract、reader / assembler 调用顺序、projection / cache / permission 边界。
- **L4 / seam**：RuleLog 按 `entity_id` 索引的存储结构、检索、查询优化和索引维护机制。
- **L2·外阻，Governance / review path**：rule promotion 发现侧（确定性发现 job 只套用 Rule 侧判句）、rule candidate queue / 审核 inbox、system-proposed promotion request、CaseLogEvidence 打包格式、accountant approval workflow。
  > 关于 rule 升级（promotion）发现侧的扫描 / 候选产出机制，参见 `独立question文档/Deterministic_Discovery_question.md`（确定性发现机制当前唯一权威文档）；promotion 资格判句的定义权仍归本（Rule）侧，确定性发现只套用。
- **L4 / Governance seam**：accountant direct rule creation 的 exact write mechanism、approval capture、写入执行者。
- **L2·外阻 / L4，Governance / 记录层**：rule modification / overwrite / versioning / supersession / deletion / restore 的 exact mutation path。
- **L3 / Governance**：跨 log lifecycle 字段命名统一，包括暂停、废除、替代、删除、归档等。
- **L3，联合 Transaction Log / Case Log**：rule-hit source / rule ref 在 Transaction Log / Case Log 中的 exact 字段形态。（**已定**，见 Decisions D9：rule-hit 引用 = `rule_id` 裸值、不钉版本；历史版本还原靠 `rule_id` + 交易时间 + Governance Log 回放。仅 exact 字段形态留 L3。）
- **L4 / seam**：match_count、last_hit、health / staleness 派生统计、rollup、cache、maintenance job。
- **L4 / Governance**：merge / split 后 rule 重判、批量 mutation、迁移 / 阻断 / 恢复执行、回滚和治理 trace。

## 进入下一阶段前必须解决

- rule schema、rule ref 形态、scope enum、condition enum、current execution state 命名未冻结前，不进入 M3 data contract。
- approved accounting treatment exact shape 与 JE Generator contract 未冻结前，不进入 M3 data contract。
- force_pending exact enum 以及哪种取值在上游 router 放行 / 改道未由 EntityLog / RuleLog / Governance 联合冻结前，不进入 M3 data contract。
- rule mutation path、writer exact contract、reader query contract、projection / cache / permission 边界未冻结前，不进入 M3 / M4。
- promotion 发现侧判句、审核 inbox、固定执行路径、CaseLogEvidence 打包格式、accountant approval workflow、accountant direct write 机制、merge / split 后 rule 重判机制未由 Rule 侧 / Governance / L4 seam 收口前，不进入实现。
  > 关于 rule 升级（promotion）发现侧的扫描 / 候选产出机制，参见 `独立question文档/Deterministic_Discovery_question.md`（确定性发现机制当前唯一权威文档）；promotion 资格判句的定义权仍归本（Rule）侧，确定性发现只套用。
