# Rule Match Node - Logic and Boundaries

## 1. 触发条件

本节点在以下条件同时满足时触发：

- ER handoff 提供 StableEntity。
- handoff 明确表明该 entity 的 automation_policy / control state 支持 RuleMatch。
- RuleLog 存在该 entity 的 active approved rule。

RuleMatchNode 不直接读取 EntityLog 来决定 eligibility。

本节点不得在以下情况触发：

- ER 输出非 stable，即 `unknown`。`unknown` 可能携带 ambiguous / unresolved / conflict 等 reason，但这些 reason 不是独立 identity 状态，也不是 RuleMatch basis。
- handoff 未表明 RuleMatch eligibility。
- handoff 缺少 RuleMatch 所需的身份基础、放行 / 控制基础或客观交易事实。

这些情况应转 CaseJudgment 或 pending / review 路径。

## 2. 上游前置条件

上游必须已经完成 RuleMatch handoff 的三桶语义：

- 桶 A：身份基础。必须提供 active stable entity 的 `entity_id`。merged entity 必须由上游按 supersession 跳到 surviving entity 后再给；archived / unknown 不组装此 handoff。不携带 stable reason、identity provenance 或 alias payload，因为 RuleMatch 不重判身份。
- 桶 B：放行 / 控制基础。必须提供投影自 EntityLog automation_policy 的 `automation_control_state` 语义，表达该 entity 是否放行 rule-based automation；authority 仍在 EntityLog。可选携带会 block rule automation 的身份级 risk / governance 限制指针，使 RuleMatch 能 fail-closed。
- 桶 C：客观交易事实。必须提供 transaction_id、direction、amount_abs、date 等客观维度，并携带 evidence_condition / evidence_refs、transaction_type / objective_tags、currency 等运行所需事实。raw description 只随 trace 携带，不作匹配条件。

exact 字段 schema 留 Stage 3；handoff 由谁组装留 Stage 4 / seam。

如果前置条件缺失：

- 本节点行为：输出 invalid_handoff / blocked 语义，或由上游转 CaseJudgment / pending / review。
- 是否 stop-and-ask：如果缺失来自未冻结的 product / contract 决策，停止并要求产品设计决策；不能由实现代理猜。

## 3. 读取对象

| Source | 读取内容 | 用途 | Authority 限制 |
| --- | --- | --- | --- |
| RuleLog | 凭 handoff 中的 `entity_id`，按 entity_id 索引读取该 stable entity 的候选 active approved 规则集；rule payload 来自 RuleLog | 用桶 C 客观交易事实逐条比对 active approved rule，找出至多一条命中；最终输出给 JE Generator 的分类 = 命中 rule 的 approved accounting treatment + 当前交易客观事实 | candidate rule 不能执行；RuleLog 是 rule payload 和 active approved status 的 source of truth |
| [不读] EntityLog | 不直接读取；automation_policy / RuleMatch eligibility 经 ER handoff 或等价 workflow handoff 投影进入 | 避免 RuleMatchNode 二次解释 entity eligibility / automation control | EntityLog 仍是 automation_policy / identity 的 source of truth，但不是本节点直接读取对象 |
| [不读] AliasLog | 不读取 alias payload 或 alias history | Alias 只通过 ER 输出的 StableEntity 间接受益 | Alias 不是 rule authority，也不是会计分类依据 |
| [不读] CaseLog | 不读取 repeated outcome、case-derived pattern 或 case history 来执行 rule | 防止运行时把 case evidence 升格为 active rule | CaseLog 可以为 rule promotion / review 提供历史依据，但不能直接作为 active rule 执行 |
| [不读] TransactionLog | 不读取历史交易审计记录来执行 rule | 防止从 finalized audit record 反向学习未治理 rule | TransactionLog 不是 RuleMatch runtime 的 rule authority |

RuleMatchNode 只读 RuleLog；RuleLog reader 的具体调用机制属于 L4 / seam。

## 4. 写入对象

### 直接执行的 durable write（例外）

本节点直接执行的 durable 写入：

- 无。

### 交给记忆 / finalization 层持久化的内容（运行 / 记忆 seam）

本节点产出、但由记忆 / finalization 层执行写入的内容：本节点只声明“存什么 + 谁有权威认定它有效”，不声明“怎么写、谁来写、什么顺序写”。

- RuleMatch 命中的交易应由后续 finalization 路径写入 TransactionLog。可持久化的语义包括当前交易事实、命中的 approved accounting treatment、StableEntity ref、Rule ref 与 `rule_match` source；有效性来自 RuleLog 的 active approved rule authority 与 finalization 路径对本次交易完成状态的认定。
- 若交易 stable-linked 且 finalized，则具备 CaseLog 写入资格。可持久化的是本次 finalized transaction 的 reusable case 语义；有效性来自 stable entity identity 与 finalized transaction outcome。

TransactionLog / CaseLog 的实际写入者、触发顺序、多 log finalization、Rule 使用次数、rule hit count、rule health 和维护统计机制均属 L4 / seam 或 RuleLog / Governance 机制。

### 只能提出 candidate

- 无。本节点不产出 candidate。

### 绝不能写入或修改

- EntityLog。
- RuleLog。
- CaseLog。
- TransactionLog。
- GovernanceLog。
- rule match_count / rule health。
- 多 log finalization 状态。

## 5. 决策权限

### Deterministic code 可以决定

- 当前 handoff 是否同时具备 StableEntity、RuleMatch 放行控制语义和客观交易事实。
- 按桶 C 与 active approved rule 中已批准客观条件做确定性比对。
- 命中数量为 0 / 1 / 大于等于 2 时，分别输出 rule_miss / rule_hit / invalid_handoff-blocked。
- 在运行时假定同一 entity 下 active rule 互不重叠，这是治理 / RuleLog 的 write-time invariant。

Deterministic code 不消解重叠、不裁剪条件、不合并 rule、不选择 winner。

同一 stable entity 下可并存多条 active rule，但各 rule 条件必须客观、可判定、两两互不重叠，任一交易最多命中一条。entity-level rule 与区分性 pattern-level rule 在同一 entity 上互斥；“默认 + 例外”应建模为多条互斥 pattern-level rule（可含 catch-all），不建模为 entity-level + 例外。

### LLM 可以判断

- 无。本节点不使用 LLM / AI。

### LLM 不能判断

- COA / HST / GST / split / allocation。
- 补全、解释、放宽或裁剪 rule condition。
- 在多个 rule 中选择 winner。
- 把 case history、AI reasoning、相似描述或 CaseLog 聚合趋势升格为 active rule。
- 运行时重算“结果唯一 / 无分叉”等 rule 资格。
- rule applicability、accounting treatment 或 conflict winner。

### Accountant 必须决定

- 运行时无。
- rule 的事前批准、rule lifecycle / promotion / 改删降级不在本节点运行时发生，归 RuleLog / Governance；见 Open Boundaries。

### Governance 必须批准

- 运行时无。
- rule promotion、rule lifecycle、rule modification / deletion / downgrade、rule health 或 staleness 处理归 RuleLog / Governance 外阻；见 Open Boundaries。

## 6. 输出类别

字段名可以暂不冻结，但语义类别必须稳定。

本表是本节点对下游唯一的契约面：下游只能依赖此处声明的输出类别，不得依赖本节点未声明的内部状态或实现。

| Output Category | 含义 | Consumer（谁消费） | 下游影响 | 不代表什么 |
| --- | --- | --- | --- | --- |
| `rule_hit` | 当前交易命中一条 active approved rule。输出 approved accounting treatment 与 rule hit context；最终分类 = 命中 rule 的 approved treatment + 桶 C 交易事实。桶 C 必要但不充分，分类“答案”来自 RuleLog。该输出是高权威（因 rule 已事前批准）的分类来源。 | JE Generator；后续 finalization / audit / learning | JE Generator 用 approved accounting treatment + 当前交易事实构造并校验 JE。后续 finalization / audit / learning 消费 rule-hit handoff 追溯 StableEntity ref 与 Rule ref。 | 不代表 JE 已成功生成、TransactionLog 已 finalized、CaseLog 已写入、accountant 已完成本次交易最终审核，或本节点已持久化任何记录。 |
| `rule_miss` | 当前交易关联支持 RuleMatch 的 StableEntity，但不满足任何适用 active approved rule。 | CaseJudgment | 交易进入 CaseJudgment；本节点不继续做 case-based judgment。 | 不代表 identity unknown，不代表 rule 可以由本节点现场生成，不代表本节点可以读取 CaseLog 继续判断。 |
| `invalid_handoff / blocked` | 大于等于 2 条互斥 rule 同时命中，或出现 authority 不一致、eligibility handoff 与 RuleLog 冲突等异常信号。 | review / governance | fail-closed，交 review / governance；本节点不设 priority，不使用 LLM、confidence 或 fallback 选 winner。 | 不代表常规 routing 分支，不代表本节点能修复 RuleLog / EntityLog / handoff authority，也不代表自动化可以继续。 |

`rule_hit` 输出必须包含最小 rule-hit handoff 语义：

- StableEntity ref。
- Rule ref。
- approved accounting treatment。
- `rule_match` source。

该 handoff 只作为 runtime handoff 交给后续 finalization / audit / learning / review，不由 RuleMatchNode 持久化。exact field schema、refs 形态和 enum 留 Stage 3。

## 7. 证据不足时的行为

如果缺少适用 active approved rule：

- 输出：`rule_miss`。
- 下游应：进入 CaseJudgment。
- 本节点不能：猜测分类、读取 CaseLog repeated outcome、调用 LLM / AI、生成 rule candidate 或放宽 rule condition。

如果缺少可执行 rule payload、RuleLog active approved status 或 handoff 客观事实：

- 输出：`invalid_handoff / blocked`，或由上游转 pending / review。
- 下游应：review / governance 或上游 contract 修正。
- 本节点不能：补全 rule payload 或用桶 C 自行产出分类。

## 8. 歧义处理

如果同一交易在同一 stable entity 下大于等于 2 条 rule 同时命中：

- 输出：`invalid_handoff / blocked`。
- 是否允许自动化：不允许。
- 是否需要 pending / review / governance：交 review / governance，fail-closed。

本节点不能为了让 workflow 继续而猜一个 winner，不能设 runtime priority，不能使用 LLM / AI 或 confidence 解除歧义。

## 9. 冲突处理

在正常治理 invariant 下，不应出现以下状态：

- EntityLog 标记支持 RuleMatch，但 RuleLog 无 active rule。
- 多条互斥 rule 同时命中。
- RuleLog 与 eligibility handoff 冲突。
- handoff 中 authority refs 与 RuleLog active approved status 不一致。

如果出现这类 source conflict：

- authority 顺序：RuleLog 是 rule payload 与 active approved status 的 authority；EntityLog 是 automation_policy / identity 的 source of truth，但只经 handoff 投影进入本节点。
- 本节点行为：视为 invalid authority state / invalid handoff，保守输出 `invalid_handoff / blocked` 或交 review。
- 是否生成 review / governance candidate：本节点不产出 candidate；异常可由 review / governance 路径消费。
- 是否阻断自动化：阻断自动化。

本轮不为低概率异常设计复杂 conflict resolution，不允许本节点猜测或由 LLM / AI 选择。

## 10. Audit / Trace 边界

本节点应保留的 trace：

- StableEntity ref。
- Rule ref。
- approved accounting treatment。
- `rule_match` source。
- 当前交易客观事实引用，例如 transaction_id、direction、amount、date、currency、evidence refs。
- invalid_handoff / blocked reason，如多 rule 命中或 authority 不一致。

这些 trace 用于：

- finalization。
- review。
- correction。
- governance。
- audit。
- learning / rule review context。

RuleMatchNode 的可追溯性来自 rule-hit handoff：后续 TransactionLog / CaseLog / Governance review 应能通过其中的 StableEntity ref 与 Rule ref 还原本次自动分类使用的 rule authority。

这些 trace 不能成为：

- rule authority。
- entity authority。
- case authority。
- accountant approval。
- governance approval。

RuleMatchNode 不输出或维护命中次数、rule health 或统计值。这些统计不能成为本节点的 authority，也不属于本节点 L2。

## 11. Legacy Constraint Translation

仅记录脱离来源、按当前产品目标重新论证后的取舍；旧系统材料只作审计对象，不构成 authority。

可借鉴保留的内核：

| 借鉴内核 | 旧系统来源 | 现在为何仍成立 |
| --- | --- | --- |
| “分类一致 / 出现第二种分类即 non_promotable” | observations 升级条件 | 即资格判据“结果唯一、无分叉”，提升为第一性判据 |
| “防偶发”的证据意图（大于等于 3 次 / 跨大于等于 2 月背后的目的） | rules / observations 升级硬条件 | 保留其作为证据强度的角色，但降级为证据、不继承为门槛，阈值下放 L3 / 治理 |
| accountant 可零积累主动建规则 | rules 来源二 | 反证次数非本质：资格看可判定 + 唯一 + 已批准，不看积累量 |
| active rule 权威只来自事前批准 | “only via approved paths 才能建 rule” | 保留精神：active rule 必须事前批准；执行机制归 RuleLog / Governance 外阻 |

明确不继承的旧行为：

| 旧行为 | 不继承原因 |
| --- | --- |
| pattern-centered exact description match | scope key 改为 stable entity；description 退回 ER / Alias 做身份原料，避免身份与表面文本耦合脆弱、易误聚、职责错位 |
| Observation 写入时破坏性聚合 count++ / classification_history 就地累加 | 已被 Case Log 决策点 4 否决：逐笔保真 + 派生聚合，非破坏、可重算 |
| Onboarding 历史数据满足硬条件即自动升级 active rule、无二次确认 | 违背 active rule 必须事前批准；即便来自历史账本，rule-level 晋升仍须走治理批准 |
| rule 只有“存在 / 删除”二态、无过期清理的贫化生命周期 | 不作为约束；rule lifecycle / health 归 RuleLog / Governance 外阻 |

## 12. Open Boundaries

以下问题未冻结：

1. automation_policy 中“哪种取值 = 放行 rule-based automation”的 exact 取值集合 / 语义尚未冻结；归 L3，需 RuleLog / EntityLog 联合对齐 Governance。
2. RuleMatch 输入 handoff 的 exact 字段 schema 尚未冻结；归 L3。handoff 由谁组装属于 L4 / seam。
3. RuleLog reader 的调用机制属于 L4 / seam。RuleLog 必须支持按 `entity_id` 索引，且每条 rule 承载 applicability；这是对 RuleLog 的前置约束，待 RuleLog 设计落定。
4. rule 资格判据的最终归档之家尚未冻结，取决于 RuleLog M1-M2；本节点只声明自己消费该资格语义标准。
5. rule promotion 发现侧尚未冻结：确定性发现 job 当前只服务 Rule Match / Rule 侧 promotion，具体阈值判据、CaseLogEvidence 打包、审核 inbox 写入和 rule 升级固定执行路径归 RuleLog / RuleMatch 治理；本节点 runtime 不产 candidate、不维护统计。
6. rule 集合 overlap-validation 算法属于 L4 / seam；catch-all pattern 是否作为 schema 语法糖属于 L3。
7. entity-level / pattern-level rule 的 exact schema、condition enum、accounting treatment（judgment-free 完整）schema、资格阈值具体数字尚未冻结，归 L3 / JE Generator。
8. JE Generator 接口尚未冻结；treatment “judgment-free 完整”的判据依赖 JE Generator 需要什么，属于跨节点 seam。

这些问题解决前，不能进入：

- [x] Stage 3 data contract
- [x] Stage 4 execution algorithm
- [x] implementation
