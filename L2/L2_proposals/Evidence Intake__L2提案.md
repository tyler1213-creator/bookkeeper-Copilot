> ⚠️ **已过时 / superseded（owner 2026-06-27）** — 本 L2 提案的 L1 / L2 结论已由 `BK_Copilot/workflow_nodes/evidence_intake_node/` 正式草案取代，仅作历史来源保留。当前权威以 `BK_Copilot/` 正式草案 +（产出后）对应 L3 schema 为准；**勿据本文件回灌已删除的概念 / 字段**。

# Evidence Intake L2 提案

> 本提案由审计窗口独立完成，结合：老系统 `data_preprocessing_agent_spec_v3` 的入口纪律、`new system` 中 `evidence_intake_preprocessing_node__design` 的待论证设计、以及 BK_Copilot 已锁正式下游契约（ER / Alias Log / Entity Log / Rule Match / Case Log / Rule Log）。
> 拟改目标文件命名建议为 `BK_Copilot/workflow_nodes/evidence_intake_node/00|01|02`（与 `entity_resolution_node`、`rule_match_node` 命名风格一致），最终命名在创建正式文档时确认。
> 边界：本轮只产出提案，不创建/编辑任何 BK_Copilot 正式文档，不碰 Entity 圈文件，不碰 new/old system；不定义 Evidence Log 存储/写入机制。

---

## 1. 依赖

### 已锁上游（被本节点消费）
- **Runtime / onboarding / supplemental 原始材料**：bank/payment account import line、receipt、cheque、invoice、contract、client/accountant note、historical attachment、external import context。这些是 source of truth；OCR/parser/LLM 提取只是 derived signal，不得覆盖原文。
- **当前 batch intake context 与已存在 Evidence 引用**：用于避免重复保全、保留 source 关系。**只读，不得读取 `Transaction Log` 作为 runtime source。**

### 下游（每个输出标 consumer；未冻结字段 / seam 统一登记于 `缺口地图.md` 的 Evidence Intake section）
- **Entity Resolution Node** ← 由 Evidence Intake 直接放行、且已带 `transaction_id` 的 objective transaction basis、traceable evidence foundation / `evidence_refs`、counterparty surface signals（raw bank text、receipt vendor、cheque payee、invoice/contract party、accountant context refs）。只作 identity 判断输入，不是 stable entity / approved alias；`transaction_id` 来源为 Evidence Intake 第 9 步，不来自任何单独身份节点。
- **Profile / Structural Match Node（圈外挂起引用）** ← 客观交易字段、source account refs、evidence refs、quality context。source account 不得被误读为 COA。
- **Alias Log（间接，经 ER）** ← raw description / descriptor / cheque payee 等身份表面字段的可追溯 refs。只能作候选线索，不能写成 approved alias。
- **Entity Log（间接，经 ER / accountant confirmation）** ← evidence refs、creation provenance 所需的可追溯证据根。
- **Rule Match Node** ← handoff 中客观交易事实 + `evidence_condition` / `evidence_refs`（raw description 只作 trace，不作匹配条件）。
- **Case Log / Rule Log（间接）** ← 可追溯 evidence refs 与证据存在性，用于 finalized case / promotion provenance；不消费 raw blob。
- **Coordinator / Pending、Review（圈外挂起引用）** ← missing / conflicting / unmatched / low-quality evidence issue signals。本节点只暴露 issue，不生成 accountant question 或 routing outcome。
- **纠错路 / 证据池（圈外挂起引用，无专门纠错 Agent）** ← 整张挂起的 BS（不达标/自洽性破裂）、未识别材料、无锚点待匹配证据。本节点只把它们保全并标记移交，不定义纠错机制与证据池存储（L4-11）。证据池=有效等锚点、纠错路=破损需返工（见 2.4(4)）。
- **交互 / 目标层 Agent（圈外挂起引用）** ← (a) 去重后的实际唯一覆盖信号（EI 实际收到且未判冗余的 账户×月份 集合）；(b) 余额链产出的中段缺口 / 重叠信号（见 2.4.1）。本节点只输出客观覆盖事实，不持有 engagement 目标、不下"齐没齐"结论、不向客户提问；期望比对与查缺提问归交互 / 目标层 Agent。
- **后续 audit / memory workflow（圈外挂起引用）** ← 可回溯 evidence references 与当时客观提取状态。不能变成 durable business memory。

---

## 2. L2 决策

### 2.1 节点唯一核心职责：可追溯证据保全 + 最小客观交易结构投影 + 证据质量闸门（对应 L1-01、L1-04）

**结论：**
Evidence Intake 保留为独立 workflow node。它的唯一核心职责不是单纯“证据保全入口”，也不是单纯 parser，而是四件不可分割的事的组合：(a) 把一切进入系统的原始材料纳入可追溯保全（不丢任何材料）；(b) 从中投影出**最小客观交易结构**（date、amount_absolute、direction、source_account_ref、raw_description、currency 等可确定性读出的客观事实）；(c) 充当**放行资格闸门**：只把满足放行资格（见 2.4）的干净、完整交易记录放入下游交易流，达不到资格的材料不进入交易流，转证据池 / 纠错路保全挂起；(d) 在流程第 9 步为每笔放行交易分配永久、唯一的 `transaction_id`。importer / parser / preservation / id minting 是本节点内部实现层，不拆成独立 workflow node。

**为什么（锚定核心产品目标）：**
锚定“有证据支持的建议”“审计性”“自动化率提升”。只有当客观事实能回溯到具体材料、且本节点先把不达标材料挡在交易流之外，下游 ER / Rule Match / JE / Transaction Log 才能在不重新解析原文、且收到的都是完整交易和稳定交易锚点的前提下做确定性判断。若退化成“纯格式转换”，本节点对核心产品目标无独立贡献，会沦为架构完整性节点——这正是约束摘要禁止的。

**拟改：**
- `evidence_intake_node/01_functional_intent.md:为什么这个节点必须存在` → `若删除/合并/内联本节点，会失去：把混乱原始材料变成可追溯、可被下游确定性消费的客观交易事实的统一入口；以及在交易进入下游前、确保只有干净完整交易记录被放行的资格闸门。`
- `evidence_intake_node/01_functional_intent.md:核心职责` → `唯一核心职责：把进入系统的原始材料保全为可追溯 evidence foundation，投影出最小客观交易结构，按放行资格判定该记录能否作为交易进入下游交易流，并在第 9 步为每笔放行交易铸造永久 transaction_id。`
- `evidence_intake_node/01_functional_intent.md:明确排除范围` → `不负责：交易级去重、跨导入 transaction_id 复用、durable registry、entity / counterparty 身份、会计分类 / HST/GST / JE、rule / case 判断、accountant question 生成、durable business memory 写入。`

**排除的替代 + 理由：**
- 把本节点降级为“纯 evidence 保全入口”，把客观结构投影推给 ER / Rule Match / JE 等下游。理由：ER / Profile / Rule Match 都直接依赖 `objective transaction basis`，若该投影不在 Evidence Intake 完成，会让多个下游各自重复解析原文，破坏“摘要不能替代 raw ref”的不变量。
- 把 importer / parser 拆成独立 workflow node。理由：它们之间没有独立的产品决策面或权限边界，拆分只增加 seam，不增加可审计性。

**open boundary：** “最小客观结构”的精确字段、单位、minimum completeness 阈值属 L3（见 L3-03、L3-11）。

---

### 2.2 运行 / 记忆 seam：不承担 durable write，只声明应持久化内容与其 authority（对应 L1-02）

**结论：**
Evidence Intake 在 L1-L2 不承担任何 durable write 语义。它只**声明**哪些 evidence foundation / refs / objective context / quality context / `transaction_id` 及其绑定关系应被持久化，以及谁有权威认定其有效（authority = raw/source 存在性 + 本节点的 intake validation + 第 9 步运行时铸号）。铸造 `transaction_id` 是运行时 handoff 动作，不是 durable write；`transaction_id + objective transaction basis + evidence binding` 应在 Transaction Log finalization 或对应最终化路径中持久化，本提案只声明语义，不定义机制。它**不**定义 `EvidenceLogRecord` 的 storage contract、写入执行者或写入顺序。设计稿中的 `EvidenceLogRecord` 只能作为“待持久化内容的逻辑语义”表达，不能写成 implementation-ready storage contract。

**为什么：**
锚定“审计性”与运行/记忆 seam 不变量。Evidence Log 尚属圈外未正式设计对象（L4-07），若在本节点偷冻其存储机制，会让后续 Evidence Log spec 失去裁决权，并把实现细节锁死在错误的层。

**拟改：**
- `evidence_intake_node/02_logic_and_boundaries.md:写入对象/直接执行的 durable write` → `无。`
- `evidence_intake_node/02_logic_and_boundaries.md:写入对象/交给记忆层持久化的内容` → `声明应持久化：raw evidence 保全引用、evidence foundation、objective normalization context、association / quality context、transaction_id 及其与放行交易/evidence refs 的绑定；authority 来源 = raw/source 存在性 + 本节点 intake validation + 第 9 步运行时铸号。不声明写入机制、执行者、顺序（Evidence 内容归 Evidence Log spec；transaction_id 绑定归 Transaction Log finalization / 多 log finalization seam）。`

**open boundary：** Evidence Log 写入发生在 identity 前 / 后 / 两阶段（L4-01）、由谁写入与失败回滚（L4-02）、snapshot/retrieval/hash/blob 机制（L4-03）全部 park 给 Evidence Log spec。

---

### 2.3 统一 intake 入口语义；supplemental 接回既有交易 DEFERRED（对应 L1-03、L2-12）

**结论：**
Evidence Intake 是覆盖 **runtime 新交易、onboarding historical evidence、supplemental / re-intake evidence** 的统一证据入口语义节点，三类来源共享同一输出契约面（以 `source_channel` 区分 provenance）。但“supplemental evidence 接回既有 transaction”在当前架构中标为 **DEFERRED**：它依赖已删除的跨导入 transaction registry / durable 交易索引。当前 EI 只保全 supplemental evidence、产出客观 basis / association candidate / issue，并将其作为待匹配证据或后续 review context 移交；它**不**查历史、不复用既有 `transaction_id`、不覆盖既有 outcome，也**绝不**静默污染 memory。将来若恢复 durable 交易 registry，可重启用“保全 evidence + 标记 supplemental_to_runtime_ref + 输出客观 basis 与 association candidate；是否 amendment/reprocess/review 交下游”的语义。

**为什么：**
锚定“审计性”“accountant control”。统一入口让历史与补充证据获得与新交易一致的可追溯保全；而把“补证后如何处理既有结论”留给下游，避免本节点越权改写已 finalize 的 outcome 或绕过 governance。

**拟改：**
- `evidence_intake_node/01_functional_intent.md:Workflow 位置` → `本节点是 runtime / onboarding / supplemental 三类证据来源的统一入口；三类共享同一输出契约面，以 source_channel 标注 provenance。`
- `evidence_intake_node/02_logic_and_boundaries.md:触发条件` → `当任一来源的新材料进入、需要被组织成可追溯证据基础、且下游尚不能安全消费未处理原文时触发。`
- `evidence_intake_node/02_logic_and_boundaries.md:不得触发` → `本节点是所有交易进入系统运行前的必经入口；任何交易进入系统都必须先经过 Evidence Intake，故不存在"绝对不得触发"的情形。模板"不得触发"项显式声明为：无。`
- `evidence_intake_node/02_logic_and_boundaries.md:证据不足时的行为` → `supplemental evidence 当前只保全 + 输出客观 basis / association candidate / issue；不查历史、不复用既有 transaction_id、不覆盖既有 outcome、不写 memory。supplemental_to_runtime_ref / 接回既有交易 / amendment/reprocess/review 触发依赖未来 durable 交易 registry，标 DEFERRED。`

**排除的替代 + 理由：** 为 onboarding / supplemental 各设独立入口节点。理由：三者的保全、客观投影、质量闸门逻辑同构，分立只会让“可追溯证据”的口径在来源之间漂移，违背统一契约面原则。

**DEFERRED 回路条件：** durable 交易 registry / 跨导入 `transaction_id` 复用机制重新设计并获批后，恢复 supplemental 接回既有交易、迟到证据归属、amendment / reprocessing / review orchestration（L4-05）；在此之前只保全、标记、移交，不裁决。

---

### 2.4 放行资格：transaction-ready 定义 + 整张 BS 全有或全无 + 不达标/未识别去向（对应 L2-01、L2-03、L2-11）

**结论：**
Evidence Intake 往下游交易流放行的，**只能是干净、完整的交易记录**；达不到资格的材料不进交易流，转证据池 / 纠错路。冻结两条边界：

**(1) 放行资格（transaction-ready）——一条记录必须同时满足以下全部，才有资格作为交易进入下游：**
- **日期**：交易发生日期（下游排序、账期判断与追溯基础）。
- **金额（非零）**：没有金额就不是一笔资金事件。
- **方向（进/出可确定）**：方向定不了 = 不知道钱怎么动，下游无法处理（“direction 那栏模糊”即卡在此）。
- **账户标识（source account reference，原文级）**：账单原文可读到的账户标识（账户号/卡号后四位、账户类型、机构/transit 等任一可定位组合）。要求“账单上印了、能读到、能追溯”，**不要求**映射/认领到 Profile 里的具体账户（那归 Profile / Structural Match）。
- **Description 原始文字（非空）**：ER/Alias 需要这段表面文字。要求“**存在**”，**不要求**“能识别对手是谁”——纯转账码 / 内部转账描述也算合格（见 2.8）。
- **可追溯回源材料**：见下条 (3)。

**(2) 放行单位 = 单张 Bank Statement，全有或全无：**
- 若该 BS 每一行都 transaction-ready，且账单内部自洽（有逐行余额时余额链连续、无断口）→ 整张 BS 的交易一起放行。
- 若任一行不达标、或自洽性被打破（解析断口、余额对不上）→ **整张 BS 全部挂起**，不放任何一笔进交易流，转纠错/挂起路；**同一次上传的其它 BS 不受影响，照常处理**。
- 原因：账期完整性与对账（running balance）依赖“所有交易齐全”；偷偷放过部分行会破坏完整性，且单行读坏往往意味着整张账单版面被读偏。余额连续性是白送的、确定性的完整性体检；无逐行余额时退化为“任一行不达标即整张挂起”。

**(3) 可追溯的强制项与形式：** 强制的是“每个客观字段可追溯回源材料”；**行号不是强制项**。代码解析的已知格式可天然带行号；模型兜底解析陌生版面时，用**该行原始文字片段**（原始 descriptor + 金额 + 日期照抄）作锚点，**不得让模型去数行号**以免幻觉/错位。

**(4) 不达标 / 无锚点 / 未识别材料的去向（不进交易流，但绝不丢）：** 两条旁路按性质分清，**证据池 ≠ 纠错路**：
- **证据池（待匹配区）= 有效但等锚点**：能看清、但暂时配不上任何流水的小票/发票等证据 → 进证据池，带 source refs，等对应账单/锚点到达后再配。池中证据**不是交易、不发 `transaction_id`**。
- **纠错路（挂起/返工路）= 破损或读不出、需返工**：未识别/读不出的材料、被整张挂起的 BS（不达标 / 余额链断口 / 自洽性破裂）→ 转纠错/挂起路，等重读、重走解析或报人工。

两条都**一律保全 + 贴问题条**；Evidence Intake 只保全 + 标记移交，不定义纠错动作、不向客户提问、不裁 outcome。具体纠错机制（重读/重走/报人工）、证据池存储/轮询/取回再匹配、出错由 Evidence Intake 还是下一节点上报，park 给纠错接缝 L4-11（目前无专门纠错 Agent）。

本节点**只**裁定“能否产出满足资格的干净交易记录”这一证据层事实，**不**裁定下游 automation routing / pending / review 业务决策——后者归编排 / Coordinator / Review。挂起一张读不干净的账单不是“拒绝一笔交易”（没读成型者尚不是交易），也不是业务路由，而是守住“只吐干净完整交易、绝不吐带病数据、绝不凭空造交易”的本职底线。

**为什么：**
锚定“自动化安全”“审计性”“accountant control”。把“能保存”与“能作为交易往下走”彻底分开：缺核心字段、读不出、只有模型摘要的东西可被保全为证据/问题，但绝不伪装成干净交易污染下游；下游因此只需处理完整交易，不必自己补救残缺数据。

**与 `transaction_id` 分配的衔接：** “必须是资金移动事件才算交易”这条主闸由本资格条 (1) 的金额+方向+账户在 Evidence Intake 拦住；未通过 2.4 放行资格与 2.5 关联闸门的材料不进入第 9 步，也不分配 `transaction_id`。

**拟改：**
- `evidence_intake_node/02_logic_and_boundaries.md:输出类别` → `transaction-ready 交易记录是放行进下游交易流的唯一对象，资格 = 日期 + 非零金额 + 可确定方向 + 原文级账户标识 + 非空 description 文字 + 可追溯；不满足者不进交易流。`
- `evidence_intake_node/02_logic_and_boundaries.md:证据不足时的行为` → `按单张 BS 全有或全无放行：任一行不达标或账单自洽性破裂 → 整张 BS 挂起转纠错路，其它 BS 不受影响；不达标/无锚点/未识别材料保全后转证据池或纠错路，不进交易流、不删不丢。`
- `evidence_intake_node/02_logic_and_boundaries.md:决策权限/Deterministic code 可以决定` → `可决定：某记录是否满足 transaction-ready 资格；某 BS 是否整体通过（含余额连续性体检）；某材料是否 readable / traceable。`
- `evidence_intake_node/02_logic_and_boundaries.md:决策权限/LLM 不能判断` → `不能决定：该交易是否进入 pending/review/自动化、是否分类、是否构成最终 routing outcome。`
- `evidence_intake_node/02_logic_and_boundaries.md:Audit / Trace 边界` → `可追溯强制项是“客观字段可回溯源材料”；行号非强制，模型兜底路用原始文字片段作锚点，不得让模型数行号。`

**排除的替代 + 理由：**
- 让缺核心字段的“带病行”作为独立交易放行、把残缺留给下游补救。理由：下游收到不完整交易根本无法处理，且违背“不污染客观字段 / 不凭空造交易”。
- 单行有问题只挂该行、放过同 BS 其余行。理由：破坏账期完整性与对账，且单行读坏常预示整张账单被读偏；以 BS 为单位整体挂起更安全。
- 强制行号作为可追溯字段。理由：模型兜底解析陌生版面时数行号会幻觉/错位，原始文字片段才是稳健锚点。

**open boundary：** transaction-ready 各字段的 exact schema 与边界值（L3-11）、BS 自洽性/余额连续性体检的 exact 规则（L3-13）、纠错机制与证据池机制（L4-11）park。

#### 2.4.1 余额链（Balance-Chain）机制（详述，供正式文档落地）

> 此机制在老/新系统正式文档中均未单独记录，特此在 L2 写清其功能逻辑与运转方式。

**定位：** 利用银行账单自带的余额数据，做**确定性**的"账期级完整性 + 查重 + 查漏"。它是账单/账期粒度去重与查漏的主力，**取代**老 DP §5 Step 7"按账户/月份数覆盖"的弱启发式；与文件 hash（文件级）、receipt 唯一确认互补，是新系统**取消交易级去重**后仍能保证"不漏单、不重单"的关键。纯确定性代码，不用 LLM，不读 Transaction Log。**位置：EI 流程第 5 步（解析 + 归一之后、解析 receipt 之前）。**

**输入：** 每张 Bank Statement 解析出的——账户标识、账期（起止日期）、期初余额、期末余额、（若有）每行 running balance、各行带符号金额。

**两层检查：**

(1) **单张账单内部自洽（intra-statement）：**
- 期初余额 + Σ(逐行带符号金额) 应 == 期末余额；
- 若有逐行 running balance：每行 `prev_running ± amount == 当前 running`，整条链不得断口；
- 任一处对不上 → 解析疑似读偏 / 漏行 → **整张 BS 挂起转纠错路**（呼应 2.4"单张 BS 全有或全无"）。

(2) **跨相邻账单连续（inter-statement，先按账户分组）：**
- 同一账户的账单按账期排序，逐对检查：`statement[n].期末余额 == statement[n+1].期初余额`；
- 相等且账期相邻 → 覆盖连续；
- **不相等（断口）→ 中间漏了账单**：输出"漏单信号"（指明缺在哪两张之间、推断缺哪个账期）；
- **两张账期相同 / 重叠（同账户）→ 冗余 / 重叠账单**：输出"重复 / 重叠账单信号"。这里能抓到"同一账单换两种格式重发"——文件 hash 抓不到（字节不同），余额链 / 账期重叠能抓到。

**输出信号（语义类别，exact schema 留 L3-13）：**
- 每账户 `statement_chain_status` ∈ {continuous, gap_detected, overlap_or_duplicate_detected, unverifiable}；
- gap → coverage gap signal（缺哪期、在哪两张之间）；
- overlap / duplicate → duplicate/overlap statement signal（账单级查重结果）。

**降级（无余额时）：** 部分信用卡账单 / receipt 不带期初期末 / 逐行余额 → 该账户标 `unverifiable`；退化为"任一行不达标即整张挂起"（intra 侧）+ 把"实际覆盖"交圈外 interaction_agent（目标层）按 账户 × 月份 与期望清单比对（coverage 侧）。

**边界（只检测、不擅自解决）：**
- 它**检测**重叠 / 漏缺，但**不**自行决定丢哪一行 / 哪张账单；重叠只作"此区间预计有重复"的信号，交后续 / 人处理。
- 它**抓不到** engagement 最头 / 最尾的漏单（没有邻居可比）——那归圈外目标层 Agent（账户 × 月份 vs 期望范围）。
- 必须先按账户分组；跨账户不串余额。

**为什么有价值：** 余额是银行已经对平的数，是"白送的、确定性的"完整性体检，比任何数文件 / 猜月份的启发式都强；它与文件 hash、目标层覆盖检查三者互补，共同替掉了被删除的交易级去重（见第 3 节去重三粒度）。

---

### 2.5 解析与关联：文档类型识别、单 BS 作用域、唯一且无歧义才 confirmed（对应 L2-02、L2-08）

**结论：**

**(1) 材料识别 = 文档类型识别，不是相关性判断。** 本节点对每份材料只判“它是哪一类已知文档”（bank statement / receipt / cheque / invoice / contract / note / **无法识别**），这是具体感知任务。识别成已知类型 → 走对应提取路；识别为“无法识别” → 保全并挂起（见 2.4），**不猜它与交易相不相关**。

**(2) 解析：代码优先、模型兜底。** 已知格式 BS 用确定性解析器读成行；陌生格式/解析失败由 LLM 兜底读成行；receipt/cheque/invoice/contract 等非结构化材料由 LLM 读出表面信息。

**(3) 关联作用域 = 单张 Bank Statement 内部。** 加拿大 BS 的支票影像印在该账单底部，必然对应**本张 BS** 的某笔交易；故配对以单张 BS 为单位就地完成，**不**做跨 10 张 BS 的全局配对。

**(4) 关联判据 = 唯一且无歧义，与“代码做还是 LLM 做”无关。**
- **`confirmed_objective_association`**：某份材料在本张 BS / 归属账期作用域内唯一对应一笔交易，且关键字段互相印证、无第二条同样合理候选。典型包括 cheque image 与 bank line 的 `cheque_number` 精确相等；也包括 receipt 由 vendor / 金额 / 日期等证据充分印证，且只对应一笔交易的情形。
- **`candidate_association`**：多条同样合理、**字段打架（金额/日期/vendor 有无法解释的冲突）**、receipt 读不清、或缺少足够印证时，只能作为候选，必要时进入证据池。字段打架不单列为独立状态——它本质上属于“尚不能确认、需更多印证”的 candidate 的一种。
- **`confirmed_objective_association` ≠ transaction identity / dedupe / final outcome**。它只说明“这份材料客观支撑这笔放行交易”，不说明跨导入同一、不复用历史 ID，也不等于会计结论。

**(5) 去重只保留两粒度：文件级 hash + 账单/账期级余额链；交易级去重取消。**
- 文件级：同一文件 hash / content fingerprint / source row 重复 → `duplicate_material_candidate`，在材料层阻断或标记，不是交易级裁决。
- 账单/账期级：同账户账期重叠、余额链断口、冗余账单 → 见 2.4.1。
- 交易级：不做 per-transaction dedupe、不做 same-transaction grouping、不查历史 registry。若文件级 + 余额链都漏过，重复交易会各自进入第 9 步分配新 `transaction_id`，下游会双重计数；当前架构接受这个载重假设，不保留 per-transaction 兜底。**（此载重假设为有意接受的产品决策，已经用户确认；未来恢复 durable 交易 registry 时可重新评估。）**

**为什么：**
锚定“审计性”“自动化安全”。配对锁在单张 BS / 归属账期作用域内既更准也更简单；“唯一且无歧义”把 confirmed 的依据锚定在证据状态，而不是实现手段，避免“LLM 做的只能 candidate / 代码做的天然 confirmed”这种错误分层。交易级去重取消后，文件 hash 与余额链成为唯一去重防线；这让系统更简单，但也明确承担“漏过即双重计数”的风险，而不是假装还有隐藏的 per-transaction 兜底。

**拟改：**
- `evidence_intake_node/02_logic_and_boundaries.md:决策权限/LLM 可以判断` → `LLM 负责：文档类型识别、读非结构化材料表面信息、陌生格式 BS 解析兜底、在单张 BS / 归属账期内综合金额/日期/vendor 语义给出 receipt↔line 关联建议、写 issue 解释。若且仅若唯一且无歧义，可输出 confirmed_objective_association；否则只能 candidate / issue / 证据池。`
- `evidence_intake_node/02_logic_and_boundaries.md:决策权限/Deterministic code 可以决定` → `代码负责：文件 hash/格式识别、结构化 BS 解析、金额日期归一、cheque_number 等确定性共享键的精确匹配（本张 BS 内）、material duplicate 标记、余额链账期级查漏/查重信号。`
- `evidence_intake_node/02_logic_and_boundaries.md:歧义处理 / 决策权限` → `证据↔交易配对是 Evidence Intake 内部处理工序，不是独立对外输出。配对判定 association_status ∈ {confirmed_objective_association, candidate_association, unassociated, duplicate_material_candidate}（不设独立 conflicting 状态：字段打架归 candidate_association）。confirmed 判据 = 唯一且无歧义；confirmed 的关联随 EvidenceFoundationHandoff 并入放行交易包、流向运行层；candidate / unassociated 转证据池；均不等于 transaction identity / 交易级 dedupe。duplicate_material_candidate 只表达文件/材料层疑似重复；账期级重复/重叠见 balance-chain signal；本节点不做交易级去重、不读 Transaction Log。`

**排除的替代 + 理由：**
- 沿用老系统把 high-confidence receipt match 直接写入 transaction 字段。理由：等于把 fuzzy 分数变成不可审计的事实写入；新口径只允许“唯一且无歧义”的客观关联，并保留 refs 与依据。
- receipt↔line 由代码先打分硬配、再交 LLM 复核。理由：代码按金额/日期硬筛会错配（金额日期同但商家不同）也会漏配（小费/税差），且错配会锚定干扰后续 LLM 判断；不如让 LLM 在结构化数据上整体配。
- 跨全部 BS 做全局支票/小票配对。理由：难度大且易跨账单错配；加拿大 BS 支票影像就在本账单内，单 BS 作用域即可。

**open boundary：** association basis refs、`candidate_strength` 表达、`content_fingerprint_ref` 的 contract-level 字段形态（L3-06、L3-10）park；receipt 防虚构、金额容差、归月口径见 `缺口地图.md` 的 Evidence Intake section。

#### 2.5.1 transaction_id 分配（第 9 步）

**结论：**
Evidence Intake 在第 9 步为每笔通过放行闸门的交易铸造永久、唯一、ULID 式 `transaction_id`。铸号只基于本次运行中已放行的交易对象，不查历史、不读取 Transaction Log、不访问 durable registry、不跨导入复用；同一批里两条长得一样但未被文件 hash / 余额链挡住的交易也分别发新号。

**consumer：**
- **ER**：以带 `transaction_id` 的交易作为 identity 判断对象。
- **JE Generation**：以 `transaction_id` 作为分录关联键，避免分录悬空。
- **Transaction Log finalization**：持久化最终交易及其 `transaction_id + evidence refs + outcome` 绑定。
- **Case / Review / Correction lineage**：用 `transaction_id` 回指具体资金事件；跨运行更正定位与迟到证据接回当前 deferred。

**边界：**
- 不建立 durable transaction registry；
- 不跨导入查重或复用既有 `transaction_id`；
- 不把 confirmed evidence association 当作 transaction identity；
- 不保证账务幂等。跨运行账务幂等依赖 durable registry / finalization 机制，当前标 DEFERRED。

**DEFERRED 回路条件：** 若未来重新设计 durable 交易 registry，可恢复跨导入 `transaction_id` 复用、迟到证据接回既有交易、跨运行更正定位与跨运行账务幂等；当前仅声明这些能力应由 registry / finalization seam 承担，不在 Evidence Intake L2 冻结机制。

#### 2.5.2 当前运行流程（九步）

1. 文件 hash / content fingerprint 查重，识别重复原始材料。
2. 标记异常、无法识别、多余或重复文件；不删除，转证据池 / 纠错路或作为 issue 移交。
3. 解析所有文件，先抽取 bank statement 交易行与其它 evidence 表面字段。
4. 归一化客观交易字段（日期、金额、方向、source account、currency、raw description、source refs）。客观钥匙字段必须是归一化后的类型（金额→数值绝对值、direction→枚举、date→日期类型），不是裸字符串；原文（descriptor/raw refs）照原样留底。
5. 余额连续性检查（单张账单内部自洽 + 跨相邻账单连续 / 重叠检测，见 2.4.1）。
6. 解析 receipt / cheque / invoice / contract / note 等非 BS evidence 的关键表面字段。
7. 在单张 BS / 归属账期内做 receipt / cheque / invoice 等 evidence ↔ 交易配对。
8. 按“唯一且无歧义”判定 confirmed / candidate / unassociated（字段打架归 candidate，不单列 conflicting）；未匹配或歧义证据进入证据池或 issue。
9. 仅对通过第 3-8 步放行闸门的交易分配永久 `transaction_id`，并输出给 ER / JE / Transaction Log / Case 血缘 consumer。

放行闸门覆盖第 3-8 步；未通过者不进入第 9 步、不发号。

> 上述九步是 L2 的**排序约束示意**——它固定的是有产品含义的先后边界（铸号必在放行闸门之后、余额链体检在 receipt 配对之前、放行闸门覆盖第 3-8 步）。**精确的执行序列 / routing sequence 属 Stage 4 执行算法，不在 L2 冻结**，已 park 至 `缺口地图.md` 的 Evidence Intake section。

---

### 2.6 Counterparty surface signals 准入：全量保全、无 authority ranking；external import context ≠ external lookup（对应 L2-06、L2-07）

**结论：**
本节点把所有身份表面证据作为**离散、可追溯的 surface signals** 保全并暴露：raw bank text / descriptor、receipt vendor、cheque payee、invoice party、contract party、accountant context refs。这些 signals 之间**不在 Evidence Intake 做 authority ranking**——它们一律是“raw identity signal，不是 stable entity / approved alias / role”。哪个字段优先、cheque payee 是否作为 bank descriptor 缺身份意义时的 fallback，是 **ER / Alias Log 的判断**，不是本节点的。

`external_import_context` 只记录**外部来源 / 导入 provenance**（材料来自哪个 feed / 导入通道），属证据来源记录；本节点**不执行也不裁定 external lookup**（“这是谁”的联网搜索）——后者锁在 ER 侧。

**为什么：**
锚定“记忆复用”“审计性”，并保护 Entity 圈已锁不变量。ER 的 stable/unknown 判断、Alias Log 的“已确认 surface text → stable entity”、Entity Log 的 provenance 都依赖这些 surface signals 可追溯地存在；但“两态 entity / 不复活 candidate”是已锁不变量，Evidence Intake 若对 signals 排序或择一，等于偷偷做身份判断，污染长期身份权威。Alias Log 已冻结的两类来源（bank description；其缺身份意义时的 cheque payee fallback）正是 ER/Alias 的裁量，本节点只负责把两者都保全。

**拟改：**
- `evidence_intake_node/02_logic_and_boundaries.md:输出类别` → `counterparty surface signals（raw bank text/descriptor、receipt vendor、cheque payee、invoice/contract party、accountant context refs）作为离散可追溯 signal 输出给 ER；本节点不对其排序、不择一、不升级为 stable entity / approved alias / role。`
- `evidence_intake_node/02_logic_and_boundaries.md:读取对象` → `external import context 只读为来源 provenance；本节点不执行 external lookup / 联网搜索，不裁定“这是谁”。`
- `evidence_intake_node/02_logic_and_boundaries.md:只能提出 candidate` → `client/accountant note 中出现的 alias/entity/role 线索只能作为 candidate signal 交 ER / governance 路径，不能成为 approved alias 或 stable entity。`

**open boundary：** surface signal 与 external evidence reference 的 exact 字段形态（L3-02、L3-05）park 给 L3 / Evidence Log。

---

### 2.7 Client / accountant note 永远是 raw evidence，绝非 approval（对应 L2-04）

**结论：**
在 Evidence Intake 中，client note 与 accountant note 一律是 raw evidence。`source_actor_type = accountant` 只表示“这句话出自会计师”，**绝不**等于 accountant approval。note 的存在只是“某人说过某事”的证据，永远不是已批准的客户政策、会计结论、approved alias 或 rule。note 携带的判断只能作为 candidate / context 信号；任何 durable authority 必须走下游 accountant / governance 路径。

**为什么：**
锚定“accountant control”“审计性”。把“会计师原话存在”误读为“会计师已批准”，会让未经审批的内容直接获得 policy / 分类 / automation authority，击穿控制权与 governance 闭环。

**拟改：**
- `evidence_intake_node/02_logic_and_boundaries.md:决策权限/Accountant 必须决定` → `note 是否构成已批准客户政策 / 会计结论 / approved alias / rule，必须由 accountant 在下游路径决定；Evidence Intake 不得据 source_actor_type=accountant 推断 approval。`
- `evidence_intake_node/02_logic_and_boundaries.md:Audit / Trace 边界` → `accountant context refs 只作可追溯证据线索，不能成为 accountant approval / governance approval。`

---

### 2.8 Issue signal 与 accountant question 的边界（对应 L2-05）

**结论：**
本节点可输出 `source clarity` / `issue summary` / `EvidenceQualityIssueSignal`（含 human-readable 解释），描述缺什么、冲突在哪、来源是否可读/可定位，并保留 source refs。它**不得**生成 accountant-facing question（如“您是否确认 X？”），**不得**决定 pending / review outcome。边界：本节点只描述 evidence condition（事实层），Coordinator / Pending / Review 才把它转成人工交互（决策层）。

**“对手是谁不清楚”不属于本节点的 issue。** counterparty / entity 识别归 ER + Case Judgment + Coordinator；内部转账、PAD、e-transfer 等本就常常看不出对方 entity，这是常态而非证据问题。只要 description 文字存在（见 2.4 资格条），该交易即合格放行，**不**因“对手不清”产出 issue 或扣留。本节点的 issue 只覆盖证据层缺陷：材料不可读、来源不可定位、字段缺失/冲突、material duplicate、未识别材料。（receipt↔line 配对的字段打架不计入 issue，归 candidate_association，见 2.5。）

**为什么：**
锚定“accountant control”。老系统由同一 Agent 端到端把摘要发给会计师并处理待确认项；新系统必须把“暴露问题”与“向人提问/裁决”分权，否则 Evidence Intake 会实质代系统向会计师发问，越过 Coordinator / Review 的契约面。

**拟改：**
- `evidence_intake_node/02_logic_and_boundaries.md:输出类别` → `EvidenceQualityIssueSignal + human-readable issue summary 描述证据缺口/冲突/来源不清，保留 source refs，consumer 为 Coordinator/Pending 与 Review；本节点不生成 accountant-facing question、不决定 pending/review outcome。`
- `evidence_intake_node/02_logic_and_boundaries.md:决策权限/LLM 不能判断` → `不能：把 evidence issue 写成向会计师的提问、替系统决定该问什么、决定是否进入 pending/review。`

**open boundary：** issue 如何实际变成 Pending/Review 问题、谁聚合、谁关闭（L4-06）park。

---

### 2.9 契约面最小化；evidence presence 由本节点出，evidence_condition 的充分性语义归下游（对应 L2-09、L2-10）

**结论：**
本节点声明的输出类别（`EvidenceFoundationHandoff`〔含本次放行交易已 confirmed 的 evidence↔交易关联结果〕、`ObjectiveNormalizedTransactionBasis`、`transaction_id`、`EvidenceQualityIssueSignal`、counterparty surface signals、raw/source refs）即为对 runtime 下游的**唯一契约面**。证据↔交易配对属本节点内部工序，其 confirmed 结果并入 `EvidenceFoundationHandoff` 随放行交易包流向运行层（顺序：Profile / Structural Match → Entity Resolution → Rule Match），**不**另列为独立对外输出；candidate / unassociated 证据转证据池，不进入该交易包。下游可按 `evidence_refs` 追溯原始材料用于 trace/audit，但不得把本节点未声明的 Evidence Log 内部 metadata 当 runtime 契约依赖。

关于 `evidence_condition`：本节点只产出**客观的“存在哪些证据”事实**（有无 receipt / invoice / cheque / 仅 bank line、各自 refs）；它**不**写 Case / Rule 语义上的“证据是否充分”。`evidence_condition` 作为 sufficiency 判断是 Case Log / Rule Match 对 evidence refs 的**后续解释**，不是 Evidence Intake 的结论。

**为什么：**
锚定“审计性”与契约面最小化原则。下游耦合本节点内部存储会让 Evidence Log 无法独立演进；而让 Evidence Intake 断言“证据充分”会越权写入 case/rule 语义，违背“Evidence 不保存业务结论”。把“证据存在性”（客观）与“证据充分性”（业务）分层，正好对齐 Case Log / Rule Match 已锁的 evidence 引用方式。

**拟改：**
- `evidence_intake_node/02_logic_and_boundaries.md:输出类别` → `本表是对下游唯一契约面；下游只能依赖此处声明的输出类别，可经 evidence_refs 追溯原文，但不得依赖未声明的 Evidence Log 内部 metadata。`
- `evidence_intake_node/02_logic_and_boundaries.md:输出类别` → `本节点输出“证据存在性”客观事实（有无 receipt/invoice/cheque/仅 bank line + refs）；不输出 case/rule 意义上的 evidence sufficiency；evidence_condition 的充分性解释归 Case Log / Rule Match。`

**open boundary：** `evidence_condition` 的 exact schema 与 sufficiency threshold（L3-08）park 给 Case Log / Rule 侧。

---

### 2.10 Legacy 迁移：继承老系统入口纪律，剥离被迫混入的下游权力

**结论：**
明确从老系统 `Data Pre-processing` **保留**与**不保留**的行为，写入 02 的 Legacy Constraint Translation，使审计可回溯。

**保留（仍服务当前产品目标）：**

| Retained Constraint | 来源 | 为什么现在仍成立 |
| --- | --- | --- |
| 代码优先识别/解析，LLM 兜底 | DP §1、§5 Step 2 | 成为本节点 deterministic vs LLM 决策权限边界：hash/格式识别/结构化 parser/金额日期归一/cheque_number 精确匹配/余额链检查交给代码；文档类型识别/非结构化读取/陌生格式解析兜底/receipt↔line 语义配对/issue 解释交给 LLM |
| 不丢弃任何文件，无法处理必须标记 | DP §4、§8 | 成为“全量保全 + 不达标/未识别材料转证据池或纠错路、不删不丢”的不变量 |
| 不确定不猜，标 unknown / 待确认 | DP §4、§5 Step 2 | 成为放行资格闸门与“唯一且无歧义才 confirmed”的语义根：缺核心字段者整张 BS 挂起，不伪装成干净交易 |
| cheque_number 优先、金额日期兜底的支票关联 | DP §5 Step 3 | 迁移为：cheque_number 精确匹配 = confirmed_objective_association（代码、单 BS 内）；receipt↔line 只有唯一且无歧义时 confirmed，否则 candidate / issue / 证据池 |
| 完整性校验（账户/月份覆盖、重复文件、未配 receipt/cheque、无法识别附件） | DP §5 Step 7 | 账期完整性迁移为单张 BS 全有或全无放行 + 余额连续性体检；重复文件 → duplicate_material_candidate；未识别附件 → 证据池/纠错路；均为自动化安全信号，非会计判断 |
| amount 归一为绝对值、direction 独立表达、direction 不可靠者不进标准集 | DP §7 | 迁移为 ObjectiveNormalizedTransactionBasis 的 amount_absolute + direction 客观投影规则 |
| `transaction_id` 分配（DP §5 Step 5 / `transaction_identity_and_dedup` 的铸号部分） | DP §5 Step 5 | 保留为 Evidence Intake 第 9 步：只为本次运行放行交易铸造永久唯一 id；不跨导入查重、不复用历史 id、不建立 registry |

**不保留（旧架构被迫混入、新系统已分权）：**

| Old Behavior | 不保留原因 |
| --- | --- |
| 交易级 dedupe / same-transaction grouping（DP §5 Step 5 / `transaction_identity_and_dedup` 的去重部分） | 交易级去重取消；去重只保留文件 hash + 账单/账期余额链。跨导入复用、迟到证据接回、跨运行幂等依赖 durable registry，当前标 DEFERRED |
| canonical description 生成 + Pattern Dictionary 读写（DP §5 Step 6 / `standardize_description`） | 身份/别名语义已迁入 Entity 圈（ER / Alias Log / Entity Log）；本节点只保全 raw surface signals，不建长期身份 authority |
| bank_account → `profile.bank_accounts[].id` 映射写入交易（DP §5 Step 5） | 映射涉及 Profile / structural context，归 Profile / Structural Match；本节点只输出 source_account_ref（非 COA、非已确认 profile id） |
| 把 receipt/cheque 写入 transaction 的 `receipt` / `cheque_info` 字段（DP §5 Step 3-4） | 改为内部 evidence↔交易关联工序（confirmed 结果并入 EvidenceFoundationHandoff），不在 Evidence 层确立交易归属 |
| 生成摘要并直接与 accountant 交互、处理待确认项（DP §5 Step 8、§9） | 人工交互归 Coordinator / Pending / Review；本节点只输出 issue signal |
| 端到端单 Agent 写出最终数据集 | 违反运行/记忆 seam；本节点只声明应持久化内容，不定义写入机制 |

**拟改：**
- `evidence_intake_node/02_logic_and_boundaries.md:Legacy Constraint Translation` → 填入上两表。

---

## 3. 分类备案

> **权威口径**：未冻结项（L3 / L4 / seam / 外阻 / DEFERRED）的**唯一权威池是 `缺口地图.md` 的 Evidence Intake section**；与本文档相关联额外提出的新概念（余额链、证据池、纠错路、内部执行序列等）也以缺口地图为承接处。本节仅作本提案"已收口 vs park"的映射留痕，便于追溯，不作为第二权威源；若与缺口地图不一致，以缺口地图为准。

- `EvidenceFoundationHandoff` exact fields / required-optional → **L3**（L3-01，留联合 L3）；放行语义已由 2.4 transaction-ready 资格收口
- `evidence_refs` / `raw_evidence_pointer` / `source_location` 字段形态 → **L3**（L3-02）；行号非强制、模型路用原始文字片段作锚点已由 2.4(3) 收口
- `ObjectiveNormalizedTransactionBasis` 字段名 / 单位 / source ref 形态 → **L3**（L3-03）
- transaction-ready 各字段 exact schema 与边界值（六项资格的精确定义）→ **L3**（L3-11）；六项资格本身已由 2.4(1) 收口
- 单张 BS 自洽性 / 余额连续性体检的 exact 规则 → **L3**（L3-13）
- direction enum 统一（`money_in/out` vs `inflow/outflow`）→ **L3**，需与 Rule Match / JE 等下游对齐（L3-04）
- `source_channel` / `source_type` / `evidence_type` / `source_actor_type` enum 跨 runtime/onboarding 对齐 → **L3**（L3-05）
- `EvidenceAssociationResult` 的 status / candidate_strength / basis refs schema → **L3**（L3-06）
- `EvidenceQualityIssueSignal` 的 issue types / severity / affected refs / summary 形态 → **L3**（L3-07）
- `evidence_condition` exact schema 与 sufficiency threshold → **L3**，归 Case Log / Rule 侧（L3-08）
- `preservation_status` 属 Evidence Intake 输出还是 Evidence Log 状态 → **L3**（L3-09）
- `content_fingerprint_ref` / duplicate material reference 字段形态 → **L3**（L3-10）
- invalid input/output validation matrix → **L3**（L3-12）
- Evidence Log 写入时点（第 9 步铸号前/后/两阶段）→ **L4 / seam-park**（L4-01）
- Evidence Log 写入执行者 / finalization / 回滚 → **L4 / seam-park**（L4-02）
- evidence refs snapshot / retrieval / cache / hash / blob storage → **L4 / seam-park**（L4-03）
- 多 log finalization 顺序（Evidence / Transaction / Case / Rule / JE）→ **L4 / seam-park**（L4-04）
- supplemental evidence amendment / reprocessing / review trigger orchestration → **L4 / seam-park**（L4-05），依赖 durable 交易 registry，当前 DEFERRED
- Evidence issue → Pending/Review 的聚合/关闭机制 → **L4 / seam-park**（L4-06）
- 纠错机制（重读/重走流程/报人工）与证据池存储 / 轮询、出错由 Evidence Intake 还是下一节点上报 → **L4 / seam-park**（L4-11）；目前无专门纠错 Agent
- parsing/OCR/LLM extraction algorithm、prompt/tool interface、test matrix → **L4**（Stage 4-7，L4-10）
- Evidence Log 无独立正式 spec → **L2·外阻**：本节点只能声明语义输出，不能冻结 Evidence Log 内部（L4-07）
- Coordinator / Pending、Profile、Governance 圈外未审 → **L2·外阻**：consumer 只能挂起引用（L4-08）
- durable 交易 registry / 跨导入 `transaction_id` 复用 / 迟到证据接回既有交易 / 跨运行更正定位 / 跨运行账务幂等 → **DEFERRED / seam-park**：只有未来恢复 registry 时重启用；当前 EI 第 9 步只铸新号，不查历史、不复用。
- 交易级去重 → **取消**：不归任何现行节点；当前唯一去重防线为文件 hash + 账单/账期余额链。
- 覆盖完整性检查（漏月/漏账户/多发重复）→ **圈外 engagement-scope 接缝**：中段漏月由 EI 的余额链（2.4.1）提供确定性信号；EI 另输出"实际唯一覆盖信号"给交互层；engagement 最头/最尾漏月、缺账户、多发重复的期望比对与查缺提问归 interaction_agent（已吸收原 coverage_completeness_check 职责），正式 spec 未冻结。详见 `缺口地图.md` 的 Evidence Intake section 与 `独立question文档/interaction_agent_question.md`。

---

## 4. 自检与判定（Stage 1-2 清单 22 项）

1. 为什么存在？——把混乱原始材料变成可追溯、可被下游确定性消费的客观交易事实的统一入口 + 证据质量闸门 + transaction_id 铸号点。✓
2. 为什么不能删/合/内联？——下游多节点依赖 objective basis + traceable refs + `transaction_id`；若推给下游会导致多节点重复解析并丢失统一放行闸门。✓
3. 唯一核心职责？——见 2.1。✓
4. 上游依赖？——runtime/onboarding/supplemental 原始材料 + batch intake context（只读）。✓
5. 下游影响？——见第 1 节 consumer 列表。✓
6. 读哪些 logs / stores？——当前 batch intake context、已存在 Evidence 引用；**不读 Transaction Log / Entity / Case / Rule / Governance**。✓
7. 直接写什么？——无 durable write（2.2）。✓
8. 只能提出什么 candidate？——candidate_association、duplicate_material_candidate、alias/entity/role candidate signal。✓
9. 绝不能写什么？——交易级 dedupe、跨导入 transaction_id 复用、durable registry、stable entity / approved alias、COA/HST/JE、rule/case 结论、Transaction Log、任何 durable business memory。✓
10. deterministic code 决定什么？——文件 hash/格式识别/结构化 parser/金额日期归一、transaction-ready 资格、单 BS 整体放行（含余额连续性体检）、余额链账期级查漏/查重、cheque_number 等确定性键关联（单 BS 内）、material duplicate 标记、运行时铸造 `transaction_id`。✓
11. LLM 判断什么？——文档类型识别、非结构化材料读取、陌生格式 BS 解析兜底、单 BS / 归属账期内 receipt↔line 语义配对、issue 解释；只有唯一且无歧义时可 confirmed，否则 candidate / issue / 证据池。✓
12. accountant 决定什么？——note 是否构成 approval / policy / 分类（2.7）。✓
13. governance 批准什么？——任何长期身份 / alias / rule / automation policy 变化（本节点只挂候选）。✓
14. 证据不足怎么处理？——缺核心字段 → 整张 BS 挂起转纠错路；无锚点证据 → 证据池待匹配；一律保全不丢、不猜、不伪装成干净交易（2.4）。✓
15. 歧义怎么处理？——输出 candidate + issue 或转证据池，不择 winner（2.5）。✓
16. source conflict 怎么处理？——证据↔交易配对的字段打架归 `candidate_association`（保全双方原文 + refs，不单列 conflicting 状态、不裁决）；证据本身的缺口/矛盾走 `EvidenceQualityIssueSignal`（2.5、2.7、2.8）。✓
17. 哪些 trace 留下？——raw evidence refs、objective extraction 状态、association / quality context。✓
18. 哪些 trace 不能成 authority？——evidence refs / issue / AI summary 不能成 entity/rule/case/accountant/governance authority。✓
19. 是否声明对外契约面 + 每输出 consumer？——是（第 1 节 + 2.9）。✓
20. 是否守住运行/记忆 seam？——是，只声明应持久化内容 + authority，不定写入机制（2.2）。✓
21. 哪些未冻结？——见第 3 节全部 L3 / L4。
22. 是否可进 Stage 3？——**否**。字段级 schema、enum 统一、direction 口径、evidence_condition schema、Evidence Log 写入时点均未冻结；且 Evidence Log / Coordinator / Profile 为圈外外阻。

**本对象 L2 可否判定完成：** 是（就 L1-L2 边界而言）。节点必要性、唯一职责、durable write seam、统一 intake 语义、放行资格闸门（transaction-ready + 单 BS 全有或全无）、文档类型识别、单 BS 作用域、唯一且无歧义关联判据、文件 hash + 余额链去重边界、transaction_id 第 9 步铸号、surface signal 准入、note authority、issue 边界（含“对手不清”不属本节点）、契约面最小化、Legacy 迁移均已收口；剩余全部落在 L3 字段层或 L4 / 外阻 seam（含纠错机制、证据池、durable 交易 registry / 跨导入复用），已备案，不阻塞创建 01/02 正式文档。
