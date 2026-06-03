# Rule Match 临时文档

## 文件角色

本文件记录当前 Rule Match / Rule Log 讨论中已经确认的临时结论。

后续正式重写 Rule Match Node 或 Rule Log memory layer 时，以本文作为本轮讨论结论入口。

## 当前结论

Rule 围绕可被确定性自动化处理的稳定重复交易模式建立。

Rule 的核心不是 Alias。

Alias 的核心作用仍然是辅助 Entity Resolution 判断当前交易主体。

Rule Match 接收 Entity Resolution 已经确认的 stable entity 结果，再判断当前交易是否满足可确定性自动化的 rule scope。

## Rule Scope

当前确认至少存在两类 rule scope。

### Entity-level Rule

Entity-level rule 围绕 stable entity 建立。

适用条件：

- 当前交易已经确认到 stable entity。
- 该 entity 本身足以稳定指向唯一会计分类结果。
- 该 entity 可以作为确定性自动化处理的核心对象。

如果一个 entity 下存在多种最终会计分类结果，该 entity 本身不应升级成 entity-level rule。

### Pattern-level Rule

Pattern-level rule 围绕 stable entity 下更窄的稳定交易模式建立。

适用条件：

- 当前交易已经确认到 stable entity。
- 当前交易需要比 entity-level 更窄的 scope 才能唯一决定会计处理。
- 当前交易满足该 pattern 的客观匹配条件。
- pattern 条件足以把当前交易和同一 entity 下其他会计处理结果区分开。

Pattern-level rule 当前确认的核心匹配条件包括：

- direction。
- amount range 或金额稳定性。
- 其他后续确认的客观条件。

Alias 的 runtime 位置是上游 Entity Resolution；Rule Match 接收已经确认的 stable entity 结果。

## 多分类 Entity

如果一个 entity 的历史交易存在多种最终会计分类结果，该 entity 进入更窄 scope 选择。

这种 entity 只能在更窄的稳定交易模式下建立 rule。

## Alias 在 Rule Match 中的位置

Rule Match 使用 Entity Resolution 输出的 stable entity 作为身份基础。

Alias lookup 发生在 Entity Resolution 阶段。

## 自动化原则

Entity-level rule 通过 stable entity 直接复用会计处理，避免 description 变化或新 Alias 阻断自动化。

Pattern-level rule 通过 direction、amount range 和其他后续确认的客观条件保持区分能力。

多分类 entity 通过更窄 scope 保持准确率。

Rule Match 的自动化能力来自 rule scope 和客观匹配条件。
