# Review Inbox Question（审核 inbox）

## 文件角色

本文件记录审核 inbox 数据层的当前草案，用于承接确定性发现 job 产出的候选，并供 Human Review Node 主动拉取消费。

它不是正式 spec，不冻结字段、状态机、去重、权限或存储实现。后续仍需送审计并决定是否落为正式 `BK_Copilot/` data layer / memory layer 文档。

## 一句话定位

**审核 inbox = 持久候选队列，不是 authority，不是 Governance Log。**

它保存“系统认为值得给会计师看的候选”，当前 MVP 只承接确定性发现 job 产生的 `rule_promotion_candidate`。候选还没被会计师确认，所以不能写进 Governance Log；候选也不是 executable rule，所以不能写进 Rule Log。

## 当前生产者与消费者

当前生产者：

- Deterministic Discovery job：自主非 LLM job，当前只写 `rule_promotion_candidate`。

当前消费者：

- Human Review Node：会计师打开复核 / 治理界面时主动 pull，展示候选和证据。Human Review Node 不自己扫描 logs；手动 rescan 也只是调用 producer job。

未来可能生产者：

- 未来 LLM 审查节点。当前不存在；若新增，必须同样只写 candidate，不写 authority。

## MVP 数据形态

MVP 只需要 append 候选记录，不设计完整队列生命周期。

候选记录至少需要表达：

- candidate type：当前仅 `rule_promotion_candidate`。
- producer：例如 `deterministic_discovery_job`。
- target：stable `entity_id`，以及 Rule 侧定义的 scope / pattern seed。
- proposed treatment：供会计师确认的候选 rule 输出；exact shape 归 Rule / JE Generator L3。
- evidence refs：CaseLogEvidence refs，且每条 evidence 必须能回到 finalized transaction proof。
- predicate refs：Rule 侧判据 id / version / run context，用来解释“为什么这次被捞出来”。
- discovered_at / run_id / trigger kind：区分批后增量、定时全扫、手动 rescan。

MVP 暂不冻结：

- candidate status（pending / accepted / rejected / deferred / superseded）。
- 去重、压制已拒绝项、过期。
- review assignment / priority。
- candidate_key / 幂等键。

这些不代表不重要，只是 MVP 不作为当前 contract。生产化第一批要补状态 / 去重 / 过期，否则定时 job 会重复冒泡。

## 写入与 authority 边界

写入：

- candidate 写入应走统一 Finalization / durable write 库的 schema 和幂等校验能力，但**免 accountant credential**，因为候选不是扩张型权威变更。
- candidate 写入不能绕过 durable persistence；runtime-only candidate 会在无人在线时丢失。

不能成为：

- Rule Log executable authority。
- Entity Log / Rule Log / Case Log mutation。
- Governance Log approval event。
- accountant approval。

一旦会计师确认 rule promotion：

```text
Human Review Node displays candidate + evidence
-> accountant read-back / sign-off
-> Rule side fixed execution path validates and builds approved rule payload
-> Finalization writes Rule Log approved rule + Governance Log audit event
```

inbox 本身不执行上述路径，也不拥有放行权。

## Open Boundaries

- inbox 是否作为正式 memory layer / data layer 落入 `BK_Copilot/`，命名和模板归属未定。
- exact schema、candidate id、refs 形态、query API 和权限未定。
- status / dedupe / expiry / suppression 机制未定，MVP 搁置。
- 多 producer 同写时的隔离、排序、幂等和权限未定。
- Human Review Node pull 的 projection / pagination / filtering contract 未定。
