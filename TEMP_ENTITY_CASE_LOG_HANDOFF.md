# Entity / Case Log 临时交接记录

## 文件角色

这是当前窗口的临时讨论交接，不是正式 spec，不是 `DECISIONS.md`，也不是当前系统设计合同。

它只记录：

- 当前窗口已经暂时确认的判断。
- 这些判断背后的第一性原理。
- 后续 compact 后最该继续讨论的问题。
- 哪些内容还不能写成 M3 data contract 或实现规则。

正式落地前，仍应根据用户确认结果更新对应标准文档，例如：

- `BK_Copilot/memory_layers/entity_log/01_memory_intent.md`
- `BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md`
- 后续可能新增的 `Case Log` M1-M2 文档

## 当前讨论范围

本窗口围绕以下问题展开：

1. `Case Log` 是否应该存在，以及它和 `Transaction Log` / `Entity Log` 的边界。
2. `Case Log` 是否需要保存 candidate entity 相关字段。
3. `Entity Log` 是否应允许 durable `candidate` entity record。
4. `candidate entity` 到 `stable / active entity` 的升级规则。
5. 旧系统 `confidence classifier` / `Observations` 中哪些机制可以翻译进新系统。
6. 第一次出现的新交易到底由哪个节点判断它是谁。

## 已暂时确认的判断

### 1. Case Log 应该存在，但必须收窄

`Case Log` 应存在，因为系统需要把历史完成交易和 accountant 判断沉淀为可复用案例先例。

它的核心职责是：

- 保存 completed-case learning memory。
- 以 entity 身份上下文为主索引。
- 保存可复用 bookkeeping precedent、evidence condition、exception context、correction reference。
- 支持 `Case Judgment Node`、`Review Node`、`Post-Batch Lint Node`、`Governance Review Node` 和 `Knowledge Compilation Node`。

它不应该变成第二份 `Transaction Log`。

它不保存：

- 完整 processing path。
- runtime trace。
- active rule。
- entity authority。
- unapproved AI reasoning。
- governance approval。

### 2. Case Log 不负责审批或推动升级

已澄清：用户并不主张 `Case Log` 审批或主动推动 entity / alias / role / rule 升级。

`Case Log` 只存信息、提供信息。

长期 authority mutation 应由对应 authority layer 或 governance path 处理，例如：

- `Entity Log` 保存 entity identity authority。
- `Rule Log` 保存 approved deterministic rules。
- `Governance Log` 保存高权限变化和审批历史。
- `Review / Governance Review` 处理批准、拒绝、合并、拆分、策略变化。

### 3. Entity 取代 pattern 成为主索引，但不是简单等价

旧系统中 `pattern` 是 Observations 的主索引。

新系统中 `entity` 承担类似主索引角色，但它比 `pattern` 多了 authority lifecycle：

- `candidate entity`
- `active / stable entity`
- `merged`
- `archived`

因此，不能把 `new_entity_candidate` 直接当成 stable entity authority。

但用户的核心直觉被暂时接受：

> 新系统中 Case Log 不应在自身内部保存大量 candidate entity 字段；如果有 entity identity handle，Case Log 应主要引用它。

### 4. 接受 durable candidate entity record

当前暂时接受：

> `Entity Log` 可以存在 durable `candidate` entity record，并允许 `Case Log` 引用它。

但含义必须严格限定：

- `candidate entity record` 是 durable identity handle。
- 它不是 stable identity authority。
- 它可以被 `Case Log` 引用。
- 它可以挂 evidence refs、surface text、candidate context 和 weak case context。
- 它不能支持 rule match。
- 它不能支持 rule promotion。
- 它不能承载 approved alias 或 confirmed role。
- 它不能放宽 automation policy。
- 它不能被下游当成 active / stable entity。

如果不允许 durable candidate entity，Case Log 会被迫保存身份快照，边界会更乱。

### 5. Stable entity 的作用是身份权威，不是记账权威

`stable / active entity` 只回答：

> 这是谁，未来交易能否安全归到这个长期对象下面。

它不回答：

- 应该进哪个 COA。
- HST / GST 怎么处理。
- journal entry 怎么生成。
- 是否可以自动落账。
- 是否已有 deterministic rule。

因此，`stable entity` 不是低级 rule，也不是“rule entity”。

更清晰的说法应是：

- `candidate entity`
- `active / stable entity`
- `entity with approved rule`
- `entity automation_policy = eligible / case_allowed_but_no_promotion / rule_required / review_required / disabled`

### 6. Candidate -> Stable 的规则应大幅简化

之前提出的 `risk pack / override evidence / policy_trace` 式规则过重，属于对旧系统高置信分类机制的过度迁移。

第一性原理下，candidate -> stable 只应判断：

> 当前 evidence 中是否存在清晰、可追溯、可人类验证的业务对象身份信号，并且没有明显身份冲突。

升级为 stable entity 只授予身份复用权，不授予分类、规则或自动化权限。

最小成立条件：

1. 有直接身份信号。
   - receipt vendor
   - invoice vendor
   - cheque payee
   - contract party
   - 清晰 bank descriptor
   - accountant-processed historical book 中的 vendor/payee
   - accountant 明确确认
2. 该身份信号能让人直接看出对象是谁。
   - 例如 `ROGERS WIRELESS`、`HOME DEPOT #4521`、`Amazon.ca`、`John Smith Plumbing`
3. 没有明显身份冲突。
   - 不撞上多个已有 active entity。
   - 不命中 rejected alias。
   - 不明显属于内部转账 / payroll / loan / tax authority 等结构性对象但 profile 未确认。

不需要证明：

- 这笔交易业务用途是什么。
- COA 是否唯一。
- HST 是否可确定。
- 是否可以自动分类。
- 是否可以升级为 rule。

### 7. Stable entity 默认不扩大 automation 权限

即使系统自动创建或升级 stable entity，也只代表“身份可复用”。

它不应自动：

- 创建 approved rule。
- 创建 confirmed role。
- 放宽 automation policy。
- 使 future transaction 自动落账。
- 使 Case Log 中所有相关案例自动变成 strong precedent。

automation policy、rule promotion、role confirmation 仍应走后续 review / governance。

### 8. 旧系统高置信分类可复用的是纪律，不是整套机制

旧系统 `confidence classifier` 的高置信度判断要求同时确认：

- vendor 身份清晰。
- COA 中只有一个合理科目。
- HST 处理方式可确定。
- 没有 unresolved blocking pack。
- soft risk 被 override evidence 覆盖。

这套完整标准不能搬到 stable entity，因为 stable entity 不需要证明 COA/HST。

可以复用的约束是：

- 不靠 LLM 猜。
- 优先使用直接 evidence。
- 不稳定 identity key 不进入长期学习层。
- 明确冲突时不自动放行。
- 人工纠正要区分“系统记忆错了”与“本次是例外”。

需要删去或降级的旧机制：

- 不需要为 stable entity 建完整 `policy_trace`。
- 不需要 activated packs / override evidence / unresolved risks 这类高置信分类轨迹。
- 不需要用 receipt items 证明 entity identity。

### 9. Entity Resolution Node 判断“是谁”，Case Judgment 判断“怎么记”

第一次出现的新交易，应该由 `Entity Resolution Node` 判断它指向哪个 entity 或是否是 new entity。

但该节点的职责必须收窄：

它可以看：

- bank descriptor 中的业务对象身份信号。
- receipt 的 `vendor_name`。
- invoice / contract party。
- cheque `payee`。
- 已有 Entity Log / alias / rejected alias。

它不应深入判断：

- receipt items 的业务用途。
- COA。
- HST/GST。
- 这笔交易能否高置信分类。
- 这笔交易是否可自动落账。

职责切分：

```text
Evidence Intake / Preprocessing
  -> 提取和配对 evidence，例如 bank text、receipt vendor、receipt items、cheque payee、invoice party

Entity Resolution
  -> 只判断：这是谁？
  -> 输出 known entity / new entity / ambiguous / unresolved

Case Judgment
  -> 判断：这次交易是什么业务用途？进哪个 COA？HST 怎么处理？
  -> 使用 entity + receipt items + case memory + profile + accountant notes
```

例如：

- `HOME DEPOT #4521` + receipt vendor = Home Depot
- `Entity Resolution` 可以创建或识别 Home Depot entity
- receipt items 是水泥还是洗碗机，不影响 Home Depot 是否是 stable entity
- receipt items 影响的是 `Case Judgment` 对当前交易的分类判断

### 10. Stable entity 最小 trace

不需要旧系统 `policy_trace`。

但 stable entity 创建仍应有最小 provenance：

- `created_from`
- `evidence_refs`
- `matched_surface_text`
- `created_at`
- 是否自动创建

这不是为了增加复杂治理，而是为了后续纠错、merge/split、rejected alias 和审计追溯时知道它为什么存在。

## 当前未解决问题

以下问题本该继续讨论，但当前窗口尚未完成。

### A. Candidate entity 持久化细节

需要决定：

- `candidate entity` 是否存入 `Entity Log` 本体，还是单独 candidate queue。
- 如果存入 `Entity Log`，它和 active entity 是否共享同一 `entity_id` namespace。
- candidate entity 最小字段是什么。
- candidate entity 何时清理、合并、归档。
- candidate entity 是否可以被多个 case 引用。

当前只暂时确认：

> durable candidate entity record 可以存在，并可被 Case Log 引用；但它不是 authority。

### B. Stable entity 自动创建的边界

需要继续细化：

- 哪些 direct identity signal 足以自动 stable。
- 清晰 bank descriptor 是否足够，还是必须要 receipt / cheque / invoice / contract。
- 第一次出现的普通 vendor 是否默认可自动 stable。
- 什么算“明显身份冲突”。
- 哪些 entity type 必须人工 review。

初步方向：

- 普通 vendor、清晰 receipt vendor / invoice vendor / cheque payee 可自动 stable。
- person / owner / employee / related party / tax authority / payroll / loan / bank transfer 相关对象需要更保守。

### C. Stable 后 alias 如何处理

仍未讨论清楚：

- 创建 stable entity 时，当前 surface text 是否自动成为 approved alias。
- receipt vendor / cheque payee / bank descriptor 的 alias authority 是否不同。
- bank descriptor 是否应先成为 candidate alias。
- approved alias 是否必须 accountant/governance approval。
- rejected alias 与新证据冲突时如何处理。

这是关键问题，因为 rule match 依赖 approved alias。

### D. Role / context 边界

仍未讨论：

- stable entity 是否可以没有 confirmed role。
- role 是否只能 accountant confirmed。
- onboarding accountant-derived role 的边界。
- `resolved_entity_with_unconfirmed_role` 后续如何路由。
- role 缺失是否影响 Case Judgment、Rule Match、Case Log 写入。

当前倾向：

> stable identity 和 confirmed role 应分开；能确认“是谁”不等于确认“它在客户关系里扮演什么角色”。

### E. Case Log 的正式 M1-M2

之前已判断 Case Log 应存在，但还没有正式写 M1-M2。

仍需讨论：

- 哪些 completed transactions 可以进入 Case Log。
- rule-handled / structural-handled 交易是否进入 Case Log。
- system-supported high-confidence outcome 是否进入 Case Log。
- accountant-reviewed / corrected outcome 是否具有更高 case authority。
- exception case 如何保存。
- candidate-linked case 是否只能作为 weak context。
- case correction / supersession 如何处理。

尤其未解决：

> 如果 candidate entity 后来升级为 stable，之前引用 candidate entity 的 cases 是否自动变强，还是需要单独 review。

### F. Case Log 与 Transaction Log 的引用关系

仍需细化：

- Case Log 是否必须引用 `transaction_log_ref` 或等价 finalization proof。
- 如果交易后来被修正、split、reprocess、reversal，Case Log 如何同步或 supersede。
- Case Log 是否保存最终 outcome 快照，还是只引用 Transaction Log。

当前倾向：

> Case Log 应引用最终完成证明，但不能替代 Transaction Log。

### G. Candidate -> stable 由哪个流程执行

仍未冻结：

- `Entity Resolution Node` 是否只能输出 new entity candidate，不直接写 durable candidate。
- durable candidate entity 由 `Case Memory Update Node` 创建，还是由独立 entity memory update/governance intake 创建。
- stable entity 自动创建是否属于 Entity Resolution 后的同步步骤，还是 batch-end / post-transaction step。
- Governance Review 是否只处理高风险 stable promotion，还是所有 candidate -> stable 都经过它。

当前倾向：

> Entity Resolution 判断“是谁/是不是新对象”，但不应同时承担 durable mutation；写入应在后续 memory update / governance intake 层完成。

### H. Automation policy 默认值

仍未讨论清楚：

- 新 stable entity 默认 `automation_policy` 是什么。
- 普通 vendor 是否默认 `case_allowed_but_no_promotion`。
- 高风险 entity 是否默认 `review_required`。
- 是否存在 `identity_only` 或类似更清晰的初始状态。

当前确认：

> stable entity 不应自动放宽 automation。

### I. 旧系统约束如何正式翻译

已读旧系统材料包括：

- `old_system_nodedesign/confidence_classifier_spec.md`
- `old_system_nodedesign/observations_spec_v2.md`
- `old_system_nodedesign/data_preprocessing_agent_spec_v3.md`
- `old_system_nodedesign/review_agent_spec_v3.md`
- `old_system_nodedesign/onboarding_agent_spec.md`
- `old_system_nodedesign/transaction_log_spec.md`

可复用意图需要进一步整理进正式文档：

- direct evidence 优先。
- unstable key 不进学习层。
- classification correction 与 exception 要分开。
- Transaction Log 不参与 runtime decision。
- 高权限升级必须和普通学习记录分开。

但不能把旧系统当 baseline 或照搬旧结构。

### J. 现有 Entity Resolution Node 草案需要回头审计

当前 `Entity Resolution Node` 草案可能仍偏重：

- governance issue。
- risk flags。
- candidate signal。
- authority annotation。

用户已指出：

> 对 entity 来说，绝大多数情况只是能识别出是谁，或不能识别出是谁；不要为 stable entity 设计过多限制。

后续应审计并可能收窄该节点：

- 保留“识别是谁”的核心。
- 去掉不必要的 risk-pack 式复杂度。
- 避免与 Case Judgment 重叠。
- 明确 receipt vendor 与 receipt items 的分工。

## 下一步建议

compact 后建议继续从一个问题开始：

> 第一次出现的新交易，如果 Entity Resolution 能根据 receipt vendor / cheque payee / invoice party / 清晰 bank descriptor 判断它是谁，是否允许系统自动创建 stable entity？如果允许，哪些 entity 类型或冲突场景必须例外进入人工 review？

这个问题解决后，再决定：

1. stable entity 创建时 alias 的默认状态。
2. candidate entity 的最小字段。
3. Case Log M1-M2 的写入边界。
4. 是否修改 `Entity Log` 和 `Entity Resolution Node` 当前草案。

