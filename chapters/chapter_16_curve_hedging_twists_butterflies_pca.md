# Chapter 16: Curve Hedging Beyond DV01 — Twists, Butterflies, and PCA

---

## Introduction

DV01 is a *one-direction* summary of rate risk: it tells you how PV changes under a small **parallel** shift of the curve (under a chosen bump definition). Real-world curves also twist and bend. As a result, a DV01-neutral book can still have large P&L under **non-parallel** moves (steepeners/flatteners, butterflies, and other local reshapes).

This chapter builds a more “vector-shaped” view of rate risk. We start with a practical taxonomy of curve moves (level/slope/curvature). We then show two ways to compress a high-dimensional curve into a few hedgeable directions:

- **Key-rate risk (KR01):** define a handful of “key” tenors, define a bump shape for each, and compute PV sensitivities to each bump.
- **Statistical factors (PCA):** extract a few orthogonal patterns from historical yield changes (often resembling level/slope/curvature).

Finally, we cover **regression hedging** as the minimum-variance answer when your hedge instrument is not a perfect maturity match, and we close with a checklist of what typically breaks curve hedges.

Prerequisites: [Chapter 14 — Key-Rate DV01 and Bucket Exposures](chapters/chapter_14_key_rate_dv01_bucket_exposures.md); [Chapter 15 — DV01 Hedging](chapters/chapter_15_dv01_hedging.md); (optional refresh) [Chapter 12 — Duration](chapters/chapter_12_duration.md); [Chapter 13 — Convexity](chapters/chapter_13_convexity.md).

Follow-on: [Chapter 17 — Curve Construction](chapters/chapter_17_curve_construction_bootstrapping_interpolation.md); [Chapter 18 — OIS Discounting](chapters/chapter_18_ois_discounting_curve.md); [Chapter 19 — Projection Curves / Multi-Curve](chapters/chapter_19_projection_curves_libor_sofr_multi_curve.md).

## Learning Objectives
- Translate “DV01-neutral” into an explicit curve-shock assumption and name what is *not* hedged.
- Represent rate risk as a vector (KR01 or factor exposures) and compute scenario P&L with correct units and sign.
- Size multi-instrument hedges by solving a linear system and understand degrees of freedom (what you can/can’t neutralize).
- Construct and interpret a butterfly (level+slope neutral) and understand wing-weight conventions.
- Use regression/PCA hedges responsibly: interpret $\beta$, $R^2$, and residual risk; recognize when factor hedges can drift.

---

## 16.1 The Taxonomy of Curve Risk

A yield curve is a function $y(T)$ (or, equivalently for pricing, the discount curve $P(0,T)$) indexed by maturity $T$. A parallel shift is only one special direction in this high-dimensional space. For hedging and communication, desks often describe curve reshaping with three stylized moves: **level**, **slope**, and **curvature**.

### 16.1.1 Level (Parallel Shifts)

A **level** move shifts all tenors by (approximately) the same number of basis points. This is the direction captured by a “parallel DV01” / duration-style risk measure.

### 16.1.2 Slope (Twists)

A **slope** move tilts the curve: short-end rates move up while long-end rates move down (or the reverse). Traders often call these **steepeners** and **flatteners**. A DV01-neutral barbell (e.g., long long-end, short short-end) can have near-zero level risk but large slope risk.

### 16.1.3 Curvature (Butterflies)

A **curvature** move changes the “hump”: the **belly** (intermediate maturities) moves relative to the **wings** (short and long ends). A **butterfly** position is designed to concentrate exposure here while being closer to neutral on level (and often slope).

> **Analogy: The Car Suspension**
>
> Think of the Yield Curve as a car chassis driving over a road.
> *   **Level (PC1)**: The road elevation changes. The whole car goes up or down.
> *   **Slope (PC2)**: The car accelerates or brakes. The nose lifts or dives (tilting).
> *   **Curvature (PC3)**: The car hits a pothole. The chassis bends or flexes in the middle.
>
> You can't fix a bent chassis (Curvature) by adjusting the ride height (Level). They are different mechanical problems.

**Check (toy shock vectors):** on tenors $(2y,5y,10y)$, three common stylized shocks are
$$
\delta^{\text{level}}=(+1,+1,+1),\quad
\delta^{\text{twist}}=(+1,0,-1),\quad
\delta^{\text{fly}}=(-1,+2,-1),
$$
all measured in bp. The last two sum to zero (no net “level”), and they are linearly independent. In later sections we turn these pictures into hedgeable objects (KR01 vectors or PCA factors) and show how to compute P&L.

---

## 16.2 Empirical Evidence: Principal Component Analysis

PCA is a data-driven way to find a small number of “standard” curve moves that explain most of the variance in historical yield changes. It is often used as an alternative to (or complement to) key-rate risk: instead of hedging a large list of tenors, you hedge a few **factors**.

### 16.2.1 PCA Objects: Loadings, Scores, Variance Explained

Pick a set of tenors $(T_1,\dots,T_n)$ and define the daily (or weekly) yield-change vector
$$
\Delta \mathbf{y}_t := (\Delta y_t(T_1),\dots,\Delta y_t(T_n))^\top.
$$
PCA computes the sample covariance matrix of $\Delta \mathbf{y}_t$, then finds eigenvectors $\mathbf{u}_j$ (“loadings”) and eigenvalues $\lambda_j$ (“factor variances”) such that:
$$
\Sigma\,\mathbf{u}_j = \lambda_j\,\mathbf{u}_j,\qquad \mathbf{u}_j^\top\mathbf{u}_j=1.
$$

The **factor score** (the realized “move” in factor $j$ on day $t$) is the projection:
$$
z_{j,t} := \mathbf{u}_j^\top \Delta \mathbf{y}_t.
$$
By construction (over the estimation sample), these scores are uncorrelated at lag 0. The fraction of variance explained by factor $j$ is:
$$
\text{VarShare}_j = \frac{\lambda_j}{\operatorname{tr}(\Sigma)}.
$$

### 16.2.2 Why the First PCs Look Like Level / Slope / Curvature

In many yield-curve datasets, the first few eigenvectors tend to resemble the stylized shocks from Section 16.1:
- **PC1 (level):** loadings have the same sign across maturities (a near-parallel shift).
- **PC2 (slope):** short end and long end have opposite signs (a twist).
- **PC3 (curvature):** belly moves against the wings (a butterfly-like shape).

This is an empirical statement about the chosen dataset: the tenors you include, the sampling frequency, and the lookback window all matter. PCA is “low-dimensional” *because you made it so*—you are choosing a small number of directions that best approximate the realized covariance structure.

### 16.2.3 Using PCA for Hedging (Factor Deltas)

Suppose your risk system reports sensitivities to small **up** moves in each tenor as a vector
$$
\mathbf{d} := \left(\frac{\partial PV}{\partial y(T_1)},\dots,\frac{\partial PV}{\partial y(T_n)}\right)^\top,
$$
with units “currency per bp” (after choosing a bump definition). A one-factor shock $z\,\mathbf{u}_j$ (in bp) produces the linearized P&L:
$$
\Delta PV \approx -\mathbf{d}^\top (z\,\mathbf{u}_j) = -(\mathbf{d}^\top \mathbf{u}_j)\,z.
$$
So the **PCA-factor DV01** for factor $j$ is simply $\mathbf{d}^\top\mathbf{u}_j$ (currency per bp of the factor score).

> **Desk Reality:** PCA hedges are instrument-efficient (few factors) but definition-sensitive. Two systems can both say “PC2” and still disagree because they used different tenors, windows, or curve definitions. If you hedge to PCA factors, document the factor construction and validate the hedge with explicit scenario shocks (level/twist/fly), not just “PC-neutral” labels.

**Check (factor decomposition $\rightarrow$ additive P\&L):** If you write the realized curve shock (in bp) as a sum of orthonormal factor directions,
$$\boldsymbol{\delta}=\sum_{j=1}^n \mathbf{u}_j\,z_j,\qquad z_j=\mathbf{u}_j^\top\boldsymbol{\delta},$$
then the linearized P\&L decomposes cleanly:
$$\Delta PV\approx -\mathbf{d}^\top\boldsymbol{\delta}=-\sum_{j=1}^n (\mathbf{d}^\top\mathbf{u}_j)\,z_j.$$
This gives two practical sanity checks:
- The **factor contributions add up** to the total P\&L (up to numerical error).
- Hedging the first few factor DV01s makes those contributions small, but **residual P\&L** comes from the remaining factors (and from any mismatch between “historical PCA shocks” and “today’s curve bumps”).

**Toy 3-tenor example (level/twist/fly as an orthonormal basis):** Take tenors $(2y,5y,10y)$ and orthonormal directions
$$\mathbf{u}_1=\frac{(1,1,1)}{\sqrt{3}},\quad \mathbf{u}_2=\frac{(1,0,-1)}{\sqrt{2}},\quad \mathbf{u}_3=\frac{(-1,2,-1)}{\sqrt{6}}.$$
Let $\mathbf{k}=(200,1500,800)$ $\$/\text{bp}$ be a KR01 vector (rates down convention) and consider an **up** shock $\boldsymbol{\delta}=(+3,0,-2)$ bp. The factor scores are $z_1=\tfrac{1}{\sqrt{3}}$, $z_2=\tfrac{5}{\sqrt{2}}$, $z_3=-\tfrac{1}{\sqrt{6}}$. The factor exposures are $\mathbf{k}^\top\mathbf{u}_1=\tfrac{2500}{\sqrt{3}}$, $\mathbf{k}^\top\mathbf{u}_2=-\tfrac{600}{\sqrt{2}}$, $\mathbf{k}^\top\mathbf{u}_3=\tfrac{2000}{\sqrt{6}}$. Summing factor P\&L gives
$$\Delta PV\approx -\sum_{j=1}^3 (\mathbf{k}^\top\mathbf{u}_j)\,z_j \approx +\$1{,}000,$$
which matches the direct dot product $-\mathbf{k}^\top\boldsymbol{\delta}=-(200\cdot 3+1500\cdot 0+800\cdot(-2))=+\$1{,}000$.

---

## 16.3 Key-Rate Risk (KR01): The Vector View of Risk

Key-rate risk (often called “key-rate DV01”) turns the yield curve into a *vector* risk report: instead of one DV01 number, you get a list of bucket sensitivities.

### 16.3.1 From Curve → PV → Risk (Definitions and Conventions)

**Anchor (PV):** for a fixed-cashflow instrument with cashflows $CF_i$ at dates $T_i$, a basic pricing representation is
$$
PV = \sum_i CF_i\,P(0,T_i),
$$
where $P(0,T)$ is the discount factor to maturity $T$. A curve shock changes discount factors (and sometimes cashflows, e.g., floating legs), which changes $PV$.

**Anchor (bump size and sign):**
- $1\text{ bp} = 10^{-4}$ in rate decimal.
- **DV01 (book convention):** $DV01 := PV(\text{rates down }1\text{ bp})-PV(\text{base})$, for a clearly stated bump object. For a long, option-free bond, $DV01>0$.
- **Key-rate DV01 (book convention):** define key tenors $(T_k)$ and a “key $k$ down 1bp” shock; then
  $$
  \mathrm{KR01}_k := PV(\text{key }k\text{ down }1\text{ bp})-PV(\text{base}).
  $$

**Expand (what “key $k$ down 1bp” means):** it is not universal. You must specify (i) the **bump object** (par yields? zero rates? instrument quotes?), and (ii) the **rebuild rule** (how the full curve is re-interpolated/rebootstrapped from bumped inputs). A common classroom construction is a piecewise-linear (“triangular”) yield shift between neighboring keys; other desks use spline-based or instrument-quote bump-and-rebuild designs.

> **Pitfall — “What is being bumped?”:** A “KR01” number is always “**shock shape + rebuild rule**,” not an intrinsic property of the bond.
> **Why it matters:** Two risk systems can report different KR01 vectors for the same position, leading to hedge ratios with the wrong magnitude (or even the wrong sign).
> **Quick check:** Confirm (i) bump object, (ii) bump size (1bp), (iii) interpolation/rebuild rule, and (iv) whether $\sum_k \mathrm{KR01}_k$ is close to the reported parallel DV01 under your chosen key-rate shift family.

**Check (checksum idea):** for some key-rate shift families, the shifts are constructed so that “adding all key shifts” approximates a parallel shift. In that case, $\sum_k \mathrm{KR01}_k$ should be close to a parallel DV01 computed under the same bump convention. Use this as a consistency check, not as an identity.

**Expand (why “triangular” key shifts can sum to parallel):** A common classroom construction defines the key-rate shocks as piecewise-linear “hat functions” (1 at one key tenor and 0 at its neighbors). These hat functions form a **partition of unity** over the tenor interval: at any maturity between the first and last key, the weights sum to 1. Under this specific construction, a 1bp parallel shift is literally “the sum of all key shifts,” which is why $\sum_k \mathrm{KR01}_k$ can track a parallel DV01.

**Check (local weight sum):** At a key $T_k$, the $k$-th hat is 1 and all others are 0. Halfway between keys $T_k$ and $T_{k+1}$, the two neighboring hats are 0.5 each. In both cases the weights sum to 1, which is exactly the “parallel as a sum of keys” idea.

### 16.3.2 Interpreting a KR01 Vector

Suppose your risk report provides key tenors $(2y,5y,10y,30y)$ and the KR01 vector (in $\$\!/\text{bp}$):

| Tenor $T_k$ | 2y | 5y | 10y | 30y | Total |
|---|---:|---:|---:|---:|---:|
| $\mathrm{KR01}_k$ | +200 | +1,500 | +800 | +400 | +2,900 |

This vector contains information that a single “total DV01” hides:
- **Concentration:** most exposure sits in the 5y bucket.
- **Scenario intuition:** if only the 5y point sells off by $+5$bp while others are unchanged, the first-order impact is about $-1{,}500\times 5=-\$7{,}500$.

> **Comparison: Sniper Rifle vs. Shotgun**
>
> * **Key rates:** a sniper rifle — local, granular exposures.
> * **PCA factors:** a shotgun — broad patterns that often track macro “modes.”
>
> Use key rates when you care about *where* on the curve the risk sits; use PCA when you care about hedging a few dominant *patterns*.

---

## 16.4 Hedging Mechanics: The Linear System

Once risk is a vector, hedging is linear algebra: you are choosing hedge positions so that the portfolio has (approximately) zero exposure to the curve moves you care about.

### 16.4.1 The Scenario P&L Formula

Let $\mathbf{k}=(\mathrm{KR01}_1,\dots,\mathrm{KR01}_m)^\top$ be the KR01 vector (units: currency per bp), and let $\boldsymbol{\delta}=(\delta_1,\dots,\delta_m)^\top$ be the realized **upward** shocks (in bp) at those keys over your horizon. The first-order P&L approximation is:

$$
\boxed{\Delta PV \approx -\mathbf{k}^\top \boldsymbol{\delta} = -\sum_{k=1}^m \mathrm{KR01}_k\,\delta_k.}
$$

**Check:**
- **Units:** $(\$\!/\text{bp})\times(\text{bp})=\$$.
- **Sign:** for a long rates position, $\mathrm{KR01}_k>0$. If rates sell off ($\delta_k>0$), the contribution $-\mathrm{KR01}_k\delta_k$ is negative (a loss).

### 16.4.2 Hedging as a System of Equations

Let $\mathbf{h}^{(j)}\in\mathbb{R}^m$ be the KR01 vector of hedge instrument $j$ per “one hedge unit” (e.g., per \$1mm face, per contract, per \$100 notional — state the unit). Stack these as columns to form the hedge matrix:
$$
\mathbf{H} := [\mathbf{h}^{(1)}\;\mathbf{h}^{(2)}\;\cdots\;\mathbf{h}^{(n)}].
$$
If $\mathbf{n}\in\mathbb{R}^n$ is the vector of hedge notionals/weights (signed; positive = long), the post-hedge exposure is:
$$
\mathbf{k}_{\text{post}} = \mathbf{k}_{\text{pre}} + \mathbf{H}\mathbf{n}.
$$
The “ideal” key-rate hedge solves $\mathbf{H}\mathbf{n}=-\mathbf{k}$, but this is feasible only when you have enough instruments and the hedge vectors span the risk directions you want to neutralize.

**Degrees of freedom (practical):**
- With $n$ hedge instruments you can neutralize at most $n$ *independent* directions (e.g., level+slope+curvature is three directions).
- If $m\gg n$, you pick what you care about (a few keys, or PCA factors), and accept residual risk elsewhere.

**Expand (exact hedge vs best-fit hedge):** When $m>n$, you typically cannot solve $\mathbf{H}\mathbf{n}=-\mathbf{k}$ exactly. Two common approaches are:
- **Constrain a low-dimensional subspace** you care about (e.g., match exposure to level/twist/fly or to a few key tenors), and ignore the rest.
- **Solve a weighted least-squares hedge** that minimizes residual exposure across keys:
  $$\min_{\mathbf{n}}\;\bigl\| \mathbf{W}^{1/2}(\mathbf{k}+\mathbf{H}\mathbf{n})\bigr\|^2,$$
  where $\mathbf{W}$ encodes which buckets/factors matter. When $\mathbf{H}^\top\mathbf{W}\mathbf{H}$ is invertible, the solution is
  $$\boxed{\mathbf{n}^* = -(\mathbf{H}^\top\mathbf{W}\mathbf{H})^{-1}\mathbf{H}^\top\mathbf{W}\mathbf{k}.}$$

> **Pitfall — Ill-conditioned hedge matrices:** If two hedge instruments have nearly identical KR01 profiles, $\mathbf{H}^\top\mathbf{W}\mathbf{H}$ can be close to singular. The hedge notionals become very large and unstable (small input changes $\rightarrow$ big hedge changes).
> **Quick check:** If net exposure is small but gross notionals are huge (near-cancellation), stress the hedge with small perturbations in KR01 mapping/interpolation to see if it is robust.

---

## 16.5 The Butterfly Trade: Isolating Curvature

Butterflies are three-leg curve trades designed to **concentrate curvature exposure** while reducing exposure to level (and often slope). The key idea is that with three legs you can usually impose **two** neutrality constraints and keep one degree of freedom for a view.

### 16.5.1 Butterfly Constraints (Level + Slope Neutrality)

Work on three keys $(2y,5y,10y)$ for clarity. Let the KR01 vector of the three-leg portfolio be $\mathbf{k}=(k_{2},k_{5},k_{10})$.

One simple definition of “level” and “slope” uses the toy shock vectors from Section 16.1:
- Level shock: $\delta^{\text{level}}=(1,1,1)$
- Twist shock: $\delta^{\text{twist}}=(1,0,-1)$

Then:
- **Level-neutral** means $\mathbf{k}^\top\delta^{\text{level}}=k_{2}+k_{5}+k_{10}=0$.
- **Twist-neutral** means $\mathbf{k}^\top\delta^{\text{twist}}=k_{2}-k_{10}=0$.

Solving these gives $k_{2}=k_{10}=-\tfrac{1}{2}k_{5}$: the two wings carry equal and opposite risk to the belly. This is the algebraic form of a “50/50 DV01-weighted” butterfly.

### 16.5.2 Worked Example: Build and Check a 2s–5s–10s Butterfly

**Example Title:** A DV01- and twist-neutral butterfly from a risk report

**Context**
- You want a position that is (approximately) neutral to small parallel moves and simple twists, but has exposure to curvature (“belly vs wings”).

**Timeline (Make Dates Concrete)**
- Trade date: 2026-03-02
- Settlement/effective date (assumption for this example): 2026-03-04
- Risk horizon: 1 business day (2026-03-02 close to 2026-03-03 close)

**Inputs**
- Instruments: par swaps at 2y, 5y, 10y (you could do the same with on-the-run bonds or futures; the mechanics below are the same once you have DV01s/KR01s).
- Risk (hypothetical, from your system): DV01 per \$1mm notional
  - 2y: \$190/bp per \$1mm
  - 5y: \$450/bp per \$1mm
  - 10y: \$830/bp per \$1mm
- Convention: $DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$. P&L approximation uses $\Delta PV\approx-\sum_k \mathrm{KR01}_k\delta_k$ with $\delta_k$ the **up** shock in bp.

**Outputs (What You Produce)**
- Notionals for a 50/50 DV01-weighted butterfly:
  - Receive fixed \$100mm 5y (belly)
  - Pay fixed \$118.4mm 2y (short wing)
  - Pay fixed \$27.1mm 10y (long wing)
- A scenario table showing the hedge is (approximately) level- and twist-neutral.

**Step-by-step**
1. Choose the belly notional: $N_5=\$100\text{mm}$. Its DV01 is $+100\times 450=+\$45{,}000/\text{bp}$.
2. For a 50/50 DV01 split, set each wing DV01 to $-\$22{,}500/\text{bp}$.
3. Convert DV01 targets into notionals:
   - 2y notional: $N_2 = 22{,}500/190 \approx \$118.4\text{mm}$ (short)
   - 10y notional: $N_{10} = 22{,}500/830 \approx \$27.1\text{mm}$ (short)

**Risk Table**
| Leg | Notional (\$mm) | DV01 per \$1mm (\$/bp) | Position DV01 (\$/bp) |
|---|---:|---:|---:|
| 2y | -118.4 | 190 | -22,500 |
| 5y | +100.0 | 450 | +45,000 |
| 10y | -27.1 | 830 | -22,500 |
| **Net** |  |  | **0** |

**Scenario P&L (1st order)**
- Level $\delta=(+10,+10,+10)$: $\Delta PV\approx -( -22{,}500+45{,}000-22{,}500)\times 10\approx 0$.
- Twist $\delta=(+10,0,-10)$: $\Delta PV\approx -(-22{,}500\times 10 + (-22{,}500)\times(-10))\approx 0$.
- Fly $\delta=(-5,+10,-5)$: $\Delta PV\approx -(112{,}500+450{,}000+112{,}500)= -\$675{,}000$. (Belly sells off while wings rally.)

**P&L / Risk Interpretation**
- This is a **curvature** position: it profits when the belly rallies relative to the wings (the “hump” increases), and loses when the belly cheapens relative to the wings.
- What can break the neutrality: a different definition of “twist” than $(+1,0,-1)$; non-local KR01 shift shapes; convexity and optionality; carry/rolldown and funding over longer holds; transaction costs and liquidity constraints.

**Sanity Checks**
- Units check: DV01 is $\$\!/\text{bp}$; multiplying by bp gives dollars.
- Sign check: rates up $\Rightarrow$ PV down for the long belly; short wings gain when rates rise on the wings and lose when wings rally.
- Limiting case: if all tenors move together, the net DV01 is near zero so P&L should be small.

**Debug Checklist (When Your Result Looks Wrong)**
- Are you using **up** shocks or **down** shocks? (This book’s DV01/KR01 is “rates down 1bp.”)
- Are DV01s/KR01s computed under the same bump definition as your scenarios?
- Did you scale DV01 per \$1mm into total DV01 correctly?

### 16.5.3 Butterfly Spreads (Yield Quote View)

Desks often monitor butterflies in **yield spread** units. A common definition of an “average” butterfly spread is:
$$
\text{AVG Fly} := \frac{y_{\text{wing1}}+y_{\text{wing2}}}{2}-y_{\text{belly}}.
$$
With DV01-style risk weights chosen so each wing carries half of the belly risk, small-horizon P&L can be written approximately as a constant (a risk-weight scale) times the change in this butterfly spread. This is useful because it turns a three-leg trade into one time series that is easy to plot and reason about.

**Expand (link to the KR01 dot product):** For the 50/50 DV01-weighted butterfly from Section 16.5.2, the exposures satisfy $k_{2}=k_{10}=-\tfrac{1}{2}k_{5}$. Plugging into $\Delta PV\approx -\mathbf{k}^\top\boldsymbol{\delta}$ gives
$$\Delta PV \approx -\bigl(k_2\delta_2+k_5\delta_5+k_{10}\delta_{10}\bigr)=k_5\left(\frac{\delta_2+\delta_{10}}{2}-\delta_5\right)=k_5\,\Delta(\text{AVG Fly}).$$
So the “risk-weight scale” is essentially the **belly DV01** (in $\$\!/\text{bp}$) under that weighting convention.

**Check (reproduce the worked fly scenario):** In Section 16.5.2, $\boldsymbol{\delta}=(-5,+10,-5)$ bp implies $\Delta(\text{AVG Fly})=\tfrac{-5-5}{2}-10=-15$ bp. With $k_5=+\$45{,}000/\text{bp}$, the predicted P\&L is $45{,}000\times(-15)=-\$675{,}000$, matching the scenario table.

---

## 16.6 Regression Hedging: Minimum-Variance Hedge Ratios

Often we cannot hedge a specific maturity with an instrument of identical maturity. Regression (or, equivalently, the minimum-variance hedge ratio) formalizes the best *linear* proxy hedge given a chosen sample and holding horizon.

### 16.6.1 Anchor: The Minimum-Variance (Regression) Hedge Ratio

Pick a target change $\Delta S$ and a hedge change $\Delta F$ measured over the same horizon (these could be PV changes, price changes, or yield changes — but pick variables consistent with how you measure P&L). Assume the relationship is approximately linear:
$$
\Delta S = a + b\,\Delta F + \epsilon.
$$
If you hedge by taking hedge ratio $h$, the “hedged change” is
$$
\Delta S - h\,\Delta F = a + (b-h)\,\Delta F + \epsilon.
$$
The standard deviation of the hedged change is minimized by setting $h=b$. This minimum-variance hedge ratio is the regression slope:
$$
h^* = b = \frac{\operatorname{Cov}(\Delta S,\Delta F)}{\operatorname{Var}(\Delta F)} = \rho\frac{\sigma_S}{\sigma_F}.
$$
In a one-factor regression, the **hedge effectiveness** (fraction of variance eliminated by hedging) is the regression $R^2$, which equals $\rho^2$.

**Check (limiting cases):**
- If $\rho=1$ and $\sigma_S=\sigma_F$, then $h^*=1$: the hedge variable mirrors the target one-for-one.
- If $\rho=0$, then $h^*=0$: the hedge variable provides no linear hedge benefit.

**Check (from $R^2$ to residual dollars):** In a one-factor hedge, $R^2=\rho^2$ and the residual standard deviation is $\sigma_\varepsilon=\sigma_S\sqrt{1-R^2}$. Example: if the target has $\sigma_S=1.5$ bp/day and $\rho=0.90$ ($R^2=0.81$), then $\sigma_\varepsilon\approx 1.5\sqrt{0.19}\approx 0.65$ bp/day. On a $DV01=+\$60{,}000/\text{bp}$ position, the 1-day residual P\&L volatility is still about $60{,}000\times 0.65\approx \$39{,}000$. Over $H$ days, a rough i.i.d. scaling is $\sigma\propto \sqrt{H}$.

### 16.6.2 Translating a Yield Regression into Notionals (DV01 Scaling)

Often desks regress **yield changes** (in bp) between a target maturity and a hedge maturity:
$$
\Delta y^{\text{tgt}}_t = \alpha + \beta\,\Delta y^{\text{hedge}}_t + \varepsilon_t.
$$
Using the first-order PV approximation (this book’s sign convention)
$$
\Delta PV \approx -DV01\cdot \Delta y_{\text{bp}},
$$
a hedged position with signed hedge units $N$ (positive = long the hedge instrument) has:
$$
\Delta PV_{\text{hedged}} \approx -DV01_{\text{tgt}}\,\Delta y^{\text{tgt}}_{\text{bp}} - \bigl(N\,DV01_{\text{hedge}}\bigr)\,\Delta y^{\text{hedge}}_{\text{bp}}.
$$
Substituting the regression model and choosing $N$ so the coefficient of $\Delta y^{\text{hedge}}_{\text{bp}}$ vanishes gives the DV01-scaled hedge ratio:
$$
\boxed{N^* \approx -\beta \frac{DV01_{\text{tgt}}}{DV01_{\text{hedge}}}.}
$$
What remains is residual P&L driven by $\varepsilon_t$.

**Check (toy numbers):** if $DV01_{\text{tgt}}=+\$50{,}000/\text{bp}$, $DV01_{\text{hedge}}=+\$20{,}000/\text{bp per unit}$, and $\beta=0.80$, then $N^*\approx -2.0$ hedge units (short 2 units).

### 16.6.3 Residual Risk: Standard Error $\rightarrow$ Dollars

If the regression residual has standard deviation $\sigma_\varepsilon$ in bp over your horizon, the residual P&L volatility is approximately:
$$
\boxed{\sigma_{\Delta PV}\approx DV01_{\text{tgt}}\;\sigma_\varepsilon.}
$$

> **Desk Reality:** Regression/PCA hedges are only as good as your mapping. Make the independent variable match how your desk reports risk (yield, PV, or par-quote changes), and validate the hedge with explicit level/twist/fly scenarios (not only “beta-neutral” labels).

### 16.6.4 Multiple Hedges (Brief)

With two hedge instruments, regress $\Delta y^{\text{tgt}}$ on $(\Delta y^{(1)},\Delta y^{(2)})$ and apply the same DV01 scaling to get two notionals. This is the same linear-algebra structure as Section 16.4: you are choosing weights so exposure to selected directions is small, accepting residual risk elsewhere.

---

## 16.7 Practical Notes and Pitfalls

### 16.7.1 The “Parallel Fallacy”

“Net DV01 $\approx 0$” means “small P&L under a particular parallel-shock definition.” It does not mean “low rate risk.” For example, a KR01 vector $(+2{,}000,\,-4{,}000,\,+2{,}000)$ at (2y, 10y, 30y) sums to zero but is a large *curvature* bet: it wins when the belly cheapens relative to the wings, and loses when the belly richens.

### 16.7.2 Granularity vs. Tradability

Key-rate and bucket frameworks can be as granular as you like, but hedge instruments are not. In practice you often:
- choose key tenors at liquid benchmarks, and/or
- map illiquid buckets to liquid proxies (nearest-tenor, PV-weighted averages, or factor-based mapping).

Mapping is itself a risk: if the relationship between the bucket and the proxy changes, your hedge “looks neutral” in the report but moves in real P&L.

### 16.7.3 What Breaks Curve Hedges (Checklist)

- **Bump mismatch:** different bump object (par vs zero vs yield) or different curve rebuild/interpolation rule.
- **Non-locality:** a “key” bump spills into neighbors; shocks are not as local as you assume.
- **Nonlinearity:** convexity and optionality (Chapter 13) make the linear approximation poor in large moves.
- **Carry/rolldown/funding:** over longer holds, P&L is not only “shock × DV01”.
- **Transaction costs and liquidity:** you may be unable to rebalance when the curve reshapes.
- **Factor drift:** PCA loadings and regression betas are sample-dependent; correlations can change.

### 16.7.4 Scenario Suite (Minimal)

Before relying on a curve hedge, run a small scenario suite using the same bump conventions as your risk numbers:
- **Level:** $(+10,+10,+10,\dots)$ bp.
- **Twist:** short end up / long end down (and the reverse).
- **Fly:** belly up / wings down (and the reverse).
- **Local node:** bump one key (or one instrument quote) and reprice.
- **Stress combo:** mix level + twist + fly (to see if “small residuals” add up).

---

## Summary
- DV01 hedges one *direction* (a chosen parallel shock); curve trades care about twists and butterflies.
- A KR01 (key-rate DV01) report makes rate risk a vector, enabling scenario P&L via a dot product.
- Hedging becomes linear algebra: choose instruments and weights to neutralize the directions you care about, accepting residual risk elsewhere.
- Butterflies use three legs to impose two neutrality constraints (often level and slope), leaving curvature exposure.
- PCA factors are an empirical compression of curve changes; they can be hedge-efficient but are definition- and sample-dependent.
- Regression hedging (minimum-variance hedge ratios) formalizes hedging with imperfect proxies and makes residual risk explicit.

## Key Concepts

| Concept | Definition (this chapter’s conventions) | Why It Matters |
|---|---|---|
| Level / slope / curvature | Stylized curve reshapes (parallel, twist, belly-vs-wings) | Names “what move you’re hedging” |
| Shock vector $\boldsymbol{\delta}$ | Upward yield/zero shocks in bp at chosen keys | Scenario P&L input |
| $DV01$ | $PV(\text{rates down }1\text{bp})-PV(\text{base})$ | Converts bp shocks into dollars |
| $\mathrm{KR01}_k$ | $PV(\text{key }k\text{ down }1\text{bp})-PV(\text{base})$ | Locates risk along the curve |
| Scenario P&L | $\Delta PV \approx -\mathbf{k}^\top\boldsymbol{\delta}$ | Unit/sign-safe first-order P&L |
| Hedge matrix $\mathbf{H}$ | Columns are hedge KR01 vectors per hedge unit | Solves multi-instrument hedges |
| Butterfly | 3-leg trade targeting curvature with neutrality constraints | The canonical “curve-shape” trade |
| PCA loading $\mathbf{u}_j$ | Eigenvector of yield-change covariance | Defines factor direction |
| PCA score $z_{j,t}$ | Projection $\mathbf{u}_j^\top \Delta \mathbf{y}_t$ | Realized factor move |
| Var share | $\lambda_j/\mathrm{tr}(\Sigma)$ | “How much variance” a factor explains |
| Hedge ratio $h^*$ | $\mathrm{Cov}(\Delta X,\Delta Y)/\mathrm{Var}(\Delta Y)$ | Minimum-variance proxy hedge |
| Residual risk | Variance not removed by hedge (e.g., $1-R^2$) | What you still own after hedging |

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $1\text{bp}$ | one basis point | $10^{-4}$ in rate decimal |
| $PV$ | present value | currency; positive = asset |
| $DV01$ | dollar value of 1bp | currency per bp; rates down 1bp convention |
| $\mathrm{KR01}_k$ | key-rate DV01 at key $k$ | currency per bp; key $k$ down 1bp convention |
| $\mathbf{k}$ | KR01 vector | currency per bp |
| $\boldsymbol{\delta}$ | upward shock vector | bp |
| $\mathbf{H}$ | hedge matrix | (currency per bp) per hedge unit |
| $\mathbf{n}$ | hedge weights / notionals | hedge units; signed (long +) |
| $\Delta \mathbf{y}_t$ | yield-change vector | bp over horizon |
| $\Sigma$ | covariance of $\Delta \mathbf{y}$ | bp$^2$ |
| $\mathbf{u}_j$ | PCA loading vector | unitless; normalized to $\|\mathbf{u}_j\|=1$ |
| $\lambda_j$ | PCA eigenvalue | bp$^2$ |
| $z_{j,t}$ | PCA score | bp |
| $\beta$ | regression slope (yield regression) | bp/bp (unitless) |
| $\rho$ | correlation | unitless in $[-1,1]$ |
| $\varepsilon_t$ | regression residual | bp |
| $R^2$ | regression goodness-of-fit | unitless in $[0,1]$ |

## Flashcards

| # | Question | Answer |
|---|---|---|
| 1 | What does “DV01-neutral” actually assume? | It assumes a specific parallel-shock definition; non-parallel reshapes can still cause P&L. |
| 2 | State this book’s DV01 convention. | $DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$. |
| 3 | What is $\mathrm{KR01}_k$? | PV change when key $k$ is shocked down 1bp (with a specified bump/rebuild rule). |
| 4 | Why can two systems disagree on KR01? | Different bump objects and curve rebuild/interpolation rules. |
| 5 | Write the first-order scenario P&L formula in KR01 form. | $\Delta PV \approx -\mathbf{k}^\top\boldsymbol{\delta}$ for upward shocks $\boldsymbol{\delta}$ (bp). |
| 6 | Unit check for $-\mathbf{k}^\top\boldsymbol{\delta}$? | $(\$/\text{bp})\cdot(\text{bp})=\$$. |
| 7 | What is a twist shock in 2y/5y/10y toy form? | $(+1,0,-1)$ bp (short up, long down). |
| 8 | What is the “parallel fallacy”? | Interpreting net DV01 near zero as “low rate risk.” |
| 9 | What constraints does a 3-leg butterfly typically impose? | Two neutrality constraints (often level and slope), leaving curvature exposure. |
| 10 | What does PCA produce? | Orthogonal loadings $\mathbf{u}_j$ and scores $z_{j,t}$ that summarize yield changes. |
| 11 | What is “variance explained” in PCA? | $\lambda_j/\mathrm{tr}(\Sigma)$, the fraction of total variance attributed to factor $j$. |
| 12 | What is the minimum-variance hedge ratio $h^*$? | $h^*=\mathrm{Cov}(\Delta X,\Delta Y)/\mathrm{Var}(\Delta Y)=\rho\sigma_X/\sigma_Y$. |
| 13 | How do you translate a yield regression $\beta$ into hedge units? | $N^* \approx -\beta\,DV01_{\text{tgt}}/DV01_{\text{hedge}}$. |
| 14 | What does $R^2$ tell you for a one-factor regression hedge? | The fraction of target variance explained; $1-R^2$ is residual variance share. |
| 15 | Name two reasons a curve hedge “breaks.” | Bump mismatch; factor/beta drift; convexity; mapping/proxy changes; liquidity/TC. |

## Mini Problem Set

1. (Compute) A position has $\mathbf{k}=(+200,\,+1500,\,+800)$ $\$/\text{bp}$ at (2y, 5y, 10y). Compute $\Delta PV$ for an upward shock $\boldsymbol{\delta}=(+3,\,+0,\,-2)$ bp.
2. (Compute) You want a 50/50 DV01-weighted 2s–5s–10s butterfly. DV01 per \$1mm is: 2y = \$210/bp, 5y = \$470/bp, 10y = \$860/bp. If you receive \$80mm 5y, what 2y and 10y notionals (pay fixed) match the 50/50 DV01 split?
3. (Compute) Regression of yield changes gives $\beta=0.75$ for target vs hedge. Your target DV01 is $+\$60{,}000/\text{bp}$. The hedge instrument DV01 is $+\$25{,}000/\text{bp}$ per unit. What hedge units $N^*$ do you take (sign matters)?
4. (Concept) Give an example of a KR01 vector that has net DV01 $\approx 0$ but large curvature risk. Explain the P&L intuition under a fly shock.
5. (Concept) Compare KR01 hedging and PCA hedging: what is gained and what can go wrong?
6. (Desk) Your risk report shows $\sum_k \mathrm{KR01}_k$ differs materially from the reported parallel DV01. List three reasons this can happen.
7. (Desk) You hedge a butterfly to be twist-neutral under $(+1,0,-1)$ at (2y,5y,10y). What alternative twist/fly scenarios would you run to check robustness?
8. (Compute) A regression hedge has residual standard deviation $\sigma_\varepsilon=0.6$ bp/day. The target DV01 is $+\$40{,}000/\text{bp}$. What is the 1-day 1-sigma residual P&L?

### Solution Sketches (Selected)
1. $\Delta PV \approx -\mathbf{k}^\top\boldsymbol{\delta}=-(200\cdot 3 + 1500\cdot 0 + 800\cdot(-2))=-(600-1600)=+\$1{,}000.$
2. Belly DV01: $80\times 470=+\$37{,}600/\text{bp}$. Each wing targets $-\$18{,}800/\text{bp}$. So $N_2\approx 18{,}800/210\approx \$89.5\text{mm}$ (pay fixed) and $N_{10}\approx 18{,}800/860\approx \$21.9\text{mm}$ (pay fixed).
3. $N^* \approx -\beta\,DV01_{\text{tgt}}/DV01_{\text{hedge}}= -0.75\times 60{,}000/25{,}000=-1.8$ units (short 1.8 hedge units against the long target).
6. Possible reasons: different bump objects (par/zero/yield); different curve rebuild/interpolation rules; key-rate shift family does not sum to a true parallel shift; DV01 computed on a different curve set (e.g., discount vs projection); numerical noise from bump-and-reprice.
8. $\sigma_{\Delta PV}\approx DV01\cdot\sigma_\varepsilon = 40{,}000\times 0.6=\$24{,}000$ per day (first-order).

## References
- (Tuckman & Serrat, *Fixed Income Securities: Tools for Today’s Markets*, “Key Rate Shifts”; “Trading Case Study: A 7s–8s–9s Butterfly”; “Multi-Factor Exposures and Risk Management”)
- (Hull, *Options, Futures, and Other Derivatives*, “Calculating the Minimum Variance Hedge Ratio”)
- (Hull, *Risk Management and Financial Institutions*, “Principal Components Analysis”)
- (Ruppert, *Statistics and Data Analysis for Financial Engineering*, “Principal Components Analysis”)
- (Meucci, *Risk and Asset Allocation*, “The continuum limit” / yield-curve dimension reduction discussion)
