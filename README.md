# A/B Testing Projects

Two A/B test analyses in Python, each with a Looker Studio dashboard.

| Project | Question | Notebook | Result |
|---|---|---|---|
| **Cookie Cats** | Should the first gate in a mobile game be at level 30 or level 40? | [cookie_cats.ipynb](cookie_cats.ipynb) | Keep the gate at **level 30** |
| **Fast Food Marketing** | Which of three promotions sells the most of a new menu item? | [Marketing_campaign.ipynb](Marketing_campaign.ipynb) | Roll out **Promotion 1** |

Each notebook follows the same structure:

1. **Goal**: what the test is trying to decide
2. **Target metric**: what we measure, and why
3. **Calculations**: data checks, exploration, statistical tests, confidence intervals
4. **Decision**: results, recommendation and limitations

---

## 1. Cookie Cats: gate at level 30 vs level 40

**Data:** 90,189 players of the mobile game *Cookie Cats*, randomly split into two groups at install.

| Column | Meaning |
|---|---|
| `userid` | player id |
| `version` | `gate_30` (control) or `gate_40` (treatment) |
| `sum_gamerounds` | game rounds played in the first 14 days |
| `retention_1` | did the player return 1 day after installing? |
| `retention_7` | did the player return 7 days after installing? |

**Primary metric:** 7-day retention.

**Methods**
- Data checks: missing values, duplicates, sample ratio mismatch (chi-square), outlier removal (one player with 49,854 rounds)
- Retention: two-proportion z-test with 95% confidence intervals, confirmed with a bootstrap (2,000 resamples)
- Game rounds: Mann–Whitney U test, because the data is very skewed

**Results**

| Metric | gate_30 | gate_40 | p-value | Significant? |
|---|---|---|---|---|
| 1-day retention | 44.8% | 44.2% | 0.074 | No |
| **7-day retention** | **19.0%** | **18.2%** | **0.002** | **Yes** |
| Median game rounds | 17 | 16 | 0.051 | No |

**Recommendation:** keep the gate at level 30. Moving it to level 40 lowers 7-day retention by about 0.8 percentage points (95% CI 0.3–1.3 pp).

**Limitations:** no revenue data; many players never reach level 30, which dilutes the effect; the sample-ratio check flagged a small imbalance (p ≈ 0.009).

---

## 2. Fast Food Marketing Campaign: Promotion 1 vs 2 vs 3

**Data:** weekly sales of a new menu item at 137 stores in 10 markets over 4 weeks. Each store ran one of three promotions.

| Column | Meaning |
|---|---|
| `MarketID`, `MarketSize` | market the store is in, and its size |
| `LocationID`, `AgeOfStore` | store id and age in years |
| `Promotion` | promotion the store ran (1, 2 or 3) |
| `week` | week of the test (1–4) |
| `SalesInThousands` | sales of the new item that week (thousands of $) |

**Primary metric:** average weekly sales per store. The four weeks are averaged so each store, the unit that was randomised, counts once.

**Methods**
- Data checks: sample ratio mismatch, balance of market size and store age across groups
- One-way ANOVA (is there any difference?), then Welch's t-tests for each pair of promotions
- **Bonferroni correction** for three comparisons (98.3% confidence intervals)
- Robustness check: regression controlling for market (market fixed effects)

**Results** (difference in average weekly sales per store)

| Comparison | Simple t-test | Controlling for market |
|---|---|---|
| Promotion 1 vs 2 | **+$10.8k**, significant | **+$9.7k**, significant |
| Promotion 3 vs 2 | **+$8.0k**, significant | **+$4.8k**, significant |
| Promotion 1 vs 3 | +$2.7k, not significant | **+$4.9k**, significant |

**Recommendation:** roll out Promotion 1. Promotion 2 is clearly the weakest. Promotion 1 beats Promotion 3 once market differences are taken into account, and it led in every one of the four weeks.

**Limitations:** only four weeks of data; no cost data for each promotion; possible cannibalisation of other menu items is not measured.

---

## Dashboard

The results are summarised in a Looker Studio dashboard:
**[A/B Test Dashboard](ADD-YOUR-LOOKER-STUDIO-LINK-HERE)**

---

## Files

| File | Description |
|---|---|
| `cookie_cats.ipynb` | Cookie Cats analysis |
| `Marketing_campaign.ipynb` | Fast Food marketing analysis |
| `cookie_cats.csv` | original Cookie Cats data ([Kaggle](https://www.kaggle.com/datasets/mursideyarkin/mobile-games-ab-testing-cookie-cats)) |
| `WA_Marketing-Campaign.csv` | original Fast Food data ([Kaggle](https://www.kaggle.com/datasets/chebotinaa/fast-food-marketing-campaign-ab-test)) |
| `cookie_cats_clean.csv` | Cookie Cats data with the outlier removed and retention as 1/0, for Looker Studio |
| `cookie_retresults.csv` | retention test results (difference, CI, p-value), for Looker Studio |
| `marketing_locations.csv` | Fast Food data averaged to one row per store, for Looker Studio |
| `weekly_cumulative.csv` | Fast Food target metric recalculated as each week is added, for Looker Studio |
| `weekly_effects.csv` | Fast Food market-adjusted treatment effect for each week, with CIs, for Looker Studio |
| `AB_test_glossary.docx` | glossary of A/B testing terms |

---

## How to run

1. Install Python 3 and the libraries:
   ```bash
   pip install pandas numpy scipy statsmodels matplotlib jupyter
   ```
2. Open a notebook in Jupyter or VS Code.
3. Click **Run All**. The CSV files must be in the same folder as the notebooks.

Built with: Python, pandas, NumPy, SciPy, statsmodels, matplotlib, Looker Studio.
