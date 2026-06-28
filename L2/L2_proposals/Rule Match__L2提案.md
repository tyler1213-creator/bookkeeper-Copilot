> ⚠️ **已过时 / superseded（owner 2026-06-27）** — 本 L2 提案的 L1 / L2 结论已由 `BK_Copilot/workflow_nodes/rule_match_node/` 正式草案取代，仅作历史来源保留。当前权威以 `BK_Copilot/` 正式草案 +（产出后）对应 L3 schema 为准；**勿据本文件回灌已删除的概念 / 字段**。

# Rule Match L2 提案

## 依赖

- ER 已锁定：RuleMatchNode 只接受 ER 输出的 `StableEntity` 身份基础；unknown / ambiguous / unresolved identity 不构成 RuleMatch basis。
- Entity Log 已锁定：Entity Log 保存 stable entity identity、entity_status 与 automation_policy / control state；不保存 rule condition、accounting treatment 或 active rule payload。
- Case Log 已锁定：Case Log 可以为 rule promotion / review 提供历史依据，但 repeated outcome / case-derived pattern 不能直接作为 active rule 执行。

## L2 决策

### RuleMatchNode 的存在理由与核心定位

- 结论:
  RuleMatchNode 保留为独立 workflow node。它不是旧系统中所有交易必经的 description-based rule matching node，而是在 ER 已识别 StableEntity 且该 entity 被允许进入 RuleMatch 路径后，执行已批准规则的 deterministic application node。

  RuleMatchNode 的核心职责是：把已经由 accountant / governance 审批过、可以反复复用的 active rule 应用于当前交易，输出高权威 accounting treatment 给 JE Generator。它不重新判断 identity，不做 case-based judgment，不生成 / 升级 / 修改 / 删除 rule，也不学习新模式。

- 为什么(锚定核心产品目标的哪条):
  支持自动化率、审计性和 accountant control。RuleMatchNode 把“这是谁”(ER) 与“已批准规则如何应用”(RuleMatch) 分开，使高自动化路径仍能保持清楚 authority boundary 和可追溯 rule hit context。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/workflow_nodes/rule_match_node/01_functional_intent.md:为什么这个节点必须存在` →
  `RuleMatchNode 必须存在，是因为系统需要一个独立运行节点来应用已经批准的 deterministic rule。Entity Resolution 只回答当前交易指向哪个 StableEntity；CaseJudgment 处理没有可执行 active rule 的判断；JE Generator 只根据 accounting treatment 构造 JE。RuleMatchNode 位于这些对象之间，负责把 active approved rule 应用于当前交易并输出 accounting treatment。`

- 排除的替代 + 理由:
  排除“由 ER 直接输出会计分类并调用 JE Generator”。理由是这会让 ER 从 identity node 膨胀为 rule application / accounting automation node，混淆 identity authority 与 rule authority。

  排除“删除 RuleMatchNode，把规则应用内联到 JE Generator”。理由是 JE Generator 应是根据 accounting treatment 构造 JE 的纯计算 / 校验层，不应承担 rule applicability 判断或 rule hit trace。

### Trigger / Routing 边界

- 结论:
  RuleMatchNode 不是所有交易必经节点。进入 RuleMatchNode 的 L2 触发语义是：

  1. ER 已输出 StableEntity。
  2. Entity Log 中该 stable entity 的 automation_policy / control state 明确支持 RuleMatch。
  3. RuleLog 对该 stable entity 存在可执行 active / approved rule。

  EntityLog 与 RuleLog 的同步创建 / 更新是 rule governance 的设计 invariant。正常情况下，只有 EntityLog 已记录该 entity 支持 RuleMatch，且 RuleLog 有对应 active rule，交易才会被交给 RuleMatchNode。

  RuleMatchNode 不直接读取 EntityLog 来判断 eligibility；ER handoff 应把 RuleMatch 所需的 StableEntity、eligibility / control context 和当前交易客观事实交给 RuleMatchNode。ER 之后是否存在单独 workflow router、RuleLog reader 或 handoff assembler 属于 L3 / L4 seam；L2 只冻结：RuleMatchNode 自身不重新查询 EntityLog、不自行决定 entity 是否支持 RuleMatch。

- 为什么(锚定核心产品目标的哪条):
  支持契约面最小化、审计性和自动化安全。RuleMatch 的触发必须来自已确认 identity 与已批准自动化控制状态，避免 RuleMatch 对 identity / policy 做二次解释。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/workflow_nodes/rule_match_node/02_logic_and_boundaries.md:触发条件` →
  `RuleMatchNode 只在 ER handoff 提供 StableEntity，且 handoff 明确表明该 entity 的 automation_policy / control state 支持 RuleMatch，并且 RuleLog 存在该 entity 的 active approved rule 时触发。RuleMatchNode 不直接读取 EntityLog 来决定 eligibility。`

  `BK_Copilot/workflow_nodes/rule_match_node/02_logic_and_boundaries.md:上游前置条件` →
  `如果 ER 输出 unknown / ambiguous / unresolved identity，或 handoff 未表明 RuleMatch eligibility，当前交易不得进入 RuleMatchNode；应进入 CaseJudgment 或 pending / review 路径。`

- 排除的替代 + 理由:
  排除“所有交易都经过 RuleMatchNode 再 miss 到 CaseJudgment”。理由是新系统已经采用 Entity-first routing，RuleMatch 只服务已具备 rule automation eligibility 的 stable entity，不应成为空转节点。

- 浮现的新 open boundary:
  ER 自己组装 RuleMatch handoff、还是由 ER 后的 workflow router / RuleLog reader 组装，属于 L3 / L4 seam，不在 RuleMatch L2 冻结。

### RuleLog / EntityLog 分工

- 结论:
  EntityLog 保存 entity 级 RuleMatch eligibility / automation control state；RuleLog 保存可执行 active rule payload。RuleMatchNode 使用 ER handoff 中的 eligibility / control context，并以 RuleLog 中 active / approved rule 为 rule authority。

  Rule payload 不放入 EntityLog。EntityLog 至多保存 eligibility flag / automation_policy，不保存 rule condition、accounting treatment、rule lifecycle 或 active rule body。EntityLog 也不保存 rule_id 成员列表——"该 entity 名下有哪些 rule"的成员关系单一归 RuleLog（按 entity_id 索引）；在 EntityLog 存 rule_id 列表会制造双 source of truth 与漂移（见"RuleMatch 输入 handoff 契约面与 RuleLog 查询定位"）。

- 为什么(锚定核心产品目标的哪条):
  支持审计性和 accountant control。EntityLog 是 identity + control 主档案；RuleLog 是 deterministic rule 的 source of truth。将 rule payload 塞回 EntityLog 会让 EntityLog 同时承担 identity authority 与 accounting rule authority，增加纠错和审计难度。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/workflow_nodes/rule_match_node/02_logic_and_boundaries.md:读取对象` →
  `RuleMatchNode 的 rule authority 来自 RuleLog 中该 stable entity 的 active approved rule。EntityLog 的 automation_policy / RuleMatch eligibility 通过 ER handoff 或等价 workflow handoff 进入 RuleMatchNode；RuleMatchNode 不直接读取 AliasLog、CaseLog 或 TransactionLog 来执行 rule。`

  `BK_Copilot/memory_layers/rule_log/01_memory_intent.md:为什么这层 memory 必须存在` →
  `RuleLog 保存 approved deterministic rules，包括 entity-level rule 与 pattern-level rule 的可执行 rule payload。EntityLog 只保存该 entity 是否支持 RuleMatch / automation control state，不保存 rule condition 或 accounting treatment。`

- 排除的替代 + 理由:
  排除“把 RuleLog 作为 EntityLog 子字段保存全部 rule payload”。理由是 rule condition、accounting treatment、rule lifecycle 与 promotion / modification / deletion authority 不属于 identity 主档案；混入 EntityLog 会制造双重职责。

### RuleMatch 输入 handoff 契约面与 RuleLog 查询定位

- 结论:
  RuleMatchNode 由上游 handoff（ER 或 ER 后 assembler 输出）获得运行所需的全部输入。该 handoff 的契约面在 L2 锁三类语义（exact 字段 schema 留 L3；assembler 谁组装留 L4）：

  - 桶 A — 身份基础（来自 ER 身份判断）：`stable_entity_ref`（= entity_id，必填，durable 身份句柄）；entity lifecycle 必须为 active（merged 由上游按 supersession 跳到 surviving entity 后再给，archived / unknown 不组装此 handoff）。不携带 stable reason / identity provenance / alias payload——RuleMatch 不重判身份，这些是审计 trace，不进匹配。
  - 桶 B — 放行 / 控制基础（投影自 EntityLog `automation_policy`）：`automation_control_state`，必填，表达该 entity 是否放行 rule-based automation 的控制语义；authority 仍在 EntityLog，handoff 只投影当前状态，ER 不拥有不决定它。可选携带会 block rule automation 的身份级 risk / governance 限制指针，使 RuleMatch 能 fail-closed。
  - 桶 C — 客观交易事实（来自 evidence intake / transaction identity，沿流程携带）：`transaction_ref`（= transaction_id）、`direction`、`amount_abs`、`transaction_date` 等必填客观维度，以及 `evidence_condition` / `evidence_refs`、`transaction_type` / `objective_tags`、`currency` 等。raw description / surface text 只随 trace 携带，不作匹配条件（scope key 是 stable entity，不是 description）。

  最终会计分类的来源拆分：最终输出给 JE Generator 的 = 命中 rule 的 approved accounting treatment（"答案"，来自 RuleLog payload）+ 桶 C 的交易事实（金额 / 方向 / 日期 / 币种 / evidence refs 等过账输入）。桶 C 是必要但不充分——分类"答案"不在桶 C 里，必须由命中的 rule 提供；让桶 C 自己产出分类等于让 RuleMatch 做会计判断，已被禁止。

  RuleLog 查询定位：全系统只以 entity_id 作为定位坐标。RuleLog 按 entity_id 索引；RuleMatch 凭 handoff 中的 entity_id 向 RuleLog 取该 entity 的候选 active approved 规则集，再用桶 C 逐条比对 rule 已批准条件，找出至多一条命中。rule payload 不在 handoff 里，来自 RuleLog（reader 是 L4 seam）。不采用"在 EntityLog 存 rule_id 成员列表并经 handoff 传 rule_id"的方案：rule 成员关系的权威单一归 RuleLog，EntityLog 存列表会制造双 source of truth 与漂移；且上游无法预知本笔命中哪条 rule（命中取决于桶 C，到 RuleMatch 才完整），传具体 rule_id 在语义上不成立。

  EntityLog 不为此存 rule_id 列表；"是否值得调 RuleMatch"的便宜卡点已由桶 B 的 automation_policy 承担，至多再加一个可由 RuleLog 派生的布尔 hint（"has active rules"，可派生、不必 durable 存）。

- 为什么(锚定核心产品目标的哪条):
  支持契约面最小化、审计性、单一 source of truth 与自动化安全。三桶契约面让 RuleMatch 只依赖显式、最小的输入（身份 + 放行 + 客观事实），不依赖任何上游内部状态；entity_id 单坐标与"成员关系单一归 RuleLog"延续 EntityLog L2 已锁的 entity_id 全系统索引键设计（Case Log 主索引、Alias Log 指向、Rule 索引键都指向 entity_id），避免反规范化造成的双真值与漂移。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/workflow_nodes/rule_match_node/02_logic_and_boundaries.md:上游前置条件` →
  `RuleMatchNode 从上游 handoff 获得三类输入：身份基础（active stable entity 的 entity_id）、放行 / 控制基础（投影自 EntityLog automation_policy 的 control state）、客观交易事实（transaction_id、direction、amount、date、evidence condition 等）。exact 字段 schema 留 Stage 3；handoff 由谁组装留 Stage 4 / seam。`

  `BK_Copilot/workflow_nodes/rule_match_node/02_logic_and_boundaries.md:读取对象` →
  `RuleMatchNode 凭 handoff 中的 entity_id 向 RuleLog（按 entity_id 索引）取该 entity 的候选 active approved 规则集，用客观交易事实逐条比对，找出至多一条命中。rule payload 来自 RuleLog，不来自 handoff。最终输出给 JE Generator 的分类 = 命中 rule 的 approved accounting treatment + 当前交易客观事实。`

- 排除的替代 + 理由:
  排除"在 EntityLog 存该 entity 的 rule_id 成员列表，由上游把 rule_id 传给 RuleMatch"（方案一）。理由：rule 成员关系的权威单一归 RuleLog，在 EntityLog 再存一份等于制造第二 source of truth，需要跨 log 原子同步、易漂移；该列表不提供任何 authority（仍须回 RuleLog 校验 active / approved 与 payload），是过早反规范化；且上游无法预知本笔命中哪条 rule，传具体 rule_id 语义不成立。

- 浮现的新 open boundary / 对 RuleLog 的前置约束:
  RuleLog 必须支持按 entity_id 索引、并在每条 rule 上承载 applicability（entity_id 及 pattern-level 客观条件）。entity_id 索引的具体存储 / 检索机制留 L4；automation_policy 的取值集合与"哪种取值 = 放行 rule-based automation"的语义留 RuleLog / EntityLog 联合 L3；rule payload（approved accounting treatment 的 judgment-free 完整形态）留 L3 / JE Generator。

### Rule scope：entity-level rule 与 pattern-level rule

- 结论:
  RuleMatch L2 只确认两类 rule scope：

  - Entity-level rule：StableEntity 本身足以稳定指向唯一 accounting treatment。
  - Pattern-level rule：同一 StableEntity 下需要更窄、客观可判定的条件才能唯一决定 accounting treatment。

  如果一个 entity 的历史交易存在多种最终会计分类结果，该 entity 本身不应升级成 entity-level rule。只有当更窄的客观条件可以稳定地区分不同 accounting treatment 时，才考虑 pattern-level rule。

- 为什么(锚定核心产品目标的哪条):
  支持自动化率和审计性。entity-level rule 让真正稳定的高频 entity 自动化；pattern-level rule 保护复杂 entity 不被粗暴自动分类。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/workflow_nodes/rule_match_node/01_functional_intent.md:核心职责` →
  `RuleMatchNode 应用的 rule scope 只有两类：entity-level rule 与 pattern-level rule。entity-level rule 适用于 stable entity 本身足以唯一决定 accounting treatment 的场景；pattern-level rule 适用于同一 stable entity 下必须满足更窄客观条件才能唯一决定 accounting treatment 的场景。`

  补充语义（资格判据归一）：entity-level rule 是下文"Rule 资格的内容判据"在 condition set 为空时的退化情形（仅 stable entity 身份即足以唯一决定 accounting treatment）；pattern-level rule 是需要非空客观条件集的一般情形。两者不是两套独立机制，而是同一资格判据的两种呈现。

- 浮现的新 open boundary:
  entity-level / pattern-level 的 exact schema、condition enum 与 validation rules 留 L3。rule priority 不在 L2 设立（见"Rule hit / miss / invalid handoff 语义"：多命中按 fail-closed 处理，不设 priority）。

### Rule 资格的内容判据（什么内容才够格成为可执行 rule）

- 结论:
  一段内容能否成为可执行的 deterministic rule，本质判据是它必须等价于一个"全函数"：在一个可被当前交易客观事实确定性判定的 scope 内，对该 scope 内任一交易输出唯一确定、可直接执行的 accounting treatment。须同时满足四条：

  1. Scope 可客观判定：scope 成员资格只凭当前交易的客观事实（身份 = stable entity，加 direction、amount range、evidence condition、transaction type 等客观维度）即可当场、确定性判断"这笔是否属于该 scope"，不依赖 LLM 解释、描述相似度或运行时模式发现。
  2. 结果唯一、无历史分叉：在该 scope 下历史会计处理收敛到唯一结果，无实质性分叉；出现第二种正当处理即丧失自动执行资格，必须收窄 scope 或不升级。被 use_level 标为例外 / 反面、或被 supersede 失效的旧 case 不计入"第二种正当处理"——判据看的是有效正面先例是否唯一。
  3. 输出为判断自由的完整处理：rule 命中后输出的 accounting treatment 必须是闭合、judgment-free、JE Generator 仅凭当前交易客观事实 + rule payload 即可直接执行的完整处理（COA / HST / GST / split 等齐全）。若应用该处理仍需运行时会计判断或解释性外部信息，则该内容不够格成为 rule，应走 CaseJudgment。
  4. 事前已批准：该唯一性已被 accountant / governance 认可。"批准必须存在"是资格的一部分；"如何批准"属机制、归外阻。

  次数 / 跨时间 / 频率不是资格门槛，只是支撑"结果唯一"判断的证据强度（防止把偶发当规律）。L2 不冻结"≥N 次 / ≥M 月"这类阈值数字；具体阈值（若需要）属 L3 / 治理风险判断。

  绝不能直接成为可执行 rule、只能进 rule review candidate 的内容：仅有 repeated cases、相似描述、LLM reasoning、CaseLog 聚合趋势、模糊自然语言。active rule 的高权威只来自"事前批准的确定性映射"，不来自运行时统计或模型判断。

- 为什么(锚定核心产品目标的哪条):
  支持自动化率、审计性和 accountant control。deterministic rule 的全部价值是"无需再判断即可执行"，等价于一个定义域可判定、映射唯一、输出可执行的全函数；缺任一支柱，rule 就退化成需要判断的建议，而判断归 CaseJudgment。自动化只有在有已批准记忆和可恢复 audit trail 下才有价值。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/workflow_nodes/rule_match_node/01_functional_intent.md:核心职责` →
  `一段内容够格成为可执行 deterministic rule，须同时满足：(1) scope 可由当前交易客观事实确定性判定；(2) 该 scope 下历史 accounting treatment 唯一、无实质分叉；(3) 输出是 judgment-free、可被 JE Generator 直接执行的完整 treatment；(4) 已被 accountant / governance 事前批准。次数 / 跨月 / 频率只是证据强度，不是资格门槛，不在本层冻结阈值。repeated cases、相似描述、LLM reasoning、CaseLog 聚合趋势、模糊自然语言不能直接成为 rule，只能进 rule review candidate。`

- 分类归属:
  本判据（资格内容判据）属【L2】本轮收口；阈值具体数字、condition / treatment 的 exact schema 属【L3】 / JE Generator；晋升审批 workflow、candidate queue、rule lifecycle / health / 改删降级属【L2·外阻】（RuleLog / Governance）。rule 的前瞻有效性 / staleness 属 rule lifecycle，归【L2·外阻】，不作为 RuleMatch 侧内容判据。

- 边界说明:
  本轮 L2 收口的是 rule validity 的"内容判据"（什么算合格的规则内容）；晋升的"执行机制"（谁审批、队列、写库、改删降级）仍属 RuleLog / Governance 外阻。两者正交：判据是机制的输入约束，机制不改变"什么算合格"。RuleMatchNode 自身不判资格升级、不执行晋升；资格判据是给 RuleLog / Governance 在晋升时用的语义标准。

### 同一 stable entity 下的 rule 集合结构

- 结论:
  同一 stable entity 下：

  1. 允许并存多条 active rule，前提是每条 rule 的适用条件客观、可判定、且两两互不重叠——任一笔交易最多落入一条。该互斥性是写入时不变量（write-time invariant），由治理 / RuleLog 在晋升时保证；RuleMatchNode 运行时假定其成立，不做重叠消解、不裁剪条件、不合并 rule。
  2. entity-level rule 与"区分性"pattern-level rule 在同一 entity 上互斥：若该 entity 已有一条覆盖全部的 entity-level rule（身份即唯一决定处理），则不能再有意在区分不同处理的 pattern-level rule——后者的存在恰证伪前者"无分叉"前提。因此同一 entity 上的 rule 集合，要么 = {单条 entity-level rule}，要么 = {若干条互斥 pattern-level rule}。
  3. "entity-level 默认 + 少量例外"不是 entity-level：它本质上说明该 entity 非单一处理，应建模为多条互斥 pattern-level rule（可含一条 catch-all pattern），而非"entity-level + 例外 pattern"。这样运行时永远最多命中一条。

- 为什么(锚定核心产品目标的哪条):
  支持自动化率、审计性和 accountant control。允许并存支撑真实 entity 的多模式自动化；把互斥性前移到写入时，使 runtime 永远面对"最多一条命中"的干净世界，无需 priority 或语义裁决，避免 RuleMatchNode 越权选 winner、破坏确定性。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/workflow_nodes/rule_match_node/02_logic_and_boundaries.md:决策权限` →
  `同一 stable entity 下可并存多条 active rule，但各 rule 条件必须客观、可判定、两两互不重叠（任一交易最多命中一条）；该互斥性由治理 / RuleLog 在写入时保证，RuleMatchNode 不在运行时消解重叠。entity-level rule 与区分性 pattern-level rule 在同一 entity 上互斥；"默认 + 例外"应建模为多条互斥 pattern-level rule（含 catch-all），不建模为 entity-level + 例外。`

- 浮现的新 open boundary:
  rule 集合 overlap-validation 的算法属 L4 / seam（推 RuleLog / 治理执行层）；catch-all pattern 是否作为 schema 语法糖属 L3。

### Pattern-level rule 的客观条件边界

- 结论:
  Pattern-level rule 的条件必须是当前交易中客观可判定、可审计复核的条件，例如 direction、amount range / 金额稳定性、当前交易已有 evidence condition，或后续确认的其他客观条件。

  L2 不冻结完整 condition schema，但先冻结负边界：RuleMatchNode 不能依赖 LLM 主观解释、AI reasoning、unapproved pattern、CaseLog repeated outcome 或模糊自然语言来判断 pattern-level rule 是否适用。

- 为什么(锚定核心产品目标的哪条):
  支持审计性和 accountant control。RuleMatch 是确定性自动化路径；适用条件必须能被复核，不能变成模型“看起来像”的判断。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/workflow_nodes/rule_match_node/02_logic_and_boundaries.md:决策权限` →
  `RuleMatchNode 只能根据 active rule 中记录的客观条件匹配当前交易。pattern-level rule 条件必须可由当前交易客观事实判定；LLM 不能解释、补全或放宽 rule condition。`

- 分类归属:
  condition 字段、enum、匹配算法和金额区间 exact validation 留 L3 / L4。

### RuleMatchNode 不具备会计判断能力

- 结论:
  RuleMatchNode 本身不做任何会计分类判断。它只是执行流程：当当前交易满足已记录的 active rule 时，将 rule 中已经批准的固定 accounting treatment 输出给 JE Generator。

  LLM 不能在 RuleMatchNode 内判断 COA / HST / GST / split / allocation，不能补全缺失 rule 条件，不能在多个 rule 中选择 winner，也不能把 case history 或 AI reasoning 升格为 rule。

  RuleMatchNode 运行时也不重新推导"结果唯一 / 无分叉"这类资格判据：资格的认定在晋升 / 写入时完成并固化于 RuleLog，运行时只信任 RuleLog 的 active + approved 状态，并把 rule 已批准的客观条件与当前交易事实做确定性比对。运行时重算资格会让输出随 Case Log（有损、可替代的学习层）漂移、破坏确定性，并制造与 RuleLog 竞争的第二 authority。因此 RuleMatchNode 不读 Case Log 来执行 rule；历史分叉证据的消费者是治理 / 晋升路径，不是 RuleMatch runtime。

- 为什么(锚定核心产品目标的哪条):
  支持自动化安全和审计性。RuleMatch 的高权威来自 rule 事前已经被 accountant / governance 审批，而不是来自运行时模型判断。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/workflow_nodes/rule_match_node/02_logic_and_boundaries.md:决策权限` →
  `RuleMatchNode 没有会计分类判断权。它只能执行 active approved rule 中已记录的 accounting treatment。LLM 不参与 rule applicability、accounting treatment 或 conflict winner 的判断。`

### Rule hit / miss / invalid handoff 语义

- 结论:
  Rule hit 表示当前交易满足 active approved rule，RuleMatchNode 输出高权威 accounting treatment 和 rule hit context。

  Rule miss 表示当前交易虽关联支持 RuleMatch 的 StableEntity，但不满足任何适用 active rule；RuleMatchNode 不继续做 case-based judgment，交易应进入 CaseJudgment。

  同一交易在同一 entity 下的命中数量语义：0 条适用 = rule_miss → CaseJudgment；1 条 = rule_hit；≥2 条同时命中 = invalid_handoff / blocked。≥2 条同时命中不是常规 routing 分支，而是"rule 集合互不重叠"这一写入时不变量被破坏的异常信号；正常设计下不应发生。RuleMatchNode 对其按 fail-closed 处理，交治理 / review，不设 priority，不用 LLM / confidence / fallback 偏好选 winner。

  在正常治理 invariant 下，不应出现 EntityLog 标记支持 RuleMatch 但 RuleLog 无 active rule、多条互斥 rule 同时命中、RuleLog 与 eligibility handoff 冲突等状态。L2 不为这些低概率异常设计复杂 conflict resolution；若运行时出现，应视为 invalid authority state / invalid handoff，保守 blocked / review，不允许 RuleMatchNode 猜测或由 LLM 选择。

- 为什么(锚定核心产品目标的哪条):
  支持审计性和 accountant control。RuleMatch 是确定性路径，不能在规则缺失、规则冲突或 authority 不一致时自行补设计。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/workflow_nodes/rule_match_node/02_logic_and_boundaries.md:输出类别` →
  `RuleMatchNode 的稳定输出语义包括 rule_hit、rule_miss、invalid_handoff / blocked。rule_hit 输出 approved accounting treatment 和 rule hit context；rule_miss 交给 CaseJudgment；invalid_handoff / blocked 交给 review / governance，不允许 LLM 或 RuleMatchNode 自行解除。字段名和 enum 留 Stage 3。`

- 排除的替代 + 理由:
  排除“为多 rule 命中设计复杂 priority / conflict resolution”。理由是 RuleMatch eligibility 与 rule creation 本应由治理保证唯一、可执行、无互斥冲突；当前 L2 只需保留 fail-closed 语义，priority / conflict enum 如未来需要再进入 L3。

### 输出给 JE Generator 的边界

- 结论:
  RuleMatchNode 输出 accounting treatment 给 JE Generator；JE Generator 根据该 treatment 和当前交易事实生成并校验 JE。

  RuleMatchNode 输出的 accounting treatment 在分类权威上是高置信度、高权威，因为 rule 已经过 accountant / governance 审批。但 RuleMatchNode 输出不等于 JE 已成功生成 / 校验，不等于 TransactionLog 已 finalized，也不等于 CaseLog 已写入。

- 为什么(锚定核心产品目标的哪条):
  支持审计性和契约面最小化。RuleMatchNode 负责高权威分类来源；JE Generator 负责 JE 构造 / 校验；Transaction finalization 负责最终审计记录。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/workflow_nodes/rule_match_node/02_logic_and_boundaries.md:输出类别` →
  `rule_hit 输出 approved accounting treatment 与 rule hit context，供 JE Generator 构造并校验 JE。该输出是高权威分类来源，但不表示 JE 已生成成功、TransactionLog 已 finalized、CaseLog 已写入或 accountant 对本次交易完成最终审核。`

  `BK_Copilot/workflow_nodes/rule_match_node/02_logic_and_boundaries.md:明确排除范围` →
  `RuleMatchNode 不拥有 JE 构造逻辑，不直接执行 TransactionLog finalization，不写 CaseLog，不创建或修改 RuleLog。`

### Runtime-only 与 finalization 写入边界

- 结论:
  RuleMatchNode 保持 runtime-only。它不直接写 EntityLog、RuleLog、CaseLog、TransactionLog 或 GovernanceLog。

  由 RuleMatchNode 完成分类的交易，在后续 finalization 路径中仍应进入 TransactionLog；只要该交易 stable-linked 且 finalized，也具备 CaseLog 写入资格。TransactionLog / CaseLog 的实际写入者、触发顺序、多 log finalization、以及 Rule 使用次数 / rule hit count / rule health 等维护逻辑，不由 RuleMatchNode 自己处理，应交给专门 finalization、RuleLog 维护或治理层机制。

- 为什么(锚定核心产品目标的哪条):
  支持审计性、记忆复用和契约面最小化。RuleMatchNode 负责执行已批准规则并产生 runtime 结果；持久化和维护逻辑由对应记录层 / 治理层承担，避免运行节点越权 mutation。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/workflow_nodes/rule_match_node/02_logic_and_boundaries.md:写入对象` →
  `RuleMatchNode 不直接执行 durable write。RuleMatch 命中的交易由后续 finalization 路径写入 TransactionLog；若满足 stable-linked finalized transaction 条件，也进入 CaseLog。Rule 使用次数、rule hit count、rule health 和维护统计由 RuleLog 维护 / governance 机制处理，不由 RuleMatchNode mutation。`

  `BK_Copilot/workflow_nodes/rule_match_node/02_logic_and_boundaries.md:明确排除范围` →
  `RuleMatchNode 不写 EntityLog、RuleLog、CaseLog、TransactionLog 或 GovernanceLog；不更新 rule match_count / health；不执行多 log finalization。`

### Rule hit handoff 的最小语义输出

- 结论:
  RuleMatchNode 命中 active approved rule 后，必须输出可供后续 finalization、audit、learning / review 使用的最小 rule-hit handoff。该 handoff 的目的不是让 RuleMatchNode 自己持久化记录，而是让后续 TransactionLog、CaseLog、RuleLog 维护 / Governance review 能追溯“本次自动分类由哪个 entity 下的哪条 rule 产生”。

  L2 只锁最小语义：rule-hit handoff 至少必须表达 StableEntity ref、Rule ref、approved accounting treatment 和 `rule_match` source 语义。字段名、对象形态、refs 结构、是否包含更多 evidence / approval refs 留 L3。

  RuleMatchNode 不输出 rule hit count、node run count、rule health 或统计结果。命中次数可以由 finalized transaction 中的 Rule ref 查询派生，或由后续 RuleLog 维护 / Governance 机制统计；这些统计和维护机制不属于 RuleMatchNode L2。

- 为什么(锚定核心产品目标的哪条):
  支持审计性、记忆复用和 accountant control。没有最小 rule-hit handoff，后续记录层只能看到 accounting treatment，无法追溯 rule authority；但如果把统计、维护或持久化职责放进 RuleMatchNode，又会让运行节点越权。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/workflow_nodes/rule_match_node/02_logic_and_boundaries.md:输出类别` →
  `rule_hit 输出必须包含最小 rule-hit handoff 语义：StableEntity ref、Rule ref、approved accounting treatment 和 rule_match source。该 handoff 只作为 runtime handoff 交给后续 finalization / audit / learning / review，不由 RuleMatchNode 持久化。exact field schema 留 Stage 3。`

  `BK_Copilot/workflow_nodes/rule_match_node/02_logic_and_boundaries.md:Audit / Trace 边界` →
  `RuleMatchNode 的可追溯性来自 rule-hit handoff：后续 TransactionLog / CaseLog / Governance review 应能通过其中的 StableEntity ref 与 Rule ref 还原本次自动分类使用的 rule authority。RuleMatchNode 不输出或维护命中次数、rule health 或统计值。`

### Rule promotion / mutation / governance 边界

- 结论:
  RuleMatchNode 不负责 rule promotion、rule review、candidate queue、rule health、rule modification、downgrade 或 deletion。CaseLog repeated outcome、CaseLog pattern evidence、AI reasoning 或 unapproved pattern 只能进入 rule review / governance path，不能在 RuleMatch runtime 直接执行。

- 为什么(锚定核心产品目标的哪条):
  支持 accountant control 和纠错学习。学习可以产生候选和依据，但 durable rule authority 必须由 RuleLog / Governance 负责，不能由运行时 RuleMatch 扩权。

- 拟改:文件:章节 → [具体文字]
  `BK_Copilot/workflow_nodes/rule_match_node/01_functional_intent.md:明确排除范围` →
  `RuleMatchNode 不创建、升级、修改、删除或降级 active rule，不处理 rule promotion candidate，不读取 CaseLog repeated outcome 来直接执行规则。rule lifecycle 与 governance approval 属于 RuleLog / Governance 设计。`

- 浮现的新 open boundary:
  Rule promotion eligibility、approval workflow、candidate queue、rule health、rule modification / downgrade / deletion 的 exact governance contract 属于 RuleLog / Governance 后续讨论。

### Legacy Constraint Translation（旧系统平移）

仅记录脱离来源、按当前产品目标重新论证后的取舍；旧系统材料只作审计对象，不构成 authority。

可借鉴保留的内核：

| 借鉴内核 | 旧系统来源 | 现在为何仍成立 |
| --- | --- | --- |
| "分类一致 / 出现第二种分类即 non_promotable" | observations 升级条件 | 即资格判据"结果唯一、无分叉"，提升为第一性判据 |
| "防偶发"的证据意图（≥3 次 / 跨 ≥2 月背后的目的） | rules / observations 升级硬条件 | 保留其作为证据强度的角色，但降级为证据、不继承为门槛，阈值下放 L3 / 治理 |
| accountant 可零积累主动建规则 | rules 来源二 | 反证次数非本质：资格看可判定 + 唯一 + 已批准，不看积累量 |
| active rule 权威只来自事前批准 | "only via approved paths 才能建 rule" | 保留精神（active rule 必须事前批准）；执行机制归 RuleLog / Governance 外阻 |

明确不继承的旧行为：

| 旧行为 | 不继承原因 |
| --- | --- |
| pattern-centered exact description match | scope key 改为 stable entity；description 退回 ER / Alias 做身份原料，避免身份与表面文本耦合脆弱、易误聚、职责错位 |
| Observation 写入时破坏性聚合 count++ / classification_history 就地累加 | 已被 Case Log 决策点 4 否决（逐笔保真 + 派生聚合，非破坏、可重算） |
| Onboarding 历史数据满足硬条件即自动升级 active rule、无二次确认 | 违背"active rule 必须事前批准"；即便来自历史账本，rule-level 晋升仍须走治理批准 |
| rule 只有"存在 / 删除"二态、无过期清理的贫化生命周期 | 不作为约束；rule lifecycle / health 归 RuleLog / Governance 外阻 |

## 分类备案

- 同一 entity 下 rule 集合 overlap-validation 算法、catch-all pattern 语法糖 → L4 / L3(推 RuleLog / 治理执行层；语法糖留 L3)
- accounting treatment 完整性（judgment-free executable）的 exact schema → L3 / JE Generator(留联合 L3)
- rule 资格阈值（重复性 / 跨时间证据强度若需参数化）的具体数字 → L3 / L2·外阻(治理风险判断)
- RuleMatchNode exact input / output fields、handoff object、rule_hit / rule_miss / blocked enum → L3(留联合 L3)
- rule-hit handoff 的 exact field schema、refs 形态、是否包含 rule approval refs / evidence refs → L3(留联合 L3)
- entity-level rule / pattern-level rule 的 exact rule schema、condition enum、accounting treatment schema → L3(留联合 L3)
- RuleMatch matching algorithm、priority order、condition validation、amount range validation → L4 / seam-park(推给记录层 / 执行层)
- ER handoff、workflow router、RuleLog reader / assembler 的具体调用顺序与责任分配 → L4 / seam-park(推给记录层)
- RuleLog 与 EntityLog 同步创建 / mutation 的具体写入机制、多 log finalization、一致性校验 → L4 / seam-park(推给记录层)
- JE Generator 的 exact contract、JE line schema、校验规则 → L3 / L4(留 JE Generator 讨论)
- Rule 使用次数、rule hit count、rule health 的统计写入与维护机制 → L4 / seam-park(推给 RuleLog 维护 / Governance)
- Rule promotion eligibility、approval workflow、candidate queue、rule health、rule modification / downgrade / deletion → L2·外阻(依赖 RuleLog / Governance,挂起)
- case-derived rule promotion evidence 的 exact governance contract → L2·外阻(依赖 RuleLog / Governance,挂起)

## 自检与判定

- 按模板 L1-L2 / Stage 1-2 清单自检: RuleMatchNode 自身的 L1-L2 面已基本收口（存在理由、trigger / routing、读取 / authority、写入 / runtime-only、决策权限、scope 与资格判据、rule 集合结构、hit / miss / invalid handoff、JE Generator 边界、rule-hit handoff、governance 排除、运行时不重算资格）。未过项不在节点内部，而在与 RuleLog / JE Generator / EntityLog automation_policy 的接口语义（见下）。
- 本对象 L2 可否判定完成: RuleMatchNode 节点侧 L2 已实质完成。剩余须继续讨论的不是节点内部，而是圈外 / 接口 seam：
  1. 资格判据最终的"家"：内容判据已在本提案收口，但其归档归属（写在 RuleMatchNode 文档作为被消费语义，还是写在尚未设计的 RuleLog memory layer 并由 RuleMatchNode 引用）取决于 RuleLog M1-M2，RuleLog 这层 memory 目前完全未设计。
  2. automation_policy → RuleMatch 的放行语义：已确认"是否放行 RuleMatch"由 EntityLog 的 automation_policy 承载，不单设 supports_rule_match 字段，"放行 RuleMatch"是 automation_policy 的一种取值语义；剩"哪种取值 = 允许 rule-based automation"的 exact 取值集合 / 语义 → L3（RuleLog / EntityLog 联合），并需与 Governance 对齐。
  3. eligibility / control handoff 的契约面：已在本提案"RuleMatch 输入 handoff 契约面"锁三桶语义（身份 / 放行 / 客观事实）；剩 exact 字段 schema → L3、assembler 谁组装 → L4。
  4. JE Generator 接口：treatment"judgment-free 完整"的判据依赖 JE Generator 需要什么，属跨节点 seam，留 JE Generator 讨论。
  5. RuleLog 查询定位已锁为 entity_id 单坐标（RuleLog 按 entity_id 索引、EntityLog 不存 rule_id 列表），作为对 RuleLog 设计的前置约束；entity_id 索引的存储 / 检索机制 → L4。
