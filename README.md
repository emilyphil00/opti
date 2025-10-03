Static Portfolio Replication: A Dollar-Variance Optimization Framework
Problem Statement
The objective is to construct a static long-short equity portfolio that replicates a target basket over a one-year holding period without rebalancing. Traditional mean-variance optimization assumes continuous rebalancing to maintain target weights—an assumption that breaks down entirely in a buy-and-hold framework. When 1,000 shares are purchased at $50 and the stock doubles, the position automatically represents twice the dollar exposure. Portfolio weights drift throughout the year as prices evolve, and standard percentage-based covariance matrices fail to capture this dynamic.
This limitation becomes particularly acute when hedging basket options. A hedge exhibiting 0.95 correlation can still leave substantial residual exposure if notional scale is misaligned. A $1 million hedge cannot offset $10 million of underlying risk, regardless of how tight the statistical relationship appears.
Methodology: Dollar-Space Optimization
Rather than modeling percentage return covariances, the framework constructs a covariance matrix based on dollar return factors. Daily dollar returns are computed as the product of shares held, lagged prices, and percentage returns. This approach captures the actual mechanics of buy-and-hold portfolios: positions in higher-priced stocks contribute more dollars to daily P&L volatility than positions in lower-priced stocks with identical percentage returns.
Objective Function
The optimizer minimizes the squared difference between the hedge portfolio's daily dollar P&L and the basket's dollar P&L across the time series:
min⁡shares∑t(∑isharesi⋅pi,t−1⋅ri,t−basket_shares⋅Bt−1⋅rtbasket)2\min_{\text{shares}} \sum_t \left( \sum_i \text{shares}_i \cdot p_{i,t-1} \cdot r_{i,t} - \text{basket\_shares} \cdot B_{t-1} \cdot r^\text{basket}_t \right)^2sharesmin​t∑​(i∑​sharesi​⋅pi,t−1​⋅ri,t​−basket_shares⋅Bt−1​⋅rtbasket​)2
The covariance matrix for dollar returns is constructed as:
Qdollar=1T(R⊙Pt−1)T(R⊙Pt−1)Q_{\text{dollar}} = \frac{1}{T}(R \odot P_{t-1})^T (R \odot P_{t-1})Qdollar​=T1​(R⊙Pt−1​)T(R⊙Pt−1​)
where RR
R is the return matrix, Pt−1P_{t-1}
Pt−1​ is the lagged price matrix, and ⊙\odot
⊙ denotes element-wise multiplication. The linear term captures the covariance between constituent dollar returns and basket dollar returns.

The optimization solves for constituent share quantities that minimize dollar-denominated tracking error. Low variance indicates consistent proximity to the target; high variance indicates substantial deviations that alternate between positive and negative.
Constraint Structure

Day-1 weight equivalence: Portfolio weights on day one must match basket composition, scaled by notional—ensuring proper initial alignment
Position bounds: Individual positions constrained between 20% and 80% of day-one weight (configurable via weight_longs and weight_shorts)
Position sizing: Binary selection variables enforce meaningful positions within bounds or complete exclusion—eliminating trivially small holdings
Maximum position count: Limits on the number of long and short positions prevent excessive concentration or over-diversification
Short multiplier: Shorts are scaled by a multiplier (default -0) to control the degree of short exposure relative to longs
Implied volatility bounds: Constituent implied volatilities constrained between min_wtiv (0.1) and max_wtiv (0.9) to ensure reasonable vol characteristics

These constraints eliminate unrealistic or untradeable portfolio configurations that may appear optimal in unconstrained solutions.
Rationale for Day-1 Weight Matching
Day-1 weight equivalence represents a fundamental requirement rather than an optional parameter:

Scale alignment: The hedge portfolio begins with weights proportional to basket composition, scaled to match basket notional ($1M in this implementation)
Greek consistency: Delta, vega, and gamma all scale with both notional exposure and position weights. Proper day-one alignment ensures risk sensitivities begin correctly calibrated
Operational acceptance: Risk management frameworks require hedges that scale proportionally with underlying liabilities from inception
Path realism: The optimization accounts for weight drift over time by minimizing variance across all future days, but proper initialization is critical for tracking effectiveness

The constraint x=shares⋅p0basket_notionalx = \frac{\text{shares} \cdot p_0}{\text{basket\_notional}}
x=basket_notionalshares⋅p0​​ ensures day-one weights align with basket composition, where xx
x can be decomposed into positive and negative components for long-short implementation.

Implementation Approach
The implementation uses CVXPY with the SCIP solver for mixed-integer quadratic programming. Binary selection variables render this computationally intensive, but the branch-and-bound algorithm efficiently searches the combinatorial space. The optimality gap tolerance is set to 5% (limits/gap: 0.05), with parallel solving across 4 cores.
Key implementation details:

Dollar return factors: Computed as return_matrix * lagged_prices, forming the basis for the dollar covariance matrix
PSD verification: The dollar covariance matrix is verified to be positive semi-definite before optimization
Weight decomposition: Day-one weights are split into positive (pos_x) and negative (neg_x) components to enable long-short construction
Selection logic: Binary variables (pos_selection, neg_selection) enforce the discrete choice of position inclusion
Filtering: Post-optimization, only positions with absolute weights exceeding 0.0005 are retained in the final output

Expected Performance Characteristics and Limitations
Tracking quality exhibits systematic time-dependent degradation. On day one, portfolio weights align precisely with optimizer specifications (scaled to basket composition). As prices evolve, weights drift automatically—rallying positions become larger, declining positions shrink. By mid-year, accumulated drift has shifted the portfolio away from optimal composition. By year-end, the static position established at inception may be substantially misaligned with what would be optimal for current tracking.
The optimizer observes the complete annual dataset during calibration but must select a single static position representing the optimal compromise across all 252 trading days. The solution minimizes aggregate variance of dollar tracking error rather than pointwise accuracy at any specific date. Tracking errors compound over time: early-period deviations combine with subsequent weight drift, creating cumulative differences beyond what simple daily volatility would suggest.
Backward normalization analysis (scaling both time series such that terminal values equal 1.0) illustrates relative growth trajectories. The spread between normalized curves identifies periods where the basket outperformed the static portfolio's ability to track. The optimization minimizes tracking error variance rather than absolute deviation, therefore consistent directional bias is acceptable if it reduces the volatility of period-to-period differences.
Summary and Evaluation Framework
Dollar-variance optimization with day-one weight matching and position constraints produces executable, risk-appropriate portfolios suitable for hedging basket option exposures. The framework explicitly acknowledges the inherent tension between static position constraints and dynamic target behavior, identifying optimal compromises that degrade in predictable and manageable fashion over the holding period.
Reasonable tracking accuracy is achievable using concentrated long-short structures with controlled position counts and implied volatility bounds. The absence of rebalancing creates unavoidable performance deterioration, but the optimizer accounts for this by finding share quantities that minimize tracking error variance across the entire time horizon. Perfect static replication is infeasible absent exceptionally stable constituent relationships, which do not exist in practical markets.
Performance evaluation hierarchy: Dollar tracking error (RMSE) serves as the primary metric, directly measuring effectiveness of monetary risk offset. R² and beta provide supplementary diagnostics but remain secondary considerations. The relevant criterion is actual P&L hedge effectiveness, not statistical elegance that fails to translate into meaningful risk reduction.
