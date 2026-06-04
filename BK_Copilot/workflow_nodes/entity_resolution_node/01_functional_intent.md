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
- accountant control（会计师控制权）要求 candidate（候选）不能自动变成 durable authority（长期权威）。
- automation rate（自动化率）只有在 identity authority（身份权威）清楚时才有价值，否则自动化会放大错误。

## 2. 核心职责

本节点的唯一核心职责是：

> 输出当前交易的 identity result（身份识别结果）和 identity authority annotations（身份权威边界说明）。

本节点可以辅助产生：

- `candidate_signal`（运行时候选信号），例如 Alias 写入需求、merge / split（合并 / 拆分）相关候选。
- `identity_risk_flags`（身份识别风险标记），例如名称相似、多对象竞争、历史误认或身份混用。
- `blocking_reason`（身份层阻断原因），说明为什么当前身份结果不能被当作更高 authority（权威）。

但这些不是主职责。

## 3. 明确排除范围

本节点不负责：

- rule match（规则匹配）。
- case judgment（案例判断）。
- COA / HST / GST treatment（会计科目 / 税务处理判断）。
- journal entry generation（分录生成）。
- accountant-facing question generation（面向会计师的问题生成）。
- transaction finalization（交易最终确认）。
- durable memory mutation（长期记忆变更），但 `new_stable_entity` entity 本体同步写入 Entity Log 是已确认例外。

本节点绝不能：

- 在未满足 stable 判断标准时把对象当作 `new_stable_entity`（新稳定实体）写入。
- 在 `new_stable_entity` 写入时同时写入 Alias（别名）、修改 automation policy（自动化策略）或创建 rule（规则）。
- 把未确认的 surface text 当作 Alias 使用。
- merge / split entity（合并 / 拆分实体）。
- 修改 `automation_policy`（自动化策略）。
- 创建、升级、修改、删除或降级 active rule（生效规则）。
- 写入 `Transaction Log`（交易审计日志）。
- 批准 governance event（治理事件）。

## 4. Workflow 位置

上游：

- `Evidence Intake / Preprocessing Node`（证据接收 / 预处理节点）：提供 traceable evidence foundation（可追溯证据基础）和 objective transaction basis（客观交易基础）。
- `Transaction Identity Node`（交易身份节点）：提供 `transaction_id`（稳定交易 ID）。
- `Profile / Structural Match Node`（客户结构匹配节点）：确认当前交易未被 structural path（结构性路径）完成。

下游：

- `Rule Match Node`（规则匹配节点）：读取身份结果和身份权威说明，自行判断 rule eligibility（规则匹配资格）。
- `Case Judgment Node`（案例判断节点）：读取身份上下文，自行判断是否可以 case-based judgment（基于案例判断）。
- `Coordinator / Pending Node`（协调 / 待确认节点）：读取身份卡点，自行生成 accountant-facing question（面向会计师的问题）。
- `Review Node`（审核节点）和 `Governance Review Node`（治理审核节点）：读取候选信号，但这些信号不是 approval（批准）。
- `Case Memory Update Node`（案例记忆更新节点）：只能在交易完成后结合 final outcome（最终结果）考虑候选信号。

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
- 通过 runtime candidate output（运行时候选输出）让系统学习有入口，但不污染 durable authority（长期权威）。
- 通过 `new_stable_entity` 的受限同步写入，让清楚、可追溯且无冲突的新对象在同 batch 后续交易中可被自然匹配。
- 通过受控 AI 联网搜索减少不必要的 accountant pending 问题；搜索只辅助判断“这是谁”，不能产生 accounting outcome（会计处理结果）或 authority（权威）。

Alias（别名）在本节点中的当前含义：

- Alias 是过去已经确认过的 transaction surface text 和 stable entity（稳定实体）的对应关系。
- 当前只确认两类信息可能成为 Alias：bank statement 中每笔交易的 description / descriptor / raw bank surface text；以及当 bank description 本身没有明确身份意义时，其他可能重复出现并能指向交易主体的字段，例如 cheque payee。
- 本节点可以查询 Alias 库：当它大致能判断当前交易主体但不能完全确定时，以该 entity 为目标查询是否存在一致 Alias；当它完全无法识别主体时，用当前 description 或等价 surface text 到 Alias 库中查询是否存在一致或高度类似的历史 Alias。
- 如果当前 surface text 命中已确认 Alias，可以复用该 Alias 指向的 stable entity。

## 6. 已知约束

- `transaction_id`（稳定交易 ID）必须来自 `Transaction Identity Node`（交易身份节点）。
- `evidence_refs`（证据引用）必须可追溯到 Evidence Log（证据日志）或 evidence foundation（证据基础）。
- `Entity Log`（实体日志）和 `Governance Log`（治理日志）优先于 Knowledge Summary（知识摘要）。
- 本节点判断为 `new_stable_entity`（新稳定实体）时，可以同步写入 Entity Log，不需要 governance approval（治理批准）；写入内容只限 entity 本体和最小创建 provenance。
- `candidate_signal`（候选信号）是 runtime-only（仅运行时）handoff，不是 durable authority（长期权威）。
- 未确认的 surface text 不能作为 Alias 使用。
- Entity Resolution 输出 unknown_entity 后，如果 accountant 在 Coordinator 交互中明确确认 identity，交易不重新进入本节点；该 accountant confirmation（会计师确认）替代本节点身份判断。
- `confidence`（身份识别置信度）只表示 identity confidence（身份置信度），不能表示 accounting classification confidence（会计分类置信度）。
- 本节点传给下游的 unknown / unresolved identity context 只能作为 runtime clue（运行时线索）和 pending 诊断材料，不能被 Case Judgment、Coordinator 或其他节点包装成已确认 identity authority。

Rule Match（规则匹配）在本节点下游的当前边界：

- Rule Match 接收本节点已经确认的 stable entity（稳定实体）结果，再判断当前交易是否满足 rule scope（规则适用范围）。
- Alias lookup（别名查询）发生在 Entity Resolution 阶段；Rule 的核心不是 Alias。
- Entity-level rule（实体级规则）围绕 stable entity 建立；pattern-level rule（模式级规则）围绕 stable entity 下更窄的稳定交易模式建立。
- 如果一个 entity 下存在多种最终会计分类结果，该 entity 本身不应升级成 entity-level rule，需要进入更窄 scope 选择。

## 7. 未决定问题

- `stable_entity_resolution_threshold`（稳定实体识别所需证据门槛）尚未冻结。
- `entity_resolution_output`（实体识别运行时输出）的 exact field schema（精确字段结构）尚未冻结。
- `new_stable_entity` 最小创建 provenance 的 exact field schema（精确字段结构）尚未冻结。
- `knowledge_summary_conflict_repair`（Knowledge Summary 与 Entity Log / Governance Log 冲突时的修复流程）尚未冻结。
