> ⚠️ **已过时 / superseded（owner 2026-06-27）** — 本 L2 提案的 L1 / L2 结论已由 `BK_Copilot/workflow_nodes/human_review_node/` 正式草案取代，仅作历史来源保留。当前权威以 `BK_Copilot/` 正式草案 +（产出后）对应 L3 schema 为准；**勿据本文件回灌已删除的概念 / 字段**。

# HumanReviewNode — L2 提案

> 状态：L1-L2 主体收口（O1–O7 + C1/C2 + Chatbot + N1/N2 已拍板；"授权确认机制"折叠进本节点已确认）。本节点为 owner 新创造的临时节点，设计来源唯一为 `独立question文档/Review_Node_question.md`（**已删除 / 已归档**，充当 owner 已拍板看法的历史 provenance；正式 draft 已落 `BK_Copilot/workflow_nodes/human_review_node/`）+ 本轮对话；契约面从 A 类正式草案重建。仍有第一性原理复查暴露的若干 L1-L2 问题待讨论（见第六节）。
> 目标：集齐足以将 HumanReviewNode 从 L1 转成正式 `BK_Copilot/workflow_nodes/human_review_node/00|01|02` 的信息。
> 纪律：每条契约结构要么指回 A 类原文、要么指回用户拍板决定；`new system/` / `old_system_nodedesign/` 永不作依据。
> 待建正式文档目录（命名待最终确认）：`BK_Copilot/workflow_nodes/human_review_node/`。
> 工作区：`_audit_work/human_review_node/`（00 规则 / 01 契约面与缺口 / 02 来源说明 / 03 用户观点与问题清单）。

---

## 一、依赖（来自 `01` 契约面）

### 上游（非 pipeline handoff —— 人发起）
- **无逐笔工作流上游。** HumanReviewNode **不属于** Coordinator 所在的逐笔交易处理流水线；它由**会计师主动发起**。典型发起时机：① 多张 Bank Statement 处理完、各自生成 GE，**导入 QuickBooks 之前**整体审核；② 周审（如周五审本周全部账目，记录可能已在 QuickBooks，也可能仍在自有 DB 未导入）。（用户拍板 O1）
- **节点位置不硬、功能要求硬。**

### 读取面（A 类已声明 Review 为合法 reader）
- **Transaction Log**：读 final outcome / 最终 COA·HST-GST / confirmed_by 留痕 / processing path，供复核视图展示。A：Transaction Log 02 §2「governance / 治理环节」reader（只读，非 authority producer）。
- **Case Log**：读 relevant precedent / exception context / `use_level` / `confirmed_by`。A：Case Log 02 §2「Review Node：帮助 accountant 理解当前 review item 的历史案例上下文；Review 本身不把案例变成 rule、entity authority 或 governance approval」。
- **Entity Log**：读 active state / authority refs / candidate context（展示当前 entity 权威与候选风险）。A：Entity Log 02 §2「Review Node」reader；§2 Coordinator 行「identity risk flags 服务 Review / Governance / Post-Batch Lint」。
- **Rule Log**：读当前 executable rule 上下文，用于判定某笔是否来自 active rule。A：Rule Log 02 §2「Governance / Review / maintenance 管理面」reader。

### 下游 / 写入语义（节点备料 → 调确定性代码落盘；"授权确认"折叠进本节点）
- **Finalization 写入机制（节点外，共享，L4/seam）= 唯一真正落盘者**。本节点**自己不裸写**；备好符合各 log Finalization schema 的 Input → 调用它精准写入。它持有跨 log 原子/顺序/幂等/append-only 等不变量，并对扩张型写入**强制校验会计师凭证**（无凭证 = 拒绝放行）。A：Transaction Log 02 §3「统一 finalization 写入机制」；用户拍板 writer 模型 + O7 分工。
- **"授权确认机制"已不再独立存在**，被切两半：**面向人的那半**（提醒/摆出待确认项/read-back/接签字/带凭证转去落盘）→ **折叠进本节点**；**强制执行那半**（无凭证拒绝放行的死代码闸）→ **留在 Finalization**。本节点是这条治理线上**唯一的人类漏斗**，但只做编排、不做审批（见决策 9）。用户拍板 O7。
- **写入对象（语义，由 Finalization 落盘）**：Transaction Log（correction append）、Intervention Log（原因 + Intervention ID）、Case Log（更新/superseded）、Entity Log / Rule Log（扩张型变更：凭证→approved mutation）、**Governance Log（每笔扩张型变更的审计，需新建）**。A：Transaction Log 02 §3「review / governance 更正路径」；Case Log 02 §3、§5；Entity Log 02 §3「Review Node candidate」；Rule Log 02 §3「会计师直接创建路径」。
- **审核 inbox（数据层，pull）**：确定性发现 job / 未来 LLM 审查节点把候选投入；本节点**主动读取**（非被推送）。需新建。

### 上游补充 —— 触发源汇成同一"待确认清单"
- **A · 会计师自己主动发起**（纠错，或升级成永久 = 撰写 rule/policy）。
- **B · 确定性发现 job 投进审核 inbox 的候选**：**目前只服务 Rule Match → 当前 = rule 升级候选**（决策 13）。本节点 pull。
- **C · 未来 LLM 审查节点的发现**（接口同 B，只产候选投入；当前不存在）。

### 已锁事实（遵守，非审查；适用范围以对应正式文档为准）
- **Coordinator/Review 分界**：Coordinator 处理 running/Pending、**不得在交易已 finalized 时触发**；finalized + 人发起这条独立轴归 HumanReviewNode。A：Coordinator 02 §1。
- **Transaction Log append-only**：原始记录永不就地改；更正 = 追加新记录（更改者 + 改后最终分类）。外部审计强制，可当公理。A：Transaction Log 02 §6。
- **rule 无自动成立 / 自动降级**：一切 rule authority 变更必须 accountant sign-off + governance；系统只能提 promotion proposal。A：Rule Log 02 §3/§5。
- **治理变更只影响 future authority**：approved governance change 不重写历史 Transaction Log。A：three_nodes_warmup A.3（指向 governance_review 设计；本提案据此提出 N1 待决，见第五节）。
- **writer 模型（owner 本轮）**：其他节点功能逻辑与 authority 不变；唯一变化 = 节点自己备料并调用确定性 finalization 代码写入（旧"candidate→指定节点写入"模型已废）。

---

## 二、L2 决策

### 决策 1 — 身份与位置：流水线外、人发起、系统↔会计师的非运行期交互层 〔指向 01 §1/§2/§4、02 §1〕
**结论**：HumanReviewNode 是一个**独立于逐笔交易处理工作流**的、**会计师主动发起**的节点。它有两层职责：
- **核心职责（不变）**：按会计师指令，对系统各步骤实施修改——但**它不是修改的直接执行者**，而是通过调用多段确定性代码（Finalization）完成；Finalization 内有明确 schema 要求，部分改动（如 RuleLog）还需会计师凭证。
- **新增职责**：充当**整个系统 ↔ 人类会计师之间、关于"规则变化 + 错误治理"的沟通桥梁**（详见决策 9 的统一治理漏斗）。

**触发与边界**（用户拍板）：① 仅在**会计师主动要求**修改时动手（另一触发"系统主动发现错误经会计师审核"**挂起**——该项可能被删除，见第六节 FP-5）；② 交易**运行过程中**的所有相关问题归 **Coordinator**；③ 交易**完成之后、非 Runtime** 时间的交互与修改，全部由本节点完成。**沟通桥梁边界已锁（FP-1）**：本节点只管"规则变化 + 错误治理 + 事后纠错"三类；onboarding/目标询问/覆盖核对归 interaction_agent，运行期 pending 归 Coordinator。位置不固定（批后/导入前/周审皆可），功能要求固定。
**为什么（锚定产品目标）**：服务「accountant control」「审计性」「纠错学习」——给会计师一个事后唯一干净入口，且人类权威天然在场。不能内联进 Coordinator：Coordinator 只处理 running 交易、finalized 后不得触发（A 已锁）。
**指回**：Coordinator 02 §1；用户 1c + O1 + O7 拍板。
**排除的替代**：① 当作 Old-System「Review Node」(JE 前 gate) 的同名延续（那是记账前 gate，B 类不具权威；A 类"Review 覆盖 final logging 前交易"等引用属旧账误挂，列待写回）；② 把 onboarding/目标询问/覆盖核对也吞进来（那归 interaction_agent，本节点的桥梁只管"规则变化 + 错误治理 + 事后纠错"，见决策 9 与第六节 FP-1）。
**open boundary**：与 Coordinator / interaction_agent 是否共用同一对话前端 = 编排/呈现层（C2，本轮不决）。

### 决策 2 — 复核视图：覆盖所有已 finalized 交易，非 LLM 投影，兼作纠错入口（N2 已收口）〔指向 02 §3 读取、§6 输出〕
**结论**：复核视图是**非 LLM 的只读投影**，按 Bank Statement 聚合展示。**覆盖所有已 finalized 的交易**——High-Confidence 自动完成、Rule 产出、Coordinator 内会计师确认过的。它既是事后查看处，也是纠错入口。
- **卡住 / terminal / blocked / 未继承成功的交易不归 HumanReview**：这些尚未 finalized，**留在 Coordinator 处理**（N2 用户拍板）。Review 的对象是**已 finalized 的事实**。
**为什么**：纠错入口对已 finalized 交易不该有盲区——任何已 finalize、可能含错的交易都应可被会计师翻到并改（accountant control / 审计性）；而未 finalized 的卡住交易属运行期处置，归 Coordinator，不在事后纠错轴上。
**指回**：用户 O2 + N2 拍板；Transaction Log 02 §2 reader；Coordinator 02 §1（running/pending 归 Coordinator）。
**排除的替代**：① 只展示"自动完成"两类（会计师触达不到"当初问过但答错"的交易，盲区）；② 把卡住/未完成交易也纳入 Review 纠错（越界到 Coordinator/运行期处置）。
**open boundary**：多来源（已在 QuickBooks vs 仍在自有 DB）+ 已 finalized 多类交易**如何呈现** = L4/编排（第四节）。

### 决策 3 — 三层结构 + 成品形态 Chatbot + 三护栏 〔指向 01 §2、02 §5 决策权限〕
**结论**：节点内含三层职责，**全留节点内、非平级独立机制**：
- **第一层 理解 + 提案（LLM）**：听懂会计师自然语言纠错 → 结构化纠错意图（指哪笔/改成什么/scope 多大）；撒大网产出一份"完整影响清单"（P_llm，故意想宽，覆盖未写进依赖表的连带）。
- **第二层 影响展开（Change List Engine，确定性）**：沿设计期声明好的依赖图遍历，独立产出"已知影响清单"（P_engine，不变量型后果）。**留在节点内**（当前唯一使用者就是本节点）；裸遍历未来若被治理流复用再议。
- **第三层 执行（确定性写手）**：对账并经会计师确认后，调用节点外 Finalization 写入机制跨 log 落盘。

**成品形态 = Chatbot**：节点对会计师的成品是一个 Chatbot，作为一切前端、专责沟通；它**同时承担第一层 LLM（Change List LLM）职责**。三条护栏（用户确认）：
1. **Chatbot = 交互层（嘴），不是干活层（手）**：Engine 遍历、对账仲裁、调用 finalization 落盘均为其背后的确定性代码，不是 Chatbot 的 LLM 在算/写。守「LLM 永不写字节」。
2. **"理解纠错意图" 与 "撒网产出 P_llm" 是两个功能步骤**：可同一模型，但 P_llm 必须与 Engine **真·独立产出**（不得偷看 Engine 结果附和），否则对账兜漏功能归零。
3. **read-back 文本由该 LLM 复述 = 设计本意**：会计师批准的是系统复述版，不是原话。
**为什么**：「记忆复用 + 自动化率」要灵活性，但灵活性只活在「声明式依赖图 + 确定性遍历 + LLM 第二意见对账」里，不在任何一方自由发挥里；Engine 闭世界会静默漏未声明的连带边，LLM 第二份清单是补「开世界覆盖」的唯一手段（doc 第 2 节理由）。
**指回**：用户 1c §1 三层结构 + Chatbot 职责拍板。
**排除的替代**：把三层拆成三个平级节点/机制（owner 明确反对：同一条交互链，拆开产生上下文接缝）；让 LLM 直接落盘（违反铁律）。
**open boundary**：Change List Engine 依赖图的声明位置/形态 = L4/seam。

### 决策 4 — 对账机制：全跑双保险，不按半径省 〔指向 02 §5/§8/§9〕
**结论**：**每一笔纠错都跑** LLM 第二意见（P_llm）+ Engine（P_engine）对账（diff）：
- 一致项 → 高置信，保留；
- LLM 多出、引擎没有 → 进 read-back 让会计师裁决（是漏写进依赖表的真连带，补进表；还是 LLM 瞎编，弃）；
- 引擎有、LLM 漏 → 信引擎（已声明的死规矩）。
**为什么**：无法事前判定改动大小，分层界定困难；低半径若省掉 LLM 第二意见，就丢了"开世界兜漏"保护（用户 O3 拍板）。
**指回**：用户 O3；doc 第 2 节第二层。
**排除的替代**：按半径/新颖度分层（低半径 Engine 单跑）。理由：界定难 + 低半径丢兜漏。
**open boundary**：diff 综合的实现级成本/性能优化（非语义）→ L4。

### 决策 5 — 三条不可逾越铁律 〔指向 01 §3、02 §5/§7〕
**结论**：① **LLM 永不写字节**——只产结构化意图与提案，所有 durable 写入由确定性代码执行（格式合法但内容错的权威账目是记账系统最致命失败模式）。② **写入前强制 read-back，每一次纠错都念，不分爆炸半径**——念的是对账后的复述版，是堵语义幻觉（LLM 把会计师理解错）的唯一防线；read-back 防语义幻觉、确定性写手防格式幻觉，缺一不可。③ **scope 绝不静默升级**——"改这一笔" vs "以后永远这么走"是两个单独、明确的会计师确认，不能合并、不能由 LLM 从单笔脑补成永久。
**为什么**：审计性 + accountant control，三铁律分别堵格式幻觉、语义幻觉、权限蔓延。
**指回**：doc 第 3 节；与每个 BK_Copilot node spec「绝不能写入或修改」一致。
**排除的替代**：高置信时跳过 read-back / 合并单笔与永久确认。理由：违反铁律，破审计与控制。

### 决策 6 — 对象范围 = 系统一切已确定事实；半径只决定"做什么 + 额外治理步骤"，不决定"是否签字"（FP-2 重构）〔指向 02 §5/§6〕
**结论**：HumanReviewNode 的对象 = **系统中一切已确定的"事实"**：(a) 已完成交易；(b) 长期文档里的 Entity；(c) 已确定的 Rules。
- **关键澄清（FP-2）**：**经 Review 的一切改动天然都已会计师签字**（决策 9），节点内**没有**"判断该不该批"的分类器。半径**不是**"过不过审批闸"，只决定**做什么 + 要不要额外治理步骤**。
- **判别线不是"影不影响未来"**（连单笔纠错都经 Case Log 影响未来学习，切不开），而是 **"有没有立下/改动一条会自动驱动未来交易的确定性长期权威（rule/policy/entity 权威）"** vs **"只更新一条软的、受 use_level 约束的先例"**：
  - **改一笔记录**（更新软先例）→ Transaction Log 追加 + Intervention Log + Case Log 先例更新（决策 7）。
  - **改 rule / entity 权威 / automation**（动确定性长期权威）→ 在签字之上**额外**加：铁律 3 永久单独确认 + 跨 rule 互斥校验 + **落 Governance Log**。
**为什么**：「accountant control」系统确定的事实都应支持人工修改；区分软先例 vs 长期确定性权威，才能正确路由额外治理步骤与 Governance Log，而不误把"是否签字"当成要判断的问题。
**指回**：用户 O4 + C1 + FP-2 拍板；Entity Log 02 §3（Review = merge_split/identity-risk/policy candidate 产出方）；Rule Log 02 §3（会计师直接创建路径）。
**排除的替代**：① Review 只能改交易、不碰 Entity/Rule（与 A 及 owner 定位冲突）；② 造一个"哪些改动需签字"的判据/清单（伪问题：经 Review 的一切本就已签字，FP-2）。
**open boundary**：Engine 影响展开覆盖 entity/rule 编辑连带的依赖图形态 = L4；**改 Rule/Entity 触及历史交易的处理** = 见决策 12。

### 决策 7 — 单笔纠错级联（低半径）确定落点 〔指向 02 §4 写入、§10 trace〕
**结论**：以"一条 active rule 跑出的分类被会计师推翻"为例，经 read-back 确认后由 Finalization 机制确定落：
| 动作 | 落点 | 约束（A 来源） |
| --- | --- | --- |
| append 更正记录（谁改 + 改后最终分类） | Transaction Log | append-only，绝不就地改（TxnLog 02 §6） |
| 记"为什么改" + 生成 Intervention ID | Intervention Log | 细节在 Intervention Log（TxnLog 02 §7 四 Log 分界） |
| Intervention ID 回写挂到该交易 | Transaction Log | 只挂 ID，不存改动细节 |
| 旧先例标 superseded | Case Log | 只作废 `confirmed_by=system` 的；**不动 `confirmed_by=accountant`**；绝不删除（CaseLog 02 §5） |
| 最终确认来源改为 Accountant | 该笔 source | confirmed_by 留痕（TxnLog 02 §1） |
| 发"该 rule 被人工推翻一次"信号 | 治理/发现层 | 仅当该笔来自 active rule |
- "作废哪些先例"是**带条件的确定性分支**（按先例 `confirmed_by` 过滤），不是 LLM 判断：作废 system 确认的；遇 accountant 确认过的则停下进 read-back 让会计师确认是否一并作废。
**为什么**：审计性（append-only + ID 链路）+ 记忆复用纯净（错先例及时失效，但不污染人工确认过的）。
**指回**：doc 第 4 节；TxnLog 02 §6/§7、CaseLog 02 §5；confirmed_by 语义 = 用户拍板（exact enum 待 CaseLog M3）。
**排除的替代**：就地改写 Transaction Log（违反 append-only）；让 LLM 决定作废哪些先例（应是确定性分支）。
**open boundary**：confirmed_by exact enum（CaseLog M3，L3）。（注：原"rule 被推翻一次"信号→确定性发现累积判定已删除——rule 失效只由会计师在 review 时判断、降级人发起，系统不自发累积、不产降级候选。）

### 决策 8 — "rule 坏了 vs 一次性例外"：只由会计师在 review 判断，系统不自发判定 rule 失效 〔指向 02 §5/§9〕
**结论**：区分 (a) rule 错了→该降级 / (b) 合法例外→rule 留着只改这笔 / (c) scope 太宽→该收窄，LLM 可在本节点辅助会计师理解，但**最终判断只由会计师在 review 时做**。单次纠错对"rule 坏没坏"是弱证据，LLM **不当场裁决"rule 坏了"**，只结构化并 append 这一笔（更正记录引用该 rule，作审计事实留 Transaction Log）。**"rule 到底坏没坏 / 该不该降级"不由系统自发判断**——rule 由人创建确认、匹配出错概率低，只有会计师在 review 结果时才能发现某条 rule 失效；若判定失效，由会计师当场人发起降级 / 废除（扩张型变更，决策 6/9）。系统**不自发累积"被推翻"次数、不自发产降级候选**；确定性发现只产 rule 升级候选、不碰降级。
**为什么**：审计性 + 不让一次性错误污染 durable memory / rule authority；rule 无自动降级，且 rule 失效是语义判断、系统自发判不出，只能人在 review 时发现（A 已锁 rule 无自动降级）。
**指回**：doc 第 5 节；Rule Log 02 §3/§4。
**排除的替代**：① 纠错当场让 LLM 裁决 rule 降级（违反 rule 无自动降级）；② 让系统自发攒证据判断 rule 坏没坏、自动产降级候选（rule 失效是语义判断、系统自发判不出，只人在 review 时发现）。
**open boundary**：确定性发现 job 的调用/节奏/"够格"阈值（与 Rule Log promotion 资格耦合）→ L2·外阻/另窗。

### 决策 9 — 本节点 = 会计师的手 + 唯一人类交互面；conductor 不 judge（O7 + FP-2 收口）〔指向 02 §5/§6/§9〕
**结论（瘦身后）**：**经 Review 的每一个改动，本质上都是会计师 read-back 确认并签字过的。** 节点内**没有**"判断该不该批"的分类器——是否需要会计师同意不是 Review 判断的事，而是 Review 的运作前提。

**(1) 角色 = conductor（调度员），不是 judge（审批者）**：
- 判断者永远是会计师；本节点（及其 LLM）不自己判断、不自己批。
- **不自己做检查**：跨 rule 互斥校验、promotion 资格判定等由确定性代码做，本节点只**调用并把结果摆给会计师**，绝不用 LLM 重判。
- **不自己放行**：放行权在 Finalization 的凭证校验死代码里。理由 = **trusted base 要小**：本节点是复杂的、面向会计师的 LLM 节点，万一出 bug 或被注入，也越不过 Finalization「无凭证即拒绝」那道闸。这是**安全机制**，与"是否签字"无关（一切 Review 改动都签字）。

**(2) 工作流程**（用户拍板）：
1. 理解会计师指令 → 转成清晰的**系统变化执行图**（要改什么 + 会连带动到什么）。
2. 会计师**确认执行图**、并清楚改动后的具体变化；scope 升级（"改这一笔" vs "以后永远这么走"）必须**单独显式确认**（铁律 3）。
3. 会计师对 **read-back 版本签字** → 生成**凭证** → 本节点据各 log Finalization 要求**备好 Input** → 调用写入机制落盘。
- **Finalization 落盘中途失败**（用户拍板）：本节点能在**不瞎编信息**前提下自行解决则不打扰会计师；需会计师额外允许/信息才回到会计师。

**(3) 半径只决定额外治理步骤（非"是否签字"，见决策 6）**：动**确定性长期权威**（rule 全生命周期 / automation 放宽·升级 / Entity 治理级 merge·split·archive·确认 stable / Alias·Role）→ 在签字之上额外加 **互斥校验 + 落 Governance Log**；只改一笔记录（软先例）→ 不进 Governance Log。**Profile / 税务 / 科目映射** 是否属"长期确定性权威"按决策 6 判别线归位（不再单独枚举挂起，FP-2）。

**为什么**：accountant control + 审计性 + **read-back 是头号安全件**（Review 是会计师的手，防"手伸错地方/LLM 把会计师理解错"的唯一防线就是 read-back）；放行权放进小而死的 Finalization，使本节点即便失效也无法产生错误 durable 写入。
**指回**：用户 O7 + FP-2 + 1c 第 6/9 节；Entity Log 02 §6；Rule Log 02 §3。
**排除的替代**：① 把审批逻辑/放行权建进本节点 LLM（违反 trusted-base-要小 + LLM 永不自批）；② 保留独立"授权确认机制"节点（已折叠）；③ **造"哪些改动需签字"的分类器/清单**（伪问题，FP-2：经 Review 的一切本就已签字）。
**open boundary**：凭证 exact 形态、read-back 完整模板、Finalization 多 log 原子机制 = L3/L4；多个并发待确认项触及同一 rule/entity 的排序/互斥 = L4/seam。

### 决策 9b — 不归本节点的范围（不是"免闸"，是压根不来）〔指向 02 §1 触发、§5〕
**结论**：以下**根本不经过 Review**（在别的路径上发生）——这是**范围排除**，不是"跳过审批闸"：
- **restrictive auto-downgrade**：automation 往保守收紧 → 自动生效、仅需治理可见。A：Entity Log 02 §6 例外。
- **Coordinator 运行中的当面身份确认**：交易还在跑时的确认 → 归 Coordinator。A：Coordinator 02。
- **运行期常规写入**：如 ER 按死规矩把解析身份写进 Entity Log → 直接走 Finalization。A：Entity Log 02 §3。
> 注：**低半径单笔纠错不在此列**——它是 Review 的改动、照样 read-back+签字（决策 9），只是不触发额外治理步骤（互斥/Governance Log）。
**为什么**：保持本节点职责面干净——只做"会计师事后、非 runtime 的纠错与治理交互"，运行期日常写入与收紧型动作不归本节点。
**指回**：用户 O7 第 6 节 + FP-2；A 各 log。

### 决策 10 — 决策权限表 〔指向 02 §5〕
| 谁 | 可以决定 |
| --- | --- |
| Deterministic code（节点内/调用） | 沿依赖图展开 P_engine / 执行图；按 `confirmed_by` 过滤作废范围；append/ID 链路/不变量校验；跨 rule 互斥校验、promotion 资格判定（本节点只取结果） |
| Finalization 死代码（节点外） | **凭证校验 = 放行权**（无凭证拒绝扩张型写入）；多 log 原子/顺序/幂等/append-only |
| LLM（本节点，conductor） | 听懂纠错、结构化意图、产执行图、撒网产出 P_llm、区分 rule 错/例外/范围过宽（**仅提案、发信号**）、做前置会计分析并给会计师选项、组织 read-back |
| LLM 不能 | 自批/放行；当场裁决某 rule"坏了"并执行降级；把 scope 从单笔升级为永久；重判确定性检查；任何 durable 写入 |
| Accountant（唯一判断者） | read-back 最终确认 + **签字**（= 这条治理线上唯一的批准）；diff 中 LLM 多出项是真连带还是幻觉；是否一并作废人工确认过的先例；scope 是否升级为永久；扩张型变更是否通过 |
| Governance Log | 不是决策者，是**审计账本**：每笔经会计师签字、落盘的扩张型变更在此留痕 |
**指回**：用户 O7；doc 第 7 节；与 Coordinator/CJ 等 A 类决策权限分层一致。

### 决策 11 — HST/GST 重算：查表优先，缺失则停下 read-back 不猜；LLM 先分析给选项 〔指向 02 §7 证据不足〕
**结论**：改分类 ⇒ 重算税。税率适用**查表优先**（依赖 entity 已记录的税务状态字段）；**字段缺失 → 引擎撞到缺失输入 → 停下进 read-back，不交 LLM 猜**。且系统不把问题原封抛给会计师——LLM 先用会计知识分析、给选项，最终人定（参照 Coordinator 二次交互模式）。
**为什么**：审计性（税不可猜）+ accountant control + 降低会计师负担。
**指回**：用户 O5 拍板；doc 第 4 节"重算税" + 第 7 节 open boundary #3。
**排除的替代**：字段缺失时让 LLM 推断税率。理由：错税是不可接受的权威账目错误。
**open boundary**：entity 是否真有税务状态字段 = Entity Log L3/外阻（缺失时按上述"停下"处理，不卡本节点）。

### 决策 12 — 改一条 rule 牵连历史交易：单笔 vs 全部 + 回溯纠正的确定落点（N1 已收口）〔指向 02 §6/§8、§10〕
**结论**：当 Review 探查到"被推翻的这笔来自一条 active rule、该 rule 名下已有 N 笔关联交易"时：
1. **询问与确认**：Agent 先向会计师说明"这是过去定的一条分类规则、已产生 N 笔关联交易"，并问：**只改这一笔，还是该 rule 名下全部？**（属铁律 3「scope 绝不静默升级」——单笔 vs 全部是会计师的显式选择，LLM 不脑补。）
2. **若会计师选"全部"——回溯纠正是真做的**，按层落点：
   - **Case Log**：把这 N 条先例记录**直接改成新分类、`confirmed_by=accountant`**。Case Log 遵循"记录即发生"、**不承担审计作用**，故只保留更正后的可复用状态、**不背版本历史**（旧值的审计痕迹在 Transaction Log + Intervention Log）。
   - **Transaction Log**：对每笔**追加一条新记录**（HumanReview 把最终会计分类改为新结果 + 附 Intervention ID），**原记录绝不删除/覆盖**——这正是"过去数据不能改"的真义：禁止就地改写，而非禁止纠正。
3. **rule 本身的去向是单独一问**（高半径，见决策 9）：会计师选 **彻底废除 / 更新该 rule 结果（以后按新结果展示）/ 废弃并降级**。
**为什么**：澄清"过去数据不可改"= Transaction Log 审计不可变（只 append），不等于不纠正历史；Case Log 是可复用记忆层（非审计层），可就地更新到正确态；rule 去向与历史交易纠正是**两件事、两次确认**（审计性 + 权限不蔓延 + accountant control）。
**指回**：用户 N1 拍板；Transaction Log 02 §6（append-only 更正追加）；Case Log 02 §1/§5（可复用先例、非审计、correction 触发 supersede/更新）、§3（Review/accountant finalization path）。
**排除的替代**：① 改 rule 自动回溯改全部历史（违反 scope 不静默升级 + 该问会计师单笔还是全部）；② 改 rule 只影响未来、历史一律不动（与"事后纠错"本旨冲突，且 owner 明确历史可经 append 纠正）。
**open boundary**：N 笔批量纠正的执行编排（逐笔 append 的顺序/原子）= Finalization 机制 L4/seam；Case Log "记录即发生 / 不留版本历史"的 exact 字段语义待与 Case Log M3 对齐。

### 决策 13 — 候选路径：rule 升级走固定执行路径，Change List Engine 只服务纠错与降级/改动（FP-3 收口）〔指向 02 §3 读取、§6〕
**结论**：
- **确定性发现 job 目前只服务 Rule Match 一个节点** → 当前 inbox 候选 = **rule 升级候选**（某 entity 下某 pattern 够格升 rule）。
- **直来直去的 rule 升级 = 固定化执行路径**：升完要写什么非常明确，**不走开放式 Change List Engine**，走一条固定路径；本节点只当**确认面 + 触发器**（把候选 + 证据摆给会计师签 → 触发固定路径）。
- **若涉及降级 / 改动现有 rule** → 才回到 Change List Engine 展开连带 + 决策 12 的历史处理。
→ 故 **Change List Engine 主要服务"纠错"与"降级/改动"的连带展开**；**纯升级不需要 Engine**。
**为什么**：最小设计——固定后果不必动用开放式展开器；保持本节点薄。
**指回**：用户 FP-3 拍板；Rule Log 02 §3/§4（promotion 须会计师 sign-off、附 CaseLogEvidence）。
**排除的替代**：用通用 Change List Engine 展开一切候选（升级后果是固定的，开放展开多余）。
**open boundary**：rule 升级"固定执行路径"的**定义归属** = 倾向 Rule 侧（Rule Log / Rule Match 治理），本节点只触发 → **L4/seam（NEW-1）**；确定性发现 job 调用/节奏/阈值 → L2·外阻。

---

## 三、分类备案

### L3（字段 / enum / schema，留 Stage 3）
- `confirmed_by` exact enum（Case Log M3，决定作废过滤）；Intervention ID / correction record schema；Change List（P_llm / P_engine）schema；entity 税务状态字段；source 字段形态；完整 read-back 模板 / 字段 / data contract。

### L4 / seam（机制，挂 open boundary，节点只声明"存什么+谁有权威"）
- Change List Engine / 执行图 依赖图的**声明位置/形态**（含覆盖 entity/rule 编辑连带的遍历）。
- Finalization 写入机制的调用契约 / schema / 凭证校验 / 写入顺序 / 原子·回滚 / 幂等键。
- 凭证（credential）的 exact 形态、read-back 完整模板。
- 确定性发现 job 的调用 / 节奏 / "够格"阈值。
- 多个并发待确认项触及同一 rule/entity 的排序 / 互斥 / 去重。
- read-back UI 与只读复核视图是否同一 UI shell；多来源（QuickBooks vs 自有 DB）+ 多类 finalized 交易的**呈现方式**（编排/呈现层）。

### L2·外阻（圈外依赖，挂 open boundary，绝不拿 B 填）
- **Intervention Log** 正式 spec（schema / writer / 与 Evidence·Review·Governance 边界）。
- **Governance Log** 数据层（当前缺件，**必须新建**；扩张型变更审计账本）。
- **审核 inbox** 数据层（确定性发现 / 未来 LLM 审查节点投候选，本节点 pull；需新建，≠ Governance Log）。
- **Finalization 写入机制 / 确定性发现 job**（共享机制，另窗细化；"授权确认机制"已折叠，不再单列）。
- **语义发现器 merge/split（系统自发那部分）**：挂起，另开窗口（记忆 `semantic-discovery-node-necessity-open`）。本节点只承接**人发起**的 merge/split。
- **rule 升级"固定执行路径"的定义归属**（NEW-1）：倾向 Rule 侧（Rule Log/Rule Match 治理）定义、本节点只触发 → 待对齐。
- **与 Coordinator / interaction_agent 是否共用同一对话前端**（编排/呈现层，C2）。
> 注：原"Profile/税务/科目映射是否归本节点"已由 FP-2 判别线（确定性长期权威 vs 软先例）自动归位，不再单挂。

---

## 四、自检与判定（Stage 1-2 清单）

对照模板 §10 二十二问：1 为什么存在✓(决1)、2 不可删并内联✓(决1)、3 唯一核心职责✓(决1)、4 上游✓(决1，人发起)、5 下游✓(依赖)、6 读哪些 log✓(读取面)、7 直接写什么✓(决7，经 Finalization 机制，节点不裸写)、8 只能提什么 candidate✓(决13，rule 升级候选来自确定性发现)、9 绝不能写什么✓(铁律1，LLM 不写字节)、10 deterministic 决定✓/11 LLM 判断✓/12 accountant 决定✓/13 governance 批准✓(决10)、14 证据不足✓(决11，停下 read-back)、15 歧义✓(决4 对账→read-back 裁决)、16 source conflict✓(决12，改 rule 牵连历史 = 单笔/全部 + append 回溯)、17 留哪些 trace✓(决7，Intervention ID 链路 / correction append)、18 trace 不成 authority✓(铁律 + A)、19 对外契约面 + consumer✓(依赖/写入语义)、20 守运行/记忆 seam✓(决3/9，节点只声明存什么+谁有权威)、21 哪些未冻结✓(三节)、22 能否进 Stage 3 → **否**。

- **能否判定 L1-L2 完成**：O1–O7 + C1/C2 + Chatbot + N1/N2 + FP-1~FP-5 全部收口（FP-2 改为"经 Review 一切皆签字、半径只路由额外治理步骤"；FP-3 改为"rule 升级走固定路径"）；**L1-L2 主体已完整**，仅余 NEW-1 一条 seam 待与 Rule 侧对齐（不阻塞 L1-L2 判定）。
- **能否进 Stage 3**：否。卡两类阻塞——① 圈外依赖落文（Intervention Log / Governance Log / 审核 inbox / Finalization 机制 / 确定性发现 / 语义发现器）；② L3 字段定稿。

---

## 五、本轮已收口
- **决策 6/9/9b（FP-2 重构 + O7）**：经 Review 的一切改动天然已会计师签字；**节点内无"该不该批"分类器**；半径只决定"做什么 + 额外治理步骤（互斥/Governance Log）"，判别线 = "确定性长期权威 vs 软先例"（非"影不影响未来"）；conductor 不 judge；放行权在 Finalization 凭证死代码（安全机制，trusted base 小）；9b 改为"范围排除"而非"免闸"。
- **决策 13（FP-3）**：确定性发现只服务 Rule Match → inbox = rule 升级候选；直来直去升级走**固定执行路径**（不用 Engine），降级/改动才用 Engine + 决策 12。
- **决策 12（N1）/决策 2（N2）/决策 8（O6）**：改 rule 牵连历史的单笔/全部 + append 回溯；卡住交易归 Coordinator；Review 接两类输入、rule 坏没坏只由会计师在 review 判断（系统不自发判定失效、不产降级候选）。
- **read-back 抬升为头号安全件**：Review = 会计师的手，防"LLM 把会计师理解错"的唯一防线。

## 六、第一性原理复查 —— 结论与剩余 seam
- **FP-1 ✓**：桥梁边界锁为"规则变化 + 错误治理 + 事后纠错"；面宽但权薄。
- **FP-2 ✓（已消解，非我原方案）**：不造"该不该批"判据/清单——经 Review 一切皆签字。半径只路由额外治理步骤；判别线 = 确定性长期权威 vs 软先例。
- **FP-3 ✓（按用户细化）**：rule 升级 = 固定路径；Engine 只服务纠错与降级/改动。
- **FP-4 → L4**：执行图"一次同意全部 vs 逐笔"留 L4；中途失败规则已定（决策 9）。
- **FP-5 → 搁置**：触发"系统主动发现错误"可能删除，入口纪律不议。
- **NEW-1（剩余 seam，不阻塞 L1-L2）**：rule 升级"固定执行路径"的定义归属——倾向 **Rule 侧（Rule Log/Rule Match 治理）**定义、本节点只触发。待 Rule 侧对齐确认。
- **第一性原理总评**：FP-2/FP-3 厘清后为**净简化**（少一个分类器、少一套漏斗审批叙事），未发现需推翻的结构。
