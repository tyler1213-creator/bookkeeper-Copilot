# Case Judgment Node

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

- `case_judgment_input` / `case_judgment_result` exact field schema、status enum 仍未冻结，归 Stage 3。
- accounting treatment 字段结构、COA / HST / GST 表达方式、refs 形态，以及它与 JE Generation Node 的接口对齐仍未冻结，归 Stage 3 / JE Generation 联合收口。
- Pending handoff / `pending_request_context` 的 exact field schema 仍未冻结，归 Stage 3；Coordinator / Pending Node 的提问模板、追问策略和交互机制仍属 L2 外阻 / seam。
- Entity Log `automation_policy`、Case Log `use_level` / `confirmed_by` 的 exact enum 取值仍未冻结，分别归 Entity Log / Case Log L3。
- 会导致 CJ 自动转 Pending 的硬阻断已经有语义清单：ER `unknown`、Entity Log `automation_policy` 收紧、Entity Log 身份级 risk flags、Case Log `use_level` 必审 / 不可复用 / 失效；但这些阻断尚未结构化为具体字段 / enum，归 L3。
- 固定加载知识的注入、缓存、retrieve 机制，以及单笔上下文的排序 / 拼装机制仍未冻结，归 L4 / seam。
- Case Log per-entity rollup / retrieval pack 的生成机制仍未冻结，归 Case Log 读取层 L4 / seam。
- Case Log 先例摘要的 exact 字段 / 呈现形态、次数 / 确认数 / 金额容差等阈值仍未冻结，归 L3；rollup 如何从逐条 case 派生仍归 L4 / seam。
- entity-level Knowledge Summary 尚无正式草案；CJ 只能把它作为未锁的 readable context / co-read 假设，具体同步 co-read 哪些 source memory 属 L2 外阻 / seam。
- Profile / Structural Match 尚无正式草案；CJ 只能挂起引用其结构性路径假设，identity-independent 交易由其处理的合同仍待该对象正式审计确认。
- JE Generation Node 尚无正式草案；CJ 只声明 High Confidence Classification 需要携带确定的 (COA, HST/GST) 决定，JE 是否以及如何纯确定性构造仍待该对象正式审计确认。
- High Confidence 进 JE 前的 COA 校验 L2 行为已定：未命中客户 COA 则反馈回 CJ 重判，仍失败则转 Pending；但校验落点（CJ 末尾独立检查器 vs 并入 JE Generation Node）、重判次数上限、反馈 payload 和纠错 UX 仍归 L4 / seam。
- CJ external lookup 只用于会计事实，不用于身份；外部来源 evidence reference 的 exact field schema、网页快照 / 缓存 / retrieval 机制、source quality taxonomy 仍归 L3 / L4，并需与 ER 侧纪律对齐。
- reasoning / audit trace 应进入 Transaction Log，但 exact 存储字段、后续治理节点读取 reasoning 的 contract、Transaction Log 写入机制仍未冻结，归 L3 / L4 / 外阻。
- `case_memory_update_candidate` 的 exact field schema、Case Memory Update Node 的写入机制、Case Log 与 Transaction Log 的 finalization trigger order 仍未冻结，归 L3 / L4 / seam。
- High Confidence 结果的事后人工查看、汇总界面、抽查机制仍未冻结，归 UI / 后续设计。
- 批次定义（如单张 Bank Statement）和一次提交多张表的串行 / 并行快照排序仍未冻结，归 Coordinator / 编排层。
- 结构性无效 / 损坏输入的拒绝与否、被 CJ 与上游均无解交易的最终归处、纠错 candidate signal 的发出者与纠错流程均为 DEFERRED。
- 交易拆分（split）的检测与执行不属于 CJ；拆分相关检测、执行、子交易重入路径仍为 DEFERRED / Coordinator 或上游外阻。
- Coordinator、Knowledge Summary、Profile / Structural Match、JE Generation、Evidence Log、Transaction Log、Governance 等圈外对象的正式草案仍未全部落定；CJ 中对这些对象的引用不得当冻结合同。

## 进入下一阶段前必须解决

- C1：Accountant / Governance authority boundary 已在 `02_logic_and_boundaries.md` §12 单列成文；进入 Stage 3 前需由 reviewer 确认其与 Entity Log / Case Log / Rule Log / Governance 后续草案无冲突。
- C2：Stage 1 Functional Intent 已在 `01_functional_intent.md` 成文；进入 Stage 3 前需由 reviewer 确认 CJ 的唯一职责、排除范围和 workflow 位置无需新增 L2 决策。
- C3：CJ 拥有 COA + HST / GST treatment 选择权已在 `01_functional_intent.md` §2 和 `02_logic_and_boundaries.md` §5 / §6 成文；进入 Stage 3 前需与 JE Generation 的纯构造边界联合确认。
- 完成 `case_judgment_input` / `case_judgment_result` / Pending handoff / case memory update candidate 的字段级 L3 schema。
- 完成硬阻断字段、automation policy / use_level / confirmed_by enum、accounting treatment schema、external evidence ref、reasoning trace、COA 校验反馈 payload 的 L3 / L4 分工。
- Coordinator / Pending Node、Case Memory Update Node、JE Generation Node、entity-level Knowledge Summary、Profile / Structural Match 的正式边界至少达到能支撑 CJ 接口收口的程度。
