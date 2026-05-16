# entity的作用和在系统中的创建方法

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
- 后续可能新增的 `Case Log` 文档

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
- `John Smith Plumbing`、`John Smith Consulting` 指向同一个 entity：John Smith。它们的区别是 role，不是 identity。

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

### 3. 支持 alias、role、rule 的后续治理

Entity 可以承载或关联：

- alias
- role
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
- 这个对象可以进入后续 alias、role、rule、automation 的治理流程。

它不表示：

- 这笔交易一定进某个 COA。
- HST / GST 一定能判断。
- 可以自动落账。
- 已经有 approved rule。
- 当前 alias 已经全部被批准。
- 当前 role 已经被确认。

因此不建议使用 `rule entity` 这个说法。

更清楚的说法是：

- candidate entity
- stable entity
- entity with approved rule
- entity with confirmed role
- entity with automation policy

## Entity 和 Role 的关系

### Role 的含义

Role 是一个 entity 对当前客户而言的身份或业务关系。

通俗地说：这个对象对我来说是什么。

Role 往往和最终会计分类（COA）直接相关。

### Entity → Role 的推断何时成立

对于单一 role 的 vendor，知道 entity 几乎等于知道 role：

- 市政府电力公司 → 缴电费 → Utilities
- Rogers Wireless → 手机账单 → Telephone / Communications
- 保险公司 → 保险费 → Insurance

这类 entity 在小企业记账中占较大比例。

### Entity → Role 的推断何时不成立

三种情况下，知道 entity 不等于知道 role：

#### 同一个 entity 有多个 role

- John Smith 可以同时是 plumber（Repairs & Maintenance）、consultant（Professional Fees）、landlord（Rent）。
- 知道 entity 是 John Smith，不能自动推断当前交易的 role。

#### Role 取决于交易具体内容而非 entity 本身

- Amazon 可以是 Office Supplies、Computer Equipment、Packaging Materials、Books。
- Costco、Canadian Tire、Home Depot 同理。
- 这类综合零售商 / 平台类 entity 的 role 取决于 receipt items，不取决于 entity identity。

#### Entity stable 但 role 的证据来源不同

- Entity stable 的证据：bank descriptor、receipt vendor name、invoice party name。
- Role 判断的证据：receipt items、invoice line items、交易金额模式、历史分类、accountant context。
- 这两种证据可以独立存在。

### 结论

- Entity 和 role 在部分场景下高度相关，在另一部分场景下是独立判断。
- Role 判断不应被视为 entity identity 的附属品。
- 对于多 role entity 和综合零售商，role 判断属于 Case Judgment 的职责，需要依赖 receipt items 或其他交易上下文。
- 当前系统设计中 `resolved_entity_with_unconfirmed_role` 这个输出类别是有必要的，不是边缘情况。

## Entity 类型命名

当前统一使用三类命名：

- `stable entity`
- `candidate entity`
- `unknown entity`

其中：

- `stable entity` 是身份权威。
- `candidate entity` 是可引用的候选身份。
- `unknown entity` 不是 durable entity record，只是当前交易无法形成可用 entity 的 runtime identity status。

## Candidate entity 的作用

Candidate entity 是持久身份线索。

它不是身份权威。

它存在的原因是：

- 新对象可能第一次出现。
- 系统需要先给它一个可引用的身份 handle。
- Case Log 不能自己保存一堆候选身份字段。
- 后续治理需要知道这些案例当时指向哪个候选对象。

Candidate entity 可以被 Case Log 引用。

但 candidate-linked case 默认只能作为弱上下文。

它不能直接支持：

- rule match
- rule promotion
- approved alias
- confirmed role
- automation 放宽
- 强 precedent

## Unknown entity 的作用

Unknown entity 表示：

> 当前交易需要 entity identity，但证据不足以形成 stable entity 或 candidate entity。

它不应写入 Entity Log。

它不应被 Case Log 当作 identity handle 引用。

它只用于说明当前交易缺少可复用身份基础。

## Stable entity 的判断标准

核心标准只有一个：

> 当前证据能不能直接、清楚、可追溯地说明这个对象是谁？

如果能，并且没有明显身份冲突，就可以考虑 stable。

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

### 不能自动 stable 的情况

如果出现明显身份冲突，不应自动 stable。

例子：

- 同一 surface text 可能对应多个已有 entity。
- 命中 rejected alias。
- 描述太泛，例如 `TRANSFER`、`PAYMENT`、`DEPOSIT`。
- 看起来是内部转账，但账户关系不清楚。
- 看起来涉及 payroll、loan、tax、owner、employee、related party，但身份关系未确认。

这些情况应进入候选、人工审核或更保守的治理路径。

### 仍需继续讨论的细节

还没有完全决定：

- 清晰 bank descriptor 是否单独足够。
- 第一次出现的普通 vendor 是否默认可以自动 stable。
- 哪些 entity type 必须人工 review。
- 什么程度算“明显身份冲突”。
- rejected alias 遇到新证据时怎么处理。

## Entity 在系统中的创建方法

### 当前状态

当前已确认：

- Entity Resolution 负责判断当前交易属于哪一类 entity。
- Entity Log 只存 stable entity，没有 durable candidate entity lifecycle state。
- Entity Resolution 可以直接将 `new_stable_entity` 同步写入 Entity Log，不需要 governance approval。
- 写入内容限于 entity 本体（entity_id、display_name、entity_type、entity_status=active、evidence_links、created_from）。
- Approved alias、confirmed role、automation policy 等更高 authority 字段不由 Entity Resolution 写入，由后续流程处理。
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

它负责把当前交易输出为五类之一：

- `matched_stable_entity`
- `matched_candidate_entity`
- `new_stable_entity`
- `new_candidate_entity`
- `unknown_entity`

它可以使用：

- bank descriptor
- receipt vendor
- invoice party
- cheque payee
- contract party
- Entity Log
- approved alias
- rejected alias

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
- Entity Log 不存在 durable candidate entity lifecycle state。

仍未决定：

- 写入 stable entity 时，当前 surface text 作为 alias 的处理方式（approved / candidate / 不写入）。
- 写入 stable entity 时，role 的处理方式。
- Approved alias、confirmed role、automation policy 的写入时机和责任归属。

### 4. Rule Match 关系

`matched_stable_entity` 可以进入 Rule Match，但只有在该已有 stable entity 下存在 active rule 时才可能 rule-handled。

`new_stable_entity` 不会命中 Rule Match，因为它是新 entity，不可能已有 active rule。

`matched_candidate_entity` 和 `new_candidate_entity` 不能支持 Rule Match。

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
- candidate entity

但两者的 authority 不同。

stable-linked case 可以成为更强的未来上下文。

candidate-linked case 默认只能是弱上下文或治理证据。

candidate 后来升级为 stable 时，旧 case 是否自动变强，还没有决定。

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

Candidate entity 不是 authority。

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

### 2. Alias 的默认状态

创建 stable entity 时，当前 surface text 是否自动成为 approved alias？

不同来源是否权威不同？

例如：

- receipt vendor
- invoice vendor
- cheque payee
- bank descriptor

### 3. Role 的确认规则

能识别“是谁”，是否等于知道它在客户业务里的 role？

当前倾向：

不是。

Stable identity 和 confirmed role 应该分开。

### 4. Candidate entity 的最小字段

需要决定：

- 是否存在于 Entity Log 本体。
- 是否共享 entity_id namespace。
- 最小字段有哪些。
- 何时清理、合并、归档。
- 是否可以被多个 Case Log 记录引用。

### 5. Candidate -> stable 谁来执行

还需要决定：

- Entity Resolution 是否只输出判断。
- Memory Update 是否负责写入。
- Governance Intake 是否负责高风险场景。
- 是否存在自动 stable 的同步路径。

### 6. Stable entity 默认 automation policy

当前确认：

stable entity 不应自动放宽 automation。

但默认 policy 名称还没定。

可能方向：

- identity_only
- case_allowed_but_no_promotion
- review_required

### 7. 正式文档更新范围

完成 Entity 讨论后，需要更新：

- Entity Log M1-M2
- Entity Resolution Node Stage 1-2
- Case Log M1-M2
- 可能受影响的 Rule / Review / Governance 边界