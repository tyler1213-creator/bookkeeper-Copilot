# AGENTS.md

## 文件职责

这是本系统审计仓库的 agent 操作入口。

它只回答五件事：

- 新窗口进入时先读什么
- 各类材料的权威顺序是什么
- 当前系统和旧系统分别该如何对待
- 什么情况下可以编辑、必须停止或必须记录交接
- 上下文快满或任务阶段变化时，应如何更新交接文档

它不负责解释当前系统功能逻辑，也不负责定义产品设计的好坏标准。

## 权威顺序

当文档之间出现冲突时，按以下顺序解决：

1. 用户的明确指令
2. `AGENTS.md`
3. `TASK_STATE.md`
4. `AUDIT_CHARTER.md`
5. `PLANS.md`
6. `DECISIONS.md`
7. `new system_副本/` 下的当前系统材料
8. `old_system_nodedesign/` 下的旧系统材料

每份文档只在自己的职责范围内有效：

- `AGENTS.md`：操作规则、必读顺序、编辑边界、停止条件、交接纪律。
- `TASK_STATE.md`：当前状态、当前目标、下一步、当前风险。
- `AUDIT_CHARTER.md`：稳定审计目的和不预设立场的原则。
- `PLANS.md`：轻量阶段导航，不定义产品设计细节。
- `DECISIONS.md`：已被用户接受的治理性决策。
- `new system_副本/`：当前被审计对象，不是正确性证明。
- `old_system_nodedesign/`：历史设计意图样本，不是评判标准。

## 必读材料

任何实质性工作前，先读：

1. `TASK_STATE.md`
2. `AUDIT_CHARTER.md`
3. `PROJECT_BRIEF.md`
4. `AGENTS.md`

如果任务涉及审计当前系统，再读：

1. `new system_副本/new_system.md`
2. 与任务直接相关的 `new system_副本/node_stage_designs/*__design.md` 合并文档
3. 必要时再读相关 `old_system_nodedesign/` 文件，用于理解历史意图，而不是作为答案

`new system_副本/node_stage_designs/` 现在每个 node 只保留一份 `__design.md` 合并摘要。不要再寻找或要求读取旧的 Stage 1 / Stage 2 / Stage 3 文件，除非用户明确要求通过 git 历史追溯旧全文。

如果任务涉及治理文档修改，再读：

- `PLANS.md`
- `DECISIONS.md`

当前仓库没有 `supporting documents/` 目录。除非之后新增该目录，否则不要要求读取不存在的 supporting files。

## 材料使用规则

- 当前系统材料只说明“当前设计如何表达自己”，不能证明设计正确。
- 旧系统材料只帮助理解原始意图、心智模型和用户曾经重视的能力，不能作为评判标准。
- 不要在治理文档中写入对当前功能逻辑的解释、成功标准或设计细节判断。
- 如果审计时需要判断某个节点、字段、流程或抽象是否合理，应从用户当前问题和源设计材料中现场推理，而不是引用治理文档里的预设结论。

## 编辑边界

默认可编辑：

- `AGENTS.md`
- `TASK_STATE.md`
- `AUDIT_CHARTER.md`
- `PLANS.md`
- `DECISIONS.md`
- 用户明确要求创建或修改的审计报告 / 交接笔记

默认只读，除非用户明确要求编辑：

- `new system_副本/new_system.md`
- `new system_副本/node_stage_designs/*.md`
- `old_system_nodedesign/*.md`

审计过程中不要静默改写产品设计 spec。发现问题时，先作为审计发现、风险、疑问或建议报告。

## 停止条件

遇到以下情况，停止并询问用户：

- 任务需要改变产品设计内容，而用户只要求审计或排查。
- 两个权威来源冲突，且无法按权威顺序解决。
- 当前工作会把旧系统材料当成当前 baseline。
- 当前工作会把治理文档写成产品设计解释。
- 任务需要大范围重写 `new system_副本/`，但用户没有明确要求。

## 交接纪律

当当前窗口完成了有意义的工作、上下文快满、或下一步行动发生变化时，更新 `TASK_STATE.md`。

`TASK_STATE.md` 至少应记录：

- 当前正在做什么
- 已经完成什么
- 哪些文件被改过或读过
- 下一步最具体应该做什么
- 仍有哪些风险、疑问或禁止事项

只有在用户明确接受某个治理性决策时，才更新 `DECISIONS.md`。

只有在阶段划分或阶段位置真的变化时，才更新 `PLANS.md`。

不要把临时讨论、产品细节判断或审计结论塞进 `AGENTS.md` 或 `AUDIT_CHARTER.md`。

## 输出要求

最终回复必须说明：

- 文件改动
- 改动原因
- 做过的验证
- 剩余风险 / 后续事项

如果没有运行验证，要明确说明。
