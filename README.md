# Predicting Life Expectancy Using Economic and Social Indicators

An exploratory data analysis and machine-learning regression project using World Bank indicators from **217 countries and economies (2010–2022)**.

## Motivation

How closely is life expectancy associated with economic conditions, healthcare expenditure, access to electricity, and unemployment? This project explores those relationships and assesses whether a model trained on historical country-level data can estimate life expectancy in later years.

## Research questions

1. What is the relationship between GDP per capita and life expectancy?
2. How is healthcare expenditure associated with life expectancy?
3. How do electricity access and unemployment relate to life expectancy?
4. Can regression models predict life expectancy in later years using these four indicators?

## Data

**Source:** [World Bank World Development Indicators](https://data.worldbank.org/).

The dataset was assembled from the World Bank API with **2,821 country-year observations** (217 countries/economies × 13 years) and eight columns: `Country`, `Country_Code`, `Year`, and five indicators. The indicators are:

| Column | World Bank indicator | Meaning |
|---|---|---|
| `Life_Expectancy` | `SP.DYN.LE00.IN` | Life expectancy at birth, total (years); prediction target |
| `GDP_Per_Capita` | `NY.GDP.PCAP.CD` | GDP per capita (current US$) |
| `Health_Expenditure` | `SH.XPD.CHEX.GD.ZS` | Current health expenditure (% of GDP) |
| `Electricity_Access` | `EG.ELC.ACCS.ZS` | Access to electricity (% of population) |
| `Unemployment` | `SL.UEM.TOTL.ZS` | Unemployment, total (% of total labor force; modeled ILO estimate) |

**Missing predictor values:** Electricity access 0.92%, GDP per capita 2.55%, healthcare expenditure 11.88%, and unemployment 13.86%. No missing target values or duplicate country-year rows were observed.

Data downloads can change when the World Bank updates its series. The original CSV is committed for reproducibility. The metrics below refer to the dataset retrieved and analyzed for this project, not necessarily future revisions.

## Methods

The main notebook follows the **CRISP-DM** framework: business understanding, gathering and assessing data, analysis/visualization, cleaning and preparation, modeling, evaluation, and communication through a new-scenario prediction. The categorical country identifiers are excluded from numerical regression rather than encoded.

- Exploratory analysis: descriptive statistics, histograms, and a Pearson correlation matrix.
- **Chronological split:** 2010–2019 for training (2,170 observations), 2020–2022 for testing (651 observations).
- **Missing values:** median imputation, fitted on the training data within each scikit-learn pipeline to avoid test-data leakage.
- Models: a mean-predicting `DummyRegressor` baseline, `LinearRegression`, and `RandomForestRegressor` (`n_estimators=200`, `min_samples_leaf=3`, `random_state=42`).
- Evaluation: MAE and RMSE (years, lower is better), and R² (higher is better).

## Results

### Correlations with life expectancy

| Indicator | Pearson correlation |
|---|---:|
| Electricity access | 0.782 |
| GDP per capita | 0.601 |
| Healthcare expenditure | 0.260 |
| Unemployment | -0.034 |

Correlations are descriptive, not evidence of causality.

### Test-set model comparison (2020–2022)

| Model | MAE (years) | RMSE (years) | R² |
|---|---:|---:|---:|
| Mean baseline | 6.166 | 7.706 | -0.004 |
| Linear regression | 3.321 | 4.459 | 0.664 |
| **Random forest** | **2.328** | **3.463** | **0.797** |

Random forest reduced MAE by approximately **29.9%** relative to linear regression on this test set. Its mean absolute error was **2.33 years**. This is an average: errors for individual observations can be far larger.

![Model comparison chart](model_comparison.svg)

### Random-forest feature importance

| Feature | Importance |
|---|---:|
| Electricity access | 0.694545 |
| GDP per capita | 0.185705 |
| Healthcare expenditure | 0.060024 |
| Unemployment | 0.059726 |

These are the model's feature-importance scores, **not causal effects** or percentages of life expectancy.

### New prediction scenario

For a **hypothetical country** with GDP per capita of **$15,000**, health expenditure of **7% of GDP**, electricity access of **100%**, and unemployment of **6%**, the fitted random forest predicted **76.77 years** of life expectancy. The scenario is illustrative, not an observed country or a reliable policy forecast.

## Limitations and data-quality notes

- An exploratory rule flagged **10 country-year observations** with a year-over-year change in recorded life expectancy greater than 10 years in absolute value. These observations were **not automatically deleted**.
- **Sensitivity check (not a replacement evaluation):** Three of the 651 test observations were flagged by a rule identifying absolute year-to-year life-expectancy changes above 10 years. Excluding these **only when calculating test metrics** yielded MAE **2.252** and R² **0.818** on the other **648** observations, versus MAE **2.328** and R² **0.797** on all 651. Since the exclusion rule looks at observed target values, the filtered scores are **post-hoc diagnostics, not unbiased deployment metrics**. The full-test results remain the primary scores.
- The largest test error was **36.35 years** for Central African Republic in 2022 (observed 18.818 versus predicted 55.168). The 18.818 value is present in the [World Bank–sourced FRED series](https://fred.stlouisfed.org/series/SPDYNLE00INCAF), but differs substantially from life expectancy estimates from other providers such as the [WHO](https://www.who.int/data/gho/data/countries/country-details/GHO/central-african-republic?countryProfileId=134de201-66b). Cross-source discrepancies warrant caution; this project does not assert that the record is definitively incorrect.
- Countries appear in both training and test sets in different years. These results show prediction for **later observations of previously represented countries**, not generalization to unseen countries.
- Health and economic variables may be correlated with other unmodeled factors. Feature importance does not identify causal mechanisms.
- The test years include 2020–2022, when global health disruptions may have affected outcomes.

## Repository contents

| File | Purpose |
|---|---|
| `World_Bank_Life_Expectancy_Prediction.ipynb` | **Reviewed main notebook**: PEP8-compliant code, documented helper functions, CRISP-DM sections, answers to four questions, models, charts and scenario prediction. Reads the included CSV without an internet connection. |
| `World_Bank_Life_Expectancy.csv` | Frozen original 2,821-row analysis dataset for reproducible results. |
| `README.md` | Project overview, setup, methods, findings, limitations, and acknowledgments. |
| `blog_post.md` | Editable Markdown source of the nontechnical blog article; includes a featured image and four question-specific visualizations. |
| `model_comparison.svg` | Model comparison chart for repository documentation. |
| `docs/index.html` | Standalone, styled, publication-ready blog article for GitHub Pages. |
| `docs/images/*.svg` | Five original data-grounded images for the hosted article: cover plus one visual for each research question. |
| `requirements.txt` | Required Python libraries. |
| `archive/Original_World_Bank_Life_Expectancy_Prediction.ipynb` | Backup of the author's original executed notebook, preserved before readability and reproducibility improvements. |

## Running the project

1. Clone or download the repository, keeping the CSV beside the notebook.
2. Install dependencies: `pip install -r requirements.txt`.
3. Open `World_Bank_Life_Expectancy_Prediction.ipynb` in Jupyter.
4. Choose **Kernel → Restart & Run All**. The main notebook loads the committed CSV locally; it does not require live World Bank API access.
5. Check the model comparison and illustrative prediction against the documented metrics. Results were also independently recomputed against the archived CSV during the project review.

## Published blog post (Udacity requirement)

Udacity reviewers require a **separately hosted blog article**, not merely a Markdown file viewed inside a GitHub code repository. The old `blog_post.md` URL alone **does not meet this requirement**.

This repository includes a standalone, illustrated article at [`docs/index.html`](docs/index.html) with an image directly after its title, four questions, original visuals, conclusions, and source links.

**To publish:** In GitHub, open **Settings → Pages → Build and deployment → Source: Deploy from a branch**, choose **`main`** and **`/docs`**, then click **Save**. Once the site is deployed and publicly accessible, provide the live hosted article URL as Udacity's second submission link. (The expected site address uses the form `https://luro7.github.io/World-Bank-Life-Expectancy-Prediction/`; verify it loads before submitting.)

Markdown source: [`blog_post.md`](blog_post.md). The GitHub source link should **not** replace the independent hosted blog URL in the Udacity submission.

## Acknowledgments

Data provided by the **World Bank World Development Indicators**, via its public API. The project uses `pandas`, `numpy`, `scikit-learn`, `matplotlib`, and `seaborn` and was completed as part of a **Udacity** learning project. Health-source cross-checks also reference the Federal Reserve Bank of St. Louis (FRED) and the World Health Organization (WHO).