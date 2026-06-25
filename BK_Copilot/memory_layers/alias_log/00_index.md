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

- `L2_proposals/Alias Log__L2提案.md` 是当前 Alias Log L2 回填的权威来源。
- `L2_proposals/Entity Resolution__L2提案.md`、`当前任务状态.md` 与 `缺口地图.md` 提供本轮已锁上游 / 不变量。
- `Alias question.md` 是 Alias 主题历史速记，不高于已审定 L2 提案。
- `Entity相关的所有问题.md` 与 `BK_Copilot/workflow_nodes/entity_resolution_node/` 提供 Entity Resolution 使用 Alias 的上下文边界。
- `BK_Copilot/memory_layers/entity_log/` 提供 stable entity authority（稳定实体权威）边界；Alias Log 与 Entity Log 的具体存储 / 投影关系尚未冻结。

## 当前标准文件

- `01_memory_intent.md`
- `02_authority_lifecycle_and_boundaries.md`

## 当前已冻结结论

- Alias 的核心作用，是辅助 Entity Resolution 判断当前交易的主体是谁。
- Alias 是过去已经确认过的 transaction surface text 和 stable entity 的对应关系。
- 写入资格已冻结：一笔交易一旦确认指向明确 stable entity，其 raw transaction description / descriptor / raw bank surface text 即具备进入 Alias Log 的资格；不要求该 surface text 曾是本次 stable identity 判断的决定性证据。
- 当前只确认两类信息可能成为 Alias：
  - bank statement 中每笔交易的 description / descriptor / raw bank surface text。
  - 当 bank description 本身没有明确身份意义时，其他可能重复出现并能指向交易主体的字段，例如 cheque payee。
- Alias 不是所有历史 description 的集合。
- 只有已经和某个明确 stable entity 建立过确认关系的 surface text，才属于 Alias。
- exact raw description match 可以支撑 Entity Resolution 直接复用 Alias 指向的 stable entity。
- Entity Resolution 是 Alias Log 唯一已冻结的直接 reader；Rule Match、Case Log、Case Judgment 或其他下游节点不得直接读取 Alias Log，只能通过 ER 输出的 stable entity 间接受益。
- 多匹配时 Alias Log 不选 winner，交给 Entity Resolution 结合当前 evidence 判断；ER 定不了则输出 `unknown + reason`。
- Entity merge 后，Alias 语义上跟随 merge 结果指向新的 stable entity；Entity split 后，原 Alias 不自动归属任一方，必须由治理 / correction 按新边界重判。
- 够格 Alias relationship 自动进入 durable Alias Log，无需额外 governance / accountant approval gate；stable entity 创建时同步写入已锁，写入执行者、确切顺序和 finalization mechanism 留到 L4 / seam。
- Alias 只提供 identity reuse（身份复用）能力，不提供 rule authority（规则权威）、会计分类结论或自动落账权限。

## 当前未冻结边界

- Alias 库具体技术形态、索引和存储实现尚未冻结。
- Alias Log 与 Entity Log 的具体存储 / 投影关系尚未冻结。
- `alias_record` 的 exact field schema（精确字段结构）尚未冻结。
- Alias 的写入执行者、确切写入顺序和统一 memory write / finalization 机制尚未冻结；写入资格、“无需额外审批”和“stable entity 创建时同步写入”已冻结。
- 受控 normalization / equivalence（标准化 / 等价匹配）规则尚未冻结。
- 高度类似历史 Alias 何时足以支持身份判断尚未冻结。
- historical Alias conflict 的 correction / governance path 尚未冻结。
- Entity split 后逐条 Alias 重新归属的执行机制、batch 操作、correction 审批和 audit trace 尚未冻结。
- Entity merge / split 时治理层如何同步更新 Alias Log 的执行机制尚未冻结。
- Alias mutation 与 Governance Log / audit trace 的边界尚未冻结。

## 进入下一阶段前必须解决

- 确定 `alias_record` 的 exact field schema。
- 确定 Alias 库技术形态、索引、存储实现，以及 Alias 与 Entity Log 的 storage / projection boundary（存储 / 投影边界）。
- 确定 Alias 写入执行者、确切写入顺序和统一 memory write / finalization 机制。
- 确定 Alias 查询、受控 normalization / equivalence 和非 exact matching 的最小语义规则。
- 确定 historical Alias conflict、split 后逐条重新归属、merge / split 同步更新和 Governance audit 边界的执行机制。
- 确定是否进入 M3 data contract；在字段、消费者和 mutation authority 稳定前不得创建 `03_data_contract.md`。
