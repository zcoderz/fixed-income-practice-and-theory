# Chapter 14: Key-Rate DV01 and Bucket Exposures

---

## Introduction

Your risk report shows the portfolio's DV01 is zero. Perfect—you're flat. Then the yield curve steepens: 30-year rates rise 10 basis points while 2-year rates fall 5 basis points. You lose $2 million. What went wrong?

The answer lies in a fundamental limitation of parallel DV01: it measures sensitivity to a single, stylized curve movement where every rate shifts by the same amount. Real yield curves, however, are dynamic objects. They steepen, flatten, "twist," and "butterfly." A portfolio can be perfectly hedged against a parallel shift while carrying enormous exposure to these shape changes. As Tuckman explicitly warns in Chapter 7, a portfolio with zero parallel DV01 "can hardly be said to have no interest-rate risk"; it simply profits from one curve shape and loses from another.

This limitation has profound practical consequences. Hull (RM Ch 9) emphasizes that "a U.S. government bond trader's portfolio is likely to consist of many bonds with different maturities" with "an exposure to movements in the one-year rate, the two-year rate, the three-year rate, and so on." The trader's delta exposure is therefore "more complicated than that of the gold trader"—a single number cannot capture this multi-dimensional risk. If you've worked in middle office, you've seen these exposures on your daily risk reports—the key-rate buckets, the partial durations, the delta ladders. This chapter teaches you what those numbers actually mean, how they're computed, and why they matter for trading decisions and P&L attribution.

This chapter develops the tools for multi-dimensional rate risk. **Key-rate DV01** (KRDV01) decomposes a single DV01 number into a vector of exposures at specific maturity points, allowing you to hedge 2-year risk with 2-year bonds and 10-year risk with 10-year bonds. **Bucket exposures** take this further, measuring sensitivity to individual forward-rate segments—a "GAP management" approach essential for swaps and futures trading.

We begin in **Section 14.1** by defining the limits of parallel DV01. **Section 14.2** introduces Tuckman's key-rate decomposition, applies it to bonds and swaps, and provides the trade vocabulary that maps curve positions to KRDV01 profiles. **Section 14.3** covers bucket exposures, their use with STIR futures, and the pack/bundle conventions traders actually use. **Section 14.4** demonstrates the "zero-risk" illusion—showing rigorously how a DV01-neutral portfolio can bleed money in a curve twist. **Section 14.5** formulates the hedging problem as a linear system, including what happens when you can't trade all instruments. **Section 14.6** addresses practical implementation issues, including P&L attribution workflows and the residual risk that hedges leave behind. Finally, **Section 14.7** previews the connection to Chapter 16's PCA approach.

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

### 14.2.6 Trade Nomenclature and KRDV01 Profiles

One of the most important skills for transitioning from middle office to the trading desk is understanding how traders describe their positions. When a trader says "I'm long 2s-10s" or "I'm running a steepener," they're describing a specific KRDV01 profile. This section maps trade vocabulary to risk signatures.

**Table 14.2: Trade Vocabulary and KRDV01 Signatures**

| Trade Name | KRDV01 Profile | Profits When | Example Position |
|:---|:---|:---|:---|
| **Bullet** | Risk concentrated at one tenor | That tenor rallies | Long 10y only |
| **Barbell** | Risk at wings, short middle | Curve bows; belly cheapens | Long 2y+30y, short 10y |
| **Steepener (2s-10s)** | Long front, short back | Curve steepens (long end rises vs short end) | Long 2y, short 10y |
| **Flattener (2s-10s)** | Short front, long back | Curve flattens (long end falls vs short end) | Short 2y, long 10y |
| **Butterfly (2s-5s-10s)** | Wings vs belly | Belly richens/cheapens | Long 2y+10y, short 5y |

> **Desk Reality: Understanding Trader Speak**
>
> When a portfolio manager says "I'm long 50k DV01 in 2s-10s," they mean:
> - **KR01(2y) = +$50,000/bp** (long the 2-year)
> - **KR01(10y) = -$50,000/bp** (short the 10-year)
> - **Net DV01 = 0** (hedged against parallel moves)
>
> The position profits if the 2s-10s spread widens (curve steepens). A 10bp steepening generates approximately $50,000 × 10 + $50,000 × 10 = $1,000,000 in profit.
>
> **Sign convention is crucial:** "Long 2s-10s" means *long the spread*, which means you profit when 2s-10s widens. To widen, either 2-year yields fall or 10-year yields rise. You are therefore **long 2y bonds** (gains when 2y yields fall) and **short 10y bonds** (gains when 10y yields rise).

**Worked Example: Constructing a 2s-5s-10s Butterfly**

A butterfly isolates curvature risk while hedging level and slope. The trader wants to express the view that the 5-year is "rich" (yield too low) relative to 2-year and 10-year.

**Setup:**
- **Body (sell):** $100mm face of 5y bonds, DV01 = $4,500/bp
- **Wings (buy):** 2y and 10y bonds

**Step 1: Determine wing DV01s**
For a standard butterfly, wings are sized to make the position:
1. DV01-neutral (hedges level)
2. Duration-neutral across slope

Common approach: split the body DV01 equally between wings.
- 2y DV01 = $2,250/bp
- 10y DV01 = $2,250/bp

**Step 2: Calculate notionals**
If 2y DV01 per $100mm = $1,800/bp → need $125mm face
If 10y DV01 per $100mm = $8,500/bp → need $26.5mm face

**Resulting KRDV01 vector:**
| 2y | 5y | 10y | Net |
|:---:|:---:|:---:|:---:|
| +$2,250 | -$4,500 | +$2,250 | **0** |

**Interpretation:** The butterfly is DV01-neutral but has **positive curvature exposure**. If the 5y yield rises 5bp while 2y and 10y are unchanged:
- P&L = -(-$4,500) × 5 = +$22,500

The trade profits when the curve becomes more convex (belly cheapens relative to wings).

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

### 14.3.4 STIR Desk Conventions: Packs, Bundles, and Colors

> **Practitioner Note: STIR Desk Pack Conventions**
>
> STIR (Short-Term Interest Rate) traders organize Eurodollar/SOFR futures into "packs" and "bundles" for risk management. This vocabulary is essential for anyone interacting with a STIR desk or reading their risk reports.
>
> **Definitions:**
> - **Pack:** 4 consecutive quarterly contracts (covering 1 year of forward exposure)
> - **Bundle:** The first N years of contracts (e.g., "2-year bundle" = contracts 1-8)
>
> **Color coding for packs (industry standard):**
>
> | Pack | Contracts | Forward Period | Color |
> |:---:|:---:|:---:|:---:|
> | 1 | 1-4 | 0-1 year | **White** |
> | 2 | 5-8 | 1-2 years | **Red** |
> | 3 | 9-12 | 2-3 years | **Green** |
> | 4 | 13-16 | 3-4 years | **Blue** |
> | 5 | 17-20 | 4-5 years | **Gold** |
>
> **Usage examples:**
> - "I'm short the Red Pack" = short exposure to the second strip of quarterly STIR contracts (often interpreted as “year-2” forward exposure; naming varies by desk)
> - "We're long the 3-year bundle" = long contracts 1-12 (0-3 years forward)
> - "Sell 200 Whites and buy 200 Reds" = a steepener in the front of the curve
>
> **Why packs matter for risk:** Instead of reporting 40 individual bucket exposures, traders aggregate risk into pack DV01s. A swap's bucket exposure might translate to: White Pack +$15k, Red Pack +$12k, Green Pack +$8k, etc. This provides a cleaner view of where risk is concentrated.
>
> Note: This terminology is desk slang, not found in academic textbooks. It evolved from the trading floor and varies slightly by institution.

**Worked Example: Translating Swap Risk to Pack DV01s**

A 5-year receiver swap has bucket exposures ($/bp) concentrated in the first 20 quarterly contracts:

| Contracts | 1-4 | 5-8 | 9-12 | 13-16 | 17-20 |
|:---|:---:|:---:|:---:|:---:|:---:|
| Bucket DV01 | +$1,200 | +$1,100 | +$950 | +$800 | +$600 |
| Pack | White | Red | Green | Blue | Gold |

**Pack DV01 summary:** The trader reports "I'm long $1,200 Whites, $1,100 Reds, $950 Greens, $800 Blues, $600 Golds."

To hedge with futures: sell White Pack (48 contracts), sell Red Pack (44 contracts), etc.

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
Use the first-order approximation (Chapter 11): for key-rate shocks in bp,
$$\Delta P \approx -\sum_k \mathrm{KR01}_k \times \Delta r_k.$$
Here, the portfolio has:
- $\mathrm{KR01}_{2y} \approx -\$8{,}500/\text{bp}$ (short 2y)
- $\mathrm{KR01}_{10y} \approx +\$8{,}500/\text{bp}$ (long 10y)

1. **2y Key:** $\Delta r_{2y} = -10$ bp  
   $$\Delta P_{2y} \approx -(-8{,}500)\times(-10)= -\$85{,}000.$$
   Intuition: the 2y bond price rises when yields fall; being short loses money.

2. **10y Key:** $\Delta r_{10y} = +10$ bp  
   $$\Delta P_{10y} \approx -(+8{,}500)\times(+10)= -\$85{,}000.$$

**Net P&L:** $-\$170{,}000$.

This is the key lesson: **DV01-neutrality does not mean “one leg wins when the other loses.”** It means the *sum* offsets under a **parallel** move. Under a twist, you can lose on both legs.

**Twist Scenario 2: Asymmetric Flattening (Front-End Sell-Off)**
The curve **flattens**: 2y rates rise 20 bps, 10y rates unchanged.

- **2y P&L:** Short position *gains* when rates rise.  
  $$\Delta P_{2y} \approx -(-8{,}500)\times(+20)= +\$170{,}000.$$
- **10y P&L:** Rates unchanged $\Rightarrow \Delta P_{10y}\approx 0$.
- **Net P&L:** **+$170,000.**

This DV01-neutral portfolio is therefore a **2s–10s flattener** (short the front end, long the back end). If you don't intend to take a slope view, net DV01 = 0 is not a hedge.

### 14.4.2 The General Formula

For any portfolio with KRDV01 vector $\mathbf{k} = (k_1, k_2, \ldots, k_n)$ and a curve shift vector $\boldsymbol{\delta} = (\delta_1, \delta_2, \ldots, \delta_n)$, the first-order P&L is:

$$\boxed{\Delta P \approx -\mathbf{k}^\top \boldsymbol{\delta} = -\sum_i k_i \delta_i}$$

A parallel shift has $\delta_i = c$ for all $i$, so:

$$\Delta P_{\parallel} = -c \sum_i k_i = -c \cdot \text{DV01}_{\parallel}$$

If $\sum_i k_i = 0$ (DV01 neutral), parallel shifts produce zero P&L. But non-parallel shifts (where $\delta_i$ varies) can produce substantial P&L if the $k_i$ are not all zero.

> **Desk Reality: The "Zero-Risk" Trap**
>
> Junior traders sometimes think a net DV01 of zero means "no interest rate risk." Experienced risk managers know to look at the KRDV01 vector.
>
> **Warning signs:**
> - Large positive and negative key-rate exposures that happen to sum to zero
> - Concentration at distant maturities (2y vs 30y) rather than adjacent ones
> - Any position described as a "steepener" or "flattener"
>
> **The rule:** If your KRDV01 vector looks like $(+100, 0, 0, -100)$, you may have zero parallel risk but $\pm$\$200k of curve risk per basis point of twist.

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

### 14.5.4 Liquidity Constraints and the Art of Approximate Hedging

The mathematically optimal hedge $\mathbf{n} = -\mathbf{H}^{-1}\mathbf{k}$ is rarely the hedge you actually trade. Real-world constraints intervene:

**Why the "optimal" hedge isn't traded:**

1. **Liquidity:** Only certain tenors are liquid (2y, 5y, 10y, 30y benchmarks). Off-the-run 7y or 15y bonds may have wide bid-ask spreads.

2. **Transaction costs:** Each hedge instrument incurs bid-ask spread and market impact. The "optimal" 12-instrument hedge may cost more in execution than it saves in risk reduction.

3. **Balance sheet:** Banks face leverage ratio and RWA constraints. Holding 20 hedge instruments may be capital-inefficient.

4. **Operational complexity:** More positions mean more settlements, more financing, more fails risk.

**The practical approach: hedging with liquid benchmarks only**

Tuckman notes: "Using 20 key rates and 20 securities to hedge a portfolio might very well be feasible if the portfolio's composition were relatively constant over time. But trading 20 securities every time the portfolio or its sensitivities change would probably prove too costly and onerous."

Most trading desks hedge with only the liquid benchmarks: 2y, 5y, 10y, and 30y. This means:
- $L = 4$ hedging instruments
- $K > 4$ risk buckets (often 8-10 or more)

The result: **residual risk at non-hedged tenors**.

> **Desk Reality: The Liquidity-Constrained Hedge Workflow**
>
> **Step 1:** Compute full KRDV01 vector (e.g., 10 buckets).
>
> **Step 2:** Map to liquid hedge points. Common approaches:
> - **Nearest neighbor:** Assign each bucket to the closest liquid hedge
> - **Pro-rata interpolation:** Distribute intermediate buckets between adjacent hedges
> - **Optimization with penalty:** Use Andersen's weighted optimization with high $U_l$ for illiquid instruments
>
> **Step 3:** Accept residual risk. If your portfolio has $3,000/bp at the 7y point and you hedge only with 5y and 10y:
> - Some risk is killed by the 5y hedge (captures 5y-7y portion)
> - Some risk is killed by the 10y hedge (captures 7y-10y portion)
> - The residual depends on how the 7y actually moves vs. the interpolated assumption
>
> **Step 4:** Monitor and document. Risk managers track the unhedged buckets separately. If the 7y "does something very different from what was assumed" (Tuckman's warning), P&L attribution will show an unexplained residual.

**Worked Example: Hedging a 15-Year Swap with Only 10y and 30y Bonds**

Your portfolio has KRDV01 of $5,000/bp concentrated at the 15-year point. Liquid hedging instruments are only 10y and 30y.

**Option 1: Nearest neighbor**
- 15y is closer to 10y than 30y (5 years vs 15 years)
- Hedge entirely with 10y bond
- Residual risk: if 15y moves differently than 10y, P&L breaks

**Option 2: Linear interpolation**
- 15y is 25% of the way from 10y to 30y
- Allocate: 75% to 10y hedge, 25% to 30y hedge
- 10y hedge: $3,750/bp; 30y hedge: $1,250/bp
- Residual: depends on how well 15y interpolates between 10y and 30y

**Option 3: Regression-based**
- Historical regression: $\Delta y_{15} = 0.82 \Delta y_{10} + 0.23 \Delta y_{30}$ (hypothetical)
- Allocate: 82% to 10y, 23% to 30y
- Residual: the regression standard error × notional

In practice, most desks use some version of Option 2 or 3, accepting that the hedge is imperfect.

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
2. **Cumulative par-point approach:** Andersen & Piterbarg describe this variant where "the shift to the $i$-th benchmark security is retained while calculating the derivative to the $(i+1)$-th (and subsequent) securities." This means each bump is cumulative, reducing the oscillatory behavior. The forward curve changes are smoother because each successive par-point shift builds on the previous ones rather than creating isolated spikes.

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

### 14.6.4 P&L Attribution Workflow and Unexplained Residuals

One powerful application of KRDV01 is **P&L attribution**. Given daily changes in key rates $\Delta \mathbf{r}$, the explained P&L is:

$$\text{P\&L}_{\text{rates}} = -\sum_k \text{KR01}_k \times \Delta r_k$$

The **unexplained residual** captures basis, credit, and model error:

$$\text{Residual} = \text{Actual P\&L} - \text{P\&L}_{\text{rates}}$$

> **Desk Reality: The Daily P&L Explain Workflow**
>
> Every morning, middle office produces a P&L attribution report. Here's the workflow:
>
> **Step 1: Capture yesterday's KRDV01 vector**
> - Start-of-day positions × key-rate sensitivities per unit
>
> **Step 2: Capture rate changes**
> - Pull closing rates for each key tenor
> - Compute $\Delta r_k$ = today's rate − yesterday's rate
>
> **Step 3: Compute explained P&L**
> - $\text{P\&L}_{\text{rates}} = -\sum_k \text{KR01}_k \times \Delta r_k$
> - Include second-order (convexity) term for large moves: $\frac{1}{2}\sum_k \gamma_k (\Delta r_k)^2$
>
> **Step 4: Compare to actual P&L**
> - Actual P&L comes from front-office systems (mark-to-market)
> - Residual = Actual − Explained
>
> **Step 5: Investigate significant residuals**
> - Tolerance typically 5-10% of absolute P&L or a fixed dollar threshold
> - Large residuals trigger escalation

**What a large unexplained residual signals:**

| Residual Pattern | Likely Cause | Investigation |
|:---|:---|:---|
| Systematic positive bias | Credit spread tightening not captured | Add spread duration to attribution |
| Systematic negative bias | Funding costs not captured | Check repo rates, FVA |
| Large single-day residual | Basis move, liquidity event, or trade error | Check individual position marks |
| Residual correlated with vol moves | Gamma/convexity understated | Review bump size in KRDV01 calculation |
| Residual correlated with time | Theta (carry) not captured | Add theta term to attribution |

**Worked Example: Daily P&L Attribution**

| Tenor | KR01 ($/bp) | Rate Δ (bp) | Explained P&L |
|:---:|:---:|:---:|:---:|
| 2y | +$3,000 | -2 | +$6,000 |
| 5y | +$8,000 | +1 | -$8,000 |
| 10y | -$5,000 | +3 | +$15,000 |
| **Total** | | | **+$13,000** |

**Actual P&L:** +$10,000
**Residual:** -$3,000 (unexplained loss)

**Investigation:** The $3,000 residual is 23% of explained P&L—above threshold. Possible causes:
- Credit spread on corporate holdings widened 2bp (spread duration was $1,500/bp → $3,000 loss)
- Funding cost increased
- Model interpolation error at 7y where the portfolio has risk

### 14.6.5 Pin Risk and Residual Hedge Exposure

When you hedge a smooth swap exposure with discrete bond maturities, the resulting hedge is not perfect at every point on the curve. The residual risk has a characteristic "stepped" profile.

**The Pin Risk Problem**

Consider hedging a 10-year par swap with only 5y and 10y bonds:
- The swap has exposure across all maturities 1-10 years
- The 5y bond kills risk at exactly 5y and nearby (4y-6y region)
- The 10y bond kills risk at exactly 10y and nearby (8y-10y region)
- The 6y-8y region has **residual risk**—the "hole" in the hedge

This residual risk is concentrated at specific points, hence "pin risk." If the 7-year rate moves independently of 5y and 10y interpolation, the hedge fails.

> **Desk Reality: Managing Pin Risk**
>
> **Monitoring:** Risk managers produce reports showing the full KRDV01 profile *after* hedging. Any large residual buckets are flagged.
>
> **Rebalancing triggers:**
> - Large P&L break attributed to unhedged buckets
> - Market regime change that affects interpolation assumptions
> - Portfolio composition change that shifts risk to unhedged tenors
>
> **Mitigation approaches:**
> - Add hedge instruments at intermediate tenors (if liquid enough)
> - Accept residual risk but size it explicitly: "We have $500/bp unhedged at 7y"
> - Use futures strips to fill in the gaps (more granular than bonds)

**Visual: Residual KRDV01 Profile After Hedging**

```
Before Hedging:
Bucket DV01:  [+800  +1200  +1500  +1400  +1100  +900  +700  +400]
Tenor:         2y     3y     5y     7y     10y   15y   20y   30y

After Hedging with 2y, 5y, 10y, 30y only:
Residual:     [  0   +300   +100   +600    0    +400  +200    0 ]

→ The 7y bucket still has +$600/bp exposure despite "hedging"
→ The 15y and 20y buckets have residual because no direct hedge exists
```

---

## 14.7 Connection to Multi-Factor Models and PCA

The key-rate approach is a pragmatic, market-driven solution to curve risk. However, it can also be connected to more formal multi-factor term structure models.

Tuckman (Chapter 13) notes: "One approach toward solving this problem is to construct a model with more than one factor. Say, for example, a short-term rate and a long-term rate were taken as factors. One would then compute a sensitivity or duration with respect to each of the two factors. Hedging and asset-liability management would be implemented with respect to both durations."

Hull (RM Ch 9) demonstrates that **Principal Component Analysis (PCA)** reveals that yield curve movements are well-explained by three factors (level, slope, curvature) that account for approximately 97% of the variance in rate moves. The first factor (level) corresponds to parallel DV01. The second and third factors (slope and curvature) are exactly the risks that KRDV01 helps manage.

**The Chapter 14 / Chapter 16 Boundary:**

| This Chapter (14) | Chapter 16 |
|:---|:---|
| **Mechanics** of key rates and buckets | **Statistical** analysis via PCA |
| How to compute KRDV01 | Why level/slope/curvature explain most variance |
| How to set up the hedge matrix | How to use regression for hedge ratios |
| Trade vocabulary (steepener, butterfly) | Butterfly trade case studies and relative value |
| Pack/bundle STIR conventions | PCA eigenvalues and factor loadings |

**Key insight:** Key-rate analysis and PCA are complementary. KRDV01 gives you hedgeable risk buckets tied to tradeable instruments, while PCA reveals the statistical structure of how those buckets move together. A trader might use KRDV01 for day-to-day hedging but consult PCA results to understand which risks tend to co-move and which are truly independent.

---

## Summary

1. **Parallel Limit:** DV01 summarizes risk into a single number, effectively assuming perfect correlation across the curve. It hides exposure to twists and butterflies.

2. **Key Rates (KRDV01):** Decompose risk into a vector of sensitivities (e.g., 2y, 5y, 10y). Summing KRDV01s recovers the parallel DV01. Tuckman's triangular shift design ensures this sum-to-parallel property.

3. **Trade Vocabulary:** Steepeners, flatteners, and butterflies are specific KRDV01 profiles. Understanding this vocabulary is essential for trading desk communication.

4. **Buckets:** Measure sensitivity to forward-rate segments. Ideal for hedging with Eurodollar/SOFR futures ($25 DV01 per contract). STIR desks organize risk into packs (4 contracts = 1 year) with color conventions (White, Red, Green, Blue, Gold).

5. **Zero ≠ Safe:** A DV01-neutral portfolio can lose money if the curve twists. Only a KRDV01-neutral portfolio is immune to shape changes (at the resolution of the chosen keys).

6. **Linear Hedging:** Multi-curve hedging is a matrix inversion problem ($\mathbf{n} = -\mathbf{H}^{-1}\mathbf{k}$). Par-bond hedges simplify this to a diagonal system.

7. **Liquidity Constraints:** Real hedges use only liquid benchmarks, leaving residual "pin risk" at unhedged tenors. Andersen's weighted optimization formalizes this tradeoff.

8. **P&L Attribution:** KRDV01 enables daily P&L explain. Large unexplained residuals signal basis risk, credit moves, or model error.

9. **Implementation:** Watch out for "bizarre" forward curves from par-yield bumps; prefer zero/forward bumps for complex portfolios. Choose key rates that match your hedging instruments.

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
| **Butterfly** | Long wings, short belly (or reverse) | Isolates curvature while hedging level/slope |
| **Jacobian Method** | Matrix approach to translate risks to hedges | Handles cross-sensitivities between hedges |
| **Pack** | 4 consecutive STIR contracts (1 year forward) | STIR desk risk aggregation unit |
| **Pin Risk** | Residual exposure at unhedged tenors | Why discrete hedges leave gaps |

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
| $W_k$ | Importance weight for bucket $k$ (Andersen) |
| $U_l$ | Penalty weight for hedge $l$ (Andersen) |

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
| 13 | According to Hull, how much of yield curve variance do the first two PCA factors explain? | Over 97% in Hull's example (level + slope). |
| 14 | When is the hedge matrix $\mathbf{H}$ diagonal? | When hedging with par bonds at exactly the key rate maturities. |
| 15 | What is the cumulative par-point approach? | A method where each key rate shift is retained when computing subsequent deltas. |
| 16 | How does forward rate delta differ from KRDV01? | Forward rate delta uses forward curve bumps; KRDV01 typically uses par yield bumps. |
| 17 | What is GAP management? | Hull's term for bucket exposure analysis, common in bank ALM. |
| 18 | Can a portfolio be DV01-neutral and still lose money? | Yes—if it has non-zero KRDV01 exposures and the curve twists. |
| 19 | What is a "steepener" trade in KRDV01 terms? | Long front-end, short back-end (e.g., long 2y, short 10y). |
| 20 | What is a "Red Pack" in STIR desk terminology? | A common term for the second strip of four consecutive quarterly STIR contracts (often interpreted as “year-2” forward exposure; naming varies by desk). |

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

Your risk report shows a +$1,250 bucket exposure to the Dec '26 SOFR contract.

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

**Problem 7: Construct a Steepener**

You want to put on a 2s-10s steepener with $50,000/bp of risk. The 2y bond has DV01 of $1,900 per $1mm face, and the 10y bond has DV01 of $8,200 per $1mm face.

(a) How much notional of each bond do you trade?
(b) Verify the net DV01 is zero.

*Solution Sketch:*
(a) You want KR01(2y) = +$50,000 and KR01(10y) = -$50,000.
- Long 2y: $50,000 / ($1,900 per $1mm) = $26.3mm face
- Short 10y: $50,000 / ($8,200 per $1mm) = $6.1mm face

(b) Net DV01 = +$50,000 - $50,000 = 0. ✓

---

**Problem 8: P&L Attribution with Residual**

A portfolio has the following start-of-day KRDV01 and experiences these rate changes:

| Tenor | KR01 ($/bp) | Δ Rate (bp) |
|:---:|:---:|:---:|
| 2y | +$2,000 | -3 |
| 5y | +$4,000 | +2 |
| 10y | -$3,000 | +5 |

(a) Calculate the explained P&L from rates.
(b) If actual P&L is +$15,000, what is the unexplained residual?
(c) Suggest two possible causes for the residual.

*Solution Sketch:*
(a) Explained = -[(2,000)(-3) + (4,000)(2) + (-3,000)(5)]
= -[-6,000 + 8,000 - 15,000] = -(-13,000) = +$13,000

(b) Residual = $15,000 - $13,000 = +$2,000 (unexplained gain)

(c) Possible causes: (i) Credit spread tightened (position has spread duration); (ii) Convexity gain from large rate move; (iii) Carry/theta from holding bonds overnight.

---

**Problem 9: Constrained Hedging**

You have KRDV01 exposures of $[+1000, +2000, +1500, +500]$ at 2y, 5y, 10y, 20y. However, you can only hedge with 2y and 10y bonds.

(a) What is the best you can do with only these two instruments?
(b) What residual risk remains?

*Solution Sketch:*
(a) Hedge the 2y directly. For 5y, 10y, 20y, you must make approximations.
- Hedge 2y: sell to kill $1,000 at 2y
- Hedge 10y: try to capture risk at 5y, 10y, and 20y

A reasonable approach: map 5y risk half to 2y, half to 10y. Map 20y risk to 10y.
- 2y hedge: $1,000 + (0.5 × $2,000) = $2,000
- 10y hedge: $1,500 + (0.5 × $2,000) + $500 = $3,000

(b) Residual: The 5y bucket has risk that won't perfectly track 2y or 10y. If 5y moves +3bp while 2y and 10y move +2bp, you have $2,000 × 1bp = $2,000 unexplained.

---

**Problem 10: Trade Vocabulary**

For each KRDV01 vector, name the trade:

(a) [+100, -200, +100] at 2y, 5y, 10y
(b) [0, +500, 0] at 2y, 5y, 10y
(c) [-300, +300, 0] at 2y, 10y, 30y

*Solutions:*
(a) **Butterfly** (long wings, short belly)
(b) **Bullet** (concentrated at one tenor)
(c) **Flattener** (short front, long back)

---

**Problem 11: Why Did My DV01-Neutral Portfolio Lose Money?**

Your portfolio has KRDV01 = [+$20k, 0, -$20k] at 2y, 5y, 30y. Net DV01 = 0. Yesterday, 2y rates fell 1bp, 5y rates fell 3bp, and 30y rates fell 4bp. You lost $60,000.

Explain why.

*Solution:*
Using $\Delta V \approx -\sum_i \text{KR01}_i \times \Delta y_i$ (with $\Delta y_i$ in bp):

P&L = -[(+20{,}000)(-1) + (0)(-3) + (-20{,}000)(-4)]
= -[-20{,}000 + 80{,}000]
= -\$60{,}000

The 5y move is irrelevant because KR01(5y) = 0. The key point is that the **30y rallied more than the 2y** (a flattening rally), which hurts a position that is **long the front end and short the long end**.

---

**Problem 12: Pin Risk Analysis**

After hedging a 10-year swap with only 5y and 10y bonds, your residual KRDV01 profile is:
[0, 0, +$800, +$400, 0, 0] at buckets 3y, 5y, 7y, 8y, 10y, 15y.

(a) Where is the pin risk concentrated?
(b) If the 7y rate rises 10bp while 5y and 10y rise only 5bp each, what is your unexplained P&L?

*Solution:*
(a) Pin risk is at 7y (+$800) and 8y (+$400). These buckets weren't directly hedged.

(b) The hedges assume 7y and 8y interpolate between 5y and 10y. Expected 7y move: 5bp. Actual: 10bp. Difference: 5bp.
Unexplained P&L = -$800 × 5bp = -$4,000 (loss because 7y rose more than expected).
Similar for 8y: if it also rose 8bp vs expected 5bp → -$400 × 3bp = -$1,200.
Total unexplained: approximately -$5,200.

---

## References

- Tuckman & Serrat, *Fixed Income Securities: Tools for Today’s Markets* (key-rate durations, triangular shifts, curve-risk intuition, butterflies).
- Hull, *Risk Management and Financial Institutions* (partial durations, yield-curve PCA, and multi-factor delta thinking).
- Andersen & Piterbarg, *Interest Rate Modeling* (curve deltas, Jacobian methods, and hedging optimization).
- Hull, *Options, Futures, and Other Derivatives* (futures risk conventions and DV01 mechanics).
