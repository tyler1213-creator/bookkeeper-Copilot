# Human Review Node - Logic and Boundaries

## 1. 触发条件

本节点在以下条件满足时触发：

- 会计师主动要求复核或修改系统已确定事实，例如多张 Bank Statement 处理完、导入 QuickBooks 前整批审，或周审本周全部账目。（决策 1 / 2）
- 处理对象是已 finalized 的事实：已完成交易、长期 Entity 事实、已确定 Rule，或审核 inbox 中等待会计师确认的 rule 升级候选。（决策 2 / 6 / 13）

本节点不得在以下情况触发：

- 交易仍在 running / Pending / 卡住 / terminal / blocked 状态，或仍需运行期交互；这些归 Coordinator 或对应运行期路径。（决策 1 / 2）
- 仅发生 onboarding、目标询问、覆盖核对；这些归 interaction_agent / onboarding 路径。
- 系统自发语义判断发现的 merge / split 或错误发现入口；该系统自发语义发现层已裁撤删除，本节点不实现此入口。（用户澄清 4）

## 2. 上游前置条件

上游必须已经完成：

- 对交易纠错：交易已经 finalized，并有 final outcome 或等价 finalization source。
- 对长期 fact 纠错：目标 Entity / Rule 是系统已确定事实，且会计师明确提出纠错或永久变更意图。
- 对 rule 升级候选：候选已由确定性发现 job 投入审核 inbox；审核 inbox 与固定执行路径均为待建 open boundary。

如果前置条件缺失：

- 本节点行为：不补造 finalized 状态、不猜 schema、不接管 running / Pending 流程。
- 是否 stop-and-ask：如果缺失来自未冻结契约或圈外对象，停止要求产品设计决策；如果交易未 finalized，返回 Coordinator / 运行期路径。

## 3. 读取对象

删除模板中不适用的 source；本节点当前只声明以下读取面。

| Source | 读取内容 | 用途 | Authority 限制 |
| --- | --- | --- | --- |
| Transaction Log | final outcome、最终 COA / HST-GST、`confirmed_by` 审计留痕、processing path、rule-hit source / rule ref（字段形态未冻结） | 展示复核视图、定位已 finalized 事实、支持 correction append 备料 | 只读，非 authority producer；reasoning / audit narrative 不能成为 reusable authority、accountant approval 或 governance approval。 |
| Case Log | relevant precedent、exception context、`use_level`、`confirmed_by`（exact enum 未冻结） | 帮助 accountant 理解历史案例；决定哪些 system-confirmed 先例语义上可被 supersede / update | Case Log 不是 deterministic rule、不是 final audit record；本节点不把案例变成 rule、entity authority 或 governance approval。 |
| Entity Log | active state、authority refs、candidate context、identity risk / merge-split / policy context | 展示当前 entity authority 与候选风险；支持人发起 merge / split 或 entity mutation 的确认面 | 本节点不批准 durable entity mutation；candidate 不是 stable identity authority。 |
| Rule Log | 当前 executable rule 上下文、rule provenance、当前执行状态、rule 是否 active | 判断某笔是否来自 active rule；支持会计师在 review 时判断 rule 是否失效，以及 rule 改动 / 降级 / 纯升级的确认面 | candidate / promotion request 不能执行；rule authority 必须经 accountant sign-off 和 Rule 侧治理路径。 |

## 4. 写入对象

### 直接执行的 durable write（例外）

本节点直接执行的 durable 写入：

- 无。本节点不裸写任何 log。

### 交给记忆 / finalization 层持久化的内容（运行 / 记忆 seam）

本节点产出、但由 Finalization / 记忆层执行写入的内容：本节点只声明“存什么 + 谁有权威认定它有效”，不声明“怎么写、谁来写、什么顺序写”。

- Transaction Log correction append：更正记录，表达谁改、改后最终分类结果、关联 Intervention ID。authority 来源是会计师 read-back 签字；Transaction Log 保持 append-only，原记录不删除、不覆盖。（决策 7 / 12）
- Intervention Log record：纠错原因、交互事实、read-back / 签字上下文和 Intervention ID。authority 来源是交互事实；Intervention Log 正式 spec 待建。（决策 7 / 9）
- Case Log 先例更新 / supersede：语义上只作废 system-confirmed 先例，不动 accountant-confirmed 先例；如果需一并作废人工确认先例，必须进入 read-back 由会计师决定。exact `confirmed_by` enum 留 L3。（决策 7 / 12）
- Entity Log / Rule Log 扩张型 mutation input：merge / split、entity lifecycle、automation 放宽、rule create / update / downgrade 等长期确定性权威变更的备料。authority 来源是会计师 read-back 签字 + Finalization 凭证死代码强制；具体 mutation path 归对应 memory / Rule 侧治理。（决策 6 / 9 / 10）
- Governance Log audit material：每笔扩张型变更的审计留痕。Governance Log 是审计账本，数据层待建；本节点不把 trace 写成 governance approval。（决策 6 / 9 / 10）

### 只能提出 candidate

- rule 升级候选的确认触发：候选来自审核 inbox，本节点只把候选和证据摆给会计师确认，并触发 Rule 侧固定执行路径；固定路径定义归 Rule 侧。（决策 13；用户澄清 3）
- merge / split、automation_policy、entity_risk 等治理候选仅是非身份候选；它们不是 candidate entity / candidate identity，也不是 stable / unknown 之外的身份状态。

### 绝不能写入或修改

- 任何 log 的裸写。
- LLM 产生的 durable 写入、approval 或 finalization 放行。
- candidate entity、candidate identity，或 stable / unknown 之外的身份状态。
- Rule Log executable authority、Entity Log stable authority、Case Log authority、Transaction Log final outcome 的直接覆盖。
- Finalization 写入顺序、原子、幂等、凭证校验机制；这些属于共享落盘地板。

## 5. 决策权限

### Deterministic code 可以决定

- 沿依赖图展开 `P_engine` / 系统变化执行图。
- 按 `confirmed_by` 语义过滤 Case Log 作废范围；exact enum 留 L3。
- append / ID 链路、不变量校验、凭证存在性传递和数据可用性检查。
- 跨 rule 互斥校验、promotion 资格判定、rule 固定路径资格检查等确定性结果；本节点只取结果并展示，不用 LLM 重判。（决策 10）

### Finalization 死代码可以决定

- 凭证校验 = 放行权；缺凭证时拒绝扩张型写入。
- 多 log 原子 / 顺序 / 幂等 / append-only 不变量；exact 机制留 L4 / seam。（决策 9 / 10）

### LLM 可以判断

- 听懂会计师纠错意图，结构化为要改什么、改成什么、scope 多大。
- 产出系统变化执行图和 `P_llm`，作为与 `P_engine` 对账的独立第二意见。
- 区分 rule 错、一次性例外、scope 过宽等语义，但只能提案、供会计师在 review 时判定，不能当场裁决 rule 降级，也不自发产降级信号 / 候选。
- 证据缺失时先用会计知识分析、组织可选项和 read-back，让会计师最终决定。

### LLM 不能判断

- 自批、放行、生成 durable write，或绕过 Finalization。
- 当场裁决某 rule “坏了”并执行降级。
- 把 scope 从单笔静默升级为永久，或把“只改这一笔”静默升级成“全部历史”。
- 重判 deterministic code 的互斥、promotion 资格、Finalization 凭证校验或写入不变量。
- 为推进流程猜一个 winner。

### Accountant 必须决定

- read-back 最终确认和签字；这条签字是本节点治理线上的唯一批准来源。
- diff 中 LLM 多出项是真连带还是幻觉。
- 是否一并作废 accountant-confirmed 先例。
- scope 是否升级为永久，或 rule 名下 N 笔历史是否全部回溯纠正。
- 扩张型变更是否通过，以及 rule 本身是废除、更新还是废弃并降级。

### Governance 必须批准

- 本节点不产生独立 governance approval；扩张型变更的批准是会计师 read-back 签字这条唯一治理线。
- Governance Log 只保存经签字并落盘的扩张型变更审计，不是决策者，不替代 accountant approval。

## 6. 输出类别

字段名可以暂不冻结，但语义类别必须稳定。

本表是本节点对下游唯一的契约面：下游只能依赖此处声明的输出类别，不得依赖本节点未声明的内部状态或实现。

| Output Category | 含义 | Consumer（谁消费） | 下游影响 | 不代表什么 |
| --- | --- | --- | --- | --- |
| correction record | 会计师 read-back 签字后的更正语义：谁改、改后最终分类、关联 Intervention ID。 | Transaction Log（经 Finalization） | append 更正记录；原记录不删除、不覆盖。 | 不代表本节点裸写 Transaction Log，不冻结 correction record schema，不代表 Case Log / Rule Log 已同步完成。 |
| Intervention 记录 + ID | 纠错原因、交互事实、read-back / 签字上下文和可追溯 Intervention ID。 | Intervention Log（待建） | 供 correction / review / audit 回查交互事实。 | 不代表业务 authority、final outcome、Case Log authority 或 governance approval。 |
| 先例更新 / supersede | 将错误或被纠正的可复用先例更新为正确态，或标记为失效 / superseded；只在语义上动符合条件的先例。 | Case Log（经 Finalization / 写入机制） | 后续 case-based judgment 不再静默复用错误先例；Case Log 仍非审计层。 | 不代表就地改 Transaction Log，不冻结 `use_level` / `confirmed_by` enum，不把 case 变成 rule authority。 |
| 扩张型 mutation 备料 | 针对 Entity / Rule / automation 等长期确定性权威的变更 input，附 read-back 签字和凭证。 | Entity Log / Rule Log（via Finalization；Rule 侧治理） | 进入对应 durable mutation path；扩张型变更需凭证和对应不变量校验。 | 不代表本节点批准 stable authority、不代表 Rule Log 固定路径已定义、不代表 Finalization 机制已冻结。 |
| 扩张型变更审计 | 每笔长期权威变更的审计材料和批准线索。 | Governance Log（待建） | 留下 governance / audit trace，支持后续复核。 | 不代表 Governance Log 是决策者，不替代 accountant 签字，不成为 entity / rule / case authority。 |
| rule 升级触发 | 对审核 inbox 中 rule 升级候选进行会计师确认后，触发 Rule 侧固定执行路径。 | Rule 侧固定执行路径（Rule Log / Rule Match 治理；NEW-1） | 纯 rule 升级不走开放式 Change List Engine；路径定义归 Rule 侧。 | 不代表本节点定义固定路径、不代表 candidate 自动升级、不代表未经会计师签字可执行。 |

所有输出都不代表本节点自身成为 authority。authority 来自会计师 read-back 签字，以及 Finalization / 对应 memory 层对凭证、schema 和不变量的确定性校验。

## 7. 证据不足时的行为

如果改分类需要重算 HST / GST：

- 输出：先查表；税率适用依赖 entity 已记录的税务状态字段。
- 下游应：如果字段存在，按确定性查表结果继续进入 read-back。
- 本节点不能：让 LLM 猜税率。

如果 entity 税务状态字段缺失：

- 输出：引擎撞到缺失输入，停下进入 read-back；LLM 可先用会计知识分析并给会计师选项。
- 下游应：等待会计师补足或确认可用选项。
- 本节点不能：以高置信度、历史经验或 LLM 推断替代税务字段。

## 8. 歧义处理

每笔纠错都同时跑：

- `P_llm`：LLM 独立撒网产出的影响清单。
- `P_engine`：Change List Engine 沿声明依赖图确定性展开的影响清单。

diff 处理：

- 一致项：保留，进入 read-back。
- LLM 多出项：进入 read-back 让会计师裁决是真连带还是幻觉；真连带可补进依赖图，幻觉则弃。
- Engine 有、LLM 漏：信 Engine；已声明的确定性后果不因 LLM 漏项被丢弃。

是否允许自动化：

- 不允许为了继续流程自动猜 winner。
- 不允许省略 LLM 第二意见或省略 read-back。

是否需要 pending / review / governance：

- diff 未解、scope 未确认、字段缺失或触及长期确定性权威时，必须进入 read-back / 会计师确认；扩张型变更进入 Governance Log 留痕。

## 9. 冲突处理

如果单次纠错暗示某条 active rule 可能错误：

- authority 顺序：单次纠错是弱证据；Rule Log active authority 不由本节点 LLM 当场废除。
- 本节点行为：照常 append 这一笔的更正记录（更正记录引用该 rule，作为审计事实留在 Transaction Log）。
- rule 坏没坏 / 是否降级：只由会计师在 review 时判断（rule 由人创建确认、匹配出错概率低，系统不自发判断 rule 失效）；若会计师判定该 rule 应降级 / 废除 / 取消，由会计师当场人发起，走扩张型变更路径（决策 6 / 9）。
- 系统不自发累积“被推翻”次数、不自发产降级候选；确定性发现只产 rule 升级候选、不碰降级。
- 是否阻断自动化：由会计师在降级决定中处理，不由系统当场自动裁决。

如果改一条 rule 牵连历史 N 笔：

- authority 顺序：会计师必须先选择只改这一笔，还是 rule 名下全部历史。
- 本节点行为：向会计师说明“这是过去定的一条分类规则、已产生 N 笔关联交易”，并要求显式选择；LLM 不脑补 scope。
- 若选择全部：Case Log 语义上直接更新为新分类和 accountant-confirmed 状态；Transaction Log 对每笔逐笔 append 新记录，原记录不删除、不覆盖。
- rule 本身去向：废除、更新、废弃并降级，是单独一问，不与历史交易纠正合并。

如果 Case Log、Transaction Log、Entity Log 或 Rule Log 之间出现冲突：

- authority 顺序：Transaction Log / 等价 finalization source 对 final outcome 与 audit trail 优先；Entity Log 对 stable identity / automation policy 优先；Rule Log 对 executable rule authority 优先；Case Log 只作为可复用先例且受 `use_level` / finalization proof 限制。
- 本节点行为：展示冲突、组织 read-back 和签字；不让 LLM 覆盖任一 source of truth。
- 是否生成 review / governance candidate：可以生成非身份治理候选或 rule / entity mutation 备料，但 candidate 不成为 durable authority。

## 10. Audit / Trace 边界

本节点应保留的 trace：

- Intervention ID 链路。
- correction append 语义。
- read-back 文本、会计师签字和凭证引用。
- Case Log supersede / update 语义。
- 扩张型变更的 Governance Log 留痕材料。

这些 trace 用于：

- review。
- correction。
- governance。
- audit。

这些 trace 不能成为：

- entity authority。
- rule authority。
- case authority。
- accountant approval 本体。
- governance approval。

approval 是会计师签字本身；trace 只帮助复核和审计。

## 11. Legacy Constraint Translation

本节点为 owner 新创节点，无旧系统继承约束。New system / Old system 材料不构成本节点 authority，本正式草案不从这些材料搬运设计。

| Retained Constraint | 来源 | 为什么现在仍成立 |
| --- | --- | --- |
| 无保留旧约束 | 不适用 | 不适用 |

不保留的旧行为：

| Old Behavior | 不保留原因 |
| --- | --- |
| 把旧 Review Node 作为 JE 前 gate 或 final logging 前覆盖节点 | 当前 Human Review Node 是 finalized 后、会计师人发起的纠错与治理沟通节点；running / Pending 归 Coordinator。 |
| 旧“candidate -> 指定节点写入”writer 模型 | 当前 writer 模型已锁为本节点备料并调用 Finalization；A 类残留写回另对象处理，本节点文件内只声明自身边界。 |

## 12. Open Boundaries

以下问题未冻结。

### L3（字段 / enum / schema）

1. `confirmed_by` exact enum；与 Case Log / Transaction Log L3 对齐。
2. Intervention ID、correction record schema；与 Intervention Log 落文对齐。
3. Change List（`P_llm` / `P_engine`）和系统变化执行图 exact schema。
4. 凭证 exact 形态、完整 read-back 模板 / 字段 / data contract。
5. entity 税务状态字段；归 Entity Log / 联合 L3，缺失时停下 read-back，不交 LLM 猜。
6. source 字段形态，包括最终确认来源改为 Accountant 的 exact 表达。

### L4 / seam（机制）

1. Change List Engine / 执行图依赖图的声明位置 / 形态，含 entity / rule 编辑连带遍历。
2. Finalization 调用契约、schema 校验、凭证校验、写入顺序、原子回滚、幂等键；本节点不重定义共享落盘地板。
3. 多个并发待确认项触及同一 rule / entity 的排序、互斥、去重。
4. N 笔批量纠正的逐笔 append 顺序、原子、部分失败时对会计师的呈现。
5. read-back UI 与只读复核视图是否同一 shell；多来源（QuickBooks vs 自有 DB）和多类 finalized 交易的呈现。

### L2·外阻（圈外依赖）

1. Intervention Log 正式 spec：schema、writer、与 Evidence / Review / Governance 边界。
2. Governance Log 数据层：扩张型变更审计账本，待建。
3. 审核 inbox 数据层：确定性发现 / 未来 LLM 审查节点投候选，本节点 pull；待建，且不等于 Governance Log。
4. Finalization 写入机制 / 确定性发现 job 正式 spec：共享机制，另窗细化。
5. 系统自发发现层（FP-5 + 语义发现器）：用户已确认二者本质同一问题，即“系统自发语义判断 -> 喂给人审”；已裁撤删除（系统不自发做此类语义判断）。本节点只承接会计师人发起的 merge / split 与纠错；可确定性化的身份/一致性冲突归确定性发现 / 写入闸 / ER 运行期判句，不在本节点。
6. NEW-1：rule 升级固定执行路径的定义归 Rule 侧（Rule Log / Rule Match 治理），本节点只触发；Rule 侧定义机制待落实。
7. C2：沟通桥梁与 Coordinator / interaction_agent 的对话面分工，以及是否共用同一对话前端；归编排 / 呈现层。
8. A 类残留待写回：废除 / 删除节点引用（Post-Batch Lint / Governance Review Node / 旧 Review Node / 语义发现器）已于 2026-06-23 在五个 log + Coordinator + ER 正式草案中清理完成；仅剩与废除节点无关的「旧 candidate→指定节点裸写」writer 模型泛化清理，按单独对象 prompt 推进；本节点只在自身范围声明新边界。

这些问题解决前，不能进入：

- [x] Stage 3 data contract
- [x] Stage 4 execution algorithm
- [x] implementation
