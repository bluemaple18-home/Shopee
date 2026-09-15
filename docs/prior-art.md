# Prior Art 研究清單

目的不是選一個大 repo 當基座，而是拆出可用零件。

採用分類：`DIRECT_REUSE` / `ADAPT` / `REFERENCE_ONLY` / `DO_NOT_ADOPT`。

## MarketMeNow
Repo：`thearnavrustagi/marketmenow`

吸收：`ContentCapsule`、publication history、derived-from / repurpose、platform adapter contract。

判定：`ADAPT`

不採：把整套 marketing agent runtime 當本專案核心；一般 browser/cookie publisher 不作安全邊界。

## social-demand-signal-agent
Repo：`jackterror/social-demand-signal-agent`

吸收：lexical relevance、intent signal、freshness、exclusion / escalation、deterministic route。

判定：`ADAPT`

## hidrix-tools
Repo：`sonpiaz/hidrix-tools`

吸收：weighted engagement scoring、time decay、Reddit top/thread reader、SQLite dedup/run history。

判定：`ADAPT`

## affiliate-product
Repo：`tody-agent/affiliate-product`

吸收：hard eligibility gate、多因素 affiliate score、seller self-selling / KOC concentration、SKU / provenance / risk gate。

判定：`REFERENCE_ONLY`，演算法層可 `ADAPT`。

限制：該 repo 是越南市場導向，包含 VN / VND 與 FastMoss/TikTok/Shopee Vietnam 依賴。本專案不採這些市場資料，也不把它當 runtime dependency。

## Shopee Affiliate SDKs
其他市場已有 SDK 可用來理解常見物件：Offer、Product Offer、Shop Offer、Generate Short Link、Conversion Report、Click / Link metrics。

判定：`REFERENCE_ONLY`

台灣 endpoint / permission 必須以 Shopee TW 官方實際能力為準，不推定跨區 API 相同。

## Pantheon
Repo：`bluemaple18-home/Pantheon`

吸收：Writer / Reviewer 分離、model route config、capability check、schema repair、approval / apply、fail-closed validation。

判定：`ADAPT` / 內部 reuse。

不帶入：算命文章格式、Pantheon SEO / site identity、domain-specific disclaimer。

## AI Core
Repo：`bluemaple18-home/aicore`

只沿用 reuse-first 架構原則與少數真正 generic 的治理 contract。Affiliate capability 預設不進 AI Core。
