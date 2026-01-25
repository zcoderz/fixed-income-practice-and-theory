# Chapter 38: CDS Contract Mechanics — Premium Leg, Protection Leg, Coupons, and Accrued Premium

---

## Conventions & Notation

| Convention | Description |
|------------|-------------|
| **Contract type** | Single-name CDS unless explicitly stated; mechanics apply similarly to index CDS with additional notional-reduction features (not fully developed here) |
| **Rate quoting** | Running premium ("CDS spread") is quoted in bp per annum; convert to decimal via $s = S_{\text{bp}} / 10{,}000$ |
| **Payment frequency & day count** | Premium is typically quarterly on an Actual/360 basis (source-backed default) |
| **Standard roll dates** | 20 Mar, 20 Jun, 20 Sep, 20 Dec, with CDS-market "IMM" meaning the 20th (not necessarily the official IMM date) |
| **Business-day adjustment** | Dates are adjusted to avoid weekends/holidays, typically rolled forward to next business day ("Following"), using a specified holiday calendar (typical, per source) |
| **Effective date convention** | Effective date is the calendar day after trade date ("T+1 calendar"), and need not be a business day (source-backed) |
| **Credit event vs default** | "Credit event" is the legal trigger; practitioners often say "default" loosely. Exact event definitions depend on ISDA Definitions and are trade-specific (see "I'm not sure" notes below) |

---

## Setup

### Conventions Used in This Chapter

- Premium payments are quarterly and use ACT/360 unless explicitly stated otherwise (this is the typical convention described in the primary reference)
- Contract maturity dates use quarterly roll conventions on 20 Mar/Jun/Sep/Dec (CDS-market "IMM" dates)
- Premium payments terminate at maturity or immediately following a credit event, with accrued premium owed up to the credit event date
- Protection is settled either via physical settlement (deliver eligible obligations for par) or cash settlement (pay par minus recovery price), with recovery price determined via dealer poll or auction per the primary source's description

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $N$ | CDS notional / face value (USD, e.g., $10{,}000{,}000$) |
| $S_{\text{bp}}$ | Running premium in bp per annum (e.g., 100 bp) |
| $s = S_{\text{bp}} / 10{,}000$ | Running premium in decimal per annum (e.g., 100 bp $\to 0.01$) |
| $t_{\text{tr}}$ | Trade date |
| $t_{\text{eff}}$ | Effective date (typically $t_{\text{tr}} + 1$ calendar day) |
| $t_i$ | Scheduled premium payment dates, $i = 1, \ldots, n$; set $t_0 = t_{\text{eff}}$. (Schedule construction described in the primary source.) |
| $\text{DayDiff}(a, b)$ | Number of calendar days between dates $a$ and $b$ |
| $\alpha(a, b)$ | Accrual fraction; for ACT/360, $\alpha(a, b) = \text{DayDiff}(a, b) / 360$ |
| $\tau$ | Credit event date/time (often called "default time" in market language) |
| $R$ | Recovery rate as a fraction of par (e.g., $0.40$) |
| $P_{\text{rec}}$ | Recovery price (cash-settlement price) as fraction of par (e.g., 38 per 100 $\to 0.38$) |

---

## Core Concepts

### 1.1 Credit Default Swap (CDS)

**Formal Definition:**

A CDS is a bilateral contract that transfers the credit risk of a reference entity from one party to another for a specified period; it is designed to protect against loss from par on a bond or loan in exchange for a premium.

**Intuition:**

Think: insurance-like cashflows on a reference entity—regular premium payments unless a credit event occurs; if a credit event occurs, the protection seller compensates the buyer for the loss from par.

**Trading/Risk/Portfolio Practice:**

- Used to hedge cash-credit exposure without selling the bond/loan ("synthetic" risk transfer)
- Used to express a directional credit view (tightening/widening and jump-to-default exposure)

---

### 1.2 Contract Parties and "Direction"

**Formal Definition:**

- **Protection buyer:** pays premium; receives contingent protection payment after a credit event
- **Protection seller:** receives premium; pays contingent protection payment after a credit event

**Intuition:**

Buying protection is economically like shorting the credit (you benefit if credit deteriorates/defaults); selling protection is like being long the credit.

**Trading/Risk/Portfolio Practice:**

- Desk language: "buying a CDS" typically means buying protection (short credit risk), which is opposite the cash-bond convention where "buying" means long credit

---

### 1.3 Premium Leg (Running Premium Leg)

**Formal Definition:**

The premium leg is the series of periodic payments made by the protection buyer to the protection seller. Payments are typically quarterly using Actual/360; they terminate at maturity or immediately following a credit event.

**Intuition:**

You pay a "subscription fee" for protection, pro-rated by the length of each accrual period.

**Trading/Risk/Portfolio Practice:**

- Booking requires: notional $N$, running spread $s$, payment schedule $t_i$, and day count

---

### 1.4 Protection Leg

**Formal Definition:**

The protection leg is the contingent payment made by the protection seller to the protection buyer "to make up to par" the value of a deliverable obligation following a credit event.

**Intuition:**

After default, the defaulted debt is worth $R$ (recovery). The CDS pays the difference between par and recovery (economically).

**Trading/Risk/Portfolio Practice:**

- Determines jump-to-default exposure and default loss settlement workflow (physical vs cash)

---

### 1.5 Accrued Premium and "Coupon Accrued at Default"

**Formal Definition:**

In the standard contract description in the primary reference, following a credit event the protection buyer must pay the fraction of premium that has accrued since the previous premium payment date.

**Intuition:**

Premium is paid in arrears. If default happens mid-period, the seller provided protection for part of the period, so the buyer owes that pro-rated premium.

**Trading/Risk/Portfolio Practice:**

- Critical for P&L explain: scheduled premium payments stop at default, but one final accrued amount is typically due

---

### 1.6 Quoting Convention: "Clean" vs "Full" MTM (Accrued Interest as a Quoting Adjustment)

**Formal Definition:**

A "clean" market value excludes the premium accrued since the last coupon date; the primary reference defines

$$\text{Clean MTM} = \text{Full MTM} - \text{Accrued}$$

with accrued sign depending on whether one is long or short protection.

**Intuition:**

Clean quoting avoids a jump in quoted MTM as you pass through coupon dates, even though cash actually moves on coupon dates.

**Trading/Risk/Portfolio Practice:**

- Don't confuse (i) quoting accrued (a market-quoting convention) with (ii) coupon accrued at default (a real contingent cashflow)

---

### 1.7 Physical vs Cash Settlement (and Auctions)

**Formal Definition:**

Two settlement methods described in the primary reference:

- **Physical settlement:** buyer delivers face value of deliverable obligations; seller pays face value in cash
- **Cash settlement:** seller pays face value minus the recovery price of the reference obligation; recovery price determined by dealer poll or auction

**Intuition:**

Both aim to deliver the same economic compensation: the difference between par and the market value (recovery value) of defaulted debt.

**Trading/Risk/Portfolio Practice:**

- Cash settlement is often operationally easier when deliverables are scarce; the primary reference notes a motivation for cash settlement is the difficulty of sourcing deliverable assets when protection notional is large relative to deliverables
- A secondary reference describes that "as is now usual" there is cash settlement and an ISDA-organized two-stage auction is used to determine the mid-market value of the cheapest deliverable bond several days after the credit event

---

## Math and Derivations

### 2.1 Premium Payment Cashflow (Scheduled Coupons)

From the primary reference, under ACT/360 the premium payment at time $t_2$ per \$1 of face value is

$$s \times \frac{\text{DayDiff}(t_1, t_2)}{360}$$

where $t_1$ is the previous premium date, $t_2$ is the payment date, and $s$ is the CDS spread (decimal per annum).

Scaling up to notional $N$:

$$\boxed{\text{PremiumPay}(t_2) = N \cdot s \cdot \alpha(t_1, t_2), \quad \alpha(t_1, t_2) = \frac{\text{DayDiff}(t_1, t_2)}{360}}$$

**Unit Check:**

| Component | Unit |
|-----------|------|
| $N$ | dollars |
| $s$ | 1/year |
| $\alpha$ | years |
| $N \cdot s \cdot \alpha$ | dollars ✓ |

**Sanity Checks:**

- If $\text{DayDiff} = 0$, payment $= 0$
- If $\text{DayDiff} = 90$ days, $\alpha = 0.25$ and payment $\approx 0.25$ of annual premium

---

### 2.2 Accrued Premium at Default (Credit Event Between Payment Dates)

**Mechanic (source-backed):** after a credit event, the protection buyer pays the premium accrued since the previous coupon date.

Let default occur at $\tau$ with $t_{i-1} < \tau \leq t_i$. The accrued premium cashflow (paid at/after the credit event, depending on settlement conventions) is:

$$\boxed{\text{AccruedPrem}(\tau) = N \cdot s \cdot \alpha(t_{i-1}, \tau) = N \cdot s \cdot \frac{\text{DayDiff}(t_{i-1}, \tau)}{360}}$$

**Unit Check:**

Same as scheduled premium: dollars ✓

**Sanity Checks:**

- $0 \leq \text{DayDiff}(t_{i-1}, \tau) \leq \text{DayDiff}(t_{i-1}, t_i)$
- Therefore $0 \leq \text{AccruedPrem}(\tau) \leq N s \alpha(t_{i-1}, t_i)$

---

### 2.3 Premium Payments Stop at Default

The premium leg payments terminate at maturity or immediately following a credit event (source-backed).

**Mechanically:**

- Scheduled payments at dates after $\tau$ are cancelled
- The only premium-related cashflow after $\tau$ is typically the accrued premium up to $\tau$

---

### 2.4 Protection Payment Cashflow

**Protection leg described (source-backed):** the contingent payment makes up to par the value of a deliverable obligation, via physical or cash settlement.

A mechanics-level representation:

**Cash settlement version (using settlement/recovery price):**

Let $P_{\text{rec}}$ be the recovery price as a fraction of par (e.g., 38 per 100 $\to 0.38$).

$$\boxed{\text{ProtPay} = N(1 - P_{\text{rec}})}$$

This matches the source description "face value minus recovery price."

**Recovery-rate version (if $R$ is used as recovery fraction of par):**

$$\boxed{\text{ProtPay} = N(1 - R)}$$

This is consistent with the primary reference's discussion of loss-on-default being a fraction $(1 - R)$ of face value.

**Unit Check:**

| Component | Unit |
|-----------|------|
| $N$ | dollars |
| $1 - R$ or $1 - P_{\text{rec}}$ | dimensionless |
| payoff | dollars ✓ |

**Bounds Sanity Check:**

If $0 \leq R \leq 1$, then $0 \leq \text{ProtPay} \leq N$

---

### 2.5 Timing Assumptions (Be Explicit)

The sources state the legs' payments occur following a credit event and describe auction/poll determination of recovery price, but do not provide a single universal "T+X days" settlement lag for all contracts in the primary reference excerpt.

A secondary reference notes auctions determine value several days after the credit event and describes a "two-stage" auction process.

**I'm not sure** about the exact settlement-day lag and operational sequence for your specific trade without:

- the ISDA Definitions vintage,
- the credit-event determination and settlement terms in the confirmation,
- the market (single-name vs index) and currency/region.

---

### 2.6 (Very Short Preview) Where This Enters Pricing Later

Mechanically, pricing later compares:

- PV of expected premium-leg payments (including expected accrued-at-default component) vs
- PV of expected protection payment

*This chapter does not derive par spread, survival curves, or risky PV01.*

---

## Measurement & Risk

### A) Contract Parties and Direction

**Protection buyer vs protection seller:**

- Protection buyer pays premiums and receives a contingent payment after a credit event
- Protection seller receives premiums and pays contingent protection

**What "buying protection" means (cashflows and risk):**

- Long protection $\Rightarrow$ short credit risk
- Short protection $\Rightarrow$ long credit risk
- **Terminology caution:** "buying a CDS" often means buying protection (short credit), opposite the cash bond market

**Notional conventions:**

- Cashflows are often quoted per \$1 of face value in modeling exposition (then scaled by $N$)
- Desk booking: notional commonly \$5mm, \$10mm, \$25mm; always specify currency and notional

---

### B) The Two Legs and Their Cashflow Logic

#### Premium Leg

Fixed coupon rate (running premium) × notional × accrual fraction:

$$\text{PremiumPay}(t_i) = N \cdot s \cdot \alpha(t_{i-1}, t_i), \quad \alpha = \frac{\text{DayDiff}}{360}$$

ACT/360 and DayDiff mechanics are explicitly described in the primary reference.

#### Payment Schedule

Premium dates are typically quarterly, and the contract maturity dates align with quarterly roll dates (20 Mar/Jun/Sep/Dec).

**Schedule-building algorithm (source-backed):**

1. Maturity date is first CDS "IMM" date $T$ years after effective date
2. Step back in 3-month increments
3. Adjust dates to avoid weekends/holidays (typically roll forward), using a specified holiday calendar

#### How Premium Payments Stop at Default

- Premium payments terminate immediately following a credit event
- The accrued portion since the last premium date is paid to the seller (contingent accrued)

---

#### Protection Leg

**Contingent payment after credit event:**

The protection leg compensates the buyer to par for a deliverable obligation after a credit event.

**Payoff form (recovery/settlement price):**

- **Physical settlement:** deliver obligations for par (exchange face for face)
- **Cash settlement:** seller pays par minus recovery price (reference obligation), determined by dealer poll or auction

**Timing of protection payment:**

**I'm not sure** of a universal timing rule from the primary reference excerpt alone; the secondary reference indicates auction-based pricing several days after the credit event.

---

### C) Accrued Premium on Default (Must Be Explicit)

Because premiums are paid in arrears, if default occurs between payment dates, accrued premium is owed up to the default date (source-backed).

**Generic accrued-premium calculation (ACT/360 form supported by source mechanics):**

$$\boxed{\text{AccruedPrem}(\tau) = N \cdot s \cdot \frac{\text{DayDiff}(t_{i-1}, \tau)}{360}}$$

where $t_{i-1}$ is the previous coupon date and $\tau$ is the credit event date.

---

### D) Settlement Methods (Mechanics, Not Pricing)

#### Physical Settlement

**Deliver an eligible obligation for par:**

The primary reference describes a basket of deliverable bonds/loans can be delivered and physical settlement involves delivering face value and receiving face value in cash.

**"Deliverable" definition:**

**I'm not sure** of the full legal eligibility criteria (deliverable obligation characteristics, ranking tests, maturity cutoffs, etc.) without the specific CDS confirmation and ISDA Definitions.

#### Cash Settlement

**Payoff determined using a market value/recovery measure:**

Cash settlement pays par minus the recovery price of the reference obligation; recovery price determined by dealer poll or auction.

**Auction process mention:**

- **Primary reference:** auction is one method to determine the cash settlement price (also notes dealer poll)
- **Secondary reference:** describes a two-stage auction process and refers to the cheapest deliverable bond valued several days after the credit event
- **Index context:** ISDA introduced a 2005 protocol allowing fallback to cash settlement with an auction method when deliverables are scarce relative to protection notional (index context)

#### Why Physical and Cash Settlement Should Align (Economic Logic)

If the cash-settlement price equals the market value of what could be delivered physically, then:

- Physical settlement gives the buyer the right to hand over something worth $P_{\text{rec}} N$ and receive $N$
- Net economic gain $= N - P_{\text{rec}} N = N(1 - P_{\text{rec}})$

which matches cash settlement.

---

### E) Standardization and Quoting (Mechanics Only)

#### Running Spread / Running Premium

Quoted as annualised spread (bp per annum); payments typically quarterly and ACT/360 in the primary reference's mechanics.

#### Upfront Trades

**Distressed-name upfront CDS (primary reference):** upfront CDS pays premium leg as a single upfront amount at the start, rather than a running stream; protection leg remains the same.

#### Standard Coupon + Upfront (Bond-Like Trading Convention)

A secondary/optional reference describes a convention where for each underlying and maturity a coupon is specified and an upfront price is exchanged; the buyer then pays the coupon on payment dates, facilitating bond-like trading.

**I'm not sure** whether your intended "modern standard coupon + upfront" regime matches this description exactly for the market you care about without specifying the ISDA Definitions vintage and market (single-name vs index, region, etc.).

#### What Is Agreed at Trade Date (Mechanics-Level Booking Inputs)

- Reference entity / contract reference
- Notional $N$
- Maturity date and coupon schedule (often tied to 20 Mar/Jun/Sep/Dec roll dates)
- Running spread $S_{\text{bp}}$ (and/or fixed coupon + upfront, depending on regime)
- Effective date (typically trade date + 1 calendar day)
- Settlement method (physical vs cash)
- Day count and business-day adjustment/calendar (ACT/360; Following; holiday calendar)

#### Important Dates

| Date | Description |
|------|-------------|
| Trade date $t_{\text{tr}}$ | Date the trade is executed |
| Effective date $t_{\text{eff}}$ | Usually the calendar day following trade date (T+1 calendar) |
| Maturity date | Standard contracts align with quarterly roll dates |
| Credit event date $\tau$ | Date the credit event occurs |
| Settlement date | Date protection payment is made |

**(Index-specific from primary reference)** settlement: index contract cash settles three days after trade date, when upfront is paid.

**I'm not sure** whether this "three days later" applies to the single-name CDS mechanics you want without additional sourcing in your chosen market conventions.

#### Protection Start ("Front-End Protection")

The primary reference explicitly notes effective date is T+1 calendar and can fall on a weekend/holiday, and that there is value in protection starting on a non-business day because defaults can occur on weekends/holidays.

**I'm not sure** of the exact intraday start time and detailed front-end protection rules without the specific ISDA/confirmation terms.

---

### F) Keep It Focused

This chapter does not build a survival curve, does not derive fair spread, and does not compute risky PV01. It teaches how to book the cashflows and place them on a timeline.

---

## Worked Examples

### Global Toy Conventions for Examples A–L (Stated Explicitly)

| Parameter | Value |
|-----------|-------|
| Notional $N$ | \$10,000,000 |
| Running coupon $S_{\text{bp}}$ | 100 bp $\Rightarrow s = 0.0100$ |
| Day count | ACT/360 (source-backed as typical) |
| Payment dates | Quarterly roll dates on the 20th, adjusted with "Following" for weekends (holiday effects ignored for the toy example) |

---

### Example A: Premium Payment Amount

**Given:**

- $N = \$10{,}000{,}000$
- $s = 0.01$ (100 bp)
- Quarterly accrual fraction $\alpha = 0.252778$ (e.g., 91 days / 360)

**Compute:**

$$\text{PremiumPay} = N \cdot s \cdot \alpha = 10{,}000{,}000 \times 0.01 \times 0.252778$$

**Step-by-step:**

1. Annual premium: $N \cdot s = 10{,}000{,}000 \times 0.01 = 100{,}000$
2. Period premium: $100{,}000 \times 0.252778 = 25{,}277.80$ (rounded)

$$\boxed{\text{Answer: } \$25{,}277.78 \text{ (approx)}}$$

**Unit check:** $N$ dollars × $s$ 1/year × $\alpha$ years = dollars ✓

---

### Example B: Build a Coupon Schedule

**Given:**

- Trade date $t_{\text{tr}} =$ Mon 19 Jan 2026
- Effective date $t_{\text{eff}} =$ Tue 20 Jan 2026 (T+1 calendar per source convention)
- Use standard roll dates: 20 Mar, 20 Jun, 20 Sep, 20 Dec (CDS "IMM" dates)
- Apply "Following" for weekends (toy: ignore holidays)

**Generate four quarterly payment dates (unadjusted):**

| Unadjusted Date | Day of Week | Adjusted Date |
|-----------------|-------------|---------------|
| 20 Mar 2026 | Friday | 20 Mar 2026 |
| 20 Jun 2026 | Saturday | 22 Jun 2026 (Mon) |
| 20 Sep 2026 | Sunday | 21 Sep 2026 (Mon) |
| 20 Dec 2026 | Sunday | 21 Dec 2026 (Mon) |

**Compute accrual fractions $\alpha = \text{DayDiff} / 360$:**

| Period | Date Range | DayDiff | $\alpha$ |
|--------|------------|---------|----------|
| 1 | 20 Jan → 20 Mar | 59 days | $59/360 = 0.1638889$ |
| 2 | 20 Mar → 22 Jun | 94 days | $94/360 = 0.2611111$ |
| 3 | 22 Jun → 21 Sep | 91 days | $91/360 = 0.2527778$ |
| 4 | 21 Sep → 21 Dec | 91 days | $91/360 = 0.2527778$ |

**(Optional) Compute coupon amounts for each period using $N s \alpha_i$:**

| Period | Calculation | Amount |
|--------|-------------|--------|
| 1 | $100{,}000 \times 0.1638889$ | \$16,388.89 |
| 2 | $100{,}000 \times 0.2611111$ | \$26,111.11 |
| 3 | $100{,}000 \times 0.2527778$ | \$25,277.78 |
| 4 | $100{,}000 \times 0.2527778$ | \$25,277.78 |

$$\boxed{\text{Payment dates (adjusted): 20 Mar 2026, 22 Jun 2026, 21 Sep 2026, 21 Dec 2026}}$$
$$\boxed{\text{Accrual fractions: } 0.1638889, \; 0.2611111, \; 0.2527778, \; 0.2527778}$$

---

### Example C: Accrued Premium on Default

**Assume:**

- Previous coupon date $t_{i-1} =$ 22 Jun 2026
- Default/credit event occurs $\tau =$ 10 Aug 2026
- DayDiff from 22 Jun to 10 Aug = 49 days (toy calendar count)
- ACT/360

**Compute accrual fraction:**

$$\alpha(t_{i-1}, \tau) = 49 / 360 = 0.1361111$$

**Compute accrued premium:**

$$\text{AccruedPrem}(\tau) = N s \alpha = 10{,}000{,}000 \times 0.01 \times 0.1361111$$

**Step-by-step:**

1. Annual premium $= 100{,}000$
2. Accrued $= 100{,}000 \times 0.1361111 = 13{,}611.11$

$$\boxed{\text{Answer: } \$13{,}611.11}$$

**Sanity check:** Accrued $<$ full quarter premium ($\approx \$25k$) ✓

---

### Example D: Premium Leg Cashflows Stop at Default

**Using Example B schedule and Example C default:**

**Scheduled premium dates:**

| Date | Status |
|------|--------|
| 20 Mar 2026 | Paid if no default before |
| 22 Jun 2026 | Paid if no default before |
| 21 Sep 2026 | **Cancelled** because default on 10 Aug 2026 occurs before this date |
| 21 Dec 2026 | **Cancelled** |

**What remains payable at default:**

- Accrued premium from 22 Jun 2026 to 10 Aug 2026 (Example C): **\$13,611.11**

$$\boxed{\text{Answer: No scheduled coupons after } \tau \text{; one accrued payment: } \$13{,}611.11}$$

*(Mechanic source-backed: premium payments terminate at credit event and accrued since prior coupon is paid.)*

---

### Example E: Protection Leg Payoff from Recovery

**Assume:**

- Notional $N = \$10{,}000{,}000$
- Recovery rate $R = 40\% = 0.40$

**Compute:**

$$\text{ProtPay} = N(1 - R) = 10{,}000{,}000 \times (1 - 0.40) = 6{,}000{,}000$$

$$\boxed{\text{Answer: } \$6{,}000{,}000}$$

**Bounds check:** Between 0 and $N$ ✓

---

### Example F: Cash Settlement with Auction Final Price

**Assume** auction final price is 38 per 100 of face $\Rightarrow P_{\text{rec}} = 0.38$.

**Compute implied recovery:**

$$R = 0.38$$

**Protection payment:**

$$\text{ProtPay} = N(1 - P_{\text{rec}}) = 10{,}000{,}000 \times (1 - 0.38) = 6{,}200{,}000$$

$$\boxed{\text{Answer: } \$6{,}200{,}000}$$

*(Auction use is described in sources; operational steps beyond this are not added here.)*

---

### Example G: Physical Settlement Economics

**Assume:**

- Deliverable bond with face $N = \$10{,}000{,}000$ trades at 38 per 100
- Market value to buy bond $= 0.38 \times 10{,}000{,}000 = \$3{,}800{,}000$

**Physical settlement described:** buyer delivers face value; seller pays face value in cash.

**Economic steps (toy, ignoring funding/frictions):**

1. Protection buyer buys deliverable bond in market: $-\$3{,}800{,}000$
2. Delivers bond for par to protection seller; receives $+\$10{,}000{,}000$
3. Net gain: $10{,}000{,}000 - 3{,}800{,}000 = \$6{,}200{,}000$

**Compare to cash settlement at 38:**

Cash settlement payoff $= N(1 - 0.38) = \$6{,}200{,}000$ (Example F)

$$\boxed{\text{Answer: Physical and cash settlement match economically at } \$6.2\text{mm if price} = 38}$$

---

### Example H: Upfront + Running Coupon Mechanics

Sources support bond-like "fixed coupon + upfront" mechanics (optional reference) where buyer pays an upfront amount and then pays a fixed coupon on payment dates.

**Assume (toy, not derived):**

- Notional $N = \$10{,}000{,}000$
- Fixed coupon $c = 100$ bp $\Rightarrow 0.01$
- Upfront = 3.50% of notional paid at settlement
- First coupon accrual fraction from Example B: $\alpha_1 = 0.1638889$

**Upfront at settlement:**

$$\text{UpfrontCash} = 0.035 \times 10{,}000{,}000 = \$350{,}000$$

**First coupon payment:**

$$\text{Coupon}_1 = N \cdot c \cdot \alpha_1 = 10{,}000{,}000 \times 0.01 \times 0.1638889 = \$16{,}388.89$$

$$\boxed{\text{At inception/settlement: buyer pays } \$350{,}000 \text{ upfront}}$$
$$\boxed{\text{At first coupon date: buyer pays } \$16{,}388.89 \text{ coupon}}$$

**I'm not sure** how the upfront is computed from the quoted market spread in your exact regime without stepping into pricing/bootstrapping; here it is taken as given.

---

### Example I: Buyer vs Seller Sign Conventions

**Define cashflow sign as "positive = receive cash, negative = pay cash."**

| Component | Protection Buyer (Long Protection) | Protection Seller (Short Protection) |
|-----------|-----------------------------------|-------------------------------------|
| Scheduled premium payments $N s \alpha(t_{i-1}, t_i)$ | $-$ | $+$ |
| Accrued premium at default $N s \alpha(t_{i-1}, \tau)$ | $-$ | $+$ |
| Protection payment $N(1 - R)$ or $N(1 - P_{\text{rec}})$ | $+$ | $-$ |
| Upfront (if applicable) | depends on quote; in Example H buyer pays upfront $-$ | $+$ |

**Symmetry check:** buyer cashflows are exactly the negative of seller cashflows (ignoring counterparty risk and fees).

---

### Example J: Timeline Diagram with Cashflows

**Using Example B schedule and Example C default:**

```
Trade date     Effective      Coupon 1        Coupon 2        Default τ         (Settlement)      Coupon 3        Coupon 4
19 Jan 2026 -> 20 Jan 2026 -> 20 Mar 2026 -> 22 Jun 2026 -> 10 Aug 2026 -> (few days later?) -> 21 Sep 2026 -> 21 Dec 2026
                 |              |               |               |                   |                |               |
Protection starts |      Pay premium if alive    Pay premium     Pay accrued prem     Pay protection   CANCELLED       CANCELLED
                  \______________________________________________________________/
                                   premiums in arrears → accrued owed at τ
```

**Notes:**

- Effective date as T+1 calendar is source-backed
- Exact settlement lag after credit event is trade-specific; secondary source indicates "several days after" for auction-based cash settlement

---

### Example K: Day Count Sensitivity Toy

Primary reference emphasizes ACT/360 as typical for premium leg. An optional reference notes CDS/index quotes are often expressed in ACT/360 and can be converted to "actual/actual" equivalents.

**Using Example C (49 days accrued, $N = \$10$mm, $s = 0.01$):**

**ACT/360:**

$$\alpha_{360} = 49/360 = 0.1361111, \quad \text{Accrued}_{360} = 100{,}000 \times 0.1361111 = 13{,}611.11$$

**ACT/365 (toy proxy for "actual/actual" style):**

$$\alpha_{365} = 49/365 = 0.1342466, \quad \text{Accrued}_{365} = 100{,}000 \times 0.1342466 = 13{,}424.66$$

**Difference:**

$$13{,}611.11 - 13{,}424.66 = 186.45$$

$$\boxed{\$13{,}611.11 \text{ (ACT/360) vs } \$13{,}424.66 \text{ (ACT/365 toy)}; \; \Delta = \$186.45}$$

**I'm not sure** which "actual/actual" convention you intend (there are multiple), so ACT/365 is used only as a conceptual warning.

---

### Example L: Common Documentation Ambiguity Toy

**Primary source note:** effective date is T+1 calendar and can be weekend/holiday; there is value to starting on a non-business day since defaults can occur on weekends/holidays.

**Toy scenario:**

- Notional $N = \$10{,}000{,}000$
- Recovery $R = 0.40 \Rightarrow$ protection payment if covered: $N(1 - R) = \$6{,}000{,}000$
- Coupon $s = 0.01$
- Suppose a credit event occurs on 20 Jan 2026

**Case 1: Effective date is 20 Jan 2026 (T+1 calendar)**

- Protection is "in effect" on that date (conceptually)
- Potential protection payment: \$6,000,000

**Case 2: Effective date were one day later (hypothetical T+2 calendar)**

- Protection might not cover that day

**Accrued premium sensitivity to a 1-day shift (ACT/360 toy):**

$$\Delta\alpha = 1/360 = 0.0027778 \Rightarrow \Delta\text{Accrued} = N s \Delta\alpha = 10{,}000{,}000 \times 0.01 \times 0.0027778 = 277.78$$

$$\boxed{\text{One-day shift can change accrued premium by about } \$277.78 \text{ (toy)}}$$
$$\boxed{\text{Coverage change could be as large as the protection payment } \$6{,}000{,}000 \text{ if a credit event occurs on the boundary day}}$$

**I'm not sure** about the exact legal coverage boundary (intraday rules, what constitutes "occur on the effective date," etc.) without the specific ISDA Definitions and trade confirmation.

---

## Practical Notes

### 5.1 Contract Inputs Checklist (What You Must Know to Book the Trade)

- [ ] Reference entity (and reference obligation if relevant to cash settlement pricing)
- [ ] Notional $N$
- [ ] Direction: buy or sell protection (long/short protection)
- [ ] Trade date, effective date (T+1 calendar)
- [ ] Maturity date / roll date convention (20 Mar/Jun/Sep/Dec)
- [ ] Premium payment dates and frequency (typically quarterly)
- [ ] Day count (typically ACT/360)
- [ ] Business day rule and holiday calendar (often Following; specify calendar)
- [ ] Settlement type: physical vs cash; recovery price determination method (dealer poll/auction)
- [ ] Whether contract includes coupon accrued at default (standard feature in primary reference description)

---

### 5.2 Common Pitfalls

| Pitfall | Description |
|---------|-------------|
| Forgetting accrued premium on default | A real contingent cashflow |
| Sign confusion (buyer vs seller) | "Buying CDS" = buying protection |
| Mixing bond "clean/dirty" intuition incorrectly into CDS | CDS has both quoting accrued (clean MTM convention) and accrued-at-default (real cashflow) |
| Misaligning dates | Trade date vs effective date (T+1 calendar); roll dates and stubs due to business day adjustments |
| Assuming auction / credit-event rules without sourcing | Exact protocol and operational steps depend on ISDA terms (trade-specific) |

---

### 5.3 Implementation Pitfalls (Systems)

**Date schedule generation:**

- Build backwards in 3-month steps on CDS "IMM" dates and then apply business day adjustments; specify the holiday calendar

**Handling default between coupon dates:**

- Cancel future coupons, compute accrued-to-$\tau$, and book protection cashflow

**Consistent units:**

- bp vs decimal (100 bp = 0.01)
- "price per 100" vs fraction (38 per 100 = 0.38)

---

### 5.4 Verification Tests (Quick Desk Sanity)

| Test | Formula / Condition |
|------|---------------------|
| Premium cashflows | $\text{PremiumPay} = N s \alpha$ with $\alpha = \text{DayDiff}/360$ (ACT/360) |
| Accrued premium bounds | $0 \leq \text{AccruedPrem}(\tau) \leq \text{PremiumPay}(t_i)$ |
| Protection payoff bounds | $0 \leq N(1 - R) \leq N$ for $0 \leq R \leq 1$ |
| Buyer/seller symmetry | Cashflows are negatives of each other (sign table in Example I) |

---

## Summary & Recall

### 6.1 Ten-Bullet Executive Summary

1. A CDS transfers credit risk: buyer pays premium; seller pays contingent protection after credit event
2. "Buying protection" is economically short credit risk; selling protection is long credit risk
3. Premium leg payments are typically quarterly on ACT/360
4. Standard roll dates are 20 Mar/Jun/Sep/Dec ("CDS IMM" dates)
5. Effective date is typically T+1 calendar day; may be weekend/holiday
6. Scheduled premium at $t_i$: $N s \alpha(t_{i-1}, t_i)$ with $\alpha = \text{DayDiff}/360$
7. Premium payments stop immediately after a credit event, but accrued premium up to $\tau$ is owed
8. Protection leg can be settled physically (deliver for par) or in cash (par minus recovery price via poll/auction)
9. Clean vs full MTM is a quoting convention; don't confuse it with accrued-at-default cashflows
10. Exact ISDA credit event and settlement details are trade-specific; document terms matter

---

### 6.2 Cheat Sheet: Key Cashflow Identities + Sign Table

#### Identities

$$\boxed{\text{Scheduled premium at } t_i: \quad \text{PremiumPay}(t_i) = N \cdot s \cdot \frac{\text{DayDiff}(t_{i-1}, t_i)}{360}}$$

$$\boxed{\text{Accrued premium at default } \tau \in (t_{i-1}, t_i]: \quad \text{AccruedPrem}(\tau) = N \cdot s \cdot \frac{\text{DayDiff}(t_{i-1}, \tau)}{360}}$$

$$\boxed{\text{Protection payoff (cash-settlement): } \quad \text{ProtPay} = N(1 - P_{\text{rec}}) = N(1 - R)}$$

#### Sign Table (Buyer of Protection vs Seller of Protection)

| Component | Buyer (Long Prot) | Seller (Short Prot) |
|-----------|-------------------|---------------------|
| Scheduled premium | $-$ | $+$ |
| Accrued at default | $-$ | $+$ |
| Protection payment | $+$ | $-$ |

---

### 6.3 Flashcards (35 Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a CDS? | A bilateral contract transferring credit risk; buyer pays premium, seller pays contingent protection after a credit event |
| 2 | Who is the protection buyer? | Pays the premium leg and receives protection payment after a credit event |
| 3 | Who is the protection seller? | Receives premiums and pays the protection leg after a credit event |
| 4 | What is the premium leg? | A series of periodic premium payments, typically quarterly on ACT/360 |
| 5 | What is the protection leg? | Contingent payment making up to par the value of a deliverable obligation after a credit event |
| 6 | What does "long protection" mean economically? | Short credit risk |
| 7 | What does "short protection" mean economically? | Long credit risk |
| 8 | Typical CDS premium frequency? | Quarterly |
| 9 | Typical CDS premium day count? | Actual/360 |
| 10 | Standard CDS roll/maturity dates? | 20 Mar, 20 Jun, 20 Sep, 20 Dec |
| 11 | What is the effective date convention described? | Calendar day after trade date (T+1 calendar) |
| 12 | Can the effective date be a non-business day? | Yes; it may fall on a Saturday/holiday |
| 13 | Why might starting protection on a non-business day matter? | Defaults can occur on weekends/holidays, so there is value in coverage starting then |
| 14 | When do premium payments terminate? | At maturity or immediately after a credit event |
| 15 | Why is there accrued premium at default? | Premium is paid in arrears; default can occur mid-period |
| 16 | Who pays accrued premium at default? | Protection buyer pays it to the protection seller |
| 17 | Formula for scheduled premium payment? | $N s \cdot \text{DayDiff}/360$ under ACT/360 |
| 18 | Formula for accrued premium at default? | $N s \cdot \text{DayDiff}(t_{i-1}, \tau)/360$ |
| 19 | What is physical settlement? | Buyer delivers face value deliverables; seller pays face value in cash |
| 20 | What is cash settlement? | Seller pays par minus recovery price of reference obligation |
| 21 | How is recovery price determined in cash settlement (per sources)? | By dealer poll or auction process |
| 22 | What is the "delivery option"? | Buyer can choose which asset from a basket of deliverables to deliver, effectively being long an option |
| 23 | Why allow cash settlement? | To overcome difficulty sourcing deliverables when protection notional is large vs deliverables |
| 24 | What is clean MTM? | Full MTM minus accrued premium (quoting convention) |
| 25 | Accrued sign for long protection in clean MTM convention? | Negative (because the long-protection position pays coupon) |
| 26 | Accrued sign for short protection in clean MTM convention? | Positive (because short protection receives coupon) |
| 27 | Should you confuse clean-MTM accrued with coupon accrued at default? | No—quoting accrued is a convention; accrued-at-default is a real contingent cashflow |
| 28 | What is an upfront CDS (distressed-name variation)? | Premium leg is a single upfront amount at the start; protection leg unchanged |
| 29 | When might market switch to upfront format (per source)? | When issuer is in distress and spreads are very high (order of 1000 bp) |
| 30 | What is a fixed coupon + upfront trading arrangement (optional ref)? | Coupon is specified; upfront exchanged; buyer then pays coupon on payment dates |
| 31 | How do you convert "38 per 100" to a fraction? | $0.38$ |
| 32 | Protection payoff in terms of recovery price fraction $P_{\text{rec}}$? | $N(1 - P_{\text{rec}})$ |
| 33 | What does "credit event" mean? | Legal trigger for protection payment; often loosely called default |
| 34 | What is the key operational effect of default on premium leg? | Future coupons cancel; accrued-to-default is due |
| 35 | Minimum booking inputs for CDS cashflows? | Notional, spread/coupon, effective date, schedule/dates, day count, settlement type, direction (buy/sell protection) |

---

## Mini Problem Set (18 Questions)

*Increasing difficulty. Solution sketches provided for questions 1–9 only.*

---

**1.** Compute the quarterly premium payment for $N = \$25$mm, $S = 150$ bp, $\alpha = 0.25$.

> **Sketch:** Convert 150 bp $\to 0.015$. Payment $= 25{,}000{,}000 \times 0.015 \times 0.25$.

---

**2.** For ACT/360, compute $\alpha$ for a period of 92 days.

> **Sketch:** $\alpha = 92/360$.

---

**3.** Using Example B schedule, recompute the coupon amount for the second period if $S = 80$ bp and $N = \$10$mm.

> **Sketch:** $s = 0.008$. Use $\alpha_2 = 94/360$. Payment $= N s \alpha_2$.

---

**4.** Default occurs 30 days after the last coupon date. Compute accrued premium for $N = \$10$mm, $S = 200$ bp under ACT/360.

> **Sketch:** $s = 0.02$. $\alpha = 30/360$. Accrued $= N s \alpha$.

---

**5.** Show numerically that accrued premium at default is always less than or equal to the full coupon for that period.

> **Sketch:** Use $\text{DayDiff}(t_{i-1}, \tau) \leq \text{DayDiff}(t_{i-1}, t_i)$ and multiply by $N s / 360$.

---

**6.** If recovery is 25%, compute the protection payment on $N = \$5$mm.

> **Sketch:** $N(1 - R) = 5{,}000{,}000 \times 0.75$.

---

**7.** If auction final price is 12 per 100, compute protection payment on $N = \$10$mm.

> **Sketch:** $P_{\text{rec}} = 0.12$. Payoff $= 10{,}000{,}000 \times 0.88$.

---

**8.** Create a sign table for buyer/seller of protection for scheduled premium, accrued premium, and protection payment.

> **Sketch:** Buyer pays premiums (negative), receives protection (positive); seller opposite.

---

**9.** Explain in one paragraph why physical and cash settlement should match economically if the cash settlement price equals deliverable market value.

> **Sketch:** Replicate Example G logic: buy deliverable at $P_{\text{rec}} N$, deliver for $N$, net $= N(1 - P_{\text{rec}})$.

---

**10.** Build a 6-payment quarterly schedule from a given effective date and a maturity aligned to a roll date; include weekend adjustments.

---

**11.** Given a default date between coupons, list all cashflows that occur and which are cancelled.

---

**12.** For a fixed coupon + upfront regime, explain what cash moves at inception and on each coupon date (mechanics only).

---

**13.** Explain the difference between "clean MTM" accrued and "accrued premium at default."

---

**14.** Given two different business-day adjustment rules ("Following" vs "Modified Following"), show how coupon dates and accrual fractions can differ (conceptual).

---

**15.** Describe what additional information is needed to fully specify deliverable obligations for physical settlement.

---

**16.** For an index CDS, explain how notional changes after constituent defaults (mechanics only).

---

**17.** Explain how a one-day change in effective date can affect (i) accrued premium and (ii) protection coverage in boundary cases.

---

**18.** List at least five trade terms that must be checked in the confirmation before booking CDS cashflows.

---

## Source Map

### (A) Verified Facts — Source Attribution

| Content | Source |
|---------|--------|
| CDS definition, premium/protection leg mechanics | O'Kane Ch 5 (primary reference) |
| ACT/360 day count, quarterly payments | O'Kane Ch 5 |
| Effective date T+1 calendar, roll date conventions | O'Kane Ch 5 |
| Physical vs cash settlement mechanics | O'Kane Ch 5, Hull Ch 25 |
| Auction process description | O'Kane Ch 5 (secondary reference notes) |
| Premium accrued at default | O'Kane Ch 5 |
| Clean vs full MTM convention | O'Kane Ch 5 |

### (B) Reasoned Inference — Derivation Logic

| Content | Derivation |
|---------|------------|
| Unit checks on all formulas | Dimensional analysis from definitions |
| Sanity bound checks | Algebraic manipulation of source formulas |
| Physical/cash settlement equivalence | Arbitrage argument from source mechanics |

### (C) Speculation — Flagged Uncertainties

All "I'm not sure" statements are preserved and cover:

- Exact settlement-day lag and operational sequence (trade-specific)
- Full legal eligibility criteria for deliverable obligations
- Universal timing rule for protection payment
- "Modern standard coupon + upfront" regime matching specific markets
- Index-specific "three days later" settlement applicability to single-name
- Exact intraday start time and front-end protection rules
- Exact legal coverage boundary for credit events on boundary days

---

*Last Updated: January 2026*
