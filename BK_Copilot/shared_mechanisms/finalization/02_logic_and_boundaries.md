# Finalization 写入机制 - Logic and Boundaries

## 1. 调用入口

本机制在以下条件满足时被同步调用：

- 调用方已经形成要 durable 写入的 payload，并能标明目标 log / store 与目标动作。（决策 1 / 3）
- payload 满足对应动作的 L2 语义校验要求；exact field schema 留 Stage 3。（决策 3）
- 若 payload 内含扩张型写入，整批 payload 必须附会计师凭证。（决策 4）

本机制不得用于：

- 语义判断、内容对错裁决、会计处理重判或 proof 真伪判断。（决策 5）
- 绕过凭证执行扩张型写入。（决策 4）
- 替调用方补全缺失 payload、判断 merge / split 批是否语义完整，或做失败后的善后决策。（决策 7 / 8）

本机制无 runtime 流水线工位；它是被调用的共享库。

## 2. 调用方前置条件

调用方必须已经完成：

- 提供符合目标动作校验的完整 payload。例如 `create` 必须提供目标 log 当前 L2 所要求的全强制语义；`update` 必须提供目标记录引用和 patch 语义；`append` 必须符合 append-only 目标；`supersede` 必须提供现有记录引用与血缘语义。（决策 3）
- 对 Case Log 等需要完成证明的写入，必须携带 finalization proof 或等价 proof reference；本机制只检查 proof 是否存在和格式是否满足输入规格，不重判 proof 内容真伪。（决策 5）
- 扩张型写入必须附会计师凭证；凭证挂整批 payload，批内扩张型动作受凭证管束，免凭证动作可随批通过。（决策 4）

如果前置条件缺失：

- 本机制行为：拒绝写入，不产生部分 durable side effect，并返回失败回执。
- 失败回执：包含结构化原因，至少说明哪个动作 / 哪条确定性校验失败；exact schema 留 L3。（决策 8）
- 是否 stop-and-ask：本机制自己不向会计师提问；善后归调用方或编排层。

## 3. 动作集

本机制只暴露四个原子动作。校验挂在动作上，不挂在一个泛化地板上。（决策 3）

| Action | L2 校验语义 | 新记录 / ID 语义 | 典型用途 | 不代表什么 |
| --- | --- | --- | --- | --- |
| `create` | 必须给目标 log 所需的全强制语义，写入后记录满足目标不变量 | 产生新记录；新 ID 由调用方铸造并随 payload 带入，本机制不铸造 / 不返还 ID | ER 建 stable entity；会计师直接建 approved rule | 不代表本机制判断“该不该建”或补全 schema |
| `update` | 必须给现有 `record_ref` + patch 语义；改后仍满足目标 log 不变量 | 不产生新 ID，引用现有记录 | 改 Entity 某个当前状态字段；更新 Case Log 可学习状态 | 不代表覆盖 append-only 审计记录 |
| `append` | 必须符合 append-only 目标，新增一条不可就地覆盖的记录 | 产生新记录；新 ID 由调用方铸造并随 payload 带入 | Transaction Log 落盘 / 更正记录 / N 笔回溯 | 不代表原记录被删除或覆盖 |
| `supersede` | 必须引用现有记录，按 `confirmed_by` 等目标语义过滤，不删除，保留血缘 | 不产生新 ID，引用现有记录并留下替代关系 | 旧先例失效；历史规则或案例被替代 | 不代表本机制判断被替代对象在业务上是否错 |

否决为机制动作的项：

- `merge` / `split`：它们是调用方侧标准化动作批，可由 `create` + `supersede` + `update` 等原子动作组合。若设为机制原语，会把“合并 / 拆分跨 log 牵连什么”的语义塞进死代码，破坏机制纯净并放大 trusted base。（决策 3 / 7）
- `link`：直接填目标 log 的 ID / ref 字段，不需要地板动作。（决策 3）

merge / split 批的语义完整性归调用方 / 治理侧；本机制只保证调用方给定的整批动作原子执行，并逐动作做确定性校验。（决策 7）

## 4. 读取对象

本机制只为执行确定性校验读取目标记录当前态。（决策 5 / N4）

| Source | 读取内容 | 用途 | Authority 限制 |
| --- | --- | --- | --- |
| 目标 log 当前记录 | `record_ref` 指向的当前态、append-only / lifecycle / supersession / confirmed_by 等目标不变量所需状态 | 校验 `update` 后是否仍满足不变量；校验 `supersede` 是否按目标规则过滤并保留血缘 | 仅作确定性校验，不作语义判断，不重判该不该写 |
| 凭证 / proof reference | 凭证存在性、proof 存在性和输入规格满足性 | 扩张型放行、Case / Transaction 等 proof gate | 不判断凭证背后的业务对错，不判断 proof 真伪 |

本机制不读取上下文来“理解”交易、身份、规则、案例或治理语义。

## 5. 写入对象（反转）

节点模板里普通节点多数不直接写 durable memory；本机制整体反转：它处在机制侧，是全系统 durable 写入的唯一直接执行者。（决策 1 / 用户澄清 2）

### 直接执行的 durable write

本机制直接执行的 durable write = 全部已允许的长期写入。目标包括：

- Transaction Log（已落文）。
- Entity Log（已落文）。
- Alias Log（已落文）。
- Case Log（已落文）。
- Rule Log（已落文）。
- Governance Log（待建；仅声明扩张型变更审计耦合契约）。
- Intervention Log（待建；具体记录边界归其正式 spec）。
- 审核 inbox（待建；候选投递目标）。

对 Transaction / Entity / Alias / Case / Rule Log，本机制用四原子动作落盘；exact action payload schema、ID scheme 和各 log 字段留 L3。（决策 3 / 7）

### 耦合写入

每笔扩张型变更落盘时，应耦合同步写 Governance Log 审计记录。该耦合由 Human Review Node 等扩张型改动触发，由确定性代码执行。Governance Log 是待建数据层；“记录哪些变动”的判定标准归 Governance Log 设计窗口。（决策 9）

### 绝不写入或修改

- 缺凭证的扩张型写入。
- 就地修改、覆盖或删除 append-only 记录。
- 因本机制自己判断语义正确而新增 / 改写 payload。
- 把 Governance Log trace、Intervention record 或审核 inbox candidate 写成 entity / rule / case authority。

## 6. 凭证闸门

凭证是唯一住在本机制里的差异。（决策 4）

| 规则 | L2 结论 | 不代表什么 |
| --- | --- | --- |
| 放行规则 | 只有拿到凭证时才放行扩张型写入；缺凭证即拒绝 | 不代表本机制重判内容是否应批准 |
| 凭证来源 | 仅会计师；系统无法自给凭证 | 不代表 Human Review、LLM、发现 job 或 Governance Log 能自批 |
| 分叉轴 | 按写入类型是否扩张型分叉，不按谁发起分叉 | 不代表 Human Review 的所有写入都要凭证，或运行期所有写入都免校验 |
| 批粒度 | 凭证挂整批 payload，批内扩张型动作受管，免凭证动作随批通过 | 不代表批内 merge / split 语义完整性由本机制判断 |

凭证闸门是确定性死代码；exact 凭证形态、防重放、签字 ref / 范围和传递方式留 L3。（决策 4）

## 7. 决策权限

### Deterministic code 可以决定

- 动作级校验：`create` / `update` / `append` / `supersede` 是否满足目标动作的确定性校验。
- 凭证校验：扩张型写入是否具备会计师凭证。
- 多 log 原子、顺序、幂等、append-only 和回滚；exact 机制留 L4 / seam。
- proof / credential 是否存在并符合输入规格。

### LLM 可以判断

- 无。本机制不含 LLM，不调用 LLM，也不接收 LLM 的语义判断作为放行权。

### LLM 不能判断

- 任何 durable write 放行。
- 凭证校验、动作校验、proof gate 或写入不变量。
- 业务内容对错、该不该写、是否完整 merge / split。

### Accountant 必须决定

- 会计师签字是凭证唯一来源。
- 扩张型长期权威变更是否被批准，来自会计师 read-back 签字；本机制只检查凭证是否存在。

### Governance 必须批准

- 本机制不产生独立 governance approval。
- Governance Log 是扩张型变更审计账本，非决策者；approval 是会计师签字本身。（决策 9）

## 8. 契约面 / 回执

字段名和 exact schema 留 L3；Stage 2 只冻结语义类别和 Consumer。

| Receipt / Output Category | 含义 | Consumer（谁消费） | 下游影响 | 不代表什么 |
| --- | --- | --- | --- | --- |
| 成功回执 | 本批 payload 的 durable 写入已经按机制事务边界成功完成 | 调用方（ER、CJ、Rule Match、Coordinator、Human Review Node、确定性发现 job 等） | 调用方 / 编排层可继续后续流程；成功后同步可见性的 exact 机制留 L4 | 不返还 ID；不代表本机制成为业务 authority；authority 来自发起节点 / 会计师签字 |
| 失败回执 | 本批写入被拒绝或事务失败并回滚；包含结构化原因 | 调用方 | 调用方据此决定自解、补料、回管道 / Coordinator、回 Human Review 或回会计师 | 不代表本机制替调用方善后、重试或改写 payload |
| 扩张型审计写入 | 扩张型变更落盘时耦合同步写 Governance Log 审计 | Governance Log（待建） | 留下扩张型变更审计账本，支持后续复核 | 不代表 Governance Log 是决策者，不替代 accountant approval，不成为 entity / rule / case authority |

所有回执都不代表本机制成为 authority。authority 来自发起节点的业务判断 / payload 来源，或会计师签字凭证；本机制只执行确定性校验和写入。（决策 5 / 7 / 8）

## 9. 失败 / 异常处理

中途失败时：

- 本机制回滚该事务，不留下部分 durable write。（决策 8）
- 本机制如实返回失败回执，带结构化原因；exact schema 留 L3。（决策 8）
- 本机制对所有调用方一视同仁，不因调用方身份改变失败规则。（决策 6 / 8）

善后归调用方：

- 运行期写入失败：回管道 / Coordinator 或对应编排层处理。
- Human Review 扩张型失败：按 Human Review 规则处理；能在不瞎编信息前提下自解则自解，否则回会计师。
- 确定性发现 job 投递失败：归发现 job / 调度处理，不由本机制决定候选语义。

本机制不决定重试策略、不补造缺失输入、不向会计师提问、不把失败改写成成功。

## 10. 守运行 / 记忆 seam（反向）

节点模板的运行 / 记忆 seam 是：节点只声明“存什么 + 谁有权威认定它有效”，不声明“怎么写、谁来写、什么顺序写”。

本机制处在反向位置：它只定“怎么写 / 是否放行”，绝不重判“该不该写 / 内容对不对”。（决策 5；用户澄清 4）

- “该不该写”来自发起节点、目标 log writer 契约、运行期业务判断或会计师签字。
- “内容对不对”来自发起节点 / 会计师签字；本机制不做语义判断。
- 本机制只做确定性校验：schema / input 完整性、proof 存在、凭证存在、目标不变量满足、append-only / supersession 等动作约束。

这道严格 input 闸会反向逼上游 output 按本机制 input 规格产出，形成全系统抗幻觉 input 闸；但它挡的是不合规格、缺 proof、缺凭证或破坏不变量的 payload，不是通过语义理解判断业务真伪。

## 11. Audit / Trace 边界

本机制应保留 / 触发的 trace：

- 成功 / 失败写入回执。
- 失败结构化原因。
- 凭证存在性校验结果。
- 每笔扩张型变更耦合 Governance Log 审计写入（Governance Log 待建）。
- 多 log 原子事务的审计线索；exact 形态留 L4 / L3。

这些 trace 用于：

- review。
- correction。
- governance。
- audit。

这些 trace 不能成为：

- entity authority。
- rule authority。
- case authority。
- accountant approval。
- governance approval。

approval 是会计师签字本身；Governance Log 是审计账本，不是决策者。（决策 9）

## 12. Open Boundaries

以下问题未冻结；解决前不得进入对应后续阶段。

### L3（字段 / enum / schema）

1. 动作 payload exact field schema、动作 exact 定义和动作类型 enum。
2. 凭证 exact 形态、生成与传递机制、防重放、签字 ref / 范围、与会计师签字的捕获方式。
3. 回执 exact schema；L2 只锁定成功 / 失败 + 失败结构化原因，且不返还 ID。
4. ID scheme；L2 只锁定 ID 由调用方铸造，`create` / `append` 产生新记录，`update` / `supersede` 引用现有记录，唯一性由 ID scheme 保证。

### L4 / seam（机制）

1. 多 log finalization 写入顺序、原子 / 回滚、幂等键。
2. 共享库 + 单一事务边界的 exact 实现；L2 已锁定否决中间写入 agent / 单一全局 LLM 写手。
3. 写成功后的同步可见性（read-after-write）保证机制。
4. 失败回滚 exact 机制、重试、部分失败呈现。

### L2·外阻（圈外依赖）

1. Governance Log 数据层及“记录哪些变动”的判定标准；本机制只声明扩张型落盘耦合同步写审计。
2. 审核 inbox 数据层；确定性发现 job 投候选目标待建。
3. Intervention Log 正式 spec；Human Review correction 的 Intervention 记录 + ID 写入目标待建。
4. merge / split 标准化动作批模板与跨 log 迁移完整性；批是否语义完整归调用方 / 治理侧。
5. 第三类正式 spec 归属目录命名在 Stage 3 前确认；本轮已按用户指令落入 `BK_Copilot/shared_mechanisms/finalization/`。
6. 确定性发现 job、JE Generation、Evidence Log、Case Memory Update Node 等相关对象的正式边界。

这些问题解决前，不能进入：

- [x] Stage 3 data contract
- [x] Stage 4 execution algorithm
- [x] implementation
