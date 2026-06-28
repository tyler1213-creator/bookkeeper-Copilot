# Evidence Intake Node - Logic and Boundaries

## 1. 触发条件

本节点在以下条件同时满足时触发：

- 任一 runtime / onboarding / supplemental 来源的新材料进入系统。
- 这些材料需要被组织成可追溯 evidence foundation。
- 下游尚不能安全消费未处理原文，或交易尚未获得 objective transaction basis 与 `transaction_id`。

本节点不得在以下情况触发：

- 无。Evidence Intake 是所有交易进入系统运行前的必经入口，任何交易进入系统都必须先经过 Evidence Intake。

## 2. 上游前置条件

上游必须已经完成：

- 原始材料已进入当前 batch intake scope，或作为 supplemental / onboarding source material 被交给本节点。
- batch intake context 可读，至少能标明本批来源、上传 / 导入上下文与 source provenance。
- 已存在 Evidence 引用如被提供，只作为只读 provenance / duplicate-preservation context 使用。

如果前置条件缺失：

- 本节点行为：缺可追溯源材料时，不产出 transaction-ready 交易，不进入第 9 步铸号；能保全的材料保全为 source issue / evidence pool context，不能保全且缺少产品级来源 contract 时输出 invalid intake context 语义。
- 是否 stop-and-ask：如果缺失来自未冻结的跨对象 contract 或 consumer 归属，停止并要求产品设计决策；不能由实现代理猜。

## 3. 读取对象

| Source | 读取内容 | 用途 | Authority 限制 |
| --- | --- | --- | --- |
| Raw / source material | bank / payment account import line、bank statement、receipt、cheque、invoice、contract、client / accountant note、historical attachment 等原始材料 | evidence foundation、文档类型识别、客观字段投影、surface signal 提取、trace anchor | raw/source 是 source of truth；OCR / parser / LLM extraction 只是不覆盖原文的 derived signal |
| Batch intake context | 当前批次来源、上传 / 导入上下文、`source_channel` provenance、batch scope | 区分 runtime / onboarding / supplemental 来源，组织本批材料 | context 只说明来源与批次，不提供会计结论、身份 authority 或 durable registry |
| 已存在 Evidence 引用 | 已保全材料的 refs / source relationship，如当前 batch 已知重复引用 | 避免重复保全、保留 source relationship | 只读；不得把 Evidence 内部 metadata 扩展为未声明 runtime contract |
| External import context | feed / 导入通道 / 外部来源 provenance | 记录材料来自哪里 | 只读为来源 provenance；本节点不执行 external lookup，不裁定“这是谁” |
| [不读] Transaction Log | 无 | 无 | 本节点不查历史交易、不复用既有 `transaction_id`、不做跨导入 dedupe |
| [不读] Entity / Case / Rule / Governance memory | 无 | 无 | 本节点不读取身份、case、rule 或 governance authority 来做运行时判断 |

## 4. 写入对象

### 直接执行的 durable write（例外）

本节点直接执行的 durable 写入：

- 无。

### 交给记忆 / finalization 层持久化的内容（运行 / 记忆 seam）

本节点产出、但由记忆 / finalization 层执行写入的内容：本节点只声明“存什么 + 谁有权威认定它有效”，不声明“怎么写、谁来写、什么顺序写”。

- 应持久化的逻辑语义：raw evidence 保全引用、evidence foundation、objective normalization context、association / quality context、`transaction_id` 及其与放行交易 / evidence refs 的绑定。
- authority 来源：raw/source 存在性 + 本节点 intake validation + 第 9 步运行时铸号。
- `EvidenceLogRecord` 只能表示待持久化内容的逻辑语义，不是 implementation-ready storage contract。
- `transaction_id + objective transaction basis + evidence binding` 应在 Transaction Log finalization 或等价最终化路径中持久化；本文不冻结该机制。

### 只能提出 candidate

- `candidate_association`：证据与交易之间尚不能 confirmed 的候选关联，属于证据层候选，不是身份状态。
- `duplicate_material_candidate`：文件 / 材料层疑似重复，不能当作交易级 dedupe 裁决。
- alias / entity / role candidate signal：来自 raw surface 或 note 的候选线索，只能交 ER / governance 路径消费，不能成为 approved alias、stable entity 或 role authority。

### 绝不能写入或修改

- 交易级 dedupe / same-transaction grouping 结果。
- 跨导入 `transaction_id` 复用或 durable transaction registry。
- stable entity、approved alias、approved role 或任何身份状态。
- Profile account mapping、COA、HST / GST、JE。
- rule / case 结论、rule candidate approval 或 case authority。
- Transaction Log。
- Evidence Log storage mechanism / write order / finalization / rollback。
- durable business memory。

## 5. 决策权限

### Deterministic code 可以决定

- 文件 hash / content fingerprint 与已知格式识别。
- 结构化 parser 对已知 bank statement 格式的解析。
- 金额、日期、currency、direction、source account reference 等客观字段归一。
- 某记录是否满足 transaction-ready 资格。
- 单张 Bank Statement 是否整体放行，包含行级 readiness 与余额连续性体检。
- 余额链账期级查漏 / 查重信号。
- `cheque_number` 等确定性共享键在单张 BS 内的精确匹配。
- material duplicate 标记。
- 第 9 步为本次运行已放行交易铸造新的永久唯一 `transaction_id`。

### LLM 可以判断

- 文档类型识别和无法识别材料的可读说明。
- 非结构化材料中的人类可见表面信息。
- 陌生格式 Bank Statement 的解析兜底。
- 在单张 BS / 归属账期内综合金额、日期、vendor 等语义，给出 receipt / cheque / invoice / contract evidence 与 bank line 的关联建议。
- evidence issue 的 human-readable 解释。
- 仅当证据在本张 BS / 归属账期作用域内唯一且无歧义时，输出 `confirmed_objective_association`；否则只能输出 `candidate_association`、issue 或转证据池。

### LLM 不能判断

- 当前材料是否进入 pending / review / 自动化路径。
- 当前交易是否分类、如何分类、或是否构成 routing outcome。
- 把 evidence issue 写成 accountant-facing question，或替系统决定该问什么。
- 在多个合理候选、字段打架、证据缺失或无法解释冲突时选择 winner。
- 把 source_actor_type=`accountant` 的 note 当作 accountant approval。
- 生成 stable entity、approved alias、rule、case conclusion、COA / HST / GST / JE。

### Accountant 必须决定

- client / accountant note 是否构成已批准客户政策、会计结论、approved alias、rule 或分类依据。
- evidence issue 或 pending context 被呈现后，人工回答是否足以形成当前交易结论或长期客户记忆。

### Governance 必须批准

- 任何长期身份 / alias / rule / force_pending / promotion_lock 变化。
- merge / split、rule promotion / modification / deletion / downgrade、force_pending / promotion_lock 设置 / 解除等高权限 lifecycle mutation。

## 6. 输出类别

字段名可以暂不冻结，但语义类别必须稳定。

本表是本节点对下游唯一的契约面：下游只能依赖此处声明的输出类别，不得依赖本节点未声明的内部状态或实现。证据与交易配对是 Evidence Intake 内部处理工序，不作为独立输出类别；confirmed 的关联结果随 `EvidenceFoundationHandoff` 并入放行交易包，candidate / unassociated 证据转证据池。

| Output Category | 含义 | Consumer（谁消费） | 下游影响 | 不代表什么 |
| --- | --- | --- | --- | --- |
| `EvidenceFoundationHandoff` | 放行交易包的 evidence foundation，包含本次放行交易已 confirmed 的 evidence-to-transaction 关联结果、evidence refs、证据存在性事实和 quality context。 | Profile / Structural Match（圈外，只挂起引用，不替它冻结 spec）-> Entity Resolution -> Rule Match | 运行层可基于同一 evidence foundation 继续 structural、identity、rule 判断；confirmed association 只随放行交易包流动。 | 不代表独立配对输出，不代表 evidence sufficiency、accounting outcome、identity、rule authority 或 durable write 已完成。 |
| `ObjectiveNormalizedTransactionBasis` | date、amount_absolute、direction、source_account_ref、raw_description、currency 等客观交易事实语义；exact field schema 留 L3。 | Profile / Structural Match（圈外，只挂起引用，不替它冻结 spec）-> Entity Resolution -> Rule Match | 下游获得可确定性消费的客观事实；Rule Match 后续桶 C 可消费 transaction_id、direction、amount、date、currency、evidence refs / condition 等。 | source_account_ref 不是 COA，不是已确认 Profile account id；raw description 只作 trace，不作 Rule Match condition。 |
| `transaction_id` | 第 9 步为本次运行已放行交易铸造的永久唯一 id。 | Entity Resolution；JE Generation（圈外，只挂起引用，不替它冻结 spec）；Transaction Log finalization（圈外，只挂起引用，不替它冻结 spec）；Case / Review / Correction lineage（圈外，只挂起引用，不替它冻结 spec） | ER 用它绑定 identity request；JE 用它关联分录；finalization 用它持久化 id + evidence + outcome 绑定；case / review / correction 用它回指具体资金事件。 | 不代表跨导入复用、交易级 dedupe、durable registry、账务幂等或 Transaction Log 已写入。 |
| `EvidenceQualityIssueSignal` + human-readable summary | 描述证据层缺口、冲突、来源不可读 / 不可定位、字段缺失、自洽性破裂、未识别材料等，并保留 source refs。 | Coordinator / Pending（圈外，只挂起引用，不替它冻结 spec）；Review（圈外，只挂起引用，不替它冻结 spec） | 下游可把 issue 作为 pending / review context。 | 不生成 accountant-facing question，不决定 pending / review outcome；“对手是谁不清楚”不是本节点 issue。 |
| counterparty surface signals | raw bank text / descriptor、receipt vendor、cheque payee、invoice / contract party、accountant context refs 等离散可追溯 identity surface signals。 | Entity Resolution | ER 可用于 identity 判断和候选线索分析。 | 不排序、不择一、不升级为 stable entity、approved alias、role、identity state 或 accounting conclusion。 |
| raw / source refs | 可追溯原始材料引用、raw text fragment anchor、source provenance。 | 后续 audit / memory workflow（圈外，只挂起引用，不替它冻结 spec）；所有下游通过 evidence_refs 追溯 | review、correction、governance、audit 可回到原始材料核验。 | 不变成 durable business memory；不替代 Entity / Rule / Case / Governance authority。 |
| 实际唯一覆盖信号 | 去重后 EI 实际收到且未判冗余的账户 x 月份集合。 | interaction_agent（圈外，只挂起引用，不替它冻结 spec） | 目标层可与 engagement 期望清单比对，核对首尾漏月、缺账户或多发重复。 | EI 不持有 engagement 目标，不下“齐没齐”结论，不向客户提问。 |
| balance-chain gap / overlap signal | 银行余额链产生的中段缺口 / 重叠信号，以及每账户 `statement_chain_status` 的语义类别。 | interaction_agent（圈外，只挂起引用，不替它冻结 spec） | 目标层可消费确定性中段缺口 / 重叠提示。 | 不自行决定丢哪张账单、补哪期材料或向客户提问。 |
| `duplicate_material_candidate` | 文件级 hash / content fingerprint / source row 层面的疑似重复材料标记。 | 材料层标记 / 阻断；纠错接缝（圈外，只挂起引用，不替它冻结 spec） | 可阻断或标记重复材料，避免重复保全和重复处理。 | 不代表交易级 dedupe；候选确认 / 关闭归纠错接缝，exact mechanism 留 L4。 |

Evidence Intake 只输出客观的证据存在性事实，例如有无 receipt / invoice / cheque / 仅 bank line 及各自 refs；不输出 Case / Rule 意义上的 evidence sufficiency。`evidence_condition` 的 exact schema 与 sufficiency threshold 归 Case / Rule 侧。

### Balance-chain L2 语义

余额链使用银行自带余额做确定性的账期级完整性、查漏和查重检查，按账户分组，不跨账户串联。其 L2 语义是：

- 单张账单内部自洽：期初余额、逐行带符号金额、期末余额及逐行 running balance 如存在，应组成连续链；断口或对不上时整张 BS 挂起。
- 跨相邻账单连续：同一账户账单按账期排序，检查相邻期末 / 期初余额连续；断口输出 gap signal，账期相同或重叠输出 overlap / duplicate statement signal。
- 输出 `statement_chain_status` 语义类别：continuous、gap、overlap、unverifiable。exact enum 取值与公式规则留 L3。
- 无余额时标记 unverifiable；intra 侧退化为任一行不达标即整张挂起，coverage 侧输出实际覆盖信号交 interaction_agent。
- 余额链只检测，不擅自解决；它不能抓 engagement 最头 / 最尾漏单，首尾与缺账户的期望比对归 interaction_agent。

### `transaction_id` 第 9 步铸号边界

- 只为本次运行通过放行闸门的交易铸造永久唯一 id。
- 不查历史、不读 Transaction Log、不访问 durable registry、不复用既有 id。
- 同一批里两条长得一样但未被文件 hash / 余额链挡住的交易分别发新号。
- 不保证跨运行账务幂等；durable transaction registry、迟到证据接回、跨运行更正定位和跨运行账务幂等均为 DEFERRED / seam-park，见 `缺口地图.md` 的 Evidence Intake section。

## 7. 证据不足时的行为

如果缺少 transaction-ready 核心字段或账单自洽性破裂：

- 输出：整张 Bank Statement 挂起的 issue / correction-path handoff 语义；已能保全的 raw/source refs 和 objective extraction context 必须保留。
- 下游应：纠错路 / Review / Pending 等圈外路径消费 issue；具体机制未冻结。
- 本节点不能：放行其中部分行、伪装成干净交易、给未通过闸门的材料铸 `transaction_id`。

单张 Bank Statement 的放行边界是全有或全无：若该 BS 每一行都 transaction-ready 且账单内部自洽，整张 BS 的交易一起放行；若任一行不达标或余额链 / 解析自洽性破裂，整张 BS 全部挂起。同一次上传的其它 BS 不受影响。

如果缺少可追溯锚点但材料本身有效：

- 输出：证据池待匹配语义，保留 source refs。
- 下游应：等待对应账单 / 锚点或纠错接缝处理。
- 本节点不能：把无锚点材料当作交易、铸号、删除或猜测归属。

supplemental evidence 当前只保全 + 输出客观 basis / association candidate / issue；不查历史、不复用既有 `transaction_id`、不覆盖既有 outcome、不写 memory。接回既有交易、amendment、reprocessing 和 review trigger 依赖未来 durable transaction registry，当前 DEFERRED。

## 8. 歧义处理

如果存在多个合理候选：

- 输出：`candidate_association` + issue context，或转证据池。
- 是否允许自动化：本节点不决定自动化；不得为了继续 workflow 选择 winner。
- 是否需要 pending / review / governance：本节点只暴露 issue / candidate；是否进入 pending、review 或 governance 由下游 consumer 决定。

本节点不能为了让 workflow 继续而猜一个 winner。

Evidence-to-transaction association status 只有以下语义类别：`confirmed_objective_association`、`candidate_association`、`unassociated`、`duplicate_material_candidate`。这些是语义类别，exact enum 取值留 L3。

## 9. 冲突处理

如果 evidence-to-transaction 配对中金额 / 日期 / vendor 等字段打架且无法解释：

- authority 顺序：双方 raw/source refs 都必须保全；本节点不裁决谁覆盖谁。
- 本节点行为：归 `candidate_association`，保留双方原文与 refs，不设独立 `conflicting_association` 状态。
- 是否生成 review / governance candidate：可暴露 association candidate context 给下游；除非证据本身另有缺口 / 矛盾，不走 `EvidenceQualityIssueSignal`。本节点不批准、不关闭。
- 是否阻断自动化：本节点不决定自动化；未 confirmed 的关联不进入放行交易包。

如果证据本身存在缺口、矛盾、不可读、来源不可定位或字段缺失：

- authority 顺序：raw/source 是 source of truth；derived extraction 不得覆盖原文。
- 本节点行为：输出 `EvidenceQualityIssueSignal` 与 human-readable summary。
- 是否生成 review / governance candidate：交 Coordinator / Pending 或 Review 消费；本节点不生成 accountant question，不决定 routing outcome。
- 是否阻断自动化：本节点只说明 evidence condition；下游决定路径。

“对手是谁不清楚”不属于 Evidence Intake issue。只要 raw description 存在且其它 transaction-ready 条件满足，该交易可以放行；counterparty / entity 识别归 ER / Case / Coordinator。

## 10. Audit / Trace 边界

本节点应保留的 trace：

- raw evidence refs 与 source provenance。
- 每个客观字段回源材料的 trace anchor。
- objective extraction 状态与 derived extraction context。
- confirmed / candidate / unassociated / duplicate material 的 association context。
- quality issue context 与 human-readable summary。
- `transaction_id` 与本次放行交易 / evidence refs 的绑定语义。

这些 trace 用于：

- review。
- correction。
- governance。
- audit。
- downstream finalization / lineage。

可追溯强制项是“客观字段可回溯源材料”；行号不是强制项。代码解析已知格式可以带行号；模型兜底陌生版面时，用原始文字片段作为锚点，不让模型数行号。

这些 trace 不能成为：

- entity authority。
- rule authority。
- case authority。
- accountant approval。
- governance approval。
- business memory authority。

accountant context refs 只表示“某句话来自会计师或上下文材料”，不等于 accountant approval 或 governance approval。

## 11. Legacy Constraint Translation

仅记录已脱离旧来源、按当前产品目标重新论证后仍成立的取舍；旧系统材料只作为被审计对象，不构成 authority。

| Retained Constraint | 来源 | 为什么现在仍成立 |
| --- | --- | --- |
| 代码优先识别 / 解析，LLM 兜底 | DP §1、§5 Step 2 | 成为 deterministic code 与 LLM 的权限边界：hash、格式识别、结构化 parser、金额日期归一、cheque_number 精确匹配、余额链检查交给代码；文档类型识别、非结构化读取、陌生格式解析兜底、receipt-to-line 语义配对、issue 解释交给 LLM |
| 不丢弃任何文件，无法处理必须标记 | DP §4、§8 | 成为全量保全、不达标 / 未识别材料转证据池或纠错路、不删不丢的不变量 |
| 不确定不猜，保留待确认标记 | DP §4、§5 Step 2 | 迁移为放行资格闸门与“唯一且无歧义才 confirmed”：缺核心字段者整张 BS 挂起，不伪装成干净交易 |
| cheque_number 优先、金额日期兜底的支票关联 | DP §5 Step 3 | 迁移为 cheque_number 精确匹配 = `confirmed_objective_association`（代码、单 BS 内）；receipt-to-line 只有唯一且无歧义时 confirmed，否则 candidate / issue / 证据池 |
| 完整性校验 | DP §5 Step 7 | 账期完整性迁移为单张 BS 全有或全无放行 + 余额连续性体检；重复文件为 `duplicate_material_candidate`；未识别附件转证据池 / 纠错路 |
| amount 归一为绝对值、direction 独立表达、direction 不可靠者不进标准集 | DP §7 | 迁移为 ObjectiveNormalizedTransactionBasis 的 amount_absolute + direction 客观投影规则 |
| `transaction_id` 分配 | DP §5 Step 5 | 保留为 Evidence Intake 第 9 步：只为本次运行放行交易铸造永久唯一 id；不跨导入查重、不复用历史 id、不建立 registry |

不保留的旧行为：

| Old Behavior | 不保留原因 |
| --- | --- |
| 交易级 dedupe / same-transaction grouping | 交易级去重取消；当前只保留文件 hash + 账单/账期余额链。跨导入复用、迟到证据接回、跨运行幂等依赖 durable registry，当前 DEFERRED |
| canonical description 生成 + Pattern Dictionary 读写 | 身份 / 别名语义已迁入 Entity 圈；本节点只保全 raw surface signals，不建长期身份 authority |
| bank_account -> `profile.bank_accounts[].id` 映射写入交易 | 映射涉及 Profile / Structural Match；本节点只输出 source_account_ref，非 COA、非已确认 Profile id |
| 把 receipt / cheque 写入 transaction 的 `receipt` / `cheque_info` 字段 | 改为内部 evidence-to-transaction 关联工序；confirmed 结果并入 `EvidenceFoundationHandoff`，不在 Evidence 层确立会计结论 |
| 生成摘要并直接与 accountant 交互、处理待确认项 | 人工交互归 Coordinator / Pending / Review；本节点只输出 issue signal |
| 端到端单 Agent 写出最终数据集 | 违反运行 / 记忆 seam；本节点只声明应持久化内容和 authority，不定义写入机制 |

## 12. Open Boundaries

以下问题未冻结：

1. Evidence Intake 的未冻结 L3 / L4 / seam / L2 external blocker / DEFERRED 项唯一权威池是根目录 `缺口地图.md` 的 Evidence Intake section；本文档不复制完整清单。
2. EI L2 提案 2.5.2 的“九步”只是排序约束示意：铸号在放行闸门之后、余额链体检在 receipt 配对之前、放行闸门覆盖第 3-8 步。exact 执行序列 / routing sequence 归 Stage 4。

这些问题解决前，不能进入：

- [x] Stage 3 data contract
- [x] Stage 4 execution algorithm
- [x] implementation
