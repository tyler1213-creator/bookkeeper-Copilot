# Entity Resolution Node - Functional Intent

## 1. 为什么这个节点必须存在

这个节点解决的问题是：

> 在非结构性交易进入 rule / case workflow（规则 / 案例流程）之前，先判断当前 evidence（证据）指向哪个 counterparty / vendor / payee（交易对手 / 商家 / 收款人），或者说明为什么不能安全确认。

如果删除、合并或内联它，会失去：

- identity gate（身份闸门）：系统会把“这是谁”混入 rule match（规则匹配）或 case judgment（案例判断）。
- authority separation（权限分离）：Alias（别名）和其他 entity 附属信息容易被下游误用。
- evidence-grounded identity trace（有证据支撑的身份追溯）：后续 review（审核）、correction（纠正）和 governance（治理）无法清楚知道身份判断来自哪些 evidence（证据）。
- safe new-entity handling（安全的新实体处理）：第一次出现的对象会被迫在旧 entity（实体）和普通分类判断之间被猜测处理。

这些能力对核心产品目标重要，因为：

- memory reuse（记忆复用）必须先知道当前交易指向哪个 entity（实体）。
- evidence-supported suggestion（有证据支持的建议）要求身份判断能回到 raw evidence（原始证据）。
- accountant control（会计师控制权）要求本节点的 runtime 身份输出（identity result + reason）不能自动变成 durable authority（长期权威）；merge / split、身份风险等治理判断只能由会计师人发起。
- automation rate（自动化率）只有在 identity authority（身份权威）清楚时才有价值，否则自动化会放大错误。

## 2. 核心职责

本节点的唯一核心职责是：

> 输出当前交易的 identity state（身份状态：`stable` 或 `unknown`），并为该判断附上可审计的 identity reason（含 `unknown` 时的 reason / context）。

本节点不产出与身份状态并行的候选 / 风险通道。原 `candidate_signal`、`merge_split_candidate`、`identity_risk_flags`、`blocking_reason` 已删除（2026-06-25 收口，理由见 §3 与 `02_logic_and_boundaries.md` §6）：身份层的冲突、歧义、缺证、alias 问题一律收敛为 `unknown` 的 reason / context；merge / split 与身份风险归会计师人发起 Human Review（身份风险并入 force_pending），本节点只读、不产。

## 3. 明确排除范围

本节点不负责：

- rule match（规则匹配）。
- case judgment（案例判断）。
- COA / HST / GST treatment（会计科目 / 税务处理判断）。
- journal entry generation（分录生成）。
- accountant-facing question generation（面向会计师的问题生成）。
- transaction finalization（交易最终确认）。
- durable memory write mechanism（长期记忆写入机制）；但 ER 判定新建 stable entity 后，必须在下游继续前发起同步 Entity Log + Alias Log finalization，且无需 governance approval（治理批准）。
- 产出 merge / split 候选，或任何与身份状态并行的候选 / 风险信号；merge / split 与身份风险归会计师人发起 Human Review（身份风险并入 force_pending），本节点只读相关 Entity Log 状态、不产候选。

本节点绝不能：

- 在未满足 stable 判断标准时把对象当作新建 stable entity 写入。
- 绕过统一 finalization 机制裸写 Alias（别名），或把未确认 / 未够格的 surface text 写成 Alias。
- 在新建 stable entity finalization 时夹带 force_pending / promotion_lock 修改或创建 rule（规则）。
- merge / split entity（合并 / 拆分实体）。
- 修改 `force_pending` / `promotion_lock`。
- 创建、升级、修改、删除或降级 active rule（生效规则）。
- 写入 `Transaction Log`（交易审计日志）。
- 批准 governance event（治理事件）。

## 4. Workflow 位置

上游：

- `Evidence Intake / Preprocessing Node`（证据接收 / 预处理节点）：提供 traceable evidence foundation（可追溯证据基础）和 objective transaction basis（客观交易基础），并为当前交易分配 `transaction_id`（稳定交易 ID）。
- `Profile / Structural Match Node`（客户结构匹配节点）：确认当前交易未被 structural path（结构性路径）完成。

下游：

- `Rule Match Node`（规则匹配节点）：读取身份结果和身份权威说明，自行判断 rule eligibility（规则匹配资格）。
- `Case Judgment Node`（案例判断节点）：读取身份上下文，自行判断是否可以 case-based judgment（基于案例判断）。
- `Coordinator / Pending Node`（协调 / 待确认节点）：读取身份卡点，自行生成 accountant-facing question（面向会计师的问题）。
- `Human Review Node`（人审节点，会计师人发起）与 Governance（授权确认 = Human Review 签字 + Finalization 凭证）：不是本节点的 runtime 下游消费者；merge / split 与身份风险纠正由会计师在此人发起，所需身份上下文从 Entity Log / Governance Log 读取，本节点不向其推送候选信号。

本节点位于流程中的原因：

- Profile（客户结构档案）先处理 internal transfer（内部转账）、loan repayment（贷款还款）等结构性交易。
- 非结构性交易进入 rule / case workflow（规则 / 案例流程）前，必须先知道 evidence（证据）指向谁。
- Rule（规则）和 Case（案例）都不应重新承担 identity resolution（实体识别）职责。

## 5. 对核心产品目标的贡献

本节点必须清楚支持以下至少一项：

- [x] 记忆复用
- [x] 有证据支持的建议
- [x] accountant correction learning
- [x] 审计性
- [x] accountant control
- [x] 自动化率提升

具体贡献：

- 通过 entity-first identity（实体优先身份识别）让不同表面写法可以汇入同一 stable entity（稳定实体）。
- 通过 evidence refs（证据引用）让身份判断可审计、可纠正。
- 通过 Alias / governance authority（别名 / 治理权威）防止模型用语义相似度越权。
- 通过新建 stable entity 的及时 Entity Log + Alias Log finalization，让清楚、可追溯且无冲突的新对象及其 confirmed surface text 在同 batch 后续交易中可被自然匹配。
- 通过受控 AI 联网搜索减少不必要的 accountant pending 问题；搜索只辅助判断“这是谁”，不能产生 accounting outcome（会计处理结果）或 authority（权威）。

Alias（别名）在本节点中的当前含义：

- Alias 是过去已经确认过的 transaction surface text 和 stable entity（稳定实体）的对应关系。
- 当前只确认两类信息可能成为 Alias：bank statement 中每笔交易的 description / descriptor / raw bank surface text；以及当 bank description 本身没有明确身份意义时，其他可能重复出现并能指向交易主体的字段，例如 cheque payee。
- 本节点可以查询 Alias 库：exact Alias match（完全命中）是确定性身份复用路径，可以直接复用该 Alias 指向的 stable entity。
- 非 exact / 高度类似的历史 Alias 只能作为 identity clue（身份线索）参与判断，不能天然等同于确认 identity。

## 6. 已知约束

- `transaction_id`（稳定交易 ID）必须来自 `Evidence Intake / Preprocessing Node`（证据接收 / 预处理节点）。
- `evidence_refs`（证据引用）必须可追溯到 Evidence Log（证据日志）或 evidence foundation（证据基础）。
- `Entity Log`（实体日志）和 `Governance Log`（治理日志）优先于 Knowledge Summary（知识摘要）。
- 本节点判断为新建 stable entity 时，state 仍为 `stable`；ER 自主决定、无需 governance approval，并必须在下游继续前发起同步 Entity Log + Alias Log finalization：Entity Log 创建 stable entity 本体、最小创建 provenance 和初始 entity-centered Alias surface，Alias Log 写入对应 confirmed surface text -> stable entity 的反查 projection。实际由 ER 直接写入还是同步调用专门写入 / 存储机制、两者写入顺序和幂等机制属于 L4 / seam。
- 未确认的 surface text 不能作为 Alias 使用。
- Entity Resolution 输出 `unknown` 后，如果 accountant 在 Coordinator 交互中明确确认 identity，交易不重新进入本节点；该 accountant confirmation（会计师确认）替代本节点身份判断。
- `confidence`（身份识别置信度）只表示 identity confidence（身份置信度），不能表示 accounting classification confidence（会计分类置信度）。
- 本节点传给下游的 `unknown` identity context 只能作为 runtime clue（运行时线索）和 pending 诊断材料，不能被 Case Judgment、Coordinator 或其他节点包装成已确认 identity authority。

Rule Match（规则匹配）在本节点下游的当前边界：

- Rule Match 接收本节点已经确认的 stable entity（稳定实体）结果，再判断当前交易是否满足 rule scope（规则适用范围）。
- Alias lookup（别名查询）发生在 Entity Resolution 阶段；Rule 的核心不是 Alias。
- Entity-level rule（实体级规则）围绕 stable entity 建立；pattern-level rule（模式级规则）围绕 stable entity 下更窄的稳定交易模式建立。
- 如果一个 entity 下存在多种最终会计分类结果，该 entity 本身不应升级成 entity-level rule，需要进入更窄 scope 选择。

## 7. 未决定问题

- stable identity 的语义判据已冻结；exact numeric threshold（精确数字门槛）、confidence cutoff（置信度切线）或评分规则如需存在，仍属 L3。
- `entity_resolution_output`（实体识别运行时输出）的 exact field schema（精确字段结构）尚未冻结。
- `new_stable_entity` 最小创建 provenance 的 exact field schema（精确字段结构）尚未冻结。
- external evidence reference（外部证据引用）的 exact field schema（精确字段结构）尚未冻结。
- 新建 stable entity 触发的 Entity Log + Alias Log 同步 finalization 的实际写入执行者、调用方式和写入顺序属于 L4 / seam，尚未冻结。
- `knowledge_summary_conflict_repair`（Knowledge Summary 与 Entity Log / Governance Log 冲突时的修复流程）尚未冻结。
