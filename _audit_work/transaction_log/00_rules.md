# Transaction Log 审计 — 规则蒸馏（Agent A）

> 本文件只做规则蒸馏：摆原文 + 出处，不下判断、不列问题清单、不做设计。
> 本轮审计对象节点：**Transaction Log**（是否属 memory layer 尚未定，故 memory layer 与 workflow node 两套模板规则都蒸馏在此）。
> 出处格式：`文件名:章节`。new/old system 一律不作为权威。

---

## 1. 必守通用规则（逐条 + 出处）

### 1.1 Source authority 铁律
- 用户明确指令永远最高。`AGENTS.md:Source Authority`
- 其余文档只在自己的职责范围内拥有 authority；authority 顺序见各文档职责声明。`AGENTS.md:Source Authority`
- 如果同一职责范围内出现冲突，按职责顺序解决；如果职责范围不同，不要互相覆盖（`当前任务状态.md` 不能改写 node 边界，`BK_Copilot/` 草案也不能改写 agent 操作规则）。`AGENTS.md:Source Authority`
- `BK_Copilot/`：对应 node / memory layer 的当前正式草案边界拥有 authority。`AGENTS.md:Source Authority`
- 已审对象的 L1 / L2 结论应以 `BK_Copilot/` 对应正式草案为准。`当前任务状态.md:当前任务状态`

### 1.2 不把 new / old system 当权威
- 以下材料不提供权威参考：`new system/`、`old_system_nodedesign/`。它们只能说明对应设计材料曾经如何表述自己，不能证明某个设计正确、必要、应保留、应回归或应作为 baseline。`AGENTS.md:Source Authority`
- `old_system_nodedesign/` 不是历史意图权威、标准答案、回归目标、可复用约束来源或评判当前系统的依据。`AGENTS.md:被审计材料处理`
- 审计某个 node 时，如需读取 New/Old system，只能把它们当作 source text；任何看似有用的想法，都必须脱离来源重新按核心产品目标、当前已确认规则和正式 `BK_Copilot/` 草案论证。`AGENTS.md:被审计材料处理`
- 从 New/Old system 看到任何可取之处时，不要当作可继承约束，而要降级为待论证假设。`审计目标与原则.md:被审计材料处理约束`
- 被审计材料中的任何设计想法都不能因来源获得 authority；必须按当前产品目标重新论证其必要性。`审计目标与原则.md:稳定审计原则`

### 1.3 可编辑范围
- 默认可编辑的治理 / 审计文档：`AGENTS.md`、`当前任务状态.md`、`审计阶段路线图.md`；`审计目标与原则.md`（仅当稳定宪章或长期原则确实改变时）；`项目背景速读.md`（仅当明显过时时）；`系统上下文地图.md`（仅当来源关系变化时）；`缺口地图.md`（仅当 L3/L4/seam/L2·外阻缺口状态变化时）；用户要求创建的新审计报告或聚焦审计笔记。`AGENTS.md:可编辑范围`
- 默认只读的被审计材料（除非用户明确要求编辑）：`new system/new_system.md`、`new system/node_stage_designs/*.md`、`old_system_nodedesign/*.md`。`AGENTS.md:可编辑范围`
- 审计过程中不要静默改写产品设计 spec；若审计发现意味着需要设计变更，应报告为 finding / recommended simplification / decision point，除非用户明确要求编辑 spec。`AGENTS.md:可编辑范围`

### 1.4 停止 / 升级条件
- 遇到以下情况停止并询问用户：任务需要改变产品设计合同而非审计；审计发现依赖用户尚未决定的业务优先级；两个权威来源冲突且无法通过职责范围和 authority 顺序解决；当前工作会把 `new system/`/`old_system_nodedesign/` 当成权威/baseline/正确性证明；当前工作会模糊被审计材料与当前有效规则/正式草案的边界；任务需要大范围重写 `new system/` 而非一次聚焦的已请求编辑。`AGENTS.md:停止 / 升级`
- 遇到以下情况宁可暂缓也不强行闭环：当前 baseline 需要真实设计决策而非措辞清理；某能力是否应保留需要真实产品决策；某简化会改变审计性、会计师控制权或自动化 authority 而用户尚未批准。`AGENTS.md:停止 / 升级`

### 1.5 已锁不变量（稳定审计原则 / 操作规则）
- 第一性原理优先于对旧架构或当前架构的忠诚。`审计目标与原则.md:稳定审计原则`
- 除非复杂度保留了真实且必要的能力，否则更简单的结构优先。`审计目标与原则.md:稳定审计原则`
- 每个 node、field、log、memory layer 和 workflow step 都必须证明自己为什么需要存在；不应仅因架构更完整而存在。`审计目标与原则.md:稳定审计原则`
- 审计性和会计师控制权是核心能力，不是附加装饰。`审计目标与原则.md:稳定审计原则`
- 自动化只有在有证据、已批准记忆、适当 authority 和可恢复 audit trail 支持时才有价值。`审计目标与原则.md:稳定审计原则`
- 从纠正中学习必须改善未来行为，但不能让一次性错误污染 durable memory。`审计目标与原则.md:稳定审计原则`
- 清楚区分 evidence、memory、judgment、governance 和 audit records。`AGENTS.md:操作规则`
- 不要把 AI reasoning 文本变成可复用 authority；可复用 authority 必须来自已批准的 memory、rules、cases、profile 或 governance state。`AGENTS.md:操作规则`
- 不要把第一版 Stage 3 data contracts 当成冻结实现文档。`AGENTS.md:操作规则`
- 长期性设计结论不写入规则文档。`AGENTS.md:操作规则`

### 1.6 守 seam（运行 / 记忆 seam）
- 节点与记忆层之间只通过显式、最小、写明的契约（接口）交互，而不依赖彼此的内部实现或隐式假设；流水线式的顺序依赖不可避免也不必规避，真正要保持松的是接口面而非顺序本身。这条原则不要求每个对象一产出就给出冻结的字段级 schema——契约的最终冻结属于 data contract 阶段（workflow Stage 3 / memory M3）；更早阶段只需明确意识到并朝它设计。`审计目标与原则.md:稳定审计原则`

### 1.7 契约面纪律
- 本节点 / 本层只通过显式、最小、写明的输出 / 读写接口面对外交互；下游 / reader 只能依赖此处声明的输出，不得依赖内部状态或实现。字段级 schema 的最终冻结属于 Stage 3 / M3。`workflow_node_spec_template_rules.md:2`、`memory_layer_spec_template_rules.md:2`

### 1.8 输出纪律（审计输出）
- 审计输出必须区分：已确认设计问题 / 设计风险 / 简化候选 / 待用户决定的问题 / 假设或未解决证据缺口。`AGENTS.md:输出要求`
- 最终总结必须包括：改了哪些文件、为什么改、做了哪些验证、剩余风险 / 后续事项；若刻意未运行验证要明确说明。`AGENTS.md:输出要求`

---

## 2. 与 Transaction Log 相关的「已锁事实」（逐条 + 出处）

> 以下从 `AGENTS.md` / `审计目标与原则.md` / `当前任务状态.md` / `缺口地图.md` 摘出可能约束 Transaction Log 的已锁事实。
> **具体适用范围以对应正式文档为准。** 本节只摆原文 + 出处，不预判 Transaction Log 是否落入其中任何一条。

### 2.1 圈层 / 落文状态事实
- Transaction Log 仍是圈外或未正式落文对象，只能通过 `缺口地图.md` 和相关主题入口追踪接缝。**具体适用范围以对应正式文档为准。** `当前任务状态.md:当前任务状态`
- JE Generation / Transaction Log / Intervention Log 是 Coordinator 进 Stage 3 的前置；推进圈外对象正式落文时为可选方向之一。**具体适用范围以对应正式文档为准。** `当前任务状态.md:下一步`

### 2.2 分类 / 职责区分事实
- 清楚区分 evidence、memory、judgment、governance 和 audit records（Transaction Log 的归属须落在此分类纪律下）。**具体适用范围以对应正式文档为准。** `AGENTS.md:操作规则`
- 不要把 AI reasoning 文本变成可复用 authority；可复用 authority 必须来自已批准的 memory、rules、cases、profile 或 governance state。**具体适用范围以对应正式文档为准。** `AGENTS.md:操作规则`

### 2.3 写入 / finalization 纪律事实（来自 `缺口地图.md`，均为未冻结 seam，但已锁「归属去向」语义）
- 多 log finalization 顺序（Evidence / Transaction / Case / Rule / JE）属 **【L4/seam】**，Evidence Log 尚未正式审计。**具体适用范围以对应正式文档为准。** `缺口地图.md:Evidence Intake`
- Case Log：`exact writer / Transaction Log trigger order / 多 log 统一写入` 属 **【L4/seam】**。**具体适用范围以对应正式文档为准。** `缺口地图.md:Case Log`
- Case Judgment：`Case Memory Update Node exact 写入机制、case_memory_update_candidate exact schema、Case Log 与 Transaction Log finalization trigger order` 属 **【L3 / L4/seam】**。**具体适用范围以对应正式文档为准。** `缺口地图.md:Case Judgment`
- Case Judgment：`reasoning 在 Transaction Log 的 exact 存储字段、后续治理节点读取 reasoning 的 contract` 属 **【L3 / L4/seam / L2·外阻】**。**具体适用范围以对应正式文档为准。** `缺口地图.md:Case Judgment`
- Rule Log：`rule-hit ref 在 Transaction Log / Case Log 的字段形态、统计派生 / rollup / cache / maintenance job` 属 **【L3 / L4/seam】**。**具体适用范围以对应正式文档为准。** `缺口地图.md:Rule Log`

### 2.4 Coordinator 侧对 Transaction Log 的圈外依赖事实
- Coordinator 把 **Transaction Log** 列为 **【L2·外阻】** 圈外依赖：`情况 Y 缺口落点 / finalization / 结果审计落点`；「绝不拿 B（new/old system）填」。**具体适用范围以对应正式文档为准。** `缺口地图.md:Coordinator / L2·外阻`

### 2.5 身份 / 会计分类归属事实（约束 Transaction Log 不得越界承载的 authority）
- 身份判断锁在 Entity Resolution（ER）：身份级 risk flags、ER `unknown`、Entity Log automation_policy、Case Log use_level 等属各自归属层；CJ hard block 语义已定。**具体适用范围以对应正式文档为准。** `缺口地图.md:Case Judgment`
- 身份冲突（疑似同名 / 疑似 merge / identity conflict）的发现与 merge/split 解决机制判定归治理（Entity Log §4 `merge_split_candidate` → Post-Batch Lint / Governance Review），不在 Coordinator 这一环。**具体适用范围以对应正式文档为准。** `缺口地图.md:Coordinator / L2·外阻`
- 会计分类归属：High Confidence Classification 携带确定 (COA, HST/GST)；CJ→JE 之间过确定性 COA 校验；JE 纯确定性构造仍待正式审计确认。**具体适用范围以对应正式文档为准。** `缺口地图.md:Case Judgment`

### 2.6 transaction_id 铸号事实
- Transaction Identity 节点已删除，`transaction_id` 铸号并入 Evidence Intake 流程第 9 步；跨导入复用 / 迟到证据接回 / durable registry 全部 DEFERRED。**具体适用范围以对应正式文档为准。** `当前任务状态.md:最近重要交接`
- durable transaction registry / 跨导入 `transaction_id` 复用 / 迟到证据接回 / 跨运行更正定位 / 跨运行账务幂等 属 **【L4/seam / DEFERRED】**。**具体适用范围以对应正式文档为准。** `缺口地图.md:Evidence Intake`

### 2.7 缺口地图回写纪律事实
- `缺口地图.md` 只记录未冻结 L3 / L4 / seam / L2·外阻缺口；已收口 L1 / L2 结论不回灌到规则文档，以 `BK_Copilot/` 对应正式草案为准。`当前任务状态.md:当前正式文档状态` / `缺口地图.md:文件职责`

---

## 3. Memory Layer 模板 M1-M2 完成清单 + 禁止写法（逐字搬运）

### 3.1 M1-M2 完成检查（逐字，出处 `memory_layer_spec_template_rules.md:10`）

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

### 3.2 Memory Layer 禁止写法（逐字，出处 `memory_layer_spec_template_rules.md:9`）

标准 memory layer spec 中不要出现以下写法：

- "保存相关信息。"
- "必要时更新 memory。"
- "LLM 判断后写入。"
- "高置信度自动沉淀为记忆。"
- "之后由 governance 处理"，但不说明是否已冻结 governance path。
- "与其他 logs 保持一致"，但不说明哪个 store 是 source of truth。
- "可被其他节点读取"，但不说明 reader、目的和限制。
- "写入 trace"，但不说明 trace 是否能成为 authority。

这些写法会让实现代理自行补设计，必须改成明确边界或 Open Boundary。

---

## 4. Workflow Node 模板 Stage 1-2 完成清单 + 禁止写法（逐字搬运）

### 4.1 Stage 1-2 完成检查（逐字，出处 `workflow_node_spec_template_rules.md:10`）

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

### 4.2 Workflow Node 禁止写法（逐字，出处 `workflow_node_spec_template_rules.md:9`）

标准 workflow node spec 中不要出现以下写法：

- "这个节点负责协调一切相关逻辑。"
- "必要时可更新相关 memory。"
- "LLM 根据上下文自行判断。"
- "如果高置信度则自动通过。"
- "后续实现时再决定。"
- "参考旧系统处理。"
- "写入日志"，但不说明写入哪个 log、什么字段、什么 authority。
- "生成候选"，但不说明候选能否持久化、由谁批准、谁消费。

这些写法会让实现代理自行补设计，必须改成明确边界。
