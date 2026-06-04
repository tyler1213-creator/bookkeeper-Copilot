# Alias Log - Authority, Lifecycle, and Boundaries

## 1. Source of Truth / Authority

只填写当前已经稳定的 authority（权威）。未稳定的关系写入 Open Boundaries（未冻结边界）。

| Concept | Source of Truth | Authority Rule | Not Authority |
| --- | --- | --- | --- |
| Alias relationship（别名关系） | `Alias Log`（别名日志）或其后续冻结的等价 Alias 库 / 投影 | 过去已经确认过的 transaction surface text 可以指向对应 stable entity（稳定实体），用于辅助 Entity Resolution 判断当前交易主体 | 所有历史 description、未确认 surface text、LLM semantic similarity（模型语义相似）、normalized display text（标准化展示文本） |
| Alias source surface text（Alias 来源表面文本） | traceable transaction evidence（可追溯交易证据）加已确认 identity relationship（身份对应关系） | 当前只确认 bank statement description / descriptor / raw bank surface text；以及在 bank description 无明确身份意义时，可重复且能指向主体的字段，例如 cheque payee | raw evidence blob 本身、任意历史字段、不可追溯摘要 |
| Alias target（Alias 指向对象） | `Entity Log`（实体日志）中的 stable entity authority | Alias 只能指向明确 stable entity；Alias 不创建 stable entity identity | unknown entity、runtime guess（运行时猜测）、classification outcome（分类结果） |
| Alias lookup result（Alias 查询结果） | Alias Log / Alias 库查询结果加 Entity Log authority check（实体日志权威检查） | 命中已确认 Alias 时，可复用该 Alias 指向的 stable entity | 高度类似但未受控确认的 surface text、单次模型判断、下游规则结果 |
| Alias trace（Alias 追溯） | evidence refs、confirmation refs、lookup basis；字段名未冻结 | 用于 review、correction、governance 和 audit | accounting classification、rule authority、accountant approval、governance approval |

## 2. 读取者

| Reader | 读取目的 | 可读内容 | 限制 |
| --- | --- | --- | --- |
| Entity Resolution Node（实体识别节点） | 从当前交易 surface text 反查过去已经确认过的 stable entity；或以某个 entity 为目标查询是否存在一致 Alias | confirmed Alias relationship（已确认别名关系）、Alias source surface text、target stable entity refs、trace refs | Alias 只支持 identity resolution；不能提供 COA / HST / JE / rule authority；高度类似历史 Alias 不能天然等同于确认 identity |

其他 reader（读取者）尚未冻结。Review、Governance、Knowledge Compilation 或其他下游节点是否读取 Alias Log，以及读取边界是什么，必须在后续对应 node / memory layer 讨论中决定。

## 3. 写入者

当前没有冻结的 durable Alias writer（长期 Alias 写入者）。

已确认的是 eligibility boundary（资格边界）：

- 只有已经和某个明确 stable entity 建立过确认关系的 surface text，才属于 Alias。
- 未确认 surface text 不能被写成 Alias authority。
- Entity Resolution 判断为 `new_stable_entity` 时，同步写入 Entity Log 的内容只限 entity 本体，不包括 Alias。

| Writer | 可以写什么 | 写入类型 | 需要 approval 吗 |
| --- | --- | --- | --- |
| 未冻结 | 未冻结 | 未冻结 | 未冻结 |

## 4. Candidate 边界

当前不定义 durable Alias candidate（长期候选别名）状态，也不使用旧 Alias 状态模型。

以下内容可以作为 runtime issue / review context（运行时问题 / 审核上下文），但不是 Alias authority：

- 当前交易中尚未确认的 surface text。
- 与历史 Alias 高度类似但未受控确认的 surface text。
- Alias 写入需求或 Alias 冲突提示。
- 同一 surface text 可能对应多个 stable entity 的竞争信号。

这些 runtime issue / review context 不能直接成为：

- Alias relationship（别名关系）。
- stable entity authority（稳定实体权威）。
- rule authority（规则权威）。
- accounting classification（会计分类结论）。
- automation permission（自动化许可）。

Candidate 或 issue 进入 durable Alias authority 的条件：

- 未冻结。当前只能确定：进入 durable Alias authority 前，必须已经确认 surface text 与明确 stable entity 的对应关系。

## 5. Lifecycle / States

当前不冻结 Alias lifecycle state enum（生命周期状态枚举），也不设计 Alias 状态机。

| State | 含义 | 谁可以进入 | 谁可以退出 | 下游含义 |
| --- | --- | --- | --- | --- |
| 已确认对应关系（不是 Alias 状态枚举） | 过去已经确认过的 transaction surface text 指向某个明确 stable entity | 未冻结 | 未冻结 | Entity Resolution 可以用它辅助识别当前交易主体；不代表 rule、classification 或 automation authority |

## 6. Mutation Path

当前只冻结 durable Alias 的资格约束，不冻结具体 mutation path（变更路径）。

已冻结约束：

```text
transaction surface text
-> confirmed relationship to a clear stable entity
-> may become Alias authority
```

未冻结 mutation path：

- 创建 stable entity 时，当前 surface text 是否默认进入 Alias 库。
- Accountant 在 pending / clarification 中确认 identity 后，是否以及如何写入 Alias。
- Alias 的写入者、审批者、review path 和 finalization mechanism。
- Alias 冲突、多 entity 竞争或相似匹配的处理路径。
- merge / split entity 后 Alias 的迁移、阻断或 supersession（替代）路径。
- Alias Log 与 Governance Log 的 mutation / audit trace 边界。

例外：

- 无已冻结例外。

## 7. 与其他 memory/log 的边界

只写已确认边界。未确认外部关系进入 Open Boundaries。

| Other Store | 已确认边界 | 未冻结边界 |
| --- | --- | --- |
| Evidence Log（证据日志） | Evidence Log 保存 raw evidence 和 evidence refs；Alias Log 只保存或引用支持 Alias 关系的可追溯来源，不复制原始证据正文 | evidence sufficiency threshold（证据充分门槛）尚未冻结 |
| Entity Log（实体日志） | Entity Log 保存 stable entity identity；Alias 只能指向明确 stable entity | Alias Log 与 Entity Log 是独立 store、嵌套记录还是查询投影尚未冻结 |
| Case Log（案例日志） | 命中 Alias 后，系统可以以对应 stable entity 为主体查询 Case Log；Alias Log 不保存 case precedent | Alias 命中后的具体 case retrieval contract 尚未冻结 |
| Rule Log（规则日志） | Alias Log 不保存 rule condition、active rule 或 accounting treatment | Alias 与规则路径的关系暂不讨论，尚未冻结 |
| Transaction Log（交易日志） | Transaction Log 保存 final transaction audit record；不是 runtime Alias authority | transaction correction 如何影响 Alias review / mutation 尚未冻结 |
| Governance Log（治理日志） | Governance Log 保存高权限变化和审批历史；Alias mutation 是否进入 Governance Log 尚未冻结 | Alias 写入、冲突处理、merge / split 迁移是否需要 governance approval 尚未冻结 |
| Knowledge Summary（知识摘要） | Knowledge Summary 是可读摘要，不能替代 Alias authority | summary conflict repair（摘要冲突修复）尚未冻结 |

## 8. 冲突处理

如果 current surface text（当前表面文本）与 Alias Log / Alias 库冲突，或同一 surface text 可能对应多个 stable entity：

- authority 顺序：已确认 stable entity authority、已确认 Alias relationship 和可追溯 evidence 优先于 LLM semantic match（模型语义匹配）。
- runtime 行为：Entity Resolution 应输出 conflict / blocked / ambiguous identity（冲突 / 阻断 / 歧义身份）语义，不得把冲突包装成 stable identity basis（稳定身份基础）。
- 是否阻断自动化：不得用该 Alias 支持 deterministic path（确定性路径）。
- 是否生成 review / governance candidate：可以生成 issue，但具体处理路径尚未冻结。

如果 current surface text 只是高度类似历史 Alias：

- authority 顺序：confirmed Alias relationship（已确认 Alias 关系）优先；相似度本身不是 authority。
- runtime 行为：它最多增强 Entity Resolution 的判断；只有在后续冻结受控 normalization / equivalence 规则、没有冲突、没有多 entity 竞争时，才可能作为更强匹配依据。
- 是否阻断自动化：如果身份仍不安全，应保守阻断相关自动化路径。
- 是否生成 review / governance candidate：未冻结。

如果 Alias Log 与 Entity Log / Governance Log 冲突：

- authority 顺序：stable entity identity 和已生效 governance authority 优先。
- runtime 行为：Entity Resolution 不应忽略 entity lifecycle、merge / split 或 governance constraint（治理限制）。
- 是否阻断自动化：如果冲突影响 identity safety（身份安全），应阻断相关自动化路径。
- 是否生成 review / governance candidate：可以；具体路径尚未冻结。

## 9. Audit / Trace 边界

这层 memory 必须留下的 trace：

- Alias source surface text（Alias 来源表面文本）。
- target stable entity ref（目标稳定实体引用）。
- supporting evidence refs（支持证据引用）。
- confirmation source（确认来源），字段名未冻结。
- created_from / updated_by（创建 / 更新来源），字段名未冻结。
- alias lookup basis（Alias 查询依据），如果用于 runtime identity decision。
- conflict / supersession refs（冲突 / 替代引用），如果发生。

这些 trace 用于：

- review（审核）。
- correction（纠正）。
- governance（治理）。
- audit（审计）。

这些 trace 不能成为：

- accounting classification（会计分类）。
- rule authority（规则权威）。
- case authority（案例权威）。
- accountant approval（会计师批准）。
- governance approval（治理批准）。

## 10. Open Boundaries

以下问题未冻结：

1. Alias 库具体技术形态。
2. Alias Log 与 Entity Log 的 storage / projection boundary（存储 / 投影边界）。
3. `alias_record` 的 exact field schema。
4. Alias 写入责任、审批路径和统一 memory write 机制。
5. 创建 stable entity 时，当前 surface text 是否默认进入 Alias 库。
6. 受控 normalization / equivalence 规则。
7. 高度类似历史 Alias 何时可以支持更强 identity match。
8. Alias 冲突、多 entity 竞争、merge / split 后迁移或阻断规则。
9. Alias 对其他下游节点的支持范围。
10. Alias mutation 与 Governance Log / audit trace 的边界。

这些问题解决前，不能进入：

- [x] M3 data contract
- [x] M4 storage and operations
- [x] implementation
