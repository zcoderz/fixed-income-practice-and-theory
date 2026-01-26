# Chapter 37: Cash Credit — Risky Bonds, Credit Spreads, and Credit DV01 (CS01 / Spread Duration)

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

### 1.7 Asset Swap Spread (Optional, Cash-Credit Practice)

**Formal Definition:**

An asset swap spread is a spread on a swap-like structure that makes the bond's value consistent with floating cashflows + spread, and it is often computed in practice as a discount-margin-type quantity; a common approximation is:

$$\text{asset swap spread} \approx \text{(bond yield spread)} - \text{(swap spread)}$$

Tuckman defines the asset swap spread as the spread $A$ that makes the fixed payments (bond coupon) equal the PV of floating + $A$ payments in an asset swap structure.

**Intuition:**

"Bond spread measured on a swap floating leg basis."

**Practice:**

Often preferred on swap desks; but requires careful curve consistency (swap vs OIS vs gov).

---

### 1.8 Spread Duration and CS01 (Credit DV01)

**Formal Definition:**

If price depends on spread $s$, spread duration is the sensitivity of price to small spread changes:

$$\frac{dP}{P} \approx -D_s \, ds$$

in a decomposition where yield $y = y_T + s$.

Analogously, **credit DV01 / CS01** is the PV change per 1 bp move in the specified spread measure; O'Kane defines "credit DV01" for a spread $F$ with a negative sign so the measure is positive for a typical long-credit position:

$$\text{credit DV01} = -\bigl(P(F + 1\text{ bp}) - P(F)\bigr)$$

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

### A) Pricing Risky Cashflows (Cash Credit Viewpoint)

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

### B) Credit Spread Measures for Cash Bonds (Cash-Credit Emphasis)

**Spread Measures Used in THIS Chapter:**

| Measure | Definition |
|---------|------------|
| **Yield spread** $s$ | $y = y_T + s$ |
| **Z-spread** $z$ (ZVS) | Constant spread to the base curve that matches price |
| **Asset swap spread** (optional) | Swap-based spread consistent with floating + spread structure |

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
> 2.  **I-Spread (Interploated)**: Yield - Interpolated Swap Rate. (Better).
> 3.  **Z-Spread (Zero-Vol)**: Constant shift to the zero curve. (Standard for pricing).
> 4.  **OAS (Option-Adjusted)**: Z-Spread minus the value of call options. (Essential for Callable Bonds).
> 5.  **ASW (Asset Swap)**: What you pay to swap the bond into floating rate cashflows. (Used by Banks/Hedge Funds).

---

### C) Credit DV01 / Spread Duration / CS01

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

### D) Hedging Mindset (Preview-Level, Not a CDS Chapter)

**Conceptual Hedges for Cash Credit Spread Risk:**

- **With other bonds:** relative value hedges (long one bond, short another) to isolate curve/spread shape.
- **With indices or CDS (preview only):** hedge spread risk using CDS indices or single-name CDS; detailed CDS valuation comes later.

**Basis Risk (Preview):**

RMFI notes a common relationship: bond yield spread is expected to be approximately equal to the CDS spread (under simplifying assumptions), and discusses CDS-bond basis concepts.

In practice, cash-bond spreads and CDS spreads can diverge (funding, liquidity, delivery options, etc.). Treat this as **basis risk**.

---

## 4. Worked Examples (10 Numeric Examples)

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

**Interpretation (Explicitly Labeled):** O'Kane conceptualizes the difference between market spread and actuarial spread as arising from additional premia (default risk premium, volatility risk premium, liquidity risk premium).

Do not treat this as an identity; it is a useful way to frame why "spread ≠ expected loss."

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

---

## 6. Summary & Recall

### 10-Bullet Executive Summary

1. Cash credit analytics separates risk-free discounting $Z(0,t)$ from credit risk via survival $Q(0,t)$ and recovery $R$.

2. Survival and hazard relate by $Q(0,T) = \exp(-\int_0^T \lambda)$.

3. Under independence, risky ZCB price factorizes: $\hat{Z}(0,T) = Z(0,T) \, Q(0,T)$.

4. A "payment at default" has PV $D(0,T) = -\int_0^T Z \, dQ$, used for recovery-like cashflows.

5. Recovery must be specified (RFV/RT/RMV); different conventions change implied hazard/spreads.

6. Yield spread is $y = y_T + s$ but is a coarse statistic and can hide curve/coupon effects.

7. Z-spread (ZVS) is the constant spread to the base curve that reprices the bond.

8. Market spreads can exceed actuarial spreads due to risk premia/liquidity premia (conceptually).

9. Spread duration comes from $dP/P \approx -D_s \, ds$.

10. CS01 is the PV change per 1 bp spread move; define precisely what spread is bumped and state sign convention.

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

**Spread Duration:**
$$\frac{dP}{P} \approx -D_s \, ds$$

**CS01 Sign Convention (Chapter Default):**
$$CS01 \equiv -\bigl(P(s + 1\text{ bp}) - P(s)\bigr)$$

**DV01 Scaling (Analogy):**
$$DV01 = \frac{PD}{10{,}000}$$

---

### 35 Flashcards (Q/A)

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

---

## 7. Mini Problem Set (18 Questions)

*Provide brief solution sketches for questions 1–9 only.*

---

**1. Risk-Free PV:** A 2y bond with annual coupons has cashflows $5, 5, 105$. Given discount factors $Z(1) = 0.98$, $Z(2) = 0.95$, compute PV.

**Sketch:** $PV = 5 \cdot Z(1) + 105 \cdot Z(2) = 5(0.98) + 105(0.95)$.

---

**2. Dirty vs Clean:** If quoted clean price is 101.20 and accrued interest is 0.35, what is dirty price?

**Sketch:** $P = P_{\text{clean}} + AI = 101.55$.

---

**3. Survival-Weighted PV:** A single cashflow 100 at $T = 3$, $Z(3) = 0.90$, $Q(3) = 0.95$. Compute survival PV.

**Sketch:** $100 \cdot Z(3) \cdot Q(3) = 100(0.90)(0.95)$.

---

**4. Payment at Default (Discrete Approximation):** Given $Z(1) = 0.98$, $Z(2) = 0.95$, $Q(0) = 1$, $Q(1) = 0.99$, $Q(2) = 0.97$, approximate $D(0,2) \approx \sum Z(t_i)(Q(t_{i-1}) - Q(t_i))$.

**Sketch:** $D \approx 0.98(1 - 0.99) + 0.95(0.99 - 0.97)$. Tie to $D(0,T) = -\int Z \, dQ$.

---

**5. Credit Triangle Hazard:** If $S = 150$ bp and $R = 40\%$, estimate $\lambda$.

**Sketch:** $\lambda \approx S/(1-R) = 0.015/0.60 = 0.025$.

---

**6. CS01 Sign:** If $P(z) = 102.00$ and $P(z + 1\text{ bp}) = 101.97$, compute CS01 under chapter convention.

**Sketch:** $CS01 = P(z) - P(z + 1\text{ bp}) = 0.03$.

---

**7. Spread Duration from CS01:** If $P = 100$ and CS01 (per 100) is 0.04, estimate spread duration.

**Sketch:** $D_s \approx 10{,}000 \cdot CS01 / P = 10{,}000 \cdot 0.04 / 100 = 4$.

---

**8. DV01 Scaling:** A bond has price 98 and modified duration 5.2. Compute DV01 (per 100).

**Sketch:** $DV01 = PD/10{,}000 = 98 \cdot 5.2/10{,}000$.

---

**9. DV01-Neutral Hedge:** Corporate DV01 is \$300/bp per \$1mm. Treasury DV01 is \$450/bp per \$1mm. What Treasury notional shorts DV01-hedge \$1mm corporate?

**Sketch:** $N_T = 300/450 = 0.6667$ mm short.

---

**10. Curve Dependence:** Explain why changing the base curve (OIS vs swaps) can change the solved Z-spread even if price is unchanged.

---

**11. Clean/Dirty Pitfall:** Describe what happens if you solve Z-spread on clean price but then reprice using dirty price.

---

**12. Recovery Convention:** Compare RFV vs RT in words and state which bond analytics outputs would change.

---

**13. Hazard Bump vs Spread Bump:** Give an example where bumping hazard by +10% and bumping Z-spread by +10 bp produce different PV changes.

---

**14. Portfolio CS01:** Two corporate bonds have CS01s \$200/bp and \$350/bp per \$1mm. You are long \$2mm of the first and short \$1mm of the second. Compute portfolio CS01.

---

**15. Spread Curve Trade:** Design a cash bond steepener/flattener and describe which spread move scenario profits.

---

**16. Interpretation:** Explain why market Z-spread might exceed actuarial spread even if expected loss is small.

---

**17. Verification Tests:** List three numerical tests to validate a CS01 implementation.

---

**18. Stress Scenario:** A rates-hedged portfolio has DV01 ≈ 0 but CS01 = \$10,000/bp. What happens under +50 bp credit widening? Compute approximate P&L.

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

- Risky ZCB factorization $\hat{Z}(0,T) = Z(0,T) \, Q(0,T)$ under independence (O'Kane)
- Payment-at-default PV $D(0,T) = -\int_0^T Z \, dQ$ (O'Kane)
- Z-spread (ZVS) definition (O'Kane Ch 4-5)
- Credit triangle $S = \lambda(1-R)$ (O'Kane)
- Spread duration expansion $dP/P \approx -D_s \, ds$ (O'Kane)
- Credit DV01 sign convention (O'Kane)
- DV01 scaling $DV01 = PD/10{,}000$ (Tuckman)
- Recovery conventions RT/RFV/RMV (QRM)
- Clean/dirty price relationship (Tuckman Ch 1-4)

### (B) Reasoned Inference — Note Derivation Logic

- Bond PV decomposition into survival PV + recovery PV derived from building blocks
- CS01 ≈ $PD_s/10{,}000$ follows from spread duration definition and DV01 analogy
- Portfolio additivity of CS01 follows from PV additivity

### (C) Speculation — Flag Uncertainties

- None in this chapter. All content traces to source material or is explicitly flagged as "I'm not sure."
