# Chapter 12: Duration — Macaulay, Modified, and the Connection to DV01

---

## Introduction

Your risk report arrives. The portfolio shows "Duration: 7.2 years." Your boss asks, "What does that mean for our P&L if rates move 25 basis points?" If you cannot answer this question instantly and correctly, you will struggle to bridge from middle office to the trading desk.

Duration is the language of interest rate risk. When a portfolio manager says "I'm running 7 years of duration," they mean their position will gain (or lose) approximately 7% for every 100 basis points rates move. When a trader says they're "long 50k DV01 in 10-year equivalents," they're expressing risk in duration-scaled units. When the risk committee reviews interest rate sensitivity, they're looking at duration metrics. Understanding duration deeply—not just as a formula but as an intuition—is essential for anyone managing fixed income portfolios.

The stakes are high. A 10 basis point move on a $100 million portfolio with 7-year duration produces approximately $700,000 of P&L. Misunderstand duration by just one year, and your hedges are wrong by 14%. This creates unexplained P&L, risk limit breaches, and—in severe cases—career consequences. The trader who doesn't understand why a deep discount bond can have duration *exceeding* a perpetuity will be caught off-guard when markets move.

This chapter builds the complete machinery of duration. We begin with **Macaulay duration**—the original concept invented by Frederick Macaulay in 1938—as a weighted-average time to receipt of cash flows. We then derive **modified duration**, showing exactly how it measures percentage price sensitivity. We examine **special cases**: zeros, par bonds, and perpetuities that serve as benchmarks. We explore how duration varies with yield, coupon, and maturity, including the surprising **deep discount paradox**. We introduce **dollar duration**, which is how risk systems actually aggregate exposure. We show how to compute **portfolio duration** and avoid the common aggregation trap. We demonstrate **immunization**, one of the most widely used analytical techniques in investment management. Finally, we connect duration to **Value-at-Risk** calculations that drive regulatory capital.

Chapter 11 introduced DV01 as the dollar measure of risk. This chapter normalizes that dollar measure into a standardized, position-size-independent quantity that allows comparing risks across bonds of different prices and sizes. As Tuckman emphasizes, understanding the distinction between these measures—and their yield-based assumptions—is critical for proper risk management.

---

## 12.1 The Yield-Based Bond Price Function

### 12.1.1 Setting Up the Framework

To understand duration, we must view the bond's price not just as a number but as a mathematical function of its yield-to-maturity $y$. For a bond with annual coupon rate $c$ (per 100 face) and maturity $T$ years, priced using the standard semiannual compounding convention, the price $P(y)$ is:

$$\boxed{P(y) = \sum_{t=1}^{2T} \frac{c/2}{(1+y/2)^t} + \frac{100}{(1+y/2)^{2T}}}$$

This yield-based framework assumes that risk to the bond comes from changes in this single parameter $y$. Tuckman calls this a "single-factor" risk measure. We are asking: *if the yield-to-maturity of this specific bond changes, how does the price change?*

### 12.1.2 Why This Matters for Risk

When a trader asks "What is the duration of this bond?", they are asking about the slope of this $P(y)$ function. Risk measures are essentially derivatives—they quantify how much the price changes for an infinitesimal change in the underlying variable.

Tuckman distinguishes two categories that are important to keep separate:

**Yield-Based Duration** (Macaulay/Modified): Assumes the cash flows are fixed and the only variable changing is the discount rate $y$. This is what we cover in this chapter.

**Effective Duration**: Measures sensitivity to a parallel shift in the *entire benchmark curve*, potentially accounting for changes in cash flows. This is important for callable bonds and mortgage-backed securities. Tuckman notes: "This general definition is also called effective duration. Many market participants, however, use the term duration to mean Macaulay duration or modified duration... These measures of interest rate sensitivity explicitly assume a change in yield-to-maturity."

> **Desk Reality: Which Duration Does Your System Use?**
>
> Most risk systems report "modified duration" or simply "duration" for plain vanilla bonds. For callable bonds, mortgage-backed securities, and other instruments with embedded optionality, they typically use "effective duration" or "OAS duration" that accounts for how cash flows change with rates.
>
> Before interpreting any duration number, verify: (1) Is this modified or effective? (2) What bump size was used to compute it (typically 1bp or 10bp)? (3) Does it assume the YTM changes or the curve shifts?

---

## 12.2 Macaulay Duration: The Original Concept

### 12.2.1 Duration as Weighted-Average Time

Macaulay duration, named after Frederick Macaulay who invented the measure in 1938, answers the question: **"On average, how long do I have to wait to receive the value of this bond?"**

Hull provides a clear definition: if a bond provides cash flows $c_i$ at times $t_i$ for $i = 1, \ldots, n$, then duration is "a weighted average of the times when payments are made, with the weight applied to time $t_i$ being equal to the proportion of the bond's total present value provided by the cash flow at time $t_i$."

Luenberger reinforces this: "The duration of a fixed-income instrument is a weighted average of the times that payments (cash flows) are made. The weighting coefficients are the present values of the individual cash flows."

For continuous compounding, Hull expresses this mathematically:

$$\boxed{D_{\text{Mac}} = \frac{\sum_{i=1}^{n} t_i \cdot c_i e^{-yt_i}}{B} = \sum_{i=1}^{n} t_i \left[\frac{c_i e^{-yt_i}}{B}\right]}$$

For semiannual compounding (the U.S. Treasury convention), Tuckman gives the explicit formula:

$$\boxed{D_{\text{Mac}} = \frac{1}{P}\left[\sum_{t=1}^{2T} \left(\frac{t}{2}\right) \frac{c/2}{(1+y/2)^t} + T \cdot \frac{100}{(1+y/2)^{2T}}\right]}$$

Each term represents:

$$ \text{Weight} \times \text{Time} = \frac{\text{PV of Cash Flow}}{\text{Total Price}} \times \text{Time of Cash Flow} $$

Luenberger observes that "duration is a time intermediate between the first and last cash flows." For any bond with coupons, $D_{\text{Mac}} < T$.

> **Analogy: The Center of Gravity**
>
> Imagine the bond's cash flows as weights placed on a timeline. Each coupon is a small weight placed at its payment date. The principal is a large weight at maturity.
>
> *Macaulay duration is where you would place the fulcrum to balance this timeline.*
>
> Early coupons pull the center of gravity leftward, making duration shorter than maturity. Only a zero-coupon bond (one large weight at the end) has Duration = Maturity.

### 12.2.2 The Replicating Portfolio Interpretation

Tuckman offers a profound insight into the duration calculation. We can think of a coupon bond as a portfolio of zero-coupon bonds—one for each cash flow. Since the Macaulay duration of a zero-coupon bond is exactly its maturity (see Section 12.3.1), the Macaulay duration of a coupon bond is the **weighted average duration of the replicating portfolio of zeros**, where weights are the relative values of those zeros.

Tuckman explains: "the present value of each cash flow in the calculation of Macaulay duration is weighted by its years to receipt because years to receipt is the duration of the corresponding zero in the replicating portfolio."

This perspective makes duration intuitive: a 10-year bond with a 5% coupon has a duration of about 8 years because, in present value terms, roughly 80% of the bond's value comes from the final principal payment (which has duration 10 years), while the remaining 20% comes from earlier coupon payments (which have shorter durations).

### 12.2.3 Worked Example A: Computing Macaulay Duration from Cash Flows

Consider Hull's canonical example: a 3-year bond with $100 face, 10% coupon paid semiannually, and yield of 12% with continuous compounding.

**Step 1: Lay out the cash flows**

The bond pays $5 every 6 months plus $105 at maturity. We discount each using $e^{-0.12 \times t}$.

**Step 2: Calculate present values and weights**

| Time (years) | Cash Flow ($) | Present Value | Weight (PV/B) | Time × Weight |
|:-------------|:--------------|:--------------|:--------------|:--------------|
| 0.5 | 5 | 4.709 | 0.050 | 0.025 |
| 1.0 | 5 | 4.435 | 0.047 | 0.047 |
| 1.5 | 5 | 4.176 | 0.044 | 0.066 |
| 2.0 | 5 | 3.933 | 0.042 | 0.083 |
| 2.5 | 5 | 3.704 | 0.039 | 0.098 |
| 3.0 | 105 | 73.256 | 0.778 | 2.333 |
| **Total** | | **94.213** | **1.000** | **2.653** |

**Result:** The Macaulay Duration is **2.653 years**.

**Sanity Check:** The duration is less than the 3-year maturity (as expected for a coupon bond). The 78% weight of the final principal repayment dominates, but early coupons pull the average time down.

---

## 12.3 Special Cases: Zeros, Par Bonds, and Perpetuities

### 12.3.1 Zero-Coupon Bonds: Duration Equals Maturity

For a zero-coupon bond, there is only one cash flow: the principal at time $T$. Since this single payment represents 100% of the bond's present value, its weight is 1.00. Tuckman states this formally:

$$\boxed{D_{\text{Mac}}\big|_{c=0} = T}$$

"Hence the Macaulay duration of a six-month zero is simply 0.5 while that of a 10-year zero is simply 10."

This property explains *why* the industry quotes duration in "years." As Tuckman emphasizes, "the price sensitivity of zeros can be taken as a benchmark against which to judge the sensitivity of other bonds." When a trader says a bond has duration of 4.4 years, they mean: **"This bond has the same percentage price sensitivity as a 4.4-year zero-coupon bond."**

The zero-coupon case also yields modified duration and DV01 for zeros:

$$\boxed{D_{\text{Mod}}\big|_{c=0} = \frac{T}{1+y/2}}$$

$$\text{DV01}\big|_{c=0} = \frac{T}{100(1+y/2)^{2T+1}}$$

**Example:** A 10-year zero at 5% yield has modified duration of $10/(1.025) = 9.76$ years.

### 12.3.2 Par Bonds: The Closed-Form Formula

For a bond selling at par ($P = 100$ and coupon rate $c = y$), Tuckman derives simplified formulas:

$$\boxed{D_{\text{Mod}}\big|_{P=100} = \frac{1}{y}\left(1 - \frac{1}{(1+y/2)^{2T}}\right)}$$

$$\boxed{D_{\text{Mac}}\big|_{P=100} = \frac{1+y/2}{y}\left(1 - \frac{1}{(1+y/2)^{2T}}\right)}$$

These formulas are useful because par bonds have a fixed price of 100, allowing us to isolate how duration varies with maturity and yield.

**Example:** A 10-year par bond at 5% yield:

$$D_{\text{Mod}} = \frac{1}{0.05}\left(1 - \frac{1}{(1.025)^{20}}\right) = 20 \times (1 - 0.6103) = 7.79 \text{ years}$$

### 12.3.3 Perpetuities: The Upper Bound

A **perpetuity** pays coupons forever ($T = \infty$). Tuckman shows:

$$\boxed{D_{\text{Mod}}\big|_{T=\infty} = \frac{1}{y}}$$

$$\boxed{D_{\text{Mac}}\big|_{T=\infty} = \frac{1+y/2}{y}}$$

At a yield of 5%, the Macaulay duration of a perpetuity is $(1.025)/0.05 = 20.5$ years.

This serves as an upper bound for coupon bonds. As Tuckman notes: "the duration of par bonds rises from zero at a maturity of zero and steadily approaches the duration of a perpetuity."

> **Desk Reality: The 20-Year Rule of Thumb**
>
> At 5% rates, a perpetuity has duration around 20 years. This means no ordinary coupon bond trading near par can have duration much above 20 years. When someone quotes a "25-year duration" bond, it's either (a) a very long-dated zero, (b) a deep discount bond (see Section 12.4.3), or (c) an error.

---

## 12.4 Modified Duration: The Risk Measure

### 12.4.1 From Time to Sensitivity

While "weighted average time" is an interesting statistic, traders are paid to manage price risk. The reason duration is ubiquitous is that this time measure is directly linked to the bond's price derivative.

By differentiating the bond price equation with respect to yield, we discover that the percentage price sensitivity is simply Macaulay duration scaled by the compounding frequency. We call this **Modified Duration**.

Hull explains the relationship. For continuous compounding, duration directly equals the (negative) semi-elasticity:

$$\frac{\Delta B}{B} \approx -D \Delta y$$

But if yields are expressed with compounding frequency $m$ times per year, the relationship becomes:

$$\Delta B \approx -\frac{BD\Delta y}{1+y/m}$$

This leads to defining **modified duration** as:

$$\boxed{D_{\text{Mod}} = \frac{D_{\text{Mac}}}{1+y/m}}$$

For U.S. Treasury bonds with semiannual compounding ($m=2$):

$$\boxed{D_{\text{Mod}} = \frac{D_{\text{Mac}}}{1+y/2}}$$

Hull notes: "This allows the duration relationship to be simplified to $\Delta B = -B D^* \Delta y$ when $y$ is expressed with a compounding frequency of $m$ times per year."

Using modified duration:

$$\boxed{\frac{1}{P}\frac{dP}{dy} = -D_{\text{Mod}}}$$

Or in the approximation form used on trading desks:

$$\frac{\Delta P}{P} \approx -D_{\text{Mod}} \times \Delta y$$

This equation states that **Modified Duration is the approximate percentage change in price for a 100 basis point (1.00%) change in yield.**

### 12.4.2 The Derivation

The factor $(1+y/m)$ comes from the chain rule when differentiating the discount factor. The full derivation (following Tuckman and Luenberger) is instructive.

Starting from the price function:

$$P(y) = \sum_{t=1}^{2T} \frac{c/2}{(1+y/2)^t} + \frac{100}{(1+y/2)^{2T}}$$

Differentiating with respect to $y$:

$$\frac{dP}{dy} = -\frac{1}{1+y/2}\left[\sum_{t=1}^{2T} \frac{t}{2} \frac{c/2}{(1+y/2)^t} + T \frac{100}{(1+y/2)^{2T}}\right]$$

The term inside the brackets is exactly $P \times D_{\text{Mac}}$. Therefore:

$$\frac{dP}{dy} = -\frac{P \cdot D_{\text{Mac}}}{1+y/2}$$

Dividing by $P$:

$$\frac{1}{P}\frac{dP}{dy} = -\frac{D_{\text{Mac}}}{1+y/2} = -D_{\text{Mod}}$$

**This proves the fundamental link: Risk (Modified Duration) is Time (Macaulay Duration) discounted by one period.**

Luenberger states this as the "price sensitivity formula":

$$\frac{dP}{d\lambda} = -D_M P$$

where $D_M$ is the modified duration.

### 12.4.3 Worked Example B: Using Modified Duration for Estimation

Using the bond from Example A (price 94.213, Macaulay duration 2.653 years, yield 12% continuous):

For semiannual compounding, the equivalent yield is 12.3673%. The modified duration is:

$$D_{\text{Mod}} = \frac{2.653}{1 + 0.123673/2} = 2.499$$

**Scenario**: Yields rise by 10 basis points (+0.10%). What is the estimated price change?

Using the duration relationship:

$$\Delta B \approx -94.213 \times 2.499 \times 0.001 = -0.236$$

The predicted new price is $94.213 - 0.236 = 93.977$.

Hull verifies: "When the yield on the bond increases by 10 basis points to 12.1%, the bond price is... 93.963, which is (to three decimal places) the same as that predicted by the duration relationship."

The small error (0.014 cents) is due to **convexity** (the curvature of the price-yield relationship), which we cover in Chapter 13.

> **Rule of Thumb: Duration ≈ Leverage**
>
> - **Duration 10**: If rates move 1%, your price moves 10%. This is like 10× leverage on rates.
> - **Duration 2**: If rates move 1%, your price moves 2%. This is like 2× leverage.
>
> Risk managers use this to normalize positions. A trader who buys $10mm of 2-year notes (Duration ≈ 2) takes the same rate risk as buying $2mm of 10-year bonds (Duration ≈ 8).

---

## 12.5 How Duration Varies: Yield, Coupon, and Maturity

### 12.5.1 Duration and Yield

Tuckman explains that increasing yield lowers duration: "The intuition behind this fact is that increasing yield lowers the present value of all payments but lowers the present value of the longer payments most. This implies that the value of the longer payments falls relative to the value of the whole bond. But since the duration of these longer payments is greatest, lowering their corresponding weights in the duration equation must lower the duration of the whole bond."

He illustrates with a 5-year 5.625% bond:

| Yield | Duration | Longest CF Weight |
|:------|:---------|:------------------|
| 7% | 4.26 | 77.3% |
| 3% | 4.40 | 79.0% |

At lower yields, longer-dated payments become relatively more valuable, pulling duration up.

### 12.5.2 Duration and Coupon

Tuckman observes: "for any given maturity, duration falls as coupon increases: Zeros have the highest duration and premium bonds the lowest."

The intuition: higher-coupon bonds have a greater fraction of their value paid earlier. "The portfolio of zeros replicating relatively high coupon bonds contains a relatively large fraction of its value in shorter-term zeros. From either of these perspectives, higher-coupon bonds are effectively shorter-term bonds and therefore have lower duration."

Luenberger's table shows this pattern at 5% yield:

| Maturity | 1% Coupon Duration | 10% Coupon Duration |
|:---------|:-------------------|:--------------------|
| 10 years | 9.42 | 7.11 |
| 25 years | 20.16 | 12.75 |
| 50 years | 26.67 | 17.38 |

The 1% coupon bonds have substantially higher durations at every maturity.

### 12.5.3 Duration and Maturity: The Deep Discount Paradox

For par and premium bonds, duration increases monotonically with maturity, approaching the perpetuity duration as an upper limit.

But Tuckman reveals a surprising exception: "If the discount is deep enough—that is, if the coupon is low enough relative to yield—the duration of a discount bond rises above the duration of a perpetuity. But since at some large maturity the duration of a discount bond must approach the duration of a perpetuity, the duration of the discount bond must eventually fall as maturity increases."

This is the **deep discount paradox**: for deeply discounted bonds, duration can *exceed* the perpetuity duration before eventually falling back.

Tuckman notes this matters for ultra-long bonds: "relatively recently bonds have been issued with 50 and 100 years to maturity. Should these sell at a substantial discount at some time in their lives, portfolio managers may find themselves holding bonds that become more sensitive to rates as they mature."

### 12.5.4 Worked Example C: The Deep Discount Paradox

Consider a 1% coupon bond at a 5% yield. At what maturities does duration exceed the perpetuity's 20.5 years?

Using numerical calculation:

| Maturity | Price | Macaulay Duration |
|:---------|:------|:------------------|
| 30 years | 38.02 | 20.3 |
| 50 years | 21.95 | 23.8 |
| 70 years | 13.60 | 24.9 |
| 100 years | 7.60 | 24.7 |
| ∞ (perpetuity) | 20.00 | 20.5 |

The 1% coupon bond has duration exceeding perpetuity duration between roughly 35 and 150 years to maturity!

> **Desk Reality: Why Deep Discounts Are "Dangerous"**
>
> The intuition: a deep discount bond's value is dominated by the distant principal payment. At 50-year maturity with a 1% coupon, the PV of the principal is still significant, but extending to 70 years shifts even more weight to the distant principal. Eventually, the principal's PV becomes negligible (approaching zero), and duration falls back.
>
> Portfolio managers holding century bonds that fall to deep discount face increasing duration exposure as the bonds age—contrary to the usual intuition that "bonds get safer as they approach maturity."

---

## 12.6 Dollar Duration: From Percentages to Dollars

### 12.6.1 Definition and Purpose

Modified duration measures *percentage* price sensitivity. But risk systems track *dollar* exposure. Hull defines **dollar duration**:

$$\boxed{D_{\$} = D_{\text{Mod}} \times P}$$

This gives the dollar change in price for a 1% (100bp) yield change:

$$\Delta P = -D_{\$} \times \Delta y$$

Hull states: "Another term that is sometimes used is dollar duration. This is the product of modified duration and bond price, so that $\Delta B = -D_{\$} \Delta y$."

### 12.6.2 Connection to DV01

DV01 is the dollar change for a **1 basis point** move. Since 1bp = 0.0001 = 0.01%:

$$\boxed{\text{DV01} = D_{\$} \times 0.0001 = \frac{D_{\text{Mod}} \times P}{10{,}000}}$$

Or equivalently:

$$D_{\$} = \text{DV01} \times 10{,}000$$

This is the critical link: **Dollar duration is DV01 scaled up by 10,000.**

### 12.6.3 Why Dollar Duration Matters

Hull RM explains the aggregation property: "The dollar duration $D_{\$}$ of a portfolio can be defined as duration of the portfolio times the value of the portfolio... The dollar duration of a portfolio consisting of a number of interest-rate-dependent assets is the sum of the dollar durations of the individual assets."

This is why risk systems use dollar duration (or DV01): **dollar durations add directly**.

If you have two positions:
- Bond A: $D_{\$}^A = 800{,}000$
- Bond B: $D_{\$}^B = 1{,}200{,}000$

Portfolio dollar duration = $800{,}000 + 1{,}200{,}000 = 2{,}000{,}000$

You cannot simply add percentage durations. A portfolio with 50% in a 5-year duration bond and 50% in a 10-year duration bond does *not* have 15-year duration—it has weighted average duration of 7.5 years.

### 12.6.4 Worked Example D: Dollar Duration Calculation

**Bond A:** $50mm face, Price = 105, Modified Duration = 6.5

$$D_{\$}^A = 50{,}000{,}000 \times \frac{105}{100} \times 6.5 = 341{,}250{,}000$$

**DV01:**

$$\text{DV01}^A = 341{,}250{,}000 \times 0.0001 = \$34{,}125 \text{ per basis point}$$

**Verification:** A 1bp rate increase costs: $-D_{\text{Mod}} \times P \times \Delta y = -6.5 \times 52{,}500{,}000 \times 0.0001 = -\$34{,}125$. ✓

> **Desk Reality: Risk Reports Use DV01, Not Percentage Duration**
>
> When your risk report shows "Interest Rate DV01: -$1.2mm," it means a 1bp parallel rise in rates loses $1.2 million. This is the *dollar duration divided by 10,000*.
>
> To interpret: multiply by the rate move in basis points. A 25bp move causes: $25 \times (-1.2\text{mm}) = -\$30\text{mm}$ P&L.

---

## 12.7 The DV01-Duration Mapping

### 12.7.1 The Fundamental Relationship

Tuckman expresses the precise relationship:

$$\boxed{\text{DV01} = \frac{P \times D_{\text{Mod}}}{10{,}000}}$$

Or using Macaulay duration:

$$\text{DV01} = \frac{P \times D_{\text{Mac}}}{10{,}000(1+y/2)}$$

The division by 10,000 appears because DV01 is for a **1 basis point** move (0.0001), while Modified Duration represents sensitivity to a **100% (unit)** move in yield.

### 12.7.2 The "Price Effect" vs "Duration Effect"

Tuckman provides crucial insight into how DV01 behaves differently from duration:

"The major difference between DV01 and duration is that DV01 measures an absolute change in price while duration measures a percentage change."

DV01's behavior with maturity depends on two competing forces:

1. **The Duration Effect**: Longer maturity → higher duration → higher DV01
2. **The Price Effect**: Price changes with maturity can amplify or offset the duration effect

For **par bonds** (price always 100), only the duration effect operates. DV01 increases steadily with maturity, approaching 0.20 for a 5% perpetuity.

For **premium bonds** (price > 100), both effects work together. As Tuckman notes: "the price and duration effects combine so that the DV01 of a premium bond increases with maturity faster than the DV01 of a par bond."

For **discount bonds** (price < 100), the effects oppose. "For a relatively short-maturity discount bond, the duration effect dominates and the DV01 of the discount bond increases with maturity. Then the price effect catches up and the DV01 of the discount bond declines with maturity."

For **zero-coupon bonds**, the effect is most dramatic: "The DV01 of a zero behaves like that of a discount bond except that it eventually falls to zero. With no coupon payments, the present value of a zero with a longer and longer maturity approaches zero, and so does its DV01."

### 12.7.3 Worked Example E: Premium vs Discount Bonds

Consider two 10-year bonds both yielding 5%:

**Premium Bond** (8% coupon):
- Price ≈ 123.16
- Macaulay Duration ≈ 7.54 years
- Modified Duration = 7.54/1.025 = 7.36
- DV01 = (123.16 × 7.36)/10,000 = **0.0906** per 100 face

**Discount Bond** (2% coupon):
- Price ≈ 76.83
- Macaulay Duration ≈ 8.94 years
- Modified Duration = 8.94/1.025 = 8.72
- DV01 = (76.83 × 8.72)/10,000 = **0.0670** per 100 face

The discount bond has *higher duration* (more "leveraged" to rates), but the premium bond has *higher DV01* (more dollars at risk because price is higher).

**Hedging implication:** To hedge $1mm face of the premium bond, you need $1mm × (0.0906/0.0670) = $1.35mm face of the discount bond.

---

## 12.8 Portfolio Duration and the Aggregation Trap

### 12.8.1 The Portfolio Duration Formula

Hull RM shows that portfolio duration is a value-weighted average:

$$\boxed{D_{\text{portfolio}} = \sum_{i} \frac{X_i}{P} D_i = \sum_{i} w_i D_i}$$

where $X_i$ is the market value of asset $i$, $P$ is total portfolio value, and $w_i = X_i/P$ are the value weights.

Hull states: "This shows that the duration $D$ of a portfolio is the weighted average of the durations of the individual assets comprising the portfolio with the weight assigned to an asset being proportional to the value of the asset."

Luenberger confirms: "The duration of a portfolio measures the interest rate sensitivity of that portfolio, just as normal duration measures it for a single bond."

### 12.8.2 The Aggregation Trap

A frequent source of confusion is computing "Portfolio Duration" for hedged portfolios.

Consider a trade: Long $100mm of a 10-year Treasury, Short $100mm of a 10-year futures contract.

- Assets ≈ Liabilities
- **Net Market Value** of the portfolio is near zero
- However, the **Risk** (DV01) might be non-zero if the hedge isn't perfect

If you try to compute weighted average duration, you divide by Net Market Value near zero, resulting in "portfolio duration" that explodes to infinity or fluctuates wildly.

**Solution:** For hedged portfolios, **aggregate DV01** (or dollar duration), not percentage duration:

$$\text{Portfolio DV01} = \sum_{i} \text{DV01}_i^{\text{Position}}$$

Hull RM emphasizes: "The dollar duration of a portfolio consisting of a number of interest-rate-dependent assets is the sum of the dollar durations of the individual assets."

### 12.8.3 Worked Example F: Portfolio DV01 Calculation

**Bond A (Long):** $5mm face, Price = 102, Duration = 4.5
- Market Value = $5.1mm
- DV01 per $1mm = $4.5 × 0.0001 × 1,000,000 = $450
- Position DV01 = $5.1mm × 450/1,000,000 = **+$2,295**

**Bond B (Short):** $3mm face, Price = 108, Duration = 7.2
- Market Value = $3.24mm
- DV01 per $1mm = $7.2 × 0.0001 × 1,000,000 = $720
- Position DV01 = $-3.24mm × 720/1,000,000 = **−$2,333**

**Portfolio DV01** = $2,295 − $2,333 = **−$38**

The portfolio gains $38 if rates rise 1bp. To neutralize, add $38 of positive DV01.

> **Desk Reality: Why "Portfolio Duration" Breaks Down**
>
> Risk reports showing "Duration: N/A" or "Duration: ERROR" usually indicate a hedged book where net market value is near zero. The system tried to compute percentage duration and got an undefined result.
>
> Solution: Look at DV01 or dollar duration instead. These remain well-defined even when net market value is small.

---

## 12.9 Immunization: Duration in Action

### 12.9.1 The Immunization Principle

Luenberger describes immunization as "one of the most (if not the most) widely used analytical techniques of investment science, shaping portfolios consisting of billions of dollars of fixed-income securities held by pension funds, insurance companies, and other financial institutions."

The principle: **match the duration of your assets to the duration of your liabilities.** If rates change, both sides move together, preserving net worth.

Luenberger explains: "If the duration of the portfolio matches that of the obligation stream, then the cash value of the portfolio and the present value of the obligation stream will respond identically (to first order) to a change in yield. Specifically, if yields increase, the present value of the asset portfolio will decrease, but the present value of the obligation will decrease by approximately the same amount."

### 12.9.2 The Immunization Conditions

To immunize a liability stream, construct a bond portfolio satisfying:

1. **Present Value Matching:** $\text{PV}(\text{Assets}) = \text{PV}(\text{Liabilities})$
2. **Duration Matching:** $D_{\text{Assets}} = D_{\text{Liabilities}}$

For a single liability of $L$ due at time $T$:
- PV of liability = $L/(1+y/2)^{2T}$
- Duration of liability = $T$ (it's equivalent to a zero)

### 12.9.3 Worked Example G: Immunizing a Future Obligation

**Setup:** The X Corporation owes $1 million in 10 years. They want to invest now to meet this obligation using two bonds (from Luenberger's example):

- **Bond 1:** 6% coupon, 30-year maturity, Price = 69.04, Duration = 11.44 years
- **Bond 2:** 11% coupon, 10-year maturity, Price = 113.04, Duration = 6.54 years

Current yield: 9% for all bonds.

**Step 1: Calculate obligation present value**

$$\text{PV} = \frac{1{,}000{,}000}{(1.045)^{20}} = \$414{,}643$$

**Step 2: Set up immunization equations**

Let $V_1$ = investment in Bond 1, $V_2$ = investment in Bond 2.

$$V_1 + V_2 = 414{,}643 \quad \text{(PV matching)}$$

$$\frac{11.44 \cdot V_1 + 6.54 \cdot V_2}{414{,}643} = 10 \quad \text{(Duration matching)}$$

**Step 3: Solve**

From the second equation: $11.44 V_1 + 6.54 V_2 = 4{,}146{,}430$

Substituting $V_2 = 414{,}643 - V_1$:

$11.44 V_1 + 6.54(414{,}643 - V_1) = 4{,}146{,}430$

$4.90 V_1 = 1{,}433{,}765$

$V_1 = \$292{,}606$ in Bond 1

$V_2 = \$122{,}037$ in Bond 2

**Verification:** If yields shift to 8% or 10%, the portfolio value still approximately matches the obligation value. Luenberger shows the "surplus" (portfolio minus obligation) remains near zero at ±1% yield changes.

### 12.9.4 Limitations of Immunization

Immunization has important limitations:

1. **First-order only:** Protects against small parallel shifts; large moves require convexity matching (Chapter 13)
2. **Parallel shift assumption:** Real curves twist and steepen, not just shift
3. **Rebalancing required:** As time passes and yields change, duration drifts and portfolios need reimmunization
4. **Equal yield assumption:** Assumes all bonds have the same yield (unrealistic for different credits)

Hull RM notes: "A portfolio consisting of long and short positions in interest-rate-dependent assets can be protected against relatively small parallel shifts in the yield curve by ensuring that its duration is zero."

> **Desk Reality: Immunization in Practice**
>
> Pension funds and insurance companies use immunization to manage asset-liability mismatches. When you see "ALM" (asset-liability management) reports showing "duration gap = 0.3 years," they're measuring how well immunized the book is.
>
> A nonzero duration gap means exposure to rate moves. A 0.3-year gap on $10bn of liabilities means roughly $300mm DV01 exposure.

---

## 12.10 Duration and Value at Risk

### 12.10.1 The Duration-Based VaR Formula

Hull RM connects duration to Value-at-Risk calculations. For a bond portfolio:

$$\boxed{\text{VaR} = D_{\text{Mod}} \times P \times \sigma_y \times z_\alpha \times \sqrt{T}}$$

where:
- $D_{\text{Mod}} \times P$ = dollar duration
- $\sigma_y$ = standard deviation of yield changes (daily)
- $z_\alpha$ = normal quantile (1.65 for 95%, 2.33 for 99%)
- $T$ = holding period in days

Equivalently, using DV01:

$$\text{VaR} = \text{DV01} \times \sigma_y(\text{in bp}) \times z_\alpha \times \sqrt{T}$$

### 12.10.2 Worked Example H: Computing Duration-Based VaR

Hull RM provides this example: "A company has a position in bonds worth $6 million. The modified duration of the portfolio is 5.2 years. Assume that only parallel shifts in the yield curve can take place and that the standard deviation of the daily yield change (when yield is measured in percent) is 0.09."

**Calculate 20-day 90% VaR:**

$$\text{VaR} = 5.2 \times 6{,}000{,}000 \times 0.0009 \times 1.282 \times \sqrt{20}$$

$$= 31{,}200{,}000 \times 0.0009 \times 1.282 \times 4.472$$

$$= \$161{,}289$$

The 90% VaR over 20 days is approximately **$161,000**.

### 12.10.3 Limitations of Duration-Based VaR

Hull RM warns: "The duration-based method for handling interest rates... significantly understate VaR in [cases where the portfolio is exposed to curve shape changes]."

Limitations include:
1. **Parallel shift assumption:** Only captures level risk, not slope or curvature
2. **Normality assumption:** Yield changes aren't always normally distributed
3. **Static assumption:** Ignores convexity effects for large moves
4. **Single-factor:** Doesn't capture tenor-specific risks

For more accurate VaR, Hull RM recommends principal components analysis (PCA), which models level, slope, and curvature factors separately.

> **Desk Reality: When Duration VaR Fails**
>
> A portfolio long 2-year bonds and short 10-year bonds might show near-zero DV01 (duration-hedged). But duration-based VaR would show minimal risk, when in fact the portfolio has massive exposure to curve steepening.
>
> This is why sophisticated risk systems compute key-rate DV01s (Chapter 14) or PCA-based VaR.

---

## 12.11 Effective Duration vs Modified Duration

### 12.11.1 The Distinction

Tuckman makes an important terminological distinction: "This general definition is also called effective duration. Many market participants, however, use the term duration to mean Macaulay duration or modified duration... These measures of interest rate sensitivity explicitly assume a change in yield-to-maturity."

**Modified Duration:**
- Assumes the bond's own YTM changes
- Cash flows are fixed
- Computed analytically from the price formula

**Effective Duration:**
- Assumes a parallel shift in the entire benchmark curve
- Cash flows may change (for callable bonds, MBS)
- Computed numerically by bumping the curve

### 12.11.2 When They Differ

For option-free bonds with flat curves, effective ≈ modified duration.

They diverge when:
1. **Embedded options:** Callable bonds have effective duration < modified (the call limits upside)
2. **MBS/ABS:** Prepayments change with rates, so cash flows aren't fixed
3. **Steep curves:** Bumping the curve differs from bumping the YTM

### 12.11.3 Calculating Effective Duration

Effective duration is computed numerically:

$$D_{\text{eff}} = \frac{P_{-} - P_{+}}{2 \times P_0 \times \Delta y}$$

where:
- $P_0$ = current price
- $P_{+}$ = price after upward curve shift of $\Delta y$
- $P_{-}$ = price after downward curve shift of $\Delta y$

This "bump and reprice" approach captures how the full price function responds to curve changes.

> **Desk Reality: Which Duration for Which Instrument?**
>
> | Instrument | Use Modified? | Use Effective? |
> |------------|--------------|----------------|
> | Treasury bonds | ✓ | ✓ (equivalent) |
> | Investment-grade corporates | ✓ | ✓ (similar) |
> | Callable bonds | Limited | ✓ |
> | MBS/ABS | No | ✓ (OAS duration) |
> | Floating rate notes | ✓ (low) | ✓ (low) |

---

## 12.12 Summary

### Key Takeaways

**Macaulay Duration** is a time measure (years). It represents the weighted-average "center of gravity" of the bond's cash flows. Hull notes that "duration, first suggested by Frederick Macaulay in 1938, has become such a popular measure" because of its direct link to price sensitivity.

**Modified Duration** is a risk measure (percentage). It tells you the percentage price change for a 100bp yield shift. The adjustment factor $(1+y/m)$ accounts for compounding.

**Dollar Duration** is modified duration times price. It measures the *dollar* P&L for a 100bp move. Risk systems use dollar duration (or DV01 = dollar duration / 10,000) because it aggregates additively.

**The Key Formulas:**

- $D_{\text{Mod}} = D_{\text{Mac}} / (1+y/m)$
- $\text{DV01} = P \times D_{\text{Mod}} / 10{,}000$
- $D_{\$} = D_{\text{Mod}} \times P = \text{DV01} \times 10{,}000$
- $\Delta P \approx -D_{\text{Mod}} \times P \times \Delta y$

**Special Cases:**
- Zero-coupon bond: $D_{\text{Mac}} = T$
- Perpetuity: $D_{\text{Mac}} = (1+y/m)/(y/m)$
- Par bond: closed-form formula using yield and maturity

**Duration Properties:**
- Falls with higher yield (distant payments worth less)
- Falls with higher coupon (more value paid early)
- Rises with maturity (for most bonds), approaching perpetuity limit
- Deep discount bonds can exceed perpetuity duration before falling back

**Portfolio Application:**
- Percentage duration is value-weighted average
- Dollar duration adds directly (preferred for hedged books)
- Immunization matches asset duration to liability duration

**Risk Application:**
- Duration-based VaR = $D \times P \times \sigma_y \times z \times \sqrt{T}$
- Effective duration (curve bump) differs from modified (YTM bump) for optionality

---

## 12.13 Key Concepts Summary

| Concept | Definition | Why It Matters |
|:--------|:-----------|:---------------|
| Macaulay Duration | Weighted-average time to receive cash flows | Original duration measure; equals maturity for zeros |
| Modified Duration | $D_{Mac}/(1+y/m)$; percentage price sensitivity | Primary risk measure quoted on trading desks |
| Dollar Duration | Duration × Price; dollar sensitivity to 100bp | How risk systems aggregate exposure |
| DV01 | Dollar duration / 10,000; dollar sensitivity to 1bp | The standard unit for expressing rate risk |
| Zero Duration | Equals maturity T | Benchmark against which other bonds compared |
| Perpetuity Duration | $(1+y/m)/(y/m)$; upper bound for coupon bonds | Limit approached by long-dated par bonds |
| Deep Discount Paradox | Discount bond duration can exceed perpetuity | Important gotcha for ultra-long dated bonds |
| Immunization | Matching asset duration to liability duration | Protects against parallel rate shifts |
| Effective Duration | Price sensitivity to curve shift (not YTM) | Needed for bonds with embedded options |

---

## 12.14 Notation for This Chapter

| Symbol | Definition |
|:-------|:-----------|
| $P$ | Bond price (clean, per 100 face) |
| $B$ | Bond price (Hull's notation) |
| $D_{\text{Mac}}$ | Macaulay Duration (years) |
| $D_{\text{Mod}}$ | Modified Duration (sensitivity measure) |
| $D_{\$}$ | Dollar Duration (dollar sensitivity to 100bp) |
| $D_{\text{eff}}$ | Effective Duration (curve-bump based) |
| $y$ | Yield to Maturity |
| $c$ | Coupon rate (annual) |
| $m$ | Compounding frequency per year |
| $T$ | Time to maturity (years) |
| $w_i$ | Portfolio weight for asset $i$ |

---

## 12.15 Flashcards

| # | Question | Answer |
|:--|:---------|:-------|
| 1 | What is Macaulay duration? | The weighted-average time to receipt of cash flows, where weights are PV of each cash flow divided by total price. |
| 2 | What is the Macaulay duration of a zero-coupon bond? | Exactly equal to its maturity T. |
| 3 | How do you convert Macaulay to Modified duration? | $D_{Mod} = D_{Mac} / (1 + y/m)$ where m is compounding frequency. |
| 4 | What does modified duration measure? | The approximate percentage price change for a 100bp (1%) change in yield. |
| 5 | What is dollar duration? | Duration × Price. It measures the dollar change for a 100bp yield move. |
| 6 | How is DV01 related to dollar duration? | DV01 = Dollar Duration / 10,000 = (D × P) / 10,000 |
| 7 | What is the Macaulay duration of a perpetuity at 5% yield? | $(1.025)/(0.05) = 20.5$ years. |
| 8 | What is the modified duration of a perpetuity at yield y? | $1/y$. At 5%, that's 20 years. |
| 9 | How does duration change as coupon increases (fixed maturity)? | Duration decreases—higher coupon means more value paid earlier. |
| 10 | Can a bond's duration ever exceed a perpetuity's duration? | Yes—deep discount bonds at very long maturities can exceed perpetuity duration. |
| 11 | Why do risk systems use dollar duration (or DV01) instead of percentage duration? | Dollar durations add directly; percentage durations require value weighting. |
| 12 | What is the "portfolio duration trap"? | Computing weighted-average duration fails when net market value is near zero (hedged books). |
| 13 | What are the two conditions for immunization? | Match present values AND match durations of assets and liabilities. |
| 14 | What is effective duration? | Price sensitivity to a parallel shift in the benchmark curve (vs. YTM shift for modified). |
| 15 | When should you use effective duration instead of modified? | For callable bonds, MBS, or any instrument with embedded options. |
| 16 | What is the VaR formula using duration? | VaR = D × P × σ_y × z_α × √T |
| 17 | If a bond has duration 7 and rates rise 50bp, what's the approximate price change? | About −3.5% (= −7 × 0.50%) |
| 18 | What competing effects determine how DV01 varies with maturity? | The "duration effect" (longer → higher D) and "price effect" (price changes with maturity). |
| 19 | For a $100mm portfolio with 5-year duration, what's the P&L from a 10bp rate rise? | About −$500,000 (= −5 × $100mm × 0.001) |
| 20 | Who invented duration and when? | Frederick Macaulay in 1938. |

---

## 12.16 Mini Problem Set

### Questions

**Q1. [Easy — Calculation]** A 5-year zero-coupon bond has a yield of 4% (semiannual compounding). Calculate its:
(a) Macaulay duration
(b) Modified duration
(c) DV01 per 100 face

---

**Q2. [Easy — Conversion]** A 10-year bond has Macaulay duration of 8.2 years. Yield is 6% (semiannual). What is:
(a) Modified duration
(b) Dollar duration for $1 million face at price 105

---

**Q3. [Medium — Portfolio]** You hold a portfolio of three bonds:

| Bond | Face ($mm) | Price | Duration |
|:-----|:-----------|:------|:---------|
| A | 10 | 102 | 3.5 |
| B | 15 | 98 | 6.2 |
| C | 5 | 110 | 9.1 |

Calculate:
(a) Total market value
(b) Portfolio duration (value-weighted)
(c) Portfolio DV01

---

**Q4. [Medium — Deep Discount]** A 50-year bond has a 1% coupon and yields 6%. Compare its Macaulay duration to a perpetuity at the same yield. Which is larger and why?

---

**Q5. [Medium — Immunization]** You have a $500,000 liability due in 8 years. You can invest in:
- Bond X: Duration 5 years, yield 5%
- Bond Y: Duration 12 years, yield 5%

Design an immunized portfolio. How much in each bond?

---

**Q6. [Medium — VaR]** A $10mm bond portfolio has modified duration 6.5 years. Daily yield standard deviation is 8bp. Calculate the 10-day 99% VaR using the duration model.

---

**Q7. [Medium — Price/Duration Effect]** Two 15-year bonds both yield 5%:
- Bond A: 8% coupon (premium)
- Bond B: 2% coupon (discount)

Which has higher duration? Which has higher DV01? Explain the relationship.

---

**Q8. [Hard — Sensitivity]** A 10-year par bond at 5% yield has duration 7.79 years. If yields fall to 4%:
(a) Estimate the new duration (qualitative direction)
(b) Why does this pose a problem for hedging?

---

**Q9. [Hard — Effective vs Modified]** Explain why a callable bond at a yield just above the call trigger has effective duration much lower than modified duration.

---

**Q10. [Hard — Perpetuity Limit]** Calculate the Macaulay duration of par bonds at 5% yield for maturities 10, 30, 50, and 100 years. Compare to the perpetuity duration of 20.5 years.

---

**Q11. [Hard — Dollar Duration Aggregation]** Prove that the dollar duration of a portfolio equals the sum of the dollar durations of its components. (Hint: use the definition and the fact that portfolio value = sum of component values.)

---

**Q12. [Hard — Integration]** A pension fund has liabilities with PV = $100mm and duration 12 years. Design a portfolio using 5-year zeros (priced at 78.35) and 20-year par bonds (duration 12.5) to immunize the liability. How many of each bond do you need?

---

### Solutions (Brief)

**Q1.**
(a) Zero-coupon: $D_{Mac} = T = 5$ years
(b) $D_{Mod} = 5/(1.02) = 4.90$ years
(c) Price = $100/(1.02)^{10} = 82.03$. DV01 = $(82.03 × 4.90)/10,000 = 0.0402$ per 100 face

**Q2.**
(a) $D_{Mod} = 8.2/(1.03) = 7.96$ years
(b) Market value = $1mm × 1.05 = $1.05mm$. Dollar duration = $7.96 × 1,050,000 = 8,358,000$

**Q3.**
(a) MV = $10mm×1.02 + $15mm×0.98 + $5mm×1.10 = $10.2 + $14.7 + $5.5 = $30.4mm$
(b) $D = (10.2×3.5 + 14.7×6.2 + 5.5×9.1)/30.4 = (35.7 + 91.14 + 50.05)/30.4 = 5.82$ years
(c) DV01 = $(30.4mm × 5.82)/10,000 = $17,693$

**Q4.** Assuming semiannual compounding, the perpetuity Macaulay duration (in years) is:
$$D_{\text{Mac}} = \frac{1+y/2}{y}$$
At $y=6\\%$, $D_{\text{Mac}} = 1.03/0.06 = 17.17$ years. The 50-year deep discount (1% coupon) has duration approximately 21 years—exceeding the perpetuity due to the heavy weighting on the distant principal.

**Q5.** Let $V_X$ in Bond X, $V_Y$ in Bond Y.
- PV matching: $V_X + V_Y = 500,000/(1.025)^{16} = 335,602$
- Duration matching: $5V_X + 12V_Y = 8 × 335,602$

Solving: $V_Y = (8×335,602 - 5×335,602)/(12-5) = 143,829$; $V_X = 191,773$

**Q6.** VaR = $6.5 × 10,000,000 × 0.0008 × 2.326 × √10 = 6.5 × 10mm × 0.0008 × 2.326 × 3.162 = $382,437$

---

Chapter 12 connects to Chapter 11 (DV01/PV01 definitions) and leads to Chapter 13 (Convexity), where we add the second-order correction that matters for large yield moves.

## References

- Tuckman & Serrat, *Fixed Income Securities: Tools for Today's Markets* (Macaulay/modified duration; yield-based DV01; perpetuities; deep discount behavior).
- Hull, *Options, Futures, and Other Derivatives* (duration/convexity definitions and worked examples).
- Hull, *Risk Management and Financial Institutions* (dollar duration, portfolio aggregation, duration-based VaR discussions).
- Luenberger, *Investment Science* (immunization: matching PV and duration; convexity considerations).
