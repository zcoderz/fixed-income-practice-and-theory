# Chapter 6: Yield-to-Maturity and Yield-Based Risk

---

## Introduction

A portfolio manager glances at two bonds on her screen. The first shows a yield of 5.2%; the second, 4.8%. The higher-yielding bond looks more attractive—40 basis points of extra return for similar credit quality. She buys the first bond.

Has she made a good decision? Not necessarily. The yield-to-maturity is one number summarizing a complex cashflow structure, and that compression can mislead. As Tuckman emphasizes in *Fixed Income Securities*, "yield is not automatically a good measure of relative value or realized return-to-maturity," and "higher yield does not necessarily mean 'better value.'" The manager may have bought a bond with worse convexity, shorter effective duration, or hidden optionality—characteristics that YTM alone cannot reveal.

Yield-to-maturity (YTM) is the fixed income market's most ubiquitous metric. It appears on every trading screen, in every research report, in every risk system. Bonds are quoted by yield as often as by price because the two are equivalent—given one, you can always compute the other. But this convenience creates a trap. Practitioners who treat YTM as "the return you'll earn" or who use yield-based risk measures without understanding their assumptions will eventually be surprised by P&L that doesn't match their expectations.

This chapter has three aims:

1. **Define YTM precisely**—including the clean/dirty price relationship, compounding and day-count conventions, and what YTM actually measures (and what it hides)
2. **Expose the reinvestment fallacy**—the dangerous myth that YTM represents a "locked-in" return
3. **Build yield-based risk measures**—DV01, modified duration, and convexity—and show why they can diverge from curve-based risk when the term structure moves in non-parallel ways

The yield-based framework developed here serves as the foundation for understanding interest rate risk, even as we acknowledge its limitations. Chapter 11 will revisit DV01 in the context of full curve construction, and Chapter 14 will introduce key-rate exposures that address the multi-factor nature of yield curve movements.

---

## 6.1 Yield-to-Maturity as an Internal Rate of Return

### 6.1.1 The Definition

Yield-to-maturity is the single discount rate that, when applied to all of a bond's promised cash flows, reproduces the bond's market price. Tuckman provides the formal definition: "Yield-to-maturity is the single rate such that discounting a security's cash flows at that rate produces the security's market price." For a standard bond with semiannual coupons, the price-yield relationship is expressed as:

$$\boxed{P(y) = \sum_{t=1}^{2T} \frac{c/2}{(1+y/2)^t} + \frac{100}{(1+y/2)^{2T}}}$$

where $c$ is the annual coupon rate, $y$ is the yield-to-maturity (annualized with semiannual compounding), $T$ is years to maturity, and the price $P$ is per 100 face value.

This is an internal rate of return (IRR) problem: given the price and cash flows, find the rate that sets net present value to zero. Tuckman notes that "YTM is solved by trial-and-error or numerical methods"—there is no closed-form solution for a general coupon bond.

**Worked Example (from Tuckman):** The 6¼s of February 15, 2003, at a price of 102-18⅛ on February 15, 2001, has its yield defined by:

$$\frac{3.125}{1+y/2}+\frac{3.125}{(1+y/2)^{2}}+\frac{3.125}{(1+y/2)^{3}}+\frac{103.125}{(1+y/2)^{4}}=102+18.125/32$$

Solving numerically yields $y \approx 4.8875\%$.

### 6.1.2 YTM as a Quoting Convention

Yield-to-maturity compresses the entire term structure of discount rates into one "average discount rate" that reproduces the observed price. This makes YTM extraordinarily useful as a **quoting convention**: traders can move between price and yield with a simple calculation, and yields are easier to compare across bonds of different maturities and coupons than raw prices.

On the trading desk, "bond yields 5.7%" is shorthand for "the single rate $y$ that reprices this bond under our standard compounding convention." Everyone understands the convention, so the communication is efficient. Tuckman notes that "it is easy to move from price to yield and back" and that "yield-to-maturity is often used as an alternate way to quote price." The danger comes when participants forget that YTM is a summary statistic, not a measure of expected return.

> **Desk Reality: YTM as Communication**
>
> When a trader says "I'm bid at 5.25," everyone on the desk immediately knows the price. The yield convention eliminates the need to quote awkward prices like 98-24+ (which would be "98 and 24.5 thirty-seconds"). For relative value discussions, yields provide a common language: "The 10-year is 15 bps cheap to the 7-year" is more informative than comparing prices directly.
>
> The trap: junior traders sometimes interpret "yield" as "return." When your PM asks "what's the yield on that position?" they want a quoting convention, not a return forecast.

### 6.1.3 The Compounding Convention

Bond yields are typically quoted with **semiannual compounding** because most bonds pay coupons twice per year. Under this convention, a yield of $y$ means:

- Each coupon period discounts by factor $1 + y/2$
- Two periods compound to $(1 + y/2)^2$ per year

When settlement doesn't align with regular coupon dates, the convention extends to fractional periods. For a Treasury bond, the actual/actual day count determines how many days remain until the next coupon, and the discounting uses appropriate fractional exponents.

> **Convention Warning:** The exact handling of day counts in yield calculations varies by market and instrument. U.S. Treasuries use actual/actual in period; corporate bonds often use 30/360. Always verify which convention your system applies, as Hull notes that "the day count convention... affects the interest accrued between two coupon payment dates."

### 6.1.4 Key Properties of the Price-Yield Relationship

Tuckman derives several important properties from the price-yield formula (equation 3.4 in his notation):

1. **Par pricing:** When $c = 100y$ and $F = 100$, then $P = 100$. If the coupon rate equals the yield, the bond trades at par.

2. **Premium bonds:** When $c > 100y$, then $P > 100$. If the coupon rate exceeds the yield, the bond trades at a premium. "In exchange for an above-market coupon, investors will demand less than their initial investment at maturity."

3. **Discount bonds:** When $c < 100y$, then $P < 100$. Below-market coupons require a capital gain at maturity.

4. **Perpetuity limit:** As $T \to \infty$, $P = c/y$. "The price of a perpetuity, a bond that pays coupons forever, equals the coupon divided by the yield." For example, at a yield of 5.50%, a 6.50 coupon in perpetuity sells for 6.50/0.055 ≈ 118.18.

5. **Monotonicity:** Price is a decreasing function of yield—higher yields mean lower prices, and vice versa.

---

## 6.2 Clean and Dirty Prices

### 6.2.1 The Fundamental Identity

Bond market convention separates the quoted price from accrued interest:

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + \text{AI}}$$

Tuckman provides the synonyms: the **dirty** price is also called the **full** or **invoice** price—it is what the buyer actually pays. The **clean** price is also called the **flat** or **quoted** price—it is what appears on trading screens.

### 6.2.2 Why Markets Quote Clean Prices

Consider what would happen if markets quoted dirty prices. Between coupon dates, interest accrues mechanically—a 5% coupon bond accrues roughly 2.5 cents per day per $100 face. If quotes reflected this accrual, bond prices would drift upward between coupons and jump down on payment dates, even if nothing changed about the bond's fundamental value.

Clean pricing solves this problem. The clean price strips out the "mechanical" accrual component, so quoted prices reflect only changes in yields and market conditions. Tuckman demonstrates algebraically that "if yield does not change then the quoted price of a bond does not fall as a result of a coupon payment." The jump occurs in accrued interest (which resets to zero), not in the quoted clean price.

### 6.2.3 Computing Accrued Interest

Under the Actual/Actual convention (used for U.S. Treasuries), accrued interest is:

$$\boxed{\text{AI} = \frac{c}{2} \times \frac{\text{days since last coupon}}{\text{days in coupon period}}}$$

where $c/2$ is the semiannual coupon payment per 100 face.

**Example:** A 6% Treasury with a 184-day coupon period. Settlement is 150 days after the last coupon.
- Coupon per period: $6/2 = 3.00$
- Accrual fraction: $150/184 = 0.8152$
- Accrued interest: $3.00 \times 0.8152 = 2.4457$

### 6.2.4 YTM Is Solved from the Dirty Price

This is a critical practical point: when solving for YTM, use the **dirty price** as the target. Tuckman notes that "the only quantity that matters is the invoice price... which the market sets equal to the present value of the future cash flows." The present value of future cash flows must equal the cash the buyer pays, which is the dirty price.

The equation to solve is:

$$P_{\text{dirty}} = \sum_{i} \frac{\text{CF}_i}{(1 + y/2)^{a_i}}$$

where $a_i$ is the time (in semiannual periods) from settlement to cash flow $i$.

---

## 6.3 What YTM Summarizes—and What It Hides

### 6.3.1 The Summary: Level of Discounting

YTM tells you the "average" rate at which the bond's cash flows are discounted. For a given term structure, YTM sits somewhere between the short-term and long-term spot rates, weighted by the present values of the cash flows.

Tuckman explains that "yield-to-maturity is a blend of [spot] rates." For the 6¼s of February 15, 2003, with spot rates of 5.008%, 4.929%, 4.864%, and 4.886% for the four semiannual periods, the yield of 4.8875% is closest to the two-year spot rate of 4.886% "because most of this bond's value comes from its principal payment to be made in two years."

For a zero-coupon bond, this average is trivial—the YTM equals the spot rate to that maturity (under matched compounding conventions). For coupon bonds, YTM blends across multiple maturities.

### 6.3.2 What YTM Hides

**The term structure:** Spot rates differ by maturity. A 10-year Treasury's cash flows should, in principle, be discounted at ten different rates. YTM replaces this structure with a single number. This means YTM cannot reveal whether a bond is "cheap at the front" (early cash flows undervalued) and "rich at the back" (late cash flows overvalued), or vice versa.

**Reinvestment assumptions:** YTM calculations implicitly assume that all intermediate coupons can be reinvested at the same YTM. This is unrealistic. Tuckman warns explicitly that "holding to maturity will not necessarily earn the initial yield" because "reinvestment and changing yields matter." If rates fall after purchase, coupons will be reinvested at lower rates, and the total realized return will fall short of the initial YTM.

**Realized return:** Many practitioners mistakenly interpret YTM as the return they will earn. This is only approximately true under very restrictive conditions—specifically, that the yield remains unchanged throughout the holding period.

### 6.3.3 When YTM Equals Realized Return

Tuckman provides one defensible interpretation: "Perhaps the most appealing interpretation of yield-to-maturity is not recognized as widely as it should be. If a bond's yield-to-maturity remains unchanged over a short time period, that bond's realized total rate of return equals its yield."

This is the **constant yield** or **roll-down** assumption—useful for short-term return attribution, but not a forecast of what will actually happen. Over longer horizons or when yields change, realized returns diverge from initial YTM due to:

- Curve movements (if different parts of the curve move differently)

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

As Tuckman cautions, relative-value screens that rank bonds by yield "can be misleading." A proper relative value analysis requires understanding the full term structure and the specific characteristics of each bond.

---

## 6.4 The Reinvestment Fallacy: Why YTM Is Not a Locked-In Return

### 6.4.1 The Myth of "Locked-In" Returns

One of the most persistent misconceptions in fixed income is that buying a bond and holding it to maturity "locks in" the yield-to-maturity as your return. This is false. Tuckman states explicitly: "The reinvestment assumption implicit in yield-to-maturity is almost certainly incorrect." The YTM calculation assumes every coupon payment can be reinvested at exactly the same rate—an assumption that virtually never holds in practice.

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

Each annual coupon of $8 is reinvested at 8%. At maturity:

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

> **Desk Reality: Why This Matters for Liability Managers**
>
> Insurance companies and pension funds that purchase bonds to "match" long-dated liabilities face reinvestment risk on every coupon. An actuary who assumes the 8% yield will compound for 30 years is making an assumption that cannot be hedged. This is why liability-driven investors increasingly favor zero-coupon bonds or strips for precise liability matching—they eliminate reinvestment risk entirely.
>
> The trade-off: zeros typically yield less than coupon bonds (you pay for the certainty), and zeros have higher duration per dollar invested.

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

> **Practitioner Note:** Total return analysis is standard for portfolio managers making allocation decisions. The question is not "what's the yield?" but "given my view on rates, what's the expected total return over my horizon?" Two bonds with identical YTM can have very different expected total returns depending on the rate scenario.

---

## 6.5 Yield-Based DV01 and Modified Duration

### 6.5.1 DV01: Dollar Value of a Basis Point

DV01 measures the change in bond price for a one-basis-point change in yield. Tuckman introduces DV01 as "an acronym for dollar value of an '01 (i.e., .01%) and gives the change in the value of a fixed income security for a one-basis point decline in rates." The general definition is:

$$\boxed{\text{DV01} = -\frac{\Delta P}{10{,}000 \cdot \Delta y}}$$

The sign convention makes DV01 positive when prices rise as yields fall (the normal case for fixed-coupon bonds). The 10,000 factor converts from "per unit of yield" to "per basis point." Tuckman explains: "The negative sign defines DV01 to be positive if price increases when rates decline and negative if price decreases when rates decline. This convention has been adopted so that DV01 is positive most of the time."

When an explicit price-yield function exists, DV01 can be expressed using derivatives. Differentiating the price-yield formula, Tuckman derives:

$$\text{DV01} = \frac{1}{10{,}000} \times \frac{1}{1+y/2}\left[\sum_{t=1}^{2T} \frac{t}{2} \frac{c/2}{(1+y/2)^t} + T \frac{100}{(1+y/2)^{2T}}\right]$$

In words: "DV01 is the sum of the time-weighted present values of a bond's cash flows divided by 10,000 multiplied by one plus half the yield."

> **Desk Reality: DV01 as the Lingua Franca of Rates**
>
> When a portfolio manager says "I'm long 50k DV01," they mean their position gains approximately $50,000 for every basis point rates fall. This is how risk is communicated on rates desks—not in terms of notional or duration, but in dollar sensitivity.
>
> Why DV01 dominates: A $100 million position in 2-year notes and a $50 million position in 10-year notes cannot be compared by notional. But if the 2-year has DV01 of $19k and the 10-year has DV01 of $42k, you immediately know the 10-year position has more than twice the rate risk.
>
> **Common gotcha:** Make sure everyone is using the same DV01 convention. "DV01 per million" vs "total DV01" vs "DV01 per bp per 100 face" can cause confusion.

### 6.5.2 Computing DV01 Numerically

For practical computation, use a central difference:

$$\boxed{\text{DV01}_y \approx \frac{P(y - 1\text{ bp}) - P(y + 1\text{ bp})}{2}}$$

This is robust and works regardless of whether you have closed-form derivatives. Tuckman notes that "the most stable numerical estimate chooses rates that are equally spaced above and below" the current rate.

### 6.5.3 Modified Duration

Duration measures percentage price sensitivity rather than dollar sensitivity. Hull defines duration as "a measure of how long the holder of the bond has to wait before receiving the present value of the cash payments." More precisely, it is "a weighted average of the times when payments are made, with the weight applied to time $t_i$ being equal to the proportion of the bond's total present value provided by the cash flow at time $t_i$."

Tuckman defines modified duration as:

$$\boxed{D_{\text{mod}} = -\frac{1}{P}\frac{dP}{dy}}$$

This gives the approximate percentage price change per unit change in yield:

$$\frac{\Delta P}{P} \approx -D_{\text{mod}} \cdot \Delta y$$

Hull notes that this relationship "is easy to use and is the reason why duration, first suggested by Frederick Macaulay in 1938, has become such a popular measure."

### 6.5.4 The DV01-Duration Link

DV01 and modified duration are related by:

$$\boxed{\text{DV01}_y = \frac{P \cdot D_{\text{mod}}}{10{,}000}}$$

Given either measure, you can compute the other if you know the price. The distinction matters: DV01 gives dollar risk (useful for hedging notional amounts), while duration gives percentage risk (useful for comparing bonds of different prices).

### 6.5.5 Macaulay Duration and Its Interpretation

Macaulay duration is a simple transformation of modified duration:

$$\boxed{D_{\text{Mac}} = (1 + y/2) \cdot D_{\text{mod}}}$$

Tuckman demonstrates "a convenient property": the Macaulay duration of a $T$-year zero-coupon bond equals $T$. Mathematically:

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
> *   **Coupon Bond**: Small weights (coupons) distributed along the beam. They pull the center of gravity (fulcrum) toward the middle. ($D < T$)
>
> Higher coupons = Heavier weights at the front = Fulcrum moves closer to zero (Lower Duration).

Hull explains that duration is "a weighted average of the times when payments are made." For a 5-year bond with Macaulay duration of 4.4 years, you wait (on a present-value-weighted basis) 4.4 years to receive your money back.

### 6.5.6 How Duration Varies with Coupon and Maturity

Tuckman provides graphical analysis showing:

- **Higher coupon → lower duration:** More cash flow arrives early, pulling the weighted-average time forward
- **Longer maturity → higher duration:** For most bonds, though the effect asymptotes
- **Zero-coupon bonds have duration = maturity:** All cash arrives at one point
- **Perpetuities have finite duration:** At yield $y$, Macaulay duration approaches $(1+y/2)/y$

**Perpetuity Duration Example:** From Tuckman's formulas, as $T \to \infty$:

$$\left.D_{\text{Mac}}\right|_{T=\infty} = \frac{1+y/2}{y}$$

At a 5% yield: $D_{\text{Mac}} = (1.025)/0.05 = 20.5$ years. Despite paying forever, a perpetuity at 5% has finite duration because early payments dominate the present-value weighting.

Luenberger provides additional insight: "Duration does not increase appreciably with maturity. In fact, with fixed yield, duration increases only to a finite limit as maturity is increased." This means that very long durations (of 20 years or more) "are achieved only by bonds that have both very long maturities and very low coupon rates."

---

## 6.6 Yield-Based Convexity

### 6.6.1 Definition and Interpretation

Convexity measures the curvature of the price-yield relationship—how duration itself changes as yields move. Tuckman defines yield-based convexity as:

$$\boxed{C_y = \frac{1}{P}\frac{d^2P}{dy^2}}$$

where $d^2P/dy^2$ is the second derivative of the price-yield function. Tuckman notes: "Mathematically, convexity is defined as" this second-derivative expression, and "just as the first derivative measures how price changes with rates, the second derivative measures how the first derivative changes with rates."

Convexity explains the asymmetry in price changes: when yields fall, prices rise by more than duration alone would predict; when yields rise, prices fall by less. This asymmetry favors bondholders.

### 6.6.2 The Duration-Convexity Approximation

The second-order Taylor expansion gives:

$$\boxed{\Delta P \approx -P \cdot D_{\text{mod}} \cdot \Delta y + \frac{1}{2} P \cdot C_y \cdot (\Delta y)^2}$$

Tuckman notes that for small yield changes, "the duration term... is much larger than the convexity term." But for larger moves, convexity becomes meaningful. In his numerical example with a 25bp move, the duration term is about 30% while the convexity term is about 3%—convexity is a correction, not the main effect.

### 6.6.3 Positive Convexity as a Benefit

For plain-vanilla fixed-rate bonds, convexity is positive ($C_y > 0$). This means:

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

Convexity is valuable, and markets price it accordingly. Tuckman emphasizes: "Bonds are priced to reflect their convexity advantage." An investor who wants more convexity must accept a lower yield—there is no free lunch.

The economic intuition is straightforward: if Bond A and Bond B have the same duration but Bond A has higher convexity, Bond A will outperform in large rate moves (either direction). The market recognizes this and bids up the price of Bond A, lowering its yield.

**Quantifying the trade-off:** Consider two duration-matched portfolios:
- Portfolio A: High convexity, yield = 5.00%
- Portfolio B: Low convexity, yield = 5.15%

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

Tuckman develops an important example demonstrating how portfolio structure affects convexity. An asset-liability manager with liabilities having 9-year duration could fund with:

- **Bullet portfolio:** Intermediate-maturity bonds with 9-year duration
- **Barbell portfolio:** A mix of short and long bonds (e.g., 2-year and 30-year) that also has 9-year portfolio duration

The barbell has higher convexity because convexity increases with the *square* of maturity. Using zero-coupon bonds, Tuckman provides a specific numerical example:

**Example: 75%/25% Barbell vs 9-Year Bullet**

At a 5% yield:
- 2-year zero: Duration = 2, Convexity ≈ 4
- 30-year zero: Duration = 30, Convexity ≈ 871
- 9-year zero (bullet): Duration = 9, Convexity ≈ 81

A barbell of 75% in 2-year zeros and 25% in 30-year zeros:
- Portfolio duration: $0.75 \times 2 + 0.25 \times 30 = 9$ years ✓
- Portfolio convexity: $0.75 \times 4 + 0.25 \times 871 = 3 + 218 = 221$

The barbell has convexity of **221** versus the bullet's **81**—nearly three times as much convexity for the same duration.

**Performance comparison:**

| Rate Move | Bullet Performance | Barbell Performance | Winner |
|-----------|-------------------|---------------------|--------|
| Small (±25 bp) | Better | Worse | Bullet |
| Medium (±100 bp) | Similar | Similar | Tie |
| Large (±200 bp) | Worse | Better | Barbell |

Tuckman explains: "The bullet outperforms if rates move by a relatively small amount, up or down, while the barbell outperforms if rates move by a relatively large amount."

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

Convexity increases with the **square** of maturity—much faster than duration, which increases approximately linearly. This is why long-dated zeros have enormous convexity. A 30-year zero at 5% yield has convexity of approximately:

$$C = \frac{30 \times 30.5}{(1.025)^2} \approx 871$$

### 6.6.8 Computing Convexity Numerically

With a finite-difference approach using $\Delta = 50$ bp:

$$\boxed{C_y \approx \frac{P(y - \Delta) + P(y + \Delta) - 2P(y)}{P(y) \cdot \Delta^2}}$$

Tuckman notes that "extra precision is often necessary when calculating second derivatives" because the second-order effect is small.

### 6.6.9 Negative Convexity and Callable Bonds

Not all fixed income securities exhibit positive convexity. Tuckman notes that "fixed income securities need not be positively convex at all rate levels. Some important examples of negative convexity are callable bonds... and mortgage-backed securities."

**The Callable Bond Pricing Relationship:**

A callable bond can be decomposed as:

$$\boxed{P_{\text{callable}} = P_{\text{non-callable}} - \text{Value of Call Option}}$$

The issuer owns an embedded call option—the right to redeem the bond early at a specified price (typically par). This option becomes valuable when rates fall significantly below the coupon rate.

**Why Callable Bonds Have Negative Convexity:**

When rates fall substantially:
- A non-callable bond's price rises without limit
- A callable bond's price is capped near the call price—the issuer will call the bond

This cap eliminates the upside that positive convexity provides. In the region where rates are low enough that the call is likely, the price-yield curve bends *downward* (concave), creating negative convexity.

Tuckman provides a numerical example showing a callable bond with convexity of **-223** when the call is deep in the money, compared to positive convexity for the same structure when rates are high.

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

$$\boxed{\text{YTW} = \min(\text{YTM}, \text{YTC}_1, \text{YTC}_2, ...)}$$

Tuckman explains that "investors will most likely use yield-to-worst—that is, the yield given the redemption price schedule that minimizes value to the investor."

> **Practitioner Note:** Yield-to-worst is conservative but not always accurate. It assumes the issuer will act optimally against the investor. In practice, issuers sometimes don't call bonds when it would be optimal (refinancing frictions, transaction costs). The "worst" case may not materialize.

---

## 6.7 Curve-Based Risk: A Preview

### 6.7.0 The Three Drivers of Yields (PCA)

Before simplifying to "one factor," know that 90%+ of yield curve movements can be explained by three "Principal Components" (see Chapter 16):

1.  **Level (Shift)**: The whole curve moves up or down parallel. (~80-90% of variance). Duration protects you here.
2.  **Slope (Twist)**: The curve steepens (long rates up, short rates down) or flattens. Duration *fails* here.
3.  **Curvature (Butterfly)**: The belly (5y) moves differently than the wings (2y/30y).

Yield-based risk (Duration/DV01) implicitly assumes **#1 only**. It is blind to #2 and #3.

### 6.7.1 The Single-Factor Assumption

All yield-based measures assume that a bond's price depends on one number: its yield. This is convenient but dangerous. Tuckman warns that "a major weakness of the approach taken in Chapters 5 and 6... is the assumption that movements in the entire term structure can be described by one interest rate factor. To put it bluntly, the change in the six-month rate is assumed to predict perfectly the change in the 10-year and 30-year rates."

In reality, the price of a coupon bond depends on the entire term structure:

$$P = \sum_{i} \text{CF}_i \cdot P(0, t_i)$$

where $P(0,t_i)$ is the discount factor for maturity $t_i$.

### 6.7.2 Curve DV01

Under a full curve framework, we can define curve DV01 as the sensitivity to a parallel bump in spot rates. For continuously compounded spot rates $\hat{r}(t)$:

$$d(t) = \exp(-\hat{r}(t) \cdot t)$$

A 1-bp parallel bump gives:

$$d_{\Delta}(t) = d(t) \cdot \exp(-\Delta \cdot t)$$

The curve DV01 is then $P_{\text{base}} - P_{\text{bumped}}$.

### 6.7.3 When Yield DV01 and Curve DV01 Differ

Yield DV01 assumes the bond's yield changes. Curve DV01 bumps all spot rates uniformly. These can differ because:

1. YTM is an average rate; bumping it doesn't replicate bumping each individual spot rate
2. The weighting of maturities differs between the two approaches

In practice, the differences are usually small for straightforward bonds but become material for:

- Bonds priced at significant premiums or discounts
- Portfolios mixing short and long maturities
- Hedges involving different instruments

### 6.7.4 Why Yield Hedges Fail Under Curve Twists

A yield-DV01 hedge matches the DV01 of bond A with bond B. This works if both yields change by the same amount. But Tuckman emphasizes in Chapter 7: "A major weakness of the approach... is the assumption that movements in the entire term structure can be described by one interest rate factor."

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

### Example A: Computing YTM from Clean Price

**Bond:**
- U.S. Treasury with semiannual coupons (Feb 15 and Aug 15)
- Coupon rate: 6% ($c/2 = 3.00$ per period)
- Maturity: February 15, 2031
- Settlement: January 12, 2026
- Quoted clean price: 101.25

**Step 1: Compute accrued interest**

- Last coupon: August 15, 2025
- Next coupon: February 15, 2026
- Days in coupon period: 184 (Aug 15 → Feb 15)
- Days accrued: 150 (Aug 15 → Jan 12)

$$\text{AI} = 3.00 \times \frac{150}{184} = 2.4457$$

**Step 2: Compute dirty price**

$$P_{\text{dirty}} = 101.25 + 2.4457 = 103.6957$$

**Step 3: Set up the YTM equation**

Let $r = 1 + y/2$. Cash flows and fractional periods from settlement:

| Payment Date | Cash Flow | Days to Payment | Periods (approx) |
|--------------|-----------|-----------------|------------------|
| Feb 15, 2026 | 3 | 34 | 0.1863 |
| Aug 15, 2026 | 3 | 215 | 1.1781 |
| Feb 15, 2027 | 3 | 399 | 2.1863 |
| ... | ... | ... | ... |
| Feb 15, 2031 | 103 | 1860 | 10.1918 |

Solve:

$$\sum_{i=1}^{11} \frac{\text{CF}_i}{r^{a_i}} = 103.6957$$

**Step 4: Numerical solution**

Using Newton-Raphson or bisection: $y \approx 5.709\%$

$$\boxed{\text{YTM} \approx 5.709\% \text{ (semiannual compounding)}}$$

**Verification:** At $y = 5.709\%$, the PV equals 103.6944, matching the dirty price within rounding tolerance.

---

### Example B: Newton-Raphson Iteration for YTM

Starting from Example A, let's trace one Newton-Raphson step.

**Setup:**
- Target: $P_{\text{dirty}} = 103.6957$
- Initial guess: $y_0 = 6.00\%$
- At $y_0$: $P(y_0) = 102.4232$
- Error: $f(y_0) = 102.4232 - 103.6957 = -1.2725$

**Derivative:**

$$\frac{dP}{dy} = -\frac{1}{2r} \sum_i a_i \cdot \text{CF}_i \cdot r^{-a_i} \approx -433.55$$

**Newton step:**

$$y_1 = y_0 - \frac{f(y_0)}{f'(y_0)} = 0.06 - \frac{-1.2725}{-433.55} = 0.05707$$

The next iterate is 5.707%, very close to the solution.

**Sanity check:** Since $f(y_0) < 0$ (PV too low), we need a lower yield to increase PV. The step correctly moves downward.

---

### Example C: Price-Yield Curve and Convexity

Using the Example A bond at $y^* = 5.709\%$:

| Yield | Price |
|-------|-------|
| 5.209% ($y^* - 50$ bp) | 105.9240 |
| 5.709% ($y^*$) | 103.6957 |
| 6.209% ($y^* + 50$ bp) | 101.5219 |

**Asymmetry observation:**
- Down-move gain: $+2.228$
- Up-move loss: $-2.174$

The gain exceeds the loss—this is positive convexity.

**Convexity estimate:**

$$C_y \approx \frac{105.9240 + 101.5219 - 2(103.6957)}{103.6957 \times (0.005)^2} \approx 21.1$$

---

### Example D: DV01 via Finite Difference vs Duration

**Bond:** Example A at $y^* = 5.709\%$, $P^* = 103.6957$

**Finite-difference DV01:**
- $P(y^* - 1\text{ bp}) = 103.7384$
- $P(y^* + 1\text{ bp}) = 103.6504$

$$\text{DV01}_y = \frac{103.7384 - 103.6504}{2} = 0.0440$$

**Duration from DV01:**

$$D_{\text{mod}} = \frac{\text{DV01} \times 10{,}000}{P} = \frac{0.0440 \times 10{,}000}{103.6957} = 4.24$$

**Duration approximation for +1 bp:**

$$\Delta P \approx -103.6957 \times 4.24 \times 0.0001 = -0.0440$$

Matches the finite-difference result.

---

### Example E: Yield DV01 vs Curve DV01

**Bond:** 2-year, 4% coupon, semiannual

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

$$\text{DV01}_{\text{curve}} = 101.186 - 101.166 = 0.0197$$

**Yield DV01:**

Solve for YTM: $y \approx 3.382\%$

$$\text{DV01}_y \approx 0.0193$$

**Comparison:** The two differ by about 2%. This gap arises because curve DV01 weights longer maturities more heavily (through the $t$ factor in $\exp(-\Delta t)$), while yield DV01 treats all cash flows through one rate.

---

### Example F: Same YTM, Different Risk

Two 5-year bonds both yielding 5%:

| | Bond A (8% coupon) | Bond B (2% coupon) |
|---|---|---|
| Price | 113.13 | 86.87 |
| $D_{\text{mod}}$ | 4.17 | 4.65 |
| DV01 | 0.0472 | 0.0404 |
| $C_y$ | 21.1 | 24.5 |

Despite identical YTM, Bond B has higher duration and convexity because its cash flows are more concentrated at maturity (lower coupons mean more principal-weighted value). YTM alone cannot distinguish these risk profiles.

---

### Example G: Perpetuity Duration

Consider a perpetuity paying $5 annually (semiannual payments of $2.50) at a 6% yield.

**Price:** $P = c/y = 5/0.06 = 83.33$ (per 100 face)

**Macaulay Duration:**

$$D_{\text{Mac}} = \frac{1+y/2}{y} = \frac{1.03}{0.06} = 17.17 \text{ years}$$

**Modified Duration:**

$$D_{\text{mod}} = \frac{D_{\text{Mac}}}{1+y/2} = \frac{17.17}{1.03} = 16.67$$

**DV01:**

$$\text{DV01} = \frac{83.33 \times 16.67}{10{,}000} = 0.139$$

Despite infinite maturity, the perpetuity has a finite 17-year duration because early payments dominate the present-value weighting.

---

### Example H: Reinvestment Risk Calculation

**Bond:** 5-year, 6% annual coupon, purchased at par (YTM = 6%)

**Scenario:** Rates fall to 4% immediately after purchase and remain there.

**Step 1: Future value of reinvested coupons at 4%**

$$FV_{\text{coupons}} = 6 \times \frac{(1.04^5 - 1)}{0.04} = 6 \times 5.416 = 32.50$$

**Step 2: Total terminal value**

$$\text{Terminal Value} = 100 + 32.50 = 132.50$$

**Step 3: Realized return**

$$r = (132.50/100)^{1/5} - 1 = 5.79\%$$

**Shortfall:** The realized return of 5.79% is 21 bps below the initial YTM of 6.00%.

---

### Example I: Callable Bond Decomposition

**Setup:** A 10-year, 8% corporate bond callable at par in 5 years.
- Current yield environment: 6%
- Non-callable 10-year bond price: 114.72
- Callable bond market price: 107.50

**Option Value:**

$$\text{Call Option Value} = 114.72 - 107.50 = 7.22$$

**Interpretation:** The issuer's call option is worth 7.22 points. This represents the value the investor is giving up in exchange for the higher coupon (8% vs market rates of 6%).

---

## 6.9 Practical Notes

### 6.9.1 Yield Quoting Conventions

Tuckman distinguishes **money market quoting** (simple interest with ACT/360) from **bond quoting** (semiannual compounding). Converting between them requires care:

- Semi-annual to continuous: $R_c = 2\ln(1 + y/2)$
- Continuous to semi-annual: $y = 2(e^{R_c/2} - 1)$

> **Note on BEY:** The term "bond-equivalent yield" (BEY) is sometimes used for semi-annual yields, but market usage varies. I'm not sure precisely which desks or markets label the semiannual-quoted bond yield as "BEY" without additional context.

### 6.9.2 Common Pitfalls

**1. Using clean price for YTM:** Settlement cash is the dirty price. Using clean price in the PV equation produces the wrong yield.

**2. Day count mismatch:** If your accrued interest calculation uses one day count but your yield discounting uses another, you'll get inconsistent results.

**3. Treating yield DV01 as curve DV01:** Yield-based sensitivity assumes parallel yield shifts—a specific and often unrealistic assumption.

**4. YTM as expected return:** Tuckman warns that "holding to maturity will not necessarily earn the initial yield" due to reinvestment and changing rates.

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

1. **YTM is the single rate** that discounts promised cash flows to the market price—a convenient summary statistic but not a return forecast.

2. **Clean price plus accrued interest equals dirty price.** YTM must be solved from the dirty price.

3. **Bond yields use semiannual compounding** by convention. Fractional-period exponents handle irregular dates.

4. **YTM hides the term structure.** It cannot reveal whether individual cash flows are cheap or rich relative to their appropriate spot rates.

5. **The reinvestment assumption is almost certainly incorrect.** YTM is not a "locked-in" return; realized returns depend on actual reinvestment rates.

6. **A defensible short-horizon interpretation:** If yield is unchanged over a short period, realized return approximately equals yield.

7. **Yield DV01** measures price change per 1 bp in YTM; computed via finite differences or from duration.

8. **Modified duration** is the percentage price sensitivity: $D_{\text{mod}} = -\frac{1}{P}\frac{dP}{dy}$, with units of years.

9. **Macaulay duration** equals the maturity of a zero with the same price sensitivity; for a zero, it equals years to maturity.

10. **Convexity** ($C_y = \frac{1}{P}\frac{d^2P}{dy^2}$) captures the curvature benefit: gains from rate decreases exceed losses from equal rate increases.

11. **Bonds are priced to reflect their convexity advantage.** Higher convexity means lower yield—there's no free lunch.

12. **Barbell vs bullet:** Spreading out cash flows (barbelling) increases convexity without changing duration, but costs yield.

13. **Callable bonds have negative convexity** when the call is in the money, because the price ceiling caps upside.

14. **Convexity is analogous to gamma** in options—both measure second-derivative exposure and benefit from volatility.

15. **Yield-based hedges can fail** when curve shape moves. Parallel-yield assumptions are a strong restriction; multi-factor approaches provide better protection.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| YTM | Single rate that reproduces price | Universal quoting convention |
| Clean/Dirty | $P_{\text{dirty}} = P_{\text{clean}} + \text{AI}$ | Clean is quoted; dirty is exchanged |
| Reinvestment Risk | Coupons reinvest at unknown future rates | YTM ≠ Realized return |
| DV01 | $-\frac{1}{10{,}000}\frac{dP}{dy}$ | Dollar risk per basis point |
| Modified Duration | $-\frac{1}{P}\frac{dP}{dy}$ | Percentage risk per yield change |
| Macaulay Duration | $(1+y/2) \times D_{\text{mod}}$ | Weighted-average time to receipt |
| Convexity | $\frac{1}{P}\frac{d^2P}{dy^2}$ | Curvature benefit; explains asymmetry |
| Cost of Convexity | Higher convexity → Lower yield | No free lunch |
| Negative Convexity | Price ceiling caps upside | Callable bonds, MBS |
| Parallel Shift | All yields move by the same amount | Implicit assumption of yield-based measures |
| Barbell | Short + long maturities | Higher convexity than bullet |
| YTW | min(YTM, all YTCs) | Conservative yield for callables |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $F$ | Face value (par), default 100 |
| $c$ | Annual coupon rate (in currency per 100 notional per year) |
| $P_{\text{clean}}$ | Quoted (flat/clean) price per 100 |
| $P_{\text{dirty}}$ | Full (dirty) price: $P_{\text{clean}} + \text{AI}$ |
| $\text{AI}$ | Accrued interest per 100 |
| $y$ | Yield-to-maturity (annualized, semiannual compounding) |
| $r = 1 + y/2$ | Per-period gross discount factor |
| $a_i$ | Fractional periods from settlement to cash flow $i$ |
| $\text{DV01}_y$ | Yield DV01: price change per 1 bp in YTM |
| $D_{\text{mod}}$ | Modified duration |
| $D_{\text{Mac}}$ | Macaulay duration |
| $C_y$ | Yield convexity |
| $P(0,t)$ or $d(t)$ | Discount factor to maturity $t$ |
| $\text{YTC}$ | Yield-to-call |
| $\text{YTW}$ | Yield-to-worst |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is YTM? | The single rate that discounts promised cash flows to the bond's market price |
| 2 | Clean or dirty price for YTM calculation? | Dirty price—it's what the buyer actually pays |
| 3 | Define dirty price | Full price = quoted (clean) price + accrued interest |
| 4 | Why can YTM mislead on relative value? | Higher yield doesn't mean better value; YTM hides term structure details |
| 5 | What is modified duration in yield space? | $D_{\text{mod}} = -\frac{1}{P}\frac{dP}{dy}$ |
| 6 | What is DV01? | Price change per 1 bp move in rates: $-\frac{\Delta P}{10{,}000 \cdot \Delta y}$ |
| 7 | How to compute yield DV01 numerically? | $\text{DV01}_y \approx \frac{P(y-1\text{bp}) - P(y+1\text{bp})}{2}$ |
| 8 | Relationship between DV01 and duration? | $\text{DV01} = \frac{P \cdot D_{\text{mod}}}{10{,}000}$ |
| 9 | Define yield convexity | $C_y = \frac{1}{P}\frac{d^2P}{dy^2}$ |
| 10 | What does positive convexity imply? | Price gains from yield decreases exceed losses from equal increases |
| 11 | When does "return = yield" hold? | If yield remains unchanged over a short period |
| 12 | Typical bond compounding convention? | Semiannual |
| 13 | Why is reinvestment risk real? | Coupons must be reinvested at unknown future rates, not the initial YTM |
| 14 | What is "curve PV"? | $P = \sum_i \text{CF}_i \cdot P(0,t_i)$ using discount factors |
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
| 26 | Tuckman quote on reinvestment assumption? | "The reinvestment assumption implicit in yield-to-maturity is almost certainly incorrect" |
| 27 | What is negative convexity? | When the price-yield curve is concave (curves downward); price gains are capped |
| 28 | Which securities exhibit negative convexity? | Callable bonds, mortgage-backed securities |
| 29 | Callable bond decomposition formula? | $P_{\text{callable}} = P_{\text{non-callable}} - \text{Call Option Value}$ |
| 30 | What is Yield-to-Worst (YTW)? | Minimum of YTM and all possible yields-to-call |
| 31 | How does convexity relate to gamma? | Both are second derivatives; long convexity = long gamma = benefit from volatility |
| 32 | Why do liability managers care about reinvestment risk? | Coupon reinvestment rates are unknown, making total return uncertain |
| 33 | Three components of total return? | Coupon income + Reinvestment income + Price change |
| 34 | Which bond type has zero reinvestment risk? | Zero-coupon bonds (no intermediate cash flows) |
| 35 | What does "long 50k DV01" mean? | Position gains $50,000 for every 1 bp decline in rates |

---

## Mini Problem Set

**1.** A 5% semiannual bond has 90 days accrued in a 180-day coupon period. What is accrued interest per 100?

**Solution:** Coupon per period = 2.5. Accrual fraction = 90/180 = 0.5. AI = 2.5 × 0.5 = **1.25**.

---

**2.** If $P_{\text{clean}} = 99.60$ and AI = 1.10, compute $P_{\text{dirty}}$.

**Solution:** $P_{\text{dirty}} = 99.60 + 1.10 =$ **100.70**.

---

**3.** Explain why $P(y)$ decreases as $y$ increases for a fixed-coupon bond.

**Solution:** Each cash flow is discounted by $(1+y/2)^{\text{positive exponent}}$. Increasing $y$ increases the denominator, reducing each present value term.

---

**4.** You solved YTM and got $y = 4.2\%$. What verification should you perform?

**Solution:** Plug $y$ into the PV formula and confirm the result matches the dirty price to your tolerance.

---

**5.** A bond price is 102.00 at yield $y$. At $y + 1$ bp, price is 101.95. Approximate DV01.

**Solution:** $\text{DV01} \approx 102.00 - 101.95 =$ **0.05** per 100 face.

---

**6.** A bond has $P = 105$ and $D_{\text{mod}} = 6$. Approximate the price change for +25 bp.

**Solution:** $\Delta P \approx -105 \times 6 \times 0.0025 =$ **-1.575**.

---

**7.** Same bond with convexity $C_y = 50$. Include the convexity correction.

**Solution:** Convexity term = $\frac{1}{2} \times 105 \times 50 \times (0.0025)^2 = 0.0164$. Total: $-1.575 + 0.016 =$ **-1.559**.

---

**8.** A 2-year 4% semiannual bond has dirty price 100.50. Estimate yield.

**Solution:** Price > 100 with 4% coupon implies yield < 4%. Since premium is small (0.5%), yield is slightly below 4%. Estimate: $y \approx 3.75\%$. (Full solution requires numerical iteration.)

---

**9.** For equal maturity and yield, which has higher modified duration: a high-coupon or low-coupon bond? Explain.

**Solution:** Low-coupon bond. Lower coupons mean a larger fraction of the total value comes from the principal repayment at maturity, pushing the weighted-average time (duration) further out.

---

**10.** Give two reasons curve DV01 can differ from yield DV01.

**Solution:** (1) YTM is an average rate; bumping it doesn't equal bumping each spot rate; (2) Maturity weightings differ between the approaches.

---

**11.** Define precisely what "parallel 1 bp bump" means for continuously compounded spot rates.

**Solution:** $r_{\text{new}}(t) = r_{\text{old}}(t) + 0.0001$ for all $t$.

---

**12.** Explain why a single YTM cannot identify whether a bond is rich or cheap at specific maturities.

**Solution:** YTM averages the discount rates. A bond could be cheap at short maturities and rich at long maturities, but YTM smooths this into one number.

---

**13.** Describe a scenario where a yield-DV01 matched hedge loses money.

**Solution:** A curve twist/steepening/flattening. E.g., Long 10y, Short 2y. Yields change non-parallelly (2y up, 10y down). The hedge ratio assumed parallel moves.

---

**14.** Compute the Macaulay duration of a perpetuity at 8% yield.

**Solution:** $D_{\text{Mac}} = (1+0.08/2)/0.08 = 1.04/0.08 = 13$ years.

---

**15.** An investor buys a 10-year, 6% annual coupon bond at par. If rates immediately drop to 4% and stay there, what is the realized return?

**Solution:**
- Coupons reinvested at 4%: $FV = 6 \times \frac{1.04^{10}-1}{0.04} = 72.04$
- Terminal value: $100 + 72.04 = 172.04$
- Return: $(172.04/100)^{0.1} - 1 = 5.58\%$ (vs initial YTM of 6%)

---

**16.** A callable bond trades at 104. The equivalent non-callable bond trades at 112. What is the embedded call option worth?

**Solution:** Call option value = $112 - 104 =$ **8 points**.

---

**17.** Explain why a barbell has higher convexity than a bullet with the same duration.

**Solution:** Convexity increases with the square of maturity. A barbell combining short and long maturities has more weight at the extremes, where convexity per dollar is highest. The long-maturity component contributes disproportionately to portfolio convexity.

---

**18.** A trader says "I'm paying 12 bps for convexity." Explain what this means.

**Solution:** The trader is accepting a yield 12 bps lower on a high-convexity position compared to a duration-matched low-convexity alternative. The 12 bps is the price of the more favorable asymmetric payoff profile.

---

## Source Map

### (A) Book-Verified Facts

| Fact | Source |
|------|--------|
| YTM is the single rate that discounts promised cash flows to market price | Tuckman Ch 3 |
| "Yield-to-maturity is often used as an alternate way to quote price" | Tuckman Ch 3 |
| YTM is solved by trial-and-error or numerical methods | Tuckman Ch 3 |
| "The reinvestment assumption implicit in yield-to-maturity is almost certainly incorrect" | Tuckman Ch 3 |
| "Holding to maturity will not necessarily earn the initial yield" | Tuckman Ch 3 |
| Dirty price = clean price + accrued interest | Tuckman Ch 4 |
| If yield unchanged, quoted price continuous across coupon dates | Tuckman Ch 4 |
| Semiannual compounding convention for bond yields | Tuckman Ch 3 |
| If yield unchanged over short period, realized return = yield | Tuckman Ch 3 |
| YTM is a blend of spot rates | Tuckman Ch 3 |
| DV01 = $-\frac{1}{10{,}000}\frac{dP}{dy}$ | Tuckman Ch 5-6 |
| "DV01 is an acronym for dollar value of an '01" | Tuckman Ch 5 |
| Modified duration = $-\frac{1}{P}\frac{dP}{dy}$ | Tuckman Ch 6, equation (6.12) |
| Macaulay duration = $(1+y/2) \times D_{\text{mod}}$ | Tuckman Ch 6, equation (6.16) |
| Macaulay duration of a zero equals years to maturity | Tuckman Ch 6, equation (6.19) |
| Duration is weighted-average time of cash flows | Hull Ch 4; Luenberger Ch 3 |
| Duration first suggested by Macaulay in 1938 | Hull Ch 4 |
| Yield convexity = $\frac{1}{P}\frac{d^2P}{dy^2}$ | Tuckman Ch 6, equation (6.33) |
| Convexity of zero = $T(T+0.5)/(1+y/2)^2$ | Tuckman Ch 6, equation (6.36) |
| Perpetuity price = $c/y$ | Tuckman Ch 3 |
| Perpetuity Macaulay duration = $(1+y/2)/y$ | Tuckman Ch 6, equation (6.31) |
| "Major weakness... movements in entire term structure described by one factor" | Tuckman Ch 7 |
| Barbell has greater convexity than bullet | Tuckman Ch 6 |
| 75%/25% barbell of 2y/30y zeros has convexity ~221 vs 9y bullet ~81 | Tuckman Ch 6 |
| "Spreading out cash flows raises convexity" | Tuckman Ch 6 |
| "Bonds are priced to reflect their convexity advantage" | Tuckman Ch 6 |
| Higher coupon → lower duration | Tuckman Ch 6 |
| Convexity increases with square of maturity | Tuckman Ch 6 |
| Duration asymptotes to finite limit as maturity → ∞ | Tuckman Ch 6; Luenberger Ch 3 |
| Callable bonds and MBS exhibit negative convexity | Tuckman Ch 6 |
| Callable bond price = non-callable - call option value | Tuckman Ch 5 |
| Callable bond can have convexity of -223 when call is in the money | Tuckman Table 5.4 |
| Yield-to-worst is yield given redemption that minimizes investor value | Tuckman Ch 5 |

### (B) Claude-Extended Content (Practitioner Knowledge)

| Content | Context |
|---------|---------|
| DV01 as "lingua franca of rates" | Extended from general fixed income practice |
| "Long 50k DV01" interpretation | Standard desk communication |
| Total return analysis approach | Standard portfolio management practice |
| Barbell vs butterfly trade structure | Standard rates trading strategy |
| P&L break examples from curve moves | Common operational experience |
| Cost of convexity as yield give-up | Derived from Tuckman + trading practice |
| Convexity-gamma analogy | Standard quantitative finance parallel |
| Callable bond trading conventions (YTC vs YTM) | Market practice |
| Liability manager concerns about reinvestment | Insurance/pension industry practice |

### (C) Reasoned Inference (Derived from A or B)

| Inference | Derivation Logic |
|-----------|------------------|
| YTM is an "average discount rate" | Follows from IRR definition—single rate matching price; confirmed by Tuckman's "blend of spot rates" |
| Clean price avoids mechanical accrual drift | Follows from $P_{\text{dirty}} = P_{\text{clean}} + \text{AI}$ and Tuckman's continuity proof |
| Convexity benefit for plain vanilla bonds | Follows from $C_y > 0$ and Taylor expansion |
| Yield-DV01 hedge failure under twists | Follows from single-factor vs multi-factor logic |
| Near-par yield ≈ coupon rate | Follows from pricing equation structure |
| Zero-coupon YTM = spot rate | Follows from single cash flow discounting |
| DV01-duration relationship | Algebraic derivation from definitions |
| Reinvestment shortfall calculation | Derived from compound interest formulas |
| Negative convexity from price ceiling | Follows from call option decomposition |

### (D) Flagged Uncertainties

| Content | Uncertainty |
|---------|-------------|
| "BEY" terminology in practice | Not explicitly defined in cited sources; desk/market usage varies; I'm not sure which markets label the semiannual-quoted bond yield as "BEY" |
| Exact day-count handling in yield calculations | Market-specific variations exist (actual/actual vs 30/360); verify for your desk |
| Fractional-period discounting conventions | Tuckman shows actual/actual for Treasuries; corporate bonds may differ |
| Precise convexity figures for specific callable structures | Depend on option modeling assumptions; values shown are illustrative |
