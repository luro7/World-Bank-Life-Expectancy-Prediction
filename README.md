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

The notebook follows the **CRISP-DM** framework: business understanding, data understanding, data preparation, modeling, evaluation, and a new-scenario prediction.

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
| `World_Bank_Life_Expectancy_Prediction.ipynb` | Original executed Jupyter notebook provided by the project author, including analysis, model evaluation, plots, and prediction. The notebook's original data-loading cell downloads the dataset from the live World Bank API. |
| `World_Bank_Life_Expectancy.csv` | Original analyzed dataset, included in this repository to reproduce the reported metrics. The notebook may also retrieve refreshed values from the World Bank API. |
| `README.md` | Project motivation, methods, key findings, limitations, and acknowledgments. |
| `blog_post.md` | Nontechnical summary for publication (e.g., GitHub Pages or adaptation to Medium). |
| `model_comparison.svg` | Comparison of test-set mean absolute errors. |
| `docs/index.md` | Public-facing, nontechnical article (ready for GitHub Pages after enabling Pages). |
| `docs/model_comparison.svg` | Chart used in the blog post. |
| `requirements.txt` | Python packages needed to run the notebook. |

## Running the project

1. Clone the repository and create a Python environment.
2. Install dependencies: `pip install -r requirements.txt`.
3. Open `World_Bank_Life_Expectancy_Prediction.ipynb` in Jupyter.
4. The original notebook's second code cell downloads live World Bank data and overwrites the CSV. To reproduce the archived dataset results, replace that cell with `df = pd.read_csv("World_Bank_Life_Expectancy.csv")`, then execute subsequent cells in order. Running the original download cell unchanged may produce different numbers if the World Bank revises data.
5. To publish the blog via GitHub Pages, select **Settings → Pages → Deploy from a branch → main → /docs** and save (if Pages is supported for your repository).

## Public-facing article

Read the [nontechnical blog post](blog_post.md), or enable GitHub Pages from the `/docs` folder to publish the [blog article](docs/index.md) as a website.

## Acknowledgments

Data provided by the **World Bank World Development Indicators**, via its public API. The project uses `pandas`, `numpy`, `scikit-learn`, `matplotlib`, and `seaborn` and was completed as part of a **Udacity** learning project. Health-source cross-checks also reference the Federal Reserve Bank of St. Louis (FRED) and the World Health Organization (WHO).