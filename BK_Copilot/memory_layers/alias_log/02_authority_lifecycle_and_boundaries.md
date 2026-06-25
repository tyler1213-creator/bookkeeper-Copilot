# Alias Log - Authority, Lifecycle, and Boundaries

## 1. Source of Truth / Authority

只填写当前已经稳定的 authority（权威）。未稳定的关系写入 Open Boundaries（未冻结边界）。

| Concept | Source of Truth | Authority Rule | Not Authority |
| --- | --- | --- | --- |
| Alias relationship（别名关系） | `Alias Log`（别名日志）或其后续冻结的等价 Alias 库 / 投影 | 已确认交易中的 raw transaction description / descriptor / raw bank surface text 可以指向对应 stable entity（稳定实体），用于 Entity Resolution 的 identity reuse；不要求该 surface text 曾是 stable identity 判断的决定性 evidence | 所有历史 description、未确认 surface text、LLM semantic similarity（模型语义相似）、normalized display text（标准化展示文本） |
| Alias source surface text（Alias 来源表面文本） | traceable transaction evidence（可追溯交易证据）加已确认 identity relationship（身份对应关系） | 当前只确认 bank statement description / descriptor / raw bank surface text；以及在 bank description 无明确身份意义时，可重复且能指向主体的字段，例如 cheque payee | raw evidence blob 本身、任意历史字段、不可追溯摘要 |
| Alias target（Alias 指向对象） | `Entity Log`（实体日志）中的 stable entity authority | Alias 只能指向明确 stable entity；Alias 不创建 stable entity identity | unknown entity、runtime guess（运行时猜测）、classification outcome（分类结果） |
| Alias lookup result（Alias 查询结果） | Alias Log / Alias 库查询结果，以及治理 / correction / finalization 层对 durable memory 的同步维护 | exact raw description match 可以支撑 Entity Resolution 复用该 Alias 指向的 stable entity；Alias 查询路径不负责每次重新校验 entity merge / split / correction 后的有效性 | 非 exact match、未冻结 normalization / equivalence、semantic / vector similarity、单次模型判断、下游规则结果 |
| Alias initial write eligibility（Alias 初始写入资格） | Entity Log / Entity Resolution 已确认的 stable entity identity，加该交易可追溯 raw surface text | 交易已确认指向明确 stable entity，且写入对象是该交易 raw surface text -> stable entity 关系时，够格成为 durable Alias identity reuse relationship；不需要额外 governance / accountant approval gate；stable entity 创建时同步写入 | 写入执行者、确切写入顺序、字段级 schema、存储或索引实现 |
| Alias trace（Alias 追溯） | evidence refs、confirmation refs、lookup basis；字段名未冻结 | 用于 review、correction、governance 和 audit | accounting classification、rule authority、accountant approval、governance approval |

## 2. 读取者

| Reader | 读取目的 | 可读内容 | 限制 |
| --- | --- | --- | --- |
| Entity Resolution Node（实体识别节点） | 从当前交易 raw transaction description / descriptor / raw bank surface text exact lookup 过去已经确认过的 stable entity；或以某个 entity 为目标查询是否存在一致 Alias | confirmed Alias relationship（已确认别名关系）、Alias source surface text、target stable entity refs、trace refs | Entity Resolution 是 Alias Log 唯一已冻结 reader；exact raw description match 可直接复用 stable entity；Alias 只支持 identity resolution，不能提供 COA / HST / JE / rule / case precedent / automation authority |

Rule Match、Case Log、Case Judgment 或其他下游节点不得直接依赖 Alias Log 作为 rule、case、classification 或 automation basis；如需受益，只能通过 Entity Resolution 输出的 stable entity 再读取各自有 authority 的 memory。

当 Alias lookup 返回多个 stable entity 时，Alias Log 不选择 winner。Entity Resolution 可以继续结合当前 evidence 判断是否能确定唯一 stable entity；若不能，则输出 `unknown + reason`，由后续 Case Judgment / Coordinator 处理。

## 3. 写入者

Alias 的写入资格、审批语义和 stable entity 创建时同步写入语义已经冻结；写入执行者、确切顺序和 finalization mechanism 尚未冻结。

已确认的 authority boundary（权威边界）：

- 写入资格 = 交易已确认指向明确 stable entity，且写入对象是该交易可追溯的 raw transaction surface text -> stable entity 关系。
- 只要满足写入资格，该 Alias relationship 够格自动进入 durable Alias Log；不需要额外 governance / accountant approval gate。
- Alias Log 不重新判断 stable entity 是否成立；stable entity authority 来自 Entity Log / Entity Resolution 已确认 identity。
- 未确认 surface text 不能被写成 Alias authority。
- Entity Resolution 判断为 `new_stable_entity` 时，Entity Log + Alias Log 必须同步 finalization；Entity Log 确认 entity 本体、最小创建 provenance 和初始 entity-centered Alias surface，Alias Log 写入该 confirmed surface text -> stable entity 的反查 projection。

| Writer | 可以写什么 | 写入类型 | 需要 approval 吗 |
| --- | --- | --- | --- |
| 待 L4 / seam 冻结的 memory write / finalization 执行机制 | 已够格的 raw transaction surface text -> stable entity identity reuse relationship | durable write；够格即自动落库；stable entity 创建时同步写入已冻结；具体执行者、调用方式、写入顺序未冻结 | 初始够格写入不需要额外 governance / accountant approval gate；correction / split 重判等治理性变更的审批路径未冻结 |

执行时机的已冻结语义：Alias projection 与 stable entity 创建同步发生，因为两者成立时机一致；exact 写入顺序、事务边界、幂等键和物理存储形态留 L4 / seam。

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

- Authority 条件已冻结：交易已确认指向明确 stable entity，且写入对象是该交易可追溯的 raw transaction surface text -> stable entity 关系。
- 它不要求该 surface text 曾经作为 Entity Resolution 的决定性 evidence。
- 具体由谁执行写入、何时写入、按什么顺序 finalization，仍是 L4 / seam open boundary。

## 5. Lifecycle / States

当前不冻结 Alias lifecycle state enum（生命周期状态枚举），也不设计 Alias 状态机。

Alias Log 不设计“不可用于指代 stable entity”的专门标签，也不为同一 raw description 多 entity 竞争设计独立状态机。

| State | 含义 | 谁可以进入 | 谁可以退出 | 下游含义 |
| --- | --- | --- | --- | --- |
| 已确认对应关系（不是 Alias 状态枚举） | 已确认交易的 raw transaction surface text 指向某个明确 stable entity | 待 L4 / seam 冻结的写入执行机制；前置 authority 条件已冻结 | correction / governance / merge-split finalization 机制未冻结 | Entity Resolution 可以用 exact match 复用 stable entity；不代表 rule、classification、case precedent 或 automation authority |

进入 Alias Log 的前置条件是交易已确认指向明确 stable entity；无法确认唯一主体的交易不得写入 Alias Log authority。如果查询时出现多个 matched stable entity，Alias Log 不选择 winner，而是把多匹配结果交给 Entity Resolution 判断。

## 6. Mutation Path

当前冻结 durable Alias 的资格约束和审批语义；具体执行机制仍未冻结。

已冻结 mutation / authority path：

```text
confirmed transaction -> clear stable entity
+ traceable raw transaction surface text
-> no extra governance / accountant approval gate
-> durable Alias identity reuse relationship
```

该资格不要求 surface text 曾经作为 Entity Resolution 的决定性 evidence；它的用途是让未来 Entity Resolution 可以通过已确认 surface text 复用 stable entity identity。

Merge / split 语义：

- Entity merge 后，Alias relationship 语义上跟随 merge 结果指向新的 stable entity。
- Entity split 后，原 Alias 不自动归属任何一方，必须由治理 / correction 环节按新的 entity 边界重新判断归属。
- 治理层在处理 merge / split 时应同步考虑并更新 Alias Log；Alias Log 查询路径不负责每次重新复查 merge / split 后的有效性。
- merge / split 后的具体执行者、correction 审批路径、batch 操作和 audit trace 属于后续治理 / finalization 机制，不在 Alias Log L2 冻结。

未冻结 mutation path：

- Alias 的写入执行者、确切顺序和 finalization mechanism；与 stable entity 创建同步写入已冻结，exact 顺序、事务边界和幂等机制需待 seam 阶段确认。
- historical Alias conflict 的 correction / governance path。
- Entity split 后逐条 Alias 重新归属的 review path、batch 操作、correction 审批和 audit trace。
- Entity merge / split 时治理层如何同步更新 Alias Log 的执行机制。
- Alias Log 与 Governance Log 的 mutation / audit trace 边界。

例外：

- 无已冻结例外。

## 7. 与其他 memory/log 的边界

只写已确认边界。未确认外部关系进入 Open Boundaries。

| Other Store | 已确认边界 | 未冻结边界 |
| --- | --- | --- |
| Evidence Log（证据日志） | Evidence Log 保存 raw evidence 和 evidence refs；Alias Log 只保存或引用支持 Alias 关系的可追溯来源，不复制原始证据正文；Alias 写入资格不要求 raw surface text 是 stable 判断的决定性 evidence | evidence / confirmation trace 的 exact field schema 尚未冻结 |
| Entity Log（实体日志） | Entity Log 保存 stable entity identity；Alias 只能指向明确 stable entity | Alias Log 与 Entity Log 是独立 store、嵌套记录还是查询投影尚未冻结 |
| Case Log（案例日志） | Case Log 不直接读取 Alias Log；它只能通过 Entity Resolution 输出的 stable entity 间接受益；Alias Log 不保存 case precedent | ER 输出 stable entity 后的具体 case retrieval contract 尚未冻结 |
| Rule Log（规则日志） | Rule Match / Rule Log 不直接读取 Alias Log；Alias Log 不保存 rule condition、active rule 或 accounting treatment | Rule Log 自身的 rule storage / lifecycle 边界不由 Alias Log 冻结 |
| Transaction Log（交易日志） | Transaction Log 保存 final transaction audit record；不是 runtime Alias authority | transaction correction 如何影响 Alias review / mutation 尚未冻结 |
| Governance Log（治理日志） | Governance Log 保存高权限变化和审批历史；初始够格 Alias 写入不需要额外 governance / accountant approval gate | historical conflict correction、split 后重新归属、merge / split 同步更新和 audit trace 边界尚未冻结 |
| Knowledge Summary（知识摘要） | Knowledge Summary 是可读摘要，不能替代 Alias authority | summary conflict repair（摘要冲突修复）尚未冻结 |

## 8. 冲突处理

本轮只冻结 exact raw description match 的 identity reuse authority。非 exact match，包括受控 normalization / equivalence、semantic similarity、vector retrieval 或其他相似匹配，暂不直接提供 stable identity authority；其技术可行性和 authority 边界留后续 matching policy / data contract 阶段讨论。

如果 current raw description exact match 到单一 confirmed Alias relationship：

- authority 顺序：confirmed Alias relationship 可支撑 Entity Resolution 复用其指向的 stable entity；该 authority 只限 identity reuse。
- runtime 行为：Entity Resolution 可以输出 stable entity，并在 reason 中标明 exact Alias match。
- 是否阻断自动化：Alias match 本身不提供 Rule、Case precedent、classification 或 automation permission。
- 是否生成 review / governance candidate：通常不需要；若发现 lifecycle / correction 异常，具体治理路径尚未冻结。

如果 current surface text 与 Alias Log / Alias 库冲突，或同一 surface text 可能对应多个 stable entity：

- authority 顺序：stable entity authority、已确认 Alias relationship 和可追溯 evidence 优先于 LLM semantic match（模型语义匹配）。
- runtime 行为：Alias Log 不选 winner；Entity Resolution 结合当前 evidence 判断能否确定唯一 stable entity，不能确定时输出 `unknown + reason`。
- 是否阻断自动化：不得用冲突 Alias 支持 deterministic path（确定性路径）。
- 是否生成 review / governance candidate：可以生成 issue，但具体处理路径尚未冻结。

如果 current surface text 只是高度类似历史 Alias：

- authority 顺序：confirmed exact Alias relationship（已确认 exact Alias 关系）优先；相似度本身不是 authority。
- runtime 行为：它最多作为 Entity Resolution 的 hint；只有在后续冻结受控 normalization / equivalence 规则、没有冲突、没有多 entity 竞争时，才可能作为更强匹配依据。
- 是否阻断自动化：如果身份仍不安全，应保守阻断相关自动化路径。
- 是否生成 review / governance candidate：未冻结。

如果 Alias Log 与 Entity Log / Governance Log 冲突：

- authority 顺序：stable entity identity 和已生效 governance authority 优先。
- runtime 行为：Entity Resolution 不应忽略 entity lifecycle、merge / split 或 governance constraint（治理限制）；Alias 查询路径不负责每次重新校验 merge / split / correction 后的有效性，durable memory 同步属于治理 / correction / finalization 层。
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

1. Alias 库具体技术形态、索引和存储实现。
2. Alias Log 与 Entity Log 的 storage / projection boundary（存储 / 投影边界）。
3. `alias_record` 的 exact field schema。
4. Alias 写入执行者、确切写入顺序和统一 memory write / finalization 机制；写入资格、“无需额外审批”和“与 stable entity 创建同步写入”已冻结。
5. 受控 normalization / equivalence 规则、银行前缀 / 噪音处理和 vector retrieval 是否只作为 hint。
6. 高度类似历史 Alias 何时可以支持更强 identity match。
7. historical Alias conflict 的 correction / governance path。
8. Entity split 后逐条 Alias 重新归属的执行机制、batch 操作、correction 审批和 audit trace。
9. Entity merge / split 时治理层如何同步更新 Alias Log。
10. Alias mutation 与 Governance Log / audit trace 的边界。

这些问题解决前，不能进入：

- [x] M3 data contract
- [x] M4 storage and operations
- [x] implementation
