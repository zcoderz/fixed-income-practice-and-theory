# Chapter 19: Projection Curves — LIBOR, SOFR, and the Multi-Curve Framework

---

## Introduction

For decades, the fixed income market operated on a beautifully simple assumption: there was only one yield curve. Whether you were pricing a government bond, a corporate loan, or an interest rate swap, the fundamental "time value of money" was derived from a single set of inputs—typically the LIBOR interbank curve. This framework relied on the premise that major financial institutions lent to each other at rates that were effectively risk-free, and that a 3-month loan could be seamlessly replicated by rolling three 1-month loans.

That assumption died in August 2007.

As the Global Financial Crisis unfolded, the "law of one price" for money broke down. Banks became wary of lending to one another, causing the spread between secured overnight rates (like the Fed Funds rate) and unsecured term rates (like LIBOR) to explode. Andersen and Piterbarg document this shift precisely: "While the spread between the Fed funds rate and 3 month Libor rate used to be very small—in the order of a few basis points—after September 2007 it went up to as much as **275 basis points**." Simultaneously, the tenor basis—the difference in pricing between a swap referencing 1-month LIBOR and one referencing 3-month LIBOR—widened from near-zero to dramatic levels: "the difference between 1 month and 3 month Libor rates was in the order of one basis point up until September 2007, but since then has been as wide as **50 basis points**."

Practitioners who clung to the single-curve model found themselves systematically mispricing risks. A trader valuing a 6-month floating-rate note using a 3-month curve would overestimate its value; a risk manager hedging a 3-month exposure with a 1-month instrument would leave a massive "basis risk" exposure completely unhedged.

This chapter details the **multi-curve framework** that rose from the ashes of the single-curve world. This is not merely an optional refinement; it is the absolute standard for modern interest rate modeling. We will explore:

1. **The Divorce of Discounting and Projection:** Why we now use one curve to value cash flows (Discounting) and completely separate curves to forecast them (Projection).
2. **The Tenor Basis:** Why a 1-month rate and a 3-month rate are no longer just different points on the same curve, but fundamentally different assets with their own supply and demand dynamics.
3. **Valuation Mechanics:** How to price swaps and floating rate notes when the curve that determines the coupon is different from the curve that determines its present value.
4. **The Risk Management Implications:** How "delta" splits into "discount risk" (sensitivity to the OIS curve) and "projection risk" (sensitivity to the LIBOR/SOFR basis).

---

## 19.1 The "Single Curve" Fallacy

To understand why the market moved to multiple curves, we must first appreciate why the old model failed. In the pre-2007 world, a trader looking at a 3-month Forward Rate Agreement (FRA) and an Overnight Index Swap (OIS) saw them as two instruments telling the same story about the future path of interest rates.

### 19.1.1 The Pre-Crisis Simplicity

Consider a simplified market scenario where the overnight risk-free rate is flat at 2.00%. In a frictionless single-curve world, the 3-month forward rate should simply mathematically compound this 2.00% rate. The discount factor required to price an instrument at par would be consistent across all markets.

However, suppose quotes in the FRA market imply a 3-month forward rate of **3.40%**, significantly higher than the 2.00% implied by the overnight OIS market. This 140 basis point difference represents the **credit and liquidity premium** banks charge for unsecured term lending.

### 19.1.2 The Impossible Contradiction

If a quant tried to fit a **single** yield curve to this market, they would face an impossible contradiction:

* **The OIS Market** says the discount factor for a cash flow in 6 months ($T=0.50$) should be derived from the 2.00% rate: $P(0, 0.50) \approx 0.9900$.
* **The FRA Market** says the forward rate is 3.40%. For the forward rate to be 3.40% when the 3-month rate is 2.00%, the 6-month discount factor would need to be much lower, roughly $0.9866$, to "justify" the high forward rate.

One curve cannot simultaneously pass through $0.9900$ and $0.9866$ at $T=0.50$. The classic "no-arbitrage" relation $P(T_2) = P(T_1) / (1 + \tau F)$ holds mathematically, but it requires $P$ and $F$ to come from the *same* underlying economic reality. When OIS and LIBOR represent different credit risks, they cannot be described by a single function.

### 19.1.3 The Resolution

This contradiction forced the market to adopt a **dual-curve approach**: use the risk-free (OIS) curve to determine the *present value* of money (Discounting), and use specific market curves (LIBOR 3M, EURIBOR 6M, etc.) to estimate *future rate fixings* (Projection).

As Andersen and Piterbarg explain: "it is now generally accepted that the Libor rate is no longer a good proxy for a discounting rate on collateralized trades."

> **Analogy: The Two Watches**
>
> In the old world, we had one watch. It told us the time (Discounting) and we used that same time to schedule our meetings (Projection).
>
> In the new world, we wear two watches:
> *   **Watch 1 (OIS - The Gold Watch)**: This tells the "Value of Time." It determines the Present Value of money today. It is Risk-Free.
> *   **Watch 2 (LIBOR - The Weather Watch)**: This tells us "How much it will rain" (Forward Rates). It determines how much the floating leg pays.
>
> You naturally check Watch 2 to see *how much* to pay, and Watch 1 to see *what it's worth* today. Mixing them up leads to disaster.

---

## 19.2 The Multi-Curve Valuation Framework

In the modern framework, we define a "universal" discount curve and a family of "index-specific" projection curves. Andersen and Piterbarg formally define this as a **multi-index curve group**: "a collection $\{P(\cdot), P^1(\cdot), \ldots, P^K(\cdot)\}$ of the universal discounting curve and one index curve per tenor."

### 19.2.1 The Roles of the Curves

**1. The Discount Curve ($P_d$)**

This curve answers the question: *"What is a risk-free dollar at time $T$ worth today?"*

For collateralized derivatives (which constitute the vast majority of the market), the discount curve is constructed from **Overnight Index Swaps (OIS)**. Andersen and Piterbarg provide the rationale: "most inter-dealer transactions are collateralized under the International Swaps and Derivatives Association (ISDA) Master Agreement, with the rate paid on collateral being the Fed funds rate (for USD; Eonia and Sonia for Euro and GBP)."

Hull similarly explains that when a derivative is fully collateralized by cash earning the overnight rate, the funding cost for that position is the overnight rate. To avoid arbitrage, we must therefore discount at OIS.

**2. The Projection Curve ($P_k$)**

This curve answers the question: *"What will the floating index $k$ fix at in the future?"*

There is not just one projection curve. There is a specific curve for **3-Month LIBOR**, another for **6-Month EURIBOR**, another for **1-Month SOFR**, and so on. These curves are often constructed from "pseudo-discount factors" $P_k(0,T)$, which are mathematical constructs designed to reproduce forward rates via the standard formula:

$$\boxed{F_k(0; T, T+\tau) = \frac{1}{\tau} \left( \frac{P_k(0, T)}{P_k(0, T+\tau)} - 1 \right)}$$

> **Important Distinction:** The pseudo-discount factors $P_k$ are not real discount factors—you cannot buy a zero-coupon bond with these prices. They are mathematical devices to ensure forwards price correctly.

> **Deep Dive: Separation of Powers**
>
> *   **Forward Curve (Projection)**: Determines the *Cash Flows*. "The coupon will be $3.50."
> *   **Discount Curve (Valuation)**: Determines the *Weight*. "$3.50 in 5 years is worth $3.00 today."
>
> You can change the Projection Curve without changing the Discount Curve (e.g., if credit risk rises, coupons go up, but the value of a risk-free dollar today stays the same).

### 19.2.2 Valuation of a Floating Rate Note (FRN): The Par Floater Paradox

Let's apply this framework to a basic instrument: a Floating Rate Note (FRN) that pays a coupon linked to 3-Month LIBOR. In the single-curve world, a par-floater always prices exactly at par ($\$100$). In the multi-curve world, this intuitive rule breaks down completely.

**Example: 1-Year Quarterly FRN**

Consider a 1-year FRN with quarterly coupons:
* **Notional:** $N = \$100$
* **Discount Curve ($P_d$):** Flat at 2.00% (OIS).
* **Projection Curve ($F_{3M}$):** The market forward rates for 3M LIBOR are 3.20%, 3.40%, 3.50%, and 3.60%.

To value this, we use the **Projection Curve** to estimate the cash flows and the **Discount Curve** to present-value them.

**Step 1: Project Cash Flows**

The coupons are $N \times 0.25 \times F_{3M}$:
* Q1: $100 \times 0.25 \times 3.20\% = \$0.800$
* Q2: $100 \times 0.25 \times 3.40\% = \$0.850$
* Q3: $100 \times 0.25 \times 3.50\% = \$0.875$
* Q4: $100 \times 0.25 \times 3.60\% = \$0.900$

**Step 2: Discount to Present Value**

We discount these amounts using the OIS factors (derived from 2.00%):

$$PV_{cpn} = 0.80(0.9950) + 0.85(0.9901) + 0.875(0.9851) + 0.90(0.9802) \approx \$3.38$$
$$PV_{prin} = 100(0.9802) \approx \$98.02$$

$$\boxed{\text{Total PV} = \$101.40}$$

**The Result:** The FRN prices at **101.40**, not 100.00.

Why? Because the coupons are "rich"—they are set by the higher risky curve (3.20%–3.60%)—but they are discounted by the lower risk-free curve (2.00%). The difference represents the present value of the credit premium embedded in the LIBOR rates.

> **The Par Floater Paradox:** A LIBOR-linked FRN discounted at OIS will trade *above* par when LIBOR > OIS. This violation of "par floater = par" is a defining feature of the multi-curve world.

> **The 101 Par Floater**
>
> *   **Classic Rule**: A bond paying floating LIBOR is always worth $100.
> *   **Why it broke**: The rule assumed you discounted at LIBOR.
> *   **New Math**:
>     *   You receive huge LIBOR coupons (e.g., 3.5%).
>     *   You discount at tiny OIS rates (e.g., 0.1%).
>     *   Result: PV > 100.
>     *   It's like buying a high-yield bond but valuing it as if it were a Treasury. Of course it looks worth more than par!

### 19.2.3 Valuation of an Interest Rate Swap

The same logic applies to Interest Rate Swaps (IRS). The par swap rate $K_{par}$ is the fixed rate that equates the PV of the fixed leg to the PV of the floating leg.

$$\boxed{K_{par} = \frac{\sum_{i} \tau_i F_k(T_{i-1}, T_i) P_d(0, T_i)}{\sum_{i} \alpha_i P_d(0, T_i)}}$$

Notice the mix of curves: the numerator uses $F_k$ (Projection) to forecast floating payments and $P_d$ (Discount) to value them. The denominator uses $P_d$ (Discount) to value the fixed annuity.

**Example Calculation:**

Using our numbers above, the "Projection PV" of the floating leg was $\$3.38$. The "Discount Annuity" (sum of OIS discount factors $\times$ 0.25) is approximately $0.9876$.

$$K_{par} \approx \frac{3.38}{100 \times 0.9876} \approx 3.42\%$$

This par rate of **3.42%** is effectively a weighted average of the high projection forwards (3.2%–3.6%), not the low OIS rates.

---

## 19.3 The Tenor Basis

So far we have distinguished between "Risk-Free" (OIS) and "Risky" (LIBOR). But "Risky" is not a monolith. In the post-crisis world, the market began to brutally distinguish between different *tenors* of risk.

### 19.3.1 1-Month vs 3-Month Risk

Pre-2007, a bank needing 3-month funding could comfortably assume that borrowing for 1 month and rolling the loan twice was roughly equivalent to borrowing for 3 months once. The market priced these nearly identically.

Andersen and Piterbarg document the historical baseline: "the difference between 1 month and 3 month Libor rates was in the order of one basis point up until September 2007."

During the liquidity crunch, this logic evaporated. A 3-month loan locks up liquidity for 90 days; a 1-month loan returns cash in 30 days. In a stressed market, that 60-day difference is valuable. Banks demanded a higher premium for the longer lock-up. Andersen and Piterbarg continue: "since then has been as wide as 50 basis points."

This phenomenon is known as **Tenor Basis**. It implies that we cannot use the 3-Month LIBOR curve to forecast 1-Month or 6-Month rates. **Each tenor requires its own projection curve.**

### 19.3.2 Economic Drivers

The tenor basis is driven by several factors, as explained by Andersen and Piterbarg:

1. **Credit Horizon:** A 6-month loan embeds more default risk than a 1-month loan.
2. **Liquidity Preference:** "Banks have a natural desire to have longer-term deposits to better match their loan commitments."
3. **Market Segmentation:** Different investors participate in different tenor markets.

### 19.3.3 The Basis Swap: Linking the Curves

> **Visual: The Fan Chart**
>
> Imagine the yield curve as a fan opening up.
> *   **Base Line (Bottom)**: OIS Curve (Risk Free).
> *   **First Blade**: 1-Month LIBOR (Slightly higher risk).
> *   **Second Blade**: 3-Month LIBOR (More risk).
> *   **Top Blade**: 6-Month LIBOR (Highest risk).
>
> The "Tenor Basis" is the space between the blades. In times of stress, the fan opens wide. In calm times, the fan closes up.

How do we construct these distinct curves? We cannot just bootstrap them independently, or they might cross in arbitrary ways. The market instrument that links them is the **Tenor Basis Swap**.

A Basis Swap is a floating-for-floating swap. For example, a "3M vs 6M Basis Swap" might exchange:
* **Leg 1:** Pay 3-Month LIBOR quarterly.
* **Leg 2:** Receive 6-Month LIBOR semi-annually **plus a spread**.

Andersen and Piterbarg provide the par condition (equation 6.49):

$$\boxed{\sum_{i=0}^{n^{2}(T)-1} L^{2}(0, t_i^2, t_{i+1}^2) \tau_i^2 P(t_{i+1}^2) = \sum_{i=0}^{n^{1}(T)-1} \left(L^{1}(0, t_i^1, t_{i+1}^1) + e^{1,2}(T)\right) \tau_i^1 P(t_{i+1}^1)}$$

Here $e^{1,2}(T)$ is the quoted floating-floating basis spread, "quoted on the $L^1$ leg. It could be positive or negative, depending on perceived desirability of payments linked to $L^1$ versus $L^2$."

**Example: Calibrating a 6M Curve**

Suppose we have already built our OIS (Discount) curve and our 3M LIBOR (Base Projection) curve. We now see a 1-year Basis Swap exchanging 3M LIBOR + 10bps for 6M LIBOR.

1. **Calculate PV of Leg 1 (3M + 10bps):** Value the 3M forwards using our known 3M curve, add 10bps to each, and discount at OIS. Let's say this equals $\$0.0348$.
2. **Solve for Leg 2 (6M):** The PV of the 6M leg must also equal $\$0.0348$.
   $$ PV_{6M} = \sum F_{6M} \times P_d(0, T) = 0.0348 $$
   The only unknown is $F_{6M}$. We solve for the 6-month forward rate that satisfies this equation.

Andersen and Piterbarg describe this "spread-based" construction: "each index curve $P^k(\cdot)$ for $k>1$ is built as a spread, or basis, curve to one of the previous curves."

---

## 19.4 The Transition to SOFR

The principles of the multi-curve framework—separating discounting from projection—remain perfectly valid in the new world of Risk-Free Rates (RFRs) like SOFR (Secured Overnight Financing Rate). However, SOFR introduces a new mechanical wrinkle: it is **backward-looking**.

### 19.4.1 Forward-Looking vs Backward-Looking Rates

Hull explains the key difference: "LIBOR rates are forward looking. They are determined at the beginning of the period to which they will apply. The new reference rates are backward looking. The rate applicable to a particular period is not known until the end of the period when all the relevant overnight rates have been observed."

For SOFR specifically, Hull describes the compounding formula: "Longer rates such as three-month rates, six-month rates, or one-year rates can be determined from overnight rates by compounding them daily."

The annualized interest rate for a period is:

$$\boxed{\left[\prod_{i=1}^{n}\left(1+r_{i} \hat{d}_{i}\right)-1\right] \times \frac{360}{D}}$$

where $r_i$ is the overnight SOFR on day $i$, $\hat{d}_i = d_i/360$ is the day fraction, and $D = \sum_i d_i$ is the total days in the period.

### 19.4.2 SOFR Projection Curves

If SOFR is just an overnight rate, do we still need a projection curve?

Yes. To value a swap that pays "Compounded SOFR" in the future, we need to estimate what that compounded rate will be. The market trades **SOFR Futures** and **SOFR OIS** which allow us to build a forward curve for SOFR.

Hull notes: "Overnight indexed swaps play an important role in determining the risk-free rates which are needed for valuing derivatives."

### 19.4.3 The Return of the Single Curve?

Interestingly, for a standard SOFR OIS swap (where we pay fixed and receive compounded SOFR), the "Projection" curve and the "Discount" curve are conceptually the same (both are SOFR). In this specific corner of the market, the "single curve" world has effectively returned.

However, as soon as we trade a **BSBY** swap, or a **Term SOFR** swap, or a legacy **LIBOR** instrument, the multi-curve distinction comes roaring back. The modern desk must handle both regimes simultaneously.

---

## 19.5 Managing Risk in a Multi-Curve World

The shift to multiple curves complicates risk management. The simple question "what is my delta?" now has two answers.

### 19.5.1 Discount Risk vs. Projection Risk

Consider the 1-year Swap from our earlier example (Receive Fix 3.42% / Pay Float).

* **Discount Risk:** If the **OIS curve** shifts up by 1 basis point, the present value of the fixed leg decreases (standard duration effect). The floating leg PV also changes slightly because the *weights* on the cash flows change.
* **Projection Risk:** If the **3M LIBOR curve** shifts up by 1 basis point, the *amount* of the floating coupons increases. This is a direct hit to the floating leg value.

**Example Sensitivity Comparison:**

For our sample trade:
* **Bump OIS +1bp:** $\Delta PV \approx 0$ (The trade is par, so sensitivity to the discount rate is minimal).
* **Bump 3M LIBOR +1bp:** $\Delta PV \approx -10$ bps (We are paying floating, so higher rates hurt us directly).

A risk manager who lumps these together into a single "Interest Rate Delta" would miss the fact that the desk is essentially neutral to Fed Funds rate moves but heavily short the 3M LIBOR basis. These risks must be hedged with different instruments: OIS swaps for discount risk, and FRAs/Eurodollar futures for projection risk.

### 19.5.2 Orthogonality in Risk Sensitivities

A robust curve construction strives for **orthogonality**. Andersen and Piterbarg describe the risk decomposition:

* "Perturbations to instruments used in building the base index curve... define risk sensitivities to the overall levels of interest rates."
* "Perturbations to funding instruments define sensitivities to discounting."
* "Perturbations to basis swap spreads for $L^k$-versus-$L^1$ floating-floating basis swaps define basis risk."

This parameterization "allows us to naturally aggregate 'similar' risks such as overall rate level risks, discounting risks, basis risks, while keeping different kinds separate for efficient risk management."

---

## 19.6 Practical Notes

### 19.6.1 Curve Construction Hierarchy

Build curves in order of liquidity:
1. **OIS (Discount):** From OIS swaps and Fed Funds/SOFR futures
2. **Base Projection (3M LIBOR or SOFR):** From vanilla fixed-float swaps
3. **Secondary Tenors (1M, 6M):** From basis swaps as spreads to the base

### 19.6.2 Common Pitfalls

1. **Using the Wrong Curve for Discounting:** Legacy LIBOR systems may still discount at LIBOR. This creates P&L mismatches.
2. **Ignoring Tenor Basis:** Hedging a 6M exposure with 3M instruments leaves residual basis risk.
3. **Confusing Pseudo-Discount Factors:** The projection curve's $P_k$ values are not real prices—don't try to buy bonds at those yields.

### 19.6.3 Implementation Checklist

* **Repricing Check:** After building curves, reprice every input instrument. Errors should be $< 10^{-10}$.
* **Orthogonality Test:** Bump basis spreads and verify only projection curves move.
* **Convention Match:** Ensure day counts (ACT/360 vs ACT/365) match the specific index.

---

## Summary

1. **Discount at the Collateral Rate:** Value all cash flows using the OIS curve (assuming standard CSA).
2. **Project at the Index Rate:** Forecast floating coupons using the specific curve for that index (3M, 6M, etc.).
3. **Respect the Basis:** Never assume 3-month rates are just compounded 1-month rates. The spread (tenor basis) is real and volatile.
4. **Par Floaters Aren't Par:** A LIBOR FRN discounted at OIS will trade above par if LIBOR > OIS.
5. **Decompose Your Risk:** Always distinguish between sensitivity to the discount curve and sensitivity to the projection curve.
6. **Sequential Construction:** Build OIS first, then base projection, then secondary tenors via basis swaps.

The multi-curve framework is more complex, certainly. But it is also more honest. It forces us to confront the reality that liquidity has a price, and that in the financial markets, not all dollars are created equal.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Discount Curve** | The OIS curve used to present-value all cash flows | Reflects the true cost of funding collateralized trades |
| **Projection Curve** | Tenor-specific curve (e.g., 3M LIBOR) used to forecast floating fixings | Ensures forward rates match market FRA/swap quotes |
| **Multi-Index Curve Group** | Collection of one discount + multiple projection curves | The only consistent framework for post-crisis pricing |
| **Tenor Basis** | Spread between forward rates of different tenors | Represents credit/liquidity risk differences by funding horizon |
| **Basis Swap** | Floating-for-floating swap on different tenors | The instrument that calibrates and hedges tenor basis |
| **Par Floater Paradox** | LIBOR FRN trades above par when LIBOR > OIS | Classic manifestation of the two-curve divergence |
| **Orthogonality** | Discount and projection risks are independent | Enables clean hedging and risk attribution |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $P_d(t,T)$ | Discount factor from the OIS/collateral curve |
| $P_k(t,T)$ | Pseudo-discount factor for projection curve $k$ |
| $F_k(t; T_1, T_2)$ | Forward rate for tenor $k$ over $[T_1, T_2]$ |
| $e^{1,2}(T)$ | Par basis spread between tenors 1 and 2 at maturity $T$ |
| $K_{par}$ | Par swap rate (makes swap PV = 0 at inception) |
| $\tau, \alpha$ | Year fractions for floating and fixed legs |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Why did the single-curve framework fail after 2007? | The spread between OIS and LIBOR widened to 275 bps, revealing that they represent different credit risks. |
| 2 | What is the discount curve used for? | Present-valuing all future cash flows (using OIS for collateralized trades). |
| 3 | What is the projection curve used for? | Forecasting future floating rate fixings for a specific index (e.g., 3M LIBOR). |
| 4 | What is a "multi-index curve group"? | A collection of one discount curve and multiple projection curves, one per tenor. |
| 5 | Why does a LIBOR FRN not price at par when discounted at OIS? | The coupons are set at the higher LIBOR rate but discounted at the lower OIS rate, creating PV > par. |
| 6 | What is tenor basis? | The spread between forward rates of different tenors (e.g., 1M vs 3M), reflecting credit/liquidity differences. |
| 7 | How wide did the 1M–3M tenor basis get during the 2008 crisis? | Up to 50 basis points (from ~1 bp pre-crisis). |
| 8 | What is a basis swap? | A floating-for-floating swap that exchanges payments based on two different tenors. |
| 9 | What is the par basis spread? | The spread added to one leg of a basis swap to make its inception value zero. |
| 10 | What is "discount risk" vs "projection risk"? | Discount risk = sensitivity to OIS curve; Projection risk = sensitivity to the specific index curve. |
| 11 | Why is orthogonality important in curve construction? | It ensures that bumping the basis doesn't affect the discount curve, enabling clean hedging. |
| 12 | What is the key difference between LIBOR and SOFR? | LIBOR is forward-looking (set at period start); SOFR is backward-looking (compounded over period). |
| 13 | How do you build a 6M projection curve? | Use basis swaps (3M vs 6M) to define the 6M curve as a spread over the 3M curve. |
| 14 | What is the par swap rate formula in multi-curve? | $K_{par} = \sum F_k P_d / \sum \tau P_d$ (project at index rate, discount at OIS). |
| 15 | When does the single-curve framework still apply? | For SOFR OIS swaps, where discount and projection are both the SOFR curve. |

---

## Mini Problem Set

### Problem 1 (Basic)

You observe the following for a 6-month period:
- OIS discount factor: $P_d(0, 0.5) = 0.9900$
- 3M LIBOR projection pseudo-factors: $P_{3M}(0, 0.25) = 0.9940$, $P_{3M}(0, 0.5) = 0.9870$

Calculate the 3M forward rate for the period $[0.25, 0.5]$.

**Solution:**
$$F = \frac{1}{0.25}\left(\frac{0.9940}{0.9870} - 1\right) = \frac{1}{0.25}(1.00709 - 1) = 2.84\%$$

---

### Problem 2 (Basic)

An FRN pays 3M LIBOR quarterly for 1 year. The four quarterly 3M forwards are 3.0%, 3.1%, 3.2%, 3.3%. The OIS curve is flat at 2.5%. Calculate the FRN's present value per $100 notional.

**Solution:**
OIS discount factors (approximate, flat curve): $P(0.25) = 0.9938$, $P(0.5) = 0.9876$, $P(0.75) = 0.9815$, $P(1.0) = 0.9754$.

Coupon PVs:
- Q1: $100 \times 0.25 \times 0.030 \times 0.9938 = 0.744$
- Q2: $100 \times 0.25 \times 0.031 \times 0.9876 = 0.766$
- Q3: $100 \times 0.25 \times 0.032 \times 0.9815 = 0.785$
- Q4: $100 \times 0.25 \times 0.033 \times 0.9754 = 0.805$

Principal PV: $100 \times 0.9754 = 97.54$

**Total PV = 0.744 + 0.766 + 0.785 + 0.805 + 97.54 = $100.64$**

The FRN trades above par because LIBOR > OIS.

---

### Problem 3 (Intermediate)

A 1-year basis swap exchanges 6M LIBOR against 3M LIBOR + spread.
- PV of 6M leg (2 payments): $1.62\%$ of notional
- PV of flat 3M leg (4 payments): $1.52\%$ of notional
- PV01 of 3M leg: $0.98\%$ per 100 bps

What is the par basis spread?

**Solution:**
$$e = \frac{PV_{6M} - PV_{3M}}{PV01_{3M}} = \frac{1.62\% - 1.52\%}{0.98\%/100} = \frac{0.10\%}{0.0098} = 10.2 \text{ bps}$$

The 3M leg pays +10.2 bps over flat 3M LIBOR.

---

### Problem 4 (Intermediate)

You hold a swap: receive fixed 3.50%, pay 3M LIBOR.
- DV01 to OIS curve: +$8,500 per 1 bp
- DV01 to 3M LIBOR curve: -$9,200 per 1 bp

If OIS rises 5 bps and 3M LIBOR rises 8 bps, what is your approximate P&L?

**Solution:**
$$\Delta PV = (+8500)(+5) + (-9200)(+8) = 42,500 - 73,600 = -\$31,100$$

You lose $31,100. The projection risk dominates because 3M LIBOR moved more.

---

### Problem 5 (Advanced)

Explain mathematically why a floating rate note paying index $k$ does not trade at par when discounted at OIS.

**Solution Sketch:**

In the single-curve world, the PV of the floating leg telescopes:
$$PV_{float} = \sum_i \tau_i F P(T_i) = \sum_i \left[\frac{P(T_{i-1}) - P(T_i)}{P(T_i)}\right] P(T_i) = \sum_i [P(T_{i-1}) - P(T_i)] = 1 - P(T_n)$$

Adding principal: $PV_{FRN} = 1 - P(T_n) + P(T_n) = 1$ (par).

In multi-curve: $F_k$ is computed from $P_k$, but discounting uses $P_d$. The telescoping fails because:
$$PV_{float} = \sum_i \tau_i F_k P_d(T_i) \neq 1 - P_d(T_n)$$

When $F_k > F_d$ (LIBOR > OIS), the coupons are "rich" relative to the discount rate, so $PV > 1$.

---

### Problem 6 (Advanced)

The OIS curve is flat at 2.00%. The 3M LIBOR projection curve implies forwards averaging 2.80%. A trader prices a 5-year swap using the single (LIBOR) curve for both discounting and projection. What error does this introduce, and in which direction?

**Solution Sketch:**

Using LIBOR for discounting (at 2.80%) instead of OIS (at 2.00%) means:
- Discount factors are too low (higher rate → lower DF)
- PVs of all legs are understated

For a receiver swap (receive fixed, pay floating):
- The fixed leg PV is understated
- The floating leg PV is also understated, but by about the same amount (since the swap is approximately par)

The net effect depends on the direction of the swap. Generally, using LIBOR discounting when you should use OIS:
- **Understates** the value of receiving fixed (you're discounting too heavily)
- **Overstates** the value of paying fixed

The magnitude can be significant: for a 5-year swap with notional $100M and an 80 bp OIS-LIBOR spread, the mispricing can exceed $300,000.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| Fed Funds-LIBOR spread widened to 275 bps post-2007 | Andersen & Piterbarg Vol 1, §6.5.3 |
| 1M-3M tenor basis widened from ~1 bp to 50 bps | Andersen & Piterbarg Vol 1, §6.5.3 |
| OIS discounting for collateralized trades | Andersen & Piterbarg Vol 1, §6.5.3; Hull Ch 4, Ch 7 |
| Definition of multi-index curve group | Andersen & Piterbarg Vol 1, §6.5.3 |
| Basis swap par condition equation | Andersen & Piterbarg Vol 1, Eq (6.49) |
| Risk decomposition: discount vs projection vs basis | Andersen & Piterbarg Vol 1, §6.5.3 |
| SOFR as backward-looking compounded rate | Hull Ch 4, Ch 6 |
| LIBOR forward-looking vs RFR backward-looking | Hull Ch 4 |
| Sequential curve construction methodology | Andersen & Piterbarg Vol 1, §6.5.3 |

### (B) Reasoned Inference (Derived from A)

| Inference | Derivation |
|-----------|------------|
| "Par Floater Paradox" (FRN $\neq$ 100) | Derived from the fact that $P_k \neq P_d$ breaks the telescoping sum identity. |
| Single curve is inconsistent | Derived from the mathematical impossibility of fitting one curve to both OIS and FRA markets. |
| Par swap formula mixes curves | Direct algebraic consequence of projecting at $F_k$ and discounting at $P_d$. |

### (C) Flagged Uncertainties

| Topic | Uncertainty |
|-------|-------------|
| **SOFR Term Rates** | I'm not sure about the exact conventions for CME Term SOFR vs compounded in arrears SOFR—these vary by product and are still evolving. |
| **Basis Swap Quoting** | Market conventions for which leg receives the spread vary by currency and dealer. Always verify the term sheet. |
| **BSBY and Other Credit Rates** | The long-term viability of credit-sensitive rates like BSBY is uncertain given regulatory pressures. |
