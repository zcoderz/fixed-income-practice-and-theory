# Chapter 25: Interest Rate Swaps — Mechanics and Valuation

---

## Introduction

A swap is not a bet on rates—it is an exchange of cashflow streams with zero initial value.

This seemingly simple statement contains the essence of what makes interest rate swaps the most important derivative in fixed income markets. When a corporation issues fixed-rate bonds but prefers floating-rate funding, it enters a swap. When a mortgage servicer needs to hedge the duration of its portfolio, it uses swaps. When a central bank wants to assess market expectations for future interest rates, it looks at the swap curve. The notional outstanding in interest rate swaps exceeds $400 trillion globally—dwarfing the underlying bond markets by an order of magnitude.

Yet for all their ubiquity, swaps create confusion because their value depends on *two distinct curves* that serve fundamentally different purposes: one curve tells us what floating payments to expect, while another tells us how to discount those payments to today. Getting this separation wrong—or not even knowing it exists—is a recipe for systematic mispricing.

This chapter builds your understanding of interest rate swaps from the ground up:

1. **Mechanics**: What exactly is exchanged, when, and how are payments calculated?
2. **Market conventions**: How swaps are quoted, executed, and cleared in modern markets
3. **Valuation fundamentals**: How to price a swap as a portfolio of cashflows
4. **The par swap rate**: Why swaps trade at par and how to derive the fixed rate
5. **Multi-curve pricing**: Why projection and discounting curves differ, and what happens when you ignore this
6. **SOFR conventions**: The post-LIBOR world and how overnight rate compounding works

We do *not* cover swap risk measures (DV01, PV01) here—Chapter 26 owns that territory. Nor do we discuss swap spreads in depth—Chapter 27 handles relative value. This chapter focuses on getting the mechanics and valuation exactly right, because everything else builds on this foundation.

---

## 25.1 What Is an Interest Rate Swap?

### 25.1.1 The Basic Contract

Hull provides a clean definition: "An interest rate swap is a swap where interest at a predetermined fixed rate, applied to a certain principal, is exchanged for interest at a floating reference rate, applied to the same principal, with regular exchanges being made for an agreed period of time."

The key elements are:

- **Notional principal** ($N$): The amount on which interest is calculated. Unlike a loan, this principal is never exchanged—it is "notional" in the sense that it exists only for calculation purposes.
- **Fixed leg**: One party pays a fixed rate $c$ applied to the notional.
- **Floating leg**: The other party pays a floating reference rate (historically LIBOR, now increasingly SOFR or other overnight rates) applied to the same notional.
- **Payment frequency**: Often different for the two legs—quarterly floating and semiannual fixed is common in USD markets.
- **Maturity**: The swap's lifetime, typically ranging from 1 to 30 years.

### 25.1.2 Payer vs. Receiver

The terminology reflects which leg you *pay*:

$$\boxed{\text{Payer swap: Pay fixed, receive floating}}$$
$$\boxed{\text{Receiver swap: Receive fixed, pay floating}}$$

A payer swap profits when rates rise (you receive higher floating payments while your fixed payments remain constant). A receiver swap profits when rates fall. This directional exposure makes swaps the primary tool for expressing interest rate views and hedging rate risk.

> **Desk Reality: The Language of Swaps**
>
> On a trading desk, you'll hear "I'm long duration" from someone who has a receiver swap (they benefit if rates fall) and "I'm short duration" from someone with a payer swap. The confusion potential is real: "paying" on a swap means paying fixed, which is *short* duration, even though you're "long" the swap contract itself.
>
> To avoid confusion, traders often just say "paying" or "receiving" rather than "long" or "short." When someone says "I'm paying 5s" they mean they're paying fixed on a 5-year swap.

### 25.1.3 Why Notional Is Not Exchanged

Tuckman explains the intuition elegantly: "The valuation of swaps without default risk is made much simpler by the following fiction. Treat the swap as if the fixed-rate payer pays the notional amount to the floating-rate payer on the termination date and as if the floating-rate payer pays the notional amount to the fixed-rate payer on the termination date. This fiction does not alter the cash flows because the payments of the notional amounts cancel."

This "bond replication" view is powerful: by imagining that notional is exchanged at maturity, the fixed leg becomes economically equivalent to a fixed-rate bond, and the floating leg becomes equivalent to a floating-rate note. We will exploit this decomposition throughout the chapter.

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

**Day count conventions matter.** Hull notes that "the fixed rate in a swap is also quoted with a day count convention. Popular fixed-rate day counts are actual/365 and 30/360." In USD swaps, the fixed leg typically uses 30/360, meaning each month is treated as having 30 days regardless of the actual calendar.

### 25.2.2 Floating Leg Cashflows

The floating leg pays amounts that depend on the reference rate observed during each period. For a standard "set in advance, pay in arrears" structure, the rate for period $[T_i^{\text{flt}}, T_{i+1}^{\text{flt}}]$ is observed (fixed) at $T_i^{\text{flt}}$ and paid at $T_{i+1}^{\text{flt}}$:

$$\boxed{CF_{i+1}^{\text{flt}} = N \cdot L_k(T_i; T_i, T_{i+1}) \cdot \tau_i^{\text{flt}}}$$

where $L_k(T_i; T_i, T_{i+1})$ is the floating reference rate for tenor $k$ observed at time $T_i$.

**The floating day count often differs from the fixed.** Tuckman explains that "floating rate cash flows are determined using the actual/360 convention" in typical USD markets. This means that even if both legs have the same notional and similar rates, they may produce different cashflows due to day count differences.

### 25.2.3 Example: A 2-Year USD Swap Schedule

Consider a \$100 million 2-year swap initiated on March 8, 2022, with:
- Fixed leg: 3% semiannual, 30/360 day count
- Floating leg: 3-month SOFR, quarterly, ACT/360 day count

The fixed leg pays \$100,000,000 × 3% × 0.5 = \$1,500,000 every six months.

The floating leg pays based on realized SOFR rates. Hull provides a worked example showing that if the 3-month SOFR reference rate for the first period is 2.2%, the floating payment would be \$100,000,000 × 2.2% × (91/360) = \$556,111, where 91 is the actual number of days in the quarter.

The payments are typically *netted*: only the difference between fixed and floating amounts changes hands on each payment date. If the fixed payment is \$750,000 (for a quarter using 30/360) and the floating payment is \$556,111, the fixed payer sends \$193,889 to the floating payer.

---

## 25.3 SOFR Swap Conventions

The transition from LIBOR to SOFR represents a fundamental shift in how floating-rate payments are determined. LIBOR was a forward-looking term rate—you knew at the start of the period what rate would apply. SOFR is backward-looking—the rate is determined by compounding daily overnight rates observed during the period.

### 25.3.1 Why SOFR Is Different

SOFR (Secured Overnight Financing Rate) is published daily by the Federal Reserve Bank of New York. Unlike LIBOR, which was a term rate (3-month LIBOR, 6-month LIBOR, etc.), SOFR is an overnight rate. To calculate a payment over a 3-month period, you must compound the daily SOFR observations.

Hull explains the compounding formula. For a period from day 1 to day $n$, with daily SOFR rates $r_1, r_2, \ldots, r_n$ and corresponding day counts $d_1, d_2, \ldots, d_n$, the compounded rate is:

$$\boxed{R_{\text{compound}} = \frac{360}{D} \left[\prod_{i=1}^{n} \left(1 + r_i \cdot \frac{d_i}{360}\right) - 1\right]}$$

where $D$ is the total number of days in the period.

### 25.3.2 Observation Shift and Payment Delay

Because SOFR is backward-looking and published with a one-day lag, several conventions have emerged to make payments operational:

**Lookback (Observation Shift):** The most common convention shifts the observation period backward by 2 business days. For a payment period from March 1 to June 1, the SOFR rates used are those from February 27 to May 28 (assuming 2-day shift). This allows the payment amount to be known 2 days before payment date.

**Payment Delay:** Some contracts keep the observation period aligned with the interest period but delay the payment by 2 business days after the period end.

**Lockout Period:** Some contracts stop observing new SOFR rates a few days before period end, using the last observed rate for the remaining days.

> **Desk Reality: SOFR Fixing and T+1 Publication**
>
> SOFR is published at approximately 8:00 AM Eastern Time on T+1—meaning the rate for Monday's transactions is published Tuesday morning. This creates operational complexity:
>
> - For a quarterly payment due June 1, if using a 2-day lookback, you need SOFR through May 28
> - The May 28 SOFR is published May 29
> - This gives 2 days (May 29-30) to calculate and confirm the payment amount
>
> Without the lookback, you wouldn't know the payment amount until the payment date itself, creating settlement risk.

### 25.3.3 Term SOFR vs. Overnight SOFR Compounding

Two SOFR-based products exist:

**Overnight SOFR Compounding (OIS-style):** This is the standard for derivatives. Daily SOFR rates are compounded in arrears. The rate is known only at period end.

**Term SOFR:** Published by CME, this is a forward-looking rate (30-day, 90-day, etc.) set at the beginning of the period. It is derived from SOFR futures prices and functions similarly to how LIBOR did.

> **Practitioner Note:** Most cleared USD interest rate swaps reference overnight SOFR compounded in arrears, not Term SOFR. The Term SOFR rate exists primarily for loans and some end-user derivatives where backward-looking rates create operational challenges.

### 25.3.4 Standard USD SOFR Swap Conventions

| Element | Fixed Leg | Floating Leg |
|---------|-----------|--------------|
| Day count | 30/360 | ACT/360 |
| Payment frequency | Semiannual (Annual for short) | Quarterly (Annual for short) |
| Rate determination | At inception | Compounded SOFR in arrears |
| Observation shift | N/A | 2 business days lookback |
| Business day | Modified following | Modified following |

### 25.3.5 Example: Computing a SOFR Floating Payment

**Problem:** Calculate the floating payment for a \$50 million notional swap over a 91-day quarter, given daily SOFR rates.

**Given:** For simplicity, assume constant SOFR of 4.25% for all 91 days (in practice, rates vary daily).

**Step 1: Compound the daily rates**

$$\prod_{i=1}^{91} \left(1 + 0.0425 \cdot \frac{1}{360}\right) = \left(1 + \frac{0.0425}{360}\right)^{91} = 1.010878$$

**Step 2: Annualize to get the period rate**

$$R = \frac{360}{91} \times (1.010878 - 1) = 0.04302 = 4.302\%$$

Note that the compounded rate (4.302%) slightly exceeds the simple average (4.25%) due to compounding.

**Step 3: Calculate the payment**

$$\text{Payment} = \$50,000,000 \times 0.04302 \times \frac{91}{360} = \$543,697$$

**Sanity check:** Simple interest would give \$50M × 4.25% × (91/360) = \$537,153. The compounded amount is about \$6,500 higher—a small but meaningful difference on large notionals.

---

## 25.4 Market Quoting Conventions and Execution

### 25.4.1 How Swaps Are Quoted: Spread to Treasuries

In the USD market, swaps are typically quoted as a *spread to Treasuries*. Tuckman explains: swap rates can be quoted directly (e.g., "5-year swaps at 3.75%") or as a spread over the benchmark Treasury yield.

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

Historically, swap spreads were positive because swaps were seen as carrying some bank credit risk (the reference rate was LIBOR, which reflected bank borrowing costs). However, since the 2008 crisis and especially after LIBOR transition, negative swap spreads have become common in certain tenors.

Tuckman notes: "The fall in swap spreads in the early 1990s reflected the recovery of the banking sector from its problems in the 1980s. The rise in swap spreads in the late 1990s, on the other hand, can be best explained by a perceived scarcity in the supply of U.S. Treasury securities relative to demand."

Negative swap spreads imply that the swap rate is *below* the Treasury yield—a counterintuitive result explained by:
- Treasury scarcity (high demand for Treasuries for regulatory purposes)
- Balance sheet constraints on dealers
- Supply/demand imbalances in the swap market itself

Chapter 27 explores swap spread dynamics in detail.

### 25.4.3 Standard Benchmark Tenors and IMM Dates

**Benchmark Tenors:** The most liquid swap tenors are: 2Y, 3Y, 5Y, 7Y, 10Y, 15Y, 20Y, 30Y. These form the "benchmark" swap curve.

**IMM Dates:** Many swaps trade with effective dates on "IMM dates"—the third Wednesday of March, June, September, and December. IMM-dated swaps are easier to hedge with futures and facilitate portfolio compression.

### 25.4.4 Trade Execution: SEF and Voice

Since the Dodd-Frank Act (2010), standardized swaps between financial institutions must be executed on Swap Execution Facilities (SEFs) and cleared through central counterparties.

Hull explains: "The Dodd-Frank act... expanded the role of the CFTC. For example, it is now responsible for rules requiring that standard over-the-counter derivatives between financial institutions be traded on swap execution facilities and cleared through central counterparties."

**Execution Methods:**
- **Request for Quote (RFQ):** Client requests prices from multiple dealers; best price wins
- **Central Limit Order Book (CLOB):** Anonymous trading on an electronic order book
- **Voice:** Large or non-standard trades may still be negotiated by phone

> **Practitioner Note:** Most interdealer trading in benchmark swaps occurs electronically on SEFs like Tradeweb and Bloomberg SEF. Client-to-dealer trades may use RFQ or voice depending on size and complexity.

### 25.4.5 CCP Clearing: How It Works

Hull describes the post-crisis clearing framework: "In over-the-counter derivatives markets, transactions are cleared either bilaterally or centrally. When central clearing is used, a central counterparty (CCP) stands between the two sides. It requires each side to provide margin and performs much the same function as an exchange clearing house."

**The CCP Process:**

1. **Trade Execution:** Two parties agree on a swap on a SEF or bilaterally
2. **Novation:** The CCP steps in as counterparty to both sides. The original bilateral trade becomes two trades: Party A vs. CCP, and CCP vs. Party B
3. **Margin Posting:** Both parties post initial margin (based on potential future exposure) and variation margin (daily mark-to-market)
4. **Daily Settlement:** Gains/losses are settled daily via variation margin transfers
5. **Default Management:** If a party defaults, the CCP uses margin and default fund resources to close out positions

**Major CCPs for USD swaps:** LCH (London) and CME (Chicago)

> **Desk Reality: Margin and Funding Costs**
>
> CCP clearing changes the economics of swaps. Before clearing, a swap between two strong credits might have required no collateral. Now:
> - Initial margin (typically 1-5% of notional for a 5-year swap) must be posted
> - Variation margin is exchanged daily
> - The cost of funding this margin affects swap pricing
>
> This "margin cost" is why cleared swaps trade differently from legacy bilateral swaps—it's embedded in the swap rate.

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

$$V = \sum_i E[\text{CF}_i] \cdot P_d(0, T_i)$$

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

**Sign interpretation:** If rates have risen since inception, forward rates exceed the fixed rate, making $PV_{\text{float}} > PV_{\text{fixed}}$. The payer swap has positive value—the payer locked in a below-market fixed rate and receives above-market floating payments.

### 25.5.3 Hull's Valuation Example

Hull provides a detailed worked example worth studying. Consider a swap with 1.2 years remaining, where the fixed rate is 3% paid semiannually and the floating rate is based on SOFR. With risk-free zero rates of 2.8%, 3.2%, and 3.4% for maturities of 0.2, 0.7, and 1.2 years respectively, Hull shows the valuation:

| Time (years) | Fixed CF | Floating CF | Net CF | Discount Factor | PV of Net CF |
|--------------|----------|-------------|--------|-----------------|--------------|
| 0.2 | -1.500 | +1.258 | -0.242 | 0.9944 | -0.241 |
| 0.7 | -1.500 | +1.694 | +0.194 | 0.9778 | +0.190 |
| 1.2 | -1.500 | +1.857 | +0.357 | 0.9600 | +0.343 |
| **Total** | | | | | **+0.292** |

The swap value of +\$0.292 million (per \$100 million notional) reflects that floating rates have risen above the fixed rate, benefiting the floating receiver.

> **Technique: Napkin Valuation**
>
> You don't always need a computer to know if you're winning or losing.
> *   **Formula**: $\text{PV} \approx \text{Notional} \times (\text{New Swap Rate} - \text{Your Fixed Rate}) \times \text{Duration}$
> *   **Example**:
>     *   You receive fixed 3.00% on \$100M for 5 years.
>     *   Market rates rise to 3.50%.
>     *   Duration $\approx$ 4.5 years.
>     *   $\text{PV} \approx \$100M \times (0.0350 - 0.0300) \times 4.5$
>     *   $\text{PV} \approx \$100M \times 0.0050 \times 4.5 = \$2.25M$
>     *   **Sign Check**: You receive 3.00%, market pays 3.50%. You are LOSING money. So Value $\approx -\$2.25M$.

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

Tuckman explains the economic logic: "Since no payment is exchanged at the initiation of the swap, the swap is only fair if its net value to each party equals zero, that is, $V_{\text{Fixed}} - V_{\text{Float}}$ equals zero."

This is not arbitrary—it reflects that both parties freely enter the agreement. If the swap had positive value to one party at inception, the other party would demand compensation (an upfront payment) to enter. Par swaps avoid this complexity.

### 25.6.3 The Single-Curve Par Rate Formula

In the traditional single-curve framework (where the same curve provides both forward rates and discount factors), the par swap rate takes an elegant closed form. Andersen and Piterbarg derive:

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

The 2008 financial crisis revealed that the single-curve framework was fundamentally flawed. Andersen and Piterbarg explain: "While the spread between the Fed funds rate and 3 month Libor rate used to be very small—in the order of a few basis points—after September 2007 it went up to as much as 275 basis points."

This spread explosion had profound implications: LIBOR was no longer a risk-free rate suitable for discounting. The rate at which you expect to receive floating payments (projection) differs from the rate at which you should discount those payments (discounting).

### 25.7.2 The Multi-Curve Framework

Modern swap valuation uses:

1. **Projection curve** ($P_k$): Generates forward rates for the floating index
2. **Discount curve** ($P_d$): Discounts all cashflows to present value

> **Analogy: The Projector and The Auditor**
>
> In the old days (Single Curve), one person guessed the future rates and valued the money. Now, these jobs are split.
>
> *   **The Projector (Blue Curve):** "I am looking at the risky bank market (LIBOR/Euribor/BSBY). I predict the floating rate in 5 years will be 4.00%."
> *   **The Auditor (Green Curve):** "I don't care about risky banks. I care about the *value of money* today. Since this trade is collateralized with cash, correct value is determined by the safe rate (OIS/SOFR). I will discount that 4.00% payment using the 3.00% risk-free rate."
>
> If you use the Projector's rate to discount (like we did pre-2008), you are over-discounting and mispricing the trade.

For a swap referencing index $k$, the floating forward rate is:

$$\boxed{L_k(0; T_i, T_{i+1}) = \frac{1}{\tau_i^k}\left(\frac{P_k(0, T_i)}{P_k(0, T_{i+1})} - 1\right)}$$

The swap value becomes:

$$V^k_{\text{swap}}(0) = \sum_j c \cdot \tau_j^{\text{fix}} \cdot P_d(0, T_j^{\text{fix}}) - \sum_i L_k(0; T_i, T_{i+1}) \cdot \tau_i^k \cdot P_d(0, T_{i+1})$$

Note carefully: forwards come from $P_k$, but discounting uses $P_d$.

### 25.7.3 OIS Discounting for Collateralized Swaps

What curve should we use for discounting? Andersen and Piterbarg argue that "most inter-dealer transactions are collateralized under the International Swaps and Derivatives Association (ISDA) Master Agreement, with the rate paid on collateral being the Fed funds rate (for USD; Eonia and Sonia for Euro and GBP)."

Since collateral earns the overnight rate, the appropriate discount rate for collateralized swaps is the OIS (Overnight Indexed Swap) curve. This leads to the modern convention:

- **USD collateralized swaps**: Discount with SOFR OIS curve, project with SOFR term curve
- **Legacy LIBOR swaps**: Discount with OIS, project with LIBOR curve (while LIBOR existed)

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

---

## 25.8 Worked Examples

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

**Setup:** Same curves as Example A, notional $N = \$100,000,000$, but the swap has a fixed rate of $c = 3.20\%$ (above par).

**Step 1: PV of floating leg**

Using the single-curve identity:
$$PV_{\text{float}} = N(1 - P(0,1)) = \$100,000,000 \times 0.0300 = \$3,000,000$$

**Step 2: PV of fixed leg**

$$PV_{\text{fixed}} = N \cdot c \cdot A^{\text{fix}}(0) = \$100,000,000 \times 0.032 \times 0.9775 = \$3,128,000$$

**Step 3: Swap NPV**

For the receiver (receive fixed, pay float):
$$V_{\text{recv}} = PV_{\text{fixed}} - PV_{\text{float}} = \$3,128,000 - \$3,000,000 = +\$128,000$$

For the payer (pay fixed, receive float):
$$V_{\text{payer}} = -\$128,000$$

**Alternative formula:** Using the off-market relationship:
$$V_{\text{recv}} = N(c - c_{\text{par}}) \cdot A^{\text{fix}}(0) = \$100,000,000 \times (0.032 - 0.03069) \times 0.9775 = \$128,000 \checkmark$$

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

**Example:** \$100 million notional, 5-year, with annuity factor 4.5:

$$\text{Upfront} = \$100M \times (0.0400 - 0.0350) \times 4.5 = \$2,250,000$$

The client pays \$2.25 million upfront and then pays 3.50% fixed over the life of the swap.

### 25.9.3 Swap Termination and Unwinds

Tuckman provides a termination example (Problem 18.3): "One year ago you paid fixed on \$10,000,000 of a 10-year interest rate swap at 5.75%. The nine-year par swap rate now is 6.25%, and the nine-year discount factor from the current swap rate curve is 0.572208."

To terminate:

**Step 1: Determine who owes whom**

You are paying 5.75% fixed but the current market rate is 6.25%. You locked in a below-market rate—your swap has positive value. The counterparty must pay you to terminate.

**Step 2: Calculate termination value**

$$V = N \times (c_{\text{market}} - c_{\text{old}}) \times A^{\text{fix}}$$

The annuity for a 9-year swap paying semiannually can be approximated from the discount factor, or computed directly. Using approximate annuity of 7.5:

$$V \approx \$10,000,000 \times (0.0625 - 0.0575) \times 7.5 = \$375,000$$

You *receive* approximately \$375,000 to terminate the swap.

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

Tuckman provides crucial intuition: "The key to valuing floaters is to start at the maturity date and work backward... on August 28, 2011, the ex-coupon value of the floating rate note must equal par. This valuation does not depend on the value of LIBOR on that date. Intuitively, as of the set date the floater earns the fair interest rate on three-month money for three months. And the value of a note earning the fair rate of interest over the single period of its life is simply par."

By backward induction, a floating-rate note is worth par on every reset date. Between reset dates, its value is:

$$P_{\text{floater}} = \frac{N + N \cdot L_{\text{current}} \cdot \tau}{1 + L_{\text{today}} \cdot \tau'}$$

where $L_{\text{current}}$ is the rate already set for the current period and $\tau'$ is the time remaining to payment.

### 25.10.2 Duration of a Floating-Rate Note

Tuckman observes: "Despite the fact that the maturity of the floater is 10 years, the price of the floating rate note depends only on the short-term rate and its effect on the present value of the next payment date... Hence, by the arguments of Chapter 6, its duration is approximately equal to the time to the next payment date, that is, 0.25 years."

This near-zero duration is precisely why floating-rate exposure is preferred by those wanting to avoid interest rate risk—and why the floating leg of a swap has minimal duration while the fixed leg has substantial duration.

---

## 25.11 The Swap as a Portfolio of FRAs

### 25.11.1 Decomposition into Period-by-Period Exposures

Hull emphasizes: "Each exchange of payment in a swap can be regarded as a forward rate agreement (FRA). Because it is nothing more than a portfolio of FRAs, an interest rate swap can also be valued by assuming that forward rates are realized."

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

A basis swap exchanges two floating rates—for example, 3-month SOFR vs. 1-month SOFR, or SOFR vs. a legacy rate like BSBY.

Andersen and Piterbarg describe these as essential instruments for curve construction: different floating tenors imply different forward rates due to credit and liquidity differences between tenors.

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

## 25.13 Practical Conventions

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

In practice, when fixed and floating payments fall on the same date, only the net amount is exchanged. This reduces operational risk and settlement flows. Tuckman notes: "The fixed and floating payments are netted with the result that Apple pays Citigroup \$200,000 on June 8, 2022" rather than separate payments of \$750,000 and \$550,000.

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
> **SOFR Fixing Confusion:**
> - Which SOFR rate applies? The rate for that day or the prior day?
> - Did the observation shift get applied correctly?
> - Is the 2-day lookback counting business days or calendar days?
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

A par swap has zero net present value at inception, but this does not mean that each cash flow exchange is worth zero. Hull explains: "The fixed rate in an interest rate swap is chosen so that the swap is worth zero initially. This does not mean that each cash flow exchange in the swap is worth zero initially. Instead, it means that the sum of the values of the cash flow exchanges is zero."

Consider a swap where the term structure is upward sloping and the fixed rate is being received while floating is paid. Forward rates increase with maturity, so early floating payments are expected to be lower than later ones. Since the fixed rate is constant, early exchanges favor the fixed receiver (fixed exceeds expected floating) while later exchanges favor the floating receiver (expected floating exceeds fixed).

### 25.15.2 Expected Value Path

This structure implies a predictable pattern for how the swap's value will evolve:

- **Fixed receiver, upward-sloping curve**: Early exchanges have positive value, later exchanges have negative value. As time passes and the positive-value exchanges are settled, the remaining swap is expected to have negative value.

- **Fixed payer, upward-sloping curve**: The opposite pattern. Early exchanges have negative value to the payer, later exchanges have positive value. The swap value is expected to become positive over time.

Hull notes: "These results can be used to determine whether the swap is expected to have a positive or negative value in the future."

**Important caveat:** These are expected values in a risk-neutral sense. Actual swap values depend on realized rate movements. If rates decline unexpectedly, a fixed-receiver swap could have positive value throughout its life despite the initial structure suggesting otherwise.

### 25.15.3 Implications for Credit Exposure

The expected value path has direct implications for counterparty risk:

- If your swap position is expected to become more valuable over time, your counterparty exposure increases—the potential loss from counterparty default grows.
- If your swap position is expected to become less valuable, your exposure decreases over time.

This asymmetry is one reason why credit risk management for swap portfolios requires careful modeling of potential future exposure (PFE) profiles, not just current mark-to-market values.

---

## 25.16 Credit Risk of Swap Agreements

### 25.16.1 Swap Credit Risk Is Much Lower Than Bond Credit Risk

A common misconception is to view swap credit risk as equivalent to bond credit risk. Tuckman clarifies: "It is not correct to view a swap as a position in a fixed rate note and an opposite position in a floating rate note when considering the implications of default."

The key differences:

1. **No principal at risk**: In a swap, notional principal is never exchanged. Default on a bond can result in loss of the entire principal; default on a swap can only cause loss of the swap's *value* at the time of default.

2. **Cross-default provisions**: Swap agreements typically allow one party to stop payments if the other party defaults. This limits the loss to the current value of the swap, not future cashflows.

3. **Value at risk much smaller than notional**: Tuckman provides an order-of-magnitude calculation: For a $100 million 5-year swap with net DV01 of 0.039, a 50 basis point rate move puts at most $1.95 million at risk—roughly 2% of notional. "Thus the default risk of a swap is in no way comparable to the default risk of a note."

### 25.16.2 Collateral Further Reduces Credit Risk

The CSA Framework governs collateral posting for most inter-dealer swaps:

- **Collateral currency**: Determines which overnight rate applies
- **Collateral rate**: Typically the overnight rate (SOFR for USD, SONIA for GBP)
- **Threshold and minimum transfer amounts**: Operational parameters

Tuckman explains the mechanism: "Swap agreements usually require cash margin, which earns interest at a short-term rate, to cover negative swap values. If the value of a swap is -$X to party A and +$X to party B, party A will have posted $X to party B as margin. If party A defaults, party B may keep the margin and terminate the swap."

This collateral mechanism means that the true credit exposure on collateralized swaps approaches zero in the limit of continuous margining.

### 25.16.3 Why Collateral Changes Discounting

When a swap is fully collateralized with daily margin calls, the credit risk is largely eliminated. The party posting collateral earns the overnight rate on that collateral. This cost/benefit must be reflected in pricing, which leads to discounting at the overnight rate rather than a credit-risky rate.

Andersen and Piterbarg note: "It is now generally accepted that the Libor rate is no longer a good proxy for a discounting rate on collateralized trades."

### 25.16.4 The Internal Inconsistency in Traditional Pricing

Tuckman identifies a subtle inconsistency in traditional swap pricing: "On the one hand it is assumed that there is no risk of counterparty default: The cash flows are assumed to be as specified in the swap agreement... On the other hand, all cash flows are discounted at LIBOR or swap rates (i.e., at rates containing the rolling credit risk of strong banks)."

The resolution: "The market convention must be viewed as a compromise. The default risk of a swap agreement is perceived small enough to assume that promised cash flows are received and to discount these cash flows at a rate appropriate for a very strong credit, namely the rolling credit of financially sound banks."

Modern OIS discounting resolves much of this inconsistency—collateralized swaps are discounted at rates reflecting minimal credit risk.

---

## 25.17 Major Uses of Interest Rate Swaps

Interest rate swaps are used for three primary purposes, as Tuckman describes:

### 25.17.1 Creating Synthetic Floating-Rate Funding

"While many corporations prefer floating rate debt, the market for long-term floating rate notes in the U.S. is very small." Corporations wanting floating-rate exposure have two options:

1. **Roll short-term debt**: Issue and roll over commercial paper or other short-term obligations. This works for highly-rated issuers but creates *liquidity risk*—the risk that credit deterioration prevents refinancing.

2. **Issue fixed + enter swap**: Issue fixed-rate bonds and receive fixed / pay floating in a swap. The fixed leg payments offset, leaving synthetic floating-rate funding. This avoids liquidity risk while achieving the desired rate profile.

### 25.17.2 The Comparative Advantage Argument

Hull presents a classic motivation for swaps through the comparative advantage argument. Consider two companies:

| Company | Fixed Rate | Floating Rate |
|---------|-----------|---------------|
| AAACorp | 4.0% | SOFR + 0.3% |
| BBBCorp | 5.2% | SOFR + 1.0% |

AAACorp has absolute advantage in both markets, but BBBCorp has *comparative* advantage in the floating market (its spread over AAACorp is smaller: 70bp in floating vs. 120bp in fixed).

If AAACorp wants floating and BBBCorp wants fixed:
- AAACorp borrows fixed at 4.0%, enters swap to receive 4.35%, pay SOFR → Net: SOFR - 0.35%
- BBBCorp borrows floating at SOFR + 1.0%, enters swap to pay 4.35%, receive SOFR → Net: 5.35%

Both are better off: AAACorp pays SOFR - 0.35% (vs. SOFR + 0.3%) and BBBCorp pays 5.35% (vs. 5.2%). Total gain: 50bp, split between them.

> **Practitioner Note:** Hull cautions that the comparative advantage argument is "open to question" because it assumes the spread differential reflects only credit risk, when liquidity and other factors also contribute. Nevertheless, the argument provides useful intuition for why swaps exist.

### 25.17.3 Mortgage Market Hedging

"Perhaps the largest use of swaps is related to the mortgage market." Government-sponsored enterprises (GSEs) like Fannie Mae and Freddie Mac sell fixed-rate debt and buy mortgages. As the interest rate risk of their portfolios changes with rates and curve shape, they use swaps to manage duration.

Mortgage servicers, who collect fees for processing mortgage payments, also face interest rate exposure and hedge with swaps. Chapter 27 discusses the interaction between mortgage hedging and swap spreads.

### 25.17.4 Hedging Future Debt Issuance

A corporation planning to issue bonds can hedge against rising rates: "To hedge the future issuance of 10-year debt, for example, it can pay fixed on a 10-year swap at the time of its decision and unwind the swap at the time of its bond sale."

**Advantage over Treasury hedging**: Swaps hedge not only the general level of rates but also, to some extent, changes in credit spreads. Tuckman notes: "The magnitude of the correlation between swap rates and credit spreads is a matter of empirical debate." However, substantial basis risk remains because swap rates reflect banking credit specifically rather than the particular corporate's credit.

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
7. CCP clearing is mandatory for standard swaps between financial institutions
8. SOFR swaps use compounded overnight rates with 2-day lookback conventions
9. Off-market swaps require upfront payments equal to NPV

**Swap dynamics and credit:**

10. While a par swap has zero net value at inception, individual cash flow exchanges may have positive or negative value—only the sum is zero
11. The expected path of swap value over time depends on the term structure shape and which leg you receive
12. Swap credit risk is much lower than bond credit risk—loss is limited to the swap's current value, not principal
13. Collateralization through CSA agreements further reduces credit exposure to near zero

**Major uses:** Corporations use swaps to create synthetic floating-rate funding, hedge future debt issuance, and manage duration. The mortgage market is the largest source of swap hedging demand.

The separation of projection and discounting curves is not academic pedantry—it reflects real economic differences between the rate you expect to receive and the rate at which you should discount. Getting this right is essential for accurate pricing and risk management.

---

## Key Concepts Summary

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

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $N$ | Notional principal |
| $c$ | Fixed swap rate (annualized) |
| $c_{\text{par}}$ | Par swap rate |
| $\tau_j^{\text{fix}}$ | Fixed leg accrual factor for period $j$ |
| $\tau_i^{\text{flt}}$ | Floating leg accrual factor for period $i$ |
| $P_d(0, T)$ | Discount factor from discount curve |
| $P_k(0, T)$ | Discount factor from projection curve for index $k$ |
| $L_k(0; T_i, T_{i+1})$ | Forward rate for index $k$ over $[T_i, T_{i+1}]$ |
| $A^{\text{fix}}(0)$ | Fixed-leg annuity |
| $R_{\text{compound}}$ | Compounded SOFR rate for a period |

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
| 23 | What is the standard SOFR observation shift? | 2 business days lookback |
| 24 | Why is SOFR backward-looking vs LIBOR forward-looking? | SOFR compounds actual observed daily rates; LIBOR was a term rate known at period start |
| 25 | What is a CCP? | Central counterparty—stands between both parties to a cleared swap |
| 26 | For a par swap, is each cash flow exchange worth zero? | No—sum of exchange values is zero, but individual exchanges have positive/negative values |
| 27 | If term structure is upward sloping and you receive fixed, how does swap value evolve? | Expected to become negative over time as positive-value early exchanges settle |
| 28 | Why is swap credit risk much lower than bond credit risk? | Notional is never exchanged; loss limited to swap value at default, not principal |
| 29 | How does collateral posting reduce swap credit risk? | Party with negative value posts margin; default loss covered by margin |
| 30 | What creates synthetic floating-rate funding? | Issue fixed-rate debt + receive fixed / pay float in a swap |

---

## Mini Problem Set

**Q1.** A 1-year spot-start single-curve swap has $P(0,1) = 0.965$. What is $PV_{\text{float}}/N$?

**Q2.** Using Q1, suppose the fixed leg is annual with $\tau = 1$. What is the par fixed rate?

**Q3.** Same as Q2 but fixed leg is semiannual with $P(0, 0.5) = 0.982$. Compute the annuity and $c_{\text{par}}$.

**Q4.** For Q3, if the swap fixed rate is 20bp above par, compute the receiver PV per unit notional.

**Q5.** A payer swap has $N = \$50M$, $A^{\text{fix}} = 4.2$, and $c$ is 10bp above par. Approximate PV.

**Q6.** Write the general multi-curve par rate formula.

**Q7.** Explain why $PV_{\text{float}} = N(1 - P_d(0,T))$ fails in multi-curve.

**Q8.** For a payer swap, which direction does PV move if forward rates increase (holding discounting fixed)?

**Q9.** The term structure is upward-sloping and you enter a 5-year receiver swap at par. Should you expect your counterparty credit exposure to increase or decrease over the first few years? Explain.

**Q10.** A $100 million swap has net DV01 of 0.045. If rates move 75bp since inception, approximately what is the maximum credit exposure?

**Q11.** The 10-year Treasury yields 4.50% and 10-year swap spreads are quoted at -5/+2 basis points. What are the bid and offer swap rates?

**Q12.** Calculate the approximate compounded SOFR payment on \$25 million notional over a 92-day period, assuming constant daily SOFR of 5.00%.

---

### Solution Sketches

**A1.** $1 - P(0,1) = 1 - 0.965 = 0.035$

**A2.** $A = P(0,1) = 0.965$; $c_{\text{par}} = 0.035/0.965 = 3.627\%$

**A3.** $A = 0.5(0.982) + 0.5(0.965) = 0.9735$; $c_{\text{par}} = 0.035/0.9735 = 3.595\%$

**A4.** $V_{\text{recv}}/N = (c - c_{\text{par}}) \cdot A = 0.002 \times 0.9735 = 0.00195$

**A5.** $V_{\text{payer}} = -N(c - c_{\text{par}}) \cdot A = -\$50M \times 0.001 \times 4.2 = -\$210,000$

**A6.** $c_{\text{par}} = \frac{\sum_i \tau_i^{\text{flt}} L_k(0; T_i, T_{i+1}) P_d(0, T_{i+1})}{\sum_j \tau_j^{\text{fix}} P_d(0, T_j^{\text{fix}})}$

**A7.** The identity relies on forwards and discounting using the same curve. In multi-curve, forwards come from $P_k$ while discounting uses $P_d$—the telescoping cancellation doesn't hold.

**A8.** Payer PV increases—higher forwards mean higher expected floating receipts.

**A9.** Decrease. With upward-sloping curve, early exchanges favor you (receive fixed > expected floating). As these positive-value exchanges settle, remaining swap expected to have negative value to you—counterparty exposure declines.

**A10.** $\$100M \times (0.045/100) \times 75 = \$3.375$ million. This is the approximate mark-to-market value at risk—much smaller than the $100M notional.

**A11.** Bid: 4.50% - 0.05% = 4.45%; Offer: 4.50% + 0.02% = 4.52%

**A12.** Compounded factor: $(1 + 0.05/360)^{92} = 1.01289$. Period rate: $(360/92) \times 0.01289 = 5.044\%$. Payment: $\$25M \times 0.05044 \times (92/360) = \$322,256$

---

## Source Map

### (A) Book-Verified Facts

| Fact | Source |
|------|--------|
| Swap definition and mechanics | Hull Ch 7 |
| Swap = long fixed bond, short floating bond | Hull Ch 7, Tuckman Ch 18 |
| Set in advance, pay in arrears structure | Hull Ch 7, Andersen Vol 1 |
| Fixed 30/360, floating ACT/360 conventions | Hull Ch 7, Tuckman Ch 18 |
| Single-curve par swap rate formula | Andersen Vol 1 (4.10), Tuckman Ch 18 |
| Multi-curve framework with universal discount + index curves | Andersen Vol 1 Ch 6 (6.47)-(6.48) |
| OIS discounting for collateralized swaps | Andersen Vol 1 Ch 6, Hull Ch 9 |
| Floating-rate note worth par on reset dates | Tuckman Ch 18 |
| Swap as portfolio of FRAs | Hull Ch 7 |
| Par swap value = 0 but individual exchanges ≠ 0 | Hull Ch 7.7 |
| Expected swap value path based on term structure | Hull Ch 7.7 |
| Swap credit risk much lower than bond credit risk | Tuckman Ch 18 |
| Loss limited to swap value, not principal | Tuckman Ch 18 |
| Collateral/margin mechanism for swaps | Tuckman Ch 18 |
| Internal inconsistency in LIBOR discounting | Tuckman Ch 18 |
| Major uses: synthetic floating funding, mortgage hedging | Tuckman Ch 18 |
| Swaps vs Treasuries for hedging debt issuance | Tuckman Ch 18 |
| CCP clearing mechanics, Dodd-Frank requirements | Hull Ch 2 |
| Swap spreads history and Treasury scarcity | Tuckman Ch 18 |
| Comparative advantage argument | Hull Ch 7.5 |
| SOFR compounding formula | Hull Ch 4 |

### (B) Claude-Extended Content

| Content | Context |
|---------|---------|
| Desk language: "5s are 10.5 out", "paying/receiving" | Extended from trading floor conventions |
| SOFR observation shift (2-day lookback) conventions | Extended from current market practice |
| SEF execution (RFQ, CLOB) details | Extended from post-Dodd-Frank market structure |
| Common P&L break sources | Extended from operational practice |
| Sanity checks for swap operations | Extended from risk management practice |
| Term SOFR vs overnight SOFR distinction | Extended from current market practice |

### (C) Reasoned Inference

| Inference | Derivation |
|-----------|------------|
| Par rate = $PV_{\text{float}}$/Annuity | Set $V = 0$, solve for $c$ |
| Single-curve telescoping to $1 - P(0,T)$ | Term-by-term cancellation |
| Off-market PV = $(c - c_{\text{par}}) \cdot N \cdot A$ | Linear algebra from leg PV formulas |
| Upfront payment = swap NPV for off-market swaps | Fair value exchange principle |

### (D) Flagged Uncertainties

| Topic | Uncertainty |
|-------|-------------|
| Exact SOFR conventions (lockout vs lookback specifics) | I'm not sure—conventions continue to evolve and may vary by counterparty |
| Desk-specific conventions (payment lags, stubs, business days) | I'm not sure—varies by currency and documentation |
| Exact CCP initial margin percentages | I'm not sure—varies by CCP, portfolio, and market conditions |
| Exact bump methodology for curve sensitivities | I'm not sure—desk convention; examples use explicit approximations |

---

*Chapter 25 complete.*
