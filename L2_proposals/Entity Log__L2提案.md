# Entity Log L2 提案

## 依赖
- ER 已锁结论：Entity identity 对外只有 stable / unknown；`new_stable_entity` 需要及时 publication；ER publication 只确认 entity 本体，不同时确认 Alias、Rule 或 automation。
- Alias 已锁结论：Alias authority 是已确认 transaction surface text -> stable entity 的身份复用关系；Alias lookup 位于 ER，上游确认 stable 后才影响 Case / Rule 等下游。
- Pending 路径已锁结论：accountant 明确确认 identity 后可以创建 stable entity；accountant 只确认分类但未确认 identity 时，不创建 Entity Log 记录。

## L2 决策

### Entity Log 的权威范围收窄为 stable identity + lifecycle
- 结论:
Entity Log 是 stable entity identity 的 source of truth，只持久化 stable entity 记录及其生命周期可用性。`unknown` 是 runtime identity status，不是 Entity Log 记录；任何 candidate entity / candidate identity 都不得进入 Entity Log 或以别名复活。`active` / `merged` / `archived` 等只表示已存在 stable entity record 对下游是否可作为当前 identity target，不是第三种 identity 状态。
- 为什么(锚定核心产品目标的哪条):
支持记忆复用、审计性和 accountant control。下游需要稳定身份锚点来读取 Case Log、Rule Match 和 Alias；但身份不清时持久化“候选实体”会污染长期记忆。
- 拟改:`BK_Copilot/memory_layers/entity_log/00_index.md:一句话定义` → `Entity Log 是 stable entity identity authority store：它保存客户账务语境中已确认 stable entity 是谁，以及该 stable entity 当前是否可作为下游 identity target。它不保存 unknown、candidate identity、classification memory、Alias authority、Rule authority 或 automation permission。`
- 拟改:`BK_Copilot/memory_layers/entity_log/01_memory_intent.md:保存什么` → `这层 memory 保存 stable entity record、entity identity authority、最小创建 provenance、生命周期可用性和必要 authority refs。Alias、automation policy、rule、case precedent 可以通过引用或投影与 entity 相关联，但不是 Entity Log 自己的会计判断权威。`
- 拟改:`BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:Lifecycle / States` → `Entity identity 对外只有 stable / unknown；Entity Log 只保存 stable。active / merged / archived 是 stable entity record 的 lifecycle availability，不是 candidate state，也不代表会计分类、rule 或 automation permission。`
- 排除的替代 + 理由:
保留 durable candidate entity。理由是它会让下游误以为可以按不稳定身份索引 Case Log / Rule Match，违背已锁 `{stable, unknown}` 前提。

### Stable entity 创建与 publication 的 authority
- 结论:
Entity Log 可以接收两类 stable creation authority：ER 基于可追溯 evidence 输出 `new_stable_entity`；accountant 在 pending / clarification 中明确确认 identity。两者都不需要 governance approval，因为它们只确认“这是谁”。写入内容限于 entity 本体与最小创建 provenance；不同时写 Alias、automation policy、active rule 或 Case Log。创建执行机制、统一 finalization 与批内顺序属于 L4 / seam-park。
- 为什么(锚定核心产品目标的哪条):
支持记忆复用、自动化率和审计性。identity 已经明确时应尽快成为可查询权威；但不能把 identity creation 偷渡成会计规则或自动化权限。
- 拟改:`BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:写入者` → `Entity Resolution Node 只可以发起 / 执行 new stable entity 本体 publication；accountant explicit identity confirmation 可以成为 stable entity creation authority。两者写入范围仅限 stable entity body 与最小 creation provenance。Alias write intent、Case Log write、Rule Log write、automation policy mutation 和 Transaction Log finalization 均不随 entity 本体隐式发生。`
- 拟改:`BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:Mutation Path` → `ER stable creation 与 accountant explicit identity confirmation 都可以形成 stable entity creation authority；如果 accountant 只给分类、未确认主体，交易可以 finalize 到 Transaction Log，但不得创建 Entity Log 记录，也不得创建 candidate entity。`
- 排除的替代 + 理由:
等交易 finalization 后才统一让 stable entity 可见。理由是 stable identity 是后续同 batch / 后续交易的身份基础，不应被会计分类完成时点阻塞。
- 浮现的新 open boundary:
ER publication 与 accountant confirmation creation 是否共用同一个 finalization writer、以及失败重试 / 幂等机制，属于 L4 / seam-park。

### Entity Log 与 Alias / Rule / automation 的契约边界
- 结论:
Entity Log 只提供 stable entity identity 与 lifecycle availability。Alias relationship 的 authority 归 Alias Log 或其后续冻结的等价 Alias 库 / 投影；Rule authority 归 Rule Log；automation permission / relaxation 依赖 Rule / Governance authority。Entity Log 可以暴露与 entity 相关的引用、投影或阻断信号，但下游不得把 `active` entity、Alias ref、历史案例数量或 automation note 当成 rule match 或自动落账许可。
- 为什么(锚定核心产品目标的哪条):
支持契约面、审计性和 accountant control。身份权威、身份复用、确定性规则和自动化权限必须分层，否则一次 identity 命中会被误解释成会计处理授权。
- 拟改:`BK_Copilot/memory_layers/entity_log/01_memory_intent.md:绝不保存什么` → `Entity Log 不保存 active rule payload、rule condition、accounting treatment、Case Log precedent、Alias authority 的内部匹配规则，或 automation permission 的审批事实。若为查询方便保留 refs / projections，下游只能按对应 source authority 的契约使用。`
- 拟改:`BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:Source of Truth / Authority` → `Alias relationship 的 source of truth 是 Alias Log / 等价 Alias 库；Rule authority 的 source of truth 是 Rule Log；automation policy / permission 的 approval authority 依赖 Governance / Rule 边界。Entity Log active status 只表示可作为 identity target，不代表自动分类或自动落账。`
- 拟改:`BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:读取者` → `Rule Match 读取 Entity Log 只能确认 stable entity 是否为 active identity target，以及是否存在已生效治理阻断；Rule Match 必须另读 Rule Log 来判断 rule scope。`
- 排除的替代 + 理由:
把 automation_policy 作为 Entity Log 内部可自行放宽的字段。理由是自动化放宽影响会计师控制权和审计风险，不能由 identity store 单独决定。
- 浮现的新 open boundary:
automation_policy 是否作为 Entity Log 字段、Governance Log projection 或 Rule / Governance 联合视图呈现，依赖 Governance / Rule 设计。

### Merge / split 后的身份使用与下游阻断语义
- 结论:
Merge / split 的 L2 核心是 identity target 是否仍可被安全复用，而不是在 Entity Log 内迁移所有下游记忆。Merge 后，被合并 entity 不再作为 active target；若 surviving entity 唯一且治理已生效，下游可以通过明确 supersession / lifecycle projection 读取到 surviving entity。Split 或多归属竞争默认阻断自动身份复用；旧 entity 及其 Alias / Case / Rule / automation 关系不得自动继承到任一新 entity，直到对应层获得明确重新归属 authority。Entity Log 只发布 identity lifecycle / supersession 语义，不直接改写 Alias Log、Case Log 或 Rule Log。
- 为什么(锚定核心产品目标的哪条):
支持审计性、记忆复用和 accountant control。merge 可以减少重复 identity；split 会让历史记忆的适用对象变得不唯一，自动迁移会污染案例、规则和身份复用。
- 拟改:`BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:Lifecycle / States` → `merged entity 不能作为新的 active identity target；它只能通过明确 surviving entity ref / supersession projection 指向仍可用的 stable entity。split 后的原 entity 默认不能作为新交易 active target，除非治理 / correction 已明确它仍代表一个唯一 stable identity。exact enum 和字段留到 M3。`
- 拟改:`BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:与其他 memory/log 的边界` → `Entity Log lifecycle mutation 不自动迁移 Alias、Case 或 Rule authority。Alias 迁移归 Alias Log；Case read-through / supersession 归 Case Log；Rule / automation 迁移或阻断归 Rule Log / Governance。Entity Log 只提供 identity lifecycle 和 supersession authority，供下游按各自契约处理。`
- 拟改:`BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:冲突处理` → `如果 current runtime signal、Alias、Case 或 Rule 指向的 entity 已 merged / archived / split-invalid，reader 必须保守阻断 deterministic identity reuse 或 automation path，并生成 review / governance candidate；不得静默改指到某个新 entity。`
- 排除的替代 + 理由:
merge / split 时由 Entity Log 批量迁移所有 Alias、Case、Rule。理由是这些层的 authority 条件不同；Entity Log 只能改变身份生命周期，不能替其他记忆层判断历史案例或规则是否仍适用。
- 浮现的新 open boundary:
Case Log 是否支持 merge 后 read-through、Rule 是否可迁移、Alias 如何批量 supersede，分别进入 Case Log / Rule Match / Alias 的 L2 或 L4 处理。

### Person entity 与 Profile 的最小边界
- 结论:
Role 字段不复活。Entity Log 只在“交易主体本身是这个 person / payee / counterparty”时保存 person stable entity；员工、owner、关联方、loan party、bank account owner 等结构性关系由 Profile / Governance 类对象决定，不能作为 Entity Log 内部 role 或 relationship 字段偷渡。完整 person/Profile 边界依赖 Profile 审计，Entity Log L2 只冻结“不保存 Role、不把 profile fact 当 entity identity authority”。
- 为什么(锚定核心产品目标的哪条):
支持契约面和审计性。交易身份与客户结构事实不同；混在 Entity Log 会让身份判断、Profile 事实和会计处理权限互相污染。
- 拟改:`BK_Copilot/memory_layers/entity_log/01_memory_intent.md:保存什么` → `Entity Log 可以保存 person stable entity，仅当该 person 是交易的 stable identity target。Entity Log 不保存 Role 字段，也不保存员工 / owner / 关联方等 Profile relationship fact。`
- 拟改:`BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:与其他 memory/log 的边界` → `Profile 是客户结构事实的 authority；Entity Log 是交易主体身份 authority。Profile fact 可以作为 ER / review 的 context，但不能直接替代 Entity Log stable identity。`
- 浮现的新 open boundary:
person entity 与 Profile employee / owner / related-party facts 的完整边界依赖 Profile 审计，当前挂起。

## 分类备案
- `entity_record` exact field schema → L3(留联合 L3)
- `new_stable_entity` 最小创建 provenance exact field schema → L3(留联合 L3)
- stable entity lifecycle enum、supersession ref、split representation 的 exact field / enum → L3(留联合 L3)
- Alias ref / projection 在 Entity Log 中是否出现及其字段形态 → L3(留联合 L3)
- Entity Log 与 Alias Log / Alias 库的存储、索引、投影关系 → L4 / seam-park(推给记录层)
- ER publication、accountant confirmation creation、幂等、失败重试、批内可见性的执行机制 → L4 / seam-park(推给记录层)
- merge / split 后 Alias / Case / Rule / automation 的批量迁移、阻断和修复执行机制 → L4 / seam-park(推给记录层)
- automation_policy 与 Governance Log / Rule Log 的 projection boundary、upgrade / relaxation authority → L2·外阻(依赖 Governance / Rule,挂起)
- person entity 与 Profile employee / owner / structural facts 的完整边界 → L2·外阻(依赖 Profile,挂起)
- automation_policy 自动降级 mutation contract 与治理可见性机制 → L4 / seam-park(推给记录层)
- case-derived entity risk / automation risk candidate 的审批路径 → L2·外阻(依赖 Governance / Case / Rule,挂起)

## 自检与判定
- 按模板 L2 清单自检:通过;未过项 = 第21条不能进 M3，因为 entity schema、provenance schema、lifecycle enum、projection fields、Governance / Profile 外部边界尚未冻结。
- 本对象 L2 可否判定完成:是。Entity Log 自身的 L2 authority、creation、lifecycle 使用语义和契约边界已收口；剩余项已归入联合 L3、L4 / seam-park 或 L2·外阻。
