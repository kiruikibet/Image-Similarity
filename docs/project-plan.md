# Project Plan & 50-Day Engineering Roadmap

This document outlines the master execution plan for developing the **Image Similarity** fraud detection system over **10 Weeks (50 Working Days)** following the **Iterative and Incremental Development Model** (Sommerville, 2016).

---

## 1. High-Level Iteration Phases

```
┌────────────────────────────────────────────────────────────────────────┐
│ Phase 1: Foundation & Domain Modeling (Weeks 1–2 / Days 1–10)          │
│ Architecture, Environment, Database Schemas & Test Datasets            │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼────────────────────────────────────┐
│ Phase 2: Vision & Similarity Pipeline (Weeks 3–4 / Days 11–20)         │
│ Preprocessing, pHash, Deep Embeddings (PyTorch) & FAISS Vector Index   │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼────────────────────────────────────┐
│ Phase 3: Multi-Modal Analysis & Intelligence (Weeks 5–6 / Days 21–30)  │
│ Image Forensics (ELA), CLIP Consistency & Seller Behavior Analytics    │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼────────────────────────────────────┐
│ Phase 4: Risk Engine, APIs & Dashboards (Weeks 7–8 / Days 31–40)       │
│ Multi-Signal Risk Scorer, REST APIs, WooCommerce Plugin & React UI     │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼────────────────────────────────────┐
│ Phase 5: Demo Store, Testing & Evaluation (Weeks 9–10 / Days 41–50)    │
│ Marketplace Simulation, Quantitative Benchmarking & Final Academic Doc │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Milestone Gates

| Milestone | Target Date | Milestone Goal | Verification Gate |
|---|---|---|---|
| **Gate 1** | End of Week 2 (Day 10) | Foundation & Synthetic Dataset | Database migrations applied, 1,500+ perturbed test images generated. |
| **Gate 2** | End of Week 4 (Day 20) | Visual Similarity Pipeline | Combined pHash + FAISS query latency $< 150\text{ms}$; precision $\ge 92\%$. |
| **Gate 3** | End of Week 6 (Day 30) | Multi-Signal Analytics | Forensics, CLIP consistency, velocity, and shared intelligence verified. |
| **Gate 4** | End of Week 8 (Day 40) | End-to-End System & UI | Real-time REST API, WooCommerce plugin, and React moderator dashboard active. |
| **Gate 5** | End of Week 10 (Day 50) | Evaluation & Final Defense | Precision $\ge 99\%$, Recall $\ge 88\%$, ROC-AUC $\ge 0.90$, final project documentation. |

---

## 3. Daily Target Breakdown (Day 1 to Day 50)

### Phase 1: Foundation, Environment & Domain Modeling (Week 1–2)

#### Week 1: Foundation & Base Domain
* **Day 1: Project Setup & Version Control Blueprint**
  * *Tasks:* Initialize Git repository, configure branching model, commit standards, `.gitignore`, and `.editorconfig`.
  * *Deliverable:* Project directory skeleton and base configuration files.
  * *Acceptance Criteria:* Git status clean; zero stray files.
* **Day 2: Backend Architecture & Environment Setup**
  * *Tasks:* Configure Python 3.12 environment, structure `requirements.txt` (Core, AI, Testing). Set up base Django project.
  * *Deliverable:* `backend/manage.py`, `backend/config/settings.py`.
  * *Acceptance Criteria:* `python manage.py check` executes with 0 errors.
* **Day 3: Authentication & Role-Based Access Control (RBAC)**
  * *Tasks:* Implement `accounts` app. Configure JWT authentication for `Moderator`, `Platform Admin`, and `Store Owner`.
  * *Deliverable:* `accounts/models.py`, `accounts/views.py`.
  * *Acceptance Criteria:* Valid JWT token returns role claims; automated test passes.
* **Day 4: Marketplace Domain Models (Platform & Seller)**
  * *Tasks:* Implement `Platform` (with unique API keys) and `Seller` (with account references and trust scores) models.
  * *Deliverable:* `marketplace/models.py` (Platform, Seller).
  * *Acceptance Criteria:* Migrations apply cleanly; platforms created with unique keys.
* **Day 5: Listing & Image Storage Models**
  * *Tasks:* Implement `Listing` (title, description, price, category, status) and `ListingImage` (file, pHash, embedding_id) models.
  * *Deliverable:* `marketplace/models.py` (Listing, ListingImage).
  * *Acceptance Criteria:* Multi-image listings can be created and queried via Django ORM.

#### Week 2: Data Engineering & Perturbation Pipeline
* **Day 6: Image Dataset Acquisition & Structuring**
  * *Tasks:* Gather 500+ legitimate product images across electronics, apparel, and footwear from open-source datasets.
  * *Deliverable:* `evaluation/datasets/raw/` organized by product category.
  * *Acceptance Criteria:* Corrupt images stripped, minimum resolution $300 \times 300$.
* **Day 7: Image Perturbation Generator (Cropping & Resizing)**
  * *Tasks:* Build script to generate modified copies using bounding-box cropping ($10\% - 40\%$) and aspect-ratio scaling.
  * *Deliverable:* `evaluation/perturbation_generator.py` (crop/resize).
  * *Acceptance Criteria:* Script generates paired crops and maintains an evaluation mapping JSON.
* **Day 8: Image Perturbation Generator (Compression, Noise & Rotation)**
  * *Tasks:* Extend perturbation script to apply JPEG compression ($Q=30, 50, 70$), rotations ($\pm 5^\circ, 15^\circ$), and noise.
  * *Deliverable:* Perturbation transforms for compression, rotation, and color jitter.
  * *Acceptance Criteria:* 1,500+ synthetic test variants generated with transformation metadata.
* **Day 9: Synthetic Text & Seller Listing Data Generator**
  * *Tasks:* Create script to simulate realistic listings, pairing legitimate images with matching descriptions, and generating deceptive mismatched listings.
  * *Deliverable:* `evaluation/synthetic_data_generator.py`.
  * *Acceptance Criteria:* Labelled synthetic dataset containing 1,000 listings (70% legitimate, 30% fraudulent).
* **Day 10: Week 1–2 Review & Milestone Gate 1**
  * *Tasks:* Run full test suite on database models, verify dataset integrity, document baseline statistics.
  * *Deliverable:* Milestone 1 Report: Validated domain models, working test dataset.
  * *Acceptance Criteria:* All Django tests pass; datasets ready for algorithmic development.

---

### Phase 2: Vision & Similarity Pipeline (Week 3–4: Iteration 1)

#### Week 3: Preprocessing & Perceptual Hashing (pHash)
* **Day 11: Image Preprocessing Service**
  * *Tasks:* Implement `ImagePreprocessor` class to normalize dimensions ($224 \times 224$), convert color spaces (RGBA $\to$ RGB), and handle file streams.
  * *Deliverable:* `backend/apps/fraud_engine/services/preprocessing.py`.
  * *Acceptance Criteria:* Processes multiple formats (JPEG, PNG, WebP) into standardized arrays.
* **Day 12: Discrete Cosine Transform Perceptual Hashing (pHash)**
  * *Tasks:* Implement 64-bit DCT perceptual hash algorithm with fallback support for standard average hashing (aHash).
  * *Deliverable:* `backend/apps/fraud_engine/services/phash_service.py`.
  * *Acceptance Criteria:* Produces consistent 16-character hexadecimal hash for identical and near-identical images.
* **Day 13: Difference Hashing (dHash) & Hamming Distance Metrics**
  * *Tasks:* Implement dHash and fast bitwise Hamming distance calculator: $\text{distance}(h_1, h_2) = \text{popcount}(h_1 \oplus h_2)$.
  * *Deliverable:* `calculate_hamming_distance()` method and dual-hash fingerprinting.
  * *Acceptance Criteria:* Identical images return distance $0$; minor crops return distance $\le 8$.
* **Day 14: Perceptual Hash Database Query & Indexing**
  * *Tasks:* Implement lookup service to search stored listing image hashes for Hamming distance $\le \tau$ (where $\tau = 10$).
  * *Deliverable:* Hash query service querying existing database images.
  * *Acceptance Criteria:* Sub-10ms lookup over 5,000 stored hashes.
* **Day 15: Perceptual Hashing Unit Testing & Robustness**
  * *Tasks:* Write automated tests measuring pHash survival against resizing, compression, and minor crops.
  * *Deliverable:* `backend/tests/test_phash.py`.
  * *Acceptance Criteria:* $\ge 95\%$ duplicate detection rate for compressed/resized variants; tests pass in CI.

#### Week 4: Deep Learning Embeddings & Vector Search (FAISS)
* **Day 16: Deep Feature Extractor Pipeline (ResNet/CLIP)**
  * *Tasks:* Integrate pre-trained vision backbone (PyTorch ResNet-50 / CLIP visual model) to extract dense 512-dimensional normalized feature vectors.
  * *Deliverable:* `backend/apps/fraud_engine/services/embedding_service.py`.
  * *Acceptance Criteria:* Extracted vectors are unit-normalized ($\|v\|_2 = 1.0$).
* **Day 17: FAISS Vector Store Integration (IndexFlatIP)**
  * *Tasks:* Implement FAISS vector store service utilizing Inner Product (`IndexFlatIP`) for ultra-fast cosine similarity computation.
  * *Deliverable:* `backend/apps/fraud_engine/services/vector_store.py`.
  * *Acceptance Criteria:* Sub-millisecond $k$-NN search over 10,000 vectors.
* **Day 18: Vector Persistence & Incremental Synchronization**
  * *Tasks:* Implement index disk serialization (`faiss.write_index`) and incremental vector insertion synchronized with Django ORM `image_id` mapping.
  * *Deliverable:* FAISS index loading, saving, and thread-safe dynamic vector insertion.
  * *Acceptance Criteria:* Persisted vector index survives server restart without data loss.
* **Day 19: Deep Similarity vs. pHash Fusion for Modified Copies**
  * *Tasks:* Build unified visual similarity matcher combining pHash (tier 1: fast exact match) with FAISS embedding cosine similarity (tier 2: complex crops/rotations).
  * *Deliverable:* `find_visual_duplicates()` method returning matched image IDs and similarity scores.
  * *Acceptance Criteria:* Detects cropped images where pHash failed, achieving $\ge 90\%$ recall on transformed test set.
* **Day 20: Week 3–4 Review & Milestone Gate 2**
  * *Tasks:* Benchmark pHash + FAISS pipeline on 1,500 synthetic transformed images. Measure precision, recall, and query latency.
  * *Deliverable:* Milestone 2 Benchmark Report: Visual similarity pipeline verified.
  * *Acceptance Criteria:* Combined visual similarity latency $< 150\text{ms}$ per image; precision $\ge 92\%$.

---

### Phase 3: Multi-Modal Analysis & Fraud Intelligence (Week 5–6: Iteration 2)

#### Week 5: Digital Image Forensics & Image-Text Consistency
* **Day 21: EXIF Metadata & Integrity Analysis**
  * *Tasks:* Implement forensic service parsing EXIF headers: check camera make/model, identify editing software signatures (Photoshop, Canva, GIMP), detect stripped metadata.
  * *Deliverable:* `backend/apps/fraud_engine/services/forensics_service.py` (EXIF parser).
  * *Acceptance Criteria:* Identifies software tags and flags completely sanitized metadata.
* **Day 22: Error Level Analysis (ELA) for JPEG Compression Artifacts**
  * *Tasks:* Implement Error Level Analysis (ELA) by re-saving image at standard quality ($Q=90$), calculating pixel-difference matrix, and computing compression inconsistency ratios.
  * *Deliverable:* `compute_ela_metrics()` within `forensics_service.py`.
  * *Acceptance Criteria:* Generates ELA difference score highlighting digitally spliced regions.
* **Day 23: Multimodal Image-Text Feature Extraction (CLIP)**
  * *Tasks:* Implement multimodal text-image alignment service utilizing CLIP tokenizer and text encoder.
  * *Deliverable:* `backend/apps/fraud_engine/services/consistency_service.py`.
  * *Acceptance Criteria:* Generates text embedding vectors aligned with the image feature space.
* **Day 24: Cross-Modal Semantic Consistency Scoring**
  * *Tasks:* Compute cosine similarity between listing text (title + description) and image vector. Calibrate semantic thresholds for deceptive mismatches.
  * *Deliverable:* `evaluate_text_image_consistency()` returning score ($0.0 - 1.0$) and mismatch warning.
  * *Acceptance Criteria:* Correctly flags severe mismatches (e.g. photo of shoes with title "MacBook Pro M3").
* **Day 25: Forensics & Consistency Unit Testing**
  * *Tasks:* Test ELA and CLIP consistency services across 200 legitimate and 100 deceptive listing pairs.
  * *Deliverable:* `backend/tests/test_forensics_consistency.py`.
  * *Acceptance Criteria:* Mismatch detector achieves $\ge 85\%$ accuracy on deliberate semantic discrepancies.

#### Week 6: Seller Behavior Analytics & Shared Fraud Intelligence
* **Day 26: Seller Listing Velocity & Burst Analyzer**
  * *Tasks:* Implement behavioral analytics service tracking listing creation frequency per seller (1-hour and 24-hour windows).
  * *Deliverable:* `backend/apps/fraud_engine/services/behavior_service.py` (velocity module).
  * *Acceptance Criteria:* Flags suspicious burst behavior (e.g. $> 10$ high-value listings within 1 hour).
* **Day 27: Category Price Anomaly & Outlier Detector**
  * *Tasks:* Calculate rolling category price distributions (mean, median, standard deviation). Compute $Z$-score for incoming listing prices.
  * *Deliverable:* `detect_price_anomaly()` within `behavior_service.py`.
  * *Acceptance Criteria:* Flags listings priced $> 65\%$ below category median.
* **Day 28: Cross-Account Image Recycling Detector**
  * *Tasks:* Check if an uploaded image hash was previously associated with a different seller account or platform.
  * *Deliverable:* Cross-seller image collision detection query.
  * *Acceptance Criteria:* Detects image reuse across distinct seller accounts in $< 15\text{ms}$.
* **Day 29: Shared Fraud Intelligence Repository**
  * *Tasks:* Implement `intelligence` app and `FraudIntelligenceRecord` model. Create fast lookup for confirmed fraudulent hashes.
  * *Deliverable:* `backend/apps/intelligence/models.py`, `backend/apps/intelligence/services.py`.
  * *Acceptance Criteria:* Allows participating platforms to check if an image hash has been blacklisted as confirmed fraud.
* **Day 30: Week 5–6 Review & Milestone Gate 3**
  * *Tasks:* Validate forensics, text consistency, seller behavior, and shared intelligence modules.
  * *Deliverable:* Milestone 3 Report: All individual fraud signals fully operational.
  * *Acceptance Criteria:* All four analytical engines produce calibrated scores between 0.0 and 1.0.

---

### Phase 4: Risk Engine, REST API & Interfaces (Week 7–8: Iteration 3 & 4)

#### Week 7: Explainable Risk Engine & REST API Endpoints
* **Day 31: Multi-Signal Risk Aggregation Formula**
  * *Tasks:* Implement `RiskAggregator` combining visual similarity ($w_1=0.35$), shared fraud blacklist ($w_2=0.25$), seller behavior ($w_3=0.15$), text-image consistency ($w_4=0.15$), and forensics ($w_5=0.10$).
  * *Deliverable:* `backend/apps/fraud_engine/services/risk_aggregator.py`.
  * *Acceptance Criteria:* Generates calibrated risk score ($0.0 - 100.0$) and categorizes into `LOW` ($<35$), `MEDIUM` ($35-70$), and `HIGH` ($>70$).
* **Day 32: Structured Explainable Evidence Generation**
  * *Tasks:* Implement evidence generator creating human-readable diagnostic records for every flag (matched image links, similarity percentages, ELA ratios, price deviation stats).
  * *Deliverable:* Evidence building methods writing to `Evidence` model.
  * *Acceptance Criteria:* Every flagged assessment includes at least one structured evidence item explaining why it was flagged.
* **Day 33: Core Real-Time Screening API (`/api/v1/listings/analyze/`)**
  * *Tasks:* Implement DRF endpoint accepting `multipart/form-data` listing payloads, orchestrating the AI pipeline, and returning real-time risk assessment JSON.
  * *Deliverable:* `backend/apps/fraud_engine/api/views.py` and `serializers.py`.
  * *Acceptance Criteria:* End-to-end request executes within $< 500\text{ms}$ returning full JSON risk payload.
* **Day 34: Moderation Queue & Decision API Endpoints**
  * *Tasks:* Implement `/api/v1/moderation/queue/` (sorted by risk score) and `POST /api/v1/moderation/decide/` (records verdict: `CONFIRMED_FRAUD`, `DISMISSED_LEGITIMATE`, `ESCALATED`).
  * *Deliverable:* Moderation API views and URL routing.
  * *Acceptance Criteria:* When fraud is confirmed, image pHash is automatically added to the Shared Fraud Intelligence repository.
* **Day 35: API Security, Key Verification & Rate Limiting**
  * *Tasks:* Implement `PlatformKeyAuthentication` middleware, request throttling, and input validation schemas.
  * *Deliverable:* Security middleware and API permission classes.
  * *Acceptance Criteria:* Unauthorized requests return `401 Unauthorized`; malformed images return `400 Bad Request`.

#### Week 8: WooCommerce Plugin & React Moderator Dashboard
* **Day 36: WooCommerce Plugin Skeleton & Settings Page**
  * *Tasks:* Create WordPress plugin `nexora-woocommerce.php`. Build admin settings page for entering Nexora API URL and Platform API Key.
  * *Deliverable:* `plugin/woocommerce/nexora-woocommerce.php` (admin configuration).
  * *Acceptance Criteria:* Plugin activates cleanly in WordPress without errors; settings save to `wp_options`.
* **Day 37: WooCommerce Listing Event Hook Integration**
  * *Tasks:* Hook into `woocommerce_new_product` and `woocommerce_update_product`. Serialize product title, price, description, and images; dispatch asynchronous `wp_remote_post` to Nexora API.
  * *Deliverable:* WooCommerce action hook dispatcher.
  * *Acceptance Criteria:* Saving a product in WooCommerce triggers API call and logs the returned risk score.
* **Day 38: WooCommerce Admin Fraud Badges & Meta-Box**
  * *Tasks:* Render risk status badge (`Low Risk`, `Flagged for Review`) in WooCommerce product list table and display evidence meta-box on product edit screen.
  * *Deliverable:* WooCommerce admin UI hooks and badge renderer.
  * *Acceptance Criteria:* Store manager sees visual fraud indicator directly inside WooCommerce admin.
* **Day 39: React Moderator Dashboard (Queue & Metrics View)**
  * *Tasks:* Build React dashboard showing flagged listings table, risk score distribution, filterable by platform, risk level, and date.
  * *Deliverable:* `frontend/src/pages/DashboardPage.jsx`, metric summary cards.
  * *Acceptance Criteria:* Dashboard loads pending flagged listings dynamically from backend API.
* **Day 40: React Side-by-Side Visual Evidence Inspector**
  * *Tasks:* Implement interactive modal/page showing uploaded listing image side-by-side with matched duplicate image, highlighting similarity %, forensic flags, and action buttons (`Confirm Fraud`, `Dismiss`).
  * *Deliverable:* `frontend/src/pages/ReviewCasePage.jsx` and `EvidenceViewer.jsx`.
  * *Acceptance Criteria:* Moderator can inspect visual comparison, submit decision, and see item removed from active queue.

---

### Phase 5: Demo Store, Testing, Benchmarks & Evaluation (Week 9–10)

#### Week 9: Demonstration Marketplace & End-to-End System Integration
* **Day 41: Demonstration Marketplace Store Setup**
  * *Tasks:* Build lightweight client store (`demo-store`) allowing simulated users to browse products and merchants to upload new listings with images.
  * *Deliverable:* `demo-store/` web interface.
  * *Acceptance Criteria:* Operates as an independent participating platform connected via API keys.
* **Day 42: End-to-End Listing Screening Simulation**
  * *Tasks:* Test end-to-end flow: create legitimate listing $\to$ low risk score $\to$ published. Upload stolen/modified image $\to$ flagged $\to$ held for review.
  * *Deliverable:* Integration test suite validating complete client $\to$ API $\to$ AI $\to$ Dashboard loop.
  * *Acceptance Criteria:* System correctly distinguishes between unique and stolen images in live flow.
* **Day 43: Cross-Platform Fraud Migration Scenario Test**
  * *Tasks:* Simulate attacker listing stolen item on Store A $\to$ moderator confirms fraud $\to$ attacker attempts listing same item on Store B $\to$ Store B immediately flags it via Shared Intelligence.
  * *Deliverable:* Cross-platform verification test script.
  * *Acceptance Criteria:* Shared intelligence repository blocks/flags migrated image on Store B instantly.
* **Day 44: System Load & Concurrency Testing (Locust)**
  * *Tasks:* Run stress test simulating 20 concurrent platform listing submissions. Measure throughput (RPS), memory usage, and response times.
  * *Deliverable:* Load test configuration and latency distribution log.
  * *Acceptance Criteria:* 95th-percentile response time $\le 1,200\text{ms}$; zero 500-series server errors.
* **Day 45: Week 9 Review & Bug Remediation**
  * *Tasks:* Address edge cases identified during integration (e.g. handling corrupt image streams, non-English descriptions, extreme pricing zeros).
  * *Deliverable:* Stability patch and bug fixes across backend and frontend.
  * *Acceptance Criteria:* All end-to-end integration scenarios execute reliably.

#### Week 10: Quantitative Evaluation, Metrics Report & Project Finalization
* **Day 46: Benchmark Evaluation Suite Execution**
  * *Tasks:* Run automated evaluation on labelled benchmark dataset (1,000 listings: legitimate, exact duplicates, crops, rotations, compression, text mismatches).
  * *Deliverable:* `evaluation/benchmark_runner.py` execution and raw metrics log.
  * *Acceptance Criteria:* Evaluation runs across all test splits without interruption.
* **Day 47: Metric Calculation (Precision, Recall, F1, ROC-AUC)**
  * *Tasks:* Calculate confusion matrix, True Positives, False Positives, True Negatives, False Negatives. Plot ROC curve and calculate AUC.
  * *Deliverable:* Evaluation script calculating:
    $$\text{Precision} = \frac{TP}{TP + FP}, \quad \text{Recall} = \frac{TP}{TP + FN}, \quad F_1 = 2 \cdot \frac{\text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}}$$
  * *Acceptance Criteria:* Generates publication-ready metrics tables and charts.
* **Day 48: Transformation Robustness Breakdown Analysis**
  * *Tasks:* Tabulate detection recall for each transformation category independently (Cropping, Resizing, Rotation, Compression, Text Injection).
  * *Deliverable:* Transformation resilience breakdown table.
  * *Acceptance Criteria:* Identifies specific transformation thresholds where pHash degrades and deep embeddings maintain recall.
* **Day 49: Final Project Documentation & Academic Alignment**
  * *Tasks:* Compile evaluation findings, screenshots of moderator dashboard, API latency benchmarks, and architectural conclusions into final project report.
  * *Deliverable:* Complete project documentation and demonstration package.
  * *Acceptance Criteria:* All requirements from Chapter 1 to 4 of proposal are demonstrably satisfied.
* **Day 50: Final Presentation & Demonstration Rehearsal**
  * *Tasks:* Conduct full dry run of live demonstration: WooCommerce upload, instant fraud scoring, moderator evidence inspection, and cross-platform intelligence sharing.
  * *Deliverable:* Final project sign-off, live demo recording/slides.
  * *Acceptance Criteria:* Seamless, robust demonstration of the complete AI-powered e-commerce security system.
