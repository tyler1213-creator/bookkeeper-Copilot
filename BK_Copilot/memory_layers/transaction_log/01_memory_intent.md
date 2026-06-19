# Transaction Log - Memory Intent

## 1. 为什么这层 memory 必须存在

它解决的问题是：

> 为每一笔被赋予 `transaction_id` 的交易提供逐笔、无损、append-only 的全过程审计 source of truth。

如果删除它，会失去：

- 事后逐笔追溯一笔交易从进入系统到最终结果之间经历了什么的能力。
- 在 final outcome、最终 COA / HST-GST、confirmed_by 审计留痕发生冲突时，定位最终交易审计真值的能力。
- 查看 terminal / blocked / 无 JE / 情况 X 或情况 Y 交易最终归处与处理路径的能力。
- 通过 entity 派生查询某主体名下全部历史交易的能力；该查询只是逐笔记录上的访问能力，不另设 entity 索引存储层。

这些能力不能由 runtime handoff 或现有 store 覆盖，因为：

- runtime handoff 是瞬时传递，处理完即逝，不能承担 durable audit trail。
- Entity Log、Case Log、Alias Log 均拒绝承载逐笔交易历史与完整 audit trail：Entity Log 负责身份，Case Log 负责可复用学习先例，Alias Log 负责 alias / identity 辅助关系。
- Evidence Log 只证明证据存在性，不记录交易处理结果、最终会计结果或完整 processing path。
- Intervention Log 负责交互事实；Transaction Log 负责最终交易审计，两者不能互相替代。

## 2. 保存什么

这层 memory 保存以下语义；exact 字段名 / enum / schema / refs 形态 / 阈值 / 存储形态 / 写入执行者 / 顺序均留 M3 或 L4 / seam：

| # | 保存语义 | 本阶段已定含义 |
| --- | --- | --- |
| 1 | `transaction_id` | 逐笔交易主索引，来自 Evidence Intake 铸号。 |
| 2 | objective transaction basis | 金额、方向、日期、账户、description 等交易客观基础快照；`transaction_id + basis + evidence binding` 应在 Transaction Log finalization 持久化。 |
| 3 | evidence binding / evidence refs | 指向 Evidence Log 的证据引用；保存指针，不保存证据原文。 |
| 4 | 身份字段 | 这笔交易归属的具体 entity，或 `unknown`。身份只有具体 entity / `unknown` 两类；`unknown` 不阻塞入库，也不是 candidate identity。 |
| 5 | 情况 Y 身份待补线索 | 身份未知但分类完成的交易，其身份待补线索落点已定为 Transaction Log；该线索不阻塞 finalization，字段形态留 M3。 |
| 6 | full processing path | 交易经过哪些环节、各环节如何处理的全过程痕迹，包括 ER 结果、rule-hit、分类、交互、JE、拦截等语义。 |
| 7 | final outcome | 最终交易结果，含最终 COA 科目、HST / GST 处理、JE 结果或 terminal 状态。 |
| 8 | 最终 COA / HST-GST 值 | 最终会计分类结果；本层保存该笔交易最终审计真值。 |
| 9 | rule-hit ref | 这笔交易最终由哪条 rule 处理的 source 语义与 rule ref；exact 字段形态留 M3。 |
| 10 | reasoning / audit narrative | Case Judgment 等环节的结构化 reasoning 与审计叙事留痕；仅 trace，非 authority。 |
| 11 | `confirmed_by` 留痕 | 结论由谁确认的具体人员与完整审计留痕。 |
| 12 | AI 高置信度自动完成标记 | High Confidence 自动完成的标注；字段形态留 M3。 |
| 13 | 更正记录 | 后续 review / 治理更改后的追加记录，表达更改者与更改后的最终会计分类结果；record schema 与触发机制留 L4 / seam。 |

覆盖面：

- 凡被赋予 `transaction_id` 的交易一律进入 Transaction Log。
- 覆盖正常 finalize、terminal、blocked、无 JE、情况 X、情况 Y 身份未知但分类完成的交易。
- 本层无损、append-only，不丢弃、不就地覆盖任何交易记录。
- terminal / blocked / 无 JE 以最终状态记录；状态 enum 和字段形态留 M3。

其中可以成为审计 authority 的是：

- final outcome。
- 最终 COA / HST-GST 值。
- `confirmed_by` 审计留痕。

只作为 trace / context / explanation 的是：

- reasoning / audit narrative。
- full processing path 中用于复盘的过程痕迹。
- AI 高置信度自动完成标记。
- 情况 Y 身份待补线索；它是复查线索，不是 candidate identity。

## 3. 绝不保存什么

这层 memory 绝不保存：

- 可复用学习先例；该职责归 Case Log。
- runtime identity / learning authority。
- candidate entity、candidate identity，或把 `unknown` / unresolved 当成第三类身份状态。
- 用于未来自动决定的可复用 reasoning authority。
- 按 entity 索引的学习层或流水副本；按 entity 查询只是对逐笔记录的派生访问能力。
- 证据原文；本层只保存 evidence binding / refs 语义。

它也不能替代：

- Case Log：可复用学习先例的 source of truth。
- Entity Log：具体身份与 stable entity 的 source of truth。
- Intervention Log：交互事实的 source of truth。
- Evidence Log：证据存在性与证据保存状态的 source of truth。

## 4. 对核心产品目标的贡献

这层 memory 必须清楚支持以下至少一项：

- [ ] 记忆复用
- [ ] 有证据支持的建议
- [ ] accountant correction learning
- [x] 审计性
- [x] accountant control
- [ ] 自动化率提升

具体贡献：

- 审计性：每一笔交易都有可追溯的进入、处理、finalization、terminal / blocked / correction 记录。
- accountant control：最终交易结果、最终 COA / HST-GST 与 confirmed_by 留痕能回答“谁确认了什么，以及最终审计结果是什么”。

## 5. 已知约束

- 交易处理层任何节点不读 Transaction Log；它永不作为 runtime decision / identity / learning source。
- governance / 治理环节可以只读本层作为 context / 线索，但不能把 reasoning 或过程叙事继承为 reusable authority。
- 未来 agent 工具读取 Transaction Log 只列为方向，不冻结为当前 contract；读取 retrieval / projection / permission 机制留 L4 / seam。
- 任何业务节点绝不裸写 Transaction Log，只声明要持久化的语义；统一 finalization 写入机制负责落盘，exact writer 与顺序留 L4 / seam。
- 写入 Transaction Log 本身不设额外 governance / 会计师审批闸；本层忠实记录上游已经定下来的结果。
- 本层无 candidate 层；写入即 finalization 定稿，不设待批 / 候选状态。
- 记录无稳定状态机；finalized / terminal / blocked 等是内容字段，enum 留 M3。
- 字段级 schema、enum、refs、阈值、存储形态、写入执行者和顺序不得在 M1-M2 冻结。

## 6. 未决定问题

- Transaction Log record 的 exact 字段名 / schema / validation，含本文件第 2 节 13 项语义的精确形态。
- reasoning 在 Transaction Log 的 exact 存储字段，以及后续治理节点读取 reasoning 的 contract。
- rule-hit source / rule ref 在 Transaction Log 的 exact 字段形态。
- 身份字段含 `unknown` 标识、情况 Y 身份待补线索的 exact 字段形态。
- terminal / blocked / 无 JE 交易最终状态的 exact enum / 字段。
- final outcome、COA、HST / GST、full processing path 各环节痕迹的 exact 字段结构。
- `confirmed_by` 审计留痕、AI 高置信度自动完成标记的 exact 字段。
