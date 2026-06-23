# Finalization 写入机制

## 文档状态

- Current standard status: draft
- Object type: 第三类对象 · 跨层共享机制；非 workflow node，非 memory layer。
- Template basis: 采用 `BK_Copilot/node:layer spec template/workflow_node_spec_template_rules.md` 的 00 / 01 / 02 结构，并按四处改写适用：调用关系替代流水线工位、写入对象反转、增加动作集 / 凭证闸门 / 回执失败、运行 / 记忆 seam 反向。
- Dedicated template: 无。本类当前只有 Finalization 一个成员，不另立通用模板。
- Covered stages:
  - [x] Stage 1: Functional Intent
  - [x] Stage 2: Logic and Boundaries
  - [ ] Stage 3: Data Contract
  - [ ] Stage 4: Execution Algorithm
  - [ ] Stage 5: Technical Implementation
  - [ ] Stage 6: Tests and Fixtures
  - [ ] Stage 7: Agent Task Contract
  - [ ] Stage 8: Maintenance
- Last reviewed: 2026-06-23
- Owner / reviewer: 待填

## 当前标准文件

- `01_functional_intent.md`
- `02_logic_and_boundaries.md`

`03_data_contract.md` 尚未创建；本轮只覆盖 Stage 1-2，不为结构完整创建空文档。

## 当前未冻结边界

### L3（字段 / enum / schema）

- 动作 payload exact field schema、动作 exact 定义和动作类型 enum。
- 凭证 exact 形态、生成与传递机制、防重放、签字 ref / 范围与会计师签字捕获方式。
- 回执 exact schema；L2 只锁定成功 / 失败 + 失败结构化原因，且不返还 ID。
- ID scheme；L2 只锁定 ID 由调用方铸造，`create` / `append` 产生新记录，`update` / `supersede` 引用现有记录。

### L4 / seam（机制）

- 多 log finalization 写入顺序、原子 / 回滚、幂等键。
- 共享库 + 单一事务边界的 exact 实现；L2 已否决中间写入 agent / 单一全局 LLM 写手。
- 写成功后的同步可见性（read-after-write）保证机制。
- 失败回滚、重试、部分失败呈现的 exact 机制。

### L2·外阻（圈外依赖）

- Governance Log 数据层及“记录哪些变动”的判定标准（待建）。
- 审核 inbox 数据层（待建）。
- Intervention Log 正式 spec（待建）。
- merge / split 标准化动作批模板与跨 log 迁移完整性契约。
- Finalization 第三类正式 spec 归属目录命名在进入 Stage 3 前需确认；本轮按用户指令落入 `BK_Copilot/shared_mechanisms/finalization/`。

## 进入下一阶段前必须解决

- 圈外依赖落文：Governance Log、审核 inbox、Intervention Log、merge / split 跨 log 迁移契约。
- L3 字段定稿：动作 payload schema、凭证形态、回执 schema、ID scheme。
- 正式 spec 归属目录命名确认：第三类共享机制是否继续采用 `BK_Copilot/shared_mechanisms/finalization/`。
