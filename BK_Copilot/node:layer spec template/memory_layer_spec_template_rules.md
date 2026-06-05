# Memory Layer Spec Template Rules

## 1. 文件角色

本文档定义 AI Bookkeeper 最新标准 `memory layer spec` 的模板规则。

它用于在 memory / log / durable store 的审计结论被用户接受后，重构该 memory layer 的正式标准设计文档。

它不是 workflow node 模板，也不是审计记录。
memory layer spec 的核心问题不是“这个步骤怎么运行”，而是“什么信息可以长期保存、谁有权改、谁能读、它能否成为 authority”。

## 2. 第一性原理

每份 memory layer spec 必须证明：

- 这层 memory 保存的信息为什么需要 durable persistence。
- 删除它会损失什么无法由 runtime handoff 或现有 store 覆盖的能力。
- 它保存什么，不保存什么。
- 哪些字段是 authority，哪些只是 trace / candidate / summary。
- 谁可以写入、谁可以修改、谁只能提出 candidate。
- accountant correction 或 governance approval 如何改变 durable state。
- 与其他 memory/log 的边界是否已经稳定。

此外，每份 memory layer spec 必须遵守两条跨节点纪律：

- 契约面：本层通过显式、最小、写明的读取者 / 写入者接口面对外交互；reader 只能依赖此处声明的接口，不得依赖本层内部实现。字段级 schema 的最终冻结属于 M3。
- 运行 / 记忆 seam：本层（记忆层）只定义「怎么写、谁来写、什么顺序写」（机制），**不**重新判断「该不该写、内容对不对」；后者由发起写入的运行节点 spec 决定，本层只校验 finalization proof / authority。

如果某层 memory 只是为了架构对称、未来可能查询、或让某个 node 看起来完整而存在，它不应进入标准文档。

## 3. 当前阶段写作深度

当前审计阶段默认只写：

```text
Stage M1: Memory Intent
Stage M2: Authority, Lifecycle, and Boundaries
Stage M3: Data Contract Draft only when stable
```

不要为了完整性提前冻结字段、跨 memory 边界或 governance path。
未冻结的依赖必须写成 Open Boundary，不能伪装成标准。

| Stage | 何时写 | 目标 |
| --- | --- | --- |
| M1 | 每个保留 memory layer 必须写 | 固定为什么存在、保存什么、不保存什么 |
| M2 | 每个保留 memory layer 必须写 | 固定读写 authority、candidate 边界、lifecycle、已知冲突行为 |
| M3 | 只有字段和消费者稳定时写 | 固定 field、enum、mutation type、validation rule |
| M4 | 实现前才写 | 固定 storage、index、migration、retention、access pattern |
| M5 | 实现前/实现中写 | 固定 tests、fixtures、audit verification |

## 4. 文件夹结构

```text
BK_Copilot/memory_layers/[memory_layer_name]/
  00_index.md
  01_memory_intent.md
  02_authority_lifecycle_and_boundaries.md
  03_data_contract.md
  04_storage_and_operations.md
  05_tests_and_fixtures.md
```

只创建已经成熟到值得写下来的文件。
不要为了结构完整创建空文档。

## 5. `00_index.md` 模板

````markdown
# [Memory Layer Name]

## 文档状态

- Current standard status: draft / accepted / superseded
- Covered stages:
  - [ ] M1: Memory Intent
  - [ ] M2: Authority, Lifecycle, and Boundaries
  - [ ] M3: Data Contract
  - [ ] M4: Storage and Operations
  - [ ] M5: Tests and Fixtures
- Last reviewed:
- Owner / reviewer:

## 当前标准文件

- `01_memory_intent.md`
- `02_authority_lifecycle_and_boundaries.md`
- `03_data_contract.md`，如果已存在

## 当前未冻结边界

- [未冻结问题 1]
- [未冻结问题 2]

## 进入下一阶段前必须解决

- [gate 1]
- [gate 2]
````

## 6. `01_memory_intent.md` 模板

```markdown
# [Memory Layer Name] - Memory Intent

## 1. 为什么这层 memory 必须存在

它解决的问题是：

> [一句话]

如果删除它，会失去：

- [能力 1]
- [能力 2]

这些能力不能由 runtime handoff 或现有 store 覆盖，因为：

- [说明]

## 2. 保存什么

这层 memory 保存：

- [内容 1]
- [内容 2]

其中可以成为 reusable authority 的是：

- [authority item 1]
- [authority item 2]

只作为 trace / context / explanation 的是：

- [non-authority item 1]
- [non-authority item 2]

## 3. 绝不保存什么

这层 memory 绝不保存：

- [禁止内容 1]
- [禁止内容 2]

它也不能替代：

- [source of truth 1]
- [source of truth 2]

## 4. 对核心产品目标的贡献

这层 memory 必须清楚支持以下至少一项：

- [ ] 记忆复用
- [ ] 有证据支持的建议
- [ ] accountant correction learning
- [ ] 审计性
- [ ] accountant control
- [ ] 自动化率提升

具体贡献：

- [说明]

## 5. 已知约束

- [约束 1]
- [约束 2]

## 6. 未决定问题

- [问题 1]
- [问题 2]
```

## 7. `02_authority_lifecycle_and_boundaries.md` 模板

````markdown
# [Memory Layer Name] - Authority, Lifecycle, and Boundaries

## 1. Source of Truth / Authority

只填写当前已经稳定的 authority。未稳定的关系写入 Open Boundaries。

| Concept | Source of Truth | Authority Rule | Not Authority |
| --- | --- | --- | --- |
| [concept] | | | |

## 2. 读取者

本表是本层对外声明的读取接口面：reader 只能依赖此处声明的内容，不得依赖本层内部实现。

| Reader | 读取目的 | 可读内容 | 限制 |
| --- | --- | --- | --- |
| [node/process] | | | |

## 3. 写入者

本表描述「谁执行写入 + 用什么机制」（运行 / 记忆 seam）；至于「写什么、是否够格、什么算 authority」，来源是发起写入的运行节点 spec，本层不重新定义，只校验 finalization proof。

| Writer | 可以写什么 | 写入类型 | 需要 approval 吗 |
| --- | --- | --- | --- |
| [node/process] | | direct / candidate / correction / governance mutation | |

## 4. Candidate 边界

以下内容可以作为 candidate：

- [candidate 1]
- [candidate 2]

Candidate 不能直接成为：

- [authority 1]
- [authority 2]

Candidate 进入 durable state 的条件：

- [条件；如果未定，写“未冻结”]

## 5. Lifecycle / States

只写已经必要且稳定的 state。不要为了完整性设计状态机。

| State | 含义 | 谁可以进入 | 谁可以退出 | 下游含义 |
| --- | --- | --- | --- | --- |
| [state] | | | | |

## 6. Mutation Path

只写已经确定的 durable mutation path。
如果 governance / accountant approval 机制还没稳定，本节必须明确标为未冻结。

已冻结 mutation path：

```text
[mutation source]
-> [review / validation]
-> [approval authority]
-> durable mutation
-> audit trace
```

未冻结 mutation path：

- [未冻结点 1]
- [未冻结点 2]

例外：

- [如果无，写“无”]

## 7. 与其他 memory/log 的边界

只写已确认边界。
不要提前补完整系统地图；不确定的外部边界进入 Open Boundaries。

| Other Store | 已确认边界 | 未冻结边界 |
| --- | --- | --- |
| Evidence Log | | |
| Transaction Log | | |
| Entity Log | | |
| Case Log | | |
| Rule Log | | |
| Governance Log | | |
| Knowledge Summary | | |

删除不适用或未讨论的 store；不要保留空行制造假确定性。

## 8. 冲突处理

只写当前 memory layer 自己必须处理的冲突。

如果本 memory 与 [other source] 冲突：

- authority 顺序：
- runtime 行为：
- 是否阻断自动化：
- 是否生成 review / governance candidate：

如果冲突规则依赖其他未定 spec：

- 标记为 Open Boundary
- 不得写成当前标准

## 9. Audit / Trace 边界

这层 memory 必须留下的 trace：

- [trace 1]
- [trace 2]

这些 trace 用于：

- review
- correction
- governance
- audit

这些 trace 不能成为：

- reusable authority
- accountant approval
- governance approval
````

## 8. `03_data_contract.md` 模板

只有当字段、消费者和 mutation authority 已经稳定时才创建此文件。

````markdown
# [Memory Layer Name] - Data Contract

## 1. Contract Status

- Status: draft / accepted
- Depends on:
- Consumers:
- Not implementation-ready because:

## 2. Record Types

| Record Type | Purpose | Durable or Ephemeral | Notes |
| --- | --- | --- | --- |
| [record] | | durable / ephemeral | |

## 3. Field Definitions

### `[record_type]`

| Field | Required | Type / Values | Authority | Meaning | Must Not Mean |
| --- | --- | --- | --- | --- | --- |
| [field] | yes/no | | | | |

## 4. Mutation Types

| Mutation Type | Writer | Approval Required | Audit Trace Required | Notes |
| --- | --- | --- | --- | --- |
| [mutation] | | yes/no | yes/no | |

## 5. Lifecycle States

```text
[state_name]:
- value_1
- value_2
```

## 6. Validation Rules

Valid record requires:

- [rule 1]
- [rule 2]

Invalid record includes:

- [invalid case 1]
- [invalid case 2]

## 7. Retention / Supersession

- Retention rule:
- Supersession rule:
- Deletion / archival rule:

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

- [unfrozen field / enum / mutation / consumer question]
````

## 9. 禁止写法

标准 memory layer spec 中不要出现以下写法：

- “保存相关信息。”
- “必要时更新 memory。”
- “LLM 判断后写入。”
- “高置信度自动沉淀为记忆。”
- “之后由 governance 处理”，但不说明是否已冻结 governance path。
- “与其他 logs 保持一致”，但不说明哪个 store 是 source of truth。
- “可被其他节点读取”，但不说明 reader、目的和限制。
- “写入 trace”，但不说明 trace 是否能成为 authority。

这些写法会让实现代理自行补设计，必须改成明确边界或 Open Boundary。

## 10. M1-M2 完成检查

一份 M1-M2 memory layer spec 完成前，必须能回答：

1. 这层 memory 为什么存在？
2. 为什么不能由 runtime handoff 或现有 store 覆盖？
3. 它保存什么？
4. 它绝不保存什么？
5. 哪些内容可以成为 reusable authority？
6. 哪些内容只是 trace / candidate / context？
7. 谁可以读？
8. 谁可以直接写？
9. 谁只能提出 candidate？
10. accountant 或 governance approval 是否必须参与？
11. 已冻结的 mutation path 是什么？
12. 哪些 mutation path 还未冻结？
13. 它与哪些其他 memory/log 的边界已确认？
14. 哪些跨 store 边界还不能写成标准？
15. 冲突时是否已有 authority 顺序？
16. 哪些 trace 会留下？
17. 哪些 trace 不能成为 authority？
18. 你是否把读取者 / 写入者声明为对外接口面，reader 只依赖声明内容？
19. 你是否守住运行 / 记忆 seam——只定写入机制，没有去重判内容该不该写？
20. 哪些问题未冻结？
21. 是否可以进入 M3 data contract？

如果不能回答，不要进入数据契约或实现阶段。
