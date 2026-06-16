# Case Judgment L2 提案

> 进行中。本提案逐主题收口；当前已讨论确认 Q1（输入加载分层）、Q2（无先例时的自动判断）、Q3（Entity Resolution 输出对本节点的身份约束）。其余 L2 主题（pending 与 review-required 边界、hard block 清单、accounting treatment 完整度、candidate signals、source conflict 优先级等）待续。

## 1. 依赖

- **上游身份（已锁）**：Entity Resolution 对下游只声明 `stable` / `unknown` 两种身份状态（`BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md`）。`unknown` 的下游影响已锁为「Case Judgment 不走高置信度自动分类通道，输出 pending」。
- **上游证据（已锁）**：Evidence Intake / Preprocessing 提供可追溯 evidence 与 evidence refs，并分配稳定 `transaction_id`。
- **上游结构（已锁）**：Profile / Structural Match 已确认当前交易未被结构性路径完成；identity-independent 的结构性交易（如 internal transfer、bank fee、interest）主要由此路径处理。
- **Case Log（已锁，下游记忆层）**：Case Log 按 `entity_id` 组织 completed-case precedent；仅 stable entity 可作为 case identity handle，unknown 不可（`BK_Copilot/memory_layers/case_log/02_authority_lifecycle_and_boundaries.md`）。Case Log 是辅助先例，不是确定性规则源。
- **entity-level Knowledge Summary（已锁，下游记忆层）**：按 `entity_id` 锁定后按需注入 Case Judgment，自带 authority / usage 护栏（`BK_Copilot/memory_layers/case_log/01_memory_intent.md`）。
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

### 2.2 无先例时的 operational classification

**结论：**
Case Log 是辅助先例，不是 operational classification 的必要条件；缺少历史先例不构成自动阻断。

是否允许进入 operational classification，取决于一个复合落地条件：当前 evidence 结合常驻知识，能否唯一、确信地落到单一站得住的 (COA 科目, HST/GST 处理) 上。具体为：
- 交易主体 / 性质清晰；
- COA 中只有一个合理科目；
- HST/GST 处理方式可确定（非 unknown）；
- 不存在未解决的 authority / identity / evidence / governance 阻断。

满足上述条件即可进入 operational classification，无论是否存在 Case Log 先例。先例存在时用于加强与校验该判断，但不能替代当前证据，也不能升格为确定性规则。

**为什么（锚定产品目标）：**
自动化率、有证据支持的建议、记忆复用。若把先例设为自动判断的必要条件，会让大量证据本已充分、但属于该 entity 首次出现的交易被迫进入 pending，压低自动化率；把闸门定义为「能否唯一落地 (COA, HST)」而非「有无先例」，使判断锚定当前证据强度，同时保持 Case Log 作为先例的辅助与校验地位、不被升格为规则。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:决策权限 / operational classification 条件` → `Case Log precedent 是辅助先例，不是 operational classification 的必要条件；无先例不构成自动阻断。进入 operational classification 的复合落地条件：当前 evidence 结合常驻知识可唯一、确信地落到单一站得住的 (COA 科目, HST/GST 处理)：交易主体 / 性质清晰、COA 中只有一个合理科目、HST/GST 可确定（非 unknown）、不存在未解决的 authority / identity / evidence / governance 阻断。先例存在时加强与校验判断，不替代当前证据，不升格为确定性规则。`

**排除的替代 + 理由：**
把 Case Log 先例设为 operational classification 的必要前提。理由是这会让首次出现、但证据充分的交易被迫 pending，削弱自动化率，并使 Case Log 从「辅助先例」事实上升格为准入门槛。

### 2.3 Entity Resolution 输出对 Case Judgment 的身份约束

**结论：**
Entity Resolution 对下游只声明两种身份状态：`stable` 与 `unknown`。Case Judgment 据此确定身份层的处理边界：

- `stable` 是 operational classification 的必要前提。身份为 `stable` 时，Case Judgment 获得可用身份锚点，可据此按 `entity_id` 读取 Case Log，并在 2.2 落地条件成立、无硬阻断时进入 operational classification。`stable` 的 provenance（复用既有 / 新建）不改变其作为身份基础的效力。
- `unknown` 一律不进入 operational classification。Case Judgment 输出 pending；可利用非身份上下文（receipt items、金额模式、bank descriptor 特征等）给出推断性建议，附于 pending，交由下游向会计师确认。

identity-independent 的交易（如 bank fee、interest）由上游 Profile / Structural Match 处理，不构成 Case Judgment 在 `unknown` 下自动分类的例外。

**为什么（锚定产品目标）：**
审计性、accountant control、自动化率。身份必要前提保证自动会计分录建立在可追溯、稳定的身份之上；把 `unknown` 一律收敛为 pending（可附推断建议），既不伪造身份、不绕过会计师确认，又不浪费已有的非身份上下文。该约束直接沿用 Entity Resolution 正式文档已锁结论，保持两节点契约一致。

**拟改：**

- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:决策权限 / 身份约束` → `Case Judgment 消费 Entity Resolution 的两态身份输出。stable 是 operational classification 的必要前提：stable 时获得可用身份锚点，可按 entity_id 读取 Case Log，并在 operational classification 落地条件成立、无硬阻断时进入 operational classification；provenance（复用 / 新建）不改变身份基础效力。unknown 一律不进入 operational classification，输出 pending，可附基于非身份上下文的推断性建议交下游向会计师确认。`
- `BK_Copilot/workflow_nodes/case_judgment_node/02_logic_and_boundaries.md:边界` → `identity-independent 的交易（如 bank fee、interest）由上游 Profile / Structural Match 处理，不构成 Case Judgment 在 unknown 下自动分类的例外。`

**排除的替代 + 理由：**
在 Case Judgment 层为 `unknown` 身份保留 identity-independent 自动分类旁路。理由是 Entity Resolution 正式文档已将该类交易归上游结构路径处理，并明确「到达 Case Judgment 的交易绝大多数需要 entity identity」；在本节点放行会与上游结构路径抢边界，且削弱「自动分类必须基于稳定身份」这一已锁约束。

## 3. 分类备案

- `case_judgment_input` / `case_judgment_result` exact field schema、status enum → L3
- accounting treatment 字段结构与 JE / GE Generator 的对齐 → L3
- 固定加载知识的注入 / 缓存 / retrieve 机制 → L4 / seam
- Case Log per-entity rollup / retrieval pack 生成机制 → L4 / seam（属 Case Log 读取层）
- entity-level Knowledge Summary 注入时具体 co-read 哪些 source memory → L2·外阻 / seam（依赖 Knowledge Summary，挂起）

## 4. 自检与判定

- **本对象 L2 可否判定完成**：否。当前仅 Q1（输入加载分层）、Q2（无先例自动判断）、Q3（身份约束）三个主题收口；pending 与 review-required 边界、hard block 清单、accounting treatment 完整度、candidate signals 与 consumer、source conflict 优先级等 L2 主题尚未讨论。
- **能否进 Stage 3**：否。待上述 L2 主题收口、且字段级 schema 仍留 L3 后再判定。
