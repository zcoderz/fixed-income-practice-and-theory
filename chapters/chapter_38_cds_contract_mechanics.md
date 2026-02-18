# Chapter 38: CDS Contract Mechanics

---

## Introduction

Before you can price a credit default swap, hedge a corporate bond position, or explain P&L after a credit event, you must understand exactly what cashflows move between the two parties and when. A CDS may function economically like credit insurance, but it is contractually a derivative with precise mechanics that differ from insurance in critical ways.

Consider a portfolio manager holding USD 50 million of XYZ Corp bonds who wants to hedge the default risk. She calls her dealer and "buys protection" on XYZ via a 5-year CDS at 150 basis points. What has she agreed to? She will pay quarterly premiums based on that spread, but the exact payment dates follow CDS-specific conventions (the 20th of March, June, September, December—not the bond's coupon dates). If XYZ experiences a credit event mid-quarter, her premium payments stop, but she owes accrued premium up to the event date. The protection payment she receives depends on whether the contract settles physically or via an auction-determined cash price.

Getting any of these details wrong—the day count, the effective date, the accrued premium at default—creates P&L breaks, failed reconciliations, and hedging errors. The CDS market has standardized many conventions, but the standardization itself requires understanding: why T+1 calendar effective dates? Why do roll dates fall on the 20th rather than month-end? Why does the market now trade with fixed coupons plus upfront payments rather than running spreads alone?

This chapter covers the mechanics of CDS contracts and introduces the **Risky PV01 (RPV01)**—the fundamental building block for CDS valuation. We establish the premium leg and protection leg cashflows, work through date conventions and settlement methods, derive the RPV01 formula that incorporates survival probabilities, and explain the upfront-plus-coupon trading regime including distressed credit conventions. We also cover mark-to-market calculation and position unwinding mechanics.

Prerequisites: [Chapter 36 — Survival Probabilities and Hazard Rates](chapters/chapter_36_survival_probabilities_hazard_rates.md), [Chapter 37 — Cash Credit — Risky Bonds, Credit Spreads, and CS01](chapters/chapter_37_cash_credit_risky_bonds_spreads_cs01.md)  
Follow-on: [Chapter 39 — CDS Credit Events and Settlement](chapters/chapter_39_cds_credit_events_settlement.md), [Chapter 40 — The CDS Auction Process](chapters/chapter_40_cds_auction_process.md), [Chapter 42 — Bootstrapping a CDS Survival Curve](chapters/chapter_42_bootstrapping_cds_survival_curve.md), [Chapter 43 — Risks in CDS and Hedging Strategies](chapters/chapter_43_cds_risks_hedging.md)

## Learning Objectives
- Translate a CDS quote into the contractual cashflows on the premium leg and protection leg.
- Build a standard premium schedule from effective date to maturity and recognize stubs/roll conventions.
- Compute and interpret accrued premium at default (including the common “mid-period” approximation).
- Define and use `RPV01` and the basic CDS MTM identity with explicit units and sign.
- Explain the fixed-coupon + upfront trading convention and compute upfront from a quoted spread and `RPV01`.

---

## 38.1 The Credit Default Swap: Definition and Economic Logic

A credit default swap (CDS) is a bilateral contract that transfers the credit risk of a reference entity for a specified period. The protection buyer pays periodic premiums; if a credit event occurs during the protection period, the protection seller makes a contingent payment intended to compensate for loss-from-par on eligible obligations.

The contract has two parties with opposite exposures:

**The protection buyer** pays periodic premiums and receives a contingent payment if a credit event occurs. Economically, buying protection is equivalent to shorting the credit—the buyer profits if the reference entity's creditworthiness deteriorates or it defaults.

**The protection seller** receives premiums and makes a contingent payment after a credit event. Selling protection is equivalent to being long credit risk—the seller profits if the entity survives and spreads tighten.

A critical terminology point: "buying a CDS" in market language typically means buying protection, which is short credit risk. This is the opposite of cash bond conventions where "buying" means going long. Confusion on this point has caused expensive booking errors.

> **Analogy: The Insurance Policy**
>
> A CDS is remarkably similar to Car Insurance.
>
> 1.  **Premium Leg**: You pay a periodic fee (premium) to the protection seller.
> 2.  **Protection Leg**: If the insured event occurs (a credit event), the seller compensates the buyer for the loss-from-par.
> 3.  **The Difference**: You do not need to own the underlying bond/loan to buy CDS protection.
>
> **Visual: CDS Cashflows**
> Buyer $\xrightarrow{\text{Premiums}}$ Seller
> Seller $\xrightarrow{\text{Protection (if Default)}}$ Buyer

---

## 38.2 The Premium Leg: Periodic Payments from Buyer to Seller

The premium leg is the series of payments made by the protection buyer to the protection seller to pay for the protection being provided. The premium leg payments are typically made quarterly according to an Actual 360 basis. The payments terminate at the contract maturity or immediately following a credit event (with accrued premium still owed up to the credit event time).

### 38.2.1 The Premium Payment Formula

For a notional $N$, running spread $s$ (in decimal form, so 100 bp = 0.01), and accrual fraction $\Delta$, the premium payment at date $t_n$ is:

$$\boxed{\text{PremiumPay}(t_n) = N \cdot s \cdot \Delta(t_{n-1}, t_n)}$$

where the accrual fraction under ACT/360 is:

$$\Delta(t_{n-1}, t_n) = \frac{\text{DayDiff}(t_{n-1}, t_n)}{360}$$

**Unit check:** $N$ (dollars) × $s$ (1/year) × $\Delta$ (years) = dollars. The spread is quoted per annum, and the accrual fraction converts the annual rate to the period's portion.

**Sign check:** If you write cashflows from the position-holder perspective (positive = received), then premium payments are **negative** for the protection buyer and **positive** for the protection seller:

$$
PremiumCashflow_{\mathrm{buyer}}(t_n)=-N s \Delta(t_{n-1},t_n),\qquad
PremiumCashflow_{\mathrm{seller}}(t_n)=+N s \Delta(t_{n-1},t_n).
$$

**Example:** Consider a USD 10 million notional CDS with a 35 bp running spread. For a 91-day quarter:

$$\text{PremiumPay} = 10{,}000{,}000 \times 0.0035 \times \frac{91}{360} = 10{,}000{,}000 \times 0.0035 \times 0.2528 = USD 8{,}847$$

### 38.2.2 Payment Schedule Construction: The Step-by-Step Algorithm

Standard CDS premium schedules are built around quarterly **standard dates**. Usually, contracts mature on the following standard dates: March 20, June 20, September 20, and December 20.

These are sometimes informally called “CDS IMM dates,” but note the distinction: the CDS market always chooses the 20th date of the month while the official IMM date is the 3rd Wednesday of the month.

**Step 1 — Determine the Maturity Date:**
Choose the scheduled termination date as the first standard date that falls **at least** $T$ years after the effective date.

For a 5-year CDS with effective date 20 January 2026, the maturity date is the first of 20 Mar/Jun/Sep/Dec that is at least 5 years after effective date—namely, 20 March 2031.

**Step 2 — Generate Payment Dates by Stepping Back:**
Starting from the maturity date, step backward in three-month increments along the standard-date grid until you reach the first coupon date after the effective date.

Working backwards from 20 March 2031:
- 20 March 2031 (maturity)
- 20 December 2030
- 20 September 2030
- ... continue stepping back ...
- 20 March 2026 (first payment date after effective)

**Step 3 — Adjust for Business Days:**
Adjust dates that fall on weekends/holidays using the contract’s business-day convention and holiday calendar (calendar choice is contract-specific).

**Step 4 — Calculate Premium Payments:**
Compute accrual fractions using ACT/360 and apply the premium payment formula from Section 38.2.1.

> **Desk Reality: Schedule Mismatches as P&L Break Source**
>
> One of the most common causes of P&L breaks in CDS trading comes from schedule mismatches between front office and operations systems. The culprits:
>
> 1. **Wrong holiday calendar**: Using US holidays when the contract specifies TARGET (European)
> 2. **Miscounting stub periods**: The first coupon is almost always a short stub—systems that assume 90-day first periods are wrong
> 3. **Wrong business-day convention**: Rolling dates differently than the confirmation specifies
> 4. **IMM date confusion**: Using the 3rd Wednesday instead of the 20th
>
> When your daily P&L shows an unexplained break on a coupon date, check the schedule first.

**Worked Example: Building a Complete Schedule**

Trade date: Wednesday, 15 January 2026
Effective date: Thursday, 16 January 2026 (T+1 calendar)
Notional: USD 10 million
Spread: 35 bp
Maturity: 5 years → First standard date ≥ 16 January 2031 → 20 March 2031

Weekend/holiday adjustments create small but real differences in accrual factors across quarters. In addition, the first accrual period is often a short front stub because most trades do not start exactly on a standard date.

| Flow Date (Adjusted) | Day Count Fraction | Payment (USD 10mm @ 35bp) |
|---------------------|-------------------|------------------------|
| 20 Mar 2026 (Fri) | 63/360 = 0.1750 | USD 6,125 |
| 22 Jun 2026 (Mon) | 94/360 = 0.2611 | USD 9,139 |
| 21 Sep 2026 (Mon) | 91/360 = 0.2528 | USD 8,847 |
| 21 Dec 2026 (Mon) | 91/360 = 0.2528 | USD 8,847 |
| ... | ... | ... |
| 20 Mar 2031 (Thu) | ~90/360 | ~USD 8,750 |

**Key observation:** The first payment (20 Mar 2026) covers only 63 days—a **short front stub**. This is normal for CDS trades that don't start exactly on a roll date.

### 38.2.3 Stub Period Mechanics

The first premium payment often involves a **stub period**—a period shorter or longer than the standard 91–92 days.

**Front stub (short first coupon):** Trade date falls between roll dates, so the first accrual period is shorter than 3 months. This is the most common case.

**Example:** Trade on 15 January means first payment on 20 March covers only 63 days (16 Jan to 20 Mar), not the standard ~90 days. The first payment is correspondingly smaller.

**Why this matters for P&L:**
If a system incorrectly assumes a full 90-day first period, the first premium payment will be overstated by approximately:

$$\frac{90 - 63}{360} \times N \times s = \frac{27}{360} \times 10{,}000{,}000 \times 0.0035 = USD 2{,}625$$

On a USD 10 million trade, that's a USD 2,625 error on day one—enough to trigger a break investigation.

### 38.2.4 Effective Date: T+1 Calendar

For the worked examples in this chapter, we will use an effective date equal to the calendar day after the trade date (**T+1 calendar**). The confirmation governs the exact effective-date rule and any business-day adjustments.

**Expand (why this exists):** Credit events (e.g., bankruptcy filings) can occur on weekends/holidays. If protection always started on a business day, a trade executed right before a weekend could have a non-trivial gap in coverage.

> **Desk Reality: The Weekend Coverage Question**
>
> When a trade is executed on Friday, the effective date is Saturday. If the reference entity files for bankruptcy on Sunday, is the protection buyer covered?
>
> Under a T+1 calendar effective date, protection can start on Saturday. A weekend credit event after the effective date is inside the protection period; under a T+1 business-day rule, the same event could fall into a coverage gap.
>
> Middle office staff marking positions should verify that systems correctly handle weekend effective dates. A common error is systems that "helpfully" adjust weekend dates to Monday.

---

## 38.3 The Protection Leg: Contingent Payment After Credit Event

The protection leg is the contingent payment from seller to buyer following a credit event during the protection period. Economically, it is intended to compensate the buyer for the loss-from-par on eligible deliverable obligations.

### 38.3.1 Protection Payment Formula

If the recovery price (the post-default market value of the reference obligation) is $R$ as a fraction of par, the protection payment is:

$$\boxed{\text{ProtPay} = N(1 - R)}$$

**Bounds check:** For $0 \leq R \leq 1$, the protection payment lies between 0 (full recovery) and $N$ (zero recovery).

**Example:** On USD 10 million notional with recovery of 40%:

$$\text{ProtPay} = 10{,}000{,}000 \times (1 - 0.40) = USD 6{,}000{,}000$$

The protection buyer receives USD 6 million, compensating for the loss from par on the reference obligations.

### 38.3.2 Settlement Methods

Two common settlement methods are:

**Physical settlement:** The protection buyer delivers a specified face value of deliverable obligations to the protection seller, and the seller pays that face value in cash.

If the buyer can source bonds trading at 38 cents on the dollar, they pay USD 3.8 million to acquire USD 10 million face value, deliver these bonds, and receive USD 10 million from the seller—a net gain of USD 6.2 million.

**Cash settlement:** The protection seller pays the protection buyer the face value minus an auction-determined recovery price. If the recovery price is 38%, the seller pays $10{,}000{,}000 \times (1 - 0.38) = USD 6{,}200{,}000$.

Both methods should produce the same economic result when the cash settlement price equals the market value of deliverable obligations. In many contracts/descriptions, cash settlement uses an ISDA-organized auction process to determine the recovery price.

**Why auctions matter:** Physical settlement can become operationally difficult when protection notional is large relative to available deliverables. An auction provides a single recovery price used for cash settlement across contracts.

---

## 38.4 Accrued Premium at Default

When a credit event occurs between payment dates, the protection buyer owes accrued premium for the portion of the period during which protection was in force. This is a real cashflow, not just an accounting convention.

### 38.4.1 The Accrued Premium Formula

If default occurs at time $\tau$ with $t_{n-1} \lt  \tau \leq t_n$:

$$\boxed{\text{AccruedPrem}(\tau) = N \cdot s \cdot \frac{\text{DayDiff}(t_{n-1}, \tau)}{360}}$$

**Example:** Using the schedule from Section 38.2.2, suppose default occurs on 10 August 2026. The previous payment date was 22 June 2026, so 49 days have elapsed:

$$\text{AccruedPrem} = 10{,}000{,}000 \times 0.0035 \times \frac{49}{360} = USD 4{,}764$$

**Sanity check:** The accrued amount (USD 4,764) is less than a full quarter's premium (USD 8,847), as expected since default occurred roughly halfway through the period.

**Check (rule of thumb):** If default time within the period is “roughly uniform” for intuition, then accrued premium at default is often on the order of **half** of the full-period premium: $AccruedPrem(\tau)\approx \frac{1}{2} N s \Delta(t_{n-1},t_n)$.
This is the same mid-period-default intuition used later when approximating the accrued-at-default contribution to `RPV01`.

### 38.4.2 Premium Payments Stop at Default

After a credit event:
- All future scheduled payments are cancelled
- The accrued premium up to $\tau$ is owed (this is the final premium cashflow)
- The protection payment is triggered

This differs from a bond, where coupons may continue accruing through a grace period. In CDS, the credit event is a hard stop.

> **Pitfall — Premium accrual on default:** Dropping the accrued-premium cashflow when valuing or settling a CDS.
> **Why it matters:** It creates a real cash break on default (or on unwind near a coupon date) even if the protection payment is correct.
> **Quick check:** If $\tau$ is between $t_{n-1}$ and $t_n$, accrued premium must be between 0 and the full-period premium $N s \Delta(t_{n-1},t_n)$.

### 38.4.3 Effect of Accrued Premium on Breakeven Spread

Including accrued premium at default **lowers** the breakeven spread. Intuitively, the protection seller receives some premium even in default scenarios (the accrued amount), which slightly reduces the par spread required to make the contract value zero.

One common approximation for the spread impact is:

$$\boxed{S(\text{without accrued}) - S(\text{with accrued}) \approx \frac{S^2}{2(1-R)f}}$$

where $f$ is the payment frequency (4 for quarterly).

**Numerical example:** For $S = 500$ bp = 0.05, $R = 40\%$, $f = 4$:

$$\text{Effect} \approx \frac{(0.05)^2}{2 \times 0.60 \times 4} = \frac{0.0025}{4.8} = 0.00052 = 5.2\text{ bp}$$

The adjustment is quadratic in $S$: it is tiny at tight spreads and grows quickly as spreads widen.

> **Practitioner Note:** Because the adjustment scales roughly like $S^2$, it is tiny for tight spreads and can become several bp (or more) for very wide/distressed spreads. Production systems include accrued-at-default explicitly; quick estimates can use the approximation above.

---

## 38.5 The Risky PV01: Valuing the Premium Leg

The **Risky PV01 (RPV01)** is the present value of paying or receiving **1 bp per annum** of running premium on the premium leg until maturity or default (including the accrued-at-default piece). It is the workhorse scaling for CDS upfront, mark-to-market, and spread risk.

To avoid unit mistakes, it helps to separate two related objects:

- **Risky annuity** $A(t,T)$: PV factor per USD 1 notional per **1.00** of running spread (units: years).
- **RPV01**: PV factor per stated notional per **1 bp** of running spread (units: currency per bp).

They are connected by:

$$\boxed{\text{RPV01}(t,T) = N \times 10^{-4} \times A(t,T)}$$

where $1\text{ bp}=10^{-4}$ and $N$ is the CDS notional.

**Check (order of magnitude):** For a 5Y quarterly CDS, the risky annuity $A(t,T)$ is typically a few “years” (less than the risk-free annuity because survival $Q$ downweights later coupons). So for $N=USD 10\text{mm}$, you expect `RPV01` to be on the order of $10{,}000{,}000\times 10^{-4}\times A \approx 1{,}000\times A$, i.e., a few thousand dollars per bp—not USD 100k/bp and not USD 10/bp.

### 38.5.1 Premium Leg Present Value

Using the risky annuity $A(t,T)$, the present value of the premium leg (excluding sign) for contractual running spread $S_0$ (decimal per annum) and notional $N$ is:

$$\boxed{\text{Premium Leg PV} = N \cdot S_0 \cdot A(t,T)}$$

Equivalently, if you express spreads in bp and use `RPV01` in currency per bp:

$$\boxed{\text{Premium Leg PV} = S_{0,\text{bp}} \cdot \text{RPV01}(t,T)}$$

**Check (units):** $(\text{bp})\times(\text{currency/bp})=\text{currency}$.

### 38.5.2 Deriving the Risky Annuity $A(t,T)$

Start from the building blocks from Chapter 36.

**Risky discount factor:** The present value of USD 1 paid at time $t_n$, conditional on survival to $t_n$, under independence of interest rates and default:

$$\hat{Z}(t, t_n) = Z(t, t_n) \cdot Q(t, t_n)$$

where `Z(t, t_n)` is the risk-free discount factor and `Q(t, t_n)` is the survival probability.

**Component 1 — Scheduled Payments:**

For scheduled premium payments made at dates $t_1, \ldots, t_N$ if the credit survives:

$$A_{\text{sched}}(t,T) = \sum_{n=1}^{N} \Delta(t_{n-1}, t_n) \cdot Z(t, t_n) \cdot Q(t, t_n)$$

**Component 2 — Accrued at Default:**

If default occurs between $t_{n-1}$ and $t_n$, accrued premium from $t_{n-1}$ to the default time $\tau$ must be paid. The exact PV involves an integral over default time, but a practical approximation is:

$$A_{\text{accr}}(t,T) \approx \frac{1}{2} \sum_{n=1}^{N} \Delta(t_{n-1}, t_n) \cdot Z(t, t_n) \cdot (Q(t, t_{n-1}) - Q(t, t_n))$$

The intuition: on average, default occurs mid-period, so the expected accrued is half the full period's premium.

**The Full Risky Annuity Formula:**

Combining both components and applying the mid-period-default approximation yields:

$$\boxed{A(t, T) = \frac{1}{2} \sum_{n=1}^{N} \Delta(t_{n-1}, t_n) \cdot Z(t, t_n) \cdot (Q(t, t_{n-1}) + Q(t, t_n))}$$

This elegant formula averages adjacent survival probabilities for each period, capturing both the scheduled payment (weighted by `Q(t, t_n)`) and the expected accrued at default (difference term).

**Sanity checks:**
- If $Q(t, t_n) = 1$ for all $n$ (no default risk), $A$ reduces to the risk-free annuity
- If $Q(t, t_n) \to 0$ rapidly, $A$ is small (payments unlikely to be received)
- Units: $\Delta$ (years) × $Z$ (dimensionless) × $Q$ (dimensionless) = years

### 38.5.3 Worked Example: Computing $A$ and RPV01

**Setup:**
- Valuation date: $t = 0$
- Maturity: $T = 2$ years (8 quarterly periods)
- Notional: USD 10,000,000
- Flat interest rate: 5% continuously compounded
- Flat hazard rate: $\lambda = 2\%$ per annum

**Step 1: Compute discount factors and survival probabilities**

| Period | $t_n$ (years) | $Z(0, t_n) = e^{-0.05 t_n}$ | $Q(0, t_n) = e^{-0.02 t_n}$ |
|--------|--------------|---------------------------|---------------------------|
| 1 | 0.25 | 0.9876 | 0.9950 |
| 2 | 0.50 | 0.9753 | 0.9900 |
| 3 | 0.75 | 0.9632 | 0.9851 |
| 4 | 1.00 | 0.9512 | 0.9802 |
| 5 | 1.25 | 0.9394 | 0.9753 |
| 6 | 1.50 | 0.9277 | 0.9704 |
| 7 | 1.75 | 0.9162 | 0.9656 |
| 8 | 2.00 | 0.9048 | 0.9608 |

**Step 2: Apply the risky annuity formula**

Using $\Delta = 0.25$ for each quarterly period and $Q(0, t_0) = 1$:

$$A(0,2) = \frac{1}{2} \sum_{n=1}^{8} 0.25 \times Z(0, t_n) \times (Q(0, t_{n-1}) + Q(0, t_n))$$

| $n$ | $\Delta \times Z$ | $Q_{n-1} + Q_n$ | Contribution |
|-----|-------------------|-----------------|--------------|
| 1 | 0.2469 | 1.9950 | 0.2462 |
| 2 | 0.2438 | 1.9850 | 0.2420 |
| 3 | 0.2408 | 1.9751 | 0.2378 |
| 4 | 0.2378 | 1.9653 | 0.2337 |
| 5 | 0.2349 | 1.9555 | 0.2296 |
| 6 | 0.2319 | 1.9458 | 0.2256 |
| 7 | 0.2291 | 1.9361 | 0.2218 |
| 8 | 0.2262 | 1.9264 | 0.2179 |
| **Sum** | | | **1.8546** |

$$A(0,2) = \frac{1}{2} \times 1.8546 = 0.9273 \text{ years}$$

**Step 3: Convert to RPV01 (currency per bp)**

$$\text{RPV01}(0,2) = N \times 10^{-4} \times A(0,2) = 10{,}000{,}000 \times 10^{-4} \times 0.9273 = USD 927.30 \text{ per bp}.$$

**Interpretation:** The risky annuity is 0.9273 years. The corresponding `RPV01` for USD 10mm notional is USD 927.30 per bp. The risk-free annuity would be higher; the difference reflects survival-weighting reducing expected payments.

> **Desk Reality: RPV01 vs “annuity”**
>
> Risk systems typically report `RPV01` in **currency per bp** for a stated notional. Quant models often work with the underlying annuity $A(t,T)$ in **years**. Always do a unit check before multiplying.
>
> A quick conversion is `RPV01 ≈ N × A × 10^{-4}`.

### 38.5.4 Spread Sensitivity (CS01 / “Credit DV01”)

`RPV01` is also the key scaling for first-order PV sensitivity to the quoted CDS par spread.

**Bump object:** the quoted par spread $S(t,T)$ for the stated maturity $T$  
**Bump size:** $1\text{ bp} = 10^{-4}$  
**Units:** currency per 1 bp for the stated notional  

**Sign convention (aligned to Chapter 37):**

$$\boxed{CS01 \equiv -\bigl(P(S+1\text{ bp}) - P(S)\bigr)}$$

so that spread widening ($+1$ bp) implies $P \downarrow$ and $CS01\gt 0$ for **long-credit** positions (e.g., selling protection).

Using the MTM identity $V(t) = (S_{\text{bp}}(t,T)-S_{0,\text{bp}})\,\text{RPV01}(t,T)$ (and holding `RPV01` fixed), a $+1$ bp widening changes PV by approximately:

- **Long protection:** $\Delta PV_{+1\text{ bp}} \approx +\text{RPV01}$, so $CS01 \approx -\text{RPV01}$
- **Short protection:** $\Delta PV_{+1\text{ bp}} \approx -\text{RPV01}$, so $CS01 \approx +\text{RPV01}$

**Example:** If $\text{RPV01}=USD 4{,}200$/bp for USD 10mm notional, then long protection has $CS01 \approx -USD 4{,}200$/bp and short protection has $CS01 \approx +USD 4{,}200$/bp.

> **Desk Reality: “What is being bumped?”**
>
> Different systems compute “CS01” differently (bump a quoted spread holding curves fixed vs re-fit the survival curve vs bump hazard nodes). `RPV01` gives the core scaling, but exact numbers can differ. Always confirm the bump design and the sign convention used in the risk report.

---

## 38.6 The Upfront-Plus-Coupon Trading Regime

Following CDS standardization (often referred to as the April 2009 “Big Bang”), many CDS and CDS indices trade with **fixed running coupons** plus an **upfront payment**. Instead of trading every contract at a bespoke running spread, the contract uses a standard coupon $c$ and the trade is brought to fair value by exchanging an upfront amount.

> **Deep Dive: The Big Bang (April 2009)**
>
> On April 8, 2009, a "Big Bang" occurred in the market for CDS contracts and the way in which they are traded. The standardization had three main parts: auction hardwiring, standardizing trading conventions, and central clearing.
>
> Under the running spread convention, no money was exchanged upfront which implied that CDS positions were implicitly leveraged. The upfront payments are made at initiation and are equal to the present value of the difference between the current market credit spread and the fixed coupon.

### 38.6.1 Standard Coupons

To facilitate trading, a fixed coupon is specified for standard transactions that trade. For illustration in this chapter, we will use coupons like 100 bp (0.01) and 500 bp (0.05); the specific coupon used is contract-defined and product-dependent.

This standardization makes CDS trade like bonds—the running payments are fixed, and the contract price adjusts via an upfront payment.

### 38.6.2 The Upfront Price Formula

Let $s$ be the current market par spread (decimal per annum) for the maturity and $c$ the fixed contractual coupon (decimal per annum). Using the risky annuity $A(t,T)$ from Section 38.5, a convenient approximation for the **clean upfront** as a fraction of notional is:

$$\boxed{U \approx A(t,T)\,(s-c)}$$

If you like bond-style quoting, define a clean price per USD 100 notional as $P=100-100U$, so:

$$\boxed{P \approx 100 - 100 \times A(t,T) \times (s - c)}$$

where:
- $P$ = price per USD 100 notional
- $A(t,T)$ = risky annuity (years)
- $s$ = quoted spread (decimal)

**Check (units and sign):** $A$ has units of years and $(s-c)$ has units of 1/year, so $U$ is dimensionless (a fraction of notional). If $s\gt c$ (market par spread above the fixed coupon), then the coupon is “too low” and the protection buyer pays a positive upfront to the seller. In dollars, a convenient equivalent is:

$$
Upfront_{\mathrm{USD}} \approx (s-c)_{\mathrm{bp}} \times RPV01(t,T).
$$
- $c$ = fixed coupon (decimal)

**Settlement mechanics:**
- If $s \gt  c$: the protection buyer pays $(100 - P)$ per USD 100 notional upfront
- If $s \lt  c$: the protection buyer receives $(P - 100)$ per USD 100 notional upfront

Equivalently, in dollar terms (using spreads in bp and `RPV01` in currency per bp):

$$\boxed{Upfront_{\mathrm{USD}} \approx (s_{\text{bp}}-c_{\text{bp}})\times \text{RPV01}(t,T)}$$

**Example:** Consider a 5-year CDS:
- Quoted spread: $s = 150$ bp = 0.015
- Standard coupon: $c = 100$ bp = 0.01
- Risky annuity: $A = 4.2$ years

$$P \approx 100 - 100 \times 4.2 \times (0.015 - 0.01) = 97.9$$

The protection buyer pays $(100 - 97.9) = 2.1$ per USD 100 notional upfront. On USD 10 million notional:

$$\text{Upfront} = 0.021 \times 10{,}000{,}000 = USD 210{,}000$$

**Intuition:** The spread (150 bp) exceeds the coupon (100 bp) by 50 bp. Over an annuity of 4.2 years, this is roughly $50 \times 4.2 = 210$ bp $\approx 2.1\%$ of notional upfront.

### 38.6.3 When Coupon Exceeds Spread

**Example:** For a safer credit with spread below the fixed coupon:
- Quoted spread: $s = 34$ bp = 0.0034
- Standard coupon: $c = 100$ bp = 0.01
- Risky annuity: $A = 4.447$ years

$$P \approx 100 - 100 \times 4.447 \times (0.0034 - 0.01) = 102.94$$

Since $P \gt  100$, the protection buyer receives USD 294,000 on USD 10 million notional at inception, compensating for the fact that they will "overpay" via the running coupon relative to fair value.

### 38.6.4 Why Fixed Coupons?

Under an all-running-spread regime, two CDS on the same name and maturity can have different coupons depending on when they were traded. This makes it harder to net positions and to quote trades in a bond-like “price” language.

Fixed coupons plus upfront payments reframe the trade:
- The **coupon** is standardized (like a bond coupon)
- The **upfront** carries the price/MTM information (like a bond clean price)

Mechanically, this reduces the number of bespoke coupons on the book and makes it easier to exchange economic value at trade time (and at unwind), rather than leaving the trade’s value embedded entirely in future running premiums.

### 38.6.5 Distressed Credits: Pure Upfront Format

Upfront contracts are non-standard for most issuers. However, when an issuer is in distress typically the market CDS spread is of the order of 1000 bp - the market switches from trading using the standard running spread to trading in an upfront format.

**Expand (why this happens):** A protection seller may prefer a sure but heavily discounted premium now rather than waiting for a very risky stream of premium cashflows.

The upfront CDS replaces the premium leg of a CDS with a single payment of $U(0)$ at the initiation of the contract. The protection leg is unchanged.

**Valuation identity (par at inception):** For a long-protection buyer who pays upfront $U(0)$, setting the contract value to zero at trade time gives:

$$\boxed{V(0)=(1-R)\int_0^T Z(0,s)\,(-dQ(0,s)) - U(0)=0}$$

so that:

$$\boxed{U(0)=(1-R)\int_0^T Z(0,s)\,(-dQ(0,s))}$$

i.e., the upfront equals the present value of the protection leg (per unit notional) when there is no running premium leg.

**Check (limits):**
- If default is very unlikely over $[0,T]$, the protection leg PV is near 0, so upfront is near 0.
- If default is near-certain very soon, the protection leg PV approaches $(1-R)$ (per unit notional), so upfront becomes large.

> **Desk Reality: Quote units**
>
> In upfront format, the premium is naturally quoted as an upfront **percentage/points per 100 notional** (e.g., “16.35% upfront”), not as bp per annum. A common break is mixing per-100 prices, decimals, and currency amounts without a unit check.

---

## 38.7 Mark-to-Market and Position Unwinding

A new CDS traded at the market par level has (approximately) zero value at inception. After inception, market spreads move and the position becomes “off-market,” creating mark-to-market (MTM) value.

### 38.7.1 A Practical MTM Identity

Let:
- $S_{0,\text{bp}}$ be the contractual running spread/coupon (in bp),
- $S_{\text{bp}}(t,T)$ be the current market par spread for maturity $T$ (in bp),
- $\text{RPV01}(t,T)$ be the risky PV of 1 bp of running premium (currency per bp for the stated notional).

**Anchor (per USD 1 face value):** $V(t)=(S(t,T)-S(0,T))\times \text{RPV01}(t,T)$.

Using spreads in bp and `RPV01` in currency per bp for the stated notional, a practical MTM identity for a **long protection** position is:

$$\boxed{V(t) = \bigl(S_{\text{bp}}(t,T)-S_{0,\text{bp}}\bigr)\,\text{RPV01}(t,T)}$$

For a **short protection** position, the value is the negative of this.

**Expand (what is held fixed):** If $S_{\text{bp}}(t,T)$ is the current par spread implied by the same curves used to compute `RPV01`, this identity is exact. It becomes an approximation when you use it to translate an exogenous spread shock into PV while treating `RPV01` as fixed (because `RPV01` itself changes with the curves).

**Check (units):** $(\text{bp})\times(\text{currency/bp})=\text{currency}$.

**Example:**
- Contractual spread: $S_0 = 75$ bp
- Current market spread: $S(t,T) = 320$ bp
- Notional: USD 10 million
- Risky annuity: $A(t,T)=4.208$ years

First compute:

$$\text{RPV01} = N \times 10^{-4} \times A = 10{,}000{,}000 \times 10^{-4} \times 4.208 = USD 4{,}208/\text{bp}$$

Then:

$$V(t) = (320-75)\times 4{,}208 = USD 1{,}030{,}960$$

The protection buyer has a mark-to-market gain of approximately USD 1.03 million.

### 38.7.2 Three Ways to Unwind (and Why They Differ)

There are three economically related but operationally different ways to remove or neutralize a position:

1. **Bilateral termination (close-out / tear-up):** agree a cash amount with the original counterparty and terminate the contract.
2. **Novation/assignment:** transfer the contract to a third party; an MTM amount is exchanged and the original counterparty is replaced.
3. **Offsetting trade:** enter the opposite CDS at current market terms. This neutralizes the protection leg, but leaves net premium payments over time; there is no immediate cash realisation unless you also terminate/novate.

**Check (scenario sanity):**
- If the name defaults immediately after you put on the offsetting trade, the protection legs offset and the realised net premium is small.
- If the name survives, you realise the spread difference over time (subject to discounting).

### 38.7.3 Clean versus Full Mark-to-Market

Like bonds, CDS can be quoted **clean** (excluding accrued premium since the last coupon date) or **full/dirty** (including accrued). Define:

The clean value of the CDS contract is calculated by subtracting away the accrued coupon.

$$\boxed{\text{Clean MTM}=\text{Full MTM}-\text{Accrued}}$$

The sign of `Accrued` depends on whether you pay or receive the running coupon: for a long-protection position, accrued premium is typically a negative cashflow (you owe it if you unwind between coupon dates).

**Critical distinction:** Clean vs full is a *quotation convention* to avoid MTM jumps on coupon dates. It does **not** change the contractual rule that accrued premium is owed on default (Section 38.4).

> **Desk Reality: Unwind settlement lag**
>
> Protection risk can begin on an effective date that is T+1 calendar, but the cash settlement of an unwind value can occur later (e.g., T+3 business). If you are marking MTM for settlement, roll the “T+0” MTM to the settlement date using the appropriate discounting rate and be consistent about whether you report clean or full.

---

## 38.8 Buyer versus Seller: Signs and Cashflow Map

The following table summarizes cashflow signs (positive = receive, negative = pay):

| Component | Protection Buyer | Protection Seller |
|-----------|------------------|-------------------|
| Scheduled premium payments | $-N s \Delta$ | $+N s \Delta$ |
| Accrued premium at default | $-$ (pays) | $+$ (receives) |
| Protection payment | $+N(1-R)$ | $-N(1-R)$ |
| Upfront (if $s \gt  c$) | $-$ (pays) | $+$ (receives) |
| Upfront (if $s \lt  c$) | $+$ (receives) | $-$ (pays) |
| MTM if spreads widen | $+$ (gain) | $-$ (loss) |
| MTM if spreads tighten | $-$ (loss) | $+$ (gain) |

The cashflows are exactly symmetric: every dollar the buyer pays, the seller receives, and vice versa.

---

## 38.9 Worked Example: Full CDS Lifecycle (Fixed Coupon + Upfront + Default)

**Example Title:** A 5Y CDS trade that defaults mid-quarter

**Context**
- A protection buyer enters a 5Y CDS at a fixed 100 bp coupon (example) when the market par spread is 150 bp, so an upfront is paid.
- The name defaults mid-quarter; we track upfront, running premium, accrued-at-default, and the protection payment.

**Timeline (Make Dates Concrete)**
- Trade date: Monday, 19 January 2026
- Effective date (protection start): Tuesday, 20 January 2026 (T+1 calendar)
- Upfront settlement date: Thursday, 22 January 2026 (assume T+3 business; confirmation-specific)
- Premium dates (adjusted): 20 Mar, 22 Jun, 21 Sep, 21 Dec 2026
- Credit event: Monday, 10 August 2026

**Inputs**
- Notional: USD 10,000,000
- Contract coupon: $c=100$ bp $=0.01$
- Market par spread at trade: $s=150$ bp
- Risky annuity (assumed): $A \approx 4.2$ years
- Day count: ACT/360

**Outputs (What You Produce)**
- Upfront: USD 210,000 paid by protection buyer
- Accrued premium at default: USD 13,611 paid by protection buyer
- Protection payment: USD 6,000,000 received by protection buyer (given 40% recovery)

**Step-by-step**
1. **Translate quote to upfront**
   - $\text{RPV01} = N \times 10^{-4} \times A = 10{,}000{,}000 \times 10^{-4} \times 4.2 = USD 4{,}200/\text{bp}$
   - $\text{Upfront} \approx (s-c)_{\text{bp}} \times \text{RPV01} = 50 \times 4{,}200 = USD 210{,}000$ (buyer pays)
2. **Compute scheduled premium payments**
   - $\text{PremiumPay} = N \cdot c \cdot \text{DayDiff}/360$
3. **Compute accrued premium at default**
   - Days since last payment (22 Jun → 10 Aug): 49
   - $\text{Accrued} = 10{,}000{,}000 \times 0.01 \times 49/360 = USD 13{,}611$
4. **Compute protection payment**
   - Recovery from auction: $R=40\%$
   - $\text{Protection} = N(1-R)=10{,}000{,}000\times 0.60=USD 6{,}000{,}000$

**Cashflows (table)**

| Date | Cashflow (Buyer) | Explanation |
|------|------------------|-------------|
| 22 Jan 2026 | $-USD 210{,}000$ | Upfront (since $s\gt c$) |
| 20 Mar 2026 | $-USD 16{,}389$ | Running coupon for 59 days at 100 bp |
| 22 Jun 2026 | $-USD 26{,}111$ | Running coupon for 94 days at 100 bp |
| 10 Aug 2026 | $-USD 13{,}611$ | Accrued coupon to credit event |
| ~Aug 2026 | $+USD 6{,}000{,}000$ | Protection payment (cash settlement) |

**P&L / Risk Interpretation**
- The upfront is the PV of paying a 100 bp coupon when the market par level is 150 bp.
- The operational “break points” are almost always timeline/convention errors: effective date, stub accrual, or forgetting accrued premium at default.

**Sanity Checks**
- Units check: $(\text{bp})\times(\text{currency}/\text{bp})=\text{currency}$.
- Sign check: if $s\gt c$, buyer pays upfront and pays coupons; on default buyer receives protection.
- Bounds check: $0 \le \text{Accrued} \le$ full-period coupon.

**Debug Checklist (When Your Result Looks Wrong)**
- Are spreads in bp or decimals?
- Did you use ACT/360 and the correct stub days?
- Did you include accrued premium at default with the correct sign?
- Are you mixing clean and full MTM conventions?

---

## 38.10 Practical Notes

### Contract Booking Checklist

Before booking a CDS trade, verify:
- [ ] Reference entity and obligation specification
- [ ] Notional amount and currency
- [ ] Direction: buy or sell protection
- [ ] Trade date and effective date (T+1 calendar)
- [ ] Maturity date and roll convention (20 Mar/Jun/Sep/Dec)
- [ ] Running coupon and upfront amount
- [ ] Day count (ACT/360) and business day convention
- [ ] Holiday calendar for date adjustments
- [ ] Settlement type: physical or cash (auction)

### Common Pitfalls

| Pitfall | Consequence |
|---------|-------------|
| Forgetting accrued premium at default | P&L break on credit event |
| Sign confusion (buyer = short credit) | Position booked wrong direction |
| Using bond clean/dirty logic for CDS | Confusing quoting accrued with cashflow accrued |
| Wrong effective date (T+1 business vs calendar) | Coverage gap on weekend defaults |
| Ignoring day count (ACT/360 vs ACT/365) | Premium calculation errors |
| Wrong holiday calendar | Schedule mismatch, premium errors |
| Confusing CDS IMM (20th) with futures IMM (3rd Wed) | Wrong roll dates |
| Ignoring survival-weighting when computing $A(t,T)$/RPV01 | Mispriced upfront and spread risk |

### Sanity Checks

| Check | What to Verify |
|-------|----------------|
| Premium reasonableness | Quarterly payment ≈ $N \times s \times 0.25$ |
| Accrued bounds | $0 \leq \text{Accrued} \leq \text{Full quarter premium}$ |
| Protection bounds | $0 \leq N(1-R) \leq N$ |
| Upfront sign | If $s \gt  c$, buyer pays; if $s \lt  c$, buyer receives |
| Risky vs risk-free annuity | For high-quality names, $A(t,T)$ is close to the risk-free annuity; for distressed names, $A(t,T)$ is smaller |
| MTM sign | Spreads widen → buyer gains; spreads tighten → buyer loses |

### Implementation Verification Tests

1. **Repricing check:** Input the agreed terms; verify the system reproduces the traded upfront
2. **Schedule check:** Verify payment dates against a trusted source (Bloomberg CDSW)
3. **RPV01 check:** Convert $\text{RPV01}/(N \times 10^{-4})$ to get a risky annuity in years, then compare to years-to-maturity for high-quality names
4. **MTM check:** If spreads are in bp and RPV01 is in USD /bp for the position notional, verify $MTM \approx (S_{\text{bp}}-S_{0,\text{bp}})\times RPV01$

---

## Summary

1. A CDS transfers credit risk: the protection buyer pays the premium leg and receives a contingent protection payment after a credit event.
2. Market language: “buy protection” = short credit risk; “sell protection” = long credit risk.
3. Premium payments are typically quarterly on ACT/360, scheduled on standard dates (20 Mar/Jun/Sep/Dec) with stubs as needed.
4. In this chapter’s examples, the effective date is the calendar day after trade date (T+1 calendar); the confirmation governs any business-day adjustments.
5. On a credit event, future premiums stop, but accrued premium up to the event date is owed.
6. The protection payment is bounded by $0 \le N(1-R) \le N$ and can settle physically or in cash (often via auction).
7. The risky annuity $A(t,T)$ values the survival-weighted premium stream (including accrued-at-default under a mid-period approximation).
8. `RPV01(t,T)` is the PV of 1 bp of running premium (currency per bp for the stated notional): `RPV01 = N × 10^{-4} × A`.
9. Fixed coupon + upfront trades can be approximated as upfront fraction $U \approx A(s-c)$, or upfront dollars $\approx (s_{\text{bp}}-c_{\text{bp}})\times RPV01$.
10. A practical MTM identity is $V(t)\approx (S_{\text{bp}}(t,T)-S_{0,\text{bp}})\times RPV01$ (with curve-rebuild caveats).
11. Clean MTM is a quoting convention: `Clean = Full − Accrued`; do not confuse it with the contractual accrued-at-default cashflow.
12. Unwind cash settlement can occur after a short lag (e.g., T+3 business); be consistent about settlement date and clean/full conventions.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Premium leg | Periodic payments from buyer to seller, quarterly ACT/360 | Determines the cost of protection |
| Protection leg | Contingent payment of $N(1-R)$ after credit event | Determines the payout on default |
| Accrued premium | Premium owed from last coupon date to credit event | Real cashflow often missed in pricing |
| Roll dates | 20 Mar/Jun/Sep/Dec schedule dates | All standard CDS follow this calendar |
| T+1 calendar | Example convention: effective date is the calendar day after trade date (confirmation governs adjustments) | Affects start-of-protection timing and breaks |
| Risky annuity $A(t,T)$ | Survival-weighted PV factor for the premium stream (years) | Turns spreads into PV and into RPV01 |
| RPV01 | PV of 1 bp running premium (currency per bp for stated notional) | Upfront and CS01 scaling |
| Upfront payment | Cash exchanged at inception when $s \neq c$ | Standardized coupons require this adjustment |
| Clean vs full MTM | Quotation: clean excludes accrued; full includes | Explains MTM jumps around coupon dates |
| Unwind vs offset | Close-out/novation exchanges MTM cash; offset leaves net premiums | Avoids “offset ≠ cash unwind” confusion |

---

## Notation

| Symbol | Definition |
|--------|------------|
| $N$ | CDS notional (face value) |
| $s$ | Running spread (decimal per annum; 100 bp = 0.01) |
| $c$ | Fixed coupon (decimal per annum) |
| $\Delta(t_1, t_2)$ | Accrual fraction = DayDiff(t_1, t_2)/360 |
| $\tau$ | Credit event date |
| $R$ | Recovery rate (fraction of par) |
| $Z(t, T)$ | Risk-free discount factor from $t$ to $T$ |
| $Q(t, T)$ | Survival probability from $t$ to $T$ |
| $A(t,T)$ | Risky annuity (years per USD 1 notional per 1.00 of running spread) |
| RPV01$(t, T)$ | PV of 1 bp running premium (currency per bp for the stated notional) |
| $P$ | Upfront price per 100 notional |
| $V(t)$ | Mark-to-market value |
| $S_0$ | Contractual spread |
| $S_t$ or $S(t,T)$ | Current market spread |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a CDS? | A bilateral contract transferring credit risk; buyer pays premium, seller pays contingent protection |
| 2 | Who pays the premium leg? | The protection buyer |
| 3 | What does "buying protection" mean economically? | Short credit risk (profit if credit deteriorates) |
| 4 | What day count does CDS premium use? | ACT/360 |
| 5 | What are the CDS roll dates? | 20 Mar, 20 Jun, 20 Sep, 20 Dec |
| 6 | What effective date convention do we use in examples? | T+1 calendar (calendar day after trade date; confirmation governs adjustments) |
| 7 | Why does effective-date convention matter? | It determines when protection starts; weekend/holiday handling can create coverage or break-risk differences |
| 8 | Formula for premium payment? | $N \cdot s \cdot \text{DayDiff}/360$ |
| 9 | What happens to premiums after a credit event? | Future payments cancelled; accrued to $\tau$ is owed |
| 10 | Formula for protection payment? | $N(1 - R)$ where $R$ is recovery rate |
| 11 | What is physical settlement? | Buyer delivers face value of bonds, seller pays par |
| 12 | What is cash settlement? | Seller pays par minus recovery price (auction) |
| 13 | Give example fixed coupons used in points-upfront trading. | 100 bp or 500 bp (illustrative; product-dependent) |
| 14 | What is RPV01? | PV of 1 bp of running premium (currency per bp for the stated notional) |
| 15 | What is the risky annuity $A(t,T)$? | The survival-weighted PV factor (years) that underlies RPV01: $RPV01 = N \times 10^{-4}\times A$ |
| 16 | If spread > coupon, who pays upfront? | Protection buyer pays |
| 17 | If spread < coupon, who pays upfront? | Protection seller pays (buyer receives) |
| 18 | Approx upfront (dollars) in fixed coupon trades? | $Upfront_{\mathrm{USD}} \approx (s_{\text{bp}}-c_{\text{bp}})\times RPV01$ |
| 19 | Approx MTM identity (long protection)? | $V\approx (S_{\text{bp}}-S_{0,\text{bp}})\times RPV01$ |
| 20 | What is clean vs full MTM? | Clean excludes accrued; full includes accrued: Clean = Full − Accrued |
| 21 | When might trades be quoted in upfront format? | When spreads are very wide (distressed); quotes naturally become upfront % of notional |
| 22 | What is the settlement timing for CDS unwind? | Often a short lag (e.g., T+3 business days; confirmation-specific) |
| 23 | Effect of accrued at default on breakeven spread? | Lowers it (approximately $\frac{S^2}{2(1-R)f}$ under simplifying assumptions) |
| 24 | Why do protection sellers prefer upfront on distressed? | Certain money now vs. risky premium stream |

---

## Mini Problem Set

1. Compute the quarterly premium payment for $N = USD 25$ million, $s = 150$ bp, and a 90-day quarter under ACT/360.
2. A CDS has effective date 15 January 2026. The first roll date is 20 March 2026. Calculate the accrual fraction for the stub period.
3. Default occurs 45 days after the last coupon date. Compute accrued premium for $N = USD 10$ million and coupon $c=200$ bp.
4. If auction recovery is 28%, compute the protection payment on USD 15 million notional.
5. Fixed-coupon trade: $s = 250$ bp, $c = 100$ bp, risky annuity $A = 4.0$ years. Compute the clean price per USD 100 notional and state who pays upfront.
6. Using $RPV01$: $N=USD 10$ million, $RPV01=USD 4{,}500$/bp, $s = 80$ bp, $c = 100$ bp. Compute upfront dollars and state direction.
7. Explain why physical and cash settlement produce the same economic result when the auction final price equals the deliverable market value.
8. Create a sign table showing buyer and seller cashflows for: (i) scheduled premium, (ii) accrued at default, (iii) protection payment, (iv) upfront (when $s\gt c$).
9. A credit event occurs on Saturday. A trade executed on Friday has effective date Saturday (T+1 calendar). Explain why the convention matters.
10. In one sentence, define clean vs full MTM and give one operational “break” it prevents or causes.
11. If $A(0,2)=1.84$ years for a 2-year CDS and $N=USD 5$ million, compute $RPV01$ in USD /bp.
12. MTM: Long protection with contractual $S_0=150$ bp, market $S=250$ bp, and $RPV01=USD 8{,}000$/bp. Compute MTM.

### Solution Sketches (Selected)

**1.** $USD 93{,}750$ (use $s=0.015$ and $90/360=0.25$).

**5.** $P \approx 100 - 100\times 4.0\times(0.025-0.01)=94$; buyer pays $6\%$ upfront (since $s\gt c$).

**12.** $MTM\approx (250-150)\times 8{,}000 = USD 800{,}000$ (long protection gains when spreads widen).

**7.** Physical settlement: buy deliverable at the market recovery price and deliver at par, netting $N(1-R)$. Cash settlement pays $N(1-R)$ directly when the auction price equals the deliverable market value.

---

## References

- O'Kane, *Modelling Single-name and Multi-name Credit Derivatives* (contract mechanics, schedules, RPV01/$A(t,T)$, MTM, clean vs full, unwinds, upfront CDS)
- Hull, *Options, Futures, and Other Derivatives* (CDS overview, physical vs cash settlement, accrued premium at default, auction intuition)
- Hull, *Risk Management and Financial Institutions* (standard maturity dates; fixed coupons and upfront direction)
- Neftci, *Principles of Financial Engineering* (April 2009 standardization; points-upfront trading; fixed coupons)
- McNeil, Frey, and Embrechts, *Quantitative Risk Management* (CDS premium leg vs default leg definition; spread quoting)
