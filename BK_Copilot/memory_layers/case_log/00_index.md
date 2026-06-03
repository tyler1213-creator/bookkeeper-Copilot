# Case Log

## 文档状态

- Current standard status: draft
- Covered stages:
  - [x] M1: Memory Intent
  - [x] M2: Authority, Lifecycle, and Boundaries
  - [ ] M3: Data Contract
  - [ ] M4: Storage and Operations
  - [ ] M5: Tests and Fixtures
- Last reviewed: 2026-06-02
- Owner / reviewer: system audit session / user review required

## 一句话定义

`Case Log`（案例日志）是以 `entity_id`（实体唯一标识）为主要索引的 completed-case learning memory（已完成案例学习记忆）：它从已完成交易中保存可复用的 bookkeeping precedent（记账先例）、evidence condition（证据条件）、exception context（例外上下文）和 correction reference（纠正引用），用于未来 case-based judgment（基于案例判断）、rule promotion candidate（规则升级候选）、automation risk review（自动化风险审核）和 entity risk update candidate（实体风险更新候选）。

它不是 transaction process log（交易处理过程日志），不是 audit-facing final transaction record（面向审计的最终交易记录），也不是 entity identity authority（实体身份权威）或 active rule authority（生效规则权威）。

## 核心来源

本 spec 的 Case Log 权威信息只来自：

- `BK_Copilot/memory_layers/case_log_boundary_note.md`
- `BK_Copilot/memory_layers/entity_log/`
- `BK_Copilot/memory_layers/alias_log/`
- `BK_Copilot/workflow_nodes/entity_resolution_node/`
- `Entity相关的所有问题.md`
- `未解决问题暂存清单.md` 中已记录的 Entity / Case Log 相关问题

本 spec 不把 `new system/` 或 `old_system_nodedesign/` 作为 Case Log 权威来源。

## 当前标准文件

- `01_memory_intent.md`
- `02_authority_lifecycle_and_boundaries.md`

## 当前已冻结结论

- `Case Log` 是 entity-indexed completed-case learning memory（按实体索引的已完成案例学习记忆）。
- `Case Log` 只从 completed transaction（已完成交易）中抽取可学习案例；它必须能引用 `transaction_log_ref`（交易日志引用）或等价 finalization proof（完成证明）。
- `Case Log` 不替代 `Transaction Log` 的 final audit record（最终审计记录），也不得复制完整 processing path（处理路径）作为学习依据。
- `Case Log` 不保存 entity identity authority（实体身份权威）、Alias authority（别名权威）、confirmed role（已确认角色）、automation policy（自动化策略）或 active rule（生效规则）。
- stable-linked case（稳定实体关联案例）可以作为较强的未来 case context（案例上下文）。
- candidate-linked case（候选实体关联案例）默认只能作为 weak context（弱上下文）或 governance evidence（治理依据）。
- unknown entity（未知实体）不是 durable identity handle（长期身份句柄），不能被 Case Log 当作 entity handle 引用。
- `Case Log` 可以为 entity-level risk（实体级风险）、automation policy candidate（自动化策略候选）或 rule promotion candidate（规则升级候选）提供依据，但不能直接修改 `Entity Log`、`Rule Log` 或 `Governance Log`。
- repeated outcome（重复结果）不能自动升级为 approved rule（已批准规则）。

## 当前未冻结边界

- 哪些 completed transaction 具备 case write eligibility（案例写入资格）尚未冻结。
- duplicate / corrected / reversed / split transaction（重复 / 纠正 / 冲销 / 拆分交易）如何影响 case supersession（案例替代关系）尚未冻结。
- candidate-linked case 在 candidate 后来升级为 stable entity 后是否自动变强，尚未冻结。
- `new_entity_candidate` 或 candidate identity signal（候选身份信号）的持久化位置、namespace 和 Case Log 引用方式尚未冻结。
- `Case Log` 与 `Transaction Log` 的 exact trigger order（精确触发顺序）尚未冻结。
- 交易完成分类后的多 log 统一写入机制尚未冻结。
- 哪些 case-derived signals（案例衍生信号）可以进入 governance candidate（治理候选），以及具体审批路径尚未冻结。

## 进入下一阶段前必须解决

- 确定 Case Log 的最小 write eligibility（写入资格）规则。
- 确定 completed case、corrected case、superseded case 和 candidate-linked case 的最小 lifecycle / supersession 语义。
- 确定 Case Log 写入者：Case Memory Update Node、统一 finalization / memory write mechanism，或其他机制。
- 确定 `Transaction Log` finalization proof 与 Case Log 写入的触发顺序。
- 确定 candidate-linked case 是否、何时、如何转为 strong precedent（强先例）。
- 在字段、消费者、mutation authority 稳定前，不得创建 `03_data_contract.md`。
