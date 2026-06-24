# 第三窗口 kickoff prompt — 系统自发发现层：先裁必要性，再谈融入

> 用途：给 Claude 自己用的讨论窗口 prompt。owner 会 fork 回"刚出完三份 prompt"的时间点后，用本 prompt 开始解决第三个问题。自包含，携带 2026-06-23 之前各轮已收口的关键结论。

```
角色与任务：第一性原理审计 AI 记账系统设计仓库（非对来源忠诚）。本窗口任务特殊——「系统自发发现层」的必要性本身存疑、已挂起。先做必要性裁决（保留 / 删除 / 降级），再谈融入。**不要默认它该存在。** 术语：一律用「判句」，绝不用「谓词」。

【本窗口议的"系统自发发现层" = 系统自发做语义判断去找问题、再喂给人审。它包含两类，作同一个问题处理：】
- ① 系统自发 merge/split（语义发现器，原已废除的 Post-Batch Lint 的语义那半）。
- ② FP-5：系统自发发现"某笔记账可能错了"（错误发现入口）。
owner 已确认 ①② 本质同一——"系统自发语义判断 → 喂给人审"，必须一起裁。

【关键先验：已有一个同类子问题被裁成"人专属"，当先例用】
2026-06-23 已收口：**"某条 rule 是否失效 / 该不该降级"不由系统自发判断，只由会计师在 review 结果时人发起**。理由（owner 拍板）：rule 由人创建确认、匹配出错概率低，系统自发判不出 rule 失效；只有人在 review 时才能发现，降级人发起（走 Human Review 扩张型变更路径）。该结论已落进 HRN 决策 8 / HRN 02 §9 / Deterministic_Discovery_question §六 / 缺口地图。
→ 这是本窗口问题的一个**微缩版**：一个"系统自发语义发现"的子类已被判为人专属。本窗口核心要问：merge/split 与 FP-5 错误发现，是否也该照此裁成人专属，从而整个"系统自发发现层"被删空？还是其中哪类和 rule 失效本质不同、值得保留系统自发？

【必须先分清三个概念，别再混（HRN 早先就因为混这三个产生过跨文档冲突）】
- **确定性发现**：非 LLM 笨 job、套 Rule 侧**确定性判句**、只产 rule 升级候选。它做的是确定性判断、不是语义自发——**合法、不在本窗口裁撤范围**。
- **系统自发发现层**：系统自发做**语义**判断找问题（merge/split、错误发现）。**这才是本窗口要裁的**。
- 本窗口只裁"系统自发语义判断"那层，不碰确定性发现。

先读（按序）：
1. bookkeeper-Copilot/AGENTS.md、bookkeeper-Copilot/当前任务状态.md
2. 项目记忆（/Users/kingbayue/.claude/projects/-Users-kingbayue-Desktop-system-audit/memory/）：semantic-discovery-node-necessity-open、governance-layer-node-mechanism-division、terminology-judgment-clause-not-predicate
3. bookkeeper-Copilot/BK_Copilot/memory_layers/entity_log/02_*.md（entity 创建规则、merge_split_candidate、identity risk、automation）、alias_log/02_*.md
4. bookkeeper-Copilot/BK_Copilot/workflow_nodes/human_review_node/01_*.md + 02_*.md（决策 6/9/9b：只承接人发起 merge/split；决策 8 / §9：rule 失效人专属）；追溯 bookkeeper-Copilot/L2_proposals/HumanReviewNode__L2提案.md 决策 8
5. bookkeeper-Copilot/独立question文档/Deterministic_Discovery_question.md（确定性发现做什么、明确不做什么——用它划清"确定性 vs 语义自发"的界）
6. bookkeeper-Copilot/缺口地图.md 顶部「跨层共享机制」+ Entity Log section（merge/split 跨 log 重判 / 迁移）+ Coordinator section 中"身份冲突（疑似同名 / 疑似 merge / identity conflict）的发现与 merge/split 解决机制"那条（原挂 Post-Batch Lint / Governance Review，已废除）

背景与已锁定（别推翻）：
- 人发起的 merge/split 与人发起的纠错都已有归宿（Human Review，签字、过 Governance Log）。本窗口不议人发起那条。
- owner 的怀疑（正面回应、别绕）：entity 创建规则写得很死、每次写入都查是否已有相关记录，所以系统似乎不该需要自发去审计长期文档判断 merge/split；同理 rule 失效已裁为人专属。"系统自发凭什么"正是存疑点。
- 对照：确定性发现能放心自主，因为依据是明确、可审计的判句；语义自发是模糊判断，依据不明。

核心四问（对 merge/split 与 FP-5 一起裁）：
1. 究竟什么真实场景，会让一个已严格创建的长期对象（entity）事后需要 merge/split？又有什么真实场景，会让系统需要自发判定"某笔记账可能错了"（而不是等人在 review 发现）？给得出具体、可辩护的场景吗？
2. 若要让大模型自发发现，依据是什么？能否做到明确、可审计，而不是"模型觉得像"？
3. 其中哪些能确定性化（套判句、像确定性发现那样）从而根本不需要语义 LLM 自发节点？——含"疑似同名 / identity conflict"的发现。
4. 裁决（对 merge/split 与 FP-5 分别 + 合并给结论）：保留为自主语义节点 / 删除（只保留 Human Review 人发起）/ 降级为"确定性发现提候选 + 人确认"。给推荐 + 理由。**并明确回答：rule 失效已被裁成人专属，这个先例是否推广到 merge/split 与 FP-5？若推广，则整个"系统自发发现层"删空；若不推广，说清 merge/split 或 FP-5 哪里和 rule 失效本质不同、值得保留系统自发。**

若裁决保留，再谈融入：与 Entity Log merge_split_candidate、审核 inbox、Human Review 确认面、Governance Log、merge/split 后跨 log 重判 / 迁移机制如何衔接？会不会退化成"确定性发现 + 人确认"而无需独立语义节点？

协调：Human Review Node 已落正式草案；本窗口不改 HRN 文档，裁决与接缝以 finding / 待用户决定项交回。

纪律：`new system/` 已作废，不再作为审计目标；`new system/` 与 `old_system_nodedesign/` 未经用户明确授权不得读取，且无权威。不要因为旧 Post-Batch Lint / Governance Review 曾设计过 merge/split 或错误检测就认定它必要（已废除、本就无权威）；裁决锚定核心产品目标（记忆复用 / 审计性 / 会计师控制权 / 不让一次性判断污染长期记忆）。产出：必要性裁决报告（覆盖 merge/split 与 FP-5）+（若保留）融入方案；结论写成 finding / 待用户决定项，不静默改正式草案。
```
