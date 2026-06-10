# Rule Log - Memory Intent

## 1. 为什么这层 memory 必须存在

它解决的问题是：

> Rule Log 必须存在，是因为系统需要一个 durable source of truth 来保存已经批准、可被 RuleMatch 确定性执行的 rule。

如果删除它，会失去：

- RuleMatch 在 stable entity 下直接复用 approved deterministic rule 的能力。
- rule applicability、approved accounting treatment、scope 类型、当前执行状态语义和来源 / 批准 provenance 的 durable authority。
- “该 entity 名下有哪些 rule”的单一成员关系真值。
- 把可执行规则权威与 identity、case evidence、runtime execution 分离的边界。

这些能力不能由 runtime handoff 或现有 store 覆盖，因为：

- runtime handoff 只表达当前交易上下文，不保存 durable rule authority。
- Entity Log 只表达 identity 和 automation control，不保存 rule condition、accounting treatment、rule lifecycle 或可执行 rule payload。
- Case Log 只表达案例先例和 promotion evidence，不是 deterministic rule authority。
- RuleMatchNode 只执行规则，不拥有规则权威、lifecycle 或 mutation 权限。

## 2. 保存什么

这层 memory 只保存 approved executable rules。每条 rule 至少表达：

- rule ref 语义，exact 字段名和 ref 形态留 M3。
- `entity_id` 主索引。
- scope 类型：entity-level rule 或 pattern-level rule。
- applicability：stable entity 身份，以及 pattern-level rule 所需的客观条件；entity-level rule 是条件集为空的特殊形态。
- approved accounting treatment：RuleMatch 命中后交给 JE Generator 的 judgment-free 完整处理。
- current execution state 语义：该 rule 当前是否可被 RuleMatch 读取并执行；字段名 / enum 不在本轮冻结。
- source / approval provenance refs：说明 rule authority 从何而来、由谁批准或经何种治理结果生效。

其中可以成为 reusable authority 的是：

- executable rule 本身，具体包括可客观判定的 applicability 和已批准、完整、judgment-free executable 的 approved accounting treatment。

只作为 trace / context / explanation 的是：

- source / approval provenance refs。它们是 rule 生效资格 trace，不是交易审计记录，不替代 Governance Log / Transaction Log / Case Log 正文，也不单独成为会计处理 authority。

Rule Log 保存 approved accounting treatment，但不保存 JE 或 JE 生成过程。Treatment 必须 judgment-free executable；exact treatment schema 与 JE Generator contract 留联合 L3。

## 3. 绝不保存什么

这层 memory 绝不保存：

- rule candidate。
- promotion queue。
- review-only guideline。
- accountant note。
- Case Log evidence 正文。
- Transaction Log 审计轨迹或完整交易历史。
- EntityLog automation_policy 本体。
- raw evidence blob。
- LLM reasoning。
- 相似描述。
- 模糊自然语言规则。
- match_count、last_hit、miss rate、health score 或 staleness score 作为 authority。

它也不能替代：

- Entity Log：stable entity identity、entity lifecycle、automation_policy / control state 的主档案。
- Case Log：completed-case precedent 和 promotion evidence layer。
- Transaction Log：最终交易审计 source of truth。
- Governance Log：审批、暂停、废除、替代等治理事件正文。

rule-hit 来源语义和 rule ref 应随 finalized transaction / case context 保留；统计由 Transaction Log / Case Log 的逐笔记录派生，或由维护视图缓存。缓存不改变 Rule Log authority。

## 4. 对核心产品目标的贡献

这层 memory 必须清楚支持以下至少一项：

- [ ] 记忆复用
- [ ] 有证据支持的建议
- [ ] accountant correction learning
- [x] 审计性
- [x] accountant control
- [x] 自动化率提升

具体贡献：

- 支持自动化率提升：Rule Log 让系统在 stable entity 与上级 automation control 已放行时，不重新判断会计处理，直接复用已批准的 deterministic rule。
- 支持审计性：Rule Log 保存 source / approval provenance refs 作为 rule 生效资格 trace，便于 review、correction、governance 和 audit 追溯。
- 支持 accountant control：只有满足 authority-content 标准且已经 accountant 批准的内容，才能成为 RuleMatch 可执行 rule；系统建议不能自动成立 rule。

## 5. 已知约束

- Rule Log 以 `entity_id` 为主索引。
- 每条 rule 自带 scope 类型和 applicability。
- Entity-level rule 是条件集为空的特殊情形。
- Pattern-level rule 在同一 stable entity 下记录额外客观条件，条件挂在 rule applicability 上。
- Pattern-level 条件必须是当前交易中可客观判定、可复核的事实条件。
- EntityLog 不保存 rule_id 成员列表；“该 entity 名下有哪些 rule”的成员关系单一归 Rule Log。
- RuleMatch 凭 `entity_id` 查询 Rule Log 当前可执行 rule 集合，再用当前交易客观事实匹配每条 rule 的 applicability。
- Rule Log 中存在 executable rule 只是 RuleMatch 自动化的必要非充分条件；EntityLog automation_policy / control state 仍是上级放行控制。

## 6. 未决定问题

- 见 `00_index.md` 的“当前未冻结边界”与“进入下一阶段前必须解决”。
