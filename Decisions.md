# Decisions

> 确定性决策记录。每条 = 做了什么 + 为什么 + 动了哪些文档。给新窗口快速对齐用，不展开论证。

## Entity Resolution（L3）

### D1 · 删除 ER 的并行候选 / 风险通道 — 2026-06-25
删除 `candidate_signal`、`merge_split_candidate`、`alias_conflict_issue`、`identity_governance_issue`、`blocking_reason`，以及作 ER **输出**的 `identity_risk_flags`。
为什么：无下游消费者（CJ 只读身份结果 + unknown reason；Human Review 从 Entity Log 读、且只接会计师人发起）；与 `unknown` reason 重复；merge/split 超出 ER 能力（单笔粒度看不到 split；merge 需被架构否决的语义相似）与职责（归 Entity Log + 会计师人发起 Human Review）。身份卡点一律经 `unknown` + reason 表达。`identity_risk_flags` 真值归 Entity Log，ER 只读。
文档：ER `00/01/02`。

### D2 · Alias 命中只认"是谁"，删"命中但不干净" — 2026-06-25
exact alias 命中 → 直接复用该 entity → `stable`，不再判该 entity 是否 active / 冲突 / merged。删 §5 的"冲突或多 entity 竞争"检测、§9 的"alias 库冲突 / 同一 surface 对多 entity → unknown"整段。
为什么：ER 只回答"是谁"；entity 干不干净是下游（Case Judgment 分类）与 Entity Log（supersession）的事，在身份这步无区别。高度相似 = 未完全命中，二者都落证据路。
文档：ER `02` §5、§9。

### D3 · §10 trace 瘦身 — 2026-06-25
ER 不设独立 trace 账。审计面 = 输出（`identity_state` / `entity_id`）+ `identity_reason`（stable）/ `unknown_reason_context`（unknown）+ `identity_evidence_refs`（外部来源作其 ref 变体）。删 `evidence_used`、`matched surface text`、`matched Alias surface text`、`Alias lookup basis`、`transaction_id`-作-trace；creation provenance 归 Entity Log；治理 / 介入 ref 折进 reason。
为什么：这些是"输出 / 落库内容 / 可重建值"的重复；ER 近乎无状态，output + reason 即审计。
文档：ER `02` §10。

### D4 · 保留两类冲突处理（非 alias 命中） — 2026-06-25
保留 §9 的"evidence vs Entity Log / Governance 冲突 → unknown"与"accountant context vs durable memory 冲突"。
为什么：它们管"认得对不对"（硬权威 > LLM 语义），不是 alias 命中的干净度，不在 D2 删除范围。
文档：ER `02` §9（未改，确认保留）。

### D5 · identity 审计材料落 Transaction Log — 2026-06-25
`identity_reason` + `identity_evidence_refs`（含外部来源 ref）是交易级审计，随交易在 finalization 时落 **Transaction Log**；ER 只产出、不写。新建实体的 creation provenance / 初始 alias 仍落 **Entity Log / Alias Log**。
为什么：身份的"为什么 + 凭什么"是交易作用域（同一 entity 各交易证据不同），Entity Log 是实体作用域、不存 per-transaction 推理（[entity_log 01:67](BK_Copilot/memory_layers/entity_log/01_memory_intent.md)）；ER 绝不写 Transaction Log（[ER 01:57](BK_Copilot/workflow_nodes/entity_resolution_node/01_functional_intent.md)），故由 finalization 落盘。"按 entity 查历史身份证据"是检索需求（Transaction Log 按 entity 建索引，L4/seam），不是复制进 Entity Log 的理由。
文档：ER `02` §10。

### D6 · ER 不预加工证据，下游读 raw — 2026-06-25
ER 不把证据预消化成共享产物给下游；Case Judgment 为会计分类**独立读 raw 证据**（经自身 evidence_refs 回 Evidence Log，与 ER 读的是同一份）。ER 的 `identity_evidence_refs` 是**指针（ref）**，非加工内容。
为什么：铁律"摘要不能替代 raw ref"（[case_judgment 02:52](BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md)、[ER 02:35](BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md)、[case_log 02:12](BK_Copilot/memory_layers/case_log/02_authority_lifecycle_and_boundaries.md)）禁止用不可追溯摘要替代 raw；让 ER 为会计预加工还会越其单一职责。重复读同一材料的成本被有意接受，换审计性 + 权威分离。"处理一次"只适用 EI 的客观交易结构（[Evidence Intake L2提案:45](L2/L2_proposals/Evidence%20Intake__L2提案.md)），不延伸到辅助证据。
文档：无改动（澄清既有不变量）；影响本轮 ER L3 = `identity_evidence_refs` 是 ref 非内容。

### D7 · ER 字段命名收敛（含 alias / raw_description / created_by / entity_id） — 2026-06-25
本轮 ER L3 锁定字段名：运行期表面文字全程 **`raw_description`**（沿用 EI 名，不漂移），无论 stable/unknown；**`alias` 不是运行期字段**，只是 stable·新建时 `raw_description` 写入 Alias Log / Entity Log 的存储记录名（alias 只配已 stable 的对象）。原 `creation_provenance` 简化为 **`created_by`**（`confirmed_by` 已被 Case Log 占用）。Entity 的输出值 = **`entity_id`**（stable 句柄，指向 Entity Log 记录，非复制本体）。`unknown_reason_context` 并入 **`identity_reason`**。
为什么："一个值一个名"——raw_description 不因状态改名；alias 作存储概念只在写 log 时出现，守"alias=身份 stable 之果"；命名能少则少。
文档：本轮 schema `L3_schema/ER_Schema/Entity_Resolution_L3_Schema.md` + `L3_schema/字段链路大表.md`（不改 ER `00/01/02`，冻结文档已与此一致）。

## Rule Match / Entity Log（automation_policy 收口）

### D8 · 删除 automation_policy，拆为 force_pending + promotion_lock 两个独立控制；系统不自发收紧 — 2026-06-26
**删除** EntityLog 的 `automation_policy`（原"entity 级自动化控制状态"单一槽）。EntityLog **新增两个互相独立、都只能会计师人发起（Human Review → Finalization 落盘）的治理级控制**：
- `force_pending`：该 entity 的每笔交易必须走人审 Pending、不许高置信度自动分类；**即使其名下存在 active rule 也不启用**（强制 Pending 压过 Rule）。由**运行期 eligibility 路由**（ER→RM seam）消费——命中即改道 Case Judgment / Pending，交易不进 Rule Match。
- `promotion_lock`：该 entity 下任何分类**永不许升级为 Rule**。由**确定性发现 job** 消费——命中即不产 `rule_promotion_candidate`（发现 job 不累积"被推翻"信号，无此锁会每次扫描重复提候选）。

**同时撤销**原"系统受控 restrictive auto-downgrade"：系统不自发设 / 收紧这两个值；凡"系统可自动收紧 automation"的表述全部删除 / 改为人发起。

为什么：原 `automation_policy` 把两件完全不同的事（能不能升 Rule vs 要不要强制 Pending）塞进一个语义模糊、enum 未冻的槽，二者消费者不同（扫描 job vs 运行期 router）、生效时机不同，合一会改一误动一。"系统受控自动收紧"无触发条件、无 owner 节点（原归"automation_policy maintenance 非节点"占位），且与架构脊梁冲突——系统不自发做语义判断，语义收紧与 rule 失效同属人专属（`缺口地图.md` 2026-06-23 收口）。Rule Match 两个都不读：`force_pending` 在上游 router 已改道、`promotion_lock` 在扫描 job 已消费，RM 只收已放行交易、只比 RuleLog 规则。

文档：本条 Decisions 为删除 / 新增的权威记录。`force_pending` / `promotion_lock` 的 exact field schema / enum / refs 形态归 **EntityLog M3**，本条不冻 exact 取值。EntityLog `00/01/02` 删 `automation_policy` + 加两值，以及 CaseJudgment / RuleLog / Rule Match / Entity Resolution / Human Review / Case Log / `缺口地图.md` 等所有引用对齐，**由后续各窗口执行**（见配套迁移 prompt）；本窗口不改圈外正式草案。

## Rule Log（rule identity / 审计）

### D9 · 每条 rule 用稳定 `rule_id`；审计靠 rule_id + Transaction Log + Governance Log 回放，不钉版本 — 2026-06-26
RuleLog 每条 rule 带一个**稳定、唯一的 `rule_id`**，贯穿其生命周期不变（改动不换 id）。它是 rule 的引用 / 审计 handle：
- RuleMatch 命中后输出 `rule_id`（命中那条规则的 id；不再设单独的 `rule_ref` 名，2026-06-26 owner 定名）；
- Transaction Log 的 rule-hit 引用 = `rule_id`（**裸 id，不钉版本**）。

RuleLog 只存**当前可执行状态**（按 `rule_id` 索引的当前 payload），**不存版本历史**——版本 / 变更历史归 **Governance Log**，RuleLog 不与治理裁定竞争（[rule_log/02:106](BK_Copilot/memory_layers/rule_log/02_authority_lifecycle_and_boundaries.md)）。历史版本审计还原 = `rule_id` + Transaction Log 交易时间 + 回放 Governance Log 变更事件。

**对 Governance Log 的前置约束**：rule（及 entity policy）变更事件必须留足"能重建当时 rule 内容"的正文（改成什么 / 前后值或快照），否则 `rule_id` 回放还原不出历史态。已写入 `L2/独立question文档/governance-log-question.md`。

为什么：第一性原理——RM 命中要区分同一 entity 下多条规则、审计回溯、生命周期寻址、命中统计，都需"指名某一条"，`entity_id` 共享不够，故必须有稳定 `rule_id`。而"还原历史版本"不必把版本钉进 Transaction Log 引用：系统已把"当前态（各权威层）"与"变更历史（Governance Log）"分开（[rule_log/02:106](BK_Copilot/memory_layers/rule_log/02_authority_lifecycle_and_boundaries.md)、[governance-log-question §32](L2/独立question文档/governance-log-question.md)），RuleLog 自存版本反而违反该分工。bare `rule_id` + Governance Log 回放即可，前提是 Governance Log 记够内容（见上约束）。`rule_id` 的存在与角色本条已定；exact 字符串 / 格式仍归 RuleLog M3。

文档：本条 Decisions 为权威记录。同步更新 RuleLog `00/01/02`（rule_id 设计 + 当前态 / 历史分工）；约束写入 `L2/独立question文档/governance-log-question.md`；RM schema 中该输出字段定名 `rule_id`（原拟 rule_ref，owner 2026-06-26 改名），存档时落。Transaction Log rule-hit ref = `rule_id` 的对齐留 Transaction Log L3。

### D10 · Rule Match L3 节点决策（schema 定稿 + 字段锁定 + 删字段） — 2026-06-26
本轮完成 Rule Match 节点 L3 拆解，产出 `L3_schema/RM_Schema/Rule_Match_L3_Schema.md` 与字段链路大表 ER → Rule Match 段。节点级决策：
- **结构**：入口闸门 → A 取规则（按 `entity_id` 读 RuleLog）→ B 逐条比对 + 计数 → 命中数闸门（0 / 1 / ≥2）→ C 打包路由。runtime-only、自身不写任何 log。
- **不读 EntityLog、不判 eligibility**：`force_pending` 命中的交易在上游 ER→RM **eligibility router** 已改道 Pending、不进 RM；RM 只收已放行交易，残留兜底仅**结构性 fail-closed**（handoff 不成形 → `invalid_handoff·blocked`）。RM 只读 RuleLog。
- **匹配维度** = `direction` / `amount` / `transaction_date` / `currency`（+ `entity_id` 作 RuleLog 查询键）；`transaction_id` / `raw_description` / `evidence_refs` / `identity_reason` / `identity_evidence_refs` 全 pass。
- **输出**：`rule_match_outcome`（`rule_hit` / `rule_miss` / `invalid_handoff·blocked`）。命中时随 rule-hit handoff 一并输出 `rule_id` + `approved_accounting_treatment`（自 RuleLog 携带）+ `entity_id` + 客观事实 + `confirmed_by=rule_match` → JE Generation（+ finalization）。
- **分类来源统一用既有 `confirmed_by`**：RM 命中 set `confirmed_by = rule_match`；**不新造 `rule_match_source` 字段**——`confirmed_by` 是 Transaction Log（完整审计）/ Case Log（来源类型）既有跨系统字段，RM 只贡献此取值，enum 归 Transaction Log L3 / Case Log M3。
- **删除（RM 不需要、不挂开口）**：`evidence_condition`（证据充分性无稳定衡量标准）、`transaction_type` / `objective_tags`（全库无产出方）。
- **`evidence_refs` ≠ `identity_evidence_refs`**：前者 EI「交易→源材料」指针、后者 ER 身份投影；在 RM 均 pass（trace），都不作匹配条件。
- **链路大表方法约定**：穿过节点但不被消费的字段一律标 `pass`（n8n 式逐节点透传），不留空白（见大表前言）。

为什么：RM 单一职责 = 应用已批准规则；eligibility 是路由决定、归上游 router，不归 RM（权威分离）。分类来源复用既有 `confirmed_by`，守「一个值一个名」、不重造。`evidence_condition` / `transaction_type` 是无稳定判据 / 无产出方的孤儿字段，硬留会制造断头。

文档：`L3_schema/RM_Schema/Rule_Match_L3_Schema.md`、`L3_schema/字段链路大表.md`（ER→RM 段 + 口径 / 开口）、`L3_schema/_L3阶段执行计划.md`（Phase 1 勾上 Rule Match）；RuleMatch `00/01/02` 与 RuleLog `00/01/02` 已同步对齐（automation_policy→force_pending / promotion_lock、Rule ref→rule_id、桶C 删字段、confirmed_by）。

## confirmed_by 拆分

### D11 · 拆分 `confirmed_by`：新增 `classified_by`（谁做出分类）vs `confirmed_by`（最终结果确认 / 审计）— 2026-06-27
**修订 D10 末段「分类来源复用既有 `confirmed_by`、不重造」。** owner 定：
- `confirmed_by` 专用于**确认最后的会计分类结果**——在 Transaction Log 发挥作用 = 谁确认的审计留痕。
- Rule Match / Case Log / Case Judgment 表达「**这笔分类判断由谁 / 什么做出**」改用新值 **`classified_by`**（取值如 `rule_match` / `case_judgment`〔system 高置信〕/ `accountant`）。
- Rule Match 命中时 set `classified_by = rule_match`（原 set `confirmed_by = rule_match` 改此）。

为什么：D10「复用 confirmed_by」把"谁做的判断"和"谁确认了最终结果"挤进一个名，二者作用域不同（前者分类作出方、随 Case Log 复用；后者最终审计、随 Transaction Log）。拆两个名守「一个值一个名」。

**命名：`classified_by` 为推荐值，owner 可改名（如 `judged_by` / `decided_by`）——确认后再向各 log 传播。**
- exact enum：`classified_by` → Case Log M3 / Transaction Log L3；`confirmed_by` → Transaction Log L3。

文档：`L3_schema/RM_Schema/Rule_Match_L3_Schema.md`、`L3_schema/字段链路大表.md`（ER→RM 段）已改；Case Log / Transaction Log / Case Judgment 的对齐随各自 L3 落（本条为权威记录）。

## 字段链路大表 / 审核视图（L3 作业基建）

### D12 · 字段链路大表权威 = markdown；HTML 为派生审核视图、改名、按需更新 — 2026-06-28
owner 定双轨：
- **权威 = `L3_schema/字段链路大表.md`（markdown）**。L3 Schema 设计、节点间逻辑理解一律以它为准；新节点读它来理解节点之间的逻辑。旧表逻辑虽繁，但对 AI 理解节点间关系更友好，故保留为**权威参考**。
- **HTML 版为派生「审核视图」、非权威**：原 `字段链路大表.html` 改名为 **`字段链路审核视图.html`**（避免与权威同名、令后续节点误当权威）。它只供人眼审核、双击浏览器即开、不依赖 App。
- **更新次序与触发**：先更新 `.md`，再**仅在 owner 单独明确指示时**依据 `.md` 把值填进 HTML 的 JSON 数据块；不得让视图领先于 `.md`，不得把视图当设计权威或读取来源。HTML 数据 / 样式分离（只改 `<script id="tableData">` 的 JSON，渲染逻辑写死），保证多次更新设计一致。

为什么：让 AI 写入与人审核解耦——`.md` 是 AI 设计 / 读取面（更利于节点间逻辑理解），HTML 是人审核面；改名 + 去权威化防止后续节点把渲染视图误当权威。

文档：`L3_schema/字段链路审核视图.html`（改名 + 去权威化）、`L3_schema/字段链路大表.md`（表头加视图说明）、`L3_schema/_L3阶段执行计划.md`（Phase 2 + 总目标）、`L3_schema/节点L3Schema拆解_模板Prompt.md`（后续跨节点段）、`当前任务状态.md`（交接）。
