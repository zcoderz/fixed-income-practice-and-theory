# Chapter 4: Compounding, Day Count, Clean vs Dirty Price, and Accrued Interest

---

## Conventions & Notation

### Time and Interest Rates

| Symbol | Definition |
|--------|------------|
| Time | Measured in **years** unless explicitly stated otherwise |
| $\alpha(t_0, t_1)$ | Year fraction (accrual factor) for date interval $[t_0, t_1]$ |
| $R^{(m)}$ | Discrete compounding rate, $m$ times per year |
| $R_c$ | Continuously compounded rate |

### Bond Coupons

| Symbol | Definition |
|--------|------------|
| $c$ | Annual coupon rate on face $F$ |
| $cF/m$ | Coupon per period when there are $m$ coupon payments per year |

### Bond Prices

| Symbol | Definition |
|--------|------------|
| $P$ | Flat / quoted price (a.k.a. **clean price**) |
| $P_{\text{full}}$ | Full / invoice / cash price (a.k.a. **dirty price**) |
| $AI$ | Accrued interest |
| Relation | $P_{\text{full}} = P + AI$ |

### Other Notation

| Symbol | Definition |
|--------|------------|
| $A$ | Principal/invested amount (currency units) |
| $n$ | Time in years |
| $R$ | Nominal annual interest rate (decimal, e.g., 0.06 = 6%) |
| $m$ | Compounding frequency (times per year) |
| $F$ | Bond face value (currency units) |
| $PV(\cdot)$ | Present value operator using discount factors |
| $d(t,T)$ | Discount factor from $t$ to $T$ |
| $r$ | Repo/financing rate (used in P&L decomposition) |

---

## Setup

This chapter is about **conventions**—mechanical "units of measure" that strongly affect:

- **Cash flows** (how much interest/coupon accrues between dates)
- **PV and price** (what you pay/receive at settlement)
- **P&L decomposition** (what is "accrual/carry" versus "mark-to-market")

**Key idea to keep in mind throughout:**

Conventions matter for cash flows and quoted rates, but valuation should be invariant once you fix the correct discount factors and settlement cash flows. Compounding frequency is like changing units (km vs miles). Compounding conventions must be understood to determine cash flows, but "market-clearing prices for cash flows on particular dates are the fundamental quantities."

---

## Conventions Used in This Chapter

### Compounding (Rates)

We use Hull's standard discrete and continuous compounding formulas:

- **Discrete $m$-times-per-year:** $A\left(1 + \frac{R}{m}\right)^{mn}$
- **Continuous:** $Ae^{Rn}$

**Default in examples:** we will quote yields/rates explicitly with their compounding basis (e.g., "6% semiannual").

### Day Count (Year Fraction $\alpha$)

Day count conventions vary by instrument and country. We will compare:

| Convention | Description | Typical Use |
|------------|-------------|-------------|
| **Actual/Actual (in period)** | Interest proportional to actual days elapsed / actual days in coupon period | U.S. Treasuries |
| **30/360** | Assume 30 days per month, 360 days per year | Corporate bonds |
| **Actual/360** | Interest for 90 days is exactly one quarter of the quoted annual rate | Money market |

**Defaults in examples:**
- For coupon-bond accrued interest: **Actual/Actual (in period)** unless stated otherwise
- For money-market simple interest: **Actual/360**

### Clean vs Dirty

| Source | Clean | Dirty |
|--------|-------|-------|
| Tuckman | Flat (quoted) price | Full (invoice) price |
| Hull | Quoted price | Cash price |

Relation: $P_{\text{full}} = P + AI$ (cash price = quoted price + accrued interest)

---

## Core Concepts

### 1) Compounding Conventions

**Formal Definition:**

An annual rate $R$ has meaning only once you specify the compounding convention:

**Compounded $m$ times per year:**

Investing $A$ for $n$ years grows to:

$$A\left(1 + \frac{R}{m}\right)^{mn}$$

**Continuous compounding (limit as $m \to \infty$):**

Investing $A$ for $n$ years grows to:

$$Ae^{Rn}$$

and discounting multiplies by $e^{-Rn}$.

**Intuition:**

Compounding frequency is a unit system for rates. Changing compounding frequency is analogous to converting kilometers and miles—different units describing the same growth factor.

**How It Appears in Trading/Risk/Portfolio Practice:**

- Rate quotes from different markets are often in different "units": e.g., a money-market simple quote (Actual/360) vs a bond yield with semiannual compounding
- Misinterpreting compounding conventions is a common source of:
  - Incorrect discount factors
  - Incorrect PVs
  - Incorrect risk numbers (PV01/DV01 computed off the wrong sensitivity)

---

### 2) Equivalent ("Effective") vs Nominal Rates

**Formal Definition:**

Two rates are **equivalent** if they imply the same accumulation factor over a given horizon.

**Discrete-to-Discrete Conversion:**

$$R_2 = m_2\left[\left(1 + \frac{R_1}{m_1}\right)^{m_1/m_2} - 1\right]$$

so that $A\left(1 + \frac{R_1}{m_1}\right)^{m_1 n} = A\left(1 + \frac{R_2}{m_2}\right)^{m_2 n}$.

**Continuous ↔ Discrete:**

$$R_c = m \ln\left(1 + \frac{R_m}{m}\right)$$

$$R_m = m\left(e^{R_c/m} - 1\right)$$

**Intuition:**

- **What changes** when you change compounding conventions: the numerical value of the quoted rate
- **What does not change** (if you do conversions correctly): the actual cash growth/discount factor over the time horizon

Compounding conventions affect cash flows, but "with respect to valuation, compounding conventions do not matter" once you work with market-clearing prices/discount factors for each date.

**Practice:**

Desk A quotes in semiannual bond yield, desk B uses continuous rates in a model. They must convert using a shared definition or lock to discount factors.

---

### 3) Day Count Conventions and the Year Fraction $\alpha(t_0, t_1)$

**Formal Definition:**

A day count convention defines the year fraction $\alpha(t_0, t_1)$ used to compute interest accrual between two dates. "The day count convention specifies how interest accrues," and conventions vary by country/instrument.

**Common Conventions:**

**Actual/Actual (in period):**

Interest earned is proportional to:

$$\frac{\text{actual days elapsed}}{\text{actual days in reference coupon period}}$$

*Example:* Between March 1 and July 3 there are 124 actual days, and the March 1–Sept 1 coupon period has 184 days, so interest is $124/184$ of the semiannual interest.

**30/360:**

Treat each month as 30 days (year = 360 days). In the same example, March 1 to July 3 counts as $4 \times 30 + 2 = 122$ days and the full period as 180 days, so interest is $122/180$ of the semiannual interest.

**Actual/360:**

Used for money-market quoting; e.g., interest for 90 days is exactly one-quarter of the quoted annual rate.

**Intuition:**

Day count is another "unit system": it changes the mapping from calendar days to year fractions, hence changes interest accrual and the interpretation of a quoted "per annum" rate.

**Practice:**

- Coupon accrual and cash settlement depend on the day count
- Money-market instruments (deposits, FRN coupons in many conventions) use $\alpha$ heavily: coupon $\approx$ rate $\times \alpha \times$ notional

---

### 4) Accrued Interest (AI)

**Formal Definition:**

Accrued interest is the portion of the next coupon payment that has been "earned" since the previous coupon date and is transferred from buyer to seller at settlement.

*Example:* The seller held the bond from Jan 31 to Feb 15 and should receive the corresponding portion of the July 31 coupon; convention makes the buyer pay that amount at settlement.

Accrued interest is computed as a fraction of the coupon based on a day count:

$$AI = \frac{\text{days since last coupon}}{\text{days in coupon period}} \times \text{coupon payment}$$

*Example:* $AI = \frac{60}{60 + 122} \times 6 = 1.9780$ (per 100) for a semiannual coupon of 6.

**Intuition:**

AI "smooths" the transfer of coupon income when trading between coupon dates: the buyer pays the seller for the interest already earned, then receives the full coupon at the next payment date.

**Practice:**

- AI is added to the quoted price to produce the settlement cash amount (invoice/cash price)
- AI is central to carry/accrual P&L and to avoiding confusion around price moves at coupon dates

---

### 5) Clean (Flat/Quoted) vs Dirty (Full/Invoice/Cash) Price

**Formal Definition:**

Flat (quoted) price $P$ plus accrued interest $AI$ equals full (invoice) price:

$$P_{\text{full}} = P + AI$$

And the full (invoice) price is what is set equal to PV of future cash flows:

$$\boxed{P + AI = PV(\text{future cash flows})}$$

**Intuition:**

- The market quotes $P$ (clean) to avoid embedding a mechanically increasing AI into the quote
- But the actual money exchanged is $P_{\text{full}}$ (dirty)

**Practice:**

- Settlement systems, repo financing, and forward pricing use invoice/full/cash values
- Traders often track:
  - **Clean price changes** = mark-to-market on the bond's quoted level
  - **Accrued interest changes** = accrual/carry component

---

## Math and Derivations

### 1) Compounding: Accumulation and Discounting

**Discrete $m$-Times-Per-Year Compounding:**

Terminal value after $n$ years:

$$FV = A\left(1 + \frac{R}{m}\right)^{mn}$$

Discount factor over $n$ years:

$$DF(n) = \left(1 + \frac{R}{m}\right)^{-mn}$$

**Continuous Compounding:**

$$FV = Ae^{Rn}, \qquad DF(n) = e^{-Rn}$$

**Unit/Sanity Checks:**

- $R$ has units "per year," $n$ has units "years," so $Rn$ is unitless
- As $R \to 0$, $DF(n) \to 1$
- As $n \to 0$, $DF(n) \to 1$

---

### 2) Converting Between Compounding Conventions

#### 2.1 Discrete $m_1$ ↔ Discrete $m_2$

Start from the equivalence condition (same growth over any horizon):

$$\left(1 + \frac{R_1}{m_1}\right)^{m_1} = \left(1 + \frac{R_2}{m_2}\right)^{m_2}$$

Rearranged:

$$\boxed{R_2 = m_2\left[\left(1 + \frac{R_1}{m_1}\right)^{m_1/m_2} - 1\right]}$$

**Sanity Checks:**

- If $m_1 = m_2$, then $R_2 = R_1$
- If you increase compounding frequency while holding the effective annual growth fixed, the quoted nominal rate typically decreases slightly

#### 2.2 Discrete $m$ ↔ Continuous

Equivalence:

$$e^{R_c} = \left(1 + \frac{R_m}{m}\right)^m$$

So:

$$\boxed{R_c = m \ln\left(1 + \frac{R_m}{m}\right)}$$

And inverse:

$$\boxed{R_m = m\left(e^{R_c/m} - 1\right)}$$

**Sanity Checks:**

- For small rates, $\ln(1+x) \approx x$, so $R_c \approx R_m$ when $R_m$ is small and $m$ is not too small

---

### 3) Day Count and Interest Accrual

**Actual/Actual (in period) (Treasuries):**

Interest earned over a partial coupon period is a fraction of the coupon equal to:

$$\frac{\text{actual days elapsed}}{\text{actual days in coupon period}}$$

**30/360 (Corporates):**

Compute "30/360 days" by assuming 30 days per month and 360 per year.

**Actual/360 (Money Market):**

Interest for 90 days is one-quarter of the annual quote:

$$\text{Interest} = \text{Principal} \times R \times \frac{\text{days}}{360}$$

Similarly, simple/"money market" interest in day-count form (example with 181 days):

$$\frac{181 \, r_s}{360} = 2.50\%$$

---

### 4) Clean vs Dirty Price, and Why "Invoice Price" Is What PV Targets

**Key Identity:**

$$P + AI = PV(\text{future cash flows})$$

**Interpretation:**

$P$ (flat/quoted) adjusts so that once you add $AI$, the settlement amount equals PV. Therefore, "the particular market convention used in calculating accrued interest does not really matter" because the flat price adjusts; "the only quantity that matters is the invoice price."

**Coupon-Date Behavior:**

Let $P_b$ be quoted price right before coupon, $P_a$ right after. Right before coupon, $AI = c/2$, and the PV of the next coupon is essentially $c/2$, so:

$$P_b + \frac{c}{2} = \frac{c}{2} + PV(\text{cash flows after next coupon})$$

which implies:

$$P_b = PV(\text{cash flows after next coupon})$$

Right after coupon, $AI = 0$, so:

$$P_a = PV(\text{cash flows after next coupon})$$

Thus, if yields don't change, **$P_a = P_b$**: the quoted price need not "drop by the coupon" under this clean/full convention.

> **Note:** Market charts sometimes show different "price series" (clean, dirty, total-return). The no-jump result above is about quoted price under the accrued-interest convention and a particular timing idealization.

---

## Measurement & Risk

### 1) Clean/Dirty/Accrual and Daily P&L

**P&L Decomposition for Holding a Bond:**

Let $P(0), P(d)$ be flat prices; $AI(0), AI(d)$ accrued; $r$ repo rate; $c$ coupon rate; $D$ actual days in the coupon period. Then:

$$P\&L = P(d) + AI(d) - (P(0) + AI(0))(1 + rd/360)$$

$$= \underbrace{[P(d) - P(0)]}_{\text{mark-to-market on clean}} + \underbrace{[AI(d) - AI(0)]}_{\text{accrual}} - \underbrace{(P(0) + AI(0))(rd/360)}_{\text{financing}}$$

$$= \text{Price change} + \text{Interest income} - \text{Financing cost}$$

$$= \text{Price change} + \text{Carry}$$

Also, the accrual term is often approximately linear between coupon dates:

$$AI(d) - AI(0) \approx \frac{cd}{D}$$

(per dollar principal).

**Practical Reading:**

- **Accrual P&L:** the change in $AI$ (and coupon receipts when they occur)
- **Mark-to-market P&L:** the change in the quoted/clean price $P$
- **Total desk P&L:** includes both plus financing, and the financing and accrual can use different day counts (actual/actual vs 30/360)

### 2) How Day Count and Compounding Conventions Change PV01/DV01 Numerically

Even for a single cash flow, PV sensitivity depends on conventions because the mapping from "a 1 bp change in the quote" to a discount factor change depends on compounding and day count.

**Worked Demonstration (1-Year Zero, 100 Face): Compounding Changes PV01**

**Continuous compounding:** $P(r) = 100e^{-r}$. Then:

$$\frac{dP}{dr} = -100e^{-r} = -P(r) \Rightarrow PV01 \approx P(r) \times 10^{-4}$$

Using $r = 5\% = 0.05$:

$$P = 100e^{-0.05} = 95.1229, \quad PV01 \approx 0.009512$$

**Annual compounding:** $P(y) = \frac{100}{1+y}$. Then:

$$\frac{dP}{dy} = -\frac{100}{(1+y)^2}, \quad PV01 \approx \frac{100}{(1+y)^2} \times 10^{-4}$$

At $y = 5\% = 0.05$:

$$P = \frac{100}{1.05} = 95.2381, \quad PV01 \approx \frac{100}{1.05^2} \times 10^{-4} = \frac{100}{1.1025} \times 10^{-4} = 0.009070$$

**Takeaway:** "1 bp of yield" is not the same perturbation across compounding conventions; you must lock the convention (or work directly with discount factors).

**Day Count Changes PV01 Through $\alpha$:**

For a simple money-market interest amount $\text{Interest} = \text{Principal} \times R \times \alpha$, PV and PV01 scale with $\alpha$: larger $\alpha$ ⇒ larger sensitivity.

---

## Worked Examples

### Example A: Convert an Interest Rate Quote Across Compounding Bases

**Given:** A quoted rate $R^{(2)} = 6.00\%$ per annum with semiannual compounding ($m = 2$).

**Goal:** Find equivalent annual-compounded, quarterly-compounded, continuous, and (1-year) simple rates.

**Step 1: Compute the effective 1-year growth factor**

$$\text{Growth over 1 year} = \left(1 + \frac{0.06}{2}\right)^2 = 1.03^2 = 1.0609$$

**Step 2: Equivalent annual-compounded rate $R^{(1)}$**

Annual compounding means growth $= 1 + R^{(1)}$. So:

$$1 + R^{(1)} = 1.0609 \Rightarrow R^{(1)} = 0.0609 = 6.09\%$$

**Step 3: Equivalent quarterly-compounded rate $R^{(4)}$**

Using the frequency conversion formula with $m_1 = 2, R_1 = 0.06, m_2 = 4$:

$$R^{(4)} = 4\left[\left(1 + \frac{0.06}{2}\right)^{2/4} - 1\right] = 4\left[1.03^{1/2} - 1\right]$$

Compute $1.03^{1/2} \approx 1.014889$. Then:

$$R^{(4)} \approx 4(0.014889) = 0.059556 \approx 5.9556\%$$

**Step 4: Equivalent continuously compounded rate $R_c$**

$$R_c = m \ln\left(1 + \frac{R_m}{m}\right) = 2\ln(1.03) \approx 2(0.0295588) = 0.0591176 \approx 5.9118\%$$

**Step 5: Equivalent simple rate for a 1-year horizon**

A (1-year) simple rate $R_{\text{simp,1y}}$ satisfies $1 + R_{\text{simp,1y}} = 1.0609$, so:

$$R_{\text{simp,1y}} = 6.09\%$$

**Key Warning:** A simple rate is horizon-dependent (you must specify the maturity). Discrete and continuous compounding formulas are defined for any $n$.

---

### Example B: Compute a Year Fraction Under ACT/360 and 30/360

**Dates:** $t_0 =$ March 8, 2022; $t_1 =$ June 8, 2022.

Assume actual days = 92 days.

**Convention 1: Actual/360**

$$\alpha_{\text{ACT/360}} = \frac{92}{360} = 0.255555\ldots$$

**Convention 2: 30/360**

From March 8 to June 8 is exactly 3 months, so under 30/360 the day count is $3 \times 30 = 90$ days:

$$\alpha_{30/360} = \frac{90}{360} = 0.25$$

**Impact on Coupon/Interest Amount:**

Take notional $N = 10,000,000$ and annual rate $R = 6\%$.

**ACT/360 interest:**

$$I_{\text{ACT/360}} = N \cdot R \cdot \alpha = 10,000,000 \cdot 0.06 \cdot 0.255555\ldots = 153,333.33$$

**30/360 interest:**

$$I_{30/360} = 10,000,000 \cdot 0.06 \cdot 0.25 = 150,000.00$$

**Difference:**

$$\Delta I = 3,333.33$$

Same calendar interval, different day count ⇒ different accrued amount.

---

### Example C: Fixed-Rate Bond Between Coupon Dates — Compute Accrued Interest

**Bond:**
- $c = 5.50\%$ annual coupon, paid semiannually ⇒ coupon payment per 100 face is $c/2 = 2.75$
- Coupon dates: Jan 31 and Jul 31
- Settlement date: Feb 15, 2001
- Days in coupon period: $D = 181$ days; days accrued since last coupon = $181 - 166 = 15$

**Step 1: Accrued Interest per 100 Face**

Semiannual coupon per 100 face = 2.75. Accrued fraction = $15/181$. So:

$$AI = 2.75 \times \frac{15}{181} = 2.75 \times 0.0828729 = 0.2279006$$

This equals $0.2279\%$ of face, or \$22.79 per \$10,000 face.

**Step 2: Convert the Quoted Bond Price and Compute Full (Dirty) Price**

Quoted/flat price on Feb 15, 2001: $101-4\frac{5}{8}$.

Convert:

$$P = 101 + \frac{4.625}{32} = 101.14453125$$

Then:

$$P_{\text{full}} = P + AI = 101.14453125 + 0.2279006 = 101.3724319 \approx 101.3724$$

**Step 3: Invoice Amount for \$10,000 Face**

$$\text{Invoice} = 10,000 \times \frac{101.3724319}{100} \approx \$10,137.24$$

**Key Identity:**

When $AI \neq 0$, the statement "price equals PV of cash flows" must be generalized to:

$$P + AI = PV(\text{future cash flows})$$

---

### Example D: Coupon-Date Behavior — Clean vs Dirty vs "Total Value"

**Assume:**
- Semiannual coupon payment = $c/2 = 3.00$ per 100 face
- Yields unchanged across the coupon date
- PV of cash flows after the just-paid coupon is 100.00

**Timeline (per 100 face):**

| Time | Clean $P$ | Accrued $AI$ | Dirty $P_{\text{full}}$ | Coupon Cash Received | Total Value |
|------|-----------|--------------|------------------------|---------------------|-------------|
| Just after previous coupon | 100.00 | 0.00 | 100.00 | 0.00 | 100.00 |
| Midway to next coupon | 100.00 | 1.50 | 101.50 | 0.00 | 101.50 |
| Right before coupon | 100.00 | 3.00 | 103.00 | 0.00 | 103.00 |
| Right after coupon payment | 100.00 | 0.00 | 100.00 | 3.00 | 103.00 |

**What You See:**

- **Quoted/clean $P$:** smooth (here flat for simplicity)
- **Dirty/full $P_{\text{full}}$:** ramps up as $AI$ accrues, then drops by the coupon when $AI$ resets
- **Total value (dirty + coupon cash at payment):** continuous across the coupon date

**Convention Flag:**

If you focus on the ex-coupon traded price process (and ignore the coupon cash that appears at the payment date), you will observe a downward jump at the payment date. In fixed income, it is often clearer to track (dirty price + coupon cash) for continuity of value.

---

### Example E: Same Bond PV Under Different Compounding Conventions — "Rate Conversion Errors"

**Principle:** Compounding conventions shouldn't matter for valuation once you use the correct market-clearing prices for dated cash flows. The PV error happens when you misinterpret a rate quote (i.e., you fail to convert), producing the wrong discount factor.

**Bond:** A single cash flow of 100 at $T = 2$ years.

**Step 1: "True" Curve in Continuous Compounding**

Assume the true (model) continuously compounded rate is $R_c = 5\%$ for all maturities.

Discount factor:

$$DF_{\text{true}}(2) = e^{-0.05 \times 2} = e^{-0.10} = 0.904837$$

PV:

$$PV_{\text{true}} = 100 \times 0.904837 = 90.4837$$

**Step 2: Correct Annual-Compounded Equivalent Rate**

To match the same discount factor with annual compounding:

$$(1+y)^{-2} = e^{-0.10} \Rightarrow 1+y = e^{0.05} \Rightarrow y = e^{0.05} - 1 = 0.051271 = 5.1271\%$$

If you use $y = 5.1271\%$ with annual compounding, you recover the same PV:

$$PV = 100(1+y)^{-2} = 100e^{-0.10} = 90.4837$$

**Step 3: Rate Conversion Error: Treating "5%" as Annual-Compounded Without Conversion**

If someone incorrectly discounts using annual compounding at $y = 5\%$ (instead of 5.1271%):

$$DF_{\text{wrong}}(2) = (1.05)^{-2} = \frac{1}{1.1025} = 0.907029$$

$$PV_{\text{wrong}} = 100 \times 0.907029 = 90.7029$$

**PV Difference:**

$$\Delta PV = PV_{\text{wrong}} - PV_{\text{true}} = 90.7029 - 90.4837 = 0.2192$$

**Why Practitioners "Lock Conventions":**

A quoted number (like "5%") is incomplete without its compounding and day-count basis. "Its precise meaning depends on the way the interest rate is measured."

The safest invariant object is the discount factor curve (or equivalently, a clearly defined zero-rate convention).

---

## Practical Notes

### Common Quoting Gotchas

| Issue | Description |
|-------|-------------|
| **Treasury bill discount quotes** | An "8% quote" on a 91-day bill corresponds to interest earned of $0.08 \times 91/360 = 2.0222\%$ of face, and the "true" annualized rate depends on dividing by the price paid (97.9778) and annualizing. Discount-yield style quotes are not the same as investment yields. |
| **Bond price in 32nds** | Quote 100-16 means $100 + 16/32 = 100.5$ |
| **Quoted (clean) vs cash (dirty)** | Quoted price excludes accrued; settlement uses quoted plus accrued |
| **Day-count mismatches** | Coupon accrual and repo interest can use different day counts (actual/actual vs 30/360), which affects carry calculations |

### Implementation Pitfalls

| Pitfall | Description |
|---------|-------------|
| **Business-day rolling rules** | Must be explicit. "Modified-following" rolling: roll to the next business day unless it crosses a month boundary, then roll back |
| **Payment lag vs accrual end date** | Build an unadjusted schedule, then roll dates to business days; payment date may differ from accrual end date |
| **30/360 variants and EOM rules** | Exact 30/360 "end-of-month" adjustment rule depends on instrument type, jurisdiction, and governing convention |
| **Rounding** | Accrued interest and invoice prices are often rounded to cents (cash) and to 1/32 or 1/64 (quotes). Round late in computations and validate invariants |

### Verification Tests (Quick Sanity Checks)

1. **Accrued interest bounds:** For a standard coupon period:
   $$0 \leq AI < \text{coupon payment}$$
   and $AI = 0$ on coupon dates (by definition)

2. **Invoice identity:** Verify $P_{\text{full}} = P + AI$ every day; compare against system "cash price"

3. **Coupon-date jump checks:**
   - If yields unchanged, quoted price need not jump on coupon date; jumps should be explainable via $AI$ reset and coupon cash
   - **Dirty price:** expect to have a coupon-date drop if you observe it just before vs just after payment
   - **Total value:** expect $P_{\text{full}} + \text{coupon cash}$ to be continuous absent yield moves

---

## Summary & Recall

### 10-Bullet Executive Summary

1. A rate quote is incomplete without its compounding convention; compounding changes the numerical rate but (if converted correctly) not the implied growth factor
2. Discrete compounding: $FV = A(1 + R/m)^{mn}$
3. Continuous compounding: $FV = Ae^{Rn}$, discounting uses $e^{-Rn}$
4. Conversion formulas are mechanical: discrete↔discrete and discrete↔continuous must be used to avoid PV errors
5. Day count determines the year fraction $\alpha(t_0, t_1)$ used in interest accrual; conventions vary across products and countries
6. Common day counts: Actual/Actual (Treasuries), 30/360 (corporates), Actual/360 (money market)
7. Accrued interest allocates coupon income between buyer and seller when trading between coupon dates
8. Clean vs dirty: $P_{\text{full}} = P + AI$. The invoice (full) price is what equates to PV of future cash flows
9. P&L decomposes into clean price change (MTM) + accrued change (accrual) − financing; carry depends on day counts and on whether financing is applied to full price
10. Coupon-date "jumps" depend on what you plot: quoted price can be smooth while dirty price drops by the coupon when accrued resets; total value including coupon cash is continuous when yields are unchanged

---

### Cheat Sheet of Key Formulas/Identities

| Formula | Expression |
|---------|------------|
| Discrete compounding | $FV = A(1 + R/m)^{mn}$, $DF = (1 + R/m)^{-mn}$ |
| Continuous compounding | $FV = Ae^{Rn}$, $DF = e^{-Rn}$ |
| Discrete↔Discrete | $R_2 = m_2\left[\left(1 + \frac{R_1}{m_1}\right)^{m_1/m_2} - 1\right]$ |
| Discrete↔Continuous | $R_c = m\ln(1 + R_m/m)$, $R_m = m(e^{R_c/m} - 1)$ |
| ACT/ACT (in period) | fraction $= \frac{\text{actual days elapsed}}{\text{actual days in period}}$ |
| 30/360 | treat each month as 30 days |
| ACT/360 | interest $\propto$ days/360 |
| Clean/dirty & PV | $P_{\text{full}} = P + AI$, $P + AI = PV(\text{future cash flows})$ |
| Holding-period P&L | $P\&L = P(d) + AI(d) - (P(0) + AI(0))(1 + rd/360)$ |

---

## Flashcards (Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What does the compounding frequency $m$ mean? | The number of times per year interest is compounded; it defines the unit system for a rate |
| 2 | Terminal value under $m$-times-per-year compounding? | $A(1 + R/m)^{mn}$ |
| 3 | Terminal value under continuous compounding? | $Ae^{Rn}$ |
| 4 | Discounting under continuous compounding? | Multiply by $e^{-Rn}$ |
| 5 | Discrete-to-discrete conversion formula? | $R_2 = m_2[(1 + R_1/m_1)^{m_1/m_2} - 1]$ |
| 6 | Discrete-to-continuous conversion? | $R_c = m\ln(1 + R_m/m)$ |
| 7 | Continuous-to-discrete conversion? | $R_m = m(e^{R_c/m} - 1)$ |
| 8 | What is a day count convention? | A rule for turning dates into a year fraction $\alpha$ for interest accrual |
| 9 | ACT/ACT (in period) accrual fraction? | actual days elapsed / actual days in coupon period |
| 10 | 30/360 basic idea? | Assume 30-day months and 360-day years |
| 11 | ACT/360 basic idea? | Interest accrues as days/360 of the annual quote |
| 12 | Define accrued interest (AI) in bond trading | The portion of the next coupon earned since the last coupon date that the buyer pays the seller at settlement |
| 13 | Clean vs dirty price relation? | $P_{\text{full}} = P + AI$ |
| 14 | What is the invoice price? | The full (dirty) price paid/received at settlement |
| 15 | Correct PV statement when $AI \neq 0$? | $P + AI = PV(\text{future cash flows})$ |
| 16 | Why doesn't the specific AI convention "really matter"? | The quoted price adjusts; the invoice price is what must equal PV |
| 17 | Give the P&L decomposition for holding a bond financed at repo | Price change + interest income − financing cost (carry) |
| 18 | Why can carry differ from coupon rate minus repo rate? | Coupon accrues on face; repo applies to full price; and day counts differ (actual/actual vs 30/360) |
| 19 | What is "cash price" in Hull's bond quotation example? | Quoted price + accrued interest |
| 20 | What's the safest invariant object to carry across systems? | The discount factors (market-clearing PVs for each cash-flow date), not raw rate quotes |

---

## Mini Problem Set

### Questions

1. A rate is quoted as 8% per annum with quarterly compounding. Convert it to an equivalent continuously compounded rate.
   *Sketch:* Use $R_c = m\ln(1 + R_m/m)$ with $m = 4$.

2. Convert 5% continuously compounded to an equivalent semiannual-compounded rate.
   *Sketch:* Use $R_m = m(e^{R_c/m} - 1)$ with $m = 2$.

3. For dates April 1 to July 1 (inclusive start, exclusive end), compute $\alpha$ under ACT/360 and 30/360.
   *Sketch:* ACT/360: count actual days; divide by 360. 30/360: 3 months → 90 days → 90/360.

4. A corporate bond has semiannual coupon 4 per 100. Using 30/360, compute AI for a settlement date 50 "30/360 days" after the last coupon in a 180-day period.
   *Sketch:* $AI = (50/180) \times 4$.

5. A bond is quoted at 99.25 and has AI = 1.10. What is the invoice price?
   *Sketch:* $P_{\text{full}} = P + AI$.

6. Using Tuckman's P&L identity, explain which term corresponds to "accrual P&L" over $d$ days.
   *Sketch:* In $P\&L = [P(d) - P(0)] + [AI(d) - AI(0)] - \text{financing}$, accrual P&L is $[AI(d) - AI(0)]$ (plus actual coupon cash when paid).

7. Show numerically how a 2-year discount factor differs if you treat a 6% semiannual-compounded quote as if it were continuously compounded without conversion.

8. A money-market deposit accrues interest using ACT/360. If someone mistakenly uses ACT/365, how does the accrued interest over 120 days change?

9. You are given a clean price time series for a bond and want to compute total return over a period containing a coupon date. What extra cash-flow adjustments do you need?

10. Explain (in words) why the quoted/clean price can be insensitive to coupon payment timing under the accrued-interest convention when yields don't change.

11. Describe how modified-following date rolling works and why it exists.

12. In Tuckman's carry discussion, list two reasons why "coupon rate − repo rate" is not exactly carry.

---

## Source Map

### (A) Verified Facts

- Discrete compounding terminal value $A(1 + R/m)^{mn}$ and continuous compounding $Ae^{Rn}$ with discounting $e^{-Rn}$
- Discrete↔discrete and discrete↔continuous conversion formulas
- Day count conventions (Actual/Actual, 30/360, Actual/360) with examples
- Clean/dirty price relation $P_{\text{full}} = P + AI$ and PV identity $P + AI = PV(\text{future cash flows})$
- Coupon-date behavior: quoted price need not jump when yields unchanged
- P&L decomposition into price change + interest income − financing cost
- Compounding frequency as "unit system" analogy

### (B) Reasoned Inference (Derived from A)

- PV01 differs under different compounding conventions for the same "1 bp" move
- Day count changes affect accrual amounts mechanically
- The invoice price is the invariant valuation anchor; quoted price adjusts to convention

### (C) Speculation (Clearly Labeled)

- Exact 30/360 "end-of-month" adjustment rule depends on instrument type, jurisdiction, and governing convention—needs specific market standard documentation
- Rounding conventions vary by market/venue
