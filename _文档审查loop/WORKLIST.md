# 夜间审查 WORKLIST（loop 队列）

进度: 41 / 41 done（✅ 全部完成，loop 已结束）· finding 数见 FINDINGS.md（F-001..F-022 + 1 条 provenance 参考）

> 规则：每次取第一个 `todo`，做完标 `done`（即使零发现）。全部 done → 结束 loop。

## P1 残留扫描（kill-list 逐词）
- [x] P1.1 `automation_policy` — status: done（F-003 残留×3行, F-004 残留, +provenance 参考）
- [x] P1.2 `automation_control_state` — status: done（权威层 0 命中，F-005）
- [x] P1.3 `candidate_signal` — status: done（全 provenance，F-005）
- [x] P1.4 `merge_split_candidate`（注意 merge/split 概念保留） — status: done（全 provenance，F-005）
- [x] P1.5 `alias_conflict_issue` — status: done（全 provenance，F-006）
- [x] P1.6 `identity_governance_issue` — status: done（全 provenance，F-006）
- [x] P1.7 `blocking_reason` — status: done（全 provenance，F-006）
- [x] P1.8 `identity_risk_flags` — status: done（ER_L3:54 risk_flags 残留确认，F-006）
- [x] P1.9 `evidence_condition` — status: done（F-007 扩展 F-002：系统级 live，矛盾跨 CaseLog+EI+缺口地图）
- [x] P1.10 `transaction_type` — status: done（全 provenance，真孤儿正确删，F-008）
- [x] P1.11 `objective_tags` — status: done（全 provenance，F-008）
- [x] P1.12 裸 `rule_ref` — status: done（F-009 命名残留，散布 TxLog/RuleLog/HR，D9 已 defer）
- [x] P1.13 `rule_match_source` — status: done（零 live，决定守住，F-008）

## P2 废节点扫描
- [x] P2.1 `Governance Review Node` — status: done（冻结档全 provenance；缺口地图:221/263 低置信残留，F-010）
- [x] P2.2 `Post-Batch Lint Node` — status: done（同上，F-010）
- [x] P2.3 `语义发现器 / FP-5 / 系统自发语义发现层` — status: done（全 provenance，极干净，F-010）

## P3 孤儿字段扫描（穷尽追 producer + consumer）
- [x] P3.1 `L3_schema/字段链路大表.md` 全字段 producer/consumer 追溯 — status: done（F-011：链内无真孤儿；structural_path_status 开口型依赖 + counterparty_signals node档↔L3 分歧转 P3.2）
- [x] P3.2 Evidence Intake 节点 输入/输出字段 — status: done（F-012：F-011 分歧解除；source_account/source_channel/evidence_type 软消费候选）
- [x] P3.3 Entity Resolution 节点 字段 — status: done（F-013：输出全接地；ER_L3:54 过度声明读未建源 + 重申 risk_flags/automation_policy 残留）
- [x] P3.4 Rule Match 节点（含 L3 schema）字段 — status: done（F-014：全接地；amount_abs/date 命名漂移 vs 权威 amount/transaction_date）
- [x] P3.5 Case Judgment 节点 字段 — status: done（F-015：CJ 是 evidence_condition 第四活跃站点，强化 F-007；CJ 是下一节点）
- [x] P3.6 Coordinator 节点 字段 — status: done（F-016：干净，单输入面、外圈落点已挂开口）
- [x] P3.7 Human Review 节点 字段 — status: done（F-017：读写面干净；candidate context 接地待核 + rule ref 命名 F-009）
- [x] P3.8 memory layers 字段表 — status: done（F-017：无新孤儿；evidence_condition=Case Log 字段#11 确认；date/amount 命名漂移）

## P4 反孤儿 / 缺失基础件
- [x] P4.1 反孤儿/缺失基础件 — status: done（F-018：confirmed_by/force_pending/eligibility router 多头寄存无 owner）
- [x] P4.2 缺口地图待定但已被引为输入的字段 — status: done（F-018：缺口地图覆盖充分，隐藏型风险低）

## P5 决策-文档矛盾（D1–D10 逐条）
- [x] P5.1–P5.10 D1–D10 — status: done（F-019：D1–D7 传播干净；矛盾收敛到 F-002/3/4/6/7/9/15）

## P6 跨文档字段定义矛盾
- [x] P6.1 共享字段口径一致性 — status: done（F-020：confirmed_by 语义分裂；direction enum 已知开口）

## P7 过时来源层
- [x] P7.1 `L2/L2_proposals/*` vs `BK_Copilot/` 分歧 — status: done（F-021：冻结节点 L2 提案无 superseded 标记；CJ 提案仍活跃例外）
- [x] P7.2 `case_log_boundary_note.md` superseded 被引为来源 — status: done（已记 F-001）
- [x] P7.3 其它 superseded 仍被引用文件 — status: done（F-021：主要是 L2 提案 + case_log_boundary_note）

## P8 悬空引用
- [x] P8.1 文档间 文件/章节/字段 引用 — status: done（F-022：1 处真悬空〔模板示例路径少 EI_Schema/〕，余为伪命中/模板占位）
