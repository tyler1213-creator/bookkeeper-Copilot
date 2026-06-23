# Deterministic Discovery Question（确定性发现）

## 文件角色

本文件记录 **确定性发现机制** 当前已确认的临时设计结论，供其他窗口理解这个机制现在如何处理。

它不是正式 spec，不冻结触发节奏、判据、字段或 data contract。后续仍需：① 送审计 → ② 写入 `L2_proposals/` 并审 L2 提案 → ③ 通过后才重写为正式 `BK_Copilot/` spec。

相关机制见 `Finalization_question.md`（落盘）与 `Human_Review_Node_question.md`（人类确认面）。

---

## 一、一句话定位

**确定性发现 = 一个挂在调度器上的自主笨 job。** 它读 log、套阈值判据、吐候选，**非 LLM**。它谁都不属于——不折进 Human Review，也不归任何节点拥有。

它只做一件事：在**没有人盯着**的时候，系统自发地把当前 Rule 侧已经定义好判据的**rule 升级候选**捞出来，投进一个审核队列，等会计师来看。它**只提候选，不落地、不审批**。

## 0. 2026-06-22 收窄后的当前职责

当前确定性发现 job **只服务 Rule Match / Rule 侧 promotion 发现**。因此当前审核 inbox 中由它产生的候选只有一类：

- `rule_promotion_candidate`：某个 stable entity 下某个客观 scope / pattern 已经具备足够 CaseLogEvidence，值得交会计师确认是否升级为 executable rule。

这条收窄覆盖此前文档里的泛化说法：

- automation drift / automation auto-downgrade **不再归当前确定性发现 job**。restrictive auto-downgrade 的检测、触发和可见性契约归 `Entity Log automation_policy` 的 maintenance / mutation contract，仍是 Entity Log 的 L4/seam open boundary；它可以未来复用调度器或判据执行基础设施，但不是本机制当前职责。
- 跨 rule 互斥 / 引用校验仍不归它；那是 Rule Log / Governance 在写入或生效前的同步守卫。
- 语义型 merge / split、系统自发发现错误仍挂起到“系统自发发现层是否必要”专题，不在本窗口处理。

## 二、为什么它必须独立、不能折进 Human Review

曾经有个想法：让 Review Agent 定期扫 log 找候选，把发现也一手包办。这条路是错的，原因是：

- **扫描本质是确定性代码，不是 LLM 的活。** 让 Review 那个 LLM 去扫 log，等于把一件便宜、确定、可审计的事，换成贵的、会幻觉的事——恰好抹掉了"确定性发现"存在的全部理由。
- **折进去什么都不赚，还搭上耦合。** 折叠只有两种实现：要么"会计师打开面板才扫"——发现节奏被 UI 活跃度绑架，该冒的升级候选迟迟不冒；要么"后台照样跑个 job"——那它根本就是这个独立 job 披了张 Review 的皮。前者错，后者只是改了个名。

所以正确形态是 **独立自主 job（生产者）+ Human Review 消费（消费者）**，两者通过一个持久队列解耦。Review 顶多在面板上放个"手动 rescan"按钮，但那个按钮也只是去**调**这个 job，不是 Review 自己扫；基线永远是自主调度。

## 三、它怎么运作

### 1. 触发分两种节奏

- **批后增量**：每批交易 finalize / correction 后，只对这批动过的 entity、case pattern、rule refs 重算 Rule 侧 promotion 判据。便宜、新鲜——抓"刚刚新增了一个可用 CaseLogEvidence / correction signal"。
- **定时全扫**：重算、补漏、backfill，以及抓 Rule 侧未来明确纳入的 clock-based predicate（例如 rule staleness）。这类信号的本质是**时间流逝 + 无活动**，所以**必须挂时钟**，批后触发天然抓不到。

MVP 可以用一个每晚的 job 把两件事一起干，但心里要清楚是两种诉求。

### 2. 触发权归调度器，不归任何 LLM 节点

cron / 编排层触发。发现的节奏是系统事（每晚 / 每次对账后），不该被 UI 活跃度绑架——否则数据新鲜度被会计师"哪天打开面板"决定。

### 3. 它是个"多判据的笨执行器",自己不拥有任何判据

每一类候选的"够格"判据，归它所治理的那个记忆层。当前只有 Rule 侧 promotion 判据：

- promotion（够不够格交给会计师看、是否可作为 rule 升级候选）判据 → 归 **Rule Log / Rule Match 治理**。

job 只是把这些判据**套用**一遍。判据变了，改的是各记忆层的治理，不是动 job。这样 job 永远薄、永远可单独审计。**别把判据逻辑也塞进 job 或 Review。**

Rule 侧判据必须和 Rule Log 的 authority-content 标准分层对齐：

- Scope 可客观判定、历史 treatment 唯一无分叉、输出 judgment-free 完整、事前 accountant approval，是 executable rule authority 的硬标准。
- N 次、跨月、频率、跨 case 模式、近期是否被推翻等，只能是“是否值得提出候选”的证据强度或反证信号，不能替代 accountant sign-off，也不能直接创建 executable rule。
- job 不拥有阈值数字、分布算法、结果唯一性判定或 CaseLogEvidence 打包规则；这些都归 Rule 侧定义。（注：不存在“rule 被推翻累计→降级候选”这类判句——rule 失效是语义判断、系统不自发产降级候选，降级只由会计师在 review 时人发起。）

### 4. 产出 = 候选，投进审核 inbox

job 把候选写进一个**持久的审核 inbox**（候选必须活到有人来看，不能是 runtime-only，否则没人在线就丢了）。

- 这个 inbox 是一个**新的数据层，且 ≠ Governance Log**——候选还没人批，不是已生效的治理事件，塞进 Governance Log 是污染。
- 一致性要点：候选写进 inbox 也是个 durable 写入，按"没有节点裸写 log"的铁律，它**也走 Finalization 那套库，但免凭证**（候选不是扩张型变更）。这正好对上 Finalization 的凭证差异：候选写入免凭证、扩张型变更落盘要凭证，同一套库、差异只在凭证。

### 5. 反馈是"拉"，不是"推"

**job 绝不调用 / 唤醒 Human Review。** 一旦 job 去 call Review，发现节奏又被 Review 的可用性绑架，绕回折叠的坑。正确解耦：job 只往 inbox 投，**Human Review 在会计师打开面板时主动来读**。生产者只管投递，消费者自己来取。

## 四、明确不归它管的

- **automation drift / restrictive auto-downgrade 不属于当前确定性发现职责。** 自动化策略收紧的检测、自动生效边界、治理可见性和 audit trace 归 Entity Log 的 automation_policy maintenance / mutation contract；未来若另设 Entity policy detector，必须独立落文，不回填进当前 rule promotion job。
- **跨 rule 互斥 / 引用校验不属于确定性发现。** 那是一道**写入 / 生效前的同步守卫**（命中冲突直接 fail-closed），长在写入闸口上，不是"事后发现"。别塞进这个 job。
- **语义型发现（系统自发 merge/split）不归它。** 那是**语义发现器，已挂起、必要性存疑**，单独另开窗口讨论，可能改为只在 Human Review 由人发起。

### 为什么确定性发现能放心自主，语义发现却要挂起

二者性质不同：

- **确定性发现**：依据是明确、可审计的阈值判据（够不够次数、结果是否唯一……）。系统自发跑没有"凭什么"的问题，且它只提候选、会计师仍把关。
- **语义发现**：依据是模糊语义判断，"系统自发凭什么"正是存疑点。

所以"自主"这顶帽子，戴在确定性发现头上合身，戴在语义发现头上才有问题。

## 五、它在整条治理线里的位置

```
确定性发现 job（自主、定时/批后、非 LLM）
        │  产出候选
        ▼
   审核 inbox（持久队列，≠ Governance Log）
        │  Human Review 主动来读（拉，不是推）
        ▼
Human Review Node（摆给会计师、read-back、会计师签字）
        │  带凭证
        ▼
Finalization（校验凭证、跨 log 落盘、写 Governance Log）
```

注意：这条链只是**发现侧候选**走的其中一条路，不是全系统唯一路径。会计师在 Human Review 当场发起的永久纠错**跳过发现 job**（人就是发现者）；ER 写 Entity Log 这类运行期写入**既不过发现、也不过闸**，直接走 Finalization。

## 六、MVP 搁置 / 未冻结（Open Boundaries）

- **inbox 候选的状态 / 去重 / 过期 —— MVP 不做、整体往后拖。** 注意：自主 job 每晚会重新发现同一条候选，真上生产时**去重 / 生命周期是第一个咬人的地方**（需要稳定候选键 + 幂等 + 压制已决项，读 inbox 里 deferred/rejected 的、读 Governance Log 里已决的，避免重复冒泡）。MVP 阶段不需要，先记着。已废的 Post-Batch Lint 设计里那套 `candidate_status`（deferred / rejected / superseded）直觉是对的——节点死了，但这个需求活下来，将来落到 job + inbox 身上。
- **触发节奏当前定为双节奏语义**：批后增量负责新 finalized case / correction 触发的 fresh promotion 发现；定时全扫负责重算、补漏、backfill，以及 Rule 侧未来明确纳入的 clock-based predicate。exact cadence、scope batching、run record 留 L4。
- **staleness 是否产生候选** 不由 job 决定。只有 Rule 侧把 staleness 定义为 Rule Match / Rule Log maintenance predicate 时，定时全扫才套用；当前 promotion-only MVP 不把 staleness 自行扩成候选类型。
- **Rule 侧判据的 exact 定义** 归 Rule Log / Rule Match 治理，与本机制对齐前未冻结。
- **rule 失效 / 降级不由确定性发现产候选（2026-06-23 收口）**：rule 由人创建确认、匹配出错概率低，"某条 rule 是否失效"是语义判断，系统自发判不出——只有会计师在 review 结果时才能发现，降级 / 废除由会计师当场人发起（走 Human Review 扩张型变更路径）。会计师纠错的事实（原交易曾由 active rule 命中、之后被 append 成不同结果）作为审计事实留在 Transaction Log correction append + rule-hit ref，但**不驱动任何系统自发的累积判定或降级候选**。确定性发现只产 rule 升级候选、不碰降级；原"'rule 被推翻一次'信号→累积/触发 inbox candidate"的设想已删除。
- **rule 升级固定执行路径（NEW-1）** 归 Rule Log / Rule Match 治理定义；job 只写候选，Human Review 只确认并触发，固定路径负责 authority-content 校验、overlap guard、凭证校验和 Rule Log / Governance Log 写入。
- 以上均为**设计结论、非正式 spec**，勿当字段级契约。
