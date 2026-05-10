# AGENTS.md

## 文件职责

这是本系统审计仓库的 agent 操作入口。

它定义：

- 文档权威顺序
- 不同任务类型的必读材料
- 如何对待当前系统材料与旧系统材料
- 可编辑范围与只读参考范围
- 审计方法与停止条件
- 必要的交接 / 总结纪律

它不负责产品设计结论、当前任务状态、阶段规划或已接受的决策历史。

## 权威顺序

当文档之间出现冲突时，按以下顺序解决：

1. 用户的明确指令
2. `AGENTS.md`
3. `TASK_STATE.md`
4. `PLANS.md`
5. `AUDIT_CHARTER.md`
6. `DECISIONS.md`
7. `new system_副本/` 下的当前系统设计材料
8. `old_system_nodedesign/` 下的旧系统参考材料

每份文档只在自己的职责范围内拥有权威：

- `AGENTS.md` 负责 agent 操作规则、source role 规则、必读材料、停止 / 升级条件和输出纪律。
- `TASK_STATE.md` 负责实时交接：当前状态、当前目标、下一步行动和当前风险。
- `PLANS.md` 负责审计阶段模型、退出标准和规划问题。
- `AUDIT_CHARTER.md` 负责稳定项目宪章、长期审计原则和长期产品约束。
- `DECISIONS.md` 负责本审计仓库中已接受的审计决策和已接受的 tradeoff。
- `new system_副本/` 包含正在被审计的当前系统。它描述当前设计说了什么，但不能证明该设计是正确的。
- `old_system_nodedesign/` 包含旧系统的设计意图样本。它不是评判标准、目标架构或默认 baseline。

## 核心审计标准

整个审计必须服务于这个产品目标：

> 帮助会计师把客户历史账务和过往判断沉淀成可复用记忆，让新交易自动获得有证据支持的记账建议，并通过会计师纠正持续学习，在不牺牲审计性和控制权的前提下逐步提高自动化率。

每一个 node、field、workflow、log、memory layer、abstraction 和拆分，都必须围绕这个目标接受判断。

如果某个设计增加了复杂度，却没有清楚改善记忆复用、有证据支持的建议、纠错学习、审计性、会计师控制权或自动化率，就应当被视为可疑。

## 材料角色

### 当前系统

`new system_副本/new_system.md` 是当前系统的主要总览，也是主要审计对象。

`new system_副本/node_stage_designs/` 包含当前 node-stage 设计材料。这些是审计输入，默认不是 implementation-ready contracts。

审计某个 node 时，如果对应 Stage 1、Stage 2 和 Stage 3 文件存在，应读取这些文件，并读取评估边界所需的相关上游 / 下游 node 文件。

### 旧系统

`old_system_nodedesign/` 下的旧系统文件可以用于推断：

- 原始设计意图
- 系统运行时心智模型
- 用户当初真正关心的问题
- 用户重视的能力
- 仍可能值得保留的可复用约束

不要把旧系统用作：

- 标准答案
- 回归目标
- 证明新系统错误的依据，除非只是因为它和旧系统不同
- 证明新系统正确的依据，除非只是因为它更新或更模块化

如果复用了旧系统中的某个想法，必须明确说明：保留下来的约束是什么，以及为什么它仍然服务于核心产品目标。

## 必读材料

进行实质性审计工作前，先读：

1. `TASK_STATE.md`
2. `AUDIT_CHARTER.md`
3. `PLANS.md`
4. `new system_副本/new_system.md`

进行 node-level audit 前，还要读：

- `new system_副本/node_stage_designs/` 下该 node 的相关文件
- 评估边界所需的上游和下游 node 文件
- 只有在理解当前系统行为之后，才读取相关旧系统文件来推断原始意图或可复用约束，除非用户明确要求先做 legacy comparison

修改治理文档前，先读：

- `AGENTS.md`
- `TASK_STATE.md`
- `AUDIT_CHARTER.md`
- `PLANS.md`
- `DECISIONS.md`

当前审计仓库没有 `supporting documents/` 目录。除非之后新增该目录，否则不要引用或要求读取不存在的 supporting files。

## 审计方法

使用第一性原理批判，而不是对某个来源的忠诚。

对每个重要设计对象，追问：

- 它为什么必须存在？
- 它解决了什么无法被更简单方式解决的问题？
- 如果删除、合并或内联它，会失去什么能力？
- 失去的能力对核心产品目标真的重要吗？
- 它是否改善了记忆复用、证据支持、纠错学习、审计性、控制权或自动化率？
- 它是否让系统更难理解、验证、审计或控制？
- 它是真实产品必要性，还是为了让架构看起来完整而存在？

默认可接受的审计结论包括：

- 保留，并说明理由
- 删除
- 合并
- 内联
- 重命名
- 收窄职责
- 只有当拆分能消除真实歧义时才拆分
- 因为需要真实产品决策而暂缓

不要因为某个 node、field、log、memory store、phase 或 contract 已经出现在当前设计中，就默认它是有效的。

## 可编辑范围

默认可编辑的治理 / 审计文档：

- `AGENTS.md`
- `TASK_STATE.md`
- `PLANS.md`
- `DECISIONS.md`
- `AUDIT_CHARTER.md`，仅当稳定宪章或长期原则确实改变时
- 用户要求创建的新审计报告或聚焦审计笔记

默认只读参考的设计文档，除非用户明确要求编辑：

- `new system_副本/new_system.md`
- `new system_副本/node_stage_designs/*.md`
- `old_system_nodedesign/*.md`

审计过程中不要静默改写产品设计 spec。如果某个审计发现意味着需要设计变更，应将其报告为 finding、recommended simplification 或 decision point，除非用户明确要求你编辑 spec。

## 操作规则

- 始终把审计锚定在核心产品目标上。
- 第一性原理优先于维护当前结构。
- 优先选择能够保留必要能力的最小设计。
- 把审计性和会计师控制权视为产品要求，而不是装饰。
- 清楚区分 evidence、memory、judgment、governance 和 audit records。
- 不要仅仅为了对称性、完整性或未来可选性而新增组件。
- 不要在没有翻译出仍然成立的约束前，把旧系统结构推广进新系统。
- 不要把第一版 Stage 3 data contracts 当成冻结的实现文档。
- 不要把 AI reasoning 文本变成可复用 authority。可复用 authority 必须来自已批准的 memory、rules、cases、profile 或 governance state。

## 停止 / 升级

遇到以下情况，停止并询问用户：

- 任务需要改变产品设计合同，而不只是审计它们
- 某个审计发现依赖用户尚未决定的业务优先级
- 两个权威来源冲突，且无法通过权威顺序解决
- 当前工作会把旧系统 spec 当成当前有效 baseline
- 当前工作会模糊当前系统材料与历史参考材料的边界
- 任务需要大范围重写 `new system_副本/`，而不是一次聚焦的已请求编辑

遇到以下情况，宁可暂缓，也不要强行闭环：

- 当前 baseline 需要真实设计决策，而不是措辞清理
- 某个旧系统能力无法干净翻译进当前系统
- 某个简化会改变审计性、会计师控制权或自动化 authority，而用户尚未批准

## 文档更新纪律

- 当前状态、下一步行动或 active risks 有实质变化后，更新 `TASK_STATE.md`。
- 审计阶段、gate 或成功标准变化后，更新 `PLANS.md`。
- 将已接受的审计决策和 tradeoff 追加到 `DECISIONS.md`；不要把源项目决策导入成虚假的 authority。
- 保持 `AUDIT_CHARTER.md` 稳定、低 churn。
- 保持 `AGENTS.md` 聚焦 agent operation，不要把临时交接细节塞进去。
- 保留有用历史意图时，要把它标为 intent 或 constraint，不要静默当成当前设计规则。

## 输出要求

审计输出必须区分：

- 已确认设计问题
- 设计风险
- 简化候选
- 待用户决定的问题
- 假设或未解决证据缺口

最终总结必须包括：

- 改了哪些文件
- 为什么改
- 做了哪些验证
- 剩余风险 / 后续事项

如果刻意没有运行验证，要明确说明。
