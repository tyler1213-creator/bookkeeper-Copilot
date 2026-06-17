# Case Judgment L2 提案

> 进行中。本提案逐主题收口；当前已讨论确认 Q1（输入加载分层）、Q2（无先例时的自动判断）、Q3（Entity Resolution 输出对本节点的身份约束）、Q4（输出状态分类法）、Q5（自动进 Pending 的触发）、Q6（JE Generator 定位）、Q7（Pending handoff 最小语义契约）、Q8（candidate signals 范围）、Q9（High Confidence 后的路由）、Q10（Reasoning / Audit Trace 形态）、Q11（mixed-use 等 soft risk 处理）、Q12（多源冲突优先级与保守原则）、Q13（external lookup 边界）、O1（代码与模型分工、执行顺序与路由机制）、O2（Pending 承接与 reason 定位）、G1（Case Log 先例读取：代码递客观事实、CJ 当辅助证据权衡）。
>
> 命名约定：本节点的自动放行输出统一称 **High Confidence Classification（高置信度分类）**，不使用 “operational classification”。

## 1. 依赖

- **上游身份（已锁）**：Entity Resolution 对下游只声明 `stable` / `unknown` 两种身份状态（`BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md`）。`unknown` 的下游影响已锁为「Case Judgment 不走高置信度自动分类通道，输出 Pending」。
- **上游证据（已锁）**：Evidence Intake / Preprocessing 提供可追溯 evidence 与 evidence refs，并分配稳定 `transaction_id`。
- **上游结构（已锁）**：Profile / Structural Match 已确认当前交易未被结构性路径完成；identity-independent 的结构性交易（如 internal transfer、bank fee、interest）主要由此路径处理。
- **Entity Log（已锁，读取）**：Case Judgment 读取 entity identity basis、risk flags 与 `automation_policy`（`BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md` §2）。`automation_policy` 是实体级控制状态，升级 / 放宽须 accountant approval，系统只能受控收紧。
- **Case Log（已锁，下游记忆层）**：Case Log 按 `entity_id` 组织 completed-case precedent；仅 stable entity 可作为 case identity handle，unknown 不可（`BK_Copilot/memory_layers/case_log/02_authority_lifecycle_and_boundaries.md`）。Case Log 是辅助先例，不是确定性规则源。
- **entity-level Knowledge Summary（已锁，下游记忆层）**：按 `entity_id` 锁定后按需注入 Case Judgment，自带 authority / usage 护栏（`BK_Copilot/memory_layers/case_log/01_memory_intent.md`）。
- **JE Generation Node（已锁，下游）**：纯确定性构造层，消费本节点 High Confidence Classification 的确定 (COA, HST/GST) 决定；不做分类判断。
- **常驻知识**：客户 COA、加拿大会计与税务准则、bookkeeping 通用常识、客户 Profile 当前批次稳定快照。

## 2. L2 决策

### 2.1 输入：固定加载与动态加载

**结论：**
Case Judgment 的输入按加载方式分为三层。

固定加载（常驻知识）：按客户 / 批次稳定可用、可缓存复用，不随单笔交易重新获取：
- 客户 COA；
- 加拿大会计与税务准则（含 HST/GST 处理原则）；
- bookkeeping 通用常识；
- 客户 Profile 的当前批次稳定快照（行业、business_type、省份、tax config、银行账户结构等）。Profile 以批次开始时已提交的快照为准，批次中途不漂移，Case Judgment 不修改 Profile。

动态加载 — 每笔必取（代码组装）：当前交易事实与 evidence（金额、方向、日期、bank account、raw/real description、transaction_id、receipt、cheque、accountant 补充说明等）；以及 Entity Resolution 的身份判断结果（`stable` / `unknown` 及其 reason / evidence refs）。

动态加载 — 按需检索（以 stable `entity_id` 为钥匙）：仅当身份为 `stable` 时，按 `entity_id` 检索并注入 Case Log 中该 entity 的相关案例先例（含 evidence condition、context note、confirmed_by、use_level），以及该 entity 的 entity-level Knowledge Summary（携带 authority / usage 护栏，作为建议而非规则）。身份为 `unknown` 时不进行此层检索。

**为什么（锚定产品目标）：**
记忆复用、有证据支持的建议、审计性、自动化率。区分常驻知识与每笔交易上下文，避免把全部上下文压平成一团，使实现层能明确哪些应提前缓存、哪些应每笔拼装、哪些应凭稳定身份按需检索；按 `entity_id` 检索而非全量灌入，保证 unknown 身份不会污染 case 复用，也避免无关历史进入判断。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:读取对象 / 输入加载` → `Case Judgment 输入分三层。固定加载（常驻知识）：客户 COA、加拿大会计与税务准则、bookkeeping 通用常识、客户 Profile 当前批次稳定快照；可缓存复用，不随单笔交易重新获取；Profile 以批次开始快照为准，批次中途不漂移。动态加载-每笔必取：当前交易事实与 evidence、Entity Resolution 身份结果（stable / unknown 及 reason / evidence refs）。动态加载-按需检索：仅当 stable 时按 entity_id 注入 Case Log 相关先例与 entity-level Knowledge Summary；unknown 不进行此层检索。`

**浮现的新 open boundary：**
固定加载知识如何注入 / 缓存 / retrieve，以及 Case Log per-entity rollup / retrieval pack 的生成机制，属于 L4 / seam（见缺口地图 Case Judgment section），不在本 L2 冻结。

### 2.2 无先例时的 high confidence classification

**结论：**
Case Log 是辅助先例，不是 high confidence classification 的必要条件；缺少历史先例不构成自动阻断。

是否允许进入 high confidence classification，取决于一个复合落地条件：当前 evidence 结合常驻知识，能否唯一、确信地落到单一站得住的 (COA 科目, HST/GST 处理) 上。具体为：
- 交易主体 / 性质清晰；
- COA 中只有一个合理科目；
- HST/GST 处理方式可确定（非 unknown）；
- 不存在未解决的 authority / identity / evidence / governance 阻断。

满足上述条件即可进入 high confidence classification，无论是否存在 Case Log 先例。先例存在时用于加强与校验该判断，但不能替代当前证据，也不能升格为确定性规则。

**为什么（锚定产品目标）：**
自动化率、有证据支持的建议、记忆复用。若把先例设为自动判断的必要条件，会让大量证据本已充分、但属于该 entity 首次出现的交易被迫进入 Pending，压低自动化率；把闸门定义为「能否唯一落地 (COA, HST)」而非「有无先例」，使判断锚定当前证据强度，同时保持 Case Log 作为先例的辅助与校验地位、不被升格为规则。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:决策权限 / high confidence classification 条件` → `Case Log precedent 是辅助先例，不是 high confidence classification 的必要条件；无先例不构成自动阻断。进入 high confidence classification 的复合落地条件：当前 evidence 结合常驻知识可唯一、确信地落到单一站得住的 (COA 科目, HST/GST 处理)：交易主体 / 性质清晰、COA 中只有一个合理科目、HST/GST 可确定（非 unknown）、不存在未解决的 authority / identity / evidence / governance 阻断。先例存在时加强与校验判断，不替代当前证据，不升格为确定性规则。`

**排除的替代 + 理由：**
把 Case Log 先例设为 high confidence classification 的必要前提。理由是这会让首次出现、但证据充分的交易被迫 Pending，削弱自动化率，并使 Case Log 从「辅助先例」事实上升格为准入门槛。

### 2.3 Entity Resolution 输出对 Case Judgment 的身份约束

**结论：**
Entity Resolution 对下游只声明两种身份状态：`stable` 与 `unknown`。Case Judgment 据此确定身份层的处理边界：

- `stable` 是 high confidence classification 的必要前提。身份为 `stable` 时，Case Judgment 获得可用身份锚点，可据此按 `entity_id` 读取 Case Log，并在 2.2 落地条件成立、无硬阻断时进入 high confidence classification。`stable` 的 provenance（复用既有 / 新建）不改变其作为身份基础的效力。
- `unknown` 一律不进入 high confidence classification。Case Judgment 输出 Pending；可利用非身份上下文（receipt items、金额模式、bank descriptor 特征等）给出推断性建议，附于 Pending，交由下游向会计师确认。

identity-independent 的交易（如 bank fee、interest）由上游 Profile / Structural Match 处理，不构成 Case Judgment 在 `unknown` 下自动分类的例外。

**为什么（锚定产品目标）：**
审计性、accountant control、自动化率。身份必要前提保证自动会计分录建立在可追溯、稳定的身份之上；把 `unknown` 一律收敛为 Pending（可附推断建议），既不伪造身份、不绕过会计师确认，又不浪费已有的非身份上下文。该约束直接沿用 Entity Resolution 正式文档已锁结论，保持两节点契约一致。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:决策权限 / 身份约束` → `Case Judgment 消费 Entity Resolution 的两态身份输出。stable 是 high confidence classification 的必要前提：stable 时获得可用身份锚点，可按 entity_id 读取 Case Log，并在落地条件成立、无硬阻断时进入 high confidence classification；provenance（复用 / 新建）不改变身份基础效力。unknown 一律不进入 high confidence classification，输出 Pending，可附基于非身份上下文的推断性建议交下游向会计师确认。`
- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:边界` → `identity-independent 的交易（如 bank fee、interest）由上游 Profile / Structural Match 处理，不构成 Case Judgment 在 unknown 下自动分类的例外。`

**排除的替代 + 理由：**
在 Case Judgment 层为 `unknown` 身份保留 identity-independent 自动分类旁路。理由是 Entity Resolution 正式文档已将该类交易归上游结构路径处理，并明确「到达 Case Judgment 的交易绝大多数需要 entity identity」；在本节点放行会与上游结构路径抢边界，且削弱「自动分类必须基于稳定身份」这一已锁约束。

### 2.4 输出状态分类法：两类输出

**结论：**
Case Judgment 对下游只声明两类输出：**High Confidence Classification（高置信度分类）** 与 **Pending**。不设独立的 review-required 第三态。

凡 hard block 命中（authority / identity / evidence / governance 级阻断），一律输出 Pending 并携带 reason，绝不进入 High Confidence Classification。Pending 的唯一下游 consumer 是 Coordinator；review / governance 级的人工处理在 Coordinator 与会计师的交互中达成（必要时由 Coordinator 在获得会计师指令后产出 candidate signal），不在本节点表达为独立输出状态。reason 只解释模型为何输出该结果，不作为下游分流开关。

**为什么（锚定产品目标）：**
accountant control、审计性、自动化率。两类输出把本节点职责收敛为「能否高置信度放行」这一个判断；将 review / governance 阻断表达为 Pending 的 reason 而非独立状态，避免在下游 Coordinator 及其后续人工处理路径尚未设计时过早冻结跨节点路由状态机，同时完整保留「hard block 一律不得自动放行」的保护。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:输出类别` → `本节点对下游只声明两类输出：High Confidence Classification 与 Pending。任何 hard block 命中时一律输出 Pending 并携带 reason，不得进入 High Confidence Classification。Pending 唯一 consumer 为 Coordinator；review / governance 级人工处理在 Coordinator 与会计师交互中达成，本节点不设独立 review-required 输出状态；reason 只解释模型为何输出该结果，不作为分流开关。`

**排除的替代 + 理由：**
三态输出（high confidence / pending / review-required）。理由是 review-required 的本质同样是「交人工」，与 Pending 同向；在下游人工路由尚未设计时增设第三态，会过早冻结跨节点状态机，却无额外保护收益——「hard block 不得自动放行」在两类模型下已由「hard block → Pending」完整承载。

### 2.5 自动进入 Pending 的触发

**结论：**
本节点自动进入 Pending（不进 High Confidence Classification）的触发，L2 收敛为两类，不展开 exact 字段：

- **identity 非 stable**：Entity Resolution 输出 `unknown`（见 2.3）。
- **entity / case 级控制状态要求人工**：
  - Entity Log `automation_policy` 处于收紧值（如 review_required / disabled）——会计师对复杂 / 高风险 entity 施加的「每次必审」约束落在这里，Case Judgment 读取（Entity Log 正式文档 §2）。
  - Case Log `use_level` / `context_note` 标记该先例必须人工复核 / 不可自动复用（exception-only、must-ask、accountant correction）——Case Judgment 凭 stable `entity_id` 按需读到。

命中任一即输出 Pending 并携带对应 reason。

**为什么（锚定产品目标）：**
accountant control、审计性。让会计师能对复杂 / 高风险 entity 施加「每次必审」约束，并让历史例外（correction、mixed-use、must-ask）阻止静默自动复用；把触发锚定到已锁来源（Entity Log `automation_policy`、Case Log `use_level` / `context_note`）而非新设字段，避免颗粒度过早冻结。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:决策权限 / hard block` → `Case Judgment 读取 Entity Log automation_policy，以及（凭 stable entity_id）Case Log use_level / context_note。当 automation_policy 处于收紧值，或 case 级约束要求人工复核 / 禁止自动复用时，本节点不得进入 High Confidence Classification，输出 Pending 并携带对应 reason。identity 非 stable 同样进入 Pending。exact enum 取值留 L3。`

**浮现的新 open boundary：**
`automation_policy` 与 `use_level` / `confirmed_by` 的 exact enum 取值分别留 Entity Log / Case Log 的 L3。

### 2.6 JE Generator 是纯确定性执行层

**结论：**
JE Generation Node 是纯确定性执行层，不做分类判断。本节点 High Confidence Classification 输出必须携带确定的 (COA 科目, HST/GST 处理) 决定，足以让 JE Generation Node 直接确定性构造分录；JE Generation Node 只把该结果转成标准分录格式并做格式校验，不重新分类、不补判断。Case Judgment 不构造、不校验分录。

**为什么（锚定产品目标）：**
审计性、自动化率、单一职责。会计分类的权威集中在 Case Judgment，分录构造与校验集中在 JE Generation Node，避免分类逻辑在两处重复或漂移。沿用旧系统 build_je_lines「纯确定性计算、单一职责、不做分类判断」的边界。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:输出类别 / 下游契约` → `High Confidence Classification 输出携带确定的 (COA 科目, HST/GST 处理) 决定，作为 JE Generation Node 的确定性构造输入。JE Generation Node 不做分类判断，只构造与校验分录格式。Case Judgment 不构造、不校验分录。`

**排除的替代 + 理由：**
让 JE Generation Node 在构造时自行决定或补全分类。理由是会把会计分类权威分散到纯计算层，破坏单一职责与审计可追溯，并使分类逻辑出现两份来源。

### 2.7 Pending Handoff 的最小语义契约

**结论：**
本节点输出 Pending 时，必须携带足以让会计师理解「客户是谁、系统做了什么、为什么仍无法解决」的最小语义内容：

- **客户背景**：当前交易主体 / 客户结构背景（来自固定加载的 profile / identity context）。
- **历史记录**：该 entity / 交易在 Case Log 中是否有相关先例（来自按 `entity_id` 的动态检索；仅 stable 时有）。
- **阻塞原因**：当前卡在哪里、为什么不能判断。
- **已执行的工作**：本节点已做过哪些尝试及其结果（如已联网查询但无可确定结果、缺乏 evidence 辅助）。
- **可能性结果**：即使已推断出 entity 或某个可疑的最终分类，但因某些项（如 HST/GST 处理）不确定而无法落地的说明，作为给会计师的推断性建议。

目标是把问题转译成会计师能回答的形式。Coordinator 尚未正式设计，本题只锁 Pending 必须携带的语义，不锁交互机制。

**为什么（锚定产品目标）：**
accountant control、审计性、有证据支持的建议、自动化率。会计师高效处理 Pending 的前提是不必重做系统已做的工作；把背景、历史、卡点、已执行工作和最佳推断一并呈现，最大化会计师单次回答即可解锁的比例。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:Pending 行为` → `Pending 输出必须携带：客户 / identity 背景、相关 Case Log 先例（如有）、阻塞原因、已执行动作及结果、以及可选的推断性建议（含不确定项说明）。这些内容构成对 Coordinator / Pending Node 的语义契约；exact field schema 与 Coordinator 交互机制留后续。`

**浮现的新 open boundary：**
Coordinator / Pending Node 的交互机制、提问模板与 pending handoff 的 exact field schema 未冻结（挂起，依赖 Coordinator 设计）。

### 2.8 Candidate Signals：唯一出口为 case memory update

**结论：**
Case Judgment 对长期记忆只有一个候选出口：**case memory update candidate**（仅 High Confidence 自动路径产生）。它不输出 entity / alias / role / rule / automation-governance 候选。

统一原则：CJ 把判断沉淀进 case 记录；其余一切——entity、alias、role、rule、automation / governance——要么属 Entity Resolution / Coordinator / accountant 的权威（身份类），要么是 case 衍生的风险 / 升级，被有意路由「穿过 case 记录」、由下游治理 / lint / review 从累积案例历史里派生，而不是由单次 CJ 运行直接断言。

逐项：

- **case memory update candidate（可发）**：High Confidence 路径完成后，交易具备 stable entity + 确定分类 + 处理细节，CJ 发出该候选供 Case Memory Update Node 读取写入。三点边界：(1) 写入时点不在 CJ——Case Log 写入资格 = stable-linked 且已 finalized 交易 + `transaction_log_ref` / finalization proof，实际写入在 finalization 时由对应节点执行；(2) CJ 的增量是判断派生的元数据（`evidence_condition`、`context_note`、`use_level` 倾向、risk flags），而非授权写入——base 写入资格对所有 stable-linked finalized 交易自动成立；(3) `confirmed_by = system` 高置信度 case 只是强辅助先例，不自动授予未来自动化放行。注：pending → accountant 确认产生的 case（`confirmed_by = accountant`）源自 Coordinator / finalization 路径，不由 CJ 发出；CJ 的 case memory 候选只覆盖 High Confidence 自动路径。
- **entity（不发）**：识别主体上 CJ 与 ER 信息对等，ER 判不出 unknown 时 CJ 同样判不出；case 衍生的 entity risk 经 Case Log 历史由下游派生，不由 CJ 直发。
- **alias（不发）**：Alias 是 Entity Log 的反查投影，与 Entity Log 写入同步；CJ 写不了 Entity Log，自然写不了 Alias。
- **role（不发）**：role 字段已删除，不重启。
- **automation / governance（不发）**：automation policy 升级 / 放宽须 accountant approval，系统只能受控收紧；CJ 不发治理候选，走「落进 case 记录 → 下游治理派生」。
- **rule（不发）**：规则的本质是可重复的确定性模式，单笔判断无法确立重复性；rule promotion 是 Case Log 历史级观察，由下游 rule / automation review flow 从累积案例依据派生并需 governance approval。

**为什么（锚定产品目标）：**
accountant control、审计性、自动化率。把 CJ 的长期记忆出口收敛为单一 case memory，防止单次运行越权进入身份 / 治理 / 规则权威；让 case 衍生的风险 / 升级统一沉淀在 case 记录这一个累积点，再由治理路径凭历史依据 + 批准派生候选，既保留学习信号又不绕过审批。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:candidate signals / 输出类别` → `Case Judgment 对长期记忆只有一个候选出口：case memory update candidate，仅在 High Confidence 自动路径产生，作为判断派生的可学习 case 内容，由 Case Memory Update Node 在交易 finalized 后凭 transaction_log_ref 写入；CJ 不授权写入，不输出 entity / alias / role / rule / automation-governance 候选。confirmed_by = system 高置信度 case 只是强辅助先例，不自动授予自动化放行。case 衍生的 entity / rule / automation 风险经 Case Log 历史由下游治理 / lint / review 派生。`

**排除的替代 + 理由：**
沿用被审计新系统设计的六信号清单（case_memory / entity / alias / role / rule / automation-governance）。理由：entity / alias 越 CJ 的信息与权威；role 字段已删；rule 需历史重复性、单笔无法确立；automation / governance 升级须人类批准；这些直接发信号会让单次运行越权并制造多个未受控的 candidate 来源，正确路径是穿过 case 记录由下游派生。

**浮现的新 open boundary：**
Case Memory Update Node 的 exact 写入机制、`Case Log` 与 `Transaction Log` 的 finalization trigger order、`case_memory_update_candidate` 的 exact field schema 留 L3 / L4。

### 2.9 High Confidence 后的路由：直接进 JE，不设强制人工门

**结论：**
判定为 High Confidence Classification 后，直接调用 JE Generation Node 生成分录，不在生成前设置强制人工审核门。生成后在 Case Log 与 Transaction Log 双记录，并明确标注该交易为「AI 高置信度自动完成」。

事后人工如何查看高置信度结果（如按置信度分栏的当日汇总界面、抽查机制）属 UI / 后续设计，不在本 L2 冻结。

**为什么（锚定产品目标）：**
自动化率、审计性。若每笔高置信度交易都强制人工介入，自动化的核心价值即丧失。直接进 JE 保住自动化率；Case Log + Transaction Log 双记录并标注来源，保证高置信度结果事后可审、可抽查、可定位，不牺牲审计性与 accountant control。沿用旧系统「高置信度直接构造 JE、并在报告中标成 AI 高置信度一档供事后审」的边界。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:下游契约 / 路由` → `High Confidence Classification 直接进入 JE Generation Node 构造分录，不设生成前强制人工审核门。结果在 Case Log 与 Transaction Log 双记录，并标注为 AI 高置信度自动完成。事后人工查看的呈现与抽查机制属 UI / 后续设计。`

**排除的替代 + 理由：**
High Confidence 也先挂一道 review 门才进 JE。理由是这会让每笔交易都需人工介入，抹掉自动化核心价值；「hard block 一律不自动放行」已由两类输出模型（2.4）承载，剩余的高置信度结果以「直接进 JE + 双记录标注 + 事后抽查」即可兼顾稳健与效率。

**浮现的新 open boundary：**
高置信度结果事后审查的呈现 / 交互（汇总界面、按置信度分栏、抽查机制）未冻结，属 UI / 后续设计。

### 2.10 Reasoning / Audit Trace：结构化论证链，归入 Transaction Log

**结论：**
两类输出（High Confidence Classification 与 Pending）都必须带一段结构化 reasoning。它是一条**收敛到两个核心结论（COA、HST/GST）的论证链**，而非并列清单。链条的承重环节按因果顺序：

1. **主体（起点，直接引用）**：主体身份由 Entity Resolution 输出，Case Judgment 不重新论证，原封不动引用 ER 的 stable entity 值及其 identity reason，作为已知前提。
2. **凭据层论证**：简明说清本次结论凭当前证据里的哪个关键点成立，可点名到具体 evidence ref（小票、支票等证据有限且证明力强）。
3. **历史佐证（entity 概要级）**：引该 entity 的历史表现，仅到实体概要级（不点具体 case 行）——出现频率、过去分类是否与本次一致、历史上出现过几种不同分类结果；来自 Case Log 读取期派生的 per-entity rollup。
4. **收敛到结论 + 论证**：COA 与 HST/GST 各自怎么定，给最能佐证的关键点。Pending 时给基于证据的论证（如「小票看 80–90% 属 X，但该 entity 历史有更多不同的类似分类 → 需人工」）。

原则：论证思维而非列举思维；尽量短，但每个承重环节都保留「最能佐证它的那一点」，颗粒度恰好够人分块定位错误（错在主体引用、凭据解读、历史佐证，还是 COA / HST 哪一步）。置信度 / 概率可作为论证的一部分，但不能用孤立的 confidence 分数替代论证、把不确定性藏进数字。

整条 reasoning 记入 Transaction Log，供后续治理 / Review / 审计节点读取。该 reasoning 始终是 runtime explanation，永不升格为 durable authority、rule 或 approval。

**为什么（锚定产品目标）：**
审计性、accountant control、correction learning、自动化率。结构化论证链让用户看到 AI 的思维不是一团浆糊，而是「证据 1 → 证据 2 → 结论」；颗粒度恰好够分块定位错误，使会计师能精准指出错在哪一环（精准纠错），纠正经 Review / Coordinator 落回并以 accountant correction 进 Case Log，形成干净的学习信号。reasoning 入 Transaction Log 与其作为审计 source of truth 一致；不升格为权威，守住「reasoning 不是 durable authority」的铁律。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:Audit / Trace 边界` → `High Confidence 与 Pending 两类输出都必须带结构化 reasoning，形态为收敛到 (COA, HST/GST) 的论证链：主体（直接引用 ER 身份值与 identity reason，不重新论证）→ 当前证据凭据论证（点名 evidence ref）→ entity 历史佐证（实体概要级：出现频率 / 过去分类是否一致 / 出现过几种不同分类，不点具体 case 行）→ 收敛到 COA 与 HST/GST 结论。论证而非列举，短而不失承重颗粒度，可分块定位错误。置信度 / 概率可入论证，不可替代论证。整条 reasoning 记入 Transaction Log，供治理 / Review / 审计读取；reasoning 不升格为 durable authority、rule 或 approval。`

**排除的替代 + 理由：**

- 逐条 case 引用（指明参考了哪一条历史 case）。理由是精确到条即退化成查 Transaction Log，而 Case Judgment 读取的本就是实体概要级 rollup，不掌握逐条 case 细节。
- 逐 token / 长篇思维链。理由是噪音多、不提升错误定位能力，且与「reasoning 不是权威」相冲。
- 用孤立 confidence 分数代替论证。理由是把不确定性藏进数字，使人无法分块定位错误，削弱信任与纠错。

**浮现的新 open boundary：**
reasoning 在 Transaction Log 的 exact 存储形态 / field schema 留 L3；后续治理节点从 Transaction Log 读取 reasoning 的 exact contract，待治理节点设计时冻结。

### 2.11 Mixed-use / 个人消费等 soft risk：默认公司支出，证据明确且重大才上交

**结论：**
Case Judgment 对 mixed-use / 个人消费风险采取「默认公司支出、不主动稽查」的姿态，而不是旧系统「默认怀疑、强证据才恢复」的姿态。

- **默认姿态翻转**：对能合理当成公司支出的交易，按正常会计分类放行（满足 2.2 落地条件即 High Confidence），不因「商家属零售 / 可能夹带私人」就默认降级。Case Judgment 不充当 mixed-use 稽查员，不主动拆解证据去逐项搜寻可疑私人物品。
- **证据照常用满**：小票及其品类明细仍是最强证据之一，必须用于确定最终会计分类（如水泥钢筋 → Supplies）。「不主动稽查」不等于放弃证据；证据既能佐证「是公司用途」（→ High Confidence），也可能明确暴露「非公司用途」。
- **唯一的停下条件**：仅当手上证据已**明确**指向「这不是公司支出」、**且金额重大**时，Case Judgment 才不打 High Confidence，把它作为一次决策交回会计师（个人 vs 公司 / 费用 vs Shareholder Loan / Owner's Draw 的处理选择）。中间「看着像公司、可能夹带私人、但分不清」的一大片，一律默认公司支出放行。
- **mixed-use 不是证据缺口**：个人 vs 公司是会计师承担的商业 / 税务立场判断，并非更多证据就能解决的事实问题（一张完美小票也回答不了「这台洗碗机是否为公司」）。因此不以「证据不足 → 补证据 Pending」套用它。

**为什么（锚定产品目标）：**
自动化率、accountant control、审计性。目标受众为加拿大中小企业代理记账，月交易量小、个人公司混用普遍且属市场常识；逐笔稽查 mixed-use 会制造大量无意义停顿、加重会计师负担，且零售类常无明细小票、根本无法逐笔分清。两条已有机制兜底使「默认公司支出」既不越权、又不失控：(1) 模糊件靠 2.9 的事后报告复核——High Confidence 进报告由会计师事后过目，系统并非无监督地替会计师赌，控制权仍在；(2) 真·明显私人件靠「High Confidence 即确信」的诚实——系统对明摆着的私人物品并不确信其为公司费用，就不该打 High Confidence，而是老实交回人。系统不默认替会计师做激进费用化决定（那是会计师的执业风险），但一旦其立场确立（见下）即自动化。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:决策权限 / soft risk 处理` → `对 mixed-use / 个人消费风险，Case Judgment 默认按公司支出正常分类放行，不充当稽查员、不主动拆解证据搜寻私人物品；小票品类等证据仍须用于确定分类。仅当手上证据明确指向非公司支出且金额重大时，不打 High Confidence，作为一次个人 vs 公司的决策交回会计师。其余「可能夹带私人但分不清」的交易默认公司支出放行，靠事后报告复核（见 2.9）兜底。mixed-use 属会计师立场判断而非证据缺口，不以补证据 Pending 套用；materiality 阈值留 L3。`

**排除的替代 + 理由：**

- 旧系统「零售即默认降级、强证据才恢复 High Confidence」。理由是它把 mixed-use 误当信息缺口，制造大量无意义 Pending、加重小企业会计师负担，而更多证据并不能解决立场问题。
- 让系统静默地把可疑私人支出一律费用化为公司支出。理由是激进费用化是会计师承担的执业风险，系统无权替其默认承担；正确路径是立场缺失时交回会计师，立场确立后再自动化。

**浮现的新 open boundary：**
「金额重大」的 materiality 阈值（具体数字）留 L3；会计师 / 客户既定处理立场如何沉淀与读取（Profile 结构事实、entity-level Knowledge Summary、Case Log 先例）的 exact 机制留对应记忆层与 seam。

### 2.12 多源冲突的优先级与保守原则

**结论：**
Case Judgment 手上同时握有多个信息来源（治理 / 权威约束、Profile 结构事实、当前证据、Case Log 先例、通用知识 / Knowledge Summary），冲突时按以下统一优先级裁定。从高到低：

- **第 0 层 · 硬治理 / 权威阻断（最高，证据再强不可越过）**：已生效 Governance Log 限制；Entity Log `automation_policy` 收紧值（review_required / disabled）；身份硬阻断（ER `unknown`）；Case Log `use_level` 标记必审 / 不可复用 / 已失效；已批准的 Rule Log 权威优先于 Case Log 先例。命中即封顶，结果为 Pending，不进 High Confidence。依据：这层均源于会计师 / 治理决定，可能基于系统看不到的额外上下文。
- **第 1 层 · 权威客户结构事实（Profile）+ 限定范围的会计师确认**：Profile 结构事实（business_type、has_hst_registration、province、tax_config、bank_accounts）对其所管维度有决定权（如 has_hst_registration=false → HST exempt，压过小票税额；business_type → Shareholder Loan vs Owner's Draw）；Intervention Log 中会计师对本 entity / case 的明确确认或纠正，仅在其确认范围内有效，模糊批注不泛化。
- **第 2 层 · 当前交易证据**：本笔自己的证据（小票、支票、本笔会计师补充说明）。当前证据优于历史，因其直接针对这笔交易。
- **第 3 层 · Case Log 先例**：该 entity 的历史案例。与特定 entity 强相关，故高于通用知识、低于当前证据。
- **第 4 层 · 通用知识 + Knowledge Summary**：加拿大税务 / 会计准则、bookkeeping 常识、可读知识摘要。最泛化垫底；Knowledge Summary 是可读背景，不能压过上层任何来源。

保守行为原则：歧义不等于可以猜；证据冲突不等于挑一个看起来顺眼的版本。任何在上述顺序内无法干净解决的冲突——尤其当前证据与更高层来源矛盾、或同层两来源互斥且都说得过去——一律不打 High Confidence，输出 Pending 并讲清冲突点交回会计师；绝不为推进流程硬编一个 winner。

**为什么（锚定产品目标）：**
审计性、accountant control、有证据支持的建议、自动化率。统一优先级让裁定可预期、可审计，并把人类 / 治理决定置于机器证据之上（守住 accountant control）；当前证据高于历史保证判断锚定这一笔的事实；保守原则防止系统用"挑顺眼版本"的方式把不确定性藏进自动放行，确保拿不准时干净地回到人。该序与 Entity Resolution（§9）、Case Log（§8）已锁的局部 authority 顺序一致。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:冲突处理 / 决策权限` → `多源冲突按统一优先级裁定（高→低）：(0) 硬治理 / 权威阻断（Governance Log 限制、Entity Log automation_policy 收紧值、ER unknown、Case Log use_level 必审 / 不可复用 / 失效、已批准 Rule Log 优先于 Case Log 先例）命中即封顶为 Pending；(1) 权威客户结构事实 Profile + 限定范围的会计师确认；(2) 当前交易证据；(3) Case Log 先例；(4) 通用知识 + Knowledge Summary。保守原则：歧义不猜、冲突不挑顺眼版本，无法在该序内干净解决即输出 Pending 并讲清冲突点，绝不硬编 winner。`

**排除的替代 + 理由：**
让当前证据可凌驾治理 / 权威阻断（强证据即放行）。理由是权威阻断承载系统看不到的会计师 / 治理上下文，被证据越过会破坏 accountant control 与审计性。

### 2.13 External Lookup：CJ 可联网，但只为会计事实、不为身份

**结论：**
Case Judgment 可以联网查询，但严格限定为确定当前交易的**会计事实**（该落哪个 COA、HST 如何处理），不用于确定身份。套用 Entity Resolution 的联网纪律（ER 02 §3 / §8 / §10）：

1. **用途边界**：CJ 联网只服务会计事实判断（COA / HST），不定身份、不"造" stable entity。身份若 `unknown`，无论联网查到什么仍走 Pending（见 2.3）。
2. **搜索行为本身不是 authority**：联网动作本身不赋予权威。
3. **clue vs evidence point**：搜索结果列表、引擎摘要、模型复述只能当线索；只有由搜索打开、可追溯的外部来源才能作为 evidence point。
4. **充分性门槛 = 2.2 的闸门**：联网得到的 evidence point，只有与本笔证据 + 常驻知识合在一起能落到唯一、站得住的 (COA, HST) 时，才支持 High Confidence；联网不降低门槛。
5. **查不实 → Pending**：若联网只给出模糊 / 同名 / 低质量聚合 / 无法确定的结果，停留在线索级，不得打 High Confidence，输出 Pending。
6. **可追溯 / 可审计**：若某条 High Confidence 依赖联网外部来源，必须在 reasoning / trace（见 2.10）保留可追溯外部来源引用供事后核验；exact 字段形态留 L3。

seam：ER 联网=查身份，CJ 联网=查会计事实，权威不重叠。

**为什么（锚定产品目标）：**
自动化率、有证据支持的建议、审计性。大量零售 / 冷门商家的 description 不足以确定品类，禁止 CJ 联网会丧失有效信息源、压低自动化率；套用 ER 已验证的"行为非权威、只认可追溯来源、不达唯一即 Pending、保留可追溯引用"纪律，使联网在提升覆盖率的同时不放大 SEO / 同名 / 结果漂移风险，并守住身份与分类的 seam。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:读取对象 / external lookup` → `Case Judgment 可联网，限定为确定会计事实（COA / HST），不用于身份；身份 unknown 时无论联网结果仍走 Pending。搜索行为非 authority；搜索结果列表 / 摘要 / 模型复述只能作 clue，只有可追溯外部来源可作 evidence point。该 evidence point 只有与本笔证据 + 常驻知识共同落到唯一站得住的 (COA, HST) 时才支持 High Confidence；查不实则 Pending。依赖联网外部来源的 High Confidence 必须在 trace 保留可追溯引用，exact 字段形态留 L3。`

**排除的替代 + 理由：**

- CJ 完全不联网（联网全锁 ER）。理由是会丧失确定会计事实所需的外部信息源（如冷门商家品类），压低自动化率；且 ER 正式文档只把 ER 自身联网限定为查身份，并未禁止 CJ 为会计事实联网，二者 seam 不重叠。
- 允许 CJ 用联网结果反推 / 确认身份。理由是身份权威锁在 ER；CJ 联网定身份会搅浑身份与分类边界，且身份 unknown 本就走 Pending，反推无收益。

**浮现的新 open boundary：**
联网外部来源 evidence reference 的 exact field schema、网页快照 / 缓存 / retrieval 机制、source quality 判定，与 ER 侧对齐，留 L3 / L4。

### 2.14 代码与模型的分工、执行顺序与路由机制

**结论：**
Case Judgment 是代码驱动的流程，大模型是其中被调用一次的判断工具，不是一个自主 agent。分工与顺序固定为「代码 → 模型 → 代码」：

- **代码（调用模型之前）**：(1) 判断本节点是否被触发；(2) 组装上下文——固定加载的常驻知识、每笔必取的交易事实与 Entity Resolution 身份结果、以及仅当身份 `stable` 时凭 `entity_id` 检索的 Case Log 先例与 entity-level Knowledge Summary（见 2.1）；(3) 读取所有确定性字段，算出这笔交易系统最多能做到哪一步（能否自动落地）；(4) 把确定性约束整理成结构化输入备好。
- **模型（在代码备好的上下文内做一次判断）**：识别交易主体 / 性质、选择 COA、确定 HST/GST、判断当前证据是否足以落到唯一站得住的 (COA, HST/GST)、生成 reasoning，并给出它对自身判断的置信度。模型不获取信息、不调用工具、不自行决定控制流。
- **代码（模型之后）**：裁定最终路由、做格式校验、转交下游 / 写入。

**置信度与许可是两件事，最终路由由代码决定：**

- 模型负责回答「这笔是什么、我有多确定」（判断内容 + 自评置信度）。
- 代码负责回答「系统能不能自己把这笔落地」（许可）。
- 两者相互独立。最终是 High Confidence 自动落地还是 Pending，一律由代码裁定：代码先读取所有确定性字段，**只要存在硬阻断（如 Entity Log `automation_policy` 收紧值、Governance 限制、身份 `unknown`、Case Log 标记必审 / 不可复用），无论模型置信度多高，一律 Pending**；不存在硬阻断时，代码才采信模型的置信度作为放行信号（足够确信 → 自动落地；否则 Pending）。
- 模型永不持有路由权；「是否输出高置信度并自动放行」这一决定的核心在代码，不在模型。

**被阻断不等于不让模型思考：**
代码已裁定必 Pending 的交易，仍可让模型对其做判断并给出建议，附在 Pending 中交会计师，以降低会计师的决策成本。告知模型「这笔将交人工」只用于让它把输出写成「供会计师参考的建议」形态，不用于压低它的置信度——用路由规则去压置信度既不可靠，又会损坏建议本身的价值和置信度信号的真实性。

**硬阻断必须由结构化字段确定性可算（前瞻不变量）：**
硬阻断的判定必须基于结构化字段（enum / flag）由代码确定性计算；任何需要读自由文本、做语义解读才能发现的约束都不算硬阻断，必须降级为 soft risk，交模型在限定框内判断。当前系统尚未设计这些上游 / 记忆层结构化字段，故暂无成形的硬阻断清单，但「硬阻断 = 代码可算的结构化字段」是未来必须满足的硬标准。

**模型调用次数：**
默认单次模型调用（主体识别 / COA / HST / 置信度各步是这一次调用内部的推理步骤，不是多次调用）。如确需多次调用，每次调用是否发生、喂什么，必须由代码的确定性条件决定，绝不由模型上一次的自由判断决定；模型不得自决是否再次调用。此约束守住「模型不是 agent、不做多步推理」。

**为什么（锚定产品目标）：**
审计性、accountant control、自动化率。把放行许可交给代码的确定性字段判定，使「系统不得自动落地」成为硬保证，不依赖模型是否听话，杜绝长上下文淹没关键约束的失败模式；同时保留模型对被阻断交易的判断与建议，最大化会计师单次即可解锁的比例，不牺牲自动化与体验。把置信度与许可分离，避免用路由规则逼模型谎报置信度、或损坏给会计师的建议。沿用旧系统「分类器不是 Agent、LLM 只在代码备好的上下文里做一次判断、硬阻断由代码处理」的已验证边界，但去掉其按 (pattern, direction) 索引与 risk-pack 的具体实现。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:决策权限 / 代码与模型分工` → `Case Judgment 是代码驱动流程，模型是被调用一次的判断工具，不是 agent。顺序：代码（触发判定 + 上下文组装 + 读所有确定性字段算出本笔最多能到哪一步 + 备好结构化约束）→ 模型（框内一次判断：主体 / 性质、COA、HST、证据充分性、reasoning、自评置信度；不获取信息、不调工具、不决定控制流）→ 代码（裁定路由 + 校验 + 转交 / 写入）。置信度与许可分离：模型答"是什么 + 多确定"，代码答"系统能否自动落地"。最终路由由代码裁定：存在硬阻断则无论置信度多高一律 Pending；无硬阻断才采信模型置信度放行。模型不持路由权。被阻断的交易仍可让模型给建议附于 Pending；告知模型"将交人工"只为塑造输出形态，不为压置信度。硬阻断必须由结构化字段确定性可算，需语义解读的降级为 soft risk。默认单次模型调用；多次调用须由代码确定性条件编排，模型不得自决是否再调。`

**排除的替代 + 理由：**

- 用「在上下文里指令模型不得高置信度」来守硬阻断。理由：能否生效取决于模型是否听话，长上下文下正是要避免的失败模式；硬保证必须落在代码。
- 用「模型若输出高置信度就打回重想」来守硬阻断。理由：(1) 会改坏模型本可给会计师的高质量建议——许可该是 Pending 与判断准不准无关；(2) 会把模型训练成"为过闸门而谎报低置信度"，使最该信任的置信度信号失真。
- 把 Case Judgment 实现为模型自主多步 / agent loop（由模型决定是否再调用、再取数）。理由：会把控制流决策权交回模型，违反「模型不是 agent、不做多步推理」的铁律。

**浮现的新 open boundary：**

- 各上游 / 记忆层用于硬阻断判定的结构化字段（enum / flag）尚未设计；硬阻断清单待这些字段成形后冻结，属未来硬标准。
- 上下文的注入 / 缓存 / 排序的 exact 机制，以及多次模型调用（如有）的 exact 编排，属 L4 / seam。
- Pending 是否携带备选、以及何时纯交人，属模型基于全部证据的语义判断，不由代码或本轮讨论预设，本轮不展开。

### 2.15 Pending 的承接、reason 的定位与下游机制

**结论：**

**单一 consumer**：Case Judgment 输出的 Pending 唯一承接者是 Coordinator。本节点不向多个下游分流 Pending，也不依赖 reason 做下游分流。

**reason 的定位**：Pending 的 reason 只解释「模型为什么输出这个结果」——为什么不确定是哪个分类，或为什么完全无法判断。它是模型思维链（CoT）精华的压缩，是 2.10 结构化论证链用在 Pending 上的特例，会自然定位出判断断在哪一环（身份 / 凭据 / 历史 / COA / HST 中的哪一处）。reason 不开「需要补什么才能解决」的药方，也不负责把问题结构化成会计师可读的提问——那是 Coordinator 的职责。

**备选的产出与呈现**：当模型基于全部证据语义判断「存在有参考价值的候选分类」时，候选分类内容由**模型**产出（模型已加载全部上下文并完成分析），随 Pending 一并交出；Coordinator 负责把 reason + 候选 + 交易 / entity 信息**组织**成更全面、可读性更强的会计师提问（如选项 1/2/3 / Other），不重做分析。是否给候选、给几个，属模型语义判断（见 2.14 open boundary），不由代码或本节点预设。

**下游机制（沿用，不在本节点冻结其内部）：**

1. Coordinator 据 reason + 交易 / entity 信息向会计师提问并收到回复。
2. 会计师给出确认性回复、且 Coordinator 判定信息足以得出准确会计分类 → Coordinator 将其转成 JE 所需格式，调用 JE Generation Node 生成结果。
3. 信息仍不清晰或会计师回答不确定 → Coordinator 基于已配备的全部交易 / entity 信息进一步追问。
4. 会计师要求对 entity 做特殊处理（如「以后不可高置信度分类」）→ Coordinator 收到信号后产出 candidate signal，由规则文档维护者读取并在对应文档落更。

**为什么（锚定产品目标）：**
accountant control、审计性、自动化率。单一 consumer + reason 只解释，避免把跨节点路由状态机过早冻结，也避免 CJ 越界去做 Coordinator 的「组织提问」职责；reason 复用 2.10 的论证链并定位断点，使 Coordinator 能据此精准提问，而 CJ 无需开补救药方；候选由模型产出、Coordinator 呈现，避免 Coordinator 在上下文不全时重做分析。该下游机制与 2.8 一致——entity / governance 类 candidate signal 从「会计师交互」路径产生，而非 CJ 单次运行直接断言。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:Pending 行为 / 下游契约` → `Case Judgment 的 Pending 唯一 consumer 是 Coordinator，不做多路分流。reason 只解释模型为何输出该 Pending（为何不确定是哪个分类 / 为何无法判断），是 2.10 论证链在 Pending 上的特例，用于定位判断断点；reason 不开补救药方、不负责把问题结构化成会计师提问（属 Coordinator）。模型语义判断有价值候选时，候选由模型随 Pending 产出，Coordinator 负责组织成可读提问（选项 1/2/3 / Other），不重做分析。下游：Coordinator 向会计师提问 → 确认且信息足够则转 JE 格式调 JE Generation Node；不够则追问；会计师要求 entity 特殊处理则 Coordinator 产出 candidate signal 交规则文档维护者落更。`

**排除的替代 + 理由：**
为 Pending 设计一套用于下游分流的 reason 语义枚举。理由：Pending 只有 Coordinator 一个 consumer，不存在按 reason 分流的需求；reason 的职责是解释模型为何输出该结果（并定位断点供 Coordinator 提问），不是路由键。把它做成路由枚举会凭空引入一台并不存在的跨节点路由状态机。

**浮现的新 open boundary：**
Coordinator 内部的提问模板、追问策略、把 reason / 候选 / 交易信息组织成会计师提问的 exact 机制，以及会计师确认后转 JE 格式的 exact contract，属 Coordinator 设计，挂起。

### 2.16 Case Log 先例的读取：代码只递客观事实，CJ 当辅助证据权衡

**结论：**
当 CJ 收到一笔 stable entity 的交易、且该 entity 在 Case Log 中已有历史记录时，对历史先例的处理遵循以下原则。

**(1) 代码把 per-entity case 摘要压成一组中性、客观的事实，不下结论、不贴标签。**
由 Case Log 读取层（rollup）将该 entity 的历史 case 压成确定性事实陈述递给 CJ，至少包括：
- 出现次数；
- 历史分类分布：是否唯一一致，或如实列出几种分类及各自次数；
- `confirmed_by` 构成：其中几次经会计师确认、几次为系统判定；
- 历史金额区间与本笔金额；
- 历史资金方向与本笔方向；
- （可选）最近一次出现时间。

代码**不得**输出「强先例 / 弱先例 / 不可信 / 不适用」这类带判断的结论词——这类标签会锚定并干扰模型输出，且把先例抬到「决定性」的位置。事实本身已携带信号（如「8 次全为 X、含会计师确认、金额方向皆一致」自然指向可重用；「3 种分类各占一些」自然表明该 entity 多用途、不能单凭历史定类），无需代码代为判断。

这组事实在认知上回答两件事，但都以事实形态呈现、由模型自行权衡：先例**可信度**（次数 + 分类一致性 + 确认来源）与对本笔的**适用性**（金额区间 + 方向是否一致）。

**(2) Case Log 先例只是 CJ 手上众多证据之一，辅助而非裁决。**
先例摘要与当前小票、descriptor、通用知识、Profile 等并列，是一种**快捷复用证据**。最终分类由 CJ 综合当前这笔交易的**全部证据**做出：
- 先例只辅助、不单独裁决；
- 当前证据与先例冲突时，按 2.12 当前证据优先，先例让位；
- 先例再一致，也不能把一笔当前证据矛盾或不足的交易单独推成 High Confidence；
- 尤其纯系统判定（无会计师确认）的重复历史，不能自我累积成「强证据」——`confirmed_by` 构成作为事实呈现，正是为防止 AI 用自己未经验证的历史给自己壮胆。

**(3) 守住「质量是事实、权限是硬门」的边界。**
先例的**质量**（次数 / 分类分布 / 确认来源 / 金额 / 方向）一律作为客观事实交模型权衡；但先例的**权限封顶**——Case Log `use_level` 标记 must-ask / exception-only / 已失效、存在 negative case、或 Entity Log `automation_policy` 收紧——属会计师 / 治理的硬决定，仍由代码确定性短路成 Pending（见 2.5 / 2.14），不作为可被证据权衡的事实下放给模型。

**为什么（锚定产品目标）：**
记忆复用、自动化率、审计性、accountant control。把维度计算前置进代码、只递客观事实，使「老客户老套路、无小票」的交易能借历史快捷分类、不被迫全进 Pending（自动化率），同时模型 CoT 短、上下文小，降低长上下文幻觉。以客观事实替代结论标签，避免代码用带倾向的词误导模型，并让借界情况（如金额略超历史区间）由模型结合全部证据细判，而非被一个二值标签粗暴决定。`confirmed_by` 构成作为事实呈现防止置信度洗白 / 错误滚雪球；use_level / policy 硬门仍由代码守，保住 accountant control 与审计性。先例只辅助不裁决，守住「Case Log 是辅助先例、不是确定性规则源」的已锁边界（见 1. 依赖、2.2）。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:读取对象 / 决策权限` → `对 stable entity 的历史先例：Case Log 读取层把 per-entity case 摘要压成中性客观事实（出现次数、历史分类分布是否一致、confirmed_by 构成、历史金额区间 vs 本笔、历史方向 vs 本笔）递给 CJ，不输出强/弱/可信/不适用等判断标签。该事实集是 CJ 众多证据之一，辅助而非裁决；最终分类由 CJ 综合当前全部证据判定，当前证据与先例冲突时当前证据优先（见冲突处理），先例不单独把证据矛盾 / 不足的交易推成 High Confidence，纯系统判定的重复历史不自我累积成强证据。先例的 use_level must-ask / exception-only / 失效、negative case、automation_policy 收紧属硬权限，由代码短路成 Pending，不作为可权衡事实下放。`

**排除的替代 + 理由：**

- 代码直接输出「强先例 / 不可信 / 不适用」等结论标签。理由：带倾向的语义词会锚定并干扰模型输出，把先例抬成决定性证据；客观事实本身已携带信号，标签多余且有害，且对借界情况（金额略超区间）的处理反而比交模型细判更差。
- 把「数次数、比金额、判一致性」的多维分析逻辑写进 LLM 的 CoT。理由：增加上下文长度与多步推理、抬高幻觉风险；这些是确定性可由代码算的事实，应前置由代码完成。
- 让先例（尤其纯系统确认的重复历史）单独构成 High Confidence 依据。理由：置信度洗白 / 错误滚雪球；先例只辅助，当前证据可推翻，且权威性不及经人工审批的 Rule。

**浮现的新 open boundary：**

- 客观事实摘要的 exact 字段 / 呈现形态、以及次数 / 确认数 / 金额容差等阈值 → L3。
- per-entity rollup 如何从逐条 case 算出这组事实（含是否为多用途 entity 建更窄检索 key）→ Case Log 读取层 L4 / seam（已在缺口地图）。

## 3. 分类备案

- `case_judgment_input` / `case_judgment_result` exact field schema、status enum → L3
- accounting treatment 字段结构与 JE Generation Node 的对齐 → L3
- Pending handoff（pending_request_context）exact field schema → L3
- Entity Log `automation_policy` / Case Log `use_level` / `confirmed_by` 的 exact enum 取值 → 分别留 Entity Log / Case Log L3
- 固定加载知识的注入 / 缓存 / retrieve 机制 → L4 / seam
- Case Log per-entity rollup / retrieval pack 生成机制 → L4 / seam（属 Case Log 读取层）
- entity-level Knowledge Summary 注入时具体 co-read 哪些 source memory → L2·外阻 / seam（依赖 Knowledge Summary，挂起）
- Coordinator / Pending Node 交互机制与提问模板 → L2·外阻 / seam（依赖 Coordinator，挂起）

## 4. 自检与判定

- **本对象 L2 可否判定完成**：当前 Q1–Q13、O1、O2 收口（输入加载分层、无先例自动判断、身份约束、两类输出、自动进 Pending 触发、JE Generator 定位、Pending handoff 语义契约、candidate signals 范围、High Confidence 后的路由、Reasoning / Audit Trace 形态、mixed-use 等 soft risk 处理、多源冲突优先级、external lookup 边界、代码与模型分工及路由机制、Pending 承接与 reason 定位、Case Log 先例读取与权衡）。是否已覆盖本节点全部 L2 主题，见下方「L2 覆盖度复检」。
- **能否进 Stage 3**：否。待下方「L2 覆盖度复检」中仍开放的 L2 主题收口、且字段级 schema 仍留 L3 后再判定。

### L2 覆盖度复检（对照模板 22 项 Stage 1-2 清单 + 第一性原理）

已覆盖：代码 / 模型分工与执行顺序(10/11, O1)、上游依赖(4)、下游影响(5)、读取对象(6, 含 Q1/Q13/G1)、直接写什么(7, =无)、candidate(8)、绝不能写(9)、证据不足(14, Q2/Q5)、Case Log 先例读取与权衡(14/16, G1)、歧义(15, Q12)、source conflict(16, Q12)、trace 保留(17, Q10)、trace 不成 authority(18, Q10)、对外契约面与 consumer(19, Q4/Q6/Q7/Q8/Q9/O2)、运行/记忆 seam(20, Q8)、未冻结项(21)、能否进 Stage 3(22)。

**仍开放、需讨论的 L2 主题：**

- **O3 · 触发条件与无效 / 被阻断 handoff 的行为（清单触发 / 上游前置）**：CJ 何时运行（非结构性 + deterministic path 未完成，已散见依赖）需成文；以及收到无效输入（缺 transaction_id、结构性路径已完成、preprocessing 未完成、缺 ER 身份结果）时是拒绝、回退还是 stop-and-ask，尚未锁定。

**已决定、只待并入正式文档（无需新讨论）：**

- **C1 · Accountant / Governance authority boundary（清单 12/13）**：CJ 不做最终会计结论、不把 runtime judgment 升格为 durable confirmation；entity merge/split、rule promotion、automation policy 升级、governance event 须 governance/accountant。结论散见 Q3/Q8/Q11/Q12，需在正式文档汇成一节。
- **C2 · Stage 1 Functional Intent（清单 1/2/3）**：为什么必须存在、为什么不可删除/合并/内联、唯一核心职责、明确排除范围、对产品目标的贡献。内容已在讨论中确立，待按 Stage 1 模板成文。
- **C3 · CJ 拥有 COA + HST treatment 选择权**：Q2/Q6 已隐含（CJ 定 (COA, HST)，JE Generation 仅确定性构造），正式文档显式确认即可。
