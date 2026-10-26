# Can a Country's Economy Predict How Long People Live?

*Four questions, 217 countries and economies, and one machine-learning experiment using 2010–2022 World Bank data.*

![Comparing the average prediction errors of three approaches to life expectancy](model_comparison.svg)

Why do some places have longer life expectancies than others? Economic conditions and public services may be part of the picture. To explore the evidence, I examined **2,821 country-year observations** across **217 countries and economies** from **2010 to 2022**.

The analysis considers four indicators—income per person, healthcare spending relative to GDP, access to electricity, and unemployment—and tests whether they can help estimate life expectancy in later years.

## Question 1: Are richer countries associated with longer life expectancy?

**Yes, there is a positive relationship in these data.** The correlation between GDP per capita and life expectancy was **+0.601**.

That does not mean an extra dollar of GDP causes longer lives. Wealthier places may differ in many other ways, including public infrastructure, education, and access to healthcare. GDP per capita also varies widely across countries: its median in this dataset was approximately **$6,730**, compared with a mean of approximately **$18,445**.

## Question 2: Is higher healthcare spending associated with longer lives?

**Somewhat, but less strongly than income.** Healthcare expenditure, measured as a **percentage of GDP**, had a correlation of **+0.260** with life expectancy.

This measure captures the share of national output devoted to healthcare—not spending per patient, accessibility, or the quality of services. The weaker relationship is not evidence that healthcare is unimportant.

## Question 3: How do electricity access and unemployment compare?

**Access to electricity showed the strongest positive correlation**, at **+0.782**. The unemployment rate had a correlation of **-0.034**, effectively showing little linear relationship in these observations.

| Indicator | Correlation with life expectancy |
|---|---:|
| Access to electricity | **+0.782** |
| GDP per capita | +0.601 |
| Healthcare spending (% of GDP) | +0.260 |
| Unemployment | -0.034 |

Reliable power supports everyday services, including medical facilities, but the data cannot tell us whether electricity *caused* the observed difference. Other development factors are intertwined.

## Question 4: Can machine learning estimate life expectancy in later years?

**Yes, within the test period, a random forest model performed better than two simpler alternatives.** Models were trained on observations from **2010–2019** and evaluated on **2020–2022**.

| Model | Average absolute error | R² |
|---|---:|---:|
| Baseline (predicting the average) | 6.17 years | -0.004 |
| Linear regression | 3.32 years | 0.664 |
| **Random forest** | **2.33 years** | **0.797** |

The random forest's average error was approximately **29.9% lower** than that of linear regression. Its R² of **0.797** indicates that it captured roughly **79.7% of the variation** in the held-out observations.

The average error hides some substantial individual misses. These results also describe later observations of countries already represented in training, not entirely new countries.

### A new prediction scenario

Imagine a hypothetical country with a GDP per person of **$15,000**, healthcare expenditure equal to **7% of GDP**, **100% electricity access**, and **6% unemployment**.

The fitted random forest estimated **76.77 years** of life expectancy. This is an illustration of the model's output, not a forecast for any actual country or proof that changing an indicator changes life expectancy.

## Why the quality of the data matters

An exploratory rule flagged **10 country-year observations** showing life expectancy changes greater than 10 years compared with the previous year. One notably unusual value was **18.818 years for the Central African Republic in 2022**, which produced the largest model error, approximately **36.35 years**. The World Bank-sourced [FRED series](https://fred.stlouisfed.org/series/SPDYNLE00INCAF) contains that value, but the cross-source discrepancy requires caution rather than automatic deletion.

Three flagged cases occurred in the **651-row test set**. When those three were excluded *only for a sensitivity calculation*, the MAE changed from **2.328 to 2.252 years** and R² from **0.797 to 0.818**. Because the review rule relied on observed outcomes, the adjusted figures are **diagnostic only**; they are not a second independent accuracy estimate.

## Takeaway

The data show stronger associations for **electricity access and income per person** than for the other two indicators studied. A random forest can make useful numerical estimates on later-year observations, but model scores must always be interpreted alongside data quality and the limits of observational information.

Machine learning can help us recognize patterns. It cannot, on its own, prove what causes people to live longer.

---

*Source: [World Bank World Development Indicators](https://data.worldbank.org/), 2010–2022. The figures come from the archived analysis dataset in this project's [public GitHub repository](README.md). All comparisons describe associations, not causal effects.*
