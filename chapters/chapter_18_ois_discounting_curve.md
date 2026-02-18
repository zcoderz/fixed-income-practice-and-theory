# Chapter 18: OIS Discounting Curve — Building the "Risk-Free-ish" Curve

---

## Introduction

Prerequisites: [Chapter 2 — Time Value, Discount Factors, and Replication](chapters/chapter_02_time_value_discount_factors_replication.md); [Chapter 3 — Zero, Forward, and Par Rates (The Rate Triangle)](chapters/chapter_03_zero_forward_par_rates_triangle.md); [Chapter 17 — Curve Construction (Bootstrapping, Interpolation)](chapters/chapter_17_curve_construction_bootstrapping_interpolation.md); [Chapter 11 — DV01/PV01 Definitions and Computation](chapters/chapter_11_dv01_pv01_definitions_computation.md).

Follow-on: [Chapter 19 — Projection Curves (LIBOR, SOFR, Multi-Curve)](chapters/chapter_19_projection_curves_libor_sofr_multi_curve.md); [Chapter 22 — Multi-Curve Risk (Jacobians, Controlled Perturbations)](chapters/chapter_22_multi_curve_risk_jacobians.md); [Chapter 33 — Collateral Discounting (OIS)](chapters/chapter_33_collateral_discounting_ois.md); [Chapter 34 — XVA Overview](chapters/chapter_34_xva_overview.md).

## Learning Objectives
- Translate an OIS quote (par fixed rate) into discount-factor constraints.
- Bootstrap an OIS discount curve and convert discount factors to zero rates.
- Choose and sanity-check an interpolation method (and understand what it implies for forward rates).
- Use the OIS curve to discount collateralized cashflows and compute a clearly-defined DV01.
- Interpret “locality” and the Jacobian view of how quote bumps propagate through the curve.

Before the 2007–2009 crisis, many market models and systems used a single curve (often LIBOR-based) to both (i) project floating cashflows and (ii) discount them. During the crisis, the spread between LIBOR (an unsecured term funding rate) and OIS (a compounded overnight rate) widened dramatically—peaking in the high-300 bp range in October 2008—highlighting that LIBOR embeds bank credit and liquidity risk.

For cash-collateralized trades under an idealized CSA (daily variation margin, cash collateral, and a specified collateral remuneration rate), no-arbitrage arguments point to discounting at the collateral rate—typically an overnight index. This led to the modern “multi-curve” framework: separate curves for discounting (collateral rate) and for projecting the contract’s index cashflows.

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
8. **Practical Considerations** (Section 18.8): Turn-of-year effects, negative rates, collateral choices, margin costs, and the operational lifecycle.
9. **USD Overnight Benchmarks** (Section 18.9): Fed Funds OIS vs SOFR OIS, and why the benchmark choice matters for discounting.

Mastering the OIS curve is the baseline for modern derivatives pricing: the discounting backbone on top of which projection curves and basis spreads are layered.

---

## 18.1 The Overnight Indexed Swap: Anatomy of the Instrument

### 18.1.1 What Gets Exchanged

An Overnight Indexed Swap (OIS) is an interest rate swap where one leg pays a fixed rate $k$, and the other pays an overnight leg that reflects the **geometric average** of overnight rates over the coupon period (typically implemented via daily compounding).

Unlike a term IBOR coupon (known at the start of the accrual period), the overnight leg is an **averaging process**, so the realized coupon rate is only known at the end of the period.

For a coupon period $[T_n, T_{n+1}]$, let $T_n = t_{n,1} \lt t_{n,2} \lt \cdots \lt t_{n,K_n}=T_{n+1}$ be the business days that bracket the overnight fixings in the period. Let:
- $r_{n,i}$ be the overnight rate fixed at $t_{n,i}$, applied over $[t_{n,i}, t_{n,i+1}]$,
- $\delta_{n,i}$ be the day-count accrual fraction for that sub-interval (e.g., ACT/360; weekends/holidays appear as larger $\delta_{n,i}$),
- $\tau_n := \sum_{i=1}^{K_n-1}\delta_{n,i}$ be the accrual year-fraction for the coupon period.

Define the **compounded accrual factor** over the period as:

$$A^{\text{flt}}_n := \prod_{i=1}^{K_n-1}\left(1+r_{n,i}\delta_{n,i}\right).$$

The **annualized compounded rate** for the period is then:

$$\boxed{\bar{L}_n = \frac{A^{\text{flt}}_n - 1}{\tau_n}}$$

so the floating-leg coupon amount is $N\tau_n\bar{L}_n = N\left(A^{\text{flt}}_n-1\right)$.

**Mechanics intuition:** `A_flt_n` is the growth factor of USD 1 rolled overnight through the period. Each day you earn simple interest `r_{n,i} * delta_{n,i}`, and the product multiplies those daily growth factors. Weekends/holidays appear as larger $\delta_{n,i}$ (e.g., a Friday fixing may apply for 3 calendar days), so the same rate accrues interest for a longer interval.

**Checks (units + limiting cases):** $r_{n,i}$ is “per year” and $\delta_{n,i}$ is “years”, so `r_{n,i} * delta_{n,i}` is dimensionless and `A_flt_n` is unitless. If all fixings equal a constant $r$ and the period is short, then $A^{\text{flt}}_n \approx 1+r\tau_n$ and $\bar{L}_n \approx r$; compounding shows up only at order $r^2\tau_n$. Toy number: if $r=5\\%$ and $\tau_n=0.25$, then $\bar{L}_n \approx (e^{0.05\cdot 0.25}-1)/0.25 \approx 5.03\\%$ (about +3 bp from compounding).

The **net payment** at $T_{n+1}$ for notional $N$ (positive from the perspective of receiving floating / paying fixed) is:

$$\boxed{\text{Payoff at } T_{n+1} = N \tau_n (\bar{L}_n - k)}$$

### 18.1.2 Contract Structure by Maturity

In practice, the exact payment schedule is a **documentation / product / CCP convention**. What stays the same is the core mechanic: within each coupon period, the floating leg is computed from **daily compounding** of the overnight rate, and a net payment is exchanged at the period end (sometimes with a payment delay).

For shorter maturities, the structure is simpler:

| Maturity | Structure |
|----------|-----------|
| 1 Month | Single exchange at maturity |
| 3 Months | Single exchange at maturity |
| 6 Months | Single exchange at maturity |
| 1 Year | Single exchange at maturity |
| 2+ Years | Periodic exchanges (frequency depends on product/currency/CCP; commonly quarterly or annual) |

### 18.1.3 Why OIS Became the Discounting Standard

The prominence of OIS stems from the mechanics of collateral in modern OTC derivatives. When cash variation margin is transferred (especially in centrally cleared setups), the party posting margin typically **earns** interest and the party receiving margin typically **pays** interest on the funds transferred; this interest is often called **price alignment interest (PAI)**.

Under the idealized “clean CSA” setup (daily margining, cash collateral, and a specified collateral remuneration rate), a no-arbitrage argument points to discounting future collateralized cashflows at the **collateral rate**. In practice, that collateral rate is typically an overnight rate index, which is why OIS curves became the natural discounting backbone.

This leads to the **separation of discounting and projection**:
- **Discounting curve:** used to PV cashflows (often an OIS curve tied to the collateral index).
- **Projection curve:** used to forecast index-linked cashflows (e.g., term IBOR, a term SOFR curve, or another RFR projection curve).

> **Visual: The Crisis Gap**
>
> Prior to August 2007, the 3-month LIBOR–OIS spread was less than 10 bp.
> In early October 2008, it reached a peak around 364 bp.
>
> *   **Pre-2008**: LIBOR and OIS were close, so discounting differences were often small.
> *   **Stress**: when the spread is large, discounting on the wrong curve can materially change PV and risk (especially for long-dated cashflows).
>
> This gap forced the "Dual Curve" shift:
> 1.  **Project** flows using LIBOR (because the contract says so).
> 2.  **Discount** flows using OIS (because the CSA collateral earns OIS).

> **Desk Reality:** Traders often say “OIS curve” and mean “the discount curve implied by my collateral agreement.”
> **Common break:** Two systems can both label a curve “OIS” but use different collateral indices/calendars/delays, producing PV and DV01 mismatches.
> **What to check:** For each trade, confirm the CSA/clearing setup (collateral currency + remuneration/index + timing). Then confirm which curve label in risk reports maps to those terms.

### 18.1.4 SOFR-Specific Conventions

USD OIS trades may reference different overnight benchmarks (e.g., the effective fed funds rate or SOFR). Throughout this chapter’s worked examples we use a simplified SOFR OIS convention set, but you should always verify the exact conventions for your product/CCP.

**Assumptions used for the worked examples in this chapter (check your product/CCP):**

| Convention | SOFR OIS |
|------------|----------|
| Day Count | ACT/360 |
| Compounding | Daily, geometric |
| Fixed Leg Payment Frequency | Annual (simplifying assumption) |
| Floating Leg Payment Frequency | Annual (simplifying assumption), compounded in arrears |
| Payment Delay | None (simplifying assumption) |
| Observation Convention | None (simplifying assumption; no lookback/lockout/shift) |
| Holiday Calendar | USNY (USD) |

> **Desk Reality:** Overnight coupons are compounded **in arrears**, so the final coupon is only known at period end. Products handle this with conventions such as payment delays, lookbacks/observation shifts, or lockouts.
> **Common break:** Valuing a trade assuming “no shift” when the confirmation uses a lookback/lockout creates small but sharp P&L effects around coupon dates and can break hedge attribution.
> **What to check:** Read the confirmation/cleared spec for (i) payment delay, (ii) observation adjustment (lookback/shift/lockout), and (iii) the exact day-count/calendar used for accrual.

**Comparison with Other Overnight Indices:**

| Index | Currency | Day Count | Compounding | Secured? |
|-------|----------|-----------|-------------|----------|
| SOFR | USD | ACT/360 | Geometric | Yes (repo) |
| Fed Funds | USD | ACT/360 | Geometric | No |
| €STR | EUR | ACT/360 | Geometric | No |
| SONIA | GBP | ACT/365 | Geometric | No |
| TONAR | JPY | ACT/365 | Geometric | No |

### 18.1.5 Worked Example: Computing a SOFR OIS Floating Payment

**Setup:** Consider a 3-month SOFR OIS over a generic 90-day period. Notional is $100 million. We use a simplified schedule with representative rates:

| Date | SOFR Rate | Days Applied |
|------|-----------|--------------|
| Jan 15-16 | 4.30% | 1 |
| Jan 16-17 | 4.32% | 1 |
| ... | ... | ... |
| (assume average 4.35%) | | 90 total days |

**Step 1: Daily Compounding**

The floating leg compounded rate is approximately:

$$\bar{L} = \left[\prod_{i=1}^{90}\left(1 + r_i \times \frac{d_i}{360}\right) - 1\right] \times \frac{360}{90}$$

For illustration, if all rates were exactly 4.35%:

$$\bar{L} \approx \left[\left(1 + \frac{0.0435}{360}\right)^{90} - 1\right] \times 4 = 4.372\\%$$

The compounding effect adds ~2bp over 3 months.

**Step 2: Net Payment Calculation**

If the fixed rate is 4.40%, the floating receiver pays:

$$\text{Net Payment} = USD 100\text{mm} \times \frac{90}{360} \times (4.40\\% - 4.372\\%) = USD 7,000$$

**Sanity Check:** Payment is small because rates are close. At 90/360 = 0.25 year fraction, each basis point is worth $2,500 per $100mm.

---

## 18.2 The Par Condition: OIS as a Par Bond

### 18.2.1 From Swaps to Bonds

To build the curve, we rely on the fundamental pricing equation: at inception, a par swap has a present value of zero. This implies the present value of the fixed leg equals the present value of the floating leg.

The key trick is to add (purely for valuation convenience) offsetting notional exchanges at maturity. This does not change the economics, but it lets you view the swap as an exchange of:
1. a **fixed-rate bond** (fixed coupons plus principal at maturity), and
2. a **floating-rate bond** (floating coupons plus principal at maturity).

**Economic intuition (why the floating leg is “par-like”):** a floating-rate note that resets to the relevant short rate is worth approximately par immediately after a reset, because it earns the prevailing funding rate for the next accrual period.

A par OIS can thus be conceptualized as two bonds:
1. **Fixed Rate Bond**: Pays coupons $k$ at times $T_i$.
2. **Floating Rate Bond**: Pays compounded overnight rates—and prices at par.

Under a single-curve OIS setup (the overnight index is consistent with the discount curve), the swap PV can be written using discount factors only. For a spot-starting swap (start at $T_0=0$):

- Floating leg PV (unit notional, no spread; idealized) is:
  $$PV_{\text{flt}} = 1 - P(0,T_N).$$
- Fixed leg PV is:
  $$PV_{\text{fix}} = k \sum_{i=1}^{N}\tau_i P(0,T_i) = k\\,A(0).$$

The **par condition** $PV_{\text{fix}} = PV_{\text{flt}}$ gives:

$$\boxed{k_{\text{par}} = \frac{1 - P(0,T_N)}{A(0)}}$$

Equivalently, with the “add principal at maturity” fiction, the par condition becomes:

$$\boxed{1 = k\sum_{i=1}^{N} \tau_i P(0,T_i) + P(0,T_N)}$$

Here, the left side (1) represents the floating leg value, and the right side represents the fixed leg coupons plus the return of notional at maturity $T_N$.

**Expand (replication story):** The identity $PV_{\text{flt}} = 1 - P(0,T_N)$ is easiest to remember as a “roll an overnight deposit” argument. Imagine lending USD 1 overnight and rolling it each day until $T_N$; the value process grows by the same daily factors that define the compounded overnight coupon. If you also discount cashflows using that same overnight curve, then “USD 1 today” and “USD 1 at $T_N$ discounted back” are the two endpoints, and the present value of the floating coupons is the difference between them.

**Check (one-period sanity):** With a single payment at $T$ and accrual $\tau$, the par equation reads $1 = k\\,\tau P(0,T) + P(0,T)$, so $k = (1-P(0,T))/(\tau P(0,T))$. Using the 1Y toy DF above $P(0,1)=0.975610$ (and $\tau=1$) gives $k\approx 2.50\\%$, matching the 1Y input quote.

> **Preview: Multi-Curve Complication**
>
> This elegant "floating = par" result depends on the floating rate being the same as the discount rate. In Chapter 19, we'll see that when the floating rate is LIBOR (or a SOFR projection curve different from the discount curve), this relationship breaks down—the floating leg no longer automatically equals par.

### 18.2.2 The Annuity Factor

The sum of discounted accrual factors is a recurring quantity known as the **annuity factor** (often called the PV01 of the fixed leg up to scaling):

$$\boxed{A(0) \equiv \sum_{i=1}^{N} \tau_i P(0,T_i)}$$

It is literally the PV today of receiving “1 per year” (scaled by accrual fractions) on the fixed-leg payment dates.

For a forward-starting swap with first fixing/payment at $T_k$ and last payment at $T_{k+m}$, the same idea is often written as a forward swap rate:

$$S_{k,m}(t) = \frac{P(t, T_k) - P(t, T_{k+m})}{A_{k,m}(t)}$$

where $S_{k,m}(t)$ is the forward swap rate.

---

## 18.3 Bootstrapping: From Par Quotes to Discount Factors

### 18.3.1 The Sequential Algorithm

We cannot solve for all discount factors simultaneously because the equation for a 5-year swap depends on the discount factors for years 1, 2, 3, and 4. Instead, we **bootstrap**: we solve for the curve one maturity at a time, starting from the shortest tenor.

**Step 1: Short End (Zero Rates)**

For short-dated OIS instruments with a single net payment, the quoted rate $R$ can be treated as a simple annualized rate over the stated day-count basis, implying a discount factor:

$$P(0,T) = \frac{1}{1 + R \cdot \tau(0,T)}$$

where $\tau(0,T)$ is the year fraction (e.g., ACT/360 for USD). If the market quote uses a different compounding basis, convert it to a consistent discount-factor representation before bootstrapping.

**Step 2: Long End (Iterative Solve)**

For an $N$-year swap, assuming we have already found the discount factors $P(0,T_1) \dots P(0,T_{N-1})$, we isolate the single unknown $P(0,T_N)$ in the par equation:

$$1 = k_N \left(\sum_{i=1}^{N-1} \tau_i P(0,T_i) + \tau_N P(0,T_N)\right) + P(0,T_N)$$

Solving for $P(0,T_N)$ yields the **bootstrap formula**:

$$\boxed{P(0,T_N) = \frac{1 - k_N \sum_{i=1}^{N-1} \tau_i P(0,T_i)}{1 + k_N \tau_N}}$$

In a production curve build, the “simple” bootstrap above must be paired with (i) exact schedule generation (stubs, business-day rules), (ii) any payment delays and observation adjustments, and (iii) an interpolation rule between market pillars.

### 18.3.2 Worked Example: Building a 5-Year Curve

**Example Title**: Bootstrapping discount factors from par OIS quotes (toy build)

**Context**
- You receive par SOFR OIS swap rates (the fixed rates that make each swap PV = 0) and need discount factors for PV and risk.
- Everything downstream—valuation, hedges, DV01 attribution—depends on getting the curve build *and* conventions right.

**Timeline (Make Dates Concrete)**
- Valuation date: 2026-02-18
- Assume annual payment dates: 2027-02-18, 2028-02-18, 2029-02-18, 2030-02-18, 2031-02-18
- Day count: ACT/360 in production. To keep the arithmetic transparent below, we set each coupon-period accrual factor to $\tau_i = 1$ (you should compute $\tau_i$ from the day count in real builds).

**Inputs**
- Unit-notional par OIS fixed rates (annual pay; single-curve OIS discounting for the OIS itself):

| Maturity | Par Rate $k$ |
|---:|---:|
| 1Y | 2.50% |
| 2Y | 2.70% |
| 3Y | 2.90% |
| 4Y | 3.05% |
| 5Y | 3.20% |

- Simplifications: ignore stubs, payment delays, and observation adjustments.

**Outputs (What You Produce)**
- Discount factors $P(0,T)$ at the market “pillar” maturities.
- A PV and DV01 for a collateralized cashflow discounted off the curve (to connect “curve build” → “risk number”).

**Step-by-step**
1. **Solve 1Y directly** (single payment):
   $$P(0,1) = \frac{1}{1 + 0.025} = 0.975610.$$
2. **Solve 2Y from the par condition**:
   $$P(0,2) = \frac{1 - 0.027\\,P(0,1)}{1 + 0.027} = 0.948061.$$
3. **Solve 3Y similarly**:
   $$P(0,3) = \frac{1 - 0.029\\,(P(0,1)+P(0,2))}{1 + 0.029} = 0.917603.$$
4. **Continue to 4Y and 5Y**:

| Maturity | $P(0,T)$ |
|---:|---:|
| 1Y | 0.975610 |
| 2Y | 0.948061 |
| 3Y | 0.917603 |
| 4Y | 0.886309 |
| 5Y | 0.853408 |

5. **Use the curve for PV**: suppose you will **receive** USD 100,000,000 at 2029-02-18 (3Y).
   $$PV = 100{,}000{,}000 \times P(0,3) = 91{,}760{,}300.$$
6. **Define and compute DV01** (explicit bump object + units + sign):
   - **Bump object:** the OIS discount curve expressed as a continuously-compounded zero curve $y_c(T) := -\ln P(0,T)/T$ (parallel shift).
   - **Bump size:** 1 bp $=10^{-4}$.
   - **Definition:** $DV01 := PV(y_c(T)\ \text{down }1\text{bp}) - PV(\text{base})$.
   - **Units:** USD per 1 bp (here: per USD 100,000,000 notional).

   For a single cashflow at maturity $T$, the approximation is
   $$DV01 \approx PV \times T \times 10^{-4}.$$
   Here, $DV01 \approx 91{,}760{,}300 \times 3 \times 10^{-4} \approx 27{,}528.$

**Cashflows (table)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2029-02-18 | +100,000,000 | Collateralized cashflow discounted on the OIS curve |

**P&L / Risk Interpretation**
- A DV01 of +USD 27.5k means: if the **OIS zero curve** shifts down 1 bp (as defined above), the PV of this USD 100mm 3Y cashflow increases by about USD 27.5k.
- On a desk, the first check is **scaling**: DV01 should scale linearly with notional and roughly with maturity (all else equal).
- What breaks in practice: if a risk report defines the bump on **par OIS quotes** (rebootstrap) rather than on a **zero curve** (parallel shift), the “DV01” numbers will differ even if both are called “OIS DV01.”

**Sanity Checks**
- Monotonicity: $P(0,T)$ should be decreasing in $T$ for positive rates.
- Magnitudes: $PV$ should be less than notional; DV01 should be “small” relative to PV (order of $PV\times T \times 10^{-4}$).

### 18.3.3 Using Futures as Inputs

In practice, the short end of the OIS curve (up to ~2 years) is often built from SOFR futures rather than OIS swaps, because the futures market can be more liquid:

- **1-Month SOFR Futures:** Settlement based on arithmetic average of daily SOFR over the contract month.
- **3-Month SOFR Futures:** Settlement based on geometric compounding of daily SOFR over the quarter.

> **Desk Reality: Convexity Adjustment**
>
> When using futures-implied rates to build a swap curve, a convexity adjustment is often needed because futures are marked-to-market daily while swaps are not. The adjustment is model-dependent, usually small at short maturities, and can grow with maturity and volatility. See Chapter 24 for the full treatment.

---

## 18.4 Zero Rates: Another View of the Same Curve

While the discount factor $P(0,T)$ is the fundamental pricing object, humans find it hard to differentiate "0.9176" from "0.9180". We prefer rates.

We can convert any discount factor into a **zero rate** (also called a spot rate or zero-coupon yield). The two most common conventions are:

**1. Continuously Compounded:**
$$y_c(T) = -\frac{\ln P(0,T)}{T}$$

**2. Annually Compounded:**
$$y_a(T) = \frac{1}{P(0,T)^{1/T}} - 1$$

Applying this to our bootstrapped points:

| Maturity | $P(0,T)$ | Continuous Zero $y_c$ | Annual Zero $y_a$ |
|----------|----------|-----------------------|-------------------|
| 1Y | 0.975610 | 2.47% | 2.50% |
| 2Y | 0.948061 | 2.67% | 2.70% |
| 3Y | 0.917603 | 2.87% | 2.91% |
| 4Y | 0.886309 | 3.02% | 3.06% |
| 5Y | 0.853408 | 3.17% | 3.22% |

### 18.4.1 Par Rates vs Zero Rates

Notice that the zero curve (e.g., `y_c(T)`) is generally *not* the same as the par curve ($k$). For an upward sloping yield curve:

- The par rate is a *weighted average* of the zero rates, weighted by discount factors
- The early coupons are discounted at lower rates (since the curve slopes upward)
- Therefore, the par rate is pulled down by the lower early rates
- The zero rate at maturity $T$ must be *higher* than the par rate at $T$

In our example: 5Y Par = 3.20%, 5Y Annual Zero = 3.22%. The zero rate exceeds the par rate because the par rate averages in the lower rates at earlier maturities.

---

## 18.5 Interpolation: Filling the Gaps

Market quotes are sparse (1Y, 2Y... 10Y, 30Y). But you might need to value a cashflow at 2.5 years. This requires **interpolation**. This is not just a mathematical nuisance; the choice of interpolation method dictates the shape of your forward rates and can impact the valuation of sensitive off-node risks.

### 18.5.1 Linear in Log Discount Factor

A standard practitioner choice is **linear interpolation of the log discount factor**:

$$\ln P(0,t) = (1-\alpha) \ln P(0,T_1) + \alpha \ln P(0,T_2)$$

where $\alpha = (t - T_1)/(T_2 - T_1)$ is the fraction of time between $T_1$ and $T_2$.

This method has a crucial physical property: it implies **piecewise constant instantaneous forward rates** between pillar dates. One way to see this is to assume that, on each interval $[T_i, T_{i+1}]$, the instantaneous forward rate is constant at some level $f_i$. Then

$$P(T) = P(T_i) e^{-f_i(T-T_i)}$$

The forward rate is flat between grid points. This is robust and creates stable local sensitivities, which is why it is preferred for trading systems over splines (which can oscillate).

**Check (recover the forward and the midpoint shortcut):** On an interval $[T_1,T_2]$ where the forward is constant, the endpoint discount factors imply
$$f_{1,2} = -\frac{\ln(P(0,T_2)/P(0,T_1))}{T_2-T_1}.$$
Then $P(0,t)=P(0,T_1)e^{-f_{1,2}(t-T_1)}$ for any $t\in[T_1,T_2]$, and at the midpoint this reduces to $P(0,(T_1+T_2)/2)=\sqrt{P(0,T_1)P(0,T_2)}$.

### 18.5.2 Why It Matters: Comparing Methods

If we interpolate our 2Y ($P=0.9481$) and 3Y ($P=0.9176$) points to 2.5 years:

**1. Log-Linear (Constant Forward):**
- Forward rate $f \approx 3.26\\%$
- $P(0,2.5) \approx 0.9327$

**2. Linear Zero Rates:**
- Averages the zeros: $(2.67\\% + 2.87\\%)/2 = 2.77\\%$
- $P(0,2.5) = e^{-0.0277 \times 2.5} \approx 0.9332$

The difference (approximately 5 basis points in price) is significant for leveraged positions. More importantly, the *risk profile* differs. Under log-linear interpolation, a risk at 2.5Y is distributed strictly to the 2Y and 3Y nodes. Under spline methods, it might "leak" to the 5Y or 10Y nodes, creating hedging noise.

### 18.5.3 Higher-Order Methods

A useful mental model is to rank interpolation methods by smoothness:

| Method | Smoothness | Forward Curve |
|--------|------------|---------------|
| Piecewise Linear Yields | $C^0$ | Discontinuous forwards |
| Piecewise Flat Forwards | $C^0$ | Piecewise constant |
| Hermite Splines | $C^1$ | Continuous, but not smooth |
| Cubic Splines | $C^2$ | Smooth, but may oscillate |
| Tension Splines | $C^2$ | Smooth with controlled curvature |

For trading systems, the piecewise flat forward approach is often preferred despite its lack of smoothness, because it produces predictable, local risk behavior.

> **Desk Reality: ASCII Visualization of Forward Rates**
>
> Under log-linear interpolation (piecewise flat forwards):
> ```
> Forward Rate
> |        ___________
> |  _____/           \______
> | /                        \___
> |/                             \
> +-------------------------------- Time
>    1Y  2Y  3Y  5Y  10Y     30Y
> ```
>
> Under cubic spline (smooth but can oscillate):
> ```
> Forward Rate
> |      /\
> |     /  \___
> |    /       \      ___
> |   /         \____/
> |  /
> +-------------------------------- Time
>    1Y  2Y  3Y  5Y  10Y     30Y
> ```
> The oscillations in the spline can create phantom forward rate humps.

---

## 18.6 Locality: How Quote Bumps Propagate

A key feature of the bootstrap method is its sequential dependency. This creates a specific direction of causality:

**Short Term affects Long Term**: If the 1Y rate changes, it changes $P(0,1)$. Since $P(0,1)$ is used to solve for $P(0,2)$, the 2Y discount factor also changes (to maintain the 2Y par condition). The change cascades all the way to 30Y.

**Long Term does NOT affect Short Term**: If the 10Y rate changes, it only affects discount factors from 10Y onwards. The 2Y swap still prices correctly using the original 2Y inputs.

This "one-way" causality is distinct from global fitting methods (like Nelson-Siegel), where a change in a long-dated point can shift the entire curve. The bootstrap's **locality** makes it easier to hedge: to hedge a 10Y risk, you primarily trade the 10Y swap, without worrying that you are ruining your 2Y fit.

### 18.6.1 The Jacobian Perspective

You can formalize “locality” with a Jacobian matrix that maps quote bumps to curve moves. Let $k_j$ denote the $j$-th input quote (e.g., a par OIS rate for a given maturity) and let $P_i$ denote the $i$-th discount factor pillar. The Jacobian $J := \partial P / \partial k$ summarizes how each discount factor responds to each quote.

For a sequential bootstrap where each new quote solves for a new longest-maturity discount factor, this Jacobian is *lower triangular*: long-dated quote bumps do not change short-dated discount factors.

**Check (toy “lower-triangular” bump):** In the toy annual-pay build in Section 18.3.2, the 3Y bootstrap is
$$P(0,3) = \frac{1 - k_3\left(\tau_1 P(0,1)+\tau_2 P(0,2)\right)}{1 + k_3\tau_3}.$$
If you bump only the 3Y input quote by +1 bp (rebootstrap), $P(0,1)$ and $P(0,2)$ stay fixed and $P(0,3)$ falls (higher par rate $\rightarrow$ lower discount factor). With $P(0,1)=0.975610$, $P(0,2)=0.948061$, $k_3=2.90\\%$, and $\tau_i=1$, bumping $k_3$ to 2.91% gives $\Delta P(0,3)\approx -0.000276$. On a USD 100mm cashflow at 3Y, that is about \-$27.6k PV — a reasonable “par-point bucket delta” scale.

### 18.6.2 Risk Measures and “What Is Being Bumped?”

To make curve risk actionable, you must say **what is bumped**, by **how much**, and how the curve is **re-built** after the bump. Otherwise two systems can report “DV01” numbers that are not comparable.

Two common bump styles are:
1. **Zero-curve shift (direct bump):** bump a zero-rate curve `y_c(T)` and recompute discount factors via $P(0,T)=e^{-y_c(T)T}$ (continuous compounding shown here).
2. **Par-quote bump (rebootstrap):** bump a specific market quote $k_j$ (e.g., the 5Y par OIS rate) and rebootstrap the curve so all instruments reprice to par under the bumped quote set.

For this chapter we use the book-wide DV01 sign convention:
- **Bump object:** a continuously-compounded zero curve `y_c(T)` for the OIS discount curve.
- **Bump size:** 1 bp $=10^{-4}$, applied as a parallel shift $y_c(T)\mapsto y_c(T)-10^{-4}$.
- **Definition:** $DV01 := PV(\text{rates down }1\text{bp}) - PV(\text{base})$.
- **Units:** currency per 1 bp (always state the notional basis).

> **Pitfall — “What is being bumped?”:** “OIS DV01” might mean a zero-curve parallel shift in one system and a par-quote rebootstrap in another.
> **Why it matters:** hedge ratios, bucket risk, and P&L explain can differ materially even if both numbers are labeled “DV01.”
> **Quick check:** ask for the exact bump description (curve object + size + direction) and test scaling on a toy cashflow (e.g., a single discount factor): DV01 should be roughly $PV\times T\times 10^{-4}$ under a continuous-zero parallel shift.

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

**Check (DV01 × spread heuristic):** A quick sanity check is to translate a curve difference into a DV01-style number. If “LIBOR discounting” is roughly “OIS discounting + 50 bp” (treated as a parallel shift in continuously-compounded zero rates), then you should expect
$$PV_{\text{OIS}}-PV_{\text{LIBOR}} \\;\approx\\; 50\times DV01,$$
where $DV01$ is defined here as the PV change for an OIS-zero curve **down** 1 bp. For the OIS-discounted bond above, a cashflow-weighted check gives $DV01 \approx 0.0467$ price points per bp (per 100 notional), so $50\times DV01\approx 2.3$ points — close to the $2.4$ points obtained by re-discounting.

> **The $1 Million Mistake**
>
> On a $USD 100,000,000$ swap book:
> *   Value with LIBOR Discounting: $USD 96.68$ million.
> *   Value with OIS Discounting: $USD 99.08$ million.
> *   **Difference**: $USD 2.4$ million.
>
> If you were a bank in 2008 shifting to OIS discounting, you suddenly "found" (or lost) millions overnight just by changing the discount curve. This was the famous "CSA Discounting Switch."

If you are a bank posting cash collateral (earning OIS) but pricing your book using LIBOR, you are systematically underestimating the value of your assets (or liabilities). This specific mismatch was a primary driver of the P&L restatements banks had to make post-crisis.

---

## 18.8 Practical Considerations

### 18.8.1 Turn-of-Year (TOY) Effects

One operational nuance is the **turn-of-year (TOY) effect**: around year-end, balance-sheet constraints can create localized dislocations in overnight funding rates.

A standard smooth curve will miss this spike. One approach is to add a localized **overlay** to the forward curve so that special-date effects live in a separate component:

$$f(t) = f_{smooth}(t) + \varepsilon_{TOY}(t)$$

Specifically, the forward curve is decomposed as:

$$\boxed{f(t) = \varepsilon_f(t) + f^*(t)}$$

where $\varepsilon_f(t)$ is user-specified (and contains discontinuities around special dates) and $f^*(t)$ is the smooth curve to be calibrated.

Practitioners may explicitly mark a "turn" spread (e.g., a special forward premium over the year-end dates) to ensure that short-dated swaps crossing the year-end price correctly. Without this, curve calibration can force the "smooth" rate up for the entire surrounding month to match the price, distorting the valuation of everything else.

> **Desk Reality: Why Turns Matter**
>
> Turn-of-year effects can be large or negligible depending on balance-sheet capacity and liquidity conditions. The key operational point is not the exact number — it’s that **special dates can create localized forward-rate spikes** that a smooth interpolation will miss unless you explicitly model them.

### 18.8.2 Fed Funds vs SOFR

Fed Funds and SOFR are both USD overnight benchmarks that appear in OIS markets and in collateral remuneration conventions. Fed Funds is an overnight interbank borrowing rate; SOFR is a secured overnight financing rate derived from overnight repo transactions. Because one is unsecured and the other is secured, the two fixings can differ; the sign and size of the basis is an empirical input and can vary with market structure and policy implementation.

| Characteristic | Fed Funds | SOFR |
|----------------|-----------|------|
| Underlying market | Interbank overnight borrowing | Overnight repo |
| Secured? | Unsecured | Secured |
| Construction | Effective fed funds rate (transaction-weighted) | Volume-weighted median of overnight repo rates |
| OIS benchmark | Fed Funds OIS | SOFR OIS |

### 18.8.3 Implementation Checklist

| Check | What to Verify |
|-------|----------------|
| **Repricing** | Every input par swap should reprice to par (error < $10^{-10}$) |
| **Bump Stability** | Bump an input rate by 1bp; changes should be localized and reasonable |
| **Convention Exactness** | Year fractions ($\tau$) must match the specific OIS conventions (ACT/360 vs ACT/365) |
| **Negative Rates** | If rates go negative, ensure your formulas still produce valid discount factors |
| **Interpolation Artifacts** | Check forward rates between nodes for unrealistic spikes or dips |

A very common implementation error is using the wrong day count in accrual calculations.

### 18.8.4 Negative Rates: Operations and Models

Negative interest rates became a feature of some major markets in the 2010s. From a curve-construction perspective, the key point is simple: negative rates are compatible with arbitrage-free discounting, but they can break naive system assumptions.

**Operational Implications:**

When OIS rates go negative, discount factors exceed 1. Mathematically:

$$P(0,T) = \frac{1}{1 + R \cdot \tau(0,T)} \gt 1 \text{ when } R \lt 0$$

This is mathematically valid—receiving $1 in the future is worth more than $1 today if you'd have to pay to store cash.

> **Desk Reality: System Limits and Negative Rates**
>
> Many legacy trading systems were built assuming rates cannot go negative:
> - Discount factor bounds: $0 \lt P(T) \lt 1$ (fails with negative rates)
> - Log-rate calculations: $\ln(r)$ fails when $r \lt 0$
> - Square root volatility models: $\sigma\sqrt{r}$ fails when $r \lt 0$
>
> When EUR and CHF rates went negative, many systems required emergency patches.

**Modeling Implications:**

For rate options (caps, floors, swaptions), negative rates break the standard Black-76 setup, which assumes rates are lognormal (and hence strictly positive). Two common fixes are:

1. **Shifted Lognormal Model:** Assume $r + \alpha$ is lognormal for some shift $\alpha \gt 0$. Options are priced using Black-76 with $F_k \rightarrow F_k + \alpha$ and $K \rightarrow K + \alpha$.

2. **Bachelier (Normal) Model:** Assume rates follow arithmetic Brownian motion: $dF = \sigma^* dz$. This produces a normal distribution for the underlying rate and allows negative rates.

The Bachelier caplet formula is:

$$\text{Caplet} = L \delta_k P(0, t_{k+1})\left[(F_k - R_K)N(d) + \sigma_k^* \sqrt{t_k} N'(d)\right]$$

where $d = \frac{F_k - R_K}{\sigma_k^* \sqrt{t_k}}$.

> **Practitioner Note:** Black (lognormal) vol and normal (Bachelier) vol are not directly comparable. Always label the vol type and its units, and avoid mixing them in risk reports.

### 18.8.5 Multi-Currency Collateral and the Collateral Option

Some CSAs allow multiple eligible collateral currencies (and sometimes eligible non-cash collateral). This creates an embedded **collateral option** that can affect the effective discount rate.

**The Collateral Option Problem:**

Many CSAs allow counterparties to post collateral in multiple currencies (e.g., USD, EUR, or GBP). The collateral poster will naturally choose the "cheapest" currency—the one with the lowest borrowing cost.

> **Practitioner Note: The "Cheapest-to-Deliver" Collateral**
>
> The collateral choice problem is analogous to the cheapest-to-deliver option in Treasury futures. Just as a futures seller delivers the cheapest eligible bond, a margin poster posts the cheapest eligible currency.
>
> **Example:**
> - You owe variation margin on a USD swap
> - Your CSA allows USD or EUR collateral
> - USD OIS is 5.25%; €STR is 3.75%
> - EUR is cheaper to borrow → post EUR
>
> The effective discount rate for this trade is no longer simply "USD OIS" but something between USD OIS and €STR, depending on the optionality.

**Simplified Pricing Approach:**

For a first approximation, use the lowest rate among allowed collateral currencies:

$$r_{eff} = \min(r_{USD}, r_{EUR}, r_{GBP}, ...)$$

More sophisticated approaches model the option explicitly, but this adds significant complexity.

> **Desk Reality: When the Collateral Option Matters**
>
> The collateral option is most valuable when:
> - Large rate differentials between eligible currencies
> - Long-dated trades (option value accumulates)
> - One-directional MTM (always posting or always receiving)
>
> For plain-vanilla swaps, the effect is often small relative to larger risk drivers, but it can become material for long-dated positions and in cross-currency funding setups.

### 18.8.6 Initial Margin and MVA

The discussion so far has focused on **variation margin (VM)**—collateral that changes daily to reflect the mark-to-market value. There is also **initial margin (IM)**—collateral posted at trade inception (and potentially adjusted over time) to protect against potential future exposure.

In many setups, variation margin is rehypothecatable by the receiver (it can be used for funding), while initial margin is often segregated and not rehypothecatable. This creates different economic implications:

- **VM can be rehypothecated** → Receiver earns OIS on it → This is why OIS discounting works
- **IM cannot be rehypothecated** → Capital drag → Creates MVA

**Margin Valuation Adjustment (MVA):**

MVA is an adjustment to reflect the cost of funds tied up in margin accounts. The cost arises because:

1. Initial margin must be funded (you borrow to post it)
2. Interest received on IM is typically below your funding cost
3. The spread represents a cost that reduces trade value

A simplified MVA formula:

$$\boxed{\text{MVA} = \int_0^T E[IM(t)] \cdot s_{fund} \cdot df(t) \\, dt}$$

where $E[IM(t)]$ is expected initial margin at time $t$, $s_{fund}$ is the funding spread (your cost of funds minus the rate received on margin), and $df(t)$ is the discount factor.

> **Desk Reality: MVA in Practice**
>
> MVA is typically quoted in two ways:
> - **Upfront cost:** "This trade has 5bp of MVA" (5bp of notional as day-one cost)
> - **Running spread:** "MVA is 0.5bp running" (0.5bp per year)
>
> For a 10-year $100mm swap with 5bp upfront MVA, the cost is $50,000. This comes directly out of the trade's profit margin.
>
> Funding-cost conventions vary (marginal vs average, desk vs treasury). Be explicit about what $s_{fund}$ represents in your system.

### 18.8.7 The Collateral Lifecycle and Price Alignment Interest (PAI)

To truly understand why OIS discounting works, you need to understand the operational lifecycle of collateral and how **Price Alignment Interest (PAI)** creates the economic link.

> **Practitioner Note: The Collateral Lifecycle**
>
> **Day 0: Trade Execution**
> - You enter a swap with counterparty
> - MTM is zero; no margin required
>
> **Day 1: First MTM**
> - Swap moves in your favor: MTM = +$1,000,000
> - Counterparty owes you variation margin
>
> **Day 2: Margin Call**
> - Your operations team issues margin call
> - Counterparty must post $1,000,000 cash
>
> **Day 3: Cash Received**
> - You receive $1,000,000 cash collateral
> - You invest it overnight at OIS (say, 5%)
>
> **Daily: PAI Accrues**
> - You earn $1,000,000 × 5% / 360 ≈ $139 per day
> - Under the CSA, you owe this PAI to the counterparty
>
> **Net Effect:**
> - You hold cash earning 5%
> - You pay PAI at 5%
> - Your funding cost on this collateralized position = 0%
>
> **Why This Justifies OIS Discounting:**
> - Your cost of funding a $1 future cashflow is the OIS rate
> - Therefore, you should discount at OIS

**PAI Mechanics:**

Price Alignment Interest is the interest paid or received on variation margin. Under standard CSAs:

- If you receive collateral: You pay PAI to the counterparty
- If you post collateral: You receive PAI from the counterparty

The PAI rate is typically an overnight benchmark rate specified by the CSA/CCP for the relevant currency.

---

## 18.9 USD Overnight Benchmarks: Fed Funds OIS vs SOFR OIS

OIS discounting is both:
1. a **principle** ("discount at the collateral remuneration rate"), and
2. an **implementation** (a curve bootstrapped from OIS instruments referencing a specific overnight index).

In USD, OIS swaps may reference different overnight benchmarks (e.g., the effective fed funds rate or SOFR). So the phrase “USD OIS curve” is incomplete unless you specify the benchmark and the compounding/observation conventions.

### 18.9.1 Post-Crisis: From LIBOR Discounting to OIS Discounting

Following the GFC, market participants started using OIS as the discount rate for pricing collateralized interest rate swaps.

A concrete milestone often cited is that, in June 2010, LCH.Clearnet announced it would use OIS instead of LIBOR to discount its interest rate swap portfolio (reported at the time as roughly USD 218 trillion notional).

### 18.9.2 Fed Funds vs SOFR (USD)

In the United States, the overnight rate used in “Fed Funds OIS” is the effective fed funds rate. SOFR (the Secured Overnight Financing Rate) is a benchmark based on overnight repo transactions (often described as a volume-weighted median average of overnight repo rates).

For curve building and risk reporting, what matters is:
- which overnight index your discount curve is built on (Fed Funds OIS vs SOFR OIS), and
- which exact conventions translate fixings into coupon cashflows (compounding-in-arrears, observation shifts/lookbacks, payment delays, calendars).

> **Desk Reality:** In many risk systems, “OIS” is a *curve label*, not a guarantee that discounting is aligned to the CSA/clearing terms.
> **Common break:** One system discounts a USD collateralized trade on “SOFR OIS” while another uses “Fed Funds OIS” (or uses different observation conventions), producing PV and DV01 differences that look like “model error” but are really convention mismatch.
> **What to check:** For a sample trade, confirm (i) collateral currency + remuneration index, (ii) discount curve label ↔ benchmark mapping, and (iii) compounding/observation/payment conventions used in both pricing and risk.

---

## Summary

- OIS discounting is the desk implementation of “discount at the collateral remuneration rate” under an idealized cash-collateral setup.
- An OIS floating coupon is computed from daily compounded overnight fixings, so the realized coupon rate is known only at period end.
- Under a single-curve OIS setup, the par condition implies $PV_{\text{flt}}=1-P(0,T_N)$ and $k_{\text{par}} = (1-P(0,T_N))/A(0)$.
- Bootstrapping is sequential: each new pillar depends on earlier pillars, creating “locality” (and a lower-triangular Jacobian in a pure sequential build).
- Zero rates (e.g., `y_c(T)`) are just a re-parameterization of discount factors; the par curve and the zero curve generally differ.
- Interpolation is a modeling choice: log-linear interpolation of discount factors implies piecewise-constant instantaneous forwards and tends to give stable, local risk.
- DV01 must specify bump object, bump size, units, and sign; zero-curve shifts and par-quote rebootstrap bumps produce different “DV01” numbers.
- Operational features (stubs, payment delays, observation shifts/lookbacks, special dates, negative rates) and funding/margin terms (PAI, IM/MVA) can dominate small PV differences.
- In USD, “OIS curve” must name the overnight benchmark (Fed Funds OIS vs SOFR OIS) and the exact cashflow conventions.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **OIS** | Overnight Indexed Swap: fixed rate vs compounded overnight | The instrument defining the "risk-free" discounting curve |
| **Par Condition** | $1 = k A(0) + P(0,T_N)$ | The constraint used to solve for discount factors |
| **Annuity Factor** | $A(0) = \sum \tau_i P(0,T_i)$ | Represents the PV of a 1bp coupon stream; used in all swap math |
| **Bootstrap** | Sequential calibration algorithm | Allows extraction of a unique discount curve from par quotes |
| **Log-Linear Interpolation** | Linear in $\ln P(T)$ | Produces piecewise constant forwards; stable risk |
| **Locality** | Input changes affect only $T \ge T_{input}$ | Crucial for stable hedging and risk management |
| **Special-date effects (“turns”)** | Local forward-rate dislocations around dates like year-end | Smooth curves can miss localized spikes, distorting PV/risk if not handled |
| **Floating leg ≈ par** | Under a matching projection/discounting rate, a floater resets near par | Enables the $1-P(0,T_N)$ floating-leg PV identity in the single-curve setup |
| **PAI** | Price Alignment Interest: interest on variation margin | The economic link between collateral and discounting |
| **MVA** | Margin Valuation Adjustment | Cost of funding initial margin over trade life |
| **Collateral Option** | Choice of which currency to post as margin | Affects effective discount rate in multi-currency CSAs |
| **USD OIS benchmark** | “Fed Funds OIS” vs “SOFR OIS” (benchmark + conventions) | Risk/PV mismatches appear if “OIS” is treated as a generic label |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $P(0,T)$ | discount factor to $T$ | unitless; $P(T,T)=1$ |
| $k$ (or $k_N$) | OIS par fixed rate | per year; day count + compounding must be stated |
| $\tau(t_1,t_2)$, $\tau_i$ | accrual year fraction | years; day count dependent (e.g., ACT/360) |
| $t_{n,i}$ | business-day fixing dates in coupon period $n$ | dates (calendar dependent) |
| $r_{n,i}$ | overnight fixing applied over $[t_{n,i},t_{n,i+1}]$ | per year on the index basis |
| $\delta_{n,i}$ | accrual fraction for $[t_{n,i},t_{n,i+1}]$ | years; weekends/holidays appear via larger $\delta$ |
| $A_n^{\mathrm{flt}}$ | compounded accrual factor over coupon period $n$ | unitless; $\prod_i (1+r_{n,i}\delta_{n,i})$ |
| $\bar{L}_n$ | annualized compounded overnight rate for coupon period $n$ | per year; $(A_n^{\mathrm{flt}}-1)/\tau_n$ |
| $A(0)$ | fixed-leg annuity factor | years; $\sum_i \tau_i P(0,T_i)$ |
| `y_c(T)` | continuous-compounded zero rate | per year; $P(0,T)=e^{-y_c(T)T}$ |
| $y_a(T)$ | annual-compounded zero rate | per year; $P(0,T)=(1+y_a(T))^{-T}$ (if $T$ in years) |
| $f(T)$ | instantaneous forward rate | per year; depends on interpolation choice |
| $\varepsilon_f(t)$ | forward-rate overlay (e.g., special dates) | per year; user-specified component |
| $s_{fund}$ | funding spread for MVA | per year; “my funding rate minus margin rate” |
| $DV01$ | PV change for a 1 bp move | currency per 1 bp; define bump object + sign |

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
| 8 | What is the turn-of-year (TOY) effect? | A localized year-end dislocation in overnight rates driven by balance sheet constraints. |
| 9 | How does a bump in the 2Y rate affect the 1Y discount factor? | It doesn't. Causality flows from short to long (lower triangular Jacobian). |
| 10 | What is the simplest relation between discount factor and continuous zero? | $P(0,T) = e^{-y_c(T) \cdot T}$. |
| 11 | Why does a floating-rate bond paying overnight compound to par? | It provides exactly the payments needed to service borrowings at overnight rates. |
| 12 | What is the key difference between OIS and LIBOR swap mechanics? | OIS floating rate is known at end of period; LIBOR is set at beginning. |
| 13 | What does "locality" mean in bootstrapping? | Changing an input rate only affects discount factors at or beyond that maturity. |
| 14 | For a short OIS with a single net payment, how do you get the discount factor from a simple quoted rate? | $P(0,T) \approx 1/(1 + R\\,\tau(0,T))$ for the stated day-count basis. |
| 15 | What is the bootstrap formula for $P(T_N)$? | $P(T_N) = [1 - k_N \sum_{i \lt N} \tau_i P(T_i)] / (1 + k_N \tau_N)$. |
| 16 | Give one concrete milestone in the post-crisis shift to OIS discounting. | LCH announced in June 2010 that it would discount its cleared IRS portfolio using OIS rather than LIBOR. |
| 17 | What is PAI (Price Alignment Interest)? | Interest earned/paid on variation margin, typically at the OIS rate. |
| 18 | What is MVA? | Margin Valuation Adjustment: the cost of funding initial margin over trade life. |
| 19 | If your CSA allows multiple collateral currencies, what does the “collateral option” mean for discounting? | The effective discounting is not purely “USD OIS”; it depends on which collateral the poster chooses and on FX/funding constraints. |
| 20 | What happens to discount factors when OIS rates go negative? | Discount factors exceed 1 ($P(T) \gt 1$), which is mathematically valid but operationally challenging. |

---

## Mini Problem Set

1. (Compute) A 6-month OIS quotes at 4.00% as a simple annual rate on ACT/360. There are 182 actual days. Compute (a) $P(0,T)$ and (b) the continuously compounded zero rate `y_c(T)` using the same year fraction $\tau=182/360$.
2. (Compute) Given annual-pay par OIS rates (assume $\tau_1=\tau_2=1$): 1Y = 3.00%, 2Y = 3.50%. Compute $P(0,1)$ and $P(0,2)$.
3. (Compute) You have $P(0,1)=0.98$, $P(0,2)=0.95$, $P(0,3)=0.91$ (annual-pay). Compute (a) the 3Y par rate and (b) $y_c(3)$.
4. (Compute) Given $P(0,2)=0.9400$ and $P(0,3)=0.9000$, compute $P(0,2.5)$ via (a) log-linear in $\ln P$ and (b) linear in $P$.
5. (Concept) Explain (in words) why log-linear interpolation of discount factors implies piecewise-constant instantaneous forward rates.
6. (Concept) In a sequential bootstrap, why does bumping a 10Y input quote not change $P(0,2)$? Relate your answer to the Jacobian “lower triangular” idea.
7. (Compute) A 3-year zero-coupon bond pays 100 at maturity. Compare PV using $P(0,3)=0.9100$ vs $P(0,3)=0.8965$. Express the PV difference in bp of face value.
8. (Desk) Two systems report “OIS DV01” for the same trade, but one uses a zero-curve parallel shift and the other uses a par-quote bump with rebootstrap. What questions do you ask to reconcile the numbers, and what toy scaling check can you run?
9. (Compute) Given $P(0,2)=0.9400$ and $P(0,3)=0.9000$, compute the 1-year simple forward rate for $[2,3]$.
10. (Desk) Your CSA allows posting USD or EUR collateral. Explain what the “cheapest-to-deliver collateral” idea means for discounting a USD trade.
11. (Compute) The 6-month €STR OIS quotes at -0.50% (ACT/360, 182 days). Compute $P(0,T)$ and verify it exceeds 1.
12. (Compute) IM is approximately USD 5mm for 10 years and your funding spread is 50 bp. Estimate the undiscounted funding cost and give a rough PV if the average discount factor over the horizon is 0.85.

### Solution Sketches (Selected)
1. $\tau=182/360=0.5056$. $P\approx 1/(1+0.04\tau)=0.9802$. $y_c(T)=-\ln P/\tau \approx 3.96\\%$.
2. $P(0,1)=1/1.03=0.9709$. $P(0,2)=(1-0.035P(0,1))/(1.035)=0.9334$.
4. (a) $\ln P(0,2.5)=\tfrac{1}{2}\ln(0.94)+\tfrac{1}{2}\ln(0.90)$ so $P(0,2.5)=\sqrt{0.94\cdot 0.90}=0.9198$. (b) Linear in $P$: $0.5(0.94+0.90)=0.9200$.
7. PVs are 91.00 and 89.65, so the difference is 1.35 price points = 135 bp of face value.
8. Ask for (i) bump object (zero curve? which representation?), (ii) bump size/direction, (iii) whether the curve is rebuilt and how (rebootstrap vs direct shift), (iv) units/notional basis and sign convention. Quick check: on a single cashflow $PV=N\\,P(0,T)$, a continuous-zero parallel shift gives $DV01\approx PV\\,T\\,10^{-4}$.
12. Undiscounted cost $\approx 5{,}000{,}000\times 0.005\times 10=250{,}000$. Rough PV $\approx 250{,}000\times 0.85\approx 210{,}000$ (level-cost approximation).

## References

- Andersen & Piterbarg, *Interest Rate Modeling* (Vol 1), “OIS definition” and “Swap Rate Dynamics” (OIS compounding; par swap rates and annuity factors).
- Hull, *Options, Futures, and Other Derivatives*, “Determining Risk-Free Rates” (OIS intuition; floating vs fixed bond view).
- Hull, *Risk Management and Financial Institutions*, “The OIS Rate” and “Central Clearing” (LIBOR–OIS stress; price alignment interest).
- Neftci, *Principles of Financial Engineering*, “OIS discounting after the GFC” and “LCH adopts OIS discounting” (June 2010 milestone).
- Crépey, *Counterparty Risk and Funding*, “Collateral and discounting” (collateral remuneration at an OIS-like rate; discount/projection separation).
- Damiano, *Counterparty Credit Risk, Collateral and Funding*, “SCSA and collateral discounting” (clean CSA intuition; OIS discounting motivation).
- *Interest Rate Models: Theory and Practice*, §6.19.1 (log-linear interpolation between discount factors).
