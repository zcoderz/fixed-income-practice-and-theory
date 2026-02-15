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

Prerequisites: [Chapter 5 — Fixed-Rate Bond Pricing](chapter_05_fixed_rate_bond_pricing.md), [Chapter 6 — YTM & Yield-Based Risk](chapter_06_ytm_yield_based_risk.md), [Chapter 7 — Bond Return Decomposition](chapter_07_bond_return_decomposition.md)  
Follow-on: [Chapter 11 — DV01/PV01 and bump design](chapter_11_dv01_pv01_definitions_computation.md), [Chapter 17 — Curve construction](chapter_17_curve_construction_bootstrapping_interpolation.md), [Chapter 27 — Swap spreads and asset swaps](chapter_27_swap_spreads_asset_swaps_swap_curve_rv.md), [Chapter 37 — Cash credit spreads and CS01](chapter_37_cash_credit_risky_bonds_spreads_cs01.md), [Chapter 44 — CDS relative value and basis](chapter_44_cds_relative_value_trading_frameworks.md)

## Learning Objectives
- After this chapter, you can name and compute common spread measures (G-spread, I-spread, Z-spread, OAS, asset swap spread) and state the benchmark curve and compounding convention.
- You can translate a spread quote into a PV equation that matches the dirty (invoice) price.
- You can compute a Z-spread from discount factors and perform a repricing check.
- You can compute and interpret spread duration and CS01 with an explicit bump object, units, and sign convention.
- You can diagnose common “spread mismatch” issues (benchmark choice, clean/dirty, compounding, and embedded options/model dependence).

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

**Benchmark selection matters.** In practice, $y_{\text{gov}}(T)$ might refer to:
- An **on-the-run** government yield quote at maturity $T$, or
- A **fitted / interpolated** government curve used to produce benchmark yields at arbitrary maturities.

Different benchmark choices (and different interpolation/curve-fitting choices) can produce different $s_G$ values. Always state your benchmark and construction method when comparing spreads across sources.

---

## 8.4 I-spread: Spread to Swap Curve

The **I-spread** (interpolated swap spread or ISDA spread) is the yield spread versus a swap curve:

$$\boxed{s_I(T) = y_{\text{bond}} - y_{\text{swap}}(T)}$$

where $y_{\text{swap}}(T)$ is typically the par swap rate at maturity $T$.

**When to use I-spread:** I-spread is natural when your benchmark (or hedge language) is a swap curve rather than a government curve.

**Which swap curve?** You must clarify:
- Which curve family (OIS/overnight vs term index curve, etc.)
- The curve construction/interpolation method used to get $y_{\text{swap}}(T)$ at off-market maturities

The difficulties are analogous to G-spread: different benchmark curves produce different I-spreads.

**Comparison:** For the same bond, G-spread and I-spread will differ because government yields and swap yields differ. This is not a calculation error—it is a definitional difference.

### 8.4.1 When I-spread Can Exceed G-spread (Pure Algebra)

Define the **swap spread** at maturity $T$ as:
$$SS(T) := y_{\text{swap}}(T) - y_{\text{gov}}(T)$$

Then:
$$s_G(T) - s_I(T) = \bigl(y_{\text{bond}} - y_{\text{gov}}(T)\bigr) - \bigl(y_{\text{bond}} - y_{\text{swap}}(T)\bigr) = SS(T)$$

So:
- If $SS(T) > 0$, then $s_I(T) < s_G(T)$.
- If $SS(T) < 0$, then $s_I(T) > s_G(T)$.

This is why the same bond can look “tight” on one benchmark and “wide” on another, even without any change in the bond’s own cash flows.

> **Desk Reality:** Risk reports often label “I-spread” or “swap spread” without stating the curve family and construction.
> **Common break:** Two systems report different I-spreads for the same bond because they use different swap curves or interpolation.
> **What to check:** Recreate $s_I(T)=y_{\text{bond}}-y_{\text{swap}}(T)$ with the exact curve inputs and confirm the repricing target (clean vs dirty).

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

O'Kane notes that practitioners sometimes prefer Z-spread to simple yield spreads because it explicitly incorporates the term structure when discounting each cash flow.

> **Deep Dive: Direction of Bias**
>
> On an upward-sloping curve, the fitted **zero (spot) curve** and the **par-yield curve** differ because coupon bonds weight early cash flows more heavily than late cash flows. G-spread is a difference between two **single** yields at one maturity, while Z-spread is a constant shift to the **spot/discount curve** used to PV *each* cash flow. That (plus coupon effects) is why Z-spread and G-spread can diverge on steep curves—sometimes materially.
>
> **Sanity check:** If Z-spread and G-spread differ a lot, verify benchmark curve construction, clean vs dirty repricing target, and compounding/day-count conventions. Example D shows a concrete case where the two are noticeably different.

---

## 8.6 Asset Swap Spread

### 8.6.1 What Is an Asset Swap?

An **asset swap** packages a bond with an interest rate swap so that (to first order) the bond’s fixed coupons are converted into **floating-rate economics**.

One common “par asset swap” structure is:
- Buy the fixed-rate bond (receive its coupons and principal).
- Enter a payer-fixed swap whose fixed leg matches the bond’s coupon schedule.
- Receive floating (e.g., the swap index) **plus an asset swap spread** on the swap’s floating leg.

Because the bond’s fixed coupons and the swap’s fixed payments are aligned, the fixed-rate cashflows largely cancel. What remains is a *spread-to-swaps* measure: how cheap/rich the bond is versus the swap curve, expressed as a running spread.

### 8.6.2 Par Asset Swap Formula

At initiation, the **par asset swap spread** $A(0)$ is set so that the *bond + swap package* is worth par (zero PV net of the funding assumption embedded in the curve you use).

$$\boxed{A(0) = \frac{P_{\text{Libor}} - P}{PV01(0,T)}}$$

where:
- $P$ is the bond's dirty price **per 1 of par** (e.g., $98.50$ per 100 corresponds to $P=0.9850$)
- $P_{\text{Libor}}$ is the PV of the bond's fixed cash flows discounted on the chosen swap/Libor curve (often written using swap discount factors $Z(0,\cdot)$)
- $PV01(0,T) = \sum_{m=1}^{M} Z(0, t_m) \cdot \Delta(t_{m-1}, t_m)$ is the fixed-leg annuity (units: years) for the swap payment dates $t_m$

**Interpretation:** If $P < P_{\text{Libor}}$ (bond is cheap versus the swap curve), then $A(0) > 0$. If $P > P_{\text{Libor}}$ (bond is rich), $A(0)$ can be negative.

### 8.6.3 Market Asset Swap

The standard **par** asset swap can present **counterparty risk** at initiation. Since the asset swap buyer pays par in exchange for a bond worth $P$, the buyer is exposed to counterparty default when the bond is trading at a discount ($P<1$); if the bond is trading at a premium ($P>1$), the seller is exposed.

For those who wish to avoid this counterparty risk, the **market asset swap** structure modifies the mechanics:

1. On settlement, the bond is delivered and the bond price $P$ is paid (not par)
2. The asset swap buyer enters into an interest rate swap paying fixed coupon $c/f$ on face value 1, receiving Libor plus the market asset swap spread $A^*(0)$ on face value $P$
3. At maturity, the floating leg pays $P$ while the fixed leg pays 1; the net payment to the buyer is $P - 1$

In O’Kane’s derivation, the **market asset swap spread** is:

$$\boxed{A^*(0) = \frac{A(0)}{P}}$$

where $A(0)$ is the equivalent par asset swap spread.

**Implication:** Discount bonds ($P<1$) have $A^*(0) > A(0)$, and premium bonds ($P>1$) have $A^*(0) < A(0)$.

> **Desk Reality:** Many screens quote “ASW” without specifying par-vs-market structure.
> **Common break:** Comparing $A(0)$ to $A^*(0)$ across sources without noticing that $P$ enters the definition.
> **What to check:** Confirm (i) par vs market ASW, and (ii) whether $P$ is “per 1” or “per 100”.

### 8.6.4 Asset Swap Mark-to-Market

Once an asset swap is initiated, a common spread-only mark-to-market approximation is:

$$\boxed{MTM(t) = (A(0) - A(t)) \cdot PV01(t, T)}$$

where:
- $A(0)$ is the asset swap spread at initiation
- $A(t)$ is the current market asset swap spread for the remaining maturity
- $PV01(t, T)$ is the remaining annuity (present value of 1 bp per period)

To convert this into a **currency** amount for notional $N$ (and $A$ quoted in bp), use a unit-safe form:
$$\boxed{MTM_{\$}(t)\approx (A(0)-A(t))_{\text{bp}}\times 10^{-4}\times PV01(t,T)\times N}$$

**Worked number (O’Kane’s example):**
- Entered at $A(0)=323.9$ bp on $N=\$10{,}000{,}000$.
- One year later: $A(t)=284.0$ bp and remaining $PV01(t,T)=3.622$.
- Then:

$$MTM_{\$}\approx (323.9-284.0)\times 10^{-4}\times 3.622\times \$10{,}000{,}000\approx \$144{,}518$$

This is the desk meaning of being “long credit via ASW”: you make money when the ASW spread tightens (all else equal).

> **Desk Reality: P&L from Asset Swap Positions**
>
> If you bought an asset swap at 185 bp and spreads tighten to 150 bp:
> - Your MTM gain is approximately $(185-150)_{\text{bp}}\times 10^{-4}\times PV01\times N$.
> - With $PV01=4.0$ and $N=\$10{,}000{,}000$: gain $\approx 35\times 10^{-4}\times 4.0\times \$10{,}000{,}000=\$140{,}000$.
>
> This is the essence of being "long credit" via an asset swap—you profit when spreads tighten.

### 8.6.5 Asset Swap Risk: What If the Bond Defaults?

If the issuer does **not** default, the asset swap package is designed so that the bond’s fixed coupons are offset by the swap’s fixed leg, leaving (approximately) a floating-rate exposure plus a running spread.

However, this equivalence breaks down if the bond **defaults**. O’Kane analyzes this scenario.

**If default occurs soon after initiation:**
- Asset swap buyer receives recovery $R$ from selling the defaulted bond.
- Asset swap buyer can unwind the interest rate swap at value $V(\tau)$.
- On the settlement date of the asset swap, $V(0)+P=1$, so $V(0)=1-P$ and the loss to the investor is $(P-R)$ (here $P$ is the bond price per 1 of face value and $R$ is recovery per 1 of face value):

$$\text{Loss} = 1 - (R + V(0)) = 1 - (R + (1 - P)) = P - R$$

Note that the size of the default loss is exactly the same as if the asset swap buyer had simply purchased the bond without any asset swap package.

**If default occurs later:**

If the bond defaults later, interest rate effects come into play as discussed earlier in this section.

This asset swap can be compared to buying a par floater if the price of the fixed rate bond is also par, that is, $P=1$. In this case the value of the interest rate swap is zero at the start of the asset swap and should remain close to zero over the life of the asset swap provided there are no large interest rate movements.

> **Desk Reality: Asset Swap vs. FRN Risk Profile**
>
> Unlike a floating rate note purchased at par where you lose $(1-R)$ on default, an asset swap buyer can face:
> - **Bond-like default loss:** Early default loss magnitude of $(P-R)$ (same as owning the bond).
> - **Path dependence through rates:** If default occurs later, interest rate effects can come into play, so do not treat the package as identical to a floater without checking assumptions.
>
> **Key insight:** Asset swaps are “float + spread” economics *until default*; after that, the realized outcome can differ from a simple FRN comparison.

### 8.6.6 Market Convention Warning

Asset swap quoting conventions vary, and “ASW” can mean different structures in different contexts.

If someone quotes "ASW," confirm whether it is:
- **Par asset swap:** Notional is fixed at par (standard definition above)
- **Market asset swap:** Floating-leg notional equals bond market value

These produce different spread numbers for the same bond.

---

## 8.7 TED Spread: Spread to Futures Rates

### 8.7.1 Definition and Purpose

Tuckman defines **TED spreads** as using rates implied by Eurodollar futures to assess the value of a security relative to Eurodollar futures rates (or relative to another security).

He notes that TED spreads were originally used to compare Treasury bill futures (no longer actively traded) and Eurodollar futures, and that “the name came from the combination of T for Treasury and ED for Eurodollar.”

The idea is to find the spread such that “discounting cash flows at Eurodollar futures rates minus that spread produces the security’s market price.”

He motivates this benchmark choice by arguing that Eurodollar futures are often (though not always) treated as fairly priced because they are “quite liquid” and “immune to many individual security effects.”

### 8.7.2 Calculation Approach

Operationally, you use rates implied by Eurodollar futures to discount the bond’s cash flows, and you solve for the spread $s_{\text{TED}}$ such that:

$$P_{\text{dirty}} = \sum_k CF_k \cdot DF_k(s_{\text{TED}})$$

where $DF_k(s_{\text{TED}})$ denotes discounting at “futures-implied rates minus $s_{\text{TED}}$” for each period.

In Tuckman’s example, the interpretation of a TED spread of 15.6 bp is that “the agency is 15.6 basis points rich to LIBOR as measured by the futures rates.”

### 8.7.3 Theoretical Caveat

Tuckman highlights an obvious theoretical caveat: discounting should be done at forward rates, not futures rates, because futures and forwards differ (the futures-forward/convexity adjustment).

He also notes that for futures expiring shortly, the difference between forward and futures rates is relatively small, so using futures rates for discounting can be “relatively accurate” for short-maturity bonds.

---

## 8.8 OAS: Option-Adjusted Spread

### 8.8.1 Definition

Hull notes that, in addition to computing theoretical prices for mortgage-backed securities and other bonds with embedded options, traders compute what is known as the **option-adjusted spread** (OAS). This is a measure of the spread over the yields on government Treasury bonds provided by the instrument when all options have been taken into account. To calculate an OAS for an instrument, it is priced as described above using Treasury rates plus a spread for discounting. The price of the instrument given by the model is compared to the price in the market. A series of iterations is then used to determine the value of the spread that causes the model price to be equal to the market price.

In Tuckman’s tree framework, the OAS is the spread that—when added to all the short rates in the risk-neutral tree for discounting purposes—produces a model price equal to the market price.

O’Kane defines OAS as the fixed spread over the Libor (or government) discount rate which reprices the bond. The concept of the OAS has its origins in the callable bond market. Originally, the OAS was used as a way to quantify the spread impact of a call option embedded in a fixed rate bond. However, in the world of credit, the same measure is used to quantify the spread over the Libor curve due to the embedded credit risk where no optionality is present. For this reason, the name **zero volatility spread (ZVS)** is preferred as it makes clear that it is not a volatility measure, even though it is calculated in exactly the same way.

### 8.8.2 Model Dependence

Tuckman emphasizes that OAS is a measure of the value of a security with respect to a particular model.

> **Desk Reality: OAS and “Cheap/Rich” (Model-Relative)**
>
> In Tuckman’s example, an OAS of 10 bp means the security is **10 bp cheap** relative to the model; if the OAS were negative, the security would be **rich**.
>
> This is model-relative: OAS is a measure of value with respect to a particular model.

### 8.8.3 DVOAS and P&L Attribution

Tuckman calls the sensitivity of model price to a one-basis-point decrease in OAS **DVOAS**. He computes it analogously to DV01: reprice at OAS $\pm 1$ bp and divide the price change by 2:

$$\text{DVOAS} \approx \frac{P(OAS - 1\text{ bp}) - P(OAS + 1\text{ bp})}{2}$$

Tuckman also shows a P&L decomposition that makes OAS desk-usable:

$$\boxed{d P=(r+\mathrm{OAS}) P\, d t+\mathrm{DV01}_{x}(d x-E[d x])+\mathrm{DVOAS} \times d \,\mathrm{OAS}}$$

The three components are:
- **Carry:** $(r + OAS) \cdot P \cdot dt$ — time value plus OAS
- **Factor exposure:** $DV01_x \cdot (dx - E[dx])$ — unexpected rate moves
- **Convergence:** $DVOAS \times dOAS$ — OAS change toward fair value

For a hedged and financed position, Tuckman shows the P&L simplifies to:

$$d P=\mathrm{OAS} \times P\, d t+\mathrm{DVOAS} \times d \,\mathrm{OAS}$$

> **Desk Reality: OAS P&L Attribution Example**
>
> Suppose you own a callable bond with:
> - OAS = 20 bp (cheap to model)
> - DVOAS = \$450 per 1 bp (per \$1mm notional)
> - Holding period = 3 months (0.25 years)
> - OAS converges to 10 bp over the period
>
> **Carry component:** $20 \text{ bp} \times 0.0001 \times \$1,000,000 \times 0.25 = \$500$
>
> **Convergence component:** $\$450 \times 10 \text{ bp} = \$4,500$
>
> **Total P&L:** $\$5,000$ from being long a cheap security
>
> In words, for a financed and hedged position, the profit comes from OAS carry plus any profit from convergence to fair value.

---

## 8.9 Spread Duration and CS01

### 8.9.1 Spread Duration

Spread duration is the spread analogue of modified duration: it is the *first-order percentage* price sensitivity to changes in a chosen spread parameter $s$ (Z-spread, OAS, ASW spread, etc.), holding the benchmark curve/model fixed.

A convenient first-order approximation is:
$$\frac{\Delta P}{P} \approx -D_s\,\Delta s$$

$$\boxed{D_s := -\frac{1}{P}\frac{\partial P}{\partial s}}$$

This sign convention is consistent with the way O’Kane writes first-order risk: the percentage price change includes a term of the form $-D_s\,ds$ (so $D_s$ is typically positive for a fixed-coupon bond when $s$ is a discount-rate spread).

For Z-spread with continuous compounding, where $P(s) = \sum_k CF_k \cdot P_{\text{bench}}(0, t_k) \cdot e^{-s t_k}$:

$$\frac{\partial P}{\partial s} = -\sum_k t_k \cdot CF_k \cdot P_{\text{bench}}(0, t_k) \cdot e^{-s t_k}$$

so:

$$D_s = \frac{\sum_k t_k \cdot PV_k(s)}{\sum_k PV_k(s)}$$

This is a PV-weighted average time—similar in structure to Macaulay duration but computed under the spreaded discounting.

### 8.9.2 CS01 (Credit Spread 01)

Pick a spread parameter $s$ (e.g., Z-spread $z$, OAS, or an asset swap spread quote). Hold the benchmark curve/model fixed and hold contractual cashflows fixed.

**Book convention (positive for long credit):**
$$\boxed{CS01 := P(s) - P(s+1\text{ bp})}$$

where $1\text{ bp} = 10^{-4}$ in the units of $s$ (a per-year spread parameter).

**Approximation via spread duration:** Using $D_s := -\frac{1}{P}\frac{\partial P}{\partial s}$,
$$CS01 \approx -\frac{\partial P}{\partial s}\cdot 10^{-4} = P \cdot D_s \cdot 10^{-4}$$

This parallels the familiar yield relationship $DV01 \approx \frac{P\times D_{\text{Mod}}}{10{,}000}$: you get an absolute “01” by multiplying a percentage-duration by price and dividing by $10{,}000$.

**Units:** If $P$ is “price points per 100 notional”, then CS01 is “price points per 100 per 1bp”. For currency units, multiply by notional (and by $0.01$ if you convert price points per 100 into a currency amount).

> **Pitfall — CS01 sign and bump direction drift:** Different systems define CS01 with different bump directions (+1bp widening vs -1bp tightening) and different sign conventions.
> **Why it matters:** You can hedge the *wrong way* if one report treats “CS01 = +\\$X” as profit-on-widening while another treats it as loss-on-widening.
> **Quick check:** Bump your spread parameter by **+1bp** holding the benchmark fixed. Price should go **down**. Under the convention here, CS01 should be **positive** because it reports the magnitude of that loss.

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

In this table, coverage ratios range from about $3.1$ to $5.5$, so the market spread exceeds the actuarial spread by several multiples. The spread premium (the non-actuarial component) grows as we move down the rating spectrum in these buckets.

In this table, the **coverage ratio** is on the order of $3$–$6$ for these rating buckets, which is one concrete way to see that traded spreads generally embed more than “expected loss”.

### 8.10.3 Implication for Spread Analysis

Even a “spread-to-swaps” measure (like ASW) can embed non-credit components: funding assumptions, security-specific financing, and liquidity/technicals. When two bonds from the same issuer trade at different spreads, the difference is not automatically a “credit view” difference.

---

## 8.11 Which Desk Uses Which Spread?

For readers transitioning from middle office to front office, the key is to recognize that desks tend to prefer the spread measure that matches their **benchmark and hedge language**.

> **Practitioner Note:** The following desk mapping comes from general market practice rather than the primary reference books. Conventions may vary by institution.

| Spread Type | Common Context | Use Case | Why This Spread? |
|-------------|--------------|----------|------------------|
| **G-spread** | Government-benchmark commentary | Govt-relative value, benchmark comparisons | Directly references a government curve |
| **I-spread** | Swap-benchmark commentary | Swap-hedged comparisons, swap-curve benchmarking | Directly references a swap curve; curve choice matters |
| **Z-spread** | PV-based valuation | Term-structure-consistent repricing | Defined by a PV match to the benchmark discount curve |
| **OAS** | Option-embedded products | Callable bonds, MBS | Requires an option model; depends on vol/model assumptions |
| **Asset swap spread** | Spread-to-swaps framing | Rate-hedged credit positions | Converts fixed bond economics into floating + spread framing |
| **TED spread** | Futures-benchmark framing | Hedgeable comparisons using futures-implied rates | Anchors the spread to liquid futures benchmarks |

> **Desk Reality:** “What spread should I use?” is often answered by “what benchmark do I hedge against, and does the bond have embedded options?”
> **Common break:** A desk talks in G-spread while the risk/P&L system reports Z-spread or OAS-based measures, creating confusing attribution.
> **What to check:** Ask (i) benchmark curve, (ii) spread definition, and (iii) whether the measure is option-adjusted.

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

### Example C: Z-spread, spread duration, and CS01 (template example)

**Example Title**: Z-spread and CS01 from benchmark discount factors

**Context**
- Price a plain-vanilla bullet bond by solving for its Z-spread versus a given benchmark discount curve.
- Convert that spread into a desk-usable *risk number* (CS01) with an explicit bump object, units, and sign.

**Timeline (Make Dates Concrete)**
- Trade date: 2026-02-16
- Settlement date: 2026-02-16 (assume settlement occurs on a coupon date, so $AI=0$)
- Payment dates: 2027-02-16, 2028-02-16, 2029-02-16 (annual)
- Day count: treat year-fractions as 1, 2, 3 for pedagogy (real implementations use the bond’s day count)

**Inputs**
- Bond: 3-year maturity, annual coupon 5%, notional 100, bullet principal.
- Dirty price: $P_{\text{dirty}} = 98.00$ (price points per 100 notional).
- Benchmark discount factors: $P(0,1)=0.97$, $P(0,2)=0.93$, $P(0,3)=0.88$.
- Z-spread convention: a continuously-compounded spread parameter $z$ applied to discounting:
  $$DF^{(z)}(0,t)=P(0,t)e^{-zt}$$
- Risk bump (for CS01): widen the spread parameter by $+1\text{ bp}$, i.e. $z \to z+10^{-4}$, holding the benchmark curve fixed.

**Outputs (What You Produce)**
- Z-spread: $z \approx 136.6$ bp (continuously compounded).
- Spread duration: $D_s \approx 2.856$ years.
- CS01 (book convention): $CS01 = P(z)-P(z+1\text{ bp}) \approx 0.0280$ price points per 100 per 1bp (positive for long credit).

**Step-by-step**
1. **Translate the price quote to a PV equation (dirty price target):**
   $$P_{\text{dirty}}=\sum_{k} CF_k\cdot P(0,t_k)e^{-zt_k}$$
2. **Plug in cashflows and benchmark discount factors:**
   $$PV(z)=5\cdot 0.97\cdot e^{-z}+5\cdot 0.93\cdot e^{-2z}+105\cdot 0.88\cdot e^{-3z}$$
3. **Solve for $z$ by bracketing + bisection (repricing target = 98.00):**
   - $z=1.00\%$: $PV\approx 99.03$ (too high)
   - $z=2.00\%$: $PV\approx 96.24$ (too low)
   - refine $\Rightarrow z\approx 1.366\%$ with $PV\approx 98.00$
4. **Repricing check:** confirm $PV(z)\approx 98.00$ to tolerance.
5. **Compute spread duration from PV weights:**
   $$D_s=\frac{\sum_k t_k\cdot PV_k(z)}{\sum_k PV_k(z)}\approx 2.856$$
6. **Compute CS01 (widening bump):**
   $$CS01=P(z)-P(z+1\text{ bp})\approx P\cdot D_s\cdot 10^{-4}\approx 98.00\times 2.856\times 10^{-4}\approx 0.0280$$

**Cashflows**

| Date | Cashflow | Explanation |
|---|---:|---|
| 2027-02-16 | 5 | coupon |
| 2028-02-16 | 5 | coupon |
| 2029-02-16 | 105 | coupon + principal |

**P&L / Risk Interpretation**
- If you are long the bond, a $+1$ bp widening in $z$ (holding the benchmark fixed) reduces price by about **0.028 points per 100**.
- Scaling: 1 price point = 1% of par. On $10{,}000{,}000$ notional, 1 point $\approx \$100{,}000$, so $0.028$ points $\approx \$2{,}800$ per bp (loss on widening; gain on tightening).

**Sanity Checks**
- **Sign check:** $z$ up $\Rightarrow$ heavier discounting $\Rightarrow P$ down, so $P(z+1\text{ bp})<P(z)$ and CS01 (as defined here) is positive.
- **Units check:** $z$ is per-year; $1\text{ bp}=10^{-4}$ in per-year units; CS01 is in price points per 100 per bp.
- **Reproduction check:** a spreadsheet with the PV formula and a bisection solve should reproduce $z\approx 136.6$ bp.

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

**Difference:** ~40 bp. In this example, the term-structure-consistent Z-spread is noticeably lower than the single-yield G-spread.

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
- Notional = \$10,000,000

**Mark-to-market:**

$$MTM = (185 - 160) \times 0.0001 \times 3.50 \times \$10,000,000 = 25 \times 0.0001 \times 3.50 \times \$10mm$$
$$MTM = \boxed{\$87,500}$$

**Interpretation:** Spreads tightened by 25 bp; the asset swap buyer (long credit) has a gain of \$87,500.

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
5. **I-spread:** Bond YTM minus swap curve yield—must specify the swap curve family and curve construction. I-spread can exceed G-spread when the swap spread is negative (Section 8.4.1).
6. **Z-spread:** Solves a PV match using the full benchmark discount curve; term-structure consistent but still benchmark-dependent. Z-spread and G-spread can differ materially on steep curves (Example D).
7. **TED spread:** A spread solved so that discounting cash flows at futures-implied rates minus the spread matches the dirty price (with a futures-vs-forwards caveat)
8. **OAS:** Solves a PV match using an interest-rate model/tree; isolates spread after removing option value; inherently model-dependent
9. **Asset swap spread:** Converts fixed bond to floating economics; par vs market conventions differ. MTM depends on spread changes × remaining PV01.
10. **Callable bonds:** Z-spread is not option-adjusted; OAS is the option-adjusted (model-based) measure
11. **Market spreads embed multiple components:** actuarial expected loss, default risk premium, volatility risk premium, liquidity premium—not just probability of default
12. **CDS-bond basis:** CDS spread minus bond Libor spread; can be positive or negative; driven by fundamental and market factors

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **G-spread** | $y_{\text{bond}} - y_{\text{gov}}(T)$ | Quick relative value vs governments; benchmark-dependent |
| **I-spread** | $y_{\text{bond}} - y_{\text{swap}}(T)$ | Relative value vs swaps; curve definition matters |
| **Z-spread** | Constant spread to benchmark discounting matching dirty price | Term-structure consistent; for non-callable bonds |
| **TED spread** | Spread such that discounting at futures-implied rates minus spread matches dirty price | Useful for valuing bonds relative to a futures-implied benchmark (with a futures-vs-forwards caveat) |
| **OAS** | Spread to model rates so model price = market price | For callable/option bonds; model-dependent |
| **Par asset swap spread** | $(P_{\text{Libor}} - P)/PV01$ | Spread-to-swaps; standard quote convention |
| **Market asset swap spread** | $A(0)/P$ | Reduces counterparty exposure vs par structure |
| **Asset swap MTM** | $\text{MTM}_{\$}(t) \\approx (A(0)-A(t))_{\\text{bp}} \\times 10^{-4} \\times PV01_{\\text{ann}}(t,T) \\times N$ | P&L from spread changes; unit checks prevent $10^4$ mistakes |
| **CS01** | $CS01 := P(s)-P(s+1\\text{bp})$ (widening bump; hold benchmark fixed) | Spread sensitivity; report units and sign explicitly |
| **Spread duration** | $-\frac{1}{P}\frac{\partial P}{\partial s}$ | PV-weighted average time under spreaded discounting |
| **Coverage ratio** | Credit spread / Actuarial spread | How much spreads exceed pure default compensation |
| **CDS-bond basis** | CDS spread − Bond Libor spread | Fundamental RV measure for credit trading |

---

## Notation

| Symbol | Meaning | Units / Convention |
|--------|---------|-------------------|
| $P_{\text{clean}}$ | clean (quoted) price | price points per 100 notional |
| $P_{\text{dirty}}$ | dirty (invoice) price | price points per 100; $P_{\text{dirty}}=P_{\text{clean}}+AI$ |
| $AI$ | accrued interest | price points per 100 |
| $y$ | yield to maturity | per year; always state compounding/day count when comparing |
| $y_{\text{gov}}(T)$ | government benchmark yield | same yield convention as $y$ when used in $s_G$ |
| $y_{\text{swap}}(T)$ | swap benchmark yield | must specify curve (OIS/RFR vs IBOR) + interpolation |
| $s_G$ | G-spread | bp of yield (on the $y$ convention) |
| $s_I$ | I-spread | bp of yield (on the $y$ convention) |
| $P_{\text{bench}}(0,t)$ | benchmark discount factor | unitless |
| $z$ (or $s_Z$) | Z-spread parameter | per year; this chapter uses $DF^{(z)}(0,t)=P_{\text{bench}}(0,t)e^{-zt}$ unless stated |
| $s_{\text{OAS}}$ | option-adjusted spread | per year; defined inside a model/tree/Monte Carlo |
| $D_s$ | spread duration | years; used as $\Delta P/P \\approx -D_s\\,\\Delta s$ |
| $CS01$ | credit spread 01 | widening bump: $P(s)-P(s+1\\text{bp})$; positive for long credit under this chapter’s convention |
| $PV01(t,T)$ | annuity (asset swap) | years: $\sum_m Z(t,t_m)\\Delta_m$; dollars per bp is $10^{-4}N\\cdot PV01$ |
| $A(0)$ | par asset swap spread | bp per year |
| $A^{*}(0)$ | market asset swap spread | bp per year; $A^{*}(0)=A(0)/P$ when $P$ is per-par price |
| $MTM(t)$ | asset swap mark-to-market | currency; $\text{MTM}_{\$}(t)\\approx (A(0)-A(t))_{\\text{bp}}\\times 10^{-4}\\times PV01(t,T)\\times N$ |
| $DVOAS$ | OAS 01 (OAS sensitivity) | currency per bp (for the stated notional); often estimated by a central difference in OAS |

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
| 11 | Why use OAS (not Z-spread) for callable bonds? | Cash flows depend on rates/option exercise; OAS is defined inside a model/tree/Monte Carlo and adjusts for the option |
| 12 | What is spread duration? | $D_s = -\frac{1}{P}\frac{\partial P}{\partial s}$ |
| 13 | What is CS01 (in this chapter)? | Widening-bump spread risk: $CS01:=P(s)-P(s+1\\text{bp})$ holding the benchmark curve/model fixed |
| 14 | Why isn't CS01 universal? | Different spread definitions (G, I, Z, OAS) have different sensitivities |
| 15 | Asset swap spread conceptually? | Spread such that discounting at swap + spread gives bond price |
| 16 | Par asset swap formula? | $A = (P_{\text{Libor}} - P) / PV01$ |
| 17 | What does PV01 represent (here)? | Annuity (in years): $PV01=\\sum Z\\Delta$. Dollars per bp $\\approx 10^{-4}N\\cdot PV01$ |
| 18 | Key asset swap ambiguity? | Par vs market structure (floating notional differs) |
| 19 | If spread widens, what happens to price? | Price decreases (all else equal) |
| 20 | Repricing test for Z-spread? | Plug $s_Z$ back in; verify PV = $P_{\text{dirty}}$ |
| 21 | Why is OAS model-dependent? | Option value depends on interest rate dynamics/volatility used |
| 22 | List three components of credit spread (O'Kane) | Actuarial spread, default risk premium, liquidity risk premium |
| 23 | What is actuarial spread? | Compensation for expected loss from historical default/recovery |
| 24 | What is coverage ratio? | Credit spread / actuarial spread |
| 25 | What question to ask when someone says "spread widened"? | "Which spread definition and which benchmark curve?" |
| 26 | What is a TED spread (Tuckman)? | Spread such that discounting cash flows at futures-implied rates minus spread matches dirty price |
| 27 | Key caveat for TED spreads? | Futures rates differ from forward rates (convexity/futures-forward adjustment), so the benchmark is not a pure no-arbitrage discount curve |
| 28 | Market asset swap spread formula? | $A^*(0) = A(0)/P$ |
| 29 | Asset swap MTM formula (unit-safe)? | $\text{MTM}_{\$}(t)\\approx (A(0)-A(t))_{\\text{bp}}\\times 10^{-4}\\times PV01(t,T)\\times N$ |
| 30 | What is CDS-bond basis? | CDS spread minus bond Libor spread (ASW) |
| 31 | When can yield spreads be misleading? | On steep curves and/or high-coupon bonds; term-structure-consistent measures (e.g., Z-spread) can differ materially |
| 32 | When do you need OAS rather than Z-spread? | When the bond has embedded options; OAS uses a model/tree/Monte Carlo to separate option value from spread |
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

**14.** You entered an asset swap at 175 bp. The current market spread is 150 bp and remaining PV01 = 4.2. On \$5mm notional, what is your MTM?

**15.** Explain why CDS-bond basis can be either positive or negative. Give one factor that pushes it each direction.

**16.** A credit analyst observes: G-spread = 280 bp, Z-spread = 230 bp (50 bp difference). What does this suggest about the curve shape and the bond's coupon?

**17.** Using Example C, suppose the bond has $P=98.00$ (per 100) and spread duration $D_s=2.856$. Estimate CS01 (price points per 100 per 1bp widening). Then scale it to a $10{,}000{,}000$ notional.

### Solution Sketches (Selected)

**1.** $P_{\text{dirty}} = 101.20 + 0.80 = 102.00$

**2.** Coupon per period $= 0.06/2 \times 100 = 3.00$. $AI = 0.40 \times 3.00 = 1.20$

**3.** Solve $99 = 5/(1+y) + 105/(1+y)^2$. At $y = 6\%$: PV = 98.18 (low). At $y = 5\%$: PV = 100 (high). So $y \approx 5.5\%$.

**4.** $s_G = 7.10\% - 4.85\% = 2.25\% = 225$ bp

**5.** Bracket with $s = 1\%$ (PV ≈ 99), $s = 2\%$ (PV ≈ 96). Bisect to $s_Z \approx 1.37\% = 137$ bp. (Exact 136.6 bp).

**6.** Yield spread compares one yield to one yield, ignoring that different cash flows should be discounted at different maturities. Z-spread revalues each cash flow using the full curve. On a steep curve, spot and par curves differ, and coupon weighting matters—so the two measures can diverge (Example D shows one concrete case).

**7.** $\Delta P \approx -P \times D_s \times \Delta s = -100 \times 4.5 \times 0.0010 = -0.45$

**8.** Fitted Treasury zeros are more appropriate because: (a) Z-spread requires discount factors at each cash flow date, and fitted zeros provide these directly; (b) on-the-run yields can be distorted by repo specialness and liquidity effects that don't reflect the true risk-free rate.

**9.** Three clarifications needed: (a) Which swap curve (OIS/RFR vs legacy IBOR, and which curve construction)? (b) What interpolation method for off-market maturities? (c) Clean or dirty price basis for the bond yield?

**10.** If matching clean price instead of dirty (and accrued interest is positive), the spread will be biased **higher**. The engine is trying to fit a *lower* target price, which requires more discounting (larger spread). (If the bond is in an ex-dividend convention where accrued can be negative, the sign can flip—always check the convention.)

**11.** For option-embedded bonds, the more meaningful measure is **OAS**, because cash flows depend on rate paths and option exercise. A naive “Z-spread” computed by discounting contractual (non-call) cash flows is not option-adjusted and will embed the value of the issuer’s call option. Intuitively, since a callable bond is an otherwise identical noncallable bond minus the value of the embedded option, fitting the same contractual cash flows to a lower price typically requires a higher static spread. The key is not the inequality but the definition: always state whether the spread is static (fixed cash flows) or option-adjusted (model-based).

**12.** Different spreads can arise from: liquidity differences, repo specialness, different coupons (yield spread distortion), supply/demand imbalances, maturity differences, settlement timing, or delivery option value in CDS basis trades.

**13.** Market ASW spread = Par ASW / P = 200 / 0.95 = 210.5 bp

**14.** MTM = (175 - 150) × 0.0001 × 4.2 × \$5,000,000 = 25 × 0.0001 × 4.2 × \$5mm = \$52,500 gain

**17.** $CS01 \approx P\cdot D_s\cdot 10^{-4} = 98.00\times 2.856\times 10^{-4} \approx 0.0280$ points per 100 per bp. On $10mm$ notional: $0.0280$ points $\approx 0.0280\%$ of par $\approx \$2{,}800$ per bp.

---

## References

- (Dominic O’Kane, *Modeling Single-name and Multi-name Credit Derivatives*, “4.2.9 The Zero Volatility Spread”; “4.4.3 Valuation of an asset swap”)
- (Bruce Tuckman, *Fixed Income Securities*, “\(P+AI = PV\) (future cash flows)”; “TED Spreads”; “Asset Swap Spreads and Asset Swaps”)
- (John C. Hull, *Options, Futures, and Other Derivatives*, “Credit Default Swaps and Bond Yields”)
- (John C. Hull, *Risk Management and Financial Institutions*, “CDS–Bond Basis”)
- (Dessislava Pachamanova and Frank J. Fabozzi, *Simulation and Optimization in Finance*, “Spread Risk”)
- (Edwin J. Elton, Martin J. Gruber, Stephen J. Brown, and William N. Goetzmann, *Modern Portfolio Theory and Investment Analysis*, “Special Considerations in Bond Pricing”)
- (Jody Gunzberg Bennett, *Trading Volatility, Correlation, Term Structure and Skew*, “Credit spread is only partly due to default risk”)
