# Evaluation & Performance Metrics Specification

This document details the quantitative evaluation framework, metric formulations, and experimental benchmark datasets used to validate the **Image Similarity** fraud detection system.

---

## 1. Success Target Metrics

| Evaluation Metric | Target Benchmark | Evaluation Method |
|---|---|---|
| **Exact Duplicate Precision** | $\ge 99.0\%$ | pHash Hamming distance $\le 5$ on uncompressed copies. |
| **Near-Duplicate Recall (Transformed)** | $\ge 88.0\%$ | Combined pHash + FAISS embeddings on crops, rotations, compression. |
| **Image-Text Consistency F1-Score** | $\ge 82.0\%$ | Multimodal CLIP cosine similarity on synthetic deceptive listings. |
| **Overall Classification ROC-AUC** | $\ge 0.90$ | Scikit-learn ROC curve on 1,000 labelled validation listings. |
| **False Positive Rate (FPR)** | $\le 5.0\%$ | Proportion of legitimate listings erroneously flagged as high risk. |
| **False Negative Rate (FNR)** | $\le 10.0\%$ | Proportion of fraudulent listings that evade detection. |
| **Average API Latency (P50)** | $< 500\text{ms}$ | Measured across 500 consecutive listing submissions. |
| **95th-Percentile Latency (P95)** | $< 1,200\text{ms}$ | Measured under 10 concurrent requests (Locust load test). |

---

## 2. Mathematical Formulations

### 2.1 Confusion Matrix & Classification Metrics
Given True Positives ($TP$), False Positives ($FP$), True Negatives ($TN$), and False Negatives ($FN$):

* **Precision (Positive Predictive Value):**
  $$\text{Precision} = \frac{TP}{TP + FP}$$
* **Recall (Sensitivity / True Positive Rate):**
  $$\text{Recall} = \frac{TP}{TP + FN}$$
* **F1-Score (Harmonic Mean):**
  $$F_1 = 2 \cdot \frac{\text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}}$$
* **False Positive Rate (Fall-out):**
  $$\text{FPR} = \frac{FP}{FP + TN}$$
* **False Negative Rate (Miss Rate):**
  $$\text{FNR} = \frac{FN}{TP + FN}$$

### 2.2 Receiver Operating Characteristic (ROC-AUC)
$$\text{AUC} = \int_{0}^{1} \text{TPR}(\text{FPR}) \, d(\text{FPR})$$
Evaluates the trade-off between True Positive Rate and False Positive Rate across all discrimination thresholds ($0.0 \le \tau \le 1.0$).

---

## 3. Image Perturbation Experimental Matrix

To simulate realistic fraudster attempts to evade hash detection, the test dataset is perturbed across five systematic categories:

| Transformation Category | Applied Parameters | Evaluated Model Component |
|---|---|---|
| **Bounding Box Cropping** | $10\%, 20\%, 30\%, 40\%$ central & corner crops | pHash vs. Deep Embeddings (ResNet/CLIP) |
| **Aspect-Ratio Resizing** | Downscale to $50\%, 75\%$; non-uniform stretch | pHash (scale invariance) |
| **Rotation & Flipping** | $\pm 5^\circ, \pm 15^\circ$, horizontal flip | Deep Embeddings (rotational invariance) |
| **JPEG Re-Compression** | Quality factor $Q = 70, 50, 30$ | Error Level Analysis (ELA) & pHash |
| **Visual Element Injection**| Watermark text, simulated border overlays | Forensics & FAISS nearest neighbors |
| **Deceptive Copy Mismatch** | Irrelevant title/description pairings | Multimodal CLIP cross-modal cosine similarity |

---

## 4. Benchmark Execution Protocol

1. **Phase 1: Dataset Partitioning**
   * 1,000 labelled listings split into $70\%$ training/calibration ($700$ samples) and $30\%$ held-out test set ($300$ samples).
2. **Phase 2: Automated Pipeline Ingestion**
   * Run `evaluation/benchmark_runner.py` to feed test listings into the API endpoints.
3. **Phase 3: Latency & Error Logging**
   * Measure request elapsed time from client dispatch to response delivery.
4. **Phase 4: Confusion Matrix Calculation & Report Generation**
   * Output ROC curves, PR curves, and transformation-by-transformation recall tables.
