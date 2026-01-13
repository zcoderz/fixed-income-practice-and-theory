# Chapter 17: Curve Construction as an Inverse Problem (Bootstrapping + Interpolation)

---

## Fact Classification

### (A) Verified Facts (Source-Backed)
- Discount curve definition and properties (Interest Rate Modeling Vol 1)
- Bootstrap methodology as sequential node solving (Interest Rate Modeling Vol 1 Ch 4-6)
- Swap PV formula: $V_{\text{swap}} = 1 - P(t_n) - \sum_{j=1}^{n} c\tau P(j\tau)$ (Interest Rate Modeling)
- Simple forward rate identity: $1 + \tau L(0,T,T+\tau) = \frac{P(T)}{P(T+\tau)}$ (Interest Rate Modeling, Hull Ch 4)
- Piecewise linear yield interpolation and its forward-curve artifacts (Tuckman, Interest Rate Modeling)
- Piecewise-flat forward rates formula: $P(T) = P(T_i)\exp(-f(T_i)(T-T_i))$ (Interest Rate Modeling)
- Smoothing/best-fit as constrained optimization with regularity norms (Interest Rate Modeling)
- Post-crisis multi-curve framework preview (Interest Rate Modeling, Hull Ch 4)
- Cubic spline curve fitting and locality concerns (Tuckman, Interest Rate Modeling)
- OIS zero curve linear interpolation assumption (Hull Ch 4)

### (B) Reasoned Inference (Derived from A)
- Linear-in-log-DF interpolation produces equal simple forwards over equal sub-intervals at midpoint (algebraic consequence of geometric mean)
- Jacobian structure is near-triangular in sequential bootstrap (follows from sequential dependence)
- Unit checks on all formulas (dimensional analysis)

### (C) Speculation (Clearly Labeled; Minimal)
- Specific smoothing algorithm details for production desks (marked with "I'm not sure" where applicable)

---

## Conventions & Notation

### Notation Glossary

| Symbol | Meaning |
|--------|---------|
| $T \geq 0$ | Time measured in years |
| $P(0,T) \equiv P(T)$ | Discount factor: time-0 price of ZCB paying 1 at $T$ |
| $P(0) = 1$ | Normalization at valuation date |
| $y(T)$ | Continuously compounded spot yield: $P(T) = e^{-y(T)T}$ |
| $f(T)$ | Instantaneous forward rate: $f(T) = -\frac{d}{dT}\ln P(T)$ |
| $L(0,T,T+\tau)$ | Simple forward (money-market) rate over $[T, T+\tau]$ |
| $\{T_i\}$ | Curve nodes: maturities where we solve for $P(T_i)$ |
| $V_i$ | Model present value of calibration instrument $i$ |
| $c$ | Fixed swap rate (per year) on the fixed leg |
| $\tau_j$ | Accrual year fraction for coupon period $j$ |
| 1 bp | Basis point $= 10^{-4}$ |

### Key Identities

Forward-yield relationship:
$$f(T) = y(T) + T y'(T)$$

Simple forward rate definition:
$$1 + \tau L(0,T,T+\tau) = \frac{P(T)}{P(T+\tau)}$$

Special case $T=0$:
$$P(\tau) = \frac{1}{1 + \tau L(0,0,\tau)}$$

### Conventions Used in This Chapter

- **Valuation time**: all prices/discount factors are at $t=0$
- **Single-curve setup**: one curve $P(T)$ is used for discounting all cash flows (pre-crisis framing; multi-curve previewed only)
- **Calibration goal**: reproduce (exactly, in bootstrap) the observed quotes of liquid benchmark instruments
- **Toy day-count**: year fractions equal year-length intervals (e.g., 6 months $= 0.5$ years)
- **Toy money-market quoting**: deposit/bill-style quotes are simple rates with $1 + \tau L = \frac{1}{P(\tau)}$

---

## Core Concepts

### 1) Discount Curve $P(T)$

**Formal Definition:**
The discount curve is the function $T \mapsto P(T) = P(0,T)$, the price at time 0 of a $T$-maturity discount bond paying 1 at $T$. In the discount-curve setup in Interest Rate Modeling, $P(T)$ is taken as continuous and monotonically decreasing.

**Intuition:**
$P(T)$ is "how much today" one unit of currency received at $T$ is worth.

**Trading / Risk / Portfolio Practice:**
- A curve lets you PV any deterministic cash-flow stream by discounting cash flows
- In Tuckman's Treasury context, a discount function derived from traded bonds can be used for rich/cheap analysis and to fill missing prices when some maturities are illiquid
- Swap desk analog: discount functions from traded swaps are used to value illiquid swap maturities

---

### 2) Yield Curve $y(T)$ and Forward Curve $f(T)$

**Formal Definition:**

*Yield curve (continuous comp):*
$$y(T) \text{ defined by } P(T) = e^{-y(T)T}$$

*Instantaneous forward:*
$$f(T) = -\frac{d}{dT}\ln P(T), \quad \text{with } f(T) = y(T) + Ty'(T)$$

*Simple forward over $[T, T+\tau]$:*
$$1 + \tau L(0,T,T+\tau) = \frac{P(T)}{P(T+\tau)}$$

**Intuition:**
- $y(T)$ compresses the whole discount factor $P(T)$ into an annualized number at horizon $T$
- $f(T)$ describes the market-implied "instantaneous" rate around time $T$; it is the derivative object controlling how $P(T)$ changes with maturity

**Trading / Risk / Portfolio Practice:**
Many hedges and risk decompositions are stated in forward-rate space (e.g., bucket exposures / key rates). Tuckman explicitly computes exposures to changes in forward rates and shows how a 1 bp bump to one forward bucket changes spot rates/discount factors and the PV of a par swap.

---

### 3) Calibration Instruments and Cash-Flow Representation

**Formal Definition:**
A fixed-cash-flow instrument's PV is a linear combination of discount factors at its cash-flow dates:
$$V_i = \sum_{j=1}^{m_i} c_{i,j} P(t_j)$$
where $c_{i,j}$ are the instrument's cash flows (scaled by notional) at dates $t_j$.

**Intuition:**
A curve is useful because it turns pricing into "cash flows $\times$ discount factors."

**Trading / Risk / Portfolio Practice:**
Curve nodes are often chosen to align with liquid benchmark maturities (bills/deposits, futures/FRAs, swaps, on-the-run bonds), because these instruments provide the best information for those maturities.

---

### 4) Curve Construction as an Inverse Problem

**Formal Definition:**
We observe a finite set of market quotes (prices/par rates) on benchmark instruments, and we infer a continuous curve object $P(T)$ (or $y(T)$, $f(T)$) consistent with those quotes. Because "only a few short-dated discount bonds are directly quoted," and in general we must "uncover the whole discount curve" from "a finite set of benchmark securities," we require an interpolation/curve-building method.

**Intuition:**
You're reconstructing a function from sparse measurements—like reconstructing a smooth line from a handful of points.

**Trading / Risk / Portfolio Practice:**
Different interpolation/representation choices can produce different implied forward curves and therefore different derivative prices and hedges, even if benchmark instruments are repriced exactly.

---

### 5) Bootstrapping (Exact Fit) vs Smoothing (Best Fit)

**Formal Definition:**

*Bootstrapping / exact fit:* Choose a curve representation and solve sequentially for curve nodes so that each calibration instrument is matched exactly (up to rounding), i.e., $V_i(\text{curve}) = V_i^{\text{mkt}}$.

*Smoothing / best fit:* When quotes are noisy/inconsistent, choose a curve that fits "well" while controlling roughness. Interest Rate Modeling describes choosing an "optimal" yield curve by minimizing a regularity norm subject to a constraint on the RMS fit error.

**Intuition:**
Exact fit forces the curve through every quote; best fit trades small pricing errors for a more stable/realistic curve shape.

**Trading / Risk / Portfolio Practice:**
Bid/ask, stale prints, or inconsistent quotes motivate smoothing/cleaning. Tuckman emphasizes that curve fitting is "part art," and choices like which bonds to include and where to place spline knots must be adapted to market context.

---

### 6) Interpolation as a Modeling Choice

**Formal Definition:**
Given solved values at nodes $\{T_i\}$, define the curve for intermediate maturities by an interpolation rule (in $P$, $\log P$, $y$, $f$, spline coefficients, etc.). Interest Rate Modeling stresses that with finite instruments we must choose a curve interpolation/parameterization because the problem is otherwise underdetermined.

**Intuition:**
Interpolation is where you "choose the shape" of the curve between quotes.

**Trading / Risk / Portfolio Practice:**
Forward-rate behavior can be strongly affected. Tuckman warns that a "common but unsatisfactory" approach is linear yield interpolation.

---

### 7) Locality

**Formal Definition:**
Locality refers to how a small change in one market quote affects curve values across maturities. Some methods produce changes concentrated near the bumped maturity ("local"), while others create broad oscillations ("global").

**Intuition:**
A good curve methodology should not make the whole long end jump violently because a single short-end quote moved.

**Trading / Risk / Portfolio Practice:**
Risk systems often rely on the mapping "quote bumps $\to$ node changes" (a Jacobian), so the curve method's locality properties directly affect DV01/key-rate/bucket risk stability. Interest Rate Modeling notes that par-point approaches and certain spline constructions can be designed for good locality, while some globally smooth splines can exhibit "ringing" from a local bump.

---

## Math and Derivations

### 2.1 Pricing Equations as a Linear Inverse Problem in Discount Factors

**Assumption (this chapter):** Deterministic cash flows and single discount curve $P(T)$.

For instrument $i$ with cash flows $\{c_{i,j}\}$ at payment times $\{t_{i,j}\}$:

$$\boxed{V_i = \sum_{j=1}^{m_i} c_{i,j} P(t_{i,j})} \tag{2.1}$$

This is exactly the linear-combination representation used for discount-curve construction.

Collect all distinct cash-flow dates across benchmark instruments into $\{t_1, \ldots, t_M\}$. Define the vector:
$$\mathbf{P} = (P(t_1), \ldots, P(t_M))^\top$$

and instrument PVs $\mathbf{V} = (V_1, \ldots, V_N)^\top$. Then:
$$\mathbf{V} = \mathbf{C}\mathbf{P}$$

where $\mathbf{C}$ is the cash-flow matrix of coefficients.

**Unit check:**
- $P(t)$ is dimensionless (price of \$1 paid later in units of \$ today)
- Cash flows $c_{i,j}$ are in currency units (or "per unit notional")
- PV $V_i$ is in currency units
- Matrix multiplication $\mathbf{C}\mathbf{P}$ preserves units: $(\text{cashflow}) \times (\text{dimensionless}) \to (\text{cashflow})$

---

### 2.2 Why It Is an Inverse Problem (Ill-Posedness / Underdetermination)

- The unknown "object" $P(\cdot)$ is a function (infinite-dimensional)
- We have only a finite set of benchmark instrument quotes
- Even if we restrict to node values $\{P(T_i)\}_{i=1}^K$, the mapping from node parameters to instrument PVs depends on interpolation between nodes

**This is why curve construction requires a modeling choice.**

Interest Rate Modeling emphasizes that because we have fewer benchmark instruments than potential cash-flow dates, we must impose additional structure: constrain interpolation, use a root-search bootstrap with interpolation, or choose an "optimal" curve (regularized fit).

---

### 2.3 Bootstrapping as Sequential Inversion (Node-by-Node Solve)

A common approach is to choose a node grid $0 < T_1 < T_2 < \cdots < T_K$ aligned with benchmark maturities and solve iteratively.

**Bootstrapping logic (per Interest Rate Modeling, paraphrased):** When we have already solved the discount factors up to $T_{i-1}$, we solve for $P(T_i)$ by:
1. Guessing $P(T_i)$
2. Interpolating the curve between $T_{i-1}$ and $T_i$
3. Repricing the $i$-th benchmark instrument
4. Adjusting the guess until the instrument PV matches its market price

**Concretely, for instrument $i$ maturing at $T_i$:**
1. Assume $P(t)$ is known for all $t \leq T_{i-1}$
2. Guess a value for $P(T_i)$
3. Use the chosen interpolation rule to get $P(t)$ for $T_{i-1} < t \leq T_i$
4. Compute the model PV $V_i$ of instrument $i$
5. Adjust the guess for $P(T_i)$ until $V_i$ matches the market price
6. Proceed to the next maturity $T_{i+1}$

**Key point:** Even in "bootstrapping," you are not just solving for a single DF—your interpolation choice is part of the model because it determines all intermediate discount factors used to price the instrument.

---

### 2.4 Closed-Form Bootstrapping for Common Toy Cases

#### 2.4.1 Deposit / Bill-Style Instrument (Simple Money-Market Rate)

From the simple forward-rate definition:
$$1 + \tau L(0,T,T+\tau) = \frac{1}{P(0,T,T+\tau)}, \quad \text{where } P(0,T,T+\tau) = \frac{P(T+\tau)}{P(T)} \tag{2.2}$$

If $T=0$, then $P(0,0,\tau) = P(\tau)$ (since $P(0) = 1$), so:
$$\boxed{1 + \tau L(0,0,\tau) = \frac{1}{P(\tau)} \Rightarrow P(\tau) = \frac{1}{1 + \tau L(0,0,\tau)}} \tag{2.3}$$

**Unit check:** $L$ has units "per year," $\tau$ is years; $\tau L$ is dimensionless; $P(\tau)$ is dimensionless.

#### 2.4.2 Par Fixed-for-Floating Swap (Single-Curve Discounting)

In Interest Rate Modeling, the PV of a fixed-for-floating swap (in their setup) can be written as:
$$V_{\text{swap}} = 1 - P(t_n) - \sum_{j=1}^{n} c\tau P(j\tau) \tag{2.4}$$

for fixed rate $c$ and payment interval $\tau$ (specializing to $t=0$ and equally spaced payments).

A par swap satisfies $V_{\text{swap}} = 0$, so:
$$\boxed{1 = P(T_n) + c\sum_{j=1}^{n} \tau_j P(t_j)} \tag{2.5}$$

If only the last DF is unknown (typical in bootstrap when earlier nodes already known), this becomes a one-equation solve for $P(T_n)$.

**Example with annual fixed payments** $\tau_1 = \tau_2 = 1$ and maturity $T_2 = 2$:
$$0 = 1 - P(2) - c(P(1) + P(2)) \Rightarrow \boxed{P(2) = \frac{1 - cP(1)}{1 + c}} \tag{2.6}$$

---

### 2.5 Interpolation Variables and Their Consequences

A key message in Interest Rate Modeling and Tuckman is that different interpolation choices have different implications for the smoothness (and realism) of the implied forward curve.

#### (i) Interpolating the Yield Curve $y(T)$

If we represent the curve by yields $y(T)$, then $P(T) = e^{-y(T)T}$.

**Piecewise linear yields:**
$$y(T) = y_i + (y_{i+1} - y_i)\frac{T - T_i}{T_{i+1} - T_i}, \quad T \in [T_i, T_{i+1}] \tag{2.7}$$

as given in Interest Rate Modeling.

**Consequence:** The instantaneous forward rate $f(T) = y(T) + Ty'(T)$ can become discontinuous and "saw-tooth."

Tuckman similarly notes that linear yield interpolation creates kinks that induce sudden jumps in the implied forward curve.

#### (ii) Interpolating the Forward Curve $f(T)$ / Piecewise-Constant Forward

Interest Rate Modeling gives piecewise-flat forward rates:
$$\boxed{P(T) = P(T_i)\exp(-f(T_i)(T - T_i)), \quad T \in [T_i, T_{i+1}]} \tag{2.8}$$

which implies $\ln P(T)$ is linear in $T$ on each interval.

**Consequence:** Forward rates are constant within an interval but generally discontinuous at nodes; bootstrapped curves often have discontinuous forward curves, and this can distort derivative prices and hedges in models sensitive to the forward curve.

#### (iii) Interpolating Discount Factors Directly $P(T)$

Direct interpolation on $P(T)$ is conceptually simple, but Interest Rate Modeling notes it is often easier to devise interpolation on yields rather than on bond prices/discount function, and points to pitfalls in interpolators that work directly on the discount function $P(T)$.

**Practical implication:** "Linear in DF" is easy, but you must check the induced forward curve for realism.

---

### 2.6 Exact Fit vs Best Fit as Optimization/Regularization (Conceptual)

When quotes are noisy or inconsistent, Interest Rate Modeling presents a constrained optimization view: minimize a curve "roughness" norm $\|y\|$ subject to bounding RMS pricing error (scaled by tolerances $D_i$, which can be interpreted as bid/ask widths or quote uncertainties).

This is the conceptual bridge to "smoothing": you intentionally do not force exact repricing of every instrument, to avoid implausible wiggles and unstable sensitivities.

---

## Measurement & Risk

### 3.1 Inverse Problem Framing (Finite Quotes $\to$ Continuous Curve)

- **Observed data:** A finite set of quotes (prices, par swap rates, money-market rates)
- **Unknown curve object:** A function $P(T)$ (or equivalently $y(T)$, $f(T)$)
- **Why ill-posed:** Many curves can fit the same finite quotes unless you impose additional structure (node choice + interpolation, or regularization)

This is explicitly highlighted by the "finite set of benchmark securities $\to$ uncover the whole discount curve" problem.

### 3.2 Bootstrapping as Sequential Inversion

Bootstrapping transforms the inverse problem into a series of low-dimensional solves:
- At step $i$, you solve for a single new degree of freedom (e.g., $P(T_i)$ or $y(T_i)$) while earlier parts are frozen
- **But:** The method is only defined once you choose an interpolation scheme for intermediate maturities (needed to price the instrument)

### 3.3 Interpolation Choice and Locality (Local vs Global Behavior)

Two distinct notions of "locality" matter on a desk:

**Calibration locality (bootstrapping structure):**
In a strict sequential bootstrap, earlier nodes are typically unchanged by later instruments, but earlier nodes can affect later nodes because later PV equations depend on earlier discount factors (e.g., swap PV depends on earlier fixed-leg discounting).

**Interpolation locality (shape choice):**
The interpolation/smoothing rule determines whether a local bump in one quote causes:
- A local shift in nearby forwards/discount factors, or
- An oscillatory global ripple

Interest Rate Modeling discusses "par-point" and spline approaches with good locality and contrasts them with globally smooth $C^2$ spline approaches that can exhibit "ringing" in the forward curve after a local bump; tension can reduce the noise.

Tuckman's bucket-exposure example also embodies locality: bumping one forward bucket changes the implied spot curve/discount factors and the PV of a swap.

### 3.4 Jacobian / Sensitivity Map Preview (Conceptual)

Let $\mathbf{q} \in \mathbb{R}^N$ denote the vector of market quotes used to build the curve, and $\mathbf{x} \in \mathbb{R}^K$ the vector of curve parameters/nodes (e.g., $\mathbf{x} = (P(T_1), \ldots, P(T_K))$).

A first-order risk approximation is:
$$\boxed{\Delta\mathbf{x} \approx \mathbf{J}\Delta\mathbf{q}, \quad \mathbf{J} = \frac{\partial\mathbf{x}}{\partial\mathbf{q}}} \tag{3.1}$$

In a bootstrap, $\mathbf{J}$ often has a near-triangular structure: short-end quotes mostly affect short-end nodes, and effects propagate forward in maturity through the sequential dependence.

(We compute a tiny finite-difference Jacobian in Example I.)

### 3.5 Exact Fit vs Smoothing in Practice

- **Exact fit:** Reprices all calibration instruments essentially to zero error (a "reprice test")
- **Best fit / smoothing:** Allow small residuals (within bid/ask or tolerances), while minimizing roughness. This is consistent with constrained formulations described in Interest Rate Modeling

Tuckman explains why the fit objective matters: minimizing price errors can overweight long bonds relative to short bonds because long-bond prices are more sensitive to yield changes; minimizing yield errors can be preferable for producing a realistic rate function across maturities.

### 3.6 Brief Preview: Multi-Curve (Not Implemented Here)

Both Interest Rate Modeling and Hull describe the post-crisis shift away from a single-curve framework:

- Interest Rate Modeling notes the "pre-crisis era" single curve and the "post-crisis era" with multiple curves and separation of discounting vs forwarding
- Hull notes that after 2007, multiple yield curves are needed: the discount curve comes from OIS, while payoffs depend on another curve (e.g., 3M/6M)

We will not build a multi-curve system in this chapter; we only flag that the inverse-problem + interpolation framework generalizes naturally.

---

## Worked Examples

### Common Conventions for Examples A–I

- Notional $= 1$
- Time in years; accrual fractions equal to year intervals shown
- Deposits/bills quoted as simple money-market rates $L(0,0,T)$ so $P(T) = \frac{1}{1 + TL}$
- Swap fixed leg: annual payments, accrual $\tau = 1$, and par swap condition from $V_{\text{swap}} = 1 - P(T_n) - \sum c\tau P(t_j)$
- For "zero rate" outputs, we report the simple spot rate:
$$z_{\text{simple}}(T) \equiv \frac{1/P(T) - 1}{T}$$

---

### Example A: Bootstrap DFs from Deposit/Bill-Style Rates

**Inputs (toy deposit quotes):**
- $T = 0.25$: $L(0,0,0.25) = 5.00\% = 0.0500$
- $T = 0.50$: $L(0,0,0.50) = 5.50\% = 0.0550$
- $T = 1.00$: $L(0,0,1.00) = 6.00\% = 0.0600$

**Formula:**
$$P(T) = \frac{1}{1 + T \cdot L(0,0,T)} \tag{A.1}$$

**Compute discount factors:**

$$P(0.25) = \frac{1}{1 + 0.25 \cdot 0.05} = \frac{1}{1.0125} = 0.9876543210$$

$$P(0.50) = \frac{1}{1 + 0.50 \cdot 0.055} = \frac{1}{1.0275} = 0.9732360097$$

$$P(1.00) = \frac{1}{1 + 1.00 \cdot 0.06} = \frac{1}{1.06} = 0.9433962264$$

**Sanity checks:**
- DFs are positive and decreasing: $1 = P(0) > P(0.25) > P(0.5) > P(1)$ ✓
- Units: $TL$ dimensionless, so $P(T)$ dimensionless ✓

---

### Example B: Add One Swap and Bootstrap Longer DF

**New input:**
- 2-year par swap fixed rate $c_{2y} = 6.50\% = 0.0650$
- Annual fixed payments at $t=1$ and $t=2$ with $\tau = 1$

**Swap par condition:**
$$0 = 1 - P(2) - c(P(1) + P(2)) \tag{B.1}$$

**Solve for $P(2)$:**
$$1 = P(2) + cP(1) + cP(2) \Rightarrow P(2)(1+c) = 1 - cP(1) \Rightarrow P(2) = \frac{1 - cP(1)}{1 + c} \tag{B.2}$$

**Plug numbers:**
- $P(1) = 0.9433962264$
- Numerator: $1 - cP(1) = 1 - 0.065 \cdot 0.9433962264 = 1 - 0.0613207547 = 0.9386792453$
- Denominator: $1 + c = 1.065$

$$P(2) = \frac{0.9386792453}{1.065} = 0.8813889627$$

**Output curve nodes so far:**
| $T$ | $P(T)$ |
|-----|--------|
| 0.25 | 0.9876543 |
| 0.50 | 0.9732360 |
| 1.00 | 0.9433962 |
| 2.00 | 0.8813890 |

---

### Example C: Repricing Check (Mandatory Reprice Test)

#### C1. Deposits Reprice

For a deposit with maturity $T$ and simple rate $L$, the par condition is:
$$1 = P(T)(1 + TL) \tag{C.1}$$

Using $P(T) = \frac{1}{1 + TL}$, the RHS equals 1 exactly (up to rounding).

For instance at $T = 0.5$:
$$P(0.5)(1 + 0.5 \cdot 0.055) = 0.9732360097 \cdot 1.0275 = 1.0000000000 \checkmark$$

#### C2. Swap Reprice

Compute swap PV using:
$$V_{\text{swap}} = 1 - P(2) - c(P(1) + P(2)) \tag{C.2}$$

Plugging our bootstrapped values:
- $P(1) = 0.9433962264$, $P(2) = 0.8813889627$, $c = 0.065$

$$V_{\text{swap}} = 1 - 0.8813889627 - 0.065(0.9433962264 + 0.8813889627) \approx 0 \text{ (within rounding)} \checkmark$$

**Conclusion:** The curve reprices all input instruments (as required for bootstrap).

---

### Example D: Interpolation Necessity — Two Schemes Compared

We need a discount factor at $T = 1.5$ years even though we only solved nodes at 1y and 2y.

**Setup:**
- $T_1 = 1$, $P_1 = P(1) = 0.9433962264$
- $T_2 = 2$, $P_2 = P(2) = 0.8813889627$
- $T = 1.5$, so $w = \frac{T - T_1}{T_2 - T_1} = 0.5$

#### Scheme 1: Linear in Discount Factor $P(T)$

$$P^{\text{linDF}}(T) = (1-w)P_1 + wP_2 \tag{D.1}$$

$$P^{\text{linDF}}(1.5) = 0.5(0.9433962264) + 0.5(0.8813889627) = 0.9123925946$$

#### Scheme 2: Linear in $\log P(T)$ (Geometric Interpolation)

"Linear in $\log P$" means:
$$\log P(T) = (1-w)\log P_1 + w\log P_2 \Rightarrow P^{\log}(T) = P_1^{1-w} P_2^w \tag{D.2}$$

At $w = 0.5$:
$$P^{\log}(1.5) = \sqrt{P_1 P_2} = \sqrt{0.9433962264 \cdot 0.8813889627} = 0.9118656817$$

#### Compare Implied Simple Spot "Zero" Rates at $T = 1.5$

$$z_{\text{simple}}(T) = \frac{1/P(T) - 1}{T} \tag{D.3}$$

**For linear DF:**
$$z_{\text{simple}}^{\text{linDF}}(1.5) = \frac{1/0.9123925946 - 1}{1.5} = 0.0640126 = 6.4013\%$$

**For log DF:**
$$z_{\text{simple}}^{\log}(1.5) = \frac{1/0.9118656817 - 1}{1.5} = 0.0644352 = 6.4435\%$$

**Observation:** Two plausible interpolation choices give different intermediate discount factors and rates, even with the same endpoints.

---

### Example E: Forward-Rate "Wiggles" Under Two Schemes

We compute simple forwards over sub-periods $[1, 1.5]$ and $[1.5, 2]$.

From the simple forward-rate identity:
$$L(0, T_1, T_2) = \frac{P(T_1)/P(T_2) - 1}{T_2 - T_1} \tag{E.1}$$

#### Under Linear-DF Interpolation

- $P(1) = 0.9433962264$
- $P(1.5) = P^{\text{linDF}}(1.5) = 0.9123925946$
- $P(2) = 0.8813889627$

**Forward over $[1, 1.5]$ (length 0.5):**
$$L^{\text{linDF}}(0, 1, 1.5) = \frac{0.9433962264/0.9123925946 - 1}{0.5} = 0.0679611 = 6.7961\%$$

**Forward over $[1.5, 2]$:**
$$L^{\text{linDF}}(0, 1.5, 2) = \frac{0.9123925946/0.8813889627 - 1}{0.5} = 0.0703520 = 7.0352\%$$

**Forward jump:** $7.0352\% - 6.7961\% \approx 0.2391\%$ ($\approx 24$ bps) between adjacent half-year periods.

#### Under Log-DF Interpolation

- $P(1.5) = P^{\log}(1.5) = 0.9118656817$

**Forward over $[1, 1.5]$:**
$$L^{\log}(0, 1, 1.5) = \frac{0.9433962264/0.9118656817 - 1}{0.5} = 0.0691561 = 6.9156\%$$

**Forward over $[1.5, 2]$:**
$$L^{\log}(0, 1.5, 2) = \frac{0.9118656817/0.8813889627 - 1}{0.5} = 0.0691561 = 6.9156\%$$

**Observation:** Log-DF interpolation produces a flatter forward profile here, while linear-DF interpolation produces a local forward-rate "kink/jump."

**Why this matters (desk intuition):** Forward curve behavior matters for pricing and hedging instruments sensitive to forwards (caps/floors, swaptions, etc.). Both Interest Rate Modeling and Tuckman emphasize that common interpolation schemes can induce discontinuities or artifacts in the forward curve.

---

### Example F: Locality Experiment — Quote Bump

We bump the 1Y deposit quote by $+1$ bp:
- Original $L(0,0,1) = 6.00\%$
- Bumped $L'(0,0,1) = 6.01\% = 0.0601$
- All other quotes unchanged (3M, 6M deposits and the 2Y swap rate)

#### Step 1: Recompute $P(1)$

$$P'(1) = \frac{1}{1 + 1 \cdot 0.0601} = \frac{1}{1.0601} = 0.9433072352$$

#### Step 2: Recompute $P(2)$ from the 2Y Swap Par Condition

Using $P(2) = \frac{1 - cP(1)}{1 + c}$ with $c = 0.065$:

$$P'(2) = \frac{1 - 0.065 \cdot P'(1)}{1.065} = \frac{1 - 0.065 \cdot 0.9433072352}{1.065} = 0.8813943941$$

#### Before/After Table at Curve Nodes

| Node $T$ | Quote used | $P(T)$ (before) | $P'(T)$ (after) | Change $\Delta P$ |
|----------|------------|-----------------|-----------------|-------------------|
| 0.25 | 3M deposit | 0.9876543210 | 0.9876543210 | 0 |
| 0.50 | 6M deposit | 0.9732360097 | 0.9732360097 | 0 |
| 1.00 | 1Y deposit | 0.9433962264 | 0.9433072352 | $-8.8991 \times 10^{-5}$ |
| 2.00 | 2Y swap | 0.8813889627 | 0.8813943941 | $+5.4314 \times 10^{-6}$ |

#### Discussion (Locality vs Propagation)

- The bump affects its own node $P(1)$ strongly (expected)
- The bump also affects later nodes (here $P(2)$) because later instrument PV equations depend on earlier discount factors (the fixed leg discounts include $P(1)$). This is "propagation through calibration equations"
- Under different representations (e.g., global splines), a local quote bump can cause broader oscillations ("ringing") in forwards/spot rates; Interest Rate Modeling shows this phenomenon and discusses tension to reduce noise

---

### Example G: Locality and Instrument Choice

We compare two curves built from the same short-end deposits but different 2Y instruments.

#### Curve 1 (Baseline): 2Y Par Swap as in Examples A–C

From Example B:
$$P_{\text{swap}}(2) = 0.8813889627$$

#### Curve 2 (Alternative): Replace 2Y Swap with a "2Y Deposit Proxy"

Assume instead we observe a (toy) 2-year deposit/bill quote:
$$L(0,0,2) = 6.20\% = 0.0620 \text{ (simple, with } T = 2 \text{)}$$

Then:
$$P_{\text{dep}}(2) = \frac{1}{1 + 2 \cdot 0.062} = \frac{1}{1.124} = 0.8896797153$$

#### Compare the Two 2Y Discount Factors

- Swap-implied: $0.8813890$
- Deposit-proxy-implied: $0.8896797$
- Difference: $0.0082908$ (almost 0.83 "DF points"), which is large

#### Interpret in Terms of Implied Swap Rate

If we used the deposit-proxy curve to compute the par swap rate (annual pay at 1 and 2), then from:
$$0 = 1 - P(2) - c(P(1) + P(2)) \Rightarrow c = \frac{1 - P(2)}{P(1) + P(2)} \tag{G.1}$$

we obtain:
$$c_{\text{implied}} = \frac{1 - 0.8896797153}{0.9433962264 + 0.8896797153} = 0.0601834 = 6.0183\%$$

But the market swap quote we used in Curve 1 was $6.50\%$. **So Curve 2 would misprice that swap.**

#### Desk Interpretation (Model Risk)

Curve construction depends on instrument set selection. Tuckman stresses that curve-fitting choices depend on context and are "part art," including which instruments/bonds to include.

Different instrument sets can produce different discount factors and therefore different valuations for off-curve trades (asset swaps, illiquid maturities, relative value).

---

### Example H: Smoothing vs Exact Fit — Toy Inconsistent Quotes

We create a deliberately inconsistent quote set to show the need for quote cleaning / constraints / best-fit.

- Keep deposits as in Example A so $P(1) = 0.9433962264$
- Now suppose a (nonsensical) 2Y par swap quote is: $c_{2y} = 120\% = 1.20$

From the par-swap bootstrap formula:
$$P(2) = \frac{1 - cP(1)}{1 + c} \tag{H.1}$$

**Compute numerator:**
$$1 - cP(1) = 1 - 1.20 \cdot 0.9433962264 = 1 - 1.1320754717 = -0.1320754717$$

**Denominator:** $1 + c = 2.20$

Thus:
$$P(2) = \frac{-0.1320754717}{2.20} = -0.0609433962$$

#### Why This Is an Arbitrage Red Flag

A discount factor is the PV of a strictly positive payoff of 1 at maturity; a negative PV is not economically meaningful in this deterministic-cashflow setting and violates basic pricing logic.

#### What Practitioners Do (Conceptual, Source-Backed)

**Quote cleaning / selection:** Tuckman notes that curve fitting requires experimentation and careful choice of instruments; poor fits can "wiggle too much to be believable," reflecting over/under-fitting and poor instrument selection.

**Smoothing / best fit rather than exact fit:** Interest Rate Modeling describes choosing an "optimal" yield curve by minimizing a regularity norm subject to limiting the RMS fit error—consistent with using bid/ask to allow small misfits rather than enforcing an impossible exact fit.

#### A Minimal Constrained Fix (No Detailed Algorithm)

- **Constraint:** Enforce $P(T) > 0$ for all maturities, and (under the book's monotone assumption) enforce non-increasing $P$
- **Action:** Reject the inconsistent quote, widen tolerances, or fit a curve that stays in the admissible set and accepts a pricing residual on the bad instrument

**I'm not sure** which exact smoothing algorithm your desk uses; we would need: (i) the exact instrument set (OIS vs LIBOR swaps vs Treasuries), (ii) the interpolation variable, (iii) the error metric (price vs yield vs weighted), and (iv) constraints (positivity, monotonicity, convexity) to specify it fully.

---

### Example I: Sensitivity/Jacobian Preview — Finite-Difference Jacobian

We build a tiny system with:
- **Quotes** $\mathbf{q} = (L_{0.5}, L_{1.0}, c_{2y})$
- **Nodes** $\mathbf{x} = (P(0.5), P(1), P(2))$

**Baseline quotes:**
- $L_{0.5} = 5.50\% = 0.0550$
- $L_{1.0} = 6.00\% = 0.0600$
- $c_{2y} = 6.50\% = 0.0650$

**Baseline nodes (from Examples A–B):**
- $P(0.5) = 0.9732360097$
- $P(1) = 0.9433962264$
- $P(2) = 0.8813889627$

#### Step 1: Bump Each Quote by +1 bp and Rebuild

A "finite-difference Jacobian" uses:
$$J_{k,j} \approx \frac{x_k(\mathbf{q} + \Delta q_j \mathbf{e}_j) - x_k(\mathbf{q})}{\Delta q_j}, \quad \Delta q_j = 0.0001 \tag{I.1}$$

**Bump 1:** $L_{0.5} \to 0.0551$
$$P'(0.5) = \frac{1}{1 + 0.5 \cdot 0.0551} = 0.9731886526$$
Other nodes unchanged.

Changes:
- $\Delta P(0.5) = -4.7357 \times 10^{-5}$
- $\Delta P(1) = 0$
- $\Delta P(2) = 0$

**Bump 2:** $L_{1.0} \to 0.0601$ (this is Example F)
- $P'(1) = 0.9433072352$
- $P'(2) = 0.8813943941$

Changes:
- $\Delta P(0.5) = 0$
- $\Delta P(1) = -8.8991 \times 10^{-5}$
- $\Delta P(2) = +5.4314 \times 10^{-6}$

**Bump 3:** $c_{2y} \to 0.0651$
$$P'(2) = \frac{1 - 0.0651 \cdot P(1)}{1.0651} = 0.8812176375$$

Changes:
- $\Delta P(0.5) = 0$
- $\Delta P(1) = 0$
- $\Delta P(2) = -1.7133 \times 10^{-4}$

#### Step 2: Present the (1 bp) Bump-to-Node Map

It is often useful to present $\mathbf{J}$ in "per 1 bp" form: entries are $\Delta P$ from a +1 bp bump.

| Node change from +1bp bump | $L_{0.5}$ | $L_{1.0}$ | $c_{2y}$ |
|----------------------------|-----------|-----------|----------|
| $\Delta P(0.5)$ | $-4.7357 \times 10^{-5}$ | $0$ | $0$ |
| $\Delta P(1.0)$ | $0$ | $-8.8991 \times 10^{-5}$ | $0$ |
| $\Delta P(2.0)$ | $0$ | $+5.4314 \times 10^{-6}$ | $-1.7133 \times 10^{-4}$ |

#### Interpretation

- This small Jacobian is mostly lower-triangular (short-end quotes affect short nodes; effects propagate forward mainly through the calibration equations)
- In more realistic curves (many swap cash-flow dates, interpolation, global splines), the Jacobian can become less local, consistent with the "ringing" phenomenon discussed in Interest Rate Modeling

---

## Practical Notes

### 5.1 Common Interpolation Choices (Defined Precisely)

Below, assume nodes $(T_i, P_i)$ and $(T_{i+1}, P_{i+1})$, and define the weight:
$$w = \frac{T - T_i}{T_{i+1} - T_i} \in [0, 1]$$

#### (a) Linear in Discount Factor $P(T)$

$$P(T) = (1-w)P_i + wP_{i+1}$$

**Status vs sources:** The books discuss interpolation choices and caution that interpolating directly on the discount function can have pitfalls; the linear-in-$P$ formula itself is a straightforward definition, but the cautions about direct $P$-interpolation are explicitly noted.

#### (b) Linear in Log Discount Factor $\log P(T)$

$$\log P(T) = (1-w)\log P_i + w\log P_{i+1} \Rightarrow P(T) = P_i^{1-w} P_{i+1}^w$$

**Connection to piecewise-constant forwards:** If you assume piecewise-flat forwards, Interest Rate Modeling shows:
$$P(T) = P(T_i)\exp(-f(T_i)(T - T_i))$$
implying $\log P$ linear in $T$ on each interval.

#### (c) Linear in Zero Rate $y(T)$ (Continuous Comp)

Let $y(T)$ satisfy $P(T) = e^{-y(T)T}$. Then "linear in $y$" means:
$$y(T) = (1-w)y_i + wy_{i+1}, \quad P(T) = \exp(-y(T)T)$$

Hull explicitly notes that a "zero curve can be assumed to be linear between maturities" in the context of constructing an OIS zero curve.

#### (d) Linear in Forward Rate / Piecewise-Constant Forward

**Piecewise-constant instantaneous forward:** Set $f(T) = f_i$ for $T \in [T_i, T_{i+1})$.

Then $P$ follows (as above) $P(T) = P(T_i)\exp(-f_i(T - T_i))$.

**Implication:** Forward is constant in each bucket but may jump at nodes; derivative pricing can be sensitive to such discontinuities.

#### (e) Spline Methods (Only Where Source-Supported)

**Piecewise cubic spline (Tuckman):** Represent the curve (e.g., spot rates) as piecewise cubic polynomials joined at knot points.

Tuckman emphasizes that knot placement/number and bond selection materially affect realism; overfitting can create implausible wiggles in spot/forward curves.

Interest Rate Modeling discusses cubic Hermite splines and par-point approaches, and highlights locality differences between methods (e.g., natural $C^2$ vs tensioned vs par-point constructions).

---

### 5.2 No-Arbitrage / Sanity Constraints and What to Check

**Minimum checks (desk-robust):**

1. **Discount factors positive:** $P(T) > 0$ (Basic PV logic; also consistent with discount-bond interpretation)

2. **Shape check:** The book's discount curve definition assumes $P(T)$ is monotonically decreasing

3. **Careful note (reasoned from definitions):** Since $f(T) = -\frac{d}{dT}\ln P(T)$, if forward rates are negative over an interval then $P(T)$ can increase there. This does not automatically imply arbitrage, but it may violate the monotone assumption used in some curve setups

4. **Reprice errors near zero** for calibration instruments (bootstrap exact fit) or within tolerance (best fit)

5. **Forward-rate reasonableness:** Avoid huge oscillations/jumps introduced by interpolation; both sources warn about artifacts and discontinuities in forward curves under some interpolation choices

---

### 5.3 Implementation Pitfalls

- **Date generation and accrual:** Stubs, business-day adjustments, and accrual factors can change PVs materially
- **Day-count mismatches:** Deposits, swaps, futures can use different day-counts; mixing inconsistently shifts implied DFs
- **Compounding basis mismatches:** Ensure conversions between quoted rates and discount factors match each instrument's market convention
- **Curve-building order dependence:** Bootstrapping is sequential by maturity (assume known up to $T_{i-1}$, solve at $T_i$)
- **Interpolation-induced pathologies:** Direct interpolation on discount factors can have pitfalls; check induced forwards

---

### 5.4 Verification Tests (Practical Checklist)

- [ ] Reprice test for every calibration instrument (Example C)
- [ ] Small bump stability test: bump one quote by 1 bp, rebuild, and ensure curve changes are sensible (Example F)
- [ ] Locality sanity: a quote should primarily affect nearby maturities unless the instrument's PV structurally depends on distant cash flows
- [ ] Regression test: identical inputs $\to$ identical curve nodes and repricing errors

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **Curve construction is an inverse problem:** Infer a continuous curve $P(T)$ from a finite set of instrument quotes

2. **Discounting a fixed cash-flow instrument is linear in discount factors:** $V_i = \sum_j c_{i,j} P(t_j)$

3. **In matrix form, $\mathbf{V} = \mathbf{C}\mathbf{P}$:** Without further structure the problem is underdetermined

4. **Bootstrapping solves curve nodes sequentially:** Guess $P(T_i)$, interpolate, reprice, and iterate until match

5. **The bootstrap is only fully defined once you choose an interpolation variable and scheme**

6. **Different interpolation schemes can generate very different forward curves** (kinks, saw-teeth, discontinuities)

7. **Locality matters:** Some curve methods propagate a local quote change globally ("ringing"); tension/par-point approaches can improve locality

8. **Exact-fit bootstraps are fragile to inconsistent/noisy quotes;** best-fit approaches use smoothing/regularization with tolerances

9. **Instrument selection is part of model risk:** Different instrument sets can imply different curves and valuations

10. **The same inverse-problem logic extends to multi-curve discounting/forwarding setups,** but this chapter stays single-curve

---

### Cheat Sheet

#### Bootstrap (Node-by-Node)

1. Choose node maturities $T_1 < \cdots < T_K$ (typically aligned with benchmark maturities)
2. For each $i$:
   - Assume $P(t)$ known for $t \leq T_{i-1}$
   - Guess $P(T_i)$
   - Interpolate $P(t)$ on $(T_{i-1}, T_i]$
   - Compute PV of instrument $i$
   - Root-search until model PV matches quote

#### Common Interpolation Choices

| Method | Formula |
|--------|---------|
| Linear in DF | $P(T) = (1-w)P_i + wP_{i+1}$ |
| Linear in log DF | $P(T) = P_i^{1-w} P_{i+1}^w$ |
| Linear in zero rate $y(T)$ | $y(T) = (1-w)y_i + wy_{i+1}$, $P(T) = e^{-y(T)T}$ |
| Piecewise-constant forward | $P(T) = P(T_i)e^{-f_i(T-T_i)}$ |
| Cubic spline families | See Tuckman, Interest Rate Modeling; check for overfitting/oscillations |

#### Sanity Checklist

- [ ] $P(T) > 0$
- [ ] Monotone decreasing under the book's setup; if negative rates are possible, interpret carefully
- [ ] Reprice errors: zero (bootstrap) or within tolerance (fit)
- [ ] Forwards not wildly oscillatory; beware artifacts from interpolation

---

### Flashcards (30)

1. **Q:** What is the discount factor $P(0,T)$?
   **A:** The time-0 price of a zero-coupon bond paying 1 at $T$.

2. **Q:** How is the continuously compounded yield $y(T)$ defined from $P(T)$?
   **A:** $P(T) = e^{-y(T)T}$.

3. **Q:** Define the instantaneous forward rate $f(T)$.
   **A:** $f(T) = -\frac{d}{dT}\ln P(T)$.

4. **Q:** What is the relationship between $f(T)$ and $y(T)$ in continuous compounding?
   **A:** $f(T) = y(T) + Ty'(T)$.

5. **Q:** What is the simple forward-rate identity connecting $L(0,T,T+\tau)$ and discount factors?
   **A:** $1 + \tau L(0,T,T+\tau) = \frac{P(T)}{P(T+\tau)}$.

6. **Q:** In the special case $T=0$, how do you get $P(\tau)$ from a simple deposit rate?
   **A:** $P(\tau) = \frac{1}{1 + \tau L(0,0,\tau)}$.

7. **Q:** Why is curve construction an inverse problem?
   **A:** You infer a continuous curve from finitely many instrument quotes.

8. **Q:** What makes the inverse problem underdetermined?
   **A:** A finite instrument set cannot uniquely determine a function without extra structure (interpolation/regularization).

9. **Q:** What is "bootstrapping" in curve construction?
   **A:** Sequentially solving for curve nodes so each benchmark instrument is repriced (exactly in bootstrap).

10. **Q:** What is the key hidden modeling choice inside a bootstrap?
    **A:** The interpolation rule used between nodes.

11. **Q:** Write the generic linear cash-flow PV representation.
    **A:** $V_i = \sum_j c_{i,j} P(t_j)$.

12. **Q:** What does the cash-flow matrix formulation look like?
    **A:** $\mathbf{V} = \mathbf{C}\mathbf{P}$.

13. **Q:** Give the swap PV expression used for curve construction in Interest Rate Modeling.
    **A:** $V_{\text{swap}} = 1 - P(t_n) - \sum_{j=1}^{n} c\tau P(j\tau)$.

14. **Q:** What defines a par swap?
    **A:** Fixed rate $c$ such that $V_{\text{swap}} = 0$.

15. **Q:** What is linear interpolation in discount factor?
    **A:** $P(T) = (1-w)P_i + wP_{i+1}$.

16. **Q:** What is linear interpolation in log discount factor?
    **A:** $P(T) = P_i^{1-w} P_{i+1}^w$.

17. **Q:** Which interpolation corresponds to piecewise-constant instantaneous forwards?
    **A:** Log-discount-factor linear (since $\log P$ is linear when $P(T) = P(T_i)e^{-f_i(T-T_i)}$).

18. **Q:** Why can linear yield interpolation be problematic?
    **A:** It creates kinks and can cause jumps/artifacts in implied forward rates.

19. **Q:** What forward-curve pathology can piecewise linear yields create?
    **A:** Discontinuous "saw-tooth" instantaneous forwards.

20. **Q:** What is meant by "locality" in curve construction?
    **A:** How concentrated the curve response is to a local quote change.

21. **Q:** What is "ringing" in curve construction?
    **A:** Global oscillations in the curve from a local perturbation (e.g., in globally smooth splines).

22. **Q:** What is a practical test for curve implementation correctness?
    **A:** Reprice every calibration instrument to its market quote (reprice test).

23. **Q:** What is a "bump stability" test?
    **A:** Bump one quote by 1 bp and ensure curve nodes/forwards move sensibly (no wild oscillations).

24. **Q:** Why might minimizing yield errors be preferable to minimizing price errors in fitting?
    **A:** Equal price error weights can underweight yield errors of short bonds relative to long bonds.

25. **Q:** What does "best fit" mean in curve fitting?
    **A:** Allow small mispricing to achieve smoother/regular curve shape.

26. **Q:** Give an example of a smoothing formulation described in Interest Rate Modeling.
    **A:** Minimize a regularity norm of the yield curve subject to an RMS error constraint.

27. **Q:** What is one key source of curve model risk besides interpolation?
    **A:** Instrument set choice (which quotes are used as "truth").

28. **Q:** What is the post-2007 multi-curve idea in one sentence?
    **A:** Discounting uses an OIS curve while forward projections depend on another curve (e.g., 3M/6M).

29. **Q:** In Hull's OIS curve construction, what simple interpolation assumption is sometimes used?
    **A:** The zero curve is assumed linear between maturities.

30. **Q:** What is the conceptual role of the Jacobian in curve risk?
    **A:** It maps small quote changes into node/parameter changes for risk attribution.

---

## Mini Problem Set

### Problems

1. Using the simple-rate relation, compute $P(0.25)$ from a 3M deposit quote of 4.80% (simple).

2. Given $P(1) = 0.95$, compute the simple spot rate $z_{\text{simple}}(1)$.

3. Derive the par swap rate formula for a 2Y annual-pay fixed leg in terms of $P(1)$ and $P(2)$.

4. Suppose you have $P(1) = 0.94$ and a 2Y par swap rate $c = 7\%$. Solve for $P(2)$.

5. Between nodes $(1, P_1)$ and $(2, P_2)$, write the formula for $P(1.25)$ under (i) linear DF and (ii) linear log DF.

6. Using $P(1) = 0.94$ and $P(2) = 0.88$, compute the simple forward rate over $[1, 2]$.

7. Show that under log-DF interpolation, the simple forward rates over two equal sub-intervals $[1, 1.5]$ and $[1.5, 2]$ are equal (midpoint case).

8. Explain why bootstrapping requires an interpolation rule even if each new instrument adds only one new node.

9. Construct a toy set of quotes that would force $P(2) < 0$ when bootstrapping from a 1Y deposit and a 2Y swap. What does this indicate?

10. In a curve built by bootstrapping yields at nodes, what does it mean for the mapping from quotes to yields to be "nonlinear"?

11. Describe the difference between local and global interpolation methods in terms of how a quote bump affects the forward curve.

12. Why might minimizing price errors in a term-structure fit overweight long maturities?

13. What is a "reprice test" and why is it necessary but not sufficient for a good curve?

14. Consider two curves that both exactly reprice benchmark instruments but differ in intermediate forwards. Give two desk consequences.

15. Define a finite-difference Jacobian for curve nodes with respect to quotes and describe how it is used in risk.

16. Briefly explain how the inverse-problem framing extends from single-curve to multi-curve.

---

### Solution Sketches (Problems 1–8)

**1.** $P(0.25) = \frac{1}{1 + 0.25 \times 0.048} = \frac{1}{1.012} = 0.98814$.

**2.** $z_{\text{simple}}(1) = \frac{1/P(1) - 1}{1} = \frac{1/0.95 - 1}{1} = \frac{0.0526}{1} = 5.26\%$.

**3.** Par swap condition: $0 = 1 - P(2) - c(P(1) + P(2))$. Solving: $c = \frac{1 - P(2)}{P(1) + P(2)}$.

**4.** $P(2) = \frac{1 - cP(1)}{1 + c} = \frac{1 - 0.07 \times 0.94}{1.07} = \frac{0.9342}{1.07} = 0.8730$.

**5.** With $w = 0.25$:
   - (i) Linear DF: $P(1.25) = 0.75 P_1 + 0.25 P_2$
   - (ii) Linear log DF: $P(1.25) = P_1^{0.75} P_2^{0.25}$

**6.** $L(0, 1, 2) = \frac{P(1)/P(2) - 1}{1} = \frac{0.94/0.88 - 1}{1} = 0.06818 = 6.82\%$.

**7.** Under log-DF, $P(1.5) = \sqrt{P(1)P(2)}$. The forward over $[1, 1.5]$ is $\frac{P(1)/P(1.5) - 1}{0.5} = \frac{\sqrt{P(1)/P(2)} - 1}{0.5}$. The forward over $[1.5, 2]$ is $\frac{P(1.5)/P(2) - 1}{0.5} = \frac{\sqrt{P(1)/P(2)} - 1}{0.5}$. These are identical, proving the result.

**8.** Even though each instrument adds one unknown ($P(T_i)$), the instrument's PV depends on discount factors at intermediate dates (e.g., coupon dates before $T_i$). Without an interpolation rule, these intermediate DFs are undefined, making the pricing equation incomplete.

---

## Source Map

### (A) Verified Facts — Specific Sources

| Fact | Source |
|------|--------|
| Discount curve definition $P(T)$ monotone decreasing | Interest Rate Modeling Vol 1 |
| Bootstrap methodology (sequential solve with interpolation) | Interest Rate Modeling Vol 1 Ch 4-6 |
| Swap PV formula $V_{\text{swap}} = 1 - P(t_n) - \sum c\tau P(j\tau)$ | Interest Rate Modeling |
| Simple forward rate identity | Interest Rate Modeling, Hull Ch 4 |
| Piecewise linear yield interpolation formula | Interest Rate Modeling |
| Forward-rate artifacts from linear yield interpolation | Tuckman, Interest Rate Modeling |
| Piecewise-flat forward formula | Interest Rate Modeling |
| Smoothing as constrained optimization | Interest Rate Modeling |
| Post-2007 multi-curve framework | Interest Rate Modeling, Hull Ch 4 |
| Cubic spline curve fitting and locality | Tuckman, Interest Rate Modeling |
| OIS curve linear interpolation assumption | Hull Ch 4 |
| Curve fitting as "part art" | Tuckman |
| "Ringing" in global splines and tension methods | Interest Rate Modeling |

### (B) Reasoned Inference — Derivation Logic

| Inference | Derivation |
|-----------|------------|
| Log-DF interpolation gives equal forwards at midpoint | Geometric mean property: $P(1.5) = \sqrt{P(1)P(2)}$ implies symmetric forward structure |
| Jacobian is near-triangular in sequential bootstrap | Sequential dependence: later nodes depend on earlier DFs through fixed-leg discounting |
| Unit checks on all formulas | Dimensional analysis applied to each expression |

### (C) Speculation — Flagged Uncertainties

| Topic | Uncertainty |
|-------|-------------|
| Specific smoothing algorithm for production desks | I'm not sure which exact algorithm (spline class, error metric, constraints) applies without knowing the specific desk/instrument set |
| Exact fails-charge and penalty mechanics | Not covered in detail in sources used for this chapter |

---

*Last Updated: January 2026*
