# Entity 相关 Node 和 Log 现存问题

## 文件角色

这是 Entity 相关 node / log 的现存问题速记。

它不是正式 spec，也不冻结任何 data contract。

本文只记录当前需要继续处理的边界问题，后续仍需回填到对应正式文档。

## 问题 A：Stable entity 判断与自动 stable 边界

已确认的核心标准：

> 当前证据能不能直接、清楚、可追溯地说明这个对象是谁？

如果能，并且没有明显身份冲突，就可以考虑 `stable entity`。

判断 stable 只看 identity，不需要证明：

- COA 是否唯一。
- HST / GST 是否可判断。
- 业务用途是否明确。
- 是否可以自动分类或自动落账。
- 是否已有 Rule，或是否可以升级 Rule。

明显身份冲突不能自动 stable。例如同一 surface text 可能对应多个 entity、Alias 关系冲突、描述过泛、内部转账关系不清、payroll / loan / tax / owner / employee / related party 身份关系未确认等。

已确认：`new_stable_entity` 本体可以由 Entity Resolution 同步写入 Entity Log。写入只限 entity 本体；Alias、rule、automation policy 不随本体写入，由后续流程处理。

仍需明确：哪些首次出现的对象可以自动 stable，清晰 bank descriptor 是否单独足够，以及明显身份冲突的操作边界。
