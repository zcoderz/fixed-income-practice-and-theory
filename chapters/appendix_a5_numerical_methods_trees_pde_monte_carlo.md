# Appendix A5: Numerical Methods — Trees, PDE / Finite Differences, Monte Carlo, and Fourier Methods

---

## Introduction

When a trader asks for the price of a Bermudan swaption, an Asian call on a basket of equities, or a callable range accrual, there is rarely a closed-form answer waiting in a textbook. The payoff depends on too many underlyings, the exercise rights create free-boundary conditions, or the path dependence makes analytical tractability impossible. At that point, the quant reaches for one of four computational workhorses: **trees**, **finite differences**, **Monte Carlo**, or—for certain European options with known characteristic functions—**Fourier transform methods**. Choosing poorly—a tree for a 20-factor model, naïve Monte Carlo for an American put, or Fourier methods for a path-dependent exotic—can mean hours of runtime, unstable Greeks, or simply wrong prices.

This appendix provides a practitioner's map for navigating that choice. We begin with the foundational concepts that underpin all methods: the pricing identity, state dimension, path dependence, and the critical trinity of **stability**, **consistency**, and **convergence**. The Lax Equivalence Theorem—"consistency plus stability implies convergence"—is the theoretical anchor that tells us when a discretization scheme actually works. We also develop the crucial distinction between **strong** and **weak** convergence for SDE simulation, which determines whether your Monte Carlo is accurate for pricing versus hedging.

We then develop each method in detail. **Section A5.4** covers binomial and trinomial trees, from the CRR construction through Hull–White interest rate trees. **Section A5.5** presents finite difference methods: explicit, implicit, Crank–Nicolson, ADI schemes for multi-factor problems, and barrier option handling. **Section A5.6** addresses Monte Carlo, including variance reduction techniques, the Longstaff–Schwartz algorithm for American options, pathwise versus likelihood ratio methods for Greeks, and the modern technique of Multilevel Monte Carlo (MLMC). **Section A5.7** introduces Fourier transform methods—particularly the elegant COS method—which achieve exceptional speed for European options in models with known characteristic functions. Throughout, we emphasize the decision framework (**Section A5.3**): which features of your problem—dimension, exercise style, path dependence, characteristic function availability—point to which method.

By the end of this appendix, you should be able to look at a derivative specification and immediately know which numerical approach to reach for, what pitfalls to avoid, and how to verify that your implementation is actually converging to the right answer.

---

## A5.1 Framework and Notation

| Convention | Description |
|------------|-------------|
| **Measure / pricing** | Risk-neutral pricing with discounting at the (possibly time-varying) short rate |
| **Pricing identity** | $V(t)=\mathbb{E}\Big[\exp\big(-\int_t^T r(u)\,du\big)\,V(T)\mid\mathcal{F}_t\Big]$ |
| **State** | $x\in\mathbb{R}^d$ denotes the **Markov** state (if it exists); $d$ is the **state dimension** |
| **Discretization** | $\Delta t$ for time step; $\Delta x$ (or $\Delta S$) for spatial step; $n$ for Monte Carlo replications |
| **Error** | **Bias** (systematic) vs **variance** (statistical); **discretization** error (grid/time) vs **simulation** error (sampling) |

### Black–Scholes Baseline

For a dividend yield $q$ (also denoted $\delta$ in some texts), a call value can be written as:

$$C(0)=e^{-qT}S_0\Phi(d_1)-e^{-rT}K\Phi(d_1-\sigma\sqrt{T})$$

with

$$d_1=\frac{\ln(S_0/K)+(r-q+\tfrac12\sigma^2)T}{\sigma\sqrt{T}}$$

(Setting $q=0$ gives the non-dividend case. Note: $d_1$ here is distinct from the dimension $d$ used elsewhere in this appendix.)

### A5.1.1 Conventions Used in This Appendix

We treat numerical methods as approximations to a target **present value** (PV) or **value function** $V(t,x)$. Throughout, we separate **modeling error** (wrong dynamics, wrong calibration) from **numerical error** (discretization/sampling). This appendix focuses exclusively on numerical error—how well our computational scheme approximates the solution to the model we have chosen, not whether that model is the right one.

We use "explicit / implicit / Crank–Nicolson" in the standard finite-difference sense. A critical practical note: Crank–Nicolson can develop oscillations for discontinuous terminal conditions unless remedied (e.g., Rannacher stepping, discussed in Section A5.5.6).

### A5.1.2 Notation Glossary

| Symbol | Definition |
|--------|------------|
| $t$ | Current time |
| $T$ | Maturity |
| $x$ | Markov state |
| $d$ | Number of state variables |
| $V(t,x)$ | Derivative value at time $t$ and state $x$ |
| $H(\cdot)$ | Payoff functional (may be path-dependent) |
| $r(t)$ | Short rate |
| $D(t,T)$ | Discount factor: $D(t,T)=\exp\!\left(-\int_t^T r(u)\,du\right)$ |
| $\Delta t$ | Time step |
| $\Delta x$ / $\Delta S$ | Spatial step |
| $n$ | Monte Carlo sample size |
| $\hat{V}_n$ | MC estimator |
| $\phi_X(u)$ | Characteristic function: $\phi_X(u) = \mathbb{E}[e^{iuX}]$ |
| $\mathcal{L}$ | Spatial differential operator (e.g., BS operator) |
| **Stability** | Scheme output remains bounded as steps go to zero (precise definition below) |
| **Consistency** | Discrete operator approximates the continuous operator; local truncation error $\to 0$ |
| **Convergence** | Numerical solution approaches the true solution as discretization $\to 0$ |

---

## A5.2 Core Concepts

### A5.2.1 Pricing Target: $V(t,x)$ or $PV=\mathbb{E}[\text{discounted payoff}]$

**Formal Definition:**

- **Value function view:** $V(t,x)$ is the price conditional on current time and state.
- **Expectation view:** $V(t)=\mathbb{E}\!\left[D(t,T)\,V(T)\mid \mathcal{F}_t\right]$.

**Intuition:**

All four numerical methods (trees, PDE/FD, MC, Fourier) are different approximations to the same object: the discounted expectation (or equivalently the PDE solution under conditions where Feynman–Kac applies).

**In Practice:**

- **Risk engines:** compute PV and Greeks across a book; need predictable runtimes and stable sensitivities.

---

### A5.2.2 State Dimension $d$ (Number of Risk Factors)

**Formal Definition:**

$d$ is the dimension of the Markov state vector $x$ needed so that future evolution depends on $(t,x)$ only.

**Intuition:**

Low $d$ enables grids/trees; high $d$ pushes you toward Monte Carlo.

**In Practice:**

Rates: 1F short-rate models (low $d$) can use PDE/trees; market models with many forward rates often drive $d$ high.

---

### A5.2.3 Path Dependence vs Markov

**Formal Definition:**

A payoff is **path dependent** if it depends on the history $\{S(u):u\le T\}$ beyond the current state $x$.

**Intuition:**

PDE/FD and standard trees need a finite-dimensional Markov state; path dependence either:
- forces **state augmentation** (add running average, running extrema, etc.), raising $d$; or
- pushes you to Monte Carlo, which naturally handles path functionals.

**In Practice:**

Asians, lookbacks, many callable features with path triggers → MC is often the first tool.

---

### A5.2.4 American vs European: Free Boundary Problems

**Formal Definition:**

- **European:** exercise only at maturity.
- **American:** exercise at any time; value is the supremum over stopping times (not derived here).
- In a PDE framing for an American put, the price must satisfy a constraint $P(S,t)\ge \max(K-S,0)$ and features an **unknown exercise boundary** $B(t)$.

**Intuition:**

Early exercise turns pricing into a **free-boundary** or inequality-constrained problem.

**In Practice:**

Equity American options: trees and PDE methods are standard tools; MC needs specialized approaches (LSMC/regression).

---

### A5.2.5 Error Types: Bias vs Variance

**Formal Definition:**

- **Bias:** $\mathbb{E}[\hat{\theta}]-\theta$.
- **Variance:** $\mathbb{V}[\hat{\theta}]$.
- FD/trees: main error is **discretization** (grid/time), typically biased unless extrapolated.
- MC: main error is **statistical** (variance), decaying like $1/\sqrt{n}$ under CLT conditions.

**Intuition:**

- FD/trees: pay more compute → smaller steps → less bias (and often more stable Greeks).
- MC: pay more compute → more paths → narrower confidence interval; but discretizing SDEs also introduces bias (time stepping).

**In Practice:**

Don't confuse "tight CI" with "accurate"—a biased estimator can have tiny variance.

---

### A5.2.6 Stability, Consistency, and Convergence

**Stability (Duffy):**

A scheme is **stable** if its output remains bounded as $\Delta t\to 0$, $\Delta x\to 0$.

**Consistency (Duffy):**

A scheme is **consistent** if the local truncation error $\tau(\Delta t,\Delta x)\to 0$ as $\Delta t,\Delta x\to 0$.

**Convergence (Andersen–Piterbarg):**

Convergence means $\max_j|\hat{v}(\tau_n,x_j)-v(\tau_n,x_j)|\to 0$ for fixed $\tau_n$ as the grid refines.

**Key Linkage (Lax Equivalence, Andersen–Piterbarg):**

$$\boxed{\text{Consistency} + \text{Stability} \Longrightarrow \text{Convergence}}$$

For well-posed linear problems, **consistency + stability ⇒ convergence**.

**Intuition:** The Lax Equivalence Theorem tells you that if your scheme has small local error (consistency) and doesn't amplify errors (stability), then the global error will be small (convergence). Consistency means you're solving approximately the right equation locally; stability means mistakes don't blow up; convergence means the final answer is correct.

---

### A5.2.7 Complexity Scaling

**Formal Objects:**

- FD grid: $N_x$ points per dimension; $N_t$ time steps.
- MC: $n$ paths and $N_t$ time steps per path (time-discretized simulation).
- Fourier: $N$ grid points for FFT.

**Source-Backed Anchors:**

- A standard tridiagonal (1D) $\theta$-scheme with $m$ spatial points and $n$ time steps has complexity $O(mn)$ (Andersen–Piterbarg).
- Finite differences are "suitable for 1, 2 and possibly 3 factors" and beyond that Monte Carlo is typical (Andersen–Piterbarg).
- Monte Carlo time grows roughly linearly with dimension, but early exercise is difficult in "original formulation" (Andersen–Piterbarg).
- FFT-based methods achieve $O(N \log N)$ complexity for European option pricing (Carr–Madan).

**Reasoned Inference (Transparent):**

- FD curse of dimensionality: number of grid points scales like $N_x^d$.
- MC scaling: runtime $\propto n\times N_t\times(\text{cost per step})$, typically linear in $d$ for factor simulations.

---

### A5.2.8 Strong vs Weak Convergence for SDE Simulation

When simulating stochastic differential equations, we must distinguish between two fundamentally different notions of convergence. Glasserman (Chapter 6) provides the definitive treatment; the distinction has profound practical implications.

**The SDE Setting:**

Consider an Itô SDE:
$$dX_t = a(X_t)\,dt + b(X_t)\,dW_t$$

We simulate using a discrete approximation $\hat{X}_n$ with time step $h = T/n$.

**Strong Convergence (Pathwise):**

$$\boxed{\text{Strong order } \beta: \quad \mathbb{E}\left[\left|\hat{X}(T) - X(T)\right|\right] \leq C \cdot h^\beta}$$

Strong convergence measures how well the **simulated path** matches the **true path** on a path-by-path basis. The same Brownian motion drives both the true and approximate processes.

**Weak Convergence (Distributional):**

$$\boxed{\text{Weak order } \beta: \quad \left|\mathbb{E}\left[f(\hat{X}(T))\right] - \mathbb{E}\left[f(X(T))\right]\right| \leq C \cdot h^\beta}$$

for all sufficiently smooth test functions $f$. Weak convergence measures how well the **distribution** of the simulated value matches the true distribution.

**Convergence Orders for Common Schemes:**

| Scheme | Strong Order | Weak Order |
|--------|--------------|------------|
| Euler–Maruyama | 1/2 | 1 |
| Milstein | 1 | 1 |
| Strong order 1.5 (Kloeden–Platen) | 1.5 | 2 |

**Euler–Maruyama Scheme:**

$$\hat{X}_{k+1} = \hat{X}_k + a(\hat{X}_k)h + b(\hat{X}_k)\sqrt{h}\,Z_k, \quad Z_k \sim N(0,1)$$

**Milstein Scheme:**

$$\hat{X}_{k+1} = \hat{X}_k + a(\hat{X}_k)h + b(\hat{X}_k)\sqrt{h}\,Z_k + \frac{1}{2}b(\hat{X}_k)b'(\hat{X}_k)(Z_k^2 - 1)h$$

The Milstein correction term $\frac{1}{2}bb'(Z^2-1)h$ captures the Itô–Stratonovich correction and improves strong convergence from order 1/2 to order 1.

**When Does It Matter?**

| Application | Convergence Type Needed | Why |
|-------------|------------------------|-----|
| **European option pricing** | Weak | Only need $\mathbb{E}[H(S_T)]$ |
| **Path-dependent (Asian, lookback)** | Strong | Path values along the way matter |
| **Greeks via pathwise method** | Strong | Differentiate through specific paths |
| **Greeks via likelihood ratio** | Weak | Only distributional properties needed |
| **Hedging simulation** | Strong | Need to track P&L path-by-path |

> **Desk Reality: The Hidden Danger of Euler**
>
> Euler–Maruyama is often "good enough" for pricing because it has weak order 1. But if you're simulating to test a hedging strategy—where you track P&L along the path—strong order 1/2 can introduce significant bias. Paths wander from their true values, and your simulated hedge performance won't match reality. For hedging simulations, consider Milstein or exact simulation when available.

---

## A5.3 Decision Framework: Which Method When?

This is the practitioner centerpiece: mapping **problem features** to **numerical method**. Before diving into the details of each technique, we establish the decision logic that should guide your choice.

### A5.3.1 Decision Tree

```
                        ┌─────────────────────┐
                        │  Start: Price a     │
                        │  Derivative         │
                        └──────────┬──────────┘
                                   │
                                   ▼
                        ┌─────────────────────┐
                        │  European with      │
                        │  known char func?   │
                        └──────────┬──────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │ Yes                │ No                 │
              ▼                    ▼
     ┌────────────────┐   ┌─────────────────────┐
     │ Fourier/COS    │   │  State dimension d? │
     │ (very fast)    │   └──────────┬──────────┘
     └────────────────┘              │
                        ┌────────────┼────────────┐
                        │            │            │
                        ▼            ▼            ▼
                   ┌────────┐  ┌──────────┐ ┌──────────┐
                   │ d = 1  │  │ d = 2-3  │ │  d > 3   │
                   └───┬────┘  └────┬─────┘ └────┬─────┘
                       │            │            │
                       ▼            ▼            ▼
             ┌───────────────────┐ ┌─────────────┐ ┌──────────────────┐
             │ Path dependent?   │ │ Use ADI/FD  │ │ Monte Carlo      │
             └────────┬──────────┘ │ or 2D Trees │ │ (only option)    │
                      │            └─────────────┘ └──────────────────┘
             ┌────────┴────────┐
             │                 │
             ▼                 ▼
          ┌──────┐         ┌──────┐
          │  No  │         │  Yes │
          └──┬───┘         └──┬───┘
             │                │
             ▼                ▼
          ┌───────────────┐  ┌───────────────────────┐
          │ American/     │  │ Can augment state?    │
          │ Bermudan?     │  │ (e.g., running avg)   │
          └──────┬────────┘  └───────────┬───────────┘
                 │                       │
            ┌────┴────┐            ┌─────┴─────┐
            ▼         ▼            ▼           ▼
         ┌────┐   ┌─────┐      ┌─────┐    ┌─────────┐
         │ No │   │ Yes │      │ Yes │    │   No    │
         └─┬──┘   └──┬──┘      └──┬──┘    └────┬────┘
           │         │            │            │
           ▼         ▼            ▼            ▼
        ┌───────┐ ┌─────────┐ ┌────────┐ ┌──────────┐
        │ Any   │ │ Tree/FD │ │ FD on  │ │ Monte    │
        │method │ │ (fast)  │ │augment.│ │ Carlo    │
        │ works │ │         │ │ grid   │ │          │
        └───────┘ └─────────┘ └────────┘ └──────────┘
```

### A5.3.2 Quick Reference Table

| Feature | Trees | Finite Differences | Monte Carlo | Fourier |
|---------|-------|-------------------|-------------|---------|
| Dimension $d=1$ | ✓✓✓ | ✓✓✓ | ✓ | ✓✓✓ |
| Dimension $d=2$ | ✓ | ✓✓ | ✓✓ | ✓ |
| Dimension $d \ge 3$ | ✗ | ✗ | ✓✓✓ | ✗ |
| European payoff | ✓✓ | ✓✓ | ✓✓ | ✓✓✓ |
| American/Bermudan | ✓✓✓ | ✓✓✓ | ✓ (LSMC) | ✗ |
| Path-dependent | ✗ | ✗ | ✓✓✓ | ✗ |
| Barriers | ✓ | ✓✓✓ | ✓ | ✗ |
| Greeks | Bumping | Direct from grid | Pathwise/LR | Analytic |
| Implementation | Simple | Moderate | Simple | Moderate |
| Convergence rate | $O(\Delta t)$ | $O(\Delta t^2)$ (C-N) | $O(1/\sqrt{n})$ | Exponential |
| Requires char func | No | No | No | Yes |

### A5.3.3 Rule of Thumb

Hull provides a practical summary:

> "Monte Carlo simulation is usually used for derivatives where the payoff is dependent on the history of the underlying variable or where there are several underlying variables. Trees and finite difference methods are usually used for American options and other derivatives where the holder has decisions to make prior to maturity."

**Extension for Fourier methods:** When the model has a known characteristic function and the option is European-style, Fourier methods (COS, Carr–Madan FFT) are often the fastest and most accurate choice.

---

## A5.4 Binomial and Trinomial Trees

### A5.4.1 Cox–Ross–Rubinstein (CRR) Binomial Tree

The CRR construction is the most widely used binomial model. Hull Ch. 13 develops this method using no-arbitrage arguments.

**Setup:** Divide the option life into $N$ time steps of length $\Delta t = T/N$. At each node, the stock price either:
- Moves **up** by factor $u$, or
- Moves **down** by factor $d$

**CRR Parameters:**

$$\boxed{u = e^{\sigma\sqrt{\Delta t}}, \quad d = \frac{1}{u} = e^{-\sigma\sqrt{\Delta t}}}$$

**Risk-Neutral Probability:**

$$\boxed{p = \frac{e^{(r-q)\Delta t} - d}{u - d}}$$

where $r$ is the risk-free rate and $q$ is the continuous dividend yield.

**Tree Construction:** At time $i\Delta t$, the possible stock prices are:
$$S_{i,j} = S_0 u^j d^{i-j}, \quad j = 0, 1, \ldots, i$$

The tree **recombines** because $ud = 1$, so an up move followed by a down move returns to the same price.

**Backward Induction for European Options:**

At maturity ($i = N$):
$$f_{N,j} = \text{payoff}(S_{N,j})$$

For earlier nodes ($i < N$):
$$f_{i,j} = e^{-r\Delta t}\left[p\,f_{i+1,j+1} + (1-p)\,f_{i+1,j}\right]$$

**Backward Induction for American Options:**

At each node, compare continuation value with immediate exercise:
$$f_{i,j} = \max\left\{\text{exercise value}, \; e^{-r\Delta t}\left[p\,f_{i+1,j+1} + (1-p)\,f_{i+1,j}\right]\right\}$$

### A5.4.2 Alternative Binomial Parameterizations

**Jarrow–Rudd (JR) / Equal-Probability Tree:**

Set $p = 0.5$ and choose $u, d$ to match mean and variance:

$$u = e^{(r-q-\sigma^2/2)\Delta t + \sigma\sqrt{\Delta t}}, \quad d = e^{(r-q-\sigma^2/2)\Delta t - \sigma\sqrt{\Delta t}}$$

This tree does **not** recombine in general.

**Leisen–Reimer Tree:**

Uses higher-order matching to improve convergence; eliminates oscillation in option prices as $N$ increases.

### A5.4.3 Trinomial Trees

Hull Section 21.4 introduces trinomial trees as an extension with three possible movements:

$$S \to \begin{cases} Su & \text{with probability } p_u \\ S & \text{with probability } p_m \\ Sd & \text{with probability } p_d \end{cases}$$

**Standard Parameterization (Hull):**

$$p_u = \frac{1}{2}\left[\frac{\sigma^2\Delta t + \nu^2\Delta t^2}{\Delta x^2} + \frac{\nu\Delta t}{\Delta x}\right]$$
$$p_d = \frac{1}{2}\left[\frac{\sigma^2\Delta t + \nu^2\Delta t^2}{\Delta x^2} - \frac{\nu\Delta t}{\Delta x}\right]$$
$$p_m = 1 - p_u - p_d = 1 - \frac{\sigma^2\Delta t + \nu^2\Delta t^2}{\Delta x^2}$$

where $\nu = r - q - \sigma^2/2$ and typically $\Delta x = \sigma\sqrt{3\Delta t}$.

**Advantages of Trinomial Trees:**
- Extra degree of freedom makes it easier to incorporate mean reversion
- Equivalent to explicit finite difference method (proven in Hull Section 21.8)
- Better suited for interest rate models with mean reversion

### A5.4.4 Trees for Interest Rate Models

**Hull–White Tree Construction:**

For the Hull–White model $dr = [\theta(t) - ar]\,dt + \sigma\,dW$:

1. Build a trinomial tree for $x = r - \alpha(t)$ where $\alpha(t)$ is the mean-reverting level
2. The tree for $x$ is standard with mean reversion
3. Shift the tree using $\alpha(t)$ to fit the initial term structure

**Forward Induction for Calibration (Hull Section 32):**

The function $\alpha(t)$ is determined by **forward induction** to match observed discount factors:
1. Start with known $P(0,t_1)$
2. Use Arrow–Debreu prices $Q_{i,j}$ (state prices) to compute $\alpha(t_i)$
3. Adjust probabilities at each node to maintain mean reversion

**Key Feature:** The tree incorporates mean reversion naturally through modified probabilities and non-uniform node spacing. Why trinomial? The three-point stencil provides the extra degree of freedom needed to match both the mean reversion and volatility simultaneously.

### A5.4.5 Convergence of Binomial Trees

Hull Ch. 13 Appendix proves that as $N \to \infty$, the CRR binomial tree price converges to the Black–Scholes price for European options.

**Convergence Rate:** Typically $O(\Delta t) = O(1/N)$ for standard constructions.

**Oscillation:** Even-numbered and odd-numbered $N$ can produce systematically different prices due to whether the final nodes straddle the strike. Practitioners often use:
- Richardson extrapolation
- Leisen–Reimer construction
- Large $N$ (e.g., $N \ge 100$)

**Richardson Extrapolation:**

When the convergence order is known, Richardson extrapolation can significantly improve accuracy. If the error is $O(h^k)$ where $h = \Delta t$:

$$\boxed{V^{(\text{extrap})} = \frac{2^k V_{2N} - V_N}{2^k - 1}}$$

For CRR trees with $k=1$:
$$V^{(\text{extrap})} = 2V_{2N} - V_N$$

> **Practitioner Note:** Richardson extrapolation is powerful but requires the true convergence order. If the payoff has discontinuities (e.g., digital options), the effective order may be lower than expected, and extrapolation can make things worse. Always verify with a convergence study first.

---

## A5.5 PDE / Finite Difference Methods

### A5.5.1 The Pricing PDE

Under geometric Brownian motion with dividend yield $q$, the option value $f(S,t)$ satisfies:

$$\boxed{\frac{\partial f}{\partial t} + (r-q)S\frac{\partial f}{\partial S} + \frac{1}{2}\sigma^2 S^2 \frac{\partial^2 f}{\partial S^2} = rf}$$

**Boundary Conditions (Put Example):**
- At $t = T$: $f(S,T) = \max(K-S, 0)$
- As $S \to 0$: $f(0,t) = Ke^{-r(T-t)}$
- As $S \to \infty$: $f(S,t) \to 0$

### A5.5.2 Discretization Setup

Define a grid with:
- Time steps: $\Delta t$, indexed by $i = 0, 1, \ldots, N$
- Spatial steps: $\Delta S$, indexed by $j = 0, 1, \ldots, M$

The grid value $f_{i,j}$ approximates $f(j\Delta S, i\Delta t)$.

**Finite Difference Approximations (Hull Section 21.8):**

First derivative (central difference):
$$\frac{\partial f}{\partial S} \approx \frac{f_{i,j+1} - f_{i,j-1}}{2\Delta S}$$

Second derivative:
$$\frac{\partial^2 f}{\partial S^2} \approx \frac{f_{i,j+1} + f_{i,j-1} - 2f_{i,j}}{\Delta S^2}$$

### A5.5.3 Implicit Finite Difference Method

The **implicit** method evaluates spatial derivatives at the **unknown** time level. Hull derives:

$$\boxed{a_j f_{i,j-1} + b_j f_{i,j} + c_j f_{i,j+1} = f_{i+1,j}}$$

where the coefficients are:

$$a_j = \frac{1}{2}(r-q)j\Delta t - \frac{1}{2}\sigma^2 j^2 \Delta t$$
$$b_j = 1 + \sigma^2 j^2 \Delta t + r\Delta t$$
$$c_j = -\frac{1}{2}(r-q)j\Delta t - \frac{1}{2}\sigma^2 j^2 \Delta t$$

**Solution:** This gives a tridiagonal system at each time step, solved efficiently in $O(M)$ operations using the Thomas algorithm.

**Thomas Algorithm (LU Decomposition for Tridiagonal):**

For a tridiagonal system $a_j x_{j-1} + b_j x_j + c_j x_{j+1} = d_j$:

```
Forward sweep:
  c'_1 = c_1/b_1
  d'_1 = d_1/b_1
  For j = 2 to M-1:
    c'_j = c_j / (b_j - a_j c'_{j-1})
    d'_j = (d_j - a_j d'_{j-1}) / (b_j - a_j c'_{j-1})

Back substitution:
  x_{M-1} = d'_{M-1}
  For j = M-2 down to 1:
    x_j = d'_j - c'_j x_{j+1}
```

Complexity: $O(M)$ per time step, $O(MN)$ total.

**Stability:** The implicit method is **unconditionally stable**—it converges for any choice of $\Delta t$ and $\Delta S$ (Hull).

### A5.5.4 Explicit Finite Difference Method

The **explicit** method evaluates spatial derivatives at the **known** time level:

$$\boxed{f_{i,j} = a_j^* f_{i+1,j-1} + b_j^* f_{i+1,j} + c_j^* f_{i+1,j+1}}$$

where:

$$a_j^* = \frac{1}{1+r\Delta t}\left(-\frac{1}{2}(r-q)j\Delta t + \frac{1}{2}\sigma^2 j^2\Delta t\right)$$
$$b_j^* = \frac{1}{1+r\Delta t}\left(1 - \sigma^2 j^2\Delta t\right)$$
$$c_j^* = \frac{1}{1+r\Delta t}\left(\frac{1}{2}(r-q)j\Delta t + \frac{1}{2}\sigma^2 j^2\Delta t\right)$$

**Interpretation:** Hull shows these coefficients can be interpreted as transition probabilities, making the explicit method **equivalent to a trinomial tree**.

**Stability Condition:** The explicit method requires all "probabilities" to be non-negative:

$$\boxed{\Delta t < \frac{1}{\sigma^2 j_{\max}^2}}$$

When this is violated, the method produces negative option prices and fails to converge.

### A5.5.5 Change of Variable to $Z = \ln S$

Hull recommends transforming to logarithmic coordinates:

$$\frac{\partial f}{\partial t} + \left(r - q - \frac{\sigma^2}{2}\right)\frac{\partial f}{\partial Z} + \frac{1}{2}\sigma^2\frac{\partial^2 f}{\partial Z^2} = rf$$

**Advantage:** The coefficients become **independent of the spatial index** $j$, simplifying implementation and improving stability.

With $\Delta Z = \sigma\sqrt{3\Delta t}$, the explicit method becomes identical to a standard trinomial tree.

### A5.5.6 Crank–Nicolson Method

The Crank–Nicolson scheme averages the implicit and explicit methods:

$$\frac{f_{i+1,j} - f_{i,j}}{\Delta t} = \frac{1}{2}\left[\mathcal{L}f_{i+1,j} + \mathcal{L}f_{i,j}\right]$$

where $\mathcal{L}$ is the spatial differential operator.

**Properties:**
- **Second-order accurate** in time: $O(\Delta t^2)$ vs $O(\Delta t)$ for pure implicit/explicit
- **Unconditionally stable** (like implicit)
- **Oscillation problem:** For discontinuous payoffs (e.g., digital options), C-N can produce oscillations near the discontinuity

**Rannacher Stepping (Andersen–Piterbarg):**

Use 2–4 fully implicit steps near maturity to damp oscillations, then switch to Crank–Nicolson for the remainder of the backward evolution.

### A5.5.7 American Options via Finite Differences

For American options, at each time step after solving the linear system:

$$f_{i,j} \leftarrow \max\left(f_{i,j}, \; \text{payoff}(j\Delta S)\right)$$

This enforces the early exercise constraint.

**Linear Complementarity Problem (LCP) Formulation:**

The mathematically proper formulation of the American option PDE is:

$$\boxed{\min\left(\frac{\partial V}{\partial t} + \mathcal{L}V - rV, \; V - h\right) = 0}$$

where $h(S) = \max(K-S, 0)$ for a put. This says: either the PDE holds with equality (continuation region) or the value equals the payoff (exercise region). Penalty methods and PSOR solve this LCP efficiently.

**Alternative (PSOR):** The Projected Successive Over-Relaxation method solves the linear complementarity problem directly.

### A5.5.8 Greeks from Finite Differences

**Delta:**
$$\Delta \approx \frac{f_{i,j+1} - f_{i,j-1}}{2\Delta S}$$

**Gamma:**
$$\Gamma \approx \frac{f_{i,j+1} + f_{i,j-1} - 2f_{i,j}}{\Delta S^2}$$

**Theta:**
$$\Theta \approx \frac{f_{i+1,j} - f_{i,j}}{\Delta t}$$

**Vega:** Requires recomputing the entire grid with $\sigma \to \sigma + \epsilon$.

### A5.5.9 Alternating Direction Implicit (ADI) Methods

When the state dimension rises to $d = 2$ or $d = 3$, direct application of implicit finite differences becomes computationally expensive. Consider a 2D grid with $m_1$ points in the first dimension and $m_2$ in the second. A fully implicit scheme requires solving a system of size $m_1 m_2$ at each time step. Using direct matrix methods, the complexity scales poorly—Andersen and Piterbarg note that direct Crank–Nicolson on a 2D grid has complexity $O((m_1 m_2)^{5/4})$ per time step due to the bandwidth of the discretization matrix.

**ADI schemes** split the multidimensional operator into a sequence of one-dimensional problems, each of which can be solved efficiently using the Thomas algorithm. The key insight is that solving a sequence of tridiagonal systems is much cheaper than solving one large banded system.

**Peaceman–Rachford Scheme (2D):**

For a 2D PDE with operators $\mathcal{L}_x$ and $\mathcal{L}_y$ corresponding to the two spatial directions, the Peaceman–Rachford scheme advances from time $n$ to $n+1$ in two half-steps:

$$\left(I - \frac{\Delta t}{2}\mathcal{L}_x\right)V^{n+1/2} = \left(I + \frac{\Delta t}{2}\mathcal{L}_y\right)V^n$$

$$\left(I - \frac{\Delta t}{2}\mathcal{L}_y\right)V^{n+1} = \left(I + \frac{\Delta t}{2}\mathcal{L}_x\right)V^{n+1/2}$$

Each half-step requires solving only tridiagonal systems: the first sweeps in the $x$-direction, the second in the $y$-direction.

**Douglas–Rachford Scheme:**

An alternative splitting that is also second-order accurate in time. Andersen and Piterbarg (Vol. 1, Section 2.11) provide detailed analysis of both schemes.

**Complexity Advantage:**

With ADI, the complexity per time step drops to $O(m_1 m_2)$—linear in the total number of grid points—because we solve $m_2$ tridiagonal systems of size $m_1$ and then $m_1$ systems of size $m_2$. This makes ADI the standard choice for 2D and 3D PDE problems in finance.

**Cross-Derivative Terms:**

Mixed partial derivatives (e.g., $\partial^2 V / \partial x \partial y$ from correlation) require special treatment. The standard approach treats these explicitly or uses modified ADI formulations:

- **Craig–Sneyd scheme:** Handles cross-derivatives with good stability properties
- **Hundsdorfer–Verwer scheme:** Another stable approach for mixed derivatives

Andersen and Piterbarg (Vol. 1, Sections 2.10–2.12) provide detailed stability analysis.

**Stability:**

ADI schemes inherit the unconditional stability of implicit methods when properly formulated, though cross-derivative terms can introduce restrictions.

### A5.5.10 Barrier Options in Finite Differences

Barrier options require special care in FD methods because the barrier introduces an additional boundary condition.

**Knock-Out Options:**

For a down-and-out call with barrier $B$:
- **Boundary condition at barrier:** $V(B, t) = 0$ (option worthless if barrier hit)
- **Grid placement:** The barrier should coincide with a grid line for accuracy

**Discrete vs Continuous Monitoring:**

Most traded barriers are **discretely monitored** (checked at market close), while the PDE naturally handles **continuous monitoring**. This creates a systematic bias.

**Broadie–Glasserman–Kou Continuity Correction:**

For discretely monitored barriers, Broadie, Glasserman, and Kou derived a correction factor. For a down barrier $B$ monitored $m$ times:

$$\boxed{B_{\text{eff}} = B \cdot e^{-\beta\sigma\sqrt{T/m}}}$$

where $\beta \approx 0.5826$ (related to the zeta function). The effective barrier $B_{\text{eff}}$ is lower than $B$, reflecting that discrete monitoring is less likely to trigger than continuous.

**Grid Stretching:**

For better accuracy near barriers, use non-uniform grids with finer spacing near $B$:
- Sinh-stretching: $S_j = B + \alpha \sinh(\beta(j/M - 1/2))$
- Exponential concentration near the barrier

> **Desk Reality: Barrier Option Pitfalls**
>
> The most common bug in barrier option pricing is boundary condition placement. If your grid doesn't hit the barrier exactly, interpolation introduces significant error. Always verify: (1) barrier falls on a grid line, (2) monitoring frequency matches the contract, (3) use continuity correction for discrete barriers priced with continuous methods.

---

## A5.6 Monte Carlo Methods

### A5.6.1 Basic Monte Carlo Principle

The price is the expected discounted payoff:

$$V_0 = \mathbb{E}\left[e^{-rT} H(S_T)\right]$$

**MC Estimator:**

Generate $n$ independent paths $\{S_T^{(1)}, \ldots, S_T^{(n)}\}$ and compute:

$$\boxed{\hat{V}_n = \frac{1}{n}\sum_{k=1}^{n} e^{-rT} H(S_T^{(k)})}$$

**Standard Error:**

$$\text{SE} = \frac{\hat{\sigma}}{\sqrt{n}}$$

where $\hat{\sigma}$ is the sample standard deviation of the payoffs.

**95% Confidence Interval:**
$$\hat{V}_n \pm 1.96 \times \text{SE}$$

### A5.6.2 Simulating Geometric Brownian Motion

For GBM: $dS = (r-q)S\,dt + \sigma S\,dW$

**Exact Simulation (over interval $[0,T]$):**

$$S_T = S_0 \exp\left[\left(r - q - \frac{\sigma^2}{2}\right)T + \sigma\sqrt{T}\,Z\right], \quad Z \sim N(0,1)$$

**Euler Discretization (time-stepping):**

$$S_{t+\Delta t} = S_t + (r-q)S_t\Delta t + \sigma S_t\sqrt{\Delta t}\,Z_t$$

**Milstein Scheme (higher order):**

$$S_{t+\Delta t} = S_t + (r-q)S_t\Delta t + \sigma S_t\sqrt{\Delta t}\,Z_t + \frac{1}{2}\sigma^2 S_t\left(Z_t^2 - 1\right)\Delta t$$

### A5.6.3 Variance Reduction Techniques

Variance reduction is critical for practical Monte Carlo. As Glasserman emphasizes, the goal is not merely to reduce variance but to do so without introducing bias and with computational overhead that is small relative to the variance reduction achieved. Hull Section 21.7 describes the main techniques; Glasserman's treatment in Chapters 4–5 provides deeper analysis.

#### Antithetic Variates

For each random draw $Z$, also use $-Z$:

$$\hat{V} = \frac{1}{2}\left[H(S_T^{(+)}) + H(S_T^{(-)})\right]$$

**Variance Reduction:** Works well when payoff is monotonic in underlying; reduces variance by inducing negative correlation.

#### Control Variates

Use a related quantity with known expected value:

$$\hat{V}_{\text{CV}} = \hat{V} - \beta\left(\hat{Y} - \mathbb{E}[Y]\right)$$

**Example:** Use the simulated terminal stock price as a control variate (known mean: $S_0 e^{(r-q)T}$).

**Optimal $\beta$:**
$$\beta^* = \frac{\text{Cov}(\hat{V}, \hat{Y})}{\text{Var}(\hat{Y})}$$

#### Importance Sampling

Change the sampling distribution to oversample "important" regions:

$$\mathbb{E}_P[H(X)] = \mathbb{E}_Q\left[H(X)\frac{dP}{dQ}(X)\right]$$

**Glasserman:** Importance sampling can dramatically reduce variance for rare-event problems (e.g., deep out-of-the-money options).

**Optimal Drift (Glasserman Theorem 4.83):**

For exponential tilting of Gaussian random variables, the variance-minimizing drift shift can be computed by solving an optimization problem. For deep OTM options, shifting the mean toward the strike reduces variance by orders of magnitude.

#### Stratified Sampling

Divide the sample space into strata and sample proportionally from each:

$$\hat{V}_{\text{strat}} = \sum_{j=1}^{K} p_j \hat{V}_j$$

where $p_j$ is the probability of stratum $j$ and $\hat{V}_j$ is the sample mean within that stratum.

#### Quasi-Monte Carlo

Quasi-Monte Carlo (QMC) replaces pseudo-random numbers with **low-discrepancy sequences** that fill the sample space more uniformly. Glasserman (Chapter 5) provides extensive treatment of both the theory and practical implementation.

**Key Low-Discrepancy Sequences:**

- **Halton sequences:** Based on radical inverses in different prime bases. Simple to implement but can exhibit correlation problems in high dimensions.
- **Sobol sequences:** Constructed using primitive polynomials and direction numbers. Generally preferred in finance due to better high-dimensional properties.
- **Niederreiter sequences:** Another class with good theoretical properties.

**Convergence Rate:**

Under favorable conditions, QMC achieves convergence rate $O((\log n)^d / n)$ where $d$ is the dimension. For moderate dimensions, this can be close to $O(1/n)$—a dramatic improvement over the $O(1/\sqrt{n})$ rate of standard MC.

**Effective Dimension:**

The practical benefit of QMC depends on the **effective dimension** of the problem. If the integrand depends strongly on only a few linear combinations of the input variables (low effective dimension), QMC can achieve near-$O(1/n)$ rates even when the nominal dimension is high. Glasserman discusses this concept and its implications for path-dependent options.

**Randomized QMC (Scrambled Sequences):**

QMC does not provide a natural error estimate (no CLT-based confidence interval). **Randomized QMC** (scrambled Sobol sequences) restores error estimation capability by running multiple scrambled replications and computing a standard error from the variance across replications.

**Practical Considerations:**

- QMC is particularly effective for smooth integrands; discontinuous payoffs may not benefit as much
- Combining QMC with variance reduction (e.g., control variates) can yield substantial further improvements

### A5.6.4 American Options: Least-Squares Monte Carlo (LSMC)

Longstaff and Schwartz (2001) introduced regression-based MC for American/Bermudan options.

**Key Idea:** Approximate the continuation value at each exercise date using regression.

**Algorithm (Glasserman Ch. 8):**

1. **Simulate paths:** Generate $n$ paths forward: $\{S_0, S_{t_1}^{(k)}, \ldots, S_{t_m}^{(k)}\}_{k=1}^{n}$

2. **Initialize at maturity:** $V_m^{(k)} = h_m(S_{t_m}^{(k)})$

3. **Backward induction:** For $i = m-1, \ldots, 1$:

   a. **Regress:** Fit continuation value on basis functions:
   $$C_i(x) \approx \sum_{j=1}^{M} \beta_{ij}\psi_j(x)$$

   b. **Exercise decision:** For each path, exercise if:
   $$h_i(S_{t_i}^{(k)}) \geq \hat{C}_i(S_{t_i}^{(k)})$$

   c. **Update cash flows:** Based on exercise decision

4. **Estimate price:** Average discounted cash flows across paths

**Common Basis Functions:**
- Powers: $1, S, S^2, S^3, \ldots$
- Laguerre polynomials
- Chebyshev polynomials

**Bias:** LSMC produces a **low-biased** estimate because the approximated continuation value leads to suboptimal exercise.

**Why Only In-The-Money Paths?**

Longstaff and Schwartz recommend regressing only over paths that are currently in the money at each exercise date. The intuition: out-of-the-money paths provide no useful information about the optimal exercise boundary (you wouldn't exercise there anyway), and including them adds noise to the regression.

### A5.6.5 Greeks via Monte Carlo

Glasserman (Chapter 7) provides the definitive treatment of sensitivity estimation in Monte Carlo. The three main approaches are finite difference bumping, the pathwise method, and the likelihood ratio method.

#### Finite Difference ("Bump and Reprice")

$$\Delta \approx \frac{V(S_0 + \epsilon) - V(S_0 - \epsilon)}{2\epsilon}$$

- Simple but requires 2 additional MC runs per Greek
- Choice of $\epsilon$ involves bias-variance tradeoff

#### Pathwise Method

Differentiate through the payoff:

$$\Delta = \mathbb{E}\left[e^{-rT}\frac{\partial H}{\partial S_T}\frac{\partial S_T}{\partial S_0}\right]$$

For GBM: $\frac{\partial S_T}{\partial S_0} = \frac{S_T}{S_0}$

**Requirement:** Payoff $H$ must be differentiable (not applicable to digitals without smoothing).

#### Likelihood Ratio Method

Differentiate the density rather than the payoff:

$$\frac{\partial}{\partial \theta}\mathbb{E}[H(X)] = \mathbb{E}\left[H(X)\frac{\partial}{\partial \theta}\log p(X;\theta)\right]$$

**Advantage:** Works for discontinuous payoffs
**Disadvantage:** Higher variance than pathwise method

### A5.6.6 Common Monte Carlo Implementation Bugs

> **Desk Reality: MC Bugs That Cost Money**
>
> 1. **Same seed for different paths:** Using the same random seed for what should be independent paths. This causes correlation between paths and wrong confidence intervals.
>
> 2. **Incorrect antithetics:** When using antithetic variates, you must generate $Z$ and $-Z$ from the **same** underlying random number. A common bug is to generate two independent normals and negate the second.
>
> 3. **Discretization bias in barriers:** Simulating at discrete time steps and checking the barrier only at those times. The probability of crossing the barrier between steps is non-zero and often significant.
>
> 4. **Memory overflow:** Storing all paths for later processing. For $n = 10^6$ paths with $N_t = 252$ time steps in double precision: $10^6 \times 252 \times 8 \approx 2$ GB. Consider on-the-fly computation instead.
>
> 5. **Forgetting discounting:** Computing $\frac{1}{n}\sum H_k$ instead of $e^{-rT}\frac{1}{n}\sum H_k$.

### A5.6.7 Multilevel Monte Carlo (MLMC)

Multilevel Monte Carlo, introduced by Giles (2008), is a technique that can dramatically reduce computational cost when discretization error and sampling error are both present. The key idea is to estimate the expectation using a **telescoping sum** across discretization levels.

**Setup:**

Let $P_\ell$ denote the option price computed with discretization level $\ell$ (e.g., $2^\ell$ time steps). We want to estimate $\mathbb{E}[P_L]$ for some fine level $L$.

**Telescoping Sum:**

$$\boxed{\mathbb{E}[P_L] = \mathbb{E}[P_0] + \sum_{\ell=1}^{L}\mathbb{E}[P_\ell - P_{\ell-1}]}$$

This is exact. The MLMC insight is that:
- $\mathbb{E}[P_0]$ can be estimated cheaply (coarse grid, many paths)
- $\mathbb{E}[P_\ell - P_{\ell-1}]$ has **small variance** when $\ell$ is large (fine grids give similar answers)
- We can allocate many paths to cheap coarse levels, few paths to expensive fine levels

**MLMC Estimator:**

$$\hat{Y} = \sum_{\ell=0}^{L} \frac{1}{n_\ell}\sum_{k=1}^{n_\ell} Y_\ell^{(k)}$$

where:
- $Y_0^{(k)} = P_0^{(k)}$
- $Y_\ell^{(k)} = P_\ell^{(k)} - P_{\ell-1}^{(k)}$ for $\ell \geq 1$

**Critical:** Each $Y_\ell^{(k)}$ uses the **same Brownian path** to compute both $P_\ell$ and $P_{\ell-1}$. This correlation makes the variance of $Y_\ell$ small.

**Complexity Theorem (Giles):**

Under certain regularity conditions, if weak convergence is $O(h)$ and variance of $P_\ell - P_{\ell-1}$ is $O(h^2)$, then MLMC achieves MSE $\epsilon^2$ with cost $O(\epsilon^{-2})$ instead of the standard MC cost $O(\epsilon^{-2-1/\alpha})$ where $\alpha$ is the weak convergence order.

**When MLMC Helps Most:**
- Payoffs with low regularity (Asian, barrier, lookback)
- Models requiring fine time-stepping (path-dependent options)
- When the cost per path is dominated by time discretization

**When MLMC Doesn't Help:**
- Exact simulation is available (standard GBM European options)
- Very smooth payoffs where standard MC is already efficient

> **Practitioner Note:** MLMC is a modern technique (2008) that isn't yet universal in production systems, but it's becoming more common for exotic pricing where discretization error dominates. The implementation requires careful handling of correlated paths across levels—using the same Brownian increments with different time-step refinements.

---

## A5.7 Fourier and Transform Methods

Fourier methods represent a fundamentally different approach to option pricing. Rather than discretizing the PDE or simulating paths, they exploit the fact that for many models, the **characteristic function** of the log-price is known analytically. This allows option prices to be computed as integrals that can be evaluated rapidly using the Fast Fourier Transform or Fourier cosine expansions.

### A5.7.1 When Fourier Methods Apply

Fourier methods are appropriate when:

1. **The model has a known characteristic function.** This includes:
   - Black–Scholes (trivially)
   - Heston stochastic volatility
   - Variance Gamma (VG)
   - CGMY (Lévy processes)
   - Bates (Heston + jumps)
   - Affine jump-diffusion models generally

2. **The option is European-style.** Fourier methods naturally handle payoffs that depend only on the terminal distribution, not on the path.

3. **The dimension is low (typically 1 or 2).** Multi-asset extensions exist but become complex.

**Why Characteristic Functions?**

The characteristic function $\phi_X(u) = \mathbb{E}[e^{iuX}]$ is the Fourier transform of the probability density. For many stochastic processes, $\phi$ has a closed-form expression even when the density does not. The option price can then be written as a Fourier integral involving $\phi$.

### A5.7.2 The Carr–Madan FFT Method

Carr and Madan (1999) showed how to price a continuum of strikes simultaneously using the FFT.

**Setup:** Let $k = \ln K$ be the log-strike, and $C_T(k)$ the call price for that strike. Carr and Madan derive:

$$C_T(k) = \frac{e^{-\alpha k}}{\pi}\int_0^\infty e^{-iuk}\psi_T(u)\,du$$

where $\alpha > 0$ is a damping parameter (needed for integrability) and $\psi_T$ is a modified characteristic function:

$$\psi_T(u) = \frac{e^{-rT}\phi_T(u - (1+\alpha)i)}{\alpha^2 + \alpha - u^2 + iu(2\alpha + 1)}$$

**FFT Implementation:**

Discretize on an equally-spaced grid and apply the FFT:
- Input: $\psi_T$ evaluated at $N$ frequencies
- Output: Call prices at $N$ log-strikes
- Complexity: $O(N \log N)$

**Limitations:**
- Strike grid is determined by the FFT (not arbitrary)
- Need to choose damping parameter $\alpha$ carefully
- Extrapolation needed for strikes outside the grid

### A5.7.3 The COS Method (Fang–Oosterlee)

The COS method, introduced by Fang and Oosterlee (2008), is an elegant Fourier cosine expansion approach that often outperforms the Carr–Madan FFT method. Oosterlee and Grzelak (Ch. 6) provide detailed exposition.

**Key Idea:** Expand the density in a Fourier cosine series on a truncated interval $[a, b]$:

$$f(x) \approx \sum_{k=0}^{N-1} A_k \cos\left(k\pi\frac{x-a}{b-a}\right)$$

where the coefficients $A_k$ are related to the characteristic function.

**COS Formula for European Options:**

$$\boxed{V = e^{-rT} \sum_{k=0}^{N-1} \text{Re}\left\{\phi\left(\frac{k\pi}{b-a}\right) e^{-ik\pi \frac{a}{b-a}}\right\} \cdot H_k}$$

where:
- $\phi(u)$ is the characteristic function of $\ln(S_T/S_0)$
- $H_k$ are **payoff coefficients** that depend on the option type (call, put)
- $[a, b]$ is the truncation range (chosen based on cumulants of the distribution)

**Payoff Coefficients for Calls:**

$$H_k = \frac{2}{b-a}\left(\chi_k(0, b) - \psi_k(0, b)\right)$$

where $\chi_k$ and $\psi_k$ involve integrals of the payoff against cosine basis functions (closed-form for vanilla options).

**Truncation Range Selection:**

The interval $[a, b]$ should capture most of the density. A common choice uses cumulants:
$$a = c_1 - L\sqrt{c_2 + \sqrt{c_4}}, \quad b = c_1 + L\sqrt{c_2 + \sqrt{c_4}}$$

where $c_1, c_2, c_4$ are the first, second, and fourth cumulants, and $L \approx 10$ for high accuracy.

**Convergence:**

For smooth densities, the COS method achieves **exponential convergence** in $N$. Oosterlee reports that $N = 64$ to $N = 256$ typically suffices for high accuracy.

**Advantages over Carr–Madan:**
- Arbitrary strike (no FFT grid constraint)
- Exponential convergence (vs polynomial for FFT)
- Simpler implementation
- Naturally handles puts, calls, digitals with different $H_k$

### A5.7.4 Characteristic Functions for Common Models

**Black–Scholes:**

$$\phi_{\text{BS}}(u) = \exp\left(iu\left(r - q - \frac{\sigma^2}{2}\right)T - \frac{\sigma^2 u^2 T}{2}\right)$$

**Heston Stochastic Volatility:**

$$\phi_{\text{Heston}}(u) = \exp\left(C(u, T) + D(u, T)v_0 + iu\ln S_0\right)$$

where $C$ and $D$ involve the Heston parameters $(\kappa, \theta, \xi, \rho, v_0)$ and complex square roots. The explicit form is given in Heston (1993) and Gatheral's book.

**Variance Gamma:**

$$\phi_{\text{VG}}(u) = \left(\frac{1}{1 - iu\theta\nu + \frac{\sigma^2\nu u^2}{2}}\right)^{T/\nu}$$

### A5.7.5 Comparison: FFT vs COS vs PDE vs MC

| Criterion | Carr–Madan FFT | COS Method | PDE/FD | Monte Carlo |
|-----------|----------------|------------|--------|-------------|
| Speed (European, 1D) | Very fast | Very fast | Fast | Slow |
| Arbitrary strike | No (grid constraint) | Yes | Yes | Yes |
| Convergence | Polynomial | Exponential | Polynomial | $O(1/\sqrt{n})$ |
| Path dependence | No | Limited | Limited | Yes |
| American exercise | No | No | Yes | LSMC |
| Implementation | Moderate | Easy | Moderate | Easy |
| Requires char func | Yes | Yes | No | No |
| Greeks | Analytic derivatives | Analytic derivatives | From grid | Noisy |

### A5.7.6 When NOT to Use Fourier Methods

Fourier methods are **not appropriate** for:

1. **American/Bermudan options:** Early exercise creates a free boundary problem incompatible with the Fourier approach (use PDE or LSMC).

2. **Path-dependent options:** Asian, lookback, barrier options require knowledge of the path, not just the terminal distribution.

3. **Models without known characteristic functions:** If you only have the SDE and no closed-form $\phi$, you can't use these methods (use MC).

4. **High-dimensional problems:** Extensions to multiple underlyings exist but become complex and lose the efficiency advantage.

> **Desk Reality: When Fourier Methods Shine**
>
> If you're pricing vanilla European options in Heston or Variance Gamma—common for equity index options—Fourier methods (especially COS) can give you 8-digit accuracy in milliseconds. For calibration loops where you need to price hundreds of strikes and maturities, this speed advantage is decisive. But the moment you have early exercise or path dependence, you're back to PDE or MC.

---

## A5.8 Method Comparison and Tradeoffs

### A5.8.1 Comprehensive Comparison Table

| Criterion | Trees | Finite Differences | Monte Carlo | Fourier |
|-----------|-------|-------------------|-------------|---------|
| **Dimensionality** | 1–2 factors | 1–3 factors | Any $d$ | 1–2 factors |
| **American options** | Natural | Natural (LCP) | LSMC (harder) | No |
| **Path dependence** | Limited | Limited | Natural | No |
| **Barriers** | Moderate | Good | Needs correction | No |
| **Greeks** | Stable from grid | Stable from grid | Noisy (variance) | Analytic |
| **Convergence** | $O(1/N)$ | $O(1/N)$ or $O(1/N^2)$ | $O(1/\sqrt{n})$ | Exponential |
| **Implementation** | Simple | Moderate | Simple | Moderate |
| **Parallelization** | Limited | Limited | Excellent | Moderate |
| **Memory** | $O(N)$ | $O(M \cdot N)$ | $O(n)$ or $O(1)$ | $O(N)$ |
| **Requires char func** | No | No | No | Yes |

### A5.8.2 When Each Method Excels

**Use Trees When:**
- Single underlying asset
- American-style exercise
- Need quick, intuitive implementation
- Interest rate models with mean reversion (Hull–White)

**Use Finite Differences When:**
- Need high accuracy for single/low-dim problems
- Want stable Greeks directly from the grid
- American options with barriers
- Crank–Nicolson for second-order accuracy

**Use Monte Carlo When:**
- High-dimensional problems ($d > 3$)
- Path-dependent payoffs (Asians, lookbacks)
- Complex model dynamics
- Parallelization is available
- European-style (or Bermudan with LSMC)

**Use Fourier Methods When:**
- European options in models with known characteristic functions
- Speed is critical (calibration loops)
- Need high precision for vanilla options
- Pricing many strikes simultaneously (Carr–Madan FFT)

### A5.8.3 Computational Cost Estimates

| Method | Typical Cost for Vanilla European | Notes |
|--------|----------------------------------|-------|
| Black–Scholes closed-form | < 1 μs | When available |
| Fourier (COS, $N=128$) | ~ 10 μs | Heston, VG |
| Binomial tree ($N=500$) | ~ 100 μs | American options |
| Implicit FD ($M=100, N=100$) | ~ 1 ms | Good for Americans |
| Monte Carlo ($n=10^5$) | ~ 10 ms | European; more for path-dependent |
| LSMC ($n=10^5$, 50 exercise dates) | ~ 100 ms | Bermudan options |

### A5.8.4 Hybrid Approaches

- **MC with PDE for early exercise:** Use PDE to determine exercise boundary, MC for valuation
- **Quasi-MC with variance reduction:** Combine Sobol sequences with control variates
- **Adaptive mesh:** Figlewski–Gao adaptive mesh model grafts fine grids onto coarse grids
- **MLMC with ADI:** Multilevel Monte Carlo combined with ADI for exotic rates products

---

## A5.9 Worked Examples

### Example A5.1: CRR Binomial Tree — American Put

**Setup:** $S_0 = 50$, $K = 50$, $T = 5$ months, $r = 10\%$, $\sigma = 40\%$, $q = 0$

**Step 1: Compute parameters**
$$\Delta t = 5/12 = 0.4167 \text{ years (using 2 steps, so } \Delta t = 0.2083\text{)}$$

Let's use $N = 5$ monthly steps, so $\Delta t = 1/12$:

$$u = e^{0.40\sqrt{1/12}} = e^{0.1155} = 1.1224$$
$$d = 1/u = 0.8909$$
$$p = \frac{e^{0.10/12} - 0.8909}{1.1224 - 0.8909} = \frac{1.0084 - 0.8909}{0.2315} = 0.507$$

**Step 2: Build the tree forward**

At each node, compute stock prices. At node $(i,j)$: $S_{i,j} = 50 \times 1.1224^j \times 0.8909^{i-j}$

**Step 3: Backward induction with early exercise check**

At maturity: $f_{5,j} = \max(50 - S_{5,j}, 0)$

At earlier nodes:
$$f_{i,j} = \max\left\{50 - S_{i,j}, \; e^{-0.10/12}[0.507 f_{i+1,j+1} + 0.493 f_{i+1,j}]\right\}$$

**Result:** The American put price is approximately $\$4.49$ (Hull Example).

---

### Example A5.2: Explicit Finite Difference for American Put

**Setup:** Same as Example A5.1, with grid $M = 20$ (stock price intervals), $N = 10$ (time steps).

**Step 1: Set up grid**
- $\Delta S = 100/20 = 5$
- $\Delta t = (5/12)/10 = 1/24$ years

**Step 2: Compute coefficients** for $j = 1, \ldots, 19$:

$$a_j^* = \frac{1}{1+0.10/24}\left(-\frac{0.10 \cdot j}{2 \cdot 24} + \frac{0.16 \cdot j^2}{2 \cdot 24}\right)$$

(Similar for $b_j^*, c_j^*$)

**Step 3: Apply recursion**

$$f_{i,j} = a_j^* f_{i+1,j-1} + b_j^* f_{i+1,j} + c_j^* f_{i+1,j+1}$$

Then apply early exercise: $f_{i,j} \leftarrow \max(f_{i,j}, 50 - j\Delta S)$

**Step 4: Read off price**

The price at $S = 50$ (i.e., $j = 10$) gives the option value: $\approx \$4.26$ (Hull Table 21.5).

**Stability Check:** Need $1 - \sigma^2 j^2 \Delta t > 0$, so $j < \sqrt{24/(0.16)} \approx 12.2$. The explicit method fails for $j \ge 13$.

---

### Example A5.3: Implicit Finite Difference with Control Variate

**Setup:** Same American put as above.

**Step 1:** Solve implicit system (tridiagonal) to get American put value: $\$4.07$

**Step 2:** Same grid gives European put: $\$3.91$

**Step 3:** Black–Scholes European put: $\$4.08$

**Step 4:** Control variate adjustment:
$$\hat{V}_{\text{American}} = 4.07 + (4.08 - 3.91) = \$4.24$$

This is more accurate than the raw grid estimate.

---

### Example A5.4: Monte Carlo for European Call

**Setup:** $S_0 = 100$, $K = 100$, $T = 1$, $r = 5\%$, $\sigma = 20\%$, $q = 0$ (no dividends)

**Step 1: Generate $n = 10{,}000$ paths**

For each path $k$, use exact GBM simulation:
$$S_T^{(k)} = S_0 \exp\left[\left(r - q - \frac{\sigma^2}{2}\right)T + \sigma\sqrt{T} Z^{(k)}\right]$$

With our parameters: $r - q - \sigma^2/2 = 0.05 - 0 - 0.02 = 0.03$, so:
$$S_T^{(k)} = 100 \exp\left[0.03 \times 1 + 0.20 \times 1 \times Z^{(k)}\right]$$
where $Z^{(k)} \sim N(0,1)$

**Step 2: Compute payoffs**
$$H^{(k)} = \max(S_T^{(k)} - 100, 0)$$

**Step 3: Average and discount**
$$\hat{V} = e^{-0.05} \times \frac{1}{10000}\sum_{k=1}^{10000} H^{(k)}$$

**Step 4: Standard error**
$$\text{SE} = \frac{\hat{\sigma}_H}{\sqrt{10000}}$$

**Typical Result:** $\hat{V} \approx 10.45$ with SE $\approx 0.12$

**Black–Scholes:** $C = 10.45$ (matches within SE)

---

### Example A5.5: Antithetic Variates

**Using the same setup as Example A5.4** ($S_0 = 100$, $K = 100$, $r = 5\%$, $\sigma = 20\%$, $q = 0$):

For each pair $(Z, -Z)$, since $r - q - \sigma^2/2 = 0.03$:
- Path 1: $S_T^{(+)} = 100\exp[0.03 + 0.20 Z]$
- Path 2: $S_T^{(-)} = 100\exp[0.03 - 0.20 Z]$

Average: $\hat{V}_{\text{pair}} = \frac{1}{2}e^{-rT}\left[\max(S_T^{(+)} - 100, 0) + \max(S_T^{(-)} - 100, 0)\right]$

**Result:** With 5,000 pairs, variance reduction of approximately 75% for at-the-money calls.

---

### Example A5.6: Control Variate for Call

**Using the same setup as Example A5.4.**

**Control:** Use $S_T$ as control variate with $\mathbb{E}[S_T] = S_0 e^{(r-q)T} = 100 e^{0.05} = 105.13$ (since $q = 0$).

**Estimator:**
$$\hat{V}_{\text{CV}} = \hat{V} - \hat{\beta}\left(\bar{S}_T - 105.13\right)$$

where $\hat{\beta}$ is estimated from sample covariance.

**Result:** Variance reduction of 90%+ for in-the-money calls.

---

### Example A5.7: LSMC for Bermudan Put

**Setup:** $S_0 = 36$, $K = 40$, $T = 1$, $r = 6\%$, $\sigma = 20\%$, 50 exercise dates

**Step 1: Simulate 50,000 paths**

Generate price paths at each exercise date using exact GBM simulation.

**Step 2: At maturity**
$$V_{50}^{(k)} = \max(40 - S_{50}^{(k)}, 0)$$

**Step 3: Backward induction with regression**

At each date $i$, for paths where immediate exercise is in-the-money:
- **Regress** discounted continuation values on basis functions: $1, S, S^2$
- **Fitted continuation value:** $\hat{C}_i(S) = \beta_0 + \beta_1 S + \beta_2 S^2$
- **Exercise if:** $40 - S_i^{(k)} > \hat{C}_i(S_i^{(k)})$

**Step 4: Average discounted cash flows**

**Result:** Bermudan put price $\approx \$4.47$ (vs. European $\approx \$3.84$)

---

### Example A5.8: Pathwise Delta

**For European call under GBM:**

$$\Delta = \mathbb{E}\left[e^{-rT}\mathbf{1}_{S_T > K}\frac{S_T}{S_0}\right]$$

For each path:
$$\Delta^{(k)} = e^{-rT}\mathbf{1}_{S_T^{(k)} > K}\frac{S_T^{(k)}}{S_0}$$

Average gives an unbiased estimate of delta.

**Result:** $\hat{\Delta} \approx 0.64$ (matches Black–Scholes $N(d_1) \approx 0.64$)

---

### Example A5.9: Trinomial Tree for Hull–White Model

**Setup:** Hull–White model $dr = [\theta(t) - 0.1r]\,dt + 0.01\,dW$

**Step 1: Build tree for $x = r - \alpha(t)$**

With mean reversion $a = 0.1$ and $\sigma = 0.01$:
- $\Delta x = \sigma\sqrt{3\Delta t}$
- Probabilities depend on $j$ to incorporate mean reversion

**Step 2: Shift tree**

Use forward induction to calibrate $\alpha(t)$ to the initial term structure.

**Step 3: Price callable bond**

Backward induction with call constraint at each exercise date.

---

### Example A5.10: Crank–Nicolson with Rannacher Stepping

**Setup:** Digital call option, $S_0 = 100$, $K = 100$

**Problem:** Standard C-N produces oscillations near strike at maturity.

**Solution:**
1. Use 4 fully implicit steps for the first $4\Delta t$ backward from maturity
2. Switch to C-N for remaining time steps

**Result:** Smooth solution without spurious oscillations.

---

### Example A5.11: COS Method for Heston European Call

**Setup:** Heston model parameters:
- $S_0 = 100$, $K = 100$, $T = 1$, $r = 5\%$
- $v_0 = 0.04$ (initial variance)
- $\kappa = 2.0$ (mean reversion speed)
- $\theta = 0.04$ (long-run variance)
- $\xi = 0.3$ (vol-of-vol)
- $\rho = -0.7$ (correlation)

**Step 1: Compute cumulants for truncation range**

Using the Heston characteristic function, compute first four cumulants $c_1, c_2, c_4$ and set:
$$a = c_1 - 10\sqrt{c_2 + \sqrt{c_4}}, \quad b = c_1 + 10\sqrt{c_2 + \sqrt{c_4}}$$

Typical result: $a \approx -0.8$, $b \approx 0.8$ for log-price.

**Step 2: Evaluate characteristic function**

For $k = 0, 1, \ldots, N-1$ with $N = 128$, compute:
$$\phi_{\text{Heston}}\left(\frac{k\pi}{b-a}\right)$$

using the Heston formula (complex arithmetic).

**Step 3: Apply COS formula**

$$V = e^{-rT} \sum_{k=0}^{127} \text{Re}\left\{\phi\left(\frac{k\pi}{b-a}\right) e^{-ik\pi \frac{a}{b-a}}\right\} \cdot H_k$$

where $H_k$ are the call payoff coefficients.

**Result:** Call price $\approx \$10.32$ (computed in < 1 ms)

**Convergence Check:**
- $N = 32$: $\$10.317$
- $N = 64$: $\$10.3198$
- $N = 128$: $\$10.3199$
- $N = 256$: $\$10.3199$

Exponential convergence is evident: accuracy stabilizes rapidly.

---

### Example A5.12: 2D ADI for Basket Option

**Setup:** Two correlated stocks:
- $S_1(0) = S_2(0) = 100$
- $\sigma_1 = 20\%$, $\sigma_2 = 25\%$
- $\rho = 0.5$ (correlation)
- Payoff: $\max(0.5 S_1 + 0.5 S_2 - K, 0)$
- $K = 100$, $T = 1$, $r = 5\%$

**Step 1: Set up 2D grid**

Transform to log-coordinates: $x_1 = \ln S_1$, $x_2 = \ln S_2$

Grid: $50 \times 50$ points in $(x_1, x_2)$, 100 time steps.

**Step 2: Split operators**

$$\mathcal{L} = \mathcal{L}_1 + \mathcal{L}_2 + \mathcal{L}_{12}$$

where:
- $\mathcal{L}_1$: terms involving only $x_1$ derivatives
- $\mathcal{L}_2$: terms involving only $x_2$ derivatives
- $\mathcal{L}_{12}$: cross-derivative from correlation

**Step 3: Douglas–Rachford ADI**

At each time step:
1. **Predictor:** Explicit step using all operators
2. **Corrector 1:** Implicit in $x_1$ direction (solve $50$ tridiagonal systems of size $50$)
3. **Corrector 2:** Implicit in $x_2$ direction (solve $50$ tridiagonal systems of size $50$)

Cross-derivative $\mathcal{L}_{12}$ is treated explicitly or via Craig–Sneyd.

**Result:** Basket call price $\approx \$13.47$

**Comparison:** MC with $10^6$ paths gives $\$13.46 \pm 0.02$. ADI is faster and deterministic.

---

### Example A5.13: Barrier Option with Continuity Correction

**Setup:** Down-and-out call
- $S_0 = 100$, $K = 100$, $T = 1$, $r = 5\%$, $\sigma = 20\%$
- Barrier $B = 90$, monitored **daily** (252 observations)

**Problem:** FD methods naturally handle continuous barriers. For discrete monitoring, we need a correction.

**Step 1: Price with continuous barrier (FD)**

Set $V(B, t) = 0$ as boundary condition. Result: $V_{\text{cont}} = \$6.21$

**Step 2: Apply Broadie–Glasserman–Kou correction**

$$B_{\text{eff}} = B \cdot e^{-\beta\sigma\sqrt{T/m}} = 90 \cdot e^{-0.5826 \times 0.20 \times \sqrt{1/252}}$$
$$B_{\text{eff}} = 90 \cdot e^{-0.00734} = 90 \cdot 0.9927 = 89.34$$

**Step 3: Re-price with effective barrier**

Use $B_{\text{eff}} = 89.34$ instead of $B = 90$. Result: $V_{\text{discrete}} = \$6.58$

**Verification:** MC simulation with explicit daily monitoring gives $\$6.55 \pm 0.05$.

**Interpretation:** The discrete barrier is "further away" in probability terms because the price has fewer chances to cross it. The effective barrier correction captures this effect elegantly.

---

## A5.10 Practical Notes

### A5.10.1 Common Pitfalls

**Trees:**
- Using too few time steps ($N < 50$) for accurate pricing
- Ignoring odd/even oscillation in convergence
- Forgetting to adjust for dividends in CRR formulas

**Finite Differences:**
- Violating explicit method stability conditions
- Boundary conditions too close to the region of interest
- Using C-N for discontinuous payoffs without smoothing
- Barrier not falling on a grid line

**Monte Carlo:**
- Confusing tight confidence intervals with accuracy (bias!)
- Using too few paths for Greeks (noisy estimates)
- Not checking variance reduction effectiveness
- Using the same random seed incorrectly

**Fourier Methods:**
- Using Fourier for American options (won't work)
- Incorrect truncation range leading to errors
- Numerical issues with characteristic function for extreme parameters

### A5.10.2 Implementation Tips

**General:**
- Always benchmark against known closed-form solutions
- Use control variates whenever a similar closed-form exists
- Store random seeds for reproducibility

**Trees:**
- Store only two columns at a time (memory efficiency)
- Precompute probabilities and discount factors

**Finite Differences:**
- Use the $Z = \ln S$ transformation for GBM
- Implement Thomas algorithm for tridiagonal systems
- Consider ADI for 2D problems

**Monte Carlo:**
- Parallelize across paths (embarrassingly parallel)
- Use antithetics by default
- Quasi-MC often outperforms pseudo-random

**Fourier:**
- Use COS method for single strikes
- Use Carr–Madan FFT for strike grids (calibration)
- Verify characteristic function implementation with limiting cases

### A5.10.3 Sanity Checks

1. **Put-call parity:** Does your European call/put satisfy parity?
2. **Boundary limits:** Does price → intrinsic as $S \to 0$ or $S \to \infty$?
3. **Convergence:** Does price stabilize as $N, M, n$ increase?
4. **American ≥ European:** Is your American price at least the European price?
5. **Greeks signs:** Is delta positive for calls? Is gamma positive?
6. **Zero vol limit:** Does option price → max(intrinsic, 0) as $\sigma \to 0$?
7. **Characteristic function:** Does $\phi(0) = 1$? Is $|\phi(u)| \le 1$?

### A5.10.4 Testing and Verification

**Benchmark Cases:**
- Black–Scholes: Closed-form prices for all Greeks
- Zero volatility: Price should equal discounted intrinsic value
- Zero interest rate: Put-call parity simplifies
- Deep ITM/OTM: Should match intrinsic value

**Convergence Studies:**
1. Run with grid size $N$, then $2N$, then $4N$
2. Check that error ratio matches expected order: $\frac{e_N}{e_{2N}} \approx 2^k$ for order $k$
3. Plot log(error) vs log($N$) — slope should equal $-k$

**Regression Testing:**
- Store baseline prices for known cases
- Alert when prices change beyond tolerance
- Essential for code refactoring

### A5.10.5 Performance Optimization

**Precomputation:**
- Compute transition probabilities once, store in arrays
- Precompute discount factors at all time steps
- For Fourier, precompute $e^{ik\pi a/(b-a)}$ factors

**Memory Layout:**
- Use row-major or column-major consistently (language-dependent)
- For FD, store only current and previous time slice
- For MC, process paths in batches to fit in cache

**Vectorization:**
- Many MC operations are element-wise (vectorize!)
- BLAS libraries for matrix operations
- SIMD intrinsics for inner loops

**When to Use Sparse vs Dense:**
- Tridiagonal: Always use Thomas algorithm ($O(n)$)
- Banded with width $w$: Use banded solver if $w \ll n$
- General sparse: Use iterative methods (CG, GMRES)
- Dense: Only for small systems ($n < 1000$)

### A5.10.6 Common Production Bugs

> **Desk Reality: Production Numerical Bugs**
>
> 1. **Day count mismatch:** Using ACT/365 when the contract specifies ACT/360. This creates small but persistent pricing errors.
>
> 2. **Wrong discounting curve:** Using OIS when the trade has LIBOR discounting (legacy), or vice versa.
>
> 3. **Settlement timing:** Off-by-one errors in settlement dates can shift discounting by a day.
>
> 4. **Integer overflow:** Large grids with 32-bit indexing: $1000 \times 1000 \times 1000 = 10^9 > 2^{31}$.
>
> 5. **NaN propagation:** One NaN in a grid infects the entire backward induction. Always check inputs.
>
> 6. **Thread safety:** Random number generators that aren't thread-safe produce correlated "independent" paths.

---

## Summary

This appendix covered four fundamental numerical methods for derivative pricing:

1. **Trees (Binomial/Trinomial):** Discrete approximations of the underlying process; natural for American options in low dimensions; CRR is the standard construction; trinomial trees are equivalent to explicit FD.

2. **Finite Differences:** Discretize the pricing PDE; implicit methods are unconditionally stable; explicit methods connect to trees but have stability restrictions; Crank–Nicolson offers second-order accuracy; ADI extends to 2–3 dimensions.

3. **Monte Carlo:** Sample paths from risk-neutral dynamics; the only practical method for high-dimensional problems; variance reduction is essential; LSMC extends MC to American options; MLMC improves efficiency for discretization-dominated problems.

4. **Fourier Methods:** Exploit known characteristic functions for rapid European option pricing; the COS method achieves exponential convergence; the Carr–Madan FFT prices strike grids efficiently.

**Key Principles:**

- **Lax Equivalence:** Consistency + Stability ⇒ Convergence
- **Strong vs Weak Convergence:** Strong (pathwise) matters for hedging; weak (distributional) suffices for pricing
- **Method Selection:** Dimension, exercise style, path dependence, and characteristic function availability are the primary drivers

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Stability** | Scheme remains bounded as discretization refines | Unstable schemes diverge |
| **Consistency** | Discrete operator approximates continuous | Ensures correct limiting PDE |
| **Convergence** | Numerical solution → true solution | The ultimate goal |
| **Lax Equivalence** | Consistency + Stability ⇒ Convergence | Fundamental theorem |
| **Strong Convergence** | Pathwise error $\to 0$ | Matters for hedging |
| **Weak Convergence** | Distributional error $\to 0$ | Suffices for pricing |
| **CFL Condition** | Stability constraint on $\Delta t / \Delta x^2$ | Explicit method restriction |
| **Recombining Tree** | $ud = 1$ so paths rejoin | Polynomial vs exponential nodes |
| **LSMC** | Regression-based MC for Americans | Key technique for high-dim |
| **Variance Reduction** | Techniques to narrow CI | Makes MC practical |
| **MLMC** | Telescoping estimator across levels | Beats $O(1/\sqrt{n})$ |
| **COS Method** | Fourier cosine pricing | Exponential convergence |
| **Characteristic Function** | $\phi_X(u) = \mathbb{E}[e^{iuX}]$ | Enables Fourier methods |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What are the CRR binomial tree parameters? | $u = e^{\sigma\sqrt{\Delta t}}$, $d = 1/u$, $p = (e^{(r-q)\Delta t} - d)/(u-d)$ |
| 2 | What does "recombining tree" mean? | $ud = 1$, so up-then-down equals down-then-up; gives $O(N)$ terminal nodes |
| 3 | State the Lax Equivalence Theorem | For well-posed linear PDEs: Consistency + Stability ⇒ Convergence |
| 4 | What is the stability condition for explicit FD? | $\Delta t < 1/(\sigma^2 j_{\max}^2)$; all "probabilities" must be non-negative |
| 5 | Why is implicit FD unconditionally stable? | It inverts a tridiagonal matrix at each step, damping all modes |
| 6 | What is the Crank–Nicolson scheme? | Average of implicit and explicit: second-order in time, unconditionally stable |
| 7 | Why does C-N oscillate for digital options? | Discontinuous payoffs create high-frequency modes that C-N doesn't damp |
| 8 | What is Rannacher stepping? | Use 2-4 implicit steps near maturity to damp oscillations, then C-N |
| 9 | How does MC error scale with sample size? | Standard error $\propto 1/\sqrt{n}$ |
| 10 | What is antithetic variance reduction? | For each $Z$, also use $-Z$; induces negative correlation |
| 11 | Explain control variates | Use a correlated quantity with known mean to reduce variance |
| 12 | What is the key idea of LSMC? | Regress continuation value on basis functions to make exercise decisions |
| 13 | Why is LSMC biased low? | Suboptimal exercise due to approximate continuation values |
| 14 | What are the two main MC methods for Greeks? | Pathwise (differentiate payoff) and likelihood ratio (differentiate density) |
| 15 | When does pathwise method fail? | For discontinuous payoffs (e.g., digitals) |
| 16 | What is the explicit FD equivalent to? | A trinomial tree |
| 17 | What limits trees and FD to low dimensions? | Grid/node count scales as $N^d$ (curse of dimensionality) |
| 18 | Why is MC preferred for path-dependent options? | Paths naturally encode history; no state augmentation needed |
| 19 | What is quasi-Monte Carlo? | Uses low-discrepancy sequences instead of pseudo-random; can achieve $O(1/n)$ |
| 20 | How do you get delta from a tree? | $(f_{0,1} - f_{0,-1})/(S_u - S_d)$ at the first step |
| 21 | Define strong convergence for SDE simulation | $\mathbb{E}[|\hat{X}(T) - X(T)|] \leq Ch^\beta$ with strong order $\beta$ |
| 22 | Define weak convergence for SDE simulation | $|\mathbb{E}[f(\hat{X})] - \mathbb{E}[f(X)]| \leq Ch^\beta$ with weak order $\beta$ |
| 23 | What are Euler-Maruyama convergence orders? | Strong order 1/2, weak order 1 |
| 24 | What are Milstein convergence orders? | Strong order 1, weak order 1 |
| 25 | What is the Thomas algorithm? | $O(M)$ solution of tridiagonal systems; used in implicit FD |
| 26 | Why transform to $Z = \ln S$ in FD? | Makes coefficients independent of $j$; improves stability |
| 27 | What is ADI? | Alternating Direction Implicit: splits 2D problem into 1D sweeps |
| 28 | What is the characteristic function? | $\phi_X(u) = \mathbb{E}[e^{iuX}]$; Fourier transform of the density |
| 29 | What is the COS method? | Fourier cosine expansion pricing; exponential convergence for smooth densities |
| 30 | When can you use Fourier pricing methods? | European options in models with known characteristic functions |
| 31 | What is the key idea of MLMC? | Telescoping sum: $\mathbb{E}[P_L] = \mathbb{E}[P_0] + \sum_{\ell}\mathbb{E}[P_\ell - P_{\ell-1}]$ |
| 32 | What is the Broadie-Glasserman-Kou correction? | $B_{\text{eff}} = B \cdot e^{-\beta\sigma\sqrt{T/m}}$ for discrete barrier monitoring |

---

## Mini Problem Set

### Warm-Up (Conceptual)

**Problem 1:** Explain why Monte Carlo becomes the only practical method when $d > 3$.

**Problem 2:** A colleague says "I used explicit FD and got negative option prices." What went wrong?

**Problem 3:** Why does variance reduction matter more for out-of-the-money options than at-the-money?

**Problem 4:** Explain the difference between strong and weak convergence. When does each matter?

### Computational (Trees)

**Problem 5:** Construct a 3-step CRR tree for a European put with $S_0 = 100$, $K = 95$, $T = 0.25$, $r = 5\%$, $\sigma = 30\%$. Compute the option price.

**Problem 6:** For the tree in Problem 5, convert to an American put. At which nodes would early exercise occur?

**Problem 7:** Build a 2-step trinomial tree for the same option. Compare the price to the binomial result.

### Computational (Finite Differences)

**Problem 8:** Set up the implicit FD difference equation for a European call. What are $a_j$, $b_j$, $c_j$?

**Problem 9:** For explicit FD with $\sigma = 40\%$, $\Delta t = 1/12$, $\Delta S = 5$, determine the maximum $j$ (stock price level) for which the method is stable.

**Problem 10:** Explain why transforming to $Z = \ln S$ makes explicit FD equivalent to a trinomial tree with constant probabilities.

### Computational (Monte Carlo)

**Problem 11:** Write pseudocode for pricing an Asian call option using Monte Carlo.

**Problem 12:** You estimate a call price with $n = 1000$ paths and get SE = 0.50. How many paths do you need to reduce SE to 0.05?

**Problem 13:** Derive the pathwise delta estimator for a European call under GBM.

### Advanced

**Problem 14:** Describe how to combine antithetic variates and control variates in a single MC simulation.

**Problem 15:** In LSMC, why do we only regress over "in-the-money" paths at each exercise date?

**Problem 16:** Explain the bias-variance tradeoff when using finite-difference bumping for Greeks in Monte Carlo.

**Problem 17:** A barrier option has payoff that depends on whether $\max_{t \le T} S_t > B$. Can you use standard FD? What modifications are needed?

**Problem 18:** When does the COS method outperform Carr–Madan FFT, and when is the reverse true?

### Solution Sketches

**Problem 1:** Grid methods scale as $O(N^d)$ in memory and computation; for $d > 3$, this becomes prohibitive. MC scales as $O(nd)$—linear in dimension.

**Problem 2:** Stability condition violated. Need $\Delta t < 1/(\sigma^2 j^2)$. Either decrease $\Delta t$, use implicit method, or transform to $\ln S$.

**Problem 4:** Strong convergence measures pathwise accuracy: does the simulated path match the true path? Weak convergence measures distributional accuracy: does the distribution of the simulated value match the true distribution? Strong matters for hedging simulations (tracking P&L along paths); weak suffices for pricing (only need $\mathbb{E}[H(S_T)]$).

**Problem 5 Sketch:**
- $\Delta t = 0.25/3 = 0.0833$
- $u = e^{0.30\sqrt{0.0833}} = 1.0905$
- $d = 0.9170$
- $p = (e^{0.05 \times 0.0833} - 0.9170)/(1.0905 - 0.9170) = 0.525$
- Build tree, compute terminal payoffs, backward induct.

**Problem 12:** SE scales as $1/\sqrt{n}$. Need $0.50/\sqrt{n'} = 0.05$, so $n' = 1000 \times 100 = 100{,}000$ paths.

**Problem 15:** Out-of-the-money paths provide no useful information about the optimal exercise boundary (you wouldn't exercise there anyway), and including them adds noise to the regression without improving the exercise decision.

**Problem 18:** COS method: single strike, arbitrary strike value, exponential convergence. Carr–Madan FFT: strike grid (calibration), $O(N \log N)$ for $N$ strikes, but strikes constrained to FFT grid. Use COS for pricing individual options; use FFT when calibrating to many strikes simultaneously.

---

## Source Map

### (A) Book-Verified Facts

| Fact | Source |
|------|--------|
| Risk-neutral pricing identity | Andersen–Piterbarg Vol 1 |
| Black–Scholes call formula with dividend yield | Standard derivation |
| CRR binomial parameters and risk-neutral probability | Hull Ch 13 |
| Binomial tree convergence to BSM | Hull Ch 13 Appendix |
| Trinomial tree probabilities | Hull Section 21.4 |
| Explicit FD equivalent to trinomial tree | Hull Section 21.8 |
| Implicit FD coefficients $a_j$, $b_j$, $c_j$ | Hull Section 21.8 |
| Explicit FD coefficients $a_j^*$, $b_j^*$, $c_j^*$ | Hull Section 21.8 |
| Crank–Nicolson as average of implicit/explicit | Hull Section 21.8 |
| Crank–Nicolson oscillation issues | Duffy, Andersen–Piterbarg |
| Rannacher stepping (2–4 implicit steps near maturity) | Andersen–Piterbarg Vol 1 Section 2.9 |
| ADI methods (Peaceman–Rachford, Douglas–Rachford) | Andersen–Piterbarg Vol 1 Sections 2.10–2.12 |
| ADI complexity $O(m_1 m_2)$ vs direct C-N $O((m_1 m_2)^{5/4})$ | Andersen–Piterbarg Vol 1 |
| Craig–Sneyd, Hundsdorfer–Verwer schemes | Andersen–Piterbarg Vol 1 Section 2.11 |
| Stability definition | Duffy |
| Consistency definition | Duffy |
| Convergence definition | Andersen–Piterbarg |
| Lax equivalence theorem | Andersen–Piterbarg Vol 1 |
| Strong convergence definition and orders | Glasserman Ch 6 pp. 339-365 |
| Weak convergence definition and orders | Glasserman Ch 6 pp. 339-365 |
| Euler-Maruyama: strong order 1/2, weak order 1 | Glasserman Ch 6 |
| Milstein: strong order 1, weak order 1 | Glasserman Ch 6 |
| Tridiagonal $\theta$-scheme complexity $O(mn)$ | Andersen–Piterbarg |
| FD suitable for 1-3 factors | Andersen–Piterbarg |
| MC difficulty with early exercise | Andersen–Piterbarg |
| MC convergence rate $O(1/\sqrt{n})$ | Standard MC theory |
| Antithetic variates technique | Hull Section 21.7, Glasserman Ch 4 |
| Control variate technique | Hull Section 21.7, Glasserman Ch 4 |
| Importance sampling | Glasserman Ch 4 |
| Stratified sampling | Glasserman Ch 4 |
| Quasi-Monte Carlo (Sobol, Halton sequences) | Glasserman Ch 5 |
| QMC convergence rate $O((\log n)^d / n)$ | Glasserman Ch 5 |
| Effective dimension concept | Glasserman Ch 5 |
| LSMC algorithm and bias properties | Glasserman Ch 8, Longstaff–Schwartz (2001) |
| Pathwise method for Greeks | Glasserman Ch 7 |
| Likelihood ratio method | Glasserman Ch 7 |
| COS method formula and exponential convergence | Oosterlee & Grzelak Ch 6 |
| Carr–Madan FFT method | Carr & Madan (1999) |
| MLMC telescoping estimator | Giles (2008), Oosterlee Ch 8 |
| Broadie–Glasserman–Kou barrier correction | Glasserman Ch 6 |
| American put free boundary constraint | Standard references |

### (B) Claude-Extended Content (Priority 2)

| Content | Context |
|---------|---------|
| Production implementation bugs section | Extended from general software engineering and quantitative finance knowledge |
| Testing and verification strategies | Extended from general numerical methods practice |
| Performance optimization tips | Extended from computational finance practice |
| "Desk Reality" boxes throughout | Practitioner color added from general fixed income/derivatives knowledge |
| When strong vs weak convergence matters (hedging vs pricing) | Extended from Glasserman's treatment with practical interpretation |
| MLMC practical notes | Extended from Giles and Oosterlee with implementation guidance |

### (C) Reasoned Inference (Derived from A or B)

- FD curse of dimensionality ($N_x^d$ scaling) derived from grid structure
- MC linear scaling in dimension derived from simulation structure
- Stability condition $\Delta t < 1/(\sigma^2 j^2)$ derived from requiring positive probabilities in explicit FD
- LSMC low bias follows from suboptimality of approximate continuation values
- ADI unconditional stability inherited from implicit method structure (with caveats for cross-derivatives)
- COS method comparison with Carr–Madan derived from their respective properties
- Richardson extrapolation formula derived from Taylor expansion analysis

### (D) Flagged Uncertainties

- **Cross-derivative treatment in ADI:** The standard ADI schemes require modification for mixed partial derivatives arising from correlation. The exact formulation (Craig–Sneyd, Hundsdorfer–Verwer, etc.) depends on the specific problem. I'm not certain about the optimal choice without further source verification for specific model types.

- **Randomized QMC error estimation:** The precise variance formulas for scrambled Sobol sequences depend on the scrambling method used. The general principle is well-established in Glasserman, but implementation details vary by library.

- **MLMC complexity constants:** The Giles complexity theorem states asymptotic behavior, but the constants and crossover points depend on the specific problem. I'm not sure about exact efficiency thresholds for when MLMC beats standard MC.

- **Heston characteristic function branch cut:** The Heston characteristic function involves complex logarithms and square roots, which can have branch cut issues for certain parameter combinations. The exact handling varies by implementation.

---

*Last Updated: January 2026*
