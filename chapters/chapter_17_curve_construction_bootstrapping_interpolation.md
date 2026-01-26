# Chapter 17: Curve Construction — Bootstrapping, Interpolation, and the Spline Zoo

---

## Introduction

Imagine you are trying to draw a map of the ocean floor, but you only have depth measurements at ten specific buoys. You know exactly how deep it is at those ten points, but what is happening in between? Is the floor flat? Does it slope gently? Or is there a sudden trench just barely missing your measurements?

This is the fundamental challenge of **curve construction**. In the fixed income market, we observe prices for a finite set of "benchmark" instruments—a few dozen deposits, futures, and swaps. From these sparse data points, we must infer a continuous **discount factor curve** that allows us to assign a value to cash flows occurring on *any* date, not just the ones where instruments mature.

This is an **inverse problem**. The market gives us the outputs (prices), and we must reverse-engineer the input (the discount function). Andersen and Piterbarg frame the issue precisely: "the problem of curve construction essentially boils down to supplementing [the pricing equations] with enough additional assumptions to allow us to extract $\mathbf{P}$ and to determine $P(T)$ for values of $T$ not in the cash flow timing set." The problem is inherently **underdetermined**—there are infinitely many curves that could perfectly explain the observed market prices. To choose one, we must impose a structure: a mathematical rule for how to connect the dots.

**Why this matters:** The choice of structure—specifically, the **interpolation method**—is not a neutral administrative detail. It is a modeling decision with real financial consequences. Two desks using identical market quotes but different interpolation schemes will calculate different forward rates, different risk sensitivities, and ultimately different valuations for non-standard instruments. As Andersen and Piterbarg emphasize, understanding the artifacts introduced by your curve construction routine—such as "saw-tooth" forwards or "ringing" oscillations—is crucial for managing a book's actual risk. Tuckman adds that "the shortcomings of a curve fitting technique are least noticeable in the discount function, more noticeable in the spot rate curve, and particularly noticeable in the forward rate curve."

**Roadmap:** This chapter will:

1. **Frame the Inverse Problem** (Section 17.1): Why finite quotes cannot uniquely determine a continuous function.
2. **Master Bootstrapping** (Section 17.2): The industry-standard sequential algorithm for extracting discount factors.
3. **Survey the Interpolation Zoo** (Section 17.3): From piecewise linear yields to cubic splines to tension splines—each method's artifacts and tradeoffs.
4. **Derive Forward Rate Behavior** (Section 17.4): The mathematical reason behind saw-tooth and staircase patterns.
5. **Examine Locality and Ringing** (Section 17.5): The critical question of whether bumping one quote disturbs distant parts of the curve.
6. **Explore Smoothing vs. Exact Fit** (Section 17.6): When to sacrifice precision for stability.

We focus here on the mechanics of building a *single* discount curve. The modern multi-curve framework (OIS discounting distinct from projection curves) builds directly on these techniques and is the subject of Chapters 18–19.

---

## 17.1 The Inverse Problem: Finite Quotes, Continuous Curve

### 17.1.1 The Pricing Equation

At its core, valuation is linear algebra. The value of any fixed-income instrument is the sum of its cash flows multiplied by the discount factors for those dates. If instrument $i$ pays cash flows $c_{i,j}$ at times $t_j$, its value $V_i$ is:

$$\boxed{V_i = \sum_{j=1}^{M} c_{i,j} \, P(t_j)}$$

Here, $P(t)$ is the **discount factor**: the value today of one unit of currency to be received at time $t$. This is the fundamental object we are trying to discover.

If we have $N$ benchmark instruments with cash flows on $M$ distinct dates, we can stack these equations into a matrix system. Let $\mathbf{V} = (V_1, \ldots, V_N)^\top$ be the vector of observed market prices, $\mathbf{P} = (P(t_1), \ldots, P(t_M))^\top$ the vector of discount factors, and $\mathbf{C}$ the $N \times M$ matrix of cash flow coefficients. We are solving:

$$\mathbf{V} = \mathbf{C} \mathbf{P}$$

### 17.1.2 The Underdetermined Nature

The difficulty lies in the dimensions. A typical curve might be built from 15 liquid instruments (deposits, futures, swaps). However, these instruments generate cash flows on hundreds of distinct dates—every coupon payment for 30 years. Worse, to price a client's specific deal, we might need a discount factor for a date like 1.42 years, which appears in none of our benchmarks.

We are solving for a continuous function $P(T)$ defined for all $T \ge 0$, but we only have $N$ constraints. This means **you cannot build a curve without an interpolation assumption**. When a trader quotes the "market rate" for a 17-month loan, but there is no 17-month liquidity, they are quoting you their interpolation model, not a traded price.

Andersen and Piterbarg identify three basic approaches to resolve this underdetermination:

1. **Add fictitious securities:** Pad the benchmark set using interpolation rules applied to existing instruments. This is ad-hoc and can lead to odd-looking curves.

2. **Parametric/Spline representation:** Assume the yield curve has a specific functional form (e.g., Nelson-Siegel) or is a spline with $N$ knots at benchmark maturities. Solve for the knot values.

3. **Optimization with regularization:** Replace exact pricing with penalized least-squares, allowing the curve to miss quotes slightly in exchange for smoothness.

In practice, option 2 (splines with bootstrapping) is the most common for constructing LIBOR/SOFR curves, while option 3 is used for noisy benchmark sets like corporate bonds.

---

## 17.2 Bootstrapping: Sequential Inversion

The most common method to solve the curve is **bootstrapping**. Instead of solving for all discount factors simultaneously, we order the instruments by maturity and solve for them one by one, extending the curve step-by-step.

> **Analogy: Building a Bridge Plank by Plank**
>
> You can't just build the middle of a bridge. You have to start at the bank (Today) and extend out.
> *   **Plank 1 (3 Months)**: You attach the 3-month deposit rate. Secure it.
> *   **Plank 2 (6 Months)**: You attach the 6-month rate, *standing on* the 3-month plank.
> *   **Plank 3 (5 Years)**: You attach the 5-year swap, *standing on* all the previous planks.
>
> If Plank 1 is loose (bad 3m rate), Plank 3 will wobble violently. This is how errors propagate in bootstrapping.

### 17.2.1 The Algorithm

Andersen and Piterbarg describe the basic bootstrap iteration:

1. Let $P(t_j)$ be known for $t_j \le T_{i-1}$, such that prices for benchmark securities $1, \ldots, i-1$ are matched.
2. Make a guess for $P(T_i)$.
3. Use an interpolation rule to fill in $P(t_j)$ for $T_{i-1} < t_j < T_i$.
4. Compute $V_i$ from the now-known values of $P(t_j)$, $t_j \le T_i$.
5. If $V_i$ equals the market value, stop. Otherwise return to Step 2.
6. If $i < N$, set $i = i + 1$ and repeat.

The updating of guesses in Steps 2–5 can be handled by any standard one-dimensional root-search algorithm (Newton-Raphson or secant method). Hull emphasizes that this bootstrap method "involves starting with short-term instruments and moving progressively to longer-term instruments, making sure that the zero rates calculated at each stage are consistent with the prices of the instruments. It is used daily by trading desks to calculate a variety of zero curves."

### 17.2.2 Closed-Form Solutions for Standard Instruments

For common instruments, each bootstrap step often has a closed-form solution.

**Money Market Deposits**

A deposit earning a simple rate $L$ over time $\tau$ pays $(1 + L\tau)$ at maturity. Its present value must equal the notional (which we normalize to 1). Thus:

$$\boxed{P(\tau) = \frac{1}{1 + L\tau}}$$

**Par Swaps**

A standard fixed-for-floating swap is quoted at a fixed rate $c$ that makes the NPV zero. Since the floating leg is worth par at inception (by the standard replication argument), the fixed leg plus principal repayment must also equal 1:

$$1 = c \sum_{j=1}^n \tau_j P(t_j) + P(t_n)$$

If we already know the discount factors for all previous coupons $P(t_1), \ldots, P(t_{n-1})$, we can rearrange to solve for the final discount factor:

$$\boxed{P(t_n) = \frac{1 - c \sum_{j=1}^{n-1} \tau_j P(t_j)}{1 + c \tau_n}}$$

### 17.2.3 Worked Example: Building a Curve from Scratch

Let's construct a simple curve to see the mechanics in action.

**Market Quotes:**
- 3M Deposit: 5.00%
- 6M Deposit: 5.50%
- 1Y Deposit: 6.00%
- 2Y Swap (Annual Pay): 6.50%

**Step 1: The Short End (Deposits)**

We convert rates to discount factors directly:

$$P(0.25) = \frac{1}{1 + 0.05 \times 0.25} = \frac{1}{1.0125} = \mathbf{0.987654}$$

$$P(0.50) = \frac{1}{1 + 0.055 \times 0.50} = \frac{1}{1.0275} = \mathbf{0.973236}$$

$$P(1.00) = \frac{1}{1 + 0.06 \times 1.00} = \frac{1}{1.0600} = \mathbf{0.943396}$$

**Step 2: The Long End (Swap)**

The 2-year swap pays coupons at Year 1 and Year 2. The par rate is 6.50%. We know $P(1)$ from the deposit. We need $P(2)$.

Using the swap formula:

$$P(2) = \frac{1 - 0.065 \times P(1)}{1 + 0.065 \times 1} = \frac{1 - 0.065 \times 0.943396}{1.065}$$

Numerator: $1 - 0.061321 = 0.938679$

Result: $P(2) = 0.938679 / 1.065 = \mathbf{0.881389}$

**Verification (The Reprice Test):**

Does this curve actually price the swap to par?

$$\text{PV Fixed Leg} = 0.065 \times P(1) + 1.065 \times P(2)$$
$$= 0.065 \times 0.943396 + 1.065 \times 0.881389$$
$$= 0.061321 + 0.938679 = 1.0000 \; \checkmark$$

The bootstrap is successful. We have solved for the nodes: $(0.25, 0.988), (0.5, 0.973), (1.0, 0.943), (2.0, 0.881)$.

---

## 17.3 The Interpolation Zoo: From Piecewise Linear to Tension Splines

We found the nodes, but what is $P(1.5)$? This is where interpolation enters. The choice of interpolation method determines the shape of the curve between solved points—and more importantly, the shape of the **forward curve**. Andersen and Piterbarg provide a comprehensive survey of methods, organized by the smoothness (differentiability) of the resulting yield curve.

### 17.3.1 $C^0$ Methods: Continuous but Not Smooth

> **Deep Dive: Spline vs. Linear (The Smooth Ride)**
>
> *   **Linear Interpolation**: Connects dots with straight lines. Easy, but the ride is jerky. The "speed" (Forward Rate) jumps instantly at every node.
> *   **Spline Interpolation (Cubic)**: Connects dots with flexible metal strips. The ride is buttery smooth. The "speed" changes gradually.
> *   **Tradeoff**: Splines look better but can "wobble" (Ring) if you hit a bump. Linear is ugly but stable. Pro desks use **Tension Splines** (stiff splines) to get the best of both.

These methods produce a yield curve that is continuous but has kinks (discontinuous first derivative) at the knot points. The resulting forward curve is discontinuous.

#### Piecewise Linear Yields

The most common bootstrap algorithm assumes that the continuously compounded yield $y(T)$ is a piecewise linear function:

$$y(T) = y(T_i) \frac{T_{i+1} - T}{T_{i+1} - T_i} + y(T_{i+1}) \frac{T - T_i}{T_{i+1} - T_i}, \quad T \in [T_i, T_{i+1}]$$

with $P(T) = e^{-y(T)T}$.

Andersen and Piterbarg observe that "the discontinuous 'saw-tooth' shape of the forward curve is characteristic for bootstrapped yield curves with piecewise linear yield." We derive this mathematically in Section 17.4.

#### Piecewise Flat (Log-Linear) Forwards

An alternative assumes the instantaneous forward rate is piecewise flat:

$$f(T) = f(T_i), \quad T \in [T_{i}, T_{i+1})$$

This corresponds to log-linear interpolation on discount factors:

$$\ln P(T) = (1-w) \ln P(T_i) + w \ln P(T_{i+1})$$

where $w = (T - T_i)/(T_{i+1} - T_i)$.

**Implications:** The forward curve is a "staircase"—constant between nodes, then jumping at each maturity date. The yield curve remains continuous but becomes nonlinear between nodes:

$$y(T) = \frac{1}{T}\left(T_i y(T_i) \frac{T_{i+1}-T}{T_{i+1}-T_i} + T_{i+1} y(T_{i+1}) \frac{T-T_i}{T_{i+1}-T_i}\right)$$

### 17.3.2 $C^1$ Methods: Hermite Splines

To produce a continuous forward curve, we need a yield curve that is at least once differentiable. Andersen and Piterbarg recommend **Hermite cubic splines**, specifically the **Catmull-Rom** variant, where derivatives at knot points are estimated from finite differences.

For $T \in [T_i, T_{i+1}]$, the Catmull-Rom spline writes:

$$y(T) = \mathbf{D}_i(T)^\top \mathbf{A}_i \begin{pmatrix} y_{i-1} \\ y_i \\ y_{i+1} \\ y_{i+2} \end{pmatrix}$$

where $\mathbf{D}_i(T)$ is a vector of powers of the normalized distance $d_i = (T - T_i)/h_i$, and $\mathbf{A}_i$ is a matrix encoding the Catmull-Rom coefficients.

**Key properties:**
- The yield curve is continuous and has a continuous first derivative.
- The forward curve is continuous but not differentiable at knot points.
- Good **locality**: the price of security $i$ depends only on $y_1, \ldots, y_{i+1}$.
- Requires iteration (not pure sequential bootstrap) since $V_i = F_i(y_1, \ldots, y_{i+1})$, but the iteration converges quickly.

### 17.3.3 $C^2$ Methods: Natural Cubic Splines

For a smooth (differentiable) forward curve, we need a yield curve that is twice differentiable. The **natural cubic spline** satisfies $y''(T_1) = y''(T_N) = 0$ at the boundaries and achieves $C^2$ continuity everywhere.

The spline representation is:

$$y(T) = \frac{(T_{i+1}-T)^3}{6h_i}y''_i + \frac{(T-T_i)^3}{6h_i}y''_{i+1} + (T_{i+1}-T)\left(\frac{y_i}{h_i} - \frac{h_i}{6}y''_i\right) + (T-T_i)\left(\frac{y_{i+1}}{h_i} - \frac{h_i}{6}y''_{i+1}\right)$$

where the second derivatives $y''_i$ are determined by solving a tridiagonal linear system that enforces $C^2$ continuity across all knots.

**The Ringing Problem:** Andersen and Piterbarg warn that $C^2$ cubic splines are "often subject to oscillatory behavior, spurious inflection points, poor extrapolatory behavior, and non-local behavior when prices in the benchmark set are perturbed." Perturbation of a single benchmark price can cause a slow-decaying "ringing" effect, spreading into the entire yield curve. This occurs because interpolation on $[T_i, T_{i+1}]$ depends on *all* values $y_1, \ldots, y_N$ through the full $(N-2) \times (N-2)$ matrix system.

### 17.3.4 Tension Splines: The Best of Both Worlds

To retain $C^2$ smoothness while controlling locality and stiffness, Andersen and Piterbarg advocate **exponential tension splines**. The key modification replaces the pure cubic with a hyperbolic form:

$$y(T) = \left(\frac{\sinh(\sigma(T_{i+1}-T))}{\sinh(\sigma h_i)} - \frac{T_{i+1}-T}{h_i}\right)\frac{y''_i}{\sigma^2} + \left(\frac{\sinh(\sigma(T-T_i))}{\sinh(\sigma h_i)} - \frac{T-T_i}{h_i}\right)\frac{y''_{i+1}}{\sigma^2} + y_i \frac{T_{i+1}-T}{h_i} + y_{i+1} \frac{T-T_i}{h_i}$$

where $\sigma \geq 0$ is the **tension factor**.

**Critical insight from Andersen and Piterbarg:**
- Setting $\sigma = 0$ recovers the ordinary $C^2$ cubic spline.
- As $\sigma \to \infty$, the tension spline uniformly approaches a linear spline.
- A tension spline is thus "a twice differentiable hybrid between a cubic spline and a linear spline."

Regarding the choice of $\sigma$, they state: "We have no definitive answers to this question... Instead, we normally treat $\sigma$ as an 'extra knob' that allows users to balance curve smoothness, shape preservation, and perturbation locality to their particular tastes. Inevitably some element of experimentation is required here."

### 17.3.5 Comparison Table

| Method | Yield Curve | Forward Curve | Locality | Comments |
|--------|-------------|---------------|----------|----------|
| Piecewise Linear $y$ | $C^0$ (kinked) | Discontinuous (saw-tooth) | Excellent | Fast, but poor forwards |
| Log-Linear (Flat $f$) | $C^0$ (curved) | Piecewise flat (staircase) | Excellent | Simple baseline |
| Hermite $C^1$ | $C^1$ (smooth) | Continuous, non-smooth | Good (4 neighbors) | Catmull-Rom common |
| Cubic $C^2$ | $C^2$ (smooth) | Smooth | Poor (global matrix) | Risk of ringing |
| Tension $C^2$ | $C^2$ (smooth) | Smooth | Tunable via $\sigma$ | Best compromise |

---

## 17.4 The Mathematics of Forward Rate Artifacts

Why do different interpolation methods produce such different forward rate curves? The answer lies in the relationship between yields and instantaneous forwards.

### 17.4.1 The Fundamental Relationship

Recall that the discount factor, zero rate, and instantaneous forward rate are related by:

$$P(T) = e^{-y(T)T} = \exp\left(-\int_0^T f(u)\, du\right)$$

Differentiating the first equality:

$$\boxed{f(T) = y(T) + T y'(T)}$$

This is the key equation. The forward rate equals the zero rate plus a term proportional to the *slope* of the zero rate curve times the maturity.

### 17.4.2 The Saw-Tooth Pattern

When yields are piecewise linear, $y(T)$ has constant slope within each interval:

$$y'(T) = \frac{y(T_{i+1}) - y(T_i)}{T_{i+1} - T_i} = \text{const} \quad \text{for } T \in [T_i, T_{i+1}]$$

Substituting into the forward rate formula:

$$f(T) = y(T_i) \frac{T_{i+1}-T}{T_{i+1}-T_i} + y(T_{i+1}) \frac{T-T_i}{T_{i+1}-T_i} + T \cdot \frac{y(T_{i+1}) - y(T_i)}{T_{i+1}-T_i}$$

This is a **linear function of $T$** within each interval. But at the boundary $T = T_{i+1}$, the slope changes abruptly. The forward rate jumps, creating the characteristic "saw-tooth" pattern.

> **Visual: The Sawtooth**
>
> If you plot the Forward Rate using linear zero-rate interpolation, it looks like the teeth of a saw.
> *   Rate rises... Jump down!
> *   Rate rises... Jump down!
> *   It's mathematically correct but financially nonsensical. (Why would rates jump exactly on Dec 15th?).
> *   This is why pros don't use simple linear interpolation for pricing options (which depend on forward volatility).

**Worked example:** Using our earlier nodes with $y(1) = 6.0\%$ and $y(2) = 6.35\%$ (implied by $P(1) = 0.9434$, $P(2) = 0.8814$):

At $T = 1.5$ (middle of interval):
$$f(1.5) = y(1.5) + 1.5 \times \frac{0.0635 - 0.0600}{1} = 6.175\% + 1.5 \times 0.35\% = 6.70\%$$

At $T$ just before 2.0:
$$f(2^-) = 6.35\% + 2.0 \times 0.35\% = 7.05\%$$

At $T$ just after 2.0 (if the next segment has different slope):
$$f(2^+) = \text{different value}$$

The jump at $T = 2$ is the "tooth" of the saw.

### 17.4.3 The Staircase Pattern

With log-linear interpolation (piecewise flat forwards):

$$\ln P(T) = \ln P(T_i) - f_i (T - T_i)$$

where $f_i$ is the constant forward rate on $[T_i, T_{i+1})$. By definition, $f(T) = -\frac{d}{dT}\ln P(T) = f_i$, which is constant on each interval.

At each node, the forward rate jumps to a new constant level, creating a staircase. While unrealistic, this method has the advantage of being extremely stable and local.

---

## 17.5 Locality, Perturbation, and Ringing

When you choose an interpolation method, you aren't just picking a shape; you are picking a **risk profile**. A critical property is **locality**: does a change in the 2-year swap rate affect the 30-year part of the curve?

### 17.5.1 The Par-Point Approach

The standard approach to computing interest rate deltas involves bumping each benchmark price $V_i$ by a small amount (typically 1 basis point on the yield), reconstructing the curve, and repricing the portfolio. Andersen and Piterbarg call this the **par-point approach**.

"For the approach to work properly, it is important that the yield curve construction algorithm is fast and produces clean, local perturbations of the yield curve when benchmark prices are shifted. For instance, perturbing a short-dated FRA price should not cause noticeable movements in long-term yields, lest we reach the erroneous conclusion... that we can perfectly hedge a 20 year swap with a 1 month FRA."

### 17.5.2 Ringing in $C^2$ Splines

Andersen and Piterbarg provide a striking illustration. When the 2-year swap yield is bumped by 1 basis point while other yields remain fixed:

- **Bootstrap/Linear:** The forward curve shifts cleanly in the 1–3 year region, with minimal effect elsewhere.
- **Hermite $C^1$:** Similar locality, affecting only 4 neighboring points.
- **Cubic $C^2$:** The move causes a "noisy, ringing perturbation... spreading into short- and long-dated parts of the forward curve."

The ringing occurs because the $C^2$ spline solves a global matrix system, so changing one input propagates through the entire solution.

### 17.5.3 Tension Dampens Ringing

Adding tension to the $C^2$ spline progressively suppresses oscillations. Andersen and Piterbarg show that as $\sigma$ increases, the perturbation response becomes more local, approaching the clean behavior of bootstrapping. "The usage of a tension factor can have a beneficial impact on risk reports produced by the par-point approach."

### 17.5.4 Why Locality Matters for P&L

Traders hate ringing because it creates **false explanatory power**. If you hedge a 30-year bond using 30-year swaps, you expect your P&L to be stable when short-term rates move. If your curve construction has poor locality, a move in the Fed Funds rate might imply a spurious P&L change in your 30-year book, purely because your spline "wiggled."

Andersen and Piterbarg note a practical workaround: "building two different curves. One—smooth—is then used for pricing and the other—bootstrapped and with good locality—used for risk computations." However, this "tends to suffer from poor P&L predict, in the sense that changes in valuations... are not well explained by first-order sensitivities."

---

## 17.6 Exact Fit vs. Smoothing

So far, we have assumed the curve must hit every market price exactly. This is the **exact fit** approach. But market data is noisy.

### 17.6.1 Sources of Noise

- **Bid-ask spreads:** The "mid" price is itself an interpolation.
- **Stale quotes:** Some prices may be hours old.
- **Data entry errors:** Typos happen.
- **Illiquid instruments:** Prices may not reflect executable levels.

If you force a curve to pass exactly through a "bad" quote (e.g., a typo where 2.15% is entered as 2.51%), the curve will dip violently to hit that point and then rip back up to the next valid point. This creates wild forward rates and massive, incorrect hedge ratios.

### 17.6.2 The Penalized Least-Squares Approach

Andersen and Piterbarg formulate the curve fitting problem as an optimization:

$$\hat{y} = \underset{y \in \mathcal{A}}{\text{argmin}} \; \mathcal{I}(y)$$

where

$$\mathcal{I}(y) = \frac{1}{N}(\mathbf{V} - \mathbf{C}\mathbf{P}(y))^\top \mathbf{W}^2 (\mathbf{V} - \mathbf{C}\mathbf{P}(y)) + \lambda \int_{t_1}^{t_M} \left[y''(t)^2 + \sigma^2 y'(t)^2\right] dt$$

The norm consists of:

1. **Pricing error penalty:** How far the model price is from the market quote, weighted by $\mathbf{W}$.
2. **Smoothness penalty:** $\lambda \int y''(t)^2 \, dt$ penalizes high curvature.
3. **Curve-length penalty:** $\lambda \sigma^2 \int y'(t)^2 \, dt$ penalizes oscillations.

The parameter $\lambda$ controls the tradeoff between fitting prices and smoothness.

### 17.6.3 A Key Result

Andersen and Piterbarg prove:

> **Proposition:** The curve $\hat{y}$ that minimizes the penalized norm is a natural exponential tension spline with tension factor $\sigma$ and knots at all cash flow dates.

This provides theoretical justification for tension splines: they are not just ad-hoc—they are the optimal curves in a well-defined variational sense.

### 17.6.4 Choosing $\lambda$

If $\lambda$ is high, you get a smooth curve that might miss prices by a fraction of a basis point. If $\lambda$ is low, you get a curve that hits every price but may wiggle unreasonably.

A practical approach replaces the unconstrained problem with a constrained one:

$$\min_{y} \int \left[y''(t)^2 + \sigma^2 y'(t)^2\right] dt \quad \text{subject to} \quad \text{RMSE} = \gamma$$

where $\gamma$ is the allowed root-mean-square pricing error (e.g., 0.1 basis points based on observed bid-offer spreads).

Tuckman emphasizes the art involved: "A RMSE of 3 basis points means that ±3 basis points correspond to a 1-standard deviation error of the fit... choosing the parameters to minimize the RMSE... is one way to fit the spot rate function."

---

## 17.7 The Forward Rate Approach and Jacobian Method

Rather than bumping benchmark prices, some practitioners apply perturbations directly to the forward curve itself.

### 17.7.1 Forward Rate Deltas

Define functional shifts $\mu_k(t)$ to the forward curve $f(t)$:

$$f(t) \mapsto f(t) + \varepsilon \mu_k(t)$$

Common choices include:
- **Piecewise flat:** $\mu_k(t) = \mathbf{1}_{\{t \in [t_k, t_{k+1})\}}$
- **Triangular:** Rises linearly to a peak at $t_k$, then falls linearly.

The functional derivatives $\partial_k V_0 = \frac{d V_0(f + \varepsilon \mu_k)}{d\varepsilon}\big|_{\varepsilon=0}$ are called **forward rate deltas**.

### 17.7.2 The Jacobian Method

To translate forward rate deltas into hedge positions, define the Jacobian matrix $\partial \mathbf{H}$ with columns $(\partial_1 H_1, \ldots, \partial_K H_L)^\top$, where $H_l$ is the value of the $l$-th hedging instrument.

The optimal hedge weights $\mathbf{p}$ solve:

$$\hat{\mathbf{p}} = \underset{\mathbf{p}}{\text{argmin}} \left(\sum_{k=1}^K W_k^2 (\partial_k H_0(\mathbf{p}) - \partial_k V_0)^2 + \sum_{l=1}^L U_l^2 p_l^2\right)$$

The solution is:

$$(\partial \mathbf{H} \mathbf{W}^2 \partial \mathbf{H}^\top + \mathbf{U}^2) \hat{\mathbf{p}} = \partial \mathbf{H} \mathbf{W}^2 \partial \mathbf{V}_0$$

Andersen and Piterbarg note that "the Jacobian method serves to decouple risk calculations from curve construction. This, potentially, allows for combining smooth curves with localized risk, a feat that is difficult to achieve by other methods."

---

## Summary

Curve construction is the bedrock of fixed income quantitative modeling. It is an inverse problem that requires inferring a continuous reality from sparse, finite observations.

1. **Underdetermination is fundamental:** There is no "true" curve, only a model that fits the prices. Every curve embeds interpolation assumptions.

2. **Bootstrapping is the workhorse:** The standard iterative technique solves the curve node-by-node, with excellent locality.

3. **Interpolation method matters profoundly:** Simpler methods (linear yield) create saw-tooth forward artifacts. Log-linear produces staircases. Cubic splines smooth the curve but risk ringing.

4. **Tension splines offer a tunable compromise:** They retain $C^2$ smoothness while allowing control over locality. The tension parameter $\sigma$ balances smoothness against stiffness.

5. **Locality is key for risk management:** A robust method ensures that shocks to short-term rates don't destabilize long-term valuations.

6. **Smoothing vs. exact fit:** Sometimes it is better to miss a quote by 0.1 bps than to wreck the curve's shape to fit a bad data point.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Discount Factor $P(T)$** | Value today of 1 unit payable at time $T$ | The universal currency of valuation |
| **Bootstrapping** | Sequential method to solve curve nodes | Most common, robust, excellent locality |
| **Interpolation** | Rule for filling gaps between solved nodes | Determines forward curve shape |
| **Inverse Problem** | Inferring the function ($P$) from outputs ($V$) | Highlights that the curve is a model |
| **Saw-Tooth** | Forward artifact from linear yield interpolation | Unrealistic jumps in forwards |
| **Staircase** | Forward artifact from log-linear interpolation | Constant forwards between nodes |
| **Ringing** | Oscillations from global $C^2$ splines | Creates false risk sensitivities |
| **Locality** | Input changes affect only nearby outputs | Essential for stable hedging |
| **Tension Spline** | $C^2$ spline with adjustable stiffness | Balances smoothness and locality |
| **Penalized Least-Squares** | Optimization allowing controlled pricing errors | Handles noisy benchmark sets |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $P(T)$, $P(0,T)$ | Discount factor for maturity $T$ |
| $y(T)$ | Continuously compounded zero rate |
| $f(T)$ | Instantaneous forward rate |
| $T_i$ | Maturity of $i$-th benchmark instrument |
| $h_i = T_{i+1} - T_i$ | Interval length |
| $\sigma$ | Tension parameter for splines |
| $\lambda$ | Smoothing parameter in optimization |
| $\mathbf{C}$ | Cash flow coefficient matrix |
| $\mathbf{V}$ | Vector of market prices |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the fundamental unknown in curve construction? | The continuous discount factor function $P(T)$. |
| 2 | Why is the curve problem "underdetermined"? | We have $N$ finite quotes but need a continuous function ($\infty$ points). |
| 3 | What is bootstrapping? | Sequential algorithm that solves for discount factors short-to-long. |
| 4 | What formula gives $P(\tau)$ from a simple deposit rate $L$? | $P(\tau) = 1/(1 + L\tau)$ |
| 5 | What formula gives $P(t_n)$ from a par swap rate $c$? | $P(t_n) = (1 - c \sum_{j=1}^{n-1} \tau_j P(t_j))/(1 + c\tau_n)$ |
| 6 | What forward artifact does linear yield interpolation produce? | Saw-tooth pattern (discontinuous forward jumps). |
| 7 | What forward artifact does log-linear interpolation produce? | Staircase pattern (piecewise constant forwards). |
| 8 | What is the relationship $f(T) = y(T) + Ty'(T)$ called? | The link between zero rates and instantaneous forwards. |
| 9 | What is curve "ringing"? | Spurious oscillations from non-local spline perturbations. |
| 10 | Which spline type risks ringing? | Natural cubic ($C^2$) splines without tension. |
| 11 | What does the tension parameter $\sigma$ control? | The tradeoff between $C^2$ cubic smoothness and linear-spline locality. |
| 12 | What happens as $\sigma \to 0$ in a tension spline? | Recovers the ordinary $C^2$ cubic spline. |
| 13 | What happens as $\sigma \to \infty$ in a tension spline? | Approaches a piecewise linear (bootstrap) spline. |
| 14 | What is the "par-point approach"? | Bumping each benchmark yield and measuring portfolio sensitivity. |
| 15 | Why might a desk prefer smoothing over exact fit? | To avoid destabilizing the curve due to noisy or erroneous data. |
| 16 | What does the penalized least-squares norm include? | Pricing error + smoothness penalty + curve-length penalty. |
| 17 | What type of spline is optimal in the variational sense? | Natural exponential tension spline with knots at cash flow dates. |
| 18 | What is "locality" in curve construction? | A bump to quote at time $T$ should only affect values near $T$. |
| 19 | What is the Jacobian method? | Using forward rate deltas to compute optimal hedging portfolios. |
| 20 | Why is poor locality problematic for P&L? | Short-rate moves create spurious P&L in long-dated positions. |

---

## Mini Problem Set

### Problem 1: Deposit Bootstrap
A 6-month deposit rate is 4.00% (simple, ACT/360, assume 180 days).
Calculate the discount factor $P(0.5)$.

### Problem 2: Swap Bootstrap
You know $P(1) = 0.96$. A 2-year annual par swap has a rate of 5.00%.
Calculate $P(2)$.

### Problem 3: Log-Linear Interpolation
Given $P(1) = 0.95$ and $P(2) = 0.90$.
Calculate $P(1.5)$ using log-linear (geometric mean) interpolation.

### Problem 4: Forward Rate Calculation
Using your $P(1.5)$ from Problem 3, calculate the annualized forward rate from 1.0 to 1.5 years. Verify it equals the forward rate from 1.5 to 2.0 years.

### Problem 5: Saw-Tooth Derivation
Suppose $y(1) = 5\%$ and $y(2) = 6\%$ with piecewise linear interpolation.
Write the formula for $f(T)$ on $[1, 2]$ and calculate $f(1.5)$.

### Problem 6: Tension Spline Intuition
Explain qualitatively what happens to the forward curve as the tension parameter $\sigma$ increases from 0 to $\infty$.

### Problem 7: Ringing Identification
A colleague proposes using a 15-knot natural cubic spline to fit 15 swap rates perfectly. What issue would you warn them about?

### Problem 8: Smoothing Tradeoff
If your benchmark set includes a clearly erroneous 5-year quote (50 bps away from adjacent maturities), should you use exact fit or smoothing? Why?

---

## Solution Sketches

**1.** $P(0.5) = 1/(1 + 0.04 \times 0.5) = 1/1.02 = 0.9804$

**2.** From $1 = 0.05 \times P(1) + 1.05 \times P(2)$:
$1 - 0.05 \times 0.96 = 1.05 \times P(2)$
$0.952 = 1.05 \times P(2)$
$P(2) = 0.9067$

**3.** $P(1.5) = \sqrt{P(1) \times P(2)} = \sqrt{0.95 \times 0.90} = 0.9247$

**4.** $F_{1.0 \to 1.5} = (1/0.5) \times (0.95/0.9247 - 1) = 2 \times 0.0274 = 5.48\%$
$F_{1.5 \to 2.0} = (1/0.5) \times (0.9247/0.90 - 1) = 2 \times 0.0274 = 5.48\%$ ✓ (constant, as expected)

**5.** $f(T) = y(T) + T \times y'(T) = [0.05 + (0.06-0.05)(T-1)] + T \times 0.01$
At $T=1.5$: $y(1.5) = 5.5\%$, so $f(1.5) = 0.055 + 1.5 \times 0.01 = 7.0\%$

**6.** At $\sigma = 0$, you get a smooth $C^2$ cubic with potential ringing. As $\sigma$ increases, the curve becomes "stiffer," oscillations are dampened, and locality improves. As $\sigma \to \infty$, the curve approaches piecewise linear, with saw-tooth forwards but excellent locality.

**7.** Ringing: a single-point perturbation (or error) will propagate as decaying oscillations throughout the entire curve, creating spurious risk sensitivities at distant maturities.

**8.** Use smoothing. Forcing exact fit would create wild forward rates around the 5-year point, distorting risk measures for all instruments in that region. A small pricing error (controlled by $\lambda$) is preferable to curve instability.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| "Curve construction boils down to supplementing with assumptions to determine $P(T)$" | Andersen & Piterbarg Vol 1, Ch 6 |
| Bootstrap algorithm steps | Andersen & Piterbarg Vol 1, §6.2.1 |
| "Saw-tooth shape characteristic for piecewise linear yield" | Andersen & Piterbarg Vol 1, §6.2.1.1 |
| Piecewise flat forward formula $f(T) = f(T_i)$ | Andersen & Piterbarg Vol 1, §6.2.1.2 |
| Catmull-Rom Hermite spline for $C^1$ curves | Andersen & Piterbarg Vol 1, §6.2.2 |
| Cubic splines "subject to oscillatory behavior, ringing" | Andersen & Piterbarg Vol 1, §6.2.3 |
| Tension spline formula with $\sinh$ terms | Andersen & Piterbarg Vol 1, §6.2.4 |
| "$\sigma = 0$ recovers cubic, $\sigma \to \infty$ approaches linear" | Andersen & Piterbarg Vol 1, §6.2.4 |
| Tension as "extra knob" for balancing smoothness and locality | Andersen & Piterbarg Vol 1, §6.2.4 |
| Penalized least-squares norm formulation | Andersen & Piterbarg Vol 1, §6.3.1 |
| Optimal curve is a tension spline (Proposition 6.3.1) | Andersen & Piterbarg Vol 1, §6.3.1 |
| Par-point approach for risk sensitivities | Andersen & Piterbarg Vol 1, §6.4.1 |
| Ringing perturbation illustration (Fig 6.7) | Andersen & Piterbarg Vol 1, §6.4.1 |
| Jacobian method for hedging | Andersen & Piterbarg Vol 1, §6.4.3 |
| Bootstrap "used daily by trading desks" | Hull, Ch 4 (§4.7) |
| Linear yield interpolation produces kinks in forwards | Tuckman, Ch 4 |
| "Shortcomings least noticeable in DF, most in forwards" | Tuckman, Ch 4 |
| RMSE as fitting criterion | Tuckman, Ch 4 |

### (B) Reasoned Inference (Derived from A)

- The formula $f(T) = y(T) + Ty'(T)$ follows from differentiating $P(T) = e^{-y(T)T}$.
- The saw-tooth pattern is derived by substituting piecewise-constant $y'(T)$ into the forward formula.
- Locality differences between methods follow from the structure of their matrix systems (diagonal for bootstrap, full for cubic $C^2$).

### (C) Flagged Uncertainties

- **Specific desk choices:** Different institutions may use proprietary spline variants (monotone convex, Hagan-West, etc.). The tension spline is presented as one well-documented choice.
- **Default tension values:** No universally agreed $\sigma$ exists; it depends on the benchmark set and practitioner judgment.
- **Jacobian implementation details:** The full numerical implementation involves choices about hedging instrument weights $\mathbf{W}$, $\mathbf{U}$ that vary by institution.

---

*Chapter 17 verified against: Andersen & Piterbarg (Vol 1, Ch 6), Tuckman (Ch 4), Hull (Ch 4, §4.7).*
