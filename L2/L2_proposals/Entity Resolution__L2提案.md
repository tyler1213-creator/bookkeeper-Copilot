> ⚠️ **已过时 / superseded（owner 2026-06-27）** — 本 L2 提案的 L1 / L2 结论已由 `BK_Copilot/workflow_nodes/entity_resolution_node/` 正式草案取代，仅作历史来源保留。当前权威以 `BK_Copilot/` 正式草案 +（产出后）对应 L3 schema 为准；**勿据本文件回灌已删除的概念 / 字段**。

# Entity Resolution L2 提案

## 1. 依赖

- **Evidence 前置**：上游 Data Pre-processing / Evidence Intake 已提供可追溯 evidence，包括交易原始数据、raw description、小票、invoice、cheque、contract 等身份相关证据。
- **Alias 已锁定义**：Alias Log 中的 Alias 是已确认 transaction surface text -> stable entity 的身份复用关系。
- **New stable entity 边界**：Entity Resolution 判断为 `new_stable_entity` 时，只认定 entity 本体可持久化；Alias、rule、automation policy 不随本体一起确认。

## 2. L2 决策

### 2.1 Stable / Unknown 分界

**结论：**
只要当前所有可追溯证据能够直接、清晰且无歧义地说明交易主体是谁，Entity Resolution 就可以将其判定为 stable；否则输出 unknown。这个判断只回答“这是谁”，不要求证明 COA、HST / GST、业务用途、Rule 或自动化权限。

**为什么：**
支持记忆复用、有证据支持的建议、审计性和自动化率。stable entity 是后续 Rule Match、Case Judgment、Case Log 查询的身份基础；如果身份清楚却仍不允许 stable，会制造不必要 pending；如果身份不清楚却 stable，会污染长期身份权威。

**拟改：**

- `BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:输出类别` → `stable entity 的 L2 判断标准是：当前所有可追溯证据能够直接、清晰且无歧义地说明交易主体是谁。该判断只限 identity，不包含会计分类、税务处理、业务用途、Rule 或自动化权限。`
- `BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:证据不足时的行为` → `当可追溯证据不能直接、清晰且无歧义地说明主体是谁时，本节点输出 unknown，并可携带身份缺口 reason；不得为了推进流程伪造 stable entity。`

### 2.2 输出状态两态化

**结论：**
ER 的 entity identity 状态只输出 stable / unknown。ambiguous、conflict、unresolved、missing evidence、alias issue 等都不是第三种 entity 状态，只能作为 unknown 的 reason / context。

**为什么：**
支持 accountant control、审计性和纠正学习。两态输出防止 Candidate Entity / Candidate Identity 以输出类别形式复活，同时让下游只依赖明确的 identity contract。

**拟改：**

- `BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:输出类别` → `本节点对外只声明两类 identity state：stable 与 unknown。unknown 可以携带 reason，例如 ambiguous、conflict、unresolved、missing evidence 或 alias issue；这些 reason 不构成 candidate entity、stable entity、Case Log handle 或 Rule Match basis。`
- `BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:歧义处理` → `如果存在多个合理身份解释，本节点输出 unknown，并将歧义原因写入 reason；不得输出 candidate entity 或选择一个 winner。`

### 2.3 Stable 输出必须带可审计 Reason

**结论：**
ER 输出 stable 时必须同时输出一条可审计 reason，说明依据哪些 evidence points 判定主体明确。reason 是凝练后的 evidence-based rationale，不是完整思维链，也不是会计分类理由。

**为什么：**
支持有证据支持的建议、审计性和 accountant control。未来 review / correction 可以直接检查 stable 判断是否来自可追溯证据，而不是依赖不可见模型判断。

**拟改：**

- `BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:Audit / Trace 边界` → `stable 输出必须保留 identity reason：用最少、可审计文字说明本次 stable 判断依赖的 evidence points。该 reason 不能成为会计分类、Rule authority、Case authority、accountant approval 或 governance approval。`

### 2.4 AI 联网搜索作为 Stable Reason Evidence Point

**结论：**
AI 联网搜索可以参与 stable 判定，但搜索行为本身不是 authority。搜索结果列表、搜索引擎摘要或模型对搜索结果的复述只能作为 clue；由搜索打开并可追溯的外部来源，可以作为 stable reason 的 evidence point 之一。该 evidence point 只能用于 identity 判断，且必须与当前交易证据共同指向一个清晰、唯一、无实质竞争的业务对象。

**为什么：**
支持记忆复用、有证据支持的建议、审计性、accountant control 和自动化率。大量交易缺少 receipt / invoice，bank descriptor 又经常短、脏、缩写或带 payment processor 前缀；如果禁止可追溯外部来源参与 stable reason，ER 会过度保守并制造不必要 pending。但如果把搜索排序、摘要或不可追溯网页当作 authority，又会放大 SEO、同名 entity 和搜索结果漂移风险。

**拟改：**

- `BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:读取对象` → `AI 联网搜索 / external lookup 只能用于辅助 identity 判断。搜索结果列表和搜索摘要只能作为 clue；由搜索打开并可追溯的外部来源，在能够与当前交易 evidence 共同指向清晰、唯一、无实质竞争的业务对象时，可以作为 stable reason 的 evidence point。`
- `BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:输出类别` → `stable entity 的 reason 可以引用可追溯 external evidence point，但该引用只支持 identity，不支持 COA / HST / GST / 业务用途 / Rule / automation permission。`
- `BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:Audit / Trace 边界` → `如果 stable reason 依赖联网搜索取得的外部来源，本节点必须保留可追溯 external evidence reference，使 reviewer 能回到来源核验 identity 判断；exact URL / source title / retrieved_at / snippet 等字段形态留到 Stage 3。`
- `BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:歧义处理` → `如果外部来源只提供相似名称、搜索排序、模糊同名对象、低质量聚合页或无法排除多个合理 identity，本节点只能把搜索结果作为 clue，输出 unknown + reason，不得用其支撑 stable。`

**排除的替代 + 理由：**

- 完全禁止联网搜索成为 stable reason evidence。理由是这会让缺少 receipt / invoice 的真实交易过度进入 unknown / pending，削弱自动化率和记忆复用。
- 允许搜索结果列表或搜索摘要直接支撑 stable。理由是搜索排序和摘要不是 source authority，容易受 SEO、同名对象和结果漂移影响，无法满足可审计 identity evidence 要求。

**浮现的新 open boundary：**
external evidence reference 的 exact field schema、网页快照 / 缓存 / retrieval 机制、source quality taxonomy 属于 L3 / L4，不在 ER L2 冻结。

### 2.5 Exact Alias Match 的 ER 语义

**结论：**
当前 transaction surface text 与 Alias Log 中既有 Alias 完全一致时，ER 可以直接输出该 Alias 指向的 stable entity，不再要求 LLM 独立重新判断主体是谁。Alias exact match 是确定性身份复用路径，不是分类路径。

**为什么：**
支持记忆复用、自动化率和控制权。Alias Log 的存在意义就是把已确认 surface text -> stable entity 关系变成可复用身份记忆；如果 exact match 后仍每次重判，Alias 会退化成提示信息。

**拟改：**

- `BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:读取对象` → `Alias Log / Alias 查询接口用于 exact Alias lookup；完全命中时可直接复用 Alias 指向的 stable entity。ER 不依赖 Alias Log 的内部存储形态。`
- `BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:输出类别` → `如果当前 surface text 完全命中 Alias Log 中既有 Alias，本节点输出 stable entity，并在 reason 中标明 exact Alias match。该输出不代表 Rule Match 成功、会计分类或自动化许可。`

**浮现的新 open boundary：**
Alias 写入资格、normalization / equivalence、冲突修正和 supersession 语义归 Alias Log L2 提案处理。

### 2.6 New Stable Entity 的及时 Publication

**结论：**
ER 判定 `new_stable_entity` 后，必须在当前交易继续进入下游 judgment / pending 路径前，发起同步的 Entity Log publication，使该 stable entity 对后续同 batch / 后续交易的 identity consumer 可见。这里冻结的是“及时可见”的 L2 语义，不冻结由 ER 直接写入，还是由 ER 同步调用专门 Entity Log writer / memory write mechanism 执行。

**为什么：**
支持记忆复用、自动化率和运行效率。stable entity 是 identity authority，不是 final accounting outcome；如果等到 accountant pending 或交易最终完成后再统一写入，系统在等待期间处理后续交易时无法复用已确认身份，可能重复解析相同 entity，增加运行成本并延长流程。

**拟改：**

- `BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:写入对象` → `当本节点判定 new_stable_entity 时，必须在下游继续处理前发起同步 Entity Log publication，使该 stable entity 对后续 identity consumer 可见。本节点只声明要持久化的 entity 本体和 authority 来源；实际写入执行者、调用方式和写入顺序属于记忆 / finalization 层机制，不在 L2 冻结。`
- `BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md:输出类别` → `new_stable_entity 输出不仅是 runtime identity result，也要求对应 stable entity 在继续下游前完成同步 publication 或等价可见化；该 publication 不代表 Alias、Rule、automation policy、Case Log 或 Transaction Log 已写入。`

**排除的替代 + 理由：**
延后到 accountant pending 结束或交易 finalization 后统一写入 Entity Log。理由是 entity identity 已确认时不应被 final accounting outcome 的等待状态阻塞；延后写入会让后续交易无法复用已确认身份。

**浮现的新 open boundary：**
ER 直接执行写入、同步调用专门 Entity Log writer，还是进入统一 memory write mechanism，属于 L4 / seam-park。

## 3. 分类备案

- `entity_resolution_output` exact field schema → L3(留联合 L3)
- `new_stable_entity` 最小创建 provenance exact field schema → L3(留联合 L3)
- external evidence reference exact field schema（URL / source title / retrieved_at / snippet 等）→ L3(留联合 L3)
- external lookup retrieval / snapshot / cache 机制 → L4 / seam-park(推给记录层)
- ER 认定 `new_stable_entity` 后的实际写入者、调用方式、写入顺序和多 log finalization → L4 / seam-park(推给记录层)
- Alias 写入责任、审批路径、normalization / equivalence 和冲突修正 → 后续 Alias Log L2
- `knowledge_summary_conflict_repair` → L2·外阻(依赖 Knowledge Summary / Governance Log,挂起)

## 4. 自检与判定

- **按模板 L2 清单自检**：通过；未过项 = 第22条不能进 Stage 3，因为字段级 schema、provenance schema 和 Knowledge Summary repair 尚未冻结。
- **本对象 L2 可否判定完成**：否。stable 阈值、输出状态两态化、stable reason、external evidence 参与 stable reason、exact Alias match 和 new_stable_entity 及时 publication 已收口；但实际写入执行者 / 调用方式仍留 L4 / seam-park。
