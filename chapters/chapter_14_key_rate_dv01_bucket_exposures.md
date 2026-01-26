# Chapter 14: Key-Rate DV01 and Bucket Exposures

---

## Introduction

Your risk report shows the portfolio's DV01 is zero. Perfect—you're flat. Then the yield curve steepens: 30-year rates rise 10 basis points while 2-year rates fall 5 basis points. You lose $2 million. What went wrong?

The answer lies in a fundamental limitation of parallel DV01: it measures sensitivity to a single, stylized curve movement where every rate shifts by the same amount. Real yield curves, however, are dynamic objects. They steepen, flatten, "twist," and "butterfly." A portfolio can be perfectly hedged against a parallel shift while carrying enormous exposure to these shape changes. As Tuckman explicitly warns in Chapter 7, a portfolio with zero parallel DV01 "can hardly be said to have no interest-rate risk"; it simply profits from one curve shape and loses from another.

This limitation has profound practical consequences. Hull (RM Ch 9) emphasizes that "a U.S. government bond trader's portfolio is likely to consist of many bonds with different maturities" with "an exposure to movements in the one-year rate, the two-year rate, the three-year rate, and so on." The trader's delta exposure is therefore "more complicated than that of the gold trader"—a single number cannot capture this multi-dimensional risk. To manage interest rate risk effectively, we need tools that decompose the single DV01 measure into a vector of exposures across the curve.

This chapter develops those tools. **Key-rate DV01** (KRDV01) decomposes a single DV01 number into a vector of exposures at specific maturity points, allowing you to hedge 2-year risk with 2-year bonds and 10-year risk with 10-year bonds. **Bucket exposures** take this further, measuring sensitivity to individual forward-rate segments—a "GAP management" approach essential for swapping and futures trading.

We begin in **Section 14.1** by defining the limits of parallel DV01. **Section 14.2** introduces Tuckman's key-rate decomposition and applies it to bonds and swaps. **Section 14.3** covers bucket exposures and their use with Eurodollar and SOFR futures. **Section 14.4** demonstrates the "zero-risk" illusion—showing rigorously how a DV01-neutral portfolio can bleed money in a curve twist. **Section 14.5** formulates the hedging problem as a linear system, enabling precise immunization against arbitrary curve changes. Finally, **Section 14.6** addresses practical implementation issues, including the "bizarre" forward curve problem that Tuckman highlights when bumping par yields.

---

## 14.1 The Limits of Parallel DV01

Before dissecting the curve, we must understand precisely what parallel DV01 does and does not measure.

### 14.1.1 The Summary Measure

Tuckman (Chapter 5) defines DV01 as the change in value for a one basis point decline in rates:

$$\boxed{\mathrm{DV01} \equiv -\frac{\Delta P}{10{,}000 \times \Delta y}}$$

where $\Delta y$ is the rate change in decimal form. The factor of 10,000 converts standard decimal changes into basis points. In the limit of small changes, this is simply the derivative $-dP/dy$ scaled by $10^{-4}$.

Critically, "parallel" is a strong behavioral assumption. It assumes that if the 10-year Treasury yield moves by 5 bps, the 2-year, 5-year, and 30-year yields also move by exactly 5 bps. This projection of high-dimensional reality onto a single dimension is useful for a quick "altitude check" of risk, but it hides the terrain below.

Tuckman states this limitation sharply: "A major weakness of the approach taken in Chapters 5 and 6... is the assumption that movements in the entire term structure can be described by one interest rate factor. To put it bluntly, the change in the six-month rate is assumed to predict perfectly the change in the 10-year and 30-year rates."

### 14.1.2 What Parallel DV01 Misses

Consider two portfolios constructed to have identical parallel DV01s:

1. **Bullet:** Long $100 million of 10-year notes.
2. **Barbell:** Long $50 million of 2-year notes and $50 million of 30-year bonds.

In a parallel shift, these portfolios perform identically. But if the curve **steepens** (long rates rise, short rates fall), the barbell suffers a massive loss on its 30-year leg that is not offset by the 2-year gain, while the bullet may be relatively unaffected.

Hull (RM Ch 9) provides a worked example with partial durations that makes this concrete. Consider a portfolio with partial durations as shown in Table 14.1:

**Table 14.1: Partial Durations for a Sample Portfolio (Hull RM Table 9.5)**

| Maturity (years) | 1 | 2 | 3 | 4 | 5 | 7 | 10 | Total |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Duration | 0.2 | 0.6 | 0.9 | 1.6 | 2.0 | -2.1 | -3.0 | 0.2 |

The total duration is only 0.2—the portfolio appears nearly immune to rate moves. But Hull demonstrates what happens in a curve rotation. If short rates fall by amounts proportional to $(3e, 2e, e, 0, -e, -3e, -6e)$ for maturities 1y through 10y (a pivot around the 4-year point), the portfolio P&L is:

$$\Delta P/P = -[0.2 \times (-3e) + 0.6 \times (-2e) + 0.9 \times (-e) + 1.6 \times 0 + 2.0 \times e - 2.1 \times 3e - 3.0 \times 6e] = +25.0e$$

Despite having near-zero total duration, the portfolio has massive twist exposure. For a parallel shift of $e$, the percentage change would be only $-0.2e$. Hull concludes: "A portfolio that gives rise to the partial durations in Table 9.5 is much more heavily exposed to a rotation of the yield curve than to a parallel shift."

Parallel DV01 cannot distinguish between these risks. To differentiate them, we need to decompose the single "rate" variable $y$ into a vector of rates.

---

## 14.2 Key-Rate DV01 (KRDV01)

Key-rate DV01 (or "partial DV01" in Hull's terminology) answers a specific question: *Which part of the curve is driving my risk?*

### 14.2.1 The Triangular Shift

To measure local risk, we cannot simply move one point on the curve; we must move a region. Tuckman (Chapter 7) introduces a widely used technique where we select a set of **key rates** (e.g., 2y, 5y, 10y, 30y par yields) and define a specific curve perturbation for each.

> **Analogy: The Piano Keys**
>
> Think of the Yield Curve as a piano keyboard.
> *   **Parallel Shift (DV01)**: You lay a wooden plank across the keys and press down. Everything moves together. This assumes perfect correlation.
> *   **Key Rate Shift**: You press a *single key* with your finger. Only that specific note (and its immediate neighbors via the string vibrations/interpolation) moves.
> *   **Reality**: Markets are played with fingers, not planks.

Tuckman describes the construction: "The two-year key rate affects all par yields of term zero to five, the five-year affects par yields of term two to 10, the 10-year affects par yields of term five to 30, and the 30-year affects par yields from 10 on. The impact of each key rate is one basis point at its own maturity and declines linearly to zero at the term of the adjacent key rate. To the left of the two-year key rate and to the right of the 30-year key rate, the effect remains at one basis point."

A **key-rate shift** for maturity $T_k$ is thus defined as follows:
1. The rate at $T_k$ moves by +1 bp.
2. The rates at adjacent keys ($T_{k-1}$ and $T_{k+1}$) remain unchanged.
3. The impact on intermediate maturities declines linearly to zero.
4. Beyond the first and last keys, the shift is flat (1 bp).

This localized "tent" or "triangle" shape allows us to shock one section of the curve while holding the rest fixed. Importantly, Tuckman notes these shifts satisfy a **sum-to-parallel property**:

$$\boxed{\sum_{k} \text{Shift}_k(t) = 1 \text{ bp for all } t}$$

If you shock all key rates simultaneously by 1 bp, the result is exactly a 1 bp parallel shift of the entire curve. Tuckman confirms: "Since the sum of the key rate shifts is a parallel shift in the par yield curve, the sums of the key rate 01s and durations closely match the DV01 and duration, respectively, under the assumption of a parallel shift in the par yield curve."

### 14.2.2 Why Triangular? A Note on Arbitrariness

Tuckman acknowledges a theoretical weakness: "The fact that the shifts are linear between key rates is not essential. Quite the contrary: The arbitrary shape of the shifts is a theoretical weakness of the key rate approach. One might easily argue, for example, that the shifts should at least be smooth curves rather than piecewise linear segments. However, in practice, the advantage of extra smoothness may not justify the increased complexity caused by abandoning the simplicity of straight lines."

This arbitrariness has practical implications. Different risk systems may produce slightly different KRDV01s for the same portfolio depending on their choice of key rates and interpolation scheme. Traders should understand their system's conventions before comparing risk reports across platforms.

### 14.2.3 Decomposing Exposure

We define the Key-Rate DV01 for key $k$ as the change in portfolio value when only key rate $k$ shifts:

$$\boxed{\mathrm{KR01}_k \equiv P_0 - P^{(k)}}$$

Because of the sum-to-parallel property, the sum of these partial exposures approximately recovers the parallel DV01:

$$\sum_{k} \mathrm{KR01}_k \approx \mathrm{DV01}_{\parallel}$$

This equality holds exactly for linear sensitivities and is a very close approximation for convex instruments like bonds and swaps.

> **The Checksum Sanity Check**
>
> In any risk report, the first thing a manager checks is:
> $$\sum \text{Key Rate Durations} \approx \text{Parallel Duration}$$
>
> If your parallel duration is 5.0, but your key rate durations sum to 3.5, something is broken (likely missing buckets or bad interpolation). This is the "Conservation of Energy" law for risk models.

### 14.2.4 Worked Example: Decomposing a 10-Year Bond

Let's see how this works in practice by decomposing a standard coupon bond.

**Instrument:** 10-year bond, 5% annual coupon, $100 par.
**Base Curve:** Flat at 4% (annual compounding).
**Key Rates:** 2y, 3y, 5y, 7y, 10y.

First, we compute the parallel DV01. The bond price is $108.11. A +1 bp parallel shift lowers the price to $108.02.

**Parallel DV01:** $0.0852 per $100 face.

Now, we apply the key-rate shifts. Each shift affects cashflows in its region:

1. **2y Shift:** Affects the year 1 and year 2 coupons fully, and tapers to zero by year 3.
2. **5y Shift:** Affects coupons in years 3–7 (peaking at 5).
3. **10y Shift:** Affects coupons in years 7–10 and, crucially, the principal repayment.

**Resulting KRDV01 Vector ($/100 face):**

| Key Rate | Exposure | % of Total | Intuition |
|:---|:---:|:---:|:---|
| **2y** | 0.0014 | 1.6% | Only affects first few coupons |
| **3y** | 0.0021 | 2.5% | Small coupon effect |
| **5y** | 0.0039 | 4.6% | Intermediate coupons |
| **7y** | 0.0066 | 7.7% | Intermediate coupons |
| **10y** | 0.0713 | 83.6% | Principal + final coupon |
| **Sum** | **0.0852** | **100%** | **Matches Parallel DV01** |

**Interpretation:**
The risk is heavily concentrated (83.6%) at the 10-year point. This confirms our intuition: a 10-year bond is primarily a 10-year rate instrument. However, 16.4% of the risk comes from the intermediate coupons. If you hedge this bond solely with a 10-year zero-coupon bond, you will be left with a residual "short" exposure to the 2y–7y part of the curve.

### 14.2.5 The Swap Profile: A Different Story

Swaps exhibit a surprisingly different profile. Consider a **5-year par swap** (receive fixed 4%, pay floating). You might expect its risk to be spread out like a bond.

**The result:**
- **1y–4y Keys:** Very small exposure (only the net coupons).
- **5y Key:** **Massive** exposure (nearly 95% of total).

**Why?** The floating leg of a swap behaves like a bond priced at par with high-frequency resets. Effectively, the floating leg has a duration near zero, but its valuation mechanics imply a "principal" payment at maturity (see Chapter 25). Sensitivity to the discount factor at maturity $DF(T)$ dominates the valuation.

**Practical Implication:** When hedging swaps, matching the maturity bucket is critical. You cannot hedge a 5-year swap effectively with a portfolio of 2-year and 3-year notes, even if the durations match, because the swap's risk is almost entirely concentrated at the 5-year point.

---

## 14.3 Bucket Exposures and Forward Rates

While key rates typically shift par yields, **bucket analysis** (or "GAP management" in Hull's terminology) often focuses on **forward rates**. This approach is favored by desks trading Eurodollar futures and short-term interest rate (STIR) products.

> **Cross-Asset Connection: Taleb's Buckets**
>
> Option traders (like Taleb) use this exact same concept for Volatility, called "Vega Buckets."
> Instead of one "Vega" number, they measure risk to 1-month vol, 3-month vol, 1-year vol, etc.
> Whether it's Rates (KRDV01) or Vol (Vega Buckets), the principle is identical: **decompose the curve to isolate the risk.**

### 14.3.1 Forward-Rate Buckets

Instead of shifting yields, we divide the time axis into segments (buckets)—e.g., 0–3 months, 3–6 months, etc. A bucket exposure is the change in portfolio value when *only the forward rate for that specific period* increases by 1 bp.

$$B_j = \frac{\Delta P}{\Delta f_j}$$

Because the forward curve is the fundamental building block of the discount curve, this method offers granular precision. As Tuckman notes, "bucket analysis usually uses very many buckets," whereas key-rate analysis uses few.

Andersen & Piterbarg (Vol 1, Section 6.4) provide additional perspective on forward-rate deltas. They define sensitivities using Gâteaux derivatives of the forward curve:

$$\partial_{k} V_{0}=\left.\frac{d V_{0}\left(f(t)+\varepsilon \mu_{k}(t)\right)}{d \varepsilon}\right|_{\varepsilon=0}$$

where $\mu_k(t)$ are basis functions—typically piecewise flat or triangular. The resulting sensitivities are called **forward rate deltas**.

Andersen notes: "It is common practice to use grids spaced three months apart, with dates on Eurodollar futures maturities. The number of deltas $K$ is thus typically a rather large number, and the $K$ derivatives give a detailed picture of where the portfolio risk is concentrated on the forward curve."

### 14.3.2 Worked Example: Hedging with Eurodollar/SOFR Futures

Eurodollar (and now SOFR) futures are the natural instruments for hedging forward buckets because each contract targets a specific 3-month forward rate.

**The Instrument:**
A standard Eurodollar futures contract has a notional of $1 million and a maturity of 3 months (0.25 years). The tick value is:

$$1{,}000{,}000 \times 1 \text{ bp} \times 0.25 \text{ years} = \$25$$

Thus, **one contract has a DV01 of $25.** (Note: Hull reminds us that futures rates are slightly higher than forward rates due to convexity, but for DV01 purposes, the $25 rule is standard).

**The Hedge Problem:**
You have a swap position where the bucket exposure to the "Sep 2026" forward rate is **-$2,500/bp** (you lose money if Sep '26 rates rise).

**The Solution:**
To hedge, you need an instrument that *gains* $2,500 when rates rise.
A short Eurodollar futures position gains when rates rise (price falls). Each contract provides $25 of gain per bp.

$$\text{Contracts Needed} = \frac{\text{Exposure}}{\text{DV01 per Contract}} = \frac{2{,}500}{25} = 100 \text{ contracts}$$

You sell 100 Sep '26 Eurodollar futures. This neutralizes the risk in that specific 3-month bucket. By repeating this for every quarterly bucket, a trader can "strip out" the entire curve risk of a swap, leaving only the spread risk.

### 14.3.3 The Sum-to-DV01 Property for Buckets

Just as key-rate DV01s sum to parallel DV01, bucket exposures exhibit a similar property. Tuckman notes that "the sum of the bucket exposures equals the total [DV01] for a one-basis-point shift across all buckets."

However, there is a subtlety. When bucket shifts sum to a parallel shift in forward rates, this does *not* generally produce a parallel shift in par yields (the relationship is nonlinear). For instruments where par yields are the natural quoting convention, this distinction matters.

---

## 14.4 The "Zero-Risk" Illusion (Twist Risk)

Proprietary traders often hunt for portfolios that are **DV01-neutral** (no directional risk) but have a view on curve shape (e.g., steepeners). Risk managers, however, fear the portfolio that looks neutral but contains hidden shape risk.

Let's prove rigorously why "Parallel DV01 = 0" offers no protection against twists.

### 14.4.1 Worked Example: The Hidden Twist

**Portfolio:**
- **Long:** $100 million of the 10-year bond (DV01 ≈ $8,500).
  - KRDV01 dominant at 10y.
- **Short:** $479 million of 2-year zero-coupon bonds (DV01 ≈ $1,780 × 4.79 ≈ $8,500).
  - KRDV01 entirely at 2y.

**Parallel Risk:**
$$8{,}500 - 8{,}500 = 0$$

The portfolio is immune to a parallel shift. The risk report says "Net DV01: 0."

**Twist Scenario 1: Symmetric Steepening**
The curve **steepens**: 2-year rates fall 10 bps, 10-year rates rise 10 bps. This is a classic "rotation" around the 5-year point.

**P&L Calculation:**
We sum the product of exposure and rate change for each key:

1. **2y Key:** We are *short* the 2y zero.
   - We gain when 2y rates fall.
   - $\text{Gain} \approx \$8{,}500/\text{bp} \times 10 \text{ bps} = \$85{,}000$.
2. **10y Key:** We are *long* the 10y bond.
   - Rates rise 10 bps → Price falls.
   - $\text{Loss} \approx \$8{,}500/\text{bp} \times 10 \text{ bps} = \$85{,}000$.

**Net P&L:** $+\$85{,}000 - \$85{,}000 = 0$.

In this specific symmetric steepener, the gains and losses offset. But what about asymmetric moves?

**Twist Scenario 2: Asymmetric Flattening**
The curve **flattens**: 2y rates rise 20 bps, 10y rates unchanged.

- **2y P&L:** Short position loses when rates rise.
  - Loss: $8{,}500 \times 20 = \$170{,}000$.
- **10y P&L:** Rates unchanged = 0.
- **Net Loss:** **$170{,}000.**

Despite being DV01 neutral, the portfolio lost $170k. This demonstrates the danger: a DV01-neutral portfolio effectively bets that the spread between 2y and 10y rates will not change. If you don't intend to take that bet, you are not hedged.

### 14.4.2 The General Formula

For any portfolio with KRDV01 vector $\mathbf{k} = (k_1, k_2, \ldots, k_n)$ and a curve shift vector $\boldsymbol{\delta} = (\delta_1, \delta_2, \ldots, \delta_n)$, the first-order P&L is:

$$\boxed{\Delta P \approx -\mathbf{k}^\top \boldsymbol{\delta} = -\sum_i k_i \delta_i}$$

A parallel shift has $\delta_i = c$ for all $i$, so:

$$\Delta P_{\parallel} = -c \sum_i k_i = -c \cdot \text{DV01}_{\parallel}$$

If $\sum_i k_i = 0$ (DV01 neutral), parallel shifts produce zero P&L. But non-parallel shifts (where $\delta_i$ varies) can produce substantial P&L if the $k_i$ are not all zero.

---

## 14.5 Hedge Construction as a Linear System

To immunize a portfolio against *any* curve movement (up to the resolution of our keys), we must set every element of the KRDV01 vector to zero. This is a linear algebra problem.

Let $\mathbf{k}$ be the vector of our portfolio's key-rate exposures.
Let $\mathbf{H}$ be a matrix where column $j$ is the KRDV01 vector of hedging instrument $j$.
We seek a vector of hedge notionals $\mathbf{n}$ such that:

$$\mathbf{k} + \mathbf{H}\mathbf{n} = \mathbf{0}$$

### 14.5.1 The Diagonal Case (Tuckman's Insight)

If we use **par bonds** (trading at par) as our hedging instruments, and our key rate definitions match the bond maturities (e.g., 2y key rate and 2y par bond), the matrix $\mathbf{H}$ becomes nearly diagonal.

Tuckman explains: "If par bonds are used as hedging securities, par yields are a particularly convenient choice. First... each hedging security has an exposure to one and only one key rate. Second, computing the sensitivity of a par bond of a given maturity with respect to the par yield of the same maturity is the same as computing DV01. In other words, the key rate exposures of the hedging securities equal their DV01s."

In this idealized case, the hedge ratio for key $k$ is simple:

$$\boxed{n_k = -\frac{\mathrm{KR01}_k^{\text{portfolio}}}{\mathrm{DV01}_k^{\text{hedge bond}}}}$$

You simply sell enough 2-year bonds to kill the 2-year risk, enough 5-year bonds to kill the 5-year risk, and so on. The hedges don't interfere with each other.

### 14.5.2 The General Case: Matrix Inversion

In reality, we often hedge with off-the-run bonds or futures, which have cross-sensitivities (e.g., a 10-year bond has some sensitivity to the 7-year rate). The matrix $\mathbf{H}$ is not diagonal. We solve for $\mathbf{n}$ using standard inversion:

$$\boxed{\mathbf{n} = -\mathbf{H}^{-1}\mathbf{k}}$$

This "unwinds" the cross-correlations, telling you, for example, to short slightly less of the 10-year because your 7-year hedge already covers some of that risk.

### 14.5.3 The Jacobian Approach (Andersen & Piterbarg)

Andersen & Piterbarg (Vol 1, Section 6.4.3) formalize this as the **Jacobian method for interest rate deltas**. They define an optimization problem for finding the optimal hedge weights:

$$\widehat{\mathbf{p}}=\underset{\mathbf{p}}{\operatorname{argmin}}\left(\sum_{k=1}^{K} W_{k}^{2}\left(\partial_{k} H_{0}(\mathbf{p})-\partial_{k} V_{0}\right)^{2}+\sum_{l=1}^{L} U_{l}^{2} p_{l}^{2}\right)$$

where $W_k$ weights the importance of offsetting the $k$-th bucket and $U_l$ penalizes use of expensive or illiquid hedging instruments. They note: "If there are fewer hedging instruments than shifts to be immunized ($L < K$), then, in general, not all risks can be offset. In this case, the weights $W$ gain in importance as they allow the user to focus hedging on risk buckets deemed more important."

The simple case with $L = K$ and invertible Jacobian yields:

$$\mathbf{p}=\left(\partial \mathbf{H}^{\top}\right)^{-1} \partial \mathbf{V}_{0}$$

which is equivalent to Tuckman's formulation.

---

## 14.6 Practical Implementation

Implementing KRDV01 in a production risk engine requires navigating several "gotchas."

### 14.6.1 The "Bizarre" Forward Curve Problem

Tuckman highlights a subtle but critical issue: how you bump the curve matters.

If you bump the 5-year **par yield** by 1 bp while holding 2y and 10y par yields constant, the implied **forward curve** between 2y and 5y must gyrate wildly to satisfy the no-arbitrage condition.

Andersen & Piterbarg provide a detailed example: "If a 30 year swap yield is shifted by 1 basis point, while 29 year and 31 year are kept unchanged, then evidently the forward Libor rate $L_{30}$ will move by 30 basis points, and the rate $L_{31}$ will move by -30 basis points." They continue: "A shift of 60 basis points... is not small, and may be inappropriate for calculating a first-order derivative. We emphasize that what appears to be a benign 1 basis point rate shift translates into a much larger forward curve move."

**Consequence:** Instruments sensitive to forward shape (like options or steepeners) can show noisy, unstable KRDV01s if par-yield bumps are used.

**Solutions:**
1. **Zero-rate or forward-rate bumps:** Many modern systems use forward-rate buckets instead of par-yield bumps to ensure smoother curve deformations.
2. **Cumulative par-point approach:** Andersen & Piterbarg describe this variant where "the shift to the $i$-th benchmark security is retained while calculating the derivative to the $(i+1)$-th (and subsequent) securities." This reduces oscillatory forward curve behavior.

### 14.6.2 Choosing Key Rates

Tuckman offers guidance on key rate selection: "Key rates are usually defined as par yields or spot rates. If par bonds are used as hedging securities, par yields are a particularly convenient choice."

The number of key rates is a tradeoff:
- **More keys:** Greater precision, but higher dimensionality and potential noise.
- **Fewer keys:** Simpler, but may miss local risk.

For Eurodollar-style hedging, Tuckman notes: "It is common to divide the first 10 years of exposure into three-month buckets. In this way any bucket exposure may, if desired, be hedged directly with Eurodollar futures. Beyond 10 years the exposures are divided according to the same considerations as when choosing the terms of key rates."

### 14.6.3 Convexity and Basis Risk

Even a KRDV01-neutral portfolio is not risk-free.

1. **Convexity:** KRDV01 is a first-order measure. Large rate moves (e.g., ±50 bps) will generate P&L due to convexity (gamma). Chapter 13 develops this second-order risk measure.

2. **Basis Risk:** Hedging a corporate bond portfolio with Treasury key rates ignores the credit spread. If spreads widen while Treasuries rally, your "perfect" hedge effectively doubles up your loss.

3. **Hedge Stability:** Tuckman cautions that "the hedge will work as intended only if the par yields between key rates move as assumed. If the 20-year rate does something very different from what was assumed... the supposedly hedged portfolio will suffer losses or experience gains."

### 14.6.4 P&L Attribution with Key Rates

One powerful application of KRDV01 is **P&L attribution**. Given daily changes in key rates $\Delta \mathbf{r}$, the explained P&L is:

$$\text{P\&L}_{\text{rates}} = -\sum_k \text{KR01}_k \times \Delta r_k$$

The **unexplained residual** captures basis, credit, and model error:

$$\text{Residual} = \text{Actual P\&L} - \text{P\&L}_{\text{rates}}$$

A large unexplained residual signals either model miscalibration or exposure to factors not captured by rate risk (credit, liquidity, volatility).

---

## 14.7 Connection to Multi-Factor Models

The key-rate approach is a pragmatic, market-driven solution to curve risk. However, it can also be connected to more formal multi-factor term structure models.

Tuckman (Chapter 13) notes: "One approach toward solving this problem is to construct a model with more than one factor. Say, for example, a short-term rate and a long-term rate were taken as factors. One would then compute a sensitivity or duration with respect to each of the two factors. Hedging and asset-liability management would be implemented with respect to both durations."

Hull (RM Ch 9) demonstrates that **Principal Component Analysis (PCA)** reveals that yield curve movements are well-explained by three factors (level, slope, curvature) that account for approximately 97% of the variance in rate moves. The first factor (level) corresponds to parallel DV01. The second and third factors (slope and curvature) are exactly the risks that KRDV01 helps manage.

Chapter 16 develops PCA-based hedging in detail. The key insight is that key-rate analysis and PCA are complementary: KRDV01 gives you hedgeable risk buckets tied to tradeable instruments, while PCA reveals the statistical structure of how those buckets move together.

---

## Summary

1. **Parallel Limit:** DV01 summarizes risk into a single number, effectively assuming perfect correlation across the curve. It hides exposure to twists and butterflies.

2. **Key Rates (KRDV01):** Decompose risk into a vector of sensitivities (e.g., 2y, 5y, 10y). Summing KRDV01s recovers the parallel DV01. Tuckman's triangular shift design ensures this sum-to-parallel property.

3. **Buckets:** Measure sensitivity to forward-rate segments. Ideal for hedging with Eurodollar/SOFR futures ($25 DV01 per contract). Andersen's Jacobian framework provides the mathematical foundation.

4. **Zero ≠ Safe:** A DV01-neutral portfolio can lose money if the curve twists. Only a KRDV01-neutral portfolio is immune to shape changes (at the resolution of the chosen keys).

5. **Linear Hedging:** Multi-curve hedging is a matrix inversion problem ($\mathbf{n} = -\mathbf{H}^{-1}\mathbf{k}$). Par-bond hedges simplify this to a diagonal system.

6. **Implementation:** Watch out for "bizarre" forward curves from par-yield bumps; prefer zero/forward bumps for complex portfolios. Choose key rates that match your hedging instruments.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|:---|:---|:---|
| **Parallel DV01** | Value change for uniform 1 bp shift | Quick summary; fails for curve twists |
| **Key-Rate Shift** | Triangular perturbation at maturity $T_k$ | Isolates risk at specific maturities |
| **Sum-to-Parallel** | Key shifts sum to 1 bp parallel shift | Ensures KRDV01 vector sums to DV01 |
| **Partial Duration** | Hull's term: $D_i = -(1/P)(\Delta P_i / \Delta y_i)$ | Equivalent to KRDV01 in duration form |
| **Bucket Exposure** | Sensitivity to one forward-rate segment | Matches Eurodollar/SOFR futures design |
| **Forward Rate Delta** | Andersen's $\partial_k V_0$ via Gâteaux derivative | Rigorous formulation of bucket risk |
| **Twist/Steepener** | Non-parallel curve movement | The main risk missed by parallel DV01 |
| **Jacobian Method** | Matrix approach to translate risks to hedges | Handles cross-sensitivities between hedges |

---

## Notation for This Chapter

| Symbol | Definition |
|:---|:---|
| $\text{DV01}$ | Dollar value of 01 (parallel shift sensitivity) |
| $\text{KR01}_k$ | Key-rate DV01 at key $k$ |
| $B_j$ | Bucket exposure to forward segment $j$ |
| $\mathbf{k}$ | Vector of portfolio key-rate exposures |
| $\mathbf{H}$ | Matrix of hedge instrument key-rate sensitivities |
| $\mathbf{n}$ | Vector of hedge notionals |
| $D_i$ | Partial duration at maturity $i$ (Hull notation) |
| $\partial_k V_0$ | Forward rate delta (Andersen notation) |

---

## Flashcards

| # | Question | Answer |
|:---|:---|:---|
| 1 | What is the sum-to-parallel property of key rates? | The sum of all key-rate partial DV01s equals the parallel DV01 (approximately). |
| 2 | Why is a 10-year bond exposed to the 5-year key rate? | Its intermediate coupons are discounted by rates in the 5-year region. |
| 3 | Contrast the KRDV01 profile of a bond vs. a swap. | Bond risk is distributed across coupon tenors; swap risk is concentrated at maturity. |
| 4 | How do you hedge a bucket exposure of -$5,000/bp with Eurodollar futures? | Sell 200 contracts (-$5000 ÷ $25 = -200). Short exposure needs short hedge for offsetting P&L. |
| 5 | What is the standard DV01 of one Eurodollar futures contract? | $25 (= $1M × 0.0001 × 0.25 years). |
| 6 | A portfolio has DV01=0 but loses money when the curve steepens. What is this? | Curve risk (or twist/shape risk). |
| 7 | What is the hedge solution formula for the linear system? | $\mathbf{n} = -\mathbf{H}^{-1}\mathbf{k}$. |
| 8 | Why does Tuckman warn against par-yield bumps for forward-sensitive products? | They imply "bizarre" oscillations in the forward curve between key rates. |
| 9 | What does Andersen call the matrix of hedge sensitivities? | The Jacobian matrix $\partial \mathbf{H}$. |
| 10 | What is "Basis Risk" in key-rate hedging? | Risk that the spread between instrument (e.g., Corp) and hedge (e.g., Treasury) changes. |
| 11 | Does KRDV01 capture convexity? | No, it is a first-order (linear) measure only. |
| 12 | What is a partial duration? (Hull terminology) | The percentage price change for a 1 bp move at a specific point on the curve. |
| 13 | According to Hull, how much of yield curve variance do the first two PCA factors explain? | Approximately 97.7% (level + slope). |
| 14 | When is the hedge matrix $\mathbf{H}$ diagonal? | When hedging with par bonds at exactly the key rate maturities. |
| 15 | What is the cumulative par-point approach? | A method where each key rate shift is retained when computing subsequent deltas. |
| 16 | How does forward rate delta differ from KRDV01? | Forward rate delta uses forward curve bumps; KRDV01 typically uses par yield bumps. |
| 17 | What is GAP management? | Hull's term for bucket exposure analysis, common in bank ALM. |
| 18 | Can a portfolio be DV01-neutral and still lose money? | Yes—if it has non-zero KRDV01 exposures and the curve twists. |

---

## Mini Problem Set

**Problem 1: Bond Decomposition**

A 3-year zero-coupon bond is priced on a flat 5% curve (continuously compounded).

(a) Calculate its parallel DV01 per $100 face.
(b) If we use key rates at 2y and 5y with triangular interpolation, which key rate captures most of the risk?

*Solution Sketch:*
(a) Price = $100 × e^{-0.05 × 3} = $86.07. DV01 ≈ 3 × 86.07 × 10^{-4} = $0.0258 per $100.
(b) The 3y maturity falls between 2y and 5y. Using linear weights: 2/3 to 2y key, 1/3 to 5y key. So KR01(2y) ≈ 0.0172 and KR01(5y) ≈ 0.0086.

---

**Problem 2: The Twist Trade**

You are Long $10M 2y notes (DV01 = $1,800) and Short $2M 10y notes (DV01 = $9,000 × 0.2 = $1,800). Net DV01 = 0.

(a) The curve pivots: 2y rates down 10 bps, 10y rates up 10 bps. Do you make or lose money?
(b) What is the position called?

*Solution Sketch:*
(a) Long 2y gains (rates down): +$18,000. Short 10y gains (rates up, short position profits): +$18,000. Total: +$36,000.
(b) This is a "Steepener" trade—profits when the curve steepens.

---

**Problem 3: Futures Hedge**

Your risk report shows a +$1,250 bucket exposure to the Dec '25 SOFR contract.

(a) How many contracts do you trade to hedge?
(b) Do you Buy or Sell?

*Solution Sketch:*
(a) Exposure is positive (long rates risk). Each contract = $25 DV01. Contracts = 1,250 / 25 = 50.
(b) Sell 50 contracts (short futures gains when rates rise, offsetting long exposure loss).

---

**Problem 4: Matrix Hedge**

You have risk vector $\mathbf{k} = [100, 200]^\top$ (in $/bp). Your two hedge instruments have KRDV01 vectors of $[10, 2]^\top$ and $[5, 8]^\top$ respectively.

(a) Set up the linear system $\mathbf{H}\mathbf{n} = -\mathbf{k}$.
(b) Solve for $\mathbf{n}$.

*Solution Sketch:*
(a) $\begin{pmatrix} 10 & 5 \\ 2 & 8 \end{pmatrix} \begin{pmatrix} n_1 \\ n_2 \end{pmatrix} = \begin{pmatrix} -100 \\ -200 \end{pmatrix}$

(b) det(H) = 80 - 10 = 70. $H^{-1} = \frac{1}{70}\begin{pmatrix} 8 & -5 \\ -2 & 10 \end{pmatrix}$.
$n_1 = \frac{1}{70}(8 × (-100) + (-5) × (-200)) = \frac{-800 + 1000}{70} = \frac{200}{70} = 2.86$
$n_2 = \frac{1}{70}((-2) × (-100) + 10 × (-200)) = \frac{200 - 2000}{70} = -25.71$

---

**Problem 5: Partial Duration P&L**

Using Hull's example (Table 14.1), calculate the P&L for a $10 million portfolio if rates move as follows:
- 1y: +5 bps, 2y: +4 bps, 3y: +3 bps, 4y: +2 bps, 5y: +1 bp, 7y: -1 bp, 10y: -3 bps

*Solution Sketch:*
P&L = -$10M × [0.2(0.0005) + 0.6(0.0004) + 0.9(0.0003) + 1.6(0.0002) + 2.0(0.0001) - 2.1(-0.0001) - 3.0(-0.0003)]
= -$10M × [0.0001 + 0.00024 + 0.00027 + 0.00032 + 0.0002 + 0.00021 + 0.0009]
= -$10M × 0.00224 = -$22,400 (loss).

---

**Problem 6: Forward Rate Sensitivity**

Andersen notes that bumping a 30y swap rate by 1 bp while holding 29y and 31y fixed causes forward rates to move by approximately ±30 bps.

(a) Why is this problematic for calculating sensitivities?
(b) What alternative bump method addresses this?

*Solution Sketch:*
(a) A 30 bp forward move is not "small" and violates the first-order derivative assumption. It can cause numerical instability and misleading delta estimates.
(b) Use cumulative par-point bumps or direct forward-rate bumps instead of isolated par-yield bumps.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|:---|:---|
| DV01 definition (−ΔP / 10,000 × Δy) | Tuckman Ch 5 |
| "Movements in the entire term structure can be described by one interest rate factor" is a weakness | Tuckman Ch 7 |
| Triangular key-rate shift design | Tuckman Ch 7 |
| Sum-to-parallel property: sum of key shifts = parallel shift | Tuckman Ch 7 |
| 10y bond has intermediate key-rate risk from coupons | Tuckman Ch 7 |
| Par-yield bumps cause bizarre forward curves | Tuckman Ch 7 |
| Par bonds have exposure only to matching key rate | Tuckman Ch 7 |
| Partial duration definition $D_i = -(1/P)(\Delta P_i / \Delta y_i)$ | Hull RM Ch 9 |
| Partial durations sum to total duration | Hull RM Ch 9 |
| Twist example: portfolio more exposed to rotation than parallel | Hull RM Ch 9 (Table 9.5) |
| First two PCA factors explain 97.7% of variance | Hull RM Ch 9 (Table 9.8) |
| Eurodollar futures DV01 = $25 | Hull OFD Ch 6, Tuckman Ch 17 |
| Forward rate deltas using Gâteaux derivatives | Andersen Vol 1, §6.4.2 |
| Jacobian method for hedge optimization | Andersen Vol 1, §6.4.3 |
| Cumulative par-point approach | Andersen Vol 1, §6.4.4 |
| 30y swap bump → ±30 bp forward move | Andersen Vol 1, §6.4.4 |

### (B) Reasoned Inference

- **Swap KRDV01 concentration:** Inferred from the valuation structure of a par swap (floating leg ≈ par), implying risk concentration at maturity.
- **Steepener P&L formula:** Derived algebraically from the dot product of KRDV01 vector and rate shift vector.
- **Bucket-to-DV01 sum property:** Follows from the definition of discount factors as products of forward rates.

### (C) Flagged Uncertainties

- **Exact bump size conventions:** Markets vary between 0.1 bp, 1 bp, or 10 bps bumps. This chapter assumes 1 bp for pedagogy.
- **Interpolation method dependence:** The exact values of KRDV01 depend on the curve interpolation method (spline vs. linear vs. monotone). This is system-specific.
- **Cross-market conventions:** Different markets (USD, EUR, JPY) may use different key rate selections and bucket definitions. I'm not sure about non-USD conventions without additional source verification.
