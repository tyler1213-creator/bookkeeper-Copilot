# PROJECT_BRIEF.md

## 文件职责

`PROJECT_BRIEF.md` 是新窗口的项目背景速读。

它只帮助 agent 快速知道这个项目大概是什么，不定义产品设计标准，不证明当前系统正确，也不把旧系统当答案。

## 项目是什么

这个项目是 AI Bookkeeping / AI Bookkeeper。

目标是辅助加拿大会计工作流，尤其围绕：

- bank transactions
- GST / HST
- double-entry bookkeeping
- CRA audit traceability
- accountant review / correction

## 核心背景

它不是一个简单的交易分类器。

系统需要从历史账务、原始证据、客户结构、entity、case memory、rules、accountant corrections 中建立客户级 bookkeeping memory。

当新交易进入时，系统应基于证据和历史上下文生成可审计的 bookkeeping suggestion，让 accountant 审核、批准或修正。

accountant 的修正结果再被结构化地反馈到长期记忆或治理流程中，避免下次重复犯同类错误。

## 使用边界

这份 brief 只提供项目背景。

具体节点、字段、流程、抽象是否合理，仍应在具体审计任务中基于 `new system_副本/` 的当前材料、用户问题和可见证据重新判断。
