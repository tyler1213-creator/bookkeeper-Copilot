# Entity相关的所有问题

## 文件角色

这是临时设计记录。

它只记录当前已经讨论到的 Entity 相关判断。

它不是正式 spec。

它不是已冻结决策。

等 Entity 相关讨论完成后，需要把这里的内容更新到对应正式文档，并修改所有受影响的文档。

可能受影响的文档包括：

- `BK_Copilot/memory_layers/entity_log/01_memory_intent.md`
- `BK_Copilot/memory_layers/entity_log/02_authority_lifecycle_and_boundaries.md`
- `BK_Copilot/workflow_nodes/entity_resolution_node/01_functional_intent.md`
- `BK_Copilot/workflow_nodes/entity_resolution_node/02_logic_and_boundaries.md`
- `BK_Copilot/memory_layers/case_log/00_index.md`
- `BK_Copilot/memory_layers/case_log/01_memory_intent.md`
- `BK_Copilot/memory_layers/case_log/02_authority_lifecycle_and_boundaries.md`

## 一句话结论

Entity 的核心作用是回答：

> 这笔交易对应的业务对象是谁？

Stable entity 的核心作用是回答：

> 这个对象已经足够清楚，未来交易可以继续归到它下面。

它不是分类规则。

它不是自动记账权限。

它不是 COA / HST / JE 判断。

## Entity 的身份粒度

Entity 是业务对象层面的语义身份，不是字段精确匹配。

例如：

- `AMZN MKTP CA*2X7Y9Z`、`Amazon.ca`、`AMAZON.CA` 都指向同一个 entity：Amazon。
- `HOME DEPOT #4521`、`HOME DEPOT #3890` 都指向同一个 entity：Home Depot。
- 同一业务对象的不同表面写法可以指向同一个 entity；具体会计处理不由 entity identity 本身决定。

Entity Resolution 使用 LLM 语义理解来判断业务对象身份，而不是做字符串精确匹配。

## Entity Resolution 可用工具

Entity Resolution 可以使用 AI 联网搜索来辅助判断 entity identity。

典型场景：

- bank descriptor 只包含支付平台名称（例如 `SQUARE *COFFEE SHOP`），真正的 vendor 需要通过搜索来确认。
- 缩写或不完整的 vendor 名称需要通过外部信息确认。

使用边界：

- 联网搜索只用于辅助 identity 判断（这是谁），不用于会计分类判断。
- 搜索结果是 evidence 的一种来源，不是 authority。
- 如果搜索仍不能确认身份，应按正常的 unresolved / ambiguous 路径处理。

设计依据：

- Accountant 在遇到无法识别的 entity 时，通常也是通过搜索来获取答案。
- 在针对特定目标做搜索这个任务上，AI 的能力不弱于人类。
- 让 AI 先搜索可以减少不必要的 accountant pending 问题量。

## Entity 在系统中的作用

### 1. 提供长期身份索引

Entity 是系统长期记忆的身份锚点。

它替代旧系统中 pattern 的一部分作用，但不等于 pattern。

旧系统的 pattern 主要是标准化描述。

新系统的 entity 是带生命周期的业务对象。

### 2. 支持历史案例复用

Case Log 应该围绕 entity 组织历史案例。

这样系统以后看到同一个对象时，可以读取过去的案例、例外和纠正记录。

### 3. 支持 Alias、rule 和 automation 的后续治理

Entity 可以承载或关联：

- Alias
- case history
- rule candidate
- automation policy
- merge / split 历史

但这些能力不等于 stable entity 自动拥有它们。

### 4. 保护系统不要把身份和记账判断混在一起

Entity 只处理“是谁”。

Case Judgment 才处理“这笔交易怎么记”。

Rule Log 才保存已批准的确定性规则。

Transaction Log 才保存最终审计记录。

## Stable entity 的作用

Stable entity 是身份权威。

它表示：

- 系统知道这个对象是谁。
- 后续交易可以安全归到这个对象下面。
- 这个对象可以作为 Case Log 的稳定索引。
- 这个对象可以进入后续 Alias、rule、automation 的治理流程。

它不表示：

- 这笔交易一定进某个 COA。
- HST / GST 一定能判断。
- 可以自动落账。
- 已经有 approved rule。
- 当前交易 surface text 已经进入 Alias 库。

因此不建议使用 `rule entity` 这个说法。

更清楚的说法是：

- stable entity
- entity with approved rule
- entity with automation policy

## Alias 当前定义（已确认）

Alias 的核心作用，是辅助 Entity Resolution 判断当前交易的主体是谁。

在当前收窄后的定义下，Alias 是过去已经确认过的 transaction surface text 和 stable entity 的对应关系：

```text
已确认 Alias surface text -> stable entity
```

当前只确认两类信息可能成为 Alias：

1. bank statement 中每笔交易的 description / descriptor / raw bank surface text。
2. 当 bank description 本身没有明确身份意义时，其他可能重复出现并能指向交易主体的字段，例如 cheque payee。

Alias 不是所有历史 description 的集合。只有已经和某个明确 stable entity 建立过确认关系的 surface text，才属于 Alias。

### Entity Resolution 如何使用 Alias

Entity Resolution 使用 Alias 的典型场景有两种：

1. Entity Resolution 大致能判断当前交易主体，但不能完全确定时，可以以该 entity 为目标查询是否存在一致的 Alias。
2. Entity Resolution 完全无法从当前 evidence 识别主体时，可以用当前 description 或等价 surface text 到 Alias 库中查询是否存在一致或高度类似的历史 Alias。

如果当前 surface text 命中已确认的 Alias，可以复用该 Alias 指向的 stable entity。

一旦通过 Alias 或受控等价匹配确认了 entity，后续系统可以以该 entity 为主体查询 Case Log 等长期记忆。

本节暂不讨论 Alias 对 Rule Match 的支持。

### Alias 库（已确认）

当前确认需要一个 Alias 库。

这里的 Alias 库指一个可被 Entity Resolution 查询的 Alias 集合，保存的是已确认 Alias surface text 到 stable entity 的对应关系。

Alias 库的作用是让系统可以从当前交易的 surface text 反查过去已经确认过的 entity。

它的必要性在于：

- 当 Entity Resolution 已经有一个不完全确定的 entity 判断时，需要查询该 entity 是否有一致 Alias 来辅助确认。
- 当 Entity Resolution 完全识别不出主体时，需要在所有历史 Alias 中查询是否存在一致或高度类似的 surface text，从而反推出对应 stable entity。
- 没有 Alias 库时，系统无法稳定复用过去已经确认过的 transaction surface text，只能依赖当前 evidence 或重新询问 accountant。

Alias 库具体以什么技术形态呈现，暂不在本文讨论；可以在后续技术细节设计时再决定。

## Entity 类型命名

当前统一使用两类命名：

- `stable entity`
- `unknown entity`

其中：

- `stable entity` 是身份权威。
- `unknown entity` 不是 durable entity record，只是当前交易无法形成可用 entity 的 runtime identity status。

## Unknown entity 的作用

Unknown entity 表示：

> 当前交易需要 entity identity，但证据不足以形成 stable entity。

它不应写入 Entity Log。

它不应被 Case Log 当作 identity handle 引用。

它只用于说明当前交易缺少可复用身份基础。

## Stable entity 的判断标准

核心标准已收口：

> 只要当前所有可追溯证据能够直接、清晰且无歧义地说明交易主体是谁，Entity Resolution 就可以将其判定为 stable；否则输出 unknown。

这个判断只回答“这是谁”，不回答会计分类、税务处理、业务用途、Rule 或自动化权限。

### 可以支持 stable 的身份信号

常见直接身份信号包括：

- receipt vendor
- invoice vendor
- cheque payee
- contract party
- 清晰 bank descriptor
- accountant 处理过的历史账本中的 vendor / payee
- accountant 明确确认

这些信号的共同点是：

人可以直接看出业务对象是谁。

例如：

- `HOME DEPOT #4521`
- `ROGERS WIRELESS`
- `Amazon.ca`
- `John Smith Plumbing`

### 不需要证明的内容

升级 stable entity 不需要证明：

- 这笔交易的业务用途。
- receipt items 具体买了什么。
- COA 是否唯一。
- HST / GST 是否可确定。
- 是否可以自动分类。
- 是否可以自动落账。
- 是否可以升级 rule。

这些属于 Case Judgment、Rule 或 Review 的问题。

### Stable 输出 reason

Entity Resolution 输出 stable 时，必须同时输出一条可审计 reason，说明依据哪些 evidence points 判定主体明确。

reason 是凝练后的 evidence-based rationale，不是完整思维链，也不是会计分类理由。

### Exact Alias match

当前 transaction surface text 与 Alias Log 中既有 Alias 完全一致时，Entity Resolution 可以直接输出该 Alias 指向的 stable entity，不再要求 LLM 独立重新判断主体是谁。

Alias exact match 是确定性身份复用路径，不是分类路径。

Alias 写入资格、normalization / equivalence、冲突修正和 supersession 归 Alias Log L2 继续处理。

## Entity 在系统中的创建方法

### 当前状态

当前已确认：

- Entity Resolution 负责判断当前交易属于哪一类 entity。
- Entity Log 只存 stable entity。
- Entity Resolution 可以直接将 `new_stable_entity` 同步写入 Entity Log，不需要 governance approval。
- 写入内容限于 entity 本体（entity_id、display_name、entity_type、entity_status=active、evidence_links、created_from）。
- Alias、automation policy 等 entity 附属信息不由 Entity Resolution 在创建 stable entity 本体时一并写入，由后续流程处理。
- Stable entity 写入 Entity Log 不等于该 entity 拥有 active rule。每一个 rule 必须经过 accountant 审核才可以启用。
- Batch 内交易按顺序处理。Entity Resolution 同步写入 stable entity 后，同 batch 后续交易可以自然匹配。

也就是说：

> Entity Resolution 判断 identity type；如果判断为 new_stable_entity，则同步写入 Entity Log。

### 1. Evidence layer

Evidence layer 负责处理原始材料。

它可以包括：

- file intake
- bank statement parsing
- document understanding
- evidence association
- evidence fact extraction

它负责产出可追溯的 evidence facts，例如：

- bank text
- receipt vendor
- receipt items
- invoice party
- cheque payee
- contract party

它不负责：

- 判断 entity 类型
- 判断 COA
- 判断 HST / GST
- 生成 JE
- 写入 Entity Log

Evidence layer 只提供 entity 判断所需的 evidence facts。

### 2. Entity Resolution

Entity Resolution 判断：

> 这是谁？

它可以使用 LLM 的语义理解能力，但判断目标只限于 identity。

它负责把当前交易输出为三类之一：

- `matched_stable_entity`
- `new_stable_entity`
- `unknown_entity`

它可以使用：

- bank descriptor
- receipt vendor
- invoice party
- cheque payee
- contract party
- Entity Log
- Alias 库

它也可以读取所有与 identity 有关的证据。

它不判断：

- COA
- HST / GST
- receipt items 的业务用途
- 是否高置信分类
- 是否自动落账

它可以判断一个第一次出现的新对象是否已经足够成为 `new_stable_entity`。

但它不能用会计分类稳定性来证明 entity stable。

### 3. Entity 持久化写入

已确认：

- `new_stable_entity` 由 Entity Resolution 同步写入 Entity Log。
- 写入内容限于 entity 本体：entity_id、display_name、entity_type、entity_status=active、evidence_links、created_from。
- 不需要 governance approval 即可写入。
- 写入后，同 batch 后续交易可以自然匹配 `matched_stable_entity`。
- `new_stable_entity` 不等于已有 active rule。Rule 必须经过 accountant 审核才可启用。
- `unknown_entity` 不写入 Entity Log。

仍未决定：

- 写入 stable entity 时，当前 surface text 是否进入 Alias 库。
- Alias、automation policy 的写入时机和责任归属。

### 4. Rule Match 关系

`matched_stable_entity` 可以进入 Rule Match，但只有在该已有 stable entity 下存在 active rule 时才可能 rule-handled。

`new_stable_entity` 不会命中 Rule Match，因为它是新 entity，不可能已有 active rule。

### 5. Case Judgment

Case Judgment 判断：

> 这笔交易怎么记？

它使用：

- resolved entity
- receipt items
- case history
- profile
- accountant notes
- rules

它才处理：

- COA
- HST / GST
- JE
- exception
- review need

### 6. Case Log

Case Log 记录完成案例。

它可以引用：

- stable entity

stable-linked case 可以成为更强的未来上下文。

## 最小 provenance

Stable entity 不需要旧系统那种完整 policy trace。

但它需要最小来源记录。

建议至少记录：

- created_from
- evidence_refs
- matched_surface_text
- created_at
- 是否自动创建

目的不是增加复杂度。

目的是后续可以纠错、合并、拆分和审计。

## 已确认的边界

Entity 不负责记账分类。

Entity Resolution 负责 entity identity type 判断。

Entity Resolution 可以使用 LLM，但只判断 identity，不判断 accounting outcome。

Stable entity 不等于 rule。

Unknown entity 不是 durable entity record。

New stable entity 可以作为当前交易的 stable identity basis。

New stable entity 不等于已有 rule，也不触发 Rule Match 自动处理。

Case Log 不负责审批 entity。

Entity Resolution 不应和 Case Judgment 职责重叠。

Receipt vendor 可以帮助判断 entity。

Receipt items 主要帮助判断这笔交易怎么记。

旧系统高置信分类机制不能整套搬过来。

旧系统可复用的是：

- 直接证据优先。
- 不靠猜测。
- 不稳定 key 不进长期学习层。
- 明确冲突时不自动放行。
- 纠正和例外要分开。

## Case Judgment 与 Entity Identity 的依赖关系（已确认）

### 结论

Case Judgment 的高置信度自动分类通道必须依赖已识别的 entity。如果 Entity Resolution 输出 unknown_entity，Case Judgment 不走高置信度分类通道，但仍然需要处理这笔交易。

### Entity 未识别时 Case Judgment 的处理方式

Entity 未识别不等于 Case Judgment 跳过这笔交易。Case Judgment 仍然处理，但输出为 pending，pending 内部分两种情况：

1. **完全无法判断**：除了 entity 未识别外，也没有其他足够的上下文支持推断。Case Judgment 输出 pending，由 Coordinator 直接向 accountant 提问。
2. **有推断但不确定**：虽然 entity 未识别，但 Case Judgment 可以利用已有的其他上下文（如 receipt items、交易金额模式、bank descriptor 特征等）给出推断性建议。输出仍为 pending，但附带推断结果，由 Coordinator 呈现给 accountant 选择或确认。

核心原则：entity 未识别 → 不自动分类，但尽可能利用已有信息辅助 accountant 决策。

### 推理链

1. 存在不依赖 entity identity 就能判断 COA 的交易类型（bank fee、interest、internal transfer 等），但这些属于 edge case。
2. 其中大部分可由 Profile / Structural Match Node 在上游处理（internal transfer、known loan repayment 等）。Profile 在 Entity Resolution 之前运行。
3. 少量未被 Profile 覆盖的 A 类交易（如未配置的 bank fee），在当前阶段不值得为其在 Case Judgment 中增加特殊路径。
4. 统一规则：entity 未识别 → Case Judgment 不走高置信度自动分类 → 输出 pending（可能附带推断性建议）→ Coordinator 向 accountant 提问或呈现选项。
5. 这消除了"基于不确定 identity 做高置信度分类 → 后续 entity 确认后需要修正"的系统复杂度。

### 对三个子问题的回答

- 是否存在不需要 entity 就能判断 COA 的交易？ → 存在，但属于 edge case，由 Profile 上游处理或按 pending 处理。
- unknown_entity 时 Case Judgment 能否用推测辅助分类？ → 不做高置信度分类，但可以给出推断性建议附在 pending 输出中，供 accountant 选择。
- 基于不确定 identity 的高置信度分类后续如何修正？ → 不存在这个场景，因为不会产生高置信度分类。

### 职责分工

- A/B 分类判断由 Case Judgment 负责，不由 Entity Resolution 负责。Entity Resolution 只管 identity，不判断"这笔交易是否需要 entity identity 才能分类"。
- Case Judgment 收到 unknown_entity 后不走高置信度自动分类通道，但仍然处理这笔交易，根据已有上下文决定是完全无法判断还是可以给出推断性建议。

## Unknown Entity 时 Entity Resolution 给 Case Judgment 的信息边界（已确认）

### 结论

当 Entity Resolution 无法识别出准确 entity 时，可以把所有与 identity 判断相关的证据、线索和判断说明作为 runtime context 传给 Case Judgment。

这些信息的作用是帮助 Case Judgment 理解：

- 当前交易为什么没有 stable identity basis。
- 当前证据中有哪些 human-visible identity clues。
- 是否存在可能的身份解释、相似对象、搜索结果或歧义来源。
- 哪些证据支持或反驳这些解释。
- 当前缺什么信息，才导致不能确认 entity。

这些信息不是 stable entity。

这些信息不是 identity authority。

### 可传递的信息类型

Entity Resolution 可以传递：

- 当前交易的 `transaction_id` 和可追溯 evidence refs。
- bank descriptor、receipt vendor、cheque payee、invoice party、contract party、accountant context refs 等身份相关证据。
- AI 联网搜索得到的 identity 线索；搜索结果是 evidence 来源，不是 authority。
- 可能的身份解释或相似对象。
- 为什么不能确认 stable entity。
- ambiguity reason、missing evidence reason、identity risk 或 Alias 冲突。
- 对下游有帮助的 identity gap 说明。

这些内容应保持为 runtime context / clue / explanation。

它们不能被命名或使用成可复用身份结论。

### Case Judgment 如何使用这些信息

Case Judgment 面对 unknown_entity 时仍然处理这笔交易，但不走高置信度自动分类通道。

Case Judgment 会读取：

- Entity Resolution 传来的 unknown / unresolved identity context。
- 当前交易 evidence，包括 receipt items、金额模式、bank descriptor 特征等。
- Profile / Structural Match 的上游结果。
- Case Log 或 customer knowledge 中与当前上下文相关、但不把 unknown entity 当作 identity handle 的信息。
- Rule / automation / governance / intervention 相关的 authority boundary。

Case Judgment 的职责不是直接向 accountant 提问，而是做 pending 诊断与问题准备。

它需要判断：

1. **完全无法判断**：除了 entity 未识别外，也没有足够上下文支持会计处理推断。
2. **有推断但不确定**：虽然 entity 未识别，但当前 evidence 或其他上下文足以形成非自动化的推断性建议。

Case Judgment 输出 pending 时，应准备：

- pending reason。
- 缺失或不清楚的信息。
- 为什么不能自动分类。
- accountant 需要确认的事实类别。
- 可选的推断性建议，例如可能的 COA / HST / 业务用途。
- 支持这些推断或问题的 evidence / case / risk context。

这些输出只用于 Coordinator 生成 accountant-facing question。

它们不能成为：

- final accounting outcome。
- stable identity basis。
- Case Log authority。
- Rule Match basis。
- durable memory mutation。

### Coordinator 如何使用这些信息

Coordinator 接收 Case Judgment 的 pending 输出后，负责把 pending reason、missing information、question focus 和可选推断性建议整理成人类可读的问题。

Coordinator 可以：

- 向 accountant 询问 counterparty / vendor / payee identity。
- 同时询问业务用途、分类相关信息或是否接受某个推断性建议。
- 在 accountant 回答不足时继续追问。
- 记录 accountant-facing question、answer、clarification 和 unresolved context 到 intervention 语境。

Coordinator 不能：

- 把 Entity Resolution 的身份推测包装成已确认事实。
- 把 accountant 的模糊回答直接写成 stable Entity Log authority。
- 把 pending 交互本身变成 Case Log、Rule Log 或 Governance Log authority。
- 替代 Case Judgment 做自动分类。

### 字段边界

本节确认的是语义边界，不冻结字段 contract。

具体字段名、enum、input/output shape 和 validation rule 属于后续 Stage 3 / Data Contract 或 Coordinator 问题 5 的讨论范围。

## Unknown Entity 后的完整处理路径（已确认）

### 结论

当 Entity Resolution 输出 unknown_entity 时，交易进入 Case Judgment 后输出 pending。Coordinator 根据 pending 子类型设计结构化问题向 accountant 提问。Accountant 的回复在当前 batch 内同步处理。Accountant 明确确认 entity identity 后直接创建 stable entity，不需要 governance approval。交易不重新进入 Entity Resolution，也不重新进入 Case Judgment。

### 路径总览

```
Entity Resolution → unknown_entity
    ↓
Case Judgment → pending（保留两种子类型标记）
    ↓
Coordinator 根据 pending 子类型设计问题
    ↓
accountant 回复（当前 batch 内同步）
    ↓
创建 stable entity + 完成分类（或追问直到充分）
    ↓
写入各 log（机制由未解决问题 7 决定）
```

### 分支 1：有推断但不确定

Case Judgment 有推断性建议时：

1. Case Judgment 输出 pending，附带推断性建议（可能的 COA、HST 处理等）。
2. Coordinator 构建结构化问题：包含 entity identity 问题 + 推断性建议供 accountant 选择或确认。
3. Accountant 一次性回复：确认 entity identity + 确认或修改分类。
4. 系统根据 accountant 确认创建 stable entity，写入 Entity Log。
5. 以确认后的 stable entity 为交易主体，完成分类。
6. 写入 Case Log、Transaction Log 等（机制待未解决问题 7 决定）。

### 分支 2：完全无法判断

Case Judgment 无推断性建议时：

1. Case Judgment 输出 pending，无推断性建议。
2. Coordinator 构建结构化问题：entity identity + 业务用途或分类相关问题（第一轮尽量收集充分信息）。
3. Accountant 回复。
4. 若回复充分：创建 stable entity → 确定分类 → 写入各 log。
5. 若回复不充分：Coordinator 追问（仍在当前 batch 内），重复直到信息充分。

### 关键设计决策

- Batch 时序：accountant 的回复在当前 batch 内同步处理，batch 不跨越等待。
- Stable entity 创建：accountant 的明确身份确认直接创建 stable entity，不需要 governance approval。Accountant 的确认是最强的身份证据。
- 不重新进入 Entity Resolution：accountant 确认替代了 Entity Resolution 的身份判断功能，交易不回头。
- 不重新进入 Case Judgment：无论哪个分支，分类都在 Coordinator 与 accountant 的交互中完成，不回 Case Judgment。
- Stable entity 创建的执行机制：留待未解决问题 7（统一 Memory Write 机制）解决。

### 与问题 2 结论的关系

问题 2 确认了 unknown_entity 时 Case Judgment 的行为（不走高置信度分类，输出 pending）。问题 4 确认了 pending 之后的完整路径：Coordinator 提问 → accountant 回复 → 创建 stable entity → 完成分类。两者构成从 Entity Resolution 输出 unknown_entity 到交易最终完成的完整链路。

## 后续必须继续讨论的问题

### 1. 自动 stable 的边界

第一次出现的新交易，如果身份信号清楚，系统是否可以自动创建 stable entity？

尤其要讨论：

- 普通 vendor
- 清晰 bank descriptor
- receipt vendor
- cheque payee
- invoice vendor
- person / owner / employee
- tax / payroll / loan / transfer

### 2. Alias 写入与 Alias 库技术形态

已确认：

- Alias 是过去已经确认过的 transaction surface text 和 stable entity 的对应关系。
- 当前只确认两类信息可能成为 Alias：bank statement 的 description / descriptor / raw bank surface text；以及 bank description 无明确身份意义时，其他可能重复出现并能指向交易主体的字段，例如 cheque payee。
- 需要一个可被 Entity Resolution 查询的 Alias 库。

仍未决定：

- 创建 stable entity 时，当前 surface text 是否自动进入 Alias 库。
- Alias 库具体以什么技术形态呈现。
- Alias、automation policy 的写入责任和审批路径。

### 3. 独立关系字段是否应该存在

已转入 `未解决问题暂存清单.md` 的问题 8。当前不在本文保留字段定义或确认规则。

### 4. 正式文档更新范围

完成 Entity 讨论后，需要更新：

- Entity Log M1-M2
- Entity Resolution Node Stage 1-2
- Case Log M1-M2（已按当前已确认边界新增；后续 Entity / finalization 问题解决后仍需更新）
- 可能受影响的 Rule / Review / Governance 边界
