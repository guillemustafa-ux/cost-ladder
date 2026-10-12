# Cost Ladder: Does a Signal Survive 0x, 1x and 2x Costs?

Description to paste as is (between the rules):

---
WHAT IT DOES
Cost Ladder replays one entry signal through a shadow ledger and re-prices the same trades at three cost levels: no costs (0x), your costs (1x) and double your costs (2x). For each level the table shows the number of trades, the mean net return per trade with a 95% confidence interval, the win rate, the profit factor and a plain verdict. A footer gives the break-even round trip: the cost per trade at which the mean net return is zero.
The signal is a built-in demo (an EMA 9/21 cross) or any plot from another indicator, chosen in the "External plot" input.

WHAT IS ORIGINAL HERE
Most backtests report one number under one cost assumption. This script stores the gross return and the round-trip cost of every trade separately, then subtracts that cost multiplied by 0, 1 and 2. All three rows come from the same trades, so only the cost changes from row to row. That shows how fast an edge erodes as costs grow, and whether the interval around the mean still clears zero. The break-even round trip is one threshold to compare with the costs you expect to pay.
The ledger keeps its own state and does not use the Strategy Tester, so it can score any signal written as a plot.

LEDGER RULES
- One position at a time. The signal is read on a closed bar and the entry is the open of the next bar.
- The exit is the first of two events: a time exit at the close of the last bar held (the entry bar counts as bar 1), or an opposite signal, which closes the position at the next open.
- An opposite signal that the direction setting does not allow still closes the position but opens nothing. An allowed one reverses it at the same open.
- A position still open on the last bar is not counted.
- Only confirmed bars update the ledger.

COST MODEL
Gross return = direction x (exit price / entry price - 1). Round trip at 1x = 2 x commission % + slippage ticks x tick size x (1 / entry price + 1 / exit price). Commission is a percentage of price per side; slippage is a number of ticks per side. Net return at level k = gross return - k x round trip, for k = 0, 1 and 2. The defaults (0.05% and 1 tick per side) are placeholders: use the costs of your own market and broker.

HOW TO USE IT
1. Add it to a chart with as much history as your plan loads.
2. Choose the signal source: the demo cross, or "External plot" with a plot from another indicator. Positive requests a long, negative a short, zero or na nothing.
3. Choose the direction and the time exit in bars.
4. Enter your commission and slippage per side, and the minimum number of trades for a verdict.
Every setting you try is another comparison, so count them.

HOW TO READ THE TABLE
Costs: the cost level. 0x charges nothing, 1x your costs, 2x twice that.
Round trip %: mean round-trip cost at that level, in percent of price.
Trades: closed trades in the ledger.
Mean net % [95% CI]: mean net return per trade in percent, with the 95% interval in square brackets.
Win rate: share of trades with a net return above zero.
Profit factor: gross profit divided by gross loss; "inf" when there is no loss.
Verdict: "positive" (green) when the whole interval is above zero, "negative" (red) when it is below, "not distinguishable from zero" when it contains zero, "too few" below the minimum number of trades.
Footer: the break-even round trip and the multiple of your costs it represents. "Negative before costs" means the mean return is not above zero even with no costs.

LIMITATIONS
The ledger is a simplified model: fills at chart-bar opens and closes, no gaps, partial fills, financing or position sizing. Every trade carries the same cost model, while real costs change with volatility, size and time of day.
The interval is a normal approximation. Few trades, fat tails and serial dependence between trades make it less reliable.
Many settings mean many comparisons; about 1 interval in 20 excludes zero by chance, and no correction is applied.
Only bars loaded on the chart count, so history depth changes the number of trades.
The demo is a placeholder, not a recommendation, and its first few bars can carry meaningless crosses.
An external signal that stays on after a time exit opens a new trade at the next open. An external plot must not repaint; the script cannot check that.
The script reports counts and statistics, not a trading rule, and it is not financial advice.

The chart shows [SYMBOL] on the [TF] timeframe with the default settings. The table is in the bottom right, and the triangles mark the entries of the demo signal.
---

## Publishing checklist

- [ ] Compile `cost_ladder.pine` in the Pine Editor with no errors or warnings.
- [ ] Add it to a liquid chart with long history. Check that the demo signal produces trades, that the 0x row has the highest mean, and that the verdict colors match the intervals.
- [ ] Set commission 0 and slippage 0: the three rows must be identical.
- [ ] Spot-check the marks: the number of entry marks should equal `Trades`, or `Trades` plus one if a position is still open on the last bar.
- [ ] Set the time exit to 1 bar and check one trade by hand: entry at the open of the marked bar, exit at its close.
- [ ] Set the chart: [SYMBOL], [TF], default inputs, table visible, some marks in view.
- [ ] Replace `[SYMBOL]` and `[TF]` in the last line with the real values of the chart.
- [ ] Check the title is at most 60 characters.
- [ ] Publish as open source, with the MIT license text if the form allows it.
- [ ] Check that the description contains no links, contact info, or performance claims.
- [ ] Choose the category and tags (for example: transaction costs, slippage, commission, trade statistics, confidence interval).
- [ ] Add the TradingView script URL to `README.md` after publishing.
