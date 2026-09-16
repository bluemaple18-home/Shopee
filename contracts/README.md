# 核心資料契約

第一版先定 contract，不綁特定 runtime 或資料庫。

> 外部欄位與平台能力的依據統一見 `docs/research-sources.md`。所有從平台取得的資料都要能區分：官方來源、人工匯入、未驗證來源；不得把開源 donor 的欄位假裝成 Shopee TW 官方欄位。

## 1. ProductSnapshot

描述商品本身在某個時間點的觀察結果。

```json
{
  "product_id": "string",
  "title": "string",
  "shop_id": "string",
  "category": "string",
  "price": 0,
  "currency": "TWD",
  "rating": 0,
  "review_count": 0,
  "sold_count": 0,
  "url": "string",
  "observed_at": "RFC3339",
  "source_method": "official_api|official_export|manual_import|unverified"
}
```

注意：`rating / review_count / sold_count` 是否能由第一版官方匯出取得，必須以實際欄位驗證；不能因 schema 有欄位就假設官方一定提供。

## 2. AffiliateOfferSnapshot

Affiliate offer 與商品分離，因為佣金與活動可能改變。Shopee TW 官方文件已確認高分潤加碼可能因賣家預算用完而即時終止，因此 freshness 是必要資料。

```json
{
  "product_id": "string",
  "commission_rate": 0,
  "estimated_commission_value": 0,
  "offer_type": "base|seller_bonus|platform_bonus|unknown",
  "availability": "active|inactive|unknown",
  "direct_order_only": false,
  "observed_at": "RFC3339",
  "refresh_after": "RFC3339|null",
  "source_method": "official_api|official_export|manual_import|unverified"
}
```

## 3. PromotionCandidate

商品機會引擎的輸出。Score 是本專案設計，不是 Shopee 官方分數。

```json
{
  "candidate_id": "string",
  "product_id": "string",
  "eligibility": "ELIGIBLE|REVIEW|REJECT",
  "profit_score": 0,
  "placement_availability_score": 0,
  "promotion_score": 0,
  "reason_codes": [],
  "input_snapshot_hash": "sha256"
}
```

## 4. ContentPack

一次生成、可多次重用的商品內容資產。

```json
{
  "content_pack_id": "string",
  "product_id": "string",
  "product_snapshot_hash": "sha256",
  "facts": {},
  "allowed_claims": [],
  "forbidden_claims": [],
  "cta_modes": ["affiliate_link", "promotion_code"],
  "variants": [
    {
      "variant_id": "reply_recommend_a",
      "type": "reply|short_post|article|video_script",
      "intent": "recommendation|comparison|qa|scenario",
      "content": "string"
    }
  ],
  "created_at": "RFC3339",
  "expires_at": "RFC3339|null"
}
```

Shopee TW 官方已提供分享推廣碼，因此 `promotion_code` 是正式支援的 CTA 類型，不是自行發明的替代方案。

## 5. DiscussionCandidate

熱門討論搜尋器抓到的候選位置。

```json
{
  "discussion_id": "string",
  "platform": "string",
  "url": "string",
  "title": "string",
  "text_excerpt": "string",
  "published_at": "RFC3339|null",
  "reactions": 0,
  "comments": 0,
  "shares": 0,
  "views": 0,
  "observed_at": "RFC3339",
  "source_method": "official_api|authorized_feed|manual|unverified_scrape",
  "source_authorization_status": "VERIFIED|UNKNOWN|PROHIBITED"
}
```

`unverified_scrape` 不得自動升級成 production source；平台未授權 scraping 時應進 `PROHIBITED` 或停用來源。

## 6. PlacementMatch

商品與討論串的配對結果。

```json
{
  "placement_match_id": "string",
  "product_id": "string",
  "discussion_id": "string",
  "relevance_score": 0,
  "intent_score": 0,
  "engagement_score": 0,
  "freshness_score": 0,
  "final_score": 0,
  "policy_decision": "ALLOW|REVIEW_REQUIRED|DENY",
  "policy_source_refs": ["string"],
  "selected_variant_id": "string|null",
  "reason_codes": []
}
```

`policy_source_refs` 用來指出平台官方規則、社群規則或內部明確政策依據，避免只保存一個無法解釋的 ALLOW/DENY。

## 7. SignedPublicationJob

Publisher 唯一允許執行的工作單位。

```json
{
  "job_id": "string",
  "platform": "string",
  "account_id": "string",
  "target_id": "string",
  "target_url": "string",
  "content_pack_id": "string",
  "variant_id": "string",
  "content_hash": "sha256",
  "cta_type": "affiliate_link|promotion_code|none",
  "affiliate_url": "string|null",
  "promotion_code": "string|null",
  "sub_ids": ["string"],
  "policy_decision": "ALLOW",
  "policy_source_refs": ["string"],
  "created_at": "RFC3339",
  "expires_at": "RFC3339",
  "idempotency_key": "string",
  "signature": "string"
}
```

硬規則：Publisher 不接受 `ALLOW` 以外的 job，也不接受任意 command、tool、callback URL 或動態 egress target。

## 8. PublicationReceipt

```json
{
  "job_id": "string",
  "idempotency_key": "string",
  "status": "SUCCESS|FAILED|UNKNOWN|PENDING",
  "attempted_at": "RFC3339",
  "published_at": "RFC3339|null",
  "remote_post_id": "string|null",
  "remote_url": "string|null",
  "error_code": "string|null",
  "content_hash": "sha256"
}
```

`PENDING` 用於 TikTok 這類可能需要非同步狀態確認的平台。

## 9. ConversionFeedback

```json
{
  "event_id": "string",
  "product_id": "string",
  "platform": "string|null",
  "placement_id": "string|null",
  "content_variant_id": "string|null",
  "campaign_id": "string|null",
  "click_id": "string|null",
  "sub_ids": ["string"],
  "order_id_hash": "string|null",
  "order_value": 0,
  "commission_value": 0,
  "status": "clicked|ordered|completed|cancelled|unknown",
  "occurred_at": "RFC3339",
  "source_method": "shopee_official_report|shopee_official_api|manual_import"
}
```

Shopee TW 官方文件已確認最多 5 個 Sub ID、點擊報告與 Excel 匯出能力；Sub ID 的五格語意由本專案定義。

## Contract 原則

- 所有外部資料要保留 `source_method` 與 `observed_at`。
- 平台規則判定要保留 `policy_source_refs`。
- 商品 facts 與 affiliate offer 分離。
- 發布工作不可攜帶任意 command / tool / callback URL。
- Receipt 不因 Publisher process 結束而消失。
- Conversion feedback 優先用官方可取得的 Sub ID 與報表資料。
- schema 有欄位不等於來源一定能提供；每個 Adapter 實作要有 field-availability matrix。
