# Chapter 42: Bootstrapping a CDS Survival Curve from Market Quotes

---

## Introduction

Before you can price a CDS, value a credit derivative, or compute the risk of a protection position, you need one fundamental object: the **survival curve**. This curve—the term structure of survival probabilities $Q(0,t)$—is to credit markets what the discount curve is to interest rate markets. Without it, every credit valuation is speculation.

But here is the challenge: the survival curve is not directly observable. The market quotes CDS spreads at a handful of maturities—perhaps 6M, 1Y, 3Y, 5Y, 7Y, and 10Y. From these sparse data points, you must infer a continuous function that tells you the survival probability to *any* date, not just the benchmark maturities. This is an **inverse problem**, and solving it requires a systematic algorithm: **bootstrapping**.

O'Kane frames the objective precisely: "Building the survival curve is essentially a process of constructing a full term structure of survival probabilities from a finite number of CDS market quotes." The process mirrors the interest rate curve construction covered in Chapter 17, but with credit-specific nuances. Just as the short rate drives discount factor dynamics, the hazard rate drives survival probability dynamics. Just as we bootstrap discount factors from swap rates, we bootstrap survival probabilities from CDS spreads.

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

This chapter builds directly on Chapter 36 (which defined survival probabilities and hazard rates) and Chapter 41 (which derived the CDS pricing formulas). If you understand those foundations, the bootstrapping algorithm becomes a natural extension. The survival curve we construct here is the input for CDS risk measures (Chapter 43) and credit relative value analysis (Chapter 44).

---

## 42.1 The Curve Construction Problem

### 42.1.1 The Analogy with Interest Rate Curves

O'Kane draws an explicit parallel between interest rate and credit curve construction that illuminates the bootstrapping approach:

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

O'Kane emphasizes: "There are in theory an infinite number of ways of doing this. The method we choose must be selected according to what we believe are the desired properties." The bootstrapping algorithm, combined with a specific interpolation rule, is a *modeling choice* that determines how the curve behaves between skeleton points.

### 42.1.3 What We Need to Solve

From Chapter 41, the par condition for a new CDS at inception is:

$$\boxed{S_0 \cdot \text{RPV01}(0,T) = \text{Protection PV}(0,T)}$$

where the risky PV01 (including accrued premium at default) is approximated as:

$$\text{RPV01}(0,T) = \frac{1}{2} \sum_{n=1}^{N} \Delta_n Z(0,t_n) \left(Q(0,t_{n-1}) + Q(0,t_n)\right)$$

and the protection leg PV is:

$$\text{Protection PV}(0,T) \approx \frac{1-R}{2} \sum_{k=1}^{K} \left(Z(0,t_{k-1}) + Z(0,t_k)\right) \left(Q(0,t_{k-1}) - Q(0,t_k)\right)$$

Both expressions depend on the survival curve $Q(0,t)$. Given market CDS spreads $\{S_m\}$ at maturities $\{T_m\}$, we must find a survival curve such that each CDS has zero mark-to-market at inception.

---

## 42.2 Desirable Curve Properties

Before specifying an algorithm, O'Kane identifies what properties a good survival curve construction should possess. These requirements guide the choice of bootstrapping and interpolation methodology.

### 42.2.1 Exact Fit to Market Quotes

O'Kane's first requirement is precision: "We want the methodology to provide an exact fit to the CDS market quotes provided. Since we require a minimum PV accuracy of $O(10^{-7})$, we define exact to mean an error of $O(10^{-4})$ basis points or less in the spread."

This is the fundamental calibration constraint. A curve that cannot reprice the input instruments is useless for mark-to-market purposes—any deviation represents model error, not market information.

### 42.2.2 Sensible Interpolation

The gap between adjacent CDS quotes can be substantial—up to 3 years between 7Y and 10Y maturities in a standard curve. O'Kane requires that "the method should interpolate between the market quotes in a sensible manner." What constitutes "sensible" depends on economic judgment about how default intensity evolves between observable points.

### 42.2.3 Locality

This requirement is crucial for risk management: "If we bump the 5Y CDS spread and rebuild the CDS curve, we would prefer a method which only changes the spread of CDS with a maturity close to 5Y. We want to avoid methods which cause the bump to 'leak' into more distant curve points."

Locality ensures that hedges behave predictably. If moving the 5Y spread affects the entire curve, your 10Y hedge ratio becomes coupled to your 5Y position in ways that are difficult to understand and explain.

### 42.2.4 Speed

O'Kane emphasizes computational efficiency: "A typical CDS book may have $O(10^4)$ CDS positions linked to $O(10^3)$ different issuer curves." Unlike interest rate curves (where perhaps four major currency curves cover most of the book), credit desks must build thousands of issuer-specific survival curves daily. The algorithm must be fast.

### 42.2.5 Smoothness vs. Locality Trade-off

O'Kane notes a tension: "We would like the curve to be smooth. However, there is usually a conflict between localness and smoothness since smoothness necessarily links together different parts of the curve via their derivatives. We prefer localness to smoothness."

This preference distinguishes credit curve construction from interest rate curve construction. In rates, the large number of liquid instruments at the short end justifies sophisticated spline methods. In credit, the sparsity of quotes and the need for speed favor simpler, more local interpolation.

---

## 42.3 The Bootstrap Algorithm

The bootstrap algorithm solves for survival probabilities sequentially, starting from the shortest maturity and working outward. Each step uses the previously determined curve segment to price the next instrument, solving for one new parameter (the survival probability at that maturity) to match the market quote.

### 42.3.1 Inputs

The algorithm requires:

1. **Market CDS quotes** $\{(T_m, S_m)\}_{m=1}^{M}$, sorted by increasing maturity
2. **Discount curve** $Z(0,t)$ at all needed times (typically OIS discounting for collateralized CDS)
3. **Recovery assumption** $R$, assumed constant across maturities (the market standard is 40% for investment-grade senior unsecured debt)
4. **Premium schedule** $\{t_n\}$ and accrual fractions $\Delta_n$ (typically quarterly, Actual/360)
5. **Integration grid** for protection leg discretization (can be aligned with premium dates or finer)

### 42.3.2 The Algorithm Steps

O'Kane describes the bootstrap procedure:

> **Step 1:** Initialize the survival curve starting with $Q(T_0 = 0) = 1.0$.
>
> **Step 2:** Set $m = 1$.
>
> **Step 3:** Solve for the value of $Q(T_m)$ for which the mark-to-market value of the $T_m$ maturity CDS with market spread $S_m$ is zero. The no-arbitrage bound on $Q(T_m)$ is $0 < Q(T_m) \leq Q(T_{m-1})$. All discount factors required to determine the CDS mark-to-market will be interpolated from the values already determined, plus $Q(T_m)$ which is the value we are solving for.
>
> **Step 4:** Once we have found a value of $Q(T_m)$ which reprices the CDS with maturity $T_m$, we add this time and value to our survival curve.
>
> **Step 5:** Set $m = m + 1$. If $m \leq M$ return to Step 3.
>
> **Step 6:** We end up with a survival curve consisting of $M + 1$ points with times at $0, T_1, T_2, \ldots, T_M$ and values $1.0, Q(T_1), Q(T_2), \ldots, Q(T_M)$.

### 42.3.3 The Root-Finding Problem

Step 3 requires solving a one-dimensional equation: find $Q(T_m)$ such that the CDS mark-to-market is zero. O'Kane notes that "solving for the next maturity survival probability requires the use of a one-dimensional root search algorithm. There are a number to choose from including bisection, Newton-Raphson, and Brent's method."

The objective function is the mark-to-market value as a function of $Q(T_m)$:

$$f(Q(T_m)) = \text{Protection PV}(0,T_m) - S_m \cdot \text{RPV01}(0,T_m) = 0$$

The bounds for the root search come from the no-arbitrage constraint: $0 < Q(T_m) \leq Q(T_{m-1})$.

### 42.3.4 Extrapolation Beyond the Quotes

O'Kane addresses what happens outside the quoted range: "Before the first maturity and beyond the last maturity CDS, we assume that the forward default rate is flat at a level of $h(0)$ before the first maturity. Beyond the last time point $T_M$ we assume that the forward default rate is flat at its last interpolated value."

This extrapolation assumption matters for pricing instruments with maturities outside the quoted range. A different extrapolation rule would imply different valuations for such instruments.

---

## 42.4 Interpolation Methods

The bootstrap algorithm requires an interpolation rule to determine survival probabilities between skeleton points. O'Kane evaluates three candidates for the quantity to be linearly interpolated, selecting one as clearly superior.

### 42.4.1 The Preferred Method: Linear Interpolation of $-\ln Q(t)$

The recommended interpolation linearly interpolates the negative log of the survival probability:

$$f(t) = -\ln Q(t)$$

O'Kane demonstrates that this is equivalent to assuming a **piecewise-constant forward default rate** $h(t)$ between skeleton points. The derivation proceeds as follows.

Since $Q(t) = \exp(-\int_0^t h(s) ds)$, we have $f(t) = \int_0^t h(s) ds$. Linear interpolation of $f(t)$ on the interval $[t_{n-1}, t_n]$ gives:

$$f(t^*) = \frac{(t_n - t^*) f(t_{n-1}) + (t^* - t_{n-1}) f(t_n)}{t_n - t_{n-1}}$$

Differentiating with respect to $t^*$:

$$h(t^*) = \frac{\partial f(t^*)}{\partial t^*} = \frac{f(t_n) - f(t_{n-1})}{t_n - t_{n-1}}$$

Since this does not depend on $t^*$, the forward default rate is constant within each interval:

$$\boxed{h(t) = \frac{1}{T_n - T_{n-1}} \ln\left(\frac{Q(T_{n-1})}{Q(T_n)}\right), \quad t \in (T_{n-1}, T_n]}$$

The survival probability at any intermediate point is then:

$$\boxed{Q(t) = Q(T_{n-1}) \exp\left(-(t - T_{n-1}) h\right)}$$

**Why this method works:** No-arbitrage requires $h(t) \geq 0$, which is guaranteed as long as $Q(T_n) \leq Q(T_{n-1})$ at each skeleton point. The piecewise-constant structure ensures that if the skeleton is arbitrage-free, the interpolated curve is also arbitrage-free everywhere.

### 42.4.2 Rejected Alternative: Linear Interpolation of the Zero Default Rate

The zero default rate is defined as $z(t) = -\frac{1}{t}\ln Q(t)$, analogous to the zero rate for bonds. Linear interpolation of $z(t)$ might seem natural, but O'Kane rejects it.

The relationship between the forward default rate and zero default rate is:

$$h(t) = \frac{\partial}{\partial t}(z(t) \cdot t) = z(t) + t \cdot \frac{\partial z(t)}{\partial t}$$

When $z(t)$ is piecewise linear, its derivative is piecewise constant but jumps at each knot. This causes $h(t)$ to exhibit "sudden jumps in the shape" that are economically unintuitive. More seriously, O'Kane notes that "if $Q(t_n) \leq Q(t_{n-1})$, it is not guaranteed that all points which have been interpolated using this algorithm will also be arbitrage free."

### 42.4.3 Rejected Alternative: Linear Interpolation of the Forward Default Rate

This approach directly interpolates $h(t)$, making it piecewise linear. O'Kane dismisses it bluntly: "It is well known that there are stability problems with this interpolation method in the sense that the resulting forward default rate curve tends to oscillate with a 'saw-tooth' pattern." The oscillations can produce negative forward default rates—arbitrage violations.

### 42.4.4 Comparison Across Curve Shapes

O'Kane tests all three methods on realistic CDS curves and reports the results. For a flat curve with a single step at 4Y, the piecewise-constant hazard method produces stable, economically sensible forward rates. The linear-in-$h(t)$ method produces "violent sawtooth shape" and even generates negative forward default rates.

For an upward-sloping curve and a steeply inverted curve (distressed credit), the piecewise-constant hazard method again produces stable results. The linear-in-$z(t)$ method produces acceptable but "jagged" forward rates that could affect the pricing of forward-starting products.

O'Kane concludes: "These results confirm that the interpolation scheme which is piecewise constant in $h(t)$ is clearly the most stable and is the preferred of the three schemes considered."

---

## 42.5 Consistency and Arbitrage Detection

Not all CDS quote sets can be calibrated to a valid survival curve. When the input quotes imply arbitrage, the bootstrap fails—and understanding why is essential for production systems.

### 42.5.1 The No-Arbitrage Constraint

The fundamental constraint is that survival probabilities must be non-increasing:

$$\boxed{\frac{\partial Q(0,t)}{\partial t} \leq 0 \quad \text{for all } t > 0}$$

Equivalently, the forward default rate must be non-negative: $h(t) \geq 0$. If this constraint is violated, you are modeling a world where the credit becomes *more* likely to survive the longer you wait—an economic absurdity.

### 42.5.2 When Arbitrage Arises

O'Kane explains the mechanism: "Arbitrage occurs in a CDS term structure as soon the value of protection leg for $T_m$ years is the same as the value of the protection leg for $T_{m-1}$ years where $T_m > T_{m-1}$. This is because we are getting the extra protection in the period $[T_{m-1}, T_m]$ for free."

This happens when the spread curve is too steeply inverted. If the 3Y spread is much lower than the 1Y spread (because the market expects the credit to survive the near-term crisis), the protection for years 1-3 may have zero or negative implied cost.

### 42.5.3 Computing the Arbitrage Boundary

For a given spread $S_{m-1}$ at maturity $T_{m-1}$, there exists a lower bound $S_m^*$ such that:

- If $S_m > S_m^*$: the curve is arbitrage-free
- If $S_m = S_m^*$: the survival probability is flat ($Q(T_m) = Q(T_{m-1})$) and hazard rate is zero
- If $S_m < S_m^*$: the curve implies negative hazard (arbitrage)

O'Kane provides an approximation for this boundary. The condition for arbitrage-free curves is approximately:

$$\boxed{S_m \gtrsim S_{m-1} \left(\frac{T_{m-1}}{T_m}\right)}$$

**Example:** For a steeply inverted curve starting at 800 bp at 6M:

| CDS Term | Approximate Lower Bound (bp) | Exact Lower Bound (bp) |
|----------|------------------------------|------------------------|
| 6M | 800 | 800 |
| 1Y | 400 | 419 |
| 2Y | 200 | 219 |
| 3Y | 133 | 150 |
| 5Y | 80 | 96 |
| 10Y | 40 | 54 |

Any spread curve that drops below these bounds cannot be calibrated without violating no-arbitrage.

### 42.5.4 Handling Calibration Failure

O'Kane recommends: "The code should handle this case, and it is up to the user to decide whether to allow this case to proceed or not by changing the bounds on the root solver. If a curve which does not have a monotonically decreasing survival curve is allowed, the fact should be reported to the user."

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
> 4. **Try recovery adjustment**: If the name is distressed, the market recovery may differ from your system's 40% default. Try 30% or 25%—this widens the arbitrage boundary.
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

When hedging a CDS book, we need to know how bumping each market quote affects each position. O'Kane develops this framework in his Chapter 8.5. Consider hedging a CDS position $V$ using on-market CDS at standard tenors $\{S_{1Y}, S_{3Y}, S_{5Y}, S_{7Y}, S_{10Y}\}$.

The hedge notionals $\{N_i\}$ satisfy the system of linear equations:

$$-\frac{\partial V}{\partial S_i} = \sum_{j=1}^{M} N_j \frac{\partial V_j}{\partial S_i} \quad \text{for } i = 1, \ldots, M$$

The key insight is in the structure of the matrix $\frac{\partial V_j}{\partial S_i}$—the sensitivity of the $j$-th hedging instrument to a bump in the $i$-th market quote.

### 42.6.2 The Diagonal Structure

O'Kane demonstrates that for on-market CDS, this matrix is **diagonal**:

$$\boxed{\frac{\partial V_j}{\partial S_i} = 0 \quad \text{for } i \neq j}$$

**Why?** Consider bumping the 10Y CDS spread while holding the 1Y, 3Y, 5Y, and 7Y spreads fixed. The 1Y on-market CDS has a spread equal to the market 1Y quote by definition. Since that quote is unchanged, the 1Y CDS still has zero mark-to-market—its sensitivity to the 10Y bump is zero.

O'Kane explains: "Since the spread of the 1Y on-market contract is held constant as we bump the 10Y spread, its mark-to-market must remain equal to zero and so this sensitivity must also be zero."

For the diagonal elements, O'Kane shows:

$$\boxed{\frac{\partial V_i}{\partial S_i} = -\text{RPV01}(t, T_i)}$$

This is simply the negative of the risky PV01—the same object we use for CS01 calculation in Chapter 43.

### 42.6.3 Example: The 5×5 Sensitivity Matrix

Using O'Kane's Table 8.5, here is the sensitivity matrix for on-market CDS:

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

> **Desk Reality: Why Locality Matters for P&L Attribution**
>
> When you run a credit book, you need to explain daily P&L. If bumping the 5Y spread affected the entire curve, your P&L attribution would show cross-exposures everywhere. You'd tell your manager: "I lost \$50k because 5Y widened, but also because that affected my 10Y positions through the curve..."
>
> With a local bootstrap, the story is simple: "5Y widened 10bp. My 5Y CS01 is \$5k/bp. I lost \$50k." Clean, auditable, explainable.
>
> Non-local curves create "unexplained" P&L that makes risk management harder and auditors nervous.

### 42.6.5 Why Off-Market Contracts Have Non-Zero Off-Diagonal Sensitivities

The diagonal structure holds perfectly only for on-market CDS. For a CDS with contractual spread different from the market spread, O'Kane shows there are small off-diagonal sensitivities.

The reason: when you bump the 3Y spread while holding the 5Y spread fixed, the bootstrap must adjust the forward hazard rate between years 3-5 to maintain the 5Y calibration. This changes the risky PV01 of the 5Y position slightly—the premium leg value depends on survival probabilities in years 1-3, which have changed.

O'Kane's Table 8.6 shows these effects are small but non-negligible for positions with large spread differentials from market.

---

## 42.7 Recovery Sensitivity and Rec01

The survival curve depends critically on the assumed recovery rate $R$. Understanding this sensitivity—both for curve construction and for risk management—is essential for credit practitioners.

### 42.7.1 The Credit Triangle: How Recovery Affects Implied Hazard

The relationship between spread, hazard rate, and recovery follows from the CDS pricing equations. For continuous premiums and constant hazard rate, the par spread approximation gives:

$$\boxed{S \approx \lambda (1 - R)}$$

This is the "credit triangle." Given an observed spread, the implied hazard rate depends on the recovery assumption:

$$\lambda \approx \frac{S}{1 - R}$$

**Higher recovery → Higher implied hazard.** If recovery is 40%, a 200 bp spread implies $\lambda = 200/(1-0.4) = 333$ bp/year hazard. If recovery is 60%, the same 200 bp spread implies $\lambda = 200/(1-0.6) = 500$ bp/year hazard.

**Why?** Higher recovery means lower loss-given-default. To explain the same protection cost, the model requires more frequent defaults. The spread reflects expected loss; if each loss is smaller, defaults must be more frequent.

### 42.7.2 O'Kane's Analytic Formula for Recovery Sensitivity

O'Kane derives an explicit formula for the recovery rate sensitivity of a CDS mark-to-market in Chapter 8.3.6. For a short protection position:

$$\boxed{\frac{\partial V(t)}{\partial R} = \frac{S_t(S_0 - S_t)\Delta}{(1-R)^2} \sum_{n=1}^{N} \tau_n \exp\left(-(r + S_t(1-R)^{-1})\tau_n\right)}$$

where $S_0$ is the contractual spread, $S_t$ is the current market spread, $\Delta$ is the accrual factor, and $\tau_n$ are the remaining times to premium dates.

**Key observations from O'Kane:**

1. **Sensitivity is zero when $S_t = S_0$**: An on-market CDS has zero MTM, so changing recovery doesn't change the mark (it's still zero).

2. **Sign depends on $(S_0 - S_t)$**: If you sold protection at 100 bp and spreads widened to 200 bp, you have negative MTM. Lowering recovery makes that worse (higher implied hazard means more discounting of your negative future cash flows).

3. **Sensitivity is quadratic in spread level**: This is the critical insight for distressed credits.

### 42.7.3 Rec01: The Recovery Rate DV01

Practitioners define **Rec01** (or "Recovery Rate DV01") as the change in mark-to-market for a 1% change in the recovery assumption:

$$\boxed{\text{Rec01} = \frac{\partial V}{\partial R} \times 0.01}$$

O'Kane provides a worked example. Consider a \$10 million short protection position:
- Contractual spread: 100 bp
- Current market spread: 120 bp
- Maturity: 5 years
- Recovery assumption: 40%

At these parameters, the recovery rate sensitivity is 0.000704. For a 5% recovery change (40% → 35%):

$$\Delta V = 0.000704 \times 5\% \times \$10\text{mm} = -\$352$$

**Now consider the same position when spreads have blown out to 500 bp:**

The recovery rate sensitivity jumps to 0.0473—roughly 67 times larger! For the same 5% recovery change:

$$\Delta V = 0.0473 \times 5\% \times \$10\text{mm} = -\$23,670$$

### 42.7.4 Why Rec01 Is Quadratic in Spread

The dramatic increase in recovery sensitivity at wider spreads follows from the formula structure. The sensitivity contains the term $(S_0 - S_t)/(1-R)^2$, which grows with the square of the spread level.

| Spread Level | Rec01 (per 1% recovery) | Interpretation |
|--------------|-------------------------|----------------|
| 50 bp | Negligible | Recovery assumption barely matters |
| 120 bp | Small | Minor impact, usually ignored |
| 300 bp | Moderate | Worth monitoring |
| 500 bp | Large | Recovery is a material risk factor |
| 1000+ bp | Dominant | Recovery uncertainty may exceed spread risk |

> **Desk Reality: The Distressed Name Problem**
>
> For investment-grade credits trading at 80-150 bp, recovery sensitivity is tiny. You can use 40% for everything and not worry.
>
> For distressed names at 500+ bp, recovery is a first-order risk. Different dealers may use different recovery assumptions—30%, 35%, 40%, or even issuer-specific estimates. This creates "model basis" in MTM calculations.
>
> **Before trading a distressed CDS:** Ask your counterparty what recovery they're using. A 10% recovery difference on a \$50mm position at 800 bp spreads can be hundreds of thousands of dollars of "model P&L" that will never converge.

### 42.7.5 Recovery Sensitivity in Curve Construction

When you bootstrap a survival curve, changing the recovery assumption affects all implied hazard rates. The table below shows hazard rates for the same spread curve under different recovery assumptions:

| Recovery $R$ | $h_1$ (0-1Y) | $h_2$ (1-3Y) | $h_3$ (3-5Y) |
|--------------|--------------|--------------|--------------|
| 20% | 1.49% | 2.25% | 3.30% |
| 40% | 1.99% | 3.00% | 4.40% |
| 60% | 2.98% | 4.50% | 6.78% |

**Production implication:** All systems feeding off the survival curve—risk, valuation, collateral—must use consistent recovery assumptions. A mismatch creates phantom P&L and reconciliation breaks.

---

## 42.8 Risk-Neutral vs. Physical Default Probabilities

The survival curve bootstrapped from CDS spreads is a **risk-neutral** object. This distinction, often overlooked by newcomers to credit derivatives, has profound implications for interpretation and risk management.

### 42.8.1 The Two Probability Measures

McNeil's *Quantitative Risk Management* provides the conceptual foundation: "We begin with a discussion of the relationship between the real-world or physical measure, which models the actual probability of default, and an equivalent martingale measure or risk-neutral measure."

The two measures serve fundamentally different purposes:

- **Physical probability** $P$: The actual likelihood that the credit defaults, as might be estimated from historical default rates, rating agency data, or fundamental analysis. This is what you need for reserve calculations and credit risk capital.

- **Risk-neutral probability** $Q$: The probability implied by market prices, which incorporates a risk premium for bearing default uncertainty. This is what you need for derivatives pricing.

Hull states this distinction clearly: "The default probabilities or hazard rates implied from credit spreads are risk-neutral estimates. They can be used to calculate expected cash flows in a risk-neutral world when there is credit risk." For valuation purposes, the risk-neutral probabilities are correct—they incorporate how the market prices default risk, not just the actuarial likelihood of default.

### 42.8.2 Quantifying the Difference: McNeil's Empirical Evidence

McNeil provides empirical analysis of the relationship between risk-neutral and physical default probabilities using CDS spreads and rating agency Expected Default Frequencies (EDFs).

The regression relationship found by Berndt et al. (2004), reported in McNeil, is:

$$\boxed{x_i^* = 52.26 + 1.627 \cdot \text{EDF}_i}$$

where $x_i^*$ is the CDS spread in basis points and $\text{EDF}_i$ is the physical default probability in basis points.

**Interpretation:** For every 10 bp increase in physical default probability, the risk-neutral probability (reflected in spreads) increases by about 16 bp. The intercept of 52 bp represents the "risk premium" even for credits with minimal physical default probability.

From this regression, McNeil derives the ratio of risk-neutral to physical hazard rates:

$$\frac{\gamma^Q}{\gamma^P} \approx \frac{x_i^* / \delta}{\text{EDF}_i} \approx \frac{1}{0.75} \times 1.6 = 2.13$$

Using a more conservative loss-given-default estimate ($\delta = 0.5$):

$$\frac{\gamma^Q}{\gamma^P} \approx \frac{1}{0.5} \times 1.6 = 3.2$$

McNeil notes: "The ratio of risk-neutral to actual default probability gets even higher" for lower-rated credits.

### 42.8.3 The Credit Spread Puzzle and Risk Premium Decomposition

O'Kane explains: "If we compare the risk-neutral default probability implied by credit spreads against the default probabilities implied by historical data, we find that the risk-neutral probability is almost always much larger."

The market spread decomposes into:

$$\text{Market Spread} = \text{Expected Loss} + \text{Risk Premium} + \text{Liquidity Premium}$$

The bootstrapped hazard rate captures all three components, not just expected loss. This is why using CDS-implied survival curves to forecast actual defaults systematically overestimates default frequency.

> **The Paranoia Premium: Risk-Neutral vs. Physical**
>
> Your bootstrapped curve might imply a 20% probability of default over 5 years. Historical data says it's only 2%. Why the 10x difference?
>
> *   **Physical Probability (2%)**: The actuary's view. "Based on history, 2 out of 100 companies fail."
> *   **Risk-Neutral Probability (20%)**: The insurer's price. "I know only 2 might fail, but I'm terrified of being the one holding the bag. I charge you as if 20 will fail."
> *   **The Spread**: The difference isn't error; it's the **Risk Premium**—compensation for fear and uncertainty.
>
> **Critical insight:** *Never use the bootstrapped curve to predict the future; use it to price the risk.*

Hull illustrates this with concrete numbers. For a Baa/BBB-rated credit, the historical 7-year cumulative default probability from rating agency data might be approximately 2.33%. But the hazard rate implied from bond yields at a typical spread of 180 bp with 40% recovery would be $0.018/(1-0.4) = 0.03$, or 3% per year—roughly nine times the historical estimate. Hull notes: "Why do we see such big differences between real-world and risk-neutral default probabilities? This is the same as asking why corporate bond traders earn more than the risk-free rate on average."

### 42.8.4 Time Variation in the Risk Premium

McNeil's Figure 9.6 shows that the ratio of risk-neutral to physical default probabilities varies substantially over time for individual credits—ranging from 1x to 6x or higher for some names. This variation reflects:

- **Credit cycle effects**: Risk premiums spike during stress periods
- **Liquidity conditions**: Illiquidity widens the wedge
- **Idiosyncratic news**: Company-specific events affect market pricing faster than rating agency assessments

### 42.8.5 Practical Implications

The distinction matters for different applications:

| Application | Which Probability? | Why |
|-------------|-------------------|-----|
| CDS/derivative pricing | Risk-neutral | Ensures consistency with market prices |
| Mark-to-market | Risk-neutral | Reflects market's current pricing of risk |
| VaR / credit risk capital | Physical | Need actual default likelihood |
| Reserve calculations | Physical | Actuarial forecasting, not pricing |
| Fundamental analysis | Both | CDS spreads reflect risk appetite + credit quality |

**What can go wrong:** If you use the bootstrapped survival curve to predict actual defaults, you will systematically overestimate default frequency. A credit trading at 200 bp may have a CDS-implied 5-year cumulative default probability of 15%, but the historical default rate for similar credits might be only 5%. The difference is risk premium—what investors demand for bearing the uncertainty, not a forecast of what will happen.

---

## 42.9 The ISDA Standard Model

> **Practitioner Note (not sourced from books/):** The content in this section describes post-2009 industry standardization that occurred after most reference books were published. This represents established market practice but is not verified against the books/ directory.

### 42.9.1 Why Industry Standardized

Before 2009, every major dealer had its own CDS valuation library. These libraries used slightly different conventions for day counts, interpolation methods, accrued-at-default handling, and numerical precision. When two counterparties calculated the same CDS's mark-to-market, they often disagreed by small but meaningful amounts.

**What went wrong:**

1. **Collateral disputes**: If you calculate MTM of \$1.2 million and your counterparty calculates \$1.15 million, who posts the extra \$50,000?

2. **Unwind disagreements**: When terminating a trade, the two sides couldn't agree on the settlement amount.

3. **Basis in "identical" trades**: The market quoted a spread, but the present value of that spread differed by dealer, creating arbitrage-like situations that weren't really arbitrage—just model differences.

### 42.9.2 What the ISDA Standard Model Standardizes

The ISDA CDS Standard Model (sometimes called the "JPMorgan model" after its origin) is an open-source C++ library that defines exactly:

- **Day count**: Actual/360 for premium accruals
- **Premium dates**: IMM quarterly cycle
- **Interpolation**: Piecewise-constant hazard (as described in Section 42.4)
- **Accrued at default**: Included using the trapezoidal approximation
- **Upfront conversion**: Standardized formula for converting running spreads to upfront payments
- **Numerical precision**: Consistent root-finding tolerances

### 42.9.3 The "Law" Status in Practice

> **Desk Reality: The Court-Appointed Ruler**
>
> Think of the ISDA Standard Model as a court-appointed ruler. You might believe your ruler is more accurate. Your counterparty might prefer theirs. But when there's a dispute, the court says: "We're using this ruler."
>
> In practice, this means:
> - **Collateral calls** are computed using ISDA Standard Model MTM
> - **Unwinds** settle at ISDA Standard Model PV
> - **If you disagree with the number**, you still post collateral based on it
>
> You can use your own model internally for risk management or trading decisions. But for anything that involves your counterparty or a central clearinghouse, the ISDA Standard Model is "the law."

### 42.9.4 Practical Implications

1. **Don't reinvent the wheel**: For production systems, use the standard library or a validated port. The numerical edge cases have already been debugged.

2. **Understand the conventions**: Even if you use a different internal model, you need to know what the standard model does so you can reconcile.

3. **"Model P&L" exists**: Your internal model may value a trade at \$1.25mm while the ISDA model says \$1.20mm. The difference is "model P&L"—real for your risk, but unrealized unless you unwind.

4. **Recovery still varies**: The ISDA model takes recovery as an input. Different dealers may still disagree on recovery assumptions for specific names.

---

## 42.10 Practical Implementation

### 42.10.1 Data Sources

CDS quotes are available from:

- **Interdealer brokers** (Markit, Bloomberg, Refinitiv): Composite quotes from multiple dealers
- **Direct dealer quotes**: Real-time executable prices from trading counterparties
- **Exchange-traded products**: Where available (limited for single-name CDS)

The standard tenors are 6M, 1Y, 2Y, 3Y, 4Y, 5Y, 7Y, and 10Y, though not all tenors may be liquid for all credits. Investment-grade names typically have liquid quotes at most tenors; high-yield or distressed names may only have reliable quotes at 5Y.

### 42.10.2 Curve Rebuild Frequency

In production systems:

- **Daily**: Full rebuild for all active issuer curves at end-of-day marks
- **Intraday**: Selective rebuilds for names with significant spread moves or active trading
- **Real-time**: Trade pricing may trigger on-the-fly calibration

The bootstrap algorithm's speed (O'Kane emphasizes this requirement) enables frequent rebuilds across thousands of issuer curves.

### 42.10.3 Common Data Issues

| Issue | Impact | Mitigation |
|-------|--------|------------|
| **Stale quotes** | Calibration to old information; inconsistent curves | Filter by quote age; flag staleness |
| **Missing tenors** | Gaps in curve; reliance on extrapolation | Use available tenors; document assumptions |
| **Crossed quotes** | Potential arbitrage; calibration failure | Verify bid/ask consistency; re-mark |
| **Inverted curves (distressed)** | May violate no-arbitrage; negative hazards | Check arbitrage bounds; adjust recovery or accept limitation |
| **Recovery mismatch** | Different recovery assumptions yield different hazards | Document recovery convention; reconcile across systems |

### 42.10.4 Day Count and Schedule Conventions

CDS premium payments follow specific conventions that must be exactly matched:

- **Day count**: Typically Actual/360 for USD and EUR CDS
- **Payment frequency**: Quarterly (every 3 months) on IMM dates
- **Business day adjustment**: Modified Following
- **Accrued at default**: Standard contracts pay accrued premium when default occurs before next payment date

O'Kane warns: "Do not confuse the quotation of accrued interest with the CDS paying or not paying coupon accrued following a credit event." The quotation convention (clean vs. dirty) is separate from the contractual cashflow mechanics.

**What happens if you get the day count wrong:** Using 30/360 instead of Actual/360 changes the year fractions in the RPV01 calculation, leading to different implied hazard rates for the same quoted spreads. The calibrated curve will not reprice market quotes correctly.

---

## 42.11 Worked Examples

The following examples use a toy discount curve and simplified annual premium schedule to illustrate the mechanics. Real implementations would use quarterly schedules with actual day counts.

### 42.11.1 Setup: Discount Curve and Market Quotes

**Discount curve** (toy: linear decline):

| $t$ (years) | $Z(0,t)$ |
|-------------|----------|
| 0 | 1.000 |
| 1 | 0.985 |
| 2 | 0.970 |
| 3 | 0.955 |
| 4 | 0.940 |
| 5 | 0.925 |

**Market CDS quotes** with $R = 40\%$:

| Maturity | Spread (bp) | Spread (decimal) |
|----------|-------------|------------------|
| 1Y | 120 | 0.0120 |
| 3Y | 160 | 0.0160 |
| 5Y | 200 | 0.0200 |

### 42.11.2 Example A: First Bootstrap Step (1Y)

We solve for $Q(1)$ such that the 1Y CDS has zero mark-to-market.

Using the simplified formulas from Section 42.1.3 with annual payment and discretization:

$$\frac{S}{1-R} = \frac{\text{Protection numerator}}{\text{Premium denominator}}$$

Let $x = S/(1-R) = 0.012/0.60 = 0.02$.

With $Z_0 = 1$, $Z_1 = 0.985$, $Q_0 = 1$:

- Protection numerator: $(Z_0 + Z_1)(Q_0 - Q_1) = 1.985(1 - Q_1)$
- Premium denominator: $Z_1(Q_0 + Q_1) = 0.985(1 + Q_1)$

The par condition gives:

$$0.02 = \frac{1.985(1 - Q_1)}{0.985(1 + Q_1)}$$

Solving:

$$Q_1 = \frac{1 - 0.02 \times 0.985/1.985}{1 + 0.02 \times 0.985/1.985} = \frac{0.990074}{1.009926} \approx 0.98033$$

**Implied hazard rate**:

$$h_1 = -\ln Q_1 = -\ln(0.98033) \approx 0.01987 \text{ (1.987%/year)}$$

### 42.11.3 Example B: Second Bootstrap Step (3Y)

With $Q(1) = 0.98033$ fixed, we solve for $h_2$ on $(1, 3]$ to match the 3Y quote of 160 bp.

Under piecewise-constant hazard: $Q(2) = Q(1)e^{-h_2}$, $Q(3) = Q(1)e^{-2h_2}$.

Target: $x = 0.016/0.60 = 0.02667$.

Trial values:
- $h_2 = 0.030 \Rightarrow Q(2) = 0.95140$, $Q(3) = 0.92283$
- Computed $x \approx 0.02666$ (matches within rounding)

**Result:** $h_2 \approx 0.0300$ (3.00%/year)

### 42.11.4 Example C: Third Bootstrap Step (5Y)

With $Q(3) = 0.92283$ fixed, solve for $h_3$ on $(3, 5]$ to match 200 bp.

Target: $x = 0.020/0.60 = 0.03333$.

Trial values converge to $h_3 \approx 0.0440$ (4.40%/year), giving $Q(5) \approx 0.84527$.

### 42.11.5 Example D: Verification (Repricing Test)

Using the bootstrapped hazards, we verify all quotes reprice:

| Maturity | Market Spread | Model Spread |
|----------|---------------|--------------|
| 1Y | 120 bp | 120 bp |
| 3Y | 160 bp | 160 bp |
| 5Y | 200 bp | 200 bp |

### 42.11.6 Example E: Distressed Credit (Inverted Curve)

Consider a distressed name with spreads:

| Maturity | Spread (bp) |
|----------|-------------|
| 1Y | 500 |
| 3Y | 100 |
| 5Y | 120 |

**Step 1 (1Y):** $x = 0.05/0.60 = 0.0833$. Solving gives $Q(1) \approx 0.9206$, implying high near-term default risk.

**Step 2 (3Y attempt):** Market wants $x = 0.01/0.60 = 0.0167$. But with $Q(1) = 0.9206$ fixed, the minimum achievable $x$ (with $h_2 = 0$, i.e., no additional defaults) is approximately 0.029, corresponding to $S_{\min} \approx 174$ bp.

**Result:** The 100 bp quote is below the arbitrage boundary. No non-negative hazard rate can fit both quotes simultaneously.

**Resolution options:**
1. Re-mark the 3Y spread higher (possibly quote is stale)
2. Use a higher recovery assumption (reducing implied hazard)
3. Accept model arbitrage and document limitation

### 42.11.7 Example F: Recovery Rate Sensitivity

Same quotes, different recovery assumptions:

| Recovery $R$ | $h_1$ (0-1Y) | $h_2$ (1-3Y) | $h_3$ (3-5Y) |
|--------------|--------------|--------------|--------------|
| 20% | 1.49% | 2.25% | 3.30% |
| 40% | 1.99% | 3.00% | 4.40% |
| 60% | 2.98% | 4.50% | 6.78% |

**Interpretation:** Higher recovery means lower loss-given-default. To explain the same spread, the model requires higher default intensity. This is the "credit triangle" relationship: $S \approx \lambda(1-R)$.

### 42.11.8 Example G: Rec01 Calculation

Using O'Kane's example from Section 8.3.6:

**Position:** \$10mm short protection, contractual spread 100 bp, 5Y maturity

**Scenario 1: Market spread = 120 bp, R = 40%**
- Recovery sensitivity: 0.000704
- Rec01 = 0.000704 × 1% × \$10mm = \$70.40 per 1% recovery change

**Scenario 2: Market spread = 500 bp, R = 40%**
- Recovery sensitivity: 0.0473
- Rec01 = 0.0473 × 1% × \$10mm = \$4,730 per 1% recovery change

**The quadratic effect:** At 500 bp, Rec01 is 67x larger than at 120 bp. Recovery assumptions matter far more for distressed names.

### 42.11.9 Example H: Wrong Day Count Impact

Suppose we incorrectly use 30/360 (year fraction = 0.25 per quarter) instead of Actual/360 (year fraction varies by actual days).

For a 91-day quarter:
- 30/360: $\Delta = 0.25$
- Actual/360: $\Delta = 91/360 = 0.2528$

This 1% difference in year fraction flows through to the RPV01 calculation, changing the implied hazard rate. Over multiple years, the cumulative effect can be significant—particularly for back-testing where small discrepancies compound.

**Lesson:** Matching the exact market convention for day count is essential for correct calibration.

### 42.11.10 Example I: Monotonicity Check

We compute the full survival curve on quarterly nodes using the piecewise-constant hazards:

| $t$ | $Q(0,t)$ |
|-----|----------|
| 0.00 | 1.00000 |
| 0.25 | 0.99504 |
| 0.50 | 0.99010 |
| 0.75 | 0.98519 |
| 1.00 | 0.98033 |
| 1.50 | 0.96573 |
| 2.00 | 0.95140 |
| 2.50 | 0.93725 |
| 3.00 | 0.92283 |
| 3.50 | 0.90275 |
| 4.00 | 0.88315 |
| 4.50 | 0.86404 |
| 5.00 | 0.84527 |

**Verification:** Each entry is $\leq$ the previous entry. The curve is strictly decreasing, confirming $h(t) > 0$ everywhere.

### 42.11.11 Example J: Locality Test (5Y Bump)

Keep 1Y=120 bp and 3Y=160 bp, but bump 5Y from 200 bp to 201 bp.

Because the bootstrap is sequential, only the final segment $h_3$ changes:

| Segment | Old hazard | New hazard |
|---------|------------|------------|
| 0–1Y | 0.01987 | 0.01987 |
| 1–3Y | 0.03000 | 0.03000 |
| 3–5Y | 0.04400 | 0.04465 |

**Locality confirmed:** Only the 3–5Y segment moved, matching O'Kane's desired property.

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
| Repricing error | $< 0.01$ bp | Calibration tolerance too loose |

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
- Consistent discounting (typically OIS for collateralized CDS)
- Robust handling of stale or inconsistent quotes
- Efficient computation for large numbers of issuer curves

**Key risk concepts:**
- **Locality**: Quote bumps affect only nearby maturities, making hedges predictable
- **Recovery sensitivity**: Quadratic in spread level; critical for distressed names
- **Risk-neutral vs physical**: CDS-implied probabilities overstate actual defaults by 2-3x

The survival curve is the foundation for CDS mark-to-market, risk measures (Chapter 43), and credit relative value analysis (Chapter 44).

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Survival curve** | $Q(0,t) = \Pr(\tau > t)$ at all times $t$ | Foundation for all CDS valuation |
| **Piecewise-constant hazard** | $h(t)$ constant between skeleton points | Stable interpolation; preserves no-arbitrage |
| **Bootstrap algorithm** | Sequential solve from short to long maturity | Fast, local, exact repricing |
| **No-arbitrage bound** | $0 < Q(T_m) \leq Q(T_{m-1})$ | Ensures non-negative hazard rates |
| **Arbitrage boundary** | $S_m \gtrsim S_{m-1}(T_{m-1}/T_m)$ | Detects impossible quote combinations |
| **Diagonal Jacobian** | $\partial V_j/\partial S_i = 0$ for $i \neq j$ | Locality; simple hedge computation |
| **Rec01** | MTM change per 1% recovery shift | Critical for distressed names |
| **Risk-neutral vs. physical** | Bootstrapped curve embeds risk premium | Not a default forecast |
| **ISDA Standard Model** | Industry-standardized valuation library | Eliminates MTM disputes |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $Q(0,t)$ | Survival probability to time $t$ |
| $h(t)$ | Instantaneous forward default rate |
| $Z(0,t)$ | Discount factor to time $t$ |
| $R$ | Expected recovery rate (% of par) |
| $S_m$ | Market CDS spread at maturity $T_m$ |
| RPV01 | Risky PV01 (premium leg value per unit spread) |
| $\Delta_n$ | Accrual year fraction for period $n$ |
| Rec01 | MTM sensitivity to 1% recovery change |

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
| 16 | What extrapolation assumption does O'Kane recommend? | Flat forward default rate beyond the last quoted maturity |
| 17 | Why does using wrong day count cause problems? | Changes year fractions in RPV01, producing incorrect implied hazards |
| 18 | What is the "credit triangle" relationship? | $S \approx \lambda(1-R)$ for continuous premiums and constant hazard |
| 19 | What is Rec01? | Change in MTM for a 1% change in recovery rate |
| 20 | Why is Rec01 quadratic in spread level? | The sensitivity formula contains $(S_0-S_t)/(1-R)^2$, growing with spread squared |
| 21 | What does the diagonal Jacobian structure imply for hedging? | Hedge notionals are simple: $N_i = (-\partial V/\partial S_i)/\text{RPV01}(T_i)$ |
| 22 | What is the ISDA Standard Model? | Industry-standardized C++ library for CDS valuation; "the law" for collateral |
| 23 | By what factor do risk-neutral default probabilities typically exceed physical? | Approximately 2-3x for IG credits; higher for lower-rated names |
| 24 | For derivatives pricing, should you use risk-neutral or physical default probabilities? | Risk-neutral—they ensure consistency with market prices |
| 25 | Why do CDS-implied hazard rates overstate actual default frequency? | They embed risk premium and liquidity premium in addition to expected loss |

---

## Mini Problem Set

**1. (Warm-up)** Convert 250 bp to decimal spread per year.

**Solution sketch:** $250 \text{ bp} = 250 \times 10^{-4} = 0.025$

---

**2. (Warm-up)** If $R = 35\%$ and $S = 200$ bp, compute $x = S/(1-R)$.

**Solution sketch:** $S = 0.020$, $1-R = 0.65$, so $x = 0.020/0.65 = 0.0308$

---

**3. (Concept)** State the no-arbitrage constraint for survival probabilities.

**Solution sketch:** $Q(0,t)$ must be non-increasing in $t$, equivalently $h(t) \geq 0$

---

**4. (Concept)** Under piecewise-constant hazard on $(T_{k-1}, T_k]$, express $Q(t)$ for $t$ in this interval.

**Solution sketch:** $Q(t) = Q(T_{k-1})\exp(-h(t - T_{k-1}))$ where $h$ is constant on the interval

---

**5. (Computation)** Given $Q(1) = 0.97$, $Q(3) = 0.91$, compute the constant hazard rate on $(1, 3]$.

**Solution sketch:** $h = \frac{1}{3-1}\ln(0.97/0.91) = \frac{1}{2}\ln(1.0659) = 0.032$ (3.2%/year)

---

**6. (Computation)** Given $Q(3) = 0.91$ and $h = 0.04$ on $(3, 5]$, compute $Q(5)$.

**Solution sketch:** $Q(5) = 0.91 \times e^{-0.04 \times 2} = 0.91 \times 0.9231 = 0.840$

---

**7. (Sanity)** A bootstrapped curve shows $Q(3) = 0.95$ and $Q(5) = 0.97$. What is wrong?

**Solution sketch:** Survival increased from 3Y to 5Y, violating monotonicity. Implies negative hazard—arbitrage.

---

**8. (Root finding)** Describe how bisection finds $Q(T_m)$ in the bootstrap.

**Solution sketch:** Define $f(Q_m) = V(0; Q_m)$ = CDS MTM. Start with bounds $[0, Q_{m-1}]$ where $f$ changes sign. Bisect interval until $|f| <$ tolerance.

---

**9. (Interpretation)** If spreads are unchanged but recovery is revised from 40% to 30%, should hazards rise or fall?

**Solution sketch:** Fall. Lower recovery means higher LGD, so same spread is explained with lower default intensity.

---

**10. (Rec01)** A \$20mm short protection position has contractual spread 150 bp, market spread 600 bp, and recovery sensitivity 0.05. Compute Rec01.

**Solution sketch:** Rec01 = 0.05 × 1% × \$20mm = \$10,000 per 1% recovery change

---

**11.** Explain why linear interpolation of $z(t) = -(1/t)\ln Q(t)$ can create jumps in the forward rate.

**Solution sketch:** The forward rate is $h(t) = z(t) + t \cdot \partial z/\partial t$. When $z(t)$ is piecewise linear, its derivative is piecewise constant but discontinuous at knot points. This causes $h(t)$ to jump at each knot.

---

**12.** For a distressed credit with 6M spread of 1000 bp, estimate the approximate lower bound for the 1Y spread.

**Solution sketch:** Using $S_m \gtrsim S_{m-1}(T_{m-1}/T_m)$: $S_{1Y} \gtrsim 1000 \times (0.5/1.0) = 500$ bp (approximate). The exact bound would be slightly higher (~525-550 bp) due to RPV01 ratio effects.

---

**13.** Derive the relationship between RPV01 and the par spread formula.

**Solution sketch:** At par, Protection PV = Premium PV. Premium PV $= S \times \text{RPV01}$. So $S_{\text{par}} = \text{Protection PV} / \text{RPV01}$.

---

**14.** Explain why the bootstrapped curve is unsuitable for forecasting actual defaults.

**Solution sketch:** The bootstrapped curve is risk-neutral—it embeds a risk premium (2-3x factor) on top of expected loss. Using it to predict defaults systematically overstates default frequency. Physical probabilities from rating agency data or structural models should be used for forecasting.

---

**15.** Why does the sensitivity matrix $\partial V_j/\partial S_i$ have diagonal structure for on-market CDS?

**Solution sketch:** An on-market CDS has spread equal to the market quote. Bumping a different tenor's quote doesn't change this CDS's spread (by construction), so its MTM stays at zero. Hence $\partial V_j/\partial S_i = 0$ for $i \neq j$.

---

**16.** Design a verification test for locality in a production bootstrap system.

**Solution sketch:** Build curve with quotes $\{S_1, S_3, S_5, S_7, S_{10}\}$. Bump $S_5$ by 1bp. Rebuild. Verify that hazard rates for intervals before 5Y are unchanged (within numerical tolerance). Verify that hazard rates change only for intervals including or after 5Y.

---

**17.** Given two dealers with different recovery assumptions, explain why their survival curves differ.

**Solution sketch:** Dealer A uses R=40%, Dealer B uses R=30%. For the same quoted spread S, Dealer B infers higher hazard (since $\lambda \approx S/(1-R)$ and $1-0.3 > 1-0.4$). Actually, $\lambda_B = S/0.7 < \lambda_A = S/0.6$—wait, let me recalculate. $\lambda_A = S/0.6$, $\lambda_B = S/0.7$. Since $0.6 < 0.7$, we have $\lambda_A > \lambda_B$. So Dealer A (higher recovery) implies higher hazard.

---

**18.** Explain how to handle missing intermediate tenors (e.g., no 2Y quote when 1Y and 3Y exist).

**Solution sketch:** Bootstrap 1Y normally. For 3Y, the piecewise-constant hazard assumption fills in years 1-3 with a single constant hazard rate. The 2Y point is interpolated automatically—no special handling needed. The curve simply has no "knot" at 2Y, so the 1-3Y hazard is one flat segment.

---

## Source Map

### (A) Book-Verified Facts

| Fact | Source |
|------|--------|
| CDS valuation framework (premium/protection leg split, risky discount factors) | O'Kane Ch 6-7 |
| Bootstrap algorithm (sequential solve, no-arbitrage bounds) | O'Kane Ch 7.5 |
| Piecewise-constant hazard interpolation derivation | O'Kane Ch 7.4 |
| Interpolation alternatives and stability warnings (saw-tooth, arbitrage risk) | O'Kane Ch 7.4, 7.6 |
| Desirable curve properties (exact fit, locality, speed, smoothness trade-off) | O'Kane Ch 7.2 |
| Arbitrage detection and boundary computation | O'Kane Ch 7.7 |
| Arbitrage boundary table (Table 7.3) | O'Kane Ch 7.7 |
| Recovery sensitivity formula and Rec01 definition | O'Kane Ch 8.3.6 |
| Rec01 quadratic dependence on spread level | O'Kane Ch 8.3.6 |
| Sensitivity matrix diagonal structure | O'Kane Ch 8.5 (Equations 8.5-8.6, Table 8.5) |
| Risk-neutral vs. physical default probabilities | McNeil QRM Ch 9.3, Hull OFD Ch 24.5 |
| Regression equation for RN vs physical | McNeil Ch 9.3 (Equation 9.12) |
| Ratio of RN to physical probabilities (~2-3x or higher) | McNeil Ch 9.3, Hull Ch 24.5 |
| "Hazard rates implied from credit spreads are risk-neutral estimates" | Hull Ch 24 |
| Parallel between IR and credit curve construction | O'Kane Ch 7.3 (Table 7.1) |
| Day count conventions (Actual/360 for CDS) | Hull OFD Ch 4, 6; O'Kane Ch 6 |
| Recovery rate consensus (40% for IG senior unsecured) | O'Kane Ch 7.7.1 |
| Root-finding methods (bisection, Newton-Raphson, Brent) | O'Kane Ch 7.5 |

### (B) Claude-Extended Content

| Content | Basis |
|---------|-------|
| ISDA Standard Model section (42.9) | Extends O'Kane's methodology to post-2009 standardization practice |
| "Desk Reality" troubleshooting workflow | Extends O'Kane's calibration failure discussion with operational practice |
| "Court-appointed ruler" analogy for ISDA model | Pedagogical extension to explain standardization |
| Time variation in risk premium (1x-6x range) | Extends McNeil's Figure 9.6 discussion |

### (C) Reasoned Inference

| Inference | Derivation |
|-----------|------------|
| Equivalence of bootstrapping $Q(T_k)$ vs. $h_k$ | Integration of $Q(t) = \exp(-\int h)$ |
| Locality follows from sequential structure | Earlier curve segments fixed in later steps |
| Higher recovery requires higher hazard | From credit triangle $S = \lambda(1-R)$ |
| Arbitrage boundary approximation | Protection leg = RPV01 × spread at boundary (O'Kane Eq 7.5) |
| Diagonal Jacobian implies simple hedge computation | Matrix inversion becomes trivial |

### (D) Flagged Uncertainties

| Uncertainty | Note |
|-------------|------|
| ISDA Standard Model details | Post-2009 standardization not covered in reference books; marked as Practitioner Note |
| OIS vs. Libor discounting in modern practice | Modern practice uses OIS for collateralized CDS; sources written in Libor era |
| Exact ISDA standardization details | Depends on ISDA definitions vintage (2003 vs 2014) and currency |
| Specific liquidity premium magnitude | Sources discuss conceptually but don't quantify |
| Current market recovery conventions by sector | May vary from the 40% standard discussed in sources |

---

*Last Updated: January 26, 2026*
