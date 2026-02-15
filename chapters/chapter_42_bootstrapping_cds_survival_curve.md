# Chapter 42: Bootstrapping a CDS Survival Curve from Market Quotes

---

## Introduction

Before you can price a CDS, value a credit derivative, or compute the risk of a protection position, you need one fundamental object: the **survival curve**. This curve—the term structure of survival probabilities $Q(0,t)$—plays the same role in credit as the discount curve $Z(0,t)$ does in rates.

But here is the challenge: the survival curve is not directly observable. The market quotes *spreads* (par CDS premium rates) at a handful of maturities—perhaps 6M, 1Y, 3Y, 5Y, 7Y, and 10Y. From these sparse data points, you must infer a continuous function that tells you the survival probability to *any* date, not just the benchmark maturities. This is an **inverse problem**, and solving it requires a systematic algorithm: **bootstrapping**.

Bootstrapping in this context means: choose an interpolation rule (how the curve behaves *between* quoted maturities), then solve sequentially for curve parameters so that each quoted CDS has zero mark-to-market at inception.

Prerequisites: [Chapter 17 — Curve Construction — Bootstrapping, Interpolation, and the Spline Zoo](chapters/chapter_17_curve_construction_bootstrapping_interpolation.md), [Chapter 36 — Survival Probabilities and Hazard Rates](chapters/chapter_36_survival_probabilities_hazard_rates.md), [Chapter 41 — CDS Indices: Mechanics, Coupons, and Rolls](chapters/chapter_41_cds_indices_mechanics_coupons_rolls.md)  
Follow-on: [Chapter 43 — CDS Risks and Hedging](chapters/chapter_43_cds_risks_hedging.md), [Chapter 44 — CDS Relative Value: Trading Frameworks](chapters/chapter_44_cds_relative_value_trading_frameworks.md)

## Learning Objectives
- Translate a CDS spread quote into the objects that enter pricing: schedule, accrual factors, discount factors, and survival probabilities.
- Implement a sequential bootstrap that solves for $Q(T_m)$ (or hazards) with explicit no-arbitrage bounds.
- Explain why linear interpolation of $-\ln Q(t)$ corresponds to piecewise-constant forward default rates.
- Run a practical QA checklist: repricing, monotonicity/positivity, locality, and convention consistency.
- Define and interpret curve risks (bucketed CS01 and Rec01) with explicit bump object, bump size, units, and sign.

**What happens if you get this wrong?** If your bootstrapped hazard rates are inconsistent with market spreads, your CDS mark-to-market values are meaningless—they reflect your model error, not the market's view of credit risk. If you use an interpolation method that creates oscillatory forward default rates, your hedges may appear local when they are not, leading to unexpected P&L when quotes move. If your curve implies negative hazard rates (an arbitrage), you have calibrated a model that contradicts basic probability theory.

This chapter covers:

1. **The Curve Construction Problem** (Section 42.1): Why finite quotes cannot uniquely determine a continuous survival curve
2. **Desirable Curve Properties** (Section 42.2): What practitioners need from a calibration algorithm
3. **The Bootstrap Algorithm** (Section 42.3): Sequential solving from short to long maturities
4. **Interpolation Methods** (Section 42.4): Piecewise-constant hazard and its alternatives
5. **Consistency and Arbitrage Detection** (Section 42.5): When curves fail and what to do about it
6. **Sensitivity Propagation and Locality** (Section 42.6): The Jacobian structure that makes hedging simple
7. **Recovery Sensitivity and Rec01** (Section 42.7): How recovery assumptions affect implied hazards
8. **Risk-Neutral vs. Physical Probabilities** (Section 42.8): Why the bootstrapped curve is *not* a forecast
9. **The ISDA Standard Model** (Section 42.9): Industry standardization and its implications
10. **Practical Implementation** (Section 42.10): Data sources, troubleshooting, and common pitfalls

This chapter builds directly on the survival/hazard definitions (Chapter 36) and the CDS cashflow/PV setup (Chapter 41). The survival curve constructed here is then an input for CDS risk measures (Chapter 43) and credit relative value analysis (Chapter 44).

---

## 42.1 The Curve Construction Problem

### 42.1.1 The Analogy with Interest Rate Curves

A useful starting point is the analogy between interest-rate curve building and credit curve building:

| Interest Rates | Credit |
|----------------|--------|
| Discount curve $Z(t)$ | Survival curve $Q(t)$ |
| Short rate $r(s)$ | Hazard rate $\lambda(s)$ |
| No-arbitrage: $r(s) > 0$ | No-arbitrage: $\lambda(s) > 0$ |
| $Z(t) = \exp(-\int_0^t r(s) ds)$ | $Q(t) = \exp(-\int_0^t \lambda(s) ds)$ |
| Zero rate $z(t) = -(1/t)\ln Z(t)$ | Zero default rate $z(t) = -(1/t)\ln Q(t)$ |
| Forward rate $f(t) = -Z(t)^{-1} \partial Z(t)/\partial t$ | Forward default rate $h(t) = -Q(t)^{-1} \partial Q(t)/\partial t$ |

This parallel extends to curve construction methodology. In Chapter 17, we saw that bootstrapping interest rate curves involves solving sequentially for discount factors that reprice benchmark instruments. The same logic applies here: we solve sequentially for survival probabilities that reprice benchmark CDS contracts.

### 42.1.2 The Underdetermined Nature of the Problem

The fundamental difficulty is that we have finite constraints (CDS quotes at discrete maturities) but need a continuous function (survival probability at all times). A typical CDS curve might include quotes at 6M, 1Y, 2Y, 3Y, 4Y, 5Y, 7Y, and 10Y—eight data points.

> **Analogy: The Invisible Path**
>
> Imagine you are tracking a hiker (the Credit). You only see them at specific checkpoints: 1 mile (1Y), 3 miles (3Y), and 5 miles (5Y).
>
> *   **The Problem**: Where were they at mile 2? Or mile 4?
> *   **Bootstrapping**: We must assume a path.
> *   **Linear Hazard (Piecewise Constant)**: We assume they walked at a *constant speed* between checkpoints. This is the safest, most stable assumption.
> *   **Bad Interpolation**: Assuming their speed accelerated and decelerated wildly (Sawtooth) just to hit the checkpoints.

There are, in principle, infinitely many continuous survival curves that can match a finite set of quotes. The bootstrapping algorithm, combined with a specific interpolation rule, is therefore a *modeling choice* that determines how the curve behaves between skeleton points.

### 42.1.3 What We Need to Solve

From Chapter 41, the par condition for a new CDS at inception is:

$$\boxed{S_0 \cdot \text{RPV01}(0,T) = \text{Protection PV}(0,T)}$$

**Expand (two-legs-must-match):** the right-hand side is the present value of the protection payment (expected discounted loss, scaled by $1-R$). The left-hand side is the present value of the running premium stream: you pay a rate $S_0$ (per year) on surviving notional, and $RPV01$ packages the discounted, survival-weighted “annuity” of those premium payments (including the expected accrued-on-default term).

**Check (units + limiting cases):**
- Units: $S_0$ has units $1/\text{year}$ and $RPV01$ has units $\text{year}$ (per unit notional), so the product is a unitless *fraction of notional* (or currency per unit notional), matching the protection PV.
- If there is no default risk ($Q(0,t)\equiv 1$), then $\text{Protection PV}=0$ and the par spread is $S_0=0$.
- If recovery $R$ increases holding the survival curve fixed, protection PV falls; to re-match the equality you would need a lower $S_0$ (all else equal).

The premium leg PV is packaged via $RPV01(0,T)$, the premium-leg PV per unit running spread (per unit notional). It has two components: scheduled premium payments conditional on survival, and the expected premium accrued up to default between coupon dates:

$$
\text{RPV01}(0,T)
= \sum_{n=1}^{N} \Delta_n Z(0,t_n) Q(0,t_n)
\;+\; \sum_{n=1}^{N} \int_{t_{n-1}}^{t_n} \Delta(t_{n-1},s)\, Z(0,s)\,(-dQ(0,s)).
$$

The integral term can be approximated cheaply on the coupon grid. A common trapezoid-style approximation is:
$$
\int_{t_{n-1}}^{t_n} \Delta(t_{n-1},s)\, Z(0,s)\,(-dQ(0,s))
\approx
\frac{1}{2}\Delta_n Z(0,t_n)\left(Q(0,t_{n-1})-Q(0,t_n)\right),
$$
which yields the compact discrete approximation:
$$
\text{RPV01}(0,T) \approx \frac{1}{2} \sum_{n=1}^{N} \Delta_n Z(0,t_n) \left(Q(0,t_{n-1}) + Q(0,t_n)\right).
$$

For the protection leg, a standard expression (per unit notional) is:
$$
\text{Protection PV}(0,T) = (1-R)\int_0^T Z(0,s)\,(-dQ(0,s)),
$$
which can be discretized on a chosen grid (e.g., premium dates) as:
$$
\text{Protection PV}(0,T) \approx \frac{1-R}{2} \sum_{k=1}^{K} \left(Z(0,t_{k-1}) + Z(0,t_k)\right) \left(Q(0,t_{k-1}) - Q(0,t_k)\right).
$$

Both expressions depend on the survival curve $Q(0,t)$. Given market CDS spreads $\{S_m\}$ at maturities $\{T_m\}$, we must find a survival curve such that each CDS has zero mark-to-market at inception.

---

## 42.2 Desirable Curve Properties

Before specifying an algorithm, it helps to make the desired properties explicit. These requirements guide the choice of bootstrapping and interpolation methodology.

### 42.2.1 Exact Fit to Market Quotes

This is the fundamental calibration constraint: the curve should reprice the input CDS quotes to a tight tolerance. If it does not, downstream valuation and risk will reflect model error, not market information.

### 42.2.2 Sensible Interpolation

The gap between adjacent CDS quotes can be substantial—e.g., a few years between 7Y and 10Y maturities. An interpolation rule should behave sensibly between quotes. What counts as “sensible” is a modelling decision about how default intensity evolves between observable points.

### 42.2.3 Locality

This requirement is crucial for risk management: if you bump the 5Y quote and rebuild the curve, you would like the *local* part of the curve (near 5Y) to move, without large unintended changes elsewhere.

Locality helps hedges behave predictably. If moving the 5Y spread reshapes the whole curve, hedge ratios and P&L attribution become harder to explain and reconcile.

### 42.2.4 Speed

Credit desks often need to build many issuer-specific curves. The calibration algorithm must be fast enough to run routinely (and robustly) at scale.

### 42.2.5 Smoothness vs. Locality Trade-off

There is a tension between smoothness and locality: enforcing smooth derivatives tends to couple distant parts of the curve. In credit, it is common to prioritize locality and robustness over global smoothness.

This preference distinguishes credit curve construction from interest rate curve construction. In rates, the large number of liquid instruments at the short end justifies sophisticated spline methods. In credit, the sparsity of quotes and the need for speed favor simpler, more local interpolation.

---

## 42.3 The Bootstrap Algorithm

The bootstrap algorithm solves for survival probabilities sequentially, starting from the shortest maturity and working outward. Each step uses the previously determined curve segment to price the next instrument, solving for one new parameter (the survival probability at that maturity) to match the market quote.

### 42.3.1 Inputs

The algorithm requires:

1. **Market CDS quotes** $\{(T_m, S_m)\}_{m=1}^{M}$, sorted by increasing maturity
2. **Discount curve** $Z(0,t)$ at all needed times (treated as an input; must be consistent with your collateral/funding assumptions)
3. **Recovery assumption** $R$ (an input to the model; often assumed constant within a curve build)
4. **Premium schedule** $\{t_n\}$ and accrual fractions $\Delta_n$ (must match the CDS contract specification used by the quotes)
5. **Integration/discretization grid** for the protection leg (e.g., premium dates, or a finer grid if you are doing numerical integration)

### 42.3.2 The Algorithm Steps

The bootstrap algorithm is:

1. Initialize the survival curve with $Q(T_0=0)=1$.
2. Set $m=1$.
3. Solve for $Q(T_m)$ such that the mark-to-market of the $T_m$ CDS at the market spread $S_m$ is zero, using the previously determined curve segments to value cashflows up to $T_m$.
4. Enforce the no-arbitrage bound $0 < Q(T_m) \le Q(T_{m-1})$.
5. Add $(T_m, Q(T_m))$ to the curve, set $m \leftarrow m+1$, and repeat until all maturities are bootstrapped.

### 42.3.3 The Root-Finding Problem

Step 3 requires solving a one-dimensional equation: find $Q(T_m)$ such that the CDS mark-to-market is zero. In practice, this is done with a robust 1D root search (e.g., bisection, Newton-Raphson, Brent).

The objective function is the mark-to-market value as a function of $Q(T_m)$:

$$f(Q(T_m)) = \text{Protection PV}(0,T_m) - S_m \cdot \text{RPV01}(0,T_m) = 0$$

The bounds for the root search come from the no-arbitrage constraint: $0 < Q(T_m) \leq Q(T_{m-1})$.

**Expand (why the root is well-behaved):** earlier curve segments are fixed from prior bootstrap steps; changing $Q(T_m)$ only changes survival (and therefore cashflow PVs) on the new interval $(T_{m-1},T_m]$ under your interpolation rule. As you increase $Q(T_m)$ (more survival on the new interval), the protection PV *decreases* while $RPV01$ *increases*. So $f(Q(T_m))$ is typically **monotone decreasing**, which is why bracketing methods like bisection are so robust here.

**Check (bracketing logic):** in a “normal” quote set, $f(Q(T_{m-1}))$ is negative (too much premium PV relative to protection when you assume *no* incremental default past $T_{m-1}$) and $f(0^+)$ is positive (too much protection PV when you assume near-certain default by $T_m$). If you cannot get a sign change over $(0,Q(T_{m-1})]$, there is no admissible $Q(T_m)$ and the calibration failure is a genuine arbitrage/inconsistency signal (Section 42.5).

### 42.3.4 Extrapolation Beyond the Quotes

Outside the quoted range, you must adopt an extrapolation rule. A common choice is to assume a flat forward default rate before the first quote and beyond the last quote (but this is a modelling decision that should be stated explicitly, because it affects extrapolated PVs and risks).

This extrapolation assumption matters for pricing instruments with maturities outside the quoted range. A different extrapolation rule would imply different valuations for such instruments.

---

## 42.4 Interpolation Methods

The bootstrap algorithm requires an interpolation rule to determine survival probabilities between skeleton points. A standard approach is to restrict yourself to simple linear schemes (fast, local) and then choose *what* to interpolate.

### 42.4.1 The Preferred Method: Linear Interpolation of $-\ln Q(t)$

A widely used interpolation scheme linearly interpolates the negative log of the survival probability:

$$f(t) = -\ln Q(t)$$

This is equivalent to assuming a **piecewise-constant forward default rate** $h(t)$ between skeleton points. The derivation proceeds as follows.

Since $Q(t) = \exp(-\int_0^t h(s) ds)$, we have $f(t) = \int_0^t h(s) ds$. Linear interpolation of $f(t)$ on the interval $[t_{n-1}, t_n]$ gives:

$$f(t^*) = \frac{(t_n - t^*) f(t_{n-1}) + (t^* - t_{n-1}) f(t_n)}{t_n - t_{n-1}}$$

Differentiating with respect to $t^*$:

$$h(t^*) = \frac{\partial f(t^*)}{\partial t^*} = \frac{f(t_n) - f(t_{n-1})}{t_n - t_{n-1}}$$

Since this does not depend on $t^*$, the forward default rate is constant within each interval:

$$\boxed{h(t) = \frac{1}{T_n - T_{n-1}} \ln\left(\frac{Q(T_{n-1})}{Q(T_n)}\right), \quad t \in (T_{n-1}, T_n]}$$

The survival probability at any intermediate point is then:

$$\boxed{Q(t) = Q(T_{n-1}) \exp\left(-(t - T_{n-1}) h\right)}$$

**Check (hazard units + scale):** $h$ has units $1/\text{year}$. If survival drops from $Q(1)=0.98$ to $Q(2)=0.94$, then
$$
h=\ln\!\left(\frac{0.98}{0.94}\right)\approx 0.0417\ \text{year}^{-1},
$$
which corresponds to a conditional default probability over that year of about $1-e^{-h}\approx 4.1\%$.

**Why this method works:** No-arbitrage requires $h(t) \geq 0$, which is guaranteed as long as $Q(T_n) \leq Q(T_{n-1})$ at each skeleton point. The piecewise-constant structure ensures that if the skeleton is arbitrage-free, the interpolated curve is also arbitrage-free everywhere.

### 42.4.2 Rejected Alternative: Linear Interpolation of the Zero Default Rate

The zero default rate is defined as $z(t) = -\frac{1}{t}\ln Q(t)$, analogous to the zero rate for bonds. Linear interpolation of $z(t)$ might seem natural, but it can create undesirable kinks in the implied forward default rate.

The relationship between the forward default rate and zero default rate is:

$$h(t) = \frac{\partial}{\partial t}(z(t) \cdot t) = z(t) + t \cdot \frac{\partial z(t)}{\partial t}$$

When $z(t)$ is piecewise linear, its derivative is piecewise constant but jumps at each knot. This causes $h(t)$ to exhibit sudden jumps. More seriously, even if the skeleton points have $Q(t_n) \le Q(t_{n-1})$, it is not guaranteed that *all interpolated points* remain arbitrage-free.

### 42.4.3 Rejected Alternative: Linear Interpolation of the Forward Default Rate

This approach directly interpolates $h(t)$, making it piecewise linear. In practice it can be numerically unstable: the implied forward default rate can oscillate (“saw-tooth” behavior) and may even dip negative between knots, creating arbitrage.

### 42.4.4 Comparison Across Curve Shapes

When these schemes are tested on realistic curve shapes, the linear-in-$h(t)$ interpolation tends to be the most fragile: it can produce a violent saw-tooth forward default rate curve and may generate negative forwards.

For an upward-sloping curve and a steeply inverted curve (distressed credit), the piecewise-constant hazard method again produces stable results. The linear-in-$z(t)$ method can produce acceptable but jagged forward rates that may matter for forward-starting products.

In practice, the piecewise-constant hazard (equivalently: linear interpolation of $-\ln Q$) is widely used because it is local, stable, and easy to bootstrap.

---

## 42.5 Consistency and Arbitrage Detection

Not all CDS quote sets can be calibrated to a valid survival curve. When the input quotes imply arbitrage, the bootstrap fails—and understanding why is essential for production systems.

### 42.5.1 The No-Arbitrage Constraint

The fundamental constraint is that survival probabilities must be non-increasing:

$$\boxed{\frac{d\,Q(0,t)}{dt} \leq 0 \quad \text{for all } t > 0}$$

Equivalently, the forward default rate must be non-negative: $h(t) \geq 0$. If this constraint is violated, you are modeling a world where the credit becomes *more* likely to survive the longer you wait—an economic absurdity.

### 42.5.2 When Arbitrage Arises

Arbitrage shows up when extending maturity provides *additional protection* in the interval $[T_{m-1},T_m]$ without requiring enough additional premium. In the limit, the incremental protection PV over $[T_{m-1},T_m]$ is effectively “free,” which forces the implied forward default rate to zero or negative on that interval.

This happens when the spread curve is too steeply inverted. If the 3Y spread is much lower than the 1Y spread (because the market expects the credit to survive the near-term crisis), the protection for years 1-3 may have zero or negative implied cost.

### 42.5.3 Computing the Arbitrage Boundary

An easy-to-use approximation for this boundary is:

$$\boxed{S_m \gtrsim S_{m-1} \left(\frac{T_{m-1}}{T_m}\right)}$$

**Example:** For a steeply inverted curve starting at 800 bp at 6M:

The approximation above implies these **approximate** lower bounds:

| CDS Term | Approximate Lower Bound (bp) |
|----------|------------------------------|
| 6M | 800 |
| 1Y | 400 |
| 2Y | 200 |
| 3Y | 133 |
| 4Y | 100 |
| 5Y | 80 |
| 7Y | 57 |
| 10Y | 40 |

Treat this as a quick screen, not a hard rule: the **exact** boundary depends on the premium schedule, discount factors, and the premium-accrued-at-default treatment. In a bootstrap implementation you detect infeasibility when the root search for $Q(T_m)$ has no solution in $(0, Q(T_{m-1})]$.

### 42.5.4 Handling Calibration Failure

In production, you want the system to (i) detect the failure, (ii) explain *which* quotes caused it, and (iii) make the decision explicit if you choose to proceed with relaxed bounds (i.e., knowingly allowing a non-monotone $Q$ / negative hazards).

Production systems should:

1. **Detect** calibration failures (root search outside bounds)
2. **Report** the problematic quotes and the implied arbitrage
3. **Allow user decision**: relax bounds (accepting model arbitrage) or fix inputs (re-mark spreads, adjust recovery)

### 42.5.5 Production Troubleshooting: When Curves Fail to Build

> **Desk Reality: The Curve That Won't Build**
>
> It's 6:30 PM and the batch that marks 3,000 issuer curves is failing on 47 names. Your quant has gone home. What do you do?
>
> **Diagnostic workflow:**
>
> 1. **Identify the failure mode**: Is the root solver returning "no root in interval"? That's an arbitrage violation. Is it timing out? That's a convergence issue.
>
> 2. **Check quote staleness**: Pull quote timestamps. If the 1Y quote is from yesterday and the 5Y updated today, the combination may be inconsistent. Filter by time-of-last-trade.
>
> 3. **Check for crossed markets**: Sometimes bid/ask confusion leads to inverted spreads that don't make economic sense.
>
> 4. **Try recovery sensitivity**: If the name is distressed, changing the recovery input used in calibration can change feasibility. As a diagnostic, try an alternative recovery assumption and see whether the curve becomes buildable (and document the choice).
>
> 5. **Document and escalate**: If you can't fix it cleanly, mark the name as "curve build failed" with a documented reason. This flags it for trader review in the morning.

**Resolution options in practice:**

| Option | When to Use | Risk |
|--------|-------------|------|
| **Re-mark the quote** | Quote is clearly stale or erroneous | Requires judgment; audit trail needed |
| **Adjust recovery** | Distressed name; market recovery likely different | Changes all implied hazards; affects other names if systematic |
| **Filter by trade time** | Mix of fresh and stale quotes | May leave gaps in the curve |
| **Accept model arbitrage** | Name is genuinely stressed; need *some* curve | Documents limitation; requires risk sign-off |
| **Fall back to single-point curve** | Only one liquid tenor | Loses term structure; extrapolation-heavy |

---

## 42.6 Sensitivity Propagation and Locality: The Jacobian Structure

Understanding *why* the bootstrap produces local sensitivities requires examining the sensitivity matrix—the Jacobian that relates quote bumps to curve changes. This structure has profound implications for hedging.

### 42.6.1 The Sensitivity Matrix

When hedging a CDS book, we need to know how bumping each market quote affects each position. Consider hedging a CDS position $V$ using on-market CDS at standard tenors $\{S_{1Y}, S_{3Y}, S_{5Y}, S_{7Y}, S_{10Y}\}$.

The hedge notionals $\{N_i\}$ satisfy the system of linear equations:

$$-\frac{\partial V}{\partial S_i} = \sum_{j=1}^{M} N_j \frac{\partial V_j}{\partial S_i} \quad \text{for } i = 1, \ldots, M$$

The key insight is in the structure of the matrix $\frac{\partial V_j}{\partial S_i}$—the sensitivity of the $j$-th hedging instrument to a bump in the $i$-th market quote.

### 42.6.2 The Diagonal Structure

For on-market CDS under a local bootstrap, this matrix is **diagonal**:

$$\boxed{\frac{\partial V_j}{\partial S_i} = 0 \quad \text{for } i \neq j}$$

**Why?** Consider bumping the 10Y CDS spread while holding the 1Y, 3Y, 5Y, and 7Y spreads fixed. The 1Y on-market CDS has a spread equal to the market 1Y quote by definition. Since that quote is unchanged, the 1Y CDS still has zero mark-to-market—its sensitivity to the 10Y bump is zero.

The intuition is simple: if you bump the 10Y quote while holding the 1Y quote fixed, then the 1Y *par* CDS is still par by construction, so its MTM remains zero and its sensitivity to the 10Y bump is zero.

**Check (when you *won't* see diagonality):** if you fit the curve with a global smoother (splines) or shape constraints that couple knots, a bump to the 10Y quote can move earlier hazards/survivals even while you “hold” earlier quotes fixed. That produces non-zero off-diagonal entries. As a regression test, bump a long-tenor quote and confirm (i) earlier *par* CDS still reprice to ~0 and (ii) the bumped quote reprices exactly—if either fails, you are not doing the bump-and-rebuild you think you are.

For the diagonal elements:

$$\boxed{\frac{\partial V_i}{\partial S_i} = -\text{RPV01}(t, T_i)}$$

This is simply the negative of the risky PV01.

### 42.6.3 Example: The 5×5 Sensitivity Matrix

Example (illustrative numbers): a diagonal on-market sensitivity matrix looks like:

| Hedge | 1Y Bump | 3Y Bump | 5Y Bump | 7Y Bump | 10Y Bump |
|-------|---------|---------|---------|---------|----------|
| CDS 1Y | -0.979 | 0 | 0 | 0 | 0 |
| CDS 3Y | 0 | -2.775 | 0 | 0 | 0 |
| CDS 5Y | 0 | 0 | -4.353 | 0 | 0 |
| CDS 7Y | 0 | 0 | 0 | -5.738 | 0 |
| CDS 10Y | 0 | 0 | 0 | 0 | -7.463 |

The diagonal values are the risky PV01s at each maturity. The zeros confirm locality.

### 42.6.4 Implications for Hedging

The diagonal structure makes hedge computation trivial. Instead of inverting a full matrix, each hedge notional is simply:

$$N_i = \frac{-\partial V / \partial S_i}{\text{RPV01}(t, T_i)}$$

This is the "CDS equivalent notional"—how much of the $T_i$-maturity on-market CDS you need to neutralize your exposure to that spread bucket.

**Definition (bucket CS01):** for any position with present value $V$, define the tenor-$T_i$ CS01 as
$$
CS01_i \;:=\; V(\,S_i + 1\text{ bp}\,;\ \text{rebuild curve}\,) - V(\,S_i\,;\ \text{base}\,)
$$
where the **bump object** is the *market par CDS spread quote* $S_i$ (in decimal), the **bump size** is $1\text{ bp}=10^{-4}$, and the curve is rebuilt using the same bootstrap + interpolation rule. Units: currency per 1bp for the stated notional. Sign: for long protection, $CS01_i$ is typically positive (spreads widen $\Rightarrow$ protection becomes more valuable); for short protection it is typically negative.

> **Desk Reality: Why Locality Matters for P&L Attribution**
>
> When you run a credit book, you need to explain daily P&L. If bumping the 5Y spread affected the entire curve, your P&L attribution would show cross-exposures everywhere. You'd tell your manager: "I lost \$50k because 5Y widened, but also because that affected my 10Y positions through the curve..."
>
> With a local bootstrap, the story is simple: "5Y widened 10bp. My 5Y CS01 is \$5k/bp. I lost \$50k." Clean, auditable, explainable.
>
> Non-local curves create "unexplained" P&L that makes risk management harder and auditors nervous.

### 42.6.5 Why Off-Market Contracts Have Non-Zero Off-Diagonal Sensitivities

The diagonal structure holds perfectly only for on-market CDS. For a CDS with contractual spread different from the market spread, there are small off-diagonal sensitivities.

The reason: when you bump the 3Y spread while holding the 5Y spread fixed, the bootstrap must adjust the forward hazard rate between years 3-5 to maintain the 5Y calibration. This changes the risky PV01 of the 5Y position slightly—the premium leg value depends on survival probabilities in years 1-3, which have changed.

These effects are usually small, but they can matter for large off-market positions or for instruments whose PV depends on survival between quoted maturities.

---

## 42.7 Recovery Sensitivity and Rec01

The survival curve depends critically on the assumed recovery rate $R$. Recovery is not identified from single-name CDS spreads alone, so it is a *model input*: changing it changes the calibrated hazard rates and therefore PVs and risks. This is especially important for stressed names and off-market positions.

### 42.7.1 The Credit Triangle (Intuition, Not an Identity)

For continuously paid premium and a constant hazard rate (and ignoring some real-world details like discrete premium payments and discounting), a common approximation is:

$$\boxed{S \approx \lambda (1 - R)}$$

This is the "credit triangle." Given an observed spread, the implied hazard rate depends on the recovery assumption:

$$\lambda \approx \frac{S}{1 - R}$$

**Higher recovery → Higher implied hazard.** If recovery is 40%, a 200 bp spread implies $\lambda = 200/(1-0.4) = 333$ bp/year hazard. If recovery is 60%, the same 200 bp spread implies $\lambda = 200/(1-0.6) = 500$ bp/year hazard.

**Why?** Higher recovery means lower loss-given-default. To explain the same protection cost, the model requires more frequent defaults. The spread reflects expected loss; if each loss is smaller, defaults must be more frequent.

### 42.7.2 Rec01 (Recovery-Rate Sensitivity)

Recovery sensitivity is convention-dependent, so you must define **what is held fixed**.

For curve construction and curve-based risk, a common desk definition is:
- Hold the *market CDS quotes* $\{S_m\}$ fixed,
- bump the recovery input by **+1% absolute** (e.g., $40\% \rightarrow 41\%$),
- rebuild the survival curve with the same bootstrap + interpolation rule,
- revalue the position.

Define:
$$
Rec01 \;:=\; PV(R+1\%) - PV(R)
$$
Units: currency per 1% recovery (absolute). Sign depends on position direction and whether the instrument is on-market or off-market.

**Check (why on-market Rec01 can be small):** for an on-market CDS used in the calibration set, the rebuild forces PV $\approx 0$ at the market quote even after you change $R$ (hazards adjust to compensate). So its Rec01 is often near zero by design. Rec01 becomes more important for off-market contracts and for instruments whose value depends on the absolute hazard level, not just matching par spreads.

> **Desk Reality:** Recovery is an input, not an output
>
> Two systems can take the same spreads and discount curve but produce different hazard curves if they use different recovery inputs. This creates “model basis” in MTM and risk. Always document the recovery convention, and reconcile it across valuation, risk, and collateral systems.

---

## 42.8 Risk-Neutral vs. Physical Default Probabilities

The survival curve bootstrapped from CDS spreads is a **risk-neutral** object: it is the curve that makes discounted expected cashflows match market prices.

### 42.8.1 The Two Probability Measures (and Why They Differ)

The two measures serve different purposes:

- **Physical probability** ($P$): a best estimate of the actual likelihood of default, built from historical data and/or fundamentals. This is the object you want for forecasting and credit-loss modeling.
- **Risk-neutral probability** ($Q$): the probability implied by market prices. This is the object you want for pricing and mark-to-market.

These can differ materially. The wedge reflects compensation investors demand for bearing default risk (and other premia such as liquidity), i.e., the same forces that produce expected excess returns in credit markets.

> **Check (interpretation):** If a CDS-implied curve produces a higher default probability than your historical estimate, that is not automatically a “bad calibration.” It can simply mean the market is charging a large premium for bearing default risk. Use $Q$ for pricing; do not treat it as a forecast.

### 42.8.2 Practical Implications

The distinction matters for different applications:

| Application | Which Probability? | Why |
|-------------|-------------------|-----|
| CDS/derivative pricing | Risk-neutral | Ensures consistency with market prices |
| Mark-to-market | Risk-neutral | Reflects market's current pricing of risk |
| VaR / credit risk capital | Physical | Need actual default likelihood |
| Reserve calculations | Physical | Actuarial forecasting, not pricing |
| Fundamental analysis | Both | CDS spreads reflect risk appetite + credit quality |

**What can go wrong:** If you use the bootstrapped survival curve to predict actual defaults, you will systematically misinterpret a pricing object as a forecasting object.

---

## 42.9 The ISDA Standard Model

Following standardization to fixed-coupon + upfront quoting, market participants need to translate upfront quotations to spread quotations (and vice versa) in a standardized manner. ISDA created the **ISDA CDS Standard Model** and made the underlying source code for CDS calculations freely available (e.g., via `cdsmodel.com`), along with specifications that pin down conventions.

### 42.9.1 Why a Reference Model Exists

Small differences in conventions and numerical methods can create meaningful PV differences even when two parties start from the “same” market quotes. Examples include:
- schedule generation and accrual fractions
- treatment of premium accrued at default vs quoting accrued in a clean/full MTM
- curve interpolation choices and extrapolation rules
- numerical tolerances (root finding, integration grids)

A reference model + specs reduces this kind of “model basis” for standardized contracts and standard quote translations.

### 42.9.2 What the Package Provides (at a Minimum)

- Versioned source code for CDS calculations
- Published specifications for standard contracts and quote conversions (e.g., fixed-coupon + upfront conventions in index markets)
- Documentation for conventions, inputs, and outputs

### 42.9.3 Desk Reality: Reconciliation Is a Version-Control Problem

When your PV differs from someone else’s, the first questions are: which specification set, which model version, and which inputs (discount curve, recovery, schedule) were used? Record these alongside marks so discrepancies can be diagnosed quickly.

### 42.9.4 Practical Implications

1. If you must match standardized market marks, use the reference implementation (or a validated port) and record the exact version/date.
2. If you use a different internal model for research or risk, you still need a clear mapping to the standard conventions for reconciliation.
3. Recovery remains an input: even under the same model/specs, different recovery choices produce different curves and PVs.

---

## 42.10 Practical Implementation

### 42.10.1 Data Sources

CDS quotes are available from:

- **Market data vendors / aggregators**: Composite or contributed quotes (often non-executable) sourced from multiple dealers
- **Direct dealer runs**: Quotes from trading counterparties (may be executable or indicative depending on the channel)
- **Trading platforms / clearing workflows**: Where available, additional quote and contract-standardization data (varies by product)

Common tenors are 6M, 1Y, 2Y, 3Y, 4Y, 5Y, 7Y, and 10Y, though liquidity and quote quality are name-dependent and can be uneven across tenors.

### 42.10.2 Curve Rebuild Frequency

In production systems:

- **Daily**: Full rebuild for all active issuer curves at end-of-day marks
- **Intraday**: Selective rebuilds for names with significant spread moves or active trading
- **Real-time**: Trade pricing may trigger on-the-fly calibration

The bootstrap algorithm's speed enables frequent curve rebuilds at scale.

### 42.10.3 Common Data Issues

| Issue | Impact | Mitigation |
|-------|--------|------------|
| **Stale quotes** | Calibration to old information; inconsistent curves | Filter by quote age; flag staleness |
| **Missing tenors** | Gaps in curve; reliance on extrapolation | Use available tenors; document assumptions |
| **Crossed quotes** | Potential arbitrage; calibration failure | Verify bid/ask consistency; re-mark |
| **Inverted curves (distressed)** | May violate no-arbitrage; negative hazards | Check arbitrage bounds; adjust recovery or accept limitation |
| **Recovery mismatch** | Different recovery assumptions yield different hazards | Document recovery convention; reconcile across systems |

### 42.10.4 Day Count and Schedule Conventions

CDS curve building is convention-sensitive because the par spread is defined by setting the premium leg PV equal to the protection leg PV. Small differences in schedule, day count, or “accrued” conventions can move $RPV01$ enough to change implied hazards.

1. **Quarterly premium schedule + Actual/360 accrual.** The contractual spread on the premium leg is typically paid with a quarterly frequency. Accrual fractions $\Delta_n$ are computed using an Actual/360 convention.
2. **Standard dates (IMM quoting).** Standard CDS are quoted on the basis of IMM dates: a “5Y” quote refers to the contract maturing on the next IMM date, not necessarily exactly five years from today.
3. **Premium accrued on default (a cashflow).** $RPV01$ includes the expected premium accrued up to default between coupon dates (the integral term in the premium leg PV).
4. **Clean vs full MTM (a quotation convention).** Between coupon dates, desks may quote an unwind MTM “clean” (excluding spread accrual since the last coupon) or “full” (including it). One common definition is:
   $$
   \text{Clean MTM} = \text{Full MTM} - \text{Accrued},
   $$
   where the accrued running spread since the last coupon date is
   $$
   \text{Accrued} =
   \begin{cases}
   +\Delta(t_{n^{*}-1},t)\,S_0 & \text{(short protection)} \\
   -\Delta(t_{n^{*}-1},t)\,S_0 & \text{(long protection).}
   \end{cases}
   $$
   Here $t$ is the valuation date and $t_{n^{*}-1}$ is the most recent coupon date before $t$.

> **Pitfall — “Accrued” means two different things:** *premium accrued on default* (a contractual cashflow) vs *spread accrual since the last coupon date* (a clean/full MTM quotation convention).
> **Why it matters:** You can be directionally right on “credit widened” but still get the unwind cash amount wrong (and fail reconciliation) if you drop accrued-on-default in $RPV01$ or mix up clean vs full MTM.
> **Quick check:** Confirm you can reconcile $\text{Full MTM} = \text{Clean MTM} + \text{Accrued}$ using your sign convention for long/short protection.

**What happens if you get the day count/schedule wrong:** Using 30/360 instead of Actual/360 (or generating a schedule that is off by a day) changes accrual fractions $\Delta_n$ and payment dates $t_n$ in $RPV01$. That can produce different implied hazard rates for the same quoted spreads and can prevent clean repricing/reconciliation across systems.

---

## 42.11 Worked Examples

This section gives one worked bootstrap example in a simplified setting and then shows a few high-signal QA diagnostics (monotonicity and locality). Real implementations use quarterly schedules, exact day counts, and precise calendaring.

### 42.11.1 Worked Example: Bootstrapping a Toy 1Y/3Y/5Y Curve

**Example Title:** Bootstrapping a survival curve from three CDS quotes (toy annual schedule)

**Context**
- You need a survival curve $Q(0,t)$ that reprices a small set of market CDS spreads so you can value off-market CDS and compute bucketed CS01.

**Timeline (Concrete Dates; Toy Schedule)**
- Trade date: 2026-02-16
- Settlement / effective date: 2026-02-18
- Premium payment dates (toy annual): 2027-02-18, 2028-02-18, 2029-02-18, 2030-02-18, 2031-02-18
- Quote maturities: 1Y $\rightarrow$ 2027-02-18, 3Y $\rightarrow$ 2029-02-18, 5Y $\rightarrow$ 2031-02-18

> **Note (simplification):** Real single-name CDS uses quarterly payments on standard dates with Actual/360 accrual and premium accrued on default. We use annual payments and year-fractions $t=1,2,\ldots$ only to keep the arithmetic short; the bootstrap logic is unchanged.

**Inputs**
- Recovery: $R = 40\%$
- Discount factors (toy):

| $t$ (years) | $Z(0,t)$ |
|-------------|----------|
| 0 | 1.000 |
| 1 | 0.985 |
| 2 | 0.970 |
| 3 | 0.955 |
| 4 | 0.940 |
| 5 | 0.925 |

- Market par CDS spreads:

| Maturity | Spread (bp) | Spread (decimal) |
|----------|-------------|------------------|
| 1Y | 120 | 0.0120 |
| 3Y | 160 | 0.0160 |
| 5Y | 200 | 0.0200 |

**Outputs (What You Produce)**
- Piecewise-constant hazards (equivalently: linear interpolation of $-\ln Q$):
  - $h_1$ on $(0,1]$
  - $h_2$ on $(1,3]$
  - $h_3$ on $(3,5]$
- Knot survivals: $Q(1)$, $Q(3)$, $Q(5)$

**Step-by-step**

1. **Bootstrap the 1Y point.** Solve for $Q(1)$ such that the 1Y CDS is par.

   Using the trapezoid approximations from Section 42.1.3 with annual payments, the par condition can be written in terms of $x = S/(1-R)$:

   $$x = \frac{(Z_0 + Z_1)(Q_0 - Q_1)}{Z_1(Q_0 + Q_1)} \quad \text{with } Q_0=1,\, Z_0=1.$$

   Here $x = 0.012/0.60 = 0.02$. Solving gives:

   $$Q(1) \approx 0.98035,\qquad h_1 = -\ln Q(1) \approx 0.01985\ \text{year}^{-1}.$$

2. **Bootstrap the 3Y point.** Hold $Q(1)$ fixed and solve for a constant hazard $h_2$ on $(1,3]$ so the 3Y CDS reprices the 160 bp quote. Under piecewise-constant hazard:

   $$Q(2) = Q(1)e^{-h_2},\qquad Q(3) = Q(1)e^{-2h_2}.$$

   A 1D root search gives:

   $$h_2 \approx 0.02998\ \text{year}^{-1},\qquad Q(3) \approx 0.92328.$$

3. **Bootstrap the 5Y point.** Hold $Q(3)$ fixed and solve for a constant hazard $h_3$ on $(3,5]$ so the 5Y CDS reprices the 200 bp quote:

   $$Q(4) = Q(3)e^{-h_3},\qquad Q(5) = Q(3)e^{-2h_3}.$$

   A 1D root search gives:

   $$h_3 \approx 0.04434\ \text{year}^{-1},\qquad Q(5) \approx 0.84494.$$

4. **Repricing check.** Using these hazards and the same discretization, the model par spreads match the input quotes (within numerical tolerance) by construction.

**Cashflows (Per Unit Notional; Toy Annual Schedule)**

| Date | Cashflow | Explanation |
|---|---|---|
| each premium date $t_n$ | $-S \cdot \Delta_n$ if no default before $t_n$ | running premium payments |
| default date $\tau \in (t_{n-1}, t_n]$ | $+(1-R)$ | protection payment (simplified) |
| default date $\tau \in (t_{n-1}, t_n]$ | $-S \cdot \Delta(t_{n-1},\tau)$ | premium accrued on default (contractual) |

**P&L / Risk Interpretation**
- The hazards $\{h_1,h_2,h_3\}$ are the curve parameters implied by the spread quotes under your interpolation rule.
- If the 5Y quote widens and you rebuild the curve while holding 1Y and 3Y quotes fixed, the change should be concentrated in the last segment (locality). This makes CS01-based P&L attribution interpretable: “5Y widened $\Rightarrow$ 5Y bucket drove the move.”

**Sanity Checks**
- **Units check:** $(1-R)$ is unitless. Spread $S$ has units 1/year. $RPV01$ has units years (per unit notional). Therefore $S \cdot RPV01$ has units of “fraction of notional,” matching the protection leg PV per unit notional.
- **Sign check:** $h(t)\ge 0 \Rightarrow Q(0,t)$ must be non-increasing. Here $Q(1)>Q(3)>Q(5)$.
- **Limit check:** If all spreads were $0$, the solution would be $h(t)=0$ and $Q(0,t)=1$.
- **Repricing check:** Plug the resulting curve back into the par-spread formula and confirm you recover 120/160/200 bp.

**Debug Checklist (When Your Curve Looks Wrong)**
- Schedule generation: payment dates, accrual start/end, and maturity dates (standard-date vs exact-year maturity).
- Accrued premium: included in $RPV01$ consistently and not double-counted with clean/full MTM quoting.
- Day count: $\Delta_n$ computed on the contract’s day count (often Actual/360).
- Bounds and root solver: enforce $0<Q(T_m)\le Q(T_{m-1})$; verify the objective changes sign in the bracket.
- Discount factors: interpolated consistently; $Z(0,0)=1$; no accidental “percent vs decimal” scaling.

### 42.11.2 QA Diagnostics: Monotonicity and Locality (Same Toy Curve)

**Monotonicity check** (quarter-year nodes, computed from the piecewise-constant hazards):

| $t$ | $Q(0,t)$ |
|-----|----------|
| 0.00 | 1.00000 |
| 0.25 | 0.99505 |
| 0.50 | 0.99012 |
| 0.75 | 0.98522 |
| 1.00 | 0.98035 |
| 1.25 | 0.97302 |
| 1.50 | 0.96576 |
| 1.75 | 0.95855 |
| 2.00 | 0.95139 |
| 2.25 | 0.94428 |
| 2.50 | 0.93723 |
| 2.75 | 0.93023 |
| 3.00 | 0.92328 |
| 3.25 | 0.91311 |
| 3.50 | 0.90304 |
| 3.75 | 0.89309 |
| 4.00 | 0.88324 |
| 4.25 | 0.87351 |
| 4.50 | 0.86388 |
| 4.75 | 0.85435 |
| 5.00 | 0.84494 |

**Locality test:** keep 1Y=120 bp and 3Y=160 bp fixed, but bump 5Y from 200 bp to 201 bp. Under the sequential bootstrap, only the final segment changes:

| Segment | Old hazard | New hazard (5Y + 1bp) |
|---------|------------|------------------------|
| 0–1Y | 0.01985 | 0.01985 |
| 1–3Y | 0.02998 | 0.02998 |
| 3–5Y | 0.04434 | 0.04479 |

This is the practical meaning of locality: quote bumps tend to map into the “new” segment introduced at that tenor.

### 42.11.3 Quick Arbitrage Screen for an Inverted Curve (Approximate)

Consider a distressed/inverted term structure:

| Maturity | Spread (bp) |
|----------|-------------|
| 1Y | 500 |
| 3Y | 100 |
| 5Y | 120 |

Using the arbitrage screen $S_m \gtrsim S_{m-1}(T_{m-1}/T_m)$, the approximate lower bound for 3Y given 1Y=500 bp is:

$$S_{3Y}^{\min} \approx 500 \times \frac{1}{3} \approx 167\ \text{bp}.$$

The 100 bp quote is below this screen, so a bootstrap that enforces non-negative hazards will typically fail (it would require negative hazard on some interval).

### 42.11.4 Recovery Sensitivity (Same Quotes, Different Recovery)

Same 1Y/3Y/5Y quotes, different recovery assumptions (rebuilding the curve each time):

| Recovery $R$ | $h_1$ (0–1Y) | $h_2$ (1–3Y) | $h_3$ (3–5Y) |
|--------------|--------------|--------------|--------------|
| 20% | 0.01489 | 0.02246 | 0.03307 |
| 40% | 0.01985 | 0.02998 | 0.04434 |
| 60% | 0.02978 | 0.04509 | 0.06729 |

**Interpretation:** Higher recovery means lower loss-given-default. To explain the same spread, the model requires higher default intensity. The “credit triangle” $S \approx \lambda(1-R)$ is a useful intuition lever here, but remember it is an approximation.

### 42.11.5 Day-Count Mismatch (Toy)

Suppose we incorrectly use 30/360 (year fraction = 0.25 per quarter) instead of Actual/360 (year fraction varies by actual days).

For a 91-day quarter:
- 30/360: $\Delta = 0.25$
- Actual/360: $\Delta = 91/360 = 0.2528$

This 1% difference in year fraction flows through to the RPV01 calculation, changing the implied hazard rate. Over multiple years, the cumulative effect can be significant—particularly for back-testing where small discrepancies compound.

**Lesson:** Matching the exact market convention for day count is essential for correct calibration.

---

## 42.12 Verification and Quality Checks

### 42.12.1 Mandatory Consistency Checks

Every bootstrapped curve should pass these tests:

| Check | Expected | Action if Failed |
|-------|----------|------------------|
| $Q(0) = 1$ | Exact | Implementation error |
| $Q(t) \in [0, 1]$ | Always | Numerical error or bad data |
| $Q(t)$ non-increasing | Always | Arbitrage violation; investigate quotes |
| $h(t) \geq 0$ | Always | Equivalent to above |
| Repricing error | small (within numerical tolerance) | Calibration tolerance too loose or inputs inconsistent |

### 42.12.2 Stability and Locality Tests

When bumping a single quote:

- **Locality:** Only nearby hazard rates should change materially
- **Stability:** Small quote bumps should produce small curve changes
- **No sign flips:** Hazard rates should not become negative from small perturbations

### 42.12.3 Regression Testing

For production systems:
- Same inputs must produce identical outputs (deterministic build)
- Historical quote sets should reproduce historical curves exactly
- Parallel builds (different hardware/precision) should match within tolerance

---

## 42.13 Connection to Interest Rate Curve Construction

The parallels between CDS survival curve bootstrapping and interest rate curve bootstrapping (Chapter 17) are instructive:

| Aspect | Interest Rates | CDS Credit |
|--------|----------------|------------|
| Primary object | Discount factor $Z(t)$ | Survival probability $Q(t)$ |
| Benchmark instruments | Deposits, futures, swaps | CDS at standard tenors |
| Calibration condition | Reprice benchmarks exactly | Reprice CDS spreads exactly |
| Preferred interpolation | Often piecewise flat forwards | Piecewise constant hazard |
| Key constraint | $r(t) > 0$ (non-negative rates)* | $h(t) \geq 0$ (non-negative hazard) |
| Speed requirement | Moderate (few curves) | High (thousands of issuers) |
| Locality preference | Moderate | Strong |

*Note: In the era of negative interest rates, the rates constraint has been relaxed in practice.

Both problems share the fundamental challenge of extracting a continuous curve from discrete quotes. Both use sequential bootstrapping with interpolation between nodes. The credit problem is simpler in some respects (fewer liquid instruments, simpler interpolation preferred) but more demanding in others (many more curves to build, stronger locality requirements).

---

## Summary

Bootstrapping a CDS survival curve transforms sparse market quotes into a continuous term structure of survival probabilities. The algorithm:

1. **Sequentially solves** for survival probabilities at quoted maturities
2. **Uses piecewise-constant hazard interpolation** between skeleton points
3. **Enforces no-arbitrage bounds** on each root-finding step
4. **Detects and reports** calibration failures due to inconsistent quotes

The resulting curve is a **risk-neutral** object—it reflects market prices, not physical default probabilities. The bootstrapped hazard rates embed both expected default loss and risk premiums demanded by protection sellers.

**Key implementation requirements:**
- Exact match to market conventions (day counts, schedules, accrued-at-default)
- Consistent inputs across systems (discount factors, recovery, schedule generation, and “clean vs full” conventions)
- Robust handling of stale or inconsistent quotes
- Efficient computation for large numbers of issuer curves

**Key risk concepts:**
- **Locality**: Quote bumps affect only nearby maturities, making hedges predictable
- **Recovery sensitivity**: Recovery is an input; changing $R$ changes fitted hazards, PVs, and risks
- **Risk-neutral vs physical**: CDS-implied curves are for pricing/MTM, not default forecasting

The survival curve is the foundation for CDS mark-to-market, risk measures (Chapter 43), and credit relative value analysis (Chapter 44).

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Survival curve** | $Q(0,t) = \Pr(\tau > t)$ at all times $t$ | Foundation for all CDS valuation |
| **Piecewise-constant hazard** | $h(t)$ constant between skeleton points | Stable interpolation; preserves no-arbitrage |
| **Par CDS spread** | Running spread $S$ that makes CDS MTM zero at inception | The quote you bootstrap to |
| **RPV01** | Premium-leg PV per unit spread (includes accrued-on-default term under the chosen approximation) | Translates quotes into PV and CS01 |
| **Protection PV** | Expected discounted default payment $(1-R)$ under the survival curve | The other leg of the par condition |
| **Bootstrap algorithm** | Sequential solve from short to long maturity | Fast, local, exact repricing |
| **No-arbitrage bound** | $0 < Q(T_m) \leq Q(T_{m-1})$ | Ensures non-negative hazard rates |
| **Arbitrage boundary** | $S_m \gtrsim S_{m-1}(T_{m-1}/T_m)$ | Detects impossible quote combinations |
| **Diagonal Jacobian** | $\partial V_j/\partial S_i = 0$ for $i \neq j$ | Locality; simple hedge computation |
| **Clean vs full MTM** | $\text{Clean MTM}=\text{Full MTM}-\text{Accrued}$ (with sign depending on long/short protection) | Reconciliation and unwind cash amounts |
| **Rec01** | MTM change per 1% recovery shift | Critical for distressed names |
| **Risk-neutral vs. physical** | Bootstrapped curve embeds risk premium | Not a default forecast |
| **ISDA CDS Standard Model** | Reference implementation + specs for standardized CDS analytics and quote conversions | Reduces convention/model basis in reconciliation |

---

## Notation

| Symbol | Meaning | Units / Convention |
|--------|---------|--------------------|
| $Q(0,t)$ | Survival probability to time $t$ | unitless; $Q(0,0)=1$; non-increasing in $t$ |
| $h(t)$ | Forward default rate / hazard | 1/year; $\ge 0$; piecewise-constant between knots in this chapter |
| $Z(0,t)$ | Discount factor to time $t$ | unitless; treated as an input |
| $R$ | Recovery rate | unitless in $[0,1]$; assumed constant within a curve build |
| $S_m$ | Market par CDS spread at maturity $T_m$ | decimal per year; $1\text{ bp}=10^{-4}$ |
| $RPV01(0,T)$ | Premium-leg annuity (risky PV01) per unit notional | years; includes accrued-on-default term under the chosen approximation |
| $\Delta_n$ | Accrual year fraction for period $n$ | years; must match contract day count (often Actual/360) |
| $CS01_i$ | Bucketed CS01 at tenor $T_i$ | currency per 1bp for stated notional; bump quote $S_i$ and rebuild curve |
| $Rec01$ | Recovery sensitivity | currency per 1% recovery (absolute) for stated notional; bump $R$ and rebuild curve |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the goal of CDS survival curve bootstrapping? | Construct a full term structure of survival probabilities from finite CDS market quotes |
| 2 | What is the preferred interpolation method? | Linear interpolation of $-\ln Q(t)$, equivalent to piecewise-constant hazard |
| 3 | What no-arbitrage bound must each bootstrap step enforce? | $0 < Q(T_m) \leq Q(T_{m-1})$, ensuring non-negative hazard |
| 4 | What happens if the 3Y spread is too low relative to the 1Y spread? | Bootstrap fails—no non-negative hazard can fit both quotes |
| 5 | Give the approximate arbitrage boundary formula | $S_m \gtrsim S_{m-1}(T_{m-1}/T_m)$ |
| 6 | Why is locality a desirable curve property? | Ensures quote bumps affect only nearby maturities, making hedges predictable |
| 7 | What numerical methods solve each bootstrap step? | Bisection, Newton-Raphson, or Brent's method |
| 8 | How does recovery assumption affect bootstrapped hazards? | Higher recovery requires higher hazard to explain same spread |
| 9 | Is the bootstrapped survival curve a default forecast? | No—it is risk-neutral and embeds risk premiums |
| 10 | What is the standard day count for USD CDS? | Actual/360 |
| 11 | Why can linear interpolation of $h(t)$ produce arbitrage? | It creates oscillating "saw-tooth" patterns that can go negative |
| 12 | What is the relationship $Q(t) = Q(T_{n-1})\exp(-(t-T_{n-1})h)$? | Survival probability under piecewise-constant hazard |
| 13 | What distinguishes credit curve construction from rates? | Many more issuer curves to build; stronger locality requirements |
| 14 | What should you check after bootstrapping? | Repricing error, monotonicity of $Q$, positivity of $h$ |
| 15 | How is the forward default rate computed from skeleton points? | $h = \frac{1}{T_n - T_{n-1}}\ln(Q(T_{n-1})/Q(T_n))$ |
| 16 | What extrapolation assumption is common beyond the last quoted maturity? | Assume a flat forward default rate beyond the last knot (state it explicitly; it affects extrapolated PV/risk) |
| 17 | Why does using wrong day count cause problems? | Changes year fractions in RPV01, producing incorrect implied hazards |
| 18 | What is the "credit triangle" relationship? | $S \approx \lambda(1-R)$ for continuous premiums and constant hazard |
| 19 | What is Rec01? | Change in MTM for a 1% change in recovery rate |
| 20 | When does recovery sensitivity become most important? | When spreads are wide and/or positions are off-market: PV depends more strongly on the recovery input, so you must define bump-and-rebuild (Rec01) precisely |
| 21 | What does the diagonal Jacobian structure imply for hedging? | Hedge notionals are simple: $N_i = (-\partial V/\partial S_i)/\text{RPV01}(T_i)$ |
| 22 | What is the ISDA CDS Standard Model? | A reference implementation and specifications used to standardize common CDS analytics and quote conversions (versioning matters for reconciliation) |
| 23 | How can risk-neutral default probabilities differ from physical? | They can differ materially because credit spreads embed risk premia; use risk-neutral for pricing and physical for forecasting/loss modeling |
| 24 | For derivatives pricing, should you use risk-neutral or physical default probabilities? | Risk-neutral—they ensure consistency with market prices |
| 25 | Why do CDS-implied hazard rates overstate actual default frequency? | They embed risk premium and liquidity premium in addition to expected loss |

---

## Mini Problem Set

1. (Compute) Convert 250 bp to decimal spread per year.
2. (Compute) If $R = 35\%$ and $S = 200$ bp, compute $x = S/(1-R)$.
3. (Compute) Given $Q(1) = 0.97$ and $Q(3) = 0.91$, compute the constant hazard rate on $(1, 3]$.
4. (Compute) Given $Q(3) = 0.91$ and $h = 0.04$ on $(3, 5]$, compute $Q(5)$.
5. (Sanity) A bootstrapped curve shows $Q(3) = 0.95$ and $Q(5) = 0.97$. What is wrong and what does it imply about $h(t)$?
6. (Concept) State the no-arbitrage constraint for survival probabilities and its hazard-rate equivalent.
7. (Compute) Using $S_m \gtrsim S_{m-1}(T_{m-1}/T_m)$, if a 6M spread is 1000 bp, estimate the approximate lower bound for the 1Y spread.
8. (Methodology) Explain why a local bootstrap leads to (approximately) diagonal sensitivities for on-market CDS.
9. (Desk) Two systems disagree on CDS MTM by \$25k. List four inputs/conventions you would check first.
10. (Compute) Rec01: A \$20mm short-protection position has recovery sensitivity 0.05 (per 1% recovery). Compute Rec01 in \$ per 1% recovery change.
11. (Concept) Explain risk-neutral vs physical default probabilities and which one to use for (i) pricing/MTM and (ii) forecasting/loss modeling.
12. (Implementation) Design a locality regression test: bump the 5Y quote by 1 bp and state what should and should not change in a piecewise-constant-hazard bootstrap.

### Solution Sketches (Selected)

**1.** $250 \text{ bp} = 250 \times 10^{-4} = 0.025$.

**3.** $h = \frac{1}{3-1}\\ln(0.97/0.91) \approx 0.0320$ year$^{-1}$.

**7.** $S_{1Y}^{\\min} \approx 1000 \times (0.5/1.0)=500$ bp.

**10.** $Rec01 = 0.05 \times 1\% \times \\$20\\text{mm} = \\$10{,}000$ per 1% recovery.

**9.** Check (i) discount factors/curve version, (ii) recovery input, (iii) schedule + day count + standard-date conventions, and (iv) clean vs full MTM / accrued conventions (including accrued-on-default treatment).

---

## References

- O'Kane, *Modelling Single-name and Multi-name Credit Derivatives* (premium/protection legs, $RPV01$, bootstrapping algorithm + bounds, interpolation of $-\ln Q$, arbitrage screens, clean vs full MTM, standard-date quoting)
- Hull, *Options, Futures, and Other Derivatives* (risk-neutral vs physical probabilities; CDS marking-to-market intuition)
- Hull, *Risk Management and Financial Institutions* (“Real-World vs. Risk-Neutral Probabilities” and why they differ)
- McNeil, Frey, and Embrechts, *Quantitative Risk Management* (risk-neutral pricing vs real-world forecasting; reduced-form credit calibration context)
- Neftci, *Principles of Financial Engineering* (CDS standardization; fixed-coupon + upfront conventions; motivation for the ISDA CDS Standard Model and public code availability)
- Crépey, *Counterparty Risk and Funding* (piecewise-constant intensity calibration and CDS bootstrapping framing)
- Brigo, Morini, and Pallavicini, *Counterparty Credit Risk, Collateral and Funding* (intensity-based survival models and calibration to CDS)
- ISDA, *ISDA CDS Standard Model* (documentation and reference implementation; versioning and standardized quote conversions)
