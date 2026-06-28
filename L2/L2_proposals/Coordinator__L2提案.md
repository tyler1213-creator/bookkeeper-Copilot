> ⚠️ **已过时 / superseded（owner 2026-06-27）** — 本 L2 提案的 L1 / L2 结论已由 `BK_Copilot/workflow_nodes/coordinator_node/` 正式草案取代，仅作历史来源保留。当前权威以 `BK_Copilot/` 正式草案 +（产出后）对应 L3 schema 为准；**勿据本文件回灌已删除的概念 / 字段**。

# Coordinator 节点 — L2 提案

> 状态：L1-L2 本轮收口（决策 1–15 + 决策7(0)/F1 已与用户讨论确认，Q12–Q16 已落盘；第一性原理复查无剩余待拍板项）。进 Stage 3 待圈外依赖落文 + L3/L4 单独推进，见第四/五节。
> 目标：集齐足以将 Coordinator 从 L1 转成正式 `BK_Copilot/workflow_nodes/coordinator_node/00|01|02` 的信息。
> 纪律：每条契约结构要么指回 A 类原文、要么指回用户拍板决定；B 类（new/old system）永不作依据。
> 待建正式文档目录（命名待最终确认）：`BK_Copilot/workflow_nodes/coordinator_node/`。
> 工作区：`_audit_work/coordinator/`（00 规则 / 01 契约面与缺口 / 02 热身与防污染）。
> 历史来源说明：下文多处「指回 Coordinator Question.md」中的 `Coordinator Question.md` 历史速记**已删除 / 已归档**，其内容已吸收进本提案与 `BK_Copilot/workflow_nodes/coordinator_node/` 正式草案；这些指回仅作历史 provenance，不再是可读入口。

---

## 一、依赖（已锁上游 + 下游，来自 `01` 契约面）

### 上游（已锁）
- **Case Judgment（唯一上游输入）**：CJ 输出的 **Pending 是 Coordinator 的唯一输入**。A：「Coordinator：Pending 的唯一 consumer」（CJ 01 §4 / CJ 02 §6）。身份卡点（ER `unknown`）也经 ER→CJ→Pending 传入；Coordinator 不直接消费 ER 输出。（用户 Q3 拍板）
  - Pending handoff 最小语义（A，CJ 02 §6）：① 客户/identity 背景；② 相关 Case Log 先例摘要（仅 stable entity）；③ 阻塞原因；④ 已执行工作及结果；⑤ 可选推断性建议与不确定项。

### 下游（部分圈外）
- **JE Generation Node**：会计师答复足够后，Coordinator 路径「转 JE 所需格式」。A：CJ 02 §6。**JE Generation 未落文 → open boundary**（缺口地图 CJ L2·外阻）。
- **Entity Log（+Alias Log）**：会计师明确确认身份后，Coordinator 路径触发 stable entity 创建。A：Entity Log §3「Accountant explicit identity confirmation path」、§6 第二条 mutation path。机制 = L4/seam。
- **Transaction Log**：情况 Y 身份缺口只 finalize 到 Transaction Log。A：Case Log §7、Coordinator Question.md。Transaction Log 未落文 → open boundary。
- **后续 governance / review / case memory 路径**：消费 Coordinator 暴露的长期记忆/治理风险 candidate。（圈外）

### 已锁事实（遵守，非审查；具体适用范围以对应正式文档为准）
- 身份判断 `stable`/`unknown` 锁在 ER；会计分类判断权锁在 CJ；AI 联网身份用途锁 ER、会计事实用途锁 CJ。
- accountant identity confirmation 创建 stable entity 无需 governance approval，只确认 identity，不隐含分类/Alias/Rule/policy/Case Log 写入（Entity Log §3）。
- `Role` 字段已删除，本轮不重启用（Coordinator Question.md）。

---

## 二、L2 决策

### 决策 1 — 核心职责：拆解 + 沟通 + 打包，不做会计判断 〔指向 01_functional_intent §1/§2/§3、02 §5〕

**结论**：Coordinator 是 CJ Pending 的 runtime 接手节点；核心职责是把 CJ 的 Pending 转成会计师可读的结构化提问，理解会计师回答，**追问直到信息足以输出 JE**，再把结果打包给下游。它**不运用会计知识自己做分类判断、不重跑 CJ**；会计师的答复即 authority。
- 三段机能（用户拍板）：
  - (a) 理解来自 CJ 的输入，向会计师做**结构化提问**；
  - (b) 理解会计师回答，**判断是否足以生成 JE**；
  - (c) 它**仍需具备会计知识**，用于两件事：① 判断「会计师当前提供的信息是否完整/可落地」；② **利用会计知识 + 当前交易辅助证据，形成「带选项的 pending 问题」给会计师选择**（降低会计师的思考路径复杂度与负担）。本质是「拆解 + 沟通」节点。

**决策 1 的会计知识使用边界（Q11 拍板）**：
1. Coordinator **不可自己给出最终分类判断、不可下结论**。
2. 它可以用会计知识 + 辅助证据**形成带选项的 pending 问题**让会计师选择。
3. **结果可写入的唯一条件 = 会计师同意**（确认即 authority；未确认不落地、不重入 CJ）。
> 说明：这与「Coordinator 不做会计判断」原定义有轻微张力，但用户判定其必要——它降低会计师判断负担、服务自动化与效率，且通过"会计师确认为唯一写入条件"守住控制权与审计性。

**为什么（锚定产品目标）**：保「accountant control」与「干净人机分界」——机器拿不准时干净交回会计师，且不让单个 runtime 节点僭越会计分类权；保「审计性」——会计师答复是可追溯 authority，Coordinator 不新增不可追溯判断。
**指回**：CJ 01 §4「Coordinator 负责人工交互，不应在上下文不全时重做 CJ 的会计分析」；ER 02 §7 Problem 4「分类在 Coordinator 与 accountant 的交互中完成」「已确认交易不重入 CJ」；用户 1c #1 + Q1。
**排除的替代**：让 Coordinator 自行做会计分类 / 重跑 CJ。理由：与 CJ 分类权、ER 不重入约束冲突，且使 Coordinator 变成第二分类器，损审计性。
**open boundary**：「转 JE 所需格式 contract」依赖未落文 JE Generation → L2·外阻。

### 决策 2 — 触发粒度：批级消费，不持控制流 〔指向 02_logic §1/§2〕

**结论**：Coordinator 锁为 **批级**——消费「一张 Bank Statement 跑完后的整批 Pending 集合」。**Coordinator 只消费，不控制、不保证整张表跑完**；「整表先处理完」属编排层控制流，不写进 Coordinator 内部逻辑。Coordinator 定位为 **runtime 交互通道**（onboarding 期交互不归本节点）。
**为什么**：批级消费是聚合提问（决策 4）的前提；把控制流留在编排层，避免 Coordinator 变成持有路由权的自主 agent（违反「不把模型写成自主控制流 agent」）。
**指回**：CJ 02 §3「批次定义和一次提交多张表的排序属于 Coordinator / 编排层」；interaction_agent_question.md（前期问目标/覆盖核对归交互 Agent，非 Coordinator）；用户 1c #4/#5 + Q2。
**排除的替代**：逐笔触发 Coordinator（无法聚合）；Coordinator 自己驱动「跑完整表」的控制流（越权成 agent）。
**open boundary**：多表提交的串行/并行排序、批起始记忆快照 → L4/seam（缺口地图 Coordinator section）。

### 决策 3 — 输入契约面：只收 CJ 的 Pending 〔指向 01 §4 读取、02_logic §2/§3〕

**结论**：Coordinator 的输入本轮锁定为**单一来源：CJ 输出的 Pending 集合**。身份卡点已并入 CJ Pending 流入（原 ER `candidate_signal` 等并行候选通道已删，见 Decisions D1）。
**为什么**：A 仅锁「Pending 唯一 consumer = Coordinator」；其余触发源在 A 无依据，按铁律不据 B 补宽。单一入口也使契约面最小、最可审计。
**指回**：CJ 02 §6；用户 Q3。
**排除的替代**：照 B（new system）把触发源画成 EI/Transaction Identity/Profile/ER/RuleMatch/orchestrator 全可触发。理由：B 不具权威，且含已删除的 Transaction Identity 节点；A 无此依据。
**open boundary（已关闭）**：原"ER `candidate_signal` 是否最终也进 Coordinator"一项已随 `candidate_signal` 删除而关闭（见 Decisions D1）；Coordinator 输入面仍锁 CJ Pending。

### 决策 4 — 聚合提问：同表内按 entity 聚合「身份提问」，分类逐笔独立 〔指向 02_logic §6 输出 / §8〕

**结论**：
- **范围锁定在同一张 Bank Statement 内**（跨表/跨运行聚合本轮不做）。
- **身份提问可按 entity 聚合**：同一张表内同一对象（exact same surface / 同一 entity）的身份提问合并为一次，会计师答一次，套用到该表内所有同对象交易。
- **分类/会计处理逐笔独立提问**：即便 entity 相同，每笔交易的最终分类仍需独立询问，不得因 entity 相同武断合并分类。
**为什么**：身份是单一事实（可扇出），分类逐笔不同（不可扇出）——区分二者是正确性边界，不只是体验；范围锁单表避免引入未冻结的等价/去重机制。锚定「自动化率」（减少会计师重复回答身份）与「分类正确性/审计性」（分类不被错误扇出污染）。
**指回**：Coordinator Question.md「身份 vs 分类不对称」「answer-time 去重」；用户 1c #2 + Q4（明确收缩到单张 bank statement）。
**排除的替代**：按「同一对象/类似交易」笼统合并（含分类）；跨表/近似 surface 语义合并。理由：会把单一身份答复错扩到逐笔分类；近似等价依赖 Alias Log 未冻结能力。
**open boundary（本轮明确推迟）**：跨表 / 跨运行聚合与去重、近似/不同写法 surface 的等价合并、answer-time 去重的持久化注册表与扇出机制 → 全部写入缺口地图 Coordinator section（L4/seam + L2·外阻），本轮不设计。

### 决策 5 — 会计师答复后的三出口骨架 〔指向 02_logic §6 输出 / §7〕

**结论**：会计师答复后，Coordinator 三类去向：
1. **答复足够** → Coordinator 打包转下游（转 JE 所需格式；或在身份确认场景触发身份写入），**不重入 CJ**；
2. **答复不足 / 模糊 / 冲突** → 继续结构化追问，保持 pending（不得用 confidence 语言掩盖未解决）；
3. **答复暴露长期记忆 / 治理风险** → 产出 candidate 交后续 governance / review 路径，Coordinator 不自行落地。
**为什么**：对应「自动化率（足够即放行下游）+ accountant control（不足即追问）+ 审计性/治理（风险走 candidate 不僭越）」。
**指回**：CJ 02 §6「会计师确认且信息足够后，由 Coordinator 路径转 JE 所需格式或继续追问」；ER 02 §7「已确认交易不重入 CJ」；用户 Q5。
**排除的替代**：答复后重入 CJ 复判（违反 ER 不重入约束）。
**open boundary**：出口①「转 JE 格式 contract」依赖 JE Generation 未落文 → L2·外阻；是否/如何在受限边界重入上游某节点 → L4/seam，本轮不冻结。

### 决策 6 — 身份确认写入：Coordinator 只发信号，由统一写入节点执行 〔指向 02_logic §4 写入对象、§5 决策权限〕

**结论**：当 Coordinator 判定会计师给出了**明确的 Entity**、且明确当前产生的是 **stable entity** 时，Coordinator **发出「stable entity 出现」信号**，通知专门负责 Entity Log 写入的节点执行；Coordinator 本身**不裸写** Entity Log / Alias Log。
- (a)（用户拍板）该写入节点与 **Entity Resolution 新建 stable entity 所用的写入节点是同一个**——身份写入入口统一，不为 Coordinator 单设第二套。
- (b) 写入节点提取内容与 ER 路径一致：本笔交易**原始 Description**、对应 **Entity**、以及 **Entity Log 强制要求的全部字段**。
- (c) Coordinator 此时手握从系统开始至今的全部字段，写入节点只需读取所需字段并写入 **Entity Log + Alias Log**。
- 守 seam：Coordinator 侧只声明「存什么（stable entity 本体 + 最小创建 provenance + alias surface）+ 谁有权威（会计师明确确认即 authority，无需 governance approval）」；**实际写入执行者、Entity Log↔Alias Log 同步机制 = L4/seam**。
- 边界：只确认 identity，**不隐含**分类结果 / Rule / automation_policy / Case Log 写入；会计师只给分类、未明确确认身份时，不创建 stable entity、不写 Entity Log。

**为什么**：统一写入入口避免两条 stable entity 创建路径产生不一致；锚定「记忆复用」（确认即沉淀可复用身份 + alias）与「审计性」（会计师确认是可追溯 authority）。
**指回**：Entity Log §3「Accountant explicit identity confirmation path」、§6 第二条 mutation path、§2 reader 限制；Coordinator Question.md「Coordinator 路径发出 entity 确认信号，触发 Entity Log 写入（与 Alias Log 同步）」；用户 Q6 拍板（统一写入节点 + 提取字段口径）。
**排除的替代**：Coordinator 直接 mutate Entity Log（违反 §2 reader 限制 + 运行/记忆 seam）；为 Coordinator 单设独立写入节点（与 ER 入口分裂，易不一致）。
**open boundary**：写入执行者 / 调用方式 / Entity Log↔Alias Log 同步写入 exact 机制 → L4/seam；最小创建 provenance、强制字段 exact schema → L3（与 Entity Log M3 对齐）。

### 决策 8 — candidate 输出权限收窄 〔指向 02_logic §4 只能提 candidate / §5〕

**结论**：Coordinator 的 durable 影响只有两类：
1. **直接触发写入（唯一）**：决策 6 的「身份确认 → 触发 Entity Log(+Alias) 写入」（无需 governance approval）。
2. **candidate（交后续，不自行落地）**：会计师答复暴露的 automation policy / rule / case-derived risk / profile 等长期风险，Coordinator **只能作为 candidate 交后续 governance / review**；policy 升级 / 放宽必须 governance approval。
**为什么**：把唯一无需审批的直接写入限定在「身份确认」，其余长期 authority 变更一律走治理——锚定「会计师控制权 + 审计性」，防止单个 runtime 节点僭越治理。
**指回**：Entity Log §1/§4/§5、Coordinator Question.md「automation policy / governance 由人类给出」；用户 Q8 确认。
**排除的替代**：让 Coordinator 直接落地 policy/rule 变更。理由：违反 governance approval 锁。
**open boundary**：各 candidate 的 exact 类型/schema 与消费方（Governance Review / Review / Case Memory Update 等多为圈外）→ L2·外阻 + L3。

### 决策 10 — LLM vs deterministic code 分工 〔指向 02_logic §5〕

**结论**（用户 Q10 确认）：
- **Deterministic code 决定**：是否被触发；组装 Pending 上下文；按三出口（决策 5）路由；执行 hard block（不可经提问解除）；防止 accountant 答复被直接写入业务记忆。
- **LLM 判断**：把 CJ 卡点转成会计师可读问题；合并同 entity 的身份提问（决策 4）；解释会计师回答是否补足卡点；判断信息是否足以生成 JE。
- **LLM 不能**：做会计分类判断；越过 hard block；把模糊回答当明确确认；自行落地任何 durable 变更。
**为什么**：置信度与许可分离——LLM 负责语言/沟通，code 负责许可与控制流，锚定「自动化率 + 控制权 + 审计性」。
**指回**：CJ 02 §5「置信度与许可分离」；Coordinator Question.md「不把模糊回答包装成已确认身份」；用户 Q10。

### 决策 7 — 主动 entity-first 引导提问 + 拿不到 entity 的归处 〔指向 01 §1/§2、02_logic §6/§7/§8〕

**结论**：Coordinator 对 Pending 交易采取**主动 entity-first 引导提问**，把"确认交易对方身份"当作首要目标，因为记录 stable entity 是系统自动化率的复利资产。
- **(0) 适用条件与全谱覆盖（F1，用户确认补入）**：Coordinator 处理 **CJ Pending 的全谱**——不仅身份缺口，也含 HST/GST 不确定、COA 分类歧义、多合理候选、证据指向非公司支出需会计师立场、hard block、COA 校验重判失败（CJ 02 §6–§9）。本决策的 **entity-first 仅在该 Pending 存在身份缺口（entity unknown）时触发**；若实体已是已知 stable entity、卡点纯为分类/税务，则**跳过身份提问，直接走决策1/5/13 的"带选项分类问题"通用路径**，不强套 entity-first。即：entity-first 是**条件分支**，不是所有 Pending 的统一开场。
- (a) **主动询问（仅身份缺口分支）**：存在身份缺口时，第一次向会计师提问就主动问"这笔交易的对方（entity）是谁"。
- (b)/(c) **引导式确认（Q11 已定稿）**：拿到 entity 后，Coordinator 用会计知识 + 当前交易辅助证据，把分类问题组织成**带选项的 pending 问题**让会计师选择/确认。Coordinator 不自下结论；**会计师同意是结果写入的唯一条件**；不重入 CJ。
- **会计师只给分类、未给对方**时：先发一次**有界追问**索取 entity，并说明"记录 entity 有助于未来自动识别"。
- **防虚构护栏**：追问**止步于会计师真实的"不知道/查不到/不需要"**，问一次即接受；绝不把模糊回答包装成已确认 entity（Coordinator Question.md）。反复追问伤害自动化与效率。
- **拿到 entity 且确认为 stable** → 走决策 6 触发 Entity Log(+Alias) 写入。
- **拿不到 entity（不知道 / 查不到 / 不需要）** → 这笔**只获得会计分类结果**，**只进 Transaction Log**，不写 Entity / Alias / Case 等长期记忆，不阻塞、直接完成。会计师"不需要(原情况 X) vs 不知道(原情况 Y)"的原因记入 Intervention Log 供日后学习，但**不改变路由**（归处相同）。

**为什么（锚定产品目标）**：系统核心价值 = 高准确率 + 高自动化率完成 bookkeeping；entity 确认是自愈复利的自动化杠杆（下次 ER exact-match 命中、不再 Pending）。锚定「自动化率 + 记忆复用」，同时用防虚构护栏守「审计性 + 控制权」。
**指回**：用户本轮 1c #1/#2/#3/#4 拍板 + F1 确认（全谱覆盖 + entity-first 条件触发）；CJ 02 §6–§9（Pending 触发面全谱）；ER 02 §7 Problem 2/4（推断性建议由 CJ 附带、Coordinator 呈现；已确认交易不重入 CJ）；Case Log §7（entity 未解析即完成分类只 finalize 到 Transaction Log）；Coordinator Question.md（情况 X/Y、不把模糊回答包装成身份）。
**排除的替代**：被动等会计师给信息（错失 entity 资产）；反复逼问 entity（伤效率、诱导虚构）；拿不到 entity 时仍强写 Case/Entity（污染记忆）；**对已知 stable entity 的纯分类 Pending 仍强套 entity-first 开场（F1 排除）**。
**open boundary**：提问 exact 话术 / 追问次数上限 / 提问顺序编排 → L3/L4；Q11 决策 1 扩展边界待确认。

### 决策 9 — Intervention Log：确认存在，Coordinator 写入交互留痕 〔指向 02_logic §4 写入、§10 trace〕

**结论**（用户拍板）：**Intervention Log 确定存在**。Coordinator 把每次与会计师的交互（提问、回答、确认、纠正、追问、X/Y 原因）写入 Intervention Log。
- **用途（Q12 副议收窄，已确认）**：**有且只有一个——供系统设计者离线改进系统**（修缺陷、提性能、更智能更自动化；人类会计师交互痕迹是宝贵学习材料）。**不承担任何运行期辅助判断，无任何节点读取它做判断**（含 Coordinator 自身，见决策 11）。
- **结果审计不挂在 Intervention Log 上**：交互的结果审计（哪个 stable entity 因哪次确认而建、哪笔因 accountant 选了哪个选项而分类）落 **Transaction Log（完整 processing path / audit trail）+ Entity Log 创建 provenance + Case Log `confirmed_by`**——避免「审计性」依赖一个圈外未冻结 schema 的 log。详见同级 `intervention-log-question.md`。
- **边界**：交互留痕**本身不构成 authority**——不直接成为 Entity / Case / Rule authority；要转成 durable 记忆/规则/policy，仍须经治理 / review / case memory 等正规路径（Coordinator Question.md「交互本身不构成 Entity Log / Case Log / Rule Log authority」）。
- 守 seam：Coordinator 声明"写入交互留痕到 Intervention Log + 谁有权威（accountant 答复是交互事实，但非业务 authority）"；**Intervention Log 自身的正式 schema / 完整 contract / 与 Evidence Log·Review·Governance 的边界仍属圈外（未正式审计）→ L2·外阻**，Coordinator 只声明最小可写 record 语义。

**为什么**：保「审计性」（结果审计由 Transaction Log / Entity Log provenance / Case Log `confirmed_by` 承担，不依赖圈外 log）+「系统改进 / correction learning」（交互痕迹离线喂未来系统升级），同时用"留痕≠authority"守不让一次交互污染 durable memory。
**指回**：用户 Q9 拍板（Intervention Log 确定存在 + 学习价值）；用户 Q12 副议（用途收窄 + 结果审计落点，已确认）；Coordinator Question.md（交互不构成 authority）；`intervention-log-question.md`。
**排除的替代**：不持有任何 durable 交互记录（丢失审计 + 学习材料）；让交互记录直接成为业务 authority（违反 authority 边界）。
**open boundary**：Intervention Log 正式 spec（schema、writer、与其他 log 边界、学习材料如何被后续节点消费）→ L2·外阻 + L3/L4，待 Intervention Log 单独审计。

### 决策 11 — 读取对象：无独立读取面，全部上下文随 CJ Pending 在手 〔指向 01 接缝 1/3、02 读取〕

**结论**（用户 Q12 拍板）：Coordinator **不独立读取任何 store**。它的全部上下文 = **CJ Pending handoff 这一单一在手来源**，其中已携带：
- **Entity Log 身份上下文**（identity 背景 / 阻塞原因 / 推断候选）——由 CJ 在构造 Pending 时读取 Entity Log（CJ 本就是 Entity Log §2 reader）并随 Pending 投影传入，对 Coordinator 是**在手字段**，不另开读取面。
  - **注（Q14/决策13 收敛后）**：Entity Log §1 的 `identity risk flags`（疑似同名 / 疑似 merge / identity conflict）**不属此处**——它们与当前交易处理无关，服务后续审核 / 治理 Agent，**Coordinator 与当前会计师都不消费**（用户 Q14 拍板）。故 CJ Pending **无需为 Coordinator 携带 risk flags**。
- **Case Log 先例摘要**（仅 stable entity）、阻塞原因、已执行工作及结果、推断性建议（CJ 02 §6 Pending 最小语义）；
- **在手的 evidence-foundation 全链路字段**（本笔原始 Description / 交易字段，沿 runtime pipeline 在手）。
- **authority**：以上全部为**只读在手 runtime context，不当业务 authority**；**accountant 答复不得由 Coordinator 直接回写任何 store**（Entity Log §2 限制；与决策 6「只发信号、统一写入节点执行」一致）。
- **Intervention Log 不读**：它对 Coordinator 是写入对象（决策 9），不是判断输入；唯一作用是供系统设计者**离线改进系统**，不承担任何运行期辅助判断（用户 Q12 拍板；详见同级新建 `intervention-log-question.md`）。

**A 对齐说明**：Entity Log §2 把「Coordinator / Pending Node」列为 direct reader。本决策判定该读取在 **Pending 构造时（CJ 侧）完成并投影进 Pending**，Coordinator 本轮**不直接读 Entity Log**，使 Coordinator 契约面收敛为单一 surface（输入与上下文同源 = CJ Pending）。这是对 §2 reader 行的口径**收敛**，非推翻（CJ 本就是 §2 reader；"Pending Node" 的读取由 Pending 构造路径承担）。

**为什么**：单一在手 surface 使 Coordinator 契约面最小、最可审计，避免「输入单源却另开多个 store 读取面」的隐性扩面；把 Entity Log 读取留在已是 reader 的 CJ 侧，不为 Coordinator 重复开读取权。锚定「审计性 + 干净契约面」。
**指回**：用户 Q12 拍板；CJ 02 §6 Pending 最小语义；Entity Log §2 reader（含「accountant answer 不能由本节点直接写入」限制）；决策 6（写入只发信号）。
**排除的替代**：Coordinator 直接读 Entity Log 取「提问时刻鲜活」risk flags（审查方原议，已撤）——用户判定 Pending 既已携带 CJ 投影的 identity 背景即充分，且 risk flags 根本不归 Coordinator 消费（决策13），契约面更干净。
**open boundary**：Intervention Log 正式 spec → 决策 9 + `intervention-log-question.md`（L2·外阻）。

### 决策 12 — 与 interaction_agent 划界：不同阶段不同节点，人机呈现留编排层 〔指向 interaction_agent_question.md〕

**结论**（用户 Q13 拍板）：Coordinator 与 interaction_agent 是**不同节点、不同阶段**：
- **interaction_agent** = 系统**开始处理 Bank Statement 之前**（材料去重之后）与会计师对话，engagement 级 / 跨批：问目标 + 覆盖完整性核对（漏月 / 漏账户 / 多发重复）。
- **Coordinator** = 系统**处理交易过程中**与会计师对话，batch 级：问已收交易的逐笔身份 / 分类疑点。
- 二者在**不同阶段**与会计师交流，A 明示「interaction_agent 不是 Coordinator」。
- **人机对话的最终呈现形态不内化进 Coordinator**：是「多 Agent 群聊式」还是「单一前台 Agent 背后多 Agent 协管」属后期交互设计 / 编排层 open boundary（L4），本轮不操心、不塞进 Coordinator 内部逻辑（同决策 2，不让 Coordinator 持有人机对话编排权）。

**为什么**：不同阶段天然分工，不必担心混淆；呈现合并留编排层既避免 Coordinator 越权成 agent，又避免过早冻结 UX（两种设计方案均可成立，正说明不该现在冻结）。
**指回**：用户 Q13 拍板；`interaction_agent_question.md` §边界「不是 Coordinator」、§当前未决「与 Coordinator 接口未冻结」。
**排除的替代**：把人机呈现 / 对话编排塞进 Coordinator 内部逻辑。
**open boundary**：人类对话通道归属 + 多节点提问的合并 / 排序 / 呈现 = 编排 / 呈现层 L4；interaction_agent 本身圈外未正式设计，更细接口属圈外，不拿 B 填。

### 决策 13 — 会计师答复处理：用会计知识独立判断"能否构建最终分类"，身份冲突归治理 〔Q14，收敛改写〕

**结论**（用户 Q14 拍板，收敛版；本轮明确不过度预设边缘情况）：
- Coordinator **不无脑遵从会计师回复**。它**用会计知识 + 系统给出的上下文，独立判断"能不能据此构建出最终会计分类"**（信息是否完整 / 可落地）。
  - **与决策1/Q11 对齐**：此处"独立判断"是**判断信息够不够、能不能落地**，**不是 Coordinator 自己拍板分类内容**；最终分类仍以**会计师确认为唯一写入条件**。够 → 构建 / 放行下游；不够 → 追问保持 pending（决策5 出口②）。
- **身份冲突 / 风险不归 Coordinator**：即便会计师指出的 entity 与 Entity Log 已有 entity 可能有风险或"打架"（疑似同名 / 疑似 merge / identity conflict），**也由治理层处置，不在 Coordinator 这一环**。Coordinator 按会计师确认正常推进（决策6 发身份写入信号、构建分类）；冲突的**发现与解决（merge/split）由会计师人发起经 Human Review 处置、Entity Log 记录其变更语义独立承担**（原 `merge_split_candidate` 候选字段 → Post-Batch Lint / Governance Review 路径已删，见 Decisions D1）。
- 本轮**不预设、不过度担心边缘情况**（证据造假、冒名等极端身份安全情形的运行期处置）。

**为什么**：① 守住决策1——Coordinator 判断"够不够落地"而非自己分类，会计师确认才写；② 把身份冲突解决放回 owner（治理 / 会计师人发起 merge / split 经 Human Review、Entity Log 记录变更语义），不让 Coordinator 变成第二个身份审计员 / 裁判（第一性原理：不重复别人职责）；③ 与 Entity Log §6 一致——accountant 明确确认即创建 stable entity、无需治理审批，冲突由后续治理处置。
**指回**：用户 Q14 拍板；决策1/Q11（不自下分类结论、会计师确认为唯一写入条件）；Entity Log §3/§6（accountant confirmation 创建 entity 无需治理审批；merge / split 归人发起 Human Review，原 `merge_split_candidate` 候选字段已删，见 Decisions D1）、§8（accountant 明确纠正进 review/governance）；决策6（身份写入只发信号）。
**排除的替代**：让 Coordinator 在运行期侦测 / 裁决身份冲突、对身份风险硬阻断（上一版 Q14 四出口表）——用户判定越权且过度预设边缘情况，冲突解决归治理。
**open boundary**：① 极端身份安全 / 证据真实性情形的运行期处置 → 治理 / 后续，本轮不设计；② 身份冲突 merge/split 解决机制 = Entity Log §4/§6/§7 治理路径（圈外）；③ **已连带清理（用户 Q14 确认）**：身份风险 flag 不被 Coordinator 消费，决策11 的"CJ Pending 强制 forward 身份风险 flag"义务已撤销，缺口地图 L3 已同步清理；决策11 核心（只消费 Pending、不独立读）不变。

### 决策 14 — 学习闭环：会计师确认的 stable-linked finalized 交易产出 Case Log 学习记录 〔Q15〕

**结论**（用户 Q15 拍板）：会计师经 Coordinator 确认身份+分类、已 finalized 且关联 stable entity 的交易，**产出 Case Log 学习记录**（`confirmed_by=accountant`，凭 `transaction_log_ref` finalization proof），让下次同 entity 类似交易有先例可复用——服务记忆复用 / 自动化率 / correction learning。
- **边界1（与决策7一致）**：只有**拿到 stable entity** 的交易进 Case Log（按 `entity_id` 索引）；**拿不到 entity 的（决策7 情况 X/Y）只 finalize 到 Transaction Log，不进 Case Log、不创建 Entity Log**。
- **边界2（三份独立记忆，不混）**：决策6 写 Entity Log（身份）、决策9 写 Intervention Log（交互痕迹·离线学习）、决策14 写 Case Log（可复用先例）——用途不同、分开。
- **守 seam**：Coordinator 只声明「存什么（confirmed_by=accountant 的可复用先例 + accounting outcome 快照 + evidence condition + use_level）+ 谁有权威（会计师确认 + finalization proof）」；**exact writer / 与 Transaction Log·Entity Log 的统一 finalization 顺序 = L4/seam**（同决策6）。
- **Coordinator 不自定分类内容**：写入的 accounting outcome 是**会计师确认的结果**，非 Coordinator 判断（守决策1）。

**为什么**：会计师教过一次即沉淀为先例，下次 ER exact-match + Case 先例复用、不再 Pending——锚定核心产品目标（记忆复用 + 自动化率 + correction learning）；用 finalization proof + stable entity 门槛守审计性、防身份不稳定时产强学习记忆。
**指回**：用户 Q15 拍板；Case Log §3 写入者「Review/Coordinator/accountant interaction finalization path」、§1 写入资格（stable-linked finalized + `confirmed_by`）、§6 mutation path（需 finalization proof）；决策7（拿不到 entity 只进 Transaction Log）；决策1（不自定分类）。
**排除的替代**：把未关联 stable entity 的交易也写 Case Log（违反 entity-index + 决策7）；Coordinator 自定 accounting outcome（违反决策1）。
**open boundary**：Case Log exact writer / trigger order / 多 log 统一 finalization、`use_level`·`confirmed_by` enum = L3/L4（Case Log Open Boundaries，圈外）。

### 决策 15 — 触发 / 不触发边界 〔Q16〕

**结论**（用户 Q16 拍板）：
- **触发**：一张 Bank Statement 跑完、其 CJ Pending 集合就绪时触发（批级，决策2/3）。
- **不触发**：① 已 finalize / 非 pending / 已解决的交易——**不重入**（ER §7「已确认交易不重入 CJ/ER」）；② onboarding 期不触发（onboarding / interaction_agent 的职责；决策2 已界定 Coordinator 为 runtime 交互通道、onboarding 期交互不归本节点）。

**为什么**：批级触发是聚合提问前提（决策4）；不重入守 ER 约束、防二次分类污染；onboarding 排除守节点边界（决策12 划界）。
**指回**：用户 Q16 拍板；决策2/3、ER §7（不重入）；决策12（与 interaction_agent / onboarding 划界）。
**排除的替代**：逐笔触发（无法聚合，决策2 已排除）；在已 finalize 交易上重入（违反 ER §7）。
**open boundary**：多表提交串行 / 并行排序、批起始记忆快照 = L4/seam（见缺口地图）。

---

## 三、分类备案（未冻结项 → 写入缺口地图 Coordinator section）

**L3（字段/enum/阈值，延后）**
- Pending handoff / `pending_request_context` exact schema、pending 子类型 enum（与 CJ L3 对齐）。CJ Pending 携带的 identity 背景 exact 字段随 CJ Pending L3 定（注：身份 risk flags 不在 Coordinator 所需之列，决策13）。
- Coordinator question 模板 exact 字段、回答结构、追问边界字段。
- 情况 X / 情况 Y 在 schema 中的落点字段。
- 「转 JE 所需格式」exact 字段 contract（依赖 JE Generation）。

**L4 / seam（机制，延后）**
- Coordinator 触发 Entity Log + Alias Log 同步写入的执行者/调用方式/顺序（语义已锁，机制未冻结）。
- 多表提交串行/并行排序、批起始记忆快照。
- 会计师答复后是否/如何受限重入上游、重跑边界。
- 同表内身份提问聚合 + 扇出的执行机制；跨表/跨运行聚合与去重、近似 surface 等价合并、去重注册表（本轮明确推迟）。
- 情况 Y 身份缺口的 triage / 复查机制（用户暂缓）。

**L2·外阻（圈外依赖，挂 open boundary，绝不拿 B 填）**
- JE Generation Node（转 JE 格式）、Transaction Log（情况 Y 缺口落点 / finalization）、Intervention Log（durable interaction record 归属）、Governance / Governance Review、Profile / Structural Match、Knowledge Summary、Review Node、Case Memory Update Node、interaction_agent（划界）、Onboarding。

---

## 四、自检与判定（Stage 1-2 清单）

按 `workflow_node_spec_template_rules.md` §10 自检，当前已可回答 / 仍开放：

| # | 问题 | 状态 |
|---|---|---|
| 1 为什么存在 | CJ Pending 的 runtime 接手 + 会计师交互桥 | ✅ 决策1 |
| 2 不能删/并/内联 | 否则 Pending 无归处、会计分类权被僭越 | ✅ 决策1 |
| 3 唯一核心职责 | 拆解CJ输入+结构化提问+判断完整性+打包，不做会计判断 | ✅ 决策1 |
| 4 上游依赖 | CJ 的 Pending（唯一输入） | ✅ 决策3 |
| 5 下游影响 | JE Generation / Entity Log+Alias / Transaction Log / Case Log（学习记录）/ governance candidate | 🟡 边界已声明，多为圈外 |
| 6 读取哪些 store | 无独立读取面；全部上下文随 CJ Pending 在手；Intervention Log 不读 | ✅ 决策11 |
| 7 直接写什么 | 身份确认→触发 Entity Log(+Alias) 写入信号（决策6）；学习记录→Case Log（决策14）；交互留痕→Intervention Log（决策9）；均守 seam 只发信号/声明语义 | ✅ 决策6/9/14 |
| 8 只能提什么 candidate | automation policy/rule/case-derived risk 等只作 governance/review candidate，不自落地 | ✅ 决策8 |
| 9 绝不能写什么 | 不裸写 Entity/Alias/Case/Rule/policy；不自定分类；交互留痕不当 authority | ✅ 决策6/8/9/14 |
| 10 deterministic code 决定 | 是否触发/组装上下文/三出口路由/hard block/防答复直写记忆 | ✅ 决策10 |
| 11 LLM 可判断 | 卡点转人话/合并同entity身份提问/解释答复是否补足/判断是否足以生成JE；不做会计分类 | ✅ 决策10 |
| 12 accountant 必须决定 | 身份、最终会计处理（已锁） | ✅ |
| 13 governance 必须批准 | 长期 authority 变化（已锁，多圈外） | 🟡 圈外 |
| 14 证据不足 | 继续追问保持 pending（决策5出口②） | ✅ |
| 15 歧义 | 不猜，追问/保持 pending；不把模糊回答包装成确认 | ✅ 决策5/13 |
| 16 source conflict | 答复够不够独立判断；身份冲突归治理（merge/split candidate），Coordinator 不自裁 | ✅ 决策13 |
| 17 留哪些 trace | 交互留痕→Intervention Log（决策9）；学习先例→Case Log（决策14）；身份创建 provenance→Entity Log（决策6）；结果审计→Transaction Log | ✅ 决策9/14/6 |
| 18 哪些 trace 不成 authority | 交互留痕不构成 Entity/Case/Rule authority（决策9） | ✅ 决策9 |
| 19 契约面 + 每输出 consumer | 出口/写入对象 consumer 已列；exact schema 待 L3 | 🟡 schema 待 L3 |
| 20 守运行/记忆 seam | 是（写入只声明语义，机制留 L4/seam） | ✅ |
| 21 哪些未冻结 | 见第三节 | ✅ |
| 22 可否进 Stage 3 | 否——L1-L2 语义骨架已齐，但卡圈外依赖（JE Gen/Transaction Log/Intervention Log 未落文）+ 全部 schema/机制属 L3/L4 | ✅ 判定 |

**判定**：Stage 1（功能意图）已成形；Stage 2（逻辑与边界）的 22 项自检**语义层已全部可回答**（Q12–Q16 落为决策11–15）。**仍不能进 Stage 3**，但原因已从"L2 待讨论"变为**纯外部/分层阻塞**：① 三个圈外下游（JE Generation、Transaction Log、Intervention Log）未落正式文，契约面无法字段级冻结；② 余下全部为 L3（schema/enum/阈值）与 L4/seam（写入机制/聚合扇出/多表排序）——按纪律本就延后。**结论：L1-L2 本轮可收口；进 Stage 3 的前置 = 圈外依赖落文 + L3/L4 单独推进。** 详见下节第一性原理复查。

---

## 五、第一性原理复查（Q12–Q16 已落盘后）

> Q 系列已全部转为决策：Q12→决策11、Q13→决策12、Q14→决策13、Q15→决策14、Q16→决策15；决策9 已按 Q12 副议收窄用途；补充① 已撤销清理。本节用第一性原理复查"Coordinator 这个 L1/L2 还有没有真问题没讨论"，区分**仍需用户拍板**与**已可定位归档**两类。

### A. 仍需用户拍板的 L1/L2 项

- **F1 — 非身份类 Pending 的处理覆盖** → ✅ **已确认补入决策7 (0)**（全谱覆盖 + entity-first 条件触发）。本节无剩余待拍板项。

### B. 已可定位归档（无需讨论，挂层级即可）

- Coordinator ↔ Review Node 划界：二者同列 Case Log §3 finalization path；Review 圈外未落文 → open boundary（L2·外阻），待 Review 审计时划清。
- hard block 交易的最终归处（不可经提问解除后去哪）：取决于 CJ 对 hard block 的定义 + 编排层 → L4/编排（圈外）。
- 会计师久不答复 / 无回应的 Pending 处置：跨运行在途窗口 + alias 自愈已分析（Coordinator Question.md）→ L4/编排。
- 会计师确认分类转 JE 前的客户 COA 校验（CJ→JE seam，JE 未落文）；Pending 批级排序 / 展示 / UI（L4）。

### C. 进 Stage 3 的前置（已在 §四判定）

L1-L2 语义骨架本轮可收口；进 Stage 3 卡两类外部/分层阻塞：① 圈外下游落文（JE Generation、Transaction Log、Intervention Log）；② L3（schema/enum/阈值）与 L4/seam（写入机制、聚合扇出、多表排序）按纪律延后。
