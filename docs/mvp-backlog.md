# MVP Backlog

## P0 — 先驗證能不能賺錢

### 1. 台灣蝦皮資料匯入
- 支援官方匯出 / 人工匯入
- 定義 `ProductSnapshot` 與 `AffiliateOfferSnapshot`
- 佣金、價格、庫存、活動等資料必須帶 `observed_at`
- 不把佣金視為永久欄位

### 2. 商品資格 Gate
- 是否仍在售
- 是否有有效 affiliate offer
- 分潤是否高於最低門檻
- 商品/賣家風險
- 資料是否 stale

### 3. 商品機會評分
先 deterministic：
- 預估每單分潤金
- 銷售/成交訊號
- 商品可信度
- 價格吸引力
- 歷史 conversion
- 外部討論機會

輸出：`PromotionCandidate`

### 4. 內容需求與內容包
- 只替 Top candidates 生成
- 優先重用既有 Writer / Reviewer
- 第一版先做文字
- 內容包至少包含：推薦型、比較型、問答型、短文型
- 商品 facts 與 affiliate offer 分離，避免佣金變動就整包重生

### 5. 熱門討論搜尋器
- 從商品 facts/use cases 產生 query
- 收集候選討論
- 去重
- 保存 source / URL / published_at / engagement metrics

### 6. 0 Token Placement Ranking
- keyword relevance
- purchase intent
- freshness
- reactions/comments/shares
- velocity
- duplicate / already handled

只有 Top candidates 才允許 optional batch LLM check。

### 7. Placement Policy Gate
輸出固定為：
- `ALLOW`
- `REVIEW_REQUIRED`
- `DENY`

不得以 LLM 自由決定平台規則。

### 8. Signed Publication Job
- 內容 hash
- target
- account
- content variant
- affiliate link / Sub ID
- policy decision
- expiry
- signature

### 9. Locked Publisher
- outbound-only
- no inbound endpoint
- no webhook dependency for MVP
- no LLM
- no MCP
- no arbitrary tools
- egress allowlist
- process 完成後退出

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

### 11. 成效回饋
以 Sub ID / click / order / commission 建立：
- Product performance
- Placement performance
- Content variant performance

## P1 — P0 有實際轉換後才做

- 影片腳本
- 短影片生成 / render queue
- 多平台 adapter 擴張
- 自動 weight calibration
- 更完整的 Dashboard
- offer freshness 自動同步（僅限官方允許來源）

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
