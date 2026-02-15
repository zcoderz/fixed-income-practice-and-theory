# Chapter 20: Tenor Basis — 1M vs 3M vs 6M and Basis Swap Logic

---

## Introduction

If you can borrow or lend at a 3-month floating rate, why not replicate it by rolling a 1-month rate three times? In a frictionless single-curve world, “same horizon” funding would be interchangeable.

In practice, it is not. Different reset tenors live in different funding and hedging markets and can embed different credit, liquidity, and balance-sheet premia. Modern rates pricing therefore uses **multi-curve** logic: one curve for discounting (typically an OIS curve for collateralized trades) and tenor-specific **projection curves** for forecasting floating coupons. Once you have more than one projection curve, the **tenor basis** is the price of transforming exposure from one tenor to another.

This chapter focuses on the single-currency tenor dimension (e.g., 1M vs 3M vs 6M). We connect:
- quote → cashflows/rates object (basis swap legs and schedules)
- cashflows → PV equation (discount curve + projection curves)
- PV equation → risk (discount DV01, projection DV01, and basis DV01 with explicit bump objects)

Prerequisites: [Chapter 17 — Curve Construction](chapter_17_curve_construction_bootstrapping_interpolation.md), [Chapter 18 — OIS Discounting Curve](chapter_18_ois_discounting_curve.md), [Chapter 19 — Projection Curves (Multi-Curve)](chapter_19_projection_curves_libor_sofr_multi_curve.md)  
Follow-on: [Chapter 21 — Cross-Currency Curves](chapter_21_cross_currency_curves.md), [Chapter 22 — Multi-Curve Risk (Jacobians)](chapter_22_multi_curve_risk_jacobians.md), [Chapter 26 — Swap PV01/DV01 and Hedging](chapter_26_swap_pv01_dv01_hedging.md)

## Learning Objectives
- After this chapter, you can define tenor basis and explain why it can persist.
- You can translate a basis-swap quote into a cashflow schedule and a par-spread PV equation.
- You can compute and interpret spread-leg PV01 (“Spread01”) and a basis DV01 with explicit units and sign.
- You can describe (at a high level) how a secondary tenor projection curve is bootstrapped from basis swaps.
- You can explain why DV01 hedges can leave residual basis risk and what instruments hedge each component.

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

Historically, IBOR-style term benchmarks were the dominant reference for these tenors. In many markets today, overnight rates dominate discounting for collateralized derivatives, but term-reset cashflows remain common in loans and legacy contracts.

### 20.1.2 Why Tenors Are Not Interchangeable

The labels “1M,” “3M,” and “6M” are not merely calendar arithmetic. They represent **different funding exposures** and are therefore priced differently when markets care about credit, liquidity, collateral, or balance-sheet constraints.

**Credit horizon risk (unsecured term rates).** Holding everything else equal, lending unsecured for longer exposes you to default over a longer horizon. That can push longer-tenor unsecured term rates above shorter-tenor ones.

**Liquidity / balance-sheet value of term funding.** Locking funding for longer can be valuable (or costly) depending on the balance sheet constraints of market participants. When balance sheet is scarce, “transforming” one reset tenor into another is not free.

**Market segmentation.** Different tenors can trade in markets with different natural supply/demand and different liquidity. Even if two tenors reference the same currency, they may not be perfectly substitutable in stressed conditions.

In calm markets, these differences can be small. In stress, they can dominate P&L and make “roll shorter tenor” and “borrow longer tenor” meaningfully different trades.

> **Why this matters:** Treat the difference between tenors as a quoted input (a basis), not as a modeling identity. If you impose “6M = compounded 3M” you are forcing a constraint the market may not satisfy, so your PV and risk decomposition will be wrong.

### 20.1.3 The RFR Transition: Tenor Basis in a Post-LIBOR World

Tenor basis is not “a LIBOR-only phenomenon”. Even if the market moves toward overnight-based discounting and new benchmark families, basis can still appear whenever:
- floating cashflows are **set** using one index/tenor/compounding convention, but
- those cashflows are **discounted** using a different curve, and/or
- different reset frequencies trade in markets with different liquidity and constraints.

So the key lesson is structural: define your discount curve, define your projection curve(s), and treat any residual differences between tenors as a quoted and hedgeable **basis**.

---

## 20.2 The Drivers of Tenor Basis

### 20.2.1 Defining Tenor Basis

**Tenor basis** is observed in the market through **basis swaps** between two reset tenors. For each maturity \(T\), a quote specifies a **par spread** \(e^{1,2}(T)\) that is added to one floating leg so that the swap has \(PV(0)=0\). Quotes across maturities form a tenor-basis **curve**.

In a multi-curve setup, different tenors use different projection curves \(P^{(1)}\) and \(P^{(2)}\) to generate forwards, so in general:

$$P^{(1)}(0,T) \neq P^{(2)}(0,T).$$

The basis swap par spread \(e^{1,2}(T)\) is then determined by equating the discounted PVs of the two floating legs using the chosen discount curve \(P_d\).

### 20.2.2 Empirical Behavior

Tenor basis is a market price: it can be positive or negative, and it can move significantly over time. Qualitatively, it is often shaped by:

- **Funding / liquidity conditions and balance-sheet constraints.** When term funding becomes scarce, the price of transforming one tenor exposure into another can change quickly.
- **Segmentation by tenor.** Natural flows (issuance, hedging demand, investor preferences) can concentrate in one reset frequency and make tenors imperfect substitutes.
- **Calendar effects.** Quarter-end and year-end “turns” can distort short-dated forwards and therefore basis swap pricing for periods spanning reporting dates.

The key modeling point is practical: the basis curve is not mechanically implied by a single yield curve. Treat it as an observable curve inferred from basis swap quotes, and be explicit about the discount curve vs the projection curves used.

### 20.2.3 Historical Context: The 2007-2008 Regime Change

Before 2007, many desks treated both (i) the difference between overnight discounting and term unsecured rates, and (ii) the difference between term tenors, as small enough to ignore for vanilla pricing and hedging. Two shortcuts were common:
- using a single curve for both discounting and projection, or
- building “one curve per tenor” without insisting on a single discounting curve for collateralized trades.

When those spreads became large and variable during the crisis, the shortcuts stopped being harmless. The market converged on two ideas:
- **Separate discounting from projection** for collateralized derivatives (discount with an OIS-style curve).
- **Separate projection curves by tenor** when the market quotes non-negligible basis swaps between tenors.

Tenor basis and index–discounting basis often move together in stress, but they are distinct objects and hedge with different instruments.

### 20.2.4 Connection to Cross-Currency Basis

Tenor basis (within a currency) and cross-currency basis (between currencies) are both “transformation prices”: they tell you what it costs to convert one set of floating exposures into another under real-world frictions (credit, collateral, liquidity, and balance sheet). The instruments and conventions differ, but the modeling philosophy is the same: be explicit about **what is discounted**, **what is projected**, and **what is treated as a quoted basis spread**.

We focus on single-currency tenor basis here; see Chapter 21 for the cross-currency dimension.

---

## 20.3 Basis Swaps: The Trading Instrument

### 20.3.1 Structure of a Basis Swap

A **basis swap** exchanges floating payments linked to one index/tenor against floating payments linked to another. Unlike a fixed–float swap, **both legs are floating**, and each leg follows its own reset and payment schedule.

For example, a "3M vs 6M" basis swap might exchange:

- **Leg 1**: 3M LIBOR + Spread, paid quarterly
- **Leg 2**: 6M LIBOR flat, paid semi-annually

A fixed spread is typically added to one of the legs to make the swap value at inception equal to zero. If a direct floating–floating basis quote is illiquid, the exposure can often be synthesized from two fixed–float swaps that reference the two tenors.

### 20.3.2 The Par Basis Spread

The **par basis spread** $e_{1,2}(T)$ is the spread added to one leg such that the net present value of the swap is zero at inception.

One convenient way to write the par condition (spread added to the tenor-1 leg) is:

$$\boxed{\sum_{i=0}^{n^{2}(T)-1} L^{2}(0; t_{i}^{2}, t_{i+1}^{2}) \,\tau_{i}^{2}\, P_d(0,t_{i+1}^{2})
= \sum_{i=0}^{n^{1}(T)-1}\left(L^{1}(0; t_{i}^{1}, t_{i+1}^{1})+e^{1,2}(T)\right)\,\tau_{i}^{1}\, P_d(0,t_{i+1}^{1})}$$

Here $P_d$ is the discount curve, and $e^{1,2}(T)$ is the quoted spread for exchanging tenor 1 for tenor 2 to maturity $T$. The market convention for which leg carries the spread varies; the equation above is a *notation choice* that makes the algebra explicit.

Algebraically, if the tenor-2 floating leg PV exceeds the tenor-1 floating leg PV when both are priced without a spread, then the par spread \(e^{1,2}(T)\) (when added to tenor 1, as written above) must be **positive** to restore \(PV(0)=0\). If the inequality flips, the par spread flips.

### 20.3.3 Interpreting the Spread Sign

Quoting conventions differ across markets and data vendors, so you must treat the sign as part of the contract definition.

> **Pitfall — Quote convention sign:** A label like “3M/6M +8bp” can mean different things in different systems (which leg carries the spread; pay vs receive).
> **Why it matters:** A sign/leg mistake flips the position: your “hedge” becomes additional basis risk.
> **Quick check:** Write the PV equation with cashflow signs. Ask: “If the quoted spread increases by 1bp, does my PV go up or down?” If you can’t answer, you don’t have the quote definition.

### 20.3.4 Who Trades Basis Swaps and Why

Basis swaps show up whenever an entity’s **assets** and **liabilities** (or hedge instruments) reference different floating indices or reset tenors. A basis swap converts one floating exposure into another so that a portfolio can:
- align floating receipts with floating payments,
- isolate the spread between two tenors as an explicit risk factor, or
- hedge a tenor mismatch created by legacy contracts and market conventions.

Dealers typically intermediate between end users, so basis levels can reflect both hedging demand and the cost of balance sheet.

### 20.3.5 Market Conventions (What to Check Before Trading)

Tenor basis swaps are deceptively simple. Before you trade, lock down the conventions:

1. **Which leg carries the spread, and the sign convention**
   - Does the quote mean *3M + e vs 6M flat*, or *6M + e vs 3M flat*?
   - Is the basis quoted as “pay spread” or “receive spread” on the shorter-tenor leg?

2. **Index definitions**
   - Day count and holiday calendar (often inherited from the underlying index).
   - Fixing conventions and any lookback/payment delay.

3. **Schedule details**
   - Effective date (spot-lag) and whether there are stubs.
   - Payment frequency mismatch (e.g., quarterly vs semiannual).

4. **Liquidity reality**
   - Major-currency IBOR markets were relatively standardized; newer RFR-based basis markets and many EM markets are often more bespoke. Treat “the term sheet” as the source of truth.

---

## 20.4 Mathematics of Multi-Curve Valuation

### 20.4.1 Forward Rates from Projection Curves

Define the tenor-$k$ projection curve $P^{(k)}$ so that forward rates over $[T_{i-1},T_i]$ satisfy:

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

The **par fixed rate** $c^*$ is the rate that makes $V^{(k)}(0) = 0$:

$$\boxed{c^\star = \frac{\sum_{i=0}^{n-1} L^{(k)}(0; t_{i}^{k}, t_{i+1}^{k}) \,\tau_{i}^{k}\, P_d(0,t_{i+1}^{k})}{\sum_{i=0}^{n-1} \tau_{i}^{k}\, P_d(0,t_{i+1}^{k})}}$$

Crucially, the par rate depends on **two** curves: the projection curve determines the forwards in the numerator, while the discount curve determines the weights in both numerator and denominator.

> **Worked Example 20.2: 3M vs 6M Par Rates**
>
> Consider a 1-year swap. Because the projection curves differ:
>
> - **3M Swap**: Uses 3M forwards averaging, say, 2.50%. Reprices quarterly. The par rate might be **2.50%**.
> - **6M Swap**: Uses 6M forwards. Since 6M rates include a higher credit/liquidity premium, the forwards might average 2.60%. The par rate might be **2.60%**.
>
> This 10 basis point difference is a direct manifestation of the tenor basis. Quoting “the 1-year swap rate” without specifying the tenor is imprecise.

### 20.4.3 Basis Swap Valuation

For a basis swap exchanging Tenor 2 versus (Tenor 1 + Spread $e$), the par condition is:

$$\sum_{j} F_j^{(2)} \tau_j P_d(0,t_j) = \sum_{i} (F_i^{(1)} + e) \tau_i P_d(0,t_i)$$

Solving for the spread $e$:

$$\boxed{e = \frac{\text{PV}(\text{Leg 2 Float}) - \text{PV}(\text{Leg 1 Float})}{\text{PV01}(\text{Leg 1})}}$$

The PV01 of Leg 1 (often called the spread-leg annuity) is $\sum_i \tau_i P_d(0,t_i)$.

> **Worked Example 20.3: Calculating the Basis Spread**
>
> **Example title:** Par spread, Spread01, and basis DV01
>
> **Context**
> - Price a 1Y basis swap exchanging 6M float vs (3M float + spread) and interpret the spread sensitivity.
>
> **Timeline (dates are illustrative)**
> - Trade date: 2026-02-15
> - Effective date: 2026-02-17
> - 3M payment dates: 2026-05-17, 2026-08-17, 2026-11-17, 2027-02-17
> - 6M payment dates: 2026-08-17, 2027-02-17
>
> **Inputs**
> - Notional: \(N=\$100{,}000{,}000\)
> - PV(6M float leg): \(0.0256\times N\)
> - PV(3M float leg): \(0.0247\times N\)
> - Spread-leg annuity (3M schedule): \(A=\sum_i \tau_i P_d(0,t_i)=0.9876\)
>
> **Outputs**
> - Par spread \(e\) (added to the 3M leg, per our notation)
> - Spread01 (PV change for a +1bp change in \(e\))
> - Basis DV01 using this book’s convention (spread down 1bp)
>
> **Step-by-step**
> 1. Par spread:
>    $$e=\frac{0.0256-0.0247}{0.9876}\approx 0.000911\approx 9.1\text{ bp}.$$
> 2. Spread01:
>    $$Spread01=N\times A\times 10^{-4}\approx 100{,}000{,}000\times 0.9876\times 10^{-4}=\$9{,}876\text{ per bp}.$$
> 3. Basis DV01 (spread down 1bp):
>    - If you **receive** the spread, \(DV01_{basis}=PV(e-1\text{bp})-PV(e)=-Spread01\).
>    - If you **pay** the spread, \(DV01_{basis}=+Spread01\).
>
> **P&L / Risk interpretation**
> - A +1bp widening of the quoted spread changes PV by approximately \(+Spread01\) for a receiver of the spread (and \(-Spread01\) for a payer).
>
> **Sanity checks**
> - Units: \(e\) is a rate, \(A\) is unitless, \(Spread01\) is currency per bp.
> - Magnitudes: \(9.1\text{ bp}\times \$9{,}876/\text{bp}\approx \$90\text{k}\), matching \(N\times(0.0256-0.0247)\).

---

## 20.5 Bootstrapping a Secondary Tenor Curve

### 20.5.1 The Hierarchical Construction Algorithm

In practice, curves are built sequentially in order of liquidity. A common hierarchy is:
discount curve (OIS) → base projection curve (most liquid tenor) → secondary projection curves inferred from basis swaps versus the base tenor.

**Step 1: Build $P_d$ (Discount Curve)**

Use OIS swaps to construct the discount curve. This reflects the cost of funding for collateralized derivatives.

**Step 2: Build $P^{(3M)}$ (Base Projection Curve)**

Use the most liquid fixed–float swaps (often 3M tenor) along with FRAs (or other money-market instruments) that reference that tenor.

**Step 3: Build $P^{(6M)}$ (Secondary Projection Curve)**

Use **basis swaps** (3M vs 6M) to determine the spread of the 6M curve relative to the 3M curve. One way to represent this is with a spread function:

$$P^{(2)}(0,t)=P^{(1)}(0,t)\exp\left(-\int_{0}^{t} \eta^{1,2}(s)\,ds\right)$$

for a piecewise-constant spread function $\eta^{1,2}(\cdot)$.

### 20.5.2 The Bootstrapping Algorithm

To bootstrap the 6M curve point by point:

1. Take the already-calibrated 3M curve and discount curve as given.

2. For each basis swap maturity $T$:
   a. Compute the PV of the 3M+Spread leg (all inputs are known from Step 2).
   b. Set the 6M leg PV equal to this value.
   c. Solve for the unknown 6M forward(s) that satisfy the equality.
   d. Convert the forward to a pseudo-discount factor:
   $$P^{(6M)}(0, T_i) = \frac{P^{(6M)}(0, T_{i-1})}{1 + \tau_i^{(6M)} F_i^{(6M)}}$$

3. Repeat for successively longer maturities.

> **Worked Example 20.4: Bootstrapping a 6M Pseudo-Discount Factor**
>
> **Given (from prior calibration):**
> - $P_d(0, 0.5) = 0.9900$ (OIS discount curve)
> - $P^{(3M)}(0, 0.25) = 0.9945$, $P^{(3M)}(0, 0.50) = 0.9886$
> - 1-year 3M/6M basis swap: +8 bps on the 3M leg (spread added to the 3M leg, per quote definition)
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

Sequential “curve group” construction also makes risk easier to interpret because each instrument set is meant to explain a specific economic object:

- Perturbations to OIS instruments define **discounting risk**.
- Perturbations to 3M swaps define **overall rate level risk**.
- Perturbations to basis spreads define **basis risk**.

---

## 20.6 Risk Decomposition in Multi-Curve

### 20.6.1 Three Distinct Risk Dimensions

In a single-curve world, you had one sensitivity: interest rate risk (DV01). In the multi-curve world, any position has three orthogonal sensitivities:

**Conventions for this chapter (bump objects, units, sign).**
- Bump size: \(1\text{bp} = 10^{-4}\) as an absolute change in the relevant quoted rate/spread.
- Sign convention: for any stated bump object \(x\), define
  $$DV01_x := PV(x-1\text{bp}) - PV(x).$$
  A positive \(DV01_x\) means PV increases when \(x\) is bumped down (and decreases when \(x\) is bumped up).
- Bump objects (what is actually bumped):
  - **Discount DV01:** bump the OIS par swap quotes used to build the discount curve \(P_d\) (rebootstrap consistently).
  - **Projection DV01 (tenor \(k\)):** bump the par FRA/IRS quotes used to build the projection curve \(P^{(k)}\).
  - **Basis DV01 (tenor pair \(1,2\)):** bump the quoted basis swap spreads \(e^{1,2}(T)\) for the relevant maturities.
- Units: currency per 1bp for the trade’s notional (you can rescale to a per-\$1mm notional basis if desired).
- Spread01 (spread-leg PV01 magnitude): if a basis swap adds spread \(e\) on a leg with annuity \(A=\sum_i \tau_i P_d(0,t_i)\), then
  $$Spread01 := N \cdot A \cdot 10^{-4}\quad (\text{currency per bp}),$$
  and the PV change for a \(+1\text{bp}\) widening of \(e\) is approximately \(+Spread01\) if you **receive** the spread and \(-Spread01\) if you **pay** it. The basis DV01 above uses the \(-1\text{bp}\) bump.

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
> - Loan (receive 6M): Coupon increases by 20 bps. For the next 6M period (\(\tau\approx 0.5\)), the incremental coupon is approximately
>   $$\Delta CF \approx N \cdot 20\text{bp}\cdot \tau = 100{,}000{,}000 \cdot 0.0020 \cdot 0.5 \approx \$100{,}000.$$
> - Swap (pay 3M): 3M rates unchanged—no offset.
> - **Net:** Unhedged P&L swing driven by basis (the magnitude scales with notional, accrual, and how long the exposure runs).
>
> **Scenario C: Reverse Position.** If the trader were *paying* 6M on a liability (instead of receiving), a basis widening would increase her cost with no hedge offset—potentially a large unexpected loss.

This example illustrates why **basis swaps** are essential hedging instruments. They allow you to isolate and manage the risk of the spread itself, independent of the overall level of rates.

### 20.6.3 Practical Risk Reporting

A modern risk system should decompose interest rate risk into at least:

1. **OIS DV01**: Sensitivity to parallel shift of the discount curve
2. **Projection DV01**: Sensitivity to parallel shift of each tenor curve
3. **Basis DV01**: Sensitivity to change in each basis spread

For more granular analysis, key-rate exposures (see Chapter 14) should be computed for each curve separately.

### 20.6.4 Computing Multi-Curve Sensitivities: Jacobian Mapping

The Jacobian method serves to decouple risk calculations from curve construction. Curve construction naturally produces sensitivities to internal curve nodes (e.g., zero-rate points or discount factors). But hedging is done with **market quotes** (par swap rates, FRA rates, basis spreads). A Jacobian mapping relates *node moves* to *quote moves* and can be used to (i) translate node deltas into **par-instrument** deltas and (ii) speed up “what-if” risk by avoiding a full curve rebuild for every small quote change.

**The Core Insight:**

When we bump a curve, we first get sensitivities to curve nodes. But desks hedge with tradable instruments and quotes. The Jacobian maps between these two representations.

**Mathematical Framework:**

Let $\mathbf{z}$ be a vector of curve nodes (zero rates or discount factors) and $\mathbf{r}$ be a vector of market instrument rates (par swap rates, FRA rates, basis spreads). The Jacobian matrix $\mathbf{J}$ relates them:

$$\mathbf{J} = \frac{\partial \mathbf{r}}{\partial \mathbf{z}}$$

For a portfolio with value $V$, the sensitivity to curve nodes is:

$$\frac{\partial V}{\partial \mathbf{z}} = \text{(curve DV01 vector)}$$

But we want sensitivity to market instruments:

$$\frac{\partial V}{\partial \mathbf{r}} = \frac{\partial V}{\partial \mathbf{z}} \cdot \mathbf{J}^{-1} = \text{(par-point DV01 vector)}$$

**Practical Implementation:**

In a multi-curve setup, the same idea is used to convert discount/projection node deltas into “par-point” deltas for a hedging set (OIS swaps, tenor swaps, tenor basis swaps). For example:

$$\begin{pmatrix} \partial V / \partial r_{\text{OIS}} \\ \partial V / \partial r_{\text{3M Swap}} \\ \partial V / \partial r_{\text{Basis}} \end{pmatrix} = \mathbf{J}^{-1} \begin{pmatrix} \partial V / \partial z_{\text{discount}} \\ \partial V / \partial z_{\text{3M proj}} \\ \partial V / \partial z_{\text{6M proj}} \end{pmatrix}$$

> **Desk Reality:** “Curve-node deltas” (e.g., a delta to an interpolated 3.5Y zero rate) are hard to hedge directly. Desks usually want deltas in the units of liquid par instruments (“sell \$X DV01 of the 3Y swap, buy \$Y DV01 of the 5Y swap”). Jacobian-based par-point deltas are one way to produce that mapping. If the Jacobian is ill-conditioned, the implied hedge ratios can be unstable—treat this as a warning about the calibration instrument set or interpolation choices.

> **Worked Example 20.6: Jacobian-Based Risk Attribution**
>
> Consider a portfolio with the following raw curve sensitivities:
>
> | Curve Point | Sensitivity ($) |
> |-------------|-----------------|
> | 2Y OIS zero | +15,000 |
> | 2Y 3M proj zero | -8,000 |
> | 2Y 6M proj zero | -12,000 |
>
> After Jacobian transformation to par-point deltas:
>
> (Here “par-point DV01” uses this chapter’s convention: PV for a 1bp **down** move in the listed hedge quote, holding the rest of the curve construction logic consistent.)
>
> | Hedge Instrument | Par-Point DV01 ($) |
> |------------------|-------------------|
> | 2Y OIS swap | +14,200 |
> | 2Y 3M vanilla swap | -7,500 |
> | 2Y 3M/6M basis swap | -4,700 |
>
> **Interpretation:** The hedge is approximately +14k OIS swaps, -7.5k 3M swaps, and -4.7k 3M/6M basis swaps at the 2Y point. The negative basis DV01 means the portfolio loses money when the 3M/6M basis **tightens** (the quoted spread moves down).

---

## 20.7 Practical Notes

### 20.7.1 Turn-of-Year (“Turn”) and Curve Overlays

Some short-dated forwards and basis quotes exhibit seasonal distortions around reporting dates, especially year-end (and sometimes quarter-end). A well-known example is the turn-of-year effect: short-dated loan premia can spike for accrual periods spanning the last business day of the year and the first business day of the next year.

A common modeling approach is to treat these as an **overlay** on top of an otherwise smooth curve. One common way of incorporating TOY-type effects is to exogenously specify an overlay curve \(\varepsilon_{f}(t)\) on the instantaneous forward curve. For example, if you work with instantaneous forwards \(f(t)\), you can represent
$$f(t) = \varepsilon_{f}(t) + f^\*(t),$$
where \(f^\*(t)\) is a baseline smooth curve and \(\varepsilon_{f}(t)\) is an exogenously specified, localized “turn” overlay.

**Where it comes from (high level):**
1. **Year-end balance sheet and regulatory reporting constraints** that change the supply/demand for term funding into year-end.
2. **Quarter-end reporting effects** that can create smaller seasonal patterns.

**Practical implications for curve building:**
- Build the payment schedule first: whether a coupon period spans a turn date is instrument-specific.
- Implement turn effects as a localized overlay (or as special “turn instruments”) so the distortion does not contaminate the whole curve.
- Expect trades spanning year-end to carry PV/risk that can look “odd” relative to a smooth key-rate view if the curve includes turn adjustments.

### 20.7.2 Convention Ambiguities

**Which leg gets the spread?** This varies by market, dealer, and data source. A label like “3M vs 6M” is not a complete contract: you must know which leg carries \(+e\) and whether you are paying or receiving that spread.

**Best practice:** Before executing or hedging a basis swap, explicitly confirm:
1. Which leg carries the spread \(+e\) (and the receive/pay direction)
2. Day count, holiday calendar, and fixing/payment conventions for each leg
3. Schedule details (stubs, frequency mismatch, effective date/spot lag)
4. Any compounding or averaging conventions if the floating leg is not a simple IBOR-style setting

> **Note:** Liquidity and standardization are highest in the most actively traded benchmarks; for newer RFR-based tenor basis markets, conventions and liquidity can vary by currency, maturity, and venue. Verify the exact contract definition before assuming a sign or a hedge mapping.

### 20.7.3 Implementation Pitfalls

**Circular Dependencies.** Ensure your curve construction code respects the hierarchy: OIS → Base Projection → Spread curves. Do not let the 6M curve feed back into the 3M curve construction. If using a global optimizer, this can introduce subtle cross-dependencies that are hard to debug.

**Interpolation Choices.** When interpolating tenor curves, remember that $P^{(k)}$ is not a real discount factor—it is a mathematical construct. Interpolating directly on the "basis spread" $\eta(t)$ often yields smoother results than interpolating the raw pseudo-discount factors. This avoids artifacts where the basis curve develops unexpected wiggles.

---

## Summary

1. **Quote → cashflows → PV.** A tenor basis quote becomes a spread \(e^{1,2}(T)\) on a specific floating leg and schedule; PV depends on both the discount curve and the projection curves.
2. **Curve roles.** Use \(P_d\) to discount cashflows; use tenor-specific \(P^{(k)}\) only to generate forwards/coupons for tenor \(k\).
3. **Par basis spread.** A basis swap is a floating–floating exchange; the par spread is chosen so that \(PV(0)=0\) at inception under the agreed curve set.
4. **Spread01 and basis DV01.** With spread-leg annuity \(A=\sum_i \tau_i P_d(0,t_i)\), a 1bp move in \(e\) scales PV by \(N\cdot A \cdot 10^{-4}\); the sign depends on whether you receive or pay the spread. This chapter’s \(DV01\) convention uses the \(-1\text{bp}\) bump.
5. **Curve construction.** A common workflow is hierarchical: build discounting first, then a base projection curve, then secondary projection curves from basis swap quotes.
6. **Risk decomposition.** Being “DV01-flat” to a single curve does not imply you are basis-flat; you need discount DV01, projection DV01, and basis DV01.
7. **Hedgeable risk reports.** Jacobian mapping can convert curve-node deltas into par-instrument deltas so risk is expressed in units the desk can actually trade.
8. **Turns/overlays.** Short-dated curves can include localized overlays around reporting dates; confirm curve conventions and schedules before using short-dated basis for pricing or hedging.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---|---|---|
| **Tenor** | Accrual length \(\tau\) associated with a floating-rate setting (e.g., 1M, 3M, 6M). | Different tenors can embed different funding/liquidity exposures. |
| **Discount curve \(P_d\)** | Curve used to discount cashflows to PV (often OIS for collateralized trades). | Discounting is distinct from forecasting floating coupons. |
| **Projection curve \(P^{(k)}\)** | Tenor-\(k\) curve used to generate forwards/coupons for that tenor. | Ensures floating coupons are consistent with tenor-\(k\) quotes. |
| **Tenor basis (curve)** | The set of spreads \(e^{1,2}(T)\) that prices exchanging tenor-1 exposure for tenor-2 exposure across maturities \(T\). | Basis is an observable curve (from basis swaps), not a single-curve identity. |
| **Basis swap** | Floating–floating swap exchanging two tenor legs; one leg often carries a fixed spread. | Primary instrument for isolating/hedging tenor mismatch. |
| **Par basis spread \(e^{1,2}(T)\)** | Spread chosen so the basis swap has \(PV(0)=0\) at inception (given curves and conventions). | Connects a quote to a PV equation and hedgeable risk factor. |
| **Spread-leg annuity \(A\)** | \(A=\sum_i \tau_i P_d(0,t_i)\) on the leg where the spread is applied. | Scales how much PV moves when the quote moves. |
| **Spread01** | \(Spread01 = N\cdot A \cdot 10^{-4}\) (currency per bp). | “PV move for a +1bp widening of the quoted spread.” |
| **Discount / Projection / Basis DV01** | DV01 w.r.t. (i) discount quotes, (ii) tenor projection quotes, (iii) basis spread quotes. | DV01-flat to one curve does not imply basis-flat. |
| **Turn / overlay** | Localized adjustment for accrual periods spanning reporting dates (e.g., year-end). | Short-dated PV/risk can be schedule-dependent. |
| **Jacobian mapping** | Map between curve-node deltas and par-instrument deltas. | Produces hedgeable risk in units the desk trades. |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| \(P_d(0,T)\) | discount factor on the discount curve | unitless; used to discount cashflows |
| \(P^{(k)}(0,T)\) | pseudo-discount factor for tenor-\(k\) projection | unitless; used only to generate forwards |
| \(L^{(k)}(0;T_1,T_2)\) | tenor-\(k\) forward rate for \([T_1,T_2]\) | per year; coupon \(\approx N\cdot L\cdot \tau\) |
| \(\tau_i^{(k)}\) | accrual year fraction for period \(i\) on tenor-\(k\) leg | years; index day count |
| \(e^{1,2}(T)\) | par basis spread for tenor 1 vs tenor 2 to maturity \(T\) | rate (or bp); quote defines spread leg + pay/receive |
| \(A\) | spread-leg annuity \(A=\sum_i \tau_i P_d(0,t_i)\) | unitless |
| \(N\) | notional | currency; \(N>0\) with cashflow sign handled separately |
| \(PV\) | present value | currency; positive = receive |
| \(Spread01\) | PV change per +1bp widening of \(e\) (magnitude) | currency per bp; \(Spread01=N\cdot A\cdot 10^{-4}\) |
| \(DV01_x\) | PV sensitivity to bump object \(x\) | currency per bp; \(DV01_x := PV(x-1bp)-PV(x)\) |
| \(\eta^{1,2}(t)\) | instantaneous spread/overlay between two projection curves | per year; smooth parameterization |
| \(f(t)\) | instantaneous forward rate (generic) | per year; used in turn/overlay discussion |
| \(\varepsilon_{f}(t)\) | turn/overlay component of \(f(t)\) | per year; localized/exogenous adjustment |
| \(f^\*(t)\) | baseline smooth component of \(f(t)\) | per year |
| \(\mathbf{J}\) | Jacobian mapping nodes \(\mathbf{z}\) → market quotes \(\mathbf{r}\) | unitless matrix; used for par-instrument deltas |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a tenor? | The accrual length \(\tau\) attached to a floating-rate setting (e.g., 3M means \(\tau\approx 0.25\)). |
| 2 | What is tenor basis in a multi-curve framework? | The set of spreads that prices exchanging exposure between two tenors across maturities; observed from basis swap quotes. |
| 3 | What is a basis swap? | A floating–floating swap exchanging two tenor legs; one leg often carries a fixed spread. |
| 4 | Which curve discounts cashflows in a collateralized swap? | The discount curve \(P_d\) (often OIS). |
| 5 | Which curve generates the floating coupons? | The relevant projection curve \(P^{(k)}\) for that tenor/index. |
| 6 | What does “par basis spread” mean? | The spread \(e\) that makes \(PV(0)=0\) for the basis swap under the agreed curves and conventions. |
| 7 | What is \(Spread01\) and its units? | \(Spread01 = N\cdot A\cdot 10^{-4}\), currency per bp; it scales PV for a 1bp move in the quoted spread. |
| 8 | How is \(DV01\) defined in this book? | \(DV01_x := PV(x-1bp)-PV(x)\) for the stated bump object \(x\) (PV for a 1bp down move). |
| 9 | What is the bump object for discount DV01? | The OIS par instrument quotes used to build \(P_d\) (bumped and re-bootstrapped consistently). |
| 10 | What is the bump object for projection DV01? | The par FRA/IRS quotes defining the tenor-\(k\) projection curve \(P^{(k)}\). |
| 11 | What is the bump object for basis DV01? | The basis swap spread quotes \(e^{1,2}(T)\) for the relevant maturity points. |
| 12 | If you receive the spread, what happens to PV when \(e\) widens by 1bp? | PV increases by approximately \(+Spread01\) (first-order). |
| 13 | Why can a DV01-neutral hedge leave basis risk? | DV01 to one curve can be hedged while the relative spread between projection curves still moves. |
| 14 | What must you confirm from a vendor quote like “3M/6M basis = +8bp”? | Which leg carries \(+e\), pay/receive direction, day count/calendars, schedule/stubs, and fixing/payment conventions. |
| 15 | What is a “turn overlay”? | A localized adjustment to forwards/basis for accrual periods spanning reporting dates (e.g., year-end). |
| 16 | Why build curves hierarchically (discount → base projection → secondary projection)? | To avoid circular dependencies and keep sensitivities local and interpretable. |
| 17 | What does Jacobian mapping do in risk? | Converts curve-node deltas into par-instrument deltas (hedgeable DV01s). |
| 18 | What does an ill-conditioned Jacobian often indicate? | Unstable hedge ratios, often due to illiquid instruments or overly flexible interpolation. |
| 19 | Why might a floating-rate note deviate from par in multi-curve? | Coupons are projected on one curve but discounted on another; the “telescoping to par” identity breaks. |
| 20 | What is a pseudo-discount factor? | A mathematical construct that reproduces forwards for a tenor; it is not used to discount cashflows. |

---

## Mini Problem Set
1. (Compute) Given \(P^{(3M)}(0,0)=1\), \(P^{(3M)}(0,0.25)=0.9940\), and \(\tau=0.25\), compute the 3M forward rate for \([0,0.25]\).
2. (Compute) A 6M vs (3M + spread) basis swap has \(PV(\text{6M leg})=\$1{,}020{,}000\) and \(PV(\text{3M leg})=\$1{,}000{,}000\). The spread-leg \(Spread01\) is \(\$5{,}000\) per bp. If the spread is added to the 3M leg, compute the par spread \(e\) (in bp).
3. (Compute) For \(N=\$100{,}000{,}000\) and spread-leg annuity \(A=0.9876\), compute \(Spread01\).
4. (Compute) Use this chapter’s DV01 convention \(DV01_x = PV(x-1bp)-PV(x)\). A portfolio has OIS DV01 \(=+\$100{,}000\), 3M projection DV01 \(=-\$80{,}000\), and 3M/6M basis DV01 \(=+\$20{,}000\). If OIS \(+5\)bp, 3M projection \(+5\)bp, and basis \(+3\)bp, estimate \(\Delta PV\).
5. (Concept) Explain why a floating-rate note can deviate from par in a multi-curve framework even on a reset date.
6. (Desk) You see a market data line “3M/6M basis = +8bp”. List at least four contract details you must confirm before trading or hedging it.
7. (Implementation) You build curves with a global optimizer and observe that bumping a 5Y basis quote also moves the discount curve. What dependency did you introduce?
8. (Concept) Why can it be smoother to interpolate/parameterize an instantaneous spread \(\eta(t)\) than to interpolate pseudo-discount factors \(P^{(k)}(0,t)\) directly?

### Solution Sketches (Selected)
1. \(F = \frac{1}{0.25}\left(\frac{1}{0.9940}-1\right)\approx \frac{0.006036}{0.25}=0.02414 \approx 2.414\%\).
2. \(e = \frac{1{,}020{,}000-1{,}000{,}000}{5{,}000}=4\) bp.
3. \(Spread01 = N\cdot A\cdot 10^{-4}\approx 100{,}000{,}000\cdot 0.9876 \cdot 10^{-4}=\$9{,}876\) per bp.
4. Since \(DV01\) is PV for a 1bp **down** move, for an **up** move \(\Delta PV \approx -DV01 \times \Delta bp\). So
   \[
   \Delta PV \approx -(100{,}000\cdot 5 + (-80{,}000)\cdot 5 + 20{,}000\cdot 3) = -(500{,}000-400{,}000+60{,}000)=-\$160{,}000.
   \]
5. Coupons are projected from a projection curve while cashflows are discounted with \(P_d\). The classic “float = par” telescoping uses one curve for both; with two curves the identity generally fails.
6. Confirm: which leg carries \(+e\) and your pay/receive direction; day count and calendar for each leg; schedule/stubs/effective date; fixing/payment conventions and any lookback/payment delay; and any compounding/averaging conventions on the floating legs.

---

## References

- Andersen & Piterbarg, *Interest Rate Modeling* (Vol 1) (multi-index curves; tenor basis swaps; projection vs discounting separation; basis curve construction and risk decomposition).
- Hull, *Options, Futures, and Other Derivatives* (“Multiple Yield Curves”; “Variations on the Standard Interest Rate Swap”).
- Hull, *Risk Management and Financial Institutions* (“The OIS Rate”).
- Neftci, *Principles of Financial Engineering* (“Basis swaps”).
- Oosterlee et al., *Mathematical Modeling and Computation in Finance* (multi-curve valuation; tenor curve forwards).
- Crépey, *Counterparty Risk and Funding* (multi-curve interest-rate modeling motivation).
