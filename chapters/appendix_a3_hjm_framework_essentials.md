# Appendix A3: HJM Framework Essentials (Study Notes)

---

## Conventions & Notation

- Time is continuous. $t \geq 0$ denotes "today/time of valuation," and $T > t$ denotes a maturity date, both measured in years.

- Rates are continuously compounded instantaneous rates unless explicitly stated otherwise.

- A **zero-coupon bond (ZCB)** maturing at $T$ pays 1 unit of currency at $T$; its time-$t$ price is $P(t,T)$.

- The **short rate** $r(t)$ is the instantaneous risk-free rate; the **money-market account / bank account** $B(t)$ is defined by

$$\frac{dB(t)}{B(t)} = r(t)\,dt, \quad B(0) = 1, \quad B(t) = \exp\!\left(\int_0^t r(s)\,ds\right).$$

- **Discount factor** between $t$ and $T$:

$$D(t,T) = \frac{B(t)}{B(T)} = \exp\!\left(-\int_t^T r(s)\,ds\right).$$

- **Measures / numeraires** (change-of-numeraire toolkit):

  - $\mathbb{Q}^B$: risk-neutral (money-market) measure with numeraire $B(t)$. Under $\mathbb{Q}^B$, any traded asset price divided by $B(t)$ is a martingale (under mild conditions).

  - $\mathbb{Q}^T$: $T$-forward measure with numeraire $P(t,T)$; pricing a payoff $H_T$ at $T$ becomes $\pi_t = P(t,T)\,\mathbb{E}^T[H_T \mid \mathcal{F}_t]$.

  - $\mathbb{Q}^{\alpha,\beta}$: swap measure with numeraire equal to the swap annuity $A_{\alpha,\beta}(t) = \sum_{i=\alpha+1}^{\beta} \tau_i P(t,T_i)$.

- **Brownian motion:** Under a chosen pricing measure, uncertainty is driven by a $d$-dimensional Brownian motion $W_t = (W_t^{(1)}, \ldots, W_t^{(d)})$.

- **Vector volatility notation:** $\sigma(t,T) \in \mathbb{R}^d$. Inner product $a \cdot b$ and norm $\|a\|^2 = a \cdot a$.

---

## 0. Setup

### Conventions used in this appendix

- Continuous-time, frictionless, arbitrage-free modeling.
- Continuously compounded instantaneous rates.
- HJM dynamics stated primarily under the money-market numeraire ($\mathbb{Q}^B$), because the drift restriction is most commonly expressed there in the sources.
- Multi-factor (vector) Brownian drivers are allowed; one-factor is a special case.

### Notation glossary (symbols + definitions)

| Symbol | Definition |
|--------|------------|
| $t$ | valuation time (years) |
| $T$ | maturity time (years), with $T > t$ |
| $P(t,T)$ | time-$t$ price of a ZCB paying 1 at $T$ |
| $r(t)$ | short rate (per year) |
| $B(t)$ | money-market account numeraire; $dB/B = r\,dt$ |
| $D(t,T)$ | discount factor $= \exp(-\int_t^T r(s)\,ds)$ |
| $f(t,T)$ | instantaneous forward rate for maturity $T$ observed at $t$ |
| $W_t$ | $d$-dimensional Brownian motion under the chosen pricing measure |
| $\sigma(t,T)$ | instantaneous volatility of $f(t,T)$ (vector in $\mathbb{R}^d$) |
| $\alpha(t,T)$ | drift of $f(t,T)$ under the stated measure |
| $\mathbb{Q}^B$ | bank-account (risk-neutral) measure |
| $\mathbb{Q}^T$ | $T$-forward measure with numeraire $P(t,T)$ |

---

## 1. Core Concepts (Definitions First)

### 1.1 Zero-coupon bond price $P(t,T)$

**Formal definition:** $P(t,T)$ is the time-$t$ price of a claim paying 1 at $T$.

**Intuition:** It is the primitive "discount factor" object in fixed income; all curve quantities (spots/forwards) can be derived from $P(t,T)$.

**In practice:** Market curves are built from liquid instruments; $P(0,T)$ is implied by the calibrated curve and used for pricing PVs and carry/roll calculations.

---

### 1.2 Short rate $r(t)$ and money-market account $B(t)$

**Formal definition:**

$$\frac{dB(t)}{B(t)} = r(t)\,dt, \quad B(0) = 1, \quad B(t) = \exp\!\left(\int_0^t r(s)\,ds\right).$$

**Intuition:** $B(t)$ is "rolling overnight" continuously; discounting by $B(t)$ is the canonical route to the risk-neutral measure.

**In practice:** Used as numeraire for $\mathbb{Q}^B$ pricing: discounted traded prices are martingales under $\mathbb{Q}^B$.

---

### 1.3 Instantaneous forward rate $f(t,T)$ and its link to $P(t,T)$

**Formal definition (instantaneous forward):**

$$f(t,T) = -\frac{\partial}{\partial T} \ln P(t,T),$$

assuming sufficient smoothness in $T$.

**Bond–forward relation:**

$$P(t,T) = \exp\!\left(-\int_t^T f(t,u)\,du\right).$$

**Unit check:** $\ln P$ is dimensionless; $\partial_T \ln P$ has units $1/\text{year}$, so $f(t,T)$ is a rate "per year".

**In practice:** $f(0,T)$ is the forward curve (useful for interpreting steepness/humps); many hedges are "bucketed" by forward-rate key points.

---

### 1.4 HJM state variable: the entire forward curve $f(t,\cdot)$

**Formal definition:** The HJM "state" at time $t$ is the function $T \mapsto f(t,T)$ for all maturities $T \geq t$. The model specifies consistent dynamics for this curve across all $T$.

**Intuition:** Unlike short-rate models (finite-dimensional state), HJM's natural object is the whole term structure; hence it is infinite-dimensional in general and "too unwieldy" without additional structure.

**In practice:** Curves are represented on a tenor grid; HJM is implemented in discretized form (see Section 7).

---

### 1.5 Volatility specification $\sigma(t,T)$

**Formal definition:** In HJM, for each maturity $T$, $f(t,T)$ follows a diffusion with instantaneous volatility $\sigma(t,T)$ (possibly vector-valued for multi-factor models).

**Intuition:** $\sigma(t,T)$ describes how each point of the forward curve responds to Brownian shocks; it is the "free input" of the framework.

**In practice:** Parametric choices (separable/exponential-decay/multi-factor) are used to fit option-implied vol surfaces and obtain tractable simulation/pricing.

---

### 1.6 No-arbitrage principle in HJM: discounted bond prices are martingales under $\mathbb{Q}^B$

**Formal statement:** Under the measure associated with numeraire $B(t)$, any traded price divided by $B(t)$ is a martingale (Fact One).

**Applied to bonds:** $P(t,T)/B(t)$ must be a martingale under $\mathbb{Q}^B$. This pins down the bond drift, which in turn pins down the forward-rate drift.

**In practice:** This is the core "no-arbitrage drift restriction" that prevents you from choosing both drift and volatility freely.

---

### 1.7 "Drift is determined by volatility"

**Formal meaning (HJM):** If you specify forward-rate volatility $\sigma(t,T)$, then the forward-rate drift $\alpha(t,T)$ is not free: it is uniquely determined (under the stated measure/numeraire conventions) by a no-arbitrage restriction.

**Intuition:** Arbitrage-free evolution forces the drift to "compensate" for convexity effects induced by volatility.

**In practice:** Calibration "chooses" $\sigma$; then drift is computed mechanically. Mistakes here create arbitrage-like artifacts in Monte Carlo.

---

## 2. HJM Setup: From Bond Prices to Forward-Rate Dynamics

### 2.1 Start from $P(t,T)$ and define $f(t,T)$

Given $P(t,T)$, define the instantaneous forward:

$$f(t,T) = -\frac{\partial}{\partial T} \ln P(t,T).$$

Conversely, given $f(t,\cdot)$, recover bond prices:

$$P(t,T) = \exp\!\left(-\int_t^T f(t,u)\,du\right).$$

**Units check:**
- $P(t,T) \in (0,1]$ dimensionless (price per unit face).
- $f(t,T)$ has units $1/\text{year}$.

---

### 2.2 Why model $f(t,T)$ directly?

**Curve fit "by construction":** If the model is initialized with the observed curve $P(0,T)$ (or $f(0,T)$), HJM dynamics can be written so that the time-0 curve is matched exactly (the initial curve is an input condition).

**Modeling convenience:** In interest-rate derivatives, payoffs typically depend on multiple maturities. Modeling the entire curve gives a consistent joint evolution (though infinite-dimensional unless restricted).

---

### 2.3 Risk-neutral (money-market) vs $T$-forward measure: conceptual difference

**Under $\mathbb{Q}^B$ (numeraire $B(t)$):**
- Pricing uses discounting by the money-market account; traded prices divided by $B(t)$ are martingales.

**Under $\mathbb{Q}^T$ (numeraire $P(t,T)$):**
- If a payoff is paid at $T$, pricing simplifies to

$$\pi_t = P(t,T)\,\mathbb{E}^T[H_T \mid \mathcal{F}_t].$$

- Certain forward objects (built as ratios of traded assets to $P(t,T)$) become martingales under $\mathbb{Q}^T$. For example, forward bond prices $P(t,T+\tau)/P(t,T)$ are martingales under $\mathbb{Q}^T$.

---

## 3. The No-Arbitrage Drift Restriction (The Centerpiece)

### 3.1 General HJM forward-rate SDE form

Under HJM, assume that for each maturity $T$ the instantaneous forward rate follows

$$df(t,T) = \alpha(t,T)\,dt + \sigma(t,T) \cdot dW_t,$$

where $W_t$ is a $d$-dimensional Brownian motion and $\sigma(t,T) \in \mathbb{R}^d$.

Also, the short rate is the "diagonal" of the forward curve:

$$r(t) = f(t,t).$$

---

### 3.2 Bond price dynamics induced by forward dynamics

Using $P(t,T) = \exp(-\int_t^T f(t,u)\,du)$, Itô calculus yields the bond dynamics

$$\frac{dP(t,T)}{P(t,T)} = \left(r(t) - \int_t^T \alpha(t,u)\,du + \frac{1}{2}\left\|\int_t^T \sigma(t,u)\,du\right\|^2\right)dt - \left(\int_t^T \sigma(t,u)\,du\right) \cdot dW_t.$$

**Interpretation of the volatility term:**

The bond's instantaneous diffusion coefficient is the integrated forward vol:

$$\sigma_P(t,T) = -\int_t^T \sigma(t,u)\,du$$

(up to sign conventions), consistent with the idea that a bond aggregates shocks to all forwards between $t$ and $T$.

---

### 3.3 No-arbitrage condition and drift restriction (money-market measure)

**State the measure and numeraire:** Work under the risk-neutral/money-market measure $\mathbb{Q}^B$ with numeraire $B(t)$. Under $\mathbb{Q}^B$, the discounted bond price $P(t,T)/B(t)$ must be a martingale (Fact One).

**Martingale condition ⇒ bond drift equals $r(t)$:** For a traded bond under $\mathbb{Q}^B$, its (undiscounted) drift must be $r(t)$. In the bond SDE above, this means the bracketed drift term must equal $r(t)$:

$$r(t) - \int_t^T \alpha(t,u)\,du + \frac{1}{2}\left\|\int_t^T \sigma(t,u)\,du\right\|^2 = r(t).$$

Cancelling $r(t)$ gives the restriction

$$-\int_t^T \alpha(t,u)\,du + \frac{1}{2}\left\|\int_t^T \sigma(t,u)\,du\right\|^2 = 0.$$

Differentiate w.r.t. $T$ (informally; made rigorous under regularity assumptions) to obtain the **HJM drift restriction:**

$$\boxed{\alpha(t,T) = \sigma(t,T) \cdot \int_t^T \sigma(t,u)\,du}$$

**Multi-factor expansion:**

If $\sigma(t,T) = (\sigma_1(t,T), \ldots, \sigma_d(t,T))$, then

$$\alpha(t,T) = \sum_{i=1}^{d} \sigma_i(t,T) \int_t^T \sigma_i(t,u)\,du,$$

which is exactly the dot product form above.

---

### 3.4 "Drift determined by volatility" made explicit

Plugging the drift restriction back into the forward-rate SDE yields the integrated HJM representation:

$$df(t,T) = \sigma(t,T) \cdot \left[\left(\int_t^T \sigma(t,u)\,du\right)dt + dW_t\right],$$

showing the drift is fully implied once $\sigma$ is specified.

A one-factor version is stated directly as an HJM definition in the sources:

$$df(t,T) = \sigma_f(t,T)\left[\left(\int_t^T \sigma_f(t,u)\,du\right)dt + dW(t)\right],$$

under the risk-neutral measure (here $W$ is one-dimensional).

---

### 3.5 Checks (units and limiting cases)

**Limiting case $\sigma \equiv 0$:** then $\alpha(t,T) = 0$ from $\alpha = \sigma \cdot \int\sigma$, so forward rates have deterministic evolution (no diffusion-driven convexity correction). This matches the idea that without randomness, no-arbitrage places no extra drift constraints beyond deterministic consistency.

**Dimensional sanity:**
- $dW$ scales like $\sqrt{dt}$.
- If $f$ has units $1/\text{year}$, then $\sigma$ has units "rate per $\sqrt{\text{year}}$" and $\alpha$ has units "rate per year". The product $\sigma \cdot \int_t^T \sigma\,du$ indeed has the same time scaling as a drift.

---

## 4. Measures Commonly Used with HJM (Risk-Neutral vs Forward Measures)

### 4.1 Why the $T$-forward measure is natural for payoffs at $T$

If a claim pays $H_T$ at time $T$, choosing the ZCB numeraire $P(t,T)$ yields the pricing identity

$$\pi_t = P(t,T)\,\mathbb{E}^T[H_T \mid \mathcal{F}_t],$$

because $P(T,T) = 1$.

**Practitioner translation:** you "factor out" discounting and work under a measure where the remaining object is just an expectation of the payoff.

---

### 4.2 How drifts change under measure change (explicit, source-backed)

The change-of-numeraire toolkit provides an explicit drift transformation for diffusion processes when switching from numeraire $S$ to $U$. If under the $S$-numeraire measure a diffusion $X$ has drift $\mu^S$, then under the $U$-numeraire measure its drift is

$$\mu^U(t) = \mu^S(t) - \sigma_X(t)\,\rho\left(\frac{\sigma_S(t)}{S(t)} - \frac{\sigma_U(t)}{U(t)}\right)^\top,$$

and the Brownian motions are related by

$$dW^U(t) = dW^S(t) - \rho\left(\frac{\sigma_U(t)}{U(t)}\right)^\top dt.$$

**Important caution:** applying this to $f(t,T)$ directly requires interpreting $f(t,T)$ as (or as a function of) traded assets in a way consistent with the model setup. The sources explicitly state martingale properties for traded ratios (e.g., forward bond prices, forward LIBOR rates) under forward measures.

---

### 4.3 Swap measure (if used)

The swap measure uses an annuity numeraire:

$$A_{\alpha,\beta}(t) = \sum_{i=\alpha+1}^{\beta} \tau_i P(t,T_i),$$

and under the associated measure the forward swap rate $S_{\alpha,\beta}(t)$ is a martingale.

**When practitioners use it:** swaption pricing/hedging, because the swap rate is the natural underlying and becomes driftless under the swap-annuity numeraire.

---

### 4.4 "When to use which measure" (practitioner summary)

- **Use $\mathbb{Q}^B$** when you want a single "global" measure tied to the short rate/money-market account, and you are enforcing the HJM drift restriction in its standard form.

- **Use $\mathbb{Q}^T$** when the payoff is at a single maturity $T$: pricing becomes $P(t,T) \times$ an expectation of the payoff. Forward bond prices are martingales under $\mathbb{Q}^T$.

- **Use swap measure** for swap-rate payoffs (swaptions), because the swap rate becomes a martingale under the annuity numeraire.

---

## 5. Choosing a Volatility Structure: What You Can Choose Freely and What You Can't

### 5.1 What you can choose

In HJM you choose $\sigma(t,T)$ (subject to model regularity/integrability).

Under the money-market measure, no-arbitrage determines

$$\alpha(t,T) = \sigma(t,T) \cdot \int_t^T \sigma(t,u)\,du.$$

So you cannot choose $\alpha$ and $\sigma$ independently.

---

### 5.2 Common $\sigma(t,T)$ shapes supported in the sources

**(i) Separable (Carverhill) structure**

A deterministic separable class is

$$\sigma_i(t,T) = \xi_i(t)\,\psi_i(T), \quad i = 1, \ldots, N.$$

In the one-factor case, this implies a Hull–White-type short-rate model (finite-dimensional Markov) with an explicit short-rate SDE derived from the separability condition.

**(ii) Exponential decay in time-to-maturity (Hull–White-type forward vol)**

A specific one-factor form highlighted is

$$\sigma(t,T) = \sigma\,e^{-a(T-t)},$$

which leads to a Hull–White-equivalent dynamics in the one-factor setting discussed (mean-reversion produces exponential decay across maturities).

**(iii) Ritchken–Sankarasubramanian (RS) / finite-dimensional Markov extension**

A volatility class of the form

$$\sigma(t,T) = \sigma(t,t)\exp\!\left(-\int_t^T a(t,u)\,du\right)$$

leads to a Markov representation with an additional state variable (helpful for trees / efficient simulation).

---

### 5.3 Practical implications (qualitative, source-aligned)

- The general HJM class is infinite-dimensional and "too unwieldy" in practice; hence the need for structured volatility families that reduce to finitely many Markov state variables.

- Volatility structure matters because it drives both:
  - option prices (directly via $\sigma$),
  - and forward drifts (indirectly via the HJM restriction), affecting curve evolution and convexity corrections.

---

## 6. Relationship to Familiar Models (Bridge Section)

### 6.1 Short-rate models induce an HJM volatility (conceptual equivalence)

If you model $r(t)$ (and hence $B(t)$), you can compute $P(t,T)$ and therefore $f(t,T)$. The induced forward-rate dynamics must be arbitrage-free and therefore satisfy an HJM drift restriction under the appropriate pricing measure.

This bridge is explicit in the sources via:
- separable HJM vol $\Rightarrow$ Hull–White-type short-rate dynamics (one-factor Carverhill condition).

---

### 6.2 Concrete bridge example: a simple short-rate diffusion produces HJM-consistent forward drift

A toy short-rate model (Merton-style) with

$$dr(t) = \theta(t)\,dt + \sigma\,dW_t$$

induces a forward-rate dynamic

$$df(t,T) = \sigma^2(T-t)\,dt + \sigma\,dW_t,$$

so the forward drift equals $\sigma^2(T-t)$, which is exactly the HJM drift restriction for constant $\sigma(t,T) = \sigma$.

This is the cleanest "bridge" statement: a short-rate model implies a forward volatility, which then implies a forward drift via HJM.

---

## 7. Practical Implementation Map (Discretization of an Infinite-Dimensional Model)

### 7.1 Discretize the forward curve on a tenor grid

- Choose maturities $T_1 < T_2 < \cdots < T_M$.
- Represent the state as the vector $\mathbf{f}_t = (f(t,T_1), \ldots, f(t,T_M))$.
- Approximate integrals like $\int_t^{T_j} \sigma(t,u)\,du$ with quadrature on the grid (piecewise-constant/trapezoid).

---

### 7.2 Simulate discretized HJM (Euler idea)

On each step $t \to t + \Delta t$, update each $f(t,T_j)$ via

$$f(t+\Delta t, T_j) \approx f(t,T_j) + \alpha(t,T_j)\Delta t + \sigma(t,T_j) \cdot \Delta W_t,$$

where $\Delta W_t \sim N(0, \Delta t\,I_d)$.

**No-arbitrage numerically:** recompute $\alpha(t,T_j)$ from the current $\sigma$ using the drift restriction at each step:

$$\alpha(t,T_j) = \sigma(t,T_j) \cdot \int_t^{T_j} \sigma(t,u)\,du.$$

---

### 7.3 Reconstruct bond prices from the simulated forward curve

Use $P(t,T) = \exp(-\int_t^T f(t,u)\,du)$, approximating the integral on the tenor grid.

---

### 7.4 Calibration workflow (high level)

1. Choose a parametric $\sigma(t,T;\vartheta)$ (separable/exponential decay/multi-factor).
2. Fit $\vartheta$ to option-implied data (caps/swaptions) and ensure the model is consistent with the initial curve. The need for structured subclasses is emphasized because unconstrained HJM is not practical to calibrate and run.
3. Validate: reprice calibration instruments; check stability of simulated curves.

---

## 8. Math and Derivations (Step-by-Step)

### 8.1 Derive $f(t,T)$ from $P(t,T)$

Starting from the definition of the instantaneous forward rate:

$$f(t,T) = -\frac{\partial}{\partial T} \ln P(t,T),$$

assuming the bond-price term structure is sufficiently smooth in $T$.

**Unit check:** $\ln P$ is dimensionless; $\partial_T \ln P$ has units $1/\text{year}$, consistent with $f$.

---

### 8.2 Derive $P(t,T)$ from $f(t,\cdot)$

Integrate the definition:

$$\ln P(t,T) - \ln P(t,t) = -\int_t^T f(t,u)\,du.$$

Since $P(t,t) = 1$, $\ln P(t,t) = 0$, hence

$$P(t,T) = \exp\!\left(-\int_t^T f(t,u)\,du\right).$$

---

### 8.3 Bond price dynamics from forward dynamics (key Itô step)

Assume the HJM forward dynamics

$$df(t,T) = \alpha(t,T)\,dt + \sigma(t,T) \cdot dW_t.$$

Let

$$X(t,T) := \int_t^T f(t,u)\,du, \quad \ln P(t,T) = -X(t,T).$$

Then (sketch of Itô logic):

- The lower limit $t$ contributes a term $-f(t,t)\,dt = -r(t)\,dt$.
- The stochastic part contributes $-\int_t^T \sigma(t,u)\,du \cdot dW_t$.
- The quadratic variation contributes $+\frac{1}{2}\|\int_t^T \sigma(t,u)\,du\|^2\,dt$.

Putting these together yields:

$$\frac{dP(t,T)}{P(t,T)} = \left(r(t) - \int_t^T \alpha(t,u)\,du + \frac{1}{2}\left\|\int_t^T \sigma(t,u)\,du\right\|^2\right)dt - \left(\int_t^T \sigma(t,u)\,du\right) \cdot dW_t.$$

---

### 8.4 No-arbitrage drift restriction derivation

Under $\mathbb{Q}^B$, discounted traded prices are martingales; for the bond this implies its drift is $r(t)$.

From the bond SDE above, set the drift equal to $r(t)$:

$$r(t) - \int_t^T \alpha(t,u)\,du + \frac{1}{2}\left\|\int_t^T \sigma(t,u)\,du\right\|^2 = r(t).$$

Cancel $r(t)$:

$$\int_t^T \alpha(t,u)\,du = \frac{1}{2}\left\|\int_t^T \sigma(t,u)\,du\right\|^2.$$

Differentiate in $T$ (regularity assumed) to obtain:

$$\boxed{\alpha(t,T) = \sigma(t,T) \cdot \int_t^T \sigma(t,u)\,du.}$$

---

### 8.5 Drift transformation under measure change (general diffusion result)

For a diffusion driven by Brownian motion, switching from numeraire $S$ to $U$ changes the drift according to Proposition 2.3.1:

$$\mu^U(t) = \mu^S(t) - \sigma_X(t)\,\rho\left(\frac{\sigma_S(t)}{S(t)} - \frac{\sigma_U(t)}{U(t)}\right)^\top,$$

and Brownian increments satisfy

$$dW^U(t) = dW^S(t) - \rho\left(\frac{\sigma_U(t)}{U(t)}\right)^\top dt.$$

---

## 9. Worked Examples (At Least 10 Numeric Examples)

> **Note:** Examples use toy numbers and simple approximations. The goal is mechanics and sanity checks, not market realism.

---

### Example 1: Compute $f(0,T)$ from a discount curve $P(0,T)$ (finite-difference)

Suppose the time-0 discount factors are:

| $T$ (years) | $P(0,T)$ | $\ln P(0,T)$ (given) |
|-------------|----------|----------------------|
| 0.5 | 0.99005 | $-0.010$ |
| 1.5 | 0.96464 | $-0.036$ |
| 2.5 | 0.93239 | $-0.070$ |

We use the definition $f(0,T) = -\partial_T \ln P(0,T)$ and approximate the derivative by a central difference.

**At $T = 1.0$ with step $h = 0.5$:**

$$f(0,1) \approx -\frac{\ln P(0,1.5) - \ln P(0,0.5)}{1.5 - 0.5} = -\frac{-0.036 - (-0.010)}{1.0} = 0.026.$$

**Answer:** $f(0,1) \approx 2.6\%$.

**At $T = 2.0$ using $(1.5, 2.5)$:**

$$f(0,2) \approx -\frac{\ln P(0,2.5) - \ln P(0,1.5)}{2.5 - 1.5} = -\frac{-0.070 - (-0.036)}{1.0} = 0.034.$$

**Answer:** $f(0,2) \approx 3.4\%$.

---

### Example 2: Given $\sigma(t,T)$, compute implied $\alpha(t,T)$ using HJM drift restriction

Assume one-factor HJM with constant forward volatility:

$$\sigma(t,T) = 0.01 \quad (\text{constant}).$$

Then

$$\alpha(t,T) = \sigma(t,T) \int_t^T \sigma(t,u)\,du = 0.01 \int_t^T 0.01\,du = 0.0001(T-t).$$

**Compute at selected points:**

| Point | Value |
|-------|-------|
| $t=0$, $T=2$ | $\alpha = 0.0001 \cdot 2 = 0.0002$ |
| $t=0.5$, $T=2$ | $\alpha = 0.0001 \cdot 1.5 = 0.00015$ |
| $t=1$, $T=3$ | $\alpha = 0.0001 \cdot 2 = 0.0002$ |

---

### Example 3: Limiting case $\sigma = 0$

If $\sigma(t,T) = 0$ for all $(t,T)$, then

$$\alpha(t,T) = \sigma(t,T) \int_t^T \sigma(t,u)\,du = 0.$$

So the forward curve evolves deterministically (no Brownian shock and no volatility-induced drift correction).

---

### Example 4: One-factor vs two-factor $\sigma$: how the drift changes (inner product)

Let $d = 2$ factors with constant volatilities:

$$\sigma(t,T) = (0.01, 0.015).$$

Then

$$\alpha(t,T) = \sum_{i=1}^{2} \sigma_i(t,T) \int_t^T \sigma_i(t,u)\,du = (0.01^2 + 0.015^2)(T-t).$$

**Compute:**
- $0.01^2 = 0.0001$
- $0.015^2 = 0.000225$
- Sum $= 0.000325$

At $t=0$, $T=2$: $\alpha = 0.000325 \cdot 2 = 0.00065$.

Compare to one-factor case with only $\sigma = 0.01$: $\alpha = 0.0002$. **Two factors produce a larger drift correction.**

---

### Example 5: $T$-forward measure intuition: price of 1 paid at $T$

**Payoff:** $H_T = 1$ at $T = 2$. Suppose $P(0,2) = 0.94933$.

**Under risk-neutral discounting:** $\pi_0 = P(0,2) = 0.94933$.

**Under $T$-forward measure:**

$$\pi_0 = P(0,2)\,\mathbb{E}^2[1] = P(0,2) \cdot 1 = 0.94933.$$

---

### Example 6: Drift change under measure change (toy application of Proposition 2.3.1)

Suppose a scalar diffusion $X$ has drift $\mu^B = 0.020$ under the bank-account numeraire $B$, and volatility $\sigma_X = 0.10$. Switch to a bond numeraire $U$ with percentage volatility $\sigma_U/U = 0.05$. Assume scalar case and correlation $\rho = 1$. Also take $\sigma_B/B = 0$ (bank account has no diffusion term).

Using

$$\mu^U = \mu^B - \sigma_X \rho\left(\frac{\sigma_B}{B} - \frac{\sigma_U}{U}\right),$$

we get

$$\mu^U = 0.020 - 0.10(0 - 0.05) = 0.020 + 0.005 = 0.025.$$

**Answer:** drift increases from $2.0\%$ to $2.5\%$ per year in the new measure in this toy setup.

---

### Example 7: "Vol choice impacts drift": compare two $\sigma$ shapes

Fix $t = 0$, $T = 2$ so $\Delta = T - t = 2$.

**Case A (flat):** $\sigma_A(0,u) = 0.01$. Then $\alpha_A(0,2) = 0.0001 \cdot 2 = 0.0002$.

**Case B (decaying in maturity):** $\sigma_B(0,u) = 0.01\,e^{-0.5u}$.

Then $\sigma_B(0,2) = 0.01\,e^{-1} \approx 0.0036788$, and

$$\int_0^2 \sigma_B(0,u)\,du = 0.01 \int_0^2 e^{-0.5u}\,du = 0.01 \cdot \frac{1 - e^{-1}}{0.5} = 0.02(1 - e^{-1}) \approx 0.0126424.$$

Hence

$$\alpha_B(0,2) = \sigma_B(0,2) \cdot \int_0^2 \sigma_B(0,u)\,du \approx 0.0036788 \cdot 0.0126424 \approx 0.0000465.$$

**Conclusion:** Different volatility term structures imply materially different HJM drifts.

---

### Example 8: One Euler step on a discretized forward curve

**Grid maturities:** $T_1 = 1$, $T_2 = 2$, $T_3 = 3$.

**Initial forwards at $t = 0$:**

| Maturity | Forward |
|----------|---------|
| $f(0,1)$ | $0.030$ |
| $f(0,2)$ | $0.032$ |
| $f(0,3)$ | $0.033$ |

Use constant $\sigma = 0.01$ (one-factor), so $\alpha(0,T) = 0.0001T$.

Choose $\Delta t = 0.25$ years, and a Brownian increment $\Delta W = 0.10$.

**Euler update:**

$$f(\Delta t, T) \approx f(0,T) + \alpha(0,T)\Delta t + \sigma\,\Delta W.$$

Compute $\sigma\,\Delta W = 0.01 \cdot 0.10 = 0.001$.

| $T$ | Drift contribution | Updated forward |
|-----|-------------------|-----------------|
| 1 | $0.0001 \cdot 1 \cdot 0.25 = 0.000025$ | $f(0.25,1) \approx 0.030 + 0.000025 + 0.001 = 0.031025$ |
| 2 | $0.0002 \cdot 0.25 = 0.000050$ | $f(0.25,2) \approx 0.032 + 0.000050 + 0.001 = 0.033050$ |
| 3 | $0.0003 \cdot 0.25 = 0.000075$ | $f(0.25,3) \approx 0.033 + 0.000075 + 0.001 = 0.034075$ |

---

### Example 9: Reconstruct $P(t,T)$ from the updated curve; check positivity/monotonicity

Approximate the integral $\int_t^T f(t,u)\,du$ by piecewise-constant forwards over:

- $[0.25, 1]$ length $0.75$ using $f(0.25,1) = 0.031025$
- $[1, 2]$ length $1$ using $f(0.25,2) = 0.033050$
- $[2, 3]$ length $1$ using $f(0.25,3) = 0.034075$

Then:

$$P(0.25,1) \approx \exp(-0.75 \cdot 0.031025) = \exp(-0.02326875) \approx 0.9770.$$

$$P(0.25,2) \approx \exp(-(0.75 \cdot 0.031025 + 1 \cdot 0.03305)) = \exp(-0.05631875) \approx 0.9452.$$

$$P(0.25,3) \approx \exp(-(0.75 \cdot 0.031025 + 0.03305 + 0.034075)) = \exp(-0.09039375) \approx 0.9136.$$

**Checks:**
- **Positivity:** all $P > 0$. ✓
- **Monotonicity in $T$:** $P(0.25,1) > P(0.25,2) > P(0.25,3)$. ✓

---

### Example 10: Bridge example (source-backed): Merton-style short-rate model ⇒ HJM drift

From the toy short-rate model $dr = \theta(t)\,dt + \sigma\,dW$, the induced forward dynamics is

$$df(t,T) = \sigma^2(T-t)\,dt + \sigma\,dW_t.$$

Choose $\sigma = 0.02$, $t = 1$, $T = 3$:

**HJM-implied drift:**

$$\alpha(t,T) = \sigma^2(T-t) = 0.02^2 \cdot 2 = 0.0004 \cdot 2 = 0.0008.$$

**HJM formula with constant $\sigma(t,T) = 0.02$ also gives:**

$$\alpha(t,T) = \sigma \int_t^T \sigma\,du = 0.02 \cdot 0.02 \cdot (3-1) = 0.0008.$$

**Match confirmed:** the short-rate model's forward drift equals the HJM drift restriction.

---

## 10. Practical Notes

### Common pitfalls

1. **Mixing measures:** Using the HJM drift restriction $\alpha = \sigma \cdot \int\sigma$ without confirming you are under the money-market measure (or the appropriate numeraire). Drift formulas are measure-dependent.

2. **Inconsistent $\sigma$ definition:** Confusing "forward-rate volatility" $\sigma(t,T)$ with "bond price volatility" (which involves $\int_t^T \sigma(t,u)\,du$).

3. **Numerical discretization artifacts:** If you discretize in $T$ and approximate $\int_t^T \sigma(t,u)\,du$ poorly, you can break the drift restriction numerically and generate arbitrage-like behavior.

4. **Calibrating $\sigma$ without checking forward-curve stability:** Even if option prices fit, the induced drift can produce unrealistic forward curve shapes over time.

---

### Verification tests

1. **Martingale check (conceptual/numeric):** Simulate many paths and verify that $\mathbb{E}[P(t,T)/B(t)]$ is (approximately) constant in $t$ under $\mathbb{Q}^B$.

2. **Positivity and monotonicity:** Confirm $P(t,T) > 0$ and typically decreasing in $T$ for each $t$ (violations often indicate discretization problems).

3. **Limiting case test:** As $\sigma \to 0$, verify drift correction $\alpha \to 0$ and the curve becomes deterministic.

4. **Stability test:** Small changes in $\sigma$ parameters should not explode curve evolution; if it does, reconsider the volatility family (e.g., use structured subclasses).

---

## 11. Summary & Recall

### 10-Bullet Executive Summary

1. A zero-coupon bond price $P(t,T)$ is the primitive discounting object in term structure modeling.

2. The money-market account $B(t)$ satisfies $dB/B = r\,dt$ and defines the risk-neutral numeraire.

3. Instantaneous forward rates satisfy $f(t,T) = -\partial_T \ln P(t,T)$ and $P(t,T) = \exp(-\int_t^T f(t,u)\,du)$.

4. HJM treats the whole curve $f(t,\cdot)$ as the state; the general class is infinite-dimensional and impractical without structure.

5. HJM posits $df(t,T) = \alpha(t,T)\,dt + \sigma(t,T) \cdot dW_t$ (multi-factor allowed).

6. Bond dynamics imply $\frac{dP}{P} = (r - \int\alpha + \frac{1}{2}\|\int\sigma\|^2)\,dt - (\int\sigma) \cdot dW$.

7. No-arbitrage under $\mathbb{Q}^B$ forces the bond drift to be $r(t)$, yielding the HJM drift restriction.

8. The drift restriction is $\alpha(t,T) = \sigma(t,T) \cdot \int_t^T \sigma(t,u)\,du$: drift is determined by volatility.

9. Forward and swap measures are numeraires $P(t,T)$ and annuities; they turn certain forward rates/swap rates into martingales, simplifying pricing.

10. Practical HJM requires structured $\sigma(t,T)$ families (separable/exponential decay/RS class) to obtain finite-dimensional Markov representations and stable numerics.

---

### Cheat Sheet (Objects + Formulas)

**Bond / forward link:**

$$f(t,T) = -\partial_T \ln P(t,T), \qquad P(t,T) = \exp\!\left(-\int_t^T f(t,u)\,du\right).$$

**Money-market account:**

$$dB(t) = r(t)B(t)\,dt, \qquad B(t) = \exp\!\left(\int_0^t r(s)\,ds\right).$$

**HJM forward dynamics:**

$$df(t,T) = \alpha(t,T)\,dt + \sigma(t,T) \cdot dW_t.$$

**Bond dynamics under HJM:**

$$\frac{dP(t,T)}{P(t,T)} = \left(r(t) - \int_t^T \alpha(t,u)\,du + \frac{1}{2}\left\|\int_t^T \sigma(t,u)\,du\right\|^2\right)dt - \left(\int_t^T \sigma(t,u)\,du\right) \cdot dW_t.$$

**HJM drift restriction (money-market measure):**

$$\boxed{\alpha(t,T) = \sigma(t,T) \cdot \int_t^T \sigma(t,u)\,du.}$$

**Pricing under $T$-forward measure:**

$$\pi_t = P(t,T)\,\mathbb{E}^T[H_T \mid \mathcal{F}_t].$$

---

### 25 Flashcards (Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What is $P(t,T)$? | Price at time $t$ of a ZCB paying 1 at $T$. |
| 2 | Define $B(t)$. | Money-market account with $dB/B = r\,dt$, $B = \exp(\int_0^t r\,ds)$. |
| 3 | What is the discount factor $D(t,T)$? | $D(t,T) = \exp(-\int_t^T r(s)\,ds)$. |
| 4 | Define instantaneous forward $f(t,T)$. | $f(t,T) = -\partial_T \ln P(t,T)$. |
| 5 | Express $P(t,T)$ via forwards. | $P(t,T) = \exp(-\int_t^T f(t,u)\,du)$. |
| 6 | What is the HJM state variable? | The whole forward curve $f(t,\cdot)$. |
| 7 | General HJM SDE form? | $df = \alpha\,dt + \sigma \cdot dW$. |
| 8 | Under $\mathbb{Q}^B$, what must $P(t,T)/B(t)$ be? | A martingale. |
| 9 | Bond volatility in HJM involves what integral? | $\int_t^T \sigma(t,u)\,du$. |
| 10 | HJM drift restriction (money-market)? | $\alpha(t,T) = \sigma(t,T) \cdot \int_t^T \sigma(t,u)\,du$. |
| 11 | What does "drift determined by volatility" mean? | Once $\sigma$ is chosen, no-arbitrage pins down $\alpha$. |
| 12 | Why use $T$-forward measure? | Payoffs at $T$ price as $P(t,T)\,\mathbb{E}^T[\text{payoff}]$. |
| 13 | Under $T$-forward measure, what is a martingale? | Forward bond prices $P(t,T+\tau)/P(t,T)$. |
| 14 | What is a swap measure numeraire? | Swap annuity $A_{\alpha,\beta}(t) = \sum \tau_i P(t,T_i)$. |
| 15 | What becomes a martingale under swap measure? | Forward swap rate $S_{\alpha,\beta}(t)$. |
| 16 | How do Brownian motions relate across numeraires? | $dW^U = dW^S - \rho(\sigma_U/U)^\top dt$. |
| 17 | Drift change across numeraires depends on what? | Covariation with the numeraire's volatility. |
| 18 | What does separable $\sigma(t,T) = \xi(t)\psi(T)$ buy you? | Hull–White-type (finite-dimensional Markov) short-rate model in one-factor case. |
| 19 | Example exponential-decay forward vol? | $\sigma(t,T) = \sigma\,e^{-a(T-t)}$. |
| 20 | Why isn't "general HJM" used directly? | It is infinite-dimensional and too unwieldy; subclasses are sought. |
| 21 | If $\sigma = 0$, what is $\alpha$? | $\alpha = 0$. |
| 22 | Relationship between short rate and forward curve? | $r(t) = f(t,t)$. |
| 23 | What is the "terminal forward measure" idea? | Use the longest maturity bond as numeraire to price multiple cashflows under one forward measure. |
| 24 | What is the HJM drift restriction enforcing? | No-arbitrage (martingale property of discounted bond prices). |
| 25 | What must be specified carefully in any drift formula? | The measure, numeraire, and the definition of $\sigma$ (forward vol vs bond vol). |

---

## 12. Mini Problem Set (16 Questions)

1. Given $P(0,1) = 0.975$ and $P(0,2) = 0.940$, compute the average continuously compounded forward rate over $[1,2]$: $-\ln(P(0,2)/P(0,1))$.

   > *Sketch:* compute ratio $P(0,2)/P(0,1)$, take $-\ln(\cdot)$ and divide by 1 year.

2. Use $f(t,T) = -\partial_T \ln P(t,T)$ to derive $P(t,T) = \exp(-\int_t^T f(t,u)\,du)$.

   > *Sketch:* integrate in $T$ and use $P(t,t) = 1$.

3. Under one-factor HJM with $\sigma(t,T) = \sigma_0$ constant, derive $\alpha(t,T)$.

   > *Sketch:* $\alpha = \sigma_0 \int_t^T \sigma_0\,du = \sigma_0^2(T-t)$.

4. For two-factor constant $\sigma = (\sigma_1, \sigma_2)$, compute $\alpha(t,T)$.

   > *Sketch:* $\alpha = (\sigma_1^2 + \sigma_2^2)(T-t)$.

5. Starting from the bond SDE in Section 8.3, show how setting bond drift to $r(t)$ implies the drift restriction.

   > *Sketch:* set drift bracket equal to $r$, cancel, differentiate in $T$.

6. Show that if $\sigma \equiv 0$, then the bond dynamics reduce to deterministic discounting.

   > *Sketch:* diffusion term vanishes; drift equals $r(t)$.

7. Explain why $\pi_t = P(t,T)\,\mathbb{E}^T[H_T \mid \mathcal{F}_t]$ holds under the $T$-forward measure.

   > *Sketch:* apply numeraire pricing with numeraire $P(t,T)$ and note $P(T,T) = 1$.

8. Using the drift transformation formula (Proposition 2.3.1), describe qualitatively how drift changes when moving from $B$ to $P(\cdot,T)$ as numeraire.

   > *Sketch:* drift shifts by covariance with the numeraire's relative volatility.

9. In a discretized HJM simulation, propose a numerical check to confirm $P(t,T)/B(t)$ is a martingale (approximately).

10. Give a reason why unconstrained HJM is hard to use directly for Bermudan swaptions.

11. For separable $\sigma(t,T) = \xi(t)\psi(T)$, describe how one expects a finite-dimensional Markov representation to arise.

12. Interpret $\int_t^T \sigma(t,u)\,du$ financially in bond risk terms.

13. Construct a toy $\sigma(t,T)$ that increases with maturity and discuss how it affects $\alpha(t,T)$.

14. Explain why a swap annuity is a natural numeraire for swaptions.

15. Suppose your simulated curve produces increasing $P(t,T)$ in $T$ for some $t$. List likely causes.

16. Describe how you would calibrate a parametric $\sigma(t,T;\vartheta)$ to both caps and swaptions in principle (high level).

---

## Source Map

### (A) Verified Facts — cite specific sources

- HJM framework fundamentals, drift restriction derivation, and measure change formulas: Andersen & Piterbarg, *Interest Rate Modeling*, Vols. 1-2
- Bond-forward relationships, short-rate bridge examples: Brigo & Mercurio, *Interest Rate Models: Theory and Practice*
- Volatility structure classifications (separable, exponential decay, RS): Andersen & Piterbarg, Vol. 2

### (B) Reasoned Inference — note derivation logic

- Worked examples (1–10) derive from applying the HJM drift restriction formula to various constant/decaying volatility specifications
- Euler discretization scheme follows standard SDE simulation methodology applied to the HJM forward-rate dynamics

### (C) Speculation — flag uncertainties

- Specific calibration workflows may vary by desk/system; the high-level description in Section 7.4 reflects general practice but implementation details differ
