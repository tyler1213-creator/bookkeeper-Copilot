# Alias question

## 当前确认结论

Alias 的核心作用，是辅助 `Entity Resolution` 判断当前交易的主体是谁。

在当前收窄后的定义下，Alias 是过去已经确认过的 transaction surface text 和 `stable entity` 的对应关系。

当前只确认两类信息可能成为 Alias：

1. bank statement 中每笔交易的 `description` / `descriptor` / raw bank surface text。
2. 当 bank description 本身没有明确身份意义时，其他可能重复出现并能指向交易主体的字段，例如 cheque payee。

Alias 不是所有历史 description 的集合。只有已经和某个明确 `stable entity` 建立过确认关系的 surface text，才属于 Alias。

## Entity Resolution 如何使用 Alias

`Entity Resolution` 使用 Alias 的典型场景有两种：

1. `Entity Resolution` 大致能判断当前交易主体，但不能完全确定时，可以以该 entity 为目标查询是否存在一致的 Alias。
2. `Entity Resolution` 完全无法从当前 evidence 识别主体时，可以用当前 description 或等价 surface text 到 Alias 库中查询是否存在一致或高度类似的历史 Alias。

如果当前 surface text 命中已确认的 Alias，可以复用该 Alias 指向的 `stable entity`。

一旦通过 Alias 或受控等价匹配确认了 entity，后续系统可以以该 entity 为主体查询 `Case Log` 等长期记忆。

## Alias 库

当前确认需要一个 Alias 库。

这里的 Alias 库指一个可被 `Entity Resolution` 查询的 Alias 集合，保存的是：

```text
已确认 Alias surface text -> stable entity
```

Alias 库的作用是让系统可以从当前交易的 surface text 反查过去已经确认过的 entity。

它的必要性在于：

- 当 `Entity Resolution` 已经有一个不完全确定的 entity 判断时，需要查询该 entity 是否有一致 Alias 来辅助确认。
- 当 `Entity Resolution` 完全识别不出主体时，需要在所有历史 Alias 中查询是否存在一致或高度类似的 surface text，从而反推出对应 `stable entity`。
- 没有 Alias 库时，系统无法稳定复用过去已经确认过的 transaction surface text，只能依赖当前 evidence 或重新询问 accountant。

Alias 库具体以什么技术形态呈现，暂不在本文讨论；可以在后续技术细节设计时再决定。

本文件暂不讨论 Alias 对 `Rule Match` 的支持。
