# Chapter 1: Market Quoting, Calendars, and Cashflow Plumbing

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- Bonds are typically quoted **clean** (a "flat/quoted price") and the **invoice/full/dirty** amount exchanged is **clean + accrued interest**
- With the accrued-interest convention, if yield is unchanged then the **quoted (clean) price does not mechanically drop** on the coupon date; the drop occurs in the **full** price via accrued resetting
- Accrued interest is defined as the portion of the next coupon that has accrued since the previous coupon date, computed via a day-count fraction that depends on the market convention
- US Treasury bond prices are quoted in **32nds** (per $100 par); "cash/dirty price = quoted/clean price + accrued interest"
- Day-count conventions are commonly written **X/Y** and determine the fraction of a coupon/reference-period interest earned between two dates
- USD LIBOR-style money-market conventions include **Actual/360**, and spot LIBOR has a **two-business-day** start delay; USD LIBOR uses **Modified Following** date rolling
- In USD vanilla swaps (typical market convention), schedule dates are rolled (often Modified Following); floating fixings are typically **2 business days before** the accrual start; fixed-leg year fraction is commonly **30/360** and floating-leg year fraction commonly **Actual/360**
- Bond settlement lags vary by instrument (examples: some bonds T+1; Eurobonds T+3)

### (B) Reasoned Inference (Derived from A)

- If you mix conventions (e.g., compute accrued on ACT/ACT but discount/price on a different basis) you will create apparent "P&L" that is purely plumbing noise; consistent conventions reconcile PV from price vs PV from yield/spread
- PV should be monotone decreasing in discount rates (sanity check) because discount factors shrink as rates rise; violations are typically implementation bugs (sign, timing, or compounding mismatch)

### (C) Speculation (Clearly Labeled; Minimal)

- **End-of-month (EOM) roll rules** are market- and contract-specific (e.g., ISDA schedule generation). The exact EOM rule wording for specific products requires the desk's contract standard (e.g., "EOM = true" conventions) or an explicit reference section

---

## Chapter Outline

### Prerequisites

- Time value of money: discounting and present value (PV)
- Basic bond cashflows (coupon + principal) and basic derivative cashflows (fixed vs floating legs)
- Comfort with dates, day counts, and simple arithmetic

### Learning Objectives

By the end of this chapter you should be able to:

1. Identify what is being quoted (price vs yield vs spread) and convert quotes into **cashflows** and **invoice amounts**
2. Build/verify a cashflow schedule: trade date vs settlement/effective date, payment dates, fixing dates, and business day rolling
3. Compute accrued interest, and explain **clean vs dirty** pricing and their P&L implications across coupon dates
4. Treat day count and compounding as a "units system" and convert between simple/discrete/continuous compounding consistently
5. Run practitioner sanity checks (monotonicity, boundary conditions, reconciliation PV-from-price vs PV-from-yield)

### Key Notation & Conventions

#### Dates

| Symbol | Definition |
|--------|------------|
| $t_0$ | Trade date |
| $t_s$ | Settlement date |
| $T_i$ | (Unadjusted) schedule dates; after rolling become adjusted dates |
| $T_i^f$ | Fixing date (often 2 business days before $T_i$) |
| $T_i^p$ | Payment date (often $T_{i+1}$, sometimes with delay) |

#### Prices

| Symbol | Definition |
|--------|------------|
| $P$ | Clean/quoted/flat price (per 100 par unless stated) |
| $AI$ | Accrued interest (per 100 par unless stated) |
| $P^{\text{dirty}}$ | Full/invoice/dirty price; $P^{\text{dirty}} = P + AI$ |

#### Rates and Accrual

| Symbol | Definition |
|--------|------------|
| $c$ | Annual coupon rate (as a fraction, e.g., 0.05 for 5%) |
| $f$ | Coupon frequency per year (e.g., $f=2$ semiannual) |
| $\Delta(t_1,t_2)$ | Accrual fraction between dates (day-count fraction) |
| $r$ | Interest rate used for financing/discounting (context-specific) |
| $m$ | Compounding frequency per year (discrete compounding) |

#### Discounting

| Symbol | Definition |
|--------|------------|
| $P(t,T)$ | Discount bond price (discount factor) paying 1 at $T$ |
| $e^{-y(t,T,T+\tau)\tau}$ | Continuous forward yield: $= P(t,T,T+\tau)$ |
| $1+\tau L(t,T,T+\tau)$ | Simple forward rate: $= 1/P(t,T,T+\tau)$ |

---

## Section-by-Section Outline

### 1. Market Quoting Basics

- Quote objects: price vs yield vs spread (conceptual)
- Bid/ask, mid; quoting units:
  - Treasuries in 32nds
  - "bp" as rate quote unit (basis point = 0.01%)
- "Par" as anchor:
  - Par yield defined as coupon rate that prices bond at par
  - Preview: par swap rate as fixed rate that makes swap PV zero

### 2. Cashflow Timing and Calendars

- Trade vs settlement date; settlement lag examples (T+1, T+3)
- Effective date vs trade date in confirmations
- Payment dates, fixing dates, payment delays
- Business day adjustments:
  - Modified Following rule
  - Other conventions (Following, Preceding, Modified Preceding)
- Stubs (front/back): why irregular first/last periods affect accrual factors and PV
- End-of-month: varies by contract

### 3. Accrual and Clean vs Dirty

- Accrued interest definition and motivation (avoid artificial price jumps)
- Identity: invoice/full price equals PV of remaining cashflows; quoted price adjusts to convention
- Coupon-date behavior: clean price continuity if yield unchanged
- P&L attribution: separating price change vs carry/financing

### 4. Day Count and Compounding as the "Units System"

- Day count X/Y and generic accrual formula
- US examples: ACT/ACT for Treasuries; 30/360 and ACT/360 common in other products
- Compounding frequency as a unit system; conversion across frequencies
- Continuous compounding and conversions

### 5. Sanity Checks Every Practitioner Should Run

- Timing checks: cashflow dates, fixing dates, payment dates, settlement/effective dates
- Sign checks: payer vs receiver PV sign; coupon vs principal sign
- Monotonicity: PV down when rates up (under standard discounting)
- Coupon boundary checks: accrued resets to 0 right after coupon; invoice price jumps by coupon cashflow but clean stays consistent
- Reconciliation: PV from price (dirty) equals PV from discounting cashflows

---

## Key Formulas

### 1. Invoice (Dirty) vs Quoted (Clean)

$$P^{\text{dirty}} = P^{\text{clean}} + AI$$

**Units:** dollars per $100 par (or percent of par); $AI$ same units.

### 2. PV Identity with Accrued Interest (Bond)

$$P + AI = PV(\text{future cashflows})$$

**Units:** price per $100 par.

### 3. Accrued Interest (Generic Proportional Accrual)

If coupon per period is $c/f$ and accrual fraction since last coupon is $\Delta$, then:

$$AI = \Delta \cdot \frac{c}{f}$$

**Units:** "dollars per $1 face" (or per $100 face).

### 4. Day Count Fraction Concept

$$\text{Interest earned} = \frac{\text{days between dates}}{\text{days in reference period}} \times \text{interest in reference period}$$

**Unit check:** dimensionless fraction × dollars = dollars.

### 5. Money-Market Simple Interest (ACT/360 Example)

Interest on $1 for $d$ days at rate $r$:

$$\text{Interest} = r \cdot \frac{d}{360}$$

### 6. Compounding Frequency

**Discrete compounding:**

$$A\left(1+\frac{R}{m}\right)^{mn}$$

after $n$ years.

**Continuous compounding:**

$$Ae^{Rn}$$

**Discounting:**

$$e^{-Rn}$$

### 7. Forward Discount Factor ↔ Rate Relationships

**Continuous:**

$$e^{-y\tau} = P(t,T,T+\tau)$$

**Simple:**

$$1 + \tau L = \frac{1}{P(t,T,T+\tau)}$$

### 8. Bond Trading P&L Decomposition (Clean + Accrued + Financing)

$$P\&L = P(d) + AI(d) - (P(0) + AI(0))\left(1 + r\frac{d}{360}\right)$$

**Interpretation:** price change + interest income − financing cost.

---

## Full Chapter Notes

### 0. Setup

This chapter is "desk plumbing": how quotes become cashflows and invoice amounts; how calendars shape accrual; how clean/dirty conventions prevent mechanical price jumps; and how day count + compounding define the *units* of every rate and PV.

#### Conventions Used in This Chapter

Because conventions vary across markets, we will:
- **Describe** the alternatives when the sources flag them
- **Adopt "USD-typical" defaults for examples**, explicitly labeled

**USD-typical example defaults:**

| Instrument | Conventions |
|------------|-------------|
| **US Treasuries** | Clean quoted in 32nds; accrued computed with ACT/ACT (in period) |
| **Corporate bonds** | Often use 30/360 and settle with varying lags; invoice = clean + accrued |
| **USD swaps** | Schedule dates rolled (typically Modified Following); floating fixings ~2 business days before accrual start; fixed leg commonly 30/360 and floating leg commonly ACT/360 |
| **USD LIBOR-style money market** | Actual/360; spot start delayed by 2 business days; Modified Following rolling |

---

### 1. Core Concepts

#### 1.1 Quote Objects: Price vs Yield vs Spread

**Formal Definitions:**

- A **price quote** is a number that, together with conventions (clean/dirty, day count, settlement), determines the **invoice amount** at settlement
- A **yield quote** is an internal-rate-of-return style rate that (together with compounding/day count conventions) equates discounted cashflows to a price
- A **spread quote** (conceptually) is a rate-like adjustment or premium over a reference. In CDS practice, "spread" is quoted and contracts often trade with fixed coupons plus an upfront to match the quoted spread

**Intuition:**

Traders like quoting the object that is "stable" or liquid for that market:
- **Cash bonds:** clean price is standard because it removes mechanical coupon jumps
- **Money markets:** rates are quoted with simple accrual conventions
- **CDS:** spreads quoted; standardized coupons mean an upfront exchanges hands to align PV

**How it appears in trading/risk practice:**

- Risk engines store **one canonical representation** (often cashflows + conventions) and can reproduce price/yield/spread consistently
- Misinterpreting the quote object (e.g., treating a clean price as dirty) creates immediate PV and P&L errors

---

#### 1.2 Bid/Ask, Mid, and Quoting Increments

**Formal Definition:**

- **Bid:** price at which market-maker buys
- **Ask/Offer:** price at which market-maker sells
- **Mid:** average of bid and ask

Bid/ask is a *trading friction* that shows up in realized P&L even if "the market didn't move."

**Intuition:**

If you buy at ask and sell at bid with no market move, you lose the spread.

**Practice:**

Example of bid/ask effect in bond trading P&L decomposition: isolating a "price change" contribution driven by bid/ask (in a Treasury-style 32nds world).

> **Note:** Exact tick sizes and rounding rules vary by venue/instrument. 32nds for US Treasuries is explicit; other increments are desk-specific.

---

#### 1.3 "Par" as an Anchor

**Formal Definition:**

- **Par yield (bond):** coupon rate per year that makes the bond price equal to par (face value)

**Intuition:**

"Par" means "no upfront premium/discount": coupon is set so the bond issues at 100.

**Practice:**

- New-issue desks talk about "printing at par" or "reoffering price ~par"
- For swaps, traders use "par swap rate" similarly: fixed rate that makes PV=0 at inception

---

### 2. Math and Derivations

#### 2.1 Clean vs Dirty and the PV Identity

**Source-backed identity (bond):**

When accrued is nonzero, the amount exchanged (full/dirty) equals PV of future cashflows:

$$P + AI = PV(\text{future cashflows})$$

**Interpretation:**

$P$ (clean) is *not* itself the PV unless $AI=0$ (e.g., exactly on coupon date).

**Unit check:**

$P$, $AI$, and $PV$ are all in "price per $100 par" (or percent of par).

**Sanity check:**

If you compute PV of future cashflows and compare to **dirty**, they should match (modulo rounding).

---

#### 2.2 Why Bonds Quote Clean: Coupon-Date Continuity

Let $P^b$ be clean price right **before** coupon payment and $P^a$ right **after** coupon payment, coupon $c/2$ (semiannual example).

**Argument:**

- Right before coupon, accrued is $c/2$ and PV includes the next coupon
- Right after coupon, accrued is 0 and PV excludes the paid coupon
- Therefore (if yield unchanged), the same remaining-cashflow PV appears on both sides, implying clean price continuity:

$$P^b = PV(\text{cashflows after next coupon}) = P^a$$

**Intuition:**

Full/dirty price *does* jump because you just received a coupon cashflow; clean price removes that mechanical jump.

---

#### 2.3 Accrued Interest Formulas

**Generic Proportional Accrual:**

Accrued interest = fraction of coupon period elapsed × coupon-per-period:

$$AI = \Delta \left(\frac{c}{f}\right)$$

where $\Delta = d_1/d_2$ is "elapsed days / period days" under the chosen day-count.

**Treasury Convention Example (ACT/ACT-in-period):**

$$AI = \frac{\text{actual days since last coupon}}{\text{actual days in coupon period}} \times \text{coupon per period}$$

**Example:** 54 days since Jan 10, 181-day period to Jul 10, coupon per period $5.50 → accrued ≈ $1.64 per $100.

---

#### 2.4 Day Count as "Time Measurement"

**General Day-Count Framing:**

Day count convention X/Y determines how to measure "days between dates" and "days in reference period," and interest earned scales with their ratio.

**Practical Implication:**

Two bonds with identical coupon and clean price can have different daily carry purely due to day-count conventions.

---

#### 2.5 Compounding as "Units": Discrete vs Continuous vs Simple

**Discrete Compounding Growth Formula:**

$$A\left(1+\frac{R}{m}\right)^{mn}$$

**Continuous Compounding:**

$$A e^{Rn}$$

**Discount factor over $n$ years:**

$$e^{-Rn}$$

**Interest-rate-modeling notation (forward setting):**

- Continuous forward yield: $e^{-y\tau} = P(t,T,T+\tau)$
- Simple forward rate: $1 + \tau L = 1/P(t,T,T+\tau)$

**Why This Matters Operationally:**

A "5%" quote is meaningless without stating (i) day count and (ii) compounding. Different compounding conventions for the same "5%" produce different interest payments over the same calendar interval.

---

#### 2.6 Calendars, Rolling, Settlement, and Schedule Plumbing

**Settlement Lag:**

| Instrument Type | Settlement |
|-----------------|------------|
| Some bonds | T+1 |
| Eurobonds | T+3 |
| LIBOR-style deposits | T+2 |

**Business Day Rolling:**

| Convention | Rule |
|------------|------|
| **Modified Following** | Roll forward unless that pushes into next calendar month, then roll back |
| **Following** | Roll forward to next business day |
| **Preceding** | Roll back to previous business day |
| **Modified Preceding** | Roll back unless that pushes into previous month, then roll forward |

**Swap Schedule Mechanics (Vanilla):**

1. Build unadjusted dates at base frequency
2. Roll them (typically Modified Following)
3. Fixing date $T_i^f$ often 2 business days before $T_i$
4. Payment date $T_i^p$ often $T_{i+1}$, possibly with a 1–2 business day delay

**Stubs:**

Schedules often generate irregular first/last accrual periods ("stubs") when effective date and regular grid don't align. Rolling dates changes period lengths (and hence accrual factors and payments).

---

### 3. Measurement & Risk (Chapter 1 Scope: PV & P&L Attribution Plumbing)

#### 3.1 Clean/Dirty and P&L Attribution

P&L for buying a bond and selling $d$ days later (with repo financing):

$$P\&L = P(d) + AI(d) - (P(0) + AI(0))\left(1 + r\frac{d}{360}\right)$$

Decomposes into: **price change + interest income − financing cost**

**Key Desk Interpretation:**

- If you track **clean price only**, you must separately track **accrual** to reconcile to economic P&L
- If you track **dirty PV**, the accrued is already embedded; adding accrued again double-counts

#### 3.2 Coupon Boundary Behavior (Risk + Reconciliation)

- Immediately after coupon payment, accrued resets to 0
- Immediately before, it approaches the full coupon amount (under typical linear accrual conventions)
- This creates a "sawtooth" pattern in accrued interest but not in clean price (when yields unchanged)
- That's why clean quoting is operationally helpful

---

### 4. Worked Examples

#### Example 1 — US Treasury: 32nds Quote → Accrued → Dirty Price

**Given:**

- Quoted (clean) price: 155-16 = $155 + 16/32 = 155.50$
- Coupon: 11% annually paid semiannually → coupon per period = $5.50 per $100
- Dates: last coupon Jan 10, next coupon Jul 10; valuation Mar 5
- Day counts: actual days since last coupon = 54; in period = 181
- Day count convention: ACT/ACT-in-period (Treasuries)

**Step 1: Accrued Interest**

$$AI = \frac{54}{181} \times 5.50 \approx 1.64$$

**Step 2: Dirty Price**

$$P^{\text{dirty}} = P^{\text{clean}} + AI = 155.50 + 1.64 = 157.14$$

**Sanity Checks:**

- AI is less than the next coupon ($5.50): ✓
- Dirty > clean between coupon dates: ✓

---

#### Example 2 — "101–4 5/8" Quote + Accrued → Invoice Amount

**Given:**

- Clean/flat price: $101-4\frac{5}{8}$
- Accrued interest: $22.79 per $10,000 face = 0.2279% of par

**Step 1: Convert 32nds**

- $4\frac{5}{8}$ thirty-seconds = $4.625/32$
- Clean in % of par: $101 + 4.625/32 = 101.14453125$

**Step 2: Add Accrued**

$$P^{\text{dirty}} = 101 + \frac{4.625}{32} + 0.2279 = 101.3724\% \text{ of par}$$

**Step 3: Invoice for $10,000 Face**

$$\text{Invoice} = 10,000 \times 1.013724 = \$10,137.24$$

**Sanity Checks:**

- Dirty > clean: ✓
- Accrued is small compared to price: ✓ (plausible for a bond not near coupon date)

---

#### Example 3 — CDS Stub Premium: Why Stubs Change Cashflows

**Given:**

- Notional $10mm
- Contractual spread 35.0 bp (= 0.0035)
- Quarterly schedule, Actual/360, Following business day rule, TARGET calendar
- Short stub at the beginning
- First accrual fraction: 0.161111
- First flow date: Thu 20 Dec 2007

**Compute:**

Premium payment per period (per notional) is $S \times \Delta$ under ACT/360; dollar payment is $N \times S \times \Delta$.

$$\text{Payment} = 10,000,000 \times 0.0035 \times 0.161111 \approx 5,638.89$$

**Takeaway:**

Stub periods mechanically change $\Delta$, so even with a "fixed" spread, cash coupons vary period to period.

---

#### Example 4 — Compounding Conversion (Semiannual → Continuous)

**Conversion Formulas:**

$$R_c = m \ln\left(1 + \frac{R_m}{m}\right)$$

$$R_m = m\left(e^{R_c/m} - 1\right)$$

**For $R_m = 10\%$ with $m = 2$:**

$$R_c = 2 \ln(1.05) \approx 0.09758 = 9.758\%$$

**Sanity Check:**

Continuous rate is slightly lower than semiannual nominal for the same effective growth: expected.

---

### 5. Practical Notes

#### 5.1 Implementation Checklist: "Quote → PV"

1. **Identify quote object:**
   - Bond clean price? CDS spread + standard coupon? Money-market rate?

2. **Confirm conventions:**
   - Day count (ACT/ACT vs ACT/360 vs 30/360)
   - Compounding (simple vs discrete vs continuous)
   - Settlement lag (T+1/T+2/T+3)
   - Business day roll (Following / Modified Following / Preceding)

3. **Build schedule:**
   - Unadjusted grid → roll dates → assign fixing/payment dates
   - Identify stubs and compute each $\Delta_i$ explicitly

4. **Compute cashflows:**
   - Coupon per period: $c/f$; premium: $S\Delta$; etc.

5. **Compute PV consistently and reconcile to invoice/dirty:**
   - Use $P + AI = PV(\text{future cashflows})$

#### 5.2 Verification Tests ("Sanity Checks")

**Date/Timing Tests:**

- Fixing date should precede accrual start
- Payment date should be at/after accrual end
- Settlement vs effective: ensure you're valuing "as of" correctly

**Boundary Tests:**

- Right after coupon date: $AI = 0$
- Right before: $AI \approx$ full coupon

**Monotonicity:**

- Increase discount rates → PV decreases (for positive cashflows)

**Reconciliation:**

- PV from discounted cashflows equals dirty price
- Clean = dirty − AI

#### 5.3 Common Plumbing Bugs

| Bug | Description |
|-----|-------------|
| Off-by-one in schedule | Forgetting to roll **all** dates, or rolling payment dates but not accrual boundaries |
| Day count mismatch | Using ACT/360 on a leg that should be 30/360 (or vice versa) |
| Clean/dirty confusion | Treating quoted clean as if it were dirty (or adding accrued twice) |
| Ignoring stubs | Assuming equal $\Delta_i$ across periods |

---

### 6. Summary & Recall

#### 10-Bullet Executive Summary

1. Always ask: **what is being quoted** (price, yield, spread) and what conventions interpret it
2. Bonds usually quote **clean**; cash exchanged at settlement is **dirty = clean + accrued**
3. Accrued interest is the pro-rated portion of the next coupon since last coupon date
4. PV identity: **dirty** price equals PV of remaining cashflows; clean adjusts to whatever accrued convention is used
5. Clean price does not mechanically jump on coupon date if yield is unchanged; accrued resets instead
6. Settlement lags matter (T+1/T+3 etc); cash moves on settlement, not trade
7. Business day rolling (Following vs Modified Following vs Preceding) changes dates and thus accrual factors
8. Stubs create irregular accrual periods; coupon amounts vary because $\Delta$ varies
9. Day count and compounding define the *units* of rates; converting is like converting km↔miles
10. Always run sanity checks: timing, sign, monotonicity, boundary (coupon), and PV reconciliation

#### Cheat Sheet of Formulas/Definitions

| Formula | Expression |
|---------|------------|
| Dirty price | $P^{\text{dirty}} = P + AI$ |
| PV identity | $P + AI = PV(\text{future cashflows})$ |
| Accrued interest | $AI = \Delta(c/f)$ |
| Day count | Earned = (days between / days in ref period) × ref-period interest |
| Simple interest (ACT/360) | Interest on $1 for $d$ days: $rd/360$ |
| Discrete compounding | $A(1+R/m)^{mn}$ |
| Continuous compounding | Growth $Ae^{Rn}$; discount $e^{-Rn}$ |
| Forward rate (continuous) | $e^{-y\tau} = P(t,T,T+\tau)$ |
| Forward rate (simple) | $1 + \tau L = 1/P(t,T,T+\tau)$ |
| Trading P&L | $P\&L = P(d) + AI(d) - (P(0) + AI(0))(1 + rd/360)$ |

---

### 7. Flashcards (Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the difference between clean and dirty price? | Clean is the quoted price; dirty is cash exchanged: clean + accrued interest |
| 2 | Why do bond markets quote clean? | To avoid mechanical price jumps at coupon dates; accrued handles the jump |
| 3 | What is accrued interest? | The pro-rated portion of the next coupon that has accrued since the last coupon date |
| 4 | State the PV identity involving accrued interest | $P + AI = PV(\text{future cashflows})$ |
| 5 | What happens to clean price on coupon date if yield is unchanged? | It does not mechanically drop; accrued resets and the coupon cashflow occurs |
| 6 | What is a day-count convention X/Y? | A rule for measuring the fraction of a reference period between two dates |
| 7 | Give common US day counts | ACT/ACT (in period), 30/360, ACT/360 |
| 8 | What day count is used for US Treasury bonds? | ACT/ACT (in period) |
| 9 | What is Modified Following? | Roll forward to next business day unless that moves into next month, then roll back |
| 10 | What is a stub period? | An irregular first/last accrual period created by schedule alignment/rolling; affects $\Delta$ and payments |
| 11 | Why do stub periods matter? | Because coupon/premium cashflows scale with $\Delta$; different $\Delta$ → different cash amounts |
| 12 | How are US Treasury bond prices quoted? | In dollars and 32nds per $100 face |
| 13 | What is the discrete compounding growth formula? | $A(1+R/m)^{mn}$ |
| 14 | What is continuous discounting over $n$ years at rate $R$? | Multiply by $e^{-Rn}$ |
| 15 | Give the bond+repo P&L plumbing identity | $P\&L = P(d) + AI(d) - (P(0) + AI(0))(1 + rd/360)$ |

---

## Mini Problem Set

### Questions

1. A Treasury bond quote is 120-05 (per $100). Convert it into a decimal clean price.

2. Using ACT/ACT-in-period, compute accrued interest given 54 days since last coupon, 181-day coupon period, and $5.50 semiannual coupon per $100.

3. Compute the dirty price given the clean price in (1) and accrued in (2).

4. A bond has clean price 101–4 5/8 and accrued interest 0.2279% of par. Compute the dirty price in % of par and the invoice amount for $10,000 face.

5. Explain (with a one-line equation) why the *clean* price does not jump down on a coupon date if yield is unchanged.

6. A bond settles T+3. If you trade on Monday, what is the earliest possible settlement day ignoring holidays? (State your assumption.)

7. For a swap, list the ordered timeline for one period: fixing date → accrual start → accrual end → payment date (include the "2 business day" convention).

8. Define Modified Following and give an example where it differs from Following.

9. A CDS has contractual spread 35 bp, notional $10mm, and first accrual fraction 0.161111. Compute the first premium payment amount.

10. Convert 10% per annum with semiannual compounding to an equivalent continuously compounded rate (use $R_c = m\ln(1+R_m/m)$).

11. Show that if you compute PV using dirty price, you must not separately add "accrued" again. (Explain in one paragraph using $P + AI = PV$.)

---

## Source Map

### Directly Source-Backed

- Clean vs dirty pricing; Treasury 32nds; dirty = clean + accrued; ACT/ACT Treasury example
- Accrued interest definition and motivation; bond settlement lag variability
- PV identity $P + AI = PV$ and clean-price coupon-date continuity argument
- LIBOR ACT/360 and T+2 settlement; USD LIBOR delay and Modified Following
- Swap schedule plumbing: rolling, fixing/payment dates, and typical USD leg day counts
- Day count and compounding conversion formulas
- CDS schedule construction/rolling and short stub illustration

### Derived (Reasoned Inference)

- Par swap rate concept as "fixed rate making swap PV=0" (derived from swap valuation expressions)
- PV monotonicity and reconciliation tests (derived from discounting identities and PV algebra)

### Uncertain / Needs More Info

- End-of-month rule specifics for target products (requires explicit contract standard or a specific source section)
- Exact tick/rounding conventions beyond Treasuries-in-32nds (depends on product/venue)
