# Coordinator Node - Functional Intent

## 1. 为什么这个节点必须存在

这个节点解决的问题是：

> Case Judgment 输出 Pending 后，系统需要一个 runtime 交互桥，把 CJ 的阻塞点转成会计师能回答的问题，并在会计师答复足够后把结果交给下游。

如果删除、合并或内联它，会失去：

- CJ Pending 的唯一归处：Pending 会停在无法被会计师理解和处理的系统内部状态。
- 会计师控制权：系统会被迫让 CJ 或其他节点在上下文不足时继续做会计分类判断。
- 干净的人机分界：机器负责拆解、沟通和完整性判断，会计师负责最终身份确认与会计处理。
- runtime 交互留痕与学习闭环入口：会计师确认后的交易无法清楚连接 Transaction Log、Entity Log、Case Log 和 Intervention Log 的各自职责。

这些能力对核心产品目标重要，因为：

- 自动化率依赖 Pending 能被高效转成可回答问题，而不是长期卡住。
- 记忆复用依赖会计师明确确认 identity 后形成 stable entity，且 stable-linked finalized 交易能进入 Case Log 学习路径。
- correction learning 依赖会计师交互被记录为可供系统设计者离线改进的材料。
- 审计性依赖会计师答复、身份确认、最终交易结果和学习记录各自落在正确 trace / memory 边界内。
- accountant control 依赖最终会计处理与身份确认始终由会计师决定。

## 2. 核心职责

本节点的唯一核心职责是：

> 接手 CJ Pending，组织结构化会计师交互，判断会计师答复是否足以生成 JE，并把已确认结果打包给下游。

本节点可以辅助产生：

- 带选项的 pending 问题，供会计师选择或确认。
- 面向客户的、针对当前交易细节的一句问话草稿，供会计师人工转发给客户。
- 身份确认信号、Case Log 学习记录语义、Intervention Log 交互留痕语义和 governance / review candidate。

但这些不是主职责。本节点不因能组织问题、草拟客户问话或判断答复完整性而获得最终会计分类权、durable write 权或 governance approval 权。

## 3. 明确排除范围

本节点不负责：

- 最终会计分类、税务处理、COA 选择或 JE 构造；会计师答复是最终会计处理的 authority，JE Generation 是下游构造层。
- 重跑 Case Judgment；已由会计师交互解决的交易不重入 CJ / ER。
- 持有 workflow 控制流、整表处理保证、多表排序、人机对话前台呈现或 onboarding 期交互。
- 直接读取 Entity Log、Intervention Log 或任何 durable store；全部上下文随 CJ Pending 在手。
- 裸写 Entity Log、Alias Log、Case Log、Rule Log、Transaction Log、Intervention Log、Governance Log、Profile 或 policy / rule。
- 在运行期裁决身份冲突、merge / split、identity risk flags 或治理风险。

本节点绝不能：

- 越过 hard block，或把通过提问不能解除的阻断包装成已解决。
- 把模糊、猜测、同名或 accountant 未明确确认的回答包装成 stable entity。
- 复活 candidate entity、candidate identity、第三态身份状态，或把 ambiguous / unresolved / conflict 当作独立身份输出类别。
- 把会计师交互痕迹直接变成 Entity / Case / Rule / policy authority。
- 在 Stage 1-2 冻结字段级 schema、enum、阈值、refs 形态、存储 / projection、写入执行者、写入顺序或 finalization 顺序。

## 4. Workflow 位置

上游：

- Case Judgment：CJ 的 Pending 集合是本节点唯一输入；Pending 必须携带最小语义，使 Coordinator 不另开读取面。

下游：

- JE Generation：消费会计师答复足够后的转 JE 所需格式；该对象仍是圈外 L2·外阻。
- Entity Log + Alias Log 统一写入节点：消费会计师明确身份确认后的 stable entity 出现信号；实际写入机制留 L4 / seam。
- Transaction Log：承接最终交易结果审计；拿不到 entity 但完成分类的交易只 finalize 到 Transaction Log。
- Case Log 写入机制：承接 stable-linked finalized 交易的学习记录语义；exact writer 和触发顺序留 L4 / seam。
- Intervention Log：承接最小交互留痕 record；正式 spec 仍是圈外 L2·外阻。
- Governance / Review 路径：消费长期记忆、policy、rule、case-derived risk 等 candidate；多为圈外。

本节点位于流程中的原因：

- ER 只判定 identity stable / unknown，不负责会计师交互。
- CJ 只输出 High Confidence Classification 或 Pending，不负责把 Pending 变成人类对话。
- JE Generation 只构造分录，不补会计判断。
- Memory layer 负责 durable authority，本节点只在运行层声明要持久化的语义和 authority 来源。

## 5. 对核心产品目标的贡献

本节点必须清楚支持以下至少一项：

- [x] 记忆复用
- [x] 有证据支持的建议
- [x] accountant correction learning
- [x] 审计性
- [x] accountant control
- [x] 自动化率提升

具体贡献：

- 记忆复用：在身份缺口分支主动询问 entity，得到会计师明确确认后触发 stable entity 写入路径；stable-linked finalized 交易可形成 Case Log 学习记录。
- 有证据支持的建议：把 CJ Pending 携带的阻塞原因、已执行工作、可选推断性建议和交易辅助证据组织成会计师可判断的问题。
- accountant correction learning：会计师交互被保留为 Intervention Log 最小留痕，并可在正确路径下形成 Case Log 学习语义。
- 审计性：会计师答复、身份确认、最终交易结果和学习记录分别指向 Intervention Log、Entity Log、Transaction Log、Case Log，不互相替代。
- accountant control：本节点只能追问、解释和判断信息是否足够；最终身份与最终会计处理必须由会计师决定。
- 自动化率提升：会计师确认一次 stable identity 或稳定处理经验后，后续可由 ER exact-match、Case Log 先例或正式规则路径复用。

## 6. 已知约束

- 身份判断 `stable` / `unknown` 锁在 Entity Resolution；Coordinator 不新增身份状态。
- 会计分类判断权锁在 Case Judgment / accountant 交互路径；Coordinator 不自定分类结论。
- CJ Pending 是唯一输入；Pending 的唯一 consumer 是 Coordinator。
- 已确认交易不重入 CJ / ER。
- accountant 明确身份确认可以创建 stable entity，无需 governance approval；该确认只确认 identity，不隐含分类、Alias、Rule、policy 或 Case Log 写入。
- Entity Log 身份背景由 CJ 在构造 Pending 时读取并投影；Coordinator 本身不直接读 Entity Log。
- identity risk flags 不投影给 Coordinator，也不由 Coordinator 或当前会计师消费；它们归 Human Review（会计师人发起）/ Governance，以及 ER 运行期判句 / 确定性发现的确定性冲突处理。
- `Role` 字段和 Transaction Identity 节点已删除，本节点不得按旧材料补回。
- Candidate 只允许作为非身份候选：automation / policy / rule / case-derived risk / review-governance candidate 等；它们不是身份状态，也不得自行持久化。

## 7. 未决定问题

- 圈外依赖尚未落文：JE Generation、Transaction Log、Intervention Log、Governance / Review、Case Memory Update、interaction_agent 等。
- 字段级 schema、enum、阈值、refs、写入执行者、写入顺序、聚合呈现机制、多表排序和 finalization 顺序均未冻结。
- 详情见 `02_logic_and_boundaries.md` §12 和 `00_index.md` 的进入下一阶段 gate。
