# File Guide

This page explains every file in the repository and how to run the analysis. For the project overview, see the [README](../README.md).

---

## 1. How to run the analysis

You don't need to run anything to read the analysis. GitHub displays `notebooks/iclr_big_data_analysis.ipynb` with all of its outputs saved. To rerun it yourself:

1. **Unzip the OpenAlex data.** Unzip `data/openalex.json.zip` to get `openalex.json` (31 MB).
2. **Create a Volume in Databricks.** In your workspace, open **Catalog**, choose or create a catalog and schema, and **create a Volume**.
3. **Upload the data.** Upload `data/iclr26v1.parquet` and the unzipped `openalex.json` into the Volume.
4. **Import the notebook.** Go to **Workspace → Import** and choose `notebooks/iclr_big_data_analysis.ipynb`.
5. **Set the data path.** In the first code cell, change `DATA_DIR` to your Volume's path, for example `/Volumes/workspace/default/iclr/`. Don't forget the trailing `/`.
6. **Click Run all.** Every cell runs in order. The random forest cross-validation (Q1 extension) takes the longest.

**Troubleshooting**

| Problem | Fix |
|---|---|
| A path or file not found error in the first cell | Check that `DATA_DIR` matches your Volume path exactly and ends with `/`, and that both `iclr26v1.parquet` and `openalex.json` are in the Volume. |
| OpenAlex loads as a single broken row | The file must be read with `multiLine = true`, which the notebook already does. Make sure you uploaded the unzipped `.json` file, not the `.zip`. |
| A CrossValidator error mentioning a temporary path | The notebook sets `SPARKML_TEMP_DFS_PATH` to `DATA_DIR`, which serverless compute needs. Make sure your Volume is writable. |

---

## 2. Notebook (`notebooks/`)

| File | Description |
|---|---|
| `iclr_big_data_analysis.ipynb` | The complete PySpark analysis, exported from Databricks with all outputs saved |

| Cell | Content | Report section |
|---|---|---|
| 1 | Imports and data loading (`DATA_DIR` setting) | 1 |
| 2 | Data-quality audit and cleaning for both datasets | 2 |
| 3 | Q1: accepted, rejected and withdrawn counts | Q1 |
| 4 | Q1 extension: random forest predicting acceptance (features, correlation check, stratified split, CrossValidator, feature importance, confusion matrix) | Q1 extension |
| 5 | Q2: accepted papers with at least one score below 5 | Q2 |
| 6 | Q2 extension: reviewer-disagreement tiers and trend over time | Q2 extension |
| 7 | Q3: top 10 papers by mean score | Q3 |
| 8 | Q3 extension: linear regression test for score inflation | Q3 extension |
| 9 | Q4: keywords that spiked from fewer than 5 to more than 50 papers | Q4 |
| 10 | Q4 extension: acceptance rate before, during and after each spike | Q4 extension |
| 11 | Q5: ICLR papers with OpenAlex records, by year | Q5 |
| 12 | Q5 extension: geographic analysis (paper counts, acceptance rate, mean score by country) | Q5 extension |
| 13 | Q6: low-scoring papers in the 95th citation percentile | Q6 |
| 14 | Q6 extension: citations of accepted vs rejected low-scoring papers | Q6 extension |

Cell numbers count code cells. The notebook also starts with a title cell.

---

## 3. Data (`data/`)

| File | Description |
|---|---|
| `iclr26v1.parquet` | 55,906 ICLR submissions, 2017–2026, from the [Berens lab ICLR dataset](https://github.com/berenslab/iclr-dataset) (MIT licence). The original is 49 MB, over GitHub's 25 MB upload limit, so this copy is re-saved with stronger zstd compression (23 MB). The data is identical, and Spark reads zstd parquet natively. Columns: `year`, `id`, `title`, `abstract`, `authors`, `author_ids`, `decision`, `scores` (array), `keywords` (array), `labels`. |
| `openalex.json.zip` | A 2,245-record OpenAlex scrape made by the module instructor to match ICLR submissions. It unzips to `openalex.json`, a single JSON array with nested `authorships` (author institutions and countries), `cited_by_count` and `citation_normalized_percentile`. It's zipped because the unzipped file (31 MB) is over GitHub's 25 MB upload limit. |

---

## 4. Results (`results/`)

Every table was exported from the outputs of the notebook, so the numbers match the notebook and the report exactly.

| File | Contents |
|---|---|
| `data_quality_summary.csv` | Row counts, empty decisions and score arrays, duplicates, null percentiles |
| `q1_decision_counts.csv` | Accepted, rejected and withdrawn counts, with shares |
| `q1_feature_correlation.csv` | Pearson correlation between the five score features |
| `q1_rf_feature_importance.csv` | Random forest feature importances |
| `q1_rf_test_results.csv` | AUC, accuracy, best hyperparameters, class counts and the confusion matrix |
| `q2_disagreement_tiers.csv` | Tier thresholds, paper counts, acceptance rates, accepted papers with a score below 5 |
| `q2_accepted_disagreement_by_year.csv` | Mean score SD and mean score of accepted papers, by year |
| `q3_top10_mean_score.csv` | The 10 papers with the highest mean score |
| `q3_score_inflation_regression.csv` | Regression slope, intercept and test R² for each group |
| `q4_keyword_spikes.csv` | The four spiking keywords, with counts before and during the spike |
| `q4_keyword_acceptance_trajectory.csv` | Year-by-year acceptance rate for each spiking keyword |
| `q5_openalex_match_by_year.csv` | Matched papers and match rate by year |
| `q5_countries.csv` | Top 10 countries: matched papers, acceptance rate, mean reviewer score |
| `q6_low_score_highly_cited.csv` | The 13 low-scoring papers in the 95th citation percentile |
| `q6_citations_by_outcome.csv` | Citation statistics for accepted vs rejected low-scoring papers |

---

## 5. Documentation (`docs/`)

| File | Description |
|---|---|
| `report.pdf` | The full report: setup, data cleaning, and for each question its assumptions, implementation, results and discussion |
| `FILE_GUIDE.md` | This file |
