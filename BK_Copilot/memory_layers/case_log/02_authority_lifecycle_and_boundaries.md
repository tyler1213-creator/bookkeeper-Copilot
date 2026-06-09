# Case Log - Authority, Lifecycle, and Boundaries

## 1. Source of Truth / Authority

只填写当前已经稳定的 authority（权威）。未稳定的字段形态、写入者和触发顺序写入 Open Boundaries（未冻结边界）。

| Concept | Source of Truth | Authority Rule | Not Authority |
| --- | --- | --- | --- |
| completed case precedent（已完成案例先例） | `Case Log`（案例日志）加 `transaction_log_ref` 或等价 finalization proof | 在记录的 evidence condition、context note 和 `use_level` 范围内，可支持 future case-based judgment | deterministic rule（确定性规则）、final audit record（最终审计记录）、完整 processing path、修改历史、unapproved AI reasoning |
| case identity index（案例身份索引） | stable entity authority 来自 `Entity Log` | stable-linked case 可作为较强案例上下文 | unknown entity、runtime guess、Alias surface text、classification outcome |
| finalization proof（完成证明） | `Transaction Log` 或等价 transaction finalization source | Case Log 必须引用完成证明，证明案例来自 finalized transaction | Case Log 自己不能替代 Transaction Log，也不能重写 final outcome、audit trail 或修改历史 |
| evidence condition（证据条件） | Evidence refs / evidence foundation 加 Case Log 记录的适用条件 | 限制 precedent 何时可被未来复用 | raw evidence blob、不可追溯摘要、LLM rationale |
| reusable permission（复用权限） | `Case Log` 中的 `use_level` 语义（exact enum 留 M3） | 限定每条 case 可作为正面先例、仅例外、反面 / 冲突、不可升级 rule，或已失效 / superseded | rule approval、governance approval、Entity Log policy |
| rule / automation / entity risk review evidence（规则 / 自动化 / 实体风险审核依据） | Case Log case history | 可以作为治理流程判断 rule promotion、automation risk、entity risk 或 policy review 的历史依据 | 直接 mutation、approved rule、relaxed automation policy、Entity Log authority、Case Log 存储字段形式的 candidate queue |

## 2. 读取者

| Reader | 读取目的 | 可读内容 | 限制 |
| --- | --- | --- | --- |
| Case Judgment Node（案例判断节点） | 读取 relevant case precedent，辅助判断当前交易是否可走 case-based judgment | stable-linked case history、evidence condition、context note、`confirm_by`、`use_level` | 不能把 Case Log 当 deterministic rule source；不能用 unknown entity 作为 case identity handle；读取聚合机制留 L4 |
| Rule / automation / entity risk review flow（规则 / 自动化 / 实体风险审核流程，具体节点未冻结） | 读取 entity-linked case history，评估 rule promotion、automation risk 或 entity risk | case pattern、exception / correction history、`use_level`、case history distribution 派生视图 | 只能作为治理依据；不能直接修改 Entity Log、Rule Log、Governance Log 或 automation policy |
| Review Node（审核节点） | 帮助 accountant 理解当前 review item 的历史案例上下文 | relevant precedent、exception context、`use_level` | Review 本身不把案例变成 rule、entity authority 或 governance approval |
| Governance Review Node（治理审核节点） | 评估 rule change、automation policy change 或 entity risk update | case evidence、supporting refs、读取层派生聚合 | 必须通过 governance / accountant approval；Case Log 只提供依据 |
| Knowledge Compilation Node（知识编译节点） | 生成 readable summary（可读摘要） | case history summary material and refs | summary 不能替代 Case Log source authority，也不能替代 Transaction Log / Entity Log / Rule Log |

## 3. 写入者

当前没有完全冻结的 durable Case Log writer（长期 Case Log 写入者）。已确认的是写入资格语义：所有已关联 stable entity 且已 finalized 的交易均具备 Case Log 写入资格，并必须引用 `transaction_log_ref` 或等价 finalization proof。

| Writer | 可以写什么 | 写入类型 | 需要 approval 吗 |
| --- | --- | --- | --- |
| Case Memory Update Node / unified finalization mechanism（候选写入机制，未冻结） | completed-case learning record、evidence condition、context note、`confirm_by`、`use_level`、transaction finalization refs | direct case write，具体 writer / trigger order 未冻结 | 必须基于 stable-linked finalized transaction 和 finalization proof；是否需要额外 approval 未冻结 |
| Review / Coordinator / accountant interaction finalization path（未冻结） | accountant-confirmed outcome 或 correction context 进入 Case Log 的来源上下文 | finalization input / correction context，具体写入机制未冻结 | accountant final confirmation 是当前交易完成依据；是否由本路径直接写 Case Log 未冻结 |
| Rule / automation / entity risk review flow（未冻结） | 读取 Case Log 生成治理候选 | temporary handoff / candidate outside Case Log storage | 是；不能直接把 candidate 变成 Entity Log / Rule Log / Governance Log mutation；candidate_signal_refs 不作为 Case Log 存储字段 |

## 4. Candidate 边界

以下内容可以由下游治理 / 审核流程基于 Case Log 历史依据生成 candidate（候选）或 weak context（弱上下文）：

- rule promotion candidate（规则升级候选）。
- automation risk review candidate（自动化风险审核候选）。
- entity risk update candidate（实体风险更新候选）。
- policy review candidate（策略审核候选）。
- exception / correction pattern（例外 / 纠正模式）。

Candidate 或 weak context 不能直接成为：

- active rule（生效规则）。
- Entity Log mutation（实体日志变更）。
- Alias relationship（别名关系）。
- relaxed automation policy（放宽后的自动化策略）。
- final audit record（最终审计记录）。

Candidate 进入 durable authority 或 governance path 的条件：

- 作为 Case Log precedent：必须来自 completed transaction，并具备 transaction finalization proof。
- 作为 Rule Log / Entity Log / automation policy mutation：必须经过治理 / review 路径，并获得 accountant / governance approval；具体路径未完全冻结。
- candidate signal 是下游临时 handoff，不作为 Case Log durable storage 字段。

## 5. Lifecycle / States

当前不冻结完整 Case Log lifecycle state enum（生命周期状态枚举），只记录已必要的复用安全语义。

| State | 含义 | 谁可以进入 | 谁可以退出 | 下游含义 |
| --- | --- | --- | --- | --- |
| usable stable-linked case（可用稳定实体案例，不是 enum） | finalized transaction 已形成逐笔案例，并 linked to stable entity，且 `use_level` 允许在限定条件下复用 | Case Log 写入机制未冻结；必须有 finalization proof | `use_level` 被更新为限制 / 失效；具体机制未冻结 | 可作为 future case-based judgment context；不等于 rule authority |
| limited / exception-only case（受限 / 仅例外案例，不是 enum） | 案例可作为例外、反面、冲突或不可升级 rule 的上下文，但不能当正常正面先例 | 由 final outcome / correction / governance result 触发；机制未冻结 | 机制未冻结 | 下游只能按 `use_level` 限制使用，不得静默当作正面先例 |
| invalid / superseded case（失效 / 被替代案例，不是 enum） | 原案例因 correction、reversal、治理限制、merge / split 重新归属等原因不再可作为有效正面先例 | 由 correction / finalization / governance result 触发；机制未冻结 | 机制未冻结 | 下游不得静默使用为有效先例；血缘 ID 链接和执行机制留 L4 |

## 6. Mutation Path

已冻结语义约束：

```text
completed transaction
-> final transaction outcome / finalization proof
-> Case Log write eligibility check
-> completed-case learning record with entity ref, transaction_log_ref, accounting outcome snapshot, evidence condition, context note, confirm_by, and use_level
-> future readers use case only within recorded authority limits
```

已冻结 candidate-to-governance 约束：

```text
Case Log evidence
-> rule / automation / entity risk review candidate generated outside Case Log storage
-> accountant / governance approval
-> Entity Log mutation or Rule Log mutation
-> Governance Log audit trail
```

未冻结 mutation path：

- Case Log 的 exact writer 是 Case Memory Update Node、统一 finalization / memory write mechanism，还是其他机制。
- `Case Log` 与 `Transaction Log` 的 exact trigger order。
- corrected / reversed / duplicate / split transaction 的 supersession 血缘链接和重判执行机制。
- accountant correction 如何进入 Case Log 的 exact write mechanism。
- case-derived evidence 进入 governance candidate 的 exact contract。
- 多 log 统一写入机制如何同时处理 Entity Log、Case Log、Transaction Log、Intervention Log 等写入。

例外：

- 无已冻结例外。

## 7. 与其他 memory/log 的边界

| Other Store | 已确认边界 | 未冻结边界 |
| --- | --- | --- |
| Evidence Log（证据日志） | Evidence Log 保存 raw evidence 和 evidence refs；Case Log 只保存 evidence refs / evidence condition，不复制原始证据正文 | evidence condition 的 exact field schema 和 sufficiency threshold 未冻结 |
| Transaction Log（交易日志） | Transaction Log 保存 final 交易事实、完整 processing path、修改历史与 audit trail，是审计 source of truth。Case Log 只引用 transaction_log_ref / finalization proof，只保存最新可学习状态，不存修改历史、不存完整 processing path、不替代审计记录。entity 未解析即完成分类的交易只 finalize 到 Transaction Log，不进入 Case Log | Case Log 与 Transaction Log 的 exact trigger order、writer、多 log finalization、correction / reversal / split 后的执行机制，以及身份缺口标记落点（初步倾向 Transaction Log）未冻结 |
| Entity Log（实体日志） | Entity Log 保存 stable entity identity、Alias、status、automation policy；Case Log 按 entity_id 组织案例，但不修改 entity authority。merge / split / archive 后旧 case 归属由治理层裁定，Case Log 不裁判但服从结果 | case-derived risk 进入 Entity Log 的治理路径、merge / split / archive 后重新归属执行机制未冻结 |
| Alias Log（别名日志） | Alias Log 保存 confirmed transaction surface text -> stable entity；Case Log 可保存本案例交易的 Alias 表面引用 / 快照用于模式区分，但不持有 Alias authority | Alias 命中后的具体 case retrieval contract 未冻结；Alias correction 如何影响 Case Log alias 快照属于后续机制 |
| Rule Log（规则日志） | Rule Log 保存 approved deterministic rules；Case Log 只提供 rule promotion 评估所需历史案例依据 | case-derived rule promotion evidence 的 exact eligibility 和 approval path 未冻结 |
| Governance Log（治理日志） | Governance Log 保存高权限变化、批准、拒绝、降级、合并和拆分历史；Case Log 的 risk / policy / rule 相关变化必须经过治理或 accountant approval 才能 mutation | Case Log evidence 与 Governance Log event / audit trace 的 exact contract 未冻结 |
| Knowledge Summary（知识摘要） | Knowledge Summary 是可读摘要，不能替代 Case Log source authority；Case Log 也不替代 readable summary | summary conflict repair 未冻结 |

## 8. 冲突处理

如果 Case Log 与 Transaction Log / finalization proof 冲突：

- authority 顺序：Transaction Log 或等价 finalization source 对 final outcome 和 audit trail 优先。
- runtime 行为：缺少有效 finalization proof 的 case 不能作为 reusable precedent。
- 是否阻断自动化：如果当前自动化依赖该案例，应保守阻断相关自动化路径。
- 是否生成 review / governance candidate：可以生成 repair / review candidate；具体路径未冻结。

如果 Case Log 与 Entity Log / Governance Log 冲突：

- authority 顺序：Entity Log 和已生效 Governance Log 对 entity identity、status、automation policy 优先。
- runtime 行为：Case Log 只能提供 risk evidence 或历史上下文，不能覆盖 active entity authority。
- 是否阻断自动化：除非已有 policy / governance constraint，否则 Case Log 本身不直接修改 automation boundary；但身份或策略冲突应保守处理。
- 是否生成 review / governance candidate：是。

如果 repeated case outcome 与 Rule Log 冲突或尚未进入 Rule Log：

- authority 顺序：Rule Log 中 approved active rule 优先；repeated outcome 本身不是 rule authority。
- runtime 行为：Case Log 可以支持 case-based judgment 或 rule promotion 评估依据，不能直接 deterministic rule match。
- 是否阻断自动化：如果没有 approved rule 或 automation policy 支持，不得把 repeated case 当 rule 自动放行。
- 是否生成 review / governance candidate：可由治理 / review 流程基于 Case Log 生成；Case Log 本身不存 candidate queue。

如果 Case Log 与 Knowledge Summary 冲突：

- authority 顺序：Case Log source records 优先于 readable summary。
- runtime 行为：summary 只能作为可读背景，不得覆盖 Case Log。
- 是否阻断自动化：如果 summary 暗示风险但 Case Log / Governance Log 未确认，不能直接改变 authority；可生成 review candidate。
- 是否生成 review / governance candidate：可以。

## 9. Audit / Trace 边界

这层 memory 必须留下的 trace：

- entity ref（stable entity；字段名未冻结）。
- `transaction_log_ref` 或等价 finalization proof。
- evidence refs / evidence condition refs。
- `confirm_by` / accountant confirmation type。
- `use_level`。
- context note / exception context refs。
- created_from / updated_by refs，字段名未冻结。

这些 trace 用于：

- review（审核）。
- correction（纠正）。
- governance（治理）。
- audit（审计）。

这些 trace 不能成为：

- entity identity authority。
- Alias authority。
- active rule authority。
- final transaction audit record。
- accountant approval。
- governance approval。

## 10. Open Boundaries

以下问题未冻结：

1. Case Log record schema、field names、enum 和 validation rules。
2. `use_level` 与 `confirm_by` 的 exact enum 取值。
3. 共享 accounting outcome（COA / HST / split / allocation）的全局字段结构。
4. Case Log exact writer / finalization mechanism。
5. Case Log 与 Transaction Log 的 exact trigger order。
6. Case Judgment 读取时的 per-entity rollup / retrieval 机制。
7. supersession 的血缘链接字段与 corrected / reversed / duplicate / split 后的重判执行机制。
8. 多 log 统一写入机制如何处理 Entity Log、Case Log、Transaction Log、Intervention Log 等写入。
9. case-derived rule promotion、automation risk 和 entity risk candidate 的 exact governance contract。
10. entity / 类别级通则批注的归属。
11. Knowledge Summary 与 Case Log 冲突时的 repair path。

这些问题解决前，不能进入：

- [x] M3 data contract
- [x] M4 storage and operations
- [x] implementation
