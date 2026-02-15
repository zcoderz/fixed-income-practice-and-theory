# Chapter 22: Curve Risk Management in a Multi-Curve World — Par-Point Deltas, Jacobians, and Controlled Perturbations

---

## Introduction

You bump the 5-year swap rate by 1 basis point. Which discount factors move? By how much? Does the change stay localized around year five, or does it ripple across the entire curve—perhaps even causing the 20-year forward rate to jump by 30 basis points in the opposite direction?

These questions lie at the heart of curve risk management. Chapters 11–14 introduced the *what* of interest rate risk: DV01, duration, key-rate exposures, and bucket sensitivities. Chapter 17 addressed the *how* of curve construction: bootstrapping, interpolation, and the underdetermined nature of fitting a continuous function to discrete market quotes. This chapter bridges these two domains by asking: *how do the choices we made in curve construction affect the risk numbers we compute?*

In practice, the answer is: *a lot*. Two curve builders can reprice the same benchmark instruments and still produce different deltas, because they distribute a quote bump differently across the curve. Some methods keep perturbations local; others propagate (or even oscillate) the shock into distant tenors. Those choices show up directly in hedges and in whether “risk explains P&L.”

Prerequisites: [Chapter 11 — DV01/PV01 — Definitions, Computation, and “What’s Being Bumped”](chapters/chapter_11_dv01_pv01_definitions_computation.md); [Chapter 14 — Key-Rate DV01 and Bucket Exposures](chapters/chapter_14_key_rate_dv01_bucket_exposures.md); [Chapter 17 — Curve Construction — Bootstrapping, Interpolation, and the Spline Zoo](chapters/chapter_17_curve_construction_bootstrapping_interpolation.md); [Chapter 18 — OIS Discounting Curve](chapters/chapter_18_ois_discounting_curve.md); [Chapter 19 — Projection Curves (LIBOR/SOFR) and Multi-Curve](chapters/chapter_19_projection_curves_libor_sofr_multi_curve.md); [Chapter 20 — Tenor Basis](chapters/chapter_20_tenor_basis.md); [Chapter 21 — Cross-Currency Curves](chapters/chapter_21_cross_currency_curves.md).  
Follow-on: [Chapter 26 — Swap PV01, DV01, and Hedging with Swaps](chapters/chapter_26_swap_pv01_dv01_hedging.md); [Chapter 33 — Collateral Discounting (OIS)](chapters/chapter_33_collateral_discounting_ois.md).

## Learning Objectives
- Translate **quote moves → curve-node moves → PV moves** using the chain rule and a curve-builder Jacobian.
- Define and compute **par-point / quote DV01** with explicit **bump object**, **bump size**, **units**, and **sign**.
- Explain (and diagnose) **locality vs. non-locality** of curve perturbations under different curve construction choices.
- Use **controlled perturbations** (forward-curve basis functions) and relate them to key-rate/bucket risk.
- Decompose multi-curve risk into **level vs discounting vs basis** components and connect the result to P&L explain/predict.

**Roadmap:** This chapter develops the tools to understand and control these effects:

- **Section 22.1** establishes the fundamental chain rule connecting quote changes to PV changes through the curve builder—the mathematical backbone of "par-point" or "quote" deltas.

- **Section 22.2** examines what happens when you bump a single quote and rebuild: the par-point approach, locality under perturbation, and the infamous see-saw effect that afflicts certain interpolation methods.

- **Section 22.3** introduces the **Jacobian method**, which decouples risk calculation from curve construction by explicitly computing how curve nodes respond to quote changes. We also cover **algorithmic differentiation**, which computes many sensitivities efficiently by applying the chain rule to valuation code.

- **Section 22.4** develops **controlled perturbations**: an alternative approach where we shock the forward curve directly with economically meaningful shapes, avoiding the artifacts of bump-and-rebuild.

- **Section 22.5** addresses multi-curve risk decomposition: in a world of OIS discounting and tenor basis, how do we separate "overall level" risk from "discounting" risk from "basis" risk? We also cover **risk limits and bucket monitoring**—the organizational framework for using these sensitivities.

- **Section 22.6** presents P&L predict and P&L explain methodologies—the ultimate validation of any risk system—including a diagnostic checklist for fixing broken P&L predict.

- **Section 22.7** covers practical implementation considerations: numerical stability, curve overlays, computational efficiency, and when to refresh your Jacobians.

Throughout, we emphasize a key desideratum: **P&L predict**. The deltas you compute should explain the P&L you observe when quotes move. If they don't, something in your curve build or risk methodology is broken.

---

## 22.1 The Chain Rule: From Quotes to PV

### 22.1.1 Portfolio Value as a Function of Curve Nodes

Consider a portfolio with present value $V_0$. This value depends on the discount factors (or equivalently, zero rates or forward rates) at various maturities. We can write:

$$V_0 = V_0(x_1, x_2, \ldots, x_M)$$

where $x = (x_1, \ldots, x_M)$ represents the curve's internal parameterization—perhaps discount factors at key dates, zero rates at node points, or spline coefficients.

For simple fixed-income instruments, this dependence is often straightforward. A bond paying cash flows $c_j$ at times $t_j$ has:

$$V_0 = \sum_{j=1}^{n} c_j   P(t_j)$$

where $P(t_j)$ is the discount factor to time $t_j$. More complex instruments—swaptions, callables, exotics—may have nonlinear dependencies on the curve, but the fundamental structure remains: PV is a function of the curve.

### 22.1.2 The Curve Builder as a Function

The curve itself is not given to us directly; it must be constructed from market quotes. Let $q = (q_1, \ldots, q_N)$ denote the vector of benchmark quotes: deposit rates, futures prices, swap rates, basis spreads, and so forth. The curve construction algorithm is a function:

$$x = B(q)$$

that maps quotes to curve parameters. This function embodies all the choices we discussed in Chapter 17: which instruments to include, how to order the bootstrap, which interpolation method to use.

Combining these, we have:

$$V_0(q) = V_0(B(q))$$

The portfolio value is ultimately a function of market quotes, mediated by the curve builder.

### 22.1.3 The Fundamental Chain Rule

What happens to $V_0$ when quote $q_i$ moves by a small amount $\Delta q_i$? By the chain rule:

$$\boxed{\frac{dV_0}{dq_i} = \sum_{j=1}^{M} \frac{\partial V_0}{\partial x_j} \cdot \frac{\partial x_j}{\partial q_i}}$$

In matrix notation, if we define:
- $\nabla_x V_0 = \left(\frac{\partial V_0}{\partial x_1}, \ldots, \frac{\partial V_0}{\partial x_M}\right)$ as the **node sensitivity** vector
- $J = \frac{\partial x}{\partial q}$ as the **Jacobian matrix** of the curve builder, with $(j,i)$ entry $\frac{\partial x_j}{\partial q_i}$

then:

$$\boxed{\frac{dV_0}{dq} = \nabla_x V_0 \cdot J}$$

This equation is the mathematical backbone of curve risk management. It says that quote risk decomposes into two pieces:
1. **Node sensitivity**: How does PV respond to changes in the curve parameterization? This is often analytically tractable.
2. **Jacobian**: How does the curve parameterization respond to changes in quotes? This depends entirely on the curve construction algorithm.

This separation is useful: the instrument/portfolio logic determines the node sensitivities $\nabla_x V_0$, while the curve-building methodology determines the Jacobian $J$. Changing the curve build changes $J$, and therefore changes the quote deltas $\frac{dV_0}{dq}$.

**Check (dimensions + bp conversion):** The object $\frac{dV_0}{dq_i}$ has units “currency per 1 unit of the quote,” where “1 unit” means whatever units your system stores $q_i$ in. If quotes are stored as *decimal rates* (e.g., 5% = 0.05), then \$1\text{bp}=10^{-4}$ and the book’s down-bump definition is, to first order,
$$DV01_{q_i}=PV(q_i-1\text{bp})-PV(q)\quad \approx\quad -\frac{dV_0}{dq_i}\times 10^{-4}.$$
If quotes are stored in bp or percent points instead, the numeric bump size changes; many “10,000×” DV01 bugs are just unit mismatches between $q$, the bump, and the reported DV01 units.

### 22.1.4 Conventions: Bumps, Units, and Sign

Before you compare risk numbers across systems, lock four knobs:

- **Bump object:** a *quote delta* means bumping a single market quote $q_i$ (par swap rate, futures price, basis spread, …), rebuilding the curve(s) with all other quotes held fixed, and repricing.
- **Bump size:** for rate-like quotes, use \$1\text{ bp} = 10^{-4}$ in decimal rate units (and be explicit if you use a different size).
- **Units:** report quote DV01s in **currency per 1bp of the quote** (or per tick for price-like quotes).
- **Sign convention (book-wide):** define
  $$
    DV01_{q_i} := PV(q_i-1\text{bp})-PV(\text{base}).
  $$
  Positive means the position gains when the relevant quote is shifted **down** by 1bp.

> **Pitfall — What is being bumped?:** Mixing “quote DV01” (bump $q_i$ and rebuild) with “node DV01” (bump a curve node $x_j$ holding the builder fixed) and calling both “DV01.”
> **Why it matters:** hedge ratios and P&L predict depend on the *mapping* from quote space to curve space; two systems can agree on PV and still disagree on DV01 if they bump different objects or rebuild differently.
> **Quick check:** confirm units (currency per 1bp of $q_i$) and that $DV01_{q_i}$ from the chain rule matches bump-and-rebuild within a small tolerance (and has the down-bump sign).

Numerical note: central differences (bump up and bump down) typically improve stability; you can compute a symmetric derivative and still report DV01 using the down-bump definition above.

### 22.1.5 Worked Example: Quote DV01 via a Bootstrapped Jacobian (Toy 3‑Pillar Curve)

**Example Title**: From the 3Y par swap quote to a bond PV and quote DV01

**Context**
- You want a risk number that maps directly to a hedge instrument quote (a par swap rate pillar).
- You also want to see how the curve builder turns that quote bump into a discount-factor bump.

**Timeline (Concrete)**
- Trade date: 2026-02-15
- Settlement date: 2026-02-17 (assume T+2; toy example)
- Payment dates: 2027-02-17, 2028-02-17, 2029-02-17

**Inputs**
- Instrument: 3Y fixed-rate bond, annual coupon 5%, face $F=\$100{,}000{,}000$.
- Market quotes (toy): annual-pay par swap rates $S_1=4.00\%$, $S_2=4.20\%$, $S_3=4.30\%$.
- Assumptions (toy): single-curve world; annual accrual factors $=1$; rates are decimals in formulas.

**Outputs (What You Produce)**
- Discount factors $P(0,1)$, $P(0,2)$, $P(0,3)$.
- Bond price per 100 face and quote DV01 to the 3Y swap quote $S_3$ (USD per 1bp).

**Step-by-step**
1. **Bootstrap discount factors from par swap quotes** (annual, unit accrual):
   $$
   S_n\sum_{i=1}^n P(0,i)=1-P(0,n).
   $$
   This yields:
   $$
   P(0,1)=\frac{1}{1+S_1},\quad
   P(0,2)=\frac{1-S_2P(0,1)}{1+S_2},\quad
   P(0,3)=\frac{1-S_3(P(0,1)+P(0,2))}{1+S_3}.
   $$
   Numerically:
   $$
   P(0,1)\approx 0.961538,\quad 
   P(0,2)\approx 0.920936,\quad 
   P(0,3)\approx 0.881164.
   $$
2. **Price the bond (per 100 face)**:
   $$
   PV_{100}=5 \cdot P(0,1)+5 \cdot P(0,2)+105 \cdot P(0,3)\approx 101.935.
   $$
3. **Compute quote DV01 to the 3Y swap quote** using bump-and-rebuild:
   $$
   DV01_{S_3}=PV_{100}(S_3-1\text{bp})-PV_{100}(S_3)\approx 0.0278\quad \text{per 100}.
   $$
   Convert to dollars:
   $$
   DV01_{S_3,\$} \approx \frac{0.0278}{100}\times \$100{,}000{,}000 \approx \$27{,}800\quad \text{per 1bp}.
   $$
   **Jacobian intuition (locality in this bootstrap):** here $S_3$ only moves $P(0,3)$, so $DV01_{S_3,100}\approx 105\cdot \Delta P(0,3)$.

**Cashflows**
| Date | Cashflow | Explanation |
|---|---|---|
| 2027-02-17 | \$5,000,000 | coupon $=0.05\times F$ |
| 2028-02-17 | \$5,000,000 | coupon |
| 2029-02-17 | \$105,000,000 | coupon + principal |

**P&L / Risk Interpretation**
- If the 3Y par swap quote moves down 1bp and you rebuild the curve using this methodology, your bond PV should rise by about \$27.8k (first-order).
- If your risk report shows large exposure to distant pillars (e.g., 10Y) for a 3Y cashflow instrument, that is a red flag for non-local curve responses (Section 22.2).

**Sanity Checks**
- Units: $PV_{100}$ is “price per 100 face”; converting to dollars multiplies by $F/100$.
- Sign: $DV01_{S_3}>0$ for a long fixed-rate bond under the down-bump convention.
- Reproduction: every step can be replicated in a spreadsheet (bootstrap → PV → bump-and-rebuild DV01).

### 22.1.6 Hedging Interpretation

Quote deltas are attractive because they line up with tradable benchmark instruments. A first-order hedge for quote $q_i$ is “take the opposite quote DV01” using instrument $i$—but always remember: this hedges moves **as defined by your curve builder**. If the curve builder produces non-local or unstable responses to bumps, the hedge ratios will be unstable too.

---

## 22.2 Par-Point Deltas and the Locality Problem

### 22.2.1 The Par-Point Approach: Bump and Rebuild

The simplest way to compute quote risk is direct numerical differentiation: bump one quote, rebuild the curve, reprice the portfolio, and report the change. This procedure is sometimes called the **par-point approach**, and the resulting sensitivities are **par-point deltas**.

**Algorithm (Par-Point Delta for Quote $q_i$):**

1. Record base portfolio value: $V_0^{\text{base}}$
2. Bump quote $q_i$ by $\delta$ (typically 1 bp = \$10^{-4}$): $q_i \to q_i + \delta$
3. Rebuild the entire curve: $x^{\text{bumped}} = B(q_1, \ldots, q_i + \delta, \ldots, q_N)$
4. Reprice the portfolio: $V_0^{\text{bumped}}$
5. Compute the delta: $\frac{\partial V_0}{\partial q_i} \approx \frac{V_0^{\text{bumped}} - V_0^{\text{base}}}{\delta}$
6. Repeat for each benchmark quote.

The appeal is simplicity and generality: it works for any portfolio and any curve construction algorithm. The risk output directly corresponds to hedging instruments—a delta to the 5-year swap quote tells you how much 5-year swap notional to trade to hedge that exposure.

### 22.2.2 Locality Under Perturbation

For bump-and-rebuild deltas to be economically interpretable, the curve construction should respond to a *local* quote bump with a reasonably *local* perturbation of the curve. If a 1M instrument bump causes large movements in 20Y forwards, a risk report can falsely suggest that short instruments hedge long risk.

**Locality** means: when you bump a short-dated benchmark, the changes in rates/forwards at much longer maturities should be small.

Different interpolation methods have very different locality properties. A common trade-off is:

| Method | Locality | Forward Curve Smoothness |
|--------|----------|-------------------------|
| Piecewise-linear yields (bootstrap) | Good | Poor (discontinuous forwards) |
| Hermite $C^1$ splines | Good | Moderate |
| $C^2$ cubic splines (no tension) | Poor | Excellent |
| $C^2$ splines with tension | Moderate | Good |

The trade-off is fundamental: methods that produce smooth, continuous forward curves often achieve this by spreading information globally, destroying locality. Methods with good locality often produce jagged or discontinuous forward curves.

### 22.2.3 The See-Saw Effect

> **Analogy: The Pin and the Rope**
>
> Imagine the yield curve is a loose rope laid on a table.
> *   **The Quotes**: You hammer pins into the table at specific points (2Y, 3Y, 5Y) to hold the rope in place.
> *   **The Bump**: Now, grab the 20Y pin and yank it up by 1 inch, while keeping the 19Y and 30Y pins stuck tight to the table.
> *   **The Result**: The rope between 19Y and 20Y must stretch violently upward. The rope between 20Y and 30Y must dive violently downward to get back to the 30Y pin.
>
> This violent up-and-down shape is the "See-Saw." It's not created by the market; it's created by the tension of your interpolation method fighting against your fixed pins.

When locality fails, a 1bp quote bump can cause a noisy, ringing perturbation that spreads into short- and long-dated parts of the forward curve. For example, bumping the 2Y swap rate by 1 bp while holding 1Y and 3Y fixed can look fairly localized under bootstrap-style methods, but a globally coupled spline fit can spread the perturbation across short and long maturities.

The mechanics can be seen from a rough approximation: swap rates are approximately linear combinations of forward rates:

$$S_n \approx \sum_{i=1}^{n} w_{i,n} L_i$$

where $S_n$ is the $n$-year swap rate, $L_i$ is the $i$-year forward rate, and $w_{i,n} \approx 1/n$ for annual swaps. Inverting this relationship:

$$\boxed{L_n \approx n \cdot S_n - (n-1) \cdot S_{n-1}}$$

This innocent-looking formula has dramatic implications. As part of a par-point report, assume $S_n$ is shifted by the amount $\delta$ while $S_{n-1}$ and $S_{n+1}$ remain unchanged. Then:
- $L_n$ shifts by approximately $n \cdot \delta$
- $L_{n+1}$ shifts by approximately $-n \cdot \delta$

This is the **see-saw effect**: at the long end, a 1 bp bump to a single par swap quote (with neighbors held fixed) can imply *tens of bp* swings in adjacent annual forward rates. For products that are sensitive to forward *spreads* (or have nonlinear payoffs), these artificially large local forward moves can push you outside the regime where first-order deltas are a good approximation.

> **Practical Warning:** “Hold everything fixed except one tenor” is a convenient numerical experiment, but it is not how markets typically move. If you want shocks that look more like realized rate moves, consider key-rate style shocks or controlled perturbations (Section 22.4).

### 22.2.4 Worked Example: Quantifying the See-Saw

**Setup:** Consider a 30-year swap rate $S_{30} = 4.50\%$ with 29-year and 31-year rates at $S_{29} = 4.48\%$ and $S_{31} = 4.52\%$.

**Base case forward rates** (using $L_n \approx nS_n - (n-1)S_{n-1}$):
- $L_{30} \approx 30 \times 4.50\% - 29 \times 4.48\% = 135.00\% - 129.92\% = 5.08\%$
- $L_{31} \approx 31 \times 4.52\% - 30 \times 4.50\% = 140.12\% - 135.00\% = 5.12\%$

**Perturbed case:** Bump $S_{30}$ to 4.51% (1 bp), holding others fixed:
- $L_{30}^{\text{new}} \approx 30 \times 4.51\% - 29 \times 4.48\% = 135.30\% - 129.92\% = 5.38\%$
- $L_{31}^{\text{new}} \approx 31 \times 4.52\% - 30 \times 4.51\% = 140.12\% - 135.30\% = 4.82\%$

**Changes:**
- $\Delta L_{30} = 5.38\% - 5.08\% = +30$ bp
- $\Delta L_{31} = 4.82\% - 5.12\% = -30$ bp

A 1 bp swap rate move produces a 60 bp swing in the 30Y/31Y forward spread in this stylized “single-tenor bump” setup.

### 22.2.5 Comparing Interpolation Methods

**Setup:** Build a curve from par swap quotes at 1Y, 2Y, 3Y, 5Y, 7Y, 10Y, 15Y, 20Y, 30Y. Consider a 7Y receiver swap.

**Diagnostic experiment:** Bump the 5Y swap quote by 1 bp (hold other quotes fixed), rebuild, and look at:
1. The forward-curve response $\Delta f(t)$ (does it stay local, or ring/oscillate?).
2. The portfolio’s quote DV01 vector across pillars (is the risk concentrated near 5Y/7Y, or spread into distant tenors?).

**What “good locality” looks like:** $\Delta f(t)$ and the quote-DV01 vector are mostly concentrated in the neighborhood of the bumped tenor (and the instrument’s cashflow window).

**What “poor locality” looks like:** $\Delta f(t)$ oscillates and you see non-trivial deltas to distant pillars. Those “phantom” deltas are not new economic exposures; they are artifacts of how your curve-fit enforces global smoothness.

### 22.2.6 Tension Splines: A Partial Remedy

One way to improve locality while retaining smoothness is to add **tension** to the spline.

Tension essentially penalizes curvature, pulling the spline toward piecewise-linear behavior in regions away from data points. Higher tension improves locality at the cost of forward curve smoothness.

Intuitively, tension damps the “ringing” that can appear when a globally smooth curve is forced through many pinned quotes.

---

## 22.3 The Jacobian Method and Algorithmic Differentiation

### 22.3.1 Motivation: Decoupling Risk from Curve Construction

The par-point approach requires a full curve rebuild for each bumped quote. For a curve with $N$ benchmark instruments, this means $N$ rebuilds per risk calculation—plus additional rebuilds for two-sided differences, scenario analysis, and multiple curves.

The Jacobian method offers an alternative: compute the relationship between quotes and curve nodes once, then use matrix multiplication for risk.

### 22.3.2 Computing the Jacobian

The Jacobian $J = \frac{\partial x}{\partial q}$ is an $M \times N$ matrix where:
- $M$ is the number of curve nodes (discount factors, zero rates, or spline coefficients)
- $N$ is the number of benchmark quotes
- Entry $J_{j,i} = \frac{\partial x_j}{\partial q_i}$

**Finite Difference Approximation:**

For column $i$ (sensitivity to quote $q_i$):

1. Bump $q_i$ up: $q^+ = (q_1, \ldots, q_i + \delta, \ldots, q_N)$
2. Rebuild curve: $x^+ = B(q^+)$
3. Bump $q_i$ down: $q^- = (q_1, \ldots, q_i - \delta, \ldots, q_N)$
4. Rebuild curve: $x^- = B(q^-)$
5. Central difference: $J_{\cdot,i} \approx \frac{x^+ - x^-}{2\delta}$

Using central differences (averaging up and down bumps) improves numerical stability, especially when PV depends nonlinearly on rates.

### 22.3.3 Using the Jacobian for Risk

Once we have $J$, computing quote risk becomes:

1. **Compute node sensitivities** $\nabla_x V_0$ analytically or semi-analytically
2. **Multiply by Jacobian**: $\frac{dV_0}{dq} = \nabla_x V_0 \cdot J$

For many portfolios, node sensitivities are simple:
- A zero-coupon bond maturing at $t_j$: $\frac{\partial V_0}{\partial P(t_j)} = 1$ (for unit notional)
- A coupon bond: $\frac{\partial V_0}{\partial P(t_j)} = c_j$ (the cash flow at $t_j$)
- A swap: sum of fixed and floating leg sensitivities

The Jacobian captures all the complexity of the curve construction, while node sensitivities capture the instrument's cash flow structure.

### 22.3.4 Advantages of the Jacobian Method

**Computational Efficiency:** For large portfolios, compute $\nabla_x V_0$ for each trade, then multiply by the (shared) Jacobian. This is much faster than rebuilding the curve for each quote bump for each trade.

**Curve Consistency:** In fast markets you often want “live-ish” risk without rebuilding curves continuously. A cached Jacobian lets you translate quote changes into approximate node changes via matrix multiplication, and you trigger a full rebuild (and a Jacobian refresh) only when quotes have moved far enough that the linear approximation is no longer adequate.

**Risk Attribution:** The Jacobian explicitly shows how quote bumps propagate to curve nodes. A column of zeros indicates that a quote doesn't affect certain tenors—useful for debugging and intuition.

### 22.3.5 Jacobian Structure and Dimensions

Consider a curve built from $N = 15$ benchmark instruments (deposits, futures, swaps) with $M = 100$ monthly forward rate nodes.

The Jacobian $J$ is \$100 \times 15$. Each column shows how a 1 bp bump to one quote moves all 100 forward rates.

**Check (how to use $J$ consistently):** Once you fix a quote-space shock $\Delta q$ (e.g., a -1bp move means $\Delta q_i=-10^{-4}$ for a decimal rate quote), the implied node move is $\Delta x \approx J \cdot \Delta q$. Quote DV01s are then the dot product $\nabla_x V_0\cdot \Delta x$ expressed per bp. Writing the shock explicitly is a reliable way to keep signs and 1bp scaling straight.

**For bootstrapping:** The Jacobian has a block-triangular structure. Bumping a short-dated instrument only affects forward rates up to its maturity.

**For global methods (cubic splines):** The Jacobian may have non-zeros everywhere, reflecting the global coupling inherent in the spline fit.

> **Visual: The Jacobian Heatmap**
>
> Imagine a grid (heatmap).
> *   **Rows**: Forward Rates (1M, 2M, ..., 30Y).
> *   **Columns**: Market Quotes (2Y Swap, 5Y Swap...).
> *   **Ideally**: You want a "Block Diagonal" heatmap. Bumping the 5Y quote should only light up the forward rates near 5Y.
> *   **The Reality**: In a bad curve build (e.g., untensioned spline), the heatmap looks like a checkerboard. Bumping the 2Y quote lights up the 30Y forward rate. This is "Pollution." It means your short-term trades are accidentally hedging your long-term risk.

### 22.3.6 Worked Example: A Small Numerical Jacobian

**Setup:** A minimal curve with 3 quotes (1Y, 2Y, 3Y deposit rates) and 3 nodes (1Y, 2Y, 3Y zero rates). Under piecewise-constant interpolation, each deposit pins exactly one zero rate.

**Quotes:** $q = (4.00\%, 4.50\%, 5.00\%)$

**Base zero rates:** $x = (4.00\%, 4.50\%, 5.00\%)$ (deposits bootstrap directly to zero rates)

**Jacobian (under this interpolation):**

$$J = \begin{pmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 1 \end{pmatrix}$$

This is the identity matrix because each quote determines exactly one node. Bumping the 2Y deposit by 1 bp changes only the 2Y zero rate by 1 bp.

**With cubic spline interpolation**, the Jacobian would typically have non-zero off-diagonal entries (illustrative numbers):

$$J \approx \begin{pmatrix} 0.95 & 0.04 & 0.01 \\ 0.08 & 0.85 & 0.07 \\ 0.02 & 0.12 & 0.86 \end{pmatrix}$$

Now bumping the 2Y deposit moves all three zero rates. The off-diagonal "pollution" is visible in the heatmap.

### 22.3.7 Relationship to Key Rate DV01

The Jacobian method is closely related to key-rate/bucket risk (Chapter 14). Key-rate DV01 can be viewed as: choose a *basis* of curve shocks (often triangular perturbations to par yields spanning between adjacent key maturities), then measure PV sensitivity to each basis shock.

A common design goal is **additivity**: the key-rate shocks sum to a parallel shift, so key-rate DV01s aggregate cleanly to total DV01. In Jacobian language, this is “just” a choice of shock basis (a choice of $\mu_k$), combined with a specific rule for how shocks are implemented in the curve builder.

### 22.3.8 Algorithmic Differentiation: Computing All Greeks in One Pass

The methods above—bump-and-rebuild and Jacobian caching—share a fundamental limitation: their computational cost scales with the number of risk factors. For a curve with $N$ quote sensitivities, you need at least $N$ curve rebuilds (or \$2N$ for central differences). For a trading desk with hundreds of curve nodes across multiple curves, this becomes expensive.

**Algorithmic differentiation (AD)** computes derivatives of the valuation code by applying the chain rule to the program’s operations. In **reverse mode**, after the valuation is complete (the “forward sweep”), the partial derivatives are recovered by performing a **reverse sweep** over the computation graph. For a scalar output (a single PV), reverse mode produces the whole gradient in one reverse sweep. The cost is a constant-factor overhead relative to one valuation, with important memory/engineering trade-offs because the computation graph must be stored (or partially recomputed).

**The Key Insight: Forward vs. Adjoint Mode**

Any computer program that computes $V = f(q_1, \ldots, q_N)$ can be viewed as a composition of elementary operations. The chain rule applies at each step. There are two ways to propagate derivatives:

**Forward (tangent) mode:** Propagate $\frac{\partial}{\partial q_i}$ forward through the computation. Cost: one forward pass per input variable. Total cost for $N$ inputs: $O(N)$ times valuation cost.

**Adjoint (reverse) mode:** Propagate sensitivities backward from the output. Define an **adjoint** $\bar{x}_i := \frac{\partial V}{\partial x_i}$ for each intermediate variable $x_i$. Initialize all adjoints to zero except at the output node, where the adjoint is set to 1, and propagate backward using the chain rule:

$$\boxed{\bar{x}_i = \sum_{j \text{ child of } i} \bar{x}_j \frac{\partial x_j}{\partial x_i}}$$

Cost: one backward pass for all $N$ sensitivities to the chosen inputs. The big win is the **scaling**: for a scalar PV output, reverse mode gives the whole gradient in one sweep, rather than “one sweep per input.”

**Back-of-the-envelope scaling (for a scalar PV output):**
- Finite differences: $O(N)$ valuations (often \$2N$ for central differences), plus curve rebuild overhead.
- Cached Jacobian: $O(N)$ curve rebuilds to refresh $J$ occasionally, then cheap matrix multiplies between refreshes.
- Reverse-mode AD: one reverse sweep per valuation graph to obtain sensitivities to many inputs (constant-factor overhead; memory-dependent). For scalar functions $f:\mathbb{R}^n\to\mathbb{R}$, the extra arithmetic associated with the gradient computation is often at most four or five times the arithmetic needed to evaluate the function alone (implementation-dependent).

**Implementation Requirements:**

AD requires **code instrumentation**. Two common approaches are:
1. **Operator overloading / “taping”:** run the valuation with special numeric types that record intermediate operations, then run a backward sweep.
2. **Source transformation:** generate adjoint code from the original valuation code.

**When to Use AD vs. Bump:**

| Scenario | Recommended Method |
|----------|-------------------|
| Few trades, simple instruments | Bump-and-rebuild (simplicity) |
| Large portfolio, many curves | AD or cached Jacobians |
| Non-smooth payoffs / discontinuities | Finite differences or carefully designed AD/smoothing |
| Monte Carlo Greeks | AD can help when pathwise derivatives are available |
| Stress testing (many scenarios) | Cached Jacobians with periodic refresh |

---

## 22.4 Controlled Perturbations: The Forward-Rate Approach

### 22.4.1 The Problem with Quote Bumps

Par-point deltas answer the question: "What happens if this specific market quote moves by 1 bp?" But this conflates two things:
1. The economic risk to that tenor of the curve
2. The artifacts of how the curve builder responds to that bump

The see-saw effect shows these can diverge dramatically. A 1 bp 30Y swap bump doesn't represent a 60 bp swing in 30Y/31Y forward spread—that's a curve-fit artifact.

### 22.4.2 Bumping the Forward Curve Directly

An alternative approach perturbs the forward curve $f(t)$ directly with specified functional shapes $\mu_k(t)$. The sensitivity is computed as a **Gâteaux derivative**:

$$\boxed{\partial_k V_0 = \left.\frac{d V_0(f(t) + \varepsilon \mu_k(t))}{d\varepsilon}\right|_{\varepsilon=0}}$$

This is a directional derivative in function space. We shift the entire forward curve in direction $\mu_k$ and measure the first-order PV response.

This is the mathematical foundation of forward-rate deltas: sensitivities to specified forward-curve shapes rather than to individual market quotes.

### 22.4.3 Standard Basis Functions

Two common choices for $\mu_k(t)$ are:

**Piecewise Triangular** ("tent"):

$$\mu_k(t) = \frac{t - t_{k-1}}{t_k - t_{k-1}} \mathbf{1}_{\{t \in [t_{k-1}, t_k)\}} + \frac{t_{k+1} - t}{t_{k+1} - t_k} \mathbf{1}_{\{t \in [t_k, t_{k+1})\}}$$

This creates a "tent" function peaking at $t_k$, declining linearly to zero at adjacent nodes. It's the forward-rate analog of key-rate shifts in yield space.

**Piecewise Flat** ("bucket"):

$$\mu_k(t) = \mathbf{1}_{\{t \in [t_k, t_{k+1})\}}$$

This simply raises all forwards in bucket $[t_k, t_{k+1})$ by 1 bp. A common choice is a quarterly grid aligned to liquid short-rate instruments, so each bucket corresponds to a familiar “risk slice.”

### 22.4.4 From Forward Deltas to Hedge Portfolios

Forward-rate deltas $\partial_k V_0$ give a detailed picture of where risk resides, but the $\mu_k$ basis is usually *not* the same as your tradable hedge set. You still need a mapping from “risk buckets” to “hedge instruments.”

To translate forward deltas into hedge notionals, we use the Jacobian in reverse. Define:
- $\partial \mathbf{H}$: the matrix where column $k$ contains forward deltas $\partial_k H_l$ for hedging instrument $l$
- $\partial \mathbf{V}_0$: the vector of forward deltas for the portfolio

The optimal hedge weights $\mathbf{p}$ solve a least-squares problem:

$$\boxed{\hat{\mathbf{p}} = \underset{\mathbf{p}}{\text{argmin}} \left( \sum_k W_k^2 \left(\mathbf{p}^\top \partial_k \mathbf{H} - \partial_k V_0\right)^2 + \sum_l U_l^2 p_l^2 \right)}$$

where:
- $W_k$ weights the importance of hedging bucket $k$
- $U_l$ weights the "reluctance" to use instrument $l$ (e.g., based on bid-ask spread)

In the simple case of equal weights and $N = K$ (same number of hedging instruments as risk buckets):

$$\mathbf{p} = (\partial \mathbf{H}^\top)^{-1} \partial \mathbf{V}_0$$

This is the **Jacobian method for interest rate deltas**.

This setup is intentionally flexible: you can choose the shock basis $\mu_k$ (risk reporting) and the hedge instrument set (execution) separately, then solve for the best hedge in a least-squares sense.

> **Deep Dive: The Exploding Hedge**
>
> What happens if you try to hedge a 10-year risk using only 2-year and 30-year swaps?
> *   Math: You are trying to invert a singular (or nearly singular) Jacobian matrix.
> *   Result: The math will give you an answer, but it will be unstable (very large offsetting notionals).
> *   Reality: You have created a massive position to hedge a small risk, relying on a delicate correlation that might break.
>
> **Rule of Thumb**: If hedge notionals are huge relative to the risk you are trying to neutralize, the problem is ill-posed. Stop and add a better hedging instrument (or change the risk basis).
>
> **Mathematical Diagnosis:** Compute the condition number of the Jacobian. A large $\kappa(J)$ means small delta errors can produce large swings in hedge notionals.

### 22.4.5 Cumulative Par-Point Approach

A practical tweak to the standard par-point method addresses the see-saw effect directly. The **cumulative par-point approach** retains earlier bumps when computing subsequent deltas.

For the $(i+1)$-th delta, the two curves are:
- Base quotes: $(q_1 + \Delta q_1, \ldots, q_i + \Delta q_i, q_{i+1}, \ldots, q_N)$
- Bumped quotes: $(q_1 + \Delta q_1, \ldots, q_i + \Delta q_i, q_{i+1} + \Delta q_{i+1}, \ldots, q_N)$

Intuition: because the “earlier” quotes have already moved, the incremental bump to quote $q_{i+1}$ tends to create less extreme forward-curve distortions than the standard “one quote at a time from the same base” approach. By construction, the sum of cumulative deltas equals the parallel delta (for the chosen bump design).

The cumulative approach can be implemented in the Jacobian framework using piecewise flat forward shifts:

$$\mu_i(t) = \mathbf{1}_{\{t \in [T_{i-1}, T_i)\}}$$

This specification yields a variation where forward-curve shocks are similarly scaled across maturities (instead of growing mechanically with maturity).

---

## 22.5 Multi-Curve Risk Decomposition

### 22.5.1 The Post-Crisis Reality

Chapters 18–21 established that modern curve construction requires multiple interrelated curves:
- A **discounting curve** (typically OIS) for present-valuing cash flows
- **Projection curves** for different tenors (3M LIBOR, 6M LIBOR, SOFR)
- **Basis curves** linking different projection curves

Each of these curves responds to different market quotes. A comprehensive risk report must show sensitivity to each type.

### 22.5.2 Orthogonal Risk Decomposition

In a spread-based multi-curve build (base index curve + basis spreads + discounting curve), you can choose the quote set so that bumps correspond to distinct *economic* risk types:

- **Overall level risk (projection curves together):** bump non-basis instruments on the base index curve (e.g., vanilla swaps/FRAs for the base tenor). In forward-rate space, this tends to move all projection curves together.
- **Discounting risk:** bump funding/discounting instruments (e.g., OIS quotes) while holding projection-curve quotes fixed. PV changes through discount factors only.
- **Basis risk:** bump basis swap spreads that define a non-base projection curve relative to the base. This moves one index curve relative to another.

The point is not that these risks are literally independent in the market; it is that the *reporting basis* keeps “like risks” together and makes aggregation/hedging cleaner.

### 22.5.3 Why Independence Matters

If you construct each tenor curve independently from its own swap set, “level risk” becomes ambiguous: a 3M-swap bump moves only the 3M curve, a 6M-swap bump moves only the 6M curve, and there is no natural separation between “all rates moved” and “basis moved.”

Consider two approaches to building a 3M LIBOR curve and 6M LIBOR curve:

**Approach A (Independent):** Build each from its own set of swaps.
- Bumping a 3M swap moves the 3M curve
- Bumping a 6M swap moves the 6M curve
- No clear way to aggregate "total level" risk

**Approach B (Spread-Based):** Build 3M as the base, then 6M as spread to 3M.
- Bumping a 3M swap moves both curves in parallel (level risk)
- Bumping a 3M/6M basis spread moves only 6M relative to 3M (basis risk)
- Risks naturally decompose

### 22.5.4 Worked Example: Decomposing a Basis Swap

**Setup:** You hold a 5-year 3M vs 6M basis swap receiving 6M vs paying 3M + 5 bps.

In a spread-based construction, a risk report can separate the sensitivity into buckets that have clear economic meaning:

| Risk Type | Quote Bumped (Example) | Interpretation |
|---|---|---|
| Level (base tenor) | 5Y base-tenor swap rate | “overall level” move that shifts projection curves together |
| Discounting | 5Y OIS rate | PV change through discount factors, holding projection curves fixed |
| Basis | 5Y 3M/6M basis spread | relative move of 6M vs 3M (basis widening/tightening) |

For a basis swap, the decomposition is especially useful because it isolates the relative-curve component (basis) from overall level and discounting effects.

### 22.5.5 Risk Limits and Bucket Monitoring

Computing sensitivities is only half the story. A trading desk must also have a framework for **risk limits**—constraints on how much exposure can be taken in each bucket. This is the organizational infrastructure that translates Greeks into risk management.

> **Desk Reality:** risk is monitored through multiple “views” (net DV01, bucket DV01, curve-spread/twist exposures, basis buckets).
> **Common break:** a portfolio can look “net flat” in DV01 while being large in **gross** buckets, or risk can accumulate between monitored tenors (“bucket holes”).
> **What to check:** always look at both net and gross exposures, and make sure bucket/PCA coverage matches the products the desk actually trades.

**Common limit views (illustrative; desk-specific):**
- Aggregate *net* DV01 and/or duration
- Bucket/key-rate DV01 limits
- Curve spread/twist limits (e.g., 2s10s, 5s30s)
- Basis limits (discounting vs projection; tenor basis)

**The Daily Risk Cycle:**

1. **End of Day (EOD):** Full curve rebuild, complete Greeks calculation, risk report generated
2. **Overnight:** Risk management reviews exposures vs limits, flags breaches
3. **Morning:** Trader receives limit breach report, must explain or reduce
4. **Intraday:** Approximate Greeks using cached Jacobians + live quotes
5. **Throughout:** Trading system enforces "hard limits" that block limit-breaching trades

---

## 22.6 P&L Predict and P&L Explain

### 22.6.1 The Gold Standard: P&L Predict

A risk system's quality is ultimately measured by **P&L predict**: do the sensitivities you compute explain the P&L you observe?

Given, at time $t$, first- and second-order terms $\nabla^H(t)$ and $A^H(t)$, and the observed market data movement over $[t,t+h]$ denoted $\delta$, we would expect the time $t+h$ portfolio value to be approximately (here $\delta$ is a realized market move vector, not the 1bp bump used earlier in curve deltas):

$$\boxed{V(t+h) \approx V(t) + \frac{\partial V}{\partial t} \cdot h + \nabla^H(t) \cdot \delta + \frac{1}{2} \delta^\top \cdot A^H(t) \cdot \delta}$$

where $A^H(t)$ is the Hessian matrix of second derivatives.

**Check (DV01 dot move sanity):** If you report quote DV01s in currency per bp using the down-bump definition $DV01_{q_i}=PV(q_i-1\text{bp})-PV(q)$, then for a realized quote move $\Delta q_i$ measured in bp,
$$\Delta V_{\text{1st order}} \approx -\sum_i DV01_{q_i} \cdot \Delta q_i.$$
Example: if $DV01_{5Y}=+\$10{,}000/\text{bp}$ and the 5Y quote moves **down** 3 bp, the predicted contribution is $+\$30{,}000$. If your sign comes out opposite, you likely mixed “up-bump delta” and “down-bump DV01.”

The right-hand side is the **P&L predict**: what yesterday’s sensitivities would have predicted for today’s value given the realized market moves. Persistent residuals (actual minus predicted) are a diagnostic: they often indicate missing risk factors, inconsistent curves, stale sensitivities, or nonlinear effects on large moves.

### 22.6.2 When P&L Predict Fails

Poor P&L predict often signals curve/risk methodology problems:

1. **Inconsistent Curves:** Using different curves for pricing vs risk
2. **Stale Jacobians:** Using yesterday's Jacobian with today's quotes
3. **Missing Risk Factors:** Not capturing basis or cross-currency exposures
4. **Interpolation Artifacts:** See-saw effects creating phantom exposures

A common failure mode is using different curves (or different curve-building rules) for valuation vs risk: PV is marked on one mapping from quotes to curves, while deltas are computed under another, so “delta × move” does not line up with the MTM.

### 22.6.3 P&L Explain: Waterfall vs Bump-and-Reset

While P&L predict uses sensitivities to forecast future P&L, **P&L explain** (or P&L attribution) works backward to attribute observed P&L to specific market moves.

**Waterfall Explain:** Bump market variables one at a time, cumulatively:

$$E_i = V(t+h; \Theta + (\delta_1, \ldots, \delta_i, 0, \ldots, 0)^\top) - V(t+h; \Theta + (\delta_1, \ldots, \delta_{i-1}, 0, \ldots, 0)^\top)$$

This always sums exactly to total P&L but depends on the ordering of market variables.

**Bump-and-Reset Explain:** Bump each variable independently:

$$E_i = V(t+h; \Theta + (0, \ldots, \delta_i, 0, \ldots, 0)^\top) - V(t+h; \Theta)$$

This is order-independent but leaves an unexplained residual due to cross-convexity terms.

### 22.6.4 Theta Calculation

A subtle but important point: when computing the time-decay ("theta") component of P&L, what should be held fixed? Simply freezing all market quotes causes problems because some instruments have fixed maturity dates (futures) while others have fixed tenors (swap quotes).

One approach is to define a “forward” market-data vector (what you expect market inputs to be at $t+h$, conditional on information at $t$):

$$\Theta_{\text{mkt}}^f(t) = \mathbb{E}^{t+h}(\Theta_{\text{mkt}}(t+h) | \mathcal{F}_t)$$

and define theta as:

$$\frac{\partial V}{\partial t} \approx \frac{V(t+h; \Theta_{\text{mkt}}^f(t)) - V(t)}{h}$$

This is consistent with rolling the discount curve forward over a short horizon via:

$$P(t+h, T) = \frac{P(t, T)}{P(t, t+h)} \approx P(t, T)(1 + r(t)h)$$

which matches the idea that a locally risk-free position earns the short rate over a short holding period.

### 22.6.5 The P&L Predict Diagnostic Checklist

When P&L predict is consistently off, work through this diagnostic workflow:

> **Desk Reality:** desks monitor “unpredicted P&L” as a control on curve builds and risk methodology.
> **Common break:** inconsistent curves, stale Jacobians, missing factors (basis/xccy), theta/roll conventions, and nonlinear effects on large moves.
> **What to check:** work through the checklist below and re-run P&L predict after each fix.

**Step 1: Check for curve consistency**
- Are valuation and risk using the same curves and curve-building rules?
- Were there any curve overrides or manual marks applied?

**Step 2: Check Jacobian freshness**
- When was the Jacobian last computed?
- Have quotes moved materially since then? If yes, force a refresh and recompute risk.

**Step 3: Check for missing risk factors**
- Is basis risk captured (discounting vs projection; tenor basis)?
- Is cross-currency basis captured for XCCY trades?
- Are all relevant curves included in the risk calculation?

**Step 4: Check for interpolation artifacts**
- Plot the forward-curve response to a bump: do you see oscillations/see-saw?
- Compare deltas from bootstrap vs spline builds; look for spurious distant-tenor exposures.

**Step 5: Check theta / roll conventions**
- Is theta computed with a consistent “forward values” convention, or by freezing all quotes?
- For futures positions, is the roll handled consistently with the product’s fixed maturity?

**Step 6: Check second-order effects**
- If market moves were large, convexity/cross-gamma can dominate.
- Compute $\tfrac{1}{2}\delta^\top A^H \delta$ explicitly and compare to the residual.

**Step 7: Check for lifecycle effects**
- Fixings, coupons, settlements, resets, and rolls can create P&L not captured by “pure delta × move.”

**Acceptable P&L Predict Metrics:**

| Metric | What you want to see | Common red flag |
|--------|-----------------------|-----------------|
| Explained ratio | stable and high | persistently low → missing factors or inconsistent curves |
| Mean residual | near zero over time | systematic bias → model/curve drift |
| Residual volatility | small vs total P&L | high → missing nonlinearities or basis/lifecycle effects |
| Autocorrelation | near zero | persistent patterns → stale risk, missing carry/roll conventions |

### 22.6.6 Best Practice: Consistency

**Use the same curves for valuation and risk.** If the pricing curve is a tension spline, compute deltas by bumping the tension spline. If this produces locality problems, address them through the Jacobian method or controlled perturbations—not by using a different risk curve.

---

## 22.7 Practical Implementation Considerations

### 22.7.1 Numerical Stability

Several techniques improve the stability of computed sensitivities:

**Central Differences:** Use $(V^+ - V^-)/(2\delta)$ rather than $(V^+ - V)/\delta$. The symmetric error cancellation reduces truncation error from $O(\delta)$ to $O(\delta^2)$.

**Fixed Random Seeds / Frozen Numerics:** When computing Greeks by Monte Carlo (or other noisy numerics), keep the random seed and other numerical choices fixed between the base and bumped runs. This reduces “Greek noise” so the finite-difference signal dominates.

**Bump Size Selection:** Too large a bump introduces nonlinearity; too small causes numerical noise. Typical practice uses 1 bp (\$10^{-4}$) for rate bumps, though this may need adjustment for illiquid instruments or volatile periods.

### 22.7.2 Curve Overlays

Real-world curves may require discontinuities that smooth interpolation cannot capture. One approach is a **curve overlay** for known special-date effects (e.g., turn-of-year).

The forward curve is written as:

$$f(t) = \varepsilon_f(t) + f^*(t)$$

where $\varepsilon_f(t)$ is a user-specified overlay containing known discontinuities (e.g., a jump on December 31 / January 1), and $f^*(t)$ is the smooth component to be fitted.

The curve construction algorithm then fits $f^*(t)$, with the overlay $\varepsilon_f(t)$ applied as a post-processing step to discount factors.

### 22.7.3 Computational Efficiency

For large portfolios with frequent risk updates:

**Cache Jacobians:** The Jacobian $J = \partial x / \partial q$ changes more slowly than intraday quotes. Refresh $J$ periodically (or after large moves), and use matrix multiplication for fast intraday updates.

**Analytical Sensitivities:** Where possible, compute $\nabla_x V_0$ analytically rather than by bumping. For vanilla instruments, cash flow sensitivities to discount factors are trivial.

**Parallel Computation:** The $N$ columns of the Jacobian can be computed independently—natural for parallel architectures.

### 22.7.4 When to Refresh the Jacobian

> **Desk Reality:** Jacobians are refreshed on a schedule and after “big” changes, not on every tick.
> **Common break:** a stale Jacobian drifts away from today’s curve, deltas degrade, and P&L predict deteriorates.
> **What to check:** track how far quotes moved since the last refresh, monitor nonlinearity, and refresh after events/rolls/large trades.

**Common refresh triggers (illustrative; desk-specific):**
- Large quote moves since the last full rebuild (linearization error grows)
- Calendar rolls / new fixings / instrument set changes (your quote set is not identical)
- Major market events or regime changes
- Very large new trades (pre-trade risk accuracy)
- End-of-day “official” risk run (limits and controls)

**Computational scaling snapshot (qualitative):**
- Full bump-and-rebuild across $N$ quotes: $O(N)$ curve builds (often \$2N$ for central differences) plus repricing.
- Jacobian refresh: similar cost to bumping $N$ quotes, but amortized across many intraday risk runs.
- Cached Jacobian risk update: node sensitivities + matrix multiplies.
- Reverse-mode AD: one backward sweep per valuation graph to obtain sensitivities to many inputs (constant-factor overhead).

---

## Summary

This chapter developed the machinery for computing and interpreting curve risk in a multi-curve world:

1. **The chain rule** connects quote risk to node risk through the curve builder's Jacobian: $\frac{dV_0}{dq} = \nabla_x V_0 \cdot J$

2. **Par-point deltas** (bump-and-rebuild) are simple but depend critically on interpolation choices. Poor locality causes curve-fit artifacts.

3. **The see-saw effect** shows that a 1 bp swap bump can translate to tens of basis points in forward space at the long end—a mathematical consequence of how swap rates relate to forward rates.

4. **The Jacobian method** decouples risk calculation from curve construction, enabling efficient computation and explicit analysis of how bumps propagate.

5. **Algorithmic differentiation (reverse mode)** can compute sensitivities to many inputs in a single backward sweep (constant-factor overhead), but it requires instrumented, differentiable valuation code.

6. **Controlled perturbations** shock the forward curve directly with economically meaningful shapes, avoiding quote-bump artifacts.

7. **Multi-curve decomposition** separates overall level risk, discounting risk, and basis risk when curves are built in a spread-based framework.

8. **Risk limits** provide the organizational framework for using sensitivities—bucket limits, twist limits, and aggregation rules.

9. **P&L predict** is the ultimate test: your deltas should explain your daily P&L. The diagnostic checklist helps identify when the risk system is broken.

10. **Implementation considerations** include numerical stability (central differences, fixed seeds), curve overlays for special dates, computational efficiency through Jacobian caching and AD, and refresh criteria.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Par-point delta** | PV change from 1 bp bump to one quote, after curve rebuild | Standard risk measure aligned with hedging instruments |
| **Locality** | Property that a local quote bump produces a local curve change | Prevents spurious risk attribution to distant tenors |
| **See-saw effect** | Large forward-rate oscillations from small quote bumps | Artifact of curve construction that distorts risk reports |
| **Jacobian** | Matrix $J = \partial x / \partial q$ mapping quotes to curve nodes | Enables efficient risk computation via chain rule |
| **Algorithmic differentiation (AD)** | Differentiate valuation code via the chain rule; reverse mode gives many sensitivities efficiently | Avoids running one separate bump per risk factor |
| **Gâteaux derivative** | Directional derivative in function space | Defines controlled perturbation sensitivities |
| **Controlled perturbation** | Direct forward-curve bump with specified shape | Avoids curve-fit artifacts, aligns with bucket hedging |
| **Orthogonal decomposition** | Separation of level, discounting, and basis risk | Enables proper aggregation and hedging |
| **P&L predict** | Ability of computed deltas to explain realized P&L | Ultimate validation of risk methodology |
| **P&L explain** | Attribution of realized P&L to specific market moves | Diagnostic tool for understanding P&L drivers |
| **Bucket limits** | Maximum allowed exposure per tenor bucket | Prevents concentration risk |
| **Condition number** | $\kappa(J)$ measures how hedge ratios react to small delta errors | Large $\kappa(J)$ → unstable hedges and “exploding notionals” |

---

## Notation

| Symbol | Meaning | Units / Convention |
|--------|---------|-------------------|
| $V(t)$ | portfolio PV at time $t$ | currency; positive = asset |
| $q = (q_1,\ldots,q_N)$ | market quote vector | each $q_i$ has its own units (rate in decimals, price, spread, …) |
| $x = (x_1,\ldots,x_M)$ | curve state / nodes | depends on parameterization (DFs, zero rates, forwards, spline coeffs, …) |
| $x = B(q)$ | curve builder mapping | curve rebuild rule is part of $B$ |
| $J=\partial x/\partial q$ | builder Jacobian | node-units per quote-unit |
| $\nabla_x V$ | node sensitivities | currency per node-unit |
| $dV/dq$ | quote sensitivities | currency per quote-unit |
| $\delta_q$ | bump size for rate-like quotes | \$1\text{bp}=10^{-4}$ in decimal rate units |
| $DV01_{q_i}$ | quote DV01 for quote $q_i$ | $PV(q_i-1\text{bp})-PV(\text{base})$; currency per 1bp (down-bump) |
| $f(t)$ | instantaneous forward rate curve | per year; day count/compounding must be stated |
| $\mu_k(t)$ | forward shock basis function | unitless shape; $f \to f+\varepsilon\mu_k$ shifts forwards by $\varepsilon$ |
| $\partial_k V$ | Gâteaux derivative w.r.t. $\mu_k$ | currency per unit $\varepsilon$ (often reported per 1bp) |
| $\Theta_{\text{mkt}}(t)$ | market data vector | the risk factors actually used by valuation |
| $\delta$ | realized market move | $\delta = \Theta_{\text{mkt}}(t+h)-\Theta_{\text{mkt}}(t)$ |
| $\nabla^H(t)$ | sensitivities w.r.t. $\Theta_{\text{mkt}}$ | currency per unit of each market input |
| $A^H(t)$ | Hessian w.r.t. $\Theta_{\text{mkt}}$ | currency per (input unit)$^2$ |
| $\bar{x}_j$ | adjoint variable in AD | $\partial V/\partial x_j$ in reverse sweep |
| $\kappa(J)$ | condition number of $J$ | dimensionless; large $\kappa(J)$ = ill-conditioned |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the chain rule for quote risk? | $\frac{dV_0}{dq} = \frac{\partial V_0}{\partial x} \cdot \frac{\partial x}{\partial q} = \nabla_x V_0 \cdot J$ |
| 2 | What is a par-point delta? | The PV change when you bump one market quote by 1 bp and rebuild the curve |
| 3 | What does "locality under perturbation" mean? | A bump to one quote produces only a local change in the curve, not global oscillations |
| 4 | What causes the see-saw effect? | When swap rates are held fixed except at one maturity, the forward rates must swing dramatically to satisfy the constraint |
| 5 | Write the approximate relationship between swap rate $S_n$ and forward rate $L_n$ | $L_n \approx n \cdot S_n - (n-1) \cdot S_{n-1}$ |
| 6 | If the 30Y swap rate bumps 1 bp while 29Y/31Y are fixed, how much does $L_{30}$ move? | Approximately 30 bp |
| 7 | What is the Jacobian in curve risk? | $J = \partial x / \partial q$, the matrix mapping quote changes to curve-node changes |
| 8 | What is a Gâteaux derivative in curve risk? | $\partial_k V_0 = \frac{d}{d\varepsilon}V_0(f + \varepsilon \mu_k)\big|_{\varepsilon=0}$, the sensitivity to a controlled forward-curve perturbation |
| 9 | Name two types of basis functions for controlled perturbations | Piecewise triangular ("tent") and piecewise flat ("bucket") |
| 10 | Why can $C^2$ cubic splines be problematic for par-point risk? | They can produce see-saw oscillations in the forward curve from a local quote bump (poor locality) |
| 11 | What does adding tension to a spline accomplish? | Improves locality by penalizing curvature, at the cost of some smoothness |
| 12 | What is the cumulative par-point approach? | Retain each bump when computing subsequent deltas, reducing forward-rate extremes |
| 13 | In multi-curve risk, what three components does the orthogonal decomposition provide? | Overall level risk, discounting risk, and basis risk |
| 14 | Why should you use the same curves for pricing and risk? | To ensure good P&L predict—different curves cause unexplained P&L |
| 15 | What is P&L predict? | The ability of computed deltas to explain the portfolio's realized daily P&L |
| 16 | What is the difference between P&L predict and P&L explain? | Predict uses sensitivities to forecast future P&L; explain attributes observed P&L to market moves |
| 17 | How do you improve numerical stability when computing Jacobians? | Use central (two-sided) finite differences, fixed random seeds, appropriate bump sizes |
| 18 | If $N$ hedging instruments exist and $K$ risk buckets, when is the hedge unique? | When $N = K$ and the Jacobian is invertible |
| 19 | What determines whether quote bumps "propagate locally"? | The interpolation method used in curve construction |
| 20 | Why might independent curve construction for different tenors cause problems? | It prevents natural aggregation of level risk vs basis risk |
| 21 | What is a curve overlay? | A user-specified adjustment $\varepsilon_f(t)$ added to the forward curve for special dates (e.g., turn-of-year) |
| 22 | Why should theta be computed using forward values rather than frozen quotes? | Because some instruments have fixed maturities (futures) while others have fixed tenors (swaps), causing distortions when quotes are simply frozen |
| 23 | What is algorithmic differentiation (AD)? | Differentiate valuation code via the chain rule; reverse mode computes sensitivities to many inputs in one backward sweep (constant-factor overhead vs valuation) |
| 24 | What is the adjoint equation in AD? | $\bar{x}_j = \sum_{k: x_j \to x_k} \bar{x}_k \frac{\partial x_k}{\partial x_j}$, propagating sensitivities backward from output to inputs |
| 25 | What does a large $\kappa(J)$ warn you about? | Ill-conditioned mapping/hedge: small delta errors can imply huge, unstable hedge notionals |
| 26 | When should you refresh the Jacobian intraday? | After large quote moves, calendar rolls/fixings, major events, or very large new trades (triggers are desk-specific) |

---

## Mini Problem Set

### Questions

**1. Chain Rule Application**
A portfolio has a **node DV01** to the 5Y zero rate defined as $DV01_{y_5}:=PV(y_5-1\text{bp})-PV(\text{base})=+\$50{,}000$ per bp. Your curve builder implies that a 1bp down bump to the 5Y par swap quote produces a 0.98bp down move in the 5Y zero rate (with other quotes held fixed). Approximate the 5Y **quote DV01** $DV01_{q_{5Y}}$ in USD per bp.

**2. See-Saw Calculation**
Using the approximation $L_n \approx n \cdot S_n - (n-1) \cdot S_{n-1}$, compute the change in $L_{10}$ and $L_{11}$ when $S_{10}$ is bumped by 1 bp while $S_9$ and $S_{11}$ are held fixed.

**3. Locality Comparison**
Explain why bootstrapping with piecewise-linear zero yields has better locality than natural cubic splines, despite producing a less smooth forward curve.

**4. Jacobian Dimensions**
A curve is built from 20 benchmark instruments. The curve is parameterized by 60 monthly forward rates. What are the dimensions of the Jacobian $J$?

**5. Controlled Perturbation**
Define a piecewise-flat basis function $\mu_3(t)$ for the bucket $[t_3, t_4) = [2Y, 3Y)$. If $\partial_3 V_0 = -12{,}500$ USD per bp of forward rate shift, interpret this economically.

**6. Multi-Curve Decomposition**
A trader holds a 5Y receiver swap (receive fixed vs 3M SOFR). List the three types of risk exposure and explain which curve bumps would reveal each.

**7. P&L Predict Diagnostic**
Yesterday's risk report showed 5Y **quote DV01** (down-bump definition) = +\$8,000 per bp. Overnight, the 5Y quote moved **down** 3 bp. First-order predicted P&L is +\$24,000. Actual P&L was +\$31,000. List three possible explanations for the discrepancy.

**8. Two-Curve Pricing**
If a desk uses a smooth cubic spline curve for pricing but a bootstrapped curve for risk, what problem might arise? How would you diagnose it?

**9. Cumulative vs Standard Par-Point**
Why does the cumulative par-point approach produce less extreme forward curve shifts than the standard approach? In what sense is the cumulative approach more "realistic"?

**10. Waterfall vs Bump-and-Reset**
A portfolio has significant gamma (second-order sensitivity). Which P&L explain method—waterfall or bump-and-reset—will produce a smaller unexplained residual? Why?

**11. AD Complexity Comparison**
A desk has 500 curve nodes and 1,000 trades. Compare the computational cost (in terms of valuation operations) of:
(a) Bump-and-rebuild with central differences
(b) Cached Jacobian approach
(c) Adjoint AD

**12. Risk Limit Scenario**
A trader has the following DV01 exposures:

| Tenor | DV01 (USD/bp) |
|-------|---------------|
| 2Y | +\$40,000 |
| 5Y | −\$80,000 |
| 10Y | +\$60,000 |
| 30Y | −\$20,000 |

The desk has limits: (i) $|\text{net DV01}| \le \$50{,}000$, where net DV01 is the signed sum across buckets, and (ii) $|DV01|\le \$50{,}000$ in any single bucket. Which limits are breached? What trades might the trader execute to come into compliance (ignore cross-bucket effects)?

### Solution Sketches (Selected)

**1.** $DV01_{q_{5Y}} \approx 0.98 \times DV01_{y_5} \approx 0.98 \times \$50{,}000 = \$49{,}000$ per bp.

**2.** $\Delta L_{10} \approx 10 \times 1 = +10$ bp. $\Delta L_{11} \approx 11 \times 0 - 10 \times 1 = -10$ bp. The 20 bp swing between adjacent forwards is the see-saw effect.

**3.** Bootstrapping with piecewise-linear yields makes each zero rate depend only on quotes up to that maturity—bumping a short quote doesn't affect long rates. Cubic splines enforce global smoothness constraints, so every quote bump potentially affects every forward rate.

**4.** $J$ is \$60 \times 20$ (rows = number of nodes, columns = number of quotes).

**5.** $\mu_3(t) = 1$ for $t \in [2Y, 3Y)$, 0 elsewhere. The sensitivity $-12{,}500$ means if all forwards between 2Y and 3Y rise by 1 bp, the portfolio loses \$12,500.

**6.** (i) Level risk: bump 5Y SOFR swap rate, (ii) Discounting risk: bump 5Y OIS rate holding SOFR fixed, (iii) Basis risk: bump SOFR-OIS basis spread. The receiver swap has mainly level risk (duration to SOFR curve) and modest discounting risk.

**7.** Possible explanations: (i) second-order effects (convexity/gamma) over a 3bp move, (ii) missing factors (basis/discounting/curve shape moved alongside the 5Y quote), (iii) stale Jacobian or inconsistent curves (valuation vs risk), (iv) theta/roll/lifecycle effects not included in the first-order predict.

**8.** The deltas explain PV changes in the bootstrapped curve, but actual MTM uses the cubic spline. If quotes move and the two curves respond differently, predicted P&L won't match actual. Diagnose by computing P&L predict ratio over time; it should be close to 1 with small scatter.

**9.** In the cumulative approach, when computing the delta to quote $i$, quotes \$1, \ldots, i-1$ have already been bumped. This means the forward curve has already shifted in a parallel-like fashion, so the marginal bump to quote $i$ produces a smaller incremental forward move. This is more realistic because in practice, if the 30Y rate moves, neighboring rates typically move too.

**10.** Waterfall produces zero unexplained residual by construction (it's an identity). Bump-and-reset leaves a residual proportional to cross-gamma terms. However, waterfall's attribution depends on the arbitrary ordering of market variables.

**11.** One reasonable way to count is “trade valuations”:
(a) Bump-and-rebuild (central diff): $\approx 2 \times 500 \times 1000 = 1{,}000{,}000$ trade valuations (plus curve rebuild overhead).
(b) Cached Jacobian: refresh $J$ costs $\approx 2 \times 500$ curve builds (amortized); per run compute node sensitivities once per trade ($\sim 1000$ valuations or analytic) + matrix multiplies.
(c) Reverse-mode AD: per run $\sim c \times 1000$ valuation-equivalents to get sensitivities to many inputs, where $c$ is a small constant (implementation- and memory-dependent).

**12.**
- Net DV01: \$40 - 80 + 60 - 20 = 0$ → $|0| \le \$50k$ ✓
- Per-bucket: 5Y at $|{-}80k| > 50k$ ✗, 10Y at $|{+}60k| > 50k$ ✗

To comply (thinking in bucket-DV01 units): add $+\$30k$ of 5Y bucket DV01 and add $-\$10k$ of 10Y bucket DV01 (e.g., via swaps whose bucket DV01 is concentrated at those tenors). This would give:
- 2Y: +\$40k, 5Y: −\$50k, 10Y: +\$50k, 30Y: −\$20k
- All buckets at or below \$50k; net DV01 = \$0 ≤ \$50k ✓

---

## References

- (Andersen & Piterbarg, *Interest Rate Modeling*, “6.4 Managing Yield Curve Risk” and “22.2 P&L Predict / P&L Explain”)
- (Tuckman & Serrat, *Fixed Income Securities: Tools for Today’s Markets*, “Key Rate Shifts”)
- (Oosterlee, *Mathematical Modeling and Computation in Finance*, “Example 12.3.5”)
- (Hull, *Risk Management and Financial Institutions*, section on OIS discounting / curve choice in swap valuation)
- (Crépey, *Counterparty Risk and Funding*, introduction (multiple curves and discounting vs projection))
- (Nocedal & Wright, *Numerical Optimization*, “The Reverse Mode” (automatic differentiation))
