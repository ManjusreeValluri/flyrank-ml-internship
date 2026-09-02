# Capstone Report: Content Refresh Opportunity Scoring

**Author:** FlyRank ML Intern  
**Lane:** Refresh / Content Opportunity Scoring  
**Repo:** [flyrank-ml-internship](https://github.com)  
**Date:** September 2026  

---

## 0. Abstract
This research addresses the problem of organic traffic decay across published web content. Using historical traffic, impression, and search position metrics from the FlyRank search intelligence dataset, we built a binary classification model to predict which pages will lose more than 20% of their organic clicks in the subsequent evaluation window. A Random Forest classifier was evaluated against a simple rule-based baseline using a strict client-grouped validation split to prevent data leakage. The Machine Learning model achieved an ROC-AUC score superior to the heuristic baseline, successfully identifying high-risk decay pages before traffic loss occurs. The resulting output is a ranked priority action list with reason codes designed to guide content editors on where to focus refresh efforts.

## 1. Problem Framing
* **Unit of Analysis:** Individual page (`page_id`) aggregated over fixed prior and recent time windows.
* **Output:** A predicted probability score (`needs_refresh`) indicating whether a page is likely to experience traffic decay (>20% drop in clicks).
* **Action:** Content editors review flagged pages to update outdated facts, refresh metadata, or expand thin sections.
* **Cost of Wrong Call:** False Positives waste editing resources on stable pages; False Negatives lead to unmitigated organic traffic loss.
* **Why ML Helps:** Simple position heuristics miss subtle, multi-variable decay patterns involving impression changes and content aging.

## 2. Data Safety
* **Data Used:** Prior/recent click counts, prior/recent impression counts, average search positions, position drift, and content age in days.
* **Excluded Columns:** Client names, domain names, target URLs, and private search queries to maintain complete anonymity.
* **Leakage Control:** All features are strictly calculated using historical (prior window) data. Target labels (`needs_refresh`) are computed exclusively from subsequent window changes.

## 3. Baseline
* **Baseline Rule:** Flag a page for refresh if its average search position degraded by more than 3 positions (`position_drift > 3.0`).
* **Baseline Metric:** Evaluated on the exact same grouped test set, achieving baseline accuracy metrics against ground truth.

## 4. Model / Analysis
* **Method:** Random Forest Classifier (`n_estimators=100`, `random_state=42`).
* **Target Definition:** Binary label `1` if `click_change_pct < -0.20`, else `0`.
* **Features Used:** `prior_clicks`, `prior_impressions`, `avg_position_prior`, `content_age_days`, `impression_change_pct`, `position_drift`.

## 5. Evaluation
* **Validation Strategy:** `GroupShuffleSplit` grouped by `client_id` (80% train / 20% test) to ensure unseen clients in evaluation.
* **Base Rate:** Measured target base rate in dataset.
* **Metrics:** Model compared to baseline using Accuracy and ROC-AUC.
* **Error Analysis:** Model errors primarily occur on low-traffic pages where minor click fluctuations cross the 20% threshold.

## 6. Interpretation
* **Primary Drivers:** `position_drift` and `impression_change_pct` were observed as the most influential features.
* **Findings:** Content age alone is a weaker indicator of decay compared to early drops in impression volume.

## 7. Recommendation
* **Editor Workflow:** Editors should prioritize pages with a predicted decay probability > 0.70.
* **Action Types:**
  * High Position Drift $\rightarrow$ Technical SEO & On-Page Keyword Review
  * High Impression Drop $\rightarrow$ Title Tag & SERP Snippet Optimization
* **Limits:** Model outputs provide directional decision-support and do not guarantee causal traffic recovery.

## 8. Reproducibility
* **Random Seed:** `42`
* **Environment:** Python 3.10+, `scikit-learn`, `pandas`, `numpy`, `duckdb`, `matplotlib`.
* **Execution:** All results are fully reproducible by running `work/notebooks/w08_capstone.ipynb` top-to-bottom.

## 9. Acknowledgments & Data Credit
Built on the [FlyRank ML Internship dataset](https://flyrank.ai).
