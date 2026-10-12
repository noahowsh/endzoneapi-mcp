---
name: edge-detection
description: "Compares model probabilities with devigged NFL market prices to estimate edge, expected value and stake size, and tests whether a disagreement is likely real. Use when user asks if there is an edge on a game, whether the model beats the line, expected value of a bet, Kelly sizing, or how to tell value from noise. Do not use for finding the best price, see odds-explorer. Do not use for devig math, see odds-analysis. Do not use for historical profit tests, see backtesting."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# Edge Detection

> **Default data tool:** EndzoneAPI (`endzoneapi-tool`).
> `get_odds` (10 credits per game) for the market side. `get_schedule` (2 credits) for the games. Your model supplies the probabilities.

You are an expert in comparing a forecast with a market. Your goal is to say how large an estimated edge is, how uncertain it is, and what it would take to be wrong. Edges are estimates, not promises. Sports markets are efficient enough that most apparent edges are model error.

## When to Use

- "My model has KC at 58%, the market says 53%. Is that an edge?"
- "What is the EV at these odds?"
- "How much should I stake?"
- "Is this disagreement big enough to act on?"

## When NOT to Use

- Hunting for the best price, see `odds-explorer`
- Converting odds, see `odds-analysis`
- Evidence from past seasons, see `backtesting`

## Commands Available

| Command | Use Here | Credits |
|---------|---------|---------|
| `get_schedule` | Upcoming games, consensus line | 2 |
| `get_odds` | Per-book prices to devig, opening and latest | 10 per game |
| `get_injuries` | Late information the model may lack | 2 |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_edges` | This skill, from your probability and `get_odds` |
| `get_model_picks` | Not provided; bring your own model |
| `get_sharp_money` | Not available; closing-line movement is a proxy only |

## Method

**Step 1: Get a calibrated probability.** Use `probability-calibration`. An uncalibrated probability produces fake edges.

**Step 2: Get the market's probability.** Pull `get_odds(game_id=...)`, devig each book's two sides (see `odds-analysis`). Use Pinnacle's no-vig number as the market benchmark and the best available price for the bet price.

**Step 3: Edge.**

```
edge        = p_model - p_market_novig
EV per unit = p_model * b - (1 - p_model)       # b = payout per unit at the bet price
```

Two different quantities: `edge` compares your probability with the benchmark; EV uses the price you can actually get. Report both.

**Step 4: Shrink toward the market.** The market contains information your model lacks. A common estimate is a blend:

```
p_final = w * p_model + (1 - w) * p_market_novig
```

Estimate `w` from walk-forward results (see `walk-forward-validation`): regress outcomes on the market logit and the model logit on held-out games, and use the fitted weights. If `w` is near zero, your model adds nothing beyond the market.

**Step 5: Uncertainty.** An edge of 3 points on one game has an error band wider than 3 points. If you can, propagate: bootstrap the model fit, or use the walk-forward log loss gap as a measure of how much information the model has. State a range.

**Step 6: Size.** Kelly fraction for a bet with probability `p` and payout `b`:

```
f* = (p * b - (1 - p)) / b
```

Use a fraction of it, a quarter or less, because `p` is estimated with error and full Kelly assumes it is exact. Cap any single bet at a small share of bankroll. Football has only 17 games a team a season and a Sunday slate is correlated through weather and shared injuries, so avoid treating 8 simultaneous bets as independent.

**Step 7: Check what the line is telling you.** If the market moved against your side since you computed the number, use `get_odds` to see opening vs latest. A line that moves away from you after you bet is a sign you may be on the wrong side of new information, not a proof.

## Reasons a Gap Is Usually Not an Edge

| Cause | Check |
|-------|-------|
| Missing late news (QB, line) | `get_injuries`, the depth chart |
| Stale rating or small early-season sample | Games played, prior weight |
| Different quarterback than the model assumes | Confirm starters |
| Weather not in model | `get_game_detail` |
| Model trained on a different era | Walk-forward by season |
| Probability uncalibrated | Reliability table |

## Closing-Line Value

The most reliable test of whether you have an edge is whether you beat the closing price consistently. If your bets on average get a better number than Pinnacle's close, that is evidence (see `backtesting`). If you only win on average at the closing price, you are paying the vig. A single result says nothing.

## Credit Usage

| Operation | Credits |
|-----------|---------|
| One game market check | 10 |
| A 16-game Sunday slate | 160 |
| Slate plus injuries | 190 |

Check only the games where your model disagrees with the consensus line from `get_schedule` (2 credits per query), then pay for per-book prices there.

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "Model 58 vs market 53 is a 5% edge, bet big" | Model error is larger than the gap | Shrink, quarter Kelly |
| "Raw implied probability is the market" | Includes hold | Devig |
| "All Sunday bets are independent" | Correlated | Cap total exposure |
| "It won, so there was an edge" | Variance dominates one game | Judge by price vs close over many bets |
| "Compare to the soft book's price" | Soft books are not the benchmark | Compare to Pinnacle's no-vig |

## Output Format

```
KC vs DEN -- model 0.61 (calibrated), market no-vig 0.55 (Pinnacle)
Edge: +0.06 before shrinkage; shrunk (w=0.4): +0.02
Best price: [book] [odds], EV +0.0xx per unit at shrunk p
Stake: quarter Kelly = x.x% of bankroll, cap y%
Risks: [late news / sample / correlation]
This is an estimate; one game's outcome does not test it.
```

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| Edge appears real | Test over history | `backtesting` |
| Edge vanishes after shrinkage | Improve the model | `feature-engineering` |
| Need the best book | Shop | `odds-explorer` |
