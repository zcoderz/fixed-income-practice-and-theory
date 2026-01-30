# Chapter 47: Hedging and Relative Value in CDS Indices

---

## Introduction

You hold $20 million notional of Ford CDS protection and want to hedge with CDX.IG. How much index notional do you need? The answer seems straightforward: match CS01s, calculate a hedge ratio, and execute. But the experienced credit trader knows this is where the real work begins.

A CS01-matched hedge can still produce substantial P&L when Ford widens by 50 basis points while the index moves only 10. Your "hedged" position just generated nearly $200,000 of profit—or loss—depending on direction. The hedge neutralized systematic credit risk, but the idiosyncratic component remained fully exposed. As O'Kane emphasizes in his treatment of portfolio CDS indices, "the portfolio CDS index allows investors to take a broad exposure to the credit markets in a single transaction," but this very feature means the hedge you construct may not behave as expected when individual constituents move independently of the index.

This distinction between *systemic* and *idiosyncratic* risk lies at the heart of index hedging. O'Kane makes this explicit in his treatment of tranche hedging: "the idiosyncratic delta is calculated by widening one credit while keeping all of the other credits fixed... This is quite different from how the systemic delta is calculated. This involves widening all of the issuer curves simultaneously." The same framework applies directly to index hedging, where choosing between these approaches—or finding the right blend—determines hedge effectiveness. As O'Kane notes, "In practice, spread movements exhibit a mixture of systemic and idiosyncratic moves and the correct hedge will be somewhere between the idiosyncratic hedge and the systemic hedge."

This chapter provides a rigorous, risk-first framework for CDS index hedging and relative value analysis. We cover the complete workflow: identifying what exposures you actually have, selecting appropriate hedge instruments, computing hedge ratios with explicit assumptions, and validating those hedges with scenario analysis. Throughout, we maintain the discipline established in Chapter 43 (CDS Risks) and Chapter 44 (Single-Name RV): define the risk precisely, then design the hedge.

**Chapter roadmap:**

- **Section 47.1** establishes core concepts: index mechanics, intrinsic value, basis, and the risk measures (CS01, VOD) that drive hedging decisions
- **Section 47.2** develops the hedge problem framework: exposure decomposition, instrument selection, ratio computation, beta estimation methodology, and hedge effectiveness metrics
- **Section 47.3** details the mathematics of CS01-based and multi-factor hedging, including the systemic vs idiosyncratic delta distinction
- **Section 47.4** analyzes top-down vs bottom-up hedging with explicit decision criteria, IG vs HY differences, proxy hedging across index families, and cross-asset applications
- **Section 47.5** examines default events and how index risk evolves through the lifecycle
- **Section 47.6** presents relative value frameworks within a risk-first lens
- **Section 47.7** provides comprehensive worked examples
- **Section 47.8** offers practical notes, pitfalls, and a hedge validation template

This chapter builds directly on Chapter 46 (Intrinsic Index Spread and Index Basis). Readers unfamiliar with index basis concepts should review that chapter first.

---

## Conventions & Notation

This chapter is **risk-first**: we describe exposures, hedges, residuals, and validation—not trade recommendations.

No index rulebook specifics (standard coupons, exact roll calendars, operational settlement mechanics) are asserted unless supported by the provided sources. Where a statement depends on your exact confirmation / rulebook / valuation policy, we flag it as **NOT SURE** and state the missing input.

All spreads are in bp unless stated; conversion: $1 \text{ bp} = 10^{-4}$ (decimal rate per year).

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $t$ | Valuation time ("today") |
| $T$ | Maturity date of the CDS / index swap |
| $M$ | Number of index constituents (alive at inception; defaults remove names without replacement) |
| $m \in \{1, \ldots, M\}$ | Constituent name index |
| $N$ | Trade notional (USD); $N_I$ for index notional, $N_m$ for constituent notional |
| $S_m(t,T)$ | Market spread for name $m$ to $T$ |
| $S_I(t,T)$ | Quoted index spread (market "index curve" level) to $T$ |
| $C(T)$ | Contractual index coupon (fixed for the series; typically chosen to be a multiple of 5 bp) |
| $U_I(t)$ | Upfront as percent of notional (positive = paid by protection buyer; negative = received) |
| $V_I(t)$ | Intrinsic value of the index (constituent-implied) |
| $\text{RPV01}(t,T)$ | Risky PV01 (PV of premium leg per unit spread; includes accrued-at-default) |
| CS01 / Credit DV01 | Spread sensitivity; O'Kane's definition uses sign convention so short-protection CS01 is positive |
| VOD | Value-on-default (jump-to-default style risk); see Chapter 43 for the single-name treatment |
| $FP$ | Auction final price / recovery proxy in cash settlement (per 100) |
| $\beta_{m,I}$ | Spread beta: sensitivity of name $m$ spread changes to index spread changes |
| $\rho$ | Correlation coefficient between spread changes |
| $\sigma_m$, $\sigma_I$ | Standard deviation of spread changes for name $m$ and index $I$ |
| HE | Hedge effectiveness ratio |

---

## 47.1 Core Concepts

### 47.1.1 The CDS Index as "One Trade, Many Names"

A CDS portfolio index is a contract on a fixed set of reference entities for the life of the series. O'Kane describes the key structural features: "Since the indices contain as many as 125 credits, the portfolio CDS index allows investors to take a broad exposure to the credit markets in a single transaction." The index pays a fixed contractual coupon $C(T)$ on the outstanding notional and is traded with an upfront $U_I(t)$ that can be positive or negative.

When a constituent defaults, it is removed without replacement—the remaining portfolio continues with reduced notional. As O'Kane notes, "the size of the coupon is reduced as the face value is reduced by $1/M$" after each credit event. This creates a dynamic composition that evolves through the index lifecycle.

**Why indices matter for hedging:**

1. **Liquidity**: Indices often trade tighter bid-ask than underlying single names
2. **Efficiency**: One transaction creates broad credit exposure
3. **Macro hedging**: Ideal for offsetting systematic credit risk in a portfolio
4. **Benchmark**: Indices serve as reference points for relative value analysis

### 47.1.2 Intrinsic Value vs Quoted Index Level

The distinction between *intrinsic* and *quoted* index values is fundamental to index hedging. Chapter 46 develops this in detail; here we summarize the key points.

**Intrinsic value** $V_I(t)$ is the value implied by constituent CDS curves—building the index "from the bottom up" as a weighted sum of single-name values. O'Kane writes the intrinsic value of an index (short protection) as:

$$\boxed{V_I(t) = \frac{1}{M} \sum_{m=1}^{M} \bigl(C(T) - S_m(t,T)\bigr) \cdot \text{RPV01}_m(t,T)}$$

**Quoted index level** uses a simplified flat index curve $S_I(t,T)$ to price the index "as though it were a single name." This market convention sacrifices precision for liquidity and quoting efficiency.

**Index basis** (in spread terms) is:

$$\text{Basis} = S_I - S_I^{\text{intrinsic}}$$

O'Kane documents that quoted and intrinsic spreads often differ and lists several drivers:

- **Restructuring clause differences**: Index often trades "No Restructuring" while underlying CDS may trade "Modified Restructuring" or other variants
- **Liquidity premium**: "The considerable size and liquidity of the CDS index market means that the CDS index spread embeds a lower liquidity risk premium than the less liquid CDS spreads"
- **Lead-lag effects**: "The CDS index may be considered to lead the CDS market. This is especially true in a widening market where investors use long protection positions in the index to hedge illiquid long credit positions"

For hedging purposes, basis represents a residual risk—you cannot simultaneously match index spread exposure *and* constituent spread exposures unless you account for basis explicitly.

### 47.1.3 Portfolio Swap Adjustment (PSA)

When hedging index products (particularly options or tranches) with constituents, practitioners often use a **portfolio swap adjustment** to reconcile intrinsic and quoted values. O'Kane describes PSA as "a way to adjust the individual CDS issuer curves so that the intrinsic of the adjusted CDS curves exactly matches the CDS index quotes."

A common implementation applies a spread multiplier $\alpha(T)$:

$$S_m^*(t,T) = \alpha(T) \cdot S_m(t,T)$$

and solves for $\alpha(T)$ so that the adjusted intrinsic equals the market index upfront. The adjustment is described as "somewhat arbitrary, chosen for stability/speed/reasonableness."

**When PSA matters:**
- Hedging index options with constituents
- Tranche pricing and hedging
- Basis trading where you need consistent marks

### 47.1.4 Spread Risk Measures: CS01 and RPV01

**CS01 (Credit DV01)** measures the dollar change in value for a 1 bp parallel increase in the CDS curve. O'Kane defines it with a sign convention so short protection DV01 is positive:

$$\boxed{\text{CS01} = -\bigl(V(S + 1\text{ bp}) - V(S)\bigr)}$$

This is the primary risk measure for non-defaulting spread moves. Chapter 43 provides the complete treatment; here we note that:

- **What is bumped matters**: A "1 bp bump" to the quoted index curve differs from bumping each constituent curve by 1 bp
- **Units are \$/bp**: For a position with notional $N$ and RPV01 of 4.0 years, approximate CS01 is $N \times 4.0 \times 10^{-4} = N \times 0.0004$

**RPV01** (risky PV01) is the premium-leg PV factor, representing the expected present value of receiving 1 unit of spread until default or maturity. For small spread changes:

$$\Delta V \approx -\text{CS01} \cdot \Delta S$$

where the sign depends on buyer/seller convention.

### 47.1.5 Default-Event Risk: VOD (Value on Default)

Even with perfect spread hedging, a sudden default creates discrete P&L. Chapter 43 defines Value on Default (VOD) for a single-name CDS using O'Kane's formula:

$$\text{VOD}_{\\text{buy protection}} = -V(t) + (1-R) - \\Delta_0 S_0$$

where $V(t)$ is the CDS mark-to-market immediately before default and $\\Delta_0 S_0$ is the accrued premium at default. For a par position with $V(t) \\approx 0$, VOD per unit notional is approximately $(1-R) - \\Delta_0 S_0$.

For indices, VOD concepts extend with portfolio scaling. A constituent default:

1. Triggers a protection payment proportional to $1/M$ of notional
2. Cancels that name's share of future premium payments
3. Reduces the outstanding index notional

O'Kane's index valuation treats each name as contributing $(1 - R_m)/M$ losses to the index buyer on default.

### 47.1.6 Roll / Series Risk

O'Kane describes indices as "rolled semi-annually" with important risk implications:

- **Maturity extension**: "The new index has a longer maturity, by six months, than the previous index. As credit curves tend to be upward sloping, this would tend to cause the new index spread to be wider"
- **Membership changes**: The new roll may add or remove names based on credit quality and liquidity criteria
- **Liquidity migration**: After a roll, liquidity concentrates in the new on-the-run series; hedging costs for off-the-run positions increase

---

## 47.2 The Hedge Problem Framework

This section provides the organizing framework for index hedging: what exposure do you have, what do you hedge with, how do you size, and how do you validate?

### 47.2.1 Decomposing Your Actual Exposure

A rigorous hedge starts by identifying *all* risk channels in your position. For index-related books, decompose into:

**Systematic (macro) credit spread risk**
The index level $S_I(t,T)$ behaves like a macro factor capturing market-wide credit conditions. This is what indices are designed to express or hedge efficiently.

**Idiosyncratic (name-specific) spread risk**
Even in an equal-weight index, constituents move differently. If you hold single-name positions or have concentrated index exposures, you face dispersion risk—the risk that individual name moves deviate from the index average.

**Index–constituent basis**
The gap between quoted and intrinsic spreads creates a distinct risk. If you hedge an index with constituents (or vice versa), basis moves generate P&L even when your aggregate CS01 is zero.

**Default-event / jump risk**
Constituents can default suddenly. The cashflow impact includes:
- Protection payment (face minus recovery)
- Accrued premium settlement
- Notional reduction affecting future premium flows

**Roll / series / liquidity risk**
Semiannual rolls create mismatches when hedging across series. Off-the-run positions face wider bid-ask and reduced liquidity.

### 47.2.2 Selecting Hedge Instruments

A practitioner's menu of hedge instruments, with their strengths and limitations:

| Instrument | Best For | Residual Risks |
|------------|----------|----------------|
| Index (same series, same maturity) | Systematic spread risk | Idiosyncratic, basis |
| Index (different series) | Systematic spread when OTR unavailable | Series basis, membership mismatch |
| Single-name CDS (constituents) | Idiosyncratic concentration | Execution cost, many line items |
| Single-name CDS (proxies) | Illiquid names | Name mismatch, beta estimation |
| Different maturity index | Curve exposures | Term structure twist |

**Decision criteria for instrument selection:**

1. **What risk dominates?** If macro spread risk dominates, use index hedges. If name-specific risk dominates, consider single-name hedges.

2. **Liquidity and cost**: O'Kane notes that "the considerable size and liquidity of the CDS index market means that the CDS index spread embeds a lower liquidity risk premium." Index hedges typically cost less to execute than equivalent single-name positions.

3. **Precision vs efficiency**: Single-name hedges are more precise but require more trades and monitoring. Index hedges are efficient but leave idiosyncratic exposure.

### 47.2.3 Computing Hedge Ratios

The fundamental hedge sizing approaches:

**CS01 / Credit DV01 matching**
The primary "first-pass" method. Choose hedge notional so total CS01 equals zero:

$$N_{\text{hedge}} = -\frac{\text{CS01}_{\text{position}}}{\text{CS01}_{\text{hedge}}(1)}$$

where $\text{CS01}_{\text{hedge}}(1)$ is the CS01 per unit notional of the hedge instrument.

**Multi-factor hedging**
When curve shape matters (e.g., hedging a 10Y exposure with 5Y and 10Y instruments), solve a linear system to neutralize multiple buckets simultaneously. See Section 47.3.2.

**Beta-adjusted proxy hedges**
When hedging a single name with an index, the name's spread may move more or less than the index systematically. A spread beta $\beta_{m,I}$ adjusts the hedge ratio:

> **Analogy: The Speedboat and the Ocean Liner**
>
> *   **The Index (Ocean Liner)**: Only moves when the whole tide (market) changes. Steady.
> *   **The High-Vol Name (Speedboat)**: Bounces aggressively on every wave.
> *   **The Hedge**: If the Speedboat moves 2 inches for every 1 inch the Liner moves ($\beta=2$), you need 2x the Index notional to offset the motion.
> *   **The Risk**: Sometimes the Speedboat engine fails (idiosyncratic shock) while the Liner sits still. Your hedge does nothing.

$$N_I = -\frac{\text{CS01}_m}{\beta_{m,I} \cdot \text{CS01}_I(1)}$$

> **Deep Dive: The Regression Hedge**
>
> How do you find $\beta$? Traders run a simple OLS regression on spread changes:
>
> $$\Delta S_{\text{name}} = \alpha + \beta \Delta S_{\text{index}} + \epsilon$$
>
> *   **$\beta$ (Systematic)**: What you hedge.
> *   **$\epsilon$ (Idiosyncratic)**: What you keep.
> *   **$R^2$**: How much of the name's spread variance is explained by the index move. Low $R^2$ means large residual tracking error—an index hedge may still reduce macro exposure, but it will not make the position "quiet."

### 47.2.4 Estimating Spread Beta in Practice

While the previous section introduced beta-adjusted hedging conceptually, this section provides the methodology for *estimating* spread beta from market data. This is essential knowledge for practitioners designing proxy hedges.

**The Minimum Variance Hedge Framework**

Hull provides the foundational framework in Chapter 3. For cross-hedging (hedging one asset with a related but different instrument), the minimum variance hedge ratio is:

$$\boxed{h^* = \rho \frac{\sigma_S}{\sigma_F}}$$

where $\rho$ is the correlation between changes in the position value ($\Delta S$) and changes in the hedge value ($\Delta F$), $\sigma_S$ is the standard deviation of $\Delta S$, and $\sigma_F$ is the standard deviation of $\Delta F$.

Hull shows this is equivalent to the regression slope: "It follows from the formula for the slope in linear regression that $h^* = b$" where $b$ is the slope from regressing $\Delta S$ on $\Delta F$.

**Application to Credit Spreads**

For CDS hedging, translate Hull's framework:
- $\Delta S$ = daily change in single-name spread (the position)
- $\Delta F$ = daily change in index spread (the hedge)
- $h^*$ = spread beta $\beta_{m,I}$

The spread beta formula becomes:

$$\boxed{\beta_{m,I} = \rho_{m,I} \frac{\sigma_m}{\sigma_I} = \frac{\text{Cov}(\Delta S_m, \Delta S_I)}{\text{Var}(\Delta S_I)}}$$

This is the standard OLS estimator.

**Variance Decomposition**

A useful identity connects beta, $R^2$, and variance:

$$\text{Var}(\Delta S_m) = \beta_{m,I}^2 \cdot \text{Var}(\Delta S_I) + \text{Var}(\epsilon_m)$$

Rearranging:

$$R^2 = \frac{\beta_{m,I}^2 \cdot \text{Var}(\Delta S_I)}{\text{Var}(\Delta S_m)} = \frac{\text{Systematic variance}}{\text{Total variance}}$$

This decomposition shows that $R^2$ measures *what fraction of the name's spread risk the index hedge can capture*. The complement $(1 - R^2)$ is the fraction that remains unhedged—the idiosyncratic component.

> **Desk Reality: Beta Estimation Workflow**
>
> **Data requirements:**
> - **History length**: Use enough observations to get a stable estimate, but short enough to reflect the current regime. Many desks use a rolling window of daily changes and monitor stability across short vs long windows.
> - **Frequency**: Daily spread changes preferred over weekly for CDS (weekly loses information)
> - **Data quality**: Stale quotes are the enemy—exclude days where the name didn't trade
>
> **Estimation steps:**
> 1. Collect daily spread changes: $\Delta S_m(t) = S_m(t) - S_m(t-1)$
> 2. Collect corresponding index changes: $\Delta S_I(t) = S_I(t) - S_I(t-1)$
> 3. Run OLS: regress $\Delta S_m$ on $\Delta S_I$
> 4. Extract: $\beta$ (slope), $\alpha$ (intercept, usually near zero), $R^2$ (fit quality)
>
> **Outlier handling:**
> - **Truncation**: Cap extreme moves at ±3 standard deviations
> - **Winsorization**: Replace outliers with boundary values
> - **Exclusion**: Drop days with major credit events (these are jump risk, not spread risk)
>
> **Common pitfalls:**
> - **Stale quotes**: Illiquid names appear to have low beta because their quotes don't update
> - **Regime changes**: Beta estimated in calm markets may not hold during stress
> - **Survivorship**: Only surviving names are in the sample—beta of eventual defaulters may differ

**When Beta Hedging Helps (and When It Doesn't)**

| Condition | What It Usually Means |
|-----------|------------------------|
| High $R^2$ | Index explains a large fraction of spread variance; hedge can materially reduce day-to-day P&L swings |
| Low $R^2$ | Large residual idiosyncratic component; hedge will still leave meaningful tracking error |
| Unstable correlation / regime shift | Historical beta may be stale; hedge ratios can drift when you need them most |

> **Practitioner Note: Beta Instability in Stress**
>
> Historical beta estimates assume stable correlation regimes. In credit crises, two phenomena occur simultaneously:
> 1. **Correlation spikes**: All credits become more correlated as systematic risk dominates
> 2. **Volatility spikes**: Both name and index volatility increase dramatically
>
> The net effect on beta is ambiguous: $\beta = \rho \cdot \sigma_m / \sigma_I$ may increase, decrease, or stay roughly constant depending on how the ratio of volatilities moves relative to correlation.
>
> **Practical response**: During stress, assume beta estimates are unreliable. Consider hedging both CS01 (for spread moves) and VOD (for default risk) separately.

### 47.2.5 Measuring Hedge Effectiveness

Beyond scenario analysis, practitioners need quantitative metrics to monitor hedge quality over time. This section introduces the key measures.

**Hedge Effectiveness Ratio (HE)**

The fundamental measure compares hedged vs unhedged P&L volatility:

$$\boxed{\text{HE} = 1 - \frac{\text{Var}(\text{hedged P\&L})}{\text{Var}(\text{unhedged P\&L})}}$$

Interpretation:
- $\text{HE} = 1.0$: Perfect hedge (zero residual variance)
- $\text{HE} = 0.0$: Hedge provides no variance reduction
- $\text{HE} < 0$: Hedge *increases* variance (possible with badly-sized hedges)

**Connection to $R^2$**

For a regression-based hedge with optimal beta, hedge effectiveness approximately equals $R^2$:

$$\text{HE} \approx R^2 \quad \text{(when hedge ratio is optimal)}$$

This follows because the hedged P&L variance equals the residual variance from the regression, and $R^2 = 1 - \text{Var}(\epsilon) / \text{Var}(y)$.

**Tracking Error**

An alternative measure is tracking error—the standard deviation of hedge residuals:

$$\text{TE} = \sigma(\text{Hedged P\&L})$$

This is expressed in dollar terms and is useful for P&L attribution.

**Hit Ratio**

For directional hedges, some practitioners track the *hit ratio*—the fraction of days where the hedge and position move in opposite directions:

$$\text{Hit Ratio} = \frac{\text{\# days position and hedge have opposite-sign P\&L}}{\text{Total days}}$$

A perfect hedge has hit ratio = 100%. A random hedge has hit ratio ≈ 50%.

> **Desk Reality: HE Thresholds Are Desk-Specific**
>
> Different desks set different action thresholds depending on mandate (market-making vs prop), instruments used, and risk limits. A robust approach is:
> - Compute HE on a P&L series that matches how you actually mark (including carry/accrual, slippage assumptions, and factor updates).
> - Backtest HE across multiple regimes and pick thresholds that align with risk governance (when to resize, when to add single-name overlays, when to stop using the proxy).
> - Monitor *deterioration* (trend breaks) as well as absolute levels.

**Rolling Window Approach**

Hedge effectiveness should be monitored continuously:

1. Compute HE over a rolling window (short and long)
2. Track the HE time series for trend detection
3. Alert when HE deteriorates materially or persistently

> **Practitioner Note: When Hedge Effectiveness Metrics Mislead**
>
> HE and $R^2$ measure performance under *normal* market conditions. They can be misleading because:
> 1. **Calm periods inflate HE**: Low-volatility periods show high HE because both numerator and denominator are small—the ratio may not hold during stress
> 2. **Tail events not captured**: A hedge that works 95% of the time but fails catastrophically in the 5% tail may show high HE
> 3. **Window dependence**: Shorter windows are more responsive but noisier; longer windows may include irrelevant regimes
>
> **Best practice**: Supplement HE with scenario analysis (Section 47.2.6) and stress testing.

### 47.2.6 Validating with Scenario Sets

CS01 matching ensures hedge effectiveness for *parallel spread moves by construction*. To validate hedge quality for realistic market behavior, run a comprehensive scenario suite:

| Scenario | What It Tests | Key Residual Revealed |
|----------|---------------|----------------------|
| Parallel ±10 bp | CS01 effectiveness | Should be near-zero |
| Dispersion (same average) | Idiosyncratic exposure | Name-level mismatches |
| Single-name gap (+50 bp) | Concentration risk | Unhedged name P&L |
| Default event (FP sweep) | Jump-to-default exposure | Event/recovery risk |
| Roll/series basis | Series mismatch | Liquidity/membership risk |
| Bid-ask widening | Execution feasibility | Unwind costs |

A hedge that passes parallel tests but fails dispersion scenarios may be inappropriate for a concentrated position.

---

## 47.3 Hedging Mathematics

### 47.3.1 CS01-Based Hedge Ratio (Two Instruments)

Consider hedging instrument A (your position) with instrument B (the hedge).

**Setup:**
- $\text{CS01}_A$: CS01 of your current position (includes its notional)
- $\text{CS01}_B(1)$: CS01 per \$1 of notional in hedge instrument B

**Total portfolio CS01:**

$$\text{CS01}_{\text{tot}} = \text{CS01}_A + N_B \cdot \text{CS01}_B(1)$$

**Hedge condition** ($\text{CS01}_{\text{tot}} = 0$):

$$\boxed{N_B^* = -\frac{\text{CS01}_A}{\text{CS01}_B(1)}}$$

**Sign sanity check:** If A is long protection (typically negative CS01 under O'Kane's convention for the holder) and B is short protection (positive CS01 per notional), then $N_B^* > 0$—you sell protection on the hedge instrument.

### 47.3.2 Multi-Factor Hedge (Two Risk Buckets)

When hedging across the term structure, matching a single CS01 is insufficient. If your exposure has distinct 5Y and 10Y components, you need two hedge instruments to neutralize both.

**Setup:**
- Position A has bucketed CS01s: $(\text{CS01}_A^{(5)}, \text{CS01}_A^{(10)})$
- Hedge instruments B and C have CS01 vectors (per \$1mm): $(\text{CS01}_B^{(5)}, \text{CS01}_B^{(10)})$ and $(\text{CS01}_C^{(5)}, \text{CS01}_C^{(10)})$

**System of equations:**

$$\begin{pmatrix} \text{CS01}_A^{(5)} \\ \text{CS01}_A^{(10)} \end{pmatrix} + x_B \begin{pmatrix} \text{CS01}_B^{(5)} \\ \text{CS01}_B^{(10)} \end{pmatrix} + x_C \begin{pmatrix} \text{CS01}_C^{(5)} \\ \text{CS01}_C^{(10)} \end{pmatrix} = \begin{pmatrix} 0 \\ 0 \end{pmatrix}$$

Solve the 2×2 linear system for hedge notionals $(x_B, x_C)$.

**Unit check:** Each CS01 component is \$/bp; $x_B, x_C$ are notionals (dollars).

### 47.3.3 Least-Squares Extension (Many Constituents)

When hedging an index with multiple single names, you may have more potential hedges than risk factors. A least-squares approach minimizes residual CS01:

$$\min_{\{x_m\}} \left\| \text{CS01}_{\text{index}} + \sum_m x_m \cdot \text{CS01}_m \right\|^2$$

subject to constraints such as:
- Non-negativity (can't short protection on illiquid names)
- Maximum notional per name
- Trade only top $K$ names for efficiency

**NOT SURE:** The exact optimization constraints are desk-specific (liquidity universe, concentration limits, minimum clip sizes, restricted names, and whether you can short protection). Provide your risk policy / execution constraints to turn this into an implementable optimization.

### 47.3.4 Systemic vs Idiosyncratic Delta: The O'Kane Framework

O'Kane's treatment of tranche hedging in Chapter 17 introduces a fundamental distinction that applies directly to index hedging: the difference between *systemic delta* and *idiosyncratic delta*. Understanding this distinction is critical for designing hedges that perform as expected.

**Idiosyncratic delta** measures the sensitivity to a single name's spread moving while all other names remain fixed. As O'Kane explains: "The idiosyncratic delta is calculated by widening one credit while keeping all of the other credits fixed. A scenario like this has only a small impact on the shape of the portfolio loss distribution. Its main effect is to make this one credit more likely to default before the rest of the credits."

**Systemic delta** measures the sensitivity to all spreads moving together. O'Kane notes: "This is quite different from how the systemic delta is calculated. This involves widening all of the issuer curves simultaneously, which will shift the entire portfolio loss to the right and make all credits more likely to impact more senior tranches. There is no change in their order of default."

**Why the distinction matters for hedge sizing:**

For indices (not just tranches), these two approaches can produce different hedge ratios. Consider hedging a single name with an index:

- **Idiosyncratic hedge**: Sizes the hedge assuming only the single name moves
- **Systemic hedge**: Sizes the hedge assuming all credits move together

O'Kane provides numerical examples for tranches that illustrate the magnitude of difference:

| Tranche | Total Systemic Delta (\$m) | Total Idiosyncratic Delta (\$m) |
|---------|---------------------------|--------------------------------|
| 0-3% | 691 | 615 |
| 3-7% | 461 | 542 |
| 7-10% | 170 | 167.5 |
| 10-15% | 133 | 108.75 |

The differences are material—over 10% in some cases.

**The practical reality**: O'Kane acknowledges that "In practice, spread movements exhibit a mixture of systemic and idiosyncratic moves and the correct hedge will be somewhere between the idiosyncratic hedge and the systemic hedge. To know where in this range would require us to know in advance the likelihood of idiosyncratic versus systemic spread moves."

**Practitioner approaches:**

1. **Factor models**: "An empirical approach to this problem might be to construct a factor model for spreads which we can calibrate to historical spread data and use to predict the proportion of systemic versus idiosyncratic moves."

2. **Discretionary adjustment**: O'Kane suggests that "the trader to take a view on the relative importance of systemic versus idiosyncratic risk depending on current events in the credit market. For example, suppose the trader expects that a specific credit is about to be downgraded. If this credit is a small issuer in the credit markets and if its downgrade would be for a totally idiosyncratic reason, the trader will tend to adjust his hedges to make sure that credits in the portfolio are hedged according to their idiosyncratic delta."

3. **Hybrid approach**: "However, if the credit is a large issuer whose downgrade could have a contagious effect on other credits, then the trader would hedge this name idiosyncratically using CDS and then hedge the other credits systemically using the CDS indices."

O'Kane summarizes: "In practice, the delta hedging of these risks is often as much an art as it is a science, i.e. it requires judgement and experience, not just a model."

**Decision Heuristics for Systemic vs Idiosyncratic Weighting:**

| Market Condition | Weight Toward | Rationale |
|------------------|---------------|-----------|
| Broad macro catalyst expected | More systemic | Many spreads can move together |
| Name-specific catalyst expected (earnings, legal, M&A) | More idiosyncratic | One name can gap relative to peers |
| Large benchmark issuer in diversified book | Blend | Can be both macro and name-driven |
| Small/illiquid or distressed issuer | More idiosyncratic | Name-level dynamics dominate |
| Stress / risk-off regime | More systemic | Correlations and co-movement often rise |
| Calm / stable regime | Data-driven | Factor models and historical betas tend to be more stable |

**Application to index hedging:**

When hedging single names with an index (or vice versa):
- If you expect a name-specific event (earnings, legal, sector-specific), weight toward idiosyncratic hedge sizing
- If you expect a macro credit move (risk-on/risk-off, rates), weight toward systemic hedge sizing
- Default to something in between, adjusted for your view on market conditions

### 47.3.5 P&L Decomposition Template

A risk-first decomposition of index book P&L:

$$\Delta V \approx -\text{CS01}_I \cdot \Delta S_I - \sum_m \text{CS01}_m \cdot \Delta S_m + \sum_{\text{defaults}} \Delta V_{\text{event}} + \Delta V_{\text{basis}} + \text{higher-order terms}$$

where:
- First term: systematic spread P&L (index level move)
- Second term: idiosyncratic P&L (constituent-specific moves)
- Third term: event P&L (defaults, using VOD-style decomposition)
- Fourth term: basis P&L (quoted vs intrinsic moves)
- Higher-order: convexity, curve twists, cross-gamma

---

## 47.4 Top-Down vs Bottom-Up Hedging

This section provides detailed guidance on when to use each approach.

### 47.4.1 Top-Down Hedge: Single Name → Index

**Use case:** You hold single-name CDS positions and want to hedge systematic credit risk while retaining idiosyncratic exposure (or accepting it as residual).

**Mechanics:**
Define a spread beta $\beta_{m,I}$ such that:

$$\Delta S_m \approx \beta_{m,I} \cdot \Delta S_I + \varepsilon_m$$

where $\varepsilon_m$ is the idiosyncratic component.

**CS01 matching hedge ratio:**

$$N_I = -\frac{\text{CS01}_m}{\text{CS01}_I(1)}$$

**What you keep:**
- Idiosyncratic spread risk ($\varepsilon_m$ shocks)
- Name–index basis (liquidity differences, contract differences)
- Default-event mismatch (single-name VOD vs index VOD mechanics)

**When to use top-down:**

| Situation | Top-Down Appropriate? |
|-----------|----------------------|
| Hedging a diversified credit portfolio | Yes—index matches systematic component |
| Concentrated position (>5% in one name) | Partial—consider adding single-name hedges |
| View on specific name vs market | Yes—top-down preserves idiosyncratic bet |
| Very tight hedge required | No—use bottom-up or name-specific hedges |

### 47.4.2 Bottom-Up Hedge: Index → Constituents

**Use case:** You hold an index position and want to replicate it with single names (perhaps to avoid index liquidity at certain tenors, or to express basis views).

**Starting point:** O'Kane's intrinsic value representation:

$$V_I(t) = \frac{1}{M} \sum_{m=1}^{M} \bigl(C(T) - S_m(t,T)\bigr) \cdot \text{RPV01}_m(t,T)$$

**Hedge ratio logic:** Choose constituent notionals $\{N_m\}$ so combined CS01 matches index CS01. For full replication, trade all $M$ names; for partial replication, trade only the most liquid or highest-impact names.

**What you keep:**
- Bid-ask dispersion: many single names have wider costs than the index
- Missing constituents: incomplete replication leaves residual
- Index basis: quoted index may not equal intrinsic
- Membership evolution: defaults remove names; you must adjust

**When to use bottom-up:**

| Situation | Bottom-Up Appropriate? |
|-----------|------------------------|
| Index options hedging | Yes—need constituent-level greeks |
| Tranche hedging | Yes—tranche risk is constituent-driven |
| Basis trading (long index, short intrinsic) | Yes—this *is* the trade |
| Simple macro hedge | No—index hedge is more efficient |

### 47.4.3 Decision Framework

```
START: What is your primary exposure?
  │
  ├─ Systematic credit spread → USE INDEX HEDGE
  │   └─ Is idiosyncratic concentration material?
  │       ├─ No → Index hedge sufficient
  │       └─ Yes → Add single-name overlays
  │
  ├─ Index position needing granular control → USE BOTTOM-UP
  │   └─ Is precision worth the cost?
  │       ├─ No → Accept basis residual
  │       └─ Yes → Full replication with PSA adjustment
  │
  └─ Basis view between index and constituents → USE BOTH
      └─ Construct basis-neutral position
```

### 47.4.4 Cost Comparison

Transaction costs often determine hedge choice. As a rule of thumb, the index is usually cheaper to trade (one line item, tighter markets), while replicating with many single names is operationally heavier and exposes you to many bid-ask spreads.

O'Kane notes that liquidity differences contribute to index basis: "The considerable size and liquidity of the CDS index market means that the CDS index spread embeds a lower liquidity risk premium than the less liquid CDS spreads."

**Illustrative arithmetic:** execution cost (USD) ≈ bid-ask (bp) × CS01 (USD/bp).
Example: for a CS01 of \$22,500/bp, a 0.25 bp bid-ask costs ≈ \$5,625. If the effective all-in replication cost were 1.5 bp, that would be ≈ \$33,750. Plug in your own bid-ask and CS01.

### 47.4.5 Investment Grade vs High Yield Hedge Differences

The dramatic difference in spread dispersion between IG and HY indices has profound implications for hedge reliability. O'Kane's Table 10.5 documents this clearly.

**Dispersion Comparison (from O'Kane Table 10.5):**

| Index | Maturity | Mean Spread (bp) | Std Dev (bp) |
|-------|----------|------------------|--------------|
| iTraxx Europe | 5Y | 26.9 | 20 |
| iTraxx Europe | 10Y | 45.8 | 30 |
| CDX NA HY | 5Y | 275 | 340 |
| CDX NA HY | 10Y | 324 | 412 |

**Key observation (O'Kane example data):** HY spread dispersion is **10–17× higher** than IG. At 5Y in his table, HY std dev (340 bp) is 17× IG std dev (20 bp).

**Implications for Hedging (Qualitative):**

| Feature | IG Indices | HY Indices |
|---------|------------|-----------|
| Spread dispersion | Lower | Much higher (driven by distressed names) |
| Hedge residuals (tracking error) | Often lower | Often higher (more idiosyncratic moves) |
| Sensitivity to outliers | Lower | Higher (a few wide names can dominate dispersion) |
| Beta stability | Tends to be more stable | Can shift quickly as names become distressed or recover |

> **Desk Reality: HY Hedging Is Fundamentally Noisier**
>
> O'Kane explains the HY dispersion: "The very high standard deviation of the high-yield index spreads can be explained by a number of distressed credits with spreads greater than 1000 bp which were in this index on the date these spreads were quoted."
>
> **Practical consequences:**
> 1. **Expect larger residuals:** HY index hedges often leave more idiosyncratic tracking error than IG hedges.
> 2. **Beta can be unstable:** When a name transitions into/out of distress, its relationship to the index can change.
> 3. **Outliers matter:** A small set of very wide names can dominate dispersion measures and drive hedge residuals.

This doesn't mean HY hedges are worthless—they can still capture systematic credit risk. But expectations should be set by *measurement*: estimate beta and residual volatility on your own history, and re-check during regime changes.

### 47.4.6 Proxy Hedging Across Index Families

When the ideal hedge instrument isn't available—perhaps your exposure is to emerging market corporates but CDX.EM is sovereign-focused—you need a proxy hedge using a different index family.

**Index Family Landscape (from O'Kane Table 10.1):**

| Index | Type | Number of Credits |
|-------|------|-------------------|
| CDX.NA.IG | North American investment grade | 125 |
| CDX.NA.HY | North American high yield | 100 |
| CDX.NA.XO | North American crossover | 35 |
| CDX.EM | Emerging market sovereign | 15 |
| CDX.EM.Diversified | EM sovereign and corporate | 40 |
| iTraxx Europe | European investment grade | 125 |

**Cross-Index Beta**

When hedging with a different index family, introduce the *index-to-index beta*:

$$\beta_{\text{position, hedge}} = \frac{\text{Cov}(\Delta S_{\text{position}}, \Delta S_{\text{hedge index}})}{\text{Var}(\Delta S_{\text{hedge index}})}$$

**Proxy Hedge Menu (Qualitative):**

| Position Type | Natural Hedge | Proxy Hedge | What You Must Estimate | Key Residual Risks |
|---------------|---------------|-------------|------------------------|-------------------|
| US IG corporate | CDX.NA.IG | iTraxx Europe | Cross-index beta | Regional and sector differences |
| US HY corporate | CDX.NA.HY | CDX.NA.IG | IG↔HY beta | Rating/distress dynamics, dispersion |
| EM sovereign | CDX.EM | — | — | Country-specific and event risk |
| EM corporate | CDX.EM.Div | CDX.EM | Corporate↔sovereign beta | Composition mismatch |
| European HY | iTraxx Crossover | iTraxx Europe | Crossover↔Main beta | Rating migration and dispersion |
| Single BB name | CDX.NA.HY | CDX.NA.IG | Name↔index beta | Sector mismatch and idiosyncratic events |

> **Practitioner Note: When Proxy Hedges Break**
>
> Cross-index hedging assumes stable correlation between index families. This breaks down when:
>
> 1. **Regime changes (risk-on/risk-off flips)**: IG and HY can decouple—HY may widen while IG is stable, or vice versa
> 2. **Regional divergence**: US and European credit can move independently during local events (European sovereign crisis, US banking stress)
> 3. **EM-specific crises**: CDX.EM may spike while developed market indices are calm
> 4. **Liquidity divergence in stress**: Proxy may become illiquid precisely when you need to adjust
>
> **Practical expectation**: Proxy hedges often leave materially larger residual volatility than the natural hedge. Quantify this via backtests (beta + residual volatility) before relying on the proxy.

**Sizing a Proxy Hedge**

Apply the beta-adjusted formula from Section 47.2.4:

$$N_{\text{proxy}} = -\frac{\text{CS01}_{\text{position}}}{\beta_{\text{position, proxy}} \cdot \text{CS01}_{\text{proxy}}(1)}$$

**Example:** Hedge Brazilian corporate with CDX.EM

Setup:
- Position: Long protection on Brazilian corporate, CS01 = -\$8,000/bp
- Proxy: CDX.EM, CS01 per \$1mm = +\$380/bp
- Estimated beta (Brazilian corporate to CDX.EM) = 1.3

$$N_{\text{proxy}} = -\frac{-8{,}000}{1.3 \times 380} = \frac{8{,}000}{494} = 16.2 \text{ mm}$$

**Residual risk**: Idiosyncratic Brazilian corporate vs EM sovereign basket, country allocation mismatch, corporate-sovereign basis.

### 47.4.7 Cross-Asset Applications: Credit Hedging Equity

While this chapter focuses on credit-to-credit hedging, credit indices are sometimes used to hedge *equity* portfolios, particularly for tail risk protection.

> **Practitioner Note: Credit Indices as Equity Tail Hedges**
>
> The theoretical basis for credit-equity linkage is the Merton model: equity is a call option on firm assets, and credit spreads reflect the probability that assets fall below the debt threshold. In practice:
>
> **Why it can work:**
> - In equity sell-offs, credit spreads often widen and CDS protection gains value, so credit can provide crash convexity.
> - Indices can be operationally simpler than managing a large basket of single-name CDS as a hedge overlay.
>
> **Why it's imperfect:**
> - **Asymmetric response**: Credit widens when equity falls, but doesn't tighten proportionally when equity rises—you're paying for protection that doesn't participate in upside
> - **Time-varying relationship**: The equity–credit linkage is not stable; correlations and betas move with regime, sector mix, and liquidity
> - **Sector mismatch**: Credit indices and equity benchmarks can have different sector weights and factor exposures
>
> **Sizing approach (recommended):**
> - Treat this as a *tail* hedge, not a delta-one hedge.
> - Pick a stress scenario (equity drawdown and credit spread widening) and size notional so the credit hedge offsets a chosen fraction of the tail loss.
> - Validate with a backtest/regression of equity P&L vs credit hedge P&L and monitor tracking error.
>
> **When to use:**
> - Portfolio has significant equity exposure and seeks crash protection
> - Equity put skew is expensive relative to credit spreads
> - Investor accepts that protection only "works" in severe stress

**NOT SURE:** The “right” hedge ratio depends on your equity benchmark, sector mix, horizon, and hedge objective (tail-loss protection vs day-to-day tracking). Provide those inputs (and preferred calibration window) to compute a desk-usable ratio.

---

## 47.5 Default Events and Index Risk Evolution

### 47.5.1 Mechanics and Risk Consequences

**What happens when a constituent defaults:**

1. **Credit event determination**: Bankruptcy, failure to pay, or restructuring triggers the CDS contract
2. **Auction process**: For cash-settled indices, a credit event auction determines the final price (see Chapter 40)
3. **Protection payment**: Index protection buyer receives $(1 - FP/100)$ per unit notional on the defaulted name's share
4. **Notional reduction**: Index premium leg is reduced by $1/M$ after each default; O'Kane: "the size of the coupon is reduced as the face value is reduced by $1/M$"
5. **Accrued premium**: Protection buyer pays accrued premium for the period since last coupon

**How index risk evolves after defaults:**

After $k$ defaults on an index with original $M$ constituents:
- Outstanding notional fraction: $(M-k)/M$
- Premium payments reduced proportionally
- Remaining names may have different average credit quality
- Index CS01 per trade decreases (fewer premium dollars at risk)

### 47.5.2 VOD for Indices and Tranches: O'Kane's Framework

O'Kane provides detailed mechanics for calculating Value on Default in the context of indices and tranches. While indices are simpler than tranches, understanding the full framework illuminates risk management.

**For indices**, VOD is relatively straightforward: a default in an equal-weight index with $M$ names and notional $N$ produces:

$$\text{VOD}_{\text{index}} = \frac{N}{M}(1 - R) - \text{PV of cancelled premium}$$

where $R$ is the recovery rate and the premium cancellation affects only the defaulted name's share.

**For tranches**, O'Kane describes a more complex calculation that depends on whether the default triggers a loss payment:

*Case 1: Default does not trigger loss payment (tranche still has subordination)*

O'Kane's procedure:
1. "Remove the defaulted credit from the reference portfolio. This will change the average portfolio spread and the notional of the reference portfolio."
2. "Reduce the attachment and detachment points of the tranche so that $K_1 \rightarrow K_1 - H_0(1-R_0)$ and $K_2 \rightarrow K_2 - H_0(1-R_0)$"
3. "Price the tranche using the pricing model giving $V'(t)$"
4. "The VOD equals $V'(t) - V(t)$"

*Case 2: Default triggers loss payment*

If the total loss after default exceeds the attachment point ($L_1 + (1-R_0)H_0 > K_1$), then:
1. Remove defaulted credit from portfolio
2. Make payment of $G = \max[L_1 + H_0(1-R_0) - K_1, 0]$ from protection seller to buyer
3. Reduce tranche notional by $G$
4. Price adjusted tranche to get $V'(t)$
5. "The VOD equals $V'(t) - V(t) \pm G$ where we have a plus for a long protection position and a minus for a short protection position"

**Numerical example from O'Kane** (125-credit portfolio, $10m per name, average spread 60bp, 40% recovery):

| Tranche | Loss Payment (\$m) | Change in MTM (\$m) | VOD (\$m) |
|---------|-------------------|---------------------|-----------|
| 0-3% | +6.0 | +2.596 | +8.596 |
| 3-7% | 0 | +1.398 | +1.398 |
| 7-10% | 0 | +0.321 | +0.321 |
| 10-15% | 0 | +0.195 | +0.195 |
| 15-30% | 0 | +0.085 | +0.085 |

O'Kane explains the perhaps surprising result for equity tranches: "It may seem surprising that the value of the protection has increased even though some of the protection has been used up. To see why, recall that the value of the equity tranche is the value of the protection leg minus the value of the premium leg. While the value of the protection leg has been reduced due to the smaller amount of remaining protection, the value of the premium leg has fallen by even more because of the reduction in the notional on which the very high equity spread has to be paid."

For senior tranches, VOD is positive for protection buyers because "each default means they lose subordination and so become more likely to sustain a loss in the future."

### 47.5.3 Default Timeline and Hedge Adjustment

| Time | Event | Hedge Action Required |
|------|-------|----------------------|
| T+0 | Credit event announced | Assess position impact; begin unwinding single-name hedges if applicable |
| T+1 to T+5 | Auction preparations | Monitor deliverable prices; estimate final price range |
| T+5 to T+10 | Auction conducted | Final price determined; calculate exact payout |
| T+10+ | Settlement | Receive/pay protection amount; adjust notional tracking |

**Post-default hedge rebalancing:**

If you held a CS01-matched hedge before default:
- Your index CS01 decreased (lower notional)
- Your hedge CS01 (if index-based) also decreased proportionally
- Single-name hedges on the defaulted name should be closed
- Re-run CS01 matching with updated index analytics

### 47.5.4 VOD-Style Risk Reporting for Indices

A comprehensive index risk report should include:

| Risk Measure | Definition | Why It Matters |
|--------------|------------|----------------|
| Index CS01 | $-[V(S_I + 1\text{bp}) - V(S_I)]$ | Day-to-day spread risk |
| Constituent CS01s | Per-name sensitivities | Concentration monitoring |
| Aggregate VOD | Sum of $(1-R_m)/M$ exposures | Event risk if any name defaults |
| Worst-case VOD | Max single-name event impact | Tail risk |
| Recovery sensitivity | $\partial \text{Payout}/\partial FP = -N/100$ | Auction outcome risk |

---

## 47.6 Relative Value Frameworks

### 47.6.1 Index Basis: Quoted vs Intrinsic

Chapter 46 develops basis computation in detail. For hedging purposes, the key insight is:

**A "basis-neutral" position** (index vs intrinsic replication) has near-zero parallel CS01 but remains exposed to:
- Liquidity premia shifts
- Restructuring clause differences
- Lead-lag effects (index may move before constituents in stress)
- Dispersion (constituent moves need not average to index move)

O'Kane's list of basis drivers informs which scenarios to stress when validating basis positions.

### 47.6.2 Series Basis / Roll Risk

New on-the-run indices have:
- Longer maturity by ~6 months (wider if curves upward sloping)
- Potentially different membership (credit quality migration, liquidity requirements)
- Concentrated liquidity (old series becomes off-the-run)

**Measuring series basis:**
- Spread difference: $\Delta S = S_{\text{new}} - S_{\text{old}}$
- CS01 mismatch: different maturities have different CS01 per notional
- Membership mismatch: credits in one series but not the other

### 47.6.3 Index vs Single-Name Dispersion

When the index appears rich or cheap versus constituents, your exposure is typically:
- Systematic spread (macro credit level)
- Dispersion (idiosyncratic residual)
- Liquidity differential (index tighter than names)
- Event mapping (basket vs single-name default mechanics)

A "cheap intrinsic" trade (long index, short constituents) profits if the index tightens relative to intrinsic, but loses if dispersion increases without index movement.

---

## 47.7 Worked Examples

### Example 1: Index CS01 via Finite Differences

A short-protection index position has the following PVs at different spread levels:

| Spread Bump | PV (\$) |
|-------------|---------|
| $S - 1$ bp | 54,300 |
| $S$ (base) | 50,000 |
| $S + 1$ bp | 45,800 |

**Central-difference slope:**

$$\frac{\partial V}{\partial S} \approx \frac{V(S+1) - V(S-1)}{2} = \frac{45{,}800 - 54{,}300}{2} = -4{,}250 \text{ \$/bp}$$

**CS01 (O'Kane convention):**

$$\text{CS01}_I = -\frac{\partial V}{\partial S} = +4{,}250 \text{ \$/bp}$$

**Unit check:** dollars per basis point. Positive sign indicates short protection (value falls when spreads rise).

---

### Example 2: Proxy Hedge — Single Name Hedged with Index

**Position:** Long protection on Ford CDS, \$20mm notional, CS01 = -4,900 \$/bp (long protection is negative under O'Kane convention).

**Hedge instrument:** CDX.IG, CS01 per \$1mm (short protection) = +425 \$/bp.

**Hedge ratio:**

$$N_I = -\frac{\text{CS01}_{\text{Ford}}}{\text{CS01}_I(1)} = -\frac{-4{,}900}{425} = 11.53 \text{ mm}$$

**Action:** Sell protection on \$11.53mm CDX.IG notional.

**Verification (parallel +10 bp):**
- Ford P&L: $-(-4{,}900) \times 10 = +49{,}000$
- Index P&L: $-(+4{,}900) \times 10 = -49{,}000$
- Net: \$0 (hedged for parallel moves)

---

### Example 3: Idiosyncratic Move Residual

Same hedge as Example 2.

**Scenario:** Ford widens +50 bp, CDX.IG widens +10 bp.

**P&L calculation:**
- Ford: $-(-4{,}900) \times 50 = +245{,}000$
- Index hedge: $-(+4{,}900) \times 10 = -49{,}000$
- **Net: +\$196,000**

**Interpretation:** The hedge removed systematic risk but the +40 bp idiosyncratic move (Ford vs index) generated substantial P&L. This is the dispersion risk inherent in top-down hedging.

---

### Example 4: Beta-Adjusted Proxy Hedge

**Setup:** You estimate Ford has spread beta $\beta_{\text{Ford,IG}} = 1.5$ versus CDX.IG (Ford moves 1.5 bp for every 1 bp index move, on average).

**Position:** Long protection on Ford, CS01 = -4,900 \$/bp.
**Index:** CDX.IG, CS01 per \$1mm = +425 \$/bp.

**Beta-adjusted hedge ratio:**

If Ford's CS01 to *its own spread* is -4,900, and Ford moves $1.5 \times$ index, then Ford's CS01 to *index moves* is $-4{,}900 \times 1.5 = -7{,}350$ \$/bp of index.

Hedge notional should be:
$$N_I = -\frac{-7{,}350}{425} = 17.29 \text{ mm}$$

**Verification:**
- Index +20 bp → Ford expected +30 bp
- Ford P&L: $-(-4{,}900) \times 30 = +147{,}000$
- Index hedge P&L: $-(425 \times 17.29) \times 20 = -147{,}000$
- Net: \$0

**Lesson:** Beta adjustment requires careful definition of "CS01 to what."

---

### Example 5: Dispersion Shock — Same Average, Different P&L

Portfolio of 5 single names, each \$10mm notional, short protection:

| Name | CS01 (\$/bp) | $\Delta S$ (bp) | P&L |
|------|--------------|-----------------|-----|
| 1 | 300 | +30 | -9,000 |
| 2 | 250 | -10 | +2,500 |
| 3 | 200 | +20 | -4,000 |
| 4 | 350 | -5 | +1,750 |
| 5 | 300 | -35 | +10,500 |
| **Total** | 1,400 | **0** (avg) | **+1,750** |

**Interpretation:** Average spread move is zero, yet net P&L is +\$1,750. This is dispersion risk—CS01-weighted moves don't cancel even when simple averages do.

---

### Example 6: Bottom-Up Intrinsic Hedge

**Index position:** Short protection, \$50mm notional, CS01 = +22,500 \$/bp.

**Five constituents, CS01 per \$1mm (short protection):**

| Name | CS01/\$1mm | Equal-weight notional | Hedge CS01 |
|------|------------|----------------------|------------|
| 1 | 500 | 10mm | +5,000 |
| 2 | 400 | 10mm | +4,000 |
| 3 | 480 | 10mm | +4,800 |
| 4 | 520 | 10mm | +5,200 |
| 5 | 450 | 10mm | +4,500 |
| **Total** | 2,350 | 50mm | +23,500 |

**Hedge (long protection on constituents):** Flip sign → -23,500 \$/bp.

**Residual CS01:** $22{,}500 + (-23{,}500) = -1{,}000$ \$/bp.

**Scale to match:**
$$k = \frac{22{,}500}{23{,}500} = 0.9574$$

Hedge each name at $10 \times 0.9574 = 9.574$ mm notional.

**Incomplete replication (name 5 illiquid):**

Hedge only names 1–4 at scaled notional:
- Hedge CS01: $-(500 + 400 + 480 + 520) \times 9.574 = -18{,}191$ \$/bp
- Residual: $22{,}500 - 18{,}191 = +4{,}309$ \$/bp

**Interpretation:** Missing one name leaves material systematic exposure.

---

### Example 7: Basis Position P&L Decomposition

**Position:** Long protection on index (\$50mm), short protection on intrinsic replication (sized to match CS01).

**CS01s:**
- Index (long protection): -22,500 \$/bp
- Constituents (short protection): +22,500 \$/bp
- Net: ~0

**Scenario (i): Parallel widening +10 bp (index and intrinsic both widen)**

Net CS01 ≈ 0 → $\Delta V \approx 0$

**Scenario (ii): Basis tightening 5 bp (index tightens, constituents unchanged)**

$\Delta S_I = -5$ bp, $\Delta S_{\text{const}} = 0$

$$\Delta V = -(-22{,}500) \times (-5) + 0 = -112{,}500$$

**Loss:** Long index protection loses when index tightens.

**Scenario (iii): Dispersion (index unchanged, constituents disperse)**

Constituent moves (bp): [+20, -15, +10, -5, -10] with CS01s summing to 22,500:

| Name | CS01 | $\Delta S$ | P&L |
|------|------|------------|-----|
| 1 | 5,000 | +20 | -100,000 |
| 2 | 4,000 | -15 | +60,000 |
| 3 | 5,500 | +10 | -55,000 |
| 4 | 4,500 | -5 | +22,500 |
| 5 | 3,500 | -10 | +35,000 |
| **Sum** | 22,500 | 0 | **-37,500** |

Index unchanged → index leg P&L ≈ 0.
**Net: -\$37,500** from dispersion.

**Key insight:** Basis-neutral to parallel moves, but not to dispersion or basis-specific moves.

---

### Example 8: Default Event — Cash Settlement

**Index:** $M = 5$, notional \$50mm → per-name notional \$10mm.
**Coupon:** $C = 100$ bp = 0.01.
**Default:** Name 3 defaults; auction final price $FP = 40$ (40% of par).
**Accrual fraction:** $\Delta_0 = 0.20$ years.

**Protection payment (to protection buyer):**

$$\text{Payout} = N_m \left(1 - \frac{FP}{100}\right) = 10{,}000{,}000 \times 0.60 = 6{,}000{,}000$$

**Accrued premium at default (paid by protection buyer):**

$$\text{Accrued} = C \times \Delta_0 \times N_m = 0.01 \times 0.20 \times 10{,}000{,}000 = 20{,}000$$

**Net default cashflow to protection buyer:**

$$6{,}000{,}000 - 20{,}000 = 5{,}980{,}000$$

**Post-default notional:** \$40mm (premium payments reduced by 1/5).

---

### Example 9: Post-Default Hedge Rebalancing

**Before default:**
- Index position: Short protection, \$50mm, CS01 = +22,500 \$/bp
- Hedge: Long protection on another index series, sized for CS01 = -22,500 \$/bp

**After default (notional reduces to \$40mm):**
- Assume CS01 per \$1mm unchanged at 450 \$/bp
- New index CS01 = $450 \times 40 = 18{,}000$ \$/bp
- Hedge CS01 (if same notional adjustment): also reduced proportionally

If both positions had the same defaulted constituent with equal weight, reduction is symmetric. If hedge is on a different index (e.g., different series with different constituents), the default affects them asymmetrically.

**Rebalancing requirement:**
- Recalculate CS01 for both legs
- Adjust hedge notional to restore CS01 match
- Consider whether default correlation assumptions should be updated

---

### Example 10: Recovery Sensitivity Sweep

Per-name notional $N_m = \$10$mm.

| Final Price $FP$ | Recovery Rate | Payout |
|------------------|---------------|--------|
| 25 | 25% | \$7.5mm |
| 40 | 40% | \$6.0mm |
| 55 | 55% | \$4.5mm |

**Range:** \$3.0mm per name for 30-point FP range.

**Sensitivity:** $\partial \text{Payout}/\partial FP = -N_m/100 = -\$100{,}000$ per price point.

---

### Example 11: Roll/Series Mechanics

**Old series:** Quoted spread 95 bp, coupon 100 bp, RPV01 ≈ 4.5 years.
**New series:** Quoted spread 100 bp (longer maturity + curve slope), same coupon.

**Upfront PV in dollars (conceptual):** $U_{\$} = (S - C) \times \text{RPV01} \times N$

- Old: $(0.0095 - 0.0100) \times 4.5 \times \$100\text{mm} = -\$225{,}000$
- New: $(0.0100 - 0.0100) \times 4.5 \times \$100\text{mm} = \$0$

**If you are long protection and roll by closing the old and entering the new**, the net cash exchanged is approximately the upfront difference: about \$225,000 in this example.

---

### Example 12: Execution Cost Impact

**Basis-neutral position:** \$50mm index vs \$50mm constituent replication, net CS01 ≈ 0.

**Illustrative bid-ask half-spreads (assumptions):**
- Index: 0.25 bp
- Constituents (aggregate): 1.50 bp

**Execution cost approximation:**

$$\text{Cost} \approx \text{Half-spread} \times |\text{CS01}|$$

- Index: $0.25 \times 22{,}500 = \$5{,}625$
- Constituents: $1.50 \times 22{,}500 = \$33{,}750$
- **Total round-trip: ~\$39,375**

**Interpretation:** Basis must move more than ~1.75 bp (= \$39,375 / \$22,500) just to cover execution costs. This is consistent with O'Kane's point about liquidity differences driving index basis.

---

### Example 13: Partial Replication with Residual Quantification

**Goal:** Hedge a \$50mm index position using only the 5 largest-CS01 constituents.

**Index CS01:** +22,500 \$/bp (short protection).

**Top 5 constituents by CS01 (per \$1mm):**

| Rank | Name | CS01/\$1mm |
|------|------|------------|
| 1 | A | 600 |
| 2 | B | 550 |
| 3 | C | 520 |
| 4 | D | 500 |
| 5 | E | 480 |
| **Sum** | — | 2,650 |

**Hedge notional (equal-weight, sized to match):**

Per-name notional: solve for $x$ such that $-2{,}650 \times x = -22{,}500$, so $x = 8.49$ mm per name.

**Total hedge:** $5 \times 8.49 = 42.45$ mm.

**Hedged CS01:** $-2{,}650 \times 8.49 = -22{,}500$ \$/bp → Net CS01 ≈ 0.

**But:** 120 names remain unhedged. If these have average CS01 of 180 \$/bp per \$1mm:
- Missing CS01 per name: $180 \times 0.4$ mm (index weight) = 72 \$/bp
- 120 names × 72 \$/bp = 8,640 \$/bp unhedged idiosyncratic exposure

**Partial replication trades off execution efficiency against idiosyncratic completeness.**

---

### Example 14: Hedge Report Template

A risk manager reviewing an index hedging book should see:

```
═══════════════════════════════════════════════════════════════
                    INDEX HEDGE VALIDATION REPORT
                    As of: [Date]  |  Book: [Name]
═══════════════════════════════════════════════════════════════

POSITION SUMMARY
───────────────────────────────────────────────────────────────
Index Position:      CDX.IG Series 42, 5Y
Direction:           Short Protection
Notional:            $50,000,000
Index CS01:          +$22,500 /bp

HEDGE SUMMARY
───────────────────────────────────────────────────────────────
Hedge Type:          Constituent Replication (Top 20)
Hedge Notional:      $48,500,000 (aggregate)
Hedge CS01:          -$21,800 /bp
Net CS01:            +$700 /bp  (0.7% unhedged)

SCENARIO VALIDATION
───────────────────────────────────────────────────────────────
Scenario                    Index P&L    Hedge P&L    Net P&L
───────────────────────────────────────────────────────────────
Parallel +10bp              -$225,000    +$218,000    -$7,000
Parallel -10bp              +$225,000    -$218,000    +$7,000
Dispersion (avg=0)          $0           -$45,000     -$45,000
Single-name +50bp           -$8,000      +$4,500      -$3,500
Default (FP=40)             -$5,980,000  +$5,500,000  -$480,000
Basis +5bp                  -$112,500    $0           -$112,500

RISK LIMITS CHECK
───────────────────────────────────────────────────────────────
Net CS01:            $700      [PASS]  Limit: $5,000
Single-name conc:    4.2%      [PASS]  Limit: 10%
Dispersion VaR:      $125,000  [PASS]  Limit: $200,000
Event exposure:      $480,000  [WARN]  Limit: $500,000

NOTES
───────────────────────────────────────────────────────────────
- 100 constituents unhedged (5% of index weight)
- Basis exposure unhedged; consider if basis view intended
- Default scenario assumes average FP=40; actual may vary
═══════════════════════════════════════════════════════════════
```

---

### Example 15: Beta Estimation from Data

**Data:** 10 days of spread changes (simplified for illustration):

| Day | $\Delta S_{\text{Ford}}$ (bp) | $\Delta S_{\text{CDX.IG}}$ (bp) |
|-----|------------------------------|--------------------------------|
| 1 | +3 | +2 |
| 2 | -2 | -1 |
| 3 | +5 | +3 |
| 4 | +1 | +1 |
| 5 | -4 | -2 |
| 6 | +6 | +4 |
| 7 | -1 | 0 |
| 8 | +2 | +1 |
| 9 | -3 | -2 |
| 10 | +4 | +2 |

**Calculations:**

Mean $\Delta S_{\text{Ford}}$ = 1.1 bp, Mean $\Delta S_{\text{CDX.IG}}$ = 0.8 bp

$\text{Cov}(\Delta S_{\text{Ford}}, \Delta S_{\text{CDX.IG}}) = \frac{1}{n-1}\sum (x_i - \bar{x})(y_i - \bar{y}) = 4.89$

$\text{Var}(\Delta S_{\text{CDX.IG}}) = \frac{1}{n-1}\sum (y_i - \bar{y})^2 = 3.96$

$$\beta = \frac{4.89}{3.96} = 1.23$$

$\text{Var}(\Delta S_{\text{Ford}}) = 11.21$

$$R^2 = \frac{\beta^2 \cdot \text{Var}(Y)}{\text{Var}(X)} = \frac{1.23^2 \times 3.96}{11.21} = 0.53$$

**Interpretation:** Ford has beta of 1.23 to CDX.IG, with $R^2$ = 0.53. The index explains about half of Ford's spread variance in this sample; the remaining half is residual idiosyncratic risk. Whether that is “good enough” depends on your hedge objective.

---

### Example 16: Hedge Effectiveness Calculation

**Setup:** Over 20 trading days, you measure:
- Unhedged position variance: \$400,000,000 (in squared dollars)
- Hedged position variance: \$120,000,000

$$\text{HE} = 1 - \frac{120{,}000{,}000}{400{,}000{,}000} = 1 - 0.30 = 0.70$$

**Interpretation:** The hedge removes 70% of the variance over this measurement window. Whether that is acceptable depends on mandate and risk limits; if your goal is tail protection rather than day-to-day tracking, you may accept lower HE.

---

## 47.8 Practical Notes

### 47.8.1 Common Pitfalls

1. **Mixing quoting regimes**: Index quotes use coupon + upfront; ensure your analytics are consistent

2. **Inconsistent bump definitions**: Bumping the index curve differs from bumping each constituent—pick one and be consistent

3. **Assuming index hedge removes idiosyncratic risk**: It does not. Examples 3 and 5 demonstrate this clearly.

4. **Ignoring roll/series liquidity**: Post-roll liquidity drops can make hedging expensive or impossible

5. **Assuming index default mechanics without verification**: O'Kane's framework provides the concepts, but specific index rulebooks and ISDA definitions govern actual trades. Always verify with documentation for your specific trade.

6. **Trusting beta estimates in stress**: Beta is estimated from historical data; it may not hold during regime changes

7. **Ignoring IG vs HY differences**: HY hedges are fundamentally noisier—plan accordingly

### 47.8.2 Implementation Pitfalls

**Constituent weights and membership:**
O'Kane notes constituents are "fixed for series life, removed on default; new series can change membership." Your analytics must track this.

**Missing data / stale quotes:**
Single-name CDS on illiquid names may have stale quotes. Mark-to-model risk compounds hedging challenges.

**Discount curve / recovery mismatches:**
Ensure consistent assumptions between index and single-name analytics.

**PSA implementation:**
Spread multiplier and bootstrap issues (maturity alignment, interpolation) are flagged by O'Kane as requiring careful handling.

### 47.8.3 Verification Tests

Before relying on any hedge:

1. **Parallel shock test**: CS01-hedged book should show near-zero P&L for parallel $\Delta S$

2. **Dispersion scenario**: Residual P&L should match explained net single-name exposures

3. **Notional scaling**: CS01 and event measures should scale linearly (first-order)

4. **Payout bounds**: For $FP \in [0, 100]$, payout should be in $[0, N]$; sensitivity is $-N/100$ \$/point

5. **Basis attribution**: If basis is hedged, verify basis P&L near-zero; if unhedged, verify it matches basis move × basis CS01

6. **Beta stability check**: Compare short-window vs long-window beta estimates; if they differ materially, investigate (regime change, stale quotes, event risk)

7. **Hedge effectiveness monitoring**: Track HE over rolling windows; alert on decline

---

## 47.9 Summary

### Executive Summary (10 Points)

1. CDS indices bundle many names into one liquid macro instrument; constituents are fixed except that defaults remove names without replacement.

2. Market quotes use a simplified "index curve" and contractual coupon $C(T)$, producing an upfront $U_I(t)$.

3. Intrinsic valuation builds index PV from constituent CDS curves; the gap between quoted and intrinsic is the **index basis**.

4. Basis drivers include restructuring clause differences, liquidity premia, and lead-lag effects where indices may move before single names.

5. Hedging starts with **CS01/Credit DV01 matching**, defined with sign conventions so short protection CS01 is positive.

6. **Top-down hedges** (single name → index) remove macro risk but keep idiosyncratic, basis, and event residuals.

7. **Bottom-up hedges** (index → constituents) aim to replicate intrinsic but keep basis, execution dispersion, and membership evolution risks.

8. **Systemic vs idiosyncratic delta**: O'Kane distinguishes hedges designed for parallel market moves (systemic) from hedges designed for single-name moves (idiosyncratic). The correct hedge is typically "somewhere between."

9. **Default events** change index cashflows and notional; premium leg reduces by $1/M$ increments, and event payout depends on auction final price.

10. **Always validate** with scenarios: parallel, dispersion, single-name gap, default event, roll shift, and execution cost stress.

### Cheat Sheet

**CS01 definition (O'Kane convention):**
$$\text{CS01} = -(V(S+1\text{bp}) - V(S))$$

**First-order P&L:**
$$\Delta V \approx -\text{CS01} \cdot \Delta S$$

**CS01 hedge notional:**
$$N_{\text{hedge}} = -\frac{\text{CS01}_{\text{position}}}{\text{CS01}_{\text{hedge}}(1)}$$

**Minimum variance hedge ratio (Hull):**
$$h^* = \rho \frac{\sigma_S}{\sigma_F}$$

**Spread beta:**
$$\beta_{m,I} = \frac{\text{Cov}(\Delta S_m, \Delta S_I)}{\text{Var}(\Delta S_I)}$$

**Hedge effectiveness:**
$$\text{HE} = 1 - \frac{\text{Var}(\text{hedged P\&L})}{\text{Var}(\text{unhedged P\&L})}$$

**Index basis:**
$$\text{Basis} = S_I - S_I^{\text{intrinsic}}$$

**Default cash settlement payout:**
- Single-name (or per-default name share) notional $N_{\text{name}}$:
  $$\text{Payout} = N_{\text{name}} \left(1 - \frac{FP}{100}\right)$$
- Index trade notional $N_I$ with $M$ names (equal-weight per-name notional $N_I/M$):
  $$\text{Payout per default} = \frac{N_I}{M}\left(1 - \frac{FP}{100}\right)$$

**Systemic vs idiosyncratic (O'Kane):**
- Systemic delta: all spreads move together
- Idiosyncratic delta: one spread moves, others fixed
- Reality: "a mixture of systemic and idiosyncratic moves"

**Validation scenario suite:**

| # | Scenario | Tests |
|---|----------|-------|
| 1 | Parallel ±10bp | CS01 effectiveness |
| 2 | Dispersion (same average) | Idiosyncratic exposure |
| 3 | Single-name gap | Concentration |
| 4 | Default (FP sweep) | Event/recovery |
| 5 | Roll/series shift | Series mismatch |
| 6 | Bid-ask widening | Execution costs |

---

## 47.10 Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Index basis | $S_I - S_I^{\text{intrinsic}}$ | Determines residual when hedging index with constituents |
| CS01 (Credit DV01) | Dollar P&L per 1bp spread move | Primary risk measure for non-default scenarios |
| VOD (Value on Default) | Jump P&L on immediate default | Captures event risk not in CS01 |
| Top-down hedge | Single name hedged with index | Efficient but keeps idiosyncratic exposure |
| Bottom-up hedge | Index replicated with constituents | Precise but expensive |
| Systemic delta | Sensitivity to all spreads moving together | Appropriate for macro/market moves |
| Idiosyncratic delta | Sensitivity to single name moving alone | Appropriate for name-specific events |
| Spread beta | $\beta = \text{Cov}/\text{Var}$ from regression | Scales hedge for high/low-vol names |
| Hedge effectiveness (HE) | $1 - \text{Var(hedged)}/\text{Var(unhedged)}$ | Quantifies hedge quality |
| PSA | Portfolio swap adjustment | Aligns intrinsic with quoted for derivative hedging |
| Roll risk | Series transition effects | Maturity extension, membership change, liquidity shift |

---

## 47.11 Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the index basis? | $S_I - S_I^{\text{intrinsic}}$; drivers include restructuring differences, liquidity, lead/lag |
| 2 | Define Credit DV01/CS01 | $-[V(S + 1\text{bp}) - V(S)]$ for a 1bp parallel spread increase |
| 3 | Why sign convention with negative? | So short-protection DV01 is positive |
| 4 | What is RPV01 conceptually? | Risky annuity: PV of receiving 1 unit spread until default/maturity |
| 5 | What happens to index premium after default? | Reduced by $1/M$ (per-name fraction) |
| 6 | Are constituents replaced after default? | No; removed without replacement for series life |
| 7 | Why can quoted differ from intrinsic? | No-Re vs Mod-Re, liquidity differences, index leads in stress |
| 8 | What is portfolio swap adjustment (PSA)? | Adjust constituent curves so intrinsic matches quoted |
| 9 | What is top-down hedging? | Hedging single-name/portfolio with index proxy |
| 10 | What residual remains in top-down hedge? | Idiosyncratic, name-index basis, event mismatch |
| 11 | What is bottom-up hedging? | Hedging index with constituents (replication) |
| 12 | What residual remains in bottom-up hedge? | Basis, execution dispersion, missing names, membership evolution |
| 13 | What is systemic delta? | Sensitivity when all spreads move together |
| 14 | What is idiosyncratic delta? | Sensitivity when one spread moves, others fixed |
| 15 | What does O'Kane say about choosing between them? | "The correct hedge will be somewhere between" |
| 16 | What is series/roll risk? | Hedging across series with different maturity/membership/liquidity |
| 17 | How is cash settlement payout determined? | Face minus recovery (auction final price) |
| 18 | Sensitivity of payout to final price? | $-N/100$ dollars per price point |
| 19 | Why doesn't index hedge remove idiosyncratic? | Constituents move differently; index captures average only |
| 20 | Best scenario to reveal dispersion risk? | High-dispersion moves with unchanged average |
| 21 | What is spread beta? | Slope from regressing name spread changes on index spread changes |
| 22 | Formula for spread beta? | $\beta = \text{Cov}(\Delta S_m, \Delta S_I) / \text{Var}(\Delta S_I)$ |
| 23 | What does a low $R^2$ mean for beta hedging? | The index explains little of the name’s spread variance; hedge leaves large residual tracking error |
| 24 | How does HY dispersion compare to IG (O'Kane example data)? | Much higher; O'Kane reports HY spread dispersion far larger than IG at comparable maturities |
| 25 | What is hedge effectiveness (HE)? | $1 - \text{Var(hedged)}/\text{Var(unhedged)}$; measures variance reduction |

---

## 47.12 Mini Problem Set

*Solution sketches provided for questions 1–10.*

**1.** Compute CS01 from PVs: $V(S-1) = 101{,}200$, $V(S) = 100{,}000$, $V(S+1) = 98{,}700$.

**Sketch:** Slope $= (98{,}700 - 101{,}200)/2 = -1{,}250$ \$/bp; CS01 $= +1{,}250$ \$/bp.

---

**2.** A long-protection position has CS01 $-6{,}000$ \$/bp. What is first-order P&L for +8bp widening?

**Sketch:** $\Delta V \approx -(-6{,}000) \times 8 = +48{,}000$.

---

**3.** Index CS01 per \$1mm is 420 \$/bp. You need +12,600 \$/bp of hedge CS01. What notional?

**Sketch:** $N = 12{,}600 / 420 = 30$ mm.

---

**4.** Single name widens +40bp, index +15bp. CS01s: name = -5,000 \$/bp, hedge = +5,000 \$/bp. Residual P&L?

**Sketch:** $\Delta V = -(-5{,}000)(40) - (+5{,}000)(15) = 200{,}000 - 75{,}000 = +125{,}000$.

---

**5.** Two-factor hedge: exposures $(10{,}000, 6{,}000)$, instruments B = $(400, 100)$, C = $(100, 300)$ per \$1mm. Solve.

**Sketch:** System: $10{,}000 + 400x_B + 100x_C = 0$; $6{,}000 + 100x_B + 300x_C = 0$. Solution: $x_C = -12.73$ mm, $x_B = -21.82$ mm.

---

**6.** Intrinsic spread from 3 names: spreads = [80, 120, 200] bp, RPV01 weights = [4.0, 3.5, 2.5].

**Sketch:** Numerator = $80(4) + 120(3.5) + 200(2.5) = 320 + 420 + 500 = 1{,}240$. Denominator = 10. Intrinsic = 124 bp.

---

**7.** Cash-settled default on a single name (or on an index per-name share): $N_{\\text{name}} = 5$ mm, $FP = 35$. Payout?

**Sketch:** $5{,}000{,}000 \times (1 - 0.35) = 3{,}250{,}000$.

---

**8.** Accrued premium: $C = 500$ bp, $\Delta_0 = 0.10$ years, $N = 5$ mm.

**Sketch:** $0.05 \times 0.10 \times 5{,}000{,}000 = 25{,}000$.

---

**9.** Index has $M = 100$, notional 200mm. After 3 defaults, remaining notional?

**Sketch:** Per-default reduction = $200/100 = 2$ mm. Remaining = $200 - 3(2) = 194$ mm.

---

**10.** Given 5 days of data: $\Delta S_{\text{name}}$ = [+4, -2, +6, +1, -3] bp, $\Delta S_{\text{index}}$ = [+2, -1, +4, 0, -2] bp. Calculate beta and $R^2$.

**Sketch:** Using sample moments (demean both series): Cov ≈ 9.10, Var(index) ≈ 5.80, so $\beta \\approx 9.10/5.80 = 1.57$. The sample correlation is ≈ 0.986, so $R^2 \\approx 0.986^2 \\approx 0.97$.

---

**11.** Explain why hedging off-the-run with on-the-run leaves residual risk. *(No sketch)*

---

**12.** Describe how PSA changes a bottom-up hedge. *(No sketch)*

---

**13.** Design a 5-scenario validation suite for a basis-neutral position. *(No sketch)*

---

**14.** Given O'Kane's distinction between systemic and idiosyncratic delta, when would you weight toward systemic hedge sizing? *(No sketch)*

---

**15.** Show how dispersion produces P&L with zero average move. *(No sketch)*

---

**16.** How does recovery risk differ between single-name and index defaults? *(No sketch)*

---

**17.** If HE = 0.45 for a single-name → index hedge, is this acceptable? What action would you take?

**Sketch:** HE = 0.45 means the hedge reduced variance by about 45% over the measurement window. Whether this is “acceptable” depends on mandate and risk limits. Next actions: review residual drivers (idiosyncratic moves, basis, event risk), check beta stability, and consider adding single-name overlays if tracking error is too large.

---

**18.** A Brazilian corporate has estimated beta of 1.3 to CDX.EM. Position CS01 = -\$10,000/bp. CDX.EM CS01/\$1mm = +\$400/bp. Calculate proxy hedge notional.

**Sketch:** $N = -(-10{,}000)/(1.3 \times 400) = 10{,}000/520 = 19.23$ mm.

---

**19.** O'Kane reports example dispersion where HY constituent spreads have much higher standard deviation than IG (e.g., 340 bp vs 20 bp at 5Y in his data). What is the ratio of variance proxies $(\\sigma^2)$?

**Sketch:** $(340/20)^2 = 17^2 = 289$.

---

**20.** You hedge a \$100mm equity portfolio with long HY protection. Describe two scenarios where this hedge (a) works well and (b) fails.

**Sketch:** (a) Works: sharp equity sell-off where credit spreads widen broadly and liquidity demand concentrates in credit hedges; the CDS protection gains value and offsets part of the equity loss. (b) Fails: equities grind higher or rotate by sector while credit is stable/tightens slowly; you keep paying premium and the hedge provides little day-to-day offset (tracking error).

---

## References

- Dominic O'Kane, *Modelling Single-name and Multi-name Credit Derivatives* (indices, basis, PSA, systemic vs idiosyncratic deltas)
- John C. Hull, *Options, Futures, and Other Derivatives* (minimum-variance hedge ratio; regression hedging)

## Cross-References

- **Chapter 43 (CDS Risks and Hedging)**: CS01, VOD, and recovery risk definitions for single-name CDS
- **Chapter 44 (CDS Relative Value)**: Single-name curve trades and RV framework
- **Chapter 46 (Intrinsic Index Spread and Index Basis)**: Detailed basis computation and drivers
- **Chapter 40 (CDS Auction Process)**: Recovery determination mechanics
- **Chapter 48-51 (Tranches)**: Extended treatment of systemic vs idiosyncratic delta for tranches

---

*Chapter 47 complete.*
