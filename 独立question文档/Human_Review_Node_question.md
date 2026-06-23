# Human Review Node Question

## 文件角色

本文件记录 **Human Review Node**（旧称 Review Node，本轮改名）当前已确认的临时设计结论，聚焦"交易已 finalized 之后、人类会计师事后主动发现一条过去已完成的分类是错的"这一整套纠错机制。

它不是正式 spec，不冻结 question 模板、字段或 data contract。Human Review Node 尚未正式审计。

后续处理流程（owner 已定）：

1. 本文先送新窗口做审计；
2. 写入 `L2_proposals/` 并对 L2 提案审计；
3. 通过后才重写为 `BK_Copilot/workflow_nodes/human_review_node/` 正式 spec。

本节点为多个旧划分项合并而成：

- 只读复核视图（原 Review Node 的"展示"职责）
- 纠错理解器 / **Change List LLM**（旧系统里"会计师跟模型说要改"的角色）
- **Change List Engine**（确定性声明式依赖引擎，归本节点范畴）

合并理由：人的纠错行为一定发生在看完复核视图之后；理解器需要把"会计师正在看的那条交易"当作上下文才能正确指代；影响展开必须与理解器产出的影响清单对账。三者是同一条交互链，拆开会产生上下文接缝。

## 背景

系统对已完成交易有两条自动路径：High-Confidence 自动完成、Rule 命中产出。这些交易 finalize 后写入 Transaction Log（append-only 权威结果层）。会计师需要一个事后入口去查看并纠正其中的错误。

Coordinator 不承担此职责，分界已确定（来源：`BK_Copilot/workflow_nodes/coordinator_node/`）：Coordinator 只处理**运行中**交易、只消费 Case Judgment 的 Pending、**不得在交易已 finalized 时触发**。纠错是"已 finalized 交易 + 人发起"这一条独立轴，因此 Human Review Node 是独立节点，不能内联进 Coordinator。

---

## 当前确认结论

### 1. 节点身份与触发

- Human Review Node 是一个**事后、人发起**的入口，独立于 Coordinator。
- 触发时机：交易已 finalized 之后，会计师复核当天 / 过往已自动完成的交易时。
- 入口形态：一个**只读投影视图**（非 LLM），按 Bank Statement 聚合展示当天所有 High-Confidence 自动完成交易 + 所有 Rule 产出交易，供会计师查看。
- 在 Human Review Node 里，**人类会计师本身就是发现器**（不是系统扫描发现），人类权威天然在场。

### 2. 三层结构（职责分层，不可揉成一团）

**第一层 · 理解 + 提案（Change List LLM）**

- 听懂会计师的自然语言纠错，结构化成一个"纠错意图"（指哪笔、改成什么、scope 多大）。
- **撒网产出一份 Change List（LLM 版）**：故意想得宽，覆盖可能没被预先写进依赖表的连带影响（例："这一改还动到年终汇总"）。
- Change List LLM 只理解和提案，永不写字节。

**第二层 · 展开影响（Change List Engine + 对账）**

- Change List Engine 独立产出一份 Change List（Engine 版）：沿设计期声明好的依赖图遍历，展开每次都一样的不变量型后果（如：改分类 ⇒ 改 source、生成 Intervention ID 链路、append 记录）。
- **两份 Change List 对账（diff）**：
  - 一致项 → 高置信，保留；
  - Change List LLM 多出、Engine 没有 → 要么是漏写进依赖表的真连带（补进表），要么是 LLM 瞎编（弃）；二者都摆进 read-back 让会计师裁决；
  - Change List Engine 有、LLM 漏 → 信 Engine（那是已声明的死规矩）。
- 设计理由：Engine 是闭世界的，单跑会**静默漏掉**没人声明过的连带边；read-back 能挡 LLM 多写、挡不住 Engine 少写。Change List LLM 这份清单是补上"开世界覆盖"的唯一手段。灵活性活在"声明式依赖图 + 确定性遍历 + LLM 清单对账"里，不在任何一方的自由发挥里。

**第三层 · 执行（确定性写手）**

- 对账并经会计师确认后，由统一 finalization 写入机制跨 log 落盘：原子、强制不变量、ID 链路。

### 3. 三条不可逾越的铁律

1. **LLM 永不写字节。** 它只产出结构化意图和提案，所有 durable 写入由确定性代码执行。理由：格式合法但内容错的权威账目是记账系统最致命的失败模式；确定性写手只保证结构完整性，不能由它给一个错误判断镀金。（与 BK_Copilot 每个 node spec 的"绝不能写入或修改"一致。）
2. **写入前强制 read-back。** 落盘前，把对账后的意图 + 后果用人话念给会计师，他批准的是这个**复述版本**，不是他的原话。**每一次纠错都要念，不分爆炸半径大小。** 这是堵语义错误（LLM 把会计师理解错了）的唯一防线——read-back 防语义幻觉，确定性写手防格式幻觉，二者缺一不可。
3. **scope 绝不静默升级。** "改这一笔"与"以后永远这么走"是两个单独的、明确的会计师确认，不能合并、不能由 LLM 从单笔脑补成永久。

### 4. 单笔纠错（低爆炸半径）的级联 —— 确定落哪些 Log

以"一条 active rule 跑出来的分类被会计师推翻"为例，确定要做的：

| 动作 | 落点 | 约束 |
| --- | --- | --- |
| append 一条更正记录（谁改的、改后的最终分类） | Transaction Log | 绝不就地改写（append-only，外部审计强制） |
| 记"为什么改"，生成 Intervention ID | Intervention Log | 细节在 Intervention Log |
| 把 Intervention ID 回写挂到该交易 | Transaction Log | Transaction Log 只挂 ID，不存改动细节 |
| 把对应旧先例标记 superseded | Case Log | 只作废系统确认的旧先例；**不动人工确认过的先例**；绝不删除 |
| 最终确认来源改为 Accountant | 该笔 source | |
| 发"该 rule 被人工推翻一次"信号 | 治理 / 发现层 | 仅当该笔来自 active rule |

- "作废哪些先例"是一条**带条件的确定性分支**（按先例的确认来源字段过滤），不是 LLM 判断：作废系统确认的，遇到人工确认过的则停下来进 read-back 让会计师确认是否一并作废。
- 低半径纠错**不过授权确认机制**，因为没产生扩张型长期权威，且发起人就是会计师，人类权威已在场。

### 5. "rule 坏了 vs 一次性例外"的处理

- 这是 LLM 在 Human Review Node 里**唯一不可替代的认知内核**：区分 (a) rule 错了 → 该降级 / (b) 这一笔是合法例外 → rule 留着只改这笔 / (c) rule scope 太宽 → 该收窄。
- 但**降级了刻度**：单次纠错对"rule 坏没坏"是弱证据，LLM **不在纠错当场裁决"rule 坏了"**。纠错时 LLM 只做两件——结构化这一笔的纠正 + 发"rule 被推翻一次"信号。"rule 到底坏没坏"是另一个**攒证据**的过程（N 次推翻 / 跨 case 模式），它本身可复核。
- 即便最终判定"rule 该降级"，LLM 也只提案，走授权确认机制（会计师签 + 写 Governance Log），再由代码落盘。仍不碰字节。

### 6. 永久纠错（高爆炸半径）路径

- 会计师显式说"以后这个 entity 都这么走" = 他在**撰写一条 rule/policy**。
- 必须并回授权线：先过**跨 rule 互斥校验**（这条强制 rule 与已有 active rule 冲不冲突，确定性代码，写入/生效前做），再经授权确认机制，最后落盘。
- 升级到永久这一步，必须会计师**单独显式点头**（铁律 3），LLM 不得从单笔纠错脑补。

### 7. 决策权限

| 谁 | 可以决定 |
| --- | --- |
| Deterministic code（Change List Engine 等） | 沿依赖图展开 Change List（Engine 版）；按先例确认来源过滤作废范围；append/ID 链路/不变量校验；互斥校验；落盘 |
| LLM 可以判断（Change List LLM） | 听懂自然语言纠错、结构化意图、定 scope 候选、撒网产出 Change List（LLM 版）、区分 rule 错/例外/范围过宽（仅作提案与发信号） |
| LLM 不能判断 | 当场裁决某 rule "坏了"并执行降级；把 scope 从单笔升级为永久；任何 durable 写入 |
| Accountant 必须决定 | read-back 的最终确认；diff 中 LLM 多出项是真连带还是幻觉；是否一并作废人工确认过的先例；scope 是否升级为永久 |
| Governance 必须批准 | 所有扩张型长期权威变更（rule 升/改/删/降、永久纠错、merge/split、automation 放宽） |

### 8. 与共享机制 / 数据层的接口（运行 / 记忆 seam）

本节点只声明"要持久化什么 + 谁有权威认定它有效"，不声明"怎么写、谁来写、什么顺序写"。相关单元：

- **Change List Engine（影响展开引擎）**：确定性代码，归 Human Review Node 范畴（要与 Change List LLM 的清单对账）。其依赖图的声明位置与形态属 L4/seam，待定。裸遍历能力为机制类，将来治理审批流可能复用，届时抽成共享、不重复实现。
- **授权确认机制**：共享机制（**非节点**，无自身判断，判断者是会计师）。它是所有扩张型变更通往写入机制的**单一、强制、不可旁路**关卡：read-back + diff 仲裁 + 会计师签字 + 带凭证触发写入。Human Review Node 在高半径 / 永久纠错时调用它；低半径单笔纠错不经过它。
- **统一 finalization 写入机制**：共享确定性代码，跨 log 落盘（原子/顺序/幂等/不变量/审计 trace）。审批与纠错共用。
- **Governance Log**：数据层，每笔权威变更的审计账本。**当前不存在，必须新建。**

### 9. source / 边界纪律

- 运行层 log（Transaction / Case / Rule / Entity / Alias）以 `BK_Copilot/` 正式草案为准。
- 规则治理层 / Human Review Node 层功能以本轮对话为设计来源。
- `new system/` 不作为任何参考来源。
- 引"已冻结不变量"时区分性质：**外部强制**（审计要求：append-only、不可就地改）可当公理引用；**自我施加**的建模选择（如 Case Log 只存最新）要靠功能理由站住（Case Log 的功能是复用最准判断，历史归 Transaction + Intervention），不能拿文档本身自证。

---

## 仍未冻结 / 待审计（Open Boundaries）

进入 L2 / Stage 3 前需解决：

1. **diff 综合的成本分层**：是否每一笔纠错都跑 Change List LLM + 对账？还是低半径走 Change List Engine 单跑、仅高半径/新型纠错才跑两份对账？owner 已确认保留对账机制本身；触发分层未定。
2. **先例确认来源字段**：第 4 节"系统确认 vs 人工确认"依赖 Case Log 的确认来源字段（对话中以 `confirmed_by=system / accountant` 表述）。确切字段名 / 语义需与 `BK_Copilot/memory_layers/case_log/` 对齐确认。
3. **HST/GST 重算的性质**：第 4 节"改分类 ⇒ 重算税"中，税率适用是**查表**（依赖 entity 已记录的税务状态字段）还是**判断**（字段缺失时需停下来问人）。依赖 entity / tax 模型是否已记录该字段，待对齐；缺失时按"引擎撞到缺失输入 → 停下来进 read-back"处理，不交 LLM 猜。
4. **read-back UI 与只读复核视图**：是否同一 UI shell（一个是"浏览找错"，一个是"逐项确认提案"），待编排时定。
5. **语义发现器（merge/split）是否并入 Human Review Node 由人发起**：系统自发审计长期文档做 merge/split 的依据存疑，可能改为只在 Human Review Node 由人发起提案。**单独挂起，将另开窗口讨论**（见项目记忆 `semantic-discovery-node-necessity-open`）。
6. **L4 / seam 机制**：影响展开引擎依赖图声明形态、授权确认机制 exact 实现、finalization 多 log 原子/顺序/幂等机制，均未冻结。
7. **完整 read-back 模板 / 字段 / data contract**：本文不冻结，留 Stage 3。
