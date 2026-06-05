# Entity Resolution L2 提案

## 依赖

- 上游 Data Pre-processing / Evidence Intake 已提供可追溯 evidence，包括交易原始数据、raw description、小票、invoice、cheque、contract 等身份相关证据。
- Alias 的已锁定义：Alias Log 中的 Alias 是已确认 transaction surface text -> stable entity 的身份复用关系。
- Entity Resolution 判断为 `new_stable_entity` 时，只认定 entity 本体可持久化；Alias、rule、automation policy 不随本体一起确认。

## L2 决策

### Stable / unknown 分界

- 结论: 只要当前所有可追溯证据能够直接、清晰且无歧义地说明交易主体是谁，Entity Resolution 就可以将其判定为 stable；否则输出 unknown。这个判断只回答“这是谁”，不要求证明 COA、HST / GST、业务用途、Rule 或自动化权限。
- 为什么: 支持记忆复用、有证据支持的建议、审计性和自动化率。stable entity 是后续 Rule Match、Case Judgment、Case Log 查询的身份基础；如果身份清楚却仍不允许 stable，会制造不必要 pending；如果身份不清楚却 stable，会污染长期身份权威。
- 拟改:`BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:输出类别` → `stable entity 的 L2 判断标准是：当前所有可追溯证据能够直接、清晰且无歧义地说明交易主体是谁。该判断只限 identity，不包含会计分类、税务处理、业务用途、Rule 或自动化权限。`
- 拟改:`BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:证据不足时的行为` → `当可追溯证据不能直接、清晰且无歧义地说明主体是谁时，本节点输出 unknown，并可携带身份缺口 reason；不得为了推进流程伪造 stable entity。`

### 输出状态两态化

- 结论: ER 的 entity identity 状态只输出 stable / unknown。ambiguous、conflict、unresolved、missing evidence、alias issue 等都不是第三种 entity 状态，只能作为 unknown 的 reason / context。
- 为什么: 支持 accountant control、审计性和纠正学习。两态输出防止 Candidate Entity / Candidate Identity 以输出类别形式复活，同时让下游只依赖明确的 identity contract。
- 拟改:`BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:输出类别` → `本节点对外只声明两类 identity state：stable 与 unknown。unknown 可以携带 reason，例如 ambiguous、conflict、unresolved、missing evidence 或 alias issue；这些 reason 不构成 candidate entity、stable entity、Case Log handle 或 Rule Match basis。`
- 拟改:`BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:歧义处理` → `如果存在多个合理身份解释，本节点输出 unknown，并将歧义原因写入 reason；不得输出 candidate entity 或选择一个 winner。`

### Stable 输出必须带可审计 reason

- 结论: ER 输出 stable 时必须同时输出一条可审计 reason，说明依据哪些 evidence points 判定主体明确。reason 是凝练后的 evidence-based rationale，不是完整思维链，也不是会计分类理由。
- 为什么: 支持有证据支持的建议、审计性和 accountant control。未来 review / correction 可以直接检查 stable 判断是否来自可追溯证据，而不是依赖不可见模型判断。
- 拟改:`BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:Audit / Trace 边界` → `stable 输出必须保留 identity reason：用最少、可审计文字说明本次 stable 判断依赖的 evidence points。该 reason 不能成为会计分类、Rule authority、Case authority、accountant approval 或 governance approval。`

### Exact Alias match 的 ER 语义

- 结论: 当前 transaction surface text 与 Alias Log 中既有 Alias 完全一致时，ER 可以直接输出该 Alias 指向的 stable entity，不再要求 LLM 独立重新判断主体是谁。Alias exact match 是确定性身份复用路径，不是分类路径。
- 为什么: 支持记忆复用、自动化率和控制权。Alias Log 的存在意义就是把已确认 surface text -> stable entity 关系变成可复用身份记忆；如果 exact match 后仍每次重判，Alias 会退化成提示信息。
- 拟改:`BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:读取对象` → `Alias Log / Alias 查询接口用于 exact Alias lookup；完全命中时可直接复用 Alias 指向的 stable entity。ER 不依赖 Alias Log 的内部存储形态。`
- 拟改:`BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:输出类别` → `如果当前 surface text 完全命中 Alias Log 中既有 Alias，本节点输出 stable entity，并在 reason 中标明 exact Alias match。该输出不代表 Rule Match 成功、会计分类或自动化许可。`
- 浮现的新 open boundary: Alias 写入资格、normalization / equivalence、冲突修正和 supersession 语义归 Alias Log L2 提案处理。

## 分类备案

- `entity_resolution_output` exact field schema → L3(留联合 L3)
- `new_stable_entity` 最小创建 provenance exact field schema → L3(留联合 L3)
- ER 认定 `new_stable_entity` 后的实际写入者、写入顺序和多 log finalization → L4 / seam-park(推给记录层)
- Alias 写入责任、审批路径、normalization / equivalence 和冲突修正 → 后续 Alias Log L2
- `knowledge_summary_conflict_repair` → L2·外阻(依赖 Knowledge Summary / Governance Log,挂起)

## 自检与判定

- 按模板 L2 清单自检:通过;未过项 = 第22条不能进 Stage 3，因为字段级 schema、provenance schema 和 Knowledge Summary repair 尚未冻结。
- 本对象 L2 可否判定完成:否。stable 阈值、输出状态两态化、stable reason 和 exact Alias match 已收口；但运行 / 记忆 seam 的正式措辞仍需审核确认。
