# Transaction Log

## 文档状态

- Current standard status: draft
- Covered stages:
  - [x] M1: Memory Intent
  - [x] M2: Authority, Lifecycle, and Boundaries
  - [ ] M3: Data Contract
  - [ ] M4: Storage and Operations
  - [ ] M5: Tests and Fixtures
- Last reviewed: 2026-06-18
- Owner / reviewer: user-reviewed Transaction Log L2 proposal; implementation pass: Codex

## 当前标准文件

- `01_memory_intent.md`
- `02_authority_lifecycle_and_boundaries.md`

## 一句话定义

Transaction Log 是以 `transaction_id` 为主索引的逐笔交易审计 source of truth；记录每一笔进入系统的交易自进入起被各环节如何处理、最终变成什么的全过程；按 entity 查询全部历史交易是对这些逐笔记录的派生访问能力，不另设 entity 索引存储层。

## 当前未冻结边界

### L3（字段 / enum / schema）

- Transaction Log record 的 exact 字段名 / schema / validation，含本层已确认的 13 项保存语义的精确形态。
- reasoning 在 Transaction Log 的 exact 存储字段，以及后续治理节点读取 reasoning 的 contract。
- rule-hit source / rule ref 在 Transaction Log 的 exact 字段形态。
- 身份字段含 `unknown` 标识、情况 Y 身份待补线索的 exact 字段形态。
- terminal / blocked / 无 JE 交易最终状态的 exact enum / 字段。
- final outcome、COA、HST / GST、full processing path 各环节痕迹的 exact 字段结构。
- `confirmed_by` 审计留痕、AI 高置信度自动完成标记的 exact 字段。

### L4 / seam（机制）

- Transaction Log 的 exact writer、统一 finalization 写入机制、多 log finalization 顺序（Evidence / Transaction / Case / Rule / JE）。
- append-only 更正记录的 exact 机制：触发者、correction record schema、改 entity 后扇出到其名下交易的跨 log 传播，以及“谁改 + 改后最终分类结果”的 exact 留痕形态。
- reversal / split / supersession 对 Transaction Log 的执行机制。
- 统计派生 / rollup / cache / maintenance job。
- governance / 未来 agent 工具读取 Transaction Log 的 exact retrieval / projection / permission 机制。
- entity-index 查询的 exact 索引 / 检索机制。

### L2·外阻（圈外依赖）

- JE Generation Node 与 Transaction Log 的 exact handoff（JE result / journal_entry 留痕）。
- Intervention Log 与 Transaction Log 的分界 exact contract。
- Human Review Node review trace 进 Transaction Log 的 exact 边界（Human Review 是 finalized 后会计师人发起的纠错节点，不再是 final logging 前 gate）。
- Case Memory Update Node 凭 `transaction_log_ref` / finalization proof 写 Case Log 的 exact 机制与 trigger order。
- Transaction Log 与对外 final output report / export artifact 的 field-level 边界。
- Governance（授权确认 = Human Review + Finalization）、Knowledge Summary / Knowledge Compilation、Evidence Log、Profile / Structural Match 与 Transaction Log 的 exact 读写边界。（Post-Batch Lint 已废除，删除。）
- transaction correction 如何影响 Alias review / mutation、entity risk candidate。

### DEFERRED

- correction / reversal / supersession / split 的完整产品机制。本轮只锁 append-only 原则与更正追加语义。
- 跨导入 `transaction_id` 复用、迟到证据接回、durable transaction registry、跨运行账务幂等。

## 进入下一阶段前必须解决

- L3 字段定稿：record schema、reasoning / rule-hit ref、身份字段、terminal / blocked / 无 JE 状态、final outcome / COA / HST / processing path、`confirmed_by` 与 AI 高置信度自动完成标记。
- 圈外依赖正式落文：JE Generation、Intervention Log、Review、Case Memory Update、对外 final output report / export artifact。
- L4 / seam 机制在对应阶段单独推进：exact writer、统一 finalization 写入机制、多 log 顺序、读取 retrieval / projection / permission、更正追加机制与 entity-index 检索机制。
