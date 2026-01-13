# Chapter 20: Tenor Basis — 1M vs 3M vs 6M and Basis Swap Logic

---

## Fact Classification

### (A) Verified Facts (Source-Backed)
- Multi-curve framework uses one discount curve $P_d$ and multiple tenor-specific projection curves $P^{(k)}$
- Tenor basis arises from credit/liquidity effects and supply/demand differences across funding horizons
- USD 1M–3M Libor basis was often <1bp in early 2007 but reached ~50bp in early 2009
- OIS discounting is used for collateralized trades
- Basis swaps exchange floating payments tied to different indices, plus or minus a spread on one leg
- Par basis spread $e_{1,2}(T)$ is determined so that the swap PV equals zero at inception

### (B) Reasoned Inference (Derived from A)
- Single-curve approaches fail when 3M and 6M instruments imply different discount factors for the same maturity
- Sequential bootstrapping of tenor curves from basis swaps follows naturally from the curve dependency structure
- Hedging a 6M swap with 3M instruments leaves residual basis exposure

### (C) Speculation (Clearly Labeled; Minimal)
- Market quoting conventions for specific currency/index pairs (see "I'm not sure" statements in Practical Notes)
- Exact fails-charge formulas and penalty mechanics not specified in sources

---

## Conventions & Notation

| Symbol | Meaning | Units/Type |
|--------|---------|------------|
| $t$ | Valuation time (mostly $t=0$) | Time |
| $T$ | Maturity | Time |
| $0 < t_1 < \cdots < t_n = T$ | Payment dates | Time grid |
| $P_d(t,T)$ | Discount factor from OIS/overnight collateral curve | Dimensionless |
| $P^{(k)}(t,T)$ | Tenor-$k$ index curve pseudo-discount factor (for forecasting only) | Dimensionless |
| $L^{(k)}(t;T_{i-1},T_i)$ | Tenor-$k$ forward rate for period $[T_{i-1},T_i]$ | Per year |
| $\tau_i^{(k)}$ | Year fraction for tenor-$k$ period $i$ | Dimensionless |
| $c$ | Fixed coupon rate in an IRS | Per year |
| $e_{1,2}(T)$ | Par basis spread at maturity $T$, added to leg 1 | Per year |
| $N$ | Swap notional | Currency |
| $F_i^{(k)}$ | Shorthand for $L^{(k)}(0;t_{i-1},t_i)$ | Per year |
| $A^{(k)}$ | Annuity (PV01) for tenor-$k$ payment schedule | Dimensionless |

### Defaults Used in Examples
- Day count: Simplified year fractions $\tau_{3M} = 0.25$, $\tau_{6M} = 0.50$
- Payment timing: Pay at period end; no notional exchanges
- Discounting: OIS discount curve $P_d(0,T)$
- Projection: Tenor-$k$ forward rates from tenor-$k$ index curves $P^{(k)}$
- Basis swap quote convention: Spread added to 3M leg when writing 6M vs (3M + spread)

---

## Core Concepts

### 1) Tenor and Tenor-Specific Indices (1M/3M/6M)

**Formal Definition:**
A **tenor** is the accrual length $\tau$ associated with a quoted simple rate over $[T, T+\tau]$. The simple forward rate $L(t,T,T+\tau)$ is defined via:

$$1 + \tau L(t,T,T+\tau) = \frac{1}{P(t,T,T+\tau)}$$

where $P(t,T,T+\tau) = P(t,T+\tau)/P(t,T)$ is a forward bond price over that interval.

For interbank term rates (Libor-like), tenors range from short to long (e.g., one week to 12 months) and are used as underlyings for floating-rate derivatives.

**Intuition:**
"1M", "3M", "6M" are not just calendar labels: they represent different funding horizons and market segments. Even within one currency, the economics of borrowing/lending for 1 month can differ from 6 months, especially under stress.

**Trading / Risk / Portfolio Practice:**
Portfolios referencing different tenors (e.g., assets indexed to 3M and liabilities to 6M) naturally create tenor mismatch. Basis swaps are the standard instrument to transfer/hedge this mismatch.

---

### 2) Discounting Curve vs Projection Curve (Multi-Curve World)

**Formal Definition:**
In a **multi-index curve group**, one uses:
- A single curve $P_d(t,T)$ for **discounting** (computing present values)
- Multiple index curves $P^{(1)}(t,T), \ldots, P^{(k)}(t,T)$ to **forecast** different Libor (or Libor-like) tenors

**Intuition:**
- **Discounting** answers: "How much is \$1 at $T$ worth today?"
- **Projection** answers: "What is the expected floating coupon for a given index over the accrual period?"

**Trading / Risk / Portfolio Practice:**
OIS discounting is used for collateralized trades. One must often build a discount curve from OIS and separate forecasting curves from Libor instruments and basis swaps between overnight and Libor.

---

### 3) Tenor Basis (1M vs 3M vs 6M)

**Formal Definition:**
**Tenor basis** is the market-observed difference between otherwise "comparable" floating rates of different tenors (e.g., 1M vs 3M vs 6M), expressed through:
- Basis swap spreads
- The fact that separate index curves $P^{(k)}$ are needed for different tenors

**Intuition:**
Different tenors embed different credit/liquidity/funding components, and supply/demand across funding horizons can diverge.

**Trading / Risk / Portfolio Practice:**
- Tenor basis is partly attributed to credit and liquidity effects and differing supply/demand for funding at different tenors
- The 1M–3M USD Libor basis was often <1bp in early 2007 but reached ~50bp in early 2009
- Basis tends to increase with the tenor difference and with maturity
- Basis is bootstrappable from quoted basis swap and OIS markets

---

### 4) Basis Swap (Single-Currency, Floating–Floating)

**Formal Definition:**
A **basis swap** exchanges floating payments tied to one index against floating payments tied to another index, plus or minus a spread on one leg.

In the tenor-basis setting, $L^{(1)}$ and $L^{(2)}$ can be different-tenor Libor indices (e.g., 3M vs 6M). The maturity-$T$ par basis swap exchanges payments of $L^{(2)}$ against $L^{(1)} + e_{1,2}(T)$, with $e_{1,2}(T)$ determined so PV = 0.

**Intuition:**
The basis spread is the "price" of converting cashflows indexed to one tenor into cashflows indexed to another tenor.

**Trading / Risk / Portfolio Practice:**
- Basis swaps hedge exposure when "assets and liabilities have interest payments based on different floating rates"
- Dealers often prefer hedging/quoting in the more liquid "spread" instruments (basis swaps) rather than building each tenor curve purely from outright tenor swaps

---

### 5) Relationship Between Tenor Curves: "Basis as a Curve"

**Formal Definition:**
It can be convenient to build one tenor index curve as a spread off another:

$$P^{(2)}(t,T) = P^{(1)}(t,T) \exp\!\left(-\int_0^T \eta_{1,2}(s)\,ds\right)$$

where $\eta_{1,2}$ is a (term-structure) basis spread process used for curve construction.

**Intuition:**
Rather than re-solving an entire curve from scratch, you treat the less-liquid tenor curve as "base tenor curve + basis term structure."

**Trading / Risk / Portfolio Practice:**
This motivates a dependency graph: build $P_d$, then a primary forecast curve $P^{(1)}$, then derive secondary tenors via basis instruments. Sequential construction of multiple-tenor basis curves is used because basis markets can be more liquid than outright markets for some tenors/maturities.

---

## Math and Derivations

### 2.1 From Discount Factors to Simple Forward Rates (Single-Curve Identity)

The simple forward rate over $[T, T+\tau]$ is defined by:

$$\boxed{1 + \tau L(t,T,T+\tau) = \frac{1}{P(t,T,T+\tau)}, \quad P(t,T,T+\tau) = \frac{P(t,T+\tau)}{P(t,T)}}$$

For a grid $T = T_0 < T_1 < \cdots < T_n$, the compounding relationship is:

$$\frac{P(t,T_n)}{P(t,T)} = \prod_{i=1}^{n} \frac{1}{1 + (T_i - T_{i-1}) L(t,T_{i-1},T_i)}$$

**Unit Check:**
- $P(\cdot,\cdot)$: dimensionless
- $L(\cdot)$: "per year"
- $(T_i - T_{i-1}) L$: dimensionless
- Product of dimensionless terms is dimensionless ✓

**Key Implication (Toy Contradiction):**
If a *single* curve must be consistent with both 3M and 6M quotes, then the 6M discount factor implied by two 3M forwards must match the 6M discount factor implied directly by the 6M quote. If not, one curve cannot fit both simultaneously.

---

### 2.2 Multi-Curve Projection: Index Curve Definition for Tenor-$k$

In a multi-index setting, the expected Libor fixing used in swap valuation is identified with a forward rate computed from a tenor-specific index curve:

$$\boxed{\mathbb{E}^{t_{i+1}^{(k)}}_t\!\big[\,L^{(k)}(t_i,t_{i+1}^{(k)})\,\big] = L^{(k)}(t; t_i,t_{i+1}^{(k)}) = \frac{1}{\tau_i^{(k)}}\left(\frac{P^{(k)}(t,t_i)}{P^{(k)}(t,t_{i+1}^{(k)})}-1\right)}$$

**Interpretation:**
$P^{(k)}$ is not "the" discount curve; it is a curve object used solely to infer forwards for tenor $k$.

---

### 2.3 Fixed–Float Swap PV in a Multi-Curve World and Par Fixed Rate

The value of a fixed–floating swap (fixed rate $c$ against $k$-Libor) is:

$$V^{(k)}(t) = \sum_{i=0}^{n^{(k)}-1} c\,\tau_i^{(k)}\,P_d(t,t_{i+1}^{(k)}) - \sum_{i=0}^{n^{(k)}-1} \mathbb{E}^{t_{i+1}^{(k)}}_t\!\big[\,L^{(k)}(t_i,t_{i+1}^{(k)})\,\big]\, \tau_i^{(k)}\,P_d(t,t_{i+1}^{(k)})$$

with the expectation replaced by the forward computed from $P^{(k)}$.

At $t=0$, write forwards $F_i^{(k)} := L^{(k)}(0;t_i^{(k)},t_{i+1}^{(k)})$. Then:

$$V^{(k)}(0) = \sum_{i=0}^{n^{(k)}-1} \tau_i^{(k)} P_d(0,t_{i+1}^{(k)})\,(c - F_i^{(k)})$$

**Par fixed rate** $c^\star$ satisfies $V^{(k)}(0) = 0$, hence:

$$\boxed{c^\star = \frac{\sum_{i=0}^{n^{(k)}-1} F_i^{(k)}\,\tau_i^{(k)}\,P_d(0,t_{i+1}^{(k)})}{\sum_{i=0}^{n^{(k)}-1} \tau_i^{(k)}\,P_d(0,t_{i+1}^{(k)})}}$$

**Sanity Checks:**
- If all forwards $F_i^{(k)}$ are equal to a constant $F$, then $c^\star = F$ ✓
- In a single-curve world where projection = discounting, many texts show the floating leg PV reduces to a simple expression; Tuckman notes that at initiation the floating leg is worth par. In a multi-curve setting, that par-floating identity generally **fails** because $P^{(k)} \neq P_d$.

---

### 2.4 Basis Swap PV Equation and Par Basis Spread

The tenor basis swap definition (two Libor indices $L_1$ and $L_2$) is given by the par PV condition:

$$\sum_{i=0}^{n_2(T)-1} L_2(t;t_i^2,t_{i+1}^2)\,\tau_i^2\,P_d(t,t_{i+1}^2) = \sum_{i=0}^{n_1(T)-1} \big(L_1(t;t_i^1,t_{i+1}^1) + e_{1,2}(T)\big)\,\tau_i^1\,P_d(t,t_{i+1}^1)$$

where $e_{1,2}(T)$ is quoted on leg 1 and can be positive or negative.

Solving for the par spread at $t=0$ gives:

$$\boxed{e_{1,2}(T) = \frac{\sum_{i=0}^{n_2(T)-1} F_i^2\,\tau_i^2\,P_d(0,t_{i+1}^2) - \sum_{i=0}^{n_1(T)-1} F_i^1\,\tau_i^1\,P_d(0,t_{i+1}^1)}{\sum_{i=0}^{n_1(T)-1} \tau_i^1\,P_d(0,t_{i+1}^1)}}$$

**Unit Check:**
- Numerator: PV "coupon sum" (dimensionless per 1 notional)
- Denominator: annuity (dimensionless)
- Ratio: rate (per year) ✓

---

### 2.5 Bootstrapping a Secondary Tenor Curve from Basis Swaps

Tenor basis swaps can be used to construct one tenor curve from another. Build each curve "as a spread off a base curve":

$$P^{(2)}(t,T) = P^{(1)}(t,T) \exp\!\left(-\int_0^T \eta_{1,2}(s)\,ds\right)$$

with sequential construction across multiple tenors.

**Practical Bootstrapping Logic (Discrete):**

1. Choose a base tenor (often the most liquid) and bootstrap $P^{(1)}$ from its outright instruments

2. For the next tenor, take a basis swap quote $e_{1,2}(T)$ and solve the par PV equation for the unknown forward(s) of tenor 2 at the next maturity node(s)

3. Convert solved forwards into the new curve's pseudo-discount factors using:

$$\boxed{P^{(2)}(0,t_{i+1}) = \frac{P^{(2)}(0,t_i)}{1 + \tau_i^2 F_i^2}}$$

This is just a rearrangement of the simple forward definition.

---

## Measurement & Risk

### 3.1 What Tenor Basis Is and Why 1M/3M/6M Forward Curves Differ

Beyond the OIS-vs-Libor split, we must also account for tenor basis between swaps trading at different frequencies (e.g., 1M, 3M, 6M).

Tenor basis is partly attributed to:
- Credit and liquidity effects
- Different supply/demand for funding across tenors
- Historical widening of USD 1M–3M basis up to ~50bp in early 2009

This motivates separate projection curves $P^{(1M)}$, $P^{(3M)}$, $P^{(6M)}$ rather than a single curve.

### 3.2 What a Basis Swap Is and How It's Quoted

A basis swap exchanges two floating legs tied to different indices, and "involves exchanging a floating rate in one tenor for a floating rate in another tenor plus or minus a spread."

**Which leg receives the spread?**
- **Source convention (default here):** the spread $e_{1,2}(T)$ is quoted and added on leg 1 in the exchange $L_2$ vs $L_1 + e_{1,2}(T)$
- **Market reality:** conventions can vary by currency, index family, and dealer standard

> **I'm not sure** what the market quoting convention is for your specific currency/index (e.g., USD 3M vs 6M term index, EURIBOR tenors, etc.).
>
> **Needed to be certain:** (i) currency, (ii) exact indices (e.g., USD term vs legacy Libor), and (iii) dealer quote convention ("spread on 3M" vs "spread on 6M", payer/receiver sign).

### 3.3 Why Tenor Basis Exists (Source-Consistent Explanations Only)

Modern basis behavior is linked to the post-crisis multi-curve world:
- OIS discounting for collateralized trades implies discounting with an OIS curve $P_d$ and forecasting with a separate term-index curve $P^{(L)}$
- Divergence between overnight rates and Libor (and between Libor tenors) reflects credit/liquidity/funding segmentation
- Fed Funds used for collateral accrual and changes in bank funding conditions contributed to basis moves
- Tenor basis can be bootstrapped explicitly from basis swap markets

### 3.4 Practical Consequences for Valuation and Risk

**Valuation:**
- Discount cashflows using $P_d(0,\cdot)$ (OIS/overnight discount curve in the collateralized setting)
- Forecast each floating leg using its own tenor curve $P^{(k)}$ via forward rates

**Risk Decomposition:**
Swap sensitivities can be expressed as sensitivities to the underlying curves. In the multi-index setting, one has separate risks to:
- The discount curve
- The OIS–Libor basis
- The basis between multiple tenor curves

**Practically:** PV01 to discount curve $P_d$, and "basis risks" to each projection curve $P^{(1M)}, P^{(3M)}, P^{(6M)}$ and to quoted basis spreads.

**Hedging:**
Hedging a 6M-referencing position with 3M instruments can leave residual exposure to the 3M–6M basis. Basis swaps are the direct hedge instruments for that risk (see Example J).

---

## Worked Examples

### Common Toy Conventions (Apply to Examples A–J Unless Overridden)

| Convention | Value |
|------------|-------|
| Day count | Simplified: $\tau_{3M} = 0.25$, $\tau_{6M} = 0.50$ |
| Payment timing | Pay at period end; no notional exchanges |
| Discounting | OIS discount curve $P_d(0,T)$ |
| Projection | Tenor-$k$ forward rates from $P^{(k)}$ |
| Basis swap quote | Spread added to 3M leg in "6M vs (3M + spread)" |

---

### Example A — Single-Curve Contradiction Toy (3M vs 6M Cannot Both Fit One Curve)

**Goal:** Construct toy market quotes where 3M instruments and 6M instruments imply different discount factors/forwards, demonstrating that one curve cannot fit both.

**Conventions:** Simple interest, no-arbitrage link between forwards and discount factors:

$$\frac{P(0,T_n)}{P(0,T_0)} = \prod_{i=1}^{n} \frac{1}{1 + \Delta T_i\,L(0,T_{i-1},T_i)}$$

**Toy Market Quotes:**

**3M deposit (0–3M):** $L_{3M}(0;0,0.25) = 2.20\%$

$$\Rightarrow 1 + 0.25L = 1 + 0.25(0.022) = 1.0055$$
$$\Rightarrow P(0,0.25) = \frac{1}{1.0055} = 0.9945300845$$

**3M FRA (3M–6M):** $L_{3M}(0;0.25,0.50) = 2.40\%$

$$\Rightarrow 1 + 0.25L = 1.0060$$
$$\Rightarrow P(0,0.50) = \frac{P(0,0.25)}{1.0060} = \frac{0.9945300845}{1.0060} = 0.9885984936$$

**6M deposit (0–6M) quote:** $L_{6M}(0;0,0.50) = 2.50\%$

$$\Rightarrow 1 + 0.50L = 1 + 0.50(0.025) = 1.0125$$
$$\Rightarrow P(0,0.50) = \frac{1}{1.0125} = 0.9876543210$$

**Contradiction:**
- From 3M instruments: $P(0,0.50) = 0.9885984936$
- From 6M instrument: $P(0,0.50) = 0.9876543210$

They disagree by:
$$0.9885984936 - 0.9876543210 = 0.0009441726$$

Equivalently, the 6M forward implied by the 3M-derived $P(0,0.50)$ is:

$$L_{implied}^{6M}(0;0,0.50) = \frac{1}{0.50}\left(\frac{1}{0.9885984936} - 1\right) = 2.3066\%$$

which differs from the quoted 2.50%.

**Conclusion:** A single curve object cannot fit both tenors; hence separate tenor curves (or basis instruments) are required in a multi-curve setting.

---

### Example B — Define Two Projection Curves: Compute 3M Forwards from $P_d$ and $P^{(3M)}$

**Goal:** Given an OIS discount curve $P_d(0,T)$ and a 3M projection curve $P^{(3M)}(0,T)$, compute 3M forwards $F_i^{3M}$.

**Conventions:**
- Discounting uses $P_d$
- Projection uses $P^{(3M)}$
- Forward rate from index curve:
$$F^{3M}(0;T_{i-1},T_i) = \frac{1}{\tau}\left(\frac{P^{(3M)}(0,T_{i-1})}{P^{(3M)}(0,T_i)} - 1\right)$$

**Toy Curves:**

| $T$ | $P_d(0,T)$ | $P^{(3M)}(0,T)$ |
|-----|------------|-----------------|
| 0.00 | 1.0000000000 | 1.0000000000 |
| 0.25 | 0.9950000000 | 0.9945300845 |
| 0.50 | 0.9900250000 | 0.9885984936 |
| 0.75 | 0.9850748750 | 0.9822141019 |
| 1.00 | 0.9801495006 | 0.9753863971 |

**Compute Forwards (units: per year):**

**Period $[0, 0.25]$**, $\tau = 0.25$:
$$F_1^{3M} = \frac{1}{0.25}\left(\frac{1}{0.9945300845} - 1\right) = 0.0220 = 2.20\%$$

**Period $[0.25, 0.50]$**:
$$F_2^{3M} = \frac{1}{0.25}\left(\frac{0.9945300845}{0.9885984936} - 1\right) = 0.0240 = 2.40\%$$

**Period $[0.50, 0.75]$**:
$$F_3^{3M} = \frac{1}{0.25}\left(\frac{0.9885984936}{0.9822141019} - 1\right) = 0.0260 = 2.60\%$$

**Period $[0.75, 1.00]$**:
$$F_4^{3M} = \frac{1}{0.25}\left(\frac{0.9822141019}{0.9753863971} - 1\right) = 0.0280 = 2.80\%$$

**Sanity Check:** Each coupon amount for notional 1 is $F_i^{3M} \tau$: $0.0055, 0.0060, 0.0065, 0.0070$, i.e., dimensionless "interest fractions" ✓

---

### Example C — Value a 3M Swap in Multi-Curve: OIS Discounting + 3M Projection

**Goal:** Compute the 1Y par fixed rate for a fixed vs 3M swap, discounting by $P_d$ and projecting by $P^{(3M)}$.

**Conventions:**
- Maturity $T = 1$ year
- Floating: 3M payments at $0.25, 0.50, 0.75, 1.00$, $\tau = 0.25$
- Fixed: quarterly payments on the same dates (simplification)
- Notional: report per $N = 1$

**Step 1: PV of Projected 3M Floating Leg**

$$\text{PV}_{3M} = \sum_{i=1}^{4} F_i^{3M}\,\tau\,P_d(0,t_i)$$

| $i$ | $F_i^{3M}$ | $\tau$ | $P_d(0,t_i)$ | Term |
|-----|------------|--------|--------------|------|
| 1 | 0.022 | 0.25 | 0.995 | $0.0055 \times 0.995 = 0.0054725$ |
| 2 | 0.024 | 0.25 | 0.990025 | $0.0060 \times 0.990025 = 0.00594015$ |
| 3 | 0.026 | 0.25 | 0.985074875 | $0.0065 \times 0.985074875 = 0.0064029866875$ |
| 4 | 0.028 | 0.25 | 0.9801495006 | $0.0070 \times 0.9801495006 = 0.0068610465044$ |

$$\text{PV}_{3M} = 0.0246766831919$$

**Step 2: Fixed-Leg Annuity**

$$A_{3M} = \sum_{i=1}^{4} \tau\,P_d(0,t_i) = 0.25(0.995 + 0.990025 + 0.985074875 + 0.9801495006) = 0.9875623439063$$

**Step 3: Par Fixed Rate**

From the multi-curve swap PV formula, par means PV fixed = PV float:

$$c_{3M}^\star = \frac{\text{PV}_{3M}}{A_{3M}} = \frac{0.0246766831919}{0.9875623439063} = 0.02498746873 = \boxed{2.498746873\%}$$

**Repricing Check (per notional 1):**
PV fixed at $c^\star$: $c^\star A_{3M} = 0.02498746873 \times 0.9875623439063 = 0.0246766831919$, equals PV float ✓

**Multi-Curve "Par Floater" Contrast:**
Single-curve intuition says a floater is near par. If we compute $1 - P_d(0,1)$:
$$1 - P_d(0,1) = 1 - 0.9801495006 = 0.0198504994$$

which is **not** equal to $\text{PV}_{3M} = 0.0246766832$. This illustrates that when discounting and projection curves differ, the floating leg PV is not simply a function of $P_d$ alone.

---

### Example D — Value a 6M Swap in Multi-Curve: OIS Discounting + 6M Projection

**Goal:** Repeat Example C for a fixed vs 6M swap; compare par rates.

**Conventions:**
- Maturity $T = 1$
- Floating: 6M payments at $0.5, 1.0$, $\tau = 0.5$
- Fixed: semiannual payments at $0.5, 1.0$ (toy)
- 6M forwards (toy): $F_1^{6M}(0;0,0.5) = 2.50\%$, $F_2^{6M}(0;0.5,1.0) = 2.70\%$
- Notional: report per $N = 1$

**Step 1: PV of 6M Floating Leg**

Coupon amounts:
- Period 1: $0.025 \times 0.5 = 0.0125$
- Period 2: $0.027 \times 0.5 = 0.0135$

PV:
- $0.0125 \times P_d(0,0.5) = 0.0125 \times 0.990025 = 0.0123753125$
- $0.0135 \times P_d(0,1.0) = 0.0135 \times 0.9801495006 = 0.0132320182584$

$$\text{PV}_{6M} = 0.0256073307584$$

**Step 2: Fixed-Leg Annuity**

$$A_{6M} = \sum_{i=1}^{2} \tau\,P_d(0,t_i) = 0.5(0.990025 + 0.9801495006) = 0.9850872503125$$

**Step 3: Par Fixed Rate**

$$c_{6M}^\star = \frac{\text{PV}_{6M}}{A_{6M}} = \frac{0.0256073307584}{0.9850872503125} = 0.02599498750 = \boxed{2.599498750\%}$$

**Interpretation:**
Par fixed rates differ: $c_{6M}^\star \approx 2.5995\%$ vs $c_{3M}^\star \approx 2.4987\%$, reflecting different projection curves (tenor basis).

---

### Example E — Define Basis Swap PV Equation (Par Quote) for 3M–6M Basis

**Goal:** Write the par PV equation that determines the basis spread for a 3M–6M basis swap.

**Conventions:**
- Maturity $T$
- Leg 1 = 3M, Leg 2 = 6M
- Spread on 3M leg (leg 1), consistent with the source's definition $L_2$ vs $L_1 + e_{1,2}(T)$
- Discounting by $P_d$

**Par Condition:**
Let payment dates be $\{t_i^{3M}\}_{i=1}^{n_3}$ and $\{t_j^{6M}\}_{j=1}^{n_6}$. At $t=0$, the par basis spread $s_{3M,6M}(T)$ satisfies:

$$\sum_{j=1}^{n_6} F_j^{6M}\,\tau_j^{6M}\,P_d(0,t_j^{6M}) = \sum_{i=1}^{n_3} \big(F_i^{3M} + s_{3M,6M}(T)\big)\,\tau_i^{3M}\,P_d(0,t_i^{3M})$$

**Solving:**

$$\boxed{s_{3M,6M}(T) = \frac{\sum_{j=1}^{n_6} F_j^{6M}\tau_j^{6M}P_d(0,t_j^{6M}) - \sum_{i=1}^{n_3} F_i^{3M}\tau_i^{3M}P_d(0,t_i^{3M})}{\sum_{i=1}^{n_3} \tau_i^{3M}P_d(0,t_i^{3M})}}$$

---

### Example F — Compute a 3M–6M Par Basis Spread from Curves (Numeric)

**Goal:** Use the toy curves from Examples B–D and compute the par basis spread.

**Conventions:**
- Maturity $T = 1$
- Discounting: $P_d$ as given
- 3M forwards: $2.2\%, 2.4\%, 2.6\%, 2.8\%$
- 6M forwards: $2.5\%, 2.7\%$
- Spread on 3M leg

**Step 1: Compute PV of Each Floating Leg (per notional 1)**

From Examples C and D:
- $\text{PV}_{3M} = 0.0246766831919$
- $\text{PV}_{6M} = 0.0256073307584$

**Step 2: Compute 3M Annuity**

$$A_{3M} = \sum_{i=1}^{4} \tau P_d(0,t_i) = 0.9875623439063$$

**Step 3: Solve for Spread**

$$s_{3M,6M} = \frac{\text{PV}_{6M} - \text{PV}_{3M}}{A_{3M}} = \frac{0.0256073307584 - 0.0246766831919}{0.9875623439063} = 0.0009423684209$$

Convert to basis points:
$$s_{3M,6M} \approx 0.0009423684 \times 10{,}000 = \boxed{9.42368421 \text{ bp}}$$

**Interpretation:**
Under this convention, the 6M leg is "richer" (higher PV) than the 3M leg, so the 3M leg must pay/receive an added +9.42bp (depending on payer/receiver orientation) to make PVs equal.

---

### Example G — Bootstrap One Curve Using Basis Swaps

**Goal:** Illustrate a sequential bootstrap step: use discount curve + calibrated 3M projection curve + one 3M–6M basis quote to infer the next 6M forward (and thus a 6M curve node).

**Conventions:**
- Maturity $T = 1$
- Discount curve $P_d$ known
- 3M curve $P^{(3M)}$ known → 3M forwards known
- 6M curve unknown beyond 0.5y
- 6M deposit gives $F_1^{6M}(0;0,0.5) = 2.50\%$ (so $P^{(6M)}(0,0.5)$ known)
- Basis quote: $s_{3M,6M}(1Y) = 9.42368421$ bp, spread on 3M leg

**Step 0: Known 6M Node at 0.5y**

From 6M deposit:
$$P^{(6M)}(0,0.5) = \frac{1}{1 + 0.5 \times 0.025} = \frac{1}{1.0125} = 0.9876543210$$

**Step 1: Compute RHS PV of (3M + spread) Leg**

- $\text{PV}_{3M} = 0.0246766831919$ (Example C)
- Spread PV per notional:
$$\text{PV}_{spr} = s \cdot A_{3M} = 0.0009423684209 \times 0.9875623439063 = 0.0009306475666$$

Total RHS:
$$\text{PV}_{3M+spr} = 0.0246766831919 + 0.0009306475666 = 0.0256073307584$$

**Step 2: Write LHS PV for 6M Leg with Unknown Second Forward**

$$\text{PV}_{6M} = F_1^{6M} \cdot 0.5 \cdot P_d(0,0.5) + F_2^{6M} \cdot 0.5 \cdot P_d(0,1.0)$$

Known first term:
$$F_1^{6M} \cdot 0.5 \cdot P_d(0,0.5) = 0.025 \times 0.5 \times 0.990025 = 0.0123753125$$

Set par PV:
$$0.0123753125 + F_2^{6M} \cdot 0.5 \cdot 0.9801495006 = 0.0256073307584$$

Solve:
$$F_2^{6M} = \frac{0.0256073307584 - 0.0123753125}{0.5 \times 0.9801495006} = \frac{0.0132320182584}{0.4900747503} = 0.0270 = \boxed{2.70\%}$$

**Step 3: Convert to 6M Curve Node at 1.0y**

$$P^{(6M)}(0,1.0) = \frac{P^{(6M)}(0,0.5)}{1 + 0.5 F_2^{6M}} = \frac{0.9876543210}{1 + 0.5 \times 0.027} = \frac{0.9876543210}{1.0135} = 0.9744985900$$

---

### Example H — Locality of Basis Quote Bumps

**Goal:** Show how a bump to one basis swap quote impacts the inferred 6M curve node (locality).

**Conventions:** Same as Example G, except bump the 1Y 3M–6M basis quote by +1bp:
$$s_{new} = s_{old} + 0.0001$$

**Before Bump (from Example G):**
- $s_{old} = 0.0009423684209$
- $F_2^{6M} = 2.700000\%$
- $P^{(6M)}(0,1.0) = 0.9744985900$

**After Bump:**

RHS PV increases by:
$$\Delta\text{PV} = 0.0001 \times A_{3M} = 0.0001 \times 0.9875623439063 = 0.0000987562343906$$

New par equation:
$$0.0123753125 + F_{2,new}^{6M} \cdot 0.5 \cdot 0.9801495006 = 0.0256073307584 + 0.0000987562343906 = 0.0257060869928$$

Solve:
$$F_{2,new}^{6M} = \frac{0.0257060869928 - 0.0123753125}{0.4900747503} = 0.02720151259 = 2.720151259\%$$

Update 6M node:
$$P_{new}^{(6M)}(0,1.0) = \frac{0.9876543210}{1 + 0.5 \times 0.02720151259} = 0.9744017207$$

**Before/After Table:**

| Quantity | Before | After (+1bp basis quote) | Change |
|----------|--------|--------------------------|--------|
| Basis quote $s_{3M,6M}(1Y)$ | 9.423684 bp | 10.423684 bp | +1.000000 bp |
| $F_2^{6M}(0;0.5,1.0)$ | 2.700000% | 2.720151% | +2.015126 bp |
| $P^{(6M)}(0,1.0)$ | 0.9744985900 | 0.9744017207 | −0.0000968693 |

**Interpretation:**
Bumping one basis quote affects the inferred adjacent 6M forward/node used to fit that quote. This is the "locality" practitioners aim for with sequential bootstrapping (though interpolation choices can spread the impact).

---

### Example I — Risk Decomposition for a Basis Swap (Finite-Difference PV01s)

**Goal:** For a 1Y 3M–6M basis swap, compute sensitivities (finite difference):
- Discount-curve PV01
- 3M-projection PV01
- 6M-projection PV01
- Basis-spread sensitivity (PV per 1bp of quoted basis)

**Conventions:**
- Instrument: 1Y basis swap with spread on 3M leg
- Position: Receive 6M, Pay (3M + spread)
- Notional: $N = 100{,}000{,}000$
- Base spread: par $s = 9.42368421$ bp (Example F), so base PV = 0
- Discount bump definition (toy): apply "+1bp" bump by multiplying each $P_d(0,t)$ by $(1 - 0.0001\,t)$

**Step 1: Base PV (Check)**

Per notional 1:
$$\text{PV}_{6M} = 0.0256073307584, \quad \text{PV}_{3M+spr} = 0.0256073307584 \Rightarrow \text{PV} = 0 \checkmark$$

#### I.1 Discount-Curve PV01 (Hold Forwards and Spread Fixed; Bump $P_d$)

**Bumped Discount Factors:**

| $T$ | $P_d'(0,T)$ |
|-----|-------------|
| 0.25 | $0.995(1 - 0.000025) = 0.9949751250$ |
| 0.50 | $0.990025(1 - 0.00005) = 0.9899754988$ |
| 0.75 | $0.985074875(1 - 0.000075) = 0.9850009944$ |
| 1.00 | $0.9801495006(1 - 0.0001) = 0.9800514857$ |

**Recompute PVs (per notional 1):**

6M leg:
- $0.0125 \times 0.9899754988 = 0.0123746937344$
- $0.0135 \times 0.9800514857 = 0.0132306950566$
- $\text{PV}_{6M}' = 0.0256053887910$

3M leg (coupons 0.0055, 0.0060, 0.0065, 0.0070):
- $0.0055 \times 0.9949751250 = 0.0054723631875$
- $0.0060 \times 0.9899754988 = 0.0059398529925$
- $0.0065 \times 0.9850009944 = 0.0064025064635$
- $0.0070 \times 0.9800514857 = 0.0068603603997$
- $\text{PV}_{3M}' = 0.0246750830432$

Spread PV:
$$A_{3M}' = 0.25(0.9949751250 + 0.9899754988 + 0.9850009944 + 0.9800514857) = 0.9875007759523$$
$$\text{PV}_{spr}' = s \cdot A_{3M}' = 0.0009423684209 \times 0.9875007759523 = 0.0009305895469$$

Pay leg total:
$$\text{PV}_{3M+spr}' = 0.0246750830432 + 0.0009305895469 = 0.0256056725901$$

Basis swap PV (receive 6M, pay 3M+spr):
$$\text{PV}' = 0.0256053887910 - 0.0256056725901 = -2.837991 \times 10^{-7}$$

**Dollar PV01:**
$$\text{PV01}_{disc} = \text{PV}' \times N \approx -2.837991 \times 10^{-7} \times 100{,}000{,}000 = \boxed{-\$28.38}$$

#### I.2 3M-Projection PV01 (Bump 3M Forwards +1bp)

A +1bp bump means each 3M coupon increases by $\tau \times 0.0001 = 0.25 \times 0.0001 = 0.000025$.

PV change per notional:
$$\Delta\text{PV}_{3M} = 0.000025 \times (P_d(0,0.25) + P_d(0,0.5) + P_d(0,0.75) + P_d(0,1))$$
$$= 0.000025 \times 3.950249375625 = 0.0000987562343906$$

Since we pay (3M + spread), the basis swap PV change is $-\Delta\text{PV}_{3M}$:
$$\text{PV01}_{3M\,proj} = -0.0000987562343906 \times 100{,}000{,}000 = \boxed{-\$9{,}875.62}$$

#### I.3 6M-Projection PV01 (Bump 6M Forwards +1bp)

Each 6M coupon increases by $\tau \times 0.0001 = 0.5 \times 0.0001 = 0.00005$.

PV change per notional:
$$\Delta\text{PV}_{6M} = 0.00005 \times (P_d(0,0.5) + P_d(0,1.0)) = 0.00005 \times 1.970174500625 = 0.0000985087250313$$

We receive 6M, so PV increases:
$$\text{PV01}_{6M\,proj} = +0.0000985087250313 \times 100{,}000{,}000 = \boxed{+\$9{,}850.87}$$

#### I.4 Basis-Spread Sensitivity (PV per 1bp on the Quoted Spread)

Spread PV per notional is $s \cdot A_{3M}$. A +1bp change means $\Delta s = 0.0001$:
$$\Delta\text{PV}_{spr} = 0.0001 \times A_{3M} = 0.0001 \times 0.9875623439063 = 0.0000987562343906$$

We pay the spread, so PV change is negative:
$$\frac{\partial \text{PV}}{\partial s}\bigg|_{1bp} = -0.0000987562343906 \times 100{,}000{,}000 = \boxed{-\$9{,}875.62 \text{ per 1bp}}$$

#### I.5 Summary Table (for $N = 100$mm)

| Risk Component | Definition (Toy FD) | Result |
|----------------|---------------------|--------|
| Discount PV01 | bump $P_d(0,t) \to P_d(0,t)(1-0.0001t)$ | $-\$28.38$ |
| 3M projection PV01 | bump all $F_i^{3M}$ by +1bp | $-\$9{,}875.62$ |
| 6M projection PV01 | bump all $F_j^{6M}$ by +1bp | $+\$9{,}850.87$ |
| Basis-spread sensitivity | bump quoted $s$ by +1bp | $-\$9{,}875.62$ |

**Hedging Interpretation:**
The dominant sensitivities are to the projection curves (and the quoted basis spread), consistent with the idea that basis swaps are primarily basis-risk instruments rather than pure discount-curve DV01 trades.

---

### Example J — Hedging Residual: Hedge a 6M Swap with a 3M Swap, Then Shock the Basis

**Goal:** Show that hedging a 6M swap using a 3M swap can neutralize "overall DV01" (common move), but a basis widening still produces residual P&L.

**Conventions:**
- Discount curve $P_d$ fixed
- Two par swaps, maturity 1Y:
  - 6M swap fixed rate $c_{6M}^\star = 2.599498750\%$ (Example D)
  - 3M swap fixed rate $c_{3M}^\star = 2.498746873\%$ (Example C)
- "Overall DV01" shock: both 3M and 6M forward curves shift up by +1bp (basis unchanged)
- Basis-widening scenario: 6M forwards shift up by +5bp, 3M forwards unchanged (basis widens)

**Positions:**
- **Trade 1 (target):** Pay fixed $c_{6M}^\star$, Receive 6M, notional $N_1 = 100{,}000{,}000$ (PV=0 at inception)
- **Trade 2 (hedge):** Receive fixed $c_{3M}^\star$, Pay 3M, notional $N_2$ to be chosen (PV=0 at inception)

#### J.1 Compute Each Swap's PV Change Under a Common +1bp Move

For a par fixed–float swap, holding discount factors fixed, a parallel bump $\Delta$ to all forwards changes the floating leg PV by:
$$\Delta\text{PV}_{float} = \Delta \sum_i \tau_i P_d(0,t_i) = \Delta \cdot A$$

**Trade 1 (receive 6M float) DV01:**
$$\text{DV01}_1 = N_1 \cdot 0.0001 \cdot A_{6M} = 100{,}000{,}000 \times 0.0001 \times 0.9850872503125 = +\$9{,}850.87$$

**Trade 2 (pay 3M float) DV01:**
$$\text{DV01}_2 = -N_2 \cdot 0.0001 \cdot A_{3M} = -N_2 \times 0.0001 \times 0.9875623439063$$

Choose $N_2$ so that $\text{DV01}_1 + \text{DV01}_2 = 0$:
$$N_2 = N_1 \frac{A_{6M}}{A_{3M}} = 100{,}000{,}000 \times \frac{0.9850872503125}{0.9875623439063} = 99{,}749{,}373.43$$

**Result:** The hedged portfolio has (approximately) zero PV change for a common +1bp move in both tenor curves.

#### J.2 Apply a +5bp Tenor-Basis Widening Shock and Compute Residual P&L

**Scenario:** 6M forwards +5bp ($\Delta_{6M} = 0.0005$), 3M forwards unchanged ($\Delta_{3M} = 0$).

**Trade 1 PV change:**
$$\Delta\text{PV}_1 = N_1 \cdot 0.0005 \cdot A_{6M} = 100{,}000{,}000 \times 0.0005 \times 0.9850872503125 = +\$49{,}254.36$$

**Trade 2 PV change:** 0 (3M forwards unchanged; fixed leg unchanged)

**Residual P&L (hedged book):**
$$\Delta\text{PV}_{hedged} = \boxed{+\$49{,}254.36}$$

**Interpretation:**
Even though the hedge eliminated "common" rate exposure (DV01), it left basis exposure—the very phenomenon the multi-index framework identifies as a distinct risk factor.

In practice, the direct hedge for this residual is a 3M–6M basis swap (or instruments used in its curve construction).

---

## Practical Notes

### 5.1 Common Quoting Conventions and Ambiguity Traps

#### Which Leg Gets the Spread

- **Default in this chapter:** the spread is added to "leg 1" in $L_2$ vs $L_1 + e_{1,2}(T)$, consistent with the reference PV condition for par basis swaps
- **Ambiguity in practice:** dealers may quote the spread on the other leg, or define the sign as "payer of basis" differently

> **I'm not sure** what convention applies in your market.
>
> **Needed:** currency + index family + market quote sheet convention ("3M+spread vs 6M" or "6M+spread vs 3M"), and which direction is "pay basis".

#### Term vs Compounded-in-Arrears

- OIS legs are typically linked to compounded overnight rates (source notes OIS involves exchanging a fixed rate for the compounded overnight rate)
- Tenor basis swaps in this chapter are described for Libor-like term indices (simple interest over the period)

> **Modern market caveat:** Some currencies have moved away from Libor and use RFR-based conventions.
>
> **I'm not sure** how your "1M/3M/6M" are defined today (term RFR? compounded in arrears with lookback?).
>
> **Needed:** the exact index definitions and ISDA fallback/compounding conventions for the product.

#### Day-Count and Payment-Frequency Differences

- Year fractions $\tau$ can be governed by complicated conventions (the text flags this explicitly)
- Fixed-leg frequency can differ from floating-leg frequency by currency (market convention). In examples we matched frequencies for clarity (see Tuckman-style simplification).

### 5.2 Implementation Pitfalls (Multi-Curve Curve-Building and Risk)

#### Dependency Graph Matters

The multi-index curve group concept implies a natural construction order: discount curve $P_d$ plus multiple tenor projection curves $P^{(k)}$.

Building secondary tenor curves from basis swaps (representing them as spreads off a base curve, sequentially across tenors) is the standard approach.

#### Interpolation Can Create Forward "Wiggles"

Interpolating $P^{(k)}$ directly vs interpolating log-discount factors vs interpolating zero rates can produce materially different forward shapes, and thus basis shapes. Keep consistent across tenors.

#### Inconsistent Bump Methodology

If you bump a basis quote but do not rebuild the dependent curve nodes consistently, you can create spurious P&L attribution. Risk should be expressed as sensitivities to underlying curves in the multi-index setting, which implicitly assumes a consistent curve-rebuild mapping from instruments to curves.

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **Multi-curve necessity:** 3M and 6M instruments imply different discount factors—one curve cannot fit both, requiring separate tenor projection curves

2. **Discounting vs projection:** Use $P_d$ (OIS) for discounting all cashflows; use $P^{(k)}$ for forecasting tenor-$k$ floating coupons

3. **Tenor basis definition:** The spread required in a basis swap to equate two floating legs of different tenors

4. **Basis swap mechanics:** Exchange $L^{(2)}$ vs $L^{(1)} + e_{1,2}(T)$; the spread $e_{1,2}(T)$ is the par basis

5. **Par basis formula:** $e_{1,2}(T) = \frac{\text{PV}(\text{leg 2 floats}) - \text{PV}(\text{leg 1 floats})}{\text{Annuity on leg 1}}$

6. **Bootstrapping secondary curves:** Use basis swap quotes to infer forwards for less-liquid tenors from a calibrated base tenor curve

7. **Risk decomposition:** Separate sensitivities to discount curve, each projection curve, and quoted basis spreads

8. **Hedging limitation:** DV01-hedging a 6M swap with 3M instruments leaves residual basis exposure

9. **Historical context:** USD 1M–3M basis widened from <1bp (2007) to ~50bp (2009) during the crisis

10. **Convention warnings:** Leg receiving the spread, day counts, and compounding conventions vary by currency and product—always verify

### Cheat Sheet of Formulas

| Formula | Expression |
|---------|------------|
| Simple forward (single curve) | $L(t,T,T+\tau) = \frac{1}{\tau}\left(\frac{P(t,T)}{P(t,T+\tau)} - 1\right)$ |
| Tenor-$k$ forward (multi-curve) | $L^{(k)}(t;T_{i-1},T_i) = \frac{1}{\tau_i^{(k)}}\left(\frac{P^{(k)}(t,T_{i-1})}{P^{(k)}(t,T_i)} - 1\right)$ |
| Par fixed rate (swap) | $c^\star = \frac{\sum_i F_i^{(k)} \tau_i P_d(0,t_i)}{\sum_i \tau_i P_d(0,t_i)}$ |
| Par basis spread | $e_{1,2}(T) = \frac{\text{PV}_{leg2} - \text{PV}_{leg1}}{A_{leg1}}$ |
| Bootstrap step | $P^{(k)}(0,t_{i+1}) = \frac{P^{(k)}(0,t_i)}{1 + \tau_i F_i^{(k)}}$ |
| Annuity | $A = \sum_i \tau_i P_d(0,t_i)$ |

### Flashcards (20 Q/A Pairs)

1. **Q:** What is tenor basis?
   **A:** The market-observed difference between floating rates of different tenors, requiring separate projection curves

2. **Q:** Why can't a single curve fit both 3M and 6M instruments?
   **A:** They imply different discount factors for the same maturity due to credit/liquidity/funding differences

3. **Q:** What is the discounting curve used for?
   **A:** Computing present values of all cashflows (typically OIS for collateralized trades)

4. **Q:** What is a projection curve used for?
   **A:** Forecasting expected floating rate fixings for a specific tenor

5. **Q:** What is a basis swap?
   **A:** An exchange of two floating legs tied to different indices, plus or minus a spread

6. **Q:** In our convention, where is the basis spread quoted?
   **A:** Added to leg 1 in the exchange $L_2$ vs $L_1 + e_{1,2}(T)$

7. **Q:** What makes a basis swap "par"?
   **A:** The spread $e_{1,2}(T)$ is set so that inception PV equals zero

8. **Q:** Write the par basis spread formula.
   **A:** $e_{1,2}(T) = \frac{\text{PV}(\text{leg 2}) - \text{PV}(\text{leg 1})}{\text{Annuity of leg 1}}$

9. **Q:** How do you bootstrap a secondary tenor curve?
   **A:** Use basis swap quotes to solve for unknown forwards, then convert to pseudo-discount factors

10. **Q:** What risk remains after DV01-hedging a 6M swap with 3M swaps?
    **A:** Basis risk—exposure to relative moves between 3M and 6M curves

11. **Q:** Why did tenor basis widen in 2008-2009?
    **A:** Credit/liquidity stress caused divergence in funding costs across different tenors

12. **Q:** What is the formula for a tenor-$k$ forward from its index curve?
    **A:** $L^{(k)}(t;T_{i-1},T_i) = \frac{1}{\tau}\left(\frac{P^{(k)}(t,T_{i-1})}{P^{(k)}(t,T_i)} - 1\right)$

13. **Q:** In multi-curve, does the floating leg PV equal $1 - P_d(0,T)$?
    **A:** No, that identity fails when $P^{(k)} \neq P_d$

14. **Q:** What is the dependency order for building multi-tenor curves?
    **A:** Build $P_d$ first, then base tenor curve, then secondary tenors via basis

15. **Q:** What causes "forward wiggles" in curve construction?
    **A:** Choice of interpolation method (discount factors vs log-DFs vs zero rates)

16. **Q:** What is $A_{3M}$ in the worked examples?
    **A:** The annuity (sum of $\tau \cdot P_d$ terms) for the 3M payment schedule

17. **Q:** If 6M leg has higher PV than 3M leg, what sign does the basis spread have?
    **A:** Positive (added to 3M leg to equalize PVs)

18. **Q:** What is the main sensitivity of a basis swap?
    **A:** Projection curve and basis spread sensitivities dominate; discount curve sensitivity is small

19. **Q:** How do you verify a bootstrap step worked correctly?
    **A:** Reprice the calibrating basis swap—it should have PV = 0

20. **Q:** What should you verify about quoting conventions before trading?
    **A:** Which leg gets the spread, sign convention, day count, and compounding method

---

## Mini Problem Set

### Problem 1 (Basic)
Given $P^{(3M)}(0,0) = 1$, $P^{(3M)}(0,0.25) = 0.9940$, and $\tau = 0.25$, compute the 3M forward rate $F_1^{3M}(0;0,0.25)$.

*Solution sketch:* $F = \frac{1}{0.25}\left(\frac{1}{0.9940} - 1\right) = \frac{1}{0.25}(1.00604 - 1) = 2.416\%$

### Problem 2 (Basic)
If $\text{PV}_{6M} = 0.0300$ and $\text{PV}_{3M} = 0.0280$ (per notional 1), and $A_{3M} = 0.98$, what is the par 3M–6M basis spread?

*Solution sketch:* $s = \frac{0.0300 - 0.0280}{0.98} = 0.00204 = 20.4$ bp

### Problem 3 (Intermediate)
A 1Y 3M swap has par rate $c_{3M}^\star = 3.00\%$ and annuity $A_{3M} = 0.985$. A 1Y 6M swap has par rate $c_{6M}^\star = 3.15\%$ and annuity $A_{6M} = 0.980$. Estimate the 3M–6M basis spread.

*Hint:* Compute floating leg PVs from $c^\star \times A$, then apply the basis formula.

### Problem 4 (Intermediate)
You receive 6M float and pay 3M + 12bp on a \$50mm 1Y basis swap. If the 3M annuity is 0.99 and 6M annuity is 0.98, and par basis is currently 10bp, what is the mark-to-market PV?

*Hint:* You locked in paying 12bp when fair is 10bp.

### Problem 5 (Intermediate)
Bootstrap: Given $P_d(0,0.5) = 0.99$, $P_d(0,1.0) = 0.98$, $P^{(6M)}(0,0.5) = 0.9875$, and $\text{PV}_{3M+spr}(1Y) = 0.0250$, solve for $F_2^{6M}(0;0.5,1.0)$.

### Problem 6 (Advanced)
Derive the formula for par basis spread starting from the PV equations for both legs. Show all steps and verify units.

### Problem 7 (Advanced)
A trader is DV01-flat on a 6M swap hedged with 3M swaps (using annuity ratio). If 6M forwards increase by 3bp while 3M forwards increase by 1bp, what is the basis P&L direction?

### Problem 8 (Advanced)
Explain why interpolating pseudo-discount factors $P^{(k)}$ directly versus interpolating zero rates can produce different forward curves. Which approach preserves forward positivity?

### Problem 9 (Risk)
A portfolio has +\$10,000 6M projection PV01 and −\$9,500 3M projection PV01. What is the approximate P&L if 3M–6M basis widens by 5bp (6M forwards up 5bp, 3M unchanged)?

### Problem 10 (Conceptual)
Why is the discount curve PV01 on a par basis swap typically small compared to projection curve sensitivities?

### Problem 11 (Implementation)
Describe the curve construction dependency graph for a system that builds: OIS discount, 3M projection, and 6M projection curves. What instruments calibrate each?

### Problem 12 (Advanced)
If you bump a 2Y basis swap quote by +1bp but fail to rebuild the 6M curve consistently, what type of error could this introduce in P&L attribution?

---

## Source Map

### (A) Verified Facts — Specific Source Citations

| Fact | Source |
|------|--------|
| Multi-curve framework definition | Andersen & Piterbarg Vol 1, Ch 6 |
| Simple forward rate definition | Andersen & Piterbarg Vol 1, §6.48 |
| Basis swap PV equation | Andersen & Piterbarg Vol 1, §6.47–6.48 |
| OIS discounting for collateralized trades | Hull Ch 4, Andersen Vol 1 Ch 6 |
| USD 1M–3M basis historical widening (~50bp in 2009) | Andersen & Piterbarg Vol 1 |
| Tenor basis attributed to credit/liquidity effects | Andersen & Piterbarg Vol 1 |
| Sequential curve construction from basis swaps | Andersen & Piterbarg Vol 1, Ch 6 |
| "Assets and liabilities with different floating rates" | Hull Ch 7 |

### (B) Reasoned Inference — Derivation Logic

| Inference | Derivation |
|-----------|------------|
| Par basis spread formula | Algebraic rearrangement of PV equation setting both legs equal |
| Single-curve contradiction | Direct computation showing different $P(0,T)$ from different instruments |
| Bootstrap step formula | Rearrangement of forward rate definition |
| DV01 hedge ratio | Setting sum of DV01s to zero and solving for notional |
| Residual basis P&L | Applying basis shock to hedged position |

### (C) Speculation — Flagged Uncertainties

| Item | Uncertainty |
|------|-------------|
| Market quoting conventions for specific currency/index pairs | Varies by market; not specified in sources |
| Modern RFR-based tenor conventions | Post-Libor transition details not in reference texts |
| Exact day-count conventions for specific products | Market-specific; simplified in examples |

---

*Last Updated: January 2026*
