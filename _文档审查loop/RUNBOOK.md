# 夜间文档审查 Loop · RUNBOOK（每次迭代的大脑）

本文件是 loop 每次迭代的唯一行为准则。不要凭记忆，每次都重读本文件 + `WORKLIST.md`。

---

## 0. 铁律（违反即作废）

1. **只读 + 追加。绝不修改、删除、重排任何被审查的文档。** 唯一允许写入的是本目录 `_文档审查loop/` 下的三个产物：`RUNBOOK.md`（一般不动）、`WORKLIST.md`（只改状态）、`FINDINGS.md`（只追加）。
2. **每条 finding 必须可验证**：必带真实 `文件:行号` + 原文引用片段。无法引用到具体行的，不写。
3. **不下命令式结论，只产候选 + 证据 + 置信度**。判断权在 owner，明早人工审核。尤其孤儿 / 反孤儿 / 矛盾三类，一律标"候选，需人工核"。
4. **宁可标低置信度，不可臆造**。拿不准 = 低置信度候选，绝不断言。
5. **吸取 `evidence_condition` 教训**：一个字段在 A 节点像"无主"，可能在 B 节点（尤其 `memory_layers/*/01_memory_intent.md` 的字段表）有娘家。判"孤儿"前必须跨**所有层**穷尽追溯，并在 finding 里写明"查了哪些地方、哪里没找到"。

---

## 1. 文档分层与扫描范围

**权威层（live，残留=真问题）**
- `BK_Copilot/**`（workflow_nodes / memory_layers / shared_mechanisms）
- `L3_schema/**`
- `Decisions.md`、`缺口地图.md`、`当前任务状态.md`、`AGENTS.md`、`审计目标与原则.md`

**来源/中间层（已被权威层取代，按 P7 处理：标"过时来源"，不当纯残留）**
- `L2/L2_proposals/**`、`L2/_audit_work/**`、`L2/独立question文档/**`
- `BK_Copilot/memory_layers/case_log/case_log_boundary_note.md`（标 superseded，却仍被引为正式草案来源 → 高危）

**排除（不扫，命中也忽略）**
- `废弃文档/**`、任何 `BK-obsidian/**`、`.git/**`、本目录 `_文档审查loop/**`

---

## 2. Kill-list（已删概念 / 已废节点的权威清单）

判残留时以此为准。命中后**必须区分 provenance（保留）vs 活残留（报告）**——见 §4。

| 已删概念 | 决策 | 注意 |
|---|---|---|
| `automation_policy` | D8 → 拆 `force_pending` + `promotion_lock` | — |
| `automation_control_state` | D8/D10（桶B 放行闸字段） | — |
| `candidate_signal` | D1 | — |
| `merge_split_candidate` | D1 | ⚠️ merge/split **概念保留**（会计师人发起），只删"自动候选字段" |
| `alias_conflict_issue` | D1 | — |
| `identity_governance_issue` | D1 | — |
| `blocking_reason` | D1 | — |
| `identity_risk_flags` | D1 | — |
| `evidence_condition` | D10（仅从 RM 输入删） | ⚠️ **已知矛盾**：Case Log 仍把它当正式字段（见 F-002）。D10 理由"无稳定衡量标准"是全局口径，与 Case Log 设计冲突。凡再遇到，归 E 类（决策-文档矛盾），不要简单当孤儿 |
| `transaction_type` | D10 | 真孤儿（全库无产出方） |
| `objective_tags` | D10 | 真孤儿 |
| `rule_ref`（裸用、未改名） | D9 → `rule_id` | 只报"还当活字段用"的裸 `rule_ref`；"原 Rule ref / 原称"是 provenance |
| `rule_match_source`（**不该存在的新造名**） | D10（决定复用 `confirmed_by`，不新造） | 一旦出现即报 |

**已废节点**（live 文档若仍把它当"现存读取者/路径"即为残留）：
- `Governance Review Node`（治理审核节点）
- `Post-Batch Lint Node`（批后体检节点）
- `语义发现器` / `FP-5` / `系统自发语义发现层`（已裁撤）

---

## 3. 检测器清单（WORKLIST 的项就是它们的实例）

- **P1 残留扫描**：对 kill-list 每个词，在权威层 grep，逐处分类 provenance vs 活残留。
- **P2 废节点扫描**：对每个已废节点，找 live 文档里"把它当现存"的引用（读取者列表、路径图、流程依赖）。
- **P3 孤儿字段扫描**：从 `L3_schema/字段链路大表.md` + 各节点输入/输出 + 各 memory layer 字段表，建字段清单；对每个字段跨全库追 **producer（谁 create/emit）** 与 **consumer（谁 read/消费）**；两者皆无 → 孤儿候选。**必须穷尽追溯（含 memory_intent 字段表）。**
- **P4 反孤儿 / 缺失基础件**：找被 ≥1 节点实际 read/消费、却**无正式定义/schema/落点**、长期挂在 "L3 open / 待定 / 未冻结" 的概念。
- **P5 决策-文档矛盾**：对 D1–D10 每条，找 live 文档里与之直接冲突的陈述（`evidence_condition` 即此类）。
- **P6 跨文档字段定义矛盾**：共享字段（`confirmed_by` / `force_pending` / `promotion_lock` / `entity_id` / `evidence_refs` vs `identity_evidence_refs` / `rule_id` 等）在不同文档的定义/口径是否一致。
- **P7 过时来源层**：`L2/L2_proposals/*` 各份结论 vs `BK_Copilot/` 对应正式草案的分歧；标 superseded 却仍被 live 文档引为来源的文件。
- **P8 悬空引用**：文档里指向的 文件 / 章节 / 字段 是否真实存在。

---

## 4. provenance vs 活残留 判别

- **provenance（保留，归 A′，仅参考、不建议改）**：句子在**记录一次删除/废除**。标志词：`已删` `已删除` `已废除` `已裁撤` `原 X`（后接"已删/改为"）`见 Decisions` `（原称…）` `不再输出` `不再设`。
- **活残留（报告，归 A）**：把死概念**当现行字段/现存节点/有效路径**在用——出现在字段表、输入/输出契约、读取者列表、流程依赖里，且**没有**删除标注。
- 拿不准 → 低置信度 A 候选，注明"疑似 provenance，待核"。

---

## 5. FINDINGS 写入格式（追加到 `FINDINGS.md` 末尾）

```
### F-<编号> · <一句话标题>
- 类型: A残留 | A′provenance(参考) | B来源层过时 | C孤儿候选 | C′缺失基础件候选 | D近义重复 | E决策-文档矛盾 | G悬空引用
- 位置: <file:line>（可多处，逐一列）
- 证据: "<原文片段>"
- 判定理由: <为什么是问题；孤儿/矛盾类必须写"查了哪些层、哪里没找到">
- 置信度: 高 | 中 | 低
- 建议动作(供审核, loop 不执行): 删 | 改 | 立 | 标superseded | 仅确认
- 关联: <Decision Dn / 其它 F-编号>
```

编号全局递增，不复用。同一问题跨多文件 = 一条 finding 多个位置，不要拆。

---

## 6. 迭代协议（每次唤醒做什么）

1. 重读本 RUNBOOK + `WORKLIST.md`。
2. 取 WORKLIST 中**第一个 `status: todo`** 的项，做它的检测（一次 1–3 项，视大小）。
3. 把发现按 §5 追加进 `FINDINGS.md`；**该项即使零发现也要记一行"零发现"**，便于明早知道已扫过。
4. 把该项标 `done`，更新 `WORKLIST.md` 顶部计数。
5. **若仍有 todo**：安排下一次唤醒（短延迟，纯计算、不等外部）继续。
   **若全部 done**：在 `FINDINGS.md` 顶部写"==== 全部完成：共 N 条 finding，扫描 M 项 ===="，**结束 loop（不再安排唤醒）**。
6. 任何一步出错 → 把错误记进 `FINDINGS.md`（标 `[LOOP-ERROR]`），跳过该项、继续，绝不修改被审文档。
