# Entity Log - Authority, Lifecycle, and Boundaries

## 1. Source of Truth / Authority

只填写当前已经稳定的 authority（权威）。未稳定的关系进入 Open Boundaries（未冻结边界）。

| Concept | Source of Truth | Authority Rule | Not Authority |
| --- | --- | --- | --- |
| entity identity（实体身份） | `Entity Log`（实体日志）加 traceable evidence refs（可追溯证据引用）和 governance refs（治理引用） | active entity（有效实体）可作为 runtime identity target（运行时身份目标） | display text（展示文本）、Knowledge Summary（知识摘要）、LLM rationale（模型理由） |
| approved alias（已批准别名） | `Entity Log`（实体日志），由 accountant / governance approval（会计师 / 治理批准）支持 | 可支持 Entity Resolution（实体识别）和 Rule Match（规则匹配）的 alias authority（别名权威） | candidate alias（候选别名）、normalized display text（标准化展示文本） |
| rejected alias（已拒绝别名） | `Entity Log`（实体日志），由 accountant / governance rejection（会计师 / 治理拒绝）支持 | 作为 negative authority（负向权威）阻止相似度误匹配 | LLM semantic similarity（LLM 语义相似度） |
| confirmed role（已确认角色） | `Entity Log`（实体日志）或受控 onboarding accountant-derived source（初始化会计历史来源） | 可支持需要该 role / context（角色 / 上下文）的 downstream judgment（下游判断） | runtime `candidate_role`（运行时候选角色） |
| entity_status（实体生命周期状态） | `Entity Log`（实体日志）加 approved governance projection（已批准治理投影） | 限制 entity（实体）是否可作为 active identity target（有效身份目标） | transaction outcome（交易结果）、case pattern（案例模式） |
| automation_policy（自动化策略） | `Entity Log`（实体日志）和 / 或 Governance Log projection（治理日志投影），具体字段边界未冻结 | 限制 entity-level automation（实体级自动化） | active status（有效状态）、case frequency（案例频率） |
| entity risk flags（实体风险标记） | `Entity Log`（实体日志），由 governance / lint / review path（治理 / 体检 / 审核路径）产生 | 提醒下游存在 identity risk（身份风险）或 automation risk（自动化风险） | 直接分类结论、rule authority（规则权威） |

## 2. 读取者

| Reader | 读取目的 | 可读内容 | 限制 |
| --- | --- | --- | --- |
| Entity Resolution Node（实体识别节点） | 判断当前 evidence（证据）是否指向 stable entity（稳定实体） | entity identity（实体身份）、aliases（别名）、roles（角色）、status（状态）、risk flags（风险标记）、automation policy（自动化策略） | 不能修改 Entity Log；不能把 candidate（候选）当 authority（权威） |
| Rule Match Node（规则匹配节点） | 判断 rule eligibility（规则匹配资格）的 identity prerequisites（身份前置条件） | active entity（有效实体）、approved alias（已批准别名）、confirmed role（已确认角色）、automation policy（自动化策略） | 必须自行读取 Rule Log（规则日志）；Entity Log 不提供 rule condition（规则条件） |
| Case Judgment Node（案例判断节点） | 使用 entity context（实体上下文）和 automation boundary（自动化边界） | identity basis（身份基础）、role context（角色上下文）、risk flags（风险标记）、automation policy（自动化策略） | 不能把 Entity Log 当 Case Log（案例日志）或 classification memory（分类记忆） |
| Coordinator / Pending Node（协调 / 待确认节点） | 将 alias / role / identity gap（别名 / 角色 / 身份缺口）转成人工问题 | blocking context（阻断上下文）、risk flags（风险标记）、candidate refs（候选引用） | accountant answer（会计师回答）不能由本节点直接写入 Entity Log |
| Review Node（审核节点） | 向 accountant（会计师）展示当前 entity authority（实体权威）和候选风险 | active state（有效状态）、authority refs（权威引用）、candidate context（候选上下文） | Review 本身不批准 durable entity mutation（长期实体变更） |
| Governance Review Node（治理审核节点） | 处理 alias / role / merge / split / policy mutation（别名 / 角色 / 合并 / 拆分 / 策略变更） | current state（当前状态）、old value（旧值）、candidate context（候选上下文）、evidence refs（证据引用） | 必须写 Governance Log（治理日志）或等价 audit trace（审计追溯） |
| Post-Batch Lint Node（批后体检节点） | 检查 entity risk（实体风险）、merge / split candidate（合并 / 拆分候选）、automation risk（自动化风险） | entity clusters（实体簇）、risk flags（风险标记）、policy state（策略状态）、case-derived hints（案例衍生提示） | 只能提出候选；自动放宽或升级 policy（策略）禁止 |
| Knowledge Compilation Node（知识编译节点） | 生成 readable customer knowledge（可读客户知识） | entity identity（实体身份）、aliases（别名）、roles（角色）、risk and policy summary（风险和策略摘要） | summary（摘要）不能替代 Entity Log authority（实体日志权威） |

## 3. 写入者

| Writer | 可以写什么 | 写入类型 | 需要 approval 吗 |
| --- | --- | --- | --- |
| Onboarding Node（初始化节点） | sufficiently supported initial entity foundation（证据充分的初始实体基础）、已有 accountant-derived role（会计历史推导角色）的受控写入 | direct / candidate，具体边界未完全冻结 | stable role（稳定角色）需要 accountant-derived source metadata（会计来源元数据）；其他候选需后续 approval（批准） |
| Governance Review Node（治理审核节点） | alias approval / rejection（别名批准 / 拒绝）、role confirmation（角色确认）、entity lifecycle mutation（实体生命周期变更）、merge / split projection（合并 / 拆分投影）、automation policy mutation（自动化策略变更） | governance mutation（治理变更） | 是，除系统受控自动降级外 |
| Post-Batch Lint Node（批后体检节点） | automation policy downgrade candidate（自动化策略降级候选）或受控 auto-applied downgrade（自动生效降级） | candidate / restrictive mutation（候选 / 收紧变更） | 放宽或升级必须 approval；自动降级必须 governance visibility（治理可见） |
| Review Node（审核节点） | entity / alias / role / policy candidate（实体 / 别名 / 角色 / 策略候选） | candidate（候选） | 是；Review 不直接写 stable authority（稳定权威） |
| Case Memory Update Node（案例记忆更新节点） | entity risk / alias / role / policy candidate（实体风险 / 别名 / 角色 / 策略候选） | candidate（候选） | 是；不能直接写 Entity Log |
| Entity Resolution Node（实体识别节点） | identity issue / candidate signal（身份问题 / 候选信号） | candidate only（仅候选） | 是；不能直接写 Entity Log |

## 4. Candidate 边界

以下内容可以作为 candidate（候选）：

- `new_entity_candidate`（新实体候选）。
- `alias_candidate`（别名候选）。
- `role_confirmation_candidate`（角色确认候选）。
- `merge_split_candidate`（合并 / 拆分候选）。
- `automation_policy_candidate`（自动化策略候选）。
- `entity_risk_candidate`（实体风险候选）。

Candidate（候选）不能直接成为：

- stable entity（稳定实体）。
- `approved_alias`（已批准别名）。
- `rejected_alias`（已拒绝别名）。
- `confirmed_role`（已确认角色）。
- active rule authority（生效规则权威）。
- relaxed automation policy（放宽后的自动化策略）。

Candidate（候选）进入 durable state（长期状态）的条件：

- 必须有 traceable evidence refs（可追溯证据引用）或 completed case refs（已完成案例引用）。
- 必须明确 mutation target（变更目标），例如 alias（别名）、role（角色）、entity lifecycle（实体生命周期）或 automation policy（自动化策略）。
- 必须经过 accountant / governance approval（会计师 / 治理批准），除非是已批准边界内的 restrictive auto-downgrade（收紧型自动降级）。
- 必须留下 Governance Log（治理日志）或等价 audit trace（审计追溯）。

## 5. Lifecycle / States

只写已经必要且稳定的 state（状态）。

| State | 含义 | 谁可以进入 | 谁可以退出 | 下游含义 |
| --- | --- | --- | --- | --- |
| `candidate`（候选） | 可能是新实体，但尚未成为 stable authority（稳定权威） | Onboarding / Review / Governance intake，具体持久化边界未冻结 | Governance Review（治理审核）或清理流程，未冻结 | 不能支持 rule match（规则匹配），只能作 case / review context（案例 / 审核上下文） |
| `active`（有效） | 已确认可作为 stable identity target（稳定身份目标） | Governance approval（治理批准）或受控 onboarding authority（初始化权威） | Governance Review（治理审核） | 可被 Entity Resolution（实体识别）作为稳定目标；不等于自动分类许可 |
| `merged`（已合并） | 该 entity（实体）已并入另一个 entity（实体） | Governance Review（治理审核） | 通常不退出，除非治理回滚，未冻结 | 不能直接作为 active target（有效目标）；应指向 surviving entity（保留实体） |
| `archived`（已归档） | entity（实体）不再作为常规 active target（有效目标） | Governance Review（治理审核） | Governance Review（治理审核） | 不支持 rule match（规则匹配）；可作历史解释 |

## 6. Mutation Path

已冻结 mutation path（变更路径）：

```text
candidate / review / lint / onboarding signal
-> evidence and authority review
-> accountant / governance approval, unless restrictive auto-downgrade is explicitly allowed
-> Entity Log mutation
-> Governance Log or equivalent audit trace
-> downstream reads updated authority
```

未冻结 mutation path：

- candidate entity（候选实体）是否直接持久化到 Entity Log，还是先进入 governance queue（治理队列）。
- alias（别名）和 role（角色）是否作为 nested records（嵌套记录）或 independent records（独立记录）写入。
- automation policy auto-downgrade（自动化策略自动降级）的 exact approval / visibility contract（精确批准 / 可见性契约）。
- merge / split（合并 / 拆分）后 approved aliases（已批准别名）、confirmed roles（已确认角色）、rules（规则）和 cases（案例）的迁移规则。

例外：

- Onboarding（初始化）可以在受控边界内写入 accountant-derived role（会计历史推导角色），但必须来自 accountant-processed historical books（会计师已处理历史账本）并保留 authority metadata（权威元数据）。
- 系统可以执行 restrictive auto-downgrade（收紧型自动降级），例如降低 automation policy（自动化策略），但必须保留治理可见性；任何 upgrade / relaxation（升级 / 放宽）都不能自动执行。

## 7. 与其他 memory/log 的边界

| Other Store | 已确认边界 | 未冻结边界 |
| --- | --- | --- |
| Evidence Log（证据日志） | 保存 raw evidence（原始证据）和 evidence refs（证据引用）；Entity Log 只保存链接，不复制原文 | evidence sufficiency threshold（证据充分门槛）尚未冻结 |
| Transaction Log（交易日志） | 保存 final transaction audit record（最终交易审计记录）；不参与 runtime identity decision（运行时身份决策） | reprocessing / correction（重处理 / 纠正）如何影响 entity risk candidate（实体风险候选）尚未冻结 |
| Case Log（案例日志） | 保存 entity-indexed completed-case learning memory（按实体索引的已完成案例学习记忆）；可为 risk / policy candidate（风险 / 策略候选）提供依据 | case-derived risk（案例衍生风险）进入 Entity Log 的 exact governance path（精确治理路径）尚未冻结 |
| Rule Log（规则日志） | 保存 approved deterministic rules（已批准确定性规则）；Entity Log 不保存 rule condition（规则条件）或 accounting treatment（会计处理） | rule/entity merge impact（规则与实体合并影响）尚未冻结 |
| Governance Log（治理日志） | 保存高权限变化、批准、拒绝、降级、合并和拆分历史 | Entity Log 是否存 projected state（投影状态）还是每次查询 Governance Log，尚未字段级冻结 |
| Knowledge Summary（知识摘要） | 可读摘要，不是 source authority（来源权威） | summary conflict repair（摘要冲突修复）尚未冻结 |
| Profile（客户结构档案） | 保存客户结构事实，例如 bank accounts（银行账户）、loans（贷款）、employees（员工） | person entity（个人实体）与 employee profile fact（员工结构事实）的边界尚未冻结 |

## 8. 冲突处理

如果 Entity Log（实体日志）与 Governance Log（治理日志）冲突：

- authority 顺序：Governance Log（治理日志）中的 approved / applied event（已批准 / 已生效事件）优先。
- runtime 行为：下游应保守使用 governance-applied authority（已生效治理权威）。
- 是否阻断自动化：如果冲突影响 alias（别名）、role（角色）、entity status（实体状态）或 automation policy（自动化策略），应阻断相关自动化路径。
- 是否生成 review / governance candidate：是。

如果 Entity Log（实体日志）与 Knowledge Summary（知识摘要）冲突：

- authority 顺序：Entity Log（实体日志）优先。
- runtime 行为：summary（摘要）只能作为可读背景，不得覆盖 authority（权威）。
- 是否阻断自动化：如果 summary 暗示风险但未进入 Entity Log / Governance Log，不能直接改变 authority；可生成 review candidate（审核候选）。
- 是否生成 review / governance candidate：可以。

如果 Case Log（案例日志）显示风险，但 Entity Log（实体日志）尚未更新：

- authority 顺序：Entity Log（实体日志）当前 authority（当前权威）仍有效；Case Log（案例日志）只提供 risk evidence（风险依据）。
- runtime 行为：由 Post-Batch Lint / Review / Governance Review（批后体检 / 审核 / 治理审核）生成 candidate（候选）。
- 是否阻断自动化：除非已有 policy / governance constraint（策略 / 治理限制），否则 Case Log 本身不直接阻断。
- 是否生成 review / governance candidate：是。

如果 current runtime signal（当前运行时信号）与 Entity Log（实体日志）冲突：

- authority 顺序：Entity Log（实体日志）优先于 runtime signal（运行时信号），除非 accountant（会计师）明确纠正并进入 review / governance path（审核 / 治理路径）。
- runtime 行为：输出 ambiguity / conflict（歧义 / 冲突）语义，不静默覆盖。
- 是否阻断自动化：如果影响 identity safety（身份安全），阻断相关自动化路径。
- 是否生成 review / governance candidate：是。

## 9. Audit / Trace 边界

这层 memory 必须留下的 trace：

- `authority_source`（权威来源）。
- `evidence_refs`（证据引用）。
- `governance_refs`（治理引用）。
- `created_from`（创建来源）。
- `updated_by`（更新者 / 更新来源），字段名未冻结。
- `superseded_by`（被替代引用），字段名未冻结。
- merge / split refs（合并 / 拆分引用）。
- policy change refs（策略变化引用）。

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

1. `entity_record`（实体记录）的 exact field schema（精确字段结构）。
2. `alias_record`（别名记录）的存储形态。
3. `role_record`（角色记录）的存储形态。
4. `candidate_entity`（候选实体）的持久化位置。
5. `automation_policy`（自动化策略）与 Governance Log（治理日志）的 projection boundary（投影边界）。
6. merge / split（合并 / 拆分）对 aliases（别名）、roles（角色）、rules（规则）和 cases（案例）的迁移或阻断规则。
7. person entity（个人实体）与 Profile employee / owner facts（客户结构中的员工 / owner 事实）的边界。

这些问题解决前，不能进入：

- [x] M3 data contract
- [x] M4 storage and operations
- [x] implementation
