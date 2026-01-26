# Chapter 1: Market Quoting, Calendars, and Cashflow Plumbing

---

## Introduction

Before you can price a bond, value a swap, or compute the risk of a portfolio, you must answer a deceptively simple question: *what exactly are we talking about?*

A trader says a Treasury is "bid at 99-16." A salesperson quotes a swap rate of "3.25%." A risk report shows duration of "7.2 years." Each of these numbers carries implicit assumptions about day counts, compounding, settlement timing, and dozens of other conventions that turn abstract rates into actual dollars. Get any of these conventions wrong, and your "price" becomes meaningless—or worse, you wire the wrong amount on settlement day.

This chapter is about the **plumbing**: the infrastructure of market conventions that translates quotes into cashflows and cashflows into cash exchanged. We cover:

1. **How markets quote instruments** — price vs yield vs spread, and why each market chooses its convention
2. **Day counts and compounding as a unit system** — the "measurement units" of interest rates
3. **Settlement, calendars, and timing** — when cash actually moves
4. **Clean vs dirty pricing** — how bond markets avoid artificial price jumps (a preview; full treatment in Chapter 5)

None of this is glamorous. But every mispriced trade, every reconciliation break, and every "mystery P&L" you will encounter in your career traces back to someone getting the plumbing wrong.

---

## 1.1 What Markets Quote: Price, Yield, and Spread

### The Quote Is a Communication Protocol

Every market has developed conventions for how participants communicate value. These conventions reflect what is *useful* for that market's participants—not what is mathematically fundamental.

**Cash bonds** are typically quoted as a **price** (per 100 face value). This makes sense: the price directly tells you the dollars exchanged per unit of face value.

**Money markets** quote **rates** (annualized). A 3-month deposit rate of 5% tells you the interest earned over that period, scaled to an annual basis.

**Credit default swaps** quote **spreads** (in basis points). The spread is the annual premium the protection buyer pays.

**Interest rate swaps** quote the **fixed rate** that makes the swap have zero value at inception—the par swap rate.

Each quote type serves its market. But here is the critical point: **the quote alone is not enough information to calculate cash**. You also need to know the conventions that interpret that quote.

### Bonds: Clean Price and 32nds

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

### T-Bill Quotations: Discount Rate Convention

Treasury bills use a different quoting convention entirely. Hull explains that T-bill prices "are sometimes quoted using a discount rate. This is the interest earned as a percentage of the final face value rather than as a percentage of the initial price paid."

If a 91-day Treasury bill is quoted at 8, this means the rate of interest earned is 8% of face value per 360 days. For a $100 face value bill:

$$\text{Interest} = \$100 \times 0.08 \times \frac{91}{360} = \$2.0222$$

The cash price is therefore $\$100 - \$2.0222 = \$97.9778$.

The relationship between the quoted discount rate $P$ and the cash price $Y$ for a T-bill with $n$ days remaining is:

$$\boxed{P = \frac{360}{n}(100 - Y)}$$

This "discount yield" convention understates the true rate of return because it divides by face value rather than the actual investment amount. The true rate for the 91-day example is $\frac{2.0222}{97.9778} = 2.064\%$ for 91 days.

### Money Markets: Rates with Day Counts

Money market instruments quote rates, not prices. But a "5% rate" is meaningless without knowing:

1. **Day count convention**: How do we measure the time period?
2. **Compounding convention**: Simple interest? Compound interest?

A 3-month rate (e.g., legacy LIBOR or Term SOFR) of 5% under **Actual/360** with simple interest means that for a loan of $1 over $d$ actual days:

$$\text{Interest} = 1 \times 0.05 \times \frac{d}{360}$$

The denominator is 360 (not 365 or actual days in the year). This is not a mathematical choice—it is a market convention that affects the actual dollars owed.

> **Source:** Tuckman explains that "lending $\$1$ for $d$ days at a rate of $r$ will earn the lender an interest payment of $r \times d/360$ dollars at the end of the $d$ days" under the actual/360 convention.

### Why Conventions Differ Across Markets

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

### Day Counts Define How Time Is Measured

Hull provides the canonical definition: "The day count defines the way in which interest accrues over time." More precisely, a day count convention determines two things:

1. How to count days between two dates (the numerator)
2. How to define the "reference period" (the denominator)

The interest earned between two dates is then:

$$\boxed{\text{Interest} = \frac{\text{Number of days between dates}}{\text{Number of days in reference period}} \times \text{Interest earned in reference period}}$$

This fraction—(days between dates) / (days in reference period)—is the **accrual fraction** or **year fraction**, often denoted $\Delta$ or $\tau$.

> **Source:** Hull states: "The day count convention is usually expressed as $X/Y$. When we are calculating the interest earned between two dates, $X$ defines the way in which the number of days between the two dates is calculated, and $Y$ defines the way in which the total number of days in the reference period is measured."

### The Three Common U.S. Conventions

**Actual/Actual (in period):** Used for U.S. Treasury bonds. Both numerator and denominator use actual calendar days.

- Numerator: actual days between dates
- Denominator: actual days in the coupon period

**Worked Example (Hull):** Consider a Treasury bond with coupon payment dates March 1 and September 1, paying an 8% annual coupon ($4 semiannually). To calculate interest earned between March 1 and July 3:

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

### The 30/360 Day Counting Algorithm

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

$$\text{Days} = 360(Y_2 - Y_1) + 30(M_2 - M_1) + (D_2 - D_1)$$

with adjustments at month boundaries (e.g., if $D_1 = 31$, set $D_1 = 30$).

**Actual/360:** Used for USD money markets. Actual calendar days, but divided by 360.

- Numerator: actual days between dates
- Denominator: 360 (always)

This means a full year of 365 days earns $365/360 \approx 1.0139$ times the quoted rate, not exactly the quoted rate. Hull confirms: "the interest earned in a whole year of 365 days is $365/360$ times the quoted rate."

### Why Day Counts Matter: A Concrete Example

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

### International Day Count Conventions

Day count conventions vary by currency and market. Hull notes that "conventions vary from country to country and from instrument to instrument":

| Market | Day Count |
|--------|-----------|
| U.S. Treasury bonds | Actual/Actual (in period) |
| U.S. Corporate bonds | 30/360 |
| USD money markets | Actual/360 |
| EUR money markets | Actual/360 |
| GBP money markets | Actual/365 |
| EUR/GBP bonds | Actual/Actual |
| Australian/Canadian money markets | Actual/365 |

O'Kane emphasizes that "every time we encounter a day count fraction, we must take care to apply the correct day count basis convention."

### Day Count as a "Unit System"

Think of day counts like measurement systems. Quoting a rate as "5%" without specifying the day count is like saying a distance is "100" without specifying whether it's meters or feet.

The same economic arrangement can have different quoted rates depending on the day count. Consider a loan from February 15, 2001, to August 15, 2001 (181 actual days):

- At 5% under Actual/360: $\text{Interest} = 5\% \times \frac{181}{360} = 2.5139\%$
- At 5% under semiannual compounding: $\text{Interest} = \frac{5\%}{2} = 2.500\%$
- At 5% under monthly compounding: $\text{Interest} = (1 + 0.05/12)^6 - 1 = 2.5262\%$
- At 5% under daily compounding: $\text{Interest} = (1 + 0.05/365)^{181} - 1 = 2.5103\%$

Different numbers, same economic reality. Tuckman emphasizes: "compounding conventions do not really matter so long as cash flows are properly computed." The danger is when conventions get mixed up during computation.

---

## 1.3 Compounding: Another Unit Choice

### What Compounding Means

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

### Continuous Compounding

**Continuous compounding** is the limit as $m \to \infty$. Hull explains that "with continuous compounding, it can be shown that an amount $A$ invested for $n$ years at rate $R$ grows to":

$$\boxed{Ae^{Rn}}$$

And discounting uses the inverse:

$$\boxed{e^{-Rn}}$$

Hull notes that "for most practical purposes, continuous compounding can be thought of as being equivalent to daily compounding."

> **Why Continuous Compounding in Derivatives:** Hull states: "In this book, interest rates will be measured with continuous compounding except where stated otherwise... continuously compounded interest rates are used to such a great extent in pricing derivatives that it makes sense to get used to working with them now." The mathematical convenience—particularly that discount factors multiply simply as $e^{-r_1 t_1} \times e^{-r_2 t_2} = e^{-(r_1 t_1 + r_2 t_2)}$—makes continuous compounding the standard in quantitative finance.

### Compounding Is a Unit Choice, Not an Economic Choice

Tuckman makes a key point: "There can be only one market-clearing interest payment for money from February 15, 2001, to August 15, 2001." The actual dollars exchanged are fixed by supply and demand. What differs is how we *express* that payment as a rate.

A 5% rate with semiannual compounding and a 4.939% rate with continuous compounding describe the **same** economic outcome over a six-month period:

- Semiannual: $1 \times (1 + 0.05/2) = 1.025$
- Continuous: $1 \times e^{0.04939 \times 0.5} = 1.025$

Tuckman states: "compounding conventions must be understood in order to determine cash flows. But with respect to valuation, compounding conventions do not matter: The market-clearing prices for cash flows on particular dates are the fundamental quantities."

### Conversion Formulas

When converting between compounding conventions, Hull provides the formulas. To convert from $m$-times-per-year compounding at rate $R_m$ to continuous at rate $R_c$:

$$\boxed{R_c = m \ln\left(1 + \frac{R_m}{m}\right)}$$

And the reverse:

$$\boxed{R_m = m\left(e^{R_c/m} - 1\right)}$$

More generally, to convert from compounding $m_1$ times per year at rate $R_1$ to compounding $m_2$ times per year at rate $R_2$:

$$R_2 = m_2\left[\left(1 + \frac{R_1}{m_1}\right)^{m_1/m_2} - 1\right]$$

> **Source:** Hull provides these conversion formulas explicitly and notes that continuous compounding "can be regarded as the limit as the compounding period becomes infinitely small." Actuaries sometimes refer to a continuously compounded rate as the "force of interest."

### Convention Risk

The practical danger is **convention risk**: accidentally using the wrong compounding assumption. A risk system that assumes continuous compounding when the market quotes semiannual will systematically misprice by a small but consistent amount—easily enough to cause P&L breaks.

**Worked Example 1:** Convert 10% semiannual to continuous (Hull Example 4.1).

$$R_c = 2 \ln(1 + 0.10/2) = 2 \ln(1.05) = 2 \times 0.04879 = 0.09758 = 9.758\%$$

The continuous rate is lower because continuous compounding "works harder"—interest compounds more frequently.

**Worked Example 2:** Convert 8% continuous to quarterly (Hull Example 4.2).

$$R_m = 4 \times (e^{0.08/4} - 1) = 4 \times (e^{0.02} - 1) = 4 \times 0.0202 = 0.0808 = 8.08\%$$

This means on a $1,000 loan with interest paid quarterly, each payment would be $\$1,000 \times 0.0808/4 = \$20.20$.

**Worked Example 3:** Convert 6% semiannual to quarterly.

$$R_2 = 4\left[\left(1 + \frac{0.06}{2}\right)^{2/4} - 1\right] = 4\left[(1.03)^{0.5} - 1\right] = 4 \times 0.01489 = 0.0596 = 5.96\%$$

---

## 1.4 Settlement, Calendars, and Timing

### Trade Date vs Settlement Date

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

### What Happens on Settlement Day

O'Kane describes the settlement mechanics for bonds: "On the settlement date, the purchaser of the bond pays the full bond price to the seller. The full bond price is determined by adding the clean price of the bond, which is how the bond price is quoted in the market, and the accrued interest calculated according to some accrual convention."

Settlement matters for two key reasons:

1. **Accrued interest is computed to the settlement date, not the trade date.** If you trade on Friday but settle on Monday, the seller gets three extra days of accrued interest.

2. **Discount factors are computed from the settlement date.** When pricing a bond, the "today" in your present value calculation is settlement, not trade date.

### Business Day Conventions

What happens when a payment date falls on a weekend or holiday? Markets use **business day adjustment rules**.

**Following:** Move to the next business day.

**Modified Following:** Move to the next business day, *unless* that would push into the next calendar month—in which case, move to the previous business day instead.

**Preceding:** Move to the previous business day.

**Modified Preceding:** Move to the previous business day, unless that would push into the previous calendar month—in which case, move forward.

Hull explains: "Another business day convention that is sometimes specified is the modified following business day convention, which is the same as the following business day convention except that when the next business day falls in a different month from the specified day, the payment is made on the immediately preceding business day."

Modified Following is the most common for swaps because it keeps payments within the same month for accounting purposes, while minimizing the delay.

> **Source:** Hull's Business Snapshot 7.1 shows a swap confirmation specifying "Following business day" convention with the "U.S." holiday calendar, meaning "if a payment date falls on a weekend or a U.S. holiday, the payment is made on the next business day."

### Holiday Calendars

Different markets use different holiday calendars:

| Currency/Market | Holiday Calendar |
|-----------------|------------------|
| USD | U.S. (Federal Reserve holidays) |
| EUR | TARGET (Trans-European Automated Real-time Gross settlement) |
| GBP | London |
| JPY | Tokyo |

For cross-currency products, payment dates typically must be business days in *all* relevant calendars.

### Fixing Dates vs Payment Dates

For **term** floating-rate instruments (like legacy LIBOR or Term SOFR), the **fixing date** (when the rate is observed) typically precedes the **accrual start date** by 2 business days. This gives operational time to calculate the payment.

O'Kane explains: "In the standard swap contract, each floating rate coupon is set in advance, and paid in arrears meaning that the value of the next coupon payment is determined by observing the appropriate term Libor on the fixing date which typically falls two days before the immediately preceding coupon payment date."

A typical vanilla term swap timeline for one period:

```
Fixing Date (T-2 business days) → Accrual Start → Accrual End → Payment Date
                                   |_____________Accrual Period_____________|
```

The payment is calculated based on the rate observed on the fixing date, but the cash moves on the payment date.

> **Expert Note on RFRs:** In modern Over-the-Counter markets using Overnight Risk-Free Rates (like SOFR and SONIA), the timeline differs. Hull notes that the new reference rates "are backward looking. The rate applicable to a particular period is not known until the end of the period when all the relevant overnight rates have been observed."
>
> Interest is typically calculated daily *during* the accrual period ("in arrears"), and the compounded rate is computed as:
> $$\left[\prod_{i=1}^{n}(1 + r_i \hat{d}_i) - 1\right] \times \frac{360}{D}$$
> where $r_i$ is the overnight rate on day $i$, $\hat{d}_i = d_i/360$, and $D$ is total days in the period.
>
> Various mechanisms (lookback, payment delay, lockout) address the operational challenge that the final payment amount is not known until the end of the period. This is covered in detail in **Chapter 18**.

### Stub Periods

When a swap or bond's effective date doesn't align with the regular payment schedule, the first or last period may be shorter or longer than standard. This creates a **stub period**.

**Front stub:** First period is irregular (shorter or longer than standard).
**Back stub:** Last period is irregular.

O'Kane notes that in a CDS premium leg schedule, "the cash flows are not all equal amounts, reflecting the small but real differences in the time between payments caused by adjustments to avoid weekends and public holidays. There is also a short stub at the beginning reflecting the shorter period to the first premium payment."

Stubs matter because the **accrual fraction** for a stub period differs from regular periods, which changes the actual cash payment. O'Kane provides an example where "the first accrual fraction is 0.161111" for a short stub period—roughly 58 days out of 360—compared to the standard quarterly fraction of 0.25.
 
 **Visualizing a Long First Coupon:**
 
 ```mermaid
 timeline
     title Long First Coupon Timeline
     section Issuance
         Issue Date : 01 Jan
     section First Coupon (Long)
         Regular Q1 (Skipped) : 01 Apr
         First Payment (4+3=7mo?) -> No, Example:
         Issue Date : 15 Nov
         Regular Date : 15 Dec (Skipped)
         First Coupon : 15 Mar (4 months later)
     section Regular Coupons
         Second Coupon : 15 Jun
         Third Coupon : 15 Sep
 ```
 
 *Diagram Explanation*: In a "Long First Coupon," the first payment covers more than one full period (e.g., 4 months instead of 3). The payment will be $\approx 1.33 \times$ a normal coupon. This often happens when a bond is issued mid-cycle.


---

## 1.5 Clean vs Dirty: Avoiding Mechanical Price Jumps (Preview)

### The Problem with Quoting Full Price

Imagine a bond pays a $5 coupon every six months. If markets quoted the **dirty** (full) price, here's what would happen:

- Day 1 after coupon: dirty price = 100 (say)
- Day 90: dirty price = 102.50 (same "underlying" value, but accrued interest has built up)
- Coupon date: dirty price jumps down by ~$5 when the coupon is paid

This sawtooth pattern would make it hard to compare prices across dates or to distinguish real price moves from mechanical accrual.

### The Clean/Dirty Convention

Markets solve this by quoting the **clean** (flat) price and separately tracking **accrued interest**:

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + \text{AI}}$$

Where:
- $P_{\text{dirty}}$ is the actual cash exchanged per 100 face (the invoice price)
- $P_{\text{clean}}$ is the quoted price
- $\text{AI}$ is accrued interest

O'Kane explains the rationale: "The purpose of accrued interest is to avoid a sudden drop in the quoted price of bonds as we pass through a coupon payment which is what must happen with the bond full price. These effects are not helpful to investors as they do not represent a change in credit quality or interest rates. The market therefore prefers to quote bonds using a clean price."

### The Continuity Proof

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

### Why This Matters for P&L

When marking a bond position to market:

- If you track **clean price**, your P&L reflects genuine market moves. Coupon payments appear as separate coupon income.
- If you track **dirty price**, your P&L includes mechanical accrual drift, which must be separated from actual market moves.

Most trading desks quote clean and track accrued interest separately, then reconcile to the full settlement amount.

> **Stock vs Bond Convention:** Tuckman notes an important difference: "The behavior of quoted bond prices differs from that of stocks that do not have an accrued dividend convention. Stock prices fall by approximately the amount of the dividend on the day ownership of the dividend payment is established. The accrued convention does make more sense in bond markets than in stock markets because dividend payment amounts are generally much less certain than coupon payments."

**Full treatment of accrued interest mechanics appears in Chapter 5.**

---

## 1.6 Practical Sanity Checks

Every practitioner should run these checks on any fixed-income calculation:

### Timing Checks

- **Fixing date before accrual start:** For term rates, the rate must be known before accrual begins.
- **Payment date at or after accrual end:** You can't pay before the period is complete.
- **Settlement vs effective:** Know which date your valuation uses.
- **Holiday consistency:** Ensure all dates respect the appropriate calendar(s).

### Accrued Interest Checks

- $0 \leq \text{AI} \leq \text{Coupon}$ between payment dates
- AI resets to 0 immediately after coupon payment
- AI approaches full coupon immediately before payment
- AI computed to settlement date, not trade date

### Monotonicity Check (Positive Rates)

If rates are positive, discount factors should decrease with maturity. A discount factor curve that increases with maturity (except in negative-rate environments) suggests a bug.

### Convention Consistency

- Day count used for accrued interest should match the instrument convention
- Compounding assumption should match the market's convention
- Settlement lag should match the market standard
- Business day convention should match the contract specification

### Invoice Price Sanity

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

---

## Summary

This chapter established the "plumbing" that underlies all fixed-income pricing:

1. **Markets quote what's useful for their participants**: Bonds quote clean prices in 32nds; T-bills quote discount rates; money markets quote rates; swaps quote fixed rates. The quote alone isn't enough—you need the conventions.

2. **Day counts are a unit system**: ACT/ACT, 30/360, and ACT/360 measure time differently. Using the wrong convention means computing the wrong interest. The same one-day period can count as 1 day (ACT) or 3 days (30/360) depending on convention.

3. **Compounding is also a unit choice**: A 5% semiannual rate is not the same as 5% continuous. Conversion formulas exist, but mixing conventions creates errors. Continuous compounding is standard in derivatives pricing for its mathematical convenience.

4. **Settlement timing matters**: Cash moves on settlement date, not trade date. Accrued interest is computed to settlement. Different markets have different settlement lags (T+1 for Treasuries, T+2 for swaps).

5. **Clean pricing removes mechanical drift**: Markets quote clean to make prices comparable across dates. The dirty price is what actually changes hands. Tuckman proves that clean prices are continuous across coupon dates when yields are unchanged.

None of this is mathematically deep, but all of it must be exactly right. The chapters that follow will build on these conventions: Chapter 2 develops discount factors and present value; Chapter 3 covers the zero-forward-par rate relationships; Chapter 5 fully develops bond pricing with accrued interest mechanics.

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
| Business day roll | What to do when date is a holiday | Modified Following is common for swaps |
| Stub | Irregular first/last period | Affects actual cash payment amount |
| Fixing date | When floating rate is observed | Typically T-2 for term rates |

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

---

## Mini Problem Set

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

11. Aug 27 → Aug 30 = 3; Aug 30 → Nov 30 = 90 (3 months); Nov 30 → Nov 15: this is negative, so instead: Aug 30 → Nov 15 = 75 days (2 months + 15 days = 60 + 15). Total = 3 + 75 = 78 days.

12. Days from Mar 1 to Jul 3 = 124; Days from Mar 1 to Sep 1 = 184; Semiannual coupon = $4; AI = $(124/184) \times 4 = \$2.6957$

---

## Source Map

### (A) Verified Facts (Source-Backed)

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
| RFR backward-looking compounding | Hull Ch 4 |
| Stock vs bond accrual convention difference | Tuckman Ch 4 footnote |
| Continuous compounding preferred in derivatives | Hull Ch 4 |

### (B) Reasoned Inference (Derived from A)

- Clean pricing removes sawtooth accrual pattern (follows from the clean/dirty identity and the coupon-date continuity proof)
- Day counts are like measurement units (conceptual framing consistent with Tuckman's statement that conventions "do not really matter so long as cash flows are properly computed")
- Convention risk exists whenever conventions are mixed (implied by the existence of multiple conventions)
- Dirty price ≈ PV of remaining cashflows (follows from Tuckman's pricing equation P + AI = PV)

### (C) Flagged Uncertainties

- **End-of-month rules:** Exact EOM conventions for 30/360 vary by specific variant (30/360 ISDA, 30E/360, etc.); specific rules require contract documentation
- **Exact settlement lags:** May vary by market, currency, instrument type, and can change over time (e.g., U.S. equities moved from T+2 to T+1)
- **Business day calendars:** Holiday calendars are jurisdiction-specific and change annually; production systems require maintained calendar data
- **RFR compounding conventions:** Specific lookback, lockout, and payment delay conventions vary by currency and product; covered in Chapter 18
