# Coordinator Question

## 文件角色

本文件记录 Coordinator 提问环节当前已确认的临时结论，聚焦 unknown entity 进入 pending 后的提问与追问行为。

它不是正式 spec，也不冻结 question 模板、字段或 data contract。Coordinator 节点尚未正式审计；后续正式重写 Coordinator Node 时，以本文作为本轮讨论结论入口。

## 背景

Entity 身份状态只有两态：`stable` / `unknown`。Entity Resolution 输出 unknown_entity 后，交易进入 Case Judgment 输出 pending，由 Coordinator 向 accountant 提问（已确认路径见 `Entity相关的所有问题.md` 的 Problem 4 / Problem 5）。

## 当前确认结论

### accountant 只回分类、未说清主体时的追问

当 Coordinator 向 accountant 提问后，accountant 给出了会计分类，但没有说清这笔交易的主体（counterparty / vendor / payee）是谁时，Coordinator 应追问，以区分两种情况：

- 情况 X — 不需要主体：accountant 认为即使知道主体，对这笔交易的记账也不重要（这笔交易按性质即可分类）。
- 情况 Y — 主体不明：accountant 自己也不知道这个主体是谁。

追问的核心问法是：

> 是你也不知道这个主体是谁，还是你认为即使知道主体、对这笔交易也不重要？

### 该区分的用途

- 这个区分由 Coordinator 在提问时显式收集。
- 情况 X：这笔交易没有身份缺口，unknown 是正确终态。
- 情况 Y：这笔交易留下一个不阻塞的身份缺口；其记录边界见 `BK_Copilot/memory_layers/case_log/`（只进 Transaction Log，不写 Case Log / Entity Log，不阻塞交易，不构成 candidate entity）。

### 边界

- Coordinator 不把 accountant 的模糊回答包装成已确认身份。
- Coordinator 不替代 Case Judgment 做自动分类。
- Coordinator 与 accountant 的交互本身不构成 Entity Log / Case Log / Rule Log authority。

### Entity / Alias 写入归属：主体确认在 Coordinator 路径，不在 Case Judgment

来源：本轮 Case Judgment candidate signals 讨论的副产物，记录与 Coordinator 相关的部分。

- **Case Judgment 不输出 entity / alias 信号，也无权写 Entity Log / Alias Log。** 理由：(1) 在识别交易主体上，Case Judgment 与 Entity Resolution 信息对等——Entity Resolution 判不出主体而输出 unknown 时，Case Judgment 本质上同样判不出；(2) Alias Log 是 Entity Log 的 alias→entity 反查投影，stable entity 的 Entity Log 与 Alias Log 写入应同步发生，Case Judgment 既不能写 Entity Log，自然不能写 Alias Log。
- **主体确认发生在 Coordinator 路径。** Entity Resolution 输出 unknown（可附 Case Judgment 的推断性建议）→ Case Judgment 输出 pending → Coordinator 向 accountant 提问；accountant 明确回复主体身份后，该主体才被确认为 stable entity。此时由 Coordinator 路径发出 entity 确认信号，触发 Entity Log 写入（设计意图：与 Alias Log 同步写入）。
- **automation policy / governance 由人类给出，不由 Case Judgment 生成。** Coordinator 收到 accountant 回复后具备触发相应写入的权限。

边界：

- 该路径与 Entity Log 正式文档 §3「accountant explicit identity confirmation path」一致：创建 stable entity 本体和最小创建 provenance 不需要 governance approval，但只确认 identity，不隐含分类结果写入。
- 由 Coordinator 亲自写、还是同步调用 Entity Log writer / 统一 finalization 机制，以及 Entity Log 与 Alias Log 的同步写入 exact 机制，均未冻结（Entity Log Open Boundary，L4 / seam）。「同步」是当前设计意图，机制待定。
- `Role` 字段此前已删除，本轮不重新启用。

## 仍未冻结 / 未讨论

- Coordinator 完整 question 模板、回答结构和追问边界（= `未解决问题暂存清单.md` 问题 5，仍开放）。
- 同 batch 内重复 unknown entity 的去重提问（= 问题 6，暂缓）；其跨运行版见下方「跨运行重复身份提问 + 多表提交排序」。
- 情况 Y 身份缺口待办由谁 triage / 复查机制（用户暂缓）。

### 跨运行重复身份提问 + 多表提交排序（来源：Case Judgment L2 G3 讨论副产物；记录备查，本轮不讨论）

**多表提交排序**：系统每次运行的输入为一张 Bank Statement 及其证据；处理完一张后 Coordinator 向 accountant 提问。多张表是串行（答完一张再跑下一张，可跨表学习但受 accountant 时延串行）还是并行快照（共享批起始记忆、互不学习、学习只流向下次提交），属 accountant 操作逻辑 + 编排层决定，Case Judgment 不参与（CJ 每次运行只按当时记忆做它能做的）。

**跨运行重复身份提问**：同一 unknown entity 在多张表中反复出现，若 accountant 未及时回答，多次运行会各自产出对该对象的重复身份提问。分析要点（待 Coordinator 正式审计时处理）：

- **自愈且有界**：accountant 回答一次后，Coordinator 路径写入 stable entity + alias（见上「Entity / Alias 写入归属」），此后每次运行 ER exact-match 命中、不再 Pending。重复只局限于「该对象第一次进 Pending 到首个回答 finalize 之间」的在途窗口，并非无限重复。
- **身份 vs 分类不对称**：身份答案（X 是谁）是单一事实，可扇出到所有同 surface 交易；多用途 entity 的分类答案逐笔不同，不可同样去重。
- **最小解法（属 Coordinator）**：answer-time 去重——按未识别 surface text 合并所有 open 身份提问，accountant 答一次扇出到所有挂接交易，由 Coordinator 确认路径解决（已确认交易不重跑 Case Judgment）。因 alias 自愈，正确性上无需重型 open-question 持久化注册表。
- **局限**：只能合并 surface 完全相同者；不同写法（如 `ACME PAYMENTS` vs `ACME CORP TFR`）需 Alias normalization / equivalence（Alias Log open boundary）才能合并。
