# 可重用模組地圖

本專案採 reuse-first。新能力一律依下列順序判斷：

`USE_AS_IS → CONFIGURE → WRAP → ADAPT → COPY_CODE → CUSTOM_REQUIRED`

## 1. Pantheon

### 可重用候選

- Writer / Reviewer 分離
- Model routing
- Model capability check
- Schema repair / retry
- Approval / apply 邊界
- Input hash / evidence pattern
- Validation / fail-closed 思路

### 不直接搬入

- Pantheon 文章 SEO 結構
- MysticPantheon identity
- FAQ 規則
- canonical / sitemap / JSON-LD policy
- 算命 domain disclaimer
- 特定文章字數 / 段落格式

### 採用策略

先抽 generic contract/interface；不要讓 Shopee repo import Pantheon domain policy。

## 2. AI Core

### 原則

本專案預設不新增 AI Core 能力。

只有真正跨多產品、穩定、治理價值高的 capability 才重新評估是否上收。Affiliate domain logic 永遠留在本 repo。

## 3. MarketMeNow

### 值得吸收

- Content Capsule 思路
- platform-agnostic content package
- media / publication history / derived-from
- adapter interface 概念

### 不採用

- 通用 marketing agent runtime 作為核心
- browser/cookie publisher 當我們的安全 publisher

### 分類

ADAPT

## 4. social-demand-signal-agent

### 值得吸收

- lexical intent / pain relevance
- freshness gate
- exclusion / escalation
- deterministic route

### 改造方向

從「公司需求訊號」改成「商品 × 討論 relevance」。

### 分類

ADAPT，部分演算法可接近 DIRECT_REUSE。

## 5. hidrix-tools

### 值得吸收

- engagement scoring
- time decay
- dedup / SQLite run history
- Reddit top / thread reader 等資料工具概念

### 分類

ADAPT

## 6. tody-agent/affiliate-product

### 值得吸收

- 商品 hard gate
- 多因素 Product Opportunity Score
- creator concentration / seller self-selling 等飽和度概念
- SKU / provenance / risk gate

### 不採用

- 越南市場資料
- VND / VN 設定
- FastMoss 強耦合
- 越南 Shopee / TikTok 作正式依賴

### 分類

REFERENCE / ADAPT ALGORITHM ONLY

## 7. Locked Publisher

現有 donor 可參考 platform adapter contract，但安全實作預計 CUSTOM_REQUIRED。

原因：本專案要求 outbound-only、無 inbound endpoint、無 LLM、無 MCP、無任意 command、嚴格 egress allowlist，與一般社群排程平台不同。
