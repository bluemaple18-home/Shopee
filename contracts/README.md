# 核心資料契約

第一版先定 contract，不綁特定 runtime 或資料庫。

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
  "source": "official_export|official_api|manual_import"
}
```

## 2. AffiliateOfferSnapshot

Affiliate offer 與商品分離，因為佣金與活動可能改變。

```json
{
  "product_id": "string",
  "commission_rate": 0,
  "estimated_commission_value": 0,
  "offer_type": "base|seller_bonus|platform_bonus|unknown",
  "availability": "active|inactive|unknown",
  "observed_at": "RFC3339",
  "refresh_after": "RFC3339|null",
  "source": "official_export|official_api|manual_import"
}
```

## 3. PromotionCandidate

商品機會引擎的輸出。

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
  "observed_at": "RFC3339"
}
```

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
  "selected_variant_id": "string|null",
  "reason_codes": []
}
```

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
  "affiliate_url": "string",
  "sub_ids": ["string"],
  "policy_decision": "ALLOW",
  "created_at": "RFC3339",
  "expires_at": "RFC3339",
  "idempotency_key": "string",
  "signature": "string"
}
```

硬規則：Publisher 不接受 `ALLOW` 以外的 job。

## 8. PublicationReceipt

```json
{
  "job_id": "string",
  "idempotency_key": "string",
  "status": "SUCCESS|FAILED|UNKNOWN",
  "attempted_at": "RFC3339",
  "published_at": "RFC3339|null",
  "remote_post_id": "string|null",
  "remote_url": "string|null",
  "error_code": "string|null",
  "content_hash": "sha256"
}
```

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
  "order_id_hash": "string|null",
  "order_value": 0,
  "commission_value": 0,
  "status": "clicked|ordered|completed|cancelled|unknown",
  "occurred_at": "RFC3339",
  "source": "shopee"
}
```

## Contract 原則

- 所有外部資料要保留 `source` 與 `observed_at`。
- 商品 facts 與 affiliate offer 分離。
- 發布工作不可攜帶任意 command / tool / callback URL。
- Receipt 不因 Publisher process 結束而消失。
- Conversion feedback 優先用官方可取得的 Sub ID 與報表資料。
