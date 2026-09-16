# 研究來源索引

最後整理日期：2026-09-16

這份文件是本專案的來源基線。架構與 backlog 中的外部事實，原則上應能回到這裡找到原始來源；若只是我們的設計選擇，必須標示為「設計決策」，不能偽裝成平台規則。

## 來源等級

1. `OFFICIAL_PRIMARY`：平台官方條款、官方 Help Center、官方 Developer Docs。
2. `FIRST_PARTY_CODE`：我們自己的既有 repo / production code。
3. `OPEN_SOURCE_PRIOR_ART`：公開 repo；只能證明「有人這樣實作」，不能當作 Shopee TW 或社群平台規則。
4. `DESIGN_DECISION`：本專案基於目標、風險、成本做的架構裁決。

---

# A. 台灣 Shopee 分潤官方來源

## A1. 蝦皮聯盟計畫約定條款

- 等級：`OFFICIAL_PRIMARY`
- URL：https://help.shopee.tw/portal/10/article/124075
- 備援條款頁：https://help.shopee.tw/portal/4/article/77310
- 支援的架構判斷：
  - 分潤率屬於推廣服務條件，不應視為 Product 的永久靜態屬性。
  - 完成交易與推廣費用有平台定義與驗證條件。
  - Affiliate Promotion 必須經 Policy Gate；找到熱門討論不等於可以發布。
  - 禁止內容與推廣規則應做 deterministic policy，而不是交給 LLM 自由判斷。

## A2. 分潤後台儀表板

- 等級：`OFFICIAL_PRIMARY`
- URL：https://help.shopee.tw/portal/10/article/123915
- 官方可觀測訊號：
  - 點擊數
  - 訂單數
  - 預估分潤
  - 商品銷售數量
  - 訂單金額
  - 新買家
  - 商品成效排名：「最熱銷」、「最高分潤」
  - 訂單成立時間、訂單完成時間
- 對應架構：
  - `ProductOpportunityEngine`
  - `AffiliatePerformanceMapper`
  - Product / Placement / Content feedback loop
- 注意：預估分潤不是最終可得分潤，最終值可能因訂單狀態及驗證改變。

## A3. 批次取得分潤連結與 Sub ID

- 等級：`OFFICIAL_PRIMARY`
- URL：https://help.shopee.tw/portal/10/article/123919
- 可確認能力：
  - App 自訂連結一次最多 5 個商品 URL。
  - 官網商品推薦可選取最多 100 個商品後批次取得連結。
  - 每條連結最多可帶 5 個 Sub ID。
  - Sub ID 可用於點擊與轉換報告追蹤。
- 對應架構：
  - P0 可先走 `OFFICIAL_EXPORT / MANUAL_IMPORT`，不必一開始就依賴未知 API。
  - Sub ID 是第一版 attribution 的主要來源。

## A4. 客製化連結 / Sub ID 報告

- 等級：`OFFICIAL_PRIMARY`
- URL：https://help.shopee.tw/portal/10/article/123920
- 可確認能力：
  - 最多 5 個 Sub ID。
  - 點擊報告可依點擊時間、點擊 ID、Sub ID、點擊區域分析。
  - 報告可輸出 Excel。
  - 官方明確建議使用 Sub ID 比較渠道、類別、產品、活動與新渠道測試。
- 本專案第一版 Sub ID 建議：
  1. product
  2. platform
  3. placement
  4. content_variant
  5. campaign
- 上述五格分配是 `DESIGN_DECISION`，不是 Shopee 規定格式。

## A5. 高分潤商品

- 等級：`OFFICIAL_PRIMARY`
- URL：https://help.shopee.tw/portal/10/article/123917
- 可確認規則：
  - 高分潤商品存在額外加碼。
  - 自 2024-03-04 起，高分潤商品額外分潤只針對直接訂單。
  - 部分賣家有推廣預算上限；額度用完後額外加碼分潤率可能即時終止。
- 對應架構：
  - `AffiliateOfferSnapshot` 必須有 `observed_at` / freshness。
  - 發布前可重新驗證 Offer，不應因佣金變化重新生成整個 Content Pack。

## A6. 分潤成效報告匯出

- 等級：`OFFICIAL_PRIMARY`
- URL：https://help.shopee.tw/portal/10/article/189689
- 可確認能力：
  - 電腦版可匯出成效報告與訂單資訊。
  - 官方後台數據是最終判讀基準之一。
- 對應架構：
  - P0 的 `ConversionFeedback` 可以從官方匯出建立，不必等待公開 API。

## A7. 分享推廣碼

- 等級：`OFFICIAL_PRIMARY`
- URL：https://help.shopee.tw/portal/10/article/153369
- 可確認能力：
  - 若社群平台禁止外部分潤 URL，可使用 Shopee 分享推廣碼。
  - 推廣碼可放在文字、影片標題、字幕或評論區，使用者回 Shopee 搜尋。
- 對應架構：
  - `ContentPack` / `PlacementPolicy` 應支援 `affiliate_link` 與 `promotion_code` 兩種 CTA 模式。

## A8. Facebook 聯盟合作（台灣 Shopee）

- 等級：`OFFICIAL_PRIMARY`
- FAQ：https://help.shopee.tw/portal/10/article/185362
- 發布教學：https://help.shopee.tw/portal/10/article/185359
- 可確認能力：
  - Facebook 內建聯盟合作可瀏覽 Shopee 分潤商品並建立貼文 / Reels。
- 對應架構：
  - 社群發布應優先考慮官方整合與正式 API，而不是瀏覽器自動化。

---

# B. 社群平台官方來源

## B1. Reddit Developer API — comment / user action

- 等級：`OFFICIAL_PRIMARY`
- RedditClient：https://developers.reddit.com/docs/api/public-api/classes/RedditClient
- User Actions：https://developers.reddit.com/docs/capabilities/server/userActions
- API Overview：https://developers.reddit.com/docs/capabilities/server/reddit-api
- 可確認能力：
  - 官方 API 支援 `submitComment()`。
  - 可指定 `runAs: USER` 或 `APP`，但須符合 Reddit app 權限與審核要求。
- 對應架構：
  - 技術上不必把 Selenium/Playwright 當唯一留言方式。
  - `PlatformAdapter` 應優先使用正式 API。

## B2. Reddit Spam Policy

- 等級：`OFFICIAL_PRIMARY`
- 繁中：https://support.reddithelp.com/hc/zh-tw/articles/360043504051
- 英文：https://support.reddithelp.com/hc/en-us/articles/360043504051-Spam
- 重要限制：
  - 禁止大量重複或不請自來的參與。
  - 明列「為曝光或經濟利益大量發布重複內容」可能違規。
  - 明列「讓 bot 在單一或多個社群持續宣傳特定產品 / 服務」為違規範例。
  - 各 subreddit 仍可有更嚴格的自我推廣規則。
- 對應架構：
  - `PlacementPolicy` 必須先於 Publisher。
  - Reddit 預設不應被當成無限制 AUTO_ALLOWED 的 affiliate placement。

## B3. Reddit Apps / scraping boundary

- 等級：`OFFICIAL_PRIMARY`
- URL：https://support.reddithelp.com/hc/en-us/articles/360043512931-Don-t-break-the-site
- 重要限制：
  - 未經授權 scraping Reddit 可能違反規則。
  - app / bot / AI agent 要受 API 與 builder policies 約束。
- 對應架構：
  - Discussion source 需有 `source_method` / authorization evidence。
  - 不把任意 scraper 當長期核心依賴。

## B4. TikTok Content Posting API

- 等級：`OFFICIAL_PRIMARY`
- Get Started：https://developers.tiktok.com/docs/en/content-posting-api-get-started
- Direct Post：https://developers.tiktok.com/docs/en/content-posting-api-reference-direct-post
- Get Post Status：https://developers.tiktok.com/docs/en/content-posting-api-reference-get-video-status
- 可確認能力：
  - Direct Post 需要正式 app、`video.publish` scope 與使用者授權。
  - 未審核 client 的內容會受可見性限制。
  - 發布為非同步流程，可用 polling 或 webhook 確認狀態。
- 對應架構：
  - `LockedPublisher` 可採 outbound polling，不必為第一版暴露 inbound webhook。
  - Publisher 應保存平台 remote ID / publish status receipt。

---

# C. 開源 Prior Art

以下只能作 donor / prior art，不代表平台正式支援。

## C1. MarketMeNow

- 等級：`OPEN_SOURCE_PRIOR_ART`
- Repo：https://github.com/thearnavrustagi/marketmenow
- License：MIT
- 吸收：
  - `ContentCapsule`
  - publication history
  - derived-from / repurpose
  - platform adapter contract
- 不採：通用 agent/browser publisher 作本專案 security boundary。

## C2. social-demand-signal-agent

- 等級：`OPEN_SOURCE_PRIOR_ART`
- Repo：https://github.com/jackterror/social-demand-signal-agent
- 吸收：
  - lexical relevance
  - intent signal
  - freshness
  - deterministic route / suppress / review

## C3. hidrix-tools

- 等級：`OPEN_SOURCE_PRIOR_ART`
- Repo：https://github.com/sonpiaz/hidrix-tools
- 吸收：
  - engagement weighted scoring
  - time decay
  - Reddit top/thread reading pattern
  - SQLite dedup / run history

## C4. affiliate-product

- 等級：`OPEN_SOURCE_PRIOR_ART`
- Repo：https://github.com/tody-agent/affiliate-product
- License：MIT
- 吸收：
  - hard eligibility gates
  - multi-factor affiliate score
  - seller self-selling / KOC concentration thinking
  - SKU / provenance / risk gate
- 明確限制：
  - 越南市場導向（VN / VND）
  - FastMoss / TikTok Shop / Shopee Vietnam 資料不可成為 Shopee TW 正式依賴
  - 只吸收 scoring / contract 思想

---

# D. 內部既有能力

## D1. Pantheon Content Runtime

- 等級：`FIRST_PARTY_CODE`
- Repo：https://github.com/bluemaple18-home/Pantheon
- 主要 donor：`scripts/agy_seo_copy_pipeline.py`
- 可重用：
  - Writer / Reviewer 分離
  - model routing
  - capability check
  - schema repair
  - approval / apply boundary
  - fail-closed validation pattern
- 不共用：
  - Pantheon SEO policy
  - 算命文章格式
  - MysticPantheon identity
  - FAQ / disclaimer / site-specific rules

## D2. AI Core reuse-first 原則

- 等級：`FIRST_PARTY_CODE`
- Repo：https://github.com/bluemaple18-home/aicore
- 本專案只沿用原則：
  - `USE_AS_IS → CONFIGURE → WRAP → ADAPT → COPY_CODE → CUSTOM_REQUIRED`
  - existing seam first
- Affiliate domain logic 預設不進 AI Core。

---

# E. 目前仍未證實 / 不可假設

以下不得在實作中假裝已存在：

1. 台灣 Shopee Affiliate 是否有對一般推廣者公開且正式文件化的 Open API。
2. 台灣 Shopee 是否能透過 API 直接批量取得完整 Product Offer / Commission / Conversion，而非後台匯出。
3. 每一個社群平台是否允許 affiliate link、promotion code 或 bot reply；必須逐平台、逐社群規則確認。
4. 任一開源 repo 的 scraping / browser automation 行為是否符合平台當下條款。

如果以上資訊沒有官方證據，Adapter 必須標為 `UNVERIFIED / UNSUPPORTED`，不能自行推定。
