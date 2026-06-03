# Entity Log - Memory Intent

## 1. 为什么这层 memory 必须存在

它解决的问题是：

> 为客户账务语境中的 counterparty / vendor / payee（交易对手 / 商家 / 收款人）保存 durable identity authority（长期身份权威），让 runtime nodes（运行时节点）能安全判断“这是谁”，而不用把身份、分类、规则和审计记录混在一起。

如果删除它，会失去：

- stable identity authority（稳定身份权威）：`Entity Resolution Node`（实体识别节点）无法判断当前 evidence（证据）是否指向已知稳定实体。
- Alias authority（别名权威）：系统无法复用过去已经确认过的 transaction surface text 到 stable entity（稳定实体）的对应关系。
- lifecycle control（生命周期控制）：系统无法处理 stable entity（稳定实体）的 active（有效）、merged（已合并）、archived（已归档）等状态。
- automation boundary（自动化边界）：系统无法在 entity level（实体级）限制 rule match（规则匹配）或 case-based automation（基于案例的自动化）。

这些能力不能由 runtime handoff（运行时交接）或现有 store（现有存储）覆盖，因为：

- runtime handoff（运行时交接）只表达当前交易上下文，不能作为未来交易可复用权威。
- `Transaction Log`（交易日志）是 audit-facing final record（面向审计的最终记录），不参与 runtime decision（运行时决策）。
- `Case Log`（案例日志）保存 entity-indexed learning memory（按实体索引的学习记忆），不保存 identity authority（身份权威）。
- `Rule Log`（规则日志）保存 approved deterministic rules（已批准确定性规则），不回答 entity identity（实体身份）。
- `Knowledge Summary`（知识摘要）是 readable context（可读上下文），不能替代 source authority（来源权威）。

## 2. 保存什么

这层 memory 保存：

- stable entity record（稳定实体记录）；Entity Log 不保存 durable candidate entity lifecycle state（长期候选实体生命周期状态）。
- `entity_id`（实体唯一标识）。
- `display_name`（展示名称）。
- `entity_type`（实体类型），例如 vendor（商家）、payee（收款人）、counterparty（交易对手）、person（个人）、organization（组织）。
- `aliases`（别名 / 表面写法）：过去已经确认过的 transaction surface text 和 stable entity 的对应关系。
- `entity_status`（实体生命周期状态）。
- `authority_metadata`（权威来源元数据）。
- `evidence_links`（证据链接），指向支持身份、别名或状态的 evidence（证据）。
- `risk_flags`（身份或自动化风险标记）。
- `automation_policy`（自动化策略）。
- `governance_refs`（治理引用），指向批准、拒绝、降级、合并、拆分或策略变更的治理事件。

其中可以成为 reusable authority（可复用权威）的是：

- `entity_id`（实体唯一标识）与 active entity identity（有效实体身份）。
- Alias：已确认 Alias surface text 到 stable entity 的对应关系。
- `entity_status`（实体生命周期状态）。
- `automation_policy`（自动化策略）。
- 已生效的 merge / split projection（合并 / 拆分投影），例如 merged entity（已合并实体）指向 surviving entity（保留实体）。

只作为 trace / context / explanation（追溯 / 上下文 / 解释）的是：

- `evidence_links`（证据链接）。
- `authority_metadata`（权威来源元数据）。
- `governance_notes`（治理备注）。
- `risk_flags`（风险标记）的解释文本。
- historical display text（历史展示文本）。
- rejected or superseded context（已拒绝或被替代上下文）。

Alias 当前定义：

- Alias 是过去已经确认过的 transaction surface text 和 stable entity 的对应关系。
- 当前只确认两类信息可能成为 Alias：bank statement 中每笔交易的 description / descriptor / raw bank surface text；以及当 bank description 本身没有明确身份意义时，其他可能重复出现并能指向交易主体的字段，例如 cheque payee。
- Alias 不是所有历史 description 的集合。只有已经和某个明确 stable entity 建立过确认关系的 surface text，才属于 Alias。
- 当前确认需要一个可被 Entity Resolution 查询的 Alias 库，用于从当前交易的 surface text 反查过去已经确认过的 entity。Alias 库具体技术形态尚未冻结。

## 3. 绝不保存什么

这层 memory 绝不保存：

- COA account（会计科目）。
- HST / GST treatment（税务处理）。
- journal entry（分录）。
- final transaction outcome（交易最终结果）。
- transaction processing path（交易处理路径）。
- case precedent（案例先例）。
- classification history（分类历史）。
- rule condition（规则条件）。
- active rule payload（生效规则内容）。
- raw evidence blob（原始证据正文）。
- unapproved AI reasoning（未经批准的 AI 推理）。
- runtime-only `entity_resolution_output`（实体识别运行时输出）。

它也不能替代：

- `Evidence Log`（证据日志）：原始 evidence（证据）和 evidence refs（证据引用）的 source of truth（事实来源）。
- `Transaction Log`（交易日志）：每笔交易最终结果和 audit trail（审计轨迹）的 source of truth（事实来源）。
- `Case Log`（案例日志）：entity-indexed completed-case learning memory（按实体索引的已完成案例学习记忆）。
- `Rule Log`（规则日志）：approved deterministic rules（已批准确定性规则）的 source of truth（事实来源）。
- `Governance Log`（治理日志）：高权限长期变化和审批历史的 source of truth（事实来源）。
- `Profile`（客户结构档案）：客户 bank accounts（银行账户）、loans（贷款）、employees（员工）和 structural relationships（结构关系）的 source of truth（事实来源）。

## 4. 对核心产品目标的贡献

这层 memory 必须清楚支持以下至少一项：

- [x] 记忆复用
- [x] 有证据支持的建议
- [x] accountant correction learning
- [x] 审计性
- [x] accountant control
- [x] 自动化率提升

具体贡献：

- 让过去已经确认过的 transaction surface text 可以通过 Alias 复用到同一 stable entity（稳定实体）。
- 让 Entity Resolution 可以在不完全确定当前交易主体时查询该 entity 是否存在一致 Alias 来辅助确认。
- 让 Entity Resolution 在完全识别不出主体时，可以通过 Alias 库查询是否存在一致或高度类似的历史 Alias，从而反推出对应 stable entity。
- 让 automation policy（自动化策略）把 entity-level risk（实体级风险）转成明确自动化边界。
- 让 governance refs（治理引用）保留 authority change（权威变化）的来源，支持审计和纠错。

## 5. 已知约束

- `active`（有效）只表示 entity（实体）可作为稳定身份目标，不等于可以自动分类。
- `Entity Resolution Node`（实体识别节点）判断为 `new_stable_entity`（新稳定实体）时，可以同步写入 Entity Log，不需要 governance approval（治理批准）；写入内容只限 entity 本体，不包括 Alias（别名）、automation policy（自动化策略）或 rule（规则）。
- Alias 只提供 identity reuse（身份复用）能力，不提供 rule authority（规则权威）或会计分类结论。
- `Case Log`（案例日志）可以提供 risk evidence（风险依据），但不能直接修改 Entity Log（实体日志）。
- `Transaction Log`（交易日志）不能作为 runtime identity source（运行时身份来源）或 learning source（学习来源）。
- automation policy upgrade / relaxation（自动化策略升级 / 放宽）必须 accountant approval（会计师批准）。

## 6. 未决定问题

- `entity_record`（实体记录）的 exact field schema（精确字段结构）尚未冻结。
- `alias_record`（别名记录）是否独立记录尚未冻结。
- Alias 库具体以什么技术形态呈现尚未冻结。
- candidate identity signal（候选身份信号）如需持久化，落点和 workflow 尚未冻结；但它不作为 Entity Log 中的 durable candidate entity lifecycle state（长期候选实体生命周期状态）。
- `automation_policy`（自动化策略）是否直接存储在 Entity Log，还是由 Governance Log 投影生成，尚未字段级冻结。
- merge / split（合并 / 拆分）后 alias（别名）、rule（规则）和 case（案例）的迁移行为尚未字段级冻结。
