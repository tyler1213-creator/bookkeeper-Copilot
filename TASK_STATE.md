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

- 正在围绕 Entity / Entity Resolution / Case Log 边界做第一性原理讨论。
- Entity 讨论是整个系统 audit 当前聚焦点，不是脱离系统审计的独立产品设计任务。
- 最新 Entity 讨论记录在 `entity的作用和在系统中的创建方法.md`。
- `TEMP_ENTITY_CASE_LOG_HANDOFF.md` 是当前 Entity / Case Log 讨论的临时交接。

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
- 已新增 `entity的作用和在系统中的创建方法.md`，记录当前已确认的 Entity 作用、类型命名、Entity Resolution 判断职责、写入时机未定和后续问题。
- 已新增 `TEMP_ENTITY_CASE_LOG_HANDOFF.md`，作为 Entity / Case Log 讨论的临时交接。

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
- `entity的作用和在系统中的创建方法.md`：Entity 讨论的当前临时记录；若内容已在该文档中，不要重复塞进治理文档。
- `TEMP_ENTITY_CASE_LOG_HANDOFF.md`：Entity / Case Log 讨论交接；新窗口继续相关问题时应读取。
- `old_system_nodedesign/`：旧系统材料，作为历史意图样本。

## 下一步

下一轮审计应：

1. 先读取 `entity的作用和在系统中的创建方法.md`、`TEMP_ENTITY_CASE_LOG_HANDOFF.md` 和 `未解决问题暂存清单.md`。
2. 按照 `未解决问题暂存清单.md` 中的 6 个问题逐个推进讨论。
3. 优先讨论问题 2（Case Judgment 是否必须依赖已识别 entity）和问题 4（unknown entity 后的处理路径），因为这两个问题影响后续多个节点的设计。
4. 继续使用一问一答方式推进；形成确认性结论后先复述给用户确认。
5. 未经用户明确要求，不要改正式 spec；Entity 讨论完成后再统一更新正式文档。
6. 始终把 Entity 讨论锚定为系统 audit 的一部分，用核心产品目标评估必要性、边界和复杂度。

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
- 不要把 `entity的作用和在系统中的创建方法.md` 中已有内容重复长篇搬进治理文档。

## 最近交接
- 已确认 Entity 类型命名：`stable entity` 和 `unknown entity`。Entity Log 只存 stable entity，没有 durable candidate entity lifecycle state。
- 已确认 Entity 是业务对象层面的语义身份，不是字段精确匹配。Entity Resolution 使用 LLM 语义理解判断。
- 已确认 Entity Resolution 可以使用 AI 联网搜索辅助 identity 判断（搜索结果是 evidence 来源，不是 authority）。
- 已确认 Entity Resolution 可以直接将 `new_stable_entity` 同步写入 Entity Log，不需要 governance approval。写入内容限于 entity 本体。
- 已确认 Stable entity 写入 Entity Log 不等于拥有 active rule。每一个 rule 必须经过 accountant 审核才可启用。
- 已确认 Batch 内交易按顺序处理。Entity Resolution 同步写入 stable entity 后，后续交易可自然匹配。
- 已确认 Accountant confirmation scope 需要结构化：区分当前交易处理指示、明确身份确认、长期归属确认。
- 已确认 Case Log 允许不绑定 stable entity 的 case 存在（如 bank fee 等由交易类型决定 COA 的场景）。
- 用户倾向：如果 Entity Resolution 无法识别 stable entity，整笔交易直接 pending，由 Coordinator 问 accountant。但此倾向尚未最终确认，待与 Case Judgment 职责边界一起讨论。
- `未解决问题暂存清单.md` 已写入 6 个用户同意记录的待讨论问题。
- 下一窗口应继续讨论：`未解决问题暂存清单.md` 中的 6 个问题。
