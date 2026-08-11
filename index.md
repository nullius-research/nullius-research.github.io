# Three Popular Strategy Ideas, Honestly Tested. All Three Failed in One Day.

Most published backtests are written by people who wanted the strategy to work. This one is not. I tested three ideas that circulate constantly in retail algo trading, and I am publishing the results because negative results are the ones nobody shares.

Everything below is reproducible. Data source, cost assumptions, sample sizes and confidence intervals are all stated. If you think I got something wrong, the numbers are there to argue with.

## The method, and why it matters more than the strategies

Every test follows the same three rules.

**1. Costs are charged.** Every test includes a round-trip cost assumption. A backtest without costs is not a backtest.

**2. Order of events, not just extremes.** Maximum favourable and adverse excursion tell you how far price travelled in each direction, but not which came first. A trade with 2.0R of favourable movement and 1.2R of adverse movement is a winner or a loser depending entirely on sequence. When a single bar spans both stop and target, I score it as a stop. That bias is deliberate and it works against the strategy.

**3. Every result is compared to a control.** This is the part almost nobody does, and it is the part that decided all three outcomes.

A strategy making money is not evidence of an edge. The relevant question is whether it makes more money than a null version of itself taken at the same moment, with the same stop, the same target and the same costs. If it does not, the signal contains no information and any profit in the sample is a fortunate draw.

---

## Test 1: Donchian breakout trend following

**Setup.** EURUSD H4, 2015 to 2025. Entry on a 40-bar Donchian breakout, filtered by EMA50 versus EMA200 and ADX(14) above 20. Initial stop 2x ATR(14), trailing stop 3x ATR(14), reversal exit on a 20-bar Donchian break. Risk 0.5% of equity per trade on a 10,000 EUR account.

**Result: 219 trades, net -616.80 EUR, profit factor 0.89, maximum equity drawdown 18.93%.**

The win rate was 34.25%. Average win 66.11 against average loss 38.72, so a payoff ratio of 1.71. At a 34.25% win rate the break-even payoff is 1.92. The strategy fell 0.21R short. That single line is the entire explanation of the loss.

The tail dependence was worse than the headline suggests:

| | Net result |
|---|---|
| All 219 trades | -616.80 |
| Excluding the single best trade | -1,071.53 |
| Excluding the best three | -1,720.04 |
| Excluding the best five | -2,143.96 |

Five trades out of 219, or 2.3% of the sample, carried 1,527 EUR. Remove them and the account is down more than 21%. That is not a weak edge, it is a lottery ticket.

Year by year, the strategy lost money in seven of eleven years. The most revealing figure is 2022: the strongest sustained EURUSD trend of the decade, the entire move toward parity, and a trend-following system extracted 3.6% from it.

**Diagnosis.** The trailing stop fired 97% of all exits. The Donchian reversal exit fired seven times in eleven years. Median hold was 21 bars and the longest trade lasted 15 days. Trend following depends on a fat right tail, and a 3x ATR trail was cutting it off. Meanwhile the loss distribution was excellent: largest loss 52.65 against an average of 38.72. The risk control worked perfectly. The problem was entirely on the profit side.

**Data caveat, stated plainly.** The broker supplied real tick data for only four months of the eleven-year window, so the tester silently fell back to generated ticks and reported 0% real tick quality. Commission recorded as zero across all 219 trades. These numbers are an estimate, not a measurement. They are reported anyway because the failure mode is structural and better tick data would not repair a payoff ratio deficit.

---

## Test 2: Asian session range breakout

**Setup.** EURUSD, August 2022 to August 2026, signals on M15. Build the range from 00:00 to 07:00 UTC. Trade the first break of that range between 07:00 and 11:00 UTC. Stop at the opposite side of the range, so R equals the range width. Sessions with a range under 12 pips excluded as noise. Cost 1.2 pips round trip.

**Control: the same entries with direction assigned by a coin flip, resampled 2,000 times.**

**Result: 772 sessions. Every target level fell inside the control band.**

| Target | Real expectancy | Control 5th to 95th percentile | Percentile |
|---|---|---|---|
| 0.5R | -0.0155R | -0.0328 to +0.0314 | 23.2 |
| 1.0R | -0.0239R | -0.1100 to -0.0036 | 84.3 |
| 1.5R | +0.0031R | -0.0964 to +0.0215 | 87.2 |
| 2.0R | +0.0199R | -0.0848 to +0.0328 | 88.9 |

At the 2.0R target the strategy shows expectancy of +0.020R and a profit factor of 1.04. Read alone, that looks like a small edge and it is exactly the kind of number that gets an EA built on it. The control band at that target runs from -0.085 to +0.033. A random direction regularly did better.

At the 0.5R target the real breakout direction placed in the 23rd percentile, meaning it underperformed roughly three quarters of coin flips.

**Diagnosis.** With a 1R stop and a 2R target you win about a third of the time and lose two thirds, which lands near break-even by construction. Almost any stop-target geometry lands there. That is arithmetic, not edge.

Two further problems. The compression filter, which was the original premise, steers you toward sessions with a median R of 17 pips, where a 1.2 pip cost consumes 7% of every trade. And 10.2% of sessions broke both directions.

---

## Test 3: Bollinger band mean reversion, nine pairs

**Setup.** Bollinger(200, 2.5) on H1, signals resampled from M15 data so the path to stop or target is walked on M15 bars. Fade the first close outside the band. Target is the middle band. Stop is 2x the target distance, deliberately wide. One position at a time per pair. Nine major pairs, August 2022 to August 2026. Cost 1.2 pips round trip.

**Control: the momentum version. Same moment, same stop, same target distance, trading with the break instead of against it.**

**Result: 790 trades across 460 distinct trading days.**

| | Expectancy | Profit factor | Win rate |
|---|---|---|---|
| Fade the break | +0.0092R | 1.035 | 61.3% |
| Follow the break | -0.0243R | 0.916 | 59.2% |
| **Sum** | **-0.0151R** | | |

That last row is the finding.

If the band break carries no directional information, then fading and following must sum to the transaction cost and nothing else. Averaging the per-trade cost across all 790 trades gives a predicted sum of -0.0139R.

Measured: -0.0151R.

The gap is 0.0012R, and it points the right way. When a single M15 bar spans both the stop and the target, the test scores it as a stop. That rule applies to the fade and the follow version alike, so it pushes both slightly negative. A small negative residual is exactly what the method predicts. For scale, the gap is roughly forty times smaller than the width of the confidence interval below.

The two opposite strategies returned the broker's cut and nothing beyond it. This is as clean a null result as this kind of test produces.

The supporting numbers agree. The 95% cluster-bootstrap interval on the fade expectancy runs from -0.040 to +0.058. Five of nine pairs showed positive expectancy, where a coin flip predicts four or five. Splitting by the slope of the middle band produced +0.028, -0.023 and +0.022 across flat, medium and steep regimes, with no monotonic pattern.

**On the bootstrap.** FX majors share a common dollar factor. Three pairs breaking their bands on the same day is one bet, not three. The interval above is therefore produced by resampling entire trading days with all symbols kept together, which is why the sample is described as 460 days rather than 790 trades. Resampling individual trades would have produced a narrower and dishonest interval.

**And a warning about sub-samples.** Year by year, 2025 returned +0.130R expectancy across 219 trades at a 68.9% win rate and a profit factor of 1.72. Tested on 2025 alone, this looks like a discovery. 2024 returned -0.118R and 2026 to date returns -0.063R.

This is the mechanism behind the familiar complaint that a strategy "worked at first and then stopped working". It never worked. The trader found the 2025-shaped section of a series that oscillates around zero.

---

## What the three tests have in common

Three different premises. Trend continuation, session volatility expansion, and mean reversion. Three different timeframes. One instrument, then one instrument, then nine.

All three produced the same answer: the price-derived signal contained no directional information once costs and a control were applied.

This is not surprising when stated plainly. Liquid FX majors are the most heavily analysed price series in existence. Expecting a moving average, a channel or a standard deviation band to extract directional information from them, when every participant computes the same functions on the same numbers, is optimistic.

Three observations that generalise beyond these particular ideas:

**A profitable-looking backtest without a control is not evidence.** Test 2 showed a profit factor of 1.04 and was beaten by coin flips. Without the control, that becomes an EA.

**Sample size on a single instrument is almost always insufficient.** Test 3 on one pair gave 89 trades and a confidence interval half an R wide. An edge of +0.05R is invisible at that resolution. Pooling nine pairs was necessary just to make the measurement meaningful, and honest clustering then gave much of that precision back.

**The distribution matters more than the total.** Test 1 was carried entirely by five trades. Any strategy where 2.3% of trades produce the entire result should be treated as unmeasured, whatever the equity curve looks like.

## Limitations

Single broker, single data source. Test 1 ran on generated rather than real ticks. Tests 2 and 3 cover four years, which is one to two regimes, not a full cycle. Cost assumptions are fixed at 1.2 pips rather than modelled from historical spread, because the exported bar data reported zero spread. Longer histories from an independent tick source would strengthen all three, and I would run them again on better data before treating any of these conclusions as final.

None of this constitutes financial advice, and none of it is a recommendation to trade or not trade any particular approach.

## Why publish a failure

Because the useful output was never the strategies. It was that three ideas were disproved in a day rather than after three months of EA development and a live account.

Retail algo trading has no shortage of profitable-looking backtests. It has a severe shortage of falsification. If a control group and a confidence interval had been standard practice, most of the strategies currently for sale would never have reached a screenshot.

If you have an idea you are about to commit real development time to, test the assumption before you build the system. It takes an afternoon and it is the cheapest thing you will ever do.
