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

- 已完成 `new system_副本/` 当前系统材料的首轮盘点。
- 本轮只做材料理解和系统地图，不做产品设计正确性判断。

已完成：

- `CLAUDE.md` 已改名为 `AUDIT_CHARTER.md`。
- 根部治理文档已中文化。
- 旧系统材料已被明确定位为历史设计意图样本，不是评判标准。
- 当前系统材料已被明确定位为审计对象，不是正确性证明。
- 已删除治理文档中主要会偷渡产品设计判断、成功标准或功能逻辑解释的内容。
- 已读取 `new system_副本/new_system.md`。
- 已盘点 `new system_副本/node_stage_designs/`：共有 15 个 node，每个 node 均有 Stage 1 / Stage 2 / Stage 3 文档，共 45 份节点阶段文档。
- 已抽取各节点 Stage 1 功能意图、Stage 2 逻辑边界目录、Stage 3 contract 结构，以及主要 Open Boundaries。

## 当前目标

让治理文档保持克制：

- 新窗口能快速接续。
- 上下文满时知道如何交接。
- 规则文档不替 agent 解释当前系统功能逻辑。
- 规则文档不预设节点、字段、流程、抽象或拆分是否合理。

## 材料角色

- `AGENTS.md`：操作入口和交接纪律。
- `AUDIT_CHARTER.md`：稳定审计目的与立场约束。
- `PLANS.md`：轻量阶段导航。
- `TASK_STATE.md`：当前状态和下一步。
- `DECISIONS.md`：用户已接受的治理性决策。
- `new system_副本/`：当前系统材料，作为审计对象。
- `old_system_nodedesign/`：旧系统材料，作为历史意图样本。

## 下一步

下一轮审计应：

1. 由用户指定一个具体审计问题、节点或流程；如果继续 Phase 1，则整理当前系统材料地图。
2. 针对用户指定范围，读取对应的 `new system_副本/node_stage_designs/` 原文细节。
3. 如需参考旧系统，先说明读取它是为了理解历史意图，而不是寻找标准答案。

## 当前风险

- 治理文档可能再次膨胀成产品设计指南。
- `PLANS.md` 容易滑向定义审计成功标准。
- `AUDIT_CHARTER.md` 容易滑向解释系统应该如何设计。
- `DECISIONS.md` 容易记录未被用户明确接受的判断。

## 禁止事项

- 不要在规则文档中加入产品功能逻辑细节。
- 不要在规则文档中定义节点、字段、流程或抽象的好坏标准。
- 不要把旧系统当评判标准。
- 不要把当前系统当正确答案。
- 不要在未获得用户明确要求时编辑 `new system_副本/` 或 `old_system_nodedesign/`。

## 最近交接

- 本轮已按用户要求先读取新项目文档，完成当前系统材料的初步了解。
- 读过的当前系统材料包括：`new system_副本/new_system.md`、`new system_副本/node_stage_designs/` 下全部文件清单、各 Stage 1 意图摘要、各 Stage 2 / Stage 3 标题结构与 Open Boundaries。
- 当前理解：新系统文档已形成总纲 + 15 个 workflow node + 9 类 logs/memory stores 的材料结构；节点文档已覆盖 Stage 1/2/3，但仍有不少跨节点 contract、routing、authority、review、logging、case memory 和 implementation 边界保持开放。
- 下一步最具体动作：等待用户指定正式审计方向；若用户要求继续材料盘点，可产出一份不带结论的系统地图。
