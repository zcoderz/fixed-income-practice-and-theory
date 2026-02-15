# Chapter 37: Cash Credit — Risky Bonds, Credit Spreads, and CS01

---

## Introduction

A portfolio manager calls with a simple question: "I'm down \$2 million on my corporate bond book today—is it rates or credit?" On the surface, this seems straightforward, but the answer requires separating two distinct sources of risk: the general level of interest rates (which affects all bonds) and the credit spread specific to the issuer (which reflects default risk, liquidity, and risk premia). Getting this decomposition wrong means misattributing P&L, mishedging positions, and misunderstanding the portfolio's true exposures.

**Why this matters for desk practice:** Middle-office professionals often see spread numbers in risk reports—Z-spread, asset swap spread, CS01—without fully understanding how these measures are constructed, what assumptions they embed, and why they sometimes diverge from CDS spreads on the same issuer. When spreads "blow out" during a credit event or market stress, understanding the mechanics becomes critical: a rates-hedged position can still hemorrhage money through credit spread widening.

This chapter develops the complete framework for cash credit analytics: pricing risky bonds using survival probabilities and recovery assumptions (building on Chapter 36), defining spread measures that isolate credit risk from rate risk, computing credit sensitivities (CS01/spread duration) that drive P&L attribution, and understanding the relationship between cash bonds and the CDS market through the lens of the cash-CDS basis. We also cover floating rate notes and asset swaps—instruments central to how bank desks view credit exposure.

**Roadmap:** We begin with conventions (how to translate a cash-credit quote into a PV object), then define common spread measures used on desks. We then write a risky-bond PV as a survival leg plus a recovery leg, and finally connect those objects to risk measures (DV01 and CS01) and desk-style P&L attribution.

Prerequisites: [Chapter 5 — Fixed-Rate Bond Pricing](chapters/chapter_05_fixed_rate_bond_pricing.md), [Chapter 7 — Bond Return Decomposition](chapters/chapter_07_bond_return_decomposition.md), [Chapter 8 — Spreads 101](chapters/chapter_08_spreads_101.md), [Chapter 11 — DV01/PV01 Definitions & Computation](chapters/chapter_11_dv01_pv01_definitions_computation.md), [Chapter 35 — Default, Recovery, and Credit Events](chapters/chapter_35_default_recovery_credit_events.md), [Chapter 36 — Survival Probabilities and Hazard Rates](chapters/chapter_36_survival_probabilities_hazard_rates.md)

Follow-on: [Chapter 38 — CDS Contract Mechanics](chapters/chapter_38_cds_contract_mechanics.md), [Chapter 42 — Bootstrapping CDS Survival Curve](chapters/chapter_42_bootstrapping_cds_survival_curve.md), [Chapter 43 — CDS Risks & Hedging](chapters/chapter_43_cds_risks_hedging.md), [Chapter 28 — Basis Trades](chapters/chapter_28_basis_trades.md)

---

## Learning Objectives
- Translate a cash-credit quote (clean price, yield spread, Z-spread, asset swap spread) into cashflows, PV, and settlement cash.
- Write a risky-bond PV as a survival leg plus a recovery leg under an explicit recovery convention.
- Define and interpret DV01 and CS01 with explicit bump objects, bump size, units, and sign convention.
- Explain why (for an option-free fixed-rate bond under a simple yield decomposition) spread duration matches rate duration.
- Diagnose cash–CDS basis as an “assumptions gap” (funding, delivery option, liquidity, counterparty/margin) rather than a free arbitrage.

---

## Setup (Prices, Curves, Signs)

### Prices, PVs, and Units
Unless stated otherwise, prices and PVs are quoted **per 100 face** ("per 100"). For notional $N$ (in currency), the dollar PV is:

$$PV_{\$} = N \cdot P / 100$$

### Clean vs Dirty (Settlement Cash Uses Dirty)
Dirty (full) price $P$ equals clean price $P_{\text{clean}}$ plus accrued interest $AI$:

$$P = P_{\text{clean}} + AI$$

When you **buy/sell bonds**, the cash settlement amount is based on the **dirty** price (clean plus accrued), scaled by notional.

### Base Curve and Discount Factors
The base (risk-free) curve is represented by discount factors $Z(0,t)$ (e.g., government/OIS). Risk-free PV of deterministic cashflows is:

$$\sum_{i} CF_i \, Z(0, t_i)$$

### Default Time
$\tau$ is the (random) default time.

### Survival Curve
$Q(0,t) = P(\tau > t)$. Hazard / default intensity $\lambda(t)$ satisfies:

$$Q(0,T) = \exp\!\left(-\int_0^T \lambda(t) \, dt\right)$$

### Recovery Rate
$R \in [0,1]$. You must state the **recovery convention** (RFV/RT/RMV) because it changes the “default-leg” cashflow and therefore the inferred hazard/spread and CS01.

### Basis Points and Risk Sign
- $1\text{ bp} = 10^{-4}$.
- Throughout this chapter we use the book sign convention (matching `refactor_plan/notation_registry.md`): risk numbers are **positive for a long bond**.
  - $DV01 := PV(\text{base rates down }1\text{ bp}) - PV(\text{base})$.
  - $CS01 := PV(\text{spread down }1\text{ bp}) - PV(\text{base})$, where you must name what “spread” means and what is held fixed (default in this chapter: constant Z-spread bump holding the base curve fixed).

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
For a full symbol table (units and sign conventions), see `## Notation` at the end of the chapter.

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

**Check (direction and units):** The Z-spread $z$ is an annualized rate (decimal per year). Increasing $z$ lowers each discount factor and lowers $P_{\text{bond}}$. Setting $z=0$ recovers the base-curve PV of the cashflows.

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

$$\boxed{CS01 := PV(s - 1\text{ bp}) - PV(s) \;\approx\; -\bigl(P(s + 1\text{ bp}) - P(s)\bigr)}$$

**The Key Insight: Spread Duration = Interest Rate Duration (Option-Free Fixed-Rate Bonds)**

For an option-free fixed-rate bond under the simple yield decomposition $y = y_T + s$, bumping the reference yield $y_T$ by 1bp or bumping the spread $s$ by 1bp changes the total yield $y$ by the same amount. In that sense, the first-order “rate duration” and “spread duration” coincide for this stylized decomposition.

$$\boxed{D_s = D_{y_T} \quad \text{(for fixed-rate bonds)}}$$

**Check (what is held fixed):** This equality is about a *stylized yield decomposition* $y=y_T+s$. In practice, your **DV01** might be defined by shifting a zero curve (holding credit inputs fixed), while your **CS01** might be defined by bumping a Z-spread (holding the base curve fixed). Those bump designs need not produce identical numbers, especially for non-flat curves, large spreads, or bonds with embedded options. Treat $D_s=D_{y_T}$ as a useful intuition for option-free bonds—not a universal identity.

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

The factorization $\hat{Z}(0,T)=Z(0,T)Q(0,T)$ relies on a modeling simplification (often phrased as an “independence” assumption) that makes default timing separable from discounting. If interest rates and hazard rates are modeled jointly and are materially correlated, the risky zero-coupon bond price need not equal $ZQ$.

In some joint Gaussian specifications, this shows up as a multiplicative correction term of the form
\[
\hat{Z}(0,T) = Z(0,T)\,Q(0,T)\,\Theta(0,T,\rho),
\]
where $\Theta$ depends on the joint dynamics and the correlation parameter $\rho$. The exact form is model-dependent: if you want to include this effect, you must specify the joint rate/hazard model and calibrate its parameters.

**Sanity check:** if $\rho=0$ (or the relevant volatilities are zero), then $\Theta \to 1$ and you recover $\hat{Z}=ZQ$.

---

### 2.3 Default-Contingent Recovery Cashflow as "Payment at Default"

Define $D(0,T)$ as the PV of **\$1 paid at the time of default** if default occurs by $T$ (a “payment-at-default” claim). Under the same independence assumption as above, this PV can be written as:

$$\boxed{D(0,T) = E\!\left[e^{-\int_0^\tau r(s)\,ds} \, \mathbf{1}_{\{\tau \leq T\}}\right] = -\int_0^T Z(0,s) \, dQ(0,s)}$$

This object is a convenient building block for recovery legs in risky-bond and credit-derivative PV formulas.

**Unit check:** $Z$ and $Q$ are dimensionless; the integral is dimensionless, consistent with a PV-per-1-unit-notional.

---

### 2.4 Reduced-Form Risky Coupon Bond PV Under Fractional Recovery of Face Value (RFV)

**Assumption (explicit):** Recovery is a fixed fraction $R$ of face value $F$ paid at default time $\tau$ if $\tau \leq T$ (RFV). This is one of several recovery conventions; see Section 0 and QRM.

Let contractual cashflows be $CF_i$ at $t_i$, and maturity $T=t_N$. Using (i) survival-contingent cashflows and (ii) the payment-at-default building block $D(0,T)$, one convenient representation of the bond PV is:

$$\boxed{P = \sum_{i=1}^{N} CF_i \, Z(0, t_i) \, Q(0, t_i) \;+\; RF \cdot D(0,T)}$$

where $D(0,T) = -\int_0^T Z(0,s) \, dQ(0,s)$.

**Check (per-100 vs dollars):** The decomposition above produces $P$ “per 100 face.” For notional $N$ in dollars, convert to dollar PV via $PV_{\$}=N\cdot P/100$. Many 10,000× errors in credit analytics come from mixing “per 100” prices with dollar notionals.

**Derivation Sketch (Step-by-Step):**

1. Decompose payoff into survival and default pieces:
   $$\text{Payoff} = \sum_i CF_i \, \mathbf{1}_{\{\tau > t_i\}} + RF \, \mathbf{1}_{\{\tau \leq T\}}$$

2. Discount and take expectation under the (pricing) measure.

3. Use $E\bigl[e^{-\int_0^{t_i} r} \, \mathbf{1}_{\{\tau > t_i\}}\bigr] = Z(0,t_i) \, Q(0,t_i)$ under independence.

4. Use "payment at default" PV for the recovery term.

**Sanity Checks:**

- If $R = 0$, recovery term vanishes and bond PV is just survival-weighted discounted cashflows.
- If $Q(0,t) = 1$ for all $t$, bond PV reduces to risk-free PV (recovery term becomes 0 because $dQ = 0$).
- **If you need a different recovery convention (RT or RMV):** Pricing and CS01 outputs depend on the chosen convention and on what is assumed to be recovered (face vs Treasury value vs market value; treatment of coupon and accrued). To proceed, specify RT vs RFV vs RMV and the cashflow rule at default, then re-derive the recovery leg accordingly. QRM explicitly distinguishes these conventions.

---

### 2.5 Z-Spread Repricing Equation

Given a base curve with discount rates $r(0, t_i)$, the Z-spread (ZVS) $z$ solves:

$$P_{\text{mkt}} = \sum_i CF_i \cdot DF\bigl(r(0, t_i) + z;\, t_i\bigr)$$

with the compounding form matching your curve inputs (discrete or continuous).

**Practical Interpretation:**

The Z-spread is not "the hazard rate." It is the constant spread that prices the bond; it can embed expected default loss, risk premia, liquidity, etc.

---

### 2.6 Spread Duration and CS01

Under a yield decomposition $y = y_T + s$, a small-move expansion gives:

$$\frac{dP}{P} = -D_{r_T} \, dy_T + \frac{1}{2} C_{r_T} \, dy_T^2 - D_s \, ds + \frac{1}{2} C_s \, ds^2 + \cdots$$

For small moves, the first-order approximation is:

$$\frac{dP}{P} \approx -D_s \, ds \quad \Rightarrow \quad \frac{\partial P}{\partial s} \approx -P \, D_s$$

**CS01 Sign Convention (Used Throughout This Chapter):**

We define CS01 as the PV change for a **1bp tightening** of the chosen spread measure:

$$\boxed{CS01 := PV(s - 1\text{ bp}) - PV(s)}$$

Equivalently (to first order), $CS01 \approx -\bigl(P(s + 1\text{ bp}) - P(s)\bigr)$: if spreads widen, prices fall, so the risk number is positive for a long position.

**Link to Spread Duration (Unit Check):**

- $D_s$ is dimensionless (interpretable as "years-like" sensitivity).
- Since 1 bp $= 10^{-4}$:

$$CS01\text{ (per 100)} \approx \frac{P \, D_s}{10{,}000}$$

This mirrors the DV01 relation in Tuckman: $DV01 = \frac{PD}{10{,}000}$.

**Check (toy magnitude):** If a bond is priced at $P=90$ (per 100) with spread duration $D_s=5$, then $CS01 \approx 90\times 5/10{,}000 = 0.045$ points per 100 per bp. On $N=\$100\text{mm}$ notional, that is about $\$45{,}000$ per 1bp spread move.

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

Observed cash-bond spreads are **price-implied** statistics: they are not “pure expected loss.” Even inside a reduced-form framework, what you call “the spread” typically mixes multiple effects:

- expected default loss (often called an “actuarial” or “expected loss” component)
- compensation for bearing credit risk (risk premia)
- liquidity / financing effects

One useful organizing approximation is:

$$\boxed{S_{\text{market}} \approx S_{\text{EL}} + \Pi_{\text{default}} + \Pi_{\text{volatility}} + \Pi_{\text{liquidity}}}$$

where $S_{\text{EL}}$ is an expected-loss spread and the $\Pi$ terms are premia.

**Anchor (credit triangle as a check):** under a flat-hazard, continuous-premium, “ignore discounting” approximation,

$$\boxed{S_{\text{EL}} \approx \lambda(1-R)}$$

This is a *useful sanity check*, not a universal identity.

**Risk-neutral vs real-world (pricing vs forecasting):** if you calibrate $\lambda(t)$ (or $Q(0,t)$) from traded spreads, you get a pricing-measure object that is appropriate for PV and hedging. It should not be read as a literal forecast of real-world default frequencies.

**Coverage ratio (a quick diagnostic):**

$$\boxed{\text{Coverage Ratio} := \frac{S_{\text{market}}}{S_{\text{EL}}}}$$

Large values are common for high-grade credits because $S_{\text{EL}}$ can be tiny even when spreads are economically meaningful.

**Check (toy numbers):** if $R=40\%$ and $\lambda=0.30\%$/year, then $S_{\text{EL}} \approx 0.003 \times 0.60 = 18$ bp. If the bond trades at 90 bp, then only $\sim 18$ bp is “expected loss” under this toy approximation; the remainder reflects premia/liquidity and model misspecification.

> **Desk Reality: What You're Really Buying**
>
> When a portfolio manager says "I'm picking up 200 bp in BBB credit," they're not just picking up 200 bp of expected loss. With a coverage ratio of ~4x, perhaps 50 bp compensates for expected defaults and 150 bp is risk premium.
>
> **Why this matters:** In a recession when defaults spike, the expected loss component may turn out to have been underestimated. But the risk premium component is your compensation for bearing exactly that scenario.

### 3.B.2) The Merton Model Perspective

Structural (Merton-style) models provide useful *intuition* for why credit spreads behave differently across maturities and across “healthy” vs “distressed” firms.

In the simplest version:

- The firm's assets $A(t)$ follow a diffusion.
- Debt is a single zero-coupon obligation with face value $F$ at maturity $T$.
- Default happens at $T$ if $A(T) < F$.

This implies a replication story: **equity is a call option on assets** (strike $F$), and **risky debt is risk-free debt minus a put** on assets. Credit spreads therefore increase with leverage and asset volatility.

**Short-maturity intuition (the “limit check”):**

- If $A(t)$ is comfortably above $F$, the model’s short-dated default probability is tiny, so the short-dated spread tends toward zero as $T-t \to 0$.
- If $A(t)$ is below $F$ (distressed), near-term default probability can be large, so short-dated spreads can be very high; longer-dated spreads can be lower because there is time for $A(t)$ to recover.

This is one structural way to understand upward-sloping vs inverted credit curves.

**Why reduced-form dominates for pricing/hedging:** the simplest Merton setup is stylized (single default date; simplified capital structure; unobserved assets), and it does not naturally produce strictly positive short-dated spreads for very high-grade issuers. Reduced-form (intensity) models are therefore the default workhorse for credit derivatives valuation and for curve calibration.

For this chapter, the key takeaway is: **use structural models as intuition**, but use **reduced-form PV building blocks** (Section 2) for quotes, risk, and P&L explain.

---

### 3.B.3) Distressed Credits and Upfront Trading

**When Does the Market Switch to Upfront?**

When spreads are very wide (distressed names), quoting CDS as a pure running spread can become awkward. Market conventions often switch to an **upfront** quote plus a **standard running coupon** (the exact standard coupons are contract conventions; check the relevant index/single-name specification in practice).

Mechanically, the switch is about separating “how much spread” from “how much PV”: when default risk is high, a large portion of the premium leg may never be paid, so quoting some value upfront can be clearer.

**Upfront Valuation:**

A common approximation relates the upfront amount $U$ to the difference between the par spread $S$ and the standard coupon $S_{\text{std}}$:

$$\boxed{U = (S - S_{\text{std}}) \times RPV01}$$

where $RPV01$ is the risky PV01 (premium-leg annuity) for the contract maturity under the chosen curve/recovery setup. Who pays $U$ depends on whether $S$ is above or below the standard coupon (this is covered carefully in the CDS valuation chapters).

**Inverted Credit Curves:**

Distressed credits can exhibit **inverted credit curves**—short-term spreads higher than long-term spreads. Intuitively:
- Near-term default probability is high
- If the issuer survives the near term, it may recover and be less risky

So “more time” can mean “more time to recover,” which can push longer spreads below shorter spreads.

**Equity-Like Convexity:**

Distressed bonds exhibit **positive convexity** in a way that resembles equity:
- If the issuer defaults, bondholders lose (but are capped at losing face minus recovery)
- If the issuer recovers, the bond can rally dramatically—potentially from \$30 back toward par

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

**Definitions (lock bump objects first):**

- Bump size: $1\text{ bp} = 10^{-4}$.
- These are **signed** risk scalars, reported in currency per 1bp, and we use the convention “positive for a long bond”.

**Spread duration** $D_s$ (relative sensitivity to a spread variable $s$):

$$\frac{dP}{P} \approx -D_s \, ds$$

**CS01** (chapter default; “tightening helps longs”):

$$\boxed{CS01 := PV(s - 1\text{ bp}) - PV(s)}$$

To first order, this is equivalent to $CS01 \approx -\bigl(P(s + 1\text{ bp}) - P(s)\bigr)$: widening $s$ lowers price, so the risk number is positive for a long position.

**DV01** (chapter default; “rates down helps longs”):

$$\boxed{DV01 := PV(r - 1\text{ bp}) - PV(r)}$$

You must state the bump object for $r$ (zero curve, par curve rebuild, yield-to-maturity, key rates, etc.). In this chapter, the default meaning is a **parallel 1bp shift of the base discount curve** used for PV.

**What Is Being Bumped? (Be Explicit)**

| Risk number | Bump object (default in this chapter) | What is held fixed? | Units |
|---|---|---|---|
| $DV01$ | Base discount curve shifted down 1bp (parallel) | Credit inputs (spread/hazard) and cashflow schedule | currency per 1bp |
| $CS01$ | Constant Z-spread $z$ shifted down 1bp (parallel) | Base curve and cashflow schedule | currency per 1bp |

Alternative “CS01” definitions exist (and can be useful), but they answer different questions:
- **Hazard bump CS01 (model-based):** bump $\lambda(t)$ (or $Q(0,t)$) under a stated recovery convention and reprice.
- **OAS bump CS01 (optioned bonds):** bump OAS in a model where cashflows depend on rates.

> **Pitfall — “What is being bumped?” mismatch:** Two systems can both print “CS01” while bumping different objects (Z-spread vs hazard vs OAS, and/or “hold fixed” vs “recalibrate”).  
> **Why it matters:** hedge ratios and P&L explains become apples-to-oranges.  
> **Quick check:** reprice the same bond with two bump definitions and compare magnitudes; if they differ materially, you are not measuring the same risk.

**First-order P&L explain (when moves are small):**

$$\boxed{\Delta PV \approx -DV01 \cdot \Delta r_{\text{bp}} \;-\; CS01 \cdot \Delta s_{\text{bp}}}$$

This equation only makes sense when $\Delta r$ and $\Delta s$ match the bump objects used to compute $DV01$ and $CS01$.

**Duration scaling (unit check):**

- Per 100 notional, $DV01 \approx \frac{P \, D_{\text{mod}}}{10{,}000}$.
- Per 100 notional, $CS01 \approx \frac{P \, D_s}{10{,}000}$.

**Additivity:**

PV is additive across instruments, so (under a common bump definition) **portfolio DV01** and **portfolio CS01** are the signed sums of position DV01/CS01.

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

> **Desk Reality:** Credit carry is “coupon + rolldown” earned by holding a spready bond when spreads don’t widen and the issuer doesn’t default.  
> **Common break:** A spread shock (or downgrade/default) can wipe out months (or years) of carry very quickly.  
> **What to check:** When you quote a carry number, also compute a stress loss estimate like $CS01 \times 50\text{ bp}$ or $CS01 \times 100\text{ bp}$ and sanity-check whether the position can survive that move.

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
- A bond trading at \$80 loses $(1-R) \times 80$ at default
- A bond trading at \$120 loses $(1-R) \times 120$ at default

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

In addition to the structural “product differences” above, the basis moves with market frictions and one-sided flows (sometimes called “technicals”):

**1. Relative liquidity / shorting constraints**

If one market is less liquid or harder to short/finance, it can trade wider because investors demand compensation to hold it (or because the “arb” is hard to implement).

**2. One-sided demand for protection (hedging flows)**

CDS is often the easiest instrument to express “short credit.” Persistent demand to buy protection (macro hedging, issuance hedging, relative-value positioning) can widen CDS spreads relative to bond-implied spreads.

**3. Funding and balance-sheet constraints**

Even if a cash–CDS trade looks attractive in PV terms, repo terms, haircuts, and internal balance-sheet constraints can prevent investors from holding the position until convergence. This can create basis dislocations that persist.

**4. Margin/counterparty/settlement frictions**

CDS requires margin and has counterparty/settlement mechanics; cash bonds require financing, borrowing, and settlement. These frictions can dominate the theoretical basis for long periods.

#### 3.E.3) Regime Note: Why the Basis Can Persist

In calm markets, the basis is often dominated by relative liquidity and supply/demand for protection. In stress, funding/margin constraints and forced selling can dominate, so the basis can gap and remain away from zero even when the “arbitrage” seems obvious on paper.

> **Desk Reality: The "Free Money" Illusion**
>
> A negative basis can look like free money: buy the bond, buy CDS protection, earn the basis. Common breaks:
>
> 1. **Funding terms:** The trade requires financing the bond position; repo rates/haircuts can reprice and financing can disappear.
>
> 2. **Counterparty risk:** If your CDS counterparty defaults, you lose protection when you need it most.
>
> 3. **Margin calls:** Both bond repo and CDS require margin. If spreads widen, margin calls can arrive long before convergence.
>
> 4. **Term mismatch:** Bond financing is often short-term while CDS is long-dated; rollover risk can force an unwind at the wrong time.
>
> The core message: basis trades are not “free carry” unless you can fund, margin, and survive mark-to-market along the path.

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

## 4. Worked Examples (Selected Numeric Examples)

### Worked Example (Template): A Dated Corporate Bond (Clean → Dirty Settlement, DV01/CS01, 1-Day P&L)

**Context**
- You buy a corporate bond at a quoted clean price and want (i) the actual settlement cash amount and (ii) a desk-style “rates vs credit” 1-day P&L explain using DV01 and CS01.

**Timeline (Make Dates Concrete)**
- Trade date: 2026-02-17
- Settlement date: 2026-02-19 (assume T+2 for this example)
- Accrual start/end for the current coupon: 2025-12-15 to 2026-06-15
- Coupon / principal payment dates: 2026-06-15, 2026-12-15, 2027-06-15, 2027-12-15, 2028-06-15 (maturity)

**Inputs**
- Instrument: USD corporate bond, 6.00% annual coupon, semiannual coupons, maturity 2028-06-15, face 100.
- Day count for coupons (assumption for this example): 30/360.
- Market quote (trade): clean price $P_{\text{clean}} = 100.25$ per 100.
- Notional: $N = \$10{,}000{,}000$.
- Toy discounting setup for the PV/risk demo (to keep arithmetic transparent):
  - Flat base curve: continuously-compounded zero rate $r = 4.00\%$.
  - Z-spread quote: $z = 180\text{ bp} = 0.0180$ (continuous-compounding interpretation).

**Outputs (What You Produce)**
- Settlement cash (dirty price × notional).
- PV per 100 and in dollars.
- $DV01$ and $CS01$ with explicit bump objects, bump size, units, and sign:
  - $DV01 := PV(r-1\text{ bp}, z) - PV(r, z)$ (base curve bumped down 1bp, $z$ held fixed).
  - $CS01 := PV(r, z-1\text{ bp}) - PV(r, z)$ (Z-spread bumped down 1bp, base curve held fixed).
  - Units: dollars per 1bp for the stated notional; sign: positive for a long bond.

**Step-by-step**
1. **Clean → dirty:** With 30/360, the accrual fraction from 2025-12-15 to 2026-02-19 is $\tau = 64/360 \approx 0.1778$. Accrued interest per 100 is $AI = 100 \cdot 0.06 \cdot \tau \approx 1.07$. So $P_{\text{dirty}} \approx 100.25 + 1.07 = 101.32$.
2. **Settlement cash:** Cash paid on settlement is $\frac{N}{100} P_{\text{dirty}} \approx 100{,}000 \times 101.32 = \$10.132\text{ mm}$.
3. **PV with Z-spread (toy setup):** Approximate ACT/365 year-fractions from settlement to cashflow dates: $t \approx \{0.318,\,0.819,\,1.318,\,1.819,\,2.318\}$. Price per 100 using
   \[
   PV(r,z) = \sum_i CF_i \exp\bigl(-(r+z)t_i\bigr).
   \]
   With $r+z = 5.80\%$, this gives $PV \approx 101.33$ per 100, consistent with the dirty price.
4. **DV01 and CS01 (bump-and-reprice):** With continuous discounting, a 1bp down-bump changes PV by approximately
   \[
   DV01 \approx CS01 \approx \left(\sum_i CF_i\, t_i\, e^{-(r+z)t_i}\right)\times 10^{-4} \approx 0.022 \quad \text{per 100 per bp}.
   \]
   For $N=\$10\text{ mm}$, $DV01 \approx CS01 \approx 0.022 \times 100{,}000 \approx \$2{,}200$ per bp.

**Cashflows (table)**
| Date | Cashflow (per 100) | Explanation |
|---|---:|---|
| 2026-02-19 | $-101.32$ | Pay dirty price to purchase the bond |
| 2026-06-15 | $+3.00$ | Semiannual coupon |
| 2026-12-15 | $+3.00$ | Semiannual coupon |
| 2027-06-15 | $+3.00$ | Semiannual coupon |
| 2027-12-15 | $+3.00$ | Semiannual coupon |
| 2028-06-15 | $+103.00$ | Final coupon + principal |

**P&L / Risk Interpretation**
- A desk-style first-order explain for a long position is:
  \[
  \Delta PV \approx -DV01 \cdot \Delta r_{\text{bp}} \;-\; CS01 \cdot \Delta z_{\text{bp}},
  \]
  where $\Delta r_{\text{bp}}$ is the base-curve move (in bp) and $\Delta z_{\text{bp}}$ is the Z-spread move (in bp), both defined consistently with the bump objects above.
- Example scenario: if rates rise by $+6$bp and Z-spread widens by $+15$bp, then
  \[
  \Delta PV \approx -2{,}200 \times 6 \;-\; 2{,}200 \times 15 \approx -\$46\text{k}.
  \]
- What breaks this explain: curve-shape changes (key-rate vs parallel), optionality (OAS vs Z-spread), and “spread” definition changes (Z-spread vs hazard vs asset-swap).

**Sanity Checks**
- Units check: “per 100” → dollars via $N/100$; $1\text{ bp}=10^{-4}$ converts a duration-like number into a price change.
- Sign check: rates down or spreads tighten $\Rightarrow$ PV up, so $DV01>0$ and $CS01>0$ for a long bond.
- Reproduction check: you can replicate this with a spreadsheet using columns $\{t_i, CF_i, DF_i, PV_i\}$.

**Debug Checklist (When Your Result Looks Wrong)**
- Did you compute accrued interest using the correct day count and the correct settlement date (not trade date)?
- Did you reprice to the same convention you solved on (clean vs dirty, and the same cashflow schedule)?
- Are you mixing “Z-spread bump” CS01 with a “hazard bump” CS01 (different objects, different answers)?
- Are you reporting DV01/CS01 per 100, per \$1mm, or for the full position?

---

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

**CS01 (Per 100; Chapter Convention):**

$$CS01 := P(z - 1\text{ bp}) - P(z) \approx 107.0152 - 106.9858 \approx 0.0294$$

Equivalently (to first order), $CS01 \approx P(z) - P(z+1\text{ bp})$.

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

$$DV01 := PV(r-1\text{ bp}) - PV(r) \approx P - P_{\text{rates}+} \approx 0.0294$$

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

Coverage ratio close to 1 means the toy expected-loss estimate is close to the market Z-spread in this constructed example. In practice, coverage ratios can be much larger than 1 because observed spreads embed premia and liquidity; if your toy calculation gives an extreme ratio, revisit assumptions (recovery, mapping from $Q$ to $\lambda$, and the spread measure being compared).

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
2. **Buy CDS protection:** \$10mm notional, pay 150 bp annually
3. **Finance bond in repo:** Assume repo rate = Libor + 50 bp

**Annual Cashflows (Assuming No Default):**

| Component | Annual Cashflow |
|-----------|----------------|
| Bond coupon (6%) | +\$600,000 |
| Swap payment (receive Libor + 200 bp, pay fixed) | ~ +\$200,000 net spread |
| CDS premium | -\$150,000 |
| Repo financing ($9.5mm × Libor + 50bp) | ~ -$500,000 (at 5.5% total) |

**Net Expected Profit:**

Theoretical basis capture: $50 \text{ bp} \times 10\text{mm} \times 5 \text{ yr duration} \approx \$250{,}000$ over life.

**But Wait—Risks:**

1. **Funding cost:** Repo at Libor + 50 bp consumes 50 bp of the 50 bp basis!
2. **Term mismatch:** Repo is overnight/short-term; trade is 5-year
3. **Margin calls:** Both CDS and repo require margin that increases in stress
4. **Counterparty risk:** CDS counterparty could fail

> **Practitioner Note:** This example illustrates why "obvious" basis arbitrage often fails. A 50 bp negative basis can be entirely consumed by funding costs and path risks, for example:
> - Funding terms reprice (repo spread/haircut changes can eat the basis)
> - Repo availability can disappear (roll risk forces an unwind)
> - Margin calls can arrive long before convergence

---

### Example N: Distressed Credit—Upfront vs. Running

**Task:** Compare running spread format to upfront format for a distressed credit.

**Distressed Credit:**
- Running CDS spread: 1500 bp (15%)
- Standard coupon: 500 bp (hypothetical standard-coupon contract for this example; check current contract conventions in practice)
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
- Pay \$500,000 annually until default or maturity

**Why Protection Sellers Prefer Upfront:**

If default occurs in year 1, the seller has received:
- **Running format:** 1500 bp × 1 year × $10mm = $1.5mm
- **Upfront format:** 2180k upfront + 500k running = \$2.68mm

The upfront format front-loads payment, protecting the seller from early default.

---

### Example O: Coverage Ratio (Toy Numbers)

**Task:** Compute a toy “expected-loss spread” via the credit triangle and compare it to a market spread.

**Inputs (hypothetical):**
- Recovery: $R = 40\%$
- Flat hazard (toy): $\lambda = 0.50\%$/year
- Observed market spread quote (any spread measure, for intuition): $S_{\text{market}} = 150$ bp

**Step 1 — Expected-loss spread (credit triangle check):**

$$S_{\text{EL}} \approx \lambda(1-R) = 0.005 \times 0.60 = 0.003 = 30\text{ bp}$$

**Step 2 — Coverage ratio:**

$$\text{Coverage Ratio} = \frac{S_{\text{market}}}{S_{\text{EL}}} = \frac{150}{30} = 5$$

**Interpretation (mechanism-level):** in this toy decomposition, only $30$ bp is “expected loss,” while the remaining $120$ bp reflects premia/liquidity and model misspecification. This is why “spread $\neq$ default probability” without specifying assumptions and a calibration target.

---

## 5. Practical Notes

### Terminology Collisions

**"Credit DV01" vs "CS01" vs "Spread DV01" vs "Spread Duration"**

This chapter:
- **CS01** = credit DV01 (PV change for a **1 bp tightening** of the chosen spread measure, with bump object stated; chapter default is a constant Z-spread bump holding the base curve fixed).
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

## Summary
1. Cash-credit PV separates the **base curve** $Z(0,t)$ from **credit** inputs (survival $Q(0,t)$ and a recovery convention).
2. Bond settlement uses the **dirty price**: $P = P_{\text{clean}} + AI$ (quoted per 100).
3. Survival and hazard relate by $Q(0,T)=\exp\!\left(-\int_0^T \lambda(t)\,dt\right)$.
4. Under an independence assumption, a risky zero-coupon bond can be written as $\hat{Z}(0,T)=Z(0,T)\,Q(0,T)$.
5. Under RFV, a risky coupon bond PV decomposes into a **survival leg** plus a **recovery leg** expressed via a payment-at-default building block.
6. Yield spread, Z-spread, asset swap spread, and par floater spread are **different quote objects**; they are not interchangeable without stating conventions.
7. Z-spread is a **constant curve shift** that reprices the bond; it is a pricing statistic, not a default probability by itself.
8. $DV01$ and $CS01$ are bump-and-reprice risk scalars; always state bump object, bump size (1bp), units, and sign.
9. For option-free fixed-rate bonds under a stylized yield decomposition, “rate duration” and “spread duration” coincide; hedging still differs because the hedge instruments differ.
10. Cash–CDS basis should be treated as an assumptions gap (funding, delivery, liquidity, counterparty/margin) and a major source of hedge and P&L drift.

## Key Concepts

| Concept | Definition | Why It Matters |
|---|---|---|
| Clean vs dirty price | $P = P_{\text{clean}} + AI$ (per 100); settlement uses dirty | Prevents settlement/P&L errors |
| Accrued interest $AI$ | Coupon accrued from last coupon date to settlement | Drives clean↔dirty conversion |
| Base discount factor $Z(0,t)$ | Time value of money on the base curve | Separates rates from credit |
| Default time $\tau$ | Random default time | Makes cashflows contingent |
| Survival $Q(0,t)$ | $P(\tau>t)$ | Core credit state variable |
| Hazard $\lambda(t)$ | Intensity with $Q=\exp(-\int \lambda)$ | Parametrizes survival |
| Recovery convention | RT vs RFV vs RMV | Changes recovery leg and sensitivities |
| Payment-at-default PV | $D(0,T)=-\int_0^T Z(0,s)\,dQ(0,s)$ | Building block for recovery legs |
| Yield spread $s$ | $y=y_T+s$ (a quoting statistic) | Easy but can mislead across structures |
| Z-spread $z$ | Constant spread that reprices the bond to a base curve | Common cash-credit quote |
| Asset swap spread | Spread on a floating leg that matches swapped bond PV | Bank-desk “spread” language |
| Par floater spread $F(t)$ | Spread that makes an FRN trade at par today | Cleaner credit measure for FRNs |
| Spread duration $D_s$ | $dP/P\approx -D_s\,ds$ | Maps spread moves to price changes |
| $DV01$ | $PV(r-1\text{bp})-PV(r)$ for a stated rates bump | Rates risk scalar for hedging |
| $CS01$ | $PV(s-1\text{bp})-PV(s)$ for a stated spread bump | Credit risk scalar for hedging |
| Cash–CDS basis | $S_{\text{CDS}}-S_{\text{bond on Libor}}$ | Basis risk / “arbitrage” failure modes |

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $N$ | Notional | currency |
| $P, P_{\text{clean}}$ | Dirty / clean price | price per 100 face |
| $AI$ | Accrued interest | per 100 face; $AI\ge 0$ for standard bonds |
| $CF_i$ | Contractual cashflow at $t_i$ | currency per 100 face |
| $Z(0,t)$ | Base discount factor to time $t$ | unitless; $Z(0,0)=1$ |
| $\tau$ | Default time | date or year-fraction (state which) |
| $Q(0,t)$ | Survival probability | unitless; decreasing; $Q(0,0)=1$ |
| $\lambda(t)$ | Hazard rate / intensity | per year |
| $R$ | Recovery rate | unitless; state RFV/RT/RMV convention |
| $s$ | Yield spread variable | per year or bp; must specify definition |
| $z$ | Z-spread | per year or bp; compounding must be stated |
| $D_s$ | Spread duration | years-like; from $dP/P\approx -D_s\,ds$ |
| $DV01$ | Rates risk scalar | currency per 1bp; $PV(r-1\text{bp})-PV(r)$ for the stated bump object |
| $CS01$ | Spread risk scalar | currency per 1bp; $PV(s-1\text{bp})-PV(s)$ for the stated spread bump object |

## Flashcards

| # | Question | Answer |
|---:|---|---|
| 1 | What price settles on a bond trade: clean or dirty? | Dirty (full) price: $P=P_{\text{clean}}+AI$. |
| 2 | What is survival probability $Q(0,t)$? | $Q(0,t)=P(\tau>t)$. |
| 3 | How do hazard and survival relate? | $Q(0,T)=\exp(-\int_0^T \lambda(t)\,dt)$. |
| 4 | Under independence, what is the risky ZCB factorization? | $\hat{Z}(0,T)=Z(0,T)\,Q(0,T)$. |
| 5 | What is the “payment at default” PV building block? | $D(0,T)=-\int_0^T Z(0,s)\,dQ(0,s)$. |
| 6 | Why must you state a recovery convention? | RT/RFV/RMV imply different default-leg cashflows and different inferred hazards/CS01. |
| 7 | What is Z-spread in one sentence? | The constant spread added to the base curve so discounted cashflows match the market price. |
| 8 | What is Z-spread *not*? | Not a default probability by itself; it can include risk premia and liquidity. |
| 9 | Define DV01 and its sign convention. | $DV01=PV(r-1\text{bp})-PV(r)$; positive for a long bond. |
| 10 | Define CS01 and its sign convention. | $CS01=PV(s-1\text{bp})-PV(s)$; positive for long credit exposure. |
| 11 | What is the most common “CS01 mismatch” pitfall? | Different systems bump different objects (Z-spread vs hazard vs OAS; hold-fixed vs recalibrate). |
| 12 | What is spread duration $D_s$? | $dP/P\approx -D_s\,ds$. |
| 13 | When do rate duration and spread duration coincide (stylized)? | For option-free fixed-rate bonds under $y=y_T+s$ (same yield bump). |
| 14 | Why can an FRN have low DV01 but material CS01? | Resets reduce rate sensitivity, but credit exposure persists over maturity. |
| 15 | What is an asset swap spread in words? | The spread on a floating leg that makes the swapped bond PV match (swap-hedged “credit spread”). |
| 16 | What is par floater spread? | The spread that makes an FRN trade at par today (current credit quality measure). |
| 17 | What is cash–CDS basis? | $S_{\text{CDS}}-S_{\text{bond on Libor}}$. |
| 18 | Give two drivers of basis risk. | Funding/financing; delivery/settlement options; liquidity; counterparty/margin. |
| 19 | What is the minimal repricing check for a solved Z-spread? | Reproduce the same dirty price (same schedule/conventions) within tolerance. |
| 20 | Quick unit check: per-100 CS01 → dollars per \$1mm? | Multiply by $10{,}000$ (since $\$1\text{mm}/100=10{,}000$). |


## Mini Problem Set

1. (Compute) **Risk-free PV:** A 2y bond with annual coupons has cashflows $5, 5, 105$. Given discount factors $Z(1)=0.98$, $Z(2)=0.95$, compute PV (per 100).
2. (Compute) **Dirty vs clean:** If quoted clean price is $101.20$ and accrued interest is $0.35$, what is dirty price?
3. (Compute) **Survival PV:** A single cashflow 100 at $T=3$, with $Z(3)=0.90$ and $Q(3)=0.95$. Compute the survival-weighted PV.
4. (Compute) **Payment-at-default (discrete):** Given $Z(1)=0.98$, $Z(2)=0.95$, $Q(0)=1$, $Q(1)=0.99$, $Q(2)=0.97$, approximate $D(0,2)\approx \sum_i Z(t_i)\bigl(Q(t_{i-1})-Q(t_i)\bigr)$.
5. (Compute) **Credit triangle hazard (rule of thumb):** If $S=150$bp and $R=40\%$, estimate $\lambda$ using $\lambda\approx S/(1-R)$.
6. (Compute) **CS01:** If $P(z)=102.00$ and $P(z-1\text{ bp})=102.03$, compute CS01 under this chapter’s convention (per 100).
7. (Compute) **Spread duration from CS01:** If $P=100$ and $CS01$ (per 100) is $0.04$, estimate spread duration.
8. (Compute) **DV01 scaling:** A bond has price $P=98$ and modified duration $D_{\text{mod}}=5.2$. Compute DV01 (per 100) using $DV01\approx P D_{\text{mod}}/10{,}000$.
9. (Desk, compute) **1-day P&L explain:** You are long \$10mm of a bond with $DV01=\$2{,}200/\text{bp}$ and $CS01=\$2{,}400/\text{bp}$. Rates rise by $+4$bp and Z-spread widens by $+12$bp. Approximate $\Delta PV$ ignoring convexity.
10. (Concept) In 2–3 sentences, explain why Z-spread is not the same thing as “default probability”.
11. (Concept) Give two reasons why a hazard-bump CS01 can differ from a Z-spread-bump CS01.
12. (Desk) A negative basis trade looks attractive. List three practical breaks that can prevent “arbitrage” P&L (funding, term mismatch, counterparty/margin, liquidity, delivery/settlement options, etc.).

### Solution Sketches (Selected)
1. $PV = 5(0.98) + 105(0.95) = 4.90 + 99.75 = 104.65$ (per 100).
2. $P = P_{\text{clean}} + AI = 101.20 + 0.35 = 101.55$.
3. $PV = 100 \cdot 0.90 \cdot 0.95 = 85.50$.
5. $\lambda \approx 0.015/0.60 = 0.025 = 2.5\%$/year.
6. $CS01 = P(z-1\text{bp})-P(z) = 102.03-102.00 = 0.03$ (per 100 per bp).
9. $\Delta PV \approx -(2{,}200)(4) - (2{,}400)(12) = -8{,}800 - 28{,}800 = -\$37{,}600$.
11. Different bumped objects (hazard vs Z-spread) imply different repricing mechanics; hazard-bump CS01 depends on the recovery convention and how the survival curve is parametrized/recalibrated.
12. Example breaks: funding/repo terms change; margin calls arrive before convergence; term mismatch (short repo vs long CDS); liquidity/borrow constraints; counterparty and settlement frictions.

## References

- O'Kane, *Modelling Single-name and Multi-name Credit Derivatives* (“Z-spread / ZVS”, “Asset Swaps”, “Risky PV / payment-at-default building blocks”, “CDS-bond basis”)
- Tuckman & Serrat, *Fixed Income Securities* (duration/DV01 scaling and interpretation; clean/dirty conventions)
- Hull, *Options, Futures, and Other Derivatives* (risk-neutral vs real-world default probabilities; CDS vs bond spread intuition)
- Hull, *Risk Management and Financial Institutions* (“CDS-bond basis” and drivers; practical impediments to arbitrage)
- McNeil, Frey, Embrechts, *Quantitative Risk Management* (recovery conventions RT/RFV/RMV; reduced-form modeling building blocks)
- Neftci, *Principles of Financial Engineering* (DV01/PV01 definitions and risk reporting conventions)
