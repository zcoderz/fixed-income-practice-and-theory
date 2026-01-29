# Chapter 24: STIR Futures and Convexity Adjustments

---

## Introduction

A Eurodollar futures contract quotes 94.50. Does that imply a forward rate of 5.50%? Not quite.

This seemingly straightforward question conceals one of the subtler issues in fixed income: the difference between futures rates and forward rates. For front-month contracts, the difference is negligible—a fraction of a basis point that traders safely ignore. But for a contract expiring in five years, the gap can exceed 10 basis points, and for ten-year expiries, it approaches 50 basis points. Ignore this difference when building a curve from futures strips, and you will systematically bias your forward rates, misprice FRAs and swaps, and generate mysterious P&L slippage.

The culprit is **daily settlement**—the mark-to-market feature that distinguishes futures from forwards. Hull provides the definitive explanation: "A Eurodollar futures contract is settled daily, whereas a FRA is not settled until the end of the forward period. When rates rise, the trader with a futures contract gains, and the trader is in a position where the interest earned on the gain is relatively low." The converse also holds: when rates fall, losses must be financed, but at the new lower rates—an apparent benefit. Yet these two effects do not cancel. The asymmetry creates a systematic wedge between the futures rate and the forward rate.

Andersen and Piterbarg formalize this insight precisely: under continuous mark-to-market, "the futures rate $F(\cdot, T, T+\tau)$ is a $Q$-martingale," where $Q$ is the risk-neutral measure. In contrast, the forward rate is a martingale under the *forward measure* associated with the payment date. The difference between expectations in these two measures is exactly the convexity adjustment—not a model artifact, but a fundamental consequence of how settlement timing interacts with stochastic discounting.

This chapter develops the intuition, mathematics, and practical tools for handling STIR (Short-Term Interest Rate) futures:

1. **What STIR futures are** — Eurodollar, SOFR, and Fed Funds contracts and their quoting conventions
2. **Why futures rates differ from forward rates** — the daily settlement mechanism and correlation effects
3. **The convexity adjustment** — Hull's definition, Tuckman's approximation formula, and order-of-magnitude calculations
4. **Tailed hedges** — why and how to adjust hedge ratios for financing costs
5. **Fed Funds futures and policy extraction** — reading the Fed's path from futures prices
6. **TED spreads and relative value** — using futures for credit-versus-rates analysis
7. **Pack and bundle conventions** — how traders quote strip positions
8. **SOFR futures** — the post-LIBOR landscape and its implications
9. **DV01 and hedging** — contract sensitivities and bucket hedging applications

This chapter focuses on the *conceptual* distinction between futures and forwards and the adjustment needed to convert between them. Chapter 17 covers the mechanics of bootstrapping a full curve; here we develop the specific tool—the convexity adjustment—that makes futures-to-forward conversion possible. Chapter 23 discusses Treasury bond and note futures, which involve different mechanics (delivery options, conversion factors) rather than rate-based settlement.

---

## 24.1 What Are STIR Futures?

### 24.1.1 Definition and Purpose

A STIR futures contract is an exchange-traded futures contract whose final settlement is linked to a short-term interest rate over a specified accrual period. Tuckman describes their purpose precisely: "Futures contracts on short-term rates are extremely useful for hedging against risks arising from changes in short-term rates and for speculating on the direction of these rates. This usefulness stems from the great liquidity of many interest rate futures contracts relative to that of the underlying assets."

Unlike physical commodity futures where delivery involves transferring the underlying asset, STIR futures settle in cash based on an interest rate observation. This cash settlement mechanism makes them ideal instruments for expressing views on short-term rates or hedging floating-rate exposures.

The key contracts in USD markets are:

**Eurodollar futures:** Historically the dominant contract, referenced to 3-month USD LIBOR. Hull explains the underlying: "The underlying security is a Eurodollar deposit, a deposit denominated in U.S. dollars at a bank outside the United States." While LIBOR is being phased out, understanding Eurodollar futures remains important for legacy positions as well as for understanding the conceptual template that SOFR futures inherit.

**3-month SOFR futures:** The successor to Eurodollar futures, based on compounded SOFR over the reference period. A critical distinction: Hull notes that SOFR futures settle at the *end* of the 3-month period based on realized overnight rates, whereas Eurodollar futures settled at the *beginning* based on a quoted term rate. This timing difference affects the futures-forward relationship.

**Fed Funds futures:** Based on the arithmetic average of the effective federal funds rate over a calendar month. Tuckman describes the contract as "designed as a hedge to a 30-day deposit in fed funds." The averaging feature means these contracts are commonly used to infer market expectations for Federal Reserve policy actions.

### 24.1.2 The Quoting Convention

STIR futures use an inverted price quote that preserves the familiar bond-pricing intuition: higher rates correspond to lower prices. For Eurodollar and SOFR-style contracts:

$$\boxed{Q = 100 - R^{\text{fut}}}$$

where $Q$ is the quoted futures price and $R^{\text{fut}}$ is the implied futures rate expressed in percent.

**Example:** A quote of $Q = 95.25$ implies:

$$R^{\text{fut}} = 100 - 95.25 = 4.75\%$$

In decimal form: $r^{\text{fut}} = 0.0475$.

This convention means that a one-basis-point increase in the implied rate corresponds to a 0.01 decrease in price. The standardized relationship makes DV01 calculations straightforward (see Section 24.9).

For Fed Funds futures, Tuckman describes an analogous "100 minus average rate" rule: the settlement price equals $100 - 100 \times \bar{r}_{\text{FF}}$, where $\bar{r}_{\text{FF}}$ is the monthly average effective fed funds rate.

### 24.1.3 Contract Value and Tick Size

The Eurodollar and 3-month SOFR futures contracts are designed so that a one-basis-point move in the implied rate has a fixed dollar value. Hull derives this from first principles. The contract notional is \$1,000,000 and the underlying deposit period is approximately one quarter (0.25 years). A one-basis-point change in the rate changes the interest on this deposit by:

$$\$1{,}000{,}000 \times 0.25 \times 0.0001 = \$25$$

Thus:

$$\boxed{\text{DV01}_{\text{ED/SOFR}} = \$25 \text{ per contract per bp}}$$

This fixed tick value is remarkably convenient for hedging: if your exposure has a DV01 of \$50,000 per basis point, you need exactly 2,000 contracts to hedge it (ignoring the sign of the hedge).

For Fed Funds futures, the contract corresponds to a \$5,000,000 30-day deposit. Using ACT/360 day count:

$$\$5{,}000{,}000 \times \frac{0.0001 \times 30}{360} = \$41.67$$

Thus:

$$\boxed{\text{DV01}_{\text{FF}} = \$41.67 \text{ per contract per bp (30-day month)}}$$

For a 31-day month, the DV01 increases proportionally to \$43.06.

> **Sanity check:** The Fed Funds DV01 is larger than the ED/SOFR DV01 despite the shorter deposit period because the notional is five times larger (\$5M vs \$1M).

---

## 24.2 Forward Rates vs. Futures Rates: The Conceptual Distinction

### 24.2.1 The Forward Rate from Discount Factors

Before addressing why futures differ from forwards, we must establish the forward rate benchmark. A forward rate agreement (FRA) is a contract to lend or borrow at a pre-agreed rate over a future period. The "fair" forward rate $L(t; T_1, T_2)$ for the period $[T_1, T_2]$ is the rate that makes the FRA have zero value at inception.

From no-arbitrage, this rate is determined by discount factors. Andersen and Piterbarg derive the standard relationship:

$$\boxed{L(t; T_1, T_2) = \frac{P(t, T_1) - P(t, T_2)}{\tau \cdot P(t, T_2)}}$$

where:
- $P(t, T)$ is the discount factor (price of a zero-coupon bond) at time $t$ for maturity $T$
- $\tau = T_2 - T_1$ is the accrual period (year fraction)

**Derivation intuition:** Consider the replication strategy. Invest \$1 in a zero maturing at $T_1$ (costing $P(t, T_1)$ today). At $T_1$, roll this \$1 forward at the forward rate $L$ until $T_2$. The terminal value is $1 + \tau L$. No-arbitrage requires this equals what you would get from directly buying $(1 + \tau L)$ zeros maturing at $T_2$:

$$P(t, T_1) = (1 + \tau L) \cdot P(t, T_2)$$

Solving for $L$ gives the formula above.

**Worked Example:** Given $P(0, T_1) = 0.9780$ and $P(0, T_2) = 0.9655$ with $\tau = 0.25$:

$$L = \frac{0.9780 - 0.9655}{0.25 \times 0.9655} = \frac{0.0125}{0.2414} = 5.18\%$$

### 24.2.2 Why Futures Rates Can Differ from Forward Rates

A forward contract settles once at maturity. A futures contract is marked to market daily, with gains and losses settled through margin calls. This difference in settlement timing creates a wedge between futures and forward prices.

Hull provides the key theoretical result: "When interest rates are constant, forward and futures prices are the same." The difference arises only when rates are stochastic and correlated with the underlying.

For most underlyings—commodities, equities, foreign exchange—this correlation is weak enough to ignore. But interest rate futures are special: the underlying *is* an interest rate, creating strong mechanical correlation. Tuckman explains the mechanism with characteristic clarity:

> "The pure futures-forward effect arises because mark-to-market gains are invested at low rates while mark-to-market losses are financed at high rates."

Let us unpack this intuition carefully:

1. **Rates rise → futures position gains:** When rates increase, the futures price falls (recall: $Q = 100 - R$), but you are short the futures through your long position's daily settlement. Actually, for a long position in "rates" terms (you want rates to rise), you would be short the futures contract. Let's be more precise: if you expect rates to rise and go *short* the futures (sell high, buy back low), your short position gains when rates rise. The settlement delivers cash immediately. But now rates are higher, so your reinvestment rate is higher—a benefit that would not exist with a forward contract, which settles only at maturity.

2. **Rates fall → futures position loses:** Your short position requires you to make margin payments. But with rates now lower, your financing cost for these payments is lower—another benefit.

Both scenarios appear favorable for the futures holder relative to the forward holder. So where is the asymmetry?

The asymmetry comes from the *magnitude* and *correlation* of these effects. When rates are high, both the price change per basis point and the investment rate are large; when rates are low, both are small. The expected value of "gain × reinvestment rate" differs from "loss × financing rate" because the underlying rate and the discount rate are the same object. For interest rate futures, this correlation is strong and negative: a high rate means a low futures price (since $Q = 100 - R$), and vice versa.

> **Analogy: The Compound Interest Advantage**
>
> Why is a Futures contract consistently better (higher rate) than a Forward?
>
> 1.  **Scenario A (Futures):** You win $100 today. You put it in the bank and earn daily interest for the rest of the year.
> 2.  **Scenario B (Forward):** You win $100 today, but the casino holds it until the end of the year. You get zero interest.
>
> **The Kicker:** In rate markets, you tend to "win" (short position gains) when rates go UP. So in Scenario A, you get your cash exactly when interest rates are high! This "positive correlation between winning and high rates" makes Futures strictly more valuable. The market charges you for this privilege by forcing you to pay a slightly higher rate to enter the contract.
>
> That extra charge is the **Convexity Adjustment**.

### 24.2.3 The Sign Convention

Hull defines the convexity adjustment $c$ by:

$$\boxed{\text{Forward rate} = \text{Futures rate} - c}$$

and explicitly states that "$c$ is positive" under typical conditions. Tuckman confirms this directional relationship: the futures rate exceeds the forward rate.

The intuition: the daily settlement advantage to the futures holder means that a futures position is "too valuable" compared to a forward. To restore equilibrium, the market sets the futures rate higher than the forward rate—the futures buyer accepts a worse rate to compensate for the settlement timing benefit.

> **Sign check:** Forward = Futures - (positive number), so Forward < Futures. This matches: futures rates are higher than forward rates.

---

## 24.3 The Convexity Adjustment: Quantifying the Difference

### 24.3.1 Deterministic Rate Limit

Hull's theoretical result provides an important sanity check: when interest rates are constant (deterministic), forward and futures prices coincide. In this limit:

$$c \to 0, \quad r^{\text{fut}} \to r^{\text{fwd}}$$

This makes perfect sense. With no rate volatility, there are no mark-to-market cash flows to reinvest or finance, so the daily settlement feature has no value. The futures contract becomes economically identical to a forward.

This gives us a verification test: in any convexity adjustment formula, setting volatility to zero should yield zero adjustment.

### 24.3.2 Tuckman's Approximation Formula

Under a simple model—a normal (Gaussian) model with no mean reversion and continuous mark-to-market—Tuckman derives an explicit approximation for the futures-forward difference. The total effect has two components.

**Pure futures-forward effect (equation 17.29):**

$$\frac{\sigma^2 t^2}{2}$$

This captures the mark-to-market reinvestment/financing asymmetry. The $t^2$ dependence reflects that longer-dated contracts have more time for daily settlements to accumulate, and the compounding of this effect over time is quadratic.

**Convexity effect (equation 17.30):**

$$\frac{\sigma^2 \beta t}{2}$$

This captures the nonlinearity (convexity) in the relationship between rates and prices. The factor $\beta$ represents the length of the underlying deposit period (typically 0.25 for 3-month contracts).

**Combined formula:**

$$\boxed{r^{\text{fut}} - r^{\text{fwd}} \approx \frac{\sigma^2 t^2}{2} + \frac{\sigma^2 \beta t}{2}}$$

where:
- $\sigma$ = annual absolute volatility of the short rate (decimal, e.g., 0.01 for 100 bp/year)
- $t$ = time to futures expiration (years)
- $\beta$ = underlying deposit period (years, typically 0.25 for 3-month)

**Key observations:**

1. **Quadratic in time:** The $t^2$ term dominates for longer-dated contracts, making the adjustment grow rapidly with maturity. A contract expiring in 10 years has approximately four times the adjustment of one expiring in 5 years.

2. **Quadratic in volatility:** Doubling volatility quadruples the adjustment. This sensitivity to volatility assumptions is a key source of model risk in curve construction.

3. **Zero when $\sigma = 0$:** Confirms the deterministic rate limit, as required.

4. **The $\beta t$ term is smaller:** For typical parameters, the pure futures-forward effect (the $t^2$ term) dominates. The convexity effect matters more for longer deposit periods.

### 24.3.3 Mean Reversion and Model Extensions

The simple Tuckman formula assumes rates follow a random walk with no mean reversion. Real interest rates, however, tend to exhibit mean-reverting behavior—rates far above or below historical norms tend to drift back toward equilibrium levels.

**Impact of mean reversion:** When rates mean-revert, distant forward rates are "pulled back" toward the long-run mean. This reduces the variance of future rate levels relative to a pure random walk, which in turn reduces the convexity adjustment. Andersen and Piterbarg show that in a Hull-White model with mean reversion parameter $\kappa$:

$$c_{\text{mean-rev}} < c_{\text{no-rev}}$$

The effect is more pronounced for longer-dated contracts. For a 10-year contract, ignoring mean reversion might overstate the convexity adjustment by 10-20%, depending on the calibrated mean-reversion speed.

> **Practitioner Note:** Most sophisticated curve-building systems incorporate mean reversion effects. The "rule of thumb" adjustments in this chapter assume no mean reversion and thus provide conservative (high) estimates. When precision matters, use a calibrated Hull-White or LMM model.

### 24.3.4 Worked Example: EDZ6 (5-Year Contract)

Tuckman provides a concrete example that we can reproduce. Consider a Eurodollar contract expiring in approximately 5.05 years with $\sigma = 100$ bp = 0.01 and $\beta = 0.25$:

**Step 1: Compute $\sigma^2$**

$$\sigma^2 = 0.01^2 = 0.0001$$

**Step 2: Compute each term**

Pure futures-forward effect:
$$\frac{\sigma^2 t^2}{2} = \frac{0.0001 \times 5.05^2}{2} = \frac{0.0001 \times 25.5025}{2} = 0.001275$$

Convexity effect:
$$\frac{\sigma^2 \beta t}{2} = \frac{0.0001 \times 0.25 \times 5.05}{2} = \frac{0.0001263}{2} = 0.0000631$$

**Step 3: Total adjustment**

$$r^{\text{fut}} - r^{\text{fwd}} = 0.001275 + 0.0000631 = 0.001338$$

In basis points: $0.001338 \times 10{,}000 = 13.4$ bp.

Tuckman confirms this result in equation (17.32): "In this case the total futures-forward effect in basis points is... 13.4."

**Interpretation:** For this 5-year contract, the futures rate exceeds the forward rate by approximately 13.4 basis points. When building a curve, you would compute:

$$r^{\text{fwd}} = r^{\text{fut}} - 13.4 \text{ bp}$$

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

The pattern is clear: front-month contracts have negligible adjustment, but beyond 2-3 years the adjustment becomes material for pricing and hedging. Tuckman's Figure 17.1 graphs this relationship, confirming that "the effect increases with the square of time to contract expiration."

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
> If you build your curve using the straight line (Futures) instead of the curved line (Forwards), your 10-year swap valuation will be wrong by massive amounts.

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

Tuckman introduces the concept: "The hedge that sets the face amounts of the hedging security to the present value of the exposure being hedged is called a **tailed hedge**."

Consider a hedger with a $100 million forward exposure at time $T$. The DV01 of this exposure, measured today, reflects discounting back to the present. A futures position with matching face amount would have the same DV01—but futures gains arrive today while the forward exposure settles at $T$. If the hedger reinvests futures gains until $T$, or finances futures losses until $T$, the effective exposure differs from the nominal DV01 match.

### 24.4.2 The Tailing Formula

To correctly hedge, the futures position must be "tailed" by the financing factor. Tuckman derives the adjustment:

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

**Problem:** You need to hedge a $500 million 6-month forward rate exposure. The financing rate is 5%. How many Eurodollar futures contracts should you use?

**Step 1: Compute the untailed hedge**

Without tailing, the hedge would use:
$$\text{Untailed DV01} = \$500{,}000{,}000 \times 0.25 \times 0.0001 = \$12{,}500$$
$$\text{Untailed contracts} = \frac{\$12{,}500}{\$25} = 500 \text{ contracts}$$

**Step 2: Apply the tail**

With $r = 0.05$ and $d = 180$ days:
$$\text{Tailing factor} = 1 + 0.05 \times \frac{180}{360} = 1.025$$

$$\text{Tailed contracts} = \frac{500}{1.025} = 488 \text{ contracts}$$

**Step 3: Verify**

The 12-contract reduction (from 500 to 488) accounts for the fact that futures gains over the next 6 months will be reinvested, generating additional return that would otherwise over-hedge the forward exposure.

> **Desk Reality: "Tailed Hedges Are Only an Approximation"**
>
> Tuckman explicitly notes that "tailed hedges are only an approximation to a theoretically correct hedge." The formula assumes:
> - Constant financing rate
> - No correlation between rate changes and financing rates
> - Linear P&L (ignores convexity of the hedge itself)
>
> In practice, tailing matters most for:
> - Longer-dated hedges (more time for reinvestment/financing effects)
> - Higher rate environments (larger financing factor)
> - Large positions where 2-3% hedge slippage is material

### 24.4.4 When to Tail

| Hedge Horizon | Tail Impact | Recommendation |
|---------------|-------------|----------------|
| < 1 month | < 0.5% | Ignore in most cases |
| 1-3 months | 0.5-1.5% | Consider for large positions |
| 3-6 months | 1.5-3% | Tail recommended |
| 6-12 months | 3-6% | Always tail |
| > 1 year | > 6% | Essential; may need periodic re-tailing |

For very long-dated hedges, the tail should be recalculated periodically as time passes and as rates change.

---

## 24.5 Fed Funds Futures and Policy Extraction

### 24.5.1 Contract Mechanics

Fed Funds futures settle based on the arithmetic average of the daily effective federal funds rate over a calendar month. Tuckman describes the contract as "designed as a hedge to a 30-day deposit in fed funds." Key features:

- **Notional:** \$5,000,000
- **Settlement:** Cash, based on average rate over the contract month
- **Quote:** $100 - \bar{r}_{\text{FF}}$ where $\bar{r}_{\text{FF}}$ is the monthly average in percent
- **DV01:** \$41.67 per bp (30-day month)

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

### 24.5.3 Worked Example: FOMC Meeting Extraction

**Problem:** The January Fed Funds futures contract is trading at 95.00 (implying 5.00% average rate). An FOMC meeting is scheduled for January 28 (day 28 of 31). The current fed funds target is 4.75%. What is the implied probability of a 25bp hike?

**Step 1: Identify parameters**
- $\bar{r} = 5.00\%$ (from futures: $100 - 95.00$)
- $r_{\text{pre}} = 4.75\%$ (current target)
- $d = 28$ (meeting day)
- $D = 31$ (days in January)

**Step 2: Solve for implied post-meeting rate**

$$r_{\text{post}} = \frac{31 \times 5.00 - 27 \times 4.75}{31 - 28 + 1} = \frac{155.00 - 128.25}{4} = \frac{26.75}{4} = 6.6875\%$$

**Step 3: Compute implied probability**

If the only possibilities are unchanged (4.75%) or +25bp (5.00%):

Wait—the implied post-meeting rate of 6.69% exceeds both scenarios. Let me recalculate:

$$r_{\text{post}} = \frac{31 \times 0.0500 - 27 \times 0.0475}{4} = \frac{1.550 - 1.2825}{4} = \frac{0.2675}{4} = 0.066875 = 6.69\%$$

This high implied rate suggests the market is pricing in not just a 25bp hike but potentially multiple hikes or a larger move. For a more realistic example:

**Revised Example:** Futures at 95.15 (4.85% average), meeting on day 28:

$$r_{\text{post}} = \frac{31 \times 0.0485 - 27 \times 0.0475}{4} = \frac{1.5035 - 1.2825}{4} = \frac{0.2210}{4} = 5.525\%$$

For a 25bp hike (to 5.00%):
$$P(\text{hike}) = \frac{5.525\% - 4.75\%}{0.25\%} = \frac{0.775\%}{0.25\%} = 3.1$$

This is greater than 100%, implying the market prices more than a single 25bp hike—perhaps a 50bp hike or multiple meetings in the month.

> **Desk Reality: Reading the "Fed Path"**
>
> Traders refer to the sequence of implied target rates from consecutive Fed Funds futures as the "Fed path" or "dot plot proxy." Key uses:
>
> - **Pre-meeting positioning:** Long futures = expect Fed to be more dovish than priced
> - **Curve trades:** Steepeners/flatteners on the Fed path express views on the pace of policy
> - **Event risk:** Options on Fed Funds futures trade actively around FOMC meetings
>
> The CME publishes "FedWatch" tools that automate these calculations and account for multiple meeting scenarios.

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

Tuckman introduces the TED (Treasury-Eurodollar) spread as "the spread such that discounting cash flows at Eurodollar futures rates minus that spread produces the security's market price." In essence, the TED spread measures a bond's cheapness or richness relative to ED futures rates.

Historically, "TED" referred to the spread between Treasury bills and Eurodollar deposits, serving as a measure of banking system credit risk. The modern TED spread framework generalizes this to any fixed-income security.

### 24.6.2 Computation

For a bond with price $P$ and cash flows $\{CF_i\}$ at times $\{t_i\}$:

$$P = \sum_i CF_i \times d(t_i)$$

where $d(t_i)$ is the discount factor computed from ED futures rates minus the TED spread $s$:

$$d(t_i) = \prod_{j < i} \frac{1}{1 + (r_j^{\text{fut}} - s) \times \tau_j}$$

The TED spread is the value of $s$ that solves this equation for the observed market price.

### 24.6.3 Interpretation and Trading

**Positive TED spread:** The bond trades cheap to ED futures; it requires a yield pick-up over LIBOR.
**Negative TED spread:** The bond trades rich to ED futures; it yields less than LIBOR-flat.

> **Desk Reality: TED Spread Analysis**
>
> Tuckman notes an "obvious theoretical flaw" in TED analysis: it discounts using futures rates rather than forward rates, ignoring the convexity adjustment. However, "the magnitude of the difference between forward and futures rates is relatively small for futures expiring shortly."
>
> Traders use TED spreads to:
> - **Compare bonds across credit quality:** How much spread does a corporate bond offer over LIBOR?
> - **Identify relative value:** Is a specific issue cheap or rich versus the curve?
> - **Hedge:** TED spread trades combine a bond position with ED futures strips

### 24.6.4 TED Spread Worked Example

**Problem:** A 2-year Treasury note (6% coupon, semiannual) trades at 100.50. The first four quarterly ED futures rates are 5.00%, 5.10%, 5.20%, 5.30%. Estimate the TED spread.

**Step 1: Build discount factors from futures**

For simplicity, assume quarterly compounding and ignore day count details:

| Period | Futures Rate | DF (no spread) |
|--------|--------------|----------------|
| 0.25 | 5.00% | $1/(1 + 0.0500 \times 0.25) = 0.9877$ |
| 0.50 | 5.10% | $0.9877/(1 + 0.0510 \times 0.25) = 0.9752$ |
| 0.75 | 5.20% | $0.9752/(1 + 0.0520 \times 0.25) = 0.9626$ |
| 1.00 | 5.30% | $0.9626/(1 + 0.0530 \times 0.25) = 0.9499$ |

(Continue similarly to 2 years)

**Step 2: Price the bond at various spreads**

Compute the bond price using discount factors adjusted by spread $s$, and iterate to find the spread that matches the market price of 100.50.

**Step 3: Interpret**

If the TED spread is, say, -15bp, the Treasury trades 15bp rich to LIBOR—reflecting its superior credit quality and liquidity.

---

## 24.7 Pack and Bundle Trading Conventions

### 24.7.1 Color Codes

STIR futures traders organize contracts by expiry using a color-coding system that dates back to Eurodollar pit trading. Understanding these conventions is essential for navigating the market:

| Color | Contracts | Typical Expiry Range |
|-------|-----------|---------------------|
| **Whites** | First 4 quarterly contracts | 0-1 year |
| **Reds** | Contracts 5-8 | 1-2 years |
| **Greens** | Contracts 9-12 | 2-3 years |
| **Blues** | Contracts 13-16 | 3-4 years |
| **Golds** | Contracts 17-20 | 4-5 years |

### 24.7.2 Packs

A **pack** is a simultaneous purchase or sale of four consecutive quarterly contracts. Each contract in the pack has equal weight.

**Example:** "Buy the Red pack at -3" means buying contracts 5, 6, 7, and 8 at a price that averages 3 ticks below the reference (often the close or a composite mid).

**Pack DV01:** Since a pack contains 4 contracts:
$$\text{Pack DV01} = 4 \times \$25 = \$100 \text{ per bp}$$

**Pack pricing:** Pack prices are quoted as the average price of the four contracts, typically expressed relative to a benchmark. The pack captures a one-year segment of the curve.

> **Desk Reality: Trading Packs**
>
> Packs are convenient for:
> - **Duration extension/reduction:** Adding a Red pack extends duration by roughly 1.5 years on average
> - **Curve trades:** Buy Whites, sell Reds = 2s1s flattener
> - **Rolling positions:** Instead of rolling individual contracts, roll entire packs

### 24.7.3 Bundles

A **bundle** is a simultaneous purchase or sale of consecutive packs, starting from the front. Bundles always begin with the White pack.

| Bundle | Packs Included | Contracts | Years Covered |
|--------|----------------|-----------|---------------|
| 2-year bundle | Whites + Reds | 8 | 0-2 years |
| 3-year bundle | Whites + Reds + Greens | 12 | 0-3 years |
| 5-year bundle | Whites through Golds | 20 | 0-5 years |

**Bundle DV01:**
$$\text{5-year bundle DV01} = 20 \times \$25 = \$500 \text{ per bp}$$

**Weighted average maturity:** Because bundles are equally weighted by contract, the duration contribution is approximately the midpoint of the coverage period.

### 24.7.4 Practical Uses

**Curve steepeners/flatteners:** Buy a 2-year bundle, sell a 5-year bundle = 5s2s flattener in rate terms.

**Swap hedging:** A 5-year swap can be approximately hedged with a 5-year bundle of futures, though convexity adjustments apply for the back contracts.

**Relative value:** Compare pack spreads (e.g., Reds - Greens) to historical ranges to identify curve richness/cheapness.

---

## 24.8 SOFR Futures: The Post-LIBOR Landscape

### 24.8.1 Contract Specifications

With the cessation of LIBOR, SOFR (Secured Overnight Financing Rate) futures have become the primary USD STIR market. Two main contract types exist:

**3-Month SOFR Futures:**
- **Reference rate:** Daily compounded SOFR over the reference quarter
- **Settlement:** At the *end* of the reference period (unlike ED, which settled at the start)
- **Notional:** \$1,000,000
- **DV01:** \$25 per bp per contract
- **Quote convention:** $100 - R^{\text{SOFR}}$

**1-Month SOFR Futures:**
- **Reference rate:** Arithmetic average of daily SOFR over the contract month
- **Notional:** \$5,000,000 (like Fed Funds)
- **DV01:** \$41.67 per bp (30-day month)

### 24.8.2 SOFR vs. Eurodollar: Key Differences

| Feature | Eurodollar | 3M SOFR |
|---------|------------|---------|
| Underlying rate | 3M LIBOR (term rate) | Compounded SOFR (overnight) |
| Settlement timing | Beginning of reference period | End of reference period |
| Credit risk | Bank credit embedded | Risk-free (secured) |
| Rate volatility | Generally lower | Can spike (quarter-end) |
| Convexity adjustment | Standard formula | Modified for settlement timing |

Hull emphasizes the settlement timing difference: SOFR futures settle based on *realized* overnight rates over the quarter, creating a fundamentally different contract than ED futures which settled based on a forward-looking term rate quote.

### 24.8.3 Convexity Adjustment for SOFR

The convexity adjustment for SOFR futures differs from the classic ED formula due to the end-of-period settlement. Hull notes this affects the futures-forward relationship, though the direction and magnitude are similar.

The key insight: because SOFR futures settle at the *end* of the reference period (time $T_2$) rather than the *beginning* (time $T_1$), the "investment period" for mark-to-market gains is shorter. This generally results in a *smaller* convexity adjustment for SOFR versus ED futures with the same expiry.

> **Practitioner Note:** For practical curve building, most systems treat 3M SOFR convexity adjustments similarly to ED adjustments, with a modest reduction (perhaps 10-20% smaller) for the settlement timing difference. The exact adjustment depends on model assumptions about overnight rate dynamics.

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

This approach ignores the convexity adjustment, systematically biasing the forward curve upward. Hull explicitly warns against this: "When the futures contracts we have just considered last longer than about two years, it does become important to distinguish between futures and forward."

The practical consequence is that swaps or FRAs priced off the naive curve will be mispriced. If you pay fixed on a 5-year swap priced using unadjusted futures rates, you are paying a rate that is systematically too high by the accumulated convexity adjustment effect.

### 24.9.2 The Stub Rate

Before the first futures expiry, there is typically a period—the **stub**—that must be handled separately. Tuckman discusses this as part of curve construction: the stub rate connects today's date to the first IMM date.

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
3. **Compute convexity adjustment:** $c_i = c(t_i, \sigma, \beta)$ using Tuckman's formula or a model-based approach
4. **Obtain forward rates:** $L_i = r_i^{\text{fut}} - c_i$
5. **Bootstrap discount factors:** Use the adjusted forward rates
6. **Stitch to longer instruments:** Beyond the liquid futures strip (typically 4-5 years), switch to swap rates

Andersen and Piterbarg note the importance of this step: "A pre-processing step is normally employed to convert the futures rate quote to a forward rate (FRA) quote" before curve construction.

> **Technique: The Stitching Problem**
>
> Building a curve from futures isn't just about convexity; it's about "Stitching."
>
> 1.  **The Stub**: The period from Today until the 1st Futures expiry. (Use Cash/LIBOR).
> 2.  **The Colors**: Futures contracts are color-coded by year.
>     *   **Whites**: First year (Front 4 contracts).
>     *   **Reds**: Second year.
>     *   **Greens**: Third year.
>     *   **Blues**: Fourth year.
> 3.  **The Stitch**: You calculate the discount factor for the 1st expiry. Then multiply by the DF of the 2nd futures period to get the next date.
>     *   $P(T_2) = P(T_1) \times \frac{1}{1 + L_{1,2} \tau}$
>     *   Repeat until you run out of liquid futures (usually 4-5 years/Blue pack). Then switch to Swaps.

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

Using Tuckman's formula $c = \frac{\sigma^2 t^2}{2} + \frac{\sigma^2 \beta t}{2}$:

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

The standardized tick value makes hedge ratio calculations straightforward. Recall:

- Eurodollar/3M SOFR futures: \$25 per bp per contract
- Fed Funds/1M SOFR futures: \$41.67 per bp per contract (30-day month)

For an exposure with DV01 of $\text{DV01}_{\text{target}}$ dollars per bp, the hedge ratio is:

$$\boxed{\text{\# contracts} = -\frac{\text{DV01}_{\text{target}}}{\text{DV01}_{\text{fut}}}}$$

The negative sign indicates that a long exposure (positive DV01, meaning you lose when rates rise) is hedged by selling futures (which gains when rates rise, since price = 100 - rate).

### 24.10.2 Bucket Hedging with STIR Futures

Tuckman describes the common practice of granular hedging: "As each Eurodollar futures contract is related to a particular three-month forward rate... it is common to divide the first 10 years of exposure into three-month buckets. In this way any bucket exposure may, if desired, be hedged directly with Eurodollar futures."

This **bucket hedging** approach offers several advantages:

1. **Precision:** Each forward rate is hedged with the matching contract, rather than relying on a duration-weighted parallel shift assumption.

2. **Curve risk management:** Non-parallel curve moves (steepeners, flatteners, butterflies) can be captured and hedged.

3. **Liquidity:** STIR futures are among the most liquid instruments in fixed income markets, allowing large positions to be established or unwound quickly.

Tuckman notes that "the longer-maturity Eurodollar futures are not nearly so liquid as the earlier ones," so practical hedge implementation may need to balance precision against liquidity.

### 24.10.3 Worked Example: Hedging a Forward Rate Exposure

**Problem:** You will pay a floating rate on \$500 million notional over the period [4.00, 4.25] years. Compute the hedge using STIR futures.

**Step 1: Compute exposure DV01**

The exposure is to the forward rate $L(0; 4.00, 4.25)$. If this forward rate increases by 1 bp, the floating payment increases by:

$$\Delta \text{Payment} = N \times \tau \times 0.0001 = 500{,}000{,}000 \times 0.25 \times 0.0001 = \$12{,}500$$

But this payment occurs at $T = 4.25$, so we must discount it:

$$\text{DV01} = N \times \tau \times P(0, 4.25) \times 0.0001$$

With $P(0, 4.25) = 0.8100$:

$$\text{DV01} = 500{,}000{,}000 \times 0.25 \times 0.8100 \times 0.0001 = \$10{,}125$$

**Step 2: Compute untailed hedge ratio**

$$\text{Untailed contracts} = -\frac{10{,}125}{25} = -405 \text{ contracts}$$

**Step 3: Apply tail (optional but recommended for 4+ year horizon)**

With financing rate 5% and 4.25 years (approximately 1,530 days):

$$\text{Tailing factor} \approx 1 + 0.05 \times \frac{1530}{360} \approx 1.21$$

$$\text{Tailed contracts} = \frac{405}{1.21} \approx 335 \text{ contracts}$$

The tailed hedge is significantly smaller, reflecting the long horizon and material reinvestment effects.

> **Practical note:** The discount factor $P(0, 4.25)$ depends on whether you use naive or adjusted forwards. Using the naive forward would give $P = 0.8099$ and DV01 = \$10,124—virtually identical here, but the difference compounds for longer-dated exposures.

---

## 24.11 When Convexity Adjustments Matter

### 24.11.1 Front-End vs. Back-End

For contracts expiring within 1-2 years, the convexity adjustment is typically less than 1-2 bp. Most practitioners ignore it at the front end—the adjustment is smaller than bid-offer spreads and smaller than the uncertainty in other curve-building assumptions.

Beyond 2-3 years, the adjustment becomes material:
- At 5 years: ~13 bp
- At 10 years: ~50 bp

These magnitudes affect pricing and hedging meaningfully. Hull notes: "When the futures contracts we have just considered last longer than about two years, it does become important to distinguish between futures and forward."

### 24.11.2 Model Dependence

The simple formula $c \approx \sigma^2 t^2/2 + \sigma^2 \beta t/2$ assumes:
- Normal (Gaussian) rate dynamics
- No mean reversion
- Continuous mark-to-market

Real markets deviate from these assumptions in several ways:

**Mean reversion:** Real interest rates exhibit mean-reverting behavior. Andersen and Piterbarg discuss how mean reversion affects convexity adjustments for CMS rates; analogous effects apply to ED futures, though the impact is smaller for shorter-dated contracts.

**Discrete settlement:** Settlement is daily, not continuous. However, Andersen and Piterbarg note that "the difference between daily and continuous settlement is quite small."

**Volatility smiles:** The simple formula assumes constant volatility. More sophisticated approaches account for the volatility smile. Andersen and Piterbarg derive formulas showing that "the ED convexity adjustment depends on prices of ED futures options at all strikes, i.e., on the volatility smile."

Hull cautions that "determining $c$ involves an assumption about the underlying interest rate model." For most practical purposes, the simple approximation provides reasonable order-of-magnitude estimates. When precision matters—curve construction at major dealers, for instance—more sophisticated model-based adjustments are employed.

---

## 24.12 Practical Implementation Notes

### 24.12.1 Common Pitfalls

**Confusing futures rates with forward rates:** Hull explicitly distinguishes them, and the distinction matters. Always ask: "Am I looking at a futures rate or a forward rate?" The answer determines whether a convexity adjustment is needed.

**Forgetting daily settlement is the key mechanism:** The adjustment exists because futures are marked to market daily. Without this feature, futures and forwards would coincide. Any analysis that treats them identically implicitly assumes zero volatility.

**Mixing settlement conventions:** Eurodollar futures settle at the *beginning* of the 3-month period (on the IMM date); 3-month SOFR futures settle at the *end* based on realized overnight compounding. This affects the relationship between the futures rate and the forward rate. Hull notes this distinction explicitly.

**Ignoring tails for long-dated hedges:** For hedges beyond 6 months, the tailing adjustment can be 2-5% of the position. Failing to tail overstates the hedge.

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

STIR futures are exchange-traded contracts on short-term interest rates, quoted so that higher rates correspond to lower prices ($Q = 100 - R$). The key insight of this chapter is that **futures rates systematically exceed forward rates** due to the daily settlement feature.

The convexity adjustment $c$ bridges the gap:

$$\text{Forward} = \text{Futures} - c$$

where $c$ is positive and grows with volatility and time to expiry.

Tuckman's approximation provides the magnitude:

$$c \approx \frac{\sigma^2 t^2}{2} + \frac{\sigma^2 \beta t}{2}$$

For front-month contracts, the adjustment is negligible (~0.1 bp). For contracts beyond 2-3 years, it becomes material for pricing and hedging (~5-15 bp at 5 years, ~50 bp at 10 years). When extracting forward curves from futures strips, always apply convexity adjustments before bootstrapping—otherwise the curve will be biased upward, affecting all downstream valuations and hedges.

**Additional key points from this chapter:**

- **Tailed hedges** adjust futures positions for reinvestment/financing effects; the tail factor is $(1 + rd/360)^{-1}$
- **Fed Funds futures** enable extraction of market-implied Fed policy expectations through the meeting-date averaging formula
- **TED spreads** measure bond value relative to ED futures, useful for credit/rates decomposition
- **Pack and bundle conventions** (Whites/Reds/Greens/Blues) organize the futures strip for trading
- **SOFR futures** have replaced Eurodollars, with end-of-period settlement affecting the convexity relationship
- **Stub rates** must be stitched to the futures strip for complete curve construction

---

## Key Concepts Summary

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
| ED/SOFR futures DV01 | \$25 per bp per contract | Standardized for easy hedging |
| FF futures DV01 | \$41.67 per bp per contract (30-day) | Different notional, different DV01 |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $Q$ | Quoted STIR futures price |
| $R^{\text{fut}}$ | Futures-implied rate (percent) |
| $r^{\text{fut}}$ | Futures-implied rate (decimal) |
| $L(t; T_1, T_2)$ | Simple forward rate for $[T_1, T_2]$ |
| $c$ | Convexity adjustment (decimal) |
| $\sigma$ | Annual absolute volatility of short rate (decimal) |
| $t$ | Time to futures expiration (years) |
| $\beta$ | Underlying deposit period (years) |
| $\tau$ | Accrual factor (year fraction) |
| $P(t, T)$ | Discount factor at time $t$ for maturity $T$ |
| $N_{\text{fut}}$, $N_{\text{fwd}}$ | Notional amounts for futures/forward |
| $\bar{r}_{\text{FF}}$ | Monthly average Fed Funds rate |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does "STIR" stand for? | Short-Term Interest Rate (futures) |
| 2 | What is the Eurodollar futures quote convention? | $Q = 100 - R^{\text{fut}}$ where $R^{\text{fut}}$ is in percent |
| 3 | If ED futures quote is 94.50, what is the implied rate? | 5.50% (since $100 - 94.50 = 5.50$) |
| 4 | What is the DV01 of one Eurodollar futures contract? | \$25 per basis point |
| 5 | How is the forward rate derived from discount factors? | $L = (P(T_1) - P(T_2)) / (\tau \cdot P(T_2))$ |
| 6 | When are futures and forward rates equal? | When interest rates are deterministic (constant) |
| 7 | What is Hull's definition of the convexity adjustment? | Forward rate = Futures rate $- c$ |
| 8 | Is the convexity adjustment typically positive or negative? | Positive |
| 9 | What does $c > 0$ imply about futures vs forward rates? | Futures rate exceeds forward rate |
| 10 | What is the key mechanism causing futures-forward differences? | Daily mark-to-market settlement |
| 11 | Why does daily settlement create a difference? | Gains reinvested at prevailing rates; losses financed at prevailing rates; correlation creates asymmetry |
| 12 | How does the adjustment scale with volatility? | Quadratically ($c \propto \sigma^2$) |
| 13 | How does the adjustment scale with time to expiry? | Approximately quadratically ($c \propto t^2$) |
| 14 | What is Tuckman's approximation formula? | $r^{\text{fut}} - r^{\text{fwd}} \approx \sigma^2 t^2/2 + \sigma^2 \beta t/2$ |
| 15 | What is $\beta$ in the approximation? | Underlying deposit period (typically 0.25 for 3M) |
| 16 | What is the adjustment for a 5-year contract at 100 bp vol? | Approximately 13 bp |
| 17 | What is the adjustment for a 10-year contract at 100 bp vol? | Approximately 50 bp |
| 18 | What is the sanity check when $\sigma = 0$? | Adjustment should equal zero |
| 19 | What is the Fed Funds futures DV01 (30-day month)? | \$41.67 per basis point |
| 20 | How do you compute the hedge ratio? | # contracts = $-\text{DV01}_{\text{target}} / \text{DV01}_{\text{fut}}$ |
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

**3.** Using Tuckman's approximation with $\sigma = 0.01$, $\beta = 0.25$, $t = 3$, compute the convexity adjustment in bp.

**4.** With the same parameters as Q3, what happens to the adjustment if volatility doubles to $\sigma = 0.02$?

**5.** Your exposure has DV01 = $-\$250,000$/bp. How many Eurodollar futures hedge this? Should you buy or sell?

**6.** In a deterministic-rate world, what should the convexity adjustment be? Explain why.

**7.** Consider four futures contracts expiring at 1y, 2y, 3y, 4y with quotes 97.50, 97.00, 96.50, 96.00. Convert to forward rates using $\sigma = 0.01$, $\beta = 0.25$.

**8.** For a contract with $t = 2$ years and $\sigma = 150$ bp, compute the convexity adjustment. Compare to the 100 bp case.

**9.** Explain in one sentence why the futures rate exceeds the forward rate.

**10.** A desk builds curves by treating STIR futures rates as forwards without adjustment. What two diagnostics would reveal the convexity-related bias?

**11.** For Fed Funds futures, compute the DV01 for a 31-day month.

**12.** You need to hedge a \$200 million 9-month forward exposure. The financing rate is 4%. Compute both the untailed and tailed hedge in ED futures contracts.

**13.** The March Fed Funds futures contract trades at 95.25. An FOMC meeting occurs on March 19 (day 19 of 31). Current target is 4.50%. What is the implied post-meeting rate?

**14.** What is the DV01 of a "Green pack" (4 contracts in the 2-3 year segment)?

**15.** Qualitatively, why does convexity adjustment matter more for longer-dated contracts?

---

### Solution Sketches (Questions 1-8)

**1.** $R^{\text{fut}} = 100 - 96.40 = 3.60\%$; decimal: $r^{\text{fut}} = 0.0360$.

**2.** $L = (0.9600 - 0.9480)/(0.25 \times 0.9480) = 0.0120/0.2370 = 5.06\%$.

**3.** $\sigma^2 = 0.0001$. Pure F-F: $0.0001 \times 9/2 = 0.00045$. Convexity: $0.0001 \times 0.25 \times 3/2 = 0.0000375$. Sum = 0.0004875 = 4.88 bp.

**4.** $\sigma^2$ quadruples when $\sigma$ doubles, so adjustment quadruples: $4 \times 4.88 = 19.5$ bp.

**5.** $\#$ contracts $= -(-250{,}000)/25 = +10{,}000$ contracts. Buy 10,000 contracts (negative DV01 means exposure is short rates, so hedge by going long futures).

**6.** Zero. With no volatility, there are no mark-to-market cash flows, so the settlement timing has no value and futures equal forwards.

**7.** Convexity adjustments: 1y = 0.63 bp, 2y = 2.25 bp, 3y = 4.88 bp, 4y = 8.50 bp. Forward rates: 1y = 2.494%, 2y = 2.978%, 3y = 3.451%, 4y = 3.915%.

**8.** At $\sigma = 0.015$: $\sigma^2 = 0.000225$. Pure F-F: $0.000225 \times 4/2 = 0.00045$. Convexity: $0.000225 \times 0.25 \times 2/2 = 0.0000563$. Total = 5.06 bp. Compare to 100 bp case: $0.0001 \times 4/2 + 0.0001 \times 0.25 \times 2/2 = 2.25$ bp. The 150 bp volatility gives 2.25× the adjustment.

---

## References

- Hull, *Options, Futures, and Other Derivatives* (futures vs forwards; STIR futures mechanics; convexity adjustments).
- Tuckman & Serrat, *Fixed Income Securities: Tools for Today’s Markets* (STIR futures, tailing, and convexity-adjustment intuition).
- Andersen & Piterbarg, *Interest Rate Modeling* (Vol 1) (measure-change view of futures rates and sensitivity mechanics).
