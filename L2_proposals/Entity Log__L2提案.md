# Entity Log L2 提案

## 依赖

- Entity Resolution 已锁定：entity identity 对外只有 `stable / unknown` 两态；stable 只表示主体身份已清楚，不包含会计分类、Rule 或自动化权限。
- Entity Resolution 已锁定：`new_stable_entity` 的及时 publication 只确认 entity 本体和最小创建 provenance；Alias、Rule、automation policy 不随本体一起确认。
- Alias Log 已锁定：Alias 是已确认 transaction surface text -> stable entity 的身份复用关系；Alias Log 的唯一已冻结直接 reader 是 Entity Resolution。

## L2 决策

### Entity Log 是 stable entity 的 identity authority 主档案

- 结论:
  Entity Log 仍应定义为 identity authority store，但这里的 identity authority 不是泛化的所有业务记忆，而是以 stable entity 为主体的身份权威主档案。

  它回答：客户账务语境中这个 stable entity 是谁、当前是否可作为 active identity target、生命周期状态如何、有哪些已确认身份相关属性、这些权威从哪些 evidence / governance / creation trace 来。

  它不回答：这笔交易怎么分类、使用哪个 COA、HST / GST 怎么处理、journal entry 如何生成、Rule 是否命中、是否允许自动落账。

- 为什么(锚定核心产品目标的哪条):
  该决策支持记忆复用、有证据支持的建议、审计性和 accountant control。系统需要一个 durable source of truth 来承载 stable entity 身份本体，否则 Entity Resolution、Case、Rule 和后续审计都会在“这是谁”这个基础问题上重复判断或互相偷用非权威来源。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/memory_layers/entity_log/00_index.md:一句话定义` →
  `Entity Log 是 stable entity 的 identity authority 主档案：它以 stable entity 为主体，保存客户账务语境中某个 entity 是谁、其生命周期状态、已确认身份相关属性、authority trace、evidence refs 和 governance refs。Entity Log 不保存 classification memory、case precedent、rule condition、journal entry、transaction audit trail 或未经批准的 AI reasoning。`

  `BK_Copilot/memory_layers/entity_log/01_memory_intent.md:为什么这层 memory 必须存在` →
  `Entity Log 必须存在，是因为系统需要一个 durable、可审计、以 stable entity 为主体的身份权威主档案。runtime handoff 只能表达当前交易上下文，Transaction Log 只保存最终交易审计记录，Case Log / Rule Log 分别保存案例记忆和规则权威；它们都不能替代 stable entity identity 的 source of truth。`

- 排除的替代 + 理由:
  排除“Entity Log 只是 entity body 的薄表，Alias、生命周期和身份属性都交给其他 log”。理由是这会让 stable entity 主档案被拆散，下游无法通过 Entity Log 恢复一个 stable entity 的完整身份权威状态。

- 浮现的新 open boundary:
  `entity_record` 的 exact field schema、authority trace 字段、evidence / governance refs 字段形态留联合 L3。

### Alias 属于 stable entity 的身份属性，Alias Log 是面向 ER 的反向查询 projection

- 结论:
  Alias 的主语义归属于 stable entity。Entity Log 可以记录某个 stable entity 下有哪些已确认 Alias；这些 Alias 是该 entity 承载的身份属性之一。

  Alias Log 不是另一套与 Entity Log 平行竞争的身份权威，而是围绕 Alias / transaction surface text 建立的反向查询 log / projection / index。它以 Alias 为主体，服务 Entity Resolution 用当前交易 description / descriptor / raw bank surface text 快速查找对应 stable entity，避免逐个扫描 Entity Log。

  因此，Entity Log 与 Alias Log 的区别不是“谁拥有 Alias 语义”，而是查询方向和接口不同：

  - Entity Log: entity -> aliases，回答“这个 stable entity 下有哪些已确认 surface text”。
  - Alias Log: alias -> stable entity，回答“这个当前 surface text 是否命中已确认 stable entity”。

  Rule Match 不直接读取 Alias Log。Rule Match 接收 Entity Resolution 输出的 stable entity，并以 Entity Log 中该 stable entity 的权威信息作为 entity 信息来源；如 Rule Match 需要理解该 entity 的 alias context，该信息应来自 Entity Log 的 entity-centered 记录，而不是把 Alias Log 当成 Rule Match reader。

- 为什么(锚定核心产品目标的哪条):
  该决策支持契约面最小化、记忆复用和自动化率。Entity Log 保留 stable entity 的完整身份主档案，避免 identity authority 被拆碎；Alias Log 提供高效、低上下文负担的 ER lookup 入口，避免 ER 为每笔交易遍历所有 entity。两者共享同一身份语义，但服务不同访问路径。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/memory_layers/entity_log/01_memory_intent.md:保存什么` →
  `Entity Log 以 stable entity 为主体保存该 entity 的身份相关属性，包括已确认 Alias。这里的 Alias 指过去已经确认过的 transaction surface text -> stable entity 身份复用关系；在 Entity Log 中，它表现为该 stable entity 承载的身份属性之一。`

  `BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:Source of Truth / Authority` →
  `Entity Log 是 stable entity identity 主档案。Alias 的 entity-centered 语义归属于 Entity Log：某个 stable entity 下有哪些已确认 Alias，是该 entity 身份权威状态的一部分。Alias Log 是基于这些 Alias 建立的 alias-centered 反向查询 projection / index，用于 Entity Resolution 从当前 surface text 查询 stable entity。`

  `BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:读取者` →
  `Rule Match Node 读取 Entity Resolution 已确认的 stable entity，并以 Entity Log 中该 stable entity 的权威信息作为 entity context 来源。Rule Match 不直接读取 Alias Log；Alias lookup 位于上游 Entity Resolution。`

  `BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:Source of Truth / Authority` →
  `Alias Log 是 Alias-centered lookup projection / index，不是独立于 Entity Log 的第二套 stable entity authority。Alias Log 服务 Entity Resolution 的 surface text -> stable entity 查询；stable entity 主档案和 entity-centered Alias context 归 Entity Log。`

- 排除的替代 + 理由:
  排除“Alias 完全不属于 Entity Log，只属于独立 Alias Log”。理由是这会把 stable entity 的身份属性拆离主档案，使 Entity Log 无法回答一个 entity 承载了哪些已确认 surface text。

  排除“Rule Match 直接读取 Alias Log”。理由是这会让 Alias Log 从 ER lookup projection 扩张成横向业务信息源，模糊 Alias、Entity 和 Rule 的 authority 边界；Rule Match 的身份基础应来自 ER stable 输出和 Entity Log。

- 浮现的新 open boundary:
  Entity Log 与 Alias Log 的物理存储关系、同步方式、projection 构建方式、查询索引形态和多 log finalization 机制属于 L4 / seam-park。

### Entity Log 的保存集合只分五类：身份脊梁 / 身份状态与重定向 / 身份复用面 / 控制状态 / 追溯指针

- 结论:
  Entity Log 在 L2 需要保存的信息只分五类。判别准则统一为一句：「下游是否存在某个 runtime 判断或控制动作，必须就地、确定性地读到这个状态」——是则进 Entity Log；只是事后想知道「为什么」则属 trace 或归其他 log。L2 只锁信息类型与作用，不锁字段 schema、enum 或 refs 形态（留 L3）。

  五类如下（括号内为类型示例，非冻结字段名）：

  1. 身份脊梁（identity backbone）：稳定 entity 主键、展示名、entity 类型（如 `entity_id` / `display_name` / `entity_type`）。
     作用：铸造并担保一个跨时间不变的身份引用。全系统 entity-indexed 记忆（Case Log 主索引、Alias Log 指向、围绕 stable entity 建立的 Rule）的索引键最终都指向它；它是参照完整性的脊梁。

  2. 身份状态与重定向（status & supersession）：entity 生命周期状态（至少 active / merged / archived 的语义）与 merge / split 后的当前 supersession 指向（旧 entity → surviving entity）。
     作用：让 runtime 立即知道这个身份还作不作数、作废后跳转到谁，避免 merge / split 静默腐蚀下游所有引用。状态只表示能否作为 active identity target，不代表自动分类许可。注意这里存的是「当前重定向状态」，不是「合并/拆分的事件历史」（事件归 Governance Log）。

  3. 身份复用面（identity reuse surface）：该 entity 名下已确认的 Alias 集合。
     作用：让未来相同 surface text 不必从零重新解析身份。其 authority 归属与 Alias Log 投影边界已在决策 2 锁定（authority 在 Entity Log，Alias Log 是反查投影）。

  4. 控制状态（entity-level control state）：entity 级 automation policy 与身份级 risk 标记（详见下一决策）。
     作用：作为 per-entity 自动化的确定性卡点，以及会计师可写的控制旋钮。

  5. 追溯指针（trace refs）：authority / 创建来源、evidence refs、governance refs、superseded-by 等「只存引用不存正文」的指针。
     作用：支撑 review / correction / audit 能回查「凭什么」。它们本身不构成 authority，正文留在 Evidence Log / Governance Log 等 source of truth。

- 为什么(锚定核心产品目标的哪条):
  支持记忆复用、审计性、accountant control、自动化率。把 Entity Log 的分量锚在「全系统身份引用的脊梁 + per-entity 控制面」上，而不是字段丰富度上；凡是「和这个 entity 有关」但不满足上述判别准则的信息，不应被吸进 Entity Log。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/memory_layers/entity_log/01_memory_intent.md:保存什么` →
  `Entity Log 保存的信息只分五类，判别准则是「下游是否有 runtime 判断或控制动作必须就地确定性读到该状态」：(1) 身份脊梁——稳定 entity 主键、展示名、entity 类型，作为全系统 entity-indexed 记忆索引键的稳定引用；(2) 身份状态与重定向——生命周期状态（active / merged / archived 语义）与 merge / split 后当前 supersession 指向，只表示能否作为 active identity target，不代表自动分类许可；(3) 身份复用面——该 entity 名下已确认 Alias 集合（authority 归属见 Alias projection 边界）；(4) 控制状态——entity 级 automation policy 与身份级 risk 标记；(5) 追溯指针——authority / created_from / evidence refs / governance refs / superseded-by 等只存引用不存正文的指针。具体字段 schema、enum 与 refs 形态留 M3。`

- 排除的替代 + 理由:
  排除「把 Entity Log 削成身份铭牌（只剩名字 + Alias + 生命周期 + refs），控制状态全部每次从 Case Log 现推」。理由：自动化卡点必须确定性、可被会计师直接设定；append-only 的 case 学习证据无法承载可写控制旋钮，每次现推还破坏确定性与运行效率，并使 Entity Log 失去其控制面价值。

- 浮现的新 open boundary:
  五类各自的 exact field schema、enum、refs 形态留 L3。

### automation_policy 的 entity 级控制语义归 Entity Log（不标外阻）

- 结论:
  automation_policy 的 entity 级控制语义在 L2 归 Entity Log（属保存集合第 4 类），不标 L2·外阻。它是「被治理设定、由 case 证据支撑、在 entity 级执行」的控制状态，对应三处分放：证据（这个 entity 历史分类是否稳定）归 Case Log；当前控制决定 / 状态归 Entity Log；审批事件历史归 Governance Log。升级 / 放宽必须 accountant approval；系统只能在受控边界内自动降级，且必须保留治理可见性。

  身份级 risk 标记同理归 Entity Log，但只保留「会在 runtime 当卡点用」的身份风险（如疑似同名、疑似 merge、identity conflict）；「分类不稳定」这类 case 衍生风险不在此，属 Case Log / governance candidate。

- 为什么(锚定核心产品目标的哪条):
  支持 accountant control 与自动化率。accountant control 要求一个 durable、entity 级、可写的控制旋钮；自动化执行要求一个确定性卡点。两者都只能落在 Entity Log，否则会计师失去直接控制入口、runtime 失去确定卡点。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:Source of Truth / Authority` →
  `automation_policy 与身份级 risk 标记的 entity 级控制语义归 Entity Log：Entity Log 存的是当前控制决定 / 状态，不存支撑它的 case 证据（归 Case Log），也不存审批事件历史（归 Governance Log）。automation policy 升级 / 放宽必须 accountant approval；系统自动降级仅限受控收紧并保留治理可见性。身份级 risk 标记只保留会作为 runtime 卡点的身份风险，不含分类不稳定等 case 衍生风险。`

- 排除的替代 + 理由:
  排除「automation_policy 整体标 L2·外阻、全部推给 Governance / Rule Match」。理由：证据（Case）/ 决定（Entity）/ 审批事件（Governance）三分本就让边界更干净；把控制状态整体推走会让 Entity Log 失去控制面，也让会计师失去可写卡点。

- 浮现的新 open boundary:
  automation_policy 是「直接存于 Entity Log」还是「由 Governance Log 投影生成」属存储 / 投影形态，留 L4 / open boundary；自动降级的 exact approval / visibility contract 留 L4。

### Entity Log 不保存什么：易误归的 entity 相关信息按归属分流

- 结论:
  以下信息虽「围绕 entity」，但不进 Entity Log，按归属分流。识别准则：会随交易演化、有条件的、属事件历史或原始正文的，都不是身份 / 控制状态。

  - 分类历史 /「这个 entity 通常记成哪个 COA」/ repeated outcome pattern → Case Log。
  - 建在 entity 上的 rule 内容（entity-level rule、pattern 条件、active rule payload）→ Rule Log；Entity Log 顶多有指针 / 标志，不存规则体。
  - 该 entity 的逐笔交易历史与完整审计轨迹 → Transaction Log；Entity Log 不得变成「按 entity 索引的流水账」。
  - 身份判断背后的推理与原始证据正文 → Evidence Log；Entity Log 只存 ref。
  - merge / split / 审批的事件历史 → Governance Log；Entity Log 只存「当前 supersession 状态 + ref」。

- 为什么(锚定核心产品目标的哪条):
  支持审计性与契约面最小化。防止 Entity Log 从「身份 + 控制权威」膨胀成又管分类、又管规则、又管流水的大杂烩，保住 evidence / memory / judgment / governance / audit 的清晰分层。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/memory_layers/entity_log/01_memory_intent.md:绝不保存什么` →
  `Entity Log 绝不保存以下「围绕 entity 但不属身份 / 控制状态」的信息，按归属分流：分类历史 / 常用 COA / repeated outcome pattern → Case Log；entity-level rule、pattern 条件、active rule payload → Rule Log（至多存指针 / 标志）；逐笔交易历史与完整审计轨迹 → Transaction Log；身份判断推理与原始证据正文 → Evidence Log（只存 ref）；merge / split / 审批的事件历史 → Governance Log（只存当前 supersession 状态 + ref）。`

- 排除的替代 + 理由:
  排除「为支持 Rule promotion 而在 Entity Log 缓存全部分类结果」。理由：Rule promotion 的历史依据应读 Case Log；在 Entity Log 缓存分类结果会复制 case 记忆、制造双 source of truth、破坏审计分层。

- 浮现的新 open boundary:
  无。

### Stable Entity 创建入口只收口为 ER new_stable_entity 与 accountant explicit identity confirmation

- 结论:
  Stable entity 的 L2 创建入口目前只确认两类：

  1. Entity Resolution 判定 `new_stable_entity`，并在下游继续处理前发起及时 Entity Log publication。
  2. Accountant 在 pending / clarification / review 路径中明确确认 identity，使该主体可以创建为 stable entity。

  两类入口都只确认 stable entity 本体与最小创建 provenance 可以进入 Entity Log；不同时确认 Alias、Rule、automation policy、Case Log 或 final transaction outcome。两类入口也都不需要 governance approval，因为它们是在建立身份主档案，而不是执行高权限治理变更。

  Accountant 只确认会计分类、但没有明确确认 identity 时，不创建 stable entity，也不写 Entity Log。

- 为什么(锚定核心产品目标的哪条):
  支持记忆复用、审计性和 accountant control。identity 已经清楚时应及时进入 Entity Log，供后续同 batch / 后续交易复用；identity 没被明确确认时，系统不能为了沉淀记忆而伪造 stable entity。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:Mutation Path` →
  `Stable entity 的 L2 创建入口只确认两类：Entity Resolution 判定 new_stable_entity 后发起及时 Entity Log publication；accountant explicit identity confirmation 后创建 stable entity。两者均只确认 entity 本体与最小创建 provenance，不隐含 Alias、Rule、automation policy、Case Log 或 final transaction outcome 写入。Accountant 只确认分类、未确认 identity 时，不创建 stable entity。实际写入执行者、调用方式和多 log finalization 属 L4 / seam-park。`

- 排除的替代 + 理由:
  排除「Case Log repeated outcome、Rule promotion candidate 或 Transaction Log finalization 自动创建 stable entity」。理由是这些对象可能证明账务处理稳定，但不能替代 explicit identity authority；让它们自动创建 entity 会污染 Entity Log 的身份主档案。

- 浮现的新 open boundary:
  两类入口的 exact write mechanism、触发顺序、finalization mechanism 与 provenance 字段形态分别留 L4 / L3。

### Stable Entity lifecycle 最小状态为 active / merged / archived

- 结论:
  Stable entity lifecycle 的 L2 最小状态收口为 `active` / `merged` / `archived` 三类语义。

  - `active`: 该 stable entity 当前可作为 runtime stable identity target。
  - `merged`: 该 entity 已并入另一个 surviving entity，不能再直接作为 active identity target；runtime 应按当前 supersession 指向跳转。
  - `archived`: 该 entity 不再作为常规 active identity target，但可保留历史解释和审计引用。

  Automation policy 不属于 lifecycle state。它是 entity-level control state，用来控制自动化卡点；lifecycle state 只回答 identity target 是否仍有效。

- 为什么(锚定核心产品目标的哪条):
  支持记忆复用、审计性和自动化安全。下游需要一个最小、确定的身份状态语义，避免已合并或归档 entity 被继续当作 active identity target 使用；同时避免把 automation permission 混进 identity lifecycle。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:Lifecycle / States` →
  `Stable entity lifecycle 的 L2 最小状态为 active / merged / archived。active 表示可作为 runtime stable identity target；merged 表示已并入 surviving entity，不能直接作为 active target；archived 表示不再作为常规 active target，但可用于历史解释。Lifecycle state 不代表自动分类许可；automation_policy 是独立的 entity-level control state。`

- 排除的替代 + 理由:
  排除「把 automation policy、risk level 或 rule eligibility 做成 lifecycle state」。理由是这些是控制或自动化判断，不是身份是否仍有效的生命周期语义。

- 浮现的新 open boundary:
  exact enum、退出 / 回滚规则、archived 恢复条件与具体 validation 留 L3 / L4。

### Merge / split 后 Entity Log 只负责 Entity 侧身份状态与重定向

- 结论:
  Entity merge / split 后，Entity Log 只负责记录 Entity 侧身份状态与当前重定向语义，不负责执行 Alias、Case 或 Rule 的迁移 / 重判。

  - merge: Entity Log 记录旧 entity 的 `merged` 状态，以及指向 surviving entity 的当前 supersession / redirection。
  - split: Entity Log 记录原 entity 的身份边界已被治理重判；新的 stable entities 作为新的身份主档案存在。原 entity、旧 aliases、旧 cases、旧 rules 如何归属，不能由 Entity Log 自动决定。

  Alias 归属重判、Case Log 记录归属或限制、Rule eligibility / 阻断 / 迁移，都应由 Governance / correction / finalization 层统筹，并分别落到 Alias Log、Case Log、Rule Log 或 Governance Log 的对应 authority 中。Entity Log 可以保存当前 identity 状态和治理 refs，但不保存跨 log 迁移事件历史。

- 为什么(锚定核心产品目标的哪条):
  支持审计性、accountant control 和契约面最小化。merge / split 会改变 identity 边界，Entity Log 必须让下游知道哪个 entity 仍可作为身份目标；但 Alias、Case、Rule 的重判属于不同 authority，不应由 Entity Log 静默迁移或推断。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:与其他 memory/log 的边界` →
  `Entity merge / split 后，Entity Log 只保存 Entity 侧当前身份状态、surviving entity / supersession 指向和 governance refs。Alias 归属重判归 Alias Log / Governance；Case 归属、限制或 supersession 归 Case Log / Governance；Rule eligibility、阻断或迁移归 Rule Match / Rule Log / Governance。Entity Log 不执行跨 log 迁移，也不保存完整治理事件历史。`

- 排除的替代 + 理由:
  排除「merge / split 后由 Entity Log 自动迁移 Alias、Case 和 Rule」。理由是这会让 Entity Log 越权承担 Alias authority、case precedent authority 和 rule authority，且容易在 identity 边界被重判时静默污染历史记忆。

- 浮现的新 open boundary:
  merge / split 的治理审批、批量重判、跨 log finalization、audit trace 和回滚机制属于 Governance / correction / finalization 的 L4 / seam-park。

### Person 类型 stable entity 与其他 entity 使用同一套 Entity Log 标准

- 结论:
  `person` 只是 stable entity 的一种 entity_type，不构成 Entity Log 的特殊外阻。只要一笔交易进入 Entity Resolution，并被识别为 stable entity，或经 accountant explicit identity confirmation 成为 stable entity，无论该 entity 是 person、vendor、organization、payee 还是其他类型，都按同一套 Entity Log 标准记录。

  Profile 与 Entity Log 属于两条不同处理逻辑：绝大多数由 Profile / routing 直接处理、没有进入 Entity Resolution 的交易，不会也不需要进入 Entity Log。反过来，如果某个 person 确实在 ER / accountant confirmation 路径中成为 stable entity，Entity Log 可以保存它的身份脊梁、身份状态、Alias、控制状态和追溯指针。

  因此，Person / Profile 边界不再作为 Entity Log L2·外阻。真正的分界不是「这个主体是不是人」，而是「这笔交易是否进入 ER 并形成 stable entity」。Profile 结构事实的具体建模仍由 Profile 自己负责；Entity Log 只按 stable entity 标准保存已经进入身份主档案的 person entity。

- 为什么(锚定核心产品目标的哪条):
  支持系统一致性和记忆复用。Entity Log 的准入标准应由 identity resolution 路径和 stable entity 资格决定，而不是由 entity_type 决定；否则 person 类型会被不必要地特殊化，削弱 Entity Log 作为统一 stable identity 主档案的作用。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/memory_layers/entity_log/01_memory_intent.md:保存什么` →
  `Entity Log 对 person、vendor、organization、payee 等 entity_type 使用同一套保存标准：凡是经 Entity Resolution 或 accountant explicit identity confirmation 成为 stable entity 的主体，都可以进入 Entity Log。Profile / routing 直接处理且未进入 Entity Resolution 的交易，不因此创建 Entity Log 记录。`

  `BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md:与其他 memory/log 的边界` →
  `Profile 与 Entity Log 是不同处理逻辑。Profile 负责其自身结构事实和直接处理路径；Entity Log 只保存进入 ER / accountant confirmation 路径后形成的 stable entity 主档案。person 类型 stable entity 不特殊排除，也不因 Profile 存在而自动挂起。`

- 排除的替代 + 理由:
  排除「person entity 与 Profile employee / owner facts 一律作为 Entity Log L2·外阻挂起」。理由是这会把 entity_type 当成准入边界；实际准入边界应是是否进入 ER 并形成 stable entity。

- 浮现的新 open boundary:
  Profile 自身如何处理 employee / owner / payroll / shareholder 等结构事实，不属于 Entity Log L2；若后续 Profile 需要与 Entity Log 建立 ref / projection，留到 Profile 或联合 L3 / L4 处理。

## 分类备案

- `entity_record` exact field schema、authority trace 字段、Alias 字段形态 → L3(留联合 L3)
- `alias_record` exact field schema、normalization / equivalence matching rule → L3(留联合 L3)
- 五类保存信息各自的 field schema、enum（含 entity_status、automation_policy、risk 标记的取值）、refs 形态 → L3(留联合 L3)
- stable entity 创建入口的 provenance 字段形态、entity_status exact enum、lifecycle validation rules → L3(留联合 L3)
- automation_policy 是「直接存于 Entity Log」还是「由 Governance Log 投影生成」的存储 / 投影形态、自动降级 exact approval / visibility contract → L4 / seam-park(推给记录层)
- Entity Log 与 Alias Log 的物理存储关系、projection / index 构建方式、同步机制、写入顺序 → L4 / seam-park(推给记录层)
- ER 认定 `new_stable_entity` 后的实际写入者、调用方式、写入顺序和多 log finalization → L4 / seam-park(推给记录层)
- accountant explicit identity confirmation 后的实际写入者、调用方式、写入顺序和多 log finalization → L4 / seam-park(推给记录层)
- merge / split 的治理审批、批量重判、跨 log finalization、audit trace 和回滚机制 → L4 / seam-park(推给记录层)
- Profile 与 Entity Log 的 ref / projection 形态、Profile 结构事实是否回显到 Entity 视图 → L3 / L4(留联合设计，不阻塞 Entity Log L2)

## 自检与判定

- 按模板 M1-M2 清单自检: 通过。Entity Log 的存在理由、保存集合、不保存边界、reusable authority / trace 分区、Alias projection 边界、automation_policy 归属、创建入口、lifecycle、merge / split Entity 侧边界、person 类型 stable entity 边界均已收口；剩余项均已分类为 L3 或 L4 / seam-park。
- 本对象 L2 可否判定完成: 是。Entity Log L2 已完成；字段、enum、refs、存储 / projection、写入执行者、多 log finalization、Profile ref / projection 形态均留后续联合 L3 / L4。
