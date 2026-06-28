> ⚠️ **已过时 / superseded（owner 2026-06-27）** — 本 L2 提案的 L1 / L2 结论已由 `BK_Copilot/memory_layers/alias_log/` 正式草案取代，仅作历史来源保留。当前权威以 `BK_Copilot/` 正式草案 +（产出后）对应 L3 schema 为准；**勿据本文件回灌已删除的概念 / 字段**。

# Alias Log L2 提案

## 依赖

- Entity Resolution 已锁定：entity identity 对外只有 `stable / unknown` 两态；stable 只表示主体身份已清楚，不包含会计分类、Rule 或自动化权限。
- Entity Log 已锁定：stable entity identity 的 source of truth 是 Entity Log；Alias Log 只能指向明确 stable entity，不能创建 stable entity。
- Entity Resolution L2 已锁定：`new_stable_entity` 的及时 publication 只确认 entity 本体；Alias 是否写入由 Alias Log 的 L2 规则决定。

## L2 决策

### Stable Entity 确认后的 Description 写入资格

- 结论:
  当一笔交易已经被确认指向某个 stable entity 时，该交易的 raw transaction description / descriptor / raw bank surface text 具备进入 Alias Log 的写入资格，可以记录为 confirmed transaction surface text -> stable entity 的 identity reuse relationship。

  这项资格不要求该 description 是本次 stable identity 判断的决定性证据。即使 stable entity 是通过 receipt、invoice、cheque、contract、联网搜索或其他 evidence 确认，只要该交易最终被确认指向该 stable entity，该交易自身的 raw description 就可以作为未来 Entity Resolution 可查询的 Alias Log 记忆。

  Alias Log 不负责判断某类交易是否应该进入 Entity Resolution，也不负责判断该交易是否应该通过主体识别来完成最终会计分类。如果某类交易应由 Profile、上游 routing 或其他节点直接处理，那是圈外 / 上游流程边界，不由 Alias Log 在写入资格中额外过滤。

- 为什么(锚定核心产品目标的哪条):
  该决策支持记忆复用和自动化率提升。Alias Log 的核心价值不是复述第一次 ER 已经使用过的 evidence，而是把已经确认过的 transaction surface text -> stable entity 关系沉淀成未来身份识别入口。未来遇到相同 raw description 时，Entity Resolution 可以通过 Alias Log 复用 stable entity identity，减少重复联网搜索、重复 evidence 读取或重复询问 accountant。

  该关系只支持 identity reuse，不代表 COA、HST / GST、journal entry、Rule Match、Case precedent 或自动化落账权限。

- 拟改:
  `BK_Copilot/memory_layers/alias_log/01_memory_intent.md:保存什么` →
  `Alias Log 保存已确认交易中的 raw transaction description / descriptor / raw bank surface text 到 stable entity 的对应关系。只要该交易最终被确认指向明确 stable entity，该 raw surface text 就具备进入 Alias Log 的资格；不要求该 surface text 是本次 stable identity 判断的决定性证据。`

  `BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:写入者` →
  `Alias Log 的 L2 写入资格是：交易已确认指向明确 stable entity，且写入对象是该交易可追溯的 raw transaction surface text -> stable entity 关系。Alias Log 不重新判断 stable entity 是否成立；stable entity authority 来自 Entity Log / ER 已确认 identity。实际 writer、approval path、finalization mechanism 和写入顺序不在 L2 冻结。`

  `BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:Mutation Path` →
  `已冻结资格约束：confirmed transaction -> stable entity + raw transaction surface text -> may become Alias Log identity reuse relationship。该资格不要求 surface text 曾经作为 ER 决定性 evidence；它的用途是让未来 Entity Resolution 可以通过已确认 surface text 复用 stable entity identity。`

- 排除的替代 + 理由:
  排除“只有当 description 是本次 stable identity 判断的决定性 evidence 时才允许进入 Alias Log”。理由是该替代会把 Alias Log 限缩为 ER evidence trace，削弱它作为未来身份检索 / 自动化入口的核心价值。

  排除“由 Alias Log 过滤哪些交易类型不该走 Entity Resolution”。理由是这会让 Alias Log 承担上游 routing / Profile 边界，模糊 memory layer 职责；Alias Log 只定义已确认 surface text -> stable entity 关系能否成为 durable identity reuse memory。

- 浮现的新 open boundary:
  未来 Entity Resolution 如何使用 Alias Log match 结果，以及 exact match、受控 equivalence、向量 / semantic retrieval hint 的 authority 边界，仍需在 Alias Log L2 后续问题中收口。

### Exact Description Match 的 ER Authority

- 结论:
  当 Entity Resolution 遇到的当前 raw transaction description 与 Alias Log 中已确认的 Alias 完全一致时，ER 可以直接复用该 Alias 指向的 stable entity，并输出 stable entity identity。

  Alias Log 的设计初衷就是为 Entity Resolution 提供一条额外的身份检索机制：如果过去某个 raw description 已经在确认交易中指向 stable entity，那么未来再次遇到同一 raw description 时，系统不必重复依赖 receipt、invoice、联网搜索或人工确认来重新证明主体是谁。

  该 match 只提供 identity resolution authority，不提供会计分类、Case precedent、Rule authority 或自动落账权限。

- 为什么(锚定核心产品目标的哪条):
  支持记忆复用和自动化率提升。Alias Log 把过去已确认的 surface text -> stable entity 关系变成可复用身份入口，使重复交易可以更快进入后续判断，而不是每次从原始 evidence 重新识别主体。

- 拟改:
  `BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:读取者` →
  `Entity Resolution Node 是 Alias Log 的唯一已冻结读取者。当当前 raw transaction description 完全命中 Alias Log 中已确认 Alias 时，ER 可以直接复用该 Alias 指向的 stable entity，并输出 stable identity。该输出只代表 identity reuse，不代表 classification、Rule、Case precedent 或 automation permission。`

  `BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:Source of Truth / Authority` →
  `Alias lookup result 的 authority 限于 ER identity resolution：exact raw description match 可以支撑 stable entity identity reuse。Alias Log 查询路径不负责每次重新校验 entity merge / split / correction 后的有效性；这些 durable memory 的同步更新属于治理 / correction / finalization 层责任。`

### Matching 边界暂按 Exact Match 收口

- 结论:
  Alias Log L2 暂时只确认 exact raw description match 可以满足直接复用 stable entity 的要求。受控 normalization / equivalence、semantic similarity、vector retrieval 或其他非 100% exact matching 是否足以支持 stable identity，暂不在本轮 L2 冻结。

  需要保留的标注是：Alias Log 的目标，是当一个 description 成为 Alias 后，系统未来再次遇到该 description 时，仍能识别为最初那个 Alias，并指向同一个 stable entity。是否必须 100% exact match，取决于真实银行 description 中噪音、前缀、乱码和变体对匹配的影响；这一点需要在后续技术 / matching policy 讨论中重点商议。

- 为什么(锚定核心产品目标的哪条):
  支持审计性、accountant control 和自动化率。先冻结 exact match，能给 ER 一个清晰、可审计、低歧义的 identity reuse 路径；同时不提前把向量库、normalization 或 fuzzy match 的技术判断写成 durable authority。

- 拟改:
  `BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:冲突处理` →
  `本轮只冻结 exact raw description match 的 identity reuse authority。非 exact match，包括受控 normalization / equivalence、semantic similarity、vector retrieval 或其他相似匹配，暂不直接提供 stable identity authority；其技术可行性和 authority 边界留后续 matching policy / data contract 阶段讨论。`

- 排除的替代 + 理由:
  排除“现在就允许 vector / semantic similarity 直接支撑 stable identity”。理由是匹配质量依赖技术实现和真实银行文本噪音，过早冻结会把 retrieval hint 误提升为 identity authority。

- 浮现的新 open boundary:
  normalization / equivalence 的最小规则、向量检索是否只作为 hint、银行前缀 / 乱码处理策略，留后续技术 / matching policy 讨论。

### 不为低概率 Description 多 Entity 竞争设计 Alias 状态机

- 结论:
  本轮不为“同一个 raw description 指向多个 stable entity”的低概率 edge case 设计专门 Alias 状态、不可用标签或冲突状态机，也不让 Alias Log 在内部选择 winner。

  原因是 raw description 首次进入 Alias Log 的前置条件已经要求交易指向明确 stable entity；如果首次无法确认唯一主体，应停留在 unknown / pending / accountant clarification，而不是写入 Alias Log。对于后期 pending 由 accountant 确认的情况，也由人工确认 stable entity 后才具备写入资格。

  如果未来查询时确实匹配到多个 stable entity，Alias Log 将该多匹配结果交给 Entity Resolution。ER 负责继续结合当前 evidence 判断能否确定一个 stable entity；如果不能确定，则输出 unknown，并在 reason 中说明存在多个 entity 且无法判定哪一个最准确。后续由 Case Judgment / Coordinator 处理。

- 为什么(锚定核心产品目标的哪条):
  支持系统可理解性和 accountant control。Alias Log 不应为了低概率异常提前承担复杂冲突治理；早期 L2 应保持它作为 ER identity reuse memory 的单一职责。

- 拟改:
  `BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:Lifecycle / States` →
  `Alias Log 不设计“不可用于指代 stable entity”的专门标签，也不为同一 raw description 多 entity 竞争设计独立状态机。进入 Alias Log 的前置条件是交易已确认指向明确 stable entity；无法确认唯一主体的交易不得写入 Alias Log authority。如果查询时出现多个 matched stable entity，Alias Log 不选择 winner，而是把多匹配结果交给 Entity Resolution 判断。`

  `BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:读取者` →
  `当 Alias lookup 返回多个 stable entity 时，ER 可以继续结合当前 evidence 判断是否能确定唯一 stable entity；若不能，则输出 unknown + reason，由后续 Case Judgment / Coordinator 处理。`

- 排除的替代 + 理由:
  排除“给误导性 description 增加不可用标签 / 多 entity 竞争状态”。理由是该设计会显著增加 Alias Log 复杂度，并把上游 Entity Resolution / accountant confirmation 应阻断的问题后移到 Alias Log 内部。

- 浮现的新 open boundary:
  如果未来实际数据中出现 historical Alias conflict，具体 correction / governance 处理路径不在 Alias Log 当前 L2 中设计，归治理 / correction / finalization 层。

### Merge / Split 后的 Alias 归属语义

- 结论:
  Entity merge 后，原 entity 的 Alias 应作为该 entity 的附属 identity memory 跟随 merge 结果，指向新的 stable entity。

  Entity split 后，原有 Alias 不应自动归属任一拆分后的 stable entity。应由治理 / correction 环节按新的 entity 边界重新判断已有 Alias 的归属，再决定哪些 Alias 指向新的 Entity 1，哪些指向新的 Entity 2，哪些暂不恢复为可用 Alias。

  Alias Log 查询路径不额外承担 merge / split 后的逐次复查责任。治理层在处理 Entity merge / split 时，应同步考虑并更新 Alias Log。

- 为什么(锚定核心产品目标的哪条):
  支持审计性、accountant control 和纠错学习。merge 语义通常表示旧 entity 的职责被新 stable entity 承接，Alias 跟随可以保留身份复用能力；split 会改变 entity 边界，自动继承 Alias 会污染身份 authority，因此必须重新判断。

- 拟改:
  `BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:Mutation Path` →
  `Entity merge 后，Alias relationship 语义上跟随 merge 结果指向新的 stable entity。Entity split 后，原 Alias 不自动归属任何一方，必须由治理 / correction 环节按新的 entity 边界重新判断归属。治理层在处理 merge / split 时应同步考虑并更新 Alias Log；Alias Log 查询路径不负责每次重新复查 merge / split 后的有效性。具体执行者、审批路径和 audit trace 属于后续治理 / finalization 机制，不在 Alias Log L2 冻结。`

- 浮现的新 open boundary:
  split 后逐条 Alias 重新归属的 review path、approval authority、batch 操作方式和 audit trace 属于 Governance / correction / finalization 机制。

### Alias Log 的唯一已冻结服务对象是 Entity Resolution

- 结论:
  Alias Log 只直接服务 Entity Resolution。它的唯一职责是帮助 ER 更快判断当前交易的 stable entity identity。

  Rule Match、Case Log、Case Judgment 或其他下游节点不应直接读取 Alias Log 作为规则、案例、分类或自动化依据。它们只能通过 ER 输出的 stable entity 间接受益。

- 为什么(锚定核心产品目标的哪条):
  支持契约面最小化、审计性和系统可控性。Alias Log 一旦成为多个下游节点的直接查询对象，就会从身份复用记忆变成横向共享索引，模糊 Rule、Case、Entity 和 Alias 的 authority 边界。

- 拟改:
  `BK_Copilot/memory_layers/alias_log/02_authority_lifecycle_and_boundaries.md:读取者` →
  `Entity Resolution Node 是 Alias Log 唯一已冻结 reader。其他节点不得直接依赖 Alias Log 作为 rule、case、classification 或 automation basis；如需受益，应通过 ER 输出的 stable entity 再读取各自有 authority 的 memory。`

  `BK_Copilot/memory_layers/alias_log/01_memory_intent.md:绝不保存什么` →
  `Alias Log 不提供面向 Rule Match、Case Judgment、Case Log 或其他下游节点的直接业务检索能力。`

## 分类备案

- `alias_record` exact field schema → L3(留联合 L3)
- Alias Log 的实际 writer、approval path、finalization mechanism、写入顺序 → L4 / seam-park(推给记录层)
- Alias Log 的技术形态、索引、向量库或存储实现 → L4 / seam-park(推给记录层)
- Entity Log 与 Alias Log 的物理存储 / projection 形态 → L4 / seam-park(推给记录层)
- normalization / equivalence exact matching rule、银行前缀 / 乱码处理、vector retrieval 是否只作为 hint → L3(留联合 L3)
- historical Alias conflict 的 correction / governance path → L4 / seam-park(推给记录层)
- Entity split 后逐条 Alias 重新归属的执行机制、审批路径、batch 操作和 audit trace → L4 / seam-park(推给记录层)
- Entity merge / split 时治理层如何同步更新 Alias Log → L4 / seam-park(推给记录层)
- 某类交易是否应跳过 Entity Resolution 并由 Profile / routing / 其他节点直接处理 → L2·外阻(依赖 Profile / workflow routing,挂起)

## 自检与判定

- 按模板 L2 清单自检: 通过;未过项 = 第21条不能进 M3，因为 alias_record schema、非 exact matching 技术策略、writer / finalization mechanism、治理层同步更新机制仍未冻结。
- 本对象 L2 可否判定完成: 是。Alias Log 的存在理由、写入资格、exact match authority、多匹配边界、merge / split 语义和唯一 reader 已收口；剩余问题均归 L3、L4 / seam-park 或治理层机制。
