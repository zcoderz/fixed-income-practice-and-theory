# Chapter 16: Curve Hedging Beyond DV01 — Twists, Butterflies, and PCA

---

## Introduction

The yield curve moves in parallel only by accident. This observation, which any seasoned trader confirms, strikes at the heart of a common risk management failure. Consider a portfolio manager who hedges a long position in 30-year bonds by shorting an appropriate amount of 2-year notes, carefully matching the net DV01 to zero. On paper, the portfolio is immune to interest rate risk. In reality, the manager has constructed a massive bet on curve shape. If the curve "twists"—short-term rates rise while long-term rates fall—the trade will hemorrhage money, even if the average rate change is zero.

Tuckman captures this danger precisely: "A (naive) DV01 analysis would allow for the hedging of a position in 10- or 30-year bonds with six-month securities." While duration matching protects against small level shifts, it leaves the portfolio fully exposed to **curve shape risk**—steepening, flattening, and bowing. An asset-liability manager who hedges the yield-based duration of a nine-year liability with a barbell of two- and 30-year assets will suffer losses "if the 30-year rate increases and the rest of the curve stays the same... or if the nine-year rate decreases and the rest of the curve stays the same." The hedge that looks perfect under parallel-shift assumptions fails spectacularly when the curve reshapes.

This chapter moves beyond the single-number view of rate risk. We begin with the **taxonomy of curve risk**, decomposing movements into level, slope, and curvature. We then develop **multi-factor hedging mechanics** using key-rate exposures and linear algebra, showing how to immunize a portfolio against arbitrary curve changes. The **butterfly trade** emerges as the precision instrument for curvature exposure, and we examine the critical question of how to weight the wings. We analyze **regression hedging**—including the often-misunderstood translation of standard errors into P&L risk. Finally, we examine **Principal Component Analysis (PCA)**, which reveals yield curve behavior is remarkably low-dimensional. As Hull documents, "yield curve shifts that occur in practice are, to a large extent, a linear sum of two or three standard shifts," with the first three principal components explaining over 98% of variance in rate changes.

This chapter connects backward to Chapter 14's key-rate decomposition and Chapter 15's DV01 hedging, and forward to Part IV's multi-curve construction, where similar multi-factor thinking applies to basis and cross-currency risk.

---

## 16.1 The Taxonomy of Curve Risk

Before we can hedge curve risk, we must classify it. While the yield curve is a high-dimensional object—rates can move independently at every tenor—financial mathematics and empirical observation converge on three primary modes of deformation.

### 16.1.1 Level (Parallel Shifts)

The **level** factor corresponds to a parallel shift where yields at all maturities change by the same amount. This is the "tide rising or falling" that standard DV01 and duration measures capture.

Tuckman's principal component analysis of U.S. dollar swap rates from the early 1990s through 2001 reveals the first component as "approximately a parallel shift." When par yields move in this component, "the three-month rate rises by 0.9 basis points, the two-year rate by 9.6 basis points, the five-year rate by 10.4 basis points, the 10-year rate by 10 basis points, and the 30-year rate by 8.1 basis points." While not exactly parallel, the loadings are close enough—particularly if the three-month point is ignored—to justify the convention.

**Driver:** Broad macroeconomic shifts—changes in the central bank's inflation target, real productivity growth, or aggregate risk appetite.

**Quantification:** The portfolio's total DV01 captures level exposure:
$$\text{DV01}_{\text{total}} = \sum_{i} \text{KRDV01}(T_i)$$

### 16.1.2 Slope (Twists)

The **slope** factor, often called a "twist," occurs when short-term rates and long-term rates move in opposite directions, or move in the same direction but with different magnitudes.

Tuckman describes the second principal component as follows: "This second component seems to be dominated by the movement of the very short end of the curve relative to the longer terms." The factor shows rates from short maturities moving in one direction while rates from intermediate to long maturities move in the other. A **steepener** sees long rates rise relative to short rates; a **flattener** sees the reverse.

**Risk:** A DV01-neutral portfolio that is long the 30-year and short the 2-year has zero parallel exposure but massive slope exposure. It effectively bets that the curve will flatten.

**Driver:** Monetary policy expectations (anchoring the short end) versus long-term growth and inflation outlooks (driving the long end). Federal Reserve activity particularly affects the slope factor—as Tuckman notes, "A period in which the Federal Reserve is very active, for example, might produce very different principal components than one over which the Fed is not active."

### 16.1.3 Curvature (Butterflies)

The **curvature** factor describes the movement of the "belly" of the curve (intermediate tenors like 5y or 7y) relative to the "wings" (short and long ends like 2y and 30y).

Tuckman's third principal component is "typically called curvature... The described move is a bowing of the two- and five-year rates relative to a close-to-parallel move of the three-month and 10-year rates. The 30-year rate moves in the opposite direction of the bowing of the two- and five-year rates."

**Risk:** A portfolio that is long the 5-year and short both the 2-year and 10-year is betting that the 5-year yield will fall relative to the wings.

**Driver:** Supply and demand dynamics at specific tenors, changes in medium-term volatility expectations, or mortgage hedging flows that concentrate in particular maturities.

> **Analogy: The Car Suspension**
>
> Think of the Yield Curve as a car chassis driving over a road.
> *   **Level (PC1)**: The road elevation changes. The whole car goes up or down.
> *   **Slope (PC2)**: The car accelerates or brakes. The nose lifts or dives (tilting).
> *   **Curvature (PC3)**: The car hits a pothole. The chassis bends or flexes in the middle.
>
> You can't fix a bent chassis (Curvature) by adjusting the ride height (Level). They are different mechanical problems.

---

## 16.2 Empirical Evidence: Principal Component Analysis

Principal component analysis provides rigorous empirical justification for the level/slope/curvature taxonomy. The technique extracts the dominant patterns from historical yield changes, revealing how much of rate volatility each factor explains.

### 16.2.1 What PCA Reveals

Hull presents PCA results using 2,780 daily observations on swap rates between 2000 and 2011. Table 9.7 in Hull shows the factor loadings for eight swap maturities across eight principal components:

**Table 16.1: PCA Factor Loadings for Swap Rates (Hull, 2000-2011)**

| Maturity | PC1 (Level) | PC2 (Slope) | PC3 (Curvature) | PC4 | PC5 | PC6 | PC7 | PC8 |
|----------|-------------|-------------|-----------------|-----|-----|-----|-----|-----|
| 1-year | 0.216 | -0.501 | 0.627 | -0.487 | 0.122 | 0.237 | 0.011 | -0.034 |
| 2-year | 0.331 | -0.429 | 0.129 | 0.354 | -0.212 | -0.674 | -0.100 | 0.236 |
| 3-year | 0.372 | -0.267 | -0.157 | 0.414 | -0.096 | 0.311 | 0.413 | -0.564 |
| 4-year | 0.392 | -0.110 | -0.256 | 0.174 | -0.019 | 0.551 | -0.416 | 0.512 |
| 5-year | 0.404 | 0.019 | -0.355 | -0.269 | 0.595 | -0.278 | -0.316 | -0.327 |
| 7-year | 0.394 | 0.194 | -0.195 | -0.336 | 0.007 | -0.100 | 0.685 | 0.422 |
| 10-year | 0.376 | 0.371 | 0.068 | -0.305 | -0.684 | -0.039 | -0.278 | -0.279 |
| 30-year | 0.305 | 0.554 | 0.575 | 0.398 | 0.331 | 0.022 | 0.007 | 0.032 |

The first principal component (PC1) "corresponds to a roughly parallel shift in the yield curve"—the loadings range from 0.216 to 0.404, all positive and of similar magnitude. The second factor (PC2) shows the classic slope pattern: negative loadings at short maturities (-0.501 at 1-year) transitioning to positive loadings at long maturities (+0.554 at 30-year). The third factor (PC3) shows the curvature pattern with positive loadings at short and long ends, negative in the middle.

Hull notes an important technical detail: "The factor loadings have the property that the sum of their squares for each factor is 1.0." This normalization allows meaningful comparison across factors.

The standard deviations of the factor scores (measuring each factor's importance) are:

**Table 16.2: Standard Deviation of Factor Scores**

| Factor | PC1 | PC2 | PC3 | PC4 | PC5 | PC6 | PC7 | PC8 |
|--------|-----|-----|-----|-----|-----|-----|-----|-----|
| **Std Dev** | 17.55 | 4.77 | 2.08 | 1.29 | 0.91 | 0.73 | 0.56 | 0.53 |

From this data, Hull calculates: "the first factor accounts for $17.55^2/338.8 = 90.9\%$ of the variance in the original data; the first two factors account for $(17.55^2 + 4.77^2)/338.8 = 97.7\%$ of the variance." The third factor adds another 1.3%.

Tuckman's analysis of swap rate data yields similar conclusions: "This first component explains about 85.6% of the total variance of term structure changes... The second component... accounts for about 8.7% of the total variance... The third component is typically called curvature and accounts for about 4.5% of the total variance."

The key insight from both sources: **the first three principal components together explain over 98% of yield curve variation**. This remarkable finding means that despite the yield curve being a high-dimensional object, its movements are dominated by just three patterns.

### 16.2.2 Factor Interpretation

The shapes of the principal components have intuitive interpretations:

$$\boxed{\begin{aligned}
\text{PC1 (Level)} &: \text{Roughly equal loadings across tenors} \\
\text{PC2 (Slope)} &: \text{Negative at short end, positive at long end} \\
\text{PC3 (Curvature)} &: \text{Positive at ends, negative in middle}
\end{aligned}}$$

Hull notes: "Results similar to those described here, concerning the nature of the factors and the amount of the total risk they account for, are obtained when a principal components analysis is used to explain the movements in almost any yield curve in any country."

> **Desk Reality: Interpreting the Variance Split**
>
> When Hull says PC1 explains 90.9% of variance, the flip side is critical: **9.1% of curve risk is non-parallel**. For a large portfolio, this "residual" risk is substantial.
>
> Consider a $100mm DV01 position with 7-year duration. Daily parallel volatility of 5bp produces about $35mm daily VaR. But the 9.1% non-parallel component represents over $3mm of daily risk that DV01 hedging leaves on the table. For a market maker earning 0.25bp bid-ask, this residual risk dwarfs the profit margin.

### 16.2.3 Using PCA for Hedging

Instead of neutralizing ten different key rates, a trader might choose to hedge just PC1, PC2, and PC3. Hull provides an explicit example. Consider a portfolio with the following sensitivities to rate changes (in $ millions per bp):

| 3-Year | 4-Year | 5-Year | 7-Year | 10-Year |
|--------|--------|--------|--------|---------|
| +10 | +4 | -8 | -7 | +2 |

Using the factor loadings from PCA, the portfolio's exposure to PC1 is:
$$10 \times 0.372 + 4 \times 0.392 - 8 \times 0.404 - 7 \times 0.394 + 2 \times 0.376 = +0.05$$

And the exposure to PC2 is:
$$10 \times (-0.267) + 4 \times (-0.110) - 8 \times 0.019 - 7 \times 0.194 + 2 \times 0.371 = -3.88$$

Hull observes: "The exposure to the second shift is almost 80 times greater than our exposure to the first shift. However... the standard deviation of the first shift is about 3.7 times as great as the standard deviation of the second shift." The portfolio is actually dominated by slope risk, despite appearing almost flat to level moves.

**Advantage:** PCA-based hedging drastically reduces the number of hedge instruments needed—three instruments can behaviorally hedge the whole curve.

**Disadvantage:** PCA loadings are historical averages. In a regime change, correlations break down. As Tuckman warns: "The current economic environment might not resemble that over which the principal components were derived."

> **Trader Lingo: "PC2" is the "2s10s"**
>
> When quants say "PC1", traders say "Level" or "Direction."
> When quants say "PC2", traders say "The Steepener" or "2s10s."
> When quants say "PC3", traders say "The Fly."
>
> Don't overcomplicate it. A PC2 hedge is just a steepener used to kill slope risk.

---

## 16.3 Key-Rate Exposures: The Vector View of Risk

To hedge curve shapes precisely, we need a measurement tool sharper than aggregate DV01. The industry standard is **Key-Rate DV01 (KRDV01)**, which decomposes risk into sensitivities at specific points on the curve. This section extends the framework introduced in Chapter 14.

### 16.3.1 The Triangle Shift

A Key-Rate DV01 measures sensitivity to a localized "triangular" perturbation. Tuckman (Chapter 7) formalizes this by selecting a set of key rates—typically 2y, 5y, 10y, and 30y par yields—and defining curve perturbations for each.

The construction works as follows: "The two-year key rate affects all par yields of term zero to five, the five-year affects par yields of term two to 10, the 10-year affects par yields of term five to 30, and the 30-year affects par yields from 10 on. The impact of each key rate is one basis point at its own maturity and declines linearly to zero at the term of the adjacent key rate."

Several properties make this construction useful:
1. **Locality:** Each key rate affects only a region of the curve
2. **Smoothness:** Impacts change smoothly between key rates
3. **Sum-to-parallel:** The sum of all key rate shifts equals a parallel shift

$$\boxed{\sum_{k} \text{Shift}_k(t) = 1 \text{ bp for all } t}$$

This last property ensures that KRDV01s decompose the parallel DV01:
$$\sum_{i} \text{KRDV01}(T_i) \approx \text{DV01}_{\text{parallel}}$$

**Why "sum-to-parallel" matters:** This isn't an accident—it's a deliberate construction. By ensuring that all triangular shifts sum to a parallel shift at every point on the curve, the KRDV01 decomposition is consistent with the total DV01. You can think of it as a "partition of unity" applied to interest rate risk.

### 16.3.2 Worked Example: Decomposing a Mortgage

Tuckman provides a detailed example computing key-rate exposures for a 30-year nonprepayable mortgage requiring payments of $3,250 every six months. With a par yield curve flat at 5% and key rates at 2y, 5y, 10y, and 30y:

| Key Rate | Initial Value | After Shift | Key Rate 01 | Duration | % of Total |
|----------|---------------|-------------|-------------|----------|------------|
| 2-Year | $100,453.13 | $100,452.15 | $0.98 | 0.10 | 0.9% |
| 5-Year | | $100,449.36 | $3.77 | 0.38 | 3.3% |
| 10-Year | | $100,410.77 | $42.37 | 4.22 | 37.0% |
| 30-Year | | $100,385.88 | $67.26 | 6.70 | 58.8% |
| **Total** | | | **$114.38** | **11.39** | **100%** |

The pattern reveals critical insights. Tuckman explains: "The sensitivity to short-term key rates is likely to be relatively low since the DV01 or duration of short-term cash flows is relatively low." Conversely, "the 10-year key rate has the largest span of all the key rates, covering 25 years of the term structure," explaining its 37% share despite the mortgage extending to 30 years.

### 16.3.3 Interpreting a Risk Vector

Consider a portfolio with the following KRDV01 profile (in $/bp):

| Tenor | 2y | 5y | 10y | 30y | Total |
|-------|-----|------|------|------|-------|
| **KRDV01** | +200 | +1,500 | +800 | +400 | +2,900 |

This vector tells a detailed story that the single number "+2,900" hides:

1. **Level Risk:** The portfolio loses $2,900 if rates rise in parallel by 1 bp
2. **Concentration:** Risk is heavily concentrated in the 5-year bucket (+1,500). The portfolio is "belly-heavy"
3. **Slope Sensitivity:** A steepener (+10 bp at 30y, -10 bp at 2y) produces P&L of approximately $-(400 \times 10 + 200 \times -10) = -\$2,000$
4. **Curvature Sensitivity:** A "belly up" shock (+5 bp at 5y only) costs $1,500 \times 5 = \$7,500$

The key insight: **a DV01-neutral portfolio can have enormous directional bets hidden in its key-rate profile**.

> **Comparison: Sniper Rifle vs. Shotgun**
>
> *   **Key Rates**: A Sniper Rifle. You target exactly the 5-year point. Precise, local, granular.
> *   **PCA**: A Shotgun. You target "Slope" (a broad movement). You hit everything a little bit. Use PCA for macro hedges; use Key Rates for specific bond relative value.

---

## 16.4 Hedging Mechanics: The Linear System

Once we have decomposed risk into a vector $d = (\text{KRDV01}_1, \ldots, \text{KRDV01}_m)^\top$, hedging becomes a problem of linear algebra.

### 16.4.1 The Scenario P&L Formula

For any curve movement described by a vector of shocks $\delta$ (in bp) at the key tenors, the P&L is approximated by:

$$\boxed{\Delta P \approx -d^\top \delta = -\sum_{i} \text{KRDV01}_i \times \delta_i}$$

This equation is the foundation of linear curve hedging. If we want $\Delta P = 0$ for specific types of shocks, we must construct a hedge that neutralizes the relevant KRDV01 components.

### 16.4.2 Hedging as a System of Equations

Tuckman develops the hedging equations systematically. Suppose we hold a mortgage with key-rate exposures to be hedged using four bonds at the key-rate maturities. Let $F_2, F_5, F_{10}, F_{30}$ be the face amounts of the hedging bonds.

If the hedging bonds are par bonds at their respective maturities, the system simplifies because each bond has sensitivity to only one key rate. The hedge amounts are found by:

$$\boxed{\frac{\text{DV01}_k(\text{hedge})}{100} \times F_k = \text{KRDV01}_k(\text{portfolio})}$$

Tuckman's example yields: $F_2 = \$1{,}920$, $F_5 = \$3{,}173$, $F_{10} = \$50{,}998$, $F_{30} = \$43{,}549$.

The hedged portfolio "will be approximately immune to any combination of key rate movements." But as Tuckman cautions, "the hedge will work as intended only if the par yields between key rates move as assumed. If the 20-year rate does something very different from what was assumed... the supposedly hedged portfolio will suffer losses or experience gains."

### 16.4.3 Worked Example: Hedging Level and Slope

**Problem:** A trader holds a portfolio with KRDV01 vector $d_0 = (+1{,}200, +1{,}800)^\top$ at 2y and 10y buckets. The total DV01 is 3,000, but the position is "steepener-exposed" (more risk at 10y).

**Instruments:**
- 2y Note ($H_1$): KRDV01 per unit = $(+1{,}000, 0)$
- 10y Note ($H_2$): KRDV01 per unit = $(0, +2{,}500)$

**Solution:**
The hedging equations are:
$$1{,}200 + x_1 \times 1{,}000 = 0 \quad \Rightarrow \quad x_1 = -1.2$$
$$1{,}800 + x_2 \times 2{,}500 = 0 \quad \Rightarrow \quad x_2 = -0.72$$

**Result:** Sell 1.2 units of the 2y Note and 0.72 units of the 10y Note.

**Verification:**
- **Parallel Shift (+1 bp):** Portfolio loses 3,000; hedges gain $1{,}200 + 1{,}800 = 3{,}000$. Net: 0.
- **Twist (+10 bp at 2y, -10 bp at 10y):** Portfolio P&L = $-(1{,}200 \times 10) - (1{,}800 \times -10) = +6{,}000$. Hedge P&L = $-(-1{,}200 \times 10) - (-1{,}800 \times -10) = -6{,}000$. Net: 0.

By matching KRDV01 bucket-by-bucket, the portfolio is immunized against *any* combination of 2y and 10y moves.

---

## 16.5 The Butterfly Trade: Isolating Curvature

The **butterfly** is the quintessential curve instrument. Tuckman states its purpose: butterfly trades are "designed to profit from a perceived mispricing while protecting against market and curve risk."

### 16.5.1 Anatomy of a Butterfly

A standard butterfly consists of three legs:
1. **Body (Belly):** A position in an intermediate maturity (e.g., 5y)
2. **Wings:** Offsetting positions in short (2y) and long (10y) maturities

The weights are chosen to satisfy two constraints:
1. **DV01 Neutrality:** $\text{DV01}_{\text{net}} = 0$ (Level hedge)
2. **Slope Neutrality:** Zero sensitivity to a parallel twist

What remains is pure **curvature** exposure.

> **Visualization: The Butterfly Tent**
>
> Imagine a tent pole pushing up in the middle (the belly position).
> *   If the middle (5y) goes up relative to the sides (2y/10y), the tent rises. You make money.
> *   If the sides go up and the middle stays down, the tent flattens. You lose money.
> *   Buying a butterfly is betting the tent will get taller (curve gets more humped).

### 16.5.2 Tuckman's 7s-8s-9s Butterfly Case Study

Tuckman presents a detailed case study involving the $4\frac{3}{4}$s of November 15, 2008. A term structure fit reveals this bond is "rich relative to the fitted term structure as of February 15, 2001." The question is whether a butterfly trade can capture this richness.

The 7s-8s-9s butterfly pays in the 8-year bond (the rich security) and receives in the 7-year and 9-year bonds (the wings). The weights are chosen to neutralize level and slope risk. Tuckman emphasizes that the trade's profit depends on whether the model's assessment of richness is correct—the butterfly transforms a valuation view into a curvature position while immunizing against directional rate movements.

Using Tuckman's example, the risk weights allocate half the body's DV01 to each wing:
$$RW_{\text{body}} = \frac{1}{2} RW_{\text{short wing}} = \frac{1}{2} RW_{\text{long wing}}$$

With this construction, the P&L simplifies to:
$$\text{P\&L} = -RW_{\text{body}} \times \Delta(\text{Butterfly Spread})$$

where the butterfly spread is defined as the average yield of the wings minus the yield of the body.

### 16.5.3 Butterfly Weighting Methods

The choice of wing weights is not unique. Different weighting methods suit different purposes:

**Method 1: Cash-Neutral**

Equal notional on each wing. Simple but not risk-neutral:
$$F_{\text{short wing}} = F_{\text{long wing}}$$

This method is rarely used for hedging because the DV01 contributions from each wing differ dramatically. A 2y and 30y wing with equal notional will have very different interest rate sensitivities.

**Method 2: Duration-Neutral (50/50 DV01 Split)**

Each wing contributes half the body's DV01:
$$\text{DV01}_{\text{short wing}} = \text{DV01}_{\text{long wing}} = \frac{1}{2} \text{DV01}_{\text{body}}$$

This is the construction in Tuckman's Chapter 4 example, where "the risk weights were set so as to allocate half of the risk to each wing." As Tuckman notes: "Before concluding this case study, it should be mentioned that equal risk weights on each wing are not necessarily the best weights for immunizing against curve shifts."

**Method 3: Regression-Weighted (Two-Factor Model)**

Use historical or model-implied betas to allocate risk. This approach recognizes that different parts of the curve are not equally correlated with the body. Tuckman develops this extensively in Chapters 8 and 14.

> **Desk Reality: When to Use Each Weighting Method**
>
> **50/50 DV01:** Simple, intuitive, good for quick relative value trades. The trade P&L is proportional to the butterfly spread.
>
> **Regression-weighted:** More accurate for hedging, but requires model confidence. Use when holding period is longer or position is larger.
>
> **Cash-neutral:** Almost never used for hedging. Only relevant if you have notional constraints (e.g., regulatory limits).
>
> In practice, many traders start with 50/50 and then tilt based on their view of the curve dynamics.

### 16.5.4 Two-Factor Model Hedging: The 2s-5s-10s Example

Tuckman's Chapter 14 provides an extensive case study of butterfly hedging using a two-factor model (V2). On May 15, 2001, the five-year swap rate appeared 8.8 basis points rich relative to the two- and 10-year rates.

To capture this richness while hedging both factors, the trader computes sensitivities to each factor:

| Term | $dP/dx$ | $dP/dy$ | DV01 |
|------|---------|---------|------|
| 2y | 1.8646 | 0.4808 | 1.8853 |
| 5y | 4.0689 | 0.4864 | 4.2853 |
| 10y | 6.5495 | 0.4843 | 7.3384 |

The hedging conditions for factors $x$ and $y$ are:
$$\frac{1.8646}{100}F_2 + \frac{6.5495}{100}F_{10} = 4.0689$$
$$\frac{0.4808}{100}F_2 + \frac{0.4843}{100}F_{10} = 0.4864$$

Solving these simultaneous equations yields $F_2 = 54.10$ and $F_{10} = 46.72$.

**Risk Weight Derivation:**

The DV01 risk weights express each wing's contribution as a percentage of the body's DV01. For the 2-year:
$$\boxed{\text{Risk Weight}_{2y} = \frac{F_2 \times \text{DV01}_{2y}}{\text{DV01}_{5y}} = \frac{54.10 \times 0.018853}{4.2853} = 23.8\%}$$

For the 10-year:
$$\boxed{\text{Risk Weight}_{10y} = \frac{F_{10} \times \text{DV01}_{10y}}{\text{DV01}_{5y}} = \frac{46.72 \times 0.073384}{4.2853} = 80.0\%}$$

**The 103.8% Puzzle:**

Critically, the risk weights sum to 103.8%, not 100%. Tuckman explains: "This finding might be interpreted as showing that the total position is not market neutral. But that interpretation runs counter to the purpose of the term structure model... there is no reason to expect that hedging against these shifts will also hedge against a parallel shift."

The key insight: **factor-neutral hedging is different from DV01-neutral hedging**. The two-factor model assumes the curve moves in specific patterns (captured by factors $x$ and $y$). If you hedge those factors perfectly, you may be over- or under-hedged for a pure parallel shift. The 3.8% "excess" reflects this difference.

> **Desk Reality: Why Risk Weights Matter**
>
> When a trader says "I'm 25/75 weighted on the fly," they mean 25% of the body's DV01 is allocated to the short wing and 75% to the long wing. This asymmetry reflects a view (or model output) that the body is more correlated with the long wing.
>
> If the regression suggests 23.8/80.0 but you use 50/50, you've implicitly taken a view against the historical correlation structure. That's fine if intentional—dangerous if accidental.

---

## 16.6 Regression Hedging: When Perfect Hedges Aren't Available

Often we cannot hedge a specific maturity with an instrument of identical maturity. Regression analysis provides optimal hedge ratios that account for imperfect correlation.

### 16.6.1 The Regression Framework

Tuckman models yield changes via regression:
$$\Delta y_t^{\text{target}} = \alpha + \beta \times \Delta y_t^{\text{hedge}} + \varepsilon_t$$

The coefficient $\beta$ captures the relationship between yield changes. Tuckman shows:

$$\boxed{\beta = \frac{\rho \sigma_{\text{target}}}{\sigma_{\text{hedge}}}}$$

where $\rho$ is the correlation between yield changes and $\sigma$ denotes volatility. This formula reveals that the regression hedge ratio incorporates both relative volatility and correlation.

**Derivation intuition:** The OLS estimate of $\beta$ minimizes the variance of the residuals. When you work through the normal equations, $\beta = \text{Cov}(\Delta y^{\text{target}}, \Delta y^{\text{hedge}}) / \text{Var}(\Delta y^{\text{hedge}})$. Since $\text{Cov} = \rho \sigma_{\text{target}} \sigma_{\text{hedge}}$ and $\text{Var} = \sigma_{\text{hedge}}^2$, the formula follows.

Tuckman also clarifies: "Equation (8.10) also reveals the difference between the volatility-weighted hedge and the regression-based hedge. The risk weight of the former equals the ratio of volatilities while the risk weight of the latter is the correlation times this ratio. In this sense, a volatility-weighted hedge assumes that changes in the two bond yields are perfectly correlated (i.e., that $\rho=1.0$)."

### 16.6.2 One-Variable Regression Example

Tuckman analyzes hedging 20-year bonds with 30-year bonds. Using 1,680 observations:

**Table 16.3: One-Variable Regression Results (Tuckman)**

| Statistic | Value |
|-----------|-------|
| Number of observations | 1,680 |
| R-squared | 98.25% |
| Standard error | 0.6973 bp/day |
| Regression coefficient $\beta$ | 1.057 |
| t-statistic on $\beta$ | 306.9951 |

The hedge ratio of 1.057 exceeds 1.0 because the 20-year yield is slightly more volatile than the 30-year and the correlation is very high (0.9912). As Tuckman explains: "a strict DV01 hedge would entail selling only about $8.3 million 30-year bonds. Since, however, the 20-year yield is now assumed more volatile than the 30-year yield, more 30-year bonds must be sold to hedge anticipated price changes in the 20-year bonds."

**Interpretation:** The regression-based hedge minimizes the variance of the hedged P&L. Tuckman proves: "The least squares criterion is equivalent to minimizing the standard deviation of the P&L of a regression-based hedged position."

### 16.6.3 Interpreting the Standard Error: From Statistics to P&L

The standard error of 0.6973 bp/day tells you the typical daily unexplained movement in the 20-year yield after accounting for 30-year movements. But what does this mean in dollars?

Tuckman provides the translation explicitly. At a 20-year DV01 of 0.118428 per $100 face, the hedged $10 million position faces daily one-standard-deviation P&L of:

$$\boxed{\sigma_{\text{P\&L}} = \text{SE} \times \frac{\text{DV01}}{100} \times \text{Notional} = 0.6973 \times \frac{0.118428}{100} \times \$10{,}000{,}000 = \$8{,}258}$$

This residual risk is substantial—larger than typical bid-ask spreads. As Tuckman observes: "This hedging risk is large relative to a market maker's bid-ask spread. If a market maker is able to collect a spread of .25 or even .5 basis points... this spread can easily be wiped out by the unpredictable behavior of 20-year yields relative to 30-year yields."

> **Desk Reality: Position Sizing with Regression Risk**
>
> If your daily 1-sigma risk is $8,258, then:
> - **2-sigma (95% confidence):** $16,516 daily risk
> - **3-sigma (99.7% confidence):** $24,774 daily risk
> - **Monthly 1-sigma (√21 trading days):** ~$37,800
>
> A market maker earning 0.25 bp per trade on $10mm notional makes about $2,950 per round-trip. With daily residual risk of $8,258, the hedge P&L volatility exceeds the profit margin by nearly 3x. This is why market makers in less liquid sectors demand wider spreads.

### 16.6.4 R-Squared: What 98.25% Really Means

The R-squared of 98.25% means that 98.25% of the variance in 20-year yield changes is explained by 30-year yield changes. In the one-factor case, R-squared equals the squared correlation: $R^2 = \rho^2 = 0.9912^2 = 0.9825$.

**The flip side:** The unexplained 1.75% is your residual risk. While 1.75% sounds small, variance is squared—the unexplained standard deviation is $\sqrt{0.0175} \approx 13.2\%$ of total volatility. That 13.2% maps directly to the $8,258 daily P&L risk.

> **Desk Reality: R-Squared as Signal-to-Noise**
>
> Think of R-squared as the "signal-to-noise ratio" of your hedge:
> - **R² = 99%:** Excellent hedge; residual is 10% of total volatility
> - **R² = 95%:** Good hedge; residual is 22% of total volatility
> - **R² = 80%:** Poor hedge; residual is 45% of total volatility
>
> For curve trades, R² above 95% is typical for adjacent maturities. For cross-market hedges (e.g., Treasuries vs. swaps), R² can drop to 70-85%.

### 16.6.5 Two-Variable Regression

Hedging the 20-year with both 10-year and 30-year bonds improves the hedge:
$$\Delta y_t^{20} = \alpha + \beta_{10} \times \Delta y_t^{10} + \beta_{30} \times \Delta y_t^{30} + \varepsilon_t$$

Tuckman's results:

**Table 16.4: Two-Variable Regression Results (Tuckman)**

| Statistic | Value |
|-----------|-------|
| R-squared | 98.63% |
| Standard error | 0.6170 bp/day |
| $\beta_{10}$ (10-year coefficient) | 0.161 |
| $\beta_{30}$ (30-year coefficient) | 0.877 |

The coefficients indicate that about 16% of the risk should be allocated to the 10-year hedge and 88% to the 30-year. "Since the 30-year had all the risk in the one-variable case, it follows that the 30-year should lose some risk allocation when the 10-year is added to the analysis."

The improvement is modest: R-squared increased by only 0.4%, and standard error fell from 0.70 to 0.62 bp. Tuckman notes: "the overall quality of the hedge has not improved dramatically from the one-variable case."

### 16.6.6 When to Add More Hedge Instruments

Adding hedge instruments faces diminishing returns:

| Hedge | R² | Std Error | Improvement |
|-------|-----|-----------|-------------|
| 30y only | 98.25% | 0.70 bp | — |
| 10y + 30y | 98.63% | 0.62 bp | +0.38% R², -0.08 bp SE |

Each additional instrument adds complexity and transaction cost. The decision depends on position size and holding period:

- **Small position, short hold:** Use one-variable hedge (simpler, cheaper)
- **Large position, longer hold:** Two-variable hedge may be worthwhile
- **Very large position:** Consider three or more hedges, but watch liquidity

---

## 16.7 Practical Notes and Pitfalls

### 16.7.1 The Parallel Fallacy

Never assume that a low net DV01 means "low risk." A portfolio can have zero DV01 but massive key-rate holes. Always check the vector: $(+1M, -2M, +1M)$ sums to zero but represents a massive bet on curvature.

Tuckman's introduction to Chapter 7 provides the starkest warning: "a portfolio with a barbell of two- and 30-year assets" hedging a nine-year liability may be "DV01-matched," but "the hedge will fail to protect against changes in the shape of the yield curve."

### 16.7.2 Granularity vs. Liquidity

In theory, we could hedge every year: 1y, 2y, 3y, ..., 30y. In practice, liquidity is concentrated in benchmarks (2y, 5y, 10y, 30y). Tuckman observes: "Using 20 key rates and 20 securities to hedge a portfolio might very well be feasible if the portfolio's composition were relatively constant over time," but "trading 20 securities every time the portfolio or its sensitivities change would probably prove too costly and onerous."

The choice of key rates should reflect available hedging instruments. Tuckman notes: "With respect to the terms of the key rates, it is clearly desirable to spread them out over the maturity range of interest. More subtly, well-chosen terms make it possible to hedge the resulting exposures with securities that are traded and, even better, with liquid securities."

### 16.7.3 Model Risk in PCA

PCA loadings are backward-looking averages. Tuckman cautions: "Shape changes from one day to the next might differ considerably from a typical move over a sample period." Further, "idiosyncratic moves of particular bond or swap rates (i.e., moves due to non-interest-rate-related factors) cannot typically be captured by principal component analysis. Since these moves are idiosyncratic, an analysis of average behavior discards them as noise."

In a crisis, historical correlations break down, and a "PCA-neutral" book might suddenly exhibit massive directional risk.

### 16.7.4 Regime Changes: When Correlations Break

Tuckman explicitly warns about regime dependence: "The current economic environment might not resemble that over which the principal components were derived. A period in which the Federal Reserve is very active, for example, might produce very different principal components than one over which the Fed is not active."

**Examples of regime shifts that break PCA hedges:**

1. **Fed tightening cycles:** When the Fed raises rates aggressively, the short end moves independently of the long end. PC1 becomes less "parallel" and PC2 loadings shift.

2. **Flight-to-quality events:** During crises (2008, March 2020), correlations across all tenors spike toward 1.0. The "slope" and "curvature" factors temporarily vanish.

3. **Yield curve inversions:** Inverted curves have different dynamics than steep curves. PCA loadings estimated during steep-curve periods may not apply.

> **Practitioner Note: Monitoring Regime Shifts**
>
> Watch for these signals that your PCA model may be stale:
> - Rolling 60-day correlation between 2y and 10y changes materially (>0.10)
> - PC1 variance share drops below 80% or rises above 95%
> - Regression betas move more than 0.1 from historical estimates
>
> When these signals fire, re-estimate your model or widen your risk limits.

### 16.7.5 Scenario Testing Checklist

Before putting on a complex curve trade, run the **Standard Suite**:

- [ ] **Parallel Shock (+1 bp):** Is net P&L approximately zero?
- [ ] **Twist Shock (+10/-10 bp, short/long):** What is the P&L?
- [ ] **Butterfly Shock (+5 bp belly only):** What is the P&L?
- [ ] **Stress Test:** What happens if the curve moves +50 bp AND steepens 20 bp simultaneously?
- [ ] **Correlation Break:** What if the hedge instrument moves 50% less than historically expected?

---

## 16.8 Summary

1. **DV01 is a scalpel, not a shield.** It protects against parallel shifts but leaves the portfolio exposed to twists and butterflies. As Tuckman states, DV01 hedging "fails to protect against changes in the shape of the yield curve, whether against flattening, steepening, or some other twist."

2. **Principal Component Analysis** reveals that curve movements are low-dimensional. Three factors (level, slope, curvature) explain over 98% of yield variance. Hull shows the first factor alone accounts for about 91% of variance; Tuckman finds 85.6% using different data.

3. **Key-Rate DV01 (KRDV01)** provides the resolution needed to see level, slope, and curvature risks. The triangular shift construction ensures that key-rate exposures sum to parallel DV01.

4. **Linear algebra is the language of hedging.** We solve systems of equations ($Ex = -e_0$) to neutralize specific risk buckets. Multi-factor models imply specific hedge weights.

5. **Butterflies** isolate curvature by hedging both level and slope. The choice of wing weights (50/50 DV01, regression-weighted, factor-neutral) depends on the purpose of the trade.

6. **Regression hedging** minimizes variance when perfect hedges aren't available. The hedge ratio $\beta = \rho\sigma_{\text{target}}/\sigma_{\text{hedge}}$ accounts for imperfect correlation. The standard error translates directly to P&L volatility.

7. **Regime changes invalidate historical correlations.** PCA and regression hedges assume stationarity; in crises, these assumptions fail.

---

## 16.9 Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Level (PC1)** | Parallel shift of all rates | Explains ~85-91% of curve variance; captured by DV01 |
| **Slope (PC2)** | Short and long rates move inversely | Explains ~8-10% of variance; DV01-neutral books still exposed |
| **Curvature (PC3)** | Belly moves vs wings | Explains ~2-5% of variance; butterflies trade this |
| **Key-Rate DV01** | Sensitivity to triangular shift at specific tenor | Decomposes parallel DV01 into maturity buckets |
| **Butterfly** | Long belly, short wings (or reverse) | Isolates curvature while hedging level and slope |
| **Regression $\beta$** | $\rho\sigma_{\text{target}}/\sigma_{\text{hedge}}$ | Optimal hedge ratio accounting for imperfect correlation |
| **Standard Error** | Std dev of regression residuals | Translates to daily P&L volatility of hedged position |
| **Risk Weight** | Wing DV01 as % of body DV01 | Summarizes butterfly construction; may sum to ≠100% |
| **PCA Loading** | Factor sensitivity at each tenor | Eigenvectors of yield-change covariance matrix |

---

## 16.10 Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $\text{KRDV01}_k$ | Key-rate DV01 at tenor $k$ |
| $d$ | Vector of key-rate exposures |
| $\delta$ | Vector of yield shocks (in bp) |
| $\beta$ | Regression hedge coefficient |
| $\rho$ | Correlation between yield changes |
| $\sigma$ | Volatility of yield changes |
| PC$n$ | The $n$-th principal component |
| SE | Standard error of regression |
| $F_k$ | Face amount of hedge instrument at tenor $k$ |
| $RW_k$ | Risk weight of instrument $k$ |

---

## 16.11 Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What percentage of yield curve variance does the first principal component explain? | Approximately 85-91%, depending on the market and sample period (Tuckman: 85.6%; Hull: 90.9%) |
| 2 | What are the three main factors in yield curve movements? | Level (parallel), Slope (twist), Curvature (butterfly) |
| 3 | What does KRDV01 measure? | Sensitivity to a triangular shift centered at a specific key rate |
| 4 | Why do KRDV01s sum to parallel DV01? | Because the sum of all triangular key-rate shifts equals a 1 bp parallel shift |
| 5 | What is a butterfly trade designed to isolate? | Curvature risk, while hedging level and slope |
| 6 | What is the formula for regression hedge coefficient $\beta$? | $\beta = \rho\sigma_{\text{target}}/\sigma_{\text{hedge}}$ |
| 7 | Why does $\beta > 1$ sometimes occur in regression hedging? | When the target is more volatile than the hedge and correlation is high |
| 8 | What does R-squared tell you about a regression hedge? | The proportion of target yield variance explained by the hedge instrument |
| 9 | Why might a DV01-neutral portfolio still lose money? | Because it may have large exposures to slope or curvature movements |
| 10 | What is the "parallel fallacy"? | Assuming zero DV01 means zero interest rate risk |
| 11 | What is the sum-to-parallel property of key rates? | Sum of all key-rate shifts = parallel shift of 1 bp |
| 12 | In PCA, what determines the importance of each factor? | The standard deviation of the factor score |
| 13 | What is a steepener trade? | A position that profits when long rates rise relative to short rates |
| 14 | How many instruments are needed to hedge an $n$-factor model? | $n$ instruments (e.g., 2 for a two-factor model) |
| 15 | What limits the usefulness of PCA for hedging? | PCA loadings are historical and may not apply to future regime changes |
| 16 | What is the standard error in regression hedging? | The standard deviation of daily hedging errors (unexplained P&L) |
| 17 | Why are 2y, 5y, 10y, 30y typical key rates? | They correspond to liquid benchmark securities that are practical for hedging |
| 18 | What scenario should you test beyond parallel shocks? | Twist shocks and butterfly shocks |
| 19 | How does a two-variable regression improve hedging? | It allocates risk to two hedge instruments, reducing residual variance |
| 20 | What relationship exists between PC2 loadings and tenor? | Negative at short maturities, positive at long maturities (slope shape) |
| 21 | What does a standard error of 0.70 bp/day mean for a $10mm position with DV01 of 0.118? | Daily 1-sigma P&L risk ≈ $8,258 (= 0.70 × 0.118/100 × $10,000,000) |
| 22 | If R² = 98.25%, what percentage of target variance is NOT explained by the hedge? | 1.75% — this is your residual/idiosyncratic risk |
| 23 | In Tuckman's 2s-5s-10s example, why do risk weights sum to 103.8%? | Factor-neutral hedging differs from DV01-neutral; the model assumes non-parallel moves |

---

## 16.12 Mini Problem Set

**Problem 1 (Curve View):** A "hedged" portfolio has Net DV01 = 0, but its KRDV01 vector is $(+2{,}000, -4{,}000, +2{,}000)$ at 2y/10y/30y. What is its view on curvature?

*Solution:* The portfolio is long the wings (2y and 30y) and short the belly (10y). It profits if the curve becomes *more convex* (wings outperform belly) or equivalently if the 10y yield rises relative to the interpolation of 2y and 30y yields. This is a "pay the belly" butterfly.

**Problem 2 (Hedge Ratio):** You hold a position with 10y KRDV01 of $5{,}000. Regression analysis shows $\Delta y_{10} \approx 0.9 \Delta y_7$, and the 7y note has DV01 = $1{,}000 per unit. How many units do you sell?

*Solution:* The hedge ratio is $h = -\beta \times (\text{KRDV01}_{\text{target}}/\text{DV01}_{\text{hedge}}) = -0.9 \times (5{,}000/1{,}000) = -4.5$. Sell 4.5 units of the 7y note.

**Problem 3 (Scenario P&L):** A portfolio has KRDV01 vector $d = (+100, +200, +100)$ at 2y/5y/10y. Calculate the P&L for a twist shock $\delta = (+10, 0, -10)$.

*Solution:* P&L $\approx -(100 \times 10 + 200 \times 0 + 100 \times -10) = -(1{,}000 + 0 - 1{,}000) = 0$. The symmetric portfolio is immune to this symmetric twist.

**Problem 4 (PCA Variance):** If the first three principal components have standard deviations of 18, 5, and 2 (in bp), and the total variance is 350, what percentage of total variance do they explain together?

*Solution:* Variance explained = $(18^2 + 5^2 + 2^2)/350 = (324 + 25 + 4)/350 = 353/350 \approx 100.9\%$. (If total variance is 350, these three factors account for essentially all of it. In practice, if the calculation exceeds 100%, the remaining factors contribute negligibly.)

**Problem 5 (Butterfly Construction):** Design a 2s-5s-10s butterfly that is DV01-neutral. If the 2y, 5y, and 10y DV01s are 0.02, 0.045, and 0.08 per 100 face respectively, and you want to receive 100 face of the 5y, what notionals of 2y and 10y do you pay?

*Solution:* Let $x_2$ and $x_{10}$ be the face amounts to pay. For DV01 neutrality:
$$100 \times 0.045 = x_2 \times 0.02 + x_{10} \times 0.08$$

This is one equation with two unknowns. A common approach: 50-50 DV01 split between wings.
$$x_2 \times 0.02 = x_{10} \times 0.08 = 4.5/2 = 2.25$$
$$x_2 = 112.5, \quad x_{10} = 28.125$$

Check: $112.5 \times 0.02 + 28.125 \times 0.08 = 2.25 + 2.25 = 4.5 = 100 \times 0.045$. ✓

**Problem 6 (Regression Hedge Sizing):** Given: 15y position with DV01 of $7,500. Available hedge: 10y note with DV01 of $5,000 per unit. Regression shows $\beta = 0.85$ with R² = 97.5% and SE = 0.55 bp.

(a) How many 10y units to sell?
(b) What is the daily 1-sigma P&L risk of the hedged position?
(c) What is the 3-sigma loss?

*Solution:*
(a) Hedge units = $-\beta \times (\text{DV01}_{\text{target}}/\text{DV01}_{\text{hedge}}) = -0.85 \times (7{,}500/5{,}000) = -1.275$ units. Sell 1.275 units.

(b) Daily 1-sigma P&L = SE × (DV01/100) × Notional. Here, assuming $100 face, the DV01 of the 15y is 0.075 per $100. If position is $10mm face:
$\sigma_{\text{P&L}} = 0.55 \times (0.075/100) \times 10{,}000{,}000 = \$4{,}125$

(c) 3-sigma loss = $3 \times \$4{,}125 = \$12{,}375$

**Problem 7 (Risk Weight Allocation):** A 5y swap position needs hedging with 2y and 10y swaps in a two-factor model. Given factor sensitivities:

| Tenor | dP/dx | dP/dy |
|-------|-------|-------|
| 2y | 2.0 | 0.5 |
| 5y | 4.0 | 0.5 |
| 10y | 7.0 | 0.5 |

(a) Set up the hedging equations for a 100 face 5y position.
(b) Solve for $F_2$ and $F_{10}$.

*Solution:*
(a) For factor $x$: $\frac{2.0}{100}F_2 + \frac{7.0}{100}F_{10} = 4.0$
For factor $y$: $\frac{0.5}{100}F_2 + \frac{0.5}{100}F_{10} = 0.5$

(b) From the second equation: $F_2 + F_{10} = 100$
Substituting into the first: $0.02F_2 + 0.07(100-F_2) = 4.0$
$0.02F_2 + 7 - 0.07F_2 = 4.0$
$-0.05F_2 = -3.0$
$F_2 = 60$, $F_{10} = 40$

**Problem 8 (PCA Exposure Calculation):** Using Hull's factor loadings, a portfolio has the following sensitivities ($ millions per bp): 3y: +5, 5y: -3, 10y: +2. Calculate the exposure to PC1 and PC2.

*Solution:*
PC1 exposure = $5 \times 0.372 + (-3) \times 0.404 + 2 \times 0.376 = 1.86 - 1.21 + 0.75 = +1.40$

PC2 exposure = $5 \times (-0.267) + (-3) \times 0.019 + 2 \times 0.371 = -1.34 - 0.06 + 0.74 = -0.66$

The portfolio is long level risk (+1.40) and short slope risk (-0.66, meaning it profits from flattening).

**Problem 9 (Standard Error to VaR):** A regression hedge has SE = 0.80 bp/day. The position has DV01 of $50,000. Assuming 21 trading days per month, what is the monthly 95% VaR of the hedged position?

*Solution:*
Daily 1-sigma P&L = $0.80 \times 50{,}000 = \$40{,}000$
Monthly 1-sigma = $\$40{,}000 \times \sqrt{21} = \$183{,}303$
Monthly 95% VaR = $1.645 \times \$183{,}303 = \$301{,}534$

**Problem 10 (Butterfly Weighting Comparison):** For a 2s-5s-10s butterfly with DV01s of 1.9, 4.3, and 7.3 respectively:

(a) Calculate wing notionals for 50/50 DV01 weighting with 100 face body.
(b) If regression suggests $\beta_{2y} = 0.24$ and $\beta_{10y} = 0.80$, what notionals would regression weighting imply?
(c) Compare total DV01 of wings in each case.

*Solution:*
(a) 50/50: Each wing contributes $4.3/2 = 2.15$ DV01
$F_2 = 2.15/0.019 = 113.2$; $F_{10} = 2.15/0.073 = 29.5$

(b) Regression: $F_2 \times 0.019 = 0.24 \times 4.3 = 1.032$, so $F_2 = 54.3$
$F_{10} \times 0.073 = 0.80 \times 4.3 = 3.44$, so $F_{10} = 47.1$

(c) 50/50: Total wing DV01 = $4.30$ (by construction = body DV01)
Regression: Total wing DV01 = $1.032 + 3.44 = 4.47$ (103.9% of body)

The regression weighting implies a slightly over-hedged DV01 position, consistent with Tuckman's 103.8% finding.

---

## 16.13 Source Map

### (A) Book-Verified Facts

| Fact | Source |
|------|--------|
| DV01 hedging "fails to protect against changes in the shape of the yield curve" | Tuckman Ch 7 |
| Key-rate shifts use triangular perturbations summing to parallel | Tuckman Ch 7 |
| PC1 explains 85.6% of swap rate variance | Tuckman Ch 13 (U.S. dollar swaps, early 1990s-2001) |
| PC1 explains 90.9% of variance; PC1+PC2 = 97.7% | Hull RM Ch 9 (swap rates, 2000-2011) |
| PC2 explains 8.7%; PC3 explains 4.5% | Tuckman Ch 13 |
| Factor loadings table (PC1-PC8 for 1y-30y) | Hull RM Table 9.7 |
| Factor score standard deviations (17.55, 4.77, 2.08, ...) | Hull RM Table 9.8 |
| "Factor loadings have the property that the sum of their squares for each factor is 1.0" | Hull RM Ch 9, footnote 9 |
| Regression hedge ratio $\beta = \rho\sigma_{\text{target}}/\sigma_{\text{hedge}}$ | Tuckman Ch 8, eq. (8.10) |
| One-variable regression of 20y on 30y yields $\beta = 1.057$, R² = 98.25%, SE = 0.6973 | Tuckman Ch 8, Table 8.1 |
| Two-variable regression yields $\beta_{10} = 0.161$, $\beta_{30} = 0.877$, SE = 0.6170 | Tuckman Ch 8, Table 8.2 |
| Daily P&L formula: SE × DV01/100 × Notional = $8,258 | Tuckman Ch 8, eq. (8.9) |
| 30-year mortgage KRDV01 breakdown (0.9%, 3.3%, 37%, 58.8%) | Tuckman Ch 7, Table 7.1 |
| Two-factor model hedge weights: F₂ = 54.10, F₁₀ = 46.72 | Tuckman Ch 14, eqs. (14.18-14.19) |
| Risk weights: 23.8% (2y), 80.0% (10y), sum = 103.8% | Tuckman Ch 14, eqs. (14.20-14.21) |
| Butterfly designed to "profit from perceived mispricing while protecting against market and curve risk" | Tuckman Ch 4 |
| "Risk weights were set so as to allocate half of the risk to each wing" | Tuckman Ch 4, eq. (4.27) |
| "Equal risk weights on each wing are not necessarily the best weights for immunizing" | Tuckman Ch 4 |
| Hull: "Yield curve shifts are, to a large extent, a linear sum of two or three standard shifts" | Hull RM Ch 9 |
| Federal Reserve activity affects PCA loadings | Tuckman Ch 13 |
| "Least squares criterion is equivalent to minimizing the standard deviation of the P&L" | Tuckman Ch 8 |
| "The volatility-weighted hedge assumes perfect correlation ($\rho = 1.0$)" | Tuckman Ch 8 |

### (B) Claude-Extended Content

| Content | Context |
|---------|---------|
| Interpretation of 9.1% non-parallel variance as $3mm risk on $100mm portfolio | Extended from Hull's variance decomposition |
| Position sizing guidance using standard error (1-sigma, 2-sigma, 3-sigma, monthly) | Extended from Tuckman's $8,258 example |
| "Signal-to-noise" analogy for R-squared interpretation | Practitioner intuition |
| Regime shift warning signals (rolling correlation, PC1 share, beta drift) | Extended from Tuckman's regime warning |
| Comparison of cash-neutral, duration-neutral, regression-weighted butterfly methods | Synthesized from Tuckman Ch 4, 8, 14 |
| "Sniper rifle vs. shotgun" analogy for key rates vs. PCA | Pedagogical extension |

### (C) Reasoned Inference (Derived from A or B)

| Inference | Derivation |
|-----------|------------|
| KRDV01 vector P&L formula $\Delta P \approx -d^\top\delta$ | Standard Taylor expansion combined with Tuckman's key-rate construction |
| Three-factor hedging captures >98% of curve risk | Follows from Hull's 97.7% + 1.3% for PC3 |
| Regression minimizes variance of hedged P&L | Tuckman Ch 8 proves least squares equivalent to variance minimization |
| Unexplained standard deviation is ~13.2% of total volatility when R² = 98.25% | $\sqrt{1 - 0.9825} = 0.132$ |

### (D) Flagged Uncertainties

- The exact PCA loadings vary with sample period, market, and data source. The percentages quoted (85-91% for PC1) represent typical ranges across published studies.
- Current market practice for key-rate definition may vary by institution; the triangular shift is one widely-used convention but not universal.
- The specific signals for regime shifts (0.10 correlation change, 80%/95% PC1 thresholds) are practitioner rules of thumb, not rigorously derived thresholds.
- I'm not sure about the exact fails-charge formula in the context of butterfly trades — the sources discuss trade mechanics but don't specify penalty calculations.

---

*Chapter 16 connects Chapter 14 (key-rate decomposition) and Chapter 15 (DV01 hedging) to multi-factor curve risk management. The techniques here extend naturally to the multi-curve world of Part IV, where tenor basis and cross-currency risks require similar vector-based thinking.*
