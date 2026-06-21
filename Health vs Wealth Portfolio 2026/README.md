# Health vs. Wealth: Does Money Buy a Longer, Happier Life?

A data science investigation using WHO and World Bank data across 187 countries and 16 years (2000–2015).

---

## The Question

National wealth and population health are obviously related — but how, and how much? This project tests that relationship across three hypotheses, progressing from physical health to mental health to meaning-making infrastructure. The goal was not to confirm assumptions but to see what the data actually supports.

---

## Data Sources

| Dataset | Source | Notes |
|---|---|---|
| Life expectancy, mortality, vaccination rates | WHO (via Kaggle) | Health metrics reliable; BMI and demographic columns excluded after quality checks |
| GDP per capita | World Bank (direct) | Replaced scrambled Kaggle version |
| Population | World Bank (direct) | Replaced Kaggle version with orders-of-magnitude variance |
| Suicide rates | WHO Global Health Observatory (direct) | |
| Religious composition | Pew Research Center (direct) | 2010 snapshot, 201 countries |

**Key cleaning decisions:**
- GDP and population columns in the original WHO Kaggle dataset failed basic sanity checks and were replaced with World Bank direct downloads
- BMI excluded: 283 rows with physiologically impossible values
- 6 countries excluded for data coverage gaps
- Final dataset: 2,938 observations across 187 countries

---

## Hypothesis 1 — Physical Health & Wealth

**Question:** Does life expectancy increase with GDP per capita?

**Finding: Confirmed.** The relationship is logarithmic, not linear — consistent with the Preston Curve. Gains in life expectancy are largest at low income levels and flatten significantly at high income levels.

| Statistic | Value |
|---|---|
| Pearson r (log GDP) | 0.784 |
| R² | 0.615 |
| p-value | ≈ 0 |

GDP per capita explains 62% of cross-national variation in life expectancy. The remaining 38% reflects healthcare systems, inequality, culture, and other structural factors.

![Life Expectancy vs GDP per Capita](charts/life_exp_vs_gdp_trendline.png)

**Regional patterns worth noting:**

| Region | Pattern |
|---|---|
| Sub-Saharan Africa | Low GDP, low life expectancy, high variance |
| Europe & Central Asia | High GDP, high life expectancy — closely follows trend |
| South & East Asia | Consistently above trend — Japan, South Korea, Sri Lanka outperform their GDP |
| North Africa & Middle East | Below trend at high GDP — Gulf state effect |

The Sub-Saharan Africa variance is analytically significant. Equatorial Guinea ($18,210 GDP / 55.4 years life expectancy) and Seychelles ($15,333 / 73.2 years) share similar per-capita wealth but diverge by nearly 18 years of life expectancy — illustrating that GDP per capita is a misleading metric when inequality is extreme.

---

## Hypothesis 2 — Mental Health & Wealth

**Question:** Do mental health outcomes follow a U-shaped curve relative to wealth — with meaning eroding beyond material comfort?

**Finding: Indeterminate.** The hypothesis cannot be conclusively tested with available country-level data.

![Suicide Rate vs GDP by Region](charts/suicide_vs_gdp_region.png)

The global suicide rate chart shows rates peaking at middle-income levels — an inverted U rather than the predicted U. However, this pattern cannot be interpreted as evidence against the hypothesis for two reasons:

**Proxy failure:** Suicide rate and alcohol consumption are downstream behavioral outcomes with many independent drivers — poverty, post-Soviet collapse, agricultural debt crises, cultural taboo. Neither directly measures internal psychological states of meaning or purpose.

**Systematic underreporting:** Suicide is underreported in Muslim-majority nations for religious and legal reasons. Alcohol consumption is suppressed by Islamic prohibition independently of mental health. Both proxies fail in the same cultural blind spot simultaneously.

**EDA — Regional Breakdown**

A regional analysis was conducted across 6 regions to test whether more granular patterns would emerge. No region produced meaningful explanatory power.

| Region | R² | Note |
|---|---|---|
| Americas | 0.007 | No relationship |
| Europe & Central Asia | 0.015 | Post-Soviet cluster effect |
| North Africa & Middle East | 0.019 | Underreporting suspected |
| South & East Asia | 0.033 | Japan/Korea effect |
| Oceania | 0.049 | Highest R², unreliable due to sample size |
| Sub-Saharan Africa | 0.001 | Complete noise |

Even within Europe & Central Asia — the region with the most culturally homogeneous suicide reporting — the alcohol-suicide correlation reached only r = 0.343, R² = 0.118.

**Conclusion:** No regional breakdown produced meaningful explanatory power. The available proxies are insufficient to test the underlying hypothesis at country level.

---

## Hypothesis 3 — Religion as a Proxy for Meaning

**Question:** Do countries with higher religious participation show lower suicide rates at equivalent GDP per capita levels?

**Rationale:** H2 failed because suicide rate is too blunt a proxy for internal psychological states. H3 reframes the question: rather than measuring mental health directly, it tests whether a known source of meaning — religious participation — produces a detectable signal in the data.

**Finding: Modest but real signal confirmed.**

Countries were binned into three religiosity tiers using natural breaks on `Religious_Prevalence = 100 - Religiously_Unaffiliated` (Pew 2010 snapshot):

| Tier | Threshold | Countries |
|---|---|---|
| Low | < 70% | 14 |
| Medium | 70–95% | 58 |
| High | > 95% | 108 |

**Mean suicide rates step down monotonically across tiers:**

| Tier | Mean Suicide Rate |
|---|---|
| Low | 13.9 |
| Medium | 11.6 |
| High | 9.8 |

![Suicide Rate vs GDP by Religiosity Tier](charts/H3_Suicide_vs_GDP_by_Religiosity_Tier_labeled.png)

![Suicide Rate Distribution by Tier](charts/H3_Suicide_Boxplot_by_Tier.png)

**Regression results by tier:**

| Tier | Pearson r | p-value | Slope | Interpretation |
|---|---|---|---|---|
| Low | 0.040 | n.s. | — | Flat, not significant |
| Medium | -0.067 | 0.04 | — | Weak negative |
| High | -0.259 | < 0.0001 | -3.105 | Significant negative relationship |

The signal is clearest in the High tier: at equivalent GDP levels, highly religious countries show meaningfully lower suicide rates, with a regression slope of -3.105.

**Analytical flags:**

- Islamic underreporting remains a confound — Muslim-majority countries cluster heavily in the High tier, which may suppress reported suicide rates independently of any protective religious effect
- North Korea represents coerced secularism, not genuine low religiosity — it sits in the Low tier and is a known outlier
- The post-Soviet cluster (Medium tier) likely inflates that tier's mean due to alcohol-related suicide patterns, which may exaggerate the step-down between Medium and High
- Correlation ≠ causation — social cohesion, community structure, and cultural conservatism are plausible alternative mechanisms

**Conclusion:** The monotonic step-down and High-tier regression result are consistent with the hypothesis. The signal is real but modest, and the confounds are significant enough that no strong causal claim is warranted.

---

## Skills Demonstrated

- Data sourcing from primary sources (World Bank API, WHO GHO, Pew Research Center)
- Data quality auditing — identifying and replacing corrupted columns, excluding implausible values
- Cross-source dataset merging with country name harmonization
- Exploratory data analysis and hypothesis testing
- Correlation analysis, regression, statistical significance testing
- Data visualization with matplotlib (scatter plots, regression overlays, box plots)
- Honest analytical closure — documenting indeterminate results rather than forcing conclusions

**Tools:** Python, pandas, matplotlib, Jupyter Notebook

---

## Repository Structure

```
health-vs-wealth/
├── README.md
├── notebooks/
│   ├── 02_WHO_Analysis.ipynb
│   ├── 03_WHO_Analysis_H2_Clean.ipynb
│   ├── 04_WHO_Analysis_H3.ipynb
│   └── 05_religion_who_merged_H3.ipynb
├── charts/
│   ├── life_exp_vs_gdp_trendline.png
│   ├── suicide_vs_gdp_region.png
│   ├── H3_Suicide_vs_GDP_by_Religiosity_Tier_labeled.png
│   └── H3_Suicide_Boxplot_by_Tier.png
└── data/
    └── README_data.md  ← source links; raw files excluded due to size
```

---

## Data Availability

Raw data files are not included in this repository due to file size. All datasets are publicly available:

- [WHO Life Expectancy Dataset](https://www.kaggle.com/datasets/kumarajarshi/life-expectancy-who)
- [World Bank GDP per Capita](https://data.worldbank.org/indicator/NY.GDP.PCAP.CD)
- [World Bank Population](https://data.worldbank.org/indicator/SP.POP.TOTL)
- [WHO Global Health Observatory — Suicide Rates](https://www.who.int/data/gho)
- [Pew Research Center — Religious Composition by Country](https://www.pewresearch.org/religion/interactives/religious-composition-by-country-2010-2050/)
