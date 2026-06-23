# 任务：把已审定的 Finalization 写入机制 L2 提案落地到 BK_Copilot 正式文档（建模式，仅此一个对象）

## 你的角色
你是实现者，不是审查者，也不是设计者。
唯一任务：把【已经过用户审定的】Finalization 写入机制 L2 提案 + 下方用户澄清，落到 BK_Copilot 的对应正式文档上。
不重新设计、不重新审计、不质疑已锁结论、不扩大范围、不碰其它对象。

—— 本次 = **建模式**：目标文件不存在，以 `BK_Copilot/node:layer spec template/workflow_node_spec_template_rules.md`（workflow node 模板）为**结构基底**，**按下方「对象类型声明」的四处改写**改造后，从零创建 `00_index` / `01_functional_intent` / `02_logic_and_boundaries`；只创建已成熟到值得写下来的文件（本轮只到 Stage 1-2，**不**创建 03 及以后），不为结构完整造空文档。

## 对象类型声明（重要——本次没有专用模板，按此在 node 模板上就地改写）
Finalization **既不是 workflow node、也不是 memory layer，而是第三类对象——跨层共享机制**；当前该类只有它一个成员，**不另立通用模板**，直接借 node 模板骨架并按以下四处改写落地（务必遵守，且在 `00_index` 显式声明本对象非 node 非 memory layer、采用 node 模板 + 这四处改写）：
- **改写① 调用关系替代流水线工位**：node 模板的「触发条件 / Workflow 位置 / 上游前置」改写为「调用入口 / 调用关系（调用方→payload、写入目标、回执）/ 调用方前置」——本机制无 runtime 流水线工位，是被同步调用的共享库。
- **改写② 写入对象反转**：node 模板里普通节点「直接 durable write = 无、交 finalization 层」，本机制相反——它是全系统 durable 写入的**唯一直接执行者**；§4 写入对象写"全部"，不写"交 finalization 层"。
- **改写③ 新增机制专属节**：在 02 增加 node 模板没有的三节——**动作集**（四原子动作 + 校验挂动作上 + 否决 merge/split/link）、**凭证闸门**（扩张型才放行 / 来源仅会计师 / 分叉轴 / 批粒度）、**回执·失败**（成功失败回执 + 善后归调用方）。
- **改写④ 反向 seam**：node 模板的运行/记忆 seam 子句（"节点只声明存什么、不定怎么写"）**整个倒过来**——本机制处机制侧，**只定"怎么写 / 是否放行"，绝不重判"该不该写 / 内容对不对"**（后者来自发起节点 / 会计师签字），只做确定性校验。

## 范围（严格）
本次允许创建的对象文件，仅限：
- `BK_Copilot/shared_mechanisms/finalization/00_index.md`
- `BK_Copilot/shared_mechanisms/finalization/01_functional_intent.md`
- `BK_Copilot/shared_mechanisms/finalization/02_logic_and_boundaries.md`

> 注：归属目录名 `shared_mechanisms/finalization/` 为建议命名；若与既有约定冲突，按「停止并问用户」处理，不要自行另立目录结构。

落地完成后，按本指令最后一节同步更新：
- `当前任务状态.md`
- `缺口地图.md`

除上述文件外，绝不触碰其它 workflow node、memory layer、未列出的治理文档、`new system/`、`old_system_nodedesign/`。
如果驱动提案里出现「改其它对象文件」的拟改条目（例如五个 log / Coordinator / Human Review 草案中旧 writer 模型的写回、Governance Log / 审核 inbox / Intervention Log 的建档），**不要执行**——那属于其它对象，由单独 prompt 处理；本 prompt 内对这些结论只在 Finalization 自己范围的文件里**作为契约 / open boundary 声明**。

## 必读（按此顺序，且只把它们当依据）
1. 主驱动：`L2_proposals/Finalization__L2提案.md` —— 本次创建的权威来源、最新唯一来源，逐条决策落地。
   （注：`独立question文档/Finalization_question.md` 是历史设计结论、B 类、**不具权威**，**不**作为契约来源，见用户澄清 ①。）
2. 结构基底模板（遵守其 00/01/02 结构、禁止写法、Stage 1-2 完成检查 22 问）：`BK_Copilot/node:layer spec template/workflow_node_spec_template_rules.md` —— 但**按上方「对象类型声明」四处改写改造后**使用（调用关系替代工位、写入对象反转、新增动作集/凭证闸门/回执·失败、反向 seam）。
3. 结构依据（建模式）：node 模板 00/01/02 章节结构 + 四处改写 + 本指令「具体改动清单」逐节口径（清单已自包含每节要写什么，不依赖任何专用模板文件）。
4. 锁定上游 / 不变量与契约面来源：`缺口地图.md`（顶部「跨层共享机制」+「Finalization 写入机制」section）、`当前任务状态.md`，以及被引用的 A 类正式草案——各调用方 02（`workflow_nodes/{entity_resolution_node,case_judgment_node,rule_match_node,coordinator_node,human_review_node}/02`）与各写入目标 02（`memory_layers/{transaction_log,entity_log,alias_log,case_log,rule_log}/02`）中把 writer/finalization 指向"统一写入机制"的原句。出现冲突时以这些已锁结论为准，按权威顺序解决，不以 new/old system 为准。
`new system/` 与 `old_system_nodedesign/` 只是审计对象、无权威，不得作依据或 baseline。

## 用户已审定的澄清（凌驾于现状文字与提案措辞之上；只存在于本指令，务必照此落地）
① **唯一来源 = 本 L2 提案**。凡把设计来源指向 `Finalization_question.md` 的措辞，一律改为指回本 L2 提案 + 用户拍板；正式文档不得出现对该 question 文件的悬空依赖。
② **第三类对象 + 无专用模板**：本类只此一个成员，**不另立通用模板**；用 node 模板 `workflow_node_spec_template_rules.md` 为基底 + 上方四处改写。目录 `shared_mechanisms/finalization/`。文档须显式声明"本对象非 node 非 memory layer，采用 node 模板 + 四处改写（调用关系 / 写入反转 / 动作集·凭证闸·回执 / 反向 seam），节点模板的触发/runtime 位置字段经改写适用"。
③ **待建依赖一律先挂 open boundary、照常建档**：Governance Log、审核 inbox、Intervention Log、确定性发现 job、JE Generation、Evidence Log、Case Memory Update Node、merge/split 跨 log 迁移契约——全部作为 open boundary 声明，**绝不拿任何材料替它们补内容**，但不因它们缺席而阻塞本机制 00/01/02 的创建。consumer 是待建对象时，照实写出名字 + 标注待建。
④ **反向 seam 必须体现**：本机制只管"怎么写 / 是否放行"，不重判"该不该写 / 内容对不对"；它的严格 input 闸**反向逼上游 output 按其 input 规格产出 = 全系统抗幻觉 input 闸**（决策 5），务必写出，但不得写成"它做语义判断"。
⑤ **merge/split/link 不设为本机制动作**：动作集恰为 `create`/`update`/`append`/`supersede` 四原子动作；merge/split = 调用方侧标准化动作批（由四动作组合）、link = 直填目标 log ID 字段；"批的语义完整性"归调用方/治理侧，非本机制（决策 3/7）。

## 不可违反的纪律
- **守反向 seam**：本机制是 durable 写入的直接执行者；写明它**不重判该不该写 / 内容对不对**（那来自发起节点 / 会计师签字），只做确定性校验（schema 完整 / proof 存在 / 凭证存在 / 不变量满足）。
- **不在 L2 冻结字段级 schema**：动作 payload exact field schema、动作类型 enum、凭证 exact 形态、回执 exact schema、ID scheme、幂等键、写入顺序，全部留 open boundary（L3 或 L4/seam），不写死。现状若写死字段名，下沉为"字段名留 L3"。
- **契约面 / Consumer**：02 的契约面 / 回执表，每条回执 / 输出必须写明 Consumer（调用方 / Governance Log 等）与含义；consumer 是待建对象时照实写名 + 标注待建，不留空。
- **不触发模板禁止写法**（"LLM 自行判断""高置信度自动通过""后续实现再定""参考旧系统""写入日志但不说哪个 store/字段/authority""生成候选但不说谁消费/谁批准/能否持久化""本机制负责所有写入相关的一切""失败后自行处理"）。本机制**不含 LLM、不做语义判断**，务必体现。

## 具体改动清单（建模式：逐条指到 文件:章节 → 写什么；来源指回提案决策）

### `00_index.md`
- **文档状态**：Current standard status = draft；Covered stages 勾选 Stage 1 + Stage 2，其余不勾；Last reviewed = 2026-06-23；Owner/reviewer 留待填。显式标注"第三类对象·跨层共享机制；采用 node 模板 workflow_node_spec_template_rules.md + 四处改写，无专用模板"。
- **当前标准文件**：`01_functional_intent.md`、`02_logic_and_boundaries.md`（03 尚未创建）。
- **当前未冻结边界**：摘 `缺口地图.md`「Finalization 写入机制」section 的 L3 / L4-seam / L2·外阻要点。
- **进入 Stage 3 前必须解决**：① 圈外依赖落文（Governance Log / 审核 inbox / Intervention Log / merge·split 迁移契约）；② L3 字段定稿（动作 payload schema / 凭证形态 / 回执 schema / ID scheme）；③ 正式 spec 归属目录命名确认。

### `01_functional_intent.md`
- **§1 为什么必须存在**（决策 1/2）：全系统唯一落盘地板；删除 / 散落进各节点会失去——跨 log 原子、append-only、幂等、凭证强制等横切不变量将在某节点被写歪（散落写入必漂移）。需要"一套共享库"是因为存在**跨 log 原子需求**（一次纠错同动 Transaction + Intervention + Case，须一起成功 / 回滚）。
- **§2 核心职责**（决策 1）：复合身份——① 共享确定性写入库（持有跨 log 原子 / 顺序 / 幂等 / append-only / 审计 trace 不变量）；② 凭证强制闸门（无凭证拒绝扩张型写入）。各节点只供 payload，不负责怎么写 / 谁写 / 什么顺序。
- **§3 明确排除范围**（决策 5/6/3/7）：不做语义判断 / 不重判内容对错（决策 5）；不认调用方身份（决策 6）；不铸造 ID（归调用方，决策 7）；不设 merge/split/link 为动作（决策 3）；不替调用方善后（决策 8）；不含 LLM。
- **§4 调用关系**（依赖节）：调用方 → payload —— ER（create，免凭证）/ CJ·RuleMatch（append，免凭证）/ Coordinator（发信号触发，免凭证）/ Human Review Node（混合动作 + 凭证，扩张型）/ 确定性发现 job（append 候选进 inbox，免凭证，待建）。写入目标 = Transaction / Entity / Alias / Case / Rule Log（已落文）+ Governance Log / Intervention Log / 审核 inbox（待建）。回执 = 成功 / 失败 + 失败结构化原因（不返还 ID）。声明无 runtime 工位、为同步被调用的共享库。
- **§5 对核心产品目标贡献**：勾选 审计性、accountant control、记忆复用纯净（散落写入漂移被消除；扩张型须人类签字凭证）。
- **§6 已知约束 / 不变量**：节点不裸写、只供 payload；凭证按写入类型分叉、来源仅会计师；放行权 = 死代码（trusted base 小）；Transaction Log append-only；共享库 + 单一事务边界（否决单一 agent）。
- **§7 未决定问题**：动作 schema / 凭证形态 / 回执 schema / ID scheme（L3）；多 log 顺序原子幂等、同步可见性、失败回滚机制（L4）；Governance Log / inbox / Intervention Log 待建（L2·外阻）。

### `02_logic_and_boundaries.md`
- **§1 调用入口**（决策 1）：被各调用方同步调用以完成 durable 写入；不被用于——语义判断、内容对错裁决、绕过凭证的扩张型写入。声明无 runtime 流水线工位。
- **§2 调用方前置条件**（决策 3/4）：调用方须供符合目标动作校验的完整 payload（如 create 须给全强制 schema、Case 写入须带 finalization proof）；扩张型写入须附会计师凭证。前置缺失 → 拒绝写入 + 回结构化原因（决策 5/8）。
- **§3 动作集**（决策 3，机制专属）：四原子动作表——`create`（须全 schema，产新 ID）/ `update`（record_ref + patch + 改后满足不变量，不产 ID）/ `append`（append-only，产新 ID）/ `supersede`（按 confirmed_by 过滤、不删、留血缘，不产 ID）。**校验挂在动作上、不挂在机制上**。否决动作 + 理由：merge/split（= create + supersede + update 组合，设为原语会把跨 log 合并语义塞进死代码、破纯净）、link（直填目标 log ID 字段）。
- **§4 读取对象**（决策 5/N4）：仅为执行确定性校验读取目标记录当前态（如 update 查改后不变量、supersede 按 confirmed_by 过滤）——属确定性校验，非语义用途。
- **§5 写入对象（反转）**（决策 1/3/9）：直接 durable write = **全部**——本机制是全系统 durable 写入的唯一直接执行者，对 Transaction / Entity / Alias / Case / Rule Log 用四动作落盘。耦合写入 = 每笔扩张型变更耦合同步写 Governance Log 审计（确定性代码、由 Human Review Node 扩张改动触发；Governance Log 待建，决策 9）。绝不写 = 不绕过凭证写扩张型；不就地改 append-only 记录。
- **§6 凭证闸门**（决策 4，机制专属）：放行规则 = 仅在拿到凭证时放行扩张型写入（死代码闸）；凭证来源 = 仅会计师（系统无法自给）；分叉轴 = 按"写入类型（是否扩张型）"开、不按"谁发起"开；批粒度 = 挂整批 payload，批内扩张型受管、免凭证随批通过。**凭证是唯一住在机制里的差异**。
- **§7 决策权限**（决策 4/5/6/9）：Deterministic code 决定——动作校验 / 凭证校验 = 放行权 / 多 log 原子·顺序·幂等·append-only / 回滚。LLM = **无**（不含 LLM、不做语义判断）。Accountant = 凭证唯一来源（read-back 签字）。Governance = Governance Log 留痕（审计账本、非决策者）。
- **§8 契约面 / 回执**（决策 7/8）：回执表——成功（Consumer = 调用方；不返还 ID，ID 由调用方自铸）/ 失败（Consumer = 调用方；带结构化原因即哪个动作 / 哪条校验失败，供善后）/ 扩张型审计（Consumer = Governance Log，待建）。所有回执"不代表"本机制成为 authority——authority 来自发起节点 / 会计师签字。
- **§9 失败 / 异常处理**（决策 8）：中途失败 → 回滚该事务 + 如实上报（结构化原因），对所有调用方一视同仁；善后归调用方（运行期 → 管道 / Coordinator；扩张型 → Human Review 自解或回会计师）。本机制不替调用方决定重试 / 处置。
- **§10 守运行 / 记忆 seam（反向）**（决策 5）：本机制处机制侧——只决定怎么写 / 是否放行，不重判该不该写 / 内容对不对（来自发起节点 / 会计师签字），只做确定性校验。写明其严格 input 闸**反向逼上游 output 按 input 规格产出 = 全系统抗幻觉 input 闸**；但不得写成"它做语义判断"。
- **§11 Audit / Trace 边界**（决策 9）：触发 Governance Log 扩张型变更审计（耦合、确定性）；这些 trace 不能成为 entity / rule / case authority、accountant approval、governance approval（approval = 会计师签字本身）。
- **§12 Open Boundaries**：把 `缺口地图.md`「Finalization 写入机制」section 全部未冻结项逐条搬入并归类——L3：动作 payload schema / 动作 enum / 凭证形态 / 回执 schema / ID scheme；L4-seam：多 log 顺序·原子·回滚·幂等键 / 共享库单事务边界实现 / 同步可见性机制 / 失败回滚机制；L2·外阻：Governance Log（+ "记哪些变动"判定标准）/ 审核 inbox / Intervention Log / merge·split 标准化动作批模板 + 跨 log 迁移完整性 / 正式 spec 归属目录命名。写明这些解决前不得进 Stage 3 / Stage 4 / 实现。

—— 建模式专项：按模板补全 00/01/02 各节；对照模板「Stage 1-2 完成检查 14 问」逐条确保可回答，不能回答的标为 open boundary，**不要猜**。覆盖阶段保持 Stage 1-2，不勾 Stage 3 及以后。

## 停止并问用户的情形
- 归属目录 `shared_mechanisms/finalization/` 命名与既有约定冲突，无法确定。
- 任一回执的 consumer / 某结论的归属无法从提案或已锁上游确定。
- 提案某条拟改与已锁上游（`当前任务状态.md` / `缺口地图.md`）冲突，且权威顺序无法解决。
- 出现需要新的产品决策、而非措辞落地的点。
遇到以上情况，停下来问，不要自行拍板或猜测。

## 落地后更新相关规则文档（务必执行）
- `当前任务状态.md`（必更）：把 Finalization 的 L1-L2 标为**已落正式 draft**（建模式，仅 00/01/02，Stage 1-2，目录 `BK_Copilot/shared_mechanisms/finalization/`）；更新「最近重要交接」指针（从"设计结论、未转 BK_Copilot"改为"已落正式草案"）；保留必要交接与指针，不搬长期结论全文。
- `缺口地图.md`（必更）：「Finalization 写入机制」section 已收口的 L1-L2 不回写；保留 / 更新仍挂起的 L3 / L4-seam / L2·外阻；顶部「跨层共享机制」§13 的 Finalization 指针由"指向 `独立question文档/Finalization_question.md`"更新为"已落 `BK_Copilot/shared_mechanisms/finalization/`，question 文档降为历史背景"。
- 其它治理文档（`系统上下文地图.md` / `审计阶段路线图.md` / `AGENTS.md` / `审计目标与原则.md` / `项目背景速读.md`）：仅当其职责范围内内容确因本次落地变化时才动（如系统上下文地图需登记 Finalization 已落草案）；否则不碰。

更新纪律：保持克制、遵守权威顺序；规则文档不承载长期设计结论正文；不加产品功能逻辑细节；不把 new/old system 当 baseline 或权威。

## 完成后输出
- 建了哪些文件、每个文件写了哪些节、对应提案哪条决策或哪条澄清。
- 哪些点按要求保留为 open boundary（L3 / L4-seam / L2·外阻），没有越权冻结。
- 同步更新了哪些规则文档（当前任务状态 / 缺口地图 / 其它），各更新了什么；哪些规则文档判断无需改动及原因。
- 一致性自检：反向 seam 守住（只管怎么写 / 是否放行、不重判内容）、契约面与每个 Consumer 已声明（含待建对象照实写名 + 标注）、动作集恰为四原子动作（merge/split/link 未设为动作）、凭证闸门 / 回执 / 失败处理已写、无 LLM / 不做语义判断、未冻结字段级 schema、未触发禁止写法、覆盖阶段仍为 Stage 1-2。
- 是否触发「停止并问用户」；若有，列出问题。
