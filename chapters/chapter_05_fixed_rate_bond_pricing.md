# Chapter 5: Fixed-Rate Bond Pricing

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- A fixed-rate bond is a deterministic stream of coupons plus principal at maturity (Tuckman Ch 1-4)
- Discount factors $P(0,t)$ give the PV today of receiving 1 unit at time $t$ (Tuckman Ch 1)
- Bond PV is the sum of each cashflow times its discount factor — the law of one price (Tuckman Ch 1)
- Dirty/full/invoice price = clean/flat/quoted price + accrued interest (Tuckman, Hull Ch 4)
- Accrued interest pro-rates the next coupon based on days elapsed in the coupon period (Tuckman example: 166/181 days)
- U.S. Treasury bonds are quoted in dollars and 32nds per $100 face (Hull Ch 4)
- Yield-to-maturity is the single rate that discounts all cashflows to the market price (Tuckman Ch 3)

### (B) Reasoned Inference (Derived from A)

- Curve shifts change PV through the discount factors: rising rates → falling discount factors → falling PV
- Longer-dated cashflows are more sensitive because their discount factors move more for a given rate change
- Clean pricing removes mechanical accrual drift, making quotes comparable across settlement dates

### (C) Speculation (Clearly Labeled; Minimal)

- None in this chapter. All content is either source-verified or logically derived.

---

## Conventions & Notation

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $F$ | Face value (principal) repaid at maturity; often 100 "per 100 par" |
| $c$ | Annual coupon rate (fixed percentage of $F$) |
| $m$ | Coupon payments per year (e.g., $m=2$ for semiannual) |
| $\text{Cpn} = Fc/m$ | Coupon cashflow each payment date |
| $T_1, \ldots, T_N$ | Future coupon payment dates after settlement/valuation |
| $\text{CF}_i$ | Cashflow at $T_i$: typically $\text{Cpn}$ for $i < N$, and $\text{Cpn} + F$ for $i = N$ |
| $t_i$ | Time in years from valuation/settlement date to $T_i$ |
| $P(0,t)$ or $d(t)$ | Discount factor from date 0 to time $t$ |
| $P_{\text{dirty}}$ | Full/dirty price (invoice/cash price) |
| $P_{\text{clean}}$ | Clean/flat/quoted price |
| $\text{AI}$ | Accrued interest from last coupon date to settlement |
| $y$ | Yield-to-maturity (preview) |

### Key Identity

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + \text{AI}}$$

### Quote Notation

- Treasury-style fractional quote: "$101\text{-}04\frac{5}{8}$" means $101 + \frac{4 + 5/8}{32}$

### Defaults Used in Examples

| Convention | Default |
|------------|---------|
| Discounting | Discount factors $P(0,t)$ taken as inputs (no bootstrapping) |
| Accrued interest day-count | Actual/Actual-in-period |
| Coupon frequency | Semiannual ($m=2$) |
| Face value | $F = 100$ |

**Market differences (flag):** Day-count conventions and coupon frequencies vary by market/instrument. Without a specific market mandate, we label conventions explicitly in each example.

---

## Setup

This chapter is the first "full instrument pricing" chapter: we go from a bond's contractual cashflows to a present value (PV) using a given discount curve, and then connect that PV to what traders quote (clean price) versus what actually changes hands (dirty price).

**Organizing idea:** The law of one price — fixed cashflows can be replicated by zero-coupon discount factors, so any fixed-cashflow bond is priced by discounting each promised cashflow at the appropriate discount factor.

---

## Core Concepts

### 1) Fixed-Rate Bond Cashflows

**Formal Definition:**

A fixed-rate bond promises deterministic cashflows:
- Coupons of amount $\text{Cpn} = Fc/m$ on scheduled coupon dates
- Principal $F$ at maturity $T_N$ (often bundled with the final coupon as $\text{Cpn} + F$)

**Intuition:**

The bond is "just" a stream of promised cash payments. Pricing reduces to:
1. Listing those payments (amounts and dates)
2. Discounting each at the right term

**Trading/Risk Practice:**

Traders talk about coupon rate (contractual) and maturity (cashflow timing). Risk arises because discount factors (interest rates) change, moving PV.

---

### 2) Discount Factors and PV of Fixed Cashflows (Curve as Input)

**Formal Definition:**

A discount factor $P(0,t)$ is the PV today (at time 0) of receiving 1 unit of currency at time $t$. Once discount factors are known, "each cash flow must be discounted at the factor or rate appropriate for the term of that cash flow," enabling valuation of any fixed-cashflow security.

**Intuition:**

Discount factors are "prices of money at future dates." A bond is a portfolio of those "future dollars."

**Trading/Risk Practice:**

On a desk, you typically do not re-derive the curve for every bond in isolation; you take a curve as given (Treasury, OIS, swap, etc.) and PV cashflows off that curve.

---

### 3) Clean Price vs Dirty (Full) Price

**Formal Definition:**

- **Dirty (full/invoice) price:** The PV of remaining promised cashflows as of settlement, and the amount actually exchanged (per 100 par)
- **Clean (flat/quoted) price:** The quoted price that excludes accrued interest

**Market convention:** The buyer pays "flat price plus accrued interest":

$$P_{\text{dirty}} = P_{\text{clean}} + \text{AI}$$

**Intuition:**

The clean price is "the bond's value net of coupon accrual," so quotes don't mechanically drift up day-by-day as interest accrues.

**Trading/Risk Practice:**

- Traders quote clean so prices are comparable across settlement dates
- Cash settlement uses dirty (invoice) so the seller is compensated for interest accrued while holding the bond

---

### 4) Accrued Interest (AI)

**Formal Definition:**

Accrued interest is the portion of the next coupon payment that economically belongs to the seller because it has accrued since the previous coupon date. It is computed by pro-rating the next coupon by the fraction of the coupon period the seller held the bond, using actual days between dates.

**Intuition:**

Between coupon dates, interest "accumulates." Rather than forcing the seller to wait for the coupon date to get paid for that accumulation, the buyer reimburses the seller at settlement.

**Trading/Risk Practice:**

Accrued interest is booked in P&L/carry and affects settlement cash; it also matters for futures conversion factors and invoice calculations.

---

### 5) Quoting Conventions (Price, Yield — Preview)

**Formal Definition:**

- **Price quote (clean):** e.g., U.S. Treasury bonds are quoted in dollars and 32nds of a dollar per $100 face
- **Yield quote (preview):** Yield-to-maturity is "the single rate that when used to discount a bond's cash flows produces the bond's market price"

**Intuition:**

- Clean price is a direct quote of PV net of accrual
- Yield is a compressed summary of price; useful, but can mislead because it forces all cashflows to be discounted at one rate

**Trading/Risk Practice:**

Many markets quote both price and yield; spreads are quoted relative to benchmark curves (preview only here).

---

## Math and Derivations

### 1) Constructing the Cashflow Schedule

**Assumptions:**
- Fixed coupon rate $c$, face value $F$, coupon frequency $m$
- Coupon dates $T_1, \ldots, T_N$ follow the bond's schedule

**Derivation:**

Coupon per period:

$$\boxed{\text{Cpn} = \frac{Fc}{m}}$$

Cashflows:

$$\text{CF}_i = \begin{cases} \text{Cpn}, & i = 1, \ldots, N-1 \\ \text{Cpn} + F, & i = N \end{cases}$$

**Unit Check:**
- $F$ is dollars (per 100 par if $F = 100$)
- $c$ is "per year"
- $m$ is "payments per year"
- $\text{Cpn}$ is dollars per payment date ✓

**Sanity Check:**
- If $c = 0$, all intermediate coupons vanish: only principal at maturity remains (zero-coupon bond limit)

---

### 2) PV Using a Discount Factor Curve (Curve is an Input)

**Assumptions:**
- Discount factors $P(0, t_i)$ are known inputs for each cashflow date $T_i$
- Valuation is at settlement/valuation date "0," and $t_i$ is the year fraction to $T_i$

**Derivation:**

By the law of one price, the value of a fixed set of future cashflows is the sum of the values of each cashflow discounted at its term:

$$\boxed{P_{\text{dirty}} = \sum_{i=1}^{N} \text{CF}_i \cdot P(0, t_i)}$$

**Unit Check:**
- $P(0, t_i)$ is dollars today per dollar at $T_i$ (dimensionless in "per $1" units)
- $\text{CF}_i$ is dollars at $T_i$
- Product is dollars today; sum is dollars today ✓

**Sanity Checks:**
- **Upper bound:** If all discount factors $\leq 1$, then $P_{\text{dirty}} \leq \sum_i \text{CF}_i$
- **Zero-coupon limit:** If $c = 0$, then $P_{\text{dirty}} = F \cdot P(0, t_N)$
- **Monotonic curve:** For positive rates, discount factors should be decreasing in maturity

---

### 3) Accrued Interest and Clean/Dirty Decomposition

**Assumptions:**
- Settlement date $S$ lies between the previous coupon date $T_k$ and next coupon date $T_{k+1}$
- Day-count convention determines the accrual fraction

**Derivation (Actual/Actual-in-period):**

Let:
- $d_{\text{elapsed}}$ = # actual days from $T_k$ to $S$
- $d_{\text{period}}$ = # actual days from $T_k$ to $T_{k+1}$

Accrual fraction:

$$\alpha = \frac{d_{\text{elapsed}}}{d_{\text{period}}}$$

Accrued interest:

$$\boxed{\text{AI} = \alpha \cdot \text{Cpn} = \frac{d_{\text{elapsed}}}{d_{\text{period}}} \cdot \text{Cpn}}$$

Clean/dirty relation:

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + \text{AI}}$$

**Unit Check:**
- $\alpha$ is unitless (days/days)
- $\text{AI}$ is dollars per $F$ face value ✓

**Sanity Checks:**
- $0 \leq \alpha \leq 1$ between coupon dates
- $0 \leq \text{AI} \leq \text{Cpn}$
- On a coupon payment date, accrued interest resets to 0

---

### 4) Quoting in 32nds (U.S. Treasury-Style) and Conversion to Decimal

**Assumptions:**
- Price quoted as $A\text{-}BB$ where $BB$ is in 32nds of a dollar per $100 face

**Derivation:**

If the quote is $A\text{-}BB$:

$$P_{\text{clean}} = A + \frac{BB}{32}$$

If the quote includes an extra fraction, e.g., $101\text{-}04\frac{5}{8}$:

$$\boxed{P_{\text{clean}} = A + \frac{BB + \frac{x}{y}}{32}}$$

**Example:** $101\text{-}04\frac{5}{8}$ means:

$$P_{\text{clean}} = 101 + \frac{4 + \frac{5}{8}}{32} = 101 + \frac{4.625}{32} = 101.14453125$$

**Sanity Check:**
- Since $1/32 = 0.03125$, a change of 1 "tick" changes the quoted price by 0.03125 per 100 face

---

## Measurement & Risk (Chapter 5 Scope)

### PV and a (Conceptual) Curve Shift

PV is the sum of discounted cashflows:

$$P_{\text{dirty}} = \sum_i \text{CF}_i \cdot P(0, t_i)$$

So if the discount curve shifts (discount factors change), PV changes through each term.

**First-order intuition:**
- If rates rise, discount factors $P(0,t)$ fall, so PV falls
- Longer-dated cashflows (larger $t$) tend to be more sensitive because their discount factors move more for a given curve shift

This sets up later chapters on duration/DV01 and key-rate risk.

### Clean/Dirty Conventions and Observed P&L Around Coupon Dates

- **Dirty price** drifts upward between coupons as accrued interest builds, then drops around the coupon date because the coupon is paid out (the bond becomes "ex-coupon")
- **Clean price** removes the mechanical accrual component by subtracting AI; this is why quoting clean makes prices more comparable across dates

**P&L terms:** If you mark the bond clean, the coupon payment appears as a cashflow (coupon income) rather than as a jump in the quoted bond price.

### DV01 Intuition (Preview Only)

DV01 is the dollar value of a 1bp ("01") change in rates:

$$\text{DV01} \equiv -\frac{\Delta P}{10{,}000 \times \Delta y}$$

Preview intuition:
- Bigger/longer cashflows ⇒ larger DV01
- Longer maturity ⇒ (usually) larger rate sensitivity

We will formalize and compute DV01/duration in Chapter 11.

---

## Worked Examples

### Example A — Build the Cashflow Schedule and PV Using a Given Discount Factor Curve

**Instrument:** $F = 100$, coupon rate $c = 5.50\%$, semiannual ($m = 2$), maturity Jan 31, 2003, coupon dates Jan 31 / Jul 31, settlement date Feb 15, 2001.

**Step 1: Coupon amount**

$$\text{Cpn} = \frac{Fc}{m} = \frac{100 \times 0.055}{2} = 2.75$$

**Step 2: Remaining cashflow schedule after settlement**

| Payment Date $T_i$ | Cashflow $\text{CF}_i$ |
|-------------------|------------------------|
| Jul 31, 2001 | 2.75 |
| Jan 31, 2002 | 2.75 |
| Jul 31, 2002 | 2.75 |
| Jan 31, 2003 | 102.75 (coupon + principal) |

**Step 3: Given discount factor curve (inputs)**

Using $t_i = \text{days}/365$:

| Time | Discount Factor |
|------|-----------------|
| $t_1$ (to Jul 31, 2001) | $P(0, t_1) = 0.9800$ |
| $t_2$ (to Jan 31, 2002) | $P(0, t_2) = 0.9550$ |
| $t_3$ (to Jul 31, 2002) | $P(0, t_3) = 0.9300$ |
| $t_4$ (to Jan 31, 2003) | $P(0, t_4) = 0.909914$ |

**Step 4: PV (dirty/full price)**

$$P_{\text{dirty}} = 2.75(0.9800 + 0.9550 + 0.9300) + 102.75(0.909914)$$

Compute coupon PV:
$$2.75 \times 2.8650 = 7.87875$$

Compute final PV:
$$102.75 \times 0.909914 = 93.4937$$

Total:
$$\boxed{P_{\text{dirty}} = 7.87875 + 93.4937 = 101.3724}$$

**Sanity Checks:**
- Price is slightly above par (101.37), reasonable since coupon (5.50%) is "rich" relative to the curve
- Discount factors are decreasing in maturity (0.98 → 0.9099), consistent with positive rates ✓

---

### Example B — Compute Accrued Interest and Clean vs Dirty Price (Explicit Day-Count)

Use the same bond and settlement date Feb 15, 2001, between coupon dates Jan 31, 2001 and Jul 31, 2001.

**Given (from day counts):**
- Days from settlement to next coupon: 166 days
- Days in coupon period (Jan 31 to Jul 31): 181 days
- Days elapsed since last coupon (Jan 31 to Feb 15): $181 - 166 = 15$ days

**Step 1: Coupon per period**

$$\text{Cpn} = 2.75$$

**Step 2: Accrual fraction (Actual/Actual-in-period)**

$$\alpha = \frac{15}{181}$$

**Step 3: Accrued interest**

$$\text{AI} = \alpha \cdot \text{Cpn} = \frac{15}{181} \times 2.75 = \frac{41.25}{181} \approx 0.2279$$

This matches Tuckman's example: \$22.79 on \$10,000 face corresponds to 0.2279 per \$100 face.

**Step 4: Clean vs dirty with a quoted clean price**

Suppose the bond is quoted at $101\text{-}04\frac{5}{8}$ (clean/flat).

Convert the quote to decimal:
$$101 + \frac{4 + \frac{5}{8}}{32} = 101 + \frac{4.625}{32} = 101 + 0.14453125 = 101.14453125$$

Compute dirty:
$$P_{\text{dirty}} = P_{\text{clean}} + \text{AI} = 101.14453125 + 0.2279 = 101.37243$$

**Output:**
- $\text{AI} \approx 0.2279$
- $P_{\text{clean}} \approx 101.1445$
- $\boxed{P_{\text{dirty}} \approx 101.3724}$

---

### Example C — Given Clean Price and Accrued Interest, Verify PV Consistency

Continue with Example A's discount curve and Example B's AI and clean quote.

**Inputs:**
- Clean quote $P_{\text{clean}} = 101.14453125$ (from $101\text{-}04\frac{5}{8}$)
- Accrued interest $\text{AI} \approx 0.2279$
- Discount factors: $0.9800, 0.9550, 0.9300, 0.909914$

**Step 1: Dirty from quote**

$$P_{\text{dirty}} = 101.14453125 + 0.2279 \approx 101.3724$$

**Step 2: Dirty from PV of cashflows (recompute)**

$$P_{\text{dirty}} = 2.75(0.9800 + 0.9550 + 0.9300) + 102.75(0.909914) \approx 101.3724$$

**Output:** The dirty price computed from market quoting ($P_{\text{clean}} + \text{AI}$) matches the PV computed from the curve (to rounding), confirming:

$$\boxed{\text{Quote-implied dirty} = \text{Curve-implied PV}}$$

---

### Example D — Price Under Two Discount Curves and Attribute the Difference

Same bond, same cashflows.

**Cashflows (per 100 face):**
- 2.75 on Jul 31, 2001; Jan 31, 2002; Jul 31, 2002
- 102.75 on Jan 31, 2003

**Curve 1 (as in Example A):**

$(0.9800, 0.9550, 0.9300, 0.909914) \Rightarrow P_{\text{dirty}}^{(1)} \approx 101.3724$

**Curve 2 (higher-rate / lower-discount-factor scenario):**

$(0.9790, 0.9520, 0.9250, 0.9000)$

**Step 1: PV under Curve 2**

Coupon PV:
$$2.75(0.9790 + 0.9520 + 0.9250) = 2.75(2.8560) = 7.8540$$

Final PV:
$$102.75(0.9000) = 92.4750$$

Total:
$$P_{\text{dirty}}^{(2)} = 7.8540 + 92.4750 = 100.3290$$

**Step 2: Price difference and attribution**

$$\Delta P_{\text{dirty}} = P_{\text{dirty}}^{(2)} - P_{\text{dirty}}^{(1)} = 100.3290 - 101.3724 = -1.0434$$

**Attribution intuition (first-order, cashflow-by-cashflow):**

$$\Delta P \approx \sum_i \text{CF}_i \cdot \Delta P(0, t_i)$$

The biggest contributions typically come from:
- The largest cashflow (final 102.75) times the change in long discount factor
- Medium contributions from coupons times changes in shorter discount factors

This is exactly why "cashflow mapping" is the starting point for curve risk.

---

### Example E — Coupon Date Effect on Clean vs Dirty Price Over a Timeline

Illustrate the mechanical behavior of AI, $P_{\text{dirty}}$, and $P_{\text{clean}}$ around coupon date Jul 31, 2001.

**Assumptions:**
- Clean price held constant at $P_{\text{clean}} = 101.1445$ (rates/curve unchanged)
- Coupon per period: $\text{Cpn} = 2.75$
- Coupon period Jan 31 to Jul 31: 181 days
- Next coupon period Jul 31 to Jan 31: 184 days

Compute AI using $\text{AI} = \frac{d_{\text{elapsed}}}{d_{\text{period}}} \times 2.75$:

| Date | Days Elapsed | AI | $P_{\text{dirty}} = P_{\text{clean}} + \text{AI}$ |
|------|--------------|-----|---------------------------------------------------|
| Jul 26, 2001 | 176 | $2.75 \times \frac{176}{181} = 2.6735$ | $101.1445 + 2.6735 = 103.8180$ |
| Jul 30, 2001 | 180 | $2.75 \times \frac{180}{181} = 2.7348$ | $101.1445 + 2.7348 = 103.8793$ |
| Jul 31, 2001 (after payment) | reset | 0 | $101.1445$ |
| Aug 01, 2001 | 1 | $2.75 \times \frac{1}{184} = 0.0150$ | $101.1445 + 0.0150 = 101.1595$ |

**Interpretation:**
- Between coupons, dirty rises smoothly as AI increases
- On the coupon date, the bond becomes ex-coupon after payment; dirty drops by roughly the coupon amount, while AI resets
- **Clean stays comparatively smooth** because it strips out AI mechanically

---

## Practical Notes

### Market Quoting Conventions

**U.S. Treasury bonds:**
- Quoted in dollars and 32nds of a dollar per \$100 face
- Example conversion shown in Example B

**Clean vs dirty terminology:**
- Hull: "quoted price" (clean) and "cash price" (dirty)
- Tuckman: "flat price" (clean) and "invoice/full price" (dirty)
- Relation: cash price = quoted price + accrued interest

**Yield quoting (preview):**
- YTM is often used as an alternate way to quote price
- We do not build yield/spread taxonomies here

### Settlement and Accrued

Accrued is computed to **settlement date**, not necessarily trade date: the buyer compensates the seller for interest accrued up to settlement.

**Market difference (flag):** Actual settlement lags (e.g., T+1/T+2) and holiday calendars depend on market and instrument. I'm not sure which lag and calendar to default without specifying the market (UST vs corporate vs other).

### Implementation Pitfalls

| Issue | Notes |
|-------|-------|
| **Schedule generation** | Ensure coupon dates are generated correctly (frequency, maturity date, date rolling rules). I'm not sure which specific business-day adjustment and stub conventions to apply without a market standard specified. |
| **Day-count convention** | Accrued interest depends on day-count; even "30/360" has multiple variants. I'm not sure which variant to apply without a market/instrument choice. |
| **Ex-coupon handling** | Some valuation contexts refer to ex-coupon prices (price after coupon payment). I'm not sure what ex-dividend/ex-coupon period rules to apply. |
| **Rounding/quoting** | Treasury quotes in 32nds (and sometimes finer increments). Decide where to round: cash price, clean price, AI, or each cashflow PV. |

### Verification Tests

- **PV bounds:** Price should be within a plausible range given coupon and curve
- **Dirty price continuity:** Dirty should increase smoothly between coupons and drop by roughly the coupon on payment date
- **Accrued interest bounds:** $0 \leq \text{AI} \leq \text{Cpn}$ and resets to 0 on coupon date
- **Zero-coupon limit check:** Set $c = 0$ and verify $P_{\text{dirty}} = F \cdot P(0, t_N)$

---

## Summary & Recall

### 10-Bullet Executive Summary

1. A fixed-rate bond is a deterministic stream of coupons plus principal at maturity
2. Discount factors $P(0,t)$ price future dollars; bond PV is the sum of each cashflow times its discount factor
3. The dirty/full price is the PV of remaining cashflows as of settlement
4. Market convention: bonds trade at clean (flat) price plus accrued interest, producing the invoice/full price paid
5. Accrued interest splits the next coupon between seller and buyer in proportion to the time each holds the bond during the coupon period
6. Accrued interest depends on day-count; U.S. Treasury convention uses Actual/Actual-in-period
7. Clean pricing removes mechanical accrual drift from quoted prices, making quotes more comparable across dates
8. U.S. Treasury bonds are quoted in 32nds per \$100 face; convert quotes to decimals to compute cash prices
9. Yield-to-maturity is a single-rate summary that reproduces the market price when used to discount the cashflows (preview only)
10. Curve shifts change PV through the discount factors; DV01 summarizes first-order price sensitivity to a 1bp rate change (preview)

### Cheat Sheet of Formulas

| Formula | Meaning |
|---------|---------|
| $\text{Cpn} = \frac{Fc}{m}$ | Cash paid each coupon date |
| $\text{CF}_i = \text{Cpn}$ for $i < N$; $\text{CF}_N = \text{Cpn} + F$ | Cashflow schedule |
| $P_{\text{dirty}} = \sum_{i=1}^{N} \text{CF}_i \cdot P(0, t_i)$ | PV using input discount factors |
| $\text{AI} = \frac{d_{\text{elapsed}}}{d_{\text{period}}} \cdot \text{Cpn}$ | Pro-rated next coupon (Actual/Actual) |
| $P_{\text{dirty}} = P_{\text{clean}} + \text{AI}$ | Invoice price identity |
| $A\text{-}BB \mapsto A + \frac{BB}{32}$ | Treasury 32nds to decimal |
| $\text{DV01} \equiv -\frac{\Delta P}{10{,}000 \cdot \Delta y}$ | Dollar PV change for 1bp move (preview) |

---

## Flashcards (20 Q/A Pairs)

**Q1:** What are the two components of a fixed-rate bond's contractual cashflows?
**A1:** Coupons on scheduled dates and principal repayment at maturity (often with final coupon).

**Q2:** What is a discount factor $P(0,t)$?
**A2:** The PV today of receiving 1 unit of currency at time $t$.

**Q3:** How do you price a fixed-cashflow bond given discount factors?
**A3:** Sum $\text{CF}_i \cdot P(0, t_i)$ across payment dates.

**Q4:** Define dirty price.
**A4:** The full/invoice/cash price paid; PV of remaining cashflows as of settlement.

**Q5:** Define clean price.
**A5:** The quoted/flat price excluding accrued interest.

**Q6:** Relationship between dirty and clean price?
**A6:** $P_{\text{dirty}} = P_{\text{clean}} + \text{AI}$

**Q7:** What is accrued interest economically trying to do?
**A7:** Split the next coupon between seller and buyer based on holding time in the coupon period.

**Q8:** In Actual/Actual-in-period, what determines the accrual fraction?
**A8:** Actual days held divided by actual days in the coupon period.

**Q9:** Why is clean price commonly quoted?
**A9:** It removes mechanical accrual effects from the quoted price between coupon dates.

**Q10:** What does "invoice price" mean in this context?
**A10:** The amount paid at settlement: flat (clean) price plus accrued interest.

**Q11:** How are U.S. Treasury bond prices quoted in the spot market?
**A11:** In dollars and 32nds of a dollar per \$100 face.

**Q12:** Convert $120\text{-}05$ into a decimal price.
**A12:** $120 + \frac{5}{32} = 120.15625$

**Q13:** What is yield-to-maturity (preview)?
**A13:** The single rate that discounts a bond's cashflows to its market price.

**Q14:** Why can yield-to-maturity be misleading?
**A14:** It compresses term-structure information into one rate; Tuckman warns yields can mislead as a value/return measure.

**Q15:** What happens to PV if discount rates rise (conceptually)?
**A15:** Discount factors fall, so PV falls.

**Q16:** What is "ex-coupon price"?
**A16:** Price after the coupon payment has been made (coupon is no longer part of the security's remaining cashflows).

**Q17:** What is DV01 (preview)?
**A17:** Dollar value change for a 1bp decline in rates.

**Q18:** Why is DV01 usually positive for fixed coupon bonds?
**A18:** Because bond prices usually rise when rates decline.

**Q19:** What's the "zero-coupon limit check" for a pricing routine?
**A19:** Set $c = 0$ and verify $P_{\text{dirty}} = F \cdot P(0, t_N)$.

**Q20:** What is the first step in pricing any bond?
**A20:** Build the correct cashflow schedule (amounts and dates).

---

## Mini Problem Set

### Problems (12 Questions)

**Problem 1:** A bond has $F = 100$, $c = 6\%$, $m = 2$. What is the coupon payment amount?
*Sketch:* $\text{Cpn} = 100 \cdot 0.06 / 2 = 3.00$

**Problem 2:** A bond pays coupons of 2.75 on each of three dates and 102.75 at maturity. Discount factors are $0.98, 0.955, 0.93, 0.91$. Compute the dirty price.
*Sketch:* $P = 2.75(0.98 + 0.955 + 0.93) + 102.75(0.91)$

**Problem 3:** Between coupon dates, $d_{\text{elapsed}} = 20$ days and $d_{\text{period}} = 182$ days. Coupon is 3.00. Compute AI (Actual/Actual-in-period).
*Sketch:* $\text{AI} = 3.00 \times 20/182 \approx 0.3297$

**Problem 4:** A clean price is 99.50 and AI is 1.20. What is the dirty price?
*Sketch:* $P_{\text{dirty}} = 99.50 + 1.20 = 100.70$

**Problem 5:** Convert the U.S. Treasury quote $155\text{-}16$ into a decimal price per \$100 face.
*Sketch:* $155 + 16/32 = 155.50$

**Problem 6:** Using Actual/Actual-in-period, last coupon date Jan 10, next coupon date Jul 10 (181-day period), settlement Mar 5 (54 days after Jan 10), coupon per period \$5.50. Compute AI.
*Sketch:* $\text{AI} = (54/181) \times 5.50 \approx 1.64$

**Problem 7:** Show algebraically that if $c = 0$, the PV formula reduces to $F \cdot P(0, t_N)$.
*Sketch:* With $c = 0$, $\text{Cpn} = 0$, so $\text{CF}_i = 0$ for $i < N$ and $\text{CF}_N = F$. Thus $P_{\text{dirty}} = F \cdot P(0, t_N)$.

**Problem 8:** A bond is quoted clean; explain why clean price is "smoother" than dirty price around coupon dates.
*Sketch:* Dirty price jumps down by the coupon amount when it's paid (ex-coupon). Clean price subtracts AI, which also resets, so the jump cancels out.

**Problem 9:** Suppose a discount curve shift reduces $P(0,t)$ more for long maturities than short maturities. Which cashflow(s) drive the bond's PV change most, and why?
*Sketch:* The final (largest) cashflow at maturity drives the change most because it's both the largest cashflow and has the largest discount factor change.

**Problem 10:** A trader reports P&L on a clean-price basis. Explain how coupon receipts should appear in P&L versus cash.
*Sketch:* Coupon appears as coupon income (cash inflow) rather than as a price jump. Clean P&L isolates mark-to-market from accrual income.

**Problem 11:** Yield-to-maturity is used to quote price. Explain (conceptually) why two bonds with different coupons can have different yields without implying the higher-yield bond is "cheaper."
*Sketch:* YTM compresses the term structure into one rate. High-coupon bonds have more weight on near-term cashflows; different coupon profiles shift the duration weighting of the yield.

**Problem 12:** Given a clean quote and settlement date, list the steps to compute the settlement cash amount for a position of \$50mm face.
*Sketch:*
1. Convert clean quote to decimal
2. Compute AI using day-count convention
3. Compute dirty = clean + AI
4. Settlement cash = dirty × (\$50mm / 100)

---

## Source Map

### (A) Verified Facts

| Fact | Source |
|------|--------|
| Bond cashflow structure (coupons + principal) | Tuckman Ch 1-4 |
| Discount factor definition and law of one price | Tuckman Ch 1 |
| PV = sum of discounted cashflows | Tuckman Ch 1 |
| Dirty = clean + AI | Tuckman Ch 1-4, Hull Ch 4 |
| Accrued interest pro-rating (166/181 example) | Tuckman Ch 1 |
| U.S. Treasury quoting in 32nds | Hull Ch 4 |
| YTM definition | Tuckman Ch 3 |

### (B) Reasoned Inference

| Inference | Derivation Logic |
|-----------|------------------|
| Curve shifts change PV through discount factors | Direct from PV formula |
| Longer cashflows more sensitive | Discount factors move more for longer maturities |
| Clean pricing removes accrual drift | Follows from $P_{\text{dirty}} = P_{\text{clean}} + \text{AI}$ identity |

### (C) Speculation

- None. All content is either source-verified or logically derived.

---

*Cross-references:*
- Discount factors: see Chapter 2
- Zero/forward/par rates: see Chapter 3
- Day count and accrued interest conventions: see Chapter 4
- DV01 and duration: see Chapter 11
- Yield-to-maturity deep dive: see Chapter 6
