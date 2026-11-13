---
name: odds-analysis
description: "Converts American odds to implied probability, removes the vig (devigging), computes hold, no-vig fair lines, spread and total price conversions, and key-number value for NFL markets. Use when user asks what a price implies, how much vig a book charges, what the fair line is, how to devig a moneyline or spread, how many points a half-point is worth, or expected value from a price. Do not use for finding the best book, see odds-explorer. Do not use for model versus market, see edge-detection. Do not use for props, see prop-modeling."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# Odds Analysis

> **Default data tool:** EndzoneAPI (`endzoneapi-tool`).
> `get_odds` (10 credits per game) returns each book's prices. Math itself uses no credits. For lines you already have: skip the tool.

You are an expert in odds mathematics. Your goal is to turn prices into probabilities and expected value correctly, state the assumptions behind a devig method, and never present a market price as a fair probability without removing the vig.

## When to Use

- "What probability does -1100 imply?"
- "How much is the book charging?"
- "What is the fair price once vig is removed?"
- "Is half a point worth buying at -120?"
- "What is the break-even win rate at -110?"

## When NOT to Use

- Finding the best price across books, see `odds-explorer`
- Model versus market edge, see `edge-detection`
- Props, see `prop-modeling`

## Commands Available

| Command | Use Here | Credits |
|---------|---------|---------|
| `get_odds` | Per-book prices to devig | 10 per game |
| `get_games` | Consensus close, moneylines for a season | 5 |
| `get_alt_lines` | Priced ladders for point-value analysis | 10 |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_implied_probability` | Compute it, the `get_odds` rows include `ml_home_prob` and `ml_away_prob` but those are the raw implied values |
| `get_fair_line` | Devig yourself as below |
| `get_hold` | Compute from the two sides |

The `ml_home_prob` and `ml_away_prob` fields in the real Pinnacle row for the 2025 Week 17 DEN at KC game were 0.1295 and 0.9019. They sum to 1.031, so they still contain the vig. Do not treat them as true probabilities.

## American Odds to Probability

```
if odds < 0:  p = -odds / (-odds + 100)
if odds > 0:  p = 100 / (odds + 100)
```

Worked, from the real Pinnacle moneyline: home +672 -> 100/772 = 0.1295. Away -920 -> 920/1020 = 0.9020. Sum 1.0315, so the hold is 3.15%.

## Devigging a Two-Way Market

Given raw probabilities `a` and `b` with `a + b > 1`:

**Proportional (multiplicative).** `a / (a + b)` and `b / (a + b)`. Simple. It assumes the vig is spread in proportion to probability. For the example: home 0.1295 / 1.0315 = 0.1256, away 0.9020 / 1.0315 = 0.8744.

**Power method.** Find `k` so `a^k + b^k = 1`. Shifts more of the vig off the longshot. Use it when one side is a heavy favorite, because proportional understates the favorite's share of the vig.

**Shin method.** Models insider trading; more complex, sometimes used for very lopsided markets.

Report which method you used. Differences between methods are small near 50/50 and grow with the price gap. For the example, proportional gives the home side 12.56% and the power method (k = 1.094) gives 10.68%, a gap of nearly two points on a +672 price. Compute both and show the range when one side is above -500.

Pinnacle's no-vig probability is the standard benchmark for the true probability, because it takes large bets and runs low hold. Treat it as a good estimate, not truth.

## Spread and Total Prices

Spreads and totals are priced around 50/50 (for example -109 / -102 on a spread, -114 / +101 on a total, both from the real Pinnacle row). Devig the pair the same way: two raw probabilities, divide by their sum. A -110 / -110 market has hold of 4.76%; break-even win rate at -110 is 52.38%.

```
break_even(odds): -odds / (-odds + 100) for negative odds
```

## What a Point Is Worth

In the NFL the final margin lands exactly on certain numbers far more often than a smooth distribution would predict. 3 is the most common, then 7, then 6 and 10. A half-point move from 2.5 to 3.5 crosses the biggest mass; 3.5 to 4.5 crosses a smaller one. Estimate it from data: pull seasons with `get_games`, take `result` (home minus away), and count the share of games whose margin equals each number. Use that empirical table to price a half-point, and re-run it on recent seasons since scoring environments shift.

To price an alternate spread from `get_alt_lines`, compare its devigged probability with the main line's. Buying points is fair when the extra probability times the payout beats the added price. Do the arithmetic, do not rely on a rule of thumb.

## Expected Value

For odds with decimal payout `b` (profit per unit risked) and your probability `p`:

```
EV per unit = p * b - (1 - p)
b = 100 / -odds  if odds < 0
b = odds / 100   if odds > 0
```

Example: probability 0.55 at -110: b = 0.909, EV = 0.55 x 0.909 - 0.45 = +0.050 per unit. At probability 0.5238 EV is zero.

The EV is only as good as `p`. A probability from an uncalibrated model overstates EV (see `probability-calibration`).

## Moneyline to Spread Rough Conversion

Do not use a remembered table. Fit one from data: take seasons of closing spreads and moneylines from `get_games`, devig, and fit a logistic curve of win probability on the spread. The curve you fit reflects the era and the key numbers.

## Credit Usage

| Operation | Credits |
|-----------|---------|
| Devig one game from one book | 10 (`get_odds`) |
| Season of consensus lines to fit a curve | 5 (`get_games`) |
| Alternate ladder for one game | 10 |

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "-110 means 50%" | It means 52.38% | Always convert, then devig |
| "Implied probability is the true probability" | Includes the hold | Devig |
| "Average the two books' prices" | Averages are not a market | Devig per book, then compare |
| "Same method for -1100 and -110" | Proportional distorts heavy favorites | Show proportional and power |
| "A half point is always worth 2%" | Depends on the number crossed | Estimate from margin frequencies |

## Output Format

```
DEN @ KC moneyline (Pinnacle close): KC +672 / DEN -920
Raw: 0.1295 / 0.9020, hold 3.15%
No-vig (proportional): KC 12.56% / DEN 87.44%
No-vig (power, k=1.094): KC 10.68% / DEN 89.32%
Fair price (proportional): KC +696 / DEN -696
Method note: [proportional / power], favorite above -500
```

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| Fair probability | Compare to your model | `edge-detection` |
| Hold across books | Pick the best price | `odds-explorer` |
| A season of fair lines | Closing-line testing | `backtesting` |
