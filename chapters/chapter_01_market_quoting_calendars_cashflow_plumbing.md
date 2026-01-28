# Chapter 1: Market Quoting, Calendars, and Cashflow Plumbing

---

## Introduction

Before you can price a bond, value a swap, or compute the risk of a portfolio, you must answer a deceptively simple question: *what exactly are we talking about?*

A trader says a Treasury is "bid at 99-16." A salesperson quotes a swap rate of "3.25%." A risk report shows duration of "7.2 years." Each of these numbers carries implicit assumptions about day counts, compounding, settlement timing, and dozens of other conventions that turn abstract rates into actual dollars. Get any of these conventions wrong, and your "price" becomes meaningless—or worse, you wire the wrong amount on settlement day.

This chapter is about the **plumbing**: the infrastructure of market conventions that translates quotes into cashflows and cashflows into cash exchanged. We cover:

1. **How markets quote instruments** — price vs yield vs spread, and why each market chooses its convention
2. **Day counts and compounding as a unit system** — the "measurement units" of interest rates
3. **Settlement, calendars, and timing** — when cash actually moves, what happens when it doesn't (fails), and how overnight rates compound
4. **Clean vs dirty pricing** — how bond markets avoid artificial price jumps (a preview; full treatment in Chapter 5)

None of this is glamorous. But every mispriced trade, every reconciliation break, and every "mystery P&L" you will encounter in your career traces back to someone getting the plumbing wrong. For those in middle office—risk, operations, product control—this chapter explains *why* the P&L breaks you investigate occur. For those moving to the trading desk, it provides the vocabulary and mechanical understanding that traders assume you already have.

---

## 1.1 What Markets Quote: Price, Yield, and Spread

### 1.1.1 The Quote Is a Communication Protocol

Every market has developed conventions for how participants communicate value. These conventions reflect what is *useful* for that market's participants—not what is mathematically fundamental.

**Cash bonds** are typically quoted as a **price** (per 100 face value). This makes sense: the price directly tells you the dollars exchanged per unit of face value.

**Money markets** quote **rates** (annualized). A 3-month deposit rate of 5% tells you the interest earned over that period, scaled to an annual basis.

**Credit default swaps** quote **spreads** (in basis points). The spread is the annual premium the protection buyer pays.

**Interest rate swaps** quote the **fixed rate** that makes the swap have zero value at inception—the par swap rate.

Each quote type serves its market. But here is the critical point: **the quote alone is not enough information to calculate cash**. You also need to know the conventions that interpret that quote.

### 1.1.2 Bonds: Clean Price and 32nds

U.S. Treasury bonds are quoted in dollars and 32nds per $100 face value. Tuckman notes that bond prices "are expressed as a percent of face value and that numbers after the hyphens denote 32nds, often called ticks." The notation "99-16" means:

$$99 + \frac{16}{32} = 99.50$$

> **Anatomy of a Quote:** `99-16+`
>
> | Part | Component | Value | Calculated As |
> | :--- | :--- | :--- | :--- |
> | **99** | Handle | 99 | $99$ |
> | **16** | Ticks (32nds) | 16/32 | $0.5$ |
> | **+** | Plus (1/64th) | 1/64 | $0.015625$ |
> | **Total** | | **99.515625** |
>
> *Note: The '+' is not a plus sign in the algebraic sense of adding 1; it's a specific shorthand for 'half a tick' (1/64).*

Finer increments use fractions of 32nds. The quote "101-04⁵⁄₈" means:

$$101 + \frac{4 + \frac{5}{8}}{32} = 101 + \frac{4.625}{32} = 101.1445$$

**Tick Precision Across Instruments:** Different Treasury futures contracts use different precision levels. Hull notes that 10-year Treasury note futures are "quoted to the nearest half of a thirty-second," so a settlement price of "139-025" means $139 + \frac{2.5}{32} = 139.078125$. The 5-year and 2-year Treasury note contracts are quoted "to the nearest quarter of a thirty-second," so "125-132" means $125 + \frac{13.25}{32} = 125.4140625$.

This is the **clean** or **flat** price—it excludes accrued interest. The actual cash exchanged (the **invoice** or **dirty** price) equals the clean price plus accrued interest.

Why quote clean? Because if markets quoted dirty prices, bond prices would mechanically drift upward between coupon dates as interest accrued, then drop on coupon payment. Clean pricing removes this sawtooth pattern, making prices more comparable across dates.

> **Source:** Hull confirms that Treasury bonds are "quoted in dollars and thirty-seconds of a dollar" and that "the quoted price, which traders refer to as the clean price, is not the same as the cash price paid by the purchaser of the bond, which is referred to by traders as the dirty price."

> **Desk Reality: Bid-Ask Spreads and Tick Size**
>
> Tuckman notes that "bid-ask spreads are particularly low" for liquid securities, especially on-the-run Treasuries. A typical bid-ask spread for the on-the-run 10-year might be 1/64th (half a tick), meaning the market-maker buys at 99-16 and sells at 99-16+.
>
> Why does this matter? On $100 million face, a one-tick (1/32) bid-ask spread costs:
> $$\$100{,}000{,}000 \times \frac{1}{32} \times \frac{1}{100} = \$31{,}250$$
>
> When a trader says "I'm offered at 16-plus," they mean they're willing to sell at 99-16+. "Bid at 16" means they're willing to buy at 99-16. The half-tick spread is the dealer's compensation for providing liquidity.

### 1.1.3 T-Bill Quotations: Discount Rate Convention

Treasury bills use a different quoting convention entirely. Hull explains that T-bill prices "are sometimes quoted using a discount rate. This is the interest earned as a percentage of the final face value rather than as a percentage of the initial price paid."

If a 91-day Treasury bill is quoted at 8, this means the rate of interest earned is 8% of face value per 360 days. For a $100 face value bill:

$$\text{Interest} = \$100 \times 0.08 \times \frac{91}{360} = \$2.0222$$

The cash price is therefore $\$100 - \$2.0222 = \$97.9778$.

The relationship between the quoted discount rate $P$ and the cash price $Y$ for a T-bill with $n$ days remaining is:

$$\boxed{P = \frac{360}{n}(100 - Y)}$$

**Why This "Understates" True Return:** The discount yield convention divides interest by face value rather than by the actual investment amount. The true rate of return for the 91-day example is:

$$\text{True return} = \frac{2.0222}{97.9778} = 2.064\% \text{ for 91 days}$$

Annualized on a simple interest basis: $\frac{2.064\%}{91} \times 365 = 8.28\%$, which exceeds the quoted 8%.

> **Worked Example A: From Discount Quote to Cash Price**
>
> A 182-day T-bill is quoted at a discount rate of 4.50%.
>
> **Step 1:** Calculate interest as percentage of face:
> $$\text{Interest} = 100 \times 0.045 \times \frac{182}{360} = 2.275$$
>
> **Step 2:** Calculate cash price:
> $$Y = 100 - 2.275 = 97.725$$
>
> **Sanity check:** The true yield is $\frac{2.275}{97.725} = 2.328\%$ for 182 days, or $4.67\%$ annualized (bond-equivalent yield), which exceeds the quoted 4.50% discount rate. ✓

### 1.1.4 Money Markets: Rates with Day Counts

Money market instruments quote rates, not prices. But a "5% rate" is meaningless without knowing:

1. **Day count convention**: How do we measure the time period?
2. **Compounding convention**: Simple interest? Compound interest?

A 3-month rate (e.g., legacy LIBOR or Term SOFR) of 5% under **Actual/360** with simple interest means that for a loan of $1 over $d$ actual days:

$$\text{Interest} = 1 \times 0.05 \times \frac{d}{360}$$

The denominator is 360 (not 365 or actual days in the year). This is not a mathematical choice—it is a market convention that affects the actual dollars owed.

> **Source:** Tuckman explains that "lending $\$1$ for $d$ days at a rate of $r$ will earn the lender an interest payment of $r \times d/360$ dollars at the end of the $d$ days" under the actual/360 convention.

### 1.1.5 Why Conventions Differ Across Markets

Conventions evolved historically and persist because changing them would require coordinated action across thousands of market participants, legal documents, and systems. A few examples:

| Market | Quote Convention | Day Count | Compounding |
|--------|------------------|-----------|-------------|
| U.S. Treasuries | Price (32nds), clean | ACT/ACT | Semiannual |
| U.S. Corporate bonds | Price (decimals), clean | 30/360 | Semiannual |
| USD term money market | Rate | ACT/360 | Simple |
| USD swaps (fixed leg) | Rate | 30/360 or ACT/360 | Varies |
| USD swaps (floating leg) | Rate | ACT/360 | Simple |
| EUR/GBP money market deposits | Rate | ACT/360 (EUR), ACT/365 (GBP) | Simple |

O'Kane notes that "in US dollars, Japanese yen and euros, the standard money-market deposit convention is Actual/360. For UK sterling, Actual/365 is used." These differences are not arbitrary annoyances—they reflect each market's history and what information participants find most useful. But they create traps for anyone who moves between markets without adjusting their mental model.

---

## 1.2 Day Counts: The Unit System of Interest Rates

### 1.2.1 Day Counts Define How Time Is Measured

Hull provides the canonical definition: "The day count defines the way in which interest accrues over time." More precisely, a day count convention determines two things:

1. How to count days between two dates (the numerator)
2. How to define the "reference period" (the denominator)

The interest earned between two dates is then:

$$\boxed{\text{Interest} = \frac{\text{Number of days between dates}}{\text{Number of days in reference period}} \times \text{Interest earned in reference period}}$$

This fraction—(days between dates) / (days in reference period)—is the **accrual fraction** or **year fraction**, often denoted $\Delta$ or $\tau$.

> **Source:** Hull states: "The day count convention is usually expressed as $X/Y$. When we are calculating the interest earned between two dates, $X$ defines the way in which the number of days between the two dates is calculated, and $Y$ defines the way in which the total number of days in the reference period is measured."

### 1.2.2 The Three Common U.S. Conventions

**Actual/Actual (in period):** Used for U.S. Treasury bonds. Both numerator and denominator use actual calendar days.

- Numerator: actual days between dates
- Denominator: actual days in the coupon period

**Worked Example B (from Hull):** Consider a Treasury bond with coupon payment dates March 1 and September 1, paying an 8% annual coupon ($4 semiannually). To calculate interest earned between March 1 and July 3:

- Reference period: March 1 to September 1 = 184 actual days
- Days elapsed: March 1 to July 3 = 124 actual days
- Interest earned: $\frac{124}{184} \times \$4 = \$2.6957$

**30/360:** Used for U.S. corporate and municipal bonds. Assumes 30 days per month and 360 days per year.

- Numerator: computed assuming each month has 30 days
- Denominator: 360

Hull notes that with 30/360, "we assume 30 days per month and 360 days per year when carrying out calculations." For the same March 1 to July 3 period:

- Days elapsed: $(4 \times 30) + 2 = 122$ days (4 full months plus 2 days)
- Reference period: 180 days (6 months × 30 days)
- Interest earned: $\frac{122}{180} \times \$4 = \$2.7111$

### 1.2.3 The 30/360 Day Counting Algorithm

The 30/360 convention requires a specific algorithm for counting days. Tuckman provides a detailed example: to count days from August 27, 2001 to February 15, 2002:

| From | To | Days |
|------|-----|------|
| 8/27/01 | 8/30/01 | 3 |
| 8/30/01 | 9/30/01 | 30 |
| 9/30/01 | 10/30/01 | 30 |
| 10/30/01 | 11/30/01 | 30 |
| 11/30/01 | 12/30/01 | 30 |
| 12/30/01 | 1/30/02 | 30 |
| 1/30/02 | 2/15/02 | 15 |
| **Total** | | **168** |

The general formula for 30/360 day count between dates $(Y_1, M_1, D_1)$ and $(Y_2, M_2, D_2)$ is:

$$\boxed{\text{Days} = 360(Y_2 - Y_1) + 30(M_2 - M_1) + (D_2 - D_1)}$$

with adjustments at month boundaries (e.g., if $D_1 = 31$, set $D_1 = 30$).

**Actual/360:** Used for USD money markets. Actual calendar days, but divided by 360.

- Numerator: actual days between dates
- Denominator: 360 (always)

This means a full year of 365 days earns $365/360 \approx 1.0139$ times the quoted rate, not exactly the quoted rate. Hull confirms: "the interest earned in a whole year of 365 days is $365/360$ times the quoted rate."

### 1.2.4 International Day Count Variants

Day count conventions vary not only across instruments but across regions and specific market standards. Beyond the three core U.S. conventions, practitioners encounter several variants:

| Convention | Numerator | Denominator | Primary Use |
|------------|-----------|-------------|-------------|
| ACT/ACT (ISDA) | Actual days | Actual days in year | Treasury bonds |
| ACT/ACT (ICMA) | Actual days | Actual days × frequency | Eurobonds |
| ACT/365 Fixed | Actual days | 365 (always) | GBP, AUD, CAD money markets |
| ACT/360 | Actual days | 360 | USD, EUR, JPY money markets |
| 30/360 (Bond Basis) | 30-day months | 360 | U.S. corporates |
| 30E/360 (Eurobond) | 30-day months, different end-of-month rules | 360 | Eurobonds |
| 30E/360 (ISDA) | 30-day months | 360 | ISDA swaps |

O'Kane emphasizes that "every time we encounter a day count fraction, we must take care to apply the correct day count basis convention."

> **Practitioner Note:** The difference between ACT/365 Fixed and ACT/360 can be significant. For a 365-day loan at 5%:
> - ACT/360: Interest = $\frac{365}{360} \times 5\% = 5.069\%$ effective
> - ACT/365: Interest = $\frac{365}{365} \times 5\% = 5.000\%$ effective
>
> This 6.9bp difference on $100mm is $6,944 per year—real money on a funding desk.

### 1.2.5 Why Day Counts Matter: The "Million Dollar Calendar Mistake"

Hull provides a vivid illustration in Business Snapshot 6.1. Between February 28 and March 1 in a non-leap year:

- Under **30/360**: There are 3 days (Feb 28 → Feb 30 → Mar 1)
- Under **ACT/ACT**: There is 1 day

If a corporate bond and a Treasury bond have the same coupon and same quoted price, the corporate bond (using 30/360) accrues **three times as much interest** over this one-day period as the Treasury (using ACT/ACT).

> **Source:** Hull's Business Snapshot 6.1: "Between February 28, 2018, and March 1, 2018, you have a choice between owning a U.S. government bond and a U.S. corporate bond. They pay the same coupon and have the same quoted price... In fact you should have a marked preference for the corporate bond."

This is not a mathematical curiosity—it affects which bond you should prefer to hold over that date.

> **Case Study: The "Million Dollar Calendar Mistake"**
>
> **The Scenario**: You hold $100 million face value of 5% coupon bonds overnight from February 28 to March 1 (non-leap year).
>
> **The Trap**:
> *   **Corporate Bond (30/360)**: Counts as **3 days** of accrual (Feb 28 $\to$ Feb 30 $\to$ Mar 1).
>     *   Interest = $\$100m \times 5\% \times \frac{3}{360} \approx \$41,666$.
> *   **Treasury Bond (Act/Act)**: Counts as **1 day** of accrual (Feb 28 $\to$ Mar 1).
>     *   Interest = $\$100m \times 5\% \times \frac{1}{365} \approx \$13,698$.
>
> **The Lesson**: The corporate bond pays $\approx 3x$ the interest for that single overnight hold. Trading systems that mismatch these conventions can generate massive "mystery P&L" breaks overnight.

> **Desk Reality: The Day Count Hedge Mismatch**
>
> Consider hedging a $100mm 5% corporate bond (30/360) with Treasury futures (ACT/ACT). Over a quarter with 92 actual days (say, January 1 to April 2):
>
> - **Corporate accrual (30/360):** $\frac{91}{360} \times 5\% \times \$100mm = \$126,389$
>   (Jan: 30, Feb: 30, Mar: 30, Apr 1: 1 = 91 days)
> - **Treasury accrual (ACT/ACT):** $\frac{92}{365} \times 5\% \times \$100mm = \$126,027$
>
> The mismatch of $362 may seem small, but it's systematic. Over a year, it compounds. Worse, around February, the mismatch explodes—creating "day count basis" P&L that your risk system may not flag.
>
> **The fix:** Acknowledge the mismatch in your hedge accounting and true up periodically. Or use matched instruments (corporate CDS vs corporate bond, not Treasury hedges).

### 1.2.6 Leap Year Edge Cases

Leap years create additional complexity:

> **Practitioner Note: February 29 and Day Count Anomalies**
>
> In leap years, February 29 adds one extra day under ACT conventions but is ignored under 30/360. From February 28 to March 1 in a leap year:
> - **ACT/ACT:** 2 days (Feb 28 → Feb 29 → Mar 1)
> - **30/360:** 3 days (same as non-leap year)
>
> This creates a brief window where ACT conventions *under-accrue* relative to 30/360 (2 vs 3 days), partially offsetting the usual February anomaly. Some arbitrage desks track these calendar effects explicitly.

### 1.2.7 Day Count as a "Unit System"

Think of day counts like measurement systems. Quoting a rate as "5%" without specifying the day count is like saying a distance is "100" without specifying whether it's meters or feet.

The same economic arrangement can have different quoted rates depending on the day count. Consider a loan from February 15, 2001, to August 15, 2001 (181 actual days):

- At 5% under Actual/360: $\text{Interest} = 5\% \times \frac{181}{360} = 2.5139\%$
- At 5% under semiannual compounding: $\text{Interest} = \frac{5\%}{2} = 2.500\%$
- At 5% under monthly compounding: $\text{Interest} = (1 + 0.05/12)^6 - 1 = 2.5262\%$
- At 5% under daily compounding: $\text{Interest} = (1 + 0.05/365)^{181} - 1 = 2.5103\%$

Different numbers, same economic reality. Tuckman emphasizes: "compounding conventions do not really matter so long as cash flows are properly computed." The danger is when conventions get mixed up during computation.

> **Worked Example C: Same Period, Three Conventions**
>
> Calculate accrued interest from March 15 to June 15 (92 actual days, in a coupon period of 182 days) on $100mm face of a 6% coupon bond.
>
> | Convention | Day Fraction | Accrued Interest |
> |------------|--------------|------------------|
> | ACT/ACT | $\frac{92}{182} = 0.5055$ | $\$100mm \times 3\% \times 0.5055 = \$1,516,484$ |
> | 30/360 | $\frac{90}{180} = 0.5000$ | $\$100mm \times 3\% \times 0.5000 = \$1,500,000$ |
> | ACT/360 | $\frac{92}{180} = 0.5111$ | $\$100mm \times 3\% \times 0.5111 = \$1,533,333$ |
>
> **Difference:** The gap between highest and lowest is $33,333—material for trade confirmation and P&L.

---

## 1.3 Compounding: Another Unit Choice

### 1.3.1 What Compounding Means

Compounding frequency determines how often interest is calculated and added to principal. Hull explains that with compounding $m$ times per year, an investment of $A$ at annual rate $R$ grows to:

$$\boxed{A\left(1 + \frac{R}{m}\right)^{mn}}$$

after $n$ years.

The effect of compounding frequency is shown in Hull's Table 4.1:

| Compounding Frequency | Value of $100 at end of 1 year (10% rate) |
|----------------------|------------------------------------------|
| Annually ($m=1$) | $110.00 |
| Semiannually ($m=2$) | $110.25 |
| Quarterly ($m=4$) | $110.38 |
| Monthly ($m=12$) | $110.47 |
| Weekly ($m=52$) | $110.51 |
| Daily ($m=365$) | $110.52 |
| Continuous | $110.52 |

### 1.3.2 Continuous Compounding

**Continuous compounding** is the limit as $m \to \infty$. Hull explains that "with continuous compounding, it can be shown that an amount $A$ invested for $n$ years at rate $R$ grows to":

$$\boxed{Ae^{Rn}}$$

And discounting uses the inverse:

$$\boxed{e^{-Rn}}$$

Hull notes that "for most practical purposes, continuous compounding can be thought of as being equivalent to daily compounding."

> **Why Continuous Compounding in Derivatives:** Hull states: "In this book, interest rates will be measured with continuous compounding except where stated otherwise... continuously compounded interest rates are used to such a great extent in pricing derivatives that it makes sense to get used to working with them now." The mathematical convenience—particularly that discount factors multiply simply as $e^{-r_1 t_1} \times e^{-r_2 t_2} = e^{-(r_1 t_1 + r_2 t_2)}$—makes continuous compounding the standard in quantitative finance.

### 1.3.3 Compounding Is a Unit Choice, Not an Economic Choice

Tuckman makes a key point: "There can be only one market-clearing interest payment for money from February 15, 2001, to August 15, 2001." The actual dollars exchanged are fixed by supply and demand. What differs is how we *express* that payment as a rate.

A 5% rate with semiannual compounding and a 4.939% rate with continuous compounding describe the **same** economic outcome over a six-month period:

- Semiannual: $1 \times (1 + 0.05/2) = 1.025$
- Continuous: $1 \times e^{0.04939 \times 0.5} = 1.025$

Tuckman states: "compounding conventions must be understood in order to determine cash flows. But with respect to valuation, compounding conventions do not matter: The market-clearing prices for cash flows on particular dates are the fundamental quantities."

### 1.3.4 Conversion Formulas

When converting between compounding conventions, Hull provides the formulas. To convert from $m$-times-per-year compounding at rate $R_m$ to continuous at rate $R_c$:

$$\boxed{R_c = m \ln\left(1 + \frac{R_m}{m}\right)}$$

And the reverse:

$$\boxed{R_m = m\left(e^{R_c/m} - 1\right)}$$

More generally, to convert from compounding $m_1$ times per year at rate $R_1$ to compounding $m_2$ times per year at rate $R_2$:

$$R_2 = m_2\left[\left(1 + \frac{R_1}{m_1}\right)^{m_1/m_2} - 1\right]$$

> **Source:** Hull provides these conversion formulas explicitly and notes that continuous compounding "can be regarded as the limit as the compounding period becomes infinitely small." Actuaries sometimes refer to a continuously compounded rate as the "force of interest."

### 1.3.5 Convention Risk

The practical danger is **convention risk**: accidentally using the wrong compounding assumption. A risk system that assumes continuous compounding when the market quotes semiannual will systematically misprice by a small but consistent amount—easily enough to cause P&L breaks.

> **Desk Reality: Compounding Mismatch in Systems**
>
> Consider a $500mm swap portfolio where your risk system assumes continuous compounding but the market convention is semiannual. The "error" at 5% rates:
>
> $$\text{Semiannual: } 5.00\% \quad \text{vs} \quad \text{Continuous: } 2 \ln(1.025) = 4.939\%$$
>
> The 6.1bp difference, applied to DV01 calculations across the portfolio, can produce systematic P&L attribution errors of tens of thousands of dollars monthly. Worse, the error is *consistent*—always in the same direction—so it doesn't average out.

**Worked Example D:** Convert 10% semiannual to continuous (Hull Example 4.1).

$$R_c = 2 \ln(1 + 0.10/2) = 2 \ln(1.05) = 2 \times 0.04879 = 0.09758 = 9.758\%$$

The continuous rate is lower because continuous compounding "works harder"—interest compounds more frequently.

**Worked Example E:** Convert 8% continuous to quarterly (Hull Example 4.2).

$$R_m = 4 \times (e^{0.08/4} - 1) = 4 \times (e^{0.02} - 1) = 4 \times 0.0202 = 0.0808 = 8.08\%$$

This means on a $1,000 loan with interest paid quarterly, each payment would be $\$1,000 \times 0.0808/4 = \$20.20$.

**Worked Example F:** Convert 6% semiannual to quarterly.

$$R_2 = 4\left[\left(1 + \frac{0.06}{2}\right)^{2/4} - 1\right] = 4\left[(1.03)^{0.5} - 1\right] = 4 \times 0.01489 = 0.0596 = 5.96\%$$

---

## 1.4 Settlement, Calendars, and Timing

### 1.4.1 Trade Date vs Settlement Date

When you trade a bond, cash does not move immediately. The **settlement date** is when the buyer pays and the seller delivers. The gap between trade date and settlement is the **settlement lag**.

Tuckman explains: "An investor purchasing a Treasury bond on a particular date must usually pay for the bond on the following business day. Similarly, the investor selling the bond on that date must usually deliver the bond on the following business day. The practice of delivery or settlement one day after a transaction is known as T+1 settle."

| Instrument | Typical Settlement |
|------------|-------------------|
| U.S. Treasuries | T+1 |
| U.S. Corporate bonds | T+2 |
| Eurobonds | T+2 or T+3 |
| USD Term Deposits/Swaps | T+2 |
| Money market deposits | T+2 (spot) |

O'Kane confirms: "An interest rate swap traded at time 0 settles at time $t_s$. This typically occurs two days later, i.e. the settlement convention is known as 'T+2'."

**Trends in Settlement Lag:**
Settlement times are shrinking globally to reduce counterparty risk.

*   **Analogy:** Settlement lag is like the time between swiping your credit card and the money actually leaving your bank. In markets, this gap creates risk: if your counterparty goes bankrupt during the lag (after trade but before cash moves), you have a problem.
*   **The Shift to T+1:** In May 2024, U.S. Equities moved to T+1 settlement. U.S. Treasuries have long been T+1. The goal is to minimize the "pending" exposure in the system.

### 1.4.2 What Happens on Settlement Day

O'Kane describes the settlement mechanics for bonds: "On the settlement date, the purchaser of the bond pays the full bond price to the seller. The full bond price is determined by adding the clean price of the bond, which is how the bond price is quoted in the market, and the accrued interest calculated according to some accrual convention."

Settlement matters for two key reasons:

1. **Accrued interest is computed to the settlement date, not the trade date.** If you trade on Friday but settle on Monday, the seller gets three extra days of accrued interest.

2. **Discount factors are computed from the settlement date.** When pricing a bond, the "today" in your present value calculation is settlement, not trade date.

### 1.4.3 When Settlement Fails

What happens when the seller cannot deliver the bonds on settlement day? This creates a **settlement fail**.

Tuckman explains the economics: "Consider a trader who is short the OTR 10-year and needs to borrow it through a repurchase agreement. If for some reason the bond cannot be borrowed, the trader will fail to deliver it and, consequently, not receive the proceeds from the sale. In effect, the trader will lose one day of interest on the proceeds."

This creates a natural floor for special repo rates: "if the repo rate is 0%, there is no point in bothering with the repo agreement: Earning 0% on the proceeds is the equivalent of having failed to deliver the bond. And certainly the trader will prefer to fail rather than accept a special rate less than 0%. Therefore, the special rate cannot fall below 0%, and, equivalently, the special spread cannot be greater than the GC rate."

> **Practitioner Note: The Fails Charge Mechanism**
>
> After the 2008 financial crisis, when repo rates approached zero and fails spiked, the Treasury Market Practices Group (TMPG) introduced a **fails charge** to discourage chronic fails. The mechanism works as a penalty:
>
> - If a party fails to deliver Treasury securities, they owe a **fails charge** to the counterparty
> - The charge is calculated as: $\max(0, 3\% - \text{Fed Funds Target}) \times \text{Face Value} \times \frac{\text{Days Failed}}{360}$
> - When the Fed Funds target is below 3%, there's a meaningful penalty for failing
> - When rates are above 3%, the opportunity cost of not having proceeds is penalty enough
>
> **Example:** Failing to deliver $100mm for 3 days when Fed Funds target is 0.25%:
> $$\text{Fails Charge} = (3\% - 0.25\%) \times \$100mm \times \frac{3}{360} = \$22,917$$
>
> **Why this matters to middle office:** Settlement fails appear as exceptions in your daily processing. Understanding the economic penalty helps you triage which fails need urgent escalation (large notional, scarce collateral) versus routine cleanup.

Tuckman documents a real example: after September 11, 2001, "the terrorist attack on the World Trade Center disrupted the specials market in two ways. First, the resulting confusion and the destruction of records caused many government bond transactions to fail... Second, heightened uncertainty and credit concerns caused many participants in the repo markets to pull their securities from the repo market. The combination of these forces caused a severe shortage of on-the-run collateral."

### 1.4.4 Business Day Conventions

What happens when a payment date falls on a weekend or holiday? Markets use **business day adjustment rules**.

**Following:** Move to the next business day.

**Modified Following:** Move to the next business day, *unless* that would push into the next calendar month—in which case, move to the previous business day instead.

**Preceding:** Move to the previous business day.

**Modified Preceding:** Move to the previous business day, unless that would push into the previous calendar month—in which case, move forward.

Hull explains: "Another business day convention that is sometimes specified is the modified following business day convention, which is the same as the following business day convention except that when the next business day falls in a different month from the specified day, the payment is made on the immediately preceding business day."

Modified Following is the most common for swaps because it keeps payments within the same month for accounting purposes, while minimizing the delay.

> **Source:** Hull's Business Snapshot 7.1 shows a swap confirmation specifying "Following business day" convention with the "U.S." holiday calendar, meaning "if a payment date falls on a weekend or a U.S. holiday, the payment is made on the next business day."

### 1.4.5 Holiday Calendars

Different markets use different holiday calendars:

| Currency/Market | Holiday Calendar |
|-----------------|------------------|
| USD | U.S. (Federal Reserve holidays) |
| EUR | TARGET (Trans-European Automated Real-time Gross settlement) |
| GBP | London |
| JPY | Tokyo |

For cross-currency products, payment dates typically must be business days in *all* relevant calendars.

Hull's glossary confirms: "Holiday Calendar: Calendar defining which days are holidays for the purposes of determining payment dates in a swap."

> **Practitioner Note: Holiday Risk and "Stranded Liquidity"**
>
> Cross-currency trades create **calendar mismatch risk**. If you have a USD/EUR cross-currency swap:
> - July 4 is a U.S. holiday but not a TARGET holiday
> - December 26 is a TARGET holiday but often not a U.S. holiday
>
> When one market is open and the other closed, you may have:
> - **Funding mismatches:** You owe EUR but can't receive USD to cover
> - **Hedging gaps:** Your USD hedge isn't trading, but EUR rates are moving
> - **Operational bottlenecks:** Half your counterparties are offline
>
> Most confirmations specify that payment dates must be business days in *both* relevant calendars. But fixing dates, margin calls, and other operational dates can still create friction. Large trading desks maintain explicit "holiday risk" reports for periods like Christmas week and Golden Week (Japan).

### 1.4.6 Fixing Dates vs Payment Dates

For **term** floating-rate instruments (like legacy LIBOR or Term SOFR), the **fixing date** (when the rate is observed) typically precedes the **accrual start date** by 2 business days. This gives operational time to calculate the payment.

O'Kane explains: "In the standard swap contract, each floating rate coupon is set in advance, and paid in arrears meaning that the value of the next coupon payment is determined by observing the appropriate term Libor on the fixing date which typically falls two days before the immediately preceding coupon payment date."

A typical vanilla term swap timeline for one period:

```
Fixing Date (T-2 business days) → Accrual Start → Accrual End → Payment Date
                                   |_____________Accrual Period_____________|
```

The payment is calculated based on the rate observed on the fixing date, but the cash moves on the payment date.

### 1.4.7 Risk-Free Rate (RFR) Compounding Mechanics

Modern markets have transitioned from forward-looking term rates (like LIBOR) to backward-looking overnight Risk-Free Rates (RFRs) like SOFR (USD), SONIA (GBP), and €STR (EUR). This creates operational complexity because the applicable rate is not known until the accrual period ends.

Hull explains: "The new reference rates are backward looking. The rate applicable to a particular period is not known until the end of the period when all the relevant overnight rates have been observed."

The compounding formula for RFRs is:

$$\boxed{\text{Compounded Rate} = \left[\prod_{i=1}^{n}(1 + r_i \hat{d}_i) - 1\right] \times \frac{360}{D}}$$

where $r_i$ is the overnight rate on day $i$, $\hat{d}_i = d_i/360$, $d_i$ is the number of days that rate applies (usually 1, but 3 over weekends), and $D = \sum_{i} d_i$ is the total number of days in the period.

> **Worked Example G: SOFR Compounding**
>
> Calculate the compounded SOFR rate for a 5-day period with the following daily rates:
>
> | Day | SOFR Rate | Days Applied |
> |-----|-----------|--------------|
> | Mon | 5.30% | 1 |
> | Tue | 5.32% | 1 |
> | Wed | 5.31% | 1 |
> | Thu | 5.29% | 1 |
> | Fri | 5.30% | 3 (includes Sat/Sun) |
>
> **Step 1:** Calculate daily growth factors:
> - Mon: $1 + 0.0530 \times \frac{1}{360} = 1.0001472$
> - Tue: $1 + 0.0532 \times \frac{1}{360} = 1.0001478$
> - Wed: $1 + 0.0531 \times \frac{1}{360} = 1.0001475$
> - Thu: $1 + 0.0529 \times \frac{1}{360} = 1.0001469$
> - Fri: $1 + 0.0530 \times \frac{3}{360} = 1.0004417$
>
> **Step 2:** Compound:
> $$\prod = 1.0001472 \times 1.0001478 \times 1.0001475 \times 1.0001469 \times 1.0004417 = 1.001031$$
>
> **Step 3:** Annualize:
> $$\text{Compounded Rate} = (1.001031 - 1) \times \frac{360}{7} = 5.303\%$$

> **Practitioner Note: Lookback vs Observation Shift vs Lockout**
>
> The challenge with backward-looking rates is that you don't know the final payment until the period ends—leaving no time for operational processing. Markets have developed three main solutions:
>
> | Convention | Mechanism | Trade-off |
> |------------|-----------|-----------|
> | **Payment Delay** | Payment occurs 2-5 days after period end | Simplest; slight timing mismatch |
> | **Lookback** | Observation period shifted back (e.g., by 5 days) | Payment known in advance; hedging mismatch |
> | **Lockout** | Rate frozen for last few days of period | Payment known in advance; rate approximation |
>
> **Example:** For a 3-month period ending June 30:
> - **Payment Delay (2 days):** Pay July 2, using actual SOFR through June 30
> - **Lookback (5 days):** Pay June 30, using SOFR from ~March 26 to ~June 25
> - **Lockout (2 days):** Pay June 30, using SOFR through June 28, then freeze
>
> The lookback approach is common for SOFR-linked loans; lockout is common for floating rate notes. Different conventions create small basis risks when hedging across instruments. Chapter 18 covers these mechanics in depth.

### 1.4.8 Stub Periods

When a swap or bond's effective date doesn't align with the regular payment schedule, the first or last period may be shorter or longer than standard. This creates a **stub period**.

**Front stub:** First period is irregular (shorter or longer than standard).
**Back stub:** Last period is irregular.

O'Kane notes that in a CDS premium leg schedule, "the cash flows are not all equal amounts, reflecting the small but real differences in the time between payments caused by adjustments to avoid weekends and public holidays. There is also a short stub at the beginning reflecting the shorter period to the first premium payment."

Stubs matter because the **accrual fraction** for a stub period differs from regular periods, which changes the actual cash payment. O'Kane provides an example where "the first accrual fraction is 0.161111" for a short stub period—roughly 58 days out of 360—compared to the standard quarterly fraction of 0.25.

> **Worked Example H: Stub Period Accrual**
>
> A CDS contract trades on October 15 with quarterly payment dates on the 20th of each month (Dec, Mar, Jun, Sep). The first payment date is December 20.
>
> **Front stub calculation (ACT/360):**
> - Days from Oct 15 to Dec 20 = 66 days
> - Accrual fraction = $\frac{66}{360} = 0.1833$
> - If spread = 100bp on $10mm notional: $10mm \times 0.01 \times 0.1833 = \$18,333$
>
> Compare to regular quarterly payment:
> - Accrual fraction = $\frac{90}{360} = 0.25$
> - Payment = $10mm \times 0.01 \times 0.25 = \$25,000$
>
> **Sanity check:** Stub payment ≈ 73% of regular payment, matching the ratio of days (66/90 = 73%). ✓

**Visualizing a Long First Coupon:**

```
Timeline: Long First Stub
|---------|-----------------|-----------------|
Issue     First Payment     Second Payment
Nov 15    Mar 15 (4 months) Jun 15 (3 months)
          ↑ Long stub       ↑ Regular
```

*In a "Long First Stub," the first payment covers more than one full period (e.g., 4 months instead of 3). The payment will be $\approx 1.33 \times$ a normal coupon.*

### 1.4.9 End-of-Month Rules

The **End-of-Month (EOM) rule** addresses what happens when a swap or bond starts on the last day of a month. Without an EOM rule, a swap starting January 31 would have a payment "February 31" which doesn't exist.

**Standard EOM rule for swaps:** If the start date is the last business day of the month, all future payment dates are also the last business day of their respective months.

| Start Date | Without EOM | With EOM |
|------------|-------------|----------|
| Jan 31 | Feb 28, Mar 31, Apr 30... | Feb 28, Mar 31, Apr 30... |
| Feb 28 (non-leap) | Mar 28, Apr 28, May 28... | Mar 31, Apr 30, May 31... |

> **Practitioner Note:** EOM rules can create surprisingly large differences in accrual periods. A swap starting February 28 with EOM rule will have a first period of either 28 or 31 days depending on whether March is "28" or "31" under EOM. This affects the first payment by roughly 10%—material on large notionals.

---

## 1.5 Clean vs Dirty: Avoiding Mechanical Price Jumps (Preview)

### 1.5.1 The Problem with Quoting Full Price

Imagine a bond pays a $5 coupon every six months. If markets quoted the **dirty** (full) price, here's what would happen:

- Day 1 after coupon: dirty price = 100 (say)
- Day 90: dirty price = 102.50 (same "underlying" value, but accrued interest has built up)
- Coupon date: dirty price jumps down by ~$5 when the coupon is paid

This sawtooth pattern would make it hard to compare prices across dates or to distinguish real price moves from mechanical accrual.

### 1.5.2 The Clean/Dirty Convention

Markets solve this by quoting the **clean** (flat) price and separately tracking **accrued interest**:

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + \text{AI}}$$

Where:
- $P_{\text{dirty}}$ is the actual cash exchanged per 100 face (the invoice price)
- $P_{\text{clean}}$ is the quoted price
- $\text{AI}$ is accrued interest

O'Kane explains the rationale: "The purpose of accrued interest is to avoid a sudden drop in the quoted price of bonds as we pass through a coupon payment which is what must happen with the bond full price. These effects are not helpful to investors as they do not represent a change in credit quality or interest rates. The market therefore prefers to quote bonds using a clean price."

### 1.5.3 The Continuity Proof

Tuckman provides a rigorous demonstration that clean prices are continuous across coupon dates when yields don't change. The argument proceeds as follows:

**Setup:** Let $P^b$ and $P^a$ be the quoted (clean) prices of a bond right before and right after a coupon payment of $c/2$, respectively. The fundamental pricing equation is:

$$P + AI = PV(\text{future cash flows})$$

**Before the coupon payment:** Right before a coupon date, accrued interest equals the full coupon ($c/2$), and the present value of the next coupon (which is about to be paid) also equals $c/2$. Therefore:

$$P^b + \frac{c}{2} = \frac{c}{2} + PV(\text{cash flows after the next coupon})$$

Simplifying:
$$P^b = PV(\text{cash flows after the next coupon})$$

**After the coupon payment:** Right after the coupon payment, accrued interest equals zero:

$$P^a + 0 = PV(\text{cash flows after the next coupon})$$

**Conclusion:** Since both expressions equal the same present value, $P^a = P^b$. The clean price is continuous across the coupon date.

By contrast, the dirty price falls from $(P^b + c/2)$ before the coupon to $P^a = P^b$ after the coupon—a drop of exactly $c/2$.

> **Source:** Tuckman states: "With an accrued interest convention, if yield does not change then the quoted price of a bond does not fall as a result of a coupon payment."

### 1.5.4 Why This Matters for P&L

When marking a bond position to market:

- If you track **clean price**, your P&L reflects genuine market moves. Coupon payments appear as separate coupon income.
- If you track **dirty price**, your P&L includes mechanical accrual drift, which must be separated from actual market moves.

Most trading desks quote clean and track accrued interest separately, then reconcile to the full settlement amount.

> **Desk Reality: P&L Reporting and Clean/Dirty**
>
> Trading systems typically separate P&L into:
> - **Mark-to-market P&L:** Change in clean price × position
> - **Carry/Accrual:** Interest income (accrued or received)
> - **Funding cost:** Cost of financing the position
>
> If your system reports dirty price P&L, you'll see a steady "gain" between coupon dates (accrual building) and a "loss" on coupon date (accrual resets). This is mechanical, not a trading signal. Traders use clean price for actual market direction.

> **Stock vs Bond Convention:** Tuckman notes an important difference: "The behavior of quoted bond prices differs from that of stocks that do not have an accrued dividend convention. Stock prices fall by approximately the amount of the dividend on the day ownership of the dividend payment is established. The accrued convention does make more sense in bond markets than in stock markets because dividend payment amounts are generally much less certain than coupon payments."

> **Worked Example I: Clean/Dirty Calculation**
>
> A 5% semiannual Treasury bond trades at a clean price of 98-16 (= 98.50). Settlement is July 15. The last coupon was May 15. The next coupon is November 15.
>
> **Step 1:** Calculate days accrued
> - May 15 to July 15 = 61 days
> - May 15 to November 15 = 184 days (full period)
>
> **Step 2:** Calculate accrued interest (ACT/ACT)
> $$AI = \frac{61}{184} \times \frac{5\%}{2} \times 100 = \frac{61}{184} \times 2.50 = 0.829$$
>
> **Step 3:** Calculate dirty price
> $$P_{\text{dirty}} = 98.50 + 0.829 = 99.329$$
>
> **Step 4:** Cash exchanged on $1mm face
> $$\text{Cash} = \$1,000,000 \times \frac{99.329}{100} = \$993,290$$
>
> **Sanity check:** Dirty price > clean price (positive accrued). Accrued ≈ 1/3 of semiannual coupon (61/184 ≈ 33%). ✓

**Full treatment of accrued interest mechanics appears in Chapter 5.**

---

## 1.6 Practical Sanity Checks

Every practitioner should run these checks on any fixed-income calculation:

### 1.6.1 Timing Checks

- **Fixing date before accrual start:** For term rates, the rate must be known before accrual begins.
- **Payment date at or after accrual end:** You can't pay before the period is complete.
- **Settlement vs effective:** Know which date your valuation uses.
- **Holiday consistency:** Ensure all dates respect the appropriate calendar(s).
- **Stub period proportionality:** Stub accrual should be roughly proportional to stub length.

### 1.6.2 Accrued Interest Checks

- $0 \leq \text{AI} \leq \text{Coupon}$ between payment dates
- AI resets to 0 immediately after coupon payment
- AI approaches full coupon immediately before payment
- AI computed to settlement date, not trade date

### 1.6.3 Monotonicity Check (Positive Rates)

If rates are positive, discount factors should decrease with maturity. A discount factor curve that increases with maturity (except in negative-rate environments) suggests a bug.

### 1.6.4 Convention Consistency

- Day count used for accrued interest should match the instrument convention
- Compounding assumption should match the market's convention
- Settlement lag should match the market standard
- Business day convention should match the contract specification

### 1.6.5 Invoice Price Sanity

Tuckman notes that "the amount paid or received for a bond (i.e., its full price) equals the present value of its cash flows." So:

$$P_{\text{dirty}} \approx PV(\text{remaining cashflows})$$

This provides a sanity check: if your computed dirty price differs materially from the sum of discounted cashflows, something is wrong.

---

## 1.7 Common Implementation Bugs

| Bug | Description | Consequence |
|-----|-------------|-------------|
| **Off-by-one in schedule** | Rolling some dates but not others, or miscounting periods | Wrong number of cashflows, wrong accrual |
| **Day count mismatch** | Using ACT/360 when ACT/ACT is correct (or vice versa) | Systematic accrual error |
| **Clean/dirty confusion** | Treating clean price as dirty, or adding accrued twice | Wrong settlement amount |
| **Ignoring stubs** | Assuming all periods have equal accrual fraction | Wrong first/last cashflows |
| **Settlement lag error** | Computing accrued to trade date instead of settlement | Wrong accrued interest |
| **Calendar mismatch** | Using wrong holiday calendar for payment date adjustment | Payments on wrong dates |
| **Compounding mismatch** | Converting rates with wrong frequency assumption | Systematic pricing error |
| **End-of-month error** | Incorrect handling of dates near month-end in 30/360 | Wrong day count |
| **Timezone error** | Using local time vs UTC for cross-border trades | Dates off by one day |
| **Rounding differences** | Different rounding in confirmation vs settlement system | Small but persistent breaks |

---

## Summary

This chapter established the "plumbing" that underlies all fixed-income pricing:

1. **Markets quote what's useful for their participants**: Bonds quote clean prices in 32nds; T-bills quote discount rates; money markets quote rates; swaps quote fixed rates. The quote alone isn't enough—you need the conventions.

2. **Day counts are a unit system**: ACT/ACT, 30/360, and ACT/360 measure time differently. Using the wrong convention means computing the wrong interest. The same one-day period can count as 1 day (ACT) or 3 days (30/360) depending on convention.

3. **Compounding is also a unit choice**: A 5% semiannual rate is not the same as 5% continuous. Conversion formulas exist, but mixing conventions creates errors. Continuous compounding is standard in derivatives pricing for its mathematical convenience.

4. **Settlement timing matters**: Cash moves on settlement date, not trade date. Accrued interest is computed to settlement. Different markets have different settlement lags (T+1 for Treasuries, T+2 for swaps). When settlement fails, there are economic consequences including fails charges.

5. **RFR mechanics are backward-looking**: Unlike term rates that are fixed in advance, overnight RFRs compound daily and aren't known until period end. Lookback, lockout, and payment delay conventions address operational challenges.

6. **Clean pricing removes mechanical drift**: Markets quote clean to make prices comparable across dates. The dirty price is what actually changes hands. Tuckman proves that clean prices are continuous across coupon dates when yields are unchanged.

None of this is mathematically deep, but all of it must be exactly right. The chapters that follow will build on these conventions: Chapter 2 develops discount factors and present value; Chapter 3 covers the zero-forward-par rate relationships; Chapter 5 fully develops bond pricing with accrued interest mechanics; Chapter 18 covers RFR curve construction in depth.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Clean vs dirty | Dirty = clean + accrued interest | Clean is quoted; dirty is exchanged |
| Day count (X/Y) | Fraction = (days between) / (days in reference period) | Different conventions → different dollars |
| 30/360 | Assume 30 days/month, 360 days/year | Used for corporates; smooths calendar irregularity |
| ACT/ACT | Actual days / actual days in period | Used for Treasuries; exact calendar counting |
| ACT/360 | Actual days / 360 | Used for money markets; inflates annual return |
| Compounding | Frequency of interest computation | Unit of rate quotes; affects conversions |
| Continuous compounding | Limit as frequency → ∞ | Standard in derivatives pricing |
| Settlement | Date when cash moves | Accrued computed to settlement |
| Settlement fail | Failure to deliver on settlement date | Creates fails charge penalty |
| Business day roll | What to do when date is a holiday | Modified Following is common for swaps |
| Stub | Irregular first/last period | Affects actual cash payment amount |
| Fixing date | When floating rate is observed | Typically T-2 for term rates |
| RFR compounding | Daily rate compounded over period | Backward-looking; not known until period end |

---

## Notation for This Book

The following notation will be used throughout subsequent chapters:

| Symbol | Definition |
|--------|------------|
| $P_{\text{clean}}$ or $P$ | Clean/quoted/flat price (per 100 par) |
| $P_{\text{dirty}}$ or $P_{\text{full}}$ | Dirty/invoice/full price (per 100 par) |
| $\text{AI}$ | Accrued interest (per 100 par) |
| $P(0,T)$ or $d(T)$ | Discount factor: PV of $1 at time $T$ |
| $c$ | Annual coupon rate (as decimal) |
| $m$ | Coupon frequency per year |
| $\tau$, $\alpha$, or $\Delta$ | Year fraction / accrual fraction |
| $z(T)$ or $r(T)$ | Zero (spot) rate to maturity $T$ |
| $f(T_1, T_2)$ | Forward rate for period $[T_1, T_2]$ |
| $R_c$ | Continuously compounded rate |
| $R_m$ | Rate compounded $m$ times per year |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the difference between clean and dirty price? | Clean is quoted; dirty = clean + accrued interest is exchanged |
| 2 | Why do bond markets quote clean prices? | To remove mechanical accrual drift and make prices comparable across dates |
| 3 | What does the day count convention determine? | How to compute the fraction of time between dates for interest calculation |
| 4 | Name the three common U.S. day count conventions | Actual/Actual (Treasuries), 30/360 (corporates), Actual/360 (money markets) |
| 5 | Under Actual/360, a full 365-day year earns how much of the quoted rate? | 365/360 = 1.0139 times the quoted rate |
| 6 | What is Modified Following? | Move to next business day unless that crosses month-end, then move back |
| 7 | What is a stub period? | An irregular first or last accrual period that doesn't match the standard length |
| 8 | What is T+1 settlement? | Cash and securities exchange one business day after trade date |
| 9 | To what date is accrued interest computed? | Settlement date (not trade date) |
| 10 | What is a fixing date? | The date when a floating rate is observed (typically 2 business days before accrual start for term rates) |
| 11 | How are U.S. Treasury prices quoted? | In dollars and 32nds per $100 face value |
| 12 | Convert 100-16 to decimal | 100 + 16/32 = 100.50 |
| 13 | Why does compounding frequency matter? | It's part of the rate's "unit"; different frequencies with same number mean different economics |
| 14 | Formula to convert semiannual rate to continuous | $R_c = 2\ln(1 + R_m/2)$ |
| 15 | What check confirms you haven't added accrued twice? | Dirty price should roughly equal PV of remaining cashflows |
| 16 | How many days between Feb 28 and Mar 1 under 30/360? | 3 days (Feb 28 → Feb 30 → Mar 1) |
| 17 | How many days between Feb 28 and Mar 1 under ACT/ACT? | 1 day |
| 18 | Why is continuous compounding used in derivatives pricing? | Mathematical convenience: discount factors multiply simply |
| 19 | What is the relationship between T-bill quoted price and cash price? | $P = \frac{360}{n}(100 - Y)$ where $P$ is quoted, $Y$ is cash, $n$ is days |
| 20 | What holiday calendar is used for EUR payments? | TARGET (Trans-European Automated Real-time Gross settlement) |
| 21 | What is a settlement fail? | When the seller cannot deliver securities on the agreed settlement date |
| 22 | What is the fails charge mechanism? | A penalty calculated as max(0, 3% - Fed Funds) to discourage chronic settlement fails |
| 23 | How are RFR (overnight) rates compounded over a period? | Daily rates are compounded: $\prod(1 + r_i \times d_i/360) - 1$, annualized |
| 24 | What is the "lookback" convention for RFRs? | Observation period is shifted back (e.g., 5 days) so payment amount is known before payment date |
| 25 | If you hedge a 30/360 corporate bond with an ACT/ACT Treasury, what basis risk exists? | Day count mismatch—accrual differs systematically, especially around February |

---

## Mini Problem Set

### Questions

1. A Treasury is quoted at 98-24. Convert to decimal.

2. Under Actual/Actual, with 60 days elapsed in a 182-day coupon period and a $3 semiannual coupon, what is the accrued interest?

3. A money market deposit of $1,000,000 for 90 days at 4% (Actual/360) earns how much interest?

4. Convert 8% with semiannual compounding to continuous compounding.

5. If you trade on Wednesday and settlement is T+2, when does cash move (assuming no holidays)?

6. A swap's effective date is March 20, but the first regular payment date would be June 20 (a Sunday). Under Modified Following, when is payment?

7. Explain why 30/360 can produce more accrued interest than Actual/Actual for February.

8. A bond's clean price is 102.50 and accrued interest is 1.875. What is the dirty price?

9. A 91-day T-bill is quoted at 5.00 (discount rate). What is the cash price per $100 face value?

10. Convert 6% semiannual to quarterly compounding.

11. Using 30/360, how many days are there from August 27 to November 15?

12. A Treasury bond has coupon dates March 1 and September 1, with an 8% annual coupon. Under ACT/ACT, what is the accrued interest on July 3 per $100 face?

13. A CDS trades on November 10 with quarterly payments on the 20th. If the spread is 150bp on $5mm notional, what is the first (stub) premium payment on December 20? (Use ACT/360)

14. You fail to deliver $50mm of Treasuries for 2 days when the Fed Funds target is 0.50%. Calculate the fails charge.

15. Calculate the compounded rate for a 3-day period where Monday's SOFR is 5.25% (applies 1 day), Tuesday's is 5.28% (applies 1 day), and Wednesday's is 5.26% (applies 3 days including the weekend). Express as an annualized rate.

16. A trader holds $100mm face of 6% corporate bonds (30/360) and hedges with $100mm Treasury futures (ACT/ACT). Over March 1-31 (31 actual days), what is the accrual mismatch in dollars?

### Brief Solutions

1. $98 + 24/32 = 98.75$

2. $\text{AI} = (60/182) \times 3 = 0.989$

3. $1{,}000{,}000 \times 0.04 \times 90/360 = \$10{,}000$

4. $R_c = 2\ln(1.04) = 2 \times 0.0392 = 7.84\%$

5. Friday (Wednesday + 2 business days)

6. Monday, June 21 (roll forward to next business day; stays in same month)

7. 30/360 counts February as having 30 days; Actual/Actual counts 28 (or 29 in leap year). End of February to March 1 is 3 days under 30/360 but only 1 day under Actual/Actual.

8. $102.50 + 1.875 = 104.375$

9. Interest = $100 \times 0.05 \times 91/360 = \$1.2639$; Cash price = $100 - 1.2639 = \$98.7361$

10. $R_2 = 4[(1.03)^{0.5} - 1] = 4 \times 0.01489 = 5.96\%$

11. Aug 27 → Aug 30 = 3 days; Aug 30 → Nov 15 = 2 months + 15 days = 75 days. Total = 78 days.

12. Days from Mar 1 to Jul 3 = 124; Days from Mar 1 to Sep 1 = 184; Semiannual coupon = $4; AI = $(124/184) \times 4 = \$2.6957$

13. Days from Nov 10 to Dec 20 = 40 days. Accrual fraction = 40/360 = 0.1111. Payment = $5mm × 0.015 × 0.1111 = $8,333.

14. Fails charge = $(3\% - 0.5\%) \times \$50mm \times \frac{2}{360} = \$6,944$

15. Compound: $(1 + 0.0525/360)(1 + 0.0528/360)(1 + 0.0526 \times 3/360) - 1 = 0.000731$. Annualized: $0.000731 \times 360/5 = 5.26\%$

16. Corporate (30/360): 30 days, accrual = $100mm × 3% × 30/360 = $250,000. Treasury (ACT/ACT): 31 days in 184-day period, accrual = $100mm × 3% × 31/184 = $50,543. Mismatch ≈ $199,457. (Note: This example assumes same coupon for comparison; actual hedge ratio would differ.)

---

## Source Map

### (A) Book-Verified Facts

| Fact | Source |
|------|--------|
| Treasury prices quoted in 32nds | Tuckman Ch 1, Hull Ch 6 |
| Futures quoted to half/quarter 32nds | Hull Ch 6 |
| T-bill discount rate quotation formula | Hull Ch 6 |
| Dirty = clean + accrued interest | Tuckman Ch 4, Hull Ch 6, O'Kane Ch 4 |
| Day count defines interest accrual (X/Y definition) | Hull Ch 6 |
| Three U.S. day counts: ACT/ACT, 30/360, ACT/360 | Hull Ch 6 |
| 30/360 day counting algorithm | Tuckman Ch 17 |
| Actual/360 money market convention | Tuckman Ch 4, Hull Ch 6 |
| ACT/365 for GBP, ACT/360 for USD/EUR/JPY money markets | Hull Ch 6, O'Kane Ch 2 |
| Clean price continuity across coupon dates (proof) | Tuckman Ch 4 |
| T+1 settlement for Treasuries | Tuckman Ch 1 |
| T+2 settlement for swaps | O'Kane Ch 2, Hull Ch 7 |
| Compounding conversion formulas | Hull Ch 4 |
| Modified Following business day rule | Hull Ch 7, O'Kane Ch 2 |
| February day count anomaly (3 days vs 1 day) | Hull Business Snapshot 6.1 |
| Fixing date 2 days before accrual start for term rates | O'Kane Ch 2 |
| Stub periods in CDS premium leg | O'Kane Ch 5 |
| RFR backward-looking compounding formula | Hull Ch 4 |
| Stock vs bond accrual convention difference | Tuckman Ch 4 footnote |
| Continuous compounding preferred in derivatives | Hull Ch 4 |
| Settlement fails economics (loss of one day's interest) | Tuckman Ch 15 |
| Special repo rate floor at 0% due to fails alternative | Tuckman Ch 15 |
| September 11, 2001 fails disruption | Tuckman Ch 15 |
| Bid-ask spreads particularly low for liquid securities | Tuckman Ch 15 |
| Holiday calendar definition | Hull Glossary |

### (B) Claude-Extended Content (Practitioner Notes)

| Content | Basis |
|---------|-------|
| Fails charge mechanism (3% - Fed Funds formula) | Extends Tuckman Ch 15 fails discussion with TMPG post-2008 practice |
| Day count hedge mismatch P&L example | Extends Hull Business Snapshot 6.1 with hedging application |
| Holiday risk and "stranded liquidity" | Extends Hull Ch 7 calendar discussion with cross-currency operational practice |
| RFR lookback vs observation shift vs lockout | Extends Hull Ch 4 backward-looking discussion with market convention details |
| Compounding mismatch in systems desk reality | Extends Hull Ch 4 compounding with operational risk application |
| Bid-ask spread tick-size economics | Extends Tuckman Ch 15 liquidity discussion with trading desk example |
| P&L reporting clean/dirty separation | General fixed income desk practice |
| EOM rule details for swaps | Standard swap market practice |
| Leap year day count anomalies | Derived from day count definitions |

### (C) Reasoned Inference (Derived from A and B)

- Clean pricing removes sawtooth accrual pattern (follows from the clean/dirty identity and the coupon-date continuity proof)
- Day counts are like measurement units (conceptual framing consistent with Tuckman's statement that conventions "do not really matter so long as cash flows are properly computed")
- Convention risk exists whenever conventions are mixed (implied by the existence of multiple conventions)
- Dirty price ≈ PV of remaining cashflows (follows from Tuckman's pricing equation P + AI = PV)
- Stub payment proportionality check (derived from accrual fraction definition)

### (D) Flagged Uncertainties

- **Exact fails charge details:** The 3% threshold was introduced by TMPG in 2009; specific implementation may vary by market and may have been updated. Readers should verify current TMPG guidance.
- **End-of-month rules:** Exact EOM conventions for 30/360 vary by specific variant (30/360 ISDA, 30E/360, etc.); specific rules require contract documentation.
- **RFR convention details:** Specific lookback, lockout, and payment delay conventions vary by currency, product, and counterparty agreement. The examples given are illustrative; Chapter 18 covers these in more depth.
- **Exact settlement lags:** May vary by market, currency, instrument type, and can change over time (e.g., U.S. equities moved from T+2 to T+1).
- **Business day calendars:** Holiday calendars are jurisdiction-specific and change annually; production systems require maintained calendar data.
