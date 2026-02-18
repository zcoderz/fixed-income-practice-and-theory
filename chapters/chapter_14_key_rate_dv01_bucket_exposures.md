# Chapter 14: Key-Rate DV01 and Bucket Exposures

---

## Introduction

Your risk report shows the portfolio’s DV01 is zero. Perfect—you’re “flat”. Then the curve steepens: long-end rates rise while front-end rates fall. You lose money. What went wrong?

Parallel DV01 measures sensitivity to one stylized move: a **uniform shift of the curve**. Real yield curves move in shapes (steepen/flatten/twist/butterfly). A portfolio can be hedged to a parallel shift and still carry large exposure to curve shape.

Prerequisites: [Chapter 11 — DV01/PV01 — Definitions, Computation, and “What’s Being Bumped”](chapters/chapter_11_dv01_pv01_definitions_computation.md), [Chapter 12 — Duration — Macaulay, Modified, and the Connection to DV01](chapters/chapter_12_duration_macaulay_modified_dv01.md), [Chapter 13 — Convexity — Second-Order P&L and When DV01 Breaks](chapters/chapter_13_convexity.md)  
Follow-on: [Chapter 15 — DV01 Hedging — Hedge Ratios, Risk Weights, and Practical Caveats](chapters/chapter_15_dv01_hedging.md), [Chapter 16 — Curve Hedging Beyond DV01 — Twists, Butterflies, and PCA](chapters/chapter_16_curve_hedging_twists_butterflies_pca.md), [Chapter 22 — Curve Risk Management in a Multi-Curve World — Par-Point Deltas, Jacobians, and Controlled Perturbations](chapters/chapter_22_multi_curve_risk_jacobians.md)

This chapter turns “rates delta” into a **vector**: key-rate DV01s and bucket exposures. You will see how these vectors explain P&L under twists, and how hedging becomes linear algebra with explicit **bump object, bump size, units, and sign**.

We cover: (i) why parallel DV01 misses shape risk, (ii) how key-rate shifts and forward buckets are defined (the bump design), (iii) a worked example with unit/sign checks, and (iv) practical caveats when mapping risk buckets to tradable hedges.

## Learning Objectives
- Translate a risk number into a precise **bump definition** (what is bumped, by how much, and what is held fixed).
- Compute and interpret a **key-rate DV01 vector** and relate it to parallel DV01.
- Compute and interpret **bucket exposures** to forward-rate segments and map them to hedges.
- Size and interpret simple curve trades (steepener/flattener/butterfly) with unit/sign checks.
- Diagnose common breaks: mismatched bump objects across systems and liquidity-constrained hedges leaving residual risk.

---

## 14.1 The Limits of Parallel DV01

Before dissecting the curve, we must understand precisely what parallel DV01 does and does not measure.

### 14.1.1 The Summary Measure

DV01 is a *first-order* (linear) sensitivity of PV to a small change in rates. Throughout this book we use the convention

$$
\mathrm{DV01} := PV(\text{rates down }1\text{bp}) - PV(\text{base}),
$$

where **1bp = $10^{-4}$** in rate units and the **bump object** must be stated (yield, zero rate, par rate, benchmark quote, etc.). For a small move $\Delta y$ expressed in **basis points**, the linear approximation is

$$
\Delta PV \approx -\mathrm{DV01}\\,\Delta y.
$$

A “parallel DV01” implicitly assumes a particular curve move: every relevant maturity shifts by the same $\Delta y$. That makes DV01 a useful one-number summary, but it cannot describe non-parallel moves.

**Check (units and signs):** $\Delta y$ is in **bp** in the formula above. If $\mathrm{DV01}=+USD 50{,}000/\text{bp}$ and rates sell off by $+12$bp under the stated bump object, then $\Delta PV\approx -50{,}000\times 12=-USD 600{,}000$. If your DV01 is reported “per 100 notional” (price points per bp), convert to dollars by multiplying by notional/100 before doing this calculation.

### 14.1.2 What Parallel DV01 Misses

Consider two portfolios constructed to have identical parallel DV01s:

1. **Bullet:** Long USD 100 million of 10-year notes.
2. **Barbell:** Long USD 50 million of 2-year notes and USD 50 million of 30-year bonds.

In a parallel shift, these portfolios perform identically. But if the curve **steepens** (long rates rise, short rates fall), the barbell suffers a massive loss on its 30-year leg that is not offset by the 2-year gain, while the bullet may be relatively unaffected.

One way to see the problem is to look at **partial durations** (or “key-rate durations”): a set of duration-like sensitivities, one per curve point/bucket. Table 14.1 is an illustrative example:

**Table 14.1: Example Partial Durations for a Portfolio**

| Maturity (years) | 1 | 2 | 3 | 4 | 5 | 7 | 10 | Total |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Duration | 0.2 | 0.6 | 0.9 | 1.6 | 2.0 | -2.1 | -3.0 | 0.2 |

The total duration is only 0.2, so the portfolio looks nearly immune to a parallel move. But consider a **rotation** of the curve (a twist) where the changes are proportional to $(3e, 2e, e, 0, -e, -3e, -6e)$ for maturities 1y through 10y. The first-order P&L is

$$\Delta P/P = -[0.2 \times (-3e) + 0.6 \times (-2e) + 0.9 \times (-e) + 1.6 \times 0 + 2.0 \times e - 2.1 \times 3e - 3.0 \times 6e] = +25.0e$$

Despite having near-zero total duration, the portfolio has massive twist exposure. For a parallel shift of $e$, the percentage change would be only $-0.2e$.

Parallel DV01 cannot distinguish between these risks. To differentiate them, we need to decompose the single "rate" variable $y$ into a vector of rates.

---

## 14.2 Key-Rate DV01 (KRDV01)

Key-rate DV01 answers a simple question: *which part of the curve drives my PV sensitivity?* Instead of one “rates DV01”, you report a **vector** of exposures across chosen key tenors.

### 14.2.1 Defining a Key-Rate Shift (Bump Design)

Key rates are usually defined as **par yields** or **spot/zero rates**. Key-rate DV01 then depends on a **methodology**—you must choose:

1. **Key tenors** $0\lt T_1\lt \dots \lt T_K$ (e.g., 2y/5y/10y/30y).
2. What you treat as the **key rates** at those tenors (par yields vs spot/zero rates).
3. An **interpolation / curve-rebuild rule** that turns the key rates into a full curve.
4. A **localized shock shape** (“shift function”) for each key.

To measure *local* risk, we cannot simply “move one point” without saying how the rest of the curve responds. One common construction uses **piecewise-linear** impacts between adjacent keys so that:
- each region of the curve is affected by a combination of the closest key rates,
- each maturity is most affected by the closest key rate,
- the impacts change smoothly across maturities, and
- the sum of the key-rate shifts is a parallel shift of the curve.

Denote the resulting shock at maturity $t$ by $Shift_k(t)$ (in bp).

A convenient property is:

$$
\sum_{k=1}^K Shift_k(t) = 1\text{ bp}\quad \text{for all }t.
$$

If the key-rate shifts sum to a parallel shift, then the key-rate 01s add up (approximately) to the parallel DV01 computed under the same bump design.

**Mechanics (why a “single key” bump moves more than one maturity):** A key-rate shift is implemented by (i) shocking one key input and (ii) re-interpolating/rebuilding the full curve. Even if you “bump the 5y key,” the implied shock $Shift_k(t)$ typically affects a whole maturity *region* between neighboring keys. That is why KR01s are exposure to a **shock shape + rebuild rule**, not exposure to an infinitesimal point on a continuous curve.

### 14.2.2 Arbitrariness and Comparability

The exact shift shapes and the curve-rebuild rule are conventions. Piecewise-linear shifts are common for simplicity, but the arbitrary shape of the shifts is also a theoretical weakness: one could argue for smoother curves at the cost of more complexity.

Key-rate 01 vectors can differ across methodologies if you bump different objects (par yields vs spot/zero rates vs benchmark quotes), use different interpolation, or impose different locality constraints.

> **Pitfall — What is being bumped?:** Key-rate DV01s are not comparable across systems unless the bump object and curve-rebuild rule match.
> **Why it matters:** A hedge ratio sized on one system’s KR01 can be the wrong sign/magnitude in another.
> **Quick check:** Confirm (i) bump object, (ii) bump size (1bp = $10^{-4}$), (iii) interpolation, and (iv) that $\sum_k KR01_k$ is close to the reported parallel DV01.

### 14.2.3 Definition, Units, and Sign

Fix a bump design and let $PV(\cdot)$ denote the re-priced PV under that design. For key $k$, define

$$
KR01_k := PV(\text{key }k\text{ down }1\text{bp}) - PV(\text{base}).
$$

(Many texts/reporting systems instead define “01” as $PV(\text{base})-PV(\text{key }k\text{ up }1\text{bp})$; to first order these agree.)

- **Bump object:** the chosen key rate (e.g., a zero rate $y(0,T_k)$ on a stated basis).
- **Bump size:** 1bp $=10^{-4}$ in rate units.
- **Units:** currency per 1bp, for the stated notional/price convention (e.g., “per 100 notional” or “per USD 1mm”).
- **Sign convention (this book):** $KR01_k\gt 0$ means you gain when that part of the curve rallies (rates fall).

For a small move $\Delta r_k$ in **bp** at each key,

$$
\Delta PV \approx -\sum_{k=1}^K KR01_k\\,\Delta r_k.
$$

If your shift functions satisfy $\sum_k Shift_k(t)=1\text{bp}$, then a practical checksum is

$$
\sum_k KR01_k \approx DV01_{\parallel}.
$$

### 14.2.4 Worked Example (Template): Key-Rate DV01 of a 10-Year Bond on a Toy Curve

**Example Title**: Key-rate DV01 of a 10-year coupon bond

**Context**
- What is being measured? A key-rate DV01 vector $\\{KR01_k\\}$ and its checksum versus parallel DV01.
- Why this matters on the desk: it tells you *where* your rates risk sits and what a “DV01-neutral” hedge might still leave behind.

**Timeline (Make Dates Concrete)**
- Trade/valuation date (assumed): 2026-02-15 (chosen to make the example easy to reproduce)
- Accrual convention: ACT/365F (here each yearly accrual factor is exactly $1.0$)
- Payment dates: 2027-02-15, 2028-02-15, …, 2036-02-15

**Inputs**
- Instrument: fixed-rate bond with notional $N=100$ (price per 100), annual coupon $c=5\\%$, maturity 2036-02-15.
- Curve object: continuously-compounded zero rates $y(0,T)$ at key tenors $T\in\\{1,2,5,10\\}$ (years):
  - $y(0,1)=3.80\\%$, $y(0,2)=4.00\\%$, $y(0,5)=4.20\\%$, $y(0,10)=4.40\\%$.
- Interpolation: linear in maturity for $y(0,T)$ between key tenors.
- Discounting: $P(0,T)=e^{-y(0,T)\\,T}$.

**Outputs (What You Produce)**
- Clean price (and PV): $PV_0 = 104.2683$ per 100 (assume zero accrued interest, so dirty = clean in this toy setup).
- Parallel DV01 (all key zero rates down 1bp): $DV01_{\parallel} \approx 0.0849$ per 100 per 1bp.
- Key-rate DV01s (one node down 1bp, others fixed, re-interpolate):

| Key tenor $T_k$ | $KR01_k$ (per 100 per 1bp) | Share of total |
|:---:|---:|---:|
| 1y | 0.0005 | 0.6% |
| 2y | 0.0024 | 2.8% |
| 5y | 0.0088 | 10.3% |
| 10y | 0.0733 | 86.3% |
| **Sum** | **0.0849** | **100.0%** |

**Check (scale to dollars):** In this example, $KR01_{10y}=0.0733$ is in **price points per 100 per 1bp**. On $N=USD 100\text{mm}$ face, that is $(0.0733/100)\times 100\text{mm} \approx USD 73{,}300/\text{bp}$. The total parallel DV01 of $0.0849$ points per 100 corresponds to about USD 84,900/bp on USD 100mm.

**Step-by-step**
1. Cashflows: yearly coupon $=N\cdot c\cdot 1.0=5$. Final cashflow at maturity $=105$.
2. Build discount factors from the interpolated zero curve.
3. Compute $PV_0=\sum_i CF_i\\,P(0,T_i)$.
4. For each key tenor $T_k$: bump $y(0,T_k)$ **down** by 1bp, rebuild the curve by interpolation, reprice, and take $KR01_k = PV_k - PV_0$.

**Cashflows (table)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2027-02-15 | 5 | coupon |
| 2028-02-15 | 5 | coupon |
| 2029-02-15 | 5 | coupon |
| 2030-02-15 | 5 | coupon |
| 2031-02-15 | 5 | coupon |
| 2032-02-15 | 5 | coupon |
| 2033-02-15 | 5 | coupon |
| 2034-02-15 | 5 | coupon |
| 2035-02-15 | 5 | coupon |
| 2036-02-15 | 105 | coupon + principal |

**P&L / Risk Interpretation**
- Most of the key-rate DV01 sits in the longest node because the principal repayment is long-dated.
- Intermediate nodes matter because coupons are discounted by intermediate maturities; a “10y-only hedge” can leave residual exposure in the belly.
- If the key rates move by $\Delta r_k$ (bp), the first-order P&L per 100 is $\Delta PV \approx -\sum_k KR01_k\\,\Delta r_k$.
  - Example twist: if $r_{2y}$ rises by $+5$bp and $r_{10y}$ falls by $-5$bp (others unchanged), then $\Delta PV \approx -(0.0024)(+5) -(0.0733)(-5) \approx +0.3546$ price points per 100.

**Check (twist P&L in dollars):** $+0.3546$ price points per 100 is $0.3546\\%$ of face. On USD 100mm notional, that is about USD 354,600.

**Sanity Checks**
- Units check: $KR01$ is “price points per 100 per 1bp” in this setup; scaling to a notional $N$ uses $N/100$.
- Sign check: rates down $\Rightarrow$ discount factors up $\Rightarrow$ $PV$ up $\Rightarrow$ $KR01_k\gt 0$ for a long bond.
- Checksum: $\sum_k KR01_k$ should be close to $DV01_{\parallel}$ under a consistent bump design.

### 14.2.5 The Swap Profile: A Different Story

A bond is a stream of fixed cashflows. Its key-rate DV01 comes from how each cashflow is discounted.

A plain-vanilla swap is a difference of two legs. In a single-curve setting, the floating leg resets frequently and tends to behave like “short duration” cash, while the fixed leg is an annuity that runs to maturity. Under many bump designs, the swap’s key-rate DV01 therefore concentrates in longer buckets (where the annuity ends), but the exact shape is **methodology-dependent**.

**Practical implication:** do not assume a swap hedge built by “matching bond durations” will be stable bucket-by-bucket. Always check the full $KR01$ (or forward-bucket) vector produced by your risk system.

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

> **Desk Reality:** Trader language is usually a shorthand for a **KR01 vector** (or a small set of buckets), not for parallel DV01.
> **Common break:** The same phrase can map to different definitions (DV01 vs PV01, up-bump vs down-bump, per 100 vs per USD 1mm).
> **What to check:** Recreate yesterday’s rates P&L with $-\sum_k KR01_k\\,\Delta r_k$ and confirm unit scaling and sign.
>
> Example: “long 50k DV01 in 2s–10s” typically means $KR01_{2y}\approx +50{,}000$ USD/bp, $KR01_{10y}\approx -50{,}000$ USD/bp, with net parallel DV01 close to 0.

**Worked Example: Constructing a 2s-5s-10s Butterfly**

A butterfly isolates curvature risk while hedging level and slope. The trader wants to express the view that the 5-year is "rich" (yield too low) relative to 2-year and 10-year.

**Setup:**
- **Body (sell):** USD 100mm face of 5y bonds. If the DV01 of a USD 100mm **long** position is +USD 4,500/bp, then selling it gives $KR01_{5y}\approx -4{,}500$ USD/bp.
- **Wings (buy):** 2y and 10y bonds

**Step 1: Determine wing DV01s**
For a standard butterfly, wings are sized to make the position:
1. DV01-neutral (hedges level)
2. Duration-neutral across slope

Common approach: split the body DV01 equally between wings.
- 2y DV01 = USD 2,250/bp
- 10y DV01 = USD 2,250/bp

**Step 2: Calculate notionals**
If 2y DV01 per USD 100mm = USD 1,800/bp, you need USD 125mm face.
If 10y DV01 per USD 100mm = USD 8,500/bp, you need USD 26.5mm face.

**Resulting KRDV01 vector:**
| 2y | 5y | 10y | Net |
|:---:|:---:|:---:|:---:|
| +USD 2,250 | -USD 4,500 | +USD 2,250 | **0** |

**Interpretation:** The butterfly is DV01-neutral but has **positive curvature exposure**. If the 5y yield rises 5bp while 2y and 10y are unchanged:
- P&L = -(-USD 4,500) × 5 = +USD 22,500

The trade profits when the curve becomes more convex (belly cheapens relative to wings).

---

## 14.3 Bucket Exposures and Forward Rates

Key-rate DV01 uses a small set of tenors. **Bucket analysis** takes the same idea and makes it more granular by defining many localized shocks, often on the **forward curve**. This is especially natural for instruments whose economics (and hedges) are tied to specific future accrual periods.

### 14.3.1 Forward-Rate Buckets

Divide the timeline into buckets, e.g. $[T_j,T_{j+1}]$ for $j=1,\dots,J$. Let $f_j$ be the forward rate for bucket $j$. A bucket sensitivity is the PV change when **only that bucket** is shifted.

One convenient convention consistent with our DV01 sign is

$$
FD01_j := PV(\text{bucket }j\text{ down }1\text{bp}) - PV(\text{base}),
$$

so that for small bucket moves $\Delta f_j$ in **bp**,

$$
\Delta PV \approx -\sum_{j=1}^J FD01_j\,\Delta f_j.
$$

To formalize bucket shocks, it is useful to write (loosely) $V_0=V_0(f)$ to highlight the dependence of PV on a forward curve $f$, and compute a functional (Gâteaux) derivative under a shaped forward-curve shock $\mu_k(t)$:

$$\partial_{k} V_{0}=\left.\frac{d V_{0}\left(f(t)+\varepsilon \mu_{k}(t)\right)}{d \varepsilon}\right|_{\varepsilon=0}$$

Standard choices for $\mu_k(t)$ are piecewise-triangular or piecewise-flat functions localized to one (or a small number of) buckets.

### 14.3.2 Worked Example: Hedging with Eurodollar/SOFR Futures

Quarterly short-term rate futures are a natural hedge instrument for forward buckets because each contract targets a specific 3-month accrual period (up to details of convexity and the exact contract definition).

**The Instrument:**
If a contract corresponds to a $3$ month accrual period (about $0.25$ years) on a $ USD 1{,}000{,}000$ notional, then a 1bp rate change is approximately

$$1{,}000{,}000 \times 1 \text{ bp} \times 0.25 \text{ years} = USD 25$$

Thus, a convenient “per contract” scale is about $USD 25$ per bp (exact details depend on contract spec and the risk system’s mapping).

**The Hedge Problem:**
You have a position whose exposure to a particular quarterly bucket is $FD01=-USD 2{,}500/\text{bp}$ (you lose money if that bucket’s rate rises).

**The Solution:**
To hedge, you need approximately $+USD 2{,}500/\text{bp}$ in that bucket. If one contract provides about $USD 25/\text{bp}$, then

$$\text{Contracts Needed} = \frac{\text{Exposure}}{\text{DV01 per Contract}} = \frac{2{,}500}{25} = 100 \text{ contracts}$$

The direction (buy/sell) is chosen to offset the sign of the bucket exposure under your system’s conventions.

### 14.3.3 The Sum-to-DV01 Property for Buckets

If your bucket shocks partition the curve (their sum is a parallel shock under the same bump design), then the bucket DV01s add up (approximately) to the parallel DV01.

In practice, sums may differ when you mix bump objects (forward buckets vs par yields) or when curve construction introduces nonlocal effects.

### 14.3.4 Desk Aggregation: Packs and Bundles

> **Desk Reality:** Risk reports sometimes aggregate quarterly bucket DV01s into “packs/bundles” (e.g., grouping consecutive quarterly buckets into annual blocks).
> **Common break:** Pack definitions and contract mappings differ by venue and benchmark; naive aggregation can miss roll conventions and convexity/basis.
> **What to check:** Confirm contract spec, the mapping from curve buckets to listed contracts, and the unit scaling (per contract vs per USD 1mm).

---

## 14.4 The "Zero-Risk" Illusion (Twist Risk)

Proprietary traders often hunt for portfolios that are **DV01-neutral** (no directional risk) but have a view on curve shape (e.g., steepeners). Risk managers, however, fear the portfolio that looks neutral but contains hidden shape risk.

Let's prove rigorously why "Parallel DV01 = 0" offers no protection against twists.

### 14.4.1 Worked Example: The Hidden Twist

**Portfolio:**
- **Long:** USD 100 million of the 10-year bond (DV01 approx USD 8,500/bp).
  - KRDV01 dominant at 10y.
- **Short:** USD 479 million of 2-year zero-coupon bonds (DV01 approx USD 1,780/bp × 4.79 approx USD 8,500/bp).
  - KRDV01 entirely at 2y.

**Parallel Risk:**
$$8{,}500 - 8{,}500 = 0$$

The portfolio is immune to a parallel shift. The risk report says "Net DV01: 0."

**Twist Scenario 1: Symmetric Steepening**
The curve **steepens**: 2-year rates fall 10 bps, 10-year rates rise 10 bps. This is a classic "rotation" around the 5-year point.

**P&L Calculation:**
Use the first-order approximation (Chapter 11): for key-rate shocks in bp,
$$\Delta P \approx -\sum_k KR01_k \times \Delta r_k.$$
Here, the portfolio has:
- $KR01_{2y} \approx -8{,}500$ USD/bp (short 2y)
- $KR01_{10y} \approx +8{,}500$ USD/bp (long 10y)

1. **2y Key:** $\Delta r_{2y} = -10$ bp  
   $$\Delta P_{2y} \approx -(-8{,}500)\times(-10)= -85{,}000\ \text{USD}.$$
   Intuition: the 2y bond price rises when yields fall; being short loses money.

2. **10y Key:** $\Delta r_{10y} = +10$ bp  
   $$\Delta P_{10y} \approx -(+8{,}500)\times(+10)= -85{,}000\ \text{USD}.$$

**Net P&L:** -USD 170,000.

This is the key lesson: **DV01-neutrality does not mean “one leg wins when the other loses.”** It means the *sum* offsets under a **parallel** move. Under a twist, you can lose on both legs.

**Twist Scenario 2: Asymmetric Flattening (Front-End Sell-Off)**
The curve **flattens**: 2y rates rise 20 bps, 10y rates unchanged.

- **2y P&L:** Short position *gains* when rates rise.  
  $$\Delta P_{2y} \approx -(-8{,}500)\times(+20)= +170{,}000\ \text{USD}.$$
- **10y P&L:** Rates unchanged $\Rightarrow \Delta P_{10y}\approx 0$.
- **Net P&L:** **+USD 170,000.**

This DV01-neutral portfolio is therefore a **2s–10s flattener** (short the front end, long the back end). If you don't intend to take a slope view, net DV01 = 0 is not a hedge.

### 14.4.2 The General Formula

For any portfolio with KRDV01 vector $\mathbf{k} = (k_1, k_2, \ldots, k_n)$ and a curve shift vector $\boldsymbol{\delta} = (\delta_1, \delta_2, \ldots, \delta_n)$, the first-order P&L is:

$$\boxed{\Delta P \approx -\mathbf{k}^\top \boldsymbol{\delta} = -\sum_i k_i \delta_i}$$

A parallel shift has $\delta_i = c$ for all $i$, so:

$$\Delta P_{\parallel} = -c \sum_i k_i = -c \cdot DV01_{\parallel}$$

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
> **The rule:** If your KRDV01 vector looks like $(+100, 0, 0, -100)$, you may have zero parallel risk but plus/minus USD 200k of curve risk per basis point of twist.

---

## 14.5 Hedge Construction as a Linear System

To immunize a portfolio against *any* curve movement (up to the resolution of our keys), we must set every element of the KRDV01 vector to zero. This is a linear algebra problem.

Let $\mathbf{k}$ be the vector of our portfolio's key-rate exposures.
Let $\mathbf{H}$ be a matrix where column $j$ is the KRDV01 vector of hedging instrument $j$.
We seek a vector of hedge notionals $\mathbf{n}$ such that:

$$\mathbf{k} + \mathbf{H}\mathbf{n} = \mathbf{0}$$

### 14.5.1 The Diagonal Case (Matched Hedge Tenors)

If you choose key rates and hedge instruments so that each hedge instrument loads almost entirely on one key (e.g., par bonds at the key maturities when key rates are defined as par yields), then the matrix $\mathbf{H}$ is close to diagonal.

In this idealized case, the hedge ratio for key $k$ is simple:

$$\boxed{n_k = -\frac{KR01_k^{\text{portfolio}}}{DV01_k^{\text{hedge bond}}}}$$

You simply sell enough 2-year bonds to kill the 2-year risk, enough 5-year bonds to kill the 5-year risk, and so on. The hedges don't interfere with each other.

### 14.5.2 The General Case: Matrix Inversion

In reality, we often hedge with off-the-run bonds or futures, which have cross-sensitivities (e.g., a 10-year bond has some sensitivity to the 7-year rate). The matrix $\mathbf{H}$ is not diagonal. We solve for $\mathbf{n}$ using standard inversion:

$$\boxed{\mathbf{n} = -\mathbf{H}^{-1}\mathbf{k}}$$

This "unwinds" the cross-correlations, telling you, for example, to short slightly less of the 10-year because your 7-year hedge already covers some of that risk.

### 14.5.3 When You Can’t Solve Exactly: Weighted Least Squares

Often you have **more buckets than hedge instruments** ($K\gt L$). Then you cannot set every bucket to zero; you choose an *approximate* hedge by prioritizing buckets and penalizing expensive instruments.

One generic formulation is a weighted least squares (with penalty) problem:

$$\widehat{\mathbf{p}}=\underset{\mathbf{p}}{\mathrm{argmin}}\left(\sum_{k=1}^{K} W_{k}^{2}\left(\partial_{k} H_{0}(\mathbf{p})-\partial_{k} V_{0}\right)^{2}+\sum_{l=1}^{L} U_{l}^{2} p_{l}^{2}\right)$$

where $W_k$ weights the importance of offsetting the $k$-th bucket and $U_l$ penalizes use of expensive or illiquid hedging instruments. This is the rates analogue of ridge regression: you trade off residual risk versus instrument usage.

The simple case with $L = K$ and invertible Jacobian yields:

$$\mathbf{p}=\left(\partial \mathbf{H}^{\top}\right)^{-1}\partial V_0$$

which reduces to the exact solution when the system is square and invertible.

### 14.5.4 Liquidity Constraints and the Art of Approximate Hedging

The mathematically optimal hedge $\mathbf{n} = -\mathbf{H}^{-1}\mathbf{k}$ is rarely the hedge you actually trade. Real-world constraints intervene:

**Why the "optimal" hedge isn't traded:**

1. **Liquidity:** Liquidity is concentrated in a small set of benchmark tenors; intermediate/off-the-run maturities can be much less liquid.

2. **Transaction costs:** Each hedge instrument incurs bid-ask spread and market impact. The "optimal" 12-instrument hedge may cost more in execution than it saves in risk reduction.

3. **Balance sheet:** Banks face leverage ratio and RWA constraints. Holding 20 hedge instruments may be capital-inefficient.

4. **Operational complexity:** More positions mean more settlements, more financing, more fails risk.

**The practical approach: hedging with liquid benchmarks only**

In practice, hedging is often restricted to a small set of liquid instruments. This means $L$ can be much smaller than $K$ (many risk buckets, few hedges).

The result: **residual risk at non-hedged tenors**.

> **Desk Reality:** When $K$ (buckets) exceeds $L$ (tradable hedges), the hedge is necessarily approximate.
> **Common break:** Mapping intermediate buckets to liquid tenors embeds an interpolation assumption; when reality deviates, you get residual P&L.
> **What to check:** Look at the *post-hedge* $KR01$ vector and track the largest residual buckets alongside realized key-rate moves.

**Worked Example: Hedging a 15-Year Swap with Only 10y and 30y Bonds**

Your portfolio has KRDV01 of USD 5,000/bp concentrated at the 15-year point. Liquid hedging instruments are only 10y and 30y.

**Option 1: Nearest neighbor**
- 15y is closer to 10y than 30y (5 years vs 15 years)
- Hedge entirely with 10y bond
- Residual risk: if 15y moves differently than 10y, P&L breaks

**Option 2: Linear interpolation**
- 15y is 25% of the way from 10y to 30y
- Allocate: 75% to 10y hedge, 25% to 30y hedge
- 10y hedge: USD 3,750/bp; 30y hedge: USD 1,250/bp
- Residual: depends on how well 15y interpolates between 10y and 30y

**Option 3: Regression-based**
- Historical regression: $\Delta y_{15} = 0.82 \Delta y_{10} + 0.23 \Delta y_{30}$ (hypothetical)
- Allocate: 82% to 10y, 23% to 30y
- Residual: the regression standard error × notional

In practice, a desk may use some version of Option 2 or 3, accepting that the hedge is imperfect.

---

## 14.6 Practical Implementation

Implementing KRDV01 in a production risk engine requires navigating several "gotchas."

### 14.6.1 Forward-Curve Pathologies from Par-Yield Bumps

The bump design is not harmless. A simple way to compute deltas with respect to benchmark instruments is: apply a **manual bump** to one benchmark quote $V_i$, **reconstruct the yield curve**, and then **reprice** the portfolio $V_0$. This procedure is sometimes known as the **par-point approach**, and resulting derivatives **par-point deltas**.

For the par-point approach to work well, it is important that the curve construction algorithm produces **clean, local perturbations** when benchmark prices are shifted.

One practical variant is the **cumulative par-point** method. The implied forward-curve shifts are less extreme than those of the ordinary par-point method, and the sum of deltas computed by the cumulative method is (by definition) exactly equal to the **parallel** delta (shifting all benchmark yields by the same amount at the same time). For the standard par-point method, this “sum of deltas equals parallel delta” property holds only in the limit of vanishing shifts, not necessarily for the finite bumps used in practice.

**Consequence:** For portfolios sensitive to forward-curve shape, par-yield bumps can make “first-order” deltas noisy and unstable.

**Mitigations (methodology choices):**
1. **Zero-rate or forward-rate bucket bumps:** Work directly with a forward curve $f(t)$ (or spot/zero curve) so locality is controlled on the object that drives cashflow discounting/projection.
2. **Cumulative par-point bumps:** Retain earlier shocks as you move along the curve; the resulting shift roughly corresponds to a piecewise-flat forward-curve move between successive benchmarks.

### 14.6.2 Choosing Key Rates

Key rates are usually defined as par yields or zero (spot) rates. A practical heuristic: pick key rates that match **your hedge instruments** and the **liquid points** on your curve.

The number of key rates is a tradeoff:
- **More keys:** Greater precision, but higher dimensionality and potential noise.
- **Fewer keys:** Simpler, but may miss local risk.

For front-end hedging, it is common to use **quarterly buckets** for the first several years (because listed futures/FRAs-like instruments provide natural hedges), and then use fewer, wider buckets further out the curve.

### 14.6.3 Convexity and Basis Risk

Even a KRDV01-neutral portfolio is not risk-free.

1. **Convexity:** KRDV01 is a first-order measure. Large rate moves (e.g., ±50 bps) will generate P&L due to convexity (gamma). Chapter 13 develops this second-order risk measure.

2. **Basis Risk:** Hedging a corporate bond portfolio with Treasury key rates ignores the credit spread. If spreads widen while Treasuries rally, your "perfect" hedge effectively doubles up your loss.

3. **Hedge Stability:** Your hedge works “as designed” only if intermediate points on the curve move in the way your bump design/interpolation assumes. When they don’t, you realize residual curve-shape P&L.

### 14.6.4 P&L Attribution Workflow and Unexplained Residuals

One powerful application of KRDV01 is **P&L attribution**. Given daily changes in key rates $\Delta \mathbf{r}$, the explained P&L is:

$$PL_{rates} = -\sum_k KR01_k \times \Delta r_k$$

The **unexplained residual** captures basis, credit, and model error:

$$Residual = Actual\ PL - PL_{rates}$$

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
> - $PL_{rates} = -\sum_k KR01_k \times \Delta r_k$
> - Include second-order (convexity) term for large moves: $\frac{1}{2}\sum_k \gamma_k (\Delta r_k)^2$
>
> **Step 4: Compare to actual P&L**
> - Actual P&L comes from front-office systems (mark-to-market)
> - Residual = Actual − Explained
>
> **Step 5: Investigate significant residuals**
> - Tolerance is desk- and product-dependent (often set as a fraction of explained P&L and/or a fixed dollar threshold)
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

| Tenor | KR01 (USD/bp) | Rate Δ (bp) | Explained P&L |
|:---:|:---:|:---:|:---:|
| 2y | +USD 3,000 | -2 | +USD 6,000 |
| 5y | +USD 8,000 | +1 | -USD 8,000 |
| 10y | -USD 5,000 | +3 | +USD 15,000 |
| **Total** | | | **+USD 13,000** |

**Actual P&L:** +USD 10,000
**Residual:** -USD 3,000 (unexplained loss)

**Investigation:** The USD 3,000 residual is 23% of explained P&L—material relative to the explained move. Possible causes:
- Credit spread on corporate holdings widened 2bp (spread duration was USD 1,500/bp, implying USD 3,000 loss)
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
> - Accept residual risk but size it explicitly: "We have USD 500/bp unhedged at 7y"
> - Use futures strips to fill in the gaps (more granular than bonds)

**Visual: Residual KRDV01 Profile After Hedging**

```
Before Hedging:
Bucket DV01:  [+800  +1200  +1500  +1400  +1100  +900  +700  +400]
Tenor:         2y     3y     5y     7y     10y   15y   20y   30y

After Hedging with 2y, 5y, 10y, 30y only:
Residual:     [  0   +300   +100   +600    0    +400  +200    0 ]

→ The 7y bucket still has +USD 600/bp exposure despite "hedging"
→ The 15y and 20y buckets have residual because no direct hedge exists
```

---

## 14.7 Connection to Multi-Factor Models and PCA

Key-rate DV01 and bucket exposures are a desk-friendly discretization of curve risk: you choose a set of shocks and report the PV sensitivity to each.

Many approaches (both model-based and empirical) describe yield curve changes with a small number of factors that look like “level / slope / curvature”. Principal Component Analysis (PCA) is one way to estimate such factors from historical rate changes. Chapter 16 focuses on that statistical view; this chapter focuses on mechanics and hedgeability.

**The Chapter 14 / Chapter 16 Boundary:**

| This Chapter (14) | Chapter 16 |
|:---|:---|
| **Mechanics** of key rates and buckets | **Statistical** analysis via PCA |
| How to compute KRDV01 | Why level/slope/curvature explain most variance |
| How to set up the hedge matrix | How to use regression for hedge ratios |
| Trade vocabulary (steepener, butterfly) | Butterfly trade case studies and relative value |
| Liquidity-constrained bucket hedging | PCA eigenvalues and factor loadings |

**Key insight:** KRDV01/buckets give you tradable, reportable risk numbers; factor models describe co-movement and can improve hedge design, but they do not replace the need to specify the bump object in daily risk.

---

## Summary

1. **Parallel Limit:** DV01 summarizes risk into a single number, effectively assuming perfect correlation across the curve. It hides exposure to twists and butterflies.

2. **Key Rates (KRDV01):** Decompose risk into a vector of $KR01$ sensitivities across key tenors. Under a consistent bump design, $\sum_k KR01_k$ is close to parallel DV01.

3. **Trade Vocabulary:** Steepeners, flatteners, and butterflies are specific KRDV01 profiles. Understanding this vocabulary is essential for trading desk communication.

4. **Buckets:** Measure sensitivity to forward-rate segments. They map naturally to hedges that target specific accrual periods (e.g., quarterly rate futures).

5. **Zero ≠ Safe:** A DV01-neutral portfolio can lose money if the curve twists. Only a KRDV01-neutral portfolio is immune to shape changes (at the resolution of the chosen keys).

6. **Linear Hedging:** Multi-curve hedging is a matrix inversion problem ($\mathbf{n} = -\mathbf{H}^{-1}\mathbf{k}$). Par-bond hedges simplify this to a diagonal system.

7. **Liquidity Constraints:** Real hedges use only liquid benchmarks, leaving residual “pin risk” at unhedged tenors.

8. **P&L Attribution:** KRDV01 enables daily P&L explain. Large unexplained residuals signal basis risk, credit moves, or model error.

9. **Implementation:** Watch out for "bizarre" forward curves from par-yield bumps; prefer zero/forward bumps for complex portfolios. Choose key rates that match your hedging instruments.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|:---|:---|:---|
| **Parallel DV01** | Value change for uniform 1 bp shift | Quick summary; fails for curve twists |
| **Key-Rate Shift** | A localized curve shock tied to key tenor $T_k$ | Makes “which tenor?” risk explicit |
| **$KR01_k$** | $PV(\text{key }k\text{ down }1\text{bp})-PV(\text{base})$ | Core bucket risk number (units + sign) |
| **Sum-to-Parallel (Checksum)** | $\sum_k KR01_k \approx DV01_{\parallel}$ under consistent bumps | Catches missing buckets / mismatched definitions |
| **Partial Duration** | Duration with respect to a single curve point/bucket | Duration-form view of key-rate risk |
| **Forward Bucket** | A forward-rate segment $[T_j,T_{j+1}]$ | Natural unit for front-end hedging |
| **$FD01_j$** | $PV(\text{bucket }j\text{ down }1\text{bp})-PV(\text{base})$ | Bucket DV01 for forwards |
| **Forward-Rate Delta** | Directional derivative under shaped forward-curve shocks | A continuous-time view of bucketing |
| **Twist/Steepener** | Non-parallel curve movement | The main risk missed by parallel DV01 |
| **Butterfly** | Long wings, short belly (or reverse) | Isolates curvature while hedging level/slope |
| **Hedge Matrix $\mathbf{H}$** | Columns are hedge-instrument bucket vectors | Turns hedging into linear algebra |
| **Pack/Bundle (Aggregation)** | Grouping of consecutive quarterly buckets | Reduces reporting dimension on desks |
| **Pin Risk** | Residual exposure at unhedged tenors | Why discrete hedges leave gaps |

---

## Notation

| Symbol | Definition |
|:---|:---|
| $DV01$ | Dollar value of 01 (parallel shift sensitivity) |
| $KR01_k$ | Key-rate DV01 at key $k$ |
| $FD01_j$ | Forward-bucket DV01 for segment $j$ |
| $\Delta r_k$ | Key-rate move at key $k$ | 
| $\mathbf{k}$ | Vector of portfolio key-rate exposures |
| $\mathbf{H}$ | Matrix of hedge instrument key-rate sensitivities |
| $\mathbf{n}$ | Vector of hedge notionals |
| $D_i$ | Partial duration at maturity/bucket $i$ |
| $\partial_k V_0$ | Forward-rate delta (directional derivative) |
| $W_k$ | Bucket importance weight (hedging objective) |
| $U_l$ | Hedge-instrument penalty weight (cost/liquidity) |

---

## Flashcards

| # | Question | Answer |
|:---|:---|:---|
| 1 | What is the sum-to-parallel property of key rates? | The sum of all key-rate partial DV01s equals the parallel DV01 (approximately). |
| 2 | Why is a 10-year bond exposed to the 5-year key rate? | Its intermediate coupons are discounted by rates in the 5-year region. |
| 3 | Contrast the $KR01$ profile of a bond vs. a swap. | A bond’s profile reflects coupon discounting; a swap’s profile depends on bump design but often has larger weight in longer buckets where the fixed-leg annuity ends. |
| 4 | How do you hedge a forward-bucket exposure of $-USD 5{,}000/\text{bp}$ with quarterly rate futures? | Roughly $5{,}000/25=200$ contracts if one contract is about $USD 25/\text{bp}$; choose buy/sell direction to offset the sign under your system’s convention. |
| 5 | Where does the “$USD 25$ per bp per contract” scale come from? | It’s notional × accrual × 1bp: e.g., $1{,}000{,}000\times 0.25\times 10^{-4}=USD 25$. |
| 6 | A portfolio has DV01=0 but loses money when the curve steepens. What is this? | Curve risk (or twist/shape risk). |
| 7 | What is the hedge solution formula for the linear system? | $\mathbf{n} = -\mathbf{H}^{-1}\mathbf{k}$. |
| 8 | Why can “bump one par yield” create unstable deltas? | Holding neighboring par yields fixed can force large implied forward-rate moves, breaking the “small shock” intuition behind first-order sensitivities. |
| 9 | What is the hedge matrix $\mathbf{H}$? | A matrix whose columns are hedge-instrument bucket vectors (e.g., $KR01$ vectors per unit notional). |
| 10 | What is "Basis Risk" in key-rate hedging? | Risk that the spread between instrument (e.g., Corp) and hedge (e.g., Treasury) changes. |
| 11 | Does KRDV01 capture convexity? | No, it is a first-order (linear) measure only. |
| 12 | What is a partial duration? | A duration-like sensitivity to a move at a specific point/bucket on the curve, holding other points fixed (per the bump design). |
| 13 | What do “level / slope / curvature” factors mean? | They are common empirical shapes for yield-curve moves; PCA is a way to estimate them from historical data. |
| 14 | When is the hedge matrix $\mathbf{H}$ diagonal? | When hedging with par bonds at exactly the key rate maturities. |
| 15 | What is the cumulative par-point approach? | A method where each key rate shift is retained when computing subsequent deltas. |
| 16 | How does forward rate delta differ from KRDV01? | Forward rate delta uses forward curve bumps; KRDV01 typically uses par yield bumps. |
| 17 | What is “gap/bucket” interest-rate risk in ALM? | A bucketed measure of how asset and liability values (or cashflows) respond to rate changes in different maturity segments. |
| 18 | Can a portfolio be DV01-neutral and still lose money? | Yes—if it has non-zero KRDV01 exposures and the curve twists. |
| 19 | What is a "steepener" trade in KRDV01 terms? | Long front-end, short back-end (e.g., long 2y, short 10y). |
| 20 | What is a “pack” in quarterly bucket language? | A grouping of consecutive quarterly buckets (often 4 contracts ≈ 1 year of forward exposure); naming conventions vary by venue. |

---

## Mini Problem Set

1. (Compute) A 3-year zero-coupon bond is priced on a flat 5% curve (continuously compounded). Compute its parallel DV01 per USD 100 face.
2. (Compute) You are long a 2y position with $KR01_{2y}=+USD 1{,}800/\text{bp}$ and short a 10y position with $KR01_{10y}=-USD 1{,}800/\text{bp}$. (a) If 2y rates fall 10bp and 10y rates rise 10bp, what is the first-order P&L? (b) Name the trade.
3. (Compute) Your risk report shows a forward-bucket exposure $FD01=+USD 1{,}250/\text{bp}$ to a quarterly bucket. If a futures contract provides about $USD 25/\text{bp}$ per contract, how many contracts hedge the exposure and in what direction?
4. (Compute) You have $\mathbf{k}=[100,200]^\top$ (in USD /bp). Two hedge instruments have bucket vectors $[10,2]^\top$ and $[5,8]^\top$ (in USD /bp per unit). (a) Write $\mathbf{H}\mathbf{n}=-\mathbf{k}$. (b) Solve for $\mathbf{n}$.
5. (Compute) Using Table 14.1 as partial durations, estimate the P&L for a USD 10 million portfolio if rates move: 1y +5bp, 2y +4bp, 3y +3bp, 4y +2bp, 5y +1bp, 7y −1bp, 10y −3bp.
6. (Concept) Why can bumping a single long-dated par swap rate by 1bp (holding neighbors fixed) imply large forward-rate moves? Why is this problematic for “first-order” sensitivities?
7. (Compute) You want a 2s–10s steepener with $USD 50{,}000/\text{bp}$ of risk. The 2y bond has DV01 of $USD 1{,}900$ per $USD 1$mm face and the 10y bond has DV01 of $USD 8{,}200$ per $USD 1$mm face. Compute the notionals (long 2y, short 10y) and verify net parallel DV01 $\approx 0$.
8. (Compute/Desk) A portfolio has $KR01_{2y}=+2{,}000$ USD/bp, $KR01_{5y}=+4{,}000$ USD/bp, $KR01_{10y}=-3{,}000$ USD/bp and the day’s moves are $-3$bp, $+2$bp, $+5$bp respectively. (a) Compute explained rates P&L. (b) If actual P&L is +USD 15,000, what is residual? (c) List two possible causes.
9. (Desk) You have key-rate exposures $[+1000,+2000,+1500,+500]$ at 2y/5y/10y/20y, but you can only hedge with 2y and 10y instruments. Propose a simple mapping and describe the residual risk.
10. (Concept) Name the trade for each KR01 vector: (a) $[+100, -200, +100]$ at 2y/5y/10y, (b) $[0, +500, 0]$ at 2y/5y/10y, (c) $[-300, +300, 0]$ at 2y/10y/30y.
11. (Desk) A portfolio has $[KR01_{2y},KR01_{5y},KR01_{30y}] = [+20\text{k},0,-20\text{k}]$ in USD/bp. Yesterday, 2y fell 1bp, 5y fell 3bp, 30y fell 4bp. Explain the sign of the P&L and compute the first-order rates P&L.
12. (Compute) After hedging a 10y swap with only 5y and 10y bonds, your residual KR01s are +USD 800/bp at 7y and +USD 400/bp at 8y. If 5y and 10y both rise 5bp but 7y rises 10bp and 8y rises 8bp, estimate the unexplained P&L.

### Solution Sketches (Selected)
1. Price $=100e^{-0.05\cdot 3}=86.07$. $DV01\approx 3\times 86.07\times 10^{-4}=0.0258$ per 100.
2. $\Delta PV\approx -(1800)(-10)-(-1800)(+10)=+USD 36{,}000$. This is a steepener (long front-end, short back-end).
3. If one contract is approximately USD 25/bp, then $1{,}250/25=50$ contracts. Since $FD01\gt 0$ (you lose when rates rise), the hedge needs -USD 1,250/bp (typically a short position).
4. $\mathbf{H}=[[10,5],[2,8]]$. $\mathbf{n}=-\mathbf{H}^{-1}\mathbf{k}=[2.86,-25.71]^\top$.
5. Using $\Delta P/P\approx-\sum_i D_i\Delta y_i$ with $\Delta y$ in decimals gives total $\approx 0.00224$. P&L is approximately -USD 22,400 on USD 10 million notional.
11. First-order P&L $= -[(+20{,}000)(-1)+(0)(-3)+(-20{,}000)(-4)] = -60{,}000\ \text{USD}$. DV01 neutrality does not help when the 30y move is larger than the 2y move (a flattening rally).

---

## References

- (Tuckman & Serrat, *Fixed Income Securities: Tools for Today’s Markets*, “Key Rate Shifts”; “Key Rate 01s and Key Rate Durations”; “Choosing Key Rates”)
- (Hull, *Risk Management and Financial Institutions*, “Interest Rate Deltas in Practice” (partial durations and bucketing approach))
- (Hull, *Options, Futures, and Other Derivatives*, “Hedging Interest Rate Derivatives” (multiple delta/bucket definitions))
- (Andersen & Piterbarg, *Interest Rate Modeling*, “Managing Yield Curve Risk”; “Par-Point Approach”)
- (Pachamanova & Fabozzi, *Simulation and Optimization in Finance: Modeling with MATLAB, @RISK, or VBA*, “Key Rate Duration” definition/interpretation)
- (Elton, Gruber, Brown, & Goetzmann, *Modern Portfolio Theory and Investment Analysis*, “Multiple duration measures” (nonparallel shifts via key rates))
