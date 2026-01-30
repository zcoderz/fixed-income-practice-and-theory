# Appendix A3: HJM Framework Essentials

---

## Introduction

What if you could model the entire yield curve—every forward rate for every maturity—in a single, internally consistent framework? And what if the only choice you needed to make was the volatility structure, with no-arbitrage automatically determining everything else?

This is precisely what the Heath-Jarrow-Morton (HJM) framework delivers. Introduced in 1992, HJM models the evolution of the **entire instantaneous forward-rate curve** $T \mapsto f(t,T)$ rather than specifying a single short-rate process and deriving everything else.

The central insight—and the practical payoff—is the **HJM drift restriction**: once you specify how volatile each point on the forward curve is (the volatility function $\sigma(t,T)$), no-arbitrage pins down the drift $\alpha(t,T)$. You cannot freely choose both. This is why people often summarize HJM as *“choose the volatility; the drift follows mechanically.”*

HJM is also a unifying viewpoint: many familiar term structure models (e.g., Ho–Lee, Hull–White) can be seen as special cases of HJM, and market models like LMM can be viewed as a discretized HJM built on **simply compounded** forward rates.

> **Desk Reality: HJM's Role in Modern Rates Trading**
>
> While HJM is foundational theory, most rates desks now use its intellectual descendants—the LIBOR Market Model (LMM) and its SOFR variants—for day-to-day pricing of vanilla products. HJM's direct use is primarily in:
> - Research and model validation
> - Understanding why certain pricing relationships must hold
> - Building intuition for convexity adjustments
> - Exotic products where custom volatility structures are needed
>
> Think of HJM as the "theory of everything" for rates modeling: it contains short-rate models as special cases and provides the intellectual foundation for market models. But its generality is also its weakness—practical implementation requires restricting to tractable subclasses.

This appendix develops the HJM framework from first principles:

1. **Section 1** establishes notation and defines the core objects (bonds, forwards, numeraires)
2. **Section 2** sets up the HJM dynamics: why model forward rates directly?
3. **Section 3** derives the drift restriction—the centerpiece of the theory—with intuition for why volatility creates drift
4. **Section 4** covers measure changes: risk-neutral, forward, and swap measures
5. **Section 5** explores volatility structures that make HJM tractable, including the Gaussian vs log-normal choice
6. **Section 6** bridges HJM to familiar short-rate models and forward to market models
7. **Section 7** provides an implementation roadmap for simulation and calibration
8. **Section 8** contains step-by-step derivations

This appendix assumes familiarity with no-arbitrage pricing (Appendix A1) and short-rate models (Appendix A2). It provides the theoretical foundation for market models (Appendix A4).

**How to use this appendix (reading paths):**

- **If you just want the “one formula” takeaway:** Sections 1.3, 3.1–3.3, and 5.1 (the drift restriction).
- **If measure changes are the stumbling block:** Sections 4.1–4.4 (money-market vs forward vs swap measures), plus Appendix A1 as a refresher.
- **If you’re heading into market models (Appendix A4):** Sections 5.4 and 6.4 (why LMM is the practical descendant of HJM).
- **If you want to implement/simulate:** Section 7 (practical implementation map) and Section 8 (derivations you can turn into code).

**Prerequisites:** Comfort with Itô calculus, the change-of-numeraire idea, and basic fixed income identities relating $P(t,T)$ and $f(t,T)$ from earlier chapters.

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

  - $\mathbb{Q}^T$: $T$-forward measure with numeraire $P(t,T)$; pricing a payoff $H_T$ at $T$ becomes $V(t) = P(t,T)\,\mathbb{E}^T[H_T \mid \mathcal{F}_t]$.

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

$$\boxed{P(t,T) = \exp\!\left(-\int_t^T f(t,u)\,du\right)}$$

**Unit check:** $\ln P$ is dimensionless; $\partial_T \ln P$ has units $1/\text{year}$, so $f(t,T)$ is a rate "per year".

**In practice:** $f(0,T)$ is the forward curve (useful for interpreting steepness/humps); many hedges are "bucketed" by forward-rate key points.

---

### 1.4 HJM state variable: the entire forward curve $f(t,\cdot)$

**Formal definition:** The HJM "state" at time $t$ is the function $T \mapsto f(t,T)$ for all maturities $T \geq t$. The model specifies consistent dynamics for this curve across all $T$.

**Intuition:** Unlike short-rate models (finite-dimensional state), HJM's natural object is the whole term structure; hence it is infinite-dimensional in general and "too unwieldy" without additional structure.

> **Beginner Bridge: From One Number to Infinitely Many**
>
> In short-rate models like Vasicek or Hull-White (Appendix A2), the "state" of the interest rate world is captured by a single number: $r(t)$. Given $r(t)$ and the model parameters, you can reconstruct bond prices for any maturity.
>
> HJM takes a radically different approach: the state is the *entire forward curve*—infinitely many numbers $f(t,T)$ for every maturity $T \geq t$. This is like knowing every point on a yield curve simultaneously, rather than just the overnight rate.
>
> Why would anyone want this complexity? Because it gives you complete freedom to specify how different parts of the curve move. Short rates might be volatile while long rates are stable, or vice versa. HJM accommodates any such pattern—but the price is computational complexity.

**In practice:** Curves are represented on a tenor grid; HJM is implemented in discretized form (see Section 7).

> **Advanced Note: The Musiela Parameterization**
>
> There is an elegant alternative way to index the forward curve. Instead of writing $f(t,T)$ where $T$ is the *calendar maturity*, define:
>
> $$g(t,x) = f(t, t+x)$$
>
> where $x = T - t$ is the *time-to-maturity*. This is the **Musiela parameterization**, introduced by Musiela (1994). Andersen and Piterbarg describe it as "forward rates with a fixed time to maturity rather than a fixed time of maturity."
>
> Under this reparameterization, the HJM dynamics become a **stochastic partial differential equation (SPDE)**. Letting $v(t,x) = \sigma(t, t+x)$, Duffie (Ch. 7, Section K) shows:
>
> $$dg(t,x) = \frac{\partial g(t,x)}{\partial x}\,dt + V(t,x)\,dt + v(t,x)\,dB_t$$
>
> where $V(t,x) = v(t,x) \cdot \int_0^x v(t,z)\,dz$ is the HJM drift restriction rewritten in time-to-maturity coordinates. The extra term $\frac{\partial g}{\partial x}\,dt$ captures the "aging" of the forward curve—as time passes, a 5-year forward becomes a 4-year forward even without any random shock.
>
> **Why it matters:** The Musiela form reveals that the entire forward-rate curve $g(t,\cdot)$ is a **function-valued Markov process**—the curve at time $t$ is a sufficient statistic for its future evolution. This connects HJM to the theory of infinite-dimensional stochastic processes (sometimes called "string" or "random field" models of the term structure).
>
> **For practical implementation**, the standard $f(t,T)$ parameterization is preferred because it maps directly to fixed-maturity instruments. The Musiela form is primarily useful for theoretical analysis and for understanding the infinite-dimensional nature of HJM.

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

**Curve fit "by construction":** If the model is initialized with the observed curve $P(0,T)$ (or $f(0,T)$), the HJM setup treats that time-0 curve as an **input condition**. In other words: you start from the market curve, and the stochastic dynamics are built around it so the model matches the curve at $t=0$.

**Modeling convenience:** In interest-rate derivatives, payoffs typically depend on multiple maturities. Modeling the entire curve gives a consistent joint evolution (though infinite-dimensional unless restricted).

> **Why This Matters for Multi-Maturity Derivatives**
>
> Consider a Bermudan swaption with exercise dates at 1y, 2y, 3y, 4y, and 5y. At each exercise date, the holder compares the value of exercising (receiving a swap) versus continuing. This requires knowing bond prices (hence forward rates) at multiple maturities simultaneously.
>
> A short-rate model gives you $r(t)$ and then computes bond prices via expectations. HJM directly models the bond prices' drivers—the forward rates. Both approaches should agree (if consistent), but HJM makes the multi-maturity structure explicit.

---

### 2.3 Risk-neutral (money-market) vs $T$-forward measure: conceptual difference

**Under $\mathbb{Q}^B$ (numeraire $B(t)$):**
- Pricing uses discounting by the money-market account; traded prices divided by $B(t)$ are martingales.

**Under $\mathbb{Q}^T$ (numeraire $P(t,T)$):**
- If a payoff is paid at $T$, pricing simplifies to

$$V(t) = P(t,T)\,\mathbb{E}^T[H_T \mid \mathcal{F}_t].$$

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

> **Derivation Recap: The Four Key Steps**
>
> 1. **Start with forward dynamics:** $df(t,T) = \alpha\,dt + \sigma \cdot dW$
> 2. **Integrate to get bond dynamics:** Use Itô's lemma on $P = \exp(-\int f)$
> 3. **Impose martingale condition:** Require $P/B$ to be a martingale under $\mathbb{Q}^B$
> 4. **Differentiate to get drift restriction:** Differentiate in $T$ to get local condition
>
> This is the centerpiece of HJM theory. Glasserman calls it "the arbitrage-free dynamics of the forward curve under the risk-neutral measure; it is the centerpiece of the HJM framework."

---

### 3.4 The Convexity Correction Interpretation

The drift restriction has a deep intuition rooted in **convexity effects**. To understand why volatility creates drift, consider what happens when you hold a zero-coupon bond.

**The core mechanism:** A bond's price $P(t,T) = \exp(-\int_t^T f(t,u)\,du)$ is a *convex* function of the integrated forward rate. When forward rates fluctuate randomly around their expected value, Jensen's inequality tells us that the expected bond price exceeds the bond price evaluated at the expected forward rate:

$$\mathbb{E}[P(t,T)] > \exp\left(-\int_t^T \mathbb{E}[f(t,u)]\,du\right)$$

This "convexity advantage" is larger when volatility is higher. But under the risk-neutral measure, discounted bond prices must be martingales—they cannot systematically drift upward. Something must compensate for the convexity advantage.

**The compensation:** The HJM drift restriction provides exactly this compensation. The positive drift $\alpha(t,T) = \sigma(t,T) \cdot \int_t^T \sigma(t,u)\,du$ pushes forward rates *upward* on average, which pushes bond prices *downward*, exactly offsetting the convexity advantage. Put differently: under HJM you *choose* $\sigma(t,T)$, and no-arbitrage *forces* the corresponding drift.

**Graphical intuition:** Imagine the forward curve as a string. Volatility causes the string to vibrate. Because bond prices are convex in rates, this vibration creates a systematic tendency for bond prices to rise. The drift restriction adds just enough upward trend to forward rates to neutralize this tendency and maintain no-arbitrage.

**Why longer maturities get more drift:** The integral $\int_t^T \sigma(t,u)\,du$ accumulates volatility from $t$ to $T$. Longer-maturity forwards have more "volatility accumulated behind them," so they require larger drift corrections. This explains why the drift term grows with time-to-maturity for typical volatility structures.

> **Numerical Example: Drift Correction Size**
>
> If forward rate volatility is 1% (= 0.01) and time-to-maturity is 5 years, the integrated volatility is approximately 5% (= 0.05) and the drift correction is:
>
> $$\alpha = 0.01 \times 0.05 = 0.0005 = 5 \text{ bp/year}$$
>
> This means the 5-year forward rate drifts upward at 5 basis points per year due to convexity, even if the short rate has no drift. Over a 10-year horizon, this compounds to meaningful effects.

> **Desk Reality: Getting the Drift Wrong**
>
> This drift correction is why your Monte Carlo simulation of forward rates must use the HJM drift—not zero drift—under the risk-neutral measure. Getting this wrong creates arbitrage in your simulation: your simulated bond prices will systematically drift in ways that can't happen in a real market.
>
> A common sanity check: after simulation, verify that $\mathbb{E}[P(t,T)/B(t)] \approx P(0,T)/B(0)$. If this fails, you likely have a drift error.

---

### 3.5 "Drift determined by volatility" made explicit

Plugging the drift restriction back into the forward-rate SDE yields the integrated HJM representation:

$$df(t,T) = \sigma(t,T) \cdot \left[\left(\int_t^T \sigma(t,u)\,du\right)dt + dW_t\right],$$

showing the drift is fully implied once $\sigma$ is specified.

A one-factor version is stated directly as an HJM definition in the sources:

$$df(t,T) = \sigma_f(t,T)\left[\left(\int_t^T \sigma_f(t,u)\,du\right)dt + dW(t)\right],$$

under the risk-neutral measure (here $W$ is one-dimensional).

---

### 3.6 Checks (units and limiting cases)

**Limiting case $\sigma \equiv 0$:** then $\alpha(t,T) = 0$ from $\alpha = \sigma \cdot \int\sigma$, so forward rates have deterministic evolution (no diffusion-driven convexity correction). This matches the idea that without randomness, no-arbitrage places no extra drift constraints beyond deterministic consistency.

**Dimensional sanity:**
- $dW$ scales like $\sqrt{dt}$.
- If $f$ has units $1/\text{year}$, then $\sigma$ has units "rate per $\sqrt{\text{year}}$" and $\alpha$ has units "rate per year". The product $\sigma \cdot \int_t^T \sigma\,du$ indeed has the same time scaling as a drift.

---

## 4. Measures Commonly Used with HJM (Risk-Neutral vs Forward Measures)

### 4.1 Why the $T$-forward measure is natural for payoffs at $T$

If a claim pays $H_T$ at time $T$, choosing the ZCB numeraire $P(t,T)$ yields the pricing identity

$$V(t) = P(t,T)\,\mathbb{E}^T[H_T \mid \mathcal{F}_t],$$

because $P(T,T) = 1$.

**Practitioner translation:** you "factor out" discounting and work under a measure where the remaining object is just an expectation of the payoff.

---

### 4.2 How drifts change under measure change (explicit, source-backed)

The change-of-numeraire toolkit provides an explicit drift transformation for diffusion processes when switching from numeraire $S$ to $U$. If under the $S$-numeraire measure a diffusion $X$ has drift $\mu^S$, then under the $U$-numeraire measure its drift is

$$\mu^U(t) = \mu^S(t) - \sigma_X(t)\,\rho\left(\frac{\sigma_S(t)}{S(t)} - \frac{\sigma_U(t)}{U(t)}\right)^\top,$$

and the Brownian motions are related by

$$dW^U(t) = dW^S(t) - \rho\left(\frac{\sigma_U(t)}{U(t)}\right)^\top dt.$$

**Important caution:** applying this to $f(t,T)$ directly requires interpreting $f(t,T)$ as (or as a function of) traded assets in a way consistent with the model setup. The sources explicitly state martingale properties for traded ratios (e.g., forward bond prices, forward LIBOR rates) under forward measures.

**HJM drift under the $T$-forward measure:** When switching from the money-market numeraire $B(t)$ to the $T$-forward numeraire $P(t,T)$, the drift of $f(t,S)$ for $S > T$ becomes:

$$\boxed{\alpha^{(T)}(t,S) = \sigma(t,S) \cdot \int_T^S \sigma(t,u)\,du}$$

Note the lower limit changes from $t$ to $T$. For the forward rate at maturity $T$ itself, $\alpha^{(T)}(t,T) = 0$—the forward rate with matching maturity is driftless under its own forward measure, a key simplification for option pricing.

> **Worked Example: Caplet Pricing Setup**
>
> When pricing a caplet resetting at $T_i$ and paying at $T_{i+1}$, work under the $T_{i+1}$-forward measure where the forward rate $L(t,T_i,T_{i+1})$ is a martingale. This eliminates the drift from the pricing expectation.
>
> Under $\mathbb{Q}^{T_{i+1}}$:
> - Forward LIBOR $L(t,T_i,T_{i+1})$ has zero drift
> - Caplet price = $P(t,T_{i+1}) \cdot \mathbb{E}^{T_{i+1}}[\tau(L(T_i) - K)^+]$
>
> This is much simpler than working under $\mathbb{Q}^B$ where the forward LIBOR has a complicated drift involving the entire volatility structure.

---

### 4.3 Swap measure (if used)

The swap measure uses an annuity numeraire:

$$A_{\alpha,\beta}(t) = \sum_{i=\alpha+1}^{\beta} \tau_i P(t,T_i),$$

and under the associated measure the forward swap rate $S_{\alpha,\beta}(t)$ is a martingale.

**When practitioners use it:** swaption pricing/hedging, because the swap rate is the natural underlying and becomes driftless under the swap-annuity numeraire.

---

### 4.4 "When to use which measure" (practitioner summary)

| Measure | Numeraire | Use When | Key Martingale |
|---------|-----------|----------|----------------|
| $\mathbb{Q}^B$ | Money-market $B(t)$ | General-purpose; enforcing HJM drift restriction | Discounted asset prices |
| $\mathbb{Q}^T$ | ZCB $P(t,T)$ | Payoff at single maturity $T$ | Forward bond prices $P(t,S)/P(t,T)$ |
| Swap measure | Annuity $A(t)$ | Swap-rate payoffs (swaptions) | Forward swap rate |

> **Numerical Example: Drift Under Different Measures**
>
> Consider one-factor HJM with constant $\sigma = 0.01$. Compute drift of $f(t,5)$ at $t=0$:
>
> **Under $\mathbb{Q}^B$:**
> $$\alpha^B(0,5) = 0.01 \cdot \int_0^5 0.01\,du = 0.01 \times 0.05 = 0.0005$$
>
> **Under $\mathbb{Q}^{T=2}$ (2-year forward measure):**
> $$\alpha^{(2)}(0,5) = 0.01 \cdot \int_2^5 0.01\,du = 0.01 \times 0.03 = 0.0003$$
>
> The drift is smaller under the $T=2$ forward measure because we're integrating over a shorter range $[2,5]$ instead of $[0,5]$.

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

In the one-factor case, a separable volatility typically yields a Hull–White-type short-rate representation: the model becomes **finite-dimensional Markov**, so you can summarize the state with a small number of factors instead of the entire curve.

**(ii) Exponential decay in time-to-maturity (Hull–White-type forward vol)**

A specific one-factor form highlighted is

$$\sigma(t,T) = \sigma\,e^{-a(T-t)},$$

which leads to a Hull–White-equivalent dynamics in the one-factor setting discussed (mean-reversion produces exponential decay across maturities).

**(iii) Ritchken–Sankarasubramanian (RS) / finite-dimensional Markov extension**

The general HJM framework is infinite-dimensional—you need the entire forward curve as state. However, Ritchken and Sankarasubramanian (1995) identified conditions under which the short rate's path-dependence can be captured by a single additional state variable, making the model computationally tractable.

**The RS Volatility Condition:** As stated by Brigo and Mercurio (Proposition 5.3.1), a necessary and sufficient condition for a one-factor HJM model to have a two-dimensional Markov representation is:

$$\boxed{\sigma(t,T) = \eta(t) \exp\!\left(-\int_t^T \kappa(x)\,dx\right)}$$

where $\eta(t)$ is an adapted process (possibly stochastic) and $\kappa(x)$ is a deterministic function.

**The auxiliary state variable:** Under the RS condition, prices depend on $(r(t), \phi(t))$ where:

$$\phi(t) = \int_0^t \sigma_{RS}^2(s,t)\,ds$$

This $\phi(t)$ captures the "accumulated variance effect" that would otherwise require knowing the entire path history.

**Joint dynamics:** The two-state Markov process evolves as:

$$d\binom{r(t)}{\phi(t)} = \binom{\mu(r,t)\,dt + \eta(t)\,dW(t)}{[\eta^2(t) - 2\kappa(t)\phi(t)]\,dt}$$

with drift $\mu(r,t) = \kappa(t)[f(0,t) - r(t)] + \phi(t) + \frac{\partial}{\partial t}f(0,t)$.

**Explicit bond price formula:** Brigo and Mercurio (Proposition 5.3.1) show that under RS volatilities, zero-coupon bond prices have the closed form:

$$\boxed{P(t,T) = \frac{P(0,T)}{P(0,t)} \exp\!\left\{-\frac{1}{2}\Lambda^2(t,T)\phi(t) + \Lambda(t,T)[f(0,t) - r(t)]\right\}}$$

where $\Lambda(t,T) = \int_t^T e^{-\int_t^u \kappa(x)\,dx}\,du$. This formula is powerful: bond prices depend only on $(r(t), \phi(t))$ and the initial curve, confirming the two-state Markov property.

**Practical significance:** The RS class enables:
- Recombining lattice implementations (Li-Ritchken-Sankarasubramanian trees)
- Efficient PDE methods in two dimensions
- Stochastic short-rate volatility ($\eta(t)$ can depend on state) while maintaining tractability

**Special case:** When $\kappa$ is constant ($\kappa(x) = a$), the RS volatility reduces to:

$$\sigma(t,T) = \eta(t) e^{-a(T-t)}$$

which is precisely the Hull-White volatility structure, confirming that Hull-White is a special case of the RS class.

---

### 5.3 Humped Volatility: The Mercurio-Moraleda Model

Market-implied forward rate volatilities often exhibit a "hump"—volatility peaks at some intermediate maturity rather than being monotone. The Mercurio-Moraleda (2000) model addresses this by specifying:

$$\sigma(t,T) = \sigma[\gamma(T-t)+1]e^{-\frac{\lambda}{2}(T-t)}$$

where $\sigma$, $\gamma$, and $\lambda$ are non-negative constants.

**Properties:**
- **Humped when $2\gamma > \lambda$:** Volatility increases for short maturities, peaks, then decays
- **Time-to-maturity dependent:** Only depends on $T-t$, not on $t$ and $T$ separately
- **Gaussian:** Rates remain normally distributed (no smile)

**Practical note:** $(\sigma,\gamma,\lambda)$ are **calibration outputs**. Their values depend on the currency, the option market you fit (caps vs swaptions), and the period you calibrate on. Treat them as knobs that control the hump’s location and height, not as fixed “standard” ranges.

> **Worked Example: Mercurio-Moraleda Volatility**
>
> With $\sigma = 0.01$, $\gamma = 0.3$, $\lambda = 0.2$:
>
> | Time-to-maturity | $\sigma(t,T)$ |
> |------------------|---------------|
> | 1 year | $0.01 \times 1.3 \times e^{-0.1} = 0.01176$ |
> | 3 years | $0.01 \times 1.9 \times e^{-0.3} = 0.01406$ |
> | 5 years | $0.01 \times 2.5 \times e^{-0.5} = 0.01516$ |
> | 10 years | $0.01 \times 4.0 \times e^{-1.0} = 0.01472$ |
> | 20 years | $0.01 \times 7.0 \times e^{-2.0} = 0.00948$ |
>
> The hump occurs around 5–7 years in this example.

> **Practitioner Note: Why Humped Vol Matters**
>
> In many markets, implied volatilities for short-to-intermediate maturities are not monotone in maturity; they often peak somewhere in the “belly” of the curve. Flat or purely decaying volatility structures can therefore struggle to match prices across maturities. If you calibrate a simple exponential-decay model to long-dated caps, you can misprice shorter-dated caps (and vice versa).
>
> The Mercurio-Moraleda model lets you match the hump shape while staying within the Gaussian HJM framework. The tradeoff: more parameters to calibrate.

---

### 5.4 Gaussian vs Log-Normal HJM: The Negative Rates Question

One of the most important practical choices in HJM is whether forward rates follow Gaussian (normal) or log-normal dynamics. This section clarifies the tradeoffs.

**Gaussian HJM (Additive Volatility)**

$$df(t,T) = \alpha(t,T)\,dt + \sigma(t,T)\,dW_t$$

where $\sigma(t,T)$ is deterministic (not depending on $f$).

**Consequences:**
- Forward rates are normally distributed
- **Negative rates are possible**: $f(t,T)$ can become negative with positive probability
- Analytically tractable: bond prices, options have closed forms
- HJM drift restriction takes the standard form

**Log-Normal HJM (Multiplicative Volatility)**

$$df(t,T) = f(t,T)\left[\alpha(t,T)\,dt + \sigma(t,T)\,dW_t\right]$$

where volatility is proportional to the rate level.

**Consequences (and why this is subtle in HJM):**

- Conditional on being well-defined, this produces a **log-normal-like** behavior: volatility scales with the level of the forward rate.
- This tends to **rule out negative forward rates** by construction (which may or may not be desirable).
- However, for **instantaneous forward-rate HJM**, prescribing multiplicative volatility is technically delicate. In particular, preventing **explosions** in the forward-rate dynamics generally imposes restrictions on how the volatility can grow with the state. In the standard treatments, these restrictions effectively *preclude* a clean “log-normal instantaneous forward rate” specification.

This is one of the reasons the industry prefers to apply log-normal dynamics to **discrete, simply-compounded forward rates** (the market model / LMM viewpoint) rather than to instantaneous forwards; with a finite tenor grid, an explosion-free log-normal specification becomes possible (Appendix A4).

**Gaussian vs log-normal (instantaneous-forward) in one table**

| Consideration | Gaussian HJM (additive) | “Log-normal” instantaneous-forward HJM |
|---|---|---|
| Can forwards be negative? | Yes | Not in the pure multiplicative form |
| Mathematical behavior | Generally well-behaved | Can be technically restrictive (explosions / ill-posedness) |
| What it’s good for | Tractable curve dynamics; theory; Gaussian subclasses (e.g., LGM/Hull–White) | Mostly conceptual; if you want log-normal behavior, do it on a tenor grid (Appendix A4) |

**Shifted/Displaced Diffusion: The Compromise**

A common middle ground is the shifted (displaced) diffusion:

$$df(t,T) = (f(t,T) + \delta)\sigma(t,T)\,dW_t + \alpha(t,T)\,dt$$

where $\delta > 0$ is a "shift" parameter. This:
- Allows forwards to be negative down to $-\delta$ (but not below)
- Can be calibrated so that $(f+\delta)$ behaves “log-normal-ish” even when $f$ is near zero/negative
- Should be viewed as a **pragmatic parameterization**: $\delta$ is a modeling/quoting choice and is not universal across currencies or desks

> **Desk Reality: What people actually mean by “log-normal” in rates**
>
> - For a **single rate** (a swap rate or a forward rate), “log-normal vs normal” often just refers to whether you use a Black-style (multiplicative) or Bachelier-style (additive) option model, possibly with a shift.
> - For a **full-curve model**, a pure log-normal instantaneous-forward HJM is rarely implemented directly; if you truly want log-normal behavior, you typically move to a **market model** on a tenor grid (Appendix A4).

---

### 5.5 Multi-Factor HJM: Correlation and Principal Components

Real yield curves don't just shift up and down—they twist, steepen, flatten, and "butterfly." Capturing this requires multiple factors.

**General Multi-Factor Setup**

$$df(t,T) = \alpha(t,T)\,dt + \sum_{i=1}^d \sigma_i(t,T)\,dW_i(t)$$

where $W_1, \ldots, W_d$ are independent Brownian motions.

**Drift restriction:** The multi-factor version is:

$$\alpha(t,T) = \sum_{i=1}^d \sigma_i(t,T) \int_t^T \sigma_i(t,u)\,du$$

**Multi-Factor Separability (Carverhill Condition)**

For multi-factor Markovian short rate:

$$\sigma_i(t,T) = \xi_i(t)\psi_i(T) \quad \text{for each factor } i$$

**Principal Component Interpretation**

In practice (both in risk reports and in model design), yield curve movements are often summarized with a **principal component** lens:
- **Factor 1 (Level):** roughly-parallel shifts
- **Factor 2 (Slope):** steepening/flattening (twists)
- **Factor 3 (Curvature):** “butterfly” moves (belly vs wings)

A 2–3 factor HJM model can often capture a large share of observed yield curve dynamics. Typical volatility structures:

| Factor | Economic Meaning | Volatility Shape |
|--------|-----------------|------------------|
| Level | Parallel shift | $\sigma_1(t,T) \approx$ constant |
| Slope | Twist | $\sigma_2(t,T)$ increasing in $T-t$ |
| Curvature | Butterfly | $\sigma_3(t,T)$ humped |

> **Practitioner Note: How Many Factors?**
>
> - **1 factor:** Works for “level-only” intuition and some simple exposures, but can miss slope/curve risk.
> - **2 factors:** Often the first practical step for capturing both level and slope moves.
> - **3 factors:** Adds curvature and is useful when belly-vs-wing risk matters (butterflies, some CMS spread structures).
> - **4+ factors:** Possible, but calibration and stability become more demanding; the incremental benefit depends on the product set.
>
> The choice depends on what you're pricing. A 5y-10y steepener needs at least 2 factors; a 2s-5s-10s butterfly needs 3.

---

### 5.6 Practical implications (qualitative, source-aligned)

- The general HJM class is infinite-dimensional and "too unwieldy" in practice (Brigo and Mercurio); hence the need for structured volatility families that reduce to finitely many Markov state variables.

- Volatility structure matters because it drives both:
  - option prices (directly via $\sigma$),
  - and forward drifts (indirectly via the HJM restriction), affecting curve evolution and convexity corrections.

---

## 6. Relationship to Familiar Models (Bridge Section)

The HJM framework is not a competitor to short-rate models—it is a unifying theory that *contains* them. Understanding the bridge between frameworks clarifies when each perspective is most useful.

### 6.1 Short-rate models induce an HJM volatility (conceptual equivalence)

If you model $r(t)$ (and hence $B(t)$), you can compute $P(t,T)$ and therefore $f(t,T)$. The induced forward-rate dynamics must be arbitrage-free and therefore satisfy an HJM drift restriction under the appropriate pricing measure.

In practice, short-rate models are attractive because they reduce the world to a small number of state variables (often one), which makes calibration and pricing fast. The flip side is that a low-dimensional short-rate state can be too restrictive if you want rich, maturity-by-maturity volatility and correlation behavior. HJM makes that tradeoff explicit by taking the forward curve itself as the state.

The key insight is that every short-rate model *implies* an HJM volatility structure. Going the other direction, the Carverhill condition tells us when an HJM model implies a Markovian short rate:

$$\sigma_i(t,T) = \xi_i(t)\psi_i(T)$$

This separability is necessary and sufficient for the short rate to be Markov in the one-factor Gaussian case.

**What you gain going from HJM to Hull-White:**
- Tractability: one state variable instead of infinity
- Closed-form bond options (Jamshidian decomposition)
- Efficient lattice methods

**What you lose going from HJM to Hull-White:**
- Flexibility: can only fit volatility structures of the form $\sigma e^{-a(T-t)}$
- Cannot independently specify volatilities at different maturities

---

### 6.2 The Merton Model: Simplest Bridge Example

Consider the "Merton toy model" (constant drift, constant volatility):

$$dr(t) = \theta\,dt + \sigma\,dW_t$$

**Step 1: Compute bond prices.** For this affine model:

$$P(t,T) = \exp\left[\frac{\sigma^2}{6}(T-t)^3 - \frac{\theta}{2}(T-t)^2 - (T-t)r_t\right]$$

**Step 2: Derive forward rates.** Using $f(t,T) = -\partial_T \ln P(t,T)$:

$$f(t,T) = -\frac{\sigma^2}{2}(T-t)^2 + \theta(T-t) + r_t$$

**Step 3: Compute forward dynamics.** Differentiate and substitute $dr$:

$$df(t,T) = \sigma^2(T-t)\,dt + \sigma\,dW_t$$

**Verification:** Check the HJM drift restriction. With constant $\sigma(t,T) = \sigma$:

$$\alpha(t,T) = \sigma \int_t^T \sigma\,du = \sigma^2(T-t) \quad \checkmark$$

The forward drift $\sigma^2(T-t)$ is exactly what HJM predicts.

---

### 6.3 Hull-White to HJM: Complete Mapping

The one-factor Hull-White model is:

$$dr(t) = [\theta(t) - a\,r(t)]\,dt + \sigma\,dW_t$$

where $a > 0$ is the mean-reversion speed and $\sigma > 0$ is the short-rate volatility.

**The HJM volatility induced by Hull-White:** Following Brigo and Mercurio, set:

$$\boxed{\sigma(t,T) = \sigma e^{-a(T-t)}}$$

This has the separable form $\xi(t) = \sigma e^{at}$ and $\psi(T) = e^{-aT}$, confirming Markovianity.

**Verify the HJM drift:** Compute:

$$\int_t^T \sigma(t,u)\,du = \int_t^T \sigma e^{-a(u-t)}\,du = \frac{\sigma}{a}(1 - e^{-a(T-t)})$$

Then:

$$\alpha(t,T) = \sigma e^{-a(T-t)} \cdot \frac{\sigma}{a}(1 - e^{-a(T-t)}) = \frac{\sigma^2}{a}e^{-a(T-t)}(1 - e^{-a(T-t)})$$

**Worked example:** With $a = 0.10$ (10% mean reversion) and $\sigma = 0.01$ (1% short-rate vol):

At $t = 0$, $T = 5$ years:
- Forward volatility: $\sigma(0,5) = 0.01 \cdot e^{-0.5} = 0.00607$
- Integrated vol: $\int_0^5 \sigma(0,u)\,du = \frac{0.01}{0.1}(1 - e^{-0.5}) = 0.0393$
- Drift: $\alpha(0,5) = 0.00607 \times 0.0393 = 0.000239$ (about 2.4 bp/year)

At $t = 0$, $T = 1$ year:
- Forward volatility: $\sigma(0,1) = 0.01 \cdot e^{-0.1} = 0.00905$
- Integrated vol: $\int_0^1 \sigma(0,u)\,du = \frac{0.01}{0.1}(1 - e^{-0.1}) = 0.00952$
- Drift: $\alpha(0,1) = 0.00905 \times 0.00952 = 0.0000861$ (about 0.9 bp/year)

**Key insight:** Longer-maturity forwards have *lower* volatility (due to mean reversion) but *more accumulated volatility behind them*, leading to a non-monotonic drift profile.

---

### 6.4 Bridge to LIBOR Market Models (Forward to Appendix A4)

The LIBOR Market Model (LMM/BGM) can also be derived from the HJM framework, as noted by Brigo and Mercurio: "Even the celebrated LIBOR market model was developed starting from instantaneous-forward-rate dynamics in Brace, Gatarek and Musiela (1997)."

**Why the industry moved from HJM to LMM:**

1. **Calibration to liquid instruments:** LMM's forward LIBOR rates correspond directly to cap/floor underlyings. In HJM, you model instantaneous forwards which aren't directly traded.

2. **The "continuous tenor" problem:** HJM requires modeling $f(t,T)$ for all $T$—infinitely many rates. This creates discretization issues and makes calibration to a finite set of instruments awkward.

3. **Log-normal forwards on a tenor grid:** In LMM, you can specify log-normal dynamics for **discrete, simply compounded** forward rates (on a finite tenor grid) without running into the same well-posedness issues that arise when you try to make *instantaneous* forward rates log-normal.

4. **Direct observation:** Forward LIBOR rates $L(t, T_i, T_{i+1})$ are directly quoted in the market. Instantaneous forwards $f(t,T)$ are theoretical constructs.

**The conceptual connection:**

LMM can be viewed as a specific choice of how to discretize HJM:
- Work with forward LIBOR rates $L(t, T_i, T_{i+1}) = \frac{1}{\tau_i}\left(\frac{P(t,T_i)}{P(t,T_{i+1})} - 1\right)$
- Specify log-normal dynamics under the $T_{i+1}$-forward measure
- Apply HJM-style drift restrictions (but now for discrete forwards)

The full development is in Appendix A4.

> **Desk Reality: HJM vs LMM in Practice**
>
> | Aspect | HJM | LMM |
> |--------|-----|-----|
> | Calibration | To volatility functions | To cap/swaption quotes |
> | State space | Infinite (entire curve) | Finite (forward rates) |
> | Direct use on desks | Rare | Standard |
> | Best for | Theory, research, exotics | Vanilla rates products |
>
> Most rates desks use LMM (or its SOFR variant) for caps, floors, and swaptions. HJM is typically encountered when:
> - Building intuition for convexity adjustments
> - Validating that an LMM implementation is consistent
> - Pricing exotics with custom volatility structures
> - Academic research

---

### 6.5 When to Use HJM in Practice

This section provides practical guidance on when HJM (vs alternatives) is the right choice.

**Use Hull-White (Markovian HJM) when:**
- Pricing vanilla caps, floors, swaptions
- Need closed-form solutions or fast lattices
- Single-factor dynamics are adequate
- Calibrating to a few key instruments

**Use general (non-Markovian) HJM when:**
- You need volatility structures that don't fit Hull-White
- Path-dependent exotics where vol term structure matters
- Research/validation of other models
- Multi-factor models with specific correlation structures

**Use LMM instead of HJM when:**
- Calibrating to cap/swaption grids
- Log-normal forward dynamics desired (without explosion)
- Working with standard rates products
- Want direct connection to quoted instruments

> **Practitioner Note: The Role of HJM on Modern Rates Desks**
>
> On most rates trading desks today, HJM is not directly used for production pricing. Here's the typical model stack:
>
> | Product | Typical Model |
> |---------|---------------|
> | Vanilla swaps | Curve-based (no stochastics) |
> | Caps/floors | LMM or SABR |
> | European swaptions | LMM or SABR |
> | Bermudan swaptions | Hull-White or LGM (Markovian HJM) |
> | CMS products | LMM with SABR marginals |
> | Exotics (TARNs, etc.) | LMM Monte Carlo |
>
> HJM's main role is **foundational**: it explains *why* the drift restrictions in LMM take the form they do, and it provides the theoretical framework for understanding convexity adjustments.
>
> If you're asked "why does the forward LIBOR rate have this particular drift under the spot measure?", the answer comes from HJM.

---

## 7. Practical Implementation Map (Discretization of an Infinite-Dimensional Model)

The general HJM framework is infinite-dimensional, which poses obvious computational challenges. This section provides a practical roadmap for implementation.

> **Important caveat (Glasserman):** "In the generality of the HJM setting, some discretization error is usually inevitable." Unlike short-rate models such as Vasicek or CIR that can be simulated exactly at discrete dates, HJM requires approximating both the time evolution *and* the maturity dimension. This means that even with small time steps, there is inherent approximation error that must be monitored.

### 7.1 Discretize the forward curve on a tenor grid

**Setup:**
- Choose maturities $T_1 < T_2 < \cdots < T_M$ (e.g., quarterly out to 30 years).
- Represent the state as the vector $\mathbf{f}_t = (f(t,T_1), \ldots, f(t,T_M))$.
- Initialize from the market curve: $f(0, T_j) = f^{\text{market}}(0, T_j)$ for all $j$.

**Integral approximations:** Use quadrature on the grid:
- Piecewise constant on the maturity grid: write the maturity knots as $u_0=t < u_1 < \cdots < u_J = T_j$ and approximate
  $$\int_t^{T_j} \sigma(t,u)\,du \;\approx\; \sum_{k=1}^{J} \sigma(t,u_k)\,(u_k-u_{k-1}).$$
- Trapezoidal rule for better accuracy on coarse maturity grids

---

### 7.2 Simulate discretized HJM (Euler scheme)

**Algorithm (one path):**

```
Input: Initial curve f(0,T_j), volatility function σ(t,T),
       time step Δt, number of steps N, d-factor Brownian motion
Output: Simulated forward curve at each time step

For n = 0 to N-1:
    t_n = n * Δt

    # Generate d-dimensional Brownian increment
    ΔW = sqrt(Δt) * randn(d)

    # Running integral over maturity (piecewise-constant quadrature)
    cum_int_sigma = 0   # vector in R^d
    prev_T = t_n

    For j = 1 to M (each tenor point):
        If T_j > t_n:  # Only update future maturities

            # Approximate ∫_{t_n}^{T_j} σ(t_n,u) du on the maturity grid
            ΔT = T_j - prev_T
            prev_T = T_j
            cum_int_sigma += σ(t_n, T_j) * ΔT

            # Compute drift from HJM restriction
            α = σ(t_n, T_j) · cum_int_sigma

            # Euler update
            f(t_{n+1}, T_j) = f(t_n, T_j) + α * Δt + σ(t_n, T_j) · ΔW
```

**Critical implementation note:** All tenor points share the *same* Brownian increment $\Delta W$ at each time step. This ensures the correlation structure is preserved.

---

### 7.3 Reconstruct bond prices from the simulated forward curve

At any simulation time $t$, compute:

$$P(t,T_j) \;\approx\; \exp\left(-\sum_{k=1}^{J} f(t,u_k)\,(u_k-u_{k-1})\right),\quad u_0=t,\;u_J=T_j.$$

**Numerical pitfalls to avoid:**
1. **Negative forward rates:** Gaussian HJM can produce negative forwards. This is not automatically a bug, but it may be inconsistent with a downstream model assumption (e.g., applying a strictly log-normal option smile to the same rate without a shift).
2. **Curve explosion:** Large volatility + long simulation can produce unrealistic curves. Monitor $\max_j |f(t, T_j)|$ for sanity.
3. **Bond-price diagnostics:** Ensure $P(t,T) > 0$ always and watch for numerical artifacts (NaNs/Infs, wild oscillations across maturities). Note: if rates can be negative, $P(t,T)$ does not have to be decreasing in $T$.

---

### 7.4 Variance reduction techniques

For Monte Carlo pricing, several techniques improve efficiency:

**Antithetic variates:** For each path using $\Delta W$, also simulate with $-\Delta W$. Average the two payoffs. This is particularly effective for HJM because the forward dynamics are linear in the Brownian increments.

**Control variates:** Use the analytical price of a simple instrument (e.g., a vanilla caplet under the same model) as a control. The residual variance after control-variate adjustment is often much smaller.

**Brownian bridge construction:** For path-dependent payoffs, generate terminal values first, then fill in intermediate points. This can be combined with low-discrepancy sequences (quasi-Monte Carlo).

**Principal component reduction:** For multi-factor HJM, the first few principal components often explain most variance. Simulate only the leading components and use importance sampling for the rest.

---

### 7.5 Calibration workflow

**Step 1: Choose a volatility parameterization**

Select a functional form $\sigma(t,T;\theta)$ with parameter vector $\theta$. Common choices:

| Parameterization | Parameters | Best For |
|------------------|------------|----------|
| Constant | $\sigma$ | Simple testing |
| Exponential decay | $\sigma, a$ | Hull-White-like behavior |
| Humped (Mercurio-Moraleda) | $\sigma_0, \gamma, \lambda$ | Realistic vol term structure |
| Multi-factor separable | $\xi_i(t), \psi_i(T)$ | Rich correlation structure |

**Step 2: Fit to market instruments**

The choice of instruments determines which aspects of the model are pinned down:

| Instrument | What It Calibrates |
|------------|-------------------|
| Caplets/Floorlets | Forward rate volatility $\sigma(t,T)$ at specific tenors |
| Swaptions | Integrated volatility over swap tenor; correlation structure |
| CMS rates | Convexity adjustments; measure-change effects |

**Calibration objective:** Minimize pricing error:

$$\min_\theta \sum_i w_i \left[V_i^{\text{model}}(\theta) - V_i^{\text{market}}\right]^2$$

where $w_i$ are weights (often inverse bid-offer or inverse vega).

> **Worked Example: Exponential-Decay HJM Calibration**
>
> **Given:** Cap volatilities at 1y, 2y, 5y, 10y maturities.
>
> **Target:** Fit $\sigma(t,T) = \sigma e^{-a(T-t)}$
>
> **Step 1:** Convert cap vols to caplet vols using cap stripping (or use ATM caplet quotes if available).
>
> | Maturity | ATM Cap Vol | Stripped Caplet Vol |
> |----------|-------------|---------------------|
> | 1y | 25% | 25% |
> | 2y | 24% | 23% |
> | 5y | 22% | 19% |
> | 10y | 20% | 16% |
>
> **Step 2:** Set up least-squares objective. Under HJM, the caplet volatility for a caplet fixing at $T$ is approximately:
>
> $$\sigma_{\text{caplet}}(T) \approx \sqrt{\frac{1}{T}\int_0^T \sigma^2(s,T)\,ds} = \sigma\sqrt{\frac{1-e^{-2aT}}{2aT}}$$
>
> **Step 3:** Minimize:
>
> $$\min_{\sigma,a} \sum_i \left[\sigma\sqrt{\frac{1-e^{-2aT_i}}{2aT_i}} - \sigma_i^{\text{market}}\right]^2$$
>
> **Step 4:** Typical result: $\sigma \approx 0.28$, $a \approx 0.08$

**Step 3: Validate**

- Reprice calibration instruments (should match closely)
- Price out-of-sample instruments (check reasonableness)
- Simulate paths and verify:
  - Bond prices positive and decreasing in maturity
  - Forward rates in reasonable range
  - Martingale tests pass (discounted bond prices should have zero mean drift)

---

### 7.6 Method Selection Guide

| Product Type | Recommended Method | Why |
|--------------|-------------------|-----|
| European bond option | Closed-form (if separable HJM) | RS formula available |
| European cap/floor | Closed-form or 1D PDE | Hull-White-equivalent gives Black formula |
| Bermudan swaption | Lattice (if Markovian) or LSM | Early exercise requires backward induction |
| Path-dependent exotic | Monte Carlo | No recombining tree possible |
| CMS spread option | Monte Carlo with control | Path-dependent, multi-factor |

---

### 7.7 Common numerical pitfalls

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Inconsistent drift | $P(t,T)/B(t)$ drifts in simulation | Recompute $\alpha$ from $\sigma$ at each step |
| Grid too coarse | Bond prices oscillate | Finer tenor grid or higher-order integration |
| Time step too large | Simulation unstable | Reduce $\Delta t$; use implicit schemes |
| Ignoring maturity roll | Error grows over long horizons | Remove expired maturities from grid |
| Wrong correlation across factors | Mispriced spread options | Use proper Cholesky decomposition for $\Delta W$ |

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

### 8.6 Bond Option Pricing Under HJM

For the RS class of HJM models, bond options have closed-form solutions. The key result from Brigo and Mercurio:

**Zero-Coupon Bond Call under RS:**

Under the RS volatility $\sigma(t,T) = \eta(t)e^{-\int_t^T \kappa(x)dx}$, the price of a European call option with strike $K$, expiry $T$, on a bond maturing at $\tau > T$ is:

$$\text{ZBC}(t,T,\tau,K) = P(t,\tau)\Phi(d_1) - KP(t,T)\Phi(d_2)$$

where:
$$d_1 = \frac{\ln(P(t,\tau)/(KP(t,T))) + \frac{1}{2}\Sigma^2}{\Sigma}$$
$$d_2 = d_1 - \Sigma$$

and $\Sigma^2$ is the variance of $\ln P(T,\tau)$ under the $T$-forward measure:

$$\Sigma^2 = \int_t^T \sigma_P^2(s,T,\tau)\,ds$$

where $\sigma_P(s,T,\tau) = \Lambda(T,\tau) \cdot \eta(s) \cdot e^{-\int_s^T \kappa(x)dx}$ is the bond volatility.

**Connection to Hull-White:** When $\kappa$ is constant ($\kappa = a$), this reduces to the standard Hull-White bond option formula.

> **Practitioner Note: Jamshidian Decomposition**
>
> For options on coupon bonds (or swaptions), the Jamshidian (1989) decomposition applies in Gaussian HJM:
>
> A call option on a coupon bond = sum of calls on the constituent zero-coupon bonds
>
> This works because, under Gaussian HJM, all bond prices are decreasing functions of the same underlying driver (the short rate or its equivalent state variable). At any rate level, there's a unique "exercise boundary."
>
> This decomposition does NOT work for non-Gaussian or multi-factor models where bond prices don't have the monotonicity property.

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

### Example 11: Humped volatility drift computation

Using Mercurio-Moraleda with $\sigma_0 = 0.01$, $\gamma = 0.3$, $\lambda = 0.2$:

$$\sigma(0,T) = 0.01[0.3T + 1]e^{-0.1T}$$

**At $T = 5$:**
$$\sigma(0,5) = 0.01 \times 2.5 \times e^{-0.5} = 0.01516$$

**Integrated volatility:** $\int_0^5 \sigma(0,u)\,du$ requires numerical integration:

| $u$ | $\sigma(0,u)$ | $\sigma(0,u) \cdot \Delta u$ |
|-----|---------------|------------------------------|
| 1 | 0.01176 | 0.01176 |
| 2 | 0.01306 | 0.01306 |
| 3 | 0.01395 | 0.01395 |
| 4 | 0.01454 | 0.01454 |
| 5 | 0.01516 | 0.01516 |

Sum $\approx 0.0685$.

**HJM drift:**
$$\alpha(0,5) = 0.01516 \times 0.0685 = 0.00104$$

This is about 10 bp/year—larger than flat-volatility case due to the hump accumulating more integrated volatility.

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

**HJM drift under $T$-forward measure:**

$$\boxed{\alpha^{(T)}(t,S) = \sigma(t,S) \cdot \int_T^S \sigma(t,u)\,du}$$

**Pricing under $T$-forward measure:**

$$V(t) = P(t,T)\,\mathbb{E}^T[H_T \mid \mathcal{F}_t].$$

**Hull-White HJM volatility:**

$$\boxed{\sigma(t,T) = \sigma e^{-a(T-t)}}$$

**RS condition:**

$$\boxed{\sigma(t,T) = \eta(t) \exp\!\left(-\int_t^T \kappa(x)\,dx\right)}$$

---

## 12. Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Instantaneous forward $f(t,T)$ | $-\partial_T \ln P(t,T)$ | Core state variable in HJM |
| HJM drift restriction | $\alpha = \sigma \cdot \int\sigma$ | No-arbitrage pins down drift |
| Convexity correction | Drift offsets Jensen's advantage | Explains why vol creates drift |
| Separable volatility | $\sigma(t,T) = \xi(t)\psi(T)$ | Implies Markovian short rate |
| RS class | $\sigma = \eta(t)e^{-\int\kappa}$ | Two-state Markov, closed-form bonds |
| $T$-forward measure | Numeraire $P(t,T)$ | Simplifies single-maturity payoffs |
| Gaussian vs “log-normal” | Additive vs multiplicative vol | Negativity vs well-posedness; log-normality is usually applied on a tenor grid (LMM) |
| Multi-factor HJM | Multiple Brownian drivers | Captures level/slope/curvature |

---

## 13. Flashcards (Q/A)

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
| 17 | What does separable $\sigma(t,T) = \xi(t)\psi(T)$ buy you? | Hull–White-type (finite-dimensional Markov) short-rate model. |
| 18 | Example exponential-decay forward vol? | $\sigma(t,T) = \sigma\,e^{-a(T-t)}$. |
| 19 | Why isn't "general HJM" used directly? | It is infinite-dimensional and too unwieldy; subclasses are sought. |
| 20 | If $\sigma = 0$, what is $\alpha$? | $\alpha = 0$. |
| 21 | Why does volatility create positive drift in forward rates? | To offset the "convexity advantage" in bond prices (Jensen's inequality). |
| 22 | What is the Ritchken-Sankarasubramanian (RS) condition? | $\sigma(t,T) = \eta(t)e^{-\int_t^T \kappa(x)dx}$ for two-state Markov. |
| 23 | What is the auxiliary state variable $\phi(t)$ in RS models? | $\phi(t) = \int_0^t \sigma_{RS}^2(s,t)ds$, capturing accumulated variance. |
| 24 | What HJM volatility corresponds to Hull-White? | $\sigma(t,T) = \sigma e^{-a(T-t)}$. |
| 25 | Why do longer-maturity forwards have more HJM drift? | More "accumulated volatility behind them" via $\int_t^T \sigma du$. |
| 26 | HJM drift under $T$-forward measure for $f(t,S)$ with $S > T$? | $\alpha^{(T)}(t,S) = \sigma(t,S) \cdot \int_T^S \sigma(t,u)du$ (lower limit is $T$). |
| 27 | What is the main issue with log-normal *instantaneous-forward* HJM? | It can be technically ill-behaved (explosions / restrictive conditions). In practice, log-normality is typically applied to **discrete** forward rates in LMM. |
| 28 | When is Gaussian HJM a reasonable choice? | When you want tractability and you are willing to allow negative forwards (or your market/portfolio needs them). |
| 29 | What is the Mercurio-Moraleda volatility form? | $\sigma(t,T) = \sigma[\gamma(T-t)+1]e^{-\frac{\lambda}{2}(T-t)}$. |
| 30 | Why did the industry move from HJM to LMM? | Better calibration to discrete instruments; log-normal dynamics are applied to **tenor-grid** forward rates without the same issues as log-normal instantaneous forwards. |
| 31 | What is the Musiela parameterization? | Reindex forward rates by time-to-maturity: $g(t,x) = f(t,t+x)$, leading to an SPDE formulation. |
| 32 | What extra term appears in the Musiela SPDE vs standard HJM? | $\frac{\partial g}{\partial x}\,dt$—the "aging" of the curve as time passes. |

---

## 14. Mini Problem Set (20 Questions)

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

7. Explain why $V(t) = P(t,T)\,\mathbb{E}^T[H_T \mid \mathcal{F}_t]$ holds under the $T$-forward measure.

   > *Sketch:* apply numeraire pricing with numeraire $P(t,T)$ and note $P(T,T) = 1$.

8. Using the drift transformation formula (Proposition 2.3.1), describe qualitatively how drift changes when moving from $B$ to $P(\cdot,T)$ as numeraire.

   > *Sketch:* drift shifts by covariance with the numeraire's relative volatility.

9. In a discretized HJM simulation, propose a numerical check to confirm $P(t,T)/B(t)$ is a martingale (approximately).

   > *Sketch:* Average $P(t,T)/B(t)$ over paths; should equal $P(0,T)/B(0)$.

10. Give a reason why unconstrained HJM is hard to use directly for Bermudan swaptions.

    > *Sketch:* Non-Markovian; no recombining tree; requires LSM or non-recombining lattice.

11. For separable $\sigma(t,T) = \xi(t)\psi(T)$, describe how one expects a finite-dimensional Markov representation to arise.

    > *Sketch:* Path dependence factors through a finite number of integrals.

12. Interpret $\int_t^T \sigma(t,u)\,du$ financially in bond risk terms.

    > *Sketch:* This is the bond's instantaneous volatility; bond aggregates forward shocks.

13. Construct a toy $\sigma(t,T)$ that increases with maturity and discuss how it affects $\alpha(t,T)$.

    > *Sketch:* e.g., $\sigma(t,T) = c(T-t)$; then $\int = c(T-t)^2/2$, $\alpha = c^2(T-t)^3/2$, growing cubically.

14. Explain why a swap annuity is a natural numeraire for swaptions.

    > *Sketch:* Swap rate is a ratio of floating leg PV to annuity; under annuity numeraire, swap rate is martingale.

15. Suppose your simulated curve produces increasing $P(t,T)$ in $T$ for some $t$. List likely causes.

    > *Sketch:* (1) Negative forwards/short rates (not automatically an arbitrage) can imply increasing $P(t,T)$. (2) Discretization/integration bugs (wrong maturity step, wrong indexing). (3) Wrong drift under the chosen measure/numeraire. (4) Numerical instability (too-large $\Delta t$, extreme vols, overflow in exponentials).

16. Describe how you would calibrate a parametric $\sigma(t,T;\vartheta)$ to both caps and swaptions in principle (high level).

    > *Sketch:* Choose $\vartheta$ to minimize pricing error across instruments; caps pin forward vol, swaptions pin correlation.

17. **Hull-White mapping:** Given Hull-White parameters $a = 0.05$ and $\sigma = 0.008$, compute the HJM forward rate volatility $\sigma(t,T)$ for $T - t = 10$ years. Then compute the corresponding drift $\alpha(t,T)$.

   > *Solution:* $\sigma(t,T) = 0.008 \cdot e^{-0.5} \approx 0.00485$. Integrated vol: $\frac{0.008}{0.05}(1-e^{-0.5}) \approx 0.0629$. Drift: $0.00485 \times 0.0629 \approx 3.05$ bp/year.

18. **RS framework:** Explain why the RS condition $\sigma(t,T) = \eta(t)e^{-\int_t^T \kappa(x)dx}$ allows for a Markovian representation even when $\eta(t)$ depends on the short rate.

   > *Solution:* The path-dependence is captured entirely by the auxiliary state $\phi(t)$; together with $r(t)$, this forms a sufficient statistic for all bond prices.

19. **Convexity intuition:** A bond with 10-year maturity has higher convexity than a 2-year bond. Explain qualitatively why the HJM drift for the 10-year forward should be larger.

   > *Solution:* Higher convexity means larger "Jensen's advantage"; more upward drift in forward rates needed to offset. Also, the integral $\int_t^T \sigma du$ is larger for longer maturities.

20. **Variance reduction:** You are pricing a 5-year Bermudan swaption via Monte Carlo under a 2-factor HJM model. Propose two variance reduction techniques and explain why they might help.

   > *Solution:* (1) Antithetic variates work well because forward dynamics are linear in $\Delta W$. (2) Control variate using European swaption (same model, analytical price) removes systematic bias related to the swaption's intrinsic value.

---

## References

- Leif B. G. Andersen & Vladimir V. Piterbarg, *Interest Rate Modeling* (HJM framework, drift restriction, market-model motivation, technical constraints behind “log-normal” instantaneous forwards)
- Damiano Brigo & Fabio Mercurio, *Interest Rate Models: Theory and Practice* (HJM drift restriction and Markovian subclasses; RS condition; structured volatility families)
- Paul Glasserman, *Monte Carlo Methods in Financial Engineering* (HJM discretization and simulation considerations)
- Darrell Duffie, *Dynamic Asset Pricing Theory* (term-structure modeling and the Musiela/SPDE viewpoint)

## Inputs Needed (NOT SURE)

- None.

---

*Appendix A3 of Fixed Income: Practice and Theory*
