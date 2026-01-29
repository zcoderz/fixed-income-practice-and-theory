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

We begin with the conceptual foundation—clean versus dirty prices and yield-to-maturity—then work through each spread definition in order of complexity: G-spread, I-spread, Z-spread, asset swap spread, TED spread, and OAS. We then explore which desks use which spreads, and conclude with O'Kane's decomposition of credit spreads into actuarial, default risk premium, and liquidity components. Finally, we preview the CDS-bond basis as a bridge to the credit chapters.

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

### 8.4.1 Negative Swap Spreads: When Traditional Relationships Invert

After the 2008 financial crisis, U.S. swap spreads at longer maturities (e.g., 30Y) notably turned negative, meaning Treasury yields exceeded swap rates. Tuckman provides historical context: "The fall in swap spreads in the early 1990s reflected the recovery of the banking sector from its problems in the 1980s. The rise in swap spreads in the late 1990s, on the other hand, can be best explained by a perceived scarcity in the supply of U.S. Treasury securities relative to demand."

Post-2008 negative swap spreads are driven by several factors:
- **Balance sheet costs**: Holding Treasuries consumes balance sheet capacity under Basel III regulations
- **Clearing advantages**: Cleared swaps have lower capital requirements than Treasury repo positions
- **Treasury scarcity as collateral**: High demand for Treasuries as high-quality collateral drives prices up (yields down)
- **Duration demand in swap format**: Insurance companies and pension funds can get duration exposure more efficiently via swaps

> **Desk Reality: Trading in a Negative Swap Spread World**
>
> When swap spreads are negative, traditional spread intuition breaks down:
> - A bond that looks "tight" on G-spread may look "wide" on I-spread
> - Hedging with swaps vs. hedging with Treasuries produces different P&L profiles
> - The basis between the two benchmarks becomes a tradeable quantity itself
>
> **Career tip:** When interviewing for a rates desk, demonstrating understanding of why swap spreads went negative shows sophisticated market knowledge.

> **History Lesson: Regime Change**
> *   **"Old Days" (Pre-2008)**: Treasuries were risk-free, Swaps were risky. I-Spread > G-Spread typically.
> *   **"New Days" (Post-2008)**: Treasuries are scarce collateral, Swaps are abundant. Swap spreads can be negative.
> *   **Key**: Spreads are relative. If the benchmark moves (swap spreads widen), your bond's I-spread might tighten even if its price didn't move!

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
>
> On an upward-sloping curve, the fitted **zero (spot) curve** and the **par-yield curve** differ because coupon bonds weight early cash flows more heavily than late cash flows. Z-spread is defined as a constant shift to the **zero** curve used to discount *each* cash flow, while G-spread is a difference between two **single** yields at one maturity. So Z-spread and G-spread can diverge on steep curves—especially for high-coupon/premium bonds.
>
> **Rule of thumb (not a theorem):** For many bullet bonds on steep upward-sloping curves, practitioners often observe **Z-spread below G-spread**, but the sign and magnitude depend on coupon, curve construction, and compounding conventions. Always validate by repricing.
>
> **Trading rule:** Never trust G-spread alone on a steep curve for high-premium bonds. The coupon effect distorts the comparison.
>
> **Sanity check:** If Z-spread and G-spread differ materially (e.g., tens of bp), check curve shape, compounding/day-count conventions, and whether your “Treasury curve” is fitted zeros or on-the-run yields.

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

### 8.6.3 Market Asset Swap

The standard par asset swap can present a **counterparty risk** to either party on initiation. O'Kane explains: "Since the asset swap buyer pays par in exchange for a bond worth $P$, they are exposed to a default by the asset swap counterparty if the bond is trading at a discount, i.e. $P < 1$. If the bond is trading at a premium $P > 1$, then the asset swap seller is taking the counterparty risk of the asset swap buyer."

For those who wish to avoid this counterparty risk, the **market asset swap** structure modifies the mechanics:

1. On settlement, the bond is delivered and the bond price $P$ is paid (not par)
2. The asset swap buyer enters into an interest rate swap paying fixed coupon $c/f$ on face value 1, receiving Libor plus the market asset swap spread $A^*(0)$ on face value $P$
3. At maturity, the floating leg pays $P$ while the fixed leg pays 1; the net payment to the buyer is $P - 1$

O'Kane derives the **market asset swap spread** formula:

$$\boxed{A^*(0) = \frac{A(0)}{P}}$$

where $A(0)$ is the equivalent par asset swap spread.

**Key insight:** "The market asset swap spread is higher for a discount bond than the par asset swap spread and is lower than the par asset swap spread for a premium bond."

> **Analogy: Car vs. Car + Warranty**
>
> Think of par vs. market asset swap like two ways to buy a used car:
> - **Par ASW**: You pay sticker price (par) regardless of the car's current market value. If the car is worth less, the dealer owes you the difference (counterparty risk on dealer)
> - **Market ASW**: You pay the car's current value. No upfront exposure, but at the end of the "warranty period," there's a residual payment
>
> The spread you "earn" differs because the capital at risk differs.

> **Desk Reality: When to Use Each Structure**
>
> - **Par asset swap:** Standard for quotes and screens; easier to compare across bonds
> - **Market asset swap:** Preferred when minimizing upfront counterparty exposure is critical
>
> For a discount bond with $P = 0.985$:
> - Par ASW spread = 185.2 bp
> - Market ASW spread = 185.2 / 0.985 = 188.0 bp
>
> The 2.8 bp difference compensates for the different counterparty risk profiles.

### 8.6.4 Asset Swap Mark-to-Market

Once an asset swap is initiated, how do we value it over time? O'Kane provides a clean formula for the **mark-to-market** of an asset swap position:

$$\boxed{MTM(t) = (A(0) - A(t)) \cdot PV01(t, T)}$$

where:
- $A(0)$ is the asset swap spread at initiation
- $A(t)$ is the current market asset swap spread for the remaining maturity
- $PV01(t, T)$ is the remaining annuity (present value of 1 bp per period)

**Intuition:** "The mark-to-market value of an asset swap is zero at initiation when $A(t) = A(0)$. The mark-to-market can then move away from zero depending on how the asset swap spread changes."

O'Kane provides a worked example: "Consider a fixed rate corporate bond with five years to maturity which pays an annualised coupon of 7.25%. The bond is currently trading at a price of $94.38 and we enter into an asset swap on a face value of $10 million at a spread of 323.9 bp. A year later, the bond is trading at a price of $96.00, a price increase of $1.62. Over the year, we also suppose that Libor swap rates have risen by 25 bp across the curve. Due to the shortening of the time to maturity by one year and the increase in Libor rates, the PV01 has fallen to 3.622. The new asset swap spread equals 284 bp. As a result, the mark-to-market of the asset swap position is:

$$MTM = (323.9 - 284.0) \text{ bp} \times 3.622 \times \$10\text{m} = \$144,518$$

What we have is a position in which the asset swap buyer is long the credit risk of the issuer, the credit quality of the issuer has improved, as shown by the increase in the bond price, and the position has made money."

> **Desk Reality: P&L from Asset Swap Positions**
>
> If you bought an asset swap at 185 bp and spreads tighten to 150 bp:
> - Your MTM gain = $(185 - 150) \times PV01 \times \text{Notional}$
> - With PV01 = 4.0 and Notional = $10mm: Gain = 35 bp × 4.0 × $10mm = $140,000
>
> This is the essence of being "long credit" via an asset swap—you profit when spreads tighten.

### 8.6.5 Asset Swap Risk: What If the Bond Defaults?

If the bond in the asset swap does **not** default, the asset swap package continues until maturity with the buyer simply passing through the bond coupons to the asset swap seller. In return, the asset swap buyer receives floating payments of Libor plus the asset swap spread. As O'Kane notes: "As they have paid par for this, provided the issuer does not default, this is equivalent to buying a floating rate note at par where the quoted margin equals the asset swap spread."

However, this equivalence breaks down if the bond **defaults**. O'Kane analyzes this scenario:

**If default occurs soon after initiation:**
- Asset swap buyer receives recovery $R$ from selling the defaulted bond
- Asset swap buyer can unwind the interest rate swap at value $V(\tau)$
- Since the investor paid par, and at initiation $V(0) + P = 1$ (so $V(0) = 1 - P$), the loss is:

$$\text{Loss} = 1 - (R + V(0)) = 1 - (R + (1 - P)) = P - R$$

Since $P > R$ (price exceeds recovery), the asset swap buyer makes a loss. Note that this loss is exactly the same as if the buyer had simply purchased the bond without any asset swap package.

**If default occurs later:**

O'Kane provides a crucial insight: "This simple analysis assumes that default happens soon after the start of the asset swap. This allowed us to assume that the value of the interest rate swap had not changed. However, in practice, when a default occurs, the value of the interest rate swap will have changed. As a result, the asset swap buyer can make a gain or loss due to interest rate movements if there is a default, i.e. we have a **default contingent interest rate risk**. This is different from the risk of a floating rate note."

> **Desk Reality: Asset Swap vs. FRN Risk Profile**
>
> Unlike a floating rate note where you simply lose $(1 - R)$ on default, an asset swap buyer has:
> - **Credit exposure:** Loss of $(P - R)$ if default happens early
> - **Interest rate exposure contingent on default timing:** If rates have moved since initiation, the swap leg may have gained or lost value
>
> **Key insight:** An asset swap buyer is long credit risk but also has interest rate exposure that manifests only if default occurs. This "default contingent interest rate risk" doesn't exist in a pure FRN position.

### 8.6.6 Market Convention Warning

O'Kane highlights an important ambiguity: "There are different asset swap market conventions, including 'market asset swap' where the floating leg notional equals the market value of the bond, reducing counterparty exposure versus a par-notional structure."

If someone quotes "ASW," confirm whether it is:
- **Par asset swap:** Notional is fixed at par (standard definition above)
- **Market asset swap:** Floating-leg notional equals bond market value

These produce different spread numbers for the same bond.

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

> **Desk Reality: OAS and Rich/Cheap Analysis**
>
> When a trader says "this callable is cheap by 10 bp OAS," they mean: even after accounting for the call option's value under the model, the bond offers 10 bp more spread than comparable issues.
>
> **Warning:** OAS is only as good as the model. Different volatility assumptions produce different OAS. Always ask: "What vol assumption?" A bond that looks cheap under one model may look fair under another.

### 8.8.3 DVOAS and P&L Attribution

Tuckman introduces **DVOAS** as the sensitivity of price to a one-basis-point change in OAS:

$$\text{DVOAS} \approx \frac{P(OAS - 1\text{ bp}) - P(OAS + 1\text{ bp})}{2}$$

For P&L attribution, Tuckman shows that a security's return can be decomposed as:

$$\boxed{dP = (r + OAS) \cdot P \cdot dt + DV01_x \cdot (dx - E[dx]) + DVOAS \times dOAS}$$

The three components are:
- **Carry:** $(r + OAS) \cdot P \cdot dt$ — time value plus OAS
- **Factor exposure:** $DV01_x \cdot (dx - E[dx])$ — unexpected rate moves
- **Convergence:** $DVOAS \times dOAS$ — OAS change toward fair value

Tuckman emphasizes: "The two decompositions highlight the usefulness of OAS as a measure of the value of a security with respect to a particular model. According to the model, **a long position in a cheap security earns superior returns in two ways. First, it earns the OAS over time intervals in which the security does not converge to its fair value. Second, it earns the DVOAS times the extent of any convergence.**"

For a hedged and financed position, assuming the factor exposure is hedged and financing cost equals the short rate $r$, the P&L simplifies to:

$$dP = OAS \times P \times dt + DVOAS \times dOAS$$

> **Desk Reality: OAS P&L Attribution Example**
>
> Suppose you own a callable bond with:
> - OAS = 20 bp (cheap to model)
> - DVOAS = $450 per bp (per $1mm notional)
> - Holding period = 3 months (0.25 years)
> - OAS converges to 10 bp over the period
>
> **Carry component:** $20 \text{ bp} \times 0.0001 \times \$1,000,000 \times 0.25 = \$500$
>
> **Convergence component:** $\$450 \times 10 \text{ bp} = \$4,500$
>
> **Total P&L:** $\$5,000$ from being long a cheap security
>
> This is the essence of OAS-based relative value: you earn both the spread over time and profit from mispricing correcting.

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

## 8.11 Which Desk Uses Which Spread?

For readers transitioning from middle office to front office, understanding which spread measure each desk uses—and why—is essential career knowledge.

> **Practitioner Note:** The following desk mapping comes from general market practice rather than the primary reference books. Conventions may vary by institution.

| Spread Type | Primary Users | Use Case | Why This Spread? |
|-------------|--------------|----------|------------------|
| **G-spread** | Treasury/Agency desk, auction analysts | Pure govt relative value, auction richness/cheapness | Direct comparison to risk-free; intuitive for govt specialists |
| **I-spread** | Bank ALM, swap desk, liability hedgers | Funding relative value, swap-hedged positions | Swap curve is the natural funding benchmark for banks |
| **Z-spread** | Quant analytics, credit research, pricing models | Precision valuation, model calibration | Term-structure consistent; better for high-coupon/steep curve situations |
| **OAS** | MBS desk, callable bond desk, mortgage traders | Option-heavy products, prepayment modeling | Separates option value from spread; essential for MBS |
| **Asset swap spread** | Credit trading desk, relative value | Hedged P&L, basis trades, cross-market comparison | Represents "what you earn" after hedging rate risk |
| **TED spread** | Agencies desk, short-duration credit | Futures hedging, money market relative value | Directly hedgeable with liquid futures contracts |

> **Desk Reality: Why Each Desk Prefers Its Spread**
>
> - **Treasury desk** uses G-spread because their benchmark is the government curve; comparing anything to swaps makes no sense for pure rates trading
> - **Credit desk** uses asset swap spread because they hedge rate risk with swaps; ASW is their "true" spread exposure after the hedge
> - **MBS desk** must use OAS because without removing the prepayment option value, spreads are meaningless; a "cheap" MBS on Z-spread may be "fair" on OAS
> - **Quant team** uses Z-spread for model calibration because it's mathematically cleaner than yield spreads and doesn't require an option model
>
> **Interview tip:** If asked "what spread would you use for X?" on a desk interview, think about what benchmark that desk naturally hedges against.

### 8.11.1 Spread Choice Matters for P&L Attribution

When your risk system reports "spread P&L," it's using some spread definition. If the desk thinks in G-spread but the system uses Z-spread, the P&L attribution will be confusing—especially on steep curves or for high-coupon bonds where the two diverge.

> **Example: Same Bond, Different Spreads**
>
> A credit analyst looking at a 10Y BBB corporate might see:
> - G-spread: 195 bp
> - I-spread: 175 bp
> - Z-spread: 180 bp
> - ASW spread: 185 bp
>
> All are "correct" given their definitions. But if the analyst is comparing to another bond using a different spread type, they may reach wrong conclusions.
>
> **Rule:** Always compare like with like. Don't mix G-spreads from one source with Z-spreads from another.

---

## 8.12 Worked Examples

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

### Example E: Par Asset Swap Spread

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

### Example G: Market Asset Swap Spread

**Given:** Same bond as Example E
- Par ASW spread $A(0) = 185.2$ bp
- Dirty price $P = 0.9850$ (as fraction of par)

**Market asset swap spread:**

$$A^*(0) = \frac{A(0)}{P} = \frac{185.2}{0.9850} = \boxed{188.0 \text{ bp}}$$

**Interpretation:** The market ASW spread is 2.8 bp higher than the par ASW spread because this is a discount bond. The market structure shifts counterparty exposure from initiation to maturity.

### Example H: Asset Swap MTM

**Given:**
- Asset swap initiated at spread $A(0) = 185$ bp
- Current market spread for remaining maturity: $A(t) = 160$ bp
- Remaining PV01 = 3.50
- Notional = $10 million

**Mark-to-market:**

$$MTM = (185 - 160) \times 0.0001 \times 3.50 \times \$10,000,000 = 25 \times 0.0001 \times 3.50 \times \$10mm$$
$$MTM = \boxed{\$87,500}$$

**Interpretation:** Spreads tightened by 25 bp; the asset swap buyer (long credit) has a gain of $87,500.

---

## 8.13 Preview: The CDS-Bond Basis

As we transition toward the credit chapters (Part IX), it's important to understand that CDS spreads and bond spreads do not always agree. The difference between them—the **CDS-bond basis**—is a fundamental concept in credit relative value.

O'Kane defines:

$$\boxed{\text{CDS basis} = \text{CDS spread} - \text{Bond Libor spread (ASW)}}$$

For a fixed-rate bond, the natural choice for bond spread is the asset swap spread. For floating-rate bonds, it's the par floater spread.

The basis can be **positive** (CDS wider than cash) or **negative** (CDS tighter than cash), depending on market conditions.

### 8.13.1 Fundamental Factors Driving the Basis

O'Kane identifies several **fundamental factors** that relate to contractual differences:

1. **Funding:** CDS are unfunded transactions; bonds are funded. For the same spread, CDS are favored by investors with funding costs above Libor, while bonds are favored by those who fund below Libor. This affects the equilibrium basis.

2. **Delivery option:** In a CDS, the protection buyer can choose which obligation to deliver from a basket. This option has value and should widen CDS spreads relative to bonds.

3. **Technical default:** CDS credit events may be broader than bond defaults (depending on restructuring clause). Protection sellers demand higher spreads, widening the basis.

4. **Loss on default:** CDS pays $(1-R)$ on face value. A bond purchased at price $P$ loses $(P-R)$. For discount bonds, the loss differs—this affects fair value spreads.

5. **Premium accrued at default:** CDS pays accrued premium to the protection seller following a credit event. Bond owners lose accrued coupon. This lowers CDS spreads and reduces the basis.

6. **Floor at zero:** CDS spreads cannot go negative (you can't get paid to take default risk). Asset swap spreads can be negative for very high-quality credits (e.g., Treasuries can have negative ASW spreads).

### 8.13.2 Market Factors Driving the Basis

O'Kane also identifies **market factors**:

1. **Relative liquidity:** CDS liquidity concentrates at standard maturities (3Y, 5Y, 7Y, 10Y); bond liquidity depends on issuance.

2. **Synthetic CDO hedging:** When dealers issue synthetic CDOs, they sell CDS protection to hedge, tightening CDS spreads and reducing the basis.

3. **New issuance dynamics:** Bond issuance can temporarily widen CDS as investors hedge new positions.

4. **Demand for protection:** It's easier to short credit via CDS than via bonds; negative news drives CDS wider first.

5. **Funding risk:** CDS locks in Libor flat funding; bonds expose you to funding uncertainty.

> **Desk Reality: The Classic Negative Basis Trade**
>
> When the basis is **negative** (CDS tighter than cash), a classic trade is:
> - Buy the bond (long credit via cash)
> - Buy CDS protection (short credit via derivatives)
>
> If funded at Libor flat, this is positive carry with minimal net credit exposure. The trade bets on basis convergence while collecting spread.
>
> **Warning:** This trade has hidden risks—liquidity, counterparty exposure, funding cost changes, and the possibility of basis widening further before converging.
>
> **See Chapter 44** for detailed treatment of CDS relative value and basis trading strategies.

---

## 8.14 Practical Notes

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
9. **Asset swap spread:** Converts fixed bond to floating economics; par vs market conventions differ. MTM depends on spread changes × remaining PV01.
10. **ZVS = OAS + option cost** for callable bonds
11. **Market spreads embed multiple components:** actuarial expected loss, default risk premium, volatility risk premium, liquidity premium—not just probability of default
12. **CDS-bond basis:** CDS spread minus bond Libor spread; can be positive or negative; driven by fundamental and market factors

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **G-spread** | $y_{\text{bond}} - y_{\text{gov}}(T)$ | Quick relative value vs governments; benchmark-dependent |
| **I-spread** | $y_{\text{bond}} - y_{\text{swap}}(T)$ | Relative value vs swaps; curve definition matters |
| **Z-spread** | Constant spread to benchmark discounting matching dirty price | Term-structure consistent; for non-callable bonds |
| **TED spread** | Spread to futures rates that reprices bond | Hedgeable with liquid futures contracts |
| **OAS** | Spread to model rates so model price = market price | For callable/option bonds; model-dependent |
| **Par asset swap spread** | $(P_{\text{Libor}} - P)/PV01$ | Spread-to-swaps; standard quote convention |
| **Market asset swap spread** | $A(0)/P$ | Reduces counterparty exposure vs par structure |
| **Asset swap MTM** | $(A(0) - A(t)) \times PV01$ | P&L from spread changes |
| **CS01** | $\partial P / \partial s \times 0.0001$ | Spread sensitivity; depends on spread definition |
| **Spread duration** | $-\frac{1}{P}\frac{\partial P}{\partial s}$ | PV-weighted average time under spreaded discounting |
| **Coverage ratio** | Credit spread / Actuarial spread | How much spreads exceed pure default compensation |
| **CDS-bond basis** | CDS spread − Bond Libor spread | Fundamental RV measure for credit trading |

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
| $A(0)$ | Par asset swap spread |
| $A^*(0)$ | Market asset swap spread |
| $MTM$ | Mark-to-market value |
| DVOAS | Sensitivity of price to OAS change |

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
| 28 | Market asset swap spread formula? | $A^*(0) = A(0)/P$ |
| 29 | Asset swap MTM formula? | $MTM(t) = (A(0) - A(t)) \times PV01(t,T)$ |
| 30 | What is CDS-bond basis? | CDS spread minus bond Libor spread (ASW) |
| 31 | Trading rule for steep curves? | Never trust G-spread alone for premium bonds; check Z-spread |
| 32 | Which desk uses Z-spread vs OAS? | Quant/credit research uses Z-spread; MBS/callable desk uses OAS |
| 33 | What is "default contingent interest rate risk"? | Interest rate exposure that manifests only if default occurs (in asset swaps) |
| 34 | What happens when swap spreads are negative? | I-spread can exceed G-spread; traditional relationships invert |

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

**13.** A bond has par asset swap spread of 200 bp and trades at a dirty price of 95.00 (per 100 par). What is the market asset swap spread?

**14.** You entered an asset swap at 175 bp. The current market spread is 150 bp and remaining PV01 = 4.2. On $5mm notional, what is your MTM?

**15.** Explain why CDS-bond basis can be either positive or negative. Give one factor that pushes it each direction.

**16.** A credit analyst observes: G-spread = 280 bp, Z-spread = 230 bp (50 bp difference). What does this suggest about the curve shape and the bond's coupon?

### Solution Sketches (1-10)

**1.** $P_{\text{dirty}} = 101.20 + 0.80 = 102.00$

**2.** Coupon per period $= 0.06/2 \times 100 = 3.00$. $AI = 0.40 \times 3.00 = 1.20$

**3.** Solve $99 = 5/(1+y) + 105/(1+y)^2$. At $y = 6\%$: PV = 98.18 (low). At $y = 5\%$: PV = 100 (high). So $y \approx 5.5\%$.

**4.** $s_G = 7.10\% - 4.85\% = 2.25\% = 225$ bp

**5.** Bracket with $s = 1\%$ (PV ≈ 99), $s = 2\%$ (PV ≈ 96). Bisect to $s_Z \approx 1.37\% = 137$ bp. (Exact 136.6 bp).

**6.** Yield spread compares one yield to one yield, ignoring that different cash flows should be discounted at different maturities. Z-spread revalues each cash flow using the full curve. On a steep curve, spot rates are often higher than par yields. Since Z-spread adds to the (higher) spot curve while G-spread adds to the (lower) par yield, Z-spread is often lower than G-spread for bullet bonds.

**7.** $\Delta P \approx -P \times D_s \times \Delta s = -100 \times 4.5 \times 0.0010 = -0.45$

**8.** Fitted Treasury zeros are more appropriate because: (a) Z-spread requires discount factors at each cash flow date, and fitted zeros provide these directly; (b) on-the-run yields can be distorted by repo specialness and liquidity effects that don't reflect the true risk-free rate.

**9.** Three clarifications needed: (a) Which swap curve—OIS, SOFR, or legacy LIBOR construction? (b) What interpolation method for off-market maturities? (c) Clean or dirty price basis for the bond yield?

**10.** If matching clean price instead of dirty (and accrued interest is positive), the spread will be biased **higher**. The engine is trying to fit a *lower* target price, which requires more discounting (larger spread). (If the bond is in an ex-dividend convention where accrued can be negative, the sign can flip—always check the convention.)

**11.** Z-spread **larger** than OAS. For a callable bond, the investor is short the call option. The option has positive value to the issuer, so the bond price is lower than an equivalent non-callable. The Z-spread must be higher to match this lower price. OAS removes the option value, leaving only the credit/liquidity spread. Hence ZVS = OAS + option cost, so ZVS > OAS.

**12.** Different spreads can arise from: liquidity differences, repo specialness, different coupons (yield spread distortion), supply/demand imbalances, maturity differences, settlement timing, or delivery option value in CDS basis trades.

**13.** Market ASW spread = Par ASW / P = 200 / 0.95 = 210.5 bp

**14.** MTM = (175 - 150) × 0.0001 × 4.2 × $5,000,000 = 25 × 0.0001 × 4.2 × $5mm = $52,500 gain

---

## References

- Dominic O’Kane, *Modeling Single-name and Multi-name Credit Derivatives* (yield spreads vs Z-spread; asset swap spreads; credit spread decomposition; CDS–bond basis).
- Bruce Tuckman, *Fixed Income Securities* (limits of yield summaries; OAS framework; benchmark choice and on-the-run distortions; spread-based attribution).
- John C. Hull, *Options, Futures, and Other Derivatives* (bond yields and credit spread interpretation; conventions and practical cautions).
