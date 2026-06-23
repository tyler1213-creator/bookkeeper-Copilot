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

`Entity Log`（实体日志）是 stable entity（稳定实体）的 identity authority 主档案：它以 stable entity 为主体，保存客户账务语境中某个 entity 是谁、其生命周期状态、已确认身份相关属性、authority trace、evidence refs 和 governance refs。

`Entity Log` 不保存 classification memory（分类记忆）、case precedent（案例先例）、rule condition（规则条件）、journal entry（分录）、transaction audit trail（交易审计轨迹）或未经批准的 AI reasoning（模型推理）。

## 当前标准文件

- `01_memory_intent.md`
- `02_authority_lifecycle_and_boundaries.md`

## 当前已冻结结论

- `Entity Log`（实体日志）独立于 `Case Log`（案例日志）和 `Transaction Log`（交易日志）。
- `Entity Log`（实体日志）回答“这是谁”和“身份权威是什么”，不回答“怎么记账”。
- `Entity Log`（实体日志）只保存 stable entity（稳定实体）记录。
- `Entity Log` L2 保存集合只分五类：身份脊梁、身份状态与重定向、身份复用面、控制状态、追溯指针；五类各自字段、enum 和 refs 形态留 M3。
- Alias（别名）的 entity-centered 语义归 `Entity Log`：某个 stable entity 名下有哪些已确认 Alias，是该 entity 身份权威状态的一部分。`Alias Log` 是面向 `Entity Resolution` 的 alias -> entity 反向查询 projection / index，不是第二套 stable entity authority。
- `automation_policy`（自动化策略）的 entity 级控制语义归 `Entity Log`；支撑证据归 `Case Log`，审批事件历史归 `Governance Log`。升级或放宽必须经 accountant approval（会计师批准）；系统自动降级只限受控收紧并保留治理可见性。
- `risk_flags`（风险标记）只保存会在 runtime 当卡点使用的身份级风险，例如疑似同名、疑似应 merge、identity conflict。分类不稳定、记账结果漂移等 case-derived risk 不进入 Entity Log，归 `Case Log` / governance candidate。
- `Entity Resolution Node`（实体识别节点）判断为 `new_stable_entity`（新稳定实体）后，发起及时 Entity Log publication；该入口不需要 governance approval（治理批准），且只确认 entity 本体和最小创建 provenance，字段名与 provenance schema 留 M3，实际写入执行者 / 顺序留 L4 / seam。
- accountant explicit identity confirmation（会计师明确身份确认）是另一条 stable entity 创建入口；同样不需要 governance approval，只确认 entity 本体和最小创建 provenance。
- stable entity 创建不隐含 Alias、Rule、automation policy、Case Log 或 final transaction outcome 写入；accountant 只确认分类、未明确确认 identity 时，不创建 stable entity。
- stable entity lifecycle 的 L2 最小状态语义已定为 active / merged / archived；exact enum、validation、退出 / 回滚规则留 M3 / L4。Lifecycle state 不代表自动分类许可，automation_policy 是独立控制状态。
- merge / split 后，`Entity Log` 只保存 Entity 侧当前身份状态、surviving entity / supersession 指向和 governance refs；Alias、Case、Rule 的迁移、重判、阻断或 finalization 归对应 log / governance 机制。
- person（个人）、vendor、organization、payee 等 entity_type 使用同一套 Entity Log 准入标准；准入边界是是否经 `Entity Resolution` 或 accountant explicit identity confirmation 成为 stable entity，不因 person / Profile 存在而特殊排除。
- `Case Log`（案例日志）可以为 automation policy candidate（自动化策略候选）或治理候选提供依据；分类不稳定等 case-derived risk 不直接进入 Entity Log risk_flags，也不能直接修改 `Entity Log`（实体日志）。
- `Transaction Log`（交易日志）不能作为 runtime entity authority（运行时实体权威）或 learning source（学习来源）。

## 当前未冻结边界

- `entity_record`（实体记录）的 exact field schema（精确字段结构）尚未冻结。
- 身份脊梁、身份状态与重定向、身份复用面、控制状态、追溯指针五类信息各自的 field schema、enum（含 entity_status、automation_policy、risk 标记取值）和 refs 形态尚未冻结。
- 最小创建 provenance、authority trace、evidence refs、governance refs 的字段形态尚未冻结。
- `Alias Log` 具体以什么技术形态呈现，以及它与 `Entity Log` 的物理存储、projection / index 构建、同步方式和写入顺序尚未冻结。
- `automation_policy_downgrade_path`（自动化策略自动降级路径）的 detection、exact mutation contract（精确变更契约）和治理可见性尚未冻结；该检测归 Entity Log automation_policy maintenance，不归当前确定性发现 job。
- `automation_policy` 是直接存于 `Entity Log` 还是由 `Governance Log` 投影生成，属于存储 / projection 形态，尚未冻结。
- ER `new_stable_entity` 与 accountant explicit identity confirmation 之后的实际写入执行者、调用方式、写入顺序和多 log finalization 机制尚未冻结，但必须保留“无需 governance approval”和“及时可见”的 L2 事实。
- merge / split 的治理审批、批量重判、跨 log finalization、audit trace 和回滚机制尚未冻结。
- Profile 自身结构事实如何建模，以及未来是否与 Entity Log 建立 ref / projection，留给 Profile 或联合 L3 / L4。

## 进入下一阶段前必须解决

- M3：冻结 `entity_record`、五类保存信息、trace refs、entity_status / automation_policy / risk 标记等 exact field / enum / refs 形态。
- M3 / L4：冻结 Profile 与 Entity Log 如需互相引用时的 ref / projection 形态；Profile 内部结构事实建模不属于 Entity Log L2。
- L4 / seam：冻结 `Entity Log` 与 `Alias Log` 的 projection / index / 同步机制，以及 stable entity 创建后的实际写入执行者、顺序和多 log finalization。
- L4 / seam：冻结 automation_policy 存储 / 投影形态、自动降级 detection + exact approval / visibility contract，以及 merge / split 的跨 log finalization / audit / rollback 机制。
