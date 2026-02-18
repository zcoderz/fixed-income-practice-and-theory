# Chapter 24: STIR Futures and Convexity Adjustments

---

## Introduction

A STIR futures contract quotes 94.50. Does that imply a forward rate of 5.50%? Not quite: $100-94.50$ is a **futures-implied rate**, and converting it to a **forward/FRA-equivalent rate** generally requires a **convexity adjustment**.

The adjustment exists because futures are **marked to market daily**. Gains and losses are paid/received throughout the life of the contract, not only at the end of the underlying accrual period. When rates are stochastic, these interim cashflows are reinvested or financed at rates that are correlated with the futures payoff. For interest-rate futures this correlation is mechanically strong, so the futures-implied rate is systematically different from the corresponding forward rate.

**Sign and magnitude (rule of thumb).** Under standard assumptions, the convexity adjustment $c$ is typically **positive**, so

$$\text{forward rate} \;=\; \text{futures-implied rate} \;-\; c.$$

In simple approximations, $c$ grows with rate volatility and (roughly) with the square of time-to-expiry. With annualized absolute volatility on the order of $100$ bp/year, $c$ is usually well below 1 bp for very short expiries but can reach $\sim 10$–15 bp around 5 years and $\sim 50$ bp around 10 years.

This matters whenever you use STIR futures to (i) build or sanity-check a forward curve, (ii) hedge forward-rate exposure, or (iii) interpret policy expectations from Fed Funds (or similar) futures.

Prerequisites: [Chapter 4 — Money-Market Building Blocks](chapter_04_money_market_building_blocks.md), [Chapter 17 — Curve Construction: Bootstrapping and Interpolation](chapter_17_curve_construction_bootstrapping_interpolation.md), [Chapter 23 — Treasury Futures — CTD, Conversion Factors, Delivery Options, and the Link to Repo](chapter_23_treasury_futures.md)  
Follow-on: [Chapter 25 — Interest Rate Swaps — Mechanics and Valuation](chapter_25_interest_rate_swaps_mechanics_valuation.md), [Chapter 26 — Swap PV01, DV01, and Hedging with Swaps](chapter_26_swap_pv01_dv01_hedging.md), [Chapter 28 — Basis Trades in Rates](chapter_28_basis_trades.md)

## Learning Objectives
- Convert a STIR futures quote $Q$ into a futures-implied rate, and then into a forward/FRA-equivalent rate using a convexity adjustment.
- Explain (in words and with a timeline) why daily settlement makes futures economically different from forwards.
- Estimate the order of magnitude of the convexity adjustment and perform quick sign/unit sanity checks.
- Compute contract DV01 with an explicit bump object, bump size (1 bp $=10^{-4}$), and sign convention; size a hedge (including “tailing”).
- Extract a one-meeting “implied post-meeting rate” and a simple hike/cut probability from a Fed Funds futures price.

In this chapter, we cover:

1. **What STIR futures are** — Eurodollar, SOFR, and Fed Funds contracts and their quoting conventions
2. **Why futures rates differ from forward rates** — daily settlement + settlement timing
3. **The convexity adjustment** — a usable approximation and order-of-magnitude calculations
4. **Tailed hedges** — sizing futures hedges for settlement-timing effects
5. **Fed Funds futures and policy extraction** — a one-meeting “implied rate” calculation
6. **TED spreads (brief)** — using STIR futures as a reference curve for short-dated credit/rates comparisons
7. **Packs and bundles (brief)** — common strip-trading terminology
8. **SOFR futures** — contract mechanics and what changes vs Eurodollars
9. **DV01 and hedging** — contract sensitivities and bucket hedging intuition

This chapter focuses on the futures-to-forward conversion tool (the convexity adjustment) and the downstream implications for curve building and hedging. Chapter 17 covers full bootstrapping mechanics; Chapter 23 covers bond futures where delivery options dominate the economics.

---

## 24.1 What Are STIR Futures?

### 24.1.1 Definition and Purpose

A STIR futures contract is an exchange-traded futures contract whose final settlement is linked to a short-term interest rate over a specified accrual period. They are widely used to hedge or take views on short-term rates because the contracts are standardized and (for the front expiries) often very liquid.

Unlike physical commodity futures, STIR futures typically settle in cash based on an interest rate observation (or an average/compounded rate over a window).

The key contracts in USD markets are:

**Eurodollar futures:** Historically referenced to 3-month USD LIBOR on a Eurodollar deposit (a USD deposit held outside the United States). Even when you trade SOFR futures, Eurodollar futures are still a useful conceptual template for “term rate over a 3M period” mechanics and for the classic convexity-adjustment intuition.

**3-month SOFR futures:** Based on compounded SOFR over the reference period. A critical distinction versus Eurodollars is settlement timing: SOFR futures settle based on realized overnight rates over the period (end-of-period settlement), while Eurodollars referenced a forward-looking term rate for the period (beginning-of-period style).

**Fed Funds futures:** Based on the arithmetic average of the effective federal funds rate over a calendar month. Because the payoff depends on a monthly average, these contracts are commonly used to infer market expectations for near-term FOMC outcomes.

### 24.1.2 The Quoting Convention

STIR futures use an inverted price quote that preserves the familiar bond-pricing intuition: higher rates correspond to lower prices. For Eurodollar and SOFR-style contracts:

$$\boxed{Q = 100 - R^{\text{fut}}}$$

where $Q$ is the quoted futures price and $R^{\text{fut}}$ is the implied futures rate expressed in percent.

**Example:** A quote of $Q = 95.25$ implies:

$$R^{\text{fut}} = 100 - 95.25 = 4.75\%$$

In decimal form: $r^{\text{fut}} = 0.0475$.

**Check (percent vs decimal):** The quote convention uses **percent** ($R^{\text{fut}}=4.75\%$), while most curve and convexity formulas use **decimals** ($r^{\text{fut}}=0.0475$). Convert once and be consistent: a 100× mistake in rate units becomes a 10,000× mistake in any $\sigma^2$ convexity term.

This convention means that a one-basis-point increase in the implied rate corresponds to a 0.01 decrease in price. The standardized relationship makes DV01 calculations straightforward (see Section 24.10).

For Fed Funds futures, the final settlement price is set to $100 - 100 \times r_{\mathrm{FF,avg}}$, where $r_{\mathrm{FF,avg}}$ is the monthly average effective fed funds rate (in percent).

### 24.1.3 Contract Value and Tick Size

The Eurodollar and 3-month SOFR futures contracts are designed so that a one-basis-point move in the implied rate has a fixed dollar value. The contract notional is USD1,000,000 and the underlying deposit period is approximately one quarter (0.25 years). A one-basis-point change in the rate changes the interest on this deposit by:

$$USD1{,}000{,}000 \times 0.25 \times 0.0001 = USD25$$

Thus:

$$\boxed{\text{DV01}_{\text{ED/SOFR}} = USD25 \text{ per contract per bp}}$$

This fixed tick value is remarkably convenient for hedging: if your exposure has a DV01 of USD50,000 per basis point, you need exactly 2,000 contracts to hedge it (ignoring the sign of the hedge).

For Fed Funds futures, the contract corresponds to a USD5,000,000 30-day deposit. Using ACT/360 day count:

$$USD5{,}000{,}000 \times \frac{0.0001 \times 30}{360} = USD41.67$$

Thus:

$$\boxed{\text{DV01}_{\text{FF}} = USD41.67 \text{ per contract per bp (30-day month)}}$$

For a 31-day month, the DV01 increases proportionally to USD43.06.

> **Sanity check:** The Fed Funds DV01 is larger than the ED/SOFR DV01 despite the shorter deposit period because the notional is five times larger (USD5M vs USD1M).

---

## 24.2 Forward Rates vs. Futures Rates: The Conceptual Distinction

### 24.2.1 The Forward Rate from Discount Factors

Before addressing why futures differ from forwards, we must establish the forward rate benchmark. A forward rate agreement (FRA) is a contract to lend or borrow at a pre-agreed rate over a future period. The "fair" forward rate $L(t; T_1, T_2)$ for the period $[T_1, T_2]$ is the rate that makes the FRA have zero value at inception.

From no-arbitrage, this rate is determined by discount factors:

$$\boxed{L(t; T_1, T_2) = \frac{P(t, T_1) - P(t, T_2)}{\tau \cdot P(t, T_2)}}$$

where:
- $P(t, T)$ is the discount factor (price of a zero-coupon bond) at time $t$ for maturity $T$
- $\tau = T_2 - T_1$ is the accrual period (year fraction)

**Derivation intuition:** Consider the replication strategy. Invest USD1 in a zero maturing at $T_1$ (costing $P(t, T_1)$ today). At $T_1$, roll this USD1 forward at the forward rate $L$ until $T_2$. The terminal value is $1 + \tau L$. No-arbitrage requires this equals what you would get from directly buying $(1 + \tau L)$ zeros maturing at $T_2$:

$$P(t, T_1) = (1 + \tau L) \cdot P(t, T_2)$$

Solving for $L$ gives the formula above.

**Worked Example:** Given $P(0, T_1) = 0.9780$ and $P(0, T_2) = 0.9655$ with $\tau = 0.25$:

$$L = \frac{0.9780 - 0.9655}{0.25 \times 0.9655} = \frac{0.0125}{0.2414} = 5.18\%$$

### 24.2.2 Why Futures Rates Can Differ from Forward Rates

A forward contract settles once at maturity. A futures contract is marked to market daily, with gains and losses exchanged through variation margin. This timing difference matters **only when rates are stochastic**: if rates were deterministic, forwards and futures would coincide.

The key economic point is that daily settlement creates intermediate cashflows that are **funded or reinvested at the prevailing short rate**. When that funding/reinvestment rate is correlated with the underlying (as it is for interest-rate futures), the expected value of the futures payoff is not the same as the expected value of the corresponding forward payoff.

One helpful way to remember this is:
- Futures are economically closer to “a stack of one-day forwards” (resetting to zero value each day).
- Forwards accumulate value until the end of the forward period.

The difference between the two shows up as a systematic wedge between the futures-implied rate and the forward rate used in no-arbitrage curve construction.

### 24.2.3 The Sign Convention

Define the convexity adjustment $c$ by:

$$\boxed{\text{Forward rate} = \text{Futures rate} - c}$$

For standard STIR futures, $c$ is typically **positive**, meaning the futures-implied rate exceeds the forward/FRA-equivalent rate.

> **Sign check:** Forward = Futures - (positive number), so Forward < Futures. This matches: futures rates are higher than forward rates.

---

## 24.3 The Convexity Adjustment: Quantifying the Difference

### 24.3.1 Deterministic Rate Limit

A useful sanity check: when interest rates are constant (deterministic), forward and futures prices coincide. In this limit:

$$c \to 0, \quad r^{\text{fut}} \to r^{\text{fwd}}$$

This makes perfect sense. With no rate volatility, there are no mark-to-market cash flows to reinvest or finance, so the daily settlement feature has no value. The futures contract becomes economically identical to a forward.

This gives us a verification test: in any convexity adjustment formula, setting volatility to zero should yield zero adjustment.

### 24.3.2 A Simple Approximation (Normal Model, No Mean Reversion)

Under a simple model—a normal (Gaussian) model with no mean reversion and continuous mark-to-market—the futures-forward wedge can be approximated as the sum of two components.

**Pure futures-forward effect:**

$$\frac{\sigma^2 t^2}{2}$$

This captures the mark-to-market reinvestment/financing asymmetry. The $t^2$ dependence reflects that longer-dated contracts have more time for daily settlements to accumulate, and the compounding of this effect over time is quadratic.

**Convexity effect:**

$$\frac{\sigma^2 \beta t}{2}$$

This captures the nonlinearity (convexity) in the relationship between rates and prices. The factor $\beta$ represents the length of the underlying deposit period (typically 0.25 for 3-month contracts).

**Combined formula:**

$$\boxed{r^{\text{fut}} - r^{\text{fwd}} \approx \frac{\sigma^2 t^2}{2} + \frac{\sigma^2 \beta t}{2}}$$

where:
- $\sigma$ = annual absolute volatility of the short rate (decimal; e.g., 100 bp/year $\approx 0.01$)
- $t$ = time to futures expiration (years)
- $\beta$ = underlying deposit period (years, typically 0.25 for 3-month)

> **Unit check:** $c$ is in **decimal rate** units. Multiply by 10,000 to convert to basis points.

**Key observations:**

1. **Quadratic in time:** The $t^2$ term dominates for longer-dated contracts, making the adjustment grow rapidly with maturity. A contract expiring in 10 years has approximately four times the adjustment of one expiring in 5 years.

2. **Quadratic in volatility:** Doubling volatility quadruples the adjustment. This sensitivity to volatility assumptions is a key source of model risk in curve construction.

3. **Zero when $\sigma = 0$:** Confirms the deterministic rate limit, as required.

4. **The $\beta t$ term is smaller:** For typical parameters, the pure futures-forward effect (the $t^2$ term) dominates. The convexity effect matters more for longer deposit periods.

### 24.3.3 Mean Reversion and Model Extensions

The simple approximation above assumes rates follow a random walk with no mean reversion. Many interest-rate models used in practice include mean reversion (e.g., Hull-White), which tends to damp long-horizon rate variance. All else equal, that usually reduces the convexity adjustment relative to a no-mean-reversion approximation—especially for longer expiries.

When precision matters, treat the approximation as an order-of-magnitude tool and compute adjustments with the same model/implementation that your curve-building and risk systems use.

### 24.3.4 Worked Example: EDZ6 (5-Year Contract)

**Example Title:** Converting a long-dated STIR futures quote to a forward rate (order of magnitude)

**Context**
- You want a forward/FRA-equivalent rate for the 3-month period associated with a STIR futures contract.
- The convexity adjustment is small at the very front end but can be material for longer expiries.

**Timeline (Make Dates Concrete)**
- Valuation date: 2001-11-30 (chosen to match the classic “EDZ6” illustration)
- Futures expiry / start of the underlying 3M period: 2006-12-20 (about $t=5.05$ years later)
- Accrual start/end: 2006-12-20 to 2007-03-20 (about 90 days; ACT/360 $\approx 0.25$)
- Payment date: 2007-03-20

**Inputs**
- Futures quote: $Q=95.00$ $\Rightarrow$ futures-implied rate $R^{\text{fut}}=5.00\%$ (so $r^{\text{fut}}=0.0500$)
- Volatility assumption: $\sigma=100$ bp/year $=0.01$ (absolute volatility, in decimal rate units)
- Deposit period: $\beta\approx 0.25$ years; time-to-expiry $t=5.05$ years

**Outputs (What You Produce)**
- Convexity adjustment $c$ (decimal and bp)
- Forward/FRA-equivalent rate $r^{\text{fwd}} = r^{\text{fut}} - c$

**Step-by-step**
1. Quote $\rightarrow$ futures-implied rate: $r^{\text{fut}}=(100-Q)/100=0.0500$.
2. Compute $\sigma^2 = 0.01^2 = 0.0001$.
3. Compute the adjustment:
   - Pure term: $\frac{\sigma^2 t^2}{2}=\frac{0.0001\times 5.05^2}{2}=0.001275$
   - Convexity term: $\frac{\sigma^2 \beta t}{2}=\frac{0.0001\times 0.25\times 5.05}{2}=0.000063$
   - Total: $c\approx 0.001338$, i.e. $13.4$ bp.
4. Convert futures $\rightarrow$ forward:
   $$r^{\text{fwd}} \approx 0.0500 - 0.001338 = 0.048662 \;\; (4.866\%).$$

**Cashflows (what the adjustment “means” in dollars)**
If you mistakenly use the futures-implied rate as a forward rate for a USD100,000,000 notional 3-month accrual, the rate error is about $13.4$ bp. The interest difference over $\tau\approx 0.25$ years is:

$$100{,}000{,}000 \times 0.25 \times 0.001338 \approx USD33{,}450.$$

**P&L / Risk Interpretation**
- In curve building, treating futures rates as forwards biases the forward curve upward and can compound across a strip.
- In hedging, “USD25 per bp” is only meaningful if your exposure DV01 is computed for the same forward bucket and settlement assumptions.

**Sanity Checks**
- Limit check: if $\sigma=0$, then $c=0$ and futures and forwards coincide.
- Units check: $c$ is in **decimal rate units**; multiply by 10,000 to get bp.
- Sign check: $c\gt 0\Rightarrow r^{\text{fwd}}\lt r^{\text{fut}}$ (forward below futures).

### 24.3.5 Adjustment Magnitude by Maturity

The following table shows how the convexity adjustment grows with contract maturity, assuming $\sigma = 100$ bp and $\beta = 0.25$:

| Time to Expiry | Pure F-F Effect | Convexity Effect | Total Adjustment |
|----------------|-----------------|------------------|------------------|
| 0.25 years | 0.03 bp | 0.03 bp | 0.06 bp |
| 1 year | 0.50 bp | 0.13 bp | 0.63 bp |
| 2 years | 2.00 bp | 0.25 bp | 2.25 bp |
| 3 years | 4.50 bp | 0.38 bp | 4.88 bp |
| 5 years | 12.50 bp | 0.63 bp | 13.1 bp |
| 10 years | 50.00 bp | 1.25 bp | 51.3 bp |

The pattern is clear: very short expiries have negligible adjustment, but beyond 2–3 years the adjustment becomes material for pricing and hedging. In this approximation, the dominant term grows like $t^2$.

> **Visual: The Convexity Gap**
>
> *   **X-Axis:** Time (0 to 10 years).
> *   **Y-Axis:** Rate (%).
> *   **Futures Rate**: A straight line at 5.00%.
> *   **Forward Rate**: A curved line that peels away downwards.
>     *   At 1Y: Tiny gap (0.5 bp).
>     *   At 5Y: Noticeable gap (13 bp).
>     *   At 10Y: Huge gap (50 bp).
>
> If you build your curve using the straight line (Futures) instead of the curved line (Forwards), long-dated valuations and hedges can be materially biased.

### 24.3.6 Sensitivity to Volatility

The adjustment scales with $\sigma^2$. For the 5-year contract:

| Volatility | Adjustment |
|------------|------------|
| 50 bp | 3.3 bp |
| 100 bp | 13.4 bp |
| 150 bp | 30.1 bp |

Doubling volatility from 100 bp to 200 bp would quadruple the adjustment to approximately 54 bp. This sensitivity to the volatility assumption is why sophisticated curve-building systems often calibrate the volatility from market data (e.g., from caps or swaptions) rather than using a single fixed number.

> **Practical note:** The choice of volatility for convexity adjustment can materially affect the resulting curve, especially for contracts beyond 3 years. Different dealers may use different volatility assumptions, creating basis between their curves.

---

## 24.4 Tailed Hedges: Adjusting for Financing

### 24.4.1 The Tailing Problem

When using futures to hedge a forward exposure, a subtle but important adjustment is required: the **tail**. The issue arises because futures gains and losses settle daily, while the forward exposure settles at maturity. Even if the DV01s match perfectly, the timing mismatch creates hedge slippage.

Consider a hedger with a USD100 million forward exposure at time $T$. The DV01 of this exposure, measured today, reflects discounting back to the present. A futures position with matching face amount would have the same DV01—but futures gains arrive today while the forward exposure settles at $T$. If the hedger reinvests futures gains until $T$, or finances futures losses until $T$, the effective exposure differs from the nominal DV01 match.

### 24.4.2 The Tailing Formula

To correct for the timing mismatch, the futures position is often **tailed** by a financing factor:

$$\boxed{N_{\text{fut}} = \frac{N_{\text{fwd}}}{1 + r \cdot d/360}}$$

where:
- $N_{\text{fwd}}$ = notional of the forward exposure
- $N_{\text{fut}}$ = notional of the futures hedge (to be tailed)
- $r$ = financing rate (annualized)
- $d$ = days until the forward settles

Equivalently, for the number of contracts:

$$\boxed{\text{Tailed contracts} = \frac{\text{Untailed contracts}}{1 + r \cdot d/360}}$$

The intuition: because futures gains can be invested (or losses must be financed) at rate $r$ for $d$ days, the futures position is effectively $(1 + rd/360)$ times as powerful as a forward position. To match exposures, reduce the futures position by this factor.

### 24.4.3 Worked Example: Tailing a 6-Month Hedge

**Problem:** You need to hedge a USD500 million 6-month forward rate exposure. The financing rate is 5%. How many Eurodollar futures contracts should you use?

**Step 1: Compute the untailed hedge**

Without tailing, the hedge would use:
$$\text{Untailed DV01} = USD500{,}000{,}000 \times 0.25 \times 0.0001 = USD12{,}500$$
$$\text{Untailed contracts} = \frac{USD12{,}500}{USD25} = 500 \text{ contracts}$$

**Step 2: Apply the tail**

With $r = 0.05$ and $d = 180$ days:
$$\text{Tailing factor} = 1 + 0.05 \times \frac{180}{360} = 1.025$$

$$\text{Tailed contracts} = \frac{500}{1.025} = 488 \text{ contracts}$$

**Step 3: Verify**

The 12-contract reduction (from 500 to 488) accounts for the fact that futures gains over the next 6 months will be reinvested, generating additional return that would otherwise over-hedge the forward exposure.

> **Desk Reality:** A “tailed hedge” is a present-value adjustment: it aligns the timing of daily variation-margin cashflows with the cashflow date of the exposure you are hedging.
> **Common break:** Ignoring tailing for long horizons can lead to systematic over-hedging/under-hedging (the gap grows with the financing horizon and rate level).
> **What to check:** compute both untailed and tailed contract counts and sanity-check that $\text{tailed} \lt \text{untailed}$ when $r\gt 0$.

### 24.4.4 When to Tail

| Hedge Horizon | Tail Impact (Heuristic) | Recommendation |
|--------------|-------------------------|----------------|
| < 1 month | usually negligible | ignore unless the position is very large |
| 1–3 months | small but noticeable in tight hedges | consider tailing for large positions |
| 3–12 months | often material | tailing recommended |
| > 1 year | can be large; re-tailing matters | tail, and re-tail as time passes |

For very long-dated hedges, the tail should be recalculated periodically as time passes and as rates change.

---

## 24.5 Fed Funds Futures and Policy Extraction

### 24.5.1 Contract Mechanics

Fed Funds futures settle based on the arithmetic average of the daily effective federal funds rate over a calendar month. The contract is designed to hedge a USD5,000,000 30-day deposit. Key features:

- **Notional:** USD5,000,000
- **Settlement:** Cash, based on average rate over the contract month
- **Quote:** $100 - r_{\mathrm{FF,avg}}$, where $r_{\mathrm{FF,avg}}$ is the monthly average in percent
- **DV01:** USD41.67 per bp (30-day month)

The averaging feature creates unique properties: the contract is sensitive to the *path* of rates during the month, not just the end-of-month level. This makes Fed Funds futures particularly useful for reading market expectations about Federal Reserve policy.

### 24.5.2 Extracting Policy Expectations

Market participants use Fed Funds futures to extract implied probabilities of Federal Reserve rate changes. The methodology exploits the fact that when an FOMC meeting occurs mid-month, the pre-meeting and post-meeting rates both contribute to the monthly average.

**The basic equation:** If a meeting occurs on day $d$ of a month with $D$ total days:

$$\bar{r} = r_{\text{pre}} \times \frac{d-1}{D} + r_{\text{post}} \times \frac{D-d+1}{D}$$

where:
- $r_{\text{pre}}$ = fed funds rate before the meeting
- $r_{\text{post}}$ = fed funds rate after the meeting
- $\bar{r}$ = implied average rate from the futures price

Solving for $r_{\text{post}}$:

$$\boxed{r_{\text{post}} = \frac{D \cdot \bar{r} - (d-1) \cdot r_{\text{pre}}}{D - d + 1}}$$

**Implied probability of a rate hike:**

If the Fed is expected to either keep rates unchanged or hike by 25bp:

$$P(\text{hike}) = \frac{r_{\text{post}} - r_{\text{pre}}}{0.25}$$

Here rates are in **percent** (so 25 bp = 0.25). If you work in decimals, replace 0.25 with 0.0025.

**Check (probability bounds):** Under this two-outcome setup, the implied probability must satisfy $0\le p\le 1$. If you compute $p\lt 0$ or $p\gt 1$, at least one assumption is inconsistent with the futures-implied average (common culprits: day weights around the meeting date, effective-vs-target spreads, or needing more than two outcomes).

### 24.5.3 Worked Example: FOMC Meeting Extraction

**Problem (toy but internally consistent):** The April Fed Funds futures contract trades at 95.20, implying $\bar{r}=4.80\%$ for the month. An FOMC meeting occurs on April 16 in a 30-day month. Assume the effective rate equals the target and is $r_{\text{pre}}=4.75\%$ before the meeting. If the only two outcomes are “no change” (4.75%) or “+25 bp hike” (5.00%), what hike probability is implied?

**Step 1: Translate the quote**
- Futures price $Q=95.20 \Rightarrow \bar{r}=100-Q=4.80\%$.

**Step 2: Solve for the implied post-meeting rate**

Assume 15 days at $r_{\text{pre}}$ and 15 days at $r_{\text{post}}$:
$$4.80=\frac{15\times 4.75 + 15\times r_{\text{post}}}{30}\quad\Rightarrow\quad r_{\text{post}}=4.85\%.$$

**Step 3: Map to a simple two-outcome probability**

If $r_{\text{post}}=4.75+0.25\times p$, then
$$p=\frac{4.85-4.75}{0.25}=0.40.$$

**Answer:** about **40%** probability of a 25 bp hike under this simplified setup.

> **Desk Reality:** The “Fed path” is often summarized from successive monthly-average futures prices and then translated into implied target probabilities.
> **Common break:** day-count weights (meeting timing, weekends/holidays), effective-vs-target spreads, multiple meetings in a month, and risk premia can all matter.
> **What to check:** write the month-weighted average explicitly and verify units (percent vs decimal) before interpreting probabilities.

### 24.5.4 Limitations of Policy Extraction

The method assumes:
- Only discrete policy moves (25bp increments)
- No inter-meeting rate changes
- Clean relationship between target and effective rate

In practice:
- The effective fed funds rate can deviate from the target due to supply/demand imbalances
- Reserve levels and regulatory changes affect the effective rate
- The method becomes less precise when meetings occur very early or late in the month

---

## 24.6 TED Spreads and Relative Value Analysis

### 24.6.1 What Is a TED Spread?

A TED (Treasury–Eurodollar) spread is a spread over a STIR-futures-implied reference curve that makes discounting match an observed bond price. In essence, it is a *cheap/rich* measure of a security versus an ED-futures-based short-rate curve.

Historically, "TED" referred to the spread between Treasury bills and Eurodollar deposits, serving as a measure of banking system credit risk. The modern TED spread framework generalizes this to any fixed-income security.

### 24.6.2 Computation

For a bond with price $P$ and cash flows $\{CF_i\}$ at times $\{t_i\}$:

$$P = \sum_i CF_i \times d(t_i)$$

where $d(t_i)$ is the discount factor computed from ED futures rates minus the TED spread $s$:

$$d(t_i) = \prod_{j \lt i} \frac{1}{1 + (r_j^{\text{fut}} - s) \times \tau_j}$$

The TED spread is the value of $s$ that solves this equation for the observed market price.

### 24.6.3 Interpretation and Trading

**Positive TED spread:** The bond trades cheap to ED futures; it requires a yield pick-up over LIBOR.
**Negative TED spread:** The bond trades rich to ED futures; it yields less than LIBOR-flat.

> **Desk Reality:** TED-style analysis is a quick way to express “cheap/rich versus a STIR curve” in a single number.
> **Common break:** Treating futures-implied rates as forwards ignores the convexity adjustment; the error is usually small for very short expiries but can grow with maturity and volatility.
> **What to check:** if the trade is sensitive to a longer part of the strip, build the reference curve from **convexity-adjusted** forwards (or at least sanity-check the futures-forward wedge you are ignoring).

Traders use TED spreads to:
> - **Compare bonds across credit quality:** How much spread does a corporate bond offer over LIBOR?
> - **Identify relative value:** Is a specific issue cheap or rich versus the curve?
> - **Hedge:** TED spread trades combine a bond position with ED futures strips

### 24.6.4 TED Spread Worked Example

**Problem (one-period toy):** A 6-month zero-coupon note pays 100 at maturity and trades at 97.75. Assume the relevant 6-month simple rate from your STIR-implied curve is $r^{\text{fut}}=5.00\%$ (ACT/360, $\tau=0.5$), and assume convexity adjustment is negligible at this horizon (or already applied). What TED spread $s$ makes discounting at $r^{\text{fut}}-s$ match the price?

**Step-by-step**
1. Price with a spread-subtracted simple rate:
   $$USD97.75 = \frac{100}{1+(r^{\text{fut}}-s)\tau}.$$
2. Solve for the implied spread-subtracted rate:
   $$r^{\text{fut}}-s = \frac{100/97.75 - 1}{0.5} \approx 4.60\%.$$
3. Therefore,
   $$s \approx 5.00\% - 4.60\% = 0.40\% = 40\text{ bp}.$$

**Interpretation:** A positive $s$ means the note trades **cheap** (higher yield) versus the STIR-implied reference curve.

---

## 24.7 Packs and Bundles: Color Codes and Strip Trades

### 24.7.1 Color Codes

Packs are often referenced by **yearly color codes** that group consecutive quarterly expiries:

| Color | Contracts | Typical Expiry Range |
|-------|-----------|---------------------|
| **Whites** | First 4 quarterly contracts | 0-1 year |
| **Reds** | Contracts 5-8 | 1-2 years |
| **Greens** | Contracts 9-12 | 2-3 years |
| **Blues** | Contracts 13-16 | 3-4 years |
| **Golds** | Contracts 17-20 | 4-5 years |

Beyond 5 years, some venues extend the color scheme further (e.g., purple/orange/pink, etc.).

### 24.7.2 Packs

A **pack** is a single trade that executes **four consecutive quarterly** STIR futures contracts (equal weight in each leg).

**Pack DV01:** Since a pack contains 4 contracts:
$$\text{Pack DV01} = 4 \times USD25 = USD100 \text{ per bp}$$

**Pack pricing (high level):** packs are typically quoted as the arithmetic average of the four constituent contract prices. Economically, a pack position has the same rate exposure as holding the four legs outright in equal size; the “pack” is just the execution wrapper.

### 24.7.3 Bundles

A **bundle** extends the same idea to a longer strip: it trades an **integer multiple of four** consecutive quarterly contracts in one order (e.g., 8, 12, 16, 20, … contracts).

| Bundle | Packs Included | Contracts | Years Covered |
|--------|----------------|-----------|---------------|
| 2-year bundle | Whites + Reds | 8 | 0-2 years |
| 3-year bundle | Whites + Reds + Greens | 12 | 0-3 years |
| 5-year bundle | Whites through Golds | 20 | 0-5 years |

**Bundle DV01:**
$$\text{5-year bundle DV01} = 20 \times USD25 = USD500 \text{ per bp}$$

**Weighted average maturity:** Because bundles are equally weighted by contract, the duration contribution is approximately the midpoint of the coverage period.

### 24.7.4 Practical Uses

- **Curve views:** packs/bundles let you express “blocks” of the forward curve with fewer tickets than trading each expiry.
- **Hedging:** they can be convenient wrappers for hedging forward-rate bucket exposure when the underlying bucket definition matches the strip.
- **Relative value:** pack-to-pack and bundle-to-bundle spreads are common curve-shape expressions (but remain sensitive to convexity and bump design).

---

## 24.8 SOFR Futures: The Post-LIBOR Landscape

### 24.8.1 Contract Specifications

SOFR (Secured Overnight Financing Rate) futures are widely used in USD STIR markets. Two main contract types exist:

**3-Month SOFR Futures:**
- **Reference rate:** Daily compounded SOFR over the reference quarter
- **Settlement:** At the *end* of the reference period (unlike ED, which settled at the start)
- **Notional:** USD1,000,000
- **DV01:** USD25 per bp per contract
- **Quote convention:** USD100 - R^{\text{SOFR}}$

**1-Month SOFR Futures:**
- **Reference rate:** Arithmetic average of daily SOFR over the contract month
- **Notional:** USD5,000,000 (like Fed Funds)
- **DV01:** USD41.67 per bp (30-day month)

### 24.8.2 SOFR vs. Eurodollar: Key Differences

| Feature | Eurodollar | 3M SOFR |
|---------|------------|---------|
| Underlying rate | 3M LIBOR (term rate) | Compounded SOFR (overnight) |
| Settlement timing | Beginning of reference period | End of reference period |
| Credit component | Unsecured term bank rate | Secured overnight repo rate |
| Rate dynamics | term unsecured | overnight secured; can show quarter-end dynamics |
| Convexity adjustment | Standard formula | Modified for settlement timing |

The settlement-timing difference is the key practical change: SOFR futures settle based on realized overnight rates over the quarter, while Eurodollars referenced a forward-looking term rate for the period.

### 24.8.3 Convexity Adjustment for SOFR

SOFR futures differ from Eurodollar futures in two important ways:
1. The underlying reference is a **compounded overnight rate** over a past-looking window (for 3M contracts).
2. Settlement is at the **end** of the reference period (after the relevant overnight rates have been observed), whereas Eurodollar futures referenced a forward-looking term rate and settled at the start of the accrual window.

As a result, the classic Eurodollar “convexity adjustment” approximation is not a drop-in formula for SOFR futures. The same daily-settlement logic still matters, but the exact mapping from a quoted SOFR futures price to a forward-equivalent curve input depends on the product’s rate definition and on modeling/implementation choices.

### 24.8.4 The SOFR-FF Basis

SOFR and Fed Funds differ conceptually—SOFR is a secured repo rate while Fed Funds is an unsecured interbank rate. The spread between them reflects:

- **Collateralization:** SOFR is generally lower due to secured nature
- **Reserve effects:** Fed Funds responds more directly to Fed policy
- **Quarter-end dynamics:** SOFR can spike at reporting dates due to balance sheet constraints

Traders watch the 1M SOFR vs. Fed Funds spread as an indicator of money market conditions. A widening spread may signal stress in the repo market or unusual demand for reserves.

---

## 24.9 Building Forward Curves from Futures Strips

### 24.9.1 The Naive Approach (and Its Flaw)

A common but flawed approach treats futures rates directly as forwards:

$$L^{\text{naive}}(0; T_i, T_{i+1}) = r_i^{\text{fut}}$$

This approach ignores the convexity adjustment, systematically biasing the forward curve upward. The distinction is usually negligible right at the front end, but it can become material as you extend the strip further out in maturity.

The practical consequence is that swaps or FRAs priced off the naive curve will be mispriced. If you pay fixed on a 5-year swap priced using unadjusted futures rates, you are paying a rate that is systematically too high by the accumulated convexity adjustment effect.

### 24.9.2 The Stub Rate

Before the first futures expiry, there is typically a period—the **stub**—that must be handled separately. The stub rate connects today's date to the first IMM date.

**Sources for the stub rate:**
- Cash LIBOR/SOFR fixing for the relevant tenor
- Overnight/Tom-next deposits
- Very short-dated FRAs

**The stitching problem:** The stub must be "stitched" to the futures strip to create a continuous curve. The stub discount factor $P(0, T_1^{\text{IMM}})$ is typically computed from:

$$P(0, T_1^{\text{IMM}}) = \frac{1}{1 + r_{\text{stub}} \times \tau_{\text{stub}}}$$

where $r_{\text{stub}}$ is the stub rate and $\tau_{\text{stub}}$ is the day count fraction to the first IMM date.

### 24.9.3 The Correct Approach

The proper procedure applies convexity adjustments before bootstrapping:

1. **Compute stub discount factor:** $P(0, T_1)$ from cash rates
2. **Convert quotes to futures rates:** $r_i^{\text{fut}} = (100 - Q_i)/100$
3. **Compute convexity adjustment:** $c_i = c(t_i, \sigma, \beta)$ using the approximation in Section 24.3.2 (or a model-based approach)
4. **Obtain forward rates:** $L_i = r_i^{\text{fut}} - c_i$
5. **Bootstrap discount factors:** Use the adjusted forward rates
6. **Stitch to longer instruments:** Beyond the liquid futures strip (typically 4-5 years), switch to swap rates

> **Desk Reality:** “Curve from futures” is a stitching workflow: stub $\rightarrow$ adjusted forwards $\rightarrow$ chained discount factors $\rightarrow$ switch to swaps beyond the liquid strip.
> **Common break:** mixing conventions (day count/compounding), skipping the stub, or feeding a risk system “forwards” that were built from unadjusted futures quotes.
> **What to check:** ensure discount factors are continuous at IMM dates and sanity-check adjusted forwards against nearby FRA/swap quotes where available.

### 24.9.4 Worked Example: Adjusting a Futures Strip

**Given:** Five quarterly STIR futures (Eurodollar-style):

| Contract | Expiry (years) | Quote | Futures Rate |
|----------|---------------|-------|--------------|
| 1 | 4.00 | 95.00 | 5.00% |
| 2 | 4.25 | 94.90 | 5.10% |
| 3 | 4.50 | 94.80 | 5.20% |
| 4 | 4.75 | 94.70 | 5.30% |
| 5 | 5.00 | 94.60 | 5.40% |

**Parameters:** $\sigma = 100$ bp = 0.01, $\beta = 0.25$

**Step 1: Compute convexity adjustments**

Using the approximation $c = \frac{\sigma^2 t^2}{2} + \frac{\sigma^2 \beta t}{2}$:

| Contract | $t$ | $c$ (bp) |
|----------|-----|----------|
| 1 | 4.00 | 8.50 |
| 2 | 4.25 | 9.56 |
| 3 | 4.50 | 10.69 |
| 4 | 4.75 | 11.88 |
| 5 | 5.00 | 13.13 |

**Step 2: Compute forward rates**

| Contract | Futures Rate | Adjustment | Forward Rate |
|----------|--------------|------------|--------------|
| 1 | 5.00% | 0.085% | 4.915% |
| 2 | 5.10% | 0.096% | 5.004% |
| 3 | 5.20% | 0.107% | 5.093% |
| 4 | 5.30% | 0.119% | 5.181% |
| 5 | 5.40% | 0.131% | 5.269% |

**Step 3: Impact on discount factors**

Starting from $P(0, 4.00) = 0.8200$, the first adjusted forward gives:

$$P(0, 4.25) = \frac{P(0, 4.00)}{1 + 0.25 \times 0.04915} = \frac{0.8200}{1.01229} = 0.8100$$

Compare to the naive approach using $L = 5.00\%$:

$$P^{\text{naive}}(0, 4.25) = \frac{0.8200}{1 + 0.25 \times 0.05} = \frac{0.8200}{1.0125} = 0.8099$$

The difference is small (0.01%) for one quarter but compounds across multiple periods. Over a full 10-year strip, the cumulative effect on long-dated discount factors can be several percent, which translates directly into swap pricing errors.

---

## 24.10 Risk Measures and Hedging

### 24.10.1 STIR Futures DV01

**Risk convention (bump object + sign).** In this book, DV01 is defined as

$$\boxed{DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})}$$

where $1\text{bp}=10^{-4}$ in **decimal** rate units, and “rates” means the **bump object** you choose (a forward bucket, a curve node, a futures-implied rate, etc.). Under this convention, DV01 is positive for positions that gain when rates fall.

For STIR futures, the natural bump object is the contract’s **implied futures rate** (the rate underlying $Q=100-R^{\text{fut}}$). A 1 bp *down* bump in the implied rate corresponds to a +0.01 move in the quoted price.

**Contract DV01 (units).** The standardized tick value makes hedge ratios straightforward. For a contract designed to hedge a money-market deposit with notional $N$ and accrual fraction $\tau$ (ACT/360),

$$\boxed{DV01_{\text{contract}} \approx N\cdot \tau \cdot 10^{-4}\;\;(USD \text{ per 1bp, for a long futures position})}$$

In USD STIR contracts this yields the familiar constants:

- Eurodollar/3M SOFR futures: USD25 per bp per contract
- Fed Funds/1M SOFR futures: USD41.67 per bp per contract (30-day month)

**Hedge ratio.** For an exposure with DV01 of $\text{DV01}_{\text{target}}$ (in USD per bp, computed under the same bump object definition), the hedge ratio is:

$$n_{\text{contracts}} = -\frac{DV01_{target}}{DV01_{fut}}$$

The negative sign says you take the opposite position: a long exposure (positive DV01) is hedged by **selling** futures so the combined DV01 is near zero.

> **Pitfall — “What is being bumped?”** A STIR futures contract’s DV01 is “USD per 1 bp of the contract’s implied rate.” Your exposure DV01 might instead be computed from a **curve** bump (zeros/par quotes) or from a different forward bucket.
> **Why it matters:** the hedge ratio can be wrong in size or even sign if the bump objects are not the same.
> **Quick check:** re-compute $\text{DV01}_{\text{target}}$ as $PV(\text{forward bucket }[T_1,T_2]\text{ down }1\text{bp})-PV(\text{base})$ and verify that dividing by USD25 (or USD41.67) gives a sensible contract count.

### 24.10.2 Bucket Hedging with STIR Futures

Because each quarterly STIR future maps naturally to a particular three-month forward-rate bucket, it is common to manage front-end curve risk in **three-month buckets** and hedge each bucket with the matching contract (when liquidity allows).

This **bucket hedging** approach offers several advantages:

1. **Precision:** Each forward rate is hedged with the matching contract, rather than relying on a duration-weighted parallel shift assumption.

2. **Curve risk management:** Non-parallel curve moves (steepeners, flatteners, butterflies) can be captured and hedged.

3. **Liquidity:** The front contracts are typically very liquid, allowing large positions to be established or unwound quickly.

Practical caveat: liquidity usually declines as you move out the strip, so hedge implementation may need to balance precision against liquidity and transaction costs.

### 24.10.3 Worked Example: Hedging a Forward Rate Exposure

**Problem:** You will pay a floating rate on USD500 million notional over the period [4.00, 4.25] years. Compute the hedge using STIR futures.

**Step 1: Compute exposure DV01**

The exposure is to the forward rate $L(0; 4.00, 4.25)$. If this forward rate increases by 1 bp, the floating payment increases by:

$$\Delta \text{Payment} = N \times \tau \times 0.0001 = 500{,}000{,}000 \times 0.25 \times 0.0001 = USD12{,}500$$

But this payment occurs at $T = 4.25$, so we must discount it:

$$\text{DV01} = N \times \tau \times P(0, 4.25) \times 0.0001$$

With $P(0, 4.25) = 0.8100$:

$$\text{DV01} = 500{,}000{,}000 \times 0.25 \times 0.8100 \times 0.0001 = USD10{,}125$$

**Step 2: Compute untailed hedge ratio**

$$\text{Untailed contracts} = -\frac{10{,}125}{25} = -405 \text{ contracts}$$

**Step 3: Apply tail (optional but recommended for 4+ year horizon)**

With financing rate 5% and 4.25 years (approximately 1,530 days):

$$\text{Tailing factor} \approx 1 + 0.05 \times \frac{1530}{360} \approx 1.21$$

$$\text{Tailed contracts} = \frac{-405}{1.21} \approx -335 \text{ contracts}$$

The tailed hedge is significantly smaller, reflecting the long horizon and material reinvestment effects.

> **Practical note:** The discount factor $P(0, 4.25)$ depends on whether you use naive or adjusted forwards. Using the naive forward would give $P = 0.8099$ and DV01 = USD10,124—virtually identical here, but the difference compounds for longer-dated exposures.

---

## 24.11 When Convexity Adjustments Matter

### 24.11.1 Front-End vs. Back-End

For contracts expiring within 1-2 years, the convexity adjustment is typically less than 1-2 bp. Most practitioners ignore it at the front end—the adjustment is smaller than bid-offer spreads and smaller than the uncertainty in other curve-building assumptions.

Beyond 2-3 years, the adjustment becomes material:
- At 5 years: ~13 bp
- At 10 years: ~50 bp

These magnitudes affect pricing and hedging meaningfully. A common rule of thumb is that once you are a couple of years out the strip, you should start treating “futures rates” and “forward rates” as different objects unless you have a reason not to.

### 24.11.2 Model Dependence

The simple formula $c \approx \sigma^2 t^2/2 + \sigma^2 \beta t/2$ assumes:
- Normal (Gaussian) rate dynamics
- No mean reversion
- Continuous mark-to-market

Real markets deviate from these assumptions in several ways:

**Mean reversion:** Many models used in practice include mean reversion, which tends to reduce the wedge versus a no-mean-reversion approximation, especially for longer expiries.

**Discrete settlement:** Settlement is daily, not continuous. The daily-vs-continuous difference is often second-order compared with volatility and model-choice uncertainty, but it can matter in a precise implementation.

**Volatility smiles:** The simple formula assumes constant volatility. Smile-aware approaches infer an effective adjustment from options across strikes rather than from a single $\sigma$.

Bottom line: for most practical purposes, the simple approximation provides reasonable order-of-magnitude estimates. When precision matters, use a model/implementation that is consistent with your curve construction and risk.

---

## 24.12 Practical Implementation Notes

### 24.12.1 Common Pitfalls

**Confusing futures rates with forward rates:** Always ask: “Am I looking at a futures-implied rate or a forward/FRA-equivalent rate?” The answer determines whether a convexity adjustment is needed.

**Forgetting daily settlement is the key mechanism:** The adjustment exists because futures are marked to market daily. Without this feature, futures and forwards would coincide. Any analysis that treats them identically implicitly assumes zero volatility.

**Mixing settlement conventions:** Eurodollar-style contracts reference a term rate over a 3M window, while 3-month SOFR futures reference realized overnight compounding and settle at the end of the window. This affects how you map a quote into a “curve input”.

**Ignoring tails for long-dated hedges:** For longer horizons, the tailing adjustment can be a few percent of the position. Failing to tail can systematically overstate the hedge.

**Assuming contract specs without verification:** Tick size, settlement calendar, and index definitions vary by contract. For specific contracts, consult exchange documentation.

### 24.12.2 Verification Tests

**Deterministic rate check:** Set $\sigma = 0$ in any adjustment formula. The result should be $c = 0$. If your formula gives a non-zero adjustment with zero volatility, something is wrong.

**Monotonicity:** The adjustment should increase with time to expiry and with volatility. Both relationships should be monotonic.

**Sign:** For standard interest rate futures, the adjustment should be positive (futures rate > forward rate). A negative adjustment would suggest an error.

**Order of magnitude:** For 5-year contracts with ~100 bp volatility, expect ~10-15 bp adjustment. For 10-year contracts, expect ~50 bp. If your numbers differ dramatically, check your formula and units.

### 24.12.3 Implementation Workflow

When building a curve from STIR futures:

1. Obtain futures quotes and convert to implied rates: $r^{\text{fut}} = (100 - Q)/100$
2. Compute stub discount factor from cash rates
3. Obtain or estimate rate volatility $\sigma$ (from caps, swaptions, or historical data)
4. Compute convexity adjustments for each maturity using the appropriate formula
5. Subtract adjustments to get forward rates: $r^{\text{fwd}} = r^{\text{fut}} - c$
6. Bootstrap discount factors from adjusted forwards
7. Beyond the liquid futures strip, switch to swap rates
8. Verify consistency with other market instruments (swaps, FRAs)

---

## Summary

1. STIR futures are quoted as $Q=100-R$ and are marked to market daily; daily settlement is the root cause of the futures-vs-forward wedge.
2. Forward rates used for curve building come from discount factors: $L(t;T_1,T_2)=\frac{P(t,T_1)-P(t,T_2)}{\tau\,P(t,T_2)}$ with $\tau$ defined by day count.
3. Define the convexity adjustment by $\text{forward}=\text{futures}-c$; typically $c\gt 0$, so forward rates are lower than futures-implied rates.
4. A usable approximation for the wedge is $c \approx \frac{\sigma^2 t^2}{2} + \frac{\sigma^2 \beta t}{2}$ (with $\beta\approx 0.25$ for a 3M deposit).
5. Order-of-magnitude: with $\sigma\sim 100$ bp/year, $c$ is sub-1bp at short expiries but can be $\sim 10$–15 bp around 5y and $\sim 50$ bp around 10y.
6. Curve workflow: pre-process futures quotes into forward/FRA-equivalent rates (convexity-adjusted) before bootstrapping discount factors.
7. Risk workflow: DV01 must specify the bump object, bump size (1 bp $=10^{-4}$), units, and sign; for USD 3M contracts, the per-contract DV01 is about USD25/bp.
8. Fed Funds futures prices embed a month-weighted average of pre- and post-meeting rates; a one-meeting implied rate (and simple hike/cut probability) comes from the day-weighting identity.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| STIR futures | Exchange-traded futures on short-term rates | Highly liquid hedging/speculation instruments |
| Quote convention | $Q = 100 - R^{\text{fut}}$ | Higher rate = lower price (like bonds) |
| Forward rate | Rate implied by discount factors for future lending | The "true" rate for FRA/swap pricing |
| Futures rate | Rate implied by futures quote | Systematically exceeds forward rate |
| Convexity adjustment | $c = r^{\text{fut}} - r^{\text{fwd}}$ | Corrects for daily settlement effect |
| Daily settlement | Mark-to-market with immediate cash flows | Creates futures-forward difference |
| Tailed hedge | Adjusted hedge accounting for financing | Prevents over-hedging on long horizons |
| Fed Funds policy extraction | Implied rate path from FF futures | Reads market Fed expectations |
| TED spread | Bond spread relative to ED futures | Credit/rates decomposition |
| Pack | 4 consecutive quarterly futures | Convenient for curve segment trades |
| Bundle | Multiple packs from front | Standardized strip positions |
| ED/SOFR futures DV01 | USD25 per bp per contract | Standardized for easy hedging |
| FF futures DV01 | USD41.67 per bp per contract (30-day) | Different notional, different DV01 |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $Q$ | STIR futures quoted price | price points; typically $Q=100-R^{\text{fut}}$ |
| $R^{\text{fut}}$ | Futures-implied rate | percent per year (convert to decimal by dividing by 100) |
| $r^{\text{fut}}$ | Futures-implied rate | decimal per year; $r^{\text{fut}}=R^{\text{fut}}/100$ |
| $L(t;T_1,T_2)$ | Simple forward rate for $[T_1,T_2]$ | per year; simple accrual $1+\tau L$; day count must be stated |
| $c$ | Convexity adjustment | decimal rate units; $\text{forward}=\text{futures}-c$; bp $=10{,}000\,c$ |
| $\sigma$ | Absolute volatility used in $c$ | per year in decimal rate units (e.g., 0.01 = 100 bp/year) |
| $t$ | Time to futures expiry | years |
| $\beta$ | Underlying deposit period | years; $\approx 0.25$ for 3M |
| $\tau$ | Accrual year-fraction | years (e.g., ACT/360) |
| $P(t,T)$ | Discount factor | unitless; $P(T,T)=1$ |
| $N$ | Notional | currency |
| $DV01$ | PV sensitivity scalar | currency per 1bp; $DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$ |
| $N_{\text{fut}}, N_{\text{fwd}}$ | Hedge notionals (futures vs forward) | currency notionals; “tailing” adjusts for timing |
| $\bar{r}_{\text{FF}}$ | Monthly average Fed Funds rate | percent per year in the policy-extraction formulas (state if using decimals) |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does "STIR" stand for? | Short-Term Interest Rate (futures) |
| 2 | What is the Eurodollar futures quote convention? | $Q = 100 - R^{\text{fut}}$ where $R^{\text{fut}}$ is in percent |
| 3 | If ED futures quote is 94.50, what is the implied rate? | 5.50% (since $100 - 94.50 = 5.50$) |
| 4 | What is the DV01 of one Eurodollar futures contract? | USD25 per basis point |
| 5 | How is the forward rate derived from discount factors? | $L = (P(T_1) - P(T_2)) / (\tau \cdot P(T_2))$ |
| 6 | When are futures and forward rates equal? | When interest rates are deterministic (constant) |
| 7 | How do we define the convexity adjustment $c$ in this chapter? | Forward rate = Futures rate $- c$ |
| 8 | Is the convexity adjustment typically positive or negative? | Positive |
| 9 | What does $c \gt 0$ imply about futures vs forward rates? | Futures rate exceeds forward rate |
| 10 | What is the key mechanism causing futures-forward differences? | Daily mark-to-market settlement |
| 11 | Why does daily settlement create a difference? | Gains reinvested at prevailing rates; losses financed at prevailing rates; correlation creates asymmetry |
| 12 | How does the adjustment scale with volatility? | Quadratically ($c \propto \sigma^2$) |
| 13 | How does the adjustment scale with time to expiry? | Approximately quadratically ($c \propto t^2$) |
| 14 | What is a simple approximation for the futures-forward wedge? | $r^{\text{fut}} - r^{\text{fwd}} \approx \sigma^2 t^2/2 + \sigma^2 \beta t/2$ |
| 15 | What is $\beta$ in the approximation? | Underlying deposit period (typically 0.25 for 3M) |
| 16 | What is the adjustment for a 5-year contract at 100 bp vol? | Approximately 13 bp |
| 17 | What is the adjustment for a 10-year contract at 100 bp vol? | Approximately 50 bp |
| 18 | What is the sanity check when $\sigma = 0$? | Adjustment should equal zero |
| 19 | What is the Fed Funds futures DV01 (30-day month)? | USD41.67 per basis point |
| 20 | How do you compute the hedge ratio? | # contracts = $-DV01_{target} / DV01_{fut}$ |
| 21 | What is the tailed hedge formula? | $N_{\text{fut}} = N_{\text{fwd}} / (1 + r \cdot d/360)$ |
| 22 | Why do we tail hedges? | To account for reinvestment/financing of daily settlement cash flows |
| 23 | What does a TED spread measure? | A bond's value relative to ED futures (spread over LIBOR) |
| 24 | What are "Whites" in pack terminology? | The first 4 quarterly futures contracts (0-1 year) |
| 25 | What is a "Red pack"? | Contracts 5-8 (1-2 year segment) |
| 26 | How many contracts in a 5-year bundle? | 20 contracts (5 packs × 4 contracts) |
| 27 | How does Eurodollar settlement timing differ from SOFR? | ED settles at beginning of period; SOFR at end |
| 28 | What is "bucket hedging"? | Hedging each maturity bucket with the matching futures |
| 29 | What happens if you ignore convexity in curve building? | Forward curve is biased upward |
| 30 | How do you extract Fed policy expectations from FF futures? | Use the averaging formula: $r_{\text{post}} = [D \cdot \bar{r} - (d-1) \cdot r_{\text{pre}}] / (D-d+1)$ |

---

## Mini Problem Set

**1.** A STIR futures contract quotes 96.40. What is the implied futures rate in percent and decimal?

**2.** Given $P(0, 1.0) = 0.9600$ and $P(0, 1.25) = 0.9480$ with $\tau = 0.25$, compute the simple forward rate.

**3.** Using the approximation in Section 24.3.2 with $\sigma = 0.01$, $\beta = 0.25$, $t = 3$, compute the convexity adjustment in bp.

**4.** With the same parameters as Q3, what happens to the adjustment if volatility doubles to $\sigma = 0.02$?

**5.** Your exposure has DV01 = $-USD250,000$/bp. How many Eurodollar futures hedge this? Should you buy or sell?

**6.** In a deterministic-rate world, what should the convexity adjustment be? Explain why.

**7.** Consider four futures contracts expiring at 1y, 2y, 3y, 4y with quotes 97.50, 97.00, 96.50, 96.00. Convert to forward rates using $\sigma = 0.01$, $\beta = 0.25$.

**8.** For a contract with $t = 2$ years and $\sigma = 150$ bp, compute the convexity adjustment. Compare to the 100 bp case.

**9.** Explain in one sentence why the futures rate exceeds the forward rate.

**10.** A desk builds curves by treating STIR futures rates as forwards without adjustment. What two diagnostics would reveal the convexity-related bias?

**11.** For Fed Funds futures, compute the DV01 for a 31-day month.

**12.** You need to hedge a USD200 million 9-month forward exposure. The financing rate is 4%. Compute both the untailed and tailed hedge in ED futures contracts.

**13.** The March Fed Funds futures contract trades at 95.25. An FOMC meeting occurs on March 19 (day 19 of 31). Current target is 4.50%. What is the implied post-meeting rate?

**14.** What is the DV01 of a "Green pack" (4 contracts in the 2-3 year segment)?

**15.** Qualitatively, why does convexity adjustment matter more for longer-dated contracts?

---

### Solution Sketches (Selected)

**1.** $R^{\text{fut}} = 100 - 96.40 = 3.60\%$; decimal: $r^{\text{fut}} = 0.0360$.

**2.** $L = (0.9600 - 0.9480)/(0.25 \times 0.9480) = 0.0120/0.2370 = 5.06\%$.

**3.** $\sigma^2 = 0.0001$. Pure F-F: $0.0001 \times 9/2 = 0.00045$. Convexity: $0.0001 \times 0.25 \times 3/2 = 0.0000375$. Sum = 0.0004875 = 4.88 bp.

**4.** $\sigma^2$ quadruples when $\sigma$ doubles, so adjustment quadruples: $4 \times 4.88 = 19.5$ bp.

**5.** Number of contracts: $-(-250{,}000)/25 = +10{,}000$. Buy 10,000 contracts (negative DV01 means exposure is short rates, so hedge by going long futures).

**6.** Zero. With no volatility, there are no mark-to-market cash flows, so the settlement timing has no value and futures equal forwards.

**7.** Convexity adjustments: 1y = 0.63 bp, 2y = 2.25 bp, 3y = 4.88 bp, 4y = 8.50 bp. Forward rates: 1y = 2.494%, 2y = 2.978%, 3y = 3.451%, 4y = 3.915%.

**8.** At $\sigma = 0.015$: $\sigma^2 = 0.000225$. Pure F-F: $0.000225 \times 4/2 = 0.00045$. Convexity: $0.000225 \times 0.25 \times 2/2 = 0.0000563$. Total = 5.06 bp. Compare to 100 bp case: $0.0001 \times 4/2 + 0.0001 \times 0.25 \times 2/2 = 2.25$ bp. The 150 bp volatility gives 2.25× the adjustment.

**9.** Because daily settlement creates intermediate cashflows that must be funded/reinvested at the short rate, and for rate futures this funding rate is correlated with the underlying rate; this makes the futures-implied rate differ systematically from the forward rate.

**10.** Two quick diagnostics:
- Compare the futures-implied forwards (with and without convexity adjustment) against nearby FRA/swap quotes; a persistent “too-high forwards” bias is a red flag.
- Recompute long-dated PVs/forwards using the same strip with and without convexity adjustment and check whether mispricings grow with maturity roughly like $t^2$ (as the simple approximation predicts).

**11.** For a 31-day month: $DV01 = 5{,}000{,}000\times 0.0001\times 31/360 \approx USD43.06$ per bp per contract.

---

## References

- Hull, *Options, Futures, and Other Derivatives*, “Convexity Adjustments”; “SOFR Futures”.
- Tuckman & Serrat, *Fixed Income Securities: Tools for Today’s Markets*, “Eurodollar and Fed Funds Futures”; “Tails: A Closer Look at Hedging with Futures”.
- Andersen & Piterbarg, *Interest Rate Modeling* (Vol. 1), “Bonds and Forward Rates” (forward measure vs risk-neutral measure; futures rate representation).
- Neftci, *Principles of Financial Engineering*, “Comparing FRAs and Eurodollar Futures”; “Convexity differences”.
- Hirsa, *Computational Methods in Finance*, “Curve construction with LIBOR, Eurodollar futures, and convexity adjustment pre-processing” (Chapter 9).
- CME Group, “SOFR Futures and Options Globex Strategy Guide” (2024), “Packs and bundles” (color codes; pack/bundle definitions; BPV scaling).
