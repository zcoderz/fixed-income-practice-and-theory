# Chapter 8: Spreads 101 — G-spread, I-spread, Z-spread, OAS, and "What Spread Are We Talking About?"

---

## Introduction

A portfolio manager calls to say her position "widened 15 basis points overnight." A credit trader reports his book is "cheap by 10 bp to fair value." A research analyst writes that spreads on BBB corporates are "historically tight." In each case, the word *spread* appears as if it were a single, well-defined concept—but it is not.

The fixed income markets use spreads as a universal language for expressing "everything in a bond price that is not the risk-free term structure." Spreads allow practitioners to compare bonds with different coupons and maturities, to decompose P&L into "rates" versus "credit" components, and to risk-manage exposures to credit deterioration and liquidity shocks. Yet there is a fundamental trap: the word "spread" can refer to at least half a dozen different quantities, and each responds differently to market movements.

O'Kane observes that the simple yield spread "does not take into account the term structure and depends on coupon"—meaning two bonds from the same issuer with identical credit risk can show different yield spreads simply because of their coupon and maturity profiles. Tuckman emphasizes that a single yield "is not a complete description when interest rates change over time." These warnings motivate the development of more sophisticated spread measures: the Z-spread (zero-volatility spread) which incorporates the full term structure, the OAS (option-adjusted spread) which accounts for embedded options, and the asset swap spread which converts fixed-rate bonds into floating-rate economics.

This chapter builds a taxonomy of spread definitions so that when someone says "the spread widened 10 bp," you can immediately ask the five essential clarifying questions:

1. Which benchmark curve? (Treasury? Fitted Treasury? OIS? Swap?)
2. Which spread type? (G-spread? I-spread? Z-spread? OAS? Asset swap? TED spread?)
3. Clean or dirty price?
4. Which compounding and day-count convention?
5. Option-adjusted or not?

We begin with the conceptual foundation—clean versus dirty prices and yield-to-maturity—then work through each spread definition in order of complexity: G-spread, I-spread, Z-spread, asset swap spread, TED spread, and OAS. We conclude with O'Kane's decomposition of credit spreads into actuarial, default risk premium, and liquidity components, which reveals that a quoted "spread" is never purely about default probability.

> **Analogy: The Speedometer Sensors**
>
> Why are there so many spreads? Think of assessing a car's speed.
> *   **G-Spread**: Speed relative to the ground (Government/Risk-Free).
> *   **I-Spread**: Speed relative to traffic flow (Swaps/Interbank).
> *   **Z-Spread**: Speed adjusted for the curvy road (Term Structure).
> *   **OAS**: Speed excluding the headwind/tailwind of embedded options.
>
> **Insight**: No spread is "true". They just measure distance from different baselines.
>
> **Clarification: Yield Spread vs Bid/Ask Spread**
> *   **Yield Spread**: A measurement of **Risk** (Why does this bond yield more than Treasuries?).
> *   **Bid/Ask Spread**: A measurement of **Transaction Cost** (Liquidity).
> Don't confuse them!

---

## 8.1 Clean vs Dirty Price: The Starting Point

Before computing any spread, we must be precise about what price we are matching. The **dirty price** (also called the *full price* or *invoice price*) is the actual cash exchanged at settlement. The **clean price** (also called the *flat price* or *quoted price*) is what markets quote, excluding accrued interest.

Tuckman states this relationship directly:

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + AI}$$

where $AI$ is accrued interest. For a bond with annual coupon rate $c$, payment frequency $f$, and accrual fraction $\Delta$ (the fraction of the coupon period elapsed since the last payment), a common linear approximation is:

$$AI = \Delta \cdot \frac{c}{f} \cdot 100$$

**Why this matters for spreads:** Spread engines—whether computing Z-spread or OAS—should be calibrated to match the dirty price, because discounting future cash flows produces the total present value that equals the settlement amount. Yield spreads in market commentary, however, can be ambiguous about which price convention is used. Always confirm.

**Unit check:** $c/f$ is the coupon per period in decimal form; multiplying by 100 (the notional) gives cash per 100 face value. If $c = 0.06$ (6%) and $f = 2$ (semiannual), then each coupon is $3$ per 100. With $\Delta = 0.40$, we have $AI = 0.40 \times 3 = 1.20$.

---

## 8.2 Yield to Maturity and Yield Spreads

### 8.2.1 The Yield as a Summary Statistic

The **yield to maturity** ($y$) is the internal rate of return that equates the present value of a bond's promised cash flows to its price. For a bond with maturity $T$ years, coupon rate $c$, and semiannual compounding, the pricing equation from Tuckman is:

$$P = \frac{c}{y}\left(1 - \frac{1}{(1 + y/2)^{2T}}\right) + \frac{100}{(1 + y/2)^{2T}}$$

where $c$ is the annual coupon in dollars per 100 (so a 6% coupon bond has $c = 6$).

This single number $y$ compresses the entire term structure into one rate. It is convenient for quick comparisons but can hide important details. Hull explains that "a bond's yield spread is the excess of the promised yield on the bond over the risk-free rate" and notes that "the usual assumption is that the excess yield is compensation for the possibility of default." However, as we will see, this assumption oversimplifies what spreads actually capture.

### 8.2.2 The Generic Yield Spread

The simplest spread definition is the **yield spread**:

$$s = y_{\text{bond}} - y_{\text{ref}}(T)$$

where $y_{\text{ref}}(T)$ is a benchmark yield at the same maturity $T$, typically interpolated from surrounding benchmarks if no exact-maturity instrument exists.

This is fast to quote and easy to compare across issues. But its limitations are serious: two bonds from the same issuer with different coupons will show different yield spreads even if they have identical credit risk. A high-coupon bond has more of its value in near-term cash flows, which are discounted less than a low-coupon bond's back-loaded cash flows, creating a coupon effect that distorts the spread comparison.

O'Kane emphasizes this limitation: the yield spread "does not take into account the term structure and depends on coupon." This motivates the development of term-structure-consistent measures like the Z-spread.

---

## 8.3 G-spread: Spread to Government Curve

The **G-spread** (government spread) is the yield spread computed versus a government benchmark:

$$\boxed{s_G(T) = y_{\text{bond}} - y_{\text{gov}}(T)}$$

where $y_{\text{gov}}(T)$ is a government (sovereign) yield at the bond's maturity. If no benchmark exists at exactly $T$, the benchmark yield is obtained by interpolation:

$$y_{\text{gov}}(T) = y_{\text{gov}}(T_1) + \frac{T - T_1}{T_2 - T_1}\left(y_{\text{gov}}(T_2) - y_{\text{gov}}(T_1)\right)$$

**Intuition:** G-spread answers the question "How much extra yield am I getting above risk-free governments?"

**Benchmark selection matters critically.** The answer depends on whether $y_{\text{gov}}(T)$ means:
- The **on-the-run Treasury yield** at that maturity (highly liquid but potentially distorted by specialness), or
- A **fitted Treasury zero/par curve** (smoother, arguably better for valuation).

Tuckman emphasizes that on-the-run government yields "can be affected by security-specific liquidity" and "specialness" effects, where strong demand to short a particular bond drives its repo rate below general collateral and inflates its price. He documents liquidity premiums of 5-6 basis points in the two- and five-year sectors on February 15, 2001. Using on-the-run yields as benchmarks can contaminate your spread measure with these technical effects. A fitted curve avoids this contamination but requires curve construction methodology.

> **Practical Note:** When comparing G-spreads across sources, always ask whether they use on-the-run Treasuries or a fitted curve. The difference can be 5-10 basis points—material for relative value analysis.

---

## 8.4 I-spread: Spread to Swap Curve

The **I-spread** (interpolated swap spread or ISDA spread) is the yield spread versus a swap curve:

$$\boxed{s_I(T) = y_{\text{bond}} - y_{\text{swap}}(T)}$$

where $y_{\text{swap}}(T)$ is typically the par swap rate at maturity $T$.

**When to use I-spread:** In markets where swaps serve as the discounting benchmark (or as the "risk-free-ish" reference), I-spread is a natural quote. Post-2008 markets increasingly use OIS for discounting and SOFR-based curves for floating-rate projections.

**Which swap curve?** This is not a trivial question. You must clarify:
- Is it an OIS curve (overnight indexed swaps)?
- A legacy single-curve LIBOR construction?
- The ISDA standard methodology with specific interpolation?

Tuckman discusses how benchmark choice issues in swap spread calculations apply equally to I-spread computations. The difficulties are analogous to G-spread: different curve constructions produce different I-spread values.

**Comparison:** For the same bond, G-spread and I-spread will differ because government yields and swap yields differ. In normal markets, swap rates exceed Treasury yields (positive swap spread), so $s_I < s_G$. This is not a calculation error—it is a definitional difference.

> **Expert Note: Negative Swap Spreads**
> After the 2008 financial crisis, U.S. swap spreads at longer maturities (e.g., 30Y) notably turned negative, meaning Treasury yields exceeded swap rates. Tuckman notes that "the fall in swap spreads in the early 1990s reflected the recovery of the banking sector from its problems in the 1980s" while "the rise in swap spreads in the late 1990s... can be best explained by a perceived scarcity in the supply of U.S. Treasury securities relative to demand." Post-2008 negative swap spreads are driven by the balance sheet costs of holding Treasuries and the demand for duration in swap format. In such a regime, the traditional relationship flips, and $s_I$ can be wider (larger) than $s_G$.
>
> **History Lesson: Regime Change**
> *   **"Old Days" (Pre-2008)**: Treasuries were risk-free, Swaps were risky. I-Spread > G-Spread.
> *   **"New Days" (Post-2008)**: Treasuries are scarce collateral, Swaps are abundant. Swap Spreads can be negative. I-Spread < G-Spread?
> *   **Key**: Spreads are relative. If the benchmark moves (Swap spreads widen), your bond's I-spread might tighten even if its price didn't move!

---

## 8.5 Z-spread: Zero-Volatility Spread

### 8.5.1 Definition and Intuition

The **Z-spread** (zero-volatility spread, or ZVS in some texts) addresses the fundamental limitation of yield spreads: it incorporates the full term structure of discount factors rather than comparing single yields.

O'Kane defines the ZVS as "the fixed spread adjustment to the Libor [or government] discount rate which reprices the bond." In continuously compounded form:

$$\boxed{P_{\text{dirty}} = \sum_k CF_k \cdot P_{\text{bench}}(0, t_k) \cdot e^{-s_Z t_k}}$$

where:
- $CF_k$ is the cash flow at time $t_k$
- $P_{\text{bench}}(0, t_k)$ is the benchmark discount factor from 0 to $t_k$
- $s_Z$ is the Z-spread (continuous compounding)

**Intuition:** Instead of comparing one yield to one benchmark yield, Z-spread asks: "What constant spread must I apply to the entire benchmark discounting curve so that the present value matches the bond's dirty price?"

> **Visualization: The Parallel Shift**
>
> 1.  Draw the **Zero Curve** (Risk-Free).
> 2.  Draw the Bond's Price (converted to yield) floating above it.
> 3.  **Action**: Shift the *entire* Zero Curve up by $X$ bps until the PV matches the Price.
> 4.  **Result**: $X$ is the Z-Spread.
>
> Contrast this with Yield Spread, which just looks at one single point.

O'Kane also provides a discrete compounding version:

$$P = \frac{c}{f} \sum_{n=1}^{N} \frac{1}{(1 + (r(0, t_n) + \theta)/f)^n} + \frac{1}{(1 + (r(0, t_N) + \theta)/f)^N}$$

where $r(0, t_n)$ is the discretely compounded zero rate and $\theta$ is the Z-spread.

### 8.5.2 Solving for Z-spread

Z-spread is solved numerically. Define:

$$f(s) = \sum_k CF_k \cdot P_{\text{bench}}(0, t_k) \cdot e^{-s t_k} - P_{\text{dirty}}$$

and find $s_Z$ such that $f(s_Z) = 0$.

**Properties of $f(s)$:**
- $f(s)$ is monotonically decreasing in $s$ (higher spread → heavier discounting → lower PV)
- This guarantees a unique solution when the bond price is between reasonable bounds
- Bisection is robust; Newton-Raphson is faster

**Repricing check:** After solving, always plug $s_Z$ back into the PV equation and verify it reproduces $P_{\text{dirty}}$ within tolerance.

### 8.5.3 Why Z-spread Differs from Yield Spreads

On a flat term structure, Z-spread and yield spreads converge. On a steep curve, they can differ materially—especially for high-coupon bonds.

Consider a high-coupon bond on an upward-sloping curve. The early cash flows (coupons) are discounted at lower short-term rates, while the yield spread calculation uses a single blended rate. The Z-spread properly weights each cash flow by its maturity, producing a different number.

O'Kane notes that "some practitioners choose to hedge using the ZVS rather than the yield. The ZVS is preferred because it takes into account the term structure of the interest rate curve."

> **Deep Dive: Direction of Bias**
> On a typical upward-sloping (steep) curve, the spot rate curve often lies above the par yield curve at the same maturity (since forward rates > spot rates > par yields). If we intuitively view Z-spread as a spread to the spot curve and G-spread as a spread to the par yield, a bond priced off the higher spot curve requires *less* added spread to match a given low price than a bond priced off the lower par yield. Thus, for standard bullet bonds on steep curves, we often observe **Z-spread < G-spread**. (See Example D below.)

---

## 8.6 Asset Swap Spread

### 8.6.1 What Is an Asset Swap?

An **asset swap** is a package consisting of a bond and an interest rate swap, structured so the combined cost equals par. As O'Kane describes: "The asset swap buyer enters into a payer interest rate swap. The payment schedule of the fixed leg is set so that it is identical to that of the fixed rate bond. On the floating leg, the asset swap buyer receives payments of Libor plus $A$, where $A$ is known as the asset swap spread."

The economics: the investor effectively converts a fixed-rate bond into a floating-rate instrument, receiving Libor plus a spread. That spread is then interpreted as a spread-to-swaps measure.

O'Kane provides the essential intuition: "What this tells us is that an asset swap is equivalent to going long a defaultable bond and short a risk-free bond with the same coupon schedule. The asset swap spread is then the amortised payment of the price difference over the life of the asset swap."

### 8.6.2 Par Asset Swap Formula

Tuckman defines asset swap spread as "the spread such that discounting the bond's cash flows by swap rates plus that spread gives the bond price."

O'Kane provides the **par asset swap** formula:

$$\boxed{A(0) = \frac{P_{\text{Libor}} - P}{PV01(0,T)}}$$

where:
- $P_{\text{Libor}} = \frac{c}{f} \sum_{n=1}^{N} Z(0, \hat{t}_n) + Z(0,T)$ is the value of the bond's cash flows discounted at flat Libor
- $P$ is the bond's market (dirty) price
- $PV01(0,T) = \sum_{m=1}^{M} Z(0, t_m) \cdot \Delta(t_{m-1}, t_m)$ is the "annuity" or present value of receiving 1 bp per period

**Unit check:** The numerator is a price difference (dimensionless per unit notional); the denominator has units of years (sum of discounted year fractions). The result is an annualized spread (rate per year).

O'Kane emphasizes: "The asset swap spread is therefore a measure of the credit quality of the fixed rate bond. If the bond issuer has the credit quality of the AA commercial banking sector, then it will probably have a price close to $P_{\text{Libor}}$ and the asset swap spread will be close to zero."

### 8.6.3 Market Convention Warning

O'Kane highlights an important ambiguity: "There are different asset swap market conventions, including 'market asset swap' where the floating leg notional equals the market value of the bond, reducing counterparty exposure versus a par-notional structure."

If someone quotes "ASW," confirm whether it is:
- **Par asset swap:** Notional is fixed at par (standard definition above)
- **Market asset swap:** Floating-leg notional equals bond market value

These produce different spread numbers for the same bond. O'Kane shows that for a market asset swap, $A^*(0) = A(0)/P$, meaning the market asset swap spread is higher for a discount bond than the par asset swap spread.

---

## 8.7 TED Spread: Spread to Futures Rates

### 8.7.1 Definition and Purpose

The **TED spread** (originally Treasury-Eurodollar spread) uses rates implied by short-term interest rate futures to assess the value of a security relative to those futures rates. Tuckman defines it as "the spread such that discounting cash flows at Eurodollar futures rates minus that spread produces the security's market price."

Put another way, it is the negative of the OAS when futures rates are used for discounting. This makes TED spreads particularly useful for:
- Comparing agency securities to money market benchmarks
- Relative value between similar bonds with different maturities
- Hedging with liquid futures contracts

### 8.7.2 Calculation Approach

Tuckman provides a detailed example using FNMA bonds. The approach:

1. Use futures rates (now SOFR futures; historically Eurodollar futures) for each period
2. Compute discount factors for each cash flow date using these rates
3. Solve for the spread $s$ such that:

$$P_{\text{dirty}} = \sum_k CF_k \cdot DF_k(s)$$

where $DF_k(s)$ adjusts the discount factor for the TED spread.

**Example:** Tuckman shows a FNMA bond with a TED spread of 15.6 basis points, meaning "the agency is 15.6 basis points rich to LIBOR as measured by the futures rates."

### 8.7.3 Theoretical Caveat

Tuckman notes a theoretical flaw: "Discounting a bond's cash flows using futures rates has an obvious theoretical flaw. According to the results of Part One, discounting should be done at forward rates, not futures rates."

However, for relatively short-dated securities, the convexity adjustment between futures and forwards is small. The TED spread methodology trades theoretical purity for practical hedgeability—the futures are liquid instruments that can be used to implement relative value trades.

---

## 8.8 OAS: Option-Adjusted Spread

### 8.8.1 Definition

For bonds with embedded options—callable bonds, mortgage-backed securities—the Z-spread includes the option's value mixed in with the credit/liquidity spread. The **option-adjusted spread** (OAS) attempts to separate these.

Tuckman defines OAS as "the spread that when added to all the short rates in the risk-neutral tree for discounting purposes produces a model price equal to the market price."

The relationship between Z-spread and OAS, as O'Kane states:

$$\boxed{ZVS = OAS + \text{Option Cost}}$$

For a **callable bond**, the investor is implicitly short a call option to the issuer. This option has positive value to the issuer (negative value to the investor). The bond trades at a lower price (higher yield) to compensate for this option. Thus, the total Z-spread (which matches the low price) is *higher* than the "pure" credit spread (OAS).

**Rule of Thumb:** Callable Bond $\implies$ Z-spread > OAS.

### 8.8.2 Model Dependence

OAS is inherently model-dependent. Different interest rate models, volatility assumptions, and tree constructions will produce different OAS values for the same market price. As Tuckman explains, the OAS calculation requires a "risk-neutral tree" for the short rate, and changing the tree's volatility structure changes the option's value and hence the OAS.

Tuckman notes that "the name option-adjusted spread... arose because the concept of OAS was first developed to analyze the embedded call options in mortgage-backed securities and callable bonds. The name is now a misnomer, however, because OAS can be and is calculated for securities that... have no option features."

O'Kane similarly observes that "in the world of credit, the same measure is used to quantify the spread over the Libor curve due to the embedded credit risk where no optionality is present." When applied to non-callable bonds, O'Kane prefers the name "zero volatility spread" to avoid confusion about whether optionality is present.

### 8.8.3 DVOAS: Sensitivity to OAS

Tuckman introduces **DVOAS** as the sensitivity of price to a one-basis-point change in OAS:

$$\text{DVOAS} \approx \frac{P(OAS - 1\text{ bp}) - P(OAS + 1\text{ bp})}{2}$$

For P&L attribution, Tuckman shows that a security's return can be decomposed as:

$$dP = (r + OAS) \cdot P \cdot dt + DV01_x \cdot (dx - E[dx]) + DVOAS \cdot dOAS$$

The three components are: **carry** (time value plus OAS), **factor exposure** (unexpected rate moves), and **convergence** (OAS change). A "cheap" security (positive OAS) earns superior returns by collecting OAS carry and by converging to fair value.

Tuckman emphasizes: "The two decompositions highlight the usefulness of OAS as a measure of the value of a security with respect to a particular model. According to the model, a long position in a cheap security earns superior returns in two ways. First, it earns the OAS over time intervals in which the security does not converge to its fair value. Second, it earns the DVOAS times the extent of any convergence."

---

## 8.9 Spread Duration and CS01

### 8.9.1 Spread Duration

Given a spread measure $s$, the **spread duration** measures price sensitivity:

$$\boxed{D_s := -\frac{1}{P}\frac{\partial P}{\partial s}}$$

For Z-spread with continuous compounding, where $P(s) = \sum_k CF_k \cdot P_{\text{bench}}(0, t_k) \cdot e^{-s t_k}$:

$$\frac{\partial P}{\partial s} = -\sum_k t_k \cdot CF_k \cdot P_{\text{bench}}(0, t_k) \cdot e^{-s t_k}$$

so:

$$D_s = \frac{\sum_k t_k \cdot PV_k(s)}{\sum_k PV_k(s)}$$

This is a PV-weighted average time—similar in structure to Macaulay duration but computed under the spreaded discounting.

### 8.9.2 CS01 (Credit Spread 01)

**CS01** is the price change for a 1 bp change in spread:

$$\boxed{CS01 \approx \frac{\partial P}{\partial s} \times 0.0001 = -P \cdot D_s \times 0.0001}$$

**Intuition:** If spread widens by 1 bp, price falls. CS01 is typically negative when expressed as "price change per 100 notional."

O'Kane defines DV01 as $-\frac{\partial P}{\partial y} \times 10^{-4}$, and spread DV01 follows the same convention for spread sensitivities.

### 8.9.3 Why CS01 Is Not Universal

A critical point: **CS01 depends on which spread definition you use.** A portfolio's "spread risk" varies by spread type:

| Spread | Risk Sensitivities |
|--------|-------------------|
| **G-spread risk** | Sensitive to bond yield changes, government benchmark yield changes, benchmark selection (on-the-run vs fitted) |
| **I-spread risk** | Depends on which swap curve (OIS vs LIBOR), curve construction, interpolation |
| **Z-spread risk** | Depends on benchmark discount factors across all maturities |
| **OAS risk** | Depends on rate model, volatility assumption, option exercise and cash flow modeling |

When someone quotes "CS01," always clarify: CS01 with respect to which spread measure?

---

## 8.10 What Spread Are We Really Talking About? O'Kane's Decomposition

### 8.10.1 The Credit Risk Premium

Market spreads are not pure measures of default probability. Hull notes that when estimating default probabilities from bond yield spreads, "the usual assumption is that the excess yield is compensation for the possibility of default." But this assumption is incomplete.

O'Kane provides a conceptual decomposition:

$$\boxed{\text{Credit spread} = \text{Actuarial spread} + \text{Default risk premium} + \text{Volatility risk premium} + \text{Liquidity risk premium}}$$

The components are:

- **Actuarial spread:** Compensation for expected loss implied by historical default rates and recovery rates—what you'd need to break even on average.
- **Default risk premium:** Additional spread for uncertainty around default predictions. Historical statistics may not reliably predict future defaults; investors demand compensation for this model risk.
- **Volatility risk premium:** Compensation for the risk that credit quality changes (spreads widen) even without default, causing mark-to-market losses.
- **Liquidity risk premium:** Compensation for the risk of not being able to sell when needed due to market illiquidity.

O'Kane defines:
$$\text{Spread premium} = \text{Credit spread} - \text{Actuarial spread}$$

and:
$$\text{Coverage ratio} = \frac{\text{Credit spread}}{\text{Actuarial spread}}$$

> **Visualization: The Risk Layer Cake**
>
> Imagine a cake slice showing the components of a corporate spread.
> *   **Base Layer (Thin)**: **Expected Loss** (Actuarial). "Cost of doing business."
> *   **Middle Layer (Thick)**: **Uncertainty Premium**. "What if our model is wrong?"
> *   **Top Layer (Thick)**: **Liquidity Premium**. "Payment for being stuck."
>
> **Insight**: When you buy a corporate bond, you are eating this whole cake. OAS measures the size of the *entire* cake, not just the default risk. Often, the Risk Premiums are 3x-5x larger than the actual Expected Loss.

### 8.10.2 Empirical Magnitudes

O'Kane and Schloegl (2002) calculated coverage ratios by rating category using CDS spreads and Moody's historical default/recovery data:

| Rating | 5Y Avg Spread (bp) | Actuarial Spread (bp) | Coverage Ratio | Spread Premium (bp) |
|--------|-------------------|----------------------|----------------|---------------------|
| AA | 28 | 9 | 3.12 | 19 |
| A | 61 | 13 | 4.67 | 48 |
| BBB | 164 | 30 | 5.54 | 134 |
| BB | 463 | 145 | 3.19 | 318 |

The pattern shows that market spreads typically cover actuarial expected loss 3-5 times over. The spread premium (non-actuarial component) increases as we descend the rating spectrum, reflecting higher default risk premia, volatility risk premia, and liquidity risk premia for lower-rated credits.

Hull provides corroborating evidence: using historical default data and implied hazard rates from market spreads, he shows that "expected excess return" on bonds ranges from 38 basis points for Aaa to 124 basis points for Ba—substantially more than the actuarial default expectation alone.

### 8.10.3 Implication for Spread Analysis

Tuckman emphasizes that "asset swap spreads are computed using a swap curve not contaminated by security-specific effects" but that asset swap spreads themselves "do reflect security-specific effects like special financing or supply/demand imbalances."

This means even a "spread-to-swaps" measure can embed non-credit components: repo specialness, technical supply/demand, and funding considerations. When analyzing why two bonds from the same issuer trade at different spreads, the answer may lie in liquidity or technical factors rather than different credit views.

---

## 8.11 Worked Examples

All examples use 100 notional. Spreads are in basis points unless stated otherwise.

### Example A: G-spread Calculation

**Given:**
- Corporate bond: 5-year maturity, 6% coupon (semiannual), settlement on coupon date (AI = 0)
- Clean price: $P_{\text{clean}} = 98.50$
- Treasury benchmark yields: $y_{\text{gov}}(4y) = 4.20\%$, $y_{\text{gov}}(6y) = 4.60\%$

**Step 1 — Dirty price:** $P_{\text{dirty}} = 98.50 + 0 = 98.50$

**Step 2 — Solve for YTM:** Using Tuckman's formula with $c = 6$:

$$P(y) = \frac{6}{y}\left(1 - \frac{1}{(1 + y/2)^{10}}\right) + \frac{100}{(1 + y/2)^{10}}$$

Trial: $y = 6.30\%$ gives $P = 98.74$; $y = 6.40\%$ gives $P = 98.28$. Interpolating to hit 98.50:

$$y_{\text{bond}} \approx 6.355\%$$

**Step 3 — Interpolate Treasury yield at 5y:**

$$y_{\text{gov}}(5) = 4.20\% + \frac{5-4}{6-4}(4.60\% - 4.20\%) = 4.40\%$$

**Step 4 — G-spread:**

$$s_G = 6.355\% - 4.40\% = 1.955\% = \boxed{195.5 \text{ bp}}$$

### Example B: I-spread (Same Bond)

**Given:** Same bond with $y_{\text{bond}} = 6.355\%$; swap yields: $y_{\text{swap}}(4y) = 4.40\%$, $y_{\text{swap}}(6y) = 4.80\%$

**Interpolate:** $y_{\text{swap}}(5) = 4.40\% + \frac{1}{2}(0.40\%) = 4.60\%$

**I-spread:** $s_I = 6.355\% - 4.60\% = 1.755\% = \boxed{175.5 \text{ bp}}$

**Compare:** G-spread is 195.5 bp; I-spread is 175.5 bp—a 20 bp difference solely from benchmark choice.

### Example C: Z-spread from Discount Factors

**Given:**
- 3-year bond, annual coupon 5%, settlement on coupon date
- Cash flows: $CF_1 = 5$, $CF_2 = 5$, $CF_3 = 105$
- Dirty price: $P_{\text{dirty}} = 98.00$
- Benchmark discount factors: $P(0,1) = 0.97$, $P(0,2) = 0.93$, $P(0,3) = 0.88$

**PV function (continuous spread):**

$$PV(s) = 5 \times 0.97 \times e^{-s} + 5 \times 0.93 \times e^{-2s} + 105 \times 0.88 \times e^{-3s}$$

**Bracket the root:**
- $s = 1.00\%$: $PV = 99.03$ (too high)
- $s = 2.00\%$: $PV = 96.24$ (too low)

**Bisection:**
- $s = 1.50\%$: $PV = 97.62$ (too low)
- $s = 1.25\%$: $PV = 98.32$ (too high)
- $s = 1.366\%$: $PV = 98.00$ ✓ (Precise value 136.6 bp)

**Result:** $s_Z \approx 1.366\% = \boxed{136.6 \text{ bp}}$

### Example D: Why G-spread ≠ Z-spread on a Steep Curve

**Given:**
- 3-year bond, annual coupon 10%, dirty price $P = 95.00$
- Government spot rates: 1y = 2%, 2y = 4%, 3y = 6% (steep curve)

**Compute discount factors:**
- $P(0,1) = 1/1.02 = 0.9804$
- $P(0,2) = 1/1.04^2 = 0.9246$
- $P(0,3) = 1/1.06^3 = 0.8396$

**Corporate YTM:** Solve $95 = \frac{10}{1+y} + \frac{10}{(1+y)^2} + \frac{110}{(1+y)^3}$. Result: $y_{\text{bond}} \approx 12.085\%$

**Government 3y par yield:** $y_{\text{gov}}(3) = \frac{1 - 0.8396}{0.9804 + 0.9246 + 0.8396} = 5.84\%$

**G-spread:** $s_G = 12.085\% - 5.84\% = 6.24\% = \boxed{624 \text{ bp}}$

**Z-spread:** Solve $95 = 10 \times 0.9804 \times e^{-s} + 10 \times 0.9246 \times e^{-2s} + 110 \times 0.8396 \times e^{-3s}$. Result: $s_Z \approx 5.84\% = \boxed{584 \text{ bp}}$

**Difference:** ~40 bp. This confirms the intuition that on a steep curve where spot > par, Z-spread is typically less than G-spread for bullet bonds.

### Example E: Asset Swap Spread

**Given:**
- 5-year annual coupon bond, $c = 6\%$, dirty price $P = 98.50$
- Swap discount factors: $Z(0,1) = 0.97$, $Z(0,2) = 0.94$, $Z(0,3) = 0.90$, $Z(0,4) = 0.85$, $Z(0,5) = 0.80$

**PV01 (annuity):** $\sum Z(0,m) \times 1 = 0.97 + 0.94 + 0.90 + 0.85 + 0.80 = 4.46$

**$P_{\text{Libor}}$:** Coupon PV + principal PV = $6 \times 4.46 + 100 \times 0.80 = 26.76 + 80 = 106.76$

**Asset swap spread (per unit notional):**

$$A(0) = \frac{1.0676 - 0.9850}{4.46} = \frac{0.0826}{4.46} = 0.01852 = \boxed{185.2 \text{ bp}}$$

### Example F: OAS in a Two-Step Rate Tree

**Setup:**
- 2-year callable bond, annual coupon 5%, call price 100 at t=1
- Short rates: $r_0 = 4\%$; at t=1: up $r_u = 6\%$, down $r_d = 3\%$
- Risk-neutral probability: 0.5 up, 0.5 down
- Market price: $P_{\text{mkt}} = 99.50$

**With OAS = 0:**
- Up node: continuation = $5 + 105/1.06 = 104.06$; call payoff = 105 → value = 104.06 (not called)
- Down node: continuation = $5 + 105/1.03 = 106.94$; call payoff = 105 → value = 105 (called)
- Expected t=1 value: $0.5 \times 104.06 + 0.5 \times 105 = 104.53$
- Price at t=0: $104.53/1.04 = 100.51$

Model price 100.51 > market 99.50 → need positive OAS.

**With OAS = 72 bp (0.72%):**
- Discounting uses $1 + r + OAS$
- Up: $5 + 105/1.0672 = 103.39$ → not called
- Down: $5 + 105/1.0372 = 106.23 > 105$ → called at 105
- Expected: $0.5 \times 103.39 + 0.5 \times 105 = 104.20$
- Price: $104.20/1.0472 = 99.50$ ✓

**Result:** $OAS \approx \boxed{72 \text{ bp}}$

**Key insight:** If you change tree volatility (spreads between 6% and 3%), option value changes and so does OAS—illustrating model dependence.

---

## 8.12 Practical Notes

### Common Ambiguity Traps

| Issue | Details |
|-------|---------|
| **Benchmark curve choice** | On-the-run yields are liquid but may be distorted by specialness; fitted curves are smoother but methodology-dependent |
| **Clean vs dirty confusion** | Z-spread/OAS should match dirty price; adding accrued twice or using clean price biases spreads |
| **Compounding mismatch** | BEY (semiannual), annual effective, and continuous spreads differ; mixing conventions produces meaningless results |
| **Par yields vs zero rates** | Par yields are coupon rates that price a par bond; they are not the same as spot rates. Using par yields as if they were zeros is a common error |
| **Callable bonds** | Z-spread includes option value; OAS attempts to remove it (but is model-dependent) |

### Implementation Pitfalls

| Issue | Details |
|-------|---------|
| **Interpolation choices** | Interpolating yields, discount factors, or forwards produces different curves and different Z-spreads |
| **Stub periods** | Irregular first/last periods require correct accrued interest and year fractions |
| **Negative rates** | Discount factors remain positive, but rate formulas may behave unexpectedly |
| **Asset swap conventions** | Par vs market structures differ; confirm before comparing ASW levels |

### Verification Checklist

- **Spread sign:** Wider spread → lower price (given fixed benchmark)
- **Repricing test:** Computed Z-spread/OAS must reproduce $P_{\text{dirty}}$ when plugged back in
- **Limiting cases:** Zero-coupon bond → yield spread ≈ Z-spread (no coupon effect); flat curve → yield spread ≈ Z-spread

---

## Summary

Spreads are the universal language of fixed income credit, but the word "spread" hides a taxonomy of distinct quantities:

1. **Clean vs dirty:** All PV-based spreads should match dirty price—the actual settlement amount
2. **Yield is a summary:** Convenient but potentially misleading when the term structure is steep or the bond has unusual cash flows
3. **Yield spreads** (G-spread, I-spread) are fast but ignore term structure and depend on coupon
4. **G-spread:** Bond YTM minus government benchmark yield—sensitive to benchmark choice (on-the-run vs fitted)
5. **I-spread:** Bond YTM minus swap curve yield—must specify which swap curve (OIS, SOFR, legacy LIBOR). **Note:** I-spread can exceed G-spread in negative swap spread regimes.
6. **Z-spread:** Solves a PV match using the full benchmark discount curve; term-structure consistent but still benchmark-dependent. Typically Z-spread < G-spread on steep curves.
7. **TED spread:** The spread to futures rates that reprices the bond—useful for hedging with liquid futures
8. **OAS:** Solves a PV match using an interest-rate model/tree; isolates spread after removing option value; inherently model-dependent
9. **Asset swap spread:** Converts fixed bond to floating economics; par vs market conventions differ
10. **ZVS = OAS + option cost** for callable bonds
11. **Market spreads embed multiple components:** actuarial expected loss, default risk premium, volatility risk premium, liquidity premium—not just probability of default

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **G-spread** | $y_{\text{bond}} - y_{\text{gov}}(T)$ | Quick relative value vs governments; benchmark-dependent |
| **I-spread** | $y_{\text{bond}} - y_{\text{swap}}(T)$ | Relative value vs swaps; curve definition matters |
| **Z-spread** | Constant spread to benchmark discounting matching dirty price | Term-structure consistent; for non-callable bonds |
| **TED spread** | Spread to futures rates that reprices bond | Hedgeable with liquid futures contracts |
| **OAS** | Spread to model rates so model price = market price | For callable/option bonds; model-dependent |
| **Asset swap spread** | $(P_{\text{Libor}} - P)/PV01$ | Spread-to-swaps; par vs market convention |
| **CS01** | $\partial P / \partial s \times 0.0001$ | Spread sensitivity; depends on spread definition |
| **Spread duration** | $-\frac{1}{P}\frac{\partial P}{\partial s}$ | PV-weighted average time under spreaded discounting |
| **Coverage ratio** | Credit spread / Actuarial spread | How much spreads exceed pure default compensation |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $P_{\text{clean}}$ | Clean (quoted) price per 100 |
| $P_{\text{dirty}}$ | Dirty (invoice) price per 100 |
| $AI$ | Accrued interest per 100 |
| $y$ | Yield to maturity |
| $y_{\text{gov}}(T)$ | Government benchmark yield at maturity $T$ |
| $y_{\text{swap}}(T)$ | Swap curve yield at maturity $T$ |
| $s_G$ | G-spread |
| $s_I$ | I-spread |
| $s_Z$ | Z-spread |
| $s_{\text{OAS}}$ | Option-adjusted spread |
| $P(0,t)$ or $Z(0,t)$ | Discount factor from 0 to $t$ |
| $D_s$ | Spread duration |
| $CS01$ | Credit spread 01 (sensitivity to 1 bp spread change) |
| $PV01$ | Annuity: $\sum Z(0,t_m) \Delta(t_{m-1}, t_m)$ |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the dirty price? | Dirty = clean + accrued interest; the actual cash exchanged |
| 2 | Why do Z-spread calculations match dirty price? | PV discounting should reproduce the total settlement amount |
| 3 | Define yield spread in simplest form | $s = y_{\text{bond}} - y_{\text{ref}}(T)$ |
| 4 | Define G-spread | Bond YTM minus government benchmark yield at same maturity |
| 5 | Define I-spread | Bond YTM minus swap curve yield at same maturity |
| 6 | Key limitation of yield spreads? | They ignore term structure and depend on coupon |
| 7 | What is Z-spread (ZVS)? | Constant spread added to benchmark discounting so PV = dirty price |
| 8 | Z-spread PV equation (continuous)? | $P = \sum_k CF_k \cdot P(0,t_k) \cdot e^{-s_Z t_k}$ |
| 9 | Why can G-spread differ from Z-spread? | G-spread is single-yield comparison; Z-spread uses full curve |
| 10 | Define OAS | Spread added to all rates in a model/tree so model price = market price |
| 11 | ZVS vs OAS for callable bonds? | ZVS = OAS + option cost (ZVS > OAS) |
| 12 | What is spread duration? | $D_s = -\frac{1}{P}\frac{\partial P}{\partial s}$ |
| 13 | What is CS01? | Price change for 1 bp spread change: $\partial P/\partial s \times 0.0001$ |
| 14 | Why isn't CS01 universal? | Different spread definitions (G, I, Z, OAS) have different sensitivities |
| 15 | Asset swap spread conceptually? | Spread such that discounting at swap + spread gives bond price |
| 16 | Par asset swap formula? | $A = (P_{\text{Libor}} - P) / PV01$ |
| 17 | What does PV01 represent? | Annuity: present value of receiving 1 bp per period |
| 18 | Key asset swap ambiguity? | Par vs market structure (floating notional differs) |
| 19 | If spread widens, what happens to price? | Price decreases (all else equal) |
| 20 | Repricing test for Z-spread? | Plug $s_Z$ back in; verify PV = $P_{\text{dirty}}$ |
| 21 | Why is OAS model-dependent? | Option value depends on interest rate dynamics/volatility used |
| 22 | List three components of credit spread (O'Kane) | Actuarial spread, default risk premium, liquidity risk premium |
| 23 | What is actuarial spread? | Compensation for expected loss from historical default/recovery |
| 24 | What is coverage ratio? | Credit spread / actuarial spread |
| 25 | What question to ask when someone says "spread widened"? | "Which spread definition and which benchmark curve?" |
| 26 | What is TED spread? | Spread to futures rates that reprices the bond |
| 27 | Why use TED spread? | Hedgeable with liquid futures contracts |

---

## Mini Problem Set

**1.** A bond has clean price 101.20 and accrued interest 0.80. What is the dirty price?

**2.** A semiannual coupon bond has coupon rate 6% and accrual fraction $\Delta = 0.40$. Compute accrued interest per 100.

**3.** A 2-year annual coupon bond with cash flows 5, 105 has dirty price 99.00. Solve for YTM approximately.

**4.** A bond YTM is 7.10%. The interpolated government yield at maturity is 4.85%. Compute G-spread in bp.

**5.** Given discount factors $P(0,1) = 0.97$, $P(0,2) = 0.93$, $P(0,3) = 0.88$ and cash flows 5, 5, 105 with dirty price 98.00, solve for Z-spread approximately.

**6.** Explain in one paragraph why yield spread can differ from Z-spread on a steep curve.

**7.** A bond has price 100 and spread duration 4.5. Estimate price change for a 10 bp spread widening.

**8.** You have two curves: fitted Treasury zeros and on-the-run Treasury yields. Which is more appropriate for Z-spread and why?

**9.** A trader quotes "I-spread 120 bp." List three clarifications you need.

**10.** A Z-spread engine accidentally matches clean price instead of dirty. What's the likely direction of bias?

**11.** For a callable bond, would you expect Z-spread larger or smaller than OAS? Explain.

**12.** Why might two bonds from the same issuer trade at different spreads?

### Solution Sketches (1-7)

**1.** $P_{\text{dirty}} = 101.20 + 0.80 = 102.00$

**2.** Coupon per period $= 0.06/2 \times 100 = 3.00$. $AI = 0.40 \times 3.00 = 1.20$

**3.** Solve $99 = 5/(1+y) + 105/(1+y)^2$. At $y = 6\%$: PV = 98.18 (low). At $y = 5\%$: PV = 100 (high). So $y \approx 5.5\%$.

**4.** $s_G = 7.10\% - 4.85\% = 2.25\% = 225$ bp

**5.** Bracket with $s = 1\%$ (PV ≈ 99), $s = 2\%$ (PV ≈ 96). Bisect to $s_Z \approx 1.37\% = 137$ bp. (Exact 136.6 bp).

**6.** Yield spread compares one yield to one yield, ignoring that different cash flows should be discounted at different maturities. Z-spread revalues each cash flow using the full curve. On a steep curve, spot rates are often higher than par yields. Since Z-spread adds to the (higher) spot curve while G-spread adds to the (lower) par yield, Z-spread is often lower than G-spread for bullet bonds.

**7.** $\Delta P \approx -P \times D_s \times \Delta s = -100 \times 4.5 \times 0.0010 = -0.45$

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| Full price = flat price + accrued interest | Tuckman Ch 1-4 |
| Yield spread ignores term structure, depends on coupon | O'Kane Ch 4-5 |
| Single yield is not complete description when rates vary | Tuckman |
| Bond yield spread is excess over risk-free rate | Hull Ch 24.4 |
| Z-spread (ZVS) definition as constant spread to discounting | O'Kane Ch 4.2.9 |
| Z-spread continuous and discrete compounding formulas | O'Kane Ch 4.2.9 |
| Practitioners prefer ZVS to yield for hedging | O'Kane Ch 4.2.9 |
| OAS definition (spread added to tree rates) | Tuckman Ch 14 |
| OAS name arose from callable bond/MBS analysis | Tuckman Ch 14 |
| OAS is now used for securities without option features | Tuckman Ch 14 |
| ZVS = OAS + Option Cost relation | O'Kane Ch 4.2.9 |
| DVOAS sensitivity definition | Tuckman Ch 14 |
| P&L attribution via OAS (carry + factor + convergence) | Tuckman Ch 14 |
| Asset swap spread definition | Tuckman Ch 18, O'Kane Ch 4.4 |
| Par asset swap formula | O'Kane Ch 4.4.2 |
| Asset swap intuition (long defaultable, short risk-free) | O'Kane Ch 4.4.2 |
| Market asset swap convention | O'Kane Ch 4.5 |
| TED spread definition (spread to futures rates) | Tuckman Ch 17 |
| TED spread as negative of OAS to futures | Tuckman Ch 17 |
| Credit spread decomposition (actuarial + risk premia + liquidity) | O'Kane Ch 3.11 |
| Coverage ratio and spread premium definitions | O'Kane Ch 3.11 |
| Coverage ratio by rating (Table 3.4) | O'Kane and Schloegl (2002) |
| On-the-run yields affected by liquidity/specialness | Tuckman Ch 15 |
| Liquidity premiums of 5-6 bp in 2y and 5y sectors | Tuckman Ch 15 |
| Asset swap spreads reflect security-specific effects | Tuckman Ch 18 |
| Swap spread history (1990s banking, late 1990s scarcity) | Tuckman Ch 18 |
| DV01 formula | O'Kane Ch 4 |
| Semiannual bond pricing formula | Tuckman Ch 3-4 |
| Expected excess return on bonds by rating | Hull Ch 24 Table 24.3 |

### (B) Reasoned Inference (Derived from A)

| Inference | Derivation |
|-----------|------------|
| G-spread inherits benchmark-selection sensitivities | Extended from Tuckman's swap spread benchmark discussion |
| I-spread inherits curve-definition sensitivities | Extended from Tuckman's swap spread discussion |
| CS01 is not universal | Follows from multiple spread definitions having different price sensitivities |
| Spread duration formula for Z-spread | Direct calculus on the continuous-form Z-spread PV equation |
| On-the-run yield contamination applies to G-spreads | Extended from Tuckman's discussion of swap spreads vs on-the-run Treasuries |
| Z-spread < G-spread on steep curves | Derived from Spot Rate vs Par Yield relationship (Spot > Par $\implies$ Z < G) |
| I-spread can exceed G-spread in negative swap spread regimes | Follows from swap spread definition and post-2008 market dynamics |

### (C) Flagged Uncertainties

- **None identified.** All content in this chapter traces to user-provided material backed by Tuckman, O'Kane, and Hull sources.
