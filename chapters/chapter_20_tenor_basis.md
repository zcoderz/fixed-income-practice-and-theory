# Chapter 20: Tenor Basis — 1M vs 3M vs 6M and Basis Swap Logic

---

## Introduction

If you can earn 3-month LIBOR, why not just roll 1-month LIBOR three times? In a textbook world of perfect liquidity and risk-free counterparties, these strategies should be equivalent. Both provide funding for the same three-month horizon. Indeed, for decades, financial models assumed precisely this: that a single yield curve could price all instruments, regardless of their payment frequency or underlying tenor.

That assumption collapsed spectacularly in August 2007. As the Global Financial Crisis began to unfold, the spread between rolling overnight rates, 1-month rates, and 3-month rates—historically a trivial fraction of a basis point—blew out to dozens of basis points. Andersen and Piterbarg document the magnitude of this shift with striking precision: "the difference between 1 month and 3 month Libor rates was in the order of one basis point up until September 2007, but since then has been as wide as 50 basis points." Simultaneously, the spread between the Fed funds rate and 3-month LIBOR widened to as much as 275 basis points—a level that would have been considered unthinkable just months earlier.

The emergence of **tenor basis** revealed a fundamental truth: **different funding tenors represent fundamentally different products**. A 6-month loan embeds more credit risk and liquidity premium than a 1-month loan, and the market began pricing this difference aggressively. Consequently, a single curve can no longer value both 3-month and 6-month swaps without creating arbitrage opportunities. Pricing a portfolio today without accounting for tenor basis is not just imprecise—it could result in systematic mispricing of millions of dollars on a standard vanilla swap portfolio.

This chapter develops the machinery for handling tenor basis within the multi-curve framework introduced in Chapter 19. We assume the reader is already familiar with the distinction between discount curves (OIS) and projection curves (tenor-specific). Here, we focus specifically on:

1. **What tenor means economically** and why different tenors are not interchangeable (Section 20.1)
2. **The drivers of tenor basis**: credit, liquidity, and supply/demand (Section 20.2)
3. **Basis swaps**: the instruments used to trade and hedge the spread between tenors (Section 20.3)
4. **The mathematics of multi-curve valuation** with tenor-specific projection curves (Section 20.4)
5. **Bootstrapping secondary tenor curves** from basis swap quotes (Section 20.5)
6. **Risk decomposition**: why DV01 hedging alone leaves residual basis exposure (Section 20.6)

**Chapter boundaries:** Chapter 19 introduced the multi-curve framework and the conceptual distinction between discount and projection curves. This chapter goes deeper into the *tenor dimension*—how to handle the differences between 1M, 3M, and 6M projection curves within a single currency. Chapter 21 extends the framework to multiple currencies, where cross-currency basis introduces additional constraints.

---

## 20.1 Tenor and the Economics of Term Funding

### 20.1.1 What "Tenor" Means

A **tenor** refers to the accrual length associated with a quoted simple rate. When a rate $L(t,T,T+\tau)$ is observed at time $t$ for borrowing over the period $[T, T+\tau]$, the tenor is the duration $\tau$.

Common tenors in interest rate markets include:

| Tenor | Description | Primary Use |
|-------|-------------|-------------|
| Overnight (ON) | The foundation of risk-free rate (RFR) markets | SOFR, SONIA, ESTR |
| 1 Month (1M) | Short-term funding | Money market, CP |
| 3 Months (3M) | Most liquid swap tenor | Standard IRS |
| 6 Months (6M) | Common in European markets | EURIBOR swaps |
| 12 Months (12M) | Less common | Some structured products |

Historically, LIBOR was the primary benchmark for these term tenors. In the post-LIBOR world, while overnight rates dominate, term rates (like Term SOFR or EURIBOR) remain critical for lending and cross-currency markets.

### 20.1.2 Why Tenors Are Not Interchangeable

The labels "1M," "3M," and "6M" are not merely calendar arithmetic. They represent **different funding markets** with distinct risk profiles. Andersen and Piterbarg explain that the tenor basis arises "partly from credit considerations and partly liquidity considerations."

**Credit Horizon Risk.** A bank lending cash for 6 months takes on strictly more credit risk than a bank lending for 1 month, even if the borrower is the same. The probability of default increases with the time horizon. The 6-month rate must therefore include a higher credit spread. This is the *term premium for credit risk*—the compensation for locking in exposure to a counterparty over a longer horizon.

**Liquidity Preference.** Banks prefer short-term liabilities because shorter funding gives them more flexibility to manage their balance sheets. As Andersen and Piterbarg note, "banks have a natural desire to have longer-term deposits to better match their loan commitments." This structural demand for longer-dated funding creates a liquidity premium for 6M and 12M cash.

**Market Segmentation.** The participants in 1-month markets (e.g., money market funds, corporate treasuries) often differ from those in 6-month markets (e.g., insurance companies, asset managers with longer duration mandates). These distinct investor bases create separate supply-demand dynamics.

In calm markets, these differences manifest as small but measurable spreads—a few basis points at most. In stressed markets, they become the dominant driver of P&L. When interbank trust evaporated in 2008, the basis between tenors widened dramatically because lenders demanded much higher compensation for the additional uncertainty of longer-term exposure.

> **Why this matters:** If you model 6-month rates as simply compounded 3-month rates, you will systematically misprice any instrument with 6-month floating payments. The error can be tens of basis points—material for trading or hedging decisions.

> **Analogy: Liquid Water vs. Ice**
>
> Think of cash as water.
> *   **1-Month Cash**: It's liquid. You can drink it (use it) in 30 days.
> *   **6-Month Cash**: It's frozen in a block of ice. You can't use it for 180 days.
> *   **The Premium**: If I ask you to freeze your water for 6 months, you demand a premium for the inconvenience (Liquidity Risk). You also worry I might steal the ice block while it's freezing (Credit Risk).
> *   **The Result**: 6-month rates are *structurally higher* than rolling 1-month rates. You can't just melt the 6-month rate into 1-month chunks without losing value.

---

## 20.2 The Drivers of Tenor Basis

### 20.2.1 Defining Tenor Basis

**Tenor basis** is the spread between projection curves of different tenors. When tenor basis is non-zero:

$$P^{(3M)}(0,T) \neq P^{(6M)}(0,T)$$

This means the pseudo-discount factors used to calculate 3-month and 6-month forward rates differ, even for the same maturity. Equivalently, compounding 3-month rates will not reproduce the 6-month rate:

$$\mathbb{E}[\text{3M Rate compounded twice}] \neq \mathbb{E}[\text{6M Rate}]$$

The difference is the tenor basis.

### 20.2.2 Empirical Behavior

The basis tends to exhibit several regularities:

1. **Wider with maturity:** The 10-year basis swap spread is typically larger than the 2-year spread, reflecting long-term uncertainty about credit and liquidity conditions.

2. **Wider with tenor difference:** The 1M-6M basis is typically larger than the 3M-6M basis because the difference in lock-up period is greater.

3. **Procyclical with stress:** During crises—2008, the European debt crisis, COVID-19 in March 2020—basis spreads widen significantly as banks hoard liquidity and prefer shorter-term funding.

4. **Mean-reverting but persistent:** Unlike temporary dislocations, tenor basis does not fully collapse to zero even in normal markets. There is a structural component reflecting the economic differences between tenors.

### 20.2.3 Historical Context: The 2007-2008 Regime Change

Before 2007, the basis was so small that practitioners largely ignored it. Andersen and Piterbarg describe the old approach: "When various basis levels were small, the small discrepancies between different Libor-tenor swaps were often accounted for by building a unique discount curve for the subset of swaps referencing the Libor rate of a particular tenor."

This created logical inconsistencies—fixed legs of swaps would be discounted at different rates depending on which tenor they happened to be paired with—but the errors were negligible in practice.

The crisis exposed this approach as fundamentally flawed. Andersen and Piterbarg note that after September 2007, the Fed funds/LIBOR spread "went up to as much as 275 basis points, and it is now generally accepted that the Libor rate is no longer a good proxy for a discounting rate on collateralized trades."

> **Historical Note:** The 275 basis point spread between Fed funds and 3-month LIBOR in late 2008 remains one of the most dramatic dislocations in modern financial history. It signaled that banks considered lending to each other for even three months to be deeply risky.

> **Visual: The Basis Spread History**
>
> Imagine a line graph of the 3M-OIS Annualized Spread.
> *   **2000-2007**: A flat line near zero (0-5 bps). "Boring."
> *   **2008**: Vertical spike to 350 bps. "Panic."
> *   **2010-2012**: Elevated, choppy (Euro Crisis). "Stress."
> *   **2020**: Sharp spike (Covid). "Liquidity Crunch."
>
> This graph proves that Tenor Basis is a **fear gauge**. When banks are scared, they hoard short-term cash, and the basis explodes.

---

## 20.3 Basis Swaps: The Trading Instrument

### 20.3.1 Structure of a Basis Swap

A **basis swap** is a swap that exchanges floating payments linked to one tenor against floating payments linked to another tenor. Unlike a standard fixed-for-floating swap, both legs are floating.

Andersen and Piterbarg define this precisely: "A floating-floating basis swap is a swap of payments linked to a Libor rate of a particular tenor such as $L^{1}$—versus payments based on a Libor rate of different tenor—such as $L^{2}$. Each leg pays on its own schedule of a corresponding tenor."

For example, a "3M vs 6M" basis swap might exchange:

- **Leg 1**: 3M LIBOR + Spread, paid quarterly
- **Leg 2**: 6M LIBOR flat, paid semi-annually

A fixed spread is typically added to one of the legs to make the swap value at inception equal to zero.

### 20.3.2 The Par Basis Spread

The **par basis spread** $e_{1,2}(T)$ is the spread added to one leg such that the net present value of the swap is zero at inception.

Andersen and Piterbarg present the equilibrium condition (equation 6.49):

$$\boxed{\sum_{i=0}^{n^{2}(T)-1} L^{2}(0, t_{i}^{2}, t_{i+1}^{2}) \tau_{i}^{2} P(t_{i+1}^{2}) = \sum_{i=0}^{n^{1}(T)-1}\left(L^{1}(0, t_{i}^{1}, t_{i+1}^{1})+e^{1,2}(T)\right) \tau_{i}^{1} P(t_{i+1}^{1})}$$

Here $e^{1,2}(T)$ is the quoted floating-floating basis spread for exchanging $L^{1}$ for $L^{2}$ to maturity $T$, quoted on the $L^{1}$ leg. As Andersen and Piterbarg explain, "It could be positive or negative, depending on perceived desirability of payments linked to $L^{1}$ versus $L^{2}$."

If the market values 6M cashflows more highly (due to the higher risk premium embedded in the rate), the 6M leg will have a higher PV than the flat 3M leg. Therefore, the 3M leg must receive an additional positive spread to compensate.

### 20.3.3 Interpreting the Spread Sign

The convention for which leg receives the spread varies by market. Understanding the sign is critical to avoid confusion.

**Standard interpretation:** If the spread is quoted as positive on the shorter tenor leg, the market is saying the shorter tenor rate is "cheaper" than what you would expect from simply compounding. This makes economic sense: the shorter tenor has less credit/liquidity risk embedded, so it needs a spread added to equate its value to the longer tenor leg.

> **Convention Warning:** Always verify the specific convention for your currency and counterparty. Is the quote "3s6s" paying the spread on the 3M leg or receiving it? Is it quoted as a spread *over* 3M or *under* 6M? A sign error guarantees a loss.

> **I'm not sure** about the exact conventions in emerging RFR markets (e.g., SOFR 1M vs SOFR 3M term) as liquidity is still developing and conventions are evolving. Always verify with the term sheet or Bloomberg `FLDS` function.

> **Deep Dive: The Synthetic Loan**
>
> Why does the basis exist? Arbitrage holds it together.
> *   **Approach A**: Borrow $100M for 3 months directly. Rate = 3M LIBOR.
> *   **Approach B (Synthetic)**:
>     1.  Borrow $100M for 1 month.
>     2.  Enter a "1M vs 3M" Basis Swap to hedge the roll risk.
>     3.  Roll the loan twice.
>
> If Approach A is expensive (Banks hate 3M lending), Approach B becomes cheaper. Traders rush to do B, pushing up the price of the Basis Swap until A and B are roughly equal. The Basis Swap is the price of *transforming* 1M risk into 3M risk.

---

## 20.4 Mathematics of Multi-Curve Valuation

### 20.4.1 Forward Rates from Projection Curves

The forward rate $L^{(k)}$ for a specific tenor $k$ over period $[T_{i-1}, T_i]$ is derived from that tenor's projection curve $P^{(k)}$. Andersen and Piterbarg provide the defining relationship (equation 6.48):

$$\boxed{L^{(k)}(0; T_{i-1}, T_i) = \frac{1}{\tau_i^{(k)}}\left(\frac{P^{(k)}(0,T_{i-1})}{P^{(k)}(0,T_i)} - 1\right)}$$

This is the familiar forward rate formula, but applied to the *projection curve* for tenor $k$, not the discount curve.

> **Worked Example 20.1: Computing Forwards from Projection Curves**
>
> Assume we have a discount curve $P_d$ and a 3M projection curve $P^{(3M)}$ with the following values:
>
> | $T$ (years) | $P_d(0,T)$ | $P^{(3M)}(0,T)$ |
> |-------------|------------|-----------------|
> | 0.00 | 1.0000 | 1.0000 |
> | 0.25 | 0.9950 | 0.9945 |
> | 0.50 | 0.9900 | 0.9886 |
>
> To compute the 3M forward rate for the period $[0.25, 0.50]$:
> $$F^{(3M)} = \frac{1}{0.25}\left(\frac{0.9945}{0.9886} - 1\right) = \frac{1}{0.25}(1.005968 - 1) = 2.387\%$$
>
> **Note:** We used $P^{(3M)}$ to calculate the rate. We did **not** use $P_d$. The discount curve $P_d$ will be used later to value the cashflow.
>
> **Sanity check:** The 3M forward rate is higher than what we would compute from the discount curve alone (which would give roughly 2.0%). This difference—about 39 basis points—is the cumulative effect of the tenor basis.

### 20.4.2 Fixed-Float Swap Pricing

The value of a swap receiving fixed rate $c$ and paying floating rate $L^{(k)}$ is:

$$V^{(k)}(0) = \underbrace{\sum_{i} c \cdot \tau_i \cdot P_d(0,t_i)}_{\text{Fixed PV}} - \underbrace{\sum_{i} F_i^{(k)} \cdot \tau_i \cdot P_d(0,t_i)}_{\text{Float PV}}$$

The **par fixed rate** $c^*$ is the rate that makes $V^{(k)}(0) = 0$. Andersen and Piterbarg express this for a swap with $n$ periods (equation 6.47):

$$\boxed{c^\star = \frac{\sum_{i=0}^{n-1} L^{k}(0, t_{i}^{k}, t_{i+1}^{k}) \tau_{i}^{k} P(t_{i+1}^{k})}{\sum_{i=0}^{n-1} \tau_{i}^{k} P(t_{i+1}^{k})}}$$

Crucially, the par rate depends on **two** curves: the projection curve determines the forwards ($L^{k}$) in the numerator, while the discount curve determines the weights ($P$) in both numerator and denominator.

> **Worked Example 20.2: 3M vs 6M Par Rates**
>
> Consider a 1-year swap. Because the projection curves differ:
>
> - **3M Swap**: Uses 3M forwards averaging, say, 2.50%. Reprices quarterly. The par rate might be **2.50%**.
> - **6M Swap**: Uses 6M forwards. Since 6M rates include a higher credit/liquidity premium, the forwards might average 2.60%. The par rate might be **2.60%**.
>
> This 10 basis point difference is a direct manifestation of the tenor basis. A trader who quotes "the 1-year swap rate" without specifying the tenor is being imprecise.

### 20.4.3 Basis Swap Valuation

For a basis swap exchanging Tenor 2 versus (Tenor 1 + Spread $e$), the par condition is:

$$\sum_{j} F_j^{(2)} \tau_j P_d(0,t_j) = \sum_{i} (F_i^{(1)} + e) \tau_i P_d(0,t_i)$$

Solving for the spread $e$:

$$\boxed{e = \frac{\text{PV}(\text{Leg 2 Float}) - \text{PV}(\text{Leg 1 Float})}{\text{PV01}(\text{Leg 1})}}$$

The PV01 of Leg 1 (sometimes called the "annuity" or "DV01 of the spread leg") is $\sum_i \tau_i P_d(0,t_i)$.

> **Worked Example 20.3: Calculating the Basis Spread**
>
> Consider a 1-year swap exchanging 6M LIBOR against 3M LIBOR + Spread.
>
> **Given:**
> - PV(6M Float Leg): \$0.0256 per unit notional
> - PV(3M Float Leg): \$0.0247 per unit notional
> - Annuity (Leg 1): 0.9876
>
> The 6M leg is more valuable (higher embedded risk premium). To balance, we add a spread to the 3M leg:
>
> $$e = \frac{0.0256 - 0.0247}{0.9876} = \frac{0.0009}{0.9876} \approx 9.1 \text{ bps}$$
>
> **Interpretation:** The market requires approximately 9 basis points added to 3M LIBOR to make receiving it equivalent to receiving 6M LIBOR flat.

---

## 20.5 Bootstrapping a Secondary Tenor Curve

### 20.5.1 The Hierarchical Construction Algorithm

In practice, curves are built sequentially in order of liquidity. Andersen and Piterbarg describe this as constructing each index curve $P^{k}(\cdot)$ for $k>1$ "as a spread, or basis, curve to one of the previous curves."

**Step 1: Build $P_d$ (Discount Curve)**

Use OIS swaps to construct the discount curve. This reflects the cost of funding for collateralized derivatives.

**Step 2: Build $P^{(3M)}$ (Base Projection Curve)**

Use the most liquid fixed-float swaps (typically 3M tenor) along with FRAs. Andersen and Piterbarg recommend using "vanilla instruments referencing $L^{1}$ such as FRAs on $L^{1}$ and fixed-floating swaps on $L^{1}$ versus a fixed rate."

**Step 3: Build $P^{(6M)}$ (Secondary Projection Curve)**

Use **basis swaps** (3M vs 6M) to determine the spread of the 6M curve relative to the 3M curve. Andersen and Piterbarg express this relationship:

$$P^{2}(t)=P^{1}(t) e^{-\int_{0}^{t} \eta^{1,2}(s) d s}$$

for a piecewise-constant spread function $\eta^{1,2}(\cdot)$.

### 20.5.2 The Bootstrapping Algorithm

To bootstrap the 6M curve point by point:

1. Take the already-calibrated 3M curve and discount curve as given.

2. For each basis swap maturity $T$:
   a. Compute the PV of the 3M+Spread leg (all inputs are known from Step 2).
   b. Set the 6M leg PV equal to this value.
   c. Solve for the unknown 6M forward(s) that satisfy the equality.
   d. Convert the forward to a pseudo-discount factor:
   $$P^{(6M)}(0, T_i) = \frac{P^{(6M)}(0, T_{i-1})}{1 + \tau F^{(6M)}}$$

3. Repeat for successively longer maturities.

> **Worked Example 20.4: Bootstrapping a 6M Discount Factor**
>
> **Given (from prior calibration):**
> - $P_d(0, 0.5) = 0.9900$ (OIS discount curve)
> - $P^{(3M)}(0, 0.25) = 0.9945$, $P^{(3M)}(0, 0.50) = 0.9886$
> - 1-year 3s6s basis swap: +8 bps on the 3M leg
>
> **Objective:** Find $P^{(6M)}(0, 0.5)$
>
> **Step 1:** Compute 3M forwards
> - $F^{(3M)}_{0-3M} = (1/0.9945 - 1)/0.25 = 2.21\%$
> - $F^{(3M)}_{3M-6M} = (0.9945/0.9886 - 1)/0.25 = 2.39\%$
>
> **Step 2:** Compute PV of 3M+8bp leg (first 6 months only for simplicity)
> $$\text{PV}_{3M} = (2.21\% + 0.08\%) \times 0.25 \times P_d(0.25) + (2.39\% + 0.08\%) \times 0.25 \times P_d(0.50)$$
> $$= 2.29\% \times 0.25 \times 0.9950 + 2.47\% \times 0.25 \times 0.9900 = 0.5696\% + 0.6113\% = 1.1809\%$$
>
> **Step 3:** Set 6M leg PV equal and solve for $F^{(6M)}$
> $$F^{(6M)}_{0-6M} \times 0.50 \times P_d(0.50) = 1.1809\%$$
> $$F^{(6M)}_{0-6M} \times 0.50 \times 0.9900 = 1.1809\%$$
> $$F^{(6M)}_{0-6M} = 2.386\%$$
>
> **Step 4:** Convert to pseudo-discount factor
> $$P^{(6M)}(0, 0.5) = \frac{1.0000}{1 + 0.50 \times 2.386\%} = 0.9882$$
>
> **Verification:** The 6M pseudo-discount factor (0.9882) is lower than the 3M factor at the same point (0.9886), reflecting the higher forward rate embedded in 6-month funding.

### 20.5.3 Why Sequential Construction Matters

This hierarchical approach has an important property: **locality**. If the 5-year basis swap spread rises by 1 bp, it primarily affects the 6M forward rates around the 5-year point.

If we solved all curves simultaneously, a change in a basis swap quote might propagate unexpectedly into the discount curve or the 3M curve, which is economically nonsensical—liquidity conditions in 6M markets shouldn't change the price of an OIS swap.

As Andersen and Piterbarg emphasize, this "spread-based method of curve group construction" ensures that "sensitivities to instruments used in the curve group have clear, and orthogonal, meaning":

- Perturbations to OIS instruments define **discounting risk**.
- Perturbations to 3M swaps define **overall rate level risk**.
- Perturbations to basis spreads define **basis risk**.

---

## 20.6 Risk Decomposition in Multi-Curve

### 20.6.1 Three Distinct Risk Dimensions

In a single-curve world, you had one sensitivity: interest rate risk (DV01). In the multi-curve world, any position has three orthogonal sensitivities:

| Risk Dimension | Sensitivity To | Hedge Instrument |
|----------------|----------------|------------------|
| **Discount Risk** | OIS curve shifts | OIS swaps |
| **Projection Risk** | Tenor curve shifts (e.g., 3M) | FRAs, Eurodollar/SOFR futures |
| **Basis Risk** | Spread between curves changing | Basis swaps |

A portfolio with zero DV01 may still have massive basis exposure. A trader who hedges a 6M exposure with 3M instruments leaves the tenor basis completely unhedged.

### 20.6.2 Why DV01 Hedging Is Insufficient

> **Worked Example 20.5: The Flawed Hedge**
>
> A trader has a loan paying **6M LIBOR** (\$100M notional). She wants to hedge the interest rate risk.
>
> **Hedge:** She sells a standard **3M LIBOR swap** (receive fixed / pay 3M float) with a matching DV01.
>
> **Scenario A: Parallel Shift.** All rates rise 100 bps in parallel.
> - Loan (receive 6M): Coupon increases—gain.
> - Swap (pay 3M): Float payment increases—loss.
> - **Net:** The hedge works. Gains and losses approximately offset.
>
> **Scenario B: Basis Blowout.** Market stress. OIS and 3M rates unchanged, but 6M rates widen by 20 bps due to credit concerns.
> - Loan (receive 6M): Coupon increases by 20 bps—gain of \$50,000 annualized.
> - Swap (pay 3M): 3M rates unchanged—no offset.
> - **Net:** Unhedged P&L swing of \$50,000.
>
> **Scenario C: Reverse Position.** If the trader were *paying* 6M on a liability (instead of receiving), a basis widening would increase her cost with no hedge offset—potentially a large unexpected loss.

This example illustrates why **basis swaps** are essential hedging instruments. They allow you to isolate and manage the risk of the spread itself, independent of the overall level of rates.

### 20.6.3 Practical Risk Reporting

A modern risk system should decompose interest rate risk into at least:

1. **OIS DV01**: Sensitivity to parallel shift of the discount curve
2. **Projection DV01**: Sensitivity to parallel shift of each tenor curve
3. **Basis DV01**: Sensitivity to change in each basis spread

For more granular analysis, key-rate exposures (see Chapter 14) should be computed for each curve separately.

---

## 20.7 Practical Notes

### 20.7.1 Implementation Pitfalls

**Circular Dependencies.** Ensure your curve construction code respects the hierarchy: OIS → Base Projection → Spread curves. Do not let the 6M curve feed back into the 3M curve construction. If using a global optimizer, this can introduce subtle cross-dependencies that are hard to debug.

**Interpolation Choices.** When interpolating tenor curves, remember that $P^{(k)}$ is not a real discount factor—it is a mathematical construct. Interpolating directly on the "basis spread" $\eta(t)$ often yields smoother results than interpolating the raw pseudo-discount factors. This avoids artifacts where the basis curve develops unexpected wiggles.

**Turn-of-Year Effects.** Some systems maintain "clean" curves (pure interest rate expectations) and "dirty" curves (including turn-of-year premiums). Basis swaps often pick up these seasonal liquidity crunches, so the December basis spread may differ materially from nearby months.

### 20.7.2 Convention Ambiguities

**Which leg gets the spread?** This varies by market and by dealer. In USD, standard inter-dealer basis swaps typically pay the spread on the *shorter* leg (e.g., 3M+spread vs 6M flat). However, quoting conventions can flip. The quote "6s3s" in one system might mean the opposite of "3s6s" in another.

**Best practice:** Before executing any trade, explicitly confirm:
1. Which leg receives the spread
2. Whether the spread is added (+) or subtracted (-)
3. The daycount convention for each leg

> **I'm not sure** about the exact conventions for newer RFR-based tenor basis markets (e.g., SOFR 1M vs SOFR 3M) as these markets are still developing. Liquidity varies by currency and maturity. Always verify with the specific term sheet or electronic trading platform.

---

## Summary

1. **The Paradigm Shift.** Since 2007, single-curve pricing is obsolete for any portfolio with exposure to multiple tenors. Distinct funding tenors (1M, 3M, 6M) require distinct projection curves.

2. **Curve Roles.** The OIS curve ($P_d$) is used for discounting all collateralized cashflows. Tenor-specific projection curves ($P^{(k)}$) are used solely for computing forward rates.

3. **Tenor Basis Economics.** The basis reflects credit risk differences (longer tenor = more default risk) and liquidity preferences (banks prefer shorter liabilities). These premiums are small in normal times but flare dramatically during crises.

4. **Basis Swap Pricing.** The par basis spread is the amount added to one leg (typically the shorter tenor) to equate the PVs of the two floating streams. The spread encodes the market's view of the relative value of different funding tenors.

5. **Sequential Construction.** Curves are built in order of liquidity: OIS → 3M → Basis curves. This ensures stable, local sensitivities and orthogonal risk dimensions.

6. **Risk Management.** Standard DV01 hedging leaves residual basis risk. A portfolio is only fully hedged if it is neutral to discount curve moves, projection curve moves, *and* basis spread changes.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Tenor** | The accrual period of a floating rate (e.g., 3M, 6M) | Different tenors embed different credit/liquidity risk |
| **Tenor Basis** | The spread between forward rates of different tenors | Creates pricing differences; prevents arbitrage between instruments |
| **Multi-Curve Framework** | Using separate curves for discounting (OIS) and projecting (tenor-specific) | The only consistent way to price post-2007 markets |
| **Basis Swap** | A floating-for-floating swap exchanging payments at two different tenors | The primary instrument for hedging tenor mismatch |
| **Projection Curve** | A pseudo-discount curve used to calculate forwards for a specific tenor | Ensures forwards match market FRA/basis quotes exactly |
| **Sequential Bootstrapping** | Building curves in hierarchy: OIS → base tenor → spread curves | Ensures locality and orthogonality of sensitivities |
| **Basis Risk** | The risk that the spread between tenors changes | Not eliminated by DV01 hedging; requires basis swaps |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $P_d(t,T)$ | Discount factor from discount curve (OIS/collateral) |
| $P^{(k)}(t,T)$ | Pseudo-discount factor for projection curve of tenor $k$ |
| $L^{(k)}(t;T_1,T_2)$ | Forward rate for tenor $k$ over period $[T_1,T_2]$ |
| $e^{1,2}(T)$ or $e_{1,2}(T)$ | Par basis spread at maturity $T$ (Tenor 1 vs Tenor 2) |
| $\tau_i^{(k)}$ | Year fraction for period $i$ in tenor structure $k$ |
| $\eta^{1,2}(t)$ | Continuous spread function between curves 1 and 2 |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the fundamental economic difference between a 1M and 6M rate? | The 6M rate embeds higher credit risk (longer exposure) and liquidity premium (longer lock-up of funds). |
| 2 | Why can't a single curve price both 3M and 6M swaps consistently? | They imply different discount factors for the same date; using one curve creates arbitrage. |
| 3 | Which curve is used for discounting in a collateralized swap? | The OIS curve (reflecting the rate paid on collateral). |
| 4 | Which curve is used for calculating floating payments? | The projection curve specific to that tenor (e.g., 3M curve for 3M LIBOR). |
| 5 | What is a basis swap? | A swap exchanging floating payments of two different tenors (e.g., 3M vs 6M). |
| 6 | In a 3s6s basis swap, which leg usually receives the spread? | Typically the shorter/lower-rate tenor leg (3M) to compensate for its lower embedded risk premium. |
| 7 | What is "basis risk"? | The risk that the spread between two tenors widens or tightens unexpectedly. |
| 8 | Does a DV01-neutral hedge eliminate basis risk? | No. You can be DV01 flat but still have large exposure if the basis widens. |
| 9 | What is the correct order for curve bootstrapping? | Discount (OIS) first, then base projection (3M), then secondary projections via basis swaps. |
| 10 | Why did the 1M-3M basis widen in 2007-2008? | Concerns over bank credit quality and liquidity hoarding made longer-term unsecured lending appear much riskier. |
| 11 | What was the approximate peak of the Fed funds/LIBOR spread during the crisis? | Up to 275 basis points. |
| 12 | How wide did the 1M-3M basis get during the crisis? | Up to approximately 50 basis points. |
| 13 | What does a positive basis spread on the 3M leg mean? | The market views 3M cash flows as "less valuable" than 6M, so a spread must be added to equate PVs. |
| 14 | Why is sequential curve construction preferred over simultaneous solving? | It ensures locality (bumping 5Y basis only affects 5Y region) and orthogonal risk sensitivities. |
| 15 | What three risk dimensions should a multi-curve position be analyzed for? | Discount risk, projection risk, and basis risk. |
| 16 | If you receive 6M LIBOR and hedge by paying 3M LIBOR, what residual risk do you have? | Basis risk—if 6M-3M spread widens, you gain; if it tightens, you lose. |
| 17 | What is a "pseudo-discount factor"? | A mathematical construct ($P^{(k)}$) that reproduces tenor-specific forwards via the standard formula, but is not used for discounting cash flows. |
| 18 | Why might interpolating on basis spreads be preferable to interpolating pseudo-discount factors? | It often produces smoother curves and avoids interpolation artifacts. |

---

## Mini Problem Set

### Problem 1 (Basic: Forward Rate Calculation)

Given $P^{(3M)}(0,0) = 1$, $P^{(3M)}(0,0.25) = 0.9940$, and $\tau = 0.25$, compute the 3M forward rate for the first period.

**Solution:**
$$F = \frac{1}{0.25}\left(\frac{1}{0.9940} - 1\right) = \frac{1}{0.25}(0.006036) = 2.414\%$$

### Problem 2 (Basic: Basis Spread Calculation)

You are pricing a 6M vs 3M basis swap with the following values:
- PV(6M Leg) = \$1,020,000
- PV(3M Leg) = \$1,000,000
- PV01(3M Leg) = \$5,000

What is the par basis spread added to the 3M leg?

**Solution:**
$$\text{Spread} = \frac{1,020,000 - 1,000,000}{5,000} = \frac{20,000}{5,000} = 4 \text{ bps}$$

### Problem 3 (Intermediate: Hedging and Basis Risk)

A trader hedges a \$100 million 6M-LIBOR loan (receiving float) by paying fixed on a 3M-LIBOR swap. The 6M-3M basis widens by 10 bps. Does the trader gain or lose money, and approximately how much (annualized)?

**Solution:** The trader receives 6M rates, which increase relative to 3M. The swap (paying 3M via receiving fixed) doesn't offset this increase. The trader **gains** approximately:
$$\$100\text{M} \times 0.0010 = \$100,000 \text{ annualized}$$

### Problem 4 (Intermediate: Pseudo-Discount Factor)

If the 6M forward rate is 2.60% and the 6-month year fraction is 0.5, and $P^{(6M)}(0,0) = 1$, what is $P^{(6M)}(0, 0.5)$?

**Solution:**
$$P^{(6M)}(0, 0.5) = \frac{1.0}{1 + 0.5 \times 0.0260} = \frac{1}{1.013} = 0.9872$$

### Problem 5 (Intermediate: Comparing Par Rates)

Two swaps have identical structures except for tenor:
- Swap A: 2-year, pays 3M LIBOR, average 3M forward = 2.80%
- Swap B: 2-year, pays 6M LIBOR, average 6M forward = 3.00%

Assuming identical discount factors, which swap has a higher par fixed rate, and by approximately how much?

**Solution:** Swap B has higher forwards (3.00% vs 2.80%), so its par rate is higher by approximately 20 bps. The par rate is roughly the weighted average of forwards, so Swap B's par rate exceeds Swap A's by about 20 bps.

### Problem 6 (Advanced: Floater Valuation)

Derive why a floating rate note (FRN) paying 3M LIBOR does not trade at par in a multi-curve world, even on a coupon reset date.

**Solution Sketch:** On a reset date in a single-curve world, the FRN prices at par because the next coupon equals the discount rate × accrual, and future coupons can be shown to telescope to principal.

In multi-curve: The coupons are set by the 3M projection curve ($F^{(3M)}$), but discounted at OIS ($P_d$). Since typically $F^{(3M)} > F^{OIS}$ (LIBOR includes credit/liquidity premium), coupons are "richer" than what the discount curve expects. The telescoping identity breaks:
$$\text{PV} = \sum F^{(3M)} \tau P_d + 100 \cdot P_d(T) \neq 100$$

The FRN trades above par when LIBOR > OIS.

### Problem 7 (Advanced: Sequential Bootstrapping)

Explain what would go wrong if you tried to bootstrap the 6M curve directly from 6M swap rates without first building the OIS and 3M curves.

**Solution Sketch:**
1. You wouldn't know how to discount the fixed leg (need OIS curve).
2. If you used the 6M curve for both projection and discounting, you'd create internal inconsistency with collateralized trades (which should discount at OIS).
3. You couldn't properly value basis swaps (which compare 6M to 3M), so the 6M curve wouldn't be consistent with basis swap quotes.
4. Risk sensitivities would not be orthogonal—a change in 6M swaps would move what should be the discount curve.

### Problem 8 (Advanced: Curve Shock Decomposition)

A portfolio has:
- OIS DV01 = +\$100,000
- 3M projection DV01 = -\$80,000
- 3s6s basis DV01 = +\$20,000

If rates move as follows: OIS +5bp, 3M projection +5bp, 3s6s basis +3bp, what is the approximate P&L?

**Solution:**
$$\text{P\&L} = (100,000 \times 5) + (-80,000 \times 5) + (20,000 \times 3)$$
$$= 500,000 - 400,000 + 60,000 = +\$160,000$$

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| Multi-index curve group definition: collection $\{P(\cdot), P^{1}(\cdot), \ldots, P^{K}(\cdot)\}$ | Andersen & Piterbarg Vol 1, §6.5.3 |
| Basis swap PV equality equation (6.49) | Andersen & Piterbarg Vol 1, §6.5.3 |
| 1M–3M basis historically ~1bp, widened to ~50bp post-2007 | Andersen & Piterbarg Vol 1, §6.5.3 |
| Fed funds/LIBOR spread reached 275bp post-2007 | Andersen & Piterbarg Vol 1, §6.5.3 |
| Tenor basis attributed to "credit considerations and partly liquidity considerations" | Andersen & Piterbarg Vol 1, §6.5.3 |
| Banks "have a natural desire to have longer-term deposits to better match their loan commitments" | Andersen & Piterbarg Vol 1, §6.5.3 |
| Sequential curve construction methodology with spread functions | Andersen & Piterbarg Vol 1, §6.5.3 |
| OIS discounting for collateralized trades | Hull Ch 7; Andersen & Piterbarg Vol 1, §6.5.3 |
| Forward rate formula from projection curve (6.48) | Andersen & Piterbarg Vol 1, §6.5.3 |
| Fixed-float swap valuation with separate discount/projection (6.47) | Andersen & Piterbarg Vol 1, §6.5.3 |
| Orthogonal risk sensitivities from spread-based construction | Andersen & Piterbarg Vol 1, §6.5.3 |
| Basis swap spread can be "positive or negative, depending on perceived desirability" | Andersen & Piterbarg Vol 1, §6.5.3 |

### (B) Reasoned Inference (Derived from A)

| Inference | Derivation |
|-----------|------------|
| Par Basis Spread Formula | Derived algebraically from the standard PV equality condition (rearranging equation 6.49). |
| Example Calculations | Constructed to demonstrate the arithmetic of the derived formulas using typical market values. |
| Risk Scenarios (hedging examples) | Inferred from the definitions of curve sensitivities and the orthogonal decomposition. |
| FRN above par in multi-curve | Derived from the fact that $F^{(3M)} > F^{OIS}$ breaks the telescoping sum identity. |

### (C) Flagged Uncertainties

| Topic | Uncertainty |
|-------|-------------|
| Market Quoting Conventions | Specifics of which leg pays the spread (3s6s vs 6s3s) vary by dealer, currency, and electronic platform. Always verify. |
| RFR Tenor Conventions | I'm not sure about the exact conventions for developing SOFR Term / RFR basis markets as liquidity is still emerging. |
| Turn-of-year effects | Seasonal effects on basis spreads are market-dependent and evolve year to year. |
