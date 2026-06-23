# Human Review Node - Functional Intent

## 1. 为什么这个节点必须存在

这个节点解决的问题是：

> 独立于逐笔交易流水线，由会计师主动发起，面向已 finalized 事实的事后纠错，以及系统与会计师之间关于规则变化、错误治理、事后纠错的沟通桥梁。（决策 1；用户澄清 1）

如果删除、合并或内联它，会失去：

- 会计师在 finalized 后唯一干净的纠错入口。
- 人类权威天然在场的 read-back、签字与凭证漏斗。
- 不污染 durable memory 的可审计 correction path。

这些能力对核心产品目标重要，因为：

- `accountant control` 要求会计师能改系统已经确定的事实。
- 审计性要求改动留下 correction append、Intervention ID 和 governance trace，而不是覆盖旧记录。
- `accountant correction learning` 要求错误先例能被作废或更新，但不能让 LLM 或运行节点自行变成 memory authority。

不能把本节点内联进 Coordinator。Coordinator 只处理 running / Pending 交易，正式边界已锁为交易 finalized 后不得触发；finalized 后的人发起纠错归本节点。（Coordinator 02 §1；决策 1）

## 2. 核心职责

本节点的唯一核心职责是：

> 按会计师指令对系统已确定事实实施修改的确认与编排面：结构化意图、展开影响、组织 read-back 和签字，并备好符合各 log Finalization 要求的 input，交 Finalization 落盘；本节点自己不是 durable write 执行者。（决策 1 / 9）

本节点可以辅助产生：

- rule 错误、例外、scope 过宽等治理信号。
- rule 升级候选的确认面与触发器。
- correction / governance / review 所需的人类可读解释和 read-back。

但这些不是主职责。它是 conductor，不是 judge；判断者是 accountant，放行权在 Finalization 凭证校验死代码。（决策 9 / 10）

## 3. 明确排除范围

本节点不负责：

- onboarding、材料提交前目标询问、覆盖完整性核对；这些归 interaction_agent / onboarding 路径。（决策 1 / 9b）
- 运行期 Pending、running 交易、blocked / terminal / 卡住交易的处置；这些归 Coordinator 或对应运行期路径。（决策 1 / 2 / 9b）
- 直接写 Transaction Log、Intervention Log、Case Log、Entity Log、Rule Log、Governance Log 或其它 durable memory。
- 系统自发 merge / split 或系统自发错误发现入口；该系统自发发现层已与 FP-5 合并挂起，本节点当前只承接会计师人发起的 merge / split 与纠错。（用户澄清 4）
- 定义纯 rule 升级的固定执行路径；该路径归 Rule 侧，本节点只作确认面和触发器。（用户澄清 3）
- 决定 Chatbot 前端是否与 Coordinator / interaction_agent 共用；这是编排 / 呈现层问题。（用户澄清 5）

本节点绝不能：

- 让 LLM 写 durable bytes，或让 LLM 自批 / 放行。
- 高置信度自动通过、跳过 read-back，或把会计师“改这一笔”静默升级成“以后永远这么走”。
- 当场裁决某条 rule “坏了”并执行降级；单次纠错只能结构化该笔并发出“rule 被推翻一次”信号。
- 重判确定性检查、跨 rule 互斥校验、promotion 资格判定或 Finalization 凭证校验。
- 产生 candidate entity / candidate identity，或把 ambiguous / unresolved / conflict 当成独立身份状态。

## 4. Workflow 位置

上游：

- 非 pipeline handoff 的会计师主动发起：批后复核、导入 QuickBooks 前整批审、周审等。（决策 1）
- 审核 inbox 候选：确定性发现 job 投入的候选，本轮当前只服务 rule 升级候选；本节点 pull 并作为确认面。（决策 13；审核 inbox 待建）
- 未来 LLM 审查节点投入的候选；当前不存在，接口同审核 inbox。（决策 13）

下游：

- Finalization 写入机制：本节点备料，Finalization 执行跨 log 持久化和凭证校验。（决策 3 / 9）
- Transaction Log：correction append。
- Intervention Log：原因、交互事实和 Intervention ID；正式 spec 待建。
- Case Log：先例更新 / supersede。
- Entity Log / Rule Log：扩张型 mutation 的 approved input。
- Governance Log：扩张型变更审计；数据层待建。
- Rule 侧固定执行路径：纯 rule 升级由本节点触发，定义归 Rule Log / Rule Match 治理。（NEW-1；用户澄清 3）

本节点位于流程中的原因：

- 它不在逐笔交易 running path 内；位置可以是批后、导入前或周审，但功能要求固定：只处理 finalized 后、非 runtime 的会计师纠错和治理沟通。

## 5. 对核心产品目标的贡献

本节点必须清楚支持以下至少一项：

- [ ] 记忆复用
- [ ] 有证据支持的建议
- [x] accountant correction learning
- [x] 审计性
- [x] accountant control
- [ ] 自动化率提升

具体贡献：

- 给会计师一个事后唯一干净入口，可以纠正已 finalized 事实。
- 通过 read-back、签字、凭证和 Finalization，把人类确认与 durable 写入隔开，避免 LLM 或运行节点污染 memory authority。
- 通过 Transaction Log append-only、Intervention ID、Case Log supersede / update、Governance Log 留痕，让纠错可复核、可追溯。

## 6. 已知约束

- LLM 永不写字节：LLM 只产结构化意图、执行图、`P_llm`、read-back 和解释；durable 写入由确定性代码 / Finalization 执行。（决策 5）
- 写入前强制 read-back：每一次纠错都念对账后的复述版，由会计师确认并签字；read-back 防语义幻觉，Finalization 防格式 / 写入幻觉。（决策 5）
- scope 绝不静默升级：单笔 vs 永久、只改这一笔 vs rule 名下全部，都是独立显式确认。（决策 5 / 12）

## 7. 未决定问题

- 系统自发发现层（FP-5 + 语义发现器）是否必要：暂不删除，合并挂起，另窗讨论；本节点当前不实现系统自发入口。（用户澄清 4）
- NEW-1：rule 升级固定执行路径的定义归 Rule 侧；本节点只触发，Rule 侧机制仍待落实。（用户澄清 3）
- C2：本节点 Chatbot 是否与 Coordinator / interaction_agent 共用同一对话前端，归编排 / 呈现层。（用户澄清 5）
