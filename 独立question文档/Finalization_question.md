# Finalization Question（统一 finalization 写入机制）

## 文件角色

本文件记录 **Finalization 写入机制** 当前已确认的临时设计结论，供其他窗口理解这个机制现在如何处理。

它不是正式 spec，不冻结写入顺序、原子 / 幂等 / 回滚的 exact 机制、凭证形态或 data contract。后续仍需：① 送审计 → ② 写入 `L2_proposals/` 并审 L2 提案 → ③ 通过后才重写为正式 `BK_Copilot/` spec。

相关机制见 `Deterministic_Discovery_question.md`（发现候选）与 `Human_Review_Node_question.md`（人类确认面）。

---

## 一、一句话定位

**Finalization = 全系统唯一的落盘地板（一套共享的确定性库）。** 任何 durable 写入的最后一脚都踩它。新设计**明确否决了"节点裸写 log"**——没有任何节点自己直接往 log 写字节。

它两个身份合在一起：
1. **共享写入库**：拥有跨 log 落盘的全部不变量（原子 / 顺序 / 幂等 / append-only / 审计 trace）。
2. **强制执行闸门**：它吸收了原"授权确认机制"的强制执行半边——**无凭证就拒绝放行扩张型写入**。

## 二、它是地板，所有人都踩

凡是要持久化的东西，最后都经过它，包括：

- Entity Resolution 把解析出的身份写进 Entity Log；
- Case Judgment 的结果进 Transaction Log；
- Human Review Node 的纠错落盘、审批后的扩张型变更落盘；
- 确定性发现 job 往审核 inbox 投候选（durable 写入，也走这套库，但免凭证）。

各节点 / 机制只负责**供"要写什么"**（payload），不负责"怎么写、谁来写、什么顺序写"——那是 Finalization 的事。

## 三、不变量必须长在库里，不能各节点自己重写一套

这是最容易被误读、也最承重的一点。

- "各节点有自己的一套 finalization 逻辑"**只能**理解成"各节点带着自己的参数去调这套共享库"，**不能**是"各节点重实现自己的写入"。
- **多 log 原子 / 幂等 / append-only / 凭证校验等不变量，由库拥有。** 库拥有不变量，节点只供"要写什么"。
- 原因：散落写入一定会漂移。一旦每个节点各写各的，append-only、跨 log 原子这些保证迟早在某个节点被写歪——这正是新设计一路在躲的坑。

为什么需要"一套共享库"而不是每个 log 各写各的：**因为存在跨 log 的原子需求。** 一次纠错会同时动 Transaction Log（append 更正记录）+ Intervention Log（记为什么改、生成 ID）+ Case Log（标 superseded 旧先例），这几笔必须**一起成功或一起回滚**。只有共享的单一事务边界能保证这点。（"单一全局写手" vs "共享库 + 单一事务边界"是 L4 实现选择，别现在就锁死成"一个 agent"。）

## 四、凭证闸门：它吸收了授权确认的"强制执行半边"

原本设想的"授权确认机制"已被拆成两半（详见 `Human_Review_Node_question.md`）：

- **面向人的那一半**（提醒会计师、read-back、接签字）→ 并进 Human Review Node。
- **强制执行那一半（凭证校验）→ 留在 Finalization。** 就是这一节。

机制是：

1. **Finalization 只在拿到凭证时，才放行扩张型写入。**
2. **凭证只在有一条"已记录的会计师签字"后才存在。**
3. **凭证差异**：
   - **扩张型变更写入**（rule 升 / 改 / 删 / 降、automation 放宽、merge / split、永久纠错……）→ **要凭证**。
   - **日常 / 运行期 / 候选写入**（ER 记身份、候选投 inbox、restrictive auto-downgrade……）→ **免凭证**。

### 为什么"放行扩张型变更"这道闸门必须住在这里、不能住在 Human Review

**Trusted base 要小。** Human Review 是个复杂的、面向会计师的 LLM 节点。如果"让永久改动通过"的权力长在它身上，它一旦出 bug / 被注入，整个治理层完整性就破了。

把闸门做成"Finalization 检查凭证、无凭证就拒绝扩张型写入"这一段**死代码**，闸门就既小又可单独审计——**哪怕 Human Review 编排出错，也越不过它。** 没签字 = 没凭证 = Finalization 拒绝放行，什么 durable 改动都不会发生。

一句话分工：**UX 折进 Human Review，强制力留在 Finalization。**

## 五、它的作用

- 保证任何账目改动都是**原子、幂等、可审计、append-only（不可就地改）**。
- 作为**最后一道死代码闸门**，挡住一切没有人类签字凭证的扩张型变更。
- 经它落盘的每一笔扩张型变更，往 **Governance Log**（新建数据层，每笔权威变更的审计账本）记一笔审计。
- 把"怎么写"从所有节点身上收走，让节点只关心"要写什么"，避免散落写入漂移。

## 六、它在整条治理线里的位置

```
来源（三条之一）
  · ER / CJ 等运行期写入 ──────────────┐ 免凭证
  · 确定性发现 job 投候选进 inbox ──────┤ 免凭证
  · 会计师永久纠错 / 发现侧候选 ── Human Review ── 会计师签字 ─┐ 带凭证
                                                              ▼
                                                       Finalization
                                              （校验凭证 → 跨 log 原子落盘
                                                → 写 Governance Log 审计）
```

注意：扩张型那条要先经 Human Review 拿到凭证；运行期写入和候选写入直接调 Finalization、免凭证。**同一套库，差异只在凭证。**

## 七、未冻结（Open Boundaries）

- **多 log finalization 的 exact 机制**：写入顺序、原子 / 回滚、幂等键——实现级（L4），未冻结。
- **凭证的 exact 形态**：未冻结。
- **"单一全局写手" vs "共享库 + 单一事务边界"**：L4 实现选择，未定，别现在锁死。
- **Governance Log 的 schema** 与它如何被 Finalization 写入：Governance Log 当前不存在、必须新建，落文前未冻结。
- 以上均为**设计结论、非正式 spec**，勿当字段级契约。
