# Chapter 6: Yield-to-Maturity and Yield-Based Risk

---

## Introduction

A portfolio manager glances at two bonds on her screen. The first shows a yield of 5.2%; the second, 4.8%. The higher-yielding bond looks more attractive—40 basis points of extra return for similar credit quality. She buys the first bond.

Has she made a good decision? Not necessarily. Yield-to-maturity is one number summarizing a complex cashflow structure, and that compression can mislead. A higher YTM is not automatically better value or a higher realized return-to-maturity: the bond may have worse convexity, shorter effective duration, lower liquidity, or embedded optionality—characteristics that a single yield number cannot reveal.

Yield-to-maturity (YTM) is the fixed income market's most ubiquitous metric. It appears on every trading screen, in every research report, in every risk system. Bonds are quoted by yield as often as by price because the two are equivalent—given one, you can always compute the other. But this convenience creates a trap. Practitioners who treat YTM as "the return you'll earn" or who use yield-based risk measures without understanding their assumptions will eventually be surprised by P&L that doesn't match their expectations.

Prerequisites: [Chapter 2](chapters/chapter_02_time_value_discount_factors_replication.md), [Chapter 3](chapters/chapter_03_zero_forward_par_rates_triangle.md), [Chapter 5](chapters/chapter_05_fixed_rate_bond_pricing.md)

Follow-on: [Chapter 7](chapters/chapter_07_bond_return_decomposition.md), [Chapter 11](chapters/chapter_11_dv01_pv01_definitions_computation.md), [Chapter 12](chapters/chapter_12_duration_macaulay_modified_dv01.md), [Chapter 13](chapters/chapter_13_convexity.md), [Chapter 14](chapters/chapter_14_key_rate_dv01_bucket_exposures.md), [Chapter 16](chapters/chapter_16_curve_hedging_twists_butterflies_pca.md)

This chapter has three aims:

1. **Define YTM precisely**—including the clean/dirty price relationship, compounding and day-count conventions, and what YTM actually measures (and what it hides)
2. **Expose the reinvestment fallacy**—the dangerous myth that YTM represents a "locked-in" return
3. **Build yield-based risk measures**—DV01, modified duration, and convexity—and show why they can diverge from curve-based risk when the term structure moves in non-parallel ways

The yield-based framework developed here serves as the foundation for understanding interest rate risk, even as we acknowledge its limitations. Chapter 11 will revisit DV01 in the context of full curve construction, and Chapter 14 will introduce key-rate exposures that address the multi-factor nature of yield curve movements.

---

## Learning Objectives
- Define YTM as an IRR-style “single discount rate” and state the compounding and price basis being used.
- Translate a quoted clean price into the settlement cash amount via accrued interest and the dirty (invoice) price.
- Solve for YTM numerically (and run a reprice check: yield $\rightarrow$ price matches the dirty price).
- Explain why YTM is not a “locked-in” realized return and when horizon/total return analysis is the right tool.
- Compute and interpret yield-based risk (DV01, modified duration, convexity) with an explicit bump object, units, and sign convention.
- Explain (qualitatively) why yield-based hedges can fail under non-parallel curve moves and why key-rate risk exists.

## 6.1 Yield-to-Maturity as an Internal Rate of Return

### 6.1.1 The Definition

Yield-to-maturity (YTM) is the single annualized yield $y$ (under a stated compounding convention) that makes the present value of a bond’s promised cash flows equal to its market price. In other words, it is the internal rate of return (IRR) implied by the bond’s cash flows and price.

$$\boxed{P(y) = \sum_{t=1}^{2T} \frac{F \cdot c/2}{(1+y/2)^t} + \frac{F}{(1+y/2)^{2T}}}$$

where:
- $F$ is face value (e.g., $F=100$ for “per 100” bond pricing),
- $c$ is the annual coupon rate (decimal; e.g., 6% $\rightarrow 0.06$),
- $y$ is the yield-to-maturity (nominal annual with semiannual compounding),
- $T$ is years to maturity (so $2T$ semiannual periods),
- $P$ is the **dirty (invoice) price** per $F$.

In practice, you usually solve for $y$ numerically (bisection/Newton) and then run a **reprice check**: plug the solved yield back into the PV equation and verify it matches the **dirty (invoice) price** on settlement. Example A works through clean $\rightarrow$ dirty $\rightarrow$ YTM with concrete dates and unit checks.

### 6.1.2 YTM as a Quoting Convention

Yield-to-maturity compresses the entire term structure of discount rates into one “average discount rate” that reproduces the observed price. This makes YTM extraordinarily useful as a **quoting convention**: traders can move between price and yield with a simple calculation, and yields are often easier to compare across bonds of different maturities and coupons than raw prices.

On the trading desk, “bond yields 5.7%” is shorthand for “the single rate $y$ that reprices this bond under our standard compounding convention.” Because price $\leftrightarrow$ yield is easy to invert, yield is commonly used as an alternate way to quote price. The danger comes when participants forget that YTM is a summary statistic, not a measure of expected return.

> **Desk Reality:** Yields are a quoting language: desks often talk in yield (and yield spreads) because it makes prices comparable across coupons and maturities.
> **Common break:** Mixing yield conventions (semiannual vs continuous vs money-market) or treating YTM as a return forecast.
> **What to check:** Reprice check (yield $\rightarrow$ dirty price) and confirm day count/compounding settings.

### 6.1.3 The Compounding Convention

Bond yields are typically quoted with **semiannual compounding** because most bonds pay coupons twice per year. Under this convention, a yield of $y$ means:

- Each coupon period discounts by factor $1 + y/2$
- Two periods compound to $(1 + y/2)^2$ per year

When settlement doesn't align with regular coupon dates, the convention extends to fractional periods. For a Treasury bond, the actual/actual day count determines how many days remain until the next coupon, and the discounting uses appropriate fractional exponents.

> **Convention Warning:** Day count and compounding conventions vary by market and instrument. Your system’s convention affects both accrued interest and the yield implied by a given price—verify the convention before comparing yields across products.

### 6.1.4 Key Properties of the Price-Yield Relationship

Several important properties follow directly from the price-yield formula:

1. **Par pricing:** When $c = y$ (same compounding convention) and $F = 100$, then $P = 100$. If the coupon rate equals the yield, the bond trades at par.

2. **Premium bonds:** When $c \gt y$, then $P \gt F$. Above-market coupons are “paid for” by an upfront premium; the bond pulls back toward par by maturity.

3. **Discount bonds:** When $c \lt y$, then $P \lt F$. Below-market coupons require a capital gain at maturity.

4. **Perpetuity limit:** As $T \to \infty$, $P = Fc/y$. For example, at a yield of 5.50%, a 6.50% perpetuity with $F=100$ sells for $100\times 0.065/0.055 \approx 118.18$.

5. **Monotonicity:** Price is a decreasing function of yield—higher yields mean lower prices, and vice versa.

---

## 6.2 Clean and Dirty Prices

### 6.2.1 The Fundamental Identity

Bond market convention separates the quoted price from accrued interest:

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + \text{AI}}$$

Synonyms: the **dirty** price is also called the **full** or **invoice** price—it is what the buyer actually pays. The **clean** price is also called the **flat** or **quoted** price—it is what appears on trading screens.

### 6.2.2 Why Markets Quote Clean Prices

Consider what would happen if markets quoted dirty prices. Between coupon dates, interest accrues mechanically—a 5% coupon bond accrues about $5/365 \approx 1.4$ cents per day per USD 100 face. If quotes reflected this accrual, bond prices would drift upward between coupons and jump down on payment dates, even if nothing changed about the bond's fundamental value.

**Check (desk scale):** “1.4 cents per day per USD 100 face” sounds tiny until you scale it. On a USD 100mm face position, $(100{,}000{,}000/100)\times 0.014 \approx 14{,}000$ USD per day of mechanical dirty-price drift when the coupon rate is 5%. Clean-vs-dirty separation keeps this accrual from masquerading as trading P&L.

Clean pricing solves this problem. The clean price strips out the mechanical accrual component, so quoted prices better reflect changes in yields and market conditions. If yields are unchanged, the “jump” at a coupon date is (mostly) an accrued-interest reset, not a sudden deterioration in the bond’s economic value.

### 6.2.3 Computing Accrued Interest

Under the Actual/Actual (in period) convention (used for U.S. Treasuries), accrued interest is:

$$\boxed{\text{AI} = F\frac{c}{2} \times \frac{\text{days since last coupon}}{\text{days in coupon period}}}$$

where $F=100$ in “per 100” price quoting and $Fc/2$ is the semiannual coupon cashflow per $F$.

**Example:** A 6% Treasury with a 184-day coupon period. Settlement is 150 days after the last coupon.
- Coupon per period: $Fc/2 = 100\times 0.06/2 = 3.00$
- Accrual fraction: $150/184 = 0.8152$
- Accrued interest: $3.00 \times 0.8152 = 2.4457$

### 6.2.4 YTM Is Solved from the Dirty Price

This is a critical practical point: when solving for YTM, use the **dirty price** as the target. The present value of future cash flows must equal the cash the buyer pays, which is the invoice/dirty price.

The equation to solve is:

$$P_{\text{dirty}} = \sum_{i} \frac{CF_i}{(1 + y/2)^{a_i}}$$

where $a_i$ is the time (in semiannual periods) from settlement to cash flow $i$.

> **Pitfall — Clean vs dirty price:** Solving YTM off a clean quote (or mixing clean PV with dirty settlement cash).
> **Why it matters:** You will compute the wrong yield, and any downstream duration/DV01 will be mis-scaled.
> **Quick check:** Verify $P_{\text{dirty}} = P_{\text{clean}} + AI$ at settlement, and the PV equation targets $P_{\text{dirty}}$.

---

## 6.3 What YTM Summarizes—and What It Hides

### 6.3.1 The Summary: Level of Discounting

YTM tells you the "average" rate at which the bond's cash flows are discounted. For a given term structure, YTM sits somewhere between the short-term and long-term spot rates, weighted by the present values of the cash flows.

Illustrative example: suppose the first four semiannual spot rates are 5.008%, 4.929%, 4.864%, and 4.886%, and the bond’s YTM comes out to 4.8875%. The YTM is a present-value-weighted blend of spot rates, and it often lands close to the maturity where most PV sits (frequently near the final principal payment).

For a zero-coupon bond, this average is trivial—the YTM equals the spot rate to that maturity (under matched compounding conventions). For coupon bonds, YTM blends across multiple maturities.

### 6.3.2 What YTM Hides

**The term structure:** Spot rates differ by maturity. A 10-year Treasury's cash flows should, in principle, be discounted at ten different rates. YTM replaces this structure with a single number. This means YTM cannot reveal whether a bond is "cheap at the front" (early cash flows undervalued) and "rich at the back" (late cash flows overvalued), or vice versa.

**Reinvestment assumptions:** Because coupons are invested at uncertain future rates, it is extremely unlikely that the realized yield of a coupon bond held to maturity will equal its original yield-to-maturity. The uncertainty of the realized yield relative to the original yield because coupons are invested at uncertain future rates is often called **reinvestment risk**.

**Realized return:** Many practitioners mistakenly interpret YTM as the return they will earn. This is only approximately true under very restrictive conditions—specifically, that the yield remains unchanged throughout the holding period.

### 6.3.3 When YTM Equals Realized Return

If a bond's yield-to-maturity remains unchanged over a short time period, that bond's realized total rate of return equals its yield.

This is the **constant yield** or **roll-down** assumption—useful for short-term return attribution, but not a forecast of what will actually happen. Over longer horizons or when yields change, realized returns diverge from initial YTM due to:

- Reinvestment rates differing from the original yield (coupon compounding risk)
- Curve moves and sale-price effects (especially if you sell before maturity)

> **Analogy: The Speedometer vs. The Destination**
>
> **Yield-to-Maturity (YTM)** is like the **Speedometer** reading at the start of a road trip.
> *   It tells you your *current* speed (rate of return) *if* traffic never changes.
> *   **Realized Return** is your actual average speed when you arrive.
>
> | **Assumption (YTM)** | **Reality (Realized Return)** |
> | :--- | :--- |
> | **Traffic**: Constant Speed (Reinvest Coupons at YTM) | **Traffic**: Rates Change (Reinvestment Risk) |
> | **Route**: No Detours (Hold to Maturity) | **Route**: Detours (Sell Early / Curve Moves) |
>
> **Lesson**: YTM is a snapshot, not a promise. To earn the YTM, you need a flat road (unchanged rates) and no stops (hold to maturity).

### 6.3.4 YTM and Relative Value

Can you compare two bonds by their yields and conclude the higher-yielding bond is "better"? Generally, no. Two bonds with the same YTM can have different:

- Cash flow timing (duration)
- Price sensitivity (convexity)
- Liquidity
- Credit risk
- Optionality

Relative-value screens that rank bonds by yield can be misleading. A proper relative value analysis requires understanding the full term structure and the specific characteristics of each bond.

---

## 6.4 The Reinvestment Fallacy: Why YTM Is Not a Locked-In Return

### 6.4.1 The Myth of "Locked-In" Returns

One of the most persistent misconceptions in fixed income is that buying a bond and holding it to maturity “locks in” the yield-to-maturity as your return. This is false. The YTM calculation implicitly assumes every coupon payment can be reinvested at exactly the same yield, an assumption that rarely holds in practice.

The mechanics are straightforward: when you receive a coupon payment, you must do something with that cash. If you reinvest it, the rate available at that future time will almost certainly differ from the original YTM. If rates have fallen, you'll reinvest at lower rates; if rates have risen, you'll reinvest at higher rates. Either way, your realized return will differ from the initial YTM.

### 6.4.2 Decomposing Total Return

To understand reinvestment risk, decompose total return into three components:

$$\boxed{\text{Total Return} = \text{Coupon Income} + \text{Reinvestment Income} + \text{Price Change}}$$

For a hold-to-maturity investor:
- **Coupon Income**: The sum of all coupon payments—this is known at purchase
- **Reinvestment Income**: Interest earned on reinvested coupons—this is unknown
- **Price Change**: For hold-to-maturity, the bond matures at par—this is known

The reinvestment component is the source of uncertainty. For high-coupon, long-maturity bonds, reinvestment income can represent a substantial fraction of total return.

### 6.4.3 Worked Example: The 8% Bond Scenarios

Consider an investor who purchases a 10-year, 8% annual coupon bond at par (price = 100, YTM = 8%). The investor plans to hold to maturity.

**Base Case: Rates Stay at 8%**

Each annual coupon of 8 USD is reinvested at 8%. At maturity:

| Year | Coupon | Years to Compound | Future Value |
|------|--------|-------------------|--------------|
| 1 | 8 | 9 | $8 \times 1.08^9 = 15.99$ |
| 2 | 8 | 8 | $8 \times 1.08^8 = 14.81$ |
| 3 | 8 | 7 | $8 \times 1.08^7 = 13.71$ |
| ... | ... | ... | ... |
| 10 | 8 | 0 | $8.00$ |

Total future value of coupons = $8 \times \frac{(1.08^{10} - 1)}{0.08} = 115.89$

Add principal: $100 + 115.89 = 215.89$

Realized return: $(215.89/100)^{1/10} - 1 = 8.00\%$ ✓

**Scenario A: Rates Drop to 6% Immediately After Purchase**

Now each coupon is reinvested at 6%:

Total future value of coupons = $8 \times \frac{(1.06^{10} - 1)}{0.06} = 105.50$

Add principal: $100 + 105.50 = 205.50$

Realized return: $(205.50/100)^{1/10} - 1 = 7.48\%$

**Scenario B: Rates Rise to 10% Immediately After Purchase**

Total future value of coupons = $8 \times \frac{(1.10^{10} - 1)}{0.10} = 127.50$

Add principal: $100 + 127.50 = 227.50$

Realized return: $(227.50/100)^{1/10} - 1 = 8.57\%$

**Summary:**

| Scenario | Reinvestment Rate | Realized Return | vs. YTM |
|----------|-------------------|-----------------|---------|
| Base | 8% | 8.00% | — |
| Rates Fall | 6% | 7.48% | -52 bps |
| Rates Rise | 10% | 8.57% | +57 bps |

The initial YTM of 8% was never "locked in." Reinvestment risk created a 109 bp range of outcomes.

> **Desk Reality: Liability Matching vs Reinvestment Risk**
>
> Liability managers care about the cash available at future dates, not just today’s YTM. Coupon bonds create intermediate cash that must be reinvested at whatever rates prevail in the future, creating reinvestment risk. Zero-coupon bonds (and strips) remove coupon reinvestment risk because there are no interim cash flows.
>
> Trade-offs: zeros concentrate PV later in time, so they typically have higher duration/convexity per dollar invested and larger mark-to-market swings for a given yield move.

### 6.4.4 The Reinvestment Risk Hierarchy

Reinvestment risk varies systematically across bond types:

| Bond Type | Reinvestment Risk | Why |
|-----------|-------------------|-----|
| Zero-coupon | None | No intermediate cash flows |
| Low-coupon | Low | Fewer/smaller coupons to reinvest |
| High-coupon | High | More cash flow arriving early |
| Short maturity | Low | Less time for rate changes |
| Long maturity | High | More coupons, longer horizon |
| Annuity/amortizing | Very high | Principal returns throughout life |

**Key insight:** Reinvestment risk and price risk move in opposite directions. If rates fall:
- Price risk: Unrealized gain (bond appreciates)
- Reinvestment risk: Future coupons reinvest at lower rates

This is the foundation of immunization strategies covered in Chapter 15.

### 6.4.5 Total Return Analysis

Because YTM is unreliable as a return forecast, practitioners use **total return analysis** (also called horizon analysis). This involves:

1. Specifying an investment horizon (e.g., 3 years)
2. Assuming a reinvestment rate for coupons during the holding period
3. Assuming an ending yield to compute the sale price
4. Calculating the total return from all three components

This approach makes assumptions explicit rather than hiding them inside the YTM calculation.

> **Practitioner Note:** Horizon/total-return analysis answers “what return do I earn over my horizon under a rate scenario?” It makes the assumptions (horizon, reinvestment rate, ending yield) explicit instead of burying them inside YTM.

---

## 6.5 Yield-Based DV01 and Modified Duration

### 6.5.1 DV01: Dollar Value of a Basis Point

In this chapter, **yield DV01** measures how the bond’s **dirty price** (quoted per 100 face) changes for a 1 bp change in the bond’s **yield-to-maturity** $y$ (the bump object).

A common yield-based definition is:

$$\boxed{DV01_y := -\frac{1}{10{,}000}\frac{dP}{dy}}$$

When an explicit price-yield function exists, DV01 can be written in closed form. Differentiating the price-yield function for a plain fixed-coupon bond gives:

$$DV01_y
= \frac{1}{10{,}000} \times \frac{1}{1+y/2}\left[\sum_{t=1}^{2T} \frac{t}{2} \frac{F c/2}{(1+y/2)^t} + T \frac{F}{(1+y/2)^{2T}}\right]$$

In words: DV01 is proportional to the **time-weighted PV** of the bond’s cash flows, scaled to a 1 bp yield bump.

**Scaling to dollars:** if prices are quoted per 100 and your position has face notional $N$, then $DV01_{USD}(N)=\frac{N}{100}DV01_y$.

> **Desk Reality: DV01 as the Lingua Franca of Rates**
>
> Desk shorthand: “long $50k DV01” means the position gains about $50,000 for a 1 bp decline in the bumped rate object (here, the bond’s YTM).
>
> Why DV01 helps: notionals are hard to compare across maturities and coupons. For example, if a 2-year position has DV01 of $19k and a 10-year has DV01 of $42k, the 10-year has more than twice the parallel rate sensitivity.
>
> **Common gotcha:** Make sure everyone is using the same DV01 convention. "DV01 per million" vs "total DV01" vs "DV01 per bp per 100 face" can cause confusion.

### 6.5.2 Computing DV01 Numerically

For practical computation, use a central difference:

$$\boxed{DV01_y \approx \frac{P(y - 1\text{ bp}) - P(y + 1\text{ bp})}{2}}$$

Central differences (equally spaced bumps above and below) are usually the most stable finite-difference estimate.

### 6.5.3 Modified Duration

Duration measures percentage price sensitivity rather than dollar sensitivity.

Modified duration is defined as:

$$\boxed{D_{\text{mod}} = -\frac{1}{P}\frac{dP}{dy}}$$

This gives the approximate percentage price change per unit change in yield:

$$\frac{\Delta P}{P} \approx -D_{\text{mod}} \cdot \Delta y$$

### 6.5.4 The DV01-Duration Link

DV01 and modified duration are related by:

$$\boxed{DV01_y = \frac{P \cdot D_{\text{mod}}}{10{,}000}}$$

Given either measure, you can compute the other if you know the price. The distinction matters: DV01 gives dollar risk (useful for hedging notional amounts), while duration gives percentage risk (useful for comparing bonds of different prices).

### 6.5.5 Macaulay Duration and Its Interpretation

Macaulay duration is a simple transformation of modified duration:

$$\boxed{D_{\text{Mac}} = (1 + y/2) \cdot D_{\text{mod}}}$$

Convenient property: the Macaulay duration of a $T$-year zero-coupon bond equals $T$. Mathematically:

$$\left.D_{\text{Mac}}\right|_{c=0} = T$$

This provides intuition for duration generally—a bond's Macaulay duration equals the maturity of a zero-coupon bond with the same price sensitivity. "The Macaulay duration of a six-month zero is simply .5 while that of a 10-year zero is simply 10."

> **Analogy: The Teeter-Totter (Seesaw)**
>
> Visualize a seesaw where the beam represents time.
> *   **Weights**: The cashflows (Coupons and Principal) sitting on the beam.
> *   **Fulcrum**: The point where the beam balances perfectly.
>
> **Duration is the Fulcrum**.
> *   **Zero Coupon Bond**: One giant weight at the very end. The fulcrum MUST be at the end to balance. ($D = T$)
> *   **Coupon Bond**: Small weights (coupons) distributed along the beam. They pull the center of gravity (fulcrum) toward the middle. ($D \lt T$)
>
> Higher coupons = Heavier weights at the front = Fulcrum moves closer to zero (Lower Duration).

Duration is a present-value-weighted average of the times when payments are made. For a 5-year bond with Macaulay duration of 4.4 years, you wait (on a present-value-weighted basis) 4.4 years to receive your money back.

### 6.5.6 How Duration Varies with Coupon and Maturity

Qualitative relationships:

- **Higher coupon → lower duration:** More cash flow arrives early, pulling the weighted-average time forward
- **Longer maturity → higher duration:** For most bonds, though the effect asymptotes
- **Zero-coupon bonds have duration = maturity:** All cash arrives at one point
- **Perpetuities have finite duration:** At yield $y$, Macaulay duration approaches $(1+y/2)/y$

**Perpetuity Duration Example:** As $T \to \infty$:

$$\left.D_{\text{Mac}}\right|_{T=\infty} = \frac{1+y/2}{y}$$

At a 5\% yield: $D_{\text{Mac}} = (1.025)/0.05 = 20.5$ years. Despite paying forever, a perpetuity at 5\% has finite duration because early payments dominate the present-value weighting.

---

## 6.6 Yield-Based Convexity

### 6.6.1 Definition and Interpretation

Convexity measures the curvature of the price-yield relationship—how duration itself changes as yields move. Yield-based convexity is defined as:

$$\boxed{C_y = \frac{1}{P}\frac{d^2P}{dy^2}}$$

where $d^2P/dy^2$ is the second derivative of the price-yield function. It measures how the first derivative (duration) changes with yield.

Convexity explains the asymmetry in price changes: when yields fall, prices rise by more than duration alone would predict; when yields rise, prices fall by less. This asymmetry favors bondholders.

### 6.6.2 The Duration-Convexity Approximation

The second-order Taylor expansion gives:

$$\boxed{\Delta P \approx -P \cdot D_{\text{mod}} \cdot \Delta y + \frac{1}{2} P \cdot C_y \cdot (\Delta y)^2}$$

For small yield changes, the duration term typically dominates and convexity is a small correction. As moves get larger, convexity becomes more meaningful.

### 6.6.3 Positive Convexity as a Benefit

For plain-vanilla fixed-rate bonds, convexity is positive ($C_y \gt 0$). This means:

- Gains from yield decreases exceed the duration prediction
- Losses from yield increases fall short of the duration prediction

Graphically, "the property of positive convexity may also be thought of as the property that DV01 falls as rates increase." The price-rate curve is convex (curving upward), which is beneficial to the bondholder.

> **Analogy: Convexity is a Smile**
>
> Plot **Price (y-axis)** vs. **Yield (x-axis)**.
> *   **Duration** is a straight line tangent to the curve. It assumes linear risk.
> *   **Convexity** is the "smile" of the actual price curve.
>
> **Why the Smile is Good**:
> *   **Rates Fall**: You are on the steep part of the curve. Price rises *faster* than linear duration predicts. (You win BIG).
> *   **Rates Rise**: You are on the flat part of the curve. Price falls *slower* than linear duration predicts. (You lose SMALL).
>
> "You win more than you lose." This is why traders pay up for convexity.

### 6.6.4 The Cost of Convexity

Convexity is valuable, and markets price it accordingly: higher convexity (all else equal) typically comes with a lower yield. An investor who wants more convexity must often accept a yield give-up—there is no free lunch.

The economic intuition is straightforward: if Bond A and Bond B have the same duration but Bond A has higher convexity, Bond A will outperform in large rate moves (either direction). The market recognizes this and bids up the price of Bond A, lowering its yield.

**Quantifying the trade-off:** Consider two duration-matched portfolios:
- Portfolio A: High convexity, yield = 5.00\%
- Portfolio B: Low convexity, yield = 5.15\%

The 15 bp yield give-up is the "price" of convexity. Whether this is worthwhile depends on:
1. Expected rate volatility (higher volatility → convexity more valuable)
2. The investor's horizon (shorter horizon → less time for convexity to help)
3. The investor's view on rate direction (asymmetric views may favor different structures)

> **Desk Reality: "Paying for Convexity"**
>
> When a trader says "I'm paying 10 bps for convexity," they mean they're accepting a 10 bp lower yield on a high-convexity position compared to a duration-matched alternative. This is the explicit cost of the asymmetric payoff profile.
>
> The decision framework: if you expect rates to move significantly (high volatility), paying for convexity is rational. If you expect rates to be stable, collecting the yield premium (selling convexity) is better.
>
> **Who sells convexity?** Mortgage investors are natural sellers—MBS have negative convexity from prepayment risk. They receive higher yields as compensation.

### 6.6.5 The Convexity-Gamma Connection

For readers familiar with options, convexity has a direct analogue: **gamma**. Both measure the second derivative of value with respect to an underlying variable.

| Concept | Options | Bonds |
|---------|---------|-------|
| First derivative | Delta ($\Delta$) | Duration |
| Second derivative | Gamma ($\Gamma$) | Convexity |
| Interpretation | How delta changes | How duration changes |
| Sign for long positions | Positive | Positive |
| Benefit | Win more, lose less | Win more, lose less |
| Cost | Pay option premium | Accept lower yield |

**Long convexity = Long gamma.** Both create a payoff profile where you benefit from volatility. Both are valuable in uncertain environments. Both cost money.

The mathematical parallel:

$$\text{Options: } \Delta V \approx \Delta \cdot \Delta S + \frac{1}{2}\Gamma \cdot (\Delta S)^2$$

$$\text{Bonds: } \Delta P \approx -D \cdot P \cdot \Delta y + \frac{1}{2}C \cdot P \cdot (\Delta y)^2$$

> **Practitioner Note:** The convexity-gamma analogy extends further. Just as options traders "gamma scalp" by rebalancing their delta hedge to capture volatility, bond traders can implement convexity trades by rebalancing duration. The principle is identical: the second derivative generates P&L from volatility of the underlying.

### 6.6.6 The Barbell versus the Bullet

A classic example demonstrating how portfolio structure affects convexity: an asset-liability manager with liabilities having 9-year duration could fund with:

- **Bullet portfolio:** Intermediate-maturity bonds with 9-year duration
- **Barbell portfolio:** A mix of short and long bonds (e.g., 2-year and 30-year) that also has 9-year portfolio duration

The barbell has higher convexity because convexity increases with the *square* of maturity. Using zero-coupon bonds, a simple numerical illustration is:

**Example: 75\%/25\% Barbell vs 9-Year Bullet**

At a 5\% yield:
- 2-year zero: Duration = 2, Convexity ≈ 4
- 30-year zero: Duration = 30, Convexity ≈ 871
- 9-year zero (bullet): Duration = 9, Convexity ≈ 81

A barbell of 75\% in 2-year zeros and 25\% in 30-year zeros:
- Portfolio duration: $0.75 \times 2 + 0.25 \times 30 = 9$ years ✓
- Portfolio convexity: $0.75 \times 4 + 0.25 \times 871 = 3 + 218 = 221$

The barbell has convexity of **221** versus the bullet's **81**—nearly three times as much convexity for the same duration.

**Performance comparison:**

| Rate Move | Bullet Performance | Barbell Performance | Winner |
|-----------|-------------------|---------------------|--------|
| Small (±25 bp) | Better | Worse | Bullet |
| Medium (±100 bp) | Similar | Similar | Tie |
| Large (±200 bp) | Worse | Better | Barbell |

Rule of thumb: bullets tend to do better for small rate moves, while barbells tend to do better for large moves (because the barbell has higher convexity).

> **Key Insight:** "Spreading out the cash flows of a portfolio (without changing its duration) raises its convexity." This is the mathematical essence of barbelling.

> **Desk Reality: Barbell vs Bullet in Practice**
>
> The barbell-bullet trade is a classic rates position. A trader who expects high volatility will prefer the barbell; one expecting stability prefers the bullet (and collects the yield advantage).
>
> **The "butterfly" trade** combines both views: long the wings (2y and 30y), short the belly (10y). This is a pure bet on convexity/volatility without a directional rate view.
>
> **Warning:** The simple barbell example assumes parallel shifts. If the curve twists (e.g., 2s flatten while 30s steepen), the barbell can underperform even with realized volatility. Real-world implementation requires careful curve analysis.

### 6.6.7 Convexity of Zero-Coupon Bonds

Setting $c=0$ in the convexity formula yields:

$$\boxed{\left.C\right|_{c=0} = \frac{T(T + 0.5)}{(1+y/2)^2}}$$

Convexity increases with the **square** of maturity—much faster than duration, which increases approximately linearly. This is why long-dated zeros have enormous convexity. A 30-year zero at 5\% yield has convexity of approximately:

$$C = \frac{30 \times 30.5}{(1.025)^2} \approx 871$$

### 6.6.8 Computing Convexity Numerically

With a finite-difference approach using $\Delta = 50$ bp:

$$\boxed{C_y \approx \frac{P(y - \Delta) + P(y + \Delta) - 2P(y)}{P(y) \cdot \Delta^2}}$$

Second derivatives are small effects, so you often need extra numerical precision when estimating convexity.

### 6.6.9 Negative Convexity and Callable Bonds

Not all fixed income securities exhibit positive convexity at all rate levels. Callable bonds and mortgage-backed securities are common examples of negative convexity.

**The Callable Bond Pricing Relationship:**

A callable bond can be decomposed as:

$$\boxed{P_{\text{callable}} = P_{\text{non-callable}} - \text{Value of Call Option}}$$

The issuer owns an embedded call option—the right to redeem the bond early at a specified price (typically par). This option becomes valuable when rates fall significantly below the coupon rate.

**Why Callable Bonds Have Negative Convexity:**

When rates fall substantially:
- A non-callable bond's price rises without limit
- A callable bond's price is capped near the call price—the issuer will call the bond

This cap eliminates the upside that positive convexity provides. In the region where rates are low enough that the call is likely, the price-yield curve bends *downward* (concave), creating negative convexity.

A numerical illustration: a callable bond can show strongly negative convexity (e.g., around **-223**) when the call is deep in the money, compared to positive convexity for the same structure when rates are high.

**Table: Convexity Across Rate Levels (Callable vs Non-Callable)**

| Yield Level | Non-Callable Convexity | Callable Convexity |
|-------------|------------------------|-------------------|
| High (call out-of-money) | +80 | +75 |
| Medium (call at-money) | +80 | +10 |
| Low (call in-money) | +80 | -223 |

> **Desk Reality: Trading Callable Bonds**
>
> Callable bonds are typically quoted in terms of "yield-to-call" (YTC) when trading near or above the call price, and "yield-to-maturity" when trading well below. The switch point is important: a bond quoted at YTM when it should be quoted at YTC will appear cheaper than it is.
>
> **The negative convexity trap:** A trader who buys a callable bond for its "high yield" without understanding the embedded short option position may be surprised when rates fall but the bond doesn't rally as expected. You're effectively short volatility.
>
> **Who issues callables?** Corporations and agencies that want to refinance if rates fall. Investors demand higher yields as compensation for bearing prepayment/call risk.

### 6.6.10 Yield-to-Call and Yield-to-Worst

For callable bonds, practitioners use additional yield measures:

**Yield-to-Call (YTC):** The yield assuming the bond is called at the next call date:

$$P_{\text{dirty}} = \sum_{t=1}^{T_{\text{call}}} \frac{CF_t}{(1+y_c/2)^t} + \frac{\text{Call Price}}{(1+y_c/2)^{T_{\text{call}}}}$$

**Yield-to-Worst (YTW):** The minimum of YTM and all possible YTCs:

$$\boxed{YTW = \min(YTM, YTC_1, YTC_2, ...)}$$

In practice, investors often use yield-to-worst: the yield given the redemption price schedule that minimizes value to the investor.

> **Practitioner Note:** Yield-to-worst is conservative but not always accurate. It assumes the issuer exercises optimally against the investor; real-world exercise decisions can be affected by frictions (transaction costs, financing, operational constraints).

---

## 6.7 Curve-Based Risk: A Preview

### 6.7.0 The Three Drivers of Yields (PCA)

In a principal-components analysis of yield changes, the first three components span about **95\%** of the variation. The first principal component has all positive values, so it corresponds to approximately parallel shifts of the yield curve. Changes along the second principal component either increase or decrease the slope of the yield curve. Changes along the third principal component either increase or decrease the curvature (hump) of the yield curve.

One common interpretation of these first three drivers is:
1. **Level (shift):** yields move up/down roughly together (duration/DV01 targets this)
2. **Slope (twist):** the curve steepens or flattens
3. **Curvature (butterfly):** the belly moves differently from the wings

Yield-based risk (duration/DV01) implicitly assumes the risk is well described by the first driver (a near-parallel move). It does not hedge twist or butterfly risk.

### 6.7.1 The Single-Factor Assumption

All yield-based measures assume that a bond's price depends on one number: its yield. This is convenient but dangerous. A key weakness is assuming that movements in the entire term structure can be described by one interest-rate factor.

In reality, the price of a coupon bond depends on the entire term structure:

$$P = \sum_{i} CF_i \cdot P(0, t_i)$$

where $P(0,t_i)$ is the discount factor for maturity $t_i$.

### 6.7.2 Curve DV01

Under a full curve framework, we can define curve DV01 as the sensitivity to a parallel bump in spot rates. For continuously compounded spot rates $\hat{r}(t)$:

$$d(t) = \exp(-\hat{r}(t) \cdot t)$$

A 1-bp parallel bump gives:

$$d_{\Delta}(t) = d(t) \cdot \exp(-\Delta \cdot t)$$

The curve DV01 is then $P_{\text{base}} - P_{\text{bumped}}$.

**Sign convention note:** $P_{\text{base}} - P_{\text{bumped}}$ corresponds to “PV(base) minus PV(rates up 1bp)”, which is positive for a long fixed-rate bond. This book’s DV01 convention is “PV(rates down 1bp) minus PV(base)”. The magnitudes match (for a symmetric design); only the bump direction differs. Always state what was bumped and in which direction.

### 6.7.3 When Yield DV01 and Curve DV01 Differ

Yield DV01 assumes the bond's yield changes. Curve DV01 bumps all spot rates uniformly. These can differ because:

1. YTM is an average rate; bumping it doesn't replicate bumping each individual spot rate
2. The weighting of maturities differs between the two approaches

The differences can be material for:

- Bonds priced at significant premiums or discounts
- Portfolios mixing short and long maturities
- Hedges involving different instruments

### 6.7.4 Why Yield Hedges Fail Under Curve Twists

A yield-DV01 hedge matches the DV01 of bond A with bond B. This works if both yields change by the same amount. A major weakness of the approach is the assumption that movements in the entire term structure can be described by one interest rate factor.

When the curve twists—say, the 2-year yield rises while the 10-year falls—a DV01-matched hedge can lose money. The hedge was built for parallel shifts; it has no protection against shape changes.

This limitation motivates the multi-factor approaches in Chapter 14 (key-rate durations) and Chapter 16 (curve hedging with PCA).

> **Desk Reality: P&L Breaks from Curve Moves**
>
> A classic P&L break: the risk system shows you're "flat" because your total DV01 is zero. But you're long 5-year and short 10-year in equal DV01. When the curve flattens (5s underperform 10s), you lose money despite being "hedged."
>
> **The conversation with your PM:**
> "But the risk report said we were flat!"
> "We were flat to parallel shifts. We were massively exposed to curve."
>
> This is why sophisticated desks run key-rate DV01 reports, not just total DV01.

---

## 6.8 Worked Examples

### Example A: Clean Quote → Dirty Price → YTM → Yield DV01 (Unit/Sign Checks)

**Example Title**: Clean quote → YTM → DV01 for a Treasury

**Context**
- You see a Treasury quoted at a clean price and want (i) the YTM and (ii) a yield-based DV01 in desk units.

**Timeline (Make Dates Concrete)**
- Settlement date: 2026-01-12
- Accrual start (last coupon date): 2025-08-15
- Next coupon: 2026-02-15
- Maturity: 2031-02-15 (semiannual coupons on Feb 15 / Aug 15)

**Inputs**
- Instrument: U.S. Treasury, coupon 6\% (semiannual coupon = 3.00 per 100)
- Clean price quote: $P_{\text{clean}}=101.25$ (price points per 100)
- Accrued interest (Actual/Actual for the example): coupon period = 184 days; days accrued = 150
- Yield convention: nominal annual YTM with semiannual compounding ($m=2$)

**Outputs (What You Produce)**
- Accrued interest: $AI = 2.4457$ (points per 100)
- Dirty price: $P_{\text{dirty}} = 103.6957$ (points per 100)
- YTM: $y \approx 5.709\%$ (annual, semiannual compounding)
- Yield DV01 (bump object $y$): $DV01_y \approx 0.0440$ (points per 100 per 1 bp; positive for a long bond when yields fall)
- Dollar DV01 for $N=1{,}000{,}000$ USD: $DV01_{USD} \approx 440$ USD per 1 bp
- Modified duration: $D_{\text{mod}} \approx 4.24$ (years)

**Step-by-step**
1. **Translate quote to settlement cash (clean → dirty).**
   Use $AI = 3.00\times \frac{150}{184} = 2.4457$ and $P_{\text{dirty}} = 101.25 + 2.4457 = 103.6957$.

2. **Solve for YTM $y$ from the dirty price.** Write $r=1+y/2$ and discount each cashflow by $r^{a_i}$. For this example we use an approximation $a_i \approx \text{(days to payment)}/182.5$ to express time in semiannual periods:
   Solve $\sum_i \frac{CF_i}{r^{a_i}} = 103.6957$, giving $y\approx 5.709\%$.

3. **Compute yield DV01 with a central 1 bp bump.**
   Use $DV01_y \approx \frac{P(y-1\text{ bp})-P(y+1\text{ bp})}{2} = \frac{103.7397-103.6516}{2} \approx 0.0440$.

4. **Scale to dollars (desk unit check).**
   Compute $DV01_{USD} = \frac{N}{100}DV01_y = \frac{1{,}000{,}000}{100}\times 0.0440 \approx 440$ USD per 1 bp.

5. **Optional cross-check: infer modified duration from DV01.**
   Infer $D_{\text{mod}} = \frac{DV01_y\times 10{,}000}{P} \approx \frac{0.0440\times 10{,}000}{103.6957} \approx 4.24$.

**Cashflows (per 100, dirty-price convention)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-02-15 | 3.00 | coupon |
| 2026-08-15 | 3.00 | coupon |
| 2027-02-15 | 3.00 | coupon |
| 2027-08-15 | 3.00 | coupon |
| 2028-02-15 | 3.00 | coupon |
| 2028-08-15 | 3.00 | coupon |
| 2029-02-15 | 3.00 | coupon |
| 2029-08-15 | 3.00 | coupon |
| 2030-02-15 | 3.00 | coupon |
| 2030-08-15 | 3.00 | coupon |
| 2031-02-15 | 103.00 | coupon + principal |

**P&L / Risk Interpretation**
- A long position has $DV01_y \gt 0$: if yields fall 1 bp (all else equal), price rises by about 0.044 points per 100.
- For $N=1$ mm USD, a 10 bp parallel move in the bond’s YTM corresponds to about $10\times 440 \approx 4{,}400$ USD of P&L (ignoring convexity and curve-shape effects).

**Sanity Checks**
- Units: $P$ and $AI$ are points per 100; $DV01_y$ is points per 100 per bp; $DV01_{USD}$ is currency per bp.
- Sign: yields down $\Rightarrow$ price up $\Rightarrow$ $DV01_y \gt 0$ for a long bond.
- Reprice check: plug the solved $y$ back into the PV equation and confirm it matches $P_{\text{dirty}}$ within rounding tolerance.

---

### Example B: Newton-Raphson Iteration for YTM

Starting from Example A, let's trace one Newton-Raphson step.

**Setup:**
- Target: $P_{\text{dirty}} = 103.6957$
- Initial guess: $y_0 = 6.00\%$
- At $y_0$: $P(y_0) = 102.4232$
- Error: $f(y_0) = 102.4232 - 103.6957 = -1.2725$

**Derivative:**

$$\frac{dP}{dy} = -\frac{1}{2r} \sum_i a_i \cdot CF_i \cdot r^{-a_i} \approx -433.55$$

**Newton step:**

$$y_1 = y_0 - \frac{f(y_0)}{f'(y_0)} = 0.06 - \frac{-1.2725}{-433.55} = 0.05707$$

The next iterate is 5.707\%, very close to the solution.

**Sanity check:** Since $f(y_0) \lt 0$ (PV too low), we need a lower yield to increase PV. The step correctly moves downward.

---

### Example C: Price-Yield Curve and Convexity

Using the Example A bond at $y^* = 5.709\%$:

| Yield | Price |
|-------|-------|
| 5.209\% ($y^* - 50$ bp) | 105.9240 |
| 5.709\% ($y^*$) | 103.6957 |
| 6.209\% ($y^* + 50$ bp) | 101.5219 |

**Asymmetry observation:**
- Down-move gain: $+2.228$
- Up-move loss: $-2.174$

The gain exceeds the loss—this is positive convexity.

**Convexity estimate:**

$$C_y \approx \frac{105.9240 + 101.5219 - 2(103.6957)}{103.6957 \times (0.005)^2} \approx 21.1$$

---

### Example E: Yield DV01 vs Curve DV01

**Bond:** 2-year, 4\% coupon, semiannual

**Given discount factors:**

| $t$ | $P(0,t)$ |
|-----|----------|
| 0.5 | 0.9850 |
| 1.0 | 0.9700 |
| 1.5 | 0.9530 |
| 2.0 | 0.9350 |

**Curve PV:**

$$P = 2(0.9850) + 2(0.9700) + 2(0.9530) + 102(0.9350) = 101.186$$

**Curve DV01** (parallel 1 bp bump to continuous spot rates):

After bumping, $P_{\Delta} = 101.166$

$$DV01_{curve} = 101.186 - 101.166 = 0.0197$$

**Yield DV01:**

Solve for YTM: $y \approx 3.382\%$

$$DV01_y \approx 0.0193$$

**Comparison:** The two differ by about 2\%. This gap arises because curve DV01 weights longer maturities more heavily (through the $t$ factor in `exp(-\Delta t)`), while yield DV01 treats all cash flows through one rate.

---

### Example F: Same YTM, Different Risk

Two 5-year bonds both yielding 5\%:

| | Bond A (8\% coupon) | Bond B (2\% coupon) |
|---|---|---|
| Price | 113.13 | 86.87 |
| $D_{\text{mod}}$ | 4.17 | 4.65 |
| $DV01_y$ | 0.0472 | 0.0404 |
| $C_y$ | 21.1 | 24.5 |

Despite identical YTM, Bond B has higher duration and convexity because its cash flows are more concentrated at maturity (lower coupons mean more principal-weighted value). YTM alone cannot distinguish these risk profiles.

---

### Example G: Perpetuity Duration

Consider a perpetuity with face value $F=100$ and coupon rate $c=5\%$ (so the annual coupon is $Fc=5$, paid as $2.50$ semiannually) at a yield of $y=6\%$.

**Price (per 100):** $P = Fc/y = 100\times 0.05/0.06 = 83.33$

**Macaulay Duration:**

$$D_{\text{Mac}} = \frac{1+y/2}{y} = \frac{1.03}{0.06} = 17.17 \text{ years}$$

**Modified Duration:**

$$D_{\text{mod}} = \frac{D_{\text{Mac}}}{1+y/2} = \frac{17.17}{1.03} = 16.67$$

**Yield DV01 (per 100):**

$$DV01_y = \frac{P \times D_{\text{mod}}}{10{,}000} = \frac{83.33 \times 16.67}{10{,}000} \approx 0.139$$

Despite infinite maturity, the perpetuity has a finite 17-year duration because early payments dominate the present-value weighting.

---

### Example H: Reinvestment Risk Calculation

**Bond:** 5-year, 6\% annual coupon, purchased at par (YTM = 6\%)

**Scenario:** Rates fall to 4\% immediately after purchase and remain there.

**Step 1: Future value of reinvested coupons at 4\%**

$$FV_{\text{coupons}} = 6 \times \frac{(1.04^5 - 1)}{0.04} = 6 \times 5.416 = 32.50$$

**Step 2: Total terminal value**

$$\text{Terminal Value} = 100 + 32.50 = 132.50$$

**Step 3: Realized return**

$$r = (132.50/100)^{1/5} - 1 = 5.79\%$$

**Shortfall:** The realized return of 5.79\% is 21 bps below the initial YTM of 6.00\%.

---

### Example I: Callable Bond Decomposition

**Setup:** A 10-year, 8\% corporate bond callable at par in 5 years.
- Assumed yield environment: 6\%
- Non-callable 10-year bond price: 114.72
- Callable bond market price: 107.50

**Option Value:**

$$\text{Call Option Value} = 114.72 - 107.50 = 7.22$$

**Interpretation:** The issuer's call option is worth 7.22 points. This represents the value the investor is giving up in exchange for the higher coupon (8\% vs market rates of 6\%).

---

## 6.9 Practical Notes

### 6.9.1 Yield Quoting Conventions

Money market quoting (simple interest with ACT/360) differs from bond quoting (semiannual compounding). Converting between them requires care:

- Semi-annual to continuous: $R_c = 2\ln(1 + y/2)$
- Continuous to semi-annual: $y = 2(e^{R_c/2} - 1)$

> **Note on BEY:** The term "bond-equivalent yield" (BEY) is sometimes used for semi-annual yields, but market usage varies. 

### 6.9.2 Common Pitfalls

**1. Using clean price for YTM:** Settlement cash is the dirty price. Using clean price in the PV equation produces the wrong yield.

**2. Day count mismatch:** If your accrued interest calculation uses one day count but your yield discounting uses another, you'll get inconsistent results.

**3. Treating yield DV01 as curve DV01:** Yield-based sensitivity assumes parallel yield shifts—a specific and often unrealistic assumption.

**4. YTM as expected return:** Holding to maturity will not necessarily earn the initial yield because realized return depends on reinvestment rates and (if sold early) the exit yield/price.

**5. Ignoring optionality:** Yield-based measures assume fixed promised cash flows. Callable bonds, MBS, and other structured products require different treatment.

**6. Confusing duration types:** Modified duration gives percentage sensitivity; Macaulay duration gives weighted-average time. Don't mix them in hedge calculations.

**7. Ignoring convexity for large moves:** Duration alone underestimates gains and overestimates losses. For moves >50 bps, convexity matters.

### 6.9.3 Verification Tests

- **Monotonicity:** Price should decrease as yield increases (for fixed cash flows)
- **Near-par check:** For bonds trading near par, yield should be near the coupon rate
- **Zero-coupon check:** For a zero, YTM should equal the spot rate to that maturity
- **Reprice check:** Plug the computed YTM back and verify you recover the dirty price

---

## Summary

1. **YTM is a single-rate IRR**: the yield $y$ that discounts promised cash flows to the **dirty (invoice) price** under a stated compounding convention.
2. **Settlement cash uses dirty price**: $P_{\text{dirty}} = P_{\text{clean}} + AI$. YTM must be solved from $P_{\text{dirty}}$, not the clean quote.
3. **YTM hides the term structure**: it is a PV-weighted blend of spot rates and cannot diagnose “rich/cheap” at specific maturities.
4. **YTM is not a locked-in return**: realized return depends on coupon reinvestment rates and (if sold early) the exit yield/price.
5. **Horizon/total-return analysis** makes assumptions explicit: horizon, reinvestment rate, and ending yield.
6. **Yield DV01 ($y$ bump object)** measures price sensitivity per 1 bp change in YTM, in points per 100; scale by notional for dollar DV01.
7. **Modified duration** converts DV01 to percentage sensitivity: $\Delta P/P \approx -D_{\text{mod}}\Delta y$.
8. **Macaulay duration** is a PV-weighted average time of cash flows and equals maturity for a zero-coupon bond.
9. **Convexity** captures curvature: for larger moves, duration-only approximations understate gains and overstate losses.
10. **Yield-based hedges are single-factor**: they hedge (near) parallel moves but can fail under twists/butterflies; key-rate and multi-factor methods address this.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| YTM | Single rate that reproduces price | Universal quoting convention |
| Clean/Dirty | $P_{\text{dirty}} = P_{\text{clean}} + \text{AI}$ | Clean is quoted; dirty is exchanged |
| Reinvestment Risk | Coupons reinvest at unknown future rates | YTM ≠ Realized return |
| Yield DV01 ($DV01_y$) | $DV01_y := -\frac{1}{10{,}000}\frac{dP}{dy}$ | Rate risk in desk units (points per 100 per bp) |
| Modified Duration | $-\frac{1}{P}\frac{dP}{dy}$ | Percentage risk per yield change |
| Macaulay Duration | $(1+y/2) \times D_{\text{mod}}$ | Weighted-average time to receipt |
| Convexity | $\frac{1}{P}\frac{d^2P}{dy^2}$ | Curvature benefit; explains asymmetry |
| Cost of Convexity | Higher convexity → Lower yield | No free lunch |
| Negative Convexity | Price ceiling caps upside | Callable bonds, MBS |
| Parallel Shift | All yields move by the same amount | Implicit assumption of yield-based measures |
| Barbell | Short + long maturities | Higher convexity than bullet |
| YTW | min(YTM, all YTCs) | Conservative yield for callables |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $F$ | Face value (par) | currency; default $F=100$ for “per 100” price quoting |
| $c$ | Coupon rate | per year; decimal (e.g., 6\% $\rightarrow 0.06$); semiannual coupon cashflow $=Fc/2$ |
| $P_{\text{clean}}$ | Clean (flat/quoted) price | points per 100 face; quoted on screens |
| $AI$ | Accrued interest | points per 100; day count per instrument (Actual/Actual in-period in Example A) |
| $P_{\text{dirty}}$ | Dirty (full/invoice) price | points per 100; $P_{\text{dirty}}=P_{\text{clean}}+AI$ at settlement |
| $y$ | Yield-to-maturity | nominal annual; semiannual compounding unless stated; in decimals |
| $r=1+y/2$ | Per-period gross discount factor | unitless; one per coupon period (0.5y) |
| $a_i$ | Time from settlement to cashflow $i$ | coupon periods (semiannual), can be fractional |
| $DV01_y$ | Yield DV01 (bump object $y$) | points per 100 per 1 bp; $DV01_y := -\frac{1}{10{,}000}\frac{dP}{dy}$ |
| $DV01_{USD}(N)$ | Dollar DV01 for notional $N$ | currency per 1 bp; $DV01_{USD}(N)=\frac{N}{100}DV01_y$ |
| $D_{\text{mod}}$ | Modified duration | years; $D_{\text{mod}}=-(1/P)\,dP/dy$ |
| $D_{\text{Mac}}$ | Macaulay duration | years; $D_{\text{Mac}}=(1+y/2)D_{\text{mod}}$ (semiannual) |
| $C_y$ | Yield convexity | years squared; $C_y=(1/P)\,d^2P/dy^2$ |
| $P(0,t)$ or $d(t)$ | Discount factor to maturity $t$ | unitless; curve PV: $P=\sum CF_i P(0,t_i)$ |
| $YTC$ | Yield-to-call | YTM-style yield for a specified call scenario |
| $YTW$ | Yield-to-worst | $\min(YTM, YTC_1,YTC_2,\ldots)$ |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is YTM? | The single rate that discounts promised cash flows to the bond's market price |
| 2 | Clean or dirty price for YTM calculation? | Dirty price—it's what the buyer actually pays |
| 3 | Define dirty price | Full price = quoted (clean) price + accrued interest |
| 4 | Why can YTM mislead on relative value? | Higher yield doesn't mean better value; YTM hides term structure details |
| 5 | What is modified duration in yield space? | $D_{\text{mod}} = -\frac{1}{P}\frac{dP}{dy}$ |
| 6 | What is yield DV01 in this chapter? | $DV01_y := -\frac{1}{10{,}000}\frac{dP}{dy}$; units: points per 100 per bp (scale by $N/100$ for dollars) |
| 7 | How to compute yield DV01 numerically? | $DV01_y \approx \frac{P(y-1\text{bp}) - P(y+1\text{bp})}{2}$ |
| 8 | Relationship between DV01 and duration? | $DV01_y = \frac{P \cdot D_{\text{mod}}}{10{,}000}$ |
| 9 | Define yield convexity | $C_y = \frac{1}{P}\frac{d^2P}{dy^2}$ |
| 10 | What does positive convexity imply? | Price gains from yield decreases exceed losses from equal increases |
| 11 | When does "return = yield" hold? | If yield remains unchanged over a short period |
| 12 | Typical bond compounding convention? | Semiannual |
| 13 | Why is reinvestment risk real? | Coupons must be reinvested at unknown future rates, not the initial YTM |
| 14 | What is "curve PV"? | $P = \sum_i CF_i \cdot P(0,t_i)$ using discount factors |
| 15 | Why might yield DV01 ≠ curve DV01? | Yield DV01 assumes one rate; curve DV01 bumps the whole term structure |
| 16 | What assumption underlies yield-based measures? | Parallel yield shifts |
| 17 | Why can't two bonds hedge a third under all curve moves? | Curve risk is multi-factor; yield hedges fail under twists |
| 18 | What is a "reprice check"? | Plug computed YTM back into PV formula and verify it matches dirty price |
| 19 | For near-par bonds, what's a rough YTM estimate? | Near the coupon rate |
| 20 | Why does higher coupon mean lower duration? | More cash flow arrives earlier, reducing weighted-average time |
| 21 | Macaulay duration of a zero-coupon bond? | Equals years to maturity |
| 22 | Perpetuity Macaulay duration formula? | $(1+y/2)/y$ |
| 23 | What is barbelling? | Using short + long maturities instead of intermediate to increase convexity |
| 24 | When does barbell outperform bullet? | When rates move by large amounts (either up or down) |
| 25 | What is the "cost of convexity"? | Higher convexity bonds have lower yields—you pay for the asymmetric payoff |
| 26 | What reinvestment assumption is embedded in YTM? | It implicitly assumes coupons can be reinvested at the original yield, which is rarely true in practice |
| 27 | What is negative convexity? | When the price-yield curve is concave (curves downward); price gains are capped |
| 28 | Which securities exhibit negative convexity? | Callable bonds, mortgage-backed securities |
| 29 | Callable bond decomposition formula? | $P_{\text{callable}} = P_{\text{non-callable}} - \text{Call Option Value}$ |
| 30 | What is Yield-to-Worst (YTW)? | Minimum of YTM and all possible yields-to-call |
| 31 | How does convexity relate to gamma? | Both are second derivatives; long convexity = long gamma = benefit from volatility |
| 32 | Why do liability managers care about reinvestment risk? | Coupon reinvestment rates are unknown, making total return uncertain |
| 33 | Three components of total return? | Coupon income + Reinvestment income + Price change |
| 34 | Which bond type has zero reinvestment risk? | Zero-coupon bonds (no intermediate cash flows) |
| 35 | What does "long 50k DV01" mean? | Position gains USD 50,000 for every 1 bp decline in rates |

---

## Mini Problem Set

1. (Compute) A 5\% semiannual bond has 90 days accrued in a 180-day coupon period. What is accrued interest per 100?
2. (Compute) If $P_{\text{clean}}=99.60$ and $AI=1.10$, compute $P_{\text{dirty}}$.
3. (Concept) Explain why $P(y)$ decreases as $y$ increases for a fixed-coupon bond with promised cash flows.
4. (Desk) Your risk report shows “total DV01 = 0,” but you lose money on a curve flattening. Give one plausible position (e.g., long 5y / short 10y in equal DV01) and name the risk report you should look at next.
5. (Compute) A bond price is 102.00 at yield $y$. At $y+1$ bp, price is 101.95. Approximate $DV01_y$ per 100.
6. (Compute) A bond has $P=105$ and $D_{\text{mod}}=6$. Approximate the price change for a +25 bp move (ignore convexity).
7. (Compute) Same bond has convexity $C_y=50$. Include the convexity correction for a +25 bp move.
8. (Concept) For equal maturity and yield, which has higher modified duration: a high-coupon bond or a low-coupon bond? Explain.
9. (Concept) Give two reasons curve DV01 can differ from yield DV01.
10. (Compute) Compute the Macaulay duration of a perpetuity at 8\% yield (semiannual compounding convention).
11. (Compute) Horizon analysis: you buy a 10-year, 6\% annual coupon bond at par. Immediately after purchase, yields drop to 4\% and stay there; coupons are reinvested at 4\%. What realized annualized return do you earn if you hold to maturity?
12. (Concept) A trader says “I’m paying 12 bps for convexity.” Explain the meaning in a duration-matched comparison.

### Solution Sketches (Selected)
1. Coupon per period $=2.5$. Accrual fraction $=90/180=0.5$. $AI=2.5\times 0.5=\mathbf{1.25}$.
2. $P_{\text{dirty}}=99.60+1.10=\mathbf{100.70}$.
4. Total DV01 only hedges (near) parallel moves. A 5s/10s DV01-neutral position can lose on twists; check **key-rate DV01** (or bucketed DV01) rather than only the scalar total.
5. $DV01_y \approx 102.00-101.95=\mathbf{0.05}$ points per 100 per 1 bp.
6. $\Delta P \approx -P\,D_{\text{mod}}\,\Delta y = -105\times 6\times 0.0025 = \mathbf{-1.575}$.
7. Convexity term $\approx \tfrac12 P C_y (\Delta y)^2 = \tfrac12\times 105\times 50\times (0.0025)^2 \approx 0.0164$. Total $\approx -1.575+0.0164=\mathbf{-1.559}$.
10. $D_{\text{Mac}}=(1+y/2)/y=(1+0.08/2)/0.08=1.04/0.08=\mathbf{13}$ years.
11. Coupons reinvested at 4\%: $FV=6\times\frac{1.04^{10}-1}{0.04}=72.04$. Terminal value $=100+72.04=172.04$. Realized annualized return $=(172.04/100)^{0.1}-1\approx 5.58\%$.

---

## References

- (Bruce Tuckman, *Fixed Income Securities*, “Definition and Interpretation”; “The Relation Between Spot and Forward Rates and the Slope of the Term Structure”; “Yield-Based DV01”; “Yield-Based Convexity”; “A Note on Yield-to-Call”)
- (John C. Hull, *Options, Futures, and Other Derivatives*, “Price Quotations of U.S. Treasury Bonds”; “Day Counts”)
- (Edwin J. Elton, Martin J. Gruber, Stephen J. Brown, and William N. Goetzmann, *Modern Portfolio Theory and Investment Analysis*, “Special Considerations in Bond Pricing”)
- (Dominic O’Kane, *Modeling Single-name and Multi-name Credit Derivatives*, “The Bond DV01”; “Risk Management”)
- (David Ruppert, *Statistics and Data Analysis for Financial Engineering*, “Principal Components Analysis”)
