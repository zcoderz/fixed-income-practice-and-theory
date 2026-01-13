# Chapter 16: Curve Hedging Beyond DV01 — Twists, Butterflies, and PCA/Regression Intuition

---

## Fact Classification

### (A) Verified Facts (Source-Backed)
- DV01 definition and first-order P&L approximation (Tuckman Ch 5-6)
- DV01 hedging assumes yields shift in parallel; limitation explicitly flagged in source (Tuckman)
- Key-rate DV01 / bucket exposures defined by shifting specific segments (Tuckman Ch 6)
- PCA shows yield curve shifts are largely a linear sum of two or three standard shifts; hedging those helps hedge typical realized shifts (risk management source)
- PCA first PC resembles parallel shift, second resembles slope shift (fixed income source example)
- Regression-based hedging estimates co-movement and chooses hedge ratios to reduce hedging error / P&L variability (Tuckman)
- Least-squares regression uses normal equations / matrix inversion structure (standard source)
- Key-rate hedging quality depends on assumed behavior between key rates (Tuckman)

### (B) Reasoned Inference (Derived from A)
- Scenario P&L formula $\Delta P \approx -d^\top \delta$ (linear combination of key-rate DV01s and shocks)
- Level/slope/curvature factor exposures as linear functionals of KRDV01 vector
- Hedge construction as linear system: exact solve or least squares
- Regression hedge ratio $h = -\beta$ derived from minimizing variance of hedged P&L
- PCA factor exposures computed as dot products with eigenvector loadings

### (C) Speculation (Clearly Labeled; Minimal)
- Specific toy covariance matrices and eigenvalues in PCA examples are illustrative
- Exact implementation details for desk PCA hedging (window, scaling, mapping to tradable hedges) are convention-dependent

---

## Conventions & Notation

### Rates and Curve Points

| Convention | Description |
|------------|-------------|
| Tenor grid | $T_1 < \cdots < T_m$ (e.g., 2y/5y/10y) |
| Rate vector | $y = (y(T_1), \ldots, y(T_m))$ in decimal (e.g., 5% = 0.05) |
| 1 bp | $10^{-4}$ in decimal rate units |
| Scenario shock $\delta$ | Vector of rate changes in bp at key tenors |

### DV01 Sign Convention

- **DV01 > 0** for a standard long fixed-rate bond (PV rises when rates fall)
- Under a +1 bp rate increase, first-order PV change is approximately $-\text{DV01}$
- This is consistent with Chapter 15 conventions

### Key-Rate Bumps

- A "key-rate shift" bumps one key tenor by 1 bp and adjusts intermediate maturities according to a stated interpolation rule
- We assume **linear yield interpolation** when needed, consistent with the key-rate shift description in the source

### First-Order Approximation

Throughout this chapter, hedges are built using linear (first-order) sensitivity. Convexity and optionality are acknowledged but not developed here.

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $P$ | PV (dollars, \$) |
| $y$ | Yield/rate (decimal) |
| $\Delta y$ | Rate change (decimal) |
| $\delta$ | Rate change (bp), so $\Delta y = \delta / 10{,}000$ |
| $\text{DV01}$ | Dollar value of 1 bp (parallel), defined by $\text{DV01} \equiv -\frac{\Delta P}{10{,}000 \, \Delta y}$ |
| $T_1, \ldots, T_m$ | Tenor grid |
| $\text{KRDV01}_i$ | Key-rate DV01 at tenor $T_i$: PV change for a 1 bp decrease in the $i$-th key rate |
| $d \in \mathbb{R}^m$ | Key-rate DV01 vector, $d = (\text{KRDV01}_1, \ldots, \text{KRDV01}_m)^\top$ |
| $\delta \in \mathbb{R}^m$ | Scenario shock vector (bp) at the key tenors |
| $\delta^{(L)}$ | Level shock: $(1, 1, 1)$ on 3-point grid |
| $\delta^{(S)}$ | Slope shock: $(1, 0, -1)$ on 3-point grid |
| $\delta^{(C)}$ | Curvature shock: $(1, -2, 1)$ on 3-point grid |
| $\Sigma$ | Covariance matrix of historical tenor rate changes |
| $\lambda_k$ | Eigenvalues |
| $v_k$ | Eigenvectors ("loadings") |
| $f_{t,k}$ | PC score: $v_k^\top (\Delta y_t - \overline{\Delta y})$ |

---

## Core Concepts

### 1) DV01 and "DV01-Neutral"

**Formal Definition:**

DV01 (as used here) is defined (for small changes) as:

$$\boxed{\text{DV01} \equiv -\frac{\Delta P}{10{,}000 \, \Delta y}}$$

where $\Delta y$ is the (parallel) yield change in decimal units.

A portfolio is **DV01-neutral** if its net DV01 is (approximately) zero.

**Intuition:**

DV01 compresses the portfolio's curve exposure into one number: sensitivity to a small parallel shift in yields.

**Trading / Risk / Portfolio Practice:**

- DV01 is the first line in rate-risk reports and the backbone of "duration-hedging"
- However, DV01 hedging is only correct if the curve moves in parallel; this limitation is explicitly flagged in the source

---

### 2) Curve-Shape Risk and Why DV01 Is Not Enough

**Formal Definition:**

**Curve-shape risk** means exposure to nonparallel yield curve movements. Common families:

- **Twists (steepeners/flatteners):** short and long tenors move in opposite directions
- **Butterflies (curvature):** belly moves relative to wings

**Intuition:**

A portfolio can have zero parallel exposure but still lose money when the curve pivots (slope) or bows (curvature).

**Trading / Risk / Portfolio Practice:**

- Traders run "steepener/flatteners" and "butterflies" as views on curve shape
- Risk managers test portfolios under twist/butterfly scenarios (not just parallel)
- A standard warning in the sources: matching a single duration/DV01 does not hedge nonparallel changes

---

### 3) Key-Rate DV01 (KRDV01) and Key-Rate Hedging

**Formal Definition:**

**Key-rate DV01** at tenor $T_i$ is the PV change from a 1 bp decrease in the $i$-th key rate, where the curve is perturbed via a **key-rate shift** (bump one key rate and adjust intermediate yields by a specified rule).

**Intuition:**

Instead of a scalar DV01, we track a vector of sensitivities across tenors, which can distinguish level vs slope vs curvature concentrations.

**Trading / Risk / Portfolio Practice:**

- If a position's exposures are decomposed into a vector $d$, then using multiple hedges you can target (approximately) multiple curve moves
- The book emphasizes that more key rates improves hedging but requires stronger assumptions about how the curve behaves between key points

---

### 4) Scenario-Based Hedging (Designed Shocks)

**Formal Definition:**

A **scenario shock** is an explicit vector $\delta$ of changes (bp) at key tenors. A **scenario-based hedge** targets zero (or small) PV change under one or more scenarios.

**Intuition:**

You do not need to "believe in a model." You select a set of plausible curve deformations (parallel, twist, butterfly) and neutralize the portfolio's linearized response.

**Trading / Risk / Portfolio Practice:**

Scenario suites are used to validate that a DV01 hedge is not accidentally a steepener or butterfly risk.

---

### 5) Regression Hedging (Exposure Estimation)

**Formal Definition (Source-Backed):**

Regression is used to estimate the relationship between yield changes of two securities:

$$\Delta y_A = a + b \, \Delta y_B + \varepsilon$$

where $b$ is estimated by least squares. This framework is used to choose hedge ratios that reduce hedging error (minimize the squared error / variability).

**Intuition:**

If instrument A tends to move $b$ times as much as instrument B, then a hedge ratio based on $b$ can outperform a purely DV01-matching hedge.

**Trading / Risk / Portfolio Practice:**

Regression-based hedges are used when "same-duration" is insufficient because instruments do not track perfectly (e.g., different sectors, different benchmarks, different curve points).

---

### 6) PCA Intuition (Empirical Factor Basis)

**Formal Definition:**

**PCA (Principal Component Analysis)** finds orthogonal linear combinations of correlated variables (rate changes at multiple tenors) that explain most variance. Practically, yield curve changes often behave like a linear combination of a small number (2-3) of standard shifts.

**Intuition:**

PCA produces "typical" curve shift shapes. The first factor usually resembles a level shift, the second a slope change, and the third a curvature move (though shapes depend on the dataset). The source gives an example where the first two principal components explain most of the variation and are interpreted as parallel and slope shifts.

**Trading / Risk / Portfolio Practice:**

- PCA factors can be used as a compact risk report ("factor DV01s") and as a hedge target
- The risk-management source explicitly states that PCA shows yield curve shifts are largely a linear sum of two or three standard shifts, and hedging those shifts tends to hedge typical realized shifts

---

## Math and Derivations

### 2.1 DV01 and First-Order P&L

From the DV01 definition in the source:

$$\text{DV01} \equiv -\frac{\Delta P}{10{,}000 \, \Delta y}$$

Rearrange:

$$\Delta P \approx -10{,}000 \cdot \text{DV01} \cdot \Delta y$$

If we express the rate shock in bp as $\delta$ (so $\Delta y = \delta / 10{,}000$), then:

$$\boxed{\Delta P \approx -\text{DV01} \cdot \delta}$$

**Unit Check:**

| Quantity | Units |
|----------|-------|
| DV01 | \$/bp |
| $\delta$ | bp |
| Product | \$ $\checkmark$ |

**Sanity Check:**

- If $\delta = +1$ bp (rates up), then $\Delta P \approx -\text{DV01} < 0$ for a long bond with $\text{DV01} > 0$ $\checkmark$
- If $\delta = -1$ bp (rates down), $\Delta P \approx +\text{DV01}$ $\checkmark$

**Assumption:** This is a linear approximation; convexity and other second-order effects are ignored here.

---

### 2.2 Key-Rate DV01 Vectors and Scenario P&L

Let $d = (\text{KRDV01}_1, \ldots, \text{KRDV01}_m)^\top$ and let $\delta$ be a vector of key-rate shocks (bp). Under linearization, the PV change is approximated by the dot product:

$$\boxed{\Delta P \approx -d^\top \delta}$$

**Why This Is Consistent with the Key-Rate Construction:**

Each $\text{KRDV01}_i$ is itself defined as the PV change under a 1 bp key-rate shift (with interpolation rules for intermediate yields). Therefore, for small changes, a linear combination of key-rate shifts yields a linear combination of PV changes. The source stresses that this relies on how intermediate yields are assumed to move and that hedging quality depends on that assumption.

**Assumptions (Explicit):**

1. PV is approximately linear in small rate changes (ignore convexity)
2. Any scenario $\delta$ is interpreted on the same key-rate grid and interpolation convention used to compute $d$
3. Actual yield moves can be more complex than a small number of key-rate shifts; the source notes this limitation

---

### 2.3 Level / Slope / Curvature Factors as Linear Functionals of $d$

Pick a tenor grid. For illustration, use $(2y, 5y, 10y)$ and define:

$$\delta^{(L)} = \begin{pmatrix} 1 \\ 1 \\ 1 \end{pmatrix}, \quad \delta^{(S)} = \begin{pmatrix} 1 \\ 0 \\ -1 \end{pmatrix}, \quad \delta^{(C)} = \begin{pmatrix} 1 \\ -2 \\ 1 \end{pmatrix}$$

Then for any key-rate DV01 vector $d = (d_2, d_5, d_{10})^\top$:

**Level exposure** (P&L per +1 bp level shift):
$$\Delta P_L \approx -d^\top \delta^{(L)} = -(d_2 + d_5 + d_{10})$$

**Slope exposure** (P&L per +1 bp slope shift):
$$\Delta P_S \approx -d^\top \delta^{(S)} = -(d_2 - d_{10})$$

**Curvature exposure** (P&L per +1 bp curvature shift):
$$\Delta P_C \approx -d^\top \delta^{(C)} = -(d_2 - 2d_5 + d_{10})$$

**Unit Check:** Each exposure is a linear combination of \$/bp numbers, giving \$/bp $\checkmark$

**Important Conceptual Point:** A portfolio can be DV01-neutral for a parallel shift (level) but still have nonzero $\Delta P_S$ or $\Delta P_C$.

---

### 2.4 Hedge Construction as a Linear System

Suppose you have a target position with exposure vector $e_0 \in \mathbb{R}^k$ (where $k$ is the number of risks you want to hedge: key rates, or factors). You choose $n$ hedge instruments with exposure vectors $e_1, \ldots, e_n \in \mathbb{R}^k$. Let $x \in \mathbb{R}^n$ be hedge notionals (positive = long, negative = short). Define the combined exposure:

$$e_{\text{tot}}(x) = e_0 + \sum_{j=1}^{n} x_j e_j$$

**Exact Solve (Square System):**

If $n = k$ and the hedge exposure matrix $E = [e_1 \; \cdots \; e_n]$ is invertible:

$$e_{\text{tot}} = 0 \implies Ex = -e_0, \quad \boxed{x = -E^{-1} e_0}$$

**Least Squares (Overdetermined System):**

If $k > n$ (more constraints than hedges), choose $x$ to minimize residual exposure:

$$\min_x \|e_0 + Ex\|^2$$

The standard least-squares solution uses normal equations:

$$\boxed{E^\top E \, x = -E^\top e_0, \quad x = -(E^\top E)^{-1} E^\top e_0}$$

when $E^\top E$ is invertible. This is the same matrix-inversion structure shown for least-squares regression in the source material.

**Unit Check:**

If $e$ is \$/bp, then $Ex$ must also be \$/bp. This requires $x_j$ be a dimensionless "position scale" (e.g., "number of contracts" or "\$ notional in some normalized unit"). In real desks, you keep a mapping from notional to DV01 to ensure unit consistency.

---

### 2.5 Regression Hedge Ratio (Link to Minimizing Hedging Error)

The source motivates regression-based hedging: estimate

$$\Delta y_A = a + b \, \Delta y_B + \varepsilon$$

then choose a hedge in $B$ that reduces the variance of the hedged position by minimizing squared residuals (least squares).

**Connecting Regression to a Hedge Ratio (Derived from Source's Logic):**

If (first-order) price changes satisfy $\Delta P \approx -\text{DV01} \cdot \delta$, then for two instruments:

$$\Delta P_A \approx -\text{DV01}_A \, \Delta y_A \cdot 10{,}000, \quad \Delta P_B \approx -\text{DV01}_B \, \Delta y_B \cdot 10{,}000$$

If $\Delta y_A \approx b \, \Delta y_B$, then:

$$\Delta P_A \approx b \left( \frac{\text{DV01}_A}{\text{DV01}_B} \right) \Delta P_B$$

A hedge ratio that targets $\Delta P_A + h \, \Delta P_B \approx 0$ is:

$$\boxed{h \approx -b \frac{\text{DV01}_A}{\text{DV01}_B}}$$

(This is the standard "regression beta $\times$ DV01 ratio" intuition consistent with the source's regression hedging setup.)

---

### 2.6 PCA Setup (Covariance $\to$ Eigenvectors $\to$ Factors)

Given historical tenor changes $\Delta y_t \in \mathbb{R}^m$, compute sample covariance:

$$S = \frac{1}{n} \sum_{t=1}^{n} (\Delta y_t - \overline{\Delta y})(\Delta y_t - \overline{\Delta y})^\top$$

and its eigen-decomposition:

$$S = G L G^\top$$

where columns of $G$ are eigenvectors ("loading vectors") and $L = \text{diag}(\lambda_1, \ldots, \lambda_m)$ are eigenvalues.

The fraction of total variance explained by the first $k$ PCs is:

$$\boxed{\frac{\sum_{j=1}^{k} \lambda_j}{\sum_{j=1}^{m} \lambda_j}}$$

(using $\sum \lambda_j = \text{trace}(\Sigma)$).

**Interpretation for Rates:**

The fixed-income source provides an example where the first PC is "parallel shift," second is "slope," and together explain most variance. Another source states that PCA shows yield curve shifts are largely a linear sum of two or three standard shifts and hedging those helps hedge typical realized shifts.

---

## Measurement & Risk

### 3.1 Curve-Shape Risk Taxonomy

We work with three canonical "shape" families:

**Level (Parallel Shifts):**
- **Definition:** $\delta(T)$ is constant across maturities: all rates move by the same bp
- **Effect:** Captured by DV01 (scalar) if the move is truly parallel

**Slope (Twists / Steepeners / Flatteners):**
- **Definition:** Short and long maturities move in opposite directions (or with different magnitudes), changing the curve slope
- **Example scenario:** +10 bp at 2y, 0 at 5y, $-10$ bp at 10y

**Curvature (Butterflies):**
- **Definition:** The belly moves relative to wings; the curve becomes more/less "humped"
- **Example scenario:** +10 bp at 5y with 0 bp at 2y and 10y; or an orthogonalized butterfly +10 bp at 2y and 10y and $-20$ bp at 5y

These three families are also aligned with how PCA often summarizes rate moves (level/slope/curvature-type components), with sources providing empirical evidence that a small number of components explains a large fraction of yield changes.

---

### 3.2 How Key-Rate DV01 Vectors Relate to Scenarios and Factors

**Key-Rate DV01 Vector as a "Basis":**

$d$ gives first-order sensitivity at each key tenor under the chosen key-rate shift method. If actual curve moves can be approximated by combinations of key-rate moves, then $\Delta P \approx -d^\top \delta$. The source emphasizes that hedging quality depends on the assumed behavior between key rates and may deteriorate for larger shifts.

**Scenario Shocks (Designed Shocks):**

A scenario is just a particular $\delta$. Applying a parallel shock corresponds to $\delta \propto \mathbf{1}$. Twists and butterflies correspond to other shapes; their P&L is computed via the dot product.

**Factor Models (PCA / Regression) as an Empirical Basis:**

PCA produces orthogonal directions $v_k$ in tenor space that explain historical variance. The fixed-income source reports that the first PC resembles a parallel shift and the second a slope shift, explaining most variance in its example. The risk management source explicitly suggests PCA as an alternative to calculating many deltas, since realized curve moves are largely combinations of a few standard shifts.

---

### 3.3 What "DV01-Neutral" Means — And Why It Can Still Have Slope/Curvature Exposure

**DV01-Neutral (Level-Neutral):**

$$\text{DV01}_{\text{net}} \approx 0$$

means PV does not change (to first order) under a small parallel shift.

**Why It Can Still Have Slope/Curvature Exposure:**

DV01 only guards against a single scenario family. The source explicitly warns that DV01 requires yields to shift in parallel; otherwise the hedge can fail.

More generally, a portfolio can have key-rate DV01 vector components that sum to zero (hence DV01 $\approx$ 0), but the vector can still have nonzero projection on slope or curvature shock vectors.

**Source-Backed Intuition via Bucket Exposures:**

The source gives an example: even if "total bucket exposures are both 0," it is not accurate to say the portfolio has no risk; it can still have curve risk (e.g., exposed to steepening).

---

### 3.4 Hedge-Construction Workflow (Desk-Relevant)

A practical curve-hedge workflow:

**1. Choose a Risk Representation:**
- **Key-rate DV01 vector:** $d$ across tenors (detailed, interpretable)
- **Designed factor set:** (level, slope, curvature) scenarios $\delta^{(L)}, \delta^{(S)}, \delta^{(C)}$
- **Empirical factor set (PCA):** first $k$ principal components from historical data (compact; data-driven). Sources support that PCA yields a small set of standard shifts capturing most variation.

**2. Compute Exposures:**
- Key-rate DV01s via the key-rate shift method
- Scenario factor exposures via dot products (e.g., $E_S = d^\top \delta^{(S)}$)
- Regression betas via OLS on yield changes (source-backed) or P&L changes (derived extension)

**3. Choose Hedge Instruments:**
- Liquid points on the curve (cash bonds, futures, swaps) with stable DV01/KRDV01 profiles
- Source notes that par bonds are convenient for key-rate hedging: each par bond is "naturally" tied to one key rate under the key-rate construction, and their key-rate exposures equal their DV01s in the example

**4. Solve for Hedge Notionals:**
- **Exact solve** if you have as many hedges as constraints and the system is well-conditioned
- **Least squares** if you have more curve points than hedges, using normal equations (same structure as least-squares regression)

**5. Validate with a Scenario Set:**
- At minimum: parallel, twist, butterfly, plus at least one "random" perturbation
- Validate both:
  - PV change under scenarios
  - Residual exposure vector (or factor residuals)

---

### 3.5 Multi-Curve Preview (Brief, Not a Full Chapter)

Post-2007–2009, interest rate practice often requires multiple inter-related curves rather than a single curve, making both curve construction and risk management more complex.

In a two-curve setup, one distinguishes a **discount curve** and an "index" (projection) curve; the source notes index curves can be treated as separate pseudo-discount curves and discusses assumptions relating them.

**Implication for This Chapter:** "DV01" and "key-rate DV01" become curve-specific (discount vs projection), so a hedge that is neutral in one curve may still have exposure to the other (basis risk). This chapter treats that as a preview only; desk conventions will dictate the exact decomposition.

---

## Worked Examples

**Global Conventions for All Examples:**

- DV01/KRDV01 measured in \$/bp for a 1 bp decrease in the relevant rate(s)
- Scenario shocks $\delta$ are in bp for rate increases (positive = rates up)
- First-order P&L approximation: $\Delta P \approx -d^\top \delta$
- "Positive exposure" means DV01/KRDV01 > 0 for a long fixed-rate position

---

### Example A: DV01-Neutral but Twist-Exposed

**Goal:** Build a 2-instrument portfolio with net parallel DV01 $\approx 0$, then show nonzero P&L under a twist scenario.

**Tenor grid / curve points:** 2y and 10y (two key rates)

**Units:** DV01 in \$/bp; P&L in \$

**Instrument DV01s (toy):**
- Instrument 1 (2y): $\text{DV01}_{2y} = +1000$ \$/bp
- Instrument 2 (10y): $\text{DV01}_{10y} = +2500$ \$/bp

**Step 1: Choose notionals to be DV01-neutral under parallel shifts**

Let positions be $n_1$ (long 2y) and $n_2$ (short 10y; negative means short).

Set $n_1 = +1$. Choose $n_2$ so net DV01 is zero:

$$\text{DV01}_{\text{net}} = n_1(1000) + n_2(2500) = 0 \implies n_2 = -\frac{1000}{2500} = -0.4$$

**Portfolio key-rate DV01 vector:**

$$d = \begin{pmatrix} d_{2y} \\ d_{10y} \end{pmatrix} = \begin{pmatrix} +1000 \\ -0.4 \times 2500 \end{pmatrix} = \begin{pmatrix} 1000 \\ -1000 \end{pmatrix}$$

**Step 2: Parallel shock test**

Parallel shock: +10 bp at both tenors

$$\delta^{(\text{parallel})} = \begin{pmatrix} 10 \\ 10 \end{pmatrix}$$

P&L:

$$\Delta P \approx -d^\top \delta = -(1000 \cdot 10 + (-1000) \cdot 10) = -(10000 - 10000) = 0$$

**Step 3: Twist scenario**

Twist: +10 bp at 2y and $-10$ bp at 10y

$$\delta^{(\text{twist})} = \begin{pmatrix} 10 \\ -10 \end{pmatrix}$$

P&L:

$$\Delta P \approx -(1000 \cdot 10 + (-1000) \cdot (-10)) = -(10000 + 10000) = -20{,}000$$

**Result:** DV01-neutral portfolio (parallel) still loses $-\$20{,}000$ under a twist.

**Curve-Shape Lens:** The portfolio has slope exposure because its key-rate vector is not zero; it is long short-end DV01 and short long-end DV01, so a steepener/flattening move generates P&L even when parallel DV01 cancels.

---

### Example B: Butterfly Exposure

**Goal:** Apply a butterfly shock and compute P&L; relate to curvature risk.

**Tenor grid:** 2y / 5y / 10y

**Position key-rate DV01 vector (toy):**

$$d = \begin{pmatrix} d_{2y} \\ d_{5y} \\ d_{10y} \end{pmatrix} = \begin{pmatrix} 500 \\ 1500 \\ 500 \end{pmatrix} \text{ (\$/bp)}$$

**Butterfly shock:** +10 bp at 5y, 0 at 2y and 10y

$$\delta^{(\text{bfly})} = \begin{pmatrix} 0 \\ 10 \\ 0 \end{pmatrix}$$

**P&L Calculation:**

$$\Delta P \approx -d^\top \delta = -(500 \cdot 0 + 1500 \cdot 10 + 500 \cdot 0) = -15{,}000$$

**Result:** $-\$15{,}000$ P&L under the "belly up 10 bp" butterfly bump.

**Curvature Interpretation:** This portfolio is concentrated in the 5y bucket (belly). A pure belly move is a curvature-like deformation (belly vs wings). In practice, traders isolate curvature more cleanly using an "orthogonalized" butterfly such as $(+1, -2, +1)$, which removes level and slope components (see Example E for that construction).

---

### Example C: Key-Rate Vector Exposures

**Goal:** Provide a KRDV01 vector across 7 tenors; interpret concentration; map slope/curvature scenarios to the vector.

**Tenor grid:** 1y, 2y, 3y, 5y, 7y, 10y, 30y

**KRDV01 vector for a position (toy, \$/bp):**

| Tenor | 1y | 2y | 3y | 5y | 7y | 10y | 30y |
|-------|-----|-----|-----|-----|-----|------|------|
| $d_i = \text{KRDV01}_i$ | 200 | 500 | 600 | 900 | 700 | 500 | 200 |

**Step 1: Where is risk concentrated?**

- The largest KRDV01 is at 5y (900) and 7y (700)
- Short-end (1–3y) sums to $200 + 500 + 600 = 1300$
- Long-end (10–30y) sums to $500 + 200 = 700$
- So the portfolio is **belly-heavy**

**Step 2: Level (parallel) exposure**

Level shock: +1 bp at all points

$$\delta^{(L)} = (1, 1, 1, 1, 1, 1, 1)^\top$$

Dot product:

$$d^\top \delta^{(L)} = 200 + 500 + 600 + 900 + 700 + 500 + 200 = 3600$$

So P&L for +1 bp parallel:

$$\Delta P_L \approx -3600 \text{ dollars}$$

**Step 3: Slope scenario mapping**

Define a simple twist (steepener/flattening-style) scenario:
- Short rates up +1 bp: 1y, 2y, 3y
- Belly unchanged: 5y
- Long rates down $-1$ bp: 7y, 10y, 30y

$$\delta^{(S)} = (1, 1, 1, 0, -1, -1, -1)^\top$$

Compute dot product:

$$d^\top \delta^{(S)} = (200 + 500 + 600) + 0 \cdot 900 - (700 + 500 + 200) = 1300 - 1400 = -100$$

So P&L:

$$\Delta P_S \approx -(-100) = +100$$

**Interpretation:** The portfolio benefits from this particular steepening/flattening shock because it is more exposed to the short end than to the long end under this sign pattern.

**Step 4: Curvature scenario mapping**

Define a curvature scenario (belly up relative to wings):
- Wings (1y, 30y): 0 bp
- 2y and 10y: 0 bp
- Near belly (3y, 7y): +5 bp
- Belly (5y): +10 bp

$$\delta^{(C)} = (0, 0, 5, 10, 5, 0, 0)^\top$$

Dot product:

$$d^\top \delta^{(C)} = 200 \cdot 0 + 500 \cdot 0 + 600 \cdot 5 + 900 \cdot 10 + 700 \cdot 5 + 500 \cdot 0 + 200 \cdot 0$$
$$= 3000 + 9000 + 3500 = 15500$$

So P&L:

$$\Delta P_C \approx -15500$$

**Interpretation:** Because risk is concentrated at 5y/7y, a belly-driven curvature shock is much more dangerous than a twist shock for this position.

---

### Example D: Hedge Level + Slope with 2 Hedges

**Goal:** Hedge a target position using 2 hedge instruments to neutralize:
1. Level exposure (parallel DV01)
2. Slope exposure (2y vs 10y twist factor)

**Tenor grid:** 2y and 10y

**Factor definitions (bp shocks):**

$$\delta^{(L)} = (1, 1)^\top, \quad \delta^{(S)} = (1, -1)^\top$$

**Target key-rate DV01 vector:**

$$d_0 = \begin{pmatrix} 1200 \\ 1800 \end{pmatrix}$$

**Hedge instruments (assume each is a "pure" key-rate hedge):**

This aligns with the idea that a par bond at a key maturity can be used to hedge that key rate (in the source's illustrative setup).

- Hedge 1 (2y instrument): $d_1 = (1000, 0)^\top$
- Hedge 2 (10y instrument): $d_2 = (0, 2500)^\top$

Let hedge notionals be $x_1, x_2$. Total KRDV01:

$$d_{\text{tot}} = d_0 + x_1 d_1 + x_2 d_2$$

**Compute factor exposures:**

For any $d = (d_{2y}, d_{10y})$:

- Level exposure: $E_L = d^\top \delta^{(L)} = d_{2y} + d_{10y}$
- Slope exposure: $E_S = d^\top \delta^{(S)} = d_{2y} - d_{10y}$

**Target:**

$$E_{L,0} = 1200 + 1800 = 3000, \quad E_{S,0} = 1200 - 1800 = -600$$

**Hedges:**

For $d_1 = (1000, 0)$:
$$E_{L,1} = 1000, \quad E_{S,1} = 1000$$

For $d_2 = (0, 2500)$:
$$E_{L,2} = 2500, \quad E_{S,2} = -2500$$

**Solve the 2×2 system:**

We want $E_L = 0$ and $E_S = 0$:

$$\begin{cases} 3000 + 1000 x_1 + 2500 x_2 = 0 \\ -600 + 1000 x_1 - 2500 x_2 = 0 \end{cases}$$

Add the equations:

$$(3000 - 600) + 2000 x_1 = 0 \implies 2400 + 2000 x_1 = 0 \implies x_1 = -1.2$$

Plug into the first equation:

$$3000 + 1000(-1.2) + 2500 x_2 = 0 \implies 3000 - 1200 + 2500 x_2 = 0$$
$$\implies 1800 + 2500 x_2 = 0 \implies x_2 = -0.72$$

**Check (key-rate DV01 residual):**

$$d_{\text{tot}} = \begin{pmatrix} 1200 \\ 1800 \end{pmatrix} + (-1.2) \begin{pmatrix} 1000 \\ 0 \end{pmatrix} + (-0.72) \begin{pmatrix} 0 \\ 2500 \end{pmatrix} = \begin{pmatrix} 0 \\ 0 \end{pmatrix}$$

**Result:** Hedged portfolio is both level- and slope-neutral (indeed, neutral to any combination of 2y and 10y moves).

---

### Example E: Hedge Level + Slope + Curvature with 3 Hedges

**Goal:** Neutralize level, slope, and curvature exposures using 3 hedges.

**Tenor grid:** 2y / 5y / 10y

**Define factors (bp shocks):**

$$\delta^{(L)} = \begin{pmatrix} 1 \\ 1 \\ 1 \end{pmatrix}, \quad \delta^{(S)} = \begin{pmatrix} 1 \\ 0 \\ -1 \end{pmatrix}, \quad \delta^{(C)} = \begin{pmatrix} 1 \\ -2 \\ 1 \end{pmatrix}$$

**Target KRDV01 vector:**

$$d_0 = \begin{pmatrix} 800 \\ 1400 \\ 1100 \end{pmatrix}$$

**Hedge instruments (pure key-rate exposures; toy):**

- 2y hedge: $d_1 = (1000, 0, 0)^\top$
- 5y hedge: $d_2 = (0, 1500, 0)^\top$
- 10y hedge: $d_3 = (0, 0, 2000)^\top$

Let notionals be $x_1, x_2, x_3$.

**Compute factor exposures (dot products with factor shocks):**

For any $d = (d_2, d_5, d_{10})$:

$$E_L = d_2 + d_5 + d_{10}, \quad E_S = d_2 - d_{10}, \quad E_C = d_2 - 2d_5 + d_{10}$$

**Target:**

$$E_{L,0} = 800 + 1400 + 1100 = 3300$$
$$E_{S,0} = 800 - 1100 = -300$$
$$E_{C,0} = 800 - 2(1400) + 1100 = 800 - 2800 + 1100 = -900$$

**Hedges:**

$d_1 = (1000, 0, 0)$:
$$E_{L,1} = 1000, \; E_{S,1} = 1000, \; E_{C,1} = 1000$$

$d_2 = (0, 1500, 0)$:
$$E_{L,2} = 1500, \; E_{S,2} = 0, \; E_{C,2} = -3000$$

$d_3 = (0, 0, 2000)$:
$$E_{L,3} = 2000, \; E_{S,3} = -2000, \; E_{C,3} = 2000$$

**Set up the 3×3 system (neutralize all three factors):**

$$\begin{cases} 3300 + 1000 x_1 + 1500 x_2 + 2000 x_3 = 0 \\ -300 + 1000 x_1 + 0 \cdot x_2 - 2000 x_3 = 0 \\ -900 + 1000 x_1 - 3000 x_2 + 2000 x_3 = 0 \end{cases}$$

**Solve step-by-step:**

From slope equation:
$$-300 + 1000 x_1 - 2000 x_3 = 0 \implies 1000 x_1 - 2000 x_3 = 300 \implies x_1 = 0.3 + 2x_3$$

Sub into level equation:
$$3300 + 1000(0.3 + 2x_3) + 1500 x_2 + 2000 x_3 = 0$$
$$3300 + 300 + 2000 x_3 + 1500 x_2 + 2000 x_3 = 0 \implies 3600 + 4000 x_3 + 1500 x_2 = 0$$
$$\implies x_2 = -2.4 - \frac{8}{3} x_3$$

Sub into curvature equation:
$$-900 + 1000(0.3 + 2x_3) - 3000 x_2 + 2000 x_3 = 0$$
$$-900 + (300 + 2000 x_3) - 3000 x_2 + 2000 x_3 = 0 \implies -600 + 4000 x_3 - 3000 x_2 = 0$$
$$\implies 3000 x_2 = 4000 x_3 - 600$$

Now substitute $x_2 = -2.4 - \frac{8}{3} x_3$:
$$3000 \left( -2.4 - \frac{8}{3} x_3 \right) = 4000 x_3 - 600$$
$$-7200 - 8000 x_3 = 4000 x_3 - 600 \implies -7200 + 600 = 12000 x_3$$
$$\implies -6600 = 12000 x_3 \implies x_3 = -0.55$$

Then:
$$x_1 = 0.3 + 2(-0.55) = -0.8$$
$$x_2 = -2.4 - \frac{8}{3}(-0.55) = -2.4 + 1.4\overline{6} = -0.9\overline{3} = -\frac{14}{15}$$

**Final hedge notionals:**

$$(x_1, x_2, x_3) = \left( -0.8, \; -\frac{14}{15}, \; -0.55 \right)$$

**Check: Residual key-rate DV01 vector**

$$d_{\text{tot}} = \begin{pmatrix} 800 \\ 1400 \\ 1100 \end{pmatrix} + (-0.8) \begin{pmatrix} 1000 \\ 0 \\ 0 \end{pmatrix} + \left( -\frac{14}{15} \right) \begin{pmatrix} 0 \\ 1500 \\ 0 \end{pmatrix} + (-0.55) \begin{pmatrix} 0 \\ 0 \\ 2000 \end{pmatrix} = \begin{pmatrix} 0 \\ 0 \\ 0 \end{pmatrix}$$

**Interpretation:** With three independent hedges tied to the three key tenors, you can neutralize level, slope, and curvature (in this 3-point world).

---

### Example F: Least-Squares Hedge with >3 Hedges

**Goal:** Use 4 hedge instruments to minimize residual exposure across 5 key rates via least squares; show matrix form, normal equations, and resulting notionals.

**Tenor grid (5 points):** 2y / 5y / 7y / 10y / 30y

**Target KRDV01 vector (toy, \$/bp):**

$$d_0 = \begin{pmatrix} 400 \\ 800 \\ 600 \\ 500 \\ 200 \end{pmatrix}$$

**Choose 4 hedge instruments with overlapping exposures (toy, \$/bp):**

- Hedge 1: $h_1 = (500, 0, 0, 0, 0)^\top$ (2y point hedge)
- Hedge 2: $h_2 = (0, 800, 400, 0, 0)^\top$ (5y+7y)
- Hedge 3: $h_3 = (0, 0, 400, 800, 0)^\top$ (7y+10y)
- Hedge 4: $h_4 = (0, 0, 0, 0, 300)^\top$ (30y point hedge)

Let $x = (x_1, x_2, x_3, x_4)^\top$. Define the exposure matrix $E = [h_1 \; h_2 \; h_3 \; h_4]$ (size $5 \times 4$):

$$E = \begin{pmatrix} 500 & 0 & 0 & 0 \\ 0 & 800 & 0 & 0 \\ 0 & 400 & 400 & 0 \\ 0 & 0 & 800 & 0 \\ 0 & 0 & 0 & 300 \end{pmatrix}$$

We want to minimize:
$$\min_x \|d_0 + Ex\|^2$$

**Normal equations:**
$$E^\top E \, x = -E^\top d_0$$

consistent with the least-squares matrix-inversion structure shown in the sources for regression/least squares.

**Step 1: Compute $E^\top E$**

Column norms:
- $\|h_1\|^2 = 500^2 = 250{,}000$
- $\|h_4\|^2 = 300^2 = 90{,}000$
- $\|h_2\|^2 = 800^2 + 400^2 = 640{,}000 + 160{,}000 = 800{,}000$
- $\|h_3\|^2 = 400^2 + 800^2 = 160{,}000 + 640{,}000 = 800{,}000$
- Cross term $h_2^\top h_3 = 400 \cdot 400 = 160{,}000$ (overlap at 7y)

Thus:

$$E^\top E = \begin{pmatrix} 250000 & 0 & 0 & 0 \\ 0 & 800000 & 160000 & 0 \\ 0 & 160000 & 800000 & 0 \\ 0 & 0 & 0 & 90000 \end{pmatrix}$$

**Step 2: Compute $E^\top d_0$**

$$E^\top d_0 = \begin{pmatrix} h_1^\top d_0 \\ h_2^\top d_0 \\ h_3^\top d_0 \\ h_4^\top d_0 \end{pmatrix} = \begin{pmatrix} 500 \cdot 400 \\ 800 \cdot 800 + 400 \cdot 600 \\ 400 \cdot 600 + 800 \cdot 500 \\ 300 \cdot 200 \end{pmatrix} = \begin{pmatrix} 200000 \\ 880000 \\ 640000 \\ 60000 \end{pmatrix}$$

So the RHS is $-E^\top d_0 = (-200000, -880000, -640000, -60000)^\top$.

**Step 3: Solve normal equations**

Equation 1:
$$250000 x_1 = -200000 \implies x_1 = -0.8$$

Equation 4:
$$90000 x_4 = -60000 \implies x_4 = -\frac{2}{3} \approx -0.6667$$

Block for $(x_2, x_3)$:
$$\begin{cases} 800000 x_2 + 160000 x_3 = -880000 \\ 160000 x_2 + 800000 x_3 = -640000 \end{cases}$$

Divide both equations by 160000:
$$\begin{cases} 5x_2 + x_3 = -5.5 \\ x_2 + 5x_3 = -4 \end{cases}$$

From first: $x_3 = -5.5 - 5x_2$

Plug into second:
$$x_2 + 5(-5.5 - 5x_2) = -4 \implies x_2 - 27.5 - 25x_2 = -4 \implies -24x_2 = 23.5$$
$$\implies x_2 = -\frac{23.5}{24} = -0.9791667 = -\frac{47}{48}$$

Then:
$$x_3 = -5.5 - 5 \left( -\frac{47}{48} \right) = -5.5 + \frac{235}{48} = -\frac{264}{48} + \frac{235}{48} = -\frac{29}{48} \approx -0.6041667$$

**Final least-squares hedge notionals:**

$$x = \begin{pmatrix} -0.8 \\ -47/48 \\ -29/48 \\ -2/3 \end{pmatrix} \approx \begin{pmatrix} -0.8000 \\ -0.9792 \\ -0.6042 \\ -0.6667 \end{pmatrix}$$

**Step 4: Compute residual exposures**

Compute $Ex$:
- $x_1 h_1 = (-0.8) \cdot (500, 0, 0, 0, 0) = (-400, 0, 0, 0, 0)$
- $x_2 h_2 = (-47/48) \cdot (0, 800, 400, 0, 0) = (0, -783.333, -391.667, 0, 0)$
- $x_3 h_3 = (-29/48) \cdot (0, 0, 400, 800, 0) = (0, 0, -241.667, -483.333, 0)$
- $x_4 h_4 = (-2/3) \cdot (0, 0, 0, 0, 300) = (0, 0, 0, 0, -200)$

Add to target:

$$d_{\text{res}} = d_0 + Ex = \begin{pmatrix} 400 - 400 \\ 800 - 783.333 \\ 600 - 391.667 - 241.667 \\ 500 - 483.333 \\ 200 - 200 \end{pmatrix} = \begin{pmatrix} 0 \\ 16.667 \\ -33.333 \\ 16.667 \\ 0 \end{pmatrix}$$

**Residual norm (optional diagnostic):**

$$\|d_{\text{res}}\|^2 \approx 0^2 + 16.667^2 + (-33.333)^2 + 16.667^2 + 0^2 \approx 277.8 + 1111.1 + 277.8 \approx 1666.7$$

So $\|d_{\text{res}}\| \approx 40.82$ \$/bp.

**Interpretation:** With only 4 hedges, you cannot perfectly hedge 5 key rates. The LS hedge spreads the error across the 5y/7y/10y region where hedge instruments overlap.

---

### Example G: Regression Hedge Intuition

**Goal:** Use a toy time series of daily P&L for a position and a candidate hedge; compute hedge ratio via regression; compare to a DV01-based ratio.

**Setup:**
- Let $T_t$ be daily P&L of target position (in \$ thousands, i.e., \$k)
- Let $H_t$ be daily P&L of a hedge instrument (in \$k)
- We choose hedge ratio $h$ such that hedged P&L is: $\Pi_t = T_t + h \, H_t$
- $h$ is chosen to minimize variance of $\Pi_t$. In a one-hedge OLS regression of $T_t$ on $H_t$, the minimizing $h$ is $-\beta$, where $\beta$ is the regression slope (same least-squares principle used in the source's regression setup)

**Toy data (10 days, means are zero):**

| Day | $H_t$ (\$k) | $T_t$ (\$k) |
|-----|-------------|-------------|
| 1 | -2 | -2.5 |
| 2 | -1 | -2.0 |
| 3 | 0 | 0.5 |
| 4 | 1 | 1.0 |
| 5 | 2 | 3.5 |
| 6 | -2 | -3.5 |
| 7 | -1 | -1.0 |
| 8 | 0 | -0.5 |
| 9 | 1 | 2.0 |
| 10 | 2 | 2.5 |

**Check means:**

$$\sum H_t = (-2-1+0+1+2) + (-2-1+0+1+2) = 0$$
$$\sum T_t = (-2.5-2.0+0.5+1.0+3.5) + (-3.5-1.0-0.5+2.0+2.5) = 0$$

**Step 1: Compute OLS slope**

With zero means:
$$\beta = \frac{\sum_{t=1}^{10} H_t T_t}{\sum_{t=1}^{10} H_t^2}$$

Compute $\sum H_t^2$:
$$H_t^2: \; 4, 1, 0, 1, 4, 4, 1, 0, 1, 4 \implies \sum H_t^2 = 20$$

Compute $\sum H_t T_t$:
- $(-2)(-2.5) = 5, \; (-1)(-2.0) = 2, \; 0(0.5) = 0, \; 1(1.0) = 1, \; 2(3.5) = 7$
- $(-2)(-3.5) = 7, \; (-1)(-1.0) = 1, \; 0(-0.5) = 0, \; 1(2.0) = 2, \; 2(2.5) = 5$

Sum:
$$\sum H_t T_t = 5 + 2 + 0 + 1 + 7 + 7 + 1 + 0 + 2 + 5 = 30$$

Thus:
$$\beta = \frac{30}{20} = 1.5$$

**Step 2: Hedge ratio**

To minimize variance of $T_t + h H_t$, choose $h = -\beta$:

$$\boxed{h = -1.5}$$

**Interpretation:**

The regression says: "Target P&L is about 1.5× hedge P&L plus residual noise; short 1.5 units of the hedge to reduce variability."

**Compare to DV01-based hedge ratio:**

Suppose the target and hedge are both primarily driven by the same rate factor and have:
- $\text{DV01}_{\text{target}} = 1500$ \$/bp
- $\text{DV01}_{\text{hedge}} = 1000$ \$/bp

A DV01-matching hedge for a single common factor would use:
$$h_{\text{DV01}} = -\frac{1500}{1000} = -1.5$$

matching the regression result.

**When do regression and DV01 ratios differ? (Intuition)**

- If the hedge instrument doesn't track the target's rate risk perfectly (different key-rate mix), the yield-change beta $b$ in the source regression model differs from 1, shifting the hedge ratio
- If P&L contains non-rate effects (spread, carry/roll, convexity, liquidity), regression absorbs those into $\beta$, while DV01 does not (DV01 is a rate-only local measure)

---

### Example H: PCA Factor Construction (Toy)

**Goal:** Provide a toy dataset of rate changes at 2y/5y/10y; compute covariance matrix and eigenvectors; interpret first 2–3 PCs as level/slope/curvature-like.

**Important Note:** Full eigen-calculation can be heavy in general, but we choose a toy dataset that makes the eigenvectors transparent and still demonstrates the PCA mechanics. This is an illustrative PCA example aligned with the sources' description that PCA extracts a small set of standard shifts (often level/slope/curvature) explaining most variance.

**Tenors:** 2y / 5y / 10y

**Data:** 6 observations of daily rate changes (bp)

Let $s = \sqrt{3/2} \approx 1.2247$, $t = 1/(2\sqrt{2}) \approx 0.3536$, $u = 1/\sqrt{2} \approx 0.7071$.

| Day | $\Delta y_{2y}$ (bp) | $\Delta y_{5y}$ (bp) | $\Delta y_{10y}$ (bp) |
|-----|----------------------|----------------------|-----------------------|
| 1 | 3 | 3 | 3 |
| 2 | -3 | -3 | -3 |
| 3 | $-s$ | 0 | $+s$ |
| 4 | $+s$ | 0 | $-s$ |
| 5 | $+t$ | $-u$ | $+t$ |
| 6 | $-t$ | $+u$ | $-t$ |

**Step 1: Mean**

The dataset is symmetric, so $\overline{\Delta y} = 0$ at each tenor.

**Step 2: Sample covariance $S$**

Using the sample covariance formula $S = \frac{1}{n} \sum (\Delta y_t)(\Delta y_t)^\top$ (since mean is 0), consistent with the source's PCA setup.

Compute entries $S_{ij} = \frac{1}{6} \sum_t \Delta y_{i,t} \Delta y_{j,t}$.

$S_{11} = \text{Var}(2y)$:
$$S_{11} = \frac{1}{6}(9 + 9 + s^2 + s^2 + t^2 + t^2)$$

where $s^2 = 3/2$ and $t^2 = 1/8$:
$$S_{11} = \frac{1}{6}\left(18 + 2 \cdot \frac{3}{2} + 2 \cdot \frac{1}{8}\right) = \frac{1}{6}(18 + 3 + 0.25) = \frac{21.25}{6} = 3.5416667$$

$S_{22} = \text{Var}(5y)$:
$$S_{22} = \frac{1}{6}(9 + 9 + 0 + 0 + u^2 + u^2)$$

with $u^2 = 1/2$:
$$S_{22} = \frac{1}{6}(18 + 1) = 3.1666667$$

$S_{33} = \text{Var}(10y) = S_{11} = 3.5416667$ by symmetry.

**Cross terms:**

$S_{12}$:
$$S_{12} = \frac{1}{6}(9 + 9 + 0 + 0 + t(-u) + (-t)(u)) = \frac{1}{6}(18 - 2tu)$$

Since $tu = \frac{1}{2\sqrt{2}} \cdot \frac{1}{\sqrt{2}} = \frac{1}{4}$:
$$S_{12} = \frac{1}{6}(18 - 0.5) = \frac{17.5}{6} = 2.9166667$$

$S_{23}$ is symmetric and equals $2.9166667$.

$S_{13}$:
$$S_{13} = \frac{1}{6}(9 + 9 + (-s)(s) + (s)(-s) + t^2 + t^2) = \frac{1}{6}(18 - 2s^2 + 2t^2)$$
$$= \frac{1}{6}(18 - 3 + 0.25) = \frac{15.25}{6} = 2.5416667$$

So the covariance matrix is:

$$S = \begin{pmatrix} 3.541667 & 2.916667 & 2.541667 \\ 2.916667 & 3.166667 & 2.916667 \\ 2.541667 & 2.916667 & 3.541667 \end{pmatrix} \; (\text{bp}^2)$$

**Step 3: Eigenvectors and eigenvalues**

We claim the following orthonormal vectors (classic "level/slope/curvature" shapes on 3 tenors):

$$v_1 = \frac{1}{\sqrt{3}}(1, 1, 1)^\top, \quad v_2 = \frac{1}{\sqrt{2}}(-1, 0, 1)^\top, \quad v_3 = \frac{1}{\sqrt{6}}(1, -2, 1)^\top$$

**Verify $S v_k = \lambda_k v_k$ by checking $S$ acting on the unnormalized vectors:**

For $(1, 1, 1)$: row sums are $9$, so:
$$S(1, 1, 1)^\top = (9, 9, 9)^\top \implies \lambda_1 = 9$$

For $(-1, 0, 1)$:
$$S(-1, 0, 1)^\top = (-1, 0, 1)^\top \implies \lambda_2 = 1$$

For $(1, -2, 1)$:
$$S(1, -2, 1)^\top = 0.25 \cdot (1, -2, 1)^\top \implies \lambda_3 = 0.25$$

Thus eigenvalues are:
$$\lambda_1 = 9, \quad \lambda_2 = 1, \quad \lambda_3 = 0.25$$

**Variance Explained:**

Total variance = trace:
$$\text{trace}(S) = 3.541667 + 3.166667 + 3.541667 = 10.25 = 9 + 1 + 0.25$$

consistent with $\sum \lambda_k = \text{trace}(\Sigma)$ property.

Percentages:
- PC1: $9/10.25 = 87.8\%$
- PC2: $1/10.25 = 9.76\%$
- PC3: $0.25/10.25 = 2.44\%$

**Interpretation:**
- **PC1 (all +):** Level
- **PC2 (short vs long):** Slope
- **PC3 (wings vs belly):** Curvature

This matches the sources' qualitative description that principal components align with parallel/slope/curvature shifts and that a few PCs explain most variation.

---

### Example I: Factor Hedge Using PCA Loadings

**Goal:** Use PCA loadings from Example H to:
1. Compute factor exposures of a KRDV01 vector
2. Choose hedge instruments and solve for notionals to neutralize the first 2–3 PCs
3. Validate with scenario shocks and show residual

**Note on Source Support:** The sources clearly support PCA as a way to summarize yield curve moves into a few standard shifts. Using PCA loadings explicitly to build a hedge system is a natural extension of the linear exposure framework (same linear algebra as key-rate hedging), but desk implementation details (window, scaling, mapping to tradable hedges) are convention-dependent. We proceed as a derived extension while keeping the mechanics fully transparent.

**Tenor grid:** 2y / 5y / 10y (same as Example H)

**PCA loadings (from Example H):**

$$v_1 = \frac{1}{\sqrt{3}}(1, 1, 1), \quad v_2 = \frac{1}{\sqrt{2}}(-1, 0, 1), \quad v_3 = \frac{1}{\sqrt{6}}(1, -2, 1)$$

**Target KRDV01 vector (reuse Example E target):**

$$d_0 = (800, 1400, 1100)^\top \; (\text{\$/bp})$$

**Step 1: Compute PCA factor exposures of the target**

Define factor exposure to PC $k$ as:
$$E_k = d_0^\top v_k \; (\text{\$/bp of PC score})$$

Compute:

**PC1:**
$$E_1 = \frac{800 + 1400 + 1100}{\sqrt{3}} = \frac{3300}{\sqrt{3}} = 1905.255$$

**PC2:**
$$E_2 = \frac{-800 + 0 + 1100}{\sqrt{2}} = \frac{300}{\sqrt{2}} = 212.132$$

**PC3:**
$$E_3 = \frac{800 - 2(1400) + 1100}{\sqrt{6}} = \frac{-900}{\sqrt{6}} = -367.423$$

So target factor exposure vector:
$$E_0 = (1905.255, \; 212.132, \; -367.423)^\top$$

**Step 2: Choose hedge instruments and compute their factor exposures**

Choose three hedges with key-rate DV01 concentrated at each tenor:

- Hedge 1 (2y): $d_1 = (1000, 0, 0)$
- Hedge 2 (5y): $d_2 = (0, 1500, 0)$
- Hedge 3 (10y): $d_3 = (0, 0, 2000)$

Compute $E_{k,j} = d_j^\top v_k$:

**For Hedge 1:**
$$E_{1,1} = \frac{1000}{\sqrt{3}} = 577.350, \quad E_{2,1} = \frac{-1000}{\sqrt{2}} = -707.107, \quad E_{3,1} = \frac{1000}{\sqrt{6}} = 408.248$$

**For Hedge 2:**
$$E_{1,2} = \frac{1500}{\sqrt{3}} = 866.025, \quad E_{2,2} = 0, \quad E_{3,2} = \frac{-3000}{\sqrt{6}} = -1224.745$$

**For Hedge 3:**
$$E_{1,3} = \frac{2000}{\sqrt{3}} = 1154.701, \quad E_{2,3} = \frac{2000}{\sqrt{2}} = 1414.214, \quad E_{3,3} = \frac{2000}{\sqrt{6}} = 816.497$$

**Step 3: Solve for hedge notionals to neutralize PCs 1–3**

Let notionals be $x_1, x_2, x_3$. We solve:

$$E_0 + \begin{pmatrix} E_{1,1} & E_{1,2} & E_{1,3} \\ E_{2,1} & E_{2,2} & E_{2,3} \\ E_{3,1} & E_{3,2} & E_{3,3} \end{pmatrix} \begin{pmatrix} x_1 \\ x_2 \\ x_3 \end{pmatrix} = 0$$

To simplify algebra, multiply each equation by the relevant $\sqrt{\cdot}$ denominator (equivalent scaling):

**PC2 equation ($\times \sqrt{2}$):**
$$300 - 1000 x_1 + 2000 x_3 = 0 \implies x_1 - 2x_3 = 0.3$$

**PC3 equation ($\times \sqrt{6}$):**
$$-900 + 1000 x_1 - 3000 x_2 + 2000 x_3 = 0$$

**PC1 equation ($\times \sqrt{3}$):**
$$3300 + 1000 x_1 + 1500 x_2 + 2000 x_3 = 0$$

This is the same linear system as Example E, so:

$$\boxed{(x_1, x_2, x_3) = \left( -0.8, \; -\frac{14}{15}, \; -0.55 \right)}$$

**Residual Factor Exposures:**

By construction, PC1/PC2/PC3 exposures are exactly 0 (within rounding). Therefore, under any curve shock that lies in the span of these three PCs (which is all of $\mathbb{R}^3$ here), linearized P&L is 0.

**Step 4: Validate with scenario shocks and show residual**

We validate using three scenario shocks (bp) on 2y/5y/10y:

- Parallel +10 bp: $\delta = (10, 10, 10)$
- Twist +10 / 0 / $-10$: $\delta = (10, 0, -10)$
- Orthogonalized butterfly +10 / $-20$ / +10: $\delta = (10, -20, 10)$

Compute residual key-rate DV01 vector (we already checked in Example E that it is exactly 0):

$$d_{\text{tot}} = d_0 + x_1 d_1 + x_2 d_2 + x_3 d_3 = (0, 0, 0)^\top$$

Therefore, for each scenario:
$$\Delta P \approx -d_{\text{tot}}^\top \delta = 0$$

**Residual Report:**

- Residual KRDV01 vector: $(0, 0, 0)$ \$/bp
- Residual factor exposures (PC1–PC3): $(0, 0, 0)$ \$/bp

**Practical Note:** In real data, hedges will not be exact due to:
- Imperfect "pure tenor" hedge instruments
- Estimation error in PCA loadings
- Nonlinearity/convexity for larger moves
- Multi-curve basis (discount vs projection)

---

## Practical Notes

### Common Pitfalls

**Confusing scenario hedges vs factor hedges:**
- **Scenario hedges:** neutralize PV under a chosen set of shocks
- **Factor hedges:** neutralize exposures to statistical factors (PCA) or regression betas
- A hedge can be scenario-neutral but still factor-exposed if the scenario set is incomplete

**Using PCA loadings from one regime to hedge another:**
PCA is data-driven; factor shapes can change in different volatility regimes or policy regimes. The sources emphasize the empirical nature of PCA factors (they arise from data).

**Mixing curve bump definitions:**
- Zero vs par vs discount-factor bump
- Linear-yield interpolation vs other methods
- Key-rate methods depend on the key-rate shift definition and interpolation; hedging quality depends on these assumptions

**Overfitting / unstable hedge ratios:**
- Too many hedges relative to data length (regression instability)
- Ill-conditioned regression matrices lead to unstable solutions; the regression source discusses ill-conditioning/near-singularity issues and the need for robustness/regularization in least-squares settings

**Ignoring convexity and spread/basis risk:**
- This chapter uses first-order rate risk
- Large moves, optionality, and spread changes can dominate realized P&L

### Implementation Pitfalls

**PCA details:**
- Choice of observation window (1y vs 5y)
- De-meaning and scaling (covariance vs correlation)
- Using yields vs zero rates vs forward rates inconsistently

**Regression details:**
- Serial correlation, heteroskedasticity, and roll effects
- Aligning P&L series (same timestamps, same carry conventions)

**Data hygiene:**
- Missing quotes, roll dates (futures), benchmark changes
- Cleaning outliers: can materially change PCA and regression estimates

### Verification Tests

**Scenario Test Suite:**
- Parallel shift (level)
- Twist (slope)
- Butterfly (curvature)
- Random perturbation (stress test)

**Stability Tests:**
- PCA loadings across rolling windows
- Regression betas under small data perturbations

**Unit Tests:**
- Scaling check: doubling notional doubles DV01 and scenario P&L
- Sign check: rate up should reduce PV for positive DV01

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **DV01 measures first-order PV sensitivity to a small parallel yield shift**; it is insufficient for curve-shape risk
2. **Curve-shape risk** can be organized into level, slope, curvature (parallel, twists, butterflies)
3. **Key-rate DV01** replaces one DV01 number with a vector across tenors, enabling more targeted hedges
4. A **DV01-neutral portfolio** can still have meaningful slope or curvature exposure (Example A)
5. **Scenario-based hedging** uses explicit designed shocks and checks P&L under each
6. **Hedging is a linear-algebra problem:** solve $Ex = -e_0$ (exact) or least squares (overdetermined)
7. **Regression hedging** estimates how yields (or P&Ls) co-move and uses the estimated slope to set hedge ratios
8. **PCA finds a small set of orthogonal curve shift shapes**; often the first few resemble level/slope/curvature and explain most variance
9. **Always validate hedges with multiple scenarios**, not just DV01
10. **Multi-curve setups** add dimensions (discount vs projection curve exposures); treat "DV01" as curve-specific and validate basis risk separately

---

### Cheat Sheet: Curve-Shape Factors, Hedge System Setup, Validation Checklist

**Core Formulas:**

**DV01 definition:**
$$\boxed{\text{DV01} \equiv -\frac{\Delta P}{10{,}000 \, \Delta y}}$$

**Scenario P&L:**
$$\boxed{\Delta P \approx -d^\top \delta}$$

**Factor exposures (level/slope/curvature on 2y/5y/10y):**
$$E_L = d_2 + d_5 + d_{10}, \quad E_S = d_2 - d_{10}, \quad E_C = d_2 - 2d_5 + d_{10}$$

**Exact hedge:**
$$\boxed{Ex = -e_0}$$

**Least squares hedge:**
$$\boxed{(E^\top E)x = -E^\top e_0}$$
(Same structure as OLS)

**Regression slope (one hedge, mean-zero data):**
$$\beta = \frac{\sum H_t T_t}{\sum H_t^2}, \quad h = -\beta$$

**Validation Checklist:**

- [ ] PV change $\approx 0$ under parallel shift
- [ ] PV change $\approx 0$ under twist/steepener
- [ ] PV change $\approx 0$ under butterfly
- [ ] Residual DV01/KRDV01 exposures small
- [ ] Hedge stable across small data/curve perturbations
- [ ] Units consistent (\$/bp × bp = \$)
- [ ] Document curve definition (par vs zero; interpolation; single vs multi-curve)

---

### Flashcards (30 Q/A)

**Q1:** What does DV01 measure?
**A:** First-order PV sensitivity to a 1 bp parallel yield shift (PV change for 1 bp fall in yields).

**Q2:** Why can DV01 hedges fail?
**A:** Because DV01 assumes yields shift in parallel; nonparallel moves create residual risk.

**Q3:** What is a twist?
**A:** A curve move where short and long yields change differently (often opposite directions).

**Q4:** What is a butterfly move?
**A:** A curvature change: the belly moves relative to wings.

**Q5:** What is KRDV01?
**A:** PV change for a 1 bp decrease in a specific key rate under a defined key-rate shift construction.

**Q6:** Why is a KRDV01 vector more informative than DV01?
**A:** It localizes sensitivity by tenor and can detect slope/curvature exposures.

**Q7:** Write the scenario P&L formula using KRDV01.
**A:** $\Delta P \approx -d^\top \delta$.

**Q8:** What does "DV01-neutral" mean?
**A:** Net PV sensitivity to a small parallel shift is approximately zero.

**Q9:** Can DV01-neutral portfolios have risk?
**A:** Yes—slope/curvature risk (nonparallel risk) can remain.

**Q10:** What is a designed shock?
**A:** A specified vector of rate changes at key tenors used for scenario tests.

**Q11:** What is level risk?
**A:** Exposure to parallel shifts.

**Q12:** What is slope risk?
**A:** Exposure to twists/steepeners/flatteners.

**Q13:** What is curvature risk?
**A:** Exposure to butterfly-type changes.

**Q14:** What is the advantage of scenario hedging?
**A:** No need for a statistical model; directly targets chosen risks.

**Q15:** What is the drawback of scenario hedging?
**A:** You might miss scenarios you didn't include.

**Q16:** What is regression hedging (in fixed income)?
**A:** Using least-squares regression of yield changes to estimate co-movements and set hedge ratios.

**Q17:** What does OLS minimize?
**A:** Sum of squared residuals (hedging error) in the regression relationship.

**Q18:** What is PCA used for in curve risk?
**A:** To find a small number of orthogonal curve shift patterns explaining most yield variation.

**Q19:** What do PCA eigenvectors represent?
**A:** Loading shapes across maturities; directions of maximum variance.

**Q20:** What do PCA eigenvalues represent?
**A:** Variances of the principal components.

**Q21:** How do you compute "variance explained" by first $k$ PCs?
**A:** $\frac{\sum_{j=1}^{k} \lambda_j}{\sum_{j=1}^{m} \lambda_j}$.

**Q22:** What is a factor exposure in PCA hedging?
**A:** Projection of DV01/KRDV01 vector onto a loading vector.

**Q23:** What is the linear system for an exact factor hedge?
**A:** $Ex = -e_0$.

**Q24:** When do you use least squares for hedging?
**A:** When there are more risk points than hedges (overdetermined system).

**Q25:** What are the normal equations for least squares?
**A:** $(E^\top E)x = -E^\top e_0$.

**Q26:** Why validate hedges with multiple scenarios?
**A:** Because neutralizing one risk measure doesn't guarantee neutrality to other curve shapes.

**Q27:** What is a key implementation risk for PCA?
**A:** Regime dependence (loadings change) and estimation noise.

**Q28:** What is a key implementation risk for regression hedges?
**A:** Ill-conditioned regressions leading to unstable hedge ratios.

**Q29:** What is a "multi-curve" issue for hedging?
**A:** Separate discount and projection curves create additional sensitivities and basis risk.

**Q30:** What is the most important practical habit for curve hedging?
**A:** Always scenario-test and monitor residual exposures.

---

## Mini Problem Set (16 Questions)

*(Brief solution sketches provided for questions 1–8 only)*

---

**Problem 1:** A bond position has DV01 = \$12,000/bp. What is the approximate P&L for a +7 bp parallel increase?

**Sketch:** $\Delta P \approx -\text{DV01} \cdot \delta = -12{,}000 \cdot 7 = -84{,}000$.

---

**Problem 2:** A portfolio is DV01-neutral but has KRDV01 vector $(+800, -800)$ across (2y, 10y). Compute P&L for twist $(+5, -5)$ bp.

**Sketch:** $\Delta P \approx -(800 \cdot 5 + (-800) \cdot (-5)) = -(4000 + 4000) = -8000$.

---

**Problem 3:** Given KRDV01 vector across (2y, 5y, 10y) of $(300, 600, 300)$, compute P&L for butterfly shock $(0, +10, 0)$.

**Sketch:** $\Delta P \approx -(300 \cdot 0 + 600 \cdot 10 + 300 \cdot 0) = -6000$.

---

**Problem 4:** Define level and slope factors on (2y, 10y) as $(1, 1)$ and $(1, -1)$. Show that hedging both factors is equivalent to hedging both key rates.

**Sketch:** In 2D, key-rate basis $\{(1,0), (0,1)\}$ and factor basis $\{(1,1), (1,-1)\}$ span the same space; mapping is invertible (linear combinations).

---

**Problem 5:** Solve for hedge notionals to neutralize level and slope for target $(d_{2y}, d_{10y}) = (1000, 1500)$ using hedges $(800, 0)$ and $(0, 2000)$.

**Sketch:** Solve:
$$\begin{cases} 1000 + 800 x_1 = 0 \\ 1500 + 2000 x_2 = 0 \end{cases} \implies x_1 = -1.25, \; x_2 = -0.75$$
(Equivalently factor-neutral since both key rates are neutral.)

---

**Problem 6:** For a 3-point grid (2y/5y/10y), compute level/slope/curvature exposures for $d = (500, 1000, 800)$.

**Sketch:** Level: $500 + 1000 + 800 = 2300$. Slope: $500 - 800 = -300$. Curvature: $500 - 2(1000) + 800 = -700$.

---

**Problem 7:** In a one-hedge regression with mean-zero data, you observe $\sum H_t^2 = 50$ and $\sum H_t T_t = 60$. What is the hedge ratio $h$?

**Sketch:** $\beta = 60/50 = 1.2$, hedge ratio $h = -1.2$.

---

**Problem 8:** A PCA on 3 tenors yields eigenvalues $(9, 1, 0.25)$. What percent of variance is explained by first two PCs?

**Sketch:** Total variance $= 9 + 1 + 0.25 = 10.25$. First two $= 10$. So $10/10.25 = 97.56\%$.

---

**Problem 9:** Explain why PCA-based hedges can fail in a regime change.

---

**Problem 10:** Discuss how multi-curve discounting changes the interpretation of "DV01."

---

**Problem 11:** Describe a scenario set you would use to validate a hedge for a 5y swap.

---

**Problem 12:** Compare "key-rate DV01 hedging" with "bucket DV01 hedging" conceptually.

---

**Problem 13:** A least-squares hedge yields small residuals in KRDV01 but still large P&L in stress. List plausible reasons.

---

**Problem 14:** If you hedge level and slope but not curvature, which trade types remain risky?

---

**Problem 15:** Explain the difference between hedging using yields vs discount factors.

---

**Problem 16:** Describe how you would choose the tenor grid for KRDV01 reporting.

---

## Source Map

### (A) Verified Facts — Specific Source Citations

| Content | Source |
|---------|--------|
| DV01 definition, first-order P&L approximation | Tuckman Ch 5-6 |
| DV01 hedging assumes parallel shift (explicit limitation) | Tuckman |
| Key-rate DV01 / bucket exposures defined by shifting specific segments | Tuckman Ch 6 |
| Hedging quality depends on assumed behavior between key rates | Tuckman |
| PCA shows yield curve shifts are largely a linear sum of 2-3 standard shifts | Risk management source |
| PCA first PC resembles parallel shift, second resembles slope | Fixed income source example |
| Regression-based hedging estimates co-movement and minimizes hedging error | Tuckman |
| Least-squares regression uses normal equations / matrix inversion | Standard source |

### (B) Reasoned Inference — Derivation Logic

| Content | Derivation |
|---------|------------|
| Scenario P&L formula $\Delta P \approx -d^\top \delta$ | Linear combination of KRDV01s under linearization |
| Level/slope/curvature factor exposures as linear functionals | Dot product of KRDV01 vector with factor shock vectors |
| Hedge construction as linear system | Direct from exposure neutralization condition |
| Regression hedge ratio $h = -\beta \cdot \text{DV01}_A / \text{DV01}_B$ | Derived from variance minimization and source's regression setup |
| PCA factor exposures as dot products with eigenvector loadings | Standard PCA projection |

### (C) Speculation — Flagged Uncertainties

| Content | Flag |
|---------|------|
| Toy covariance matrices and eigenvalues in PCA examples | Illustrative numbers |
| Desk PCA implementation details (window, scaling, mapping) | Convention-dependent |

---

*Last Updated: January 2026*
