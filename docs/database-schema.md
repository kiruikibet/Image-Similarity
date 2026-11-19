# Database Schema & Entity-Relationship Specification

This document details the complete relational data models, fields, types, constraints, and relationships for the **Image Similarity** fraud detection system.

---

## 1. Entity-Relationship Diagram (ERD)

```mermaid
erDiagram
    PLATFORM ||--o{ SELLER : registers
    SELLER ||--o{ LISTING : owns
    LISTING ||--o{ LISTING_IMAGE : contains
    LISTING ||--o{ RISK_ASSESSMENT : evaluated_by
    RISK_ASSESSMENT ||--o{ EVIDENCE : justified_by
    RISK_ASSESSMENT ||--o| MODERATION_DECISION : reviewed_as
    LISTING_IMAGE ||--o| FRAUD_INTEL_RECORD : flagged_as

    PLATFORM {
        uuid platform_id PK
        string name
        string api_key UK
        string webhook_url
        string status
        datetime created_at
    }

    SELLER {
        uuid seller_id PK
        uuid platform_id FK
        string account_ref
        string email
        float trust_score
        datetime registered_at
    }

    LISTING {
        uuid listing_id PK
        uuid seller_id FK
        string external_id
        string title
        text description
        decimal price
        string currency
        string category
        string status
        datetime created_at
        datetime updated_at
    }

    LISTING_IMAGE {
        uuid image_id PK
        uuid listing_id FK
        string file_path
        string phash
        integer embedding_id
        datetime uploaded_at
    }

    RISK_ASSESSMENT {
        uuid assessment_id PK
        uuid listing_id FK
        float risk_score
        string risk_level
        jsonb signal_summary
        string recommendation
        datetime created_at
    }

    EVIDENCE {
        uuid evidence_id PK
        uuid assessment_id FK
        string signal_type
        float confidence
        uuid matched_image_id FK
        jsonb details
        datetime created_at
    }

    MODERATION_DECISION {
        uuid decision_id PK
        uuid assessment_id FK
        integer moderator_id FK
        string outcome
        text notes
        datetime decided_at
    }

    FRAUD_INTEL_RECORD {
        uuid record_id PK
        uuid image_id FK
        string phash
        uuid source_platform_id FK
        string fraud_category
        datetime confirmed_at
    }
```

---

## 2. Table Specifications

### 2.1 Table: `platforms`
Client stores and marketplaces authorized to connect with the fraud screening service.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `platform_id` | UUID | Primary Key | Unique platform identifier. |
| `name` | VARCHAR(150) | Not Null | Display name (e.g. "Mombasa Tech Hub"). |
| `api_key` | VARCHAR(64) | Unique, Indexed | Hashed secret key for API requests. |
| `webhook_url` | VARCHAR(500) | Nullable | Asynchronous callback URL. |
| `status` | VARCHAR(20) | Default: `'ACTIVE'` | `'ACTIVE'`, `'SUSPENDED'`, `'INACTIVE'`. |
| `created_at` | TIMESTAMPTZ | Default: `now()` | Registration timestamp. |

---

### 2.2 Table: `sellers`
Seller accounts operating within participating platforms.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `seller_id` | UUID | Primary Key | Internal UUID identifier. |
| `platform_id` | UUID | Foreign Key (`platforms`) | Associated platform. |
| `account_ref` | VARCHAR(120) | Composite Unique | Store-specific merchant ID. |
| `email` | VARCHAR(255) | Nullable | Merchant contact email. |
| `trust_score` | FLOAT | Default: `1.0` (0.0 to 1.0) | Historical reliability index. |
| `registered_at` | TIMESTAMPTZ | Default: `now()` | Platform onboarding timestamp. |

---

### 2.3 Table: `listings`
Product listings screened for fraudulent content.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `listing_id` | UUID | Primary Key | Unique listing record identifier. |
| `seller_id` | UUID | Foreign Key (`sellers`) | Merchant offering the item. |
| `external_id` | VARCHAR(120) | Indexed | Source store product ID. |
| `title` | VARCHAR(255) | Not Null | Product advertisement title. |
| `description` | TEXT | Nullable | Full product description copy. |
| `price` | DECIMAL(12,2) | Not Null | Unit price offered. |
| `currency` | VARCHAR(10) | Default: `'KES'` | ISO currency code. |
| `category` | VARCHAR(100) | Indexed | Product classification. |
| `status` | VARCHAR(20) | Default: `'PENDING'` | `'PENDING'`, `'APPROVED'`, `'FLAGGED'`, `'REJECTED'`. |
| `created_at` | TIMESTAMPTZ | Default: `now()` | Submission timestamp. |
| `updated_at` | TIMESTAMPTZ | Default: `now()` | Last modification timestamp. |

---

### 2.4 Table: `listing_images`
Product imagery indexed with perceptual hashes and vector embeddings.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `image_id` | UUID | Primary Key | Unique image UUID. |
| `listing_id` | UUID | Foreign Key (`listings`) | Associated listing record. |
| `image_file` | VARCHAR(500) | Nullable | Local disk or object storage path. |
| `file_path` | VARCHAR(500) | Nullable | External URL if hotlinked. |
| `phash` | VARCHAR(64) | Indexed | 64-bit hexadecimal pHash fingerprint. |
| `dhash` | VARCHAR(64) | Indexed | 64-bit difference hash. |
| `embedding_id` | INTEGER | Indexed | Vector index row pointer in FAISS. |
| `uploaded_at` | TIMESTAMPTZ | Default: `now()` | Image ingest timestamp. |

---

### 2.5 Table: `risk_assessments`
Composite multi-signal fraud evaluations.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `assessment_id` | UUID | Primary Key | Unique evaluation UUID. |
| `listing_id` | UUID | Foreign Key (`listings`) | Target listing screened. |
| `risk_score` | FLOAT | Range $0.0 - 100.0$ | Aggregate composite risk rating. |
| `risk_level` | VARCHAR(10) | Indexed | `'LOW'`, `'MEDIUM'`, `'HIGH'`. |
| `signal_summary` | JSONB | Default: `{}` | Breakdown of individual module outputs. |
| `recommendation` | VARCHAR(30) | Not Null | `'APPROVE'`, `'MANUAL_REVIEW'`, `'AUTO_HOLD'`. |
| `created_at` | TIMESTAMPTZ | Default: `now()` | Evaluation completion timestamp. |

---

### 2.6 Table: `evidence`
Individual diagnostic signals justifying an assessment.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `evidence_id` | UUID | Primary Key | Unique evidence item identifier. |
| `assessment_id` | UUID | Foreign Key (`risk_assessments`) | Parent assessment. |
| `signal_type` | VARCHAR(50) | Indexed | e.g. `'VECTOR_SIMILARITY'`, `'TEXT_MISMATCH'`. |
| `confidence` | FLOAT | Range $0.0 - 1.0$ | Signal severity / confidence rating. |
| `matched_image_id`| UUID | Foreign Key (`listing_images`) | Matched duplicate image if applicable. |
| `details` | JSONB | Default: `{}` | Diagnostic metadata (e.g. cosine distance, ELA ratio). |
| `created_at` | TIMESTAMPTZ | Default: `now()` | Logging timestamp. |

---

### 2.7 Table: `moderation_decisions`
Audit trail of human moderator reviews.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `decision_id` | UUID | Primary Key | Unique decision UUID. |
| `assessment_id` | UUID | One-to-One (`risk_assessments`) | Assessed case under review. |
| `moderator_id` | INTEGER | Foreign Key (`auth_user`) | Reviewing human moderator. |
| `outcome` | VARCHAR(30) | Not Null | `'CONFIRMED_FRAUD'`, `'DISMISSED_LEGITIMATE'`, `'ESCALATED'`. |
| `notes` | TEXT | Nullable | Moderator rationale notes. |
| `decided_at` | TIMESTAMPTZ | Default: `now()` | Final verdict timestamp. |

---

### 2.8 Table: `fraud_intel_records`
Cross-platform shared repository of confirmed fraud fingerprints.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `record_id` | UUID | Primary Key | Unique intelligence record ID. |
| `image_id` | UUID | Foreign Key (`listing_images`) | Source image if stored in Nexora. |
| `phash` | VARCHAR(64) | Indexed | Perceptual hash of confirmed fraudulent image. |
| `source_platform_id`| UUID | Foreign Key (`platforms`) | Platform that reported the confirmed fraud. |
| `fraud_category` | VARCHAR(50) | Default: `'STOLEN_PHOTO'` | `'STOLEN_PHOTO'`, `'COUNTERFEIT'`, `'SCAM'`. |
| `notes` | VARCHAR(255) | Nullable | Brief non-PII case summary. |
| `confirmed_at` | TIMESTAMPTZ | Default: `now()` | Confirmation timestamp. |
