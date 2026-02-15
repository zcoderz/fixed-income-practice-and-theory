# Chapter 12: Duration — Macaulay, Modified, and the Connection to DV01

---

## Introduction

Your risk report arrives. The portfolio shows "Duration: 7.2 years." Your boss asks, "What does that mean for our P&L if rates move 25 basis points?" If you cannot answer this question instantly and correctly, you will struggle to bridge from middle office to the trading desk.

Duration is the language of interest rate risk. When a portfolio manager says "I'm running 7 years of duration," they mean their position will gain (or lose) approximately 7% for every 100 basis points rates move. When a trader says they're "long 50k DV01 in 10-year equivalents," they're expressing risk in duration-scaled units. When the risk committee reviews interest rate sensitivity, they're looking at duration metrics. Understanding duration deeply—not just as a formula but as an intuition—is essential for anyone managing fixed income portfolios.

The stakes are high. A 10 basis point move on a \$100 million portfolio with 7-year duration produces approximately \$700,000 of P&L. Misunderstand duration by just one year, and your hedges are wrong by ~14%. This shows up as unexplained P&L, limit breaches, and bad hedge ratios.

This chapter builds the machinery behind the duration number on a risk report. We start with **Macaulay duration** as a present-value-weighted average time of cashflows. We then derive **modified duration** as the first-order price sensitivity to the bond’s yield (with a stated compounding basis). We translate that sensitivity into **dollar duration** and **DV01**, explain which quantities aggregate cleanly across positions, and show how duration matching appears in simple immunization and duration-based VaR approximations. Throughout, we do explicit unit/sign checks and state *what is being bumped*.

Prerequisites: [Chapter 5 — Fixed-Rate Bond Pricing](chapters/chapter_05_fixed_rate_bond_pricing.md), [Chapter 11 — DV01/PV01 — Definitions, Computation, and “What’s Being Bumped”](chapters/chapter_11_dv01_pv01_definitions_computation.md)  
Follow-on: [Chapter 13 — Convexity — Second-Order P&L and When DV01 Breaks](chapters/chapter_13_convexity.md), [Chapter 14 — Key-Rate DV01 and Bucket Exposures](chapters/chapter_14_key_rate_dv01_bucket_exposures.md), [Chapter 15 — DV01 Hedging — Hedge Ratios, Risk Weights, and Practical Caveats](chapters/chapter_15_dv01_hedging.md)

## Learning Objectives
- Translate a bond’s quote into a yield-based pricing function $P(y)$ and interpret what “duration” is differentiating.
- Compute and interpret Macaulay duration (time measure) and modified duration (price sensitivity) with explicit units.
- Convert modified duration to dollar duration and DV01, stating the bump object, bump size, and sign convention.
- Aggregate risk across positions correctly (when durations add, when they don’t).
- Explain where yield-based duration breaks down (curve moves, optionality, convexity).

---

## 12.1 The Yield-Based Bond Price Function

### 12.1.1 Setting Up the Framework

To understand duration, we must view the bond's price not just as a number but as a mathematical function of its yield-to-maturity $y$. For a bond with annual coupon rate $c$ (per 100 face) and maturity $T$ years, priced using the standard semiannual compounding convention, the price $P(y)$ is:

$$\boxed{P(y) = \sum_{t=1}^{2T} \frac{c/2}{(1+y/2)^t} + \frac{100}{(1+y/2)^{2T}}}$$

This yield-based framework treats the bond’s yield-to-maturity $y$ as the single scalar risk factor. We are asking: *if the bond’s yield changes by $\Delta y$, how does $P$ change?*

### 12.1.2 Why This Matters for Risk

When a trader asks "What is the duration of this bond?", they are asking about the slope of this $P(y)$ function. Risk measures are essentially derivatives—they quantify how much the price changes for an infinitesimal change in the underlying variable.

Two related (but different) sensitivities are often called “duration”:

**Yield-based duration (Macaulay/modified).** You bump the bond’s own yield-to-maturity $y$, assume the promised cashflows are fixed, and measure $\partial P/\partial y$.

**Curve-based (effective) duration.** You bump the benchmark discount curve (often a small parallel shift, or a set of key-rate shifts) and reprice. This is the right framing when you care about curve moves directly or when expected cashflows can change with rates (callables, MBS).

> **Desk Reality:** Risk systems often label different objects as “duration.”  
> **Common break:** Comparing yield-duration to effective/OAS duration (or key-rate duration) produces wrong hedge ratios.  
> **What to check:** (i) bump object (bond YTM vs curve), (ii) bump size (e.g., 1bp or 10bp), (iii) whether cashflows were held fixed.

---

## 12.2 Macaulay Duration: The Original Concept

### 12.2.1 Duration as Weighted-Average Time

Macaulay duration (introduced by Frederick Macaulay, 1938) is the present-value-weighted average time to receipt of cashflows. If a bond pays cashflows $c_i$ at times $t_i$ with present values $v_i$, and the **full/dirty price** is $P_{\text{dirty}}=\sum_i v_i$, then:

$$\boxed{D_{\text{Mac}} = \frac{\sum_{i=1}^{n} t_i\,v_i}{P_{\text{dirty}}} = \sum_{i=1}^{n} t_i\left(\frac{v_i}{P_{\text{dirty}}}\right)}$$

For a level-coupon bond priced off a semiannually-compounded yield $y$ (coupon rate $c$ per 100 notional, maturity $T$ years), an explicit form is:

$$\boxed{D_{\text{Mac}} = \frac{1}{P}\left[\sum_{t=1}^{2T} \left(\frac{t}{2}\right) \frac{c/2}{(1+y/2)^t} + T \cdot \frac{100}{(1+y/2)^{2T}}\right]}$$

Each term represents:

$$ \text{Weight} \times \text{Time} = \frac{\text{PV of Cash Flow}}{\text{Total Price}} \times \text{Time of Cash Flow} $$

In plain language, Macaulay duration is between the first and last cashflow times. For any coupon-bearing bond, $D_{\text{Mac}} < T$ because some PV weight is pulled earlier by coupon payments.

> **Analogy: The Center of Gravity**
>
> Imagine the bond's cash flows as weights placed on a timeline. Each coupon is a small weight placed at its payment date. The principal is a large weight at maturity.
>
> *Macaulay duration is where you would place the fulcrum to balance this timeline.*
>
> Early coupons pull the center of gravity leftward, making duration shorter than maturity. Only a zero-coupon bond (one large weight at the end) has Duration = Maturity.

### 12.2.2 The Replicating Portfolio Interpretation

One useful way to build intuition is to view a coupon bond as a portfolio of zero-coupon bonds—one zero for each promised cashflow. A zero’s Macaulay duration equals its maturity, so the coupon bond’s Macaulay duration is the **value-weighted average maturity of the replicating zeros**.

This perspective makes duration intuitive: a 10-year bond with a 5% coupon has a duration of about 8 years because, in present value terms, roughly 80% of the bond's value comes from the final principal payment (which has duration 10 years), while the remaining 20% comes from earlier coupon payments (which have shorter durations).

### 12.2.3 Worked Example A: Computing Macaulay Duration from Cash Flows

Consider the following example: a 3-year bond with \$100 face, 10% coupon paid semiannually, and yield of 12% with continuous compounding.

**Step 1: Lay out the cash flows**

The bond pays \$5 every 6 months plus \$105 at maturity. We discount each using $e^{-0.12 \times t}$.

**Step 2: Calculate present values and weights**

| Time (years) | Cash Flow ($) | Present Value | Weight (PV/B) | Time × Weight |
|:-------------|:--------------|:--------------|:--------------|:--------------|
| 0.5 | 5 | 4.709 | 0.050 | 0.025 |
| 1.0 | 5 | 4.435 | 0.047 | 0.047 |
| 1.5 | 5 | 4.176 | 0.044 | 0.066 |
| 2.0 | 5 | 3.933 | 0.042 | 0.083 |
| 2.5 | 5 | 3.704 | 0.039 | 0.098 |
| 3.0 | 105 | 73.256 | 0.778 | 2.333 |
| **Total** | | **94.213** | **1.000** | **2.653** |

**Result:** The Macaulay Duration is **2.653 years**.

**Sanity Check:** The duration is less than the 3-year maturity (as expected for a coupon bond). The 78% weight of the final principal repayment dominates, but early coupons pull the average time down.

---

## 12.3 Special Cases: Zeros, Par Bonds, and Perpetuities

### 12.3.1 Zero-Coupon Bonds: Duration Equals Maturity

For a zero-coupon bond, there is only one cash flow: the principal at time $T$. Since this single payment represents 100% of the bond's present value, its weight is 1.00. Formally:

$$\boxed{D_{\text{Mac}}\big|_{c=0} = T}$$

For a zero-coupon bond, Macaulay duration equals time to maturity: a 6‑month zero has duration $0.5$ years; a 10‑year zero has duration $10$ years.

This is why practitioners quote duration in “years”: a coupon bond with Macaulay duration $D$ has (to first order) the same *percentage* price sensitivity as a zero-coupon bond maturing in $D$ years.

The zero-coupon case also yields modified duration and DV01 for zeros:

$$\boxed{D_{\text{Mod}}\big|_{c=0} = \frac{T}{1+y/2}}$$

$$\text{DV01}\big|_{c=0} = \frac{T}{100(1+y/2)^{2T+1}}$$

**Example:** A 10-year zero at 5% yield has modified duration of \$10/(1.025) = 9.76$ years.

### 12.3.2 Par Bonds: The Closed-Form Formula

For a bond selling at par ($P=100$ and coupon rate $c=y$ under the same compounding basis), the following closed forms are convenient:

$$\boxed{D_{\text{Mod}}\big|_{P=100} = \frac{1}{y}\left(1 - \frac{1}{(1+y/2)^{2T}}\right)}$$

$$\boxed{D_{\text{Mac}}\big|_{P=100} = \frac{1+y/2}{y}\left(1 - \frac{1}{(1+y/2)^{2T}}\right)}$$

These formulas are useful because par bonds have a fixed price of 100, allowing us to isolate how duration varies with maturity and yield.

**Example:** A 10-year par bond at 5% yield:

$$D_{\text{Mod}} = \frac{1}{0.05}\left(1 - \frac{1}{(1.025)^{20}}\right) = 20 \times (1 - 0.6103) = 7.79 \text{ years}$$

### 12.3.3 Perpetuities: The Upper Bound

A **perpetuity** pays coupons forever ($T=\infty$). Taking the maturity $\to\infty$ limit gives:

$$\boxed{D_{\text{Mod}}\big|_{T=\infty} = \frac{1}{y}}$$

$$\boxed{D_{\text{Mac}}\big|_{T=\infty} = \frac{1+y/2}{y}}$$

At a yield of 5%, the Macaulay duration of a perpetuity is $(1.025)/0.05 = 20.5$ years.

For par coupon bonds, modified and Macaulay duration increase with maturity and approach the corresponding perpetuity limit as maturity becomes very long.

> **Desk Reality:** The “perpetuity duration” is a useful back-of-the-envelope bound.  
> **Common break:** Treating it as a hard cap without checking yield level and coupon/price regime.  
> **What to check:** At yield $y$ (semiannual), a perpetuity has $D_{\text{Mod}}\approx 1/y$ and $D_{\text{Mac}}\approx (1+y/2)/y$. Coupon bonds near par with long maturities tend to approach this benchmark; zeros and deep-discount bonds can behave differently.

---

## 12.4 Modified Duration: The Risk Measure

### 12.4.1 From Time to Sensitivity

While "weighted average time" is an interesting statistic, traders are paid to manage price risk. The reason duration is ubiquitous is that this time measure is directly linked to the bond's price derivative.

By differentiating the bond price equation with respect to yield, we get a direct link between Macaulay duration and first-order price sensitivity. This motivates **modified duration**.

For continuous compounding, the (Macaulay) duration satisfies the familiar semi-elasticity approximation:

$$\frac{\Delta B}{B} \approx -D \Delta y$$

If yields are expressed with compounding frequency $m$ times per year, the chain rule introduces an adjustment:

$$\Delta B \approx -\frac{BD\Delta y}{1+y/m}$$

This motivates defining **modified duration** as:

$$\boxed{D_{\text{Mod}} = \frac{D_{\text{Mac}}}{1+y/m}}$$

For U.S. Treasury bonds with semiannual compounding ($m=2$):

$$\boxed{D_{\text{Mod}} = \frac{D_{\text{Mac}}}{1+y/2}}$$

Using modified duration:

$$\boxed{\frac{1}{P}\frac{dP}{dy} = -D_{\text{Mod}}}$$

Or in the approximation form used on trading desks:

$$\frac{\Delta P}{P} \approx -D_{\text{Mod}} \times \Delta y$$

Interpretation: $D_{\text{Mod}}$ is the approximate percentage change in price for a **100bp (1.00%)** change in yield (holding cashflows fixed and using the stated yield/compounding convention).

**Check (bp-to-decimal and dollars):** $25\text{bp}=0.0025$. If a bond has $D_{\text{Mod}}=7.8$ and is priced at 100 (per 100 face), the first-order estimate is $\Delta P/P\approx -7.8\times 0.0025=-1.95\%$, i.e. $\Delta P\approx -1.95$ price points. On $N=\$10\text{mm}$ face, 1 price point is $\$100{,}000$, so this is about $-\$195{,}000$.

### 12.4.2 The Derivation

The factor $(1+y/m)$ comes from the chain rule when differentiating the discount factor.

Starting from the price function:

$$P(y) = \sum_{t=1}^{2T} \frac{c/2}{(1+y/2)^t} + \frac{100}{(1+y/2)^{2T}}$$

Differentiating with respect to $y$:

$$\frac{dP}{dy} = -\frac{1}{1+y/2}\left[\sum_{t=1}^{2T} \frac{t}{2} \frac{c/2}{(1+y/2)^t} + T \frac{100}{(1+y/2)^{2T}}\right]$$

The term inside the brackets is exactly $P \times D_{\text{Mac}}$. Therefore:

$$\frac{dP}{dy} = -\frac{P \cdot D_{\text{Mac}}}{1+y/2}$$

Dividing by $P$:

$$\frac{1}{P}\frac{dP}{dy} = -\frac{D_{\text{Mac}}}{1+y/2} = -D_{\text{Mod}}$$

This gives the key sensitivity identity:

$$\frac{dP}{d\lambda} = -D_M P$$

where $D_M$ is the modified duration (for the chosen compounding basis).

### 12.4.3 Worked Example B: Using Modified Duration for Estimation

Using the bond from Example A (price 94.213, Macaulay duration 2.653 years, yield 12% continuous):

For semiannual compounding, the equivalent yield is 12.3673%. The modified duration is:

$$D_{\text{Mod}} = \frac{2.653}{1 + 0.123673/2} = 2.499$$

**Scenario**: Yields rise by 10 basis points (+0.10%). What is the estimated price change?

Using the duration relationship:

$$\Delta B \approx -94.213 \times 2.499 \times 0.001 = -0.236$$

The predicted new price is \$94.213 - 0.236 = 93.977$.

An exact repricing at the bumped yield gives a price of about $93.963$, very close to the duration estimate.

The small error (0.014 cents) is due to **convexity** (the curvature of the price-yield relationship), which we cover in Chapter 13.

> **Rule of Thumb: Duration ≈ Leverage**
>
> - **Duration 10**: If rates move 1%, your price moves 10%. This is like 10× leverage on rates.
> - **Duration 2**: If rates move 1%, your price moves 2%. This is like 2× leverage.
>
> Risk managers use this to normalize positions. A trader who buys \$10mm of 2-year notes (Duration ≈ 2) takes the same rate risk as buying \$2mm of 10-year bonds (Duration ≈ 8).

---

## 12.5 How Duration Varies: Yield, Coupon, and Maturity

### 12.5.1 Duration and Yield

All else equal, higher yields tend to **lower** duration. Intuition: increasing the discount rate reduces the present value of all cashflows, but it reduces *distant* cashflows more, shifting PV weight toward earlier payments. Since duration is a PV-weighted average time, shifting weight earlier pulls duration down.

For example, consider a 5-year 5.625% bond:

| Yield | Duration | Longest CF Weight |
|:------|:---------|:------------------|
| 7% | 4.26 | 77.3% |
| 3% | 4.40 | 79.0% |

At lower yields, longer-dated payments become relatively more valuable, pulling duration up.

### 12.5.2 Duration and Coupon

For a fixed maturity and yield, higher coupons tend to **lower** duration. A higher coupon means more PV comes from earlier cashflows (coupons), so the PV-weighted “center of gravity” moves toward the present. Zeros (no early coupons) therefore have the highest duration for a given maturity.

The table below shows this pattern at 5% yield:

| Maturity | 1% Coupon Duration | 10% Coupon Duration |
|:---------|:-------------------|:--------------------|
| 10 years | 9.42 | 7.11 |
| 25 years | 20.16 | 12.75 |
| 50 years | 26.67 | 17.38 |

The 1% coupon bonds have substantially higher durations at every maturity.

### 12.5.3 Duration and Maturity: The Deep Discount Paradox

For par and premium bonds, duration increases monotonically with maturity, approaching the perpetuity duration as an upper limit.

There is a less intuitive corner case for very low-coupon, very long-maturity **discount** bonds: as maturity increases, Macaulay duration can rise above the perpetuity benchmark for a range of maturities and then eventually fall back toward the perpetuity limit as maturity becomes extremely large. This is sometimes called the **deep discount paradox**.

Why it matters: ultra-long bonds that drift into deep-discount territory can see duration *increase* as they age, which is the opposite of the usual “as maturity shortens, risk goes down” intuition.

### 12.5.4 Worked Example C: The Deep Discount Paradox

Consider a 1% coupon bond at a 5% yield. At what maturities does duration exceed the perpetuity's 20.5 years?

Using numerical calculation:

| Maturity | Price | Macaulay Duration |
|:---------|:------|:------------------|
| 30 years | 38.02 | 20.3 |
| 50 years | 21.95 | 23.8 |
| 70 years | 13.60 | 24.9 |
| 100 years | 7.60 | 24.7 |
| ∞ (perpetuity) | 20.00 | 20.5 |

The 1% coupon bond has duration exceeding perpetuity duration between roughly 35 and 150 years to maturity!

> **Desk Reality:** Deep-discount ultra-long bonds can get *more* rate-sensitive as time passes.  
> **Common break:** Assuming “aging = de-risking” and sizing hedges off stale duration.  
> **What to check:** Recompute duration/DV01 after big yield moves; the PV weight on the final principal can dominate and shift non-monotonically with maturity for low coupons.

---

## 12.6 Dollar Duration and DV01: From Percentages to Dollars

### 12.6.1 Definition and Purpose

Modified duration is a *percentage* sensitivity. Converting it into dollar P&L requires the full (dirty) price and the position size.

Let $P_{\text{clean}}$ be the quoted clean price per 100 notional, and let $AI$ be accrued interest per 100. The dirty (cash/invoice) price is:
$$
P_{\text{dirty}} = P_{\text{clean}} + AI.
$$

Throughout this chapter, prices are per 100 notional. For a position with notional $N$, define the (dirty) market value / PV as:
$$
V := \frac{N}{100}\,P_{\text{dirty}}.
$$

**Check (points $\leftrightarrow$ dollars):** If prices are quoted “per 100,” then 1.00 price point is $1\%$ of face value. So a 0.01-point move is $0.01\%$ of face. Example: if $N=\$100\text{mm}$, then 1.00 point is $\$1{,}000{,}000$ and 0.01 point is $\$10{,}000$. This is the quickest way to sanity-check whether a reported DV01 magnitude is plausible.

Using the duration approximation $\Delta P/P \approx -D_{\text{Mod}}\,\Delta y$ and scaling by the position size gives:
$$
\Delta V \approx -(D_{\text{Mod}}V)\,\Delta y.
$$

One definition of **dollar duration** is the PV slope with respect to a small yield change:
$$
D_{\$} := -\frac{\Delta V}{\Delta y}.
$$
Under the duration approximation above, $D_{\$} \approx D_{\text{Mod}}V$. In this chapter we write:
$$
DD := D_{\text{Mod}}\,V,
$$
so $\Delta V \approx -DD\,\Delta y$. For a 100bp move ($\Delta y=0.01$), the magnitude of the price change is roughly $|DD|\times 0.01$.

### 12.6.2 DV01 Conventions (Bump Object, Size, Units, Sign)

DV01 is the change in PV for a **1 basis point** move. In this chapter, DV01 is **yield-based**: we bump the bond’s own yield-to-maturity $y$ (under the same compounding basis used to define $D_{\text{Mod}}$) while holding promised cashflows fixed.

- **Bump object:** the bond’s yield-to-maturity $y$  
- **Bump size:** $1\text{ bp} = 10^{-4}$ (in yield decimal units)  
- **Definition (book convention):** $DV01 := V(y-1\text{bp}) - V(y)$  
- **Units:** currency per 1bp (always state whether “per 100 notional”, “per \$1mm”, or for the actual position)  
- **Sign:** for a long fixed-rate bond, DV01 is typically positive (yields down $\Rightarrow$ PV up)

With a small-bump approximation, DV01 is duration times market value times $0.0001$:
$$
DV01 \approx DD \times 0.0001 = \frac{D_{\text{Mod}}\,V}{10{,}000}.
$$

If you work in **price points per 100 notional**, the corresponding DV01 is:
$$
DV01_{\text{pts}} \approx \frac{P_{\text{dirty}}\,D_{\text{Mod}}}{10{,}000},
$$
and you convert to dollars for notional $N$ by multiplying by $N/100$.

> **Pitfall — DV01 scaling and sign drift:** DV01 is routinely misread because of unit conversions.  
> **Why it matters:** Hedge ratios end up off by $10{,}000\times$ or with the wrong sign.  
> **Quick check:** For \$1mm notional, $DV01_{\$}\approx P_{\text{dirty}}\times D_{\text{Mod}}$ (in dollars per 1bp). If you get a number like $0.05$ without stating “per 100”, you are probably mixing units.

### 12.6.3 Worked Example — From Duration to DV01 to P&L (Concrete Timeline)

**Example Title**: Par 10‑year coupon bond — compute DV01 and a 25bp P&L

**Context**
- You see “Duration: 7.8” on a risk report and need to translate it into dollars.
- The goal is to connect $D_{\text{Mod}}$ $\rightarrow$ DV01 $\rightarrow$ P&L with explicit units and sign.

**Timeline (Make Dates Concrete)**
- Trade/valuation date: 2026-04-15 (assume valuation is just after the coupon is paid, so $AI\approx 0$)
- Settlement date: 2026-04-15 (ignore settlement lag and business-day adjustments for this toy example)
- Accrual start/end: 2026-04-15 to 2026-10-15 (semiannual)
- Payment dates: 2026-10-15, 2027-04-15, …, 2036-04-15

**Inputs**
- Instrument: fixed-rate bond, notional $N=\$10{,}000{,}000$, maturity 2036-04-15, coupon $c=5\%$ paid semiannually
- Market quote: yield $y=5\%$ with semiannual compounding ($m=2$)
- Day count / compounding: treat coupon periods as $0.5$ years (duration math); $AI=0$ at the coupon date

**Outputs (What You Produce)**
- $P_{\text{clean}}=P_{\text{dirty}}=100$ per 100 notional (par at $c=y$ under this convention)
- $V = \frac{N}{100}P_{\text{dirty}}=\$10{,}000{,}000$
- Modified duration (par-bond formula): $D_{\text{Mod}} \approx 7.79$ years
- Position DV01: $\approx \$7{,}790$ per 1bp (book convention: PV change for a 1bp *fall* in yield)

**Step-by-step**
1. **Compute modified duration** for a 10‑year par bond at yield $y$ (semiannual):
   $$
   D_{\text{Mod}}=\frac{1}{y}\left(1-\frac{1}{(1+y/2)^{2T}}\right)
   =\frac{1}{0.05}\left(1-\frac{1}{1.025^{20}}\right)\approx 7.79.
   $$
2. **Compute DV01** using $DV01 \approx \frac{D_{\text{Mod}}V}{10{,}000}$:
   $$
   DV01 \approx \frac{7.79\times 10{,}000{,}000}{10{,}000}=\$7{,}790\text{ per bp}.
   $$
3. **Translate a rate move into P&L.** For a +25bp yield increase ($\Delta y=+0.0025$):
   $$
   \Delta V \approx -DD\,\Delta y = -(D_{\text{Mod}}V)\Delta y \approx -(7.79)(10{,}000{,}000)(0.0025)=-\$194{,}750.
   $$

**Cashflows (table)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-10-15 | \$250,000 | coupon $=N\times c/2$ |
| 2036-04-15 | \$10,250,000 | final coupon + principal |

**P&L / Risk Interpretation**
- The DV01 of $\$7.79k$ means: *if yield falls 1bp, PV increases about $\$7.79k$* (for this $N$ and bump object).
- A +25bp move is roughly $-25\times DV01$ for a long bond (first-order).
- For larger moves, convexity (Chapter 13) and curve-shape moves (Chapter 14) matter.

**Sanity Checks**
- Units: DV01 is in dollars per 1bp; here it is in the “thousands of dollars” range for a \$10mm 10‑year bond, which is plausible.
- Sign: yields up $\Rightarrow$ PV down, so $\Delta V$ is negative for $\Delta y>0$.
- Reproduction: you can reproduce $DV01$ by bumping $y$ by $\pm 1$bp and repricing the cashflows.

### 12.6.4 Why Dollar Duration Matters

The main operational reason risk systems report DV01 (or dollar duration) is that these *dollar* sensitivities aggregate cleanly across positions: you can sum them.

If you have two positions:
- Bond A: $DD^A = 800{,}000$
- Bond B: $DD^B = 1{,}200{,}000$

Portfolio dollar duration = \$800{,}000 + 1{,}200{,}000 = 2{,}000{,}000$

You cannot simply add percentage durations. A portfolio with 50% in a 5-year duration bond and 50% in a 10-year duration bond does *not* have 15-year duration—it has weighted average duration of 7.5 years.

### 12.6.5 Worked Example D: Dollar Duration Calculation

**Bond A:** \$50mm face, Price = 105, Modified Duration = 6.5

Market value: $V = 50{,}000{,}000 \times \frac{105}{100} = 52{,}500{,}000$.

Dollar duration:
$$
DD = D_{\text{Mod}}V = 6.5 \times 52{,}500{,}000 = 341{,}250{,}000.
$$

**DV01:**

$$\text{DV01}^A \approx 341{,}250{,}000 \times 0.0001 = \$34{,}125 \text{ per basis point}$$

**Verification:** A 1bp rate increase gives $\Delta V \approx -DD \times 0.0001 = -\$34{,}125$. ✓

> **Desk Reality:** Risk reports usually show DV01/PV01 in dollars, not “duration in years.”  
> **Common break:** The report’s sign convention may be “up‑1bp” (PV01) rather than “down‑1bp” (DV01 as defined in this book).  
> **What to check:** Confirm (i) bump object (bond YTM vs curve), (ii) bump size, and (iii) whether the report is $V(y+1\text{bp})-V(y)$ or $V(y-1\text{bp})-V(y)$. Then translate moves as $\Delta V\approx \pm (\text{bp move})\times (\text{reported bp-risk})$ with the correct sign.

---

## 12.7 The DV01-Duration Mapping

### 12.7.1 The Fundamental Relationship

The yield-based DV01 relationship is a unit conversion: duration is a *percentage* sensitivity, and DV01 is a *dollar* (or price-point) sensitivity.

Per **100 notional** (DV01 in price points per 1bp):
$$
\boxed{DV01_{\text{pts}} \approx \frac{P_{\text{dirty}}\,D_{\text{Mod}}}{10{,}000}}
$$

For a **position** with notional $N$ and market value $V=(N/100)P_{\text{dirty}}$ (DV01 in dollars per 1bp):
$$
\boxed{DV01_{\$} \approx \frac{D_{\text{Mod}}\,V}{10{,}000}}.
$$

Or using Macaulay duration:

$$\text{DV01} = \frac{P \times D_{\text{Mac}}}{10{,}000(1+y/2)}$$

The division by 10,000 appears because DV01 is for a **1 basis point** move ($10^{-4}$), while modified duration is sensitivity per **1.00** change in yield.

### 12.7.2 The "Price Effect" vs "Duration Effect"

Duration and DV01 move differently with maturity because they measure different objects:
- **Duration** is a *percentage* sensitivity (per unit price).
- **DV01** is an *absolute* sensitivity (scaled by price/market value).

As maturity changes, DV01 reflects two competing effects:

1. **The Duration Effect**: Longer maturity → higher duration → higher DV01
2. **The Price Effect**: Price changes with maturity can amplify or offset the duration effect

For **par bonds** (price fixed at 100 per 100), only the duration effect operates.

For **premium bonds** (price > 100), higher price can reinforce the duration effect.

For **discount bonds** (price < 100), the lower price can partially offset higher duration, and DV01 may peak and then decline at long maturities.

For **zero-coupon bonds**, duration rises with maturity but price falls rapidly as maturity extends; for $y>0$, price $\to 0$ as $T\to\infty$, so DV01 eventually falls toward 0.

### 12.7.3 Worked Example E: Premium vs Discount Bonds

Consider two 10-year bonds both yielding 5%:

**Premium Bond** (8% coupon):
- Price ≈ 123.16
- Macaulay Duration ≈ 7.54 years
- Modified Duration = 7.54/1.025 = 7.36
- DV01 ≈ $(123.16 \times 7.36)/10{,}000 = 0.0906$ price points per 100 per 1bp  
  (≈ $\$906$ per 1bp per \$1mm notional)

**Discount Bond** (2% coupon):
- Price ≈ 76.83
- Macaulay Duration ≈ 8.94 years
- Modified Duration = 8.94/1.025 = 8.72
- DV01 ≈ $(76.83 \times 8.72)/10{,}000 = 0.0670$ price points per 100 per 1bp  
  (≈ $\$670$ per 1bp per \$1mm notional)

The discount bond has *higher duration* (more "leveraged" to rates), but the premium bond has *higher DV01* (more dollars at risk because price is higher).

**Hedging implication:** To DV01-hedge \$1mm notional of the premium bond with the discount bond, you need a notional ratio of about $0.0906/0.0670\approx 1.35$.

---

## 12.8 Portfolio Duration and the Aggregation Trap

### 12.8.1 The Portfolio Duration Formula

For a long-only portfolio with total market value $P$, portfolio duration is a market-value-weighted average:

$$\boxed{D_{\text{portfolio}} = \sum_{i} \frac{X_i}{P} D_i = \sum_{i} w_i D_i}$$

where $X_i$ is the market value of asset $i$, $P$ is total portfolio value, and $w_i = X_i/P$ are the value weights.

### 12.8.2 The Aggregation Trap

A frequent source of confusion is computing "Portfolio Duration" for hedged portfolios.

Consider a trade: Long \$100mm of a 10-year Treasury, Short \$100mm of a 10-year futures contract.

- Assets ≈ Liabilities
- **Net Market Value** of the portfolio is near zero
- However, the **Risk** (DV01) might be non-zero if the hedge isn't perfect

If you try to compute weighted average duration, you divide by Net Market Value near zero, resulting in "portfolio duration" that explodes to infinity or fluctuates wildly.

**Solution:** For hedged portfolios, **aggregate DV01** (or dollar duration), not percentage duration:

$$\text{Portfolio DV01} = \sum_{i} \text{DV01}_i^{\text{Position}}$$

### 12.8.3 Worked Example F: Portfolio DV01 Calculation

**Bond A (Long):** \$5mm face, Price = 102, Duration = 4.5
- Market Value = \$5.1mm
- Position DV01 $\approx D_{\text{Mod}} \times V / 10{,}000 = 4.5 \times 5.1\text{mm}/10{,}000 = \mathbf{+\$2{,}295}$

**Bond B (Short):** \$3mm face, Price = 108, Duration = 7.2
- Market Value = \$3.24mm
- Position DV01 $\approx -\,7.2 \times 3.24\text{mm}/10{,}000 = \mathbf{−\$2{,}333}$

**Portfolio DV01** = \$2,295 − \$2,333 = **−\$38**

Interpretation (with this chapter’s DV01 convention): a portfolio DV01 of $-\$38$ means PV *falls* by about $\$38$ for a 1bp **fall** in yields, so it *gains* about $\$38$ for a 1bp **rise** in yields. To neutralize yield-based DV01, add $\$38$ of positive DV01.

> **Desk Reality:** Portfolio “duration” can be unstable or undefined in hedged books.  
> **Common break:** Net market value near zero makes “percentage duration” explode, while DV01 remains finite.  
> **What to check:** Use position-level DV01/dollar duration for aggregation; verify the sign convention with a 1bp bump.

---

## 12.9 Immunization: Duration in Action

### 12.9.1 The Immunization Principle

Immunization is the classical duration-matching idea in asset–liability management: choose an asset portfolio so that, for small **parallel** yield shifts, the PV of assets and the PV of liabilities move together (to first order).

The core principle is: **match PV and match duration** of assets to liabilities.

### 12.9.2 The Immunization Conditions

To immunize a liability stream, construct a bond portfolio satisfying:

1. **Present Value Matching:** $\text{PV}(\text{Assets}) = \text{PV}(\text{Liabilities})$
2. **Duration Matching:** $D_{\text{Assets}} = D_{\text{Liabilities}}$

For a single liability of $L$ due at time $T$:
- PV of liability = $L/(1+y/2)^{2T}$
- Duration of liability = $T$ (it's equivalent to a zero)

### 12.9.3 Worked Example G: Immunizing a Future Obligation

**Setup:** The X Corporation owes \$1 million in 10 years. They want to invest now to meet this obligation using two bonds:

- **Bond 1:** 6% coupon, 30-year maturity, Price = 69.04, Duration = 11.44 years
- **Bond 2:** 11% coupon, 10-year maturity, Price = 113.04, Duration = 6.54 years

Current yield: 9% for all bonds.

**Step 1: Calculate obligation present value**

$$\text{PV} = \frac{1{,}000{,}000}{(1.045)^{20}} = \$414{,}643$$

**Step 2: Set up immunization equations**

Let $V_1$ = investment in Bond 1, $V_2$ = investment in Bond 2.

$$V_1 + V_2 = 414{,}643 \quad \text{(PV matching)}$$

$$\frac{11.44 \cdot V_1 + 6.54 \cdot V_2}{414{,}643} = 10 \quad \text{(Duration matching)}$$

**Step 3: Solve**

From the second equation: $11.44 V_1 + 6.54 V_2 = 4{,}146{,}430$

Substituting $V_2 = 414{,}643 - V_1$:

$11.44 V_1 + 6.54(414{,}643 - V_1) = 4{,}146{,}430$

$4.90 V_1 = 1{,}433{,}765$

$V_1 = \$292{,}606$ in Bond 1

$V_2 = \$122{,}037$ in Bond 2

**Verification (idea):** If yields shift to 8% or 10%, repricing the portfolio and the liability stream gives a small “surplus” (assets minus liabilities), illustrating that the match is first-order accurate for parallel shifts.

### 12.9.4 Limitations of Immunization

Immunization has important limitations:

1. **First-order only:** Protects against small parallel shifts; large moves require convexity matching (Chapter 13)
2. **Parallel shift assumption:** Real curves twist and steepen, not just shift
3. **Rebalancing required:** As time passes and yields change, duration drifts and portfolios need reimmunization
4. **Equal yield assumption:** Assumes all bonds have the same yield (unrealistic for different credits)

For portfolios with long and short positions, immunization is usually implemented by making the **net dollar duration / DV01** close to zero (for a stated bump object).

> **Desk Reality:** “Duration gap” is the ALM headline number.  
> **Common break:** Treating it as a full hedge while ignoring curve-shape moves, convexity, and rebalancing drift.  
> **What to check:** Confirm the bump design (parallel vs key rates), how often the book is rebalanced, and whether the liabilities are modeled with the same discounting assumptions as assets.

---

## 12.10 Duration and Value at Risk

### 12.10.1 The Duration-Based VaR Formula

If you approximate portfolio P&L under **parallel yield shifts** with the linear duration model,

$$\Delta V \approx -DD\,\Delta y,$$

then under a normal-yield-change approximation a horizon-$T$ VaR can be written as:

$$\boxed{\text{VaR} \approx |DD|\,\sigma_y\,z_\alpha\,\sqrt{T}}$$

where:
- $DD=D_{\text{Mod}}\,V$ is dollar duration (currency per 100% yield move),
- $\sigma_y$ is the standard deviation of daily yield changes (in yield decimals),
- $z_\alpha$ is the normal quantile (e.g., 1.65 for 95%, 2.33 for 99%),
- $T$ is the holding period in days.

Equivalently, using DV01 and $\sigma_y$ measured in bp:

$$\text{VaR} \approx |\text{DV01}| \times \sigma_y(\text{in bp}) \times z_\alpha \times \sqrt{T}$$

### 12.10.2 Worked Example H: Computing Duration-Based VaR

Assume a bond portfolio has value $V=\$6{,}000{,}000$ and modified duration $D_{\text{Mod}}=5.2$. Assume daily parallel yield changes have standard deviation $0.09\%$ (so $\sigma_y=0.0009$ in yield decimals).

**Calculate 20-day 90% VaR:**

$$\text{VaR} = 5.2 \times 6{,}000{,}000 \times 0.0009 \times 1.282 \times \sqrt{20}$$

$$= 31{,}200{,}000 \times 0.0009 \times 1.282 \times 4.472$$

$$= \$161{,}289$$

The 90% VaR over 20 days is approximately **\$161,000**.

### 12.10.3 Limitations of Duration-Based VaR

Limitations include:
1. **Parallel shift assumption:** Only captures level risk, not slope or curvature
2. **Normality assumption:** Yield changes aren't always normally distributed
3. **Static assumption:** Ignores convexity effects for large moves
4. **Single-factor:** Doesn't capture tenor-specific risks

More accurate approaches use multiple curve factors (e.g., key-rate DV01s or PCA-style level/slope/curvature factors) and include convexity where material.

> **Desk Reality:** Duration-VaR can look “too small” for curve trades.  
> **Common break:** A steepener/flattening position may have near-zero *parallel* DV01 but large key-rate exposures.  
> **What to check:** Look at key-rate DV01s (Chapter 14) or factor exposures (level/slope/curvature), not just parallel duration.

---

## 12.11 Effective Duration vs Modified Duration

### 12.11.1 The Distinction

Terminology varies across desks and systems, so always ask “*what is being bumped?*” In this book we separate:

**Modified Duration:**
- Assumes the bond's own YTM changes
- Cash flows are fixed
- Computed analytically from the price formula

**Effective Duration:**
- Assumes a small shift in the benchmark curve (often a parallel shift, or a set of key-rate shifts)
- Cash flows may change (callable bonds, MBS/ABS) because the *expected* cashflows depend on rates
- Computed numerically by bumping the curve and repricing

### 12.11.2 When They Differ

For option-free bonds with flat curves, effective ≈ modified duration.

They diverge when:
1. **Embedded options:** Callable bonds have effective duration < modified (the call limits upside)
2. **MBS/ABS:** Prepayments change with rates, so cash flows aren't fixed
3. **Steep curves:** Bumping the curve differs from bumping the YTM

### 12.11.3 Calculating Effective Duration

Effective duration is computed numerically:

$$D_{\text{eff}} = \frac{P_{-} - P_{+}}{2 \times P_0 \times \Delta y}$$

where:
- $P_0$ = current price
- $P_{+}$ = price after upward curve shift of $\Delta y$
- $P_{-}$ = price after downward curve shift of $\Delta y$

**Check (bp vs decimal):** In this formula, $\Delta y$ is in **decimal yield units**. A 1bp bump is $\Delta y=10^{-4}$; a 10bp bump is $\Delta y=10^{-3}$. Forgetting this conversion is an easy way to be off by 10,000× in an “effective duration” calculation.

This "bump and reprice" approach captures how the full price function responds to curve changes.

> **Desk Reality: Which Duration for Which Instrument?**
>
> | Instrument | Use Modified? | Use Effective? |
> |------------|--------------|----------------|
> | Treasury bonds | ✓ | ✓ (equivalent) |
> | Investment-grade corporates | ✓ | ✓ (similar) |
> | Callable bonds | Limited | ✓ |
> | MBS/ABS | No | ✓ (OAS duration) |
> | Floating rate notes | ✓ (low) | ✓ (low) |

---

## Summary
1. **Macaulay duration** $D_{\text{Mac}}$ is a PV-weighted average cashflow time (in years) computed using the full/dirty price.
2. **Modified duration** $D_{\text{Mod}}$ is the first-order price sensitivity to the bond’s yield: $-\frac{1}{P}\frac{dP}{dy}=D_{\text{Mod}}$. In yield-based settings, $D_{\text{Mod}} = D_{\text{Mac}}/(1+y/m)$ for compounding frequency $m$.
3. For small yield moves (in decimals), $\Delta V \approx -D_{\text{Mod}}\,V\,\Delta y$. Always state what $y$ means (compounding/day count) and what is being bumped.
4. **Dollar duration** $DD := D_{\text{Mod}}V$ converts “years” into currency exposure. It is currency per 1.00 (100%) yield change.
5. **DV01** is a 1bp version of dollar duration. With this book’s convention, $DV01 := V(y-1\text{bp})-V(y) \approx DD/10{,}000$ and is typically positive for a long fixed-rate bond.
6. Duration is a *percentage* sensitivity; DV01 is an *absolute* sensitivity. DV01 depends on both duration and the price/market value level (the “price effect”).
7. Special cases: zeros have $D_{\text{Mac}}=T$; par bonds admit simple closed forms; long-maturity coupon bonds approach a perpetuity benchmark.
8. Duration usually falls with higher yield and higher coupon. Ultra-long deep-discount bonds can show non-monotone maturity behavior (“deep discount paradox”).
9. Portfolio duration is a market-value-weighted average for positive-value long-only portfolios. For hedged books with near-zero PV, aggregate DV01/dollar duration instead.
10. Duration-based VaR uses $\text{VaR} \approx |DD|\sigma_y z_\alpha\sqrt{T}$ under a parallel-shift, linear-P&L approximation; it can miss curve-shape risk and convexity.

## Key Concepts

| Concept | Definition | Why It Matters |
|:--------|:-----------|:---------------|
| Macaulay duration $D_{\text{Mac}}$ | PV-weighted average cashflow time (years) | Time interpretation; equals maturity for zeros |
| Modified duration $D_{\text{Mod}}$ | $-\frac{1}{P}\frac{dP}{dy}$ for the stated yield definition | First-order % price sensitivity; links to DV01 |
| Dollar duration $DD$ | $DD=D_{\text{Mod}}V$ | Currency exposure that aggregates across positions |
| DV01 | $DV01 := V(y-1\text{bp})-V(y) \approx DD/10{,}000$ | Standard “bp-risk” number on a risk report |
| Bump object | The variable being shifted (bond YTM, curve nodes, par quotes, etc.) | Different bump objects produce different “DV01s” |
| Price effect vs duration effect | DV01 depends on both duration and price level | Explains why high-price premium bonds can have larger DV01 |
| Deep discount paradox | Duration can exceed perpetuity benchmark at ultra-long maturities for low coupons | Hedge ratios can move the “wrong way” as a bond ages |
| Portfolio duration | Value-weighted average of component durations | Valid for positive-value long-only portfolios |
| Aggregation trap | “Portfolio duration” blows up when net PV is near zero | Use DV01/dollar duration instead |
| Effective duration | Curve-bump, model-based duration when cashflows can change | Used for callables/MBS; depends on model and bump design |
| Immunization | Match PV and duration of assets to liabilities | First-order hedge for parallel shifts; requires rebalancing |
| Convexity | Second-order curvature of $P(y)$ | Explains duration approximation error for large moves |

---

## Notation

| Symbol | Meaning | Units / Convention |
|:-------|:--------|:------------------|
| $P_{\text{clean}}$ | clean price | price per 100 notional |
| $AI$ | accrued interest | price per 100 notional; $P_{\text{dirty}}=P_{\text{clean}}+AI$ |
| $P_{\text{dirty}}$ | full/dirty price | price per 100 notional |
| $N$ | notional | currency |
| $V$ | market value / PV | currency; $V=(N/100)P_{\text{dirty}}$ |
| $y$ | yield-to-maturity | per year (decimal); compounding basis must be stated |
| $m$ | compounding frequency | payments per year |
| $T$ | time to maturity | years |
| $D_{\text{Mac}}$ | Macaulay duration | years |
| $D_{\text{Mod}}$ | modified duration | years; $-\frac{1}{P}\frac{dP}{dy}$ for stated $y$ |
| $DD$ | dollar duration | currency per 1.00 yield change; $DD=D_{\text{Mod}}V$ |
| $DV01$ | dollar value of 1bp | currency per 1bp; book: $V(y-1\text{bp})-V(y)$ |
| $D_{\text{eff}}$ | effective duration | years; curve-bump, bump-and-reprice |
| $w_i$ | portfolio value weight | $w_i=X_i/P$ for positive-value portfolios |

---

## Flashcards

| # | Question | Answer |
|:--|:---------|:-------|
| 1 | What is Macaulay duration? | The weighted-average time to receipt of cash flows, where weights are PV of each cash flow divided by total price. |
| 2 | What is the Macaulay duration of a zero-coupon bond? | Exactly equal to its maturity T. |
| 3 | How do you convert Macaulay to Modified duration? | $D_{Mod} = D_{Mac} / (1 + y/m)$ where m is compounding frequency. |
| 4 | What does modified duration measure? | The approximate percentage price change for a 100bp (1%) change in yield. |
| 5 | What is dollar duration? | $DD=D_{Mod}V$. It measures the dollar change for a 100bp (1.00%) yield move (under the stated bump object). |
| 6 | How is DV01 related to dollar duration? | $DV01 \approx DD/10{,}000$. Always check whether your system uses an up-1bp or down-1bp convention. |
| 7 | What is the Macaulay duration of a perpetuity at 5% yield? | $(1.025)/(0.05) = 20.5$ years. |
| 8 | What is the modified duration of a perpetuity at yield y? | $1/y$. At 5%, that's 20 years. |
| 9 | How does duration change as coupon increases (fixed maturity)? | Duration decreases—higher coupon means more value paid earlier. |
| 10 | Can a bond's duration ever exceed a perpetuity's duration? | Yes—deep discount bonds at very long maturities can exceed perpetuity duration. |
| 11 | Why do risk systems use dollar duration (or DV01) instead of percentage duration? | Dollar durations add directly; percentage durations require value weighting. |
| 12 | What is the "portfolio duration trap"? | Computing weighted-average duration fails when net market value is near zero (hedged books). |
| 13 | What are the two conditions for immunization? | Match present values AND match durations of assets and liabilities. |
| 14 | What is effective duration? | Price sensitivity to a parallel shift in the benchmark curve (vs. YTM shift for modified). |
| 15 | When should you use effective duration instead of modified? | For callable bonds, MBS, or any instrument with embedded options. |
| 16 | What is the VaR formula using duration? | $\text{VaR} \approx |DD|\sigma_y z_\alpha \sqrt{T}$ under a parallel-shift, linear-P&L approximation. |
| 17 | If a bond has duration 7 and rates rise 50bp, what's the approximate price change? | About −3.5% (= −7 × 0.50%) |
| 18 | What competing effects determine how DV01 varies with maturity? | The "duration effect" (longer → higher D) and "price effect" (price changes with maturity). |
| 19 | For a \$100mm portfolio with 5-year duration, what's the P&L from a 10bp rate rise? | About −\$500,000 (= −5 × \$100mm × 0.001) |
| 20 | Who invented duration and when? | Frederick Macaulay in 1938. |

---

## Mini Problem Set

### Questions

**Q1. [Easy — Calculation]** A 5-year zero-coupon bond has a yield of 4% (semiannual compounding). Calculate its:
(a) Macaulay duration
(b) Modified duration
(c) DV01 per 100 face

---

**Q2. [Easy — Conversion]** A 10-year bond has Macaulay duration of 8.2 years. Yield is 6% (semiannual). What is:
(a) Modified duration
(b) Dollar duration for \$1 million face at price 105

---

**Q3. [Medium — Portfolio]** You hold a portfolio of three bonds:

| Bond | Face ($mm) | Price | Duration |
|:-----|:-----------|:------|:---------|
| A | 10 | 102 | 3.5 |
| B | 15 | 98 | 6.2 |
| C | 5 | 110 | 9.1 |

Calculate:
(a) Total market value
(b) Portfolio duration (value-weighted)
(c) Portfolio DV01

---

**Q4. [Medium — Deep Discount]** A 50-year bond has a 1% coupon and yields 6%. Compare its Macaulay duration to a perpetuity at the same yield. Which is larger and why?

---

**Q5. [Medium — Immunization]** You have a \$500,000 liability due in 8 years. You can invest in:
- Bond X: Duration 5 years, yield 5%
- Bond Y: Duration 12 years, yield 5%

Design an immunized portfolio. How much in each bond?

---

**Q6. [Medium — VaR]** A \$10mm bond portfolio has modified duration 6.5 years. Daily yield standard deviation is 8bp. Calculate the 10-day 99% VaR using the duration model.

---

**Q7. [Medium — Price/Duration Effect]** Two 15-year bonds both yield 5%:
- Bond A: 8% coupon (premium)
- Bond B: 2% coupon (discount)

Which has higher duration? Which has higher DV01? Explain the relationship.

---

**Q8. [Hard — Sensitivity]** A 10-year par bond at 5% yield has duration 7.79 years. If yields fall to 4%:
(a) Estimate the new duration (qualitative direction)
(b) Why does this pose a problem for hedging?

---

**Q9. [Hard — Effective vs Modified]** Explain why a callable bond at a yield just above the call trigger has effective duration much lower than modified duration.

---

**Q10. [Hard — Perpetuity Limit]** Calculate the Macaulay duration of par bonds at 5% yield for maturities 10, 30, 50, and 100 years. Compare to the perpetuity duration of 20.5 years.

---

**Q11. [Hard — Dollar Duration Aggregation]** Prove that the dollar duration of a portfolio equals the sum of the dollar durations of its components. (Hint: use the definition and the fact that portfolio value = sum of component values.)

---

**Q12. [Hard — Integration]** A pension fund has liabilities with PV = \$100mm and duration 12 years. Design a portfolio using 5-year zeros (priced at 78.35) and 20-year par bonds (duration 12.5) to immunize the liability. How many of each bond do you need?

---

### Solution Sketches (Selected)

**Q1.**
(a) Zero-coupon: $D_{Mac} = T = 5$ years
(b) $D_{Mod} = 5/(1.02) = 4.90$ years
(c) Price = $100/(1.02)^{10} = 82.03$. DV01 = $(82.03 × 4.90)/10,000 = 0.0402$ per 100 face

**Q2.**
(a) $D_{Mod} = 8.2/(1.03) = 7.96$ years
(b) Market value = \$1mm × 1.05 = \$1.05mm. Dollar duration = \$7.96 × 1,050,000 = 8,358,000

**Q3.**
(a) MV = \$10mm×1.02 + \$15mm×0.98 + \$5mm×1.10 = \$10.2 + \$14.7 + \$5.5 = \$30.4mm
(b) $D = (10.2×3.5 + 14.7×6.2 + 5.5×9.1)/30.4 = (35.7 + 91.14 + 50.05)/30.4 = 5.82$ years
(c) DV01 = $(30.4mm × 5.82)/10,000 = \$17,693$

**Q4.** Assuming semiannual compounding, the perpetuity Macaulay duration (in years) is:
$$D_{\text{Mac}} = \frac{1+y/2}{y}$$
At $y=6\%$, $D_{\text{Mac}} = 1.03/0.06 = 17.17$ years. The 50-year deep discount (1% coupon) has duration approximately 21 years—exceeding the perpetuity due to the heavy weighting on the distant principal.

**Q5.** Let $V_X$ in Bond X, $V_Y$ in Bond Y.
- PV matching: $V_X + V_Y = 500,000/(1.025)^{16} = 335,602$
- Duration matching: $5V_X + 12V_Y = 8 × 335,602$

Solving: $V_Y = (8×335,602 - 5×335,602)/(12-5) = 143,829$; $V_X = 191,773$

**Q6.** VaR = \$6.5 × 10,000,000 × 0.0008 × 2.326 × √10 = 6.5 × 10mm × 0.0008 × 2.326 × 3.162 = \$382,437

**Q7 (sketch).** The discount bond has higher duration (more PV weight on the distant principal). The premium bond can still have higher DV01 because its price level is higher, scaling the *absolute* dollar sensitivity.

**Q8 (sketch).** Duration increases when yields fall (PV weight shifts toward later cashflows), so a DV01 hedge sized at 5% yield will generally be under-hedged after a rally unless you rebalance; convexity also matters for larger moves.

## References

- Tuckman & Serrat, *Fixed Income Securities: Tools for Today's Markets* (Macaulay/modified duration; yield-based DV01; perpetuities; deep discount behavior).
- Hull, *Options, Futures, and Other Derivatives* (duration/convexity definitions and worked examples).
- Hull, *Risk Management and Financial Institutions* (dollar duration, portfolio aggregation, duration-based VaR discussions).
- Luenberger, *Investment Science* (immunization: matching PV and duration; convexity considerations).
- Neftci, *Principles of Financial Engineering* (“DV01 and PV01”; duration and yield-based sensitivity measures).
- Brandimarte, *Numerical Methods in Finance and Economics* (duration/convexity definitions and the sensitivity link $\frac{dP}{d\lambda}=-D_M P$).
- *Simulation and Optimization in Finance* (modified vs effective/option-adjusted duration for instruments with rate-dependent cashflows).
