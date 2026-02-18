# Chapter 19: Projection Curves — LIBOR, SOFR, and the Multi-Curve Framework

---

## Introduction

In a single-curve setup, one curve does two jobs:
1. **Discounting:** converting future cashflows to present value.
2. **Projection:** generating forward rates used to forecast floating coupons.

After the 2007–2008 crisis, spreads between overnight OIS rates and unsecured term rates (and spreads between different tenors) became large and time-varying. The practical result is a **multi-curve** world: one curve for discounting and one (or more) index-specific curves for projection.

Prerequisites: [Chapter 17 — Curve Construction — Bootstrapping, Interpolation, and the Spline Zoo](chapters/chapter_17_curve_construction_bootstrapping_interpolation.md); [Chapter 18 — OIS Discounting Curve — Building the “Risk-Free-ish” Curve](chapters/chapter_18_ois_discounting_curve.md); [Chapter 25 — Interest Rate Swaps — Mechanics and Valuation](chapters/chapter_25_interest_rate_swaps_mechanics_valuation.md)  
Follow-on: [Chapter 20 — Tenor Basis — 1M vs 3M vs 6M and Basis Swap Logic](chapters/chapter_20_tenor_basis.md); [Chapter 21 — Cross-Currency Curves — CIP, FX Forwards, and Cross-Currency Basis as Curve Constraints](chapters/chapter_21_cross_currency_curves.md); [Chapter 22 — Curve Risk Management in a Multi-Curve World — Par-Point Deltas, Jacobians, and Controlled Perturbations](chapters/chapter_22_multi_curve_risk_jacobians.md)

## Learning Objectives
- Translate a float-leg quote → projected cashflows → PV under a stated discount curve.
- Explain why “par floater = par” fails when projection and discount curves differ.
- Write the multi-curve PV equation for a floater or vanilla swap.
- Define discount DV01, projection DV01, and basis DV01: bump object, bump size (1bp = $10^{-4}$), units, and sign.
- Recognize why SOFR-style compounded coupons are operationally different from LIBOR-style term coupons.

---

## 19.1 The "Single Curve" Fallacy

The single-curve model assumes that the curve used to discount cashflows is also the curve that generates the forward rates used to project floating coupons. Pre-crisis, this approximation was often “good enough” because key spreads were small. During 2007–2008, spreads such as the **LIBOR–OIS spread** became large and volatile (often around 10bp pre-2007 and spiking to hundreds of bp in 2008), and basis spreads between tenors stopped being negligible. Once these spreads exist, forcing everything into one curve creates internal inconsistencies and misleading risk.

### 19.1.1 A Toy Contradiction

Assume (for illustration):
- The OIS discount curve is flat at 2.00% (continuous compounding).
- A 3M FRA market quote implies a forward rate of 3.40% for a future 3M period.

In a single-curve world, those two statements cannot both be true, because a forward rate over $[T_1,T_2]$ is mechanically tied to discount factors.

### 19.1.2 Where the Single-Curve Identity Breaks

If one curve $P(0,T)$ drives both discounting and projection, then for any $\tau=T_2-T_1$:

$$
F(0;T_1,T_2)=\frac{1}{\tau}\left(\frac{P(0,T_1)}{P(0,T_2)}-1\right).
$$

Using the OIS curve at 2.00% gives $P(0,0.25)\approx e^{-0.02\cdot 0.25}$ and $P(0,0.50)\approx e^{-0.02\cdot 0.50}$, implying an OIS-implied 3M forward close to 2.00%. If the FRA quote says 3.40%, a single curve cannot fit both sets of instruments without “breaking” the identity above.

**Check (toy numbers):** Under a flat 2.00% continuously-compounded discount curve, $P(0,0.25)=e^{-0.005}\approx 0.9950$ and $P(0,0.50)=e^{-0.01}\approx 0.9900$. The implied 3M forward is
$$F_d(0;0.25,0.50) \approx \frac{1}{0.25}\left(\frac{0.9950}{0.9900}-1\right)\approx 0.0201,$$
i.e., about 2.01% (annualized), not 3.40%. Turning that around: a 3.40% FRA over a 0.25-year accrual implies a growth factor $1+F\tau = 1+0.034\times 0.25=1.0085$, i.e., it demands a materially different “projection” curve object than the OIS discount curve.

### 19.1.3 The Multi-Curve Resolution

The resolution is to separate:
- **Discounting:** a discount curve $P_d(0,T)$ used to PV cashflows (often OIS for collateralized trades).
- **Projection:** one or more index curves used to generate forward rates and forecast floating coupons.

> **Analogy — Two Watches**
>
> One watch tells you what a dollar tomorrow is worth today (discounting). The other tells you what the floating index will fix at (projection). They are related, but they are not the same object once credit/liquidity spreads and tenor basis exist.

---

## 19.2 The Multi-Curve Valuation Framework

In the multi-curve framework, we work with:
- a **discount curve** $P_d(0,T)$ used to present-value cashflows; and
- one or more **index (projection) curves** $P^{(k)}(0,T)$ used to generate forward rates for a particular floating index $k$ (3M term, 6M term, compounded SOFR, etc.).

A convenient abstraction is a **multi-index curve group**,
$$
\\{P_d(\cdot), P^{(1)}(\cdot),\ldots,P^{(K)}(\cdot)\\},
$$
i.e., one discounting curve and one index curve per tenor/index.

### 19.2.1 The Roles of the Curves

**1. Discount curve $P_d$ (discounting / PV)**

This curve answers: “What is USD 1 paid at time $T$ worth today?” It is the curve used in the discount factors multiplying every cashflow in the PV sum.

For collateralized derivatives, a common modeling choice is to use an OIS curve as the discount curve, because the overnight rate is a proxy for the collateral remuneration / funding rate in many CSA-style setups (see Chapter 18).

**2. Index / projection curve $P^{(k)}$ (projection / forwards)**

This curve answers: “What rate will index $k$ fix at for $[T,T+\tau]$?” One convenient representation is via **pseudo-discount factors** (or “index curves”) that generate forwards through the familiar formula:

$$
F_k(0; T, T+\tau) = \frac{1}{\tau} \left( \frac{P^{(k)}(0, T)}{P^{(k)}(0, T+\tau)} - 1 \right).
$$

The key point: $P^{(k)}$ is **not** a tradable discount factor; it is a curve object calibrated so that the forwards used to project coupons match market quotes.

> **Pitfall — Discounting with a projection curve:** Treating $P^{(k)}$ as if it were a discount curve.
> **Why it matters:** PV and DV01 can be wrong even if forwards “look right”; hedges sized off a single delta can silently embed basis risk.
> **Quick check:** For a collateralized trade, ask: “Which curve produces discount factors in the PV sum?” If the answer is not the OIS/CSA discount curve, stop and reconcile conventions.

> **Desk Reality:** Most systems maintain a “curve group” (discount + index curves) and label each leg with a discount curve and an index curve.
> **Common break:** A report shows a single “interest-rate delta” even though the position has both discount and projection/basis exposures.
> **What to check:** Confirm the curve labels used in PV, and whether risk is decomposed into discount PV01, index PV01, and basis PV01.

### 19.2.2 Valuation of a Floating Rate Note (FRN): the “par floater” is not par

In a single-curve world, a standard FRN that resets to the curve trades at par (ignoring credit/liquidity, accrued interest, and idiosyncratic funding), because the floating-leg PV telescopes. In a multi-curve world, the coupons are projected from an index curve $P^{(k)}$ but discounted on $P_d$, so that telescoping identity no longer holds.

A simple PV expression (toy, ignoring spreads/convexity) is:
$$
PV \approx \sum_{i=1}^n P_d(0,T_i)\\,N\\,\tau_i\\,F_k(0;T_{i-1},T_i) + P_d(0,T_n)\\,N.
$$

**Expand (where the “par” identity went):** Define the *discount-implied* forward on the same accrual period,
$$F_d(0;T_{i-1},T_i)=\frac{1}{\tau_i}\left(\frac{P_d(0,T_{i-1})}{P_d(0,T_i)}-1\right).$$
In a single-curve world $F_k=F_d$ and the identity $\tau_i F_d(0;T_{i-1},T_i)P_d(0,T_i)=P_d(0,T_{i-1})-P_d(0,T_i)$ makes the floating PV telescope to par. In a multi-curve world, $F_k$ comes from $P^{(k)}$ while PV uses $P_d$, so you can think of the floater as:
$$PV \approx N + N\sum_{i=1}^n P_d(0,T_i)\\,\tau_i\\,\bigl(F_k(0;T_{i-1},T_i)-F_d(0;T_{i-1},T_i)\bigr).$$
The second term is a discounted sum of “basis spreads” between the projection forwards and the discount-implied forwards.

**Check (sanity on Example A):** In the toy setup, the discount-implied forwards are about 2.00% while the 3M projection forwards average around 3.4%, so the spread is about 1.4% per year. With total accrual $\sum\tau_i=1$ and discount factors near 1, the par premium is on the order of $100\times 1.4\\%\approx 1.4$ price points — consistent with the computed $PV\approx 101.40$.

**Example Title**: Pricing a 1Y quarterly 3M-index FRN under OIS discounting

**Context**
- What is being priced? A “par” floater when projection (3M) > discount (OIS).
- Why it matters on the desk: this is the simplest place the single-curve telescoping identity fails, and it creates a basis exposure between projection and discount curves.

**Timeline (Make Dates Concrete)**
- Trade date: 2026-01-05
- Settlement / accrual start: 2026-01-07
- Payment dates: 2026-04-07, 2026-07-07, 2026-10-07, 2027-01-07
- Maturity / principal repayment: 2027-01-07

**Inputs**
- Instrument details: FRN pays a 3M term index set in advance and paid in arrears (toy).
- Notional: $N=USD 100$
- Market quotes (toy):
  - Discount curve (OIS): flat zero rate $r_d=2.00\\%$ with continuous compounding.
  - Projection forwards (3M): 3.20%, 3.40%, 3.50%, 3.60% for the four quarters.
- Day count / compounding: ACT/360 approximated as $\tau_i=0.25$ each quarter.
- Settlement conventions: ignore accrued interest (treat PV as “dirty” per USD 100 at valuation).

**Outputs (What You Produce)**
- PV per USD 100 notional.
- Discount DV01 and projection DV01 (per 1bp), using the chapter convention $DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$.

**Step-by-step**
1. Translate quote to cashflows: project coupons $C_i = N\\tau_i F_i$.
2. Build accrual factors: here $\tau_i=0.25$ (toy ACT/360).
3. Discount / compute PV: use $P_d(0,T_i)=e^{-r_d T_i}$.
4. Compute risk metric (toy parallel bumps, other curves held fixed):
   - Discount DV01: bump $r_d \\to r_d-1\\text{bp}$.
   - Projection DV01: bump each $F_i \\to F_i-1\\text{bp}$.

**Cashflows (projected)**

| Date | Cashflow | Explanation |
|---|---|---|
| 2026-04-07 | USD 0.800 | $100 \\times 0.25 \\times 3.20\\%USD  |
| 2026-07-07 | USD 0.850 | $100 \\times 0.25 \\times 3.40\\%USD  |
| 2026-10-07 | USD 0.875 | $100 \\times 0.25 \\times 3.50\\%USD  |
| 2027-01-07 | USD 0.900 | $100 \\times 0.25 \\times 3.60\\%USD  |
| 2027-01-07 | USD 100.000 | principal |

**PV (toy numbers)**
- Coupon PV: $\approx USD 3.38$
- Principal PV: $\approx USD 98.02$
- **Total PV:** $\approx USD 101.40$ per USD 100 notional

**Risk (toy numbers, per 1bp = 0.0001)**
- Discount DV01 (OIS zero rates down 1bp): $\approx +0.0100$ dollars per USD 100 notional per 1bp.
- Projection DV01 (3M forwards down 1bp): $\approx -0.0099$ dollars per USD 100 notional per 1bp.

**Check (why the projection DV01 is about $-0.0099$):** Bumping each quarterly forward down 1 bp reduces each coupon by $N\tau\times 10^{-4}$. The PV impact is approximately
$$\Delta PV \approx -N\\,10^{-4}\sum_{i=1}^n P_d(0,T_i)\\,\tau_i.$$
In Example B the “discount annuity” is about $0.9876$, so with $N=100$ this gives $-100\times 10^{-4}\times 0.9876\approx -0.0099$, matching the toy number and making the units/sign explicit.

**P&L / Risk Interpretation**
- $PV\gt 100$ because coupons are projected off a higher curve but discounted on a lower curve.
- The opposite signs of the two DV01s are the simplest “basis intuition”: a long floater benefits from lower discount rates but loses value when projected coupons fall.
- Real risk reports usually compute *par-point deltas* by bumping calibration instruments and rebuilding curves (see Chapter 22); the point here is to make the bump object explicit.

**Sanity Checks**
- Units check: coupons are dollars; DV01s are dollars per 1bp per USD 100 notional.
- Sign check: under $DV01 := PV(\\text{rates down})-PV(\\text{base})$, discount DV01 is positive for a long position.
- Limit check: if projection = discount, the PV moves toward par (the single-curve telescoping identity is recovered).

### 19.2.3 Valuation of an Interest Rate Swap

The same logic applies to Interest Rate Swaps (IRS). The par swap rate $K_{par}$ is the fixed rate that equates the PV of the fixed leg to the PV of the floating leg.

$$\boxed{K_{par} = \frac{\sum_{i} \tau_i F_k(T_{i-1}, T_i) P_d(0, T_i)}{\sum_{i} \alpha_i P_d(0, T_i)}}$$

Notice the mix of curves: the numerator uses $F_k$ (Projection) to forecast floating payments and $P_d$ (Discount) to value them. The denominator uses $P_d$ (Discount) to value the fixed annuity.

**Example B: Par Swap Rate Calculation**

Using our numbers above, the "Projection PV" of the floating leg was $USD 3.38$. The "Discount Annuity" (sum of OIS discount factors $\times$ 0.25) is:

$$\text{Annuity} = 0.25 \times (0.9950 + 0.9901 + 0.9851 + 0.9802) = 0.25 \times 3.9504 = 0.9876$$

$$K_{par} \approx \frac{3.38}{100 \times 0.9876} \approx 3.42\\%$$

This par rate of **3.42%** is effectively a weighted average of the high projection forwards (3.2%–3.6%), not the low OIS rates.

### 19.2.4 Building the Projection Curve as a Spread to OIS

Projection curves are usually built in a *curve group* rather than independently. One common approach is:
1. Build a discount curve $P_d$ (often OIS) and a “base” index curve $P^{(1)}$ for the most liquid tenor.
2. Build other tenors as **spread curves** to the base using basis swap quotes.

A convenient representation is multiplicative:
$$
P^{(2)}(t)=P^{(1)}(t)\exp\left(-\int_0^t \eta^{1,2}(s)\\,ds\right),
$$
where $\eta^{1,2}$ is a (typically piecewise-constant) spread function calibrated to basis instruments.

This spread-based construction helps keep risk interpretable: moves in instruments that build the base curve tend to shift many forwards together, while moves in basis quotes primarily change the relative spread between tenors (see Section 19.5).

**Example C: Extracting the Spread**

Suppose we have:
* OIS discount factor: $P_d(0, 1) = 0.9500$
* 3M LIBOR projection pseudo-factor: $P^{3M}(0, 1) = 0.9450$

The implied spread:
$$e^{-\eta \times 1} = \frac{P^{3M}(1)}{P_d(1)} = \frac{0.9450}{0.9500} = 0.9947$$

$$\eta = -\ln(0.9947) = 0.53\\% = 53 \text{ bps}$$

This 53bp spread represents the average OIS-LIBOR basis over the 1-year tenor.

**Expand (what $\eta$ means operationally):** In a spread-based construction, $\eta\gt 0$ corresponds to “projection rates above OIS,” which shows up as $P^{(k)}(0,t)\lt P_d(0,t)$. For small spreads and short accruals, a roughly constant $\eta$ acts like an approximate forward-rate add-on: the projection forward over $[T,T+\tau]$ is about the discount-implied forward plus $\eta$ (to first order).

**Check (order-of-magnitude PV impact):** If $\eta\approx 53$ bp is roughly “extra coupon per year” on a 1Y floater, then a back-of-the-envelope PV premium is
$$N\\,\eta\sum_i \tau_i P_d(0,T_i).$$
With $N=100$ and $\sum \tau_i P_d$ around 0.95–1.00, this is about $100\times 0.0053\times 0.97\approx 0.5$ price points: small per USD 100, but large on real notionals.

> **Desk Reality:** Traders often talk in “spreads to OIS” or “basis” (e.g., “3M over OIS”, “6M vs 3M”).
> **Common break:** Risk is aggregated into a single delta even though the position’s P&L depends on both overall rate level and basis spreads.
> **What to check:** Whether the risk report shows separate buckets for discounting risk vs basis risk (and which curve instruments define each).

### 19.2.5 Cross-Currency Context (Preview)

The multi-curve framework extends naturally to cross-currency settings. When valuing a EUR/USD cross-currency swap, we need:

* **USD OIS curve** for discounting USD cash flows
* **EUR OIS curve** (built via FX forwards and cross-currency basis swaps)
* **EUR EURIBOR projection curve** for forecasting EUR floating payments
* **USD SOFR projection curve** for forecasting USD floating payments

Covered Interest Parity (CIP) provides the linkage: FX forwards, OIS curves, and projection curves must be mutually consistent to avoid arbitrage. When they are not (typically in stress), the cross-currency basis widens.

**Chapter 21 develops this cross-currency framework fully.** Here we note only that the projection curve concept extends seamlessly: each currency has its own projection curves for each relevant tenor, all built as spreads to the appropriate discount curve.

---

## 19.3 The Tenor Basis: A Preview

So far we have distinguished between "Risk-Free" (OIS) and "Risky" (LIBOR/SOFR term). But "Risky" is not a monolith. In the post-crisis world, the market began to brutally distinguish between different *tenors* of risk.

### 19.3.1 Why 1-Month and 3-Month Rates Diverge

Pre-2007, a bank needing 3-month funding could comfortably assume that borrowing for 1 month and rolling the loan twice was roughly equivalent to borrowing for 3 months once. The market priced these nearly identically.

Different tenors are different funding instruments. A 3‑month unsecured loan locks funding for longer than a 1‑month loan; rolling 1‑month funding introduces rollover risk. When credit and liquidity premia are time-varying, the market prices these tenors differently, so “roll three 1M loans = one 3M loan” is not a safe approximation.

Before the crisis, basis spreads between common tenors were often tiny; once they became material, each tenor needed its own projection curve.

### 19.3.2 Economic Drivers of Tenor Basis

Tenor basis is usually explained by a mix of:

1. **Credit Horizon Risk:** A 6-month loan embeds more default risk than a 1-month loan to the same counterparty.
2. **Liquidity / Rollover Risk:** Longer funding reduces refinancing risk, so it can command a premium in stress.
3. **Market Segmentation / Balance Sheet:** Different tenors attract different participants with distinct supply/demand dynamics.

> **Visual: The Fan Chart**
>
> Imagine the yield curve as a fan opening up:
> * **Base Line (Bottom)**: OIS Curve (Risk Free)
> * **First Blade**: 1-Month Projection (Slightly higher)
> * **Second Blade**: 3-Month Projection (Higher still)
> * **Top Blade**: 6-Month Projection (Highest)
>
> The "Tenor Basis" is the space between the blades. In times of stress, the fan opens wide (basis widens). In calm times, the fan closes up (basis compresses).

### 19.3.3 Implications for Multi-Curve Construction

Because different tenors represent fundamentally different risk, each tenor requires its own projection curve:
* $P^{1M}(t)$ — projection curve for 1-month rates
* $P^{3M}(t)$ — projection curve for 3-month rates
* $P^{6M}(t)$ — projection curve for 6-month rates

These curves are linked by **basis swaps** (floating-for-floating swaps that exchange payments on different tenors). The basis swap spread calibrates the relationship between tenor curves.

**Chapter 20 develops the mathematics of tenor basis and basis swap bootstrapping in detail.** Here we emphasize only the conceptual point: you cannot use the 3-month curve to forecast 1-month or 6-month rates without introducing systematic errors.

---

## 19.4 The Transition to SOFR

The principles of the multi-curve framework—separating discounting from projection—remain perfectly valid in the new world of Risk-Free Rates (RFRs) like SOFR (Secured Overnight Financing Rate). However, SOFR introduces new mechanical complexity: it is **backward-looking**.

### 19.4.1 Forward-Looking vs Backward-Looking Rates

A key mechanical difference is *when the coupon rate becomes known*:
- **LIBOR-style term rates:** set at the beginning of the accrual period (known in advance).
- **SOFR-style overnight-compounded rates:** built from realized overnight rates through the accrual period (known only near the end).

Key point: “The rate applicable to a particular period is not known until the end of the period when all the relevant overnight rates have been observed.”

> **Analogy: Weather Forecast vs Thermometer**
>
> * **LIBOR** is like a weather forecast: At the start of the week, someone predicts "It will average 75°F this week." You know upfront.
> * **SOFR** is like reading a thermometer: You record the temperature each day, and at week's end you calculate "This week averaged 74.3°F." You only know afterward.
>
> The operational challenge: How do you make a payment on time if the coupon is only known at the period end?

### 19.4.2 SOFR Index Types

The SOFR market has developed multiple index types to address different operational and hedging needs. Understanding these is essential for middle-office professionals who see them on trade confirmations.

**1. Compound-in-Arrears (Standard)**

This is the most common convention for SOFR derivatives. Daily overnight rates are compounded over the accrual period:

Short version: “Longer rates such as three-month rates, six-month rates, or one-year rates can be determined from overnight rates by compounding them daily.”

$$\boxed{\text{Compounded Rate} = \left[\prod_{i=1}^{n}(1 + r_i \cdot \hat{d}_i) - 1\right] \times \frac{360}{D}}$$

where $r_i$ is the overnight SOFR on day $i$, $\hat{d}_i = d_i/360$ is the day fraction, $d_i$ is the number of days that rate applies (usually 1, but 3 over weekends), and $D = \sum_i d_i$ is the total days in the period.

This is the standard “daily compounding” construction: longer-tenor rates are built from realized overnight rates by compounding day by day.

**Checks (units + “don’t forget the annualization”):** Each factor $(1+r_i\hat d_i)$ is dimensionless, so the product is a pure growth factor over the period. The bracketed term $\left[\prod(1+r_i\hat d_i)-1\right]$ is the *period* return; multiplying by $360/D$ annualizes it on an ACT/360 basis. A common implementation error is to annualize twice (or not at all), producing coupons that are off by a factor of roughly $D/360$.

**Example D: SOFR Compound-in-Arrears Calculation**

Calculate the compounded SOFR rate for a 7-day period (Monday through Sunday):

| Day | SOFR Rate | Days Applied | Growth Factor |
|-----|-----------|--------------|---------------|
| Mon | 5.30% | 1 | $1 + 0.0530 \times \frac{1}{360} = 1.0001472$ |
| Tue | 5.32% | 1 | $1 + 0.0532 \times \frac{1}{360} = 1.0001478$ |
| Wed | 5.31% | 1 | $1 + 0.0531 \times \frac{1}{360} = 1.0001475$ |
| Thu | 5.29% | 1 | $1 + 0.0529 \times \frac{1}{360} = 1.0001469$ |
| Fri | 5.30% | 3 | $1 + 0.0530 \times \frac{3}{360} = 1.0004417$ |

**Step 1:** Compound the daily factors:
$$\prod = 1.0001472 \times 1.0001478 \times 1.0001475 \times 1.0001469 \times 1.0004417 = 1.001031$$

**Step 2:** Annualize:
$$\text{Compounded Rate} = (1.001031 - 1) \times \frac{360}{7} = 5.303\\%$$

**Sanity check:** The compounded rate (5.303%) is slightly higher than the arithmetic average of daily rates (~5.30%) because compounding adds a small convexity pickup. ✓

**2. Daily Simple SOFR**

For some lending products, particularly bilateral loans, daily rates are simply averaged (not compounded):

$$\text{Simple Average} = \frac{\sum_i r_i \cdot d_i}{D}$$

This convention is operationally simpler but mathematically less precise than compounding.

**3. Other published variants (overview)**

In practice you may encounter additional published variants (e.g., rolling averages or forward-looking “term” rates). For multi-curve thinking, the key is simple: treat each variant as a **distinct index**. If your cash exposure references one variant but your hedge references another, you should expect **basis risk** on top of ordinary curve risk.

### 19.4.3 Observation Conventions

Because the effective SOFR rate is only known near the end of the accrual period, contracts need a convention that gives operations time to compute and pay the coupon. Common approaches include:

**1. Pay after the period ends**

The accrual period ends on date $T$, but the payment date is a few business days later.

**2. Use an earlier observation window**

The coupon is computed from overnight rates observed over a window that ends before $T$, so the amount can be calculated before payment.

**3. Approximate the last few days**

Some products approximate the final observations (e.g., by freezing a rate for the last few days) to reduce operational timing pressure.

Different conventions produce slightly different coupons and slightly different hedging behavior.

**Expand (what changes in risk terms):** Observation conventions are not just “ops detail” — they change *which days’ fixings* map into a coupon. A lookback/shift moves the exposure earlier; a lockout/freezing convention reduces sensitivity to the last few days’ fixings (replacing them with an earlier rate). If your hedge is calibrated to one convention but your cash exposure uses another, you are holding a small but persistent “convention basis” position.

**Check (toy lockout effect):** Suppose a coupon uses a 2-day lockout (the last 2 days use the last observed rate) and overnight rates jump by +100 bp on the final 2 days of a 1-week accrual. The coupon on a notional $N$ is lower by roughly $N\times 0.01\times (2/360)\approx 0.0000556N$. On $N=USD 100$mm, that is about $USD 5{,}600$ for that coupon — “small,” but large enough to show up as unexplained P&L if you are hedged under a different convention.

> **Desk Reality:** A common “mystery P&L” on RFR books is a convention mismatch between the cash exposure and the hedge.
> **Common break:** A loan’s coupon is computed with one convention but the hedge swap uses another; the difference is small each period but persistent.
> **What to check:** The exact convention fields in the confirmation (observation window and payment timing) on both legs.

### 19.4.4 The Credit-Sensitivity Gap: Why Banks Resisted SOFR

A crucial economic concept that explains the contentious nature of the LIBOR transition is the **credit-sensitivity gap**.

LIBOR included a credit spread—it reflected the cost of unsecured lending between banks. When market stress increased, LIBOR rose (banks charged more to lend to each other). This meant that banks holding LIBOR-linked assets saw their income rise when their funding costs rose. There was a natural hedge.

SOFR, being a secured overnight rate, does not include credit spread. In a stress scenario:
* **Bank funding costs rise** (banks still borrow unsecured; counterparties demand more)
* **SOFR-linked asset income stays flat** (SOFR is secured, unaffected by bank credit)

This creates a **funding mismatch**: banks' assets don't re-price when their liabilities do.

> **Analogy: Insurance Premium That Doesn't Rise**
>
> Imagine you're a homeowner. Your insurance premium rises every time there's a hurricane (that's LIBOR—you pay more when risk rises, but your income from renting the house also rises because tenants pay LIBOR-linked rent).
>
> Now imagine your rent income is locked to a "hurricane-free rate" (that's SOFR). When hurricanes hit:
> * Your insurance costs spike
> * Your rental income stays flat
> * You have a cash flow crisis
>
> This is the bank's problem with SOFR. It's economically "safer" for the system, but it shifts risk onto bank balance sheets.

### 19.4.5 Basis Risk from Index Variants (Desk View)

Even when two trades are both labeled “SOFR,” they may reference **different coupon definitions**:
- compounded-in-arrears vs daily simple averaging; and/or
- different observation window and payment timing conventions.

If the exposure and hedge use different definitions, you are effectively holding a **basis position** between two highly correlated but non-identical indices.

**Example E (Conceptual): Simple vs Compounded**

If the daily overnight rate were constant through the accrual period, simple averaging and compounding would produce (approximately) the same coupon. When rates vary day to day, compounding typically produces a slightly higher realized rate than a simple average. The difference is small per period, but it can accumulate into persistent P&L on large notionals.

### 19.4.6 Credit-Sensitive Rate Alternatives

Because SOFR is designed to be “risk-free-ish,” some market participants explored adding a credit-spread component for certain cash products. These alternatives can reduce the credit-sensitivity gap for some balance sheets, but they also introduce **basis risk** versus SOFR-based derivatives.

This chapter does not attempt to standardize or recommend a specific credit-sensitive benchmark; treat it as a reminder that “everything is SOFR” does not eliminate basis.

### 19.4.7 The Return of the Single Curve?

Interestingly, for a standard SOFR OIS swap (where we pay fixed and receive compounded SOFR), the "Projection" curve and the "Discount" curve are conceptually the same (both are SOFR). In this specific corner of the market, the "single curve" world has effectively returned.

However, as soon as we deal with a **legacy LIBOR** exposure, a different **index/tenor**, or **foreign currency** cash flows, the multi-curve distinction comes roaring back. The modern desk must handle both regimes simultaneously.

---

## 19.5 Managing Risk in a Multi-Curve World

The shift to multiple curves complicates risk management. The simple question "what is my delta?" now has two answers.

### 19.5.1 Discount Risk vs. Projection Risk

Risk numbers are only meaningful once you state:
- **bump object:** which curve (discount or which projection curve), and whether you bump zero rates, par quotes, or spreads;
- **bump size:** $1\text{bp}=10^{-4}$;
- **units:** currency per 1bp (per trade notional); and
- **sign convention:** how the bump is defined.

In this book we use the convention:
$$
DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base}),
$$
for the stated bump object. (So a “long rates” position typically has positive DV01.)

In a multi-curve setup you typically need at least three scalars:
- **Discount DV01:** bump the *discount curve* zero rates down 1bp (parallel), rebuild discount factors; hold projection curves fixed.
- **Projection DV01 (index $k$):** bump the *projection curve* used to generate forwards for index $k$ down 1bp (parallel), so projected coupons fall; hold the discount curve fixed.
- **Basis DV01:** bump the *spread* between a projection curve and its chosen base curve down 1bp (e.g., $\eta^{1,k}\\to\\eta^{1,k}-1\\text{bp}$), holding the base curve fixed.

Units should always be stated (e.g., “USD per 1bp per USD 100 notional” in toy examples; “USD per 1bp per USD 100mm notional” on a desk).

In the worked FRN example in Section 19.2.2, the toy numbers illustrate the decomposition: the discount DV01 is positive (rates down increases PV via discounting), while the projection DV01 is negative for a long floater (rates down reduces projected coupons).

### 19.5.2 Hedge Instrument Mapping

| Risk Type | Description | Hedge Instruments |
|-----------|-------------|-------------------|
| **OIS/Discount Risk** | Sensitivity to collateral rate | OIS swaps, Fed Funds futures, SOFR futures |
| **Projection Risk (SOFR)** | Sensitivity to SOFR curve | SOFR OIS swaps, SOFR futures |
| **Index Variant / Convention Basis** | Sensitivity to differences between index definitions (compounded vs simple; observation window/payment timing) | Match conventions when hedging; otherwise treat as basis risk |
| **Tenor Basis Risk** | Sensitivity to spread between tenors | Basis swaps (3M vs 6M, etc.) |
| **OIS-Index Basis** | Sensitivity to OIS-LIBOR/SOFR spread | OIS vs 3M basis swaps |

### 19.5.3 Orthogonality in Risk Sensitivities

A robust curve construction strives for **orthogonality**: the market inputs used to build the curve group are chosen so that different perturbations have clear, mostly non-overlapping meanings. In a spread-based curve group, a useful mental model is:

- Perturbations to instruments that build the **base index curve** (e.g., non-basis swaps/FRAs on the base tenor) largely define *overall rate level* risk.
- Perturbations to **funding / discounting instruments** largely define *discounting* risk.
- Perturbations to **basis swap spreads** largely define *basis* risk (tenor-vs-tenor or index-vs-OIS spreads).

This is not magic: real implementations can still have cross-effects (depending on interpolation, pillar choices, and rebuild logic). The point is that *how you build curves* strongly influences *how your risk reports behave* (see Chapter 22).

### 19.5.4 Basis Trading

> **Desk Reality:** “Long basis” usually means you benefit if a spread (index–OIS or tenor–tenor) widens; “short basis” means you benefit if it tightens.
> **Common break:** A “rates hedge” built only with OIS instruments hedges discounting risk but can leave projection/basis risk largely untouched.
> **What to check:** Which spread you are implicitly long/short (index–OIS, 6M–3M, convention basis) and which instruments actually trade that spread.

---

## 19.6 Practical Notes

### 19.6.1 Curve Construction Hierarchy

Build curves in order of liquidity:
1. **OIS (Discount):** From OIS swaps and Fed Funds/SOFR futures (Chapter 18)
2. **Base Projection (SOFR or legacy 3M LIBOR):** From vanilla fixed-float swaps
3. **Secondary Tenors (1M, 6M):** From basis swaps as spreads to the base (Chapter 20)

### 19.6.2 Common Pitfalls

| Pitfall | Description | Consequence |
|---------|-------------|-------------|
| **Wrong discount curve** | Legacy systems may still discount at LIBOR | Systematic mispricing; P&L breaks |
| **Ignoring tenor basis** | Hedging 6M exposure with 3M instruments | Residual basis risk |
| **Confusing pseudo-DF** | Treating projection curve $P_k$ as real prices | Wrong present values |
| **Convention mismatch** | Mixing SOFR conventions (compounded vs simple; observation window/payment timing) | Persistent basis P&L |
| **Missing credit gap** | Not understanding SOFR vs funding cost | Unexpected P&L in stress |

### 19.6.3 Implementation Checklist

* **Repricing Check:** After building curves, reprice every input instrument. Errors should be $\lt 10^{-10}$.
* **Orthogonality Test:** Bump basis spreads and verify only projection curves move, not discount.
* **Convention Match:** Ensure day counts (ACT/360 vs ACT/365) match the specific index.
* **RFR Convention Match:** Verify the observation window and payment timing convention match the instrument.

---

## Summary

1. **Discount at the Collateral Curve:** Value cash flows using the OIS discount curve (when the trade is collateralized under a standard CSA).

2. **Project at the Index Curve:** Forecast floating coupons using the projection curve for that index/tenor (e.g., 3M SOFR, 6M EURIBOR).

3. **Par Floaters Aren’t Par (Multi-Curve):** A floater paying index $k$ will generally not trade at par when discounted on OIS, because the coupon forecasts and discounting use different curve objects.

4. **SOFR is Backward-Looking:** Compounded-in-arrears coupons are not known until the end of the accrual period; “term” variants exist to meet operational needs.

5. **Credit-Sensitivity Gap:** A secured overnight rate can diverge from bank funding costs in stress, which is why “LIBOR replacement” involved real economics, not just mechanics.

6. **Index-Variant (Convention) Basis Exists:** Even within “SOFR,” coupon definitions can differ (compounded vs simple; observation window/payment timing). Mismatches between exposure and hedge create basis P&L.

7. **Decompose Your Risk:** Separate discount-curve DV01, projection-curve DV01, and basis (spread) DV01 so your hedges target the right object.

8. **Tenor Basis Preview:** Different funding tenors (1M, 3M, 6M) can require distinct projection curves, developed in Chapter 20.

The multi-curve framework is more complex than the single-curve world. But it is also more honest. It forces us to confront the reality that liquidity has a price, credit has a price, and in financial markets, not all dollars are created equal.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Discount Curve** | The OIS curve used to present-value all cash flows | Reflects the true cost of funding collateralized trades |
| **Projection Curve** | Tenor-specific curve (e.g., 3M SOFR) used to forecast floating fixings | Ensures forward rates match market FRA/swap quotes |
| **Pseudo-Discount Factor** | $P_k(T)$ values that reproduce forward rates via standard formula | Not real prices—mathematical machinery for projection |
| **Multi-Index Curve Group** | Collection of one discount + multiple projection curves | The only consistent framework for post-crisis pricing |
| **Spread-Based Construction** | Building projection curve as $P^{(k)} = P_d \cdot e^{-\int \eta}$ | Ensures orthogonal risk decomposition |
| **Tenor Basis** | Spread between forward rates of different tenors | Represents credit/liquidity risk differences by funding horizon |
| **Par Floater Paradox** | LIBOR FRN trades above par when LIBOR > OIS | Classic manifestation of the two-curve divergence |
| **SOFR Compound-in-Arrears** | Standard SOFR: daily rates compounded over period | Backward-looking; not known until period end |
| **Index Variant / Convention Basis** | Different contractual definitions of “SOFR” (compounded vs simple; observation window/payment timing) | Creates basis risk even when both legs are labeled “SOFR” |
| **Credit-Sensitivity Gap** | SOFR doesn't rise with bank funding costs | Explains bank resistance to SOFR transition |
| **Orthogonality** | Discount and projection risks are independent | Enables clean hedging and risk attribution |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $P_d(t,T)$ | discount factor on the discount (OIS/collateral) curve | unitless; $P_d(T,T)=1$ |
| $P_k(t,T)$ or $P^{(k)}(t,T)$ | pseudo-discount factor used to imply forwards for index/tenor $k$ | unitless; mathematical construct (not a bond price) |
| $F_k(t;T_1,T_2)$ | forward rate for index $k$ over $[T_1,T_2]$ | annualized; uses accrual factor $\tau$ |
| $\tau$ | floating-leg year fraction | ACT/360 in toy examples unless stated |
| $\alpha$ | fixed-leg year fraction | instrument-specific |
| $\eta^{(k)}(t)$ | spread function linking a projection curve to a base/discount curve | in rate units; often piecewise-constant in practice |
| $K_{par}$ | par swap fixed rate | annualized |
| $DV01$ | PV sensitivity to a 1bp down bump | currency per 1bp; $DV01=PV(\text{rates down }1\text{bp})-PV(\text{base})$ for the stated bump object |
| $r_i$ | overnight SOFR on day $i$ | annualized |
| $d_i$ | number of days that $r_i$ applies | days |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Why do we separate discounting and projection curves? | Because the curve that PVs collateralized cashflows (often OIS) need not be the same curve that implies forwards for a risky/tenor index; a single curve cannot fit both sets of market quotes when spreads are material. |
| 2 | What is the discount curve $P_d$? | The discount factors used to present-value cashflows (often an OIS curve under a CSA). |
| 3 | What is a projection curve $P^{(k)}$? | A curve object used to generate forwards and forecast coupons for a specific index/tenor $k$. |
| 4 | What is a pseudo-discount factor? | A mathematical construct that reproduces forwards via $F_k=\frac{1}{\tau}\left(\frac{P^{(k)}(T_1)}{P^{(k)}(T_2)}-1\right)$; it is not a tradable bond price. |
| 5 | Why does a floater paying index $k$ generally not price at par under OIS discounting? | Coupon forecasts come from $P^{(k)}$ but PV uses $P_d$, so the single-curve telescoping identity fails when the curve objects differ. |
| 6 | What is a multi-index curve group? | One discount curve plus multiple projection curves (one per index/tenor), calibrated as a group. |
| 7 | What does spread-based construction mean? | Building a projection curve as a spread to a base curve (e.g., $P^{(k)}=P_d e^{-\int \eta^{(k)}}$) so basis quotes map cleanly into basis risk. |
| 8 | What is tenor basis? | The spread between forward rates of different tenors (e.g., 3M vs 6M), reflecting different credit/liquidity horizons and supply/demand. |
| 9 | In this book, what does “DV01” mean? | $DV01=PV(\text{rates down }1\text{bp})-PV(\text{base})$ for the stated bump object; units are currency per 1bp per notional. |
| 10 | What is discount DV01 vs projection DV01? | Discount DV01 bumps the discount curve; projection DV01 bumps the index curve used for forwards (holding the other fixed). |
| 11 | What is basis DV01? | Sensitivity to the spread between curves (e.g., a spread function $\eta$), not to an overall rate level. |
| 12 | What does “orthogonality” mean in curve risk? | Curve construction aims for level, discounting, and basis risk to respond to different market inputs with minimal cross-talk. |
| 13 | Desk reality: why can “a single DV01” be misleading? | You can be hedged to OIS discounting and still have large projection/basis exposure if the report aggregates everything into one number. |
| 14 | What is “compound-in-arrears” SOFR? | A backward-looking coupon where overnight rates are compounded through the accrual period. |
| 15 | What creates “convention basis” on RFR books? | Mismatches in operational fields (observation window and payment timing) between the exposure and the hedge. |
| 16 | What is the “credit-sensitivity gap”? | A secured overnight rate can diverge from bank funding costs in stress; assets indexed to it may not reprice when liabilities do. |
| 17 | What instruments hedge projection and basis risks? | Projection: futures/FRAs/swaps on the same index; basis: basis swaps (floating-for-floating) and convention-matched hedges. |
| 18 | What should you write down before trusting a risk number? | Bump object, bump size (1bp), units, and sign convention. |

---

## Mini Problem Set

1. (Compute) You observe the following curve objects:
   - OIS discount factor: $P_d(0,0.5)=0.9900$
   - 3M projection pseudo-factors: $P^{3M}(0,0.25)=0.9940$, $P^{3M}(0,0.5)=0.9870$
   
   Compute the 3M forward rate for $[0.25,0.5]$ using $\tau=0.25$.

2. (Compute) An FRN pays 3M LIBOR quarterly for 1 year. The four quarterly 3M forwards are 3.0%, 3.1%, 3.2%, 3.3%. The OIS curve is flat at 2.5% (continuous compounding). Compute the FRN PV per USD 100 notional (assume $\tau=0.25$ each quarter).

3. (Compute) Given $P_d(0,1)=0.9500$ and a 3M projection pseudo-factor $P^{3M}(0,1)=0.9430$, compute the implied average OIS–3M spread $\eta$ (in bp) in the representation:
   $$
   P^{3M}(0,1)=P_d(0,1)e^{-\eta\cdot 1}.
   $$

4. (Compute) A receiver swap (receive fixed, pay 3M) has the following risk measures under this chapter’s convention $DV01=PV(\text{rates down }1\text{bp})-PV(\text{base})$:
   - Discount DV01 (OIS zero rates down 1bp): +USD 8,500 per 1bp
   - Projection DV01 (3M forwards down 1bp): +USD 9,200 per 1bp
   
   Estimate P&L if OIS rates rise 5bp and 3M forwards rise 8bp using $\Delta PV\approx- DV01_d\\,\Delta bp_d- DV01_{3M}\\,\Delta bp_{3M}$.

5. (Compute) Calculate the compounded-in-arrears rate for a 7-day period (ACT/360) with daily overnight rates:

   | Day | Rate | Days applied |
   |---|---:|---:|
   | Mon | 4.80% | 1 |
   | Tue | 4.82% | 1 |
   | Wed | 4.79% | 1 |
   | Thu | 4.81% | 1 |
   | Fri | 4.80% | 3 |
   
   Use $\left[\prod_i (1+r_i d_i/360)-1\right]\times 360/7$.

6. (Concept) Show the single-curve telescoping identity that makes a floater price at par, and explain precisely why it fails when forwards come from $P^{(k)}$ but discounting uses $P_d$.

7. (Desk) A risk report shows only one “USD DV01” for a book of LIBOR floaters hedged with OIS swaps. Give two ways this can break and one concrete check you would run.

8. (Desk/Concept) You hedge a compounded-in-arrears exposure with a hedge that has a different observation window/payment timing convention. What risk remains and how would you detect it in P&L?

### Solution Sketches (Selected)
1. $F=\frac{1}{0.25}\left(\frac{0.9940}{0.9870}-1\right)=2.84\\%$ (annualized).
2. $PV\approx 100.63$ per USD 100 notional (coupons $\approx 3.10$ and principal $100e^{-0.025}\approx 97.53$).
3. $\eta=-\ln\left(\frac{0.9430}{0.9500}\right)\approx 0.7396\\%\approx 74\text{ bp}$.
4. $\Delta PV\approx-(8{,}500)(5)-(9{,}200)(8)=-USD 116{,}100$.
5. $\prod_i (1+r_i d_i/360)\approx 1.0009342$, so the annualized rate is $(1.0009342-1)\times 360/7\approx 4.805\\%$.
6. In a single curve, $F=\frac{1}{\tau}\left(\frac{P(T_{i-1})}{P(T_i)}-1\right)$ implies $\tau F P(T_i)=P(T_{i-1})-P(T_i)$, so the float PV telescopes. In multi-curve, $F_k$ comes from $P^{(k)}$ but PV uses $P_d$, so $\tau F_k P_d(T_i)$ no longer equals $P_d(T_{i-1})-P_d(T_i)$.
7. Example breaks: (i) you are hedged to discounting moves but not to projection/basis moves; (ii) curve rebuild choices leak basis moves into “rates DV01.” A concrete check: run two bumps (discount curve vs projection curve) and confirm the PV changes match the reported buckets.

---

## References

- Andersen & Piterbarg, *Interest Rate Modeling* (Vol. I), “6.4.2 Forward Rate Approach” (multi-index curve groups; pseudo-discount factors; spread-based curve construction; risk decomposition).
- Oosterlee, *Mathematical Modeling and Computation in Finance*, “14.4.2 Multiple curves and the Libor rate” and “14.4.3 Valuation in a multiple curves setting” (two-curve valuation mechanics; basis swaps as linking instruments).
- Neftci, *Principles of Financial Engineering*, “24.9 Choice of the Discount Rate and Multiple Curves” (OIS discounting motivation; multiple-curve framing).
- Hull, *Options, Futures, and Other Derivatives*, “The New Reference Rates” (SOFR vs LIBOR; daily compounding; forward-looking vs backward-looking).
- Hull, *Risk Management and Financial Institutions*, “The OIS Rate” (LIBOR–OIS spread and stress intuition).
- Crépey, *Counterparty Risk and Funding*, “Remark 6.4.2 (Multi-curve interest-rate modeling)” (post-crisis multi-curve modeling motivation).
