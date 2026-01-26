# Chapter 13: Convexity — Second-Order P&L and When DV01 Breaks

---

## Introduction

If duration is your delta, convexity is your gamma. In the world of derivatives, delta measures immediate directional risk, but gamma reveals how that risk evolves as the market moves. A delta-neutral portfolio might be safe for small fluctuations, but if a trader is "short gamma," a significant move in either direction can lead to catastrophic losses. This same logic governs the fixed income markets.

Practitioners typically hedge interest rate risk with DV01 (duration), creating a linear hedge that works beautifully for standard daily moves of 1 to 5 basis points. However, bonds are **not linear instruments**. The relationship between price and yield is curved. As yields change, duration itself changes. If a risk manager ignores this curvature—**convexity**—they are effectively trading with a blind spot for large market shocks.

For a 100 basis point rate shock, a duration-only model might overestimate losses by several percent of a bond's value. Even more dangerously, for instruments like callable bonds and mortgage-backed securities, the curvature can flip direction entirely. This phenomenon, known as negative convexity, causes a position to lose money faster than duration predicts precisely when protection is needed most.

This chapter extends the risk framework beyond the linear world of DV01. We cover:

1. **The Geometry of Convexity** (Section 13.1): Why "curvature" is the mathematically correct way to visualize second-order risk.
2. **Defining Convexity Precisely** (Section 13.2): The normalized second derivative and its numerical estimation.
3. **Convexity for Standard Bonds** (Section 13.3): Analytical formulas and the critical $T^2$ scaling rule.
4. **When DV01 Breaks** (Section 13.4): Quantifying the error of linear hedging during large volatility events.
5. **DV01 Hedges with Convexity Mismatch** (Section 13.5): The "self-funding" trade and convexity arbitrage.
6. **Positive vs Negative Convexity** (Section 13.6): Understanding the "short gamma" trap of callable bonds and mortgages.
7. **The Barbell vs Bullet Trade** (Section 13.7): Structuring portfolios to monetize convexity differences.
8. **Convexity as Volatility Exposure** (Section 13.8): Jensen's Inequality and why convexity is priced.

---

## 13.1 The Price-Yield Curve Is Not a Straight Line

### 13.1.1 Why Curvature Matters

Consider a simple zero-coupon bond maturing in 10 years. At a yield of 5%, its price is approximately $61.03. If we plot price against yield, the resulting curve slopes downward—higher yields mean lower prices—but crucially, it bends *upward*, like a smile. This geometric property is what we call **convexity**.

This curvature has immediate financial consequences. As yields **rise** and prices fall, the slope of the curve flattens. This means the bondholder loses money, but their sensitivity to further rate increases (DV01) decreases, essentially "cushioning" the loss. Conversely, as yields **fall** and prices rise, the slope steepens. The bondholder makes money, and their sensitivity increases, accelerating the gains.

Tuckman emphasizes this point directly: "the property of positive convexity may also be thought of as the property that DV01 falls as rates increase." In simple terms: positive convexity implies that the bond's duration works in your favor in both market directions.

> **Analogy: The Shock Absorber**
>
> Convexity is the shock absorber on a car.
> *   **Rates Rise (Bad News)**: Price falls, but convexity *slows* the fall (Duration decreases). The shock is dampened.
> *   **Rates Fall (Good News)**: Price rises, and convexity *accelerates* the rise (Duration increases). The gain is amplified.
> *   **Result**: Positive convexity helps you win more when you're right, and lose less when you're wrong.

### 13.1.2 Taylor Expansion: The Mathematical Foundation

To quantify this, let $P(y)$ be the price given yield $y$. For a sudden shock $\Delta y$, we can approximate the price change using a second-order Taylor series expansion around the current yield. While duration captures the first-order effect, convexity captures the second-order term. As Tuckman develops in Chapter 5, the Taylor approximation is:

$$P(y + \Delta y) \approx P(y) + \frac{dP}{dy}\Delta y + \frac{1}{2}\frac{d^2P}{dy^2}(\Delta y)^2$$

Dividing by price $P$ gives us the standard percentage change decomposition used by practitioners:

$$\boxed{\frac{\Delta P}{P} \approx -D\,\Delta y + \frac{1}{2}C\,(\Delta y)^2}$$

Here, $D = -\frac{1}{P}\frac{dP}{dy}$ represents Modified Duration (linear risk), and $C = \frac{1}{P}\frac{d^2P}{dy^2}$ represents **Convexity** (curvature risk). This is Tuckman's equation (5.20), which Hull presents in identical form in Chapter 4.

Since the term $(\Delta y)^2$ is always positive regardless of the direction of the market move, the convexity term $\frac{1}{2}C(\Delta y)^2$ acts as a positive adjustment to returns for positive convexity instruments. It boosts gains in rallies and reduces losses in sell-offs, quantifying the "cushion" provided by the bond's curvature.

---

## 13.2 Defining Convexity Precisely

### 13.2.1 The Normalized Second Derivative

We define convexity formally as the second derivative of price with respect to yield, normalized by the price itself. Tuckman states this as equation (5.14):

$$\boxed{C \equiv \frac{1}{P}\frac{d^2P}{dy^2}}$$

Hull provides an equivalent definition: "A measure of convexity is $C = \frac{1}{B}\frac{d^2B}{dy^2}$."

We divide by price $P$ for the same reason we normalize duration: to interpret the metric as a relative or percentage sensitivity. While **duration** provides a standardized measure of percentage price change, **convexity** provides a standardized measure of how much that duration itself changes.

> **Note on Units:** Convexity has units of time squared (years²). Some trading systems report "Dollar Convexity" ($P \times C = d^2P/dy^2$) instead. It is critical to always check units before using these numbers in risk reports.

### 13.2.2 Computing Convexity Numerically

In practice, analytic formulas are not always available for complex instruments. In such cases, convexity is estimated using the same "bump and reprice" logic used for DV01. Instead of measuring a single slope, we estimate the *change* in slope using a **central difference method**:

$$\boxed{C \approx \frac{1}{P_0} \left[ \frac{P_+ - 2P_0 + P_-}{(\Delta y)^2} \right]}$$

where:
- $P_-$ is the price at $y - \Delta y$
- $P_0$ is the current price
- $P_+$ is the price at $y + \Delta y$

Tuckman demonstrates this method in detail in Table 5.3, noting that "extra precision is often necessary when calculating second derivatives" because the calculation involves the difference of differences, which can amplify numerical noise. A bump size of 1 to 5 basis points is typically optimal; bump sizes smaller than 0.1 bps risk numerical instability, while deviations larger than 20 bps may inadvertently capture higher-order effects beyond convexity.

### 13.2.3 Worked Example: Tuckman's Table 5.3

Tuckman provides a detailed numerical example for estimating the convexity of a 5% coupon bond at a 5% yield (par pricing). The methodology is:

**Step 1: Estimate the first derivative at 4.995%** (between 4.99% and 5.00%):
$$\frac{dP}{dy}\bigg|_{4.995\%} = \frac{100.0000 - 100.0780}{0.05 - 0.0499} = -779.83$$

**Step 2: Estimate the first derivative at 5.005%** (between 5.00% and 5.01%):
$$\frac{dP}{dy}\bigg|_{5.005\%} = \frac{99.9221 - 100.0000}{0.0501 - 0.05} = -779.09$$

**Step 3: Estimate the second derivative at 5.00%**:
$$\frac{d^2P}{dy^2} = \frac{-779.09 - (-779.83)}{0.05005 - 0.04995} = \frac{0.7363}{0.0001} = 7{,}363$$

**Step 4: Compute convexity**:
$$C = \frac{7{,}363}{100} = \mathbf{73.63}$$

This matches Tuckman's Table 5.3 value of 73.6287 for the bond at 5%.

---

## 13.3 Convexity for Standard Bonds

### 13.3.1 Zero-Coupon Bond Convexity

For a zero-coupon bond, convexity can be derived analytically, offering clear intuition for how it behaves across maturities. Given the price of a zero-coupon bond $P = 100 / (1 + y/2)^{2T}$ under semiannual compounding, differentiating twice and dividing by $P$ yields the closed-form solution. Tuckman derives this as equation (6.36):

$$\boxed{C_{\text{zero}} = \frac{T(T + 0.5)}{(1 + y/2)^2}}$$

This formula reveals a fundamental rule of thumb for fixed income traders: **Convexity scales with the square of maturity ($T^2$).**

Tuckman explains the intuition: "From (6.36) it is clear that longer-maturity zeros have greater convexity. In fact, convexity increases with the square of maturity."

### 13.3.2 Worked Example: The Power of $T^2$

To visualize this scaling, let us compare the convexity of zero-coupon bonds at a 5% yield across different maturities:

| Maturity ($T$) | Approx $T^2$ | Actual Convexity | Interpretation |
| :--- | :--- | :--- | :--- |
| **2 Years** | 4 | **4.76** | Negligible curvature |
| **10 Years** | 100 | **99.94** | Significant curvature |
| **30 Years** | 900 | **870.91** | Massive curvature |

A 30-year bond has roughly $9\times$ the convexity of a 10-year bond, mirroring the relationship $30^2 = 9 \times 10^2$. This geometric explosion is why "convexity trades" almost invariably focus on the long end of the yield curve, where the "gamma" is most potent.

### 13.3.3 Coupon Bond Convexity

For coupon bonds, Tuckman derives the yield-based convexity formula (equation 6.35):

$$C = \frac{1}{P(1+y/2)^2}\left[\sum_{t=1}^{2T} \frac{t}{2} \cdot \frac{t+1}{2} \cdot \frac{c/2}{(1+y/2)^{t}} + T(T+0.5) \cdot \frac{100}{(1+y/2)^{2T}}\right]$$

In words, as Tuckman explains: "the present values of the cash flows are weighted by time multiplied by time plus one half, summed up, and divided by price multiplied by one plus half the yield squared."

Since a coupon bond is a portfolio of zeros (one for each cash flow), its convexity equals the value-weighted average of the individual zero convexities. Longer-maturity coupon bonds generally have greater convexity than shorter-maturity coupon bonds.

### 13.3.4 Analytical vs. Numerical Check

We can verify the numerical approximation method against the analytic formula. Consider a 5-year zero-coupon bond at a 5% yield.

**Analytic Result**: Using the formula:
$$C = \frac{5 \times 5.5}{(1.025)^2} = \frac{27.5}{1.050625} \approx \mathbf{26.17}$$

**Finite Difference**: Bumping yields by 1 bp ($P_-$ at 4.99%, $P_+$ at 5.01%):
$$\frac{78.0818 - 2(78.1198) + 78.1579}{78.1198 \times (0.0001)^2} \approx \mathbf{26.17}$$

The results match perfectly, confirming that for standard instruments, the "bump and reprice" methodology is robust.

---

## 13.4 When DV01 Breaks: The Case for Convexity

When is duration "good enough," and when is it dangerous? The answer lies in the magnitude of the market move. The error of a duration-only model does not scale linearly; it scales with the *square* of the interest rate shock:

$$\text{Error} \approx \frac{1}{2} C (\Delta y)^2$$

Tuckman illustrates this in Figure 5.6, showing how the first-order (duration-only) approximation deviates from actual prices for large rate moves, while the second-order approximation (including convexity) provides a much tighter fit.

### 13.4.1 Worked Example: A 100 Basis Point Shock

Consider a **3-year, 6% coupon bond** priced near par with a 5% yield. The relevant metrics are:
- **Price**: 102.754
- **Modified Duration ($D$)**: 2.725
- **Convexity ($C$)**: 9.08

If yields rise by **+100 basis points** (to 6%), we can compare the linear prediction against reality:

| Method | Prediction Formula | Predicted Price Change | Error vs Actual |
| :--- | :--- | :--- | :--- |
| **Duration Only** | $-102.754 \times 2.725 \times 0.01$ | **-2.800** | **-0.046** (Overstates loss) |
| **Duration + Convexity** | $-2.800 + \frac{1}{2}(102.754)(9.08)(0.01)^2$ | **-2.753** | **+0.001** (Nearly perfect) |
| **Actual Repricing** | $P(6\%) - P(5\%)$ | **-2.754** | — |

For a 100 bp move, duration was off by roughly 4.6 cents. While this sounds small, it represents a 1.7% relative error in the P&L estimate. Adding the convexity adjustment reduces this error to a negligible 0.1 cents.

### 13.4.2 Scaling of Errors

The danger arises because doubling the shock size quadruples the error:

| Shock Size | Duration Error | Scaling Factor |
| :--- | :--- | :--- |
| **25 bp** | -0.003 | 1× |
| **50 bp** | -0.012 | 4× |
| **100 bp** | -0.046 | 16× |

This exponential growth is why linear models fail spectacularly during crashes. Tuckman notes: "While convexity is usually a larger number than duration, for relatively small changes in rate the change in rate is so much larger than the change in rate squared that the duration effect dominates."

For standard daily moves of 5–10 bps, duration is adequate. For "Value at Risk" (VaR) scenarios or stress tests involving 50–100 bp moves, ignoring convexity is professional malpractice.

---

## 13.5 DV01 Hedges with Convexity Mismatch

Traders often construct "DV01 Neutral" portfolios and believe they are fully hedged. However, if the long and short legs of the trade have significantly different convexities, the portfolio is only hedged for *small* moves. This is known as a **convexity mismatch**.

Tuckman provides a detailed example in his "Hedging Example, Part II" where he shows that a market maker hedging options with bonds is "short convexity" and loses money whether rates rise or fall.

### 13.5.1 Worked Example: The "Self-Funding" Trade

Imagine a portfolio that is DV01 neutral but "Long Convexity":
- **Long**: 1 unit of a 10-year par bond ($C=73.6$, DV01=0.0779)
- **Short**: 4.14 units of a 2-year par bond ($C=4.5$, DV01=0.0188)

The position is delta-neutral:
$$\text{Net DV01} = 0.0779 - (4.144 \times 0.0188) \approx 0$$

However, the position is heavily gamma-positive:
$$\text{Net Convexity} = 73.6 - (4.144 \times 4.5) = +54.9$$

Because of this positive net convexity, the portfolio acts like a long straddle option. If rates move **100 bps** in *either* direction, the trader profits:

1. **Rates RALLY (-100 bps)**: The long 10y position gains +$8.18, while the short 2y position loses -$7.89. **Net P&L: +$0.29**.
2. **Rates SELL OFF (+100 bps)**: The long 10y position loses -$7.44, while the short 2y position gains +$7.70. **Net P&L: +$0.26**.

In both scenarios, the portfolio makes money. This profit comes from the second-order term $\frac{1}{2} C (\Delta y)^2$, which is always positive for a net positive $C$.

As Tuckman explains: "Since the P&L of the long option position is always above that of the long bond position, the market maker loses this $80,000 whether rates rise or fall... the hedged position loses whether rates rise or fall because the option is more convex than the bond. In market jargon, the hedged position is short convexity."

Market-neutral hedge funds actively manage this mismatch, strictly deciding whether to be long or short convexity based on their volatility expectations.

> **Trader Talk: Gamma Trading**
>
> Traders call convexity "Gamma" (borrowed from options everywhere).
> *   **Long Gamma**: You own convexity (like a straddle). You want the market to go crazy (High Volatility). You make money on the wiggles.
> *   **Short Gamma**: You sold convexity. You want the market to sleep (Low Volatility). You bleed money if the market moves.

---

## 13.6 Positive vs Negative Convexity

So far, we have assumed convexity is positive (the "smile"). However, for instruments with embedded options—specifically **Callable Bonds** and **Mortgage-Backed Securities (MBS)**—convexity can turn **negative** (a "frown").

Tuckman states explicitly: "Fixed income securities need not be positively convex at all rate levels. Some important examples of negative convexity are callable bonds... and mortgage-backed securities."

> **Visualization: The Smile and The Frown**
>
> *   **Positive Convexity**: A "Smiley Face" curve. Price rises faster than expected.
> *   **Negative Convexity**: A "Frowning Face" curve. Price rises *slower* than expected (or even falls).
>
> **The Mortgage Trap (Negative Convexity)**
> *   **Scenario**: You own an MBS. Rates drop to 3%.
> *   **Theory**: Your bond price should skyrocket!
> *   **Reality**: Homeowners refinance. You get your cash back at par (100) right when the bond was worth 105. Your upside is stolen.
> *   **Result**: The "cushion" turns into a "ceiling."

### 13.6.1 The Mechanics of Negative Convexity

In callable bonds, as yields fall, the issuer is likely to call the bond to refinance at a lower rate. This caps the price appreciation at the call price (e.g., 100). Graphically, the price-yield curve flattens out and then bends downwards, creating a concave shape.

Tuckman explains: "Figure 19.4 also shows that an embedded call option induces negative convexity. For the callable bond price curve to resemble the three-year curve at low rates and the 10-year curve at high rates, the callable bond curve must be negatively convex."

Similarly, in MBS, as rates fall, homeowners refinance their mortgages, causing the bond to return principal precisely when it would otherwise be most valuable.

### 13.6.2 Worked Example: The Callable Trap

Consider a 5% Callable Bond (callable at par in 1 year) compared to a vanilla non-callable bond. As yields drop, the difference becomes stark:

| Yield Level | Vanilla Price | Callable Price | Convexity (Callable) |
| :--- | :--- | :--- | :--- |
| **6% (OTM)** | 92.56 | 91.87 | **-120** (Negative) |
| **5% (ATM)** | 100.00 | 96.95 | **-223** (Deeply Negative) |
| **4% (ITM)** | 108.18 | 100.03 | **-147** (Negative) |

At a 5% yield, the convexity is **-223**. This negative number has profound hedging implications. If rates drop 100 bps to 4%, the vanilla bond gains 8.18, but the callable bond gains only 3.08 because it is capped at par. If you hedged this callable bond with a standard DV01 ratio, you would massively underperform the hedge.

### 13.6.3 Extension Risk: The Destabilizing Dynamic

**Deep Insight**: Negative convexity implies that **DV01 increases as yields rise**.

- **Positive Convexity**: Yields $\uparrow$ $\implies$ Price $\downarrow$ but Duration $\downarrow$ (Self-stabilizing).
- **Negative Convexity**: Yields $\uparrow$ $\implies$ Price $\downarrow$ and Duration $\uparrow$ (Destabilizing).

This phenomenon is known as "extension risk." Tuckman illustrates this in Figure 19.5, showing how "the duration of callable bonds increases as rates rise above the coupon."

It explains why MBS hedging is notoriously difficult: as rates rise and the market sells off, your hedge ratio increases, forcing you to sell *more* into a falling market to remain hedged. This pro-cyclical selling can exacerbate market crashes.

Tuckman warns: "care must be exercised when mixing securities of positive and negative convexity because the resulting hedges or comparative return estimates are inherently unstable."

---

## 13.7 The Barbell vs Bullet Trade

The most classic convexity trade is the **Barbell vs. Bullet**, which Tuckman treats extensively in Chapter 6.

- **Bullet**: A portfolio concentrated at a single maturity (e.g., 9-year zero).
- **Barbell**: A portfolio split between short (e.g., 2-year) and long (e.g., 30-year) maturities.

You can weight them to have the **exact same duration**. So why choose one over the other? **Convexity.**

### 13.7.1 Worked Example: Tuckman's Barbell vs Bullet

Tuckman provides a canonical example. Assume a flat yield curve at 5%:

**Bullet Portfolio**: A 9-year zero-coupon bond.
- Duration: 9 years
- Convexity: $\frac{9 \times 9.5}{(1.025)^2} = 81.38$

**Barbell Portfolio**: 75% in 2-year zeros and 25% in 30-year zeros.
- Duration: $0.75 \times 2 + 0.25 \times 30 = 9$ years (matches bullet)
- Convexity: $0.75 \times \frac{2 \times 2.5}{(1.025)^2} + 0.25 \times \frac{30 \times 30.5}{(1.025)^2} = 221.30$

The barbell has convexity of **221.30** vs the bullet's **81.38**—nearly three times as much!

Tuckman explains the mathematics: "A barbell has greater convexity than a bullet because duration increases linearly with maturity while convexity increases with the square of maturity. If a combination of short and long durations, essentially maturities, equals the duration of the bullet, that same combination of the two convexities, essentially maturities squared, must be greater than the convexity of the bullet."

### 13.7.2 The Cost of Convexity: Negative Carry

If the Barbell has the same duration but higher convexity, isn't it strictly better? As Tuckman notes: "this does not imply that the barbell portfolio is superior to the bullet."

The yield curve is typically concave (humped) or upward sloping. To buy the Barbell, you must sell the intermediate 9-year bond (which usually yields more) and buy 2-year and 30-year bonds (which, on average, yield less in a concave curve environment).

This price you pay—giving up yield to own potential non-linear gains—is the **cost of convexity**:

- **Long Volatility**: The Barbell outperforms if rates move significantly (due to convexity).
- **Short Volatility**: The Bullet outperforms if rates stay stable (due to higher yield/carry).

Tuckman summarizes: "the bullet outperforms if rates move by a relatively small amount, up or down, while the barbell outperforms if rates move by a relatively large amount."

Traders view this as a volatility trade: buying a Barbell is equivalent to buying a straddle on rates, financed by the yield spread.

### 13.7.3 General Principle

Tuckman provides the generalization: "The intuition gained from the barbell-bullet example can be used to understand the convexity properties of other portfolios. In general, spreading out the cash flows of a portfolio (without changing its duration) raises its convexity."

This principle applies to any portfolio construction: spreading cash flows increases convexity; concentrating them reduces it.

---

## 13.8 Convexity as Volatility Exposure

Why does convexity behave like volatility? The answer lies in **Jensen's Inequality**: $E[f(x)] \geq f(E[x])$ for a convex function $f$.

If you own a convex bond, the average price after a random volatility event is *higher* than the price at the average yield.

### 13.8.1 Jensen's Inequality and Bond Pricing

Tuckman develops this rigorously in Chapter 10. He shows that the pricing function of a zero, $1/(1+r)$, is convex, which implies:

$$E\left[\frac{1}{1+r}\right] > \frac{1}{E[1+r]} = \frac{1}{1+E[r]}$$

This means that in a world with interest rate volatility, bond prices are *higher* than they would be if rates were certain to equal their expected value. Equivalently, yields are *lower* than the expected rate—this difference is the "convexity value."

Tuckman illustrates: "Even though the one-year rate is 10% and the expected one-year rate in one year is 10%, the two-year spot rate is 9.982%. The 1.8-basis point difference... is the effect of convexity on that spot rate."

### 13.8.2 Convexity Value Increases with Volatility

Tuckman shows that this convexity effect is proportional to volatility squared. In his example, doubling volatility from 200 bps to 400 bps significantly increases the convexity effect on yields.

The mathematical relationship, derived in Chapter 10, is:

$$\text{Convexity Effect on Yield} \approx -\frac{1}{2} C \sigma^2$$

where $\sigma$ is the yield volatility. This is why:
- **Long Convexity**: You are long gamma. You want realized volatility > implied volatility.
- **Short Convexity**: You are short gamma. You want realized volatility < implied volatility.

Tuckman concludes: "it is in this sense... that a long convexity position is long volatility while a short convexity position is short volatility."

### 13.8.3 Intuitive Summary

- **Scenario A**: Yields stay flat. Price = $P_0$.
- **Scenario B**: Yields go up 50 bps or down 50 bps with 50/50 probability.
  - If rates Up: Loss is dampened by convexity.
  - If rates Down: Gain is amplified by convexity.
  - **Average Price**: $(P_{up} + P_{down})/2 > P_0$.

This "convexity bias" means that in a volatile market, high-convexity bonds tend to outperform low-convexity bonds, all else equal. This is why convexity is priced at a premium.

---

## 13.9 Practical Notes and Sanity Checks

### 13.9.1 Bump Size Selection

When computing convexity numerically:
- **Too small** (< 0.1 bp): Numerical precision errors dominate
- **Too large** (> 20 bp): Captures higher-order effects beyond convexity
- **Optimal**: 1–5 bp provides the best balance

### 13.9.2 Sign Checks

- **Vanilla bonds**: Always positive convexity (smile)
- **Callable bonds**: Negative convexity at low yields (frown)
- **MBS**: Typically negative convexity due to prepayment
- **Long receiver swaptions**: Very high positive convexity

### 13.9.3 Order of Magnitude

For vanilla bonds:
- **2-year zero**: Convexity ~ 5
- **10-year zero**: Convexity ~ 100
- **30-year zero**: Convexity ~ 900

Rule of thumb: Convexity ≈ $T^2$ for zeros at moderate yields.

### 13.9.4 When Convexity Matters

| Scenario | Convexity Relevance |
| :--- | :--- |
| Daily hedging (5-10 bp moves) | Low — duration sufficient |
| Weekly rebalancing (20-30 bp moves) | Moderate — consider convexity |
| Stress testing (50-100 bp moves) | High — convexity mandatory |
| Callable/MBS portfolios | Critical — sign can flip |

---

## 13.10 Summary

Convexity is the second derivative of price with respect to yield. While DV01 (duration) describes the linear tangent to the price-yield curve, convexity describes the curvature of the bond itself.

1. **Safety Cushion**: Positive convexity dampens losses when yields rise and amplifies gains when yields fall.

2. **$T^2$ Scaling**: Convexity grows with the square of maturity. 30-year bonds are convexity monsters.

3. **Error Correction**: For shocks >50 bps, linear DV01 models fail. Convexity corrections are mandatory for stress testing.

4. **Negative Convexity**: Callable bonds and MBS exhibit negative convexity ("frown"), meaning duration extends during sell-offs, exacerbating losses. As Tuckman warns, hedges mixing positive and negative convexity are "inherently unstable."

5. **Barbell vs Bullet**: Spreading cash flows (barbell) increases convexity; concentrating them (bullet) reduces it. The barbell wins in volatile markets; the bullet wins in stable markets.

6. **Volatility Trade**: Long convexity = Long volatility. Short convexity = Short volatility. Convexity lowers yields as compensation for its insurance value.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
| :--- | :--- | :--- |
| **Convexity ($C$)** | $\frac{1}{P}\frac{d^2P}{dy^2}$ | Corrects linear duration errors for large rate moves |
| **Positive Convexity** | Curvature "smile" | You gain more on rallies and lose less on sell-offs |
| **Negative Convexity** | Curvature "frown" | Found in Callables/MBS; price capped on rallies |
| **$T^2$ Scaling** | $C_{\text{zero}} \approx T^2$ | Long-dated bonds have exponentially more convexity |
| **Barbell** | Portfolio of Short + Long bonds | Higher convexity than a Bullet; long volatility |
| **Convexity Bias** | $\frac{1}{2}C(\Delta y)^2$ | The P&L "cushion" provided by curvature |
| **Jensen's Inequality** | $E[f(x)] > f(E[x])$ for convex $f$ | Why convexity lowers yields in a volatile world |
| **Extension Risk** | Duration rises as rates rise | Negative convexity creates pro-cyclical hedging |

---

## Notation for This Chapter

| Symbol | Definition |
| :--- | :--- |
| $P(y)$ | Bond price as function of yield |
| $C$ | Convexity (normalized second derivative) |
| $D$ | Modified Duration |
| $\Delta y$ | Yield change (in decimal, e.g., 0.01 = 100 bp) |
| $T$ | Maturity in years |
| $P_+$, $P_-$, $P_0$ | Prices at bumped-up, bumped-down, and base yields |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Define convexity mathematically. | $C = \frac{1}{P}\frac{d^2P}{dy^2}$ |
| 2 | What is the sign of the convexity term in P&L approximation for vanilla bonds? | Always positive: $+ \frac{1}{2} C (\Delta y)^2$ |
| 3 | How does convexity error scale with shock size? | Like the square of the shock: $(\Delta y)^2$ |
| 4 | If duration is matched, does a Barbell or Bullet have higher convexity? | The Barbell (convexity scales with $T^2$) |
| 5 | What creates negative convexity? | Embedded options (Callables) or prepayments (MBS) |
| 6 | What does negative convexity imply for DV01 as rates rise? | DV01 **increases** as rates rise (extension risk) |
| 7 | What is the relationship between convexity and volatility? | Long Convexity ≈ Long Volatility |
| 8 | Why don't traders always hold maximum convexity? | Convexity usually costs yield (negative carry) |
| 9 | For a zero-coupon bond, how does convexity relate to maturity? | Roughly proportional to $T^2$: $C = \frac{T(T+0.5)}{(1+y/2)^2}$ |
| 10 | What is the typical "bump size" for numeric convexity? | 1–5 basis points (balance noise vs accuracy) |
| 11 | State the Taylor expansion for bond price change including convexity. | $\frac{\Delta P}{P} \approx -D\Delta y + \frac{1}{2}C(\Delta y)^2$ |
| 12 | What is Jensen's Inequality for a convex function? | $E[f(x)] > f(E[x])$ |
| 13 | How does convexity affect bond yields? | Higher convexity → Lower yields (convexity is valuable) |
| 14 | For a 10-year zero at 5% yield, approximate the convexity. | $C \approx \frac{10 \times 10.5}{(1.025)^2} \approx 100$ |
| 15 | What is "extension risk"? | Duration increases as rates rise in negative convexity instruments |
| 16 | Why is MBS hedging difficult? | Negative convexity forces pro-cyclical rebalancing |
| 17 | What does positive convexity mean for DV01? | DV01 **falls** as rates increase |
| 18 | In a stable rate environment, which outperforms: Barbell or Bullet? | Bullet (higher yield, no convexity gains realized) |
| 19 | What units does convexity have? | Time squared (years²) |
| 20 | When should you include convexity in risk calculations? | For moves > 50 bp, stress tests, and optioned instruments |

---

## Mini Problem Set

**1. Basic Calculation**
A bond has $D = 5$ and $C = 30$. Calculate the estimated percentage price change for:
(a) A +50 bp shock using duration only
(b) A +50 bp shock using duration + convexity
(c) A +200 bp shock using duration + convexity

*Solution Sketch:*
(a) $-5 \times 0.005 = -2.50\%$
(b) $-2.50\% + \frac{1}{2}(30)(0.005)^2 = -2.50\% + 0.04\% = -2.46\%$
(c) $-5 \times 0.02 + \frac{1}{2}(30)(0.02)^2 = -10\% + 0.60\% = -9.40\%$

**2. Convexity Scaling**
If a 10-year zero has convexity $C_{10} \approx 100$, estimate the convexity of:
(a) A 5-year zero
(b) A 20-year zero
(c) A 30-year zero

*Solution Sketch:* Using $C \propto T^2$:
(a) $C_5 \approx 100 \times (5/10)^2 = 25$
(b) $C_{20} \approx 100 \times (20/10)^2 = 400$
(c) $C_{30} \approx 100 \times (30/10)^2 = 900$

**3. Barbell Construction**
Design a barbell using 2-year and 30-year zeros to match a 10-year bullet's duration. Assume a flat 5% curve.
(a) What are the weights?
(b) What is the convexity of each portfolio?
(c) Which outperforms if rates move ±100 bp?

*Solution Sketch:*
(a) Let $w$ be the 30-year weight. $w(30) + (1-w)(2) = 10$ → $w = 8/28 = 28.6\%$
(b) Bullet: $C \approx 100$. Barbell: $0.714 \times 5 + 0.286 \times 900 \approx 261$
(c) Barbell outperforms in both directions due to higher convexity.

**4. Callable Bond Analysis**
You are hedging a Callable Bond using a 5-year Treasury. The callable has negative convexity at low yields.
(a) If rates rally 50 bps, will your hedge over- or under-compensate?
(b) Explain why in terms of DV01 behavior.

*Solution Sketch:*
(a) Under-compensate. The callable's price is capped while the Treasury continues to rise.
(b) As rates fall, the callable's DV01 *decreases* (approaching call price), while the Treasury's DV01 increases. The hedge ratio becomes wrong.

**5. Numerical Estimation**
Given: $P_{-1bp} = 101.05$, $P_0 = 101.00$, $P_{+1bp} = 100.95$
(a) Estimate convexity using the central difference formula.
(b) Is this bond positively or negatively convex?

*Solution Sketch:*
(a) $C = \frac{100.95 - 2(101.00) + 101.05}{101.00 \times (0.0001)^2} = \frac{0.00}{0.00000101} = 0$
Wait—this gives zero. The prices are symmetric around $P_0$, implying near-zero local convexity. Check: $P_+ - P_0 = -0.05$, $P_0 - P_- = -0.05$. Same slopes → zero second derivative at this point.

**6. Convexity Value**
In a world where 1-year rates are expected to be 5% but could be 3% or 7% with equal probability:
(a) What is the expected value of a 1-year discount factor?
(b) What would the discount factor be at the expected rate?
(c) Which is larger? Explain using Jensen's Inequality.

*Solution Sketch:*
(a) $E[1/(1+r)] = 0.5/(1.03) + 0.5/(1.07) = 0.4854 + 0.4673 = 0.9527$
(b) $1/E[1+r] = 1/(1.05) = 0.9524$
(c) 0.9527 > 0.9524. Jensen's Inequality: $E[1/(1+r)] > 1/E[1+r]$ because $1/(1+r)$ is convex.

**7. Extension Risk Scenario**
An MBS portfolio has duration of 4 years at current rates. If rates rise 100 bp, duration extends to 6 years. You initially hedged with $100M of 4-year Treasuries.
(a) Are you over-hedged or under-hedged after the rate move?
(b) What must you do to restore the hedge?

*Solution Sketch:*
(a) Under-hedged. MBS duration increased to 6 years, but hedge is still sized for 4 years.
(b) Sell additional Treasuries (or add duration to hedge) to match the extended duration.

**8. Duration-Convexity Approximation Accuracy**
A 30-year zero has $D = 28.6$ and $C = 870$. Rates rise 150 bp.
(a) Duration-only estimate of % price change?
(b) Duration + convexity estimate?
(c) If actual change is -35.2%, what accounts for the remaining error?

*Solution Sketch:*
(a) $-28.6 \times 0.015 = -42.9\%$
(b) $-42.9\% + \frac{1}{2}(870)(0.015)^2 = -42.9\% + 9.8\% = -33.1\%$
(c) Error = $|-35.2\% - (-33.1\%)| = 2.1\%$. Higher-order terms (third derivative and beyond).

---

## Source Map

### (A) Verified Facts — Source-Backed

| Fact | Source |
|------|--------|
| Convexity definition $C = \frac{1}{P}\frac{d^2P}{dy^2}$ | Tuckman Ch 5 Eq 5.14; Hull Ch 4.11 |
| Second-order Taylor expansion $\frac{\Delta P}{P} \approx -D\Delta y + \frac{1}{2}C(\Delta y)^2$ | Tuckman Ch 5 Eq 5.20; Hull Ch 4.11 |
| Numerical estimation via central differences | Tuckman Ch 5 Eqs 5.15-5.17, Table 5.3 |
| "Extra precision is often necessary when calculating second derivatives" | Tuckman Ch 5 (explicit quote) |
| Positive convexity = "DV01 falls as rates increase" | Tuckman Ch 5 (explicit quote) |
| Zero-coupon convexity $C = \frac{T(T+0.5)}{(1+y/2)^2}$ | Tuckman Ch 6 Eq 6.36 |
| "Convexity increases with the square of maturity" | Tuckman Ch 6 (explicit statement) |
| Barbell vs Bullet example with convexity 221.30 vs 81.38 | Tuckman Ch 6 Eq 6.37 |
| "Spreading out cash flows raises convexity" | Tuckman Ch 6 (explicit quote) |
| Negative convexity in callable bonds and MBS | Tuckman Ch 5, Ch 19, Ch 21 |
| "Care must be exercised when mixing securities of positive and negative convexity" | Tuckman Ch 5 (explicit quote) |
| Jensen's Inequality convexity effect: $E[1/(1+r)] > 1/E[1+r]$ | Tuckman Ch 10 Eq 10.6 |
| Convexity lowers yields; effect increases with volatility | Tuckman Ch 10 |
| "Long convexity position is long volatility" | Tuckman Ch 5 (explicit quote) |
| Yield-based coupon bond convexity formula | Tuckman Ch 6 Eq 6.35 |
| Convexity of portfolio = weighted convexity of components | Tuckman Ch 5; Hull Ch 4.11 |
| "Bullet outperforms if rates move by a small amount; barbell outperforms if rates move by a large amount" | Tuckman Ch 6 (explicit quote) |

### (B) Reasoned Inference — Derived from (A)

| Inference | Derivation |
|-----------|------------|
| All worked example numerical calculations | Algebraic application of source-backed formulas |
| Unit conversion formulas (bp ↔ decimal) | Dimensional analysis from definitions |
| Symmetry of convexity term for $\pm\Delta y$ | Follows from $(\Delta y)^2$ dependence |
| Duration-only error scaling as $(\Delta y)^2$ | Taylor expansion remainder analysis |
| DV01-neutral portfolio with positive net convexity profits both ways | Second-order term always positive for $C > 0$ |
| $T^2$ scaling pattern for convexity | Structure of zero-coupon formula |
| Extension risk mechanism | Negative convexity implies rising duration as rates rise |

### (C) Flagged Uncertainties

| Topic | Uncertainty |
|-------|-------------|
| Callable bond numerical convexity values | The specific numbers (-120, -223, -147) in the callable example are illustrative. Actual values depend on call schedule, option model, and volatility assumptions. |
| MBS convexity dynamics | MBS convexity is path-dependent and model-sensitive; numerical values require prepayment models not specified in the sources. |
| Optimal bump size | Sources agree on 1-5 bp as typical, but optimal choice depends on instrument and curve smoothness. |
