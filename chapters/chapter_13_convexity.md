# Chapter 13: Convexity — Second-Order P&L and When DV01 Breaks

---

## Introduction

Your risk system shows the portfolio is duration-hedged. DV01 is flat. Then the Fed surprises everyone—rates move 75 basis points in an afternoon. Your "hedged" position loses $3 million. How?

The answer lies in the one dimension that duration ignores: **curvature**. Duration captures the slope of the price-yield relationship, but bonds are not linear instruments. The price-yield curve bends, and this bending—called **convexity**—determines whether your hedge holds in a crisis or falls apart precisely when you need it most.

For small moves, the linear duration approximation often dominates. For larger shocks (or very large notionals), the second-order term becomes visible in P&L. For instruments with embedded options (callables, MBS), cashflows can change with rates and convexity can become negative, making DV01-only hedges unstable.

## Learning Objectives
- Translate a yield change into an approximate price/PV change using duration and convexity, with explicit units.
- Define convexity, dollar convexity, and Convexity01 and reconcile them to a risk report.
- Explain why DV01 hedges break for large moves and for negative-convexity instruments.
- Compare a duration-matched barbell and bullet and interpret the trade as “carry vs convexity”.

Prerequisites: [Chapter 11 — DV01/PV01](chapters/chapter_11_dv01_pv01_definitions_computation.md), [Chapter 12 — Duration](chapters/chapter_12_duration_macaulay_modified_dv01.md) (optional: [Chapter 5 — Bonds](chapters/chapter_05_bonds.md)).  
Follow-on: [Chapter 14 — Key-Rate DV01](chapters/chapter_14_key_rate_dv01_bucket_exposures.md), [Chapter 15 — DV01 Hedging](chapters/chapter_15_dv01_hedging.md).

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

A practical implication of **positive convexity** is that DV01 tends to **fall** as yields rise (and **rise** as yields fall). Equivalently, the price-yield curve becomes less steep after a sell-off and more steep after a rally.

> **Analogy: The Shock Absorber**
>
> Convexity is the shock absorber on a car.
> - **Rates Rise (Bad News)**: Price falls, but convexity *slows* the fall (DV01 decreases). The shock is dampened.
> - **Rates Fall (Good News)**: Price rises, and convexity *accelerates* the rise (DV01 increases). The gain is amplified.
> - **Result**: Positive convexity helps you win more when you're right, and lose less when you're wrong.

### 13.1.2 Taylor Expansion: The Mathematical Foundation

To quantify this, let $P(y)$ be the price given yield $y$. For a shock $\Delta y$, we can approximate the price change using a second-order Taylor series expansion around the current yield. Duration captures the first-order effect, and convexity captures the second-order term:

$$P(y + \Delta y) \approx P(y) + \frac{dP}{dy}\Delta y + \frac{1}{2}\frac{d^2P}{dy^2}(\Delta y)^2$$

Dividing by price $P$ gives us the standard percentage change decomposition used by practitioners:

$$\boxed{\frac{\Delta P}{P} \approx -D\,\Delta y + \frac{1}{2}Cvx\,(\Delta y)^2}$$

Here, $D = -\frac{1}{P}\frac{dP}{dy}$ is modified duration (linear risk), and $Cvx = \frac{1}{P}\frac{d^2P}{dy^2}$ is convexity (curvature risk).

Since the term $(\Delta y)^2$ is always positive regardless of the direction of the move, the convexity term $\frac{1}{2}Cvx(\Delta y)^2$ is positive when $Cvx>0$. It boosts gains in rallies and reduces losses in sell-offs, quantifying the “curvature cushion.”

---

## 13.2 Defining Convexity Precisely

### 13.2.1 The Normalized Second Derivative

Modified duration measures the relative slope of the price-yield curve at a given point. Convexity is the relative curvature at that point. Specifically, convexity is the value of \(C\) defined as:

$$\boxed{C=\frac{1}{P}\frac{\mathrm{d}^2 P}{\mathrm{d}y^2}}$$

Equivalently, \(C = \frac{1}{P}\frac{\mathrm{d}^2P}{\mathrm{d}y^2}\). In this chapter we denote this normalized convexity by \(Cvx\).

### 13.2.2 Dollar Convexity: The Gamma Analog

The dollar convexity of a bond, \(C_{\$}\), is defined analogously to dollar duration as the product of convexity and the value of the bond. This means that:

$$\boxed{C_{\$}=P \times Cvx = \frac{d^2P}{dy^2}}$$

**Why Dollar Convexity Matters:** Even when a system does not report it directly, dollar convexity is a natural object for P&L attribution because the second-order (convexity) contribution for a yield move \(\Delta y\) can be written as:

$$\boxed{\text{Convexity P\&L} = \frac{1}{2}\,C_{\$}\,(\Delta y)^2}$$

If you want to plug in a yield move in **basis points** (instead of decimal yield units), define:

- $\Delta y_{\text{bp}}$: change in yield in basis points
- $\Delta y = 0.0001 \times \Delta y_{\text{bp}}$

Then the convexity P&L can be written as:

$$\boxed{\text{Convexity P\&L}=\underbrace{\left[\frac{1}{2}\times C_{\$}\times (0.0001)^2\right]}_{\text{Convexity01 } (\$/\text{bp}^2)} \times (\Delta y_{\text{bp}})^2}$$

This is often the most desk-friendly way to compute convexity P&L because it takes the move in bps and makes the quadratic scaling explicit.

> **Pitfall — bp vs decimal (and the missing \(1/2\)):** It is easy to plug \(\Delta y\) in **bp** into a formula that expects **decimal yield**, or to forget the \(1/2\) in the second-order term.
> **Why it matters:** The convexity contribution scales as \((\Delta y)^2\), so a units mistake becomes a 10×–10,000× P&L error.
> **Quick check:** Recompute using \(\Delta y = 10^{-4}\Delta y_{\text{bp}}\) and verify the second-order term is \(\tfrac{1}{2}P\cdot Cvx\cdot (\Delta y)^2\).

> **Desk Reality:** Risk reports may show **normalized convexity** (\(Cvx\)), **dollar convexity** (\(C_{\$}\)), or a pre-scaled **Convexity01** in \(\$/\text{bp}^2\).
> **Common break:** Mixing \(\Delta y\) in bp with formulas written in decimal yield, or mixing \(Cvx\) vs \(C_{\$}\) vs Convexity01, creates order-of-magnitude errors.
> **What to check:** Take one position, compute \(PV(y\pm 1\text{bp})\) and \(PV(y\pm \Delta y)\) in a spreadsheet, and reconcile to the system’s DV01 and convexity/Convexity01 outputs.

**Conventions used in this chapter (risk definitions):**
- **Bump object:** the yield input \(y\) used in the bond pricing function \(P(y)\) (examples use annual yield with semiannual compounding). For “modified” convexity, **cashflows are held fixed** when yields move.
- **Bump size:** \(1\text{ bp} = 10^{-4}\) in yield units.
- **DV01 (sign + units):** \(DV01 := PV(y-1\text{bp})-PV(y)\). For a long non-callable bond, \(DV01>0\). Units: currency per 1bp (often per 100 price points and then scaled by notional).
- **Second-order P&L (consistent units):** \(\Delta PV \approx -DV01\cdot \Delta y_{\text{bp}} + Convexity01\cdot (\Delta y_{\text{bp}})^2\).

**Check (turn \(Cvx\) into a desk-usable \(Convexity01\)):** If your system reports normalized convexity \(Cvx\) (per yield\(^2\)) and you know the position PV \(V\) (in dollars), then position dollar convexity is \(C_{\$}=V\cdot Cvx\) and \(Convexity01=\tfrac{1}{2}C_{\$}(10^{-4})^2\) in \(\$/\text{bp}^2\). Toy scale: if \(V=\$100\text{mm}\) and \(Cvx=50\), then \(C_{\$}=5{,}000\text{mm}\) and \(Convexity01\approx \tfrac{1}{2}\times 5{,}000\text{mm}\times 10^{-8}=\$25/\text{bp}^2\). A 50bp move has \((\Delta y_{\text{bp}})^2=2{,}500\), so convexity P&L is about \(25\times 2{,}500=\$62{,}500\) (directionally helpful for long positive-convexity).

### 13.2.3 Computing Convexity Numerically

In practice, analytic formulas are not always available for complex instruments. In such cases, convexity is estimated using the same "bump and reprice" logic used for DV01. Instead of measuring a single slope, we estimate the *change* in slope using a **central difference method**:

$$\boxed{Cvx \approx \frac{P_- - 2P_0 + P_+}{P_0 \times (\Delta y)^2}}$$

where:
- $P_-$ is the price at $y - \Delta y$
- $P_0$ is the current price
- $P_+$ is the price at $y + \Delta y$

Second derivatives are numerically delicate because this is a “difference of differences.” In practice, choose the bump size to balance numerical noise (too small) and higher-order contamination (too large); the right choice depends on the instrument and your curve representation.

### 13.2.4 Worked Example A: Finite-Difference Convexity from Repricing

One common methodology is:

**Step 1: Estimate the first derivative at 4.995%** (between 4.99% and 5.00%):
$$\frac{dP}{dy}\bigg|_{4.995\%} = \frac{100.0000 - 100.0780}{0.05 - 0.0499} = -779.83$$

**Step 2: Estimate the first derivative at 5.005%** (between 5.00% and 5.01%):
$$\frac{dP}{dy}\bigg|_{5.005\%} = \frac{99.9221 - 100.0000}{0.0501 - 0.05} = -779.09$$

**Step 3: Estimate the second derivative at 5.00%**:
$$\frac{d^2P}{dy^2} = \frac{-779.09 - (-779.83)}{0.05005 - 0.04995} = \frac{0.7363}{0.0001} = 7{,}363$$

**Step 4: Compute convexity**:
$$Cvx = \frac{7{,}363}{100} = \mathbf{73.63}$$

**Sanity Check:** The convexity is positive (as expected for a vanilla bond), and the magnitude (73.6) is plausible for a bond with roughly 8-year duration. ✓

---

## 13.3 Convexity for Standard Bonds

### 13.3.1 Zero-Coupon Bond Convexity

For a zero-coupon bond, convexity can be derived analytically, offering clear intuition for how it behaves across maturities. Given the price of a zero-coupon bond $P = 100 / (1 + y/2)^{2T}$ under semiannual compounding, differentiating twice and dividing by $P$ yields:

$$\boxed{Cvx_{\text{zero}} = \frac{T(T + 0.5)}{(1 + y/2)^2}}$$

This formula reveals a fundamental rule of thumb for fixed income traders: **Convexity scales with the square of maturity ($T^2$).**

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

> **Desk Reality:** Convexity exposure concentrates in longer maturities because (for option-free zeros) \(Cvx\) grows roughly like \(T^2\).
> **Common break:** Assuming two duration-matched hedges are “equivalent” without checking their convexity (the residual shows up only in big moves).
> **What to check:** Compare both DV01 *and* \(Cvx\) (or Convexity01) of each leg on the same bump object before calling a trade “hedged.”

### 13.3.3 Coupon Bond Convexity

For coupon bonds, yield-based convexity (under semiannual compounding) can be written as:

$$Cvx = \frac{1}{P(1+y/2)^2}\left[\sum_{t=1}^{2T} \frac{t}{2} \cdot \frac{t+1}{2} \cdot \frac{c/2}{(1+y/2)^{t}} + T(T+0.5) \cdot \frac{100}{(1+y/2)^{2T}}\right]$$

Since a coupon bond is a portfolio of zeros (one for each cash flow), its convexity equals the value-weighted average of the individual zero convexities. Longer-maturity coupon bonds generally have greater convexity than shorter-maturity coupon bonds.

### 13.3.4 Analytical vs. Numerical Check

We can verify the numerical approximation method against the analytic formula. Consider a 5-year zero-coupon bond at a 5% yield.

**Analytic Result**: Using the formula:
$$Cvx = \frac{5 \times 5.5}{(1.025)^2} = \frac{27.5}{1.050625} \approx \mathbf{26.17}$$

**Finite Difference**: Bumping yields by 1 bp ($P_-$ at 4.99%, $P_+$ at 5.01%):
$$\frac{78.0818 - 2(78.1198) + 78.1579}{78.1198 \times (0.0001)^2} \approx \mathbf{26.17}$$

The results match perfectly, confirming that for standard instruments, the "bump and reprice" methodology is robust.

---

## 13.4 When DV01 Breaks: The Case for Convexity

When is duration "good enough," and when is it dangerous? The answer lies in the magnitude of the market move. The error of a duration-only model does not scale linearly; it scales with the *square* of the interest rate shock:

$$\boxed{\text{Duration-only error (fractional)} \approx \frac{1}{2}\,Cvx\,(\Delta y)^2}$$

Including the quadratic term makes the approximation dramatically more accurate for large moves.

### 13.4.1 Worked Example C: A 100 Basis Point Shock

**Example Title**: DV01-only vs DV01+convexity for a 3y fixed-rate bond

**Context**
- You hold a plain, non-callable bond. You want a quick estimate of P&L for a large yield move, and you want to reconcile it to a risk report (\(DV01\), Convexity01).

**Timeline (Make Dates Concrete)**
- Valuation date: 2026-02-16 (assume settlement same day; price on coupon date so \(AI\approx 0\))
- Coupon dates: 2026-08-16, 2027-02-16, 2027-08-16, 2028-02-16, 2028-08-16, 2029-02-16 (maturity)
- Simplification: ignore business-day adjustment; assume each coupon period has accrual \(\tau=0.5\)

**Inputs**
- Notional: \(N=\$100{,}000{,}000\)
- Coupon: 6% annual, paid semiannually (cashflow = \(100\times 0.06/2 = 3.00\) per \$100 each period)
- Yield quote: \(y=5\%\) (annual yield with semiannual compounding)
- Shock: \(\Delta y = +100\text{bp} = +0.01\)

**Outputs (What You Produce)**
- Price per \$100: \(P(y)=102.754\)
- Modified duration: \(D=2.725\)
- Convexity: \(Cvx=9.08\)
- \(DV01 := PV(y-1\text{bp})-PV(y)\) for this bump object

**Step-by-step**
1. **PV / price:** compute \(P(y)=\sum_{t} CF_t/(1+y/2)^t\).
2. **DV01 (1bp yield bump):** reprice at \(y-1\text{bp}\) and take the difference.
3. **Convexity (central difference):** reprice at \(y\pm 1\text{bp}\) and apply \(Cvx\approx (P_- -2P_0 + P_+)/[P_0(\Delta y)^2]\).
4. **P&L estimate:** use \(\Delta P \approx -P D \Delta y + \tfrac{1}{2} P Cvx (\Delta y)^2\), then scale by notional.

**Cashflows (per \$100 notional)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-08-16 | 3.00 | coupon |
| 2027-02-16 | 3.00 | coupon |
| 2027-08-16 | 3.00 | coupon |
| 2028-02-16 | 3.00 | coupon |
| 2028-08-16 | 3.00 | coupon |
| 2029-02-16 | 103.00 | coupon + principal |

**P&L / Risk Interpretation**
- **Duration-only price change:** \(\Delta P_{dur}\approx -102.754\times 2.725\times 0.01 = -2.800\) (price points per \$100).
- **Convexity correction:** \(\Delta P_{cvx}\approx \tfrac{1}{2}\times 102.754\times 9.08\times 0.01^2 = +0.0466\).
- **Duration + convexity:** \(\Delta P\approx -2.800 + 0.0466 = -2.753\).
- **Actual repricing:** at \(y=6\%\) this bond is at par, so \(\Delta P_{actual}=100.000-102.754=-2.754\).

Scaling to notional \(N\):
- \(\Delta PV_{dur} \approx (-2.800/100)\times 100{,}000{,}000 = -\$2.800\text{mm}\)
- \(\Delta PV_{cvx} \approx (+0.0466/100)\times 100{,}000{,}000 = +\$46.6\text{k}\)

**Check (reconcile to DV01 and \(Convexity01\)):** From the duration estimate, the position DV01 is approximately \(DV01\approx \$2.800\text{mm}/100 \approx \$28{,}000/\text{bp}\) (because the duration-only term is \(-DV01\times 100\)). From the convexity estimate, \(Convexity01\approx \$46.6\text{k}/100^2\approx \$4.66/\text{bp}^2\). Plugging these into \(\Delta PV\approx -DV01\cdot \Delta y_{\text{bp}} + Convexity01\cdot (\Delta y_{\text{bp}})^2\) reproduces the same \(-\$2.80\text{mm} + \$46.6\text{k}\) decomposition.

**What breaks in practice:** if the bond is callable/MBS-like, cashflows change with rates and you need **effective** (option-adjusted) duration/convexity rather than the “hold cashflows fixed” measures.

**Sanity Checks**
- Units: \(DV01\) is \(\$/\text{bp}\); Convexity01 is \(\$/\text{bp}^2\); the convexity term scales with \((\Delta y_{bp})^2\).
- Sign: for \(Cvx>0\), the convexity contribution is positive in both rallies and sell-offs.
- Repricing check: bump and reprice at the shocked yield to validate the approximation.

### 13.4.2 Scaling of Errors

The danger arises because doubling the shock size quadruples the error:

| Shock Size | Duration Error (price points) | Scaling Factor |
| :--- | :--- | :--- |
| **25 bp** | -0.003 | 1× |
| **50 bp** | -0.012 | 4× |
| **100 bp** | -0.046 | 16× |
| **200 bp** | -0.184 | 64× |

The key point is the **quadratic scaling**: doubling the shock size quadruples the convexity correction.

For small daily moves, convexity P&L is often small relative to duration P&L. For stress-sized moves, ignoring convexity can materially misstate risk.

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
> A 50bp move on \$100mm with convexity of 100: Convexity P\&L = $\frac{1}{2} \times 100 \times 0.005^2 \times \$100\text{mm} = \$125,000$. If this isn't in your explain, you have a \$125k hole.

---

## 13.5 Convexity and Hedging

### 13.5.1 DV01 Hedges with Convexity Mismatch

Traders often construct "DV01 Neutral" portfolios and believe they are fully hedged. However, if the long and short legs of the trade have significantly different convexities, the portfolio is only hedged for *small* moves. This is known as a **convexity mismatch**.

Intuition: DV01-neutral hedges the **first derivative** (slope). If the trade has non-zero net convexity, the **second derivative** remains, and the position will tend to make or lose money based on the *size* of the move, not just its direction.

### 13.5.2 Worked Example D: The "Self-Funding" Trade

Imagine a portfolio that is DV01 neutral but "Long Convexity":
- **Long**: 1 unit of a 10-year par bond (\(Cvx=73.6\), DV01=0.0779)
- **Short**: 4.14 units of a 2-year par bond (\(Cvx=4.5\), DV01=0.0188)

The position is delta-neutral:
$$\text{Net DV01} = 0.0779 - (4.144 \times 0.0188) \approx 0$$

However, the position is heavily gamma-positive:
$$\text{Net }Cvx = 73.6 - (4.144 \times 4.5) = +54.9$$

Because of this positive net convexity, the portfolio acts like a long straddle option. If rates move **100 bps** in *either* direction, the trader profits:

1. **Rates RALLY (-100 bps)**: The long 10y position gains +$8.18, while the short 2y position loses -$7.89. **Net P&L: +$0.29**.
2. **Rates SELL OFF (+100 bps)**: The long 10y position loses -$7.44, while the short 2y position gains +$7.70. **Net P&L: +$0.26**.

In both scenarios, the portfolio makes money. This profit comes from the second-order term $\frac{1}{2} Cvx (\Delta y)^2$, which is always positive for a net positive \(Cvx\).

Market-neutral hedge funds actively manage this mismatch, strictly deciding whether to be long or short convexity based on their volatility expectations.

> **Trader Talk: “Gamma” language**
>
> By analogy to options, desks often describe convexity risk using “gamma” language.
> - **Long Gamma**: You own convexity (like a straddle). You want the market to move—high volatility. You make money on the wiggles.
> - **Short Gamma**: You sold convexity. You want the market to sleep—low volatility. You bleed money if the market moves.
>
> The key practical point: DV01 neutrality does not eliminate convexity P&L in large moves.

### 13.5.3 Immunization with Convexity Matching

Duration matching is a *local* hedge: it protects against small (parallel) yield shifts around the current level. A more robust second-order immunization also matches convexity.

One common way to state the second-order conditions (in addition to matching present value at today’s curve) is:

$$\boxed{\begin{aligned}
D_{\text{assets}} &= D_{\text{liabilities}} \\
Cvx_{\text{assets}} &\geq Cvx_{\text{liabilities}}
\end{aligned}}$$

**The Duration-Only Trap:** A duration-matched hedge is only locally immunized. Large rate moves break the hedge because duration itself changes.

**Convexity Enhancement:** By matching both duration AND convexity, the approximation holds over a wider range of yield moves.

**Implementation:** Generally, at least three bonds are required to match present value, duration, and convexity simultaneously.

> **Desk Reality: Pension Fund ALM**
>
> Long-dated liability books are highly sensitive to the long end of the curve. A duration-only hedge can look fine for small moves but drift in larger moves if asset and liability convexities differ.

---

## 13.6 Positive vs Negative Convexity

So far, we have assumed convexity is positive (the "smile"). However, for instruments with embedded options—specifically **Callable Bonds** and **Mortgage-Backed Securities (MBS)**—convexity can turn **negative** (a "frown").

> **Visualization: The Smile and The Frown**
>
> - **Positive Convexity**: A "Smiley Face" curve. Price rises faster than expected in rallies, falls slower in sell-offs.
> - **Negative Convexity**: A "Frowning Face" curve. Price rises *slower* than expected in rallies (upside capped), falls faster in sell-offs (downside accelerated).

### 13.6.1 The Mechanics of Negative Convexity

In callable bonds, as yields fall, the issuer is likely to call the bond to refinance at a lower rate. This caps the price appreciation at the call price (e.g., 100). Graphically, the price-yield curve flattens out and then bends downwards, creating a concave shape.

Similarly, in MBS, as rates fall, homeowners refinance their mortgages, causing the bond to return principal precisely when it would otherwise be most valuable.

### 13.6.2 Worked Example E: The Callable Trap

Consider a 5% Callable Bond (callable at par in 1 year) compared to a vanilla non-callable bond. As yields drop, the difference becomes stark:

| Yield Level | Vanilla Price | Callable Price | Effective Convexity (Callable) |
| :--- | :--- | :--- | :--- |
| **6% (OTM)** | 92.56 | 91.87 | **-120** (Negative) |
| **5% (ATM)** | 100.00 | 96.95 | **-223** (Deeply Negative) |
| **4% (ITM)** | 108.18 | 100.03 | **-147** (Negative) |

These numbers are illustrative; actual convexity depends on the call schedule and the option model/volatility assumptions used for pricing.

At a 5% yield, the (effective) convexity is **-223**. This negative number has profound hedging implications. If rates drop 100 bps to 4%, the vanilla bond gains 8.18, but the callable bond gains only 3.08 because it is capped at par. If you hedged this callable bond with a standard DV01 ratio, you would massively underperform the hedge.

Mixing positive- and negative-convexity instruments in one hedge can be unstable unless you explicitly account for how cashflows (and therefore DV01) change with the yield level.

### 13.6.3 Extension Risk: The Destabilizing Dynamic

**Deep Insight**: Negative convexity implies that **DV01 increases as yields rise**.

- **Positive Convexity**: Yields $\uparrow$ $\implies$ Price $\downarrow$ but Duration $\downarrow$ (Self-stabilizing).
- **Negative Convexity**: Yields $\uparrow$ $\implies$ Price $\downarrow$ and Duration $\uparrow$ (Destabilizing).

This phenomenon is known as "extension risk."

**Check (how to see extension risk in your own numbers):** Compute DV01 at two yield levels using the *same* model and bump design (e.g., \(DV01(y)\) and \(DV01(y+50\text{bp})\)). For a negative-convexity instrument, it is common to see \(DV01(y+50\text{bp})>DV01(y)\) (risk “extends” in sell-offs). If you are running a DV01 hedge, this means a sell-off can make you under-hedged unless you rebalance.

It explains why MBS hedging is difficult: as rates move, the duration (and DV01) of the underlying instrument can change materially, which means yesterday's hedge ratio can be wrong today. When many participants must rebalance in the same direction, this can amplify volatility.

### 13.6.4 The MBS Convexity "Death Spiral"

The extension risk mechanism in MBS can create a dangerous feedback loop when many market participants are positioned similarly:

**Anatomy of a Sell-off:**
1. **Rates rise** → MBS duration extends (prepayments slow as refinancing becomes unattractive)
2. **Portfolio managers become under-hedged** → Their hedge was sized for shorter duration
3. **Must sell Treasuries/futures to add duration hedge** → Adds to selling pressure
4. **Selling pressure pushes rates higher** → Returns to step 1
5. **Cycle repeats** → Pro-cyclical feedback loop

### 13.6.5 Effective Convexity (Model-Based)

When computing convexity by “bump and reprice,” the bumped prices can be obtained either by assuming the expected cashflows **do not change** when yields change, or by allowing expected cashflows **to change** when yields change.

- **Standard / modified (yield-based) convexity:** cashflows held fixed as yields move.
- **Effective convexity:** cashflows are allowed to change with yields (the same distinction as for duration).

**Definition (effective convexity):** compute prices at shifted yields using an option model (often implemented in an OAS framework), then apply the same central-difference formula:

$$\boxed{Cvx_{\text{effective}} = \frac{P_- - 2P_0 + P_+}{P_0 \times (\Delta y)^2}}$$

For option-free bonds, either convexity measure will be positive and typically similar. For bonds with embedded options, effective convexity can be negative even when yield-based convexity is positive.

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

A classic convexity trade is the **barbell vs bullet**: two portfolios with the same duration (same first-order risk) but different convexity (different second-order risk).

- **Bullet**: A portfolio concentrated at a single maturity (e.g., 9-year zero).
- **Barbell**: A portfolio split between short (e.g., 2-year) and long (e.g., 30-year) maturities.

You can weight them to have the **exact same duration**. So why choose one over the other? **Convexity.**

### 13.7.1 Worked Example F: Barbell vs Bullet (Duration-Matched)

Assume a flat yield curve at 5%:

**Bullet Portfolio**: A 9-year zero-coupon bond.
- Duration: 9 years
- Convexity (\(Cvx\)): $\frac{9 \times 9.5}{(1.025)^2} = 81.38$

**Barbell Portfolio**: 75% in 2-year zeros and 25% in 30-year zeros.
- Duration: $0.75 \times 2 + 0.25 \times 30 = 9$ years (matches bullet)
- Convexity (\(Cvx\)): $0.75 \times \frac{2 \times 2.5}{(1.025)^2} + 0.25 \times \frac{30 \times 30.5}{(1.025)^2} = 221.30$

The barbell has convexity of **221.30** vs the bullet's **81.38**—nearly three times as much. The reason is structural: duration scales roughly linearly with maturity, while zero-bond convexity scales roughly with \(T^2\).

### 13.7.2 The Cost of Convexity: Negative Carry

If the barbell has the same duration but higher convexity, why isn’t it strictly better?

The yield curve is typically concave (humped) or upward sloping. To buy the Barbell, you must sell the intermediate 9-year bond (which usually yields more) and buy 2-year and 30-year bonds (which, on average, yield less in a concave curve environment).

This price you pay—giving up yield to own potential non-linear gains—is the **cost of convexity**:

- **Long Volatility**: The Barbell outperforms if rates move significantly (due to convexity).
- **Short Volatility**: The Bullet outperforms if rates stay stable (due to higher yield/carry).
In practice, the higher-carry structure tends to win in calm markets, while the higher-convexity structure tends to win when rates move a lot.

This is often framed as a volatility trade: you give up some carry to own a payoff profile that benefits from large moves (via convexity).

### 13.7.3 Quantifying the Breakeven

How much must rates move for the barbell to break even? We can calculate this explicitly.

**Setup:**
- Barbell convexity advantage: $\Delta Cvx = 221.30 - 81.38 = 139.92$
- Yield give-up (illustrative): 8 bp/year
- Holding period: 1 year

**Breakeven Calculation:**

The convexity gain must offset the yield loss:
$$\frac{1}{2} \Delta Cvx \times (\Delta y)^2 = \text{Yield Give-up}$$

Solving for $\Delta y$:
$$\Delta y = \sqrt{\frac{2 \times 0.0008}{139.92}} = \sqrt{0.0000114} \approx 0.0034 = 34 \text{ bp}$$

**Interpretation:** If rates move more than 34bp in either direction over the year, the barbell wins. If rates move less, the bullet wins.

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

This principle applies to any portfolio construction: spreading cash flows increases convexity; concentrating them reduces it.

---

## 13.8 Convexity as Volatility Exposure

Why does convexity behave like volatility? The answer lies in **Jensen's Inequality**: for a convex function $f$, $E[f(x)] \geq f(E[x])$.

If you own a convex bond, the average price after a random volatility event is *higher* than the price at the average yield.

### 13.8.1 Jensen's Inequality and Bond Pricing

The pricing function of a one-period discount factor, $1/(1+r)$, is convex in $r$ (for $r>-1$). Jensen's inequality then implies:

$$E\left[\frac{1}{1+r}\right] > \frac{1}{E[1+r]} = \frac{1}{1+E[r]}$$

This means that in a world with interest rate uncertainty, bond prices are *higher* than they would be if rates were certain to equal their expected value. Equivalently, yields are *lower* than the “no-uncertainty” rate—this difference is sometimes called the convexity value.

Numerical illustration (from a stylized example): even if the one-year rate is 10% and the expected one-year rate in one year is 10%, the two-year spot rate can be 9.982% (a 1.8bp “convexity” effect).

### 13.8.2 Worked Example G: Jensen's Inequality Numerical

Consider a two-state world where rates are 5% today. In one year, rates will be either 3% or 7% with equal probability.

**Step 1: Expected price of a 1-year zero at $t=1$:**
$$E[P] = 0.5 \times \frac{100}{1.03} + 0.5 \times \frac{100}{1.07} = 0.5 \times 97.087 + 0.5 \times 93.458 = 95.272$$

**Step 2: Price at the expected rate:**
$$P(E[r]) = \frac{100}{1.05} = 95.238$$

**Step 3: Convexity value:**
$$\text{Convexity Value} = 95.272 - 95.238 = 0.034 \text{ (3.4 cents per 100 face)}$$

**Check (translate cents to dollars):** 3.4 cents per 100 is 0.034 price points. On \(N=\$100\text{mm}\) face, 1.00 price point is \(\$1{,}000{,}000\), so 0.034 points is about \(\$34{,}000\). The convexity value in this toy example is small, but it scales with both notional and rate volatility.

**Volatility Scaling:** Now double the volatility—rates are either 2% or 8%:
$$E[P] = 0.5 \times \frac{100}{1.02} + 0.5 \times \frac{100}{1.08} = 0.5 \times 98.039 + 0.5 \times 92.593 = 95.316$$
$$\text{Convexity Value} = 95.316 - 95.238 = 0.078 \text{ (7.8 cents)}$$

The convexity value increases when the dispersion of future rates increases.

### 13.8.3 Convexity Value Increases with Volatility

For small yield volatility, a common rule-of-thumb approximation is:

$$\boxed{\text{Convexity effect on yield} \approx -\frac{1}{2}\,Cvx\,\sigma^2}$$

where $\sigma$ is the yield volatility. This formula has profound implications:

- **Long Convexity**: You are long gamma. You want realized volatility > implied volatility.
- **Short Convexity**: You are short gamma. You want realized volatility < implied volatility.

The key takeaway is qualitative: the value of convexity increases with volatility. In this sense, a long-convexity position is long volatility, and a short-convexity position is short volatility.

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
- **Practical approach**: Start with small symmetric bumps (e.g., 1bp), then sanity-check stability by varying the bump size.

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

1. **Safety Cushion**: With positive convexity, the second-order term \(\tfrac{1}{2}Cvx(\Delta y)^2\) in \(\Delta P/P\) is positive, so a duration-only linear estimate is “too pessimistic” in sell-offs and “too conservative” in rallies.

2. **$T^2$ Scaling (Zeros)**: For option-free zeros, convexity grows roughly with the square of maturity. Long maturities can have orders-of-magnitude more convexity than short maturities.

3. **Error Correction**: The convexity correction scales with $(\Delta y)^2$, so it grows quickly in stress-sized moves.

4. **Dollar Convexity (Gamma Analog)**: Dollar convexity is \(C_{\$} = P \times Cvx\). Some systems report this, while others report a pre-scaled \(Convexity01\) in \(\$/\text{bp}^2\); always verify units.

5. **Negative Convexity**: Embedded options can create negative convexity (“frown”), meaning DV01 can extend in sell-offs and hedges can become unstable if you ignore cashflow changes.

6. **Extension Risk and the Death Spiral**: MBS negative convexity can create pro-cyclical hedging flows that amplify volatility in stressed markets.

7. **Barbell vs Bullet**: Spreading cash flows (barbell) increases convexity; concentrating them (bullet) reduces it. The barbell wins in volatile markets; the bullet wins in stable markets. The breakeven can be calculated explicitly.

8. **Volatility Trade**: Long convexity is long volatility (qualitatively). A common small-vol rule-of-thumb is a yield “convexity effect” of about $-\frac{1}{2}Cvx\sigma^2$ (units must be consistent).

9. **Immunization**: Duration matching alone is insufficient for large moves; robust immunization typically targets PV, duration, and convexity together.

---

## Key Concepts

| Concept | Definition | Why It Matters |
| :--- | :--- | :--- |
| **Convexity (\(Cvx\))** | $\frac{1}{P}\frac{d^2P}{dy^2}$ | Corrects linear duration errors for large rate moves |
| **Dollar Convexity (\(C_{\$}\))** | $P \times Cvx = \frac{d^2P}{dy^2}$ | Currency-scaled curvature; check bump object + units |
| **Convexity01** | $\frac{1}{2}C_{\$}(10^{-4})^2$ | \(\$/\text{bp}^2\) coefficient used with $(\Delta y_{bp})^2$ |
| **Positive Convexity** | Curvature "smile" | You gain more on rallies and lose less on sell-offs |
| **Negative Convexity** | Curvature "frown" | Found in Callables/MBS; price capped on rallies |
| **$T^2$ Scaling** | $Cvx_{\text{zero}} \approx T^2$ | Long-dated option-free zeros have much more convexity |
| **Effective Convexity** | Model-based numerical convexity | Required for option-embedded instruments |
| **Barbell** | Portfolio of Short + Long bonds | Higher convexity than a Bullet; long volatility |
| **Convexity Bias** | $\frac{1}{2}Cvx(\Delta y)^2$ | The P&L "cushion" provided by curvature |
| **Jensen's Inequality** | $E[f(x)] > f(E[x])$ for convex $f$ | Why convexity lowers yields in a volatile world |
| **Extension Risk** | Duration rises as rates rise | Negative convexity creates pro-cyclical hedging |
| **Immunization** | Match duration AND convexity | Required for robust ALM against large moves |

---

## Notation

| Symbol | Meaning | Units / Convention |
| :--- | :--- | :--- |
| \(y\) | Yield input in \(P(y)\) | Annualized yield; compounding frequency stated (examples: semiannual) |
| \(P(y)\) | Bond price as a function of \(y\) | Price per \$100 notional unless stated |
| \(D\) | Modified duration | Years; \(D=-\frac{1}{P}\frac{dP}{dy}\) for the chapter’s bump object \(y\) |
| \(Cvx\) | Convexity | Years\(^2\); \(Cvx=\frac{1}{P}\frac{d^2P}{dy^2}\) |
| \(C_{\$}\) | Dollar convexity | Currency per yield\(^2\); \(C_{\$}=P\cdot Cvx=\frac{d^2P}{dy^2}\) |
| \(\Delta y\) | Yield shock | Decimal yield (e.g., 0.01 = 100 bp) |
| \(\Delta y_{\text{bp}}\) | Yield shock in bp | \(\Delta y=10^{-4}\Delta y_{\text{bp}}\) |
| \(DV01\) | 1bp PV sensitivity | \(DV01:=PV(y-1\text{bp})-PV(y)\); currency per 1bp |
| Convexity01 | 2nd-order coefficient | \(\$/\text{bp}^2\); \(Convexity01=\tfrac{1}{2}C_{\$}(10^{-4})^2\) |
| \(P_-,P_0,P_+\) | Bumped/base prices | \(P_\pm=P(y\pm \Delta y)\), \(P_0=P(y)\) |
| \(Cvx_{\text{effective}}\) | Effective convexity | Model-based repricing; cashflows may change with rates |
| \(\sigma\) | Yield volatility | Annualized standard deviation; use consistent yield units |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Define convexity mathematically. | $Cvx = \frac{1}{P}\frac{d^2P}{dy^2}$ |
| 2 | What is dollar convexity? | $C_{\$} = P \times Cvx = \frac{d^2P}{dy^2}$ |
| 3 | What is the sign of the convexity term in the 2nd-order P&L approximation for positive convexity? | Always positive: $+ \frac{1}{2} Cvx (\Delta y)^2$ |
| 4 | How does convexity error scale with shock size? | Like the square of the shock: $(\Delta y)^2$ |
| 5 | If duration is matched, does a Barbell or Bullet have higher convexity? | The Barbell (convexity scales with $T^2$) |
| 6 | What creates negative convexity? | Embedded options (Callables) or prepayments (MBS) |
| 7 | What does negative convexity imply for DV01 as rates rise? | DV01 **increases** as rates rise (extension risk) |
| 8 | What is the relationship between convexity and volatility? | Long Convexity ≈ Long Volatility |
| 9 | Why don't traders always hold maximum convexity? | Convexity usually costs yield (negative carry) |
| 10 | For a zero-coupon bond, how does convexity relate to maturity? | Roughly proportional to $T^2$: $Cvx_{zero} = \frac{T(T+0.5)}{(1+y/2)^2}$ |
| 11 | What is a reasonable bump size for numeric convexity? | Use small symmetric yield bumps (e.g., 1bp) and sanity-check stability by varying the bump size. |
| 12 | State the Taylor expansion for bond price change including convexity. | $\frac{\Delta P}{P} \approx -D\Delta y + \frac{1}{2}Cvx(\Delta y)^2$ |
| 13 | What is Jensen's Inequality for a convex function? | $E[f(X)] \ge f(E[X])$ (strict if $X$ is non-degenerate and $f$ is strictly convex). |
| 14 | How does convexity affect bond yields? | Higher convexity → Lower yields (convexity is valuable) |
| 15 | For a 10-year zero at 5% yield, approximate the convexity. | $Cvx \approx \frac{10 \times 10.5}{(1.025)^2} \approx 100$ |
| 16 | What is "extension risk"? | Duration increases as rates rise in negative convexity instruments |
| 17 | Why is MBS hedging difficult? | Negative convexity forces pro-cyclical rebalancing |
| 18 | What does positive convexity mean for DV01? | DV01 **falls** as rates increase |
| 19 | In a stable rate environment, which outperforms: Barbell or Bullet? | Bullet (higher yield, no convexity gains realized) |
| 20 | What units does convexity have? | Time squared (years²) |
| 21 | When should you include convexity in risk calculations? | For moves > 50 bp, stress tests, VaR, and optioned instruments |
| 22 | What is effective convexity? | Model-based convexity that allows cashflows to change with rates (often implemented in an OAS framework). |
| 23 | What conditions are needed for robust immunization? | Match PV and duration, and also match convexity (often \(Cvx_A \ge Cvx_L\)) for larger moves. |
| 24 | How is convexity P&L calculated for daily attribution? | Convexity P&L $\approx$ Convexity01 $\times (\Delta y_{\text{bp}})^2 = \frac{1}{2}C_{\$}(\Delta y)^2$ (consistent units). |
| 25 | What is the convexity effect on yield? | A common small-vol approximation is $-\frac{1}{2}Cvx\sigma^2$ (consistent units). |

---

## Mini Problem Set

1. (Compute) A bond has modified duration \(D=5\) and convexity \(Cvx=30\). Estimate the percentage price change for:
   (a) \(\Delta y=+50\) bp using duration only
   (b) \(\Delta y=+50\) bp using duration + convexity
   (c) \(\Delta y=+200\) bp using duration + convexity
2. (Compute) If a 10-year zero has convexity \(Cvx_{10}\approx 100\) at the current yield, use \(Cvx\propto T^2\) to estimate \(Cvx_5\), \(Cvx_{20}\), and \(Cvx_{30}\).
3. (Compute) Construct a duration-matched barbell using 2-year and 30-year zeros to match a 10-year zero (flat curve assumption).
   (a) What weights \(w_{2}, w_{30}\) match duration?
   (b) Which portfolio has higher convexity? Explain in one sentence.
4. (Compute) A portfolio has dollar convexity \(C_{\$}=80{,}000{,}000{,}000\) (currency per yield\(^2\)) and \(DV01=\$400{,}000\) (currency per bp). Rates sell off by \(+75\) bp. Estimate duration P&L, convexity P&L, and total.
5. (Desk) You hedge a callable bond with a Treasury of similar DV01. If rates rally 50 bp, do you expect the hedge to over- or under-hedge? Explain using how the callable’s DV01 changes as it goes “into the money”.
6. (Compute) Numerical convexity estimate. Given \(P_{-10\text{bp}}=101.05\), \(P_0=101.00\), \(P_{+10\text{bp}}=100.96\), estimate \(Cvx\) using the central difference formula with \(\Delta y=10\) bp. Is the convexity positive or negative?
7. (Concept) Jensen's inequality check. Show that \(E[1/(1+r)] > 1/(1+E[r])\) when \(r\) is random and \(r>-1\), and interpret this in bond-pricing terms.
8. (Desk) Extension risk. An MBS has duration 4 years today but extends to 6 years after a +100 bp move. You originally hedged with Treasuries sized to 4-year duration. After the sell-off, are you over- or under-hedged, and what trading action tends to follow?
9. (Compute) Barbell breakeven. A barbell gives up 10 bp/year in yield versus an equivalent-duration bullet, but has \(\Delta Cvx=150\) more convexity. Using the second-order approximation, estimate the one-year breakeven absolute rate move \(|\Delta y|\).
10. (Compute/Concept) Two bonds have \((D, Cvx)\): Bond A \((8, 80)\), Bond B \((20, 500)\). Liabilities have duration 12 and convexity 200. Find the weight \(w\) in Bond B that matches duration. What convexity does the resulting asset portfolio have, and what does that imply for large shocks?

### Solution Sketches (Selected)

1. \(\Delta P/P\approx -D\Delta y + \tfrac{1}{2}Cvx(\Delta y)^2\).
   - (a) \(-5\times 0.005=-2.50\%\)
   - (b) \(-2.50\% + \tfrac{1}{2}\times 30\times 0.005^2=-2.4625\%\) (about \(-2.46\%\))
   - (c) \(-5\times 0.02 + \tfrac{1}{2}\times 30\times 0.02^2=-9.40\%\)
4. Use \(\Delta y_{\text{bp}}=+75\), \(\Delta y=0.0075\).
   - Duration P&L \(\approx -DV01\cdot \Delta y_{\text{bp}}=-(400{,}000)\times 75=-\$30.0\text{mm}\).
   - Convexity P&L \(\approx +\tfrac{1}{2}C_{\$}(\Delta y)^2=\tfrac{1}{2}\times 80\text{bn}\times 0.0075^2=+\$2.25\text{mm}\).
   - Total \(\approx -\$27.75\text{mm}\).
6. With \(\Delta y=10\text{bp}=0.001\),
   \[
   Cvx\approx \frac{101.05-2(101.00)+100.96}{101.00\times (0.001)^2}
   =\frac{0.01}{0.000101}\approx 99.
   \]
   Positive convexity because the numerator is positive.
5. Typically under-hedged on a rally: as rates fall, the callable’s price approaches the call region and its DV01 tends to drop, while the Treasury’s DV01 rises, so a static DV01 hedge ratio becomes too small.
8. Under-hedged after the sell-off (duration extended). Restoring the hedge typically means adding duration hedges (e.g., selling more Treasuries / paying fixed), which can add to selling pressure if many do it at once.

---
## References

- Tuckman & Serrat, *Fixed Income Securities: Tools for Today's Markets*, “Estimating price changes and returns with DV01, duration, and convexity”; “Yield-based convexity”; “Expectations” (Jensen/convexity value intuition).
- Hull, *Risk Management and Financial Institutions*, “Dollar Convexity” (dollar convexity definition; portfolio aggregation; convexity in P&L approximations).
- Hull, *Options, Futures, and Other Derivatives*, “Proof of the convexity adjustment formula” (Taylor expansion framing).
- Luenberger, *Investment Science*, “Convexity” (normalized convexity definition; immunization with convexity).
- Elton et al., *Modern Portfolio Theory and Investment Analysis*, “Convexity” (quadratic correction; callable bonds and curvature changes).
- Neftci, *Principles of Financial Engineering*, “Prepayment Options” (prepayment as embedded option; negative convexity intuition).
- *Simulation and Optimization in Finance*, “Standard vs Effective Convexity” (modified vs effective convexity; effective convexity can be negative for optioned instruments).
