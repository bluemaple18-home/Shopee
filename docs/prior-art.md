# Prior Art 研究清單

目的不是選一個大 repo 當基座，而是拆出可用零件。

採用分類：`DIRECT_REUSE` / `ADAPT` / `REFERENCE_ONLY` / `DO_NOT_ADOPT`。

> 平台規則、Shopee TW 能力與合規判斷，以 `docs/research-sources.md` 的官方來源為準。開源 repo 只能作實作 Prior Art，不能取代官方文件。

## MarketMeNow

- Repo：https://github.com/thearnavrustagi/marketmenow
- License：MIT
- 定位：agentic outbound marketing automation。
- 已確認實作：`ContentCapsule` 可保存 modality、caption、media、thread entries、generation params、publication history 與 derived-from 關係。

吸收：
- `ContentCapsule` 的「一次生成、後續重用」資料模型
- publication history
- derived-from / repurpose
- platform adapter contract

判定：`ADAPT`

不採：
- 把整套 marketing agent runtime 當本專案核心
- browser/cookie publisher 作安全邊界
- 讓內容生成 Agent 直接擁有任意發布權限

## social-demand-signal-agent

- Repo：https://github.com/jackterror/social-demand-signal-agent
- 定位：social demand signal routing / review workflow。
- 已確認實作：lexical relevance、pain/intent term scoring、freshness、exclusion/escalation、deterministic route。

吸收：
- lexical relevance
- intent signal
- freshness
- exclusion / escalation
- deterministic route

判定：`ADAPT`

用途：作 `PlacementScore` 第一層低成本相關性篩選的 donor，不直接採它的 agent/server runtime。

## hidrix-tools

- Repo：https://github.com/sonpiaz/hidrix-tools
- 已確認實作：`content_scorer` 以 reactions/comments/shares 加權，再乘 time-decay 排名；可作純計算，不需要外部 LLM。

吸收：
- weighted engagement scoring
- time decay
- Reddit top/thread reader pattern
- SQLite dedup / run history

判定：`ADAPT`

用途：熱門討論 ranking 與 0-token preprocessing。

## affiliate-product

- Repo：https://github.com/tody-agent/affiliate-product
- License：MIT
- 定位：Shopee / TikTok Shop / FastMoss 的 affiliate marketplace intelligence；原專案以越南市場為中心。

吸收：
- hard eligibility gate
- 多因素 affiliate score
- seller self-selling / KOC concentration 思考
- SKU / provenance / risk gate
- `Offer freshness / evidence` 思維

判定：`REFERENCE_ONLY`，演算法層可 `ADAPT`。

限制：
- repo 內明確存在 VN / VND 與越南市場設定
- FastMoss / TikTok Shop / Shopee Vietnam 不作本專案正式 dependency
- 不 fork 作基座
- 不使用其市場 threshold 當台灣 threshold

## Shopee Affiliate SDKs（其他市場）

其他市場已有 SDK 可用來理解常見物件，例如：
- Offer
- Product Offer
- Shop Offer
- Generate Short Link
- Conversion Report
- Click / Link metrics

判定：`REFERENCE_ONLY`

台灣 endpoint、permission、rate limit 與可用欄位必須以 Shopee TW 官方實際能力為準，不推定跨區 API 相同。

## Pantheon

- Repo：https://github.com/bluemaple18-home/Pantheon
- 主要 donor：`scripts/agy_seo_copy_pipeline.py`

已確認可吸收機制：
- Writer / Reviewer 分離
- model route config 與 role isolation
- model capability check
- schema repair
- approval / apply
- fail-closed validation
- input hash / evidence pattern

判定：`ADAPT` / 內部 reuse。

不帶入：
- 算命文章格式
- Pantheon SEO / canonical / sitemap / JSON-LD 規則
- MysticPantheon site identity
- 算命 disclaimer
- FAQ 與 Pantheon-specific presentation constraints

## AI Core

- Repo：https://github.com/bluemaple18-home/aicore

只沿用：
- reuse-first 原則
- existing seam first
- 少數真正 generic 的治理 contract

判定：`REFERENCE_ONLY` / architecture principle。

Affiliate capability 預設不進 AI Core。只有跨多個獨立產品、contract 已穩定、治理價值很高，而且 shared package 無法合理承擔時，才重新評估。

---

# Prior Art 使用規則

1. `DIRECT_REUSE`：依授權可直接依賴且責任邊界吻合。
2. `ADAPT`：吸收 schema / algorithm / contract，但保留本專案自己的 domain boundary。
3. `REFERENCE_ONLY`：只作研究依據，不形成 runtime dependency。
4. `DO_NOT_ADOPT`：跟安全、政策、成本或架構方向衝突。
5. 所有 copy / reuse 前都要再次確認 license 與版本；README 宣稱不是授權證據的替代品。
