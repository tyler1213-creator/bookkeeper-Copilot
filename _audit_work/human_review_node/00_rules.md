# 00 · 规则蒸馏（Human Review Node 审计）

> 本文件是第 2 步「问题梳理」的标尺，也是 L2 Proposal 的自检表。逐字搬自 `workflow_node_spec_template_rules.md` §10 + `AGENTS.md` + `审计目标与原则.md`。

## A. 必守通用规则（逐条 + 出处）

1. **Source authority 铁律**（`AGENTS.md` §Source Authority）：用户明确指令最高；其余文档只在自身职责内有权威；`new system/`、`old_system_nodedesign/` 不提供任何权威参考。→ 本轮 owner 明确：Human Review Node 功能设计来源 = `Human_Review_Node_question.md` 本轮对话；运行层 log 以 `BK_Copilot/` 正式草案为准；`new system/` 不作参考。
2. **运行 / 记忆 seam**（template §2）：节点只声明「要持久化什么 + 谁有权威认定它有效」，**不**声明「怎么写、谁来写、什么顺序写」。后者属记忆层 spec / finalization 机制。
3. **契约面**（template §2 / §6）：节点只通过显式、最小、写明的输出契约面与下游交互；每个输出必须写明 consumer；字段级 schema 冻结属 Stage 3。
4. **不为对称 / 完整 / 未来可能性而存在**（template §2、`AGENTS.md` §审计方法）：只因架构完整性存在的对象不得进正式文档。
5. **第一性原理优先**（`审计目标与原则.md`）：每个 node/field/log 必须证明为何必须存在；删除/合并/内联不实质损害核心产品目标者，缺乏独立存在理由。
6. **核心产品目标（锚）**：帮会计师把历史账务与判断沉淀成可复用记忆 → 新交易获得有证据支持的建议 → 经纠正持续学习 → 不牺牲审计性与控制权地提高自动化率。六项贡献清单：记忆复用 / 有证据支持的建议 / 纠错学习 / 审计性 / 会计师控制权 / 自动化率。
7. **区分 evidence / memory / judgment / governance / audit**（`AGENTS.md`）；AI reasoning 文本不得变成可复用 authority。
8. **停止 / 升级**（`AGENTS.md` §停止）：需真实产品决策、两权威源冲突无法靠职责+顺序解决、会模糊被审计材料与正式草案边界时 → 停下问用户。

## B. Stage 1-2 完成检查清单（template §10，22 问 —— 标尺 + 自检表）

1. 节点为什么存在？ 2. 为什么不能删/并/内联？ 3. 唯一核心职责？ 4. 上游依赖什么？ 5. 下游影响什么？ 6. 读哪些 log/memory？ 7. 直接写什么？ 8. 只能提什么 candidate？ 9. 绝不能写什么？ 10. deterministic code 决定什么？ 11. LLM 判断什么？ 12. accountant 必须决定什么？ 13. governance 必须批准什么？ 14. 证据不足怎么处理？ 15. 歧义怎么处理？ 16. source conflict 怎么处理？ 17. 留哪些 trace？ 18. 哪些 trace 不能成 authority？ 19. 是否声明了对外契约面 + 每个输出的 consumer？ 20. 是否守住运行/记忆 seam？ 21. 哪些未冻结？ 22. 能否进 Stage 3？

## C. 禁止写法（template §9）

「协调一切相关逻辑」「必要时更新相关 memory」「LLM 根据上下文自行判断」「高置信度则自动通过」「后续实现时再决定」「参考旧系统处理」「写入日志但不说哪个 log/字段/authority」「生成候选但不说能否持久化/谁批准/谁消费」。
