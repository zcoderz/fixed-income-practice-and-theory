# Chapter 19: Projection Curves — LIBOR/SOFR Term Curves and Why We Need Them (Multi-Curve Framework)

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- Post-crisis markets require multiple curves; a single Libor curve is not sufficient (Andersen & Piterbarg Vol 1 Ch 6)
- OIS rates are used to determine risk-free rates needed to value derivatives (Hull Ch 4, 7)
- Forward rates from an index curve are defined via conditional expectations under the appropriate forward measure (Andersen & Piterbarg Vol 1)
- Tenor basis was historically ~1 bp pre-Sept 2007 but widened to as much as 50 bp post-crisis (Andersen & Piterbarg Vol 1 Ch 6)
- The practical multi-curve rule: compute cashflows using forward rates from the floating reference curve and discount at OIS (Hull Ch 7)
- Basis swap par condition links projection curves (Andersen & Piterbarg Eq 6.49)
- LIBOR is considered unsatisfactory as a reference rate because it is based on bank estimates rather than actual transactions (Hull)

### (B) Reasoned Inference (Derived from A)

- FRN prices deviate from par in multi-curve because the telescoping identity fails when $P_k \neq P_d$
- Par swap rate is a discount-weighted average of projection forwards, not OIS-implied forwards
- Risk decomposition into discount PV01 vs projection PV01 follows from the separate curve roles in valuation

### (C) Speculation (Clearly Labeled; Minimal)

- I'm not sure about exact SOFR compounding conventions across all markets; market conventions vary and the sources discuss general principles rather than exhaustive market-by-market detail

---

## Conventions & Notation

### Time and Valuation

| Symbol | Definition |
|--------|------------|
| $t$ | Valuation time (today $t=0$) |
| $T$, $T_i$ | Future times/dates measured in years |

### Accrual Fractions

| Symbol | Definition |
|--------|------------|
| $\tau_i$ | Floating-leg accrual fraction for period $[T_{i-1}, T_i]$ |
| $\alpha_i$ | Fixed-leg accrual fraction for period $[T_{i-1}, T_i]$ |

### Discount and Projection Curves

| Symbol | Definition |
|--------|------------|
| $P_d(0,T)$ | Discount factor to time 0 from $T$ on the **discount curve** (typically OIS-based under collateralization) |
| $P_k(0,T)$ | Pseudo-discount factor on the **projection/index curve** for index $k$ (e.g., 3M, 6M, SOFR-compounded) |

### Forward Rates

| Symbol | Definition |
|--------|------------|
| $F_k(0;T_{i-1},T_i)$ | Forward rate for index $k$ over $[T_{i-1},T_i]$ implied by $P_k$ |
| $F_d(0;T_{i-1},T_i)$ | Forward rate implied by the discount curve $P_d$ |

### Swap Notation

| Symbol | Definition |
|--------|------------|
| $N$ | Notional |
| $K$ | Fixed swap rate |
| $A_d$ | Discount-curve annuity: $A_d = \sum_i \alpha_i P_d(0,T_i)$ |

### Basis Swaps

| Symbol | Definition |
|--------|------------|
| $e_{1,2}(T)$ | Par basis spread on the $L_1$ leg that makes a floating–floating basis swap (exchanging $L_1$ vs $L_2$) have zero PV |

### Units and Conventions

- **Rates** in decimals (e.g., 3.40% = 0.0340)
- **1 bp** = $10^{-4}$
- **Discount factors** are unitless in $(0,1]$
- In worked examples (toy): ACT/360 approximated as 3M = 0.25, 6M = 0.5, 1Y = 1.0

---

## Setup

### Multi-Curve Framework

We assume a multi-curve setup:
- **Discounting** is done off a discount curve $P_d$
- **Floating cashflows** are projected off an index/tenor-specific projection curve $P_k$

This aligns with the practical rule described in Hull: compute cashflows using forward rates from the curve for the floating reference rate and discount them using the risk-free (OIS) rate.

### Scope

- This chapter is **curve-risk focused**
- Credit/funding adjustments (CVA, etc.) are explicitly flagged as outside scope—these are complex and portfolio-level
- We treat "LIBOR-like" indices as **term rates** (simple compounding over $\tau$)
- For SOFR-style overnight compounding, conventions vary (see Practical Notes)

---

## Core Concepts

### 1) Discount Curve (PV Curve, Typically OIS Under Collateralization)

**Formal Definition:**

A discount curve is a function $T \mapsto P_d(0,T) \in (0,1]$ giving the time-0 price of a zero-coupon bond paying 1 at $T$. It is continuous and monotonically decreasing, and security prices can be expressed as linear combinations of discount factors.

**Intuition:**

"How much is \$1 at time $T$ worth today?" Everything discounted uses this curve.

**Trading / Risk / Portfolio Practice:**

- In modern collateralized derivatives practice, OIS rates are used to determine risk-free rates needed to value derivatives
- The OIS curve can be bootstrapped analogously to a Treasury zero curve
- The overnight rate (historically Fed funds in USD) is considered a proxy for risk-free and is used as the contractual rate to accrue interest on posted collateral, motivating OIS discounting under CSA-like assumptions

---

### 2) Projection Curve (Index Curve, Forwarding Curve)

**Formal Definition:**

A projection (index) curve for a given tenor $\tau$ is defined by the requirement that forward rates for that index equal conditional expectations of future spot index fixings under the appropriate forward measure. In the two-curve setup, the forward Libor rate is defined via a conditional expectation under the $(T+\tau)$-forward measure.

**Intuition:**

The projection curve is your **forecasting engine** for the floating index cashflows. It's not (necessarily) a tradable funding curve; it is a curve that reproduces market quotes for instruments referencing that index/tenor.

**Trading / Risk / Portfolio Practice:**

- For a 3M term floating leg, you build a 3M projection curve calibrated to 3M deposits/FRAs/futures/swaps, and use its forwards to project coupons
- The quoted Libor rate can be treated as an observable index with little relationship to a true discount rate; hence the pseudo-discount curves $P^{(L)}$ are called **index curves**

---

### 3) Why One Curve Is Not Enough (Post-Crisis Motivation)

**Formal Statement:**

Pre-crisis it was often sufficient to build a single (Libor) curve. Post-crisis, "a whole collection of inter-related curves is required," including separate discount and forward curves, to reflect market realities such as index–discounting basis and tenor basis.

**Intuition:**

Term unsecured rates (LIBOR-like) embed credit + liquidity components that are not present in overnight collateralized discounting. So:
- Discounting "risk-free-ish" cashflows uses OIS
- Projecting "LIBOR-like" cashflows uses a tenor-specific curve

**Trading / Risk / Portfolio Practice:**

Hull's practical rule: compute cashflows using forward rates from the curve for the floating reference rate and discount them at the risk-free (OIS) rate.

Desk translation:
- "Discount everything at OIS"
- "Forecast LIBOR 3M coupons off the 3M curve; forecast LIBOR 6M coupons off the 6M curve; forecast SOFR-compounded coupons off the SOFR/OIS curve (if that is the reference)"

---

### 4) Index–Discounting Basis (Forwarding vs Discounting Basis)

**Formal Definition:**

The difference between an index curve $P^{(L)}(t)$ and a discount curve $P(t)$ can be measured via a yield spread $s(t)$:

$$P^{(L)}(t) = P(t) \, e^{-s(t)t}$$

**Intuition:**

If $s(t) \neq 0$, the forwards generated by the index curve differ from discount-curve-implied forwards. A single curve cannot fit both markets.

**Trading / Risk / Portfolio Practice:**

Traders see this as "OIS discounting vs LIBOR forwarding." Risk teams see it as separate buckets: OIS PV01 vs LIBOR forward PV01.

---

### 5) Tenor Basis (Evidence for Multiple Projection Curves)

**Formal Definition:**

Tenor basis is the difference between swaps referencing different tenors (e.g., 1M vs 3M vs 6M). If you build curves from 3M instruments and price a 1M swap, you typically fail to match market quotes—evidence of a basis between the 3M and 1M index curves.

**Intuition:**

Different tenors embody different liquidity/credit/funding preferences (e.g., preference for longer-term deposits to match loan commitments).

**Trading / Risk / Portfolio Practice:**

Historically small (order of 1 bp pre-Sept 2007), but became large (as wide as 50 bp) post-crisis, making "ignore basis" a bad approximation.

---

### 6) Basis Swaps as Curve "Glue"

**Formal Definition:**

A floating–floating basis swap at par satisfies equality of PVs of its legs:

$$\sum_i L_2(0,t_i^2,t_{i+1}^2) \tau_i^2 P_d(t_{i+1}^2) = \sum_i \left(L_1(0,t_i^1,t_{i+1}^1) + e_{1,2}(T)\right) \tau_i^1 P_d(t_{i+1}^1)$$

where $e_{1,2}(T)$ is the quoted basis spread (on the $L_1$ leg).

**Intuition:**

Basis swaps tell you how markets price "6M cashflows vs 3M cashflows." They impose constraints that link projection curves.

**Trading / Risk / Portfolio Practice:**

Additional index curves $P_k$ can be constructed as spread curves to a base index curve, using basis swaps as calibration instruments.

---

### 7) LIBOR vs SOFR and "Term vs Overnight" Style References

**Formal Definition:**

- **Libor:** a filtered average of bank estimates of unsecured interbank borrowing costs for various deposit maturities; commonly used as a reference rate for many contracts
- **Overnight risk-free proxies:** in the US, the effective Fed funds rate is considered a proxy for risk-free and is used for collateral accrual in many agreements; OIS market tied to compounded overnight rates is liquid

Hull notes LIBOR is unsatisfactory as a reference rate because it is based on bank estimates rather than actual transactions.

**Intuition:**

"LIBOR-like term" indices behave like unsecured bank funding benchmarks; "SOFR/overnight" indices behave like overnight (secured/collateral) benchmarks. Their curves and compounding conventions differ.

**Trading / Risk / Portfolio Practice:**

- Swaps referencing SOFR compounded conventions (e.g., "3-month compounded SOFR plus spread") are discussed in Hull
- For overnight-based reference rates, the reference is not observed purely at period start; it is built from one-day rates during the accrual period, and adjustments may be needed when using forward rates for valuation

---

## Math and Derivations

### 2.1 Two-Curve Primitives: What Is "Given" and What Is "Built"

We work with:
- A **discount curve** $P_d(0,T)$ used for PV
- One or more **projection (index) curves** $P_k(0,T)$ used to imply forwards for specific indices

The separation is motivated by the observation that a single curve is not compatible with stressed market conditions and post-crisis curve construction; separating discounting and forwarding ensures linear instruments (swaps, etc.) can be correctly priced at time 0 in stressed conditions.

---

### 2.2 Forward Rates and Index Curves

**Source-backed definition:** In the two-curve setup, the index-curve forward rate $\tilde{L}$ is defined via conditional expectation under the $(T+\tau)$-forward measure and relates to the index curve via a pseudo-discount representation.

For tenor $k$ (term, simple-compounded):

$$\boxed{F_k(0;T,T+\tau) = \frac{1}{\tau} \left( \frac{P_k(0,T)}{P_k(0,T+\tau)} - 1 \right)}$$

This matches the pseudo-discount curve representation of forward Libor rates in the reference text.

**Important subtlety (measure/numeraire):** The forward rate is tied to the forward measure associated with the true discount curve numeraire $P(t,T)$, not the pseudo-discount curve $P^{(L)}$.

**Interpretation:** $P_k$ is a parameterization device for forwards, not necessarily a funding curve.

**Unit check:**
- $P_k(\cdot)$ is unitless
- Ratio $P_k(0,T)/P_k(0,T+\tau)$ is unitless
- Subtracting 1 gives unitless
- Dividing by $\tau$ (years) gives a **per-year rate** ✓

---

### 2.3 Pricing Projected Floating Cashflows Under Separate Discounting

We rely on the general PV principle that prices are linear combinations of discount factors and cashflow coefficients.

#### (a) Floating-Rate Note (FRN) with Separate Curves

Consider an FRN with notional $N$, coupon dates $T_1, \ldots, T_n$. The coupon paid at $T_i$ is:

$$\text{CF}_i = N \, \tau_i \, L_k(T_{i-1}; T_{i-1}, T_i)$$

where $L_k(\cdot)$ is the realized fixing of index $k$ for that accrual period.

**Projection step:** Replace the future fixing with its forward estimate:

$$\mathbb{E}_0[\text{CF}_i] \approx N \, \tau_i \, F_k(0; T_{i-1}, T_i)$$

**Discounting step:** PV each expected cashflow using $P_d$:

$$\boxed{\text{PV}_{\text{FRN}}(0) = \sum_{i=1}^{n} N \, \tau_i \, F_k(0; T_{i-1}, T_i) \, P_d(0, T_i) + N \, P_d(0, T_n)}$$

**Sanity check (single-curve special case):**

If $P_k = P_d = P$ and $F_k$ is computed from $P$, then:

$$\tau_i F(0; T_{i-1}, T_i) = \frac{P(0, T_{i-1})}{P(0, T_i)} - 1$$

So the PV of coupons becomes:

$$\sum_{i=1}^{n} N \left( \frac{P(0, T_{i-1})}{P(0, T_i)} - 1 \right) P(0, T_i) = \sum_{i=1}^{n} N \left( P(0, T_{i-1}) - P(0, T_i) \right) = N(1 - P(0, T_n))$$

Adding principal $N P(0, T_n)$ yields $N$.

This is the classic **"par floater is worth par"** result—and it **breaks** when $P_k \neq P_d$.

#### (b) Fixed-for-Floating IRS Under Separate Curves

Consider a swap with fixed payments at dates $T_i$ (same schedule as floating for simplicity).
- Fixed cashflow at $T_i$: $N K \alpha_i$
- Floating cashflow at $T_i$: $N \tau_i F_k(0; T_{i-1}, T_i)$ (in expectation)

**PV of fixed leg:**

$$\boxed{\text{PV}_{\text{fix}}(0) = \sum_{i=1}^{n} N K \alpha_i P_d(0, T_i) = N K A_d, \quad A_d := \sum_{i=1}^{n} \alpha_i P_d(0, T_i)}$$

**PV of floating leg (no principal exchange):**

$$\boxed{\text{PV}_{\text{flt}}(0) = \sum_{i=1}^{n} N \tau_i F_k(0; T_{i-1}, T_i) P_d(0, T_i)}$$

**Par swap rate** $K_{\text{par}}$ solves $\text{PV}_{\text{fix}} = \text{PV}_{\text{flt}}$:

$$\boxed{K_{\text{par}} = \frac{\displaystyle\sum_{i=1}^{n} \tau_i F_k(0; T_{i-1}, T_i) P_d(0, T_i)}{\displaystyle\sum_{i=1}^{n} \alpha_i P_d(0, T_i)}}$$

**Unit check:**
- Numerator: $\sum (\text{year}) \cdot (1/\text{year}) \cdot (\text{unitless}) = \text{unitless}$
- Denominator: $\sum (\text{year}) \cdot (\text{unitless}) = \text{year}$
- Ratio: $1/\text{year}$ (a rate) ✓

---

### 2.4 Tenor Basis Swap as a Constraint Linking Projection Curves

A floating–floating basis swap exchanging $L_1$ vs $L_2$ has a par condition:

$$\boxed{\sum_{i=0}^{n_2(T)-1} L_2(0, t_i^2, t_{i+1}^2) \tau_i^2 P_d(t_{i+1}^2) = \sum_{i=0}^{n_1(T)-1} \left( L_1(0, t_i^1, t_{i+1}^1) + e_{1,2}(T) \right) \tau_i^1 P_d(t_{i+1}^1)}$$

This equation is what makes tenor basis swaps "glue" curves: given $P_d$ and curve 1 forwards, the observed $e_{1,2}(T)$ constrains curve 2 forwards.

A proposed construction builds $P_2$ as a multiplicative spread to $P_1$:

$$P_2(t) = P_1(t) \exp\left( -\int_0^t \eta_{1,2}(s) \, ds \right)$$

and solves for $\eta_{1,2}$ across maturities $T$.

---

## Measurement & Risk

### 3.1 Why One Curve Is Not Enough Post-Crisis

The 2007–2009 crisis changed the foundations of curve construction: pre-crisis a single Libor discount curve was often enough, but now a collection of inter-related curves is required.

Two concrete market inconsistencies drive this:

1. **Index–discounting basis:** Libor is no longer a good proxy for a risk-free discount rate, so separate discount and index curves are needed (historically recognized in cross-currency markets; later seen in domestic markets too)

2. **Tenor basis:** Building curves from one tenor (e.g., 3M) and using them to price another tenor (e.g., 1M) typically fails; this basis was historically tiny (~1 bp) but widened dramatically (as wide as 50 bp) after Sept 2007

---

### 3.2 The Separation: Discount Curve vs Projection Curve

| Curve | Purpose | Typical Construction |
|-------|---------|---------------------|
| **Discount curve (PV)** | Present-value cashflows | OIS-based curve under collateralized pricing assumptions |
| **Projection curve (expected float)** | Generate forward rates for floating reference | Index-specific curve calibrated to deposits/FRAs/swaps |

**Desk translation:**
- "Discount everything at OIS"
- "Forecast LIBOR 3M coupons off the 3M curve; forecast LIBOR 6M coupons off the 6M curve; forecast SOFR-compounded coupons off the SOFR/OIS curve (if that is the reference)"

---

### 3.3 What "Projection Curve" Means Precisely

A projection curve is **not** "the curve you discount with." It is the curve that produces the forward rates you plug into the floating cashflow formula.

The forward rate being projected (example tenor: 3M):

$$F_{3M}(t; T_{i-1}, T_i) = \frac{1}{\tau_i} \left( \frac{P_{3M}(t, T_{i-1})}{P_{3M}(t, T_i)} - 1 \right)$$

**Link to instruments:**
- Deposits / FRAs / futures pin down short-end forwards for that index
- Fixed–floating swaps referencing the same index pin down longer segments (given a discount curve)
- Tenor basis swaps link multiple projection curves via par conditions

---

### 3.4 Tenor Basis as Evidence of Multiple Forward Curves

Tenor basis exists because the market prices swaps of different frequencies differently, even after accounting for the same discount curve.

| Period | Approximate Tenor Basis (3M/6M) |
|--------|--------------------------------|
| Pre-Sept 2007 | ~1 bp |
| Post-2007 crisis | Up to ~50 bp |

---

### 3.5 Risk Reporting Preview: "Bump Which Curve?"

In multi-curve risk, the first question is: are you bumping discounting or projection?

A spread-based curve-group construction yields orthogonal meanings:

| Perturbation | Risk Interpretation |
|--------------|---------------------|
| Perturb base-index instruments | Overall rate level risk (moves index curves together) |
| Perturb funding instruments | Discounting risk |
| Perturb basis swap spreads | Basis risk (tenors do not move in lockstep) |

Tuckman shows bucket exposures of a par swap to individual forward rates and notes these exposures can be hedged directly (e.g., with Eurodollar futures), motivating a "which forwards did you bump?" mindset.

---

## Worked Examples

### Common Conventions for Examples A–J

Unless otherwise stated:
- **Currency:** USD toy (for illustration)
- **Discount curve:** OIS flat continuously-compounded rate $r_d = 2.00\%$, so $P_d(0,T) = e^{-0.02T}$
- **Projection curve:** 3M term (LIBOR-like) index curve with simple-compounded forwards
- **Day count:** ACT/360 approximated so quarterly accrual $\tau = 0.25$

**Reference discount factors:**

| $T$ (years) | 0.25 | 0.50 | 0.75 | 1.00 | 1.25 | 1.50 | 1.75 | 2.00 |
|-------------|------|------|------|------|------|------|------|------|
| $P_d(0,T) = e^{-0.02T}$ | 0.995012 | 0.990050 | 0.985112 | 0.980199 | 0.975310 | 0.970446 | 0.965605 | 0.960789 |

---

### Example A: Single-Curve vs Multi-Curve Inconsistency

**Goal:** Show a toy market where the OIS curve implies one forward, but FRA/swap quotes imply another forward → one curve cannot fit both.

**Given:**
- Discount curve (OIS): $r_d = 2\%$, so $P_d(0, 0.25) = e^{-0.005} = 0.995012$, $P_d(0, 0.50) = e^{-0.01} = 0.990050$
- Market 3x6 FRA quote for a 3M term index: $K_{\text{FRA}} = 3.40\%$ for $[0.25, 0.50]$, $\tau = 0.25$

**Step 1: OIS-implied forward for $[0.25, 0.50]$**

$$F_d(0; 0.25, 0.50) = \frac{1}{0.25} \left( \frac{0.995012}{0.990050} - 1 \right)$$

Compute ratio: $\frac{0.995012}{0.990050} = e^{0.005} \approx 1.0050125$

$$F_d = 4(1.0050125 - 1) = 4(0.0050125) = 0.020050 \approx 2.0050\%$$

**Step 2: FRA-implied discount factor at 0.50 if you forced a single curve**

Single-curve logic would require $F(0; 0.25, 0.50) = K_{\text{FRA}} = 3.40\%$, implying:

$$P(0, 0.50) = \frac{P(0, 0.25)}{1 + \tau K_{\text{FRA}}} = \frac{0.995012}{1 + 0.25 \cdot 0.0340} = \frac{0.995012}{1.0085} \approx 0.986626$$

**Conclusion:**
- OIS says $P_d(0, 0.50) = 0.990050$
- FRA quote would force $P(0, 0.50) \approx 0.986626$ if single curve

These cannot both be true for the same curve → **one curve cannot fit both OIS discounting instruments and 3M FRA instruments**. This is exactly the motivation for separating discount and projection curves.

---

### Example B: Define Projection Forward Rate from a Projection Curve

**Goal:** Given a projection curve in discount-factor form, compute a forward and interpret it.

**Given (3M projection/index curve):**
- $P_{3M}(0, 0.25) = 0.992063$
- $P_{3M}(0, 0.50) = 0.983702$
- $\tau = 0.25$

**Compute forward:**

$$F_{3M}(0; 0.25, 0.50) = \frac{1}{0.25} \left( \frac{0.992063}{0.983702} - 1 \right)$$

Compute ratio: $\frac{0.992063}{0.983702} \approx 1.0085$

$$F_{3M} = 4(1.0085 - 1) = 4(0.0085) = 0.0340 = 3.40\%$$

**Interpretation:** This forward is the model-implied market forecast (in the sense of the index-curve definition) for the 3M term fixing over $[0.25, 0.50]$, used to project the coupon paid at $T = 0.50$.

---

### Example C: Value a Floater with Separate Curves

**Goal:** Price an FRN where coupons are projected using the 3M projection curve and discounted using OIS.

**Instrument:**
- Notional $N = 100$
- Quarterly coupons at $T_1 = 0.25, T_2 = 0.50, T_3 = 0.75, T_4 = 1.00$
- Coupon at $T_i$: $N \tau F_{3M}(0; T_{i-1}, T_i)$, with $\tau = 0.25$
- Principal $N$ repaid at $T_4 = 1.00$

**Projection forwards (assume given from projection curve quotes):**
- $F_1 = 3.20\%$ for $[0, 0.25]$
- $F_2 = 3.40\%$ for $[0.25, 0.50]$
- $F_3 = 3.50\%$ for $[0.50, 0.75]$
- $F_4 = 3.60\%$ for $[0.75, 1.00]$

**Step 1: Compute coupon cashflows**

$C_i = N \tau F_i$

| $i$ | $F_i$ | $C_i = 100 \cdot 0.25 \cdot F_i$ |
|-----|-------|----------------------------------|
| 1 | 3.20% | 0.8000 |
| 2 | 3.40% | 0.8500 |
| 3 | 3.50% | 0.8750 |
| 4 | 3.60% | 0.9000 |

**Step 2: Discount using OIS discount factors**

| $T_i$ | $P_d(0, T_i)$ | $C_i$ | $\text{PV}_i = C_i \cdot P_d$ |
|-------|---------------|-------|-------------------------------|
| 0.25 | 0.995012 | 0.8000 | 0.796010 |
| 0.50 | 0.990050 | 0.8500 | 0.841542 |
| 0.75 | 0.985112 | 0.8750 | 0.861973 |
| 1.00 | 0.980199 | 0.9000 | 0.882179 |

**Sum coupons PV:**
$$\text{PV}_{\text{cpn}} = 0.796010 + 0.841542 + 0.861973 + 0.882179 = 3.381704$$

**PV of principal:**
$$\text{PV}_{\text{prin}} = 100 \cdot 0.980199 = 98.019867$$

**Total PV:**
$$\text{PV}_{\text{FRN}} = 3.381704 + 98.019867 = 101.401571$$

**Interpretation:** The FRN is worth **above par** (101.40) because the projected 3M forwards (3.2–3.6%) are above the OIS discounting level (2%). In single-curve land, this would have collapsed back to par by the telescoping identity; the fact it doesn't is the **index–discounting basis showing up in PV**.

---

### Example D: Par Swap Rate with Multi-Curve

**Goal:** Compute the par fixed rate for a 1Y quarterly fixed-for-3M swap, discounting at OIS and projecting 3M forwards off $P_{3M}$.

**Swap:**
- Notional $N = 100$
- Pay dates $T_1 = 0.25, \ldots, T_4 = 1.00$
- Fixed leg accrual $\alpha = 0.25$ (toy)
- Floating forwards $F_i$ as in Example C
- Discount curve $P_d$ as in Example C

**Step 1: PV of floating leg**

$$\text{PV}_{\text{flt}} = \sum_{i=1}^{4} N \tau F_i P_d(0, T_i) = 3.381704$$

(Same as coupon PV computed in Example C)

**Step 2: Discount-curve annuity**

$$A_d = \sum_{i=1}^{4} \alpha P_d(0, T_i) = 0.25 (0.995012 + 0.990050 + 0.985112 + 0.980199)$$

Sum of discount factors: $0.995012 + 0.990050 + 0.985112 + 0.980199 = 3.950373$

$$A_d = 0.25 \cdot 3.950373 = 0.987593$$

**Step 3: Solve par rate**

$$K_{\text{par}} = \frac{\text{PV}_{\text{flt}}}{N A_d} = \frac{3.381704}{100 \cdot 0.987593} = \frac{3.381704}{98.7593} \approx 0.034242 = 3.4242\%$$

**Interpretation:** The par fixed rate is essentially a **discount-weighted average of the projection forwards** (not of OIS-implied forwards).

---

### Example E: Bootstrap a Short-Tenor Projection Curve from Deposits/FRAs

**Goal:** Bootstrap the first year of a 3M projection curve $P_{3M}(0,T)$ from short instruments.

**Conventions:**
- 3M tenor, $\tau = 0.25$
- Simple compounding for the term index

**Market quotes (toy):**
- 3M deposit (0 to 0.25): $L_{0,0.25} = 3.20\%$
- 3x6 FRA (0.25 to 0.50): $K_{0.25,0.50} = 3.40\%$
- 6x9 FRA (0.50 to 0.75): $K_{0.50,0.75} = 3.50\%$
- 9x12 FRA (0.75 to 1.00): $K_{0.75,1.00} = 3.60\%$

**Step 0: Start**
$$P_{3M}(0,0) = 1$$

**Step 1: Deposit gives $P_{3M}(0, 0.25)$**

Treat the deposit as a unit notional loan repaid at $T = 0.25$ with interest $L\tau$. No-arbitrage PV equation:

$$1 = P_{3M}(0, 0.25)(1 + L\tau) \quad \Rightarrow \quad P_{3M}(0, 0.25) = \frac{1}{1 + 0.25 \cdot 0.0320} = \frac{1}{1.008} = 0.992063$$

**Step 2: FRA gives $P_{3M}(0, 0.50)$**

The forward rate implied by the index curve over $[0.25, 0.50]$ is:

$$F_{3M}(0; 0.25, 0.50) = \frac{1}{0.25} \left( \frac{P_{3M}(0, 0.25)}{P_{3M}(0, 0.50)} - 1 \right)$$

Set $F_{3M} = K_{0.25,0.50} = 0.0340$ and solve:

$$\frac{P_{3M}(0, 0.25)}{P_{3M}(0, 0.50)} - 1 = 0.25 \cdot 0.0340 = 0.0085$$

$$P_{3M}(0, 0.50) = \frac{P_{3M}(0, 0.25)}{1.0085} = \frac{0.992063}{1.0085} \approx 0.983702$$

**Step 3: Continue for 0.75**

$$P_{3M}(0, 0.75) = \frac{P_{3M}(0, 0.50)}{1 + 0.25 \cdot 0.0350} = \frac{0.983702}{1.00875} \approx 0.975169$$

**Step 4: Continue for 1.00**

$$P_{3M}(0, 1.00) = \frac{P_{3M}(0, 0.75)}{1 + 0.25 \cdot 0.0360} = \frac{0.975169}{1.009} \approx 0.966471$$

**Output (bootstrapped 3M projection DFs):**

| $T$ | $P_{3M}(0, T)$ |
|-----|----------------|
| 0.25 | 0.992063 |
| 0.50 | 0.983702 |
| 0.75 | 0.975169 |
| 1.00 | 0.966471 |

---

### Example F: Extend Projection Curve Using Swaps

**Goal:** Add a longer par swap quote and solve for a longer projection node, holding the OIS discount curve fixed.

**Setup:**
- Discount curve: OIS $r_d = 2\%$ (fixed)
- Known 3M projection curve through 1Y from Example E
- We add a 2Y par swap quote referencing the same 3M index, with quarterly payments on both legs (toy)
- Assume the unknown forwards beyond 1Y are **flat at a constant $x$** for the four quarters from 1.00 to 2.00 (piecewise-constant forward assumption)

**Market quote (toy):**
- 2Y par fixed rate: $K_{2Y} = 3.60\% = 0.0360$

**Step 1: Compute OIS annuity to 2Y**

$$A_d^{2Y} = \sum_{i=1}^{8} 0.25 \, P_d(0, T_i), \quad T_i = 0.25, 0.50, \ldots, 2.00$$

Sum discount factors (from table): $\sum_{i=1}^{8} P_d(0, T_i) = 7.822523$

$$A_d^{2Y} = 0.25 \cdot 7.822523 = 1.955631$$

Fixed-leg PV at par:
$$\text{PV}_{\text{fix}} = N K_{2Y} A_d^{2Y} = 100 \cdot 0.0360 \cdot 1.955631 = 7.040271$$

**Step 2: Floating-leg PV splits into known first-year part + unknown second-year part**

First-year floating PV (already computed in Example C coupon PV):
$$\text{PV}_{\text{flt,0–1Y}} = 3.381704$$

Second-year floating PV under flat forward $x$:
$$\text{PV}_{\text{flt,1–2Y}} = \sum_{i=5}^{8} N \cdot 0.25 \cdot x \cdot P_d(0, T_i) = Nx \left( \sum_{i=5}^{8} 0.25 P_d(0, T_i) \right)$$

Compute the second-year annuity piece:
$$\sum_{i=5}^{8} 0.25 P_d(0, T_i) = 0.25 (0.975310 + 0.970446 + 0.965605 + 0.960789) = 0.968038$$

$$\text{PV}_{\text{flt,1–2Y}} = 100 \cdot x \cdot 0.968038 = 96.8038 \, x$$

Total floating PV:
$$\text{PV}_{\text{flt}} = 3.381704 + 96.8038 \, x$$

**Step 3: Par condition $\text{PV}_{\text{fix}} = \text{PV}_{\text{flt}}$**

$$7.040271 = 3.381704 + 96.8038 \, x$$
$$96.8038 \, x = 3.658567$$
$$x \approx 0.037794 = 3.7794\%$$

**Step 4: Extend the projection curve discount factors**

Use $P_{3M}(0, 1.00) = 0.966471$ and the recurrence:
$$P_{3M}(0, T+\tau) = \frac{P_{3M}(0, T)}{1 + \tau x}, \quad \tau = 0.25$$

Compute $1 + \tau x = 1 + 0.25 \cdot 0.037794 = 1.0094485$

| $T$ | $P_{3M}(0, T)$ |
|-----|----------------|
| 1.25 | $0.966471 / 1.0094485 \approx 0.957425$ |
| 1.50 | $0.957425 / 1.0094485 \approx 0.948469$ |
| 1.75 | $0.948469 / 1.0094485 \approx 0.939597$ |
| 2.00 | $0.939597 / 1.0094485 \approx 0.930808$ |

**Interpretation:** A longer swap quote pins down longer-dated forward expectations on the projection curve given a discount curve. With more quotes (and a more flexible parameterization), you'd solve for a richer term structure rather than one flat $x$.

---

### Example G: Tenor Basis Swap as a Constraint

**Goal:** Use a toy 3M vs 6M basis swap quote to show how basis instruments link two projection curves.

**Setup:**
- Discount curve: OIS $P_d$ as above
- Known 3M projection forwards (from Example E): $F_{1..4}^{3M} = (3.20\%, 3.40\%, 3.50\%, 3.60\%)$
- We consider a 1Y basis swap:
  - 3M leg pays quarterly at $0.25, 0.50, 0.75, 1.00$, $\tau_{3M} = 0.25$
  - 6M leg pays semiannually at $0.50, 1.00$, $\tau_{6M} = 0.5$
  - Quoted basis spread on 3M leg: $e_{3M,6M}(1Y) = +10 \text{ bp} = 0.0010$
- For toy solvability, assume the 6M forward is constant $y$ for both half-year periods

**Step 1: PV of 3M leg (without basis spread)**

Per unit notional:
$$\text{PV}_{3M} = \sum_{i=1}^{4} 0.25 F_i^{3M} P_d(0, T_i) = 0.03381704$$

(We already computed for $N = 100$: $\text{PV}_{\text{flt}} = 3.381704$)

**Step 2: PV of basis spread on 3M leg**

$$\text{PV}_{\text{spr}} = \sum_{i=1}^{4} 0.25 \cdot e \cdot P_d(0, T_i) = e \cdot \underbrace{\sum_{i=1}^{4} 0.25 P_d(0, T_i)}_{A_d^{1Y}}$$

We computed $A_d^{1Y} = 0.987593$:
$$\text{PV}_{\text{spr}} = 0.0010 \cdot 0.987593 = 0.000987593$$

PV of "3M + spread" leg:
$$\text{PV}_{3M+e} = 0.03381704 + 0.000987593 = 0.03480463$$

**Step 3: PV of 6M leg under constant forward $y$**

$$\text{PV}_{6M} = 0.5 \cdot y \cdot P_d(0, 0.50) + 0.5 \cdot y \cdot P_d(0, 1.00) = y \cdot 0.5(0.990050 + 0.980199)$$

$$0.5(0.990050 + 0.980199) = 0.985124$$

$$\text{PV}_{6M} = 0.985124 \, y$$

**Step 4: Par condition $\text{PV}_{6M} = \text{PV}_{3M+e}$**

$$0.985124 \, y = 0.03480463 \quad \Rightarrow \quad y = 0.035330 = 3.5330\%$$

**Interpretation:** A 3M/6M basis quote pins down the relative level of 6M forwards vs 3M forwards (given discounting). With multiple maturities, you can build the entire 6M curve as a spread curve to the 3M curve.

---

### Example H: Quote Bump Attribution — Discount vs Projection

**Goal:** Show that "bump which curve?" matters. We compute PV change under:
1. +1 bp bump to OIS discount curve (projection fixed)
2. +1 bp bump to a projection quote (discount fixed)

**Instrument:**
- 1Y quarterly swap, notional $N = 100$
- Receive fixed $K = 3.30\% = 0.0330$, pay 3M floating
- Base curves: as in Examples C–D

**Base PV:**

$\text{PV}_{\text{fix}} = N K A_d^{1Y}$, with $A_d^{1Y} = 0.987593$:
$$\text{PV}_{\text{fix}} = 100 \cdot 0.0330 \cdot 0.987593 = 3.259058$$

$\text{PV}_{\text{flt}} = 3.381704$ (Example C coupon PV)

Swap PV (receive fixed – pay float):
$$\text{PV}_0 = 3.259058 - 3.381704 = -0.122646$$

#### H1) Bump OIS by +1 bp, rebuild discount curve; keep projection fixed

New OIS flat rate: $r_d' = 2.01\%$, so $P_d'(0,T) = e^{-0.0201T}$

Discount factors at pay dates (computed via $P_d'(T) = P_d(T) e^{-0.0001T}$):

| $T$ | $P_d'(0, T)$ |
|-----|--------------|
| 0.25 | 0.994988 |
| 0.50 | 0.990000 |
| 0.75 | 0.985038 |
| 1.00 | 0.980101 |

**Recompute annuity:**
$$A_d'^{1Y} = 0.25(0.994988 + 0.990000 + 0.985038 + 0.980101) = 0.987532$$

$$\text{PV}_{\text{fix}}' = 100 \cdot 0.0330 \cdot 0.987532 = 3.258854$$

**Recompute floating PV** (forwards unchanged, discounting changed):

Coupons are unchanged (0.8000, 0.8500, 0.8750, 0.9000) but discounted with $P_d'$:
$$\text{PV}_{\text{flt}}' \approx 3.381489$$

**Swap PV after discount bump:**
$$\text{PV}_1 = \text{PV}_{\text{fix}}' - \text{PV}_{\text{flt}}' = 3.258854 - 3.381489 = -0.122635$$

**Discount-curve PV change:**
$$\Delta\text{PV}_{\text{disc}} = \text{PV}_1 - \text{PV}_0 = (-0.122635) - (-0.122646) = +0.0000117$$

(per $N = 100$ notional)

#### H2) Bump a projection quote by +1 bp; keep discount curve fixed

We bump the 3x6 FRA quote (0.25 to 0.50) by +1 bp:
$$K_{0.25,0.50}: 3.40\% \to 3.41\% \quad (\Delta F = 0.0001)$$

Only the second floating coupon (paid at 0.50) changes.

**Change in that coupon:**
$$\Delta C_2 = N \tau \Delta F = 100 \cdot 0.25 \cdot 0.0001 = 0.0025$$

**Discounted change:**
$$\Delta\text{PV}_{\text{flt}} = \Delta C_2 \cdot P_d(0, 0.50) = 0.0025 \cdot 0.990050 = 0.002475$$

Since we pay float, swap PV decreases:
$$\Delta\text{PV}_{\text{proj}} = -0.002475$$

$$\text{PV}_2 = \text{PV}_0 + \Delta\text{PV}_{\text{proj}} = -0.122646 - 0.002475 = -0.125122$$

**Comparison / interpretation:**

| Bump Type | PV Change (per 100 notional) |
|-----------|------------------------------|
| Discount curve +1 bp | $+1.17 \times 10^{-5}$ |
| Projection FRA +1 bp | $-2.48 \times 10^{-3}$ |

This is a concrete illustration of why risk reports must distinguish:
- **Discount PV01** (curve used only for PV)
- **Projection PV01** (curve used to generate forward coupons)

---

### Example I: "Which Curve for Forward?" Sanity

**Goal:** Quantify that in multi-curve, the forward rate used to project float cashflows is not necessarily the discounting-implied forward from OIS DFs.

Consider period $[0.25, 0.50]$, $\tau = 0.25$.

**OIS-implied forward:**
$$F_d(0; 0.25, 0.50) = \frac{1}{0.25} \left( \frac{P_d(0, 0.25)}{P_d(0, 0.50)} - 1 \right) \approx 2.0050\%$$

**Projection forward (3M):**

From projection curve / FRA quote:
$$F_{3M}(0; 0.25, 0.50) = 3.40\%$$

**Difference:**
$$F_{3M} - F_d \approx 3.40\% - 2.005\% = 1.395\% = 139.5 \text{ bp}$$

**Interpretation:** Discount-curve-implied forwards are the right forwards for cashflows indexed to the discount curve's underlying rate (e.g., OIS/overnight). But for LIBOR-like term coupons, you must project using the index curve; otherwise you will misprice and mis-hedge.

---

### Example J: Locality/Interpolation Effect on Forwards

**Goal:** Show that interpolation choices can materially change implied forwards ("forward wiggles").

**Given nodes:**
- $P(0) = 1.00$
- $P(1.0) = 0.90$

We need $P(0.5)$ to compute 6M forwards. Two interpolation rules:

**Method 1: Linear interpolation of discount factors**
$$P_{\text{lin}}(0.5) = \frac{1.00 + 0.90}{2} = 0.95$$

**Method 2: Log-linear interpolation of discount factors**

(Equivalent to linear interpolation of continuously-compounded zero rates)
$$\ln P_{\text{log}}(0.5) = \frac{\ln 1.00 + \ln 0.90}{2} = \frac{0 + \ln 0.90}{2}$$
$$P_{\text{log}}(0.5) = \sqrt{0.90} = 0.948683$$

**Now compute 6M simple forwards ($\tau = 0.5$):**

**Forward for $[0, 0.5]$:**
$$F(0; 0, 0.5) = \frac{1}{0.5} \left( \frac{P(0)}{P(0.5)} - 1 \right) = 2 \left( \frac{1}{P(0.5)} - 1 \right)$$

| Method | $P(0.5)$ | $F(0; 0, 0.5)$ |
|--------|----------|----------------|
| Linear DF | 0.95 | $2(1.0526316 - 1) = 10.5263\%$ |
| Log-linear DF | 0.948683 | $2(1.0540926 - 1) = 10.8185\%$ |

**Forward for $[0.5, 1.0]$:**
$$F(0; 0.5, 1.0) = \frac{1}{0.5} \left( \frac{P(0.5)}{P(1.0)} - 1 \right)$$

| Method | $F(0; 0.5, 1.0)$ |
|--------|------------------|
| Linear DF | $2(0.95/0.90 - 1) = 11.1111\%$ |
| Log-linear DF | $2(0.948683/0.90 - 1) = 10.8185\%$ |

**Conclusion:**
- Under **log-linear interpolation**, the forwards are **flat** (10.8185%, 10.8185%)—a smooth exponential DF shape
- Under **linear-DF interpolation**, forwards "**wiggle**" (10.5263%, 11.1111%)

This is why interpolation choices matter for projected forwards and for the stability of risk under bumps.

---

## Practical Notes

### 5.1 Standard Curve Families (Conceptual)

A modern single-currency setup often involves:

| Curve Type | Purpose | Notes |
|------------|---------|-------|
| **Discount curve** | PV (collateralized) | OIS-based; defines risk-free zero curve |
| **Projection curves** | One per index/tenor | Legacy LIBOR 1M/3M/6M; SOFR-based curves |
| **Cross-curve links** | Tenor basis swaps | Link 3M vs 6M (or similar) via par basis spreads |

---

### 5.2 Common Pitfalls (Desk-Relevant)

**1. Projecting float coupons off the discount curve by accident**

Leads to systematic mispricing when $F_k \neq F_d$ (see Example I showing 139.5 bp difference).

**2. Mixing day counts / compounding conventions across curves**

Particularly relevant for overnight-based indices where rates are observed throughout the accrual period; the usual "beginning-of-period observed" assumption fails and adjustments are needed.

**3. Ignoring tenor basis because it's "small"**

Historically it could be ~1 bp, but post-2007 it has been far larger (up to ~50 bp), making it material.

**4. Not distinguishing curve risk**

Risk depends on **what curve you bump**: discount vs projection vs basis (Example H). The orthogonal risk interpretation under spread-based curve groups is essential.

---

### 5.3 Implementation Pitfalls (Curve Construction Dependency Graph)

A practical construction workflow:

1. **Build discount curve first** (OIS discounting under collateralization)

2. **Build a base projection curve** for a primary tenor $L_1$ (e.g., 3M) from deposits/FRAs/swaps referencing that tenor

3. **Build additional projection curves** as spread curves to the base using basis swaps and a spread parameterization

4. **Validate:** all calibration instruments should reprice to (near) zero PV / par

---

### 5.4 Verification Tests (Must-Have)

| Test | Purpose |
|------|---------|
| **Repricing checks** | Every OIS, FRA, swap, basis swap included in calibration must reprice (within tolerance) |
| **Curve-link consistency** | Basis swaps used to link curves must reprice (otherwise curve group is internally inconsistent) |
| **Bump stability** | Small quote bumps should not induce wild forward oscillations; interpolation choice and tension can matter |

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **Post-crisis markets require multiple curves**; one Libor curve is not enough

2. The **discount curve** $P_d$ is used for PV; under collateralization it is typically OIS-based

3. A **projection curve** $P_k$ is index/tenor-specific and is used to generate forward rates for cashflow projection

4. Swaps can be valued by **projecting float cashflows using forwards from the floating reference curve** and **discounting at OIS**

5. The pseudo-discount representation $F = \frac{1}{\tau}\left(\frac{P_k(T)}{P_k(T+\tau)} - 1\right)$ defines forwards from an index curve

6. **Index–discounting basis** means $F_k$ (projection) generally differs from $F_d$ (discounting-implied). Example I quantified this as 139.5 bp

7. **Tenor basis** (1M vs 3M vs 6M) is evidence that multiple projection curves are needed; it widened dramatically post-2007

8. **Basis swaps** provide constraints linking curves (par condition)

9. **Risk must be decomposed**: discounting risk vs projection (index) risk vs basis risk; spread-based curve groups yield orthogonal interpretations

10. **Interpolation choices** can create forward "wiggles" and unstable risk; tension/smoothing can help

---

### Cheat Sheet of Formulas

**Discount vs Projection:**
- Discount factor: $P_d(0,T)$ for PV
- Projection (index) pseudo-discount: $P_k(0,T)$ for forwards

**Forward rate from projection curve:**
$$\boxed{F_k(0; T_{i-1}, T_i) = \frac{1}{\tau_i} \left( \frac{P_k(0, T_{i-1})}{P_k(0, T_i)} - 1 \right)}$$

**FRN PV (separate curves):**
$$\boxed{\text{PV}_{\text{FRN}}(0) = \sum_{i=1}^{n} N \tau_i F_k(0; T_{i-1}, T_i) P_d(0, T_i) + N P_d(0, T_n)}$$

**Swap PV:**
$$\text{PV}_{\text{swap}}(0) = NK \sum_i \alpha_i P_d(0, T_i) - N \sum_i \tau_i F_k(0; T_{i-1}, T_i) P_d(0, T_i)$$

**Par fixed rate:**
$$\boxed{K_{\text{par}} = \frac{\displaystyle\sum_i \tau_i F_k(0; T_{i-1}, T_i) P_d(0, T_i)}{\displaystyle\sum_i \alpha_i P_d(0, T_i)}}$$

**Tenor basis swap par condition:**
$$\sum_i L_2 \tau_2 P_d = \sum_i (L_1 + e_{1,2}) \tau_1 P_d$$

---

### Flashcards (30 Q/A)

1. **Q:** What is the discount curve used for?
   **A:** Present-valuing cashflows; in collateralized setups it's typically OIS-based.

2. **Q:** What is a projection curve used for?
   **A:** Producing forward rates for a specific floating index/tenor to project cashflows.

3. **Q:** State the forward-from-index-curve formula.
   **A:** $F_k(0; T, T+\tau) = \frac{1}{\tau}\left(\frac{P_k(0,T)}{P_k(0,T+\tau)} - 1\right)$

4. **Q:** What changed post-2007 about curve construction?
   **A:** A single Libor curve was often sufficient pre-crisis; now multiple inter-related curves are required.

5. **Q:** What is tenor basis?
   **A:** The pricing difference between swaps of different floating tenors (e.g., 1M vs 3M vs 6M) requiring separate index curves.

6. **Q:** Why does tenor basis exist (intuition)?
   **A:** Credit/liquidity and funding preferences differ across tenors; e.g., desire for longer-term deposits.

7. **Q:** What is the practical multi-curve swap valuation rule?
   **A:** Project cashflows using forward rates from the floating reference curve and discount at the risk-free (OIS) rate.

8. **Q:** What is index–discounting basis?
   **A:** The difference between the discount curve and the index curve; can be expressed as $P^{(L)} = P e^{-s(t)t}$.

9. **Q:** When do $F_k$ and $F_d$ coincide?
   **A:** In a single-curve setup where the index curve equals the discount curve (pre-crisis assumption in some markets).

10. **Q:** What does "bump which curve?" mean?
    **A:** Risk depends on whether you bump discounting, projection, or basis instruments; they measure different exposures.

11. **Q:** What instrument links two projection curves?
    **A:** A floating–floating basis swap with a quoted basis spread $e_{1,2}(T)$.

12. **Q:** Write the par condition for a tenor basis swap.
    **A:** PV(floating leg 2) = PV(floating leg 1 + basis spread).

13. **Q:** What is PV01 to the discount curve?
    **A:** Sensitivity of PV to discount-curve instruments (OIS); "discounting risk."

14. **Q:** What is PV01 to a projection curve?
    **A:** Sensitivity of PV to instruments that calibrate the index curve (e.g., FRAs, swaps referencing that index).

15. **Q:** How does Tuckman motivate forward-rate bucket exposures?
    **A:** By showing how a swap's value changes when a particular forward rate bucket is bumped and noting futures can hedge forward-rate risk.

16. **Q:** Why are interpolation choices important?
    **A:** They change implied forwards and risk stability; different methods cause different forward-curve moves under perturbations.

17. **Q:** In an FRN PV formula (multi-curve), which curve discounts?
    **A:** The discount curve $P_d$ (OIS).

18. **Q:** In an FRN PV formula (multi-curve), which curve projects coupons?
    **A:** The projection curve $P_k$ for the floating index.

19. **Q:** Why might an FRN price differ from par in multi-curve?
    **A:** If projection forwards differ from discounting forwards, the telescoping par argument fails.

20. **Q:** What is the annuity $A_d$ in swap pricing?
    **A:** $A_d = \sum_i \alpha_i P_d(0, T_i)$

21. **Q:** Par swap rate equals what ratio in multi-curve?
    **A:** Discounted projected floating PV divided by discount-curve annuity.

22. **Q:** What are "index curves" in the reference text?
    **A:** Pseudo-discount curves used to represent forward Libor rates as observable indices rather than true discount rates.

23. **Q:** What is the role of OIS in determining risk-free rates?
    **A:** OIS rates help determine the risk-free zero curve used for valuing derivatives.

24. **Q:** What makes LIBOR problematic as a reference rate in Hull's text?
    **A:** It is based on bank estimates, not actual transactions.

25. **Q:** How are overnight-based reference rates different in observation timing?
    **A:** They are observed throughout the accrual period; midpoint-type adjustments may be needed for modeling.

26. **Q:** What is "basis risk" in a multi-index curve group?
    **A:** Risk that different tenor index curves do not move together; tied to basis swap spread perturbations.

27. **Q:** What is "overall rate level risk" in a spread-based curve group?
    **A:** Sensitivity to base index instruments that move all index curves together.

28. **Q:** Why do we build discount curve first in practice?
    **A:** Projection curve pricing of swaps uses discount factors; curve dependency graph typically requires discounting first.

29. **Q:** What is a simple sanity check for a curve build?
    **A:** Reprice all calibration instruments to their market quotes (within tolerances).

30. **Q:** What must you verify desk-by-desk?
    **A:** Exact index definitions (term vs compounded), day counts, compounding, CSA/collateral assumptions.

---

## Mini Problem Set

### Questions 1–8 (with solution sketches)

**1. (Warm-up)** Using $P_d(0, 0.25) = 0.9950$ and $P_d(0, 0.50) = 0.9900$ with $\tau = 0.25$, compute the discounting-implied forward $F_d(0; 0.25, 0.50)$.

**Sketch:** Use $F_d = \frac{1}{\tau}\left(\frac{P(0,0.25)}{P(0,0.50)} - 1\right) = 4\left(\frac{0.9950}{0.9900} - 1\right) = 4(0.00505) \approx 2.02\%$

---

**2. (Warm-up)** Given $P_{3M}(0, 0.25) = 0.9920$ and $P_{3M}(0, 0.50) = 0.9837$, compute $F_{3M}(0; 0.25, 0.50)$.

**Sketch:** Same formula with $P_{3M}$: $F_{3M} = 4\left(\frac{0.9920}{0.9837} - 1\right) = 4(0.00844) \approx 3.38\%$

---

**3. (Concept check)** Explain in one paragraph why a single curve cannot fit both OIS quotes and LIBOR FRA quotes if those imply different forwards.

**Sketch:** The same ratio $P(0,0.25)/P(0,0.50)$ would have to satisfy both $F_{\text{OIS}} = 2.02\%$ and $F_{\text{FRA}} = 3.40\%$; this is a contradiction since one ratio cannot equal two different values.

---

**4. (FRN PV)** Price a 6M FRN with two coupons using $F_1 = 3.0\%$, $F_2 = 3.2\%$, $\tau = 0.25$ each, and discount factors $P_d(0, 0.25) = 0.995$, $P_d(0, 0.50) = 0.990$. Notional $N = 100$.

**Sketch:**
- $C_1 = 100 \cdot 0.25 \cdot 0.030 = 0.75$, $C_2 = 100 \cdot 0.25 \cdot 0.032 = 0.80$
- $\text{PV} = 0.75 \cdot 0.995 + 0.80 \cdot 0.990 + 100 \cdot 0.990 = 0.746 + 0.792 + 99.0 = 100.538$

---

**5. (Par swap rate)** Compute the par fixed rate for a 1Y quarterly swap given forwards $(3.0\%, 3.1\%, 3.2\%, 3.3\%)$ and discount factors $(0.995, 0.990, 0.985, 0.980)$, with $\alpha = \tau = 0.25$.

**Sketch:**
- Floating PV = $\sum 0.25 \cdot F_i \cdot P_d(T_i)$
- Annuity = $0.25 \cdot (0.995 + 0.990 + 0.985 + 0.980) = 0.9875$
- $K_{\text{par}} = \text{Floating PV} / (100 \cdot 0.9875)$

---

**6. (Discount PV01 vs projection PV01)** For a 1Y swap, describe qualitatively what changes when you bump the discount curve by +1 bp holding projection fixed vs bump a projection FRA quote by +1 bp holding discount fixed.

**Sketch:** Discount bump changes PV weights (how cashflows are discounted) but not the projected cashflow amounts. Projection bump changes the actual cashflow amounts on the float leg but discounting remains unchanged.

---

**7. (Tenor basis par equation)** Write the par condition for a 3M vs 6M basis swap to maturity 1Y, identifying what is discounted and what is projected.

**Sketch:** PV(6M floating leg) = PV(3M floating leg + basis spread), where both legs are discounted with the same discount curve $P_d$, and each leg uses its own projection curve for forward rates.

---

**8. (Interpolation)** Given $P(0) = 1$ and $P(1) = 0.92$, compute $P(0.5)$ under (i) linear DF and (ii) log-linear DF, then compute $F(0; 0, 0.5)$ under each with $\tau = 0.5$.

**Sketch:**
- Linear DF: $P(0.5) = 0.96$, $F = 2(1/0.96 - 1) = 8.33\%$
- Log-linear: $P(0.5) = \sqrt{0.92} = 0.9592$, $F = 2(1/0.9592 - 1) = 8.51\%$

---

### Questions 9–16 (no sketches)

**9.** Construct a toy 3M projection curve from a 3M deposit and two FRAs, then price a 9M swap (quarterly) using a given OIS curve.

**10.** Show algebraically that in single-curve, a par FRN prices at par, and identify exactly which step fails when $P_k \neq P_d$.

**11.** Using a 3M curve and a 3M/6M basis swap quote, solve for a 6M forward rate assumption and compute an implied 6M DF at 0.5.

**12.** Compute PV sensitivity of a swap to a localized bump in one forward bucket (projection curve) vs a localized bump in one OIS DF node.

**13.** Explain how a spread-based curve group construction enables aggregation of "overall rates" vs "basis" risk.

**14.** Discuss why inconsistent market quotes can lead to unstable bootstraps, and propose a "quote cleaning" approach (conceptual).

**15.** Describe how you would test whether your multi-curve build introduces arbitrage opportunities (conceptual checks).

**16.** Extend the Example F approach to solve for two unknown forward segments using two swap quotes (piecewise-constant forwards).

---

## Source Map

### (A) Verified Facts — Specific Sources

| Fact | Source |
|------|--------|
| Post-crisis requires multiple curves | Andersen & Piterbarg Vol 1 Ch 6 |
| OIS rates used for risk-free/discounting | Hull Ch 4, Ch 7 |
| Forward rate definition via forward measure | Andersen & Piterbarg Vol 1 |
| Tenor basis ~1 bp pre-2007, up to 50 bp post-crisis | Andersen & Piterbarg Vol 1 Ch 6 |
| Practical rule: project at index curve, discount at OIS | Hull Ch 7 |
| Basis swap par condition (Eq 6.49) | Andersen & Piterbarg Vol 1 |
| LIBOR based on estimates not transactions | Hull |
| Index curves as pseudo-discount representation | Andersen & Piterbarg Vol 1 |
| Bucket exposures and forward-rate hedging | Tuckman Ch 5–7 |
| Interpolation effects on forward stability | Andersen & Piterbarg Vol 1 Ch 4–5 |

### (B) Reasoned Inference — Derivation Logic

| Inference | Derivation |
|-----------|------------|
| FRN deviates from par in multi-curve | Telescoping identity in §2.3(a) requires $P_k = P_d$; fails otherwise |
| Par swap rate is projection-weighted average | Follows from PV equality condition in §2.3(b) |
| Risk decomposition into discount vs projection | Follows from valuation formula structure: $P_d$ enters differently than $F_k$ |

### (C) Speculation — Flagged Uncertainties

| Topic | Uncertainty |
|-------|-------------|
| SOFR compounding conventions | I'm not sure about exact conventions across all markets; sources discuss general principles |
| Overnight rate observation timing adjustments | Hull mentions adjustments may be needed but exact methodology varies by market |

---

*Last Updated: January 2026*
