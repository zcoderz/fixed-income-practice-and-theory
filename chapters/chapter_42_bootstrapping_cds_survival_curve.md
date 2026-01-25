# Chapter 42: Bootstrapping a CDS Survival Curve from Market Quotes

---

## Conventions & Notation (used in this chapter only)

Time origin is valuation date $t=0$. Maturity is $T$. Premium payment dates are $t_n$ with $t_0=0$ and $t_N=T$.

| Symbol | Definition |
|--------|------------|
| $Z(0,t)$ | Discount factor to time $t$ (treated as an input curve). In the primary source, $Z(t,T)$ is a "Libor discount curve." |
| $Q(0,t)$ | Survival probability to time $t$ |
| $h(t)$ | Continuously compounded forward default rate, linked by $Q(t) = \exp\left(-\int_0^t h(s)\,ds\right)$ |
| $R$ | Expected recovery rate as a percentage of par |
| $S$ | CDS spread (decimal per year; e.g., 100 bp $= 0.01$) |
| $\Delta(t_{n-1}, t_n)$ | Accrual year fraction between premium dates (input; market is "typically Actual/360" per the primary source) |

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- A CDS is valued by splitting into a premium leg and protection leg.
- Under independence, the PV of a risky \$1 paid at $t_n$ conditional on survival is $Z(0,t_n) Q(0,t_n)$.
- Premium leg PV at initiation can be written as $S_0 \cdot \text{RPV01}(0,T)$.
- RPV01 includes coupon accrued at default via an integral term; a common approximation yields:
$$\text{RPV01}(0,T) = \frac{1}{2} \sum_{n=1}^{N} \Delta_n Z(0,t_n) \left(Q(0,t_{n-1}) + Q(0,t_n)\right)$$
- Protection leg PV is:
$$(1-R) \int_0^T Z(0,s) \left(-dQ(0,s)\right)$$
  and can be discretized into sums over small intervals.
- A more accurate "trapezoid-style" approximation averages upper/lower bounds and gives protection PV proportional to $\frac{1}{2}(Z_{k-1} + Z_k)(Q_{k-1} - Q_k)$.
- The breakeven spread $S_0$ of a new CDS satisfies $V(0)=0$ and is computed from protection PV divided by RPV01 (equation (6.5) in the source).
- The bootstrapping algorithm solves sequentially for $Q(T_m)$ such that the $T_m$-maturity CDS has zero MTM, enforcing the no-arbitrage bound $0 < Q(T_m) \le Q(T_{m-1})$, using a 1D root search (bisection/Newton/Brent).
- Linear interpolation of $-\ln Q(t)$ implies piecewise-constant forward default rate $h(t)$ between skeleton points, and no-arbitrage requires $h(t) \ge 0$ (equivalently $Q$ non-increasing).

### (B) Reasoned Inference (Derived from A)

- Given piecewise-constant $h$ on $(T_{k-1}, T_k]$, we can equivalently bootstrap either $Q(T_k)$ or $h_k$, because $Q(T_k) = Q(T_{k-1}) e^{-h_k(T_k - T_{k-1})}$. (This follows by integrating $Q(t) = \exp\left(-\int_0^t h\right)$.)
- The objective function for each bootstrap step can be written either as MTM $V(0; Q(T_m))$ or as spread error $f(h_m) = S_{\text{model}}(T_m; h_1, \ldots, h_m) - S_{\text{mkt}}(T_m)$. Monotonicity of $Q$ gives natural bounds for root search.
- A bump in a single quote should ideally have local impact on nearby maturities; "localness" is a stated desirable curve property and motivates bootstrapping choices.

### (C) Speculation (Clearly Labeled; Minimal)

- **Speculation:** In many modern markets, the discount curve for collateralized CDS is often aligned to OIS discounting. This is not established in the attached primary source passages (which reference a Libor discount curve), so we do not adopt it as a fact here. To be certain for your desk, we need your discounting policy and curve set.

---

## 0. Setup

### Conventions used in this chapter

We follow the primary source's reduced-form CDS valuation setup: pricing uses a discount curve $Z(0,t)$, a survival curve $Q(0,t)$, and an expected recovery $R$.

Independence between default time and interest rates is assumed when writing "risky discount factors" as products $Z(0,t) Q(0,t)$.

Premium leg includes the contingent payment of coupon accrued at default; the primary source provides an integral form and a standard approximation.

Bootstrapping is performed on a set of market CDS quotes $(S_m, T_m)$, sorted by maturity, to produce survival probabilities at "skeleton" points $Q(T_m)$ that reprice the quotes exactly (within tolerance).

**Market-convention caveat (non-negotiable):** The attached sources describe common CDS mechanics (e.g., premium is "typically" quarterly on Actual/360, IMM-style date conventions, etc.). But the sources here do not fully pin down modern ISDA standardization details (standard coupon + upfront regime by currency, exact accrual-at-default conventions per ISDA definitions/vintage, exact settlement calendars, OIS vs Libor discounting in post-crisis standard practice). I'm not sure for those. To be certain we would need: (i) the exact ISDA Definitions version/vintage used for the market quotes, (ii) the currency/region and its standard coupons and day count, (iii) the desk's discounting policy and curve set, and (iv) the exact accrual and settlement assumptions.

### Notation glossary (symbols + definitions)

| Symbol | Definition |
|--------|------------|
| $t$ | Time (years) |
| $T$ | CDS protection end date (years) |
| $t_n$ | Premium payment dates, $n=1,\ldots,N$, with $t_0=0$ |
| $\Delta_n := \Delta(t_{n-1}, t_n)$ | Accrual fraction between $t_{n-1}$ and $t_n$ |
| $Z(0,t)$ | Discount factor from $0$ to $t$ |
| $Q(0,t)$ | Survival probability to $t$ |
| $h(t)$ | Instantaneous forward default rate with $Q(t) = \exp\left(-\int_0^t h\right)$ |
| $R$ | Expected recovery rate (% of par) |
| $\text{RPV01}(0,T)$ | Risky PV01 of premium leg (PV of 1 bp/year premium stream, including accrued-at-default, in the approximation used) |
| $S_m$ | Market spread quote for maturity $T_m$ |

---

## 1. Core Concepts (Definitions First)

### 1.1 Survival curve $Q(0,t)$

**Formal Definition:** $Q(0,t) = \mathbb{Q}(\tau > t)$, the risk-neutral probability the reference entity survives past $t$.

**Intuition:** $Q$ is a "credit discount factor": it shrinks expected cash flows by the chance they are canceled by default.

**Trading/Risk Practice:** A CDS curve desk needs $Q(0,t)$ to mark CDS positions and compute sensitivities; without calibration to market spreads, MTMs are "meaningless."

### 1.2 Hazard / forward default rate $h(t)$

**Formal Definition:**

$$Q(t) = \exp\left(-\int_0^t h(s)\,ds\right)$$

**Intuition:** $h(t)$ is the instantaneous default intensity "per year," conditional on survival to $t$.

**In Practice:** Bootstrapping often parameterizes the curve in terms of $h(t)$ because piecewise-constant $h$ is a stable interpolation choice (via linear interpolation in $-\ln Q$).

### 1.3 "Skeleton points" and interpolation

**Formal Definition:** Skeleton points are maturities $\{T_m\}$ where market quotes are observed; the bootstrap solves for $\{Q(T_m)\}$.

**Intuition:** Quotes are sparse; we need a smooth-enough curve between them to price off-the-run maturities.

**In Practice:** The primary source discusses alternative interpolation quantities and rejects some for stability and arbitrage reasons (e.g., certain forward-rate interpolations can oscillate).

### 1.4 Bootstrapping (sequential calibration)

**Formal Definition:** A curve construction approach that starts at the shortest maturity and solves sequentially for one new parameter (e.g., $Q(T_m)$ or $h_m$) at each step so each market quote is repriced exactly.

**Intuition:** Each quote "pins down" one new degree of freedom; earlier parts of the curve stay fixed when adding longer maturities.

**In Practice:** Localness (limited propagation of quote bumps) and speed are desirable properties, and the bootstrap is chosen for these reasons.

---

## 2. Pricing Objects Needed for Bootstrapping (Build the Minimum Toolkit)

### 2.1 Discount curve inputs $Z(0,t)$

Treated as given inputs (we do not bootstrap $Z$ here). The CDS valuation formulas in the primary source use $Z(t,T)$ as a Libor discount curve anchored to the CDS effective date.

### 2.2 Survival curve outputs $Q(0,t)$, default increments, hazard

Outputs are $Q(0,t_n)$ at relevant grid points and/or piecewise hazards $h_k$.

Default probability over an interval $(t_{i-1}, t_i]$ is approximated on a grid as:

$$\Delta PD_i \approx Q(0, t_{i-1}) - Q(0, t_i)$$

consistent with discretizing $-dQ$ over intervals.

### 2.3 Recovery convention $R$

The primary source uses $R$ as an expected recovery rate as a percentage of par and values the protection payment as $(1-R)$ independent of rates and default time.

If your desk uses a different recovery convention (e.g., recovery of market value, recovery of Treasury, stochastic recovery), I'm not sure based on the excerpts above. We would need the explicit convention used in your documentation/model governance.

### 2.4 CDS legs and the "par condition"

We build the minimum valuation objects needed to impose par (zero MTM at inception):

#### (i) Premium leg PV

In the primary source, the premium leg PV is written as:

$$\text{Premium PV} = S_0 \cdot \text{RPV01}(0,T)$$

for a new CDS at $t=0$.

The risky PV01 includes both scheduled premium payments and coupon accrued at default.

#### (ii) Protection leg PV

$$\text{Protection PV}(0,T) = (1-R) \int_0^T Z(0,s) \left(-dQ(0,s)\right)$$

#### (iii) Accrued premium on default

The primary source explicitly includes this as a contingent payment; it also provides a practical approximation, motivated by "default occurs on average halfway through the period," producing a convenient trapezoid-style RPV01.

**Impact if omitted:** the source notes incorporating accrued premium at default lowers breakeven spreads (qualitatively shown in its figure discussion).

---

## 3. Math and Derivations (Step-by-Step)

### 3.1 Par condition for bootstrapping

A new CDS at $t=0$ has zero value at inception: $V(0) = 0$.

For a long-protection position, the primary source writes an MTM of the form:

$$V(0) = \text{Protection PV}(0,T) - S_0 \cdot \text{RPV01}(0,T)$$

leading to the breakeven spread formula (equation (6.5)).

So the par condition is:

$$\boxed{\text{Premium PV} = \text{Protection PV} \iff S_0 \cdot \text{RPV01}(0,T) = \text{Protection PV}(0,T)}$$

### 3.2 Express both legs in terms of $Z(0,t)$, $Q(0,t)$, default increments, and $R$

#### Premium leg: RPV01 with accrued-at-default approximation

The source gives (after approximation):

$$\boxed{\text{RPV01}(0,T) = \frac{1}{2} \sum_{n=1}^{N} \Delta_n Z(0,t_n) \left(Q(0,t_{n-1}) + Q(0,t_n)\right)}$$

and Premium PV $= S_0 \cdot \text{RPV01}(0,T)$.

**Unit check:**
- $S_0$ is "per year" (decimal)
- $\Delta_n$ is "years"
- So $S_0 \Delta_n$ is dimensionless cashflow fraction of notional, and multiplying by discount factors yields PV per unit notional.

#### Protection leg: integral and discretization

The source expresses:

$$\text{Protection PV}(0,T) = (1-R) \int_0^T Z(0,s) \left(-dQ(0,s)\right)$$

Discretizing time into points $t_k$ (not necessarily the coupon dates) gives a Riemann-sum approximation:

$$\text{Protection PV} \approx (1-R) \sum_{k=1}^{K} Z(0,t_k) \left(Q(0,t_{k-1}) - Q(0,t_k)\right)$$

and the primary source improves accuracy by averaging lower/upper bounds (trapezoid) to obtain:

$$\boxed{\text{Protection PV}(0,T) \approx \frac{1-R}{2} \sum_{k=1}^{K} \left(Z(0,t_{k-1}) + Z(0,t_k)\right) \left(Q(0,t_{k-1}) - Q(0,t_k)\right)}$$

**Unit check:**
- $(1-R)$ is dimensionless (loss fraction)
- The sum is discount factor $\times$ default probability increment, dimensionless
- So protection PV is PV per unit notional.

### 3.3 Derive the breakeven spread formula used for bootstrapping

Plugging the discretized protection PV and RPV01 into $V(0)=0$, the primary source obtains:

$$\boxed{S_0 = \frac{(1-R)}{2} \cdot \frac{\sum_{k=1}^{K} \left(Z(0,t_{k-1}) + Z(0,t_k)\right) \left(Q(0,t_{k-1}) - Q(0,t_k)\right)}{\text{RPV01}(0,T)}}$$

### 3.4 Discretization assumptions and what is approximated

Premium payments occur on premium dates $t_n$ with accrual fractions $\Delta_n$. In practice the schedule is commonly quarterly and built from "IMM-like" dates and rolled for business days, with day count "typically Actual/360," per the primary source.

The protection leg integral is approximated on a grid $\{t_k\}$; accuracy improves as step size shrinks, and the trapezoid averaging improves accuracy materially.

### 3.5 Sanity checks (do these while deriving/implementing)

| Check | Expected |
|-------|----------|
| Spreads | 100 bp $= 0.01$; avoid mixing bp and decimals |
| Hazard units | $h(t)$ is in $1/\text{year}$ |
| Survival | $Q \in [0,1]$, non-increasing in $t$ |
| PV | Per unit notional (multiply by notional for currency PV) |
| Default increments | $Q(t_{k-1}) - Q(t_k) \ge 0$ |

---

## 4. Bootstrapping Algorithm (The Core of the Chapter)

This section turns the par condition into a sequential curve-construction workflow.

### A) Inputs

You need:

1. **Market CDS quotes** $\{(T_m, S_m)\}_{m=1}^{M}$, sorted by maturity.
2. **Discount curve** $Z(0,t)$ at needed times (input).
3. **Recovery assumption** $R$, assumed constant across maturities in the bootstrap description.
4. **Payment schedule** $\{t_n\}$ and accrual fractions $\Delta_n$. Premiums are "typically" quarterly Actual/360 with date adjustments; this is input-dependent.
5. **Integration grid** $\{t_k\}$ for protection PV discretization (can be aligned with coupon dates or finer).

**Quotes format:**

The bootstrap algorithm in the primary source is described for CDS market spreads $S_m$.

Upfront CDS exist and are discussed as a variation where the premium leg is exchanged for a single upfront payment; the protection leg is unchanged.

I'm not sure how to map your specific desk's standard-coupon+upfront quotes into the exact bootstrapping objective without the precise market standard/ISDA quoting convention and settlement assumptions.

### B) Parameterization choices (must be explicit)

#### Preferred in the primary source (stable): piecewise-constant $h(t)$

The source shows that linearly interpolating $f(t) = -\ln Q(t)$ implies a piecewise-constant forward default rate $h(t)$ between skeleton points.

For $T_{n-1} < t < T_n$:

$$\boxed{h(t) = \frac{1}{T_n - T_{n-1}} \ln\left(\frac{Q(T_{n-1})}{Q(T_n)}\right), \quad Q(t) = Q(T_{n-1}) \exp\left(-(t - T_{n-1}) h(t)\right)}$$

No-arbitrage requires $h(t) \ge 0$, which corresponds to $Q(T_n) \le Q(T_{n-1})$.

#### Alternatives discussed in the source

The source discusses linear interpolation of the zero default rate $z(t) = -\frac{1}{t} \ln Q(t)$ and notes it can create undesirable jumps in the forward default rate and may not preserve arbitrage-free properties between skeleton points.

The source also warns about instability (saw-tooth oscillations) when interpolating the instantaneous forward default rate directly.

### C) Sequential solve ("bootstrap")

**The primary source's bootstrap steps:**

1. Initialize $Q(T_0 = 0) = 1$.
2. For $m = 1, 2, \ldots, M$: solve for $Q(T_m)$ such that the MTM of the $T_m$-maturity CDS with market spread $S_m$ is zero, using equation (6.5).
3. Enforce the no-arbitrage bound $0 < Q(T_m) \le Q(T_{m-1})$.
4. Add $(T_m, Q(T_m))$ to the curve and proceed.

**Extrapolation outside quotes:**

Before the first maturity and beyond the last, the source assumes flat forward default rate at the boundary level.

### D) Numerical methods

Step (3) requires a 1D root search; the source names bisection, Newton-Raphson, Brent.

**Objective function (spread form):**

$$f(Q(T_m)) = V(0; Q(T_m)) \quad \text{or} \quad f(h_m) = S_{\text{model}}(T_m) - S_m$$

You stop when $|f| \le \text{tolerance}$.

**Bounds:** $0 < Q(T_m) \le Q(T_{m-1})$ implies $h_m \ge 0$ under piecewise-constant $h$.

**Failure to converge / inconsistent quotes:**

The source notes root search may fail for steeply inverted spread curves implying arbitrage; the user must decide whether to relax bounds or fix inputs (re-mark spreads or adjust recovery), and the issue should be reported.

### E) Output

- **Skeleton survival probabilities** $\{Q(T_m)\}$
- **Piecewise hazards** $\{h_m\}$ implied by $-\ln Q$ interpolation
- **Interpolated** $Q(0,t)$ for off-grid $t$ via the chosen interpolation rule (here: piecewise constant $h$)

---

## 5. Consistency Checks and No-Arbitrage Sanity Constraints (Mandatory Section)

### 5.1 Hard constraints (must hold)

- $Q(0,0) = 1$
- $0 \le Q(0,t) \le 1$ for all $t$ (Survival probability meaning; also $-\ln Q \ge 0$)
- $Q(0,t)$ non-increasing in $t$ (No-arbitrage bound in bootstrap; also $h(t) \ge 0$)
- Interval default probabilities $\Delta PD_k = Q(t_{k-1}) - Q(t_k) \ge 0$
- Hazard rates $h_k \ge 0$ under piecewise-constant hazard interpolation; the source explicitly states no-arbitrage requires $h(t) \ge 0$

### 5.2 Repricing (calibration) checks

Reprice each calibration CDS using the bootstrapped curve and verify model spread equals market spread to tolerance (the source's "exact fit" goal is extremely tight in bp terms).

### 5.3 Stability checks

- **Localness:** bumping a 5Y quote should mainly affect nearby maturities, not the whole curve (a desired property stated by the source)
- **Numerical stability:** small quote bumps should not create oscillatory hazards; the source highlights instability risks for certain interpolation schemes

### 5.4 What to do when quotes are inconsistent

The source: handle root-failure; user decides whether to relax bounds; report non-monotone survival; address by re-marking spreads or adjusting recovery.

I'm not sure about a "standard" smoothing/constrained fitting procedure beyond this, based on the provided excerpts. To be certain, we'd need the book sections (or your policy) that specify smoothing/regularization choices and constraints.

---

## 6. Measurement & Risk (Only What Belongs in Chapter 42)

### 6.1 Downstream uses of the bootstrapped survival curve

**Pricing CDS:** Given $Z(0,t)$, $Q(0,t)$, and $R$, the CDS PV is determined; calibration ensures MTMs are meaningful.

**Preview (not a full chapter here):** Once $Q$ is known, you can compute expected loss–type quantities and price other credit cashflows that depend on default timing via $-dQ$. This follows from the protection PV integral structure.

### 6.2 High-level curve sensitivities (conceptual only)

A CDS's sensitivity to the spread curve is often summarized by measures like Credit DV01 (change in value for a 1 bp move). The primary source defines Credit DV01 and links it to RPV01 in certain cases.

**Conceptual distinction:** bumping a market quote changes calibrated $Q$ and therefore changes both premium and protection leg PVs; this is "curve risk," not simply multiplying RPV01 by a spread change.

### 6.3 Failure modes (common in implementations)

| Failure Mode | Impact |
|--------------|--------|
| **Recovery sensitivity** | $R$ enters protection PV as $(1-R)$; different $R$ implies different hazards for the same spreads |
| **Discount curve sensitivity** | $Z(0,t)$ weights all PV terms; mis-specified discounting shifts the implied curve |
| **Schedule/day-count errors** | Premium leg mechanics use day count fractions; using inconsistent calendars/day counts will distort calibration |
| **Accrued-at-default treatment** | Ignoring accrued-at-default changes RPV01 and hence par spread |
| **Discretization bias** | Coarse integration grid for protection PV can introduce errors; the source motivates improved trapezoid averaging for accuracy |

---

## 7. Worked Examples (Fully Numeric)

All examples use unit notional unless otherwise stated. Spreads are annualized.

**Toy simplifications (stated explicitly):**

- We use a yearly premium payment schedule $t_n = 1, 2, \ldots, T$ with $\Delta_n = 1.0$ for arithmetic clarity. (Real CDS premium legs are "typically quarterly Actual/360," per the primary source; this is a toy simplification.)
- We discretize the protection leg integral on the same yearly grid with the trapezoid-style approximation consistent with the breakeven spread equation (6.5).
- Interpolation between skeleton points uses piecewise-constant hazard (linear in $-\ln Q$).

---

### Example A (Setup): Toy discount curve $Z(0,t)$ at quarterly dates out to 5y

We provide quarterly times $t = 0.25, 0.50, \ldots, 5.00$ and a toy discount factor that declines linearly (purely as an input curve).

Let the quarter-to-quarter decrement be $0.00375$ so that $Z(0,5) = 0.925$.

| $t$ (years) | $Z(0,t)$ |
|-------------|----------|
| 0.25 | 0.99625 |
| 0.50 | 0.99250 |
| 0.75 | 0.98875 |
| 1.00 | 0.98500 |
| 1.25 | 0.98125 |
| 1.50 | 0.97750 |
| 1.75 | 0.97375 |
| 2.00 | 0.97000 |
| 2.25 | 0.96625 |
| 2.50 | 0.96250 |
| 2.75 | 0.95875 |
| 3.00 | 0.95500 |
| 3.25 | 0.95125 |
| 3.50 | 0.94750 |
| 3.75 | 0.94375 |
| 4.00 | 0.94000 |
| 4.25 | 0.93625 |
| 4.50 | 0.93250 |
| 4.75 | 0.92875 |
| 5.00 | 0.92500 |

**Annual subset used in PV sums (toy schedule):**

$$Z_1 = 0.985, \; Z_2 = 0.970, \; Z_3 = 0.955, \; Z_4 = 0.940, \; Z_5 = 0.925$$

---

### Example B (Market quotes): Toy par CDS spreads for 1y, 3y, 5y and recovery

Assume recovery $R = 40\%$ (so $1-R = 0.60$).

**Toy market par spreads:**

| Maturity $T$ | Spread $S_{\text{mkt}}$ (bp) | Spread $S_{\text{mkt}}$ (decimal) |
|--------------|------------------------------|-----------------------------------|
| 1y | 120 | 0.0120 |
| 3y | 160 | 0.0160 |
| 5y | 200 | 0.0200 |

---

### Example C (First bootstrap step): Solve $h_1$ on $(0,1]$ to match 1y quote

**Goal:** find $Q(1)$ (equivalently $h_1$) so the model 1y par spread equals 120 bp using equation (6.5).

For $T=1$ with yearly grid:

**Protection PV numerator term** (using trapezoid discretization with $t_0=0, t_1=1$):

$$N = (Z_0 + Z_1)(Q_0 - Q_1) = (1 + 0.985)(1 - Q_1) = 1.985(1 - Q_1)$$

**Denominator term** (without the $\frac{1}{2}$ because it cancels as shown in Section 3):

$$D = Z_1(Q_0 + Q_1) = 0.985(1 + Q_1)$$

**Par condition implies:**

$$\frac{S}{1-R} = \frac{N}{D}$$

Here $S = 0.012$, $1-R = 0.60$, so:

$$x := \frac{S}{1-R} = \frac{0.012}{0.60} = 0.02$$

So solve:

$$0.02 = \frac{1.985(1-Q_1)}{0.985(1+Q_1)}$$

Compute helpful constant:

$$A = \frac{1.985}{0.985} = 2.015228$$

Then:

$$\frac{1-Q_1}{1+Q_1} = \frac{0.02}{2.015228} = 0.009926$$

Hence:

$$Q_1 = \frac{1 - 0.009926}{1 + 0.009926} = \frac{0.990074}{1.009926} \approx 0.98033$$

**Implied hazard on $(0,1]$:**

$$h_1 = -\ln Q_1 = -\ln(0.98033) \approx 0.01987 \;(= 1.987\%/\text{yr})$$

**Unit check:** $h_1$ is per year.

#### Root-finding illustration (one bisection iteration)

Define $f(h_1) = S_{\text{model}}(h_1) - 0.012$, with $Q_1 = e^{-h_1}$.

- **Lower bound** $h_L = 0 \Rightarrow Q_1 = 1 \Rightarrow (1-Q_1) = 0 \Rightarrow S_{\text{model}} = 0 \Rightarrow f(h_L) < 0$

- **Upper bound** $h_U = 0.10 \Rightarrow Q_1 = e^{-0.10} = 0.9048$
  - Compute $N = 1.985(1-0.9048) = 1.985(0.0952) = 0.1890$
  - Compute $D = 0.985(1+0.9048) = 0.985(1.9048) = 1.876$
  - Then $x = N/D = 0.1890/1.876 = 0.1007 \Rightarrow S_{\text{model}} = 0.60x = 0.0604$ (6040 bp) so $f(h_U) > 0$

- **Midpoint** $h_M = 0.05 \Rightarrow Q_1 = e^{-0.05} = 0.9512$:
  - $N = 1.985(1-0.9512) = 1.985(0.0488) = 0.0969$
  - $D = 0.985(1+0.9512) = 0.985(1.9512) = 1.921$
  - $x = 0.0969/1.921 = 0.0505 \Rightarrow S_{\text{model}} = 0.0303$ (3030 bp), still $> 120$ bp so $f(h_M) > 0$

So the next bisection interval becomes $[0, 0.05]$. (Continuing converges to $h_1 \approx 0.01987$.)

---

### Example D (Second step): Solve $h_2$ on $(1,3]$ to match 3y quote

We keep $h_1$ fixed (equivalently $Q(1) = 0.98033$) and solve $h_2$ so the 3y par spread is 160 bp. This is the sequential bootstrap idea.

For piecewise-constant hazard:

$$Q(2) = Q(1) e^{-h_2 \cdot (2-1)} = Q_1 e^{-h_2}, \quad Q(3) = Q(1) e^{-h_2 \cdot (3-1)} = Q_1 e^{-2h_2}$$

Let $e = \exp(-h_2)$. Then $Q_2 = Q_1 e$, $Q_3 = Q_1 e^2$.

**Target spread:**

$$S_{3Y} = 0.016, \quad x = \frac{S}{1-R} = \frac{0.016}{0.60} = 0.0266667$$

Using the simplified ratio $x = N/D$ with yearly grid:

**Known values:** $Z_0 = 1, Z_1 = 0.985, Z_2 = 0.970, Z_3 = 0.955$; $Q_0 = 1, Q_1 = 0.98033$.

Compute:

$$N = \sum_{k=1}^{3} (Z_{k-1} + Z_k)(Q_{k-1} - Q_k) = 0.039045 + (1-e)(1.91655 + 1.88713e)$$

$$D = \sum_{n=1}^{3} Z_n(Q_{n-1} + Q_n) = 2.90234 + 1.88793e + 0.93621e^2$$

where the constants come from substituting the $Z$'s and $Q_1$.

**Bisection / trial values:**

- Try $h_2 = 0.027 \Rightarrow e = 0.9734$: $x \approx 0.02468$ (too low vs 0.026667)
- Try $h_2 = 0.032 \Rightarrow e = 0.9685$: $x \approx 0.02799$ (too high)
- Midpoint $h_2 = 0.030 \Rightarrow e = \exp(-0.03) = 0.97045$:
  - $Q_2 = Q_1 e = 0.98033 \times 0.97045 = 0.95140$
  - $Q_3 = Q_1 e^2 = 0.98033 \times 0.94177 = 0.92283$
  - With these, $x \approx 0.02666 \approx 0.026667$ (match within rounding)

**So we take:**

$$\boxed{h_2 \approx 0.0300 \;(3.00\%/\text{yr}), \quad Q(3) \approx 0.92283}$$

**Unit check:** $h_2$ in $1/\text{yr}$; $Q$ dimensionless.

---

### Example E (Third step): Solve $h_3$ on $(3,5]$ to match 5y quote

Keep $h_1, h_2$ fixed (so $Q_1, Q_2, Q_3$ fixed) and solve $h_3$ so 5y spread is 200 bp.

**Piecewise-constant hazard on $(3,5]$:**

$$Q(4) = Q(3) e^{-h_3}, \quad Q(5) = Q(3) e^{-2h_3}$$

Let $e = \exp(-h_3)$. Then $Q_4 = Q_3 e$, $Q_5 = Q_3 e^2$.

**Target:**

$$S_{5Y} = 0.020, \quad x = \frac{S}{1-R} = \frac{0.020}{0.60} = 0.0333333$$

**Known values:**

- $Z_0 = 1, Z_1 = 0.985, Z_2 = 0.970, Z_3 = 0.955, Z_4 = 0.940, Z_5 = 0.925$
- $Q_0 = 1, Q_1 = 0.98033, Q_2 = 0.95140, Q_3 = 0.92283$

**Compute ratio $x = N/D$ with yearly grid:**

Known partial sums (years 0–3):
- $N_{1:3} = 0.15061$ (from $k=1,2,3$ terms)
- $D_{1:3} = 5.61429$ (from $n=1,2,3$ terms)

Unknown contribution (years 3–5) leads to:

$$N = 0.15061 + (1-e)(1.74876 + 1.72108e)$$
$$D = 6.48175 + 1.72108e + 0.85362e^2$$

**Trial hazards:**

- $h_3 = 0.040 \Rightarrow e = 0.96079 \Rightarrow x \approx 0.03183$ (too low)
- $h_3 = 0.045 \Rightarrow e = 0.95599 \Rightarrow x \approx 0.03368$ (too high)
- $h_3 = 0.044 \Rightarrow e = 0.95697 \Rightarrow x \approx 0.03331$ (very close)

**So take:**

$$\boxed{h_3 \approx 0.0440 \;(4.40\%/\text{yr})}$$

Then:

$$Q_4 = Q_3 e = 0.92283 \times 0.95697 = 0.88315$$
$$Q_5 = Q_3 e^2 = 0.92283 \times 0.91580 = 0.84527$$

---

### Example F (Repricing check): Reprice 1y/3y/5y CDS with $(h_1, h_2, h_3)$

We verify that using the bootstrapped curve reproduces the market spreads (within rounding).

**Bootstrapped hazards:**
- $h_1 = 0.01987$, $h_2 = 0.0300$, $h_3 = 0.0440$

**Survival at annual nodes:**
- $Q_1 = 0.98033$, $Q_2 = 0.95140$, $Q_3 = 0.92283$, $Q_4 = 0.88315$, $Q_5 = 0.84527$

| Maturity | Repricing |
|----------|-----------|
| **1y** | From Example C, model spread $= 120$ bp ✓ |
| **3y** | Using the 3y formula with $e = \exp(-0.03) = 0.97045$, we obtained $x \approx 0.026666$. Multiply by $(1-R) = 0.60$: $S \approx 0.01600 = 160$ bp ✓ |
| **5y** | Using Example E's ratio, $x \approx 0.03333$. Multiply by 0.60 gives $S \approx 0.02000 = 200$ bp ✓ |

---

### Example G (Monotonicity check): Compute $Q$ at quarterly nodes and verify non-increasing

We now compute the survival curve on quarterly nodes using the piecewise-constant hazards, using:

$$Q(t) = Q(T_{k-1}) \exp\left(-h_k (t - T_{k-1})\right), \quad t \in (T_{k-1}, T_k]$$

**Quarterly decay factors:**

| Segment | Decay Factor |
|---------|--------------|
| 0–1y | $a_1 = \exp(-h_1 \cdot 0.25) = \exp(-0.01987 \times 0.25) = \exp(-0.0049675) \approx 0.99504$ |
| 1–3y | $a_2 = \exp(-0.03 \times 0.25) = \exp(-0.0075) \approx 0.99253$ |
| 3–5y | $a_3 = \exp(-0.044 \times 0.25) = \exp(-0.011) \approx 0.98905$ |

**Quarterly $Q(t)$ table (rounded to 5 d.p.):**

| $t$ | $Q(0,t)$ |
|-----|----------|
| 0.00 | 1.00000 |
| 0.25 | 0.99504 |
| 0.50 | 0.99010 |
| 0.75 | 0.98519 |
| 1.00 | 0.98033 |
| 1.25 | 0.97300 |
| 1.50 | 0.96573 |
| 1.75 | 0.95851 |
| 2.00 | 0.95140 |
| 2.25 | 0.94429 |
| 2.50 | 0.93725 |
| 2.75 | 0.93030 |
| 3.00 | 0.92283 |
| 3.25 | 0.91273 |
| 3.50 | 0.90275 |
| 3.75 | 0.89286 |
| 4.00 | 0.88315 |
| 4.25 | 0.87355 |
| 4.50 | 0.86404 |
| 4.75 | 0.85461 |
| 5.00 | 0.84527 |

**Monotonicity:** Each entry is $\le$ the previous entry, so $Q$ is non-increasing and stays in $[0,1]$. This is consistent with no-arbitrage $h(t) \ge 0$.

**Numerical issues to watch:** rounding can make tiny increases; in production, enforce monotonicity by construction (bounds in the solver) as per the bootstrap algorithm.

---

### Example H (Discount curve sensitivity): Re-do Example E with a +20 bp parallel shift

We keep the same market spreads and recovery $R = 40\%$ but slightly change discounting.

**Toy shift rule:** apply a multiplicative factor $\exp(-0.002t)$ to discount factors (20 bp = 0.002 in continuous-rate terms).

**Annual discount factors become:**

| Year | Original | Shifted (+20 bp) |
|------|----------|------------------|
| 1 | 0.985 | $Z'_1 = 0.985 \times e^{-0.002} \approx 0.98303$ |
| 2 | 0.970 | $Z'_2 = 0.970 \times e^{-0.004} \approx 0.96612$ |
| 3 | 0.955 | $Z'_3 = 0.955 \times e^{-0.006} \approx 0.94927$ |
| 4 | 0.940 | $Z'_4 = 0.940 \times e^{-0.008} \approx 0.93248$ |
| 5 | 0.925 | $Z'_5 = 0.925 \times e^{-0.010} \approx 0.91575$ |

**Rebootstrap results (same sequential algorithm):**

- 1y step: $Q'_1 \approx 0.98036 \Rightarrow h'_1 \approx 0.01984$
- 3y step: solving gives essentially $h'_2 \approx 0.0300$ (very close in this toy)
- 5y step: solving gives $h'_3 \approx 0.0441$ (slightly higher than 0.0440)

| Parameter | Base curve | Shifted curve (+20 bp) |
|-----------|------------|------------------------|
| $h_1$ | 0.01987 | 0.01984 |
| $h_2$ | 0.03000 | 0.03000 |
| $h_3$ | 0.04400 | 0.04410 |

**Interpretation:** with heavier discounting, later cashflows are slightly less valuable; in this toy setup the calibration impact is small, but in real curves (with quarterly schedules and realistic discounting) the sensitivity can be larger.

---

### Example I (Recovery sensitivity): Rebootstrap with $R = 20\%, 40\%, 60\%$

We keep the same spreads and discount curve but change $R$. The protection leg scales with $(1-R)$.

We report the bootstrapped piecewise hazards:

| Recovery $R$ | $1-R$ | $h_1$ (0–1y) | $h_2$ (1–3y) | $h_3$ (3–5y) |
|--------------|-------|--------------|--------------|--------------|
| 20% | 0.80 | 0.0149 | 0.0225 | 0.0330 |
| 40% | 0.60 | 0.0199 | 0.0300 | 0.0440 |
| 60% | 0.40 | 0.0298 | 0.0450 | 0.0678 |

**Why hazards rise with $R$:** to justify the same spread with lower loss-given-default $(1-R)$, the model needs a higher default intensity. This aligns with the "credit triangle" intuition for continuously paid premiums $S = \lambda(1-R)$.

---

### Example J (Inconsistent quotes toy): A set implying negative hazards in naive bootstrap

Use $R = 40\%$ and the same discount curve. Consider an inverted quote set:

| Maturity | Spread |
|----------|--------|
| 1y | 500 bp |
| 3y | 100 bp |
| 5y | 120 bp |

**Step 1 (1y):**

$S_{1Y} = 0.05 \Rightarrow x = 0.05/0.60 = 0.08333$

Using the 1y closed-form (Example C), this implies $Q(1) \approx 0.9206$ (so hazard is high early).

**Step 2 (3y): can we match 100 bp with $Q(1) = 0.9206$?**

Market $S_{3Y} = 0.01 \Rightarrow x_{\text{mkt}} = 0.01/0.60 = 0.01667$

Now compute the minimum achievable $x$ under the no-arbitrage constraint $Q(3) \le Q(1)$. The smallest hazard on $(1,3]$ is $h_2 = 0 \Rightarrow Q(2) = Q(3) = Q(1)$ (no extra defaults after 1y). Under that,

Only the first-year interval contributes to protection PV:

$$N = (1 + Z_1)(1 - Q_1) = 1.985(1 - 0.9206) = 1.985(0.0794) = 0.1576$$

Premium denominator over 3 years is:

$$D = Z_1(1 + Q_1) + Z_2(Q_1 + Q_2) + Z_3(Q_2 + Q_3) \approx 5.4351$$

So:

$$x_{\min} = N/D \approx 0.1576/5.4351 = 0.0290$$
$$S_{\min} = (1-R) x_{\min} \approx 0.60 \times 0.0290 = 0.0174 \;(= 174 \text{ bp})$$

**But the market wants 100 bp $< 174$ bp.** Therefore, no non-increasing survival curve can fit both 1y=500 bp and 3y=100 bp in this setup: matching 3y would require $Q(3) > Q(1)$, i.e., a negative hazard on $(1,3]$.

This corresponds to the source's warning: root search may fail for steeply inverted curves implying arbitrage; the user may relax bounds or fix inputs (re-mark spreads or adjust recovery).

---

### Example K (Interpolation effect): Two interpolation methods for $Q(2y)$

We use the calibrated skeleton points from the base case $R = 40\%$:

- $Q(1) = 0.98033$, $Q(3) = 0.92283$

#### Method 1 (source-preferred): piecewise-constant hazard between 1 and 3

We already found $h_2 = 0.03$. Then:

$$Q(2) = Q(1) \exp(-0.03 \cdot (2-1)) = 0.98033 \times 0.97045 = 0.95140$$

#### Method 2 (source-discussed but not preferred): linear interpolation of zero default rate $z(t)$

The source defines $z(t)$ by $Q(t) = \exp(-z(t) \cdot t)$ and discusses linear interpolation of $z(t)$.

Compute:

$$z(1) = -\ln Q(1) / 1 = -\ln(0.98033) = 0.01987$$

$$z(3) = -\ln Q(3) / 3$$

Since $\ln(0.92283) \approx -0.0803$:

$$z(3) \approx 0.0803/3 = 0.02677$$

Linear interpolation in $z$ between 1 and 3 gives:

$$z(2) = \frac{z(1) + z(3)}{2} = \frac{0.01987 + 0.02677}{2} = 0.02332$$

Then:

$$Q(2) = \exp(-z(2) \cdot 2) = \exp(-0.04664) \approx 0.9544$$

**Comparison:**

| Method | $Q(2)$ |
|--------|--------|
| Piecewise-constant hazard | $\approx 0.9514$ |
| Linear-in-$z(t)$ | $\approx 0.9544$ |

**Implication for pricing a 2y instrument:** A slightly higher $Q(2)$ reduces expected default over $[0,2]$, impacting protection PV and par spread. The source cautions that certain interpolation schemes can produce undesirable forward-rate behavior and may not preserve arbitrage-free properties.

---

### Example L (Quote bump locality): Bump the 5y quote by +1 bp and rebootstrap

Keep 1y=120 bp and 3y=160 bp, but bump 5y from 200 bp to 201 bp. This tests localness.

**New 5y spread:** $S'_{5Y} = 0.0201$

**New target:** $x' = \frac{0.0201}{0.60} = 0.0335$

Because the bootstrap is sequential, $h_1$ and $h_2$ remain fixed (they are set by the 1y and 3y quotes). Only the last segment $h_3$ changes.

Re-solving Example E's 5y equation for $h_3$ gives:

- **Old:** $h_3 = 0.0440$ matched $x \approx 0.03333$
- **New:** $h'_3 \approx 0.04465$ matches $x' \approx 0.0335$

| Segment | Old hazard | New hazard |
|---------|------------|------------|
| 0–1y | 0.01987 | 0.01987 |
| 1–3y | 0.03000 | 0.03000 |
| 3–5y | 0.04400 | 0.04465 |

**Localness summary:** Only the 3–5y segment moved materially, matching the "local" behavior desired by the source.

---

## 8. Practical Notes

### 8.1 Minimum input checklist (production bootstrap)

1. Discount curve $Z(0,t)$ at all needed times (and clearly documented discounting convention)
2. Market CDS quotes $(T_m, S_m)$ or quote format details (par spread vs upfront)
3. Recovery assumption $R$ (and whether it is constant across maturities)
4. Premium schedule $t_n$, day count fractions $\Delta_n$, business day convention and calendar
5. Accrual-at-default treatment (include or exclude; approximation choice)
6. Protection leg payment timing / integration grid resolution (step size $M$ per year)

### 8.2 Common pitfalls

| Pitfall | Description |
|---------|-------------|
| **bp vs decimals** | 100 bp is 0.01, not 0.0001 |
| **Ignoring coupon accrued at default** | Changes RPV01 and breakeven spread |
| **Confusing quotation vs cashflow accrual** | The source explicitly warns these are different; quotation is a convention, accrued-at-default changes cashflows and must be modeled |
| **Discount curve mismatch** | Using a curve inconsistent with market practice will distort implied survival |
| **Wrong calendar / schedule** | Premium dates are adjusted for weekends/holidays; day count is typically Actual/360; mismatches cause miscalibration |
| **Too-coarse protection integration** | Accuracy issues; the source motivates improved approximations and step refinement |

### 8.3 Verification tests

- **Repricing:** all calibration tenors reproduce market quotes to tolerance
- **Positivity/monotonicity:** $Q$ decreasing, $h \ge 0$
- **Stability under bumps:** check localness of hazards
- **Regression tests:** same inputs $\Rightarrow$ same curve (deterministic build)
- **Arbitrage flagging:** detect cases where solver cannot find root in bounds; report

---

## 9. Summary & Recall

### 9.1 Ten-Bullet Executive Summary

1. A CDS is priced as premium leg PV minus protection leg PV; at inception, par implies $V(0)=0$.
2. Premium PV at inception equals $S_0 \cdot \text{RPV01}(0,T)$.
3. RPV01 includes coupon accrued at default; the source provides a practical approximation that is trapezoidal in survival probabilities.
4. Protection PV is $(1-R)\int Z(0,s)(-dQ)$ and is discretized for computation.
5. A trapezoid-style approximation for protection PV uses $\frac{1}{2}(Z_{k-1} + Z_k)(Q_{k-1} - Q_k)$ and improves accuracy.
6. The breakeven spread formula (equation (6.5)) expresses $S_0$ as protection PV divided by RPV01.
7. Bootstrapping constructs $Q(T_m)$ at market maturities sequentially so each quoted CDS reprices exactly.
8. The source-preferred interpolation is linear in $-\ln Q$, equivalent to piecewise-constant forward default rate $h(t)$.
9. No-arbitrage requires $Q$ non-increasing and $h(t) \ge 0$; enforce bounds $0 < Q(T_m) \le Q(T_{m-1})$ in the solver.
10. Inconsistent/inverted quotes can break the bootstrap (no root in bounds); report and address by re-marking inputs or revisiting recovery assumptions.

### 9.2 Cheat Sheet

#### Core equations (initiation, par CDS)

**Premium PV:**
$$\text{Premium PV} = S_0 \cdot \text{RPV01}(0,T)$$

**RPV01 (including accrued-at-default approximation):**
$$\boxed{\text{RPV01}(0,T) = \frac{1}{2} \sum_{n=1}^{N} \Delta_n Z(0,t_n) \left(Q(0,t_{n-1}) + Q(0,t_n)\right)}$$

**Protection PV (trapezoid discretization):**
$$\boxed{\text{Protection PV}(0,T) \approx \frac{1-R}{2} \sum_{k=1}^{K} \left(Z(0,t_{k-1}) + Z(0,t_k)\right) \left(Q(0,t_{k-1}) - Q(0,t_k)\right)}$$

**Breakeven spread (equation (6.5)):**
$$\boxed{S_0 = \frac{\text{Protection PV}(0,T)}{\text{RPV01}(0,T)}}$$

**Piecewise-constant hazard interpolation:**

$$Q(t) = \exp\left(-\int_0^t h\right)$$

Between skeleton points, $h$ constant and:

$$\boxed{h = \frac{1}{T_n - T_{n-1}} \ln\left(\frac{Q(T_{n-1})}{Q(T_n)}\right), \quad Q(t) = Q(T_{n-1}) e^{-h(t - T_{n-1})}}$$

#### Bootstrap steps (primary source)

1. Set $Q(0) = 1$
2. For each maturity $T_m$: solve for $Q(T_m)$ s.t. MTM of CDS with spread $S_m$ is zero (use eq. 6.5)
3. Enforce $0 < Q(T_m) \le Q(T_{m-1})$
4. Use 1D root finding (bisection/Newton/Brent)

#### Sanity checklist

- [ ] $Q(0) = 1$, $Q \in [0,1]$, decreasing
- [ ] $\Delta PD \ge 0$
- [ ] $h \ge 0$
- [ ] Reprice all calibration tenors to market

### 9.3 Flashcards (35 Q/A)

| Q | A |
|---|---|
| What is $Q(0,t)$? | Risk-neutral survival probability to $t$ |
| Define $h(t)$ in terms of $Q(t)$ | $Q(t) = \exp\left(-\int_0^t h(s)\,ds\right)$ |
| What is the CDS "par condition" at inception? | $V(0) = 0$ so Premium PV = Protection PV |
| Premium PV at inception equals what? | $S_0 \cdot \text{RPV01}(0,T)$ |
| What does RPV01 represent? | PV of risky premium payments per unit spread (includes accrued-at-default in the approximation) |
| Protection PV integral form? | $(1-R) \int_0^T Z(0,s) (-dQ(0,s))$ |
| Why does $Z(0,t) Q(0,t)$ appear? | Under independence of rates and default, risky discount factor factorizes |
| What does $R$ denote in the primary source? | Expected recovery as % of par |
| Why is bootstrapping "local"? | Changing a quote ideally affects nearby maturities; localness is a desired curve property |
| What is a "skeleton point"? | A maturity $T_m$ where the market provides a quote and we solve for $Q(T_m)$ |
| What bounds does the source impose on $Q(T_m)$? | $0 < Q(T_m) \le Q(T_{m-1})$ |
| How do we get piecewise-constant $h$ from $Q$ skeleton points? | Linearly interpolate $-\ln Q$; then $h$ is constant between points |
| Give the formula for $h$ on $(T_{n-1}, T_n)$ | $h = \frac{1}{T_n - T_{n-1}} \ln\left(\frac{Q(T_{n-1})}{Q(T_n)}\right)$ |
| How do we compute $Q(t)$ between skeleton points under piecewise-constant hazard? | $Q(t) = Q(T_{n-1}) \exp(-(t - T_{n-1}) h)$ |
| What no-arbitrage condition does the source state for $h(t)$? | $h(t) \ge 0$ |
| What does "accrued premium at default" mean in this model? | If default occurs during a coupon period, premium accrues and is paid contingent on default; the source approximates it as half-period on average |
| What numerical methods are suggested for solving each bootstrap step? | Bisection, Newton-Raphson, Brent |
| What can cause solver failure per the source? | Steeply inverted curve implying arbitrage; root search may fail |
| What remedies does the source mention for such inconsistency? | Decide whether to relax solver bounds; report non-monotone survival; address by re-marking spreads or adjusting recovery |
| What is the main objective of survival curve calibration? | Build a curve that reprices the full term structure of quoted CDS spreads |
| What is meant by "exact fit" in the source's curve properties? | Extremely tight spread error tolerance (order $10^{-4}$ bp) |
| Why can interpolation choice matter for stability? | Some schemes can create oscillatory forward default rates |
| What is an upfront CDS (conceptually)? | A CDS where the premium leg is a single upfront amount rather than running premiums; protection leg unchanged |
| Is "clean MTM" the same as "accrued at default"? | No; clean MTM is a quotation convention; accrued-at-default changes cashflows |
| What does $-dQ$ represent in protection PV? | Default probability mass over an interval, discretized as $Q_{k-1} - Q_k$ |
| What does increasing $R$ do to implied hazard (holding spreads fixed)? | Typically increases implied hazard because $(1-R)$ decreases |
| What is the "credit triangle" in the source? | In a continuous premium approximation with constant hazard, par spread satisfies $S = \lambda(1-R)$ |
| Why do we not bootstrap the discount curve in this chapter? | Discounting is treated as an input; survival curve calibration is the focus |
| What is meant by the "integration steps per year" $M$ in protection leg discretization? | The number of subintervals used to approximate the protection integral; larger $M$ improves accuracy |
| What accuracy improvement does the source propose for protection PV integration? | Average upper and lower bounds (trapezoid) to make error quadratic in step size |
| What does it mean that PV is "linear in $Q(t)$" in the bootstrap step? | The source notes MTM is linear in survival probabilities, helping control PV tolerance via tolerance in $Q$ |
| What is the output of the bootstrap algorithm? | A survival curve with points $(0, 1.0), (T_1, Q(T_1)), \ldots, (T_M, Q(T_M))$ |
| How does the source extrapolate beyond the last maturity? | Flat forward default rate at the last interpolated value |
| Why is schedule construction a risk for curve building? | Premium payments depend on day count and business day adjustments; errors distort PVs |
| What is the key reason a non-calibrated survival curve is unusable for MTM? | It will not reprice market spreads, so the resulting MTM is meaningless |

---

## 10. Mini Problem Set (18 Questions)

*Provide brief solution sketches for questions 1–9 only.*

---

**1. (Warm-up)** Convert 75 bp to decimal spread per year.

**Sketch:** $75 \text{ bp} = 75 \times 10^{-4} = 0.0075$

---

**2. (Warm-up)** If $R = 40\%$ and $S = 150$ bp, compute $x = S/(1-R)$.

**Sketch:** $S = 0.015$, $1-R = 0.6$, so $x = 0.015/0.6 = 0.025$

---

**3. (Concept)** State the par condition for a new CDS in terms of premium PV and protection PV.

**Sketch:** $V(0) = 0 \Rightarrow S_0 \cdot \text{RPV01}(0,T) = \text{Protection PV}(0,T)$

---

**4. (Concept)** Under piecewise-constant hazard $h$ on $(T_{k-1}, T_k]$, express $Q(T_k)$ in terms of $Q(T_{k-1})$ and $h$.

**Sketch:** $Q(T_k) = Q(T_{k-1}) \exp(-h(T_k - T_{k-1}))$

---

**5. (Computation)** Using the 1y closed-form derived in Example C, compute $Q(1)$ for $Z_1 = 0.98$, $R = 40\%$, $S = 100$ bp.

**Sketch:** $x = 0.01/0.6 = 0.0166667$. $A = (1+0.98)/0.98 = 2.02041$. $y = x/A = 0.008247$. $Q_1 = (1-y)/(1+y) \approx 0.9836$

---

**6. (Computation)** Given $Q(1) = 0.98$ and $h_2 = 0.03$ on $(1,3]$, compute $Q(3)$.

**Sketch:** $Q(3) = Q(1) e^{-0.03 \cdot 2} = 0.98 e^{-0.06} \approx 0.98 \times 0.9418 = 0.9230$

---

**7. (Sanity)** List three no-arbitrage checks for a survival curve.

**Sketch:** $Q(0) = 1$; $Q$ non-increasing; $0 \le Q \le 1$; $\Delta PD \ge 0$; $h \ge 0$

---

**8. (Numerical method)** Describe how bisection would be used to solve for $Q(T_m)$.

**Sketch:** Define $f(Q_m) = V(0; Q_m)$. Start with bounds $Q_L \in (0, Q_{m-1}]$, $Q_U = Q_{m-1}$ s.t. $f(Q_L)$ and $f(Q_U)$ have opposite signs; bisect until tolerance. Bounds follow no-arbitrage.

---

**9. (Interpretation)** If recovery $R$ increases but spreads are unchanged, should hazards rise or fall (qualitatively)?

**Sketch:** Rise, because $(1-R)$ decreases so to keep $S$ the same, default intensity must increase (credit triangle intuition).

---

**10.** Implement (on paper) one bootstrap step for $T_1$ using equation (6.5) with a quarterly schedule (no need to fully compute).

---

**11.** Show how changing the integration grid $M$ affects protection PV approximation error qualitatively.

---

**12.** For a given curve, explain why ignoring accrued-at-default changes the par spread.

---

**13.** Construct an example where $Q$ increases between two maturities and explain why this violates no-arbitrage.

---

**14.** Explain why linear interpolation of zero default rate can create jumps in forward default rate.

---

**15.** Describe how you would detect inconsistent quotes before bootstrapping (conceptual).

---

**16.** Given two curves calibrated to the same quotes but different interpolation schemes, which curve might be preferred and why (stability/localness)?

---

**17.** Explain how you would compute a 2y par spread using a 1y/3y-calibrated curve.

---

**18.** Explain why extrapolation assumptions (before $T_1$, after $T_M$) can matter for pricing off-the-run maturities.

---

## Source Map

### (A) Verified Facts — cite specific sources

- CDS valuation framework (premium/protection leg split, independence assumption, risky discount factors): O'Kane Ch 6-7
- RPV01 formula with accrued-at-default approximation: O'Kane Ch 6
- Protection PV integral and trapezoid discretization: O'Kane Ch 6
- Breakeven spread formula (equation 6.5): O'Kane Ch 6
- Bootstrap algorithm (sequential solve, no-arbitrage bounds, root search methods): O'Kane Ch 7
- Piecewise-constant hazard interpolation (linear in $-\ln Q$): O'Kane Ch 7
- Interpolation alternatives and stability warnings: O'Kane Ch 7
- Recovery convention (% of par, constant): O'Kane Ch 3, 6

### (B) Reasoned Inference — note derivation logic

- Equivalence of bootstrapping $Q(T_k)$ vs $h_k$: follows from $Q(T_k) = Q(T_{k-1}) e^{-h_k(T_k - T_{k-1})}$
- Objective function forms (MTM vs spread error): algebraic rearrangement of par condition
- Localness property: follows from sequential structure (earlier curve segments fixed)

### (C) Speculation — flag uncertainties

- OIS vs Libor discounting in modern practice: not established in primary source excerpts
- Exact ISDA standardization details (standard coupon + upfront by currency, exact conventions): not covered in primary source excerpts
- Smoothing/constrained fitting procedures beyond basic bootstrap: not specified in excerpts
