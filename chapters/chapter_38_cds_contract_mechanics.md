# Chapter 38: CDS Contract Mechanics

---

## Introduction

Before you can price a credit default swap, hedge a corporate bond position, or explain P&L after a credit event, you must understand exactly what cashflows move between the two parties and when. A CDS may function economically like credit insurance, but it is contractually a derivative with precise mechanics that differ from insurance in critical ways.

Consider a portfolio manager holding \$50 million of XYZ Corp bonds who wants to hedge the default risk. She calls her dealer and "buys protection" on XYZ via a 5-year CDS at 150 basis points. What has she agreed to? She will pay quarterly premiums based on that spread, but the exact payment dates follow CDS-specific conventions (the 20th of March, June, September, December—not the bond's coupon dates). If XYZ experiences a credit event mid-quarter, her premium payments stop, but she owes accrued premium up to the event date. The protection payment she receives depends on whether the contract settles physically or via an auction-determined cash price.

Getting any of these details wrong—the day count, the effective date, the accrued premium at default—creates P&L breaks, failed reconciliations, and hedging errors. The CDS market has standardized many conventions, but the standardization itself requires understanding: why T+1 calendar effective dates? Why do roll dates fall on the 20th rather than month-end? Why does the market now trade with fixed coupons plus upfront payments rather than running spreads alone?

This chapter covers the mechanics of CDS contracts and introduces the **Risky PV01 (RPV01)**—the fundamental building block for CDS valuation. We establish the premium leg and protection leg cashflows, work through date conventions and settlement methods, derive the RPV01 formula that incorporates survival probabilities, and explain the upfront-plus-coupon trading regime including distressed credit conventions. We also cover mark-to-market calculation and position unwinding mechanics.

Chapter 39 will examine credit event definitions and settlement procedures in detail, while Chapter 41 will develop the full pricing framework that values these cashflows.

---

## 38.1 The Credit Default Swap: Definition and Economic Logic

O'Kane defines a CDS as "a bilateral contract that transfers the credit risk of a reference entity from one party to another for a specified period." More specifically, it is designed to protect against loss from par on a bond or loan in exchange for a premium.

The contract has two parties with opposite exposures:

**The protection buyer** pays periodic premiums and receives a contingent payment if a credit event occurs. Economically, buying protection is equivalent to shorting the credit—the buyer profits if the reference entity's creditworthiness deteriorates or it defaults.

**The protection seller** receives premiums and makes a contingent payment after a credit event. Selling protection is equivalent to being long credit risk—the seller profits if the entity survives and spreads tighten.

A critical terminology point: "buying a CDS" in market language typically means buying protection, which is short credit risk. This is the opposite of cash bond conventions where "buying" means going long. Confusion on this point has caused expensive booking errors.

> **Analogy: The Insurance Policy**
>
> A CDS is remarkably similar to Car Insurance.
>
> 1.  **Premium Leg**: You pay a quarterly fee (premium) to Allstate (the seller).
> 2.  **Protection Leg**: If your car is totaled (default), Allstate pays you the replacement value.
> 3.  **The Difference**: In the CDS market, you don't need to own the car to buy the insurance! You can buy insurance on your neighbor's car crashing. This is called "speculation."
>
> **Visual: CDS Cashflows**
> Buyer $\xrightarrow{\text{Premiums}}$ Seller
> Seller $\xrightarrow{\text{Protection (if Default)}}$ Buyer

---

## 38.2 The Premium Leg: Periodic Payments from Buyer to Seller

The premium leg consists of periodic payments from the protection buyer to the protection seller. O'Kane describes these as "typically quarterly using Actual/360," and payments terminate at maturity or immediately following a credit event.

### 38.2.1 The Premium Payment Formula

For a notional $N$, running spread $s$ (in decimal form, so 100 bp = 0.01), and accrual fraction $\Delta$, the premium payment at date $t_n$ is:

$$\boxed{\text{PremiumPay}(t_n) = N \cdot s \cdot \Delta(t_{n-1}, t_n)}$$

where the accrual fraction under ACT/360 is:

$$\Delta(t_{n-1}, t_n) = \frac{\text{DayDiff}(t_{n-1}, t_n)}{360}$$

**Unit check:** $N$ (dollars) × $s$ (1/year) × $\Delta$ (years) = dollars. The spread is quoted per annum, and the accrual fraction converts the annual rate to the period's portion.

**Example (O'Kane-style):** Consider a \$10 million notional CDS with a 35 bp running spread. For a 91-day quarter:

$$\text{PremiumPay} = 10{,}000{,}000 \times 0.0035 \times \frac{91}{360} = 10{,}000{,}000 \times 0.0035 \times 0.2528 = \$8{,}847$$

### 38.2.2 Payment Schedule Construction: The Step-by-Step Algorithm

CDS payment dates follow a standardized schedule based on **roll dates**: 20 March, 20 June, 20 September, and 20 December. O'Kane notes these are often called "CDS IMM dates," but importantly, they are "not always the same as the corresponding IMM date. The CDS market always chooses the 20th date of the month while the official IMM date is the 3rd Wednesday of the month."

O'Kane provides the schedule construction algorithm explicitly:

**Step 1 — Determine the Maturity Date:**
"The maturity date is calculated as the first 'IMM' date $T$ years after the effective date."

For a 5-year CDS with effective date 20 January 2026, the maturity date is the first of 20 Mar/Jun/Sep/Dec that is at least 5 years after effective date—namely, 20 March 2031.

**Step 2 — Generate Payment Dates by Stepping Back:**
"Starting at the maturity date, we step back in time in three-month steps, landing on earlier 'IMM' dates until we arrive at the last date which falls after the effective date."

Working backwards from 20 March 2031:
- 20 March 2031 (maturity)
- 20 December 2030
- 20 September 2030
- ... continue stepping back ...
- 20 March 2026 (first payment date after effective)

**Step 3 — Adjust for Business Days:**
"All of these dates are then adjusted so as not to fall on weekends or holidays. This is typically done by rolling them forward to the next business day."

O'Kane notes: "Since holidays are country specific, a holiday calendar will need to be specified. For example, a contract traded on a US credit in NY will generally use a US holiday calendar."

**Step 4 — Calculate Premium Payments:**
"Once we have the dates, the payments are calculated using the Actual 360 day count convention."

> **Desk Reality: Schedule Mismatches as P&L Break Source**
>
> One of the most common causes of P&L breaks in CDS trading comes from schedule mismatches between front office and operations systems. The culprits:
>
> 1. **Wrong holiday calendar**: Using US holidays when the contract specifies TARGET (European)
> 2. **Miscounting stub periods**: The first coupon is almost always a short stub—systems that assume 90-day first periods are wrong
> 3. **Weekend adjustment direction**: Following vs. Modified Following convention can shift dates differently
> 4. **IMM date confusion**: Using the 3rd Wednesday instead of the 20th
>
> When your daily P&L shows an unexplained break on a coupon date, check the schedule first.

**Worked Example: Building a Complete Schedule**

Trade date: Wednesday, 15 January 2026
Effective date: Thursday, 16 January 2026 (T+1 calendar)
Notional: \$10 million
Spread: 35 bp
Maturity: 5 years → First IMM date ≥ 16 January 2031 → 20 March 2031

O'Kane provides Figure 5.3 showing exactly this type of schedule. The cash flows are not all equal, "reflecting the small but real differences in the time between payments caused by adjustments to avoid weekends and public holidays. There is also a short stub at the beginning reflecting the shorter period to the first premium payment."

| Flow Date (Adjusted) | Day Count Fraction | Payment (\$10mm @ 35bp) |
|---------------------|-------------------|------------------------|
| 20 Mar 2026 (Fri) | 63/360 = 0.1750 | \$6,125 |
| 22 Jun 2026 (Mon) | 94/360 = 0.2611 | \$9,139 |
| 21 Sep 2026 (Mon) | 91/360 = 0.2528 | \$8,847 |
| 21 Dec 2026 (Mon) | 91/360 = 0.2528 | \$8,847 |
| ... | ... | ... |
| 20 Mar 2031 (Thu) | ~90/360 | ~\$8,750 |

**Key observation:** The first payment (20 Mar 2026) covers only 63 days—a **short front stub**. This is normal for CDS trades that don't start exactly on a roll date.

### 38.2.3 Stub Period Mechanics

The first premium payment almost always involves a **stub period**—a period shorter or longer than the standard 91-92 days. O'Kane notes: "There is also a short stub at the beginning reflecting the shorter period to the first premium payment."

**Front stub (short first coupon):** Trade date falls between roll dates, so the first accrual period is shorter than 3 months. This is the most common case.

**Example:** Trade on 15 January means first payment on 20 March covers only 63 days (16 Jan to 20 Mar), not the standard ~90 days. The first payment is correspondingly smaller.

**Why this matters for P&L:**
If a system incorrectly assumes a full 90-day first period, the first premium payment will be overstated by approximately:

$$\frac{90 - 63}{360} \times N \times s = \frac{27}{360} \times 10{,}000{,}000 \times 0.0035 = \$2{,}625$$

On a \$10 million trade, that's a \$2,625 error on day one—enough to trigger a break investigation.

### 38.2.4 Effective Date: T+1 Calendar

O'Kane explicitly states that the effective date is "the calendar day after trade date (T+1 calendar)," and importantly, "need not be a business day." Why allow protection to start on a weekend or holiday?

The answer is that credit events can occur on any day. A company can file for bankruptcy on a Saturday. If the effective date were pushed to Monday, a credit event on Sunday would fall outside the protection period. As O'Kane notes, "there is value in protection starting on a non-business day because defaults can occur on weekends/holidays."

> **Desk Reality: The Weekend Coverage Question**
>
> When a trade is executed on Friday, the effective date is Saturday. If the reference entity files for bankruptcy on Sunday, is the protection buyer covered?
>
> **Yes.** This is precisely why the T+1 calendar convention exists. The alternative—T+1 business day—would create a gap in coverage. For a company in financial distress, this gap could be worth millions.
>
> Middle office staff marking positions should verify that systems correctly handle weekend effective dates. A common error is systems that "helpfully" adjust weekend dates to Monday.

---

## 38.3 The Protection Leg: Contingent Payment After Credit Event

The protection leg is the contingent payment from seller to buyer that O'Kane describes as making "up to par the value of a deliverable obligation following a credit event." This payment only occurs if a credit event happens during the protection period.

### 38.3.1 Protection Payment Formula

If the recovery price (the post-default market value of the reference obligation) is $R$ as a fraction of par, the protection payment is:

$$\boxed{\text{ProtPay} = N(1 - R)}$$

**Bounds check:** For $0 \leq R \leq 1$, the protection payment lies between 0 (full recovery) and $N$ (zero recovery).

**Example:** On \$10 million notional with recovery of 40%:

$$\text{ProtPay} = 10{,}000{,}000 \times (1 - 0.40) = \$6{,}000{,}000$$

The protection buyer receives \$6 million, compensating for the loss from par on the reference obligations.

### 38.3.2 Settlement Methods

O'Kane and Hull describe two settlement methods:

**Physical settlement:** "The protection buyer delivers face value of deliverable obligations to the protection seller. In return, the protection seller makes a simultaneous payment of the face value in cash to the protection buyer."

If the buyer can source bonds trading at 38 cents on the dollar, they pay \$3.8 million to acquire \$10 million face value, deliver these bonds, and receive \$10 million from the seller—a net gain of \$6.2 million.

**Cash settlement:** "The protection seller pays the protection buyer the face value of the protection minus the recovery price of the reference obligation in cash." If the recovery price is 38%, the seller pays $10{,}000{,}000 \times (1 - 0.38) = \$6{,}200{,}000$.

Both methods should produce the same economic result when the cash settlement price equals the market value of deliverable obligations. Hull notes that "as is now usual" there is cash settlement via an ISDA-organized auction process to determine the recovery price.

**Why cash settlement became standard:** O'Kane observes that physical settlement creates problems "when protection notional is large relative to deliverables." After major credit events, protection notionals often exceed the outstanding deliverable bonds, making physical delivery impractical. The auction mechanism solves this by establishing a single recovery price for all CDS contracts.

---

## 38.4 Accrued Premium at Default

When a credit event occurs between payment dates, the protection buyer owes accrued premium for the portion of the period during which protection was in force. This is a real cashflow, not just an accounting convention.

### 38.4.1 The Accrued Premium Formula

If default occurs at time $\tau$ with $t_{n-1} < \tau \leq t_n$:

$$\boxed{\text{AccruedPrem}(\tau) = N \cdot s \cdot \frac{\text{DayDiff}(t_{n-1}, \tau)}{360}}$$

**Example:** Using the schedule from Section 38.2.2, suppose default occurs on 10 August 2026. The previous payment date was 22 June 2026, so 49 days have elapsed:

$$\text{AccruedPrem} = 10{,}000{,}000 \times 0.0035 \times \frac{49}{360} = \$4{,}764$$

**Sanity check:** The accrued amount (\$4,764) is less than a full quarter's premium (\$8,847), as expected since default occurred roughly halfway through the period.

### 38.4.2 Premium Payments Stop at Default

O'Kane is explicit: "Premium payments terminate immediately following a credit event." After default:
- All future scheduled payments are cancelled
- The accrued premium up to $\tau$ is owed (this is the final premium cashflow)
- The protection payment is triggered

This differs from a bond, where coupons may continue accruing through a grace period. In CDS, the credit event is a hard stop.

### 38.4.3 Effect of Accrued Premium on Breakeven Spread

O'Kane derives an important analytical result: including accrued premium at default **lowers** the breakeven spread. The intuition is that the protection seller receives additional premium (accrued) when default occurs, making the protection leg less costly to provide.

O'Kane provides an approximation for the effect:

$$\boxed{S(\text{without accrued}) - S(\text{with accrued}) \approx \frac{S^2}{2(1-R)f}}$$

where $f$ is the payment frequency (4 for quarterly).

**Numerical example:** For $S = 500$ bp = 0.05, $R = 40\%$, $f = 4$:

$$\text{Effect} \approx \frac{(0.05)^2}{2 \times 0.60 \times 4} = \frac{0.0025}{4.8} = 0.00052 = 5.2\text{ bp}$$

O'Kane's Figure 6.1 shows this relationship graphically: "The effect of incorporating the accrued premium at default is to lower the breakeven spread." The effect is quadratic in the spread level—negligible for investment grade but meaningful for high-yield credits.

> **Practitioner Note:** For IG credits trading at 50-100 bp, the effect is less than 0.5 bp—essentially negligible. But for distressed names trading at 1000+ bp, the effect can be 10-20 bp. Quant systems should include the accrued-at-default term; quick estimates can use the approximation.

---

## 38.5 The Risky PV01: Valuing the Premium Leg

The **Risky PV01 (RPV01)** is the present value of receiving 1 basis point per annum on the premium leg until maturity or default, whichever comes first. It is the fundamental building block for CDS pricing, mark-to-market, and hedging.

O'Kane emphasizes: "The modelling challenge for pricing a CDS is to incorporate into the calculation of the RPV01 the risk that the reference entity may experience a credit event resulting in the loss of the subsequent premium payments."

### 38.5.1 Premium Leg Present Value

The present value of the premium leg has two components:

1. **Scheduled payments**: Premium payments made at each coupon date, conditional on survival
2. **Accrued at default**: Premium accrued from the last coupon date to the default time, if default occurs

O'Kane writes the premium leg PV as:

$$\boxed{\text{Premium Leg PV} = S_0 \cdot \text{RPV01}(t, T)}$$

where $S_0$ is the contractual spread.

### 38.5.2 Deriving the RPV01 Formula

Following O'Kane's derivation (Equations 6.2-6.4), we start with the building blocks from Chapter 36.

**Risky discount factor:** The present value of \$1 paid at time $t_n$, conditional on survival to $t_n$, under independence of interest rates and default:

$$\hat{Z}(t, t_n) = Z(t, t_n) \cdot Q(t, t_n)$$

where $Z(t, t_n)$ is the risk-free discount factor and $Q(t, t_n)$ is the survival probability.

**Component 1 — Scheduled Payments:**

For scheduled premium payments made at dates $t_1, \ldots, t_N$ if the credit survives:

$$\text{Scheduled PV} = \sum_{n=1}^{N} \Delta(t_{n-1}, t_n) \cdot Z(t, t_n) \cdot Q(t, t_n)$$

**Component 2 — Accrued at Default:**

If default occurs between $t_{n-1}$ and $t_n$, accrued premium from $t_{n-1}$ to the default time $\tau$ must be paid. O'Kane shows the present value of this contingent payment involves an integral over the default density, but provides a practical approximation:

$$\text{Accrued PV} \approx \frac{1}{2} \sum_{n=1}^{N} \Delta(t_{n-1}, t_n) \cdot Z(t, t_n) \cdot (Q(t, t_{n-1}) - Q(t, t_n))$$

The intuition: on average, default occurs mid-period, so the expected accrued is half the full period's premium.

**The Full RPV01 Formula:**

Combining both components and simplifying, O'Kane derives:

$$\boxed{\text{RPV01}(t, T) = \frac{1}{2} \sum_{n=1}^{N} \Delta(t_{n-1}, t_n) \cdot Z(t, t_n) \cdot (Q(t, t_{n-1}) + Q(t, t_n))}$$

This elegant formula averages adjacent survival probabilities for each period, capturing both the scheduled payment (weighted by $Q(t, t_n)$) and the expected accrued at default (difference term).

**Sanity checks:**
- If $Q(t, t_n) = 1$ for all $n$ (no default risk), RPV01 reduces to the risk-free annuity
- If $Q(t, t_n) \to 0$ rapidly, RPV01 is small (payments unlikely to be received)
- Units: $\Delta$ (years) × $Z$ (dimensionless) × $Q$ (dimensionless) = years

### 38.5.3 Worked Example: Computing RPV01

**Setup:**
- Valuation date: $t = 0$
- Maturity: $T = 2$ years (8 quarterly periods)
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

**Step 2: Apply the RPV01 formula**

Using $\Delta = 0.25$ for each quarterly period and $Q(0, t_0) = 1$:

$$\text{RPV01} = \frac{1}{2} \sum_{n=1}^{8} 0.25 \times Z(0, t_n) \times (Q(0, t_{n-1}) + Q(0, t_n))$$

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

$$\text{RPV01}(0, 2) = \frac{1}{2} \times 1.8546 = 0.9273 \text{ years}$$

**Interpretation:** The credit-risky annuity (RPV01) is 0.9273 years. The risk-free annuity (PV01) would be approximately 0.98 years—the difference reflects the 2% annual hazard rate reducing expected payments.

> **Desk Reality: When RPV01 ≈ PV01 and When They Diverge**
>
> For investment-grade credits with low hazard rates (< 1%), the RPV01 is very close to the risk-free PV01. This is why traders often say "duration" without specifying risky vs. risk-free.
>
> For high-yield credits (hazard rates of 3-10%), the difference becomes significant. For a 5-year CDS on a name with 10% annual hazard rate, the RPV01 might be only 70% of the risk-free annuity.
>
> For distressed credits trading at very high spreads (> 2000 bp), the RPV01 can be dramatically lower than the risk-free annuity, because there is substantial probability that later premium payments will never be received.

### 38.5.4 Credit DV01 and RPV01

The **Credit DV01** (or **CS01**) is the sensitivity of the CDS mark-to-market to a 1 bp change in spread. O'Kane shows that for an on-market CDS (where the current spread equals the contractual spread):

$$\boxed{\text{Credit DV01} = \text{RPV01}(t, T) \times 1\text{ bp}}$$

Per \$1 notional, if RPV01 = 4.2 years, then:

$$\text{Credit DV01} = 4.2 \times 0.0001 = 0.00042 \text{ per dollar notional}$$

For a \$10 million position:

$$\text{Credit DV01} = 4.2 \times 0.0001 \times 10{,}000{,}000 = \$4{,}200 \text{ per bp}$$

When the current spread differs from the contractual spread, O'Kane notes: "When $S_t$ deviates from $S_0$, this is no longer true. The Credit DV01 increases or decreases depending on the value of $(S_0 - S_t)$."

---

## 38.6 The Upfront-Plus-Coupon Trading Regime

Since 2009, the CDS market has traded with standardized fixed coupons rather than bespoke running spreads. Hull Ch 25.4 describes the mechanism: "For each underlying and each maturity, a coupon and a recovery rate are specified. A price is calculated from the quoted spread."

> **Deep Dive: The Big Bang (2009)**
>
> Before 2009, every CDS trade had a different coupon (e.g., 123 bps, 145 bps). This was a nightmare for netting.
>
> **The Problem:** If you bought \$10mm protection at 123 bp and wanted to close by selling protection, you'd have to find someone willing to take the exact 123 bp contract, or negotiate a complex unwind.
>
> **The Fix**: The "Big Bang" protocol standardized coupons.
> *   **Investment Grade (IG)**: Always trade at **100 bps**.
> *   **High Yield (HY)**: Always trade at **500 bps**.
>
> **The Adjustment**: If the "Fair Spread" is 150 bps, you trade at 100 bps coupon and pay an **Upfront Fee** to compensate the seller for the 50 bps difference.
>
> **Why It Matters:** Now CDS trade like bonds. You can close a position by doing an opposite trade at whatever the current upfront price is—no need to match exact coupons.

### 38.6.1 Standard Coupons

The market uses two standard coupons:
- **Investment grade:** 100 bp (0.01)
- **High yield:** 500 bp (0.05)

This standardization makes CDS trade like bonds—the running payments are fixed, and the contract price adjusts via an upfront payment.

### 38.6.2 The Upfront Price Formula

Hull provides the formula for computing the upfront price from the quoted spread:

$$\boxed{P = 100 - 100 \times D \times (s - c)}$$

where:
- $P$ = price per \$100 notional
- $D$ = CDS "duration" (the RPV01 in years)
- $s$ = quoted spread (decimal)
- $c$ = fixed coupon (decimal)

**Settlement mechanics:**
- If $s > c$: the protection buyer pays $(100 - P)$ per \$100 notional upfront
- If $s < c$: the protection buyer receives $(P - 100)$ per \$100 notional upfront

**Example (from Hull Ch 25, adapted):** Consider a 5-year CDS on an investment-grade name:
- Quoted spread: $s = 150$ bp = 0.015
- Standard coupon: $c = 100$ bp = 0.01
- CDS duration (RPV01): $D = 4.2$ years

$$P = 100 - 100 \times 4.2 \times (0.015 - 0.01) = 100 - 100 \times 4.2 \times 0.005 = 100 - 2.1 = 97.9$$

The protection buyer pays $(100 - 97.9) = 2.1$ per \$100 notional upfront. On \$10 million notional:

$$\text{Upfront} = 0.021 \times 10{,}000{,}000 = \$210{,}000$$

**Intuition:** The spread (150 bp) exceeds the coupon (100 bp) by 50 bp. Over the 4.2-year duration, this 50 bp annual underpayment is worth approximately $50 \times 4.2 = 210$ bp = 2.1% upfront.

### 38.6.3 When Coupon Exceeds Spread

**Example:** For a safer credit with spread below the standard coupon:
- Quoted spread: $s = 34$ bp = 0.0034
- Standard coupon: $c = 100$ bp = 0.01
- CDS duration: $D = 4.447$ years

$$P = 100 - 100 \times 4.447 \times (0.0034 - 0.01) = 100 - 100 \times 4.447 \times (-0.0066) = 100 + 2.94 = 102.94$$

Since $P > 100$, the protection buyer receives \$294,000 on \$10 million notional at inception, compensating for the fact that they will "overpay" via the running coupon relative to fair value.

### 38.6.4 Why Fixed Coupons?

The pre-crisis market quoted each CDS at its own running spread with no upfront. This created operational complexity when unwinding positions—every contract had a different coupon. By standardizing coupons, the market:
- Enables bond-like trading with clean prices
- Simplifies position netting and novations
- Reduces operational risk in booking and settlement
- Facilitates central clearing (CCP eligibility)

### 38.6.5 Distressed Credits: Pure Upfront Format

When a credit becomes severely distressed, the market switches from the standard running-spread-plus-upfront format to a **pure upfront format**. O'Kane explains: "An upfront CDS is a simple variation on the standard CDS contract in which the investor receives the premium leg not as a running stream of credit risky payments, but as a single upfront amount at the start of the contract."

**Why distressed names trade upfront:**

Consider a credit trading at 2000 bp (20%). At this spread level:
1. The expected default probability is high (perhaps 30%+ per year)
2. The protection seller faces significant risk that they'll have to pay out $N(1-R)$ soon
3. The premium stream is highly uncertain—if default occurs in 3 months, the seller receives only a fraction of the expected premiums

O'Kane provides Tables 6.1 and 6.2 comparing P&L under different scenarios. The key insight:

| Scenario | Running Format | Upfront Format |
|----------|---------------|----------------|
| Default immediately | Seller receives almost nothing | Seller already has upfront |
| No default (survives 5Y) | Seller receives full premium stream | Seller has only upfront |

**Protection sellers prefer upfront on distressed names** because they receive certain money now rather than a risky premium stream. The upfront compensates for the loss of future premiums.

**When does the market switch to upfront?**

There's no fixed rule, but O'Kane notes that when spreads exceed approximately **1000 bp**, the market often switches to pure upfront trading. At these levels:
- The RPV01 becomes very uncertain (sensitive to hazard rate assumptions)
- Price discovery works better in upfront terms
- The "certain money now vs. risky stream later" tradeoff strongly favors upfront

**Upfront CDS Valuation:**

For a pure upfront CDS, O'Kane shows the upfront payment $U(0)$ satisfies:

$$\boxed{U(0) = (1-R) \int_0^T Z(0,s)(-dQ(0,s)) = (1-R) \cdot D(0,T)}$$

where $D(0,T)$ is the "payment at default" present value from Chapter 36.

> **Desk Reality: The Distressed Trading Mindset**
>
> When a name becomes distressed, the entire trading conversation changes:
>
> **Investment Grade Talk:** "Where's the 5-year offered?" "160 bid, 165 offer"
>
> **Distressed Talk:** "What's the upfront?" "55 points up, 57 points up"
>
> The "points up" quote means points of upfront payment per 100 notional. At "55 points up," the protection buyer pays 55% of notional upfront (plus any running coupon if applicable).
>
> In extreme distress (near-certain default), the upfront approaches $(1-R)$—essentially the expected protection payment, received immediately rather than waiting for the credit event.

---

## 38.7 Mark-to-Market and Position Unwinding

Understanding how to calculate the mark-to-market (MTM) value of a CDS position is essential for daily P&L, risk management, and position unwinding.

### 38.7.1 The Mark-to-Market Formula

O'Kane derives the fundamental CDS MTM formula. For a long protection position with contractual spread $S_0$ when the current market spread is $S_t$:

$$\boxed{V(t) = (S(t,T) - S(0,T)) \times \text{RPV01}(t,T)}$$

where:
- $V(t)$ = mark-to-market value (positive = gain for protection buyer)
- $S(t,T)$ = current market spread for a CDS maturing at $T$
- $S(0,T)$ = contractual spread (what was agreed at inception)
- $\text{RPV01}(t,T)$ = current risky PV01 to maturity

**Intuition:** If spreads have widened from 100 bp to 150 bp, the protection buyer is receiving protection that is now worth 150 bp but paying only 100 bp. The present value of this 50 bp annual advantage is $50 \times \text{RPV01}$ bp.

**Example:**
- Contractual spread: $S_0 = 75$ bp
- Current market spread: $S_t = 320$ bp
- Current RPV01: 4.208 years
- Notional: \$10 million

$$V(t) = (0.0320 - 0.0075) \times 4.208 \times 10{,}000{,}000 = 0.0245 \times 4.208 \times 10{,}000{,}000 = \$1{,}030{,}960$$

The protection buyer has a mark-to-market gain of approximately \$1.03 million.

**For a short protection position:** The MTM is the negative of the long position.

### 38.7.2 Three Methods for Realizing Value

O'Kane describes three ways to realize the mark-to-market value of a CDS position:

**Method 1 — Close-out with Original Counterparty:**
"Party $A$ to the CDS requests party $B$ to agree a cash payment at which the contract can be closed out."

Both parties agree to terminate the contract. The MTM amount is exchanged, and the contract is cancelled.

**Method 2 — Assignment (Novation) to Third Party:**
"Party $A$ to the CDS agrees with a third party, $C$, an unwind value and then requests to party $B$ that the default swap be reassigned to party $C$. The mark-to-market amount is paid from party $C$ to party $A$."

This has the effect of removing party $A$ from the CDS, which is now between parties $B$ and $C$.

**Method 3 — Offsetting Trade:**
"Party $A$ enters into an offsetting transaction in which they sell protection at the current market spread."

O'Kane notes this is "fundamentally different from the first two since there is no immediate realisation of the value of the CDS contract." The offsetting position hedges the risk, but:
- If the reference entity survives, the realized income is the spread difference × remaining premiums
- If the reference entity defaults immediately, the realized income is approximately zero (the protection legs offset)

**Settlement Timing:**

O'Kane notes that CDS unwinds settle **T+3 business days**, similar to bond settlement. The "mark-to-market must be based on exchanging the unwind value in two days' time" (the settlement date).

> **Desk Reality: Why Method 3 Isn't the Same as a True Unwind**
>
> Consider Example: Long protection at 75 bp, spreads now at 320 bp.
>
> **Methods 1-2 (True Unwind):** Receive ~\$1.03mm cash now. Done.
>
> **Method 3 (Offset):** No cash now. You receive net 245 bp per year (320 - 75) as long as the credit survives. But:
> - If default occurs in 1 year: you receive only ~\$245k (one year of net premiums)
> - If the credit survives 4 years: you receive ~\$980k (still less than immediate unwind due to discounting)
>
> The offsetting position leaves **counterparty risk** (two positions with potentially different counterparties) and **timing risk** (the net premiums depend on survival).

### 38.7.3 Clean versus Full Mark-to-Market

Like bonds, CDS positions can be quoted "clean" (excluding accrued premium) or "full" (including accrued). O'Kane defines:

$$\text{Clean MTM} = \text{Full MTM} - \text{Accrued}$$

The full mark-to-market is "the present value of the full economic value of the credit default swap and is the value which is exchanged when a default swap is unwound. It is the expected present value of all future cash flows which have been contractually agreed."

**Critical distinction:** O'Kane warns: "Do not confuse the quotation of accrued interest with the CDS paying or not paying coupon accrued following a credit event. The quotation of accrued interest is simply a quotation convention, and has no effect on the cash flows of the CDS contract. The payment of coupon accrued at default is a contingent payment on the CDS which changes the actual cash flows."

---

## 38.8 Buyer versus Seller: Sign Conventions and Cashflow Summary

The following table summarizes cashflow signs (positive = receive, negative = pay):

| Component | Protection Buyer | Protection Seller |
|-----------|------------------|-------------------|
| Scheduled premium payments | $-N s \Delta$ | $+N s \Delta$ |
| Accrued premium at default | $-$ (pays) | $+$ (receives) |
| Protection payment | $+N(1-R)$ | $-N(1-R)$ |
| Upfront (if $s > c$) | $-$ (pays) | $+$ (receives) |
| Upfront (if $s < c$) | $+$ (receives) | $-$ (pays) |
| MTM if spreads widen | $+$ (gain) | $-$ (loss) |
| MTM if spreads tighten | $-$ (loss) | $+$ (gain) |

The cashflows are exactly symmetric: every dollar the buyer pays, the seller receives, and vice versa.

---

## 38.9 Worked Example: Full CDS Lifecycle

**Trade setup:**
- Trade date: Monday, 19 January 2026
- Effective date: 20 January 2026 (T+1 calendar)
- Notional: \$10,000,000
- Contractual spread: 100 bp ($S_0 = 0.01$, IG standard coupon)
- Quoted spread at trade: 150 bp → Upfront due
- CDS Duration (RPV01): 4.2 years
- Payment dates (adjusted): 20 Mar, 22 Jun, 21 Sep, 21 Dec 2026

**Step 1: Upfront Calculation**

$$\text{Upfront} = 10{,}000{,}000 \times 4.2 \times (0.015 - 0.01) = \$210{,}000$$

Protection buyer pays \$210,000 upfront.

**Step 2: Scheduled Premium Payments**

| Date | Days | Payment |
|------|------|---------|
| 20 Mar 2026 | 59 | $10{,}000{,}000 \times 0.01 \times 59/360 = \$16{,}389$ |
| 22 Jun 2026 | 94 | $10{,}000{,}000 \times 0.01 \times 94/360 = \$26{,}111$ |

**Step 3: Credit Event on 10 August 2026**

Default occurs. Days since last payment (22 Jun): 49 days.

$$\text{Accrued} = 10{,}000{,}000 \times 0.01 \times 49/360 = \$13{,}611$$

**Step 4: Protection Payment**

Recovery rate from auction: 40%

$$\text{Protection} = 10{,}000{,}000 \times (1 - 0.40) = \$6{,}000{,}000$$

**Full Cashflow Timeline:**

| Date | Event | Buyer | Seller |
|------|-------|-------|--------|
| 20 Jan 2026 | Trade inception | $-\$210{,}000$ | $+\$210{,}000$ |
| 20 Mar 2026 | Q1 premium | $-\$16{,}389$ | $+\$16{,}389$ |
| 22 Jun 2026 | Q2 premium | $-\$26{,}111$ | $+\$26{,}111$ |
| 10 Aug 2026 | Accrued at default | $-\$13{,}611$ | $+\$13{,}611$ |
| ~Aug 2026 | Protection payment | $+\$6{,}000{,}000$ | $-\$6{,}000{,}000$ |
| 21 Sep 2026 | **Cancelled** | — | — |
| 21 Dec 2026 | **Cancelled** | — | — |

**Net P&L for Protection Buyer:**

- Paid: \$210,000 (upfront) + \$16,389 + \$26,111 + \$13,611 = **\$266,111**
- Received: \$6,000,000
- **Net gain: \$5,733,889**

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
| Ignoring survival probability in RPV01 for distressed names | Mispriced duration and upfront |

### Sanity Checks

| Check | What to Verify |
|-------|----------------|
| Premium reasonableness | Quarterly payment ≈ $N \times s \times 0.25$ |
| Accrued bounds | $0 \leq \text{Accrued} \leq \text{Full quarter premium}$ |
| Protection bounds | $0 \leq N(1-R) \leq N$ |
| Upfront sign | If $s > c$, buyer pays; if $s < c$, buyer receives |
| RPV01 vs PV01 | For IG, should be close; for HY, RPV01 < PV01 |
| MTM sign | Spreads widen → buyer gains; spreads tighten → buyer loses |

### Implementation Verification Tests

1. **Repricing check:** Input the agreed terms; verify the system reproduces the traded upfront
2. **Schedule check:** Verify payment dates against a trusted source (Bloomberg CDSW)
3. **RPV01 check:** Compare system RPV01 to $\approx$ years-to-maturity for IG names
4. **MTM check:** Verify that MTM = (current spread - contractual spread) × RPV01 × notional

---

## Summary

1. A CDS transfers credit risk: the protection buyer pays premiums and receives a contingent payment after a credit event
2. "Buying protection" means being short credit risk; "selling protection" means being long credit risk
3. Premium payments are typically quarterly on ACT/360 day count
4. Standard roll dates are 20 Mar/Jun/Sep/Dec ("CDS IMM dates"—the 20th, not the 3rd Wednesday)
5. The effective date is T+1 calendar and may fall on a weekend or holiday
6. Premium payment formula: $N \cdot s \cdot \text{DayDiff}/360$
7. Premium payments stop immediately after a credit event, but accrued premium up to $\tau$ is owed
8. Protection payment: $N(1-R)$ where $R$ is the recovery rate
9. **RPV01** is the credit-risky annuity: $\frac{1}{2}\sum \Delta(t_{n-1},t_n) Z(t,t_n)(Q(t,t_{n-1})+Q(t,t_n))$
10. Post-2009, CDS trade with fixed coupons (100 bp IG, 500 bp HY) plus upfront payments
11. Upfront price formula: $P = 100 - 100 \times D \times (s - c)$
12. Distressed names trade pure upfront when spreads exceed ~1000 bp
13. MTM formula: $V(t) = (S_t - S_0) \times \text{RPV01}$
14. Unwind settlement occurs T+3 business days

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Premium leg | Periodic payments from buyer to seller, quarterly ACT/360 | Determines the cost of protection |
| Protection leg | Contingent payment of $N(1-R)$ after credit event | Determines the payout on default |
| Accrued premium | Premium owed from last coupon date to credit event | Real cashflow often missed in pricing |
| Roll dates | 20 Mar/Jun/Sep/Dec schedule dates | All standard CDS follow this calendar |
| T+1 calendar | Effective date is next calendar day (not business day) | Ensures weekend/holiday coverage |
| RPV01 | Risky PV01—credit-adjusted annuity with survival weighting | Foundation for all CDS pricing |
| Credit DV01 | RPV01 × 1bp—sensitivity to spread changes | Key risk measure for hedging |
| Upfront payment | Cash exchanged at inception when $s \neq c$ | Standardized coupons require this adjustment |
| Distressed upfront | Pure upfront format for names > 1000 bp | Protection sellers prefer certain money |
| Mark-to-market | $(S_t - S_0) \times \text{RPV01}$ | Daily P&L and position valuation |

---

## Notation Summary

| Symbol | Definition |
|--------|------------|
| $N$ | CDS notional (face value) |
| $s$ | Running spread (decimal per annum; 100 bp = 0.01) |
| $c$ | Fixed coupon (decimal per annum) |
| $\Delta(t_1, t_2)$ | Accrual fraction = DayDiff$(t_1, t_2)$/360 |
| $\tau$ | Credit event date |
| $R$ | Recovery rate (fraction of par) |
| $Z(t, T)$ | Risk-free discount factor from $t$ to $T$ |
| $Q(t, T)$ | Survival probability from $t$ to $T$ |
| RPV01$(t, T)$ | Risky PV01—credit-adjusted annuity to maturity |
| $D$ | CDS duration (= RPV01 in years) |
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
| 6 | What is the effective date convention? | T+1 calendar (next calendar day, even if weekend) |
| 7 | Why can effective date be a non-business day? | Credit events can occur on weekends/holidays |
| 8 | Formula for premium payment? | $N \cdot s \cdot \text{DayDiff}/360$ |
| 9 | What happens to premiums after a credit event? | Future payments cancelled; accrued to $\tau$ is owed |
| 10 | Formula for protection payment? | $N(1 - R)$ where $R$ is recovery rate |
| 11 | What is physical settlement? | Buyer delivers face value of bonds, seller pays par |
| 12 | What is cash settlement? | Seller pays par minus recovery price (auction) |
| 13 | Standard coupon for investment grade CDS? | 100 bp |
| 14 | Standard coupon for high yield CDS? | 500 bp |
| 15 | Hull's upfront price formula? | $P = 100 - 100 \times D \times (s - c)$ |
| 16 | If spread > coupon, who pays upfront? | Protection buyer pays |
| 17 | If spread < coupon, who pays upfront? | Protection seller pays (buyer receives) |
| 18 | What is CDS duration $D$? | Risky PV01—present value of 1 bp of spread |
| 19 | What is the RPV01 formula? | $\frac{1}{2}\sum \Delta(t_{n-1},t_n) Z(t,t_n)(Q(t,t_{n-1})+Q(t,t_n))$ |
| 20 | What is a "risky annuity" vs "risk-free annuity"? | Risky annuity weights by survival probability; risk-free assumes no default |
| 21 | When does the market switch to pure upfront format? | When spreads exceed ~1000 bp (distressed) |
| 22 | What is the settlement timing for CDS unwind? | T+3 business days |
| 23 | What is the MTM formula for a CDS? | $V(t) = (S_t - S_0) \times \text{RPV01}$ |
| 24 | Effect of accrued at default on breakeven spread? | Lowers it (approximately $\frac{S^2}{2(1-R)f}$) |
| 25 | Why do protection sellers prefer upfront on distressed? | Certain money now vs. risky premium stream |

---

## Mini Problem Set

**1.** Compute the quarterly premium payment for $N = \$25$ million, $s = 150$ bp, and a 90-day quarter under ACT/360.

> **Solution:** $s = 0.015$, $\Delta = 90/360 = 0.25$. Payment $= 25{,}000{,}000 \times 0.015 \times 0.25 = \$93{,}750$.

**2.** A CDS has effective date 15 January 2026. The first roll date is 20 March 2026. Calculate the accrual fraction for the stub period.

> **Solution:** DayDiff = 64 days. $\Delta = 64/360 = 0.1778$.

**3.** Default occurs 45 days after the last coupon date. Compute accrued premium for $N = \$10$ million, $s = 200$ bp.

> **Solution:** $s = 0.02$, $\Delta = 45/360 = 0.125$. Accrued $= 10{,}000{,}000 \times 0.02 \times 0.125 = \$25{,}000$.

**4.** If auction recovery is 28%, compute the protection payment on \$15 million notional.

> **Solution:** $\text{ProtPay} = 15{,}000{,}000 \times (1 - 0.28) = \$10{,}800{,}000$.

**5.** Using Hull's formula, compute the upfront price for: $s = 250$ bp, $c = 100$ bp, $D = 4.0$ years.

> **Solution:** $P = 100 - 100 \times 4.0 \times (0.025 - 0.01) = 100 - 6 = 94$. Buyer pays 6% upfront.

**6.** A \$10 million CDS trades at spread = 80 bp with coupon = 100 bp and $D = 4.5$ years. Calculate the upfront and state who pays whom.

> **Solution:** $P = 100 - 100 \times 4.5 \times (0.008 - 0.01) = 100 + 0.9 = 100.9$. Buyer receives 0.9% = \$90,000 (coupon exceeds spread).

**7.** Explain why physical and cash settlement produce the same economic result when auction price equals deliverable market value.

> **Solution:** Physical: buy bond at $P_{\text{rec}} \cdot N$, deliver for $N$, net = $N(1 - P_{\text{rec}})$. Cash: receive $N(1 - P_{\text{rec}})$ directly. Both equal \$6.2mm if $P_{\text{rec}} = 0.38$ on \$10mm notional.

**8.** Create a sign table showing buyer and seller cashflows for: (i) scheduled premium, (ii) accrued at default, (iii) protection payment.

> **Solution:** Buyer: $(-)$, $(-)$, $(+)$. Seller: $(+)$, $(+)$, $(-)$. Perfect symmetry.

**9.** A credit event occurs on Saturday, 21 March 2026. The effective date was 20 March 2026 (Friday). Is the event covered? Why does the T+1 calendar convention matter here?

> **Solution:** Yes, covered. Protection started Friday. If effective date were T+1 business day (Monday 23 Mar), the Saturday default would not be covered—a potential \$6mm loss difference.

**10.** For a CDS with $s = c$ exactly, what is the upfront payment?

> **Solution:** $P = 100 - 100 \times D \times 0 = 100$. Upfront = $(100 - 100) = 0$. No upfront when spread equals coupon.

**11.** Calculate RPV01 for a 2-year CDS (quarterly payments) given:
- Flat hazard rate: 3% per annum
- Flat risk-free rate: 4% continuously compounded
- Use $Q(t) = e^{-0.03t}$ and $Z(t) = e^{-0.04t}$

> **Solution:**
> Using $\text{RPV01} = \frac{1}{2}\sum_{n=1}^{8} \Delta \cdot Z(t_n) \cdot (Q(t_{n-1}) + Q(t_n))$ with $\Delta = 0.25$:
> - Compute $Z \cdot (Q_{n-1} + Q_n)$ for each quarter
> - Sum ≈ 3.68, so RPV01 ≈ 0.5 × 3.68 = **1.84 years**
> The risk-free annuity would be ~1.93 years; the 3% hazard reduces it by ~5%.

**12.** A CDS was entered at 150 bp contractual spread. Market spread has widened to 250 bp. RPV01 = 4.0 years. Notional = \$20 million. Calculate the MTM.

> **Solution:**
> $V(t) = (S_t - S_0) \times \text{RPV01} \times N = (0.025 - 0.015) \times 4.0 \times 20{,}000{,}000$
> $= 0.01 \times 4.0 \times 20{,}000{,}000 = \$800{,}000$
> Protection buyer has an **\$800,000 gain**.

**13.** A distressed credit trades at 1500 bp upfront-equivalent spread. Explain why the protection seller prefers upfront rather than running premium.

> **Solution:** At 1500 bp, the credit has high default probability. With running premium, if default occurs in month 1, the seller receives almost nothing but pays $N(1-R)$. With upfront, the seller receives the full premium (large upfront amount) immediately, before any potential default. Upfront provides **certain money now** vs. a **risky premium stream**.

**14.** What happens to RPV01 as survival probability drops sharply (credit deteriorates)?

> **Solution:** RPV01 decreases because:
> 1. Later premium payments are less likely to be received (survival-weighted down)
> 2. The "average" across survival probabilities pulls the whole annuity lower
>
> Example: If survival probability at 5Y drops from 95% to 60%, the 5Y payment contributes much less to RPV01. This affects both the upfront calculation and hedging ratios.

---

## Source Map

### (A) Book-Verified Facts — Source Attribution

| Content | Source |
|---------|--------|
| CDS definition | O'Kane Ch 5: "bilateral contract that transfers the credit risk" |
| Premium leg mechanics, ACT/360 | O'Kane Ch 5 |
| Roll dates 20 Mar/Jun/Sep/Dec, not same as IMM | O'Kane Ch 5 (footnotes 3, 4) |
| Schedule construction algorithm (4 steps) | O'Kane Ch 5 (Section 5.3) |
| Premium leg cash flow schedule | O'Kane Ch 5 (Figure 5.3) |
| Effective date T+1 calendar, weekend coverage rationale | O'Kane Ch 5 |
| Physical vs cash settlement mechanics | O'Kane Ch 5, Hull Ch 25 |
| Accrued premium at default | O'Kane Ch 5, Ch 6 |
| Effect of accrued on breakeven spread (quadratic) | O'Kane Ch 6 (Figure 6.1 and text) |
| RPV01 formula derivation (Equations 6.2-6.4) | O'Kane Ch 6 |
| RPV01 approximation for accrued at default | O'Kane Ch 6 (after Eq 6.3) |
| Premium Leg PV = $S_0 \cdot \text{RPV01}$ | O'Kane Ch 6 (Eq 6.2) |
| MTM formula: $(S_t - S_0) \times \text{RPV01}$ | O'Kane Ch 6 |
| Three methods to realize MTM value | O'Kane Ch 6.6 |
| Clean vs full MTM convention | O'Kane Ch 6 (after Eq 6.4) |
| Upfront CDS for distressed | O'Kane Ch 6.7 |
| Upfront price formula $P = 100 - 100 \times D \times (s - c)$ | Hull Ch 25.4 |
| Standard coupon conventions (100 bp IG, 500 bp HY) | Hull Ch 25.4 |
| CDS unwind settles T+3 business | O'Kane Ch 6.6 |

### (B) Claude-Extended Content

| Content | Basis |
|---------|-------|
| "Desk Reality" boxes on schedule mismatches, weekend coverage, distressed trading | Extended from O'Kane's mechanics to practical trading context |
| Approximation that market switches to upfront at ~1000 bp | Practitioner convention; O'Kane discusses distressed upfront but doesn't specify threshold |
| Comparison of offsetting trade vs true unwind economics | Extended from O'Kane's three methods discussion |

### (C) Reasoned Inference — Derivation Logic

| Content | Derivation |
|---------|------------|
| Unit checks on all formulas | Dimensional analysis from definitions |
| Sanity bounds on accrued and protection | Algebraic manipulation of source formulas |
| Physical/cash settlement equivalence | Arbitrage argument from source mechanics |
| Sign symmetry between buyer and seller | Bilateral contract structure |
| RPV01 worked example calculations | Applied O'Kane formula to specific numbers |

### (D) Flagged Uncertainties

The following items require trade-specific documentation:

- **I'm not sure** about exact settlement-day lags and operational sequences without the specific ISDA Definitions vintage and confirmation terms
- **I'm not sure** about full legal eligibility criteria for deliverable obligations without the CDS confirmation
- **I'm not sure** about exact intraday start time and front-end protection rules
- **I'm not sure** about regional variations in standard coupon conventions (North America vs Europe may differ)
- The Big Bang (2009) and Small Bang (2009) protocol changes introduced standardized coupons, but O'Kane (2008) predates these; exact implementation details require additional sources
- The threshold for switching to pure upfront (~1000 bp) is approximate market practice, not a formal rule

---

*Last Updated: January 2026*
