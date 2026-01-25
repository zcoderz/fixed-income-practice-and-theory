# Appendix A5: Numerical Methods Map — Trees, PDE / Finite Differences, and Monte Carlo

---

## Conventions & Notation

| Convention | Description |
|------------|-------------|
| **Measure / pricing** | Risk-neutral pricing with discounting at the (possibly time-varying) short rate |
| **Pricing identity** | $V(t)=\mathbb{E}\Big[\exp\big(-\int_t^T r(u)\,du\big)\,V(T)\mid\mathcal{F}_t\Big]$ |
| **State** | $x\in\mathbb{R}^d$ denotes the **Markov** state (if it exists); $d$ is the **state dimension** |
| **Discretization** | $\Delta t$ for time step; $\Delta x$ (or $\Delta S$) for spatial step; $n$ for Monte Carlo replications |
| **Error** | **Bias** (systematic) vs **variance** (statistical); **discretization** error (grid/time) vs **simulation** error (sampling) |

### Black–Scholes Baseline

For a dividend yield $\delta$, a call value can be written as:

$$C(0)=e^{-\delta T}S_0\Phi(d)-e^{-rT}K\Phi(d-\sigma\sqrt{T})$$

with

$$d=\frac{\ln(S_0/K)+(r-\delta+\tfrac12\sigma^2)T}{\sigma\sqrt{T}}$$

(Setting $\delta=0$ gives the non-dividend case.)

---

## 0. Setup

### Conventions Used in This Appendix

- We treat numerical methods as approximations to a target **present value** (PV) or **value function** $V(t,x)$.
- We separate **modeling error** (wrong dynamics, wrong calibration) from **numerical error** (discretization/sampling).
- We use "explicit / implicit / Crank–Nicolson" in the standard finite-difference sense; note that Crank–Nicolson can develop oscillations for discontinuous terminal conditions unless remedied (e.g., Rannacher stepping).

### Notation Glossary

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

## 1. Core Concepts (Definitions First)

### 1.1 Pricing Target: $V(t,x)$ or $PV=\mathbb{E}[\text{discounted payoff}]$

**Formal Definition:**

- **Value function view:** $V(t,x)$ is the price conditional on current time and state.
- **Expectation view:** $V(t)=\mathbb{E}\!\left[D(t,T)\,V(T)\mid \mathcal{F}_t\right]$.

**Intuition:**

All three numerical methods (trees, PDE/FD, MC) are different approximations to the same object: the discounted expectation (or equivalently the PDE solution under conditions where Feynman–Kac applies).

**In Practice:**

- **Risk engines:** compute PV and Greeks across a book; need predictable runtimes and stable sensitivities.

---

### 1.2 State Dimension $d$ (Number of Risk Factors)

**Formal Definition:**

$d$ is the dimension of the Markov state vector $x$ needed so that future evolution depends on $(t,x)$ only.

**Intuition:**

Low $d$ enables grids/trees; high $d$ pushes you toward Monte Carlo.

**In Practice:**

Rates: 1F short-rate models (low $d$) can use PDE/trees; market models with many forward rates often drive $d$ high.

---

### 1.3 Path Dependence vs Markov (What Breaks PDE Methods)

**Formal Definition:**

A payoff is **path dependent** if it depends on the history $\{S(u):u\le T\}$ beyond the current state $x$.

**Intuition:**

PDE/FD and standard trees need a finite-dimensional Markov state; path dependence either:
- forces **state augmentation** (add running average, running extrema, etc.), raising $d$; or
- pushes you to Monte Carlo, which naturally handles path functionals.

**In Practice:**

Asians, lookbacks, many callable features with path triggers → MC is often the first tool.

---

### 1.4 American Early Exercise vs European (Free Boundary / LCP)

**Formal Definition:**

- **European:** exercise only at maturity.
- **American:** exercise at any time; value is the supremum over stopping times (not derived here).
- In a PDE framing for an American put, the price must satisfy a constraint $P(S,t)\ge \max(K-S,0)$ and features an **unknown exercise boundary** $B(t)$.

**Intuition:**

Early exercise turns pricing into a **free-boundary** or inequality-constrained problem.

**In Practice:**

Equity American options: trees and PDE methods are standard tools; MC needs specialized approaches (LSMC/regression).

---

### 1.5 Error Types: Bias vs Variance; Discretization vs Statistical

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

### 1.6 Stability, Consistency, Convergence (As in the Sources)

**Stability (Duffy):**

A scheme is **stable** if its output remains bounded as $\Delta t\to 0$, $\Delta x\to 0$.

**Consistency (Duffy):**

A scheme is **consistent** if the local truncation error $\tau(\Delta t,\Delta x)\to 0$ as $\Delta t,\Delta x\to 0$.

**Convergence (Andersen–Piterbarg):**

Convergence means $\max_j|\hat{v}(\tau_n,x_j)-v(\tau_n,x_j)|\to 0$ for fixed $\tau_n$ as the grid refines.

**Key Linkage (Lax Equivalence, Andersen–Piterbarg):**

For well-posed linear problems, **consistency + stability ⇒ convergence**.

---

### 1.7 Complexity Scaling (Runtime vs Grid/Paths/Dimension)

**Formal Objects:**

- FD grid: $N_x$ points per dimension; $N_t$ time steps.
- MC: $n$ paths and $N_t$ time steps per path (time-discretized simulation).

**Source-Backed Anchors:**

- A standard tridiagonal (1D) $\theta$-scheme with $m$ spatial points and $n$ time steps has complexity $O(mn)$.
- Finite differences are "suitable for 1, 2 and possibly 3 factors" and beyond that Monte Carlo is typical.
- Monte Carlo time grows roughly linearly with dimension, but early exercise is difficult in "original formulation."

**Reasoned Inference (Transparent):**

- FD curse of dimensionality: number of grid points scales like $N_x^d$.
- MC scaling: runtime $\propto n\times N_t\times(\text{cost per step})$, typically linear in $d$ for factor simulations.

---

## 2. Decision Framework: "Which Method When?"

This is the practitioner centerpiece: **features → method**.

### 2.1 ASCII Decision Tree

*(Content to be continued)*

---

## Source Map

### (A) Verified Facts

| Fact | Source |
|------|--------|
| Risk-neutral pricing identity | Andersen–Piterbarg Vol 1 |
| Black–Scholes call formula with dividend yield | Standard derivation |
| Crank–Nicolson oscillation issues | Duffy |
| Stability definition | Duffy |
| Consistency definition | Duffy |
| Convergence definition | Andersen–Piterbarg |
| Lax equivalence theorem | Andersen–Piterbarg |
| Tridiagonal $\theta$-scheme complexity $O(mn)$ | Andersen–Piterbarg |
| FD suitable for 1-3 factors | Andersen–Piterbarg |
| MC difficulty with early exercise | Andersen–Piterbarg |
| American put free boundary constraint | Standard references |

### (B) Reasoned Inference

- FD curse of dimensionality ($N_x^d$ scaling) derived from grid structure
- MC linear scaling in dimension derived from simulation structure

### (C) Speculation

- None in this appendix. All content is either source-verified or logically derived.

---

*Last Updated: January 2026*
