# Chapter 17: Curve Construction — Bootstrapping, Interpolation, and the Spline Zoo

---

## Introduction

Imagine you are trying to draw a map of the ocean floor, but you only have depth measurements at ten specific buoys. You know exactly how deep it is at those ten points, but what is happening in between? Is the floor flat? Does it slope gently? Or is there a sudden trench just barely missing your measurements?

This is the fundamental challenge of **curve construction**. In the fixed income market, we observe prices for a finite set of "benchmark" instruments—a few dozen deposits, futures, and swaps. From these sparse data points, we must infer a continuous **discount factor curve** that allows us to assign a value to cash flows occurring on *any* date, not just the ones where instruments mature.

This is an **inverse problem**. The market gives us the outputs (prices), and we must reverse-engineer the input (the discount function). Andersen and Piterbarg frame the issue precisely: "the problem of curve construction essentially boils down to supplementing [the pricing equations] with enough additional assumptions to allow us to extract $\mathbf{P}$ and to determine $P(T)$ for values of $T$ not in the cash flow timing set." The problem is inherently **underdetermined**—there are infinitely many curves that could perfectly explain the observed market prices. To choose one, we must impose a structure: a mathematical rule for how to connect the dots.

**Why this matters:** The choice of structure—specifically, the **interpolation method**—is not a neutral administrative detail. It is a modeling decision with real financial consequences. Two desks using identical market quotes but different interpolation schemes will calculate different forward rates, different risk sensitivities, and ultimately different valuations for non-standard instruments. As Andersen and Piterbarg emphasize, understanding the artifacts introduced by your curve construction routine—such as "saw-tooth" forwards or "ringing" oscillations—is crucial for managing a book's actual risk. Tuckman adds that "the shortcomings of a curve fitting technique are least noticeable in the discount function, more noticeable in the spot rate curve, and particularly noticeable in the forward rate curve."

> **Desk Reality: Why Your P&L Report Disagrees with Front Office**
>
> When your daily P&L report shows different valuations from the front office system, the culprit is often curve construction methodology. Bank A might use linear yield interpolation; Bank B might use cubic splines. Both can reprice the same benchmark swaps perfectly—but for a 7.5-year custom coupon bond, they will disagree by 0.1–0.5 basis points. On a $500mm notional, that's $25,000–$125,000 of "model difference" that has nothing to do with market moves. Understanding *why* this happens is the first step to explaining—and trading—these discrepancies.

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

$$\boxed{V_i = \sum_{j=1}^{M} c_{i,j} \, P(t_j)}$$

Here, $P(t)$ is the **discount factor**: the value today of one unit of currency to be received at time $t$. This is the fundamental object we are trying to discover.

If we have $N$ benchmark instruments with cash flows on $M$ distinct dates, we can stack these equations into a matrix system. Let $\mathbf{V} = (V_1, \ldots, V_N)^\top$ be the vector of observed market prices, $\mathbf{P} = (P(t_1), \ldots, P(t_M))^\top$ the vector of discount factors, and $\mathbf{C}$ the $N \times M$ matrix of cash flow coefficients. We are solving:

$$\mathbf{V} = \mathbf{C} \mathbf{P}$$

### 17.1.2 The Underdetermined Nature

The difficulty lies in the dimensions. A curve built from 15 liquid benchmark swaps generates cash flows on approximately 120 distinct dates over 30 years—every semiannual or quarterly coupon. Worse, to price a client's specific deal, we might need a discount factor for a date like 1.42 years, which appears in none of our benchmarks.

We are solving for a continuous function $P(T)$ defined for all $T \ge 0$, but we only have $N$ constraints. This means **you cannot build a curve without an interpolation assumption**. When a trader quotes the "market rate" for a 17-month loan, but there is no 17-month liquidity, they are quoting you their interpolation model, not a traded price.

Andersen and Piterbarg identify three basic approaches to resolve this underdetermination:

1. **Add fictitious securities:** Pad the benchmark set using interpolation rules applied to existing instruments. This is ad-hoc and can lead to odd-looking curves.

2. **Parametric/Spline representation:** Assume the yield curve has a specific functional form (e.g., Nelson-Siegel) or is a spline with $N$ knots at benchmark maturities. Solve for the knot values.

3. **Optimization with regularization:** Replace exact pricing with penalized least-squares, allowing the curve to miss quotes slightly in exchange for smoothness.

In practice, option 2 (splines with bootstrapping) is the most common for constructing SOFR/OIS curves, while option 3 is used for noisy benchmark sets like corporate bonds or emerging market sovereigns.

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

The updating of guesses in Steps 2–5 can be handled by any standard one-dimensional root-search algorithm. Newton-Raphson typically converges in 3–5 iterations for well-behaved instruments; the secant method is equally common and avoids computing derivatives explicitly. Hull emphasizes that this bootstrap method "involves starting with short-term instruments and moving progressively to longer-term instruments, making sure that the zero rates calculated at each stage are consistent with the prices of the instruments. It is used daily by trading desks to calculate a variety of zero curves."

### 17.2.2 Closed-Form Solutions for Standard Instruments

For common instruments, each bootstrap step often has a closed-form solution, avoiding iteration entirely.

**Money Market Deposits**

A deposit earning a simple rate $L$ over time $\tau$ pays $(1 + L\tau)$ at maturity. Its present value must equal the notional (which we normalize to 1). Thus:

$$\boxed{P(\tau) = \frac{1}{1 + L\tau}}$$

**Par Swaps**

A standard fixed-for-floating swap is quoted at a fixed rate $c$ that makes the NPV zero. Since the floating leg is worth par at inception (by the standard replication argument), the fixed leg plus principal repayment must also equal 1:

$$1 = c \sum_{j=1}^n \tau_j P(t_j) + P(t_n)$$

If we already know the discount factors for all previous coupons $P(t_1), \ldots, P(t_{n-1})$, we can rearrange to solve for the final discount factor:

$$\boxed{P(t_n) = \frac{1 - c \sum_{j=1}^{n-1} \tau_j P(t_j)}{1 + c \tau_n}}$$

### 17.2.3 The Stub Rate: Handling the First Period

Before bootstrapping can begin, we need the discount factor from today to the first benchmark maturity. This period—called the **stub**—must be handled specially because there is no instrument that matures exactly at settlement.

Andersen and Piterbarg note: "One common choice is to simply set $y(t) = y(T_1)$ for $t < T_1$." In other words, extend the first benchmark's yield flat backward to today. Alternatively, if you have an overnight rate (like Fed Funds or SOFR), you can bootstrap that first segment from overnight markets. Tuckman illustrates the stub rate concept with Eurodollar futures, where "the rate on the stub—the period of time from the settlement date to the beginning of the period spanned by the first Eurodollar contract—is $2.085\%$."

> **Desk Reality: Stub Rate Errors Propagate Everywhere**
>
> Because bootstrapping is sequential, a bad stub rate affects *every* subsequent discount factor. If your stub is 5 bps too high, the entire curve shifts. This is why curve builders obsess over the first segment—getting it wrong is the fastest way to break a curve.

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
$$= 0.061321 + 0.938679 = 1.0000 \; \checkmark$$

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

We found the nodes, but what is $P(1.5)$? This is where interpolation enters. The choice of interpolation method determines the shape of the curve between solved points—and more importantly, the shape of the **forward curve**. Andersen and Piterbarg provide a comprehensive survey of methods, organized by the smoothness (differentiability) of the resulting yield curve.

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

Andersen and Piterbarg observe that "the discontinuous 'saw-tooth' shape of the forward curve is characteristic for bootstrapped yield curves with piecewise linear yield." We derive this mathematically in Section 17.4.

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
- $y(1) = -\ln(0.9434)/1 = 5.83\%$
- $y(2) = -\ln(0.8814)/2 = 6.31\%$

Linear interpolation:
$$y(1.5) = 0.5 \times 5.83\% + 0.5 \times 6.31\% = 6.07\%$$

Discount factor:
$$P(1.5) = e^{-0.0607 \times 1.5} = e^{-0.0911} = 0.9129$$

Forward rate (see Section 17.4):
$$f(1.5) = y(1.5) + 1.5 \times y'(1.5) = 6.07\% + 1.5 \times (6.31\% - 5.83\%) = 6.07\% + 0.72\% = 6.79\%$$

**Method B: Log-Linear on Discount Factors**

$$P(1.5) = \sqrt{P(1) \times P(2)} = \sqrt{0.9434 \times 0.8814} = 0.9119$$

Yield:
$$y(1.5) = -\ln(0.9119)/1.5 = 6.14\%$$

Forward rate (constant on interval):
$$f_{[1,1.5]} = f_{[1.5,2]} = \frac{-\ln(P(2)/P(1))}{1} = -\ln(0.8814/0.9434) = 6.79\%$$

**Comparison:**

| Method | $P(1.5)$ | $y(1.5)$ | $f(1.5)$ |
|--------|----------|----------|----------|
| Linear Yield | 0.9129 | 6.07% | 6.79% (varies across interval) |
| Log-Linear DF | 0.9119 | 6.14% | 6.79% (constant on interval) |

The difference in $P(1.5)$ is about 10 cents per $100—small for a single cash flow, but meaningful when aggregated across a large portfolio.

### 17.3.3 $C^1$ Methods: Hermite Splines

To produce a continuous forward curve, we need a yield curve that is at least once differentiable. Andersen and Piterbarg recommend **Hermite cubic splines**, specifically the **Catmull-Rom** variant, where derivatives at knot points are estimated from finite differences.

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

**The Ringing Problem:** Andersen and Piterbarg warn that $C^2$ cubic splines are "often subject to oscillatory behavior, spurious inflection points, poor extrapolatory behavior, and non-local behavior when prices in the benchmark set are perturbed." Perturbation of a single benchmark price can cause a slow-decaying "ringing" effect, spreading into the entire yield curve. This occurs because interpolation on $[T_i, T_{i+1}]$ depends on *all* values $y_1, \ldots, y_N$ through the full $(N-2) \times (N-2)$ matrix system.

### 17.3.5 Tension Splines: The Best of Both Worlds

To retain $C^2$ smoothness while controlling locality and stiffness, Andersen and Piterbarg advocate **exponential tension splines**. The key modification replaces the pure cubic with a hyperbolic form:

$$y(T) = \left(\frac{\sinh(\sigma(T_{i+1}-T))}{\sinh(\sigma h_i)} - \frac{T_{i+1}-T}{h_i}\right)\frac{y''_i}{\sigma^2} + \left(\frac{\sinh(\sigma(T-T_i))}{\sinh(\sigma h_i)} - \frac{T-T_i}{h_i}\right)\frac{y''_{i+1}}{\sigma^2} + y_i \frac{T_{i+1}-T}{h_i} + y_{i+1} \frac{T-T_i}{h_i}$$

where $\sigma \geq 0$ is the **tension factor**.

**Critical insight from Andersen and Piterbarg:**
- Setting $\sigma = 0$ recovers the ordinary $C^2$ cubic spline.
- As $\sigma \to \infty$, the tension spline uniformly approaches a linear spline.
- A tension spline is thus "a twice differentiable hybrid between a cubic spline and a linear spline."

Regarding the choice of $\sigma$, they state: "We have no definitive answers to this question... Instead, we normally treat $\sigma$ as an 'extra knob' that allows users to balance curve smoothness, shape preservation, and perturbation locality to their particular tastes. Inevitably some element of experimentation is required here."

> **Analogy: The Flexible Rod**
>
> Think of a tension spline as a flexible rod that you can stiffen by increasing $\sigma$. At $\sigma = 0$, it's maximally flexible (standard cubic, which can wiggle). As $\sigma$ increases, it becomes stiffer—eventually so stiff that it's practically a straight edge (piecewise linear). Most desks pick something in between.

### 17.3.6 Monotone Convex and Shape-Preserving Methods

Some practitioners use **monotone convex** interpolation to guarantee positive forward rates while maintaining smoothness. Andersen and Piterbarg's index references "shape preserving" splines as a category of methods designed to avoid "overshooting" artifacts where forward rates briefly go negative or become implausibly large.

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

> **Desk Reality: The Trading Impact of Interpolation**
>
> If Bank A uses linear yield interpolation and Bank B uses cubic splines, they will price "off-node" instruments differently. For a vanilla 7.5-year swap (halfway between 5Y and 10Y benchmarks), differences can be on the order of 0.1–0.5 basis points depending on the curve shape and method. This seems small, but:
>
> 1. **For exotics with many intermediate cashflows** (CMO, amortizing swap), differences compound.
> 2. **For illiquid tenors** (17-month, 9-year), differences can exceed 1 bp.
> 3. **Sophisticated desks monitor these "model differences"** as potential micro-arbitrage. If Bank A will buy at their curve's level and Bank B will sell at theirs, you can clip the difference.
>
> The practical takeaway: know what interpolation method your counterparty uses, and price accordingly.

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
> *   This is why many pricing/risk systems avoid simple linear yield interpolation when valuing instruments sensitive to the forward curve shape (e.g., options).

### 17.4.3 Worked Example: The Boundary Jump

Using $y(1) = 5.83\%$ and $y(2) = 6.31\%$:

**At $T = 1.5$ (middle of interval):**

$$y(1.5) = 6.07\%$$
$$y'(1.5) = \frac{6.31\% - 5.83\%}{1} = 0.48\%$$
$$f(1.5) = 6.07\% + 1.5 \times 0.48\% = 6.79\%$$

**At $T = 2^-$ (just before node):**

$$y(2^-) = 6.31\%$$
$$f(2^-) = 6.31\% + 2.0 \times 0.48\% = 7.27\%$$

**At $T = 2^+$ (just after node):**

Suppose the next segment has $y(3) = 6.70\%$, implying slope $= 0.39\%$. Then:

$$f(2^+) = 6.31\% + 2.0 \times 0.39\% = 7.09\%$$

**The jump:** $f(2^-) = 7.27\%$ but $f(2^+) = 7.09\%$. The forward rate drops by 18 bps at the node—this is the "tooth" of the saw.

### 17.4.4 The Staircase Pattern

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

In their illustration, a 1bp bump to the 2Y swap causes oscillations of ±0.2bp propagating to 10Y+ forwards in the natural cubic case.

The ringing occurs because the $C^2$ spline solves a global matrix system, so changing one input propagates through the entire solution.

### 17.5.3 Tension Dampens Ringing

Adding tension to the $C^2$ spline progressively suppresses oscillations. Andersen and Piterbarg show that as $\sigma$ increases, the perturbation response becomes more local, approaching the clean behavior of bootstrapping. "The usage of a tension factor can have a beneficial impact on risk reports produced by the par-point approach."

### 17.5.4 Why Locality Matters for P&L

Traders hate ringing because it creates **false explanatory power**. If you hedge a 30-year bond using 30-year swaps, you expect your P&L to be stable when short-term rates move. If your curve construction has poor locality, a move in the Fed Funds rate might imply a spurious P&L change in your 30-year book, purely because your spline "wiggled."

Andersen and Piterbarg note a practical workaround: "building two different curves. One—smooth—is then used for pricing and the other—bootstrapped and with good locality—used for risk computations." However, this "tends to suffer from poor P&L predict, in the sense that changes in valuations... are not well explained by first-order sensitivities."

> **Desk Reality: The Two-Curve Approach**
>
> Some desks maintain two curve sets:
> 1. A **smooth $C^2$ spline** for pricing exotic products where the forward curve shape matters
> 2. A **bootstrapped/Hermite curve** for risk reports where locality matters
>
> The mismatch creates P&L "predict" issues—daily P&L doesn't match delta × rate move. Risk teams spend considerable time reconciling the difference. The ideal solution is a tension spline tuned to achieve both goals, but in practice, many desks live with the dual-curve compromise.

---

## 17.6 Smoothing, Exact Fit, and Parametric Alternatives

So far, we have assumed the curve must hit every market price exactly. This is the **exact fit** approach. But market data is noisy.

### 17.6.1 Sources of Noise

- **Bid-ask spreads:** The "mid" price is itself an interpolation.
- **Stale quotes:** Some prices may be hours old.
- **Data entry errors:** Typos happen.
- **Illiquid instruments:** Prices may not reflect executable levels.

If you force a curve to pass exactly through a "bad" quote (e.g., a typo where 2.15% is entered as 2.51%), the curve will dip violently to hit that point and then rip back up to the next valid point. This creates wild forward rates and massive, incorrect hedge ratios.

> **Desk Reality: When Smoothing Saves You**
>
> Before bootstrapping, verify quote reasonableness—a 50bp typo in a 5Y swap can create wild forward rates. Many production systems run "quote health checks" before curve construction:
> - Is the quote within 3 standard deviations of yesterday's close?
> - Is it monotonic with surrounding tenors (approximately)?
> - Does it create any negative forward rates?
>
> Failing these checks should trigger an alert, not automatic smoothing. But in fragmented markets (corporate bonds, emerging markets, distressed credits), standard bootstrapping fails because inputs are inherently noisy. Global optimization with smoothing penalty becomes "relative value defined mathematically"—the smoothed curve is your prior, and deviations from it are RV signals.

### 17.6.2 The Penalized Least-Squares Approach

Andersen and Piterbarg formulate the curve fitting problem as an optimization:

$$\hat{y} = \underset{y \in \mathcal{A}}{\text{argmin}} \; \mathcal{I}(y)$$

where

$$\boxed{\mathcal{I}(y) = \frac{1}{N}(\mathbf{V} - \mathbf{C}\mathbf{P}(y))^\top \mathbf{W}^2 (\mathbf{V} - \mathbf{C}\mathbf{P}(y)) + \lambda \int_{t_1}^{t_M} \left[y''(t)^2 + \sigma^2 y'(t)^2\right] dt}$$

The norm consists of:

1. **Pricing error penalty:** How far the model price is from the market quote, weighted by $\mathbf{W}$.
2. **Smoothness penalty:** $\lambda \int y''(t)^2 \, dt$ penalizes high curvature.
3. **Curve-length penalty:** $\lambda \sigma^2 \int y'(t)^2 \, dt$ penalizes oscillations.

The parameter $\lambda$ controls the tradeoff between fitting prices and smoothness.

### 17.6.3 A Key Result

Andersen and Piterbarg prove:

> **Proposition:** The curve $\hat{y}$ that minimizes the penalized norm is a natural exponential tension spline with tension factor $\sigma$ and knots at all cash flow dates.

This provides theoretical justification for tension splines: they are not just ad-hoc—they are the optimal curves in a well-defined variational sense.

### 17.6.4 Choosing $\lambda$ and RMSE Targets

If $\lambda$ is high, you get a smooth curve that might miss prices by a fraction of a basis point. If $\lambda$ is low, you get a curve that hits every price but may wiggle unreasonably.

Tuckman defines the root mean square error (RMSE) as:

$$\text{RMSE} = \sqrt{\frac{1}{N} \sum_{i=1}^{N} (y_i - \hat{y}_i)^2}$$

and notes: "A RMSE of 3 basis points means that ±3 basis points correspond to a 1-standard deviation error of the fit."

**Practical RMSE targets:**
- **Liquid government curves:** Target RMSE < 0.5 bp. These markets are so liquid that any larger error suggests a data problem.
- **Corporate/EM curves:** 2–5 bp may be acceptable given bid-offer spreads of 5–20 bp.
- **Distressed credits:** 10+ bp is common; the curve is more of a smoothed prior than a precise fit.

A practical approach replaces the unconstrained problem with a constrained one:

$$\min_{y} \int \left[y''(t)^2 + \sigma^2 y'(t)^2\right] dt \quad \text{subject to} \quad \text{RMSE} = \gamma$$

where $\gamma$ is the allowed root-mean-square pricing error.

### 17.6.5 Nelson-Siegel-Svensson: Why Traders Can't Use It

Andersen and Piterbarg note that "parametric functional forms (e.g. Nelson and Siegel [1987]) are sometimes used" for curve construction. The Nelson-Siegel model expresses the yield curve as:

$$y(T) = \beta_0 + \beta_1 \frac{1 - e^{-T/\tau}}{T/\tau} + \beta_2 \left(\frac{1 - e^{-T/\tau}}{T/\tau} - e^{-T/\tau}\right)$$

The Svensson extension adds a second curvature term with 6 parameters total. These models are widely used by central banks (Federal Reserve, ECB, Bank of England) for macro analysis because:

- **Interpretable factors:** $\beta_0$ is the long-run level, $\beta_1$ captures slope, $\beta_2$ captures curvature.
- **Smooth extrapolation:** The curve has sensible behavior at very long maturities.

**Why traders can't use NSS:**

1. **Too rigid:** 4–6 parameters cannot match 15+ benchmark quotes exactly. Trading desks need curves that reprice all benchmarks to within 0.1 bp.
2. **Poor locality:** Changing one parameter (to fit one maturity better) affects the *entire* curve. There's no way to bump the 2Y rate without moving 10Y.
3. **No micro-structure:** NSS assumes smooth fundamentals; it cannot capture turn-of-year spikes, Fed meeting discontinuities, or other calendar effects.

> **Practitioner Note:** Central banks want smooth macro interpretation; traders need exact benchmark repricing. These goals are incompatible. NSS is appropriate for economic research and regulatory stress testing, but not for desk-level pricing and hedging.

---

## 17.7 Special Calendar Effects: Turns, Meetings, and Stubs

The standard bootstrapping framework assumes cash flows occur on generic dates determined by tenor. In reality, specific calendar dates create discontinuities that must be handled specially.

### 17.7.1 The Turn-of-Year Effect

Every December 31st, overnight and short-term rates spike due to year-end funding pressures. Banks face regulatory constraints (balance sheet limits, capital charges) that make them reluctant to lend over the turn. The "turn premium" can be 10–50 basis points depending on the year.

If you simply bootstrap through this spike, the elevated overnight rate will propagate into your entire curve, making January look like a high-rate environment when it's actually returning to normal.

**Implementation approach:**
1. Identify the turn period (typically December 31 through January 1–3).
2. Insert a special "turn date" node in your curve.
3. Bootstrap the turn rate separately, treating it as an isolated event.
4. After the turn, resume normal bootstrapping without carrying the turn rate forward.

### 17.7.2 Central Bank Meetings and Fed Funds Steps

The FOMC meets roughly 8 times per year, and each meeting creates a potential step change in the overnight rate. The OIS curve should embed these expected steps, not smooth through them.

For example, if the market expects a 25bp hike at the March FOMC meeting:
- The forward overnight rate from today to the meeting should reflect current policy (say, 5.00%).
- The forward overnight rate after the meeting should reflect expected new policy (5.25%).

Curve strippers handle this by:
1. Inserting nodes at each FOMC meeting date.
2. Allowing discrete jumps in the forward curve at those dates.
3. Using OIS futures or Fed Funds futures to extract the probability-weighted expected rate change.

### 17.7.3 Reserve Maintenance Periods

In the U.S., banks maintain reserves over 14-day "maintenance periods." The effective Fed Funds rate often shows patterns within these periods—higher at the start (when banks are uncertain about their needs) and lower at the end (when banks with excess reserves scramble to lend).

For very short-dated curves (overnight to 1 month), this creates a "heartbeat" pattern that affects pricing of overnight swaps and Fed Funds futures.

### 17.7.4 Stub Period Handling

Tuckman emphasizes the stub rate concept: "the rate on the stub—the period of time from the settlement date to the beginning of the period spanned by the first Eurodollar contract—is $2.085\%$."

The stub period from settlement to the first benchmark must be carefully specified to avoid absorbing turn or meeting-date effects. Common approaches:

1. **Extend first rate flat backward:** Set $y(t) = y(T_1)$ for $t < T_1$.
2. **Use overnight rate:** Bootstrap the stub from Fed Funds or SOFR.
3. **Use specific stub quote:** If available, use a quoted rate for the exact stub period.

> **Desk Reality: The "Polluted Curve" Problem**
>
> A common mistake: the curve builder ignores the turn-of-year and simply bootstraps through December. The result is a curve that overstates forward rates for the entire first half of January. When trades settle, they price differently than expected. The fix is laborious: insert the turn node, strip it, remove it from propagation. Most production curve systems have explicit turn-date handling built in.

---

## 17.8 The Jacobian Method: From Forward Deltas to Hedges

Rather than bumping benchmark prices, some practitioners apply perturbations directly to the forward curve itself.

### 17.8.1 Forward Rate Deltas

Define functional shifts $\mu_k(t)$ to the forward curve $f(t)$:

$$f(t) \mapsto f(t) + \varepsilon \mu_k(t)$$

Common choices include:
- **Piecewise flat:** $\mu_k(t) = \mathbf{1}_{\{t \in [t_k, t_{k+1})\}}$
- **Triangular:** Rises linearly to a peak at $t_k$, then falls linearly.

The functional derivatives $\partial_k V_0 = \frac{d V_0(f + \varepsilon \mu_k)}{d\varepsilon}\big|_{\varepsilon=0}$ are called **forward rate deltas**.

### 17.8.2 The Jacobian Method

To translate forward rate deltas into hedge positions, define the Jacobian matrix $\partial \mathbf{H}$ with columns $(\partial_1 H_1, \ldots, \partial_K H_L)^\top$, where $H_l$ is the value of the $l$-th hedging instrument.

The optimal hedge weights $\mathbf{p}$ solve:

$$\hat{\mathbf{p}} = \underset{\mathbf{p}}{\text{argmin}} \left(\sum_{k=1}^K W_k^2 (\partial_k H_0(\mathbf{p}) - \partial_k V_0)^2 + \sum_{l=1}^L U_l^2 p_l^2\right)$$

The solution is:

$$(\partial \mathbf{H} \mathbf{W}^2 \partial \mathbf{H}^\top + \mathbf{U}^2) \hat{\mathbf{p}} = \partial \mathbf{H} \mathbf{W}^2 \partial \mathbf{V}_0$$

Andersen and Piterbarg note that "the Jacobian method serves to decouple risk calculations from curve construction. This, potentially, allows for combining smooth curves with localized risk, a feat that is difficult to achieve by other methods."

### 17.8.3 Worked Example: Simple Hedge Calculation

**Setup:** A portfolio has forward rate deltas $(+5, +3, -2, -6)$ at 1Y, 2Y, 3Y, 5Y buckets (in $thousands per bp). Available hedges are 2Y and 5Y swaps.

**Step 1: Hedge Instrument Deltas**

A 2Y swap has approximate deltas $(+0.5, +1.5, 0, 0)$ per $1mm notional.
A 5Y swap has approximate deltas $(+0.2, +0.4, +0.6, +3.8)$ per $1mm notional.

**Step 2: Set Up System**

Let $p_1$ = 2Y swap notional (in $mm), $p_2$ = 5Y swap notional.

We want to match portfolio deltas as closely as possible:

At 1Y: $0.5 p_1 + 0.2 p_2 \approx -5$ (negative because we're hedging)
At 2Y: $1.5 p_1 + 0.4 p_2 \approx -3$
At 3Y: $0 p_1 + 0.6 p_2 \approx +2$
At 5Y: $0 p_1 + 3.8 p_2 \approx +6$

**Step 3: Solve**

This is overdetermined (4 equations, 2 unknowns). The Jacobian method finds the least-squares solution:

$$\mathbf{J} = \begin{pmatrix} 0.5 & 0.2 \\ 1.5 & 0.4 \\ 0 & 0.6 \\ 0 & 3.8 \end{pmatrix}, \quad \mathbf{d} = \begin{pmatrix} -5 \\ -3 \\ 2 \\ 6 \end{pmatrix}$$

$$\hat{\mathbf{p}} = (\mathbf{J}^\top \mathbf{J})^{-1} \mathbf{J}^\top \mathbf{d}$$

Solving gives approximately $p_1 \approx -3.1$ mm, $p_2 \approx +1.5$ mm.

The hedge is: short $3.1mm 2Y swaps, long $1.5mm 5Y swaps.

---

## Summary

Curve construction is the bedrock of fixed income quantitative modeling. It is an inverse problem that requires inferring a continuous reality from sparse, finite observations.

1. **Underdetermination is fundamental:** There is no "true" curve, only a model that fits the prices. Every curve embeds interpolation assumptions.

2. **Bootstrapping is the workhorse:** The standard iterative technique solves the curve node-by-node, with excellent locality. Closed-form solutions exist for deposits and swaps.

3. **Stub rate handling matters:** The first segment of the curve must be specified carefully to avoid propagating errors through the entire curve.

4. **Interpolation method matters profoundly:** Simpler methods (linear yield) create saw-tooth forward artifacts. Log-linear produces staircases. Cubic splines smooth the curve but risk ringing.

5. **Tension splines offer a tunable compromise:** They retain $C^2$ smoothness while allowing control over locality. The tension parameter $\sigma$ balances smoothness against stiffness.

6. **Locality is key for risk management:** A robust method ensures that shocks to short-term rates don't destabilize long-term valuations.

7. **Smoothing vs. exact fit:** Sometimes it is better to miss a quote by 0.1 bps than to wreck the curve's shape to fit a bad data point.

8. **Special calendar effects require special handling:** Turn-of-year spikes, Fed meetings, and stub periods need explicit treatment.

9. **Different interpolation methods create trading opportunities:** When counterparties use different curve methodologies, sophisticated desks can exploit the pricing differences.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Discount Factor $P(T)$** | Value today of 1 unit payable at time $T$ | The universal currency of valuation |
| **Bootstrapping** | Sequential method to solve curve nodes | Most common, robust, excellent locality |
| **Stub Rate** | Rate from settlement to first benchmark | Errors propagate through entire curve |
| **Interpolation** | Rule for filling gaps between solved nodes | Determines forward curve shape |
| **Inverse Problem** | Inferring the function ($P$) from outputs ($V$) | Highlights that the curve is a model |
| **Saw-Tooth** | Forward artifact from linear yield interpolation | Unrealistic jumps in forwards |
| **Staircase** | Forward artifact from log-linear interpolation | Constant forwards between nodes |
| **Ringing** | Oscillations from global $C^2$ splines | Creates false risk sensitivities |
| **Locality** | Input changes affect only nearby outputs | Essential for stable hedging |
| **Tension Spline** | $C^2$ spline with adjustable stiffness | Balances smoothness and locality |
| **Penalized Least-Squares** | Optimization allowing controlled pricing errors | Handles noisy benchmark sets |
| **Turn-of-Year** | December 31st funding spike | Must handle separately |
| **RMSE** | Root mean square pricing error | Measures curve fitting quality |

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
| $\mathbf{W}$ | Weighting matrix for pricing errors |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the fundamental unknown in curve construction? | The continuous discount factor function $P(T)$. |
| 2 | Why is the curve problem "underdetermined"? | We have $N$ finite quotes but need a continuous function ($\infty$ points). |
| 3 | What is bootstrapping? | Sequential algorithm that solves for discount factors short-to-long. |
| 4 | What formula gives $P(\tau)$ from a simple deposit rate $L$? | $P(\tau) = 1/(1 + L\tau)$ |
| 5 | What formula gives $P(t_n)$ from a par swap rate $c$? | $P(t_n) = (1 - c \sum_{j=1}^{n-1} \tau_j P(t_j))/(1 + c\tau_n)$ |
| 6 | What is the stub rate? | The rate from settlement to the first benchmark maturity. |
| 7 | What forward artifact does linear yield interpolation produce? | Saw-tooth pattern (discontinuous forward jumps). |
| 8 | What forward artifact does log-linear interpolation produce? | Staircase pattern (piecewise constant forwards). |
| 9 | What is the relationship $f(T) = y(T) + Ty'(T)$ called? | The link between zero rates and instantaneous forwards. |
| 10 | What is curve "ringing"? | Spurious oscillations from non-local spline perturbations. |
| 11 | Which spline type risks ringing? | Natural cubic ($C^2$) splines without tension. |
| 12 | What does the tension parameter $\sigma$ control? | The tradeoff between $C^2$ cubic smoothness and linear-spline locality. |
| 13 | What happens as $\sigma \to 0$ in a tension spline? | Recovers the ordinary $C^2$ cubic spline. |
| 14 | What happens as $\sigma \to \infty$ in a tension spline? | Approaches a piecewise linear (bootstrap) spline. |
| 15 | What is the "par-point approach"? | Bumping each benchmark yield and measuring portfolio sensitivity. |
| 16 | What is the turn-of-year effect? | December 31st funding spike from year-end balance sheet pressures. |
| 17 | Why might a desk prefer smoothing over exact fit? | To avoid destabilizing the curve due to noisy or erroneous data. |
| 18 | What RMSE target is appropriate for liquid government curves? | Less than 0.5 basis points. |
| 19 | Why can't traders use Nelson-Siegel-Svensson models? | Too rigid (4-6 params can't match 15+ benchmarks); poor locality. |
| 20 | What is the Jacobian method? | Using forward rate deltas to compute optimal hedging portfolios. |

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
Write the formula for $f(T)$ on $[1, 2]$ and calculate $f(1.5)$ and $f(2^-)$.

### Problem 6: Tension Spline Intuition
Explain qualitatively what happens to the forward curve as the tension parameter $\sigma$ increases from 0 to $\infty$.

### Problem 7: Ringing Identification
A colleague proposes using a 15-knot natural cubic spline to fit 15 swap rates perfectly. What issue would you warn them about?

### Problem 8: Smoothing Tradeoff
If your benchmark set includes a clearly erroneous 5-year quote (50 bps away from adjacent maturities), should you use exact fit or smoothing? Why?

### Problem 9: Two-Bank Pricing
Bank A uses linear yield interpolation; Bank B uses cubic splines. Both reprice the same 5Y and 10Y benchmark swaps exactly. For a 7.5Y swap, Bank A quotes 5.50% and Bank B quotes 5.48%. Which is "correct"? Explain.

### Problem 10: Turn-of-Year
The overnight rate is 5.00% normally but spikes to 5.50% over December 31st. If you bootstrap naively (without special turn handling), what happens to your January forward rates? How should you handle this?

### Problem 11: Nelson-Siegel Limitations
An economist colleague suggests using Nelson-Siegel to build your trading curve because it has "nice economic interpretation." Give two specific reasons why this won't work for your trading desk.

### Problem 12: Jacobian Hedge
A portfolio has forward rate deltas of $(+10, -5)$ at 2Y and 5Y respectively (in $thousands per bp). A 2Y swap has delta $(+2, 0)$ per $1mm notional; a 5Y swap has delta $(+0.5, +4)$ per $1mm notional. Find the hedge weights that best offset the portfolio deltas.

---

## Solution Sketches

**1.** $P(0.5) = 1/(1 + 0.04 \times 0.5) = 1/1.02 = 0.9804$

**2.** From $1 = 0.05 \times P(1) + 1.05 \times P(2)$:
$1 - 0.05 \times 0.96 = 1.05 \times P(2)$
$0.952 = 1.05 \times P(2)$
$P(2) = 0.9067$

**3.** $P(1.5) = \sqrt{P(1) \times P(2)} = \sqrt{0.95 \times 0.90} = 0.9247$

**4.** $F_{1.0 \to 1.5} = (1/0.5) \times (0.95/0.9247 - 1) = 2 \times 0.0274 = 5.48\%$
$F_{1.5 \to 2.0} = (1/0.5) \times (0.9247/0.90 - 1) = 2 \times 0.0274 = 5.48\%$ ✓ (constant, as expected under log-linear)

**5.** $f(T) = y(T) + T \times y'(T) = [0.05 + (0.06-0.05)(T-1)] + T \times 0.01$
At $T=1.5$: $y(1.5) = 5.5\%$, so $f(1.5) = 0.055 + 1.5 \times 0.01 = 7.0\%$
At $T=2^-$: $f(2^-) = 0.06 + 2.0 \times 0.01 = 8.0\%$

**6.** At $\sigma = 0$, you get a smooth $C^2$ cubic with potential ringing. As $\sigma$ increases, the curve becomes "stiffer," oscillations are dampened, and locality improves. As $\sigma \to \infty$, the curve approaches piecewise linear, with saw-tooth forwards but excellent locality.

**7.** Ringing: a single-point perturbation (or error) will propagate as decaying oscillations throughout the entire curve, creating spurious risk sensitivities at distant maturities. A 1bp bump to the 2Y rate might cause ±0.2bp oscillations at 10Y+.

**8.** Use smoothing. Forcing exact fit would create wild forward rates around the 5-year point, distorting risk measures for all instruments in that region. A small pricing error (controlled by $\lambda$) is preferable to curve instability.

**9.** Neither is objectively "correct"—both are model-dependent interpolations. The 7.5Y point is not traded, so any price is the output of your interpolation assumption. The difference (2 bps) represents model uncertainty, not a tradeable arbitrage (unless you can find someone willing to trade at the other bank's level).

**10.** Naive bootstrapping will propagate the 5.50% overnight rate into your forward curve, making January forwards appear elevated. You should insert a special "turn date" node, bootstrap the turn rate separately, and not let it affect post-turn forwards.

**11.** (1) Nelson-Siegel has only 4 parameters—it cannot reprice 15+ benchmark instruments exactly; you'd have mispricings of 1-5 bps. (2) Poor locality: adjusting any parameter affects the entire curve, so you can't bump the 2Y rate without moving the 30Y rate. Trading requires exact benchmark repricing and localized risk.

**12.** Set up: $2p_1 + 0.5p_2 = -10$ (offset 2Y delta), $0p_1 + 4p_2 = +5$ (offset 5Y delta, sign flip because we're hedging).
From second equation: $p_2 = 1.25$ mm.
Substitute: $2p_1 + 0.625 = -10$, so $p_1 = -5.31$ mm.
Hedge: Short $5.31mm 2Y swaps, long $1.25mm 5Y swaps.

---

## References

- Andersen & Piterbarg, *Interest Rate Modeling* (Vol 1) (curve construction as an inverse problem; bootstrapping; locality vs smoothness; tension splines; Jacobian hedging).
- Tuckman & Serrat, *Fixed Income Securities: Tools for Today’s Markets* (curve fitting, forward-curve artifacts, and interpolation pitfalls).
- Hull, *Options, Futures, and Other Derivatives* (bootstrapping and zero-curve construction basics).
