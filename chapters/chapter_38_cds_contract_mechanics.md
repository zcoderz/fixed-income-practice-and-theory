# Chapter 38: CDS Contract Mechanics

---

## Introduction

Before you can price a credit default swap, hedge a corporate bond position, or explain P&L after a credit event, you must understand exactly what cashflows move between the two parties and when. A CDS may function economically like credit insurance, but it is contractually a derivative with precise mechanics that differ from insurance in critical ways.

Consider a portfolio manager holding \$50 million of XYZ Corp bonds who wants to hedge the default risk. She calls her dealer and "buys protection" on XYZ via a 5-year CDS at 150 basis points. What has she agreed to? She will pay quarterly premiums based on that spread, but the exact payment dates follow CDS-specific conventions (the 20th of March, June, September, December—not the bond's coupon dates). If XYZ experiences a credit event mid-quarter, her premium payments stop, but she owes accrued premium up to the event date. The protection payment she receives depends on whether the contract settles physically or via an auction-determined cash price.

Getting any of these details wrong—the day count, the effective date, the accrued premium at default—creates P&L breaks, failed reconciliations, and hedging errors. The CDS market has standardized many conventions, but the standardization itself requires understanding: why T+1 calendar effective dates? Why do roll dates fall on the 20th rather than month-end? Why does the market now trade with fixed coupons plus upfront payments rather than running spreads alone?

This chapter covers the mechanics of CDS contracts without yet deriving fair spreads or survival curves. We establish the premium leg and protection leg cashflows, work through date conventions and settlement methods, and explain the upfront-plus-coupon trading regime. Chapter 39 will examine credit event definitions and settlement procedures in detail, while Chapter 41 will develop the pricing framework that values these cashflows.

---

## 38.1 The Credit Default Swap: Definition and Economic Logic

O'Kane defines a CDS as "a bilateral contract that transfers the credit risk of a reference entity from one party to another for a specified period." More specifically, it is designed to protect against loss from par on a bond or loan in exchange for a premium.

The contract has two parties with opposite exposures:

**The protection buyer** pays periodic premiums and receives a contingent payment if a credit event occurs. Economically, buying protection is equivalent to shorting the credit—the buyer profits if the reference entity's creditworthiness deteriorates or it defaults.

**The protection seller** receives premiums and makes a contingent payment after a credit event. Selling protection is equivalent to being long the credit risk—the seller profits if the entity survives and spreads tighten.

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

For a notional $N$, running spread $s$ (in decimal form, so 100 bp = 0.01), and accrual fraction $\alpha$, the premium payment at date $t_i$ is:

$$\boxed{\text{PremiumPay}(t_i) = N \cdot s \cdot \alpha(t_{i-1}, t_i)}$$

where the accrual fraction under ACT/360 is:

$$\alpha(t_{i-1}, t_i) = \frac{\text{DayDiff}(t_{i-1}, t_i)}{360}$$

**Unit check:** $N$ (dollars) × $s$ (1/year) × $\alpha$ (years) = dollars. The spread is quoted per annum, and the accrual fraction converts the annual rate to the period's portion.

**Example (O'Kane-style):** Consider a \$10 million notional CDS with a 100 bp running spread. For a 91-day quarter:

$$\text{PremiumPay} = 10{,}000{,}000 \times 0.01 \times \frac{91}{360} = 10{,}000{,}000 \times 0.01 \times 0.2528 = \$25{,}278$$

The annual premium is \$100,000; the quarterly payment is roughly one-quarter of that, adjusted for actual days.

### 38.2.2 Payment Schedule and Roll Dates

CDS payment dates follow a standardized schedule based on **roll dates**: 20 March, 20 June, 20 September, and 20 December. These are often called "CDS IMM dates," though O'Kane notes they are not the same as official IMM dates used for Eurodollar futures.

The schedule-building algorithm works backwards from maturity:

1. The maturity date is the first roll date (20 Mar/Jun/Sep/Dec) that is at least $T$ years after the effective date
2. Step back in 3-month increments to generate payment dates
3. Adjust each date to avoid weekends and holidays using a "Following" convention (roll forward to the next business day) with a specified holiday calendar

**Example:** For a trade date of Monday, 19 January 2026:
- Effective date: 20 January 2026 (T+1 calendar)
- First payment date: 20 March 2026 (Friday—no adjustment needed)
- Second payment date: 20 June 2026 falls on Saturday → adjusted to 22 June 2026 (Monday)

The accrual fraction for the first period (20 Jan to 20 Mar) is 59/360 = 0.1639, not the standard 90/360 because the contract started mid-cycle.

### 38.2.3 Effective Date: T+1 Calendar

O'Kane explicitly states that the effective date is "the calendar day after trade date (T+1 calendar)," and importantly, "need not be a business day." Why allow protection to start on a weekend or holiday?

The answer is that credit events can occur on any day. A company can file for bankruptcy on a Saturday. If the effective date were pushed to Monday, a credit event on Sunday would fall outside the protection period. As O'Kane notes, "there is value in protection starting on a non-business day because defaults can occur on weekends/holidays."

---

## 38.3 The Protection Leg: Contingent Payment After Credit Event

The protection leg is the contingent payment from seller to buyer that "makes up to par the value of a deliverable obligation following a credit event." This payment only occurs if a credit event happens during the protection period.

### 38.3.1 Protection Payment Formula

If the recovery price (the post-default market value of the reference obligation) is $P_{\text{rec}}$ as a fraction of par, the protection payment is:

$$\boxed{\text{ProtPay} = N(1 - P_{\text{rec}})}$$

Equivalently, if recovery rate $R$ represents the fraction of par recovered:

$$\text{ProtPay} = N(1 - R)$$

**Bounds check:** For $0 \leq R \leq 1$, the protection payment lies between 0 (full recovery) and $N$ (zero recovery).

**Example:** On \$10 million notional with recovery of 40%:

$$\text{ProtPay} = 10{,}000{,}000 \times (1 - 0.40) = \$6{,}000{,}000$$

The protection buyer receives \$6 million, compensating for the loss from par on the reference obligations.

### 38.3.2 Settlement Methods

O'Kane and Hull describe two settlement methods:

**Physical settlement:** The protection buyer delivers face value of deliverable obligations to the protection seller, who pays face value in cash. If the buyer can source bonds trading at 38 cents on the dollar, they pay \$3.8 million to acquire \$10 million face value, deliver these bonds, and receive \$10 million from the seller—a net gain of \$6.2 million.

**Cash settlement:** The protection seller pays par minus the recovery price directly. If the recovery price is 38%, the seller pays $10{,}000{,}000 \times (1 - 0.38) = \$6{,}200{,}000$.

Both methods should produce the same economic result when the cash settlement price equals the market value of deliverable obligations. Hull notes that "as is now usual" there is cash settlement via an ISDA-organized auction process to determine the recovery price.

**Why cash settlement became standard:** O'Kane observes that physical settlement creates problems "when protection notional is large relative to deliverables." After major credit events, protection notionals often exceed the outstanding deliverable bonds, making physical delivery impractical. The auction mechanism solves this by establishing a single recovery price for all CDS contracts.

---

## 38.4 Accrued Premium at Default

When a credit event occurs between payment dates, the protection buyer owes accrued premium for the portion of the period during which protection was in force. This is a real cashflow, not just an accounting convention.

### 38.4.1 The Accrued Premium Formula

If default occurs at time $\tau$ with $t_{i-1} < \tau \leq t_i$:

$$\boxed{\text{AccruedPrem}(\tau) = N \cdot s \cdot \frac{\text{DayDiff}(t_{i-1}, \tau)}{360}}$$

**Example:** Using the schedule from Section 38.2.2, suppose default occurs on 10 August 2026. The previous payment date was 22 June 2026, so 49 days have elapsed:

$$\text{AccruedPrem} = 10{,}000{,}000 \times 0.01 \times \frac{49}{360} = \$13{,}611$$

**Sanity check:** The accrued amount (\$13,611) is less than a full quarter's premium (\$25,278), as expected since default occurred roughly halfway through the period.

### 38.4.2 Premium Payments Stop at Default

O'Kane is explicit: "Premium payments terminate immediately following a credit event." After default:
- All future scheduled payments are cancelled
- The accrued premium up to $\tau$ is owed (this is the final premium cashflow)
- The protection payment is triggered

This differs from a bond, where coupons may continue accruing through a grace period. In CDS, the credit event is a hard stop.

---

## 38.5 The Upfront-Plus-Coupon Trading Regime

Since 2009, the CDS market has traded with standardized fixed coupons rather than bespoke running spreads. Hull Ch 25.4 describes the mechanism: "For each underlying and each maturity, a coupon and a recovery rate are specified. A price is calculated from the quoted spread."

> **Deep Dive: The Big Bang (2009)**
>
> Before 2009, every CDS trade had a different coupon (e.g., 123 bps, 145 bps). This was a nightmare for netting.
> *   **The Fix**: The "Big Bang" protocol standardized coupons.
> *   **Investment Grade (IG)**: Always trade at **100 bps**.
> *   **High Yield (HY)**: Always trade at **500 bps**.
> *   **The Adjustment**: If the "Fair Spread" is 150 bps, you trade at 100 bps coupon and pay an **Upfront Fee** to compensate the seller for the 50 bps difference.

### 38.5.1 Standard Coupons

The market uses two standard coupons:
- **Investment grade:** 100 bp (0.01)
- **High yield:** 500 bp (0.05)

This standardization makes CDS trade like bonds—the running payments are fixed, and the contract price adjusts via an upfront payment.

### 38.5.2 The Upfront Price Formula

Hull provides the formula for computing the upfront price from the quoted spread:

$$\boxed{P = 100 - 100 \times D \times (s - c)}$$

where:
- $P$ = price per \$100 notional
- $D$ = CDS "duration" (the risky PV01, expressed in years)
- $s$ = quoted spread (decimal)
- $c$ = fixed coupon (decimal)

**Settlement mechanics:**
- If $s > c$: the protection buyer pays $(100 - P)$ per \$100 notional upfront
- If $s < c$: the protection buyer receives $(P - 100)$ per \$100 notional upfront

**Example (from Hull Ch 25, adapted):** Consider a 5-year CDS on an investment-grade name:
- Quoted spread: $s = 150$ bp = 0.015
- Standard coupon: $c = 100$ bp = 0.01
- CDS duration: $D = 4.2$ years (computed from survival probabilities)

$$P = 100 - 100 \times 4.2 \times (0.015 - 0.01) = 100 - 100 \times 4.2 \times 0.005 = 100 - 2.1 = 97.9$$

The protection buyer pays $(100 - 97.9) = 2.1$ per \$100 notional upfront. On \$10 million notional:

$$\text{Upfront} = 0.021 \times 10{,}000{,}000 = \$210{,}000$$

**Intuition:** The spread (150 bp) exceeds the coupon (100 bp) by 50 bp. Over the 4.2-year duration, this 50 bp annual underpayment is worth approximately $50 \times 4.2 = 210$ bp = 2.1% upfront.

### 38.5.3 When Coupon Exceeds Spread

**Example:** For a safer credit with spread below the standard coupon:
- Quoted spread: $s = 34$ bp = 0.0034
- Standard coupon: $c = 40$ bp = 0.004 (or using IG standard of 100 bp)
- CDS duration: $D = 4.447$ years

$$P = 100 - 100 \times 4.447 \times (0.0034 - 0.004) = 100 + 0.27 = 100.27$$

Since $P > 100$, the protection buyer receives \$27,000 on \$10 million notional at inception, compensating for the fact that they will "overpay" via the running coupon relative to fair value.

### 38.5.4 Why Fixed Coupons?

The pre-crisis market quoted each CDS at its own running spread with no upfront. This created operational complexity when unwinding positions—every contract had a different coupon. By standardizing coupons, the market:
- Enables bond-like trading with clean prices
- Simplifies position netting and novations
- Reduces operational risk in booking and settlement

---

## 38.6 Clean versus Full Mark-to-Market

Like bonds, CDS positions can be quoted "clean" (excluding accrued premium) or "full" (including accrued). O'Kane defines:

$$\text{Clean MTM} = \text{Full MTM} - \text{Accrued}$$

The sign of the accrued adjustment depends on position direction:
- **Long protection (buyer):** pays coupon, so accrued is subtracted from full to get clean
- **Short protection (seller):** receives coupon, so accrued is added

**Critical distinction:** Do not confuse this quoting convention with accrued premium at default. Clean/full MTM is an accounting convention for marking positions daily. Accrued premium at default is a real contingent cashflow that must be paid when a credit event occurs.

---

## 38.7 Buyer versus Seller: Sign Conventions and Cashflow Summary

The following table summarizes cashflow signs (positive = receive, negative = pay):

| Component | Protection Buyer | Protection Seller |
|-----------|------------------|-------------------|
| Scheduled premium payments | $-N s \alpha$ | $+N s \alpha$ |
| Accrued premium at default | $-$ (pays) | $+$ (receives) |
| Protection payment | $+N(1-R)$ | $-N(1-R)$ |
| Upfront (if $s > c$) | $-$ (pays) | $+$ (receives) |
| Upfront (if $s < c$) | $+$ (receives) | $-$ (pays) |

The cashflows are exactly symmetric: every dollar the buyer pays, the seller receives, and vice versa.

---

## 38.8 Worked Example: Full CDS Cashflow Timeline

**Trade setup:**
- Trade date: Monday, 19 January 2026
- Effective date: 20 January 2026 (T+1 calendar)
- Notional: \$10,000,000
- Running spread: 100 bp (s = 0.01)
- Payment dates (adjusted): 20 Mar, 22 Jun, 21 Sep, 21 Dec 2026
- Credit event occurs: 10 August 2026
- Recovery rate: 40%

**Cashflows that occur:**

| Date | Event | Buyer Cashflow | Seller Cashflow |
|------|-------|----------------|-----------------|
| 20 Mar 2026 | Q1 coupon (59 days) | $-\$16{,}389$ | $+\$16{,}389$ |
| 22 Jun 2026 | Q2 coupon (94 days) | $-\$26{,}111$ | $+\$26{,}111$ |
| 10 Aug 2026 | Default (49 days accrued) | $-\$13{,}611$ | $+\$13{,}611$ |
| ~Aug 2026 | Protection payment | $+\$6{,}000{,}000$ | $-\$6{,}000{,}000$ |
| 21 Sep 2026 | **Cancelled** | — | — |
| 21 Dec 2026 | **Cancelled** | — | — |

**Net result for buyer:** Paid \$56,111 in premiums, received \$6,000,000 in protection. Net gain: \$5,943,889 (before transaction costs and present-value adjustments).

---

## 38.9 Practical Notes

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

### Sanity Checks

| Check | What to Verify |
|-------|----------------|
| Premium reasonableness | Quarterly payment ≈ $N \times s \times 0.25$ |
| Accrued bounds | $0 \leq \text{Accrued} \leq \text{Full quarter premium}$ |
| Protection bounds | $0 \leq N(1-R) \leq N$ |
| Upfront sign | If $s > c$, buyer pays; if $s < c$, buyer receives |

---

## Summary

1. A CDS transfers credit risk: the protection buyer pays premiums and receives a contingent payment after a credit event
2. "Buying protection" means being short credit risk; "selling protection" means being long credit risk
3. Premium payments are typically quarterly on ACT/360 day count
4. Standard roll dates are 20 Mar/Jun/Sep/Dec (CDS IMM dates)
5. The effective date is T+1 calendar and may fall on a weekend or holiday
6. Premium payment formula: $N \cdot s \cdot \text{DayDiff}/360$
7. Premium payments stop immediately after a credit event, but accrued premium up to $\tau$ is owed
8. Protection payment: $N(1-R)$ where $R$ is the recovery rate
9. Settlement can be physical (deliver obligations for par) or cash (pay par minus auction price)
10. Post-2009, CDS trade with fixed coupons (100 bp IG, 500 bp HY) plus upfront payments
11. Upfront price formula: $P = 100 - 100 \times D \times (s - c)$

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Premium leg | Periodic payments from buyer to seller, quarterly ACT/360 | Determines the cost of protection |
| Protection leg | Contingent payment of $N(1-R)$ after credit event | Determines the payout on default |
| Accrued premium | Premium owed from last coupon date to credit event | Real cashflow often missed in pricing |
| Roll dates | 20 Mar/Jun/Sep/Dec schedule dates | All standard CDS follow this calendar |
| T+1 calendar | Effective date is next calendar day (not business day) | Ensures weekend/holiday coverage |
| Upfront payment | Cash exchanged at inception when $s \neq c$ | Standardized coupons require this adjustment |
| CDS duration ($D$) | Risky PV01 used in upfront formula | Converts spread-coupon difference to price |

---

## Notation Summary

| Symbol | Definition |
|--------|------------|
| $N$ | CDS notional (face value) |
| $s$ | Running spread (decimal per annum; 100 bp = 0.01) |
| $c$ | Fixed coupon (decimal per annum) |
| $\alpha(t_1, t_2)$ | Accrual fraction = DayDiff$(t_1, t_2)$/360 |
| $\tau$ | Credit event date |
| $R$ or $P_{\text{rec}}$ | Recovery rate/price (fraction of par) |
| $D$ | CDS duration (risky PV01 in years) |
| $P$ | Upfront price per 100 notional |

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

---

## Mini Problem Set

**1.** Compute the quarterly premium payment for $N = \$25$ million, $s = 150$ bp, and a 90-day quarter under ACT/360.

> **Solution:** $s = 0.015$, $\alpha = 90/360 = 0.25$. Payment $= 25{,}000{,}000 \times 0.015 \times 0.25 = \$93{,}750$.

**2.** A CDS has effective date 15 January 2026. The first roll date is 20 March 2026. Calculate the accrual fraction for the stub period.

> **Solution:** DayDiff = 64 days. $\alpha = 64/360 = 0.1778$.

**3.** Default occurs 45 days after the last coupon date. Compute accrued premium for $N = \$10$ million, $s = 200$ bp.

> **Solution:** $s = 0.02$, $\alpha = 45/360 = 0.125$. Accrued $= 10{,}000{,}000 \times 0.02 \times 0.125 = \$25{,}000$.

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

---

## Source Map

### (A) Verified Facts — Source Attribution

| Content | Source |
|---------|--------|
| CDS definition, premium/protection leg mechanics | O'Kane Ch 5 |
| ACT/360 day count, quarterly payments | O'Kane Ch 5 |
| Roll dates 20 Mar/Jun/Sep/Dec | O'Kane Ch 5 |
| Effective date T+1 calendar, weekend coverage rationale | O'Kane Ch 5 |
| Physical vs cash settlement mechanics | O'Kane Ch 5, Hull Ch 25 |
| Accrued premium at default | O'Kane Ch 5 |
| Auction process for cash settlement | O'Kane Ch 5, Hull Ch 25 |
| Clean vs full MTM convention | O'Kane Ch 5 |
| Upfront price formula $P = 100 - 100 \times D \times (s - c)$ | Hull Ch 25.4 |
| Standard coupon conventions (100 bp IG, 500 bp HY) | Hull Ch 25.4 |
| Upfront CDS for distressed names | O'Kane Ch 6.7 |

### (B) Reasoned Inference — Derivation Logic

| Content | Derivation |
|---------|------------|
| Unit checks on all formulas | Dimensional analysis from definitions |
| Sanity bounds on accrued and protection | Algebraic manipulation of source formulas |
| Physical/cash settlement equivalence | Arbitrage argument from source mechanics |
| Sign symmetry between buyer and seller | Bilateral contract structure |

### (C) Flagged Uncertainties

The following items require trade-specific documentation:

- **I'm not sure** about exact settlement-day lags and operational sequences without the specific ISDA Definitions vintage and confirmation terms
- **I'm not sure** about full legal eligibility criteria for deliverable obligations without the CDS confirmation
- **I'm not sure** about exact intraday start time and front-end protection rules
- **I'm not sure** about regional variations in standard coupon conventions (North America vs Europe may differ)
- The Big Bang (2009) and Small Bang (2009) protocol changes introduced standardized coupons, but O'Kane (2008) predates these; exact implementation details require additional sources

---

*Last Updated: January 2026*
