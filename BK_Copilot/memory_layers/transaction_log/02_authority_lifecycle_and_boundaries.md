# Transaction Log - Authority, Lifecycle, and Boundaries

## 1. Source of Truth / Authority

只填写当前已经稳定的 authority。未稳定的字段形态、读取机制和跨 store 传播进入 Open Boundaries。

| Concept | Source of Truth | Authority Rule | Not Authority |
| --- | --- | --- | --- |
| final outcome | Transaction Log，或等价 finalization source | 对一笔交易最终变成什么，本层是审计 source of truth；冲突时对 final outcome 优先。 | reasoning / audit narrative、runtime handoff、缺 finalization proof 的片段。 |
| 最终 COA / HST-GST 值 | Transaction Log，或等价 finalization source | 对该笔交易最终会计分类结果，本层保存最终审计真值。 | rule-hit 过程痕迹、AI reasoning、Case Log 中缺少有效 finalization proof 的片段。 |
| `confirmed_by` 审计留痕 | Transaction Log | 对“谁确认了该最终结论”的完整审计留痕，本层优先；Case Log 只可保存较粗的确认类型。 | 仅用于解释的 narrative、未确认的候选语义。 |
| reasoning / audit narrative | Transaction Log 保存 trace | 可供 review、correction、governance、audit 阅读；不能成为未来自动决定或可复用判断依据。 | reusable authority、accountant approval、governance approval。 |
| full processing path | Transaction Log 保存 trace | 用于复盘一笔交易经过哪些环节与如何处理。 | runtime source、identity source、learning source。 |

## 2. 读取者

本表是本层对外声明的读取接口面：reader 只能依赖此处声明的内容，不得依赖本层内部实现。

| Reader | 读取目的 | 可读内容 | 限制 |
| --- | --- | --- | --- |
| 交易处理层节点 | 无；运行期隔离。 | 无。 | Evidence Intake、Entity Resolution、Rule Match、Case Judgment、Coordinator、JE 生成等交易处理层节点一律不读 Transaction Log；本层不作为 runtime decision / identity / learning source。 |
| governance / 治理环节 | review、correction、governance、audit 时查看逐笔交易最终结果与处理痕迹。 | final outcome、最终 COA / HST-GST、confirmed_by 审计留痕、full processing path、reasoning / audit narrative、更正追加记录。 | 只读，非 authority producer；不得把 reasoning / audit narrative 当 reusable authority、accountant approval 或 governance approval。 |
| 未来 agent 工具 | 作为 context / 线索辅助查找、解释或复查。 | 方向上可读取逐笔审计记录与派生 entity 历史查询结果。 | 列为方向，不冻结为当前 contract；读取后仍须重新判断，不得继承 reasoning 为 authority。 |

retrieval / projection / permission 机制是 L4 / seam open boundary。

## 3. 写入者

本表描述“谁执行写入 + 用什么机制”（运行 / 记忆 seam）；至于“写什么、是否够格、什么算 authority”，来源是发起写入的运行节点 spec，本层不重新判断。

| Writer | 可以写什么 | 写入类型 | 需要 approval 吗 |
| --- | --- | --- | --- |
| 业务 workflow node | 不能直接写 Transaction Log；只能声明要持久化的语义。 | 不适用。 | 不适用；节点不得裸写。 |
| 统一 finalization 写入机制（exact writer 未冻结） | 上游已经定稿的逐笔交易审计记录，包括 final outcome、最终 COA / HST-GST、confirmed_by 留痕、processing path、terminal / blocked / 无 JE 最终状态、情况 X / Y finalization。 | direct finalization。 | 否。本层不设额外 governance / 会计师审批闸；上游该审的已经审完，本层忠实记录已定结果。 |
| review / governance 更正路径（exact trigger / writer 未冻结） | 更正追加记录，表达更改者与更改后的最终会计分类结果。 | correction append。 | 本层不在 M1-M2 冻结 approval 机制；correction / governance 触发与跨 log 传播留 L4 / seam。 |

exact writer、finalization trigger order、多 log 统一写入机制均为 L4 / seam open boundary。

## 4. Candidate 边界

Transaction Log 无 candidate 层。

- 写入即 finalization 定稿，不设“先提候选、待批准再生效”的环节。
- 本层不设待批状态、不设候选记录状态、不发明审批流程。
- 身份字段只有具体 entity / `unknown` 两类；`unknown` 不阻塞入库，也不是 candidate identity。
- 情况 Y 身份待补线索是复查线索，不是 candidate entity / candidate identity。

## 5. Lifecycle / States

Transaction Log 记录无稳定状态机。

- 记录写入即固定，不做状态流转。
- finalized、terminal、blocked、无 JE 等是交易最终结果或处理内容字段，不是记录自身的 lifecycle state。
- 要更正不是改某条记录状态，而是 append 新记录。
- finalized / terminal / blocked / 无 JE 的 exact enum 与字段形态留 M3。

## 6. Mutation Path

已冻结 mutation path：

```text
upstream finalized transaction semantics
-> unified finalization write mechanism (exact writer / order open)
-> append Transaction Log record
-> audit trace
```

```text
review / governance correction result
-> correction append path (exact trigger / writer / schema open)
-> append new Transaction Log record with changer + corrected final accounting result
-> audit trace
```

已冻结原则：

- Transaction Log 为 append-only；原始记录永不就地覆盖或删除。
- 后续 review / 治理更改某 entity 或其名下交易分类时，追加一条新记录注明更改者与更改后的最终会计分类结果，而非改写原记录。
- 写入 Transaction Log 本身不设额外 governance / 会计师审批闸；本层记录上游已定结果。

未冻结 mutation path：

- correction / reversal / split / supersession 的 exact 触发机制。
- correction record schema。
- 改 entity 后扇出到其名下交易的跨 log 传播机制。
- “谁改 + 改后结果”的 exact 留痕形态。
- reversal / split 的完整产品机制，本轮 DEFERRED。

例外：

- 无。

## 7. 与其他 memory/log 的边界

只写已确认边界。不确定的外部边界进入 Open Boundaries。

| Other Store | 已确认边界 | 未冻结边界 |
| --- | --- | --- |
| Evidence Log | Evidence Log 证明证据存在性；Transaction Log 保存 evidence binding / refs 与最终交易审计，不保存证据原文。 | Evidence Log 正式 spec、evidence refs exact 字段、Evidence Log 与 Transaction Log finalization 顺序。 |
| Intervention Log | 交互事实归 Intervention Log；最终交易审计、final outcome、processing path 归 Transaction Log。 | Intervention Log 正式 spec、交互事实进入 Transaction Log processing path 的 exact ref / projection contract。 |
| Entity Log | Entity Log 是具体身份与 stable entity 的 source of truth；Transaction Log 只保存该笔交易的具体 entity 或 `unknown` 审计内容。情况 X / Y 只 finalize 到 Transaction Log，不进 Entity Log。 | 身份字段 exact 形态；entity 更改后如何扇出影响名下交易、Alias review / mutation、entity risk candidate。 |
| Case Log | Case Log 是可复用学习先例的 source of truth；Transaction Log 是最终交易审计 source of truth。Case Log 通过 `transaction_log_ref` / finalization proof 引用 Transaction Log，不存完整 processing path。情况 X / Y 不进 Case Log。 | Case Memory Update Node 凭 finalization proof 写 Case Log 的 exact 机制与 trigger order。 |
| Rule Log | Rule Log 是 approved rule 的 source of truth；Transaction Log 可保存本笔交易的 rule-hit source 语义与 rule ref 作为审计痕迹。 | rule-hit ref exact 字段形态；rule-hit 统计派生 / rollup / cache / maintenance job。 |
| Governance Log | Governance / Governance Review 可只读 Transaction Log 做 review、correction、governance、audit context；Transaction Log 不把 reasoning 变成 governance approval。 | Governance / Governance Review 正式 spec；governance 读取 Transaction Log 的 exact retrieval / projection / permission；correction path 与跨 log 传播。 |

四 Log 分界：

- 交互事实 -> Intervention Log。
- 身份 -> Entity Log。
- 可复用先例 -> Case Log。
- 最终交易审计 -> Transaction Log。

四者互不替代。

## 8. 冲突处理

如果本 memory 与 Case Log / finalization proof / runtime 片段冲突：

- authority 顺序：对 final outcome 与 audit trail，Transaction Log（或等价 finalization source）优先。
- runtime 行为：交易处理层不读 Transaction Log，因此本冲突处理不回流为 runtime decision。
- 是否阻断自动化：缺有效 finalization proof 的片段不能作为下游可复用证明；是否阻断某个未来自动化路径取决于对应对象 spec。
- 是否生成 review / governance candidate：未冻结；进入 Open Boundary，不写成当前标准。

如果冲突规则依赖其他未定 spec：

- 标记为 Open Boundary。
- 不得写成当前标准。

## 9. Audit / Trace 边界

这层 memory 必须留下的 trace：

- full processing path。
- reasoning / audit narrative。
- `confirmed_by` 审计留痕。
- terminal / blocked / 无 JE 的最终状态语义。
- 更正追加记录，含更改者与更改后的最终会计分类结果。
- 情况 Y 身份待补线索：身份未知但分类完成的交易，其复查线索落点已定为本层；它是不阻塞 finalization 的复查线索，不是 candidate identity / stable entity authority / 第三类身份状态。

这些 trace 用于：

- review。
- correction。
- governance。
- audit。

这些 trace 不能成为：

- reusable authority。
- accountant approval。
- governance approval。
- runtime identity / learning source。

## Open Boundaries

### L3（字段 / enum / schema）

- Transaction Log record 的 exact 字段名 / schema / validation，含 13 项保存语义的精确形态。
- reasoning 在 Transaction Log 的 exact 存储字段，以及后续治理节点读取 reasoning 的 contract。
- rule-hit source / rule ref 在 Transaction Log 的 exact 字段形态。
- 身份字段含 `unknown` 标识、情况 Y 身份待补线索的 exact 字段形态。
- terminal / blocked / 无 JE 交易最终状态的 exact enum / 字段。
- final outcome、COA、HST / GST、full processing path 各环节痕迹的 exact 字段结构。
- `confirmed_by` 审计留痕、AI 高置信度自动完成标记的 exact 字段。

### L4 / seam（机制）

- Transaction Log 的 exact writer、统一 finalization 写入机制、多 log finalization 顺序（Evidence / Transaction / Case / Rule / JE）。
- append-only 更正记录的 exact 机制：触发者、correction record schema、改 entity 后扇出到其名下交易的跨 log 传播，以及“谁改 + 改后最终分类结果”的 exact 留痕形态。
- reversal / split / supersession 对 Transaction Log 的执行机制。
- 统计派生 / rollup / cache / maintenance job。
- governance / 未来 agent 工具读取 Transaction Log 的 exact retrieval / projection / permission 机制。
- entity-index 查询的 exact 索引 / 检索机制。

### L2·外阻（圈外依赖）

- JE Generation Node 与 Transaction Log 的 exact handoff（JE result / journal_entry 留痕）。
- Intervention Log 与 Transaction Log 的分界 exact contract。
- Review Node 是否覆盖所有 final logging 前交易，以及 review trace 进 Transaction Log 的 exact 边界。
- Case Memory Update Node 凭 `transaction_log_ref` / finalization proof 写 Case Log 的 exact 机制与 trigger order。
- Transaction Log 与对外 final output report / export artifact 的 field-level 边界。
- Governance / Governance Review、Knowledge Summary / Knowledge Compilation、Evidence Log、Post-Batch Lint、Profile / Structural Match 与 Transaction Log 的 exact 读写边界。
- transaction correction 如何影响 Alias review / mutation、entity risk candidate。

### DEFERRED

- correction / reversal / supersession / split 的完整产品机制。本轮只锁 append-only 原则与更正追加语义。
- 跨导入 `transaction_id` 复用、迟到证据接回、durable transaction registry、跨运行账务幂等。
