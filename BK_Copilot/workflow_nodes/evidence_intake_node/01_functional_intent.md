# Evidence Intake Node - Functional Intent

## 1. 为什么这个节点必须存在

这个节点解决的问题是：

> 把非结构化原始材料转化为可追溯保全的、可被下游确定性消费的结构化客观交易事实。

如果删除、合并或内联它，会失去：

- 把混乱原始材料变成可追溯、可被下游确定性消费的客观交易事实的统一入口。
- 交易进入下游前的放行资格闸门，无法保证只有干净、完整、可追溯的交易进入运行层。
- `transaction_id` 的统一铸号点，下游 ER / JE / Transaction Log / Case lineage 会失去同一笔资金事件的稳定锚点。

这些能力对核心产品目标重要，因为：

- 有证据支持的建议要求每个客观交易字段都能回到 raw/source material。
- 审计性要求 Evidence Intake 在交易进入身份、规则和分录流程前先建立可追溯 evidence foundation。
- 自动化率提升只有在下游收到完整交易、稳定交易锚点和清楚 evidence refs 时才安全。

## 2. 核心职责

本节点的唯一核心职责是：

> 把非结构化原始材料转化为“可追溯保全的、可被下游确定性消费的”结构化客观交易事实。

这项职责由四件不可分割的事组成：

- 可追溯证据保全：所有进入系统的原始材料都被纳入 evidence foundation，不丢弃材料。
- 最小客观交易结构投影：从原始材料中投影 date、amount_absolute、direction、source_account_ref、raw_description、currency 等客观事实语义。
- 放行资格闸门：只把满足 transaction-ready 资格且账单自洽的干净交易放入下游交易流；不达标材料保全后转证据池或纠错路。
- 第 9 步 `transaction_id` 铸号：只为本次运行已放行交易分配永久唯一 id，作为下游绑定同一资金事件的运行时锚点。

本节点可以辅助产生：

- counterparty surface signals，如 raw bank text / descriptor、receipt vendor、cheque payee、invoice / contract party、accountant context refs。
- 实际唯一覆盖信号与余额链 gap / overlap 信号。
- `EvidenceQualityIssueSignal` 与 human-readable issue summary。
- Evidence Intake 内部 evidence-to-transaction 配对结果；confirmed 的关联随 `EvidenceFoundationHandoff` 并入放行交易包，candidate / unassociated 证据转证据池。

但这些不是主职责，且不得把本节点退化为纯 parser、纯格式转换器或纯 evidence collector。

## 3. 明确排除范围

本节点不负责：

- 交易级去重或 same-transaction grouping。
- 跨导入 `transaction_id` 复用。
- durable transaction registry。
- entity / counterparty 身份判断。
- source account 到 Profile / COA 的映射。
- COA / HST / GST treatment 或 JE 构造。
- rule / case 判断。
- accountant-facing question 生成。
- durable business memory 写入。
- Evidence Log / Transaction Log 的写入执行者、写入顺序、finalization 或回滚机制。

本节点绝不能：

- 输出 stable / unknown 等身份状态；这些属于 Entity Resolution。
- 把 alias / entity / role candidate signal 升级成 approved alias、stable entity 或长期 role authority。
- 把 client / accountant note 当作 accountant approval、policy approval、会计结论或 rule authority。
- 把 evidence issue 改写成面向会计师的问题，或决定 pending / review / automation routing outcome。
- 在不读历史 registry 的当前架构中假装存在隐藏的 per-transaction dedupe 兜底。

## 4. Workflow 位置

上游：

- runtime 新交易原始材料：bank / payment account import line、bank statement、receipt、cheque、invoice、contract、client / accountant note 等。
- onboarding historical evidence。
- supplemental / re-intake evidence。
- 当前 batch intake context 与已存在 Evidence 引用；这些只读，用于 provenance 与避免重复保全。

三类来源共享同一 Evidence Intake 输出契约面，以 `source_channel` 标注 provenance。本节点不读取 Transaction Log 作为 runtime source。

下游：

- Profile / Structural Match（圈外对象，只挂起引用，不替它冻结 spec）。
- Entity Resolution。
- Rule Match。
- JE Generation（圈外对象，只挂起引用，不替它冻结 spec）。
- Transaction Log finalization（圈外对象，只挂起引用，不替它冻结 spec）。
- Coordinator / Pending、Review、interaction_agent、后续 audit / memory workflow（均为圈外 consumer，只挂起引用，不替它们冻结 spec）。

本节点位于流程中的原因：

- 下游不应各自重新解析原文，也不应在缺少 evidence foundation 与 objective transaction basis 时做 identity、rule、case 或 JE 判断。
- Evidence Intake 是所有交易进入系统运行前的必经入口；运行层顺序为 Profile / Structural Match -> Entity Resolution -> Rule Match。

## 5. 对核心产品目标的贡献

本节点必须清楚支持以下至少一项：

- [ ] 记忆复用
- [x] 有证据支持的建议
- [ ] accountant correction learning
- [x] 审计性
- [ ] accountant control
- [x] 自动化率提升

具体贡献：

- 有证据支持的建议：objective basis 与 evidence refs 一起输出，使下游 suggestion 能追溯到原始材料。
- 审计性：raw/source 是 source of truth，所有客观字段必须保留回源路径，AI / OCR / parser output 只作为 derived signal。
- 自动化率提升：本节点先挡住不完整、不可追溯或账单自洽性破裂的材料，减少下游处理带病交易的风险。

## 6. 已知约束

- raw/source material 是 source of truth；OCR、parser、LLM extraction 都是 derived signal，不得覆盖原文。
- 本节点不读 Transaction Log、Entity / Case / Rule / Governance memory 作为运行时判断来源。
- 交易级去重已经取消；当前只保留文件级 hash / content fingerprint 与账单/账期级余额链两道去重。若两道都漏过，重复交易会各自获得新的 `transaction_id` 并可能导致下游重复计数；这是已确认接受的产品载重假设。
- 字段级 schema、enum 取值、refs 形态、存储 / projection 形态、写入执行者和执行顺序均未在 Stage 1-2 冻结。

## 7. 未决定问题

- Evidence Intake 的未冻结 L3 / L4 / seam / L2 external blocker / DEFERRED 项唯一权威池是根目录 `缺口地图.md` 的 Evidence Intake section。
