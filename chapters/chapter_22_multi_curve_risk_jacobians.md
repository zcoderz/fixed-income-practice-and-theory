# Chapter 22: Curve Risk Management in a Multi-Curve World — Par-Point Deltas, Jacobians, and Controlled Perturbations

---

## Introduction

You bump the 5-year swap rate by 1 basis point. Which discount factors move? By how much? Does the change stay localized around year five, or does it ripple across the entire curve—perhaps even causing the 20-year forward rate to jump by 30 basis points in the opposite direction?

These questions lie at the heart of curve risk management. Chapters 11–14 introduced the *what* of interest rate risk: DV01, duration, key-rate exposures, and bucket sensitivities. Chapter 17 addressed the *how* of curve construction: bootstrapping, interpolation, and the underdetermined nature of fitting a continuous function to discrete market quotes. This chapter bridges these two domains by asking: *how do the choices we made in curve construction affect the risk numbers we compute?*

The answer, it turns out, is "profoundly." As Andersen and Piterbarg emphasize, "the curve construction algorithm employed" directly determines both "the function $V_0(\cdot)$" and the sensitivities we derive from it. A trader using cubic splines might report a very different 5-year delta than one using piecewise-linear bootstrapping—even though both curves reprice the same market instruments exactly. The difference lies in how each method propagates a 1 bp quote bump through the forward curve: one method might keep the perturbation localized, while the other creates a "see-saw" oscillation that spreads risk exposure to unexpected tenors.

**Stakes:** When sensitivities fail to explain daily P&L movements—when the predicted P&L diverges systematically from realized P&L—something in your curve construction or risk methodology is broken. Andersen and Piterbarg warn that using "two different curves—one smooth for pricing, one bootstrapped for risk... tends to suffer from poor P&L predict." The deltas you compute must be consistent with the curves you price on, or your hedges will systematically fail.

**Roadmap:** This chapter develops the tools to understand and control these effects:

- **Section 22.1** establishes the fundamental chain rule connecting quote changes to PV changes through the curve builder—the mathematical backbone of "par-point" or "quote" deltas.

- **Section 22.2** examines what happens when you bump a single quote and rebuild: the par-point approach, locality under perturbation, and the infamous see-saw effect that afflicts certain interpolation methods.

- **Section 22.3** introduces the **Jacobian method**, which decouples risk calculation from curve construction by explicitly computing how curve nodes respond to quote changes.

- **Section 22.4** develops **controlled perturbations**: an alternative approach where we shock the forward curve directly with economically meaningful shapes, avoiding the artifacts of bump-and-rebuild.

- **Section 22.5** addresses multi-curve risk decomposition: in a world of OIS discounting and tenor basis, how do we separate "overall level" risk from "discounting" risk from "basis" risk?

- **Section 22.6** presents P&L predict and P&L explain methodologies—the ultimate validation of any risk system.

- **Section 22.7** covers practical implementation considerations: numerical stability, curve overlays, and computational efficiency.

Throughout, we emphasize a key desideratum: **P&L predict**. The deltas you compute should explain the P&L you observe when quotes move. If they don't, something in your curve build or risk methodology is broken.

---

## 22.1 The Chain Rule: From Quotes to PV

### 22.1.1 Portfolio Value as a Function of Curve Nodes

Consider a portfolio with present value $V_0$. This value depends on the discount factors (or equivalently, zero rates or forward rates) at various maturities. We can write:

$$V_0 = V_0(x_1, x_2, \ldots, x_M)$$

where $x = (x_1, \ldots, x_M)$ represents the curve's internal parameterization—perhaps discount factors at key dates, zero rates at node points, or spline coefficients.

For simple fixed-income instruments, this dependence is often straightforward. A bond paying cash flows $c_j$ at times $t_j$ has:

$$V_0 = \sum_{j=1}^{n} c_j \, P(t_j)$$

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

As Andersen and Piterbarg note, "the Jacobian method serves to decouple risk calculations from curve construction," allowing us to compute the first piece once and multiply by different Jacobians for different curve builds.

### 22.1.4 Hedging Interpretation

The practical significance of this decomposition becomes clear when we consider hedging. If all derivatives $\partial V_0 / \partial q_i$ are zero, "our portfolio would, to first order, be immunized against any move in the yield curve that is consistent with the chosen curve construction algorithm," as Andersen and Piterbarg explain.

If some or all derivatives are non-zero, we can hedge by setting up a portfolio of benchmark securities with notional $-\partial V_0 / \partial q_i$ on the $i$-th security. This is the essence of **bucket-by-bucket immunization**—a practice that, while theoretically "overkill" compared to what a one-factor model would suggest, "has proven to be robust" in practice.

Importantly, Andersen and Piterbarg note that bucket hedging "would correctly reject the notion that we could perfectly hedge a 20 year swap with a 1 month FRA, something that a one-factor interest rate model... would happily accept."

---

## 22.2 Par-Point Deltas and the Locality Problem

### 22.2.1 The Par-Point Approach: Bump and Rebuild

The simplest way to compute quote risk is direct numerical differentiation: bump one quote, rebuild the curve, reprice the portfolio, and report the change. This procedure is sometimes called the **par-point approach**, and the resulting sensitivities are **par-point deltas**.

Andersen and Piterbarg describe it precisely: "The simplest approach to computation of the delta $\partial V_0 / \partial V_i$ involves a manual bump to $V_i$, followed by a reconstruction of the yield curve, and a subsequent repricing of the portfolio $V_0$."

**Algorithm (Par-Point Delta for Quote $q_i$):**

1. Record base portfolio value: $V_0^{\text{base}}$
2. Bump quote $q_i$ by $\delta$ (typically 1 bp = $10^{-4}$): $q_i \to q_i + \delta$
3. Rebuild the entire curve: $x^{\text{bumped}} = B(q_1, \ldots, q_i + \delta, \ldots, q_N)$
4. Reprice the portfolio: $V_0^{\text{bumped}}$
5. Compute the delta: $\frac{\partial V_0}{\partial q_i} \approx \frac{V_0^{\text{bumped}} - V_0^{\text{base}}}{\delta}$
6. Repeat for each benchmark quote.

The appeal is simplicity and generality: it works for any portfolio and any curve construction algorithm. The risk output directly corresponds to hedging instruments—a delta to the 5-year swap quote tells you how much 5-year swap notional to trade to hedge that exposure.

### 22.2.2 Locality Under Perturbation

For the par-point approach to "work properly," as Andersen and Piterbarg emphasize, "it is important that the yield curve construction algorithm is fast and produces clean, local perturbations of the yield curve when benchmark prices are shifted."

**Locality** means that bumping a short-dated instrument should not cause "noticeable movements in long-term yields." If it does, we would reach "the erroneous conclusion that we can perfectly hedge a 20 year swap with a 1 month FRA."

Different interpolation methods have vastly different locality properties. Andersen and Piterbarg illustrate this with Figure 6.7, showing the effect on forward curves from a 1 basis point bump to the 2-year swap in their test data set:

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

What happens when locality fails? Andersen and Piterbarg provide a vivid illustration.

Consider bumping the 2-year swap rate by 1 basis point while holding the 1-year and 3-year rates fixed. With bootstrapping or Hermite splines, the forward curve shows a localized spike near year 2. But with natural cubic $C^2$ splines, the bump "causes a noisy, ringing perturbation... spreading into short- and long-dated parts of the forward curve."

The mathematics explains why. For a rough approximation, swap rates are approximately linear combinations of forward rates. Andersen and Piterbarg derive (equation 6.33):

$$S_n \approx \sum_{i=1}^{n} w_{i,n} L_i$$

where $S_n$ is the $n$-year swap rate, $L_i$ is the $i$-year forward rate, and $w_{i,n} \approx 1/n$ for annual swaps. Inverting this relationship:

$$\boxed{L_n \approx n \cdot S_n - (n-1) \cdot S_{n-1}}$$

This innocent-looking formula has dramatic implications. If we bump $S_n$ by $\delta$ while holding $S_{n-1}$ and $S_{n+1}$ fixed, then:
- $L_n$ shifts by approximately $n \cdot \delta$
- $L_{n+1}$ shifts by approximately $-n \cdot \delta$

Andersen and Piterbarg make this concrete: "If a 30 year swap yield is shifted by 1 basis point, while 29 year and 31 year are kept unchanged, then evidently the forward Libor rate $L_{30}$ will move by 30 basis points, and the rate $L_{31}$ will move by −30 basis points."

This is the **see-saw effect**: a 1 bp quote bump translates to a 60 bp swing in adjacent forward rates at the long end. "If the portfolio whose deltas we are computing happens to contain, say, a spread option on the difference between $L_{30}$ and $L_{31}$," that option's underlying would shift by 60 basis points—far outside the linear regime where first-order sensitivities are meaningful.

> **Practical Warning:** The see-saw effect is most severe at the long end of the curve where swap maturities are spaced 1 year apart. It highlights "the importance of applying shifts to the forward curve that are consistent with real moves of interest rates." In reality, if the 30-year rate moves, the 29-year and 31-year rates almost certainly move as well.

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

A 1 bp swap rate move produces a 60 bp swing in the 30Y/31Y forward spread—exactly as Andersen and Piterbarg predict.

### 22.2.5 Comparing Interpolation Methods

**Setup:** A swap desk builds a curve from par swap quotes at 1Y, 2Y, 3Y, 5Y, 7Y, 10Y, 15Y, 20Y, 30Y. They hold a 7-year receiver swap with $100M notional.

**Experiment:** Bump the 5-year swap rate by 1 bp and compute the 5Y delta under two methods.

**Method A: Piecewise-Linear Zero Yields (Bootstrap)**

The forward curve shows a localized spike between 3Y and 7Y. The 7Y swap shows:
- 5Y Delta: −$3,850 per bp
- Spurious 10Y Delta: $0 per bp

**Method B: Natural Cubic Splines**

The forward curve shows oscillations extending from 2Y to 15Y. The 7Y swap shows:
- 5Y Delta: −$4,120 per bp
- Spurious 10Y Delta: +$480 per bp

The cubic spline method reports a 10Y exposure even though the 7Y swap has no economic reason to be sensitive to 10Y rates (holding other quotes fixed). This is a curve-fit artifact, not economic risk.

### 22.2.6 Tension Splines: A Partial Remedy

One way to improve locality while retaining smoothness is to add **tension** to the spline. Andersen and Piterbarg note that "the usage of a tension factor can have a beneficial impact on risk reports produced by the par-point approach."

Tension essentially penalizes curvature, pulling the spline toward piecewise-linear behavior in regions away from data points. Higher tension improves locality at the cost of forward curve smoothness.

In Figure 6.8, Andersen and Piterbarg demonstrate this dampening effect, showing how adding tension to the $C^2$ spline reduces the "ringing perturbation noise" from Figure 6.7.

---

## 22.3 The Jacobian Method

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

Using central differences (averaging up and down bumps) improves numerical stability. As Andersen and Piterbarg note, "using averaged deltas is typically particularly useful for security prices that depend on yields in a strongly non-linear fashion."

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

**Curve Consistency:** When curves "need to be rebuilt over and over" as quotes move rapidly, the Jacobian provides "changes in benchmark prices [that] can be quickly translated into changes of the forward curve via a matrix multiplication." A full rebuild is only triggered "when the benchmark prices have moved sufficiently far from their initial values."

**Risk Attribution:** The Jacobian explicitly shows how quote bumps propagate to curve nodes. A column of zeros indicates that a quote doesn't affect certain tenors—useful for debugging and intuition.

### 22.3.5 Jacobian Structure and Dimensions

Consider a curve built from $N = 15$ benchmark instruments (deposits, futures, swaps) with $M = 100$ monthly forward rate nodes.

The Jacobian $J$ is $100 \times 15$. Each column shows how a 1 bp bump to one quote moves all 100 forward rates.

**For bootstrapping:** The Jacobian has a block-triangular structure. Bumping a short-dated instrument only affects forward rates up to its maturity.

**For global methods (cubic splines):** The Jacobian may have non-zeros everywhere, reflecting the global coupling inherent in the spline fit.

> **Visual: The Jacobian Heatmap**
>
> Imagine a grid (heatmap).
> *   **Rows**: Forward Rates (1M, 2M, ..., 30Y).
> *   **Columns**: Market Quotes (2Y Swap, 5Y Swap...).
> *   **Ideally**: You want a "Block Diagonal" heatmap. Bumping the 5Y quote should only light up the forward rates near 5Y.
> *   **The Reality**: In a bad curve build (e.g., untensioned spline), the heatmap looks like a checkerboard. Bumping the 2Y quote lights up the 30Y forward rate. This is "Pollution." It means your short-term trades are accidentally hedging your long-term risk.

### 22.3.6 Relationship to Key Rate DV01

The Jacobian method is closely related to the key-rate methodology of Tuckman (Chapter 7). Tuckman defines key rate shifts as triangular perturbations to par yields:

> "Each key rate affects par yields from the term of the previous key rate (or zero) to the term of the next key rate (or the last term). Specifically, the two-year key rate affects all par yields of term zero to five, the five-year affects par yields of term two to 10..."

The key rate 01 then measures the portfolio's sensitivity to each such shift. Tuckman emphasizes that "the sum of the key rate shifts equals a parallel shift in the par yield curve," ensuring that key rate 01s aggregate to total DV01.

The Jacobian framework generalizes this: key rate shifts are specific choices of basis functions $\mu_k(t)$ applied to the yield or forward curve, and the corresponding sensitivities are obtained via the chain rule.

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

Andersen and Piterbarg (equation 6.26) formalize this as the mathematical foundation for "forward rate deltas."

### 22.4.3 Standard Basis Functions

Andersen and Piterbarg describe two common choices for $\mu_k(t)$:

**Piecewise Triangular** (equation 6.27):

$$\mu_k(t) = \frac{t - t_{k-1}}{t_k - t_{k-1}} \mathbf{1}_{\{t \in [t_{k-1}, t_k)\}} + \frac{t_{k+1} - t}{t_{k+1} - t_k} \mathbf{1}_{\{t \in [t_k, t_{k+1})\}}$$

This creates a "tent" function peaking at $t_k$, declining linearly to zero at adjacent nodes. It's the forward-rate analog of key-rate shifts in yield space.

**Piecewise Flat** (equation 6.28):

$$\mu_k(t) = \mathbf{1}_{\{t \in [t_k, t_{k+1})\}}$$

This simply raises all forwards in bucket $[t_k, t_{k+1})$ by 1 bp. This aligns naturally with Eurodollar/SOFR futures buckets.

As Andersen and Piterbarg note, "it is common practice to use $\{t_k\}$ grids spaced three months apart, with dates on Eurodollar futures maturities," giving a detailed picture of where portfolio risk is concentrated.

### 22.4.4 From Forward Deltas to Hedge Portfolios

Forward-rate deltas $\partial_k V_0$ give a detailed picture of where risk resides, but "forward rate deltas do not directly suggest hedging instruments for the medium and long end of the yield curve exposure."

To translate forward deltas into hedge notionals, we use the Jacobian in reverse. Define:
- $\partial \mathbf{H}$: the matrix where column $k$ contains forward deltas $\partial_k H_l$ for hedging instrument $l$
- $\partial \mathbf{V}_0$: the vector of forward deltas for the portfolio

The optimal hedge weights $\mathbf{p}$ solve a least-squares problem (equation 6.29):

$$\boxed{\hat{\mathbf{p}} = \underset{\mathbf{p}}{\text{argmin}} \left( \sum_k W_k^2 \left(\mathbf{p}^\top \partial_k \mathbf{H} - \partial_k V_0\right)^2 + \sum_l U_l^2 p_l^2 \right)}$$

where:
- $W_k$ weights the importance of hedging bucket $k$
- $U_l$ weights the "reluctance" to use instrument $l$ (e.g., based on bid-ask spread)

In the simple case of equal weights and $N = K$ (same number of hedging instruments as risk buckets):

$$\mathbf{p} = (\partial \mathbf{H}^\top)^{-1} \partial \mathbf{V}_0$$

This is the **Jacobian method for interest rate deltas**. As Andersen and Piterbarg note, "the approach has considerable generality as the risk basis functions $\mu_k$ and the hedge portfolio can be chosen freely by the user."

> **Deep Dive: The Exploding Hedge**
>
> What happens if you try to hedge a 10-year risk using only 2-year and 30-year swaps?
> *   Math: You are trying to invert a singular (or nearly singular) Jacobian matrix.
> *   Result: The math will give you an answer, but it will be insane. It might say: "Short \$10 Billion of 2Y and Long \$10 Billion of 30Y."
> *   Reality: You have created a massive position to hedge a small risk, relying on a delicate correlation that might break.
>
> **Rule of Thumb**: If your hedge notionals are 10x larger than your risk notional, your Jacobian is telling you: "You can't get there from here." Stop and find a better hedging instrument.

### 22.4.5 Cumulative Par-Point Approach

A practical tweak to the standard par-point method addresses the see-saw effect directly. The **cumulative par-point approach** (or "waterfall" method) retains each bump when computing subsequent deltas.

For the $(i+1)$-th delta, the two curves are:
- Base: $(V_1 + \Delta V_1, \ldots, V_i + \Delta V_i, V_{i+1}, \ldots, V_N)$
- Bumped: $(V_1 + \Delta V_1, \ldots, V_i + \Delta V_i, V_{i+1} + \Delta V_{i+1}, \ldots, V_N)$

Andersen and Piterbarg explain: "The forward curve shifts implied by the cumulative par-point method are less extreme than those of the ordinary par-point method." Additionally, "the sum of deltas computed by the method is always (by definition) exactly equal to the parallel delta."

The cumulative approach can be implemented in the Jacobian framework using piecewise flat forward shifts (equation 6.34):

$$\mu_i(t) = \mathbf{1}_{\{t \in [T_{i-1}, T_i)\}}$$

This specification "yields an attractive variation of the cumulative par-point method where all forward curve shocks are similarly scaled," unlike the basic cumulative par-point where "the size of forward curve shocks grows linearly with maturity."

---

## 22.5 Multi-Curve Risk Decomposition

### 22.5.1 The Post-Crisis Reality

Chapters 18–21 established that modern curve construction requires multiple interrelated curves:
- A **discounting curve** (typically OIS) for present-valuing cash flows
- **Projection curves** for different tenors (3M LIBOR, 6M LIBOR, SOFR)
- **Basis curves** linking different projection curves

Each of these curves responds to different market quotes. A comprehensive risk report must show sensitivity to each type.

### 22.5.2 Orthogonal Risk Decomposition

With the spread-based curve construction from Chapter 20, sensitivities have "clear, and orthogonal, meaning" (Andersen and Piterbarg):

> **Overall Level Risk:** "Perturbations to instruments used in building the base index curve, e.g. non-basis swaps and FRAs referencing $L^1$, define risk sensitivities to the overall levels of interest rates." These bumps "move all index curves together by the same amount in forward rate space."

> **Discounting Risk:** "Perturbations to funding instruments define sensitivities to discounting." If OIS rates move while projection curves stay fixed, the PV changes through the discount factors only.

> **Basis Risk:** "Perturbations to basis swap spreads for $L^k$-versus-$L^1$ floating-floating basis swaps define basis risk, i.e. the risk that index curves of different tenors do not move in lock step."

This decomposition is powerful because it "allows us to naturally aggregate 'similar' risks such as overall rate level risks, discounting risks, basis risks, while keeping different kinds separate for efficient risk management."

### 22.5.3 Why Independence Matters

Had we "constructed all index curves in separation from each other (from multiple sets of vanilla fixed-floating swaps, say) such automatic aggregation would not be possible."

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

**Setup:** You hold a 5-year 3M vs 6M basis swap receiving 6M LIBOR vs paying 3M LIBOR + 5 bps. Notional $100M.

**Risk Report (Spread-Based Construction):**

| Risk Type | Quote Bumped | Delta (USD/bp) |
|-----------|--------------|----------------|
| Level (3M) | 5Y 3M swap rate | −$2,340 |
| Discounting | 5Y OIS rate | +$45 |
| Basis | 5Y 3M/6M basis spread | +$4,520 |

**Interpretation:**
- If all rates rise 1 bp (parallel), you lose $2,340 (you're short duration through the 3M leg)
- If OIS rises but everything else fixed, you gain $45 (discounting effect)
- If 6M widens vs 3M by 1 bp, you gain $4,520 (you receive the wider leg)

This decomposition tells you exactly what economic exposures you have and which instruments to use for hedging each.

---

## 22.6 P&L Predict and P&L Explain

### 22.6.1 The Gold Standard: P&L Predict

A risk system's quality is ultimately measured by **P&L predict**: do the sensitivities you compute explain the P&L you observe?

Andersen and Piterbarg (Chapter 22) formalize this. If yesterday's position had delta $\nabla^H(t)$ to market data $\Theta_{\text{mkt}}$, and the market moved by $\delta = \Theta_{\text{mkt}}(t+h) - \Theta_{\text{mkt}}(t)$ over a short period $h$, then:

$$\boxed{V(t+h) \approx V(t) + \frac{\partial V}{\partial t} \cdot h + \nabla^H(t) \cdot \delta + \frac{1}{2} \delta^\top \cdot A^H(t) \cdot \delta}$$

where $A^H(t)$ is the Hessian matrix of second derivatives.

The right-hand side is the **P&L predict**—what we expect the portfolio to be worth given observed market moves. The difference between predicted and actual P&L should be small and unbiased. "If systems and models are working properly, the P&L predict should generally be an accurate and unbiased estimate of actual P&L, so monitoring of the unpredicted P&L serves an important control purpose."

### 22.6.2 When P&L Predict Fails

Poor P&L predict often signals curve/risk methodology problems:

1. **Inconsistent Curves:** Using different curves for pricing vs risk
2. **Stale Jacobians:** Using yesterday's Jacobian with today's quotes
3. **Missing Risk Factors:** Not capturing basis or cross-currency exposures
4. **Interpolation Artifacts:** See-saw effects creating phantom exposures

Andersen and Piterbarg explicitly warn about the danger of "building two different curves—one smooth for pricing, one bootstrapped for risk." This approach "tends to suffer from poor P&L predict, in the sense that changes in valuations of a portfolio between two dates are not well explained by first-order sensitivities."

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

Andersen and Piterbarg recommend computing the expected change in market data:

$$\Theta_{\text{mkt}}^f(t) = \mathbb{E}^{t+h}(\Theta_{\text{mkt}}(t+h) | \mathcal{F}_t)$$

and defining theta as:

$$\frac{\partial V}{\partial t} \approx \frac{V(t+h; \Theta_{\text{mkt}}^f(t)) - V(t)}{h}$$

This "rolls" the discount curve forward using:

$$P(t+h, T) = \frac{P(t, T)}{P(t, t+h)} \approx P(t, T)(1 + r(t)h)$$

which is "consistent with the notion that a risk-free portfolio should earn a rate of $r(t)$ over a short holding period."

### 22.6.5 Best Practice: Consistency

**Use the same curves for valuation and risk.** If the pricing curve is a tension spline, compute deltas by bumping the tension spline. If this produces locality problems, address them through the Jacobian method or controlled perturbations—not by using a different risk curve.

---

## 22.7 Practical Implementation Considerations

### 22.7.1 Numerical Stability

Several techniques improve the stability of computed sensitivities:

**Central Differences:** Use $(V^+ - V^-)/(2\delta)$ rather than $(V^+ - V)/\delta$. The symmetric error cancellation reduces truncation error from $O(\delta)$ to $O(\delta^2)$.

**Fixed Random Seeds:** When computing Greeks by Monte Carlo, "fixing the random seed... significantly reduces the standard error." More generally, "one should ideally attempt to freeze as many aspects of a numerical calculation (such as a Monte Carlo seed, the geometry of a PDE grid, etc.) as possible when doing perturbation analysis."

**Bump Size Selection:** Too large a bump introduces nonlinearity; too small causes numerical noise. Typical practice uses 1 bp ($10^{-4}$) for rate bumps, though this may need adjustment for illiquid instruments or volatile periods.

### 22.7.2 Curve Overlays

Real-world curves may require discontinuities that smooth interpolation cannot capture. Andersen and Piterbarg describe the **curve overlay** approach for handling special dates like turn-of-year effects.

The forward curve is written as:

$$f(t) = \varepsilon_f(t) + f^*(t)$$

where $\varepsilon_f(t)$ is a user-specified overlay containing known discontinuities (e.g., a jump on December 31 / January 1), and $f^*(t)$ is the smooth component to be fitted.

The curve construction algorithm then fits $f^*(t)$, with the overlay $\varepsilon_f(t)$ applied as a post-processing step to discount factors.

### 22.7.3 Computational Efficiency

For large portfolios with frequent risk updates:

**Cache Jacobians:** The Jacobian $J = \partial x / \partial q$ changes slowly compared to quotes. Recompute only when quotes have moved "sufficiently far from their initial values."

**Analytical Sensitivities:** Where possible, compute $\nabla_x V_0$ analytically rather than by bumping. For vanilla instruments, cash flow sensitivities to discount factors are trivial.

**Parallel Computation:** The $N$ columns of the Jacobian can be computed independently—natural for parallel architectures.

---

## Summary

This chapter developed the machinery for computing and interpreting curve risk in a multi-curve world:

1. **The chain rule** connects quote risk to node risk through the curve builder's Jacobian: $\frac{dV_0}{dq} = \nabla_x V_0 \cdot J$

2. **Par-point deltas** (bump-and-rebuild) are simple but depend critically on interpolation choices. Poor locality causes curve-fit artifacts.

3. **The see-saw effect** shows that a 1 bp swap bump can translate to tens of basis points in forward space at the long end—a mathematical consequence of how swap rates relate to forward rates.

4. **The Jacobian method** decouples risk calculation from curve construction, enabling efficient computation and explicit analysis of how bumps propagate.

5. **Controlled perturbations** shock the forward curve directly with economically meaningful shapes, avoiding quote-bump artifacts.

6. **Multi-curve decomposition** separates overall level risk, discounting risk, and basis risk when curves are built in a spread-based framework.

7. **P&L predict** is the ultimate test: your deltas should explain your daily P&L. P&L explain attributes observed P&L to specific market moves.

8. **Implementation considerations** include numerical stability (central differences, fixed seeds), curve overlays for special dates, and computational efficiency through Jacobian caching.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Par-point delta** | PV change from 1 bp bump to one quote, after curve rebuild | Standard risk measure aligned with hedging instruments |
| **Locality** | Property that a local quote bump produces a local curve change | Prevents spurious risk attribution to distant tenors |
| **See-saw effect** | Large forward-rate oscillations from small quote bumps | Artifact of curve construction that distorts risk reports |
| **Jacobian** | Matrix $J = \partial x / \partial q$ mapping quotes to curve nodes | Enables efficient risk computation via chain rule |
| **Gâteaux derivative** | Directional derivative in function space | Defines controlled perturbation sensitivities |
| **Controlled perturbation** | Direct forward-curve bump with specified shape | Avoids curve-fit artifacts, aligns with bucket hedging |
| **Orthogonal decomposition** | Separation of level, discounting, and basis risk | Enables proper aggregation and hedging |
| **P&L predict** | Ability of computed deltas to explain realized P&L | Ultimate validation of risk methodology |
| **P&L explain** | Attribution of realized P&L to specific market moves | Diagnostic tool for understanding P&L drivers |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $V_0$ | Portfolio present value |
| $q = (q_1, \ldots, q_N)$ | Vector of benchmark quotes |
| $x = (x_1, \ldots, x_M)$ | Vector of curve nodes/parameters |
| $B(\cdot)$ | Curve builder function: $x = B(q)$ |
| $J = \frac{\partial x}{\partial q}$ | Jacobian matrix of curve builder |
| $\nabla_x V_0$ | Node sensitivity vector |
| $f(t)$ | Instantaneous forward rate curve |
| $\mu_k(t)$ | Basis function for controlled perturbation |
| $\partial_k V_0$ | Gâteaux derivative w.r.t. shift $\mu_k$ |
| $\delta$ | Bump size (typically 1 bp = $10^{-4}$) or market move vector |
| $\Theta_{\text{mkt}}$ | Vector of market data |
| $\nabla^H$ | Sensitivity vector w.r.t. market data |
| $A^H$ | Hessian matrix of second derivatives |

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

---

## Mini Problem Set

### Questions

**1. Chain Rule Application**
A portfolio has node sensitivities $\frac{\partial V_0}{\partial y_5} = -50{,}000$ (USD per unit 5Y zero rate). The Jacobian entry $\frac{\partial y_5}{\partial q_{5Y}} = 0.98$ (the 5Y zero rate moves 0.98 units per unit bump to the 5Y swap rate). What is the 5Y par-point delta in USD per bp?

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
Yesterday's risk report showed 5Y delta = +$8,000 per bp. Overnight, 5Y swaps moved −3 bp. Predicted P&L = −$24,000. Actual P&L = −$31,000. List three possible explanations for the discrepancy.

**8. Two-Curve Pricing**
If a desk uses a smooth cubic spline curve for pricing but a bootstrapped curve for risk, what problem might arise? How would you diagnose it?

**9. Cumulative vs Standard Par-Point**
Why does the cumulative par-point approach produce less extreme forward curve shifts than the standard approach? In what sense is the cumulative approach more "realistic"?

**10. Waterfall vs Bump-and-Reset**
A portfolio has significant gamma (second-order sensitivity). Which P&L explain method—waterfall or bump-and-reset—will produce a smaller unexplained residual? Why?

### Solution Sketches

**1.** Delta = $\frac{\partial V_0}{\partial y_5} \cdot \frac{\partial y_5}{\partial q_{5Y}} \cdot 10^{-4}$ = $(-50{,}000)(0.98)(10^{-4})$ = −$4.90 per bp.

**2.** $\Delta L_{10} \approx 10 \times 1 = +10$ bp. $\Delta L_{11} \approx 11 \times 0 - 10 \times 1 = -10$ bp. The 20 bp swing between adjacent forwards is the see-saw effect.

**3.** Bootstrapping with piecewise-linear yields makes each zero rate depend only on quotes up to that maturity—bumping a short quote doesn't affect long rates. Cubic splines enforce global smoothness constraints, so every quote bump potentially affects every forward rate.

**4.** $J$ is $60 \times 20$ (rows = number of nodes, columns = number of quotes).

**5.** $\mu_3(t) = 1$ for $t \in [2Y, 3Y)$, 0 elsewhere. The sensitivity $-12{,}500$ means if all forwards between 2Y and 3Y rise by 1 bp, the portfolio loses $12,500.

**6.** (i) Level risk: bump 5Y SOFR swap rate, (ii) Discounting risk: bump 5Y OIS rate holding SOFR fixed, (iii) Basis risk: bump SOFR-OIS basis spread. The receiver swap has mainly level risk (duration to SOFR curve) and modest discounting risk.

**7.** (i) Convexity: the move was −3 bp, large enough for second-order effects. (ii) Missing risk factors: perhaps basis or curve shape moved. (iii) Stale Jacobian: delta was computed with yesterday's curve shape. (iv) Theta: overnight time decay not accounted for.

**8.** The deltas explain PV changes in the bootstrapped curve, but actual MTM uses the cubic spline. If quotes move and the two curves respond differently, predicted P&L won't match actual. Diagnose by computing P&L predict ratio over time; it should be close to 1 with small scatter.

**9.** In the cumulative approach, when computing the delta to quote $i$, quotes $1, \ldots, i-1$ have already been bumped. This means the forward curve has already shifted in a parallel-like fashion, so the marginal bump to quote $i$ produces a smaller incremental forward move. This is more realistic because in practice, if the 30Y rate moves, neighboring rates typically move too.

**10.** Waterfall produces zero unexplained residual by construction (it's an identity). Bump-and-reset leaves a residual proportional to cross-gamma terms. However, waterfall's attribution depends on the arbitrary ordering of market variables.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| Par-point approach: bump → rebuild → reprice | Andersen Vol 1 Ch 6, Section 6.4.1 |
| Locality requirement for clean perturbations | Andersen Vol 1 Ch 6, Section 6.4.1 |
| See-saw effect: "30 year swap yield shifted by 1 bp... $L_{30}$ will move by 30 bp" | Andersen Vol 1 Ch 6, below Eq. 6.33 |
| $L_n \approx n S_n - (n-1) S_{n-1}$ approximation | Andersen Vol 1 Ch 6, Eq. 6.33 |
| Jacobian method decouples risk from curve construction | Andersen Vol 1 Ch 6, Section 6.4.3 |
| Gâteaux derivatives for controlled perturbations | Andersen Vol 1 Ch 6, Eq. 6.26 |
| Triangular and flat basis functions | Andersen Vol 1 Ch 6, Eqs. 6.27–6.28 |
| Cumulative par-point approach description | Andersen Vol 1 Ch 6, below Eq. 6.33 |
| Cumulative deltas sum to parallel delta | Andersen Vol 1 Ch 6, below Eq. 6.34 |
| Orthogonal risk decomposition in spread-based construction | Andersen Vol 1 Ch 6, Section 6.5.3 |
| Tension splines improve locality | Andersen Vol 1 Ch 6, Fig. 6.8 discussion |
| Poor P&L predict from inconsistent pricing/risk curves | Andersen Vol 1 Ch 6, Section 6.4.4 |
| P&L predict formulation with Taylor expansion | Andersen Vol 2 Ch 22, Eq. 22.19 |
| Theta calculation using forward values | Andersen Vol 2 Ch 22, Eqs. 22.20–22.21 |
| Waterfall P&L explain | Andersen Vol 2 Ch 22, Section 22.2.2.1 |
| Bump-and-reset P&L explain | Andersen Vol 2 Ch 22, Section 22.2.2.2 |
| Key-rate shifts sum to parallel shift | Tuckman Ch 7 |
| Key-rate methodology for hedging | Tuckman Ch 7 |
| Bucket exposures for forward-rate segments | Tuckman Ch 7 |

### (B) Reasoned Inference (Derived from A)

| Inference | Derivation |
|-----------|------------|
| Chain rule $dV_0/dq = (\partial V_0/\partial x)(\partial x/\partial q)$ | Standard multivariable calculus applied to curve builder |
| $L_{n+1}$ shifts by $-n\delta$ when $S_n$ bumps $\delta$ with neighbors fixed | Direct algebra from $L_n \approx n S_n - (n-1) S_{n-1}$ |
| Central differences improve numerical stability | Standard numerical analysis (symmetric error cancellation) |
| Jacobian has block-triangular structure for bootstrapping | Follows from sequential nature of bootstrap algorithm |

### (C) Flagged Uncertainties

| Item | Uncertainty |
|------|-------------|
| Specific desk bump conventions (size, one-sided vs central) | Varies by institution; 1 bp is common but not universal |
| Optimal tension parameters for splines | Implementation and data-dependent; no universal formula |
| Turn-of-year and event-date curve overlays | Market-specific; overlay specification varies by desk |
| Ordering of market variables in waterfall explain | Arbitrary choice affects attribution; no canonical ordering |

---

*Chapter 22 — v3.0*
