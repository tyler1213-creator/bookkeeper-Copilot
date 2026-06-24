# 00_rules — Governance Log 审计必守规则 + 记忆层模板 M1-M2 标尺

> 本轮对象：**Governance Log**（扩张型权威变更审计账本 / 数据层，待建）
> 审计套路：新节点审计 Prompt；**成文标准对齐 `memory_layer_spec_template`（M1-M2）**，正式文档将来归 `BK_Copilot/memory_layers/governance_log/`（用户 2026-06-23 拍板）。

---

## A. 必守通用规则（逐条 + 出处）

1. **用户指令最高，其余文档只在自己职责范围内有 authority**（AGENTS.md:43-58）。同一职责冲突按顺序解决；不同职责不互相覆盖。
2. **权威分层 / 真相锚只在 A 类**：A 类 = `BK_Copilot/` 已落文正式 workflow_node / memory_layer / shared_mechanism 草案（00/01/02）+ 模板规则 + 本任务声明的「已锁事实」。**契约面只能从 A 类重建**（新节点审计模板 §一）。
3. **New System 已作废、New/Old 未授权不读**：`new system/` 不再作为审计目标；`new system/` 与 `old_system_nodedesign/` 均需用户明确授权具体范围才可读，未授权只读现行 BKCopilot 内容（AGENTS.md:60-65, 105-115；当前任务状态.md:52）。本轮**默认不读**，且没有理由读。
4. **不可静默改写正式 spec**：审计发现→报告为 finding / 简化候选 / 决策点，除非用户明确要求编辑（AGENTS.md:163）。本 Agent **不创建 / 不编辑** 任何 `BK_Copilot/` 正式文档，只产出 L2 提案 + `_audit_work/` 工作区。
5. **守运行 / 记忆 seam**（memory 模板 §2、审计目标与原则.md:75）：记忆层只定义「怎么写、谁来写、什么顺序写」（机制），**不**重新判断「该不该写、内容对不对」；后者由发起写入的运行节点 spec 决定，本层只校验 finalization proof / authority。
6. **契约面是显式、最小、写明的接口**：reader 只能依赖声明的接口，不得依赖内部实现；字段级 schema 冻结属 M3（memory 模板 §2、审计目标与原则.md:75）。
7. **AI reasoning 文本不得变可复用 authority**；可复用 authority 只来自已批准的 memory / rules / cases / profile / governance state（AGENTS.md:176）。
8. **已废除节点不得复活**：Governance Review 审批节点已废除（缺口地图 §17），其审批并入「授权确认」（Human Review Node + Finalization）。系统自发语义发现层（语义发现器 + FP-5）整体已删除（2026-06-23）。本轮不得把 Governance Log 当成「会审批的节点」。
9. **未落文 / 圈外对象只挂 open boundary**，绝不拿授权历史材料或任何非权威材料填（新节点审计模板 §五.4）。
10. **停止并问用户**：需真实产品决策（非措辞）、A 类权威源未能定位、设计与已锁下游冲突且权威顺序无法解决、output 的 consumer 无法从已锁结论确定（新节点审计模板 §七；AGENTS.md:181-195）。

## B. 本轮「已锁事实」（遵守，非审查对象；具体适用范围以对应正式文档为准）

- **Governance Log 是数据层 / 审计账本，不是决策者 / 不是节点**：不替代会计师签字、不构成 governance approval、不成为 entity/rule/case authority（权威文档据 HRN 02:111、02:125）。
- **写入耦合 Finalization、确定性同步写**：经 Finalization 落盘的每一笔扩张型变更，耦合触发一条同步审计写入；写入由确定性代码承担（Finalization L2 决策 9）。
- **触发节点 = Human Review Node** 的扩张型改动落盘那一刻（原 Governance Review 已废除）。
- **三轴分工**：vs Transaction Log（per-交易）、vs 各权威层（per-对象当前态 + ref）、vs Intervention Log（交互留痕、runtime 不读）、vs 审核 inbox（候选队列）——互不替代。
- 以上「已锁」均来自权威文档对 A 类邻居正文的转引；正式深读 A 类邻居时须逐条回指原文校验。

## C. 记忆层模板 M1-M2 完成清单（第 2 步梳理问题的标尺 + Proposal 自检表）

> 逐字搬运自 `memory_layer_spec_template_rules.md` §10。一份 M1-M2 memory layer spec 完成前必须能回答：

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
18. 是否把读取者 / 写入者声明为对外接口面，reader 只依赖声明内容？
19. 是否守住运行 / 记忆 seam——只定写入机制，没有去重判内容该不该写？
20. 哪些问题未冻结？
21. 是否可以进入 M3 data contract？

## D. 记忆层模板禁止写法（Proposal 严禁出现）

> 逐字搬运自 `memory_layer_spec_template_rules.md` §9：

- "保存相关信息。"
- "必要时更新 memory。"
- "LLM 判断后写入。"
- "高置信度自动沉淀为记忆。"
- "之后由 governance 处理"，但不说明是否已冻结 governance path。
- "与其他 logs 保持一致"，但不说明哪个 store 是 source of truth。
- "可被其他节点读取"，但不说明 reader、目的和限制。
- "写入 trace"，但不说明 trace 是否能成为 authority。

## E. M1-M2 写作深度（当前阶段）

- **M1 Memory Intent**：固定为什么存在、保存什么、不保存什么。
- **M2 Authority, Lifecycle, and Boundaries**：固定读写 authority、candidate 边界、lifecycle、已知冲突行为。
- M3（data contract）只有字段和消费者稳定时才写；当前**不**提前冻结字段、跨 memory 边界或 governance path；未冻结依赖写成 Open Boundary（memory 模板 §3）。
