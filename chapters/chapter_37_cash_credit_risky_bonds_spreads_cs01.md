# Chapter 37: Cash Credit — Risky Bonds, Credit Spreads, and CS01

---

## Introduction

A portfolio manager calls with a simple question: "I'm down $2 million on my corporate bond book today—is it rates or credit?" On the surface, this seems straightforward, but the answer requires separating two distinct sources of risk: the general level of interest rates (which affects all bonds) and the credit spread specific to the issuer (which reflects default risk, liquidity, and risk premia). Getting this decomposition wrong means misattributing P&L, mishedging positions, and misunderstanding the portfolio's true exposures.

**Why this matters for desk practice:** Middle-office professionals often see spread numbers in risk reports—Z-spread, asset swap spread, CS01—without fully understanding how these measures are constructed, what assumptions they embed, and why they sometimes diverge from CDS spreads on the same issuer. When spreads "blow out" during a credit event or market stress, understanding the mechanics becomes critical: a rates-hedged position can still hemorrhage money through credit spread widening.

This chapter develops the complete framework for cash credit analytics: pricing risky bonds using survival probabilities and recovery assumptions (building on Chapter 36), defining spread measures that isolate credit risk from rate risk, computing credit sensitivities (CS01/spread duration) that drive P&L attribution, and understanding the relationship between cash bonds and the CDS market through the lens of the cash-CDS basis. We also cover floating rate notes and asset swaps—instruments central to how bank desks view credit exposure.

**Roadmap:** We begin with conventions and notation (Section 0), then develop core definitions including FRN pricing and asset swaps (Section 1). Section 2 derives the mathematical framework for risky bond pricing. Section 3 covers measurement and risk analytics, including spread duration, CS01, credit risk premium decomposition, distressed debt dynamics, and the comprehensive treatment of cash-CDS basis drivers. The chapter concludes with worked examples, practical notes, and a substantial problem set.

---

## Conventions & Notation

### Price Units
Unless stated otherwise, prices/PVs are per $100 face ("per 100"). Dollar PV for notional $N$ is:

$$PV_{\$} = N \cdot P / 100$$

### Clean vs Dirty
Dirty (full) price $P$ equals clean price $P_{\text{clean}}$ plus accrued interest $AI$:

$$P = P_{\text{clean}} + AI$$

### Risk-Free Discounting
Base curve is represented by risk-free discount factors $Z(0,t)$ (e.g., government/OIS). Risk-free PV of deterministic cashflows is:

$$\sum_{i} CF_i \, Z(0, t_i)$$

### Default Time
$\tau$ is the (random) default time.

### Survival Curve
$Q(0,t) = P(\tau > t)$. Hazard / default intensity $\lambda(t)$ satisfies:

$$Q(0,T) = \exp\!\left(-\int_0^T \lambda(t) \, dt\right)$$

### Recovery Rate
$R \in [0,1]$. In examples we default to fractional recovery of face value (par) unless stated, but we explicitly flag that other recovery conventions exist (see Setup).

### Z-spread / ZVS
$z$ is the constant spread added to the base discounting rates so that discounted cashflows match the market price (O'Kane calls this the zero-volatility spread, ZVS).

### DV01
Interest-rate DV01 is the dollar price change for a 1 bp move in a stated rate measure; Tuckman defines:

$$DV01 = \frac{PD}{10{,}000}$$

when $D$ is modified duration and $P$ is full price.

### CS01 / Credit DV01
In this chapter, CS01 is defined analogously as the (signed) PV change per 1 bp change in a stated credit spread measure (e.g., Z-spread), with sign convention made explicit in Section 3.

---

## 0. Setup

### Conventions Used in This Chapter

- **Coupon bonds** pay fixed coupons at deterministic dates $t_1, \ldots, t_N$ with contractual cashflows $CF_i$ if no default occurs before payment.

- **Default** is modeled in a reduced-form / intensity style via survival probabilities $Q(0,t)$ and hazard $\lambda(t)$.

- We separate:
  - **Base (risk-free) curve** $Z(0,t)$ as the time-value-of-money input.
  - **Credit component** $Q(0,t)$ and $R$ to account for default and recovery.

### Recovery Conventions (Important)

Multiple recovery assumptions exist in the literature:

| Convention | Description |
|------------|-------------|
| **RT** | Recovery of Treasury |
| **RFV** | Recovery of Face Value |
| **RMV** | Recovery of Market Value |

We use **RFV** for numeric examples unless otherwise stated, because it maps directly to "payment at default" building blocks that occur "commonly in bonds as recovery payments."

### Notation Glossary

| Symbol | Meaning |
|--------|---------|
| $t$ | Time in years |
| $t_i$ | Coupon payment dates; $i = 1, \ldots, N$ |
| $CF_i$ | Contractual cashflow at $t_i$ (coupon, and principal at maturity) |
| $P$ | Dirty (full) price per 100 |
| $P_{\text{clean}}$ | Clean price per 100 |
| $AI$ | Accrued interest per 100 |
| $Z(0,t)$ | Risk-free discount factor to time $t$ |
| $\tau$ | Default time |
| $Q(0,t) = P(\tau > t)$ | Survival probability to $t$ |
| $\lambda(t)$ | Hazard rate / default intensity; $Q(0,T) = \exp(-\int_0^T \lambda)$ |
| $R$ | Recovery rate (dimensionless) |
| $s$ | Yield spread (corporate yield minus reference yield) |
| $z$ | Z-spread (ZVS) to the base curve (in decimals; 1 bp = $10^{-4}$) |
| $D$ | Modified duration (rate duration) |
| $D_s$ | Spread duration (spread sensitivity) |
| $DV01$ | Rate DV01 (dollars per 1 bp in the chosen rate measure) |
| $CS01$ | Credit spread DV01 / credit DV01 (dollars per 1 bp in the chosen spread measure) |
| $F(t)$ | Par floater spread at time $t$ |
| $F(0)$ | Quoted margin (original spread at issuance) |

---

## 1. Core Concepts (Definitions First)

### 1.1 Risk-Free PV (Baseline)

**Formal Definition:**

For deterministic cashflows $CF_i$ at times $t_i$, risk-free PV:

$$\boxed{PV_{\text{rf}} = \sum_{i=1}^{N} CF_i \, Z(0, t_i)}$$

**Intuition:**

This is "time value of money only," assuming payments are certain.

**Practice:**

Used for government bonds, or as a baseline "base curve PV" for corporates before layering credit risk.

---

### 1.2 Default Time, Survival Curve, Hazard Rate

**Formal Definition:**

- **Survival probability:** $Q(0,t) = P(\tau > t)$

- **Hazard rate:** conditional default probability per unit time (under the model). In intensity form:

$$\boxed{Q(0,T) = \exp\!\left(-\int_0^T \lambda(t) \, dt\right)}$$

**Intuition:**

- $Q(0,t)$ declines from 1 toward 0 as the horizon increases.
- Larger $\lambda$ means faster decline in $Q$: "more default risk per unit time."

**Practice:**

Desks often handle credit via survival curves (from CDS/bonds) and separate them from the risk-free discount curve.

---

### 1.3 Recovery Conventions (What Is Paid After Default)

**Formal Definition:**

Different recovery models specify what is recovered at default:

| Model | Recovery Definition |
|-------|---------------------|
| **RT** | Recover a fraction of a default-free bond value |
| **RFV** | Recover a fraction of par |
| **RMV** | Recover a fraction of the firm's market value at default |

**Intuition:**

"Recovery" is not a single number without a convention; it's a cashflow rule after default.

**Practice:**

Corporate bond analytics frequently require choosing RFV vs market-value recovery; this choice affects implied hazard/spreads and CS01 attribution.

---

### 1.4 Defaultable (Risky) Bond Cashflows

**Formal Definition:**

A defaultable bond pays scheduled coupons/principal only if $\tau$ occurs after the payment date. If default occurs before maturity, some recovery cashflow may be paid at (or near) default.

**Intuition:**

The bond is like:
- A "survival-contingent" stream of coupons/principal, plus
- A "default-contingent" recovery payment.

**Practice:**

Desk PV decompositions often separate survival PV and recovery PV.

---

### 1.5 Yield Spread (A Quoting Statistic)

**Formal Definition:**

Corporate yield $y$ is often written as reference yield plus spread:

$$y = y_T + s$$

**Intuition:**

One number summarizing "how much higher the corporate yield is than the benchmark."

**Practice:**

Very common, but can be misleading across coupon structures and curve shapes.

---

### 1.6 Z-Spread (ZVS / Zero-Volatility Spread)

**Formal Definition:**

The ZVS/Z-spread is the constant spread $z$ added to the base curve such that discounted cashflows equal the market price:

$$\boxed{P_{\text{bond}} = \sum_i \frac{c/f}{\left(1 + \frac{r(0,t_i) + z}{f}\right)^{t_i \cdot f}} + \frac{1}{\left(1 + \frac{r(0,t_N) + z}{f}\right)^{t_N \cdot f}}}$$

and similarly for continuous compounding.

**Intuition:**

It is a single spread number that "tilts" the entire discount curve to match the market price.

**Practice:**

Used widely in cash credit to compare bonds across maturities and coupons (but still disordered by base curve and compounding convention).

> **Analogy: The Level Playing Field**
>
> Why use Z-Spread?
>
> *   **Yield Spread**: Like comparing heights of two people standing on different hills. (Bad comparison).
> *   **Z-Spread**: We build a platform (the Z-spread) that lifts the *entire* risk-free curve by a constant amount until the bond's price makes sense. It levels the field across different maturities.
> *   **Result**: It's the "average" risk premium over the whole life of the bond.

---

### 1.7 Asset Swap Spread

**Formal Definition:**

An asset swap is a package that transforms a fixed-rate bond into a synthetic floating-rate exposure. O'Kane defines two variants:

**Par Asset Swap:** The investor buys the bond at par (100), and the difference from market price is settled upfront. The asset swap spread $A$ is the spread over Libor that makes the PV of floating payments equal the PV of bond coupons:

$$\boxed{A(0) = \frac{P_{\text{Libor}} - P}{RPV01}}$$

where $P$ is the bond dirty price, $P_{\text{Libor}}$ is the Libor-discounted PV of bond cashflows, and $RPV01$ is the risky PV01 of the swap (annuity factor).

**Market Asset Swap:** The investor buys the bond at market price (not par). The structure is slightly different—the investor pays the clean bond price upfront and receives floating + spread on the notional.

**Intuition:**

"Bond spread measured on a swap floating leg basis." The asset swap spread answers: "If I swap the bond's fixed coupons for floating + spread, what spread do I receive?"

O'Kane explains that the par asset swap spread "corresponds to the amortization of any premium or discount over the life of the asset swap." A bond trading at a discount (below par) will have a higher asset swap spread than its Z-spread, because the discount is amortized over the swap life.

**Practice:**

Asset swaps are preferred by bank desks because they isolate credit risk from rate risk. The structure involves three components:
1. **Bond position:** Long the corporate bond
2. **Interest rate swap:** Receive fixed (matching bond coupon), pay Libor + spread
3. **Financing:** Fund the bond purchase in repo

> **Desk Reality: Why Asset Swaps?**
>
> Bank trading desks prefer asset swap spreads because:
> - They strip out interest rate risk (the swap hedges duration)
> - The spread is quoted over Libor, matching funding cost
> - Mark-to-market is cleaner for capital treatment
>
> **The catch:** If the bond defaults, the swap continues—the investor loses the bond but still owes swap payments. This "default contingent" risk is why asset swap spreads don't perfectly match CDS spreads.

---

### 1.8 Floating Rate Notes (FRN) and Par Floater Spread

**Formal Definition:**

A floating rate note pays coupons equal to a reference rate (e.g., Libor) plus a fixed spread called the **quoted margin** $F(0)$:

$$\text{Coupon}_i = (\text{Libor}_i + F(0)) \times \alpha_i \times \text{Notional}$$

where $\alpha_i$ is the accrual factor for period $i$.

**Par Floater Spread:** The spread $F(t)$ that would make a new FRN trade at par today. O'Kane shows that for a defaultable FRN:

$$\boxed{F(t) = \frac{1 - P_{\text{risky}}(t,T)}{RPV01(t,T)}}$$

where $P_{\text{risky}}(t,T)$ is the risky discount factor and $RPV01$ is the risky annuity.

**Intuition:**

The par floater spread measures **current credit quality** versus the credit quality at issuance. If credit has deteriorated since issuance, $F(t) > F(0)$ and the FRN trades below par. If credit has improved, $F(t) < F(0)$ and the FRN trades above par.

O'Kane emphasizes that the par floater spread is a cleaner credit measure than the asset swap spread because "the FRN is naturally a floating rate instrument" without the need for a swap overlay.

**FRN Price:**

Given a quoted margin $F(0)$ and current par floater spread $F(t)$, the FRN price is approximately:

$$P_{\text{FRN}} \approx 100 - (F(t) - F(0)) \times RPV01$$

**Credit DV01 for FRN:**

O'Kane calculates that a 5-year FRN has credit DV01 of approximately $430 per $10 million notional per bp—comparable to a fixed-rate bond of the same maturity, despite the FRN having near-zero interest rate duration.

**Practice:**

> **Desk Reality: FRN Credit Duration**
>
> A common misconception is that FRNs have low "duration" and therefore low risk. This confuses **interest rate duration** (which is indeed low—the floating rate resets eliminate rate sensitivity) with **credit duration** (which is NOT low—you still have years of credit exposure).
>
> An FRN maturing in 5 years has 5 years of credit risk. If the issuer's spread widens by 100 bp, the FRN price drops by approximately $RPV01 \times 100 \text{ bp} \approx 4\%$—similar to a fixed-rate bond.

---

### 1.9 Spread Duration and CS01 (Credit DV01)

**Formal Definition:**

If price depends on spread $s$, spread duration is the sensitivity of price to small spread changes:

$$\frac{dP}{P} \approx -D_s \, ds$$

in a decomposition where yield $y = y_T + s$.

Analogously, **credit DV01 / CS01** is the PV change per 1 bp move in the specified spread measure; O'Kane defines "credit DV01" for a spread $F$ with a negative sign so the measure is positive for a typical long-credit position:

$$\text{credit DV01} = -\bigl(P(F + 1\text{ bp}) - P(F)\bigr)$$

**The Key Insight: Spread Duration = Interest Rate Duration**

O'Kane emphasizes: "For a fixed rate bond, the interest rate duration and the spread duration are exactly the same." This follows directly from the yield decomposition $y = y_T + s$: bumping $y_T$ by 1 bp or bumping $s$ by 1 bp has the identical effect on the yield $y$ and therefore on the bond price.

$$\boxed{D_s = D_{y_T} \quad \text{(for fixed-rate bonds)}}$$

**Implication:** "A fixed rate corporate bond is therefore as much an interest rate play as it is a credit play." A trader who is long a 10-year corporate bond has 10-year duration exposure to BOTH rates AND spreads.

**Intuition:**

"How much money do I lose if credit spreads widen by 1 bp?"

**Practice:**

Used for risk reports, hedging with CDS/indices, and scenario analysis. Always depends on the chosen spread definition and base curve.

---

## 2. Math and Derivations (Step-by-Step)

**Goal:** Start from deterministic PV, introduce default via $\tau$, and obtain a workable risky-bond PV decomposition using survival and recovery.

### 2.1 Deterministic PV (No Default)

Let cashflows $CF_i$ be paid certainly at $t_i$. Then:

$$P_{\text{rf}} = \sum_{i=1}^{N} CF_i \, Z(0, t_i)$$

**Unit check:** $CF_i$ is dollars-per-100, $Z$ is dimensionless, so $P$ is dollars-per-100.

---

### 2.2 Introducing Default Time $\tau$: Survival-Contingent Cashflows

For a defaultable bond that pays $CF_i$ only if $\tau > t_i$, the PV contribution of that cashflow is:

$$E\bigl[Z(0, t_i) \, CF_i \, \mathbf{1}_{\{\tau > t_i\}}\bigr]$$

If we assume deterministic discounting and use survival probability $Q(0, t_i) = P(\tau > t_i)$, a key building block is the price of a **risky zero-coupon bond**:

$$\hat{Z}(0,T) = E\!\left[e^{-\int_0^T r(s)\,ds} \, \mathbf{1}_{\{\tau > T\}}\right]$$

and under independence of interest rates and default, this separates as:

$$\boxed{\hat{Z}(0,T) = Z(0,T) \, Q(0,T)}$$

**Sanity check:** If $Q(0,T) = 1$ (no default risk), then $\hat{Z}(0,T) = Z(0,T)$.

#### The Correlation Correction

The independence assumption is convenient but not always exact. O'Kane shows that when interest rates and hazard rates follow correlated processes, the risky zero-coupon bond price takes the form:

$$\hat{Z}(0, T) = Z(0, T) \cdot Q(0, T) \cdot \Theta(0, T, \rho)$$

where $\Theta$ is a multiplicative correction. For Gaussian dynamics with correlation $\rho$ between rates and hazards:

$$\Theta(0, T, \rho) = \exp\left(\frac{\rho \sigma_r \sigma_\lambda T^3}{3}\right)$$

O'Kane provides numerical examples showing this effect is typically small: "We can see that even for long maturities, the maximum correction to the risky zero coupon bond price is within a few tens of basis points." For a 10-year bond with 50% correlation and typical volatilities, $\Theta \approx 1.007$ (a 0.7% correction). This justifies the market-standard independence assumption for most pricing purposes, though it can matter for long-dated or high-correlation scenarios.

---

### 2.3 Default-Contingent Recovery Cashflow as "Payment at Default"

A standard recovery modeling building block is a claim paying 1 unit at default time if default occurs before $T$. O'Kane gives its PV:

$$\boxed{D(0,T) = E\!\left[e^{-\int_0^\tau r(s)\,ds} \, \mathbf{1}_{\{\tau \leq T\}}\right] = -\int_0^T Z(0,s) \, dQ(0,s)}$$

He notes such payments "occur commonly in bonds as recovery payments."

**Unit check:** $Z$ and $Q$ are dimensionless; the integral is dimensionless, consistent with a PV-per-1-unit-notional.

---

### 2.4 Reduced-Form Risky Coupon Bond PV Under Fractional Recovery of Face Value (RFV)

**Assumption (explicit):** Recovery is a fixed fraction $R$ of face value $F$ paid at default time $\tau$ if $\tau \leq T$ (RFV). This is one of several recovery conventions; see Section 0 and QRM.

Let contractual cashflows be $CF_i$ at $t_i$, and maturity $T = t_N$. Then the bond PV can be written as:

$$\boxed{P = \sum_{i=1}^{N} CF_i \, Z(0, t_i) \, Q(0, t_i) \;+\; RF \cdot D(0,T)}$$

where $D(0,T) = -\int_0^T Z(0,s) \, dQ(0,s)$.

**Derivation Sketch (Step-by-Step):**

1. Decompose payoff into survival and default pieces:
   $$\text{Payoff} = \sum_i CF_i \, \mathbf{1}_{\{\tau > t_i\}} + RF \, \mathbf{1}_{\{\tau \leq T\}}$$

2. Discount and take expectation under the (pricing) measure.

3. Use $E\bigl[e^{-\int_0^{t_i} r} \, \mathbf{1}_{\{\tau > t_i\}}\bigr] = Z(0,t_i) \, Q(0,t_i)$ under independence.

4. Use "payment at default" PV for the recovery term.

**Sanity Checks:**

- If $R = 0$, recovery term vanishes and bond PV is just survival-weighted discounted cashflows.
- If $Q(0,t) = 1$ for all $t$, bond PV reduces to risk-free PV (recovery term becomes 0 because $dQ = 0$).
- **If you need a different recovery convention (RT or RMV):** I'm not sure how your desk defines recovery cashflows without an explicit convention choice. You must specify RT vs RFV vs RMV (and whether coupons/accrual are recovered). QRM explicitly distinguishes these conventions.

---

### 2.5 Z-Spread Repricing Equation

Given a base curve with discount rates $r(0, t_i)$, the Z-spread (ZVS) $z$ solves:

$$P_{\text{mkt}} = \sum_i CF_i \cdot DF\bigl(r(0, t_i) + z;\, t_i\bigr)$$

with the exact compounding form as in O'Kane's ZVS definition (discrete or continuous).

**Practical Interpretation:**

The Z-spread is not "the hazard rate." It is the constant spread that prices the bond; it can embed expected default loss, risk premia, liquidity, etc.

---

### 2.6 Spread Duration and CS01

O'Kane decomposes yield $y = y_T + s$ and expands price changes:

$$\frac{dP}{P} = -D_{r_T} \, dy_T + \frac{1}{2} C_{r_T} \, dy_T^2 - D_s \, ds + \frac{1}{2} C_s \, ds^2 + \cdots$$

For small moves, the first-order approximation is:

$$\frac{dP}{P} \approx -D_s \, ds \quad \Rightarrow \quad \frac{\partial P}{\partial s} \approx -P \, D_s$$

**CS01 Sign Convention (We Will Use):**

In O'Kane's "credit DV01" definition, the measure includes a negative sign so the DV01 is positive for long exposure:

$$\text{credit DV01} = -\bigl(P(s + 1\text{ bp}) - P(s)\bigr)$$

We adopt the analogous convention for bonds:

$$\boxed{CS01 \equiv -\bigl(P(s + 1\text{ bp}) - P(s)\bigr)}$$

so that spread widening (+1 bp) implies $P \downarrow$ and $CS01 > 0$.

**Link to Spread Duration (Unit Check):**

- $D_s$ is dimensionless (interpretable as "years-like" sensitivity).
- Since 1 bp $= 10^{-4}$:

$$CS01\text{ (per 100)} \approx \frac{P \, D_s}{10{,}000}$$

This mirrors the DV01 relation in Tuckman: $DV01 = \frac{PD}{10{,}000}$.

---

## 3. Measurement & Risk (Cash-Credit Analytics)

### 3.A) Pricing Risky Cashflows (Cash Credit Viewpoint)

1. **Start from deterministic PV:** $PV_{\text{rf}} = \sum CF_i \, Z(0, t_i)$

2. **Introduce default timing:** cashflows become contingent on $\tau$.

3. **Separate inputs clearly:**
   - **Base discount curve** $Z(0,t)$: rates/time value of money.
   - **Credit component:**
     - Survival $Q(0,t)$, modeled via hazard $\lambda(t)$ with $Q(0,T) = \exp(-\int_0^T \lambda)$.
     - Recovery convention (RFV/RT/RMV).

**Reduced-Form Pricing Building Blocks (Source-Backed):**

- Under independence, risky ZCB: $\hat{Z}(0,T) = Z(0,T) \, Q(0,T)$
- Payment-at-default PV: $D(0,T) = -\int_0^T Z(0,s) \, dQ(0,s)$

**Bond PV Decomposition (Derived from the Above):**

Survival PV + recovery PV (RFV) as in Section 2.4.

---

### 3.B) Credit Spread Measures for Cash Bonds (Cash-Credit Emphasis)

**Spread Measures Used in THIS Chapter:**

| Measure | Definition |
|---------|------------|
| **Yield spread** $s$ | $y = y_T + s$ |
| **Z-spread** $z$ (ZVS) | Constant spread to the base curve that matches price |
| **Asset swap spread** | Swap-based spread consistent with floating + spread structure |
| **Par floater spread** | Spread that makes an FRN trade at par today |

**"Spread as Quoting Statistic" vs "Spread as Default Intensity + Premia":**

Under a stylized intensity setup, O'Kane presents the **credit triangle approximation**:

$$\boxed{S = \lambda(1-R), \quad \lambda = \frac{S}{1-R}}$$

But O'Kane also emphasizes that observed spreads can include multiple premia; he decomposes the difference between market spread and "actuarial spread" into:
- Default risk premium
- Volatility risk premium
- Liquidity risk premium

(as a conceptual decomposition).

**Practical Takeaway:** Z-spread is a pricing statistic; it should not be equated mechanically to expected loss without acknowledging modeling assumptions and risk premia.

> **Deep Dive: The Spread Zoo**
>
> Which spread should I use?
>
> 1.  **G-Spread (Govt)**: Yield - Treasury Yield. (Quick & Dirty).
> 2.  **I-Spread (Interpolated)**: Yield - Interpolated Swap Rate. (Better).
> 3.  **Z-Spread (Zero-Vol)**: Constant shift to the zero curve. (Standard for pricing).
> 4.  **OAS (Option-Adjusted)**: Z-Spread minus the value of call options. (Essential for Callable Bonds).
> 5.  **ASW (Asset Swap)**: What you pay to swap the bond into floating rate cashflows. (Used by Banks/Hedge Funds).
> 6.  **Par Floater Spread**: What spread an FRN would pay if issued today. (Cleanest credit measure).

---

### 3.B.1) What Credit Spread Contains: The Risk Premium Decomposition

O'Kane presents a framework for decomposing credit spreads into their economic components. The **market spread** exceeds the **actuarial spread** (expected loss) by a margin that reflects risk compensation.

**Actuarial Spread:**

The actuarial spread is the spread that would compensate exactly for expected loss under risk-neutral probabilities. Using the credit triangle:

$$S_{\text{actuarial}} = \lambda_{\text{actual}}(1-R)$$

where $\lambda_{\text{actual}}$ is the historical (not risk-neutral) default intensity.

**Credit Risk Premium Decomposition:**

O'Kane decomposes the excess of market spread over actuarial spread into three components:

$$\boxed{S_{\text{market}} = S_{\text{actuarial}} + \underbrace{\Pi_{\text{default}} + \Pi_{\text{volatility}} + \Pi_{\text{liquidity}}}_{\text{Credit Risk Premium}}}$$

| Component | Description |
|-----------|-------------|
| **Default Risk Premium** $\Pi_{\text{default}}$ | Compensation for bearing the systematic component of default risk that cannot be diversified away |
| **Volatility Risk Premium** $\Pi_{\text{volatility}}$ | Compensation for uncertainty in the timing and amount of defaults |
| **Liquidity Risk Premium** $\Pi_{\text{liquidity}}$ | Compensation for illiquidity—difficulty selling in stressed markets |

**Coverage Ratio:**

O'Kane defines the **coverage ratio** as the ratio of market spread to actuarial spread:

$$\boxed{\text{Coverage Ratio} = \frac{S_{\text{market}}}{S_{\text{actuarial}}}}$$

This tells you "how many times over" the market charges for expected loss. O'Kane provides empirical evidence using 2002 CDS data:

| Rating | 5Y Avg Spread (bp) | Actuarial Spread (bp) | Coverage Ratio | Spread Premium (bp) |
|--------|-------------------|----------------------|----------------|---------------------|
| AA | 28 | 9 | 3.12 | 19 |
| A | 61 | 13 | 4.67 | 48 |
| BBB | 164 | 30 | 5.54 | 134 |
| BB | 463 | 145 | 3.19 | 318 |

O'Kane observes: "We see that the coverage ratio tends to remain fairly constant as a function of rating... The spread premium increases as we descend the rating spectrum."

This means that inferring hazard rates directly from spreads (via $\lambda = s/(1-R)$) yields **risk-neutral** default intensities that exceed historical rates. Hull reinforces this: "The default probabilities or hazard rates implied from credit spreads are risk-neutral estimates."

Hull's Table 24.3 shows the expected excess return on bonds, further illustrating the wedge between market spreads and historical default experience:

| Rating | Bond Yield Spread (bp) | Spread for Historical Defaults (bp) | Excess Return (bp) |
|--------|----------------------|--------------------------------------|---------------------|
| Aaa | 40 | 2 | 38 |
| A | 77 | 8 | 69 |
| Baa | 143 | 28 | 115 |
| Ba | 304 | 144 | 160 |

**Practical implication:** When calibrating survival curves to bond or CDS spreads, you are obtaining $\mathbb{Q}$-measure (pricing) hazards, not $\mathbb{P}$-measure (forecasting) hazards. These are appropriate for pricing and hedging but not for predicting actual default rates.

**Interpreting Coverage Ratios:**

O'Kane's data shows that coverage ratios for investment grade (AA through BBB) range from about 3× to 5.5×, while high-yield (BB) drops to about 3×. For investment-grade credits, the expected loss is small but spreads remain material—the risk premium is relatively large because:
1. Defaults are rare but catastrophic when they occur (jump-to-default risk)
2. IG defaults tend to cluster in recessions (systematic risk)
3. IG bonds are held by investors who value liquidity highly

For high-yield credits, expected loss is larger and the premium shrinks as a multiple—though the absolute spread premium (318 bp for BB) exceeds that of IG names.

> **Desk Reality: What You're Really Buying**
>
> When a portfolio manager says "I'm picking up 200 bp in BBB credit," they're not just picking up 200 bp of expected loss. With a coverage ratio of ~4x, perhaps 50 bp compensates for expected defaults and 150 bp is risk premium.
>
> **Why this matters:** In a recession when defaults spike, the expected loss component may turn out to have been underestimated. But the risk premium component is your compensation for bearing exactly that scenario.

### 3.B.2) The Merton Model Perspective

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

Reduced-form models (intensity-based) better fit short-dated spread behavior and dominate in the credit derivatives market. This is why the reduced-form framework developed in Sections 2.1–2.4 above is the standard tool for cash credit analytics.

---

### 3.B.3) Distressed Credits and Upfront Trading

**When Does the Market Switch to Upfront?**

For highly distressed credits (typically spreads above 1000 bp), the market switches from **running spread** (periodic premium payments) to **upfront** format. O'Kane explains this occurs because:

1. **Protection seller preference:** At wide spreads, the seller prefers a certain upfront payment now rather than a risky stream of premiums that stops at default
2. **Payment timing:** The risk of default before receiving much premium is high
3. **Market convention:** CDS on distressed names trade with a standardized "standard coupon" (typically 500 bp) plus upfront

**Upfront Valuation:**

O'Kane provides the formula relating upfront payment $U$ to running spread $S$:

$$\boxed{U = (S - S_{\text{std}}) \times RPV01}$$

where $S_{\text{std}}$ is the standard coupon (e.g., 500 bp) and $RPV01$ is the risky PV01.

**Inverted Credit Curves:**

Distressed credits often exhibit **inverted credit curves**—short-term spreads higher than long-term spreads. This seems counterintuitive (shouldn't more time mean more default risk?), but reflects the market's view that:
- Near-term default probability is high
- If the issuer survives the near term, it may recover and be less risky

O'Kane shows examples where 1-year CDS spreads exceed 5-year spreads for distressed names.

**Equity-Like Convexity:**

Distressed bonds exhibit **positive convexity** in a way that resembles equity:
- If the issuer defaults, bondholders lose (but are capped at losing face minus recovery)
- If the issuer recovers, the bond can rally dramatically—potentially from $30 back toward par

This asymmetric payoff makes distressed bond analysis resemble equity analysis more than investment-grade credit analysis.

> **Practitioner Note: Pull-to-Par vs. Pull-to-Recovery**
>
> Normal bonds "pull to par" as they approach maturity—premium bonds decline and discount bonds rise.
>
> Distressed bonds "pull toward recovery"—if the market expects default, the bond price gravitates toward the expected recovery value (often 30-50 cents on the dollar), not par.
>
> This creates a very different carry dynamic. A distressed bond trading at $40 with expected recovery of $35 may have **negative carry**—it drifts down toward recovery even as coupons are paid.

---

### 3.C) Credit DV01 / Spread Duration / CS01

**Definitions (Precise and Bump-Specific):**

**Spread duration** $D_s$: from O'Kane's yield-spread Taylor expansion:

$$\frac{dP}{P} \approx -D_s \, ds$$

**CS01 / credit DV01** (our chapter convention):

$$CS01 \equiv -\bigl(P(s + 1\text{ bp}) - P(s)\bigr)$$

matching O'Kane's sign convention for credit DV01.

**What Is Being Bumped? (Be Explicit)**

| Bump Type | Description |
|-----------|-------------|
| **Constant Z-spread bump** | Hold $Z(0,t)$ fixed and reprice with $z \to z + \Delta z$. This matches the ZVS definition equation. |
| **Hazard-rate bump** (preview only) | Bump $\lambda(t)$ (or survival $Q$) and reprice using a recovery convention. The link between hazard and spread can be approximated by the credit triangle $S = \lambda(1-R)$ under strong simplifications. |
| **OAS bump** (not pursued here) | O'Kane notes that for bonds with embedded options, the option-adjusted spread (OAS) is used rather than ZVS for a "correct" valuation. We keep this chapter to non-option cash bonds. |

**Risk Interpretation:**

**Rates DV01 vs CS01:**
- **Rates DV01** measures PV sensitivity to the base curve/rates.
- **CS01** measures PV sensitivity to the chosen credit spread measure (e.g., Z-spread).

A rates-hedged corporate bond position (DV01-neutral via swaps/Treasuries) can still lose money when credit spreads widen (CS01 exposure remains).

**Additivity:**

PV is additive across instruments, so first-order sensitivities in dollars add. Tuckman shows portfolio duration as a value-weighted average of component durations, and DV01 scales with price and duration.

Hence, under a common bump definition, **portfolio DV01** and **portfolio CS01** are the (signed) sums of position DV01/CS01.

---

### 3.D) Pull-to-Par in Credit Context

**Normal Pull-to-Par:**

For risk-free bonds, Tuckman explains that a bond trading above par (premium bond) will drift down toward par as it ages, while a bond trading below par (discount bond) will drift up. This is pure time value—as the bond approaches maturity, the price converges to the redemption amount (par).

**Credit Pull-to-Par Dynamics:**

For credit bonds, the same mechanics apply BUT are modified by credit risk:

1. **Surviving bonds still pull to par:** If the issuer survives to maturity, the bond pays par. So conditional on survival, pull-to-par works normally.

2. **Survival probability declines:** As each day passes without default, the remaining survival probability for the next period is reassessed. For stable credits, this has minimal effect.

3. **Spread roll-down:** If the credit curve is upward-sloping (normal), a 5-year bond "rolling" to 4-year maturity experiences spread compression, adding to returns (see Chapter 7 on carry and rolldown).

**Credit Carry:**

The "carry" trade in credit involves:
- Collecting spread income (coupon above risk-free rate)
- Hoping for no default or rating downgrade
- Benefiting from spread roll-down if curves are upward-sloping

> **Desk Reality: The Carry Trade**
>
> A classic trade in high-yield is simply owning the bonds and collecting spread. The math: if you buy a BB bond at +350 bp and hold it for a year with no default and unchanged spreads, you earn roughly 3.5% excess return.
>
> **The risk:** If defaults spike or spreads widen, you lose more than a year's carry in a day. The 2008 crisis saw high-yield spreads widen from ~300 bp to ~2000 bp in months—wiping out years of accumulated carry.

---

### 3.E) The Cash-CDS Basis: Comprehensive Treatment

The **cash-CDS basis** measures the difference between credit risk pricing in the cash bond market versus the CDS market. O'Kane defines:

$$\boxed{\text{CDS Basis} = S_{\text{CDS}} - S_{\text{Bond,Libor}}}$$

where $S_{\text{Bond,Libor}}$ is the bond's spread measured on a Libor basis (typically the par floater spread or asset swap spread).

**Why Should They Be Equal (In Theory)?**

Under idealized conditions, buying a risky bond and buying CDS protection should create a risk-free position. If so, the bond spread and CDS spread should match (after adjusting for funding at Libor). But in practice, many factors cause the basis to deviate from zero.

#### 3.E.1) Fundamental Factors Affecting the Basis

O'Kane identifies six **fundamental factors** that create structural differences between bond and CDS pricing:

**1. Funding (Unfunded vs. Funded)**

| Product | Funding |
|---------|---------|
| Bond | **Funded**: You must pay the bond price upfront |
| CDS | **Unfunded**: No initial investment (except margin) |

A bond investor must finance the purchase, typically in repo. If funding costs exceed Libor, bond spreads should be higher than CDS spreads (negative basis). If funding is below Libor (e.g., for a strong bank), bond spreads can be lower (positive basis).

**2. Delivery Option in CDS**

CDS protection buyers have a delivery option: they can deliver the **cheapest-to-deliver** bond from a basket of deliverable obligations. This option has value, especially when:
- Multiple bonds with different prices exist
- Some bonds trade cheaper due to liquidity
- Restructuring events allow broader deliverables

The delivery option makes CDS protection more valuable, pushing CDS spreads higher relative to bonds (positive basis contribution).

**3. Technical Default (Broader Credit Events)**

CDS credit events can be triggered by events that don't necessarily trigger bond default:
- **Restructuring:** CDS may pay out on restructuring even if bonds continue to pay
- **Failure to pay:** May trigger CDS before formal bankruptcy

This makes CDS protection more comprehensive, pushing CDS spreads higher (positive basis contribution).

**4. Loss on Default (Par vs. Off-Par Bonds)**

CDS protection pays $(1 - R) \times \text{Notional}$, assuming you bought protection on par notional. But:
- A bond trading at $80 loses $(1-R) \times 80$ at default
- A bond trading at $120 loses $(1-R) \times 120$ at default

For discount bonds (price < 100), CDS provides **more** protection than needed—positive basis contribution.
For premium bonds (price > 100), CDS provides **less** protection than needed—negative basis contribution.

**5. Premium Accrued at Default**

When a CDS default occurs between premium payment dates:
- **CDS:** Protection buyer must pay accrued premium
- **Bond:** Holder loses accrued coupon (if the issuer stops paying)

The accrued premium payment slightly reduces CDS value—minor negative basis contribution.

**6. CDS Spreads Cannot Be Negative**

Bond spreads can be negative (e.g., supranational bonds trading through Libor), but CDS spreads have a zero floor. This creates asymmetry for very high-quality issuers.

#### 3.E.2) Market Factors Affecting the Basis

O'Kane identifies six **market factors** that cause the basis to vary over time:

**1. Relative Liquidity**

Whichever market is more liquid tends to be "fairer" priced. In normal times, the CDS market for large issuers is often more liquid than cash bonds, making CDS spreads the "true" credit measure. The less liquid market trades wider.

**2. Synthetic CDO Technicals**

Structured products like synthetic CDOs require large amounts of CDS protection. Dealers who create CDOs need to buy protection, pushing CDS spreads wider (positive basis). This was significant in 2005-2007.

**3. New Issuance / Loan Hedging**

When companies issue new bonds or loans, underwriters often hedge by buying CDS protection. This temporarily widens CDS spreads (positive basis).

**4. Convertible Bond Issuance**

Convertible bond arbitrage strategies involve:
- Long convertible bond (for the equity option)
- Hedge credit risk by buying CDS protection

Large convertible issuance creates CDS protection demand, widening CDS spreads (positive basis).

**5. Demand for Protection (Short Positioning)**

CDS is the easiest way to go short credit. When the market is bearish:
- Investors buy CDS protection to short credit
- Cash bond shorting is harder (need to borrow bonds)

One-sided protection demand widens CDS spreads (positive basis).

**6. Funding Risk (CDS Locks in Libor Flat)**

CDS premium payments are fixed; the contract doesn't change if funding costs change. Bonds require ongoing financing that may become expensive in stress. This makes CDS more attractive to protection sellers in volatile times.

#### 3.E.3) Basis Regimes: Positive vs. Negative

| Regime | When | Why |
|--------|------|-----|
| **Positive Basis** (CDS > Bond) | Pre-2007 "normal" times | Synthetic CDO demand, delivery option, short demand via CDS |
| **Negative Basis** (Bond > CDS) | 2008-2009 crisis | Funding stress, forced bond selling, cash market dislocation |

Hull RM provides historical context: "Prior to the market turmoil starting in 2007, the basis tended to be positive. For example, De Witt estimates that the average CDS-bond basis in 2004 and 2005 was 16 basis points." During the 2008 crisis, the basis became severely negative as bond prices collapsed while CDS protection became expensive. Hull RM explains: "It was difficult for financial institutions to arbitrage between bonds and CDSs because of a shortage of liquidity and other considerations." Since the crisis, Hull notes that "the magnitude of the CDS-bond basis (sometimes positive and sometimes negative) has become much smaller."

> **Desk Reality: The "Free Money" Illusion**
>
> A negative basis looks like free money: buy the bond, buy CDS protection, earn the basis. But:
>
> 1. **Funding cost:** The trade requires financing the bond position. During the 2008 crisis, repo rates spiked and financing became unavailable at any price.
>
> 2. **Counterparty risk:** If your CDS counterparty defaults, you lose your protection just when you need it most.
>
> 3. **Margin calls:** Both bond repo and CDS require margin. If spreads widen, you face margin calls before the trade converges.
>
> 4. **Term mismatch:** Bond repo is typically short-term; CDS is 5-year. If repo markets freeze, you're forced to sell the bond at distressed prices.
>
> Many "negative basis" trades lost money in 2008-2009 despite the theoretical arbitrage.

#### 3.E.4) Basis Driver Summary Table

| Factor | Direction | Mechanism |
|--------|-----------|-----------|
| **Fundamental Factors** | | |
| Funding cost > Libor | Negative basis | Bonds require funding |
| Delivery option value | Positive basis | CDS buyer can deliver CTD |
| Technical default (restructuring) | Positive basis | CDS credit events broader |
| Discount bonds | Positive basis | CDS overprotects |
| Premium bonds | Negative basis | CDS underprotects |
| Accrued premium at default | Negative basis | CDS buyer pays accrued |
| **Market Factors** | | |
| CDO protection demand | Positive basis | Technical CDS buying |
| New issuance hedging | Positive basis | Underwriters buy CDS |
| Convertible arb activity | Positive basis | Arbs buy CDS |
| One-sided short demand | Positive basis | CDS easier to short |
| Funding market stress | Negative basis | Cash bonds dislocate |
| Relative liquidity | Varies | Less liquid market trades wider |

---

### 3.F) Hedging Mindset (Preview-Level, Not a CDS Chapter)

**Conceptual Hedges for Cash Credit Spread Risk:**

- **With other bonds:** relative value hedges (long one bond, short another) to isolate curve/spread shape.
- **With indices or CDS (preview only):** hedge spread risk using CDS indices or single-name CDS; detailed CDS valuation comes later.

**Basis Risk:**

RMFI notes a common relationship: bond yield spread is expected to be approximately equal to the CDS spread (under simplifying assumptions), and discusses CDS-bond basis concepts.

In practice, cash-bond spreads and CDS spreads can diverge (funding, liquidity, delivery options, etc.). Treat this as **basis risk**.

---

## 4. Worked Examples (15 Numeric Examples)

### Common Conventions Across Examples A–F (Unless Stated)

- **Prices** are dirty/full per 100 (settlement at a coupon date $\Rightarrow AI = 0$). Clean = dirty. $P = P_{\text{clean}} + AI$.
- **Coupon frequency** $f = 2$ (semiannual). Times $t_n = n/2$.
- **Base curve** given by spot rates $r(0, t_n)$ compounded semiannually; discount factor:
  $$Z(0, t_n) = \frac{1}{\left(1 + \frac{r(0, t_n)}{2}\right)^n}$$
- **Z-spread** $z$ is added to each spot rate before discounting (ZVS notion).
- 1 bp $= 0.0001$ in decimal.

---

### Example A: Risk-Free PV Baseline

**Task:** Price a fixed-rate bond from a given risk-free discount curve.

**Bond A (Cash Bond):**

| Parameter | Value |
|-----------|-------|
| Face $F$ | 100 |
| Coupon $c$ | 6% annual, semiannual coupon = 3.00 |
| Maturity $T$ | 3 years $\Rightarrow N = 6$ coupons |
| Cashflows | $CF_1 = \cdots = CF_5 = 3$, $CF_6 = 103$ |

**Base Curve Spot Rates and Discount Factors:**

| $n$ | $t_n$ | $r(0, t_n)$ | $Z(0, t_n)$ |
|-----|-------|-------------|-------------|
| 1 | 0.5 | 2.00% | 0.9900990 |
| 2 | 1.0 | 2.20% | 0.9783577 |
| 3 | 1.5 | 2.35% | 0.9655624 |
| 4 | 2.0 | 2.50% | 0.9515243 |
| 5 | 2.5 | 2.60% | 0.9374601 |
| 6 | 3.0 | 2.70% | 0.9226935 |

**PV Contributions (Per 100):**

| $n$ | $CF_n$ | $Z(0, t_n)$ | $PV_n = CF_n \cdot Z$ |
|-----|--------|-------------|----------------------|
| 1 | 3 | 0.9900990 | 2.9702970 |
| 2 | 3 | 0.9783577 | 2.9350732 |
| 3 | 3 | 0.9655624 | 2.8966873 |
| 4 | 3 | 0.9515243 | 2.8545728 |
| 5 | 3 | 0.9374601 | 2.8123802 |
| 6 | 103 | 0.9226935 | 95.0374350 |

**Risk-Free Price:**

$$P_{\text{rf}} = \sum_{n=1}^{6} PV_n = 109.5064$$

**Sanity check:** Coupon 6% is far above the ~2–3% curve $\Rightarrow$ price above par, consistent.

---

### Example B: Introduce a Survival Curve

**Task:** Given survival $Q(0,t)$ at coupon dates and recovery, compute risky PV (cash credit viewpoint).

Use the same Bond A.

**Survival Curve (Risk-Neutral Survival Used for Pricing):**

$$Q(0, t_n) = \{0.995,\, 0.990,\, 0.984,\, 0.977,\, 0.969,\, 0.960\}$$

**Recovery:** RFV with $R = 40\%$ of face, paid at default (illustrative discretization using the "payment at default" building block).

**Step 1: Survival PV of Contractual Cashflows**

$$PV_{\text{surv}} = \sum_{n=1}^{6} CF_n \, Z(0, t_n) \, Q(0, t_n)$$

| $n$ | $CF_n$ | $Z(0, t_n)$ | $Q(0, t_n)$ | $CF \cdot Z \cdot Q$ |
|-----|--------|-------------|-------------|----------------------|
| 1 | 3 | 0.9900990 | 0.995 | 2.9554455 |
| 2 | 3 | 0.9783577 | 0.990 | 2.9057225 |
| 3 | 3 | 0.9655624 | 0.984 | 2.8503403 |
| 4 | 3 | 0.9515243 | 0.977 | 2.7889177 |
| 5 | 3 | 0.9374601 | 0.969 | 2.7251964 |
| 6 | 103 | 0.9226935 | 0.960 | 91.2359376 |

**Sum:**

$$PV_{\text{surv}} = 105.4616$$

**Step 2: Recovery PV (RFV)**

Using $D(0,T) = -\int_0^T Z(0,s) \, dQ(0,s)$, a discrete approximation with default mass in intervals $(t_{n-1}, t_n]$ is:

$$PV_{\text{rec}} \approx RF \sum_{n=1}^{6} Z(0, t_n) \bigl(Q(0, t_{n-1}) - Q(0, t_n)\bigr), \quad Q(0, t_0) = 1$$

(Interpretation: expected discounted recovery paid at the end of the interval; this is an illustrative discretization consistent with the "payment at default" building block.)

**Default Probabilities Per Interval:**

$$\Delta Q_n = Q_{n-1} - Q_n = (0.005,\, 0.005,\, 0.006,\, 0.007,\, 0.008,\, 0.009)$$

**Compute $\sum Z \, \Delta Q$:**

| Term | Calculation |
|------|-------------|
| $0.9900990 \cdot 0.005$ | $= 0.0049505$ |
| $0.9783577 \cdot 0.005$ | $= 0.0048918$ |
| $0.9655624 \cdot 0.006$ | $= 0.0057934$ |
| $0.9515243 \cdot 0.007$ | $= 0.0066607$ |
| $0.9374601 \cdot 0.008$ | $= 0.0074997$ |
| $0.9226935 \cdot 0.009$ | $= 0.0083042$ |

**Sum:**

$$\sum Z \, \Delta Q = 0.0381003$$

**Thus:**

$$PV_{\text{rec}} \approx 0.40 \cdot 100 \cdot 0.0381003 = 1.5240$$

**Step 3: Risky Bond PV**

$$P_{\text{risky}} \approx PV_{\text{surv}} + PV_{\text{rec}} = 105.4616 + 1.5240 = 106.9856$$

**Sanity Checks:**

- $P_{\text{risky}} < P_{\text{rf}}$ (default risk reduces value): $106.99 < 109.51$. ✓
- Recovery PV is small here because cumulative default probability is only 4% by $T$.

---

### Example C: Risky Yield vs Risk-Free Yield

**Task:** Compute YTM for the risky bond price and compare to risk-free yield; interpret the yield spread.

Yield is defined as the internal rate of return that equates PV of promised cashflows to price.

We solve $y$ (nominal annual, comp semiannually) from:

$$P = \sum_{n=1}^{5} \frac{3}{(1 + y/2)^n} + \frac{103}{(1 + y/2)^6}$$

**Risk-Free YTM for $P = 109.5064$:**

- At $y = 2.68\%$: $P(y) \approx 109.5091$
- At $y = 2.69\%$: $P(y) \approx 109.4788$
- Interpolate $\Rightarrow y_{\text{rf}} \approx 2.68\%$

**Risky YTM for $P = 106.9856$:**

- At $y = 3.52\%$: $P(y) \approx 107.0024$
- At $y = 3.526\%$: $P(y) \approx 106.9847$
- Interpolate $\Rightarrow y_{\text{risky}} \approx 3.526\%$

**Yield Spread:**

$$s_{\text{YTM}} \approx y_{\text{risky}} - y_{\text{rf}} \approx 3.526\% - 2.68\% = 0.846\% \approx 84.6\text{ bp}$$

**Interpretation (Desk-Relevant):**

This yield spread is a summary statistic. It does not cleanly separate expected default loss, liquidity, or risk premia. O'Kane explicitly discusses decomposing market spread into actuarial spread + various premia.

---

### Example D: Z-Spread Solve

**Task:** Given base curve and market dirty price, solve for Z-spread $z$ that reprices the bond.

Use O'Kane's ZVS definition (discrete compounding version).

**Inputs:**

- Bond A cashflows as before.
- Base curve spot rates $r(0, t_n)$ as in Example A.
- Market price: $P_{\text{mkt}} = 106.9856$ (use Example B price as "market").

**Pricing Function Under Z-Spread $z$:**

$$P(z) = \sum_{n=1}^{5} \frac{3}{\left(1 + \frac{r(0, t_n) + z}{2}\right)^n} + \frac{103}{\left(1 + \frac{r(0, t_6) + z}{2}\right)^6}$$

**Iteration / Bracketing:**

- Try $z_0 = 80$ bp $= 0.0080$:
  $$P(80\text{ bp}) \approx 107.1183$$

- Try $z_1 = 100$ bp $= 0.0100$:
  $$P(100\text{ bp}) \approx 106.5312$$

Target $106.9856$ lies between $\Rightarrow z \in (80, 100)$ bp.

**Linear Interpolation (One Explicit Step):**

$$z \approx 80\text{ bp} + \frac{107.1183 - 106.9856}{107.1183 - 106.5312} \cdot (20\text{ bp}) \approx 84.5\text{ bp}$$

**Repricing Check:**

At $z = 84.5$ bp, $P(z) \approx 106.9858$ (matches $106.9856$ within rounding). ✓

---

### Example E: Spread Duration / CS01 from Finite Differences

**Task:** Reprice at $z \pm 1$ bp and compute CS01 and spread duration.

Let $z = 84.5$ bp (Example D). Compute:

| Price | Value |
|-------|-------|
| $P(z)$ | $\approx 106.9858$ |
| $P(z + 1\text{ bp})$ | $\approx 106.9564$ |
| $P(z - 1\text{ bp})$ | $\approx 107.0152$ |

**CS01 (Per 100) Using O'Kane-Style Sign Convention:**

$$CS01 \equiv -\bigl(P(z + 1\text{ bp}) - P(z)\bigr) = P(z) - P(z + 1\text{ bp}) \approx 0.029403$$

**Unit check:** price points per 100 per 1 bp.

**Per \$1mm Notional:**

$$CS01_{\$} \approx \frac{1{,}000{,}000}{100} \cdot 0.029403 \approx \$294.03\text{ per bp}$$

**Spread Duration (Central Difference):**

$$\frac{\partial P}{\partial z} \approx \frac{P(z + 1\text{ bp}) - P(z - 1\text{ bp})}{2 \cdot 0.0001}$$

$$D_s \approx -\frac{1}{P(z)} \frac{\partial P}{\partial z} \approx 2.7488$$

**Consistency Check with DV01-Style Scaling:**

$$\frac{P \, D_s}{10{,}000} \approx \frac{106.9858 \cdot 2.7488}{10{,}000} \approx 0.0294 \approx CS01\text{ (per 100)}$$

This mirrors Tuckman's DV01 scaling. ✓

---

### Example F: Rates DV01 vs CS01

**Task:** For the same bond, compute:
1. Rates DV01 under a +1 bp parallel bump to the base curve (define bump),
2. CS01 under +1 bp Z-spread widening, compare magnitudes.

**Define the Rate Bump:** Increase each base curve spot rate by 1 bp:

$$r(0, t_n) \to r(0, t_n) + 0.0001$$

hold $z$ fixed at 84.5 bp.

**Reprice:**

| Scenario | Price |
|----------|-------|
| Base | $P \approx 106.9858$ |
| Rates +1 bp | $P_{\text{rates}+} \approx 106.9564$ |

**Rates DV01 (Per 100)**

Using DV01 sign convention (positive for a long bond):

$$DV01 \equiv -\bigl(P_{\text{rates}+} - P\bigr) = P - P_{\text{rates}+} \approx 0.029403$$

**Per \$1mm:** $\approx \$294.03$/bp.

**Compare to CS01 (Example E):**

| Risk Measure | Value |
|--------------|-------|
| DV01 | $\approx \$294$/bp |
| CS01 | $\approx \$294$/bp |

**Interpretation (Important Even When Magnitudes Match):**

Even if the numerical sensitivities are similar in this toy Z-spread setup, they correspond to **different risk factors**:

- **DV01** hedged with swaps/Treasuries (rates instruments),
- **CS01** hedged with credit instruments (other cash bonds, CDS/indices), and basis risk can remain.

This illustrates O'Kane's point: for a fixed-rate corporate bond, spread duration equals interest rate duration. The bond is "as much an interest rate play as it is a credit play."

---

### Example G: Recovery Sensitivity

**Task:** Show how recovery assumptions change (i) price via recovery PV and/or (ii) implied hazard/spread (toy).

**(G1) Hold Survival Curve Fixed; Vary Recovery (RFV)**

From Example B, $\sum Z \, \Delta Q = 0.0381003$. Recovery PV under RFV:

$$PV_{\text{rec}}(R) = R \cdot 100 \cdot 0.0381003$$

| Recovery $R$ | $PV_{\text{rec}}$ |
|--------------|-------------------|
| 40% | 1.5240 |
| 20% | 0.7620 |

**Price Change (Holding Survival Fixed):**

$$\Delta P \approx 0.7620\text{ points per 100}$$

**(G2) Hold Market Spread Fixed; Infer Hazard (Credit Triangle Toy)**

O'Kane's credit triangle: $S = \lambda(1-R)$

Take market spread $S = z = 84.5$ bp $= 0.00845$.

| Recovery $R$ | $1 - R$ | Implied $\lambda$ |
|--------------|---------|-------------------|
| 40% | 0.60 | $\lambda \approx \frac{0.00845}{0.60} = 0.014083 \approx 1.41\%$/yr |
| 20% | 0.80 | $\lambda \approx \frac{0.00845}{0.80} = 0.010563 \approx 1.06\%$/yr |

**Interpretation:** Recovery assumptions materially affect implied default intensity; you must state the recovery convention (RFV/RT/RMV).

---

### Example H: Portfolio Additivity

**Task:** Two-bond portfolio with different CS01 and DV01. Compute portfolio DV01 and CS01 as signed sums; show a DV01-neutral hedge where CS01 remains.

**Bond 1: Corporate Bond A (Examples D–F)**

| Parameter | Value |
|-----------|-------|
| Notional $N_C$ | \$1.0mm |
| $DV01_C$ | $\approx \$294.03$/bp |
| $CS01_C$ | $\approx \$294.03$/bp |

**Bond 2: Treasury Hedge Bond T (Risk-Free; CS01 = 0 by Definition)**

| Parameter | Value |
|-----------|-------|
| Face | 100 |
| Coupon | 3% annual (1.5 semiannual) |
| Maturity | 5y (10 periods) |

- Base-curve price: $P_T \approx 99.4359$
- Base curve +1 bp repriced price: $P_{T,\text{rates}+} \approx 99.3901$

**Thus:**

$$DV01_T \approx P_T - P_{T,\text{rates}+} = 0.045770\text{ per 100} \Rightarrow DV01_{T,\$1\text{mm}} \approx \$457.70/\text{bp}$$

$$CS01_T \approx 0$$

**Hedge Ratio for DV01-Neutral Portfolio**

Choose Treasury notional $N_T$ short such that:

$$DV01_{\text{port}} = DV01_C - N_T \cdot DV01_T = 0 \Rightarrow N_T = \frac{294.03}{457.70} \approx 0.6424\text{ mm}$$

**Portfolio Risks:**

| Risk | Calculation | Result |
|------|-------------|--------|
| DV01 | $294.03 - 0.6424 \cdot 457.70$ | $\approx 0$ |
| CS01 | $294.03 - 0.6424 \cdot 0$ | $= 294.03$ |

**Message:** A rates-hedged corporate bond can still have meaningful credit spread risk.

---

### Example I: Credit Curve Steepening/Flattening Preview

**Task:** Two bonds of same issuer different maturities and spreads. Build a tiny "spread curve" and compute P&L for a "credit steepener" scenario (cash-credit, simple).

**Bond A (3y):** $z_A = 84.5$ bp, $CS01_A \approx \$294$/bp per \$1mm.

**Bond B (5y, Same Issuer, Higher Spread):** Take a 5y corporate bond (coupon 4%) priced at $z_B = 120$ bp:
- Price $P_B \approx 98.6032$
- $CS01_B \approx \$441.33$/bp per \$1mm (computed by $z \pm 1$ bp finite difference).

**Tiny Spread Curve:** $(3y, 84.5\text{ bp}),\, (5y, 120\text{ bp})$

**Trade ("Steepener"):** Long \$1mm of 3y bond A, short \$1mm of 5y bond B.

**Scenario:** Short spread tightens by 10 bp, long spread widens by 20 bp:
- $\Delta z_A = -10$ bp
- $\Delta z_B = +20$ bp

**Use First-Order P&L Approximation for a Long Position:**

$$\Delta PV \approx -CS01 \cdot \Delta z_{\text{bp}}$$

For a short position, flip the sign.

**P&L on Long A:**

$$\Delta PV_A \approx -294 \cdot (-10) = +\$2{,}940$$

**P&L on Short B:**

- Long B would have $\Delta PV_{B,\text{long}} \approx -441.33 \cdot (+20) = -\$8{,}826.6$
- Short B $\Rightarrow \Delta PV_B \approx +\$8{,}826.6$

**Total P&L:**

$$\Delta PV_{\text{total}} \approx 2{,}940 + 8{,}826.6 = \$11{,}766.6$$

**Preview Note:** This ignores rates changes and any bond–CDS basis behavior.

---

### Example J: What the Spread Contains — Toy Decomposition

**Task:** Construct an expected-loss-only spread from hazard + recovery and compare to market Z-spread; interpret difference as "risk premium + liquidity/funding" (interpretation only).

**From Example B:**

$Q(0,3) = 0.960$ $\Rightarrow$ cumulative default probability by 3y is 4%.

**Toy Hazard Estimate (Simple Average):**

$$\lambda_{\text{avg}} \approx \frac{1 - Q(0,3)}{3} = \frac{0.04}{3} = 0.01333 \approx 1.33\%/\text{yr}$$

(Interpretation: crude average intensity; a more exact mapping uses $Q = \exp(-\int \lambda)$.)

**Expected-Loss-Only ("Actuarial") Spread via Credit Triangle:**

$$S_{\text{EL}} \approx \lambda_{\text{avg}}(1-R) = 0.01333 \cdot 0.60 = 0.00800 = 80.0\text{ bp}$$

**Market Z-Spread:**

From Example D: $z_{\text{mkt}} \approx 84.5$ bp.

**Difference:**

$$z_{\text{mkt}} - S_{\text{EL}} \approx 4.5\text{ bp}$$

**Coverage Ratio:**

$$\text{Coverage Ratio} = \frac{z_{\text{mkt}}}{S_{\text{EL}}} = \frac{84.5}{80.0} = 1.056$$

This is low compared to typical IG coverage ratios (4-5x), suggesting either the actuarial spread calculation is overstated or this is a relatively risky credit where expected loss dominates.

**Interpretation (Explicitly Labeled):** O'Kane conceptualizes the difference between market spread and actuarial spread as arising from additional premia (default risk premium, volatility risk premium, liquidity risk premium).

Do not treat this as an identity; it is a useful way to frame why "spread ≠ expected loss."

---

### Example K: FRN Pricing and Par Floater Spread

**Task:** Price a floating rate note and compute the par floater spread.

**FRN Characteristics:**

| Parameter | Value |
|-----------|-------|
| Maturity | 5 years |
| Coupon | 3M Libor + 150 bp (quoted margin) |
| Face | 100 |
| Current 3M Libor | 3.00% |

**Assume:**
- Flat Libor curve at 3.00%
- Credit spread has widened since issuance; current par floater spread is 200 bp

**FRN Price Approximation:**

$$P_{\text{FRN}} \approx 100 - (F(t) - F(0)) \times RPV01$$

For a 5-year FRN with Libor discounting, $RPV01 \approx 4.5$ years.

$$P_{\text{FRN}} \approx 100 - (200 - 150) \times 0.045 = 100 - 2.25 = 97.75$$

**Interpretation:** The FRN trades at a discount because credit has deteriorated—the market now requires 200 bp over Libor, but this FRN only pays 150 bp.

**Credit DV01 (CS01):**

$$CS01 \approx RPV01 = 4.5 \times 100 / 10{,}000 = 0.045 \text{ per 100 per bp}$$

Per \$10mm notional: $CS01 \approx \$4{,}500$ per bp.

**Comparison to Fixed-Rate Bond:**

A 5-year fixed-rate bond with 7-year duration might have:
- Rate DV01: $\approx 0.07$ per 100 per bp
- CS01: $\approx 0.07$ per 100 per bp (spread duration = rate duration)

The FRN has:
- Rate DV01: $\approx 0$ (floating rate resets)
- CS01: $\approx 0.045$ per 100 per bp

**Key insight:** FRNs have credit duration comparable to their maturity, even though rate duration is near zero.

---

### Example L: Par Asset Swap Spread Calculation

**Task:** Compute the par asset swap spread for a corporate bond.

**Bond Characteristics:**

| Parameter | Value |
|-----------|-------|
| Maturity | 7 years |
| Coupon | 7.25% (annual) |
| Price (dirty) | 94.38 |
| Yield | 8.25% |

**Swap Rates:**
- 7-year swap rate: 6.00%
- Libor: flat at 5.00%

**Step 1: Compute Libor-discounted PV of bond cashflows**

Using Libor discounting at 5.00%:

$$P_{\text{Libor}} = \sum_{t=1}^{6} \frac{7.25}{1.05^t} + \frac{107.25}{1.05^7} \approx 113.15$$

**Step 2: Compute RPV01**

$$RPV01 = \sum_{t=1}^{7} \frac{1}{1.05^t} \approx 5.786$$

**Step 3: Compute Par Asset Swap Spread**

$$A = \frac{P_{\text{Libor}} - P_{\text{bond}}}{RPV01} = \frac{113.15 - 94.38}{5.786} = \frac{18.77}{5.786} = 3.24\% = 324 \text{ bp}$$

**Interpretation:**

The par asset swap spread (324 bp) exceeds the simple yield spread because the bond trades at a discount. The discount (100 - 94.38 = 5.62 points) is amortized over the swap life, adding to the spread.

If the bond traded at par, the asset swap spread would be closer to the yield spread over swaps.

---

### Example M: Negative Basis Trade Construction

**Task:** Construct and analyze a negative basis trade where bond spread exceeds CDS spread.

**Setup:**

| Measure | Value |
|---------|-------|
| Bond asset swap spread | 200 bp |
| 5-year CDS spread | 150 bp |
| **Basis** | -50 bp (negative) |

**Trade Construction:**

1. **Buy bond:** $10mm face, price 95, cash outlay = $9.5mm
2. **Buy CDS protection:** $10mm notional, pay 150 bp annually
3. **Finance bond in repo:** Assume repo rate = Libor + 50 bp

**Annual Cashflows (Assuming No Default):**

| Component | Annual Cashflow |
|-----------|----------------|
| Bond coupon (6%) | +$600,000 |
| Swap payment (receive Libor + 200 bp, pay fixed) | ~ +$200,000 net spread |
| CDS premium | -$150,000 |
| Repo financing ($9.5mm × Libor + 50bp) | ~ -$500,000 (at 5.5% total) |

**Net Expected Profit:**

Theoretical basis capture: $50 \text{ bp} \times 10\text{mm} \times 5 \text{ yr duration} \approx \$250{,}000$ over life.

**But Wait—Risks:**

1. **Funding cost:** Repo at Libor + 50 bp consumes 50 bp of the 50 bp basis!
2. **Term mismatch:** Repo is overnight/short-term; trade is 5-year
3. **Margin calls:** Both CDS and repo require margin that increases in stress
4. **Counterparty risk:** CDS counterparty could fail

> **Practitioner Note:** This example illustrates why "obvious" basis arbitrage often fails. The 50 bp negative basis may be entirely consumed by funding costs and risks. In the 2008 crisis, many negative basis trades lost money because:
> - Funding costs spiked (repo spreads widened dramatically)
> - Repo availability collapsed (couldn't roll financing)
> - Margin calls forced liquidation at worst prices

---

### Example N: Distressed Credit—Upfront vs. Running

**Task:** Compare running spread format to upfront format for a distressed credit.

**Distressed Credit:**
- Running CDS spread: 1500 bp (15%)
- Standard coupon: 500 bp (market convention for distressed)
- Maturity: 5 years

**Risky PV01 Calculation:**

For a distressed credit with 1500 bp spread, survival is low. Approximate $RPV01$:

Using flat hazard $\lambda = 1500/(1-40\%) = 2500$ bp = 25%/year:

$$Q(5) = e^{-0.25 \times 5} = e^{-1.25} \approx 0.287$$

$$RPV01 \approx \sum_{t=1}^{5} e^{-0.05t} \cdot e^{-0.25t} \approx 2.18 \text{ years}$$

**Upfront Payment:**

$$U = (S - S_{\text{std}}) \times RPV01 = (1500 - 500) \times 2.18 / 100 = 21.8\%$$

**Interpretation:**

The protection buyer pays 21.8% upfront plus 500 bp running. Equivalently:
- Pay $2.18mm now on $10mm notional
- Pay $500,000 annually until default or maturity

**Why Protection Sellers Prefer Upfront:**

If default occurs in year 1, the seller has received:
- **Running format:** 1500 bp × 1 year × $10mm = $1.5mm
- **Upfront format:** 2180k upfront + 500k running = $2.68mm

The upfront format front-loads payment, protecting the seller from early default.

---

### Example O: Coverage Ratio by Rating

**Task:** Verify the coverage ratio framework using O'Kane's empirical data.

From Section 3.B.1, O'Kane's 2002 CDS data shows:

| Rating | 5Y Avg Spread (bp) | Actuarial Spread (bp) | Coverage Ratio | Spread Premium (bp) |
|--------|-------------------|----------------------|----------------|---------------------|
| AA | 28 | 9 | 3.12 | 19 |
| A | 61 | 13 | 4.67 | 48 |
| BBB | 164 | 30 | 5.54 | 134 |
| BB | 463 | 145 | 3.19 | 318 |

**Verify BBB:** Coverage = 164 / 30 = 5.47 ≈ 5.54 ✓ (small rounding). Premium = 164 − 30 = 134 bp ✓.

**Observations:**

1. **Coverage ratios cluster around 3–5.5× for these ratings**, rather than diverging dramatically
2. **Absolute spread premium increases with risk:** BB premium (318 bp) >> BBB premium (134 bp)
3. **Coverage ratio is non-monotonic:** BBB has the highest ratio (5.54×), not the highest-quality AA

Hull's Table 24.3 reinforces this pattern: Aaa bonds earn 38 bp excess over historical defaults, while Ba bonds earn 160 bp—both reflecting substantial risk premia beyond expected loss.

---

## 5. Practical Notes

### Terminology Collisions

**"Credit DV01" vs "CS01" vs "Spread DV01" vs "Spread Duration"**

This chapter:
- **CS01** = credit DV01 (PV change per 1 bp of the chosen spread), using O'Kane's negative-sign convention.
- **Spread duration** $D_s$ = first-order sensitivity in $dP/P \approx -D_s \, ds$.

### Spread Definition Dependence

| Spread | Description |
|--------|-------------|
| Yield spread $s$ | To Treasuries |
| Z-spread $z$ | To a base curve |
| Asset swap spread | To swaps |
| Par floater spread | Current credit quality measure |
| OAS | For option-embedded bonds |

O'Kane explicitly notes OAS vs ZVS distinction for bonds with embedded options.

### Common Pitfalls (Desk-Relevant)

1. **Mixing clean vs dirty prices** in spread solves; spreads must reprice the same price convention (dirty typically).

2. **Using a spread tied to one benchmark curve** while discounting off another (e.g., quoting spread-to-swaps while using OIS discounting).

3. **Treating Z-spread as "default probability"** without acknowledging:
   - Recovery convention dependence, and
   - Additional premia beyond actuarial spread.

4. **Confusing hazard bumps** (change $\lambda$ or $Q$) **with spread bumps** (change $z$); they coincide only under strong simplifications (credit triangle).

5. **Ignoring recovery sensitivity** when mapping spreads to hazard (Example G).

6. **Assuming negative basis is free money** without accounting for funding costs, term mismatch, and counterparty risk.

7. **Treating FRN as "low duration"** without distinguishing interest rate duration (low) from credit duration (material).

### Implementation Pitfalls

**Root-Finding Stability:**
- For Z-spread, use robust methods (bisection/bracketing) because $P(z)$ is monotone decreasing in $z$.

**Interpolation Artifacts:**
- Z-spread depends on the entire base curve; inconsistent interpolation of $r(0,t)$ can change the solved $z$.

**Schedule/Date Handling:**
- Stub coupons and day-count affect $CF_i$, $AI$, and the timing $t_i$, shifting both PV and implied spreads.

### Verification Tests (Must-Do)

1. **Repricing check:** Solved spread must reproduce market price within tolerance (Example D).

2. **Sign checks:**
   - If $z$ widens $\Rightarrow P$ should fall $\Rightarrow$ with our convention $CS01 > 0$ and $\Delta P \approx -CS01 \cdot \Delta z_{\text{bp}}$.

3. **Scaling:**
   - CS01 scales linearly with notional (per-100 → per-\$1mm multiply by 10,000).

4. **Limiting cases:**
   - **Zero-coupon bond:** spread duration near maturity (approx equals maturity in years under simple discounting).
   - **Zero hazard:** risky PV reduces to risk-free PV.
   - **Recovery extremes:** $R \to 0$ reduces recovery PV; $R \to 1$ increases recovery PV under RFV but may be unrealistic.

5. **Spread Duration = Rate Duration:** For fixed-rate bonds, verify these are equal (Example F).

---

## 6. Summary & Recall

### 12-Bullet Executive Summary

1. Cash credit analytics separates risk-free discounting $Z(0,t)$ from credit risk via survival $Q(0,t)$ and recovery $R$.

2. Survival and hazard relate by $Q(0,T) = \exp(-\int_0^T \lambda)$.

3. Under independence, risky ZCB price factorizes: $\hat{Z}(0,T) = Z(0,T) \, Q(0,T)$.

4. A "payment at default" has PV $D(0,T) = -\int_0^T Z \, dQ$, used for recovery-like cashflows.

5. Recovery must be specified (RFV/RT/RMV); different conventions change implied hazard/spreads.

6. Yield spread is $y = y_T + s$ but is a coarse statistic and can hide curve/coupon effects.

7. Z-spread (ZVS) is the constant spread to the base curve that reprices the bond.

8. Market spreads exceed actuarial spreads due to risk premia (default, volatility, liquidity); coverage ratio quantifies this multiple.

9. Spread duration equals interest rate duration for fixed-rate bonds—corporate bonds are "as much an interest rate play as a credit play."

10. CS01 is the PV change per 1 bp spread move; define precisely what spread is bumped and state sign convention.

11. FRNs have low rate duration but material credit duration; par floater spread measures current vs. issuance credit quality.

12. Cash-CDS basis reflects 12+ fundamental and market factors; negative basis "arbitrage" often fails due to funding and execution risks.

---

### Cheat Sheet (Formulas + Unit Checks)

**Clean/Dirty:**
$$P = P_{\text{clean}} + AI \quad \text{(price per 100)}$$

**Hazard / Survival:**
$$Q(0,T) = \exp\!\left(-\int_0^T \lambda(t) \, dt\right)$$

**Risky ZCB Under Independence:**
$$\hat{Z}(0,T) = Z(0,T) \, Q(0,T)$$

**Payment at Default:**
$$D(0,T) = -\int_0^T Z(0,s) \, dQ(0,s)$$

**Z-Spread (ZVS) Pricing Equation:**
$$P_{\text{mkt}} = \sum_i CF_i \cdot DF\bigl(r(0, t_i) + z;\, t_i\bigr)$$

**Par Floater Spread:**
$$F(t) = \frac{1 - P_{\text{risky}}(t,T)}{RPV01(t,T)}$$

**Asset Swap Spread (Par):**
$$A(0) = \frac{P_{\text{Libor}} - P}{RPV01}$$

**CDS Basis:**
$$\text{Basis} = S_{\text{CDS}} - S_{\text{Bond,Libor}}$$

**Spread Duration:**
$$\frac{dP}{P} \approx -D_s \, ds$$

**Key Identity (Fixed-Rate Bonds):**
$$D_s = D_{y_T}$$

**CS01 Sign Convention (Chapter Default):**
$$CS01 \equiv -\bigl(P(s + 1\text{ bp}) - P(s)\bigr)$$

**DV01 Scaling (Analogy):**
$$DV01 = \frac{PD}{10{,}000}$$

**Coverage Ratio:**
$$\text{Coverage} = \frac{S_{\text{market}}}{S_{\text{actuarial}}}$$

**Upfront CDS:**
$$U = (S - S_{\text{std}}) \times RPV01$$

---

### 46 Flashcards (Q/A)

**Q1:** What is the difference between clean and dirty price?
**A:** Dirty $P$ includes accrued interest; clean excludes it: $P = P_{\text{clean}} + AI$.

**Q2:** Define survival probability $Q(0,t)$.
**A:** $Q(0,t) = P(\tau > t)$.

**Q3:** Define hazard rate $\lambda(t)$ in the intensity model.
**A:** It's the conditional default rate; $Q(0,T) = \exp(-\int_0^T \lambda)$.

**Q4:** What does $Z(0,t)$ represent?
**A:** Risk-free discount factor to time $t$.

**Q5:** Under independence, what is risky ZCB price?
**A:** $\hat{Z}(0,T) = Z(0,T) \, Q(0,T)$.

**Q6:** What is $D(0,T)$ in O'Kane's notation?
**A:** PV of a claim paying 1 at default if default occurs by $T$: $D(0,T) = -\int_0^T Z \, dQ$.

**Q7:** Name three recovery conventions from QRM.
**A:** Recovery of Treasury, Face Value, and Market Value.

**Q8:** Why must recovery convention be specified?
**A:** Because different conventions produce different recovery cashflows and implied hazards/spreads.

**Q9:** What is yield spread $s$ in $y = y_T + s$?
**A:** Corporate yield minus reference Treasury yield.

**Q10:** What is Z-spread (ZVS)?
**A:** Constant spread $z$ added to base curve so PV matches market price.

**Q11:** What is the key difference between ZVS and OAS?
**A:** OAS adjusts for embedded option value; ZVS does not (for option-embedded bonds).

**Q12:** Give the credit triangle formula.
**A:** $S = \lambda(1-R)$.

**Q13:** What is "actuarial spread" in O'Kane's discussion?
**A:** Spread implied by expected loss only; market spread can exceed it due to premia.

**Q14:** Define spread duration $D_s$.
**A:** From $dP/P \approx -D_s \, ds$.

**Q15:** What is CS01 in this chapter's sign convention?
**A:** $CS01 = -(P(s + 1\text{ bp}) - P(s))$.

**Q16:** What is the sign of CS01 for a long credit bond?
**A:** Positive (widening decreases price, so CS01 defined with negative sign is positive).

**Q17:** DV01 formula in terms of price and duration?
**A:** $DV01 = \frac{PD}{10{,}000}$.

**Q18:** Why does CS01 depend on spread definition?
**A:** Because different spreads (yield spread, Z-spread, ASW, OAS) imply different bump directions and repricing rules.

**Q19:** What is a common pitfall when solving spreads?
**A:** Using clean price in one step and dirty price in another.

**Q20:** How do you verify a solved Z-spread?
**A:** Reprice the bond with the solved $z$ and confirm PV matches market price.

**Q21:** What does it mean to be DV01-neutral?
**A:** Portfolio PV is first-order insensitive to the chosen rate bump.

**Q22:** Why can DV01-neutral portfolios still lose money?
**A:** Because they can retain CS01 exposure; credit spreads can widen.

**Q23:** What is basis risk in cash vs CDS?
**A:** Cash bond spread and CDS spread can differ; hedging one with the other is imperfect.

**Q24:** What's the practical meaning of $Q(0,t)$ being risk-neutral?
**A:** It is the survival probability consistent with pricing under the discounting measure.

**Q25:** How does increasing recovery affect implied hazard (credit triangle)?
**A:** For fixed spread $S$, higher $R$ implies higher $\lambda = S/(1-R)$.

**Q26:** How does increasing recovery affect recovery PV under RFV?
**A:** It increases linearly in $R$.

**Q27:** What is the key modeling assumption behind $\hat{Z} = ZQ$?
**A:** Independence of interest rates and default.

**Q28:** What does "payment at default" represent?
**A:** A payoff that occurs at $\tau$, used to model recovery-like payments.

**Q29:** Why might Z-spread change if you change the base curve?
**A:** Because $z$ is defined relative to the chosen $Z(0,t)$ / $r(0,t)$.

**Q30:** What does "spread curve steepening" mean?
**A:** Long-maturity spreads rise relative to short-maturity spreads.

**Q31:** What is a simple P&L approximation using CS01?
**A:** $\Delta PV \approx -CS01 \cdot \Delta s_{\text{bp}}$.

**Q32:** What is asset swap spread approximately in RMFI?
**A:** Asset swap spread $\approx$ bond yield spread − swap spread.

**Q33:** What is the key-rate exposure concept?
**A:** Splitting DV01 across maturity buckets; sum of key-rate 01s equals DV01 under the defined scheme.

**Q34:** Why might market spread exceed actuarial spread?
**A:** Due to default risk premia, volatility, and liquidity premia (conceptual).

**Q35:** What must be stated to make CS01 interpretable?
**A:** The spread definition (Z/OAS/ASW/yield spread), the base curve, compounding, and recovery convention.

**Q36:** What is the cash-CDS basis?
**A:** CDS spread minus bond Libor spread (e.g., par floater spread or asset swap spread).

**Q37:** What is a positive basis? A negative basis?
**A:** Positive: CDS spread > bond spread. Negative: bond spread > CDS spread.

**Q38:** Name three fundamental factors driving CDS basis.
**A:** Funding differences, delivery option in CDS, technical default (broader credit events).

**Q39:** Name three market factors driving CDS basis.
**A:** Synthetic CDO technicals, new issuance hedging, relative liquidity differences.

**Q40:** What is par floater spread?
**A:** The spread that would make a new FRN trade at par today; measures current credit quality.

**Q41:** What is the quoted margin on an FRN?
**A:** The fixed spread over Libor that the FRN pays, set at issuance.

**Q42:** What is the par asset swap spread formula?
**A:** $A = (P_{\text{Libor}} - P) / RPV01$.

**Q43:** Why is spread duration equal to interest rate duration for fixed-rate bonds?
**A:** Because yield = Treasury yield + spread, so bumping either has identical effect on price.

**Q44:** What is coverage ratio?
**A:** Market spread divided by actuarial spread; measures how many times expected loss the market charges.

**Q45:** When does the market switch to upfront format for CDS?
**A:** For distressed credits, typically when spreads exceed 1000 bp.

**Q46:** Why do negative basis trades often lose money?
**A:** Funding costs consume the basis; term mismatch causes forced liquidation; margin calls in stress.

---

## 7. Mini Problem Set (24 Questions)

*Provide brief solution sketches for questions 1–12 only.*

---

**1. Risk-Free PV:** A 2y bond with annual coupons has cashflows $5, 5, 105$. Given discount factors $Z(1) = 0.98$, $Z(2) = 0.95$, compute PV.

**Sketch:** $PV = 5 \cdot Z(1) + 105 \cdot Z(2) = 5(0.98) + 105(0.95) = 4.90 + 99.75 = 104.65$.

---

**2. Dirty vs Clean:** If quoted clean price is 101.20 and accrued interest is 0.35, what is dirty price?

**Sketch:** $P = P_{\text{clean}} + AI = 101.55$.

---

**3. Survival-Weighted PV:** A single cashflow 100 at $T = 3$, $Z(3) = 0.90$, $Q(3) = 0.95$. Compute survival PV.

**Sketch:** $100 \cdot Z(3) \cdot Q(3) = 100(0.90)(0.95) = 85.50$.

---

**4. Payment at Default (Discrete Approximation):** Given $Z(1) = 0.98$, $Z(2) = 0.95$, $Q(0) = 1$, $Q(1) = 0.99$, $Q(2) = 0.97$, approximate $D(0,2) \approx \sum Z(t_i)(Q(t_{i-1}) - Q(t_i))$.

**Sketch:** $D \approx 0.98(1 - 0.99) + 0.95(0.99 - 0.97) = 0.98(0.01) + 0.95(0.02) = 0.0098 + 0.019 = 0.0288$.

---

**5. Credit Triangle Hazard:** If $S = 150$ bp and $R = 40\%$, estimate $\lambda$.

**Sketch:** $\lambda \approx S/(1-R) = 0.015/0.60 = 0.025 = 2.5\%$/year.

---

**6. CS01 Sign:** If $P(z) = 102.00$ and $P(z + 1\text{ bp}) = 101.97$, compute CS01 under chapter convention.

**Sketch:** $CS01 = P(z) - P(z + 1\text{ bp}) = 0.03$ per 100.

---

**7. Spread Duration from CS01:** If $P = 100$ and CS01 (per 100) is 0.04, estimate spread duration.

**Sketch:** $D_s \approx 10{,}000 \cdot CS01 / P = 10{,}000 \cdot 0.04 / 100 = 4$ years.

---

**8. DV01 Scaling:** A bond has price 98 and modified duration 5.2. Compute DV01 (per 100).

**Sketch:** $DV01 = PD/10{,}000 = 98 \cdot 5.2/10{,}000 = 0.05096$ per 100.

---

**9. DV01-Neutral Hedge:** Corporate DV01 is \$300/bp per \$1mm. Treasury DV01 is \$450/bp per \$1mm. What Treasury notional shorts DV01-hedge \$1mm corporate?

**Sketch:** $N_T = 300/450 = 0.6667$ mm short.

---

**10. Coverage Ratio:** Market spread is 180 bp, historical default rate is 1%, recovery is 40%. Compute actuarial spread and coverage ratio.

**Sketch:** $S_{\text{actuarial}} = 0.01 \times 0.60 = 60$ bp. Coverage $= 180/60 = 3.0$.

---

**11. FRN Price:** An FRN has quoted margin 100 bp, par floater spread is now 150 bp, RPV01 = 4.0. What is the approximate price?

**Sketch:** $P \approx 100 - (150 - 100) \times 0.04 = 100 - 2.0 = 98.00$.

---

**12. CDS Basis:** Bond asset swap spread is 200 bp, CDS spread is 175 bp. What is the basis? Is it positive or negative?

**Sketch:** Basis $= 175 - 200 = -25$ bp. Negative basis (bond trades wider than CDS).

---

**13. Curve Dependence:** Explain why changing the base curve (OIS vs swaps) can change the solved Z-spread even if price is unchanged.

---

**14. Clean/Dirty Pitfall:** Describe what happens if you solve Z-spread on clean price but then reprice using dirty price.

---

**15. Recovery Convention:** Compare RFV vs RT in words and state which bond analytics outputs would change.

---

**16. Hazard Bump vs Spread Bump:** Give an example where bumping hazard by +10% and bumping Z-spread by +10 bp produce different PV changes.

---

**17. Portfolio CS01:** Two corporate bonds have CS01s \$200/bp and \$350/bp per \$1mm. You are long \$2mm of the first and short \$1mm of the second. Compute portfolio CS01.

---

**18. Spread Curve Trade:** Design a cash bond steepener/flattener and describe which spread move scenario profits.

---

**19. Basis Trade P&L:** You enter a negative basis trade with 30 bp theoretical pickup. Repo spreads widen 40 bp during the trade. Analyze P&L.

---

**20. Basis Driver Analysis:** A company announces large bond issuance. Which direction should the basis move and why?

---

**21. FRN vs Fixed Credit Duration:** A 5-year FRN and a 5-year fixed-rate bond have the same issuer. Compare their rate durations and credit durations.

---

**22. Upfront Calculation:** CDS running spread is 1200 bp, standard coupon is 500 bp, RPV01 is 2.5. What is the upfront payment?

---

**23. Distressed Curve Shape:** Explain why distressed credits often have inverted credit curves (short-term spreads higher than long-term).

---

**24. Spread Duration = Rate Duration:** Prove that for a fixed-rate bond where $y = y_T + s$, the spread duration equals the rate duration.

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

- Risky ZCB factorization $\hat{Z}(0,T) = Z(0,T) \, Q(0,T)$ under independence (O'Kane Ch 3)
- Payment-at-default PV $D(0,T) = -\int_0^T Z \, dQ$ (O'Kane Ch 3)
- Z-spread (ZVS) definition (O'Kane Ch 4.2.9)
- Credit triangle $S = \lambda(1-R)$ (O'Kane Ch 3.6)
- Spread duration expansion $dP/P \approx -D_s \, ds$ (O'Kane Ch 4.2.7)
- Spread duration = interest rate duration for fixed-rate bonds (O'Kane Ch 4.2.7)
- Credit DV01 sign convention (O'Kane Ch 4.2.7)
- DV01 scaling $DV01 = PD/10{,}000$ (Tuckman Ch 5)
- Recovery conventions RT/RFV/RMV (QRM)
- Clean/dirty price relationship (Tuckman Ch 1-4)
- Par floater spread definition (O'Kane Ch 4.3)
- FRN pricing mechanics (O'Kane Ch 4.3)
- Par asset swap spread formula (O'Kane Ch 4.4, Equation 4.37)
- Market asset swap (O'Kane Ch 4.5)
- CDS basis definition: CDS spread - Bond Libor spread (O'Kane Ch 5.6)
- Six fundamental basis factors (O'Kane Ch 5.6.1): funding, delivery option, technical default, loss on default, premium accrued, spread floor
- Six market basis factors (O'Kane Ch 5.6.2): liquidity, synthetic CDO, new issuance, convertible issuance, protection demand, funding risk
- Credit risk premium decomposition (O'Kane Ch 3.11)
- Coverage ratio = market spread / actuarial spread (O'Kane Ch 3.11)
- Upfront CDS format for distressed credits (O'Kane Ch 6.7)
- Inverted credit curves for distressed names (O'Kane Ch 7.4)
- Correlation correction $\Theta$ for rates-hazard dependence (O'Kane Ch 3.8.1)
- Merton model spread term structure and limitations (O'Kane Ch 3.4, Figure 3.4)
- Coverage ratio/spread premium empirical data (O'Kane Ch 3.11 Table 3.4)
- Risk-neutral vs historical hazards (Hull OFD Ch 24.5, O'Kane Ch 3.11)
- Risk-neutral excess return table (Hull OFD Ch 24.5 Table 24.3)
- CDS-bond basis definition and historical behavior (Hull RM Ch 19.5)

### (B) Claude-Extended Content

- "Desk Reality" boxes providing practitioner perspective on:
  - Why asset swaps are used (isolating credit from rates)
  - FRN credit duration misconceptions
  - Negative basis trade failures in 2008-2009
  - Carry trade dynamics in high-yield
  - Pull-to-par vs. pull-to-recovery
- Coverage ratio interpretation by rating
- Examples M-O constructed from principles using O'Kane framework
- Basis driver summary table synthesizing O'Kane's taxonomy

### (C) Reasoned Inference — Note Derivation Logic

- Bond PV decomposition into survival PV + recovery PV derived from building blocks
- CS01 ≈ $PD_s/10{,}000$ follows from spread duration definition and DV01 analogy
- Portfolio additivity of CS01 follows from PV additivity
- FRN price approximation from par floater spread formula
- Upfront payment calculation from O'Kane formula

### (D) Flagged Uncertainties

- Specific funding cost levels during 2008-2009 crisis — O'Kane written before crisis fully resolved; marked as practitioner note
- Current basis levels and trends — books data is dated; not included as fact
- Coverage ratio data sourced from O'Kane Table 3.4 (2002 CDS data); current market levels may differ
- Correlation correction $\Theta$ calibration is desk-specific; O'Kane provides formula but I'm not sure about current market practice

