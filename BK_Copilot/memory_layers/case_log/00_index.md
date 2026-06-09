# Case Log

## 文档状态

- Current standard status: draft
- Covered stages:
  - [x] M1: Memory Intent
  - [x] M2: Authority, Lifecycle, and Boundaries
  - [ ] M3: Data Contract
  - [ ] M4: Storage and Operations
  - [ ] M5: Tests and Fixtures
- Last reviewed: 2026-06-08
- Owner / reviewer: system audit session / user reviewed L2 proposal

## 一句话定义

`Case Log`（案例日志）是以 `entity_id`（实体唯一标识）为主索引的 entity-indexed reusable-precedent layer（已完成案例可复用先例层）：它从 stable entity（稳定实体）关联且已 finalized（完成）的交易中保存逐笔可复用先例，服务未来 case-based judgment（基于案例判断）、rule promotion 评估、automation / entity risk 识别。

它不是 `Transaction Log`（交易日志）的 entity 视图副本，不承担逐笔审计完整性；按 entity 查询全部历史交易属于 Transaction Log 的 entity 索引查询能力。它也不是 transaction process log（交易处理过程日志）、audit-facing final transaction record（面向审计的最终交易记录）、entity identity authority（实体身份权威）或 active rule authority（生效规则权威）。

## 核心来源

本 spec 的 Case Log 权威信息只来自：

- `bookkeeper-Copilot/L2_proposals/Case Log__L2提案.md`
- `BK_Copilot/memory_layers/case_log_boundary_note.md`
- `BK_Copilot/memory_layers/entity_log/`
- `BK_Copilot/memory_layers/alias_log/`
- `BK_Copilot/workflow_nodes/entity_resolution_node/`
- `Entity相关的所有问题.md`
- `Coordinator Question.md`

本 spec 不把 `new system/` 或 `old_system_nodedesign/` 作为 Case Log 权威来源。

## 当前标准文件

- `01_memory_intent.md`
- `02_authority_lifecycle_and_boundaries.md`

## 当前已冻结结论

- `Case Log` 是 entity-indexed reusable-precedent layer（按实体索引的可复用先例层），不是 `Transaction Log` 的 entity 副本。
- 所有已关联 stable entity 且已 finalized 的交易均具备 Case Log 写入资格；系统不在写入时预判未来学习价值。
- `Case Log` 的真值单位是逐笔已完成案例先例；某 entity 下的模式聚合、分布、次数、一致性和最近出现等都是读取层派生视图，不作为 Case Log 存储真值。
- `Case Log` 必须引用 `transaction_log_ref`（交易日志引用）或等价 finalization proof（完成证明）。
- `Transaction Log` 是 final 交易事实、完整 processing path、修改历史和 audit trail 的 source of truth；`Case Log` 只保存最新可学习状态，不替代审计记录。
- `Case Log` 不保存 entity identity authority（实体身份权威）、Alias authority（别名权威）、automation policy（自动化策略）或 active rule（生效规则）。
- unknown entity（未知实体）不是 durable identity handle（长期身份句柄）；unknown entity 交易即使被 accountant 完成分类并 finalize，也只进入 `Transaction Log`，不写 `Case Log` / `Entity Log`，且不得创建以 description / 类别为索引的学习记录。
- 每条 case 必须具备复用权限语义（`use_level`，exact enum 留 M3），表达该案例未来可作为正面先例、仅例外上下文、反面 / 冲突案例、不可升级为 rule，或已失效 / superseded。
- 最小 supersession L2 语义已冻结为复用安全：被纠正 / 冲销 / 治理限制后失效的旧 case 不得被未来节点静默当作有效正面先例复用；血缘链接和执行机制不在 M1-M2 冻结。
- `Case Log` 可以为 rule promotion、automation risk review、entity risk update、policy review 提供历史案例依据，但不直接创建 / 升级 / 修改 / 降级 active rule，不修改 `Entity Log`，不改 `Governance Log`。
- merge / split / archive 后旧 case 的归属由治理层裁定，`Case Log` 不裁判但必须服从治理结果。

## 当前未冻结边界

- Case Log record 的 exact 字段名、enum、schema、validation 尚未冻结。
- `use_level` / `confirm_by` 的 exact enum 取值尚未冻结。
- 共享 accounting outcome（COA / HST / split / allocation）的全局字段结构尚未冻结。
- Case Judgment 读取时的 per-entity 固定化聚合、rollup、轻量展示层、检索工具或 subagent 属于 L4 / seam-park，尚未冻结。
- Case Log 的 exact writer、与 Transaction Log 的 trigger order、多 log 统一 finalization 机制尚未冻结。
- supersession 的血缘链接字段与 corrected / reversed / split 后的重判执行机制尚未冻结。
- merge / split / archive 后旧 case 重新归属的治理执行机制尚未冻结。
- case-derived rule / automation / entity risk candidate 进入治理的 exact contract 依赖 Governance / Rule Log，未在本层冻结。
- entity / 类别级通则批注的归属依赖 Knowledge Summary / Governance，未在本层冻结。

## 进入下一阶段前必须解决

- 冻结 record schema、字段名、enum 和 validation。
- 冻结 `use_level` / `confirm_by` enum。
- 与 Rule Match / Case Judgment / Transaction Log / JE 相关对象对齐共享 accounting outcome 结构。
- 冻结 Case Log 写入者、与 Transaction Log 的 trigger order，以及多 log 统一 finalization 机制。
- 冻结 Case Judgment 读取 Case Log 时的 rollup / retrieval 机制。
