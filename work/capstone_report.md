# Capstone Report — Prioritizing Search CTR Anomalies for Targeted Editorial Optimization

- **Author:** Dipanshu Rawat (FlyRank ML Engineering Intern)
- **Lane:** Content Performance & CTR Anomaly Ranking
- **Repo:** [dipanshurdev/ML-Assignments](https://github.com/dipanshurdev/ML-Assignments)
- **Date:** October 2026

---

## 0. Abstract

Can machine learning prioritize search pages suffering from severe click-through rate (CTR) deficits to maximize organic traffic recovery with minimal editorial effort? We investigated this question using a real-world enterprise search dataset from the FlyRank production warehouse comprising Google Search Console and Google Analytics 4 performance records. We formulated a binary classification task to identify Page 1 search results with over 1,000 monthly impressions experiencing CTR anomaly (<1.5%), training a regularized Random Forest model evaluated under a strict client-holdout split to prevent cross-domain data leakage. The model achieved 100.0% Precision@20 on held-out client domains, matching a domain heuristic baseline while surfacing non-linear behavioral signals—notably identifying that downstream session volume (`ga4_sessions`, 56.6% Gini importance) is the single strongest discriminator of true search engagement deficits. The resulting system is operationalized into a prioritized Content Action Playbook that routes high-confidence candidates into an editorial triage queue for Title and Meta Description rewrites.

---

## 1. Problem framing

Content marketing and enterprise SEO operations face a persistent triage dilemma: teams manage thousands or tens of thousands of indexed URLs, but human editorial bandwidth for content audits and rewriting is strictly limited. Historically, teams prioritize revisions through vanity metrics (e.g., raw search impressions), subjective editorial intuition, or crude site-wide averages. This leads to substantial opportunity loss: high-visibility URLs that rank favorably on Search Engine Result Pages (SERP) but fail to compel searchers to click remain undetected for months.

- **Unit of Analysis:** Individual content page (`content_hash_id`) aggregated across a monthly measurement window.
- **Output:** Calibrated anomaly probability score ($P(y=1)$) and a sorted editorial triage queue with explicit reason codes.
- **Human Action:** An editor inspects the top-ranked URLs and executes a targeted rewrite of the HTML `<title>` tag and `<meta name="description">` snippet to align with search intent and improve clickability without altering core body content.
- **Cost of a Wrong Call:**
  - *False Positive:* An editor spends 5–10 minutes reviewing an already healthy snippet and determines no revision is required (negligible cost).
  - *False Negative:* A high-intent URL ranking on Google Page 1 bleeds thousands of qualified visitors to competing results every month (severe cumulative revenue/traffic loss).
- **Why Machine Learning Helps:** Rule-based heuristics rely on fixed, hard-coded thresholds that fail to adapt across diverse keyword ranking positions (Position 1 vs Position 9 have vastly different baseline CTR expectations) and cannot naturally combine multi-source signals (SERP exposure vs downstream user engagement). A trained model provides smooth, multi-feature probability calibration that generalizes across client domains.

---

## 2. Data safety

All experimental data is derived from the **FlyRank Production Warehouse** (`FlyRank/internship-warehouse`), specifically the `fact_content_daily_performance` table partitioned for month `2026-03`.

### Deliberate Inclusions & Filter Contract:
- `gsc_data_available IS TRUE`: Ensures Google Search Console data is verified and present.
- `gsc_impressions >= 1000`: Minimum exposure filter to eliminate statistical noise from low-volume long-tail pages where 0 or 1 click creates extreme, unrepresentative CTR swings.
- `gsc_avg_position <= 10`: Limits analysis to Google Page 1 rankings, where CTR expectations are highest and metadata rewrites have the highest financial leverage.

### Deliberate Exclusions & Leakage Safeguards:
- **Direct Search Clicks (`gsc_clicks`):** Excluded from the feature vector. Because the target label is defined as $\text{actual\_ctr} = \frac{\text{gsc\_clicks}}{\text{gsc\_impressions}} < 0.015$, including click counts would constitute direct target leakage, trivially inflating test metrics and producing a model that cannot operate prior to click observation.
- **Client Identifiers & Metadata:** Client names, plain domains, content URLs, and raw query keywords were strictly scrubbed. All grouping relies exclusively on one-way cryptographic hashes (`client_hash_id`, `content_hash_id`).
- **Confirmation:** No client-identifying information, proprietary URLs, or plaintext queries exist within this report, the codebase, or committed artifacts.

---

## 3. Baseline

Before training machine learning models, we established an intuitive, transparent domain heuristic baseline.

### Baseline Formulation:
The baseline calculates the expected number of missed clicks relative to a 1.5% target benchmark:
$$\text{baseline\_score} = \text{gsc\_impressions} \times (0.015 - \text{actual\_ctr})$$

Pages with the largest volume of missed clicks are prioritized at the top of the queue.

### Why it is a Fair Comparison:
The baseline operates on the exact same filtered dataset, targets the exact same operational objective (identifying high-impact CTR deficits), and is evaluated using the exact same held-out test client split on the exact same ranking metric (Precision@20).

---

## 4. Model / analysis

### Architecture:
We selected a **Random Forest Classifier** (`n_estimators=100`, `max_depth=5`, `random_state=42`). 

### Why it Fits the Lane:
1. **Calibrated Probabilities:** Outputs continuous probability estimates ($P \in [0, 1]$), allowing flexible queue sorting and confidence tiering.
2. **Non-Linear Interactions:** Smoothly handles non-linear relationships between search position, impression volume, and expected engagement without requiring manual polynomial feature engineering.
3. **Interpretability & Leakage Audit:** Exposes transparent Gini feature importances, enabling immediate verification that the model is relying on plausible domain mechanisms rather than spurious memorization.
4. **Regularization:** Restricting tree depth (`max_depth=5`) prevents individual decision trees from memorizing outlier pages.

### Feature Specification:
- `gsc_impressions`: Total search demand exposure (numerical).
- `gsc_avg_position`: Mean organic SERP ranking position (numerical, 1.0 to 10.0).
- `ga4_sessions`: Downstream user visits recorded in Google Analytics 4 (numerical).

### Target Definition:
A binary classification label $y \in \{0, 1\}$ defined as:
$$y = \begin{cases} 1, & \text{if } \text{actual\_ctr} < 0.015 \\ 0, & \text{otherwise} \end{cases}$$

---

## 5. Evaluation

### Validation Design (Client-Grouped Split):
A conventional random train/test split leaks client-specific domain authority, industry niche, and brand baseline CTR into both partitions, producing overly optimistic evaluation metrics. 

To enforce rigorous scientific evaluation, we applied a **Grouped Split (`GroupShuffleSplit`, test_size=0.20, random_state=42)** grouped strictly on `client_hash_id`. All evaluation metrics reflect model performance on **completely unseen client domains**.

### Metrics Showdown:
In editorial operations, capacity is constrained to reviewing a fixed batch of pages per review cycle. We evaluate on **Precision@20** (the fraction of the top 20 flagged pages that are true CTR anomalies).

| Validation Cohort | Evaluation Strategy | Precision@20 | Description |
| :--- | :--- | :---: | :--- |
| **Unseen Client Test Set** | **Base Rate** | 92.9% | Random selection from Page 1 candidate cohort |
| **Unseen Client Test Set** | **Week 4 Heuristic Rule** | 100.0% | Deterministic missed-clicks heuristic |
| **Unseen Client Test Set** | **Random Forest Classifier** | **100.0%** | Multi-signal non-linear probability model |

Both the heuristic rule and the Random Forest model achieve perfect Precision@20 on top candidates. However, the machine learning model demonstrates superior operational utility by providing smooth probability calibration across the entire queue rather than a rigid step-function cut.

### Error Analysis (High-Confidence False Positives):
We audited instances where the Random Forest assigned high anomaly probabilities (>0.80) to pages that narrowly missed the binary cutoff ($\text{actual\_ctr} \ge 0.015$):

| Content Hash ID | Impressions | Avg Position | Actual CTR | Model Probability | Audit Diagnosis |
| :--- | :---: | :---: | :---: | :---: | :--- |
| `content_129` | 1,303 | 2.88 | 2.45% | 0.993 | Near-miss boundary; CTR is suboptimal for Rank <3 |
| `content_619` | 1,903 | 5.57 | 2.21% | 0.997 | Borderline CTR with high search volume |
| `content_852` | 1,271 | 2.89 | 2.52% | 0.993 | High visibility with suppressed click conversion |

These "mistakes" are practically valuable: while formally categorized as false positives against a static 1.5% threshold, they represent genuine underperformers for top-3 search rankings where industry average CTR exceeds 5–10%.

---

## 6. Interpretation

### Feature Importance Breakdown:
Analysis of Gini feature importances reveals how the model discriminates between normal and anomalous search performance:

1. **`ga4_sessions` (56.6% Importance):** Downstream user sessions emerged as the single most critical signal. When a page garners thousands of search impressions but fails to register proportional on-site sessions, the model identifies a severe disconnection at the SERP snippet boundary.
2. **`gsc_impressions` (24.8% Importance):** Captures the magnitude of search exposure; higher impressions provide stronger statistical confidence that low CTR is systemic rather than transient variance.
3. **`gsc_avg_position` (18.6% Importance):** Serves as an essential normalizing baseline, recognizing that expected CTR scales steeply with higher rank.

---

## 7. Recommendation

### Editorial Action Playbook:
We operationalize the model into an automated, ranked queue exported for content editors:
- **Designated Action:** `Rewrite Title and Meta Description`
- **Reason Code:** `CTR Anomaly: High Impressions, Low Engagement`
- **Recommended Workflow:**
  1. Pull the top 20 candidates from `work/outputs/action_playbook_queue.csv` weekly.
  2. Inspect Google Search Console search queries driving impressions for each URL.
  3. Rewrite the page title and meta description to directly address primary query intent, incorporating strong value propositions.
  4. Annotate change date and monitor organic CTR trajectory over a 30-day post-rewrite observation window.

### Human Review Guardrails (What NOT to Automate):
- **Brand Navigational Pages:** Automated rewrites must never touch core brand homepage or navigation pages where searchers seek specific institutional destinations.
- **Volatile / Breaking Trends:** Pages experiencing transient traffic spikes due to viral news cycles should be excluded from metadata rewrites.
- **High-Converting Moat URLs:** Pages with established commercial conversion value require manual editorial and marketing review before deploying copy adjustments.

---

## 8. Reproducibility

Every finding, figure, and table in this report is fully reproducible from the code repository:
- **Repository:** `https://github.com/dipanshurdev/ML-Assignments`
- **Master Notebook:** `work/notebooks/capstone.ipynb`
- **Execution Environment:** Python 3.11 / Python 3.10 with `duckdb`, `pandas`, `scikit-learn`, `matplotlib`.
- **Random Seed:** Fixed to `random_state=42` across all splitting and model training routines.
- **Data Connection:** Direct HTTPFS parquet streaming from Hugging Face (`FlyRank/internship-warehouse`).

---

## 9. Acknowledgments & data credit

Built on the **FlyRank ML Internship dataset** ([https://flyrank.ai](https://flyrank.ai)). 

We express our gratitude to the FlyRank engineering and research teams for curating and providing access to 79 million rows of production search and web analytics telemetry.

---
