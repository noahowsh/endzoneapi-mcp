---
name: probability-calibration
description: "Checks and repairs predicted probabilities for NFL outcomes: reliability tables, Brier score, log loss, expected calibration error, Platt scaling, isotonic regression, and sample-size limits. Use when user asks if a model's 65% means 65%, how to calibrate, Brier score, log loss, why a model is overconfident, or before sizing bets from model output. Do not use for building the model, see feature-engineering. Do not use for time-ordered testing, see walk-forward-validation. Do not use for turning odds into probability, see odds-analysis."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# Probability Calibration

> **Default data tool:** EndzoneAPI (`endzoneapi-tool`) for outcomes only: `get_games` (5 credits) returns `result`, `spread_line`, `total`, `total_line` for scoring predictions. Calibration itself is local code.
> For your own predictions CSV: no credits used.

You are an expert in probabilistic forecast quality. Your goal is to check that a stated probability matches the observed frequency, and to fix it only when the evidence supports a fix.

## When to Use

- "My model says 70% for the home team, is that right?"
- "Is my model overconfident?"
- "Compute Brier score and log loss"
- "Should I use Platt scaling or isotonic?"
- "Are my cover probabilities good enough to size bets?"

## When NOT to Use

- Constructing inputs, see `feature-engineering`
- Choosing training windows, see `walk-forward-validation`
- Devigging market odds, see `odds-analysis`

## Commands Available

| Command | Use Here | Credits |
|---------|---------|---------|
| `get_games` | Realized outcomes and consensus lines for scoring | 5 per query |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_calibration` | Compute from your predictions and `get_games` outcomes |
| `get_model_probabilities` | Not provided; bring your own |

## Metrics

For predictions `p_i` in [0, 1] and outcomes `y_i` in {0, 1}:

```
Brier    = mean((p - y)^2)
LogLoss  = -mean(y * ln(p) + (1 - y) * ln(1 - p))
```

Reference points for a binary game outcome at 50/50 forecasts: Brier 0.25, log loss 0.693. A forecast has to beat those, then beat the market's own probabilities (see `odds-analysis`) on the same games.

**Reliability table.** Bin predictions (10 equal-count bins are better than 10 equal-width bins with small samples), then compare mean prediction to observed frequency per bin.

```python
df["bin"] = pd.qcut(df.p, 10, duplicates="drop")
rel = df.groupby("bin").agg(pred=("p", "mean"), obs=("y", "mean"), n=("y", "size"))
```

**Expected calibration error:** `sum(n_bin / N * |obs - pred|)`. Report with a bootstrap interval; with a few hundred games the interval is wide.

## Reading the Table

| Pattern | Meaning | Fix |
|---------|---------|-----|
| Predictions spread wider than outcomes (80% bin hits 68%) | Overconfident | Shrink toward the base rate, Platt scaling |
| Predictions narrower than outcomes | Underconfident | Rare; sharpen with Platt slope above 1 |
| Good in the middle, bad at the tails | Few tail samples | Do not fix tails with a flexible method on tiny n |
| Systematic offset (home always 3 points too high) | Bias | Intercept in Platt scaling |

## The Sample-Size Problem in Football

An NFL season has 272 regular season games. In 10 bins, that is about 27 games a bin per season. The standard error of an observed frequency near 0.5 at n = 27 is about 0.10. A bin at 0.60 predicted and 0.50 observed is inside the noise. Pool several seasons, use fewer bins (5), and show intervals. Do not conclude miscalibration from one season of one bin.

## Repairs

**Platt scaling (default).** Fit a logistic regression of outcome on the logit of your prediction:

```python
from sklearn.linear_model import LogisticRegression
z = np.log(p / (1 - p)).reshape(-1, 1)
cal = LogisticRegression(C=1e6).fit(z, y)      # two parameters: slope and intercept
p_cal = cal.predict_proba(z)[:, 1]
```

Two parameters, stable on a few hundred games. A slope below 1 confirms overconfidence.

**Isotonic regression.** Flexible monotone fit. Needs thousands of rows; on a few hundred games it overfits steps. Use only with enough data.

**Fit the calibrator on out-of-sample predictions.** The calibration map must be learned on predictions from a walk-forward run (see `walk-forward-validation`), then applied forward in time. Fitting it on the same data used to score it reports an inflated gain.

## Regression Predictions

If the model predicts a margin, convert to a win probability with a normal approximation:

```
P(home wins) = Phi(predicted_margin / sigma)
```

with `sigma` estimated from your own residuals (`result` minus predicted margin). NFL margins are not normal around the spread; key numbers (3, 7) put extra mass at specific margins. For cover probabilities near the spread, estimate the empirical distribution of `result - spread_line` from the seasons you have and use it instead of the normal curve. Compute both and compare on held-out games.

## Using Calibration for Bets

Edges computed from an uncalibrated probability are inflated by the model's own overconfidence. Calibrate first, then compute edge (see `edge-detection`). If calibrated probability and the de-vigged market probability differ by only a point or two, the model is not separated from the market.

## Credit Usage

Five credits per `get_games` query. Everything else is local.

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "Accuracy is 64% so it's good" | Says nothing about confidence | Report Brier and log loss next to the market's |
| "Calibrate on the test set" | Leaks the answer | Calibrate on prior out-of-sample predictions |
| "Isotonic is more flexible so better" | Overfits small samples | Platt first |
| "One season's reliability chart shows bias" | 27 games a bin | Pool seasons, show intervals |
| "Clip probabilities at 0.99" | Log loss punishes confident misses | Check the model, do not hide it |

## Output Format

```
Calibration, out-of-sample, 2022-2025 (N = 1,088 games)
Brier: 0.xxx (market 0.xxx) | Log loss: 0.xxx (market 0.xxx)
Platt slope 0.xx, intercept 0.xx -> [overconfident / fine]
Reliability (5 bins): pred x.xx obs x.xx n=NNN (95% CI x.xx to x.xx) ...
Verdict: [calibrated / adjust / insufficient data]
```

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| Calibrated probabilities | Compare to the market | `edge-detection` |
| Poorly calibrated | Rebuild inputs or add data | `feature-engineering` |
| Calibrated and separated | Simulate betting | `backtesting` |
