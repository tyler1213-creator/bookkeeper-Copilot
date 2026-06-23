# Human Review Node

## 文档状态

- Current standard status: draft
- Covered stages:
  - [x] Stage 1: Functional Intent
  - [x] Stage 2: Logic and Boundaries
  - [ ] Stage 3: Data Contract
  - [ ] Stage 4: Execution Algorithm
  - [ ] Stage 5: Technical Implementation
  - [ ] Stage 6: Tests and Fixtures
  - [ ] Stage 7: Agent Task Contract
  - [ ] Stage 8: Maintenance
- Last reviewed: 2026-06-22
- Owner / reviewer: 待填

## 当前标准文件

- `01_functional_intent.md`
- `02_logic_and_boundaries.md`

## 当前未冻结边界

### L3（字段 / enum / schema）

- `confirmed_by` exact enum；Intervention ID / correction record schema。
- Change List（`P_llm` / `P_engine`）和系统变化执行图的 exact schema。
- 凭证 exact 形态、完整 read-back 模板 / 字段 / data contract。
- entity 税务状态字段（HST / GST 查表依赖）和 source 字段形态。

### L4 / seam（机制）

- Change List Engine 依赖图的声明位置 / 形态。
- Finalization 调用契约、schema 校验、凭证校验、写入顺序、原子回滚、幂等键。
- 多个并发待确认项触及同一 rule / entity 时的排序、互斥、去重。
- N 笔批量 correction append 的顺序、原子、部分失败呈现。
- read-back UI 与只读复核视图是否同一 shell；QuickBooks / 自有 DB 等多来源呈现。
- “rule 被推翻一次”信号的 exact 落点与累积判定。

### L2·外阻（圈外依赖）

- Intervention Log 正式 spec。
- Governance Log 数据层（待建）。
- 审核 inbox 数据层（待建）。
- Finalization 写入机制 / 确定性发现 job 正式 spec。
- 系统自发发现层（FP-5 + 语义发现器）是否必要：挂起；本节点当前只承接会计师人发起的 merge / split 与纠错。
- NEW-1：rule 升级“固定执行路径”的定义归 Rule 侧（Rule Log / Rule Match 治理），本节点只触发。
- C2：本节点成品 Chatbot 是否与 Coordinator / interaction_agent 共用同一对话前端，归编排 / 呈现层。
- A 类残留待写回：五个 log + Coordinator 正式草案中旧 writer 模型 / 旧 Review Node 引用的写回，不在本节点文件内执行。

## 进入下一阶段前必须解决

- 圈外依赖正式落文：Intervention Log、Governance Log、审核 inbox、Finalization 写入机制、确定性发现 job。
- L3 字段定稿：`confirmed_by`、correction record、Change List、凭证、read-back、entity 税务字段、source 字段。
