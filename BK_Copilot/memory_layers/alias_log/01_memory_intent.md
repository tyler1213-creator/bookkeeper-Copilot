# Alias Log - Memory Intent

## 1. 为什么这层 memory 必须存在

它解决的问题是：

> 为 Entity Resolution 保存过去已经确认过的 transaction surface text 到 stable entity 的对应关系，让系统能在未来交易中复用已确认身份，而不是每次只靠当前 evidence 重新判断。

如果删除它，会失去：

- Alias-based identity reuse（基于 Alias 的身份复用）：当前交易 surface text 命中过去已确认 Alias 时，系统无法反查对应 stable entity。
- fallback identity lookup（兜底身份反查）：当 Entity Resolution 完全无法从当前 evidence 识别主体时，无法用当前 raw description / surface text 查询历史已确认 Alias。
- accountant question reduction（减少会计师提问）：重复出现但表面文本不直观的交易更容易反复进入 pending。
- ER identity handoff（ER 身份交接入口）：系统无法先通过 Alias 帮助 Entity Resolution 找到 stable entity，再让下游通过 ER 输出的 stable entity 查询各自有 authority 的长期记忆。

这些能力不能由 runtime handoff 或现有 store 覆盖，因为：

- runtime handoff 只描述当前交易，不能保存未来可查询的 surface text -> stable entity 对应关系。
- Transaction Log 是最终审计记录，不参与 runtime identity decision。
- Case Log 保存完成案例学习记忆，不负责从交易表面文本反查 entity identity。
- Rule Log 保存已批准确定性规则，不回答当前交易主体是谁。
- Entity Log 保存 stable entity authority；Alias 与 Entity Log 的具体存储 / 投影关系尚未冻结，但 Alias 关系本身需要被清楚定义为可查询的 identity reuse memory。

## 2. 保存什么

这层 memory 保存：

- 已确认交易中的 raw transaction description / descriptor / raw bank surface text 到 stable entity 的对应关系。
- 只要该交易最终被确认指向明确 stable entity，该 raw surface text 就具备进入 Alias Log 的资格；不要求该 surface text 是本次 stable identity 判断的决定性证据。
- 可被 Entity Resolution 查询的 Alias 集合。
- 支持该对应关系可追溯的 evidence / confirmation trace；字段名和结构尚未冻结。

当前只确认两类信息可能成为 Alias：

- bank statement 中每笔交易的 description / descriptor / raw bank surface text。
- 当 bank description 本身没有明确身份意义时，其他可能重复出现并能指向交易主体的字段，例如 cheque payee。

其中可以成为 reusable authority（可复用权威）的是：

- 已确认 Alias surface text -> stable entity 的对应关系。

只作为 trace / context / explanation（追溯 / 上下文 / 解释）的是：

- 支持 Alias 关系的 evidence refs（证据引用）。
- Alias 被创建或确认的来源说明。
- Entity Resolution 查询 Alias 时的 lookup basis（查询依据）。
- 冲突、歧义或历史变更的说明文本。

## 3. 绝不保存什么

这层 memory 绝不保存：

- 所有历史 description 的集合。
- 未确认的 transaction surface text。
- 仅由 LLM 语义相似推断出的 surface text -> entity 关系。
- 旧 Alias 状态模型。
- COA account（会计科目）。
- HST / GST treatment（税务处理）。
- journal entry（分录）。
- rule condition（规则条件）。
- active rule payload（生效规则内容）。
- 面向 Rule Match、Case Judgment、Case Log 或其他下游节点的直接业务检索能力。
- final transaction outcome（交易最终结果）。
- raw evidence blob（原始证据正文）。
- unapproved AI reasoning（未经批准的 AI 推理）。
- 任何具体技术实现定义。

它也不能替代：

- `Entity Log`（实体日志）：stable entity identity（稳定实体身份）的 source of truth。
- `Evidence Log`（证据日志）：raw evidence（原始证据）和 evidence refs（证据引用）的 source of truth。
- `Case Log`（案例日志）：entity-indexed completed-case learning memory（按实体索引的已完成案例学习记忆）。
- `Rule Log`（规则日志）：approved deterministic rules（已批准确定性规则）的 source of truth。
- `Transaction Log`（交易日志）：final transaction audit record（最终交易审计记录）。
- `Governance Log`（治理日志）：高权限变化和审批历史；Alias mutation 与 Governance Log 的边界尚未冻结。

## 4. 对核心产品目标的贡献

这层 memory 必须清楚支持以下至少一项：

- [x] 记忆复用
- [x] 有证据支持的建议
- [x] accountant correction learning
- [x] 审计性
- [x] accountant control
- [x] 自动化率提升

具体贡献：

- 让过去已经确认过的 transaction surface text 可以在未来交易中复用到同一 stable entity。
- 让 Entity Resolution 在大致判断当前交易主体但不能完全确定时，可以查询该 entity 是否存在一致 Alias。
- 让 Entity Resolution 在完全无法从当前 evidence 识别主体时，可以用当前 surface text 查询历史已确认 Alias。
- 让 Entity Resolution 命中 Alias 后输出 stable entity，使下游可以通过该 stable entity 读取各自有 authority 的长期记忆。
- 通过只保存已确认对应关系，防止系统把所有历史 description、未确认相似文本或模型推理误当成身份权威。

## 5. 已知约束

- Alias Log 当前 L2 authority 来源是 `L2_proposals/Alias Log__L2提案.md`；`Alias question.md` 是历史主题速记。
- Alias 只辅助判断当前交易主体是谁。
- Alias 只保存已确认 transaction surface text 和 stable entity 的对应关系。
- Alias 不是 Rule Match 的规则库。
- Entity Resolution 是 Alias Log 唯一已冻结的直接 reader；其他节点只能通过 ER 输出的 stable entity 间接受益。
- Alias 不是所有历史 description 的垃圾桶。
- 未确认 surface text 不能作为 Alias 使用。
- 高度类似历史 Alias 不能天然等同于确认 identity；它最多增强 Entity Resolution 的判断，除非后续冻结受控 normalization / equivalence 规则。
- Alias 库具体技术形态不在当前阶段冻结。

## 6. 未决定问题

- Alias 的写入执行者、确切写入时机 / 顺序和具体 memory write / finalization 机制；倾向是在创建 stable entity 成立时同步写入 Alias，但需待 seam 阶段确认。
- Alias 库具体以什么技术形态呈现。
- Alias Log 与 Entity Log 是独立 store、嵌套记录、查询投影，还是其他形态。
- `alias_record` 的 exact field schema。
- 受控 normalization / equivalence 规则。
- 非 exact / 高度类似历史 Alias 何时可以支持更强 identity match。
- historical Alias conflict 的 correction / governance path。
- Entity split 后逐条 Alias 重新归属的执行机制、batch 操作和 audit trace。
- Alias mutation 与 Governance Log / audit trace 的边界。
