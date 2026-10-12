# Cost Ladder

Open-source TradingView indicator (Pine Script v6) that replays one entry signal through a shadow ledger and re-prices the same trades at 0x, 1x and 2x of your trading costs, so you can see whether the signal still shows an edge once costs are in.

Published on TradingView: https://www.tradingview.com/script/kQRPTxtz-Cost-Ladder-Does-a-Signal-Survive-0x-1x-and-2x-Costs/

## What it measures

1. **Gross return of every trade.** One position at a time. The return of a trade is `direction * (exit / entry - 1)`, before any cost.
2. **Round-trip cost of every trade.** Computed from the commission and slippage inputs and the trade's own entry and exit prices (see "Cost model").
3. **Net return at three cost levels.** For `k = 0, 1, 2`, each trade's net return is `x = gross - k * cost`. The three rows of the table are the same trades, priced with a different `k`.
4. **Statistics per level.** Number of trades `n`, mean `x`, sample standard deviation `sd`, the 95% interval `mean +/- 1.96 * sd / sqrt(n)`, win rate (share of `x > 0`) and profit factor (`sum(x > 0) / |sum(x < 0)|`).
5. **Break-even round trip.** The cost per trade at which the mean net return is zero, which is the mean gross return. If it is not above zero the footer reads "negative before costs". Otherwise it also shows how many times your own costs fit inside it (`mean gross / mean cost at 1x`).

The signal is either a built-in demo (EMA 9 crossing EMA 21, there only so the script works out of the box) or any plot from another indicator, read through the `External plot` input: positive means a long request, negative a short request, zero or `na` no request.

## Ledger rules

The ledger is a shadow of a strategy: it keeps its own state and does not use the Strategy Tester. It updates only on confirmed bars, in this order:

1. A pending exit closes the position at this bar's open.
2. A pending entry opens a position at this bar's open, if flat. The pending entry is cleared either way.
3. If the position has been held for the chosen number of bars (the entry bar counts as bar 1), it closes at this bar's close.
4. This bar's request is read. If a position is open and the raw signal is the opposite direction, an exit is queued for the next open, and a new entry in the opposite direction is queued if the `Direction` setting allows it. If flat and the signal is allowed, an entry is queued. Otherwise nothing is queued.

Consequences:

- One position at a time. Same-direction signals while a position is open are ignored.
- Entries and signal-based exits happen at the next open, never on the signal bar.
- An opposite signal that `Direction` does not allow still closes the position, but opens nothing.
- A signal that stays on after a time exit opens a new trade at the next open.
- A position still open on the last bar is not counted.
- A trade with a missing or non-positive entry or exit price is skipped.

## Cost model

```text
gross return     = direction * (exit / entry - 1)
round trip at 1x = 2 * commission% / 100 + slippage_ticks * mintick * (1 / entry + 1 / exit)
net at level k   = gross return - k * round trip,   k = 0, 1, 2
```

Commission is a percentage of price charged on each side, so the round trip pays it twice. Slippage is a number of ticks charged against you on each side, converted into a return with the entry and exit prices. The defaults (0.05% and 1 tick per side) are placeholders: replace them with the costs of your own market and broker.

## Settings

| Input | Default | Meaning |
| --- | --- | --- |
| Signal source | Built-in demo: EMA 9/21 cross | `Built-in demo: EMA 9/21 cross` or `External plot`. |
| External plot | close | Source used when the signal source is `External plot`. Positive = long request, negative = short request, zero or `na` = no request. |
| Direction | Long and short | `Long and short`, `Long only` or `Short only`. |
| Time exit (bars) | 10 | Bars a position is held before it closes at the close (1 to 500). |
| Commission per side (% of price) | 0.05 | Percentage of price charged on each side. |
| Slippage per side (ticks) | 1 | Ticks charged on each side, using the symbol's minimum tick. |
| Minimum trades for a verdict | 30 | Below this many trades the verdict is "too few". |
| Mark entries on the chart | on | Small triangles labeled `L` or `S`, drawn only on confirmed bars. |
| Table position | Bottom right | Four corners. |

## How to read the table

One row per cost level:

| Column | Meaning |
| --- | --- |
| Costs | `0x (none)`, `1x (yours)` or `2x`. |
| Round trip % | Mean round-trip cost at that level, in percent of price (three decimals). |
| Trades | Closed trades in the ledger. |
| Mean net % [95% CI] | Mean net return per trade in percent, with the 95% interval in square brackets. |
| Win rate | Share of trades with a net return above zero. |
| Profit factor | Gross profit divided by gross loss at that level; `inf` when there is no losing trade. |
| Verdict | `positive` (green) when the whole interval is above zero; `negative` (red) when it is below zero; `not distinguishable from zero` when it contains zero; `too few` when the number of trades is below the minimum. |

Two footer rows follow. The first gives the break-even round trip (with the multiple of your costs when it can be computed), the signal, the exit rule and the direction. The second states that this is a shadow ledger, not the Strategy Tester.

## Files

- `cost_ladder.pine`: the indicator.
- `TRADINGVIEW_PUBLICATION.md`: title, description text and checklist for publishing.
- `LICENSE`: MIT.

## Check it yourself

1. Add the indicator to a liquid chart with a long history from the Pine Editor and confirm it compiles without errors or warnings.
2. Set commission and slippage to 0. The three rows must be identical and the footer shows the break-even without a multiple.
3. Count the entry marks. They should equal `Trades`, or `Trades` plus one if a position is still open on the last bar.
4. Set the time exit to 1 bar. Each trade is then the open-to-close return of its entry bar: check one by hand.
5. Add a cost: each row's mean net return must equal the 0x mean minus that row's "Round trip %", up to rounding.
6. To check for look-ahead, use Bar Replay: start a few hundred bars back and step forward. Marks must not disappear once a bar has closed, and `Trades` should only change when a bar closes.

Result on 12 Oct 2026, BTCUSDT (Binance) 1 hour, default settings (demo EMA 9/21 cross, long and short, 10-bar time exit, 0.05% commission and 1 tick per side), 1,090 trades. Mean net return per trade in %, 95% CI in brackets:

| Costs | Round trip % | Mean net % [95% CI] | Win rate | Profit factor | Verdict |
|---|---|---|---|---|---|
| 0x | 0.000 | +0.02 [-0.06, +0.10] | 42.0% | 1.04 | not distinguishable from zero |
| 1x | 0.100 | -0.08 [-0.16, +0.00] | 39.4% | 0.85 | not distinguishable from zero |
| 2x | 0.200 | -0.18 [-0.26, -0.10] | 35.6% | 0.71 | negative |

Break-even round trip: +0.02%, about 0.2x these costs. On SPY daily with the same settings (367 trades) the demo is already negative before costs. These are results for one demo signal, not a statement about any strategy. Run it on your own signal and read your own numbers.

## Limitations

- The ledger is a simplified model: fills at the open or close of chart bars, no gaps, no partial fills, no financing, no position sizing. The next open is a price on the chart, not a guaranteed execution.
- Every trade carries the same cost model. Real costs change with volatility, size and time of day, and the spread is only what you enter as slippage.
- The interval is a normal approximation. Few trades, fat tails and serial dependence between trades make it less reliable; there is no correction for any of them.
- Many settings and many signals mean many comparisons. About 1 interval in 20 excludes zero by chance at the 95% level, and the script applies no multiple-testing correction.
- Only bars loaded on the chart are counted, so history depth, symbol and timeframe change `Trades`.
- The demo EMAs start at the first loaded bar, so the first few bars can carry meaningless crosses. The demo is a placeholder, not a recommendation.
- An external plot must not repaint. The script reads it on confirmed bars but cannot check how it was produced.
- It reports counts and statistics, not a trading rule. Not financial advice.
- The script has not been compiled outside TradingView; compile it in the Pine Editor before relying on it.

## License

MIT. See `LICENSE`.
