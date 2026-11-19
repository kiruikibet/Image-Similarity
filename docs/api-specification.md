# REST API Specification (v1)

Base URL: `http://localhost:8000/api/v1` (Production: `https://api.nexora.security/api/v1`)

---

## 1. Authentication
* **Platform Authorization:** Pass API Key in header:
  ```http
  X-Nexora-Key: nex_live_8f0a3e817bc24c5b90d23f114c
  ```
* **Moderator Dashboard Authorization:** Pass standard JWT Bearer token:
  ```http
  Authorization: Bearer <jwt_access_token>
  ```

---

## 2. API Endpoints Catalog

### 2.1 Register Platform Client
Registers a participating e-commerce store and generates cryptographic credentials.
* **Method & Path:** `POST /platforms/register/`
* **Access:** Admin only
* **Request Payload:**
```json
{
  "name": "Nairobi Electronics Market",
  "webhook_url": "https://nairobielectronics.co.ke/wp-json/nexora/v1/webhook"
}
```
* **Response (`201 Created`):**
```json
{
  "platform_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
  "name": "Nairobi Electronics Market",
  "api_key": "nex_live_8f0a3e817bc24c5b90d23f114c",
  "status": "ACTIVE",
  "created_at": "2026-11-19T10:00:00Z"
}
```

---

### 2.2 Screen Listing for Fraud (Real-Time)
Submits a complete product listing with imagery for immediate multi-signal evaluation.
* **Method & Path:** `POST /listings/analyze/`
* **Content-Type:** `multipart/form-data`
* **Headers:** `X-Nexora-Key: <api_key>`
* **Form Parameters:**
  * `external_id`: `string` (e.g. `"wc_prod_5021"`)
  * `seller_account_ref`: `string` (e.g. `"merchant_49"`)
  * `seller_email`: `string` (optional)
  * `title`: `string` (e.g. `"Sony WH-1000XM5 Wireless Headphones"`)
  * `description`: `string` (e.g. `"Brand new unopened original box with receipt..."`)
  * `price`: `number` (e.g. `12500.00`)
  * `currency`: `string` (e.g. `"KES"`)
  * `category`: `string` (e.g. `"Audio & Electronics"`)
  * `images`: `file[]` (1 to 10 image files, JPEG/PNG/WebP)

* **Response (`200 OK` / `201 Created`):**
```json
{
  "listing_id": "8f3b1a20-41be-4d56-b08e-9c24efb14902",
  "external_id": "wc_prod_5021",
  "status": "FLAGGED",
  "assessment": {
    "assessment_id": "e3a479b1-50de-4411-9a7c-bc3e74811a43",
    "risk_score": 84.2,
    "risk_level": "HIGH",
    "recommendation": "MANUAL_REVIEW",
    "signal_summary": {
      "phash_duplicate_score": 0.0,
      "deep_vector_similarity": 0.92,
      "forensic_manipulation": 0.70,
      "text_consistency": 0.28,
      "seller_behavior_risk": 0.85
    },
    "evidence": [
      {
        "signal_type": "VECTOR_SIMILARITY",
        "confidence": 0.92,
        "matched_listing_id": "22bc9910-33de-4011-88fc-1209bcfa4410",
        "details": {
          "model": "ResNet50_FAISS",
          "cosine_similarity": 0.921,
          "source_platform": "Kilimani Electronics",
          "note": "Near-duplicate cropped image found on another platform."
        }
      },
      {
        "signal_type": "TEXT_MISMATCH",
        "confidence": 0.72,
        "details": {
          "clip_similarity": 0.28,
          "note": "Visual content does not match title and audio specs."
        }
      },
      {
        "signal_type": "PRICING_ANOMALY",
        "confidence": 0.85,
        "details": {
          "category_median": 42000.00,
          "offered_price": 12500.00,
          "deviation_percentage": -70.2,
          "note": "Price is 70% below category benchmark."
        }
      }
    ],
    "evaluated_at": "2026-11-19T10:00:00Z",
    "latency_ms": 385
  }
}
```

---

### 2.3 Retrieve Moderation Queue
Fetches pending flagged listings sorted by composite risk score descending.
* **Method & Path:** `GET /moderation/queue/?status=FLAGGED&limit=20`
* **Headers:** `Authorization: Bearer <jwt_moderator_token>`
* **Response (`200 OK`):**
```json
{
  "count": 14,
  "results": [
    {
      "listing_id": "8f3b1a20-41be-4d56-b08e-9c24efb14902",
      "platform_name": "Nairobi Electronics Market",
      "title": "Sony WH-1000XM5 Wireless Headphones",
      "price": 12500.00,
      "currency": "KES",
      "risk_score": 84.2,
      "risk_level": "HIGH",
      "primary_image_url": "/media/listings/8f3b1a20/img1.jpg",
      "created_at": "2026-11-19T10:00:00Z"
    }
  ]
}
```

---

### 2.4 Record Moderation Decision
Audits a human moderator's finding and automatically updates the shared intelligence repository if confirmed fraud.
* **Method & Path:** `POST /moderation/decide/`
* **Headers:** `Authorization: Bearer <jwt_moderator_token>`
* **Request Payload:**
```json
{
  "assessment_id": "e3a479b1-50de-4411-9a7c-bc3e74811a43",
  "outcome": "CONFIRMED_FRAUD",
  "notes": "Verified image was stolen from another seller and cropped; price is an extreme outlier.",
  "add_to_shared_intelligence": true
}
```
* **Response (`200 OK`):**
```json
{
  "decision_id": "58c30b22-92a1-4ee1-b0f3-8b7c3d2e1a90",
  "status": "RECORDED",
  "intel_records_created": 1,
  "decided_at": "2026-11-19T10:05:00Z"
}
```

---

### 2.5 Query Shared Fraud Fingerprint
Allows participating stores to check if an image hash has been flagged as confirmed fraud.
* **Method & Path:** `GET /intelligence/lookup/?phash=a1f0c3d4e8b91234`
* **Headers:** `X-Nexora-Key: <api_key>`
* **Response (`200 OK`):**
```json
{
  "match_found": true,
  "confidence": 1.0,
  "records": [
    {
      "record_id": "33b2a9f1-44de-4a92-8012-6b9f1d0a2c34",
      "fraud_category": "STOLEN_PHOTO",
      "confirmed_at": "2026-11-15T08:30:00Z"
    }
  ]
}
```
