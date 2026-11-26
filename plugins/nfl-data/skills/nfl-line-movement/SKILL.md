---
name: nfl-line-movement
description: "Use when the user asks how an NFL spread, total or moneyline moved during the week, in fan wording: 'how did the Chiefs line move', 'did the total drop for Bills Chiefs', 'opening vs closing spread for Sunday', 'which book moved first', 'what are the alternate spreads', 'did the line change after the QB news', 'line history for Thursday night'. Lays out snapshots by sportsbook, opening against closing, key numbers crossed (3 and 7), the size and timing of each move, and alt-line ladders, as information. Never says which side to take or reads who is betting. Not for player prop movement (see nfl-player-props-research), game previews (see nfl-game-preview), injuries (see nfl-injury-impact) or college football (a school such as Alabama or Ohio State is not an NFL team and these tools do not cover it)."
metadata:
  version: 2.2.0
  author: SportsDataHQ
---

# NFL line movement

How an NFL game's spread, total and moneyline changed over the week, by sportsbook, and what the alternate-line ladders look like. Information about the market only. The NFL only; college games are not available through these tools, so say so if asked.

## Before you start
- For anything across games, seasons or books, apply the sports-data-preflight checks first (ids from lookups, the season convention, one league, enough data).
- Tools come from the EndzoneAPI MCP server. Lines move all week; always call, never recall.
- `season` is the year the regular season starts (2025 = 2025-26). Game ids look like `2025_12_BUF_KC` (season, week, away, home); get them from `get_schedule` or `get_games`.
- Snapshots are hourly captures where a line changed, plus the latest; `all_snapshots: true` returns every capture. Per-book snapshots begin with recent seasons; earlier seasons hold the consensus closing line only, so say what coverage a past game has.

## Default path (about 37 credits)
1. **Find the game.** `get_schedule(team, week)` (2 cr) for the game id, kickoff and current line. For a past game `get_games(season, week, team)` (5 cr).
2. **The movement.** `get_line_movement(game_id)` (25 cr, the most expensive tool): each bookmaker's snapshots with `captured_at`, moneyline, spread, total and prices. Pass `bookmaker` for one book. If the user only wants open and close, use `get_odds` (10 cr) instead and skip this call.
3. **Opening and closing, all books.** `get_odds(game_id)` (10 cr): consensus closing and each book's opening, closing and latest line.
4. **Alt-line ladders (when asked).** `get_alt_lines(game_id, market, snapshot_type)` (10 cr): every point on each book's alternate spread or total ladder with its price, for 2023 on. `market`: `alternate_spreads`, `alternate_totals`.
5. **Period lines (optional).** `get_market_odds(game_id, market)` (10 cr): first half, first quarter and quarter-total lines as history.
6. **Context (optional).** `get_injuries(team, week)` (2 cr) for the report entries to line up in time with a move; the report and the move can coincide but the data does not link them.

## How to read a move
- **Size:** closing minus opening, in points. Half a point is small; a full point or more is large for an NFL spread or total. Say the numbers.
- **Key numbers:** NFL margins cluster on 3 and 7, so a move from -2.5 to -3.5 crosses 3 and matters more than -4.5 to -5.5. State which key numbers a move crossed.
- **Direction:** say "the home spread went from -3.5 to -5.5", not "money came in".
- **Timing:** list the capture times the move happened between. Lines are often quiet from Monday until news or Sunday morning; a step at one capture differs from a drift over many.
- **Across books:** several books moving on the same captures describes a shared repricing; one book alone then the others following is a lead and follow. Describe the timing you can see. The data holds prices, not who bet, so "sharp" or "public" money cannot be read from it and must not be claimed.
- **Steam vs drift (descriptions only):** steam is a fast move of a point or more across several books within one or two captures; drift is a slow change over many. Label by what the captures show, with the times.
- **Price vs line:** a spread can hold at -3 while the price moves from -105 to -120. Report line and price; the implied probability moved even though the line did not.
- **Alt ladders:** show rows (point, price) per book around the main line and where books differ at the same point. Older games have one capture, so opening and closing ladders match.
- **Weather and rest:** `get_game_detail(game_id)` (10 cr) has weather and officials; use it only when the user asks why, and word any link as "coincided with".

## Output template
```
<AWAY> at <HOME> | week <n>, <date, kickoff ET> | <venue>
Open to close: spread <home open> to <home close> (<+/- x>, crossed <3 / 7 / none>) | total <open> to <close> (<+/- x>) | home ML <open> to <close>
By book: <book: open -> close for spread and total>, <n> books
Timeline: <captured_at: what changed>, in order returned
Pattern: <step / drift / books moved together or one lead and follow>, with the capture times
Alt ladder (closing): <book: points and prices around the main line>
Context: <injury report entries with times, "coincided with">
Data: get_schedule, get_line_movement, get_odds, get_alt_lines | season <yyyy>, week <n>
```

## Rules
- Information only: no picks, leans, "value", stakes, payouts, "sharp money" claims or sportsbook links. If the user asks what to bet, say this skill shows the market and offer the data instead.
- Two sources of "opening": the opening line from `get_odds` is the book's own first posted line, and the first snapshot in `get_line_movement` is our first capture, which can be later and different. Label which one a figure comes from, never mix them in one column, and say when a book's first capture is already after it moved.
- Describe what moved and when; do not explain why a move happened or what a bettor should do with it.
- No posted line or too few captures: say how many snapshots exist and stop; don't borrow another game.
- Every number names its tool (the Data line, plus inline where it could be doubted).
- Sequences (last 5, recent form, a run of results, a series of line changes) are quoted in the order the tool returns them, with dates and times; never retell them from memory. Lists in the answer must match the tool rows.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.
