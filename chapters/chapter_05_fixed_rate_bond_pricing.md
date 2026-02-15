# Chapter 5: Fixed-Rate Bond Pricing

---

## Introduction

Since Treasury bonds promise future cash flows, discount factors can be extracted from Treasury bond prices.

Treasury bond prices in the United States are quoted in dollars and thirty-seconds (e.g., `120-05` means $120 + 5/32$). The quoted (clean) price is not the cash paid at settlement: the cash (dirty) price equals the quoted price plus accrued interest since the last coupon date.

This chapter builds a quote → cashflows → discount factors → invoice workflow, and adds a first look at rate risk using DV01 with an explicit bump object, units, and sign convention.

The chapter covers:

1. **Cashflow schedule construction** — How to specify the amounts and dates of a bond's promised payments
2. **Present value using discount factors** — Pricing bonds using a discount curve as input
3. **Clean vs dirty price mechanics** — The market convention that separates "quoted" from "exchanged"
4. **Accrued interest calculation** — How the next coupon gets split between buyer and seller, including day count comparisons
5. **Treasury price quotation in 32nds** — Converting market quotes to decimal prices, plus tick value and P&L calculations
6. **Price sensitivity intuition** — Why bond prices move when rates change (a preview for Chapter 11)
7. **Edge cases and exceptions** — Stub periods, settlement fails, and flat trading

Prerequisites: [Chapter 1](chapters/chapter_01_market_quoting_calendars_cashflow_plumbing.md), [Chapter 2](chapters/chapter_02_time_value_discount_factors_replication.md), [Chapter 3](chapters/chapter_03_zero_forward_par_rates_triangle.md), [Chapter 4](chapters/chapter_04_money_market_building_blocks.md)

Follow-on: [Chapter 6](chapters/chapter_06_ytm_yield_based_risk.md), [Chapter 11](chapters/chapter_11_dv01_pv01_definitions_computation.md), [Chapter 12](chapters/chapter_12_duration_macaulay_modified_dv01.md), [Chapter 23](chapters/chapter_23_treasury_futures.md)

## Learning Objectives
- After this chapter, you can translate a fixed-rate bond quote into cash settlement (invoice amount).
- You can go from quote → cashflows → present value using discount factors.
- You can compute accrued interest under Actual/Actual (in period) and 30/360 and explain clean vs dirty pricing.
- You can compute and interpret DV01 with an explicit bump object, units, and sign convention.
- You can apply quick checks to avoid common implementation errors (32nds conversion, day count, clean/dirty).

---

## 5.1 Fixed-Rate Bond Cashflows

### 5.1.1 The Bond as a Cashflow Schedule

A fixed-rate bond promises a deterministic stream of payments. Tuckman describes the structure simply: a bond with face value $F$, coupon rate $c$, and semiannual payments promises coupon payments of $F \times c/2$ every six months, plus a principal repayment of $F$ at maturity. There is no uncertainty about the amounts or timing—only about what those future dollars are worth today.

More formally, let:
- $F$ = face value (principal), typically 100 "per 100 par"
- $c$ = annual coupon rate (as a decimal)
- $m$ = coupon payments per year (typically $m=2$ for semiannual)
- $T_1, T_2, \ldots, T_N$ = future payment dates after settlement

The coupon payment each period is:

$$\boxed{\text{Cpn} = \frac{Fc}{m}}$$

And the cashflow schedule is:

$$\text{CF}_i = \begin{cases}
\text{Cpn}, & i = 1, \ldots, N-1 \\
\text{Cpn} + F, & i = N
\end{cases}$$

The final cashflow bundles the last coupon with the principal repayment.

**Worked Example A:** If $F = 100$ and $c = 5.50\%$ with $m = 2$, then $\text{Cpn} = 100 \times 0.055 / 2 = 2.75$. This represents $2.75 payable on each coupon date for every $100 face value.

### 5.1.2 Zero-Coupon Bonds as a Limiting Case

Setting $c = 0$ produces a zero-coupon bond: all intermediate cashflows vanish, leaving only the principal at maturity. This is useful as both a conceptual building block and a sanity check for any pricing implementation: the price should be just the discounted principal.

### 5.1.3 From Schedule to Value

> **Analogy: The Bag of Zeroes**
>
> A coupon bond is not a single instrument. It is just a "stapler" holding together a bag of different zero-coupon bonds.
> *   **$F=100$, 5% Coupon, 10-Year Bond** =
>     *   20 small coupons (each is a $2.50 zero-coupon bond).
>     *   1 large principal repayment (a $100 zero-coupon bond).
>
> **The Insight**: You don't price "The Bond." You price the 21 individual zeroes using their specific discount factors, then add them up. A bond is just a portfolio.

Once the cashflow schedule is specified, pricing reduces to a single question: what is the present value of these future payments? The answer depends on the discount curve, which brings us to the law of one price.

---

## 5.2 Present Value Using Discount Factors

### 5.2.1 The Law of One Price

Chapter 2 introduced the discount factor $P(0,t)$ as "the value today of receiving one unit of currency at time $t$." Tuckman emphasizes that the **law of one price** allows "discount factors extracted from one set of bonds... to price any other bond with cash flows on the same set of dates."

This means we can value a bond by decomposing it into its component cashflows and pricing each separately. The total price is simply the sum of the present values of all future cashflows:

$$\boxed{P_{\text{dirty}} = \sum_{i=1}^{N} \text{CF}_i \cdot P(0, t_i)}$$

where $t_i$ is the time (in years) from the valuation date to payment date $T_i$.

### 5.2.2 Curve-Based vs Yield-Based Pricing

A useful distinction is **curve-based** vs **yield-based** pricing:
- **Curve-based:** discount each cashflow using the discount factor (or zero rate) for its specific payment date.
- **Yield-based (YTM):** use a single internal rate of return that reprices the bond.

The formula above uses the **curve-based approach**—each cashflow is discounted at its own rate. The alternative, yield-to-maturity pricing (covered in Chapter 6), forces a single rate on all cashflows. While yield-based pricing is convenient for quoting, curve-based pricing is more accurate and is what arbitrage-free pricing models use.

You can think of a bond's yield as a blend of spot rates: it is a weighted average chosen to reproduce the bond's price, not a fundamental curve primitive. In this book we treat curve-based PV as the primary object and treat yield as a convenient summary.

**When do they differ?** If the yield curve is flat (all spot rates equal), both methods give identical prices. When the curve is upward-sloping, the single-yield method uses a rate that is "too high" for near-term cashflows and "too low" for distant cashflows. The numerical difference is usually small (a few cents per 100 face value) but can matter for spread analysis and relative value.

### 5.2.3 Worked Example B: Curve-Based vs Yield-Based Pricing

Consider a 2-year bond with a 5% annual coupon (paid semiannually) and face value 100. Assume an upward-sloping discount curve:

| Maturity | Zero Rate | Discount Factor |
|----------|-----------|-----------------|
| 0.5 yr | 4.00% | 0.9802 |
| 1.0 yr | 4.25% | 0.9583 |
| 1.5 yr | 4.50% | 0.9344 |
| 2.0 yr | 4.75% | 0.9091 |

**Curve-Based Pricing:**
$$P_{\text{dirty}} = 2.5(0.9802) + 2.5(0.9583) + 2.5(0.9344) + 102.5(0.9091)$$
$$P_{\text{dirty}} = 2.4505 + 2.3958 + 2.3360 + 93.1828 = 100.3651$$

**Yield-Based Pricing:**
The YTM that reprices this bond is approximately 4.70% (solved numerically). Using this single rate:
$$P_{\text{yield}} = \frac{2.5}{1.0235} + \frac{2.5}{1.0235^2} + \frac{2.5}{1.0235^3} + \frac{102.5}{1.0235^4} = 100.3651$$

In this case, the two methods agree because the YTM was solved to match the curve price. But if you *start* with the YTM and use it to price a different bond with the same maturity but different cashflow timing, you'll get a slightly different result than curve-based pricing.

**Key insight:** The curve-based price is the "correct" arbitrage-free price. The YTM is a convenient summary, but it is not a discount rate—it is an internal rate of return.

### 5.2.4 Why "Dirty" Price?

The formula above produces what traders call the **dirty price** (also called **full price**, **invoice price**, or **cash price**). This is the actual amount paid for the bond at settlement. It represents the comprehensive economic value of the remaining cashflows. Tuckman states clearly: "the money paid by the buyer and received by the seller" is the invoice price.

---

## 5.3 Premium, Discount, and Par Bonds

### 5.3.1 The Relationship Between Coupon and Price

A fundamental relationship connects the bond's coupon rate to its price relative to par:

| Condition | Bond Type | Price |
|-----------|-----------|-------|
| Coupon > Market Yield | **Premium** | $P > 100$ |
| Coupon = Market Yield | **Par** | $P = 100$ |
| Coupon < Market Yield | **Discount** | $P < 100$ |

**Why this holds:** If a bond's coupon is higher than current market rates, investors are willing to pay more than $100 for the stream of above-market payments. Conversely, if the coupon is below market rates, the bond must sell below $100 to compensate buyers for the sub-market cashflows.

Hull defines the **par yield** as "the coupon rate that causes the bond price to equal its par value." When the coupon rate exactly matches the par yield for that maturity, the bond trades at exactly 100.

### 5.3.2 Par Bond Sanity Check

This provides a critical sanity check for any pricing implementation. If you input:
- A bond with coupon $c$
- A flat yield curve at rate $c$

Then the computed dirty price (at a coupon date, so AI = 0) should be exactly 100. If it's not, there's a bug.

> **Implementation Note:** This check catches numerous errors: wrong compounding conventions, off-by-one errors in cashflow schedules, incorrect day count fractions, and more.

### 5.3.3 Visualizing "Pull to Par"

> **The Gravity of Par**
>
> If yields remain constant, a bond's price will naturally drift toward 100 as it approaches maturity.
>
> ```
> Premium Bond (105) ───────────────> 100 (Par)
>                      \           /
> Discount Bond (95)  ────────────> 100 (Par)
> ```
>
> *   **Premium Bond**: You paid extra upfront. You "lose" slightly every day as price falls to 100. This loss offsets the high coupon.
> *   **Discount Bond**: You paid less upfront. You "gain" slightly every day as price rises to 100. This gain supplements the low coupon.
>
> **Result (with a caveat):** Under a constant-yield assumption (and reinvestment at that yield), total return is approximately the yield regardless of coupon—the price “pull to par” offsets coupon differences. Chapter 6 explains why realized return can still differ when yields move or reinvestment rates change.

### 5.3.4 Price Trajectory: Pull to Par Over Time

Consider a 5% coupon bond when market yields are constant at 4% (so the bond trades at a premium). As the bond approaches maturity, its price converges to par:

| Years to Maturity | Price (@ 4% YTM) | Price Change |
|-------------------|------------------|--------------|
| 5.0 | 104.49 | — |
| 4.0 | 103.66 | -0.83 |
| 3.0 | 102.80 | -0.86 |
| 2.0 | 101.90 | -0.90 |
| 1.0 | 100.97 | -0.93 |
| 0.0 | 100.00 | -0.97 |

The price decline accelerates as maturity approaches because the "extra" coupon payments become fewer. For discount bonds, the pattern is reversed—price rises accelerate toward maturity.

> **Practitioner Note:** For accounting purposes, buy-and-hold investors amortize the premium or accrete the discount over the bond's remaining life. This premium amortization reduces reported interest income; discount accretion increases it. The formulas are beyond this chapter's scope, but the economic intuition is the pull-to-par effect.

---

## 5.4 Clean Price, Dirty Price, and Accrued Interest

### 5.4.1 The Problem Clean Pricing Solves

Elton et al. emphasize that the **quoted price** at which a bond is bought or sold is not the price the customer pays or receives: the settlement amount is the quoted price **plus accrued interest**. They also note a useful consequence of quoting this way: when a bond makes an interest payment, accrued interest becomes zero but the quoted price remains unchanged (assuming yields do not change).

### 5.4.2 The Market Convention

In Hull’s U.S. Treasury quoting convention, traders call the quoted price the **clean price** and the cash settlement amount the **dirty price**. In general:

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + \text{AI}}$$

Where:
- $P_{\text{clean}}$ is the quoted (flat) price per $100$ face
- $\text{AI}$ is accrued interest since the last coupon date (computed to settlement)
- $P_{\text{dirty}}$ is the invoice / cash / full price actually exchanged at settlement

### 5.4.3 Why This Convention Works

Tuckman writes the relationship between flat (quoted) price, accrued interest, and present value as:

$$\boxed{P + AI = PV(\text{future cash flows})}$$

where $P$ is the quoted (flat/clean) price.

The particular market convention used in calculating accrued interest does not really matter: if the convention in place is too generous to the seller, the flat price adjusts downward to offset it. Put differently, the economically relevant quantity is the **invoice price** (the money that changes hands), and the market sets that invoice price equal to the present value of the future cash flows.

### 5.4.4 Algebraic Proof of Clean Price Continuity

Tuckman shows that **if yield does not change then the quoted price of a bond does not fall as a result of a coupon payment**. The algebra is:

Let $P^b$ and $P^a$ be the quoted prices immediately before and after a coupon payment of $c/2$. Right before the coupon date:
- Accrued interest equals the full coupon: $\text{AI} = c/2$
- The present value of the next coupon equals $c/2$

Invoking the identity $P+AI=PV(\text{future cash flows})$:

$$P^b + c/2 = c/2 + PV(\text{cashflows after the next coupon})$$

Simplifying:

$$P^b = PV(\text{cashflows after the next coupon})$$

Right after the coupon is paid, accrued interest resets to zero:
$P^a + 0 = PV(\text{cashflows after the next coupon})$

Therefore $P^a = P^b$—the clean price is continuous across coupon dates.

> **P&L Perspective:** When traders mark a bond position to market, they track the change in *clean* price to measure performance. The coupon payments and accrued interest are handled as separate cash accounting entries. This isolates "price risk" (rates moving) from "time" (accrual).

### 5.4.5 Example: Clean Price Continuity

To illustrate the sawtooth removal, consider a bond around a coupon date. Assume market yields are unchanged, so the clean price remains steady at 101.14.

| Date | Days Elapsed | Accrued Interest (AI) | Dirty Price | Note |
|------|--------------|-----------------------|-------------|------|
| Jul 30 | 180 (of 181) | $2.73$ | $103.87$ | Full coupon almost accrued |
| **Jul 31** | **Coupon Paid** | **Reset to 0** | **101.14** | **Dirty price drops by coupon** |
| Aug 01 | 1 (of 184) | $0.015$ | $101.155$ | Accrual starts again |

The *dirty* price drops from 103.87 to 101.14 (the coupon amount is paid out). The *clean* price stays at 101.14. This stability allows traders to compare prices easily across time.

---

## 5.5 Accrued Interest Mechanics

### 5.5.1 Economic Purpose

Accrued interest compensates the seller for the portion of the coupon period during which they held the bond. In Tuckman's 5 1/2s example: if the bond makes a coupon payment on July 31 but the seller sells on February 15, the seller should receive compensation for having held the bond from the prior coupon date (January 31) to settlement. The buyer pays this amount at settlement, then collects the full coupon when it's paid.

> **Analogy: The Taxi Meter**
>
> Think of accrued interest as a taxi meter that starts at zero on the coupon date and ticks upward every day. When you buy the bond, you pay the seller for how far the meter has run. Then you ride to the next coupon date and collect the full fare.

### 5.5.2 Actual/Actual Day Count (U.S. Treasuries)

For U.S. Treasury bonds, accrued interest is calculated using the **Actual/Actual (in period)** day count convention.

Let:
- $d_{\text{elapsed}}$ = actual number of days from the previous coupon date to the settlement date
- $d_{\text{period}}$ = actual number of days in the full coupon period

The accrued interest is the pro-rated share of the full coupon:

$$\boxed{\text{AI}_{\text{Act/Act}} = \frac{d_{\text{elapsed}}}{d_{\text{period}}} \times \text{Cpn}}$$

### 5.5.3 Worked Example C: Tuckman's 5 1/2s of January 31, 2003

This is Tuckman's canonical example, which we reproduce exactly:

**Scenario:**
- **Bond:** 5 1/2s of January 31, 2003
- **Coupon:** 5.50% ($2.75 semiannual payment per $100 face, or $275 per $10,000 face)
- **Settlement Date:** February 15, 2001
- **Previous Coupon:** January 31, 2001
- **Next Coupon:** July 31, 2001

**Calculation:**
1. **Days in period ($d_{\text{period}}$):** From January 31 to July 31 is 181 days.
2. **Days elapsed ($d_{\text{elapsed}}$):** From January 31 to February 15 is 15 days.
3. **Accrual Fraction:** $15 / 181 = 0.08287$.
4. **Accrued Interest per $100 face:**
   $$\text{AI} = \frac{15}{181} \times 2.75 = 0.2279$$
5. **Accrued Interest per $10,000 face:** $0.2279 \times 100 = \$22.79$

As Tuckman states: the buyer pays "$22.79 of accrued interest" and "having paid this $22.79 of accrued interest, investor B may keep the entire $275 coupon payment of July 31, 2001."

### 5.5.4 The 30/360 Day Count (U.S. Corporates)

Corporate and municipal bonds in the United States typically use the **30/360** day count convention, which assumes every month has 30 days and every year has 360 days.

The general formula for counting days between dates $(Y_1, M_1, D_1)$ and $(Y_2, M_2, D_2)$ is:

$$\boxed{\text{Days}_{30/360} = 360(Y_2 - Y_1) + 30(M_2 - M_1) + (D_2 - D_1)}$$

with standard adjustments at month boundaries (if $D_1 = 31$, set $D_1 = 30$; similar for $D_2$ depending on $D_1$).

The accrued interest under 30/360 is:

Let:
- $d_{\text{elapsed}}^{30/360}$ = 30/360 day count from last coupon date to settlement
- $d_{\text{period}}^{30/360}$ = 30/360 day count for the full coupon period (for a regular semiannual bond, $d_{\text{period}}^{30/360} = 180$)

Then (per $100$ face value):

$$\boxed{\text{AI}_{30/360} = \frac{d_{\text{elapsed}}^{30/360}}{d_{\text{period}}^{30/360}} \times \text{Cpn}}$$

where $\text{Cpn} = 100 \times c / 2$ for a semiannual coupon bond.

### 5.5.5 Day Count Comparison: Act/Act vs 30/360

Hull's examples highlight how Actual/Actual (in period) and 30/360 can produce different accrued interest, especially around month boundaries.

**Example: March 1 to July 3, 8% annual coupon ($4 semiannual)**

| Convention | Days Elapsed | Days in Period | Accrued Interest |
|------------|--------------|----------------|------------------|
| **Actual/Actual** | 124 actual days | 184 actual days | $(124/184) \times 4 = 2.6957$ |
| **30/360** | 122 days | 180 days | $(122/180) \times 4 = 2.7111$ |

The 30/360 convention produces slightly more accrued interest in this case.

**But the differences can be dramatic at month boundaries:**

| Convention | Days Between Feb 28 and Mar 1 |
|------------|-------------------------------|
| **Actual/Actual** | 1 day |
| **30/360** | 3 days (Feb 28 → Feb 30 → Mar 1) |

All else equal, the corporate bond accrues about three times as much interest over that interval under 30/360.

### 5.5.6 Worked Example D: Same Dates, Different Conventions

**Scenario:** A bond with 6% annual coupon, settlement March 15, previous coupon December 15.

**Actual/Actual Calculation:**
- Days from Dec 15 to Mar 15: 90 days (actual)
- Coupon period Dec 15 to Jun 15: 182 days
- AI = $(90/182) \times 3.00 = 1.4835$

**30/360 Calculation:**
- Days from Dec 15 to Mar 15: $30 \times 3 + 0 = 90$ days
- Denominator: 180 (always, for semiannual)
- AI = $(90/180) \times 3.00 = 1.5000$

**Difference:** The 30/360 convention produces 1.65 cents more AI per $100 face. On $100 million notional, this is $16,500—material for settlement purposes.

> **Desk Reality: Day Count Mismatch P&L Breaks**
>
> A common source of unexplained P&L: your risk system uses one day count while the settlement system uses another. This is especially likely during system migrations or when trading instruments across markets (e.g., hedging a corporate bond with Treasury futures).
>
> **How to detect:** AI calculated by front-office differs from operations; daily P&L doesn't match position changes.
>
> **The fix:** Always verify which convention the instrument requires before building the trade. Treasury = Act/Act. Corporate = 30/360. Swaps = varies (check the confirm).

---

## 5.6 Treasury Price Quotation in 32nds

### 5.6.1 The Quote Format

In the U.S. Treasury market, prices are quoted in **32nds** of a dollar rather than decimals. Numbers after the hyphen denote 32nds (often called *ticks*).

A quote consists of the full dollar amount (the "handle") followed by the number of 32nds:

- A quote of "**99-16**" means $99 + 16/32 = 99.50$
- A quote of "**101-04**" means $101 + 4/32 = 101.125$

For finer precision, markets use **half-ticks** (indicated by "+") representing 1/64th:

- "**108-31+**" means $108 + 31/32 + 1/64 = 108 + 31.5/32 = 108.984375$

> **Pitfall — Treasury 32nds quoting:** interpreting `120-05` as `120.05`.
> **Why it matters:** the mistake is about $0.10625 per $100 face; that's $106,250 per $100mm notional.
> **Quick check:** convert `XX-YY` to `XX + YY/32` (and `+` adds $1/64$) before you do PV, risk, or P&L.

### 5.6.2 Worked Example E: From Quote to Invoice Price

**Inputs:**
- **Bond:** 5 1/2s of January 31, 2003
- **Quote:** "101-04 5/8" (which is 101-4.625)
- **Accrued Interest:** $0.2279$ per $100 face (from previous example)
- **Face Amount:** $10,000

**Step 1: Convert Quote to Decimal Clean Price**
$$P_{\text{clean}} = 101 + \frac{4.625}{32} = 101 + 0.14453125 = 101.14453125$$

**Step 2: Add Accrued Interest to Get Dirty Price**
$$P_{\text{dirty}} = 101.14453125 + 0.2279 = 101.37243125$$

**Step 3: Calculate Invoice Amount**
For $10,000 face value (which is 100 "units" of $100 par):
$$\text{Invoice} = 100 \times 101.37243125 = \$10,137.24$$

In this example, the invoice cash amount is $10,137.24.

### 5.6.3 Tick Value and P&L Calculation

Every trader needs to know how much money a tick move represents. The **tick value** is the dollar change in position value for a one-tick (1/32nd) price move:

$$\boxed{\text{Tick Value} = \frac{\text{Notional}}{100} \times \frac{1}{32}}$$

**For $100 million notional:**
$$\text{Tick Value} = \frac{\$100,000,000}{100} \times \frac{1}{32} = \$1,000,000 \times 0.03125 = \$31,250$$

> **Desk Reality: The Handle Game**
>
> Traders often speak in shorthand that assumes you know the "handle" (the integer part of the price):
>
> | Trader Says | Full Quote | Means |
> |-------------|------------|-------|
> | "16" | 99-16 | 99.50 (if handle is known to be 99) |
> | "16-plus" or "16+" | 99-16+ | 99.515625 |
> | "bid at the handle" | 100-00 | 100.00 exactly |
> | "half" | XX-16 | The ticks part = 16 (since 16/32 = 0.5) |
>
> **Defending the handle:** When a bond's price is just above a round number (e.g., 100-02), traders watch whether it will "break the handle" and trade below 100. This psychological level can act as support/resistance.

### 5.6.4 Worked Example F: P&L Calculation

**Scenario:** You're long $50 million face value of Treasuries at 99-16. The price moves to 99-20.

**Price Change:**
- Entry: 99-16 = 99.50
- Exit: 99-20 = 99.625
- Change: +4 ticks

**P&L Calculation:**
$$\text{Tick Value}_{50mm} = \frac{\$50,000,000}{100} \times \frac{1}{32} = \$15,625$$

$$\text{P\&L} = 4 \text{ ticks} \times \$15,625 = \$62,500$$

**Quick Reference Table: P&L per Tick by Notional**

| Notional | Tick Value | 5-Tick Move |
|----------|------------|-------------|
| $10mm | $3,125 | $15,625 |
| $50mm | $15,625 | $78,125 |
| $100mm | $31,250 | $156,250 |
| $500mm | $156,250 | $781,250 |

> **Practitioner Note:** Electronic trading platforms sometimes quote in 1/64ths (half-ticks) or even 1/128ths (quarter-ticks) for highly liquid on-the-run issues. The tick value scales accordingly: a 1/64th on $100mm is $15,625.

---

## 5.7 Price Sensitivity to Rate Changes (Preview)

### 5.7.1 The Mechanism

Since a bond's dirty price is the sum of discounted cashflows, any change in the discount curve immediately affects the price:
- If **rates rise**, discount factors fall ($1/(1+r)^t$ decreases). **Price falls.**
- If **rates fall**, discount factors rise. **Price rises.**

This inverse relationship is the most fundamental property of fixed-income risk.

### 5.7.2 Why Longer Bonds Are More Sensitive

Cashflows further in the future are more sensitive to rate changes than near-term cashflows. This is because the discounting effect compounds over time.

**Sensitivity Check:**
Consider two cashflows, each of $100:
1. Payment in 6 months ($t=0.5$)
2. Payment in 10 years ($t=10$)

If rates rise by 1% (100 basis points):
- The 6-month discount factor drops slightly (e.g., from 0.975 to 0.970), a ~0.5% loss in PV.
- The 10-year discount factor drops substantially (e.g., from 0.60 to 0.55), a ~8% loss in PV.

Because the final repayment of principal occurs at maturity ($T_N$), the price of a long-term bond is dominated by this highly sensitive distant cashflow. This is why "long duration" bonds fall harder when rates rise. We formalize this as **DV01** (Dollar Value of a 01) and **Duration** in Chapters 11-12.

### 5.7.3 DV01 (One-Number Rate Risk) — Define the Bump

DV01 is short for "dollar value of a 01" (one basis point). If $\Delta B$ is the change in bond price for a yield change $\Delta y$, Neftci defines:

$$\boxed{DV01 = -\frac{\Delta B}{10{,}000 \times \Delta y}}$$

Tuckman gives the corresponding derivative form:

$$\boxed{DV01 = -\frac{1}{10{,}000}\frac{d P(y)}{d y}}$$

The minus sign is there because bond prices typically fall when yields rise; DV01 is usually reported as a positive number for a long position.

**Bump object matters.** You must state what is being shifted (a bond’s yield, a zero curve, a set of par instruments, etc.), because different “1 bp” bumps can produce different numbers.

**This chapter’s bump object:** a *parallel* $-1$ bp shift to the **discounting zero curve** used to generate the discount factors $P(0,t_i)$, holding the bond’s cashflows fixed.

With the book-wide sign convention (positive for a long bond when rates fall):

$$\boxed{DV01 := PV(\text{rates down }1\text{bp}) - PV(\text{base})}$$

For a curve-based PV computed from discount factors, that definition becomes a simple “bump-and-reprice” difference:

$$\boxed{DV01 = \sum_{i=1}^{N} CF_i\,P_{\downarrow 1\text{bp}}(0,t_i) \;-\; \sum_{i=1}^{N} CF_i\,P(0,t_i)}$$

**Scaling:** if PV is expressed per $100$ face, multiply by $N/100$ to convert to a notional-$N$ dollar DV01.

**Worked Example (House Template): Quote → Invoice → PV → DV01 (1-Year Note)**

**Context**
- You are buying $50mm face of a coupon-bearing Treasury note. You need the invoice cash amount, the PV (dirty price), and a one-number rate risk.

**Timeline (dates are simplified; ignore business-day adjustments)**
- Trade date: 2026-02-13
- Settlement date: 2026-02-16
- Last coupon date: 2026-01-15
- Next coupon date: 2026-07-15
- Maturity date: 2027-01-15

**Inputs**
- Notional: $N = \\$50{,}000{,}000$
- Coupon: $c = 4.00\\%$ per year, semiannual
- Clean quote (Treasury 32nds): `101-12`
- Accrued interest day count: Actual/Actual (in period)
  - $d_{\\text{elapsed}} = 32$ days, $d_{\\text{period}} = 181$ days
- Discount factors (from the discounting curve; illustrative): $P(0,\\text{2026-07-15})=0.9900$, $P(0,\\text{2027-01-15})=0.977927$
- Discount factors after a $-1$ bp parallel shift (illustrative): $P_{\\downarrow 1\\text{bp}}(0,\\text{2026-07-15})=0.9900404$, $P_{\\downarrow 1\\text{bp}}(0,\\text{2027-01-15})=0.9780162$
- DV01 bump object: parallel $-1$ bp shift of the discounting zero curve; $DV01 := PV(\\text{rates down }1\\text{bp})-PV(\\text{base})$

**Outputs (what you produce)**
- $P_{\\text{clean}}$ and $P_{\\text{dirty}}$ (per $100$ face)
- Invoice cash amount (USD)
- $DV01$ (USD per 1bp)

**Step-by-step**
1. **Quote → decimal clean price**
   $$P_{\\text{clean}} = 101 + \\frac{12}{32} = 101.3750$$
2. **Coupon per period (per $100$ face)**
   $$\\text{Cpn} = 100\\times \\frac{c}{2} = 2.00$$
3. **Accrued interest to settlement (per $100$ face)**
   $$AI = \\frac{32}{181}\\times 2.00 = 0.3536$$
4. **Dirty price and invoice cash amount**
   $$P_{\\text{dirty}} = P_{\\text{clean}} + AI = 101.3750 + 0.3536 = 101.7286$$
   $$\\text{Invoice} = \\frac{N}{100}\\times P_{\\text{dirty}} = 500{,}000\\times 101.7286 = \\$50{,}864{,}300$$
5. **PV from discount factors (per $100$ face)**
   - Cashflows: $CF_1=2.00$ (2026-07-15), $CF_2=102.00$ (2027-01-15)
   $$PV = 2.00(0.9900) + 102.00(0.977927) \\approx 101.7286$$
6. **DV01 (per $100$ face, then scaled to notional)**
   $$DV01_{\\text{per }100} = \\left[2.00(0.9900404)+102.00(0.9780162)\\right]-\\left[2.00(0.9900)+102.00(0.977927)\\right]\\approx 0.00918$$
   $$DV01_{\\$50mm} \\approx 0.00918\\times 500{,}000 = \\$4{,}591 \\text{ per bp}$$

**Cashflows (per $100$ face)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-07-15 | 2.00 | coupon |
| 2027-01-15 | 102.00 | coupon + principal |

**P&L / Risk Interpretation**
- If the discounting curve shifts down by 1 bp (our bump object), the position's PV increases by about $\\$4.6k$; a +1 bp move decreases PV by about $\\$4.6k$.
- Clean-vs-dirty matters for *cash settlement* (invoice) and for *daily P&L explain* (clean price moves vs accrued interest carry).

**Sanity Checks**
- $0 \\le AI < \\text{Cpn}$: $0.3536 < 2.00$.
- Units: prices are per $100$; invoice scales by $N/100$; DV01 is USD per 1bp for the stated bump object.
- Sign: with the book convention, a long bond has $DV01>0$.

---

## 5.8 Stub Periods and Odd First Coupons

### 5.8.1 What Is a Stub Period?

A **stub period** is a coupon period that is shorter or longer than the standard 6 months. This typically occurs with the first coupon of a newly issued bond when the auction date doesn't align with the regular coupon schedule.

**Types of stubs:**
- **Short first coupon**: First period < 6 months. First coupon is less than $c/2$.
- **Long first coupon**: First period > 6 months. First coupon is more than $c/2$.

### 5.8.2 Why Stubs Occur

Treasury bonds are issued according to a set auction calendar, but the coupon dates are set to regular schedules (e.g., February 15 / August 15). If a bond is issued on March 1 with coupons on August 15 / February 15, the first period (March 1 to August 15) is only about 5.5 months—a short stub.

### 5.8.3 Pricing Bonds with Stub Periods

For a stub period, the coupon payment is pro-rated based on the actual length of the stub relative to a regular period:

$$\text{Stub Coupon} = \text{Regular Coupon} \times \frac{\text{Stub Days}}{\text{Regular Period Days}}$$

The accrued interest calculation also uses the stub period length.

### 5.8.4 Worked Example G: Short First Coupon

**Scenario:** A 5% semiannual bond is issued March 1, with regular coupon dates of August 15 and February 15. Settlement is March 15.

**Stub Analysis:**
- Issue date: March 1
- First coupon: August 15 (167 days from March 1)
- Regular period would be: 181 days (Feb 15 to Aug 15)
- Stub fraction: $167/181 = 0.923$

**First Coupon Amount:**
$$\text{First Cpn} = 2.50 \times 0.923 = 2.3066$$

**Accrued Interest (Settlement March 15):**
- Days from March 1 to March 15: 14 days
- Days in stub period (March 1 to Aug 15): 167 days
- AI = $(14/167) \times 2.3066 = 0.1934$

**The Pricing Twist:** When pricing this bond, the first cashflow is $2.3066$, not $2.50$. The discount factor exponent must also reflect the stub: if settlement is 153 days (March 15 to Aug 15) before the first coupon, use $153/181 \approx 0.845$ of a period, not $1.0$.

> **Desk Reality: The New Issue Stub Trap**
>
> **The mistake:** Using the standard coupon amount for the first period of a new issue.
>
> **How to detect:** First accrual period days ≠ 181-184 days.
>
> **Why it matters:** On $100mm notional, the difference between a full $2.50 coupon and a short-stub $2.30 coupon is $20,000. If your system assumes a full first coupon, your P&L will show a "loss" when the actual first coupon pays.

---

## 5.9 Settlement Exceptions: Fails and Flat Trading

### 5.9.1 What Is a Settlement Fail?

A **settlement fail** occurs when the seller does not deliver securities to the buyer by the settlement date. The trade remains "open" until delivery occurs. Importantly, the economic exposure continues—the buyer is still owed the securities, and the seller is still owed the cash.

### 5.9.2 Why Fails Happen

One important driver is *financing mechanics*: when a trader is short an on-the-run Treasury and needs to borrow it via repo, the trade can fail if the security cannot be borrowed in time to make delivery.

### 5.9.3 The Funding Intuition: Why Fails Matter

Tuckman gives a simple financing interpretation in the context of specials. If the bond cannot be borrowed, the trader fails to deliver and does not receive the proceeds from the sale. In effect, the trader loses (at least) a day of interest on those proceeds.

If the bond *can* be borrowed, the trader can deliver, receive the proceeds, and lend them at the special repo rate. When the special repo rate is near $0\\%$, earning $0\\%$ on proceeds is economically similar to failing to deliver.

> **Desk Reality:** Fails are not just an ops nuisance; they change the *cash timeline*.
> **Common break:** P&L explain shows an unexpected funding/carry gap around settlement when deliveries slip.
> **What to check:** whether the position is financing at GC vs specials, and whether any deliveries failed.

### 5.9.4 Flat Trading (Defaulted Bonds)

There are exceptions to the rule that the bond buyer pays the bond seller accrued interest. One important exception is when the issuer has not fulfilled its promise to make periodic interest payments (default). In such instances, the bond is sold **without accrued interest** and is said to trade **flat**.

$$\text{Flat Trading: } P_{\text{dirty}} = P_{\text{clean}} \text{ (AI = 0)}$$

> **Practitioner Note:** The transition from accruing to flat can cause confusion in systems. If a bond was marked with accrued interest yesterday and today it trades flat, the drop in accrued is not a “real” funding P&L—it reflects a change in the convention used to quote/settle the bond.

---

## 5.10 Practical Notes and Verification

### 5.10.1 Settlement and Accrued

Accrued interest is always computed to the **settlement date**, not the trade date.

Settlement lag varies by instrument and venue (and can change over time). Always check the trade confirmation/market convention for the instrument you are pricing.

If the settlement date lands on a Monday because of a weekend, accrued interest includes Friday, Saturday, and Sunday (the seller "earns" interest for holding through the weekend).

### 5.10.2 Common Implementation Pitfalls

| Pitfall | Consequence |
|---------|-------------|
| **Wrong Day Count** | Using 30/360 for a Treasury (or vice versa) results in incorrect AI and incorrect dirty price. |
| **Clean/Dirty Confusion** | Settling based on the clean price ignores the AI payment, causing a P&L break. |
| **Stub Periods** | The first or last coupon period may be "short" or "long." Standard formulas fail here; you must generate the schedule dates explicitly. |
| **32nds Handling** | Excel and programming languages use decimals. Manually entering "101.16" for "101-16" is a common data entry error (101.5 vs 101.16). |
| **Off-by-One in Schedules** | Miscounting the number of remaining cashflows leads to systematic pricing errors. |
| **Settlement Lag Error** | Computing accrued to trade date instead of settlement date. |

### 5.10.3 Test Your Implementation: Verification Checklist

Any pricing engine should pass these checks:

| Check | What to Verify | Expected Result |
|-------|----------------|-----------------|
| **Zero Coupon** | If coupon $c=0$ | Price = $F \times P(0, T_N)$ |
| **Par Check** | Coupon = par yield, at coupon date (AI=0) | Dirty price = 100.00 |
| **Continuity** | Price across coupon date | Clean price continuous; dirty drops by coupon |
| **AI Bounds** | Accrued interest | $0 \le \text{AI} < \text{Coupon}$ |
| **AI Sums** | Daily AI over full period | Should sum to full coupon |
| **Premium/Discount** | Coupon > yield | Price > 100 |
| **Zero AI Bond** | 0% coupon bond | AI = 0 always |
| **32nds Conversion** | "100-16" | = 100.50 (not 100.16) |

---

## Summary

1. **A bond is a cashflow schedule.** It consists of known coupon payments ($Fc/m$) and a final principal payment ($F$).

2. **Value equals discounted cashflows.** The **law of one price** dictates that $P_{\text{dirty}} = \sum \text{CF}_i \times P(0, t_i)$.

3. **Curve-based pricing discounts cashflows date-by-date.** Use maturity-specific discount factors (or spot/zero rates) for each payment date. Yield-to-maturity is a one-number IRR summary, not a curve primitive.

4. **Markets quote clean, settle dirty.** The invoice price is the clean price plus accrued interest: $P_{\text{dirty}} = P_{\text{clean}} + \text{AI}$.

5. **Accrued interest is fair compensation.** It ensures the seller receives the interest earned during their holding period.

6. **Day counts matter.** Treasuries use Actual/Actual; corporates often use 30/360. The same dates can produce 3x different accrued interest at month boundaries.

7. **Treasuries quote in 32nds.** "101-04" means $101 + 4/32 = 101.125$. One tick on $100mm = $31,250.

8. **Premium bonds have coupon > yield; discount bonds have coupon < yield.** Par bonds have coupon = yield and trade at 100. All bonds "pull to par" over time.

9. **Rates risk needs an explicit bump definition.** Use DV01 with a stated bump object, units, and sign (in this book: $DV01 := PV(\\text{rates down }1\\text{bp})-PV(\\text{base})$).

10. **Edge cases require care.** Stub periods need explicit accrual logic; settlement fails change the cash timeline (and therefore funding/carry); defaulted bonds may trade flat.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Cashflow Schedule** | The series of future dates and amounts ($T_i, \text{CF}_i$) a bond promises to pay. | The starting point for all valuation. |
| **Discount Factor** | The value today of $1 at a future date $t$. | Converts future cashflows to present value. |
| **Dirty (Invoice) Price** | PV of all future cashflows. | The actual money transferred at settlement. |
| **Clean (Quoted) Price** | Dirty Price minus Accrued Interest. | Removes accrual drift for price comparison. |
| **Accrued Interest (AI)** | Pro-rata share of the next coupon earned since the last payment. | Compensates seller for holding time. |
| **Invoice Amount** | Cash settlement: $\text{Invoice}=(N/100)\,P_{\text{dirty}}$. | Prevents “quoted price = cash” mistakes. |
| **Actual/Actual** | Day count using actual calendar days. | Used for U.S. Treasuries. |
| **30/360** | Day count assuming 30-day months, 360-day year. | Used for U.S. corporates. |
| **Premium Bond** | Bond trading above par (coupon > yield). | Indicates above-market coupon payments. |
| **Discount Bond** | Bond trading below par (coupon < yield). | Indicates below-market coupon payments. |
| **32nds (Ticks)** | Treasury pricing unit ($1/32 \approx 0.03125$). | Historical convention for quoting. |
| **Tick Value** | Dollar P&L per 1/32 price move. | $31,250 per tick on $100mm. |
| **DV01** | $DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$ for a stated bump object. | A one-number rate risk; must define what is being bumped. |
| **Stub Period** | First or last coupon period that differs from standard length. | Requires pro-rated coupon calculation. |
| **Flat Trading** | Trading without accrued interest (AI = 0). | Applies to defaulted bonds. |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $N$ | Notional / face amount | currency (e.g., USD) |
| $F$ | Face value basis for quoting | usually $100$ for “per 100” quotes |
| $c$ | Annual coupon rate | per year (decimal) |
| $m$ | Coupon payments per year | typically $m=2$ |
| $T_i$ | Cashflow payment date | date |
| $t_i$ | Year-fraction from valuation/settle to $T_i$ | years; day count must be stated |
| $\tau(\cdot)$ | Year fraction between dates | years; depends on day count |
| $\text{Cpn}$ | Coupon per period | price points per $100$ face (for $F=100$: $\text{Cpn}=100c/m$) |
| $CF_i$ | Cashflow at $T_i$ | currency per $100$ face; receive-positive |
| $P(0,t)$ | Discount factor to time $t$ | unitless |
| $P_{\text{clean}}, P_{\text{dirty}}$ | Clean / dirty price | price per $100$ face |
| $AI$ | Accrued interest | price per $100$ face; $P_{\text{dirty}}=P_{\text{clean}}+AI$ |
| $PV$ | Present value | currency |
| $DV01$ | Dollar value of 1bp (book convention) | currency per 1bp; $PV(\text{rates down }1\text{bp})-PV(\text{base})$ for stated bump |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the fundamental formula for bond pricing? | $P_{\text{dirty}} = \sum \text{CF}_i \cdot P(0, t_i)$ |
| 2 | What is the relationship between clean and dirty price? | Dirty = Clean + Accrued Interest |
| 3 | Which price do traders usually quote? | Clean price |
| 4 | Which price is actually paid at settlement? | Dirty (Invoice) price |
| 5 | Why do markets use clean prices? | To avoid the "sawtooth" price drop when coupons are paid; makes prices comparable over time |
| 6 | Convert the Treasury quote "99-16" to decimal. | $99 + 16/32 = 99.50$ |
| 7 | Convert the Treasury quote "100-01+" to decimal. | $100 + 1.5/32 = 100.046875$ |
| 8 | What is the tick value for $100mm notional? | $31,250 per tick (1/32) |
| 9 | What is the formula for Accrued Interest (Actual/Actual)? | (Days Elapsed / Days in Period) × Coupon |
| 10 | How many days between Feb 28 and Mar 1 under 30/360? | 3 days |
| 11 | How many days between Feb 28 and Mar 1 under Actual/Actual? | 1 day |
| 12 | What is the "32nds trap"? | Confusing "101-16" (= 101.50) with "101.16" decimal |
| 13 | When does a bond trade at a premium? | When its coupon rate exceeds the market yield |
| 14 | When does a bond trade at par (price = 100)? | When its coupon rate equals the par yield |
| 15 | What does "flat trading" mean? | Bond trades without accrued interest (AI = 0), typically for defaulted bonds |
| 16 | What is a "stub period"? | A first or last coupon period that is shorter or longer than standard |
| 17 | What happens to clean price across a coupon payment date? | It is continuous (does not jump) |
| 18 | What happens to dirty price when a coupon is paid? | It drops by approximately the coupon amount |
| 19 | Define DV01 in this book. | $DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$ for a stated bump object; units = currency per 1bp. |
| 20 | How do you calculate P&L for a 4-tick move on $50mm? | 4 × ($50mm/100 × 1/32) = 4 × $15,625 = $62,500 |
| 21 | What is the key difference between curve-based and yield-based pricing? | Curve-based discounts each cashflow with its own discount factor; yield-based uses a single IRR (YTM). |

---

## Mini Problem Set

### Questions

**Problem 1 (Easy):** A Treasury is quoted at **98-24**. Convert to decimal.

**Problem 2 (Easy):** Calculate the semiannual coupon payment for a $10,000 face value bond with a 4.50% coupon.

**Problem 3 (Easy):** A bond is settled with 60 days elapsed since the last coupon. The full coupon period is 182 days. The semiannual coupon payment is $3.00 per $100 face. Calculate the Accrued Interest using Actual/Actual.

**Problem 4 (Medium):** Discount factors are: $P(0.5)=0.98$, $P(1.0)=0.96$. A bond pays $5 in 6 months and $105 in 1 year. Calculate the dirty price.

**Problem 5 (Medium):** Compare accrued interest using Actual/Actual vs 30/360 for a 6% annual coupon bond, with previous coupon March 1 and settlement April 15. Assume the coupon period is March 1 to September 1 (184 actual days).

**Problem 6 (Medium):** You're long $100 million face value of Treasuries. The price moves from 101-08 to 101-13. Calculate your P&L.

**Problem 7 (Medium):** A new-issue bond has a short first coupon period of 120 days (regular period = 180 days). If the regular semiannual coupon is $2.50, what is the first coupon payment?

**Problem 8 (Analysis):** Explain why a premium bond's price converges to par as maturity approaches, even if yields don't change.

**Problem 9 (Verification):** Your system shows accrued interest of $3.50 for a bond with a semiannual coupon of $3.00. What's wrong?

**Problem 10 (Application):** A Treasury sale fails to settle for 3 days, so you do not receive $50mm of proceeds until 3 days later. Using a 3.00% p.a. funding rate on an Act/360 basis, what is the approximate interest you missed?

**Problem 11 (Analysis):** Why can curve-based pricing differ from yield-based pricing when the yield curve is not flat?

**Problem 12 (Hard):** Verify the Tuckman example: 5 1/2s of Jan 31, 2003, quoted at 101-04 5/8 with AI = 0.2279 per $100 face. What is the dirty price? What is the invoice amount on $10,000 face?

---

### Solutions (Brief)

**1.** $98 + 24/32 = 98.75$

**2.** $10,000 \times 0.045 / 2 = \$225$

**3.** $\text{AI} = (60/182) \times 3.00 = 0.989$

**4.** $P = 5(0.98) + 105(0.96) = 4.90 + 100.80 = 105.70$

**5.**
- Actual/Actual: Days = 45 (Mar 1 to Apr 15). AI = $(45/184) \times 3.00 = 0.7337$
- 30/360: Days = $30 + 14 = 44$. AI = $(44/180) \times 3.00 = 0.7333$
- Difference: negligible in this case.

**6.** Price change = 5 ticks. Tick value = $31,250. P&L = $5 \times 31,250 = \$156,250$ (profit)

**7.** First coupon = $2.50 \times (120/180) = \$1.6667$

**8.** The premium reflects the PV of above-market coupons. As maturity approaches, fewer above-market coupons remain, so their PV decreases. At maturity, no coupons remain and the bond pays exactly par.

**9.** AI cannot exceed the coupon amount. $3.50 > 3.00$ violates the bound check. Either the AI calculation is wrong (day count error) or the coupon amount is wrong.

**10.** Missed interest $\approx 50{,}000{,}000 \times 0.03 \times 3/360 = \$12{,}500$.

**11.** Curve-based pricing discounts each cashflow at its own maturity (via discount factors / spot rates). A single YTM is an IRR chosen to match the bond's price; it does not, in general, discount every cashflow correctly unless the curve is flat (or the cashflow timing happens to align).

**12.** Clean = $101 + 4.625/32 = 101.1445$. Dirty = $101.1445 + 0.2279 = 101.3724$. Invoice on $10,000 face = $101.3724 \times 100 = \$10,137.24$

---

## References

- (Bruce Tuckman, *Fixed Income Securities*, “The Law of One Price”; “Treasury Bond Quotations”; “Yield-Based DV01”; “Special Repo Rates and the Auction Cycle”)
- (John C. Hull, *Options, Futures, and Other Derivatives*, “Day Counts”; “Price Quotations of U.S. Treasury Bonds”)
- (Edwin J. Elton, Martin J. Gruber, Stephen J. Brown, William N. Goetzmann, *Modern Portfolio Theory and Investment Analysis*, “Special Considerations in Bond Pricing”; “Appendix B — Estimating Spot Rates”)
- (Salih N. Neftci, *Principles of Financial Engineering*, “Dollar duration DV01”)
- (David G. Luenberger, *Investment Science*, “Spot Rates”; “Discount Factors and Present Value”; “Bond Details”)
- (Dessislava A. Pachamanova, Frank J. Fabozzi, *Simulation and Optimization in Finance*, “Accrued Interest”; “Clean vs Dirty (Full) Price”; “Traded Flat”)
