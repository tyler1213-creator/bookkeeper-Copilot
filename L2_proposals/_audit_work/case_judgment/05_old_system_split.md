# 旧系统 — 交易拆分（Split）逐字引文与出处索引

> 全部材料为 **历史材料，不具权威性**。本文件仅做逐字引文与出处距离化（distillation），不含任何判断、问题清单、建议或分析。
> 范围说明：`je_generator_spec.md` / `observations_spec_v2.md` / `profile_spec.md` 中出现的"拆分"均指 **HST 金额拆分（pre_tax + hst_amount）**，与"一笔银行交易拆为多笔真实交易"无关，已排除在本距离化之外（见末尾"排除项"）。

---

## 焦点问题 1：Definition — "拆分"在旧系统的含义（一笔银行交易 → 多笔真实/子交易）

**[历史材料，不具权威性]**
> "调用 split_transaction.py 拆分交易为多条标准格式"
— 出处：`coordinator_agent_spec_v3.md` §1 文件树注释（行 98）

**[历史材料，不具权威性]**
> | 要求拆分 | "这笔 $5000 其中 $3000 是材料，$2000 是老板自己买的" | 调用 split_transaction.py 拆分 |
— 出处：`coordinator_agent_spec_v3.md` §需要处理的回复类型表（行 212）

**[历史材料，不具权威性]**
> "**用途**：将一笔交易拆分为多条标准格式交易数据"
> "**输入**：original_transaction, splits（每条包含 amount 和 context）"
> "**执行逻辑**：验证 splits 金额之和 = 原始金额 → 生成多条标准格式交易数据"
> "**约束**：金额之和必须等于原始交易金额，不等则报错拒绝执行"
— 出处：`coordinator_agent_spec_v3.md` §6 Scripts / split_transaction.py（行 317-322）

**[历史材料，不具权威性]**
> "transaction_id ... 它是 split / review / intervention / audit chain 的统一关联键"
— 出处：`transaction_log_spec.md` §字段说明 / 第一层（行 73）

---

## 焦点问题 2：Detection — 谁/什么检测到一笔交易需要拆分

**[历史材料，不具权威性]**
> | 要求拆分 | "这笔 $5000 其中 $3000 是材料，$2000 是老板自己买的" | 调用 split_transaction.py 拆分 |
— 出处：`coordinator_agent_spec_v3.md` §需要处理的回复类型表（行 212）
（语境：检测发生在 Coordinator 解析 **accountant 回复** 的阶段，即 accountant 主动提出拆分要求。）

**[历史材料，不具权威性]**
> | Accountant | 确认拆分交易的金额明细 | Accountant 要求拆分但未给出精确金额时 |
— 出处：`coordinator_agent_spec_v3.md` §8 与人的交互表（行 393）

**[历史材料，不具权威性]**（预处理阶段不参与）
> "`supplementary_context` 字段：预处理阶段不写入；仅由 Coordinator Agent 在拆分交易后子交易重新走 workflow 时注入额外说明"
— 出处：`data_preprocessing_agent_spec_v3.md` §字段说明（行 353）

---

## 焦点问题 3：Who performs the split — 谁执行拆分、子交易如何被创建

**[历史材料，不具权威性]**
> "#### 操作：拆分交易
> →  调用 split_transaction.py
>   约束：splits 金额之和必须等于原始交易金额
> →  每条子交易带着 accountant 提供的业务信息
> →  如果 accountant 已给出每条子交易的明确分类 → 每条走情况 A
> →  如果 accountant 只给出了金额拆分和模糊描述 → 每条走情况 B
> →  如果 accountant 只给出了金额拆分未说明用途 → 每条子交易调用 retrigger_workflow.py 从 Node 1 重新走 workflow"
— 出处：`coordinator_agent_spec_v3.md` §Step 4 操作：拆分交易（行 272-281）

**[历史材料，不具权威性]**（权限边界确认执行主体为 Coordinator）
> "✅ 拆分交易为多条标准格式"
> "✅ 触发 workflow 重处理"
> "✅ 向交易注入 supplementary_context"
— 出处：`coordinator_agent_spec_v3.md` §7 权限边界 / 允许（行 364-366）

**[历史材料，不具权威性]**（创建机制：脚本生成多条标准格式交易数据，见焦点 1 的 split_transaction.py 条目）

---

## 焦点问题 4：拆分后原交易与子交易的去向 — 子交易是否重入 workflow、从哪个节点、精确路由

**[历史材料，不具权威性]**（三条路由分支，逐字）
> "→ 如果 accountant 已给出每条子交易的明确分类 → 每条走情况 A
> →  如果 accountant 只给出了金额拆分和模糊描述 → 每条走情况 B
> →  如果 accountant 只给出了金额拆分未说明用途 → 每条子交易调用 retrigger_workflow.py 从 Node 1 重新走 workflow"
— 出处：`coordinator_agent_spec_v3.md` §Step 4 操作：拆分交易（行 278-280）

**[历史材料，不具权威性]**（retrigger 的 start_node 选择）
> "**start_node 选择**：
>   - 拆分交易后 accountant 未给出分类 → Node 1"
— 出处：`coordinator_agent_spec_v3.md` §6 retrigger_workflow.py（行 328-329）

**[历史材料，不具权威性]**（下游去向表）
> | 拆分交易后 accountant 未给出分类 | Node 1（Profile 匹配） | 每条子交易从 Node 1 走完整 workflow |
— 出处：`coordinator_agent_spec_v3.md` §9 下游表（行 424）

**[历史材料，不具权威性]**（原交易记录的更新）
> "交易被拆分：
>   → 原始交易记录更新 child_transaction_ids
>   → 为每个子交易创建独立的 Transaction Log 记录，parent_transaction_id 指向原始交易"
— 出处：`transaction_log_spec.md` §4 状态与生命周期（行 171-173）

---

## 焦点问题 5：置信度分类器（Node 3）在拆分中的角色

**[历史材料，不具权威性]**（Node 3 不检测、不执行拆分；仅在子交易重入时接收 supplementary_context）
> | `supplementary_context` | Coordinator 在拆分交易后子交易重新走 workflow 时注入的额外说明 | 非空时提供 |
— 出处：`confidence_classifier_spec.md` §2.4 信息源装配表（行 96）

**[历史材料，不具权威性]**（supplementary_context 在 Node 3 信息源优先级中的位置）
> "1. **blocking risk packs**
> 2. **supplementary_context**
> 3. **accountant_notes**
> ..."
— 出处：`confidence_classifier_spec.md` §2.6 信息源优先级（行 126-128）

**[历史材料，不具权威性]**（supplementary_context 作为强覆盖证据）
> | `supplementary_context_confirmed_purpose` | `transaction.supplementary_context` | accountant 已明确提供业务用途，强覆盖证据 |
— 出处：`confidence_classifier_spec.md` §override evidence 表（行 188）；亦见 §override evidence 简表（行 29）、行 251、行 281 列出 `supplementary_context_confirmed_purpose`

**[历史材料，不具权威性]**（Node 3 只判 HST 处理方式，不做金额拆分 — HST 语境）
> "HST 判断只输出方式，不做金额拆分"
— 出处：`confidence_classifier_spec.md`（行 161）
> "LLM 只判断 HST 处理方式（inclusive / exempt / unknown），金额拆分由下游 build_je_lines.py 完成。"
— 出处：`confidence_classifier_spec.md`（行 416）

**[历史材料，不具权威性]**（注入工具，仅子交易重走时使用）
> "inject_context.py — **用途**：向指定交易的 supplementary_context 字段写入信息 ... **说明**：仅在拆分交易后子交易需要重新走 workflow 时使用"
— 出处：`coordinator_agent_spec_v3.md` §6 inject_context.py（行 310-315）

---

## 焦点问题 6：Recording / Logging — 拆分如何记录（父子链接、Transaction Log）

**[历史材料，不具权威性]**（父子字段定义）
> "**parent_transaction_id**：如果该交易是拆分后的子交易，指向原始交易的 transaction_id"
> "**child_transaction_ids**：如果该交易被拆分，指向所有子交易的 transaction_id 列表"
— 出处：`transaction_log_spec.md` §字段说明 / 第五层 元数据（行 124-125）

**[历史材料，不具权威性]**（示例字段初值）
> "supplementary_context: \"\""
> "child_transaction_ids: null"
— 出处：`transaction_log_spec.md` §示例（行 60、行 66）

**[历史材料，不具权威性]**（生命周期中的记录动作）
> "交易被拆分：
>   → 原始交易记录更新 child_transaction_ids
>   → 为每个子交易创建独立的 Transaction Log 记录，parent_transaction_id 指向原始交易"
— 出处：`transaction_log_spec.md` §4（行 171-173）

**[历史材料，不具权威性]**（审核 Agent 对 Log 的修改触发条件）
> | 更新 child_transaction_ids | 交易被拆分 |
> | 新增子交易记录 | 拆分交易时为每个子交易创建记录 |
— 出处：`transaction_log_spec.md` §6 被谁修改 / 审核 Agent（行 207-208）

**[历史材料，不具权威性]**（supplementary_context 写入来源）
> "**supplementary_context**：Coordinator Agent 在处理拆分交易后子交易需要重新走 workflow 时注入的额外说明；其他情况为空字符串"
— 出处：`transaction_log_spec.md` §字段说明 / 第四层（行 118）
> | 第四层（补充证据） | 数据预处理 Agent（receipt / cheque_info），Coordinator Agent（supplementary_context） |
— 出处：`transaction_log_spec.md` §3 各层字段写入来源（行 149）

**[历史材料，不具权威性]**（Coordinator 可修改文件表）
> | 交易数据 | 写入 supplementary_context | 拆分交易后子交易需要重新走 workflow 时 |
> | 交易数据 | 拆分为多条 | Accountant 要求拆分 |
— 出处：`coordinator_agent_spec_v3.md` §9 可修改的文件（行 434-435）

---

## 结构化索引（出处速查）

| 焦点 | 文件 | 行号 / 章节 |
| --- | --- | --- |
| Definition | coordinator_agent_spec_v3.md | 行 98, 212, 317-322；transaction_log_spec.md 行 73 |
| Detection | coordinator_agent_spec_v3.md | 行 212, 393；data_preprocessing_agent_spec_v3.md 行 353 |
| Who performs | coordinator_agent_spec_v3.md | 行 272-281, 317-322, 364-366 |
| 拆分后去向/路由 | coordinator_agent_spec_v3.md | 行 278-280, 328-329, 424；transaction_log_spec.md 行 171-173 |
| Node 3 角色 | confidence_classifier_spec.md | 行 29, 96, 126-128, 161, 188, 251, 281, 416；coordinator_agent_spec_v3.md 行 310-315 |
| Recording / Logging | transaction_log_spec.md | 行 60, 66, 118, 124-125, 149, 171-173, 207-208；coordinator_agent_spec_v3.md 行 434-435 |

### 关键脚本一览（逐字出处）
- `split_transaction.py` — coordinator_agent_spec_v3.md 行 317-322（执行拆分，校验金额之和=原始金额）
- `inject_context.py` — coordinator_agent_spec_v3.md 行 310-315（写 supplementary_context，仅子交易重走时）
- `retrigger_workflow.py` — coordinator_agent_spec_v3.md 行 324-330（触发重入，未给分类时 start_node=Node 1）

### 排除项（"拆分"=HST 金额拆分，非交易拆分，不在本距离化范围）
- `je_generator_spec.md` 行 184-186, 273（HST pre_tax + hst_amount 拆分公式）
- `observations_spec_v2.md`（无交易拆分相关命中）
- `profile_spec.md` 行 26, 73, 75-77, 140（tax_config inclusive 拆分计算）
- `output_report_spec.md` 行 43（Pre-tax = HST 拆分后税前金额）
- `review_agent_spec_v3.md` 行 95-97, 407-433, 536-563, 664, 730（pattern 的合并/拆分，针对 Pattern Dictionary，非交易拆分）
