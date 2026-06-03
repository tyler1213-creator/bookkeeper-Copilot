# Entity Log

## 文档状态

- Current standard status: draft
- Covered stages:
  - [x] M1: Memory Intent
  - [x] M2: Authority, Lifecycle, and Boundaries
  - [ ] M3: Data Contract
  - [ ] M4: Storage and Operations
  - [ ] M5: Tests and Fixtures
- Last reviewed: 2026-05-11
- Owner / reviewer: system audit session / user review required

## 一句话定义

`Entity Log`（实体日志）是 identity authority store（身份权威存储）：它保存客户账务语境中 counterparty / vendor / payee（交易对手 / 商家 / 收款人）是谁、哪些已确认 transaction surface text 可以通过 Alias 指向该 stable entity、处于什么生命周期状态，以及当前实体级自动化权限是什么。

它不保存 classification memory（分类记忆）、case precedent（案例先例）、rule condition（规则条件）、journal entry（分录）或 transaction audit trail（交易审计轨迹）。

## 当前标准文件

- `01_memory_intent.md`
- `02_authority_lifecycle_and_boundaries.md`

## 当前已冻结结论

- `Entity Log`（实体日志）独立于 `Case Log`（案例日志）和 `Transaction Log`（交易日志）。
- `Entity Log`（实体日志）回答“这是谁”和“身份权威是什么”，不回答“怎么记账”。
- `Entity Log`（实体日志）只保存 stable entity（稳定实体）记录；没有 durable candidate entity lifecycle state（长期候选实体生命周期状态）。
- `Entity Resolution Node`（实体识别节点）判断为 `new_stable_entity`（新稳定实体）时，可以同步写入 Entity Log，不需要 governance approval（治理批准）；写入内容限于 entity 本体：`entity_id`、`display_name`、`entity_type`、`entity_status=active`、`evidence_links`、`created_from`。
- `new_stable_entity`（新稳定实体）写入不等于写入 Alias（别名）、放宽 automation policy（自动化策略）或创建 active rule（生效规则）。
- `Case Log`（案例日志）可以为 entity-level risk（实体级风险）或 automation policy candidate（自动化策略候选）提供依据，但不能直接修改 `Entity Log`（实体日志）。
- `Transaction Log`（交易日志）不能作为 runtime entity authority（运行时实体权威）或 learning source（学习来源）。
- Alias（别名）是过去已经确认过的 transaction surface text 和 stable entity（稳定实体）的对应关系。
- 当前只确认两类信息可能成为 Alias：bank statement 中每笔交易的 description / descriptor / raw bank surface text；以及当 bank description 本身没有明确身份意义时，其他可能重复出现并能指向交易主体的字段，例如 cheque payee。
- 当前确认需要一个可被 Entity Resolution（实体识别）查询的 Alias 库；Alias 库具体技术形态尚未冻结。
- `new_entity_candidate`（新实体候选）不能成为 reusable authority（可复用权威），除非经过 accountant / governance approval（会计师 / 治理批准）。
- `automation_policy`（自动化策略）属于 entity-level authority（实体级权威），升级或放宽必须经 accountant approval（会计师批准）；系统自动降级可以在受控边界内生效，但必须保留治理可见性。

## 当前未冻结边界

- `entity_record`（实体记录）的 exact field schema（精确字段结构）尚未冻结。
- Alias 库具体以什么技术形态呈现尚未冻结。
- `alias_record`（别名记录）是否作为嵌套字段、独立记录或由查询库投影呈现尚未冻结。
- candidate identity signal（候选身份信号）如需持久化，落点尚未冻结；但它不作为 Entity Log 中的 durable candidate entity lifecycle state（长期候选实体生命周期状态）。
- `automation_policy_downgrade_path`（自动化策略自动降级路径）的 exact mutation contract（精确变更契约）尚未冻结。
- `merge_split_supersession_behavior`（实体合并 / 拆分后的替代关系行为）尚未冻结到字段级。

## 进入下一阶段前必须解决

- 确定哪些 fields（字段）属于 M3 data contract（数据契约）必须冻结。
- 确定 stable entity lifecycle states（稳定实体生命周期状态）的最小必要 enum（枚举）。
- 确定 Alias / automation policy mutation（别名 / 自动化策略变更）的 exact approval path（精确批准路径）。
- 确定 `Entity Log`（实体日志）与 `Governance Log`（治理日志）的 projection boundary（投影边界）：哪些状态直接存于 Entity Log，哪些只从 Governance Log 投影。
