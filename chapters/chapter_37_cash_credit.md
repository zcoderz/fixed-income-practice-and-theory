# Chapter 37: Cash Credit — Risky Bonds, Credit Spreads, and CS01

---

## Introduction

With survival probabilities and hazard rates now in hand from Chapter 36, we can turn to the central question of credit investing: *how do you price and risk-manage a bond that might default?*

A corporate bond is not merely a stream of promised cash flows discounted at a risk-free rate. Each coupon payment and the principal redemption are contingent on the issuer's survival. If default occurs before maturity, the bondholder receives not par but some recovery value—typically a fraction of face value determined through bankruptcy proceedings or an auction process. The *credit spread* that a bond trades at over risk-free benchmarks compensates investors for bearing this default risk, as well as the mark-to-market volatility, liquidity risk, and other premia that Chapter 8 introduced.

This chapter bridges two worlds: the survival probability framework of Chapter 36 and the spread taxonomy of Chapter 8. We develop the **risky discounting** approach that prices bonds using survival curves, connect this to observable credit spreads, and define the **Credit DV01** (or **CS01**)—the price sensitivity to credit spread changes. The key insight, emphasized by O'Kane, is that "a floating rate note is predominantly a credit play. The interest rate sensitivity is small. However, the credit risk remains and is comparable to that of a fixed rate bond of a similar maturity."

We cover:

1. **Risky bond pricing** — how survival probabilities enter the valuation of coupon-bearing bonds
2. **The credit spread connection** — relating spread levels to default intensity and recovery
3. **Credit DV01 and spread duration** — measuring and hedging credit risk
4. **Interest rate hedging** — isolating the credit play by neutralizing rate risk
5. **The CDS-bond basis** — why bond spreads and CDS spreads diverge (preview of Part IX)

Understanding these mechanics is essential for anyone managing credit portfolios, constructing basis trades, or decomposing P&L into rates versus credit components.

---

## 37.1 Risky Bond Pricing: The Survival-Weighted Framework

### 37.1.1 The Zero-Recovery Zero-Coupon Bond

The simplest credit-risky instrument is a zero-coupon bond that pays nothing upon default. O'Kane develops this from first principles in his treatment of reduced-form models. Consider "a credit risky zero coupon bond with a face value of $1 which is due to mature at time $T$. Assume also that we know that if there is a default, the recovered amount will be zero." The present value is the discounted expectation of the payoff at maturity:

$$\hat{Z}(0, T) = \mathbb{E}\left[\exp\left(-\int_0^T r(s) ds\right) \mathbf{1}_{\tau > T}\right]$$

where $\tau$ is the random default time. The indicator function captures survival: it equals 1 if the credit survives to $T$, zero otherwise.

When interest rates and hazard rates are independent—a standard assumption justified by O'Kane because "the correlation between interest rates and the default intensity process has only a small effect on the pricing of a risky zero coupon bond"—this expectation factors into a product:

$$\boxed{\hat{Z}(0,T) = Z(0,T) \cdot Q(0,T)}$$

where:
- $\hat{Z}(0,T)$ is the price of the risky zero-coupon bond (zero recovery)
- $Z(0,T)$ is the risk-free discount factor (e.g., from the LIBOR/OIS curve)
- $Q(0,T)$ is the survival probability to time $T$

O'Kane states this result directly: "As a result we can write the price of the zero recovery zero coupon bond as the product of the risk-free zero coupon bond price and the survival probability."

**Intuition:** The risky bond pays $1 at maturity only if the issuer survives. Its value is the risk-free present value of $1, multiplied by the probability of receiving that payment. This is the credit analog of the interest rate discount factor—survival probability "discounts" for the possibility the credit won't survive to pay.

When hazard rates are deterministic and constant at $\lambda$, we have $Q(0,T) = e^{-\lambda T}$, giving:

$$\hat{Z}(0,T) = e^{-rT} \cdot e^{-\lambda T} = e^{-(r+\lambda)T}$$

O'Kane confirms: "If we also assume a constant risk-free short rate, i.e. $r(t) = r$, the price of a risky zero recovery zero coupon bond is given by $\hat{Z}(0, T) = \exp(-(r + \lambda)T)$."

This reveals that **default intensity acts like an additional discount rate**. The risky bond is priced as if the short rate were $r + \lambda$ rather than $r$ alone.

### 37.1.2 The Correlation Correction

The independence assumption is convenient but not always exact. O'Kane shows that when interest rates and hazard rates follow correlated processes, the risky zero-coupon bond price takes the form:

$$\hat{Z}(0, T) = Z(0, T) \cdot Q(0, T) \cdot \Theta(0, T, \rho)$$

where $\Theta$ is a multiplicative correction. For Gaussian dynamics with correlation $\rho$ between rates and hazards:

$$\Theta(0, T, \rho) = \exp\left(\frac{\rho \sigma_r \sigma_\lambda T^3}{3}\right)$$

O'Kane provides numerical examples showing this effect is typically small: "We can see that even for long maturities, the maximum correction to the risky zero coupon bond price is within a few tens of basis points." For a 10-year bond with 50% correlation and typical volatilities, $\Theta \approx 1.007$ (a 0.7% correction). This justifies the market-standard independence assumption for most pricing purposes.

### 37.1.3 Incorporating Recovery

Real bonds have recovery value. If default occurs, the bondholder typically receives some fraction $R$ of face value (or some recovery payment determined by auction or workout). Under the standard reduced-form assumption with recovery of face value at default, O'Kane shows the risky zero-coupon bond price becomes:

$$\hat{Z}(0,T) = Z(0,T) \cdot Q(0,T) + R \cdot \int_0^T Z(0,s) \cdot f(s) \, ds$$

where $f(s) = \lambda(s) Q(0,s)$ is the default density. The first term values the promised redemption conditional on survival; the second values the recovery payment received if default occurs before maturity.

**Practical approximation:** For small default probabilities and flat hazard rates, this simplifies considerably. Many practitioners use:

$$\hat{Z}(0,T) \approx Z(0,T) \cdot [Q(0,T) + R(1 - Q(0,T))]$$

which assigns the full recovery $R$ at maturity regardless of when default occurs—a reasonable approximation for investment-grade names where cumulative default probabilities are low.

### 37.1.4 Fixed-Rate Coupon Bond Pricing

A fixed-rate bond with coupon $c$, payment frequency $f$, and maturity $T$ has cash flows at dates $t_1, \ldots, t_N = T$. Under survival-weighted pricing:

$$\boxed{P = \sum_{n=1}^{N} \frac{c}{f} \cdot Z(0, t_n) \cdot Q(0, t_n) + Z(0, T) \cdot Q(0, T) + R \cdot PV(\text{default payment})}$$

The first sum values the coupons (received only if the issuer survives to each payment date). The second term values the principal redemption (received only if no default by maturity). The third term values recovery upon default.

O'Kane simplifies notation by defining the **credit risky discount factor**:

$$\hat{Z}(0, t) = Z(0, t) \cdot Q(0, t) \cdot \Theta(0, t, \rho)$$

When correlation is zero (a common assumption for tractability), we have $\Theta = 1$ and $\hat{Z}(0,t) = Z(0,t) \cdot Q(0,t)$.

**Key observation:** For practical pricing of investment-grade bonds, many desks use the **Z-spread approach** (from Chapter 8) rather than explicitly modeling survival curves. The Z-spread implicitly captures default intensity and recovery assumptions in a single spread parameter. However, for CDS hedging and basis analysis, explicit survival curve modeling becomes essential.

---

## 37.2 The Credit Spread Connection

### 37.2.1 From Survival Probabilities to Spreads

Chapter 36 introduced the **credit triangle** approximation:

$$S = \lambda(1 - R) \quad \Rightarrow \quad \lambda = \frac{S}{1-R}$$

This connects the continuously compounded credit spread $S$ to the hazard rate $\lambda$ under simplifying assumptions. O'Kane emphasizes that this is valid for continuous premium payments and constant hazard rates.

For a zero-coupon bond with zero recovery, the credit spread $s$ is defined as the excess continuously compounded yield over the risk-free rate. From $\hat{Z}(0,T) = e^{-(r+s)T}$ and $\hat{Z}(0,T) = Z(0,T) \cdot Q(0,T)$:

$$e^{-(r+s)T} = e^{-rT} \cdot e^{-\int_0^T \lambda(u) du}$$

Taking logs:

$$\boxed{s = \frac{1}{T} \int_0^T \lambda(u) \, du}$$

The credit spread equals the **average hazard rate** over the bond's life. For constant hazard, $s = \lambda$.

### 37.2.2 The Merton Model Perspective

O'Kane discusses Merton's structural model, which provides economic intuition for credit spreads. Under Merton's framework:

- The firm's assets follow geometric Brownian motion
- Debt is a zero-coupon obligation with face value $F$ maturing at time $T$
- Default occurs only at maturity if assets fall below $F$

The resulting credit spread depends on leverage and asset volatility. O'Kane presents the term structure of Merton spreads with specific examples using $\sigma_A = 20\%$ and $r = 5\%$:

> "When $A(t) = \$120$, we have $A(t) > F$ and so a bond maturing immediately can be repaid in full. As a result, the spread tends to zero as $T - t \to 0$. With increasing maturity, the risk of the asset value falling below $F$ increases and so the credit spread rises."

This generates upward-sloping credit spread curves for typical investment-grade issuers. However, for distressed credits where $A(t) < F$:

> "When $A(t) = \$99$ and $F = \$100$, we have $A(t) < F$. In this scenario, it would not be possible to redeem the bond if it matured immediately. As a result, the credit spread as $T - t \to 0$ tends to infinity. However, for longer maturities, there is a finite probability that the asset value will grow to exceed the face value and the bond becomes more likely to be repaid. As a result, the credit spread falls with increasing time to maturity."

**Limitations of Merton's model:** O'Kane notes several reasons why structural models are not widely used for credit derivatives pricing:

1. "The credit spread for firms for which $A(t) > F$ always tends to zero as $T - t \to 0$. This is not consistent with the credit markets in which even corporates with very high credit ratings have a finite spread at very short maturities."
2. The highly simplified capital structure is unrealistic
3. The model only allows default at a single time $T$
4. Limited transparency regarding firm asset values

O'Kane concludes: "Structural models perform best as a tool for augmenting the traditional balance sheet analysis methods of credit analysts... However, for the reasons listed above, structural models are not widely used in credit derivatives pricing."

Reduced-form models (intensity-based) better fit short-dated spread behavior and dominate in the credit derivatives market.

### 37.2.3 What Credit Spreads Really Contain

Chapter 8 introduced O'Kane's decomposition of market spreads. O'Kane breaks the credit spread into specific components:

> "**Actuarial spread:** This is what an investor in a credit security has to be paid to compensate them for the expected loss of the security as implied by historical default probabilities and historical recovery rates."

> "**Default risk premium:** This is the additional spread paid to compensate the investor for the risk that the historical default rate and recovery rate statistics do not reliably predict the default risk of the credit to which the investor is exposed."

> "**Volatility risk premium:** This is the additional spread paid to compensate the investor for the risk that the credit quality of the issuer changes, as evidenced by a change in the market spread of the issuer."

> "**Liquidity risk premium:** This premium compensates the investor for the risk that a reduction of the liquidity of the instrument prevents the investor from being able to sell the bond when they wish."

In summary:

$$\text{Credit spread} = \text{Actuarial spread} + \text{Default risk premium} + \text{Volatility risk premium} + \text{Liquidity risk premium}$$

O'Kane provides empirical evidence using 2002 CDS data:

| Rating | 5Y Avg Spread (bp) | Actuarial Spread (bp) | Coverage Ratio | Spread Premium (bp) |
|--------|-------------------|----------------------|----------------|---------------------|
| AA | 28 | 9 | 3.12 | 19 |
| A | 61 | 13 | 4.67 | 48 |
| BBB | 164 | 30 | 5.54 | 134 |
| BB | 463 | 145 | 3.19 | 318 |

O'Kane observes: "We see that the coverage ratio tends to remain fairly constant as a function of rating... The spread premium increases as we descend the rating spectrum."

This means that inferring hazard rates directly from spreads (via $\lambda = s/(1-R)$) yields **risk-neutral** default intensities that exceed historical rates. Hull reinforces this: "The default probabilities or hazard rates implied from credit spreads are risk-neutral estimates."

Hull's Table 24.3 shows the expected excess return on bonds:

| Rating | Bond Yield Spread | Spread for Historical Defaults | Excess Return |
|--------|-------------------|-------------------------------|---------------|
| Aaa | 40 bp | 2 bp | 38 bp |
| A | 77 bp | 8 bp | 69 bp |
| Baa | 143 bp | 28 bp | 115 bp |
| Ba | 304 bp | 144 bp | 160 bp |

**Practical implication:** When calibrating survival curves to bond or CDS spreads, you are obtaining $\mathbb{Q}$-measure (pricing) hazards, not $\mathbb{P}$-measure (forecasting) hazards. These are appropriate for pricing and hedging but not for predicting actual default rates.

---

## 37.3 Credit DV01 and Spread Duration

### 37.3.1 Spread Duration: The Core Definition

Spread duration measures the percentage price sensitivity to a one basis point change in spread. O'Kane defines the **modified duration** with respect to the yield spread as:

$$\boxed{D_s = -\frac{1}{P} \frac{\partial P}{\partial s}}$$

For a fixed-rate corporate bond with yield decomposed as $y = y_T + s$ (Treasury yield plus spread), O'Kane shows a striking result through symmetry:

$$D_s = D_{y_T}$$

**The spread duration equals the interest rate duration.** This follows from symmetry: the bond price depends on $y = y_T + s$, so:

$$\frac{\partial P}{\partial s} = \frac{\partial P}{\partial y_T}$$

O'Kane emphasizes: "We see that for the defaultable bond the interest rate duration and spread duration are exactly the same. A fixed rate corporate bond is therefore as much an interest rate play, as it is a credit play, i.e. a change of 1 bp in the yield spread has the same effect on the price as a change of 1 bp in the Treasury yield."

This is a crucial insight for portfolio management: a corporate bond holder has *dual exposure* to rates and credit, both measured by the same duration number.

### 37.3.2 Credit DV01 (CS01)

The **Credit DV01** measures the absolute dollar change in value for a 1 bp spread increase. O'Kane defines it for CDS positions:

$$\boxed{\text{Credit DV01} = -(P(s + 1\text{ bp}) - P(s))}$$

The negative sign ensures that Credit DV01 is positive for a long bond position (price falls when spreads widen). Analytically:

$$\text{Credit DV01} \approx -\frac{\partial P}{\partial s} \times 1\text{ bp} = P \cdot D_s \times 0.0001$$

**Example (from O'Kane):** Consider a \$10 million face value CDS with an RPV01 (risky PV01) of 4.205. A 1 bp spread widening causes:

$$\Delta V = -\text{Credit DV01} = -4.205 \times \frac{1}{10000} \times \$10\text{m} = -\$4,205$$

### 37.3.3 Credit DV01 for CDS Positions

For a CDS contract at market spread $S_t$ with contractual spread $S_0$, O'Kane derives:

$$\frac{\text{Credit DV01}}{1\text{ bp}} = \text{RPV01}(t, T) + \frac{\Delta(S_0 - S_t)}{(1-R)} \sum_{n=1}^{N} \tau_n \exp\left(-(r_t + S_t(1-R)^{-1})\tau_n\right)$$

O'Kane identifies the key relationships:

> "When $S_t = S_0$, the Credit DV01 equals the RPV01 of the contract times 1bp."

> "When $S_t$ deviates from $S_0$, this is no longer true. The Credit DV01 increases or decreases depending on the value of ($S_0 - S_t$)."

This formula shows that **the risky PV01 is the primary driver of spread sensitivity**, with adjustments when the position is in-the-money or out-of-the-money.

### 37.3.4 Floating Rate Notes: Pure Credit Play

For floating rate notes (FRNs), O'Kane demonstrates a fundamentally different risk profile. The LIBOR DV01 is minimal, while the Credit DV01 is comparable to a fixed-rate bond.

O'Kane provides a detailed example of a $1 million face value FRN with 5-year maturity, quoted margin of 75 bp, and current par floater spread of 100 bp:

> "The interest rate DV01 is $-\$2.70$... We find that the credit DV01 is much larger at \$430. The credit modified duration is 4.348. This is similar to a fixed rate corporate bond of the same maturity."

O'Kane concludes forcefully:

> "These results make it very clear that a floating rate note is predominantly a credit play. The interest rate sensitivity is small. However, the credit risk remains and is comparable to that of a fixed rate bond of a similar maturity."

**Why is the Libor DV01 small?** FRN coupons reset to market rates, so the bond trades near par regardless of rate level (assuming credit quality unchanged). The only interest rate sensitivity comes from the stub period to the next coupon reset.

**Why is Credit DV01 large?** Spread changes affect the *discounting* of all future cash flows, not just coupon determination. A wider par floater spread reduces the present value of all future payments.

This explains why FRNs are used in asset-liability matching by banks: they eliminate duration risk while retaining credit exposure.

---

## 37.4 Interest Rate Hedging: Isolating the Credit Play

### 37.4.1 Duration Hedging with Treasuries

To convert a corporate bond into a pure credit exposure, practitioners hedge the interest rate risk by shorting Treasury bonds. O'Kane provides the hedge ratio derivation.

For a hedged position:

$$V = F \cdot P - F_T \cdot P_T$$

where $F$ is corporate face value, $F_T$ is Treasury face value, $P$ and $P_T$ are the respective full prices. A first-order hedged position requires $\partial V / \partial y_T = 0$, giving:

$$\boxed{F_T = \frac{\partial P / \partial y_T}{\partial P_T / \partial y_T} \times F}$$

Equivalently, using DV01s:

$$F_T = \frac{\text{DV01}_{\text{corp}}}{\text{DV01}_{\text{Tsy}}} \times F$$

**Example (from O'Kane):** For a 5-year corporate bond with DV01 = 4.3668 per 100 face and a Treasury with DV01 = 4.4879:

$$F_T = \frac{4.3668}{4.4879} \times \$100 = \$97.30$$

O'Kane explains: "The face value of the Treasury position is less than \$100, reflecting the higher interest rate sensitivity of the Treasury bond." The Treasury (with lower yield) has higher duration per dollar of face.

### 37.4.2 Quality of the Hedge

O'Kane demonstrates hedge effectiveness with a stress test:

> "A reasonably large 1% movement in the Treasury yield from 5% to 6% causes the hedged position to lose 0.32 cents on a \$100 face value corporate bond position. This is small compared to the \$4.26 the bondholder would lose on the unhedged position."

The residual exposure is **spread convexity** (second-order) plus any basis risk between corporate and Treasury yield movements. O'Kane also notes that the hedged position exhibits short yield convexity: "the value of the net position is a concave function of the yield."

### 37.4.3 The Hedged Position Is a Pure Credit Play

O'Kane states directly:

> "It is clear from this example that the value of the corporate bond plus interest rate duration hedge is almost exclusively driven by changes in the yield spread, i.e. it is almost a pure credit play."

This is the foundation of credit investing: use rate hedges to neutralize parallel rate moves, leaving spread (credit) risk as the dominant P&L driver. The construction is analogous to an asset swap (Chapter 8), which also transforms a fixed-rate bond into a credit exposure.

---

## 37.5 The Z-Spread and Zero Volatility Spread

### 37.5.1 Definition

Chapter 8 introduced the Z-spread. O'Kane defines it precisely for credit bonds:

> "The ZVS for a specific bond is defined as the fixed spread adjustment to the Libor discount rate which reprices the bond."

In discrete compounding:

$$P = \frac{c}{f} \sum_{n=1}^{N} \frac{1}{(1 + (r(0,t_n) + \theta)/f)^n} + \frac{1}{(1 + (r(0,t_N) + \theta)/f)^N}$$

where $r(0,t)$ is the discretely compounded LIBOR zero rate and $\theta$ is the Z-spread.

In continuous compounding:

$$P = \frac{c}{f} \sum_{n=1}^{N} Z(0, t_n) \cdot e^{-\theta t_n} + Z(0, T) \cdot e^{-\theta T}$$

### 37.5.2 Z-Spread vs Yield Spread

O'Kane explains the motivation for Z-spread:

> "The problem with the yield spread is that it is the spread to a fixed Treasury yield $y_T$ and so does not take into account the shape of the Treasury bond implied term structure of yields—all of the coupons of a bond, irrespective of when they are paid, are discounted at the same yield."

The Z-spread incorporates the full term structure, providing a more consistent measure across bonds with different coupons and maturities.

O'Kane notes: "Some practitioners choose to hedge using the ZVS rather than the yield. The ZVS is preferred because it takes into account the term structure of the interest rate curve. As with the yield spread, it is possible to write a ZVS-based duration and convexity measure."

### 37.5.3 Connection to Hazard Rates

Under simplifying assumptions (deterministic rates and hazards, zero correlation), the Z-spread relates to the hazard rate. For zero-recovery bonds:

$$\theta \approx \lambda$$

More generally, accounting for recovery:

$$\theta \approx \lambda(1 - R)$$

This connects the market-observable Z-spread to the survival curve parameters, establishing the link between spread-based and survival-based pricing approaches.

---

## 37.6 The CDS-Bond Basis: Why Spreads Diverge

### 37.6.1 Definition

Hull RM defines the CDS-bond basis:

$$\boxed{\text{CDS-bond basis} = \text{CDS spread} - \text{Bond yield spread}}$$

The bond yield spread is typically measured as the asset swap spread (Chapter 8). In theory, arbitrage forces should keep this basis near zero: buying a bond and CDS protection creates a near-riskless position earning close to the risk-free rate.

### 37.6.2 Why the Basis Is Not Zero

Hull RM lists several factors driving non-zero basis:

1. **Bond price vs par:** "Bond prices above par tend to give rise to a negative basis; bond prices below par tend to give rise to a positive basis."

2. **Counterparty risk in CDS:** "There is counterparty default risk in a CDS. (This pushes the basis in a negative direction.)"

3. **Cheapest-to-deliver option:** "There is a cheapest-to-deliver bond option in a CDS. (This pushes the basis in a positive direction.)"

4. **Accrued interest treatment:** Different accrual mechanics in bonds versus CDS.

5. **Restructuring clauses:** "The restructuring clause in a CDS contract may lead to a payoff when there is no default. (This pushes the basis in a positive direction.)"

6. **Funding differences:** "LIBOR is greater than the risk-free rate being assumed by the market. (This pushes the basis in a positive direction.)"

O'Kane similarly defines the CDS-cash basis: "CDS basis = CDS spread - Bond Libor spread" and notes the same drivers apply.

### 37.6.3 Historical Behavior

Hull RM notes:

> "Prior to the market turmoil starting in 2007, the basis tended to be positive. For example, De Witt estimates that the average CDS-bond basis in 2004 and 2005 was 16 basis points."

During the 2008 crisis, the basis became severely negative as bond prices collapsed while CDS protection became expensive. Hull RM explains:

> "It was difficult for financial institutions to arbitrage between bonds and CDSs because of a shortage of liquidity and other considerations."

Since the crisis, Hull notes that "the magnitude of the CDS-bond basis (sometimes positive and sometimes negative) has become much smaller."

The CDS-bond basis trade is a major application of the credit analysis in this chapter and Part IX. Understanding the drivers of the basis is essential for relative value trading.

---

## 37.7 Worked Examples

All examples use 100 notional unless stated. Spreads in basis points.

### Example A: Risky Zero-Coupon Bond Price

**Given:**
- Risk-free 5-year zero rate: $r = 4\%$ (continuous)
- Constant hazard rate: $\lambda = 2\%$
- Recovery: $R = 0$ (for simplicity)

**Compute price:**

$$Z(0,5) = e^{-0.04 \times 5} = e^{-0.20} = 0.8187$$

$$Q(0,5) = e^{-0.02 \times 5} = e^{-0.10} = 0.9048$$

$$\hat{Z}(0,5) = Z(0,5) \cdot Q(0,5) = 0.8187 \times 0.9048 = 0.7408$$

**Output:** Risky ZCB price = **74.08** per 100 face.

**Verification:** Alternatively, $\hat{Z}(0,5) = e^{-(r+\lambda) \times 5} = e^{-0.30} = 0.7408$ ✓

### Example B: Credit Spread from Risky Bond Price

**Given:**
- Same risky ZCB with price $\hat{Z}(0,5) = 74.08$
- Risk-free ZCB price $Z(0,5) = 81.87$

**Compute credit spread:**

The risky yield is $-\frac{1}{5}\ln(0.7408) = 0.060$ (6.0% continuous).
The risk-free yield is $-\frac{1}{5}\ln(0.8187) = 0.040$ (4.0% continuous).

$$s = 6.0\% - 4.0\% = 2.0\% = 200 \text{ bp}$$

**Using hazard:** With $\lambda = 2\%$ and $R = 0$:
$$s = \lambda(1-R) = 0.02 \times 1.0 = 0.02 = 200 \text{ bp ✓}$$

### Example C: Credit DV01 Calculation

**Given:**
- 5-year fixed-rate bond: 6% coupon (semiannual), par price = 100
- Spread duration $D_s = 4.20$

**Compute Credit DV01:**

$$\text{Credit DV01} = P \cdot D_s \times 0.0001 = 100 \times 4.20 \times 0.0001 = 0.042$$

**Interpretation:** For each 1 bp spread widening, the bond price falls by **\$0.042 per \$100 face**, or \$42 per \$100,000 position.

### Example D: Duration Hedge Ratio

**Given:**
- Corporate bond: DV01 = 4.50 per 100
- Treasury bond: DV01 = 4.80 per 100
- Corporate position: \$10 million face

**Compute Treasury hedge:**

$$F_T = \frac{4.50}{4.80} \times \$10\text{m} = 0.9375 \times \$10\text{m} = \$9.375\text{m}$$

**Output:** Short **\$9.375 million** face value of Treasuries.

### Example E: Survival Probability to Spread

**Given:**
- $Q(0,5) = 0.90$ (10% cumulative default probability over 5 years)
- Recovery $R = 40\%$

**Compute average hazard:**

$$\bar{\lambda} = -\frac{\ln(Q(0,5))}{5} = -\frac{\ln(0.90)}{5} = \frac{0.1054}{5} = 2.11\%$$

**Compute approximate spread (using credit triangle):**

$$s \approx \bar{\lambda}(1-R) = 0.0211 \times 0.60 = 0.0127 = 127 \text{ bp}$$

**Output:** Spread ≈ **127 bp**.

### Example F: FRN Credit DV01 vs LIBOR DV01

**Given (from O'Kane's example):**
- 5-year FRN, \$1 million face
- Quoted margin: 75 bp; current par floater spread: 100 bp
- Price: 98.92

**Results:**
- LIBOR DV01 = -\$2.70 (small, negative—position benefits from rate rises due to discount bond economics)
- Credit DV01 = \$430 (large, positive—sensitive to spread changes)

**Interpretation:** The FRN is dominated by credit risk. A 100 bp spread widening would cause roughly:

$$\Delta P \approx 100 \times \frac{\$430}{1\text{ bp}} = \$43,000$$

on a \$1 million position.

### Example G: Risk Premium Decomposition

**Given (from O'Kane Table 3.4):**
- BBB-rated credit: 5-year CDS spread = 164 bp
- Historical 5-year default probability = 2.50%
- Expected loss (with 40% recovery) = 1.50%
- Actuarial spread = 30 bp

**Compute coverage ratio and spread premium:**

$$\text{Coverage ratio} = \frac{164}{30} = 5.47$$

$$\text{Spread premium} = 164 - 30 = 134 \text{ bp}$$

**Interpretation:** The market spread "covers" the actuarial default risk about 5.5 times. The 134 bp excess compensates for default risk premium, volatility risk premium, and liquidity risk premium.

---

## 37.8 Practical Notes

### 37.8.1 Which Spread Measure to Use?

| Situation | Preferred Spread |
|-----------|------------------|
| Quick relative value across similar bonds | Yield spread or G-spread |
| Curve-consistent valuation | Z-spread |
| CDS hedging and basis analysis | Par CDS spread |
| Asset swap structuring | Asset swap spread |
| Bonds with optionality | OAS |

O'Kane notes: "Some practitioners choose to hedge using the ZVS rather than the yield. The ZVS is preferred because it takes into account the term structure of the interest rate curve."

### 37.8.2 Recovery Assumption Sensitivity

From Chapter 36, recall: $\lambda = s/(1-R)$. Changing recovery assumptions significantly affects implied hazard rates:

| Spread (bp) | R = 20% | R = 40% | R = 60% |
|-------------|---------|---------|---------|
| 120 | 1.50% | 2.00% | 3.00% |
| 200 | 2.50% | 3.33% | 5.00% |

Higher assumed recovery implies higher hazard rate to justify the same spread.

### 37.8.3 Common Implementation Pitfalls

**Day count mismatches:** Bond accrual uses 30/360 or ACT/ACT; swap/CDS uses ACT/360. Ensure consistency when computing asset swap spreads.

**Clean vs dirty price:** Spread engines should calibrate to dirty price. Market quotes are clean price. Always add accrued interest.

**Risk-neutral vs historical:** Spreads imply $\mathbb{Q}$-hazards. Do not use these for credit forecasting without adjustment. Hull's data shows risk-neutral hazard rates can be 5-10 times historical rates.

**Basis ambiguity:** "Bond spread" can mean G-spread, I-spread, Z-spread, or asset swap spread. Always clarify which measure is being discussed.

**Independence assumption:** The standard pricing framework assumes interest rates and hazard rates are independent. O'Kane shows this effect is typically small, but it can matter for long-dated or high-correlation scenarios.

### 37.8.4 Sanity Checks

- Credit DV01 should be comparable to interest rate DV01 for fixed-rate bonds
- Credit DV01 should dominate for FRNs (small LIBOR DV01)
- Spread × duration × notional ≈ potential P&L for large spread moves
- Coverage ratio (market spread / actuarial spread) typically 3-6× for investment grade

---

## Summary

This chapter developed the pricing and risk management framework for credit-risky bonds:

1. **Risky discounting** factors survival probability into bond valuation: $\hat{Z}(0,T) = Z(0,T) \cdot Q(0,T)$

2. **Credit spreads** embed default intensity, recovery, and risk premia: $s \approx \lambda(1-R)$

3. **Market spreads exceed actuarial spreads** by 3-6× for investment grade, reflecting default, volatility, and liquidity risk premia

4. **Spread duration equals interest rate duration** for fixed-rate corporates: a 1 bp spread move has the same effect as a 1 bp rate move

5. **Credit DV01** measures dollar price change per 1 bp spread widening: Credit DV01 $= P \cdot D_s \times 0.0001$

6. **Interest rate hedging** isolates credit exposure, converting a bond into a "pure credit play"

7. **FRNs are predominantly credit plays** with minimal rate risk

8. **The CDS-bond basis** diverges from zero due to funding, delivery options, and market technicals

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Risky discount factor | $\hat{Z}(0,T) = Z(0,T) \cdot Q(0,T)$ | Fundamental pricing building block |
| Credit spread | Excess yield over risk-free rate | Compensation for default/liquidity risk |
| Spread duration | $D_s = -\frac{1}{P}\frac{\partial P}{\partial s}$ | Measures credit sensitivity |
| Credit DV01 (CS01) | Dollar change per 1 bp spread | Primary risk metric for credit positions |
| Duration hedge ratio | $\frac{\text{DV01}_{\text{corp}}}{\text{DV01}_{\text{Tsy}}}$ | Converts bond to pure credit exposure |
| CDS-bond basis | CDS spread minus bond spread | Measures relative value, arbitrage opportunity |
| Coverage ratio | Market spread / actuarial spread | Measures risk premium embedded in spreads |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $\hat{Z}(0,T)$ | Credit-risky zero-coupon bond price (zero recovery) |
| $Z(0,T)$ | Risk-free discount factor |
| $Q(0,T)$ | Survival probability to time $T$ |
| $\lambda$ | Hazard rate / default intensity |
| $R$ | Recovery fraction |
| $s$ | Credit spread (continuously compounded) |
| $D_s$ | Spread duration (modified duration w.r.t. spread) |
| Credit DV01 | Dollar price change per 1 bp spread widening |
| RPV01 | Risky PV01 (CDS premium leg present value per 1 bp) |
| $\Theta$ | Correlation correction factor for rates/hazards |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | How does survival probability enter zero-coupon bond pricing? | $\hat{Z}(0,T) = Z(0,T) \cdot Q(0,T)$ |
| 2 | What does $\lambda(1-R) = s$ represent? | Credit triangle: spread ≈ hazard × loss given default |
| 3 | Define spread duration | $D_s = -\frac{1}{P}\frac{\partial P}{\partial s}$ |
| 4 | For a fixed-rate corporate, how does $D_s$ relate to $D_{y_T}$? | They are equal: $D_s = D_{y_T}$ |
| 5 | What is Credit DV01? | Dollar price change per 1 bp spread widening |
| 6 | Formula for Credit DV01? | Credit DV01 $= P \cdot D_s \times 0.0001$ |
| 7 | How do you hedge a corporate bond's rate risk? | Short Treasuries with hedge ratio = DV01$_\text{corp}$/DV01$_\text{Tsy}$ |
| 8 | Why is an FRN a "pure credit play"? | LIBOR DV01 is minimal; Credit DV01 dominates |
| 9 | What is the CDS-bond basis? | CDS spread minus bond yield spread |
| 10 | When is CDS-bond basis typically positive? | Normal markets (bond cheap vs CDS) |
| 11 | When did CDS-bond basis become severely negative? | 2008 financial crisis (liquidity constraints) |
| 12 | What measure are spread-implied hazard rates? | Risk-neutral ($\mathbb{Q}$) |
| 13 | Why does Merton model give zero short-term spread for solvent firms? | Default only at maturity; immediate repayment certain if $A > F$ |
| 14 | What are the components of market credit spreads (O'Kane)? | Actuarial + default risk premium + volatility premium + liquidity premium |
| 15 | Why is Z-spread preferred over yield spread? | Z-spread incorporates the full term structure |
| 16 | How does recovery affect implied hazard? | Higher $R$ implies higher $\lambda$ for same spread |
| 17 | What is the RPV01? | Risky PV01—present value of CDS premium leg per 1 bp |
| 18 | When does Credit DV01 equal RPV01 × 1 bp? | When CDS is at-market ($S_t = S_0$) |
| 19 | What is the typical coverage ratio for investment-grade credits? | 3-6× (market spread / actuarial spread) |
| 20 | What is a "duration-hedged bond position"? | Long corporate + short Treasury to neutralize rate risk |

---

## Mini Problem Set

**1.** A zero-recovery zero-coupon bond has 3-year maturity. Risk-free 3-year zero rate is 3% (continuous). Hazard rate is constant at 1.5%. Compute the bond price and credit spread.

**Solution sketch:** $Z = e^{-0.09} = 0.9139$; $Q = e^{-0.045} = 0.9560$; $\hat{Z} = 0.8737$. Risky yield = $-\ln(0.8737)/3 = 4.50\%$; spread = $4.50\% - 3\% = 1.50\% = 150$ bp.

---

**2.** A 5-year corporate bond has DV01 = 4.25. Treasury DV01 = 4.50. What face value of Treasury should be shorted to hedge \$5 million face of corporate?

**Solution sketch:** Hedge = $\frac{4.25}{4.50} \times 5\text{m} = 4.722\text{m}$.

---

**3.** Using credit triangle, compute implied hazard rate from 200 bp spread with 40% recovery.

**Solution sketch:** $\lambda = 0.02 / 0.6 = 3.33\%$ per year.

---

**4.** A bond has price 102 and spread duration 4.5. Compute CS01.

**Solution sketch:** CS01 $= 102 \times 4.5 \times 0.0001 = 0.0459$ per 100 face.

---

**5.** If $Q(0,5) = 0.92$ and $R = 0.35$, estimate the 5-year credit spread.

**Solution sketch:** $\bar{\lambda} = -\ln(0.92)/5 = 1.67\%$; $s \approx 1.67\% \times 0.65 = 1.09\% = 109$ bp.

---

**6.** Explain why FRNs have low LIBOR DV01 but high Credit DV01.

**Solution sketch:** FRN coupons reset to market rates, so price is insensitive to rate level (only stub period matters). But spread changes affect discounting of all future cash flows, creating full-maturity spread sensitivity.

---

**7.** List three reasons the CDS-bond basis can be positive.

**Solution sketch:** (1) Cheapest-to-deliver option in CDS; (2) Restructuring clause pays out without technical default; (3) LIBOR exceeds risk-free rate assumed by market.

---

**8.** If Merton model gives spread = 0 for short-term bonds, why do market spreads stay positive?

**Solution sketch:** Merton allows default only at maturity. If assets exceed liabilities, immediate repayment is certain. Real markets have "surprise" default risk modeled by reduced-form hazard rates, which are positive at all horizons.

---

**9.** A trader says "CS01 is 45 per million." What information is missing to interpret this?

**Solution sketch:** The currency and whether "per million" refers to face value or market value. Also, whether this is per 1 bp or per 100 bp move (convention varies).

---

**10.** Derive the relationship $s = \bar{\lambda}$ for zero-recovery ZCBs from first principles.

**Solution sketch:** Start with $\hat{Z}(0,T) = Z(0,T) \cdot Q(0,T)$. Take logs: $\ln\hat{Z} = \ln Z + \ln Q$. Write in terms of yields: $-(r+s)T = -rT - \int_0^T \lambda(u)du$. Solve: $s = \frac{1}{T}\int_0^T \lambda(u)du = \bar{\lambda}$.

---

**11.** A BBB bond has 5Y CDS spread of 150 bp. Historical 5Y default rate is 2.5%, recovery 40%. Calculate the coverage ratio and spread premium.

**Solution sketch:** Actuarial spread $\approx 2.5\% \times 0.6 / 5 \times 10000 = 30$ bp. Coverage = $150/30 = 5.0$. Premium = $150 - 30 = 120$ bp.

---

**12.** A \$50 million corporate bond position has Credit DV01 of \$21,000 (total). If spreads widen 25 bp, estimate the P&L.

**Solution sketch:** P&L $\approx -25 \times \$21,000 = -\$525,000$ loss.

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

| Concept | Source |
|---------|--------|
| Risky ZCB = risk-free × survival | O'Kane Ch 3.5.1, 3.7.1, 3.9.1 |
| Correlation correction $\Theta$ | O'Kane Ch 3.8.1 |
| Credit DV01 definition for CDS | O'Kane Ch 8.3.2 |
| Spread duration = interest rate duration for corporates | O'Kane Ch 4.2.5 |
| FRN is predominantly a credit play | O'Kane Ch 4.3.3 (explicit quote) |
| FRN numerical example (DV01 values) | O'Kane Ch 4.3.3 Table 4.3 |
| Duration hedge ratio formula | O'Kane Ch 4.2.6 |
| Hedge effectiveness example | O'Kane Ch 4.2.6 |
| Z-spread (ZVS) definition | O'Kane Ch 4.2.9 |
| CDS-bond basis definition | Hull RM Ch 19.5 |
| CDS-bond basis drivers (6 factors) | Hull RM Ch 19.5 |
| Historical basis behavior | Hull RM Ch 19.5 (De Witt reference) |
| Merton model spread term structure | O'Kane Ch 3.4, Figure 3.4 |
| Merton model limitations | O'Kane Ch 3.4 |
| Credit spread decomposition (4 components) | O'Kane Ch 3.11 |
| Coverage ratio/spread premium table | O'Kane Ch 3.11 Table 3.4 |
| Risk-neutral vs historical hazards | Hull OFD Ch 24.5, O'Kane Ch 3.11 |
| Risk-neutral excess return table | Hull OFD Ch 24.5 Table 24.3 |

### (B) Reasoned Inference — Note Derivation Logic

| Concept | Derivation |
|---------|------------|
| $s = \bar{\lambda}$ for zero-recovery | Taking logs of $\hat{Z} = Z \cdot Q$ |
| Credit DV01 = $P \cdot D_s \times 0.0001$ | Definition of duration applied |
| Hedge ratio from DV01 | First-order Taylor expansion for rate-neutrality |
| Recovery sensitivity of hazard | Algebraic rearrangement of $\lambda = s/(1-R)$ |
| Z-spread ≈ $\lambda(1-R)$ | Combining survival-weighted pricing with spread definition |

### (C) Flagged Uncertainties

| Item | Uncertainty |
|------|-------------|
| Exact accrued treatment in CDS vs bond | I'm not sure about specific ISDA conventions without checking current contract documentation |
| Correlation adjustment $\Theta$ calibration | O'Kane provides formula but practical calibration is desk-specific and I'm not sure about current market practice |
| Market vs par asset swap conventions | Different desks use different standards; verify with counterparty |
| Current CDS-bond basis levels | Hull cites historical data; current levels vary by name and market conditions |
