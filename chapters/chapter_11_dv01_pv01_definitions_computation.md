# Chapter 11: DV01/PV01 — Definitions, Computation, and "What's Being Bumped"

---

## Introduction

A portfolio manager reports that her bond position has a "DV01 of $50,000." A swaps trader says his book is "flat on PV01." A risk officer asks for "key-rate exposures by bucket." Each statement sounds precise—a dollar measure of interest rate risk per basis point. But hidden in these simple phrases lurks a critical question that, if misunderstood, can render all these numbers meaningless: *what exactly is being bumped?*

Consider the portfolio manager's $50,000 DV01. Does that number assume the bond's yield moves by 1 basis point? Or does it assume every zero rate on the discount curve shifts by 1bp? Perhaps it reflects bumping the par swap quotes and rebuilding the curve? These are different economic scenarios, and they can produce materially different dollar sensitivities for the same position. As Tuckman emphasizes in *Fixed Income Securities*, DV01 is defined as "the change in value of a fixed income security for a one-basis point decline in rates"—but this definition is meaningful only after you specify *which* interest rate measure moves and *how*.

The consequences of ambiguity are real—and expensive. A trader who hedges using yield-based DV01 but marks-to-market against a curve-based pricing engine will experience unexplained P&L when the curve twists. A risk system that aggregates DV01s computed under inconsistent bump definitions will produce portfolio sensitivities that don't add up. And a hedger who matches total DV01 without understanding key-rate exposures may find the hedge worthless when the curve steepens instead of shifting in parallel.

> **The Tower of Babel Problem**
>
> On a large trading floor, the rates desk might compute "DV01" by bumping the swap curve and rebuilding zeros. The Treasury desk might bump bond yields directly. The risk team might use a third definition involving par-point deltas. When these numbers flow into a single risk report, the sum is nonsensical—like adding feet to meters. Before you can aggregate risk, you must ensure everyone speaks the same language.

This chapter provides a rigorous foundation for interest rate sensitivity measures. We define DV01 formally in **Section 11.1**, emphasizing the sign convention and the requirement for specificity. **Section 11.2** covers **yield-based DV01**—the trader's measure for individual bonds—while **Section 11.3** develops **curve-based DV01** and explains why it differs. **Section 11.4** introduces **PVBP** for swaps and the critical multi-curve decomposition. **Section 11.5** details the computational mechanics of bump-and-reprice. **Section 11.6** presents a complete taxonomy of bump specifications—from parallel shifts to key-rate perturbations—answering definitively: "What is being bumped?" **Section 11.7** covers practical pitfalls and verification tests. Finally, **Section 11.8** connects DV01 to the classical immunization framework.

The concepts developed here are foundational for all subsequent chapters on risk measures. Chapter 12 derives duration from DV01. Chapter 13 explains convexity—the reason our numerical methods must use central differences. Chapter 14 extends the bump taxonomy to multi-factor key-rate analysis. And Chapter 15 applies these tools to hedging strategies.

---

## Conventions and Notation

| Symbol | Definition |
|--------|------------|
| $P$ | Bond full (dirty) price per 100 face (unless stated otherwise) |
| $P_{\text{flat}}$ | Bond flat/clean price; $P_{\text{flat}} = P_{\text{full}} - \text{AI}$ |
| $\text{AI}$ | Accrued interest |
| $y$ | Yield to maturity (IRR of cashflows against price) |
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
>
> **Translation rule:** $\text{DV01}_{\text{Tuckman}} = -\text{DV01}_{\text{Hull}}$

---

## 11.1 DV01: The Dollar Value of a Basis Point

### 11.1.1 The Formal Definition

Tuckman introduces DV01 by considering the **price-rate function** $P(y)$, where $y$ is some interest rate factor. He then defines:

> "Letting $\Delta P$ and $\Delta y$ denote the changes in price and rate and noting that the change measured in basis points is $10,000 \times \Delta y$, define the following measure of price sensitivity: DV01 is an acronym for dollar value of an '01 (i.e., .01%) and gives the change in the value of a fixed income security for a one-basis point decline in rates." (Tuckman, Chapter 5)

The formal expression, using this convention, is:

$$\boxed{\text{DV01} \equiv -\frac{\Delta P}{10,000 \times \Delta y}}$$

The factor of 10,000 converts the rate change from decimal units (where 1bp = 0.0001) to a "per basis point" measure. The negative sign ensures that DV01 is positive when prices increase as rates decline—which is the typical case for fixed-coupon bonds. Tuckman explains: "The negative sign defines DV01 to be positive if price increases when rates decline and negative if price decreases when rates decline. This convention has been adopted so that DV01 is positive most of the time."

When an explicit price-rate function $P(y)$ is available and differentiable, DV01 can be expressed using the derivative:

$$\boxed{\text{DV01} = -\frac{1}{10,000}\frac{dP}{dy}}$$

This derivative form is exact in the limit of infinitesimal rate changes. Graphically, $dP/dy$ represents the slope of the tangent line to the price-rate curve at the current rate level.

> **Analogy: The Speedometer on Your P&L Dashboard**
>
> Think of DV01 as the speedometer on your P&L dashboard:
> - **Yield Change**: The car's speed (how fast rates are moving, in bp).
> - **DV01**: The gear ratio (how much P&L per unit of rate movement).
> - **Equation**: Speed (bp) × Gear (DV01) = Distance (Dollars P&L).
>
> If your DV01 is $50,000 and the market moves 10 bps, you just traveled $500,000 in P&L space. The direction depends on whether you're long or short rates.

### 11.1.2 What DV01 Answers

DV01 answers a simple question: *"If rates move by 1bp (in the way I've defined), how many dollars do I gain or lose?"*

This is a **first-order Taylor approximation** of the price change. For a small rate move $\Delta y$, the price change is approximately:

$$\Delta P \approx -10,000 \times \text{DV01} \times \Delta y$$

Hull (*Risk Management and Financial Institutions*) provides a complementary perspective, defining **dollar duration** as the product of duration and price: $D_{\$} = -dB/dy$. This is mathematically equivalent to $10,000 \times \text{DV01}$ under Tuckman's convention. Hull emphasizes that "dollar duration is similar to the delta measure discussed in [options]"—reinforcing that DV01 is the fixed-income equivalent of the options trader's delta.

> **The Einstein Formula of Fixed Income**
>
> We can rewrite risk in percentage terms to get the most important equation in bond math:
>
> $$\frac{dP}{P} \approx -D \times dy$$
>
> where $D$ is modified duration. This says: **Change in Price (%)** equals **Duration** times **Change in Yield**. DV01 is just the *dollar version* of this: $\text{DV01} = P \times D_{\text{Mod}} / 10{,}000$.

Crucially, the DV01 definition requires specifying the "rate measure" $y$ being bumped. While DV01 can be computed "for any interest rate measure," market usage typically implies **yield-based DV01** for individual bonds and **curve-based DV01** for portfolios and derivatives.

### 11.1.3 Portfolio Aggregation and Additivity

Tuckman proves that for a portfolio of positions, DV01 is additive: "the DV01 of a portfolio is the sum of the individual DV01 values." This follows from the linearity of the derivative operator. If portfolio value is $V_{\text{port}} = \sum V_i$, then:

$$\text{DV01}_{\text{portfolio}} = \sum_i \text{DV01}_i$$

**But there is a critical caveat**: this additivity holds only if the same bump definition applies to all positions. If different positions use different bump definitions (e.g., one bumps yield, another bumps the zero curve), aggregation creates an "apples to oranges" sum that lacks economic meaning.

> **Example 11.1: Portfolio Additivity Under Consistent Bumps**
>
> Consider a portfolio with three positions valued on a flat 4% curve. We define "DV01" as the change in value for a parallel 1bp fall in all zero rates.
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

### 11.1.4 The Dollar Scaling Imperative

Why is DV01 the standard normalization? Why not just use duration?

Consider two traders with identical duration exposure:

- **Retail Trader ($10,000 trade)**: 5-year bond, Duration = 4.5.
  - 10bp move → 0.45% price change.
  - P&L = $45. Manageable.

- **Institutional Desk ($100,000,000 trade)**: Same bond, Duration = 4.5.
  - 10bp move → 0.45% price change.
  - P&L = **$450,000**. Material.

**Lesson**: Percentage change (Duration) hides the scale. DV01 converts everything into "Dollars per bp", which is the only currency that matters for the P&L statement.

> **Superpower: Risk is Additive in Dollars**
>
> You cannot add yields (5% + 4% = ??). You cannot add prices ($100 + $98 = ??). But you **can** add DV01s:
> - $50k DV01 + $20k DV01 = $70k DV01.
>
> This forces a "Common Currency" of risk across the entire trading floor. Everyone from the rates desk to the credit desk speaks in dollars-per-basis-point.

### 11.1.5 The DV01-Duration Mapping

The relationship between DV01 and modified duration is fundamental. Starting from the definition of modified duration:

$$D_{\text{Mod}} = -\frac{1}{P}\frac{dP}{dy}$$

and the DV01 formula:

$$\text{DV01} = -\frac{1}{10,000}\frac{dP}{dy}$$

we can derive:

$$\boxed{\text{DV01} = \frac{P \times D_{\text{Mod}}}{10{,}000}}$$

This formula is essential for translating between the "percentage risk" language of asset managers (duration) and the "dollar risk" language of traders (DV01).

> **Example 11.2: DV01-Duration Conversion**
>
> A 10-year Treasury bond priced at 95.50 has modified duration 7.8. What is its DV01?
>
> $$\text{DV01} = \frac{95.50 \times 7.8}{10{,}000} = 0.0745 \text{ per 100 face}$$
>
> For a $10,000,000 position: $\text{DV01}_{\text{total}} = 0.0745 \times 100{,}000 = \$7,450$ per bp.

---

## 11.2 Yield-Based DV01: Bumping a Single Scalar

### 11.2.1 The Yield-Based Framework

When bond traders speak of "DV01," they typically mean **yield-based DV01**. This measures the sensitivity to the bond's own yield-to-maturity (YTM). Tuckman describes this as "a special case of DV01...the yield of a security is the interest rate factor."

Recall from Chapter 6 that YTM is the single discount rate that equates cashflows to the price. For a bond with coupon $c$ per 100 face, maturity $T$ years, and semiannual compounding:

$$P(y) = \sum_{t=1}^{2T} \frac{c/2}{(1+y/2)^t} + \frac{100}{(1+y/2)^{2T}}$$

Yield-based DV01 is computed by bumping this single scalar $y$. Using central differences:

$$\text{DV01}_{\text{yield}} = \frac{P(y - 1\text{bp}) - P(y + 1\text{bp})}{2}$$

The factor of 2 in the denominator (rather than the $10,000 \times \Delta y$ in the general formula) appears because we're using a 2bp total shift (±1bp from the base) and expressing the result in "per bp" terms.

### 11.2.2 Explicit Formula for Yield-Based DV01

Tuckman derives an explicit formula by differentiating the bond price equation. The result is:

$$\boxed{\text{DV01} = \frac{1}{10,000} \times \frac{1}{1+y/2}\left[\sum_{t=1}^{2T} \frac{t}{2} \cdot \frac{c/2}{(1+y/2)^t} + T \cdot \frac{100}{(1+y/2)^{2T}}\right]}$$

This formula has a clear interpretation: DV01 equals "the sum of the time-weighted present values of a bond's cash flows divided by 10,000 multiplied by one plus half the yield." The time-weighting reflects that longer-dated cashflows contribute more to interest rate sensitivity.

The term in brackets is the sum of (time × PV) for each cashflow—this is precisely the numerator of the Macaulay duration formula, which explains the tight relationship between DV01 and duration.

### 11.2.3 Worked Example: Computing DV01 from First Principles

Tuckman illustrates the computation with a detailed example (Table 6.1 in his text). Let us replicate this methodology completely.

> **Example 11.3: Time-Weighted PV Calculation for DV01**
>
> **Bond:** 5-year U.S. Treasury with 5.625% annual coupon, yield 5.0441%, settling on a coupon date.
>
> | Period | Term (yrs) | Cash Flow | PV Factor $(1+y/2)^{-t}$ | Present Value | Time × PV |
> |--------|-----------|-----------|--------------------------|---------------|-----------|
> | 1 | 0.5 | 2.8125 | 0.9755 | 2.7433 | 1.3717 |
> | 2 | 1.0 | 2.8125 | 0.9517 | 2.6758 | 2.6758 |
> | 3 | 1.5 | 2.8125 | 0.9283 | 2.6100 | 3.9150 |
> | 4 | 2.0 | 2.8125 | 0.9055 | 2.5458 | 5.0916 |
> | 5 | 2.5 | 2.8125 | 0.8832 | 2.4832 | 6.2079 |
> | 6 | 3.0 | 2.8125 | 0.8614 | 2.4221 | 7.2662 |
> | 7 | 3.5 | 2.8125 | 0.8402 | 2.3625 | 8.2687 |
> | 8 | 4.0 | 2.8125 | 0.8194 | 2.3044 | 9.2175 |
> | 9 | 4.5 | 2.8125 | 0.7990 | 2.2477 | 10.1146 |
> | 10 | 5.0 | 102.8125 | 0.7792 | 80.1444 | 400.7219 |
> | **Sums** | | | | **102.5391** | **454.8511** |
>
> The DV01 calculation follows Tuckman's formula:
>
> $$\text{DV01} = \frac{1}{10,000} \times \frac{454.8511}{1 + 0.050441/2} = \frac{1}{10,000} \times \frac{454.8511}{1.0252} = \mathbf{0.044366}$$
>
> **Interpretation:** A one-basis point decline in the bond's yield increases its price by about 4.44 cents per $100 face value.
>
> **Sanity Check:** For a $100 million position, DV01 = $0.044366 × 1,000,000 = $44,366 per bp. A 10bp rally gains approximately $443,660.

### 11.2.4 The Limitation: Parallel Yield Shifts

Tuckman explicitly warns about the key limitation of yield-based DV01:

> "Given the assumptions in computing DV01, however, this hedge will work as intended only if the yield of the bond bought changes by the same amount as the yield of the [hedge]. It is in this sense that DV01 (along with the other measures of price sensitivity in this chapter) requires parallel yield shifts."

This is a fundamental weakness. If you hedge a position using yield-based DV01, but the yields of the two bonds move by different amounts, the hedge will fail. Yields can diverge due to:

1. **Curve shape changes:** The yield curve steepens or flattens
2. **Credit spread moves:** One bond's spread widens while another's tightens
3. **Liquidity shifts:** On-the-run vs off-the-run dynamics
4. **Repo specials:** Financing costs affect implied yields differently

This limitation motivates the multi-factor methods in Chapter 14 (key-rate analysis) and Chapter 16 (curve hedging).

---

## 11.3 Curve-Based DV01: Bumping a Curve Object

### 11.3.1 From Yield to Curve

An alternative approach is **curve-based DV01**. Here, we price the bond (or any instrument) using a discount factor curve $P(0,t)$ derived from a term structure model, rather than the bond's own yield:

$$V = \sum_i \text{CF}_i \times P(0, t_i)$$

Curve-based DV01 specifies a perturbation to the curve—for example, a parallel shift to all continuously compounded zero rates:

$$z(t) \mapsto z(t) + 1\text{bp} \quad \text{for all } t$$

After the shift, we recalculate all discount factors:

$$P_{\text{shifted}}(0,t) = e^{-(z(t) + 0.0001) \times t}$$

and reprice the instrument.

### 11.3.2 Worked Example: Curve DV01 via Zero-Curve Bump

> **Example 11.4: Computing Curve DV01 Step-by-Step**
>
> Consider a 5-year bond with 5% annual coupon, priced off a flat zero curve at 4.5% continuously compounded.
>
> **Step 1: Compute base discount factors**
>
> | Maturity | Zero Rate $z(t)$ | Discount Factor $P(0,t) = e^{-z \cdot t}$ |
> |----------|------------------|-------------------------------------------|
> | 1 | 4.50% | 0.95600 |
> | 2 | 4.50% | 0.91393 |
> | 3 | 4.50% | 0.87372 |
> | 4 | 4.50% | 0.83527 |
> | 5 | 4.50% | 0.79852 |
>
> **Step 2: Compute base price**
>
> $$P_0 = 5 \times (0.95600 + 0.91393 + 0.87372 + 0.83527) + 105 \times 0.79852$$
> $$P_0 = 5 \times 3.57892 + 83.8446 = 17.8946 + 83.8446 = 101.7392$$
>
> **Step 3: Bump zero rates up by 1bp (to 4.51%)**
>
> | Maturity | Shifted DF |
> |----------|------------|
> | 1 | 0.95590 |
> | 2 | 0.91375 |
> | 3 | 0.87346 |
> | 4 | 0.83493 |
> | 5 | 0.79812 |
>
> $$P_{\uparrow} = 5 \times 3.57804 + 105 \times 0.79812 = 17.8902 + 83.8026 = 101.6928$$
>
> **Step 4: Bump zero rates down by 1bp (to 4.49%)**
>
> | Maturity | Shifted DF |
> |----------|------------|
> | 1 | 0.95610 |
> | 2 | 0.91412 |
> | 3 | 0.87398 |
> | 4 | 0.83561 |
> | 5 | 0.79893 |
>
> $$P_{\downarrow} = 5 \times 3.57981 + 105 \times 0.79893 = 17.8991 + 83.8877 = 101.7868$$
>
> **Step 5: Compute Curve DV01 via central difference**
>
> $$\text{DV01}_{\text{curve}} = \frac{P_{\downarrow} - P_{\uparrow}}{2} = \frac{101.7868 - 101.6928}{2} = \mathbf{0.0470}$$
>
> **For comparison:** If we computed yield-based DV01 for this same bond (using its yield rather than the zero curve), we would get approximately 0.0458—a 2.6% difference.

### 11.3.3 Why Yield DV01 and Curve DV01 Differ

The divergence between yield DV01 and curve DV01 is not a computational error—it reflects a fundamental conceptual difference.

**The yield is a weighted average of spot rates.** When we compute a bond's YTM, we're finding a single rate that, when applied to all cashflows, reproduces the bond's price. This rate is a complex weighted average of the spot rates that would actually apply to each cashflow.

**A 1bp change in yield ≠ a 1bp parallel shift in zeros.** When you bump the yield by 1bp, you're implicitly changing the "implied spot curve" that would price this specific bond. This implied curve pivots in a complex way that depends on the bond's cash flow structure. Conversely, when you shift all zeros by 1bp, the bond's yield changes by something other than exactly 1bp—typically close, but not identical.

Tuckman warns that yield-based DV01 "can mis-measure risk when the curve does not move in the assumed way." If you hedge a curve-based P&L using yield-based metrics, you will see residual risk when the curve twists or steepens.

> **Desk Reality: The Basis Trap**
>
> Imagine you're a portfolio manager who hedges a 10-year corporate bond using Treasury futures. Your risk system shows:
> - **Corporate bond:** Yield DV01 = $78,000
> - **Treasury futures hedge:** Curve DV01 = $78,000 (notionally matched)
>
> Everything looks hedged. But tomorrow:
> - The Treasury curve shifts parallel by −5bp
> - Credit spreads widen by 3bp on your corporate
>
> Your Treasury hedge gained $390,000 (5bp × $78k). But your corporate lost only $156,000 (net 2bp × $78k after spread widening). Your "hedged" position just made $234,000 of unexplained P&L.
>
> **Lesson:** Yield DV01 conflates rate risk with spread risk. Curve DV01 separates them—but you need to hedge spreads separately.

### 11.3.4 When to Use Which

| Scenario | Preferred Measure | Rationale |
|----------|------------------|-----------|
| Single bond trading | Yield DV01 | Market quotes bonds by yield; traders think in yield terms |
| Portfolio risk aggregation | Curve DV01 | Consistent bump across all instruments |
| Swap/derivatives trading | Curve DV01 | Swaps don't have a single "yield" |
| Hedging across instruments | Curve DV01 | Ensures hedge and position use same curve |
| Relative value (bonds vs bonds) | Yield DV01 | Natural for spread analysis |
| Relative value (bonds vs swaps) | Curve DV01 | Required for meaningful comparison |

---

## 11.4 PVBP, Swap Annuity, and Multi-Curve Decomposition

### 11.4.1 The Swap Annuity Factor

For interest rate swaps, the concept of "duration" is handled via the **Annuity Factor** (or "PV01 of the fixed leg"). Andersen and Piterbarg define the annuity as:

$$\boxed{A(t) = \sum_{n} \tau_n P(t, T_{n+1})}$$

where $\tau_n$ is the year fraction for payment period $n$ and $P(t, T_{n+1})$ is the discount factor to payment date $T_{n+1}$.

Andersen notes that "The quantity $A(\cdot)$ is the annuity of the swap (or its PVBP, for Present Value of a Basis Point)"—making explicit the connection between the annuity factor and basis-point sensitivity.

The par swap rate $S$ is the fixed rate that makes the swap have zero value at inception:

$$S = \frac{P(0,T_0) - P(0,T_m)}{A}$$

where $T_0$ is the start date and $T_m$ is the maturity. For a spot-starting swap, $P(0,T_0) = 1$.

### 11.4.2 PVBP: Sensitivity to the Fixed Rate

In swap trading, **PVBP** (Present Value of a Basis Point) often refers to the change in PV resulting from bumping the **contractual fixed rate** $K$ by 1bp, while holding the curve fixed.

For a receiver swap (receive fixed, pay floating) with notional $N$:

$$V_{\text{swap}} = N(K - S) A$$

The sensitivity to the fixed rate $K$ is:

$$\frac{\partial V}{\partial K} = N \times A$$

Thus:

$$\boxed{\text{PVBP} = N \times A \times 0.0001}$$

(The 0.0001 converts to basis points.)

> **Example 11.5: Swap PVBP Calculation**
>
> Consider a $100,000,000 notional 5-year receiver swap at fixed 4.50% vs floating. The discount curve is flat at 4%.
>
> **Step 1: Compute Annuity Factor** (annual payments for simplicity)
>
> $$A = \sum_{t=1}^5 P(0,t) = \sum_{t=1}^5 (1.04)^{-t} = 0.9615 + 0.9246 + 0.8890 + 0.8548 + 0.8219 = 4.4518$$
>
> **Step 2: Compute PVBP**
>
> $$\text{PVBP} = \$100{,}000{,}000 \times 4.4518 \times 0.0001 = \$44{,}518$$
>
> **Interpretation:** A 1bp increase in the fixed rate you receive (from 4.50% to 4.51%) adds $44,518 to the swap's value. This is independent of market rate moves—it's purely about the contract terms.
>
> **Note:** If we calculated a **Curve DV01** by bumping the discount rates, we would get a different number, because changing discount rates affects both the annuity and the implicit floating leg value. PVBP (fixed rate sensitivity) and Curve DV01 are conceptually distinct.

### 11.4.3 The "PV01" Naming Collision

On trading desks, "PV01" is an overloaded term that can mean different things depending on context:

| Term | Meaning | Context |
|------|---------|---------|
| **Curve PV01** | Bumping the discount curve by 1bp | Risk management, hedging |
| **PVBP / Rate PV01** | Bumping the fixed rate by 1bp | New deal pricing, annuity calculation |
| **Risky PV01** | Bumping credit spreads by 1bp | CDS trading (Chapter 41) |
| **Par-Point PV01** | Bumping a single input quote, rebuilding curve | Curve risk (Section 11.6.2) |

Always clarify: "Is this PV01 from bumping the curve, or the coupon?" Quantitative developers must strictly separate these in code to avoid confusion.

### 11.4.4 Multi-Curve Risk Decomposition

In modern post-crisis markets, we use different curves for:
- **Discounting:** OIS (overnight indexed swap) curve, reflecting collateralized funding
- **Projecting floating rates:** Term SOFR or legacy IBOR curves

This splits "PV01" into two components: **OIS PV01** (discount curve sensitivity) and **Projection PV01** (forecast curve sensitivity).

> **Example 11.6: Multi-Curve Risk Split for a 3-Year Swap**
>
> Consider a 3-year receiver swap, receiving fixed 4.00% vs 3M SOFR. Notional = $10,000,000.
> - **Discount Curve:** OIS flat at 3.50%
> - **Projection Curve:** 3M SOFR flat at 3.75%
>
> The swap is approximately at-the-money (near par value).
>
> **Risk decomposition:**
>
> | Risk Component | Bump Applied | Result |
> |----------------|--------------|--------|
> | **OIS Discount PV01** | Bump OIS curve only by +1bp | $8.50 |
> | **Projection Curve PV01** | Bump projection curve only by +1bp | $295.00 |
> | **Total PV01** | Both curves +1bp | $303.50 |
>
> **Why does Projection PV01 dominate?** For a near-par swap, the present value is close to zero—the fixed and floating legs nearly offset. When you bump the discount curve, both legs are affected similarly, so the net effect is small. But when you bump the projection curve, you directly increase the expected floating payments, which changes the swap value significantly.

> **Desk Reality: When OIS Risk Bites**
>
> Junior traders often ignore OIS risk because it's small for near-par swaps. But consider:
>
> 1. **Deep in-the-money swap:** You entered a receiver swap at 5.00% when rates were high. Now rates are 3.00%, and your swap has positive MTM of $2 million. The present value of this $2 million gain is highly sensitive to how you discount it—OIS PV01 is now material.
>
> 2. **Funding squeeze:** During the 2020 COVID crisis, OIS rates moved 50bp in days while projection curves moved less. A portfolio "hedged" on total PV01 but mismatched on the OIS/projection split showed large unexplained P&L.
>
> **Rule of thumb:** OIS risk is proportional to the swap's MTM (net PV). Projection risk is proportional to the swap's notional.

**Table: OIS vs Projection Sensitivity by Swap MTM**

| Swap Status | OIS PV01 | Projection PV01 | Primary Risk |
|-------------|----------|-----------------|--------------|
| At par (MTM ≈ 0) | Small | Large | Projection |
| Deep ITM (MTM >> 0) | Material | Large | Both |
| Deep OTM (MTM << 0) | Material | Large | Both |

---

## 11.5 Bump-and-Reprice Mechanics

### 11.5.1 The Central Finite Difference Algorithm

The gold standard for computing DV01 numerically is **central finite differences**:

1. **Base:** Price at current rates → $V_0$
2. **Up:** Price at rates $+1\text{bp}$ → $V_{\uparrow}$
3. **Down:** Price at rates $-1\text{bp}$ → $V_{\downarrow}$
4. **Calculate:** $\text{DV01} = (V_{\downarrow} - V_{\uparrow}) / 2$

Using a central difference cancels out the second-order (convexity) error term, providing a much more accurate estimate of the tangent slope than a one-sided bump.

Tuckman explains the rationale: "The most stable numerical estimate chooses rates that are equally spaced above and below" the rate level in question. This avoids the bias introduced by the curvature of the price-rate relationship.

$$\boxed{\text{DV01}_{\text{central}} = \frac{V_{\downarrow} - V_{\uparrow}}{2}}$$

### 11.5.2 Why Central Differences Outperform One-Sided Bumps

Consider a bond with price $P(y)$ that exhibits positive convexity. The price-rate relationship is curved—it's not a straight line.

**For a one-sided up bump:**

$$\text{DV01}_{\text{one-sided up}} = V_0 - V_{\uparrow}$$

This underestimates the true sensitivity because convexity causes the price to fall less for a rate increase than it rises for a rate decrease.

**For a one-sided down bump:**

$$\text{DV01}_{\text{one-sided down}} = V_{\downarrow} - V_0$$

This overestimates the true sensitivity.

**The central difference:**

$$\text{DV01}_{\text{central}} = \frac{V_{\downarrow} - V_{\uparrow}}{2}$$

averages the upward and downward sensitivities, effectively canceling the convexity bias.

Mathematically, the one-sided error is $O(\Delta y)$ (first-order), while the central difference error is $O(\Delta y^2)$ (second-order). For a 1bp bump, this means the central difference is roughly 100× more accurate.

> **Example 11.7: Central vs One-Sided Comparison**
>
> For a 10-year 5% bond at 4% yield (Price = 108.18):
>
> | Bump Direction | New Rate | New Price | DV01 Estimate |
> |----------------|----------|-----------|---------------|
> | Base | 4.00% | 108.1800 | — |
> | Up (+1bp) | 4.01% | 108.0907 | 0.0893 (one-sided) |
> | Down (−1bp) | 3.99% | 108.2707 | 0.0907 (one-sided) |
> | **Central** | | | **(0.0907 + 0.0893) / 2 = 0.0900** |
>
> **Observation:** The asymmetry (0.0907 vs 0.0893) reflects the bond's positive convexity—it gains more when rates fall than it loses when rates rise by the same amount.
>
> **The central estimate (0.0900) is closest to the true derivative.** The one-sided up estimate understates sensitivity; the one-sided down overstates it.

### 11.5.3 Explicit Derivative Formula

For simple bonds, we can derive the analytical derivative directly. Starting from:

$$P(y) = \sum_i \text{CF}_i (1+y/2)^{-t_i}$$

where $t_i$ is in semiannual periods (so a 5-year bond has $t_{10} = 10$).

Differentiating with respect to $y$:

$$\frac{dP}{dy} = -\sum_i \frac{t_i}{2} \cdot \frac{\text{CF}_i}{(1+y/2)^{t_i+1}}$$

Thus:

$$\boxed{\text{DV01} = \frac{1}{10,000} \sum_i \frac{(t_i/2) \cdot \text{CF}_i}{(1+y/2)^{t_i+1}}}$$

This formula confirms the intuition that longer-maturity cashflows (large $t_i$) contribute more to DV01, weighted by both the cashflow size and the discount factor.

### 11.5.4 Bump Size Selection

**Why is 1bp the standard bump size?**

1. **Market convention:** Rate quotes move in basis points
2. **Numerical stability:** Large enough to avoid floating-point noise, small enough for linearity
3. **Interpretability:** Direct mapping to "dollars per basis point"

**When to use smaller bumps (0.1bp or 0.01bp):**

1. **Exotic options with discontinuities:** Barrier options, digital options
2. **Stress testing convergence:** Verify that your DV01 engine is stable
3. **High-convexity instruments:** MBS, callable bonds with rate-dependent prepayment

**When to use larger bumps (5bp or 10bp):**

1. **Stress testing / VaR scenarios:** Direct computation of P&L under stress
2. **Instruments with path-dependency:** Monte Carlo pricing where re-simulation is expensive

### 11.5.5 Nonlinearity: Why Exact Bumps Matter

Does it matter if we bump the rate $r$ or the discount factor $DF$ directly?

Yes. The relationship $DF = 1/(1+r)^T$ is nonlinear.

> **Example 11.8: Rate Bump vs DF Approximation**
>
> Consider a single cashflow of 100 at $T=10$ years, $r=5\%$ (annual compounding).
>
> **Base values:** $DF_0 = 1/(1.05)^{10} = 0.6139$, $V_0 = 61.39$.
>
> **Method 1: Exact Rate Bump**
> Bump $r$ to $5.01\%$: $DF_{\uparrow} = 1/(1.0501)^{10} = 0.6133$.
> $V_{\uparrow} = 61.33$, DV01 = $61.39 - 61.33 = \mathbf{0.0584}$.
>
> **Method 2: Linear DF Approximation**
> Approximate $\Delta DF \approx -T \cdot DF \cdot \Delta r / (1+r) = -10 \times 0.6139 \times 0.0001 / 1.05 = -0.000585$.
> $DF_{\text{approx}} = 0.6139 - 0.000585 = 0.6133$, giving DV01 ≈ $\mathbf{0.0585}$.
>
> The difference is small for 1bp, but for larger shocks (VaR, stress tests), linear approximations break down significantly. **Production systems should always use exact re-pricing.**

---

## 11.6 "What's Being Bumped?"—A Taxonomy

The critical insight of this chapter is that **DV01 is a property of the bump definition, not just the instrument**. The same bond can have different DV01s depending on how you perturb rates.

### 11.6.1 The Bump Taxonomy

| Bump Type | What Moves | Definition | Primary Use Case |
|-----------|------------|------------|------------------|
| **Yield Bump** | Single scalar YTM | Bump bond's own yield ±1bp | Bond trading, "Yield DV01" |
| **Zero-Rate Bump** | All zero rates | Parallel ±1bp to $z(t)$ for all $t$ | Standard "Curve DV01" for portfolios |
| **Par-Point Bump** | Market input quotes | Bump one quote ±1bp, rebuild curve | "Quote PV01," hedging with liquid instruments |
| **Key-Rate Bump** | One maturity bucket | Triangular ±1bp at specific tenor | Localized curve risk (Chapter 14) |
| **Forward-Rate Bump** | Forward rates | Bump $f(t)$ for all $t$ | Alternative to zero-rate for some models |

> **Visual: The Bump Taxonomy Flowchart**
>
> ```
> "What's my DV01?"
>       │
>       ▼
> ┌─────────────────────────────────────────┐
> │     What are you bumping?               │
> └─────────────────────────────────────────┘
>       │
>       ├──► Yield (single bond) ──► Yield DV01
>       │
>       ├──► Zero curve (all tenors) ──► Curve DV01
>       │
>       ├──► Par quotes (inputs) ──► Quote PV01
>       │
>       └──► Key rates (specific tenors) ──► Key-Rate DV01s
> ```

### 11.6.2 Par-Point Bump (Quote PV01)

Risk engines often report sensitivity to the *input quotes* used to build the curve (e.g., deposit rates, futures prices, swap rates). This is called **Quote PV01**, **Par-Point Delta**, or **Instrument PV01**.

The algorithm is:
1. Take the set of market quotes $\{S_1, S_2, \ldots, S_N\}$ that define the curve
2. Bump quote $S_n$ by 1bp (holding all others fixed)
3. Rebuild the entire curve from the perturbed quotes
4. Reprice the instrument
5. The change in value is the Quote PV01 to instrument $n$

Andersen describes this as the "par-point approach" and notes that it provides sensitivities that "directly suggest hedging instruments."

> **Example 11.9: Quote PV01 and the Recipe Concept**
>
> A curve is built from par swap rates at 2y, 5y, 10y. We want the sensitivity of a **7-year bond** to each input.
>
> | Bump | Rebuilt Curve Effect | Bond Price Change | Quote PV01 |
> |------|---------------------|-------------------|------------|
> | 2y swap +1bp | Short end shifts | 108.25 → 108.23 | 0.02 |
> | **5y swap +1bp** | Mid-curve shifts | 108.25 → 108.21 | **0.04** |
> | 10y swap +1bp | Long end shifts | 108.25 → 108.24 | 0.01 |
> | **Total** | | | **0.07** |
>
> **The Recipe:** To hedge this bond's curve risk, trade:
> - Short $\frac{0.02}{\text{DV01}_{2y}}$ of 2y swap
> - Short $\frac{0.04}{\text{DV01}_{5y}}$ of 5y swap
> - Short $\frac{0.01}{\text{DV01}_{10y}}$ of 10y swap
>
> This gives you exactly the hedge amounts in each liquid instrument. Tuckman emphasizes: "To compute the hedge amount...simply divide each key rate exposure by the DV01" of the hedging instrument.

> **Desk Reality: Quote PV01 as Hedging Recipe**
>
> Quote PV01 tells you exactly how many contracts of each benchmark instrument to trade. This is why it's the preferred measure for desk-level hedging:
>
> - You see: "Bond has Quote PV01 of $4,000 to 5y swap"
> - You know: 5y swap has DV01 of $450 per million notional
> - You trade: Short $4,000 / 450 × 1mm = \$8.89mm$ notional of 5y swap
>
> No translation needed. The hedge amounts fall directly out of the Quote PV01 report.

### 11.6.3 The Jacobian Approach for Hedging

Andersen describes a more sophisticated approach using the **Jacobian matrix**. If you compute sensitivities to a set of forward rate bumps $\{\mu_k(t)\}$, you can translate these into hedge amounts via:

$$\mathbf{p} = \left(\partial \mathbf{H}^{\top}\right)^{-1} \partial \mathbf{V}_0$$

where $\mathbf{p}$ is the vector of hedge positions, $\partial \mathbf{H}$ is the Jacobian matrix of hedge instrument sensitivities, and $\partial \mathbf{V}_0$ is the vector of portfolio sensitivities.

Andersen notes: "The method of constructing a hedge portfolio from derivatives to arbitrary shocks of the forward curve via [this] optimization problem is known as the Jacobian method for interest rate deltas."

For most desk purposes, Quote PV01 provides a simpler alternative that directly maps to hedge amounts. The Jacobian approach is useful when:
- Hedging with non-standard instruments
- Optimizing across hedge costs
- Handling instruments that span multiple curve points

### 11.6.4 Key-Rate Shifts

When bumping a single point on a curve (Key-Rate DV01), you must define how the rest of the curve behaves. Tuckman's standard approach uses a **triangular bump**:

- 0bp shift at adjacent key rates
- +1bp shift at the target key rate
- Linear interpolation between

> **Tuckman's Key-Rate Shift Shape**
>
> ```
> Change in Rate (bp)
>     │
>   1 │           ╱╲
>     │          ╱  ╲
>     │         ╱    ╲
>     │        ╱      ╲
>   0 │───────╱────────╲───────
>     │     2y   5y    10y    Maturity
> ```
>
> For a 5y key-rate bump with 2y and 10y as neighbors:
> - 0bp at 2y
> - +1bp at 5y
> - 0bp at 10y
> - Linear interpolation between

Tuckman explains the rationale: "In this technique a set of key rates is assumed to describe the movements of the entire term structure. Put another way, the technique assumes that given the key rates any other rate may be determined."

The key insight is that **key-rate 01s sum to total DV01**: "Since the sum of the key rate shifts is a parallel shift in the par yield curve, the sums of the key rate 01s and durations closely match the DV01 and duration, respectively, under the assumption of a parallel shift."

> **Example 11.10: Key-Rate Decomposition**
>
> A 7-year bond has total DV01 = 0.055 (for a parallel shift). Key-rate analysis decomposes this:
>
> | Key Rate | Key-Rate 01 | % of Total |
> |----------|-------------|------------|
> | 2-year | 0.008 | 14.5% |
> | 5-year | 0.032 | 58.2% |
> | 10-year | 0.015 | 27.3% |
> | **Sum** | **0.055** | **100%** |
>
> **Interpretation:** The bond's risk is concentrated in the 5-year sector, with some exposure to 10-year moves. This tells you which curve movements will hurt (or help) the most.

Full treatment of key-rate analysis is in **Chapter 14**.

---

## 11.7 Practical Notes

### 11.7.1 Common Ambiguity Traps

**1. Sign conventions:** The Tuckman convention (PV change for 1bp *decline*) and Hull convention (1bp *increase*) differ by sign. Always verify before combining risk numbers.

> **Example: Sign Convention Translation**
>
> A trader says "The DV01 is −500."
>
> **Interpretation 1 (Hull convention):** The portfolio loses $500 for every 1bp increase in rates. This is a long rates position.
>
> **Interpretation 2 (Error in Tuckman):** The portfolio is short rates (gains when rates rise). Unusual but possible for payer swaps or floating-rate positions.
>
> **Ask:** "Is that for a 1bp up or 1bp down move?"

**2. Clean vs Dirty:** Risk systems typically compute sensitivities on **full price** (dirty), but traders quote **clean price**. Since accrued interest is time-dependent only (not rate-dependent): $d(\text{AI})/dy \approx 0$.

Therefore: **Clean DV01 ≈ Dirty DV01**. However, precise systems should use full/dirty price to avoid edge case errors near coupon dates.

**3. Compounding:** A "1bp bump" to a continuous rate is economically smaller than a "1bp bump" to an annual rate. Ensure consistency across all curve inputs and outputs.

| Compounding | 1bp in Decimal | Equivalent Annual |
|-------------|----------------|-------------------|
| Continuous | 0.0001 | ~0.00010001 |
| Semi-annual | 0.0001 | ~0.00010025 |
| Annual | 0.0001 | 0.0001 |

**4. Day count:** When bumping swap curves, be precise about whether the bump is to the rate expressed in the curve's native day count (e.g., ACT/360) or converted.

### 11.7.2 P&L Traps for the Unwary

> **Desk Reality: The Coupon Drop P&L**
>
> Your bond pays its coupon tomorrow. Today's clean price is 102.50, with accrued interest of 2.50 (dirty = 105.00). Tomorrow, after the coupon:
> - Clean price: 102.50 (unchanged, assuming no rate move)
> - Accrued interest: 0.00 (reset)
> - Dirty price: 102.50
>
> Your "risk system" shows the position value dropped from 105.00 to 102.50—a loss of $2.50 per 100 face. Your P&L report screams RED.
>
> **Reality:** You received the 2.50 coupon in cash. Total value is unchanged.
>
> **The trap:** If your DV01 is computed on dirty price but your P&L is tracked on clean price (or vice versa), coupon dates create massive "phantom" P&L that masks the real risk attribution.

> **Practitioner Note: Repo Effects on Effective DV01**
>
> Consider a leveraged bond position financed in repo. The total return comprises:
> - **Duration return:** DV01 × rate move
> - **Carry:** Coupon income minus repo financing cost
>
> If repo rates spike (as during funding stress), your carry goes negative. This doesn't change your DV01 directly, but it changes the *break-even rate move* needed for positive P&L.
>
> **Implication:** In a high-repo environment, even a DV01-hedged position bleeds carry. Traders say the position has negative "carry-adjusted DV01" or "financed duration."

### 11.7.3 The Sensitivity Spectrum

Not all instruments have equally well-defined DV01. Ambiguity increases with optionality:

| Instrument | DV01 Precision | Why |
|------------|---------------|-----|
| Zero-coupon bond | Exact | $\text{DV01} = T \times P / (1+y) / 10{,}000$ |
| Plain coupon bond | Precise | Well-defined price-yield relationship |
| Amortizing bond | Precise | Fixed cashflows, standard calculation |
| Callable bond | Model-dependent | DV01 changes as rates approach call threshold |
| MBS | Highly model-dependent | Prepayment model choice dominates DV01 |
| Exotic rate option | Extremely model-dependent | DV01 varies with vol surface, correlation |

**Rule:** The more path-dependent or optionality-laden the instrument, the more your "DV01" depends on modeling choices.

### 11.7.4 Verification Tests for DV01 Engines

When building or validating a DV01 engine, run these tests:

| Test | What to Check | Expected Result |
|------|---------------|-----------------|
| **Sign Check** | Long fixed-rate bond, Tuckman convention | DV01 > 0 |
| **Notional Scaling** | $100mm face vs $1mm face | Exactly 100× |
| **Additivity** | Portfolio DV01 vs sum of parts | Equal (under consistent bumps) |
| **Convergence** | 0.1bp bump vs 1bp bump | Very close for smooth functions |
| **Symmetry** | $(V_\downarrow - V_0)$ vs $(V_0 - V_\uparrow)$ | Nearly equal for low convexity |
| **Zero-Coupon Benchmark** | $T$-year zero DV01 | $= T \times P / (1+y) / 10{,}000$ |
| **Par Bond** | Par bond DV01 vs closed-form | Must match analytical formula |

---

## 11.8 Connection to Immunization

The concepts of DV01 and duration connect directly to **immunization**—a strategy that protects portfolio value against interest rate changes.

### 11.8.1 The Immunization Principle

Luenberger describes immunization as a procedure that "'immunizes' the portfolio value against interest rate changes. The procedure...is in fact one of the most (if not the most) widely used analytical techniques of investment science, shaping portfolios consisting of billions of dollars of fixed-income securities held by pension funds, insurance companies, and other financial institutions."

The basic principle: **match the duration of assets to the duration of liabilities**. If both have the same duration, a parallel shift in rates affects both equally—the funded status is preserved.

### 11.8.2 The DV01 Matching Condition

For a pension fund with a liability $L$ and assets $A$:

**Basic immunization condition:**

$$\text{DV01}_A = \text{DV01}_L$$

This ensures that for small parallel rate moves:

$$\Delta A \approx \Delta L$$

**Enhanced immunization (with convexity):**

Luenberger notes: "Convexity can be used to improve immunization in the sense that, compared to ordinary immunization, a closer match of asset portfolio value and obligation value is maintained as yields vary."

For tighter protection:

$$\text{DV01}_A = \text{DV01}_L \quad \text{and} \quad \text{Convexity}_A \geq \text{Convexity}_L$$

With higher asset convexity, parallel shifts cause the assets to outperform (or at least match) liabilities.

### 11.8.3 Limitations of DV01-Based Immunization

Immunization via DV01 matching assumes parallel shifts. In practice:

1. **Curve twist:** Long and short rates move differently
2. **Spread changes:** Credit spreads on assets don't match liability discount rates
3. **Liability uncertainty:** Cash outflows may depend on behavior (lapse, mortality)

These limitations motivate the key-rate and multi-factor approaches in Chapters 14-16.

> **Desk Reality: The ALM Officer's Dilemma**
>
> An insurance company has liabilities with duration 15 years. Investment-grade corporate bonds have typical duration 7-8 years. To DV01-match:
>
> - **Option 1:** Buy 2× the notional in corporates (leverage)
> - **Option 2:** Use interest rate swaps to extend duration
> - **Option 3:** Buy long-dated zeros or strips (limited supply)
>
> Each choice has different risks, costs, and regulatory implications. DV01 matching is the starting point, not the final answer.

---

## 11.9 The Hedge Ratio Formula

The most immediate application of DV01 is computing hedge ratios—how much of one instrument to trade to offset the risk of another.

### 11.9.1 The Basic Formula

Tuckman derives the hedge ratio by equating the DV01 of the position to be hedged with the DV01 of the hedge:

$$F_{\text{hedge}} \times \frac{\text{DV01}_{\text{hedge}}}{100} = F_{\text{position}} \times \frac{\text{DV01}_{\text{position}}}{100}$$

Solving for the hedge face amount:

$$\boxed{F_{\text{hedge}} = \frac{\text{DV01}_{\text{position}}}{\text{DV01}_{\text{hedge}}} \times F_{\text{position}}}$$

> **Example 11.11: DV01 Hedge Calculation**
>
> You own $10 million face of Bond A with DV01 = 0.065 per 100 face. You want to hedge using Bond B with DV01 = 0.045 per 100 face.
>
> **Step 1: Calculate position DV01**
> $$\text{DV01}_A = 0.065 \times \frac{10{,}000{,}000}{100} = \$6{,}500$$
>
> **Step 2: Calculate hedge ratio**
> $$\text{Hedge Ratio} = \frac{0.065}{0.045} = 1.444$$
>
> **Step 3: Calculate hedge amount**
> $$F_B = 1.444 \times \$10{,}000{,}000 = \$14{,}440{,}000$$
>
> **You should short $14.44 million face of Bond B** to neutralize the DV01 of Bond A.
>
> **Verification:** DV01 of hedge = $0.045 \times 144{,}400 = \$6{,}500 = DV01$ of position. ✓

### 11.9.2 When One DV01 is Negative

Tuckman notes: "There are occasions in which one DV01 is negative. In these cases equation (5.7) shows that a hedged position consists of simultaneous longs or shorts in both securities."

For example, hedging a payer swap (negative DV01) with a long bond (positive DV01) requires going long both—counterintuitive until you trace through the signs.

### 11.9.3 Hedge Stability and Rebalancing

A DV01-neutral hedge is not "set and forget." As rates move:

1. **DV01s change:** Duration shortens as bonds approach maturity
2. **Convexity effects:** The hedge may drift off-neutral
3. **Cash flows:** Coupons reduce the hedged position size

For active hedging, traders rebalance periodically—daily for large books, less frequently for smaller positions.

---

## Summary

1. **DV01** is the change in dollar value for a 1 basis point change in rates. It requires a specific definition of *which* rate changes.

2. **Yield DV01** (bumping YTM) is standard for bonds but fails to capture curve shape risk. Tuckman: "this hedge will work as intended only if the yield of the bond bought changes by the same amount" as the hedge.

3. **Curve DV01** (bumping the zero curve) is standard for portfolios and swaps. It differs from Yield DV01 because yield is a weighted average of spot rates.

4. **PVBP** typically refers to the sensitivity of a swap to its *fixed rate* (an instrument feature), distinct from its sensitivity to the *discount curve* (a market feature).

5. **Multi-curve frameworks** split risk into Discount (OIS) and Projection (term rate) components. For near-par swaps, projection risk dominates.

6. **Bump-and-Reprice** with central differences $(V_\downarrow - V_\uparrow) / 2$ is the robust numerical method, eliminating convexity bias.

7. **Portfolio Additivity** only holds if all instruments are bumped by the same defined market shift.

8. **Quote PV01** (Par-Point Delta) provides hedge "recipes"—direct mappings from risk to hedge amounts in liquid instruments.

9. **Key-rate 01s** decompose total DV01 into sector-specific exposures; they sum to the parallel-shift DV01.

10. **Immunization** uses DV01/duration matching to protect portfolio value against rate changes, with limitations for non-parallel moves.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Yield DV01** | Sensitivity to own YTM ($dP/dy$) | "Trader's DV01" for bonds; assumes parallel yield shift across securities |
| **Curve DV01** | Sensitivity to discount curve | "Risk Manager's DV01"; additive across portfolios under consistent bump |
| **PVBP** | Sensitivity to contractual fixed rate | Used for pricing new swaps; equals $N \times A \times 0.0001$ |
| **Quote PV01** | Sensitivity to market input quotes | Recipes for hedging with liquid instruments |
| **Key-Rate 01** | Sensitivity to one curve sector | Decomposes risk by maturity; sums to total DV01 |
| **Central Difference** | $(V_\downarrow - V_\uparrow)/2$ | Removes convexity bias from first-order risk estimates |
| **Annuity Factor** | $\sum \tau_i P(0,T_i)$ | Present value of receiving 1 unit at each payment date |
| **Hedge Ratio** | $\text{DV01}_A / \text{DV01}_B$ | Determines how much of B to trade to hedge A |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $\text{DV01}$ | Dollar value of one basis point |
| $P(y)$ | Price as function of yield |
| $D_{\text{Mod}}$ | Modified duration |
| $A$ | Annuity factor for swaps |
| $S$ | Par swap rate |
| $K$ | Contractual fixed rate |
| $P(0,T)$ | Discount factor from time 0 to $T$ |
| $z(T)$ | Zero/spot rate to maturity $T$ |
| $\tau_i$ | Year fraction for accrual period $i$ |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is Tuckman's sign convention for DV01? | DV01 is positive for a long bond (PV change for 1bp **fall** in rates). |
| 2 | What is Hull's sign convention for DV01? | DV01 measures change for a 1bp **increase** in rates (opposite to Tuckman). |
| 3 | What is the difference between Yield DV01 and Curve DV01? | Yield DV01 bumps a single scalar (bond's YTM); Curve DV01 bumps the entire term structure of zero rates. |
| 4 | Formula for DV01 using central finite differences? | $\text{DV01} = (V(y-1\text{bp}) - V(y+1\text{bp})) / 2$ |
| 5 | If Portfolio A has yield-based DV01 and Portfolio B has curve-based DV01, can you add them? | No, they represent sensitivities to different stress scenarios and are not comparable. |
| 6 | What is the swap annuity factor $A$? | The present value of 1 unit paid on each fixed-leg date: $\sum \tau_i P(0,T_i)$. |
| 7 | What does "PVBP" typically mean for a swap? | Sensitivity to the fixed coupon rate $K$ (equals $N \times A \times 0.0001$), not the discount curve. |
| 8 | In a multi-curve swap (Receive Fixed), which curve usually drives the risk? | The Projection curve (drives floating payments), though Discount curve affects PV factors. |
| 9 | Why use central differences instead of one-sided bumps? | To eliminate second-order (convexity) errors for better first-derivative accuracy. |
| 10 | What is the unit consistency check for DV01? | If prices are in dollars, DV01 is in dollars per bp. If price is per 100, DV01 is per 100 per bp. |
| 11 | Why is Yield DV01 poor for hedging curve twists? | It assumes a rigid shift in the bond's average rate, ignoring how different segments of the curve might move differently. |
| 12 | What is Quote PV01 (Par-Point Delta)? | Sensitivity to bumping a single input quote (like 5y swap rate) and rebuilding the curve. |
| 13 | How does Tuckman define "key rate 01"? | The change in value when only one key rate shifts by 1bp (triangular bump with neighbors fixed). |
| 14 | What is the relationship between DV01 and Modified Duration? | $\text{DV01} = P \times D_{\text{Mod}} / 10{,}000$ |
| 15 | For a zero-coupon bond, what is the DV01 formula? | $\text{DV01} = T \times P / (1+y) / 10{,}000$ where $P$ is price and $T$ is maturity. |
| 16 | Why does bumping rates vs bumping discount factors give different results? | The relationship $DF = 1/(1+r)^T$ is nonlinear, so linear DF bumps don't match exact rate bumps. |
| 17 | What is the hedge ratio formula? | $F_{\text{hedge}} = (\text{DV01}_{\text{position}} / \text{DV01}_{\text{hedge}}) \times F_{\text{position}}$ |
| 18 | Why is OIS PV01 small for at-par swaps? | The swap's net PV is near zero, so discounting effects largely cancel between the two legs. |
| 19 | What does immunization via DV01 matching achieve? | It protects portfolio value against small parallel rate shifts by matching asset and liability durations. |
| 20 | Why do key-rate 01s sum to total DV01? | Because the sum of triangular key-rate shifts equals a parallel shift. |

---

## Mini Problem Set

### Questions

**1. Basic DV01 Calculation**
A bond priced at 98.50 increases to 98.55 if yields fall 1bp. It falls to 98.45 if yields rise 1bp. Calculate the DV01 per 100 face using Tuckman's convention.

**2. Dollar Risk Scaling**
You own $10 million face value of the bond in Q1. What is the total dollar DV01?

**3. Hedge Ratio**
You are long $5mm face of Bond A (DV01 = 0.072 per 100). You want to hedge with Bond B (DV01 = 0.048 per 100). What face amount of Bond B should you short?

**4. Par Swap PVBP**
A 5-year par swap has rate 4.00%. The Annuity Factor is 4.50. What is the PVBP for $100mm notional?

**5. DV01-Duration Conversion**
A bond priced at 92.00 has modified duration 6.5. What is its DV01 per 100 face?

**6. Curve vs Yield Intuition**
Explain why bumping the zero curve by +1bp might change a bond's yield by 0.98bp rather than 1.00bp.

**7. Convexity Error**
If you only bumped UP by 1bp (one-sided), why might your DV01 estimate be biased?

**8. Multi-Curve Intuition**
In a Receive-Fixed swap that starts at par, why is Discount Curve risk (OIS PV01) usually small?

**9. Sign Convention Translation**
A trader says "The DV01 is -500." They use the Hull convention. Translate this to Tuckman convention and explain what the position is.

**10. Key-Rate Decomposition**
A 10-year bond has the following key-rate 01s: 2y = 0.005, 5y = 0.025, 10y = 0.035, 30y = 0.003. What is the total DV01 for a parallel shift? Which sector has the most risk?

**11. Quote PV01 Hedge**
A bond has Quote PV01 of $3,200 to the 5y swap rate. The 5y swap has DV01 of $480 per $1mm notional. How much 5y swap notional should you trade to hedge the 5y exposure?

**12. Immunization Setup**
A pension fund has liabilities with DV01 of $125,000. Available bonds have DV01 of 0.065 per 100 face. What face amount of bonds is needed to immunize?

---

### Solutions (Brief)

**1.** Using central differences: $\text{DV01} = (98.55 - 98.45)/2 = 0.05$ per 100 face.

**2.** $0.05 \times (10,000,000 / 100) = \$5,000$ per bp.

**3.** Hedge ratio = $0.072 / 0.048 = 1.5$. Short $1.5 \times \$5mm = \$7.5mm$ face of Bond B.

**4.** $\text{PVBP} = \$100,000,000 \times 4.50 \times 0.0001 = \$45,000$ per bp.

**5.** $\text{DV01} = 92.00 \times 6.5 / 10,000 = 0.0598$ per 100 face.

**6.** The bond's price is a weighted sum of discounted cashflows. A parallel shift in zeros affects near and far cashflows differently. The single "Yield" that explains the new price is a complex weighted average that doesn't move 1-to-1 with the curve shift.

**7.** Bond prices are convex. The price loss for +1bp is smaller than the price gain for -1bp. A one-sided up bump underestimates the average sensitivity.

**8.** The swap starts with PV ≈ 0. The discount factor sensitivity scales with the net PV. If PV is small, the "discounting effect" is minor.

**9.** Under Hull convention, DV01 = -500 means the portfolio loses $500 when rates rise 1bp. Under Tuckman convention (1bp fall), this is DV01 = +500. The position is long rates (benefits from rate declines).

**10.** Total DV01 = $0.005 + 0.025 + 0.035 + 0.003 = 0.068$. The 10-year sector has the most risk (51% of total).

**11.** Hedge notional = $\$3,200 / \$480 \times \$1mm = \$6.67mm$ of 5y swap (short if you want to offset).

**12.** Required face = $\$125,000 / (0.065 / 100) = \$125,000 / 0.00065 = \$192,307,692 \approx \$192.3mm$.

---

## Source Map

### (A) Book-Verified Facts

| Fact | Source |
|------|--------|
| DV01 definition: "change in value for a one-basis point decline in rates" | Tuckman Ch 5 |
| DV01 formula: $-\Delta P / (10,000 \times \Delta y)$ | Tuckman Ch 5, Eq 5.1 |
| "The negative sign defines DV01 to be positive if price increases when rates decline" | Tuckman Ch 5 |
| Yield-based DV01 is special case where yield is the rate factor | Tuckman Ch 6 |
| "DV01 of a portfolio is the sum of the individual DV01 values" | Tuckman Ch 5 |
| Explicit DV01 formula with time-weighted PVs | Tuckman Ch 6, Eq 6.5 |
| "This hedge will work...only if the yield of the bond bought changes by the same amount" | Tuckman Ch 6 |
| "The most stable numerical estimate chooses rates that are equally spaced above and below" | Tuckman Ch 5 |
| Swap Annuity Factor definition $A = \sum \tau_n P(t, T_{n+1})$ | Andersen & Piterbarg Vol 1, Ch 4-5 |
| "The quantity $A(\cdot)$ is the annuity of the swap (or its PVBP)" | Andersen & Piterbarg Vol 1 |
| Par-point approach for curve risk | Andersen Vol 1 Ch 6, Section 6.4.1 |
| "The Jacobian method for interest rate deltas" | Andersen Vol 1 Ch 6, Section 6.4.3 |
| Key rate shifts technique | Tuckman Ch 7, pp 133-142 |
| "Key rate exposures essentially decompose a sensitivity measure like DV01" | Tuckman Ch 7 |
| Hedge ratio derivation | Tuckman Ch 5, Eq 5.7 |
| Dollar duration: $D_{\$} = -dB/dy$ | Hull RM Ch 9 |
| "Dollar duration is similar to the delta measure" | Hull RM Ch 9 |
| Partial durations for nonparallel shifts | Hull RM Ch 9, Section 9.6 |
| "The sum of all the partial duration measures equals the usual duration measure" | Hull RM Ch 9 |
| Immunization as protection against interest rate changes | Luenberger Ch 3, Section 3.6 |
| "Immunization...is one of the most widely used analytical techniques of investment science" | Luenberger Ch 3 |
| Convexity improvement to immunization | Luenberger Ch 3, Section 3.7 |
| Duration as "sensitivity of value...to a small parallel shift in the zero-coupon yield curve" | Hull RM Ch 9 |
| Hull's DV01 convention as "price change from a 1-basis-point increase" | Hull OFD Glossary |

### (B) Claude-Extended Content

| Content | Basis |
|---------|-------|
| "Tower of Babel Problem" analogy in introduction | Extended from Tuckman's emphasis on bump definition specificity |
| "Desk Reality: The Basis Trap" example | Extended from Tuckman's warning about yield vs curve DV01 mismatch |
| "Desk Reality: When OIS Risk Bites" funding squeeze scenario | Extended from Andersen's multi-curve framework; practical desk knowledge |
| "Desk Reality: The Coupon Drop P&L" example | Practitioner knowledge about clean/dirty price accounting |
| "Practitioner Note: Repo Effects on Effective DV01" | Practitioner knowledge about carry interaction |
| "Desk Reality: Quote PV01 as Hedging Recipe" explanation | Extended from Andersen's par-point approach |
| "Sensitivity Spectrum" ranking of instrument ambiguity | Inferred from model-dependency discussion in sources |
| Multi-curve table showing OIS vs Projection sensitivity by swap MTM | Extended from Andersen multi-curve framework |

### (C) Reasoned Inference

- The divergence between Yield and Curve DV01 (Section 11.3.3) is derived from the mathematical fact that yield is a complex weighted average of spot rates; bumping one changes the other non-linearly.
- Multi-curve decomposition logic follows directly from the definition of decoupled discount/projection curves in post-crisis frameworks (Andersen Vol 1).
- Sign convention translation rule ($\text{DV01}_{\text{Tuckman}} = -\text{DV01}_{\text{Hull}}$) follows from comparing explicit definitions.
- The DV01-Duration mapping formula ($\text{DV01} = P \times D_{\text{Mod}} / 10,000$) is derived algebraically from the definitions of each term.

### (D) Flagged Uncertainties

- **Desk Naming Conventions:** "PV01," "PVBP," and related terms are used differently across firms and systems. Complete standardization is impossible to verify.
- **Exact numbers in multi-curve example (Section 11.4.4):** Illustrative; actual values depend on specific curve shapes and swap terms.
- **Repo "carry-adjusted DV01" terminology:** Exact terminology varies by desk; the concept is standard but naming is not universal.

---

*Last Updated: January 26, 2026*
