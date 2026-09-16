# MVP Backlog

> 每一個 P0 能力都必須能對應到 `docs/research-sources.md` 的官方來源、內部既有能力或明確設計決策。若能力未被官方證實，Adapter 必須標為未驗證，不得偷偷假設存在。

## P0 — 先驗證能不能賺錢

### 1. 台灣蝦皮資料匯入

證據：Shopee TW 官方後台目前可批次取得商品分潤連結、最多 100 個商品，並可匯出成效報告；尚未確認一般推廣者有正式公開 Affiliate Open API。

- 支援官方匯出 / 人工匯入
- 定義 `ProductSnapshot` 與 `AffiliateOfferSnapshot`
- 佣金、價格、庫存、活動等資料必須帶 `observed_at`
- 不把佣金視為永久欄位
- source method 必須標 `OFFICIAL_API / OFFICIAL_EXPORT / MANUAL_IMPORT / UNSUPPORTED`

來源：`research-sources.md` A2 / A3 / A5 / A6 / E。

### 2. 商品資格 Gate

證據：Shopee 分潤率、完成交易、直接/間接訂單、高分潤加碼都有平台條件；高分潤加碼也可能因賣家預算即時終止。

- 是否仍在售
- 是否有有效 affiliate offer
- 分潤是否高於最低門檻
- 商品/賣家風險
- 資料是否 stale
- offer 是否仍有效

來源：A1 / A5。

### 3. 商品機會評分

先 deterministic：
- 預估每單分潤金
- 銷售/成交訊號
- 商品可信度
- 價格吸引力
- 歷史 conversion
- 外部討論機會

Shopee 官方後台已提供點擊、訂單、預估分潤、銷售數量、訂單金額、新買家與「最熱銷 / 最高分潤」排行，可作第一版真實 feedback signal。

輸出：`PromotionCandidate`

來源：A2；affiliate scoring 方法可參考 Prior Art C4，但台灣 threshold 不沿用越南設定。

### 4. 內容需求與內容包

- 只替 Top candidates 生成
- 優先重用 Pantheon generic Writer / Reviewer 機制
- 第一版先做文字
- 內容包至少包含：推薦型、比較型、問答型、短文型
- 商品 facts 與 affiliate offer 分離，避免佣金變動就整包重生
- 支援 `affiliate_link` 與 `promotion_code` CTA 型態

來源：內部 D1；Shopee 分享推廣碼能力 A7；Content Pack Prior Art C1。

### 5. 熱門討論搜尋器

- 從商品 facts / use cases 產生 query
- 收集候選討論
- 去重
- 保存 source / URL / published_at / engagement metrics
- 保存 `source_method` 與授權/規則狀態
- 未經授權 scraping 不列為長期正式 dependency

來源：Reddit B3；Prior Art C2 / C3。

### 6. 0 Token Placement Ranking

- keyword relevance
- purchase intent
- freshness
- reactions/comments/shares
- velocity
- duplicate / already handled

只有 Top candidates 才允許 optional batch LLM check。

來源：Prior Art C2 / C3。這一層是 `DESIGN_DECISION`，不是平台規則。

### 7. Placement Policy Gate

輸出固定為：
- `ALLOW`
- `REVIEW_REQUIRED`
- `DENY`

不得以 LLM 自由決定平台規則。

Reddit P0 預設不能因為官方 API 支援 `submitComment()` 就直接視為 AUTO_ALLOWED；Reddit 官方 Spam Policy 明確禁止大量、重複、未經請求的商業曝光，並把持續宣傳產品的 bot 列為違規範例。每個 subreddit 還可能有更嚴格規則。

來源：Shopee A1；Reddit B1 / B2 / B3。

### 8. Signed Publication Job

- 內容 hash
- target
- account
- content variant
- affiliate link / promotion code / Sub ID
- policy decision
- expiry
- signature

這是 `DESIGN_DECISION`，用途是隔離 Content Runtime 與 Publisher 權限。

### 9. Locked Publisher

- outbound-only
- no inbound endpoint
- no webhook dependency for MVP
- no LLM
- no MCP
- no arbitrary tools
- egress allowlist
- process 完成後退出

平台正式 API 優先於 browser automation。TikTok 官方 Content Posting API 支援 Direct Post，發布狀態可透過 polling 或 webhook 確認，因此 MVP 可以選 outbound polling，不必為狀態 callback 暴露 inbound endpoint。

來源：TikTok B4；安全邊界本身是 `DESIGN_DECISION`。

### 10. Publication Receipt

至少保存：
- job_id
- target
- content_hash
- attempted_at
- published_at
- remote_post_id / URL（若平台提供）
- status / error_code
- idempotency_key

來源：TikTok B4 顯示發布本身可能是非同步流程；Receipt schema 是 `DESIGN_DECISION`。

### 11. 成效回饋

以 Sub ID / click / order / commission 建立：
- Product performance
- Placement performance
- Content variant performance

Shopee 官方允許最多 5 個 Sub ID，並可依 Sub ID、click ID、時間、區域分析與 Excel 匯出。第一版建議把五格用於 product / platform / placement / content_variant / campaign；這個分配是本專案設計，不是 Shopee 固定欄位語意。

來源：A2 / A3 / A4 / A6。

## P1 — P0 有實際轉換後才做

- 影片腳本
- 短影片生成 / render queue
- 多平台 adapter 擴張
- 自動 weight calibration
- 更完整的 Dashboard
- offer freshness 自動同步（僅限官方允許來源）
- 更多正式社群 API adapter

## Later

- 更進階的 bandit / experiment allocation
- 跨 affiliate network
- 更完整的 creator / audience saturation model

## 明確不做

- Multi-Agent 掃商品
- LLM 逐篇讀全網
- 第二套 Writer/Reviewer runtime
- 第二套 scheduler/runtime 只為這個專案存在
- Affiliate domain logic 上收 AI Core
- Publisher 對外暴露可呼叫 Agent endpoint
- 把第三方 scraper 的「能抓到」誤當成「平台允許」
- 把其他國家 Shopee Affiliate API/threshold 假裝成台灣正式能力
