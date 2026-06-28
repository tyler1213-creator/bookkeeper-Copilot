# 夜间文档审查 · FINDINGS（明早审核用）

> loop 只读 + 追加产出。**没有任何被审文档被修改。** 每条都可按"位置"回到原文核对。
> 类型图例：A残留 · A′provenance(仅参考) · B来源层过时 · C孤儿候选 · C′缺失基础件候选 · D近义重复 · E决策-文档矛盾 · G悬空引用

---

==== ✅ 全部完成：41/41 项扫完，共 22 条 finding（F-001..F-022）+ provenance 参考。loop 已结束。 ====

## 早晨审核：先看这几条（按优先级）

**P0 — 开下一个节点（Case Judgment）前必须先拍**
- **F-002 / F-007 / F-015 · `evidence_condition`**：D10 写"已删/无稳定衡量标准/不挂开口"是**全局措辞**，但它在 **Case Log 字段#11 + EI 边界 + 缺口地图:63 + Case Judgment 读&产** 四处都活着。CJ 是下一个要设计的节点、其学习回路就靠它——开工前先定它到底"全系统死"还是"仅非 RM 输入"。

**P1 — 高置信、点状可改（安全，决策已存在）**
- **F-004 / F-006 / F-013 · `ER_L3:54` 一行残留**：把已删的 `risk_flags`(D1) + `automation_policy`(D8) 当 ER 现行读取字段，还过度声明读未建源（Governance/Intervention/Knowledge Summary）。一行集中改。
- **F-003 · `当前任务状态.md:103/109/124`**：仍把 auto-downgrade 归已删的 `automation_policy maintenance`（pre-D8 残留，且是"新窗口入口"文档，危害放大）。
- **F-001 · `case_log_boundary_note.md`**：superseded 却把 Post-Batch Lint/Governance Review 当现存读取者，且被引为正式草案来源。

**P2 — 治理/命名，宜早定 owner（防"下一个 rule_id"）**
- **F-018 / F-020 · `confirmed_by`**：既多头寄存（enum 挂 TxLog L3 + Case Log M3 + HR L3，无单一 owner），又**语义分裂**（Case Log/RM=来源类型 enum，Transaction Log=谁确认+审计叙事）。像 D9 给 rule_id 定 owner 那样给它定一个。
- **F-009 / F-014 · 命名漂移**：`rule ref`(旧名,散 TxLog/RuleLog) / `amount_abs`·`date`(RM 02:27 vs 权威 amount/transaction_date)。

**P3 — 低置信 / 观察项（设计下游时再核）**
- F-010（缺口地图把废节点列为待定边界，owner 自维护）、F-012（EI 的 source_account/source_channel/evidence_type 软消费）、F-017（HR 读"candidate context"接地待核）、F-021（冻结节点 L2 提案无 superseded 标记；CJ 提案仍活跃·勿动）、F-022（模板示例路径少 EI_Schema/）。

## 三个总体结论（对你那场对话的复盘）
1. **冻结节点档（`BK_Copilot/`）很干净**：删除几乎都写成 provenance（F-005/F-006/F-008/F-019），不是脏。你"全量清 70 份"是高估了。
2. **真问题集中在两处**：L3 schema 接缝行（尤其 `ER_L3:54`）+ 跨作用域措辞（D10 的 evidence_condition）。点状可清，不必重写。
3. **你担心的"反孤儿"在本库不是"无人登记"**（缺口地图很勤），而是**多头寄存**（confirmed_by 最像下一个 rule_id）——这是流程问题，不是脏。

> 全部只读，零文档改动，删/改/立全留给你拍。

---

### F-001 · `case_log_boundary_note.md` 是 superseded 文档，却仍把已废节点当现存读取者，且被引为正式草案来源
- 类型: B来源层过时 + A残留
- 位置:
  - `BK_Copilot/memory_layers/case_log/case_log_boundary_note.md`（"允许的读取者"段列 `Post-Batch Lint Node`、`Governance Review Node`；"如果历史案例显示…"路径图写 `Post-Batch Lint / Review / Governance Review candidate`；多处 `evidence condition`、`automation policy change`）
  - `当前任务状态.md`（把该文件标为 Case Log "正式 M1-M2 草案来源之一"）
- 证据: "`Post-Batch Lint Node`（批后体检节点）：读取 entity-linked case history…" / "`Governance Review Node`（治理审核节点）：读取案例依据…" / "-> Post-Batch Lint / Review / Governance Review candidate"
- 判定理由: Post-Batch Lint 与 Governance Review 两节点已废除（见 entity_log/02 §"已删除的读取者"），本文件却仍把它们当现存读取者与合法路径，且未加任何"已废除"标注 = 活残留；更危险的是它被 `当前任务状态.md` 列为正式草案来源，会把废弃概念向前回灌。
- 置信度: 高
- 建议动作(供审核, loop 不执行): 标 superseded 并清理读取者/路径，或从"草案来源"中除名
- 关联: 见 entity_log/02_authority_lifecycle_and_boundaries.md §27（两节点废除的权威记录）

### F-002 · D10 删 `evidence_condition` 的全局理由与 Case Log 仍把它当正式字段冲突
- 类型: E决策-文档矛盾
- 位置:
  - `Decisions.md:77`（D10："删除…`evidence_condition`（证据充分性无稳定衡量标准）"）
  - `BK_Copilot/memory_layers/case_log/01_memory_intent.md:42`（字段表 #11 `evidence_condition`："当时有哪类证据支撑…决定该先例何时可被复用"）
  - 同文件:7 / :12 / :51 / :58 / :110（整套 "evidence-conditioned precedent" 设计都建立在它上面）
- 证据: D10 "`evidence_condition`（证据充分性无稳定衡量标准）" ↔ case_log/01 "| 11 | `evidence_condition` | 当时有哪类证据支撑…决定该先例何时可被复用 |"
- 判定理由: D10 的删除本意是"从 Rule Match 输入里拿掉"，但写下的理由"无稳定衡量标准"是全局口径。Case Log 把 `evidence_condition` 当核心可定义字段——若有人据 D10 认为它全系统作废，会破坏 Case Log。上一轮把它判为"无主孤儿"是追溯不全（只查了 EI/ER，没查 Case Log 字段表）。
- 置信度: 高
- 建议动作(供审核, loop 不执行): owner 拍板——`evidence_condition` 是"全系统死"还是"仅非 RM 输入"；据此修 D10 措辞或 Case Log 字段
- 关联: D10；F 之后 P1.9 / P5.10 会补全所有出现点

---

<!-- loop 从此行下方继续追加 -->

### F-003 · `当前任务状态.md` 仍把 "automation 自动收紧 / auto-downgrade" 路由到已删的 `automation_policy maintenance`（pre-D8 残留，且在"新窗口入口"文档）
- 类型: A残留（与 D8 冲突）
- 位置:
  - `当前任务状态.md:103`（"auto-downgrade 归 automation_policy maintenance"）
  - `当前任务状态.md:109`（"automation 收紧归 Entity Log `automation_policy` maintenance / mutation contract"）
  - `当前任务状态.md:124`（"automation auto-downgrade 的检测 / 自动收紧 / 治理可见性改归 Entity Log `automation_policy` maintenance / mutation contract"）
- 证据: "automation auto-downgrade 的检测 / 自动收紧 / 治理可见性改归 Entity Log `automation_policy` maintenance / mutation contract，不归当前确定性发现 job"
- 判定理由: D8（Decisions.md:44）已删 `automation_policy`、并明确"系统不自发收紧"，auto-downgrade/自动收紧整体废除。这三行仍把它当 live 归属，是 D8 前的旧叙述未传播。危害放大：本文件是"新窗口最快理解当前情况的入口"，新会话先读它就会拿到 pre-D8 的错误地图。
- 置信度: 中（在 changelog/状态文本里，部分像历史描述；但 109/124 是现行归属陈述）
- 建议动作(供审核, loop 不执行): 改——按 D8 改成 force_pending/promotion_lock 会计师人发起、删"自动收紧"归属
- 关联: D8；F-005? 待 P5.8 复核

### F-004 · ER L3 Schema 把 `automation_policy` 列为 ER 读取的 live Entity Log 字段（应为 force_pending/promotion_lock）
- 类型: A残留
- 位置: `L3_schema/ER_Schema/Entity_Resolution_L3_Schema.md:54`
- 证据: "Entity Log（实体 / status / risk_flags / automation_policy）/ Alias Log / Governance Log … （read）"
- 判定理由: D8 后 Entity Log 治理字段是 `force_pending` / `promotion_lock`，不再有 `automation_policy`。此处把 `automation_policy` 当 ER 读取的现行字段 = 活残留，无删除标注。**附带**：同一行的 `risk_flags` 疑似 D1 删除的 `identity_risk_flags` 残留 → 移交 P1.8 核。
- 置信度: 高
- 建议动作(供审核, loop 不执行): 改——`automation_policy` → `force_pending`/`promotion_lock`；`risk_flags` 待 P1.8 判
- 关联: D8、D1；P1.8

### F-A′(参考) · `automation_policy` 的合法 provenance（不需改，仅列以免误删）
- 类型: A′provenance(仅参考)
- 位置: `Decisions.md:42/44/45/51/53/83`（D8/D10 删除记录）、`缺口地图.md:17`（"已随 D8 撤销"）、`L3_schema/RM_Schema/Rule_Match_L3_Schema.md:6/72`（"automation_policy 已删，拆为…"）
- 判定理由: 均为记录"已删/已拆"的审计性文字，是有意保留的 provenance。清理时**勿删**。
- 置信度: 高
- 建议动作(供审核, loop 不执行): 仅确认，保留

### F-005 · P1.2–P1.4 残留扫描：权威层零活残留（全为 provenance）
- 类型: A′provenance(仅参考)
- 位置 / 判定:
  - `automation_control_state`：权威层 **0 命中**（仅 `L2/L2_proposals/Rule Match__L2提案.md` 有，归 P7 来源层）。零残留。
  - `candidate_signal`：权威层全部为删除记录——`当前任务状态.md:101`、`Decisions.md:8`、`缺口地图.md:224`、`coordinator_node/02:282`、`ER_L3:26/50`、`entity_resolution_node/02:66`·`01:29`·`00:45/50`、`case_log/02:34`（"原拟 candidate_signal_refs 不设"）。**无当活字段用的残留。**
  - `merge_split_candidate`：权威层全部为删除记录——`Decisions.md:8`、`缺口地图.md:222`、`当前任务状态.md:101`、`coordinator_node/02:194`、`ER_L3:50`、`entity_resolution_node/02:66`·`01:29`·`00:45`、`entity_log/02:44`。merge/split **概念正确保留**（会计师人发起），只删了自动候选字段。**无残留。**
- 判定理由: 三项命中均带"已删/原X已删除/不再输出/不设"标注，是有意保留的审计 provenance；权威层 ER/coordinator/entity_log/case_log 在这三个词上是干净的。
- 置信度: 高
- 建议动作(供审核, loop 不执行): 仅确认，全部保留
- 关联: D1（candidate_signal/merge_split_candidate 删除权威记录）

### F-006 · P1.5–P1.8 残留扫描：ER 节点全 provenance；但 ER_L3:54 读了 Entity Log 不存在的 `risk_flags`（确认 F-004 的疑点）
- 类型: P1.5/P1.6/P1.7 → A′provenance(仅参考)；P1.8 `identity_risk_flags` → **A残留（确认）** + 次级 E
- 位置 / 判定:
  - `alias_conflict_issue`（P1.5）：权威层仅 `Decisions.md:8`、`ER_L3:50`、`entity_resolution_node/02:66`·`00:45` — **全 provenance，无残留。**
  - `identity_governance_issue`（P1.6）：仅 `Decisions.md:8`、`ER_L3:50`、`entity_resolution_node/02:66`·`00:45` — **全 provenance，无残留。**
  - `blocking_reason`（P1.7）：仅 `Decisions.md:8`、`ER_L3:50`、`entity_resolution_node/01:29`·`00:45` — **全 provenance，无残留。**
  - `identity_risk_flags` / `risk_flags`（P1.8）：ER 节点档与 ER_L3:50 均为删除记录（provenance）；**但 `L3_schema/ER_Schema/Entity_Resolution_L3_Schema.md:54` 把 `risk_flags` 当 ER 读取的 live Entity Log 字段**。
- 证据: `ER_L3:54` "Entity Log（实体 / status / **risk_flags** / automation_policy）… （read）" ↔ `entity_log/00_index.md:33` Entity Log 治理字段只有 `force_pending` + `promotion_lock`；`entity_log/02:27` "系统自发语义判断 merge/split/**身份风险**的发现层已裁撤删除"；ER 节点档 "身份风险**并入 force_pending**"。
- 判定理由: Entity Log 无 `risk_flags` 字段，身份风险已折叠进 `force_pending`。ER_L3:54 仍把 `risk_flags`（连同已在 F-004 记的 `automation_policy`）列为 ER 读取的现行字段 = 活残留。**次级**：`Decisions.md:9` 理由句"`identity_risk_flags` 真值归 Entity Log，ER 只读"与"风险并入 force_pending、Entity Log 无独立 risk_flags 字段"轻微不一致，可能是 D1 当时的折中措辞 → 低置信度 E，待 owner 确认口径。
- 置信度: ER_L3:54 残留=高；Decisions:9 口径不一致=低
- 建议动作(供审核, loop 不执行): ER_L3:54 把 `risk_flags`/`automation_policy` 改为 `force_pending`/`promotion_lock`（与 F-004 合并处理）；Decisions:9 口径由 owner 确认是否改写
- 关联: D1、D8；F-004（同一行 automation_policy）

### F-007 · `evidence_condition` 是系统级 live 概念，D10 仅在 RM 作用域删除却用了全局措辞——矛盾范围比 F-002 更广（扩展 F-002）
- 类型: E决策-文档矛盾（扩展 F-002，补全所有出现点）
- 位置:
  - **RM 作用域删除记录（provenance，正确，勿改）**：`Decisions.md:77/81`、`L3_schema/字段链路大表.md:90`、`L3_schema/RM_Schema/Rule_Match_L3_Schema.md:71`、`rule_match_node/02:27` —— 均为"RM 不需要、已删"。
  - **系统级 live 使用（与上面全局措辞冲突）**：`case_log/01_memory_intent.md:42`（字段 #11，见 F-002）、`evidence_intake_node/02:133` 与 `00:32`（"evidence_condition 的 exact schema 归 Case/Rule 侧完成联合 L3"）、`缺口地图.md:63`（"evidence_condition exact schema 与 sufficiency threshold → 【L3】"，明确把它当**未冻结的 L3 开口**）。
- 证据: `Decisions.md:77` "删除…`evidence_condition`（证据充分性无稳定衡量标准）…不挂开口" ↔ `缺口地图.md:63` "`evidence_condition` exact schema 与 sufficiency threshold → **【L3】**" ↔ `EI 00:32` "归 Case/Rule 侧完成**联合 L3**"
- 判定理由: 正确的事实是 `evidence_condition` **只是不作 RM 输入**——它在 Case Log 是字段、在 EI 是"归下游定义的概念"、在缺口地图是 L3 开口。D10 写的"已删 / 无稳定衡量标准 / 不挂开口"是全局措辞，与 EI/缺口地图/Case Log 三处直接冲突。任何人据 D10 认为它全系统作废，会同时破坏 Case Log + EI 边界 + 缺口地图 L3 计划。**RM 侧删除本身没错**（那几处是正确 provenance），错的是 Decisions 的措辞作用域。
- 置信度: 高
- 建议动作(供审核, loop 不执行): owner 改 D10 措辞为"仅从 RM 输入移除，不代表系统级删除；其 schema 仍是 Case/Rule 侧 L3 开口"；或反向——若真要全系统杀，则需同步改 Case Log #11 + EI 边界 + 缺口地图:63
- 关联: F-002、D10；缺口地图:63 的 L3 开口

### F-008 · `transaction_type` / `objective_tags` / `rule_match_source`：确认干净（孤儿正确删除 / 新造决定守住）
- 类型: A′provenance(仅参考)
- 位置 / 判定:
  - `transaction_type`（P1.10）、`objective_tags`（P1.11）：权威层仅 `Decisions.md:77(/81)`、`字段链路大表:90`、`RM_L3:71`、`rule_match_node/02:27` —— **全为删除记录**，全库无任何节点产出/消费，真孤儿正确删除。
  - `rule_match_source`（P1.13）：权威层仅 `Decisions.md:76`（"不新造 `rule_match_source` 字段"）—— **零 live 出现，"不重造、复用 confirmed_by"的决定守住了。**
- 判定理由: 三者均无活残留；全部是有意 provenance 或"未发生的新造"。
- 置信度: 高
- 建议动作(供审核, loop 不执行): 仅确认，保留
- 关联: D10

### F-009 · "rule ref" 旧名仍散布于 Transaction Log / Rule Log / Human Review（D9 已改名 `rule_id`，但这些 log 的对齐被 D9 显式推迟到其 L3/M3）
- 类型: A残留（命名，pending-rename）
- 位置:
  - 仍用裸 "rule ref" 作概念名：`human_review_node/02:35`、`transaction_log/01:37`·`01:112`·`00:30`·`02:103`·`02:160`、`rule_log/01:69`·`00:44`·`02:104`·`02:105`
  - 已与 rule_id 对齐的（半 provenance，可不动）：`rule_log/00:38`·`02:170`（"…已定，见 D9：rule-hit 引用 = `rule_id` 裸值…"）
  - 纯 provenance（勿改）：`Decisions.md:59/68/83`、`rule_match_node/02:130`·`02:183`（"原称 Rule ref"）
- 证据: `Decisions.md:59` "不再设单独的 `rule_ref` 名" ↔ `transaction_log/01:37` "| 9 | rule-hit ref | …source 语义与 rule ref；exact 字段形态留 M3 |"
- 判定理由: D9 把 RM 输出字段定名 `rule_id`，但 Transaction Log / Rule Log / Human Review 仍普遍用旧名"rule ref"指同一概念。**缓和因素**：`Decisions.md:68` 明确"Transaction Log rule-hit ref = rule_id 的对齐留 Transaction Log L3"——即这些是 D9 **已知推迟**的对齐，不是漏网。是否现在统一改名是 owner 的取舍。
- 置信度: 中（确属旧名；但 D9 已显式 defer，非纯漏改）
- 建议动作(供审核, loop 不执行): owner 决定——现在统一"rule ref"→"rule_id"，或保持 D9 的"留各 log L3 对齐"
- 关联: D9（Decisions:59/68）

### F-010 · P2 废节点扫描：冻结档全 provenance（干净）；仅 `缺口地图.md` 规划清单仍把 Governance Review / Post-Batch Lint 当待定边界对象
- 类型: P2.3 → A′provenance(干净)；P2.1/P2.2 → A残留（低置信，缺口地图规划清单）
- 位置 / 判定:
  - `语义发现器 / FP-5 / 系统自发语义发现层`（P2.3）：`当前任务状态:97/98/103/118/123/129/134`、`缺口地图:17/299`、`human_review_node 02:14/258/261`·`01:44/103`·`00:46/49` —— **全部"已裁撤删除"记录，零活引用。极干净。**
  - `Governance Review`（P2.1）/`Post-Batch Lint`（P2.2）：冻结档全部为废除/provenance（`entity_log/02:27`、`coordinator/02:278` "无独立 Governance Review Node"、`case_log/02:23` 同、`transaction_log 00:52`·`02:182` "Post-Batch Lint 已废除，删除"、`human_review 00:49`·`02:261`、`当前任务状态:79/103/129`、`缺口地图:17` 废除节点声明）。
  - **唯一残留**：`缺口地图.md:221`（把 **Governance / Governance Review**、**Review Node** 列为 L2·外阻待划界对象）、`缺口地图.md:263`（"**Governance / Governance Review**、…**Post-Batch Lint**…与 Transaction Log 的 exact 读写边界 → L2·外阻"）——把已废节点当"仍需定义读写边界的待定对象"。
- 证据: `缺口地图:263` "…Evidence Log、**Post-Batch Lint**、Profile / Structural Match 与 Transaction Log 的 exact 读写边界 → 【L2·外阻】" ↔ `缺口地图:17` 同文件已声明 Post-Batch Lint/Governance Review 废除
- 判定理由: 这两行仍为废节点保留"待划界"占位，逻辑上自相矛盾（边界对象不该是已废节点）。**缓和**：同文件顶部 `:17` 有"各对象 section 若以圈外节点指代，按此改读"的 remap 声明，且 `缺口地图.md` 由 owner 单独维护。故低置信、供 owner 自查，不一定算错。
- 置信度: 低（被 :17 remap 覆盖，且 owner 自维护）
- 建议动作(供审核, loop 不执行): owner 自查 `缺口地图:221/263`——把 Governance Review / Post-Batch Lint 从待定边界清单移除或替换为现行机制（Governance=HR签字+Finalization；Post-Batch Lint=确定性发现 job）
- 关联: D1、D8；缺口地图:17 remap 声明

### F-011 · P3.1 字段链路大表追溯：链内无真孤儿；但有 1 个"消费无产出"（已挂开口）+ 1 个 node档/L3 分歧 + 命名接地小问题
- 类型: C′缺失基础件候选（已挂开口）+ node档↔L3 分歧 + 低severity 命名
- 位置 / 判定:
  - **`structural_path_status`（C′：消费无 live 产出，已挂开口）**：ER 读它（`ER_L3:54/61`、`entity_resolution_node/02:14/38`），产出方 = Profile / Structural Match Node，**该节点未落实**（`字段链路大表:24`、`entity_resolution_node/02:24`）。`大表:24` 已明确挂为开口（"ER 对非结构性确认的触发依赖挂为开口，待 Profile 落文"）→ **已知开口，非漏网孤儿**，但确属"消费者在、生产者尚不存在"的反孤儿型依赖。
  - **`counterparty_signals` / `evidence_association`（node档↔L3 分歧）**：在 `L3_schema/EI_Schema/` + `ER_Schema/` + `大表` 有定义，但 **EI 冻结节点档 `evidence_intake_node/00·01·02` 无任何出现**。即 L3 层把它们当 EI 产出，节点档却未承载 → 待 P3.2 判是"L3 合理细化"还是"无节点档接地的软幻影"。
  - **`approved_accounting_treatment`（命名接地，低）**：snake_case 字段名只在 `RM_L3` + `大表`；RuleLog 用散文形 "approved accounting treatment" 接地（`rule_log/01:12/31/37/43`、`00:17/29`、`02:11/107/161`），并把 exact shape 留 JE Generator L3。→ 有真实产出方（RuleLog），只是 snake_case 名是 RM 侧造的、RuleLog 未用同名。非孤儿。
  - **下游落点未建节点（dangling destination，已挂开口）**：`rule_id` / `approved_accounting_treatment` / `rule_match_outcome` 落点 = JE Generation / Case Judgment，JE Generation 未落文（`大表:88` 挂开口）。→ 已知开口，非孤儿。
- 证据: `字段链路大表:72` "上表无断头线、无无主字段" ↔ `大表:24` 自己把 `structural_path_status` 挂开口（说明"无断头"是指 EI→ER→RM 链内，跨未建节点的依赖另挂开口）
- 判定理由: 表内（EI→ER→RM 三节点之间）确实 producer/consumer 完整，line 72 成立；真正的"缺口"都在与**未落实节点**（Profile、JE Generation）的接缝处，且都已挂开口。唯一未挂开口的异常是 counterparty_signals/evidence_association 的 node档↔L3 不一致，交 P3.2。
- 置信度: structural_path_status/JE 落点=高（确为开口型依赖）；counterparty_signals 分歧=中（待 P3.2）；命名=低
- 建议动作(供审核, loop 不执行): 仅确认开口型依赖；P3.2 复核 counterparty_signals/evidence_association 的节点档接地
- 关联: 大表:24/88、D9；F-004/F-006（ER_L3:54 同行 risk_flags/automation_policy 残留）；转 P3.2

### F-012 · P3.2 Evidence Intake 字段：F-011 分歧解除（概念已接地）；新增 3 个"软消费"候选（低置信，供下游设计时核）
- 类型: C孤儿候选（软，低置信）+ F-011 降级说明
- 位置 / 判定:
  - **F-011 分歧解除**：`counterparty_signals` / `evidence_association` 在 EI L3（`Evidence_Intake_L3_Schema.md:47/48`，D·create）定义；EI 节点档**以散文形接地**——`evidence_intake_node/02:127`、`01:36`（"counterparty surface signals → Entity Resolution"）。即概念在节点档有娘家，L3 只是给了 snake_case 名。**非幻影，降为低 severity 命名。** （`evidence_association` 散文接地略弱，但"证据↔交易配对"是 EI 核心职责。）
  - **`source_account`（软孤儿候选，低-中）**：EI 产（`EI_L3:30`，B·remake），`大表:22` 标"透传下游"，但**无命名消费者**——RM 匹配维度只有 direction/amount/date/currency（不含 source_account），JE Generation 未建。produced + passed，但目前无人 read。
  - **`source_channel` / `evidence_type`（软孤儿候选，低）**：EI 产（`EI_L3:40/41`，A·create），`大表:21` 标 ER "read（按需）"——消费是"按需/on-demand"，非确定消费。可能 produced 后实际无人 read。
  - **旁路信号去未建层（dangling destination，已知）**：`statement_chain_status`/`coverage_signal`→目标层、`duplicate_material_signal`/`content_fingerprint`→材料/纠错层、`quality_issue_signal`→Coordinator/Review；`大表:26` 已声明这些不入主链、去外圈层。目标层/纠错层未建 = 落点悬空，但已 acknowledged。
- 证据: `EI_L3:30` "`source_account` 来源账户标识（原文级，非会计科目）" + `大表:22` "透传下游" ↔ RM 匹配维度（`Decisions:D10` / `大表:51`）不含 source_account
- 判定理由: EI 产出比下游实际消费的多（source_account / source_channel / evidence_type 是 produced-but-maybe-unread）。这不一定是 bug——可能是为未建的 JE Generation / 目标层预留。但属"产出方在、确定消费方缺"的软孤儿，应在下游节点设计时确认是否真有人读，否则就是噪音字段。
- 置信度: 低（多为 acknowledged 透传 / 预留；需下游节点落文才能定论）
- 建议动作(供审核, loop 不执行): 标记观察项——设计 JE Generation / 目标层时核 source_account / source_channel / evidence_type 是否真被消费；若否则删
- 关联: F-011（分歧已解除）；大表:21/22/26；D10（RM 匹配维度）

### F-013 · P3.3 Entity Resolution 字段：输出全接地（无孤儿）；输入第 54 行声明读取多个未建源，疑似过度声明 + 重申 F-004/F-006 残留
- 类型: C′缺失基础件候选（dangling read，低-中）+ 重申 A残留
- 位置 / 判定:
  - **ER 输出全部接地（无孤儿）**：`identity_state`/`entity_id`/`identity_reason`/`identity_evidence_refs`（→ RM/CJ + Transaction Log）、`created_by`（→ Entity Log）、`alias`（→ Alias Log + Entity Log）——均有 create + 落点（`ER_L3:32-37/45-46`、`大表:32-37`）。旁路信号已清零（`ER_L3:50` provenance）。
  - **输入过度声明候选（dangling read，低-中）**：`ER_L3:54` 读取源列了 `Governance Log` / `Intervention Log` / `Knowledge Summary`——三者均未落 `BK_Copilot/` 冻结档（圈外/L2·外阻）。而分块分析（入口/A/B/C，`ER_L3:61-78`）可见消费的只有 Entity Log + Alias Log + AI 外部搜索；D 块是否用到那三者本轮未核 → 疑似"声明读取 > 实际消费"的过度声明。
  - **重申残留**：同 `ER_L3:54` 的 `risk_flags` / `automation_policy` 已在 F-004 / F-006 记为活残留（Entity Log 无此二字段）。
- 证据: `ER_L3:54` "…Entity Log（实体 / status / risk_flags / automation_policy）/ Alias Log / Governance Log / Intervention Log / Knowledge Summary / 外部搜索（read）" ↔ 分块分析仅见 Entity Log/Alias Log/外部搜索被实际用
- 判定理由: ER 输出侧干净；隐患都在第 54 行这条"大杂烩读取声明"——既混入了已删字段（risk_flags/automation_policy），又列了未建且未被任何可见块消费的源（Governance Log/Intervention Log/Knowledge Summary）。这正是"L3 一行把所有可能读的都写上"导致的接地不清。
- 置信度: 残留=高（同 F-004/F-006）；过度声明=低-中（D 块未穷尽核，需 owner 确认 ER 是否真读那三源）
- 建议动作(供审核, loop 不执行): 收紧 `ER_L3:54`——删 risk_flags/automation_policy；核 Governance Log/Intervention Log/Knowledge Summary 是否真被某块消费，否则降为开口或删
- 关联: F-004、F-006、F-011；ER_L3:54

### F-014 · P3.4 Rule Match 字段：全接地（无孤儿）；`rule_match_node/02:27` 用 `amount_abs`/`date`，与权威名 `amount`/`transaction_date` 不一致（违 大表:70 同值同名）
- 类型: A残留（命名漂移，低-中）
- 位置 / 判定:
  - **RM 字段全接地（无孤儿，无残留）**：输入 `entity_id`(ER)/`direction`/`amount`/`transaction_date`/`currency`(EI 匹配维度)/`transaction_id`/`raw_description`/`evidence_refs`(pass)/`identity_*`(ER pass)；输出 `rule_match_outcome`(create)/`rule_id`/`approved_accounting_treatment`(read→输出)/`confirmed_by`(set)——全有 producer + 落点（`RM_L3:50-64/79-101`）。落点 JE Generation 未建已挂开口（`RM_L3:107/108`）。**本节点是上轮新设计，最干净。**
  - **唯一问题——命名漂移**：`rule_match_node/02:27`（桶C）写 "transaction_id、direction、**amount_abs**、**date**、currency"，而**权威名**是 `amount` / `transaction_date`（`字段链路大表:22/51`〔唯一权威〕、`EI_L3:27`、`RM_L3:60/79`，连同一文件 `02:186` 都用 `amount`/`date`→`transaction_date`）。`amount_abs` 全库仅此一处。
- 证据: `rule_match_node/02:27` "…direction、amount_abs、date、currency…" ↔ `字段链路大表:70` "同值同名" 原则 + `大表:51` 用 `amount`/`transaction_date`
- 判定理由: 同一字段两个名（`amount_abs` vs 权威 `amount`；`date` vs `transaction_date`），违反字段链路大表自定的"同值同名"。非功能 bug，但正是"接表阶段"该统一的命名噪音；`amount_abs` 这种孤名最易在后续 schema 派生时造成断头。
- 置信度: 中（确为同值异名；但属节点档措辞，非 L3 权威表）
- 建议动作(供审核, loop 不执行): 把 `rule_match_node/02:27` 的 `amount_abs`→`amount`、`date`→`transaction_date`，与权威表统一
- 关联: 大表:70/51；D10（RM 匹配维度）

### F-015 · P3.5 Case Judgment：字段接地良好；但 CJ 是 `evidence_condition` 的又一活跃产消点（强化 F-007，且 CJ 是下一个要设计的节点）
- 类型: E决策-文档矛盾（强化 F-002/F-007）+ C′读未建源（已知）
- 位置:
  - **CJ 读 `evidence condition`**：`case_judgment_node/02:57`（"Case Log | …precedent 摘要、**evidence condition**、context note、`confirmed_by`、`use_level`"）
  - **CJ 产 `evidence condition`**：`case_judgment_node/02:98`（"High Confidence Classification 自带…例如 **evidence condition**、context note、use_level 倾向…经 finalization 沉淀为 Case Log 先例"）
  - CJ 读未建源：`02:58` entity-level Knowledge Summary（"尚无正式草案"，acknowledged）
- 证据: `Decisions.md:77`(D10) "删除…`evidence_condition`…不挂开口" ↔ `case_judgment_node/02:57` CJ 从 Case Log 读它 + `02:98` CJ 产出它喂回 Case Log
- 判定理由: 继 Case Log（F-002）、EI、缺口地图（F-007）之后，**Case Judgment 是 evidence_condition 的第四个活跃站点**，且是产 + 消两端都用。D10 的全局"已删"措辞与 CJ↔Case Log 的核心学习回路直接冲突。**这条对你尤其要紧：CJ 是主链下一个要设计的节点，开工前必须先拍定 evidence_condition 到底死没死。** 其余 CJ 字段（rule_miss/unknown 入口、High Confidence/Pending 输出、confirmed_by）均接地。
- 置信度: 高
- 建议动作(供审核, loop 不执行): 把 D10 的 evidence_condition 口径先于 CJ 设计敲定（见 F-002/F-007 建议）；Knowledge Summary 读取保持开口
- 关联: F-002、F-007、D10；CJ 为下一节点

### F-016 · P3.6 Coordinator：干净——单一输入面、输出去 acknowledged 外圈，无残留无孤儿
- 类型: A′provenance(仅参考，干净)
- 位置 / 判定:
  - 输入唯一面：`coordinator_node/02:36/40`（"无独立读取面，全部上下文来自 CJ Pending handoff 单一来源"）；`02:46` 明确不读 Intervention Log。
  - 输出：`02:129` 转 JE 格式 → JE Generation（圈外 L2·外阻，acknowledged）、`02:134` 交互留痕 → Intervention Log（圈外 L2·外阻，acknowledged）、candidate → governance/review 外圈。
  - 治理字段 `force_pending`/`promotion_lock`/merge/split 仅作"非身份治理候选交外圈"，Coordinator 不裁决（`02:68/69/105/117`）——用法正确，无残留。
- 判定理由: 单输入、输出落点都是已声明的外圈未建节点（已挂开口），无悬空、无已删词残留、无孤儿字段。
- 置信度: 高
- 建议动作(供审核, loop 不执行): 仅确认
- 关联: 大表:88（JE Generation/Intervention Log 开口）

### F-017 · P3.7 Human Review + P3.8 memory layers：无新孤儿；HR 读"candidate context"待核 + evidence_condition 确认为 Case Log 字段#11
- 类型: 多项（A残留低置信 + E 强化 + 命名）
- 位置 / 判定:
  - **Human Review（P3.7）读写面基本干净**：读 Transaction Log / Case Log / Entity Log（`human_review_node/02:35-37`）、写经 Finalization（`02:52-55`）均接地。**低置信项**：`02:37` 读 Entity Log 的"**candidate context**、merge-split…context"——D1 删了自动候选字段，保留的候选是确定性发现 job / 会计师人发起（`entity_log/02:155`），其在 Entity Log 的存储字段不明。需核"candidate context"是否对应 D1 后真实存在的字段，否则是旧自动候选模型的措辞残留。
  - **memory layers（P3.8）无新孤儿**：
    - `case_log/01` 字段 1-14 全有定义；**字段#11 `evidence_condition` 是该词的定义性娘家**（`case_log/01:42`）——再次坐实 F-002/F-007/F-015：它是 Case Log 正式字段，D10"已删"全局措辞与之冲突。
    - `transaction_log/01` 字段 1-13 全为审计字段、无孤儿；字段#9 "rule-hit ref…rule ref"= F-009 命名。
    - 命名漂移：`case_log/01:38` 用 `date`（字段7）、`account`（字段8），非权威 `transaction_date`；与 F-014（RM `amount_abs`/`date`）同类的跨层命名不统一。
- 证据: `human_review_node/02:37` "Entity Log | …candidate context、merge-split / force_pending…" ↔ D1 删自动候选字段；`case_log/01:42` 字段#11 `evidence_condition`
- 判定理由: HR/memory 层无真孤儿、无已删节点残留；遗留的是（a）HR 读"candidate context"接地待确认、（b）evidence_condition 在 Case Log 的定义性存在再确认、（c）跨层 date/amount 命名不统一（与 F-009/F-014 同根）。
- 置信度: candidate context=低；evidence_condition 字段=高；命名=低
- 建议动作(供审核, loop 不执行): 核 HR "candidate context" 字段接地；evidence_condition 随 F-007 统一口径；命名统一随 F-014 一并处理
- 关联: F-002/F-007/F-015（evidence_condition）、F-009/F-014（命名）、D1

### F-018 · P4 反孤儿/缺失基础件：缺口地图很全（隐藏型基础件风险低）；真正的"下一个 rule_id"风险在多头寄存、无单一 owner 的共享基元
- 类型: C′缺失基础件候选（多头寄存型，中置信）
- 位置 / 判定:
  - **缺口地图覆盖充分**：`缺口地图.md` 把被依赖却未定义的概念都登记为显式开口（JE Generation、Intervention Log、Governance Log〔待建〕、Knowledge Summary、Profile、eligibility router、审核 inbox、确定性发现 spec 等，见 :221/:258-298）。→ **隐藏型（无人登记的）缺失基础件风险低**——这正是 `缺口地图` 在做的事。
  - **真风险 = 多头寄存基元（rule_id 式）**：
    - **`confirmed_by` enum**：被全系统产消（RM set `rule_match`、CJ set `system`、HR set accountant、Case Log/Transaction Log 存），但 exact enum 同时挂在 **Transaction Log L3 + Case Log M3 + Human Review L3** 三处（`RM_L3:109`、`case_log/01:43`字段12、`缺口地图:277`、`Decisions D10`）——**无单一 owner**。
    - **`force_pending` enum**：被 eligibility router + Entity Log + RuleLog 依赖，enum 挂 "EntityLog / RuleLog / Governance 联合 L3"（`大表:87`、`entity_log:46/56`）——多头。
    - **eligibility router 机制**：RM"只收已放行交易"的整个前提依赖它，却只挂 L4/seam（`大表:87`）、无 owner 节点——承重却无人认领。
- 证据: `RM_L3:109` "`confirmed_by` 的 exact enum…→ Transaction Log L3 / Case Log M3" + `缺口地图:277` "`confirmed_by` exact enum → 【L3】（与 Case Log M3 对齐）" —— 同一 enum 三处挂，无主 owner
- 判定理由: 你担心的"反孤儿"在本库不表现为"无人登记"（缺口地图很勤），而表现为**多头寄存**：一个被全系统依赖的基元，其权威定义被分散挂到 2-3 个 L3/M3，谁都能拖、谁都不必先做——这正是 `rule_id` 当年的处境（躲在 L3-open 里没人立）。`confirmed_by` 是当前最像"下一个 rule_id"的。
- 置信度: 中（均已登记为开口，非隐藏；风险在治理流程——多头=易拖）
- 建议动作(供审核, loop 不执行): 给 `confirmed_by` / `force_pending` 各指定**单一 owner 层**先定 enum（像 D9 给 rule_id 定 owner 那样），避免多头拖成第二个 rule_id
- 关联: F-009（rule_id 先例）、D9、D10；大表:87

### F-019 · P5 决策-文档矛盾（D1–D10 逐条）：D1–D7 传播干净；所有矛盾收敛到已记 finding
- 类型: 汇总（E 类已分散记录）+ D1–D7 清洁确认
- 逐条结果:
  - **D1**（删并行候选通道）：live 全 provenance（F-005/F-006）。**唯一矛盾**：D1:9 理由"`identity_risk_flags` 真值归 Entity Log，ER 只读"，但 Entity Log 无 risk_flags 字段（身份风险并入 force_pending）——已记 **F-006**（低置信 E）。
  - **D2**（alias 命中只认是谁）：ER 02 已删 "冲突或多 entity 竞争 / alias 库冲突→unknown"（grep 空）→ **传播干净**。
  - **D3**（trace 瘦身）：ER 02 已删 `evidence_used` / `matched surface text` / `Alias lookup basis`（grep 空）→ **干净**。
  - **D4**（保留两类非-alias 冲突）：与 D2 不冲突，§9 保留 → **一致**。
  - **D5**（身份审计落 Transaction Log，ER 不写）：ER 无"写 Transaction Log"矛盾（grep 空）→ **一致**。
  - **D6**（ER 不预加工证据）：澄清型、无文档改 → **一致**。
  - **D7**（ER 命名收敛 created_by 等）：`created_by` 一致使用，`creation_provenance` 仅作"替代原 creation_provenance"provenance、无残留字段 → **干净**。
  - **D8**：automation_policy 残留 → 已记 **F-003 / F-004**。
  - **D9**：rule ref→rule_id 命名 → 已记 **F-009**。
  - **D10**：evidence_condition 全局措辞 → 已记 **F-002 / F-007 / F-015**。
- 判定理由: D1–D7（ER 那批）传播质量高，无新矛盾；D8–D10 的矛盾此前各 phase 已抓全。P5 未新增独立 finding，只确认覆盖完整。
- 置信度: 高
- 建议动作(供审核, loop 不执行): 无新增；按 F-002/3/4/6/7/9/15 处理
- 关联: F-002/3/4/5/6/7/9/15

### F-020 · P6 跨文档字段口径：`confirmed_by` 语义分裂（类型 enum vs 人员+审计叙事）——加重 F-018
- 类型: E决策-文档矛盾（语义分裂，中置信）+ direction enum 已知开口（低）
- 位置:
  - **`confirmed_by` = 来源类型 enum**：`case_log/01:43`（字段12 "记录结论的**权威来源类型**…具体人员和完整审计留痕**归 Transaction Log**"）、RM set `confirmed_by=rule_match`（类型值，`Decisions D10`）。
  - **`confirmed_by` = 谁确认 + 审计叙事**：`transaction_log/01:39`（字段11 "`confirmed_by` 留痕 | 结论由**谁**确认的具体人员与完整审计留痕"）。
  - direction enum：`EI 00:30`、`大表:73`、`_L3阶段执行计划:85/132` 已明确"待与 RM/JE 统一"（例 money_in/out vs inflow/outflow）→ 已知开口。
- 证据: `case_log/01:43` "权威来源**类型**" ↔ `transaction_log/01:39` "由**谁**确认的具体人员与完整审计留痕" —— 同名 `confirmed_by`，一处是类型 enum、一处是人员+叙事
- 判定理由: `confirmed_by` 在 RM/Case Log 是"分类来源类型"（rule_match/system/accountant 这种 enum 值），在 Transaction Log 是"谁确认 + 完整审计留痕"。可能是有意分面（类型在一处、人员细节归 Transaction Log），但同名承载两种数据，是"一个名两个值"的隐患，正好加重 F-018（confirmed_by 多头寄存无 owner）。direction enum 不同名是已登记开口，非隐藏。
- 置信度: confirmed_by 分裂=中（措辞可能是有意分面，需 owner 确认）；direction=低（已知开口）
- 建议动作(供审核, loop 不执行): 与 F-018 一起处理——给 confirmed_by 指定单一 owner，明确"类型 enum"与"确认人/审计留痕"是同字段两facet 还是两字段
- 关联: F-018、D10；大表:73（direction）

### F-021 · P7 过时来源层：冻结节点的 L2 提案无 superseded 标记、仍把 pre-D8 `automation_policy` 当"已锁定"，且被引为蒸馏来源
- 类型: B来源层过时（中置信）
- 位置:
  - `L2/L2_proposals/Rule Match__L2提案.md:5`（"Entity Log 已锁定：…保存 stable entity identity、entity_status 与 **automation_policy** / control state"）、`Entity Log__L2提案.md`（"automation policy 不随本体一起确认"）——把 D8 已删的 automation_policy 当现行"已锁定"事实。
  - 无 superseded banner（header 直接是 "# X L2 提案" + 依赖）；`当前任务状态.md` 仍把各 L2 提案列为对应节点正式草案的来源。
  - **重要例外**：`Case Judgment__L2提案.md` header 标 "> 进行中"——CJ 尚未 L3，其 L2 提案是**当前活跃文档**，不算过时来源，勿清。
- 证据: `Rule Match__L2提案.md:5` "Entity Log 已锁定：…automation_policy / control state" ↔ `Decisions D8` automation_policy 已删
- 判定理由: 冻结节点（Rule Match / Entity Log / ER）的 L2 提案是整份过时的来源层（陈述 pre-D8 概念为"已锁定"），无 superseded 标记，仍坐活动目录且被当蒸馏源——这是 F-001 模式的规模化版本，潜在再污染源。但 CJ 的 L2 提案在建中、是活的，必须区别对待。
- 置信度: 中（确为过时来源；但 BK_Copilot 已取代它们作权威，危害是"被误读回灌"而非当前生效）
- 建议动作(供审核, loop 不执行): 给已冻结节点的 L2 提案加 superseded banner（指向 BK_Copilot 正式草案），保留作历史；CJ 的 L2 提案不动
- 关联: F-001（同模式）、D8；当前任务状态.md（来源声明）

### F-022 · P8 悬空引用：1 处真悬空（模板示例指针路径少了 EI_Schema/ 子目录），其余为正则伪命中/模板占位
- 类型: G悬空引用（低）
- 位置: `L3_schema/节点L3Schema拆解_模板Prompt.md:4`
- 证据: "已完成的 `L3_schema/Evidence_Intake_L3_Schema.md` 就是标准产出" —— 但真实路径是 `L3_schema/EI_Schema/Evidence_Intake_L3_Schema.md`（在 `EI_Schema/` 子目录下）
- 判定理由: 该引用缺 `EI_Schema/` 子目录段，指向不存在的 `L3_schema/Evidence_Intake_L3_Schema.md`。属模板里的"范例指针"，低危（不是承重契约引用）。其余检查：`字段链路大表.md`、`Rule_Match_L3_Schema.md`、RM schema 的 `../../缺口地图.md`/`../../BK_Copilot/...` 相对引用**全部解析存在**；`03_data_contract.md` / `{{节点名}}_L3_Schema.md` / `n` 是模板命名占位、非悬空。
- 置信度: 中（确为错路径，但属模板示例）
- 建议动作(供审核, loop 不执行): 把 `节点L3Schema拆解_模板Prompt.md:4` 路径补为 `L3_schema/EI_Schema/Evidence_Intake_L3_Schema.md`
- 关联: 无
