# Entity Log - Authority, Lifecycle, and Boundaries

## 1. Source of Truth / Authority

只填写当前已经稳定的 authority（权威），并标明已确认的非权威候选边界。未稳定的字段形态进入 Open Boundaries（未冻结边界）。

| Concept | Source of Truth | Authority Rule | Not Authority |
| --- | --- | --- | --- |
| entity identity（实体身份） | `Entity Log`（实体日志）作为 stable entity identity 主档案，并通过 traceable evidence refs（可追溯证据引用）和 governance refs（治理引用）支撑 | active entity（有效实体）可作为 runtime identity target（运行时身份目标）；merged / archived 不能被静默当作 active target | display text（展示文本）、Knowledge Summary（知识摘要）、LLM rationale（模型理由）、Transaction Log outcome（交易结果） |
| entity-centered Alias（以 entity 为中心的别名集合） | `Entity Log`（实体日志） | 某个 stable entity 名下有哪些已确认 Alias，是该 entity 身份权威状态的一部分 | 所有历史 description、未确认 surface text、normalized display text（标准化展示文本）、Alias Log 作为第二套身份权威、LLM rationale（模型理由） |
| alias lookup projection（别名反查投影） | `Alias Log`（别名日志）作为由已确认 Alias 建立的 alias -> entity projection / index | 服务 Entity Resolution 从当前 surface text 查询 stable entity；查询方向不同，不改变 Alias entity-centered 语义归属 | Rule Match 横向信息源、独立身份主档案、未确认 alias authority |
| entity_status / supersession（生命周期状态 / 重定向） | `Entity Log`（实体日志）加 approved governance refs（已批准治理引用） | 限制 entity 是否可作为 active identity target；merge / split 后提供当前 surviving entity / supersession 指向 | transaction outcome（交易结果）、case pattern（案例模式）、automation permission（自动化许可）、merge / split 事件历史 |
| automation_policy（自动化策略） | `Entity Log`（实体日志）保存当前 entity 级控制决定 / 状态；Case Log 保存支撑证据；Governance Log 保存审批事件历史 | 限制 entity-level automation（实体级自动化）；upgrade / relaxation 必须 accountant approval，系统自动降级只限受控收紧并保留治理可见性 | active status（有效状态）、case frequency（案例频率）、rule payload（规则内容）、审批事件历史 |
| identity risk flags（身份级风险标记） | `Entity Log`（实体日志）保存当前会作为 runtime 卡点的身份风险 | 只保留疑似同名、疑似应 merge、identity conflict 等身份风险；可阻断或降级依赖身份安全的自动化路径 | 分类不稳定、记账结果漂移、case-derived automation risk、直接分类结论、rule authority（规则权威） |
| trace refs（追溯指针） | Evidence Log / Governance Log / creation trace 等对应来源 | Entity Log 只保存 refs，支撑 review / correction / governance / audit 回查“凭什么” | 原始证据正文、推理正文、accountant approval 本身、governance approval 本身 |

## 2. 读取者

| Reader | 读取目的 | 可读内容 | 限制 |
| --- | --- | --- | --- |
| Entity Resolution Node（实体识别节点） | 判断当前 evidence（证据）是否指向 stable entity（稳定实体） | entity identity（实体身份）、Alias 库、status（状态）、risk flags（风险标记）、automation policy（自动化策略） | 只有 `new_stable_entity` entity 本体可同步写入；Alias / automation policy / rule 不随 entity 本体同步写入 |
| Rule Match Node（规则匹配节点，经 Entity Resolution handoff 投影） | 消费上游已确认 stable entity 的放行 / 卡点上下文，用于 rule route 和 eligibility 判断 | 不直接读取 Entity Log；由 Entity Resolution 读取 Entity Log 后，将该 entity 的 automation_policy / eligibility 当前状态和身份基础投影进 handoff | Entity Log 仍是 automation_policy / identity control 的 source of truth，但不是 Rule Match 的直接读取对象；Rule Match 必须自行读取 Rule Log，Entity Log 不提供 rule condition；Alias lookup 只发生在上游 Entity Resolution |
| Case Judgment Node（案例判断节点） | 使用 entity context（实体上下文）和 automation boundary（自动化边界） | identity basis（身份基础）、risk flags（风险标记）、automation policy（自动化策略） | 不能把 Entity Log 当 Case Log（案例日志）或 classification memory（分类记忆） |
| Coordinator / Pending 路径（协调 / Pending 路径） | 将 identity gap（身份缺口）转成人工问题所需的 identity 背景 | blocking context（阻断上下文）、candidate refs（候选引用）——均由 Case Judgment 在构造 Pending handoff 时读取并投影进 Pending | Coordinator 本身不直接读取 Entity Log，全部 identity 上下文随 CJ Pending 在手；identity risk flags（风险标记）不投影给 Coordinator（服务 Review / Governance / Post-Batch Lint），accountant answer（会计师回答）不能由 Coordinator 直接写入 Entity Log |
| Review Node（审核节点） | 向 accountant（会计师）展示当前 entity authority（实体权威）和候选风险 | active state（有效状态）、authority refs（权威引用）、candidate context（候选上下文） | Review 本身不批准 durable entity mutation（长期实体变更） |
| Governance Review Node（治理审核节点） | 处理 Alias / merge / split / policy mutation（别名 / 合并 / 拆分 / 策略变更） | current state（当前状态）、old value（旧值）、candidate context（候选上下文）、evidence refs（证据引用） | 必须写 Governance Log（治理日志）或等价 audit trace（审计追溯） |
| Post-Batch Lint Node（批后体检节点） | 检查身份风险、merge / split candidate（合并 / 拆分候选）、automation risk（自动化风险） | entity clusters（实体簇）、身份级 risk flags、policy state（策略状态）、case-derived hints（案例衍生提示） | 只能提出候选；case-derived hints 不直接成为 Entity Log risk_flags；自动放宽或升级 policy（策略）禁止 |
| Knowledge Compilation Node（知识编译节点） | 生成 readable customer knowledge（可读客户知识） | entity identity（实体身份）、aliases（别名）、risk and policy summary（风险和策略摘要） | summary（摘要）不能替代 Entity Log authority（实体日志权威） |

## 3. 写入者

| Writer | 可以写什么 | 写入类型 | 需要 approval 吗 |
| --- | --- | --- | --- |
| Governance Review Node（治理审核节点） | entity lifecycle mutation（实体生命周期变更）、merge / split projection（合并 / 拆分投影）、automation policy mutation（自动化策略变更） | governance mutation（治理变更） | 是，除系统受控自动降级外 |
| Post-Batch Lint Node（批后体检节点） | automation policy downgrade candidate（自动化策略降级候选）或受控 auto-applied downgrade（自动生效降级） | candidate / restrictive mutation（候选 / 收紧变更） | 放宽或升级必须 approval；自动降级必须 governance visibility（治理可见） |
| Review Node（审核节点） | merge / split、identity risk、policy mutation candidate（合并 / 拆分、身份风险、策略变更候选） | candidate（候选） | 是；Review 不直接写 stable authority（稳定权威） |
| Case Memory Update Node（案例记忆更新节点） | entity risk / policy candidate（实体风险 / 策略候选） | candidate（候选） | 是；不能直接写 Entity Log |
| Entity Resolution Node（实体识别节点） | `new_stable_entity` entity 本体和最小创建 provenance（具体字段名留 M3），以及 identity conflict / risk signal（身份冲突 / 风险信号） | direct stable entity body plus creation trace / runtime risk signal（直接稳定实体本体加创建追溯 / 运行时风险信号） | `new_stable_entity` 本体和创建追溯写入不需要 governance approval；Alias / Rule / automation_policy / Case Log / final transaction outcome 不随 entity 本体写入 |
| Accountant explicit identity confirmation path（会计师明确身份确认路径，执行者留 L4 / seam） | stable entity 本体和最小创建 provenance（具体字段名留 M3） | direct stable entity creation via finalization / memory write mechanism（实际机制未冻结） | 不需要 governance approval；只确认 identity，不隐含分类结果、Alias、Rule、automation_policy 或 Case Log 写入 |

## 4. Candidate 边界

以下内容可以作为 candidate（候选）：

- `merge_split_candidate`（合并 / 拆分候选）。
- `automation_policy_candidate`（自动化策略候选）。
- `entity_risk_candidate`（实体风险候选），仅限身份级风险候选；分类不稳定等 case-derived risk 归 Case Log / governance candidate。

Candidate signal（候选信号）不能直接成为：

- stable entity identity authority（稳定实体身份权威）。
- Alias。
- active rule authority（生效规则权威）。
- relaxed automation policy（放宽后的自动化策略）。

Candidate signal（候选信号）进入 durable non-authority context 或 governance path（长期非权威上下文或治理路径）的条件：

- 必须有 traceable evidence refs（可追溯证据引用）或 completed case refs（已完成案例引用）。
- 必须明确 mutation target（变更目标），例如 entity lifecycle（实体生命周期）或 automation policy（自动化策略）。
- 必须经过 accountant / governance approval（会计师 / 治理批准），除非是已批准边界内的 restrictive auto-downgrade（收紧型自动降级）。
- 必须留下 Governance Log（治理日志）或等价 audit trace（审计追溯）。

Alias（别名）的当前定义已确认：

- Alias 是过去已经确认过的 transaction surface text 和 stable entity 的对应关系。
- 当前只确认两类信息可能成为 Alias：bank statement 中每笔交易的 description / descriptor / raw bank surface text；以及当 bank description 本身没有明确身份意义时，其他可能重复出现并能指向交易主体的字段，例如 cheque payee。
- 某个 stable entity 名下有哪些已确认 Alias，source of truth 在 Entity Log；Alias Log 是面向 Entity Resolution 的 alias -> entity 反查 projection / index。
- Entity Log 与 Alias Log 在 merge / split 治理时需要同步更新；同步执行机制留 L4 / seam。

## 5. Lifecycle / States

只写已经必要且稳定的 state（状态）。

| State | 含义 | 谁可以进入 | 谁可以退出 | 下游含义 |
| --- | --- | --- | --- | --- |
| `active`（有效） | 该 stable entity 当前可作为 runtime stable identity target（运行时稳定身份目标） | Entity Resolution `new_stable_entity` 及时 publication、accountant explicit identity confirmation、Governance approval（治理批准） | Governance Review（治理审核） | 可被 Entity Resolution（实体识别）作为稳定目标；不等于自动分类许可 |
| `merged`（已合并） | 该 entity 已并入 surviving entity，不能再直接作为 active target | Governance Review（治理审核） | 通常不退出；治理回滚规则留 L4 | runtime 应按当前 supersession 指向跳转到 surviving entity |
| `archived`（已归档） | 该 entity 不再作为常规 active target，但保留历史解释和审计引用 | Governance Review（治理审核） | Governance Review（治理审核）；恢复条件留 L3 / L4 | 不得被静默当作 active identity target；可作历史解释 |

Automation policy 不属于 lifecycle state。它是独立的 entity-level control state，用来控制自动化卡点；lifecycle state 只回答 identity target 是否仍有效。

## 6. Mutation Path

已冻结 mutation path（变更路径）包括两类 stable entity 创建入口，以及已批准候选变更的通用路径：

```text
Entity Resolution judges new_stable_entity
-> timely Entity Log publication before downstream depends on the new stable identity
-> mutation limited to stable entity body and minimal creation provenance
-> downstream can read stable identity
-> no Alias / Rule / automation_policy / Case Log / final transaction outcome write is implied
```

```text
accountant explicit identity confirmation in downstream pending path
-> stable entity creation without governance approval
-> mutation limited to stable entity body and minimal creation provenance
-> no Alias / Rule / automation_policy / Case Log / final transaction outcome write is implied
```

```text
candidate / review / lint signal
-> evidence and authority review
-> accountant / governance approval, unless restrictive auto-downgrade is explicitly allowed
-> Entity Log mutation
-> Governance Log or equivalent audit trace
-> downstream reads updated authority
```

未冻结 mutation path：

- Entity Resolution `new_stable_entity` 后的实际写入执行者、调用方式、写入顺序和多 log finalization 机制；但及时可见和无需 governance approval 已冻结。
- accountant explicit identity confirmation 后由哪个 finalization / memory write mechanism（完成 / 记忆写入机制）执行 stable entity 创建；但无需 governance approval 已冻结。
- Alias（别名）在 Entity Log 与 Alias Log 之间的存储形态、projection / index 构建、同步更新和写入顺序尚未冻结。
- automation policy auto-downgrade（自动化策略自动降级）的 exact approval / visibility contract（精确批准 / 可见性契约）。
- merge / split（合并 / 拆分）后 aliases（别名）、rules（规则）和 cases（案例）的跨 log 迁移、阻断、finalization、audit trace 和回滚机制。

约束：

- Accountant 只确认会计分类、没有明确确认 identity 时，不创建 stable entity，也不写 Entity Log。
- 两类 stable entity 创建入口都只确认 entity 本体和最小创建 provenance；本层不重新判断“是否够格”，只记录已由发起路径确认的 authority。

例外：

- 系统可以执行 restrictive auto-downgrade（收紧型自动降级），例如降低 automation policy（自动化策略），但必须保留治理可见性；任何 upgrade / relaxation（升级 / 放宽）都不能自动执行。

## 7. 与其他 memory/log 的边界

| Other Store | 已确认边界 | 未冻结边界 |
| --- | --- | --- |
| Evidence Log（证据日志） | 保存 raw evidence（原始证据）和 evidence refs（证据引用）；Entity Log 只保存链接，不复制原文 | evidence sufficiency threshold（证据充分门槛）尚未冻结 |
| Transaction Log（交易日志） | 保存 final transaction audit record（最终交易审计记录）；不参与 runtime identity decision（运行时身份决策） | reprocessing / correction（重处理 / 纠正）如何影响 entity risk candidate（实体风险候选）尚未冻结 |
| Alias Log（别名日志） | Alias 的 entity-centered source of truth 在 Entity Log；Alias Log 是面向 Entity Resolution 的 alias -> entity 反查 projection / index。治理 merge / split 时 Entity Log 与 Alias Log 同步更新 | 物理存储关系、projection / index 构建方式、同步执行机制、写入顺序和多 log finalization 尚未冻结 |
| Case Log（案例日志） | 保存 entity-indexed completed-case learning memory（按实体索引的已完成案例学习记忆）；分类历史、常用 COA、repeated outcome pattern 和分类不稳定等 case-derived risk 归 Case Log / governance candidate；Entity Log 只保存当前 automation_policy 与身份级 risk flags | case-derived automation / policy candidate 的 exact governance contract 尚未冻结；merge / split 后 case 归属、限制或 supersession 的执行机制留 L4 / seam |
| Rule Log（规则日志） | 保存 approved deterministic rules（已批准确定性规则）；Entity Log 不保存 rule condition、rule payload、approved accounting treatment 或 rule_id 成员列表 | Rule eligibility、阻断或迁移在 merge / split 后如何 finalization，归 Rule Match / Rule Log / Governance，机制尚未冻结 |
| Governance Log（治理日志） | 保存高权限变化、批准、拒绝、降级、合并和拆分历史；Entity Log 只保存当前 lifecycle / supersession / control state 和 governance refs | Entity Log 是否直接存 projected state（投影状态）还是由 Governance Log 投影生成，属于存储 / projection 形态，尚未冻结 |
| Knowledge Summary（知识摘要） | 可读摘要，不是 source authority（来源权威）；entity 级自由经验批注归 entity_level Knowledge Summary，不进 Entity Log | summary conflict repair（摘要冲突修复）尚未冻结 |
| Profile（客户结构档案） | Profile 与 Entity Log 是不同处理逻辑。Profile 负责自身结构事实和直接处理路径；Entity Log 只保存进入 ER / accountant confirmation 路径后形成的 stable entity 主档案。person 类型 stable entity 不特殊排除，也不因 Profile 存在而自动挂起 | Profile 内部结构事实建模、Profile 与 Entity Log 的 ref / projection 形态、Profile 事实是否回显到 Entity 视图留给 Profile 或联合 L3 / L4 |

Entity merge / split 后，Entity Log 只保存 Entity 侧当前身份状态、surviving entity / supersession 指向和 governance refs。Alias 归属重判归 Alias Log / Governance；Case 归属、限制或 supersession 归 Case Log / Governance；Rule eligibility、阻断或迁移归 Rule Match / Rule Log / Governance。Entity Log 不执行跨 log 迁移，也不保存完整治理事件历史。

## 8. 冲突处理

如果 Entity Log（实体日志）与 Governance Log（治理日志）冲突：

- authority 顺序：Governance Log（治理日志）中的 approved / applied event（已批准 / 已生效事件）优先。
- runtime 行为：下游应保守使用 governance-applied authority（已生效治理权威）。
- 是否阻断自动化：如果冲突影响 Alias（别名）、entity status（实体状态）或 automation policy（自动化策略），应阻断相关自动化路径。
- 是否生成 review / governance candidate：是。

如果 Entity Log（实体日志）与 Knowledge Summary（知识摘要）冲突：

- authority 顺序：Entity Log（实体日志）优先。
- runtime 行为：summary（摘要）只能作为可读背景，不得覆盖 authority（权威）。
- 是否阻断自动化：如果 summary 暗示风险但未进入 Entity Log / Governance Log，不能直接改变 authority；可生成 review candidate（审核候选）。
- 是否生成 review / governance candidate：可以。

如果 Case Log（案例日志）显示分类不稳定、记账结果漂移或 automation / policy 风险，但 Entity Log（实体日志）尚未更新：

- authority 顺序：Entity Log（实体日志）当前 identity / control authority（身份 / 控制权威）仍有效；Case Log（案例日志）只提供 case-derived evidence（案例衍生依据）。
- runtime 行为：由 Post-Batch Lint / Review / Governance Review（批后体检 / 审核 / 治理审核）生成 candidate（候选）。
- 是否阻断自动化：除非已有 policy / governance constraint（策略 / 治理限制），否则 Case Log 本身不直接阻断；分类不稳定本身不写入 Entity Log risk_flags。
- 是否生成 review / governance candidate：是。

如果 current runtime signal（当前运行时信号）与 Entity Log（实体日志）冲突：

- authority 顺序：Entity Log（实体日志）优先于 runtime signal（运行时信号），除非 accountant（会计师）明确纠正并进入 review / governance path（审核 / 治理路径）。
- runtime 行为：输出 ambiguity / conflict（歧义 / 冲突）语义，不静默覆盖。
- 是否阻断自动化：如果影响 identity safety（身份安全），阻断相关自动化路径。
- 是否生成 review / governance candidate：是。

## 9. Audit / Trace 边界

这层 memory 必须留下的 trace：

- authority / creation source refs（权威 / 创建来源指针）。
- evidence refs（证据引用），只指向证据来源，不复制正文。
- governance refs（治理引用），只指向批准、拒绝、降级、合并、拆分或策略变更事件，不复制事件历史。
- supersession / redirection refs（替代 / 重定向引用）。
- risk / policy rationale refs（风险 / 策略理由引用），用于解释结构化控制状态，不是自由文本 note 字段。
- matched surface text / created time / updated-by / auto-created marker 等 provenance 信息如需字段化，字段名和形态留 M3。

这些 trace 用于：

- review（审核）。
- correction（纠正）。
- governance（治理）。
- audit（审计）。

这些 trace 不能成为：

- accounting classification（会计分类）。
- rule authority（规则权威）。
- case authority（案例权威）。
- stable entity identity 本体。
- Alias authority 本体。
- accountant approval（会计师批准）。
- governance approval（治理批准）。

## 10. Open Boundaries

以下问题未冻结：

1. `entity_record`（实体记录）的 exact field schema（精确字段结构）。
2. 五类保存信息各自的 exact field schema、enum（含 entity_status、automation_policy、risk 标记取值）、validation 和 refs 形态。
3. `new_stable_entity` 与 accountant explicit identity confirmation 的最小创建 provenance 字段形态。
4. `Alias Log` 的技术形态，以及 Entity Log 与 Alias Log 的物理存储关系、projection / index 构建方式、同步机制和写入顺序。
5. `automation_policy`（自动化策略）与 Governance Log（治理日志）的存储 / projection 形态，以及自动降级 exact approval / visibility contract。
6. ER `new_stable_entity` 与 accountant explicit identity confirmation 后的实际写入执行者、调用方式、写入顺序和多 log finalization；但无需 governance approval 与及时可见语义已冻结。
7. merge / split（合并 / 拆分）的治理审批、批量重判、跨 log finalization、audit trace 和回滚机制。
8. Profile 内部结构事实建模，以及 Profile 与 Entity Log 的 ref / projection 形态；person 类型 stable entity 不特殊排除这一点已冻结。

这些问题解决前，不能进入：

- [x] M3 data contract
- [x] M4 storage and operations
- [x] implementation
