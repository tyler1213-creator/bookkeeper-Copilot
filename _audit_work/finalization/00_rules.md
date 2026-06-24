# 00_rules — Finalization 审计必守规则蒸馏

> 本轮审计对象 **Finalization（统一 finalization 写入机制）** 是「第三类对象：跨层共享机制」（非 workflow node、非 memory layer）。落正式文档时模板归属另定（B 文档 §文件角色）。
> 历史工作区提示：若后续复制复用本文件，以根目录 `AGENTS.md`、`当前任务状态.md` 的最新口径为准；`new system/` 已作废且不再作为审计目标，`new system/` 与 `old_system_nodedesign/` 未经用户明确授权不得读取。

## A. 必守通用规则（带出处）

1. **Source authority 铁律（A/授权历史材料分层）** — 审计目标与原则.md / AGENTS.md：正式 `BK_Copilot/` 文档 + 已锁事实是唯一设计约束来源（A 类）；`new system/` 已作废且不再作为审计目标，`new system/`、`old_system_nodedesign/` 未经用户明确授权不得读取；即使获准读取，也只作历史追溯 + 问题发生器，永不进设计、不当结构模板、不当正确性证明。Proposal 每条契约必须指回 A 原文或用户拍板决定。
2. **可编辑范围** — 只产出 `L2_proposals/` 提案 + `_audit_work/` 工作区；不创建/编辑 `BK_Copilot/` 正式文档、不改已锁文件；未经用户明确授权，不读取或编辑 new/old system。
3. **停止并问用户** — 出现真实产品决策点、B 类假设与 A 类/已锁事实冲突、关键 A 源按路径找不到、output 的 consumer 无法从已锁结论确定、正式文档之间或与已锁不变量矛盾 → 停下报告，不得用 B 覆盖 A。
4. **守「运行 / 记忆」seam** — 各节点 02「交给记忆/finalization 层持久化的内容」一节统一表述：节点（运行层）只声明「存什么 + 谁有权威认定它有效」，**不声明「怎么写、谁来写、什么顺序写」**（见 rule_match 02:58、human_review 02:50、case_judgment 02:90、coordinator 02:59）。Finalization 在「机制」侧，**只决定「怎么写」，不重判「该不该写、内容对不对」**（B §七，待 A 化）。
5. **契约面规则** — 每个 output 写明 consumer，不与已锁下游抵触；契约面只能从 A 类重建。

## B. 与 Finalization 直接相关的「已锁事实」（遵守，非审查对象）

来源 = 缺口地图.md §13-19 + 各正式 02 + human_review_node 正式草案：

1. **全系统唯一落盘地板**：各 memory layer / 节点 02 中「统一写入 / exact writer 未冻结 / 多 log finalization 顺序」**均指向它**；它拥有跨 log 原子 / 幂等 / append-only / 凭证校验不变量；**节点不裸写、只供 payload**。（缺口地图 §13）
2. **凭证强制半边住在 Finalization**：原「授权确认」已分解 —— UX/确认/调度 → Human Review Node 正式草案；**凭证强制半边并入 Finalization**。（缺口地图 §14）
3. **凭证按写入类型开、不按谁发起开**：扩张型长期权威变更（rule 升/改/删/降、automation 放宽、merge/split、永久纠错）→ 要凭证；低半径单笔纠错、restrictive auto-downgrade、Coordinator 运行中身份确认 → 免闸。（缺口地图 §14）
4. **凭证 = 已记录的会计师 read-back 签字**：扩张型变更的批准是会计师 read-back 签字这条唯一治理线；**Finalization 凭证校验 = 放行权死代码**，缺凭证拒绝扩张型写入（human_review 02 §5「Finalization 死代码可以决定」:80-83；01:35）。
5. **Governance Log 是审计账本、非决策者**：只保存经签字并落盘的扩张型变更审计，不替代 accountant approval（human_review 02 §5 Governance 必须批准段）。
6. **Trusted base 要小**：放行权做成 Finalization 死代码，哪怕 Human Review 编排出错也越不过它（B §五，与 human_review 01:35 一致）。
7. **append-only 已锁**：Transaction Log append-only、原记录永不就地覆盖/删除（transaction_log 02:77；02:187 本轮只锁 append-only 原则 + 更正以追加承载）。

## C. 模板 Stage 1-2 标尺（问题清单 & Proposal 自检表）

来自 `workflow_node_spec_template_rules.md`（注：Finalization 落正式文档时模板另定，但 Stage 1-2 完成度标准可借用为「足以转正式文档」的判据）：

- **Stage 1（00/01 功能意图）必答**：为什么必须存在 / 删除失去什么；唯一核心职责；明确排除范围与越权禁止；workflow 位置与理由；对核心产品目标贡献。
- **Stage 2（02 逻辑边界）必答**：触发条件与禁止触发；上游前置条件及缺失行为；读取对象 + authority 限制；写入对象分类（直接 durable / 交记忆层 / candidate / 禁止）；权限分布（代码/LLM/人工/治理）；输出类别 + consumer 契约表；证据缺失/歧义/冲突处理；audit trace 边界。
- **禁止写法**：「负责协调一切」「必要时可更新相关 memory」「LLM 自行判断」「高置信度则自动通过」「后续实现再决定」「参考旧系统」；「写入日志」却不说哪个 log/字段/authority；「生成候选」却不说能否持久化/谁批准/谁消费。

> 注意：本对象是「机制」非「节点」，模板的「触发条件 / runtime 位置」等需按机制语义改写；动作集（create/update/append/supersede）+ 凭证闸门是其特有契约面。
