# Finalization 写入机制 — L2 提案

> 状态：L1-L2 主体收口（G1–G8 原始八缺口 + 第一性原理 N1–N6 全部拍板）。Finalization 是 owner 已有设计结论的**第三类对象——跨层共享机制**（非 workflow node、非 memory layer），设计来源唯一为 `独立question文档/Finalization_question.md`（充当 owner 已拍板看法）+ 本轮对话（2026-06-23）；契约面从 A 类正式草案重建。
> 目标：集齐足以将 Finalization 从 L1 转成正式 spec 的信息。**模板归属另定**——节点模板的「触发条件 / runtime 位置」等字段不适用于"被调用的共享库"，转正式文档时需改写（见 L2·外阻）。
> 纪律：每条契约结构要么指回 A 类原文、要么指回用户拍板决定；`new system/` / `old_system_nodedesign/` 永不作依据。
> 待建正式文档目录（命名待最终确认）：`BK_Copilot/`（第三类对象，归属目录待定）。
> 工作区：`_audit_work/finalization/`（00 规则 / 01 契约面与缺口 / 02 热身与防污染）。

---

## 一、依赖（来自 `01` 契约面）

Finalization 是"地板"，**没有独立 00/01/02 正式文档**；它的契约面 = 每个已落文邻居在自己正式文档里把 writer / finalization **明文留空、统一指向本机制**的那些原句。

### 上游 / 调用方（供 payload 一侧，A 类已声明"不裸写、只供 payload"）
- **Entity Resolution**：`create` 新 stable entity（免凭证，需全 schema）。A：ER 02 §4「交给记忆 / finalization 层持久化」+ 02:301（写入执行者 / 顺序 / 多 log finalization open boundary）；ER 不直接写长期记忆（02:148）。
- **Case Judgment**：`append` High-Confidence 最终交易结果入 Transaction Log（免凭证）。A：CJ 02:92「应由后续 finalization / Transaction Log 机制按其资格保存」；CJ 不授权写入（02:93）。
- **Rule Match**：`append` rule-hit 交易入 Transaction Log（免凭证）。A：RuleMatch 02:60「应由后续 finalization 路径写入 TransactionLog」；runtime-only、不直接 durable write（01:113）。
- **Coordinator**：发"stable entity 出现"信号触发与 ER 共用的统一写入（身份确认免凭证）。A：Coordinator 02:61-64；绝不裸写任何 log（02:71-74）。
- **Human Review Node**：供"一串混合动作 + 凭证"（create + update + N×append + supersede），扩张型。A：HRN 01:27「备好符合各 log Finalization 要求的 input，交 Finalization 落盘；本节点自己不是 durable write 执行者」；02 §4/§5/§6（最完整的调用方规范）。
- **确定性发现 job**（未落文）：免凭证 `append` 候选进审核 inbox。

### 写入目标 log（A 类各 02 把 durable writer 明文指向本机制）
| Log | A 原文（writer 留空指向本机制） |
| --- | --- |
| **Transaction Log** | 02:34「统一 finalization 写入机制（exact writer 未冻结）」直 finalization、写入即定稿无 candidate 层；02:77 append-only；02:37/168 exact writer / 顺序 / 多 log 统一写入 = L4/seam |
| **Entity Log** | 02:39「direct stable entity creation via finalization / memory write mechanism（实际机制未冻结）」免 governance approval；02:113 confirmation 后由哪个 finalization 机制执行未冻结 |
| **Alias Log** | 02:40「待 L4/seam 冻结的 memory write / finalization 执行机制」够格即自动落库、免额外 approval |
| **Case Log** | 02:32「Case Memory Update Node / unified finalization mechanism（候选写入机制，未冻结）」；02:68 invalid/superseded 由 correction/finalization/governance 触发 |
| **Rule Log** | 02:64「写入执行者和写入顺序留 Governance/L4 seam」；02:40-41 approved/governance mutation 需 accountant 批准（=扩张型，要凭证） |

### 已锁事实（遵守，非审查；适用范围以对应正式文档为准）
- **全系统唯一落盘地板**：各 memory layer / 节点 02 的 writer 均指向它；它拥有跨 log 原子 / 幂等 / append-only / 凭证校验不变量；节点不裸写、只供 payload。A：缺口地图 §13。
- **凭证强制半边住在 Finalization**：原"授权确认"已分解，UX/确认/调度 → Human Review Node；凭证强制半边 → Finalization。A：缺口地图 §14；HRN 01:35。
- **凭证按写入类型开、不按谁发起开**：扩张型（rule 升/改/删/降、automation 放宽、merge/split、永久纠错）要凭证；低半径单笔纠错、restrictive auto-downgrade、Coordinator 运行中身份确认免闸。A：缺口地图 §14。
- **放行权 = Finalization 死代码**：缺凭证拒绝扩张型写入；哪怕 Human Review 编排出错也越不过。A：HRN 02:80-83「Finalization 死代码可以决定」。
- **Transaction Log append-only**：原记录永不就地覆盖/删除；更正 = 追加。A：Transaction Log 02:77。
- **writer 模型（owner）**：节点功能与 authority 不变，唯一变化 = 节点备料 → 调确定性 finalization 代码写入（旧"candidate→指定节点写入"模型已废）。

---

## 二、L2 决策

### 决策 1 — 身份定位：第三类跨层共享机制 = 共享写入库 + 凭证强制闸，全系统唯一落盘地板 〔指向各 log 02 writer 行、缺口地图 §13/14〕
**结论**：Finalization 是一套**共享的确定性写入库**，不是 node、不是 memory layer。任何 durable 写入的最后一脚都踩它；**没有任何节点自己直接往 log 写字节**。它两个身份合一：① 共享写入库（拥有跨 log 落盘的全部不变量：原子 / 顺序 / 幂等 / append-only / 审计 trace）；② 凭证强制闸门（无凭证拒绝扩张型写入）。各节点 / 机制只负责供"要写什么"（payload），不负责"怎么写、谁来写、什么顺序写"。
**为什么（锚定产品目标）**：服务「审计性」「记忆复用纯净」——散落写入一定漂移：一旦每个节点各写各的，append-only、跨 log 原子迟早在某节点被写歪。不变量长在库里、只此一处，才守得住。需要"一套共享库"而非"每 log 各写"是因为**存在跨 log 原子需求**（一次纠错同时动 Transaction + Intervention + Case Log，必须一起成功或一起回滚）。
**指回**：各 log 02 writer 均指向"统一 finalization 写入机制"；缺口地图 §13；用户 1c（Finalization 文档第一/二/三节）。
**排除的替代**：① 节点裸写 log（新设计明确否决）；② 按调用方拆"运行期机制 / 人发起机制"两块地板（每块都得重实现 create/update、制造漂移、还错放凭证闸——见决策 3）。

### 决策 2 — 实现形态：共享库 + 单一事务边界，否决"中间加一个写入 agent" 〔指向 缺口地图 §224、Finalization 文档 §三〕
**结论**：本机制是**一套共享代码库 + 单一事务边界**，**不是**一个居中的写入 agent。每个写入动作的 input 由各自节点备好，执行写入的是确定性代码；跨 log 原子由"同一事务边界"保证。
**为什么**：在节点与 log 之间多塞一个 agent 没有额外帮助——本质只是把信息再传一遍，既耗 token 又可能引入幻觉，还把"放行权"放到一个会出错的 LLM 旁边。共享库 + 单一事务边界用最小可信基达到同样的原子保证。
**指回**：用户 2026-06-23 拍板（G4）；缺口地图 §224「用户设想：所有 log 由统一写入；机制未冻结」；Finalization 文档 §三括注「『单一全局写手』vs『共享库 + 单一事务边界』是 L4 实现选择，别现在锁死」。
**排除的替代**：单一全局写手 agent（多一跳传递 + token + 幻觉 + 放行权错位）。
**open boundary**："共享库 + 单一事务边界"的 exact 实现机制 → L4。

### 决策 3 — 接口形态：一组有类型的确定性动作（create / update / append / supersede），校验挂在动作上 〔指向 Finalization 文档 §四、各 log 02 写入类型〕
**结论**：地板不是一个 `write(完整记录)`，而是一**小套有类型的确定性动作**，每个动作自带校验规则——**校验挂在动作上，不挂在地板上**。最小动作集 = 四个**原子动作**：

| 动作 | 校验语义 | 产 ID？ | 典型用途 |
| --- | --- | --- | --- |
| `create`（建新记录） | 必须给全强制 schema | **产新 ID**（新记录） | ER 建 stable entity；会计师直接建一条新 rule |
| `update`（改字段） | 只需 record_ref + patch + 改后仍满足不变量 | 引用现有 record_ref | 会计师改 entity 一维；Case Log「记录即发生」就地更新 |
| `append`（追加） | append-only，绝不就地改 | **产新 ID**（新记录） | Transaction Log 落盘 / 更正记录 / N 笔回溯 |
| `supersede`（标失效） | 按 `confirmed_by` 过滤、不删除、留血缘 | 引用现有 record_ref | 旧先例失效 |

- **merge / split / link 不设为地板动作**：`link` 已否决——每个 schema 自带 link 字段，直接填目标 log ID 即可，不需专门动作。`merge` / `split` 已否决设为地板原语——一个 merge 本质 = `create` 新记录 + `supersede` 两条旧记录 +（重指名下记录的）`update`，四原子动作可完整拼出；把 merge 做成地板原语会逼地板"理解合并跨 log 牵连什么"= 语义知识塞进死代码，破坏纯净、放大可信基。merge/split 是**调用方侧的标准化动作批**（见决策 7 责任归属）。
**为什么**：若地板只有一个"必须一次给全所有值"的写调用，"只改一个字段"就永远过不去——地板就死了。把完整性校验挂到 `create` 动作上、而非地板上，伪两难就解开了。动作类型与"谁发起"正交（运行期也 `update`、Human Review 也 `create`），所以**不能按调用方拆两套机制**——正确切法是"一块地板 + 一组有类型动作"。
**指回**：Finalization 文档 §四；各 log 02 写入类型（Transaction「direct finalization」/「correction append」、Rule「approved/governance mutation」、Case「direct case write」）；用户 2026-06-23 拍板（G1 + link/merge/split 否决）。
**排除的替代**：① 单个 `write(完整记录)`（被"完整 schema 必填"卡死）；② 把 merge/split/link 设为地板原语（语义入死代码、破纯净）；③ 按"运行期 / 人发起"拆两块地板（重复实现不变量、制造漂移、错放凭证闸）。
**open boundary**：动作集 exact 定义、各动作 payload exact field schema、动作类型 enum → L3。

### 决策 4 — 凭证闸门：扩张型才要、来源仅会计师、批粒度、唯一住地板的差异 〔指向 HRN 02:80-83、缺口地图 §14〕
**结论**：
- **闸门规则**：Finalization 只在拿到凭证时才放行**扩张型写入**；凭证只在有一条"已记录的会计师签字"后才存在。这是地板里的一段**死代码**。
- **凭证来源 = 仅会计师**（会计师发出或同意）。**系统无法自给凭证**——哪怕系统自己发现的扩张型变更，也必须绕到会计师签字才拿得到凭证。
- **凭证按写入类型开、不按谁发起开**：扩张型 → 要凭证；日常 / 运行期 / 候选写入 → 免凭证。证据：Human Review 内部同时有要凭证（扩张型）和免凭证（低半径单笔纠错）两种写入。
- **批粒度**：凭证挂在**整批 payload** 上；一张凭证 cover 一批混合动作，批内扩张型动作受其管、免凭证动作随批通过。
- **凭证是唯一住在地板里的差异**；其余差异（payload 多寡、失败上报对象、是否候选）都住在调用方 / 目标数据层。
**为什么**：**trusted base 要小**。把"放行扩张型变更"做成 Finalization 检查凭证的死代码，闸门既小又可单独审计——哪怕 Human Review（复杂 LLM 节点）出 bug / 被注入，没签字 = 没凭证 = 拒绝放行，什么 durable 改动都不会发生。一句话：UX 折进 Human Review，强制力留在 Finalization。
**指回**：HRN 02:80-83「Finalization 死代码可以决定 = 凭证校验 = 放行权」；缺口地图 §14；用户 2026-06-23 拍板（G2 凭证来源 + N6 批粒度）。
**排除的替代**：① 把放行权放进 Human Review（trusted base 变大、易被注入）；② 凭证按"是不是人发起"开（错切——Human Review 内部就有免凭证写入）；③ 系统自给凭证（绕过人类签字这条唯一治理线）。
**open boundary**：凭证 exact 形态（布尔开关 vs 带签字 ref / 范围）、防重放、与"会计师签字"的对应捕获方式 → L3。

### 决策 5 — 地板纯净：确定性 input/schema/proof 校验，非语义判断；兼作全系统抗幻觉 input 闸 〔指向 Finalization 文档 §七、CJ 02:92 / RuleMatch 02:60〕
**结论**：地板是确定性死代码，**只决定"怎么写"，不重判"该不该写、内容对不对"**——不含 LLM、不做语义判断。多个上游 02 说"有效性来自 finalization 对本次交易完成状态的认定"（RuleMatch 02:60、CJ 02:92），其**真义 = 地板对 input 有固定且严格的要求，信息不满足要求就拒绝写入**（如 `create` 必须给全 schema、Case Log 写入必须带 finalization proof）——这是**确定性的 input/schema/proof 校验，不是语义裁决**。地板检查 proof **是否存在**，不判 proof 的真伪。
- **连带性质（被低估的第二身份）**：这道严格 input 闸**反向逼上游 output 必须按 Finalization 的 input 规格产出**，从而在落盘那一刻**变相挡住模型幻觉**——它是全系统的"输入纪律闸 / 抗幻觉防火墙"。
**为什么**：守"运行 / 记忆"seam——判断（该不该写、内容对不对）属运行节点 / 会计师签字，落盘（怎么写）属机制；把语义塞进地板会破坏其可单独审计的死代码性质。格式合法但内容错的权威账目是记账系统最致命失败模式，确定性 input 闸是堵格式幻觉的关键一环。
**指回**：Finalization 文档 §七；RuleMatch 02:60、CJ 02:92（"finalization 对完成状态的认定"）；用户 2026-06-23 拍板（G6 澄清）。
**排除的替代**：地板做语义判断 / 重判业务对错（破纯净、破 seam、变大可信基）。

### 决策 6 — 授权分工：地板不认调用方身份；免凭证越权防护落在 writer 契约 + "扩张动作一律要凭证" 〔指向各 log 02 Writer 表、缺口地图 §14〕
**结论**：地板**不按调用方身份做授权**——它只是被调用、被写入的确定性代码，只做两件校验：① action 级校验（决策 3/5）；② 凭证校验（决策 4，仅扩张型）。免凭证写入（create / append）的越权防护**不在地板**，而落在两层：**① 各 memory layer 的 Writer 契约**（每个 02 的 Writer 表规定"谁能对本 log 写什么"）+ **② 扩张型动作一律要凭证**。
**为什么**：保持地板薄、确定、可审计——把"谁能写什么"的授权语义留在各 log 的 writer 契约里（那里本就是 authority 归属的权威源），地板不重复一套。真正高危的扩张型动作已被凭证闸兜住；免凭证动作的边界由 writer 契约约束。
**指回**：各 memory layer 02 Writer 表（如 Rule Log 02:38 Writer 表、Entity Log 02:32、Alias Log 02:38、Case Log 02:30）；用户 2026-06-23 拍板（N5）。
**排除的替代**：地板内建"调用方→log→动作"授权矩阵（与各 log writer 契约重复、变大可信基、违反地板纯净）。

### 决策 7 — ID 铸造归调用方；产 ID 规则按"动作是否产生新记录"；merge/split 批完整性归调用方 〔指向 Finalization 文档 §四、Entity Log 02:116〕
**结论**：
- **ID 由调用方铸造**、随 payload 带进；地板不铸造、不返还 ID（地板不认身份，照写即可）。系统涉及的各类 ID（entity_id、transaction 记录 ref 等）已设计明确。
- **产 ID 规则按动作语义分布**：`create` / `append`（产生新记录）→ 产生新 ID；`update` / `supersede`（改现有记录）→ 不产生新 ID、只引用现有 `record_ref`。
- **唯一性须由 ID scheme 自身保证**（地板不协调），调用方铸造的 ID **可兼作幂等键**（同 ID 重复提交 → 地板去重）。
- **merge/split 的"批完整性"归调用方 / 治理侧**：地板只保证调用方给的整批原子执行 + 各动作校验；它**不负责判断"这批算不算一个完整的 merge"**。若调用方拼的 merge 批漏了某条记录重指，留下孤儿引用 = 调用方的责任，非地板。
**为什么**：地板纯净（决策 5）+ trusted base 小（决策 4）——caller 铸 ID 让地板保持哑、且 ID 天然兼幂等键，与跨 log 原子/幂等（L4）正好咬合。merge/split 的跨 log 迁移语义本就归 Entity Log merge 契约 + Governance（A 已标 open boundary），不该塞进地板。
**指回**：Finalization 文档 §四（动作语义）；Entity Log 02:116「merge/split 后跨 log 迁移、finalization、audit trace、回滚机制」open boundary；Rule Log merge/split 服从 Governance 裁定（02:101）；用户 2026-06-23 拍板（N2 + create/update 产 ID 推理）。
**排除的替代**：① 地板铸造 ID（与"地板不认身份、纯写入"冲突，且需地板协调）；② 把 merge/split 完整性校验放进地板（需地板理解合并语义、破纯净）。
**open boundary**：每动作每 log 的 exact ID scheme → L3；merge/split 标准化动作批模板 + 跨 log 迁移机制 → L2·外阻（Entity Log merge 契约 + Governance）。

### 决策 8 — 中途失败：地板回滚 + 如实上报（带结构化原因），善后归调用方 〔指向 Finalization 文档 §六、HRN 02 §9〕
**结论**：Finalization 落盘中途失败时，**地板对所有调用方一视同仁**：回滚该事务 + 如实上报失败，且**回执带结构化原因**（哪个动作 / 哪条校验失败）。**谁被告知、怎么处置 = 调用方的事，不在地板里**：
- 运行期写入失败 → 回管道 / Coordinator 的错误处置；
- Human Review 扩张型失败 → 按 Human Review 规则：能在不瞎编信息前提下自解则自解，否则回会计师（HRN 02 §9）。
- **地板回执形态**：写入成功 / 失败 + 失败时的结构化原因；不返还 ID（决策 7，ID 调用方自有）。
**为什么**：失败的"善后策略"是发起方的语义，不是落盘机制的语义。地板保持薄、确定、可审计；结构化失败原因是调用方决定"自解 vs 回会计师"的必要输入。
**指回**：Finalization 文档 §六；HRN 02 §9（冲突 / 失败处理）；用户 2026-06-23 拍板（G8 + N1 结构化回执）。
**排除的替代**：地板替调用方做善后 / 重试决策（把发起方语义塞进机制）；失败只返布尔（调用方无法决定善后）。
**open boundary**：失败回滚 exact 机制、重试 / 部分失败呈现 → L4。

### 决策 9 — Governance Log 耦合：扩张型落盘耦合同步写、确定性代码执行 〔指向 HRN 02 §5、缺口地图 §19/271〕
**结论**：经地板落盘的每一笔**扩张型变更**，耦合触发一条 **Governance Log**（扩张型变更审计账本，**待建新数据层**）的**同步**审计写入；写 Governance Log 的也是**一套确定性代码**。该耦合由 Human Review Node 等的扩张型改动落盘时触发（**注意：触发节点是 Human Review Node，原"Governance Review"独立节点已废除**——缺口地图 §17）。
- **本对象只声明耦合契约、不建 Governance Log**。Governance Log 的 schema、以及"何种变动才入账"（即记录触发轴是否=凭证轴，含 restrictive auto-downgrade 等免凭证变动是否也入账）归 **Governance Log 设计窗口**。
**为什么**：每笔权威变更要有审计账本支撑后续复核（审计性）；用确定性代码同步写、与扩张型落盘耦合，保证"改了就有审计、不会改了没记"。Governance Log 是审计账本、非决策者，不替代会计师签字（HRN 02 §5）。
**指回**：HRN 02 §5「Governance 必须批准」段 + §6 输出「扩张型变更审计 → Governance Log（待建）」；缺口地图 §19/§271；用户 2026-06-23 拍板（G5）。
**排除的替代**：审计写入异步 / 解耦（可能改了没记）；本对象自建 Governance Log（越界——它未落文，属另一窗口）。
**open boundary**：Governance Log schema + "记录哪些变动"判定标准 → L2·外阻（Governance Log 窗口）。

### 决策 10 — 定稿型与候选型一视同仁：候选性住目标数据层，不住地板 〔指向 Transaction Log 02:43、Finalization 文档 §二〕
**结论**：地板对**定稿型写入**（如 Transaction Log「写入即 finalization 定稿、无 candidate 层」）与**候选型写入**（确定性发现 job 把候选投进审核 inbox）**一视同仁、写法完全相同**——都是"把这批字节原子地写进目标"。两者差异不在"怎么写"，而在**写入后对系统的影响 / 威胁程度**：定稿型出错概率低、对未来影响小；候选型往往涉及规则改动。
- **"候选"语义（待审 / 待采纳 / 可丢弃）住在目标数据层（审核 inbox）和治理流程里，不住地板**。地板不需要"知道"这是候选还是定稿。
- **候选两段**：(a) 确定性发现 job 把候选**投进审核 inbox** = durable 写入但**免凭证**（候选只是"待人看的建议"）；(b) 候选**被采纳、真变成一条 rule 改动** = 扩张型、**要会计师凭证**。"人确认"卡在 (b)，不卡在 (a)。
**为什么**：保持地板纯净（决策 5）——"候选会不会生效"是 inbox / 治理语义，让地板去懂它就会往"做判断"滑。把候选性留在目标数据层，地板依旧只管原子写入。
**指回**：Transaction Log 02:43「写入即 finalization 定稿、不设先提候选待批准」；Finalization 文档 §二（候选投 inbox 也走这套库、免凭证）；缺口地图 §14（确定性发现免闸）/ §272（审核 inbox 待建）；用户 2026-06-23 拍板（G7）。
**排除的替代**：地板区分定稿 / 候选并对候选做特殊处理（需地板懂"候选"语义、破纯净）。
**open boundary**：审核 inbox 数据层（候选状态 / 去重 / 过期）→ L2·外阻。

---

## 三、分类备案

### L3（字段 / enum / schema，留 Stage 3）
- **动作集 exact 定义 + 各动作 payload exact field schema + 动作类型 enum**（L2 已锁=四原子动作；link/merge/split 已否决为地板动作）。
- **凭证 exact 形态 + 生成与传递机制**（L2 已锁=来源仅会计师、挂整批 payload；未定=布尔 vs 带签字 ref/范围、防重放、签字捕获方式）。
- **地板回执 exact schema**（L2 已锁=不返还 ID、回执=成功/失败 + 失败结构化原因）。
- **ID 铸造 scheme + 各动作产 ID 规则的 exact 形态**（L2 已锁=调用方铸造；create/append 产新 ID、update/supersede 引用现有 ref；唯一性由 scheme 保证、兼幂等键）。

### L4 / seam（机制，挂 open boundary）
- **多 log finalization 写入顺序、原子 / 回滚、幂等键**（Evidence / Transaction / Case / Rule / JE）。
- **"共享库 + 单一事务边界"的 exact 实现**（L2 已锁=否决单一全局写手 agent）。
- **写成功的同步可见性（read-after-write）保证机制**（语义=返回成功后记录立即可读）。
- **失败回滚 exact 机制 / 重试 / 部分失败呈现**（L2 已锁=回滚 + 上报、善后归调用方）。

### L2·外阻（圈外依赖，挂 open boundary，绝不拿 B 填）
- **Governance Log** 数据层 + "记录哪些变动"判定标准（**待建**；本对象只声明耦合契约）。
- **审核 inbox** 数据层（候选投递目标，**待建**；草案 `独立question文档/Review_Inbox_question.md`）。
- **Intervention Log** 正式 spec（Human Review correction 的 Intervention 记录 + ID 写入目标）。
- **merge / split 标准化动作批模板 + 跨 log 迁移完整性**（归 Entity Log merge/split 契约 + Governance）。
- **Finalization 正式 spec 的模板归属**（第三类跨层共享机制，模板需改写；"触发条件 / runtime 位置"等节点字段不适用）。
> 全部已写入 `缺口地图.md` 的「Finalization 写入机制」section。

---

## 四、自检与判定

对照模板 Stage 1-2 标尺（按"机制"语义改写节点字段）：
1. **为什么必须存在 / 删除失去什么** ✓（决1：唯一落盘地板，删则散落写入漂移、跨 log 原子无处保证）。
2. **唯一核心职责** ✓（决1：跨 log 确定性落盘 + 凭证强制；不判该不该写）。
3. **明确排除 / 越权禁止** ✓（决5：不做语义判断；决6：不认调用方身份；决3：不设 merge/split/link 动作）。
4. **触发 / 上游** ✓（依赖：被各节点 / HRN / 发现 job 调用；非 runtime 流水线位置——机制特性）。
5. **读取面** ✓（决5/N4：为校验可读目标 log 当前态，属确定性校验）。
6. **写入对象分类** ✓（依赖表：5 个已落文 log + 待建 Governance/Intervention/inbox）。
7. **直接 durable write** ✓（决3：四原子动作；它是唯一真正落盘者）。
8. **权限分布（代码 / LLM / 人 / 治理）** ✓（决4/5/6：地板=确定性死代码、不含 LLM；凭证来源=会计师；授权=writer 契约）。
9. **输出类别 + consumer 契约** ✓（决8：回执=成功/失败+结构化原因 → 调用方；决9：审计 → Governance Log）。
10. **失败 / 歧义 / 冲突处理** ✓（决8：回滚+上报、善后归调用方）。
11. **audit trace 边界** ✓（决9：扩张型耦合写 Governance Log）。
12. **守运行 / 记忆 seam** ✓（决5：只管怎么写、不重判该不该写）。
13. **哪些未冻结** ✓（第三节）。

- **能否判定 L1-L2 完成**：G1–G8 + N1–N6 全部收口；**L1-L2 主体已完整**。
- **能否进 Stage 3**：**否**。卡两类阻塞——① 圈外依赖落文（Governance Log / 审核 inbox / Intervention Log 必须新建；merge/split 迁移契约归 Entity Log/Governance）；② L3 字段定稿（动作 schema / 凭证形态 / 回执 schema / ID scheme）；③ 正式 spec 模板归属待定（第三类对象）。

---

## 五、本轮已收口
- **决策 1/2（G4）**：第三类跨层共享机制 = 共享库 + 单一事务边界；否决"中间加写入 agent"。
- **决策 3（G1 + link/merge/split 否决）**：四原子动作 create/update/append/supersede，校验挂动作上；merge/split = 调用方侧标准化动作批、非地板原语；link 直填目标 ID 字段。
- **决策 4（G2 + N6 + 已锁事实）**：凭证闸=扩张型才要、来源仅会计师、批粒度、唯一住地板的差异；放行权=死代码（trusted base 小）。
- **决策 5（G6）**：地板纯净=确定性 input/schema/proof 校验、非语义判断；兼作全系统抗幻觉 input 闸。
- **决策 6（N5）**：地板不认调用方身份；免凭证越权防护=各 log writer 契约 + 扩张动作一律要凭证。
- **决策 7（N2 + create/update 产 ID 推理）**：ID 调用方铸造；create/append 产新 ID、update/supersede 引用现有；唯一性由 scheme 保证、兼幂等键；merge/split 批完整性归调用方。
- **决策 8（G8 + N1）**：地板回滚 + 如实上报（结构化原因）、不返还 ID；善后归调用方。
- **决策 9（G5）**：扩张型落盘耦合同步写 Governance Log、确定性代码执行、由 Human Review Node 触发；"记哪些变动"归 Governance Log 窗口。
- **决策 10（G7）**：定稿 / 候选一视同仁；候选性住目标数据层；候选两段（投递免凭证 / 采纳需凭证）。
- **N3/N4 收口**：写成功同步可见（机制 L4）；地板为校验可读目标 log 当前态（确定性、不破纯净）。

## 六、第一性原理复查 —— 结论与剩余项
- **N1（回执）✓**：caller 铸 ID 使"地板返还 ID"自动消解；回执 = 成功 / 失败 + 失败结构化原因。
- **N2（ID 铸造）✓**：归调用方；唯一性由 scheme 保证、兼幂等键。
- **N3（同步可见）✓ → 机制 L4**。
- **N4（地板读权）✓**：为校验可读目标当前态，属确定性、不破纯净。
- **N5（授权分工）✓**：地板不认身份；越权防护在 writer 契约 + 扩张要凭证。
- **N6（凭证批粒度）✓**：凭证挂整批，批内扩张受管、免凭证随批。
- **第一性原理总评**：八缺口 + 六复查项厘清后，本机制为**最小可信基的纯写入地板**：四原子动作 + 一道凭证死代码 + 跨 log 单一事务边界，语义全部留在调用方 / 各 log writer 契约 / 治理流程。未发现需推翻的结构。剩余全部为 L3 字段 / L4 机制 / L2·外阻待建对象，不阻塞 L1-L2 判定。
