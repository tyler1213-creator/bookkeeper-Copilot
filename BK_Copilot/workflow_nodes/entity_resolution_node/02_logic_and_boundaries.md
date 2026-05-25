# Entity Resolution Node - Logic and Boundaries

## 1. 触发条件

本节点在以下条件同时满足时触发：

- Evidence Intake / Preprocessing（证据接收 / 预处理）已经形成 traceable evidence foundation（可追溯证据基础）。
- Transaction Identity（交易身份）已经分配或复用稳定 `transaction_id`（稳定交易 ID）。
- Profile / Structural Match（客户结构匹配）已经确认当前交易没有被 structural path（结构性路径）完成。
- 当前非结构性交易需要知道 evidence（证据）指向哪个 counterparty / vendor / payee（交易对手 / 商家 / 收款人），或为什么不能安全确认。

本节点不得在以下情况触发：

- `structural_path_status`（结构性路径状态）表示交易已经被结构性路径完成。
- `transaction_id`（稳定交易 ID）缺失或不稳定。
- `evidence_refs`（证据引用）缺失或不可追溯。
- 当前任务是 onboarding initialization（初始化历史学习）、review approval（审核批准）、governance approval（治理批准）或 transaction logging（交易审计记录）。

## 2. 上游前置条件

上游必须已经完成：

- Evidence Intake / Preprocessing Node（证据接收 / 预处理节点）：提供 objective transaction basis（客观交易基础）和 evidence refs（证据引用）。
- Transaction Identity Node（交易身份节点）：提供 `transaction_id`（稳定交易 ID）。
- Profile / Structural Match Node（客户结构匹配节点）：提供 non-structural handoff（非结构性交易交接）。

如果前置条件缺失：

- 本节点行为：拒绝把输入当作有效 identity request（身份识别请求），输出 invalid handoff / blocked handoff（无效交接 / 被阻断交接）语义。
- 是否 stop-and-ask：如果缺失来自未冻结的跨节点 contract（跨节点契约），停止并要求产品设计决策；不能由实现代理猜。

## 3. 读取对象

| Source | 读取内容 | 用途 | Authority 限制 |
| --- | --- | --- | --- |
| Evidence Log（证据日志） / evidence foundation（证据基础） | `evidence_refs`（证据引用）、raw bank text（银行原始文本）、receipt vendor（小票商家）、cheque payee（支票收款人）、invoice party（发票主体）、contract party（合同主体）、accountant context refs（会计师上下文引用） | 识别当前 evidence（证据）表面指向谁 | Evidence（证据）不保存业务结论；模型摘要不能替代 evidence ref（证据引用） |
| Transaction Identity layer（交易身份层） | `transaction_id`（稳定交易 ID）和 identity handoff（身份交接） | 保证本次身份判断绑定到稳定交易对象 | 交易身份不等于 entity identity（实体身份），也不等于 accounting authority（会计权威） |
| Profile / Structural Match handoff（客户结构匹配交接） | `structural_path_status`（结构性路径状态）、non-structural reason（非结构性原因）、profile context refs（客户结构上下文引用） | 防止结构性交易穿透后被普通实体流程处理 | candidate profile fact（候选客户结构事实）不能当 stable profile truth（稳定客户结构事实） |
| Entity Log（实体日志） | known entities（已知实体）、aliases（别名）、roles（角色）、`entity_status`（实体生命周期状态）、authority metadata（权威元数据）、risk flags（风险标记） | 判断 evidence（证据）是否安全指向稳定 entity（实体） | Summary（摘要）不能覆盖 Entity Log（实体日志）；candidate entity（候选实体）不能当 stable entity（稳定实体） |
| Governance Log（治理日志） | approved / applied alias, role, merge / split, lifecycle, policy constraints（已批准 / 已生效的别名、角色、合并 / 拆分、生命周期、策略限制） | 限制或解释当前身份判断 | pending / rejected governance event（待批准 / 已拒绝治理事件）不能当 positive authority（正向权威） |
| Intervention Log（人工介入日志） | identity correction（身份纠正）、accountant confirmation（会计师确认）、recent dispute（近期争议） | 识别当前身份风险或限制 | 人工介入记录只有明确确认对象时才可作为对应 authority（权威）；不能泛化 |
| Knowledge Summary（知识摘要） | readable identity notes（可读身份说明）、known ambiguity notes（已知歧义说明） | 辅助理解客户语境 | 不能替代 Entity Log（实体日志）或 Governance Log（治理日志） |

## 4. 写入对象

本节点可以直接写入：

- 无。

本节点只能提出 candidate：

- `new_entity_candidate`（新实体候选）。
- `alias_candidate`（别名候选）。
- `role_confirmation_candidate`（角色确认候选）。
- `merge_split_candidate`（实体合并 / 拆分候选）。
- `identity_ambiguity_issue`（身份歧义问题）。
- `unresolved_identity_issue`（身份无法识别问题）。
- `rejected_alias_conflict_issue`（已拒绝别名冲突问题）。
- `identity_governance_issue`（身份相关治理问题）。

本节点绝不能写入或修改：

- `Transaction Log`（交易审计日志）。
- stable `Entity Log` authority（稳定实体日志权威）。
- `Case Log`（案例日志）。
- `Rule Log`（规则日志）。
- `Governance Log`（治理日志）。
- `Profile`（客户结构档案）。
- active rule（生效规则）。
- `automation_policy`（自动化策略）。

## 5. 决策权限

### Deterministic code 可以决定

- 本节点是否应被触发。
- 输入是否具备 `transaction_id`（稳定交易 ID）、objective transaction basis（客观交易基础）和 traceable `evidence_refs`（可追溯证据引用）。
- 当前交易是否已经被 structural path（结构性路径）完成；若已完成，本节点不得继续。
- `alias_status`（别名权威状态）、`entity_status`（实体生命周期状态）、confirmed role（已确认角色）、governance constraint（治理限制）等 authority check（权威检查）是否满足。
- 哪些候选或风险只能作为 runtime handoff（运行时交接），不能成为 durable memory（长期记忆）。

### LLM 可以判断

- messy evidence（杂乱证据）中的 vendor / payee / counterparty 表面写法是否语义上接近某个已知 entity（实体）。
- receipt（小票）、cheque（支票）、invoice（发票）、contract（合同）或 accountant note（会计师备注）中有哪些 human-visible identity clues（人类可见身份线索）。
- 当前 evidence（证据）更像 stable entity（稳定实体）、new entity candidate（新实体候选）、ambiguous candidates（歧义候选）还是 unresolved identity（无法识别身份）。
- identity rationale（身份判断理由）、ambiguity reason（歧义原因）和 missing evidence reason（缺失证据原因）的可读摘要。

### LLM 不能判断

- 扩大 entity / alias / role authority（实体 / 别名 / 角色权威）。
- 把 `candidate_alias`（候选别名）当作 `approved_alias`（已批准别名）。
- 忽略 `rejected_alias`（已拒绝别名）。
- 确认 `candidate_role`（候选角色）。
- 创建 stable entity（稳定实体）。
- merge / split entity（合并 / 拆分实体）。
- 修改 `automation_policy`（自动化策略）。
- 执行 rule match（规则匹配）。
- 判断 case precedent（历史案例先例）是否足以支持分类。
- 选择 COA / HST / GST treatment（会计科目 / 税务处理）。
- 批准 governance event（治理事件）。
- 写入 `Transaction Log`（交易审计日志）。

### Accountant 必须决定

- 是否确认 role / context（角色 / 上下文）。
- 是否接受当前交易的最终 accounting outcome（会计处理结果）。
- 是否把当前交易 clarification（澄清）提升为 durable customer memory（长期客户记忆）。
- 模糊自然语言回答是否表达长期规则或只是当前交易说明。

### Governance 必须批准

- approve / reject alias（批准 / 拒绝别名）。
- confirm role（确认角色）。
- create stable entity authority（创建稳定实体权威）。
- merge / split entity（合并 / 拆分实体）。
- archive / reactivate entity（归档 / 重新激活实体）。
- upgrade or relax `automation_policy`（升级或放宽自动化策略）。
- create / promote / modify / delete / downgrade active rule（创建 / 升级 / 修改 / 删除 / 降级生效规则）。

## 6. 输出类别

字段名可以暂不冻结，但语义类别必须稳定。

| Output Category | 含义 | 下游影响 | 不代表什么 |
| --- | --- | --- | --- |
| `resolved_entity`（已识别稳定实体） | 当前 evidence（证据）可以安全指向一个 active stable entity（有效稳定实体） | 下游可读取 identity basis（身份基础）；Rule Match（规则匹配）仍需自行检查 alias、role、policy、rule | 不代表 rule match（规则匹配）成功，不代表会计分类，不代表自动化许可 |
| `resolved_entity_with_unconfirmed_role`（已识别实体但角色未确认） | 对象本身可识别，但当前交易所需 role / context（角色 / 上下文）未被 accountant-confirmed（会计师确认） | 下游可把实体作为上下文，但必须尊重 role authority gap（角色权威缺口） | 不代表 confirmed role（已确认角色），不支持需要该角色的 rule match（规则匹配） |
| `new_entity_candidate`（新实体候选） | 当前 evidence（证据）更像一个尚未稳定存在的新对象 | Case Judgment（案例判断）可在自身 authority boundary（权限边界）内读取；Review / Governance 可后续处理候选 | 不代表 stable entity（稳定实体），不支持 rule match（规则匹配），不支持 rule promotion（规则升级） |
| `ambiguous_entity_candidates`（多实体歧义候选） | 当前 evidence（证据）可能对应多个 entity / alias / role（实体 / 别名 / 角色），不能安全归属 | 下游应进入 pending / review / governance 相关处理，或保守阻断自动化 | 不代表 LLM 可以选一个 winner（胜出者） |
| `unresolved`（无法识别身份） | 当前 evidence（证据）不足以判断 counterparty / vendor / payee（交易对手 / 商家 / 收款人） | 下游应看到缺什么、为什么不能识别。**已确认：Case Judgment 收到此输出后不走高置信度自动分类通道，输出 pending（见下方已确认下游影响）** | 不代表会计分类失败的总称，不允许伪造 entity（实体） |
| `candidate_signal`（运行时候选信号） | 指出后续可能需要处理的新实体、别名、角色、合并 / 拆分或身份治理问题 | 只供 Coordinator / Review / Case Memory Update / Governance Review 消费 | 不代表 durable approval（长期批准），不写入长期记忆 |

## 7. 证据不足时的行为

如果缺少 identity signal（身份信号）：

- 输出：`unresolved`（无法识别身份）或 `unresolved_identity_issue`（身份无法识别问题）。
- 下游应：由 Coordinator / Pending Node（协调 / 待确认节点）决定是否向 accountant（会计师）补问，或由 Review Node（审核节点）处理。
- 本节点不能：为了让 workflow（流程）继续而伪造 entity（实体）。

### 已确认的下游影响（Problem 2 结论）

当本节点输出 `unresolved`（无法识别身份）时：

- Case Judgment Node（案例判断节点）不走高置信度自动分类通道，输出 pending。
- 但 Case Judgment 仍然处理这笔交易，pending 内部分两种情况：
  - **完全无法判断**：除 entity 未识别外，也没有其他足够上下文支持推断。由 Coordinator 直接向 accountant 提问。
  - **有推断但不确定**：Case Judgment 可利用其他已有上下文（如 receipt items、交易金额模式、bank descriptor 特征等）给出推断性建议，附在 pending 输出中，由 Coordinator 呈现给 accountant 选择或确认。
- 存在不依赖 entity identity 就能判断 COA 的交易类型（bank fee、interest 等），但这些属于 edge case，大部分由上游 Profile / Structural Match Node 处理。到达 Case Judgment 的交易绝大多数需要 entity identity。
- A/B 分类判断（是否需要 entity 才能分类）的职责在 Case Judgment，不在本节点。本节点只管 identity。

如果缺少 traceable evidence ref（可追溯证据引用）：

- 输出：invalid handoff / blocked handoff（无效交接 / 被阻断交接）语义。
- 下游应：返回 Evidence Intake / Preprocessing（证据接收 / 预处理）或停止设计讨论。
- 本节点不能：使用不可追溯摘要作为 identity authority（身份权威）。

## 8. 歧义处理

如果存在多个合理候选：

- 输出：`ambiguous_entity_candidates`（多实体歧义候选）。
- 是否允许自动化：本节点不决定自动化；但不得把歧义输出包装成 stable identity（稳定身份）。
- 是否需要 pending / review / governance：由 Coordinator / Pending（协调 / 待确认）、Review（审核）或 Governance Review（治理审核）根据卡点决定。

本节点不能为了让 workflow 继续而猜一个 winner（胜出实体）。

## 9. 冲突处理

如果 current evidence（当前证据）与 Entity Log（实体日志）或 Governance Log（治理日志）冲突：

- authority 顺序：Governance Log（治理日志）/ Entity Log（实体日志）中的 approved / applied authority（已批准 / 已生效权威）优先于 Knowledge Summary（知识摘要）和 LLM semantic match（LLM 语义匹配）。
- 本节点行为：保守输出 conflict / blocked / ambiguous identity（冲突 / 阻断 / 歧义身份）语义。
- 是否生成 review / governance candidate：可以生成 `identity_governance_issue`（身份相关治理问题），但不能批准。
- 是否阻断自动化：本节点只说明 identity authority problem（身份权威问题）；下游决定 automation path（自动化路径）。

如果 current evidence（当前证据）命中 `rejected_alias`（已拒绝别名）：

- authority 顺序：rejection（拒绝记录）优先于相似度。
- 本节点行为：输出 `rejected_alias_conflict_issue`（已拒绝别名冲突问题）或 blocked identity（被阻断身份）语义。
- 是否生成 review / governance candidate：可以。
- 是否阻断自动化：不得输出 stable identity basis（稳定身份基础）来支持 deterministic path（确定性路径）。

如果 accountant context（会计师上下文）与 durable memory（长期记忆）冲突：

- authority 顺序：明确 accountant confirmation（会计师确认）只在其确认范围内有效；模糊说明不能泛化成 durable authority（长期权威）。
- 本节点行为：输出 conflict reason（冲突原因）和 relevant refs（相关引用）。
- 是否生成 review / governance candidate：可以。
- 是否阻断自动化：如果冲突影响当前身份是否安全，不能输出 stable identity basis（稳定身份基础）。

## 10. Audit / Trace 边界

本节点应保留的 trace：

- `transaction_id`（稳定交易 ID）。
- `evidence_used`（用于身份判断的证据）。
- `matched_alias`（命中的表面写法 / 别名）。
- `alias_status`（别名权威状态）。
- `candidate_role`（候选角色），如果存在。
- `blocking_reason`（身份层阻断原因），如果存在。
- `identity_risk_flags`（身份识别风险标记），如果存在。
- `governance_constraint_refs`（治理限制引用），如果存在。
- `intervention_context_refs`（人工介入上下文引用），如果存在。

这些 trace 用于：

- review（审核）。
- correction（纠正）。
- governance（治理）。
- audit（审计）。

这些 trace 不能成为：

- entity authority（实体权威）。
- rule authority（规则权威）。
- case authority（案例权威）。
- accountant approval（会计师批准）。
- governance approval（治理批准）。

## 11. Legacy Constraint Translation

仅保留仍服务当前产品目标的旧系统约束。

| Retained Constraint | 来源 | 为什么现在仍成立 |
| --- | --- | --- |
| Transaction Log（交易审计日志）不参与 runtime decision（运行时决策） | 旧系统 Transaction Log 边界 | 身份识别必须基于 current evidence（当前证据）和 active authority（当前有效权威），不能从最终历史审计记录中反向学习未治理结论 |
| JE Generation（分录生成）应是纯计算层 | 旧系统 JE generator 边界 | Entity Resolution（实体识别）只回答“是谁”，不应提前做会计分录或税务判断 |
| PENDING（待确认）必须能转成人能回答的问题 | 旧系统 Coordinator 意图 | 本节点只暴露身份卡点；问题生成应由 Coordinator / Pending Node（协调 / 待确认节点）完成 |

不保留的旧行为：

| Old Behavior | 不保留原因 |
| --- | --- |
| 用 canonical description / pattern（标准描述 / 模式）作为学习链条核心身份 | 新系统的 identity source（身份来源）应是 evidence-grounded entity（有证据支撑的实体），pattern（模式）只能作为表面信号或展示字段 |
| 用 repeated outcome（重复结果）直接靠近 rule authority（规则权威） | 当前系统要求 rule promotion（规则升级）经过 accountant / governance approval（会计师 / 治理批准） |

## 12. Open Boundaries

以下问题未冻结：

1. `stable_entity_resolution_threshold`（稳定实体识别所需证据门槛）。
2. `entity_resolution_output`（实体识别运行时输出）的 exact field schema（精确字段结构）。
3. `same_batch_retrigger`（会计师补充确认后是否同批次重跑本节点）。
4. `unconfirmed_role_routing`（角色/上下文未确认时下游路由）。
5. `knowledge_summary_conflict_repair`（Knowledge Summary 与 Entity Log / Governance Log 冲突时的修复流程）。

这些问题解决前，不能进入：

- [x] Stage 3 data contract
- [x] Stage 4 execution algorithm
- [x] implementation
