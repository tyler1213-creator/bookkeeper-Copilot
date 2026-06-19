# Intervention Log Question

## 文件角色

本文件记录 Intervention Log 当前已确认的临时结论，是 Coordinator L2 审计的副产物入口。Intervention Log 记忆层/节点尚未正式审计；本文**不是正式 spec**，不冻结 schema、字段、writer 或 data contract。后续正式审计 Intervention Log 时，以本文作为本轮讨论结论入口。

与 `Coordinator Question.md`、`interaction_agent.md` 同级存放（均为"讨论结论入口"类文档，非正式 spec）。

## 背景

Coordinator 在与 accountant 的每次交互（结构化提问、回答、确认、纠正、追问、情况 X / 情况 Y 原因）中产生交互痕迹。Coordinator L2 决策 9 已确认：**Intervention Log 确定存在**，由 Coordinator 写入这些交互留痕。

## 当前确认结论

### 唯一作用：离线系统改进材料

- Intervention Log 的作用**有且只有一个**：帮助系统设计者根据记录改进系统缺陷、提升系统性能，使系统更智能、更自动化。
- 它**不承担任何运行期（runtime）辅助判断工作**：系统中**没有任何节点读取 Intervention Log 用于判断**。Coordinator 自身也不读取它（Coordinator L2 决策 11：Coordinator 无独立读取面，全部上下文随 CJ Pending 在手）。
- 即：Intervention Log 是**只写**的离线学习 / 改进材料，不是 runtime 判断输入。

### 写入者与内容

- 写入者：Coordinator（决策 9）。
- 内容：每次与 accountant 的交互留痕——提问、回答、确认、纠正、追问、情况 X / 情况 Y 的原因。

### 边界：留痕 ≠ authority

- 交互留痕本身**不构成** Entity Log / Case Log / Rule Log authority。
- 要把交互转成 durable 记忆 / 规则 / policy，仍须经治理 / review / case memory 等正规路径（`Coordinator Question.md`）。

## 待确认 / 本轮新提

- **审计边界（本轮新提，需用户拍板；可能 refine 决策 9 的"审计追溯"措辞）**：交互的**结果审计**（哪个 stable entity 因 accountant 哪次确认而创建、哪笔交易因 accountant 选了哪个选项而分类）应落在已定义的 store——**Transaction Log（完整 processing path / audit trail）+ Entity Log 创建 provenance + Case Log `confirmed_by`**，而非依赖 Intervention Log。Intervention Log 专责**交互过程痕迹供离线系统改进**。如此"审计性"（核心产品目标）不依赖一个尚未冻结 schema 的圈外 log。待拍板后回写决策 9。

## 仍未冻结 / 圈外

- Intervention Log 正式 spec：record schema、字段名、exact writer、retention。
- 与 Evidence Log / Review / Governance / Transaction Log 的边界。
- 离线改进流程**如何消费** Intervention Log（哪个系统设计 / 升级过程读取它、以何形态）。
- 以上待 Intervention Log 单独审计（L2·外阻 + L3 / L4）。
