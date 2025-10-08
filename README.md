Objective Function: Intuition and Economic Motivation

The optimizer chooses the number of shares in each stock so that the portfolio’s dollar value tracks the basket’s dollar value as closely as possible over time.
In plain terms, it builds a buy-and-hold hedge that moves dollar-for-dollar with the basket — not just in returns, but in actual money.

1. What the Objective Minimizes

The objective is a weighted mix of two goals:

Objective
=
(
1
−
𝜆
)
 
Var
[
daily P&L difference
]
+
𝜆
 
Var
[
value difference
]
Objective=(1−λ)Var[daily P&L difference]+λVar[value difference]

Daily P&L difference:
How much your hedge’s day-to-day profit and loss deviates from the basket’s.

Value difference:
How far apart your hedge’s total dollar value is from the basket’s level across time.

The parameter λ controls the trade-off:

Low λ → focus on daily P&L stability.

High λ → focus on long-term value tracking.

2. Why Daily P&L Variance Matters

Suppose you’ve sold a $10 million basket option and are using this optimizer to build a hedge.
Each day, the basket moves up or down, changing the option’s mark-to-market.

If the basket loses $400,000 but your hedge gains only $250,000, you still lose $150,000.
That difference is your tracking error in dollars.
Minimizing the variance of these daily differences creates a hedge that closely mirrors the basket’s daily dollar swings — your hedge moves when the basket moves, and by a similar amount.

This reduces mark-to-market volatility and stabilizes your hedge’s P&L, which is crucial for margin, capital, and risk reporting.

3. Why Value Variance Matters

Even if your hedge offsets daily P&L fairly well, it can drift in value over time.
Here’s how that happens:

Imagine a basket and a hedge that start at $10 million each.

Day	Basket Value	Hedge Value	Daily P&L Difference
1	$10.00M	$10.00M	—
2	$10.50M	$10.30M	+$200K
3	$10.25M	$10.05M	+$200K
4	$10.80M	$10.40M	+$400K

The hedge tracks the direction of every move — when the basket rises, the hedge rises too — but it’s consistently smaller in magnitude.
So by the end of the period, the basket is up $800K, while the hedge is up only $400K.
Daily P&L tracking looks fine, but cumulative value has diverged badly.

This happens because the hedge’s total dollar exposure isn’t sized or balanced perfectly relative to the basket’s — small mismatches compound over time.
Minimizing the variance of value differences forces the optimizer to pick share quantities that keep the overall level of the hedge aligned with the basket’s through time.

4. Why Combine the Two

The two objectives target different, complementary risks:

Focus	Purpose	If Ignored
Daily P&L variance	Controls day-to-day volatility of hedge P&L	Hedge looks unstable; large mark-to-market swings
Value variance	Keeps cumulative dollar value aligned with basket	Hedge drifts away; poor final hedge P&L

A good hedge must do both — stay stable and stay on track.
λ gives you control over that balance:

Smaller λ → smooths daily P&L but tolerates some long-term drift.

Larger λ → tightens long-term tracking at the cost of more short-term noise.

5. Why Everything Is in Dollar Terms

This optimizer works in dollars rather than returns because options are dollar exposures, not percentage ones.
A 1% error in a $10M basket means $100K of risk.
A 1% error in a $1M hedge is only $10K — even if correlations look perfect, the hedge covers just one-tenth of the loss.
Optimizing in dollar space directly matches the hedge’s economic impact to the size of the actual liability.

In Summary

The objective function minimizes how far your hedge portfolio’s dollar path deviates from the basket’s dollar path, balancing:

Smooth daily P&L behavior (short-term stability), and

Accurate long-term value alignment (terminal payoff accuracy).

This dual-variance framework produces a hedge that behaves like a scaled, dollar-for-dollar shadow of the basket — one that’s economically realistic, robust to drift, and effective for hedging basket option exposures.
