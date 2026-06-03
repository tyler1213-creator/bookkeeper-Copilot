# Candidate Entity question

## 文件角色

本文记录 Candidate Entity / Candidate Identity 的已确认结论。

它不是 data contract，不冻结字段 schema、storage implementation 或 writer contract。

## 必要性

系统需要支持一种中间场景：

- 当前 identity 尚未达到 stable entity。
- 当前交易仍可能基于 receipt、invoice、金额模式、业务证据或其他上下文完成分类。
- 完成后的 case 需要保留可追踪的弱身份 handle，避免变成无法按身份复用的孤立案例。
- accountant 不一定能在当前处理时点及时确认 identity。

如果系统要求所有非 stable identity 一律 pending，并等待 accountant 明确确认 identity 后才完成交易，则不需要 durable candidate identity handle。

当前确认保留这个中间场景。

## 已确认结论

- 保留 `candidate identity handle` 支持 completed candidate-linked case。
- 正式命名优先使用 `candidate identity` / `candidate_identity_id`。
- `Candidate Entity` 可作为讨论名，但不作为 stable entity lifecycle state。
- `candidate_identity_id` 与 stable `entity_id` 使用隔离 namespace。
- `candidate_identity_id` 不与 stable `entity_id` 共用语义。
- Candidate identity 持久化在 Entity Log / Identity Layer 内部的隔离区域。
- 该隔离区域可命名为 `candidate_identity_index`。
- 不单独新增完整 `Candidate Identity Log`。
- 不使用 `entity_status=candidate` 表达 candidate identity。
- Entity Resolution 可以读取 `candidate_identity_index`。
- Entity Resolution 的 identity lookup 顺序包含：
  - stable entity match -> `entity_id`
  - confirmed Alias match -> `entity_id`
  - candidate identity match -> `candidate_identity_id`
  - unresolved / ambiguous -> no reusable identity handle
- Case Log 可以引用 `candidate_identity_id`。
- Candidate-linked case 默认只能作为 weak context 或 governance evidence。
- Candidate-linked case 不支持 Rule Match。
- Candidate-linked case 不支持 Rule promotion。
- Candidate-linked case 不支持 Alias 写入。
- Candidate-linked case 不支持 automation 放宽。
- Candidate-linked case 不作为 strong precedent。
- Candidate identity 后续升级为 stable entity 时，必须经过 accountant / governance approval 或其他已确认 stable creation path。
- Candidate identity 升级为 stable entity 后，旧 candidate-linked case 不自动变成 strong precedent。
- Candidate identity 可以累积 linked candidate cases、evidence refs、surface clues 和 risk flags，供 review / governance 使用。

## 最小应保留信息

字段名不冻结，但语义上至少需要支持：

- `candidate_identity_id`
- surface text / normalized identity clues
- evidence refs
- linked candidate case refs
- risk flags
- created_from
- promoted_to_entity_id
- rejected / superseded refs

## 仍未冻结

- Case Judgment 在 candidate identity 下完成本笔交易分类的具体条件。
- `candidate_identity_index` 的 exact field schema。
- `candidate_identity_index` 的 exact writer。
- Candidate identity 的创建时机。
- pending / rejected / not-finalized transaction 是否创建 candidate identity。
- Candidate-linked case 在 Case Log 中的 exact schema。
- Candidate identity 与 stable entity 的 promotion workflow。
- Candidate-linked case 升级为 strong precedent 的 review 条件。
- Candidate identity merge / split / rejection / supersession 行为。
- Candidate -> stable 相关 Case Log / Entity Log / Transaction Log 多 log 写入机制。
