# Case Log - Authority, Lifecycle, and Boundaries

## 1. Source of Truth / Authority

只填写当前已经稳定的 authority（权威）。未稳定的字段形态、写入者和触发顺序写入 Open Boundaries（未冻结边界）。

| Concept | Source of Truth | Authority Rule | Not Authority |
| --- | --- | --- | --- |
| completed case precedent（已完成案例先例） | `Case Log`（案例日志）加 `transaction_log_ref` 或等价 finalization proof | 在记录的 evidence condition 和 exception context 范围内，可支持 future case-based judgment | deterministic rule（确定性规则）、final audit record（最终审计记录）、完整 processing path、unapproved AI reasoning |
| case identity index（案例身份索引） | stable entity authority 来自 `Entity Log` | stable-linked case 可作为较强案例上下文 | unknown entity、runtime guess、Alias surface text、classification outcome |
| finalization proof（完成证明） | `Transaction Log` 或等价 transaction finalization source | Case Log 必须引用完成证明，证明案例来自 completed transaction | Case Log 自己不能替代 Transaction Log，也不能重写 final outcome |
| evidence condition（证据条件） | Evidence refs / evidence foundation 加 Case Log 记录的适用条件 | 限制 precedent 何时可被未来复用 | raw evidence blob、不可追溯摘要、LLM rationale |
| correction reference（纠正引用） | accountant correction / final confirmation refs，具体 finalization path 未冻结 | 可作为未来避免重复错误或解释例外的案例依据 | governance approval、rule authority、entity authority |
| case-derived candidate signal（案例衍生候选信号） | Case Log case history，进入 Post-Batch Lint / Review / Governance Review 路径 | 可以提出 rule promotion、automation risk、entity risk 或 policy review candidate | 直接 mutation、approved rule、relaxed automation policy、Entity Log authority |

## 2. 读取者

| Reader | 读取目的 | 可读内容 | 限制 |
| --- | --- | --- | --- |
| Case Judgment Node（案例判断节点） | 读取 relevant case precedent，辅助判断当前交易是否可走 case-based judgment | stable-linked case history、evidence condition、exception context、correction refs | 不能把 Case Log 当 deterministic rule source；不能用 unknown entity 作为 case identity handle |
| Post-Batch Lint Node（批后体检节点） | 读取 entity-linked case history，发现 rule promotion、automation risk 或 entity risk candidate | case pattern、exception / correction history、case-derived risk hints | 只能提出候选；不能直接修改 Entity Log、Rule Log 或 automation policy |
| Review Node（审核节点） | 帮助 accountant 理解当前 review item 的历史案例上下文 | relevant precedent、exception context、correction refs | Review 本身不把案例变成 rule、entity authority 或 governance approval |
| Governance Review Node（治理审核节点） | 评估 rule change、automation policy change 或 entity risk update | case evidence、case-derived candidate signal、supporting refs | 必须通过 governance / accountant approval；Case Log 只提供依据 |
| Knowledge Compilation Node（知识编译节点） | 生成 readable summary（可读摘要） | case history summary material and refs | summary 不能替代 Case Log source authority，也不能替代 Transaction Log / Entity Log / Rule Log |

## 3. 写入者

当前没有完全冻结的 durable Case Log writer（长期 Case Log 写入者）。已确认的是写入资格语义：Case Log 只能从 completed transaction 中抽取可学习案例，并必须引用 `transaction_log_ref` 或等价 finalization proof。

| Writer | 可以写什么 | 写入类型 | 需要 approval 吗 |
| --- | --- | --- | --- |
| Case Memory Update Node / unified finalization mechanism（候选写入机制，未冻结） | completed-case learning record、evidence condition、exception context、correction reference、transaction finalization refs、case-derived candidate signals | direct case write，具体 writer / trigger order 未冻结 | 必须基于 completed transaction 和 finalization proof；是否需要额外 approval 未冻结 |
| Review / Coordinator / accountant interaction finalization path（未冻结） | accountant-confirmed outcome 或 correction refs 进入 Case Log 的来源上下文 | finalization input / correction context，具体写入机制未冻结 | accountant final confirmation 是当前交易完成依据；是否由本路径直接写 Case Log 未冻结 |
| Post-Batch Lint / Governance Review（候选路径） | rule promotion、automation risk、entity risk 或 policy review candidate | candidate | 是；不能直接把 candidate 变成 Entity Log / Rule Log / Governance Log mutation |

## 4. Candidate 边界

以下内容可以作为 candidate（候选）或 weak context（弱上下文）：

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
- 作为 Rule Log / Entity Log / automation policy mutation：必须经过 Post-Batch Lint / Review / Governance Review 路径，并获得 accountant / governance approval；具体路径未完全冻结。

## 5. Lifecycle / States

当前不冻结完整 Case Log lifecycle state enum（生命周期状态枚举），只记录已必要的语义类别。

| State | 含义 | 谁可以进入 | 谁可以退出 | 下游含义 |
| --- | --- | --- | --- | --- |
| completed stable-linked case（已完成稳定实体案例，不是 enum） | completed transaction 已形成可学习案例，并 linked to stable entity | Case Log 写入机制未冻结；必须有 finalization proof | supersession / correction path 未冻结 | 可作为较强 future case-based judgment context；不等于 rule authority |
| corrected / superseded case（纠正 / 替代案例，未冻结） | 原案例因 correction、reversal、duplicate、split 或其他 finalization change 需要被限制或替代 | 未冻结 | 未冻结 | 下游不能静默使用过期案例；具体行为未冻结 |

## 6. Mutation Path

已冻结语义约束：

```text
completed transaction
-> final transaction outcome / finalization proof
-> Case Log write eligibility check
-> completed-case learning record with entity ref, evidence condition, exception context, correction reference, and transaction_log_ref
-> future readers use case only within recorded authority limits
```

已冻结 candidate-to-governance 约束：

```text
Case Log evidence
-> Post-Batch Lint / Review / Governance Review candidate
-> accountant / governance approval
-> Entity Log mutation or Rule Log mutation
-> Governance Log audit trail
```

未冻结 mutation path：

- Case Log 的 exact writer 是 Case Memory Update Node、统一 finalization / memory write mechanism，还是其他机制。
- `Case Log` 与 `Transaction Log` 的 exact trigger order。
- completed transaction 的 case write eligibility。
- corrected / reversed / duplicate / split transaction 的 supersession 行为。
- accountant correction 如何进入 Case Log 的 exact write mechanism。
- case-derived signals 进入 governance candidate 的 exact contract。
- 多 log 统一写入机制如何同时处理 Entity Log、Case Log、Transaction Log、Intervention Log 等写入。

例外：

- 无已冻结例外。

## 7. 与其他 memory/log 的边界

| Other Store | 已确认边界 | 未冻结边界 |
| --- | --- | --- |
| Evidence Log（证据日志） | Evidence Log 保存 raw evidence 和 evidence refs；Case Log 只保存 evidence refs / evidence condition，不复制原始证据正文 | evidence condition 的 exact field schema 和 sufficiency threshold 未冻结 |
| Transaction Log（交易日志） | Transaction Log 保存 final transaction audit record；Case Log 保存从已完成交易中抽取的可学习案例，并必须引用 transaction_log_ref 或等价 finalization proof；entity 未解析即完成分类的交易只 finalize 到 Transaction Log，不进入 Case Log | Case Log 与 Transaction Log 的 exact trigger order、correction / reversal / split 后的 supersession 行为，以及身份缺口标记落点（初步倾向 Transaction Log）未冻结 |
| Entity Log（实体日志） | Entity Log 保存 stable entity identity、Alias、status、automation policy；Case Log 按 entity_id 组织案例，但不修改 entity authority | case-derived risk 进入 Entity Log 的治理路径未冻结 |
| Alias Log（别名日志） | Alias Log 保存 confirmed transaction surface text -> stable entity；命中 Alias 后，系统可以以对应 stable entity 为主体查询 Case Log | Alias 命中后的具体 case retrieval contract 未冻结；Case Log 不保存 Alias authority |
| Rule Log（规则日志） | Rule Log 保存 approved deterministic rules；Case Log 只可提供 rule promotion candidate 的案例依据 | case-derived rule promotion signal 的 exact eligibility 和 approval path 未冻结 |
| Governance Log（治理日志） | Governance Log 保存高权限变化、批准、拒绝、降级、合并和拆分历史；Case Log 的 risk / policy / rule candidates 必须经过治理或 accountant approval 才能 mutation | Case Log candidate 与 Governance Log event / audit trace 的 exact contract 未冻结 |
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
- runtime 行为：Case Log 可以支持 case-based judgment 或 rule promotion candidate，不能直接 deterministic rule match。
- 是否阻断自动化：如果没有 approved rule 或 automation policy 支持，不得把 repeated case 当 rule 自动放行。
- 是否生成 review / governance candidate：可以生成 rule promotion candidate。

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
- correction refs / accountant confirmation refs。
- exception context refs。
- case-derived candidate refs，如果用于 rule promotion、automation risk 或 entity risk review。
- created_from / updated_by / superseded_by refs，字段名未冻结。

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

1. Case Log write eligibility。
2. Case Log exact writer / finalization mechanism。
3. Case Log 与 Transaction Log 的 exact trigger order。
4. completed / corrected / reversed / duplicate / split case 的 supersession behavior。
5. case-derived rule promotion、automation risk 和 entity risk candidate 的 exact governance contract。
6. Case Log record schema、field names、enum 和 validation rules。
7. 多 log 统一写入机制如何处理 Entity Log、Case Log、Transaction Log、Intervention Log 等写入。
8. Knowledge Summary 与 Case Log 冲突时的 repair path。

这些问题解决前，不能进入：

- [x] M3 data contract
- [x] M4 storage and operations
- [x] implementation
