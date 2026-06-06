# Alias L2 提案

## 依赖
- ER 已锁结论：Entity Resolution 只输出 stable / unknown；exact Alias match 可以直接复用 Alias 指向的 stable entity；`new_stable_entity` publication 只确认 entity 本体，不同时确认 Alias / Rule / automation。
- Entity Log 已锁边界：stable entity identity 的 source of truth 是 Entity Log；Alias 只能指向明确 stable entity，不能指向 unknown、runtime guess 或分类结果。
- Rule Match 已锁临时结论：Alias lookup 位于上游 Entity Resolution；Rule Match 消费已确认 stable entity 和 rule scope，不直接以 Alias 建立确定性自动化。

## L2 决策

### Alias 的权威对象与对外支持范围
- 结论:
Alias Log 的 reusable authority 只是一类关系：已确认 transaction surface text -> stable entity。它对外支持的 L2 范围限于 identity reuse：Entity Resolution 可以读取 Alias 来判断当前交易主体；其他下游如果需要 Case Log / Rule Log / Transaction Log，只能先经过 ER 得到 stable entity，再按各自契约读取对应记忆。Alias 不直接支持 Rule Match、Case Judgment 会计分类、JE 生成或 automation permission。
- 为什么(锚定核心产品目标的哪条):
支持记忆复用、审计性、accountant control 和自动化率。Alias 让历史已确认身份关系可复用，但不把表面文本误升级成会计规则或落账权限。
- 拟改:`BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:读取者` → `当前冻结的 reader 只有 Entity Resolution Node。Alias 对外只提供 identity reuse 查询接口：输入当前交易 surface text 或受控等价查询条件，返回已确认 Alias relationship 及其 target stable entity ref。任何下游会计判断、rule match 或 automation path 都不得直接依赖 Alias；它们只能依赖 ER 输出的 stable entity 及各自正式契约。`
- 拟改:`BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:与其他 memory/log 的边界` → `Rule Match 不读取 Alias 作为 rule condition 或 automation basis。Alias lookup 发生在 Entity Resolution；Rule Match 只消费 stable entity、approved rule scope 和客观匹配条件。`
- 排除的替代 + 理由:
让 Rule Match 直接按 Alias 命中自动匹配规则。理由是 Alias 只证明“这是谁”，不证明“这笔交易怎么记”或“是否允许自动化”；直接接入 Rule Match 会把身份复用误变成 rule authority。

### 已确认 identity 后的 Alias 写入资格
- 结论:
当一笔交易已经形成 stable identity，且当前交易存在可追溯、可作为身份表面线索的 Alias-eligible surface text 时，系统应生成 Alias 写入意图 / finalization request。可进入 Alias 的关系是“该 surface text 已被确认指向该 stable entity”，不是“stable entity 已创建”本身。ER 自动确认 stable、accountant 在 pending / clarification 中明确确认 identity，都可以提供 Alias 关系的 authority basis；不需要为同一 surface text -> stable entity 关系再要求一次额外 accountant approval。实际由谁写、怎样写、与多 log finalization 的顺序，属于 L4 / seam-park。
- 为什么(锚定核心产品目标的哪条):
支持记忆复用、纠正学习和自动化率。若身份已经确认却不把可复用 surface text 进入 Alias 写入流程，未来相同交易会反复解析或提问；若把写入绑定到 entity 本体创建，则会越过 ER 已锁的运行 / 记忆 seam。
- 拟改:`BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:写入者` → `已冻结的语义资格：只有已经被确认为指向明确 stable entity 的 Alias-eligible surface text，才可以进入 durable Alias 写入。ER stable reason 或 accountant 明确 identity confirmation 可以作为 authority basis；Alias Log 不重新判断 identity，只校验写入请求是否携带 stable entity authority、surface text 来源和确认依据。exact writer、调用方式、批内顺序和统一 finalization mechanism 尚未冻结。`
- 拟改:`BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:Mutation Path` → `已冻结 L2 path: transaction surface text -> confirmed relationship to a clear stable entity -> Alias write intent / finalization request -> durable Alias authority。该 path 不表示 ER 在创建 new_stable_entity 时内联写 Alias，也不表示 Alias 与 Entity Log 必须同一存储。`
- 拟改:`BK_Copilot/memory_layers/alias_log/01_memory_intent.md:保存什么` → `Alias Log 保存的是已确认 surface text -> stable entity 关系；如果当前交易的 surface text 只是在交易中出现、但没有被确认与 stable entity 建立身份关系，则不能成为 Alias authority。`
- 排除的替代 + 理由:
要求每一条 Alias creation 都额外经过 accountant 单独批准。理由是 ER stable 或 accountant identity confirmation 已经确认了同一交易表面文本与主体的身份关系，重复审批会削弱自动化率且不增加实质审计性。
- 排除的替代 + 理由:
ER 创建 `new_stable_entity` 时顺手直接写 Alias。理由是 ER L2 已锁定 publication 只确认 entity 本体；Alias 写入属于记忆 / finalization seam，不能被运行节点内联成隐式副作用。
- 浮现的新 open boundary:
同一笔交易中多个 Alias-eligible surface text 是否全部生成写入意图，还是按优先级选择，属于 L3 / L4 联合边界。

### 受控 normalization / equivalence 匹配边界
- 结论:
Alias 匹配分三层语义：exact match 可以作为确定性身份复用；受控 normalization / equivalence 只有在规则已冻结、可审计、可复现、且不引入多 entity 竞争时，才可以提升为等同 exact match 的 identity reuse basis；LLM semantic similarity 或“高度类似”本身不是 Alias authority，只能作为 runtime clue / review context。
- 为什么(锚定核心产品目标的哪条):
支持有证据支持的建议、审计性和自动化率。系统需要处理 descriptor 轻微变化，但不能让模型相似度污染 durable identity authority。
- 拟改:`BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:Source of Truth / Authority` → `Alias lookup result 分为 exact match、controlled equivalence match 和 similarity clue。exact match 可直接支持 ER stable；controlled equivalence match 只有在等价规则已冻结且无冲突时可按等价 identity basis 使用；similarity clue 不能成为 Alias authority。具体 normalization / equivalence rule set 和 match output schema 留到 M3。`
- 拟改:`BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:冲突处理` → `未冻结等价规则前，非 exact 的高度类似 surface text 只能作为 runtime clue。若存在多个可能 target stable entity、规则无法解释为什么等价，或 reviewer 无法复现匹配依据，ER 必须保守输出 unknown + alias issue / reason，不得用其包装 stable identity。`
- 排除的替代 + 理由:
把 LLM semantic similarity 当成 Alias match。理由是相似度不可稳定复现，且无法提供足够可审计的 identity authority。
- 浮现的新 open boundary:
具体 normalization transformations、equivalence rule registry、match_basis 字段与阈值属于 L3；索引、检索和缓存机制属于 L4。

### Alias 冲突、多 entity 竞争与 merge / split 后语义
- 结论:
Alias 关系必须服从 Entity Log 的 stable entity lifecycle。若同一 surface text 指向多个 active stable entity、target entity 已被 merge / split / retired 后无法唯一映射，或 Alias 与 Governance / Entity Log authority 冲突，则该 Alias 不得支持 deterministic identity reuse；ER 应输出 unknown + alias issue / reason 或阻断相关自动化路径。Merge 后 Alias 可以在明确 surviving stable entity 且无竞争时迁移 / supersede；split 或多 entity 竞争时默认阻断，直到有明确 identity authority 重新确认归属。具体迁移、supersession、审计记录和批处理顺序属于 L4 / seam-park。
- 为什么(锚定核心产品目标的哪条):
支持审计性、accountant control 和记忆复用的安全性。Alias 的价值来自复用稳定身份；一旦身份归属不唯一，继续复用会污染 Case Log、Rule Match 和后续自动化。
- 拟改:`BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:冲突处理` → `如果同一 Alias surface text 对应多个 active stable entity，或 Alias target 的 Entity Log lifecycle 已不能唯一指向一个 active stable entity，Alias lookup 结果不得作为 stable identity basis。Entity Resolution 必须输出 unknown + alias issue / reason，或要求 review / correction；不得选择一个 winner。`
- 拟改:`BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:Lifecycle / States` → `Alias 自身不新增 candidate state enum。L2 只冻结使用语义：confirmed Alias 可被 ER 复用；conflicted / non-unique / lifecycle-invalid Alias 不得被 ER 用作 deterministic identity basis。exact state names、supersession records 和 mutation schema 留到 M3 / L4。`
- 拟改:`BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:与其他 memory/log 的边界` → `Entity Log lifecycle / merge / split authority 优先于 Alias lookup。Merge 后的 Alias 迁移只在 target stable entity 仍唯一时成立；split 或多归属竞争默认阻断，直到通过外部确认路径重新建立 Alias relationship。`
- 排除的替代 + 理由:
merge / split 后自动把所有 Alias 迁到新 entity。理由是 split 场景下历史 surface text 可能只属于拆分后的一部分主体；自动迁移会扩大错误身份权威。

### Alias mutation 与 Governance / audit 边界
- 结论:
普通 Alias creation 的 authority 来自已确认 identity relationship；Alias Log 本身不要求额外 governance approval。高风险 mutation，包括冲突解决、split 后重新归属、批量迁移、删除 / 停用、以及可能影响 automation safety 的 mutation，必须留下 audit trace，并挂起到 Governance / correction / finalization 层确定审批路径。Alias L2 只冻结“哪些 mutation 不能被普通 creation path 静默完成”，不设计治理流程。
- 为什么(锚定核心产品目标的哪条):
支持审计性、accountant control 和纠正学习。低风险身份复用需要顺畅写入；高风险身份权威变更需要可追溯审批，避免一次错误污染长期记忆。
- 拟改:`BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:Mutation Path` → `普通 Alias creation 可由已确认 identity relationship 支撑；冲突解决、split 后归属变更、批量迁移、删除 / 停用等高风险 mutation 不得走普通 creation path。它们需要 review / correction / governance 或 finalization 层给出明确 approval proof；exact approval path 尚未冻结。`
- 拟改:`BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:Audit / Trace 边界` → `Alias creation、supersession、conflict、reassignment、deactivation 都必须保留可追溯 trace。trace 只用于 review / correction / governance / audit，不能成为 accountant approval 或 governance approval 本身。`
- 浮现的新 open boundary:
Governance Log 是否记录所有 Alias mutation，还是只记录高风险 mutation，依赖 Governance / correction 层，属于 L2·外阻。

## 分类备案
- `alias_record` exact field schema → L3(留联合 L3)
- normalization / equivalence 的具体规则、阈值、match_basis / output schema → L3(留联合 L3)
- 多个 Alias-eligible surface text 的选择、优先级和去重规则 → L3(留联合 L3)
- Alias lifecycle state enum、supersession record schema、conflict record schema → L3(留联合 L3)
- Alias 库技术形态、索引、检索、缓存、与 Entity Log 的存储 / 投影关系 → L4 / seam-park(推给记录层)
- Alias 实际写入者、调用方式、批内顺序、多 log finalization、统一 memory write mechanism → L4 / seam-park(推给记录层)
- merge / split 后 Alias 迁移、阻断、批量修复的执行机制 → L4 / seam-park(推给记录层)
- Alias mutation 的 Governance Log 记录范围和审批路径 → L2·外阻(依赖 Governance / correction / finalization,挂起)
- Review / Knowledge Compilation 是否读取 Alias，以及读取边界 → L2·外阻(依赖对应圈外对象,挂起)

## 自检与判定
- 按模板 L2 清单自检:通过;未过项 = 第21条不能进 M3，因为 alias_record schema、normalization / equivalence 规则、mutation schema 和 Governance approval path 尚未冻结。
- 本对象 L2 可否判定完成:是。Alias 的 L2 策略、authority、生命周期使用语义、对外支持范围和 internal L2 open point 已收口；剩余问题已归入联合 L3、L4 / seam-park 或 L2·外阻。
