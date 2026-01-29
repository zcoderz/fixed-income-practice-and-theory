# Chapter 26: Swap PV01, DV01, and Hedging with Swaps

---

## Introduction

A 10-year interest rate swap has roughly 10 years of duration—but what exactly are we measuring, and how does it compare to a 10-year bond? When a risk report shows "swap PV01 = $46,350," what does that number actually mean, and what assumptions lurk behind it?

These questions matter because swaps are the workhorse instruments for interest rate hedging. A corporate treasurer converting fixed-rate debt to floating, a mortgage portfolio manager hedging duration, or a pension fund immunizing liabilities—all rely on swaps and all need accurate sensitivity measures to size their hedges. Get the sensitivity definition wrong, and you'll hedge with the wrong notional. Mix up which curve you're bumping, and your "hedged" position may retain substantial risk.

The challenge is that "PV01" and "DV01" are overloaded terms in fixed income. Tuckman notes that "PV01" in swap markets is "normally meant to mean the change in the value of a swap for a one-basis point change in its fixed rate"—but many risk systems use the same label to mean something entirely different: the sensitivity to a curve bump. Understanding these distinctions is essential for anyone who trades, hedges, or risk-manages swap positions.

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

## 26.1 Swap PV Refresher: Fixed Receiver vs Fixed Payer

### 26.1.1 The Sign Convention

Before computing sensitivities, we must be precise about whose perspective we're taking. Following Tuckman's convention, we define swap value from the fixed-rate receiver's perspective:

$$PV_{\text{rec}} = V_{\text{fixed}} - V_{\text{float}}$$

where $V_{\text{fixed}}$ is the present value of the fixed leg and $V_{\text{float}}$ is the present value of the floating leg. Tuckman states this explicitly: "The value of a swap to the fixed receiver/floating payer is $V_{\text{Fixed}} - V_{\text{Float}}$, and the value of a swap to the floating receiver/fixed payer is $V_{\text{Float}} - V_{\text{Fixed}}$."

The fixed payer's value is simply the negative:

$$PV_{\text{pay}} = V_{\text{float}} - V_{\text{fixed}} = -PV_{\text{rec}}$$

This convention means that if $V_{\text{fixed}} - V_{\text{float}} > 0$, the swap has positive value to the fixed receiver and negative value to the fixed payer. As Tuckman explains, "If the parties wish to terminate the swap, so that neither party need make any more payments to the other, the fixed payer will have to make a positive payment of $V_{\text{Fixed}} - V_{\text{Float}}$ to the fixed receiver."

### 26.1.2 The Generic Valuation Formula

With payment dates $T_1, T_2, \ldots, T_n$, accrual fractions $\alpha_i$, fixed rate $K$, forward rates $F_i$, and discount factors $P_d(0, T_i)$, the swap value to the fixed receiver is:

$$\boxed{PV_{\text{rec}} = N \sum_{i=1}^{n} (K - F_i) \alpha_i P_d(0, T_i)}$$

This formula encapsulates the modern multi-curve approach. Hull describes the valuation method: "Assume that forward rates will be realized... then discount the resulting cash flows at the OIS... rate." The forward rates $F_i$ come from a projection curve (historically LIBOR, now typically SOFR), while the discount factors $P_d(0, T_i)$ come from the OIS discounting curve.

---

## 26.2 The Swap Annuity: The Central Quantity

### 26.2.1 Definition and Interpretation

The **swap annuity** (also called the **annuity factor** or **PVBP kernel**) is the present value of a 1% per annum fixed-rate stream on the swap's payment schedule:

$$\boxed{A \equiv \sum_{i=1}^{n} \alpha_i P_d(0, T_i)}$$

Andersen and Piterbarg provide the formal definition: "Given a tenor structure, for any two integers $k, m$ satisfying $0 \leq k < N$, $m > 0$, and $k + m \leq N$, we can define an annuity factor $A_{k,m}$ by $A_{k,m}(t) = \sum_{n=k}^{k+m-1} P(t, T_{n+1}) \tau_n$."

They further note that "the quantity $A(\cdot)$ is the annuity of the swap (or its PVBP, for Present Value of a Basis Point)." This equivalence between "annuity" and "PVBP" is important: the annuity measures both the PV of a 1% stream and the dollar sensitivity to a 1bp rate change (scaled appropriately).

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

1. **Par swap rate formula:** The par swap rate $K^*$ is the ratio of the discounted floating-leg payments to the annuity. Andersen and Piterbarg derive:
   $$K^* = \frac{P(t, T_k) - P(t, T_{k+m})}{A_{k,m}(t)} = \frac{\sum_{n=k}^{k+m-1} \tau_n P(t, T_{n+1}) L_n(t)}{A_{k,m}(t)}$$

2. **Swap PV formula:** For an off-market swap, Andersen and Piterbarg show that the value is simply:
   $$V_{\text{swap}}(t) = A(t)(S(t) - k)$$
   where $S(t)$ is the forward swap rate and $k$ is the contracted fixed rate.

3. **Swaption payoffs:** A payer swaption exercised at expiry is worth $N \times A \times \max(K^{\text{swap}} - K^{\text{strike}}, 0)$

4. **Fixed-rate PV01:** As we'll see, the sensitivity to the fixed rate is exactly $N \times A \times 0.0001$

---

## 26.3 PV01 to the Fixed Rate: The "Swap Market" Definition

### 26.3.1 What Tuckman Means by PV01

Tuckman states clearly: "Another measure of risk used in the swap markets is PV01, normally meant to mean the change in the value of a swap for a one-basis point change in its fixed rate."

This is a crucial distinction. In swap markets, "PV01" often refers to the sensitivity to the **contracted fixed rate $K$**, holding all market curves fixed. This is fundamentally different from a curve sensitivity (which measures how value changes when the **market** moves).

Tuckman adds: "Writing a bond pricing equation in terms of discount factors reveals that PV01 equals the sum of the discount factors used to discount the fixed cash flows."

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

**Setup:** Consider a 3-year annual-pay swap with $N = \$100\text{mm}$, accrual fractions $\alpha_i = 1$, and discount factors $P_d(0,1) = 0.98$, $P_d(0,2) = 0.955$, $P_d(0,3) = 0.93$.

**Step 1: Compute the annuity**

$$A = 1 \times 0.98 + 1 \times 0.955 + 1 \times 0.93 = 2.865$$

**Step 2: Compute PV01-to-$K$**

$$PV01_{K,\text{rec}} = 100,000,000 \times 2.865 \times 0.0001 = \$28,650$$

**Interpretation:** If you receive fixed, a 1bp increase in the fixed rate $K$ adds $28,650 to the swap's value. Equivalently, this swap's annuity PV01 is approximately $28,650 per bp.

**Sanity check:** The annuity of 2.865 is roughly the sum of 3 annual discount factors, each close to 0.95. ✓

---

## 26.4 Curve DV01: Sensitivity to Market Rate Changes

### 26.4.1 The Naming Collision

Here's where confusion often arises. Many risk systems report "PV01" or "DV01" to mean something entirely different: the change in swap value when the **yield curve** shifts by 1bp. This is a market sensitivity, not a contract-term sensitivity.

To be precise, we define:

- **PV01-to-$K$:** Sensitivity to the contracted fixed rate (holding curves fixed)
- **Curve PV01 or DV01:** Sensitivity to a curve bump (e.g., +1bp parallel shift)

The distinction matters enormously. Consider an at-market swap where $K = K^*$ and $PV \approx 0$. The PV01-to-$K$ is $N \times A \times 0.0001$—a substantial number. But the curve PV01 might be nearly zero if the parallel bump affects both legs similarly (though this depends on the exact bump methodology and whether we're in a single-curve or multi-curve world).

> **Desk Reality: Always Ask "Which PV01?"**
>
> Before using any risk report, confirm:
>
> | Question | Why It Matters |
> |----------|----------------|
> | Is PV01 defined vs the fixed rate $K$ or vs a curve bump? | Entirely different numbers and interpretations |
> | Which curve is bumped (discount, projection, or both)? | Multi-curve setups give different answers |
> | What's the bump rule (zero rates, par rates, instrument quotes)? | Affects the magnitude and distribution of sensitivities |
> | What's the sign convention (payer vs receiver)? | Getting the sign wrong flips the hedge direction |
>
> The first question you should ask when someone hands you a PV01 number: "PV01 of *what* with respect to *what*?"

### 26.4.2 Tuckman on Swap DV01

Tuckman observes that "strictly speaking, it is correct to say that the DV01 of a swap from the perspective of the fixed receiver is the DV01 of the fixed leg minus the DV01 of the floating leg. But this observation is not used much in practice."

Why not? Because "the DV01 of the fixed leg depends on the swap rate curve out to the maturity of the swap, whereas the DV01 of the floating leg depends on LIBOR out to the first payment date. Hence, adding these DV01 values for a 10-year swap, for example, mixes 10-year risk with three-month risk."

This insight is critical: **the fixed and floating legs have fundamentally different risk horizons**, and combining them into a single DV01 obscures rather than clarifies the risk profile.

### 26.4.3 Managing Fixed and Floating Risk Separately

Tuckman continues: "For this reason it is common in the industry to manage the interest rate risk of the fixed and floating legs separately. The fixed side of a swap is hedged with other swaps or bonds, and the floating side of a swap is hedged with Eurodollar or fed funds futures... or other short-term securities."

This separation reflects a key practical reality: the floating leg of a swap behaves like a floating-rate note with duration approximately equal to the time to the next reset (often just a few months), while the fixed leg has duration roughly equal to the swap's maturity.

### 26.4.4 The Floating Leg's Short Duration

Tuckman explains the floating leg's behavior through the valuation of floating-rate notes: "Despite the fact that the maturity of the floater is 10 years, the price of the floating rate note depends only on the short-term rate and its effect on the present value of the next payment date. Put another way, since the floating rate note is always worth par on set dates, it behaves like a zero coupon bond maturing on the next payment date. Hence... its duration is approximately equal to the time to the next payment date, that is, .25 years."

He continues: "Unlike the case of a fixed coupon bond, a change in interest rates does not affect the value of all the payments of a floater. The provision resetting each floating payment to reflect a fair market rate at the time of reset makes the value of later payments on the floater immune to changes in interest rates."

---

## 26.5 Multi-Curve Considerations: Discount vs Projection PV01

### 26.5.1 The Post-Crisis Framework

Before 2008, swaps were typically valued using a single curve: LIBOR discount factors were used for both discounting and forward rate projection. The financial crisis revealed that this approach conflated credit and funding considerations with rate dynamics.

Andersen and Piterbarg describe the industry evolution: the move to separate discount and projection curves became the post-crisis standard. Hull states the modern approach explicitly: "Assume that forward rates will be realized... then discount the resulting cash flows at the OIS... rate."

Under this framework:
- **Discount curve:** An OIS-based curve that reflects the risk-free rate for collateralized transactions
- **Projection curve:** A curve specific to the floating index (e.g., Term SOFR, legacy LIBOR) that determines expected floating payments

### 26.5.2 Splitting Risk by Curve

In multi-curve valuation, the swap PV depends on both curves:

$$PV_{\text{rec}} = N \sum_{i=1}^{n} (K - F_i) \alpha_i P_d(0, T_i)$$

where $F_i$ comes from the projection curve and $P_d(0, T_i)$ comes from the discount curve.

This creates two distinct sensitivities:

1. **Discount curve PV01:** Bump the discount curve (holding forward rates fixed) and measure the PV change. This primarily scales the present values of both legs.

2. **Projection curve PV01:** Bump the projection curve (holding discount factors fixed) and measure the PV change. This changes the expected floating payments.

Andersen and Piterbarg note that with a spread-based method of curve group construction, "sensitivities to instruments used in the curve group have clear, and orthogonal, meaning."

### 26.5.3 Worked Example: Discount vs Projection PV01

**Setup:** 3-year annual-pay payer swap, $N = \$100\text{mm}$, $K = 4\%$, forwards $F_1 = 3.0\%$, $F_2 = 3.4\%$, $F_3 = 3.8\%$, discount factors $P_1 = 0.98$, $P_2 = 0.955$, $P_3 = 0.93$.

**Base PV:**
$$\sum F_i \alpha_i P_i = 0.03 \times 0.98 + 0.034 \times 0.955 + 0.038 \times 0.93 = 0.09721$$
$$K \times A = 0.04 \times 2.865 = 0.1146$$
$$PV_{\text{pay}} = 100\text{mm} \times (0.09721 - 0.1146) = -\$1,739,000$$

**Discount curve bump (+1bp parallel to zero rates):**

Using the bump rule $P_d^b(0,T) = P_d(0,T) e^{-0.0001 \times T}$, the bumped discount factors are approximately $P_1^b = 0.979902$, $P_2^b = 0.954809$, $P_3^b = 0.929721$.

Holding forwards fixed:
$$PV_{\text{pay}}^{\text{bump,disc}} = 100\text{mm} \times (0.097190 - 0.114577) = -\$1,738,732$$

$$PV01_{\text{disc}} = -1,738,732 - (-1,739,000) = +\$268$$

**Projection curve bump (+1bp to each forward):**

With $F_i^b = F_i + 0.0001$:
$$\sum F_i^b P_i = 0.09721 + 0.0001 \times 2.865 = 0.09750$$
$$PV_{\text{pay}}^{\text{bump,proj}} = 100\text{mm} \times (0.09750 - 0.1146) = -\$1,710,350$$

$$PV01_{\text{proj}} = -1,710,350 - (-1,739,000) = +\$28,650$$

**Interpretation:** The projection curve PV01 ($28,650) is much larger than the discount curve PV01 ($268) for this payer swap. The projection bump directly changes expected floating receipts, while the discount bump merely rescales both legs' PVs.

**Sanity check:** The projection PV01 equals the annuity PV01, which makes sense—bumping all forward rates by 1bp increases the expected floating leg PV by exactly $N \times A \times 0.0001$. ✓

---

## 26.6 Bond DV01 vs Swap DV01

### 26.6.1 The Bond DV01 Definition

Tuckman defines DV01 as "the change in the value of a fixed income security for a one-basis point decline in rates," giving the formula:

$$\boxed{DV01 = \frac{1}{10,000} \left( -\frac{dP}{dy} \right)}$$

where $P$ is the bond price and $y$ is the yield. The factor of $1/10,000$ converts from a rate derivative to a "per basis point" measure, and the negative sign makes DV01 positive for a long position (since $dP/dy < 0$).

Equivalently, in terms of modified duration $D_{\text{mod}}$ and price $P$:

$$\boxed{DV01 = \frac{P \times D_{\text{mod}}}{10,000}}$$

### 26.6.2 Why Swap and Bond DV01 Differ

Even for a swap and bond with identical maturity, their DV01s can differ substantially:

1. **Cashflow timing:** A bond pays coupons plus principal at maturity; a swap's fixed leg pays only coupons (no principal exchange). The principal payment at maturity gives the bond more weight at the long end.

2. **Floating leg duration:** Tuckman emphasizes that "the DV01 of the floating leg depends on time to next payment and can be tiny compared to the fixed leg." A bond has no floating leg at all.

3. **Curve choice:** In multi-curve valuation, the swap's sensitivity depends on which curve you bump (discount, projection, or both). A bond's yield-based DV01 uses a single yield.

4. **Instrument-specific factors:** Bonds may have embedded optionality, special repo, or credit-spread dynamics that swaps lack.

### 26.6.3 Numerical Comparison

Consider a 5-year 3% annual coupon bond priced at 100.905 (per $100 face) versus a 5-year at-market payer swap.

**Bond DV01 (per $100 face):**

Under a +1bp bump to yields, the bond price falls to approximately 100.857.

$$DV01_{\text{bond}} = 100.905 - 100.857 = 0.0476 \text{ per \$100 face}$$

For $10mm face: $DV01 = \$4,758$ per bp.

**Swap "annuity PV01" (per $1mm notional):**

With annuity $A = 4.635$:

$$PV01_K = 1,000,000 \times 4.635 \times 0.0001 = \$463.5 \text{ per bp per \$1mm}$$

For $10mm notional: $PV01 = \$4,635$ per bp.

The bond DV01 ($4,758/bp) and swap annuity PV01 ($4,635/bp) are similar but not identical, reflecting the different cashflow structures.

---

## 26.7 Hedging with Swaps

### 26.7.1 The DV01 Hedge Ratio

Tuckman provides the fundamental hedging formula. To hedge position A with instrument B, choose position sizes such that first-order price changes offset:

$$\boxed{F_B = -F_A \frac{DV01_A}{DV01_B}}$$

Tuckman explains: "To avoid careless trading mistakes, it is worth emphasizing the simple implications of this equation, assuming that, as usually is the case, each DV01 is positive. First, hedging a long position in security A requires a short position in security B and hedging a short position in security A requires a long position in security B... Second, the security with the higher DV01 is traded in smaller quantity than the security with the lower DV01."

Hull notes this is also called the "duration-based hedge ratio" or "price sensitivity hedge ratio."

### 26.7.2 Worked Example: Hedging a Bond with a Swap

**Objective:** Hedge a $10mm face 5-year 3% bond using a 5-year payer swap.

**Bond characteristics:** DV01 = $4,758 per bp (long bond loses when rates rise).

**Swap characteristics:** Annuity PV01 = $463.5 per bp per $1mm notional.

**Hedge sizing:**

A long bond position loses money when rates rise. A payer swap (pay fixed, receive floating) gains when rates rise—it has negative duration exposure. Therefore, we need to go long the payer swap.

$$N_{\text{swap}} = \frac{DV01_{\text{bond}}}{PV01_{\text{swap per mm}}} = \frac{4,758}{463.5} \approx 10.27 \text{ mm}$$

**Result:** Hedge the $10mm bond with approximately $10.3mm payer swap notional to achieve DV01 neutrality under parallel shifts.

> **Technique: The Hedge Ratio Shortcut**
>
> Need a quick hedge ratio on the trading desk? Use the ratio of Durations.
>
> $$\text{Hedge Ratio} \approx \frac{\text{Duration}_{\text{Bond}}}{\text{Duration}_{\text{Swap}}}$$
>
> *   **Scenario**: Hedge \$100M of 10-Year Bonds (Duration $\approx$ 8.5) with 5-Year Swaps (Duration $\approx$ 4.5).
> *   **Ratio**: $8.5 / 4.5 \approx 1.9$.
> *   **Trade**: You need roughly $1.9 \times \$100M = \$190M$ of 5-year swaps.
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

$$\boxed{\text{Residual P&L} \approx DV01 \times (\Delta y_{\text{bond}} - \Delta y_{\text{swap}})}$$

If Treasury yields rise 10bp and swap rates rise 10bp, your hedged position is flat. But if Treasury yields rise 10bp and swap rates rise only 6bp (a 4bp spread widening), you lose money.

> **Desk Reality: "I'm Hedged" vs "Hedged Against What?"**
>
> When a portfolio manager says "I'm hedged," the first question should be: "Hedged against what?"
>
> - **Rate-hedged with swaps:** Protected against parallel rate moves; exposed to spread changes
> - **Spread-hedged:** Requires both rate hedge AND spread lock or offsetting spread position
> - **Fully hedged:** Rare; would require eliminating all systematic risk
>
> Most "hedged" positions in practice retain spread exposure. This is why P&L on seemingly hedged books can move significantly when swap spreads widen or tighten.

**Table: What a Swap Hedge Protects Against vs What It Doesn't**

| Risk Factor | Hedged by DV01 Match? | Explanation |
|-------------|----------------------|-------------|
| Parallel rate shift | Yes | Both legs move together |
| Curve twist (steepening/flattening) | Partially | Depends on tenor match |
| Swap spread change | No | Bond and swap respond differently |
| Convexity (large moves) | No | Second-order effect not captured |
| Credit spread of specific bond | No | Swap doesn't track issuer credit |

Tuckman confirms this risk: "Even if the correlation is high, substantial basis risk remains when hedging changes in corporate rates with swaps. First, swap rates reflect banking credit specifically rather than general corporate credits. Second, swap rates cannot possibly hedge the risk that the credit spread of a particular corporation might change relative to the general level of credit spreads."

---

## 26.8 Bucket Exposures for Swap Books

### 26.8.1 Why Bucket Analysis?

Tuckman explains that "the practice of accumulating swaps leads to large portfolios that change in composition only slowly." This makes it "reasonable to hedge against possible changes in many small segments of the term structure. While hedging against these many possible shifts requires many initial trades, the stability of the underlying portfolio composition assures that these hedges need not be adjusted very frequently."

Rather than a single DV01 number, bucket analysis computes sensitivity to each segment (or "bucket") of the forward rate curve. This reveals the **term structure of risk** embedded in a swap position.

### 26.8.2 The Mechanics of Bucket Shifts

Tuckman defines the approach: "A bucket is jargon for a region of some curve, like a term structure of interest rates. Bucket shifts are similar to key rate shifts but differ in two respects. First, bucket analysis usually uses very many buckets while key rate analysis tends to use a relatively small number of key rates. Second, each bucket shift is a parallel shift of forward rates as opposed to the shapes of the key rate shifts."

The procedure is straightforward:

1. Define buckets (e.g., 6-month forward rates at 0.5y, 1y, 1.5y, ...)
2. Bump each forward rate by 1bp while holding all others fixed
3. Recompute the swap PV
4. The bucket exposure is the PV change for that bucket

### 26.8.3 Tuckman's Example: 6-Year Par Swap

Tuckman illustrates bucket exposures for receiving fixed on $100mm of a 6% par swap with a flat 6% rate curve. He explains: "The graph shows, for example, that the exposure to the six-month rate 2.5 years forward is about $4,200."

The computation follows from repricing: "The original forward rate curve is flat at 6%, and the par swap, by definition, is priced at 100% of face amount. For the perturbed forward curve, the six-month rate 2.5 years forward is raised to 6.01%, and all other forwards are kept the same." After recomputing discount factors and repricing:

$$-\$100,000,000 \times (99.995813\% - 100\%) = \$4,187$$

Tuckman notes that "the sum of the bucket exposures, in this case $49,768, is the exposure of the swap to a simultaneous one-basis point change to all the forwards. If the swap rate curve is flat, as in this simple example, this sum exactly equals the DV01 of the fixed side of the swap."

### 26.8.4 Using Bucket Exposures

The sum of bucket exposures equals (approximately) the total DV01 under a parallel shift. But the granular view enables more precise hedging:

- **Futures hedging:** Historically Eurodollar futures, now SOFR futures, can hedge specific forward rate buckets directly
- **Curve trades** can be designed to neutralize exposure to specific curve segments
- **Risk limits** can be set by bucket rather than in aggregate

Tuckman notes: "As each Eurodollar futures contract is related to a particular three-month forward rate and as 10 years of these futures trade at all times, it is common to divide the first 10 years of exposure into three-month buckets."

> **Practitioner Note: SOFR Transition**
>
> With the phase-out of LIBOR and Eurodollar futures, SOFR futures have taken their place for bucket hedging. The principle remains identical: each SOFR futures contract corresponds to a specific forward rate period (typically 3-month segments), allowing bucket exposures to be hedged directly with futures.
>
> The granularity convention persists: 3-month buckets for the front 2-3 years (where SOFR futures are most liquid), transitioning to 6-month or 1-year buckets beyond.

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
> Tuckman confirms: "So while the portfolio has no risk with respect to parallel shifts of the forward curve, it can hardly be said that the portfolio has no interest rate risk. The portfolio will make money in a flattening of the forward curve... Conversely, the portfolio will lose money in a steepening."

### 26.8.6 Hedging Swap Books: Fixed Leg with Swaps, Floating with Futures

Given the different risk horizons of fixed and floating legs, large swap books are often hedged in two stages:

1. **Fixed-leg risk:** Hedged with offsetting swaps or bonds at the relevant maturities
2. **Floating-leg risk:** Hedged with short-term futures (SOFR futures) that target the specific forward rate buckets

This approach aligns with Tuckman's recommendation to manage fixed and floating risks separately rather than netting them into a single DV01.

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

- The zero bond gains as the 5-year rate falls: approximately +$43,610
- The swap loses because the annuity's PV weights extend across the curve, and the short end (where rates rose) pulls value down: approximately -$41,433

**Residual P&L:** +$2,177

This residual arises from "key-rate mismatch"—the instruments have different exposures across the term structure.

**Quantifying curve risk:** The P&L from a curve twist can be expressed in terms of key-rate exposures:

$$\text{P&L}_{\text{twist}} = \sum_i KR01_i^{\text{portfolio}} \times \Delta y_i$$

where $KR01_i$ is the key-rate exposure to tenor $i$ and $\Delta y_i$ is the rate change at that tenor. For a DV01-neutral portfolio, $\sum_i KR01_i = 0$, but individual $KR01_i$ values can be large with opposite signs—creating twist exposure.

### 26.9.2 Basis Risk

Swap hedges against Treasury positions leave exposure to swap spread changes. If Treasury yields rise 10bp but swap rates rise only 6bp (spread widening), a DV01-neutral hedge loses money:

- Bond loss: $-4,758 \times 10 = -\$47,580$
- Swap gain: $+4,758 \times 6 = +\$28,548$
- Residual: $-\$19,032$

This basis risk reflects that Treasury and swap markets embed different credit, liquidity, and supply/demand factors. Tuckman notes that "substantial basis risk remains when hedging changes in corporate rates with swaps."

**Worked Example: Basis Risk Under Spread Widening**

**Setup:** Long $10mm face 5-year Treasury (DV01 = $4,758), hedged with $10.3mm payer swap (DV01 = $4,758).

**Scenario:** Treasury yield rises 15bp, swap rate rises 10bp (5bp spread widening).

**P&L calculation:**
- Treasury loss: $-4,758 \times 15 = -\$71,370$
- Swap gain: $+4,758 \times 10 = +\$47,580$
- **Net loss: $-\$23,790$**

**Interpretation:** The portfolio was "hedged" against rate moves, but it lost money because the hedge only worked if Treasury and swap moved together. The 5bp spread widening cost nearly $24,000.

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

Tuckman explains: "Investors and traders long an on-the-run security for liquidity reasons require compensation to sacrifice liquidity by lending those securities in the repo market. At the same time, investors and traders wanting to short the on-the-run securities are willing to pay for the liquidity of shorting these securities when borrowing them in the repo market. As a result, the on-the-run securities tend to trade special in the repo market."

> **Practitioner Note: Why Swaps "Trade Rich" During Squeezes**
>
> When Treasuries become special or hard to short:
> - Shorting bonds becomes expensive → traders avoid it
> - Paying fixed on swaps achieves similar directional exposure → demand increases
> - Higher swap demand → swap rates fall relative to Treasury yields → swap spreads compress
>
> This dynamic explains some of the short-term volatility in swap spreads, particularly around month-end and quarter-end when balance sheet constraints bind.

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

Tuckman describes corporate swap usage: "A solution for these corporations is to issue fixed rate debt and then receive fixed and pay floating in a swap. The net effect of the fixed debt and the swap is floating rate debt."

But the flows don't stop with the corporate. Here's the full chain:

### 26.11.2 Step-by-Step: Corporate Issues, Bank Hedges

**Step 1: Corporate Issues Fixed Bond**

Company XYZ issues $500mm of 5-year fixed-rate bonds at 4.50% to institutional investors. The company now has fixed-rate debt: it pays 4.50% annually regardless of where rates go.

**Step 2: Corporate Enters Swap**

XYZ wants floating-rate exposure (perhaps it has floating-rate assets). It enters a payer swap with a bank:
- XYZ pays fixed 4.00% (the 5-year swap rate)
- XYZ receives 3-month SOFR

**Net result for XYZ:**
- Pays: 4.50% (bond) + 4.00% (swap fixed) - SOFR (swap floating) = 4.50% + 4.00% - SOFR
- Simplifies to: SOFR + 0.50% (effectively floating-rate debt at SOFR + 50bp)

**Step 3: Bank Hedges**

The bank is now receiving fixed, paying floating. This is equivalent to being long a bond—the bank loses if rates rise. To hedge, the bank typically:

- **Sells Treasuries** to offset the duration (or pays fixed on another swap)
- This Treasury selling puts downward pressure on Treasury prices / upward pressure on yields

**Step 4: Market Impact**

Heavy corporate issuance (followed by banks hedging) tends to:
- Increase Treasury supply → yields rise
- Increase swap receiving (banks hedge by receiving elsewhere) → swap rates may fall
- Net effect: swap spreads can compress during heavy issuance periods

> **Desk Reality: Corporate Issuance and Swap Spread Dynamics**
>
> When you see swap spreads tightening during a week of heavy investment-grade corporate issuance, this is often the mechanism:
> 1. Corporates issue → enter payer swaps
> 2. Banks receive fixed → hedge by selling Treasuries
> 3. Treasury selling → Treasury yields rise
> 4. Swap rates don't rise as much (different buyer base)
> 5. Spread = Swap - Treasury → compresses
>
> Tuckman's footnote mentions that "rather than pay fixed on a swap, a corporation may sell Treasuries to hedge changes in the level of rates and purchase a spread lock, which makes money if swap spreads widen, to hedge credit spreads."

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

Tuckman cautions: "Note that the DV01 values, quoted per 100 face value, must be divided by 100 before being multiplied by the face amount."

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

This chapter developed the risk measures for interest rate swaps, distinguishing between:

1. **PV01-to-fixed-rate** (the "swap market" definition): Sensitivity to the contracted fixed rate, equal to $N \times A \times 0.0001$ where $A$ is the annuity

2. **Curve PV01/DV01:** Sensitivity to market curve bumps, which in multi-curve frameworks splits into discount-curve and projection-curve sensitivities

3. **Bucket exposures:** Granular sensitivities to individual forward rate segments, enabling precise hedging with futures

The annuity factor $A = \sum \alpha_i P_d(0, T_i)$ is the central quantity for swap risk: it determines the PV01-to-fixed-rate, appears in the par swap rate formula, and drives swaption payoffs. Andersen and Piterbarg emphasize that this quantity is "the annuity of the swap (or its PVBP)."

DV01 hedging uses the ratio $F_B = -F_A(DV01_A/DV01_B)$ to match first-order exposures. This neutralizes parallel shift risk but **transforms rate risk into spread risk**—a critical distinction that risk reports often obscure. Basis moves, curve twists, and convexity effects can all cause "hedged" positions to move.

Tuckman's key insight—that fixed and floating leg risks should be managed separately because they have fundamentally different horizons—informs practical risk management. The fixed leg resembles a coupon bond with duration roughly equal to maturity; the floating leg, as Tuckman explains, "behaves like a zero coupon bond maturing on the next payment date" with duration equal to the time to next reset.

Operationally, paying fixed on a swap and shorting a bond serve similar economic purposes but differ dramatically in balance sheet treatment, repo exposure, and capital requirements. Understanding these differences explains why swap demand rises during repo squeezes and why corporate hedging flows affect swap spreads.

---

## Key Concepts Summary

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

## Notation for This Chapter

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
| 2 | What does Tuckman say "PV01" normally means in swap markets? | Change in swap value for 1bp change in the fixed rate $K$ |
| 3 | Formula for PV01-to-$K$ for a fixed receiver? | $PV01_{K,\text{rec}} = +N \times A \times 0.0001$ |
| 4 | What is $PV_{\text{rec}}$ in terms of leg values? | $PV_{\text{rec}} = V_{\text{fixed}} - V_{\text{float}}$ |
| 5 | Why doesn't Tuckman recommend using combined swap DV01? | It mixes long-term fixed-rate risk with short-term floating-rate risk |
| 6 | What's the duration of a floating leg approximately? | Time to next reset (often just a few months) |
| 7 | Bond DV01 formula (Tuckman)? | $DV01 = (1/10,000)(-dP/dy)$ |
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
| 19 | Par swap rate formula (simple form)? | $K^* = (P(t,T_k) - P(t,T_{k+m})) / A$ |
| 20 | Operational difference: paying fixed vs shorting bond? | Swap is off-balance sheet, no repo exposure; short bond requires borrowing, exposed to squeeze |

---

## Mini Problem Set

### Questions

1. **[Easy - Calculation]** Given a 4-year semiannual-pay swap with discount factors $P(0.5) = 0.985$, $P(1) = 0.97$, $P(1.5) = 0.955$, $P(2) = 0.94$, $P(2.5) = 0.925$, $P(3) = 0.91$, $P(3.5) = 0.895$, $P(4) = 0.88$, compute the annuity $A$ and PV01-to-$K$ for $N = \$50\text{mm}$.

2. **[Easy - Calculation]** With forwards $F_1 = 2.5\%$, $F_2 = 2.8\%$, $F_3 = 3.1\%$, $F_4 = 3.4\%$ (annual) and discount factors $P(1) = 0.975$, $P(2) = 0.95$, $P(3) = 0.92$, $P(4) = 0.89$, compute the par swap rate $K^*$.

3. **[Medium - Application]** For a payer swap with $K = 3.5\%$ (above par), using the data from problem 2, compute the swap PV per $\$100\text{mm}$ notional. Is it positive or negative to the payer?

4. **[Medium - Hedging]** A 10-year bond has DV01 = $8,500/bp. A 10-year swap has annuity PV01 = $750/bp per $1mm notional. What swap notional hedges the bond?

5. **[Medium - Scenario]** Explain why a DV01-neutral position can lose money under a curve steepening. Give a specific example.

6. **[Medium - Basis Risk]** A Treasury rises 12bp while the matched-maturity swap rate rises only 8bp. If you hedged $10mm Treasury (DV01 = $4,800) with a payer swap, compute the P&L.

7. **[Medium - Multi-curve]** In a multi-curve framework, if you bump only the discount curve by +1bp, what happens qualitatively to a payer swap's PV?

8. **[Medium - Corporate]** A corporation issues $200mm 5-year fixed bonds at 5.00% and enters a payer swap at 4.50%. What is the corporation's effective funding rate?

9. **[Hard - Floating Leg]** Why is the floating leg's DV01 approximately proportional to the time to next reset? What happens to the floating leg DV01 immediately after a rate fixing?

10. **[Hard - Debugging]** A par swap calculation shows PV = +$50,000 instead of zero. List three possible causes.

11. **[Hard - Bucket]** A 6-year receive-fixed swap is hedged with a 4-year pay-fixed swap to zero parallel DV01. The curve then steepens (short rates up, long rates down). Does the portfolio gain or lose?

12. **[Hard - Integration]** Explain why swaps are preferred over bonds for systematic factor (PCA) hedging. What bond-specific issues do swaps avoid?

---

### Solution Sketches (Problems 1-6)

**1.** Annuity $A = \sum_{i=1}^{8} 0.5 \times P(0.5i) = 0.5 \times (0.985 + 0.97 + 0.955 + 0.94 + 0.925 + 0.91 + 0.895 + 0.88) = 0.5 \times 7.46 = 3.73$.

$PV01_K = 50\text{mm} \times 3.73 \times 0.0001 = \$18,650$.

**2.** Numerator: $\sum F_i \times 1 \times P_i = 0.025 \times 0.975 + 0.028 \times 0.95 + 0.031 \times 0.92 + 0.034 \times 0.89 = 0.02438 + 0.0266 + 0.02852 + 0.03026 = 0.10976$.

Annuity: $A = 0.975 + 0.95 + 0.92 + 0.89 = 3.735$.

$K^* = 0.10976 / 3.735 = 2.94\%$.

**3.** $PV_{\text{pay}} = N(\sum F_i P_i - KA) = N(0.10976 - 0.035 \times 3.735) = N(0.10976 - 0.13073) = -0.02097 \times N$.

For $N = 100\text{mm}$: $PV_{\text{pay}} = -\$2,097,000$ (negative to payer because they pay above-market fixed rate).

**4.** $N_{\text{swap}} = DV01_{\text{bond}} / PV01_{\text{swap per mm}} = 8,500 / 750 = 11.33\text{mm}$. Enter a payer swap to offset the long-bond exposure.

**5.** A DV01-neutral portfolio has offsetting exposures that sum to zero for parallel shifts but may have opposite signs across tenors. In a steepening (short rates up, long rates down):
- If you're long short-end risk (positive bucket exposure at short end) and short long-end risk (negative bucket exposure at long end), you lose on both: the short end rises (your long position loses) and the long end falls (your short position gains, but you're short, so the falling rates hurt you).
- Example: Receive fixed 10Y, pay fixed 2Y sized to zero DV01. Steepening: 2Y rate +20bp, 10Y rate -10bp. The 2Y payer leg gains (good), but the 10Y receiver leg loses more because it has higher weight at the now-more-valuable long end.

**6.**
- Treasury loss: $-4,800 \times 12 = -\$57,600$
- Swap gain: $+4,800 \times 8 = +\$38,400$ (payer swap gains when rates rise)
- **Net loss: $-\$19,200$** (the 4bp spread widening caused the loss)

---

## References

- Tuckman & Serrat, *Fixed Income Securities* (swap valuation; DV01/PV01; swap spreads; repo specialness)
- Andersen & Piterbarg, *Interest Rate Modeling* (swap annuity/PVBP; multi-curve framework)
- Hull, *Options, Futures, and Other Derivatives* (multi-curve discounting overview; hedging intuition)
