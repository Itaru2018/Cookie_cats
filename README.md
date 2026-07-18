# Cookie Cats — A/B Test on Gate Placement & Player Retention

Statistical analysis of an A/B test from the mobile game **Cookie Cats**, evaluating whether moving a progression "gate" from level 30 to level 40 affects player retention and engagement.

Repo: [github.com/Itaru2018/Cookie_cats](https://github.com/Itaru2018/Cookie_cats)

## Business Question

Cookie Cats places a gate at level 30 that forces players to wait or pay to continue. The test moves that gate to level 40. **Does this change improve or hurt player engagement and retention — and should it be rolled out?**

## Dataset

- 90,189 players randomly assigned to `gate_30` (44,700) or `gate_40` (45,489) on install
- Metrics: `sum_gamerounds` (rounds played in the first 14 days), `retention_1`, `retention_7`

## EDA Highlights

- `sum_gamerounds` is extremely right-skewed: the single highest value in `gate_30` (49,854 rounds) is ~17x the second-highest (2,961). Rather than dropping it automatically, this was flagged, investigated, and **kept** — with an explicit note that it likely inflates `gate_30`'s mean/variance, and a plan to check its influence later (see Robustness Check below).
- Log, square-root, and sine transforms were all tried on `sum_gamerounds` before settling on **log(1+x)**, which was the only one that meaningfully improved distributional shape.

## Methodology

**1. `sum_gamerounds`**
- Assumption check: both groups have n > 30 (CLT applies), but Q-Q plots on the raw data showed clear non-normality. After log transformation, skewness dropped from 163.7 / 5.97 to 0.10 / 0.11, and kurtosis landed within ±1 for both groups — reasonably normal.
- **Welch's t-test** (two-sided, unequal variance) applied to the log-transformed values.
- **95% CI** computed for the mean difference.
- **Robustness check:** the test was re-run with the extreme `gate_30` outlier removed, to confirm it wasn't driving the result.

**2. Retention (`retention_1`, `retention_7`)**
- Independence confirmed (no duplicate user IDs, random assignment).
- Sample-size condition for the normal approximation checked directly (n·p̂ ≥ 10 and n·(1−p̂) ≥ 10 in every cell) rather than assumed.
- **Two-proportion z-test**, one-sided (`gate_30 > gate_40`), matching the direction already visible in the EDA.
- **95% CI for the proportion difference** computed to check practical, not just statistical, significance.

## Results

| Metric | gate_30 | gate_40 | Test | Result |
|---|---|---|---|---|
| `sum_gamerounds` (log) | mean 2.889 | mean 2.871 | Welch's t-test, two-sided | p = 0.0696 (n.s.); 95% CI includes 0 |
| — same, outlier removed | — | — | Welch's t-test, two-sided | p = 0.0724 (n.s.) — result unchanged |
| Day-1 retention | 44.8% | 44.2% | Two-proportion z-test, one-sided | z = 1.78, p = 0.0372; 95% CI for diff **[-0.0006, 0.0124]** (includes 0) |
| Day-7 retention | 19.0% | 18.2% | Two-proportion z-test, one-sided | z = 3.16, p = 0.0008; 95% CI for diff **[0.0031, 0.0133]** (excludes 0) |

## Conclusion

- **Gameplay volume:** no significant difference, with or without the extreme outlier.
- **Day-1 retention:** one-sided test is significant (p = 0.037), but the confidence interval for the actual proportion difference includes zero — **no meaningful practical difference**.
- **Day-7 retention:** significant and the CI stays entirely above zero — a **small but real** advantage for `gate_30` (roughly 0–1 percentage point).
- **Recommendation:** moving the gate to level 40 is **not supported by the data**. `gate_30` performs marginally better on long-term retention, but the effect is too small on its own to justify the change. Other product/engagement levers should be explored instead.

## Methodology Note

The retention test uses a **one-sided** alternative (`gate_30 > gate_40`) in the same direction already observed in the EDA — worth being upfront about in an interview, since testing a direction after seeing the data technically inflates the false-positive rate versus pre-registering the direction. It doesn't change the bottom line here: the CI is checked independently of the p-value for both metrics, and Day-1's "significant" result is explicitly called out as **not** practically meaningful — the business conclusion holds either way.

## Tech Stack

Python · pandas · numpy · scipy · statsmodels · matplotlib · seaborn

## Repo Structure

```
Cookie_cats/
├── data/
│   └── cookie_cats.csv
├── notebook/
│   └── cookie_cats.ipynb
├── .gitignore
└── cookie_cat.yaml
```

## How to Run

```bash
git clone https://github.com/Itaru2018/Cookie_cats.git
cd Cookie_cats
conda env create -f cookie_cat.yaml
conda activate cookie_cat
jupyter notebook notebook/cookie_cats.ipynb
```
