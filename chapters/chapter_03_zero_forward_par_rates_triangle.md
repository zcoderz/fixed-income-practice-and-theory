# Chapter 3: Zero Rates, Forward Rates, Par Rates — The Triangle

---

## Introduction

You have a discount factor curve from your risk engine. A colleague has a zero rate curve in Excel. The swap desk quotes par swap rates. The bond trading system reports yields. Are these four different things—or four different ways of describing the same thing?

The answer is both. Discount factors, zero rates, forward rates, and par rates are **equivalent representations** of the term structure of interest rates. They are the four corners of a square (or triangle, if you group yields and par rates), and once you know any one of them (with appropriate conventions), you can derive all the others through pure algebra.

Why does this matter? Because different markets speak different languages:
*   **Quants** and pricing engines think in **discount factors** (the primitive).
*   **Risk managers** often view **zero rates** (spot rates) to understand term structure shape.
*   **Traders** price floating-rate notes and swaps using **forward rates**.
*   **The Market** quotes prices in **par rates** (swap rates) or **yields** (bond yields).

If you cannot fluently convert between these representations, you cannot reconcile positions across systems, you cannot price a swap against a bond curve, and you cannot understand why your model disagrees with a trader's quote.

This chapter develops the "Rate Triangle"—the mathematical machinery for converting between these representations. We will strip away the complexity of curve construction (bootstrapping, which is covered in **Chapter 17**) to focus purely on the algebra of these definitions.

---

## 3.1 The Discount Factor: The Primitive Object

### Definition and Economic Meaning

Every rate concept derives from one fundamental building block: the discount factor.

Tuckman defines the discount factor $d(t)$ (or $P(0,t)$) as "the present value of $1 to be received at time $t$" (Tuckman Ch 1). Equivalently, it is the price today of a risk-free zero-coupon bond paying 1 unit of currency at maturity $T$.

$$\boxed{P(0,T) = \text{Price today of \$1 paid at time } T}$$

The notation $P(t,T)$ represents the price at time $t$ of a payment at $T$. When $t=0$ (today), we often shorten this to $P(T)$ or $d(T)$.

### Why Discount Factors Are Fundamental

Discount factors are the "atomic units" of valuation. They tell you directly **how much a future dollar is worth today**, stripping away all conventions about compounding frequencies, day counts, or accrual periods.

The pricing equation for **any** deterministic cashflow stream is simply:

$$PV = \sum_{i} CF_i \times P(0,T_i)$$

Everything else—yields, spreads, durations—is just a layer of interpretation on top of this equation.

### The No-Arbitrage Requirements

Tuckman (Ch 1) establishes the fundamental requirements that discount factors must satisfy:

1.  **Positivity:** $P(0,T) > 0$ for all $T$. If $P(0,T)$ were 0 or negative, you could buy money at $T$ for free (or get paid to take it), leading to infinite arbitrage.
2.  **Normalization:** $P(0,0) = 1$. A dollar today is worth a dollar today.
3.  **Monotonicity (Usually):** In normal markets, $P(0,T)$ decreases as $T$ increases. However, in **negative interest rate environments** (like EUR and JPY post-2014), discount factors can exceed 1 and increase with maturity over short intervals. The algebra in this chapter holds regardless of rate signs, provided $P(0,T) > 0$.

> **Expert Note: The OIS Discounting Standard**
>
> In modern (post-2008) practice, the "risk-free" discount factor $P(0,T)$ is almost universally derived from the **Overnight Indexed Swap (OIS)** curve (e.g., Fed Funds or SOFR in USD, ESTR in EUR).
>
> Prior to 2008, practitioners often used the LIBOR curve for risk-free discounting. Today, we strictly separate the **discounting curve** (OIS) from the **projection curves** (used to estimate future floating cashflows like LIBOR). Unless stated otherwise, $P(0,T)$ in this book refers to the OIS discount factor. See Andersen & Piterbarg Vol 1, Ch 6 for a detailed treatment of multi-curve discounting.

---

## 3.2 Zero (Spot) Rates: One Rate for the Whole Horizon

### The Concept

A zero rate (or spot rate) answers the question: *What single annual interest rate, applied over the entire horizon from 0 to $T$, produces the discount factor $P(0,T)$?*

Tuckman defines the spot rate as "the rate on a spot loan... in which the lender gives money to the borrower at the time of the agreement" (Tuckman Ch 2). Hull similarly defines the $n$-year zero rate as "the rate of interest earned on an investment that starts today and lasts for $n$ years," with **no intermediate payments** (Hull Ch 4).

Crucially, **you cannot quote a zero rate without specifying a compounding convention.** The same discount factor implies different numerical rates depending on how you compound.

### Continuous Compounding

This is the standard for derivatives pricing code (e.g., Black-Scholes, QuantLib internals). Hull (eq. 4.3) gives the fundamental relationship:

$$P(0,T) = e^{-z_c(T) \cdot T}$$

Solving for the rate (Hull eq. 4.5):

$$\boxed{z_c(T) = -\frac{\ln P(0,T)}{T}}$$

**Why it's preferred by quants:** The math of calculus works best with geometric (continuous) growth. It is the limit of compounding as frequency $m \to \infty$. Tuckman (eq. 2.9) provides the equivalent formulation.

### Semiannual Compounding (Treasury Convention)

This is the standard for U.S. bond markets. Tuckman (eq. 2.10) derives this relationship: "What coupon rate would a bond pay if it sold at par and had only one payment at maturity $T$?"

$$P(0,T) = \frac{1}{\left(1 + \frac{z_{sa}(T)}{2}\right)^{2T}}$$

Solving for the rate:

$$\boxed{z_{sa}(T) = 2 \left[ \left( \frac{1}{P(0,T)} \right)^{\frac{1}{2T}} - 1 \right]}$$

### Simple Interest (Money Market Convention)

Used for short-term rates (typically $T < 1$ year), like LIBOR/SOFR fixings and T-Bills. There is no "compounding" of interest on interest.

$$P(0,T) = \frac{1}{1 + z_{simple}(T) \cdot T}$$

Solving for the rate:

$$\boxed{z_{simple}(T) = \frac{1}{T} \left( \frac{1}{P(0,T)} - 1 \right)}$$

### Convention Risk: The Map is Not the Territory

A common rookie mistake is to assume "5% is 5%."

**Example (adapted from Hull Ch 4):**
Suppose $P(0, 1) = 0.9512$ (roughly 5%).
*   **Simple Rate:** $(1/0.9512 - 1) / 1 = 5.130\%$
*   **Semiannual Rate:** $2 \times ((1/0.9512)^{0.5} - 1) = 5.063\%$
*   **Continuous Rate:** $-\ln(0.9512) = 5.003\%$

All three represent the **exact same economic value**. If you plug a 5.13% simple rate into a formula expecting a continuous rate, you will misprice the instrument.

### Conversion Between Compounding Conventions

Hull (eq. 4.3, 4.4) provides the general conversion formula. If $R_c$ is a continuously compounded rate and $R_m$ is a rate with compounding $m$ times per year:

$$R_c = m \ln\left(1 + \frac{R_m}{m}\right)$$

$$R_m = m\left(e^{R_c/m} - 1\right)$$

**Example:** Converting 6% continuous to semiannual:
$$R_2 = 2(e^{0.06/2} - 1) = 2(e^{0.03} - 1) = 2(1.03045 - 1) = 6.09\%$$

---

## 3.3 Forward Rates: Rates for Future Periods

### The Concept

A forward rate answers the question: *What rate for a future period $[T_1, T_2]$ is consistent with today's discount factors?*

It is **not** a forecast of where rates will be in the future. It is a "breakeven" rate that prevents arbitrage between investing long-term vs. rolling over short-term investments.

Hull defines forward rates as "the rates of interest implied by current zero rates for periods of time in the future" (Hull Ch 4).

### The No-Arbitrage Derivation: Locking in the Forward Rate

Hull provides an illuminating example of how forward rates can be locked in (Hull Ch 4, "Forward Rates" section). Consider an investor who wants to guarantee a rate for a future period.

**Hull's Example (adapted):**
Suppose the 1-year zero rate is 3% and the 2-year zero rate is 4% (both continuously compounded). The forward rate for year 2 can be locked in as follows:

**Strategy:**
1. Borrow $e^{-0.04 \times 2} = \$0.9231$ for 2 years at 4%. Repayment at year 2: $\$1.00$
2. Invest the $\$0.9231$ for 1 year at 3%. Receive at year 1: $0.9231 \times e^{0.03} = \$0.9512$

**Net Result:**
- Today: Net zero (borrow and immediately invest)
- Year 1: Receive $\$0.9512$
- Year 2: Pay $\$1.00$

This is equivalent to borrowing $\$0.9512$ at year 1 and repaying $\$1.00$ at year 2—a 1-year loan starting in 1 year. The implied rate is:
$$\ln(1.00/0.9512) = 5.13\%$$

This matches the no-arbitrage forward rate formula:
$$f = \frac{0.04 \times 2 - 0.03 \times 1}{2 - 1} = 5\%$$ (using continuous compounding, the exact calculation gives 5.00%)

### The General No-Arbitrage Derivation

Imagine you want to guarantee a borrowing rate for a period starting at $T_1$ and ending at $T_2$ (where $\tau = T_2 - T_1$).

**Strategy A (The Forward Loan):**
Borrow \$1 at $T_1$ and repay $1 + F \cdot \tau$ at $T_2$. Value at $T_1$ is 1.

**Strategy B (Synthetic Construction):**
1.  **Borrow Long:** Borrow $1/P(0, T_1)$ dollars today for term $T_2$ at rate $z(T_2)$. You owe $1/P(0, T_1) \div P(0, T_2)$ at time $T_2$.
2.  **Lend Short:** Lend the $1/P(0, T_1)$ dollars today for term $T_1$. You receive exactly \$1 at time $T_1$.

**Net Cashflows:**
*   **Today:** $+ \frac{1}{P(0,T_1)}$ (from borrowing) $- \frac{1}{P(0,T_1)}$ (from lending) = **0**.
*   **Time $T_1$:** Receive **\$1** from the lending leg.
*   **Time $T_2$:** Owe $\frac{P(0,T_1)}{P(0,T_2)}$ from the borrowing leg.

This synthetic structure creates a loan of \$1 at $T_1$ with a repayment obligation at $T_2$. To avoid arbitrage, the forward contract rate $F$ must match this repayment obligation.

$$1 + F_{\text{simple}} \cdot \tau = \frac{P(0,T_1)}{P(0,T_2)}$$

### Forward Rate Formulas

**Simple Forward Rate (Money Market / LIBOR style):**
Used for FRAs and floating rate legs. Tuckman (eq. 2.14) provides:

$$\boxed{F_{\text{simple}}(0; T_1, T_2) = \frac{1}{\tau} \left( \frac{P(0,T_1)}{P(0, T_2)} - 1 \right)}$$

**Continuously Compounded Forward Rate:**
Used in theoretical modeling. Hull (eq. 4.5) and Tuckman (eq. 2.16) give:

$$\boxed{f_c(0; T_1, T_2) = \frac{z_c(T_2) \cdot T_2 - z_c(T_1) \cdot T_1}{T_2 - T_1} = \frac{\ln P(0, T_1) - \ln P(0, T_2)}{T_2 - T_1}}$$

**Semiannual Forward Rate:**
Tuckman (eq. 2.17) derives:

$$\boxed{f_{sa}(0; T_1, T_2) = 2 \left[ \left(\frac{P(0,T_1)}{P(0,T_2)}\right)^{\frac{1}{2(T_2-T_1)}} - 1 \right]}$$

### Instantaneous Forward Rate

The limit as $T_2 \to T_1$. This describes the forward rate "at" a single point in time $t$. Brigo & Mercurio (Definition 1.4.2) define:

$$\boxed{f(0,T) = -\frac{\partial}{\partial T} \ln P(0,T)}$$

This derivative form is central to the **HJM framework** (Appendix A3) and modern curve construction.

**The Integral Relationship:**
Brigo & Mercurio also establish the inverse relationship—the discount factor can be recovered from instantaneous forwards:

$$\boxed{P(0,T) = \exp\left(-\int_0^T f(0,u) \, du\right)}$$

This integral form shows that the discount factor is determined by the entire path of instantaneous forward rates from 0 to $T$. Hull (eq. 4.6) provides the equivalent formulation.

---

## 3.4 The Spot–Forward Relationship

### The Key Identity

Under continuous compounding, there is a beautiful relationship between the spot rate curve $z(T)$ and the instantaneous forward rate $f(T)$. Tuckman derives this in Appendix 2A:

$$\boxed{f(T) = z(T) + T \cdot \frac{\partial z(T)}{\partial T}}$$

**Derivation:**
Starting from $P(0,T) = e^{-z(T) \cdot T}$, we have $\ln P(0,T) = -z(T) \cdot T$.

Taking the derivative with respect to $T$:
$$f(T) = -\frac{\partial}{\partial T} \ln P(0,T) = -\frac{\partial}{\partial T}[-z(T) \cdot T] = z(T) + T \cdot z'(T)$$

### Interpreting Curve Shape: The Marginal vs. Average Intuition

> **Analogy: The Grade Point Average (GPA)**
>
> Think of the **Spot Rate** ($z$) as your cumulative **GPA**.
> Think of the **Forward Rate** ($f$) as your **Semester Grade**.
>
> *   If your Semester Grade (Forward) is **higher** than your GPA, your GPA **rises** ($f > z \implies z' > 0$).
> *   If your Semester Grade is **lower** than your GPA, your GPA **falls** ($f < z \implies z' < 0$).
> *   If your Semester Grade equals your GPA, your GPA stays **flat** ($f = z \implies z' = 0$).
>
> The forward rate is the "marginal" rate pulling the "average" (spot) rate up or down.

This equation gives us the **"Marginal vs. Average"** intuition, similar to costs in economics. Tuckman emphasizes this interpretation:


1.  **Upward Sloping Curve ($z' > 0$):** If spot rates are rising, the forward rate must be **higher** than the spot rate ($f > z$). (To raise the average, the new marginal rate must be higher).
2.  **Downward Sloping Curve ($z' < 0$):** If spot rates are falling (inverted), the forward rate must be **lower** than the spot rate ($f < z$).
3.  **Flat Curve ($z' = 0$):** Spot and forward rates are equal.

**Numerical Example (from Tuckman):**
If the 5-year spot rate is 4% and rising at 0.20% per year (i.e., $z'(5) = 0.002$), then:
$$f(5) = 0.04 + 5 \times 0.002 = 0.04 + 0.01 = 5\%$$

The forward rate at 5 years is 100bp higher than the spot rate because the curve is upward sloping.

### Practical Implication: Forwards are "Wigglier"

> **Analogy: The Speedometer**
>
> *   **Spot Rate**: Your Average Speed over a trip ($D/T$).
> *   **Forward Rate**: Your Instantaneous Speed *right now* (reading on the speedometer).
>
> You can maintain a smooth Average Speed of 60mph (Spot) even if you slam on the brakes or gas (Forwards) periodically. But if you want to change your Average Speed quickly (a kink in the Spot curve), you have to drive *extremely* fast or slow for a short time.
>
> *Result:* Small bumps in the "smooth" spot curve require huge spikes in the forward curve.

Because forward rates depend on the *slope* (derivative) of the spot curve, small bumps in the spot curve become large spikes in the forward curve.

*   A localized "kink" in spot rates $\rightarrow$ A massive jump in forward rates.
*   This is why **smoothness** is a key constraint in curve fitting (Chapter 17). If you bootstrap a curve naively, your forward rates will look like a saw blade, which implies arbitrage opportunities (e.g., implausible calendar spread valuations).

---

## 3.5 Par Rates: Annuity-Weighted Summaries

### The Concept

A par rate answers: *What coupon rate $C$ makes a bond (or swap) worth exactly 100% of par?*

While zero rates apply to single cashflows, par rates apply to **series** of cashflows. Hull defines the par yield as "the coupon rate that causes the bond price to equal its par value" (Hull Ch 4). They are weighted averages of the discount curve.

### The Par Equation

For a bond paying coupon $C$ semiannually on dates $T_1, \dots, T_n$:

$$\text{Price} = C \sum_{i=1}^n \tau_i P(0,T_i) + 1 \cdot P(0,T_n) = 1$$

Solving for $C$:

$$\boxed{C_{\text{par}} = \frac{1 - P(0,T_n)}{\sum_{i=1}^n \tau_i P(0,T_i)}}$$

Hull (eq. 4.7, adapted) provides an equivalent form using continuous compounding, but the structure is identical.

### The Annuity

The denominator is called the **PV01** or the **Annuity** of the bond/swap:

$$A(0) = \sum_{i=1}^n \tau_i P(0,T_i)$$

This represents the present value of receiving 1 basis point (or 1 unit) of coupon payments.

### Par Swap Rates

This is exactly how swap rates are quoted. A "5-year Swap Rate" is simply the fixed rate that makes the PV of the fixed leg equal the PV of the floating leg. Since the floating leg is typically valued at par (initially), the swap rate is just the par coupon calculation above.

### Why Par Rates Matter

Par rates are the **observable market quotes**. You don't typically observe zero rates or discount factors directly—you observe Treasury yields (which are par-like) and swap rates (which are par rates). The bootstrap process (Chapter 17) inverts these par quotes to recover the underlying discount factors.

---

## 3.6 The Complete Triangle: Converting Between Representations

> **Visualizing the Triangle of Rates**
>
> Luenberger introduces the "Triangular Array" to visualize how today's curve implies the entire future.
>
> | | $T=1$ | $T=2$ | $T=3$ | $T=4$ |
> | :--- | :--- | :--- | :--- | :--- |
> | **Today ($t=0$)** | $z_1$ (Spot) | $z_2$ (Spot) | $z_3$ (Spot) | $z_4$ (Spot) |
> | **Future ($t=1$)** | | $f_{1,2}$ (Fwd) | $f_{1,3}$ | $f_{1,4}$ |
> | **Future ($t=2$)** | | | $f_{2,3}$ (Fwd)| $f_{2,4}$ |
> | **Future ($t=3$)** | | | | $f_{3,4}$ (Fwd)|
>
> *   **Row 1**: The Spot Curve observed today.
> *   **Diagonals**: The path of short-term rates implied by the Expectations Hypothesis.
> *   **Insight**: One spot curve implies an entire matrix of future forward rates.

We can now navigate the full map.

| From $\downarrow$ To $\rightarrow$ | **Discount Factors $P(0,T)$** | **Zero Rates $z_c(T)$** (Continuous) | **Par Rates $C_{par}$** |
| :--- | :--- | :--- | :--- |
| **Discount Factors** | — | $z_c = -\frac{\ln P}{T}$ | $C = \frac{1 - P(T_n)}{A(0)}$ |
| **Zero Rates** | $P = e^{-z_c T}$ | — | Sample $P(T_i)$ from curve, then use Par formula |
| **Par Rates** | **Bootstrapping** (See Ch 17) | Convert Par $\to$ DF $\to$ Zero | — |

*   **Discount Factors $\longleftrightarrow$ Zero Rates**: Simple algebraic inversion (Tuckman eq. 2.9, 2.10).
*   **Discount Factors $\longrightarrow$ Par Rates**: Weighted average calculation (Tuckman eq. 4.40).
*   **Par Rates $\longrightarrow$ Discount Factors**: Requires **Bootstrapping**.

> **Analogy: The Bootstrapping Ladder**
>
> You cannot reach the 5-year rung (5y Zero) without climbing past the 1, 2, 3, and 4-year rungs.
> *   To value a 5-year Par Bond, you first need to value its coupons at years 1, 2, 3, and 4.
> *   You must use the *already known* discount factors for years 1-4 to "strip out" those coupons.
> *   Only then is the 5-year principal exposed, allowing you to solve for the 5-year discount factor.
>
> This is why we call it *bootstrapping* (pulling yourself up step-by-step).

You cannot convert a single par rate to a single discount factor in isolation; you need the entire path of prior discount factors to strip out the coupons.

### Forward Rates in the Triangle

Forward rates can be derived from either discount factors or zero rates:
*   From discount factors: $F = \frac{1}{\tau}(\frac{P_1}{P_2} - 1)$ (Tuckman eq. 2.14)
*   From zero rates: $f_c = \frac{z_2 T_2 - z_1 T_1}{T_2 - T_1}$ (Tuckman eq. 2.16)

---

## 3.7 Worked Examples

### Example A: Discount Factor to Zero Rates (Multiple Conventions)

**Given:** $P(0, 1y) = 0.9700$.

1.  **Simple Zero:**
    $$z_s = \frac{1}{1} \left( \frac{1}{0.9700} - 1 \right) = 3.092\%$$
2.  **Continuous Zero:**
    $$z_c = -\frac{\ln(0.9700)}{1} = 3.046\%$$
3.  **Semiannual Zero:**
    $$z_{sa} = 2 \left( (1/0.9700)^{0.5} - 1 \right) = 2(1.01532 - 1) = 3.065\%$$

**Sanity Check:** Continuous < Semiannual < Simple. This ordering always holds for positive rates.

### Example B: Forward Rate Calculation

**Given:** $P(0, 1y) = 0.9615$ and $P(0, 2y) = 0.9157$.

**Objective:** Find the 1-year forward rate starting in 1 year ($1y \times 1y$). $\tau = 1$.

**Simple forward rate:**
$$F = \frac{1}{1} \left( \frac{0.9615}{0.9157} - 1 \right) = 1.0500 - 1 = 5.00\%$$

**Continuous forward rate:**
$$f_c = \frac{\ln(0.9615) - \ln(0.9157)}{1} = \frac{-0.0393 - (-0.0881)}{1} = 4.88\%$$

**Verification:** Check that rolling over gives the same result as investing for 2 years:
- Invest for 2 years: $1 \times 0.9157 = 0.9157$
- Invest for 1 year, then forward: $1 \times 0.9615 \times \frac{1}{1.05} = 0.9157$ ✓

### Example C: Calculating a Forward Rate (Negative Rate Case)

**Given:** $P(0, 1y) = 1.0020$ and $P(0, 1.5y) = 1.0030$.
*Note: $P > 1$ implies negative interest rates.*

**Objective:** Find the 6-month forward rate $6 \times 12$ (starting in 1y, ending 1.5y). $\tau = 0.5$.

$$1 + F \cdot \tau = \frac{P(0, 1y)}{P(0, 1.5y)} = \frac{1.0020}{1.0030} \approx 0.999003$$

$$F = \frac{0.999003 - 1}{0.5} \approx -0.199\%$$

**Interpretation:** The forward rate is negative. You would *pay* interest to lend money for this period. The math holds seamlessly.

### Example D: Par Rate Calculation

**Given:** Discount Factors: 0.5y = 0.99, 1.0y = 0.97.
**Objective:** Calculate the 1-year semiannual par rate $C$.

1.  **Calculate Annuity ($A$):**
    $$A = 0.5 \times 0.99 + 0.5 \times 0.97 = 0.495 + 0.485 = 0.980$$
2.  **Calculate Par Numerator:**
    $$1 - P(0, 1y) = 1 - 0.97 = 0.03$$
3.  **Solve:**
    $$C = \frac{0.03}{0.980} \approx 0.03061 \implies 3.061\%$$

**Sanity Check:** The par rate should be between the 6-month and 1-year zero rates. Check: 6-month zero ≈ 2.02%, 1-year zero ≈ 3.09%. Par rate 3.061% is in range. ✓

### Example E: Spot-Forward Relationship

**Given:** Zero rate curve $z(T) = 0.03 + 0.005T$ (continuous compounding, linear in $T$).

**Objective:** Find the instantaneous forward rate curve $f(T)$.

Using $f(T) = z(T) + T \cdot z'(T)$:
- $z'(T) = 0.005$
- $f(T) = (0.03 + 0.005T) + T \cdot 0.005 = 0.03 + 0.01T$

**Interpretation:** The forward curve has twice the slope of the zero curve. At $T=5$: $z(5) = 5.5\%$ but $f(5) = 8\%$.

---

## 3.8 Practical Notes

### 1. The "Wiggle" Problem

If you linearly interpolate zero rates, your forward curve will be piecewise constant (steps). If you linearly interpolate discount factors, your forward curve will have massive saw-tooth jumps.

**Lesson:** Curve construction method choice (splines vs linear) determines the usability of your forward rates. See Chapter 17 for detailed treatment.

### 2. Negative Rates

As shown in Example C, negative rates are handled naturally by the $P(T)$ framework. Do not use code that assumes $z > 0$ or uses log-normal rate models (like Black-76) without shifts.

### 3. Bid-Ask Spreads

In practice, you have a **Bid** curve and an **Ask** curve.
*   **Asset Valuation (Long)**: If you are valuing a long position you want to sell, market makers pay the **Bid** price. You discount cash flows at the (higher) Bid yield.
*   **Liability Valuation (Short)**: If you need to cover a short, you pay the **Ask** price. You discount at the (lower) Ask yield.

> **Rule of Thumb**: Books are often marked to **Mid** for daily P&L, but reliable exit value is Bid (for assets) or Ask (for liabilities).

Always ensure your curve logic accounts for the side of the trade!

### 4. Day Count Conventions in Forward Rates

The year fraction $\tau$ in forward rate calculations must use the appropriate day count convention:
- Money market rates: ACT/360 (USD, EUR) or ACT/365 (GBP)
- Bond rates: ACT/ACT or 30/360

Mismatched day counts are a common source of small pricing discrepancies.

---

## Summary

1.  **Discount Factors ($P(0,T)$)** are the primitive, risk-free price of future money. In modern markets, these come from the OIS curve.
2.  **Zero Rates** are just discount factors expressed in per-annum terms. They require a **compounding convention** to be meaningful. Same DF → different rate numbers.
3.  **Forward Rates** are the rates for future periods implied by the term structure to prevent arbitrage. They reflect the slope of the zero curve and can be locked in via synthetic borrowing/lending.
4.  **Par Rates** are the coupons that value a bond/swap at par. They are annuity-weighted averages of the forward curve.
5.  **The Triangle:** You can translate freely between these forms. Quants use DFs; Traders use Forwards/Pars; Risk uses Zeros. The key equations are:
    - $P = e^{-zT}$ (DF from zero)
    - $z = -\ln(P)/T$ (zero from DF)
    - $F = (P_1/P_2 - 1)/\tau$ (forward from DFs)
    - $f(T) = z(T) + Tz'(T)$ (instantaneous forward from zero curve)
    - $C_{par} = (1 - P_n) / A$ (par from DFs)

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
| :--- | :--- | :--- |
| **Discount Factor** | $P(0,T) = \text{PV of } \$1$ | The universal valuation primitive. OIS-based. |
| **Zero Rate (Spot)** | Rate solving $P = e^{-zT}$ or $(1+z/m)^{-mT}$ | Useful for visualizing term structure levels. Convention-dependent. |
| **Forward Rate** | $F = \frac{1}{\tau}(\frac{P_1}{P_2}-1)$ | Used to project floating cashflows; implies "No Arbitrage". Can be locked in. |
| **Instantaneous Forward** | $f(T) = -\frac{\partial}{\partial T}\ln P(0,T)$ | Foundation of HJM framework; drives curve smoothness requirements. |
| **Par Rate** | $C = \frac{1-P_n}{\text{Annuity}}$ | The language of the market (Swap/Bond quotes). |
| **Annuity (PV01)** | $\sum \tau_i P(T_i)$ | Sensitivity to parallel rate shifts; denominator for par rates. |
| **Spot-Forward Relation** | $f = z + Tz'$ | Links curve shape to forward rates; marginal vs average intuition. |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $P(0,T)$ or $d(T)$ | Discount factor for maturity $T$ |
| $z_c(T)$ | Continuously compounded zero rate |
| $z_{sa}(T)$ | Semiannually compounded zero rate |
| $z_s(T)$ | Simple (money market) zero rate |
| $f(0;T_1,T_2)$ | Forward rate from $T_1$ to $T_2$ |
| $f(0,T)$ | Instantaneous forward rate at $T$ |
| $C_{par}$ | Par coupon rate |
| $A(0)$ | Annuity (PV01) |
| $\tau$ | Year fraction ($T_2 - T_1$) |

---

## Flashcards

| # | Question | Answer |
|---|---|---|
| 1 | What is the fundamental difference between a Zero Rate and a Par Rate? | Zero rate applies to a single cashflow (terminal); Par rate applies to a series of cashflows (coupon/annuity). |
| 2 | What is the formula for a continuous zero rate given $P(0,T)$? | $z_c = -\frac{\ln P(0,T)}{T}$ |
| 3 | If the Zero Curve is upward sloping, is the Forward Rate higher or lower than the Zero Rate? | Higher. Marginal > Average to pull the average up. |
| 4 | How do you calculate a simple forward rate from $T_1$ to $T_2$? | $F = \frac{1}{\tau} (\frac{P(T_1)}{P(T_2)} - 1)$ |
| 5 | What is the standard discount curve in post-2008 markets? | The OIS (Overnight Indexed Swap) curve. |
| 6 | Why are forward rates "wigglier" than zero rates? | Forward rates depend on the derivative (slope) of the zero curve; differentiation amplifies noise. |
| 7 | What is the "Annuity" in the context of par rates? | The PV of receiving 1 unit of currency per period: $\sum \tau_i P(T_i)$. |
| 8 | Can Discount Factors be greater than 1? | Yes, if interest rates are negative. |
| 9 | What arbitrage strategy enforces the forward rate? | Borrowing long and lending short (or vice versa) to lock in a future rate. |
| 10 | Why is Linear Interpolation of Discount Factors dangerous? | It creates discontinuous forward rates, leading to pricing anomalies in forward-starting products. |
| 11 | What is the instantaneous forward rate $f(T)$ mathematically? | $f(T) = -\frac{\partial}{\partial T} \ln P(0,T)$ |
| 12 | How do you recover $P(0,T)$ from instantaneous forwards? | $P(0,T) = \exp(-\int_0^T f(0,u)\,du)$ |
| 13 | State the spot-forward relationship for continuous compounding. | $f(T) = z(T) + T \cdot z'(T)$ |
| 14 | Convert 5% continuous to semiannual compounding. | $R_{sa} = 2(e^{0.05/2} - 1) \approx 5.06\%$ |
| 15 | If the zero curve is flat at 4%, what is the forward rate? | 4% — flat curve means forward equals spot. |
| 16 | Given $P(1) = 0.95$ and $P(2) = 0.89$, what is the 1y×1y simple forward? | $F = (0.95/0.89 - 1)/1 = 6.74\%$ |
| 17 | What does a negative forward rate mean economically? | You pay interest to lend money (lender pays borrower). |
| 18 | Why can't you convert a single par rate to a single discount factor? | Par rates depend on multiple DFs (the annuity). You need the full path via bootstrapping. |

---

## Mini Problem Set

1.  **Conversion:** $P(0, 2y) = 0.92$. Calculate the continuous zero rate and the semiannual zero rate.

2.  **Forward:** $P(0, 1y) = 0.98, P(0, 2y) = 0.95$. Calculate the simple forward rate for the 2nd year ($1y \times 1y$).

3.  **Par Rate:** A 2-year annual-pay bond has DFs: $P(1)=0.98, P(2)=0.95$. Calculate the Par Coupon.

4.  **Arbitrage:** You see a quoted forward rate of 4% for year 2. Your calculated forward rate from the curve is 3%. Describe the arbitrage trade.

5.  **Slope:** The zero curve is flat at 5%. What is the instantaneous forward rate?

6.  **Negative Rates:** $P(0, 1) = 1.01$. What is the simple zero rate? What is the continuous zero rate?

7.  **Annuity**: Calculate the PV01 of a 2-year annual bond if $P(1)=0.95, P(2)=0.90$.

8.  **Spot-Forward:** If $z(T) = 0.05 + 0.01T$ (continuous), find $f(T)$.

9.  **Convention:** Convert 5% continuous to semiannual.

10. **Bootstrapping Logic:** If you know the 1y par rate and 2y par rate, can you find $P(0,2)$? Explain the steps.

11. **Instantaneous Forward:** Given $P(0,T) = e^{-0.04T - 0.002T^2}$, find $f(0,T)$.

12. **Integral Form:** If $f(0,T) = 0.03 + 0.01T$, find $P(0,3)$ using the integral formula.

### Solutions

1. **Conversion**: $z_c = -\ln(0.92)/2 = 0.0834/2 = 4.17\%$. $z_{sa} = 2((1/0.92)^{1/4} - 1) = 2(1.0210 - 1) = 4.21\%$.

2. **Forward**: $F = (0.98/0.95 - 1)/1 = 0.0316 = 3.16\%$.

3. **Par Rate**: $A = 1 \times 0.98 + 1 \times 0.95 = 1.93$. $C = (1 - 0.95)/1.93 = 0.05/1.93 = 2.59\%$.

4. **Arb**: Market forward (4%) is too high relative to curve (3%). Strategy: "Lend" at the market forward (buy FRA/receive fixed), "Borrow" at the theoretical forward (via synthetic using curve). You receive 4%, pay ~3%, profit ~1%.

5. **Slope**: 5%. Flat curve means $z'=0$, so $f = z + T \cdot 0 = z = 5\%$.

6. **Negative**: Simple: $z_s = (1/1.01 - 1)/1 = -0.99\%$. Continuous: $z_c = -\ln(1.01)/1 = -0.995\%$.

7. **Annuity**: $A = 1 \times 0.95 + 1 \times 0.90 = 1.85$.

8. **Spot-Forward**: $f(T) = (0.05+0.01T) + T(0.01) = 0.05 + 0.02T$.

9. **Convention**: $R_{sa} = 2(e^{0.05/2} - 1) = 2(1.0253 - 1) = 5.06\%$.

10. **Bootstrapping**: Yes. Step 1: Use 1y par rate $C_1 = (1-P_1)/(P_1)$ to solve for $P(1)$ (assuming annual, $P_1 = 1/(1+C_1)$). Step 2: Use 2y par equation: $1 = C_2 \cdot P_1 + C_2 \cdot P_2 + P_2 = C_2(P_1 + P_2) + P_2$. Rearrange: $P_2 = (1 - C_2 \cdot P_1)/(1 + C_2)$.

11. **Instantaneous Forward**: $f(0,T) = -\frac{d}{dT}(-0.04T - 0.002T^2) = 0.04 + 0.004T$.

12. **Integral Form**: $\int_0^3 f(0,u)\,du = \int_0^3 (0.03 + 0.01u)\,du = [0.03u + 0.005u^2]_0^3 = 0.09 + 0.045 = 0.135$. So $P(0,3) = e^{-0.135} = 0.8737$.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| Discount Factor Definition | Tuckman Ch 1: "the present value of $1 to be received at time t" |
| Zero Rate (Continuous) Formula | Hull Ch 4, eq. 4.3, 4.5; Tuckman Ch 2, eq. 2.9 |
| Zero Rate (Semiannual) Formula | Tuckman Ch 2, eq. 2.10 |
| Compounding Conversion | Hull Ch 4, eq. 4.3, 4.4 |
| Forward Rate (Simple) | Tuckman Ch 2, eq. 2.14 |
| Forward Rate (Continuous) | Hull Ch 4, eq. 4.5; Tuckman Ch 2, eq. 2.16 |
| Forward Rate (Semiannual) | Tuckman Ch 2, eq. 2.17 |
| Instantaneous Forward Definition | Brigo & Mercurio Ch 1, Definition 1.4.2; Hull Ch 4, eq. 4.6 |
| Integral Form for Discount Factor | Brigo & Mercurio Ch 1, following Definition 1.4.2 |
| Forward Rate Locking Strategy | Hull Ch 4, "Forward Rates" section |
| Spot-Forward Relationship $f=z+Tz'$ | Tuckman Appendix 2A |
| Par Rate Formula | Tuckman eq. 4.40; Hull Ch 4 (par yield definition) |
| No-Arbitrage Requirements | Tuckman Ch 1 (positivity, monotonicity) |

### (B) Reasoned Inference (Derived from A)

*   **OIS as Universal Standard**: Derived from modern practice (post-CSA discounting) as documented in Andersen & Piterbarg Vol 1 Ch 6. Standard texts now mention OIS; the "universal" nature is a practitioner synthesis of industry evolution.
*   **Wigglier Forwards**: Mathematical consequence of differentiation—if $f = z + Tz'$, then noise in $z$ is amplified in $f$.
*   **Ordering of Rates by Convention**: For positive rates, continuous < discrete compounding follows from convexity of exponential function.

### (C) Flagged Uncertainties

*   **Simple Interest Day Counts**: Specific ACT/360 vs ACT/365 conventions depend on the currency (USD vs GBP), not specified here as universal. Consult market-specific documentation.
*   **Par Yield vs Par Coupon**: Hull uses "par yield" while Tuckman uses "par coupon"—these are equivalent concepts but terminology varies across texts.
