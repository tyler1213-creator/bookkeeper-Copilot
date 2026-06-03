# Entity 相关 Node 和 Log 现存问题

## 文件角色

这是 Entity 相关 node / log 的现存问题速记。

它不是正式 spec，也不冻结任何 data contract。

本文只记录当前需要继续处理的边界问题，后续仍需回填到对应正式文档。

## 问题 A：Stable entity 判断与自动 stable 边界

已确认的核心标准：

> 当前证据能不能直接、清楚、可追溯地说明这个对象是谁？

如果能，并且没有明显身份冲突，就可以考虑 `stable entity`。

判断 stable 只看 identity，不需要证明：

- COA 是否唯一。
- HST / GST 是否可判断。
- 业务用途是否明确。
- 是否可以自动分类或自动落账。
- 是否已有 Rule，或是否可以升级 Rule。

明显身份冲突不能自动 stable。例如同一 surface text 可能对应多个 entity、Alias 关系冲突、描述过泛、内部转账关系不清、payroll / loan / tax / owner / employee / related party 身份关系未确认等。

已确认：`new_stable_entity` 本体可以由 Entity Resolution 同步写入 Entity Log。写入只限 entity 本体；Alias、rule、automation policy 不随本体写入，由后续流程处理。

仍需明确：哪些首次出现的对象可以自动 stable，清晰 bank descriptor 是否单独足够，以及明显身份冲突的操作边界。

## 问题 B：Candidate identity signal / candidate-linked case 的表达、持久化与升级

`candidate entity` 是持久身份线索，不是身份权威。它可以作为 handle 被 Case Log 引用，但 `candidate-linked case` 默认只能作为弱上下文或治理证据。

当前对 Candidate Entity 的必要性判断：

> 如果系统允许“entity 还没有达到 stable，但当前交易仍可以基于其他 evidence 完成会计分类，并把这笔 completed transaction 作为弱记忆保留下来”，那么 Candidate Entity 是必要的。
>
> 如果系统不允许这种情况，而是要求所有非 stable identity 都进入 pending，并等 accountant 明确确认 identity 后再完成交易，那么 durable Candidate Entity 基本不必要，只需要 runtime `candidate_identity_signal` / `identity_issue` 即可。

因此，Candidate Entity 的核心产品作用不是提供 authority，而是处理一种介于 stable entity 和 unknown entity 之间的场景：

```text
unknown entity
= 完全没有可用身份 handle
= 不能被 Case Log 当作 identity handle 引用
= 应进入 pending，由 Coordinator 向 accountant 提问

candidate entity
= 系统有一个疑似身份 handle
= 还不能作为 stable identity authority
= 但当前 evidence 可能足够支持本笔交易完成分类
= 可以形成 candidate-linked completed case
= 未来短期再遇到相似对象时，只能作为 weak context 或治理证据
```

这一层设计成立的前提是：系统不总能保证 accountant 在当前处理时点及时确认 entity identity。对于完全识别不出来的 `unknown_entity`，仍然走 pending；但对于“疑似是某个新对象 / 某个尚未稳定对象”的情况，如果 Case Judgment 可以基于 receipt、invoice、金额模式、业务证据或其他上下文完成本笔会计分类，Candidate Entity 可以让系统保留一个可追踪的弱身份 handle，而不是把这笔 completed case 变成无法按身份复用的孤立记录。

这也是 Candidate Entity 与 Unknown Entity 的根本区别：

- `unknown_entity` 表示当前交易缺少可复用身份基础，不能作为 Case Log identity handle。
- `candidate entity` 表示当前交易有一个可引用但未稳定的身份线索，可以被 Case Log 弱引用。

当前已经确认的边界：

- Candidate Entity 不是 stable entity。
- Candidate Entity 不是 Entity Log authority。
- Entity Log 不保存 durable candidate entity lifecycle state。
- Candidate-linked case 只能作为 weak context 或 governance evidence。
- Candidate-linked case 不能直接支持 Rule Match、Rule promotion、Alias、automation 放宽或 strong precedent。
- Candidate Entity 如果后续要变成 stable entity，必须经过 accountant / governance approval 或其他已确认的 stable creation path。

当前未解决的问题集中在：

- 是否正式保留上述“candidate identity handle 支持 completed candidate-linked case”的产品场景。
- 如果保留，Case Judgment 在什么条件下可以在 candidate entity 下完成本笔交易分类，而不是强制 pending。
- 如果不保留，Candidate Entity 是否应降级为 runtime-only `candidate_identity_signal` / `identity_issue`，不再作为可被 Case Log 引用的对象。
- Candidate identity signal 应如何表达、持久化、引用。
- Candidate-linked case 在 Case Log 中如何记录，避免被误用为 stable authority。
- Candidate 后来升级为 stable 后，旧 candidate-linked case 是否升级为 strong precedent。
- 如果可以升级，升级条件是什么，是否自动升级，如何保留 provenance。
- Candidate -> stable 以及相关 Case Log / Entity Log / Transaction Log 等多 log 写入，由 Entity Resolution、Memory Update / Finalization、Coordinator，还是其他机制执行。

当前倾向：

- 必须保留的不是“Candidate Entity 作为 Entity Log 里的正式生命周期状态”，而是“可被 Case Log 弱引用的 candidate identity handle”。
- 只有当系统确认需要支持“人工不能及时确认 identity，但当前交易可以完成分类并形成弱案例记忆”的场景时，Candidate Entity 才具备独立存在的充分理由。
- 否则，Candidate Entity 应被删除或降级为 runtime / review / governance context，避免引入一个没有真实运行场景的长期对象。
