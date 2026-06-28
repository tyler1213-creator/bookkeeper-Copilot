# Case Judgment Node - Logic and Boundaries

## 1. 触发条件

本节点的触发是兜底路由语义，不是 ER 式硬前置门。

任何一笔能在系统中流通并被路由到 Case Judgment 的交易，本节点都应处理，并产出以下两类分类结果之一：

- High Confidence Classification。
- Pending。

进入本节点的正常入口有三类：

- ER 对 stable entity 判定“该 entity 没有 active rule”后，直接路由来的交易。
- Rule Match 对 stable entity 下“有 active rule 但本笔无命中”输出的 `rule_miss`。
- ER 输出 `unknown` 的交易。

本节点不得被用作：

- onboarding initialization 或历史回填学习节点。
- Profile / Structural Match 已经完成的结构性路径交易的重复判断节点。
- Rule Match `rule_hit` 的复判节点。
- JE construction / posting 节点。
- Coordinator 已经通过 accountant 交互解决后的交易重入节点。

路由由 ER 自身还是 workflow router 执行，属于 L4 / seam。

## 2. 上游前置条件

上游通常必须已经提供：

- Evidence Intake / Preprocessing 提供的稳定 `transaction_id`、当前交易事实和可追溯 evidence refs。
- Entity Resolution 的身份结果：`stable` / `unknown`、identity reason 和相关 evidence refs；若为 `stable`，提供 stable `entity_id`。
- 路由判定：当前交易没有可执行 active rule，或 Rule Match 已输出 `rule_miss`，或 ER 输出 `unknown`。
- 如果 stable entity 为 ER 本批新建，ER 必须在当前交易进入下游 judgment / pending 前完成同步 publication 或等价可见化，使 CJ 可把该 stable entity 当身份基础。

如果前置条件缺失：

- 本节点行为：结构性无效 / 损坏输入的拒绝与否、被 CJ 与上游均无解交易的最终归处、纠错 candidate signal 的发出者和纠错流程均为 DEFERRED；本 L2 不设计新的拒绝路径。
- 是否 stop-and-ask：如果实现阶段发现缺失来自未冻结的跨节点 contract，停止并要求产品设计决策；不能由实现代理猜测字段、拒收规则或补救流程。

## 3. 读取对象

Case Judgment 输入按加载方式分为三层：

- 固定加载：客户 COA、加拿大会计 / 税务准则（含 HST / GST）、bookkeeping 通用常识、客户 Profile 当前批次稳定快照。
- 动态加载 - 每笔必取：当前交易事实与 evidence、`transaction_id`、ER 身份结果。
- 动态加载 - 按需检索：仅当身份为 `stable` 时，按 `entity_id` 读取 Case Log 相关先例摘要和 entity-level Knowledge Summary；`unknown` 不进行此层检索。

| Source | 读取内容 | 用途 | Authority 限制 |
| --- | --- | --- | --- |
| Evidence foundation / 当前交易事实 | 金额、方向、日期、账户、raw / real description、receipt、cheque、invoice、accountant note、evidence refs、`transaction_id` | 判断当前交易性质、COA 与 HST / GST 是否可唯一落地 | Evidence 不保存业务结论；不可追溯摘要不能替代 evidence ref |
| Entity Resolution output | `stable` / `unknown`、stable `entity_id`、identity reason、identity evidence refs、unknown reason / context | 提供身份基础与身份层 hard block | CJ 不重新论证身份、不用联网反推身份、不把 `unknown` 包装成 stable |
| 客户 COA / 加拿大会计税务准则 / bookkeeping 常识 | COA 全集、HST / GST 处理原则、通用记账知识 | 固定加载知识，支撑 COA / HST / GST 判断与 COA 校验 | 固定知识的注入 / 缓存 / retrieve 机制留 L4 / seam；COA 字段结构留 L3 |
| Profile 当前批次稳定快照 | industry、business_type、省份、tax config、bank account structure 等结构事实 | 提供客户结构上下文；在其所管维度可压过下层证据 | Profile / Structural Match 尚无正式草案；CJ 不修改 Profile，不把 candidate profile fact 当 stable truth |
| Entity Log | identity basis、`force_pending` | 判断身份可用性和 entity-level automation boundary | Entity Log 不是 classification memory；CJ 只读已落到 CJ 可见标记上的 force_pending，不另读 Governance Log 原始账本 |
| Case Log | stable entity 下的 precedent 摘要、evidence condition、context note、`confirmed_by`、`use_level` | 作为辅助先例和 hard permission cap；支持记忆复用与冲突识别 | Case Log 不是 deterministic rule source；unknown entity 不可作为 case identity handle；exact rollup 机制留 L4 / seam |
| entity-level Knowledge Summary | 按 stable `entity_id` 注入的可读经验 / 背景和 usage 护栏 | 辅助理解 entity 语境 | Knowledge Summary 尚无正式草案；只能作为 readable context，不能替代 Entity Log、Case Log、Profile 或 Rule authority |
| AI external lookup | 针对会计事实的外部来源：商家品类、交易性质、COA / HST 相关公开信息 | 当当前证据不足以判断会计事实时，补充 COA / HST 取证 | 只为会计事实，不定身份；搜索结果列表 / 摘要 / 模型复述只能作 clue，只有可追溯外部来源可作 evidence point |
| [不读] Rule Log | 不读取 active rule payload 或 rule history | Rule Match 负责 rule application；CJ 处理无可执行 rule 的交易 | CJ 不从 Rule Log 推断规则、不生成 rule candidate |
| [不读] Governance Log 原始账本 | 不直接读取治理事件历史 | hard block 只通过上游 / 记忆层已投影到 CJ 可见标记上的结果生效 | CJ 不重新解释 governance event，不绕过 Entity Log / Case Log 的投影边界 |

### Case Log 先例读取与权衡

对 stable entity 的历史先例，Case Log 读取层应把 per-entity case 摘要压成中性、客观事实递给 CJ，例如出现次数、历史分类分布、`confirmed_by` 构成、历史金额区间与本笔金额、历史方向与本笔方向。代码不得输出“强先例 / 弱先例 / 不可信 / 不适用”等判断标签。

这组事实只是 CJ 的辅助证据之一。最终分类由 CJ 综合当前全部证据判断；当前证据与先例冲突时，当前证据优先。纯系统判定的重复历史不能自我累积成 rule authority 或自动化许可。

Case Log `use_level` 必审 / exception-only / 失效、negative case，或 Entity Log `force_pending`，属于硬权限封顶，由代码转 Pending，不作为可被模型权衡后越过的普通事实。

### 批次内记忆一致性

- Case Log 先例读取批起始固定快照；同一批次内 CJ 各次运行互不学习，本批判断不在批内回写、不影响本批后续交易。
- 批内由 ER 新建的 stable entity 应即时可见；CJ 可把该 stable entity 当身份基础。
- 同一新对象在同一批次内多次出现时，各次共享 stable identity，但彼此不构成 case precedent。
- 跨批次学习只经 finalized memory write 发生。

批次定义和一次提交多张表的排序属于 Coordinator / 编排层，不在本节点冻结。

## 4. 写入对象

### 直接执行的 durable write（例外）

本节点直接执行的 durable 写入：

- 无。

### 交给记忆 / finalization 层持久化的内容（运行 / 记忆 seam）

本节点产出、但由记忆 / finalization 层执行写入的内容：本节点只声明“存什么 + 谁有权威认定它有效”，不声明“怎么写、谁来写、什么顺序写”。

- High Confidence Classification 的最终交易结果、结构化 reasoning、AI 高置信度自动完成标记和相关 evidence / external source refs，应由后续 finalization / Transaction Log 机制按其资格保存；有效性来自 CJ runtime judgment、deterministic COA 校验和 finalization 对本次交易完成状态的认定。
- stable-linked 且 finalized 的 High Confidence 交易可形成 reusable case 语义，进入 Case Log 的资格仍由 Case Log / finalization proof 约束；CJ 只在分类输出中携带可学习 case 内容，不授权写入。
- Pending 的 reason、已执行动作、可选候选建议和阻塞语义交 Coordinator 消费；是否以及如何进入长期 audit / interaction record 由 Coordinator / Transaction Log / finalization 设计决定。

### 可学习 case 内容（非独立候选）

- High Confidence Classification 自带 CJ 判断派生的可学习 case 内容，例如 evidence condition、context note、use_level 倾向或 risk hints；它不是独立候选信号，必须等交易 finalized 后由统一 finalization 写入机制凭 `transaction_log_ref` 或等价 finalization proof 沉淀为 Case Log 先例。

### 绝不能写入或修改

- Entity Log。
- Alias Log。
- Rule Log。
- Case Log。
- Transaction Log。
- Governance Log。
- Profile。
- active rule。
- force_pending / promotion_lock。
- entity / alias / role / rule / 控制状态（force_pending / promotion_lock）candidate。
- 交易集合或 split 子交易。

`confirmed_by = system` 的高置信度 case 只是强辅助先例，不自动授予未来自动化放行。

## 5. 决策权限

### Deterministic code 可以决定

- 本节点是否被路由触发，以及当前交易来自 ER direct-to-CJ、Rule Match `rule_miss` 还是 ER `unknown`。
- 固定加载、每笔必取和按需检索上下文的组装。
- ER `unknown` 是否构成身份层 hard block。
- Entity Log `force_pending`、Case Log `use_level` 必审 / 不可复用 / 失效等已结构化标记是否命中 hard block。
- 在 hard block 存在时，无论模型置信度多高，最终输出 Pending。
- 在 hard block 不存在时，是否允许采信模型判断与置信度作为 High Confidence 放行信号。
- High Confidence 输出的 COA 是否命中客户 COA 全集；未命中时触发受控重判，仍失败时转 Pending。
- 最终路由：High Confidence 经 COA 校验进入 JE Generation Node；Pending 交 Coordinator。High Confidence 路径的可学习 case 内容随交易 finalize 经统一 finalization 写入机制沉淀为 Case Log 先例，不再有独立 candidate 或 consumer 节点。
- reason 仅用于解释和定位，不作为多路分流开关。

### LLM 可以判断

- 当前交易主体 / 性质在给定 ER 身份基础下如何理解。
- 当前 evidence、固定知识、Profile 结构事实、Case Log 先例摘要和 Knowledge Summary 是否共同支持唯一、站得住的 (COA, HST/GST)。
- COA 科目与 HST / GST treatment 的运行时选择。
- 当前证据是否不足、是否存在多个合理分类、是否存在软风险或冲突。
- 结构化 reasoning 和自评置信度。
- Pending 中是否有有参考价值的候选分类或建议，以及如何说明不确定项。
- 在判断回合内自行发起联网搜索以补充会计事实证据；联网只能服务 COA / HST / GST，不服务 identity。

### LLM 不能判断

- 最终路由或控制流。
- 系统是否被许可自动落地；许可由代码根据 hard block 与确定性校验裁定。
- 身份是否 stable，或用联网结果反推 / 确认身份。
- 越过 ER `unknown`、Entity Log force_pending、Case Log `use_level` 等 hard block。
- 创建、修改或授权写入 durable memory。
- 创建 entity / alias / role / rule / 控制状态（force_pending / promotion_lock）candidate。
- 把 Case Log precedent、Knowledge Summary、reasoning 或 repeated history 升格为 deterministic rule。
- 构造或校验 JE。
- 批准 accountant / governance decision。

### Accountant 必须决定

- Pending 交易的最终会计处理，尤其是系统无法唯一落到 (COA, HST/GST) 的交易。
- 证据明确指向非公司支出时，个人 vs 公司、费用 vs Shareholder Loan / Owner's Draw 等处理立场。
- 是否接受、纠正或补充当前交易的分类。
- 会计师回答是仅适用于当前交易，还是应沉淀为长期客户记忆、规则、policy 或特殊处理。
- 是否设置 / 解除 force_pending / promotion_lock。

### Governance 必须批准

- entity merge / split、archive / reactivate 等 stable entity lifecycle mutation。
- active rule 的 create / promote / modify / delete / downgrade。
- force_pending / promotion_lock 的设置 / 解除。
- case-derived rule / force_pending / promotion_lock candidate 转成 durable authority。
- 高权限长期状态变更及其 audit trail。

### 置信度与许可分离

模型回答“这笔是什么、我有多确定”。代码回答“系统能不能自己把这笔落地”。两者独立：存在 hard block 时，模型可以给高质量建议，但最终仍是 Pending；不存在 hard block 时，代码才采信模型的置信度与判断结果作为 High Confidence 放行信号。

## 6. 输出类别

字段名可以暂不冻结，但语义类别必须稳定。

本表是本节点对下游唯一的契约面：下游只能依赖此处声明的输出类别，不得依赖本节点未声明的内部状态或实现。

本节点对下游只声明两类分类输出：High Confidence Classification 与 Pending。High Confidence 路径自带的可学习 case 内容随后经统一 finalization 写入机制沉淀为 Case Log 先例，不是第三种分类状态，也不表达为独立候选信号。

| Output Category | 含义 | Consumer（谁消费） | 下游影响 | 不代表什么 |
| --- | --- | --- | --- | --- |
| High Confidence Classification | CJ 在 stable identity、无 hard block、证据足够且可唯一落到 (COA, HST/GST) 的情况下给出的高置信度会计分类。必须携带确定的 COA 与 HST / GST treatment、结构化 reasoning、evidence refs 和必要 source refs。 | JE Generation Node（经 COA 确定性校验后）；后续 finalization / audit / learning | 进入 JE Generation Node 构造分录，不设生成前强制人工审核门；finalized 后由后续机制在 Transaction Log / Case Log 留痕并标注 AI 高置信度自动完成。 | 不代表 JE 已生成成功、Transaction Log / Case Log 已写入、accountant 已批准本次交易、未来自动化已获许可，或 CJ 可以直接写长期记忆。 |
| Pending | CJ 不能或不得高置信度自动落地的结果。包括 hard block、identity unknown、证据不足、HST / GST 不确定、多合理候选、冲突无法干净解决、证据明确指向非公司支出需会计师立场等。必须携带 reason、已执行动作、阻塞点和可选推断性建议。 | Coordinator（唯一 consumer） | Coordinator 负责把 reason、候选、交易 / entity 信息组织成会计师可回答的问题；会计师确认且信息足够后，由 Coordinator 路径转 JE 所需格式或继续追问。 | 不代表独立人工复核第三态，不代表 reason 是路由枚举，不代表 CJ 已开补救药方，不代表 durable authority 或身份类候选。 |

### High Confidence 后路由与 COA 校验

High Confidence Classification 输出后、进入 JE Generation Node 前，必须经过确定性 COA 校验：将 CJ 输出的 COA 科目与该客户 COA 全集匹配。

- 命中：放行进入 JE Generation Node。
- 未命中：由代码告知 CJ“所选 COA 不在客户 COA 全集内”，触发受控重判。
- 重判后仍无法落到合法 COA：输出 Pending，交 Coordinator。

COA 校验落点（CJ 末尾独立检查器 vs 并入 JE Generation Node）、重判次数上限、反馈 payload 和纠错 UX 属 L4 / seam。

### Pending handoff 最小语义

Pending 必须至少表达：

- 客户 / identity 背景。
- 相关 Case Log 先例摘要；仅 stable entity 时存在。
- 阻塞原因：当前卡在哪里、为什么不能判断或不能放行。
- 已执行工作及结果，例如已联网查询但没有可确定结果。
- 可选的推断性建议和不确定项说明。

reason 只解释模型为何输出 Pending，并定位判断断在哪一环；它不开补救药方，也不作为下游分流开关。

## 7. 证据不足时的行为

如果缺少 Case Log 先例：

- 输出：不因“无先例”自动 Pending。
- 下游应：在当前 evidence + 固定知识能唯一落到 (COA, HST/GST) 且无 hard block 时，可 High Confidence。
- 本节点不能：把 Case Log 先例设为 high confidence 的必要条件。

如果身份为 ER `unknown`：

- 输出：Pending。
- 下游应：Coordinator 向会计师确认或呈现 CJ 的非身份上下文建议。
- 本节点不能：联网反推身份、伪造 stable entity、走 High Confidence。

如果当前 evidence、固定知识、先例和可追溯外部来源仍不能唯一落到 (COA, HST/GST)：

- 输出：Pending。
- 下游应：Coordinator 将阻塞点与候选建议组织成会计师问题。
- 本节点不能：为了让 workflow 继续猜一个分类。

如果 HST / GST 处理不确定：

- 输出：Pending。
- 下游应：由 Coordinator / accountant 补足立场或确认。
- 本节点不能：用 COA 置信度掩盖税务处理不确定。

如果联网只得到模糊、同名、低质量聚合或不可追溯结果：

- 输出：Pending 或不采信该 lookup 继续基于其他证据判断。
- 下游应：保留已执行查询与结果等级，供 Coordinator 理解系统做过什么。
- 本节点不能：把搜索摘要、搜索排序或模型复述当 evidence point。

如果证据明确指向非公司支出：

- 输出：Pending。
- 下游应：由 accountant 决定个人 vs 公司及对应处理。
- 本节点不能：因金额小就自动费用化，也不能把 mixed-use 当作普通补证据缺口。

## 8. 歧义处理

如果存在多个合理 COA / HST / GST 候选：

- 输出：Pending。
- 是否允许自动化：不允许 High Confidence。
- 是否需要 pending / review / governance：交 Coordinator；是否进入 review / governance 取决于会计师回复和后续节点，不由 CJ 预设。

如果当前证据显示交易可能是复合交易 / split case：

- 输出：若 (COA, HST/GST) 不能唯一落地，则 Pending，并在 reason 中说明疑似复合点。
- 是否允许自动化：CJ 不执行拆分；只有整笔能唯一合法分类且无 hard block 时才可能 High Confidence。
- 是否需要 pending / review / governance：拆分检测与执行归 CJ 外部，当前为 DEFERRED。

如果 mixed-use / 个人消费只是“可能夹带私人但分不清”：

- 输出：在其他条件满足时按公司支出正常分类，可 High Confidence。
- 是否允许自动化：允许；CJ 不主动稽查。
- 是否需要 pending / review / governance：只有证据明确指向非公司支出时才 Pending。

本节点不能为了让 workflow 继续而猜一个 winner。

## 9. 冲突处理

多源冲突按以下 authority 顺序裁定，从高到低：

1. 第 0 层：硬权威阻断。包括 ER `unknown`、Entity Log `force_pending`、Case Log `use_level` 必审 / 不可复用 / 已失效。命中即封顶为 Pending，当前证据再强也不能越过。CJ 不另读 Governance Log / Rule Log 原始账本。
2. 第 1 层：Profile 结构事实 + 限定范围的 accountant confirmation。Profile 在其所管维度有决定权；Case Log 中 `confirmed_by=accountant` 的明确确认只在其确认范围内有效，模糊批注不泛化。
3. 第 2 层：当前交易证据，例如小票、支票、本笔 accountant note。
4. 第 3 层：Case Log 先例。
5. 第 4 层：通用知识 + Knowledge Summary。

如果 current evidence 与更高层 authority 冲突：

- 本节点行为：不能自动越过更高层；无法干净解释时输出 Pending 并说明冲突点。
- 是否生成 review / governance candidate：CJ 不生成治理候选；后续 Coordinator / Review / Governance 路径可消费冲突语义。
- 是否阻断自动化：影响 hard block 或核心分类时阻断 High Confidence。

如果同层两个来源互斥且都说得过去：

- 本节点行为：输出 Pending。
- 是否生成 review / governance candidate：CJ 不生成候选；只交出 reason / evidence refs。
- 是否阻断自动化：阻断 High Confidence。

如果 Case Log 先例与当前证据冲突：

- authority 顺序：当前证据优先。
- 本节点行为：当前证据足以唯一落地且无 hard block 时，可按当前证据分类；若冲突导致无法干净收敛，则 Pending。
- 是否阻断自动化：视冲突是否可按 authority 顺序干净解决。

如果 Knowledge Summary 与 Entity Log / Case Log / Profile 冲突：

- authority 顺序：Knowledge Summary 垫底。
- 本节点行为：不得用 Summary 覆盖 source authority；必要时 Pending 并说明冲突。
- 是否阻断自动化：若冲突影响分类且无法干净解决，则阻断 High Confidence。

保守原则：歧义不猜，冲突不挑顺眼版本；无法在上述顺序内干净解决时，一律 Pending。

## 10. Audit / Trace 边界

本节点应保留的 trace：

- `transaction_id`。
- 输入身份状态与 ER identity reason；CJ 只引用，不重新论证。
- 当前 evidence refs 和关键证据解释。
- Case Log 先例摘要使用情况；只到 entity 概要级，不点具体 case 行作为逐条引用。
- 如果依赖联网外部来源，保留可追溯 external evidence reference；exact URL / title / retrieved_at / snapshot 字段留 L3。
- COA 与 HST / GST 的结构化 reasoning。
- 模型自评置信度；置信度可进入论证，但不能替代论证。
- hard block / Pending reason。
- COA 校验失败和受控重判结果；exact payload 留 L4。
- High Confidence 路径可学习 case 内容的 runtime refs；exact schema 留 L3。

结构化 reasoning 应是一条收敛到 (COA, HST/GST) 的论证链：

```text
ER 身份前提
-> 当前证据凭据论证
-> entity 历史佐证（如有）
-> COA 与 HST / GST 结论
```

这些 trace 用于：

- review。
- correction。
- governance。
- audit。
- finalization / learning context。

这些 trace 不能成为：

- entity authority。
- Alias authority。
- rule authority。
- case authority。
- accountant approval。
- governance approval。
- automation permission。

reasoning 始终是 runtime explanation，不是 durable authority、rule 或 approval。

## 11. Legacy Constraint Translation

仅记录已经脱离来源、按当前产品目标重新论证后的取舍。New system / Old system 材料不构成本节点 authority。

可借鉴保留的内核：

| Retained Constraint | 来源 | 为什么现在仍成立 |
| --- | --- | --- |
| JE Generation 应是纯确定性构造层，不做分类判断 | 历史材料中的 JE 构造边界 | 当前产品仍需要把会计判断集中在 CJ，把分录构造 / 格式校验集中在 JE Generation，以保持单一职责和审计可追溯 |
| High Confidence 可直接进入 JE，不设生成前强制人工门 | 历史材料中的自动分类后路径 | 当前产品目标需要保留自动化率；hard block 已由 CJ 两态输出和代码许可 gate 承载 |
| 分类判断不是自主 agent loop，最终路由由代码裁定 | 历史材料中的分类器边界 | 当前产品需要把 automation permission 建成确定性保障，不依赖模型听话或自我约束 |
| Pending 必须能转成人能回答的问题 | 历史材料中的 Coordinator 意图 | 当前产品需要 accountant control；CJ 只交 reason / 候选 / context，问题组织归 Coordinator |

明确不继承的旧行为：

| Old Behavior | 不保留原因 |
| --- | --- |
| 使用旧的自动放行命名 | 当前命名已锁为 High Confidence Classification |
| 把 hard block 表达为独立人工复核第三态 | 当前契约只保留 High Confidence Classification / Pending；review / governance 卡点以 Pending reason 呈现 |
| 身份类候选 / 多类治理候选由 CJ 直接产出 | 单次 CJ runtime judgment 不能越权进入身份、规则或治理 authority；CJ 的长期记忆贡献只有 High Confidence 路径携带的可学习 case 内容一个出口 |
| mixed-use 零售风险默认降级或设 materiality gate | 当前姿态已锁为默认公司支出、不主动稽查；证据明确非公司即交回会计师，不因金额小自动费用化 |
| 用旧系统字段、pattern、risk-pack 或路由状态机作为当前 baseline | 字段级 schema、路由机制和存储形态均未冻结，必须在 Stage 3 / L4 重新收口 |

## 12. Open Boundaries

### C1. Accountant / Governance authority boundary

以下权限边界已在 Stage 1-2 成文，进入 Stage 3 前需与外部对象复核一致：

- CJ 不做 durable final accounting confirmation；High Confidence 是 runtime judgment，经 downstream finalization 才形成最终交易记录。
- CJ 不把 runtime judgment、LLM reasoning、Case Log precedent 或 Knowledge Summary 升格为 durable authority。
- CJ 不批准 entity merge / split、rule promotion、force_pending / promotion_lock 设置 / 解除 或 governance event。
- accountant 必须决定 Pending 的最终会计处理、个人 vs 公司立场、correction，以及某次说明是否应沉淀为长期 memory / rule / policy。
- governance 必须批准长期 authority 变化。
- CJ 的唯一长期记忆出口是 High Confidence 路径携带的可学习 case 内容，交易 finalized 后由统一 finalization 写入机制沉淀为 Case Log 先例；CJ 自身不直接写入长期记忆，也不另产独立候选。

### 未冻结问题

1. `case_judgment_input` / `case_judgment_result` exact field schema、status enum。
2. accounting treatment 字段结构、COA / HST / GST 表达方式、JE Generation 所需完整性口径。
3. Pending handoff / `pending_request_context` exact field schema。
4. Entity Log `force_pending` / `promotion_lock`、Case Log `use_level` / `confirmed_by` exact enum。
5. hard block 的结构化字段 / flag / enum 形态。
6. 固定加载知识的注入 / 缓存 / retrieve 机制和上下文组装顺序。
7. Case Log per-entity rollup / retrieval pack 生成机制，以及 rollup 的 exact 字段 / 阈值。
8. entity-level Knowledge Summary 的正式边界、co-read source memory 和注入 contract。
9. Profile / Structural Match 的正式边界，尤其 identity-independent 结构性交易的处理合同。
10. JE Generation Node 的正式边界，以及 CJ 与 JE 间的 accounting treatment / COA 校验 seam。
11. COA 校验落点、重判次数上限、反馈 payload 和纠错 UX。
12. external lookup evidence reference exact field schema、网页快照 / 缓存 / retrieval 机制、source quality taxonomy。
13. reasoning 在 Transaction Log 的 exact 存储字段、后续治理节点读取 reasoning 的 contract。
14. High Confidence 路径可学习 case 内容的 exact schema、经统一 finalization 写入机制沉淀为 Case Log 先例的写入机制、Case Log 与 Transaction Log finalization trigger order。
15. Coordinator / Pending Node 的提问模板、追问策略、reason / 候选组织机制、会计师确认后转 JE 格式 contract。
16. High Confidence 结果的事后查看 / 抽查 UI。
17. 批次定义和多 Bank Statement 提交的排序 / 快照机制。
18. 结构性无效 / 损坏输入的拒绝与否、无解交易最终归处、纠错 candidate signal 与纠错流程。
19. 交易拆分的检测、执行和子交易重入路径；CJ 不检测也不执行 split。
20. Coordinator、Knowledge Summary、Profile / Structural Match、JE Generation、Evidence Log、Transaction Log、Governance 等圈外对象的正式草案。

这些问题解决前，不能进入：

- [x] Stage 3 data contract
- [x] Stage 4 execution algorithm
- [x] implementation
