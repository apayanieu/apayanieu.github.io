---
title: "Good Investments: Does Higher Capital Formation Drive GDP Growth?"
excerpt: "A cross-country panel data analysis testing whether higher Gross Capital Formation rates lead to GDP per capita growth, using World Bank data from 1993–2023 with lagged investment rates, regional decomposition, and multiple hypothesis tests."
collection: portfolio
tags:
  - R
  - Econometrics
  - Data Analysis
  - Panel Data
use_math: true
---

*Joint work with Katrín Árnadóttir, Candela Berzal, Marta Merino, and Alejandra Sancho (IE University, Spring 2025)*

---

## Overview

A standard claim in development economics is that investment drives growth: countries that allocate a larger share of GDP to **Gross Capital Formation** should, in theory, see faster GDP per capita growth. This project puts that claim to a rigorous empirical test.

Using **World Bank panel data** spanning **214 countries over 30 years** (1993–2023), we ask a deceptively simple question: *does a higher investment rate actually lead to higher GDP growth?* The answer turns out to be far more nuanced than textbook models suggest. A simple linear regression finds **no significant overall relationship** ($R^2 = 0.003$, $p = 0.78$). But decomposing by income group and region reveals that **investment effectiveness is deeply context-dependent** — strong in high-income and East Asian economies, weak or even negative in low-income and Sub-Saharan African ones.

The analysis combines exploratory visualization, 3-year lagged investment rates, effective investment rate construction, and a battery of statistical tests — simple and multivariate regression, Pearson correlation, Welch's t-test, and chi-square tests — to build a layered picture of when and where investment pays off.

![Scatter plot of average GDP growth versus lagged investment rate at the country level](/images/gi-scatter-growth-vs-investment.png)
*Average GDP per capita growth vs. 3-year lagged investment rate (country-level averages, 1993–2023). The flat cloud of points foreshadows the null result from simple regression.*

---

## Data

**Source:** World Bank Open Data (1990–2023).

**Variables:**

| Variable | Definition | Type |
|---|---|---|
| Investment Rate | Gross Capital Formation as % of GDP | Continuous (predictor) |
| GDP Growth | Annual % change in GDP per capita | Continuous (outcome) |
| Region | World Bank geographic classification (7 regions) | Categorical |
| Income Group | World Bank income classification (4 levels) | Categorical |

**Sample:** 214 countries, filtered to 1993–2023 after applying a 3-year lag. Countries without region or income-group assignments were excluded (aggregates, territories). Venezuela was manually assigned to "Upper middle income" where missing.

### Derived Variables

- **3-year lagged investment rate:** $\text{Investment\_Rate}_{t-3}$ — captures the delayed effect of capital accumulation on growth.
- **Effective investment rate:** $EIR_t = \frac{\text{GDP\_Growth}_t}{\text{Investment\_Rate}_{t-3}}$ — a ratio measuring how much growth each unit of investment produces.

---

## Methodology

### Data Preparation

1. **Wide-to-long reshaping** — raw World Bank CSVs pivoted from year-columns to tidy format.
2. **Merging** — GDP growth and investment datasets joined on Country × Year, with NAs dropped.
3. **Lagging** — 3-year lagged investment rates computed per country using `dplyr::lag()`.
4. **Effective rate construction** — GDP growth divided by lagged investment, with Inf/NaN values replaced by NA.
5. **Trimming** — top/bottom 5% of effective rates trimmed for visualization to reduce outlier distortion.

### Exploratory Analysis

- **Regional distributions** — density histograms of effective investment rate for each of 7 World Bank regions, revealing sharply different shapes (tight unimodal in South Asia, flat and dispersed in Europe & Central Asia, right-skewed in Latin America).
- **Income-group distributions** — same analysis across Low, Lower-middle, Upper-middle, and High income groups, showing that higher income corresponds to tighter, more symmetric distributions.
- **Boxplot comparisons** — side-by-side boxplots across regions and income groups with mean markers.

### Statistical Tests

We applied five complementary tests to attack the research question from different angles:

| Test | What It Tests | Level |
|---|---|---|
| **Simple linear regression** | Overall investment → growth relationship | Global |
| **Multivariate regression** | Investment × Income Group interaction effects | Income group |
| **Multivariate regression** | Investment × Region interaction effects | Regional |
| **Pearson correlation** | Linear association strength | Country, region, income |
| **Welch's t-test** | Difference in mean effective rates: High vs Low income | Income group |
| **Chi-square test** | Independence of investment category and income group | Income group |

---

## Key Findings

### Simple Linear Regression: No Overall Relationship

| Metric | Value |
|---|---|
| Slope | ≈ 0 (slightly negative) |
| $R^2$ | **0.003** |
| $p$-value | **0.78** |

**Result:** We fail to reject the null hypothesis. Investment rate alone explains virtually none of the variation in GDP growth globally.

![Simple regression: GDP growth vs lagged investment](/images/gi-simple-regression.png)
*World-level regression of GDP growth on 3-year lagged investment rate. The nearly flat line and wide confidence band confirm no meaningful linear association ($R^2 = 0.003$, $p = 0.78$).*

### Multivariate Regression: Context Matters

When we allow the investment–growth slope to vary by income group and region, the picture changes dramatically:

**By income group:**

| Income Group | Marginal Effect (per 1% investment) | Pattern |
|---|---|---|
| **High income** | **+0.15%** GDP growth | Strongest, most consistent returns |
| Upper middle income | Intermediate | Transitional efficiency |
| Lower middle income | Intermediate | Transitional efficiency |
| **Low income** | **+0.05%** (negative below ~20% investment) | Weak; investment alone insufficient |

**By region:**

| Region | Marginal Return | Interpretation |
|---|---|---|
| **East Asia & Pacific** | **+0.12–0.15%** | Highest ROI; export-led growth, sound institutions |
| **South Asia** | Consistently positive | Favorable demographics, rising tech integration |
| Latin America & Caribbean | Moderate, volatile | Macroeconomic instability, commodity dependency |
| **Sub-Saharan Africa** | **Lowest, occasionally negative** | Capital alone insufficient without structural reform |
| North America & Europe | High intercept, low slope | Mature economies; growth via productivity, not capital accumulation |

![GDP growth response to investment by world region](/images/gi-multivariate-region.png)
*Predicted GDP growth by investment rate and region. East Asia & Pacific shows the steepest positive response; Sub-Saharan Africa the flattest.*

### Correlation Analysis

Pearson correlation tests across **180 countries** (minimum 10 observations each):

| Correlation Strength | Number of Countries |
|---|---|
| Strong ($\|r\| \geq 0.7$, $p < 0.05$) | **5** |
| Moderate ($\|r\| \geq 0.4$, $p < 0.05$) | 27 |
| Weak ($\|r\| \geq 0.2$, $p < 0.05$) | 15 |
| Very weak ($p < 0.1$) | 14 |
| **Not significant** | **113** |
| Insufficient data | 6 |

**The vast majority of countries show no meaningful correlation** between investment and growth. Even at the regional and income-group level, correlations are at best weak.

### Welch's t-test: High vs Low Income

| Metric | Value |
|---|---|
| Mean (High income) | 0.0882 |
| Mean (Low income) | 0.1306 |
| $t$-statistic | −0.35 |
| $p$-value | **0.7285** |
| 95% CI for difference | [−0.282, 0.197] |

**Result:** No significant difference in effective investment rates between income groups. The confidence interval spans zero.

### Chi-Square Test: Investment × Income Group

| Metric | Value |
|---|---|
| $\chi^2$ statistic | **185.04** |
| Degrees of freedom | 4 |
| $p$-value | **< 2.2 × 10⁻¹⁶** |

**Result:** Reject independence. Investment *patterns* (high vs. low) are strongly associated with income group — high-income countries are far more likely to have high investment rates. The relationship exists at the categorical level, even though it doesn't translate into a linear growth effect.

| | High Income | Low Income | Lower Middle | Upper Middle |
|---|---|---|---|---|
| **High Investment** | 992 | 156 | 653 | 719 |
| **Low Investment** | 800 | 430 | 582 | 739 |

---

## Discussion

### What This Means

The central finding is a **negative result with rich substructure**: investment does not universally drive growth, but it does in specific contexts. This matters because:

1. **Policy implications differ by context.** Telling low-income countries to "invest more" without addressing institutional quality, governance, and human capital is unlikely to produce growth. The ~20% threshold in low-income countries suggests a minimum scale below which investment is ineffective.

2. **The chi-square result resolves an apparent paradox.** Investment *levels* are associated with income group (wealthier countries invest more), yet investment *returns* are not linearly related to growth globally. This means the investment–growth channel is mediated by institutional quality, not just quantity.

3. **Regional heterogeneity is not noise — it's the signal.** East Asia's high returns and Sub-Saharan Africa's low returns reflect structural differences (export orientation, governance, infrastructure) that a global regression washes out entirely.

---

## Limitations

- **Omitted variables.** The analysis controls for region and income group but lacks direct measures of institutional quality, human capital, trade openness, or political stability — all known mediators of investment effectiveness.
- **Endogeneity.** Investment and growth are likely jointly determined. The 3-year lag partially addresses reverse causality but does not eliminate it. Instrumental variable methods would strengthen causal claims.
- **Aggregation.** Gross Capital Formation includes both public and private investment, infrastructure and financial assets. Decomposing by investment type could reveal which components actually drive growth.
- **Panel structure underexploited.** While we use lagged variables and fixed effects in some specifications, a full panel econometrics treatment (GMM, Arellano–Bond) would be more rigorous for dynamic panel data.

---

## Future Work

1. **Instrumental variable regression** using geographic or institutional instruments to address endogeneity.
2. **Decomposition by investment type** — separating infrastructure, education, and R&D investment to identify which channels drive growth.
3. **Threshold regression models** to formally estimate the investment rate below which returns turn negative in low-income countries.

---

## Tools

**Language:** R  
**Data wrangling:** tidyverse (dplyr, tidyr, readr)  
**Visualization:** ggplot2, patchwork, ggthemes  
**Statistical testing:** base R (lm, cor.test, t.test, chisq.test), car, moments, plm (panel data), ggeffects, multcomp  
**Data source:** World Bank Open Data

---

## Links

- [GitHub Repository](https://github.com/apayanieu/good_investments)

---

## Publication & Attribution

This work was submitted as a final project for the **Probability and Statistics** course at IE University, Spring 2025. Data sourcing, cleaning, analysis, and visualization were done collaboratively by all authors. AI tools (ChatGPT) were used for code debugging and interpretation guidance, with all outputs reviewed and validated by the team.

