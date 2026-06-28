# Entity Log - Memory Intent

## 1. 为什么这层 memory 必须存在

它解决的问题是：

> 为客户账务语境中的 stable entity（稳定实体）保存 durable、可审计、以 entity 为主体的 identity authority 主档案，让 runtime nodes（运行时节点）能安全判断“这是谁”，而不用把身份、分类、规则、控制状态和审计记录混在一起。

如果删除它，会失去：

- stable identity authority（稳定身份权威）：系统无法把“这个主体是谁”作为跨时间可复用、可审计的 source of truth。
- identity reuse surface（身份复用面）：系统无法复用过去已经确认过的 transaction surface text -> stable entity 对应关系。
- lifecycle / supersession control（生命周期 / 重定向控制）：系统无法确定某个 stable entity 当前是否还能作为 active identity target，或 merge / split 后应跳转到谁。
- entity-level control state（实体级控制状态）：系统无法在 runtime 就地、确定性读取 force_pending / promotion_lock。

这些能力不能由 runtime handoff（运行时交接）或现有 store（现有存储）覆盖，因为：

- runtime handoff（运行时交接）只表达当前交易上下文，不能作为未来交易可复用权威。
- `Transaction Log`（交易日志）是 audit-facing final record（面向审计的最终记录），不参与 runtime decision（运行时决策）。
- `Case Log`（案例日志）保存 entity-indexed learning memory（按实体索引的学习记忆），不保存 identity authority（身份权威）。
- `Rule Log`（规则日志）保存 approved deterministic rules（已批准确定性规则），不回答 entity identity（实体身份）。
- `Knowledge Summary`（知识摘要）是 readable context（可读上下文），不能替代 source authority（来源权威）。

## 2. 保存什么

判别准则：

> 下游是否有 runtime 判断或控制动作，必须就地、确定性地读到该状态。是则进入 Entity Log；只是事后想知道“为什么”，则只存 trace refs 或归其他 log。

Entity Log 保存的信息只分五类。以下括号内是类型示例，不冻结字段名、enum 或 refs 形态：

- 身份脊梁（identity backbone）：稳定 entity 主键、展示名、entity type 等用于铸造并担保跨时间身份引用的信息。全系统 entity-indexed 记忆最终都指向这条身份脊梁。
- 身份状态与重定向（status & supersession）：active / merged / archived 的生命周期语义，以及 merge / split 后当前 supersession / redirection 指向。它只表示是否能作为 active identity target，不代表自动分类许可；事件历史归 Governance Log。
- 身份复用面（identity reuse surface）：该 stable entity 名下已确认 Alias 集合。Alias 是已确认 transaction surface text -> stable entity 身份复用关系；在 Entity Log 中表现为 entity-centered 身份属性。Alias Log 是面向 Entity Resolution 的 alias -> entity 反查 projection / index。
- 控制状态（entity-level control state）：entity 级 force_pending 与 promotion_lock 两个互相独立、都只能会计师人发起的治理控制。会计师表达“以后遇到这个 entity 要当心 / 系统老判错”时，不新增自由文本 note 字段；应落成 force_pending / promotion_lock 和对应 rationale ref。
- 追溯指针（trace refs）：authority / 创建来源、evidence refs、governance refs、supersession refs、risk / policy rationale refs 等只存引用不存正文的指针。它们支撑 review、correction、governance 和 audit，但本身不构成业务 authority。

Entity Log 对 person、vendor、organization、payee 等 entity_type 使用同一套保存标准：凡是经 Entity Resolution 或 accountant explicit identity confirmation 成为 stable entity 的主体，都可以进入 Entity Log。Profile / routing 直接处理且未进入 Entity Resolution 的交易，不因此创建 Entity Log 记录。

其中可以成为 reusable authority（可复用权威）的是：

- stable entity identity（稳定实体身份）与其身份脊梁。
- 该 stable entity 名下已确认 Alias 集合。
- entity lifecycle / supersession 当前状态。
- force_pending / promotion_lock 当前控制状态。

只作为 trace / context / explanation（追溯 / 上下文 / 解释）的是：

- evidence refs、creation provenance refs、governance refs。
- risk / policy rationale refs。
- rejected / superseded context refs。
- historical display text refs。

Alias 当前定义：

- Alias 是过去已经确认过的 transaction surface text 和 stable entity 的对应关系。
- 当前只确认两类信息可能成为 Alias：bank statement 中每笔交易的 description / descriptor / raw bank surface text；以及当 bank description 本身没有明确身份意义时，其他可能重复出现并能指向交易主体的字段，例如 cheque payee。
- Alias 不是所有历史 description 的集合。只有已经和某个明确 stable entity 建立过确认关系的 surface text，才属于 Alias。
- Alias 的 entity-centered source of truth 归 Entity Log；Alias Log 只是面向 Entity Resolution 的反向查询 projection / index。Entity Log 与 Alias Log 在治理 merge / split 时需要同步更新，具体同步机制留 L4 / seam。

## 3. 绝不保存什么

这层 memory 绝不保存以下“围绕 entity 但不属身份 / 控制状态”的信息，按归属分流：

- COA account、HST / GST treatment、journal entry、final transaction outcome、transaction processing path：归 Transaction Log 或对应运行节点输出；Entity Log 不保存交易处理结果。
- classification history、“这个 entity 通常记成哪个 COA”、repeated outcome pattern：归 Case Log；分类不稳定、记账结果漂移等 case-derived risk 不写入 Entity Log。
- entity-level rule、pattern condition、active rule payload、approved accounting treatment：归 Rule Log；Entity Log 至多保存非规则体的指针 / 标志，且字段形态留后续阶段。
- 逐笔交易历史与完整 audit trail：归 Transaction Log；Entity Log 不得变成按 entity 索引的流水账。
- 身份判断背后的推理、证据正文、raw evidence blob、未经批准的 AI reasoning：归 Evidence Log；Entity Log 只存 ref。
- merge / split / 审批 / policy change 的事件历史：归 Governance Log；Entity Log 只存当前 lifecycle / supersession / control state 和治理 refs。
- entity 级自由经验批注，例如“关于这个 entity 的一般会计经验，下次判断参考”：不进 Entity Log，归 entity_level Knowledge Summary。Entity Log 不新增自由文本 note 字段。
- runtime-only `entity_resolution_output`（实体识别运行时输出）：只属于运行时 handoff，不作为 Entity Log durable record。

它也不能替代：

- `Evidence Log`（证据日志）：原始 evidence（证据）和 evidence refs（证据引用）的 source of truth（事实来源）。
- `Transaction Log`（交易日志）：每笔交易最终结果和 audit trail（审计轨迹）的 source of truth（事实来源）。
- `Case Log`（案例日志）：entity-indexed completed-case learning memory（按实体索引的已完成案例学习记忆）。
- `Rule Log`（规则日志）：approved deterministic rules（已批准确定性规则）的 source of truth（事实来源）。
- `Governance Log`（治理日志）：高权限长期变化和审批历史的 source of truth（事实来源）。
- `Knowledge Summary`（知识摘要）：entity 级自由经验批注和可读经验总结的归属位置。
- `Profile`（客户结构档案）：客户 bank accounts（银行账户）、loans（贷款）、employees（员工）和 structural relationships（结构关系）的 source of truth（事实来源）。

## 4. 对核心产品目标的贡献

这层 memory 必须清楚支持以下至少一项：

- [x] 记忆复用
- [x] 有证据支持的建议
- [x] accountant correction learning
- [x] 审计性
- [x] accountant control
- [x] 自动化率提升

具体贡献：

- 让过去已经确认过的 transaction surface text 可以作为该 stable entity 的身份复用面，并通过 Alias Log projection 支持 Entity Resolution 反查。
- 让下游通过稳定 entity identity、lifecycle / supersession 避免误用已合并、归档或身份冲突的 entity。
- 让 force_pending / promotion_lock 把 entity-level control state 转成明确、可审计、会计师可控制的自动化边界。
- 让 evidence refs、governance refs 和 rationale refs 保留 authority change 的来源，支持审计和纠错，但不复制正文或推理。

## 5. 已知约束

- `active`（有效）只表示 entity（实体）可作为稳定身份目标，不等于可以自动分类。
- Stable entity 的 L2 创建入口只确认两类：`Entity Resolution Node` 判断 `new_stable_entity` 后发起及时 Entity Log + Alias Log finalization；accountant explicit identity confirmation 后触发与 ER 共用的 Entity Log + Alias Log 统一写入路径。
- 两类创建入口都不需要 governance approval（治理批准），且确认 entity 本体、最小创建 provenance 和该交易对应的已确认 Alias identity reuse surface；stable entity 创建必须同步写入 Alias Log projection，但不隐含 Rule、force_pending / promotion_lock、Case Log 或 final transaction outcome 写入。
- `new_stable_entity` 的实际写入执行者、调用方式、写入顺序和多 log finalization 属 L4 / seam；但“无需 governance approval”“及时可见”和“Entity Log + Alias Log 同步写入”是 L2 已定事实。
- Alias 只提供 identity reuse（身份复用）能力，不提供 rule authority（规则权威）或会计分类结论。
- Rule Match 不直接读取 Entity Log；Entity Resolution 读取 Entity Log 后，把该 entity 的 eligibility 当前状态投影进 handoff 传给 Rule Match（force_pending 命中已在上游 ER→RM router 改道 Case Judgment / Pending、不进 Rule Match；promotion_lock 由确定性发现 job 消费；两者 Rule Match 都不读）。
- `Case Log`（案例日志）可以提供 case-derived policy / governance evidence（案例衍生的策略 / 治理依据），但不能直接修改 Entity Log（实体日志）。
- `Transaction Log`（交易日志）不能作为 runtime identity source（运行时身份来源）或 learning source（学习来源）。
- force_pending / promotion_lock 的设置与解除都必须由会计师人发起（Human Review → Finalization 落盘），系统不自发设 / 收紧 / 放宽。
- accountant 只确认会计分类、未确认 identity 时，不创建 entity record；该交易以 unknown entity 终态 finalize（具体记录边界见 `BK_Copilot/memory_layers/case_log/` 和 `BK_Copilot/workflow_nodes/coordinator_node/02_logic_and_boundaries.md`，原 `Coordinator Question.md` 已删除、内容已吸收）。
- person 类型 stable entity 不特殊排除；是否进入 Entity Log 取决于是否经 ER / accountant confirmation 成为 stable entity。

## 6. 未决定问题

- `entity_record`（实体记录）和五类保存信息各自的 exact field schema、enum、validation 和 refs 形态尚未冻结。
- `new_stable_entity` 与 accountant explicit identity confirmation 的最小创建 provenance 字段形态尚未冻结。
- Alias 在 Entity Log 与 Alias Log 之间的物理存储、projection / index 构建、同步机制和写入顺序尚未冻结。
- `force_pending` / `promotion_lock` 是直接存储在 Entity Log，还是由 Governance Log 投影生成，属于存储 / projection 形态，尚未冻结。
- Stable entity 创建后 Entity Log + Alias Log 同步 finalization 的实际写入执行者、调用方式、写入顺序和多 log finalization 属 L4 / seam。
- merge / split 后 Alias、Rule、Case 的跨 log 迁移 / 阻断 / finalization / rollback 机制尚未冻结；Entity Log L2 只定 Entity 侧身份状态与重定向。
- Profile 与 Entity Log 的 ref / projection 形态、Profile 结构事实是否回显到 Entity 视图，留给 Profile 或联合 L3 / L4。
