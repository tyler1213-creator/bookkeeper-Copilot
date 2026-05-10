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

- 已按用户要求瘦身 `new system_副本/node_stage_designs/`。
- 原 15 个 node × 3 份 Stage 文档已合并为 15 份 `__design.md` 文档。
- 本轮是文档压缩与结构合并，不做产品设计正确性判断。

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
- `old_system_nodedesign/`：旧系统材料，作为历史意图样本。

## 下一步

下一轮审计应：

1. 由用户指定一个具体审计问题、节点或流程；如果继续 Phase 1，则整理当前系统材料地图。
2. 针对用户指定范围，读取对应的 `new system_副本/node_stage_designs/*__design.md` 合并文档。
3. 如需参考旧系统，先说明读取它是为了理解历史意图，而不是寻找标准答案。
4. 若用户觉得合并摘要仍然过长，继续压缩 `__design.md`；不要重新生成 45 份 Stage 文档，除非用户明确要求。

## 当前风险

- 治理文档可能再次膨胀成产品设计指南。
- `PLANS.md` 容易滑向定义审计成功标准。
- `AUDIT_CHARTER.md` 容易滑向解释系统应该如何设计。
- `DECISIONS.md` 容易记录未被用户明确接受的判断。
- 合并文档是瘦身摘要，不再逐字保留所有原 Stage 文档解释性文本；如需追溯旧全文，应通过 git 历史查看被删除的 Stage 文件。

## 禁止事项

- 不要在规则文档中加入产品功能逻辑细节。
- 不要在规则文档中定义节点、字段、流程或抽象的好坏标准。
- 不要把旧系统当评判标准。
- 不要把当前系统当正确答案。
- 不要在未获得用户明确要求时编辑 `new system_副本/` 或 `old_system_nodedesign/`。
- 不要重新生成 45 份 Stage 文档，除非用户明确要求。

## 最近交接
- 本窗口已按 `AGENTS.md` 必读顺序读取 `TASK_STATE.md`、`AUDIT_CHARTER.md`、`PROJECT_BRIEF.md`、`AGENTS.md`，并补读 `PLANS.md`、`DECISIONS.md` 以确认阶段与已接受治理决策。
- 当前任务理解：先保持 Phase 1 的材料盘点 / 系统地图状态，等待用户指定具体审计问题、节点或流程；在进入具体审计前再读取 `new system_副本/new_system.md` 和对应 node 的 `__design.md`。
- 本轮已按用户要求先读取新项目文档，完成当前系统材料的初步了解。
- 读过 / 改过的当前系统材料包括：`PROJECT_BRIEF.md`、`new system_副本/new_system.md`、原 `new system_副本/node_stage_designs/` 下全部 Stage 文件结构与样本文档。
- 当前理解：新系统文档已形成总纲 + 15 个 workflow node + 9 类 logs/memory stores 的材料结构；节点文档已覆盖 Stage 1/2/3，但仍有不少跨节点 contract、routing、authority、review、logging、case memory 和 implementation 边界保持开放。
- 本轮最新动作：45 份原 Stage 文件已删除，替换为 15 份 `__design.md` 合并摘要。
- 之后新增 `SYSTEM_CONTEXT_SUMMARY.md`，记录新系统主流程、旧系统主流程、新旧差异、旧系统仍值得保留的约束和后续审计使用方式。
