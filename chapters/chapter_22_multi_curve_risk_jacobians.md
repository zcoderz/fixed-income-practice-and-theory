# Chapter 22: Curve Risk Management in a Multi-Curve World — Par-Point Deltas, Jacobians, and Controlled Perturbations

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- Building a continuous discount curve from market data requires interpolation/regularization because only a finite benchmark set is quoted (Andersen Vol 1)
- Post-crisis, curve construction/risk management often requires a collection of inter-related curves rather than a single Libor curve (Andersen Vol 1 Ch 6)
- Benchmark instrument prices $V_i$ can be expressed as linear combinations of discount factors at cashflow dates: $V_i = \sum_{j=1}^{M} c_{i,j} P(t_j)$ (Andersen Vol 1)
- Bucketed (par-point) deltas: for small benchmark price changes $\Delta V_i$, $\Delta V_0 \approx \sum_{i=1}^{N} \frac{\partial V_0}{\partial V_i} \Delta V_i$, and the vector $(\partial V_0 / \partial V_i)$ is explicitly called "bucketed interest rate deltas" (Andersen Vol 1)
- Par-point approach: risk can be computed by applying a 1 bp shift to a single benchmark instrument, reconstructing the curve, and revaluing $V_0$. This method can suffer from poor locality, and interpolation choices (e.g., $C^2$ splines) can introduce "see-saw"/global perturbations (Andersen Vol 1)
- A forward-curve (controlled-perturbation) approach computes Gâteaux derivatives by perturbing the forward curve $f$ with functional shifts $\mu_k$: $\partial_k V_0 = \frac{d V_0(f + \varepsilon \mu_k)}{d\varepsilon}\big|_{\varepsilon=0}$ (Andersen Vol 1)
- For tenor basis: a floating-floating basis swap at par satisfies an equality of discounted expected floating legs (Andersen Vol 1 Ch 6, equation 6.49), and index curves can be represented as multiplicative spreads to a base index curve
- DV01 (Tuckman): a general definition is $\mathrm{DV01} = -\frac{\Delta P}{10{,}000 \, \Delta y}$, for a chosen yield/rate measure $y$ (Tuckman Ch 5-6)
- Key-rate shifts (Tuckman): a key-rate shift raises one par yield by 1 bp, declines linearly to zero at adjacent key rates, and key-rate shifts sum to a parallel shift (Tuckman Ch 7)

### (B) Reasoned Inference (Derived from A)

- If a portfolio PV is a function of curve parameters $x$ calibrated from market quotes $q$, then quote-risk is obtained by the chain rule: $\frac{dV_0}{dq} = \frac{\partial V_0}{\partial x} \frac{\partial x}{\partial q}$. This is the mathematical backbone of "PV sensitivity → quote sensitivity."
- In a multi-curve setup, $x$ naturally decomposes into blocks (discounting nodes, projection nodes, basis nodes), so quote-risk also decomposes into discounting risk, projection risk, and basis risk—consistent with the "orthogonal meaning" highlighted in the spread-based curve-group construction.

### (C) Speculation (Clearly Labeled; Minimal)

- I'm not sure about specific desk implementations (e.g., bump size 0.5 bp vs 1 bp, central vs forward differences, "sticky" interpolation vs full rebuild, handling of turn-of-year effects). These vary widely across institutions. If you need to match a particular risk report, you must specify the desk's curve build and bumping conventions. (Not fully specified in the provided sources.)

---

## Conventions & Notation

### Defaults Used in This Chapter

| Convention | Default | Notes |
|------------|---------|-------|
| Time reference | $t = 0$ is "today" | Maturities $T \in (0, \mathcal{T}]$ in years |
| Discount factors | $P(0,T)$ or $P(T)$ | Price of ZCB paying 1 at $T$; $P: [0,\mathcal{T}] \to (0,1]$ continuous, decreasing |
| Zero yields (when used) | Continuously compounded | $P(T) = \exp(-y(T) \cdot T)$ |
| Bump size | 1 bp = $10^{-4}$ in rate units | Standard for DV01/par-point calculations |
| Multi-curve context | Post-crisis (OIS discounting + projection curves) | Single Libor curve is pre-crisis legacy |

### Notation Glossary

| Symbol | Meaning | Units |
|--------|---------|-------|
| $P(0,T)$ or $P(T)$ | Discount factor to $T$ | Dimensionless |
| $y(T)$ | Continuously-compounded zero yield | Per annum (decimal) |
| $f(T)$ | Instantaneous forward rate | Per annum (decimal) |
| $L(t, T_n, T_{n+1})$ | Forward Libor rate over $[T_n, T_{n+1}]$ | Per annum (decimal) |
| $\tau_n$ | Accrual factor $T_{n+1} - T_n$ | Years |
| $V_0$ | Portfolio PV (target trade/portfolio) | Currency |
| $V_i$ | Price of benchmark instrument $i$ | Currency / price units |
| $q_i$ | Market quote for benchmark $i$ (swap rate, basis spread, etc.) | Rate or spread units |
| $x$ | Vector of curve nodes/parameters | DFs, rates, or spline coefficients |
| $J$ | Jacobian $\partial x / \partial q$ | Depends on parameterization |
| $\Delta$ | Finite difference / bump operator | — |
| $\mathrm{DV01}$ | Dollar value of 1 bp change | Currency per bp |
| $P_k(\cdot)$ | Index/projection curve for tenor $k$ | Dimensionless |
| $\eta_{1,2}(s)$ | Basis spread intensity (spread-curve representation) | Per annum |
| $\mu_k(t)$ | Forward-rate basis function for controlled perturbations | — |
| $\partial_k V_0$ | Gâteaux derivative of PV w.r.t. shift $\mu_k$ | Currency per unit shock |

### Multi-Curve Objects (Single-Currency Tenor-Basis Context)

- $P(\cdot)$: discounting curve (built from "funding" instruments such as OIS in USD examples)
- $P_k(\cdot)$: index/projection curve for tenor $k$ (built as a spread to a base index curve)
- Basis between index curves is represented multiplicatively:
$$P_2(t) = P_1(t) \exp\!\left(-\int_0^t \eta_{1,2}(s)\, ds\right)$$

### Forward Libor Rates (Discrete-Tenor Setting)

For tenor dates $0 = T_0 < T_1 < \cdots < T_N$ with accrual $\tau_n = T_{n+1} - T_n$:
$$L(t, T_n, T_{n+1}) = \frac{1}{\tau_n} \left( \frac{P(t, T_n)}{P(t, T_{n+1})} - 1 \right)$$

---

## Core Concepts

### 1) Single-Curve vs Multi-Curve: "What Changed" for Risk

**Formal Definition:**

- **Single-curve world:** One curve (often "Libor discount curve") is used both for discounting cashflows and for generating forward rates.
- **Multi-curve world:** Valuation uses separate curves—at minimum a discounting curve and one or more projection/index curves—linked by basis instruments; curve construction requires a "collection of inter-related curves."

**Intuition:**

Post-crisis, the market recognized that "Libor" embedded bank credit/liquidity and was not a clean proxy for the risk-free discount rate; additionally, tenor basis appeared across swap payment frequencies. The pre-2007 assumption that 1M, 3M, and 6M Libor curves were interchangeable broke down visibly during the credit crisis.

**Trading / Risk / Portfolio Practice:**

- Desks must choose: which instruments define discounting (e.g., OIS) vs which define projection (e.g., 3M Libor swaps/FRAs), and how basis swaps link tenors
- The sources describe constructing "a pair of curves" from OIS and Libor instruments, and then additionally accounting for tenor basis
- Risk reports now require decomposition into discounting risk, projection risk, and basis risk—not just a single "rate delta"

---

### 2) Curve Representation: Nodes + Interpolation Rule + "Locality"

**Formal Definition:**

A yield/discount curve is the term structure mapping maturities to discount factors or rates. Benchmark prices are linear combinations of $P(t_j)$ at cashflow dates:
$$V_i = \sum_{j=1}^{M} c_{i,j} P(t_j)$$

Since only finitely many instruments are quoted, the curve construction problem is **underdetermined** without an interpolation rule.

**Intuition:**

Interpolation is not just "cosmetics." The curve shape between quotes changes forward rates and discount factors, and therefore changes risk—especially when you "bump" one quote and rebuild.

**Locality under perturbation** is the property that a bump to a local instrument quote produces a local change in the curve, not a global oscillation. The sources stress that locality can be lost and must be controlled, e.g., through different spline/bootstrapping choices and tension.

**Trading / Risk / Portfolio Practice:**

- Risk managers want deltas that (i) are stable (don't jump due to numeric artifacts), (ii) correspond to meaningful hedges (bucket instruments), and (iii) explain daily P&L
- Practical curve builders compare methods like piecewise-linear yields bootstrapping, $C^1$ Hermite splines, $C^2$ natural cubic splines, and $C^2$ splines with tension
- Perturbation plots show very different forward-curve moves for the same 1 bp swap quote bump across construction methods

---

### 3) PV Sensitivity Objects: DV01, Key-Rate/Bucket Risk, and Why "Parallel DV01" Is Not Enough

**Formal Definition:**

DV01 is a price sensitivity to a 1 bp change in a chosen yield/rate measure:
$$\boxed{\mathrm{DV01} = -\frac{\Delta P}{10{,}000 \, \Delta y}}$$

**Key-rate shifts (Tuckman):** Shift one key par yield by 1 bp, with linear tapering to adjacent key rates; key-rate shifts add up to a parallel shift.

**Bucket exposures (Tuckman):** Sensitivities to changes in forward rates in specific time buckets can be computed by raising one forward rate in a bucket and recomputing the implied spot curve and discount factors.

**Intuition:**

A single number (parallel DV01) compresses all curve risk into one dimension. Key-rate and bucket exposures restore shape risk. In curve-building terms, key-rate and bucket shifts are controlled perturbations of the curve.

**Trading / Risk / Portfolio Practice:**

- Traders hedge with instruments that correspond to curve points/buckets (e.g., swaps/futures along maturities) rather than only "parallel DV01," because most rate moves are not parallel
- Tuckman explicitly illustrates bucket exposures on a par swap by shifting one forward bucket (e.g., a 6M forward rate 2.5 years forward) by 1 bp and recomputing PV

---

### 4) Par-Point Delta / Quote Delta: "Bump-and-Rebuild" as a Definition

**Formal Definition:**

Bucketed interest-rate deltas are defined with respect to benchmark instrument prices $V_i$ through the linearized approximation:
$$\boxed{\Delta V_0 \approx \sum_{i} \frac{\partial V_0}{\partial V_i} \Delta V_i}$$

The **par-point approach** computes these sensitivities by shifting one benchmark instrument by (typically) 1 bp, reconstructing the curve, and revaluing $V_0$.

**Intuition:**

A "quote delta" answers: "If the market quote for instrument $i$ moved by 1 bp and the curve were rebuilt in the standard way, what's the PV change?"

**Trading / Risk / Portfolio Practice:**

- This is how many desks generate risk reports aligned with hedging instruments: each delta corresponds to a market-quoted instrument
- But the method's results depend on the curve algorithm: poor locality can produce large/unintuitive risk distribution ("see-saw" forward-curve impact from a small swap quote bump)

---

### 5) Jacobians: Mapping Quote Moves to Curve-Node Moves (and Vice Versa)

**Formal Definition:**

The Jacobian method provides a way to translate changes in benchmark prices to changes in a curve representation (e.g., forward curve):
- Compute $\partial f(t) / \partial V_i$
- Use it to map $\Delta V_i \mapsto \Delta f(t)$

In the spread-based multi-curve construction, "sensitivities to instruments used in the curve group have clear, and orthogonal, meaning" (overall level vs discounting vs basis).

**Intuition:**

The curve builder is a function from quotes to curve nodes. Jacobians are the local linearization of that function.

Once you have Jacobians, you can:
1. Compute PV sensitivities to curve nodes analytically/semianalytically, then
2. Map those to quote sensitivities by multiplication

**Trading / Risk / Portfolio Practice:**

- Jacobians can reduce computational cost vs "bump every quote and rebuild"
- Jacobians can help produce more stable deltas (when combined with controlled perturbations)
- The sources emphasize Jacobian techniques as "advanced methods" for curve risk management

---

### 6) Controlled Perturbations: Forward-Rate Basis Functions and Functional Derivatives

**Formal Definition:**

Instead of perturbing benchmark instrument prices, perturb the forward curve $f(t)$ directly by functional shifts $\mu_k(t)$, and compute the Gâteaux derivative:
$$\boxed{\partial_k V_0 = \frac{d V_0(f + \varepsilon \mu_k)}{d\varepsilon}\bigg|_{\varepsilon=0}}$$

**Intuition:**

You want shocks that represent economically meaningful curve moves (e.g., "raise 2y forward segment") without being polluted by curve-fit artifacts created by "bump-and-rebuild" under a wiggly spline.

**Trading / Risk / Portfolio Practice:**

- Controlled shocks align with bucket/key-rate hedging
- Can be translated back into quote space via Jacobians if the desk risk report needs "par-point deltas"
- Examples of basis functions include piecewise triangular and piecewise flat shapes in forward space

---

## Math and Derivations

### 2.1 PV as a Function of Curve Objects

A basic PV statement: "PV is the present value of discounted future cash flows." This is used throughout fixed income; e.g., Tuckman values swap legs by discounting their cash flows.

In the curve-construction framework, benchmark instrument prices are linear combinations of discount factors at relevant cashflow dates:
$$V_i = \sum_{j=1}^{M} c_{i,j} \, P(t_j), \quad i = 1, \ldots, N$$

**Interpretation:** If we take the set of curve "nodes" as $\{P(t_j)\}_{j=1}^{M}$, then $V_i$ is linear in those nodes. For a portfolio $V_0$ (not necessarily linear in the nodes), we can still view $V_0$ as a function of the curve:
$$V_0 = V_0(P(\cdot)) \quad \text{or} \quad V_0 = V_0(x)$$
where $x$ is a finite parameterization (nodes, spline coefficients, etc.).

---

### 2.2 Bucketed Deltas to Benchmark Instrument Values

The first-order approximation:
$$\boxed{\Delta V_0 \approx \sum_{i=1}^{N} \frac{\partial V_0}{\partial V_i} \, \Delta V_i} \tag{2.1}$$

The vector $\left(\frac{\partial V_0}{\partial V_i}\right)_{i=1}^{N}$ is the delta vector in benchmark-instrument space.

**Units check:**
- $V_0$ is in currency (e.g., USD)
- $V_i$ is in currency per unit notional (or price units)
- Therefore $\partial V_0 / \partial V_i$ is dimensionless if both are consistent price units, or "currency per price unit" in general

---

### 2.3 Par-Point / Quote Delta via Bump-and-Rebuild

**Definition via procedure (par-point approach):**

1. Shift the market quote of benchmark security $i$ by 1 bp
2. Reconstruct the curve
3. Reprice the portfolio
4. Report the PV change as the sensitivity

Let $q_i$ be the market quote for instrument $i$, and let $B(\cdot)$ denote the curve builder that maps quotes $q$ to curve parameters $x$:
$$x = B(q)$$

Then the PV is:
$$V_0(q) := V_0(x(q)) = V_0(B(q))$$

A quote delta (par-point delta) for quote $q_i$ is approximated numerically by:
$$\boxed{\frac{\partial V_0}{\partial q_i} \approx \frac{V_0(q_1, \ldots, q_i + \delta, \ldots, q_N) - V_0(q_1, \ldots, q_i, \ldots, q_N)}{\delta}} \tag{2.2}$$
where $\delta = 1 \text{ bp} = 10^{-4}$ in rate units.

**Why this is not "just DV01":** DV01 is sensitivity to a chosen yield/rate measure $y$ (often a parallel yield shift); quote delta is sensitivity to a market instrument quote after rebuilding the entire curve, which can produce non-parallel changes in forwards and discount factors.

**Locality and artifacts:** Certain interpolation rules can cause a local bump to create a global oscillation ("see-saw impact"); $C^2$ cubic splines are explicitly cited as problematic for locality under perturbation.

---

### 2.4 Jacobian Mapping and the Chain Rule

**General chain rule for quote risk:**

Let:
- $q \in \mathbb{R}^N$ be the vector of market quotes (or benchmark prices)
- $x \in \mathbb{R}^M$ be the vector of curve parameters/nodes (DFs, zero rates, forward rates, spline coefficients)
- $V_0 = V_0(x)$ be PV as a function of the curve parameterization

Then:
$$\boxed{\frac{dV_0}{dq} = \frac{\partial V_0}{\partial x} \, \frac{\partial x}{\partial q}} \tag{2.3}$$

- $\frac{\partial V_0}{\partial x}$ is the **node sensitivity** (in "curve space")
- $J := \frac{\partial x}{\partial q}$ is the **Jacobian** of the curve builder (in "build space")

This is the mathematical statement of the desk process: "compute PV sensitivity in curve space, then map to quote space."

**Jacobian method in the sources:**
- Jacobian techniques allow risk computations to be decoupled from the curve construction algorithm, potentially improving stability and interpretability
- Two-sided finite differences and averaging "up/down" delta vectors are suggested to improve numerical stability

**Units check (crucial in practice):**

Suppose:
- $V_0$ in USD
- A swap quote $q_i$ in "rate" units (decimal), where 1 bp $= 10^{-4}$
- A curve node $x_j$ is either a DF (dimensionless) or a rate

Then:
- $\frac{\partial V_0}{\partial q_i}$ is USD per unit rate; desk reports often scale to USD per bp:
$$\text{USD/bp} = 10^{-4} \, \frac{\partial V_0}{\partial q_i}$$
- If $x_j = P(t_j)$ is a DF, $\frac{\partial V_0}{\partial x_j}$ is USD (per unit DF)
- If $x_j = y(t_j)$ is a continuously-compounded zero yield, $\frac{\partial V_0}{\partial x_j}$ is USD per unit rate

---

### 2.5 Controlled Perturbations: Forward-Rate Basis Functions

We write "loosely" $V_0 = V_0(f)$ and compute functional derivatives:
$$\boxed{\partial_k V_0 = \frac{d V_0(f(t) + \varepsilon \mu_k(t))}{d\varepsilon}\bigg|_{\varepsilon=0}, \quad k = 1, \ldots, K} \tag{2.4}$$

Examples of basis functions include piecewise triangular and piecewise flat shapes.

**Risk interpretation:**
- Choose $\mu_k$ to correspond to maturities/tenor buckets relevant for hedging
- Sensitivities $\partial_k V_0$ represent "bucket PV01" in forward space—conceptually similar to Tuckman bucket exposure (shift one forward bucket, recompute PV)

---

### 2.6 Multi-Curve Dependency: Discounting vs Projection vs Basis

**Separate discount and projection curves:**

The sources describe constructing:
1. A curve for discounting, and
2. A curve for projecting 3-month forward Libor rates,

from market quotes on deposits, FRAs, swaps with 3m frequency, and OIS. They also emphasize the need to account for tenor basis between swaps of different floating-leg frequencies (e.g., 1m vs 3m vs 6m).

**Basis instruments link curves (tenor basis):**

For a floating-floating basis swap traded at par:
$$\boxed{\sum_{i=0}^{n_2(T)-1} L_2(0, t_i^2, t_{i+1}^2) \tau_i^2 \, P(t_{i+1}^2) = \sum_{i=0}^{n_1(T)-1} \left(L_1(0, t_i^1, t_{i+1}^1) + e_{1,2}(T)\right) \tau_i^1 \, P(t_{i+1}^1)} \tag{2.5}$$
where $e_{1,2}(T)$ is the quoted basis spread (on the $L_1$ leg).

**Spread-curve representation:**
$$\boxed{P_2(t) = P_1(t) \exp\!\left(-\int_0^t \eta_{1,2}(s) \, ds\right)} \tag{2.6}$$

**Risk meaning in a curve-group (multi-index) construction:**

With spread-based curve group construction, sensitivities to curve-group instruments have "clear, and orthogonal, meaning":
- **Base index curve instruments** ⇒ overall rate level sensitivities (moving all index curves together in forward space)
- **Funding instruments** ⇒ discounting sensitivities
- **Basis swap spreads** ⇒ basis risk (tenors not moving in lock step)

---

## Worked Examples

*Note: The source material provides conceptual/procedural examples but limited fully worked numeric examples. The examples below capture the procedural workflow described in the sources.*

### Example 1: Par-Point Delta Calculation (Conceptual Procedure)

**Setup:** You hold a 5y receiver swap and want to compute par-point deltas.

**Procedure:**
1. Mark base case: Record $V_0^{\text{base}}$ using current curve build
2. Bump 2y swap quote by 1 bp: $q_{2y} \to q_{2y} + 0.0001$
3. Rebuild all affected curves (discounting and projection)
4. Reprice: Compute $V_0^{\text{bumped}}$
5. Par-point delta to 2y: $\frac{\partial V_0}{\partial q_{2y}} \approx \frac{V_0^{\text{bumped}} - V_0^{\text{base}}}{0.0001}$
6. Repeat for each benchmark instrument (3y, 5y, 7y, 10y swaps, etc.)

**Output:** Vector of par-point deltas in USD per bp (after scaling by $10^{-4}$).

---

### Example 2: Jacobian Computation via Finite Differences

**Setup:** Curve parameterized by zero rates $x = (y_1, y_2, \ldots, y_M)$ at nodes $t_1, \ldots, t_M$.

**Procedure to compute column $i$ of Jacobian $J = \partial x / \partial q$:**
1. Bump quote $q_i$ up by $\delta$: $q_i^+ = q_i + \delta$
2. Rebuild curve: Obtain $x^+$
3. Bump quote $q_i$ down by $\delta$: $q_i^- = q_i - \delta$
4. Rebuild curve: Obtain $x^-$
5. Central difference: $J_{\cdot, i} \approx \frac{x^+ - x^-}{2\delta}$

**Chain rule application:**
$$\frac{dV_0}{dq_i} = \sum_j \frac{\partial V_0}{\partial x_j} \cdot J_{j,i}$$

---

### Example 3: Tenor Basis — Identifying the Spread in a Basis Swap

**Setup:** 3M vs 6M basis swap, 5y maturity, traded at par.

**Par condition (from Eq. 2.5):**
- 6M leg pays $L_{6M}$ every 6 months
- 3M leg pays $L_{3M} + e_{3M,6M}$ every 3 months
- At par: PV(6M leg) = PV(3M leg)

**The quoted spread** $e_{3M,6M}$ is the basis spread added to the 3M leg. If 6M Libor trades "rich" relative to 3M Libor (i.e., 6M forwards are lower than what 3M forwards would imply), then $e_{3M,6M} > 0$.

---

### Example 4: Locality Under Perturbation — Comparing Interpolation Methods

**Setup:** Bump 2y swap rate by 1 bp under two curve construction methods.

**Method A: Piecewise-linear zero yields (bootstrap)**
- Forward curve shows a localized spike near 2y
- Delta to 2y swap is concentrated in 2y bucket

**Method B: $C^2$ natural cubic spline**
- Forward curve shows oscillations extending to 5y and beyond ("see-saw")
- Delta to 2y swap shows spurious exposure in 3y, 5y buckets

**Implication:** Method B produces unstable risk that doesn't correspond to economic hedges.

---

## Practical Notes

### Curve Representation Choices

**Node choice: DFs vs zero rates vs forward rates**

| Parameterization | Pros | Cons |
|------------------|------|------|
| DF nodes $x_j = P(t_j)$ | Linear benchmark structure $V_i = \sum c_{ij} P(t_j)$ | Interpolation between nodes can be awkward |
| Zero-yield nodes $x_j = y(t_j)$ | Often more stable/monotone under interpolation | Node sensitivities less directly "cashflow weights" |
| Forward-rate representation $f(t)$ | Most economically interpretable for shape risk | Requires care in numerical implementation |

**Interpolation rule implications:**
- Different methods (bootstrapping, Hermite $C^1$, natural cubic $C^2$) respond very differently to the same quote bump
- Adding "tension" to a $C^2$ spline can damp perturbation noise and improve par-point risk reports

### Common Pitfalls

1. **Ambiguous bump definition ("bump what?")**
   - Swap quote bump = bump fixed rate?
   - Basis quote bump = bump spread $e_{1,2}(T)$?
   - OIS bump = bump OIS fixed rate?
   - Without a clear mapping, quote deltas are not comparable

2. **Interpolation-driven artifacts**
   - Poor locality (e.g., $C^2$ spline see-saw) can produce unintuitive deltas

3. **Perturbation noise from rebuild**
   - Tension can damp perturbation noise and improve risk reports

4. **Finite-difference instability**
   - Use two-sided bumps and/or average up/down deltas to reduce numerical noise

5. **Mixing DV01 notions**
   - DV01 depends on the chosen yield/rate measure and bumping rule
   - Do not compare DV01s computed under different measures without normalization

6. **Multi-curve aggregation mistakes**
   - If you build all index curves independently, you lose the "orthogonal" decomposition (level vs discounting vs basis) that the spread-based curve-group construction provides

### P&L Predict Consistency

Using different curves for pricing and risk can lead to poor P&L predict.

**Desk policy recommendations:**
- Use a consistent curve build for valuation and risk wherever possible
- If using "frozen" Jacobians or alternative risk curves, validate P&L explain vs realized quote moves

### Multi-Curve Dependency Graph (Conceptual)

```
Market quotes q
  ├─ Funding/OIS quotes ──► discount curve P(·) ──► discounting of all cashflows
  ├─ Tenor-1 (e.g., 3M) quotes ─► base index curve P^1(·) ─► forwards L^1
  ├─ Tenor-k basis spreads e^{1,k}(T) ─► basis spread curve η^{1,k}(·) ─► P^k(·) ─► forwards L^k
  └─ (optional) cross-currency basis quotes ─► xccy basis curve(s)
```

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **Multi-curve reality:** Post-crisis valuation requires separate discounting (OIS) and projection curves, linked by basis instruments—not a single Libor curve.

2. **Curve construction is underdetermined:** Interpolation/regularization choices affect both pricing and risk; locality under perturbation is a key desideratum.

3. **Par-point deltas:** Bump one market quote by 1 bp, rebuild curves, reprice—this is the standard "quote delta" but inherits curve-fit artifacts.

4. **DV01 vs quote delta:** DV01 is sensitivity to a chosen rate measure; quote deltas are sensitivities to market instruments after full curve rebuild—these are related but not identical.

5. **Jacobians linearize the curve builder:** $J = \partial x / \partial q$ maps quote changes to node changes, enabling efficient risk computation via chain rule.

6. **Controlled perturbations:** Perturb the forward curve directly with economically meaningful shapes $\mu_k$ to avoid spline artifacts.

7. **Orthogonal decomposition:** With spread-based curve-group construction, risk decomposes cleanly into (i) overall level, (ii) discounting, and (iii) basis components.

8. **Locality matters:** Poor interpolation choices (e.g., $C^2$ splines without tension) can create "see-saw" forward moves from local quote bumps.

9. **Finite-difference stability:** Use central differences and average up/down vectors to improve numerical stability of Jacobians.

10. **P&L consistency:** Use the same curves for valuation and risk; mismatches cause poor P&L predict.

### Cheat Sheet of Key Formulas

| Formula | Description |
|---------|-------------|
| $\mathrm{DV01} = -\frac{\Delta P}{10{,}000 \, \Delta y}$ | DV01 definition (Tuckman) |
| $\Delta V_0 \approx \sum_i \frac{\partial V_0}{\partial V_i} \Delta V_i$ | Bucketed delta approximation |
| $\frac{dV_0}{dq} = \frac{\partial V_0}{\partial x} \cdot \frac{\partial x}{\partial q}$ | Chain rule for quote risk |
| $\partial_k V_0 = \frac{d}{d\varepsilon} V_0(f + \varepsilon \mu_k)\big|_{\varepsilon=0}$ | Controlled perturbation (Gâteaux derivative) |
| $P_2(t) = P_1(t) \exp\left(-\int_0^t \eta_{1,2}(s) ds\right)$ | Spread-curve representation for tenor basis |
| $L(t, T_n, T_{n+1}) = \frac{1}{\tau_n}\left(\frac{P(t,T_n)}{P(t,T_{n+1})} - 1\right)$ | Forward Libor rate |

### Flashcards (Q/A Pairs)

1. **Q:** What is the key difference between single-curve and multi-curve frameworks?
   **A:** Single-curve uses one curve for both discounting and projection; multi-curve uses separate OIS discount curve and tenor-specific projection curves linked by basis instruments.

2. **Q:** What does "locality under perturbation" mean?
   **A:** A bump to one quote produces only a local change in the curve, not global oscillations.

3. **Q:** Define DV01 using Tuckman's formula.
   **A:** $\mathrm{DV01} = -\Delta P / (10{,}000 \cdot \Delta y)$

4. **Q:** What is a par-point delta?
   **A:** The PV change when you bump one market quote by 1 bp and rebuild the curve.

5. **Q:** What is the Jacobian $J$ in curve risk?
   **A:** $J = \partial x / \partial q$, the matrix mapping quote changes to curve-node changes.

6. **Q:** Write the chain rule for quote risk.
   **A:** $dV_0/dq = (\partial V_0 / \partial x) \cdot (\partial x / \partial q)$

7. **Q:** What is a Gâteaux derivative in the context of curve risk?
   **A:** $\partial_k V_0 = \frac{d}{d\varepsilon} V_0(f + \varepsilon \mu_k)|_{\varepsilon=0}$, the sensitivity to a controlled forward-curve perturbation.

8. **Q:** Why can $C^2$ cubic splines be problematic for par-point risk?
   **A:** They can produce "see-saw" oscillations in the forward curve from a local quote bump (poor locality).

9. **Q:** What is the spread-curve representation for tenor basis?
   **A:** $P_2(t) = P_1(t) \exp(-\int_0^t \eta_{1,2}(s) ds)$

10. **Q:** In a basis swap par condition, what is $e_{1,2}(T)$?
    **A:** The quoted basis spread added to the shorter-tenor leg.

11. **Q:** What three risk components does the spread-based curve-group construction decompose?
    **A:** (i) Overall rate level, (ii) discounting risk, (iii) basis risk.

12. **Q:** Why use two-sided finite differences for Jacobians?
    **A:** To improve numerical stability and reduce noise.

13. **Q:** What happens if you use different curves for pricing vs risk?
    **A:** Poor P&L predict (P&L explain doesn't match realized quote moves).

14. **Q:** What is the formula for forward Libor rate in terms of discount factors?
    **A:** $L(t, T_n, T_{n+1}) = \frac{1}{\tau_n}\left(\frac{P(t,T_n)}{P(t,T_{n+1})} - 1\right)$

15. **Q:** What does "bump what?" ambiguity refer to?
    **A:** The need to specify exactly which quote (swap rate, basis spread, OIS rate) is being bumped in a par-point calculation.

---

## Mini Problem Set

### Questions (Increasing Difficulty)

**1. DV01 definition check**

Using Tuckman's definition $\mathrm{DV01} = -\Delta P / (10{,}000 \, \Delta y)$, explain why the negative sign is included and what sign DV01 has when rates fall.

**2. Key-rate shift construction**

Define a key-rate shift at the 5-year key rate (with neighbors 2y and 10y). Why do key-rate shifts sum to a parallel shift?

**3. Bucketed delta interpretation**

In the approximation $\Delta V_0 \approx \sum_i (\partial V_0 / \partial V_i) \Delta V_i$, what are the units of $\partial V_0 / \partial V_i$?

**4. Par-point method locality**

Explain why a 1 bp bump to a 2y swap rate could produce a "see-saw" effect in the forward curve when using a $C^2$ cubic spline curve construction.

**5. Forward-rate controlled shock**

Define $\partial_k V_0$ as in the forward-rate approach and interpret it as a bucket sensitivity.

**6. Tenor basis par condition**

Write down the par condition for a floating-floating basis swap and identify which term is the quoted basis spread.

**7. Spread-curve representation**

Given $P_2(t) = P_1(t) \exp\left(-\int_0^t \eta_{1,2}(s) ds\right)$, explain in words what $\eta_{1,2}$ represents.

**8. Multi-curve risk decomposition**

Explain (conceptually) why base index curve bumps correspond to "overall level" risk, funding instrument bumps correspond to "discounting" risk, and basis spread bumps correspond to "basis" risk.

**9. Jacobian mapping (conceptual)**

Suppose you have node sensitivities $\partial V_0 / \partial x$ and a Jacobian $J = \partial x / \partial q$. Explain how you compute quote deltas and how you would report them in USD/bp. (Use unit checks.)

**10. P&L explain thought experiment**

Give two reasons why using different curves for valuation and risk could lead to poor P&L predict, and propose one diagnostic test you would run on historical data.

### Solution Sketches

**1.** The negative sign makes DV01 positive for typical fixed-income positions (bonds, receiver swaps) where price rises when rates fall. When rates fall, $\Delta y < 0$ and $\Delta P > 0$ for a long bond, so $-\Delta P / \Delta y > 0$.

**2.** A 5y key-rate shift: +1 bp at 5y, declining linearly to 0 at 2y and 10y, zero outside. They sum to parallel because each maturity receives exactly 1 bp total when you add all triangular key-rate shifts.

**3.** If $V_0$ and $V_i$ are both in currency units, then $\partial V_0 / \partial V_i$ is dimensionless. If $V_i$ is a rate/price per unit notional, units may differ; the key is consistency.

**4.** $C^2$ splines enforce continuous second derivatives at knots. A local bump can propagate curvature constraints across knots, creating oscillations in the reconstructed forward curve far from the bumped point.

**5.** $\partial_k V_0 = \frac{d}{d\varepsilon} V_0(f + \varepsilon \mu_k)|_{\varepsilon=0}$ is the directional derivative of PV in the direction of the basis function $\mu_k$. If $\mu_k$ is localized to a time bucket, this is the bucket sensitivity.

**6.** Par condition: $\sum_{i} L_2 \tau_i^2 P(t_{i+1}^2) = \sum_i (L_1 + e_{1,2}) \tau_i^1 P(t_{i+1}^1)$. The term $e_{1,2}$ is the quoted basis spread.

**7.** $\eta_{1,2}(s)$ is the instantaneous spread intensity that transforms the base index curve $P_1$ into the secondary index curve $P_2$. It captures the tenor basis in continuous time.

**8.** Base index bumps move all projection curves together (level). Funding bumps change only discounting (present value weights). Basis bumps change one tenor relative to another (relative value between tenors).

**9.** Quote delta: $dV_0/dq = (\partial V_0/\partial x) \cdot J$. If $\partial V_0/\partial x$ is in USD and $J$ is dimensionless (rate-to-rate), then $dV_0/dq$ is USD per unit rate. Scale by $10^{-4}$ for USD/bp.

**10.** (i) Curve rebuilds may follow different paths in quote space vs node space, creating basis risk. (ii) Jacobians computed at one curve state may not apply at another. Diagnostic: regress realized P&L on predicted P&L from risk; slope should be ~1, intercept ~0.

---

## Source Map

### (A) Verified Facts

| Fact | Source |
|------|--------|
| Benchmark prices as linear combinations of DFs | Andersen Vol 1 Ch 4-6 |
| Bucketed interest rate deltas definition | Andersen Vol 1 |
| Par-point approach and locality issues | Andersen Vol 1 |
| Gâteaux derivatives for curve risk | Andersen Vol 1 |
| Tenor basis via floating-floating basis swap | Andersen Vol 1 Ch 6 (Eq. 6.49) |
| Spread-curve representation $P_2 = P_1 \exp(\cdot)$ | Andersen Vol 1 Ch 6 |
| DV01 definition | Tuckman Ch 5-6 |
| Key-rate shifts sum to parallel | Tuckman Ch 7 |
| Multi-curve construction (OIS + projection) | Andersen Vol 1 Ch 6 |

### (B) Reasoned Inference

| Inference | Derivation |
|-----------|------------|
| Chain rule $dV_0/dq = (\partial V_0/\partial x)(\partial x/\partial q)$ | Standard calculus applied to curve builder as a function |
| Risk decomposition into level/discounting/basis | Follows from spread-based parameterization orthogonality |

### (C) Speculation

| Item | Uncertainty |
|------|-------------|
| Specific desk bump conventions (size, central vs forward, sticky vs rebuild) | Varies by institution; not fully specified in sources |
| Exact tension parameters for spline damping | Implementation-dependent |
| Turn-of-year and other calendar effects in curve building | Market-specific; not covered in detail |

---

*Chapter 22 — v1.0*
