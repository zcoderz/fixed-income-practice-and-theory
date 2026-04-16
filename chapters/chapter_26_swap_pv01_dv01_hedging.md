# Chapter 26: Swap PV01, DV01, and Hedging with Swaps

---

## Introduction

A 10-year interest rate swap has roughly 10 years of duration—but what exactly are we measuring, and how does it compare to a 10-year bond? When a risk report shows "swap PV01 = USD46,350," what does that number actually mean, and what assumptions lurk behind it?

These questions matter because swaps are the workhorse instruments for interest rate hedging. A corporate treasurer converting fixed-rate debt to floating, a mortgage portfolio manager hedging duration, or a pension fund immunizing liabilities—all rely on swaps and all need accurate sensitivity measures to size their hedges. Get the sensitivity definition wrong, and you'll hedge with the wrong notional. Mix up which curve you're bumping, and your "hedged" position may retain substantial risk.

The challenge is that "PV01" and "DV01" are overloaded terms in fixed income. Depending on the desk and the risk system, "PV01" might mean a sensitivity to a *contract term* (the fixed rate on the swap), or it might mean a sensitivity to a *market move* (a particular curve bump). This chapter makes these definitions explicit, because the hedge you put on depends on *what is being bumped*.

This chapter covers:

1. **The annuity factor and swap PV01** — how the sensitivity to the fixed rate emerges from the discounted fixed-leg payments
2. **Swap DV01 as a curve sensitivity** — what it means to bump a curve, and why multi-curve frameworks complicate the picture
3. **The relationship between swap and bond DV01** — why they measure related but distinct exposures
4. **Hedging with swaps** — the mechanics of DV01 matching, its limitations, and a critical distinction: rate hedging vs spread hedging
5. **Bucket exposures** — how large swap books manage risk across the term structure
6. **Why hedges fail** — curve twists, basis risk, convexity, and when to rebalance
7. **Operational realities** — paying fixed on a swap vs shorting a bond, and corporate hedging flows

Chapter 25 established the mechanics of swap valuation. This chapter builds on that foundation to develop the risk measures that practitioners use daily.

---

## Learning Objectives
- Distinguish **PV01-to-fixed-rate** (quote sensitivity) from **curve DV01/PV01** (market sensitivity).
- Compute a swap annuity (PVBP) from accrual factors and discount factors and use it to compute PV01-to-fixed-rate.
- Define a curve DV01 with an explicit bump object, bump size, units, and sign convention.
- Size a first-order DV01 hedge with swaps and explain the main residual risks: curve shape (twists), basis/spreads, and convexity.
- Debug common scaling/sign errors in swap risk reports (per USD1mm vs per USD100, bp vs decimal, payer vs receiver).

Prerequisites: [Chapter 11: DV01/PV01 — Definitions, Computation, and "What's Being Bumped"](chapters/chapter_11_dv01_pv01_definitions_computation.md), [Chapter 12: Duration — Macaulay, Modified, and the Connection to DV01](chapters/chapter_12_duration_macaulay_modified_dv01.md), [Chapter 14: Key-Rate DV01 and Bucket Exposures](chapters/chapter_14_key_rate_dv01_bucket_exposures.md), [Chapter 22: Curve Risk Management in a Multi-Curve World — Par-Point Deltas, Jacobians, and Controlled Perturbations](chapters/chapter_22_multi_curve_risk_jacobians.md), [Chapter 25: Interest Rate Swaps — Mechanics and Valuation](chapters/chapter_25_interest_rate_swaps_mechanics_valuation.md)

Follow-on: [Chapter 27: Swap Spreads, Asset Swaps, and Swap-Curve Relative Value](chapters/chapter_27_swap_spreads_asset_swaps_swap_curve_rv.md), [Chapter 28: Basis Trades in Rates — OIS-IBOR Basis, Treasury Futures Basis, Swap Spread RV, Curve RV](chapters/chapter_28_basis_trades.md)

## 26.1 Swap PV Refresher: Fixed Receiver vs Fixed Payer

### 26.1.1 The Sign Convention

Before computing sensitivities, we must be precise about whose perspective we're taking. In this chapter, we define swap value from the **fixed-rate receiver's** perspective (receive fixed, pay float):

$$PV_{\text{rec}} = V_{\text{fixed}} - V_{\text{float}}$$

where $V_{\text{fixed}}$ is the present value of the fixed leg and $V_{\text{float}}$ is the present value of the floating leg.

The fixed payer's value is simply the negative:

$$PV_{\text{pay}} = V_{\text{float}} - V_{\text{fixed}} = -PV_{\text{rec}}$$

Interpretation (termination cash amount): if $PV_{\text{rec}}\gt 0$, the swap is an asset to the fixed receiver. A clean way to think about this is: **to terminate the swap today**, the fixed payer would pay approximately $PV_{\text{rec}}$ to the fixed receiver (ignoring any operational decomposition into accrued vs clean PV).

### 26.1.2 The Generic Valuation Formula

With payment dates $T_1, T_2, \ldots, T_n$, accrual fractions $\alpha_i$, fixed rate $K$, forward rates $F_i$, and discount factors $P_d(0, T_i)$, the swap value to the fixed receiver is:

$$\boxed{PV_{\text{rec}} = N \sum_{i=1}^{n} (K - F_i) \alpha_i P_d(0, T_i)}$$

This is a common multi-curve representation: project floating cashflows using a curve appropriate for the floating index, and discount all cashflows using the chosen discount curve (often an OIS-based curve for collateralized swaps).

This representation separates:
- **Projection**: the forward rates $F_i$ used to forecast floating cashflows (a curve specific to the floating index).
- **Discounting**: the discount factors $P_d(0,T_i)$ used to PV cashflows (often an OIS-based curve for collateralized swaps).

---

## 26.2 The Swap Annuity: The Central Quantity

### 26.2.1 Definition and Interpretation

The **swap annuity** (also called the **annuity factor** or **PVBP kernel**) is the present value of a 1% per annum fixed-rate stream on the swap's payment schedule:

$$\boxed{A \equiv \sum_{i=1}^{n} \alpha_i P_d(0, T_i)}$$

This quantity is also commonly called **PVBP** (present value of a basis point) or (sometimes) **PV01**. It is the kernel that converts a rate difference into a dollar PV.

The annuity has a natural interpretation: it is the present value of receiving 1 unit of currency at each payment date, weighted by the accrual fraction. For a 5-year annual-pay swap with discount factors summing to 4.635, the annuity is 4.635—meaning a 1% per annum fixed-rate stream on $100mm notional has PV of approximately $4.635 million.

> **Analogy: The Volume Knob**
>
> Think of a swap's value as music coming from a speaker.
>
> *   **The Signal**: The difference between the Rates (Fixed Rate - Floating Rate). This fluctuates every day.
> *   **The Amplifier (Annuity)**: The Annuity Factor ($A$) determines how *loud* that signal is in dollars.
>
> $$PV = \text{Signal} \times \text{Amplifier}$$
>
> *   **1-Year Swap**: Tiny speakers (Annuity $\approx$ 1.0). A 10bp rate move ("Signal") is barely audible in P&L.
> *   **30-Year Swap**: Stadium concert speakers (Annuity $\approx$ 20.0). A 10bp rate move will blow out your eardrums (huge P&L).
>
> When traders ask "What's the Annuity?", they are asking "How big are the speakers?"

### 26.2.2 Why the Annuity Matters

The annuity appears everywhere in swap analysis:

**1) Par swap rate:** the par fixed rate $K^{\ast}$ is the value of $K$ that makes the swap PV zero. For a forward-starting swap that begins at $T_k$ and ends at $T_{k+m}$, a standard expression is:

$$
\boxed{K^{\ast}(t) = \frac{P_d(t, T_k) - P_d(t, T_{k+m})}{A_{k,m}(t)}} \quad \text{where} \quad A_{k,m}(t)=\sum_{i=k+1}^{k+m} \alpha_i P_d(t,T_i).
$$

**2) Off-market swap PV (rate difference × annuity):** a useful intuition (and often a good approximation for small moves) is:

$$
\boxed{PV_{\text{rec}}(t)\approx N \\, A(t)\\, (K - K^{\ast}(t))}
$$

with the sign flipping for the payer. The main caveat is that $A(t)$ itself depends on the curve(s), so for larger moves you generally reprice rather than treat $A$ as constant.

**3) Swaption payoffs:** A payer swaption exercised at expiry is worth $N \times A \times \max(K^{\text{swap}} - K^{\text{strike}}, 0)$

**4) Fixed-rate PV01:** As we'll see, the sensitivity to the fixed rate is exactly $N \times A \times 0.0001$

---

## 26.3 PV01 to the Fixed Rate: The "Swap Market" Definition

### 26.3.1 Definition: PV01-to-$K$ (Contract-Term Sensitivity)

To avoid ambiguity, define **PV01-to-$K$** as the change in PV when the *contracted fixed rate* $K$ is bumped by 1bp, holding all curves fixed:

$$\boxed{PV01_{K} := PV(K+1\text{ bp}) - PV(K)} \qquad (1\text{ bp}=10^{-4})$$

This is a **contract-term sensitivity** (it answers: “what if we rewrote the fixed rate on the swap?”), and it is closely related to the swap annuity/PVBP.

### 26.3.2 Derivation

Starting from the swap PV formula for a fixed receiver:

$$PV_{\text{rec}} = N \sum_{i=1}^{n} (K - F_i) \alpha_i P_d(0, T_i)$$

Differentiating with respect to $K$ while holding the discount factors and forward rates fixed:

$$\frac{\partial PV_{\text{rec}}}{\partial K} = N \sum_{i=1}^{n} \alpha_i P_d(0, T_i) = N \times A$$

The PV change for a +1bp change in the fixed rate is therefore:

$$\boxed{PV01_{K,\text{rec}} = N \times A \times 0.0001}$$

For the fixed payer, the sign flips:

$$\boxed{PV01_{K,\text{pay}} = -N \times A \times 0.0001}$$

### 26.3.3 Intuition

What does this measure capture? If you receive fixed at rate $K$ and the contractual rate were magically increased by 1bp to $K + 0.0001$, your swap would become more valuable by $N \times A \times 0.0001$. This isn't a market move—it's a change in the contract terms.

When is this relevant?

- **Swaption pricing:** A swaption's payoff depends on the difference between the swap rate at expiry and the strike; the annuity converts this rate difference into dollar value
- **Restrike negotiations:** If a counterparty offers to modify the fixed rate on an existing swap, PV01-to-$K$ tells you the value impact
- **Spread analysis:** Asset swap spreads are often quoted in basis points; converting to dollar value requires multiplying by the annuity

### 26.3.4 Worked Example: Computing the Annuity PV01

**Setup:** Consider a 3-year annual-pay swap with $N = USD100\text{mm}$, accrual fractions $\alpha_i = 1$, and discount factors $P_d(0,1) = 0.98$, $P_d(0,2) = 0.955$, $P_d(0,3) = 0.93$.

**Step 1: Compute the annuity**

$$A = 1 \times 0.98 + 1 \times 0.955 + 1 \times 0.93 = 2.865$$

**Step 2: Compute PV01-to-$K$**

$$PV01_{K,\text{rec}} = 100,000,000 \times 2.865 \times 0.0001 = USD28,650$$

**Interpretation:** If you receive fixed, a 1bp increase in the fixed rate $K$ adds $28,650 to the swap's value. Equivalently, this swap's annuity PV01 is approximately $28,650 per bp.

**Sanity check:** The annuity of 2.865 is roughly the sum of 3 annual discount factors, each close to 0.95. ✓

---

## 26.4 Curve DV01: Sensitivity to Market Rate Changes

### 26.4.1 The Naming Collision

Here's where confusion often arises. Many risk systems report "PV01" or "DV01" to mean something entirely different: the change in swap value when the **yield curve** shifts by 1bp. This is a market sensitivity, not a contract-term sensitivity.

To be precise, we define:

- **PV01-to-$K$:** Sensitivity to the contracted fixed rate (holding curves fixed)
- **Curve PV01 or DV01:** Sensitivity to a curve bump (e.g., +1bp parallel shift)

The distinction matters enormously. Consider an at-market swap where $K = K^{\ast}$ and $PV \approx 0$.

- The PV01-to-$K$ is, by construction, $N \times A \times 0.0001$: it answers “what if the fixed rate in the contract were 1bp higher?”
- A curve DV01 answers a different question: “what if the market curve(s) move by 1bp under a specified bump-and-rebuild rule?”

Under a simple parallel shift that moves the par swap rate, the curve DV01 is often on the same order as $N \times A \times 0.0001$ (with opposite signs for payer vs receiver), but in multi-curve setups and/or partial bumps (discount curve only vs projection curve only) the reported number can differ substantially.

> **Pitfall — “PV01” label collision:** using PV01-to-$K$ (contract-term sensitivity) as if it were a curve DV01 (market sensitivity).
> **Why it matters:** you can size a hedge that is off by a large factor (or the wrong sign) because you hedged the wrong risk.
> **Quick check:** compute $PV01_{K}=N\times A\times 10^{-4}$ from the annuity. If a system’s “PV01” is wildly different, you are probably looking at a different bump object.

From here on, when we say **curve DV01** we mean a *market* sensitivity with an explicit bump definition:
- **Bump object:** a specified curve (at minimum: “discount curve” vs “projection curve”).
- **Bump size:** $1\text{ bp} = 10^{-4}$ in rate units.
- **Units:** currency per 1bp for the stated notional (we will often quote “per $1\text{mm}$ notional”).
- **Sign convention (aligns with the book-wide registry):** $DV01 := PV(\text{rates down }1\text{ bp}) - PV(\text{base})$, so long-duration positions have positive DV01.

**Check (sign, payer vs receiver):** With $DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$, a **receiver** swap is DV01-positive and a **payer** swap DV01-negative under a simple parallel bump that lowers par swap rates. Intuition: when rates fall, receiving fixed at a locked rate becomes more valuable, while paying fixed becomes less valuable.

Because $PV_{\text{pay}}=-PV_{\text{rec}}$ for the same curve inputs, any bump rule implies $DV01_{\text{pay}}=-DV01_{\text{rec}}$ (same bump object, same units).

If you also use the local approximation $PV_{\text{rec}}\approx N\\,A\\,(K-K^{\ast})$ and assume a 1bp move shifts $K^{\ast}$ by about 1bp while $A$ is roughly unchanged, then $DV01_{\text{rec}}\approx N\\,A\times 10^{-4}$ and $DV01_{\text{pay}}\approx -N\\,A\times 10^{-4}$. For the toy annuity $A=2.865$ on $N=USD100$mm, that scale is $\pm 28{,}650\ \text{USD}$ per bp under that bump definition.

> **Desk Reality:** risk reports often label multiple different objects “PV01” or “DV01”.
> **Common break:** the bump rule differs across systems (zero-curve shift vs par-quote shift; “bump and rebuild” vs direct node bump; discount vs projection curve).
> **What to check:** ask for the bump definition and reproduce a 1bp bump-and-reprice in your own pricer with the same curve inputs and units.

### 26.4.2 Why Netting Fixed and Floating DV01 Can Mislead

In principle (for a chosen curve bump), the DV01 of receiving fixed on a swap equals:

$$DV01_{\text{swap}} \approx DV01_{\text{fixed leg}} - DV01_{\text{floating leg}}.$$

The problem is interpretability: the fixed leg’s risk is spread across the swap’s maturity, while the floating leg’s risk is concentrated near the next reset/payment. Collapsing them into one scalar mixes long-horizon and short-horizon risks.

### 26.4.3 Managing Fixed and Floating Risk Separately

In practice, it is often clearer to manage:
- **Fixed-leg risk** with swaps or bonds at relevant maturities (long-horizon curve exposure).
- **Floating-leg risk** with short-dated instruments tied to the index/reset (front-end exposure).

### 26.4.4 The Floating Leg's Short Duration

Intuition: a standard floating-rate note (and, by analogy, the floating leg of a par swap) tends to be **near par at reset dates** because the coupon is reset to a market rate. Between resets, the dominant rate sensitivity comes from discounting the next known payment and the par value at the next reset date. As a result, the floating leg’s duration is typically on the order of the time to the next reset/payment (months, not years).

Here is a simple timeline mental model. Let $T_{\text{next}}$ be the next reset/payment date.

- **Right after a reset:** the floating coupon for the coming accrual period is set to a then-current fixing. The floating leg resembles a par floater, so there is little long-horizon fixed-cashflow exposure left to “drag around.”
- **Between resets:** the most “locked” cashflow is the next coupon amount implied by the last fixing (plus any short stub). The main PV sensitivity is therefore discounting over the short horizon to $T_{\text{next}}$, which is why the floating leg’s DV01 is small compared with the fixed leg’s DV01.

**Toy scale check:** Suppose `T_next = 0.25y` (3 months), $\alpha=0.25$, $N=USD100$mm, and the last fixing is $L=4\\%$. The next coupon cash amount is approximately $100\text{mm}\times 0.25\times 0.04=1.0$ million USD. If you approximate discounting sensitivity as $d(PV)/dy\approx -T_{\text{next}}\cdot PV$ for a parallel zero-rate move, then a 1bp *down* move changes the PV of that coupon by about $T_{\text{next}}\times PV\times 10^{-4}\approx 0.25\times 1{,}000{,}000\times 10^{-4}=25$ USD. That is tiny compared with the tens of thousands of dollars per bp that the fixed leg can generate on the same notional - hence “short duration.”

---

## 26.5 Multi-Curve Considerations: Discount vs Projection PV01

### 26.5.1 The Post-Crisis Framework

A common modern setup uses **different curves** for:
- **Discounting** (to PV cashflows), and
- **Projecting** the floating index (to forecast floating cashflows).

For collateralized swaps, a frequent choice is to discount using an OIS-based curve and to project the floating coupons from a curve tied to the floating index.

### 26.5.2 Splitting Risk by Curve

In multi-curve valuation, the swap PV depends on both curves:

$$PV_{\text{rec}} = N \sum_{i=1}^{n} (K - F_i) \alpha_i P_d(0, T_i)$$

where $F_i$ comes from the projection curve and $P_d(0, T_i)$ comes from the discount curve.

This creates two distinct sensitivities:

1. **Discount-curve DV01:** Bump the discount curve (holding projected forwards fixed) and measure the PV change. This primarily rescales the PV of all cashflows.

2. **Projection-curve DV01:** Bump the projection curve (holding discount factors fixed) and measure the PV change. This changes the expected floating cashflows.

This “split” is a risk attribution choice: it depends on what you are holding fixed while bumping the other curve.

### 26.5.3 Worked Example: Discount vs Projection PV01

**Setup:** 3-year annual-pay payer swap, $N = USD100\text{mm}$, $K = 4\\%$, forwards $F_1 = 3.0\\%$, $F_2 = 3.4\\%$, $F_3 = 3.8\\%$, discount factors $P_1 = 0.98$, $P_2 = 0.955$, $P_3 = 0.93$.

**Base PV:**

$$\sum F_i \alpha_i P_i = 0.03 \times 0.98 + 0.034 \times 0.955 + 0.038 \times 0.93 = 0.09721$$

$$K \times A = 0.04 \times 2.865 = 0.1146$$

$$PV_{\text{pay}} = 100\text{mm} \times (0.09721 - 0.1146) = -USD1,739,000$$

**Discount curve bump (rates down 1bp, discount curve only):**

Using a parallel zero-rate shift with $P_d^b(0,T) = P_d(0,T) e^{+0.0001 \times T}$, the bumped discount factors are approximately $P_1^b = 0.980098$, $P_2^b = 0.955191$, $P_3^b = 0.930279$.

Holding forwards fixed:

$$PV_{\text{pay}}^{\text{bump,disc}} \approx -USD1,739,268$$

$$DV01_{\text{disc}} = PV(\text{rates down }1\text{bp})-PV(\text{base}) \approx -1,739,268 - (-1,739,000) = -USD268$$

**Check (why discount DV01 can be small):** A discount-curve shift mainly rescales *present values*. If the swap is near par, the fixed and floating legs are both large and mostly offset, so the net PV being rescaled is small. A back-of-the-envelope scale is $DV01_{\text{disc}}\sim (\text{net PV})\times (\text{average maturity})\times 10^{-4}$. Here the net PV is about $-USD1.74$mm and the average cashflow horizon is a couple of years, so the scale is on the order of a few hundred dollars per bp—consistent with $-USD268$.

**Projection curve bump (rates down 1bp, projection curve only):**

With $F_i^b = F_i - 0.0001$:

$$\sum F_i^b P_i = 0.09721 - 0.0001 \times 2.865 = 0.0969235$$

$$PV_{\text{pay}}^{\text{bump,proj}} = 100\text{mm} \times (0.0969235 - 0.1146) = -USD1,767,650$$

$$DV01_{\text{proj}} = -1,767,650 - (-1,739,000) = -USD28,650$$

**Interpretation:** For this toy payer swap, the projection-curve DV01 magnitude ($28,650/bp) is much larger than the discount-curve DV01 magnitude ($268/bp). The projection bump directly changes expected floating receipts, while the discount bump mostly rescales PVs.

**Sanity check (magnitude):** $|DV01_{\text{proj}}| = N \times A \times 0.0001$ here because bumping all forwards by 1bp changes the expected floating PV by $N \times A \times 0.0001$ when discount factors are held fixed. ✓

---

## 26.6 Bond DV01 vs Swap DV01

### 26.6.1 The Bond DV01 Definition

A common **yield-based DV01** for a bond is the PV change for a 1bp **decline** in its yield:

$$DV01 := P(y-1\text{ bp})-P(y).$$

For small bumps this is well-approximated by the derivative form:

$$\boxed{DV01 = \frac{1}{10,000} \left( -\frac{dP}{dy} \right)}$$

where $P$ is the bond price and $y$ is the yield. The factor of $1/10,000$ converts from a rate derivative to a "per basis point" measure, and the negative sign makes DV01 positive for a long position (since $dP/dy \lt 0$).

Equivalently, in terms of modified duration $D_{\text{mod}}$ and price $P$:

$$\boxed{DV01 = \frac{P \times D_{\text{mod}}}{10,000}}$$

### 26.6.2 Why Swap and Bond DV01 Differ

Even for a swap and bond with identical maturity, their DV01s can differ substantially:

1. **Cashflow timing:** A bond pays coupons plus principal at maturity; a swap's fixed leg pays only coupons (no principal exchange). The principal payment at maturity gives the bond more weight at the long end.

2. **Floating leg duration:** A swap’s floating leg behaves like a short-maturity instrument (duration on the order of “time to next reset/payment”), so its DV01 is often small compared with the fixed leg. A bond has no floating leg at all.

3. **Curve choice:** In multi-curve valuation, the swap's sensitivity depends on which curve you bump (discount, projection, or both). A bond's yield-based DV01 uses a single yield.

4. **Instrument-specific factors:** Bonds may have embedded optionality, special repo, or credit-spread dynamics that swaps lack.

### 26.6.3 Numerical Comparison

Consider a 5-year 3% annual coupon bond priced at 100.905 (per USD100 face) versus a 5-year at-market payer swap.

**Bond DV01 (per USD100 face):**

Under a +1bp bump to yields, the bond price falls to approximately 100.857.

$$DV01_{\text{bond}} = 100.905 - 100.857 = 0.0476 \text{ per USD100 face}$$

For USD10mm face: $DV01 = USD4{,}758$ per bp.

**Swap "annuity PV01" (per USD1mm notional):**

With annuity $A = 4.635$:

$$PV01_K = 1,000,000 \times 4.635 \times 0.0001 = USD463.5 \text{ per bp per USD1mm}$$

For USD10mm notional: $PV01 = USD4{,}635$ per bp.

The bond DV01 (USD4,758/bp) and swap annuity PV01 (USD4,635/bp) are similar but not identical, reflecting the different cashflow structures.

---

## 26.7 Hedging with Swaps

### 26.7.1 The DV01 Hedge Ratio

To hedge position A with instrument B using a first-order DV01 match (under your chosen bump definition), choose position sizes such that small PV changes offset:

$$\boxed{F_B = -F_A \frac{DV01_A}{DV01_B}}$$

Two quick implications (when $DV01_A$ and $DV01_B$ are reported as **magnitudes**):
- Hedging a **long** position typically requires a **short** hedge instrument (and vice versa).
- The instrument with **higher DV01 per unit** requires **less notional** to hedge.

### 26.7.2 Worked Example (Template): Hedging a Bond with a Swap

**Example Title**: DV01-match a 5Y bond with a 5Y payer swap

**Context**
- You are long a fixed-rate bond and want to neutralize first-order exposure to a *parallel rates move* using a swap.
- This is a common “rates hedge” that leaves you with residual spread/basis and curve-shape risk.

**Timeline (Make Dates Concrete)**
- Trade date: 2026-02-17
- Settlement / effective date: 2026-02-19
- Bond maturity: 2031-02-19 (5Y)
- Swap fixed payment dates (simplified): 2027-02-19, 2028-02-19, 2029-02-19, 2030-02-19, 2031-02-19

**Inputs**
- Bond position: long 10mm face of a 5Y 3% annual-coupon bond.
- Bond DV01 (given by your bond pricer / risk system): $DV01_{\text{bond}}=+USD4{,}758$ per 1bp (positive = gains when yields fall).
- Swap instrument: 5Y **payer** swap (pay fixed, receive float), sized in notional.
- Swap annuity (from discount factors for the swap schedule): $A=4.635$ (annual-pay simplification).
- PV01-to-$K$ per $1\text{mm}$ swap notional: $|PV01_{K}| = 1{,}000{,}000 \times 4.635 \times 10^{-4} = USD463.5 \text{ per bp per } USD1\text{mm}.$

**Outputs (What You Produce)**
- Swap notional $N_{\text{swap}}$ to make the package approximately DV01-neutral to a parallel move.

**Step-by-step**
1. Put both risks in the same units: dollars per 1bp for the positions you will actually hold.
2. A payer swap has **negative duration** (it tends to gain when rates rise), so it is the natural hedge against a long bond.
3. Size the swap so the DV01s offset in magnitude: $N_{\text{swap}} \approx \frac{DV01_{\text{bond}}}{|PV01_{K,\text{swap per mm}}|} = \frac{4{,}758}{463.5} \approx 10.27\text{ mm}.$
   (In a production setup, use the risk system’s *curve DV01* for the swap under the same bump definition as the bond, not just the annuity.)

**Cashflows (schematic)**
| Date | Cashflow | Explanation |
|---|---|---|
| each coupon date | Bond: +0.03 x 10mm | fixed coupons received |
| maturity | Bond: +10mm | principal received |
| each fixed leg date | Swap: pay $K_{\text{swap}} \times \alpha_i \times N_{\text{swap}}$ | fixed coupons paid |
| each float leg date | Swap: receive floating coupon | index-linked receipts |

**P&L / Risk Interpretation**
- Parallel rates +10bp (small move): bond loses about $4{,}758\times 10 \approx USD47{,}580$; payer swap gains about $USD47{,}580$ → net near 0.
- Residual risk: if bond yields and swap rates do not move together (swap spread/basis), the hedge produces P&L even when “DV01-neutral”.

**Sanity Checks**
- Units: $A$ is in years; $N\times A\times 10^{-4}$ is currency per bp.
- Sign: long bond has $DV01\gt 0$; payer swap has $DV01\lt 0$ under the “rates down” convention.
- Methodology: confirm the bump object (par swap rate curve vs zero curve; discount vs projection) before trusting the hedge size.

> **Technique: The Hedge Ratio Shortcut**
>
> Need a quick hedge ratio on the trading desk? Use the ratio of Durations.
>
> $$\text{Hedge Ratio} \approx \frac{\text{Duration}_{\text{Bond}}}{\text{Duration}_{\text{Swap}}}$$
>
> *   **Scenario**: Hedge USD100M of 10-Year Bonds (Duration $\approx$ 8.5) with 5-Year Swaps (Duration $\approx$ 4.5).
> *   **Ratio**: $8.5 / 4.5 \approx 1.9$.
> *   **Trade**: You need roughly $1.9 \times USD100M = USD190M$ of 5-year swaps.
>
> *Why?* Because the 5-year swap has "smaller speakers" (lower duration/annuity), you need *more* of them to match the "loudness" (risk) of the 10-year bond.

### 26.7.3 What DV01 Hedging Does—and Doesn't—Achieve

DV01 matching neutralizes **first-order** exposure to **parallel** rate moves. It does not guarantee protection against:

1. **Curve twists:** If the yield curve steepens or flattens, instruments with different cashflow profiles will respond differently even if they had matched parallel DV01s.

2. **Convexity effects:** For large rate moves, second-order (convexity) terms matter. A bond and swap with matched DV01s but different convexities will diverge.

3. **Basis risk:** Treasury yields and swap rates can move differently. A swap hedge against a Treasury bond leaves exposure to swap spread changes.

### 26.7.4 Rate Hedging vs Spread Hedging: A Critical Distinction

Here is a fundamental insight that risk reports often obscure: **a DV01 hedge does not eliminate interest rate risk—it transforms rate risk into spread risk.**

When you hedge a bond with a swap, your P&L is no longer driven by rate levels. Instead, it's driven by the *relative movement* of bond yields vs swap rates—the swap spread.

$$\boxed{\text{Residual PnL} \approx DV01 \times (\Delta y_{\text{bond}} - \Delta y_{\text{swap}})}$$

If Treasury yields rise 10bp and swap rates rise 10bp, your hedged position is flat. But if Treasury yields rise 10bp and swap rates rise only 6bp (a 4bp spread widening), you lose money.

> **Desk Reality:** “DV01-neutral” often gets paraphrased as “hedged.”
> **Common break:** a DV01 match can remove parallel *rates* exposure but leave large *swap-spread/basis* and *curve-shape* exposures.
> **What to check:** stress the package under (i) a parallel shift, (ii) a steepener/flattener, and (iii) a swap-spread move; confirm which risk you are actually neutralizing.

**Table: What a Swap Hedge Protects Against vs What It Doesn't**

| Risk Factor | Hedged by DV01 Match? | Explanation |
|-------------|----------------------|-------------|
| Parallel rate shift | Yes | Both legs move together |
| Curve twist (steepening/flattening) | Partially | Depends on tenor match |
| Swap spread change | No | Bond and swap respond differently |
| Convexity (large moves) | No | Second-order effect not captured |
| Credit spread of specific bond | No | Swap doesn't track issuer credit |

Even when bond yields and swap rates are highly correlated, **basis/spread risk** remains. A swap hedge cannot eliminate security-specific effects (liquidity, repo specialness, issuer credit spread) and will generally leave exposure to the relative move between the bond yield and the swap rate.

---

## 26.8 Bucket Exposures for Swap Books

### 26.8.1 Why Bucket Analysis?

Large swap books often have exposures distributed across the curve in a way that a single scalar DV01 cannot summarize. **Bucket analysis** computes sensitivity to many small regions of the curve so you can see (and hedge) the term structure of the risk.

### 26.8.2 The Mechanics of Bucket Shifts

In this context, a **bucket** is a small region of the curve (often expressed in terms of forward rates). A **bucket shift** is a 1bp shift applied to the rates inside one bucket while holding other buckets fixed.

The procedure is straightforward:

1. Define buckets (e.g., 6-month forward rates at 0.5y, 1y, 1.5y, ...)
2. Bump each forward rate by 1bp while holding all others fixed
3. Recompute the swap PV
4. The bucket exposure is the PV change for that bucket

### 26.8.3 Example: 6-Year Par Swap (One Bucket)

Consider receiving fixed on $100\text{mm}$ of a 6-year par swap under a flat curve (for illustration). To compute the exposure to the six-month forward rate 2.5 years forward, bump **that** forward rate by +1bp, keep all other forwards the same, rebuild the discount factors implied by the perturbed forwards, and reprice the swap.

$$-USD100,000,000 \times (99.995813\\% - 100\\%) = USD4,187$$

**Check (order of magnitude):** A single 6-month forward bucket bumped by 1bp changes expected cashflows for roughly one accrual period. A crude scale is

$$|\Delta PV|\approx N\times \alpha \times P(0,T)\times 10^{-4}.$$

With $N=USD100$mm, $\alpha\approx 0.5$, and a mid-curve discount factor around $0.95$-$0.99$, this gives about USD4.75k-USD4.95k per bp - consistent with the same ballpark as USD4,187. The exact number depends on the bump-and-rebuild rule and the position’s bucket weights.

Across all buckets, the **sum of bucket exposures** is (approximately) the exposure to a simultaneous 1bp shift in all forwards. Under simplifying assumptions (e.g., a flat curve and consistent bump/rebuild rules), this sum can be very close to the fixed-leg DV01.

### 26.8.4 Using Bucket Exposures

The sum of bucket exposures equals (approximately) the total DV01 under a parallel shift. But the granular view enables more precise hedging:

- **Front-end hedging:** short-term rate futures and other short-dated instruments can hedge specific forward buckets (instrument choice depends on the index/market)
- **Curve trades** can be designed to neutralize exposure to specific curve segments
- **Risk limits** can be set by bucket rather than in aggregate

### 26.8.5 Connection to Key-Rate DV01

Bucket exposures are to swaps what key-rate DV01s are to bonds. Chapter 14 develops the key-rate methodology for bonds, showing how to compute sensitivity to specific points on the yield curve. The main differences:

- **Key rates** typically use fewer points (e.g., 2Y, 5Y, 10Y, 30Y) with interpolated shapes
- **Buckets** use many narrow segments (every 3 or 6 months) with flat shifts within each bucket

Both approaches reveal the term structure of risk. For swap books, buckets are preferred because the forward rate structure maps directly to hedgeable instruments (futures).

> **Visual: The Bucket Risk Histogram**
>
> Consider a position: Receive Fixed 6Y Swap vs Pay Fixed 4Y Swap, sized to zero parallel DV01.
>
> *   **6Y Swap**: Risk is distributed evenly across 6 years
> *   **4Y Swap (short)**: Negative risk concentrated in first 4 years
>
> *   **Net Profile**:
>     *   Years 0-4: Net negative exposure (short risk)
>     *   Years 4-6: Net positive exposure (long risk)
>     *   Sum: Zero (parallel DV01 neutral)
>
> Even with zero *parallel* DV01, the portfolio can have large offsetting bucket exposures and therefore meaningful P&L under steepening/flattening moves.

### 26.8.6 Hedging Swap Books: Fixed Leg with Swaps, Floating with Futures

Given the different risk horizons of fixed and floating legs, large swap books are often hedged in two stages:

1. **Fixed-leg risk:** Hedged with offsetting swaps or bonds at the relevant maturities
2. **Floating-leg risk:** Hedged with short-term futures or short-dated instruments that target the specific forward buckets

### 26.8.7 Swaps as Factor Instruments

Unlike bonds—which carry idiosyncratic features like specific coupons, odd maturities, and repo specialness—swaps are standardized curve points. A 5-year swap is the 5-year point on the swap curve; a 10-year swap is the 10-year point.

This makes swaps preferred instruments for systematic curve trading and factor hedging. Chapter 16 develops PCA-based curve hedging, where level, slope, and curvature factors are extracted from the term structure. Swaps provide "clean" exposure to these factors without the noise of bond-specific effects.

---

## 26.9 Why Swap Hedges Fail

### 26.9.1 Curve Twist Risk

A DV01-neutral hedge can have significant P&L under non-parallel curve moves. Consider hedging a 5-year zero-coupon bond with a 5-year swap:

**The mismatch:** The zero bond's exposure is concentrated entirely at the 5-year point. The swap's fixed leg has PV weight at each payment date (years 1 through 5). Under a parallel shift, these exposures offset. Under a twist where short rates rise and long rates fall, they don't.

**Worked Example:**

With DV01-matched positions (bond + payer swap), apply a twist: +10bp at years 1-2, 0bp at year 3, -10bp at years 4-5.

These dollar figures are **illustrative**. The point is that a "DV01-neutral" hedge can still carry meaningful **key-rate** exposure that produces P&L under twists.

- The zero bond gains as the 5-year rate falls: approximately +USD43,610
- The swap loses because the annuity's PV weights extend across the curve, and the short end (where rates rose) pulls value down: approximately -USD41,433

**Residual P&L:** +USD2,177

This residual arises from "key-rate mismatch"—the instruments have different exposures across the term structure.

**Quantifying curve risk:** The P&L from a curve twist can be expressed in terms of key-rate exposures:

$$\text{PnL}_{\text{twist}} = \sum_i KR01_i^{\text{portfolio}} \times \Delta y_i$$

where $KR01_i$ is the key-rate exposure to tenor $i$ and $\Delta y_i$ is the rate change at that tenor. For a DV01-neutral portfolio, $\sum_i KR01_i = 0$, but individual $KR01_i$ values can be large with opposite signs—creating twist exposure.

### 26.9.2 Basis Risk

Swap hedges against Treasury positions leave exposure to swap spread changes. If Treasury yields rise 10bp but swap rates rise only 6bp (spread widening), a DV01-neutral hedge loses money:

- Bond loss: $-4,758 \times 10 = -USD47,580$
- Swap gain: $+4,758 \times 6 = +USD28,548$
- Residual: $-USD19,032$

This basis risk reflects that Treasury yields and swap rates can move differently because they embed different forces (liquidity, funding, and supply/demand). For corporate bonds, the mismatch is typically larger because a swap hedge does not remove issuer-specific credit-spread risk.

**Worked Example: Basis Risk Under Spread Widening**

**Setup:** Long $10mm face 5-year Treasury (DV01 = $4,758), hedged with $10.3mm payer swap (DV01 = $4,758).

**Scenario:** Treasury yield rises 15bp, swap rate rises 10bp (5bp spread widening).

**P&L calculation:**
- Treasury loss: $-4,758 \times 15 = -USD71,370$
- Swap gain: $+4,758 \times 10 = +USD47,580$
- **Net loss: $-USD23,790$**

**Interpretation:** The portfolio was "hedged" against rate moves, but it lost money because the hedge only worked if Treasury and swap moved together. The 5bp spread widening cost nearly USD24,000.

### 26.9.3 Convexity Mismatch

For large rate moves, first-order (DV01) hedging becomes inadequate. A bond and swap with matched DV01s but different convexities will diverge as rates move significantly. The more convex instrument outperforms when rates move in either direction.

**Why convexity differs:** A coupon bond and a swap fixed leg have similar but not identical convexity profiles because the bond has a principal payment at maturity while the swap does not. Additionally, if the hedge uses a different maturity instrument (e.g., hedging a 10-year bond with a 5-year swap), the convexities will differ substantially.

### 26.9.4 When to Rebalance Your Hedge

Static hedges drift out of alignment as time passes and curves move. Rebalancing triggers include:

| Trigger | Action |
|---------|--------|
| DV01 drift > ~5-10% of original (illustrative) | Resize hedge position |
| Curve shape changed significantly | Check bucket exposures; may need term adjustment |
| Large rate move (> 50bp) | Check convexity impact; may need gamma hedge |
| Near payment date | Floating leg DV01 changes at reset |
| Approaching roll date | If hedging with futures, roll or close |

The cost of rebalancing (bid-ask, market impact) must be weighed against the cost of carrying a misaligned hedge. The exact thresholds and cadence are desk-specific.

> **Desk Reality: The Drift You Don't See**
>
> A hedge that was perfectly sized three months ago may now be off by 10%+ simply due to:
> - Time decay (both DV01 and annuity decrease as maturity shortens)
> - Curve shape changes (key-rate profile shifts)
> - Payment resets (floating leg DV01 resets at each fixing)
>
> Many "mystery P&L" events trace to hedges that weren't rebalanced. Set calendar reminders to review hedge ratios, especially for long-dated positions.

---

## 26.10 Paying Fixed vs Shorting a Bond: Operational Differences

### 26.10.1 Economic Equivalence in Theory

In frictionless markets, paying fixed on a swap is economically similar to shorting a par bond of the same maturity: both positions profit when rates rise. The cash flows are nearly equivalent—both involve paying fixed coupons and having offsetting floating-rate exposure.

Specifically:
- **Receive fixed + Long bond** ≈ zero rate exposure (if matched)
- **Pay fixed** ≈ **Short bond** (in terms of rate direction)

### 26.10.2 Why They Differ in Practice

Despite the economic similarity, paying fixed on a swap and shorting a bond differ dramatically in operational and balance sheet terms:

| Aspect | Pay Fixed on Swap | Short a Bond |
|--------|-------------------|--------------|
| **Balance sheet** | Largely off-balance sheet (only MTM recognized) | On-balance sheet (repo liability, borrowed securities) |
| **Initial funding** | No cash outlay (zero-NPV at inception) | Must borrow the bond (repo); may require margin |
| **Repo exposure** | None | Exposed to squeeze if bond becomes special |
| **Delivery risk** | None | May face fails if unable to deliver |
| **Counterparty credit** | Swap counterparty risk | No counterparty risk (you owe them) |
| **Accounting** | Often hedge accounting eligible | More complex accounting treatment |
| **Capital treatment** | Generally lower capital charge | Higher capital for some institutions |

### 26.10.3 The Repo Squeeze Dynamic

When a Treasury becomes "special" in the repo market—meaning it's expensive to borrow—shorting that bond becomes costly. The short seller must pay the repo rate to borrow the bond, and for special securities, this rate can be well below general collateral rates.

Mechanically, scarcity of a particular bond increases the value of *borrowing* it. Lenders can demand a more favorable repo rate, and shorts effectively pay for the privilege of borrowing the security. This is one reason “short bond” and “pay fixed” can diverge in realized P&L even if they look similar in a frictionless model.

> **Desk Reality:** if a bond is special (hard/expensive to borrow), it can be operationally cheaper to express a similar “rates short” view via swaps than by shorting the bond.
> **Common break:** treating “pay fixed ≈ short bond” as identical ignores financing/borrow costs; the package can pick up repo-specialness and swap-spread P&L.
> **What to check:** stress (i) a repo-specialness move (borrow cost) and (ii) a swap-spread move, in addition to the parallel-rate stress that the hedge ratio was built on.

### 26.10.4 When to Use Each

| Use Case | Preferred Instrument | Reason |
|----------|---------------------|--------|
| Hedging rate exposure, balance sheet constrained | Swap | Off-balance sheet, no repo hassle |
| Expressing view on specific issue | Short the bond | Direct exposure to that security |
| Hedging corporate issuance | Swap | Natural offset to fixed-rate debt |
| Relative value (specific bond vs curve) | Short bond + receive fixed | Isolates bond-specific value |
| Short-term tactical trade | Either | Depends on liquidity and cost |

---

## 26.11 Corporate Hedging: An End-to-End Example

### 26.11.1 The Canonical Use Case

One canonical corporate use case is to **convert fixed-rate debt into floating**: issue fixed-rate bonds, then enter a **receive-fixed / pay-floating** swap. The fixed receipts on the swap offset the fixed coupons on the bond, leaving a floating-rate liability.

But the flows don't stop with the corporate. Here's the full chain:

### 26.11.2 Step-by-Step: Corporate Issues, Bank Hedges

**Step 1: Corporate Issues Fixed Bond**

Company XYZ issues USD500mm of 5-year fixed-rate bonds at 4.50% to institutional investors. The company now has fixed-rate debt: it pays 4.50% annually regardless of where rates go.

**Step 2: Corporate Enters Swap**

XYZ wants floating-rate exposure (perhaps it has floating-rate assets). It enters a payer swap with a bank:
- XYZ **receives** fixed 4.00% (the 5-year swap rate)
- XYZ **pays** 3-month SOFR

**Net result for XYZ:**
- Pays: 4.50% (bond fixed) + SOFR (swap floating) - 4.00% (swap fixed receipt)
- Simplifies to: **SOFR + 0.50%** (effectively floating-rate debt at SOFR + 50bp)

**Step 3: Bank Hedges**

The bank is now **paying fixed, receiving floating**. To manage the resulting rate risk, the bank may hedge with other swaps and/or government bonds (the exact hedge depends on the desk’s risk methodology and constraints).

For example, a bank might combine:
- a swap hedge (to offset swap-curve DV01), and/or
- a government bond hedge (to offset Treasury-curve DV01),
which in turn can create flows that move swap rates and Treasury yields differently.

**Step 4: Market Impact**

Heavy corporate issuance plus hedging flows can move swap spreads because the “rates hedge” is not the same as the “spread hedge”. The direction and size depend on the instruments used for hedging and on relative supply/demand in the Treasury and swap markets.

> **Desk Reality:** A “rates hedge” can leave a position dominated by *spread* moves.
> **Common break:** a portfolio is DV01-neutral but still has large P&L when swap spreads move.
> **What to check:** decompose P&L into (i) rate level, (ii) curve shape, and (iii) swap-spread/basis components; then decide whether you need a separate spread hedge (e.g., a spread lock) in addition to the rates hedge.

### 26.11.3 Cash Flow Summary

| Party | Pays | Receives | Net Exposure |
|-------|------|----------|--------------|
| **Bond investors** | Cash (to buy bonds) | 4.50% fixed | Long fixed-rate credit |
| **XYZ Corp** | 4.50% (bond) + 4.00% (swap) | SOFR | Floating-rate liability |
| **Bank (swap dealer)** | SOFR | 4.00% | Long fixed / short floating |
| **Treasury market** | (receives supply from bank hedging) | | |

---

## 26.12 Practical Notes

### 26.12.1 Common Scaling Errors

| Error | Impact |
|-------|--------|
| Confusing per $100 with per $1mm | Off by factor of 10,000 |
| Forgetting the 0.0001 bp factor | Off by factor of 10,000 |
| Wrong notional (contract vs risk notional) | Can be off by factor of 2 for certain amortizing swaps |

Rule of thumb: if a DV01 is quoted **per 100 face**, divide by 100 before scaling by the face amount.

### 26.12.2 Implementation Considerations

| Issue | Consideration |
|-------|---------------|
| Schedule generation | Stub periods, business day adjustments, day counts all affect $\alpha_i$ |
| Accrual precision | A 28-day stub is not 0.25 years; use exact day count fractions |
| Bump methodology | "Bump and rebuild" maintains curve arbitrage-freeness but distributes risk differently than direct node bumps |
| Numerical stability | 1bp bumps are standard; too small creates numerical noise, too large introduces convexity |

### 26.12.3 Verification Tests

| Test | What to Check |
|------|---------------|
| Par swap PV | A par swap should have PV ≈ 0 |
| PV01-to-$K$ identity | Should equal $\pm N \times A \times 0.0001$ exactly |
| Notional scaling | PV and PV01 should scale linearly with $N$ |
| Sign consistency | Payer and receiver PV01s should be opposite |
| Bucket sum | Sum of bucket exposures ≈ parallel DV01 |

---

## Summary

1. “PV01” is ambiguous unless you specify **what is being bumped** (contract fixed rate vs a market curve).
2. The swap annuity / PVBP is $A=\sum_i \alpha_i P_d(0,T_i)$; it converts a rate difference into a dollar PV.
3. PV01-to-$K$ is a **contract-term** sensitivity: $PV01_{K}=\pm N\times A\times 10^{-4}$ (sign depends on receiver vs payer).
4. Curve DV01 is a **market** sensitivity: $DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$, with an explicit bump object and units.
5. In a multi-curve setup, you must state whether the bump hits the **discount curve**, the **projection curve**, or both.
6. DV01 hedging matches first-order exposure to the chosen bump, but it does **not** hedge twists (key-rate/bucket mismatch), convexity, or swap-spread/basis moves.
7. Bucket exposures reveal the term structure of risk; their sum is typically close to a parallel DV01 under consistent bump rules.
8. Operational details (repo specialness, funding/balance sheet, settlement conventions) can make “pay fixed” behave differently from “short the bond.”

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Swap annuity | $A = \sum \alpha_i P_d(0, T_i)$ | Converts rate changes to dollar values; central to PV01 |
| PV01-to-$K$ | $\pm N \times A \times 0.0001$ | Sensitivity to fixed rate; swaption payoffs |
| Curve PV01 | PV change from curve bump | Market risk measure for hedging |
| Discount vs projection PV01 | Separate sensitivities to each curve | Multi-curve risk attribution |
| Bucket exposure | PV01 to individual forward rates | Granular term structure risk |
| DV01 hedge ratio | $F_B = -F_A(DV01_A/DV01_B)$ | Sizing hedges for first-order neutrality |
| Rate hedge vs spread hedge | DV01 hedge transforms rate risk to spread risk | "Hedged" doesn't mean risk-free |
| Basis risk | Treasury and swap move differently | Residual exposure in bond-swap hedges |

---

## Notation

| Symbol | Definition |
|--------|------------|
| $N$ | Swap notional (currency units) |
| $T_1, \ldots, T_n$ | Payment dates |
| $\alpha_i$ | Accrual fraction for period $i$ |
| $K$ | Fixed swap rate |
| $F_i$ | Forward rate for floating period $i$ |
| $P_d(0, T_i)$ | Discount factor |
| $A$ | Swap annuity: $\sum \alpha_i P_d(0, T_i)$ |
| $V_{\text{fixed}}, V_{\text{float}}$ | PV of fixed and floating legs |
| $PV_{\text{rec}}, PV_{\text{pay}}$ | Swap PV to receiver/payer |
| $PV01_K$ | Sensitivity to fixed rate $K$ |
| $DV01$ | Dollar value of one basis point (curve sensitivity) |
| $KR01_i$ | Key-rate 01 for tenor $i$ |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the swap annuity? | $A = \sum \alpha_i P_d(0, T_i)$, the PV of a 1% p.a. stream on the swap schedule |
| 2 | In swap markets, what does "PV01" often mean? | Change in swap value for 1bp change in the fixed rate $K$ |
| 3 | Formula for PV01-to-$K$ for a fixed receiver? | $PV01_{K,\text{rec}} = +N \times A \times 0.0001$ |
| 4 | What is $PV_{\text{rec}}$ in terms of leg values? | $PV_{\text{rec}} = V_{\text{fixed}} - V_{\text{float}}$ |
| 5 | Why can a single "swap DV01" number be misleading? | It mixes long-term fixed-rate risk with short-term floating-rate risk |
| 6 | What's the duration of a floating leg approximately? | Time to next reset (often just a few months) |
| 7 | Bond DV01 formula (yield-based)? | $DV01 = (1/10,000)(-dP/dy)$ |
| 8 | Relation between DV01 and modified duration? | $DV01 = P \times D_{\text{mod}} / 10,000$ |
| 9 | DV01 hedge ratio formula? | $F_B = -F_A(DV01_A/DV01_B)$ |
| 10 | What does DV01 matching neutralize? | First-order exposure to parallel rate moves |
| 11 | What does a DV01 hedge transform rate risk into? | Spread risk (exposure to swap spread changes) |
| 12 | Three reasons DV01 hedges can fail? | Curve twists, convexity mismatch, basis risk |
| 13 | What is "basis risk" in a Treasury vs swap hedge? | Treasury and swap rates can move differently (spread changes) |
| 14 | What is a bucket exposure? | PV01 to a single forward rate segment |
| 15 | Why use bucket analysis for swap books? | Reveals term structure of risk; enables precise hedging with futures |
| 16 | In multi-curve, what does the projection curve determine? | Expected floating cashflows (forward rates) |
| 17 | In multi-curve, what does the discount curve determine? | Present values of all cashflows |
| 18 | Why is discount PV01 often smaller than projection PV01? | Discount bump scales both legs; projection bump changes only floating |
| 19 | Par swap rate formula (simple form)? | $K^{\ast} = (P(t,T_k) - P(t,T_{k+m})) / A$ |
| 20 | Operational difference: paying fixed vs shorting bond? | Swap is off-balance sheet, no repo exposure; short bond requires borrowing, exposed to squeeze |

---

## Mini Problem Set

### Questions

1. **[Easy - Calculation]** Given a 4-year semiannual-pay swap with discount factors $P(0.5) = 0.985$, $P(1) = 0.97$, $P(1.5) = 0.955$, $P(2) = 0.94$, $P(2.5) = 0.925$, $P(3) = 0.91$, $P(3.5) = 0.895$, $P(4) = 0.88$, compute the annuity $A$ and PV01-to-$K$ for $N = USD50\text{mm}$.

2. **[Easy - Calculation]** With forwards $F_1 = 2.5\\%$, $F_2 = 2.8\\%$, $F_3 = 3.1\\%$, $F_4 = 3.4\\%$ (annual) and discount factors $P(1) = 0.975$, $P(2) = 0.95$, $P(3) = 0.92$, $P(4) = 0.89$, compute the par swap rate $K^{\ast}$.

3. **[Medium - Application]** For a payer swap with $K = 3.5\\%$ (above par), using the data from problem 2, compute the swap PV per $USD100\text{mm}$ notional. Is it positive or negative to the payer?

4. **[Medium - Hedging]** A 10-year bond has DV01 = USD8,500/bp. A 10-year swap has annuity PV01 = USD750/bp per USD1mm notional. What swap notional hedges the bond?

5. **[Medium - Scenario]** Explain why a DV01-neutral position can lose money under a curve steepening. Give a specific example.

6. **[Medium - Basis Risk]** A Treasury rises 12bp while the matched-maturity swap rate rises only 8bp. If you hedged $10mm Treasury (DV01 = $4,800) with a payer swap, compute the P&L.

7. **[Medium - Multi-curve]** In a multi-curve framework, if you bump only the discount curve by +1bp, what happens qualitatively to a payer swap's PV?

8. **[Medium - Corporate]** A corporation issues USD200mm 5-year fixed bonds at 5.00% and enters a payer swap at 4.50%. What is the corporation's effective funding rate?

9. **[Hard - Floating Leg]** Why is the floating leg's DV01 approximately proportional to the time to next reset? What happens to the floating leg DV01 immediately after a rate fixing?

10. **[Hard - Debugging]** A par swap calculation shows PV = +USD50,000 instead of zero. List three possible causes.

11. **[Hard - Bucket]** A 6-year receive-fixed swap is hedged with a 4-year pay-fixed swap to zero parallel DV01. The curve then steepens (short rates up, long rates down). Does the portfolio gain or lose?

12. **[Hard - Integration]** Explain why swaps are preferred over bonds for systematic factor (PCA) hedging. What bond-specific issues do swaps avoid?

---

### Solution Sketches (Selected)

**1.** Annuity $A = \sum_{i=1}^{8} 0.5 \times P(0.5i) = 0.5 \times (0.985 + 0.97 + 0.955 + 0.94 + 0.925 + 0.91 + 0.895 + 0.88) = 0.5 \times 7.46 = 3.73$.

$PV01_K = 50\text{mm} \times 3.73 \times 0.0001 = USD18,650$.

**2.** Numerator: $\sum F_i \times 1 \times P_i = 0.025 \times 0.975 + 0.028 \times 0.95 + 0.031 \times 0.92 + 0.034 \times 0.89 = 0.02438 + 0.0266 + 0.02852 + 0.03026 = 0.10976$.

Annuity: $A = 0.975 + 0.95 + 0.92 + 0.89 = 3.735$.

$K^{\ast} = 0.10976 / 3.735 = 2.94\\%$.

**3.** $PV_{\text{pay}} = N(\sum F_i P_i - KA) = N(0.10976 - 0.035 \times 3.735) = N(0.10976 - 0.13073) = -0.02097 \times N$.

For $N = 100\text{mm}$: $PV_{\text{pay}} = -USD2,097,000$ (negative to payer because they pay above-market fixed rate).

**4.** $N_{\text{swap}} = DV01_{\text{bond}} / PV01_{\text{swap per mm}} = 8,500 / 750 = 11.33\text{mm}$. Enter a payer swap to offset the long-bond exposure.

**5.** A DV01-neutral portfolio has offsetting exposures that sum to zero for parallel shifts but may have opposite signs across tenors. In a steepening (short rates up, long rates down):
- If you're long short-end risk (positive bucket exposure at short end) and short long-end risk (negative bucket exposure at long end), you lose on both: the short end rises (your long position loses) and the long end falls (your short position gains, but you're short, so the falling rates hurt you).
- Example: Receive fixed 10Y, pay fixed 2Y sized to zero DV01. Steepening: 2Y rate +20bp, 10Y rate -10bp. The 2Y payer leg gains (good), but the 10Y receiver leg loses more because it has higher weight at the now-more-valuable long end.

**6.**
- Treasury loss: $-4,800 \times 12 = -USD57,600$
- Swap gain: $+4,800 \times 8 = +USD38,400$ (payer swap gains when rates rise)
- **Net loss: $-USD19,200$** (the 4bp spread widening caused the loss)

---

## References

- (Tuckman and Serrat, *Fixed Income Securities*, “Valuation of Swaps, Continued”; “Note on the Measurement of Fixed and Floating Interest Rate Risk”; “Bucket Shifts and Exposures”.)
- (Hull, *Options, Futures, and Other Derivatives*, “29.4 Hedging Interest Rate Derivatives”; “Multiple Yield Curves”.)
- (Musiela and Rutkowski, *Martingale Methods in Financial Modelling*, Section 13.3 (formulas 13.12–13.13: forward swap rate and level process / PVBP).)
- (O’Kane, *Modelling Single-name and Multi-name Credit Derivatives*, “2.6.5 The Breakeven Swap Rate” (PV01 and swap-rate sensitivity).)
- (Neftci, *Principles of Financial Engineering*, “3.8.1 DV01 and PV01”; “4.6.3.1 Changing portfolio duration” (DV01/PV01 language applied to swaps).)
- (Andersen and Piterbarg, *Interest Rate Modeling*, “6.5.3 Tenor Basis and Multi-Index Curve Group Construction” (curve separation and multi-curve setup).)
