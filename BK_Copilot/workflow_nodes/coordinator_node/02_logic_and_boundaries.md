# Coordinator Node - Logic and Boundaries

## 1. 触发条件

本节点在以下条件同时满足时触发：

- 一张 Bank Statement 已由上游处理完毕。
- 该 Bank Statement 对应的 CJ Pending 集合已经就绪。
- Pending 集合需要 accountant runtime 交互才能继续 finalization 或下游打包。

本节点不得在以下情况触发：

- 交易已经 finalized。
- 交易不是 Pending，或 Pending 已经被解决。
- 已经通过 accountant 交互完成确认的交易被再次送回 CJ / ER。
- onboarding 期、材料提交前目标询问、覆盖完整性核对；这些归 interaction_agent / onboarding 路径。

批级触发不等于本节点持有控制流。保证整张表先跑完、多张 Bank Statement 串行 / 并行排序、批起始记忆快照和人机对话呈现均属编排层 / L4 seam。

## 2. 上游前置条件

上游必须已经完成：

- Case Judgment 已输出 Pending，且 Pending 交 Coordinator 作为唯一 consumer。
- Pending handoff 至少携带：客户 / identity 背景、相关 Case Log 先例摘要（仅 stable entity 时存在）、阻塞原因、已执行工作及结果、可选推断性建议与不确定项。
- 当前交易事实和 evidence-foundation 在 runtime handoff 中可用，例如原始 description、金额、日期、方向、account / source refs 等；exact field 留 L3。
- 若 Pending 涉及 stable entity，所需 stable identity 基础已由上游完成可见化。

如果前置条件缺失：

- 本节点行为：不得自行补读 store、猜字段、造 schema 或把缺失信息当作已确认；应保持 Pending 并暴露缺失的 contract / handoff 问题。
- 是否 stop-and-ask：如果缺失来自未冻结契约或圈外对象，停止并要求产品设计决策；实现代理不得自行补全。

## 3. 读取对象

Coordinator 无独立读取面。全部上下文来自 CJ Pending handoff 这一单一在手来源。

| Source | 读取内容 | 用途 | Authority 限制 |
| --- | --- | --- | --- |
| CJ Pending handoff（runtime context，不是 durable store） | 客户 / identity 背景、阻塞原因、已执行工作及结果、可选推断性建议、当前交易事实、evidence refs、仅 stable entity 时存在的 Case Log 先例摘要 | 组织会计师问题、理解答复是否补足卡点、判断信息是否足以生成 JE 或继续 Pending | 在手 context 不等于业务 authority；最终身份与会计处理以 accountant 明确答复为 authority；exact fields 留 L3 |

明确不读取：

- 不直接读取 Entity Log；身份背景由 CJ 在构造 Pending 时读取 Entity Log 并投影进 Pending。
- 不读取 identity risk flags；疑似同名、疑似 merge、identity conflict 等归 Review / Governance / Post-Batch Lint，不归 Coordinator 消费。
- 不读取 Intervention Log；它是交互留痕写入对象，不是运行期判断输入。
- 不直接读取 Case Log、Rule Log、Governance Log、Profile、Knowledge Summary 或 Transaction Log。

## 4. 写入对象

### 直接执行的 durable write（例外）

本节点直接执行的 durable 写入：

- 无。

### 交给记忆 / finalization 层持久化的内容（运行 / 记忆 seam）

本节点产出、但由记忆 / finalization 层执行写入的内容：本节点只声明“存什么 + 谁有权威认定它有效”，不声明“怎么写、谁来写、什么顺序写”。

- 身份确认：当 accountant 明确确认 stable entity 时，本节点发出“stable entity 出现”信号，触发与 ER 共用的统一写入节点处理 Entity Log + Alias Log。有效性来自 accountant explicit identity confirmation；该确认不需要 governance approval，且只确认 identity。
- Case Log 学习记录语义：stable-linked 且 finalized、会计师经 Coordinator 确认身份和分类的交易，可以产生 `confirmed_by=accountant` 的学习记录语义，并必须凭 Transaction Log 或等价 finalization proof。有效性来自 accountant confirmation + finalization proof。
- 交互留痕：本节点与 accountant 的提问、回答、确认、纠正、追问、身份问一次后停止的原因等，形成 Intervention Log 最小 record 语义。有效性来自交互事实；它不是业务 authority。
- 最终交易结果审计：会计师答复足够后形成的交易 finalization 语义应由 Transaction Log 或等价 finalization 机制承接；exact contract 属圈外。

### 只能提出 candidate

- automation policy / rule / case-derived risk / profile / review-governance candidate 等长期记忆或治理风险候选。
- merge / split、identity risk 等身份治理候选只可作为非身份状态的治理 / review candidate 交给对应圈外路径；Coordinator 不裁决、不落地。

### 绝不能写入或修改

- Entity Log 或 Alias Log 的裸写。
- Case Log、Rule Log、Transaction Log、Intervention Log、Governance Log、Profile 或 Knowledge Summary 的裸写。
- active rule、automation policy、governance event、case authority、classification memory。
- 会计分类结论、JE、交易集合或 split 子交易。
- candidate entity、candidate identity 或 stable / unknown 之外的身份状态。

## 5. 决策权限

### Deterministic code 可以决定

- 本节点是否在批级 Pending 集合就绪后被触发。
- Pending handoff 上下文的组装和可用性检查。
- 三出口路由：答复足够转下游、答复不足继续追问 / 保持 Pending、长期风险输出 candidate。
- hard block 是否存在，以及 hard block 不得经普通提问解除。
- accountant 答复不得被本节点直接写入业务记忆。
- 身份问一次后停止的状态是否按未解析身份完成后续路由。

### LLM 可以判断

- 如何把 CJ Pending 的卡点转成会计师可读问题。
- 如何把当前交易辅助证据与会计知识组织成带选项的 pending 问题。
- 会计师答复是否补足卡点、是否足以生成 JE。
- stable entity 已知且仅分类不确定时，如何把同一 entity 的多笔交易聚在一起呈现 / 处理以提高效率。
- 如何草拟一句面向客户的、针对当前交易细节的问题，供会计师人工转发。
- 如何解释还缺什么信息，并继续结构化追问。

### LLM 不能判断

- 最终会计分类、COA / HST / GST 结论或 JE 内容。
- 最终路由许可、hard block 是否可越过、durable write 是否可执行。
- 身份是否 stable，或把模糊回答包装成 stable entity。
- 身份冲突、疑似同名、merge / split 或 identity risk 的运行期裁决。
- 自行落地 Entity / Alias / Case / Rule / policy / Transaction / Intervention durable 变更。
- 把一个 stable entity 的单笔分类答复自动套用到该 entity 名下所有交易。

### Accountant 必须决定

- 当前交易对方 entity 是谁；若不知道 / 查不到 / 不需要，Coordinator 必须接受该边界。
- 最终会计处理，包括 COA、税务、split / allocation、个人 vs 公司立场等。
- 是否确认 Coordinator 提供的选项或纠正其组织方式。
- 是否把草拟的客户问话发给客户、发给谁、何时发；客户回复经 accountant 进入本节点。

### Governance 必须批准

- 长期 authority 变化，例如 rule / policy create、upgrade、relaxation、merge / split、entity lifecycle mutation 等。
- case-derived rule、automation、entity risk candidate 转成 durable authority。
- 身份冲突和极端身份安全 / 证据真实性问题的后续治理处置。

## 6. 输出类别

字段名可以暂不冻结，但语义类别必须稳定。

本表是本节点对下游唯一的契约面：下游只能依赖此处声明的输出类别，不得依赖本节点未声明的内部状态或实现。

| Output Category | 含义 | Consumer（谁消费） | 下游影响 | 不代表什么 |
| --- | --- | --- | --- | --- |
| 转 JE 所需格式 | accountant 答复已经足够，Coordinator 将当前交易的已确认会计处理打包成 JE Generation 可消费的语义输出。 | JE Generation（圈外，L2·外阻） | 下游可尝试确定性构造 JE；exact fields 和 COA 校验 seam 留给 JE Generation / L3。 | 不代表 JE 已生成、Transaction Log 已写入、字段 contract 已冻结，或 Coordinator 自己做了分类判断。 |
| 身份确认信号 | accountant 明确确认当前交易对方是 stable entity，且满足身份确认路径。 | Entity Log + Alias Log 统一写入节点 | 触发统一写入路径创建 / 记录 stable entity 本体、最小创建 provenance 和 alias surface；机制留 L4 / seam。 | 不代表 Coordinator 裸写 Entity / Alias，不代表分类、Rule、policy 或 Case Log 写入。 |
| 继续追问 / 保持 Pending | accountant 答复不足、模糊、冲突，或分类信息尚不足以生成 JE。 | Accountant / runtime 交互通道 | 继续结构化追问，并保持 Pending；分类线持续到 JE-ready。 | 不代表系统可以用 confidence 语言掩盖未解决，也不代表进入独立 review 第三态。 |
| governance / review candidate | accountant 答复暴露长期记忆、policy、rule、case-derived risk、profile 或身份治理风险。 | Governance / Review / Case Memory Update 等圈外路径 | 对应路径可评估是否需要 approval、review 或后续治理。 | 不代表 candidate 已持久化、已批准、已成为身份状态或 durable authority。 |
| Case Log 学习记录语义 | stable-linked finalized 交易经 accountant 确认后，可形成 `confirmed_by=accountant` 的可复用先例语义。 | Case Log 写入机制 / Case Memory Update 或统一 finalization mechanism（圈外机制，L4 / seam） | 后续可凭 finalization proof 评估写入资格。 | 不代表 Coordinator 直接写 Case Log，不适用于 unknown entity 交易，不代表 exact schema 已冻结。 |
| 交互留痕 | 提问、回答、确认、纠正、追问、身份停止原因等最小交互 record 语义。 | Intervention Log（圈外，L2·外阻） | 供系统设计者离线改进系统；不作为运行期判断输入。 | 不代表 Entity / Case / Rule / policy authority，不代表结果审计替代 Transaction Log。 |
| 草拟的客户问话 | 当 accountant 需要问客户或无法自行分类时，Coordinator 用会计知识和当前辅助证据草拟一句面向客户的问题。 | Accountant（人工转发） | accountant 可复制粘贴给客户；客户回复回到 accountant 后，再以 accountant 答复形态进入本节点。 | 不代表问题已发送、客户已回复、Coordinator 直接联系客户、或 Coordinator 给出分类答案 / authority。 |

### 聚合处理

- entity unknown 时，不按 entity 聚合；连对方是谁都不知道时必须逐笔向 accountant 提问。
- stable entity 已知、仅会计分类不确定时，可把同一 stable entity 的多笔交易聚在一起呈现 / 处理，以减少重复上下文切换。
- 聚合呈现不改变逐笔确认要求：每一笔交易的最终分类仍必须由 accountant 单独确认，不得把一个分类答案自动扇出到该 entity 名下所有交易。
- 跨表 / 跨运行聚合与去重、近似 surface 等价合并、answer-time 去重注册机制均未冻结。

## 7. 证据不足时的行为

如果分类、税务、split / allocation、个人 vs 公司立场或 JE 构造所需信息不足：

- 输出：继续追问 / 保持 Pending。
- 下游应：等待 accountant 补足信息，或由 accountant 决定是否需要客户问话。
- 本节点不能：用 confidence、推断性建议或历史模式掩盖未解决，也不能自行生成最终分类。

如果 accountant 自己也拿不准如何分类、需要问客户：

- 输出：可草拟一句客户问话给 accountant。
- 下游应：由 accountant 决定是否转发；客户回复经 accountant 返回。
- 本节点不能：直接联系客户、把问题当答案、或把未返回的客户答复当已知事实。

## 8. 歧义处理

身份线与分类线必须分开处理。

### 身份线：问“对方 entity 是谁”

如果 entity 不确定：

- 输出：Coordinator 主动、有界地追问一次。
- 是否允许自动化：不得把未知或模糊回答变成 stable entity。
- 是否需要 pending / review / governance：如果 accountant 也判断不出来、查不到或认为不需要，本节点就此收手，不反复逼问。
- 后续路由：这笔交易只走会计分类；分类完成后只 finalize 到 Transaction Log，不写 Entity Log / Alias Log / Case Log，不因身份未知而阻塞。

### 分类线：问“这笔怎么记账”

如果会计分类信息不足、模糊或冲突：

- 输出：继续结构化追问 / 保持 Pending。
- 是否允许自动化：不允许 Coordinator 自行分类，不允许为推进 workflow 猜一个 winner。
- 是否需要 pending / review / governance：持续追问到信息足以生成 JE；若答复暴露长期风险，另产出 governance / review candidate。

本节点不能把身份线的一次停止规则误用于分类线，也不能把分类线的持续追问规则误用于身份线。

## 9. 冲突处理

如果 accountant 答复与 Pending 上下文、当前证据或候选建议之间存在分类冲突：

- authority 顺序：accountant 对最终会计处理拥有决定权，但 Coordinator 可以用会计知识判断答复是否完整、是否足以构建最终分类。
- 本节点行为：够落地则打包给下游；不够落地则继续追问并说明缺口。
- 是否生成 review / governance candidate：如果答复暴露长期规则、policy、case-derived risk 或治理风险，可输出 candidate。
- 是否阻断自动化：信息不足或冲突未解时保持 Pending。

如果出现身份冲突、疑似同名、疑似 merge / split、identity conflict 或身份风险：

- authority 顺序：Coordinator 不在运行期裁决身份冲突；accountant 明确 identity confirmation 可按身份确认路径推进，冲突发现与解决归治理 / Entity Log 相关路径。
- 本节点行为：不消费 identity risk flags，不把风险 flag 投影成当前会计师问题；按已确认交互结果推进。
- 是否生成 review / governance candidate：身份冲突由 Entity Log §4 `merge_split_candidate`、Post-Batch Lint / Governance Review 等圈外路径处理。
- 是否阻断自动化：本轮不预设证据造假、冒名等极端边缘情形的运行期处置；这类问题归治理 / 后续设计。

本节点不能为了让 workflow 继续而猜一个 winner。

## 10. Audit / Trace 边界

本节点应保留的 trace：

- 交互留痕：问题、回答、确认、纠正、追问、身份问一次停止原因，交 Intervention Log 最小 record。
- 学习先例：stable-linked finalized 交易的 accountant-confirmed 学习语义，交 Case Log 写入机制。
- 身份创建 provenance：accountant explicit identity confirmation 触发的 stable entity 创建 provenance，交 Entity Log + Alias Log 统一写入路径。
- 结果审计：最终交易 outcome、processing path、身份未知但分类已完成的 finalization，交 Transaction Log 或等价 finalization source。

这些 trace 用于：

- audit。
- correction。
- offline system improvement。
- future learning eligibility。
- governance / review context。

这些 trace 不能成为：

- entity authority。
- Alias authority。
- rule authority。
- case authority。
- accountant approval。
- governance approval。
- automation permission。

Intervention Log 只记录交互事实；结果审计不挂在 Intervention Log 上。Case Log 只保存具备 stable entity + finalization proof 的学习先例；Transaction Log 是最终交易审计 source of truth。

## 11. Legacy Constraint Translation

New system / Old system 材料不构成本节点 authority。本节点只保留已经脱离来源、并按当前产品目标重新论证后的约束。

明确不继承的旧行为：

| Old Behavior | 不保留原因 |
| --- | --- |
| 把 EI / Transaction Identity / Profile / ER / RuleMatch / orchestrator 等都画成 Coordinator 触发源 | 当前正式上游锁为 CJ Pending；Transaction Identity 已删除，B 类材料不具 authority。 |
| 让 Coordinator 持有整表控制流、人机前台呈现或自主 agent routing | 批级触发和交互呈现归编排层；Coordinator 只消费 Pending，不控制 workflow。 |
| 让 Coordinator 直接写 Entity / Alias / Case / Rule / policy / Transaction 等 durable memory | 违反运行 / 记忆 seam；本节点只声明语义和 authority 来源，写入机制归对应记忆 / finalization 层。 |
| 把 ambiguous / unresolved / conflict 当作 candidate identity 或第三态身份 | 当前身份状态只允许 stable / unknown；治理 candidate 不是身份状态。 |
| 用一个 stable entity 的分类答复自动套用到所有同 entity 交易 | 分类逐笔独立，聚合只允许作为呈现 / 处理效率手段。 |

## 12. Open Boundaries

以下问题未冻结。

### L3（字段 / enum / 阈值）

1. CJ Pending handoff / `pending_request_context` exact field schema、pending 子类型 enum；与 CJ L3 对齐。
2. Pending 中 identity 背景、阻塞原因、已执行工作、可选推断性建议、evidence refs 的 exact 字段；identity risk flags 不属于 Coordinator 所需字段。
3. Coordinator question 模板 exact 字段、回答结构、追问边界字段。
4. 情况 X（不需要主体）/ 情况 Y（主体不明）在 schema 中的落点字段。
5. 带选项 pending 问题的选项结构、accountant 回答 / 确认的结构化字段。
6. 草拟客户问话的结构化承载字段；不冻结对外发送机制。
7. 身份确认写入所需最小创建 provenance + Entity Log 强制字段 exact schema；与 Entity Log M3 对齐。
8. Case Log 学习记录的 `use_level` / `confirmed_by` enum、accounting outcome 快照字段；与 Case Log M3 对齐。
9. automation policy / rule / case-derived risk / profile / review-governance candidate 的 exact 类型 / schema；消费方多为圈外。
10. 转 JE 所需格式 exact field contract；依赖 JE Generation 落文。

### L4 / seam（机制）

1. Coordinator 触发 Entity Log + Alias Log 同步写入的执行者、调用方式、写入顺序和多 log finalization。
2. Case Log 学习记录 exact writer、trigger order，以及与 Transaction Log / Entity Log 的统一 finalization。
3. stable entity 下分类问题聚合呈现 / 处理机制，以及逐笔确认如何防止分类自动扇出。
4. 跨表 / 跨运行聚合与去重、近似 / 不同写法 surface 的等价合并、answer-time 去重持久化注册表；本轮不做。
5. 多表提交串行 / 并行排序、批起始记忆快照。
6. 会计师答复后是否 / 如何受限重入上游、重跑边界；已确认交易不重入 CJ / ER 已锁。
7. 提问 exact 话术、entity-first 条件分支的呈现顺序、身份问一次停止后的 UX。
8. 情况 Y 身份缺口的 triage / 复查机制。
9. hard block 交易经提问不可解除后的最终归处；取决于 CJ hard block 定义和编排层。
10. 会计师久不答复 / 无回应的 Pending 处置。
11. 人机对话通道归属、多节点提问合并 / 排序 / 呈现。

### L2·外阻（圈外依赖）

1. JE Generation Node：转 JE 格式、CJ / Coordinator 到 JE 的 COA 校验 seam。
2. Transaction Log：身份未知但分类完成交易的落点、finalization、结果审计 source of truth。
3. Intervention Log：durable interaction record 的正式 spec、schema、writer、与 Evidence Log / Review / Governance 边界、离线学习材料消费方式。
4. Governance / Governance Review：长期 authority 变化、policy / rule / merge / split 等批准路径。
5. Profile / Structural Match、Knowledge Summary、Review Node、Case Memory Update Node、interaction_agent、Onboarding 的正式边界。
6. 身份冲突、疑似同名、疑似 merge / split、identity conflict 的发现与解决机制；归治理 / Entity Log 相关路径，不在 Coordinator 运行期裁决。
7. 极端身份安全 / 证据真实性情形的运行期处置；本轮不预设边缘情形。
8. ER `candidate_signal`（非身份状态）是否最终也进入 Coordinator；本轮输入面仍锁为 CJ Pending。

### 进入限制

这些问题解决前，不能进入：

- [x] Stage 3 data contract
- [x] Stage 4 execution algorithm
- [x] implementation
