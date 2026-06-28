> ⚠️ **已过时 / superseded（owner 2026-06-27）** — 本 L2 提案的 L1 / L2 结论已由 `BK_Copilot/memory_layers/transaction_log/` 正式草案取代，仅作历史来源保留。当前权威以 `BK_Copilot/` 正式草案 +（产出后）对应 L3 schema 为准；**勿据本文件回灌已删除的概念 / 字段**。

# Transaction Log L2 提案

> 对象类型：**memory layer**（本轮已与用户确认：Transaction Log 是围绕「每一条交易」构建的记忆层，与 Entity Log / Case Log / Alias Log 同族，不是 workflow node）。
> 拟改指向待建 `BK_Copilot/memory_layers/transaction_log/00|01|02`（命名可在后续确认）。
> 权威来源：每条契约结构均指回 ① A 类原文（已落文邻居正式 00/01/02，见 `_audit_work/transaction_log/01_契约面与缺口.md`）或 ② 用户本轮拍板的决定。new / old system（B 类）一字不作依据。

---

## 依赖

**已锁上游 / 旁锁邻居（A 类，约束 Transaction Log 的接缝）：**

- **Evidence Intake 已锁定**：`transaction_id` 的统一铸号点在 Evidence Intake（Transaction Identity 节点已删除，铸号并入其流程第 9 步）；`transaction_id + objective transaction basis + evidence binding` 应在 Transaction Log finalization 持久化；Evidence Intake **不读** Transaction Log 作 runtime source。（`evidence_intake_node/01:13,75`、`02:54,125`；已锁事实 `当前任务状态.md`）
- **Case Judgment 已锁定**：CJ **绝不**直接写 Transaction Log；High Confidence Classification 的「最终交易结果、结构化 reasoning、AI 高置信度自动完成标记、相关 evidence / external source refs」由后续 finalization / Transaction Log 机制保存；reasoning 进 Transaction Log「可供 review / correction / governance / audit，但不成为 authority」。（`case_judgment_node/01:63,81,106`、`02:92,182`）
- **Coordinator 已锁定**：Coordinator **绝不裸写、也不读取** Transaction Log；拿不到 entity 但完成分类的交易（情况 X / Y）**只 finalize 到 Transaction Log**，不进 Case Log / Entity Log；最终交易 outcome、processing path、身份未知但分类完成的 finalization 交 Transaction Log；「Transaction Log 是最终交易审计 source of truth」。（`coordinator_node/01:46,67,95`、`02:62,169,206,226`）
- **Case Log 已锁定**：Case Log 通过 `transaction_log_ref` / finalization proof **引用** Transaction Log，不存修改历史、不存完整 processing path、不替代审计记录；冲突时 **Transaction Log 对 final outcome 与 audit trail 优先**；「按 entity 查询全部历史交易属于 Transaction Log 的 entity 索引查询能力」。（`case_log/01:20,89,119,122,123,126,127`、`02:110,119-121`、`00:19,48`）
- **Entity Log / Alias Log / Entity Resolution 已锁定**：三者均声明 Transaction Log 是 audit-facing final record、**不参与 runtime identity / learning decision**；逐笔交易历史与完整 audit trail、COA / HST / JE / final outcome / processing path 归 Transaction Log；ER **绝不写** Transaction Log，身份识别不得从 Transaction Log 反向学习。（`entity_log/01:19,66,69,112`、`alias_log/01:19`、`02:125`、`entity_resolution_node/01:57`、`02:120,282`）

**下游 consumer：**

- **交易处理层全部节点：不读 Transaction Log**（已锁运行时隔离）。
- **Governance / 治理环节：可读（只读，非 authority）**（用户 1c）。
- **未来 agent 工具：可读作 context / 线索**（用户 1c，列为方向，不冻结为当前 contract）。

**未落文邻居（挂 open boundary，绝不拿 B 填）：** JE Generation、Intervention Log、Review Node、Case Memory Update Node、Governance / Governance Review、Profile / Structural Match、Knowledge Summary / Knowledge Compilation、Evidence Log、interaction_agent、Post-Batch Lint、对外 final output report / export artifact。

---

## L2 决策

### 决策点 1：Transaction Log 是 transaction-indexed 的逐笔交易全过程审计层；真值单位 = 每一笔交易

- 结论：
  Transaction Log 是以 `transaction_id` 为主索引的「逐笔交易审计 source of truth」。它的真值单位是**每一笔进入系统的交易**：记录一笔交易自进入系统起、被各环节如何处理、最终变成了什么的**全过程**。

  「按 entity 查询某主体名下全部历史交易」是**对这些逐笔记录的派生访问能力**（A 已锁此能力归 Transaction Log），不是另设一个 entity 索引存储层——逐笔记录是真值，entity 视图是其上的查询。

- 为什么不能由 runtime handoff 或现有 store 覆盖（M1-M2 Q2）：
  - runtime handoff 是瞬时的，处理完即逝，无法事后逐笔追溯。
  - Entity Log / Case Log / Alias Log 均**明确拒绝**承载逐笔交易历史与完整 audit trail（A 原文：「逐笔交易历史与完整 audit trail 归 Transaction Log」`entity_log/01:69`；「Case Log 不是 Transaction Log 的 entity 副本」`case_log/01:119`）。
  - Evidence Log 只证明证据存在性，不记录交易处理结果。
  故必须有一个**独立的、按交易索引的持久审计层**回答「这一笔交易到底经历了什么」。

- 为什么（锚定核心产品目标）：锚定「审计性」——审计性与会计师控制权是核心能力。逐笔可追溯是审计性的物理载体。

- 拟改：
  `BK_Copilot/memory_layers/transaction_log/00_index.md:一句话定义` →
  `Transaction Log 是以 transaction_id 为主索引的逐笔交易审计 source of truth。它记录每一笔进入系统的交易自进入起被各环节如何处理、最终变成什么的全过程；按 entity 查询全部历史交易是对这些逐笔记录的派生访问能力，不另设 entity 索引存储层。`

- 排除的替代 + 理由：
  排除「Transaction Log = 各 entity / 各 case 的交易镜像副本」。理由：会与 Entity / Case Log 形成双 source of truth；逐笔交易真值只应有一处。

### 决策点 2：覆盖面 = 每一笔被赋予 transaction_id 的交易（含 terminal / blocked / 无 JE）；无损完整

- 结论：
  **只要一笔交易被赋予了 `transaction_id`，就必须进入 Transaction Log**（用户拍板）。覆盖面包含：正常完成并出 JE 的交易、被拦下 / 无法生成 JE / not-finalizable 的 terminal 交易、情况 X / Y（身份未知但分类完成）的交易。

  与 Case Log（学习层、允许有损：可丢弃 / 替代过期案例）根本不同，Transaction Log 是**无损、完整、append-only 的审计层**：每一笔交易都在、全过程都留、原始记录不被抹除（append-only 见决策点 8）。

  terminal / blocked 交易在 Transaction Log 中以其**最终状态**记录（如：被拦原因、走到的最后阶段、无 JE 的 terminal outcome）；具体状态枚举 / 字段形态 → L3。

- 为什么（锚定核心产品目标）：锚定「审计性」。系统对任何一笔进入的交易都做过行为，这些行为必须无遗漏可查；「有头无尾」的交易恰恰是审计与排错最需要看到的。

- 拟改：
  `BK_Copilot/memory_layers/transaction_log/01_memory_intent.md:覆盖面` →
  `凡被赋予 transaction_id 的交易一律进入 Transaction Log，含正常 finalize、terminal / blocked / 无 JE、以及情况 X / Y（身份未知但分类完成）的交易。Transaction Log 无损完整、append-only，不丢弃、不就地覆盖任何交易记录。terminal / blocked 以最终状态记录，状态枚举留 L3。`

- 排除的替代 + 理由：
  排除「只记录 finalize / 出了 JE 的交易」。理由：会让被拦截 / 异常终止的交易在审计中消失，正是排错最需要的盲区。

### 决策点 3：存什么——全过程 processing path + final outcome（语义清单，字段留 L3）

- 结论：
  Transaction Log 保存的不只是 final outcome，而是**一笔交易的全过程 processing path + 最终结果**。语义清单（exact 字段名 / enum / schema / validation 全部留 L3）：

  | # | 语义内容 | 作用 / 来源 |
  |---|---|---|
  | 1 | `transaction_id` | 主索引——逐笔锚点（来自 Evidence Intake 铸号）。 |
  | 2 | objective transaction basis | 交易客观基础（金额 / 方向 / 日期 / 账户 / description 等的客观快照）；`transaction_id + basis + evidence binding` 应在此持久化。 |
  | 3 | evidence binding / `evidence_refs` | 指向 Evidence Log 的证据引用（指针，不存原文）。 |
  | 4 | 身份字段（含 `unknown` 标识） | 这笔交易归属哪个 entity；**认不出时写「unknown / 未知」标识**（用户 L2-D），不阻塞入库。 |
  | 5 | 情况 Y 身份待补线索 | 身份未知但分类完成的交易，挂在 Transaction Log 上的**不阻塞复查线索**（A 原为「初步倾向 Transaction Log，未冻结」，本轮用户拍板**锁定**落点为 Transaction Log，Case Log 对应文字已同步改为「已定」；字段形态 L3）。 |
  | 6 | full processing path | 这笔交易经过了哪些环节、各环节怎么处理的全过程痕迹（ER 结果 / rule-hit / 分类 / 交互 / JE / 拦截等）。 |
  | 7 | final outcome | 最终交易结果（最终 COA 科目、HST/GST 处理、JE 结果或 terminal 状态）。 |
  | 8 | 最终 COA / HST-GST 值 | 最终会计分类结果（**最终权威在此**，A 已锁）。 |
  | 9 | rule-hit ref | 这笔交易最终由哪条 rule 处理的 source 语义 + rule ref（A：适合存 Transaction Log）。 |
  | 10 | reasoning / audit trace | CJ 等环节的结构化 reasoning 留痕（**仅 trace，非 authority**，见决策点 5）。 |
  | 11 | `confirmed_by` 留痕 | 结论由谁确认的**具体人员 + 完整审计留痕**（A：归 Transaction Log；Case Log 只存类型）。 |
  | 12 | AI 高置信度自动完成标记 | High Confidence 自动完成的标注（字段形态 L3）。 |
  | 13 | 更正记录（append） | 后续被 review / 治理更改后追加的记录：谁改的 + 改后的最终分类结果（见决策点 8）。 |

- 为什么（锚定核心产品目标）：锚定「审计性」与「会计师控制权」。全过程可追溯 + 最终结果权威，是事后刨根问底（某分类 / 某 entity 错在哪）的唯一依据。

- 拟改：
  `BK_Copilot/memory_layers/transaction_log/01_memory_intent.md:保存什么` → 以上 13 项语义清单（标注未冻结、schema 留 L3）。

### 决策点 4：绝不存什么 + runtime 隔离（保护未来「agent 读 TL」设想不越界）

- 结论：
  - **绝不存**：可复用的学习先例（那归 Case Log）；runtime identity / learning authority；不把自己变成「按 entity 索引的学习层 / 流水副本」——Transaction Log 只做 audit-facing 的逐笔记录。
  - **runtime 隔离（硬边界）**：Transaction Log **永不作为 runtime decision / identity / learning source**。交易处理层任何节点都不读它。
  - **未来 agent / governance 读取的护栏**：即便未来 agent 工具或治理环节读 Transaction Log，也只能把它当**参考 context / 线索**——读完仍须自己重新判断，**绝不把其中内容（尤其 reasoning）当成可复用 authority 直接继承**。

- 为什么（锚定核心产品目标）：锚定「审计性」与「自动化只有在已批准记忆 / 适当 authority 支撑下才有价值」。身份识别 / 自动决定若从最终历史审计记录反向学习未治理结论，会让一次性错误污染 durable 决策（A `entity_resolution_node/02:282`）。

- 拟改：
  `BK_Copilot/memory_layers/transaction_log/02_authority_lifecycle_and_boundaries.md:负边界 / runtime 隔离` →
  `Transaction Log 永不作 runtime decision / identity / learning source，交易处理层不读它；不存可复用学习先例（归 Case Log），不变成 entity 索引学习层。governance 与未来 agent 工具读取只能当 context / 线索，不得把其内容当可复用 authority 继承。`

### 决策点 5：reusable authority vs trace —— final outcome 是审计权威，reasoning 只是留痕

- 结论：
  - **可成 authority（审计 source of truth）**：final outcome、最终 COA / HST-GST 值、`confirmed_by` 审计留痕——这是「这笔交易到底怎么了」的权威答案，审计以它为准，冲突时它优先（决策点 9）。
  - **只是 trace、永不可复用 authority**：AI 的 reasoning / audit narrative。它可供 review / correction / governance / audit 阅读，但系统**绝不**将其当成未来自动决定的依据（已锁铁律：不得把 AI reasoning 文本变成可复用 authority）。

- 为什么（锚定核心产品目标）：锚定「审计性」与「从纠正中学习不能让一次性错误污染 durable memory」。可复用 authority 只能来自已批准的 memory / rule / case / profile / governance state。

- 拟改：
  `BK_Copilot/memory_layers/transaction_log/01_memory_intent.md:authority vs trace` →
  `final outcome / 最终 COA·HST / confirmed_by 留痕是审计 authority；reasoning / audit narrative 只是 trace，可供 review/correction/governance/audit 阅读，永不成为系统可复用的判断依据。`

### 决策点 6：谁可以读

- 结论：
  - **交易处理层节点：一律不读**（runtime 隔离，已锁）。
  - **Governance / 治理环节：可读，只读、非 authority**（用户 1c：规则治理等环节可能从 Transaction Log 取信息，但交易处理层不取）。
  - **未来 agent 工具：可读作 context / 线索**——列为产品方向，**不在 M1-M2 冻结为当前 contract**（避免过度设计）。
  - 具体 retrieval / projection / permission 机制 → L4/seam。

- 为什么（锚定核心产品目标）：锚定「审计性」。读取面要写明 reader、目的与限制（模板禁止「可被其他节点读取」却不说 reader / 目的 / 限制）。

- 拟改：
  `BK_Copilot/memory_layers/transaction_log/02_authority_lifecycle_and_boundaries.md:谁可以读` →
  `交易处理层节点一律不读 Transaction Log；governance / 治理环节可读（只读、非 authority）；未来 agent 工具读取作 context / 线索列为方向、不冻结为当前 contract。retrieval / permission 机制留 L4/seam。`

### 决策点 7：谁可以直接写 / 谁只能提 candidate（写入不设额外审批闸）

- 结论：
  - **任何节点都绝不裸写 Transaction Log**（A 已锁：CJ / Coordinator / ER 一致「绝不写」）。节点只**声明要持久化什么语义**，不声明 writer / 顺序 / 机制。
  - Transaction Log 由**统一的 finalization 写入机制 / agent** 落盘（用户 1c 设想：所有 log 由一个统一 agent 负责写入）。
  - **写入 Transaction Log 本身不设 governance / 会计师额外审批闸**（用户 L2-H）：它忠实记录上游**已经定下来**的结果（该审批的在上游已审过），落盘动作本身不再过闸。
  - **无 candidate 层（用户拍板）**：Transaction Log **不存在**「先提候选、待批准再生效」的 candidate 环节。写入即 finalization 落盘（上游已审），不设待批 / 候选状态。其他记忆层（如 Case Log）的候选机制不适用于 Transaction Log。
  - exact writer、多 log finalization 顺序（Evidence / Transaction / Case / Rule / JE）、统一写入机制 → L4/seam（守运行/记忆 seam，本轮不冻结）。

- 为什么（锚定核心产品目标）：锚定「审计性」与「守 seam」。写入者声明为接口面、节点只声明语义，避免节点越权裸写造成多 source of truth；机制冻结属 M3。

- 拟改：
  `BK_Copilot/memory_layers/transaction_log/02_authority_lifecycle_and_boundaries.md:谁可以写` →
  `任何节点绝不裸写 Transaction Log，只声明要持久化的语义；Transaction Log 由统一 finalization 写入机制落盘；写入动作本身不设额外 governance/会计师审批闸（上游已审）；Transaction Log 无 candidate 层，写入即 finalization 定稿、不设待批/候选状态。exact writer、finalization trigger order、多 log 统一写入机制留 L4/seam。`

- 排除的替代 + 理由：
  排除「由各业务节点各自写入 Transaction Log」。理由：A 已锁所有节点「绝不写」；分散写入会破坏单 source of truth 与 finalization 一致性。

### 决策点 8：mutation path = append-only；更正不删旧，追加「谁改 + 改后最终结果」

- 结论：
  Transaction Log 是 **append-only（只追加、不就地覆盖）** 的审计账。它记录的是一笔交易在系统中**真实发生过的全部变化**——哪怕最初分类是错的。

  当后续 review agent 或治理节点修改了某个 entity（或修改了该 entity 名下所有交易的分类结果），导致这些 transaction 的会计分类被改变时，**不是删去 Transaction Log 中原先错误的分类，而是在后面追加一条新记录**，注明：**由谁做的更改 + 更改后的最终（最新）会计分类结果**（用户 L2-G）。

  **记录无状态机（用户拍板）**：Transaction Log 的记录**写入即固定、不做状态流转**。交易的 finalized / terminal / blocked 等是**内容字段**（enum 留 L3），不是记录自身的生命周期状态机；要更正不是改某条记录的状态，而是 append 新记录。故本层 02 文档的「Lifecycle / States」节为「无稳定状态机，状态属内容字段、enum 留 L3」。

  本轮锁定的是 **append-only 原则 + 更正以追加记录承载**；correction / reversal / split / supersession 的 **exact 触发机制、record schema、跨 log 传播（改 entity 后如何扇出到其名下交易）** → L4/seam（reversal / split 的完整产品机制按早前约定本轮搁置）。

- 为什么（锚定核心产品目标）：锚定「审计性」与「从纠正中学习」。审计 source of truth 的命脉是原始记录永不被抹除；更正以追加保留「错→改→对」的完整轨迹，正是排错与学习的依据。

- 拟改：
  `BK_Copilot/memory_layers/transaction_log/02_authority_lifecycle_and_boundaries.md:mutation path` →
  `Transaction Log 为 append-only：原始记录永不就地覆盖或删除。记录写入即固定、无状态机（finalized/terminal/blocked 等为内容字段、enum 留 L3，不是记录自身的生命周期状态流转）。后续 review / 治理更改某 entity 或其名下交易分类时，追加一条新记录注明更改者与更改后的最终会计分类结果，而非改写原记录。correction / reversal / split / supersession 的 exact 机制、record schema、跨 log 传播留 L4/seam。`

- 排除的替代 + 理由：
  排除「发现错误后就地修正原记录」。理由：会抹除「曾经错过」的事实，摧毁审计 source of truth 的不可篡改性。

### 决策点 9：边界确认 + 冲突 authority 顺序

- 结论：
  - **四 Log 分界（A 已锁）**：交互事实 → Intervention Log；身份确认 → Entity Log；可复用学习先例（stable entity + finalization proof）→ Case Log；**最终交易结果审计 → Transaction Log**；四者不互相替代。
  - **冲突 authority 顺序**：对 **final outcome 与 audit trail**，Transaction Log（或等价 finalization source）**优先**；缺少有效 finalization proof 的片段不能作为下游可复用证明。

- 为什么（锚定核心产品目标）：锚定「审计性」与「清楚区分 evidence / memory / judgment / governance / audit records」。各层各守一职，避免边界塌陷与双 source of truth。

- 拟改：
  `BK_Copilot/memory_layers/transaction_log/02_authority_lifecycle_and_boundaries.md:与其他 memory/log 的边界` →
  `四 Log 分界：交互事实→Intervention Log、身份→Entity Log、可复用先例→Case Log、最终交易审计→Transaction Log，互不替代。冲突时 Transaction Log 对 final outcome 与 audit trail 优先。`

---

## 分类备案

> 未冻结项已写入 `缺口地图.md` 新增「Transaction Log」section。

**L3（字段 / enum / schema，Stage 3·M3 冻结）：**
- Transaction Log record 的 exact 字段名 / schema / validation（含决策点 3 的 13 项语义的精确形态）。
- reasoning 在 Transaction Log 的 exact 存储字段 + 后续治理节点读取 reasoning 的 contract（联合 CJ L3）。
- rule-hit source / rule ref 在 Transaction Log 的 exact 字段形态（联合 Rule Log / Case Log L3）。
- 身份字段含「unknown」标识、情况 Y 身份待补线索的 exact 字段形态。
- terminal / blocked 交易最终状态的 exact 枚举 / 字段。
- final outcome / COA / HST / full processing path 各环节痕迹的 exact 字段结构。
- `confirmed_by` 审计留痕、AI 高置信度自动完成标记的 exact 字段。

**L4 / seam（机制）：**
- Transaction Log 的 exact writer / 统一 finalization 写入机制 / 多 log finalization 顺序（Evidence / Transaction / Case / Rule / JE）。
- append-only 更正记录的 exact 机制：触发者（review / 治理 agent）、correction record schema、改 entity 后扇出到其名下交易的跨 log 传播、「谁改 + 改后结果」的 exact 留痕形态。
- reversal / split / supersession 对 Transaction Log 的执行机制（完整产品机制本轮搁置）。
- 统计派生 / rollup / cache / maintenance job（rule-hit 统计等由 Transaction Log 逐笔记录派生）。
- governance / 未来 agent 工具读取 Transaction Log 的 exact retrieval / projection / permission 机制。
- entity-index 查询的 exact 索引 / 检索机制。

**L2·外阻（圈外依赖，挂 open boundary，绝不拿 B 填）：**
- JE Generation Node ↔ Transaction Log 的 exact handoff（JE result / journal_entry 留痕）。
- Intervention Log 与 Transaction Log 的分界 exact contract（Intervention Log 未落文）。
- Review Node 是否覆盖所有 final logging 前交易 + review trace 进 Transaction Log 的 exact 边界。
- Case Memory Update Node 凭 `transaction_log_ref` / finalization proof 写 Case Log 的 exact 机制与 trigger order。
- Transaction Log 与对外 final output report / export artifact 的 field-level 边界（report 未落文）。
- Governance / Governance Review、Knowledge Summary / Knowledge Compilation、Evidence Log、Post-Batch Lint、Profile / Structural Match 与 Transaction Log 的 exact 读写边界。
- transaction correction 如何影响 Alias review / mutation、entity risk candidate（联合 Alias Log / Entity Log，跨层 reprocessing）。

**DEFERRED（本轮搁置）：**
- correction / reversal / supersession / split 的完整产品机制（本轮只锁 append-only 原则 + 更正追加语义，机制搁置）。
- 跨导入 `transaction_id` 复用 / 迟到证据接回 / durable transaction registry / 跨运行账务幂等（沿用 Evidence Intake DEFERRED）。

---

## 自检与判定

- **按模板 M1-M2 清单自检（21 项）**：
  - 为何存在 / 为何不能由 runtime handoff 或现有 store 覆盖（决策 1）✓；存什么（决策 3）✓；绝不存什么（决策 4）✓；哪些可成 reusable authority / 哪些只是 trace（决策 5）✓；谁读（决策 6）✓；谁直接写 / 谁只提 candidate（决策 7）✓；accountant / governance approval 是否必须参与（决策 7：写入不设闸）✓；已冻结 mutation path（决策 8：append-only + 更正追加）✓；哪些 mutation path 未冻结（correction / reversal exact 机制 → L4）✓；与哪些 memory / log 边界已确认（决策 9）✓；冲突 authority 顺序（决策 9）✓；哪些 trace 留下 / 哪些 trace 不能成 authority（决策 5）✓；读者 / 写者声明为接口面（决策 6/7）✓；守运行 / 记忆 seam（决策 7：只声明语义，写入机制留 M3）✓；哪些问题未冻结（分类备案）✓。
- **本对象 L2 可否判定完成**：**是**。Transaction Log 的定位（transaction-indexed 逐笔审计层、真值单位 = 每笔交易）、覆盖面（凡有 transaction_id 即入、无损 append-only）、存什么 / 绝不存什么、runtime 隔离与未来读取护栏、authority vs trace、谁读 / 谁写 / 写入不设闸、append-only 更正语义、四 Log 边界与冲突顺序均已收口；剩余问题归 L3、L4/seam 或圈外依赖。
- **能否进 Stage 3 / M3**：**否**。record schema、reasoning / rule-hit ref / 身份字段 / terminal 状态字段形态（L3），以及 exact writer / finalization 顺序 / 更正机制 / 读取 retrieval 机制（L4/seam）均未冻结；多个下游接缝（JE Generation / Intervention Log / Review / Case Memory Update / output report）为未落文圈外依赖。需待 L3 字段定稿 + 圈外依赖落文后方可进 M3。
