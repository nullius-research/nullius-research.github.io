# Three popular strategy ideas, tested against control groups. All three failed.

I tested three popular retail algo trading ideas. All three were tested against a control group, with real trading costs and the actual sequence of price movements included.

The conclusion was simple. **None of the three showed a real directional edge.**

The four Python scripts, the data assumptions and the exact tests are public here: [github.com/nullius-research/strategy-validation](https://github.com/nullius-research/strategy-validation)

---

## My main conclusion: test the assumption first, build the strategy afterwards

The longer I work on developing trading strategies, the more convinced I get that the strategy itself is not the most important part. The most important part is the method you use to decide whether a strategy has any right to exist at all.

What I want to avoid is seeing an idea that works well in a backtest, then spending months optimising indicators, parameters and exits, only to find out afterwards that there was never a structural edge in it. So I want to approach my strategy development differently.

Not starting with:

> "How can we make this strategy more profitable?"

But starting with:

> "Is there any information in this signal beyond chance?"

That difference looks small, but in my view it decides whether you are building a serious trading model or just optimising a nice-looking backtest.

## My starting points

I want to use the same basis for every test.

### 1. Costs are part of reality

I always include transaction costs. A strategy that is only profitable before spread, commission and other costs are taken into account is not interesting to me. In the end it is not about the theoretical movement of the market. It is about what actually remains.

### 2. I want to know what happens first

So I do not only look at MFE and MAE. Those tell me how far a trade went in the right and the wrong direction. But that says nothing about what happened along the way.

A trade can first go +2R and then hit -1.2R. If I only look at MFE and MAE, I can get a far too positive picture of that.

So the order matters.

If both the stop loss level and the target are hit within the same bar, I deliberately choose the stop loss. That is conservative, but I think that is a good thing. If a strategy only works when I use the most favourable interpretation of the data, then I do not want to build it anyway.

### 3. I always want a null measurement

For me this has become perhaps the most important step.

A strategy that makes money does not necessarily have an edge.

So what I want to know is: does my strategy do better than if I had picked the direction at random at exactly the same moments?

Same entry moment. Same stop. Same target. Same costs. Only the direction is random.

If my strategy does not clearly come out above that, then I have no reason to assume the signal has any predictive value.

For me that is the core of falsification.

---

## Test 1. Donchian breakout trend follower

The first strategy I tested is a classic trend follower on EURUSD H4. The reasoning behind it is logical. If price breaks out of a relatively long range, combined with a trend filter and enough trend strength, you would expect the move to continue.

The setup was:

- Donchian breakout of 40 bars
- EMA50 versus EMA200 as trend filter
- ADX(14) > 20
- initial stop at 2x ATR
- trailing stop at 3x ATR
- Donchian breakout of 20 bars as reversal exit
- risk of 0.5% per trade

The test ran from 2015 through 2025.

The outcome:

- 219 trades
- Net result: -EUR 616.80
- Profit factor: 0.89
- Maximum drawdown: 18.93%
- Win rate: 34.25%
- Average winner EUR 66.11, average loss EUR 38.72
- Payoff: 1.71

On its own that does not even look dramatic. But at a win rate of 34.25% I need roughly 1.92 to break even. So that is where the problem is. I am about 0.21R short. And with that the strategy is basically explained. But it gets more interesting when I look at the distribution.

The total result does not tell the whole story.

| Situation | Result |
|---|---|
| All 219 trades | -EUR 616.80 |
| Without the best trade | -EUR 1,071.53 |
| Without the best 3 trades | -EUR 1,720.04 |
| Without the best 5 trades | -EUR 2,143.96 |

Only five trades out of 219 were responsible for EUR 1,527 of the result. That is just 2.3% of all trades. If I leave those five out, the strategy sits more than 21% lower.

For me that is an important warning sign.

I do not want a strategy where the total result depends on a handful of exceptional trades. A trend follower is of course allowed to depend on a few large winners. That is part of trend following. But here the right tail is apparently not strong enough to compensate for the rest of the trades.

And then 2022 is interesting. 2022 was exactly the kind of year where I would expect a trend follower to perform well. EURUSD made a very clear and sustained move towards parity. Yet this strategy made only about 3.6% that year. That makes me extra critical of the trailing stop I chose. 97% of all exits came from the trailing stop.

The Donchian reversal exit was used only seven times in eleven years. Median hold was 21 bars and the longest trade lasted 15 days. So my interpretation is that the problem is not really on the loss side. The average loss was EUR 38.72 and the largest loss EUR 52.65. That is actually fine. So the losses are well controlled. The problem is on the other side.

The winners apparently do not get enough room to create the large right tail that a trend following strategy needs. For me that is a much more interesting conclusion than simply saying the strategy "does not work".

**Data limitation.** The broker only had four months of real tick data over a window of eleven years. The tester therefore fell back on generated ticks and reported 0% tick quality. Commission was recorded as 0 across all 219 trades. So this is an estimate, not a measurement.

I am publishing it anyway because you do not fix a payoff shortfall of 0.21R with better ticks.

---

## Test 2. Asian session breakout

The second idea was very different. Here I try to use the range that forms during the Asian session. The reasoning is simple. If the market builds a range between 00:00 and 07:00 UTC and then breaks out between 07:00 and 11:00 UTC, that first move might contain information about the direction of the rest of the session.

Ranges smaller than 12 pips are excluded. The stop sits on the other side of the range, so the range is 1R. Costs are 1.2 pips per round trip.

Here I deliberately put the control group at the centre. I took the same moments where the strategy would open a trade. But then I determined the direction at random with a coin flip. I repeated that 2,000 times. In total 772 sessions were tested.

And here I see exactly why a backtest without a control group can be dangerous.

At 2R the strategy came out at:

- +0.020R expectancy
- profit factor of 1.04

If I only looked at those numbers I might think: this may be a small edge, let's optimise it further. But the control shows that this result falls well inside the range of randomness. At 2R the control band ran from about -0.085R to +0.033R. So my +0.020R is nothing special. In fact, a random direction regularly gives the same result.

For me that is a far more important conclusion than the profit factor of 1.04.

What I take from this: a risk/reward ratio can very easily create the impression of an edge. With a stop of 1R and a target of 2R you only need to win about a third of your trades to end up near break even. If a small positive difference comes on top of that, a backtest can suddenly look interesting. But that does not mean the signal predicts anything. That is simply the arithmetic of the ratio you chose.

On top of that the transaction costs are relatively heavy. The median range was about 17 pips. At 1.2 pips of cost, roughly 7% of every trade goes to costs. And in 10.2% of sessions price broke both sides of the range.

So for me the conclusion is clear. **The breakout itself provides no convincing evidence of directional information in this test.**

---

## Test 3. Bollinger Band mean reversion

The third test I find perhaps even more interesting. Here I test the opposite idea. If price closes outside a relatively wide Bollinger Band, you might expect the move to come back to the mean eventually.

The setup uses:

- Bollinger 200
- standard deviation 2.5
- H1
- M15 data to determine the path to stop and target
- target at the middle band
- stop at 2x the target distance
- maximum one position per pair
- nine major FX pairs
- costs of 1.2 pips
- August 2022 to August 2026

But the interesting part is again in the control.

I used exactly the same moment, the same stop and the same target distance. Only the direction changed. One strategy fades the band break. The other follows the band break. If the band break itself contains no information, those two strategies together should produce roughly nothing but the transaction costs.

And that is almost exactly what I see.

| Strategy | Expectancy | Profit factor | Win rate |
|---|---|---|---|
| Fade the break | +0.0092R | 1.035 | 61.3% |
| Follow the break | -0.0243R | 0.916 | 59.2% |
| **Together** | **-0.0151R** | | |

The expected outcome based on transaction costs alone was about -0.0139R. The actual outcome was -0.0151R. A difference of only 0.0012R.

For me this is almost precisely what I would expect if there is no structural directional information in the band break.

I find that a very valuable outcome. Not because the strategy is successful, but because I have been able to rule out an assumption.

### The bootstrap

Nine FX majors share a common dollar factor. Three pairs breaking their band on the same day is one bet, not three. So I resample whole trading days, keeping all symbols together.

The effective sample is 460 days, not 790 trades. Resampling individual trades would make the interval artificially narrow.

The 95% interval on the fade expectancy runs from -0.040R to +0.058R. So it runs through zero.

### The biggest trap: a strategy can look very good temporarily

One of the things I find perhaps even more important from these tests is how easily you can convince yourself that a strategy works. Take 2025.

The Bollinger fade strategy then had:

- expectancy: +0.130R
- win rate: 68.9%
- profit factor: 1.72
- 219 trades

If I had only tested 2025 I would probably have been quite enthusiastic. I might then have built an optimisation process around it. Testing other parameters, other exits, other filters, adjusting position sizing, maybe even building an EA.

But as soon as I look at the surrounding years the picture changes completely.

- 2024: -0.118R
- 2026 so far: -0.063R

That makes it clear to me how easily you can discover a strategy that never structurally worked. You have simply found a period where the market happened to behave in a way that suited your rules. That is exactly the kind of situation I want to avoid in my own strategy development.

---

## What I take from these three tests

What strikes me most is that the three ideas are completely different in substance.

- Trend following
- Breakout
- Mean reversion

And yet they end up at roughly the same point.

As soon as I add transaction costs and a control group, the convincing evidence of an edge disappears. That makes the control group, for me, perhaps more important than the strategy itself.

- A nice equity curve is not enough.
- A high profit factor is not enough.
- A high win rate is not enough.
- Even a positive expectancy is not enough.

What I want to know is: why does this strategy make money?

And more importantly: does this strategy make more money than a random variant of the same strategy?

If I do not get a convincing answer to that, I do not want to invest further in it.

## Three lessons I want to carry forward

**1. Falsify first, optimise afterwards**

I no longer want to start by optimising a strategy. First I want to try to knock the idea down. If the idea survives even under critical conditions, it becomes interesting to develop further. That reverses the traditional approach. Not first looking for ways to make a strategy look better. First trying to prove it does not work.

**2. A larger number of trades does not automatically mean better information**

89 trades on one currency pair is few. So it makes sense to test multiple instruments. But I have to be careful there too. Nine FX pairs are not nine fully independent experiments. If they are all driven by the same dollar factor, I get less independent information than the number of trades suggests. So more data is not automatically more evidence.

**3. Above all I want to understand where the return comes from**

A total return tells me too little. I want to know how that return was built. Does it come from many small wins? From a few exceptional trades? From one particular year? From one specific market regime? From one instrument? Or does the result hold up when I remove the best trades, periods or instruments? In the end I find that far more important than the height of the equity curve.

## My conclusion

These tests did not give me three profitable strategies. But they gave me something I think is perhaps more important. I was able to rule out three ideas relatively quickly, before putting a lot of development time into them.

For me that is exactly what a good development method should do. I do not want to spend months building an EA only to find out afterwards that the original assumption had no basis. I want to know first whether there is anything to build at all.

So my starting point becomes:

**Test the hypothesis first. Then falsify. Then optimise. And only then build.**

A strategy that does not convincingly beat a control group does not move on to the next phase for me. A strategy that does has my attention. But even then I am not there yet. That is when the real work starts. Because after that I still have to show that the edge is robust across different periods, instruments and market regimes. And that the edge survives once I add realistic costs, slippage and out-of-sample tests. Only then do I consider it justified to put time into further development of a trading engine.

## Limitations

I used one broker and one data source.

Test 1 ran on generated ticks instead of real ticks.

Tests 2 and 3 cover four years. That is one to two regimes.

Costs are fixed at 1.2 pips for tests 2 and 3. They are not modelled from historical spread, because the bar data reported zero spread. I would run them again on independent tick data before calling them final.

## Disclaimer

None of this is financial advice, and none of it is a recommendation to trade or not trade any particular approach.

