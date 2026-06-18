# Case Judgment Node - Functional Intent

## 1. 为什么这个节点必须存在

这个节点解决的问题是：

> 当交易没有可执行 active rule、或身份无法支撑自动分类时，系统仍需要一个受控的兜底判断节点，把单笔交易产出为可交付结果：High Confidence Classification 或 Pending。

如果删除、合并或内联它，会失去：

- 失去全系统的兜底归处保证：结构化路径和 rule path 处理不了的交易会变成没有明确负责节点的孤儿。
- 失去高自动化核心：大量证据足够、但尚无 active rule 或历史先例的交易会被迫进入人工路径。
- 失去干净的人机分界：机器能确信时自动给出会计分类，拿不准或被 authority 阻断时回到会计师。
- 失去与 Entity Resolution、Rule Match、JE Generation 的职责分离：identity、approved rule application、case-based accounting judgment 和 JE 构造会互相污染。

这些能力对核心产品目标重要，因为：

- 自动化率依赖 CJ 能处理“没有 rule 但证据足够”的常见交易。
- 审计性依赖 CJ 用结构化 reasoning 说明证据如何收敛到 COA 与 HST / GST，而不是只留下结果。
- accountant control 依赖 CJ 在 hard block、歧义、冲突或非公司支出立场不明时干净交回会计师，而不是静默放行。
- correction learning 依赖 CJ 把 High Confidence 路径产生的可学习 case 内容交给后续 Case Memory Update，而不越权写入长期记忆。

## 2. 核心职责

本节点的唯一核心职责是：

> 对流通到本节点的单笔交易做受控会计分类判断，并输出 High Confidence Classification 或 Pending。

本节点拥有运行时选择 (COA, HST/GST treatment) 的判断权：High Confidence Classification 必须携带足以让下游 JE Generation Node 确定性构造分录的 COA 与 HST / GST 决定。

本节点可以辅助产生：

- 结构化 reasoning / audit trace，用于 review、correction、governance 和 audit。
- Pending 的 reason、已执行动作摘要和可选推断性建议，供 Coordinator 组织会计师问题。
- `case_memory_update_candidate`，仅在 High Confidence 自动路径产生，供 Case Memory Update Node 在 finalized 后处理。

但这些不是主职责。本节点不因产生 reasoning、Pending 建议或 case memory candidate 而获得 durable write、governance approval 或长期规则 authority。

## 3. 明确排除范围

本节点不负责：

- 交易拆分（split）的检测或执行；拆分检测与执行均在 CJ 外，当前为 DEFERRED / 上游或 Coordinator 外阻。
- 身份判定；身份状态来自 Entity Resolution 的 `stable` / `unknown`，CJ 不重新造 stable entity。
- 构造、校验或过账 journal entry；JE Generation Node 是下游纯构造层，COA 命中客户 COA 的确定性校验可在 CJ 末尾检查器或 JE 前置检查中完成。
- onboarding initialization、历史回填学习或 knowledge compilation；CJ 是 runtime-only 的逐笔节点。
- 充当 mixed-use / 个人消费稽查员；CJ 默认公司支出、不主动搜寻私人物品，只在证据明确指向非公司支出时交回会计师。
- 读取 Governance Log / Rule Log 原始账本来重新解释 hard block；CJ 只消费上游 / 记忆层已经落到 CJ 可读标记上的结果。

本节点不负责提出以下候选：

- 身份类候选（包括 entity / identity 方向）。
- Alias / role candidate。
- rule promotion / active rule candidate。
- automation-governance / policy candidate。

本节点绝不能：

- 输出 High Confidence Classification / Pending 之外的第三种下游分类状态；hard block 一律转 Pending，不设独立人工复核态。
- 复活身份类候选或 `new_stable_entity` 旁路。
- 把 Case Log 先例、Knowledge Summary、LLM reasoning 或 repeated history 升格为 deterministic rule。
- 把模型写成自主持有路由权 / 控制流权的 agent。
- 直接写 Entity Log、Alias Log、Rule Log、Case Log、Transaction Log、Governance Log 或 Profile。
- 在 L2 冻结字段级 schema、enum、阈值、存储形态、写入执行者或 finalization 顺序。

## 4. Workflow 位置

上游：

- Entity Resolution：提供 `stable` / `unknown` 身份状态、identity reason 和 evidence refs。
- Entity Resolution / workflow router：当 ER 认出 stable entity 且该 entity 没有 active rule 时，交易直接进入 CJ；该路由由 ER 还是 workflow router 执行属于 L4 / seam。
- Rule Match：当 stable entity 有 active rule 但本笔交易不命中任何 applicable rule 时，输出 `rule_miss` 进入 CJ。
- Evidence Intake / Preprocessing：提供可追溯 evidence foundation 与稳定 `transaction_id`。
- Profile / Structural Match：当前仅作为圈外未锁依赖；CJ 假设结构性路径未完成的交易才会进入本节点，但该对象尚无正式草案。

下游：

- JE Generation Node：消费经 deterministic COA 校验后的 High Confidence Classification，用确定的 (COA, HST/GST) 构造 JE；该对象尚无正式草案，CJ 不冻结其内部。
- Coordinator：Pending 的唯一 consumer，负责把 reason、候选建议、交易 / entity 信息组织成会计师可回答的问题。
- Case Memory Update Node：消费 `case_memory_update_candidate`；在交易 finalized 后凭 `transaction_log_ref` 或等价 finalization proof 处理 Case Log 写入。
- 后续 finalization / audit / learning 机制：负责 Transaction Log、Case Log 等持久化；CJ 只声明要持久化的语义，不声明写入者、顺序或存储机制。

本节点位于流程中的原因：

- Entity Resolution 只回答“是谁 / 是否未知”，不拥有会计分类权。
- Rule Match 只应用已经批准的 deterministic active rule，不做 case-based judgment。
- JE Generation 只消费确定 accounting treatment 构造分录，不补分类判断。
- Coordinator 负责人工交互，不应在上下文不全时重做 CJ 的会计分析。

## 5. 对核心产品目标的贡献

本节点必须清楚支持以下至少一项：

- [x] 记忆复用
- [x] 有证据支持的建议
- [x] accountant correction learning
- [x] 审计性
- [x] accountant control
- [x] 自动化率提升

具体贡献：

- 记忆复用：stable entity 下读取 Case Log 先例摘要和 entity-level Knowledge Summary 作为辅助上下文，但不把它们升格为 rule。
- 有证据支持的建议：High Confidence 与 Pending 都必须带结构化 reasoning，说明当前证据、历史佐证和 COA / HST 结论的关系。
- accountant correction learning：CJ 只发 High Confidence 路径的 case memory update candidate，后续 correction / finalization 路径再决定是否进入 Case Log。
- 审计性：reasoning 进入 Transaction Log 或等价 audit trace；trace 可供 review / correction / governance / audit，但不成为 authority。
- accountant control：hard block、identity unknown、authority 冲突、个人 vs 公司立场明确不属于公司时均交回会计师。
- 自动化率提升：没有 active rule、没有历史先例但证据足够唯一落到 (COA, HST/GST) 的交易，可以走 High Confidence Classification。

## 6. 已知约束

- 本节点只对下游声明两类分类输出：High Confidence Classification 与 Pending。
- `stable` 身份是 High Confidence Classification 的必要前提；ER `unknown` 一律输出 Pending，可附推断性建议。
- CJ 不设 ER 式硬前置触发门；任何能流通到 CJ 的交易都应被处理为两类输出之一。结构性无效 / 损坏输入的拒绝与否是 DEFERRED，不在本 L2 冻结。
- Case Log 是辅助先例，不是 high confidence 的必要条件，也不是 deterministic rule source。
- 代码和模型分工为“代码 -> 模型 -> 代码”：模型可以在判断回合内为会计事实联网取证，但最终 High Confidence vs Pending 由代码根据确定性约束裁定。
- 硬阻断必须由结构化字段 / flag 被代码确定性计算；语义级清单已定，exact 字段与 enum 留 L3。
- High Confidence 进 JE 前必须过客户 COA 确定性校验；未命中反馈回 CJ 重判，仍无法合法落到 COA 时转 Pending。
- mixed-use / 个人消费风险默认公司支出、不主动稽查；证据明确指向非公司支出时交回会计师，不设 materiality 阈值 gate。
- 本节点只产生 case memory update candidate；不产生身份、alias、rule、automation 或 governance 候选。
- Profile / Structural Match、JE Generation Node、entity-level Knowledge Summary、Coordinator / Pending、Case Memory Update、Transaction Log 等对象的正式边界仍有外阻；本节点引用它们时只能声明自身契约面和 open boundary。

## 7. 未决定问题

- 见 `00_index.md` 的“当前未冻结边界”。
