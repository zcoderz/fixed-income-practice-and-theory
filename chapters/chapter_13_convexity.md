# Chapter 13: Convexity — Second-Order P&L and When DV01 Breaks

---

## Introduction

Your risk system shows the portfolio is duration-hedged. DV01 is flat. Then the Fed surprises everyone—rates move 75 basis points in an afternoon. Your "hedged" position loses $3 million. How?

The answer lies in the one dimension that duration ignores: **curvature**. Duration captures the slope of the price-yield relationship, but bonds are not linear instruments. The price-yield curve bends, and this bending—called **convexity**—determines whether your hedge holds in a crisis or falls apart precisely when you need it most.

For small daily moves of 3-5 basis points, duration is adequate. But for the 50-100 basis point shocks that define stress scenarios, convexity dominates the error. A duration-only model can misestimate losses by several percent of portfolio value. Even more dangerously, for instruments with embedded options—callable bonds and mortgage-backed securities—convexity can flip negative, causing positions to hemorrhage money faster than duration predicts precisely when protection is needed most.

This chapter extends the risk framework beyond the linear world of DV01. We cover:

1. **The Geometry of Convexity** (Section 13.1): Why "curvature" is the mathematically correct way to visualize second-order risk.
2. **Defining Convexity Precisely** (Section 13.2): The normalized second derivative, dollar convexity (gamma), and numerical estimation.
3. **Convexity for Standard Bonds** (Section 13.3): Analytical formulas and the critical $T^2$ scaling rule.
4. **When DV01 Breaks** (Section 13.4): Quantifying the error of linear hedging during large volatility events, including P&L attribution.
5. **Convexity and Hedging** (Section 13.5): DV01 mismatches, immunization with convexity matching, and the "self-funding" trade.
6. **Positive vs Negative Convexity** (Section 13.6): Understanding the "short gamma" trap of callable bonds and MBS, including effective convexity and the "death spiral."
7. **The Barbell vs Bullet Trade** (Section 13.7): Structuring portfolios to monetize convexity differences, with breakeven analysis.
8. **Convexity as Volatility Exposure** (Section 13.8): Jensen's Inequality and why convexity is priced.
9. **Beyond Parallel: Key-Rate Convexity** (Section 13.9): A brief forward reference to multi-dimensional second-order risk.

Chapter 12 established duration as the first-order risk measure. This chapter reveals its limitations and provides the tools to manage second-order risk. For those moving from middle office to trading desks, understanding convexity is non-negotiable: it separates traders who survive volatility events from those who don't.

---

## 13.1 The Price-Yield Curve Is Not a Straight Line

### 13.1.1 Why Curvature Matters

Consider a simple zero-coupon bond maturing in 10 years. At a yield of 5%, its price is approximately $61.03 per $100 face. If we plot price against yield, the resulting curve slopes downward—higher yields mean lower prices—but crucially, it bends *upward*, like a smile. This geometric property is what we call **convexity**.

This curvature has immediate financial consequences. As yields **rise** and prices fall, the slope of the curve flattens. This means the bondholder loses money, but their sensitivity to further rate increases (DV01) decreases, essentially "cushioning" the loss. Conversely, as yields **fall** and prices rise, the slope steepens. The bondholder makes money, and their sensitivity increases, accelerating the gains.

Tuckman emphasizes this point directly: "the property of positive convexity may also be thought of as the property that DV01 falls as rates increase." In simple terms: positive convexity implies that the bond's duration works in your favor in both market directions.

> **Analogy: The Shock Absorber**
>
> Convexity is the shock absorber on a car.
> - **Rates Rise (Bad News)**: Price falls, but convexity *slows* the fall (DV01 decreases). The shock is dampened.
> - **Rates Fall (Good News)**: Price rises, and convexity *accelerates* the rise (DV01 increases). The gain is amplified.
> - **Result**: Positive convexity helps you win more when you're right, and lose less when you're wrong.

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

> **Note on Units:** Convexity has units of time squared (years²). This becomes clear from the zero-coupon formula in Section 13.3.1, where convexity scales with $T^2$.

### 13.2.2 Dollar Convexity: The Gamma Analog

While normalized convexity is useful for comparing bonds of different prices, it is also useful to work with **dollar convexity**—the absolute second derivative without normalization.

Hull (RM Ch 9) defines this explicitly: "The dollar convexity of a bond, $C_\$$, can be defined analogously to dollar duration as the product of convexity and the value of the bond. This means that $C_\$ = d^2B/dy^2$."

Hull further notes that "dollar convexity is similar to the gamma measure introduced in Chapter 8"—the same gamma that options traders live and die by.

$$\boxed{\text{Dollar Convexity} = P \times C = \frac{d^2P}{dy^2}}$$

**Why Dollar Convexity Matters:** Even when a system does not report it directly, dollar convexity is a natural object for P&L attribution because curvature P&L can be written as:

$$\text{Convexity P&L} = \frac{1}{2} \times \text{Dollar Convexity} \times (\Delta y)^2$$

Hull (RM Ch 9) confirms: "The dollar convexity for a portfolio worth $P$ can be defined as $P$ times the convexity. This is a measure of the gamma of the portfolio with respect to interest rates."

If you want to plug in a yield move in **basis points** (instead of decimal yield units), define:

- $\Delta y_{\text{bp}}$: change in yield in basis points
- $\Delta y = 0.0001 \times \Delta y_{\text{bp}}$

Then the convexity P&L can be written as:

$$\boxed{\text{Convexity P\&L}=\underbrace{\left[\frac{1}{2}\times \text{Dollar Convexity}\times (0.0001)^2\right]}_{\text{Convexity01 (\\$/bp}^2\text{)}}\times (\Delta y_{\text{bp}})^2}$$

This is often the most desk-friendly way to compute convexity P&L because it takes the move in bps and makes the quadratic scaling explicit.

> **Desk Reality: Unit Confusion Creates 10×–1000× Errors**
>
> Dollar convexity is defined with respect to **decimal yield** ($y=5\\%$ is $0.05$). That makes the raw number look enormous at the portfolio level.
>
> Example: if a $\\$100mm portfolio has normalized convexity $C=100$, then dollar convexity is $P\\times C = 10\\text{bn}$. The convexity P&L is still intuitive:
> - 10bp move: $\frac{1}{2} \\times 10\\text{bn} \\times (0.001)^2 = \\$5{,}000$
> - 50bp move: $\frac{1}{2} \\times 10\\text{bn} \\times (0.005)^2 = \\$125{,}000$
> - 100bp move: $\frac{1}{2} \\times 10\\text{bn} \\times (0.01)^2 = \\$500{,}000$
>
> The same example implies $\text{Convexity01}=\\frac{1}{2}\\times 10\\text{bn}\\times (0.0001)^2=\\$50$ per bp², so $\text{Convexity P\&L}\\approx 50\\times(\\Delta y_{\\text{bp}})^2$.
>
> **Critical:** Always verify whether your system reports normalized convexity, dollar convexity, or a pre-scaled “convexity01/gamma01.” Confusing them causes order-of-magnitude errors.

### 13.2.3 Computing Convexity Numerically

In practice, analytic formulas are not always available for complex instruments. In such cases, convexity is estimated using the same "bump and reprice" logic used for DV01. Instead of measuring a single slope, we estimate the *change* in slope using a **central difference method**:

$$\boxed{C \approx \frac{1}{P_0} \left[ \frac{P_+ - 2P_0 + P_-}{(\Delta y)^2} \right]}$$

where:
- $P_-$ is the price at $y - \Delta y$
- $P_0$ is the current price
- $P_+$ is the price at $y + \Delta y$

Tuckman demonstrates this method in detail in Table 5.3, noting that "extra precision is often necessary when calculating second derivatives" because the calculation involves a difference of differences. In practice, choose the bump size to balance numerical noise (too small) and higher-order contamination (too large); the right choice depends on the instrument and your curve representation.

### 13.2.4 Worked Example A: Tuckman's Table 5.3 Methodology

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

**Sanity Check:** The convexity is positive (as expected for a vanilla bond), and the magnitude (73.6) is plausible for a bond with roughly 8-year duration. ✓

---

## 13.3 Convexity for Standard Bonds

### 13.3.1 Zero-Coupon Bond Convexity

For a zero-coupon bond, convexity can be derived analytically, offering clear intuition for how it behaves across maturities. Given the price of a zero-coupon bond $P = 100 / (1 + y/2)^{2T}$ under semiannual compounding, differentiating twice and dividing by $P$ yields the closed-form solution. Tuckman derives this as equation (6.36):

$$\boxed{C_{\text{zero}} = \frac{T(T + 0.5)}{(1 + y/2)^2}}$$

This formula reveals a fundamental rule of thumb for fixed income traders: **Convexity scales with the square of maturity ($T^2$).**

Tuckman explains the intuition: "From (6.36) it is clear that longer-maturity zeros have greater convexity. In fact, convexity increases with the square of maturity."

### 13.3.2 Worked Example B: The Power of $T^2$

To visualize this scaling, let us compare the convexity of zero-coupon bonds at a 5% yield across different maturities:

| Maturity ($T$) | Approx $T^2$ | Actual Convexity | Interpretation |
| :--- | :--- | :--- | :--- |
| **2 Years** | 4 | **4.76** | Negligible curvature |
| **5 Years** | 25 | **26.17** | Small curvature |
| **10 Years** | 100 | **99.94** | Significant curvature |
| **20 Years** | 400 | **389.04** | Large curvature |
| **30 Years** | 900 | **870.91** | Massive curvature |

A 30-year bond has roughly $9\times$ the convexity of a 10-year bond, mirroring the relationship $30^2 = 9 \times 10^2$. This geometric explosion is why "convexity trades" almost invariably focus on the long end of the yield curve, where the "gamma" is most potent.

> **Desk Reality: The "30-Year Dominance"**
>
> In practice, 90%+ of convexity trading happens in the 30-year sector. Given $C \propto T^2$, the 2y/5y/10y sectors are effectively "linear" compared to 30y. When a trader says "I'm putting on a convexity trade," they almost certainly mean long-end exposure. The 2-year bond has convexity of ~5; the 30-year has convexity of ~900. The 30-year is literally 180x more convex.

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

### 13.4.1 Worked Example C: A 100 Basis Point Shock

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
| **200 bp** | -0.184 | 64× |

This exponential growth is why linear models fail spectacularly during crashes. Tuckman notes: "While convexity is usually a larger number than duration, for relatively small changes in rate the change in rate is so much larger than the change in rate squared that the duration effect dominates."

For standard daily moves of 5–10 bps, duration is often adequate. For VaR scenarios or stress tests involving 50–100 bp moves, ignoring convexity can materially misstate risk.

### 13.4.3 Convexity in Daily P&L Attribution

When risk managers decompose daily P&L, the convexity term appears explicitly. A common second-order attribution uses the yield move in basis points:

$$\boxed{\text{Daily P\&L}\approx \underbrace{-\text{DV01}\times \Delta y_{\text{bp}}}_{\text{Duration P\&L}}+\underbrace{\text{Convexity01}\times(\Delta y_{\text{bp}})^2}_{\text{Convexity P\&L}}+\text{Carry}+\text{Unexplained}}$$

Where:
- **DV01** is in dollars per bp (see Chapter 11 for conventions)
- $\Delta y_{\text{bp}}$ is the signed yield change in bp
- **Convexity01** is the $/bp² coefficient defined in Section 13.2.2

**Sign Convention:** For vanilla bonds with positive convexity, the convexity term is **always positive**—whether rates rise or fall. This is the mathematical signature of convexity's "cushion."

**When It Shows Up:** Convexity P&L is typically negligible for daily moves (5–10bp) because it scales with $(\Delta y_{\text{bp}})^2$. But during volatility events (50bp+ moves), convexity can dominate the explain.

> **Desk Reality: Diagnosing Large Unexplained P&L**
>
> If your desk's P&L explain has large unexplained residuals after a vol spike, the first place to look is whether your convexity is being calculated correctly. Common culprits:
> - **Wrong bump size** in numerical convexity (too large → captures higher-order effects)
> - **Normalized vs dollar convexity confusion** (off by a factor of price)
> - **Stale convexity** (hasn't been recalculated as yields moved)
> - **Missing optionality** (callable bonds need effective convexity, not modified)
>
> A 50bp move on $100mm with convexity of 100: Convexity P&L = $\frac{1}{2} \times 100 \times 0.005^2 \times \$100mm = \$125,000$. If this isn't in your explain, you have a $125k hole.

---

## 13.5 Convexity and Hedging

### 13.5.1 DV01 Hedges with Convexity Mismatch

Traders often construct "DV01 Neutral" portfolios and believe they are fully hedged. However, if the long and short legs of the trade have significantly different convexities, the portfolio is only hedged for *small* moves. This is known as a **convexity mismatch**.

Tuckman provides a detailed example in his "Hedging Example, Part II" where he shows that a market maker hedging options with bonds is "short convexity" and loses money whether rates rise or fall.

### 13.5.2 Worked Example D: The "Self-Funding" Trade

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
> Traders call convexity "Gamma" (borrowed from options parlance).
> - **Long Gamma**: You own convexity (like a straddle). You want the market to move—high volatility. You make money on the wiggles.
> - **Short Gamma**: You sold convexity. You want the market to sleep—low volatility. You bleed money if the market moves.
>
> The choice depends on your volatility view: is realized vol going to exceed the implied vol priced into the yield curve?

### 13.5.3 Immunization with Convexity Matching

Hull (RM Ch 9) provides the principle for immunization: "A portfolio consisting of long and short positions in interest-rate-dependent assets can be protected against relatively small parallel shifts in the yield curve by ensuring that its duration is zero. It can be protected against relatively large parallel shifts in the yield curve by ensuring that its duration and convexity are both zero or close to zero."

Luenberger (Ch 3.7) extends this to asset-liability management. The conditions for full immunization are:

$$\boxed{\begin{aligned}
D_{\text{assets}} &= D_{\text{liabilities}} \\
C_{\text{assets}} &\geq C_{\text{liabilities}}
\end{aligned}}$$

**The Duration-Only Trap:** A duration-matched hedge is only locally immunized. Large rate moves break the hedge because duration itself changes.

**Convexity Enhancement:** By matching both duration AND convexity, the immunization holds for larger moves. Luenberger notes: "Convexity can be used to improve immunization in the sense that, compared to ordinary immunization, a closer match of asset portfolio value and obligation value is maintained as yields vary."

**Implementation:** Generally, at least three bonds are required to match present value, duration, and convexity simultaneously.

> **Desk Reality: Pension Fund ALM**
>
> Pension funds with long-dated liabilities (20-40 years) must match both duration AND convexity. A 100bp rate move can shift liability values by 15-20%. If the asset portfolio has lower convexity than the liabilities, the fund becomes underfunded in a rally (assets rise less than liabilities) AND underfunded in a sell-off (assets fall more than liabilities). This is the "no-win" scenario that convexity matching prevents.

---

## 13.6 Positive vs Negative Convexity

So far, we have assumed convexity is positive (the "smile"). However, for instruments with embedded options—specifically **Callable Bonds** and **Mortgage-Backed Securities (MBS)**—convexity can turn **negative** (a "frown").

Tuckman states explicitly: "Fixed income securities need not be positively convex at all rate levels. Some important examples of negative convexity are callable bonds... and mortgage-backed securities."

> **Visualization: The Smile and The Frown**
>
> - **Positive Convexity**: A "Smiley Face" curve. Price rises faster than expected in rallies, falls slower in sell-offs.
> - **Negative Convexity**: A "Frowning Face" curve. Price rises *slower* than expected in rallies (upside capped), falls faster in sell-offs (downside accelerated).

### 13.6.1 The Mechanics of Negative Convexity

In callable bonds, as yields fall, the issuer is likely to call the bond to refinance at a lower rate. This caps the price appreciation at the call price (e.g., 100). Graphically, the price-yield curve flattens out and then bends downwards, creating a concave shape.

Tuckman explains: "Figure 19.4 also shows that an embedded call option induces negative convexity. For the callable bond price curve to resemble the three-year curve at low rates and the 10-year curve at high rates, the callable bond curve must be negatively convex."

Similarly, in MBS, as rates fall, homeowners refinance their mortgages, causing the bond to return principal precisely when it would otherwise be most valuable.

### 13.6.2 Worked Example E: The Callable Trap

Consider a 5% Callable Bond (callable at par in 1 year) compared to a vanilla non-callable bond. As yields drop, the difference becomes stark:

| Yield Level | Vanilla Price | Callable Price | Convexity (Callable) |
| :--- | :--- | :--- | :--- |
| **6% (OTM)** | 92.56 | 91.87 | **-120** (Negative) |
| **5% (ATM)** | 100.00 | 96.95 | **-223** (Deeply Negative) |
| **4% (ITM)** | 108.18 | 100.03 | **-147** (Negative) |

These numbers are illustrative; actual convexity depends on the call schedule and the option model/volatility assumptions used for pricing.

At a 5% yield, the convexity is **-223**. This negative number has profound hedging implications. If rates drop 100 bps to 4%, the vanilla bond gains 8.18, but the callable bond gains only 3.08 because it is capped at par. If you hedged this callable bond with a standard DV01 ratio, you would massively underperform the hedge.

Tuckman warns: "care must be exercised when mixing securities of positive and negative convexity because the resulting hedges or comparative return estimates are inherently unstable."

### 13.6.3 Extension Risk: The Destabilizing Dynamic

**Deep Insight**: Negative convexity implies that **DV01 increases as yields rise**.

- **Positive Convexity**: Yields $\uparrow$ $\implies$ Price $\downarrow$ but Duration $\downarrow$ (Self-stabilizing).
- **Negative Convexity**: Yields $\uparrow$ $\implies$ Price $\downarrow$ and Duration $\uparrow$ (Destabilizing).

This phenomenon is known as "extension risk." Tuckman illustrates this in Figure 19.5, showing how "the duration of callable bonds increases as rates rise above the coupon."

It explains why MBS hedging is difficult: as rates move, the duration (and DV01) of the underlying instrument can change materially, which means yesterday's hedge ratio can be wrong today. When many participants must rebalance in the same direction, this can amplify volatility.

### 13.6.4 The MBS Convexity "Death Spiral"

The extension risk mechanism in MBS can create a dangerous feedback loop when many market participants are positioned similarly:

**Anatomy of a Sell-off:**
1. **Rates rise** → MBS duration extends (prepayments slow as refinancing becomes unattractive)
2. **Portfolio managers become under-hedged** → Their hedge was sized for shorter duration
3. **Must sell Treasuries/futures to add duration hedge** → Adds to selling pressure
4. **Selling pressure pushes rates higher** → Returns to step 1
5. **Cycle repeats** → Pro-cyclical feedback loop

> **Practitioner Note: Mortgage Hedging Flows and Swap Spreads**
>
> Tuckman documents episodes where swap spreads tended to narrow in sharp rallies and widen in sharp sell-offs, and market commentators attributed part of this pattern to mortgage hedging flows.
>
> One desk story (following Tuckman's discussion) is:
> - **Rates fall (rally):** MBS duration shortens. Hedgers may need to **add duration**, e.g. by **receiving in swaps**, which can contribute to **swap spread narrowing**.
> - **Rates rise (sell-off):** MBS duration extends. Hedgers may need to **shed duration**, e.g. by **paying in swaps**, which can contribute to **swap spread widening**.

### 13.6.5 Effective Convexity (OAS-Based)

For callable bonds and MBS, yield-based convexity is misleading because the embedded option's value changes with rates. We need **effective convexity**.

Tuckman notes: "duration may be computed for any assumed change in the term structure of interest rates. This general definition is also called effective duration." The same logic applies to convexity.

**Definition:** Effective Convexity uses an option model to compute prices at shifted rates:

$$\boxed{C_{\text{effective}} = \frac{P_- - 2P_0 + P_+}{P_0 \times (\Delta y)^2}}$$

where $P_-$, $P_0$, $P_+$ are computed using an OAS model that accounts for the changing option value.

**Key Difference:**
- **Modified (Yield-Based) Convexity:** Holds cash flows constant; appropriate for vanilla bonds
- **Effective Convexity:** Allows cash flows to change (prepayments accelerate, calls become likely); required for optioned instruments

**When to Use Which:**
| Instrument | Convexity Measure |
| :--- | :--- |
| Treasury bonds | Modified convexity |
| Non-callable corporates | Modified convexity |
| Callable bonds | Effective convexity |
| MBS/CMOs | Effective convexity |
| Swaptions (if held) | Effective convexity |

---

## 13.7 The Barbell vs Bullet Trade

The most classic convexity trade is the **Barbell vs. Bullet**, which Tuckman treats extensively in Chapter 6.

- **Bullet**: A portfolio concentrated at a single maturity (e.g., 9-year zero).
- **Barbell**: A portfolio split between short (e.g., 2-year) and long (e.g., 30-year) maturities.

You can weight them to have the **exact same duration**. So why choose one over the other? **Convexity.**

### 13.7.1 Worked Example F: Tuckman's Barbell vs Bullet

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

### 13.7.3 Quantifying the Breakeven

How much must rates move for the barbell to break even? We can calculate this explicitly.

**Setup:**
- Barbell convexity advantage: $\Delta C = 221.30 - 81.38 = 139.92$
- Yield give-up (assume): 8 bp/year
- Holding period: 1 year

**Breakeven Calculation:**

The convexity gain must offset the yield loss:
$$\frac{1}{2} \Delta C \times (\Delta y)^2 = \text{Yield Give-up}$$

Solving for $\Delta y$:
$$\Delta y = \sqrt{\frac{2 \times 0.0008}{139.92}} = \sqrt{0.0000114} \approx 0.0034 = 34 \text{ bp}$$

**Interpretation:** If rates move more than 34bp in either direction over the year, the barbell wins. If rates move less, the bullet wins.

> **Practitioner Note: Convexity Cost Rule of Thumb**
>
> In normal volatility regimes, the market prices convexity at roughly 1-2bp of yield per 100 units of convexity difference. This varies significantly with the vol regime:
> - **Low vol environment (VIX < 15):** Convexity is cheap; barbells attract buyers
> - **High vol environment (VIX > 25):** Convexity is expensive; market has already priced in the moves
>
> The barbell trade is fundamentally a bet that realized volatility will exceed the implied volatility embedded in the yield spread.

### 13.7.4 P&L Scenario Table

The following table shows approximate P&L for duration-matched barbell vs bullet across rate scenarios (assuming 1-year holding period):

| Scenario | Rate Move | Bullet P&L | Barbell P&L | Winner |
|----------|-----------|------------|-------------|--------|
| **Quiet** | 0 bp | +8bp carry | 0 bp | **Bullet** |
| **Modest rally** | -30 bp | +2.60% | +2.64% | **Barbell** |
| **Modest selloff** | +30 bp | -2.44% | -2.40% | **Barbell** |
| **Large rally** | -100 bp | +9.35% | +9.87% | **Barbell** |
| **Large selloff** | +100 bp | -7.56% | -7.04% | **Barbell** |

**The Pattern:** The barbell "straddle" profile—loses in calm, wins in chaos. The breakeven is somewhere around 30-35bp moves.

### 13.7.5 General Principle

Tuckman provides the generalization: "The intuition gained from the barbell-bullet example can be used to understand the convexity properties of other portfolios. In general, spreading out the cash flows of a portfolio (without changing its duration) raises its convexity."

This principle applies to any portfolio construction: spreading cash flows increases convexity; concentrating them reduces it.

---

## 13.8 Convexity as Volatility Exposure

Why does convexity behave like volatility? The answer lies in **Jensen's Inequality**: for a convex function $f$, $E[f(x)] \geq f(E[x])$.

If you own a convex bond, the average price after a random volatility event is *higher* than the price at the average yield.

### 13.8.1 Jensen's Inequality and Bond Pricing

Tuckman develops this rigorously in Chapter 10. He shows that the pricing function of a zero, $1/(1+r)$, is convex, which implies:

$$E\left[\frac{1}{1+r}\right] > \frac{1}{E[1+r]} = \frac{1}{1+E[r]}$$

This means that in a world with interest rate volatility, bond prices are *higher* than they would be if rates were certain to equal their expected value. Equivalently, yields are *lower* than the expected rate—this difference is the "convexity value."

Tuckman illustrates: "Even though the one-year rate is 10% and the expected one-year rate in one year is 10%, the two-year spot rate is 9.982%. The 1.8-basis point difference... is the effect of convexity on that spot rate."

### 13.8.2 Worked Example G: Jensen's Inequality Numerical

Consider a two-state world where rates are 5% today. In one year, rates will be either 3% or 7% with equal probability.

**Step 1: Expected price of a 1-year zero at $t=1$:**
$$E[P] = 0.5 \times \frac{100}{1.03} + 0.5 \times \frac{100}{1.07} = 0.5 \times 97.087 + 0.5 \times 93.458 = 95.272$$

**Step 2: Price at the expected rate:**
$$P(E[r]) = \frac{100}{1.05} = 95.238$$

**Step 3: Convexity value:**
$$\text{Convexity Value} = 95.272 - 95.238 = 0.034 \text{ (3.4 cents per 100 face)}$$

**Volatility Scaling:** Now double the volatility—rates are either 2% or 8%:
$$E[P] = 0.5 \times \frac{100}{1.02} + 0.5 \times \frac{100}{1.08} = 0.5 \times 98.039 + 0.5 \times 92.593 = 95.316$$
$$\text{Convexity Value} = 95.316 - 95.238 = 0.078 \text{ (7.8 cents)}$$

The convexity value more than doubled when volatility doubled. In fact, Tuckman shows the relationship is roughly quadratic in volatility.

### 13.8.3 Convexity Value Increases with Volatility

Tuckman shows that the convexity effect on yield is approximately:

$$\boxed{\text{Convexity Effect on Yield} \approx -\frac{1}{2} C \sigma^2}$$

where $\sigma$ is the yield volatility. This formula has profound implications:

- **Long Convexity**: You are long gamma. You want realized volatility > implied volatility.
- **Short Convexity**: You are short gamma. You want realized volatility < implied volatility.

Tuckman confirms: "the value of convexity, measured by the distance between the rates assuming no volatility and the rates assuming volatility, increases with volatility."

And concludes: "it is in this sense... that a long convexity position is long volatility while a short convexity position is short volatility."

### 13.8.4 Intuitive Summary

- **Scenario A**: Yields stay flat. Price = $P_0$.
- **Scenario B**: Yields go up 50 bps or down 50 bps with 50/50 probability.
  - If rates Up: Loss is dampened by convexity.
  - If rates Down: Gain is amplified by convexity.
  - **Average Price**: $(P_{up} + P_{down})/2 > P_0$.

This "convexity bias" means that in a volatile market, high-convexity bonds tend to outperform low-convexity bonds, all else equal. This is why convexity is priced at a premium—manifested as lower yields on high-convexity instruments.

---

## 13.9 Beyond Parallel: Key-Rate Convexity

### 13.9.1 The Limitation of Scalar Convexity

Standard convexity assumes parallel shifts—all rates move by the same amount. But real yield curves twist, steepen, and flatten. Just as key-rate DV01 decomposes duration risk by tenor, **key-rate convexity** decomposes gamma risk.

### 13.9.2 When Key-Rate Convexity Matters

Key-rate convexity is most important for:
- **Swaption books**: Convexity is concentrated at specific exercise tenors
- **MBS portfolios**: Prepayment-driven convexity varies by mortgage vintage and rate environment
- **Exotic structures**: CMOs, callables with specific call schedules

**Forward Reference:** Chapter 14 covers key-rate DV01 in detail; the same bucketing concept applies to second-order sensitivities. For most vanilla portfolios, scalar convexity is sufficient. For options-intensive books, key-rate decomposition becomes essential.

---

## 13.10 Practical Notes and Sanity Checks

### 13.10.1 Bump Size Selection

When computing convexity numerically:
- **Too small**: Numerical precision errors can dominate (you are taking a “difference of differences”).
- **Too large**: The estimate can mix in higher-order effects beyond the quadratic term.
- **Practical approach**: Start with small symmetric bumps (Tuckman illustrates 1bp increments), then sanity-check stability by varying the bump size.

### 13.10.2 Sign Checks

- **Vanilla bonds**: Always positive convexity (smile)
- **Callable bonds**: Negative convexity at low yields (frown)
- **MBS**: Typically negative convexity due to prepayment
- **Long receiver swaptions**: Very high positive convexity

### 13.10.3 Order of Magnitude

For vanilla bonds:
- **2-year zero**: Convexity ~ 5
- **5-year zero**: Convexity ~ 26
- **10-year zero**: Convexity ~ 100
- **20-year zero**: Convexity ~ 390
- **30-year zero**: Convexity ~ 900

Rule of thumb: Convexity ≈ $T^2$ for zeros at moderate yields.

### 13.10.4 When Convexity Matters

| Scenario | Convexity Relevance |
| :--- | :--- |
| Daily hedging (5-10 bp moves) | Low — duration sufficient |
| Weekly rebalancing (20-30 bp moves) | Moderate — consider convexity |
| Stress testing (50-100 bp moves) | High — convexity mandatory |
| Callable/MBS portfolios | Critical — sign can flip |
| VaR calculations | High — especially tail risk |

---

## Summary

Convexity is the second derivative of price with respect to yield. While DV01 (duration) describes the linear tangent to the price-yield curve, convexity describes the curvature of the bond itself.

1. **Safety Cushion**: Positive convexity dampens losses when yields rise and amplifies gains when yields fall. It's a one-way benefit—always helps, never hurts.

2. **$T^2$ Scaling**: Convexity grows with the square of maturity. 30-year bonds are convexity monsters—literally 180x more convex than 2-year bonds.

3. **Error Correction**: For shocks >50 bps, linear DV01 models fail. Convexity corrections are mandatory for stress testing. The error scales with $(\Delta y)^2$.

4. **Dollar Convexity (Gamma)**: Dollar convexity = $P \times C$ is the un-normalized second derivative used in convexity P&L calculations. In practice, always verify units/scaling (some systems report a pre-scaled $/bp^2$ number like Convexity01).

5. **Negative Convexity**: Callable bonds and MBS exhibit negative convexity ("frown"), meaning duration extends during sell-offs, exacerbating losses. As Tuckman warns, hedges mixing positive and negative convexity are "inherently unstable."

6. **Extension Risk and the Death Spiral**: MBS negative convexity can create pro-cyclical hedging flows that amplify volatility in stressed markets.

7. **Barbell vs Bullet**: Spreading cash flows (barbell) increases convexity; concentrating them (bullet) reduces it. The barbell wins in volatile markets; the bullet wins in stable markets. The breakeven can be calculated explicitly.

8. **Volatility Trade**: Long convexity = Long volatility. Short convexity = Short volatility. Convexity lowers yields as compensation for its insurance value. The relationship is $-\frac{1}{2}C\sigma^2$.

9. **Immunization**: Duration matching alone is insufficient for large moves. Hull and Luenberger emphasize matching both duration AND convexity for robust immunization.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
| :--- | :--- | :--- |
| **Convexity ($C$)** | $\frac{1}{P}\frac{d^2P}{dy^2}$ | Corrects linear duration errors for large rate moves |
| **Dollar Convexity ($\Gamma$)** | $P \times C = \frac{d^2P}{dy^2}$ | Un-normalized second derivative; check unit conventions |
| **Convexity01** | $\frac{1}{2}\Gamma(0.0001)^2$ | $/bp^2 coefficient used with $(\Delta y_{bp})^2$ |
| **Positive Convexity** | Curvature "smile" | You gain more on rallies and lose less on sell-offs |
| **Negative Convexity** | Curvature "frown" | Found in Callables/MBS; price capped on rallies |
| **$T^2$ Scaling** | $C_{\text{zero}} \approx T^2$ | Long-dated bonds have exponentially more convexity |
| **Effective Convexity** | OAS-based numerical convexity | Required for option-embedded instruments |
| **Barbell** | Portfolio of Short + Long bonds | Higher convexity than a Bullet; long volatility |
| **Convexity Bias** | $\frac{1}{2}C(\Delta y)^2$ | The P&L "cushion" provided by curvature |
| **Jensen's Inequality** | $E[f(x)] > f(E[x])$ for convex $f$ | Why convexity lowers yields in a volatile world |
| **Extension Risk** | Duration rises as rates rise | Negative convexity creates pro-cyclical hedging |
| **Immunization** | Match duration AND convexity | Required for robust ALM against large moves |

---

## Notation for This Chapter

| Symbol | Definition |
| :--- | :--- |
| $P(y)$ | Bond price as function of yield |
| $C$ | Convexity (normalized second derivative) |
| $C_\$$ | Dollar convexity ($P \times C$) |
| $D$ | Modified Duration |
| $\Delta y$ | Yield change (in decimal, e.g., 0.01 = 100 bp) |
| $\Delta y_{\text{bp}}$ | Yield change in basis points |
| Convexity01 | $\frac{1}{2}\,C_\$(0.0001)^2$ (a $/bp^2$ coefficient) |
| $T$ | Maturity in years |
| $P_+$, $P_-$, $P_0$ | Prices at bumped-up, bumped-down, and base yields |
| $\sigma$ | Yield volatility (annualized standard deviation) |
| $C_{\text{effective}}$ | Effective (OAS-based) convexity |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Define convexity mathematically. | $C = \frac{1}{P}\frac{d^2P}{dy^2}$ |
| 2 | What is dollar convexity? | $C_\$ = P \times C = \frac{d^2P}{dy^2}$; the absolute second derivative |
| 3 | What is the sign of the convexity term in P&L approximation for vanilla bonds? | Always positive: $+ \frac{1}{2} C (\Delta y)^2$ |
| 4 | How does convexity error scale with shock size? | Like the square of the shock: $(\Delta y)^2$ |
| 5 | If duration is matched, does a Barbell or Bullet have higher convexity? | The Barbell (convexity scales with $T^2$) |
| 6 | What creates negative convexity? | Embedded options (Callables) or prepayments (MBS) |
| 7 | What does negative convexity imply for DV01 as rates rise? | DV01 **increases** as rates rise (extension risk) |
| 8 | What is the relationship between convexity and volatility? | Long Convexity ≈ Long Volatility |
| 9 | Why don't traders always hold maximum convexity? | Convexity usually costs yield (negative carry) |
| 10 | For a zero-coupon bond, how does convexity relate to maturity? | Roughly proportional to $T^2$: $C = \frac{T(T+0.5)}{(1+y/2)^2}$ |
| 11 | What is the typical "bump size" for numeric convexity? | Use small symmetric yield bumps; Tuckman illustrates 1bp increments. Choose bump size to balance numerical noise vs higher-order effects. |
| 12 | State the Taylor expansion for bond price change including convexity. | $\frac{\Delta P}{P} \approx -D\Delta y + \frac{1}{2}C(\Delta y)^2$ |
| 13 | What is Jensen's Inequality for a convex function? | $E[f(x)] > f(E[x])$ |
| 14 | How does convexity affect bond yields? | Higher convexity → Lower yields (convexity is valuable) |
| 15 | For a 10-year zero at 5% yield, approximate the convexity. | $C \approx \frac{10 \times 10.5}{(1.025)^2} \approx 100$ |
| 16 | What is "extension risk"? | Duration increases as rates rise in negative convexity instruments |
| 17 | Why is MBS hedging difficult? | Negative convexity forces pro-cyclical rebalancing |
| 18 | What does positive convexity mean for DV01? | DV01 **falls** as rates increase |
| 19 | In a stable rate environment, which outperforms: Barbell or Bullet? | Bullet (higher yield, no convexity gains realized) |
| 20 | What units does convexity have? | Time squared (years²) |
| 21 | When should you include convexity in risk calculations? | For moves > 50 bp, stress tests, VaR, and optioned instruments |
| 22 | What is effective convexity? | OAS-based convexity that accounts for changing option values |
| 23 | What conditions are needed for robust immunization? | Match both duration AND convexity: $D_A = D_L$ and $C_A \geq C_L$ |
| 24 | How is convexity P&L calculated for daily attribution? | Convexity P&L = Convexity01 $\times (\Delta y_{\text{bp}})^2 = \frac{1}{2} \times$ Dollar Convexity $\times (\Delta y)^2$ (consistent units) |
| 25 | What is the convexity effect on yield? | Approximately $-\frac{1}{2}C\sigma^2$; convexity lowers yields |

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

**4. Dollar Convexity and P&L**
A portfolio has dollar convexity (as defined in Section 13.2.2) of $80 billion. Rates move 75 bp.
(a) Calculate the convexity P&L.
(b) If the portfolio also has DV01 of $400,000, what is the total estimated P&L?

*Solution Sketch:*
(a) Convexity P&L = $\frac{1}{2} \times \$80bn \times (0.0075)^2 = \$2,250,000$ gain
(b) Duration P&L = $-\$400,000 \times 75 = -\$30,000,000$. Total ≈ $-\$27,750,000$

**5. Callable Bond Analysis**
You are hedging a Callable Bond using a 5-year Treasury. The callable has negative convexity at low yields.
(a) If rates rally 50 bps, will your hedge over- or under-compensate?
(b) Explain why in terms of DV01 behavior.

*Solution Sketch:*
(a) Under-compensate. The callable's price is capped while the Treasury continues to rise.
(b) As rates fall, the callable's DV01 *decreases* (approaching call price), while the Treasury's DV01 increases. The hedge ratio becomes wrong.

**6. Numerical Estimation**
Given: $P_{-1bp} = 101.05$, $P_0 = 101.00$, $P_{+1bp} = 100.96$
(a) Estimate convexity using the central difference formula.
(b) Is this bond positively or negatively convex?

*Solution Sketch:*
(a) $C = \frac{100.96 - 2(101.00) + 101.05}{101.00 \times (0.0001)^2} = \frac{0.01}{0.00000101} \approx 99$
(b) Positive—the second derivative is positive (price-yield curve is convex).

**7. Jensen's Inequality**
In a world where 1-year rates are expected to be 5% but could be 3% or 7% with equal probability:
(a) What is the expected value of a 1-year discount factor?
(b) What would the discount factor be at the expected rate?
(c) Which is larger? Explain using Jensen's Inequality.

*Solution Sketch:*
(a) $E[1/(1+r)] = 0.5/(1.03) + 0.5/(1.07) = 0.4854 + 0.4673 = 0.9527$
(b) $1/E[1+r] = 1/(1.05) = 0.9524$
(c) 0.9527 > 0.9524. Jensen's Inequality: $E[1/(1+r)] > 1/E[1+r]$ because $1/(1+r)$ is convex.

**8. Extension Risk Scenario**
An MBS portfolio has duration of 4 years at current rates. If rates rise 100 bp, duration extends to 6 years. You initially hedged with $100M of 4-year Treasuries.
(a) Are you over-hedged or under-hedged after the rate move?
(b) What must you do to restore the hedge?
(c) How does this action contribute to the "death spiral"?

*Solution Sketch:*
(a) Under-hedged. MBS duration increased to 6 years, but hedge is still sized for 4 years.
(b) Sell additional Treasuries (or add duration to hedge) to match the extended duration.
(c) Selling Treasuries in a rising-rate environment adds to selling pressure, pushing rates higher, which extends MBS duration further—the feedback loop.

**9. Barbell Breakeven**
A barbell yields 10bp less than an equivalent-duration bullet. The convexity difference is 150 (barbell minus bullet).
(a) Calculate the breakeven rate move for a 1-year holding period.
(b) If you expect rates to move less than this, which position should you take?

*Solution Sketch:*
(a) $\Delta y = \sqrt{\frac{2 \times 0.0010}{150}} = \sqrt{0.0000133} = 0.00365 = 36.5$ bp
(b) If expecting moves less than 36.5bp, take the bullet (higher yield, convexity gains won't offset).

**10. Immunization**
A pension fund has liabilities with duration 12 years and convexity 200. They can invest in Bond A (D=8, C=80) and Bond B (D=20, C=500). Find the portfolio weights that match both duration and convexity.

*Solution Sketch:*
Let $w$ be weight in Bond B. Duration match: $8(1-w) + 20w = 12$ → $w = 1/3$
Convexity: $80(2/3) + 500(1/3) = 53.3 + 166.7 = 220$
This exceeds 200, so the fund has excess convexity (desirable per Luenberger's $C_A \geq C_L$).

---
## References

- Tuckman & Serrat, *Fixed Income Securities: Tools for Today's Markets* (convexity definition, Taylor approximation, numerical estimation, barbell vs bullet, callable/negative convexity, mortgage hedging discussion).
- Hull, *Risk Management and Financial Institutions* (duration/convexity approximation, dollar convexity and portfolio aggregation, immunization with convexity).
- Hull, *Options, Futures, and Other Derivatives* (bond convexity and interest-rate Greeks connections).
- Luenberger, *Investment Science* (immunization and convexity considerations).
