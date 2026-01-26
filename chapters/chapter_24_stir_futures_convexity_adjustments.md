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
4. **Practical curve building** — how to extract forward rates from futures strips
5. **DV01 and hedging** — contract sensitivities and bucket hedging applications

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

This convention means that a one-basis-point increase in the implied rate corresponds to a 0.01 decrease in price. The standardized relationship makes DV01 calculations straightforward (see Section 24.5).

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

### 24.3.3 Worked Example: EDZ6 (5-Year Contract)

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

### 24.3.4 Adjustment Magnitude by Maturity

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

### 24.3.5 Sensitivity to Volatility

The adjustment scales with $\sigma^2$. For the 5-year contract:

| Volatility | Adjustment |
|------------|------------|
| 50 bp | 3.3 bp |
| 100 bp | 13.4 bp |
| 150 bp | 30.1 bp |

Doubling volatility from 100 bp to 200 bp would quadruple the adjustment to approximately 54 bp. This sensitivity to the volatility assumption is why sophisticated curve-building systems often calibrate the volatility from market data (e.g., from caps or swaptions) rather than using a single fixed number.

> **Practical note:** The choice of volatility for convexity adjustment can materially affect the resulting curve, especially for contracts beyond 3 years. Different dealers may use different volatility assumptions, creating basis between their curves.

---

## 24.4 Building Forward Curves from Futures Strips

### 24.4.1 The Naive Approach (and Its Flaw)

A common but flawed approach treats futures rates directly as forwards:

$$L^{\text{naive}}(0; T_i, T_{i+1}) = r_i^{\text{fut}}$$

This approach ignores the convexity adjustment, systematically biasing the forward curve upward. Hull explicitly warns against this: "When the futures contracts we have just considered last longer than about two years, it does become important to distinguish between futures and forward."

The practical consequence is that swaps or FRAs priced off the naive curve will be mispriced. If you pay fixed on a 5-year swap priced using unadjusted futures rates, you are paying a rate that is systematically too high by the accumulated convexity adjustment effect.

### 24.4.2 The Correct Approach

The proper procedure applies convexity adjustments before bootstrapping:

1. **Convert quotes to futures rates:** $r_i^{\text{fut}} = (100 - Q_i)/100$
2. **Compute convexity adjustment:** $c_i = c(t_i, \sigma, \beta)$ using Tuckman's formula or a model-based approach
3. **Obtain forward rates:** $L_i = r_i^{\text{fut}} - c_i$
4. **Bootstrap discount factors:** Use the adjusted forward rates as in Chapter 17

Andersen and Piterbarg note the importance of this step: "A pre-processing step is normally employed to convert the futures rate quote to a forward rate (FRA) quote" before curve construction.

> **Technique: The Stitching Problem**
>
> Building a curve from futures isn't just about convexity; it's about "Stitching."
>
> 1.  **The Stub**: The period from Today until the 1st Futures expiry. (Use Cash/LIBOR).
> 2.  **The Reds/Greens/Blues**: Futures contracts are color-coded by year.
>     *   **Whites**: First year (Front 4 contracts).
>     *   **Reds**: Second year.
>     *   **Greens**: Third year.
>     *   **Blues**: Fourth year.
> 3.  **The Stitch**: You calculate the discount factor for the 1st expiry. Then multiply by the DF of the 2nd futures period to get the next date.
>     *   $P(T_2) = P(T_1) \times \frac{1}{1 + L_{1,2} \tau}$
>     *   Repeat until you run out of liquid futures (usually 4-5 years/Blue pack). Then switch to Swaps.

### 24.4.3 Worked Example: Adjusting a Futures Strip

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

## 24.5 Risk Measures and Hedging

### 24.5.1 STIR Futures DV01

The standardized tick value makes hedge ratio calculations straightforward. Recall:

- Eurodollar/3M SOFR futures: \$25 per bp per contract
- Fed Funds/1M SOFR futures: \$41.67 per bp per contract (30-day month)

For an exposure with DV01 of $\text{DV01}_{\text{target}}$ dollars per bp, the hedge ratio is:

$$\boxed{\text{\# contracts} = -\frac{\text{DV01}_{\text{target}}}{\text{DV01}_{\text{fut}}}}$$

The negative sign indicates that a long exposure (positive DV01, meaning you lose when rates rise) is hedged by selling futures (which gains when rates rise, since price = 100 - rate).

### 24.5.2 Bucket Hedging with STIR Futures

Tuckman describes the common practice of granular hedging: "As each Eurodollar futures contract is related to a particular three-month forward rate... it is common to divide the first 10 years of exposure into three-month buckets. In this way any bucket exposure may, if desired, be hedged directly with Eurodollar futures."

This **bucket hedging** approach offers several advantages:

1. **Precision:** Each forward rate is hedged with the matching contract, rather than relying on a duration-weighted parallel shift assumption.

2. **Curve risk management:** Non-parallel curve moves (steepeners, flatteners, butterflies) can be captured and hedged.

3. **Liquidity:** STIR futures are among the most liquid instruments in fixed income markets, allowing large positions to be established or unwound quickly.

Tuckman notes that "the longer-maturity Eurodollar futures are not nearly so liquid as the earlier ones," so practical hedge implementation may need to balance precision against liquidity.

### 24.5.3 Worked Example: Hedging a Forward Rate Exposure

**Problem:** You will pay a floating rate on \$500 million notional over the period [4.00, 4.25] years. Compute the hedge using STIR futures.

**Step 1: Compute exposure DV01**

The exposure is to the forward rate $L(0; 4.00, 4.25)$. If this forward rate increases by 1 bp, the floating payment increases by:

$$\Delta \text{Payment} = N \times \tau \times 0.0001 = 500{,}000{,}000 \times 0.25 \times 0.0001 = \$12{,}500$$

But this payment occurs at $T = 4.25$, so we must discount it:

$$\text{DV01} = N \times \tau \times P(0, 4.25) \times 0.0001$$

With $P(0, 4.25) = 0.8100$:

$$\text{DV01} = 500{,}000{,}000 \times 0.25 \times 0.8100 \times 0.0001 = \$10{,}125$$

**Step 2: Compute hedge ratio**

$$\text{\# contracts} = -\frac{10{,}125}{25} = -405 \text{ contracts}$$

The negative sign indicates selling 405 futures contracts. When rates rise, your floating payment increases (a loss), but your short futures position gains (since the futures price falls when rates rise).

> **Practical note:** The discount factor $P(0, 4.25)$ depends on whether you use naive or adjusted forwards. Using the naive forward would give $P = 0.8099$ and DV01 = \$10,124—virtually identical here, but the difference compounds for longer-dated exposures.

---

## 24.6 When Convexity Adjustments Matter

### 24.6.1 Front-End vs. Back-End

For contracts expiring within 1-2 years, the convexity adjustment is typically less than 1-2 bp. Most practitioners ignore it at the front end—the adjustment is smaller than bid-offer spreads and smaller than the uncertainty in other curve-building assumptions.

Beyond 2-3 years, the adjustment becomes material:
- At 5 years: ~13 bp
- At 10 years: ~50 bp

These magnitudes affect pricing and hedging meaningfully. Hull notes: "When the futures contracts we have just considered last longer than about two years, it does become important to distinguish between futures and forward."

### 24.6.2 TED Spread Analysis

Tuckman describes TED (Treasury-Eurodollar) spread analysis, a technique where security values are computed relative to Eurodollar futures rates. The TED spread of a bond is "the spread such that discounting cash flows at Eurodollar futures rates minus that spread produces the security's market price."

Tuckman notes the "obvious theoretical flaw" in this approach: discounting should use forward rates, not futures rates. However, "the magnitude of the difference between forward and futures rates is relatively small for futures expiring shortly."

For bonds maturing within 2-3 years, the bias from ignoring convexity adjustment is typically tolerable. For longer maturities, proper adjustment becomes necessary if accuracy is required.

### 24.6.3 Model Dependence

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

## 24.7 Practical Implementation Notes

### 24.7.1 Common Pitfalls

**Confusing futures rates with forward rates:** Hull explicitly distinguishes them, and the distinction matters. Always ask: "Am I looking at a futures rate or a forward rate?" The answer determines whether a convexity adjustment is needed.

**Forgetting daily settlement is the key mechanism:** The adjustment exists because futures are marked to market daily. Without this feature, futures and forwards would coincide. Any analysis that treats them identically implicitly assumes zero volatility.

**Mixing settlement conventions:** Eurodollar futures settle at the *beginning* of the 3-month period (on the IMM date); 3-month SOFR futures settle at the *end* based on realized overnight compounding. This affects the relationship between the futures rate and the forward rate. Hull notes this distinction explicitly.

**Assuming contract specs without verification:** Tick size, settlement calendar, and index definitions vary by contract. For specific contracts, consult exchange documentation.

### 24.7.2 Verification Tests

**Deterministic rate check:** Set $\sigma = 0$ in any adjustment formula. The result should be $c = 0$. If your formula gives a non-zero adjustment with zero volatility, something is wrong.

**Monotonicity:** The adjustment should increase with time to expiry and with volatility. Both relationships should be monotonic.

**Sign:** For standard interest rate futures, the adjustment should be positive (futures rate > forward rate). A negative adjustment would suggest an error.

**Order of magnitude:** For 5-year contracts with ~100 bp volatility, expect ~10-15 bp adjustment. For 10-year contracts, expect ~50 bp. If your numbers differ dramatically, check your formula and units.

### 24.7.3 Implementation Workflow

When building a curve from STIR futures:

1. Obtain futures quotes and convert to implied rates: $r^{\text{fut}} = (100 - Q)/100$
2. Obtain or estimate rate volatility $\sigma$ (from caps, swaptions, or historical data)
3. Compute convexity adjustments for each maturity using the appropriate formula
4. Subtract adjustments to get forward rates: $r^{\text{fwd}} = r^{\text{fut}} - c$
5. Bootstrap discount factors from adjusted forwards (see Chapter 17)
6. Verify consistency with other market instruments (swaps, FRAs)

---

## Summary

STIR futures are exchange-traded contracts on short-term interest rates, quoted so that higher rates correspond to lower prices ($Q = 100 - R$). The key insight of this chapter is that **futures rates systematically exceed forward rates** due to the daily settlement feature.

The convexity adjustment $c$ bridges the gap:

$$\text{Forward} = \text{Futures} - c$$

where $c$ is positive and grows with volatility and time to expiry.

Tuckman's approximation provides the magnitude:

$$c \approx \frac{\sigma^2 t^2}{2} + \frac{\sigma^2 \beta t}{2}$$

For front-month contracts, the adjustment is negligible (~0.1 bp). For contracts beyond 2-3 years, it becomes material for pricing and hedging (~5-15 bp at 5 years, ~50 bp at 10 years). When extracting forward curves from futures strips, always apply convexity adjustments before bootstrapping—otherwise the curve will be biased upward, affecting all downstream valuations and hedges.

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
| 21 | When can the convexity adjustment be ignored? | For front-month contracts (< 1-2 years) |
| 22 | How does Eurodollar settlement timing differ from SOFR? | ED settles at beginning of period; SOFR at end |
| 23 | What is "bucket hedging"? | Hedging each maturity bucket with the matching futures |
| 24 | What happens if you ignore convexity in curve building? | Forward curve is biased upward |
| 25 | What is a TED spread? | Spread of a security relative to Eurodollar futures rates |
| 26 | What model assumptions underlie Tuckman's formula? | Normal model, no mean reversion, continuous MTM |
| 27 | How do you convert futures rates to forwards for curve building? | Subtract convexity adjustment before bootstrapping |
| 28 | What is the 3M SOFR futures notional? | \$1,000,000 (same as Eurodollar) |
| 29 | What is the Fed Funds futures notional? | \$5,000,000 |
| 30 | What creates the asymmetry in daily settlement effects? | Correlation between rate level and discount rate (they are the same object) |

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

**12.** Qualitatively, why does convexity adjustment matter more for longer-dated contracts?

---

### Solution Sketches (Questions 1-6)

**1.** $R^{\text{fut}} = 100 - 96.40 = 3.60\%$; decimal: $r^{\text{fut}} = 0.0360$.

**2.** $L = (0.9600 - 0.9480)/(0.25 \times 0.9480) = 0.0120/0.2370 = 5.06\%$.

**3.** $\sigma^2 = 0.0001$. Pure F-F: $0.0001 \times 9/2 = 0.00045$. Convexity: $0.0001 \times 0.25 \times 3/2 = 0.0000375$. Sum = 0.0004875 = 4.88 bp.

**4.** $\sigma^2$ quadruples when $\sigma$ doubles, so adjustment quadruples: $4 \times 4.88 = 19.5$ bp.

**5.** $\#$ contracts $= -(-250{,}000)/25 = +10{,}000$ contracts. Buy 10,000 contracts (positive DV01 means exposure is short rates, so hedge by going long futures).

**6.** Zero. With no volatility, there are no mark-to-market cash flows, so the settlement timing has no value and futures equal forwards.

---

## Source Map

### (A) Verified Facts — Specific Sources

| Fact | Source |
|------|--------|
| STIR futures quote convention $Q = 100 - R$ | Hull Ch 6, Tuckman Ch 17, Andersen Vol 1 |
| Eurodollar futures reference 3M USD LIBOR | Hull Ch 6 |
| 3M SOFR futures reference compounded SOFR, settle at end of period | Hull Ch 6 |
| Fed funds futures reference monthly average fed funds rate | Tuckman Ch 17 |
| Forward rate formula from discount factors | Andersen Vol 1 Ch 4, Hull Ch 4 |
| Convexity adjustment definition: Forward = Futures $- c$ | Hull Ch 6: "Forward Rate = Futures Rate - c" |
| Convexity adjustment is positive | Hull Ch 6: "$c$ is positive" |
| Futures = forwards when rates are constant | Hull Ch 5-6: "When interest rates are constant, forward and futures prices are the same" |
| Eurodollar/3M SOFR futures DV01 = \$25/bp | Hull Ch 6: "\$1,000,000 × 0.0001 × 0.25 = 25" |
| Fed funds/1M SOFR futures DV01 = \$41.67/bp (30-day) | Hull Ch 6, Tuckman Ch 17 |
| Tuckman approximation: $\sigma^2 t^2/2 + \sigma^2 \beta t/2$ | Tuckman Ch 17, equations (17.29)-(17.30) |
| "Mark-to-market gains invested at low rates, losses financed at high rates" | Tuckman Ch 17 |
| Eurodollar futures settle at beginning of 3M period | Hull Ch 6 |
| Bucket hedging: "divide first 10 years into three-month buckets" | Tuckman Ch 7, Ch 17 |
| Longer-maturity ED futures less liquid | Tuckman Ch 17 footnote |
| ED convexity adjustment = difference of expectations under Q vs forward measure | Andersen Vol 1 Ch 4, Ch 16.8 |
| Futures rate is Q-martingale under continuous MTM | Andersen Vol 1 Lemma 4.2.2 |
| "Difference between daily and continuous settlement is quite small" | Andersen Vol 1 Ch 4 |
| TED spread definition and "obvious theoretical flaw" | Tuckman Ch 17 |
| "Determining c involves assumption about the underlying interest rate model" | Hull Ch 6 footnote |
| EDZ6 example: 13.4 bp adjustment | Tuckman Ch 17, equation (17.32) |

### (B) Reasoned Inference — Derivation Logic

| Inference | Logic |
|-----------|-------|
| Convexity adjustment grows with $\sigma^2$ and $t^2$ | Direct from Tuckman approximation formula |
| Deterministic-rate limit implies $c \to 0$ | Follows from Hull's statement about constant rates; also formula gives zero when $\sigma = 0$ |
| Ignoring convexity adjustment biases forward curve upward | If $c > 0$, treating futures as forwards overstates forwards by $c$ |
| Hedge ratio formula | Standard DV01 matching logic with sign for direction |
| Adjustment magnitude table | Computed from Tuckman formula with stated parameters |
| Sensitivity to volatility table | Computed from Tuckman formula ($c \propto \sigma^2$) |
| Fed Funds DV01 for 31-day month = \$43.06 | $5{,}000{,}000 \times 0.0001 \times 31/360$ |

### (C) Flagged Uncertainties

| Item | Uncertainty |
|------|-------------|
| Exact exchange tick sizes, settlement calendars for specific current contracts | I'm not sure — need exchange rulebook for specific contracts; conventions may have changed since source publication |
| Exact index definitions beyond cited examples (e.g., specific SOFR compounding rules) | I'm not sure — contract-specific and evolving |
| Mean reversion effects on ED convexity adjustment | Discussed qualitatively; exact magnitude is model-dependent |
| Convexity adjustment for SOFR futures (end-period settlement) | Hull notes difference exists; exact formula may differ from ED case |
| Current market conventions for convexity adjustment (volatility assumptions) | I'm not sure — dealer practices vary and may have evolved |

---

*Last Updated: January 2026*
