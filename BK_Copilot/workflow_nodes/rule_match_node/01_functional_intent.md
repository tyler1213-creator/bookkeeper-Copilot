# Rule Match Node - Functional Intent

## 1. 为什么这个节点必须存在

这个节点解决的问题是：

> RuleMatchNode 必须存在，是因为系统需要一个独立运行节点来应用已经批准的 deterministic rule。Entity Resolution 只回答当前交易指向哪个 StableEntity；CaseJudgment 处理没有可执行 active rule 的判断；JE Generator 只根据 accounting treatment 构造 JE。RuleMatchNode 位于这些对象之间，负责把 active approved rule 应用于当前交易并输出 accounting treatment。

如果删除、合并或内联它，会失去：

- 如果由 Entity Resolution 直接输出会计分类，ER 会从 identity node 膨胀为 rule application / accounting automation node，混淆 identity authority 与 rule authority。
- 如果把规则应用内联进 JE Generator，纯构造 / 校验层会被迫承担 rule applicability 判断和 rule hit trace。

这些能力对核心产品目标重要，因为：

- RuleMatchNode 把“这是谁”和“已批准规则如何应用”分开，使高自动化路径仍有清楚的 authority boundary。
- RuleMatchNode 让每次自动分类都能追溯到命中的 approved rule，而不是只留下最终 treatment。
- RuleMatchNode 保护 accountant control：active rule 的权威来自事前批准，而不是运行时推断或模型判断。

## 2. 核心职责

本节点的唯一核心职责是：

> 把已经由 accountant / governance 审批过、可以反复复用的 active rule 应用于当前交易，输出高权威（因 rule 已事前批准）的 accounting treatment 给 JE Generator。

RuleMatchNode 应用的 rule scope 只有两类：

- Entity-level rule：stable entity 本身足以唯一决定 accounting treatment。
- Pattern-level rule：同一 stable entity 下必须满足更窄客观条件，才能唯一决定 accounting treatment。

entity-level rule 是下文“Rule 资格的内容判据”在 condition set 为空时的退化情形；pattern-level rule 是需要非空客观条件集的一般情形。两者不是两套独立机制，而是同一资格判据的两种呈现。

以下为 RuleMatchNode 所消费的资格语义标准；其 authority 与最终归档归 RuleLog / Governance。RuleMatchNode 运行时不判定资格、不重算资格，只信任 RuleLog 中 active + approved 的规则：

- Scope 可由当前交易客观事实确定性判定。
- 该 scope 下历史 accounting treatment 唯一、无实质分叉。
- 输出是 judgment-free、可被 JE Generator 直接执行的完整 treatment。
- 已被 accountant / governance 事前批准。

次数 / 跨月 / 频率只是证据强度，不是资格门槛，不在本层冻结阈值。repeated cases、相似描述、LLM reasoning、CaseLog 聚合趋势、模糊自然语言不能直接成为 rule；它们只能进入 RuleLog / Governance 侧的 rule review path，本节点不处理。

本节点可以辅助产生：

- rule-hit context / trace，用于后续 finalization、audit、learning / review 追溯本次自动分类使用的 StableEntity ref 与 Rule ref。

但这些不是主职责。

## 3. 明确排除范围

本节点不负责：

- 重新判断 identity。
- 做 case-based judgment。
- 创建、升级、修改、删除或降级 active rule。
- 处理 rule promotion candidate 或 rule review candidate。
- 读取 CaseLog repeated outcome 来直接执行规则。
- JE 构造。
- TransactionLog finalization。
- 写 CaseLog。
- 创建或修改 RuleLog。

本节点不产出 candidate：无。

本节点绝不能：

- 把 ER 的 unknown 或 unknown reason 包装成 RuleMatch basis。
- 把 CaseLog 聚合趋势、AI reasoning、相似描述或模糊自然语言当作 active rule 执行。
- 在运行时重算“结果唯一 / 无分叉”等 rule 资格判据。
- 使用 LLM / AI 选择 COA、HST / GST、split、allocation 或 conflict winner。
- 直接写入 EntityLog、RuleLog、CaseLog、TransactionLog 或 GovernanceLog。
- 更新 rule match_count、rule health 或执行多 log finalization。

## 4. Workflow 位置

上游：

- Entity Resolution：提供 StableEntity 身份基础。
- ER 后 handoff assembler / workflow router：可以组装 RuleMatch handoff；由谁组装属于 L4 / seam。

下游：

- JE Generator：消费 `rule_hit` 的 approved accounting treatment 和 rule hit context。
- CaseJudgment：消费 `rule_miss`。
- 后续 finalization / audit / learning：消费 rule-hit handoff，决定 TransactionLog / CaseLog 等持久化路径。
- review / governance：消费 `invalid_handoff / blocked`。

本节点位于流程中的原因：

- Entity Resolution 只提供 identity basis，不拥有 rule authority。
- CaseJudgment 处理没有可执行 active rule 的交易，不应被 RuleMatch 的 miss 空转污染。
- JE Generator 负责根据已给定 treatment 构造并校验 JE，不应承担 rule applicability 判断。

## 5. 对核心产品目标的贡献

本节点必须清楚支持以下至少一项：

- [ ] 记忆复用
- [ ] 有证据支持的建议
- [ ] accountant correction learning
- [x] 审计性
- [x] accountant control
- [x] 自动化率提升

具体贡献：

- 自动化率提升：对已具备 StableEntity、放行控制状态和 active approved rule 的交易，直接应用 deterministic rule，减少重复人工判断。
- 审计性：每次自动分类都可经 rule-hit context 回到 StableEntity ref 与 Rule ref，说明本次 treatment 的 rule authority。
- accountant control：active rule 的可执行权威来自 accountant / governance 的事前批准；运行时 RuleMatchNode 不扩权、不学习新 rule、不放宽条件。

## 6. 已知约束

- rule authority 仅来自 RuleLog 中 active approved rule。
- RuleMatchNode 是 runtime-only 节点，不直接执行 durable write。
- RuleMatchNode 运行时不重算 rule 资格，只信任 RuleLog 中 active + approved 的规则。
- RuleMatchNode 只消费上游 handoff 中投影的 automation_policy / control state，不直接读取 EntityLog 判 eligibility。
- RuleMatchNode 不使用 LLM / AI；LLM 可判断项：无。
- RuleMatchNode 自身不产出 candidate；rule promotion / review candidate 属于 RuleLog / Governance 路径。

## 7. 未决定问题

- 见 `00_index.md` 的“当前未冻结边界”。
