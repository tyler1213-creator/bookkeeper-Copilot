# AGENTS.md

## 文件职责

这是本系统审计仓库的 agent 操作入口。

它负责：

- 新窗口读档路线
- source authority 规则
- 不同任务类型的必读材料
- New system / Old system 的作废与授权读取边界
- 可编辑范围与历史材料授权边界
- 审计方法、停止条件和输出纪律

它不负责项目背景、当前状态、阶段规划、系统地图、缺口清单或具体主题结论沉淀。

## 当前 active 规则 / 上下文文档

根目录 active 规则 / 上下文文档只保留：

- `AGENTS.md`
- `当前任务状态.md`
- `审计目标与原则.md`
- `审计阶段路线图.md`
- `项目背景速读.md`
- `系统上下文地图.md`
- `缺口地图.md`

其他材料角色：

- `BK_Copilot/`：已开始重写的正式 node / memory layer 草案；对应对象的当前标准边界以这里为准。
- `BK_Copilot/node:layer spec template/`：正式 node / memory layer spec 的模板规则。
- `L2_proposals/`：已审或讨论中对象的 L2 提案与追溯入口；不替代 `BK_Copilot/` 正式草案。
- `独立question文档/`：主题讨论 / 机制结论入口；角色逐文件决定。若用户明确把某一 question 文档升为“当前唯一权威设计文档”，则在其主题范围内按该声明使用，直到落入正式 `BK_Copilot/` spec。已被正式草案替代的 question 文档只作历史背景。
- `interaction_agent.md`：交互 / 目标层 Agent 的根目录概念入口；正式 spec 未冻结。
- `未解决问题暂存清单.md`：已废弃历史参考，不再作为当前待办入口。
- `new system/`：New System 旧材料，已作废；不再作为审计目标、baseline、authority 或默认读档材料。未经用户明确同意，不得读取其文档内容。
- `old_system_nodedesign/`：Old-System 旧材料；不作为审计目标、baseline、authority 或默认读档材料。未经用户明确同意，不得读取其文档内容。

## Source Authority

用户明确指令永远最高。

其余文档只在自己的职责范围内拥有 authority：

- `AGENTS.md`：agent 操作规则、source authority、必读材料、停止 / 升级条件和输出纪律。
- `当前任务状态.md`：实时交接、下一步行动和 active risks。
- `审计目标与原则.md`：稳定项目宪章、长期产品目标和审计原则。
- `审计阶段路线图.md`：审计阶段模型、阶段目标和规划问题。
- `项目背景速读.md`：项目背景导览。
- `系统上下文地图.md`：材料结构导航和来源边界。
- `缺口地图.md`：未冻结 L3 / L4 / seam / L2·外阻缺口池。
- `BK_Copilot/`：对应 node / memory layer 的当前正式草案边界。
- `L2_proposals/`：L2 讨论与提案追溯入口。
- 用户指定为当前权威的 `独立question文档/` 主题文档：仅在该主题尚未落入正式 `BK_Copilot/` spec、且用户明确声明其权威状态时生效；不替代无关对象的正式草案。

如果同一职责范围内出现冲突，按上述顺序解决；如果职责范围不同，不要互相覆盖。比如 `当前任务状态.md` 不能改写 node 边界，`BK_Copilot/` 草案也不能改写 agent 操作规则。

以下材料不提供权威参考：

- `new system/`
- `old_system_nodedesign/`

其中 `new system/` 已作废，不再作为审计目标。`new system/` 与 `old_system_nodedesign/` 均是授权读取材料：只有在用户明确同意读取具体 New System / Old-System 文档时，才可进入；即使读取，也只能作为历史 source text / 追溯背景，不能证明某个设计正确、必要、应保留、应回归或应作为 baseline。

## 必读路线

任何新窗口先读：

1. `AGENTS.md`
2. `当前任务状态.md`

按任务补读：

- 不熟悉项目背景：读 `项目背景速读.md`。
- 需要稳定审计目标或原则：读 `审计目标与原则.md`。
- 需要阶段位置、gate 或路线图：读 `审计阶段路线图.md`。
- 需要材料结构和来源边界：读 `系统上下文地图.md`。
- 需要查未冻结缺口、seam 或圈外依赖：读 `缺口地图.md`。
- 需要当前正式草案：读 `BK_Copilot/` 下对应 node / memory layer 文档。
- 需要追溯已审对象 L2 讨论：读 `L2_proposals/[对象]__L2提案.md`。
- 默认读档范围只限现行 BKCopilot 内容：active 治理 / 跟踪文档、`BK_Copilot/` 正式草案、模板规则、必要的 `L2_proposals/` 与用户指定的主题记录。
- 需要读取 New System 或 Old-System 文档时，必须先取得用户对具体读取范围的明确同意；未获同意前不得读取 `new system/` 或 `old_system_nodedesign/` 下任何文档内容。
- 即使获得同意，`new system/` 已作废且不再作为审计目标；New System / Old-System 只能作为历史 source text / 追溯背景，不得进入 authority、baseline 或可复用约束来源。

修改治理文档前，先读：

- `AGENTS.md`
- `当前任务状态.md`
- `审计目标与原则.md`
- `审计阶段路线图.md`
- `项目背景速读.md`
- `系统上下文地图.md`

修改正式 node / memory layer 草案前，先读：

- 对应 `BK_Copilot/` 正式草案
- 对应模板规则：`workflow_node_spec_template_rules.md` 或 `memory_layer_spec_template_rules.md`
- `缺口地图.md` 中该对象的未冻结项
- 必要的 `L2_proposals/` 追溯材料

当前审计仓库没有 `supporting documents/` 目录。除非之后新增该目录，否则不要引用或要求读取不存在的 supporting files。

## New / Old 材料处理

`new system/` 下所有 New System 文档已经作废，不再作为审计目标。

`old_system_nodedesign/` 下 Old-System 文档也不作为审计目标、历史意图权威、标准答案、回归目标、可复用约束来源或评判当前系统的依据。

未经用户明确同意，不得读取 New System 或 Old-System 文档内容。若用户同意读取，需满足：

- 用户明确授权具体读取范围或具体文件。
- 读取目的只能是历史追溯、差异说明或用户指定的比较，不得把 New System 恢复为审计目标。
- 任何看似有用的想法，都必须脱离来源重新按核心产品目标、当前已确认规则和正式 `BK_Copilot/` 草案论证。

## 审计方法

使用第一性原理批判，而不是对某个来源的忠诚。

对每个重要设计对象，追问：

- 它为什么必须存在？
- 它解决了什么无法被更简单方式解决的问题？
- 如果删除、合并或内联，会失去什么能力？
- 失去的能力对核心产品目标是否重要？
- 它是否改善记忆复用、证据支持、纠错学习、审计性、会计师控制权或自动化率？
- 它是否让系统更难理解、验证、审计或控制？
- 它是真实产品必要性，还是架构完整性装饰？

默认可接受的审计结论包括：

- 保留，并说明理由
- 删除
- 合并
- 内联
- 重命名
- 收窄职责
- 只有当拆分能消除真实歧义时才拆分
- 因需要真实产品决策而暂缓

不要因为某个 node、field、log、memory store、phase 或 contract 已经出现在当前设计中，就默认它有效。

## 可编辑范围

默认可编辑的治理 / 审计文档：

- `AGENTS.md`
- `当前任务状态.md`
- `审计阶段路线图.md`
- `审计目标与原则.md`，仅当稳定宪章或长期原则确实改变时
- `项目背景速读.md`，仅当项目背景导览明显过时时
- `系统上下文地图.md`，仅当系统材料入口、结构导航或来源关系变化时
- `缺口地图.md`，仅当 L3 / L4 / seam / L2·外阻缺口状态变化时
- 用户要求创建的新审计报告或聚焦审计笔记

默认不可读、不可编辑的历史材料，除非用户明确授权具体读取或编辑：

- `new system/new_system.md`
- `new system/node_stage_designs/*.md`
- `old_system_nodedesign/*.md`

审计过程中不要静默改写产品设计 spec。如果某个审计发现意味着需要设计变更，应将其报告为 finding、recommended simplification 或 decision point，除非用户明确要求编辑 spec。

## 操作规则

- 始终把审计锚定在核心产品目标上。
- 第一性原理优先于维护当前结构。
- 优先选择能够保留必要能力的最小设计。
- 把审计性和会计师控制权视为产品要求。
- 清楚区分 evidence、memory、judgment、governance 和 audit records。
- 不要仅为对称性、完整性或未来可选性新增组件。
- 不要读取未获授权的 New System 或 Old-System 文档。
- 不要把 New System 或 Old-System 中的结构、字段、流程或能力推广成当前结论。
- 不要把第一版 Stage 3 data contracts 当成冻结实现文档。
- 不要把 AI reasoning 文本变成可复用 authority；可复用 authority 必须来自已批准的 memory、rules、cases、profile 或 governance state。
- 长期性设计结论不写入规则文档；应写入用户指定的长期结论文档、对应主题记录或正式 `BK_Copilot/` 文档。

## 停止 / 升级

遇到以下情况，停止并询问用户：

- 任务需要改变产品设计合同，而不只是审计它们。
- 某个审计发现依赖用户尚未决定的业务优先级。
- 两个权威来源冲突，且无法通过职责范围和 authority 顺序解决。
- 当前工作需要读取 `new system/` 或 `old_system_nodedesign/`，但用户尚未明确授权具体读取范围。
- 当前工作会把 `new system/` 或 `old_system_nodedesign/` 当成审计目标、权威参考、当前有效 baseline 或正确性证明。
- 当前工作会模糊历史材料与当前有效规则 / 正式草案的边界。
- 任务需要编辑 `new system/` 或 `old_system_nodedesign/`，但用户尚未明确要求该具体编辑。

遇到以下情况，宁可暂缓，也不要强行闭环：

- 当前 baseline 需要真实设计决策，而不是措辞清理。
- 某个历史材料或待审对象中的能力是否应保留需要真实产品决策。
- 某个简化会改变审计性、会计师控制权或自动化 authority，而用户尚未批准。

## 文档更新纪律

- 当前状态、下一步行动或 active risks 有实质变化后，更新 `当前任务状态.md`。
- 审计阶段、gate 或成功标准变化后，更新 `审计阶段路线图.md`。
- 项目背景导览明显过时时，更新 `项目背景速读.md`。
- 系统材料入口、结构导航或来源关系变化后，更新 `系统上下文地图.md`。
- 任一对象的 L3 / L4 / seam / L2·外阻缺口状态变化后，更新 `缺口地图.md`。
- `当前任务状态.md` 只保留必要交接和指针，不沉淀长期结论全文。
- `审计目标与原则.md` 保持稳定、低 churn。
- `AGENTS.md` 保持聚焦 agent operation，不放临时交接细节。
- `缺口地图.md` 不记录已收口 L1 / L2 结论。
- `未解决问题暂存清单.md` 不再更新。
- 从已授权读取的 New System / Old-System 历史材料中发现看似有用的想法时，只能标为待论证假设，不得静默当成当前设计规则、intent 或 constraint。

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
