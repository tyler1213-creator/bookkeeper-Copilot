# 任务：把已审定的 Human Review Node L2 提案落地到 BK_Copilot 正式文档（建模式，仅此一个对象）

## 你的角色
你是实现者，不是审查者，也不是设计者。
唯一任务：把【已经过用户审定的】Human Review Node 提案 + 下方用户澄清，落到 BK_Copilot 的对应正式文档上。
不重新设计、不重新审计、不质疑已锁结论、不扩大范围、不碰其它对象。

—— 本次 = **建模式**：目标文件不存在，严格按 `BK_Copilot/node:layer spec template/workflow_node_spec_template_rules.md` 的结构从零创建 `00_index` / `01_functional_intent` / `02_logic_and_boundaries`；只创建已成熟到值得写下来的文件（本轮只到 Stage 1-2，**不**创建 03 及以后），不为结构完整造空文档。

## 范围（严格）
本次允许创建的对象文件，仅限：
- `BK_Copilot/workflow_nodes/human_review_node/00_index.md`
- `BK_Copilot/workflow_nodes/human_review_node/01_functional_intent.md`
- `BK_Copilot/workflow_nodes/human_review_node/02_logic_and_boundaries.md`

落地完成后，按本指令最后一节同步更新：
- `当前任务状态.md`
- `缺口地图.md`

除上述文件外，绝不触碰其它 workflow node、memory layer、未列出的治理文档、`new system/`、`old_system_nodedesign/`。
如果驱动提案里出现「改其它对象文件」的拟改条目（例如「A 类五个 log + Coordinator 草案中旧 writer 模型 / 旧 Review Node 引用的写回」），**不要执行**——那属于另一个对象，由单独 prompt 处理；本 prompt 内对该结论只在本节点自己范围的文件里声明，并把它登记为本节点的 open boundary（见 02 §12 「A 类残留待写回」）。

## 必读（按此顺序，且只把它们当依据）
1. 主驱动：`L2_proposals/HumanReviewNode__L2提案.md` —— 本次创建的权威来源、最新唯一来源，逐条决策落地。
   （注：旧 `独立question文档/Human_Review_Node_question.md` 已降为历史背景，**不**作为契约来源，见用户澄清 ①。）
2. 模板（必须遵守其结构与「禁止写法」与「Stage 1-2 完成清单 22 问」）：`BK_Copilot/node:layer spec template/workflow_node_spec_template_rules.md`
3. 结构依据（建模式）：上述模板中 00_index / 01 / 02 的章节结构与各「完成清单」必答项。
4. 锁定上游 / 不变量：`当前任务状态.md`、`缺口地图.md`（Human Review Node section + 顶部「跨层共享机制」），以及被引用的 A 类正式草案（`BK_Copilot/workflow_nodes/coordinator_node/02`、`memory_layers/{transaction_log,case_log,entity_log,rule_log}/02`）。出现冲突时以这些已锁结论为准，按权威顺序解决，不以 new/old system 为准。
`new system/` 和 `old_system_nodedesign/` 只是审计对象，没有任何权威性，不得作为依据或 baseline。

## 用户已审定的澄清（凌驾于现状文字与提案措辞之上；只存在于本指令，务必照此落地）
① **唯一来源 = 本 L2 提案**。提案/工作区里凡把设计来源指向 `Human_Review_Node_question.md` 的措辞，一律改为指回本 L2 提案 + 用户拍板；正式文档不得出现对该 question 文件的悬空依赖。
② **待建依赖一律先挂 open boundary、照常建档**。Governance Log、审核 inbox、Intervention Log、Finalization 写入机制、确定性发现 job 这些「待建 / 无正式 spec / 另窗推进」的对象，全部作为 open boundary（L2·外阻）声明，**绝不拿任何材料替它们补内容**，但不因它们缺席而阻塞本节点 00/01/02 的创建。
③ **NEW-1 归 Rule 侧**：rule 升级「固定执行路径」的**定义**归 Rule 侧（Rule Log / Rule Match 治理），本节点**只触发**、不定义该路径。文档里写明本节点对纯 rule 升级仅作「确认面 + 触发器」，路径定义归属标为 open boundary（指向 Rule 侧）。
④ **系统自发发现层挂起、本节点只接人发起**：FP-5（系统自发发现错误）与「语义发现器（系统自发 merge/split）」被用户确认为**同一个问题**（系统自发语义判断 → 喂给人审），暂不删除、合并挂起、另窗讨论。本节点当前**只承接会计师人发起**的 merge/split 与纠错；系统自发那部分标为 open boundary（L2·外阻），不在本节点内实现入口。
⑤ **C2 对话前端留编排层**：本节点成品 Chatbot 是否与 Coordinator / interaction_agent 共用同一对话前端 = 编排 / 呈现层问题，标为 open boundary，本轮不决、不在本节点内闭环。

## 不可违反的纪律
- 不以任何形式复活 candidate entity / candidate identity（含第三态、把 ambiguous/unresolved/conflict 当独立身份状态保留）。
  注意：文档里合法的「非身份候选」——`merge_split_candidate`、`automation_policy_candidate`、`entity_risk_candidate`、rule 升级候选——是 Alias/合并拆分/治理候选，保留；但它们不是身份状态，要与 stable/unknown 身份分类清楚分开。
- 不在 L2 冻结字段级 schema：`confirmed_by` enum、Intervention ID / correction record schema、Change List（P_llm / P_engine）schema、凭证 exact 形态、read-back 模板 / 字段、entity 税务状态字段、source 字段形态，全部留作 open boundary（L3 或 L4/seam），不写死。
- 守运行 / 记忆 seam（本节点是 workflow node）：只声明「要持久化什么 + 谁有权威认定它有效」，**不**定「谁来写、怎么写、什么顺序写」。Finalization 写入机制（跨 log 原子 / 顺序 / 幂等 / 凭证校验）是共享落盘地板，本节点**只调用、不重定义**；写明本节点不裸写任何 log。
- 契约面 / Consumer：02 的输出类别表，每个输出必须写明 Consumer（谁消费）与下游含义。consumer 是「待建」对象（Governance Log / 审核 inbox / Intervention Log）时，照实写出该 consumer 名 + 标注其为待建 open boundary，**不要**因此留空或省略 consumer。
- 不触发模板「禁止写法」（「LLM 自行判断」「高置信度自动通过 / 自动沉淀」「后续实现再定」「参考旧系统」「写入日志但不说哪个 store/字段/authority」「生成候选但不说谁消费/谁批准/能否持久化」）。本节点三铁律之一即「LLM 永不写字节」，务必体现。

## 具体改动清单（建模式：逐条指到 文件:章节 → 写什么；来源指回提案决策）

### `00_index.md`
- **文档状态**：Current standard status = draft；Covered stages 勾选 Stage 1 + Stage 2，其余不勾。Last reviewed = 2026-06-22；Owner/reviewer 留待填。
- **当前标准文件**：列 `01_functional_intent.md`、`02_logic_and_boundaries.md`（03 尚未创建）。
- **当前未冻结边界**：摘 `缺口地图.md` Human Review Node section 的 L3 / L4-seam / L2·外阻要点（confirmed_by enum、Change List/凭证/read-back schema、Change List Engine 依赖图声明形态、Finalization 调用契约、并发待确认项排序、N 笔批量 append 顺序、"rule 被推翻一次"信号落点；Intervention Log / Governance Log（待建）/ 审核 inbox（待建）/ Finalization / 确定性发现 / 系统自发发现层（挂起）/ NEW-1 归 Rule 侧 / C2 对话前端）。
- **进入下一阶段（Stage 3）前必须解决**：① 圈外依赖落文（Intervention Log、Governance Log、审核 inbox、Finalization、确定性发现）；② L3 字段定稿（confirmed_by / correction record / Change List / 凭证 / read-back / entity 税务字段 / source 字段）。

### `01_functional_intent.md`
- **§1 为什么必须存在**（决策 1）：独立于逐笔交易流水线、由会计师主动发起、面向「已 finalized 事实」的事后纠错 + 系统↔会计师「规则变化 / 错误治理」沟通桥梁。删除/内联会失去：会计师事后唯一干净纠错入口、人类权威天然在场、审计性。不能内联进 Coordinator（Coordinator 只处理 running、finalized 后不得触发，A 已锁：`coordinator_node/02 §1`）。
- **§2 核心职责**（决策 1 / 9）：按会计师指令对系统已确定事实实施修改，但**自己不是直接执行者**——备好符合各 log Finalization schema 的 Input → 调用 Finalization 落盘；同时充当「规则变化 + 错误治理 + 事后纠错」的唯一人类交互面（conductor，不 judge）。
- **§3 明确排除范围**（决策 1 / 3 / 9 / 9b + 澄清 ④）：不做 onboarding / 目标询问 / 覆盖核对（归 interaction_agent）；不处理运行期 pending（归 Coordinator）；不裸写任何 log；LLM 绝不写字节、不自批 / 不放行、不当场裁决某 rule「坏了」并执行降级、不把 scope 从单笔脑补成永久、不重判确定性检查；不接「系统自发」merge/split 与「系统自发」错误发现（挂起，只接人发起）。
- **§4 Workflow 位置**（依赖节 + 决策 13）：上游 = 非 pipeline handoff——(A) 会计师主动发起；(B) 审核 inbox 候选（确定性发现 job 投，当前 = rule 升级候选，本节点 pull）；(C) 未来 LLM 审查节点（当前不存在）。下游 = 调 Finalization 落盘各 log（Transaction / Intervention / Case / Entity / Rule / Governance Log）。位置不固定（批后 / 导入前 / 周审），功能要求固定。
- **§5 对核心产品目标的贡献**：勾选 accountant control、审计性、accountant correction learning（并简述：给会计师事后唯一干净入口 + 纠错不污染 durable memory）。
- **§6 已知约束**（决策 5 三铁律）：① LLM 永不写字节；② 写入前强制 read-back（每次都念，念对账后的复述版）；③ scope 绝不静默升级（单笔 vs 永久是两次单独显式确认）。
- **§7 未决定问题**：系统自发发现层（FP-5 + 语义发现器）是否必要（挂起、另窗）；NEW-1 rule 升级固定路径定义归属（倾向 Rule 侧）；C2 对话前端归属。

### `02_logic_and_boundaries.md`
- **§1 触发条件**（决策 1 / 2）：会计师主动发起（典型：多张 BS 处理完导入 QuickBooks 前整批审 / 周审）；对象 = 已 finalized 的事实。**不得**在交易仍 running / pending / 卡住 / terminal / blocked 时触发（那些归 Coordinator）。
- **§2 上游前置条件**：交易已 finalized（已有最终 outcome）。前置缺失（交易未 finalized）→ 不归本节点，留 Coordinator。
- **§3 读取对象**（依赖·读取面）：Transaction Log（final outcome / COA·HST-GST / confirmed_by / processing path，只读、非 authority producer）、Case Log（precedent / exception context / use_level / confirmed_by，不把案例变 rule/entity/governance authority）、Entity Log（active state / authority refs / candidate context，不批准 durable entity mutation）、Rule Log（executable rule 上下文，判某笔是否来自 active rule）。删掉模板表中不适用的 source（Evidence Log / Profile / Knowledge Summary 等若本节点不读则删行）。
- **§4 写入对象**：
  - 直接执行的 durable write = **无**（本节点不裸写，写明这一点）。
  - 交 Finalization 持久化（只声明「存什么 + 谁有权威」，不定机制）：Transaction Log（correction append：谁改 + 改后最终分类，append-only）、Intervention Log（原因 + Intervention ID，**待建 spec**）、Case Log（先例 superseded / 更新；只作废 `confirmed_by=system`，不动 `confirmed_by=accountant`，绝不删）、Entity / Rule Log（扩张型变更：凭证→approved mutation）、Governance Log（每笔扩张型变更审计，**待建**）。authority 来源 = 会计师 read-back 签字（+ Finalization 凭证死代码强制）。
  - 只能提出 candidate：rule 升级候选（来自审核 inbox，本节点作确认面 + 触发器，触发 Rule 侧固定路径——NEW-1 归 Rule 侧）。
  - 绝不能写入或修改：不裸写任何 log；LLM 不产生任何 durable 写入。
- **§5 决策权限**（决策 10 表，逐行落）：Deterministic code（依赖图展开 P_engine / 执行图、按 confirmed_by 过滤作废范围、append/ID 链路/不变量校验、跨 rule 互斥校验 / promotion 资格判定——本节点只取结果）；Finalization 死代码（凭证校验 = 放行权、多 log 原子/顺序/幂等/append-only）；LLM·conductor（听懂纠错、结构化意图、产执行图、撒网产 P_llm、区分 rule 错/例外/范围过宽——仅提案/发信号、组织 read-back）；LLM 不能（自批/放行、当场裁决 rule 坏了并降级、scope 单笔→永久、重判确定性检查、任何 durable 写入）；Accountant 唯一判断者（read-back 最终确认 + 签字、diff 多出项真连带还是幻觉、是否一并作废人工确认先例、scope 是否升永久、扩张型变更是否通过）；Governance（写明本节点不产生 governance 批准，扩张型变更的批准 = 会计师签字这条唯一治理线，留痕入 Governance Log）。
- **§6 输出类别 / 契约面**（每个输出写 Consumer + 下游含义 + 不代表什么）：correction record（Consumer = Transaction Log；append 更正）、Intervention 记录 + ID（Consumer = Intervention Log，待建）、先例更新/supersede（Consumer = Case Log）、扩张型 mutation 备料（Consumer = Entity/Rule Log via Finalization，须凭证）、扩张型变更审计（Consumer = Governance Log，待建）、rule 升级触发（Consumer = Rule 侧固定执行路径，NEW-1）。所有输出「不代表」自身成为 authority——authority 来自会计师签字 + Finalization。
- **§7 证据不足**（决策 11）：改分类⇒重算税，税率查表优先；entity 税务状态字段缺失 → 引擎撞缺失输入 → 停下进 read-back，**不交 LLM 猜**；LLM 先用会计知识分析给选项，最终人定。
- **§8 歧义处理**（决策 3 / 4）：每笔纠错都跑 P_llm（LLM 第二意见，与 Engine 真独立产出）+ P_engine（确定性遍历）对账（diff）；一致项保留、LLM 多出项→read-back 让会计师裁决（真连带补进依赖表 / 幻觉则弃）、引擎有 LLM 漏→信引擎。本节点不为让流程继续而自动猜 winner。
- **§9 冲突处理**（决策 8 / 12）：单次纠错对「rule 坏没坏」是弱证据——本节点**不当场裁决**，只结构化这一笔 + 发「rule 被推翻一次」信号（归确定性发现 / Rule Log promotion，本身可复核）。改一条 rule 牵连历史 N 笔 → 先向会计师说明「过去定的分类规则、已产生 N 笔」，问「只改这一笔还是全部」（铁律 3，LLM 不脑补）；选「全部」则回溯纠正按层落：Case Log 直接改成新分类 `confirmed_by=accountant`（可复用记忆、非审计、不背版本历史）、Transaction Log 逐笔 append 新记录（原记录绝不删/覆盖）；rule 本身去向（废除/更新/降级）是单独一问。
- **§10 Audit / Trace 边界**：保留 Intervention ID 链路、correction append、Governance Log 留痕；这些 trace 用于 review/correction/governance/audit，**不能**成为 entity/rule/case authority 或 accountant/governance approval（approval = 会计师签字本身）。
- **§11 Legacy Constraint Translation**：本节点为 owner 新创节点、无旧系统继承约束 → 此节写「无保留旧约束」或留空，**不得**从 old/new system 搬设计。
- **§12 Open Boundaries**：把 `缺口地图.md` Human Review Node section 的全部未冻结项逐条搬入并归类（L3 / L4-seam / L2·外阻），含：confirmed_by enum、Change List/凭证/read-back/correction schema、entity 税务字段、source 字段（L3）；Change List Engine 依赖图声明形态、Finalization 调用契约/凭证校验/写入顺序/原子回滚/幂等、并发待确认项排序去重、N 笔批量 append 顺序、read-back UI 与复核视图是否同一 shell、多来源呈现、"rule 被推翻一次"信号落点（L4-seam）；Intervention Log、Governance Log（待建）、审核 inbox（待建）、Finalization / 确定性发现 job、系统自发发现层（FP-5 + 语义发现器，挂起）、NEW-1 rule 升级固定路径定义归属（归 Rule 侧）、C2 对话前端、A 类残留待写回（L2·外阻）。写明这些解决前不得进 Stage 3 / Stage 4 / 实现。

—— 建模式专项：按模板补全 00/01/02 各节；对照模板「Stage 1-2 完成清单 22 问」逐条确保可回答，不能回答的标为 open boundary，**不要猜**。覆盖阶段保持 Stage 1-2，不勾 Stage 3 及以后。

## 停止并问用户的情形
- 任一输出的 consumer / 某结论的归属无法从提案或已锁上游确定。
- 提案某条拟改与已锁上游（`当前任务状态.md` / `缺口地图.md`）冲突，且权威顺序无法解决。
- 出现需要新的产品决策、而非措辞落地的点。
遇到以上情况，停下来问，不要自行拍板或猜测。

## 落地后更新相关规则文档（务必执行）
- `当前任务状态.md`（必更）：把 Human Review Node 的 L1-L2 标为**已落正式 draft**（建模式，仅 00/01/02，覆盖 Stage 1-2）；更新「最近重要交接」对应指针（从「设计结论、未转 BK_Copilot」改为「已落 `BK_Copilot/workflow_nodes/human_review_node/`」）；保留必要交接与指针，不搬长期结论全文。
- `缺口地图.md`（必更）：把已收口 L1-L2 不再回写；保留 / 更新本节点 L3 / L4-seam / L2·外阻挂起项（注意本轮已记录的：NEW-1 归 Rule 侧、系统自发发现层合并挂起、C2 留编排层、待建依赖 open boundary）；确认正式文档已建后指针指向 `BK_Copilot/workflow_nodes/human_review_node/`。
- 其它治理文档（`系统上下文地图.md` / `审计阶段路线图.md` / `AGENTS.md` / `审计目标与原则.md` / `项目背景速读.md`）：仅当其职责范围内内容确因本次落地变化时才动；否则不碰。

更新纪律：保持克制、遵守权威顺序；规则文档不承载长期设计结论正文；不加产品功能逻辑细节；不把 new/old system 当 baseline 或权威。

## 完成后输出
- 建了哪些文件、每个文件写了哪些节、对应提案哪条决策或哪条澄清。
- 哪些点按要求保留为 open boundary（L3 / L4-seam / L2·外阻），没有越权冻结。
- 同步更新了哪些规则文档（当前任务状态 / 缺口地图 / 其它），各更新了什么；哪些规则文档判断无需改动及原因。
- 一致性自检：无 candidate identity 复活、seam 守住（不裸写、不重定义 Finalization）、契约面与每个 Consumer 已声明（含待建对象照实写名 + 标注）、未冻结字段级 schema、未触发禁止写法、覆盖阶段仍为 Stage 1-2。
- 是否触发「停止并问用户」；若有，列出问题。
