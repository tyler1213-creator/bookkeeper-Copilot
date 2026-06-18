# 00 规则蒸馏（Coordinator 审计）

> 用途：第 2 步梳理问题的标尺 + Proposal 自检表。来源逐条标出。

## A. 必守通用规则（逐条 + 出处）

1. **Source authority 铁律**：用户指令最高；其余文档只在职责范围内有 authority；`new system/`、`old_system_nodedesign/` 不进 authority，只能说明材料如何表达自己，不能证明某设计正确/必要/应保留/应回归/应作 baseline。（`AGENTS.md` §Source Authority / §被审计材料处理；`审计目标与原则.md` §仓库边界）
2. **契约面纪律**：节点只通过显式、最小、写明的输出契约面与下游交互；下游只能依赖声明的输出类别，不得依赖内部状态/实现。字段级 schema 冻结属 Stage 3，更早阶段只声明语义契约面并朝它设计。（模板 §2；`审计目标与原则.md` 稳定原则末条）
3. **运行/记忆 seam**：运行层节点只声明「要持久化什么 + 谁有权威认定它有效」，不声明「怎么写、谁来写、什么顺序写」（后者属记忆层 spec）。（模板 §2、§7.4）
4. **第一性原理**：每个 node/field/log/step 必须自证为何存在；删除/合并/内联不实质损害核心产品目标者，缺乏独立存在理由。审计性与会计师控制权是核心能力，非装饰。（`审计目标与原则.md` §稳定审计原则；`AGENTS.md` §审计方法）
5. **核心产品目标（唯一标尺）**：把会计师历史账务与判断沉淀成可复用记忆，让新交易自动获得有证据支持的记账建议，并通过纠正持续学习，在不牺牲审计性与控制权的前提下提高自动化率。（`审计目标与原则.md` §核心产品目标）
6. **已锁不变量（遵守，非审查对象）**：身份判断 `stable`/`unknown` 锁在 ER 侧；会计分类归 Case Judgment；AI 联网/external lookup 身份用途锁在 ER 侧、会计事实用途锁在 CJ 侧；身份缺口记录边界见 Case Log/Transaction Log。（各正式草案 + Coordinator Question.md）
7. **停止/升级**：需真实产品决策、权威冲突无法按职责顺序解决、任一关键 A 源无法定位、output consumer 无法从已锁结论确定、B 与 A 冲突 → 停下报告，以 A 为准，不得用 B 填。（模板 §七；`AGENTS.md` §停止/升级）
8. **B 类纪律**：`new system/coordinator_pending_node__design.md`、`old_system_nodedesign/coordinator_agent_spec_v3.md` 只作热身/问题发生器；绝不继承其字段名/枚举/状态机/输入输出形状，绝不作正确性证明/baseline。（模板 §一 B 类；`AGENTS.md`）
9. **输出纪律**：本 Agent 只产出 L2 Proposal + `_audit_work/` 工作区文件；不创建/编辑 BK_Copilot 正式文档，不改已锁文件，不碰 new/old system。（模板 §六、§五）

## B. 模板 Stage 1-2 完成清单（逐字搬运，作 Proposal 自检表）

来源：`workflow_node_spec_template_rules.md` §10。一份 Stage 1-2 node spec 完成前必须能回答：

1. 这个节点为什么存在？
2. 为什么不能删除、合并或内联？
3. 它的唯一核心职责是什么？
4. 它上游依赖什么？
5. 它下游影响什么？
6. 它读取哪些 logs / memory stores？
7. 它直接写什么？
8. 它只能提出什么 candidate？
9. 它绝不能写什么？
10. deterministic code 可以决定什么？
11. LLM 可以判断什么？
12. accountant 必须决定什么？
13. governance 必须批准什么？
14. 证据不足时怎么处理？
15. 歧义时怎么处理？
16. source conflict 时怎么处理？
17. 哪些 trace 会留下？
18. 哪些 trace 不能成为 authority？
19. 是否声明了对外契约面，以及每个输出的 consumer？
20. 是否守住运行/记忆 seam——只声明要持久化什么 + 谁有权威，没去定写入机制？
21. 哪些问题未冻结？
22. 是否可以进入 Stage 3？

## C. 禁止写法（来源 §9，Proposal 不得出现）

- "协调一切相关逻辑" / "必要时可更新相关 memory" / "LLM 根据上下文自行判断" / "高置信度则自动通过" / "后续实现时再决定" / "参考旧系统处理"。
- "写入日志" 但不说哪个 log/字段/authority；"生成候选" 但不说能否持久化、谁批准、谁消费。
