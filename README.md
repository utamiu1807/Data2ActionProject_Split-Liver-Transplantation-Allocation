# Split liver allocation: Standard UCB vs. L-UCB

Course project for *From Data to Action* (CMU Heinz). It asks whether a multi-armed bandit that models hospital **learning curves** (L-UCB, from Tang, Li, Scheller-Wolf & Tayur) allocates split livers better than standard UCB. It uses three Pittsburgh adult programs as arms: UPMC, AGH and VAPHS.

## Files

| File | What it does |
|---|---|
| `1._EDA_basic_model_FINAl.ipynb` | Exploratory analysis of the STAR liver data. It checks each input of the simulation against real data. |
| `2.Simulation.ipynb` | Simulates both algorithms on the three programs and compares them, with sensitivity and robustness checks. It uses no patient data. |

## 1. EDA notebook (STAR data)

Checks that the simulation's assumptions are reasonable:

1. **Horizon T**: how many split livers arrive per year, nationally and in Pittsburgh (so T = 1,000 can be read as years of activity). Also the waiting-list context for adult liver candidates and how many split-liver segments go untransplanted.
2. **Reward**: the one-year graft survival rate for split and whole livers, by recipient age.
3. **Arms and α**: the three Pittsburgh programs in STAR, their adult split volume, and how STAR survival compares with the published SRTR program-specific survival used as α (0.890, 0.934, 0.884).
4. **Learning curve ω**: whether survival rises with a center's prior split experience. A year-adjusted model finds no significant effect, so ω is not fitted from data and is instead drawn from the 1–14 range that Tang et al. elicited from surgeons.
5. **Covariate prior**: whether living-donor volume predicts split-graft survival (it does not appear to).

**Data.** The STAR files are restricted under a Data Use Agreement and are **not** in this repository. To rerun the notebook you need your own approved STAR liver, deceased-donor and center-ID files, and you must edit the path constants at the top of the notebook (`LIVER_DIR`, `CENTER_DTA`, `DD_DIR`, `WLH_DAT`, `OUT`). The notebook prints only aggregated counts; any cell below 11 is shown as `<11`.

Requires: Python 3, numpy, pandas, matplotlib, statsmodels, and an HTML parser for `pandas.read_html` (lxml).

## 2. Simulation notebook

Success probability of a program is `θ(s) = α · g_ω(s + s0)` with `g_ω(x) = 1 / (1 + e^-(x - ω))`, where `s` is the number of split cases the program has done, `s0` is its starting experience, and ω is the number of cases to reach half of its potential. Regret is the survival probability lost relative to always choosing the best program (AGH). T = 1,000 livers per run.

| Section | What it does |
|---|---|
| 2. Baseline | ω ~ U(1,14) per program, c = 0.5, no starting experience. Shows how each algorithm's estimate of each program's success rate behaves and compares cumulative regret. |
| 3. Sensitivity | A: the world's ω swept while L-UCB must *assume* one value; B: exploration scale c swept; C: c × ω grid. |
| 4. Fixed parameters | Same as the baseline but with starting experience from STAR (UPMC 30, AGH 0, VAPHS 9). |
| 5. Comparison | Baseline vs. fixed side by side; 5b repeats the headline numbers with L-UCB assuming ω rather than being told it. |
| 6. Robustness | Uncertainty in α and which program is truly best, horizon T (200 to 3,600), and different ω per program. Ends with a verdict table. |

Differences are paired by run (same ω, same rewards) and called significant when more than 1.96 standard errors from zero.

**Main takeaway.** Without starting-experience differences the two algorithms are similar on average, though L-UCB is much less variable across runs. L-UCB's clear advantage appears when programs start with different experience. This advantage depends on AGH being the true best program. Results also depend on c and on what ω L-UCB assumes, so read the sensitivity sections before drawing policy conclusions. The α values come from small survival differences (about 4–5 points), and the model leaves out capacity and urgency.

**Running it.** Requires Python 3, numpy, pandas and matplotlib. Run the cells top to bottom. The full notebook takes on the order of an hour; for a quick check, lower `N_RUNS`, `N_RUNS_UNK`, `N_RB` and `N_RUNS_C` near the top of each section.

## Reference

Tang, Li, Scheller-Wolf & Tayur, *multi-armed bandits with endogenous learning curves* (see the project folder for the paper).
