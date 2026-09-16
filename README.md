# Shopee Affiliate Opportunity Engine

台灣蝦皮分潤的 Product-first 自動化專案。

## 專案目標

這個專案不是一般社群排程器，也不是從熱門話題反推商品。

主流程固定為：

1. 先從台灣蝦皮找出值得推的商品。
2. 以分潤、成交、商品品質、歷史成效等訊號做 deterministic-first 排名。
3. 只替真正值得投入的商品建立內容包。
4. 再出去找與商品高度相關、正在升溫的熱門文章或討論串。
5. 以熱度、時效、購買意圖、商品相關性做 Placement Ranking。
6. 經平台／分潤政策檢查後，產生已核准的發布任務。
7. 由鎖死的 Publisher 執行；Publisher 不含 LLM、不接受任意指令、不暴露 inbound endpoint。
8. 以 Sub ID、點擊、訂單、實際分潤回饋商品評分、Placement 評分與內容版本。

## 核心原則

- **Product-first**：商品先行，熱門討論是 placement 驗證與投放位置，不反過來決定商品。
- **Deterministic first**：搜尋、去重、熱度、時效、基本相關性、成本與政策 gate 優先用規則與統計，不用 LLM。
- **低 Token**：LLM 主要用於少量內容生成與必要的最終語意判斷；禁止用 LLM 遍歷大量候選。
- **Reuse first**：先盤點既有模組，再依 `USE_AS_IS → CONFIGURE → WRAP → ADAPT → COPY_CODE → CUSTOM_REQUIRED` 決定是否新寫。
- **Domain isolation**：算命、分潤彼此不是上下游；只共用真正 generic 的薄模組。
- **AI Core hard boundary**：除非能力跨多專案、長期穩定且治理價值極高，否則不放入 AI Core。
- **Official source first**：台灣 Shopee 資料優先使用官方後台、官方匯出或正式 API；不把未經允許的抓取當核心依賴。
- **Evidence before assumption**：平台規則、API 能力、分潤條件要能回到官方來源；沒證據的能力標成未驗證，不自行假設。
- **Publisher is dumb**：Publisher 只驗證並執行已核准任務，不思考、不聊天、不呼叫任意工具。

## 主流程

```text
台灣蝦皮商品 / 分潤資料
        ↓
商品資格檢查
        ↓
商品賺錢潛力評分
        ↓
討論機會快速驗證
        ↓
可推廣商品候選
        ↓
內容需求
        ↓
內容包
文章 / 短文 / 留言版本 / 未來影片
        ↓
熱門討論搜尋
        ↓
0 Token 初篩與排序
        ↓
商品 × 討論串配對
        ↓
平台 / 分潤政策 Gate
        ↓
已核准發布任務
        ↓
鎖死 Publisher
        ↓
發布紀錄
        ↓
點擊 / 訂單 / 實際分潤
        ↓
回饋商品 / Placement / 內容版本評分
```

## 目前預計的新模組

- `TaiwanShopeeAdapter`：台灣蝦皮官方資料來源適配。
- `ProductOpportunityEngine`：商品資格、經濟性與推廣優先級。
- `AffiliateContentProfile`：分潤內容的 domain 規則與 facts contract。
- `PlacementHunter`：依商品產生搜尋意圖並尋找熱門討論。
- `PlacementScore`：熱度、時效、購買意圖、商品相關性排名。
- `AffiliatePlacementPolicy`：平台與分潤政策 gate。
- `LockedPublisher`：outbound-only、無 LLM、無 MCP、無任意 command 的發布器。
- `AffiliatePerformanceMapper`：把 Sub ID、點擊、訂單、分潤回寫到商品／Placement／內容版本。

## 可重用能力

Pantheon 只作為既有能力來源，不是本專案上游。優先評估重用：

- Writer / Reviewer 分離
- Model routing / capability check
- generation repair
- approval / validation pattern
- input hash / evidence

Pantheon 的算命內容規則、SEO 結構、站點 identity、FAQ 與 MysticPantheon-specific policy 不帶入本專案。

## 文件

- `docs/architecture.md`：架構與責任邊界
- `docs/research-sources.md`：官方文件、平台規則、內部與開源 Prior Art 的研究來源索引
- `docs/reuse-map.md`：既有模組重用決策
- `docs/prior-art.md`：開源 donor 與採用方式
- `docs/mvp-backlog.md`：MVP 優先級
- `contracts/README.md`：核心資料契約

## 來源治理

來源優先級固定為：

1. 平台官方條款 / Help Center / Developer Docs
2. 我們自己的既有 production code
3. 開源 repo 作 Prior Art
4. 本專案設計決策

開源實作只能證明「有人這樣做過」，不能用來證明 Shopee TW 或社群平台允許這樣做。詳細來源與未證實事項統一維護在 `docs/research-sources.md`。

## 非目標

第一版不做：

- Multi-Agent orchestration
- 自建第二套 Writer/Reviewer runtime
- 全平台社群管理 SaaS
- 大型 Dashboard
- 影片生成工廠
- 把所有能力塞進 AI Core
- 讓 Publisher 成為可對話、可呼叫工具的 Agent
