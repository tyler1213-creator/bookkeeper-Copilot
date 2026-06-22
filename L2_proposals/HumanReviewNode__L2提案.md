# HumanReviewNode — L2 提案

> 状态：L1-L2 主体收口（O1–O7 + C1/C2 + Chatbot + N1/N2 已拍板；"授权确认机制"折叠进本节点已确认）。本节点为 owner 新创造的临时节点，设计来源唯一为 `独立question文档/Review_Node_question.md`（充当 owner 已拍板看法）+ 本轮对话；契约面从 A 类正式草案重建。仍有第一性原理复查暴露的若干 L1-L2 问题待讨论（见第六节）。
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

### 上游补充 —— 三类触发源汇成同一"待确认清单"
- **A · 会计师自己的纠错升级成永久**（"以后这个 entity 都这么走" = 撰写 rule/policy）。
- **B · 确定性发现 job 投进审核 inbox 的候选**（系统自发扫出的阈值型机会）。
- **C · 未来 LLM 审查节点的发现**（接口同 B，只产候选投入）。

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

**触发与边界**（用户拍板）：① 仅在**会计师主动要求**、或**系统主动发现错误并经会计师审核通过**时才修改；② 交易**运行过程中**的所有相关问题归 **Coordinator**；③ 交易**完成之后、非 Runtime** 时间的交互与修改，全部由本节点完成。位置不固定（批后/导入前/周审皆可），功能要求固定。
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

### 决策 6 — 对象范围 = 系统一切已确定事实；半径决定走不走授权闸（C1）〔指向 02 §5/§6〕
**结论**：HumanReviewNode 的纠错对象 = **系统中一切已确定的"事实"**：(a) 已/未完成交易；(b) 长期文档里的 Entity；(c) 已确定的 Rules。判定标准：
- **只动这一笔交易自己**（含"这笔其实该指向另一个 entity"）→ 交易级、**低半径**，走单笔级联（决策 7）、**不过授权闸**（人已在场、不产生扩张型长期权威）。
- **改 Entity 记录本身 / 改·降级 Rule / merge·split / automation 放宽** → **扩张型长期权威**，**一律过本节点的统一治理漏斗（决策 9）：会计师签字→凭证→Finalization 落盘 + Governance Log 审计**。会计师经 Review 直接改 rule = Rule Log §3「会计师直接创建路径」（带本人签字，合法）。
**为什么**：「accountant control」——系统确定的事实都应支持人工修改；「审计性 + 权限不蔓延」——扩张型变更必经唯一强制闸。
**指回**：用户 O4 + C1 拍板；Entity Log 02 §3（Review = merge_split/identity-risk/policy candidate 产出方）；Rule Log 02 §3（会计师直接创建路径）。
**排除的替代**：Review 只能改交易、不碰 Entity/Rule。理由：与 A（Entity Log 已列 Review 为 candidate 产出方）及 owner 的"交互层=系统一切事实"定位冲突。
**open boundary**：Engine 影响展开覆盖 entity/rule 编辑连带（改 entity X ⇒ 扇出其名下交易）的依赖图形态 = L4；**改 Rule/Entity 触及历史交易的处理** = 见决策 12（N1 已收口）。

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
**open boundary**：confirmed_by exact enum（CaseLog M3，L3）；"rule 被推翻一次"信号的 exact 落点与累积判定（归确定性发现/Rule Log promotion，L2·外阻）。

### 决策 8 — "rule 坏了 vs 一次性例外"：LLM 唯一真判断，但降级，不当场裁决 〔指向 02 §5〕
**结论**：区分 (a) rule 错了→该降级 / (b) 合法例外→rule 留着只改这笔 / (c) scope 太宽→该收窄，是 LLM 在本节点**唯一不可替代的认知内核**。但**降级了刻度**：单次纠错对"rule 坏没坏"是弱证据，LLM **不在纠错当场裁决"rule 坏了"**——只做两件：结构化这一笔的纠正 + 发"rule 被推翻一次"信号。"rule 到底坏没坏"是另一个**攒证据**的过程（N 次推翻 / 跨 case 模式），归节点外的确定性发现 / Rule Log promotion，且本身可复核。
**为什么**：审计性 + 不让一次性错误污染 durable memory / rule authority；rule 无自动降级（A 已锁）。
**指回**：doc 第 5 节；Rule Log 02 §3/§4。
**排除的替代**：纠错当场让 LLM 裁决 rule 降级。理由：单次为弱证据 + 违反 rule 无自动降级。
**open boundary**：确定性发现 job 的调用/节奏/"够格"阈值（与 Rule Log promotion 资格耦合）→ L2·外阻/另窗。

### 决策 9 — 统一治理确认漏斗：本节点是唯一的人类漏斗，但是 conductor 不是 judge（O7 收口）〔指向 02 §5/§6/§9〕
**结论**：系统里**任何需要会计师点头的"扩张型长期变更"**，没有别的路，全部汇到本节点的确认面，再到达会计师、再到达 Finalization。本节点是这条治理线上**唯一的人类漏斗**。

**(1) 角色 = conductor（调度员），不是 judge（审批者）**：
- 判断者永远是会计师；本节点（及其 LLM）不自己判断。
- **不自己做检查**：跨 rule 互斥校验、promotion 资格判定等由确定性代码做，本节点只**调用并把结果摆给会计师**，绝不用 LLM 重判。
- **不自己放行**：放行权在 Finalization 的凭证校验死代码里，不在本节点 LLM 里。理由 = **trusted base 要小**：本节点是复杂的、面向会计师的 LLM 节点，万一出 bug 或被注入，也越不过 Finalization「无凭证即拒绝」那道闸。

**(2) 工作流程**（用户拍板）：
1. 理解会计师指令 → 转成清晰的**系统变化执行图**（要改什么 + 会连带动到什么；纠错路径的连带由节点内 Change List Engine 沿依赖图展开，候选路径把候选影响范围摆清）。
2. 会计师**确认执行图**、并清楚改动后的具体变化后；scope 升级（"改这一笔" vs "以后永远这么走"）必须**单独显式确认**（铁律 3）。
3. 会计师对 **read-back 版本签字** → 生成**凭证** → 本节点据各 log Finalization 要求**备好 Input** → 调用相应写入机制落盘 + 往 **Governance Log** 记一笔审计。**无签字 = 无凭证 = Finalization 拒绝，任何 durable 改动都不发生。**

**(3) 需经本漏斗的"扩张型长期变更"清单**：Rule 全生命周期（创建/promotion/修改/降级/删除）；automation policy 放宽/升级；Entity 治理级变更（merge/split/archive/生命周期，及把候选确认成 stable）；Alias 批准/拒绝、Role 确认；永久纠错（来源 A）。**Profile / 税务配置 / 科目映射变更归属未定 → 先挂，别默认全吞**（第六节 FP-2）。

**为什么**：权限不蔓延 + 审计性 + 单一可审计漏斗（正对应产品上"网页一个统一审核板块"）；把放行权放进小而死的 Finalization，使本节点即便失效也无法产生错误 durable 写入。
**指回**：用户 O7 + 1c 第 6/9 节；Entity Log 02 §6 candidate→approval→mutation→Governance Log；Rule Log 02 §3（无自动成立、须会计师 sign-off）。
**排除的替代**：① 把审批逻辑/放行权建进本节点 LLM（违反 trusted-base-要小 + LLM 永不自批）；② 保留一个独立的"授权确认机制"节点（已折叠：人面那半进本节点、强制半留 Finalization）。
**open boundary**：凭证 exact 形态、read-back 完整模板、Finalization 多 log 原子机制 = L3/L4（留 Stage 3）；多个并发待确认项触及同一 rule/entity 时的排序/互斥 = L4/seam。

### 决策 9b — 明确不经过本漏斗 / 不归本节点（免闸清单）〔指向 02 §1 触发、§5〕
**结论**：以下本来就**免闸**，绝不接进来：
- **低半径单笔纠错**：只改当前这一笔、不产生扩张型长期权威，且发起人即会计师本人（人类权威已在场）→ 走纠错落盘（决策 7），不经漏斗。
- **restrictive auto-downgrade**：automation 往保守收紧 → 免审、自动生效、仅需治理可见，不经本节点（**只有放宽才过闸**）。A：Entity Log 02 §6 例外。
- **Coordinator 运行中的当面身份确认**：交易还在跑、Coordinator 当场问会计师的确认 → 归 Coordinator。A：Coordinator 02。
- **运行期常规写入**：如 ER 按死规矩把解析身份写进 Entity Log → 直接走 Finalization，不碰本节点。A：Entity Log 02 §3。
**为什么**：保持漏斗的意义与本节点职责面干净——只管"扩张型长期权威变更"，运行期日常写入与收紧型动作都不归本节点。
**指回**：用户 O7 第 6 节；A 各 log。

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
- **Profile / 税务配置 / 科目映射变更**是否归本节点确认范围：未定，先挂（FP-2）。
- **与 Coordinator / interaction_agent 是否共用同一对话前端**（编排/呈现层，C2）。

---

## 四、自检与判定（Stage 1-2 清单）

对照模板 §10 二十二问：1 为什么存在✓(决1)、2 不可删并内联✓(决1)、3 唯一核心职责✓(决1)、4 上游✓(决1，人发起)、5 下游✓(依赖)、6 读哪些 log✓(读取面)、7 直接写什么✓(决7，经 Finalization 机制，节点不裸写)、8 只能提什么 candidate✓(决6/9，扩张型变更 candidate 经授权闸)、9 绝不能写什么✓(铁律1，LLM 不写字节)、10 deterministic 决定✓/11 LLM 判断✓/12 accountant 决定✓/13 governance 批准✓(决10)、14 证据不足✓(决11，停下 read-back)、15 歧义✓(决4 对账→read-back 裁决)、16 source conflict✓(决12，改 rule 牵连历史 = 单笔/全部 + append 回溯)、17 留哪些 trace✓(决7，Intervention ID 链路 / correction append)、18 trace 不成 authority✓(铁律 + A)、19 对外契约面 + consumer✓(依赖/写入语义)、20 守运行/记忆 seam✓(决3/9，节点只声明存什么+谁有权威)、21 哪些未冻结✓(三节)、22 能否进 Stage 3 → **否**。

- **能否判定 L1-L2 完成**：O1–O7 + C1/C2 + Chatbot + N1/N2 已收口（含"授权确认机制折叠进本节点"）；**尚余第六节 FP-1~FP-5 第一性原理复查项**讨论后才判定完成。
- **能否进 Stage 3**：否。卡两类阻塞——① 圈外依赖落文（Intervention Log / Governance Log / 审核 inbox / Finalization 机制 / 确定性发现 / 语义发现器）；② L3 字段定稿。

---

## 五、本轮已收口
- **决策 1/9/9b/10 + 依赖（O7）**：授权确认机制折叠进本节点（人面那半），强制半留 Finalization；本节点 = 唯一人类漏斗 + conductor 不 judge；三来源、扩张型清单、工作流程、免闸清单、凭证→Finalization 落盘已写明。
- **决策 12（N1）**：改 rule 牵连历史 = 先问单笔/全部；全部则 Case Log 改记录、Transaction Log 逐笔 append；rule 去向单独确认。
- **决策 2（N2）**：卡住/未完成交易归 Coordinator；Review 对象 = 已 finalized 事实。
- **决策 8（O6）**：Review 接两类输入；对"rule 坏没坏"职责到"发信号"为止。

## 六、第一性原理复查 —— 仍需讨论的 L1-L2 问题
- **FP-1 沟通桥梁的边界**：决策 1 说本节点是"系统↔会计师"的沟通桥梁，但这措辞太宽。需钉死它**只**管"规则变化 + 错误治理 + 事后纠错"；onboarding/目标询问/覆盖核对归 interaction_agent、运行期 pending 归 Coordinator。三者对会计师的对话面如何分工 = 待议（这是"god-node"风险的正面回答：面宽但权薄——只在会计师请求或已批准的发现错误时动手、只在非 runtime、且 conductor 不 judge）。
- **FP-2 确认范围的外延**：Profile / 税务配置 / 科目映射变更归不归本节点漏斗？owner 已说"先挂"。需要一个判据来决定"什么算扩张型长期变更"，而不是逐个枚举。
- **FP-3 候选路径 vs 纠错路径的影响展开**：纠错路径由节点内 Change List Engine 展开连带；**inbox 候选（来源 B/C）的"执行图/影响范围"由谁产出**——本节点也跑 Engine，还是候选到达时已附带影响？决定 Engine 是"纠错专用"还是"通用展开器"。
- **FP-4 会计师签的"执行图"批量边界**：当一次确认牵连 N 笔（决策 12）或一个候选触及多对象时，会计师签的是"一个整体执行图"还是逐项？部分失败时（Finalization 落盘中途失败）本节点对会计师呈现什么——这关系到"他批准的到底是什么"。
- **FP-5 "系统主动发现的错误"的入口纪律**：决策 1 触发条件之一是"系统主动发现错误经会计师审核通过"。这类错误（≠ 阈值型 promotion 候选，而是"疑似记错了"）由谁发现、经哪个 inbox、与确定性发现 job 是否同一队列 = 待厘清。
