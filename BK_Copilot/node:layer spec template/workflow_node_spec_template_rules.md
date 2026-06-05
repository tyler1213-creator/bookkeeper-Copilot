# Workflow Node Spec Template Rules

## 1. 文件角色

本文档定义 AI Bookkeeper 最新标准 `workflow node spec` 的模板规则。

它用于在节点审计结论被用户接受后，重构该节点的正式标准设计文档。

它不是审计记录，不记录某次审计过程，也不保存临时讨论结论。
按本模板完成并被用户接受后的 node spec，才应作为该节点的当前标准设计文档。

## 2. 第一性原理

每份 workflow node spec 必须服务于两个目标：

1. 人类能在重读后恢复设计意图、边界决策和关键细节。
2. 编码代理能在不猜测产品逻辑、权限边界、数据契约或验证要求的情况下执行后续任务。

因此文档必须回答：

- 这个节点为什么必须存在？
- 它解决什么不能被更简单方式解决的问题？
- 它读什么？
- 它写什么？
- 它可以决定什么？
- 它绝不能决定什么？
- 哪些变化需要 accountant 或 governance approval？
- 证据缺失、歧义、冲突时怎么处理？
- 哪些部分已经成为当前标准？
- 哪些部分仍未冻结，不能交给实现代理猜？

此外，每份 node spec 必须遵守两条跨节点纪律：

- 契约面：本节点只通过显式、最小、写明的输出契约面与下游交互；下游只能依赖此处声明的输出，不得依赖本节点的内部状态或实现。字段级 schema 的最终冻结属于 Stage 3，更早阶段只需声明语义契约面并朝它设计。
- 运行 / 记忆 seam：本节点（运行层）只声明「要持久化什么 + 谁有权威认定它有效」，不声明「怎么写、谁来写、什么顺序写」；后者属于记忆层 spec。

如果某个设计对象只因为架构完整性、对称性或未来可能性而存在，它不应进入标准文档。

## 3. 文档成熟度

当前系统重构阶段默认目标是：

```text
Stage 1: Functional Intent
Stage 2: Logic and Boundaries
Stage 3: Data Contract Draft only when interface is stable
```

不要默认写到 Stage 4-7。
Stage 4-7 只有在用户明确批准进入实现准备时才写。

| Stage | 何时写 | 目标 |
| --- | --- | --- |
| Stage 1 | 每个保留节点必须写 | 固定为什么存在、核心职责和排除范围 |
| Stage 2 | 每个保留节点必须写 | 固定触发、读写、权限、边界、异常行为 |
| Stage 3 | 只有接口足够稳定时写 | 固定 input/output、字段、enum、验证规则 |
| Stage 4 | 实现前才写 | 固定执行算法和 routing sequence |
| Stage 5 | 已有 repo 实现目标时写 | 固定技术映射、文件路径、依赖、storage |
| Stage 6 | 实现前/实现中写 | 固定测试、fixtures、验证命令 |
| Stage 7 | 准备交给编码代理时写 | 固定任务边界、验收标准、stop conditions |
| Stage 8 | 实现后维护时写 | 记录实现后标准、弃用、维护要求 |

## 4. 文件夹结构

重要 workflow node 使用文件夹，不使用超大单文件。

```text
BK_Copilot/workflow_nodes/[workflow_node_name]/
  00_index.md
  01_functional_intent.md
  02_logic_and_boundaries.md
  03_data_contract.md
  04_execution_algorithm.md
  05_technical_implementation.md
  06_tests_and_fixtures.md
  07_agent_task_contract.md
  08_maintenance.md
```

只创建已经成熟到值得写下来的文件。
不要为了结构完整创建空文档。

## 5. `00_index.md` 模板

```markdown
# [Node Name]

## 文档状态

- Current standard status: draft / accepted / superseded
- Covered stages:
  - [ ] Stage 1: Functional Intent
  - [ ] Stage 2: Logic and Boundaries
  - [ ] Stage 3: Data Contract
  - [ ] Stage 4: Execution Algorithm
  - [ ] Stage 5: Technical Implementation
  - [ ] Stage 6: Tests and Fixtures
  - [ ] Stage 7: Agent Task Contract
  - [ ] Stage 8: Maintenance
- Last reviewed:
- Owner / reviewer:

## 当前标准文件

- `01_functional_intent.md`
- `02_logic_and_boundaries.md`
- `03_data_contract.md`，如果已存在

## 当前未冻结边界

- [未冻结问题 1]
- [未冻结问题 2]

## 进入下一阶段前必须解决

- [gate 1]
- [gate 2]
```

## 6. `01_functional_intent.md` 模板

```markdown
# [Node Name] - Functional Intent

## 1. 为什么这个节点必须存在

这个节点解决的问题是：

> [一句话]

如果删除、合并或内联它，会失去：

- [能力 1]
- [能力 2]

这些能力对核心产品目标重要，因为：

- [说明]

## 2. 核心职责

本节点的唯一核心职责是：

> [一句话]

本节点可以辅助产生：

- [辅助 context / signal]

但这些不是主职责。

## 3. 明确排除范围

本节点不负责：

- [事项 1]
- [事项 2]
- [事项 3]

本节点绝不能：

- [越权行为 1]
- [越权行为 2]

## 4. Workflow 位置

上游：

- [upstream node / source]

下游：

- [downstream node / consumer]

本节点位于流程中的原因：

- [说明]

## 5. 对核心产品目标的贡献

本节点必须清楚支持以下至少一项：

- [ ] 记忆复用
- [ ] 有证据支持的建议
- [ ] accountant correction learning
- [ ] 审计性
- [ ] accountant control
- [ ] 自动化率提升

具体贡献：

- [说明]

## 6. 已知约束

- [约束 1]
- [约束 2]

## 7. 未决定问题

- [问题 1]
- [问题 2]
```

## 7. `02_logic_and_boundaries.md` 模板

```markdown
# [Node Name] - Logic and Boundaries

## 1. 触发条件

本节点在以下条件同时满足时触发：

- [条件 1]
- [条件 2]

本节点不得在以下情况触发：

- [情况 1]
- [情况 2]

## 2. 上游前置条件

上游必须已经完成：

- [前置条件 1]
- [前置条件 2]

如果前置条件缺失：

- 本节点行为：
- 是否 stop-and-ask：

## 3. 读取对象

删除不适用的 source；不要保留空行。

| Source | 读取内容 | 用途 | Authority 限制 |
| --- | --- | --- | --- |
| Evidence Log | | evidence | 不保存业务结论 |
| Profile | | structural context | candidate profile fact 不能当 stable truth |
| Entity Log | | entity authority/context | summary 不能覆盖 Entity Log |
| Case Log | | precedent | case 不能当 rule |
| Rule Log | | approved rule context | candidate rule 不能执行 |
| Governance Log | | approved/applied authority | pending/rejected event 不能当 authority |
| Intervention Log | | correction/review context | 不能直接变成 active memory |
| Knowledge Summary | | readable context | 不能替代 source authority |

## 4. 写入对象

### 直接执行的 durable write（例外）

本节点直接执行的 durable 写入（应是少数例外；多数节点这里写“无”）：

- [store / record；如果无，写“无”]

### 交给记忆 / finalization 层持久化的内容（运行 / 记忆 seam）

本节点产出、但由记忆层执行写入的内容：本节点只声明「存什么 + 谁有权威认定它有效」，**不**声明「怎么写、谁来写、什么顺序写」（机制属于对应记忆层 spec）。

- [要持久化的内容 + 其 authority 来源；如果无，写“无”]

### 只能提出 candidate

- [candidate 1]
- [candidate 2]

### 绝不能写入或修改

- [forbidden store 1]
- [forbidden store 2]

## 5. 决策权限

### Deterministic code 可以决定

- [决定 1]
- [决定 2]

### LLM 可以判断

- [语义判断 1]
- [语义判断 2]

### LLM 不能判断

- [越权判断 1]
- [越权判断 2]

### Accountant 必须决定

- [人工决定 1]
- [人工决定 2]

### Governance 必须批准

- [长期 authority 变化 1]
- [长期 authority 变化 2]

## 6. 输出类别

字段名可以暂不冻结，但语义类别必须稳定。

本表是本节点对下游唯一的契约面：下游只能依赖此处声明的输出类别，不得依赖本节点未声明的内部状态或实现。

| Output Category | 含义 | Consumer（谁消费） | 下游影响 | 不代表什么 |
| --- | --- | --- | --- | --- |
| [category] | | | | |
| [category] | | | | |

## 7. 证据不足时的行为

如果缺少 [信息]：

- 输出：
- 下游应：
- 本节点不能：

## 8. 歧义处理

如果存在多个合理候选：

- 输出：
- 是否允许自动化：
- 是否需要 pending / review / governance：

本节点不能为了让 workflow 继续而猜一个 winner。

## 9. 冲突处理

如果 [source A] 与 [source B] 冲突：

- authority 顺序：
- 本节点行为：
- 是否生成 review / governance candidate：
- 是否阻断自动化：

## 10. Audit / Trace 边界

本节点应保留的 trace：

- [trace 1]
- [trace 2]

这些 trace 用于：

- review
- correction
- governance
- audit

这些 trace 不能成为：

- entity authority
- rule authority
- case authority
- accountant approval
- governance approval

## 11. Legacy Constraint Translation

仅当旧系统约束仍服务当前产品目标时填写。

| Retained Constraint | 来源 | 为什么现在仍成立 |
| --- | --- | --- |
| [constraint] | [old file / concept] | |

不保留的旧行为：

| Old Behavior | 不保留原因 |
| --- | --- |
| [behavior] | |

## 12. Open Boundaries

以下问题未冻结：

1. [问题 1]
2. [问题 2]

这些问题解决前，不能进入：

- [ ] Stage 3 data contract
- [ ] Stage 4 execution algorithm
- [ ] implementation
```

## 8. `03_data_contract.md` 模板

只有当接口已经稳定时才创建此文件。

````markdown
# [Node Name] - Data Contract

## 1. Contract Status

- Status: draft / accepted
- Depends on:
- Consumers:
- Not implementation-ready because:

## 2. Input Objects

| Input Object | Required | Source of Truth | Runtime or Durable | Notes |
| --- | --- | --- | --- | --- |
| [input] | yes/no | | | |

## 3. Output Objects

| Output Object | Required | Consumer | Runtime or Durable | Notes |
| --- | --- | --- | --- | --- |
| [output] | yes/no | | | |

## 4. Field Definitions

### `[object_name]`

| Field | Required | Type / Values | Source / Authority | Meaning | Must Not Mean |
| --- | --- | --- | --- | --- | --- |
| [field] | yes/no | | | | |

## 5. Enums / States

```text
[enum_name]:
- value_1
- value_2
```

## 6. Validation Rules

Valid input requires:

- [rule 1]
- [rule 2]

Valid output requires:

- [rule 1]
- [rule 2]

Invalid output includes:

- [invalid case 1]
- [invalid case 2]

## 7. Durable Memory Effect

This node output:

- [ ] writes durable memory
- [ ] only carries runtime handoff
- [ ] only proposes candidate signals

If durable write is allowed, specify exact store and approval authority:

- Store:
- Approval:
- Mutation type:

## 8. Examples

### Valid Example

```yaml
# example
```

### Invalid Example

```yaml
# example
```

Reason invalid:

- [reason]

## 9. Open Contract Boundaries

- [unfrozen field / enum / routing question]
````

## 9. 禁止写法

标准 workflow node spec 中不要出现以下写法：

- “这个节点负责协调一切相关逻辑。”
- “必要时可更新相关 memory。”
- “LLM 根据上下文自行判断。”
- “如果高置信度则自动通过。”
- “后续实现时再决定。”
- “参考旧系统处理。”
- “写入日志”，但不说明写入哪个 log、什么字段、什么 authority。
- “生成候选”，但不说明候选能否持久化、由谁批准、谁消费。

这些写法会让实现代理自行补设计，必须改成明确边界。

## 10. Stage 1-2 完成检查

一份 Stage 1-2 workflow node spec 完成前，必须能回答：

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
19. 你是否声明了对外契约面，以及每个输出的 consumer？
20. 你是否守住运行 / 记忆 seam——只声明要持久化什么 + 谁有权威，没有去定写入机制？
21. 哪些问题未冻结？
22. 是否可以进入 Stage 3？

如果不能回答，不要进入数据契约或实现阶段。
