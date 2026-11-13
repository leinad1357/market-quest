# Chapter 4 案例录入说明

**The Responsible Sourcing Challenge · 负责任的采购挑战**

本文件说明如何在 **不改动游戏逻辑** 的前提下，把五个供应商案例的真实内容填进游戏。
所有案例数据集中在一个地方：`index.html` 中的 `const C4_CASES`（约第 5547 行）。
五个场景共用同一套三阶段界面（广告 → 背景调查 → 结算）与同一套结算函数，因此**只改数据、不改代码**。

---

## 0. 章节开始页与采购预算

第四章有一张独立的**任务简报开始页**（路由 `#/ch4/intro`）。从首页点击第四章、或任何走 `enterChapter4()` 的路径，都会先落到这一页，而不是直接进场景。

开始页只承担三件事，不涉及任何场景数据：

1. **角色说明**：学生是学校采购代表，五家供应商要争取这笔订单，学生要读广告、要证据、做决定。
2. **预算发放**：`C4_BUDGET = 8000`，与第三章余额直接累加，界面上用 `第三章结余 + 本章预算 = 当前可用` 一条式子显示。
3. **结算规则**：所有采购与额外费用（检测、召回、重新采购）都直接从顶部和侧栏的金币计数器扣除；报价不等于最终花费。

预算相关的状态字段：

| 字段 | 含义 |
|---|---|
| `state.ch4.carriedCoins` | 发放时第三章带过来的余额（用于开始页显示，之后不再变） |
| `state.ch4.budget` | 本章预算额（当前 8000） |
| `state.ch4.budgetGranted` | 是否已发放，防止重复领取 |
| `state.ch4.briefingSeen` | 是否已看过开始页；为 false 时 `#/ch4` 会重定向到 `#/ch4/intro` |

规则要点：

- 预算**每轮只发一次**：`c4GrantBudget()` 在 `enterChapter4()` 与开始页渲染时调用，`budgetGranted` 为 true 后直接返回。
- 发放后的 `coins = carriedCoins + 8000`，随后所有场景结算都从这个数字里扣；不够时按原有规则转成债务。
- 重玩单个场景（`c4ReplayScene`）只回滚该场景的结算，不会重发预算。
- **报告页的「Replay Chapter 4 · 重玩第四章」按钮**（`c4ReplayChapter`）会清空整章：五个案例结果、星星、风险、债务、剩余预算全部归零，并回到开始页重新发放一次 8,000 预算（第三章结余与进度不受影响）。点击前有确认弹窗，避免误触。
- 想改预算额，改 `C4_BUDGET` 常量即可；对已经发放过的存档，改大不会自动补差（如需补发要清掉 `state.ch4.budgetGranted` 或整档重置）。

---

## 1. 五个案例的位置

```js
const C4_CASES = [
  { id:"ecobottle",  sceneNumber:1, status:"complete", ... },  // EcoBottle  危险
  { id:"everpack",   sceneNumber:2, status:"complete", ... },  // EverPack   负责
  { id:"brightbox",  sceneNumber:3, status:"complete", ... },  // BrightBox  危险
  { id:"renew",      sceneNumber:4, status:"complete", ... },  // ReNew      负责
  { id:"soundmax",   sceneNumber:5, status:"complete", ... }   // SoundMax   危险
];
```

**五个案例（EcoBottle / EverPack / BrightBox / ReNew / SoundMax）均已录入完整正式内容**，本文件从此转为维护参考：新增或替换案例时照着现有五个案例的结构写即可。

三个新用法的示例：
1. **BrightBox** — 供应商给的文件是真的，但归属另一个型号。做法是把「受检产品/型号」与「采购产品/型号」并列写进 `evidence.rows`，给型号不符与「未提供」两行加 `state:'bad'`，证据状态用 `warning`，再用 `gap` 点明「真实证书不等于本产品安全」。
2. **ReNew** — 示范「社区/本地经济」领域：`area:"community"` 的证据行放本地员工数、带薪学徒数、本地维修伙伴、安全培训与可查阅记录，把「订单留在本地」写成可核查的事实，而不是情绪化表述。负风险（−15）同样受 0 下限约束。
3. **SoundMax** — 报价单是 `verified`，但 rows 里把 6 个月保修、电池衰减不保、无过保电池服务、仅邮件支持逐条标 `bad`。状态表示「这份文件是真的」，`bad` 表示「条款对采购方不利」，两者可以并存；结论交给学生在 VAL 里自己判断。

- **产品名称 / 中文名 / 产品图片**：改每个案例的 `product.nameEn`、`product.nameZh`、`product.image`（填一个图片 URL，留 `null` 则显示占位图框）。
- **产品介绍**：`product.descriptionEn` / `product.descriptionZh`。
- **供应商名称**：`supplier.name`。

> `C4_DEMO_CASE` 是独立的测试案例（标 `TEST DATA`），不属于五个正式场景，不要把它当作正式内容，也不要删除——它用于每次改动后验证机制是否正常。

---

## 2. 广告与报价

```js
supplier: {
  name: "...",
  advertisement: {
    sloganEn: "广告标语", sloganZh: "广告标语中文",
    claims: ["宣传优势 1", "宣传优势 2", "宣传优势 3"]
  },
  unitPrice: 6,        // 单价（金币）
  minimumOrder: 100,   // 最小订购量
  quotation: 600       // 总报价（建议 = 单价 × 最小订购量）
}
```

- `claims` 里可以放吸引人的宣传语，但**不要写进真实证据**——证据只放在 `investigation.options[].response`。
- 总报价应与结算里的 `purchaseCost` 保持一致，否则学生会看到两个不同的数字。

---

## 3. 五个调查领域

每个案例从 8 个责任领域中挑 5 个，写在 `investigation.options` 里：

| 领域 ID | 英文 | 中文 |
|---|---|---|
| `env` | Environmental Sustainability | 环境可持续性 |
| `fair` | Fair Trade & Ethical Sourcing | 公平贸易与道德采购 |
| `worker` | Workers' Rights & Safe Conditions | 劳工权益与工作安全 |
| `waste` | Waste Reduction & Circular Consumption | 减少浪费与循环消费 |
| `honest` | Honest Advertising & Informed Choices | 真实广告与知情消费 |
| `needs` | Needs, Wants & Responsible Spending | 需求、欲望与理性消费 |
| `community` | Supporting Communities & Local Economies | 支持社区与本地经济 |
| `safety` | Product Safety & Quality | 产品安全与质量 |

`maxRequests: 3` 表示学生最多选 3 个领域调查，一般不需要改。

---

## 4. 供应商回复与证据

每个领域的回复结构：

```js
{
  area: "safety",
  questionEn: "...", questionZh: "...",       // 留空则自动用该领域的默认问题
  response: {
    state: "incomplete",                      // 见下表
    answerEn: "供应商的回答",
    answerZh: "供应商的回答（中文）",
    evidence: {
      titleEn: "文件标题", titleZh: "文件标题中文",
      source: "来源机构", date: "2025-09",
      // 可选：把检测表/报价单做成小表格（EcoBottle 的 EB-QC-02 就是这样做的）
      rows: [
        { en:"Food-contact Material Test / 食品接触材料检测", valueEn:"NOT PROVIDED", valueZh:"未提供", state:"bad" }
      ],
      // 可选：这份文件没有证明什么
      gapEn: "A leak test does not prove the bottle is safe for drinking water.",
      gapZh: "漏水测试不能证明水瓶可以安全接触饮用水。"
    }
  }
}
```

- `rows[].state`：`ok` 显示为正面色，`bad` 显示为负面色，不填则用普通色。
- `gapEn` / `gapZh` 会渲染成「What the document does not provide · 文件未提供」，这是训练学生分辨「有文件」与「文件能证明什么」的关键一行，建议每个回复都写。

六种证据状态（`state`）：

| 值 | 界面显示 | 适用情形 |
|---|---|---|
| `verified` | Verified Evidence · 已核实证据 | 有第三方/可核验文件 |
| `statement` | Supplier Statement · 供应商声明 | 只有供应商自述，无独立证据 |
| `incomplete` | Incomplete Information · 信息不完整 | 给了部分资料，答不全 |
| `unavailable` | Information Unavailable · 无法提供信息 | 表示暂时没有资料 |
| `refused` | Refused to Disclose · 拒绝披露 | 明确拒绝提供 |
| `warning` | Warning Evidence · 风险证据 | 提供的材料本身有安全/环境/劳工问题 |

> 证据状态 ≠ 供应商是否安全。危险供应商可以给出「看起来完整但不可核验」的资料；负责供应商也可能在非关键问题上只给有限信息。请不要把「有资料」直接写成 `verified`。

**未填写的字段会怎样**：界面顶部显示 `Missing fields · 缺失字段` 列表，回复正文显示 `PLACEHOLDER — CASE CONTENT PENDING`。填完后该列表自动消失。

---

## 5. 进一步核实

```js
verification: {
  available: true,
  maxAttempts: 1,          // 最多核实几次（建议 1）
  cost: 0,                 // 每次核实的金币成本
  responseEn: "...", responseZh: "..."
}
```

- 核实成本在点击当时扣除，并计入结算页的 `Verification Cost`。
- 核实后学生仍要自己选「采购」或「拒绝」，对应 `verifiedPurchase` / `verifiedReject` 两套结果。

---

## 6. 隐藏背景（不得展示给学生）

```js
hiddenProfile: {
  supplierType: "dangerous",   // 或 "responsible"
  internalNotes: "给老师/开发者看的备注"
}
```

`supplierType` 只驱动学习反馈与默认评分，界面上**不会**出现「危险 / 负责」标签。

---

## 7. 采购决定的结果

四种结果，结构相同：

```js
outcomes: {
  purchase:         { ... },   // 直接采购
  reject:           { ... },   // 直接拒绝
  verifiedPurchase: { ... },   // 核实后采购
  verifiedReject:   { ... }    // 核实后拒绝
}
```

每个结果里：

```js
{
  purchaseCost: 600,          // 采购成本（拒绝时为 0）
  additionalLoss: 1100,       // 额外损失（检测、处理、替换、召回等）
  refund: 0,                  // 退款
  riskD: 25,                  // 风险变化，可为负
  breakdown: [                // 额外费用明细（可空）
    { en:"Independent Product Testing", zh:"独立产品检测", amount:100 }
  ],
  eventEn: "发生了什么", eventZh: "发生了什么（中文）",
  stakeholders: [             // 利益相关者影响（可多条）
    { en:"Students", zh:"学生", area:"safety", dir:"neg",
      explainEn:"...", explainZh:"..." }
  ]
}
```

`dir` 取值：`pos`（正面）/ `neg`（负面）；`area` 用第 3 节的领域 ID。

**财务规则（由程序统一计算，不要自己写死余额）**：
- 总支出 = `purchaseCost + additionalLoss − refund`，同一笔只扣一次。
- 余额不足时余额归零，差额自动记入 `debt`（结算页显示「未支付」），学生仍可继续后续场景。
- 拒绝的默认值是全 0，但**不要**让「拒绝一切」变成零成本万能策略——可在 `reject` 的 `stakeholders` 里写时间成本、错失机会等负面项。

---

## 8. 星星与风险

```js
scoring: {
  keyAreas: ["safety", "env"],                              // 该案例的主要风险领域
  starsOverride: { purchase: null, reject: null }            // 需要固定分数时填 3 / 1 / 0
}
```

默认评分（`starsOverride` 为 `null` 时生效）：

| 供应商 | 决定 | 条件 | 星星 |
|---|---|---|---|
| dangerous | reject | 调查命中 `keyAreas` 或做过进一步核实 | 3 |
| dangerous | reject | 其他 | 1 |
| dangerous | purchase | — | 0 |
| responsible | purchase | 调查命中 `keyAreas` | 3 |
| responsible | purchase | 其他 | 1 |
| responsible | reject | 命中 `keyAreas` 或做过进一步核实 | 1 |
| responsible | reject | 其他 | 0 |

- 想按案例自定义评分，直接写 `starsOverride: { purchase: 3, reject: 1 }`。
- 风险与星星独立：风险只由 `riskD` 决定。
- **`riskD` 可以是负数**（例如负责任供应商采购后 −10）。风险有下限 0：若计算后小于 0，实际风险停在 0，结算页会标注 `floor 0 · 已到下限`，不会出现负风险。
- `keyAreas` 留空时（纯占位案例）评分兜底 1 星，避免因缺数据惩罚学生。

---

## 9. 学习反馈

两种方式，二选一：

**A. 自动生成（不填 `feedback`）** —— 代码按「供应商类型 + 最终决定」生成四段通用文案。
只有案例 `status` 为 `complete` 或 `demo` 时才启用；`placeholder` 会显示「学习反馈待录入」。

**B. 按案例自定义（推荐，EcoBottle 用的就是这种方式）**

```js
feedback: {
  areas: ["safety", "env", "worker", "fair", "honest", "needs"],  // 本案例涉及的责任领域，会显示成标签
  purchase:         { en:"…", zh:"…" },
  reject:           { en:"…", zh:"…" },
  verifiedPurchase: { en:"…", zh:"…" },
  verifiedReject:   { en:"…", zh:"…" },
  finalEn: "给学生的总结句",
  finalZh: "给学生的总结句（中文）"
}
```

四个分支分别对应「直接采购 / 直接拒绝 / 核实后采购 / 核实后拒绝」。
`finalEn` / `finalZh` 会渲染成 `Final message · 总结` 卡片，放在结算页最下方。

案例内容全部填完后，记得把：

```js
status: "placeholder"  →  status: "complete"
```

否则学生会看到「学习反馈待录入」（若已填 `feedback` 则不受影响，但仍建议改状态）。

---

## 10. 填完一个案例后的自检清单

1. 浏览器打开 `index.html#/ch4/scene/N`，页面顶部的 `Missing fields · 缺失字段` 应为空（或只剩你确实想留空的项）。
2. 广告页显示的单价、最小订购量、总报价与结算页 `Purchase Cost` 一致。
3. 选 3 个领域 → 提交 → 三张回复卡的状态标签正确（六种之一）。
4. 分别试「直接采购 / 直接拒绝 / 核实后采购 / 核实后拒绝」四条路径，确认余额、债务、风险、星星都与配置一致。
5. 点「重玩本案例」，金币、星星、风险应回到进入本案例前的数值，可重复测试。
6. 五个案例全部完成后，`#/ch4/report` 应能生成 Responsible Sourcing Report。

---

## 11. 不要做的事

- 不要在学生做决定之前，把 `hiddenProfile.supplierType` 或「危险 / 负责」字样显示到界面。
- 不要把临时测试数据（`TEST DATA`）写进五个正式案例。
- 不要在同一案例里让 `quotation` 与 `purchaseCost` 冲突，或在 `additionalLoss` 里重复计入采购成本。
- 不要靠改 `renderC4Ad` / `renderC4Check` / `renderC4Summary` 来适配单个案例——这些函数五个场景共用。
