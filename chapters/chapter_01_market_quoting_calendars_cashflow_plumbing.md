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

Prerequisites: none (start here).

Follow-on: [Chapter 2 — Time Value, Discount Factors, and Replication](chapters/chapter_02_time_value_discount_factors_replication.md); [Chapter 4 — Money-Market Building Blocks](chapters/chapter_04_money_market_building_blocks.md); [Chapter 5 — Fixed-Rate Bond Pricing](chapters/chapter_05_fixed_rate_bond_pricing.md); [Chapter 11 — DV01/PV01 Definitions and Computation](chapters/chapter_11_dv01_pv01_definitions_computation.md); [Chapter 18 — OIS Discounting Curve](chapters/chapter_18_ois_discounting_curve.md).

---

## Learning Objectives
- Translate common market quotes into a precise “quote object” with units (price per 100, rate with day count/compounding, spread in bp).
- Build a simple schedule (trade/settle/accrual/payment) using business-day conventions and holiday calendars.
- Compute accrual/year-fractions under ACT/ACT, ACT/360, and 30/360 (and know when 30/360 variants matter).
- Convert rates across compounding conventions and explain why conversion is a unit change (not “alpha”).
- Compute the cash settlement amount for a bond trade from clean price, accrued interest, and notional.
- Write the “cashflows → PV” equation and interpret a DV01-style sensitivity with an explicit bump object, units, and sign.

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

US Treasury bond prices are quoted in dollars and 32nds per $100 face value. The notation `99-16` means:

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

Finer increments use fractions of 32nds. The quote `101-04 5/8` means:

$$101 + \frac{4 + \frac{5}{8}}{32} = 101 + \frac{4.625}{32} = 101.14453125$$

**Tick Precision Across Instruments:** Different Treasury futures contracts use different precision levels. For example, some contracts quote in half-32nds (1/64) and others in quarter-32nds (1/128). A settlement price of `139-025` means $139 + \frac{2.5}{32} = 139.078125$. A settlement price of `125-132` means $125 + \frac{13.25}{32} = 125.4140625$.

The bond price quoted in the market is known as the **clean** (flat) price. It is different from the amount which is actually paid to buy a bond which is known as the **full** (dirty) price. The difference between the clean price and the full price is known as **accrued interest**.

Quoting clean prices and separating accrued interest avoids a mechanical sawtooth in quoted prices around coupon dates that does not reflect a change in rates or credit quality.

> **Example: Tick Size and Bid/Ask**
>
> Quotes in 32nds make small spreads natural to discuss in “ticks.” For illustration, a one‑tick (1/32 of par) spread on $100 million face corresponds to:
> `100,000,000 USD × (1/32) × (1/100) = 31,250 USD`
>
> A half‑tick (1/64) would be half that amount.

> **Desk Reality: What “Bid at 16, Offered at 16+” Means**
>
> Traders will often speak in shorthand:
> - “**Bid 16**” means the dealer is willing to **buy** at `…-16`.
> - “**Offered 16+**” means the dealer is willing to **sell** at `…-16+` (half‑tick higher).
>
> Crossing the bid/ask is the economic cost of immediacy. For large notionals, even “tiny” tick spreads show up in P&L, so desks obsess over price format, tick size, and where liquidity is deepest (on‑the‑run vs off‑the‑run).

### 1.1.3 T-Bill Quotations: Discount Rate Convention

The prices of money market instruments are sometimes quoted using a **discount rate**. This is the interest earned as a percentage of the final face value rather than as a percentage of the initial price paid for the instrument. An example is Treasury bills in the United States.

Let:
- $Y$ be the cash price per $100$ of face value,
- $q_{\text{disc}}$ be the quoted discount rate in **percent per year** (ACT/360, applied to face value),
- $n$ be days to maturity (calendar days).

Then the market relationship between the cash price and the quote is:

$$\boxed{q_{\text{disc}} = \frac{360}{n}(100 - Y)}$$

Many texts write the same relationship as:

$$P = \frac{360}{n}(100 - Y)$$

where $P$ is the quoted price, $Y$ is the cash price, and $n$ is the remaining life of the Treasury bill measured in calendar days. (In this chapter, $q_{\text{disc}}$ plays the role of $P$ in that notation.)

Equivalently:

$$Y = 100\left(1 - \frac{q_{\text{disc}}}{100}\frac{n}{360}\right).$$

**Why the discount quote understates investor return:** the quote divides the discount $(100-Y)$ by face value, but the investor’s cash outlay is $Y$. The simple-hold return over the bill’s life is $\frac{100-Y}{Y}$, which is larger than $\frac{100-Y}{100}$ when $Y\lt 100$.

> **Worked Example A (House Template): T-bill discount quote → cash price → PV → DV01 (quote bump)**
>
> **Context**
> - Instrument: a $91$-day zero-coupon Treasury bill (face value repaid at maturity).
> - Why it matters: the “quote object” (discount rate vs yield) changes both the cash price and any “01” number you report.
>
> **Timeline (Concrete Dates)**
> - Trade date: 2026-01-06
> - Settlement date: 2026-01-07 (assume $T+1$)
> - Maturity date: 2026-04-08 (91 calendar days after settlement)
>
> **Inputs**
> - Face value notional: $N = 10{,}000{,}000\ \text{USD}$
> - Quoted discount rate: $q_{\text{disc}} = 5.00$ (percent p.a., ACT/360, on face value)
> - Days to maturity: $n = 91$
>
> **Outputs (What You Produce)**
> - Cash price per $100$ face, $Y$
> - Settlement cash amount (buyer pays)
> - PV at settlement (value of the bill)
> - $DV01_{\text{disc}}$ (bump object: $q_{\text{disc}}$ down by $1$ bp; units: USD per 1 bp; sign: positive for a long)
>
> **Step-by-step**
> 1. Translate quote to interest on face:
>    $$\text{Interest per }100 = 100 \times \frac{5.00}{100} \times \frac{91}{360} = 1.2639.$$
> 2. Compute cash price per $100$ face:
>    $$Y = 100 - 1.2639 = 98.7361.$$
> 3. Compute settlement cash amount:
>    $$\text{Cash paid} = N \times \frac{Y}{100} = 10{,}000{,}000 \times 0.987361 = 9{,}873{,}611\text{ USD}.$$
> 4. PV equation (zero-coupon special case): the discount factor from settlement to maturity is
>    $$P(\text{settle},\text{mat}) = \frac{Y}{100} = 0.987361,$$
>    so the PV of the maturity cashflow is
>    $$PV = N \cdot P(\text{settle},\text{mat}) = 9{,}873{,}611\text{ USD}.$$
> 5. Risk (DV01-style “01”): define (book convention)
>    $$DV01_{\text{disc}} := PV(q_{\text{disc}}-1\text{bp}) - PV(q_{\text{disc}}).$$
>    Since $PV = N\left(1 - (q_{\text{disc}}/100)\frac{n}{360}\right)$,
>    $$DV01_{\text{disc}} = N \cdot \frac{n}{360} \cdot 10^{-4} = 10{,}000{,}000 \times \frac{91}{360} \times 10^{-4} = 252.78\text{ USD}.$$
>
> **Cashflows**
>
> | Date | Cashflow (long bill) | Explanation |
> |---|---:|---|
> | 2026-01-07 | -\$9,873,611 | Pay cash price at settlement |
> | 2026-04-08 | +\$10,000,000 | Receive face value at maturity |
>
> **P&L / Risk Interpretation**
> - If the quoted discount rate falls, the cash price rises. The computed $DV01_{\text{disc}}=252.78$ (USD per 1 bp) means: a **1 bp drop** in the *discount quote* increases PV by about \$253 for a \$10 million face position.
> - A different bump object (money-market yield, zero rate, curve node) will generally produce a different “01”. Always state what was bumped.
>
> **Sanity Checks**
> - Units: $q_{\text{disc}}$ is in percent p.a.; $1$ bp $=0.01$ percent points $=10^{-4}$ in decimal.
> - Sign: for a long bill, lower rates $\Rightarrow$ higher price $\Rightarrow DV01 \gt 0$ under the book convention.
> - Scaling: doubling $N$ doubles both PV and $DV01$; longer $n$ increases $DV01$ linearly.
>
> **Pitfall — “01” without a bump object:** saying “DV01 is \$253” is incomplete unless you also specify **what you bumped** (yield, zero rate, par rate, or a quote like $q_{\text{disc}}$). Two systems can both report “DV01” for the same trade and disagree simply because they bumped different objects.
> **Why it matters:** hedge ratios and P&L explain will be wrong even if both systems are “internally consistent.”
> **Quick check:** when reconciling risk, write down the bump object and units (e.g., “$q_{\text{disc}}$ down 1 bp” vs “zero curve down 1 bp”).

### 1.1.4 Money Markets: Rates with Day Counts

Money market instruments quote rates, not prices. But a "5% rate" is meaningless without knowing:

1. **Day count convention**: How do we measure the time period?
2. **Compounding convention**: Simple interest? Compound interest?

A 3-month rate (e.g., legacy LIBOR or Term SOFR) of 5% under **Actual/360** with simple interest means that for a loan of $1 over $d$ actual days:

$$\text{Interest} = 1 \times 0.05 \times \frac{d}{360}$$

The denominator is 360 (not 365 or actual days in the year). This is not a mathematical choice—it is a market convention that affects the actual dollars owed.

### 1.1.5 Why Conventions Differ Across Markets

Conventions are historical and contractual: they exist to standardize communication and settlement, not to optimize mathematical elegance.

The takeaway is simple: treat the quote *and* its conventions as one package. If you are missing the day count, compounding, calendar, or settlement assumptions, you are missing part of the price.

---

## 1.2 Day Counts: The Unit System of Interest Rates

### 1.2.1 Day Counts Define How Time Is Measured

A **day count convention** defines how to convert a pair of dates into a **year fraction** (also called an accrual fraction). That year fraction is the time unit used in interest calculations. Concretely, a day count convention determines two things:

1. How to count days between two dates (the numerator)
2. How to define the "reference period" (the denominator)

The interest earned between two dates is then:

$$\boxed{\text{Interest} = \frac{\text{Number of days between dates}}{\text{Number of days in reference period}} \times \text{Interest earned in reference period}}$$

This fraction—(days between dates) / (days in reference period)—is the **accrual fraction** or **year fraction**, often denoted $\Delta$ or $\tau$.

### 1.2.2 The Three Common US Conventions

**Actual/Actual (in period):** Used for US Treasury bonds. Both numerator and denominator use actual calendar days.

- Numerator: actual days between dates
- Denominator: actual days in the coupon period

**Worked Example B:** Consider a Treasury bond with coupon payment dates March 1 and September 1, paying an 8% annual coupon ($4 semiannually). To calculate interest earned between March 1 and July 3:

- Reference period: March 1 to September 1 = 184 actual days
- Days elapsed: March 1 to July 3 = 124 actual days
- Interest earned: $\frac{124}{184} \times 4\ \text{USD} = 2.6957\ \text{USD}$

**30/360:** Used for US corporate and municipal bonds. Assumes 30 days per month and 360 days per year.

- Numerator: computed assuming each month has 30 days
- Denominator: 360

Under 30/360, we assume 30 days per month and 360 days per year when carrying out accrual calculations. For the same March 1 to July 3 period:

- Days elapsed: $(4 \times 30) + 2 = 122$ days (4 full months plus 2 days)
- Reference period: 180 days (6 months × 30 days)
- Interest earned: $\frac{122}{180} \times 4\ \text{USD} = 2.7111\ \text{USD}$

### 1.2.3 The 30/360 Day Counting Algorithm

The 30/360 convention requires a specific algorithm for counting days. For example, to count days from August 27, 2001 to February 15, 2002:

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

This means a full year of 365 days earns $365/360 \approx 1.0139$ times the quoted rate, not exactly the quoted rate.

### 1.2.4 International Day Count Variants

Day count conventions vary by market and instrument, and there are multiple “flavors” of the same headline convention (especially for 30/360, where end‑of‑month rules matter).

Treat the day count as part of the contract: the correct choice is the one specified in the instrument documentation (confirmation, term sheet, rulebook).

A quick sanity check: under ACT/360 with simple interest, a “5%” rate applied for 365 actual days accrues \(0.05\times 365/360 \approx 5.069\%\) of notional over the year.

Here is a compact “you will see these on the desk” list:

| Convention (headline) | Numerator | Denominator | Where you’ll commonly see it |
|---|---:|---:|---|
| ACT/360 | Actual days | 360 | USD/EUR/JPY money markets; many floating legs |
| ACT/365F | Actual days | 365 | Many GBP money market instruments |
| 30/360 (Bond basis) | 30‑day months | 360 | US corporate bonds (coupon accrual) |
| 30E/360 (Eurobond) | 30‑day months (Euro end‑of‑month rules) | 360 | Some EUR bond markets |
| 30E/360 (ISDA) | 30‑day months (ISDA rules) | 360 | Some swaps / ISDA definitions |

**Important:** “30/360” is not one thing. End‑of‑month handling differs across variants, and that difference can move accrued interest and PVs around coupon dates. Always verify which variant your system is using.

### 1.2.5 Why Day Counts Matter: The "Million Dollar Calendar Mistake"

A classic gotcha happens around the end of February. Between February 28 and March 1 in a non-leap year:

- Under **30/360**: There are 3 days (Feb 28 → Feb 30 → Mar 1)
- Under **ACT/ACT**: There is 1 day

If a corporate bond and a Treasury bond have the same coupon and same quoted price, the corporate bond (using 30/360) accrues **three times as much interest** over this one-day period as the Treasury (using ACT/ACT).

This is not a mathematical curiosity—it affects which bond you should prefer to hold over that date.

### 1.2.6 Day Count as a "Unit System"

Think of day counts like measurement systems. Quoting a rate as "5%" without specifying the day count is like saying a distance is "100" without specifying whether it's meters or feet.

The same economic arrangement can have different quoted rates depending on the day count. Consider a loan from February 15, 2001, to August 15, 2001 (181 actual days):

- At 5% under Actual/360: $\text{Interest} = 5\\% \times \frac{181}{360} = 2.5139\\%$
- At 5% under semiannual compounding: $\text{Interest} = \frac{5\\%}{2} = 2.500\\%$
- At 5% under monthly compounding: $\text{Interest} = (1 + 0.05/12)^6 - 1 = 2.5262\\%$
- At 5% under daily compounding: $\text{Interest} = (1 + 0.05/365)^{181} - 1 = 2.5103\\%$

Different numbers, same economic reality: if cashflows are computed correctly, the economic payment is the same. The danger is mixing conventions mid-calculation.

> **Worked Example C: Same Period, Three Conventions**
>
> Calculate accrued interest from March 15 to June 15 (92 actual days, in a coupon period of 182 days) on $100mm face of a 6% coupon bond.
>
> | Convention | Day Fraction | Accrued Interest |
> |------------|--------------|------------------|
> | ACT/ACT | $\frac{92}{182} = 0.5055$ | $100\text{mm} \times 3\% \times 0.5055 = 1{,}516{,}484\ \text{USD}$ |
> | 30/360 | $\frac{90}{180} = 0.5000$ | $100\text{mm} \times 3\% \times 0.5000 = 1{,}500{,}000\ \text{USD}$ |
> | ACT/360 | $\frac{92}{180} = 0.5111$ | $100\text{mm} \times 3\% \times 0.5111 = 1{,}533{,}333\ \text{USD}$ |
>
> **Difference:** The gap between highest and lowest is $33,333—material for trade confirmation and P&L.

---

## 1.3 Compounding: Another Unit Choice

### 1.3.1 What Compounding Means

Compounding frequency determines how often interest is calculated and added to principal. With compounding $m$ times per year, an investment of $A$ at annual rate $R$ grows to:

$$\boxed{A\left(1 + \frac{R}{m}\right)^{mn}}$$

after $n$ years.

The effect of compounding frequency is illustrated below:

| Compounding Frequency | Value of $100$ at end of 1 year (10% rate) |
|----------------------|------------------------------------------|
| Annually ($m=1$) | $110.00$ |
| Semiannually ($m=2$) | $110.25$ |
| Quarterly ($m=4$) | $110.38$ |
| Monthly ($m=12$) | $110.47$ |
| Weekly ($m=52$) | $110.51$ |
| Daily ($m=365$) | $110.52$ |
| Continuous | $110.52$ |

### 1.3.2 Continuous Compounding

**Continuous compounding** is the limit as $m \to \infty$. With continuous compounding, an amount $A$ invested for $n$ years at rate $R$ grows to:

$$\boxed{Ae^{Rn}}$$

And discounting uses the inverse:

$$\boxed{e^{-Rn}}$$

For many practical purposes, continuous compounding is close to daily compounding.

> **Why Continuous Compounding in Derivatives**
>
> Continuous compounding is widely used in derivatives because it makes formulas cleaner: discount factors become exponentials and multiply simply, e.g.
> $$e^{-r_1 t_1}\times e^{-r_2 t_2}=e^{-(r_1 t_1+r_2 t_2)}.$$
> Treat compounding basis as part of the rate’s unit. Convert to the market’s quoting convention when comparing prices or risks across systems.

### 1.3.3 Compounding Is a Unit Choice, Not an Economic Choice

There can be only one market-clearing interest payment over a period. The actual dollars exchanged are fixed by supply and demand. What differs is how we *express* that payment as a rate.

A 5% rate with semiannual compounding and a 4.939% rate with continuous compounding describe the **same** economic outcome over a six-month period:

- Semiannual: $1 \times (1 + 0.05/2) = 1.025$
- Continuous: $1 \times e^{0.04939 \times 0.5} = 1.025$

Compounding conventions matter for computing cashflows. For valuation, the fundamental quantities are prices of cashflows on specific dates (discount factors); the quote format is just a representation.

### 1.3.4 Conversion Formulas

When converting between compounding conventions, use the conversion formulas. To convert from $m$-times-per-year compounding at rate $R_m$ to continuous at rate $R_c$:

$$\boxed{R_c = m \ln\left(1 + \frac{R_m}{m}\right)}$$

And the reverse:

$$\boxed{R_m = m\left(e^{R_c/m} - 1\right)}$$

More generally, to convert from compounding $m_1$ times per year at rate $R_1$ to compounding $m_2$ times per year at rate $R_2$:

$$R_2 = m_2\left[\left(1 + \frac{R_1}{m_1}\right)^{m_1/m_2} - 1\right]$$

### 1.3.5 Convention Risk

The practical danger is **convention risk**: accidentally using the wrong compounding assumption. A risk system that assumes continuous compounding when the market quotes semiannual will systematically misprice by a small but consistent amount—easily enough to cause P&L breaks.

Practical rule: before comparing rates across systems, convert them to a common convention (or convert to discount factors using consistent year fractions) and then compare.

**Worked Example D:** Convert 10% semiannual to continuous.

$$R_c = 2 \ln(1 + 0.10/2) = 2 \ln(1.05) = 2 \times 0.04879 = 0.09758 = 9.758\\%$$

The continuous rate is lower because interest is being compounded more frequently.

**Worked Example E:** Convert 8% continuous to quarterly.

$$R_m = 4 \times (e^{0.08/4} - 1) = 4 \times (e^{0.02} - 1) = 4 \times 0.0202 = 0.0808 = 8.08\\%$$

This means on a 1,000 USD loan with interest paid quarterly, each payment would be $1{,}000 \times 0.0808/4 = 20.20$ USD.

**Worked Example F:** Convert 6% semiannual to quarterly.

$$R_2 = 4\left[\left(1 + \frac{0.06}{2}\right)^{2/4} - 1\right] = 4\left[(1.03)^{0.5} - 1\right] = 4 \times 0.01489 = 0.0596 = 5.96\\%$$

---

## 1.4 Settlement, Calendars, and Timing

### 1.4.1 Trade Date vs Settlement Date

When you trade a bond, cash does not move immediately. The **settlement date** is when the buyer pays and the seller delivers. The gap between trade date and settlement is the **settlement lag**.

Many markets have a standard settlement lag. For example, US Treasuries often settle T+1 (one business day after trade), while vanilla interest rate swaps typically settle T+2.

| Instrument | Typical Settlement |
|------------|-------------------|
| US Treasuries | T+1 |
| Vanilla interest rate swaps | T+2 |

Settlement conventions differ across markets and can change over time. In practice, treat settlement lag as part of the instrument definition and confirm the relevant convention before pricing or risk.

### 1.4.2 What Happens on Settlement Day

On the settlement date, the purchaser of the bond pays the full bond price to the seller. The full (dirty) price is the quoted clean price plus accrued interest computed under the relevant accrual convention.

Settlement matters for two key reasons:

1. **Accrued interest is computed to the settlement date, not the trade date.** If you trade on Friday but settle on Monday, the seller gets three extra days of accrued interest.

2. **Discount factors are computed from the settlement date.** When pricing a bond, the "today" in your present value calculation is settlement, not trade date.

**Mechanics (cashflows → PV):** a fixed-income instrument is a schedule of future cashflows. Given discount factors $P(t,T_i)$, the present value at valuation date $t$ is:

$$\boxed{PV(t) = \sum_i CF_i \\, P(t, T_i)}$$

Settlement then specifies *when* the invoice cash is exchanged. To reconcile a quoted price to a model PV, be explicit about your valuation date (trade date vs settlement date vs end-of-day) and carry the value between dates consistently. Over a short settlement lag, that adjustment is essentially “interest on the cash you pay/receive because settlement is later,” so it scales with notional and short rates.

> **Check (order of magnitude): settlement lag is funding-sized**
>
> Suppose you buy \$100,000,000 face of a bond at a dirty price of 99.30, so the invoice amount is about \$99.30mm. If settlement is 3 calendar days later and you use a hypothetical 5% p.a. discount rate for that lag (ACT/360, simple interest), the time value over the lag is roughly:
> $$99.30\text{mm}\times 0.05 \times \frac{3}{360} \approx 41{,}000\ \text{USD}.$$
> This is not “alpha”; it’s a mechanical timing effect. If your system prices as of trade date but accrues AI to settlement (or vice versa), this is the kind of reconciliation break you’ll see.

### 1.4.3 When Settlement Fails

What happens when the seller cannot deliver the bonds on settlement day? This creates a **settlement fail**.

When a security is hard to source, a short position may be unable to borrow it in repo/securities lending to make delivery, leading to a fail. Economically, failing delays receipt of sale proceeds, so the short forgoes earning interest on those proceeds until it can deliver.

A useful (but imperfect) intuition is that failing can compete with borrowing the security: if avoiding a fail is the only objective, a trader will not accept an arbitrarily punitive special repo rate. Real markets can include explicit fails charges, balance sheet costs, and even negative rates, so treat this as intuition rather than a hard bound.

> **Desk Reality: Fails Are Funding Events**
>
> A fail is not just an “ops exception.” Economically, it is a **funding / financing disruption**:
> - the short can’t deliver, so it doesn’t receive sale proceeds when expected;
> - the long doesn’t receive the bond, so it may have to replace it (or fail onward).
>
> **US Treasuries: Fails Charge (Illustrative Formula)**
>
> A common market convention is to apply a daily fails charge for delivery-versus-payment (DVP) Treasury trades. One widely used functional form is:
>
> $$\boxed{C \\;=\\; \frac{1}{360}\times 0.01 \times \max(3 - R,\\; F)\times P}$$
>
> where:
> - $C$ = fails charge amount, in dollars (accrues each calendar day during the fail);
> - $P$ = “trade proceeds”, i.e., the amount of funds due from the non-failing party (for DVP);  
>   *(Here P denotes proceeds. Do not confuse it with bond price notation elsewhere in this chapter.)*
> - $R$ = reference short rate used for the charge calculation (definition is convention-specific);
> - $F$ = floor in percentage points per annum so the charge does not collapse to zero when rates are very low (definition is convention-specific).
>
> The exact definitions of \(R\) and \(F\) (including any floors/thresholds and effective dates) are rulebook inputs: confirm them for your product and venue.
>
> **Intuition:** when short rates are low, failing is otherwise “cheap,” so the fails charge creates an explicit cost. When short rates are high enough, the opportunity cost of not receiving proceeds is already meaningful and the explicit charge may be small or zero (subject to any floor).
>
> **Worked number (order of magnitude):** If $P=100{,}000{,}000\ \text{USD}$ and the applicable fails charge rate is 2% per annum, then $C \approx 100{,}000{,}000 \times 0.02/360 = 5{,}556\ \text{USD}$ per day.

One historical stress episode: after September 11, 2001, operational disruption increased Treasury settlement fails and reduced the availability of on-the-run collateral in the specials market, contributing to shortages.

### 1.4.4 Business Day Conventions

What happens when a payment date falls on a weekend or holiday? Markets use **business day adjustment rules**.

**Following:** Move to the next business day.

**Modified Following:** Move to the next business day, *unless* that would push into the next calendar month—in which case, move to the previous business day instead.

**Preceding:** Move to the previous business day.

**Modified Preceding:** Move to the previous business day, unless that would push into the previous calendar month—in which case, move forward.

Which convention applies is a contract term (swap confirmations typically specify it).

### 1.4.5 Holiday Calendars

Different markets use different holiday calendars:

| Example market | Example holiday calendar |
|----------------|--------------------------|
| USD | US calendar (as specified in the confirmation) |
| EUR | TARGET (used in some EUR contracts; product-specific) |

For multi-currency products, contracts often specify joint business-day rules (e.g., “must be a business day in both calendars”).

Treat the holiday calendar as a contract input (don’t assume “USD calendar” means the same thing across products).

> **Desk Reality: Calendar Mismatch Can Create “Stranded Liquidity”**
>
> Multi-currency and cross-market books are exposed to calendar mismatch:
> - one currency’s market can be open while the other is closed;
> - margin calls, FX funding, and hedges may not line up cleanly.
>
> Practical habit: when planning funding and risk for a week (especially around year‑end), check the joint calendar for your major currencies and the business-day convention in each confirmation.

### 1.4.6 Fixing Dates vs Payment Dates

For **term** floating-rate instruments (like legacy LIBOR or Term SOFR), the **fixing date** (when the rate is observed) typically precedes the **accrual start date** by 2 business days. This gives operational time to calculate the payment.

In a standard term swap, the floating coupon is set in advance (on a fixing date), accrues over the period, and is paid in arrears (on the payment date). The fixing date typically precedes the accrual start date by 2 business days for term rates.

A typical vanilla term swap timeline for one period:

```
Fixing Date (T-2 business days) → Accrual Start → Accrual End → Payment Date
                                   |_____________Accrual Period_____________|
```

The payment is calculated based on the rate observed on the fixing date, but the cash moves on the payment date.

### 1.4.7 Risk-Free Rate (RFR) Compounding Mechanics

In many markets, LIBOR-type term rates have been replaced by backward-looking overnight reference rates such as SOFR (USD) and SONIA (GBP). This creates operational complexity because the applicable rate is not known until the accrual period ends.

Overnight reference rates are backward-looking: the rate applicable to a particular period is not known until the end of the period when all the relevant overnight rates have been observed.

The compounding formula for RFRs is:

$$\boxed{\text{Compounded Rate} = \left[\prod_{i=1}^{n}(1 + r_i \hat{d}_i) - 1\right] \times \frac{360}{D}}$$

where $r_i$ is the overnight rate on day $i$, `d_hat_i = d_i/360`, `d_i` is the number of days that rate applies (usually 1, but 3 over weekends), and `D = sum_i d_i` is the total number of days in the period.

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
> $$\text{Compounded Rate} = (1.001031 - 1) \times \frac{360}{7} = 5.303\\%$$

Because the rate is determined from realized overnight fixings, operational timing conventions (when you observe rates and when you pay) are part of the contract specification. This book treats those timing conventions as inputs (we return to them when discussing curve construction and instrument mechanics later).

> **Desk Reality: “Observation Shift”, “Lookback”, “Lockout”, “Payment Delay”**
>
> For overnight-compounded legs, desks care about *when* a fixing is taken versus *when* cash is paid. Market standards vary by currency and product, but common operational patterns include:
> - **Observation shift / lookback:** use an earlier fixing (e.g., shift the observation window) so the payment amount can be known before the payment date.
> - **Lockout:** keep the final few daily rates fixed (reuse an earlier fixing) near period end to simplify operations.
> - **Payment delay:** pay a few business days after period end, so all overnight rates are known before cash moves.
>
> The names (and exact definitions) are not universal. If you are reconciling cashflows across systems, pull the trade confirmation and identify which timing convention each system implemented.

### 1.4.8 Stub Periods

When a swap or bond's effective date doesn't align with the regular payment schedule, the first or last period may be shorter or longer than standard. This creates a **stub period**.

**Front stub:** First period is irregular (shorter or longer than standard).
**Back stub:** Last period is irregular.

In real schedules, cashflows are not all equal amounts: small differences arise from weekend/holiday adjustments, and there can be stub periods at the start or end.

Stubs matter because the **accrual fraction** for a stub period differs from a regular period (for example, it is not exactly $0.25$ for a quarterly period under ACT/360), which changes the actual cash payment.

> **Worked Example H: Stub Period Accrual**
>
> A CDS contract trades on October 15 with quarterly payment dates on the 20th of each month (Dec, Mar, Jun, Sep). The first payment date is December 20.
>
> **Front stub calculation (ACT/360):**
> - Days from Oct 15 to Dec 20 = 66 days
> - Accrual fraction = $\frac{66}{360} = 0.1833$
> - If spread = 100bp on \$10mm notional: $10{,}000{,}000 \times 0.01 \times 0.1833 \approx 18{,}333$ (USD)
>
> Compare to regular quarterly payment:
> - Accrual fraction = $\frac{90}{360} = 0.25$
> - Payment = $10{,}000{,}000 \times 0.01 \times 0.25 = 25{,}000$ (USD)
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

*In a "Long First Stub," the first payment covers more than one full period (e.g., 4 months instead of 3). The payment will be approximately 1.33x a normal coupon.*

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

The purpose of separating accrued interest is to avoid a sudden drop in the *quoted* price as you pass through a coupon payment (a drop that must occur in the full/dirty price). These mechanical effects are not helpful to investors because they do not represent a change in credit quality or interest rates.

### 1.5.3 The Continuity Proof

Clean prices are continuous across coupon dates when yields don't change. The argument is a simple bookkeeping identity:

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


### 1.5.4 Why This Matters for P&L

When marking a bond position to market:

- If you track **clean price**, your P&L reflects genuine market moves. Coupon payments appear as separate coupon income.
- If you track **dirty price**, your P&L includes mechanical accrual drift, which must be separated from actual market moves.

Most trading desks quote clean and track accrued interest separately, then reconcile to the full settlement amount.

**Mechanics (clean/dirty is bookkeeping, not economics):** a useful way to think about bond “value through time” is that your total value is clean price plus accrued interest plus any coupon cash received since the last coupon date. When yields are unchanged, clean price should be roughly flat; the dirty price drifts mainly because AI builds up. On coupon date, AI resets but coupon cash arrives, leaving total value continuous.

**Toy numbers (per \$100 face):** suppose a bond pays a \$2.50 semiannual coupon. If yields do not move, take $P_{\text{clean}}=100$ throughout. Halfway through the coupon period, $AI\approx 1.25$ so $P_{\text{dirty}}\approx 101.25$. Just before the coupon, $AI\approx 2.50$ so $P_{\text{dirty}}\approx 102.50$. On coupon date you receive \$2.50 and $AI\to 0$, so $P_{\text{dirty}}\to 100$. Total value: $102.50$ before; $100 + 2.50 = 102.50$ after. The “dirty drop” is a cashflow, not a mark-to-market loss.

> **Desk Reality: P&L Reporting and Clean/Dirty**
>
> Trading systems typically separate P&L into:
> - **Mark-to-market P&L:** Change in clean price × position
> - **Carry/Accrual:** Interest income (accrued or received)
> - **Funding cost:** Cost of financing the position
>
> If your system reports dirty price P&L, you'll see a steady "gain" between coupon dates (accrual building) and a "loss" on coupon date (accrual resets). This is mechanical, not a trading signal. Traders use clean price for actual market direction.

> **Stock vs Bond Convention**
>
> Stock prices often drop by roughly the dividend on the ex-dividend day. Bond **clean** prices do not mechanically drop by the coupon on coupon dates because the accrued-interest convention offsets it.

> **Worked Example I: Clean/Dirty Calculation**
>
> A 5% semiannual Treasury bond trades at a clean price of 98-16 (= 98.50). Settlement is July 15. The last coupon was May 15. The next coupon is November 15.
>
> **Step 1:** Calculate days accrued
> - May 15 to July 15 = 61 days
> - May 15 to November 15 = 184 days (full period)
>
> **Step 2:** Calculate accrued interest (ACT/ACT)
> $$AI = \frac{61}{184} \times \frac{5\\%}{2} \times 100 = \frac{61}{184} \times 2.50 = 0.829$$
>
> **Step 3:** Calculate dirty price
> $$P_{\text{dirty}} = 98.50 + 0.829 = 99.329$$
>
> **Step 4:** Cash exchanged on $1mm face
> `Cash = 1,000,000 USD × (99.329/100) = 993,290 USD`
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

A useful sanity identity (when your cashflows and discounting are consistent) is:

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

2. **Quotes become PV only after you build cashflows**: the desk pipeline is quote → conventions/schedule → cashflows → PV. Discount factors give the present value today of one unit of currency to be received on a date. Summing the present value of each cashflow gives the value of the security (previewed in Worked Example A).

3. **Day counts are a unit system**: ACT/ACT, 30/360, and ACT/360 measure time differently. Using the wrong convention means computing the wrong interest. The same one-day period can count as 1 day (ACT) or 3 days (30/360) depending on convention.

4. **Compounding is also a unit choice**: A 5% semiannual rate is not the same as 5% continuous. Conversion formulas exist, but mixing conventions creates errors. Continuous compounding is standard in derivatives pricing for its mathematical convenience.

5. **Settlement timing matters**: Cash moves on settlement date, not trade date. Accrued interest is computed to settlement. Different markets have different settlement lags (e.g., T+1 for Treasuries, T+2 for swaps). When settlement fails, there are economic consequences and it often shows up as a funding/financing event.

6. **Overnight reference rates are backward-looking**: Unlike term rates that are fixed in advance, overnight reference rates compound from realized overnight fixings and are known only at period end. Operational timing conventions are contract-specific.

7. **Clean pricing removes mechanical drift**: Markets quote clean to make prices comparable across dates. The dirty price is what actually changes hands. When yields are unchanged, clean prices can be continuous across coupon dates even though the dirty price drops by the coupon as accrued interest resets.

8. **Risk numbers need a bump object**: DV01 is a PV sensitivity per 1 bp *for a stated bump design* (“what is being bumped?”). Two systems can both report “DV01” and disagree simply because they bumped different objects (yield vs curve nodes vs a quote like a T-bill discount rate).

None of this is mathematically deep, but all of it must be exactly right. The chapters that follow build on these conventions: Chapter 2 develops discount factors and present value; Chapter 3 covers the zero-forward-par rate relationships; Chapter 5 develops bond pricing with accrued interest mechanics; Chapter 11 formalizes DV01/PV01; Chapter 18 covers RFR curve construction in depth.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Clean vs dirty | Dirty = clean + accrued interest | Clean is quoted; dirty is exchanged |
| Day count (X/Y) | Fraction = (days between) / (days in reference period) | Different conventions → different dollars |
| 30/360 | Assume 30 days/month, 360 days/year | Used for corporates; smooths calendar irregularity |
| ACT/ACT | Actual days / actual days in period | Used for Treasuries; exact calendar counting |
| ACT/360 | Actual days / 360 | Used for money markets; inflates annual return |
| Compounding | Frequency of interest computation | Unit of rate quotes; affects conversions |
| Continuous compounding | Limit as frequency → ∞ | Standard in derivatives pricing |
| Discount factor | PV today of \$1 paid on a date | Turns dated cashflows into PV; foundation for curves and risk |
| Settlement | Date when cash moves | Accrued computed to settlement |
| Settlement fail | Failure to deliver on settlement date | Has operational and economic consequences (e.g., foregone interest on sale proceeds; repo implications) |
| DV01 (dollar duration) | PV change for a 1 bp move in a **specified** rate/quote (“bump object”) | Risk numbers and hedge ratios are meaningless without a bump definition |
| Business day roll | What to do when date is a holiday | Modified Following is common for swaps |
| Stub | Irregular first/last period | Affects actual cash payment amount |
| Fixing date | When floating rate is observed | Typically T-2 for term rates |
| RFR compounding | Daily rate compounded over period | Backward-looking; not known until period end |

---

## Notation

The following notation appears in this chapter (and many symbols reappear in later chapters):

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $P_{\text{clean}}$ | Clean/quoted/flat price | price per $100$ notional |
| $P_{\text{dirty}}$ | Dirty/invoice/full price | price per $100$ notional; $P_{\text{dirty}} = P_{\text{clean}} + AI$ |
| $AI$ | Accrued interest | price points per $100$ notional; computed to settlement date |
| $N$ | Notional / face value | currency |
| $CF_i$ | Cashflow paid at date $T_i$ | currency; signed with positive = receive |
| $P(t,T)$ or $d(T)$ | Discount factor | unitless; PV at $t$ of $1$ paid at $T$ |
| $\tau(t_1,t_2)$ (also $\alpha$, $\Delta$) | Year fraction between dates | years; depends on day count convention |
| $c$ | Coupon rate | per year; coupon cashflow $\approx N\\,c\\,\tau$ |
| $m$ | Compounding / payment frequency | per year |
| $z(T)$ | Zero (spot) rate to maturity $T$ | per year; compounding basis must be stated |
| $f(T_1, T_2)$ | Forward rate for period $[T_1, T_2]$ | per year; day count + compounding must be stated |
| `q_disc` | T-bill quoted discount rate | percent points p.a. on face value, ACT/360 |
| $R_c$ | Continuously compounded rate | per year |
| $R_m$ | Rate compounded $m$ times per year | per year |
| $DV01$ | “01” PV sensitivity | currency per 1 bp for a *stated bump object*; book convention: $PV(\text{rates down }1\text{bp})-PV(\text{base})$ |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the difference between clean and dirty price? | Clean is quoted; dirty = clean + accrued interest is exchanged |
| 2 | Why do bond markets quote clean prices? | To remove mechanical accrual drift and make prices comparable across dates |
| 3 | What does the day count convention determine? | How to compute the fraction of time between dates for interest calculation |
| 4 | Name the three common US day count conventions | Actual/Actual (Treasuries), 30/360 (corporates), Actual/360 (money markets) |
| 5 | Under Actual/360, a full 365-day year earns how much of the quoted rate? | 365/360 = 1.0139 times the quoted rate |
| 6 | What is Modified Following? | Move to next business day unless that crosses month-end, then move back |
| 7 | What is a stub period? | An irregular first or last accrual period that doesn't match the standard length |
| 8 | What is T+1 settlement? | Cash and securities exchange one business day after trade date |
| 9 | To what date is accrued interest computed? | Settlement date (not trade date) |
| 10 | What is a fixing date? | The date when a floating rate is observed (typically 2 business days before accrual start for term rates) |
| 11 | How are US Treasury prices quoted? | In dollars and 32nds per $100 face value |
| 12 | Convert 100-16 to decimal | 100 + 16/32 = 100.50 |
| 13 | Why does compounding frequency matter? | It's part of the rate's "unit"; different frequencies with same number mean different economics |
| 14 | Formula to convert semiannual rate to continuous | $R_c = 2\ln(1 + R_m/2)$ |
| 15 | What check confirms you haven't added accrued twice? | Dirty price should roughly equal PV of remaining cashflows |
| 16 | How many days between Feb 28 and Mar 1 under 30/360? | 3 days (Feb 28 → Feb 30 → Mar 1) |
| 17 | How many days between Feb 28 and Mar 1 under ACT/ACT? | 1 day |
| 18 | Why is continuous compounding used in derivatives pricing? | Mathematical convenience: discount factors multiply simply |
| 19 | How are T-bill discount quotes related to cash prices? | `q_disc = (360/n) * (100 - Y)` where `q_disc` is the discount quote (% p.a.) and `Y` is cash price per `100` face |
| 20 | Give an example of a holiday calendar used in EUR contracts | TARGET (product-specific) |
| 21 | What is a settlement fail? | When the seller cannot deliver securities on the agreed settlement date |
| 22 | What is DV01 and what is the “bump object”? | DV01 is PV change per 1 bp for a defined rate/quote bump; you must state what is being bumped (yield/zero/par/quote) and the sign convention |
| 23 | How are RFR (overnight) rates compounded over a period? | Daily rates are compounded: $\prod(1 + r_i \times d_i/360) - 1$, annualized |

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

14. Calculate the compounded rate for a **5-day** period where Monday's SOFR is 5.25% (applies 1 day), Tuesday's is 5.28% (applies 1 day), and Wednesday's is 5.26% (applies 3 days). Express as an annualized rate.

15. (Compute) A 182-day T-bill with face value $25,000,000 is quoted at a discount rate of 4.50. Using ACT/360 bank discount quoting, compute:
    (a) the cash price per $100$ face,
    (b) the settlement cash paid,
    (c) $DV01_{\text{disc}} := PV(q_{\text{disc}}-1\text{bp}) - PV(q_{\text{disc}})$.

### Solution Sketches (Selected)

1. $98 + 24/32 = 98.75$

2. $\text{AI} = (60/182) \times 3 = 0.989$

3. $1{,}000{,}000 \times 0.04 \times 90/360 = 10{,}000\ \text{USD}$

4. $R_c = 2\ln(1.04) = 2 \times 0.0392 = 7.84\\%$

5. Friday (Wednesday + 2 business days)

6. Monday, June 21 (roll forward to next business day; stays in same month)

7. 30/360 counts February as having 30 days; Actual/Actual counts 28 (or 29 in leap year). End of February to March 1 is 3 days under 30/360 but only 1 day under Actual/Actual.

8. $102.50 + 1.875 = 104.375$

9. Interest = $100 \times 0.05 \times 91/360 = 1.2639\ \text{USD}$; Cash price = $100 - 1.2639 = 98.7361\ \text{USD}$

10. $R_2 = 4[(1.03)^{0.5} - 1] = 4 \times 0.01489 = 5.96\\%$

11. Aug 27 → Aug 30 = 3 days; Aug 30 → Nov 15 = 2 months + 15 days = 75 days. Total = 78 days.

12. Days from Mar 1 to Jul 3 = 124; Days from Mar 1 to Sep 1 = 184; Semiannual coupon = $4$; AI = $(124/184) \times 4 = 2.6957\ \text{USD}$

13. Days from Nov 10 to Dec 20 = 40 days. Accrual fraction = 40/360 = 0.1111. Payment = $5{,}000{,}000 \times 0.015 \times 0.1111 \approx 8{,}333$ (USD).

14. Compound: $(1 + 0.0525/360)(1 + 0.0528/360)(1 + 0.0526 \times 3/360) - 1 = 0.000731$. Annualized: $0.000731 \times 360/5 = 5.26\\%$

15. (a) Interest per $100$: $100 \times 0.045 \times 182/360 = 2.275$ so $Y=97.725$. (b) Cash paid: $25{,}000{,}000 \times 0.97725 = 24{,}431{,}250\text{ USD}$. (c) $DV01_{\text{disc}} = N \times (n/360) \times 10^{-4} = 25{,}000{,}000 \times 182/360 \times 10^{-4} = 1{,}263.89\text{ USD}$.
---

## References

- (Bruce Tuckman, *Fixed Income Securities*, “Treasury Bond Quotations”; “Discount Factors”; “The Law of One Price”)
- (John C. Hull, *Options, Futures, and Other Derivatives*, “Day Counts”; “Confirmations”; “Price Quotations of U.S. Treasury Bills/Bonds”)
- (Dominic O’Kane, *Modeling Single-name and Multi-name Credit Derivatives*, “Accrued Interest”; “Mechanics”)
- (Salih N. Neftci, *Principles of Financial Engineering*, “DV01 and PV01”)
- (Edwin J. Elton et al., *Modern Portfolio Theory and Investment Analysis*, “Special Considerations in Bond Pricing”)
- (Treasury Market Practices Group (TMPG), *U.S. Treasury Securities Fails Charge Trading Practice*, revised 2018-04-23)
