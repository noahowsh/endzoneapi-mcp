---
name: walk-forward-validation
description: "Evaluates sports models in time order: expanding or rolling windows, season-by-season holdouts, bye weeks and season rollover, baselines, and significance. Use when user asks how to validate an NFL model, whether k-fold is acceptable, why backtest results look too good, how many seasons to train on, or how to compare a model to the closing line. Do not use for building features, see feature-engineering. Do not use for betting profit simulation, see backtesting. Do not use for probability checks, see probability-calibration."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# Walk-Forward Validation

> **Default data tool:** EndzoneAPI (`endzoneapi-tool`).
> Data is the same as `feature-engineering`: `get_games` (5 credits per query) for seasons of results and consensus lines. Validation itself uses no credits.

You are an expert in evaluating predictive models on time-ordered data. Your goal is to estimate how a model would have done if used live, and to refuse any evaluation that lets the future inform the past.

## When to Use

- "How do I test my NFL model?"
- "Can I use cross-validation?"
- "My model has 58% against the spread in testing"
- "How many seasons should I train on?"
- "Does my model beat the closing line?"

## When NOT to Use

- Constructing inputs, see `feature-engineering`
- Betting returns, bankroll, drawdown, see `backtesting`
- Checking if 70% means 70%, see `probability-calibration`

## Commands Available

| Command | Use Here | Credits |
|---------|---------|---------|
| `get_games` | Season results, scores, consensus lines for the test and baseline | 5 per query |
| `get_standings` | Sanity check of season records | 2 |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_train_test_split` | Do it in your own code by season and week |
| `get_model_predictions` | Not provided; bring your own |
| `get_closing_line` | `get_games` consensus close |

## Why Random Splits Fail

Random k-fold puts week 12 games in the training set for a week 5 test game. Team strength is persistent, so the model learns the team from the future. In football, a coach, quarterback or scheme carries across weeks. Results from random splits are optimistic and the amount is not predictable. Walk-forward only.

## The Procedure

1. Sort games by kickoff time.
2. Choose an initial training window and a step. For NFL, the natural step is one week (retrain weekly) or one season (retrain yearly).
3. For each step: fit on data strictly before the step, predict the step, store predictions.
4. Concatenate predictions. Evaluate once, on the stitched out-of-sample set.

```python
def walk_forward(df, first_test_season, step="week"):
    preds = []
    for (season, week), test in df[df.season >= first_test_season].groupby(["season", "week"]):
        train = df[(df.season < season) | ((df.season == season) & (df.week < week))]
        model = fit(train)                 # features already shift(1)-safe
        test = test.assign(pred=model.predict(test))
        preds.append(test)
    return pd.concat(preds)
```

Fit hyperparameters inside the training window (nested), not on the final test set. Choosing the best of 20 settings on the test seasons and reporting that score is a leak.

## Football Specifics

**Season rollover.** Rosters, coaches and quarterbacks change. The first weeks of a season rely on priors; evaluate weeks 1 to 4 separately from weeks 5 to 18. A model that is strong only late hides an early-season problem.

**17-game seasons.** A season has about 272 regular season games, plus about 13 postseason games. Test over several seasons. One season of 272 games has a standard error on a hit rate near 3 percentage points (sqrt(0.25 / 272) is about 3.0 points). A 55% result on one season is within noise of 50%.

**Window choice.** Expanding window (all past data) is stable. Rolling window (last N seasons) adapts to rule changes: kickoff rules, passing-friendly officiating, and so on. Compare both by walk-forward error.

**Bye weeks and short weeks.** Rows with a bye the prior week have no prior-week game; handle features for them explicitly.

**Postseason.** Fewer games, different incentives. Report regular season and postseason separately, and do not tune on the postseason.

**Weeks 17 and 18.** Teams with clinched seeds rest starters. The market sees that; a model without lineup information will not.

## Baselines (the comparison that matters)

| Baseline | What It Tests |
|----------|---------------|
| Home team always wins | Floor for accuracy |
| Market-implied probability from the closing line | The hard bar |
| Elo-only model | Does your complexity add value |
| Previous season win percentage | Persistence |

Metrics to report, vs each baseline on the same games:

- **Log loss** and **Brier score** for probabilities (lower is better). Accuracy alone hides confidence.
- **Mean absolute error** of the predicted margin vs the closing spread's error. In public NFL data the closing spread's margin error is a hard benchmark to beat.
- **Calibration** (see `probability-calibration`).

A model that does not beat the closing line on log loss does not produce an edge by betting at the closing line, however well it picks winners.

## Significance

Differences between two models on the same games are paired. Use a paired bootstrap over games (resample game indices, recompute the log loss difference) and report the interval. With a few hundred games, a difference of 0.003 in log loss is not distinguishable from zero. Report the interval, not only the point estimate.

```python
rng = np.random.default_rng(0)
diffs = []
for _ in range(2000):
    idx = rng.integers(0, n, n)
    diffs.append(loss_a[idx].mean() - loss_b[idx].mean())
lo, hi = np.percentile(diffs, [2.5, 97.5])
```

If the interval contains zero, say the models are not separated on this data.

## Multiple Comparisons

Trying 30 feature sets and reporting the best inflates results. Count the variants you tried. Hold out the most recent complete season untouched until the final check, run it once, and report it as is.

## Credit Usage

Data pull only. Five seasons of consensus lines and results: 25 credits through `get_games`. Evaluation is local.

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "Cross-validation is standard" | It is for exchangeable data; games are ordered | Walk forward |
| "Test on 2024 and tune on 2024" | The test is no longer out of sample | Tune only on earlier seasons |
| "58% ATS in one season" | Standard error is about 3 points | Use more seasons and an interval |
| "Win rate is the metric" | Hides confidence | Use log loss and Brier |
| "Skip the closing-line baseline" | It is the only baseline that matters for betting | Always include it |

## Output Format

```
Walk-forward, expanding window, weekly refit, test seasons 2021-2025 (N games)
Log loss: model 0.xxx | closing line 0.xxx | Elo 0.xxx
Paired bootstrap, model minus closing: -0.00x (95% CI -0.0xx to +0.0xx)
By phase: weeks 1-4 [..], weeks 5-18 [..], postseason [..]
Verdict: [separated from the market / not separated], sample caveat
```

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| Model separated from baselines | Check probability quality | `probability-calibration` |
| Model plausible vs close | Simulate betting | `backtesting` |
| Weak model | Revisit inputs | `feature-engineering` |
