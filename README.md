# Kickstarter Campaign Success Prediction & Historical Segmentation

An individual academic machine-learning project exploring two questions:

1. **Can Kickstarter campaign success be predicted from features available around launch?**
2. **How do historical Kickstarter campaigns differ in their observed attributes and outcomes?**

## Project overview

| Analysis | Technique | Key result from the original saved run |
| --- | --- | --- |
| Classification | Lasso feature selection + tuned Gradient Boosting Classifier | **75.73% originally; 75.80% reproduced** (random 70/30 train/test split) |
| Retrospective clustering | StandardScaler + K-Means | **4 clusters** reproduced with the supplied workbook |

**Scope:** This repository is a cleaned portfolio adaptation of the original INSY 446 individual project. Using the original workbook privately, the classification test accuracy and all four cluster counts were reproduced on November 2, 2026. The full GridSearchCV was not rerun.

## Classification workflow

- Retain projects labeled `successful` or `failed`.
- Exclude clearly outcome-dependent fields (`pledged`, `usd_pledged`, `backers_count`, `spotlight`, etc.) for classification.
- Engineer campaign duration, launch preparation time, a weekend-launch indicator, and log-transformed funding goals in USD.
- Encode categoricals and standardize features with the scaler fit on the training split.
- Use Lasso for feature selection, followed by `GridSearchCV` for a Gradient Boosting Classifier.
- Evaluate against the held-out 30% set.

**Results recorded in the original notebook:**
- Best cross-validation accuracy: **75.08%**
- Held-out test accuracy: **75.73% originally**, **75.80% reproduced** (current environment)
- Selected Gradient Boosting settings: `n_estimators=550`, `max_depth=2`, `learning_rate=0.1`, `subsample=0.8`.

The original evaluation uses a **random split**, so results should not be interpreted as proven performance for future campaigns or different time periods.

### Independent execution check (November 2, 2026)

A run of the portfolio notebook with the supplied source workbook (**15,215 rows × 46 columns; 14,463 successful/failed projects**) completed using the originally selected hyperparameters. The original 360-fit GridSearchCV was not repeated.

| Evaluation | Result |
| --- | ---: |
| Test accuracy (reproduced) | **75.80%** |
| Majority-class baseline | 58.24% |
| Balanced accuracy | 74.20% |
| F1 score (successful = positive) | 80.16% |

The small difference from the original 75.73% test accuracy has not been isolated to a specific cause; the two runs should not be described as numerically identical.

## Retrospective clustering

Historical Kickstarter campaigns were analyzed with K-Means. The original run produced:

| Cluster | Number of projects | Brief description from the report |
| --- | ---: | --- |
| 0 | 865 | Smaller, lower-goal campaigns, often Crafts |
| 1 | 1,615 | Category-driven campaigns, especially Music |
| 2 | 6,737 | High engagement/visibility in the original report |
| 3 | 5,246 | High-goal projects with weaker presentation signals in the original report |

**Verification:** The four cluster sizes were reproduced exactly. A sampled silhouette score at `k=4` was **0.132** (sample of 1,000 rows), indicating weak separation; it is not a full-dataset silhouette calculation.

**Critical limitation:** Unlike the classification analysis, the original clustering includes `state` as well as post-launch indicators such as `backers_count`, `usd_pledged`, and `spotlight`. Because known outcomes were used to construct the clusters, the analysis is not a prospective outcome-independent segmentation. This is useful only as a **retrospective descriptive exercise**; it cannot support an early-warning segmentation claim or be used to forecast unknown outcomes as implemented. Moreover, clusters had similar observed success shares (~55–58%) in a fresh run; do not characterize individual clusters as essentially successful or failed. A future iteration should exclude outcome/post-launch variables and rerun clustering before making prospective recommendations.

## Repository contents

```text
notebooks/kickstarter_analysis.ipynb    # Portfolio-cleaned notebook (outputs cleared)
reports/individual_project_report.pdf   # Original academic writeup
data/README.md                          # Required data file and publishing notes
requirements.txt                        # Python libraries
.gitignore                              # Excludes datasets, caches, credentials
```

## How to run

1. Obtain the original `Kickstarter.xlsx` from an authorized source.
2. Put it in `data/Kickstarter.xlsx` (the file is intentionally **not committed**).
3. Create a Python environment, then install dependencies:

   ```bash
   python -m pip install -r requirements.txt
   ```

4. From the repository root, launch Jupyter:

   ```bash
   jupyter notebook notebooks/kickstarter_analysis.ipynb
   ```

5. Run cells in order. By default, the exhaustive 360-fit GridSearchCV is skipped; set `RUN_GRID_SEARCH = True` to enable it. Silhouette comparisons use a 1,500-row sample to reduce memory requirements.

## Changes from the submitted notebook

- Replaced a personal Mac filesystem path with `data/Kickstarter.xlsx`.
- Removed the course-specific `Kickstarter-Grading.xlsx` scoring section.
- Cleared stored outputs and normalized the Jupyter kernel metadata.
- Corrected the silhouette-per-cluster calculation to use **the final 4-cluster labels** rather than labels remaining from an earlier tuning loop. Uses sampling to avoid excessive pairwise distance computation on laptops.
- Made 360-fit GridSearchCV optional and capped model-search parallelism at 2 workers to reduce system resource usage.
- Made dataset lookup work from either repository root or `notebooks/`, and the optional missingness plot is skipped if `missingno` is unavailable.
- Preserved the substantive original modeling choices instead of silently rebuilding the analysis.

## Limitations and next steps

- The Excel workbook was supplied for verification, but is intentionally omitted from the public portfolio package. Validation reproduced classification accuracy and cluster sizes. Full grid-search rerun is not yet verified.
- Many preprocessing steps are performed before the classification split; a production workflow should use a `Pipeline` and fit data-dependent transforms only on training data.
- A temporal holdout, balanced classification metrics (precision/recall, ROC-AUC), and class-prevalence baseline would strengthen the evidence.
- The retrospective K-Means features include campaign outcomes, so the clusters should **not** be interpreted as pre-launch segments.
- Check the source dataset license and course-publication rules before making the repository public.

## Attribution

Individual academic analysis for McGill INSY 446. See the original report for the submitted discussion and methodological context.
