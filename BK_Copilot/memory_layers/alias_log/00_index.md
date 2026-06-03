# Alias Log

## 文档状态

- Current standard status: draft
- Covered stages:
  - [x] M1: Memory Intent
  - [x] M2: Authority, Lifecycle, and Boundaries
  - [ ] M3: Data Contract
  - [ ] M4: Storage and Operations
  - [ ] M5: Tests and Fixtures
- Last reviewed: 2026-06-02
- Owner / reviewer: system audit session / user review required

## 一句话定义

`Alias Log`（别名日志）是 confirmed transaction surface text -> stable entity（已确认交易表面文本到稳定实体）的 identity reuse memory layer（身份复用记忆层）：它让 `Entity Resolution Node`（实体识别节点）可以从当前交易的 description / descriptor / raw bank surface text，或在特定场景下从 cheque payee 等可重复身份表面字段，反查过去已经确认过的 stable entity（稳定实体）。

它不保存所有历史 description，不保存 rule condition（规则条件），不保存 accounting classification（会计分类）、journal entry（分录）或 transaction audit trail（交易审计轨迹）。

## 核心来源

- `Alias question.md` 是当前 Alias 定义的核心来源。
- `Entity相关的所有问题.md` 与 `BK_Copilot/workflow_nodes/entity_resolution_node/` 提供 Entity Resolution 使用 Alias 的上下文边界。
- `BK_Copilot/memory_layers/entity_log/` 提供 stable entity authority（稳定实体权威）边界；Alias Log 与 Entity Log 的具体存储 / 投影关系尚未冻结。

## 当前标准文件

- `01_memory_intent.md`
- `02_authority_lifecycle_and_boundaries.md`

## 当前已冻结结论

- Alias 的核心作用，是辅助 Entity Resolution 判断当前交易的主体是谁。
- Alias 是过去已经确认过的 transaction surface text 和 stable entity 的对应关系。
- 当前只确认两类信息可能成为 Alias：
  - bank statement 中每笔交易的 description / descriptor / raw bank surface text。
  - 当 bank description 本身没有明确身份意义时，其他可能重复出现并能指向交易主体的字段，例如 cheque payee。
- Alias 不是所有历史 description 的集合。
- 只有已经和某个明确 stable entity 建立过确认关系的 surface text，才属于 Alias。
- 当前确认需要一个可被 Entity Resolution 查询的 Alias 库。
- Alias 库用于让系统从当前交易的 surface text 反查过去已经确认过的 stable entity。
- Alias 只提供 identity reuse（身份复用）能力，不提供 rule authority（规则权威）、会计分类结论或自动落账权限。

## 当前未冻结边界

- Alias 库具体技术形态尚未冻结。
- Alias Log 与 Entity Log 的具体存储 / 投影关系尚未冻结。
- `alias_record` 的 exact field schema（精确字段结构）尚未冻结。
- 创建 stable entity 时，当前 surface text 是否默认进入 Alias 库尚未冻结。
- Alias 的写入责任、审批路径和统一 memory write 机制尚未冻结。
- 受控 normalization / equivalence（标准化 / 等价匹配）规则尚未冻结。
- 高度类似历史 Alias 何时足以支持身份判断尚未冻结。
- Alias 冲突、多 entity 竞争、merge / split 后迁移或阻断规则尚未冻结。
- Alias 对 Rule Match 的支持暂不讨论，尚未冻结。

## 进入下一阶段前必须解决

- 确定 Alias 与 Entity Log 的 authority / projection boundary（权威 / 投影边界）。
- 确定 Alias 写入责任与 approval path（批准路径）。
- 确定 stable entity 创建时当前 surface text 是否进入 Alias 库。
- 确定 Alias 查询和受控等价匹配的最小语义规则。
- 确定是否进入 M3 data contract；在字段、消费者和 mutation authority 稳定前不得创建 `03_data_contract.md`。
