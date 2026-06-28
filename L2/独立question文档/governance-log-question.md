# Governance Log Question

## 文件角色

本文件记录 Governance Log 当前已确认的临时结论，是 Finalization L2 审计（决策 9）与 Human Review Node 正式草案的副产物入口，并综合各正式 memory layer 草案对它的引用整理而成。Governance Log 数据层尚未正式审计、当前不存在、**待建**；本文**不是正式 spec**，不冻结 schema、字段、writer、retention 或 data contract。后续正式审计 Governance Log 时，以本文作为讨论结论入口。

与 `intervention-log-question.md`、`interaction_agent_question.md` 同级存放（均为"讨论结论入口"类文档，非正式 spec）；原 `Coordinator_question.md` 已删除，Coordinator 主题已落正式草案。

## 背景

系统的长期确定性权威（rule、stable entity、automation_policy、可复用先例）会被会计师通过 Human Review 发起的扩张型变更修改：rule 新建 / 更新 / 降级 / 废除、automation 放宽、entity merge / split、永久性纠错等。这些变更改变的是系统对**所有未来交易**的处理方式，必须留下可独立追溯的审计账本（核心产品目标：审计性 / CRA 可追溯）。

旧的独立 **Governance Review 审批节点已废除**（缺口地图 §17）：审批职责并入"授权确认"（Human Review Node + Finalization），不再有独立治理审批节点。Governance Log 由此从"节点"退化为单纯的**数据层（审计账本）**，触发与写入由确定性代码承担。

## 当前确认结论

### 唯一作用：扩张型权威变更的审计账本（ledger-of-record）

- Governance Log 是**每笔扩张型权威变更的审计账本** = "高权限长期变化和审批历史的 source of truth"（Entity Log 01:81）。它回答的审计问题是：**这套系统的权威被改过哪些、何时、由谁、附什么凭证 / 签字**。
- 它是**审计账本，不是决策者**：不替代会计师签字，不构成 governance approval，不成为 entity / rule / case authority（HRN 02:111、02:125）。真正的批准是会计师 read-back 签字这条唯一治理线（HRN 02:108-111）。

### 写入触发与机制（耦合 Finalization、确定性同步写）

- 经 Finalization 落盘的**每一笔扩张型变更**，耦合触发一条 Governance Log 的**同步**审计写入；写 Governance Log 的也是**一套确定性代码**（Finalization L2 决策 9）。
- 触发节点是 **Human Review Node** 的扩张型改动落盘那一刻（原 Governance Review 独立节点已废除）。
- 设计意图：用确定性代码同步写、与扩张型落盘耦合，保证 **"改了就有审计、不会改了没记"**（Finalization 决策 9）。
- Finalization 侧只**声明耦合契约、不建 Governance Log**；Human Review Node 只产出"扩张型变更审计"材料投向 Governance Log，**不把 trace 写成 governance approval**（HRN 02:56、02:125）。

### 与其它层的边界：三轴分工，互不替代

- **vs Transaction Log（per-交易最终审计）**：Transaction Log 按 `transaction_id` 索引，结构上只装得下挂在某笔交易上的事实；而 rule 升级、entity merge/split、automation 放宽等权威变更**通常不挂任何单笔交易**，进不了 Transaction Log。两者明确互不替代（Transaction Log 01:21；Rule Log 01:41）。**唯一重叠**：一笔交易被重新分类的纠正会同时落 Transaction Log（correction append）与触发 Governance Log（若属扩张型）——这是同一事件的两个视角（交易历史 vs 权威变更总账），非重复存储。
- **vs 各权威层（Rule / Entity / Case Log）**：各权威层被**有意设计成"只存当前状态 + 指向治理的 refs"**，事件历史被明确推给 Governance Log：
  - Entity Log 01:33 / 01:71：「merge / split / 审批 / policy change 的**事件历史归 Governance Log**；Entity Log 只存当前 lifecycle / supersession / control state 和治理 refs」。
  - Rule Log 01:41：rule 的 source / approval provenance refs「**不替代 Governance Log / Transaction Log / Case Log 正文**」。
  - Case Log L2 提案 §104：Case Log「**不改 Governance Log**」；候选的产生与审批走治理路径。
  - 即：权威层放的是一个**指向 Governance Log 的 ref**，事件正文住在 Governance Log。删掉 Governance Log，整条"权威变更事件史 + 谁签字 + 凭证"将无家可归。
- **vs Intervention Log**：Intervention Log 记**交互过程痕迹**、只供离线系统改进、runtime 不读、留痕≠authority；Governance Log 记**权威变更结果事件 + 授权绑定**、供审计复核。两者不同轴、不替代。
- **vs 审核 inbox**：Governance Log ≠ 审核 inbox（候选队列）（系统上下文地图 46、缺口地图 §298）。inbox 装"待人看的候选"，Governance Log 装"已签字已落盘的变更审计"。

### 必要性结论（本轮讨论确认）

- Governance Log **没有被 Transaction Log 替代**，也不是对称性装饰。它是系统中**唯一**承载"跨所有权威类型的变更事件总账 + 授权绑定（签字 / 凭证）"的地方；Transaction Log（per-交易）与各权威层（per-对象当前态）都按设计回避了这一职责。三者是干净的三分。

## 待确认 / 本轮新提

- **记录哪些变动（触发轴 vs 凭证轴是否一致）〔留待之后拍板〕**：Governance Log 到底记**所有扩张型变更**，还是只记"凭证轴"上需要签字的那批？例如 restrictive auto-downgrade 等**免凭证变动**是否也入账（Finalization 决策 9 open boundary、缺口地图 §41）。此判定决定 Governance Log 的"厚薄"——划清后才能保证它是一本**薄的、事件级 ledger-of-record**，不去重复 Transaction Log `confirmed_by` / correction 与权威层 provenance refs 里已有的内容。
- **审计内容与 Transaction Log / 权威层 refs 的去重边界**：交易绑定的纠正已在 Transaction Log 有 `confirmed_by` + correction，权威层已有 provenance refs；Governance Log 记录的最小必要正文 vs 仅记 ref，需在正式审计时与上述去重边界一并定。
- **〔本轮新提，2026-06-26｜来自 Decisions D9〕rule（及 entity policy）变更事件必须留足"可重建历史态"的正文**：RuleLog 已定每条 rule 只存**当前态**、用稳定 `rule_id` 作 handle，**版本 / 变更历史不进 RuleLog 而归本账本**（Decisions D9；RuleLog 不与治理裁定竞争，Rule Log 02:106）。因此 Transaction Log 只记 `rule_id` 裸值；审计某笔历史交易"当时那条 rule 是什么" = `rule_id` + 交易时间 + **回放本账本的变更事件**。**这要求本账本的 rule 变更事件记到"能重建当时 rule 内容"的程度**（改成什么 / 前后值或快照），不能只记"rule X 被改过"。此约束直接收窄上面"记录哪些变动 / 记最小正文 vs 仅记 ref"的判定：至少 rule（及 entity policy）的变更必须含可重建内容，否则 `rule_id` 回放审计断链。同理适用于 `force_pending` / `promotion_lock` 等 entity policy 变更（Decisions D8）。

## 仍未冻结 / 圈外

- Governance Log 正式 spec：record schema、字段名、exact writer、与确定性写入代码的接口、retention。
- "记录哪些变动"判定标准（触发轴 vs 凭证轴；免凭证变动是否入账）。
- 与 Finalization 耦合的 exact 契约（同步写顺序、原子 / 回滚、幂等如何与多 log finalization 协同）。
- 各权威层 governance refs 指向 Governance Log 的 exact 引用形态。
- 以上待 Governance Log 单独审计（L2·外阻 + L3 / L4）。
