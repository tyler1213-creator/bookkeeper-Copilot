# Case Log L2 提案

## 依赖

- Entity Resolution 已锁定：entity identity 对外只有 `stable / unknown` 两态；只有 stable entity 才有 durable identity handle，unknown 不进入 durable identity 复用。
- Entity Log 已锁定：stable entity identity、Alias、`entity_status`、`automation_policy` 的 source of truth 是 Entity Log；Case Log 按 `entity_id` 组织案例，但不创建、不修改 entity authority。
- Alias Log 已锁定：交易确认指向 stable entity 后，其 raw transaction surface text 具备 Alias 写入资格；Alias 的 authority 在 Alias Log。Case Log 只能引用 / 快照该 alias 表面，不持有 Alias authority。

---

## L2 决策

### 决策点 1：Case Log 是 entity-indexed reusable-precedent layer，不是 Transaction Log 的 entity 副本

- 结论:
  Case Log 是以 `entity_id` 为主索引的「已完成案例可复用先例层」。它的职责是：让未来判断（Case Judgment 会计分类、Rule promotion 评估、automation / entity risk 识别）能复用某 stable entity 下已完成交易的先例，而不必从零重判。

  它**不是**「某 entity 名下全部交易的完整镜像」。「按 entity 查全部历史交易」属于审计 / 报表需求，应由对 Transaction Log 的 entity 索引查询满足，不由 Case Log 承担完整性。因此 Case Log 被允许是**有损的**：可丢弃过期案例、可用新案例替代旧案例、只保留最新可学习状态——这是学习层相对审计层的根本自由。

- 为什么(锚定核心产品目标的哪条):
  锚定「记忆复用」与「自动化率提升」。把 Case Log 定位为先例层而非审计副本，避免与 Transaction Log 产生双 source of truth，并让它能为提速而聚合、为纠错而替代。

- 拟改:
  `BK_Copilot/memory_layers/case_log/00_index.md:一句话定义` →
  `Case Log 是以 entity_id 为主索引的 entity-indexed reusable-precedent layer（已完成案例可复用先例层）。它从已完成交易中保存可复用先例，服务未来 case-based judgment、rule promotion、automation / entity risk 识别；它不是 Transaction Log 的 entity 视图副本，不承担逐笔审计完整性，按 entity 查全部交易由 Transaction Log 的 entity 索引满足。`

- 排除的替代 + 理由:
  排除「Case Log = 某 entity 全量交易列表 / Transaction Log 的 entity 镜像」。理由：会造成双 source of truth，并把审计完整性的不可删改约束套到学习层，使其失去聚合与替代的自由。

### 决策点 2：写入资格 = 所有 stable-linked 且已 finalized 的交易

- 结论:
  所有「已关联 stable entity 且已完成 / finalized」的交易都具备 Case Log 写入资格。系统**不在写入时预判**某笔是否「有未来学习价值」——因为写入当下无可靠依据做这种价值判断，强行判断等于引入一个无法审计的价值过滤器。

  负边界：
  - unknown entity 的交易，即使被 accountant 完成分类并 finalize，也**不进 Case Log**（无 entity 索引键），不写 Entity Log，且**不得**为其创建以 description / 类别为索引的学习记录。
  - pending / 未 finalized 的交易不进 Case Log。

  「是否所有交易都逐笔独立保存」是**存储/读取形态**问题，与「写入资格」是两件事，见决策点 4。

- 为什么(锚定核心产品目标的哪条):
  锚定「自动化率提升」与「记忆复用」。稳定主体下高频、可复用的交易正是自动化价值所在；不预判价值可避免错杀未来高频先例。

- 拟改:
  `BK_Copilot/memory_layers/case_log/01_memory_intent.md:已知约束` →
  `写入资格：所有已关联 stable entity 且已 finalized 的交易均可写入 Case Log，不在写入时预判学习价值。unknown entity 交易即使完成分类也只 finalize 到 Transaction Log，不进 Case Log、不进 Entity Log，也不得创建 description / 类别索引的学习记录。pending / 未 finalized 交易不写入。`

- 排除的替代 + 理由:
  排除「只有具备复用 / 例外 / 纠正价值的交易才写入」。理由：价值在写入当下不可靠可判，且会让系统在 L2 发明无法审计的价值判断器；复用安全改由 `use_level` 标记承担（决策点 5），而非靠入口过滤。

### 决策点 3：Case Log 与 Transaction Log 的边界——learning view vs audit source of truth

- 结论:
  - Transaction Log 是每笔交易**最终事实 + 完整可追溯 + 修改历史**的 source of truth（审计层）。
  - Case Log 是 entity-indexed 学习视图：它**引用** `transaction_log_ref` / 等价 finalization proof 作为完成证明；它**只保存最新可学习状态**，不保存修改历史，不复制完整 processing path，不替代最终审计记录。
  - 冲突时，Transaction Log / finalization source 对 final outcome 与 audit trail 优先；缺少有效 finalization proof 的 case 不能作为可复用先例。

- 为什么(锚定核心产品目标的哪条):
  锚定「审计性」与「记忆复用」。两层各守一职：审计的完整与不可删改归 Transaction Log，学习的可复用与可替代归 Case Log，避免边界塌陷。

- 拟改:
  `BK_Copilot/memory_layers/case_log/02_authority_lifecycle_and_boundaries.md:与其他 memory/log 的边界（Transaction Log 行）` →
  `Transaction Log 保存 final 交易事实、完整 processing path、修改历史与 audit trail，是审计 source of truth。Case Log 只引用 transaction_log_ref / finalization proof，只保存最新可学习状态，不存修改历史、不存完整 processing path、不替代审计记录。缺少 finalization proof 的片段不能作为可复用先例。`

### 决策点 4：真值单位 = 逐笔案例；模式聚合 = 派生读取视图（归 L4）

- 结论:
  Case Log 的**真值单位（unit of truth）是「逐笔已完成案例先例」**：每条 case 对应一笔完成交易，带 entity 索引、finalization proof、复用条件、复用强度、复用权限。

  「模式聚合」（某 entity 下按方向 / 金额区间 / 分类聚成的分布、次数、一致性、最近出现等）是**对逐笔 case 的纯派生结果**，不作为 Case Log 的存储字段。它在 Case Judgment 读取时由固定化聚合机制产出（详见缺口地图 Case Log 段 L4 条目）。

  关键澄清：不存聚合层 ≠ 让 Case Judgment 裸读 N 条流水。保护读取精度的恰恰是**读取时的聚合**（返回压缩摘要 + 相关例外），这是 L4 retrieval 机制，本轮只标归属。

- 为什么(锚定核心产品目标的哪条):
  锚定「记忆复用」与「有证据支持的建议」。逐笔保真保住每条先例的证据条件与例外；聚合作为派生视图避免双真值不同步，并解决高频 entity 的读取压力。

- 拟改:
  `BK_Copilot/memory_layers/case_log/01_memory_intent.md:保存什么` →
  `Case Log 的真值单位是逐笔已完成案例先例（per-completed-case precedent）。某 entity 下的分布 / 次数 / 一致性等聚合信息全部由逐笔 case 派生，不作为 Case Log 存储内容，由读取层聚合机制在需要时产出。`

- 排除的替代 + 理由:
  排除「以 pattern 为主存储对象、新交易匹配上就 count++」（老系统 Observations 式）。理由：在写入时做破坏性聚合判断，一旦漏判区分性细节即永久丢失，且聚合规则无法重算；改用「逐笔 source + 派生 rollup」可非破坏、可重算，并直接消解人工批注导致的「是否新建模式」两难。

### 决策点 5：`use_level` 复用权限 —— 让「全写」安全的核心字段 + 最小 supersession 复用安全

- 结论:
  每条 case 必须带一个**复用权限标记 `use_level`**，表达「这条案例未来允许被怎么用」：正面先例 / 仅例外上下文 / 反面或冲突（「别这么记」）/ 可判断但不可升级为 rule。`use_level` 同时吞并最小 supersession 的**复用安全**取值：被纠正 / 冲销而失效的旧案例，标为失效（invalid / superseded 类取值），下游不得静默将其当作有效正面先例复用。

  本轮**只收口复用安全**（旧案例失效后不被静默复用）。supersession 的**血缘链接**（A 替代 B、B 被 A 替代的具体 ID 指向）与治理执行机制**延后**，不在本轮设计。

- 为什么(锚定核心产品目标的哪条):
  锚定「accountant correction learning」与「审计性」。逐笔全写若无复用权限标记，例外与纠正会被当正面先例复用；`use_level` 是让「全写」安全的前提，也是纠错学习落地的载体。

- 拟改:
  `BK_Copilot/memory_layers/case_log/02_authority_lifecycle_and_boundaries.md:Lifecycle / States` →
  `每条 case 带复用权限标记 use_level（正面先例 / 仅例外 / 反面或冲突 / 不可升级），并以其失效取值承载最小 supersession 复用安全：被纠正 / 冲销失效的旧案例标为失效，下游不得静默当作有效先例复用。supersession 的血缘 ID 链接与治理执行机制不在本轮冻结。`

- 浮现的新 open boundary:
  supersession 的血缘链接字段与具体重判 / 治理执行机制（延后）。

### 决策点 6：case-derived 信号只提供依据，不直接 mutation

- 结论:
  Case Log 可以为 rule promotion、automation risk review、entity risk update、policy review 提供历史案例**依据**；但它不直接创建 / 升级 / 修改 / 降级 active rule，不修改 Entity Log（`risk_flags` / `automation_policy` / Alias / `entity_status`），不改 Governance Log。候选的产生与审批走治理路径。

  候选信号是**给下游节点的临时 handoff**，不是 durable 学习内容，因此**不作为 Case Log 的存储字段**（不设 `candidate_signal_refs`）。

- 为什么(锚定核心产品目标的哪条):
  锚定「accountant control」与「审计性」。把「提依据」与「做变更」分离，避免学习层越权变成规则 / 治理执行者，也避免把治理工作队列混进学习记忆。

- 拟改:
  `BK_Copilot/memory_layers/case_log/02_authority_lifecycle_and_boundaries.md:Candidate 边界` →
  `Case Log 提供 rule / automation / entity risk / policy review 的历史依据，但不执行任何 mutation；候选生成与审批归治理路径。候选信号是下游临时 handoff，不作为 Case Log 存储字段。`

- 浮现的新 open boundary:
  case-derived candidate 进入治理的 exact contract → 见分类备案 L2·外阻。

### 决策点 7：merge / split 后记录归属由治理裁定，Case Log 服从不裁判

- 结论:
  entity merge / split / archive 后，旧 case 如何在新 entity 边界下重新归属，由治理层判断，Case Log 不自行裁判。但 Case Log 必须**服从治理结果**：已被治理限制 / 替代 / 废弃的旧 case 不得被未来节点静默当作正常先例使用。

- 为什么(锚定核心产品目标的哪条):
  锚定「审计性」与「accountant control」。归属变更属身份治理职责；Case Log 只需保证不复用已失效先例。

- 拟改:
  `BK_Copilot/memory_layers/case_log/02_authority_lifecycle_and_boundaries.md:与 Entity Log 的关系` →
  `merge / split / archive 后旧 case 的归属由治理层裁定，Case Log 不裁判但服从结果；被治理限制 / 替代 / 废弃的旧 case 不得被静默复用。具体重判与执行机制归治理 / finalization 层。`

### 决策点 8：Case Log 保存信息清单（预设字段，未冻结）

- 结论:
  以下为 Case Log 需保存的信息清单。**字段尚未冻结**；此处以字段形式列出，是为了说明「需要保存哪些信息、各自起什么作用」，exact 字段名 / enum / schema / validation 留 L3。核心只保住四类：**快照答案 / 复用条件 / 复用强度 / 复用权限**；凡「能从别处算出的、纯留痕的、别的库当家的」一律不进。

| # | 预设字段 | 保存的信息 / 作用 |
|---|---|---|
| 1 | `case_id` | 案例唯一标识。用于 dedup（防同笔重复写入）与被引用的锚点。 |
| 2 | `entity_id` | 主索引——「这是谁的案例」。Case Log 按 entity 组织的前提。 |
| 3 | `transaction_log_ref` | finalization proof / 审计锚点。证明案例来自已完成交易；并作为通往所有「被刻意不存的留痕信息」（修改历史、完整 JE、原文等）的桥，顺它回 Transaction Log 查。 |
| 4 | `alias` | 本案例交易的 alias 表面（即该交易确认指向 stable entity 后其 raw surface text 对应的 alias）。作用：同一 entity 下不同 alias 往往对应不同交易模式（采购 / 订阅 / 退款），它是「这次长什么样」的模式区分与相似度原料。**只是 alias 表面的引用 / 快照；Alias authority 在 Alias Log，Case Log 不持有、不立。** |
| 5 | `direction` | 资金方向（debit / credit）。改变会计含义，是模式区分的必要维度。 |
| 6 | `amount` | 交易金额（绝对值）。判断是否落在同一模式区间的基础。 |
| 7 | `date` | 交易日期。用于 recency 与读取层聚合的时间维度（period 可由它派生，不单存）。 |
| 8 | `account` | 最终 COA 科目（快照）。被学习的「答案」本身；权威在 Transaction Log。 |
| 9 | `hst_gst_treatment` | 税务处理方式（快照）。答案的一部分；权威在 Transaction Log。 |
| 10 | `evidence_refs` | 证据引用（指向 Evidence Log 的指针，不存原文）。用于回溯与下钻。 |
| 11 | `evidence_condition` | 当时有哪类证据支撑（有小票 / invoice / 支票 / 仅银行流水）。**决定该先例何时可被复用**——学习层区别于傻瓜历史的关键。 |
| 12 | `confirm_by` | 结论的权威来源类型（accountant 确认 / accountant 纠正 / rule 命中 / structural / system 高置信）。**决定复用强度**（继承老系统 confirmed_by）。只存「类型」；具体由哪个人确认属审计留痕，归 Transaction Log。 |
| 13 | `use_level` | 复用权限——这条案例未来允许被怎么用（正面先例 / 仅例外 / 反面或冲突 / 不可升级），并以失效取值承载最小 supersession 复用安全。**让「全写」安全的核心字段。** |
| 14 | `context_note` | 例外说明 + 针对「这一笔」的人工批注（合并字段）。解释为什么这笔不能简单泛化。针对整个 entity / 类别的通则批注不在此（属 Knowledge Summary / 治理候选，圈外）。 |

  已砍除并记录理由：`bank_account_ref`（对学习判断弱相关，已确认砍）、`period`（可由 date 派生）、`je_summary`（可由 account+hst+amount+direction 推出，且属 Transaction Log / JE 节点产物）、`authority_refs`（纯留痕，顺 transaction_log_ref 回查）、`entity_identity_snapshot` 的 alias_status / role / display（身份权威在 Entity Log，快照会过期冗余）、`correction_ref` 与 `supersedes/superseded_by`（血缘留痕，与 supersession 链接一并延后）、`candidate_signal_refs`（临时 handoff，非存储内容）、`pattern_ref` 及整层模式 rollup（派生，归 L4）。

- 为什么(锚定核心产品目标的哪条):
  锚定「有证据支持的建议」「记忆复用」「accountant correction learning」。四类核心信息直接支撑未来判断的可复用、可条件化、可信任与可安全复用。

- 拟改:
  `BK_Copilot/memory_layers/case_log/01_memory_intent.md:保存什么` → 以上 14 项预设字段表（标注未冻结、schema 留 L3）。

---

## 分类备案

- Case Log record 的 exact 字段名 / enum / schema / validation（含上表 14 项的精确形态）→ L3(留联合 L3)
- `use_level` 与 `confirm_by` 的 exact enum 取值 → L3(留联合 L3)
- 共享 accounting outcome（COA / HST / split / allocation）的全局字段结构 → L3(留联合 L3)
- Case Judgment 读取时的 per-entity 固定化聚合（rollup / 轻量展示层 / 检索工具或 subagent）→ L4 / seam-park(推给记录层；已记入缺口地图)
- Case Log 的 exact writer、与 Transaction Log 的 trigger order、多 log 统一写入机制 → L4 / seam-park(推给记录层)
- supersession 的血缘链接字段与 corrected / reversed / split 后的重判执行机制 → L4 / seam-park(推给记录层)
- merge / split / archive 后旧 case 重新归属的治理执行机制 → L4 / seam-park(推给记录层)
- 与 Knowledge Summary 冲突的修复流程 → L4 / seam-park(推给记录层)
- case-derived rule / automation / entity risk candidate 进入治理的 exact governance contract → L2·外阻(依赖 Governance / Rule Log,挂起)
- entity / 类别级「通则批注」的归属（Knowledge Summary / 治理候选）→ L2·外阻(依赖 Knowledge Summary / Governance,挂起)

---

## 自检与判定

- 按模板 M1-M2 清单自检: 通过；未过项 = 不能进 M3，因为 record schema、`use_level` / `confirm_by` enum、共享 accounting outcome 结构、读取层聚合机制、writer / trigger order 均未冻结。
- 本对象 L2 可否判定完成: 是。Case Log 的定位（先例层非审计副本）、写入资格、与 Transaction Log 边界、真值单位（逐笔 + 聚合归 L4）、`use_level` 复用权限与最小 supersession 复用安全、candidate 只提依据、merge/split 服从治理、以及保存信息清单均已收口；剩余问题归 L3、L4 / seam-park 或圈外依赖。
