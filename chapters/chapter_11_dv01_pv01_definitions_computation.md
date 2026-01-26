# Chapter 11: DV01/PV01 — Definitions, Computation, and "What's Being Bumped"

---

## Introduction

A portfolio manager reports that her bond position has a "DV01 of $50,000." A swaps trader says his book is "flat on PV01." A risk officer asks for "key-rate exposures by bucket." Each statement sounds precise—a dollar measure of interest rate risk per basis point. But hidden in these simple phrases lurks a critical question that, if misunderstood, can render all these numbers meaningless: *what exactly is being bumped?*

Consider the portfolio manager's $50,000 DV01. Does that number assume the bond's yield moves by 1 basis point? Or does it assume every zero rate on the discount curve shifts by 1bp? Perhaps it reflects bumping the par swap quotes and rebuilding the curve? These are different economic scenarios, and they can produce different dollar sensitivities for the same position. As Tuckman emphasizes in *Fixed Income Securities*, DV01 is defined as "the change in value of a fixed income security for a one-basis point decline in rates"—but this definition is meaningful only after you specify *which* interest rate measure moves and *how*.

The consequences of ambiguity are real. A trader who hedges using yield-based DV01 but marks-to-market against a curve-based pricing engine will experience unexplained P&L when the curve twists. A risk system that aggregates DV01s computed under inconsistent bump definitions will produce portfolio sensitivities that don't add up. And a hedger who matches total DV01 without understanding key-rate exposures may find the hedge worthless when the curve steepens instead of shifting in parallel.

This chapter provides a rigorous foundation for interest rate sensitivity measures. We define DV01 formally in **Section 11.1**, distinguishing strictly between **yield-based DV01** (Section 11.2) and **curve-based DV01** (Section 11.3). We then explore the concept of **PVBP** and swap annuity in **Section 11.4**, showing how it differs from curve risk. **Section 11.5** provides the computational mechanics of bump-and-reprice. Finally, **Section 11.6** presents a taxonomy of bump specifications—from parallel shifts to key-rate perturbations—ensuring you can answer the critical question: "What is being bumped?"

---

## Conventions and Notation

| Symbol | Definition |
|--------|------------|
| $P$ | Bond full (dirty) price per 100 face (unless stated otherwise) |
| $P_{\text{flat}}$ | Bond flat/clean price; $P_{\text{flat}} = P_{\text{full}} - \text{AI}$ |
| $\text{AI}$ | Accrued interest |
| $y$ | Yield to maturity (IRR) |
| $\Delta y$ | Yield change in decimal units (e.g., 1bp $= 0.0001$) |
| $\text{DV01}$ | Dollar Value of 01; PV change for 1bp move in the specified rate measure |
| $P(0,T)$ | Discount factor / zero-coupon bond price at time 0 for maturity $T$ |
| $z(T)$ | Spot/zero rate for maturity $T$ (compounding basis must be stated) |
| $\tau_i$ | Accrual year fraction for period $i$ |
| $K$ | Swap fixed rate |
| $A$ | Swap annuity factor: $A = \sum_i \tau_i P(0,T_i)$ |
| $S$ | Par/forward swap rate: $S = \frac{P(0,T_0) - P(0,T_m)}{A}$ |
| 1 bp | $10^{-4}$ in decimal rate units |

**Default conventions for examples:**
- Settlement at $t=0$ on a coupon date (AI = 0, so clean = dirty)
- Semiannual compounding unless stated otherwise
- Bond prices per 100 face; swap PVs in dollars for stated notional
- **Sign convention:** Tuckman convention (PV change for 1bp **fall** in rates, so DV01 positive for long bonds)

> **Sign Convention Warning**
>
> Tuckman defines DV01 with a minus sign so that DV01 is **positive** for securities that gain value when rates decline (e.g., long bonds). Hull's glossary, by contrast, describes DV01 as "the price change from a 1-basis-point increase in all rates"—the opposite sign convention. Always verify before combining risk numbers from different systems: "Is DV01 defined for +1bp up (a loss), or −1bp down (a gain)?"

---

## 11.1 DV01: The Dollar Value of a Basis Point

### 11.1.1 The Formal Definition

Tuckman introduces DV01 by considering the **price-rate function** $P(y)$, where $y$ is some interest rate factor. He then defines:

> "Letting $\Delta P$ and $\Delta y$ denote the changes in price and rate and noting that the change measured in basis points is $10,000 \times \Delta y$, define the following measure of price sensitivity: DV01 is an acronym for dollar value of an '01 (i.e., .01%) and gives the change in the value of a fixed income security for a one-basis point decline in rates." (Tuckman, Chapter 5)

The formal expression, using this convention, is:

$$\boxed{\text{DV01} \equiv -\frac{\Delta P}{10,000 \times \Delta y}}$$

The factor of 10,000 converts the rate change from decimal units (where 1bp = 0.0001) to a "per basis point" measure. The negative sign ensures that DV01 is positive when prices increase as rates decline—which is the typical case for fixed-coupon bonds. Tuckman explains: "The negative sign defines DV01 to be positive if price increases when rates decline and negative if price decreases when rates decline. This convention has been adopted so that DV01 is positive most of the time."

> **Analogy: The Speedometer**
>
> Think of DV01 as the speedometer on your P&L dashboard.
> *   **Yield Change**: The car's speed (how fast rates are moving).
> *   **DV01**: The gear ratio.
> *   **Equation**: Speed (bp) × Gear (DV01) = Distance (Dollars P&L).
>
> If your DV01 is $50,000 and the market moves 10 bps, you just traveled $500,000 in P&L space.

When an explicit price-rate function $P(y)$ is available and differentiable, DV01 can be expressed using the derivative:

When an explicit price-rate function $P(y)$ is available and differentiable, DV01 can be expressed using the derivative:

$$\boxed{\text{DV01} = -\frac{1}{10,000}\frac{dP}{dy}}$$

This derivative form is exact in the limit of infinitesimal rate changes. Graphically, $dP/dy$ represents the slope of the tangent line to the price-rate curve at the current rate level.

### 11.1.2 What DV01 Answers

DV01 answers a simple question: *"If rates move by 1bp (in the way I've defined), how many dollars do I gain or lose?"*

This is a **first-order Taylor approximation** of the price change. For a small rate move $\Delta y$, the price change is approximately:

$$\Delta P \approx -10,000 \times \text{DV01} \times \Delta y$$

> **The Einstein Formula of Fixed Income**
>
> We can rewrite risk in percentage terms to get the most important equation in bond math:
> 
> $$\frac{dP}{P} \approx -D \times dy$$
>
> *   **Change in Price (%)** equals **Duration** times **Change in Yield**.
> *   DV01 is just the *dollar version* of this: $d\text{Dollar} = -\text{DV01} \times dy$.

> **Visualization: The Price-Yield Seesaw**
>
> Imagine a seesaw.
> *   **Fulcrum**: The Yield.
> *   **Seat**: The Price.
> *   **Weight on Seat**: The DV01.
> *   **Action**: If you push the Fulcrum (Yield) *up*, the Seat (Price) goes *down*.
> *   **Magnitude**: A heavier weight (High DV01/Duration) means the seat crashes down harder for the same push.

Hull (in *Risk Management and Financial Institutions*) provides a complementary perspective: "Duration measures the sensitivity of the value of a portfolio to a small parallel shift in the zero-coupon yield curve." The relationship between DV01 and duration will be explored in Chapter 12, but the essential point is that DV01 gives dollar risk while duration gives percentage risk.

Crucially, the DV01 definition requires specifying the "rate measure" $y$ being bumped. While DV01 can be computed "for any interest rate measure," market usage typically implies **yield-based DV01** for individual bonds and **curve-based DV01** for portfolios and derivatives.

### 11.1.3 Portfolio Aggregation and Additivity

Tuckman proves that for a portfolio of positions, DV01 is additive: "the DV01 of a portfolio is the sum of the individual DV01 values." This follows from the linearity of the derivative operator. If portfolio value is $V_{\text{port}} = \sum V_i$, then:

$$\text{DV01}_{\text{portfolio}} = \sum_i \text{DV01}_i$$

**But there is a critical caveat**: this additivity holds only if the same bump definition applies to all positions. If different positions use different bump definitions (e.g., one bumps yield, another bumps the zero curve), aggregation creates an "apples to oranges" sum that lacks economic meaning.

> **Example: Portfolio Additivity**
>
> Consider a portfolio with three positions valued on a flat 4% curve. We define "DV01" as the change in value for a parallel 1bp fall in the curve.
>
> | Position | Face/Notional | DV01 per 100 | Position DV01 |
> |----------|---------------|--------------|---------------|
> | **Bond 1** (long 5y 5% coupon) | +$1,000,000 | 0.0458 | +$458 |
> | **Bond 2** (short 2y 2% coupon) | −$1,000,000 | 0.0192 | −$192 |
> | **Swap** (receive fixed 3y) | $1,000,000 | — | +$280 |
>
> **Sum of individual DV01s:** $458 - 192 + 280 = \$546$.
>
> **Verification:** If we reprice the entire portfolio under a −1bp curve shift (3.99%), the total portfolio value increases by approximately $546. This confirms that DV01 is additive under a consistent bump definition.

### 11.1.4 The $100 vs $100mm Mistake

Why is DV01 the standard normalization? Why not just use duration?

*   **Retail Trader ($10,000 trade)**: 5-year bond, Duration = 4.5.
    *   10bp move $\approx$ 0.45% price change.
    *   P&L = $45. No big deal.
*   **Institutional Desk ($100,000,000 trade)**: Same bond, Duration = 4.5.
    *   10bp move $\approx$ 0.45% price change.
    *   P&L = **$450,000**. Big deal.

**Lesson**: Percentage change (Duration) hides the scale. DV01 converts everything into "Dollars per bp", which is the only thing that matters for the P&L statement.

> **Superpower: Risk is Additive**
> You cannot add yields (5% + 4% = ??).
> You cannot add prices ($100 + $98 = ??).
> But you **can** add DV01s.
> *   $50k risk + $20k risk = $70k risk.
> *   This forces a "Common Currency" of risk across the entire trading floor.

---

## 11.2 Yield-Based DV01: Bumping a Single Scalar

### 11.2.1 The Yield-Based Framework

When bond traders speak of "DV01," they typically mean **yield-based DV01**. This measures the sensitivity to the bond's own yield-to-maturity (YTM). Tuckman describes this as "a special case of DV01...the yield of a security is the interest rate factor."

Recall from Chapter 6 that YTM is the single discount rate that equates cashflows to the price. For a bond with coupon $c$ per 100 face, maturity $T$ years, and semiannual compounding:

$$P(y) = \sum_{t=1}^{2T} \frac{c/2}{(1+y/2)^t} + \frac{100}{(1+y/2)^{2T}}$$

Yield-based DV01 is computed by bumping this single scalar $y$. Using central differences:

$$\text{DV01}_{\text{yield}} = -\frac{P(y - 1\text{bp}) - P(y + 1\text{bp})}{2 \times 10,000 \times (1\text{bp})} = \frac{P(y - 1\text{bp}) - P(y + 1\text{bp})}{2}$$

### 11.2.2 Explicit Formula for Yield-Based DV01

Tuckman derives an explicit formula by differentiating the bond price equation. The result is:

$$\boxed{\text{DV01} = \frac{1}{10,000} \times \frac{1}{1+y/2}\left[\sum_{t=1}^{2T} \frac{t}{2} \cdot \frac{c/2}{(1+y/2)^t} + T \cdot \frac{100}{(1+y/2)^{2T}}\right]}$$

This formula has a clear interpretation: DV01 equals "the sum of the time-weighted present values of a bond's cash flows divided by 10,000 multiplied by one plus half the yield." The time-weighting reflects that longer-dated cashflows contribute more to interest rate sensitivity.

### 11.2.3 Worked Example: Computing DV01 from First Principles

Tuckman illustrates the computation with a detailed example (Table 6.1 in his text). Let us replicate this methodology.

**Bond:** 5-year U.S. Treasury with 5.625% annual coupon, yield 5.0441%, settling on a coupon date.

| Date | Term (yrs) | Cash Flow | PV Factor | Present Value | Time × PV |
|------|-----------|-----------|-----------|---------------|-----------|
| 0.5 | 0.5 | 2.8125 | 0.9755 | 2.7433 | 1.3717 |
| 1.0 | 1.0 | 2.8125 | 0.9517 | 2.6758 | 2.6758 |
| 1.5 | 1.5 | 2.8125 | 0.9283 | 2.6100 | 3.9150 |
| 2.0 | 2.0 | 2.8125 | 0.9055 | 2.5458 | 5.0916 |
| 2.5 | 2.5 | 2.8125 | 0.8832 | 2.4832 | 6.2079 |
| 3.0 | 3.0 | 2.8125 | 0.8614 | 2.4221 | 7.2662 |
| 3.5 | 3.5 | 2.8125 | 0.8402 | 2.3625 | 8.2687 |
| 4.0 | 4.0 | 2.8125 | 0.8194 | 2.3044 | 9.2175 |
| 4.5 | 4.5 | 2.8125 | 0.7990 | 2.2477 | 10.1146 |
| 5.0 | 5.0 | 102.8125 | 0.7792 | 80.1444 | 400.7219 |
| **Sums** | | | | **102.5391** | **454.8511** |

The DV01 calculation follows Tuckman's formula:

$$\text{DV01} = \frac{1}{10,000} \times \frac{1}{1 + 0.050441/2} \times 454.8511 = \mathbf{0.044366}$$

**Interpretation:** A one-basis point decline in the bond's yield increases its price by about 4.4 cents per $100 face value.

### 11.2.4 The Limitation: Parallel Yield Shifts

Tuckman explicitly warns about the key limitation:

> "Given the assumptions in computing DV01, however, this hedge will work as intended only if the yield of the bond bought changes by the same amount as the yield of the [hedge]. It is in this sense that DV01 (along with the other measures of price sensitivity in this chapter) requires parallel yield shifts."

If you hedge a position using yield-based DV01, but the yields of the two bonds move by different amounts, the hedge will fail. This is the fundamental weakness of one-factor approaches and motivates the multi-factor methods in Chapter 14.

---

## 11.3 Curve-Based DV01: Bumping a Curve Object

### 11.3.1 From Yield to Curve

An alternative approach is **curve-based DV01**. Here, we price the bond using a discount factor curve $P(0,t)$ derived from a term structure model, rather than the bond's own yield:

$$V = \sum_i \text{CF}_i \times P(0, t_i)$$

Curve-based DV01 specifies a perturbation to the curve—for example, a parallel shift to all zero rates: $z(t) \mapsto z(t) + 1\text{bp}$ for all $t$.

> **Example: Bond DV01 via Zero-Curve Bump**
>
> Take the same 5-year 5.625% bond. Suppose we price it off a zero curve that happens to reproduce the exact same base price of 102.5391.
>
> Apply a **parallel +1bp bump** to every zero rate on the curve:
> - $P(z+1\text{bp}) = 102.4934$ (price falls when rates rise)
> - $P(z-1\text{bp}) = 102.5849$ (price rises when rates fall)
>
> $$\text{DV01}_{\text{curve}} = \frac{102.5849 - 102.4934}{2} = 0.04575$$

### 11.3.2 Why Yield DV01 and Curve DV01 Differ

In the examples above, yield DV01 was **0.04437** while curve DV01 was **0.04575**. The difference of approximately 0.001 seems small, but it reveals a fundamental truth: **Yield DV01 $\neq$ Curve DV01 in general.**

The reason is subtle but important. Yield is a weighted average of the spot rates implied by the bond's price. A 1bp change in yield implies a specific, non-parallel pivoting of the implicit spot curve that prices this particular bond. Conversely, a parallel shift of the spot curve changes the bond's yield by an amount that is not exactly 1bp—it depends on the cash flow structure.

Tuckman warns that yield-based DV01 "can mis-measure risk when the curve does not move in the assumed way." If you hedge a curve-based P&L using yield-based metrics, you will see residual risk (often called "model basis" or "curve risk") when the curve twists or steepens.

### 11.3.3 When to Use Which

| Scenario | Preferred Measure | Rationale |
|----------|------------------|-----------|
| Single bond trading | Yield DV01 | Market quotes bonds by yield; traders think in yield terms |
| Portfolio risk aggregation | Curve DV01 | Consistent bump across all instruments |
| Swap/derivatives trading | Curve DV01 | Swaps don't have a single "yield" |
| Hedging across instruments | Curve DV01 | Ensures hedge and position use same curve |

---

## 11.4 PVBP, Swap Annuity, and Naming Collisions

### 11.4.1 The Swap Annuity Factor

For swaps, the concept of "duration" is handled via the **Annuity Factor** (or "PV01 of the fixed leg"). Andersen and Piterbarg define the annuity $A$ as the present value of 1bp paid on the fixed leg schedule:

$$\boxed{A(t) = \sum_{n} \tau_n P(t, T_{n+1})}$$

where $\tau_n$ is the year fraction for period $n$ and $P(t, T_{n+1})$ is the discount factor to payment date $T_{n+1}$.

The par swap rate $S$ is simply the ratio that makes the swap have zero value at inception:

$$S = \frac{P(0,T_0) - P(0,T_m)}{A}$$

where $T_0$ is the start date and $T_m$ is the maturity. For a spot-starting swap, $P(0,T_0) = 1$.

### 11.4.2 PVBP: Sensitivity to the Fixed Rate

In swap trading, **PVBP** (Present Value of a Basis Point) often refers to the change in PV resulting from bumping the **contractual fixed rate** $K$ by 1bp, while holding the curve fixed.

For a receiver swap with notional $N$:

$$V_{\text{swap}} = N(K - S) A$$

The sensitivity to the fixed rate $K$ is:

$$\frac{\partial V}{\partial K} = N \times A$$

Thus, **PVBP = Notional × Annuity × 0.0001** (the 0.0001 converts to basis points).

> **Example: Swap PVBP**
>
> Consider a $100,000,000 notional 5-year swap receiving fixed 4.50% versus floating. The discount curve is flat at 4%.
>
> 1. **Annuity Factor:** $A = \sum_{t=1}^5 (1.04)^{-t} = 4.4518$ (annual payments for simplicity).
> 2. **PVBP Definition:** Change in PV if we bump the *coupon* $K$ by 1bp.
>    $$\text{PVBP} = \$100,000,000 \times 4.4518 \times 0.0001 = \$44,518$$
>
> This means a 1bp increase in the fixed rate you receive adds $44,518 to the swap's value.
>
> **Note:** If we calculated a **Curve PV01** by bumping the discount rates, we would get a different number, because changing discount rates affects both the annuity and the implicit floating leg value. PVBP (fixed rate sensitivity) and Curve PV01 are conceptually distinct.

### 11.4.3 The "PV01" Naming Collision

On trading desks, "PV01" is an overloaded term that can mean different things depending on context:

| Term | Meaning | Context |
|------|---------|---------|
| **Curve PV01** | Bumping the discount curve by 1bp | Risk management, hedging |
| **PVBP / Rate PV01** | Bumping the fixed rate by 1bp | New deal pricing, annuity calculation |
| **Risky PV01** | Bumping credit spreads by 1bp | CDS trading (Chapter 41) |
| **Par-Point PV01** | Bumping a single input quote, rebuilding curve | Curve risk (Chapter 22) |

Always clarify: "Is this PV01 from bumping the curve, or the coupon?" Quantitative developers must strictly separate these in code to avoid confusion.

### 11.4.4 Multi-Curve PV01

In modern markets, we use different curves for discounting (OIS) and projecting floating rates (term SOFR or legacy IBOR). This splits "PV01" into two components.

> **Example: Multi-Curve Split**
>
> Consider a 3-year swap, receiving fixed 4.00%.
> - **Discount Curve:** OIS flat at 3.50%.
> - **Projection Curve:** Term SOFR flat at 3.75%.
>
> For $10,000,000 notional:
>
> **OIS Discount PV01** (bump OIS curve only by 1bp): **$8.50**.
> The swap is near par (PV $\approx$ 0). Changing the discount rate has a small effect because it affects the PV of both legs similarly.
>
> **Projection Curve PV01** (bump projection curve only by 1bp): **$295.00**.
> Bumping the projection curve increases the expected floating payments directly. This is where the primary risk resides.
>
> **Total PV01:** $303.50.
>
> A single-curve model would miss this distinction, leading to improper hedging if OIS and projection curves move independently (basis risk).

---

## 11.5 Bump-and-Reprice Mechanics

### 11.5.1 The Central Finite Difference Algorithm

The gold standard for computing DV01 numerically is **central finite differences**:

1. **Base:** Price at current rates $V_0$.
2. **Up:** Price at rates $+1\text{bp}$ → $V_\uparrow$.
3. **Down:** Price at rates $-1\text{bp}$ → $V_\downarrow$.
4. **Calc:** $\text{DV01} = (V_\downarrow - V_\uparrow) / 2$.

Using a central difference cancels out the second-order (convexity) error term, providing a much more accurate estimate of the tangent slope than a one-sided bump.

Tuckman explains the rationale: "The most stable numerical estimate chooses rates that are equally spaced above and below" the rate level in question. This avoids the bias introduced by the curvature of the price-rate relationship.

### 11.5.2 Why Central Differences Outperform One-Sided Bumps

Consider a bond with price $P(y)$ that exhibits positive convexity. For a one-sided up bump:

$$\text{DV01}_{\text{one-sided}} = V_0 - V_\uparrow$$

This underestimates the true sensitivity because convexity causes the price to fall less for a rate increase than it rises for a rate decrease.

The central difference:

$$\text{DV01}_{\text{central}} = \frac{V_\downarrow - V_\uparrow}{2}$$

averages the upward and downward sensitivities, effectively canceling the convexity bias.

> **Example: Central vs One-Sided**
>
> For a 10-year 5% bond at 4% yield (Price = 108.18):
> - $P(4.01\%) = 108.09$ (down 0.0893)
> - $P(3.99\%) = 108.27$ (up 0.0907)
>
> **One-sided (up):** DV01 = 0.0893
> **One-sided (down):** DV01 = 0.0907
> **Central:** DV01 = (0.0907 + 0.0893) / 2 = **0.0900**
>
> The central estimate is the average and is closest to the true derivative. The asymmetry (0.0907 vs 0.0893) reflects the bond's positive convexity.

### 11.5.3 Explicit Derivative Formula

For simple bonds, we can derive the analytical derivative directly. Starting from:

$$P(y) = \sum_i \text{CF}_i (1+y/2)^{-t_i}$$

Differentiating:

$$\frac{dP}{dy} = -\sum_i \frac{t_i}{2} \cdot \frac{\text{CF}_i}{(1+y/2)^{t_i+1}}$$

Thus:

$$\boxed{\text{DV01} = \frac{1}{10,000} \sum_i \frac{t_i/2 \cdot \text{CF}_i}{(1+y/2)^{t_i+1}}}$$

This formula confirms the intuition that longer-maturity cashflows (large $t_i$) contribute more to DV01, weighted by both the cashflow size and the discount factor.

### 11.5.4 Nonlinearity: Why Exact Bumps Matter

Does it matter if we bump the rate $r$ or the discount factor $DF$ directly?

Yes. The relationship $DF = 1/(1+r)^T$ is nonlinear.

> **Example: Rate Bump vs DF Bump**
>
> Consider a single cashflow of 100 at $T=10$ years, $r=5\%$ (annual compounding).
> - Base $DF = 0.6139$, $V_0 = 61.39$.
>
> **Method 1: Exact Rate Bump**
> Bump $r$ to $5.01\%$: $DF_\uparrow = 1/(1.0501)^{10} = 0.6133$.
> $V_\uparrow = 61.33$, DV01 = 61.39 - 61.33 = **0.0584**.
>
> **Method 2: Linear DF Approximation**
> Use $\Delta DF \approx -T \cdot DF \cdot \Delta r / (1+r)$.
> This gives DV01 ≈ **0.0585**.
>
> The difference is small for 1bp, but for larger shocks (VaR, stress tests), linear approximations break down significantly. Production systems should always use exact re-pricing.

---

## 11.6 "What's Being Bumped?"—A Taxonomy

The critical insight of this chapter is that **DV01 is a property of the bump, not just the instrument**. The same bond can have different DV01s depending on how you perturb rates.

### 11.6.1 The Bump Taxonomy

| Bump Type | What Moves | Definition | Primary Use Case |
|-----------|------------|------------|------------------|
| **Yield Bump** | Single scalar YTM | Bump bond's own yield ±1bp | Bond trading, "Yield DV01" |
| **Zero-Rate Bump** | All zero rates | Parallel ±1bp to $z(t)$ for all $t$ | Standard "Curve DV01" for portfolios |
| **Par-Point Bump** | Market input quotes | Bump one quote ±1bp, rebuild curve | "Quote PV01," hedging with liquid instruments |
| **Key-Rate Bump** | One maturity bucket | Triangular ±1bp at specific tenor | Localized curve risk (Chapter 14) |

### 11.6.2 Par-Instrument Bump (Quote PV01)

Risk engines often report sensitivity to the *input quotes* used to build the curve (e.g., cash rates, futures, swap rates). This is called **Quote PV01** or **Par-Point Delta**.

> **Example: Quote PV01**
>
> Suppose a curve is built from par swap rates at 2y, 5y, 10y.
> We want the sensitivity of a **7-year bond** to the **5-year swap rate**.
>
> 1. **Base:** Build curve from market rates. Price bond = 108.25.
> 2. **Bump:** Increase the *5y swap rate input* by 1bp. Re-run the bootstrap.
> 3. **Reprice:** New bond price = 108.21.
> 4. **Result:** The 7-year bond has Quote PV01 of 0.04 to the 5-year swap rate.
>
> This is highly practical for hedging: it tells you exactly how much of the 5y swap to trade to hedge the bond's sensitivity to that input.

### 11.6.3 Key-Rate Constraints

When bumping a single point on a curve (Key-Rate DV01), you must define how the rest of the curve behaves. Tuckman's standard approach uses a "triangular" bump:

- 0bp shift at adjacent key rates
- +1bp shift at the target key rate
- Linear interpolation between

For example, a 5y key-rate bump with 2y and 10y as neighbors:
- 0bp at 2y
- +1bp at 5y
- 0bp at 10y
- Linear interpolation in between

This isolates the risk of the 5-year sector. Full treatment is in **Chapter 14**.

---

## 11.7 Practical Notes

### 11.7.1 Common Ambiguity Traps

1. **Sign conventions:** The Tuckman convention (PV change for 1bp *decline*) and Hull convention (1bp *increase*) differ by sign. Always verify before combining risk numbers.

2. **Clean vs Dirty:** Risk systems typically compute sensitivities on **full price** (dirty), but traders quote **clean price**. Since accrued interest is locally time-dependent only (not rate-dependent), $d(AI)/dy \approx 0$, so Clean DV01 ≈ Dirty DV01. However, precise systems should use full/dirty price to avoid edge case errors.

3. **Compounding:** A "1bp bump" to a continuous rate is economically smaller than a "1bp bump" to an annual rate. Ensure consistency across all curve inputs and outputs.

4. **Day count:** When bumping swap curves, be precise about whether the bump is to the rate expressed in the curve's native day count (e.g., ACT/360) or converted.

### 11.7.2 Verification Tests for DV01 Engines

When building or validating a DV01 engine, run these tests:

1. **Sign Check:** For a long fixed-rate bond, DV01 must be positive under Tuckman convention.

2. **Notional Scaling:** $100m face should have exactly 100x the risk of $1m face.

3. **Additivity:** Ensure Portfolio DV01 equals the sum of the parts (under consistent bumps).

4. **Convergence:** DV01 computed with 0.1bp bump should be very close to 1bp bump (for smooth pricing functions).

5. **Symmetry:** For instruments without optionality, $(V_\downarrow - V_0) \approx (V_0 - V_\uparrow)$ when convexity is small.

6. **Zero-Coupon Benchmark:** A zero-coupon bond's DV01 should equal $T \times DF / (1+r) / 10,000$ (modified duration × price / 10,000).

---

## 11.8 Summary

1. **DV01** is the change in dollar value for a 1 basis point change in rates. It requires a specific definition of *which* rate changes.

2. **Yield DV01** (bumping YTM) is standard for bonds but fails to capture curve shape risk. Tuckman: "this hedge will work as intended only if the yield of the bond bought changes by the same amount" as the hedge.

3. **Curve DV01** (bumping the zero curve) is standard for portfolios and swaps. It differs from Yield DV01 because yield is a weighted average of spot rates.

4. **PVBP** usually refers to the sensitivity of a swap to its *fixed rate* (an instrument feature), distinct from its sensitivity to the *discount curve* (a market feature).

5. **Bump-and-Reprice** with central differences $(V_\downarrow - V_\uparrow) / 2$ is the robust numerical method for computation.

6. **Portfolio Additivity** only holds if all instruments are bumped by the same defined market shift.

7. **Multi-Curve** frameworks split risk into Discount (OIS) and Projection (term rate) components, essential for proper swap hedging.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Yield DV01** | Sensitivity to own YTM ($dP/dy$) | "Trader's DV01" for bonds; assumes parallel yield shift across securities |
| **Curve DV01** | Sensitivity to discount curve | "Risk Manager's DV01"; additive across portfolios under consistent bump |
| **PVBP** | Sensitivity to contractual fixed rate | Used for pricing new swaps; equals Notional × Annuity × 0.0001 |
| **Quote PV01** | Sensitivity to market input quotes | Recipes for hedging with liquid instruments |
| **Central Difference** | $(V_\downarrow - V_\uparrow)/2$ | Removes convexity bias from first-order risk estimates |
| **Annuity Factor** | $\sum \tau_i P(0,T_i)$ | Present value of receiving 1 unit at each payment date |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $\text{DV01}$ | Dollar value of one basis point |
| $P(y)$ | Price as function of yield |
| $A$ | Annuity factor for swaps |
| $S$ | Par swap rate |
| $K$ | Contractual fixed rate |
| $P(0,T)$ | Discount factor from time 0 to $T$ |
| $z(T)$ | Zero/spot rate to maturity $T$ |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the Tuckman sign convention for DV01? | DV01 is positive for a long bond (PV change for 1bp **fall** in rates). |
| 2 | What is the difference between Yield DV01 and Curve DV01? | Yield DV01 bumps a single scalar (bond's YTM); Curve DV01 bumps the entire term structure of zero rates. |
| 3 | Formula for DV01 using central finite differences? | $\text{DV01} = (V(y-1\text{bp}) - V(y+1\text{bp})) / 2$ |
| 4 | If Portfolio A has yield-based DV01 and Portfolio B has curve-based DV01, can you add them? | No, they represent sensitivities to different stress scenarios and are not comparable. |
| 5 | What is the swap annuity factor $A$? | The present value of 1 unit paid on each fixed-leg date: $\sum \tau_i P(0,T_i)$. |
| 6 | What does "PVBP" typically mean for a swap? | Sensitivity to the fixed coupon rate $K$ (equals Notional × Annuity × 0.0001), not the discount curve. |
| 7 | In a multi-curve swap (Receive Fixed), which curve usually drives the risk? | The Projection curve (drives floating payments), though Discount curve affects PV factors. |
| 8 | Why use central differences instead of one-sided bumps? | To eliminate second-order (convexity) errors for better first-derivative accuracy. |
| 9 | What is the unit consistency check for DV01? | If prices are in dollars, DV01 is in dollars per bp. If price is per 100, DV01 is per 100 per bp. |
| 10 | Why is Yield DV01 poor for hedging curve twists? | It assumes a rigid shift in the bond's average rate, ignoring how different segments of the curve might move differently. |
| 11 | What is Quote PV01? | Sensitivity to bumping a single input quote (like 5y swap rate) and rebuilding the curve. |
| 12 | How does Tuckman define "key rate 01"? | The change in value when only one key rate shifts by 1bp (triangular bump with neighbors fixed). |
| 13 | What is the relationship between DV01 and Modified Duration? | $\text{DV01} = P \times D_{\text{Mod}} / 10,000$ |
| 14 | For a zero-coupon bond, what is the DV01 formula? | $\text{DV01} = T \times P / (1+y) / 10,000$ where $P$ is price and $T$ is maturity. |
| 15 | Why does bumping rates vs bumping discount factors give different results? | The relationship $DF = 1/(1+r)^T$ is nonlinear, so linear DF bumps don't match exact rate bumps. |

---

## Mini Problem Set

**1. Basic DV01 Calculation**
A bond priced at 98.50 increases to 98.55 if yields fall 1bp. It falls to 98.45 if yields rise 1bp. Calculate the DV01 per 100 face using Tuckman's convention.

*Solution:* Using central differences: $\text{DV01} = (98.55 - 98.45)/2 = 0.05$ per 100 face.

**2. Dollar Risk Scaling**
You own $10 million face value of the bond in Q1. What is the total dollar DV01?

*Solution:* $0.05 \times (10,000,000 / 100) = \$5,000$ per bp.

**3. Hedge Ratio**
You are long Bond A (DV01 = 0.06 per 100). You want to hedge with Bond B (DV01 = 0.04 per 100). What is the hedge ratio?

*Solution:* Ratio $= 0.06 / 0.04 = 1.5$. Short 1.5× the face amount of Bond B.

**4. Par Swap PVBP**
A 5-year par swap has rate 4.00%. The Annuity Factor is 4.50. What is the PVBP for $100mm notional?

*Solution:* $\text{PVBP} = 100,000,000 \times 4.50 \times 0.0001 = \$45,000$ per bp.

**5. Curve vs Yield Intuition**
Explain why bumping the zero curve by +1bp might change a bond's yield by 0.98bp rather than 1.00bp.

*Solution:* The bond's price is a weighted sum of discounted cashflows. A parallel shift in zeros affects near and far cashflows differently. The single "Yield" that explains the new price is a complex weighted average that doesn't move 1-to-1 with the curve shift.

**6. Convexity Error**
If you only bumped UP by 1bp (one-sided), why might your DV01 estimate be biased?

*Solution:* Bond prices are convex. The price loss for +1bp is smaller than the price gain for -1bp. A one-sided up bump underestimates the average sensitivity, while a one-sided down bump overestimates it. Central differences average these effects.

**7. Multi-Curve Intuition**
In a Receive-Fixed swap that starts at par, why is Discount Curve risk (OIS PV01) usually small?

*Solution:* The swap starts with PV ≈ 0. The discount factor sensitivity scales with the net PV. If PV is small, the "discounting effect" is minor. The main risk is the "projection effect" on the floating leg expectations.

**8. Sign Convention Translation**
A trader says "The DV01 is -500." What does this likely mean?

*Solution:* They are using the "Hull" or "calculus" convention where positive DV01 means the portfolio gains when rates rise. A negative value means it loses if rates rise. Under Tuckman convention, we would say "The risk is +500" (implying a long position that benefits from rate declines).

---

## Source Map

### (A) Verified Facts — Source Citations

| Fact | Source |
|------|--------|
| DV01 definition: "change in value for a one-basis point decline in rates" | Tuckman Ch 5 |
| DV01 formula: $-\Delta P / (10,000 \times \Delta y)$ | Tuckman Ch 5, Eq 5.1 |
| "The negative sign defines DV01 to be positive if price increases when rates decline" | Tuckman Ch 5 |
| Yield-based DV01 is special case where yield is the rate factor | Tuckman Ch 6 |
| "DV01 of a portfolio is the sum of the individual DV01 values" | Tuckman Ch 5 |
| Explicit DV01 formula with time-weighted PVs | Tuckman Ch 6, Eq 6.5 |
| "This hedge will work...only if the yield of the bond bought changes by the same amount" | Tuckman Ch 6 |
| Swap Annuity Factor definition $A = \sum \tau_n P(t, T_{n+1})$ | Andersen & Piterbarg Vol 1, Ch 4 |
| Annuity as PVBP concept | Andersen & Piterbarg Vol 1 |
| Multi-Curve risk decomposition | Andersen & Piterbarg Vol 1, Ch 6 |
| Duration measures "sensitivity of value...to a small parallel shift in the zero-coupon yield curve" | Hull, Risk Mgmt Ch 9 |
| DV01 as "dollar value of a 1-basis-point increase in all rates" (Hull convention) | Hull, OFD Glossary |
| Key rate exposures "decompose a sensitivity measure like DV01...into component sensitivities" | Tuckman Ch 7 |
| "The most stable numerical estimate chooses rates that are equally spaced above and below" | Tuckman Ch 5 |

### (B) Reasoned Inference

- **Section 11.3.2:** The divergence between Yield and Curve DV01 is derived from the mathematical fact that yield is a complex weighted average of spot rates; bumping one changes the other non-linearly.
- **Section 11.4.4:** Multi-curve example logic follows directly from the definition of decoupled discount/projection curves in post-crisis frameworks (Andersen & Piterbarg Vol 1).
- **Sign convention translation (Q8):** Follows from comparing Tuckman's explicit convention with Hull's glossary definition.

### (C) Flagged Uncertainties

- **Desk Naming Conventions:** "PV01," "PVBP," and related terms are used differently across firms and systems. We have flagged this as a "Collision" to warn readers; complete standardization is impossible to verify.
- **Exact numbers in multi-curve example (11.4.4):** Illustrative; actual values depend on specific curve shapes and swap terms.
