# Chapter 11: DV01/PV01 — Definitions, Computation, and "What's Being Bumped"

---

## Introduction

A portfolio manager reports that her bond position has a "DV01 of $50,000." A swaps trader says his book is "flat on PV01." A risk officer asks for "key-rate exposures by bucket." Each statement sounds precise—a dollar measure of interest rate risk per basis point. But hidden in these simple phrases lurks a critical question that, if misunderstood, can render all these numbers meaningless: *what exactly is being bumped?*

Consider the portfolio manager's $50,000 DV01. Does that number assume the bond's yield moves by 1 basis point? Or does it assume every zero rate on the discount curve shifts by 1bp? Perhaps it reflects bumping *par instrument quotes* (like swap rates) and rebuilding a curve? These are different economic scenarios, and they can produce materially different dollar sensitivities for the same position. A DV01/PV01 quote is only meaningful after you specify:
- **Bump object:** yield, zero curve, par quotes, key rates, etc.
- **Bump size:** typically \(1\text{bp}=10^{-4}\) in decimal rate units.
- **Units + sign:** “$ per 1bp” and for what notional (per 100 face, per $1mm, etc.), and whether DV01 is defined for rates **down** or **up**.

The consequences of ambiguity are real—and expensive. A trader who hedges using yield-based DV01 but marks-to-market against a curve-based pricing engine will experience unexplained P&L when the curve twists. A risk system that aggregates DV01s computed under inconsistent bump definitions will produce portfolio sensitivities that don't add up. And a hedger who matches total DV01 without understanding key-rate exposures may find the hedge worthless when the curve steepens instead of shifting in parallel.

> **Desk Reality:** Different teams compute “DV01” under different bump definitions (yield vs curve vs par-quote rebuild).
> **Common break:** Aggregated DV01s don’t add up, and hedges look “neutral” but still generate unexplained P&L.
> **What to check:** Require every DV01/PV01 number to carry its bump object + sign convention, then validate by repricing the whole book under the same shock.

This chapter provides a rigorous foundation for interest rate sensitivity measures. We define DV01 formally in **Section 11.1**, emphasizing the sign convention and the requirement for specificity. **Section 11.2** covers **yield-based DV01**—the trader's measure for individual bonds—while **Section 11.3** develops **curve-based DV01** and explains why it differs. **Section 11.4** introduces **PVBP** for swaps and the critical multi-curve decomposition. **Section 11.5** details the computational mechanics of bump-and-reprice. **Section 11.6** presents a complete taxonomy of bump specifications—from parallel shifts to key-rate perturbations—answering definitively: "What is being bumped?" **Section 11.7** covers practical pitfalls and verification tests. **Section 11.8** connects DV01 to immunization concepts. Finally, **Section 11.9** gives the basic DV01 hedge-ratio idea and practical caveats.

Prerequisites: [Chapter 2](chapters/chapter_02_time_value_discount_factors_replication.md), [Chapter 5](chapters/chapter_05_fixed_rate_bond_pricing.md), [Chapter 6](chapters/chapter_06_ytm_yield_based_risk.md)  
Follow-on: [Chapter 12](chapters/chapter_12_duration_macaulay_modified_dv01.md), [Chapter 13](chapters/chapter_13_convexity.md), [Chapter 14](chapters/chapter_14_key_rate_dv01_bucket_exposures.md), [Chapter 15](chapters/chapter_15_dv01_hedging.md)

---

## Learning Objectives
- Translate a DV01/PV01 quote into a precise bump definition (what is bumped, by how much, with what sign).
- Compute a bond’s yield-based DV01 via bump-and-reprice and scale it to a position DV01 in dollars.
- Compute a cashflow stream’s curve-based DV01 by bumping a curve object (e.g., a zero curve).
- Interpret swap PVBP/PV01 as the PV of a 1bp fixed-rate annuity and distinguish it from curve PV01.
- Build simple hedge ratios and sanity-check risk reports for unit/sign mistakes.

---

## 11.1 DV01: The Dollar Value of a Basis Point

### 11.1.1 A Definition You Can Implement

In practice, a number of different approaches are used to calculate interest rate deltas.

One approach is to define delta as the **dollar duration**, the sensitivity of the portfolio to a parallel shift in the zero-coupon yield curve. A measure related to this definition of delta is **DV01**, the impact of a one-basis-point increase in all rates. Under this definition, DV01 is the dollar duration multiplied by \(0.0001\). Alternatively, it is the duration of the portfolio multiplied by the value of the portfolio multiplied by \(0.0001\).

It is important to emphasize that DV01 depends on the rate shift definition.

- **Yield DV01 (single-bond risk):** the dollar price change for a 1bp change in the bond’s yield-to-maturity \(y\).
- **Curve DV01 (portfolio/derivatives risk):** DV01 computed under a curve shift \(b\) (e.g., a parallel zero-curve shift), as developed in Section 11.3 and Section 11.6.

**House convention (used in this book):**

$$\boxed{\text{DV01}_b := V(\text{rates down }1\text{bp under }b)-V(\text{base})}$$

where \(1\text{bp}=10^{-4}\) in decimal rate units and \(b\) names the bump object / rebuild rule.

Many sources instead define DV01 for a \(+1\text{bp}\) move and attach an explicit minus sign so that DV01 is positive for standard fixed-rate bonds; this is just a sign convention as long as you state it.

If a price–rate function \(P(y)\) is differentiable in a **scalar** rate \(y\) (e.g., bond YTM), then yield DV01 can be written as:

$$\boxed{\text{DV01} = -\frac{1}{10{,}000}\frac{dP}{dy}}$$

A common numerical approach is to compute **dollar duration** (and hence DV01) from the **average price change** when the chosen rate measure is increased and decreased by the same number of basis points. With a symmetric \(\pm 1\text{bp}\) bump:

$$\boxed{\text{DV01}_b \approx \frac{V_{\downarrow}-V_{\uparrow}}{2}}$$

where \(V_{\uparrow}\) and \(V_{\downarrow}\) are the PVs after applying a \(+1\text{bp}\) and \(-1\text{bp}\) bump under the *same* bump definition \(b\).

**Units**
- If \(P\) is “price per 100 face”, then DV01 is “price points per 100 per 1bp”.
- Position DV01 (currency per 1bp) is \(\text{DV01}_{\text{per 100}}\times \frac{\text{Face}}{100}\).

**Default conventions for examples (unless stated otherwise)**
- Settlement on a coupon date so \(AI=0\) and \(P_{\text{clean}}=P_{\text{dirty}}\).
- Bond YTM uses semiannual compounding; curve examples state compounding locally.
- Cashflows/PV are signed from the holder’s perspective (positive = receive).

> **Pitfall — DV01 sign + bump mismatch:** Two systems can both report “DV01” but mean different shocks (rates up vs down; yield vs curve vs par-quote rebuild).
> **Why it matters:** Hedge ratios flip sign, and portfolio DV01s stop being additive in any meaningful economic sense.
> **Quick check:** Ask: (i) what is bumped, (ii) is it \(+1\text{bp}\) or \(-1\text{bp}\), and (iii) does repricing under that shock move PV by approximately the reported DV01?

### 11.1.2 What DV01 Answers

DV01 answers a simple question: *"If rates move by 1bp (in the way I've defined), how many dollars do I gain or lose?"*

This is a **first-order Taylor approximation** of the price change. For a small rate move $\Delta y$, the price change is approximately:

$$\Delta P \approx -10,000 \times \text{DV01} \times \Delta y$$

A closely related object is **dollar duration** (for a yield-based bump):

$$D_{\$}:=-\frac{dP}{dy}$$

Under the “rates down” DV01 convention used in this chapter, \(D_{\$}=10{,}000\times \text{DV01}\).

> **The Einstein Formula of Fixed Income**
>
> We can rewrite risk in percentage terms to get the most important equation in bond math:
>
> $$\frac{dP}{P} \approx -D \times dy$$
>
> where $D$ is modified duration. This says: **Change in Price (%)** equals **Duration** times **Change in Yield**. DV01 is just the *dollar version* of this: $\text{DV01} = P \times D_{\text{Mod}} / 10{,}000$.

Crucially, the DV01 definition requires specifying the "rate measure" $y$ being bumped. While DV01 can be computed "for any interest rate measure," market usage typically implies **yield-based DV01** for individual bonds and **curve-based DV01** for portfolios and derivatives.

### 11.1.3 Portfolio Aggregation and Additivity

DV01 is additive when it is computed under a **consistent bump definition**. If portfolio value is a sum of position values, \(V_{\text{port}}=\sum_i V_i\), then under the same bump definition \(b\):

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

Here \(P\) is the bond’s price in the same units you want DV01 to be reported in (often **dirty price per 100 face**).

> **Example 11.2: Worked DV01 (Yield Bump) for a 2-Year Bond**
>
> **Context**
> - You want a position DV01 for a bond quoted and risk-managed in yield terms.
>
> **Timeline (make dates concrete)**
> - Trade date: 2026-02-17
> - Settlement date: 2026-02-19 (assume settlement on a coupon date so \(AI=0\))
> - Payment dates: 2026-08-19, 2027-02-19, 2027-08-19, 2028-02-19 (maturity)
>
> **Inputs**
> - Face \(N=\$25{,}000{,}000\)
> - Coupon \(c=4.00\\%\\) per year, paid semiannually (2.00 per 100 every 6 months)
> - Yield \(y=4.50\\%\\) (bond-style YTM with semiannual compounding)
>
> **Outputs**
> - Dirty price per 100 (here \(AI=0\), so clean = dirty): \(P_0 \\approx 99.0538\)
> - DV01 per 100 (rates **down** 1bp in yield): \(\text{DV01} \\approx 0.01881\)
> - Position DV01: \(\text{DV01}_{\\$} \\approx 0.01881\\times\\frac{25{,}000{,}000}{100} \\approx \\$4{,}702\\) per 1bp
>
> **Step-by-step**
> 1. Price the bond at \(y\): \(P_0=\\sum_{t=1}^{4} \\frac{CF_t}{(1+y/2)^t}\\).
> 2. Reprice at \(y\\pm 1\\text{bp}\): \(P_{\\uparrow}=P(y+10^{-4})\\), \(P_{\\downarrow}=P(y-10^{-4})\\).
> 3. Central-difference DV01: \(\text{DV01}=(P_{\\downarrow}-P_{\\uparrow})/2\\).
> 4. Scale to position: \(\text{DV01}_{\\$}=\\text{DV01}_{\\text{per 100}}\\times N/100\\).
>
> **Cashflows (per 100 face)**
>
> | Date | Cashflow | Explanation |
> |---|---:|---|
> | 2026-08-19 | 2.00 | coupon |
> | 2027-02-19 | 2.00 | coupon |
> | 2027-08-19 | 2.00 | coupon |
> | 2028-02-19 | 102.00 | final coupon + principal |
>
> **P&L / risk interpretation**
> - DV01 \(\approx \\$4{,}702/\\text{bp}\) means a 10bp rally in yield (all else equal) is about \(+\\$47k\) of price P&L.
> - A yield-DV01 hedge assumes the two yields you are matching will actually move together.
>
> **Sanity checks**
> - Units: “per 100” \(\times\) “face/100” \(\rightarrow\) dollars per bp.
> - Sign: a lower yield raises price, so DV01 \(>0\) for a long fixed-rate bond under this convention.

---

## 11.2 Yield-Based DV01: Bumping a Single Scalar

### 11.2.1 The Yield-Based Framework

When bond traders speak of "DV01," they often mean **yield-based DV01**: sensitivity to the bond’s own yield-to-maturity (YTM), treated as a single scalar risk factor.

Recall from Chapter 6 that YTM is the single discount rate that equates cashflows to the price. For a bond with coupon $c$ per 100 face, maturity $T$ years, and semiannual compounding:

$$P(y) = \sum_{t=1}^{2T} \frac{c/2}{(1+y/2)^t} + \frac{100}{(1+y/2)^{2T}}$$

Yield-based DV01 is computed by bumping this single scalar $y$. Using central differences:

$$\text{DV01}_{\text{yield}} = \frac{P(y - 1\text{bp}) - P(y + 1\text{bp})}{2}$$

The factor of 2 in the denominator (rather than the $10,000 \times \Delta y$ in the general formula) appears because we're using a 2bp total shift (±1bp from the base) and expressing the result in "per bp" terms.

### 11.2.2 Explicit Formula for Yield-Based DV01

Differentiating the bond price equation yields an explicit formula:

$$\boxed{\text{DV01} = \frac{1}{10,000} \times \frac{1}{1+y/2}\left[\sum_{t=1}^{2T} \frac{t}{2} \cdot \frac{c/2}{(1+y/2)^t} + T \cdot \frac{100}{(1+y/2)^{2T}}\right]}$$

Interpretation: DV01 is proportional to a **time-weighted present value** of the bond’s cashflows. Longer-dated cashflows contribute more to interest-rate sensitivity, but they are also discounted more heavily.

The term in brackets is the sum of (time × PV) for each cashflow—this is precisely the numerator of the Macaulay duration formula, which explains the tight relationship between DV01 and duration.

### 11.2.3 Worked Example: Computing DV01 from First Principles

One transparent way to compute yield DV01 is to build a cashflow table, compute PVs, and then compute the time-weighted PV sum that drives modified duration (and therefore DV01).

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
> DV01 then follows from the time-weighted PV sum:
>
> $$\text{DV01} = \frac{1}{10,000} \times \frac{454.8511}{1 + 0.050441/2} = \frac{1}{10,000} \times \frac{454.8511}{1.0252} = \mathbf{0.044366}$$
>
> **Interpretation:** A one-basis point decline in the bond's yield increases its price by about 4.44 cents per $100 face value.
>
> **Sanity Check:** For a $100 million position, DV01 = $0.044366 × 1,000,000 = $44,366 per bp. A 10bp rally gains approximately $443,660.

### 11.2.4 The Limitation: Parallel Yield Shifts

Yield-based DV01 is a **parallel-shift assumption**: it treats the bond’s yield as the risk factor. If you hedge a position using yield-based DV01, the hedge works as intended only if the yields of the hedged bond and the hedging instrument actually move together.

In practice yields can diverge due to:

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

**Mechanics (discount-factor view):** A parallel +1bp shift in continuously compounded zero rates multiplies every discount factor by the same maturity-dependent factor:
\[
P_{\text{shifted}}(0,t)=P(0,t)\,e^{-0.0001\,t}.
\]
Longer-dated cashflows are hit more because the exponential factor depends on \(t\).

**Check (duration-style scaling):** For small bumps, \(e^{-0.0001 t}\approx 1-0.0001 t\), so the PV change is approximately
\[
\Delta V \approx -0.0001\sum_i t_i\,CF_i\,P(0,t_i),
\]
which is why curve DV01 is closely related to a PV-weighted average maturity. As a rough desk check: if \(V\approx 100\) (per 100 face) and the PV-weighted time is about 4 years, then a 1bp parallel shift should move value by roughly \(100\times 4/10{,}000 \approx 0.04\) price points per 100.

### 11.3.2 Worked Example: Curve DV01 via Zero-Curve Bump

> **Example 11.4: Curve DV01 via a Parallel Zero-Curve Bump**
>
> Consider a 5-year bond with 5% annual coupon (cashflows: 5 at years 1–4, and 105 at year 5). Price it off a flat **continuously compounded** zero curve at \(z=4.50\\%\\), so \(P(0,t)=e^{-z t}\).
>
> **Base PV**
> $$P_0=\\sum_{t=1}^{4} 5e^{-0.045t}+105e^{-0.045\\cdot 5}\\approx 101.7388$$
>
> **Bump the curve up and down by 1bp**
> $$P_{\\uparrow}\\approx 101.6925 \\quad (z=4.51\\%)$$
> $$P_{\\downarrow}\\approx 101.7851 \\quad (z=4.49\\%)$$
>
> **Curve DV01 (rates down convention)**
> $$\\text{DV01}_{\\text{curve}}=\\frac{P_{\\downarrow}-P_{\\uparrow}}{2}\\approx 0.0463\\ \\text{per 100 face}$$

### 11.3.3 Why Yield DV01 and Curve DV01 Differ

The divergence between yield DV01 and curve DV01 is not a computational error—it reflects a fundamental conceptual difference.

**The yield is a weighted average of spot rates.** When we compute a bond's YTM, we're finding a single rate that, when applied to all cashflows, reproduces the bond's price. This rate is a complex weighted average of the spot rates that would actually apply to each cashflow.

**A 1bp change in yield ≠ a 1bp parallel shift in zeros.** When you bump the yield by 1bp, you're implicitly changing the "implied spot curve" that would price this specific bond. This implied curve pivots in a complex way that depends on the bond's cash flow structure. Conversely, when you shift all zeros by 1bp, the bond's yield changes by something other than exactly 1bp—typically close, but not identical.

If you hedge a curve-priced P&L using yield-based DV01, you should expect residual risk when the curve twists or steepens: you are hedging a **different bump object** than the one your pricing engine uses.

> **Desk Reality:** Yield DV01 is “one-number risk” for a bond, but it can mix **rate** and **spread/basis** effects.
> **Common break:** A hedge sized on yield DV01 looks neutral, yet P&L still moves when the curve twists or spreads move.
> **What to check:** Separate the rate bump you care about (curve DV01) from spread bumps (e.g., CS01) and confirm your hedge is aligned to the same pricing inputs.

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

For interest rate swaps, a key quantity is the **fixed-leg annuity factor** (often called the swap annuity, PVBP, or PV01 *per unit notional*):

$$\boxed{A(t) = \sum_{n} \tau_n P(t, T_{n+1})}$$

where $\tau_n$ is the year fraction for payment period $n$ and $P(t, T_{n+1})$ is the discount factor to payment date $T_{n+1}$.

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
> **Interpretation:** Holding the curve fixed, a 1bp increase in the fixed rate you receive (from 4.50% to 4.51%) adds $44,518 to the swap's value.
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

In a **multi-curve** pricing framework (common for collateralized swaps), we separate:
- **Discounting:** an overnight curve (e.g., OIS)
- **Projecting floating rates:** a term index curve (e.g., term SOFR or legacy IBOR-style curves)

This splits "PV01" into two components: **OIS PV01** (discount curve sensitivity) and **Projection PV01** (forecast curve sensitivity).

A practical way to report multi-curve risk is to compute “one-curve-at-a-time” PV01s:
- **Discount PV01:** bump the discounting curve by \(\pm 1\text{bp}\) (projection held fixed) and reprice.
- **Projection PV01:** bump the projection curve by \(\pm 1\text{bp}\) (discount held fixed) and reprice.

For swaps near par, discount-curve PV01 is often smaller because discounting affects both legs similarly and partly cancels, while the projection curve directly changes expected floating cashflows. As the swap’s MTM moves away from zero, discount-curve PV01 becomes more important.

> **Desk Reality:** Multi-curve PV01s show up as separate buckets on risk reports (discount vs projection).
> **Common break:** Hedging only “total PV01” can leave residual P&L when the two curves move differently.
> **What to check:** Confirm which curve(s) your hedges actually neutralize, and whether the book is exposed to discount–projection basis.

---

## 11.5 Bump-and-Reprice Mechanics

### 11.5.1 The Central Finite Difference Algorithm

A common approach for computing DV01 numerically is **central finite differences**:

1. **Base:** Price at current rates → $V_0$
2. **Up:** Price at rates $+1\text{bp}$ → $V_{\uparrow}$
3. **Down:** Price at rates $-1\text{bp}$ → $V_{\downarrow}$
4. **Calculate:** $\text{DV01} = (V_{\downarrow} - V_{\uparrow}) / 2$

Central differences use symmetric bumps around the base point, reducing bias from curvature (convexity) relative to a one-sided bump.

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

> **Example 11.7: Central vs One-Sided Comparison**
>
> For a 10-year 5% bond at 4% yield (semiannual compounding; price \(\approx 108.175717\) per 100 face):
>
> | Bump Direction | New Rate | New Price | DV01 Estimate |
> |----------------|----------|-----------|---------------|
> | Base | 4.00% | 108.175717 | — |
> | Up (+1bp) | 4.01% | 108.090055 | 0.085661 (one-sided) |
> | Down (−1bp) | 3.99% | 108.261460 | 0.085743 (one-sided) |
> | **Central** | | | **(108.261460 − 108.090055) / 2 = 0.085702** |
>
> **Observation:** The (small) asymmetry between one-sided up and down estimates reflects positive convexity: the bond gains slightly more when rates fall than it loses when rates rise by the same amount.

**Check (bump size is part of the definition):** DV01 is a derivative concept implemented with finite differences. For highly convex or path-dependent instruments, the reported “DV01” can change depending on whether you use a 1bp, 0.5bp, or 5bp bump, and whether you use a central or one-sided scheme. For plain-vanilla bonds the dependence is usually tiny at 1bp, but you should still treat **bump size + bump scheme** as part of the risk number’s specification.

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
> **Method 1: Exact Rate Bump (central difference)**
> Bump $r$ to $5.01\%$ and $4.99\%$:
> - $DF_{\uparrow} = 1/(1.0501)^{10} \approx 0.6133$, so $V_{\uparrow} \approx 61.33$
> - $DF_{\downarrow} = 1/(1.0499)^{10} \approx 0.6145$, so $V_{\downarrow} \approx 61.45$
>
> Central DV01 \(\approx (V_{\downarrow}-V_{\uparrow})/2 \approx \mathbf{0.0585}\).
>
> **Method 2: Linear DF Approximation**
> Approximate $\Delta DF \approx -T \cdot DF \cdot \Delta r / (1+r) \approx -10 \times 0.6139 \times 0.0001 / 1.05 \approx -0.000585$.
> This gives \(V_{\uparrow}\approx 100(DF_0+\Delta DF)\), \(V_{\downarrow}\approx 100(DF_0-\Delta DF)\), so central DV01 \(\approx 100|\Delta DF| \approx \mathbf{0.0585}\).
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

**Mechanics (rebuild makes the bump non-local):** When you bump a par quote and rebuild the curve, you typically change *many* underlying curve nodes/discount factors—not just “the 5-year point.” So Quote PV01 is a sensitivity to a **market quote instrument** (something you can actually trade), not a pure key-rate sensitivity. That is why Quote PV01 is often a better hedging coordinate system, while key-rate DV01 is often a better exposure/interpretation coordinate system.

The operational appeal is that the sensitivity is reported in the same “coordinates” as liquid hedging instruments (the quotes you actually trade).

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
> This gives hedge amounts in each benchmark instrument: divide each bucket exposure by the hedge instrument’s own DV01, computed under the same bump definition.

> **Desk Reality:** Quote PV01 often functions as a “hedge recipe” because it is already expressed in the quotes/instruments you can trade.
> **Common break:** Confusing quote PV01 (bump one input quote + rebuild) with curve DV01 (bump the entire curve object) or PVBP (bump the contractual fixed rate).
> **What to check:** Confirm (i) which quote is bumped, (ii) whether and how the curve is rebuilt, and (iii) that the hedge instrument DV01 is computed under the same methodology.

### 11.6.3 The Jacobian Approach for Hedging

If you compute sensitivities in one set of risk factors (e.g., forward-rate bumps) but hedge with a set of instruments (e.g., benchmark swaps), you are implicitly doing a **linear mapping** between bases. In matrix form, this looks like solving a Jacobian system:

$$\mathbf{p} = \left(\partial \mathbf{H}^{\top}\right)^{-1} \partial \mathbf{V}_0$$

where $\mathbf{p}$ is the vector of hedge positions, $\partial \mathbf{H}$ is the Jacobian matrix of hedge instrument sensitivities, and $\partial \mathbf{V}_0$ is the vector of portfolio sensitivities.

For most desk purposes, Quote PV01 provides a simpler alternative that directly maps to hedge amounts. The Jacobian approach is useful when:
- Hedging with non-standard instruments
- Optimizing across hedge costs
- Handling instruments that span multiple curve points

### 11.6.4 Key-Rate Shifts

When bumping a single point on a curve (Key-Rate DV01), you must define how the rest of the curve behaves. A common approach uses a **triangular bump**:

- 0bp shift at adjacent key rates
- +1bp shift at the target key rate
- Linear interpolation between

> **Key-Rate Shift Shape (Triangular Bump)**
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

The key insight is that (under the same curve construction and bump design) **key-rate 01s approximately sum to the parallel-shift DV01**: the triangular bumps are designed so that adding them up resembles a parallel shift.

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

**1. Sign conventions:** Two common conventions differ by sign:
- **Rates-down DV01 (used in this book):** \(DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})\).
- **Rates-up DV01:** \(DV01^{(+)} := PV(\text{rates up }1\text{bp})-PV(\text{base})\).

For symmetric \(\pm 1\text{bp}\) bumps, \(DV01^{(+)} \approx -DV01\).

**Check (quick sign sanity):** Under the “rates down 1bp” convention in this book:
- A long option-free fixed-rate bond typically has \(DV01>0\) (rates down \(\Rightarrow\) price up).
- A receiver fixed swap (receive fixed, pay float) typically has \(DV01>0\); a payer swap typically has \(DV01<0\).
If your report shows the opposite sign, first check whether the system is using a “rates up” convention or reporting a loss-on-up number as “DV01” without the sign flip.

> **Example: Sign Convention Translation**
>
> A trader says "The DV01 is −500."
>
> **Interpretation (rates-up convention):** The portfolio loses $500 for every 1bp increase in rates.
>
> **Ask:** "Is that for a 1bp up or 1bp down move?"

**2. Clean vs dirty (and cash settlement):** A quoted clean price excludes accrued interest \(AI\). Dirty price is
\(P_{\text{dirty}}=P_{\text{clean}}+AI\), and the cash settlement amount for face \(N\) is \(\frac{N}{100}P_{\text{dirty}}\).

Accrued interest is time-dependent, so for yield/curve bumps it is usually treated as insensitive to rates. As a result, **clean DV01 \(\approx\) dirty DV01** when the bump only changes discounting/projection rates.

**3. Compounding/day count:** Apply the bump in the curve’s **native quote units** (day count + compounding). A “1bp bump” is not meaningful unless you also specify which rate representation is being bumped.

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
| **Sign Check** | Long fixed-rate bond, rates-down DV01 convention | DV01 > 0 |
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

Classical immunization is an asset–liability strategy: construct an asset portfolio so that (at the current curve) small **parallel** yield shifts have limited impact on funded status. In its simplest form this means matching the interest-rate sensitivity of assets and liabilities.

The basic principle: **match the duration of assets to the duration of liabilities**. If both have the same duration, a parallel shift in rates affects both equally—the funded status is preserved.

### 11.8.2 The DV01 Matching Condition

For a pension fund with a liability $L$ and assets $A$:

**Basic immunization condition:**

$$\text{DV01}_A = \text{DV01}_L$$

This ensures that for small parallel rate moves:

$$\Delta A \approx \Delta L$$

**Enhanced immunization (with convexity):**

Convexity can be used to improve immunization: if assets have at least as much convexity as liabilities (under the same shock definition), the match tends to be more robust as yields vary.

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

A basic DV01 hedge ratio comes from equating the DV01 of the position and the hedge (under the same bump definition):

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

Sometimes one DV01 is negative under your chosen convention (e.g., a payer swap can have negative DV01). In these cases, a DV01-neutral hedge can require going long (or short) both instruments—so always trace through the sign rather than relying on intuition.

### 11.9.3 Hedge Stability and Rebalancing

A DV01-neutral hedge is not "set and forget." As rates move:

1. **DV01s change:** Duration shortens as bonds approach maturity
2. **Convexity effects:** The hedge may drift off-neutral
3. **Cash flows:** Coupons reduce the hedged position size

For active hedging, traders rebalance periodically—daily for large books, less frequently for smaller positions.

---

## Summary

1. **DV01** is the change in dollar value for a 1 basis point change in rates. It requires a specific definition of *which* rate changes.

2. **Yield DV01** (bumping YTM) is standard for bonds but can miss curve-shape and basis/spread effects; a yield-DV01 hedge assumes the yields you are matching actually move together.

3. **Curve DV01** (bumping the zero curve) is standard for portfolios and swaps. It differs from Yield DV01 because yield is a weighted average of spot rates.

4. **PVBP** typically refers to the sensitivity of a swap to its *fixed rate* (an instrument feature), distinct from its sensitivity to the *discount curve* (a market feature).

5. **Multi-curve frameworks** split risk into Discount (OIS) and Projection (term rate) components. For near-par swaps, projection risk dominates.

6. **Bump-and-Reprice** with central differences $(V_\downarrow - V_\uparrow) / 2$ is the robust numerical method, eliminating convexity bias.

7. **Portfolio Additivity** only holds if all instruments are bumped by the same defined market shift.

8. **Quote PV01** (Par-Point Delta) provides hedge "recipes"—direct mappings from risk to hedge amounts in liquid instruments.

9. **Key-rate 01s** decompose total DV01 into sector-specific exposures; they sum to the parallel-shift DV01.

10. **Immunization** uses DV01/duration matching to protect portfolio value against rate changes, with limitations for non-parallel moves.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **DV01 (rates-down)** | \(DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})\) for a stated bump object | Fixes sign and makes DV01 comparable within a report |
| **DV01 (rates-up)** | \(DV01^{(+)} := PV(\text{rates up }1\text{bp})-PV(\text{base}) \approx -DV01\) | Many systems use this; translate before aggregating |
| **Yield DV01** | Bump a bond’s YTM \(y\) by \(\pm 1\text{bp}\) | Natural for bond trading, but assumes the yields you match move together |
| **Curve DV01** | Bump a curve object (e.g., all zero rates) by \(\pm 1\text{bp}\) | Consistent for portfolios/derivatives priced off a curve |
| **Dollar duration** | \(D_{\$}:=-dP/dy\) (yield-based) | Connects DV01 to duration: \(DV01=D_{\$}/10{,}000\) |
| **PVBP (swap “rate PV01”)** | PV change from bumping swap fixed rate \(K\) by 1bp (curve held fixed) | Used in swap pricing and risk; equals \(N\\times A\\times 10^{-4}\) |
| **Annuity factor \(A\)** | \(A=\\sum_i \\tau_i P(0,T_i)\) | Present value of 1 unit paid on fixed-leg dates |
| **Quote PV01 (par-point delta)** | Bump one input quote, rebuild curve, reprice | Maps sensitivities directly to hedging instruments |
| **Key-Rate 01** | Localized bump around a tenor (triangular bump) | Decomposes curve risk by maturity “sectors” |
| **Clean vs dirty** | \(P_{dirty}=P_{clean}+AI\); settle \(\frac{N}{100}P_{dirty}\) | Prevents settlement and coupon-date P&L errors |
| **Hedge ratio** | \(F_{hedge}=(DV01_{pos}/DV01_{hedge})\\times F_{pos}\) | Converts risk numbers into trade sizes |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| \(P_{\text{clean}}, P_{\text{dirty}}\) | clean/dirty price | price per 100 face; \(P_{\text{dirty}}=P_{\text{clean}}+AI\) |
| \(AI\) | accrued interest | price points per 100; time-dependent |
| \(y\) | bond YTM | per year; state compounding (examples use semiannual) |
| \(P(y)\) | bond price–yield function | price per 100 face |
| \(V\) | present value (PV) | currency; signed (positive = receive) |
| \(DV01\) | DV01 (rates-down) | currency per 1bp, or price points per 100 per 1bp |
| \(DV01^{(+)}\) | DV01 (rates-up) | \(\approx -DV01\) for symmetric bumps |
| \(P(0,T)\) | discount factor | unitless |
| \(z(T)\) | zero/spot rate | per year; compounding stated locally |
| \(\tau_i\) | accrual year fraction | years; depends on day count |
| \(A\) | swap annuity | years (unitless PV weight); \(A=\\sum \\tau_i P(0,T_i)\) |
| \(K, S\) | swap fixed rate / par swap rate | per year; same compounding/day count as the leg |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What DV01 sign convention does this book use? | Rates-down DV01: \(DV01 = PV(\text{rates down }1\text{bp})-PV(\text{base})\); typically positive for a long fixed-rate bond. |
| 2 | How do you translate a “rates-up DV01” into this book’s convention? | For symmetric bumps, \(DV01^{(+)} \\approx -DV01\). Always confirm the bump object. |
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
| 13 | How is a key-rate 01 defined? | The PV change from applying a localized 1bp bump around one tenor (often triangular with adjacent tenors fixed). |
| 14 | What is the relationship between DV01 and Modified Duration? | $\text{DV01} = P \times D_{\text{Mod}} / 10{,}000$ |
| 15 | For a zero-coupon bond, what is the DV01 formula? | $\text{DV01} = T \times P / (1+y) / 10{,}000$ where $P$ is price and $T$ is maturity. |
| 16 | Why does bumping rates vs bumping discount factors give different results? | The relationship $DF = 1/(1+r)^T$ is nonlinear, so linear DF bumps don't match exact rate bumps. |
| 17 | What is the hedge ratio formula? | $F_{\text{hedge}} = (\text{DV01}_{\text{position}} / \text{DV01}_{\text{hedge}}) \times F_{\text{position}}$ |
| 18 | Why is discount-curve PV01 often smaller for an at-par swap? | When swap PV \(\approx 0\), discounting changes affect both legs similarly and partly cancel; projection changes move expected floating cashflows more directly. |
| 19 | What does immunization via DV01 matching achieve? | It protects portfolio value against small parallel rate shifts by matching asset and liability durations. |
| 20 | Why do key-rate 01s sum to total DV01? | Because the sum of triangular key-rate shifts equals a parallel shift. |

---

## Mini Problem Set

### Questions

**1. Basic DV01 Calculation**
A bond priced at 98.50 increases to 98.55 if yields fall 1bp. It falls to 98.45 if yields rise 1bp. Calculate the DV01 per 100 face using the rates-down DV01 convention.

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
A trader says "The DV01 is -500." They use a rates-up DV01 convention. Translate this to the rates-down convention and explain what the position is.

**10. Key-Rate Decomposition**
A 10-year bond has the following key-rate 01s: 2y = 0.005, 5y = 0.025, 10y = 0.035, 30y = 0.003. What is the total DV01 for a parallel shift? Which sector has the most risk?

**11. Quote PV01 Hedge**
A bond has Quote PV01 of $3,200 to the 5y swap rate. The 5y swap has DV01 of $480 per $1mm notional. How much 5y swap notional should you trade to hedge the 5y exposure?

**12. Immunization Setup**
A pension fund has liabilities with DV01 of $125,000. Available bonds have DV01 of 0.065 per 100 face. What face amount of bonds is needed to immunize?

---

### Solution Sketches (Selected)

**1.** Using central differences: $\text{DV01} = (98.55 - 98.45)/2 = 0.05$ per 100 face.

**2.** $0.05 \times (10,000,000 / 100) = \$5,000$ per bp.

**3.** Hedge ratio = $0.072 / 0.048 = 1.5$. Short $1.5 \times \$5mm = \$7.5mm$ face of Bond B.

**4.** $\text{PVBP} = \$100,000,000 \times 4.50 \times 0.0001 = \$45,000$ per bp.

**5.** $\text{DV01} = 92.00 \times 6.5 / 10,000 = 0.0598$ per 100 face.

**6.** The bond's price is a weighted sum of discounted cashflows. A parallel shift in zeros affects near and far cashflows differently. The single "Yield" that explains the new price is a complex weighted average that doesn't move 1-to-1 with the curve shift.

**7.** Bond prices are convex. The price loss for +1bp is smaller than the price gain for -1bp. A one-sided up bump underestimates the average sensitivity.

**8.** The swap starts with PV ≈ 0. The discount factor sensitivity scales with the net PV. If PV is small, the "discounting effect" is minor.

**9.** Under a rates-up convention, DV01 = -500 means the portfolio loses $500 when rates rise 1bp. Under the rates-down convention, this is \(DV01 \\approx +500\). The position is long rates (benefits from rate declines).

**10.** Total DV01 = $0.005 + 0.025 + 0.035 + 0.003 = 0.068$. The 10-year sector has the most risk (51% of total).

**11.** Hedge notional = $\$3,200 / \$480 \times \$1mm = \$6.67mm$ of 5y swap (short if you want to offset).

**12.** Required face = $\$125,000 / (0.065 / 100) = \$125,000 / 0.00065 = \$192,307,692 \approx \$192.3mm$.

---

## References

- (Tuckman & Serrat, *Fixed Income Securities: Tools for Today's Markets*, “Yield-Based DV01”; “Key Rate 01s and Key Rate Durations”)
- (Neftci, *Principles of Financial Engineering*, “3.8.1 DV01 AND PV01”)
- (Hull, *Risk Management and Financial Institutions*, “9.7 Interest Rate Deltas in Practice”)
- (Pachamanova & Fabozzi, *Simulation and Optimization in Finance*, “Dollar Duration”; “Key Rate Duration”; “Classical immunization”)
- (O’Kane, *Modelling Single-name and Multi-name Credit Derivatives*, “4.2.7 The Bond DV01”; “The Discount Margin”)
- (Musiela & Rutkowski, *Martingale Methods in Financial Modelling*, Eq. 13.13 (swap annuity / PVBP))
