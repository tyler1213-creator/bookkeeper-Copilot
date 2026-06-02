# TASK_STATE.md

## 文件职责

`TASK_STATE.md` 是新窗口最快理解当前情况的入口。

它只记录：

- 当前在做什么
- 已经完成什么
- 下一步最具体该做什么
- 当前风险和禁止事项
- 上一个窗口留下的必要交接

它不负责长期规则、阶段理论、产品功能解释或审计结论沉淀。

## 当前状态

当前阶段：

- Phase 1，材料盘点与系统地图

当前工作：

- 正在围绕 Entity / Entity Resolution / Case Log 边界做第一性原理讨论，并按用户确认结果瘦身 alias / role / write path 相关设计。
- Entity 讨论是整个系统 audit 当前聚焦点，不是脱离系统审计的独立产品设计任务。
- 最新 Entity 讨论记录在 `Entity相关的所有问题.md`。

已完成：

- `CLAUDE.md` 已改名为 `AUDIT_CHARTER.md`。
- 已新增 `PROJECT_BRIEF.md`，作为新窗口快速理解 AI Bookkeeping 项目背景的短文档。
- 根部治理文档已中文化。
- 旧系统材料已被明确定位为历史设计意图样本，不是评判标准。
- 当前系统材料已被明确定位为审计对象，不是正确性证明。
- 已删除治理文档中主要会偷渡产品设计判断、成功标准或功能逻辑解释的内容。
- 已读取 `new system_副本/new_system.md`。
- 已盘点 `new system_副本/node_stage_designs/`：共有 15 个 node。
- 已将每个 node 的 Stage 1 / Stage 2 / Stage 3 合并为一份对应的 `*_node__design.md`。
- 合并文档保留定位、逻辑边界、contract 字段摘要和 Open Boundaries；删除阶段说明、examples、self-review、历史读档记录和重复解释。
- 已新增 `SYSTEM_CONTEXT_SUMMARY.md`，提炼当前新系统、旧系统和两者关键差异，供新窗口快速建立上下文。
- 已将最新标准 spec 模板规则放入 `BK_Copilot/node_specs/`，并拆分为 workflow node 与 memory layer 两份模板规则文件。
- 已按 workflow node 标准模板新增 `BK_Copilot/node_specs/workflow_nodes/entity_resolution_node/`，目前只覆盖 Stage 1-2，不创建 Stage 3 data contract。
- 已新增 `Entity相关的所有问题.md`，记录当前已确认的 Entity 作用、类型命名、Entity Resolution 判断职责、写入时机未定和后续问题。
- `TEMP_ENTITY_CASE_LOG_HANDOFF.md` 的内容已吸收进 `Entity相关的所有问题.md`。
- 已解决 Entity 未解决清单中的问题 2、问题 3 和问题 4。
- 已确认问题 5 不属于 Entity 本身，标注为后续处理 Coordinator 时处理。
- 已确认 Alias 的当前窄定义：Alias 是过去已经确认过的 transaction surface text 和 `stable entity` 的对应关系；当前只确认 bank statement description / descriptor / raw bank surface text，以及 bank description 无明确身份意义时的可重复主体字段（例如 cheque payee）可能成为 Alias。
- 已确认需要一个可被 Entity Resolution 查询的 Alias 库；具体技术形态后续再定。
- 已将 BK_Copilot 正式 Entity Log / Entity Resolution 文档对齐到最新 Entity 讨论：Entity Resolution 可以同步写入 `new_stable_entity` entity 本体，Entity Log 不保存 durable candidate entity lifecycle state。

## 当前目标

让治理文档保持克制：

- 新窗口能快速接续。
- 上下文满时知道如何交接。
- 规则文档不替 agent 解释当前系统功能逻辑。
- 规则文档不预设节点、字段、流程、抽象或拆分是否合理。

## 材料角色

- `AGENTS.md`：操作入口和交接纪律。
- `AUDIT_CHARTER.md`：稳定审计目的与立场约束。
- `PROJECT_BRIEF.md`：项目背景速读。
- `SYSTEM_CONTEXT_SUMMARY.md`：新系统、旧系统和关键差异的高密度上下文速读。
- `PLANS.md`：轻量阶段导航。
- `TASK_STATE.md`：当前状态和下一步。
- `DECISIONS.md`：用户已接受的治理性决策。
- `new system_副本/`：当前系统材料，作为审计对象。
- `BK_Copilot/node_specs/workflow_node_spec_template_rules.md`：后续最新标准 workflow node spec 的模板规则。
- `BK_Copilot/node_specs/memory_layer_spec_template_rules.md`：后续最新标准 memory layer spec 的模板规则。
- `BK_Copilot/node_specs/workflow_nodes/entity_resolution_node/`：Entity Resolution Node 的当前标准草案，覆盖 Functional Intent 与 Logic and Boundaries。
- `Entity相关的所有问题.md`：Entity 讨论的当前临时记录；若内容已在该文档中，不要重复塞进治理文档。
- `Alias question.md`：当前 Alias 窄定义、两类 Alias 来源、Entity Resolution 使用方式和 Alias 库必要性的临时确认记录。
- `TEMP_ENTITY_CASE_LOG_HANDOFF.md`：已不存在，内容已吸收进 `Entity相关的所有问题.md`。
- `old_system_nodedesign/`：旧系统材料，作为历史意图样本。

## 下一步

下一轮审计应：

1. 先读取 `Entity相关的所有问题.md`、`Alias question.md` 和 `未解决问题暂存清单.md`。
2. 继续处理问题 1 的剩余部分：创建 stable entity 时当前 surface text 是否进入 Alias 库；Alias 库具体技术形态；role 和 automation policy 的写入路径。
3. 问题 5 已标注为处理 Coordinator 时处理，不在 Entity 问题中继续展开。
4. 后续再处理问题 6（同 batch unknown entity 去重）和问题 7（交易完成分类后的多 log 统一写入机制）。
5. 继续使用一问一答方式推进；形成确认性结论后先复述给用户确认。
6. 后续问题解决后同样需要更新 `Entity相关的所有问题.md`、`BK_Copilot/` 中受影响的正式边界文档、`TASK_STATE.md` 和 `未解决问题暂存清单.md`。
7. 始终把 Entity 讨论锚定为系统 audit 的一部分，用核心产品目标评估必要性、边界和复杂度。

## 当前风险

- 治理文档可能再次膨胀成产品设计指南。
- `PLANS.md` 容易滑向定义审计成功标准。
- `AUDIT_CHARTER.md` 容易滑向解释系统应该如何设计。
- `DECISIONS.md` 容易记录未被用户明确接受的判断。
- 合并文档是瘦身摘要，不再逐字保留所有原 Stage 文档解释性文本；如需追溯旧全文，应通过 git 历史查看被删除的 Stage 文件。
- Entity 讨论仍有未决问题，不能把临时记录当成正式 spec 或冻结 data contract。

## 禁止事项

- 不要在规则文档中加入产品功能逻辑细节。
- 不要在规则文档中定义节点、字段、流程或抽象的好坏标准。
- 不要把旧系统当评判标准。
- 不要把当前系统当正确答案。
- 不要在未获得用户明确要求时编辑 `new system_副本/` 或 `old_system_nodedesign/`。
- 不要重新生成 45 份 Stage 文档，除非用户明确要求。
- 不要把未确认的 Entity 判断写入 `DECISIONS.md`。
- `未解决问题暂存清单.md` 只记录用户明确同意写入的问题，不得自行添加。
- 不要把 `Entity相关的所有问题.md` 中已有内容重复长篇搬进治理文档。

## 最近交接
- 已确认 Entity 类型命名：`stable entity` 和 `unknown entity`。Entity Log 只存 stable entity，没有 durable candidate entity lifecycle state。
- 已确认 Entity 是业务对象层面的语义身份，不是字段精确匹配。Entity Resolution 使用 LLM 语义理解判断。
- 已确认 Entity Resolution 可以使用 AI 联网搜索辅助 identity 判断（搜索结果是 evidence 来源，不是 authority）。
- 已确认 Entity Resolution 可以直接将 `new_stable_entity` 同步写入 Entity Log，不需要 governance approval。写入内容限于 entity 本体。
- 已同步正式文档：该写入例外不包含 Alias、confirmed role、automation policy 或 rule；candidate identity signal 不作为 Entity Log 的 durable lifecycle state。
- 已确认 Stable entity 写入 Entity Log 不等于拥有 active rule。每一个 rule 必须经过 accountant 审核才可启用。
- 已确认 Batch 内交易按顺序处理。Entity Resolution 同步写入 stable entity 后，后续交易可自然匹配。
- 已确认 Accountant confirmation scope 需要结构化：区分当前交易处理指示、明确身份确认、长期归属确认。
- 已确认 Case Log 允许不绑定 stable entity 的 case 存在（如 bank fee 等由交易类型决定 COA 的场景）。
- **已解决 问题 2**：Case Judgment 的高置信度自动分类通道必须依赖已识别的 entity。Entity Resolution 输出 unknown_entity 时，Case Judgment 不走高置信度分类通道，但仍然处理这笔交易，输出 pending。Pending 分两种：完全无法判断（直接问 accountant）和有推断但不确定（附带推断性建议让 accountant 选择）。不依赖 entity 的交易（bank fee 等）属于 edge case，大部分由 Profile 上游处理。
- **已解决 问题 3**：Entity Resolution 输出 unknown_entity 时，可以把所有与 identity 判断相关的证据、线索和判断说明作为 runtime context 传给 Case Judgment；这些信息不是 identity authority，不能支持 Rule Match 或高置信自动分类。
- **已解决 问题 4**：unknown_entity 后进入 Case Judgment pending，再由 Coordinator 向 accountant 提问；accountant 明确确认 identity 后直接创建 stable entity，交易不重新进入 Entity Resolution 或 Case Judgment。
- **问题 5 已标注延期**：Coordinator 结构化问题设计在处理 Coordinator 时处理，不作为 Entity 当前问题继续展开。
- **问题 1 部分确认**：Alias 是过去已经确认过的 transaction surface text 和 `stable entity` 的对应关系；当前只确认两类来源：bank statement 的 description / descriptor / raw bank surface text，以及 bank description 无明确身份意义时的可重复主体字段（例如 cheque payee）。已确认需要一个可被 Entity Resolution 查询的 Alias 库，具体技术形态后续再定。仍需继续讨论 stable entity 创建时当前 surface text 是否进入 Alias 库、role / automation policy 写入路径。
