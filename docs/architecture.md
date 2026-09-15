# 架構基線

## 1. 主流程

```text
[台灣蝦皮官方資料]
        ↓
[商品資格 Gate]
        ↓
[商品機會評分]
        ↓
[討論機會快速驗證]
        ↓
[可推廣商品候選]
        ↓
[內容需求]
        ↓
[內容包]
        ↓
[熱門討論搜尋]
        ↓
[Deterministic 初篩]
        ↓
[Placement 配對評分]
        ↓
[平台/分潤政策 Gate]
        ↓
[Signed Publication Job]
        ↓
[Locked Publisher]
        ↓
[Publication Receipt]
        ↓
[Click / Order / Commission]
        ↓
[Feedback]
```

## 2. 責任邊界

### 台灣蝦皮 Adapter
只負責取得與標準化官方資料，不決定商品值不值得推。

優先來源：
1. 官方 API（若帳號／計畫實際提供）
2. 官方匯出
3. 人工匯入
4. 其他來源必須另做政策與證據評估

### 商品機會引擎
回答：「現在什麼商品值得投入？」

先做 hard gate，再做 score。佣金是有時間性的 Offer Snapshot，不視為商品永久屬性。

### 內容工廠
回答：「這個商品有哪些可重複使用的內容資產？」

優先重用既有 generic Writer / Reviewer；Affiliate 只提供 domain facts、限制、輸出 profile。

### 熱門討論搜尋器
回答：「這個已選商品現在有哪些值得看的討論位置？」

商品已先確立，搜尋 query 從商品 facts / use cases / intent terms 產生。

### Placement 評分
回答：「哪個討論位置最值得切入？」

優先 deterministic：
- 商品相關性
- 購買意圖
- 熱度
- 時效
- 熱度速度
- 重複/已處理
- 平台政策

只有少量 Top candidates 才允許選擇性 LLM 語意判斷。

### Policy Gate
找到熱門串不代表可以發布。必須先判斷：
- ALLOW
- REVIEW_REQUIRED
- DENY

### Locked Publisher
安全邊界：
- 不提供公開 inbound API
- 不接 webhook 作為第一版必要條件
- 不含 LLM
- 不含 MCP
- 不接受 arbitrary command
- 不允許任意網域 egress
- 不自行改內容、帳號、時間或目標
- 只讀取已核准、完整、可驗證的 publication job
- 執行後留下 receipt 並退出

### Feedback
把實際成果回寫，不讓 LLM 自己「學」。

核心 attribution：
- product
- platform
- placement
- content variant
- campaign
- clicks
- orders
- commission

優先利用 Shopee Sub ID 與官方成效資料做實際歸因。

## 3. 內容產製與算命的關係

算命不是 Affiliate 的上游；兩個 domain 只可能共用薄的 generic content capability。

```text
             [共用內容能力]
             /          \
      [算命 Adapter]   [Affiliate Adapter]
           ↓                 ↓
       算命內容           分潤內容
```

允許重用：Writer、Reviewer、model routing、repair、approval、validation pattern。

禁止直接共用：算命 SEO 結構、Pantheon identity、算命 disclaimer、FAQ 格式、站點 policy。

## 4. AI Core 邊界

預設不進 AI Core。

只有以下條件同時成立才重新評估：
- 至少跨多個獨立產品反覆需要
- contract 穩定
- 有清楚治理價值
- 不會把 domain logic 帶進核心
- 現有 shared package / wrapper 已無法合理承擔

## 5. Token 預算原則

禁止：
- 用 LLM 掃全商品
- 用 LLM 逐篇讀數千討論
- 每找到一個 placement 就重新生成完整內容

允許：
- Top 商品生成內容包
- 少量 Top placements 做 batch semantic check
- 必要時做極小 contextual rewrite

目標：Placement Hunter 可以在很多 run 中完全 0 token。
