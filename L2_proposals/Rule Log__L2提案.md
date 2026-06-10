# Rule Log L2 提案

## 依赖

- Entity Resolution 已锁定：RuleMatch / RuleLog 的身份基础只能是 stable entity；unknown / ambiguous / unresolved identity 不构成 rule basis。
- Entity Log 已锁定：Entity Log 保存 stable entity identity、entity_status 与 automation_policy / control state；不保存 rule condition、accounting treatment、rule lifecycle 或 active rule payload。
- Case Log 已锁定：Case Log 可以为 rule promotion / review 提供历史案例依据，但 repeated outcome / case-derived pattern 不能直接创建、升级、修改、删除或降级 active rule。
- Rule Match 已锁定：RuleMatchNode 是 deterministic rule application node，只读取 RuleLog 中的可执行 approved rule，不创建 / 修改 / 删除 rule，不读取 CaseLog 重算资格，不在 runtime 做会计判断。
- Rule Match 已锁定：RuleLog 按 `entity_id` 索引；EntityLog 不存 rule_id 成员列表；Rule payload 至少包含 applicability、approved accounting treatment、scope 类型、来源 / 批准 provenance。
- Rule Match 已锁定：Rule 资格内容判据为“全函数”：scope 可客观判定 + 历史 treatment 唯一无分叉 + 输出 judgment-free 完整 + 事前已批准；次数 / 跨月 / 频率只是证据强度，不是资格门槛。

---

## L2 决策

### 决策点 1：Rule Log 是 RuleMatch 自动化执行所需的 executable rule authority

- 结论:
  Rule Log 必须存在，但它的存在理由非常窄：保存 RuleMatch 可以执行的 approved deterministic rules。它回答的问题是：在某个 stable entity 下，当前有哪些已经批准、条件可客观判定、输出完整且可被 RuleMatch 自动执行的 rule。

  Rule Log 不为了系统结构完整而存在，也不为了替 Transaction Log 做历史解释而存在。它唯一稳定目的，是让系统在满足上游身份与自动化控制条件时，可以不重新判断会计处理，直接复用已批准 rule 来提升自动化。

  因此 Rule Log 不能合并进 Entity Log、Case Log，也不能 inline 到 RuleMatch：

  - Entity Log 是 identity + automation control state 的 authority，不保存 rule condition 或 accounting treatment。
  - Case Log 是 completed-case precedent / promotion evidence layer，不是 deterministic rule authority。
  - RuleMatch 是 runtime reader / executor，不拥有 durable rule authority、lifecycle 或 mutation 权限。

- 为什么(锚定核心产品目标的哪条):
  锚定自动化率与 accountant control。Rule Log 把“可执行规则权威”从身份、案例证据和运行节点中分离出来，使自动化只依赖已批准的 deterministic rule，而不是运行时模型判断、案例趋势或 EntityLog 中的混合字段。

- 拟改:
  `BK_Copilot/memory_layers/rule_log/00_index.md:一句话定义` →
  `Rule Log 是以 entity_id 为主索引的 executable rule authority layer：它保存 RuleMatch 可以执行的 approved deterministic rules，包括 rule applicability、approved accounting treatment、scope 类型、当前执行状态语义和来源 / 批准 provenance。Rule Log 不保存 rule candidate、case evidence、transaction audit trail、EntityLog automation_policy 或非可执行会计建议。`

  `BK_Copilot/memory_layers/rule_log/01_memory_intent.md:为什么这层 memory 必须存在` →
  `Rule Log 必须存在，是因为系统需要一个 durable source of truth 来保存已经批准、可被 RuleMatch 确定性执行的 rule。runtime handoff 只表达当前交易上下文；Entity Log 只表达 identity 和 automation control；Case Log 只表达案例先例和 promotion evidence；RuleMatchNode 只执行规则，不拥有规则权威。`

- 排除的替代 + 理由:
  排除“把 rule payload 放入 Entity Log”。理由是这会把 identity authority、automation control 和 accounting rule authority 混在同一层，扩大 Entity Log 职责。

  排除“把 repeated case pattern 当作 Rule Log”。理由是 Case Log 的案例趋势只能提供依据，不能替代事前批准的 deterministic mapping。

  排除“RuleMatch 内联保存规则”。理由是 RuleMatch 是 runtime-only reader；让运行节点持有 durable rule authority 会混淆执行和记忆。

### 决策点 2：Rule Log 只保存可执行 rule，不保存 candidate / guideline / note

- 结论:
  Rule Log 只保存 executable rule。这里的 executable rule 指：已经批准、scope 可客观判定、输出 treatment 完整、当前允许作为 RuleMatch 执行 authority 的 deterministic rule。

  Rule Log 保存的信息类型包括：

  - rule identity / ref 语义（exact 字段名留 L3）。
  - `entity_id` 主索引。
  - scope 类型：entity-level rule 或 pattern-level rule。
  - applicability：stable entity 身份 + pattern-level 客观条件；entity-level rule 是条件集为空时的特殊形态。
  - approved accounting treatment：RuleMatch 命中后交给 JE Generator 的 judgment-free 完整处理。
  - current execution state 语义：该 rule 当前是否可被 RuleMatch 读取并执行；字段名 / enum 不在本轮冻结。
  - source / approval provenance refs：说明 rule authority 从何而来、由谁批准或经何种治理结果生效。

  Rule Log 绝不保存：

  - rule candidate queue。
  - promotion request 工作队列。
  - review-only guideline / 人工提醒规则 / accountant note。
  - Case Log 逐笔 evidence 正文或聚合趋势。
  - Transaction Log 审计轨迹或完整交易历史。
  - Entity Log 的 automation_policy 本体。
  - raw evidence blob。
  - LLM reasoning、相似描述、自然语言模糊规则。
  - match_count、health score、staleness score 等运行统计真值。

  source / approval provenance refs 是 rule 生效资格的 trace，不是交易审计记录，也不替代 Governance Log / Transaction Log / Case Log 的正文。

- 为什么(锚定核心产品目标的哪条):
  锚定自动化率与契约面最小化。Rule Log 的价值来自“可执行”，不是“相关信息很多”。只保存可执行 rule，可以避免它膨胀成会计知识库、候选队列、统计缓存或审计副本。

- 拟改:
  `BK_Copilot/memory_layers/rule_log/01_memory_intent.md:保存什么` →
  `Rule Log 只保存 approved executable rules。每条 rule 至少表达：rule ref、entity_id、scope 类型、applicability、approved accounting treatment、当前执行状态语义，以及 source / approval provenance refs。字段名、enum、condition schema 和 treatment schema 留 M3。`

  `BK_Copilot/memory_layers/rule_log/01_memory_intent.md:绝不保存什么` →
  `Rule Log 不保存 rule candidate、promotion queue、review-only guideline、accountant note、Case Log evidence 正文、Transaction Log 审计轨迹、EntityLog automation_policy、raw evidence blob、LLM reasoning、相似描述或 match_count / health / staleness 等运行统计真值。`

- 排除的替代 + 理由:
  排除“Rule Log 同时保存候选 rule”。理由是 candidate 还不是 executable authority；把它放入 Rule Log 会迫使 reader 区分未批准内容与可执行内容，扩大误用风险。

  排除“Rule Log 保存人工提醒规则 / 会计师备注”。理由是这些内容可能有业务价值，但不是 RuleMatch 可以确定性执行的 rule；其中 entity / 类别级通则批注 / 快捷经验归 entity_level Knowledge Summary，其他治理或 review-only 事项走 Governance / review context。

### 决策点 3：Rule 资格内容判据最终归 Rule Log authority 语义

- 结论:
  Rule 资格的内容判据最终归 Rule Log 语义，而不是 RuleMatch runtime 语义。RuleMatch 可以引用和消费该判据，但不在运行时重新证明资格。

  一段内容能进入 Rule Log 成为 executable rule，须满足已在 RuleMatch L2 锁定的“全函数”判据：

  1. Scope 可客观判定：仅凭当前交易客观事实和 stable entity 身份即可确定当前交易是否属于该 scope。
  2. 历史 treatment 唯一无分叉：在该 scope 下有效正面先例收敛到唯一 approved accounting treatment；若有正当分叉，必须收窄 scope 或不进入 Rule Log。
  3. 输出 judgment-free 完整：rule 命中后输出的 treatment 必须足够 JE Generator 结合当前交易事实直接执行，不再需要运行时会计判断。
  4. 事前已批准：该映射必须被 accountant / governance 批准后，才可成为 Rule Log authority。

  次数、跨月、频率不是资格门槛，只是支持 promotion 判断的证据强度。CaseLog evidence 可以支撑系统发起 promotion request，但 repeated cases / CaseLog 趋势本身不能直接进入 Rule Log 成为 active rule。

- 为什么(锚定核心产品目标的哪条):
  锚定 accountant control 和自动化安全。Rule Log 的 authority 必须来自事前批准的确定性映射，而不是运行时统计、LLM 判断或案例趋势；否则 RuleMatch 会退化成未批准的会计判断。

- 拟改:
  `BK_Copilot/memory_layers/rule_log/02_authority_lifecycle_and_boundaries.md:Source of Truth / Authority` →
  `Rule Log 是 executable rule authority。只有满足 scope 可客观判定、scope 内 treatment 唯一无分叉、输出 judgment-free 完整、且事前已批准的内容，才能成为 Rule Log 中可执行 rule。RuleMatch runtime 不重新证明这些资格，只读取当前可执行 rule 并做确定性匹配。`

- 边界说明:
  本决策定义的是“什么内容够格成为 Rule Log authority”。候选如何生成、谁审批、何时写入、如何通知 accountant，属于 Governance / review / mutation path，不在 Rule Log L2 冻结。

### 决策点 4：Rule promotion candidate 不归 Rule Log；system-proposed promotion 必须有 CaseLogEvidence

- 结论:
  Rule Log 不管理 rule candidate queue，也不负责提出哪些 entity 或 entity scope 应升级为 rule。候选可以由 CaseLog evidence、lint、review 或其他治理流程生成，但候选本身不进入 Rule Log。

  需要区分两种创建来源：

  - 系统自动提出 promotion request：在向 accountant / governance 发送升级请求之前，CaseLogEvidence 必须作为必备 promotion 依据之一，用于说明该 entity 或 entity scope 下存在可复核的历史处理一致性。
  - Accountant 主动直接创建 rule：不要求先有 CaseLog 积累；只要 accountant 明确判断该 rule 可执行，且 rule 内容满足可客观判定和 treatment 完整性要求，即可经相应写入机制进入 Rule Log。

  因此 CaseLog evidence 是“系统主动建议升级”的必要依据，不是“所有 rule 进入 Rule Log”的必要来源。

- 为什么(锚定核心产品目标的哪条):
  锚定 accountant control。系统提出自动化升级时必须有可复核案例依据，避免凭模型感觉或相似描述诱导自动化；但 accountant 可以直接基于专业判断创建规则，避免把积累次数误当成 rule authority 的本质门槛。

- 拟改:
  `BK_Copilot/memory_layers/rule_log/02_authority_lifecycle_and_boundaries.md:Candidate 边界` →
  `Rule candidate / promotion queue 不属于 Rule Log。系统发起 rule promotion request 前，必须有 CaseLogEvidence 作为 promotion 依据之一；accountant 主动直接创建 rule 时，不要求 CaseLog 积累，但 rule 内容仍须满足 Rule Log 的 executable rule authority 判据。Candidate 进入 Rule Log 的审批 workflow 与写入机制留 Governance / L4。`

- 排除的替代 + 理由:
  排除“CaseLog repeated outcome 自动写入 Rule Log”。理由是 repeated outcome 只是 promotion evidence，不是事前批准。

  排除“所有 rule 都必须先有 N 条 CaseLog 记录”。理由是 accountant direct creation 反证次数不是资格本质；资格本质是可判定、唯一、完整和批准。

### 决策点 5：Rule Log 按 entity_id 主索引；rule 条件挂在 rule 上

- 结论:
  Rule Log 按 `entity_id` 主索引。RuleMatch 凭 handoff 中的 `entity_id` 查询该 stable entity 下当前可执行 rule 集合，再用当前交易客观事实匹配每条 rule 的 applicability。

  EntityLog 不保存 rule_id 成员列表。“该 entity 名下有哪些 rule”的成员关系单一归 Rule Log；EntityLog 只保存该 entity 的 identity / lifecycle / automation control state。

  Rule scope 只有两类：

  - Entity-level rule：stable entity 身份本身足以唯一决定 approved accounting treatment；可视为条件集为空的特殊情形。
  - Pattern-level rule：同一 stable entity 下需要额外客观条件才能唯一决定 approved accounting treatment；条件挂在 rule 的 applicability 上。

  pattern-level 条件必须是当前交易中可客观判定、可复核的事实条件。condition enum、schema、匹配算法和存储索引均不在 L2 冻结。

- 为什么(锚定核心产品目标的哪条):
  锚定单一 source of truth 和自动化安全。entity_id 单坐标让 RuleMatch 查询路径稳定；把条件挂在 RuleLog rule body 上，避免 EntityLog 成为 rule membership 的第二真值，也避免上游预判具体 rule_id。

- 拟改:
  `BK_Copilot/memory_layers/rule_log/01_memory_intent.md:已知约束` →
  `Rule Log 以 entity_id 为主索引。每条 rule 自带 scope 类型和 applicability：entity-level rule 是条件集为空的特殊情形；pattern-level rule 在同一 stable entity 下记录额外客观条件。EntityLog 不保存 rule_id 成员列表；RuleMatch 凭 entity_id 查询 Rule Log 当前可执行 rule 集合。`

- 排除的替代 + 理由:
  排除“EntityLog 保存该 entity 的 rule_id 列表”。理由是 rule membership authority 应单一归 Rule Log；EntityLog 存列表会制造双真值和同步漂移。

  排除“RuleLog 以 description / alias / category 为主索引”。理由是 RuleMatch 的身份基础已经由 ER 输出 stable entity，Alias 只服务 ER，不是 rule scope 的主坐标。

### 决策点 6：EntityLog automation_policy 是 RuleMatch 的上级放行控制

- 结论:
  Rule Log 中存在当前可执行 rule，是进入 RuleMatch 的必要条件，但不是充分条件。只有 Entity Log 中该 entity 的 automation_policy / control state 明确表示允许 rule-based automation / 允许 RuleMatch 时，当前交易信息才会进入 RuleMatch。

  这里存在上下级关系：

  - EntityLog automation_policy：决定这个 stable entity 当前是否允许走 RuleMatch 自动化路径。
  - RuleLog executable rule：在已经被允许走 RuleMatch 的前提下，提供具体可执行的 approved treatment。

  因此，即使 Rule Log 中存在可执行 rule，只要 EntityLog automation_policy 未明确放行、处于 review-only / blocked / risk-hold / unknown 等不放行语义，当前交易也不得进入 RuleMatch。exact automation_policy 取值名和枚举留 L3 / EntityLog + RuleLog + Governance 联合设计。

- 为什么(锚定核心产品目标的哪条):
  锚定 accountant control。会计师必须有 entity-level 控制旋钮来决定某个 stable entity 是否允许自动化；Rule Log 不能绕过 EntityLog 的控制状态自行触发 RuleMatch。

- 拟改:
  `BK_Copilot/memory_layers/rule_log/02_authority_lifecycle_and_boundaries.md:与 Entity Log 的边界` →
  `EntityLog automation_policy 是 RuleMatch 的上级放行控制。RuleLog 中存在 executable rule 只是可执行内容条件；只有 EntityLog 当前 control state 明确允许 rule-based automation / RuleMatch 时，上游才可把交易交给 RuleMatch。automation_policy exact enum 留联合 L3。`

- 排除的替代 + 理由:
  排除“RuleLog active rule 本身就足以触发 RuleMatch”。理由是这会绕过 EntityLog 中 accountant 可控的 automation boundary。

### 决策点 7：RuleMatch 只能读取执行面；Governance / Review 才能读取管理面

- 结论:
  Rule Log 对外读取面需要区分执行面与管理面。

  RuleMatch 的读取面只能是：给定 `entity_id` 后，读取当前可被 RuleMatch 执行的 rule 集合及其 applicability / approved treatment。RuleMatch 不应读取 paused、retired、replaced、deleted、candidate 或历史管理信息后自行判断哪些还能执行。

  Governance / Review / maintenance 可以为了管理目的读取更完整的 rule 状态、provenance refs、暂停 / 废除 / 替代语义、冲突上下文或候选关联信息。但这些管理信息不能被 RuleMatch 当作 runtime 选择依据。

- 为什么(锚定核心产品目标的哪条):
  锚定契约面最小化和自动化安全。RuleMatch 是执行器，不是规则管理员；让它只看到执行面，可以避免 runtime 自行解释 lifecycle、历史版本或治理备注。

- 拟改:
  `BK_Copilot/memory_layers/rule_log/02_authority_lifecycle_and_boundaries.md:读取者` →
  `RuleMatchNode 只读取 Rule Log 的执行面：按 entity_id 查询当前可执行 rule 集合、applicability 和 approved treatment。Governance / Review / maintenance 可以读取管理面用于审批、暂停、废除、恢复或修复，但管理面信息不构成 RuleMatch runtime 选择依据。`

- 浮现的新 open boundary:
  RuleLog reader / assembler 的具体调用顺序、执行面 projection、查询缓存和权限边界属于 L4 / seam。

### 决策点 8：同一 entity 下可执行 rule 集合的互斥性归 RuleLog / Governance 生效前不变量

- 结论:
  同一 stable entity 下可以存在多条可执行 rule，但当前执行面必须满足互斥性：任一当前交易在同一 entity 下最多命中一条 executable rule。

  该互斥性是 RuleLog / Governance 在 rule 写入或生效前必须保证的 authority invariant；RuleMatch runtime 假定执行面已经满足该不变量。若运行时出现同一交易命中多条 rule，RuleMatch 只能 fail-closed blocked / review，不设 priority，不用 LLM / confidence / fallback 选择 winner。

  同一 entity 下的集合结构延续 RuleMatch L2：

  - entity-level rule 与区分性 pattern-level rule 互斥。
  - “默认 + 例外”不建模为 entity-level + exception，而建模为多条互斥 pattern-level rule，可包含 catch-all pattern。
  - overlap validation 算法、catch-all 语法糖和条件可判定性校验留 L3 / L4。

- 为什么(锚定核心产品目标的哪条):
  锚定确定性自动化。RuleMatch 不能在多个可执行规则中做会计判断；互斥性必须在规则成为执行面 authority 前解决。

- 拟改:
  `BK_Copilot/memory_layers/rule_log/02_authority_lifecycle_and_boundaries.md:Source of Truth / Authority` →
  `Rule Log 的执行面必须保证同一 entity 下的 executable rules 两两互不重叠。该互斥性由 RuleLog / Governance 在 rule 写入或生效前保证；RuleMatch runtime 不消解 overlap。`

- 浮现的新 open boundary:
  overlap validation 的算法、condition satisfiability 检查、catch-all pattern 表达方式和冲突 repair path 属于 L3 / L4 / Governance。

### 决策点 9：Rule lifecycle 只收执行状态语义，不冻结字段名 / enum

- 结论:
  Rule Log 需要表达 rule 当前是否能进入 RuleMatch 执行面，但本轮不冻结字段名、enum 或跨 log lifecycle 命名。原因是系统中多个 log 都会涉及暂停、废除、替代、合并、归档等状态，命名必须后续统一。

  L2 只收以下功能语义：

  - 可执行：该 rule 当前可被 RuleMatch 读取为 executable authority。
  - 暂不可执行 / blocked：该 rule 存在，但当前不得被 RuleMatch 执行。
  - 不再可执行：该 rule 已被废除、删除、替代、合并影响或治理裁定退出执行面。

  具体是原地覆盖、版本化、替代链、物理删除还是投影生成，不在 Rule Log L2 定义。Rule Log L2 只要求：执行面能确定地告诉 RuleMatch 当前哪些 rule 可执行；非执行状态不得泄漏成 runtime 可选规则。

- 为什么(锚定核心产品目标的哪条):
  锚定自动化安全和系统一致性。RuleMatch 需要确定可执行集合，但 lifecycle 命名和 mutation 机制应在全系统统一，而不是由 Rule Log 单独发明。

- 拟改:
  `BK_Copilot/memory_layers/rule_log/02_authority_lifecycle_and_boundaries.md:Lifecycle / States` →
  `Rule Log 本轮只冻结 lifecycle 功能语义，不冻结字段名或 enum：可执行、暂不可执行 / blocked、不再可执行。字段命名、跨 log 生命周期统一、原地覆盖 vs 版本化 / 替代链、物理删除 vs 投影删除，留 Governance / 记录层后续设计。`

- 排除的替代 + 理由:
  排除“本轮为 Rule Log 单独发明 active / suspended / retired / superseded 等最终 enum”。理由是 lifecycle 命名需要跨多个 log 统一，不能一个 log 一套叫法。

### 决策点 10：Rule modification / overwrite / supersession 机制不在 Rule Log L2 冻结

- 结论:
  Rule Log 本轮不定义 rule 修改时必须原地覆盖、必须新建版本、必须 supersession，或必须保留历史版本。该问题属于 Governance / 记录层 mutation design。

  Rule Log L2 只要求两点：

  1. RuleMatch 读取时必须得到当前可执行 rule body。
  2. 非当前执行面内容不得被 RuleMatch 当作可执行 rule。

  Rule Log 不承担替 Transaction Log 做历史解释的职责；历史交易由 Transaction Log / finalization source 记录最终发生过什么。Rule Log 只保证当前自动化所需的 executable rule authority。

- 为什么(锚定核心产品目标的哪条):
  锚定职责收窄。Rule Log 的本质是帮助系统更自动化地处理当前和未来交易，不应在 M1-M2 为审计解释提前冻结版本化方案。

- 拟改:
  `BK_Copilot/memory_layers/rule_log/02_authority_lifecycle_and_boundaries.md:Mutation Path` →
  `Rule 修改、原地覆盖、版本化、替代链、删除和恢复的具体 mutation path 不在 Rule Log L2 冻结。Rule Log 只声明当前执行面必须能确定返回可执行 rule body；mutation approval workflow 与历史解释机制留 Governance / 记录层。`

### 决策点 11：RuleLog 不保存 match_count / health / staleness 统计真值

- 结论:
  Rule Log 不在每次 rule 命中后执行 `match_count + 1`。RuleMatch 已锁定为 runtime-only；每次命中就 mutation RuleLog，会把 RuleLog 从 rule authority 层扩张成运行统计层。

  每笔由 rule 命中的 finalized transaction，应在后续 finalization / record path 中保留 `rule_match` source 语义和 rule ref，使系统未来可以从完成记录派生统计：

  - Transaction Log 是最终审计层，适合保存“这笔交易最终由哪条 rule 处理”的来源语义。
  - Case Log 是学习 / 复用层，可以保存 rule-hit case 的 rule ref，或通过 transaction_log_ref 回查 rule ref，用于 rule review、promotion evidence、automation health 评估。
  - Rule Log 只保存该 rule 当前是否可执行、适用条件、approved treatment 和 approval/source provenance。

  因此 match count / last hit / miss rate / health score / staleness score 都不作为 RuleLog authority。它们可以由 TransactionLog / CaseLog 的逐笔记录派生，或由 L4 maintenance / rollup 机制缓存；缓存不改变 Rule Log authority。

  Staleness / health 评估也不由 Rule Log 自己判断。只有当治理、review、lint 或维护机制基于外部证据把 rule 变成暂不可执行 / 不再可执行时，Rule Log 才反映那个当前执行状态。

- 为什么(锚定核心产品目标的哪条):
  锚定单一职责和可重算性。运行统计属于完成记录的派生视图，不应污染可执行规则权威；从逐笔记录派生统计，也延续 Case Log 已锁的“逐笔真值 + 聚合派生”原则。

- 拟改:
  `BK_Copilot/memory_layers/rule_log/01_memory_intent.md:绝不保存什么` →
  `Rule Log 不保存 match_count、last_hit、miss rate、health score 或 staleness score 作为 authority。rule-hit 来源语义和 rule ref 应随 finalized transaction / case context 保留；统计由 Transaction Log / Case Log 的逐笔记录派生或由维护视图缓存。`

- 浮现的新 open boundary:
  rule-hit ref 在 Transaction Log / Case Log 的 exact 字段形态、派生统计 / rollup / cache 的实现机制、health / staleness 评估路径均留 L3 / L4 / Governance。

### 决策点 12：merge / split 后旧 rule 不自动迁移、不自动继承

- 结论:
  Entity merge / split 改变 stable identity 边界后，旧 rule 如何归属、是否迁移、是否继续有效，必须服从 Governance 裁定。Rule Log 不自行判断，也不自动迁移或继承 rule。

  默认执行语义是：如果 entity 边界被治理重判，而旧 rule 尚未被明确保留、迁移、收窄、废除或恢复执行，则旧 rule 不应自动在新 entity 上继续作为 executable authority。

  merge 后，旧 entity 的 rule 不因 EntityLog supersession 自动变成 surviving entity 的 executable rule。split 后，原 entity 下的旧 rule 也不自动分配给新 stable entities。具体重判、批量修复、恢复执行和 trace 归 Governance / mutation path。

- 为什么(锚定核心产品目标的哪条):
  锚定自动化安全。rule 的 applicability 依赖 stable entity identity boundary；一旦身份边界改变，旧 rule 的 scope 前提可能失效，不能静默继承。

- 拟改:
  `BK_Copilot/memory_layers/rule_log/02_authority_lifecycle_and_boundaries.md:与 Entity Log 的边界` →
  `Entity merge / split 后，Rule Log 服从 Governance 裁定。旧 rule 不自动迁移、不自动继承；在治理明确保留、迁移、收窄、废除或恢复前，不应作为新 entity 边界下的 executable authority。`

- 浮现的新 open boundary:
  merge / split 后 rule 重判、批量 mutation、回滚和治理 trace 属 L4 / Governance。

### 决策点 13：Rule Log 与 JE Generator 的边界

- 结论:
  Rule Log 保存 approved accounting treatment；RuleMatch 命中 rule 后，把该 treatment 结合当前交易客观事实交给 JE Generator。JE Generator 负责生成并校验 JE。

  Rule Log 中的 treatment 必须 judgment-free executable：RuleMatch 命中后，不应还需要 LLM 或 runtime accountant judgment 补全 COA、HST / GST、split、allocation 或其他执行要素。若 treatment 不完整或仍需判断，该内容不能成为 Rule Log executable rule，应走 CaseJudgment / review。

  approved accounting treatment 的 exact shape、COA / tax / split / allocation schema、JE Generator exact contract 和 validation rule 留 L3 / JE Generator 联合设计。

- 为什么(锚定核心产品目标的哪条):
  锚定确定性自动化。Rule Log 不能只保存“倾向”或“建议”；它保存的 treatment 必须足以支撑 JE Generator 直接执行。

- 拟改:
  `BK_Copilot/memory_layers/rule_log/01_memory_intent.md:保存什么` →
  `Rule Log 保存 approved accounting treatment，但不保存 JE 或 JE 生成过程。Treatment 必须 judgment-free executable；exact treatment schema 与 JE Generator contract 留联合 L3。`

  `BK_Copilot/memory_layers/rule_log/02_authority_lifecycle_and_boundaries.md:与 JE Generator 的边界` →
  `Rule Log 是 approved treatment authority；JE Generator 是 JE construction / validation layer。Rule Log 不生成 JE，JE Generator 不判断 rule applicability。`

---

## 分类备案

- Rule record exact 字段名、rule ref / rule_id 形态、scope enum、current execution state 字段名 / enum、source / approval provenance refs 形态 → L3(留联合 L3)
- Entity-level / pattern-level rule 的 exact schema、condition enum、客观条件表达、catch-all pattern 语法糖 → L3(留联合 L3)
- approved accounting treatment 的 exact shape、COA / HST / GST / split / allocation schema、JE Generator contract → L3 / JE Generator(留联合设计)
- automation_policy 中“哪种取值 = 允许 rule-based automation / RuleMatch”的 exact enum → L3(EntityLog / RuleLog / Governance 联合)
- RuleMatch 读取 RuleLog 执行面的 exact query contract、reader / assembler 调用顺序、projection / cache / permission 边界 → L4 / seam-park(推给记录层)
- RuleLog 按 entity_id 索引的存储结构、查询优化、索引维护机制 → L4 / seam-park(推给记录层)
- Rule 集合 overlap-validation、condition satisfiability 检查、amount range validation、冲突 repair path → L4 / seam-park(推给 RuleLog / Governance 执行层)
- Rule candidate queue、system-proposed promotion request、CaseLogEvidence 打包格式、accountant approval workflow → L2·外阻(Governance / review path)
- Accountant direct rule creation 的 exact write mechanism、approval capture、写入执行者 → L4 / Governance seam
- Rule modification / overwrite / versioning / supersession / deletion / restore 的 exact mutation path → L2·外阻 / L4(Governance / 记录层；本轮不冻结)
- 跨 log lifecycle 字段命名统一（暂停、废除、替代、删除、归档等）→ L3 / Governance(全系统命名统一)
- rule-hit source / rule ref 在 Transaction Log / Case Log 中的 exact 字段形态 → L3(联合 Transaction Log / Case Log)
- match_count、last_hit、health / staleness 派生统计、rollup、cache、maintenance job → L4 / seam-park(从 Transaction Log / Case Log 派生，不作为 RuleLog authority)
- merge / split 后 rule 重判、批量 mutation、迁移 / 阻断 / 恢复执行、回滚和治理 trace → L4 / Governance

---

## 自检与判定

- 按 memory layer M1-M2 清单自检: 通过。Rule Log 的存在理由、保存内容、不保存边界、source of truth、reader 面、writer / candidate 边界、entity_id 索引语义、rule 资格内容判据归属、EntityLog automation_policy 上级放行关系、rule 集合互斥不变量、lifecycle 功能语义、match_count / health / staleness 边界、merge / split 后行为、JE Generator 边界均已收口。
- 本对象 L2 可否判定完成: 是。Rule Log M1-M2 / L1-L2 已完成；剩余项均为 L3 schema / enum、L4 storage / write / projection seam，或 Governance / JE Generator / Transaction Log / Case Log 的外部依赖。
- 是否可以进入 M3 data contract: 否。rule schema、condition enum、treatment exact shape、execution state 命名、automation_policy enum、mutation path 和 reader / writer contract 尚未冻结。

## 后续需要对齐

- 回填 `BK_Copilot/memory_layers/rule_log/` 正式 M1-M2 草案。
- 后续对齐 Entity Log 正式草案中“RuleMatch 自行读取 EntityLog automation policy”的旧表述；以当前 RuleMatch / RuleLog L2 的“handoff 投影 + EntityLog 上级放行控制”语义为准。
- 后续讨论 Governance 时，补齐 rule candidate queue、promotion approval workflow、direct accountant rule creation、modification / overwrite / deletion / restore、merge / split 后 rule 重判等机制。
