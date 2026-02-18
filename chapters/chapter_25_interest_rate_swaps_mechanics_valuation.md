# Chapter 25: Interest Rate Swaps — Mechanics and Valuation

---

## Introduction

A swap is not a bet on rates—it is an exchange of cashflow streams with (approximately) zero value at inception once the fixed rate is set.

Swaps show up everywhere in rates markets because they let you **transform** cashflows (fixed ↔ floating) without refinancing the underlying debt and they provide a clean “swap-curve” yardstick for pricing and hedging.

The mechanics are simple; the pitfalls are not. A vanilla interest rate swap is specified in *rate language* (fixed rate, floating index, day count, reset/payment dates), but it is valued in *cashflow language*. And in modern markets you usually need **two curve objects**:
- a **projection** curve to turn the floating index into expected coupons, and
- a **discounting** curve to PV every cashflow.

This chapter builds the quote-to-PV pipeline:
1. Mechanics and timeline (trade/effective/reset/payment)
2. Cashflow construction (day counts, accrual factors, and net settlement)
3. Valuation (fixed leg, floating leg, par swap rate, and the annuity/PVBP object)
4. Single-curve vs multi-curve (where the telescoping shortcut breaks)
5. A worked example with unit/sign checks and a short risk “one page”

## Learning Objectives
- After this chapter, you can describe a vanilla fixed-for-floating IRS and its timeline.
- You can translate a swap quote into fixed and floating cashflows (with day count and compounding stated).
- You can compute a par swap rate from discount factors and explain the annuity/PVBP object.
- You can explain projection vs discounting curves and why multi-curve valuation does not telescope.
- You can state a DV01/PVBP definition with explicit bump object, units, and sign (and know what is deferred to Chapter 26).

Prerequisites: [Chapter 1 — Market Quoting, Calendars, and Cashflow Plumbing](chapter_01_market_quoting_calendars_cashflow_plumbing.md), [Chapter 2 — Time Value, Discount Factors, and Replication](chapter_02_time_value_discount_factors_replication.md), [Chapter 3 — Zero/Forward/Par Rates (The Triangle)](chapter_03_zero_forward_par_rates_triangle.md), [Chapter 17 — Curve Construction (Bootstrapping and Interpolation)](chapter_17_curve_construction_bootstrapping_interpolation.md), [Chapter 19 — Projection Curves (LIBOR/SOFR) and Multi-Curve](chapter_19_projection_curves_libor_sofr_multi_curve.md)

Follow-on: [Chapter 26 — Swap PV01, DV01, and Hedging with Swaps](chapter_26_swap_pv01_dv01_hedging.md), [Chapter 27 — Swap Spreads, Asset Swaps, and Swap-Curve Relative Value](chapter_27_swap_spreads_asset_swaps_swap_curve_rv.md), [Chapter 28 — Basis Trades in Rates](chapter_28_basis_trades.md), [Chapter 33 — Collateral Discounting and OIS](chapter_33_collateral_discounting_ois.md)

---

## 25.1 What Is an Interest Rate Swap?

### 25.1.1 The Basic Contract

An **interest rate swap (IRS)** is a bilateral contract to exchange interest payments on a notional principal: one leg pays a fixed rate and the other leg pays a floating reference rate, on specified schedules, for an agreed term. The notional is a calculation amount and is typically **not** exchanged.

The key elements are:

- **Notional principal** ($N$): The amount on which interest is calculated. Unlike a loan, this principal is never exchanged—it is "notional" in the sense that it exists only for calculation purposes.
- **Fixed leg**: One party pays a fixed rate $c$ applied to the notional.
- **Floating leg**: The other party pays a floating reference index (e.g., a term rate or an overnight rate compounded over the accrual period) applied to the same notional.
- **Payment frequency**: Often different for the two legs—quarterly floating and semiannual fixed is common in USD markets.
- **Maturity**: The swap's lifetime, typically ranging from 1 to 30 years.

### 25.1.2 Payer vs. Receiver

The terminology reflects which leg you *pay*:

$$\boxed{\text{Payer swap: Pay fixed, receive floating}}$$
$$\boxed{\text{Receiver swap: Receive fixed, pay floating}}$$

All else equal, a payer swap benefits when rates rise (floating receipts rise while fixed payments are unchanged), and a receiver swap benefits when rates fall.

> **Desk Reality:** Traders say “paying” / “receiving” (meaning paying/receiving fixed).
> **Common break:** People translate “pay fixed” into “long the swap” and flip the PV/DV01 sign.
> **What to check:** Write $V_{\text{payer}}=PV_{\text{float}}-PV_{\text{fixed}}$ and apply the book DV01 sign convention before you size a hedge.

### 25.1.3 Why Notional Is Not Exchanged

Even though the notional is not exchanged, it is often helpful for valuation to **pretend** that both parties exchange notional at maturity. Those notional exchanges cancel out, so you have not changed the economics—but you gain a powerful replication:
- the fixed leg behaves like a fixed-rate bond, and
- the floating leg behaves like a floating-rate note.

This perspective is one reason swaps can be priced with the same “discount cashflows” machinery used for bonds.

### 25.1.4 The Loan Analogy

> **Analogy: A Swap as a Funded Bond Position**
>
> A **receiver swap** (receive fixed, pay floating) is economically equivalent to:
> - Buying a fixed-rate bond, AND
> - Funding that purchase with a floating-rate loan
>
> Think about it: if you buy a 5-year bond paying 4% fixed and borrow the purchase price at SOFR + spread, your net cashflow is:
> - Receive: 4% fixed coupon
> - Pay: SOFR + spread on the loan
>
> This is exactly a receiver swap! The notional on both legs (bond face value and loan amount) cancels at maturity, just like in a swap.
>
> A **payer swap** (pay fixed, receive floating) is the opposite:
> - Shorting a fixed-rate bond, AND
> - Lending the proceeds at a floating rate
>
> This analogy helps explain why swaps are used for hedging bond portfolios: entering a payer swap is economically similar to selling the bond without actually selling it.

---

## 25.2 Cashflow Structure

### 25.2.1 Fixed Leg Cashflows

The fixed leg pays predetermined amounts at each payment date. For payment dates $\{T_1^{\text{fix}}, T_2^{\text{fix}}, \ldots, T_{n_f}^{\text{fix}}\}$, the cashflow at $T_j^{\text{fix}}$ is:

$$\boxed{CF_j^{\text{fix}} = N \cdot c \cdot \tau_j^{\text{fix}}}$$

where $\tau_j^{\text{fix}}$ is the accrual factor (year fraction) for that period, computed according to the applicable day count convention.

**Day count conventions matter.** The fixed rate is always quoted with a day count convention (commonly 30/360 or ACT/365). The day count determines $\tau_j^{\text{fix}}$ and therefore the cash amount exchanged.

### 25.2.2 Floating Leg Cashflows

The floating leg pays amounts that depend on the reference rate observed during each period. For a standard "set in advance, pay in arrears" structure, the rate for period $[T_i^{\text{flt}}, T_{i+1}^{\text{flt}}]$ is observed (fixed) at $T_i^{\text{flt}}$ and paid at $T_{i+1}^{\text{flt}}$:

$$\boxed{CF_{i+1}^{\text{flt}} = N \cdot L_k(T_i; T_i, T_{i+1}) \cdot \tau_i^{\text{flt}}}$$

where $L_k(T_i; T_i, T_{i+1})$ is the floating reference rate for tenor $k$ observed at time $T_i$.

**The floating day count often differs from the fixed.** Many money-market indices (including SOFR and legacy IBORs) use ACT/360. That means a “one-year” interval can have a year fraction greater than 1 under ACT/360 (e.g., $365/360$), while the fixed leg might accrue exactly 1.0 under 30/360.

### 25.2.3 Example: A 2-Year USD Swap Schedule

Consider a USD100 million swap with:
- Fixed leg: 3.00% semiannual, 30/360
- Floating leg: quarterly money-market index, ACT/360 (set in advance, paid in arrears)

The fixed leg pays (every six months):
USD100,000,000 × 3.00% × 0.5 = USD1,500,000.

The floating leg pays (each quarter):
USD100,000,000 × $R$ × $(n/360)$, where $R$ is the annualized floating rate and $n$ is the actual number of days in the accrual period.

Example: if $R=2.20\%$ and the quarter has $n=92$ days, the floating coupon is:
USD100,000,000 × 0.022 × (92/360) = USD562,222.

Cash settlement is typically **netted** on dates where both legs pay: only the difference between the fixed and floating amounts is exchanged. On dates where only one leg pays, that leg’s payment is exchanged.

---

## 25.3 SOFR Swaps: Compounding and Payment Mechanics

SOFR-based swap legs differ from term-index swap legs mainly in how the floating coupon is set. In a term-index leg, the coupon rate is fixed at the start of the accrual period (set in advance, paid in arrears). In a compounded SOFR leg, the coupon rate depends on daily overnight fixings through the accrual period and is finalized near period end.

### 25.3.1 Why SOFR Is Different

SOFR is an overnight rate. To calculate a floating coupon over a multi-day accrual period, you compound the daily overnight rates over that period and then annualize the result on the relevant day-count basis.

For a period from day 1 to day $n$, with daily overnight rates $r_1, r_2, \ldots, r_n$ and corresponding day counts $d_1, d_2, \ldots, d_n$, one common annualized compounding convention is:

$$\boxed{R_{\text{compound}} = \frac{360}{D} \left[\prod_{i=1}^{n} \left(1 + r_i \cdot \frac{d_i}{360}\right) - 1\right]}$$

where $D$ is the total number of days in the period. On most days $d_i=1$, but weekends and holidays mean an overnight fixing can apply to multiple days.

### 25.3.2 Operational Timing: Observation Schedule and Payment Delay

For compounded-in-arrears coupons, the rate is not fully known until the accrual period ends. To compute the coupon (and to settle cash), you need the confirmation to specify the schedule: trade/effective/termination dates, calendars and business day convention, day count, payment dates, and (for compounded legs) the observation/compounding dates used in the calculation.

> **Desk Reality:** For RFR legs, ops needs the final coupon amount before cash settlement.
> **Common break:** Assuming there is a single “standard” observation rule when the confirmation uses a different one.
> **What to check:** Read the confirmation fields for observation dates and payment date, then reproduce the compounding on a 2–3 day toy period.

### 25.3.3 Term SOFR vs. Overnight SOFR Compounding

SOFR-linked coupons can be constructed in different ways. Two common structures are:

**Overnight SOFR compounding (OIS-style):** Daily SOFR rates are compounded in arrears. The coupon rate is known only near period end.

**Term-style reference rates:** Some products reference forward-looking term rates derived from overnight-rate futures. The desk-relevant distinction is whether the floating coupon is known at period start (term-style) or only near period end (compounded in arrears).

### 25.3.4 Standard USD SOFR Swap Conventions

A common USD “fixed vs SOFR” swap structure looks similar to the legacy USD fixed-vs-term swap: fixed coupons are paid periodically (often semiannual) and floating coupons are paid periodically (often quarterly), but the floating coupon is computed from **compounded SOFR in arrears**.

| Element | Fixed Leg (common) | Floating Leg (common) |
|---------|---------------------|------------------------|
| Day count | 30/360 | ACT/360 |
| Payment frequency | Semiannual | Quarterly |
| Rate determination | Fixed at inception | Compounded SOFR in arrears over the accrual period |
| Operational convention | — | Payment delay and/or lookback/lockout (per documentation) |
| Business day | Modified following | Modified following |

*Note:* There are other SOFR swap conventions (including OIS-style annual payment structures). Always confirm the product definition you are trading/valuing.

### 25.3.5 Example: Computing a SOFR Floating Payment

**Problem:** Calculate the floating payment for a USD50 million notional swap over a 91-day quarter, given daily SOFR rates.

**Given:** For simplicity, assume constant SOFR of 4.25% for all 91 days (in practice, rates vary daily).

**Step 1: Compound the daily rates**

$$\prod_{i=1}^{91} \left(1 + 0.0425 \cdot \frac{1}{360}\right) = \left(1 + \frac{0.0425}{360}\right)^{91} = 1.010878$$

**Step 2: Annualize to get the period rate**

$$R = \frac{360}{91} \times (1.010878 - 1) = 0.04302 = 4.302\%$$

Note that the compounded rate (4.302%) slightly exceeds the simple average (4.25%) due to compounding.

**Step 3: Calculate the payment**

$$\text{Payment} = USD50,000,000 \times 0.04302 \times \frac{91}{360} = USD543,697$$

**Sanity check:** Simple interest would give USD50M × 4.25% × (91/360) = USD537,153. The compounded amount is about USD6,500 higher—a small but meaningful difference on large notionals.

---

## 25.4 Market Quoting and Execution

### 25.4.1 How Swaps Are Quoted: Outright Rate vs. Swap Spread

Swaps are commonly quoted as an **outright fixed rate** (e.g., “5-year at 3.75%”). In relative-value discussions, traders also quote the **swap spread**: the difference between the swap fixed rate and a reference Treasury yield.

$$\boxed{\text{Swap Rate} = \text{Treasury Yield} + \text{Swap Spread}}$$

For example, if the 5-year Treasury yields 3.50% and the 5-year swap spread is 25 basis points, the swap rate is 3.75%.

> **Desk Reality: "5s Are 10.5 Out"**
>
> When a trader says "5-year swaps are 10.5 out" or "5s are plus 10.5," they mean the 5-year swap spread is 10.5 basis points over Treasuries. If the 5-year Treasury is at 4.00%, the swap rate is 4.105%.
>
> Bid/offer on swap rates might be quoted as "3.73/3.75" meaning:
> - A dealer will *pay* fixed at 3.73% (you receive fixed)
> - A dealer will *receive* fixed at 3.75% (you pay fixed)
>
> The 2bp bid-offer is typical for liquid benchmark tenors. Less liquid tenors or larger sizes may see wider spreads.

### 25.4.2 Why Swap Spreads Can Be Negative

Swap spreads can be **positive or negative**. Intuitively, they reflect a mix of forces: the funding and balance-sheet cost of holding swap exposure, the relative supply/demand for Treasuries vs swaps, and the details of collateralization/discounting conventions.

Negative swap spreads imply that the swap rate is *below* the Treasury yield—a counterintuitive result explained by:
- Treasury scarcity (high demand for Treasuries for regulatory purposes)
- Balance sheet constraints on dealers
- Supply/demand imbalances in the swap market itself

Chapter 27 explores swap spread dynamics in detail.

### 25.4.3 Standard Benchmark Tenors and IMM Dates

**Benchmark Tenors:** The most liquid swap tenors are: 2Y, 3Y, 5Y, 7Y, 10Y, 15Y, 20Y, 30Y. These form the "benchmark" swap curve.

**IMM Dates:** Many swaps trade with effective dates on "IMM dates"—the third Wednesday of March, June, September, and December. IMM-dated swaps are easier to hedge with futures and facilitate portfolio compression.

### 25.4.4 Trade Execution: SEF and Voice

In the post-crisis market structure, many standardized swaps are executed electronically and cleared through central counterparties (CCPs), while bespoke trades may remain bilateral.

**Execution Methods:**
- **Request for Quote (RFQ):** Client requests prices from multiple dealers; best price wins
- **Central Limit Order Book (CLOB):** Anonymous trading on an electronic order book
- **Voice:** Large or non-standard trades may still be negotiated by phone

> **Practitioner Note:** Interdealer trading in benchmark tenors is often electronic; client-to-dealer trades may use RFQ or voice depending on size and complexity.

### 25.4.5 CCP Clearing: How It Works

In centrally cleared markets, a **central counterparty (CCP)** stands between the two original parties. It requires margin and performs functions similar to an exchange clearing house.

**The CCP Process:**

1. **Trade Execution:** Two parties agree on a swap on a SEF or bilaterally
2. **Novation:** The CCP steps in as counterparty to both sides. The original bilateral trade becomes two trades: Party A vs. CCP, and CCP vs. Party B
3. **Margin Posting:** Both parties post initial margin (based on potential future exposure) and variation margin (daily mark-to-market)
4. **Daily Settlement:** Gains/losses are settled daily via variation margin transfers
5. **Default Management:** If a party defaults, the CCP uses margin and default fund resources to close out positions

> **Desk Reality: Margin and Funding Costs**
>
> CCP clearing changes the economics of swaps: initial margin is posted, variation margin is exchanged (often daily), and the cost of funding margin can matter for pricing and for P&L attribution.

### 25.4.6 Bilateral Clearing (Non-Cleared Swaps)

Some swaps remain uncleared—typically non-standard swaps or swaps with end-users (corporates, sovereigns). These are governed by ISDA Master Agreements with Credit Support Annexes (CSAs).

The CSA specifies:
- **Threshold:** Below this exposure, no collateral required
- **Minimum Transfer Amount:** Smallest collateral movement
- **Eligible Collateral:** Cash, government bonds, etc.
- **Collateral Rate:** Interest paid on posted collateral (typically overnight rate)

---

## 25.5 Valuation Fundamentals

### 25.5.1 The Basic Principle: Discount Cashflows

To value any derivative, we discount expected cashflows to the present:

$$V = \sum_i E[CF_i] \cdot P_d(0, T_i)$$

where $P_d(0, T)$ is the discount factor from the *discounting curve* for maturity $T$.

For the fixed leg, cashflows are deterministic once the swap is initiated:

$$\boxed{PV_{\text{fixed}}(0) = N \sum_{j=1}^{n_f} c \cdot \tau_j^{\text{fix}} \cdot P_d(0, T_j^{\text{fix}})}$$

For the floating leg, future rates are unknown. Under risk-neutral pricing, we replace future rates with *forward rates* implied by the market curve:

$$\boxed{PV_{\text{float}}(0) = N \sum_{i=0}^{n_p-1} L_k(0; T_i, T_{i+1}) \cdot \tau_i^{\text{flt}} \cdot P_d(0, T_{i+1}^{\text{flt}})}$$

where $L_k(0; T_i, T_{i+1})$ is the forward rate for the floating index.

### 25.5.2 Swap Value from the Two Perspectives

The swap has zero value at inception by construction—that is what defines the "par" swap rate. After inception, market rates move, and the swap acquires value to one party or the other:

$$\boxed{V_{\text{payer}} = PV_{\text{float}} - PV_{\text{fixed}}}$$
$$\boxed{V_{\text{recv}} = PV_{\text{fixed}} - PV_{\text{float}} = -V_{\text{payer}}}$$

**Sign interpretation:** If rates have risen since inception, forward rates exceed the fixed rate, making $PV_{\text{float}} \gt PV_{\text{fixed}}$. The payer swap has positive value—the payer locked in a below-market fixed rate and receives above-market floating payments.

### 25.5.3 Worked Example: Valuing a Swap from Leg Cashflows

Consider a swap with 1.2 years remaining, fixed rate 3% paid semiannually, and a floating leg based on a short-term index. Suppose discount factors (or equivalently zero rates) for maturities of 0.2, 0.7, and 1.2 years are available. A leg-by-leg valuation produces:

| Time (years) | Fixed CF | Floating CF | Net CF | Discount Factor | PV of Net CF |
|--------------|----------|-------------|--------|-----------------|--------------|
| 0.2 | -1.500 | +1.258 | -0.242 | 0.9944 | -0.241 |
| 0.7 | -1.500 | +1.694 | +0.194 | 0.9778 | +0.190 |
| 1.2 | -1.500 | +1.857 | +0.357 | 0.9600 | +0.343 |
| **Total** | | | | | **+0.292** |

The swap value of +USD0.292 million (per USD100 million notional) reflects that floating rates have risen above the fixed rate, benefiting the floating receiver.

> **Technique: Napkin Valuation**
>
> You don't always need a computer to know if you're winning or losing.
> *   **Rule of thumb (receiver-fixed):** $\text{PV} \approx -\text{Notional} \times (\text{New Par Swap Rate} - \text{Your Fixed Rate}) \times \text{Duration}$
> *   **Example**:
>     *   You receive fixed 3.00% on USD100M for 5 years.
>     *   Market rates rise to 3.50%.
>     *   Duration $\approx$ 4.5 years.
>     *   $\text{PV} \approx -USD100\text{M} \times (0.0350 - 0.0300) \times 4.5 = -USD2.25\text{M}$
>     *   **Sign check:** You receive 3.00%, but the market now pays 3.50%, so the position is worth less.

---

## 25.6 The Par Swap Rate

### 25.6.1 Definition and Intuition

The par swap rate is the fixed rate that makes a new swap have zero value at inception:

$$V_{\text{payer}} = PV_{\text{float}} - PV_{\text{fixed}} = 0$$

Solving for the fixed rate:

$$\boxed{c_{\text{par}} = \frac{PV_{\text{float}}/N}{\sum_{j=1}^{n_f} \tau_j^{\text{fix}} \cdot P_d(0, T_j^{\text{fix}})}}$$

The denominator is the *fixed-leg annuity*, denoted $A^{\text{fix}}(0)$:

$$\boxed{A^{\text{fix}}(0) = \sum_{j=1}^{n_f} \tau_j^{\text{fix}} \cdot P_d(0, T_j^{\text{fix}})}$$

The annuity represents the present value of receiving 1 unit of coupon (as a rate) on the fixed schedule. It is sometimes called the PVBP (present value of a basis point) when scaled appropriately.

### 25.6.2 Why Swaps Trade at Par

Most vanilla swaps are traded with **no upfront exchange**. The fixed rate is therefore set so that the trade is fair at inception:
$PV_{\text{fixed}} = PV_{\text{float}}$.

If the fixed rate were set above or below par, one party would be receiving something of value at inception and (in a frictionless world) an upfront payment would be required to restore fairness.

### 25.6.3 The Single-Curve Par Rate Formula

In the traditional single-curve framework (where the same curve provides both forward rates and discount factors), the par swap rate has a closed form:

$$\boxed{S_{k,m}(t) = \frac{P(t, T_k) - P(t, T_{k+m})}{A_{k,m}(t)}}$$

where $A_{k,m}(t) = \sum_{n=k}^{k+m-1} \tau_n P(t, T_{n+1})$ is the annuity factor.

**Interpretation:** The numerator $P(t, T_k) - P(t, T_{k+m})$ is the present value of a unit notional exchanged at the start and returned at the end. The denominator is the annuity. The ratio gives the coupon rate that equates these values.

### 25.6.4 The Telescoping Identity for Floating Legs

In single-curve pricing, the floating leg PV simplifies dramatically. Consider a spot-starting swap with floating payments at $T_1, T_2, \ldots, T_n$. The forward rate for period $i$ is:

$$L_i = \frac{1}{\tau_i}\left(\frac{P(0, T_{i-1})}{P(0, T_i)} - 1\right)$$

Each floating coupon's contribution to PV is:

$$\tau_i L_i P(0, T_i) = \left(\frac{P(0, T_{i-1})}{P(0, T_i)} - 1\right) P(0, T_i) = P(0, T_{i-1}) - P(0, T_i)$$

Summing from $i=1$ to $n$:

$$\sum_{i=1}^{n} \tau_i L_i P(0, T_i) = P(0, T_0) - P(0, T_n) = 1 - P(0, T_n)$$

This is the *telescoping identity*:

$$\boxed{\frac{PV_{\text{float}}}{N} = 1 - P(0, T_n) \quad \text{(single-curve, spot-start)}}$$

**Intuition:** A floating-rate note that resets at market rates is always worth par on reset dates. The PV of the coupons alone equals the discount from par to today's value of receiving par at maturity.

---

## 25.7 Multi-Curve Valuation: Projection vs. Discounting

### 25.7.1 Why One Curve Is Not Enough

A common valuation pipeline for a fixed-for-floating swap (often described as “assume forward rates are realized”) is:
1. **Project** the floating coupons using forward rates implied by the yield curve for the floating reference rate.
2. **Discount** those cashflows using the risk-free (OIS) curve (more precisely: the collateral discount curve implied by the CSA/clearing setup).

This is already a “multi-curve” description: one curve object to generate forwards (projection) and one curve object to PV cashflows (discounting).

### 25.7.2 The Multi-Curve Framework

Modern swap valuation uses:

1. **Projection curve** ($P_k$): Generates forward rates for the floating index
2. **Discount curve** ($P_d$): Discounts all cashflows to present value

> **Analogy: The Projector and The Auditor**
>
> In the old days (Single Curve), one person guessed the future rates and valued the money. Now, these jobs are split.
>
> *   **The Projector (Blue Curve):** "I am looking at the term index market (legacy IBOR). I predict the floating rate in 5 years will be 4.00%."
> *   **The Auditor (Green Curve):** "I don't care about risky banks. I care about the *value of money* today. Since this trade is collateralized with cash, correct value is determined by the safe rate (OIS/SOFR). I will discount that 4.00% payment using the 3.00% risk-free rate."
>
> The key point is that the curve used to forecast floating coupons need not be the curve used to discount those cashflows.

For a swap referencing index $k$, the floating forward rate is:

$$\boxed{L_k(0; T_i, T_{i+1}) = \frac{1}{\tau_i^k}\left(\frac{P_k(0, T_i)}{P_k(0, T_{i+1})} - 1\right)}$$

The swap value becomes:

$$V^k_{\text{swap}}(0) = \sum_j c \cdot \tau_j^{\text{fix}} \cdot P_d(0, T_j^{\text{fix}}) - \sum_i L_k(0; T_i, T_{i+1}) \cdot \tau_i^k \cdot P_d(0, T_{i+1})$$

Note carefully: forwards come from $P_k$, but discounting uses $P_d$.

### 25.7.3 OIS Discounting for Collateralized Swaps

What curve should we use for discounting? For a collateralized swap, the discounting choice is tied to the **collateral rate** (the rate earned/paid on variation margin or collateral under the CSA/clearing setup). In many rate markets this leads to an OIS-style discount curve for collateralized trades.

One argument: if collateral remuneration is tied to an overnight rate (e.g., Fed funds/OIS in USD), then a term bank rate is not a good proxy for the discount rate on collateralized trades.

Practical takeaway: in a multi-curve world, always state explicitly:
- which curve is used to **project** the floating coupons, and
- which curve is used to **discount** all cashflows.

> **Visual: The Dual Curve Diagram**
>
> *   **X-Axis:** Time.
> *   **Y-Axis:** Rate %.
> *   **Blue Line (Projection)**: High and volatile. Represents the 3-month Bank Rate. This determines *how much cash flows*.
> *   **Green Line (Discount)**: Lower and stable. Represents the Overnight Risk-Free Rate. This determines *how much that cash is worth*.
> *   **The Spread**: The gap between them is the "Basis." It fluctuates with credit stress.
>
> **The Key**: Even if the Basis blows out (Blue line spikes), the Green line might stay flat. Your *cash flows* increase, but your *discount rate* doesn't. This is why multi-curve valuation is critical during crises.

### 25.7.4 The Par Rate Under Multi-Curve

In the multi-curve framework, setting $V_{\text{swap}} = 0$ and solving for the fixed rate:

$$\boxed{c_{\text{par}}^{\text{multi}} = \frac{\sum_i \tau_i^{\text{flt}} \cdot L_k(0; T_i, T_{i+1}) \cdot P_d(0, T_{i+1}^{\text{flt}})}{\sum_j \tau_j^{\text{fix}} \cdot P_d(0, T_j^{\text{fix}})}}$$

**Critical observation:** The telescoping identity fails in multi-curve. Since forwards come from $P_k$ but discounting uses $P_d$, the convenient $1 - P(0,T)$ result no longer applies. You must compute each floating coupon's PV explicitly.

> **Pitfall — Telescoping in Multi-Curve:** Using $PV_{\text{float}}/N = 1-P_d(0,T)$ when your forwards come from a different curve.
> **Why it matters:** You will misprice the floating leg and get misleading “basis” P&L and risk.
> **Quick check:** If your spreadsheet can compute the floating PV without referencing the projection curve, you implicitly assumed single-curve.

### 25.7.5 Risk in One Page: PVBP and DV01 (Preview)

Full swap risk reports and hedging workflows are covered in Chapter 26, but you should already be able to read a risk scalar **with units and sign**.

- **Bump object (state it explicitly):** the curve object you bump and rebuild. A common choice is the **discounting-curve zero rates** (equivalently discount factors), with the projection curve held fixed for a “discounting DV01” view.
- **Bump size:** 1bp $=10^{-4}$.
- **Definition (this book):** $DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$ for the stated bump object.
- **Units:** currency per 1bp for the stated notional (e.g., USD per 1bp for $N=USD100\text{ million}$).
- **Sign (implied by the definition):** receiver-fixed has $DV01\gt 0$; payer-fixed has $DV01\lt 0$.

**Pitfall — PVBP vs DV01:** $\text{PVBP}_{\text{fixed}}$ answers “what happens if I bump the contractual fixed rate $c$ by 1 bp, holding curves fixed?” DV01 answers “what happens if I bump a curve object by 1 bp and rebuild/reprice?” Both are currency per bp, but they are sensitivities to different objects; if a report says “PVBP” without stating the bump object, treat it as ambiguous.

**Expand (multi-curve risk buckets):** In a two-curve setup, it is common to report at least:
- **Discount DV01:** bump the discount curve (projection held fixed).
- **Projection DV01:** bump the projection curve for the floating index (discount held fixed).
- **Basis DV01:** bump the spread/basis linking projection to discount (or between projection curves).
These buckets are all “rate-like” but hedge with different instruments; Chapter 26 covers the standard hedging workflow.

A useful intermediate object is the fixed-leg annuity $A^{\text{fix}}(0)$. Holding curves fixed, bumping the contractual fixed rate $c$ up by 1bp changes the receiver-fixed PV by approximately:
$$\boxed{\text{PVBP}_{\text{fixed}} \approx N \cdot A^{\text{fix}}(0) \cdot 10^{-4}.}$$

Rule of thumb: the DV01 magnitude is often on the same order as the fixed-leg PVBP because the floating leg has little duration beyond the next reset, but the exact relation depends on the curve-bump methodology and the valuation date.

---

## 25.8 Worked Examples

### Worked Example (Template): Pricing a 1-Year USD Swap from Two Curves

**Context**
- Price a 1-year receive-fixed/pay-float swap at inception and value an off-market fixed rate.
- Why this matters: this is the desk “quote → cashflows → PV” pipeline, plus the PVBP scale you use to sanity-check risk.

**Timeline (Make Dates Concrete)**
- Trade date: 2026-03-02
- Effective (settlement) date: 2026-03-04 (assume spot-start, T+2)
- Floating payment dates (quarterly): 2026-06-04, 2026-09-04, 2026-12-04, 2027-03-04
- Fixed payment dates (semiannual): 2026-09-04, 2027-03-04

**Inputs**
- Notional: $N=USD100{,}000{,}000$
- Fixed leg: semiannual, 30/360 (so $\tau^{\text{fix}}=0.5$ each period)
- Floating leg: quarterly, ACT/360 (so $\tau_i = \text{days}/360$)
- Discounting curve discount factors $P_d(0,T)$:

| Date | $P_d$ |
|---|---:|
| 2026-06-04 | 0.9940 |
| 2026-09-04 | 0.9880 |
| 2026-12-04 | 0.9820 |
| 2027-03-04 | 0.9760 |

- Projection curve discount factors $P_{3M}(0,T)$ for the floating index (assume $P_{3M}(0)=1$):

| Date | $P_{3M}$ |
|---|---:|
| 2026-06-04 | 0.9930 |
| 2026-09-04 | 0.9860 |
| 2026-12-04 | 0.9790 |
| 2027-03-04 | 0.9720 |

**Outputs (What You Produce)**
- Par fixed rate: $c_{\text{par}} \approx 2.859\%$
- Receiver-fixed NPV at $c=3.20\%$: $V_{\text{recv}} \approx +USD335{,}249$
- Fixed-leg PVBP (1bp): $\text{PVBP}_{\text{fixed}} \approx USD9{,}820/\text{bp}$ for $N=USD100\text{ million}$

**Step-by-step**
1. **Build accrual factors (ACT/360) for the floating leg**
   - 2026-03-04 → 2026-06-04: $92/360 \approx 0.2556$
   - 2026-06-04 → 2026-09-04: $92/360 \approx 0.2556$
   - 2026-09-04 → 2026-12-04: $91/360 \approx 0.2528$
   - 2026-12-04 → 2027-03-04: $90/360 = 0.25$

2. **Compute forward rates from the projection curve**
   $$L_i = \frac{1}{\tau_i}\left(\frac{P_{3M}(0,T_{i-1})}{P_{3M}(0,T_i)}-1\right)$$
   Using $P_{3M}(0)=1$, the implied quarterly forwards are approximately:
   - $L_1 \approx 2.758\%$, $L_2 \approx 2.778\%$, $L_3 \approx 2.829\%$, $L_4 \approx 2.881\%$

3. **PV the floating leg using the discount curve**
   $$PV_{\text{float}} = N\sum_i \tau_i\,L_i\,P_d(0,T_i) \approx USD2{,}807{,}151$$

4. **Compute the fixed-leg annuity and the par rate**
   $$A^{\text{fix}}(0)=0.5\,P_d(0,\text{2026-09-04})+0.5\,P_d(0,\text{2027-03-04})=0.9820$$
   $$c_{\text{par}}=\frac{(PV_{\text{float}}/N)}{A^{\text{fix}}(0)}\approx 2.859\%$$

5. **Value an off-market fixed rate**
   For a receive-fixed swap at $c=3.20\%$,
   $$V_{\text{recv}} \approx N\,(c-c_{\text{par}})\,A^{\text{fix}}(0) \approx +USD335{,}249.$$

**Cashflows (Receiver-Fixed, Pay-Float; $c=3.20\%$)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-06-04 | $-USD704{,}935$ | pay floating |
| 2026-09-04 | $+USD890{,}061$ | receive fixed, pay floating |
| 2026-12-04 | $-USD715{,}015$ | pay floating |
| 2027-03-04 | $+USD879{,}835$ | receive fixed, pay floating |

**P&L / Risk Interpretation**
- The NPV is the upfront amount (in PV terms) you would receive/pay to enter a swap at a fixed rate away from par.
- Receiver-fixed is long rates risk: if the par swap rate rises after you trade, your PV tends to fall.

**Sanity Checks**
- **Units:** discount factors are unitless; rates are “per year”; $\tau$ is in years; cashflows are in USD.
- **Day-count check:** over this calendar year, the floating-leg ACT/360 accrual sums to $365/360\approx 1.0139$, while the fixed-leg 30/360 accrual sums to exactly 1.0.
- **Par check:** setting $c=c_{\text{par}}$ makes the PV approximately zero (up to rounding).

### Example A: Par Swap Rate from Discount Factors (Single-Curve)

**Problem:** Given discount factors, compute the par rate for a 1-year spot-start swap with semiannual fixed payments.

**Given:**
- $P(0, 0.5) = 0.9850$
- $P(0, 1.0) = 0.9700$
- Fixed leg: semiannual, $\tau^{\text{fix}} = 0.5$ each period

**Step 1: Compute the fixed-leg annuity**

$$A^{\text{fix}}(0) = 0.5 \times 0.9850 + 0.5 \times 0.9700 = 0.4925 + 0.4850 = 0.9775$$

**Step 2: Compute floating leg PV using the telescoping identity**

For a spot-start swap in single-curve:

$$\frac{PV_{\text{float}}}{N} = 1 - P(0, 1.0) = 1 - 0.9700 = 0.0300$$

**Step 3: Compute the par fixed rate**

$$c_{\text{par}} = \frac{0.0300}{0.9775} = 0.03069 \approx 3.069\%$$

**Sanity check:** With discount factors near 0.97-0.98, the 1-year zero rate is roughly 3%, so a par swap rate around 3.07% is reasonable (slightly higher due to the annuity effect).

### Example B: Off-Market Swap Valuation

**Problem:** Value a swap where the fixed rate differs from the par rate.

**Setup:** Same curves as Example A, notional $N = USD100,000,000$, but the swap has a fixed rate of $c = 3.20\%$ (above par).

**Step 1: PV of floating leg**

Using the single-curve identity:
$$PV_{\text{float}} = N(1 - P(0,1)) = USD100,000,000 \times 0.0300 = USD3,000,000$$

**Step 2: PV of fixed leg**

$$PV_{\text{fixed}} = N \cdot c \cdot A^{\text{fix}}(0) = USD100,000,000 \times 0.032 \times 0.9775 = USD3,128,000$$

**Step 3: Swap NPV**

For the receiver (receive fixed, pay float):
$$V_{\text{recv}} = PV_{\text{fixed}} - PV_{\text{float}} = USD3,128,000 - USD3,000,000 = +USD128,000$$

For the payer (pay fixed, receive float):
$$V_{\text{payer}} = -USD128,000$$

**Alternative formula:** Using the off-market relationship:
$$V_{\text{recv}} = N(c - c_{\text{par}}) \cdot A^{\text{fix}}(0) = USD100,000,000 \times (0.032 - 0.03069) \times 0.9775 = USD128,000 \checkmark$$

### Example C: Multi-Curve vs. Single-Curve Pricing

**Problem:** Show how using a projection curve different from the discount curve changes the par swap rate.

**Setup:** 1-year swap, quarterly floating, semiannual fixed.

**Given curves:**

| $T$ | OIS Discount $P_d$ | 3M Projection $P_{3M}$ |
|-----|-------------------|------------------------|
| 0.25 | 0.9940 | 0.9930 |
| 0.50 | 0.9880 | 0.9860 |
| 0.75 | 0.9820 | 0.9790 |
| 1.00 | 0.9760 | 0.9720 |

**Single-curve par rate (using OIS for both):**

Annuity: $A_d^{\text{fix}} = 0.5 \times 0.9880 + 0.5 \times 0.9760 = 0.9820$

Floating PV (telescoping): $1 - 0.9760 = 0.0240$

Par rate: $c_{\text{par}}^{\text{single}} = 0.0240 / 0.9820 = 2.444\%$

**Multi-curve par rate (project with 3M, discount with OIS):**

*Step 1: Compute 3M forwards from projection curve*

$$L_i^{3M} = \frac{1}{0.25}\left(\frac{P_{3M}(0, T_{i-1})}{P_{3M}(0, T_i)} - 1\right)$$

| Period | Forward Rate |
|--------|--------------|
| $0 \to 0.25$ | $\frac{1}{0.25}(1/0.9930 - 1) = 2.820\%$ |
| $0.25 \to 0.50$ | $\frac{1}{0.25}(0.9930/0.9860 - 1) = 2.840\%$ |
| $0.50 \to 0.75$ | $\frac{1}{0.25}(0.9860/0.9790 - 1) = 2.860\%$ |
| $0.75 \to 1.00$ | $\frac{1}{0.25}(0.9790/0.9720 - 1) = 2.881\%$ |

*Step 2: PV floating coupons using OIS discount factors*

$$\frac{PV_{\text{float}}^{\text{multi}}}{N} = \sum_{i=1}^{4} 0.25 \times L_i^{3M} \times P_d(0, T_i) = 0.02807$$

*Step 3: Par fixed rate*

$$c_{\text{par}}^{\text{multi}} = \frac{0.02807}{0.9820} = 2.859\%$$

**Basis effect:**

| Method | Par Rate |
|--------|----------|
| Single-curve | 2.444% |
| Multi-curve | 2.859% |
| **Difference** | **41.5 bp** |

The projection curve implies higher forward rates than those from the OIS curve, so the par fixed rate is higher in the multi-curve framework.

### Example D: Swap Spread Conversion

**Problem:** Given market quotes, compute the swap rate.

**Given:**
- 5-year on-the-run Treasury yield: 4.25%
- 5-year swap spread: +8.5 basis points

**Solution:**

$$\text{5-year Swap Rate} = 4.25\% + 0.085\% = 4.335\%$$

If bid/offer on the spread is 7.5/9.5 bp:
- Dealer pays fixed at: 4.25% + 0.075% = 4.325%
- Dealer receives fixed at: 4.25% + 0.095% = 4.345%

---

## 25.9 Off-Market Swaps and Unwinds

### 25.9.1 When Swaps Trade Off-Market

Not all swaps trade at par. Off-market swaps occur when:
- A swap is restructured or rolled
- A specific fixed rate is desired for accounting reasons
- A legacy swap is assigned to a new counterparty

For an off-market swap, the party receiving the "better" rate pays an upfront amount equal to the swap's NPV to the other party.

### 25.9.2 Upfront Payments

If a 5-year par swap rate is 4.00% but a client wants to pay fixed at 3.50%, they must compensate the dealer for the below-market rate. The upfront payment is:

$$\text{Upfront} = N \times (c_{\text{par}} - c_{\text{off-market}}) \times A^{\text{fix}}$$

**Example:** USD100 million notional, 5-year, with annuity factor 4.5:

$$\text{Upfront} = USD100M \times (0.0400 - 0.0350) \times 4.5 = USD2,250,000$$

The client pays USD2.25 million upfront and then pays 3.50% fixed over the life of the swap.

### 25.9.3 Swap Termination and Unwinds

When you terminate (“unwind”) a swap, the economic idea is simple: exchange a cash amount equal to the swap’s NPV (plus any unpaid amounts/accruals as specified by the contract) on the termination settlement date.

**Rule of thumb (near par):** for the remaining maturity,
$$V \approx N \times (c_{\text{market}} - c_{\text{old}}) \times A^{\text{fix}}.$$

**Step 1: Determine who owes whom**

You are paying 5.75% fixed but the current market rate is 6.25%. You locked in a below-market rate—your swap has positive value. The counterparty must pay you to terminate.

**Step 2: Calculate termination value**

$$V = N \times (c_{\text{market}} - c_{\text{old}}) \times A^{\text{fix}}$$

The annuity for a 9-year swap paying semiannually can be approximated from the discount factor, or computed directly. Using approximate annuity of 7.5:

$$V \approx USD10,000,000 \times (0.0625 - 0.0575) \times 7.5 = USD375,000$$

You *receive* approximately USD375,000 to terminate the swap.

> **Desk Reality: Unwind vs. Offset**
>
> There are two ways to exit a swap position:
>
> **Unwind (Termination):** Negotiate with the original counterparty to cancel the swap. Cash changes hands equal to the NPV.
>
> **Offset (New Trade):** Enter an opposite swap with the same terms. This creates two swaps that largely cancel each other, but you now have two positions (and potentially two margin requirements if cleared).
>
> Unwinding is cleaner but requires counterparty cooperation. Offsetting can be done anytime with any counterparty.

### 25.9.4 Assignment and Novation

**Assignment:** Transferring your swap obligations to a third party. Requires consent of original counterparty.

**Novation:** The legal process of substituting a new counterparty into an existing contract. The CCP clearing process is essentially a novation—the original bilateral trade becomes two trades with the CCP.

---

## 25.10 The Floating-Rate Note Perspective

### 25.10.1 Why Floaters Trade at Par

On each reset date, a floating-rate note is approximately worth **par**: the coupon for the next period is set to the prevailing market rate for that period, so (ignoring credit and frictions) the note is “fair” for the next accrual interval.

Between reset dates, the floater’s value is close to par and can be approximated by discounting the next cashflow:

$$P_{\text{floater}} = \frac{N + N \cdot L_{\text{current}} \cdot \tau}{1 + L_{\text{today}} \cdot \tau'}$$

where $L_{\text{current}}$ is the rate already set for the current period and $\tau'$ is the time remaining to payment.

### 25.10.2 Duration of a Floating-Rate Note

The key risk intuition: even if a floater has a long final maturity, its price sensitivity is dominated by the **next reset/payment**. As a result, the duration of a floater is often close to the time to the next reset (e.g., ~0.25 years for quarterly resets). This is why the floating leg of a swap has much less duration than the fixed leg.

---

## 25.11 The Swap as a Portfolio of FRAs

### 25.11.1 Decomposition into Period-by-Period Exposures

You can view a vanilla swap as a strip of period-by-period forward contracts: each payment date exchanges “floating minus fixed” for that accrual period. This is closely related to thinking of each period as an FRA-like cash exchange.

For a payer swap with aligned fixed and floating schedules, the value can be written:

$$V_{\text{payer}} = N \sum_{i=1}^{n} \tau_i (L_i - c) \cdot P_d(0, T_i)$$

Each term $(L_i - c) \cdot \tau_i \cdot P_d(0, T_i)$ resembles the PV of an FRA that locks in receiving $L_i$ and paying $c$ over period $i$.

### 25.11.2 Example: FRA Strip Decomposition

**Setup:** 3-period swap with quarterly payments at $T = \{0.25, 0.50, 0.75\}$, single-curve, $N = 1$, fixed rate $c = 3.10\%$.

**Given:**
- Discount factors: $P(0, 0.25) = 0.9925$, $P(0, 0.50) = 0.9850$, $P(0, 0.75) = 0.9775$
- Forwards: $L_1 = 3.023\%$, $L_2 = 3.046\%$, $L_3 = 3.069\%$

**FRA-like decomposition:**

| Period | $(L_i - c)$ | $\tau_i(L_i - c) \cdot P(0,T_i)$ |
|--------|-------------|----------------------------------|
| 1 | -0.077% | $-0.000192$ |
| 2 | -0.054% | $-0.000133$ |
| 3 | -0.031% | $-0.000076$ |
| **Total** | | $-0.000401$ |

The payer swap has negative value because the fixed rate (3.10%) exceeds each forward rate—the payer locked in an above-market rate.

---

## 25.12 Non-Vanilla Swap Structures

While this chapter focuses on vanilla fixed-for-floating swaps, practitioners encounter several variations:

### 25.12.1 Basis Swaps (Floating-Floating)

A basis swap exchanges two floating rates—for example, 3-month vs. 1-month terms of the same index, or two different floating indices.

Basis swaps matter because different tenors/indices can embed different credit/liquidity/funding components, so their forward curves can move differently. They are important both for curve construction and for managing **tenor/index basis** risk.

**Why basis swaps exist:**
- Banks have mismatched funding (borrow at 1M, lend at 3M)
- Hedging specific rate exposures
- Exploiting perceived mispricing between tenors

Basis swaps are covered in detail in Chapter 28.

### 25.12.2 Forward-Starting Swaps

A forward-starting swap has an effective date in the future. For example, a "1y2y" swap starts in 1 year and runs for 2 years thereafter.

**Valuation:** Same framework as spot-starting swaps, but:
- The forward rate for the first floating period is the forward rate from today to the first fixing
- The annuity factor starts from the effective date

Forward-starting swaps are commonly used to:
- Hedge anticipated bond issuance
- Express views on future rate levels
- Structure swaptions (options on forward-starting swaps)

### 25.12.3 Amortizing and Accreting Swaps

**Amortizing swap:** Notional declines over time (common for mortgage-backed securities hedging)

**Accreting swap:** Notional increases over time (matches construction loan drawdowns)

Valuation treats each period as a separate swap with that period's notional.

---

## 25.13 Practical Setup Checklist

### 25.13.1 Standard USD Swap Conventions (Summary)

| Element | Fixed Leg | Floating Leg |
|---------|-----------|--------------|
| Day count | 30/360 | ACT/360 |
| Payment frequency | Semiannual | Quarterly (3M SOFR) |
| Rate setting | At inception | Compounded SOFR in arrears |
| Business day | Modified following | Modified following |
| Holiday calendar | US Federal Reserve | US Federal Reserve |

**Important:** These conventions vary by currency and evolve over time. The LIBOR-to-SOFR transition, for example, has changed floating leg conventions significantly.

### 25.13.2 Day Count Impact

The different day counts on fixed and floating legs create subtle effects. Consider a 1-year period:

- Under 30/360: accrual factor = 360/360 = 1.000
- Under ACT/360: accrual factor = 365/360 = 1.0139

Even if both legs have the same quoted rate, the ACT/360 leg generates ~1.4% more cashflow per year than the 30/360 leg.

### 25.13.3 Payment Netting

In practice, when fixed and floating payments fall on the same date, the payments are often **netted** so only the difference is exchanged. This reduces operational settlement flows.

---

## 25.14 Common Operational Errors and P&L Breaks

> **Desk Reality: What Goes Wrong**
>
> Middle-office and operations teams regularly encounter P&L breaks on swaps. Here are the most common causes:
>
> **Day Count Mismatches:**
> - Front-office system uses ACT/365; risk system uses ACT/360
> - One system treats 30/360 as "US" convention; another uses "European"
> - Result: Small but persistent discrepancies in accrued interest
>
> **Stub Period Errors:**
> - First period is shorter than standard (a "short stub")
> - Systems disagree on how to handle the stub accrual factor
> - Example: First period is 47 days instead of 90 days—is the accrual factor 47/360 or should it be adjusted?
>
> **Holiday Calendar Misalignment:**
> - One system uses NYSE holidays; another uses Federal Reserve calendar
> - Payment date falls on a holiday in one calendar but not the other
> - Result: One-day timing differences that compound over the swap's life
>
> **RFR Observation / Fixing Confusion:**
> - Which dates are *observed* vs. which dates define the accrual period?
> - How are weekends/holidays handled (day-count weights and “carried” rates)?
> - Is there a payment delay or shifted observation window in the contract?
>
> **Modified Following Errors:**
> - If a payment date falls on a weekend, does it move forward or back?
> - What if moving forward crosses month-end?
> - Systems may implement "modified following" differently

### 25.14.1 Sanity Checks for Swap Operations

| Check | What to Verify | Red Flag |
|-------|----------------|----------|
| Par swap PV | Should be ~0 for a new par swap | PV > 0.1% of notional |
| Cashflow sum | Notional × rate × accrual ≈ cashflow | Difference > 1 cent per million |
| Bucket PV01 | Sum of bucket PV01s ≈ total PV01 | Difference > 5% |
| First fixing | First floating payment matches market rate | Difference > 1bp |
| Duration | Approximately half the maturity for a par swap | Duration > maturity |

---

## 25.15 How Swap Value Changes Over Time

### 25.15.1 Initial Value Structure

A par swap has zero net present value at inception, but this does not mean that each cash flow exchange is worth zero. Individual exchanges can have positive or negative value; only the **sum** is zero.

Consider a swap where the term structure is upward sloping and the fixed rate is being received while floating is paid. Forward rates increase with maturity, so early floating payments are expected to be lower than later ones. Since the fixed rate is constant, early exchanges favor the fixed receiver (fixed exceeds expected floating) while later exchanges favor the floating receiver (expected floating exceeds fixed).

### 25.15.2 Expected Value Path

This structure implies a predictable pattern for how the swap's value will evolve:

- **Fixed receiver, upward-sloping curve**: Early exchanges have positive value, later exchanges have negative value. As time passes and the positive-value exchanges are settled, the remaining swap is expected to have negative value.

- **Fixed payer, upward-sloping curve**: The opposite pattern. Early exchanges have negative value to the payer, later exchanges have positive value. The swap value is expected to become positive over time.

These patterns are useful for building intuition about expected exposure profiles, but they are not a substitute for simulation or for actual market moves.

**Important caveat:** These are expected values in a risk-neutral sense. Actual swap values depend on realized rate movements. If rates decline unexpectedly, a fixed-receiver swap could have positive value throughout its life despite the initial structure suggesting otherwise.

### 25.15.3 Implications for Credit Exposure

The expected value path has direct implications for counterparty risk:

- If your swap position is expected to become more valuable over time, your counterparty exposure increases—the potential loss from counterparty default grows.
- If your swap position is expected to become less valuable, your exposure decreases over time.

This asymmetry is one reason why credit risk management for swap portfolios requires careful modeling of potential future exposure (PFE) profiles, not just current mark-to-market values.

---

## 25.16 Credit Risk of Swap Agreements

### 25.16.1 Why Swap Credit Exposure Is Not “Notional”

A swap’s notional is not exchanged. Credit exposure is therefore not “USDN at risk”; it is the **mark-to-market (NPV)**: what you would lose if the counterparty defaulted today and you had to replace the trade.

Key implications:
1. **No principal at risk:** losses are bounded by the swap’s value at default, not by notional.
2. **Netting matters:** under master agreements/clearing, offsetting trades can net, reducing exposure.
3. **Order-of-magnitude intuition:** PV swings are often on the order of $|DV01| \times |\Delta y_{bp}|$. For example, if a USD100 million swap has $DV01 \approx USD40{,}000/\text{bp}$, then a 50bp move is on the order of USD2 million of PV swing—not USD100 million.

### 25.16.2 Collateral and Margin (CSA / CCP)

For many swaps, collateralization dominates the practical credit picture:
- **Variation margin** transfers track current mark-to-market.
- **Initial margin** is posted to cover adverse moves over the margin period of risk.
- **Thresholds, MTAs, and margin frequency** determine residual “gap risk”.

In the idealized limit of continuous margining with zero thresholds, current exposure is largely collateralized; the remaining risk is operational and jump-to-default-style gap risk.

### 25.16.3 Link Back to Discounting

Because collateral typically earns an overnight rate, collateralization connects naturally to OIS-style discounting (Section 25.7). Uncollateralized trades require additional credit/funding adjustments beyond this chapter’s scope; see Chapter 33 for discounting implications and later chapters for CVA/XVA concepts.

---

## 25.17 Major Uses of Interest Rate Swaps

Interest rate swaps are used for three primary purposes:

### 25.17.1 Creating Synthetic Floating-Rate Funding

Corporations often issue fixed-rate debt (because that investor base is deep) but prefer floating-rate exposure for internal funding or risk reasons. Two stylized approaches:

1. **Roll short-term debt**: Issue and roll over commercial paper or other short-term obligations. This works for highly-rated issuers but creates *liquidity risk*—the risk that credit deterioration prevents refinancing.

2. **Issue fixed + enter swap**: Issue fixed-rate bonds and receive fixed / pay floating in a swap. The fixed leg payments offset, leaving synthetic floating-rate funding. This avoids liquidity risk while achieving the desired rate profile.

### 25.17.2 The Comparative Advantage Argument

A common textbook motivation for swaps is the *comparative advantage* story. Consider two companies:

| Company | Fixed Rate | Floating Rate |
|---------|-----------|---------------|
| AAACorp | 4.0% | SOFR + 0.3% |
| BBBCorp | 5.2% | SOFR + 1.0% |

AAACorp has absolute advantage in both markets, but BBBCorp has *comparative* advantage in the floating market (its spread over AAACorp is smaller: 70bp in floating vs. 120bp in fixed).

If AAACorp wants floating and BBBCorp wants fixed:
- AAACorp borrows fixed at 4.0%, enters swap to receive 4.35%, pay SOFR → Net: SOFR - 0.35%
- BBBCorp borrows floating at SOFR + 1.0%, enters swap to pay 4.35%, receive SOFR → Net: 5.35%

Both are better off: AAACorp pays SOFR - 0.35% (vs. SOFR + 0.3%) and BBBCorp pays 5.35% (vs. 5.2%). Total gain: 50bp, split between them.

> **Practitioner Note:** Treat this as intuition, not an arbitrage. In real markets, liquidity, balance sheet, regulation, and transaction costs complicate the story.

### 25.17.3 Mortgage Market Hedging

Mortgage-related portfolios and other callable exposures have duration that changes as rates move. Swaps let desks add or remove fixed-rate duration without buying/selling large bond positions.

Mortgage servicers, who collect fees for processing mortgage payments, also face interest rate exposure and hedge with swaps. Chapter 27 discusses the interaction between mortgage hedging and swap spreads.

### 25.17.4 Hedging Future Debt Issuance

A corporation planning to issue bonds can hedge against rising rates by paying fixed on a swap (or entering a forward-starting swap) when the issuance decision is made, and then unwinding the hedge when the bond is priced.

**Versus Treasury hedging:** depending on the issuer and market, swaps can track funding-relevant rates more closely than Treasuries, but substantial basis risk remains.

---

## Summary

Interest rate swaps exchange fixed for floating payments on a notional principal that is never exchanged. The fixed leg generates deterministic cashflows; the floating leg generates cashflows based on a reference rate that resets periodically.

**Key valuation principles:**

1. The par swap rate is the fixed rate that makes a new swap worth zero
2. In single-curve pricing, the floating leg PV telescopes to $1 - P(0,T)$
3. In multi-curve pricing, projection and discounting curves differ
4. For collateralized swaps, the OIS curve provides the appropriate discount rate
5. A swap can be viewed as a portfolio of FRAs or as "long fixed bond, short floating bond"

**Market mechanics:**

6. Swaps are quoted as a spread to Treasuries in USD markets
7. Many standardized swaps are centrally cleared and/or collateralized; discounting and exposure depend on the collateral/margining setup
8. SOFR swaps use compounded overnight rates in arrears, with an observation schedule and payment timing specified in the confirmation
9. Off-market swaps require upfront payments equal to NPV

**Swap dynamics and credit:**

10. While a par swap has zero net value at inception, individual cash flow exchanges may have positive or negative value—only the sum is zero
11. The expected path of swap value over time depends on the term structure shape and which leg you receive
12. Swap credit risk is much lower than bond credit risk—loss is limited to the swap's current value, not principal
13. Collateralization through CSA agreements further reduces credit exposure to near zero

**Major uses:** Corporations use swaps to create synthetic floating-rate funding, hedge future debt issuance, and manage duration. The mortgage market is the largest source of swap hedging demand.

The separation of projection and discounting curves is not academic pedantry—it reflects real economic differences between the rate you expect to receive and the rate at which you should discount. Getting this right is essential for accurate pricing and risk management.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Par swap rate | Fixed rate making $V = 0$ at inception | Defines fair value for new swaps |
| Annuity | $A = \sum \tau_j P_d(0, T_j)$ | Converts between rate and PV |
| Projection curve | Curve generating forward rates | Determines expected floating cashflows |
| Discount curve | Curve discounting all cashflows | Determines present value |
| Telescoping identity | $PV_{\text{float}}/N = 1 - P(0,T)$ | Only valid in single-curve |
| OIS discounting | Using overnight rate curve for $P_d$ | Standard for collateralized swaps |
| Swap spread | Swap rate minus Treasury yield | Market convention for USD quoting |
| SOFR compounding | Daily rates compounded in arrears | Post-LIBOR floating rate convention |
| CCP clearing | Central counterparty stands between parties | Reduces counterparty risk |
| Expected value path | Swap value expected to evolve based on term structure | Affects counterparty exposure profiles |
| Swap credit risk | Loss limited to swap value, not notional | Much lower than bond default risk |
| CSA | Credit Support Annex governing collateral | Reduces swap credit risk to near zero |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $N$ | Notional principal | currency (e.g., USD) |
| $c$ | Contractual fixed swap rate | per year; fixed-leg day count applies |
| $c_{\text{par}}$ | Par swap rate | per year; makes $V=0$ at inception (given curves) |
| $\tau_j^{\text{fix}}$ | Fixed leg accrual factor for period $j$ | years; fixed-leg day count (e.g., 30/360) |
| $\tau_i^{\text{flt}}$ | Floating leg accrual factor for period $i$ | years; floating-leg day count (e.g., ACT/360) |
| $P_d(0, T)$ | Discount factor from discount curve | unitless; used for PV of all cashflows |
| $P_k(0, T)$ | Projection-curve discount factor for index $k$ | unitless; used to imply forwards for leg $k$ |
| $L_k(0; T_i, T_{i+1})$ | Forward rate for index $k$ over $[T_i, T_{i+1}]$ | per year; money-market convention for $k$ |
| $A^{\text{fix}}(0)$ | Fixed-leg annuity (sum of $\tau P_d$ on fixed schedule) | years |
| $\text{PVBP}_{\text{fixed}}$ | PV change from bumping the contractual fixed rate by 1bp (curves fixed) | currency per 1bp; sign depends on pay/receive |
| $DV01$ | PV change from a stated curve bump (book convention: rates down 1bp) | currency per 1bp; bump object must be specified |
| $R_{\text{compound}}$ | Compounded overnight (RFR) rate for a period | per year; compounding-in-arrears convention |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the par swap rate? | The fixed rate that makes a new swap worth zero at inception |
| 2 | In a payer swap, which leg do you pay? | Pay fixed, receive floating |
| 3 | Write payer swap PV in terms of leg PVs | $V_{\text{payer}} = PV_{\text{float}} - PV_{\text{fixed}}$ |
| 4 | What is the fixed-leg annuity? | $A^{\text{fix}} = \sum_j \tau_j^{\text{fix}} P_d(0, T_j^{\text{fix}})$ |
| 5 | Formula for forward rate from discount factors? | $L = \frac{1}{\tau}\left(\frac{P(0, T_i)}{P(0, T_{i+1})} - 1\right)$ |
| 6 | What does the discount curve do? | Converts future cashflows to present value |
| 7 | What does the projection curve do? | Generates forward rates for floating coupons |
| 8 | Single-curve par swap rate formula? | $S = \frac{P(0, T_\alpha) - P(0, T_\beta)}{A}$ |
| 9 | Floating leg PV in single-curve spot-start? | $PV_{\text{float}}/N = 1 - P(0, T)$ |
| 10 | Why does telescoping fail in multi-curve? | Forwards from $P_k$, discounting from $P_d$—different curves |
| 11 | What is OIS discounting? | Using overnight index curve as discount curve |
| 12 | Why use OIS for discounting? | Collateral earns overnight rate, not LIBOR |
| 13 | Typical USD fixed leg day count? | 30/360 |
| 14 | Typical USD floating leg day count? | ACT/360 |
| 15 | Duration of a floating-rate note? | Approximately time to next reset (~0.25 years) |
| 16 | Off-market receiver PV formula? | $V_{\text{recv}} = N(c - c_{\text{par}}) \cdot A^{\text{fix}}$ |
| 17 | What is a CSA? | Credit Support Annex—governs collateral posting |
| 18 | How can a swap be viewed as bonds? | Long fixed-rate bond, short floating-rate note |
| 19 | How can a swap be viewed as FRAs? | Portfolio of period-by-period forward contracts |
| 20 | What happens to payer swap if rates rise? | Gains value (receive higher float, pay same fixed) |
| 21 | What does "5s are 10.5 out" mean? | 5-year swap spread is 10.5bp over Treasuries |
| 22 | What is SOFR compounding in arrears? | Daily overnight rates are compounded over the period to determine payment |
| 23 | What defines the RFR observation schedule? | The confirmation: observation dates, compounding convention, and any payment delay/lockout; do not assume a single “standard” rule |
| 24 | Why is SOFR backward-looking vs LIBOR forward-looking? | SOFR compounds actual observed daily rates; LIBOR was a term rate known at period start |
| 25 | What is a CCP? | Central counterparty—stands between both parties to a cleared swap |
| 26 | For a par swap, is each cash flow exchange worth zero? | No—sum of exchange values is zero, but individual exchanges have positive/negative values |
| 27 | If term structure is upward sloping and you receive fixed, how does swap value evolve? | Expected to become negative over time as positive-value early exchanges settle |
| 28 | Why is swap credit risk much lower than bond credit risk? | Notional is never exchanged; loss limited to swap value at default, not principal |
| 29 | How does collateral posting reduce swap credit risk? | Party with negative value posts margin; default loss covered by margin |
| 30 | DV01 (book convention): what is it and what is bumped? | $DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$ for a stated bump object (e.g., discounting-curve zeros); units are currency per bp |

---

## Mini Problem Set

**Q1.** A 1-year spot-start single-curve swap has $P(0,1) = 0.965$. What is $PV_{\text{float}}/N$?

**Q2.** Using Q1, suppose the fixed leg is annual with $\tau = 1$. What is the par fixed rate?

**Q3.** Same as Q2 but fixed leg is semiannual with $P(0, 0.5) = 0.982$. Compute the annuity and $c_{\text{par}}$.

**Q4.** For Q3, if the swap fixed rate is 20bp above par, compute the receiver PV per unit notional.

**Q5.** A payer swap has $N = USD50M$, $A^{\text{fix}} = 4.2$, and $c$ is 10bp above par. Approximate PV.

**Q6.** Write the general multi-curve par rate formula.

**Q7.** Explain why $PV_{\text{float}} = N(1 - P_d(0,T))$ fails in multi-curve.

**Q8.** For a payer swap, which direction does PV move if forward rates increase (holding discounting fixed)?

**Q9.** The term structure is upward-sloping and you enter a 5-year receiver swap at par. Should you expect your counterparty credit exposure to increase or decrease over the first few years? Explain.

**Q10.** A $100 million receiver-fixed swap has $DV01 = USD45{,}000/\text{bp}$ (book convention; stated bump object). If discounting rates move **up** by 75bp since inception and the swap was entered at par, approximate the PV change and state which side has positive exposure (ignore collateral and convexity).

**Q11.** The 10-year Treasury yields 4.50% and 10-year swap spreads are quoted at -5/+2 basis points. What are the bid and offer swap rates?

**Q12.** Calculate the approximate compounded SOFR payment on USD25 million notional over a 92-day period, assuming constant daily SOFR of 5.00%.

---

### Solution Sketches (Selected)

**A1.** $1 - P(0,1) = 1 - 0.965 = 0.035$

**A2.** $A = P(0,1) = 0.965$; $c_{\text{par}} = 0.035/0.965 = 3.627\%$

**A3.** $A = 0.5(0.982) + 0.5(0.965) = 0.9735$; $c_{\text{par}} = 0.035/0.9735 = 3.595\%$

**A4.** $V_{\text{recv}}/N = (c - c_{\text{par}}) \cdot A = 0.002 \times 0.9735 = 0.00195$

**A5.** $V_{\text{payer}} = -N(c - c_{\text{par}}) \cdot A = -USD50M \times 0.001 \times 4.2 = -USD210,000$

**A6.** $c_{\text{par}} = \frac{\sum_i \tau_i^{\text{flt}} L_k(0; T_i, T_{i+1}) P_d(0, T_{i+1})}{\sum_j \tau_j^{\text{fix}} P_d(0, T_j^{\text{fix}})}$

**A7.** The identity relies on forwards and discounting using the same curve. In multi-curve, forwards come from $P_k$ while discounting uses $P_d$—the telescoping cancellation doesn't hold.

**A8.** Payer PV increases—higher forwards mean higher expected floating receipts.

**A9.** Decrease. With upward-sloping curve, early exchanges favor you (receive fixed > expected floating). As these positive-value exchanges settle, remaining swap expected to have negative value to you—counterparty exposure declines.

**A10.** Receiver-fixed has $DV01\gt 0$ under the book convention, so a **rates up** move is approximately the opposite sign:
$$
\Delta PV \approx -DV01 \times 75 \approx -USD45{,}000 \times 75 = -USD3.375\text{ million}
$$
So the receiver-fixed position is roughly $-USD3.375$ million; the payer-fixed counterparty has positive exposure of about USD3.375 million (ignoring collateral and convexity).

**A11.** Bid: 4.50% - 0.05% = 4.45%; Offer: 4.50% + 0.02% = 4.52%

**A12.** Compounded factor: $(1 + 0.05/360)^{92} = 1.01289$. Period rate: $(360/92) \times 0.01289 = 5.044\%$. Payment: $USD25M \times 0.05044 \times (92/360) = USD322,256$

---

## References

- Hull, *Options, Futures, and Other Derivatives*, “Swaps” and “Day Count Issues”.
- Tuckman & Serrat, *Fixed Income Securities: Tools for Today’s Markets*, “Interest Rate Swaps” and “Floating Rate Notes”.
- Andersen & Piterbarg, *Interest Rate Modeling* (Vol 1), “Multi-Curve Framework” and collateral discounting discussion.
- Brigo & Mercurio, *Interest Rate Models — Theory and Practice*, par swap rate / annuity formulation.
- Neftci, *Principles of Financial Engineering*, swap replication and “swap as a portfolio of FRAs”.
- Luenberger, *Investment Science*, swap valuation intuition (floater-at-par perspective).
