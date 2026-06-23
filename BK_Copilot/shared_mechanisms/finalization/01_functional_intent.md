# Finalization 写入机制 - Functional Intent

## 1. 为什么这个机制必须存在

这个机制解决的问题是：

> 给全系统一个唯一、确定性、可审计的 durable 写入地板，让所有长期写入通过同一套跨 log 原子 / 顺序 / 幂等 / append-only / 凭证校验不变量。（决策 1 / 2）

如果删除、合并或内联它，会失去：

- 跨 log 原子写入能力。一次纠错可能同时动 Transaction Log、Intervention Log、Case Log，必须一起成功或一起回滚。（决策 1）
- append-only、幂等、凭证强制等横切不变量的单点执行位置；散落到各节点后，迟早会在某个节点写歪。（决策 1）
- 一个足够小、确定、可单独审计的 trusted base；用中间写入 agent 或让节点裸写都会增加传递、token、幻觉和放行权错位风险。（决策 2）

这些能力对核心产品目标重要，因为：

- 审计性要求每笔长期写入可追溯、可回滚、可复核，而不是分散在各节点的隐式副作用里。
- `accountant control` 要求扩张型权威变更必须被会计师签字凭证挡住，系统不能自给凭证。
- 记忆复用纯净要求长期 memory 只接收符合 input / schema / proof 纪律的写入，避免模型输出幻觉直接污染 durable memory。

## 2. 核心职责

本机制的唯一核心职责是：

> 作为全系统 durable 写入的共享确定性库，直接执行四原子动作落盘，并对扩张型写入执行凭证强制闸门。（决策 1 / 3 / 4）

本机制的复合身份是：

- 共享确定性写入库：持有跨 log 原子、顺序、幂等、append-only、审计 trace 等写入不变量。（决策 1 / 2）
- 凭证强制闸门：扩张型写入缺会计师凭证即拒绝，凭证来源只能是会计师签字。（决策 4）

各节点 / 机制只供 payload，不负责怎么写、谁写、什么顺序写。本机制处在机制侧，负责怎么写 / 是否放行；它不重判该不该写或内容对不对。（决策 5）

## 3. 明确排除范围

本机制不负责：

- 语义判断、业务对错裁决、会计处理对错重判，或 proof 真伪判断；这些来自发起节点 / 会计师签字，本机制只做确定性 input / schema / proof 存在性校验。（决策 5）
- 按调用方身份做授权矩阵；调用方能对哪个 log 供什么 payload，归各 memory layer 的 writer 契约和上游正式草案。（决策 6）
- 铸造 ID 或返还 ID；ID 由调用方按 ID scheme 铸造并随 payload 带入，具体 scheme 留 L3。（决策 7）
- 把 merge / split / link 设为机制动作；merge / split 是调用方侧标准化动作批，link 是目标 log ID 字段直填。（决策 3 / 7；用户澄清 5）
- 替调用方做失败后的重试、补料、回会计师或运行期善后；本机制只回滚并上报结构化原因。（决策 8）
- 包含 LLM、生成候选、或把候选采纳为 authority。（决策 5 / 10）

本机制绝不能：

- 绕过凭证执行扩张型写入。
- 为了让写入继续而猜测缺失字段、补造 proof、改写 payload 语义或替调用方判断“这批 merge / split 是否语义完整”。
- 把 Governance Log trace 当成 entity / rule / case authority、accountant approval 或 governance approval。（决策 9）

## 4. 调用关系

本机制没有 runtime 流水线工位；它是被各调用方同步调用的共享库。节点模板中的“Workflow 位置 / 上游 / 下游”在本文改写为“调用方 -> payload / 写入目标 / 回执”。

| 调用方 | 传入 payload 语义 | 凭证要求 | 写入目标 | 回执 / 下游 |
| --- | --- | --- | --- | --- |
| Entity Resolution | `create` 新 stable entity；需满足目标动作校验和目标 log 强制语义 | 免凭证 | Entity Log / Alias Log（已落文，exact 顺序留 L4 / seam） | 成功 / 失败回执给 Entity Resolution 或其编排方 |
| Case Judgment | `append` High-Confidence 最终交易结果入 Transaction Log | 免凭证 | Transaction Log（已落文） | 成功 / 失败回执给 Case Judgment / 运行期编排 |
| Rule Match | `append` rule-hit 交易结果入 Transaction Log | 免凭证 | Transaction Log（已落文） | 成功 / 失败回执给 Rule Match / 运行期编排 |
| Coordinator | 发出 stable entity 出现信号，触发与 ER 共用的统一写入；只供身份确认后的 payload | 免凭证 | Entity Log / Alias Log；可能形成 Transaction / Case 相关 finalization 语义，exact 机制留 seam | 成功 / 失败回执给 Coordinator / 管道 |
| Human Review Node | 一批混合动作：`create` / `update` / `append` / `supersede`，用于纠错、扩张型 mutation、先例更新等 | 扩张型动作需会计师凭证；凭证挂整批 payload | Transaction / Entity / Alias / Case / Rule Log；扩张型耦合 Governance Log（待建）；Intervention Log（待建） | 成功 / 失败回执给 Human Review Node；扩张型审计写入给 Governance Log（待建） |
| 确定性发现 job（待建） | `append` 候选进审核 inbox | 免凭证；候选采纳为 rule mutation 时另走扩张型凭证 | 审核 inbox（待建） | 成功 / 失败回执给发现 job / 调度；候选由 Human Review pull |

本机制可直接写入的目标 log / store 范围是：Transaction Log、Entity Log、Alias Log、Case Log、Rule Log（已落文），以及 Governance Log、Intervention Log、审核 inbox（待建，本文只声明 open boundary，不替它们补内容）。

回执语义只锁定到 Stage 2：成功 / 失败 + 失败结构化原因；失败原因需说明哪个动作 / 哪条确定性校验失败。不返还 ID，ID 属调用方自有。（决策 7 / 8）

## 5. 对核心产品目标的贡献

本机制必须清楚支持以下至少一项：

- [x] 记忆复用
- [ ] 有证据支持的建议
- [x] accountant correction learning
- [x] 审计性
- [x] accountant control
- [ ] 自动化率提升

具体贡献：

- 消除散落写入漂移，把 append-only、幂等、原子、回滚和凭证校验放在一个确定性地板上。
- 通过扩张型写入必须带会计师签字凭证，保证长期权威变更不能由系统自发越权完成。
- 通过严格 input / schema / proof 闸，反向逼上游按本机制 input 规格产出，减少幻觉 payload 污染长期 memory 的机会。

## 6. 已知约束 / 不变量

- 节点不裸写 durable log，只供 payload；本机制是 durable 写入的唯一直接执行者。（决策 1）
- 凭证按写入类型分叉，不按调用方身份分叉；扩张型要凭证，候选 / 运行期 / 低半径写入按其写入类型免凭证。（决策 4）
- 凭证来源仅会计师；系统不能自给凭证。（决策 4）
- 放行权是确定性死代码，trusted base 要小；LLM 不参与凭证校验或写入放行。（决策 4 / 5）
- Transaction Log append-only；更正通过 append / supersede 等语义处理，不就地覆盖或删除原记录。（决策 3 / 已锁上游）
- 机制形态是共享库 + 单一事务边界；已否决中间写入 agent / 单一全局 LLM 写手。（决策 2）
- 校验挂在动作上，不挂在一个泛化 `write(完整记录)` 上。（决策 3）

## 7. 未决定问题

### L3

- 动作 payload schema、动作 exact 定义和动作类型 enum。
- 凭证 exact 形态、生成与传递机制、防重放和签字捕获方式。
- 回执 exact schema。
- ID scheme 和每动作 / 每 log 的 exact ID 形态。

### L4 / seam

- 多 log 顺序、原子、幂等键和失败回滚机制。
- 共享库 + 单一事务边界的具体实现。
- 写成功后的同步可见性保证。
- 重试 / 部分失败呈现。

### L2·外阻

- Governance Log 数据层和“记录哪些变动”的判定标准。
- 审核 inbox 数据层。
- Intervention Log 正式 spec。
- 确定性发现 job、JE Generation、Evidence Log 等待建或未对齐对象的正式边界。
- merge / split 标准化动作批模板与跨 log 迁移完整性契约。
- 第三类正式 spec 归属目录命名在 Stage 3 前的最终确认。
