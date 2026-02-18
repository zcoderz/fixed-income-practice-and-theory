# Chapter 17: Curve Construction — Bootstrapping, Interpolation, and the Spline Zoo

---

## Learning Objectives
- Translate benchmark quotes → a curve object → discount factors → PV for cashflows on any date.
- Explain bootstrapping as sequential inversion and identify where interpolation enters.
- Compare interpolation “spaces” (zero rates, discount factors, forwards) and predict artifacts (saw-tooth, staircase, ringing).
- Define and interpret curve risk (par-point vs forward-bucket deltas) with explicit bump object, units, and sign convention.
- Apply basic curve sanity checks (monotone discount factors, plausible forwards, locality under bumps).

## Introduction

Imagine you are trying to draw a map of the ocean floor, but you only have depth measurements at ten specific buoys. You know exactly how deep it is at those ten points, but what is happening in between? Is the floor flat? Does it slope gently? Or is there a sudden trench just barely missing your measurements?

This is the fundamental challenge of **curve construction**. In the fixed income market, we observe prices for a finite set of "benchmark" instruments—a few dozen deposits, futures, and swaps. From these sparse data points, we must infer a continuous **discount factor curve** that allows us to assign a value to cash flows occurring on *any* date, not just the ones where instruments mature.

This is an **inverse problem**. The market gives us the outputs (prices), and we must reverse-engineer the input (a discount function). The problem is inherently **underdetermined**: many different continuous curves can reprice the same finite set of benchmarks. To get discount factors for dates that are not instrument maturities, you must add assumptions—interpolation/extrapolation and sometimes smoothing.

**Why this matters:** The choice of structure—especially the **interpolation method**—is not a neutral implementation detail. Two desks can reprice the same benchmarks but still imply different off-node discount factors and different forward curves. Those differences show up in valuation, hedge ratios, and in the “shape” of curve risk reports. Interpolation artifacts are often least visible in discount factors, more visible in zero rates, and most visible in instantaneous forward rates.

Prerequisites: [Chapter 2 — Time Value, Discount Factors, Replication](chapters/chapter_02_time_value_discount_factors_replication.md), [Chapter 3 — Zero, Forward, Par Rates Triangle](chapters/chapter_03_zero_forward_par_rates_triangle.md), [Chapter 14 — Key-Rate DV01 and Bucket Exposures](chapters/chapter_14_key_rate_dv01_bucket_exposures.md).  
Follow-on: [Chapter 18 — OIS Discounting Curve](chapters/chapter_18_ois_discounting_curve.md), [Chapter 19 — Projection Curves (Multi-Curve)](chapters/chapter_19_projection_curves_libor_sofr_multi_curve.md), [Chapter 22 — Multi-Curve Risk (Jacobians)](chapters/chapter_22_multi_curve_risk_jacobians.md).

> **Desk Reality:** Two systems can reprice benchmarks but disagree on off-node trades because interpolation choices differ.
> **Common break:** “Model difference” appears in valuation reconciliations and daily P&L explain, especially for irregular cashflow schedules.
> **What to check:** Interpolation space (zero rates vs $\ln P$ vs forwards), knot set, and locality under single-quote bumps.

**Roadmap:** This chapter will:

1. **Frame the Inverse Problem** (Section 17.1): Why finite quotes cannot uniquely determine a continuous function.
2. **Master Bootstrapping** (Section 17.2): The industry-standard sequential algorithm for extracting discount factors.
3. **Survey the Interpolation Zoo** (Section 17.3): From piecewise linear yields to cubic splines to tension splines—each method's artifacts and tradeoffs.
4. **Derive Forward Rate Behavior** (Section 17.4): The mathematical reason behind saw-tooth and staircase patterns.
5. **Examine Locality and Ringing** (Section 17.5): The critical question of whether bumping one quote disturbs distant parts of the curve.
6. **Explore Smoothing vs. Exact Fit** (Section 17.6): When to sacrifice precision for stability, with Nelson-Siegel-Svensson as a contrast.
7. **Address Special Calendar Effects** (Section 17.7): Turn-of-year spikes, central bank meetings, and stub rate handling.
8. **Introduce the Jacobian Method** (Section 17.8): Translating forward rate deltas into hedge positions.

We focus here on the mechanics of building a *single* discount curve. The modern multi-curve framework (OIS discounting distinct from projection curves) builds directly on these techniques and is the subject of Chapters 18–19.

---

## 17.1 The Inverse Problem: Finite Quotes, Continuous Curve

### 17.1.1 The Pricing Equation

At its core, valuation is linear algebra. The value of any fixed-income instrument is the sum of its cash flows multiplied by the discount factors for those dates. If instrument $i$ pays cash flows $c_{i,j}$ at times $t_j$, its value $V_i$ is:

$$\boxed{V_i = \sum_{j=1}^{M} c_{i,j} \\, P(t_j)}$$

Here, $P(t)$ is the **discount factor**: the value today of one unit of currency to be received at time $t$. This is the fundamental object we are trying to discover.

If we have $N$ benchmark instruments with cash flows on $M$ distinct dates, we can stack these equations into a matrix system. Let $\mathbf{V} = (V_1, \ldots, V_N)^\top$ be the vector of observed market prices, $\mathbf{P} = (P(t_1), \ldots, P(t_M))^\top$ the vector of discount factors, and $\mathbf{C}$ the $N \times M$ matrix of cash flow coefficients. We are solving:

$$\mathbf{V} = \mathbf{C} \mathbf{P}$$

### 17.1.2 The Underdetermined Nature

The difficulty lies in the dimensions. A curve built from 15 liquid benchmark swaps generates cash flows on approximately 120 distinct dates over 30 years—every semiannual or quarterly coupon. Worse, to price a client's specific deal, we might need a discount factor for a date like 1.42 years, which appears in none of our benchmarks.

We are solving for a continuous function $P(T)$ defined for all $T \ge 0$, but we only have $N$ constraints. This means **you cannot build a curve without an interpolation assumption**. When a trader quotes the "market rate" for a 17-month loan, but there is no 17-month liquidity, they are quoting you their interpolation model, not a traded price.

There are three broad ways people close the gap between finite quotes and a continuous curve:

1. **Add fictitious securities:** Pad the benchmark set using interpolation rules applied to existing instruments. This is ad-hoc and can lead to odd-looking curves.

2. **Parametric/Spline representation:** Assume the yield curve has a specific functional form (e.g., Nelson-Siegel) or is a spline with $N$ knots at benchmark maturities. Solve for the knot values.

3. **Optimization with regularization:** Replace exact pricing with penalized least-squares, allowing the curve to miss quotes slightly in exchange for smoothness.

In liquid rates markets, option 2 (a knot-based representation + bootstrapping) is a common default. In noisier markets (or when you want extra stability), option 3 (regularization/smoothing) becomes more attractive.

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

A generic bootstrap iteration looks like:

1. Let $P(t_j)$ be known for $t_j \le T_{i-1}$, such that prices for benchmark securities $1, \ldots, i-1$ are matched.
2. Make a guess for $P(T_i)$.
3. Use an interpolation rule to fill in $P(t_j)$ for $T_{i-1} t t_j t T_i$.
4. Compute $V_i$ from the now-known values of $P(t_j)$, $t_j \le T_i$.
5. If $V_i$ equals the market value, stop. Otherwise return to Step 2.
6. If $i t N$, set $i = i + 1$ and repeat.

The updating of guesses in Steps 2–5 is handled by a standard one-dimensional root-finder (Newton, secant, bisection). For many vanilla instruments (deposits, par swaps), each bootstrap step can be rearranged into a closed-form expression, so no iteration is needed.

### 17.2.2 Closed-Form Solutions for Standard Instruments

For common instruments, each bootstrap step often has a closed-form solution, avoiding iteration entirely.

**Money Market Deposits**

A deposit earning a simple rate $L$ over time $\tau$ pays $(1 + L\tau)$ at maturity. Its present value must equal the notional (which we normalize to 1). Thus:

$$\boxed{P(\tau) = \frac{1}{1 + L\tau}}$$

**Check (units and limiting cases):** $L$ is a per-year rate, $\tau$ is a year-fraction, so $L\tau$ is dimensionless. Sanity checks: if $L=0$, then $P(\tau)=1$; if $L\gt 0$, then $0\lt P(\tau)\lt 1$ and $P(\tau)$ decreases as $\tau$ increases. If $L\lt 0$ (possible in some regimes), then $P(\tau)\gt 1$; that is not a bug, it is the arithmetic consequence of being paid to borrow.

**Par Swaps**

A standard fixed-for-floating swap is quoted at a fixed rate $c$ that makes the NPV zero. Since the floating leg is worth par at inception (by the standard replication argument), the fixed leg plus principal repayment must also equal 1:

$$1 = c \sum_{j=1}^n \tau_j P(t_j) + P(t_n)$$

If we already know the discount factors for all previous coupons $P(t_1), \ldots, P(t_{n-1})$, we can rearrange to solve for the final discount factor:

$$\boxed{P(t_n) = \frac{1 - c \sum_{j=1}^{n-1} \tau_j P(t_j)}{1 + c \tau_n}}$$

**Check (does the last discount factor look plausible?):** For positive rates, you should typically get $0t P(t_n)t P(t_{n-1})t 1$. If the algebra produces a negative discount factor or a discount factor greater than 1 in a positive-rate environment, it is almost always a unit mistake (e.g., using $c$ in percent instead of decimal, or mixing $\tau$ day-count conventions). A quick internal check is to recompute the fixed-leg PV at the solved $P(t_n)$ and verify it sums to 1 with the final principal term (the “reprice test”).

### 17.2.3 The Stub Rate: Handling the First Period

Before bootstrapping can begin, we need the discount factor from today to the first benchmark maturity. This period—called the **stub**—must be handled specially because there is no instrument that matures exactly at settlement.

Common stub treatments include:
- Extend the first benchmark zero rate/yield flat back to today (a simple extrapolation).
- Bootstrap the stub from overnight funding/OIS instruments, then splice into the rest of the curve.
- Use an instrument whose accrual period spans the stub (common in STIR futures-style builds where the first contract starts after spot).

> **Desk Reality: Stub Rate Errors Propagate Everywhere**
> **Common break:** A sloppy stub choice “pollutes” the short end and then leaks into downstream nodes because the curve is solved sequentially.
> **What to check:** Reprice the shortest instruments and inspect very-short forwards; if the stub looks implausible, fix it before trusting anything further out.

### 17.2.4 Worked Example: Building a Curve from Scratch

Let's construct a simple curve to see the mechanics in action.

**Market Quotes:**
- 3M Deposit: 5.00% (ACT/360)
- 6M Deposit: 5.50% (ACT/360)
- 1Y Deposit: 6.00% (ACT/360)
- 2Y Swap (Annual Pay): 6.50%

**Step 1: The Short End (Deposits)**

We convert rates to discount factors directly. Assuming ACT/360 and exact quarter/half/full year periods:

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
$$= 0.061321 + 0.938679 = 1.0000 \\; \checkmark$$

The bootstrap is successful. We have solved for the nodes: $(0.25, 0.9877), (0.5, 0.9732), (1.0, 0.9434), (2.0, 0.8814)$.

### 17.2.5 Worked Example: Building from Deposits, Futures, and Swaps

In practice, most curves are built from a mix of instruments: overnight/short deposits for the very front, futures for the 3M–3Y region, and swaps for the long end.

**Market Quotes (Simplified):**
- ON Deposit: 4.95% (stub rate)
- 3M SOFR Future: 95.20 (implies 4.80% rate)
- 6M SOFR Future: 95.00 (implies 5.00% rate)
- 2Y Swap: 5.25%
- 5Y Swap: 5.50%

**Step 1: Stub**

The overnight deposit provides $P(1/365) \approx 0.999864$ (negligible). For the period from today to the 3M futures start date (say, 2 weeks), we extend the overnight rate flat.

**Step 2: Futures Strip**

Each futures contract locks in a 3-month rate starting at a specific date. The 3M contract implying 4.80% gives us:

$$P(T_{\text{3M end}}) = P(T_{\text{3M start}}) \times \frac{1}{1 + 0.048 \times 0.25}$$

We chain these forward, each contract building on the previous. Note: for Eurodollar/SOFR futures, a **convexity adjustment** is needed to convert futures rates to forward rates—this is covered in Chapter 24.

**Step 3: Transition to Swaps**

Once we've built the curve through the futures strip (say, to 2 years), we bootstrap the 2Y and 5Y swaps as in the previous example. The key is ensuring the curve is continuous across the transition.

---

## 17.3 The Interpolation Zoo: From Piecewise Linear to Tension Splines

We found the nodes, but what is $P(1.5)$? This is where interpolation enters. The choice of interpolation method determines the shape of the curve between solved points—and more importantly, the shape of the **forward curve**. A useful way to organize methods is by (i) *which object you interpolate* (zero rates $y$, log discount factors $\ln P$, or forwards $f$) and (ii) the smoothness class ($C^0$, $C^1$, $C^2$) of the implied yield/forward curves.

### 17.3.1 $C^0$ Methods: Continuous but Not Smooth

> **Deep Dive: Spline vs. Linear (The Smooth Ride)**
>
> *   **Linear Interpolation**: Connects dots with straight lines. Easy, but the ride is jerky. The "speed" (Forward Rate) jumps instantly at every node.
> *   **Spline Interpolation (Cubic)**: Connects dots with flexible metal strips. The ride is buttery smooth. The "speed" changes gradually.
> *   **Tradeoff**: Splines look better but can "wobble" (Ring) if you hit a bump. Linear is ugly but stable. Many desks use **tension / shape-preserving splines** to balance smoothness and stability.

These methods produce a yield curve that is continuous but has kinks (discontinuous first derivative) at the knot points. The resulting forward curve is discontinuous.

#### Piecewise Linear Yields

The most common bootstrap algorithm assumes that the continuously compounded yield $y(T)$ is a piecewise linear function:

$$y(T) = y(T_i) \frac{T_{i+1} - T}{T_{i+1} - T_i} + y(T_{i+1}) \frac{T - T_i}{T_{i+1} - T_i}, \quad T \in [T_i, T_{i+1}]$$

with $P(T) = e^{-y(T)T}$.

Piecewise linear yields typically imply a discontinuous, “saw-tooth” instantaneous forward curve. We derive this in Section 17.4.

#### Piecewise Flat (Log-Linear) Forwards

An alternative assumes the instantaneous forward rate is piecewise flat:

$$f(T) = f(T_i), \quad T \in [T_{i}, T_{i+1})$$

This corresponds to log-linear interpolation on discount factors:

$$\ln P(T) = (1-w) \ln P(T_i) + w \ln P(T_{i+1})$$

where $w = (T - T_i)/(T_{i+1} - T_i)$.

**Equivalently**, this is geometric mean interpolation:

$$P(T) = P(T_i)^{1-w} \cdot P(T_{i+1})^{w}$$

**Implications:** The forward curve is a "staircase"—constant between nodes, then jumping at each maturity date. The yield curve remains continuous but becomes nonlinear between nodes:

$$y(T) = \frac{1}{T}\left(T_i y(T_i) \frac{T_{i+1}-T}{T_{i+1}-T_i} + T_{i+1} y(T_{i+1}) \frac{T-T_i}{T_{i+1}-T_i}\right)$$

> **Key Clarification:** Log-linear interpolation on discount factors ($\ln P$) is equivalent to piecewise-flat forwards, while linear interpolation on yields produces saw-tooth forwards. These are different methods with different artifacts.

### 17.3.2 Worked Example: Comparing Interpolation Methods

Using our bootstrapped nodes $(1.0, 0.9434)$ and $(2.0, 0.8814)$, compute $P(1.5)$, $y(1.5)$, and $f(1.5)$ under different methods.

**Method A: Linear on Yields**

First, convert discount factors to yields:
- $y(1) = -\ln(0.9434)/1 = 5.83\\%$
- $y(2) = -\ln(0.8814)/2 = 6.31\\%$

Linear interpolation:
$$y(1.5) = 0.5 \times 5.83\\% + 0.5 \times 6.31\\% = 6.07\\%$$

Discount factor:
$$P(1.5) = e^{-0.0607 \times 1.5} = e^{-0.0911} = 0.9129$$

Forward rate (see Section 17.4):
$$f(1.5) = y(1.5) + 1.5 \times y'(1.5) = 6.07\\% + 1.5 \times (6.31\\% - 5.83\\%) = 6.07\\% + 0.72\\% = 6.79\\%$$

**Method B: Log-Linear on Discount Factors**

$$P(1.5) = \sqrt{P(1) \times P(2)} = \sqrt{0.9434 \times 0.8814} = 0.9119$$

Yield:
$$y(1.5) = -\ln(0.9119)/1.5 = 6.14\\%$$

Forward rate (constant on interval):
$$f_{[1,1.5]} = f_{[1.5,2]} = \frac{-\ln(P(2)/P(1))}{1} = -\ln(0.8814/0.9434) = 6.79\\%$$

**Comparison:**

| Method | $P(1.5)$ | $y(1.5)$ | $f(1.5)$ |
|--------|----------|----------|----------|
| Linear Yield | 0.9129 | 6.07% | 6.79% (varies across interval) |
| Log-Linear DF | 0.9119 | 6.14% | 6.79% (constant on interval) |

The difference in $P(1.5)$ is about 10 cents per 100 of notional—small for a single cash flow, but meaningful when aggregated across a large portfolio.

### 17.3.3 $C^1$ Methods: Hermite Splines

To produce a continuous forward curve, we need a yield curve that is at least once differentiable. **Hermite cubic splines** (a common choice is the **Catmull–Rom** variant) estimate derivatives at knot points from finite differences.

For $T \in [T_i, T_{i+1}]$, the Catmull-Rom spline writes:

$$y(T) = \mathbf{D}_i(T)^\top \mathbf{A}_i \begin{pmatrix} y_{i-1} \\ y_i \\ y_{i+1} \\ y_{i+2} \end{pmatrix}$$

where $\mathbf{D}_i(T)$ is a vector of powers of the normalized distance $d_i = (T - T_i)/h_i$, and $\mathbf{A}_i$ is a matrix encoding the Catmull-Rom coefficients.

**Key properties:**
- The yield curve is continuous and has a continuous first derivative.
- The forward curve is continuous but not differentiable at knot points.
- Good **locality**: the price of security $i$ depends only on $y_1, \ldots, y_{i+1}$.
- Requires iteration (not pure sequential bootstrap) since $V_i = F_i(y_1, \ldots, y_{i+1})$, but the iteration converges quickly.

### 17.3.4 $C^2$ Methods: Natural Cubic Splines

For a smooth (differentiable) forward curve, we need a yield curve that is twice differentiable. The **natural cubic spline** satisfies $y''(T_1) = y''(T_N) = 0$ at the boundaries and achieves $C^2$ continuity everywhere.

The spline representation is:

$$y(T) = \frac{(T_{i+1}-T)^3}{6h_i}y''_i + \frac{(T-T_i)^3}{6h_i}y''_{i+1} + (T_{i+1}-T)\left(\frac{y_i}{h_i} - \frac{h_i}{6}y''_i\right) + (T-T_i)\left(\frac{y_{i+1}}{h_i} - \frac{h_i}{6}y''_{i+1}\right)$$

where the second derivatives $y''_i$ are determined by solving a tridiagonal linear system that enforces $C^2$ continuity across all knots.

**The ringing problem:** Natural cubic splines can be **non-local**. Perturbing one benchmark can create slow-decaying oscillations (spurious inflection points) that spread across the curve, because the $C^2$ coefficients are obtained by solving a global linear system.

### 17.3.5 Tension Splines: The Best of Both Worlds

To retain $C^2$ smoothness while controlling locality and stiffness, a common compromise is an **exponential tension spline**. The key modification replaces the pure cubic with a hyperbolic form:

$$y(T) = \left(\frac{\sinh(\sigma(T_{i+1}-T))}{\sinh(\sigma h_i)} - \frac{T_{i+1}-T}{h_i}\right)\frac{y''_i}{\sigma^2} + \left(\frac{\sinh(\sigma(T-T_i))}{\sinh(\sigma h_i)} - \frac{T-T_i}{h_i}\right)\frac{y''_{i+1}}{\sigma^2} + y_i \frac{T_{i+1}-T}{h_i} + y_{i+1} \frac{T-T_i}{h_i}$$

where $\sigma \geq 0$ is the **tension factor**.

Key limits:
- $\sigma = 0$ recovers the ordinary $C^2$ cubic spline.
- As $\sigma \to \infty$, the tension spline approaches a linear spline.

In practice, $\sigma$ is a tuning knob: increase it to damp oscillations and improve locality; decrease it to get a smoother forward curve. The right choice is the one that passes your pricing and risk sanity checks.

> **Analogy: The Flexible Rod**
>
> Think of a tension spline as a flexible rod that you can stiffen by increasing $\sigma$. At $\sigma = 0$, it's maximally flexible (standard cubic, which can wiggle). As $\sigma$ increases, it becomes stiffer—eventually so stiff that it's practically a straight edge (piecewise linear). Most desks pick something in between.

### 17.3.6 Monotone Convex and Shape-Preserving Methods

Some practitioners use **monotone convex** (shape-preserving) interpolation to avoid overshooting artifacts where forwards briefly go negative or become implausibly large.

These methods impose constraints during curve construction to ensure forwards remain economically sensible—particularly important when pricing exotics that are sensitive to the entire forward curve shape.

### 17.3.7 Comparison Table

| Method | Yield Curve | Forward Curve | Locality | Comments |
|--------|-------------|---------------|----------|----------|
| Piecewise Linear $y$ | $C^0$ (kinked) | Discontinuous (saw-tooth) | Excellent | Fast, but poor forwards |
| Log-Linear (Flat $f$) | $C^0$ (curved) | Piecewise flat (staircase) | Excellent | Simple baseline |
| Hermite $C^1$ | $C^1$ (smooth) | Continuous, non-smooth | Good (4 neighbors) | Catmull-Rom common |
| Cubic $C^2$ | $C^2$ (smooth) | Smooth | Poor (global matrix) | Risk of ringing |
| Tension $C^2$ | $C^2$ (smooth) | Smooth | Tunable via $\sigma$ | Best compromise |
| Monotone Convex | $C^1$ | Positive, smooth | Good | Shape-preserving |

> **Desk Reality:** Off-node valuations often disagree across systems because interpolation choices differ (even when both systems reprice the benchmarks).
> **Common break:** Valuation reconciliations and daily P&L explain show “model difference” for irregular schedules or off-the-run maturities.
> **What to check:** Interpolation space ($y$ vs $\ln P$ vs $f$), knot set, and whether single-quote bumps create local (not global) curve moves.

---

## 17.4 The Mathematics of Forward Rate Artifacts

Why do different interpolation methods produce such different forward rate curves? The answer lies in the relationship between yields and instantaneous forwards.

### 17.4.1 The Fundamental Relationship

Recall that the discount factor, zero rate, and instantaneous forward rate are related by:

$$P(T) = e^{-y(T)T} = \exp\left(-\int_0^T f(u)\\, du\right)$$

Differentiating the first equality:

$$\boxed{f(T) = y(T) + T y'(T)}$$

This is the key equation. The forward rate equals the zero rate plus a term proportional to the *slope* of the zero rate curve times the maturity.

**Check (directionality):** If the zero curve is locally flat ($y'(T)=0$), then $f(T)=y(T)$. If the zero curve is locally upward sloping ($y'(T)gt 0$), then $f(T)gt y(T)$; if the zero curve is downward sloping ($y'(T)t 0$), then $f(T)t y(T)$. This is a useful mental model: forwards “amplify” the local slope of the zero curve by a factor of $T$.

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
> *   This is why many pricing/risk systems avoid simple linear yield interpolation when valuing instruments sensitive to the forward curve shape (e.g., options).

### 17.4.3 Worked Example: The Boundary Jump

Using $y(1) = 5.83\\%$ and $y(2) = 6.31\\%$:

**At $T = 1.5$ (middle of interval):**

$$y(1.5) = 6.07\\%$$
$$y'(1.5) = \frac{6.31\\% - 5.83\\%}{1} = 0.48\\%$$
$$f(1.5) = 6.07\\% + 1.5 \times 0.48\\% = 6.79\\%$$

**At $T = 2^-$ (just before node):**

$$y(2^-) = 6.31\\%$$
$$f(2^-) = 6.31\\% + 2.0 \times 0.48\\% = 7.27\\%$$

**At $T = 2^+$ (just after node):**

Suppose the next segment has $y(3) = 6.70\\%$, implying slope $= 0.39\\%$. Then:

$$f(2^+) = 6.31\\% + 2.0 \times 0.39\\% = 7.09\\%$$

**The jump:** $f(2^-) = 7.27\\%$ but $f(2^+) = 7.09\\%$. The forward rate drops by 18 bps at the node—this is the "tooth" of the saw.

### 17.4.4 The Staircase Pattern

With log-linear interpolation (piecewise flat forwards):

$$\ln P(T) = \ln P(T_i) - f_i (T - T_i)$$

where $f_i$ is the constant forward rate on $[T_i, T_{i+1})$. By definition, $f(T) = -\frac{d}{dT}\ln P(T) = f_i$, which is constant on each interval.

At each node, the forward rate jumps to a new constant level, creating a staircase. While unrealistic, this method has the advantage of being extremely stable and local.

---

## 17.5 Locality, Perturbation, and Ringing

When you choose an interpolation method, you aren't just picking a shape; you are picking a **risk profile**. A critical property is **locality**: does a bump to a 2-year benchmark mostly affect the 1–3 year region, or does it “leak” into the long end?

### 17.5.1 Curve Risk Conventions (What Is Being Bumped?)

There is no single “DV01 of the curve” until you specify the **bump object**.

**Book conventions used in this chapter**
- **Bump size:** $1\text{bp} = 10^{-4}$ in rate units.
- **Sign:** $\displaystyle DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$. For long fixed-income PV, DV01 is typically positive.
- **Units:** report in currency per 1bp for the stated portfolio (and state the notional / scaling).

Two common bump objects in curve work are:

1. **Par-point DV01 (benchmark bump):** pick benchmark quote $q_i$ (deposit rate, futures-implied rate, par swap rate, etc.). Shift that quote **down** by 1bp, rebuild the curve using the same construction settings, then reprice. This produces a vector $\\{DV01_i\\}$ in currency per 1bp.

2. **Forward-bucket DV01 (functional bump):** pick a maturity bucket $[t_k,t_{k+1})$ and shift the instantaneous forward curve down by 1bp on that bucket (keeping other buckets unchanged), then reprice. This produces $\\{DV01^{(f)}_k\\}$ and cleanly answers “which part of the time axis is driving PV?”

> **Pitfall — What is being bumped?:** Mixing par-quote bumps, zero-node bumps, and forward-bucket bumps without stating which one you used.
> **Why it matters:** Hedge ratios and risk attribution differ across bump designs even when the base curve reprices the same benchmarks.
> **Quick check:** Take one instrument and compute sensitivity two ways (rebuild-the-curve vs forward-bucket shift) and confirm you understand why the numbers differ.

### 17.5.2 Par-Point Deltas and the Locality Requirement

Let $V_0$ be the PV of the portfolio you care about, and let $V_i$ be the PV/price of benchmark instrument $i$ used in the curve build. The simplest approach to computation of the delta $\partial V_0/\partial V_i$ involves a manual bump to $V_i$. Then reconstruct the yield curve and reprice the portfolio $V_0$ (a finite-difference approximation to $\partial V_0/\partial V_i$). This procedure is sometimes known as the par-point approach, and the resulting derivatives are par-point deltas.

In practice, the “bump” is often implemented as a 1bp shift to the benchmark’s quoted rate (or price), followed by a rebuild with the *same* interpolation and smoothing settings. That means par-point deltas are not just “about the market quotes”; they also depend on the construction choices that govern how a local change in one input propagates to off-node discount factors and forwards.

Two common ways to summarize curve risk are (i) **factor-based** deltas (e.g., principal components or stylized curve moves) and (ii) **quote-based** deltas (sensitivities to the instruments used to compute the curve). In practice, traders tend to prefer the quote-based view. They argue that the only way the zero curve can change is if the quote for one of the instruments used to compute the zero curve changes. They therefore feel that it makes sense to focus on the exposures arising from changes in the prices of these instruments.

For instance, perturbing a short-dated FRA price should not cause noticeable movements in long-term yields. Otherwise a risk report can suggest nonsensical hedges (e.g., “hedge a 20Y swap with a 1M instrument”).

### 17.5.3 Worked Example: Forward-Bucket DV01 for a Broken-Date Cashflow

**Example Title**: Bucket DV01 for a single cashflow (piecewise-flat forwards)

**Context**
- Price and risk a single future cashflow using a piecewise-flat instantaneous forward curve.
- This mirrors how “bucket risk” allocates PV sensitivity across maturity segments.

**Timeline (Make Dates Concrete)**
- Valuation date: 2026-02-15
- Payment date: 2027-08-15

**Inputs**
- Cashflow: receive $CF = +1{,}000{,}000$ USD on 2027-08-15
- Day count: ACT/365 (toy convention for transparency)
- Curve object: instantaneous forward $f(t)$ is piecewise-flat (continuous compounding)
  - $f(t)=5.00\\%$ for $[0,1Y)$
  - $f(t)=5.50\\%$ for $[1Y,2Y)$

**Outputs (What You Produce)**
- Discount factor $P(0,T)$
- PV
- Forward-bucket $DV01_{[1Y,2Y)}$ in USD per 1bp (book sign convention)

**Step-by-step**
1. Year fractions (ACT/365):
   - $\tau_{0\to 1Y} = 365/365 = 1.0000$
   - $\tau_{1Y\to T} = 181/365 \approx 0.4959$ (2027-02-15 to 2027-08-15)
2. Discount factor under piecewise-flat forwards:
   $$P(0,T)=\exp\left(-0.0500\cdot 1.0000 -0.0550\cdot 0.4959\right) \approx 0.92564$$
3. PV:
   $$PV = CF\cdot P(0,T) \approx 1{,}000{,}000\times 0.92564 = 925{,}636$$
4. Bucket DV01 for the $[1Y,2Y)$ forward segment:
   - Bump object: $f(t)$ for $t\in[1Y,2Y)$
   - Bump size: down 1bp = $-10^{-4}$
   - Exact reprice (by bumping $f_1$ only):
     $$DV01_{[1Y,2Y)} = PV(f_1-10^{-4}) - PV(f_1) \approx 45.9$$
   - First-order check:
     $$DV01_{[1Y,2Y)} \approx CF\cdot P(0,T)\cdot \tau_{1Y\to T}\cdot 10^{-4} \approx 45.9$$

**Cashflows (table)**
| Date | Cashflow | Explanation |
|---|---|---|
| 2027-08-15 | +1,000,000 | single payment |

**P&L / Risk Interpretation**
- “USD 45.9 per bp” means: if forwards between 1Y and 2Y fall by 1bp (with other buckets unchanged), PV increases by about USD 45.9.
- The bucket allocation is time-local: only the portion of the discount integral inside the bucket contributes (here $\tau_{1Y\to T}$).

**Sanity Checks**
- Units: $CF \times P \times \tau \times 10^{-4}$ is currency.
- Sign: rates down increases PV for a receivable cashflow, so DV01 is positive.
- Limit: if the payment date moved to before 1Y, $DV01_{[1Y,2Y)}\to 0$.

### 17.5.4 Ringing: When Smoothness Becomes Non-Local

Global smoothness conditions (e.g., some $C^2$ spline choices) can create **non-local perturbations**: a single-quote bump produces oscillations that spread into distant forwards and deltas. This is a property of the interpolation algorithm, not a market signal.

**Check (a fast ringing diagnostic):** Pick one short-dated benchmark quote, bump it by 1bp, rebuild the curve, and plot (or tabulate) the change in instantaneous forwards across the whole maturity axis. If a short bump produces alternating “up/down” ripples far out the curve (or materially moves long-dated forwards), you are seeing spline non-locality. This matters because a par-point risk report can then suggest implausible hedges (e.g., “hedge a 20Y swap with a 1M instrument”) even though the base curve reprices perfectly.

### 17.5.5 Managing the Trade-Off

- If you need locality (risk reports and hedging), prefer methods designed to be local (bootstrapped $C^0$, Hermite $C^1$, or tension splines with enough tension).
- If you need very smooth forwards (some option models), you may accept less locality—but you should test bump response, and consider shape-preserving or tension variants.

> **Desk Reality:** Some organizations separate “pricing curves” (very smooth) from “risk curves” (very local).
> **Common break:** Daily P&L changes are not well explained by first-order deltas if pricing and risk are computed off different curve objects.
> **What to check:** Confirm pricing and risk use the same curve build and bump design; if not, document the expected explain breaks.

---

## 17.6 Smoothing, Exact Fit, and Parametric Alternatives

So far, we have assumed the curve must hit every market price exactly. This is the **exact fit** approach. But market data is noisy.

### 17.6.1 Sources of Noise

- **Bid-ask spreads:** The "mid" price is itself an interpolation.
- **Stale quotes:** Some prices may be hours old.
- **Data entry errors:** Typos happen.
- **Illiquid instruments:** Prices may not reflect executable levels.

If you force a curve to pass exactly through a "bad" quote (e.g., a typo where 2.15% is entered as 2.51%), the curve will dip violently to hit that point and then rip back up to the next valid point. This creates wild forward rates and massive, incorrect hedge ratios.

> **Desk Reality:** Most curve builds include quote hygiene checks before construction or calibration.
> **Common break:** One bad/stale quote forces an exact-fit curve to contort, creating wild forwards and unstable risk.
> **What to check:** Compare each quote to adjacent tenors and recent levels; inspect implied discount factors/forwards for spikes or sign violations; down-weight or exclude outliers before you “fix it with smoothing”.

### 17.6.2 The Penalized Least-Squares Approach

One common approach is to fit a curve by penalized least squares:

$$\hat{y} = \underset{y \in \mathcal{A}}{\text{argmin}} \\; \mathcal{I}(y)$$

where

$$\boxed{\mathcal{I}(y) = \frac{1}{N}(\mathbf{V} - \mathbf{C}\mathbf{P}(y))^\top \mathbf{W}^2 (\mathbf{V} - \mathbf{C}\mathbf{P}(y)) + \lambda \int_{t_1}^{t_M} \left[y''(t)^2 + \sigma^2 y'(t)^2\right] dt}$$

The norm consists of:

1. **Pricing error penalty:** How far the model price is from the market quote, weighted by $\mathbf{W}$.
2. **Smoothness penalty:** $\lambda \int y''(t)^2 \\, dt$ penalizes high curvature.
3. **Curve-length penalty:** $\lambda \sigma^2 \int y'(t)^2 \\, dt$ penalizes oscillations.

The parameter $\lambda$ controls the tradeoff between fitting prices and smoothness.

### 17.6.3 Why Tension Splines Reappear Here

Under standard smoothness assumptions, the minimizer of the penalized objective can be characterized as an exponential tension spline (with knots at cashflow dates). This is one reason tension splines are a common “middle ground”: they arise naturally from a smoothing objective rather than being a purely ad-hoc interpolation choice.

### 17.6.4 Choosing Lambda and RMSE Targets

If $\lambda$ is high, you get a smooth curve that might miss prices by a fraction of a basis point. If $\lambda$ is low, you get a curve that hits every price but may wiggle unreasonably.

Define the root mean square error (RMSE) of fitted yields (or fitted benchmark quotes) as:

$$\text{RMSE} = \sqrt{\frac{1}{N} \sum_{i=1}^{N} (y_i - \hat{y}_i)^2}$$

RMSE is a compact way to summarize “typical” fit error in the same units as your inputs (often basis points if you measure errors in bp).

**How to choose $\lambda$ / an RMSE target (practical, convention-light)**
- Choose weights $\mathbf{W}$ that reflect quote uncertainty (bid/ask, staleness, liquidity).
- Increase smoothing until the curve passes sanity checks (monotone discount factors, plausible forwards) and the risk response is stable under small bumps.
- Avoid smoothing as a substitute for bad data: remove or down-weight obvious outliers first, then smooth what remains.

A practical approach replaces the unconstrained problem with a constrained one:

$$\min_{y} \int \left[y''(t)^2 + \sigma^2 y'(t)^2\right] dt \quad \text{subject to} \quad \text{RMSE} = \gamma$$

where $\gamma$ is the allowed root-mean-square pricing error.

### 17.6.5 Nelson-Siegel-Svensson: A Low-Parameter Alternative

Parametric functional forms (e.g., Nelson–Siegel and Svensson extensions) model the entire yield curve with a small number of parameters. The Nelson–Siegel model expresses the yield curve as:

$$y(T) = \beta_0 + \beta_1 \frac{1 - e^{-T/\tau}}{T/\tau} + \beta_2 \left(\frac{1 - e^{-T/\tau}}{T/\tau} - e^{-T/\tau}\right)$$

The Svensson extension adds another curvature term (more parameters, more flexibility).

**Trade-offs vs spline/bootstrapped curves**
1. **Fit:** A low-parameter curve may not match a large set of benchmark quotes tightly, especially if the curve has local features.
2. **Locality:** Changing one parameter generally affects the entire curve, so bumping “one tenor” is not localized.
3. **Micro-features:** Local calendar effects (turns, event dates) are hard to represent without adding explicit knots/overlays.

In practice, these forms are often most useful as smooth priors, for extrapolation, or for macro-style summaries—while trading curves still need a construction that tightly reprices the chosen benchmark set and has predictable bump behavior.

---

## 17.7 Special Calendar Effects: Turns, Meetings, and Stubs

The standard bootstrapping discussion treats maturities as “tenors” (1W, 1M, 3M, …). In reality, **specific calendar dates** can create localized discontinuities in very short rates. If you smooth through a true calendar discontinuity, you can unintentionally contaminate the rest of the curve.

### 17.7.1 Turns (Localized Calendar Premiums)

Many curve construction algorithms are designed around the implicit idea that the forward curve should ideally be smooth, but the **turn-of-year (TOY)** effect is a well-known exception: short-dated loan premiums can spike for loans between the last business day of the year and the first business day of the following calendar year.

One common way of incorporating TOY-type effects is to exogenously specify an **overlay** curve $\varepsilon_{f}(t)$ on the instantaneous forward curve. Specifically, the forward curve $f(t)=f(0,t)$ is written as $f(t)=\varepsilon_{f}(t)+f^{\ast}(t)$:

$$\boxed{f(t)=f(0,t)=\varepsilon_{f}(t)+f^{\ast}(t)}$$

where $\varepsilon_{f}(t)$ is user-specified (and most likely contains discontinuities around special event dates) and $f^{\ast}(t)$ is the unknown curve to be fitted. The yield curve algorithm is then applied to the construction of $f^{\ast}(t)$. Under continuous compounding:

$$P(T)=e^{-\int_0^T f(u)\\,du}=e^{-\int_0^T \varepsilon_{f}(u)\\,du}\\,e^{-\int_0^T f^{\ast}(u)\\,du}\triangleq P_{\varepsilon}(T)\\,P^{\ast}(T).$$

Once the curve $P^{*}(t)$ is constructed, any subsequent use of the curve for cash flow discounting requires a multiplicative adjustment of time-$t$ discount factors by the quantity $P_{\varepsilon}(t)$.

**Check (toy magnitude):** Suppose you model a short “turn” window as an overlay of $+200$bp for 3 calendar days and zero elsewhere (purely illustrative). Under continuous compounding, the multiplicative factor for any maturity beyond the window is approximately $P_{\varepsilon}\approx e^{-0.02\times 3/365}\approx 0.999836$. This is a $0.0164\\%$ PV hit (about 0.0164 price points per 100) applied uniformly to all cashflows beyond the window. The point of the overlay is not that it is “large” in PV terms, but that it localizes a calendar premium to the short window rather than forcing the fitted curve to twist in unrelated maturities.

### 17.7.2 Event Dates and Step-Like Forwards

For overnight-indexed curves, scheduled policy meetings and other known event dates can justify allowing **step changes** in the short forward curve. A practical pattern is to include the event dates as knots and allow piecewise-constant (or piecewise-smooth) forwards between them.

### 17.7.3 Stub Handling Without “Polluting” the Short End

The stub from settlement to the first benchmark (Section 17.2.3) should be specified so it does not inadvertently absorb a turn or event-date premium. Common treatments are: (i) flat extrapolation of the first benchmark rate, (ii) bootstrapping from overnight/OIS instruments, or (iii) using a stub-spanning quote if one exists.

> **Desk Reality:** Calendar effects often show up as a localized “bump” in overnight forwards; treating it as a global level shift creates confusing valuation/risk.
> **Common break:** The curve prices benchmarks but produces implausible overnight forwards immediately after an event window, leading to recon breaks on short-dated trades.
> **What to check:** Plot the very-short forward curve around the event dates; ensure the feature is localized and that bumping a short quote does not move the long end.

---

## 17.8 The Jacobian Method: From Forward Deltas to Hedges

Par-point deltas bundle “curve construction + risk” into one number: you bump a benchmark, rebuild, and reprice. An alternative is to define shocks directly on the **forward curve**, compute **forward-bucket DV01s**, and then map those bucket DV01s to hedge instruments.

### 17.8.1 Forward-Bucket Shocks and Deltas

Choose basis functions $\mu_k(t)$ that localize shocks in maturity:

$$f(t) \mapsto f(t) - \varepsilon \mu_k(t)$$

Common choices include:
- **Piecewise flat:** $\mu_k(t)=1$ for $t\in[t_k,t_{k+1})$, and $\mu_k(t)=0$ otherwise.
- **Triangular:** Rises linearly to a peak at $t_k$, then falls linearly.

Define the **forward-bucket DV01** (book convention) as:

$$DV01^{(f)}_k := PV\\!\left(f - 10^{-4}\mu_k\right) - PV(f)$$

Units are currency per 1bp for the stated portfolio. For a long PV, $DV01^{(f)}_k$ is typically positive.

### 17.8.2 The Jacobian Method

Let:
- $\mathbf{d}\in\mathbb{R}^K$ be the portfolio’s forward-bucket DV01 vector (currency per 1bp).
- $\mathbf{J}\in\mathbb{R}^{K\times L}$ be the matrix of hedging-instrument bucket DV01s per unit notional (or per USD 1mm notional, as stated).
- $\mathbf{p}\in\mathbb{R}^L$ be the hedge notionals.

To hedge, we want $\mathbf{J}\mathbf{p}\approx -\mathbf{d}$. A common weighted/regularized formulation is:

$$\hat{\mathbf{p}} = \underset{\mathbf{p}}{\text{argmin}} \left\\|\mathbf{W}(\mathbf{J}\mathbf{p}+\mathbf{d})\right\\|^2 + \left\\|\mathbf{U}\mathbf{p}\right\\|^2$$

Written in terms of $\mathbf{d}$ and $\mathbf{J}$, the normal equations become:

$$(\mathbf{J}^\top \mathbf{W}^2 \mathbf{J} + \mathbf{U}^2)\hat{\mathbf{p}} = -\mathbf{J}^\top \mathbf{W}^2 \mathbf{d}$$

Interpretation: $\mathbf{W}$ prioritizes matching some buckets more than others; $\mathbf{U}$ discourages overly large notionals (a ridge penalty).

**Check (dimensions and units):** $\mathbf{J}\mathbf{p}$ lives in bucket space ($\mathbb{R}^K$), so it can be compared directly to $-\mathbf{d}$. A practical way to keep units straight is:
- $\mathbf{d}$: currency per bp (per portfolio)
- $\mathbf{J}$: currency per bp **per unit notional** of each hedge instrument (state the unit, e.g., per $USD 1$mm)
- $\mathbf{p}$: hedge notionals (in the same units used for $\mathbf{J}$)

After solving, compute the residual bucket vector $\mathbf{r}=\mathbf{J}\hat{\mathbf{p}}+\mathbf{d}$. If $\mathbf{r}$ is large in a particular bucket, the hedge instruments you allowed cannot span that exposure (or you intentionally down-weighted it via $\mathbf{W}$).

### 17.8.3 Worked Example: Simple Hedge Calculation

**Setup:** A portfolio has forward-bucket DV01s $(+5, +3, -2, -6)$ across 1Y, 2Y, 3Y, 5Y buckets (in thousands of currency per bp, using the “rates down” convention). Available hedges are 2Y and 5Y swaps.

**Step 1: Hedge Instrument Deltas**

A 2Y swap has approximate deltas $(+0.5, +1.5, 0, 0)$ per USD 1mm notional.
A 5Y swap has approximate deltas $(+0.2, +0.4, +0.6, +3.8)$ per USD 1mm notional.

**Step 2: Set Up System**

Let $p_1$ be the 2Y swap notional and $p_2$ be the 5Y swap notional, measured in USD millions (so $p_1=-1$ means short USD 1mm of 2Y swaps).

We want to match portfolio deltas as closely as possible:

At 1Y: $0.5 p_1 + 0.2 p_2 \approx -5$ (negative because we're hedging)
At 2Y: $1.5 p_1 + 0.4 p_2 \approx -3$
At 3Y: $0 p_1 + 0.6 p_2 \approx +2$
At 5Y: $0 p_1 + 3.8 p_2 \approx +6$

**Step 3: Solve**

This is overdetermined (4 equations, 2 unknowns). We solve a least-squares hedge:

$$\mathbf{J} = \begin{pmatrix} 0.5 & 0.2 \\ 1.5 & 0.4 \\ 0 & 0.6 \\ 0 & 3.8 \end{pmatrix}, \quad \mathbf{d} = \begin{pmatrix} -5 \\ -3 \\ 2 \\ 6 \end{pmatrix}$$

$$\hat{\mathbf{p}} = (\mathbf{J}^\top \mathbf{J})^{-1} \mathbf{J}^\top \mathbf{d}$$

Solving gives approximately $p_1 \approx -3.25$ and $p_2 \approx +1.60$ (USD millions).

The hedge is: short $USD 3.25$mm 2Y swaps and long $USD 1.60$mm 5Y swaps.

---

## Summary

1. Curve construction maps benchmark quotes $\to$ a curve object $\to$ discount factors $\to$ PV for cashflows on any date.
2. Bootstrapping is sequential inversion; interpolation is the extra assumption that defines off-node values and (therefore) implied forwards.
3. Interpolating in different “spaces” creates different forward artifacts: linear $y(T)$ $\to$ saw-tooth $f(T)$; log-linear $P(0,T)$ $\to$ piecewise-flat $f(T)$; very smooth $C^2$ choices $\to$ smooth forwards but potential non-local behavior.
4. Risk depends on the bump object: par-point DV01 bumps a benchmark quote and rebuilds; forward-bucket DV01 bumps the forward curve on a maturity bucket.
5. Always state bump size ($1\text{bp}=10^{-4}$), units (currency per 1bp for the stated notional/portfolio), and sign ($DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$).
6. Locality matters: a short-dated bump should not materially move far-dated forwards; “ringing” produces misleading hedge ratios and P&L explain.
7. Smoothing/regularization trades fit vs stability; use weights to reflect quote quality and avoid smoothing as a substitute for bad data.
8. Calendar features (turns, event dates, stubs) are best treated as localized knots/overlays so they do not “pollute” the rest of the curve.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---|---|---|
| Discount factor $P(0,T)$ | PV of 1 unit paid at $T$ | Primary curve object used for PV |
| Zero rate $y(T)$ | $P(0,T)=e^{-y(T)T}$ under continuous compounding | A convenient “rate” view; interpolation here affects forwards |
| Instantaneous forward $f(T)$ | $f(T)=-\\frac{d}{dT}\\ln P(0,T)$ | Forward shape drives option inputs and some risk attribution |
| Bootstrapping | Solve curve nodes short $\to$ long to match benchmarks | Standard construction; clear dependency chain |
| Benchmark set | Chosen quotes/instruments used to pin the curve | Defines what the curve “is” (and what it isn’t) |
| Interpolation space | The object you interpolate ($y$, $\ln P$, $f$, …) | Different choices $\Rightarrow$ different artifacts and hedges |
| Piecewise-linear $y(T)$ | $y$ is linear between knots | Typically implies saw-tooth instantaneous forwards |
| Log-linear $P(0,T)$ | $\ln P$ linear between knots | Equivalent to piecewise-flat forwards (staircase $f$) |
| Locality | A bump mainly affects nearby maturities | Needed for stable, interpretable risk reports |
| Ringing | Non-local oscillations after a local bump | Creates spurious far-dated deltas and confusing P&L explain |
| Tension spline | $C^2$ spline with stiffness parameter $\sigma$ | Tunable compromise between smoothness and locality |
| Smoothing / regularization | Allow small mispricings to stabilize the curve | Prevents overfitting noisy quotes; improves risk stability |
| Bump object | What is perturbed (quote, node, forward bucket) | Different bump objects produce different “DV01” vectors |
| Par-point DV01 | Rebuild-the-curve sensitivity to one benchmark quote | Natural for benchmark hedging; depends on build choices |
| Forward-bucket DV01 | Sensitivity to shifting forwards on a time bucket | Time-local risk; convenient for Jacobian hedging |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $P(0,T)$ | discount factor to $T$ | unitless; positive |
| $y(T)$ | zero rate (continuous comp in this chapter) | 1/year |
| $f(T)$ | instantaneous forward rate | 1/year; $f(T)=-\\frac{d}{dT}\\ln P(0,T)$ |
| $\tau$ | year fraction | day-count dependent (ACT/365 in toy example) |
| $1\\text{bp}$ | one basis point | $10^{-4}$ in rate units |
| $q_i$ | benchmark quote $i$ | rate or price; definition must be stated |
| $DV01_i$ | par-point DV01 for quote $q_i$ | currency per 1bp; $PV(\\text{rates down})-PV(\\text{base})$ |
| $DV01^{(f)}_k$ | forward-bucket DV01 for bucket $k$ | currency per 1bp; bucket bump on $f(t)$ |
| $T_i$ | knot / benchmark maturity $i$ | years from valuation date |
| $h_i$ | interval length $T_{i+1}-T_i$ | years |
| $\mu_k(t)$ | bucket basis function | unitless; localized in time |
| $\sigma$ | tension parameter | controls cubic $\leftrightarrow$ linear limit |
| $\lambda$ | smoothing strength | larger $\lambda\Rightarrow$ smoother, less exact fit |
| $\mathbf{C}$ | cashflow matrix | currency per unit notional |
| $\mathbf{V}$ | observed benchmark prices | currency |
| $\mathbf{W}$ | quote-quality weights | often inverse-uncertainty |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the fundamental unknown in curve construction? | The discount factor function $P(0,T)$ (or an equivalent representation like $y(T)$ or $f(T)$). |
| 2 | Why is the curve problem underdetermined? | A finite set of benchmark quotes cannot uniquely determine a continuous function; interpolation/smoothing closes the gap. |
| 3 | What is bootstrapping? | Solving curve nodes sequentially (short $\to$ long) so each benchmark reprices. |
| 4 | What formula gives $P(\tau)$ from a simple deposit rate $L$? | $P(\tau)=\\frac{1}{1+L\\tau}$. |
| 5 | What formula gives $P(t_n)$ from a par swap rate $c$ (when earlier $P(t_j)$ are known)? | $P(t_n)=\\frac{1-c\\sum_{j=1}^{n-1}\\tau_j P(t_j)}{1+c\\tau_n}$. |
| 6 | What does “interpolation space” mean? | Which object you interpolate (e.g., $y(T)$, $\ln P(0,T)$, or $f(T)$). |
| 7 | What forward artifact does linear yield interpolation typically produce? | A discontinuous, saw-tooth instantaneous forward curve. |
| 8 | What forward artifact does log-linear discount-factor interpolation produce? | Piecewise-flat (staircase) instantaneous forwards. |
| 9 | Under continuous compounding, what links zero rates and forwards? | $f(T)=y(T)+T y'(T)$. |
| 10 | What is locality? | A bump to one quote mainly affects nearby maturities (not the whole curve). |
| 11 | What is ringing? | Non-local oscillations in the curve (and deltas) caused by global smoothness constraints. |
| 12 | What does the tension parameter $\sigma$ control? | The trade-off between $C^2$ smoothness and locality (as $\sigma\to\infty$ the curve approaches piecewise linear). |
| 13 | What must you state to make a DV01 meaningful? | Bump object, bump size, units, and sign convention. |
| 14 | What is the book DV01 sign convention? | `DV01 := PV(rates down 1bp) - PV(base)` |
| 15 | What is par-point DV01? | Sensitivity computed by bumping one benchmark quote and rebuilding the curve. |
| 16 | What is forward-bucket DV01? | Sensitivity to shifting the forward curve down by 1bp on a maturity bucket (with other buckets unchanged). |
| 17 | Desk: Two systems reprice benchmarks but disagree on an off-node PV. What is the first thing to compare? | Interpolation space ($y$ vs $\ln P$ vs $f$), knot set, and short-end stub treatment. |
| 18 | Desk: A 1bp bump to a short quote generates big long-end deltas. What might be going on? | Non-locality/ringing (or an inconsistent bump design). Test bump response and consider more local or tension/shape-preserving methods. |
| 19 | What does RMSE summarize in a smoothed fit? | The typical size of pricing/yield errors across benchmarks (in the units you measure errors). |
| 20 | What is the Jacobian method used for? | Mapping forward-bucket DV01s into hedge instrument notionals via a (weighted) linear system. |

---

## Mini Problem Set

1. (Compute) A 6-month deposit rate is 4.00% (simple, ACT/360, assume 180 days). Compute $P(0.5)$.
2. (Compute) You know $P(1)=0.96$. A 2-year annual par swap has fixed rate 5.00%. Compute $P(2)$.
3. (Compute) Given $P(1)=0.95$ and $P(2)=0.90$, compute $P(1.5)$ using log-linear (geometric mean) interpolation.
4. (Compute) Using your $P(1.5)$ from (3), compute the annualized forward rate from 1.0 to 1.5 years. Verify it equals the forward from 1.5 to 2.0 years under log-linear $P$.
5. (Compute) Use the setup of Section 17.5.3 (ACT/365; $CF=+1{,}000{,}000$ on 2027-08-15; piecewise-flat forwards $f_0=5.00\\%$ on $[0,1Y)$ and $f_1=5.50\\%$ on $[1Y,2Y)$). Compute PV and $DV01_{[1Y,2Y)}$.
6. (Compute) Suppose $y(1)=5\\%$ and $y(2)=6\\%$ with piecewise linear interpolation. Write $f(T)$ on $[1,2]$ and compute $f(1.5)$ and $f(2^-)$.
7. (Concept) Explain qualitatively what happens to forward curves as the tension parameter $\sigma$ increases from 0 to $\infty$.
8. (Desk) A colleague proposes a natural cubic spline that exactly fits 15 swap rates. What risk-report behavior might surprise them after a 1bp bump to one quote?
9. (Desk) Your benchmark set has one clearly suspicious quote. Why is “exact fit” risky, and what should you do before turning on smoothing?
10. (Concept) Two banks reprice the same benchmark swaps but quote different rates for an off-node maturity. Explain why neither quote is “the” market rate.
11. (Desk) A short end curve shows an obvious event-date discontinuity. How should you represent it so it does not contaminate the rest of the curve?
12. (Concept) Give two reasons a low-parameter functional form (e.g., Nelson–Siegel) can be problematic for trading-curve hedging.
13. (Compute) A portfolio has forward-bucket DV01s (rates down 1bp) of $(+10,-5)$ at 2Y and 5Y buckets (in thousands per bp). A 2Y swap has bucket DV01 loadings $(+2,0)$ per USD 1mm notional; a 5Y swap has $(+0.5,+4)$ per USD 1mm notional. Find hedge notionals $(p_1,p_2)$ in USD mm that offset the portfolio DV01s.

### Solution Sketches (Selected)
1. $P(0.5)=\frac{1}{1+0.04\times 0.5}=0.9804$.
2. $1=0.05P(1)+1.05P(2)\Rightarrow P(2)=\frac{1-0.05\times 0.96}{1.05}=0.9067$.
3. $P(1.5)=\sqrt{0.95\times 0.90}=0.9247$.
4. $F_{1.0\to 1.5}=\frac{1}{0.5}\left(\frac{P(1)}{P(1.5)}-1\right)=5.48\\%$. Under log-linear $P$, the forward is constant on $[1,2]$, so $F_{1.5\to 2.0}=5.48\\%$ as well.
5. From Section 17.5.3, $PV\approx 925{,}636$. Bucket DV01: $DV01_{[1Y,2Y)}=PV(f_1-10^{-4})-PV(f_1)\approx 45.9$ (USD per 1bp). Sign check: rates down $\Rightarrow$ PV up for a receivable cashflow.
6. On $[1,2]$, $y(T)=0.05+0.01(T-1)$ so $y'(T)=0.01$ and $f(T)=y(T)+T y'(T)$. Then $f(1.5)=0.055+1.5\times 0.01=7.0\\%$ and $f(2^-)=0.06+2\times 0.01=8.0\\%$.
8. Ringing / non-locality: a single-quote bump can induce oscillations in far-dated forwards, creating spurious deltas far from the bumped quote.
13. Hedge should offset $(+10,-5)$, so solve $2p_1+0.5p_2=-10$ and $4p_2=+5$. Then $p_2=1.25$ and $p_1=-5.31$. Hedge: short $USD 5.31$mm 2Y swaps and long $USD 1.25$mm 5Y swaps.

---

## References

- Andersen & Piterbarg, *Interest Rate Modeling*, Vol. I, Chapter 6 (“Yield Curve Construction and Risk Management”), especially §6.5.1 “Curve Overlays and Turn-of-Year Effects”.
- Tuckman & Serrat, *Fixed Income Securities: Tools for Today’s Markets*, sections on continuous compounding relationships and curve-fitting/interpolation artifacts.
- Hull, *Options, Futures, and Other Derivatives*, sections on zero rates/discount factors, bootstrapping, and forward rates.
- Hirsa, *Computational Methods in Finance*, “Construction of the Discount Curve” (practical curve construction; spline and tension-spline methods).
- Oosterlee, *Mathematical Modeling and Computation in Finance*, sections on yield-curve construction criteria (locality/hedge-locality) and Jacobian-style calibration/risk framing.
