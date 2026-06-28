# Entity Resolution Node - Logic and Boundaries

## 1. 触发条件

本节点在以下条件同时满足时触发：

- Evidence Intake / Preprocessing（证据接收 / 预处理）已经形成 traceable evidence foundation（可追溯证据基础）。
- Evidence Intake / Preprocessing（证据接收 / 预处理）已经为当前交易分配稳定 `transaction_id`（稳定交易 ID）。
- Profile / Structural Match（客户结构匹配）已经确认当前交易没有被 structural path（结构性路径）完成。
- 当前非结构性交易需要知道 evidence（证据）指向哪个 counterparty / vendor / payee（交易对手 / 商家 / 收款人），或为什么不能安全确认。

本节点不得在以下情况触发：

- `structural_path_status`（结构性路径状态）表示交易已经被结构性路径完成。
- `transaction_id`（稳定交易 ID）缺失或不稳定。
- `evidence_refs`（证据引用）缺失或不可追溯。
- 当前任务是 onboarding initialization（初始化历史学习）、review approval（审核批准）、governance approval（治理批准）或 transaction logging（交易审计记录）。

## 2. 上游前置条件

上游必须已经完成：

- Evidence Intake / Preprocessing Node（证据接收 / 预处理节点）：提供 objective transaction basis（客观交易基础）和 evidence refs（证据引用），并为当前交易分配 `transaction_id`（稳定交易 ID）。
- Profile / Structural Match Node（客户结构匹配节点）：提供 non-structural handoff（非结构性交易交接）。

如果前置条件缺失：

- 本节点行为：拒绝把输入当作有效 identity request（身份识别请求），输出 invalid handoff / blocked handoff（无效交接 / 被阻断交接）语义。
- 是否 stop-and-ask：如果缺失来自未冻结的跨节点 contract（跨节点契约），停止并要求产品设计决策；不能由实现代理猜。

## 3. 读取对象

| Source | 读取内容 | 用途 | Authority 限制 |
| --- | --- | --- | --- |
| Evidence Log（证据日志） / evidence foundation（证据基础） | `evidence_refs`（证据引用）、raw bank text（银行原始文本）、receipt vendor（小票商家）、cheque payee（支票收款人）、invoice party（发票主体）、contract party（合同主体）、accountant context refs（会计师上下文引用） | 识别当前 evidence（证据）表面指向谁 | Evidence（证据）不保存业务结论；模型摘要不能替代 evidence ref（证据引用） |
| AI 联网搜索 / external lookup（外部查询） | 针对 vendor / payee / counterparty 身份的搜索线索、相似对象、公开身份解释、由搜索打开并可追溯的外部来源 | 当 bank descriptor 只显示支付平台、缩写或不完整名称时，辅助判断当前交易主体是谁；搜索结果列表和搜索摘要只能作为 clue；可追溯外部来源在与当前交易 evidence 共同指向清晰、唯一、无实质竞争的业务对象时，可作为 stable reason 的 evidence point | 搜索行为本身不是 authority；外部来源 evidence point 只支持 identity，不支持 COA / HST / GST / 业务用途 / Rule / automation permission |
| Evidence Intake（证据接收 / 预处理）分配的 transaction_id | `transaction_id`（稳定交易 ID） | 保证本次身份判断绑定到稳定交易对象 | 交易身份不等于 entity identity（实体身份），也不等于 accounting authority（会计权威） |
| Profile / Structural Match handoff（客户结构匹配交接） | `structural_path_status`（结构性路径状态）、non-structural reason（非结构性原因）、profile context refs（客户结构上下文引用） | 防止结构性交易穿透后被普通实体流程处理 | candidate profile fact（候选客户结构事实）不能当 stable profile truth（稳定客户结构事实） |
| Entity Log（实体日志） | known entities（已知实体）、`entity_status`（实体生命周期状态）、authority metadata（权威元数据）、force_pending | 判断 evidence（证据）是否安全指向稳定 entity（实体），或新建 stable entity 是否与既有 entity 竞争 | Summary（摘要）不能覆盖 Entity Log（实体日志）；Entity Log identity authority 不携带会计分类、Rule 或 automation permission 判断 |
| Alias Log / Alias 查询接口 | 已确认 Alias surface text 到 stable entity 的对应关系 | exact Alias lookup（完全命中查询）；完全命中时可直接复用 Alias 指向的 stable entity | Alias Log / Alias 库具体技术形态未冻结；Alias exact match 是身份复用路径，不是分类路径；未确认 surface text 不能当 Alias |
| Governance Log（治理日志） | applied merge / split, lifecycle, policy constraints（已生效的合并 / 拆分、生命周期、策略限制） | 限制或解释当前身份判断 | pending / rejected governance event（待批准 / 已拒绝治理事件）不能当 positive authority（正向权威） |
| Intervention Log（人工介入日志） | identity correction（身份纠正）、accountant confirmation（会计师确认）、recent dispute（近期争议） | 识别当前身份风险或限制 | 人工介入记录只有明确确认对象时才可作为对应 authority（权威）；不能泛化 |
| Knowledge Summary（知识摘要） | readable identity notes（可读身份说明）、known ambiguity notes（已知歧义说明） | 辅助理解客户语境 | 不能替代 Entity Log（实体日志）或 Governance Log（治理日志） |

## 4. 写入对象

### 直接执行的 durable write（例外）

本 L2 不冻结 ER 是亲手执行 durable write，还是同步调用专门 Entity Log writer / memory write mechanism。直接写入与调用写入的执行机制、调用方式和顺序属于 L4 / seam。

### 交给记忆 / finalization 层持久化的内容（运行 / 记忆 seam）

当 ER 判定新建 stable entity 后：

- ER 自主决定该 stable entity 本体有效。
- 该 finalization 不需要 governance approval（治理批准）。
- ER 必须在当前交易进入下游 judgment / pending 前发起同步 Entity Log + Alias Log finalization，使该 stable entity 及其对应 confirmed Alias projection 对后续同 batch / 后续交易的 identity consumer 可见。
- 本节点冻结要 finalization 的语义对象：新建 stable entity 本体、能说明其由 ER 基于本次可追溯 identity evidence 判定为 stable 的最小创建 provenance 语义，以及该交易符合 Alias Log 写入资格的可追溯 raw surface text -> stable entity identity reuse relationship。
- Entity Log 侧保存 stable entity 本体、最小创建 provenance 和初始 entity-centered Alias surface；Alias Log 侧写入面向 ER 查询的 alias -> entity 反查 projection。两者是同一已确认身份关系的两个访问面，不是两套独立身份 authority。
- 实际由 ER 亲手写入，还是由 ER 同步调用专门写入 / 存储节点立即完成，属于 L4 / seam；exact provenance / alias_record field schema 属于 L3。

该 finalization 不代表 Rule（规则）、force_pending / promotion_lock、Case Log（案例日志）或 Transaction Log（交易审计日志）已经写入。

### 非身份输出只剩 `unknown` 的 reason / context

本节点不产出与身份状态并行的候选 / 风险通道。原 `candidate_signal`、`merge_split_candidate`、`alias_conflict_issue`、`identity_governance_issue` **已删除（2026-06-25 收口）**：它们无下游消费者（Case Judgment 只读身份结果与 `unknown` reason，见 §6；Human Review 从 Entity Log 读 merge / split / force_pending 上下文并只接会计师人发起）、与 `unknown` reason 重复、或超出本节点能力 / 职责（merge / split 见 §5、§9）。

除新建 stable entity 的同步 Entity Log + Alias Log finalization 外，本节点唯一的非身份输出是：

- `unknown` reason / context（例如 ambiguous、conflict、unresolved、missing evidence、alias issue），只能作为 runtime context，不构成 candidate entity、stable entity、Case Log handle 或 Rule Match basis。

### 绝不能写入或修改

本节点绝不能写入或修改：

- `Transaction Log`（交易审计日志）。
- 未够格 Alias（别名）、Alias correction / governance mutation、force_pending / promotion_lock、merge / split projection（合并 / 拆分投影）等高权限或非初始写入；初始够格 Alias 随新建 stable entity 同步 finalization。
- `Case Log`（案例日志）。
- `Rule Log`（规则日志）。
- `Governance Log`（治理日志）。
- `Profile`（客户结构档案）。
- active rule（生效规则）。
- `force_pending` / `promotion_lock`。

## 5. 决策权限

### Deterministic code 可以决定

- 本节点是否应被触发。
- 输入是否具备 `transaction_id`（稳定交易 ID）、objective transaction basis（客观交易基础）和 traceable `evidence_refs`（可追溯证据引用）。
- 当前交易是否已经被 structural path（结构性路径）完成；若已完成，本节点不得继续。
- `entity_status`（实体生命周期状态）、governance constraint（治理限制）等 authority check（权威检查）是否满足。
- 当前 surface text 是否 exact match（完全命中）已确认 Alias；命中时直接复用该 Alias 指向的 stable entity，不再判断该 entity 是否"干净"（lifecycle / 冲突等留下游与 Entity Log）。
- 当本节点判定新建 stable entity 时，Entity Log + Alias Log finalization 的语义范围是否只限 stable entity 本体、最小创建 provenance 和已够格 raw surface text -> stable entity Alias projection，不夹带 rule / automation authority（规则 / 自动化权威）。

### LLM 可以判断

- messy evidence（杂乱证据）中的 vendor / payee / counterparty 表面写法是否语义上接近某个已知 entity（实体）。
- receipt（小票）、cheque（支票）、invoice（发票）、contract（合同）或 accountant note（会计师备注）中有哪些 human-visible identity clues（人类可见身份线索）。
- 目标明确的 AI 联网搜索结果是否提供了 identity clue（身份线索），以及由搜索打开并可追溯的外部来源是否能与当前交易 evidence 共同支持 stable identity reason。
- 当前所有可追溯 evidence 是否能够直接、清晰且无歧义地说明主体是谁；如果不能，只能输出 `unknown` 并说明 reason。
- 第一次出现的新对象是否已有直接、清楚、可追溯且无明显冲突的 identity signal（身份信号），足以作为 provenance=新建的 `stable`。
- identity reason（身份判断理由）、ambiguity reason（歧义原因）和 missing evidence reason（缺失证据原因）的可读摘要。

### LLM 不能判断

- 扩大 entity / Alias authority（实体 / 别名权威）。
- 把未确认的 surface text 当作 Alias 使用。
- 在本节点新建 stable entity finalization 边界之外创建 stable entity（稳定实体）。
- 在存在多个合理 identity 解释、冲突、证据缺失或 alias issue 时选择一个 winner（胜出实体）。
- merge / split entity（合并 / 拆分实体）。
- 修改 `force_pending` / `promotion_lock`。
- 执行 rule match（规则匹配）。
- 判断 case precedent（历史案例先例）是否足以支持分类。
- 选择 COA / HST / GST treatment（会计科目 / 税务处理）。
- 批准 governance event（治理事件）。
- 写入 `Transaction Log`（交易审计日志）。

### Accountant 必须决定

- 是否接受当前交易的最终 accounting outcome（会计处理结果）。
- 是否把当前交易 clarification（澄清）提升为 durable customer memory（长期客户记忆）。
- 模糊自然语言回答是否表达长期规则或只是当前交易说明。

### Governance 必须批准

- 新建 stable entity 的即时 Entity Log + Alias Log finalization 不需要 governance approval；除此之外的 stable entity lifecycle mutation（稳定实体生命周期变更）必须按治理边界处理。
- merge / split entity（合并 / 拆分实体）。
- archive / reactivate entity（归档 / 重新激活实体）。
- 设置或解除 `force_pending` / `promotion_lock`。
- create / promote / modify / delete / downgrade active rule（创建 / 升级 / 修改 / 删除 / 降级生效规则）。

## 6. 输出类别

字段名可以暂不冻结，但语义类别必须稳定。

本表是本节点对下游唯一的契约面：下游只能依赖此处声明的输出类别，不得依赖本节点未声明的内部状态或实现。

本节点对外只声明两类 identity state：`stable` 与 `unknown`。除身份结果及其 reason / context 外，本节点不声明任何与身份状态并行的候选 / 风险输出通道。

| Output Category | 含义 | Consumer（谁消费） | 下游影响 | 不代表什么 |
| --- | --- | --- | --- | --- |
| `stable` | 当前所有可追溯 evidence 能够直接、清晰且无歧义地说明交易主体是谁。stable 内部有两种 provenance：复用既有（命中 Entity Log 中已存在的 stable entity，或 exact Alias match 指向既有 stable entity）与新建（第一次识别到的新对象、需新建档案）。两者都输出 state=`stable`，并必须携带可审计 identity reason。 | Rule Match（身份基础）；Case Judgment | 下游可把它作为当前交易 identity basis。若 provenance=新建，ER 必须在当前交易进入下游 judgment / pending 前发起同步 Entity Log + Alias Log finalization，使该 stable entity 及其 confirmed Alias projection 对后续 identity consumer 可见。 | 不代表 rule match 成功、会计分类、COA / HST / GST / 业务用途、automation permission、Case Log authority、accountant approval 或 governance approval。 |
| `unknown` | 当前可追溯 evidence 不能直接、清晰且无歧义地说明主体是谁。unknown 可以携带 reason / context，例如 ambiguous、conflict、unresolved、missing evidence 或 alias issue；这些 reason 不构成 candidate entity、stable entity、Case Log handle 或 Rule Match basis。 | Case Judgment（输出 pending） | Case Judgment 不走高置信度自动分类通道，输出 pending；可读取身份线索、搜索线索、相似对象、reason 和 evidence refs 作为 runtime context。 | 不代表会计分类失败的总称，不允许伪造 entity，不代表 LLM 可以选择一个 winner，不支持 Rule Match 或 durable memory mutation。 |

stable reason 的证据边界：

- 如果当前 surface text 完全命中 Alias Log / Alias 库中既有 Alias，本节点输出 `stable`，并在 reason 中标明 exact Alias match。该输出不代表 Rule Match 成功、会计分类或自动化许可。
- stable reason 可以引用可追溯 external evidence point，但该引用只支持 identity，不支持 COA / HST / GST / 业务用途 / Rule / automation permission。
- AI 联网搜索的搜索结果列表、搜索摘要或模型复述只能作为 clue；只有由搜索打开并可追溯的外部来源，在与当前交易 evidence 共同指向唯一、清晰、无实质竞争的业务对象时，才可以作为 stable reason 的 evidence point。
- `new_stable_entity` 不再是独立 identity state；它是 `stable` 的 provenance=新建语义。对应 stable entity 必须在继续下游前完成同步 Entity Log + Alias Log finalization 或等价可见化；该 finalization 不代表 Rule、force_pending / promotion_lock、Case Log 或 Transaction Log 已写入。

Alias 查询对输出的影响：

- 如果当前 surface text exact match 已确认 Alias，可以直接复用该 Alias 指向的 stable entity。
- 如果只是高度类似历史 Alias，不能天然等同于确认 identity；它最多作为 clue 参与 ER 的 stable / unknown 判断。
- Alias 初始写入资格以 Alias Log / Entity Log 为准：当前交易已确认指向明确 stable entity，且写入对象是该交易可追溯 raw surface text -> stable entity 关系时，必须随 stable entity 创建同步进入 Alias Log projection；normalization / equivalence、冲突修正和 supersession 语义不在本节点 L2 冻结。

Rule Match 对输出的读取边界：

- Rule Match 接收本节点已经确认的 stable entity 结果，再判断当前交易是否满足 rule scope。
- Rule 的核心不是 Alias；Alias lookup 发生在 Entity Resolution 阶段。
- Entity-level rule 围绕 stable entity 建立；pattern-level rule 围绕 stable entity 下更窄的稳定交易模式建立。
- 如果一个 entity 下存在多种最终会计分类结果，该 entity 本身不应升级成 entity-level rule，需要进入更窄 scope 选择。

## 7. 证据不足时的行为

如果缺少 identity signal（身份信号）：

- 输出：`unknown` + reason（例如 missing evidence / unresolved）。
- 下游应：Case Judgment Node（案例判断节点）输出 pending；Coordinator / Pending Node（协调 / 待确认节点）再决定是否向 accountant（会计师）补问，或由 Human Review Node（人审节点，会计师人发起）处理。
- 本节点不能：为了让 workflow（流程）继续而伪造 entity（实体）。

### 已确认的下游影响（Problem 2 结论）

当本节点输出 `unknown` + reason（无法清楚识别身份）时：

- Case Judgment Node（案例判断节点）不走高置信度自动分类通道，输出 pending。
- 本节点可以把 identity 判断相关的 runtime context 传给 Case Judgment：当前 evidence 中的人类可见身份线索、搜索线索、可能的身份解释、相似对象、ambiguity reason、missing evidence reason、Alias 冲突和相关 evidence refs。
- 这些 runtime context 不是 stable entity 或 identity authority，不能支持 Rule Match、Case Log authority 或 durable memory mutation。
- 但 Case Judgment 仍然处理这笔交易，pending 内部分两种情况：
  - **完全无法判断**：除 entity 未识别外，也没有其他足够上下文支持推断。由 Coordinator 直接向 accountant 提问。
  - **有推断但不确定**：Case Judgment 可利用其他已有上下文（如 receipt items、交易金额模式、bank descriptor 特征等）给出推断性建议，附在 pending 输出中，由 Coordinator 呈现给 accountant 选择或确认。
- 存在不依赖 entity identity 就能判断 COA 的交易类型（bank fee、interest 等），但这些属于 edge case，大部分由上游 Profile / Structural Match Node 处理。到达 Case Judgment 的交易绝大多数需要 entity identity。
- A/B 分类判断（是否需要 entity 才能分类）的职责在 Case Judgment，不在本节点。本节点只管 identity。

### 已确认的后续路径（Problem 4 结论）

当本节点输出 `unknown` 后：

- Case Judgment 输出 pending。
- Coordinator 根据 pending 子类型向 accountant 提问。
- Accountant 明确确认 identity 后，交易不重新进入本节点。
- Accountant confirmation（会计师确认）替代 Entity Resolution 的身份判断；stable entity 创建和最终分类由后续路径完成。
- 已确认交易不重新进入 Case Judgment；分类在 Coordinator 与 accountant 的交互中完成。
- Coordinator 追问结构、问题模板和 finalization / memory write 机制仍属于下游未冻结问题，不由本文冻结。

如果缺少 traceable evidence ref（可追溯证据引用）：

- 输出：invalid handoff / blocked handoff（无效交接 / 被阻断交接）语义。
- 下游应：返回 Evidence Intake / Preprocessing（证据接收 / 预处理）或停止设计讨论。
- 本节点不能：使用不可追溯摘要作为 identity authority（身份权威）。

## 8. 歧义处理

如果存在多个合理 identity 解释：

- 输出：`unknown` + reason（例如 ambiguous / competing identities）。
- 是否允许自动化：本节点不决定自动化；但不得把歧义输出包装成 stable identity（稳定身份）。
- 是否需要 pending / review / governance：Case Judgment 输出 pending 后，由 Coordinator / Pending（协调 / 待确认）、Human Review（会计师人发起）或 Governance（授权确认 = Human Review + Finalization）根据卡点和 `unknown` reason 决定。

本节点不能为了让 workflow 继续而猜一个 winner（胜出实体）。

如果 external lookup 只提供相似名称、搜索排序、搜索摘要、模糊同名对象、低质量聚合页或无法排除多个合理 identity，本节点只能把搜索结果作为 clue，输出 `unknown` + reason，不得用其支撑 stable。

## 9. 冲突处理

如果 current evidence（当前证据）与 Entity Log（实体日志）或 Governance Log（治理日志）冲突：

- authority 顺序：Governance Log（治理日志）/ Entity Log（实体日志）中的 approved / applied authority（已批准 / 已生效权威）优先于 Knowledge Summary（知识摘要）和 LLM semantic match（LLM 语义匹配）。
- 本节点行为：保守输出 `unknown` + reason（例如 conflict / blocked / ambiguous）。
- 是否生成 review / governance candidate：不生成并行候选；冲突经 `unknown` + reason（conflict / blocked）表达，治理处置由会计师人发起 Human Review + Governance 决定。
- 是否阻断自动化：本节点只说明 identity authority problem（身份权威问题）；下游决定 automation path（自动化路径）。

如果 accountant context（会计师上下文）与 durable memory（长期记忆）冲突：

- authority 顺序：明确 accountant confirmation（会计师确认）只在其确认范围内有效；模糊说明不能泛化成 durable authority（长期权威）。
- 本节点行为：输出 conflict reason（冲突原因）和 relevant refs（相关引用）。
- 是否生成 review / governance candidate：不生成并行候选；冲突经 conflict reason + relevant refs 表达，治理处置由会计师人发起。
- 是否阻断自动化：如果冲突影响当前身份是否安全，不能输出 stable identity basis（稳定身份基础）。

## 10. Audit / Trace 边界

本节点不另设独立 trace 账：ER 是近乎无状态的身份判断步，审计面就是它自己的输出。审计材料 = 身份结果（`identity_state` / `entity_id`）+ `identity_reason`（stable）/ `unknown_reason_context`（unknown）+ `identity_evidence_refs`。

- `identity_evidence_refs` 的 ref 可指向内部 Evidence Log，或在 stable 依赖联网时指向外部可追溯来源（外部来源的 exact URL / source title / retrieved_at / snippet 等字段形态留到 Stage 3）。
- 治理 / 介入限制若影响了本次判断，作为 ref 挂在 reason 上，不单设字段。
- 新建实体的 creation provenance 归 Entity Log（E 同步 finalization 写入），不是 ER 的运行期 trace。
- 交易级审计材料（`identity_reason` + `identity_evidence_refs`，含外部来源 ref）随交易在 finalization 时落入 Transaction Log；ER 只产出、不写 Transaction Log。

这些审计材料用于 review（审核）/ correction（纠正）/ governance（治理）/ audit（审计），但本身不能绕过 stable output / Entity Log 成为独立的 entity authority（实体权威）、accounting classification（会计分类）、rule authority（规则权威）、case authority（案例权威）、accountant approval（会计师批准）或 governance approval（治理批准）。

## 11. Legacy Constraint Translation

仅保留仍服务当前产品目标的旧系统约束。

| Retained Constraint | 来源 | 为什么现在仍成立 |
| --- | --- | --- |
| Transaction Log（交易审计日志）不参与 runtime decision（运行时决策） | 旧系统 Transaction Log 边界 | 身份识别必须基于 current evidence（当前证据）和 active authority（当前有效权威），不能从最终历史审计记录中反向学习未治理结论 |
| JE Generation（分录生成）应是纯计算层 | 旧系统 JE generator 边界 | Entity Resolution（实体识别）只回答“是谁”，不应提前做会计分录或税务判断 |
| PENDING（待确认）必须能转成人能回答的问题 | 旧系统 Coordinator 意图 | 本节点只暴露身份卡点；问题生成应由 Coordinator / Pending Node（协调 / 待确认节点）完成 |

不保留的旧行为：

| Old Behavior | 不保留原因 |
| --- | --- |
| 用 canonical description / pattern（标准描述 / 模式）作为学习链条核心身份 | 新系统的 identity source（身份来源）应是 evidence-grounded entity（有证据支撑的实体），pattern（模式）只能作为表面信号或展示字段 |
| 用 repeated outcome（重复结果）直接靠近 rule authority（规则权威） | 当前系统要求 rule promotion（规则升级）经过 accountant / governance approval（会计师 / 治理批准） |

## 12. Open Boundaries

以下问题未冻结：

1. stable identity 的语义判据已冻结；exact numeric threshold（精确数字门槛）、confidence cutoff（置信度切线）或评分规则如需存在，仍属 L3。
2. `entity_resolution_output`（实体识别运行时输出）的 exact field schema（精确字段结构）。
3. `new_stable_entity` 最小创建 provenance 的 exact field schema（精确字段结构）。
4. external evidence reference（外部证据引用）的 exact field schema、网页快照 / 缓存 / retrieval 机制、source quality taxonomy。
5. ER 判定新建 stable entity 后，Entity Log + Alias Log 同步 finalization 的实际写入执行者、调用方式、写入顺序和多 log finalization。
6. Alias normalization / equivalence、冲突修正和 supersession 语义。
7. `knowledge_summary_conflict_repair`（Knowledge Summary 与 Entity Log / Governance Log 冲突时的修复流程）。

这些问题解决前，不能进入：

- [x] Stage 3 data contract
- [x] Stage 4 execution algorithm
- [x] implementation
