# Chapter 47: Hedging and Relative Value in CDS Indices

---

## Introduction

You hold USD 20 million notional of Ford CDS protection and want to hedge with CDX.IG. How much index notional do you need? The answer seems straightforward: match CS01s, calculate a hedge ratio, and execute. But the experienced credit trader knows this is where the real work begins.

A CS01-matched hedge can still produce substantial P&L when Ford widens by 50 basis points while the index moves only 10. Your *hedged* position just generated nearly USD 200,000 of profit—or loss—depending on direction. The hedge neutralized *one component* of spread risk, but the residual (idiosyncratic and basis-related) component remained exposed.

This distinction between *systemic* and *idiosyncratic* moves is at the heart of index hedging. A “systemic” bump design assumes many names move together (market-wide repricing). An “idiosyncratic” bump design assumes one name moves while peers stay put. Real spread moves mix the two, so hedge ratios depend on what you are trying to neutralize and what you are willing to keep.

This chapter provides a rigorous, risk-first framework for CDS index hedging and relative value analysis. We cover the complete workflow: identifying what exposures you actually have, selecting appropriate hedge instruments, computing hedge ratios with explicit assumptions, and validating those hedges with scenario analysis. Throughout, we maintain the discipline established in Chapter 43 (CDS Risks) and Chapter 44 (Single-Name RV): define the risk precisely, then design the hedge.

Prerequisites: [Chapter 43 — Risks in CDS and Hedging Strategies](chapter_43_cds_risks_hedging.md); [Chapter 45 — CDS Indices — Structure, Quoting, and Lifecycle](chapter_45_cds_indices_structure_quoting_lifecycle.md); [Chapter 46 — Intrinsic Index Spread and Index Basis](chapter_46_intrinsic_index_spread_and_index_basis.md)  
Follow-on: [Chapter 48 — CDO / Tranche Products — What They Are (Product Map)](chapter_48_cdo_tranche_products_product_map.md) (systemic/idiosyncratic risk generalized to tranches)

## Learning Objectives
- Translate index and single-name CDS quotes into the PV/risk objects you actually hedge (upfront, RPV01, CS01, VOD), with explicit units and sign.
- Size a top-down hedge (single name → index) using CS01 and (optionally) beta, and interpret the residual tracking error.
- Size a bottom-up hedge (index → constituents) and explain what risks remain (basis, dispersion, lifecycle/defaults, execution).
- Distinguish systemic vs idiosyncratic bump designs and why “what is being bumped?” changes hedge ratios.
- Validate a hedge with a scenario suite and a simple hedge-effectiveness diagnostic.

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

## Bumps, Units, and Signs

This chapter is **risk-first**: we describe exposures, hedges, residuals, and validation—not trade recommendations.

We avoid contract/rulebook specifics (exact roll calendars, settlement mechanics, series coupons) unless they are explicitly stated as an assumption in a worked example. Where a statement depends on your rulebook/confirmation or valuation policy, treat it as an input and restate it in your own documentation.

All spreads are in bp unless stated; conversion: $1 \text{ bp} = 10^{-4}$ (decimal rate per year).

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $t$ | Valuation time (today) |
| $T$ | Maturity date of the CDS / index swap |
| $M$ | Number of index constituents (alive at inception; defaults remove names without replacement) |
| $m \in \\{1, \ldots, M\\}$ | Constituent name index |
| $N$ | Trade notional (USD); $N_I$ for index notional, $N_m$ for constituent notional |
| $S_m(t,T)$ | Market spread for name $m$ to $T$ |
| $S_I(t,T)$ | Quoted index spread (market index-curve level) to $T$ |
| $C(T)$ | Contractual index coupon (fixed for the series) |
| $U_I(t)$ | Upfront as percent of notional (positive = paid by protection buyer; negative = received) |
| $V_I(t)$ | Intrinsic value of the index (constituent-implied) |
| $RPV01(t,T)$ | Risky PV01 (PV of paying 1 bp/year on the premium leg until default/maturity; includes accrued-at-default) |
| CS01 | Spread sensitivity defined as PV change for a **1 bp tightening** (spread down 1 bp) of the stated bump object; units $USD/\text{bp}$; typically positive for short protection |
| VOD | Value-on-default (jump-to-default style risk); see Chapter 43 for the single-name treatment |
| $FP$ | Auction final price / recovery proxy in cash settlement (per 100) |
| $\beta_{m,I}$ | Spread beta: sensitivity of name $m$ spread changes to index spread changes |
| $\rho$ | Correlation coefficient between spread changes |
| $\sigma_m$, $\sigma_I$ | Standard deviation of spread changes for name $m$ and index $I$ |
| HE | Hedge effectiveness ratio |

**Convention bridge (spread up vs spread down):** you will also see CS01 defined as an **up-bump** $PV(\text{spread up }1\text{bp})-PV(\text{base})$, under which long protection is typically positive. For small bumps under the same “held fixed” assumptions, up-bump and down-bump conventions are approximately related by a sign flip. The number only becomes operational once you pin down the bump object (spread vs price), whether the curve is rebuilt, and the sign convention used by your risk reports.

---

## 47.1 Core Concepts

### 47.1.1 The CDS Index as One Trade on Many Names

A CDS portfolio index is a contract on a fixed set of reference entities for the life of the series, giving broad credit exposure in a single transaction. The index pays a fixed contractual coupon $C(T)$ on the outstanding notional and is traded with an upfront $U_I(t)$ that can be positive or negative.

When a constituent defaults, it is removed without replacement. The remaining portfolio continues with reduced notional; under the equal-weight stylization, outstanding notional (and therefore premium) steps down in $1/M$ increments after each default.

**Why indices matter for hedging:**

1. **Liquidity**: Indices often trade tighter bid-ask than underlying single names
2. **Efficiency**: One transaction creates broad credit exposure
3. **Macro hedging**: Ideal for offsetting systematic credit risk in a portfolio
4. **Benchmark**: Indices serve as reference points for relative value analysis

### 47.1.2 Intrinsic Value vs Quoted Index Level

The distinction between *intrinsic* and *quoted* index values is fundamental to index hedging. Chapter 46 develops this in detail; here we summarize the key points.

**Intrinsic value** $V_I(t)$ is the value implied by constituent CDS curves—building the index from the bottom up as an aggregate of single-name values. One representation for the intrinsic value of a short-protection index position is:

$$\boxed{V_I(t) = \frac{1}{M} \sum_{m=1}^{M} \bigl(C(T) - S_m(t,T)\bigr) \cdot RPV01_m(t,T)}$$

**Quoted index level** uses a simplified flat index curve $S_I(t,T)$ to price the index as if it were a single-name contract. This market convention sacrifices precision for quoting and trading convenience.

**Index basis** (in spread terms) is:

$$\text{Basis} = S_I - S_I^{\text{intrinsic}}$$

O'Kane documents that quoted and intrinsic spreads often differ and lists several drivers:

- **Restructuring clause differences**: index vs single-name documentation can differ (e.g., No-Restructuring vs Modified-Restructuring)
- **Liquidity effects**: indices can embed different liquidity premia than individual CDS
- **Lead-lag effects**: in fast markets, index levels can move before many single-name quotes update

For hedging purposes, basis represents a residual risk—you cannot simultaneously match index spread exposure *and* constituent spread exposures unless you account for basis explicitly.

### 47.1.3 Portfolio Swap Adjustment (PSA)

When hedging index products (particularly options or tranches) with constituents, practitioners often use a **portfolio swap adjustment** to reconcile intrinsic and quoted values. PSA is a curve adjustment that forces the intrinsic (built from adjusted constituent curves) to match the market index quote.

A common implementation applies a spread multiplier $\alpha(T)$:

$$S_m^*(t,T) = \alpha(T) \cdot S_m(t,T)$$

and solves for $\alpha(T)$ so that the adjusted intrinsic equals the market index upfront. The adjustment choice is ultimately a modeling decision (stability vs speed vs “reasonableness”) rather than a unique arbitrage-free implication.

**When PSA matters:**
- Hedging index options with constituents
- Tranche pricing and hedging
- Basis trading where you need consistent marks

### 47.1.4 Spread Risk Measures: CS01 and RPV01

**Anchor (pricing identity).** For a CDS-style contract, a convenient mark-to-market decomposition for a long-protection position is:

$$V(t,T)=\text{ProtectionLegPV}(t,T)-S(t,T)\\,RPV01(t,T)$$

RPV01 (“risky PV01”) is the premium-leg PV factor, and it includes the effect of accrued premium paid at default.

**Convention (bump, units, sign).** In this chapter:

- **Bump size:** $1$ bp, where $1\text{ bp} = 10^{-4}$ (decimal spread per year).
- **CS01:** PV change for a **1 bp tightening** of the stated spread factor:

$$\boxed{CS01 := V(S-1\text{ bp}) - V(S)}$$

To first order, $CS01 \approx -\bigl(V(S+1\text{ bp})-V(S)\bigr)$. Units are $USD/\text{bp}$ for the stated notional; short protection typically has **positive** CS01 under this convention.

**Expand (intuition).** CS01 is the “credit DV01”: it tells you how many dollars you make/lose for a small spread move, holding everything else fixed per the bump design. RPV01 is the “risky annuity”: it is the PV of paying/receiving 1 bp per year (on running premium) until default or maturity, reduced by default risk.

**Bump object (make it explicit).** Any CS01 depends on “what is being bumped.” One common choice is to bump the quoted par-spread curve in parallel and recalibrate the hazard/intensity curve consistently, holding the discount curve fixed. If your system bumps hazards directly or bumps constituents instead of the index curve, you must restate CS01 accordingly before sizing hedges.

For a near-par position, a useful approximation is:

$$CS01 \approx N \cdot RPV01 \cdot 10^{-4}\quad (USD /\text{bp})$$

**Check (sign + units).** If you are **short protection** with $N=USD 10$mm and $RPV01=4.2$ years, then $CS01\approx 10{,}000{,}000 \times 4.2 \times 10^{-4}=4{,}200\ USD/\text{bp}$. A +20 bp widening implies $\Delta V \approx -4{,}200 \times 20 = -84{,}000$ (loss for short protection), consistent with “wider spreads hurt long-credit exposure.”

> **Pitfall — What is being bumped?:** “Index CS01” can mean “bump the quoted index par-spread curve” or “bump each constituent spread curve and re-aggregate.”
> **Why it matters:** Those two objects can produce different hedge ratios and different residuals (especially in dispersion and basis scenarios).
> **Quick check:** Run a +10 bp parallel scenario and confirm the PV change matches $\Delta V \approx -CS01\cdot \Delta S$ for the *same bump object* you intend to neutralize.

### 47.1.5 Default-Event Risk: VOD (Value on Default)

Even with perfect spread hedging, a sudden default creates discrete P&L. Chapter 43 defines Value on Default (VOD) for a single-name CDS using O'Kane's formula:

$$\text{VOD}_{\text{buy protection}} = -V(t) + (1-R) - \Delta_0 S_0$$

where $V(t)$ is the CDS mark-to-market immediately before default and $\Delta_0 S_0$ is the accrued premium at default. For a par position with $V(t) \approx 0$, VOD per unit notional is approximately $(1-R) - \Delta_0 S_0$.

For indices, VOD concepts extend with portfolio scaling. A constituent default:

1. Triggers a protection payment proportional to $1/M$ of notional
2. Cancels that name's share of future premium payments
3. Reduces the outstanding index notional

O'Kane's index valuation treats each name as contributing $(1 - R_m)/M$ losses to the index buyer on default.

### 47.1.6 Roll / Series Risk

CDS portfolio indices trade in **series** that roll every six months. A roll changes what the index refers to, so hedges can pick up unintended basis risk if you mix series.

- **Maturity extension:** the new index has a longer maturity, by six months, than the previous index. If credit curves slope upward, the new series can price at a wider spread even with no change in credit quality.
- **Membership changes:** the new series can add/remove constituents (e.g., downgrades or liquidity screens), changing sector mix and idiosyncratic risk.
- **Liquidity migration:** liquidity typically concentrates in the on-the-run series; off-the-run hedges can become more expensive and harder to adjust.

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

2. **Liquidity and cost**: if the index market is deeper/tighter than many single names, an index hedge can be cheaper to execute. Differences in liquidity premia can also show up as index–constituent basis.

3. **Precision vs efficiency**: Single-name hedges are more precise but require more trades and monitoring. Index hedges are efficient but leave idiosyncratic exposure.

### 47.2.3 Computing Hedge Ratios

The fundamental hedge sizing approaches:

**CS01 / Credit DV01 matching**
The primary first-pass method. Choose hedge notional so total CS01 equals zero:

$$N_{\text{hedge}} = -\frac{CS01_{\text{position}}}{CS01_{\text{hedge}}(1)}$$

where $CS01_{\text{hedge}}(1)$ is the CS01 per unit notional of the hedge instrument.

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

$$N_I = -\frac{\beta_{m,I}\\,CS01_m}{CS01_I(1)}$$

**Check (units + toy sizing):** $\beta$ is unitless. If your CS01s are reported in $USD/\text{bp}$ per $USD 10\text{mm}$ notional and include direction (under this chapter’s spread-down convention: long protection typically negative, short protection typically positive), the ratio produces a notional. Example: a $USD 10\text{mm}$ long-protection name position has $CS01_m=-USD 4{,}000/\text{bp}$, the index has $CS01_I(1)=+USD 3{,}000/\text{bp}$ per $USD 10\text{mm}$ of **short** index protection, and $\beta=1.5$. Then $N_I \approx -(1.5\times -4{,}000)/3{,}000 \approx 2$ “units”, i.e., about $USD 20\text{mm}$ of short index protection.

> **Deep Dive: The Regression Hedge**
>
> How do you find $\beta$? One practical approach is an OLS regression on spread changes:
>
> $$\Delta S_{\text{name}} = \alpha + \beta \Delta S_{\text{index}} + \epsilon$$
>
> *   **$\beta$ (Systematic)**: What you hedge.
> *   **$\epsilon$ (Idiosyncratic)**: What you keep.
> *   **$R^2$**: How much of the name's spread variance is explained by the index move. Low $R^2$ means large residual tracking error—an index hedge may still reduce macro exposure, but it will not make the position P&L-stable.

### 47.2.4 Estimating Spread Beta in Practice

While the previous section introduced beta-adjusted hedging conceptually, this section provides the methodology for *estimating* spread beta from market data. This is essential knowledge for practitioners designing proxy hedges.

**The Minimum Variance Hedge Framework**

A minimum-variance hedge chooses a hedge ratio to minimize the variance of the *hedged* change. In a two-variable setting, if you write the hedged change as
$$v = x + hF,$$
then the hedge ratio that minimizes $\mathrm{Var}(v)$ is:
$$\boxed{h^* = -\frac{\mathrm{Cov}(x, F)}{\mathrm{Var}(F)}}$$

If instead you write $v = x - hF$, the optimal $h$ is $+\mathrm{Cov}(x,F)/\mathrm{Var}(F)$. These are the same result with different sign conventions.

**Application to Credit Spreads**

For CDS hedging, take $x=\Delta S_m$ (daily change in single-name spread) and $F=\Delta S_I$ (daily change in index spread). With the residual written as $\Delta S_m-\beta\\,\Delta S_I$, the minimum-variance hedge ratio is the familiar covariance-over-variance slope.

The spread beta formula becomes:

$$\boxed{\beta_{m,I} = \rho_{m,I} \frac{\sigma_m}{\sigma_I} = \frac{\text{Cov}(\Delta S_m, \Delta S_I)}{\text{Var}(\Delta S_I)}}$$

This is also the OLS regression slope from regressing $\Delta S_m$ on $\Delta S_I$.

**Variance Decomposition**

A useful identity connects beta, $R^2$, and variance:

$$\text{Var}(\Delta S_m) = \beta_{m,I}^2 \cdot \text{Var}(\Delta S_I) + \text{Var}(\epsilon_m)$$

Rearranging:

$$R^2 = \frac{\beta_{m,I}^2 \cdot \text{Var}(\Delta S_I)}{\text{Var}(\Delta S_m)} = \frac{\text{Systematic variance}}{\text{Total variance}}$$

This decomposition shows that $R^2$ measures *what fraction of the name's spread risk the index hedge can capture*. The complement $(1 - R^2)$ is the fraction that remains unhedged—the idiosyncratic component.

> **Desk Reality: Beta Estimation Workflow**
>
> **Data requirements:**
> - **History length**: Use enough observations to get a stable estimate, but short enough to reflect the current regime; monitor stability across short vs long windows.
> - **Frequency**: Daily spread changes are a common choice; weekly data can hide short-lived dislocations.
> - **Data quality**: Watch for stale quotes (e.g., days where the name didn’t update).
>
> **Estimation steps:**
> 1. Collect daily spread changes: $\Delta S_m(t) = S_m(t) - S_m(t-1)$
> 2. Collect corresponding index changes: $\Delta S_I(t) = S_I(t) - S_I(t-1)$
> 3. Run OLS: regress $\Delta S_m$ on $\Delta S_I$
> 4. Extract: $\beta$ (slope), $\alpha$ (intercept, usually near zero), $R^2$ (fit quality)
>
> **Outlier handling:**
> - **Truncation**: Cap extreme moves at a chosen threshold (e.g., a few standard deviations)
> - **Winsorization**: Replace outliers with boundary values
> - **Exclusion**: Consider dropping major credit-event days if you want a “diffusive spread” beta (not jump risk)
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

$$\boxed{\text{HE} = 1 - \frac{\text{Var}(\text{hedged P\\&L})}{\text{Var}(\text{unhedged P\\&L})}}$$

Interpretation:
- $\text{HE} = 1.0$: Perfect hedge (zero residual variance)
- $\text{HE} = 0.0$: Hedge provides no variance reduction
- $\text{HE} \lt 0$: Hedge *increases* variance (possible with badly-sized hedges)

**Connection to $R^2$**

In the linear regression setup ($\Delta S \approx a + b\\,\Delta F + \epsilon$) with the optimal hedge ratio, the hedge effectiveness (variance eliminated) is the $R^2$ from the regression and equals $\rho^2$:

$$\text{HE} = R^2 = \rho^2 \quad \text{(within the regression setup)}$$

**Check (how much risk is left):** if $\rho=0.7$, then $R^2\approx 0.49$: the hedge removes about 49% of the *variance*, but the residual standard deviation is still $\sqrt{1-R^2}\approx 0.71$ of the unhedged level. This is why a “high correlation” hedge can still have large day-to-day tracking error.

When you apply this idea to a CDS hedge, treat it as a diagnostic: if the relationship is nonlinear, regime-dependent, or dominated by idiosyncratic jumps, the realized HE can deviate from the historical $R^2$.

**Tracking Error**

An alternative measure is tracking error—the standard deviation of hedge residuals:

$$\text{TE} = \sigma(\text{Hedged P\\&L})$$

This is expressed in dollar terms and is useful for P&L attribution.

**Hit Ratio**

For directional hedges, some practitioners track the *hit ratio*—the fraction of days where the hedge and position move in opposite directions:

$$\text{Hit Ratio} = \frac{\text{\\# days position and hedge have opposite-sign P\\&L}}{\text{Total days}}$$

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
- $CS01_A$: CS01 of your current position (includes its notional)
- $CS01_B(1)$: CS01 per USD 1 of notional in hedge instrument B

**Total portfolio CS01:**

$$CS01_{\text{tot}} = CS01_A + N_B \cdot CS01_B(1)$$

**Hedge condition** ($CS01_{\text{tot}} = 0$):

$$\boxed{N_B^* = -\frac{CS01_A}{CS01_B(1)}}$$

**Sign sanity check:** If A is long protection (negative CS01) and B is short protection (positive CS01 per notional), then $N_B^* \gt 0$—you sell protection on the hedge instrument.

### 47.3.2 Multi-Factor Hedge (Two Risk Buckets)

When hedging across the term structure, matching a single CS01 is insufficient. If your exposure has distinct 5Y and 10Y components, you need two hedge instruments to neutralize both.

**Setup:**
- Position A has bucketed CS01s: $(CS01_A^{(5)}, CS01_A^{(10)})$
- Hedge instruments B and C have CS01 vectors (per USD 1mm): $(CS01_B^{(5)}, CS01_B^{(10)})$ and $(CS01_C^{(5)}, CS01_C^{(10)})$

**System of equations:**

$$\begin{pmatrix} CS01_A^{(5)} \\ CS01_A^{(10)} \end{pmatrix} + x_B \begin{pmatrix} CS01_B^{(5)} \\ CS01_B^{(10)} \end{pmatrix} + x_C \begin{pmatrix} CS01_C^{(5)} \\ CS01_C^{(10)} \end{pmatrix} = \begin{pmatrix} 0 \\ 0 \end{pmatrix}$$

Solve the 2×2 linear system for hedge notionals $(x_B, x_C)$.

**Unit check:** Each CS01 component is USD /bp; $x_B, x_C$ are notionals (dollars).

### 47.3.3 Least-Squares Extension (Many Constituents)

When hedging an index with multiple single names, you may have more potential hedges than risk factors. A least-squares approach minimizes residual CS01:

$$\min_{\\{x_m\\}} \left\\| CS01_{\text{index}} + \sum_m x_m \cdot CS01_m \right\\|^2$$

subject to constraints such as:
- Non-negativity (can't short protection on illiquid names)
- Maximum notional per name
- Trade only top $K$ names for efficiency

The exact optimization constraints are desk-specific (liquidity universe, concentration limits, minimum clip sizes, restricted names, and whether you can short protection). Provide your risk policy / execution constraints to turn this into an implementable optimization.

### 47.3.4 Systemic vs Idiosyncratic Delta: The O'Kane Framework

O'Kane’s tranche-hedging discussion highlights a distinction that carries over directly to index hedging: **“delta” depends on the bump design**.

**Idiosyncratic delta (one-name bump)** widens/tightens one issuer’s spread curve while holding all other issuer curves fixed. This is the right object for name-specific shocks and dispersion risk.

**Systemic delta (all-names bump)** widens/tightens all issuer spread curves simultaneously. This is the right object for broad market repricing where many credits move together.

**Why the distinction matters for hedge sizing:**

For indices (not just tranches), these two approaches can produce different hedge ratios. Consider hedging a single name with an index:

- **Idiosyncratic hedge**: Sizes the hedge assuming only the single name moves
- **Systemic hedge**: Sizes the hedge assuming all credits move together

O'Kane’s tranche examples illustrate that the difference can be material:

| Tranche | Total Systemic Delta (USD m) | Total Idiosyncratic Delta (USD m) |
|---------|---------------------------|--------------------------------|
| 0-3% | 691 | 615 |
| 3-7% | 461 | 542 |
| 7-10% | 170 | 167.5 |
| 10-15% | 133 | 108.75 |

In practice, realized spread moves mix systematic and idiosyncratic components, so hedge sizing often blends the two extremes. Two common approaches are (i) empirical factor/regression models to estimate the systematic fraction, and (ii) scenario-weighted blends (lean systemic in broad macro regimes; lean idiosyncratic around name catalysts). A pragmatic hybrid is to hedge the name-specific piece with single-name CDS and the remainder with indices.

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

$$\Delta V \approx -CS01_I \cdot \Delta S_I - \sum_m CS01_m \cdot \Delta S_m + \sum_{\text{defaults}} \Delta V_{\text{event}} + \Delta V_{\text{basis}} + \text{higher-order terms}$$

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

$$N_I = -\frac{CS01_m}{CS01_I(1)}$$

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

$$V_I(t) = \frac{1}{M} \sum_{m=1}^{M} \bigl(C(T) - S_m(t,T)\bigr) \cdot RPV01_m(t,T)$$

**Hedge ratio logic:** Choose constituent notionals $\\{N_m\\}$ so combined CS01 matches index CS01. For full replication, trade all $M$ names; for partial replication, trade only the most liquid or highest-impact names.

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

Transaction costs often determine hedge choice. An index hedge is one line item and can trade tighter than the full single-name basket, while replication is operationally heavier and exposes you to many bid-ask spreads.

If index and single-name CDS embed different liquidity premia, quoted and intrinsic spreads can diverge even with the same underlying credit risk.

**Illustrative arithmetic:** execution cost (USD) ≈ bid-ask (bp) × CS01 (USD/bp).
Example: for a CS01 of USD 22,500/bp, a 0.25 bp bid-ask costs ≈ USD 5,625. If the effective all-in replication cost were 1.5 bp, that would be ≈ USD 33,750. Plug in your own bid-ask and CS01.

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
> In a high-yield CDS index, a very high standard deviation can be driven by distressed credits with spreads above 1000 bp in the sample, inflating dispersion measures and tracking error.
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

$$N_{\text{proxy}} = -\frac{\beta_{\text{position, proxy}}\\,CS01_{\text{position}}}{CS01_{\text{proxy}}(1)}$$

**Example:** Hedge Brazilian corporate with CDX.EM

Setup:
- Position: Long protection on Brazilian corporate, CS01 = -USD 8,000/bp
- Proxy: CDX.EM, CS01 per USD 1mm = +USD 380/bp
- Estimated beta (Brazilian corporate to CDX.EM) = 1.3

Because $\Delta S_{\text{position}} \approx \beta\\,\Delta S_{\text{proxy}}$, the systematic sizing multiplies the CS01 exposure:

$$N_{\text{proxy}} = -\frac{1.3 \times (-8{,}000)}{380} = \frac{10{,}400}{380} = 27.4 \text{ mm}$$

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
> - Investor accepts that protection primarily pays off in severe stress

There is no universal “right” hedge ratio: it depends on your equity benchmark, sector mix, horizon, and hedge objective (tail-loss protection vs day-to-day tracking). Estimate it via stress-scenario sizing and/or a regression/backtest over a chosen window, then monitor tracking error and regime drift.

---

## 47.5 Default Events and Index Risk Evolution

### 47.5.1 Mechanics and Risk Consequences

**What happens when a constituent defaults:**

1. **Credit event determination**: Bankruptcy, failure to pay, or restructuring triggers the CDS contract
2. **Auction process**: For cash-settled indices, a credit event auction determines the final price (see Chapter 40)
3. **Protection payment**: Index protection buyer receives $(1 - FP/100)$ per unit notional on the defaulted name's share
4. **Notional reduction**: In a CDS index, after each default, the size of the coupon payments is reduced because the face value / outstanding notional is reduced by $1/M$ (equal-weight stylization)
5. **Accrued premium**: Protection buyer pays accrued premium for the period since last coupon

**How index risk evolves after defaults:**

After $k$ defaults on an index with original $M$ constituents:
- Outstanding notional fraction: $(M-k)/M$
- Premium payments reduced proportionally
- Remaining names may have different average credit quality
- Index CS01 per trade decreases (fewer premium dollars at risk)

**Check (factor scale):** with $M=125$, one default reduces the outstanding fraction by $1/125\approx 0.8\\%$. To first order, premium cashflows and index CS01 scale down by the same fraction, so any hedge ratios that ignore the updated factor can drift immediately after defaults.

### 47.5.2 VOD for Indices and Tranches: O'Kane's Framework

O'Kane provides detailed mechanics for calculating Value on Default in the context of indices and tranches. While indices are simpler than tranches, understanding the full framework illuminates risk management.

**For indices**, VOD is relatively straightforward: a default in an equal-weight index with $M$ names and notional $N$ produces:

$$\text{VOD}_{\text{index}} = \frac{N}{M}(1 - R) - \text{PV of cancelled premium}$$

where $R$ is the recovery rate and the premium cancellation affects only the defaulted name's share.

**For tranches**, “VOD” is more complex because a default can (i) change the portfolio loss distribution, (ii) trigger an immediate loss payment for equity/mezz tranches, and (iii) reduce tranche notional and therefore future premium. Computing tranche VOD requires revaluing the tranche after updating portfolio composition and tranche attachment/detachment mechanics; that is out of scope here (see the tranche chapters and follow-on Chapter 48).

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
| Index CS01 | $V(S_I-1\text{bp}) - V(S_I)$ | Day-to-day spread risk |
| Constituent CS01s | Per-name sensitivities | Concentration monitoring |
| Aggregate VOD | Sum of $(1-R_m)/M$ exposures | Event risk if any name defaults |
| Worst-case VOD | Max single-name event impact | Tail risk |
| Recovery sensitivity | $\partial \text{Payout}/\partial FP = -N/100$ | Auction outcome risk |

---

## 47.6 Relative Value Frameworks

### 47.6.1 Index Basis: Quoted vs Intrinsic

Chapter 46 develops basis computation in detail. For hedging purposes, the key insight is:

**A basis-neutral position** (index vs intrinsic replication) has near-zero parallel CS01 but remains exposed to:
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

A cheap intrinsic trade (long index, short constituents) profits if the index tightens relative to intrinsic, but loses if dispersion increases without index movement.

---

## 47.7 Worked Examples

### Example 1: Index CS01 via Finite Differences

A short-protection index position has the following PVs at different spread levels:

| Spread Bump | PV (USD ) |
|-------------|---------|
| $S - 1$ bp | 54,300 |
| $S$ (base) | 50,000 |
| $S + 1$ bp | 45,800 |

**Central-difference slope:**

$$\frac{\partial V}{\partial S} \approx \frac{V(S+1) - V(S-1)}{2} = \frac{45{,}800 - 54{,}300}{2} = -4{,}250 \text{ USD  /bp}$$

**Finite-difference CS01 (this chapter):**

$$CS01_I \approx V(S-1) - V(S) = 54{,}300 - 50{,}000 = 4{,}300\ \text{ USD  /bp}$$

**Cross-check (link to slope):** Since $CS01\approx -\partial V/\partial S$, the central-difference estimate implies $CS01\approx -(-4{,}250)=4{,}250\ USD/\text{bp}$, close to the one-sided $V(S-1)-V(S)$ estimate.

**Unit/sign check:** dollars per basis point. Positive CS01 indicates short protection (value falls when spreads rise).

---

### Example 2: Proxy Hedge — Single Name Hedged with Index

**Example Title**: Hedge a single-name CDS with an index (CS01 match + beta adjustment)

**Context**
- You hold **long protection** on a single name (Ford) and want to hedge the **systematic** component with a liquid index (CDX.IG).
- The desk goal is *not* “eliminate all risk,” but “make P&L less sensitive to broad market moves,” while understanding the residual (idiosyncratic, basis, event risk).

**Timeline (Make Dates Concrete)**
- Trade date: 2026-03-20
- Settlement date (upfront): 2026-03-24 *(example assumption; use your confirmation/venue conventions)*
- Accrual start/end for the next premium period: 2026-03-20 to 2026-06-20
- Next payment date: 2026-06-20 (then quarterly)

**Inputs**
- **Position:** Ford CDS, notional `N_Ford = USD 20mm`, long protection.
  - Risk system output (under this chapter’s CS01 convention): $CS01_{\text{Ford}}=-4{,}900\ USD/\text{bp}$.
- **Hedge instrument:** CDX.IG index (same maturity bucket), short protection CS01 per USD 1mm:
  - $CS01_{I}(1)=+425\ USD /\text{bp per USD 1mm}$.
- **Cashflow assumptions (for settlement intuition only):** index coupon $C=100$ bp, quarterly ACT/360; quoted upfront to the protection buyer $U=2.20\\%$ of notional. *(All numbers are hypothetical inputs for the example.)*

**Outputs (What You Produce)**
- CS01-matched index hedge notional: `N_I = 11.53mm` (sell protection).
- Beta-adjusted index hedge notional (if $\beta_{\text{Ford,IG}}=1.5$): `N_I = 17.29mm`.
- Example settlement cash on the index hedge (seller receives upfront on settlement): $USD 0.0220 \times 11.53\text{mm} \approx USD 253{,}660$ (accrual is zero here because we trade on a coupon date).

**Step-by-step**
1. **Align definitions:** confirm both CS01s use the same bump size (1 bp), bump object (par spreads with hazard recalibration), and units (USD /bp).
2. **Compute CS01 hedge ratio:**
   $$N_I = -\frac{CS01_{\text{Ford}}}{CS01_I(1)} = -\frac{-4{,}900}{425} = 11.53\ \text{mm}$$
3. **Add beta (systematic scaling):** if $\Delta S_{\text{Ford}} \approx \beta\\,\Delta S_I$ with $\beta=1.5$, then size to hedge *systematic* Ford moves:
   $$N_I^{(\beta)} = -\frac{\beta\\,CS01_{\text{Ford}}}{CS01_I(1)} = -\frac{1.5 \times (-4{,}900)}{425} = 17.29\ \text{mm}$$
4. **Validate with scenarios (first-order):**
   - Parallel +10 bp (systematic): CS01-matched hedge should be ~0 P&L by construction.
   - Name-specific gap: Ford +50 bp while index +10 bp leaves residual P&L (dispersion).
   - Basis move: if index vs intrinsic basis moves, an index-vs-constituent hedge can leak even when CS01 matched.

**Cashflows (table)**
| Date | Cashflow (seller of index protection) | Explanation |
|---|---:|---|
| 2026-03-24 | +USD 253,660 | Upfront received on `N_I = 11.53mm` at $U=2.20\\%$ (example) |
| 2026-06-20 | +USD 29,500 | Premium: $C\times \Delta \times N$ with $C=100$ bp and $\Delta\approx 92/360$ |
| Default date (if any constituent defaults) | $-\frac{N_I}{M}\left(1-\frac{FP}{100}\right)$ | Protection payment for one defaulted name share (equal-weight stylization) |

**P&L / Risk Interpretation**
- CS01 matching neutralizes **first-order PV sensitivity** to the chosen spread factor; it does *not* neutralize dispersion (idiosyncratic $\epsilon$), basis, or default-event risk.
- Beta adjustment changes *what you mean by “systematic hedge”*: you are hedging Ford’s expected move conditional on the index moving.

**Sanity Checks**
- Units: $CS01$ in USD /bp and $N$ in USD mm → P&L in dollars.
- Sign: long protection CS01 < 0; selling index protection creates CS01 > 0, so the hedge notional comes out positive.
- Limit: if $\beta=0$, the beta-adjusted notional goes to 0 (index is not a useful systematic hedge for that name).

**Debug Checklist (When Your Result Looks Wrong)**
- Quote direction: are you *buying* or *selling* protection, and is upfront quoted as “to buyer” or “to seller”?
- “What was bumped?”: index par-spread curve vs constituent curves vs hazard rates.
- Bucket mismatch: same maturity and series? if not, expect residual curve/series basis.
- Accrual timeline: trade date vs settlement vs coupon date (accrued premium can dominate small upfronts for tight indices).

---

### Example 3: Idiosyncratic Move Residual

Same hedge as Example 2.

**Scenario:** Ford widens +50 bp, CDX.IG widens +10 bp.

**P&L calculation:**
- Ford: $-(-4{,}900) \times 50 = +245{,}000$
- Index hedge: $-(+4{,}900) \times 10 = -49{,}000$
- **Net: +USD 196,000**

**Interpretation:** The hedge removed systematic risk but the +40 bp idiosyncratic move (Ford vs index) generated substantial P&L. This is the dispersion risk inherent in top-down hedging.

---

### Example 4: Beta-Adjusted Proxy Hedge

**Setup:** You estimate Ford has spread beta $\beta_{\text{Ford,IG}} = 1.5$ versus CDX.IG (Ford moves 1.5 bp for every 1 bp index move, on average).

**Position:** Long protection on Ford, CS01 = -4,900 USD /bp.
**Index:** CDX.IG, CS01 per USD 1mm = +425 USD /bp.

**Beta-adjusted hedge ratio:**

If Ford's CS01 to its own spread is -4,900, and Ford moves $1.5 \times$ index, then Ford's CS01 to index moves is $-4{,}900 \times 1.5 = -7{,}350$ USD /bp of index.

Hedge notional should be:
$$N_I = -\frac{-7{,}350}{425} = 17.29 \text{ mm}$$

**Verification:**
- Index +20 bp → Ford expected +30 bp
- Ford P&L: $-(-4{,}900) \times 30 = +147{,}000$
- Index hedge P&L: $-(425 \times 17.29) \times 20 = -147{,}000$
- Net: USD 0

**Lesson:** Beta adjustment requires careful definition of “CS01 to what?”

---

### Example 5: Dispersion Shock — Same Average, Different P&L

Portfolio of 5 single names, each USD 10mm notional, short protection:

| Name | CS01 (USD /bp) | $\Delta S$ (bp) | P&L |
|------|--------------|-----------------|-----|
| 1 | 300 | +30 | -9,000 |
| 2 | 250 | -10 | +2,500 |
| 3 | 200 | +20 | -4,000 |
| 4 | 350 | -5 | +1,750 |
| 5 | 300 | -35 | +10,500 |
| **Total** | 1,400 | **0** (avg) | **+1,750** |

**Interpretation:** Average spread move is zero, yet net P&L is +USD 1,750. This is dispersion risk—CS01-weighted moves don't cancel even when simple averages do.

---

### Example 6: Bottom-Up Intrinsic Hedge

**Index position:** Short protection, USD 50mm notional, CS01 = +22,500 USD /bp.

**Five constituents, CS01 per USD 1mm (short protection):**

| Name | CS01/USD 1mm | Equal-weight notional | Hedge CS01 |
|------|------------|----------------------|------------|
| 1 | 500 | 10mm | +5,000 |
| 2 | 400 | 10mm | +4,000 |
| 3 | 480 | 10mm | +4,800 |
| 4 | 520 | 10mm | +5,200 |
| 5 | 450 | 10mm | +4,500 |
| **Total** | 2,350 | 50mm | +23,500 |

**Hedge (long protection on constituents):** Flip sign → -23,500 USD /bp.

**Residual CS01:** $22{,}500 + (-23{,}500) = -1{,}000$ USD /bp.

**Scale to match:**
$$k = \frac{22{,}500}{23{,}500} = 0.9574$$

Hedge each name at $10 \times 0.9574 = 9.574$ mm notional.

**Incomplete replication (name 5 illiquid):**

Hedge only names 1–4 at scaled notional:
- Hedge CS01: $-(500 + 400 + 480 + 520) \times 9.574 = -18{,}191$ USD /bp
- Residual: $22{,}500 - 18{,}191 = +4{,}309$ USD /bp

**Interpretation:** Missing one name leaves material systematic exposure.

---

### Example 7: Basis Position P&L Decomposition

**Position:** Long protection on index (USD 50mm), short protection on intrinsic replication (sized to match CS01).

**CS01s:**
- Index (long protection): -22,500 USD /bp
- Constituents (short protection): +22,500 USD /bp
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
**Net: -USD 37,500** from dispersion.

**Key insight:** Basis-neutral to parallel moves, but not to dispersion or basis-specific moves.

---

### Example 8: Default Event — Cash Settlement

**Index:** $M = 5$, notional USD 50mm → per-name notional USD 10mm.
**Coupon:** $C = 100$ bp = 0.01.
**Default:** Name 3 defaults; auction final price $FP = 40$ (40% of par).
**Accrual fraction:** $\Delta_0 = 0.20$ years.

**Protection payment (to protection buyer):**

$$\text{Payout} = N_m \left(1 - \frac{FP}{100}\right) = 10{,}000{,}000 \times 0.60 = 6{,}000{,}000$$

**Accrued premium at default (paid by protection buyer):**

$$\text{Accrued} = C \times \Delta_0 \times N_m = 0.01 \times 0.20 \times 10{,}000{,}000 = 20{,}000$$

**Net default cashflow to protection buyer:**

$$6{,}000{,}000 - 20{,}000 = 5{,}980{,}000$$

**Post-default notional:** USD 40mm (premium payments reduced by 1/5).

---

### Example 9: Post-Default Hedge Rebalancing

**Before default:**
- Index position: Short protection, USD 50mm, CS01 = +22,500 USD /bp
- Hedge: Long protection on another index series, sized for CS01 = -22,500 USD /bp

**After default (notional reduces to USD 40mm):**
- Assume CS01 per USD 1mm unchanged at 450 USD /bp
- New index CS01 = $450 \times 40 = 18{,}000$ USD /bp
- Hedge CS01 (if same notional adjustment): also reduced proportionally

If both positions had the same defaulted constituent with equal weight, reduction is symmetric. If hedge is on a different index (e.g., different series with different constituents), the default affects them asymmetrically.

**Rebalancing requirement:**
- Recalculate CS01 for both legs
- Adjust hedge notional to restore CS01 match
- Consider whether default correlation assumptions should be updated

---

### Example 10: Recovery Sensitivity Sweep

Per-name notional `N_m = USD 10mm`.

| Final Price $FP$ | Recovery Rate | Payout |
|------------------|---------------|--------|
| 25 | 25% | USD 7.5mm |
| 40 | 40% | USD 6.0mm |
| 55 | 55% | USD 4.5mm |

**Range:** USD 3.0mm per name for 30-point FP range.

**Sensitivity:** $\partial \text{Payout}/\partial FP = -N_m/100 = -USD 100{,}000$ per price point.

---

### Example 11: Roll/Series Mechanics

**Old series:** Quoted spread 95 bp, coupon 100 bp, RPV01 ≈ 4.5 years.
**New series:** Quoted spread 100 bp (longer maturity + curve slope), same coupon.

**Upfront PV in dollars (conceptual):** $U_{\mathrm{USD}} = (S - C) \times RPV01 \times N$

- Old: $(0.0095 - 0.0100) \times 4.5 \times USD 100\text{mm} = -USD 225{,}000$
- New: $(0.0100 - 0.0100) \times 4.5 \times USD 100\text{mm} = USD 0$

**If you are long protection and roll by closing the old and entering the new**, the net cash exchanged is approximately the upfront difference: about USD 225,000 in this example.

---

### Example 12: Execution Cost Impact

**Basis-neutral position:** USD 50mm index vs USD 50mm constituent replication, net CS01 ≈ 0.

**Illustrative bid-ask half-spreads (assumptions):**
- Index: 0.25 bp
- Constituents (aggregate): 1.50 bp

**Execution cost approximation:**

$$\text{Cost} \approx \text{Half-spread} \times |CS01|$$

- Index: $0.25 \times 22{,}500 = USD 5{,}625$
- Constituents: $1.50 \times 22{,}500 = USD 33{,}750$
- **Total round-trip: ~USD 39,375**

**Interpretation:** Basis must move more than ~1.75 bp (= USD 39,375 / USD 22,500) just to cover execution costs. This is consistent with O'Kane's point about liquidity differences driving index basis.

---

### Example 13: Partial Replication with Residual Quantification

**Goal:** Hedge a USD 50mm index position using only the 5 largest-CS01 constituents.

**Index CS01:** +22,500 USD /bp (short protection).

**Top 5 constituents by CS01 (per USD 1mm):**

| Rank | Name | CS01/USD 1mm |
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

**Hedged CS01:** $-2{,}650 \times 8.49 = -22{,}500$ USD /bp → Net CS01 ≈ 0.

**But:** 120 names remain unhedged. If these have average CS01 of 180 USD /bp per USD 1mm:
- Missing CS01 per name: $180 \times 0.4$ mm (index weight) = 72 USD /bp
- 120 names × 72 USD /bp = 8,640 USD /bp unhedged idiosyncratic exposure

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
Notional:            USD 50,000,000
Index CS01:          +USD 22,500 /bp

HEDGE SUMMARY
───────────────────────────────────────────────────────────────
Hedge Type:          Constituent Replication (Top 20)
Hedge Notional:      USD 48,500,000 (aggregate)
Hedge CS01:          -USD 21,800 /bp
Net CS01:            +USD 700 /bp  (0.7% unhedged)

SCENARIO VALIDATION
───────────────────────────────────────────────────────────────
Scenario                    Index P&L    Hedge P&L    Net P&L
───────────────────────────────────────────────────────────────
Parallel +10bp              -USD 225,000    +$218,000    -$7,000
Parallel -10bp              +USD 225,000    -$218,000    +$7,000
Dispersion (avg=0)          USD 0           -$45,000     -$45,000
Single-name +50bp           -USD 8,000      +$4,500      -$3,500
Default (FP=40)             -USD 5,980,000  +$5,500,000  -$480,000
Basis +5bp                  -USD 112,500    $0           -$112,500

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
- Unhedged position variance: USD 400,000,000 (in squared dollars)
- Hedged position variance: USD 120,000,000

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
Index constituents are fixed for the life of a series; defaults remove names; a new series can change membership. Your analytics must track this.

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

4. **Payout bounds**: For $FP \in [0, 100]$, payout should be in $[0, N]$; sensitivity is $-N/100$ USD /point

5. **Basis attribution**: If basis is hedged, verify basis P&L near-zero; if unhedged, verify it matches basis move × basis CS01

6. **Beta stability check**: Compare short-window vs long-window beta estimates; if they differ materially, investigate (regime change, stale quotes, event risk)

7. **Hedge effectiveness monitoring**: Track HE over rolling windows; alert on decline

---

## Summary

### Executive Summary (10 Points)

1. CDS indices bundle many names into one liquid macro instrument; constituents are fixed except that defaults remove names without replacement.

2. Market quotes use a simplified index curve and contractual coupon $C(T)$, producing an upfront $U_I(t)$.

3. Intrinsic valuation builds index PV from constituent CDS curves; the gap between quoted and intrinsic is the **index basis**.

4. Basis drivers include restructuring clause differences, liquidity premia, and lead-lag effects where indices may move before single names.

5. Hedging starts with **CS01/Credit DV01 matching**, defined with sign conventions so short protection CS01 is positive.

6. **Top-down hedges** (single name → index) remove macro risk but keep idiosyncratic, basis, and event residuals.

7. **Bottom-up hedges** (index → constituents) aim to replicate intrinsic but keep basis, execution dispersion, and membership evolution risks.

8. **Systemic vs idiosyncratic delta**: the right hedge depends on whether you want to neutralize market-wide moves, name-specific moves, or a blend (real moves typically mix both components).

9. **Default events** change index cashflows and notional; premium leg reduces by $1/M$ increments, and event payout depends on auction final price.

10. **Always validate** with scenarios: parallel, dispersion, single-name gap, default event, roll shift, and execution cost stress.

### Cheat Sheet

**CS01 definition (this chapter):**
$$CS01 := V(S-1\text{bp}) - V(S) \approx -(V(S+1\text{bp}) - V(S))$$

**First-order P&L:**
$$\Delta V \approx -CS01 \cdot \Delta S \qquad (\Delta S\ \text{in bp})$$

**CS01 hedge notional:**
$$N_{\text{hedge}} = -\frac{CS01_{\text{position}}}{CS01_{\text{hedge}}(1)}$$

**Minimum-variance hedge ratio (cross-hedge):**
$$h^* = -\frac{\mathrm{Cov}(x,F)}{\mathrm{Var}(F)} \qquad \text{(sign depends on how you write the hedge)}$$

**Spread beta:**
$$\beta_{m,I} = \frac{\text{Cov}(\Delta S_m, \Delta S_I)}{\text{Var}(\Delta S_I)}$$

**Hedge effectiveness:**
$$\text{HE} = 1 - \frac{\text{Var}(\text{hedged P\\&L})}{\text{Var}(\text{unhedged P\\&L})}$$

**Index basis:**
$$\text{Basis} = S_I - S_I^{\text{intrinsic}}$$

**Default cash settlement payout:**
- Single-name (or per-default name share) notional $N_{\text{name}}$:
  $$\text{Payout} = N_{\text{name}} \left(1 - \frac{FP}{100}\right)$$
- Index trade notional $N_I$ with $M$ names (equal-weight per-name notional $N_I/M$):
  $$\text{Payout per default} = \frac{N_I}{M}\left(1 - \frac{FP}{100}\right)$$

**Systemic vs idiosyncratic (bump design):**
- Systemic delta: all spreads move together
- Idiosyncratic delta: one spread moves, others fixed
- Reality: spread moves mix both components

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

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Index basis | $S_I - S_I^{\text{intrinsic}}$ | Determines residual when hedging index with constituents |
| CS01 (Credit DV01) | PV change per 1 bp tightening of the stated spread factor (USD /bp) | Primary risk measure for non-default scenarios |
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

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the index basis? | $S_I - S_I^{\text{intrinsic}}$; drivers include restructuring differences, liquidity, lead/lag |
| 2 | Define Credit DV01/CS01 | PV change for **1 bp tightening**: $V(S-1\text{bp}) - V(S)$ (≈ $-[V(S+1\text{bp}) - V(S)]$) |
| 3 | Why do some CS01 definitions include a minus sign? | To report short-protection exposure as positive and align with $CS01\approx -\partial V/\partial S$ |
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
| 15 | How do you choose between systemic and idiosyncratic hedges? | Real moves blend both; a blended hedge is common and should be validated with scenarios |
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

## Mini Problem Set

**1.** Compute CS01 from PVs: $V(S-1) = 101{,}200$, $V(S) = 100{,}000$, $V(S+1) = 98{,}700$.

**2.** A long-protection position has CS01 $-6{,}000$ USD /bp. What is first-order P&L for +8bp widening?

**3.** Index CS01 per USD 1mm is 420 USD /bp. You need +12,600 USD /bp of hedge CS01. What index notional?

**4.** Single name widens +40bp, index +15bp. CS01s: name = -5,000 USD /bp, hedge = +5,000 USD /bp. Residual first-order P&L?

**5.** Two-factor hedge: exposures $(10{,}000, 6{,}000)$, instruments B = $(400, 100)$, C = $(100, 300)$ per USD 1mm. Solve for notionals $(x_B, x_C)$.

**6.** Intrinsic spread from 3 names: spreads = [80, 120, 200] bp, RPV01 weights = [4.0, 3.5, 2.5].

**7.** Cash-settled default on a single name (or on an index per-name share): $N_{\text{name}} = 5$ mm, $FP = 35$. Payout?

**8.** Accrued premium: $C = 500$ bp, $\Delta_0 = 0.10$ years, $N = 5$ mm. Accrued amount?

**9.** Index has $M = 100$, notional 200mm. After 3 defaults, remaining index notional?

**10.** Given 5 days of data: $\Delta S_{\text{name}}$ = [+4, -2, +6, +1, -3] bp, $\Delta S_{\text{index}}$ = [+2, -1, +4, 0, -2] bp. Calculate beta and $R^2$.

**11.** Explain why hedging off-the-run with on-the-run leaves residual risk.

**12.** Describe how PSA changes a bottom-up hedge.

**13.** Design a 5-scenario validation suite for a basis-neutral position.

**14.** Given the systemic vs idiosyncratic delta distinction, when would you weight toward systemic hedge sizing?

**15.** Show how dispersion produces P&L with zero average move.

**16.** How does recovery risk differ between single-name and index defaults?

**17.** If HE = 0.45 for a single-name → index hedge, is this acceptable? What action would you take?

**18.** A Brazilian corporate has estimated beta of 1.3 to CDX.EM (i.e., $\Delta S_{\text{name}} \approx \beta\\,\Delta S_{\text{index}}$). Position CS01 = -USD 10,000/bp. CDX.EM CS01/USD 1mm = +USD 400/bp. Calculate proxy hedge notional.

**19.** O'Kane reports example dispersion where HY constituent spreads have much higher standard deviation than IG (e.g., 340 bp vs 20 bp at 5Y in his data). What is the ratio of variance proxies $(\sigma^2)$?

**20.** You hedge a USD 100mm equity portfolio with long HY protection. Describe two scenarios where this hedge (a) works well and (b) fails.

### Solution Sketches (Selected)

**1.** Central-difference slope $\approx (98{,}700-101{,}200)/2=-1{,}250$ USD /bp, so $CS01\approx -\partial V/\partial S \approx 1{,}250$ USD /bp.

**6.** Intrinsic spread $\approx \frac{80(4.0)+120(3.5)+200(2.5)}{4.0+3.5+2.5}=\frac{1{,}240}{10}=124$ bp.

**11.** OTR vs off-the-run differs in (i) maturity and coupon (series basis), (ii) membership (index composition), and (iii) liquidity. Even if CS01 matched, those differences show up in dispersion/basis scenarios and in execution P&L.

**17.** HE = 0.45 means the hedge reduced variance by about 45% over the measurement window. Next actions: attribute residual drivers (idiosyncratic, basis, event), check beta stability, and decide whether to resize or add single-name overlays based on tracking-error limits.

**18.** Systematic sizing: $N_{\text{proxy}}=-\frac{\beta\\,CS01_{\text{position}}}{CS01_{\text{proxy}}(1)}=-\frac{1.3\times(-10{,}000)}{400}=32.5$ mm.

## References

- Dominic O’Kane, *Modelling Single-name and Multi-name Credit Derivatives* (CDS indices: mechanics, valuation, intrinsic spread, basis, PSA; systemic vs idiosyncratic delta; RPV01/CS01/VOD conventions).
- John C. Hull, *Risk Management and Financial Institutions* (credit indices overview; fixed coupon + upfront quoting intuition).
- John C. Hull, *Options, Futures, and Other Derivatives* (minimum-variance hedge ratio; regression hedging; hedge effectiveness; CDS payment date conventions and fixed-coupon quoting).
- Salih N. Neftci, *Principles of Financial Engineering* (risky annuity / RPV01 intuition in credit).
- David G. Luenberger, *Investment Science* (minimum-variance hedge in cov/var form).
