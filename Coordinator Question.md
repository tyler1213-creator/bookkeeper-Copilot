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

## 仍未冻结 / 未讨论

- Coordinator 完整 question 模板、回答结构和追问边界（= `未解决问题暂存清单.md` 问题 5，仍开放）。
- 同 batch 内重复 unknown entity 的去重提问（= 问题 6，暂缓）。
- 情况 Y 身份缺口待办由谁 triage / 复查机制（用户暂缓）。
