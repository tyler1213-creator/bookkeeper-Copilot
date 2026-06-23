# Deterministic Discovery Question（确定性发现）

## 文件角色

本文件是 **确定性发现机制当前唯一权威设计文档**（pending 未来正式 `BK_Copilot/` spec）。

它记录用户已审定的当前机制结论，供 Rule 侧、Human Review、Finalization / 审核 inbox 接缝引用。它尚未写入正式 `BK_Copilot/` spec，不表示任何 Stage / M 阶段完成；不冻结 candidate schema、审核 inbox 字段 / 状态、触发 cadence exact 值、ID / 幂等键或 Rule 侧 promotion 资格判句 exact 定义。

未来正式落文时，本机制应作为第三类对象（跨层共享机制）处理，与 Finalization 同类：既不是 workflow node，也不是 memory layer。

相关入口：

- `BK_Copilot/shared_mechanisms/finalization/`：durable 写入 / 凭证闸门。
- `独立question文档/Review_Inbox_question.md`：审核 inbox 草案。
- `BK_Copilot/workflow_nodes/human_review_node/`：人类确认与复核面。

---

## 一、一句话定位

**确定性发现 = 挂在调度器上的、独立的、通用的非 LLM 笨 job。** 它读 log、套 Rule 侧已经定义好的判句、吐候选，**只提候选，不落地、不审批**。

它属于第三类对象：跨层共享机制。它不折进 Human Review、不折进 Rule Match、不折进 Rule Log，也不归任何单一节点或记忆层拥有。

seam 理由是：promotion 资格判句归 Rule 侧治理定义；扫描、调度、重算、补漏和写入审核 inbox 是机制层管道，不该住进记忆层。即使今天只有一个客户，这条边界也成立。

## 0. 当前职责：只发现确定性可发现的扩张型变更

确定性发现机制只发现 **确定性可发现的扩张型变更**。

今天唯一客户 = Rule 升级发现侧；今天唯一候选类型 =

- `rule_promotion_candidate`：某个 stable entity 下某个客观 scope / pattern 已经具备足够 CaseLogEvidence，值得交会计师确认是否升级为 executable rule。

Rule 升级归本机制发现，是因为“某 entity 的某 pattern 是否够格升 rule”可以由 Rule 侧的客观判句判断：scope 可判定、历史 treatment 唯一无分叉、输出 judgment-free 完整、事前 accountant approval。job 只能判断“是否值得作为候选交给会计师”，不能替代 accountant approval，也不能直接创建 executable rule。

当前收窄后的职责不包括：

- rule 降级 / 失效判断。
- automation 收紧 / restrictive auto-downgrade。
- 语义型 merge / split。
- 跨 rule 互斥 / 引用校验。

这些边界在“四、明确不归它管的”中冻结。

## 二、为什么它必须独立，不能折进 Human Review 或 Rule 侧

曾经有个想法：让 Review Agent 定期扫 log 找候选，把发现也一手包办。这条路是错的，原因是：

- **扫描本质是确定性代码，不是 LLM 的活。** 让 Review 那个 LLM 去扫 log，等于把一件便宜、确定、可审计的事，换成贵的、会幻觉的事；这会抹掉“确定性发现”存在的全部理由。
- **折进 Human Review 会把发现节奏绑到 UI 活跃度上。** 如果会计师打开面板才扫，升级候选会迟到；如果后台仍然跑 job，那它本质上仍是独立 job。
- **折进 Rule 侧会混淆判句定义权和机制管道。** Rule 侧拥有 promotion 资格判句与固定执行路径；确定性发现只负责扫描、调度、候选产出和写 inbox。

正确形态是 **独立自主 job（生产者）+ Human Review 消费（消费者）**。两者通过持久审核 inbox 解耦。Review 面板可以提供“手动 rescan”，但按钮只是调这个 job，不是 Review 自己扫；基线永远是自主调度。

## 三、它怎么运作

### 1. 触发分两种节奏

- **批后增量**：每批交易 finalize / correction 后，只对这批动过的 entity、case pattern、rule refs 重算 Rule 侧 promotion 资格判句，抓新 finalized case / correction 触发的 fresh promotion。
- **定时全扫**：按调度器节奏重算、补漏、backfill，修复批后增量漏掉或历史数据补齐后的候选。

MVP 可以用一个 nightly job 同时覆盖两类诉求，但语义上仍是“双节奏”：批后增量追新鲜度，定时全扫追完整性与可重算。

### 2. 触发权归调度器，不归任何 LLM 节点或 UI

cron / 编排层触发。发现节奏是系统机制，不被 UI 活跃度绑架，也不由 Human Review 可用性决定。

手动 rescan 只是从 UI 调度这个 job；它不改变生产者 / 消费者关系。

### 3. 它是“多判句的笨执行器”，自己不拥有任何判句

每一类候选的“够格”判句，归它所服务对象的治理侧定义。当前只有 Rule 侧 promotion：

- promotion（够不够格交给会计师看、是否可作为 rule 升级候选）资格判句 → 归 **Rule Log / Rule Match 治理**。

job 只是把这些判句**套用**一遍。判句变了，改的是 Rule 侧治理，不是动 job。这样 job 永远薄、可单独审计。不要把判句逻辑塞进 job 或 Review。

Rule 侧判句必须和 Rule Log 的 authority-content 标准分层对齐：

- Scope 可客观判定、历史 treatment 唯一无分叉、输出 judgment-free 完整、事前 accountant approval，是 executable rule authority 的硬标准。
- N 次、跨月、频率、跨 case 模式等，只能是“是否值得提出候选”的证据强度，不能替代 accountant sign-off，也不能直接创建 executable rule。
- job 不拥有阈值数字、分布算法、结果唯一性判定或 CaseLogEvidence 打包规则；这些都归 Rule 侧定义。

**NEW-1**：rule 升级“固定执行路径”的定义也归 Rule 侧。job 只写候选，Human Review 只确认并触发；固定路径负责 authority-content 校验、overlap guard、凭证校验，以及 Rule Log / Governance Log 写入。

### 4. 产出 = 候选，投进审核 inbox

job 把候选写进一个**持久的审核 inbox**。候选必须活到有人来看，不能是 runtime-only。

- 审核 inbox 是新的数据层，且 **不等于 Governance Log**。候选还没人批，不是已生效治理事件；塞进 Governance Log 会污染 authority 审计账本。
- 候选写入也是 durable 写入，按“没有节点裸写 log”的铁律走 Finalization 那套库，但**免凭证**，因为候选不是扩张型变更。
- 候选写 inbox 不冻结字段级 schema、ID、幂等键、状态机或去重策略。

### 5. 反馈是“拉”，不是“推”

**job 永不调用 / 唤醒 Human Review。**

正确解耦是：job 只往 inbox 投，Human Review 在会计师打开面板时主动 pull。生产者只管投递，消费者自己来取。

## 四、明确不归它管的

- **rule 降级 / 失效判断彻底不归确定性发现，归人。** rule 由人创建确认，匹配出错概率低；“某条 rule 是否坏了 / 失效了”是语义判断，系统自发判不出。只有会计师在 review 结果时人发起：一种场景是 Coordinator runtime 发现异常后交 review，另一种场景是审核阶段会计师发现某结果由 rule 做出但错了。后续走 Human Review 扩张型变更路径。确定性发现不产任何降级候选、不累积“被推翻”信号、不做 rule 失效的攒证据判定。
- **automation 收紧 / restrictive auto-downgrade 不归确定性发现。** 这是安全方向的自动收紧，自动免闸生效，不需要人审批；归 Entity Log `automation_policy` maintenance / mutation contract。未来若另设 Entity policy detector，必须独立落文，不回填进当前 rule promotion job。
- **语义型 merge / split 不归确定性发现。** 这是语义自发发现，「系统自发凭什么」无可审计依据；该系统自发语义发现层已于 2026-06-23 裁撤删除。merge / split 只保留会计师人发起（Human Review）；可确定性化的身份/一致性冲突（exact 冲突、stale projection 等）归确定性发现 / 写入闸 / ER 运行期判句，仍非本机制当前职责。
- **跨 rule 互斥 / 引用校验不归确定性发现。** 这是一道写入 / 生效前的同步守卫，命中冲突直接 fail-closed；它长在写入闸口上，不是事后发现。

### 为什么确定性发现能放心自主，系统自发语义发现却被删除

二者性质不同：

- **确定性发现**：依据是明确、可审计的判句。系统自发跑没有“凭什么”的问题，且它只提候选，会计师仍把关。
- **系统自发语义发现**：依据是模糊语义判断，“系统自发凭什么”无可审计依据——架构已拒绝 semantic similarity 作 identity authority。因此它已于 2026-06-23 裁撤删除（同 rule 失效，归人专属）。

所以自主只适合确定性发现；系统自发语义发现不保留，相关 merge / split 与错误纠正只走会计师人发起。

## 五、它在整条治理线里的位置

```text
确定性发现 job（自主、定时 / 批后、非 LLM）
        │  产出候选
        ▼
   审核 inbox（持久队列，≠ Governance Log）
        │  Human Review 主动来读（拉，不是推）
        ▼
Human Review Node（摆给会计师、read-back、会计师签字）
        │  带凭证
        ▼
Rule 侧固定执行路径（authority-content 校验、overlap guard、凭证校验）
        │
        ▼
Finalization（跨 log 落盘、写 Rule Log / Governance Log）
```

注意：这条链只是**发现侧候选**走的其中一条路，不是全系统唯一路径。会计师在 Human Review 当场发起的永久纠错**跳过发现 job**，因为人就是发现者；ER 写 Entity Log 这类运行期写入既不过发现，也不过人类审批闸，只走对应 Finalization 写入路径。

## 六、MVP 搁置 / 未冻结（Open Boundaries）

- **推进顺序**：确定性发现未来落入正式 `BK_Copilot/` shared mechanism spec 前，必须排在 Rule 侧 promotion 资格判句定义之后。Rule 侧 promotion 判句当前仍是 Rule Log / Rule Match open boundary；先写正式 spec 会空转。
- **审核 inbox 候选状态 / 去重 / 过期**：MVP 搁置，但这是生产化第一批会咬人的点。上线第一批必须补状态、去重、过期、压制已决项、稳定候选键和幂等策略，避免 nightly job 重复冒泡同一候选。
- **审核 inbox 与 Governance Log 均待建**：inbox 承接候选，Governance Log 承载已批准治理事件；二者不能互相替代。
- **触发细节仍未冻结**：双节奏语义已定；exact cadence、scope batching、run record、手动 rescan 的调度 contract 留 L4 / seam。
- **字段级 schema 不冻结**：candidate schema、inbox 字段、candidate id、幂等键、refs 形态、query API、权限、projection / pagination / filtering contract 均保持 open boundary。
- **Rule 侧判句与固定执行路径仍归 Rule 侧收口**：promotion 资格判句、CaseLogEvidence 资格、结果唯一性判定、证据强度、CaseLogEvidence 打包格式、固定执行路径和 approval workflow 不在本机制内冻结。
- **rule 失效 / 降级不由确定性发现承接**：rule 失效判断归人，本机制不产降级候选、不累积“被推翻”信号、不触发 rule 失效候选。会计师纠错事实可作为 Transaction Log correction append + rule-hit ref 的审计事实存在，但不驱动系统自发攒证据判定。
- **排除项若未来重启，必须另行落文**：automation 收紧归 Entity Log automation_policy maintenance / mutation contract；语义型 merge / split 的**系统自发发现已裁撤删除**（只保留人发起，可确定性化部分归确定性发现 / 写入闸 / ER 判句）；跨 rule 互斥 / 引用校验归写入 / 生效前同步守卫。

以上是当前唯一权威设计结论，但仍不是字段级 contract，也不表示正式 `BK_Copilot/` spec 已完成。
