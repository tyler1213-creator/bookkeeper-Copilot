# Rule Log - Authority, Lifecycle, and Boundaries

## 1. Source of Truth / Authority

Rule Log 是 executable rule authority 与 rule membership 的唯一真值。Rule Log 定义什么内容够格成为 executable rule authority，即 authority-content 标准，并校验已批准 provenance；该标准在 Governance / accountant 审批时应用与执行，Rule Log 不在写入时自行重跑每条 rule 的资格判断。

只有满足以下四条“全函数”判句并已获得 accountant 批准的内容，才能成为 Rule Log 中可执行 rule：

- Scope 可客观判定：仅凭当前交易客观事实和 stable entity 身份即可确定当前交易是否属于该 scope。
- 历史 treatment 唯一无分叉：在该 scope 下有效正面先例收敛到唯一 approved accounting treatment；若有正当分叉，必须收窄 scope 或不进入 Rule Log。
- 输出 judgment-free 完整：rule 命中后输出的 treatment 必须足够 JE Generator 结合当前交易事实直接执行，不再需要运行时会计判断。
- 事前已批准：该映射必须被 accountant / governance 批准后，才可成为 Rule Log authority。

次数、跨月、频率不是资格门槛，只是支持 promotion 判断的证据强度。CaseLog evidence 可以支撑系统发起 promotion request，但 repeated cases / CaseLog 趋势本身不能直接进入 Rule Log 成为 executable rule。

Rule Log 的执行面必须保证同一 entity 下的 executable rules 两两互不重叠。该互斥性由 RuleLog / Governance 在 rule 写入或生效前保证；RuleMatch runtime 不消解 overlap。

| Concept | Source of Truth | Authority Rule | Not Authority |
| --- | --- | --- | --- |
| Executable rule authority | Rule Log | 已满足 authority-content 标准、已获得 accountant 批准、当前处于可执行功能语义的 rule，才能被 RuleMatch 读取为 executable authority。 | rule candidate、promotion request、review-only guideline、accountant note、CaseLog 趋势、LLM reasoning、相似描述、模糊自然语言规则。 |
| Rule membership under stable entity | Rule Log | “该 `entity_id` 名下有哪些 rule”的成员关系单一归 Rule Log；RuleMatch 按 `entity_id` 查询当前可执行集合。 | EntityLog 中的 rule_id 列表；上游 handoff 预选具体 rule_id。 |
| Authority-content 标准 | Rule Log 语义；Governance / accountant 审批路径应用与执行 | Scope 可客观判定、treatment 唯一无分叉、输出 judgment-free 完整、事前 accountant 批准。 | RuleMatch runtime 重算资格；Rule Log 写入时重判内容对错；次数 / 跨月 / 频率作为资格门槛。 |
| 执行面互斥不变量 | RuleLog / Governance 生效前保证 | 同一 stable entity 下当前可执行 rule 集合必须两两不重叠；任一当前交易最多命中一条 executable rule。 | RuleMatch runtime 设 priority、用 LLM / confidence / fallback 选 winner。 |

## 2. 读取者

本表是本层对外声明的读取接口面：reader 只能依赖此处声明的内容，不得依赖本层内部实现。

| Reader | 读取目的 | 可读内容 | 限制 |
| --- | --- | --- | --- |
| RuleMatchNode 执行面 | 在已获得 stable `entity_id` 且上级 eligibility 放行（force_pending 未命中、未在上游 router 改道）后，确定当前交易是否命中已批准 rule。 | 按 `entity_id` 查询当前可执行 rule 集合、applicability 和 approved treatment。 | 不读取 paused / retired / replaced / deleted / candidate / 历史管理信息来自行判断哪些还能执行；不读取 CaseLog 重算资格；不做会计判断。 |
| Governance / Review / maintenance 管理面 | 审批、暂停、废除、恢复、替代、修复、复核冲突和维护 rule 集合。 | 更完整的 rule 状态、provenance refs、暂停 / 废除 / 替代语义、冲突上下文；外部 Governance / review context 中的 candidate 关联引用如需展示，只能作为外部管理上下文。 | 管理面信息不构成 RuleMatch runtime 选择依据；candidate 关联引用不表示 candidate 存在于 Rule Log；exact reader / assembler / projection / cache / permission 边界留 L4 / seam。 |

## 3. 写入者

本表描述本层的对外写入接口面与运行 / 记忆 seam。本轮只声明写入来源类别、写入类型和 approval 语义；exact 执行者、机制、顺序、approval capture 和 durable mutation path 留 L4 / Governance seam。

| Writer | 可以写什么 | 写入类型 | 需要 approval 吗 |
| --- | --- | --- | --- |
| 治理审批通过后的写入路径 | 已经通过治理 / 会计师审核批准的 executable rule，以及对当前执行状态的 approved mutation。系统建议路径必须附全部 CaseLogEvidence 交人类会计师审核。 | approved / governance mutation | 是。未经会计师明确批准的内容不得成为 executable rule。 |
| 会计师直接创建路径 | 会计师直接创建且本人明确批准的 executable rule；不要求先有 CaseLog 积累。 | approved / governance mutation | 是。会计师直接创建即其本人批准。 |

系统对 rule 的成立没有任何自动介入 / 自动创建能力。系统能做的只有提出哪些 entity 或哪些 pattern 可以升级为 rule 的 proposal / promotion request，并附上所有相关历史案例证据交人类会计师审核。未经会计师批准的内容不得成为 Rule Log 中的 executable authority。

## 4. Candidate 边界

以下内容可以作为 Rule Log 外部的 candidate / promotion 输入：

- CaseLogEvidence 支撑的 system-proposed promotion request。
- lint / review / governance process 生成的 rule promotion candidate。
- accountant 在直接创建前观察到的可执行 rule 设想。

> 关于 rule 升级（promotion）发现侧的扫描 / 候选产出机制，参见 `独立question文档/Deterministic_Discovery_question.md`（确定性发现机制当前唯一权威文档）；promotion 资格判句的定义权仍归本（Rule）侧，确定性发现只套用。

Candidate 不能直接成为：

- Rule Log executable rule authority。
- RuleMatch 可读取的当前执行面 rule。
- approved accounting treatment authority。

Candidate 不能以 candidate 身份进入 durable Rule Log state；外部 candidate 转化为 executable authority 的条件：

- Candidate 本身不进入 Rule Log；只有经过 accountant sign-off 后形成的 approved executable rule，才可以通过未冻结的 Governance / L4 写入机制进入 Rule Log。
- 系统发起 promotion request 前，必须附上所有相关历史案例证据 CaseLogEvidence；该证据用于向 governance / accountant 解释为什么认为该项满足治理准入，并直接交给人类会计师审核。
- Accountant 主动直接创建 rule 时，不要求 CaseLog 积累；会计师直接创建即其本人批准，但 rule 内容仍须满足 Rule Log 的 executable rule authority-content 标准。
- 审批 workflow、证据打包格式、approval capture、写入执行者和写入顺序留 Governance / L4 seam，本轮不冻结。

系统无自动创建 executable rule、自动升级 candidate 或自动写入 Rule Log 的能力。所有 rule 的成立，无论来自系统建议还是会计师直接创建，都必须有 accountant sign-off。

## 5. Lifecycle / States

本轮只冻结 lifecycle 功能语义，不冻结字段名、enum、跨 log lifecycle 命名、原地覆盖 vs 版本化 / 替代链、物理删除 vs 投影删除。本表中的 State 是功能语义标签，不是字段级 enum。

| State | 含义 | 谁可以进入 | 谁可以退出 | 下游含义 |
| --- | --- | --- | --- | --- |
| 可执行 | 该 rule 当前可被 RuleMatch 读取为 executable authority。 | 未冻结；只能经 approved / governance mutation 或会计师直接批准路径进入。 | 未冻结；暂停、废除、替代、删除、恢复等 mutation path 留 Governance / 记录层。 | RuleMatch 执行面可读取。 |
| 暂不可执行 / blocked | 该 rule 存在，但当前不得被 RuleMatch 执行。 | 未冻结；依赖 Governance / review / maintenance 或 accountant correction。 | 未冻结；恢复或退出执行面的 exact path 留 Governance / 记录层。 | 不得泄漏成 RuleMatch runtime 可选规则。 |
| 不再可执行 | 该 rule 已被废除、删除、替代、合并影响或治理裁定退出执行面。 | 未冻结；依赖 Governance / review / maintenance 或 accountant correction。 | 未冻结；恢复、迁移、替代链或回滚 path 留 Governance / 记录层。 | 不得被 RuleMatch 当作当前 executable rule。 |

## 6. Mutation Path

已冻结 mutation path：

- 无字段级或机制级 durable mutation path 在 M2 冻结。
- 本轮只锁定两项执行面结果约束：RuleMatch 读取时必须得到当前可执行 rule body；非当前执行面内容不得被 RuleMatch 当作可执行 rule。

未冻结 mutation path：

- rule 修改、原地覆盖、版本化、替代链、删除和恢复的具体 mutation path。
- system-proposed promotion request 的审批 workflow、CaseLogEvidence 打包格式和 approval capture。
- accountant direct rule creation 的 exact write mechanism、写入执行者和顺序。
- merge / split 后 rule 重判、迁移、阻断、恢复执行、回滚和治理 trace。
- 多 log finalization、Transaction Log / Case Log 的 rule-hit ref 写入顺序和记录机制。

例外：

- 无。

## 7. 与其他 memory/log 的边界

| Other Store | 已确认边界 | 未冻结边界 |
| --- | --- | --- |
| Entity Log | Entity Log 是 stable entity identity、entity_status 和 force_pending / promotion_lock 等 control state 的主档案；不保存 rule_id 列表、rule condition、approved treatment 或 rule lifecycle。EntityLog force_pending（经上游 eligibility router 改道）是 RuleMatch 的上级放行控制；RuleLog 中存在 executable rule 只是可执行内容条件。Entity merge / split 后，Rule Log 服从 Governance 裁定，旧 rule 不自动迁移、不自动继承；在治理明确保留、迁移、收窄、废除或恢复前，不应作为新 entity 边界下的 executable authority。 | force_pending / promotion_lock exact enum；RuleMatch eligibility projection / handoff exact contract；merge / split 后 rule 重判、批量 mutation、迁移 / 阻断 / 恢复执行、回滚和治理 trace。 |
| Case Log | Case Log 可以提供 promotion 依据和历史案例证据，但 repeated cases / CaseLog 趋势不能直接创建、升级、修改、删除或降级 executable rule。Rule-hit case 的 rule ref 可以由 Transaction Log / Case Log 逐笔记录派生统计。 | CaseLogEvidence 打包格式；case-derived rule / force_pending / promotion_lock governance contract；rule-hit ref 在 Case Log 中的 exact 字段形态；统计 / rollup / cache 机制。 |
| Transaction Log | Transaction Log 是最终交易审计 source of truth，适合保存某笔 finalized transaction 最终由哪条 rule 处理的 source 语义和 rule ref。Rule Log 不做历史解释。 | rule-hit source / rule ref exact 字段形态；finalization / record path 的写入顺序；历史修正与 rule ref 追溯机制。 |
| Governance Log | Governance Log 承载审批、暂停、废除、替代、恢复、merge / split 裁定等治理事件正文；Rule Log 只反映当前执行状态，不与治理裁定竞争。 | approval workflow、approval capture、candidate queue、direct accountant write、mutation path、跨 log lifecycle 命名统一。 |
| JE Generator | Rule Log 是 approved treatment authority；JE Generator 是 JE construction / validation layer。Rule Log 不生成 JE，JE Generator 不判断 rule applicability。 | approved treatment exact shape；COA / HST / GST / split / allocation schema；JE Generator contract 与 validation rule。 |

## 8. 冲突处理

如果 Rule Log 与 Entity Log force_pending / control state 冲突：

- authority 顺序：EntityLog force_pending 优先于 RuleLog executable rule：force_pending 命中即在上游 router 改道、压过名下 active rule。
- runtime 行为：RuleLog 有 executable rule 只是必要非充分条件；force_pending 命中（上游 eligibility router 改道）时，当前交易不得进入 RuleMatch。
- 是否阻断自动化：是。
- 是否生成 review / governance candidate：可生成，但 exact 机制留 Governance / L4 seam。

如果同一 stable entity 下当前交易命中两条或更多 executable rules：

- authority 顺序：不设 rule priority，不由 RuleMatch 选择 winner。
- runtime 行为：RuleMatch 只能 fail-closed blocked / review。
- 是否阻断自动化：是。
- 是否生成 review / governance candidate：可生成 overlap repair / governance candidate；exact repair path 留 L3 / L4 / Governance。

如果 Rule Log 与 Governance Log 冲突：

- authority 顺序：Governance 已批准 / 已生效事件优先；Rule Log 只反映当前执行状态，不与治理裁定竞争。
- runtime 行为：在 Rule Log 被修复前，不得把与已生效治理裁定冲突的内容当作 executable rule。
- 是否阻断自动化：是，直到当前执行面与治理裁定一致。
- 是否生成 review / governance candidate：是，具体机制留 Governance / L4 seam。

## 9. Audit / Trace 边界

这层 memory 必须留下的 trace：

- source / approval provenance refs，用于说明 rule authority 从何而来、由谁批准或经何种治理结果生效。
- 当前执行状态语义的来源 trace，用于支持 review、correction、governance 和 audit。

这些 trace 用于：

- review。
- correction。
- governance。
- audit。

这些 trace 不能成为：

- 交易审计记录。
- reusable accounting authority。
- accountant approval 本体。
- governance approval 本体。
- Transaction Log / Case Log / Governance Log 正文的替代品。

## Open Boundaries

以下问题解决前，不能进入 M3 data contract、M4 storage and operations 或 implementation：

- **L3**：rule record exact 字段名、`rule_id` 的 exact 字符串 / 格式、scope enum、current execution state 字段名 / enum、source / approval provenance refs 形态。（`rule_id` 的**存在与角色已定**，见 Decisions D9：稳定唯一、贯穿生命周期、作引用 / 审计 handle；仅 exact 格式留 M3。）
- **L3**：entity-level / pattern-level rule exact schema、condition enum、客观条件表达、catch-all pattern 语法糖。
- **L3 / L4**：overlap-validation 算法、condition satisfiability 检查、amount range validation、冲突 repair path。
- **L3 / JE Generator**：approved accounting treatment exact shape、COA / HST / GST / split / allocation schema、JE Generator contract 与 validation rule。
- **L3，EntityLog / RuleLog / Governance 联合**：force_pending 取值如何在上游 eligibility router 决定交易是否放行进入 rule-based automation / RuleMatch（命中即改道、不进 RuleMatch）。
- **L4 / seam**：RuleLog 执行面 query contract、reader / assembler 调用顺序、projection / cache / permission 边界。
- **L4 / seam**：RuleLog 按 `entity_id` 索引的存储结构、检索和维护机制。
- **L2·外阻，Governance / review path**：rule promotion 发现侧判句（含 CaseLogEvidence 资格、结果唯一性、证据强度；不承接 rule 失效 / 降级判断）、rule candidate queue / 审核 inbox、system-proposed promotion request、CaseLogEvidence 打包格式、accountant approval workflow。
  > 关于 rule 升级（promotion）发现侧的扫描 / 候选产出机制，参见 `独立question文档/Deterministic_Discovery_question.md`（确定性发现机制当前唯一权威文档）；promotion 资格判句的定义权仍归本（Rule）侧，确定性发现只套用。
- **L4 / Governance seam**：accountant direct rule creation 的 exact write mechanism、approval capture、写入执行者。
- **L2·外阻 / L4，Governance / 记录层**：rule modification / overwrite / versioning / supersession / deletion / restore exact mutation path。
- **L3 / Governance**：跨 log lifecycle 字段命名统一。
- **L3，联合 Transaction Log / Case Log**：rule-hit source / rule ref 在 Transaction Log / Case Log 中的 exact 字段形态。（**已定**，见 Decisions D9：rule-hit 引用 = `rule_id` 裸值、不钉版本；历史版本还原靠 `rule_id` + 交易时间 + Governance Log 回放。仅 exact 字段形态留 L3。）
- **L4 / seam**：match_count、last_hit、health / staleness 派生统计、rollup、cache、maintenance job。
- **L4 / Governance**：merge / split 后 rule 重判、批量 mutation、迁移 / 阻断 / 恢复执行、回滚和治理 trace。
