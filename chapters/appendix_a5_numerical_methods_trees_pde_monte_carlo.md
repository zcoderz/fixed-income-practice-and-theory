# Appendix A5: Numerical Methods — Trees, PDE / Finite Differences, and Monte Carlo

---

## Introduction

When a trader asks for the price of a Bermudan swaption, an Asian call on a basket of equities, or a callable range accrual, there is rarely a closed-form answer waiting in a textbook. The payoff depends on too many underlyings, the exercise rights create free-boundary conditions, or the path dependence makes analytical tractability impossible. At that point, the quant reaches for one of three computational workhorses: **trees**, **finite differences**, or **Monte Carlo**. Choosing poorly—a tree for a 20-factor model, or naïve Monte Carlo for an American put—can mean hours of runtime, unstable Greeks, or simply wrong prices.

This appendix provides a practitioner's map for navigating that choice. We begin with the foundational concepts that underpin all three methods: the pricing identity, state dimension, path dependence, and the critical trinity of **stability**, **consistency**, and **convergence**. The Lax Equivalence Theorem—"consistency plus stability implies convergence"—is the theoretical anchor that tells us when a discretization scheme actually works.

We then develop each method in detail. **Section A5.4** covers binomial and trinomial trees, from the CRR construction through Hull–White interest rate trees. **Section A5.5** presents finite difference methods: explicit, implicit, Crank–Nicolson, and the Alternating Direction Implicit (ADI) schemes essential for multi-factor problems. **Section A5.6** addresses Monte Carlo, including variance reduction techniques, the Longstaff–Schwartz algorithm for American options, and pathwise versus likelihood ratio methods for Greeks. Throughout, we emphasize the decision framework (**Section A5.3**): which features of your problem—dimension, exercise style, path dependence—point to which method.

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

All three numerical methods (trees, PDE/FD, MC) are different approximations to the same object: the discounted expectation (or equivalently the PDE solution under conditions where Feynman–Kac applies).

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

---

### A5.2.7 Complexity Scaling

**Formal Objects:**

- FD grid: $N_x$ points per dimension; $N_t$ time steps.
- MC: $n$ paths and $N_t$ time steps per path (time-discretized simulation).

**Source-Backed Anchors:**

- A standard tridiagonal (1D) $\theta$-scheme with $m$ spatial points and $n$ time steps has complexity $O(mn)$ (Andersen–Piterbarg).
- Finite differences are "suitable for 1, 2 and possibly 3 factors" and beyond that Monte Carlo is typical (Andersen–Piterbarg).
- Monte Carlo time grows roughly linearly with dimension, but early exercise is difficult in "original formulation" (Andersen–Piterbarg).

**Reasoned Inference (Transparent):**

- FD curse of dimensionality: number of grid points scales like $N_x^d$.
- MC scaling: runtime $\propto n\times N_t\times(\text{cost per step})$, typically linear in $d$ for factor simulations.

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
                        │  State dimension d? │
                        └──────────┬──────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
              ▼                    ▼                    ▼
         ┌────────┐          ┌──────────┐         ┌──────────┐
         │ d = 1  │          │ d = 2-3  │         │  d > 3   │
         └───┬────┘          └────┬─────┘         └────┬─────┘
             │                    │                    │
             ▼                    ▼                    ▼
   ┌───────────────────┐   ┌─────────────┐    ┌──────────────────┐
   │ Path dependent?   │   │ Use ADI/FD  │    │ Monte Carlo      │
   └────────┬──────────┘   │ or 2D Trees │    │ (only option)    │
            │              └─────────────┘    └──────────────────┘
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

| Feature | Trees | Finite Differences | Monte Carlo |
|---------|-------|-------------------|-------------|
| Dimension $d=1$ | ✓✓✓ | ✓✓✓ | ✓ |
| Dimension $d=2$ | ✓ | ✓✓ | ✓✓ |
| Dimension $d \ge 3$ | ✗ | ✗ | ✓✓✓ |
| European payoff | ✓✓ | ✓✓ | ✓✓ |
| American/Bermudan | ✓✓✓ | ✓✓✓ | ✓ (LSMC) |
| Path-dependent | ✗ | ✗ | ✓✓✓ |
| Greeks | Bumping | Direct from grid | Pathwise/LR |
| Implementation | Simple | Moderate | Simple |
| Convergence rate | $O(\Delta t)$ | $O(\Delta t^2)$ (C-N) | $O(1/\sqrt{n})$ |

### A5.3.3 Rule of Thumb

Hull provides a practical summary:

> "Monte Carlo simulation is usually used for derivatives where the payoff is dependent on the history of the underlying variable or where there are several underlying variables. Trees and finite difference methods are usually used for American options and other derivatives where the holder has decisions to make prior to maturity."

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

**Key Feature:** The tree incorporates mean reversion naturally through modified probabilities and non-uniform node spacing.

### A5.4.5 Convergence of Binomial Trees

Hull Ch. 13 Appendix proves that as $N \to \infty$, the CRR binomial tree price converges to the Black–Scholes price for European options.

**Convergence Rate:** Typically $O(\Delta t) = O(1/N)$ for standard constructions.

**Oscillation:** Even-numbered and odd-numbered $N$ can produce systematically different prices due to whether the final nodes straddle the strike. Practitioners often use:
- Richardson extrapolation
- Leisen–Reimer construction
- Large $N$ (e.g., $N \ge 100$)

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

**Alternative (PSOR):** The Projected Successive Over-Relaxation method solves the linear complementarity problem (LCP) directly.

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

Mixed partial derivatives (e.g., $\partial^2 V / \partial x \partial y$ from correlation) require special treatment. The standard approach treats these explicitly or uses modified ADI formulations such as the Craig–Sneyd or Hundsdorfer–Verwer schemes.

**Stability:**

ADI schemes inherit the unconditional stability of implicit methods when properly formulated, though cross-derivative terms can introduce restrictions. Andersen and Piterbarg provide detailed stability analysis in their treatment.

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

**Practical Considerations:**

- QMC does not provide a natural error estimate (no CLT-based confidence interval)
- **Randomized QMC** (scrambled sequences) restores error estimation capability
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

---

## A5.7 Method Comparison and Tradeoffs

### A5.7.1 Comprehensive Comparison Table

| Criterion | Trees | Finite Differences | Monte Carlo |
|-----------|-------|-------------------|-------------|
| **Dimensionality** | 1–2 factors | 1–3 factors | Any $d$ |
| **American options** | Natural | Natural (LCP) | LSMC (harder) |
| **Path dependence** | Limited | Limited | Natural |
| **Greeks** | Stable from grid | Stable from grid | Noisy (variance) |
| **Convergence** | $O(1/N)$ | $O(1/N)$ or $O(1/N^2)$ | $O(1/\sqrt{n})$ |
| **Implementation** | Simple | Moderate | Simple |
| **Parallelization** | Limited | Limited | Excellent |
| **Memory** | $O(N)$ | $O(M \cdot N)$ | $O(n)$ or $O(1)$ |

### A5.7.2 When Each Method Excels

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

### A5.7.3 Hybrid Approaches

- **MC with PDE for early exercise:** Use PDE to determine exercise boundary, MC for valuation
- **Quasi-MC with variance reduction:** Combine Sobol sequences with control variates
- **Adaptive mesh:** Figlewski–Gao adaptive mesh model grafts fine grids onto coarse grids

---

## A5.8 Worked Examples

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

**Setup:** Same as Example 7.1, with grid $M = 20$ (stock price intervals), $N = 10$ (time steps).

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

**Using the same setup as Example 7.4** ($S_0 = 100$, $K = 100$, $r = 5\%$, $\sigma = 20\%$, $q = 0$):

For each pair $(Z, -Z)$, since $r - q - \sigma^2/2 = 0.03$:
- Path 1: $S_T^{(+)} = 100\exp[0.03 + 0.20 Z]$
- Path 2: $S_T^{(-)} = 100\exp[0.03 - 0.20 Z]$

Average: $\hat{V}_{\text{pair}} = \frac{1}{2}e^{-rT}\left[\max(S_T^{(+)} - 100, 0) + \max(S_T^{(-)} - 100, 0)\right]$

**Result:** With 5,000 pairs, variance reduction of approximately 75% for at-the-money calls.

---

### Example A5.6: Control Variate for Call

**Using the same setup as Example 7.4.**

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

## A5.9 Practical Notes

### A5.9.1 Common Pitfalls

**Trees:**
- Using too few time steps ($N < 50$) for accurate pricing
- Ignoring odd/even oscillation in convergence
- Forgetting to adjust for dividends in CRR formulas

**Finite Differences:**
- Violating explicit method stability conditions
- Boundary conditions too close to the region of interest
- Using C-N for discontinuous payoffs without smoothing

**Monte Carlo:**
- Confusing tight confidence intervals with accuracy (bias!)
- Using too few paths for Greeks (noisy estimates)
- Not checking variance reduction effectiveness

### A5.9.2 Implementation Tips

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

### A5.9.3 Sanity Checks

1. **Put-call parity:** Does your European call/put satisfy parity?
2. **Boundary limits:** Does price → intrinsic as $S \to 0$ or $S \to \infty$?
3. **Convergence:** Does price stabilize as $N, M, n$ increase?
4. **American ≥ European:** Is your American price at least the European price?
5. **Greeks signs:** Is delta positive for calls? Is gamma positive?

---

## Summary

This appendix covered three fundamental numerical methods for derivative pricing:

1. **Trees (Binomial/Trinomial):** Discrete approximations of the underlying process; natural for American options in low dimensions; CRR is the standard construction; trinomial trees are equivalent to explicit FD.

2. **Finite Differences:** Discretize the pricing PDE; implicit methods are unconditionally stable; explicit methods connect to trees but have stability restrictions; Crank–Nicolson offers second-order accuracy.

3. **Monte Carlo:** Sample paths from risk-neutral dynamics; the only practical method for high-dimensional problems; variance reduction is essential; LSMC extends MC to American options.

**Key Principle (Lax Equivalence):** Consistency + Stability ⇒ Convergence

**Method Selection:** Dimension and exercise style are the primary drivers—use trees/FD for low dimensions with early exercise, MC for high dimensions or path dependence.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Stability** | Scheme remains bounded as discretization refines | Unstable schemes diverge |
| **Consistency** | Discrete operator approximates continuous | Ensures correct limiting PDE |
| **Convergence** | Numerical solution → true solution | The ultimate goal |
| **Lax Equivalence** | Consistency + Stability ⇒ Convergence | Fundamental theorem |
| **CFL Condition** | Stability constraint on $\Delta t / \Delta x^2$ | Explicit method restriction |
| **Recombining Tree** | $ud = 1$ so paths rejoin | Polynomial vs exponential nodes |
| **LSMC** | Regression-based MC for Americans | Key technique for high-dim |
| **Variance Reduction** | Techniques to narrow CI | Makes MC practical |

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
| 21 | How do you get gamma from a tree? | Use prices at three nodes around the center |
| 22 | What is importance sampling? | Change measure to oversample important regions; reweight by likelihood ratio |
| 23 | What basis functions are common in LSMC? | Powers $(1, S, S^2, \ldots)$, Laguerre, Chebyshev polynomials |
| 24 | What is the Thomas algorithm? | $O(M)$ solution of tridiagonal systems; used in implicit FD |
| 25 | Why transform to $Z = \ln S$ in FD? | Makes coefficients independent of $j$; improves stability |
| 26 | What is ADI? | Alternating Direction Implicit: splits 2D problem into 1D sweeps |
| 27 | How does mean reversion affect trees? | Requires modified probabilities or non-uniform spacing |

---

## Mini Problem Set

### Warm-Up (Conceptual)

**Problem 1:** Explain why Monte Carlo becomes the only practical method when $d > 3$.

**Problem 2:** A colleague says "I used explicit FD and got negative option prices." What went wrong?

**Problem 3:** Why does variance reduction matter more for out-of-the-money options than at-the-money?

### Computational (Trees)

**Problem 4:** Construct a 3-step CRR tree for a European put with $S_0 = 100$, $K = 95$, $T = 0.25$, $r = 5\%$, $\sigma = 30\%$. Compute the option price.

**Problem 5:** For the tree in Problem 4, convert to an American put. At which nodes would early exercise occur?

**Problem 6:** Build a 2-step trinomial tree for the same option. Compare the price to the binomial result.

### Computational (Finite Differences)

**Problem 7:** Set up the implicit FD difference equation for a European call. What are $a_j$, $b_j$, $c_j$?

**Problem 8:** For explicit FD with $\sigma = 40\%$, $\Delta t = 1/12$, $\Delta S = 5$, determine the maximum $j$ (stock price level) for which the method is stable.

**Problem 9:** Explain why transforming to $Z = \ln S$ makes explicit FD equivalent to a trinomial tree with constant probabilities.

### Computational (Monte Carlo)

**Problem 10:** Write pseudocode for pricing an Asian call option using Monte Carlo.

**Problem 11:** You estimate a call price with $n = 1000$ paths and get SE = 0.50. How many paths do you need to reduce SE to 0.05?

**Problem 12:** Derive the pathwise delta estimator for a European call under GBM.

### Advanced

**Problem 13:** Describe how to combine antithetic variates and control variates in a single MC simulation.

**Problem 14:** In LSMC, why do we only regress over "in-the-money" paths at each exercise date?

**Problem 15:** Explain the bias-variance tradeoff when using finite-difference bumping for Greeks in Monte Carlo.

**Problem 16:** A barrier option has payoff that depends on whether $\max_{t \le T} S_t > B$. Can you use standard FD? What modifications are needed?

### Solution Sketches

**Problem 1:** Grid methods scale as $O(N^d)$ in memory and computation; for $d > 3$, this becomes prohibitive. MC scales as $O(nd)$—linear in dimension.

**Problem 2:** Stability condition violated. Need $\Delta t < 1/(\sigma^2 j^2)$. Either decrease $\Delta t$, use implicit method, or transform to $\ln S$.

**Problem 4 Sketch:**
- $\Delta t = 0.25/3 = 0.0833$
- $u = e^{0.30\sqrt{0.0833}} = 1.0905$
- $d = 0.9170$
- $p = (e^{0.05 \times 0.0833} - 0.9170)/(1.0905 - 0.9170) = 0.525$
- Build tree, compute terminal payoffs, backward induct.

**Problem 11:** SE scales as $1/\sqrt{n}$. Need $0.50/\sqrt{n'} = 0.05$, so $n' = 1000 \times 100 = 100{,}000$ paths.

---

## Source Map

### (A) Verified Facts

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
| Stability definition | Duffy |
| Consistency definition | Duffy |
| Convergence definition | Andersen–Piterbarg |
| Lax equivalence theorem | Andersen–Piterbarg Vol 1 |
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
| American put free boundary constraint | Standard references |

### (B) Reasoned Inference

- FD curse of dimensionality ($N_x^d$ scaling) derived from grid structure
- MC linear scaling in dimension derived from simulation structure
- Stability condition $\Delta t < 1/(\sigma^2 j^2)$ derived from requiring positive probabilities in explicit FD
- LSMC low bias follows from suboptimality of approximate continuation values
- ADI unconditional stability inherited from implicit method structure (with caveats for cross-derivatives)

### (C) Flagged Uncertainties

- Cross-derivative treatment in ADI: The standard ADI schemes require modification for mixed partial derivatives arising from correlation. The exact formulation (Craig–Sneyd, Hundsdorfer–Verwer, etc.) depends on the specific problem. I'm not certain about the optimal choice without further source verification.
- Randomized QMC error estimation: The precise variance formulas for scrambled Sobol sequences depend on the scrambling method used. The general principle is well-established in Glasserman, but implementation details vary.

---

*Last Updated: January 2026*
