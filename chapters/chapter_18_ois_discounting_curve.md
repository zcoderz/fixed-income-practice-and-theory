# Chapter 18: OIS Discounting Curve — Building the "Risk-Free-ish" Curve

---

## Introduction

In the years before 2008, fixed income practitioners lived in a convenient fiction: they assumed that "risk-free" rates were observable in the interbank market, and that a single yield curve could serve both for forecasting cashflows and for discounting them. LIBOR was the universal standard, used to price everything from mortgages to complex derivatives.

The 2008 financial crisis shattered this consensus. As banks failed, the spread between LIBOR (an unsecured lending rate) and OIS (a geometric average of overnight rates) exploded from a negligible few basis points to over 350 basis points. The market realized an uncomfortable truth: LIBOR embeds credit risk. Discounting a risk-free (or collateralized) cashflow at LIBOR effectively undervalues it, as if the cashflow itself were subject to bank default risk.

For collateralized trades backed by cash margin, the economically correct discount rate is not LIBOR, but the rate earned on that collateral—typically the overnight rate. This realization forced a wholesale re-architecture of valuation frameworks, moving from a "single-curve" world to a "multi-curve" world where the discounting curve is separated from the projection curve.

> **Analogy: Gold vs. Silver Currency**
>
> Imagine a country with two currencies: Gold (fully secured, 0% risk) and Silver (unsecured, can be devalued).
> *   **OIS represents Gold**: It is backed by cash collateral. If your counterparty vanishes, you have the cash.
> *   **LIBOR represents Silver**: It is an unsecured promise. If the bank vanishes, you are an unsecured creditor.
> *   **The Mistake**: Pre-2008, we treated Silver and Gold as trading 1:1.
> *   **The Reality**: When the world ended in 2008, Silver collapsed. We realized we should never value Gold assets using Silver rates.

This chapter guides you through constructing that discounting backbone: the **OIS Curve**. We will cover:

1. **The Instrument** (Section 18.1): What an OIS actually exchanges and how it differs from a standard swap.
2. **The Par Condition** (Section 18.2): How to mechanically translate market quotes into discount factor constraints.
3. **The Bootstrap** (Section 18.3): A step-by-step algorithm to extract the discount curve from par rates.
4. **Zero Rates** (Section 18.4): Converting discount factors to the rate representation humans prefer.
5. **Interpolation** (Section 18.5): The art of filling the gaps between market quotes, and the implications for forward rates.
6. **Locality** (Section 18.6): How quote bumps propagate through the curve.
7. **Valuation Impact** (Section 18.7): Demonstrating the P&L difference between OIS and LIBOR discounting.
8. **Practical Considerations** (Section 18.8): Turn-of-year effects and implementation pitfalls.

Mastering the OIS curve is no longer optional. It is the baseline for all modern derivatives pricing, the foundation upon which every other curve (LIBOR, SOFR, BSBY) is built.

---

## 18.1 The Overnight Indexed Swap: Anatomy of the Instrument

### 18.1.1 What Gets Exchanged

An Overnight Indexed Swap (OIS) is an interest rate swap where one leg pays a fixed rate $k$, and the other pays a floating rate based on a daily compounded overnight index (such as the Fed Funds Rate or SOFR).

Unlike a standard LIBOR swap where the floating rate is set at the *beginning* of the period, the floating payment in an OIS is determined at the *end*, based on the geometric average of daily rates realized over the period. Hull emphasizes this distinction: "LIBOR rates for a period are known at the beginning of the period to which they apply, whereas the result of the averaging process for overnight rates is known only at the end of the period."

Andersen and Piterbarg describe the floating leg payment for a period $[T_n, T_{n+1}]$ as being driven by the compounded rate $\bar{L}_n$:

$$\boxed{\bar{L}_n = \frac{\prod_{j=1}^{\ell_n}\left(1 + r(T_n + t_j^n)\Delta t_j^n\right) - 1}{\tau_n}}$$

where:
- $r(T_n + t_j^n)$ is the overnight rate on day $j$.
- $\Delta t_j^n$ is the accrual fraction for that day (e.g., 1/360 for most days, but 3/360 for a rate applied over a weekend).
- $\tau_n$ is the total year fraction for the payment period.

This formula captures the economic reality of rolling over an overnight deposit daily. The "minus 1" removes the principal, leaving just the interest, and dividing by $\tau_n$ annualizes the rate for the period.

Hull provides a concrete example: "Suppose that the (annualized) SOFR overnight rate on the $i$th business day of a period is $r_i$ and the rate applies to $d_i$ days. The (annualized) interest rate for the period is:

$$\left[\left(1+r_1 \hat{d}_1\right)\left(1+r_2 \hat{d}_2\right) \ldots\left(1+r_n \hat{d}_n\right)-1\right] \times \frac{360}{D}$$

where $\hat{d}_i = d_i/360$ and $D = \sum_i d_i$ is the number of days in the period."

The net payment at $T_{n+1}$ for a notional $N$ is:

$$\boxed{\text{Payoff at } T_{n+1} = N \tau_n (\bar{L}_n - k)}$$

### 18.1.2 Contract Structure by Maturity

Hull clarifies the structure of OIS contracts across different maturities: "When the life of the OIS is greater than one year, it is typically divided into three-month subperiods, with the fixed rate being exchanged at the end of each three-month period for the three-month reference rate that is calculated for that period from one-day rates. OISs lasting ten years or longer are now traded."

For shorter maturities, the structure is simpler:

| Maturity | Structure |
|----------|-----------|
| 1 Month | Single exchange at maturity |
| 3 Months | Single exchange at maturity |
| 6 Months | Single exchange at maturity |
| 1 Year | Single exchange at maturity |
| 2+ Years | Quarterly exchanges |

### 18.1.3 Why OIS Became the Discounting Standard

The prominence of OIS stems from the mechanics of collateral. In modern OTC derivatives, counterparties post variation margin (typically cash) to cover daily mark-to-market changes. This cash usually earns the overnight rate (e.g., Fed Funds or SOFR).

Hull explains the arbitrage argument concisely: derivatives that are collateralized by cash earning the overnight rate should be discounted at that overnight rate. As he notes, "overnight reference rates are considered to be better proxies for risk-free rates than Treasury rates."

This leads to the **separation of discount and forward curves** (discussed in detail in Andersen & Piterbarg, Vol 1, Section 6.5.2.2). The OIS curve $P(0,T)$ is used for discounting *all* collateralized cashflows, regardless of whether those cashflows themselves are linked to LIBOR, SOFR, or FX rates.

> **Visual: The Crisis Gap**
>
> In 2007, the spread between 3-month LIBOR and OIS was ~5 basis points. It was noise.
> In 2008, that spread hit **350 basis points**.
>
> *   **Pre-2008**: OIS $\approx$ LIBOR. Using LIBOR to discount was "close enough."
> *   **Post-2008**: OIS $\ll$ LIBOR. Using LIBOR to discount collateralized cash flows meant throwing away 3% of the value per year.
>
> This gap forced the "Dual Curve" shift:
> 1.  **Project** flows using LIBOR (because the contract says so).
> 2.  **Discount** flows using OIS (because the CSA collateral earns OIS).

> **Practitioner Note**: While we often call this the "risk-free" curve, practitioners know it as the "collateral curve." It reflects the specific terms of the Credit Support Annex (CSA). A trade collateralized in USD cash uses the USD OIS curve; a trade collateralized in EUR uses the €STR curve. The choice of discounting curve is determined by the CSA, not by the underlying instrument.

---

## 18.2 The Par Condition: OIS as a Par Bond

### 18.2.1 From Swaps to Bonds

To build the curve, we rely on the fundamental pricing equation: at inception, a par swap has a present value of zero. This implies the present value of the fixed leg equals the present value of the floating leg.

Hull provides the key insight that allows us to treat the floating leg simply. By adding notional principal exchanges (which net to zero), we can view the swap as an exchange of bonds: "A key point is that the floating-rate bond... is worth [$N$]. This is because it provides the payments necessary to service [$N$] of borrowings at overnight rates."

A par OIS can thus be conceptualized as two bonds:
1. **Fixed Rate Bond**: Pays coupons $k$ at times $T_i$.
2. **Floating Rate Bond**: Pays compounded overnight rates—and prices at par.

This allows us to write the **par condition** solely in terms of the fixed leg and the discount factors:

$$\boxed{1 = k\sum_{i=1}^{N} \tau_i P(0,T_i) + P(0,T_N)}$$

Here, the left side (1) represents the floating leg value, and the right side represents the fixed leg coupons plus the return of notional at maturity $T_N$.

As Hull confirms: "the OIS rate is the interest rate on a fixed-rate bond that is worth par."

### 18.2.2 The Annuity Factor

The sum of the discounted accrual factors is a recurring quantity known as the **annuity factor** (or PV01 of the fixed leg). Andersen and Piterbarg define it precisely:

$$\boxed{A(0) \equiv \sum_{i=1}^{N} \tau_i P(0,T_i)}$$

This quantity represents "the time $t$ value of a security making $m$ coupon payments of $\tau_n$ at all $T_{n+1}$." Using this shorthand, the par rate $k_{\text{par}}$ can be expressed elegantly as:

$$\boxed{k_{\text{par}} = \frac{1 - P(0,T_N)}{A(0)}}$$

This ratio has a physical interpretation: the numerator is the total discount (time value lost) to maturity, and the denominator distributes this loss across the coupon payment dates. Andersen and Piterbarg express this same relationship as:

$$S_{k,m}(t) = \frac{P(t, T_k) - P(t, T_{k+m})}{A_{k,m}(t)}$$

where $S_{k,m}(t)$ is the forward swap rate.

---

## 18.3 Bootstrapping: From Par Quotes to Discount Factors

### 18.3.1 The Sequential Algorithm

We cannot solve for all discount factors simultaneously because the equation for a 5-year swap depends on the discount factors for years 1, 2, 3, and 4. Instead, we **bootstrap**: we solve for the curve one maturity at a time, starting from the shortest tenor.

As Hull describes, "OIS rates with maturities one year or less, because they lead to just one exchange, have a straightforward interpretation. They provide the risk-free zero rates that are equivalent to the underlying overnight rates."

**Step 1: Short End (Zero Rates)**
For short-dated OIS (e.g., 1 month, 3 months), there is only one payment. The quoted rate $R$ acts as a simple zero rate:

$$P(0,T) = \frac{1}{1 + R \cdot T}$$

Hull's Table 7.3 provides an example: "The OIS rates out to one year define zero rates in a direct way."

**Step 2: Long End (Iterative Solve)**
For an $N$-year swap, assuming we have already found the discount factors $P(0,T_1) \dots P(0,T_{N-1})$, we isolate the single unknown $P(0,T_N)$ in the par equation:

$$1 = k_N \left(\sum_{i=1}^{N-1} \tau_i P(0,T_i) + \tau_N P(0,T_N)\right) + P(0,T_N)$$

Solving for $P(0,T_N)$ yields the **bootstrap formula**:

$$\boxed{P(0,T_N) = \frac{1 - k_N \sum_{i=1}^{N-1} \tau_i P(0,T_i)}{1 + k_N \tau_N}}$$

As Hull explains, for OIS rates with maturities greater than a year, "the zero curve can be assumed to be linear between maturities and calculations can be carried out." The two-year and five-year zero rates are "chosen using an iterative search procedure... so that they are consistent with" bonds of those maturities pricing at par.

### 18.3.2 Worked Example: Building a 5-Year Curve

Let's apply this to a concrete set of market quotes. Assume annual payments ($\tau=1$) for simplicity.

**Market Quotes:**
| Maturity | Par Rate |
|----------|----------|
| 1Y | 2.50% |
| 2Y | 2.70% |
| 3Y | 2.90% |
| 4Y | 3.05% |
| 5Y | 3.20% |

**First Calculation (1Y):**
The 1Y instrument has a single flow. We solve directly:

$$P(0,1) = \frac{1}{1 + 0.025} = 0.975610$$

**Second Calculation (2Y):**
Using the 1Y discount factor we just found, we solve for the 2Y factor. The 2.70% swap pays coupons at year 1 and year 2.

$$P(0,2) = \frac{1 - 0.027(0.975610)}{1 + 0.027} = \frac{0.973659}{1.027} = 0.948061$$

**Third Calculation (3Y):**
Now utilizing both previous factors, we solve for the 3Y factor. The 2.90% swap pays at years 1, 2, and 3.

$$P(0,3) = \frac{1 - 0.029(0.975610 + 0.948061)}{1 + 0.029} = \frac{0.944214}{1.029} = 0.917603$$

**Continuing the Chain (4Y & 5Y):**
By repeating this pattern:

| Maturity | Discount Factor |
|----------|-----------------|
| 4Y | 0.886309 |
| 5Y | 0.853408 |

The result is a consistent curve where every input instrument reprices exactly to par.

---

## 18.4 Zero Rates: Another View of the Same Curve

While the discount factor $P(0,T)$ is the fundamental pricing object, humans find it hard to differentiate "0.9176" from "0.9180". We prefer rates.

We can convert any discount factor into a **zero rate** (yield). Hull notes that zero rates are "often referred to as spot rates, or sometimes as zero-coupon rates." The two most common conventions are:

**1. Continuously Compounded:**
$$z_c(T) = -\frac{\ln P(0,T)}{T}$$

**2. Annually Compounded:**
$$z_a(T) = \frac{1}{P(0,T)^{1/T}} - 1$$

Applying this to our bootstrapped points:

| Maturity | $P(0,T)$ | Continuous Zero $z_c$ | Annual Zero $z_a$ |
|----------|----------|-----------------------|-------------------|
| 1Y | 0.975610 | 2.47% | 2.50% |
| 2Y | 0.948061 | 2.67% | 2.70% |
| 3Y | 0.917603 | 2.87% | 2.91% |
| 4Y | 0.886309 | 3.02% | 3.06% |
| 5Y | 0.853408 | 3.17% | 3.22% |

### 18.4.1 Par Rates vs Zero Rates

Notice that the zero curve ($z$) is generally *not* the same as the par curve ($k$). For an upward sloping yield curve:

- The par rate is a *weighted average* of the zero rates, weighted by discount factors
- The early coupons are discounted at lower rates (since the curve slopes upward)
- Therefore, the par rate is pulled down by the lower early rates
- The zero rate at maturity $T$ must be *higher* than the par rate at $T$

In our example: 5Y Par = 3.20%, 5Y Annual Zero = 3.22%. The zero rate exceeds the par rate because the par rate averages in the lower rates at earlier maturities.

---

## 18.5 Interpolation: Filling the Gaps

Market quotes are sparse (1Y, 2Y... 10Y, 30Y). But you might need to value a cashflow at 2.5 years. This requires **interpolation**. This is not just a mathematical nuisance; the choice of interpolation method dictates the shape of your forward rates and can impact the valuation of sensitive off-node risks.

### 18.5.1 Linear in Log Discount Factor

A standard practitioner choice (recommended by Andersen & Piterbarg) is **linear interpolation of the log discount factor**:

$$\ln P(0,t) = (1-\alpha) \ln P(0,T_1) + \alpha \ln P(0,T_2)$$

where $\alpha = (t - T_1)/(T_2 - T_1)$ is the fraction of time between $T_1$ and $T_2$.

This method has a crucial physical property: it implies **piecewise constant forward rates**. As Andersen and Piterbarg note, under the assumption that "the instantaneous forward curve is piecewise flat, switching to a new level at each point in $\{T_i\}$":

$$P(T) = P(T_i) e^{-f(T_i)(T-T_i)}$$

The forward rate is flat between grid points. This is robust and creates stable local sensitivities, which is why it is preferred for trading systems over splines (which can oscillate).

### 18.5.2 Why It Matters: Comparing Methods

If we interpolate our 2Y ($P=0.9481$) and 3Y ($P=0.9176$) points to 2.5 years:

**1. Log-Linear (Constant Forward):**
- Forward rate $f \approx 3.26\%$
- $P(0,2.5) \approx 0.9327$

**2. Linear Zero Rates:**
- Averages the zeros: $(2.67\% + 2.87\%)/2 = 2.77\%$
- $P(0,2.5) = e^{-0.0277 \times 2.5} \approx 0.9332$

The difference (approximately 5 basis points in price) is significant for leveraged positions. More importantly, the *risk profile* differs. Under log-linear interpolation, a risk at 2.5Y is distributed strictly to the 2Y and 3Y nodes. Under spline methods, it might "leak" to the 5Y or 10Y nodes, creating hedging noise.

### 18.5.3 Higher-Order Methods

Andersen and Piterbarg catalog a hierarchy of smoothness:

| Method | Smoothness | Forward Curve |
|--------|------------|---------------|
| Piecewise Linear Yields | $C^0$ | Discontinuous forwards |
| Piecewise Flat Forwards | $C^0$ | Piecewise constant |
| Hermite Splines | $C^1$ | Continuous, but not smooth |
| Cubic Splines | $C^2$ | Smooth, but may oscillate |
| Tension Splines | $C^2$ | Smooth with controlled curvature |

For trading systems, the piecewise flat forward approach is often preferred despite its lack of smoothness, because it produces predictable, local risk behavior.

---

## 18.6 Locality: How Quote Bumps Propagate

A key feature of the bootstrap method is its sequential dependency. This creates a specific direction of causality:

**Short Term affects Long Term**: If the 1Y rate changes, it changes $P(0,1)$. Since $P(0,1)$ is used to solve for $P(0,2)$, the 2Y discount factor also changes (to maintain the 2Y par condition). The change cascades all the way to 30Y.

**Long Term does NOT affect Short Term**: If the 10Y rate changes, it only affects discount factors from 10Y onwards. The 2Y swap still prices correctly using the original 2Y inputs.

This "one-way" causality is distinct from global fitting methods (like Nelson-Siegel), where a change in a long-dated point can shift the entire curve. The bootstrap's **locality** makes it easier to hedge: to hedge a 10Y risk, you primarily trade the 10Y swap, without worrying that you are ruining your 2Y fit.

### 18.6.1 The Jacobian Perspective

Andersen and Piterbarg formalize this with the Jacobian matrix $\partial P / \partial k$, which shows how discount factors respond to changes in input rates. For a bootstrapped curve, this matrix is *lower triangular*: the $(i,j)$ entry (sensitivity of $P_i$ to rate $k_j$) is zero whenever $j > i$. This triangular structure is the mathematical expression of locality.

---

## 18.7 Discounting Impact: A Valuation Example

Why go through all this trouble? Why not just use LIBOR?

Consider a 5-year trade with annual coupons of 3%. Notional 100.

**OIS Discounting** (using our curve where 5Y discount = 0.8534):

$$PV = 3 \times (0.9756 + 0.9481 + 0.9176 + 0.8863 + 0.8534) + 100 \times 0.8534$$
$$PV = 3 \times 4.5810 + 85.34 = 13.74 + 85.34 = 99.08$$

**LIBOR Discounting** (assuming LIBOR is 50bps higher, so 5Y discount ≈ 0.8325):

$$PV = 3 \times (0.9633 + 0.9276 + 0.8934 + 0.8604 + 0.8325) + 100 \times 0.8325$$
$$PV = 3 \times 4.4772 + 83.25 = 13.43 + 83.25 = 96.68$$

The difference is approximately **2.4% of notional**. In the fixed income world, where we fight for fractions of a basis point, a 240 basis point discrepancy is enormous. It represents the entire profit margin of the trade many times over.

> **The $1 Million Mistake**
>
> On a $\$100,000,000$ swap book:
> *   Value with LIBOR Discounting: $\$96.68$ million.
> *   Value with OIS Discounting: $\$99.08$ million.
> *   **Difference**: $\$2.4$ million.
>
> If you were a bank in 2008 shifting to OIS discounting, you suddenly "found" (or lost) millions overnight just by changing the discount curve. This was the famous "CSA Discounting Switch."

If you are a bank posting cash collateral (earning OIS) but pricing your book using LIBOR, you are systematically underestimating the value of your assets (or liabilities). This specific mismatch was a primary driver of the P&L restatements banks had to make post-crisis.

---

## 18.8 Practical Considerations

### 18.8.1 Turn-of-Year (TOY) Effects

One practitioner nuance often missed in textbooks is the **Turn-of-Year effect**. Over the year-end regulatory reporting period, balance sheet capacity becomes scarce, and overnight borrowing rates often spike (or plummet, depending on excess liquidity).

A standard smooth curve will miss this spike. Andersen & Piterbarg (Vol 1, Section 6.5.1) describe using a specific **overlay curve** $\varepsilon_f(t)$ to handle these jumps:

$$f(t) = f_{smooth}(t) + \varepsilon_{TOY}(t)$$

Practitioners explicitly mark a "turn" spread (e.g., +20bps for Dec 31 to Jan 2) to ensure that short-dated swaps crossing the year-end price correctly. Without this, your curve calibration would force the "smooth" rate up for the entire surrounding month to match the price, distorting the valuation of everything else.

### 18.8.2 Fed Funds vs SOFR

The transition from Fed Funds to SOFR as the primary USD overnight rate has implications for curve construction. Tuckman notes that "the GC rate is typically below the fed funds target rate because loans through repurchase agreement are effectively secured by collateral, while loans in the fed funds market are not."

SOFR, being based on Treasury repo transactions, is generally lower and less volatile than Fed Funds. Practitioners building OIS curves must be clear about which overnight rate their curve references.

### 18.8.3 Implementation Checklist

| Check | What to Verify |
|-------|----------------|
| **Repricing** | Every input par swap should reprice to par (error < $10^{-10}$) |
| **Bump Stability** | Bump an input rate by 1bp; changes should be localized and reasonable |
| **Convention Exactness** | Year fractions ($\tau$) must match the specific OIS conventions (ACT/360 vs ACT/365) |
| **Negative Rates** | If rates go negative, ensure your formulas still produce valid discount factors |
| **Interpolation Artifacts** | Check forward rates between nodes for unrealistic spikes or dips |

Using the wrong day count is the most common "rookie error" in curve construction.

---

## Summary

- **OIS represents the collateral rate**: Post-2008, we use OIS curves for discounting because they reflect the funding cost of collateralized trades.
- **OIS as a par bond**: Hull's insight that the floating leg equals par allows us to write valuation in terms of fixed leg only.
- **Bootstrapping is sequential**: We strip discount factors one by one, ensuring each instrument prices to par.
- **Par Condition**: The core equation is $k_{\text{par}} = (1 - P_N)/A(0)$.
- **Interpolation matters**: Log-linear interpolation on discount factors is the industry standard for its stability and implied piecewise-constant forwards.
- **Locality protects hedging**: Changes to long-dated inputs don't affect short-dated discount factors.
- **Valuation impact is large**: Mixing up funding curves leads to massive valuation errors.
- **TOY overlays required**: Year-end rate spikes need explicit handling to avoid curve distortion.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **OIS** | Overnight Indexed Swap: fixed rate vs compounded overnight | The instrument defining the "risk-free" discounting curve |
| **Par Condition** | $1 = k A(0) + P(0,T_N)$ | The constraint used to solve for discount factors |
| **Annuity Factor** | $A(0) = \sum \tau_i P(0,T_i)$ | Represents the PV of a 1bp coupon stream; used in all swap math |
| **Bootstrap** | Sequential calibration algorithm | Allows extraction of a unique discount curve from par quotes |
| **Log-Linear Interpolation** | Linear in $\ln P(T)$ | Produces piecewise constant forwards; stable risk |
| **Locality** | Input changes affect only $T \ge T_{input}$ | Crucial for stable hedging and risk management |
| **TOY Effect** | Turn-of-Year rate spike | Requires a manual "overlay" to avoid distorting the smooth curve |
| **Floating Leg = Par** | A floating bond paying the discount rate prices at par | Fundamental insight enabling par swap valuation |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $P(0,T)$ or $P(T)$ | Discount factor from 0 to $T$ |
| $k$ or $k_N$ | Par swap rate for an $N$-period swap |
| $\tau_i$ | Year fraction for period $i$ |
| $A(0)$ | Annuity factor $\sum \tau_i P(0,T_i)$ |
| $z_c(T)$ | Continuously compounded zero rate |
| $z_a(T)$ | Annually compounded zero rate |
| $\bar{L}_n$ | Compounded overnight rate for period $n$ |
| $f(T)$ | Instantaneous forward rate at time $T$ |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Why is OIS used for discounting instead of LIBOR? | Collateral usually earns the OIS rate; discounting must match funding cost. |
| 2 | What is the "Par Condition" for a swap? | The fixed leg PV equals the floating leg PV (usually par, 1.0). |
| 3 | Define the Annuity Factor $A(0)$. | The sum of discount factors weighted by year fractions: $\sum P(0,T_i)\tau_i$. |
| 4 | How does bootstrapping work? | Solving for discount factors one maturity at a time, using previous results. |
| 5 | What is the formula for par rate given annuity and final DF? | $k = (1 - P(T_N))/A(0)$. |
| 6 | What does log-linear interpolation of DFs imply for forward rates? | Forward rates are piecewise constant (flat between nodes). |
| 7 | If the curve is upward sloping, is the zero rate higher or lower than the par rate? | Higher. The zero rate at $T$ exceeds the par rate because par rate averages in lower early rates. |
| 8 | What is the Turn-of-Year effect? | A spike in overnight rates around Dec 31 due to balance sheet constraints. |
| 9 | How does a bump in the 2Y rate affect the 1Y discount factor? | It doesn't. Causality flows from short to long (lower triangular Jacobian). |
| 10 | What is the simplest relation between discount factor and continuous zero? | $P(0,T) = e^{-z_c(T) \cdot T}$. |
| 11 | Why does a floating-rate bond paying overnight compound to par? | It provides exactly the payments needed to service borrowings at overnight rates. |
| 12 | What is the key difference between OIS and LIBOR swap mechanics? | OIS floating rate is known at end of period; LIBOR is set at beginning. |
| 13 | What does "locality" mean in bootstrapping? | Changing an input rate only affects discount factors at or beyond that maturity. |
| 14 | For a 1Y OIS, how do you get the discount factor from the quoted rate? | $P(0,1) = 1/(1 + R)$ since there's only one payment. |
| 15 | What is the bootstrap formula for $P(T_N)$? | $P(T_N) = [1 - k_N \sum_{i < N} \tau_i P(T_i)] / (1 + k_N \tau_N)$. |
| 16 | Why might spline interpolation be problematic for hedging? | Risk can "leak" to distant nodes, making hedge ratios unstable. |
| 17 | What should the repricing error be after curve construction? | Effectively zero (< $10^{-10}$). Every input instrument must reprice exactly. |
| 18 | What day count is commonly used for USD OIS? | ACT/360 for the floating leg. |
| 19 | How is SOFR different from Fed Funds? | SOFR is repo-based (secured), typically lower and less volatile than unsecured Fed Funds. |
| 20 | What is the valuation impact of using wrong discounting curve? | Potentially hundreds of basis points of NPV error on leveraged positions. |

---

## Mini Problem Set

**Problem 1** (Warm-up)
A 6-month OIS quotes at 4.00% (simple rate, ACT/360). There are 182 actual days.
(a) Calculate the discount factor $P(0, 0.5)$.
(b) What is the continuously compounded zero rate?

*Solution sketch:* (a) $P = 1/(1 + 0.04 \times 182/360) = 0.9801$. (b) $z_c = -\ln(0.9801)/(182/365) = 4.03\%$.

---

**Problem 2** (Bootstrap Mechanics)
Given the following par OIS rates (annual pay):
- 1Y: 3.00%
- 2Y: 3.50%

Calculate $P(0,1)$ and $P(0,2)$.

*Solution sketch:* $P(0,1) = 1/1.03 = 0.9709$. Then $P(0,2) = (1 - 0.035 \times 0.9709)/(1.035) = 0.9322$.

---

**Problem 3** (Par vs Zero)
You've bootstrapped: $P(0,1) = 0.98$, $P(0,2) = 0.95$, $P(0,3) = 0.91$.
(a) Calculate the 3Y par rate (annual pay).
(b) Calculate the 3Y continuously compounded zero rate.
(c) Which is higher, and why?

*Solution sketch:* (a) $k = (1 - 0.91)/(0.98 + 0.95 + 0.91) = 0.09/2.84 = 3.17\%$. (b) $z_c = -\ln(0.91)/3 = 3.14\%$. (c) With annual compounding, the zero rate would be $0.91^{-1/3} - 1 = 3.21\%$, which exceeds the par rate. The par rate is a weighted average pulled down by higher early discount factors.

---

**Problem 4** (Interpolation)
Given $P(0,2) = 0.9400$ and $P(0,3) = 0.9000$, find $P(0,2.5)$ using:
(a) Log-linear interpolation
(b) Linear interpolation of discount factors directly

*Solution sketch:* (a) $\ln P(0,2.5) = 0.5 \ln(0.94) + 0.5 \ln(0.90) = -0.0840$, so $P(0,2.5) = 0.9194$. (b) Linear: $P(0,2.5) = 0.5(0.94) + 0.5(0.90) = 0.9200$. Difference is small but affects forward rate calculation.

---

**Problem 5** (Locality Test)
You have a 5-point curve. You bump the 3Y par rate up by 1bp.
(a) Which discount factors change: $P(0,1)$, $P(0,2)$, $P(0,3)$, $P(0,4)$, $P(0,5)$?
(b) Explain why.

*Solution sketch:* (a) Only $P(0,3)$, $P(0,4)$, $P(0,5)$ change. (b) Bootstrap is sequential. The 1Y and 2Y swap equations don't include the 3Y rate. But the 4Y and 5Y swaps depend on $P(0,3)$, so when it changes, they must be re-solved.

---

**Problem 6** (Valuation Impact)
A 3-year zero-coupon bond pays 100 at maturity. You can discount at:
- OIS: $P(0,3) = 0.9100$
- LIBOR+50bp curve: $P(0,3) = 0.8965$

Calculate the PV difference and express it in bp of face value.

*Solution sketch:* OIS: PV = 91.00. LIBOR: PV = 89.65. Difference = 1.35, or 135 bp of face value.

---

**Problem 7** (Forward Rate Implication)
Given $P(0,2) = 0.9400$ and $P(0,3) = 0.9000$, calculate the 1-year forward rate starting in 2 years (simple, annual).

*Solution sketch:* $F(0;2,3) = [P(0,2)/P(0,3) - 1]/1 = [0.94/0.90 - 1] = 4.44\%$.

---

**Problem 8** (OIS Mechanics)
An OIS quotes at 5.00% (quarterly pay, 2-year maturity). At inception:
(a) What is the NPV?
(b) If the floating leg pays realized rates averaging 4.80% over the life, who benefits and by approximately how much (per 100 notional)?

*Solution sketch:* (a) NPV = 0 at inception by definition. (b) Fixed payer pays 5%, receives 4.80%, net outflow of 0.20% per year. Over 2 years on 100 notional, approximately 0.40 in undiscounted terms—fixed payer is worse off.

---

## Source Map

### (A) Verified Facts

| Fact | Source |
|------|--------|
| OIS floating rate is known at end of period, LIBOR at beginning | Hull, Ch 7 ("LIBOR rates for a period are known at the beginning...") |
| Floating-rate bond paying overnight rates is worth par | Hull, Ch 7.2 ("A key point is that the floating-rate bond... is worth [$N$]") |
| OIS rate is the rate on a par bond | Hull, Ch 7.2 ("the OIS rate is the interest rate on a fixed-rate bond that is worth par") |
| OIS rates define zero rates directly for maturities ≤ 1 year | Hull, Ch 7.2 ("OIS rates with maturities one year or less... provide the risk-free zero rates") |
| Separation of Discount and Forward Curves | Andersen & Piterbarg, Vol 1, Section 6.5.2.2 |
| Turn-of-Year Overlays | Andersen & Piterbarg, Vol 1, Section 6.5.1 |
| Piecewise flat forward rates from log-linear interpolation | Andersen & Piterbarg, Vol 1, Section 6.2.1.2 |
| OIS structure: single exchange ≤1Y, quarterly >1Y | Hull, Ch 7 ("When the life of the OIS is greater than one year...") |
| Annuity factor definition | Andersen & Piterbarg, Vol 1, Section 4.1.3 |
| Swap rate formula $S = (P_0 - P_N)/A$ | Andersen & Piterbarg, Vol 1, Eq 4.10 |
| GC rate typically below Fed Funds (secured vs unsecured) | Tuckman, Ch 15 |
| Overnight reference rates considered better risk-free proxies than Treasuries | Hull, Ch 4 (footnote) |
| Compounded overnight rate formula | Hull, Ch 4.2; Andersen & Piterbarg, Eq 5.7 |

### (B) Reasoned Inference

- **Par Rate vs Zero Rate Logic**: For upward-sloping curves, the par rate is a weighted average of zeros, so the final zero rate must exceed the par rate. This follows from the mathematical structure of the par equation.
- **Jacobian is Lower Triangular**: Follows from the bootstrap's sequential structure—each $P(T_i)$ depends only on inputs for $T \le T_i$.
- **Valuation Impact Example**: Constructed from the bootstrapped curve to demonstrate magnitude of discounting differences.

### (C) Flagged Uncertainties

- **Exact TOY Spread Magnitudes**: The +20bp example is illustrative. Actual turn spreads vary by year and market conditions. Sources discuss the mechanism but don't specify current market levels.
- **Index-Specific Day Counts**: Exact day counts (ACT/360 vs ACT/365) and lag conventions vary by currency (USD SOFR vs GBP SONIA vs EUR €STR). Implementation requires checking specific index definitions.
- **SOFR vs Fed Funds Spread**: The relationship between SOFR and Fed Funds varies with market conditions. I'm not sure about typical current spreads without more recent market data.
