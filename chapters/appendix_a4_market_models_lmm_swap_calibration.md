# Appendix A4: Market Models — LMM and Swap Market Model; Calibration Logic

---

## Introduction

When a swaption trader at a major dealer quotes a volatility smile across strikes, the number they show you is intimately connected to a pricing model—but which one? The answer, for most rates desks since the mid-1990s, has been some variant of a **market model**: either the LIBOR Market Model (LMM, also called BGM after Brace, Gatarek, and Musiela) for caps and floors, or the Swap Market Model (SMM) for swaptions. These models took the rates world by storm because they price caps and swaptions using the same Black formula that traders already used for quoting, while providing an internally consistent arbitrage-free framework for exotic derivatives.

But the story doesn't end with Black's formula. In modern rates markets you often have to navigate additional layers: volatility *smiles* (SABR-style parameterizations), near-zero / sometimes-negative rate regimes (normal vs shifted-lognormal conventions), and multi-curve pricing (discounting and projection curves can differ). A desk quant needs to be fluent in the model layer *and* in the quoting/convention layer.

This appendix provides a comprehensive treatment of market models suitable for practitioners moving from middle office to front office roles. We cover:

- **Section A4.1**: Notation conventions used throughout
- **Section A4.2**: LMM (BGM) fundamentals—forward rate dynamics, measure changes, and the drift calculation
- **Section A4.3**: Swap Market Model (SMM)—swap rate dynamics and the Rebonato approximation
- **Section A4.4**: Connection to the HJM framework
- **Section A4.5**: SABR model for volatility smiles—the dominant framework for swaption quoting
- **Section A4.6**: Negative rate handling—Bachelier/Normal model and shifted lognormal approaches
- **Section A4.7**: Model selection guide for practitioners
- **Section A4.8**: Greeks computation—pathwise differentiation and likelihood ratio methods
- **Section A4.9**: Calibration methodology—regularization, the freezing approximation, and CMS adjustments
- **Section A4.10**: Simulation, pricing, and SOFR adaptation
- **Section A4.11**: Mathematical derivations
- **Section A4.12**: Worked examples
- **Section A4.13**: Summary

The material draws primarily from Brigo & Mercurio's *Interest Rate Models: Theory and Practice* and Andersen & Piterbarg's *Interest Rate Modeling*, with practitioner insights from desk experience.

**How to use this appendix (reading paths):**

- **If you’re here for caps/floors:** Sections A4.1–A4.2 (LMM setup + measure changes), then skim A4.9–A4.10 for calibration/simulation.
- **If you’re here for swaptions:** Section A4.3 (swap market model + Rebonato approximation), then Sections A4.5–A4.6 (SABR / normal vs shifted conventions).
- **If you keep getting lost in measures:** Read Appendix A1 (numeraires) and Appendix A3 (HJM drift restriction) first, then return to A4.2.2 and A4.4 here.
- **If you’re implementing Monte Carlo + Greeks:** Sections A4.8–A4.10 (Greeks + calibration + simulation).

**Prerequisites:** Appendix A1 (change of numeraire) and Appendix A3 (HJM viewpoint), plus comfort with basic swap PV/annuity algebra from Chapters 18–25.

---

## A4.1 Conventions and Notation

Before diving into the models, we establish notation that will be used consistently throughout this appendix.

### A4.1.1 Tenor Structure and Forward Rates

Let $0 = T_0 < T_1 < T_2 < \cdots < T_n$ denote a tenor structure with associated year fractions:
$$\tau_j = T_j - T_{j-1}, \quad j = 1, \ldots, n$$

The **simply-compounded forward rate** $L_j(t)$ for the period $[T_{j-1}, T_j]$, observed at time $t \leq T_{j-1}$, is defined implicitly by:
$$P(t, T_{j-1}) = P(t, T_j)(1 + \tau_j L_j(t))$$

where $P(t,T)$ denotes the time-$t$ price of a zero-coupon bond maturing at $T$.

Solving for the forward rate:
$$\boxed{L_j(t) = \frac{1}{\tau_j}\left(\frac{P(t, T_{j-1})}{P(t, T_j)} - 1\right)}$$

### A4.1.2 Swap Rates and Annuities

For a swap starting at $T_a$ and ending at $T_b$ (with $a < b$), the **swap rate** $S_{a,b}(t)$ is the fixed rate that makes the swap have zero present value at time $t$:
$$S_{a,b}(t) = \frac{P(t, T_a) - P(t, T_b)}{\sum_{j=a+1}^{b} \tau_j P(t, T_j)}$$

The denominator is the **annuity** (or **PVBP**):
$$A_{a,b}(t) = \sum_{j=a+1}^{b} \tau_j P(t, T_j)$$

Thus:
$$\boxed{S_{a,b}(t) = \frac{P(t, T_a) - P(t, T_b)}{A_{a,b}(t)}}$$

### A4.1.3 Probability Measures

Market models use specific probability measures chosen so that particular numeraires make the forward rate or swap rate a martingale:

| Measure | Numeraire | Key Property |
|---------|-----------|--------------|
| $\mathbb{Q}^j$ (Forward measure) | $P(t, T_j)$ | $L_j(t)$ is a martingale |
| $\mathbb{Q}^{a,b}$ (Swap measure) | $A_{a,b}(t)$ | $S_{a,b}(t)$ is a martingale |
| $\mathbb{Q}^{\mathrm{spot}}$ (Spot-LIBOR measure) | Discretely compounded bank account $B_d(t)$ | Convenient single measure for simulating the full tenor |
| $\mathbb{Q}^n$ (Terminal measure) | $P(t, T_n)$ | Alternative simulation measure (useful in some Bermudan setups) |

### A4.1.4 Black's Formula

Under the forward measure $\mathbb{Q}^j$, a caplet pays $\tau_j (L_j(T_{j-1}) - K)^+$ at time $T_j$. If $L_j(t)$ follows geometric Brownian motion with volatility $\sigma_j$, the caplet price is:
$$\text{Caplet}(t) = \tau_j P(t, T_j) \left[ L_j(t) \Phi(d_+) - K \Phi(d_-) \right]$$

where:
$$d_\pm = \frac{\ln(L_j(t)/K) \pm \frac{1}{2}\sigma_j^2(T_{j-1} - t)}{\sigma_j \sqrt{T_{j-1} - t}}$$

Similarly, for a payer swaption under $\mathbb{Q}^{a,b}$:
$$\text{Swaption}(t) = A_{a,b}(t) \left[ S_{a,b}(t) \Phi(d_+) - K \Phi(d_-) \right]$$

> **Desk Reality: Why Black's Formula Dominates**
>
> When a swaption trader quotes "22 vol on the 5y10y," they mean the Black implied volatility—the number you plug into the formula above to match the market price. The entire rates market uses this common language. The genius of market models is that they are constructed *specifically* so that Black's formula applies under the appropriate measure, making the connection between model dynamics and market quotes direct and intuitive.

---

## A4.2 LMM Essentials (LIBOR/BGM Market Model)

The LIBOR Market Model (LMM), developed independently by Brace, Gatarek, and Musiela (1997) and others, models the joint evolution of forward LIBOR rates under a common probability measure. Its key innovation is that each forward rate is a martingale under its own forward measure, ensuring that Black's formula applies directly to caplet pricing.

### A4.2.1 Forward Rate Dynamics Under the Forward Measure

Under the $T_j$-forward measure $\mathbb{Q}^j$, the forward rate $L_j(t)$ is a martingale. Brigo & Mercurio (Ch. 6) specify the dynamics as:
$$\boxed{dL_j(t) = \sigma_j(t) L_j(t) \, dW_j^j(t)}$$

where $W_j^j$ is a $\mathbb{Q}^j$-Brownian motion and $\sigma_j(t)$ is a deterministic (or state-dependent) volatility function.

This immediately implies that under $\mathbb{Q}^j$:
$$L_j(T_{j-1}) = L_j(t) \exp\left(-\frac{1}{2}\int_t^{T_{j-1}} \sigma_j(s)^2 ds + \int_t^{T_{j-1}} \sigma_j(s) dW_j^j(s)\right)$$

which is lognormal, and Black's formula applies exactly for caplets.

### A4.2.2 The Drift Adjustment: Changing Measures

The challenge arises when we want to simulate *multiple* forward rates jointly. Under a single measure, only one rate is a martingale; the others acquire drifts.

Suppose we work under the **spot-LIBOR measure** $\mathbb{Q}^{\mathrm{spot}}$, associated with a **discretely compounded bank account** (often denoted $B_d(t)$ in the literature). Brigo & Mercurio derive the drift adjustment via the change-of-numeraire / Girsanov toolkit.

To state the drift cleanly, define $\beta(t)$ as the index of the **first forward rate that is still stochastic at time $t$** (intuitively: the next reset date on the tenor schedule). Then, for $j \ge \beta(t)$:
$$\boxed{dL_j(t) = \mu_j(t) L_j(t) \, dt + \sigma_j(t) L_j(t) \, dW_j^{\mathrm{spot}}(t)}$$

where the drift is:
$$\mu_j(t) = \sum_{k=\beta(t)}^{j} \frac{\tau_k \rho_{jk} \sigma_j(t) \sigma_k(t) L_k(t)}{1 + \tau_k L_k(t)}$$

and $\rho_{jk}$ is the instantaneous correlation between the Brownian drivers of $L_j$ and $L_k$.

**The Coupled Drift Problem:** Notice that $\mu_j(t)$ depends on *other* forward rates $L_k(t)$. This creates a system of coupled SDEs that cannot be solved in closed form. Simulation is required for exotic pricing.

### A4.2.3 Terminal Measure Dynamics

Alternatively, under the terminal measure $\mathbb{Q}^n$ (numeraire $P(t, T_n)$), forward rates with $j < n$ have *negative* drifts:
$$dL_j(t) = -\sum_{k=j+1}^{n} \frac{\tau_k \rho_{jk} \sigma_j(t) \sigma_k(t) L_k(t)}{1 + \tau_k L_k(t)} L_j(t) dt + \sigma_j(t) L_j(t) \, dW_j^n(t)$$

> **Practitioner Note:** Terminal measure is sometimes used for Bermudan-style simulations because the numeraire is a fixed final bond and you have a single measure for all rates. Note that the drift typically depends on later-maturity rates, so “simplicity” is relative—choose the simulation measure that best matches your implementation and payoff structure.

### A4.2.4 Volatility and Correlation Structures

The LMM requires specification of:
1. **Volatility functions** $\sigma_j(t)$: How does the instantaneous volatility of $L_j$ evolve?
2. **Correlation matrix** $\rho_{jk}$: How are different forward rates correlated?

**Common volatility parameterizations:**

*Piecewise constant:*
$$\sigma_j(t) = \sigma_{j,k} \quad \text{for } T_{k-1} \leq t < T_k$$

*Parametric (e.g., Rebonato):*
$$\sigma_j(t) = (a + b(T_j - t)) e^{-c(T_j - t)} + d$$

**Common correlation structures:**

*Exponential decay:*
$$\rho_{jk} = e^{-\beta |j-k|}$$

*Two-factor approximation:*
$$\rho_{jk} = 1 - \gamma(1 - e^{-\delta |T_j - T_k|})$$

> **Desk Reality: The Correlation Problem**
>
> Traders often complain that LMM correlations are "wrong" for spread options and CMS products. The problem is that historical correlations, risk-neutral correlations, and correlations implied by spread options can all differ substantially. A common desk hack: calibrate correlations to spread options or CMS spread caps rather than trusting historical estimates.

### A4.2.5 Caplet Pricing: Black's Formula Directly

Because $L_j$ is lognormal under $\mathbb{Q}^j$, the caplet price is exactly:
$$\boxed{\text{Caplet}_j = \tau_j P(0, T_j) \left[ L_j(0) \Phi(d_+) - K \Phi(d_-) \right]}$$

where:
$$d_\pm = \frac{\ln(L_j(0)/K) \pm \frac{1}{2} v_j^2}{v_j}, \quad v_j = \sqrt{\int_0^{T_{j-1}} \sigma_j(t)^2 dt}$$

The cap price is the sum of caplets:
$$\text{Cap} = \sum_{j=1}^{n} \text{Caplet}_j$$

---

## A4.3 Swap Market Model Essentials

While the LMM is natural for caps/floors, swaptions are more directly handled by the **Swap Market Model (SMM)**, which models swap rates directly.

### A4.3.1 Swap Rate Dynamics Under the Swap Measure

Under the swap measure $\mathbb{Q}^{a,b}$ with numeraire $A_{a,b}(t)$, the swap rate $S_{a,b}(t)$ is a martingale. Assuming lognormal dynamics:
$$\boxed{dS_{a,b}(t) = \sigma_{a,b}(t) S_{a,b}(t) \, dW^{a,b}(t)}$$

This directly implies Black's swaption formula:
$$\text{Swaption}_{a,b}(0) = A_{a,b}(0) \left[ S_{a,b}(0) \Phi(d_+) - K \Phi(d_-) \right]$$

### A4.3.2 The Inconsistency Problem

Here's the rub: **different swap rates cannot all be lognormal under the same measure.** If $S_{1,5}$ is lognormal under $\mathbb{Q}^{1,5}$, then $S_{1,10}$ is *not* lognormal under $\mathbb{Q}^{1,10}$, and vice versa.

This is known as the **co-terminal swaption problem**. The swap market model is really a family of models, each exact for one particular swap rate but approximate for others.

### A4.3.3 Rebonato's Swaption Approximation: The Freezing Technique

Rebonato provides a powerful approximation that connects LMM dynamics to swaption volatilities. The key insight is that swap rates are complicated functions of forward rates:
$$S_{a,b}(t) = \frac{P(t, T_a) - P(t, T_b)}{A_{a,b}(t)} = \sum_{j=a+1}^{b} w_j(t)\,L_j(t)$$

where the weights $w_j(t)$ are annuity-normalized discount factors:
$$w_j(t) = \frac{\tau_j P(t, T_j)}{A_{a,b}(t)}, \qquad \sum_{j=a+1}^{b} w_j(t) = 1.$$

**The Freezing Approximation:** Freeze the weights at their time-0 values:
$$w_j(t) \approx w_j(0) = \frac{\tau_j P(0, T_j)}{A_{a,b}(0)}$$

Under this approximation, Brigo & Mercurio derive:
$$\boxed{v_{a,b}^2 T_a \approx \sum_{j,k=a+1}^{b} w_j(0) w_k(0) \rho_{jk} \int_0^{T_a} \sigma_j(t) \sigma_k(t) dt}$$

where $v_{a,b}$ is the swaption implied volatility.

> **Desk Reality: How Good Is Freezing?**
>
> Freezing is a **workhorse approximation** for relating an LMM calibration to a swaption surface. It is often a decent first pass for vanilla products, but it can degrade when:
> - Forward rates move significantly from initial values
> - Volatility is highly time-dependent
> - The swap has very long tenor

### A4.3.4 The Rebonato Formula in Practice

The Rebonato formula is used in two directions:

**Calibration direction:** Given LMM parameters ($\sigma_j$, $\rho_{jk}$), compute approximate swaption volatilities $v_{a,b}$ and compare to market.

**Diagnostic direction:** Given calibrated LMM and market swaption vols, check how well the model reproduces the swaption surface.

---

## A4.4 Connection to HJM

The LMM is a special case of the Heath-Jarrow-Morton (HJM) framework. Recall from Appendix A3 that HJM models the instantaneous forward rate $f(t,T)$ with dynamics:
$$df(t,T) = \alpha(t,T) dt + \sigma^{\text{HJM}}(t,T) dW(t)$$

where the no-arbitrage drift restriction is:
$$\alpha(t,T) = \sigma^{\text{HJM}}(t,T) \int_t^T \sigma^{\text{HJM}}(t,u) du$$

### A4.4.1 LMM as Discrete HJM

The forward LIBOR rate $L_j(t)$ relates to instantaneous forwards by:
$$1 + \tau_j L_j(t) = \frac{P(t, T_{j-1})}{P(t, T_j)} = \exp\left(\int_{T_{j-1}}^{T_j} f(t,u) du\right)$$

Applying Itô's lemma to this relationship and choosing specific HJM volatility structures yields the LMM dynamics. Specifically, choosing:
$$\sigma^{\text{HJM}}(t,T) = \sum_{j: T_{j-1} \leq T < T_j} \frac{\sigma_j(t) L_j(t)}{1 + \tau_j L_j(t)} \mathbf{1}_{t \leq T_{j-1}}$$

recovers the LMM.

### A4.4.2 Why This Matters

The HJM embedding ensures:
1. **No-arbitrage:** LMM inherits the no-arbitrage properties of HJM
2. **Consistency:** Bond prices, forward rates, and swap rates are all internally consistent
3. **Theoretical foundation:** The drift restriction emerges from fundamental no-arbitrage principles

---

## A4.5 The SABR Model for Volatility Smiles

While the basic LMM assumes constant or deterministic volatility, real swaption markets exhibit pronounced volatility smiles—implied volatility varies with strike. The SABR model (Stochastic Alpha, Beta, Rho) has become the dominant framework for capturing this smile.

### A4.5.1 SABR Model Dynamics

Brigo & Mercurio (Ch. 11) present the SABR model as developed by Hagan, Kumar, Lesniewski, and Woodward (2002). Under the appropriate forward or swap measure, the forward rate (or swap rate) $F$ evolves as:
$$\boxed{\begin{aligned}
dF(t) &= V(t) F(t)^\beta \, dZ(t) \\
dV(t) &= \nu V(t) \, dW(t) \\
V(0) &= \alpha
\end{aligned}}$$

where:
- $Z$ and $W$ are correlated Brownian motions with $dZ \cdot dW = \rho \, dt$
- $\beta \in [0, 1]$ controls the CEV-like backbone (with $\beta = 0$ corresponding to a normal/Bachelier-style backbone)
- $\alpha > 0$ is the initial volatility level
- $\nu > 0$ is the volatility-of-volatility ("vol-of-vol")
- $\rho \in [-1, 1]$ is the correlation between forward and volatility

### A4.5.2 The Hagan Approximation

The breakthrough of SABR is the closed-form approximation for implied volatility. For a European option with forward $F = F(0)$ and strike $K$:
$$\boxed{\sigma^{\text{impl}}(K, F) = \frac{\alpha}{(FK)^{(1-\beta)/2} \left[1 + \frac{(1-\beta)^2}{24}\ln^2\frac{F}{K} + \cdots\right]} \cdot \frac{z}{x(z)} \cdot \left\{1 + \epsilon(T)\right\}}$$

where:
$$z = \frac{\nu}{\alpha}(FK)^{(1-\beta)/2} \ln\frac{F}{K}$$
$$x(z) = \ln\left\{\frac{\sqrt{1 - 2\rho z + z^2} + z - \rho}{1 - \rho}\right\}$$

and the time-dependent correction is:
$$\epsilon(T) = \left[\frac{(1-\beta)^2 \alpha^2}{24(FK)^{1-\beta}} + \frac{\rho \beta \nu \alpha}{4(FK)^{(1-\beta)/2}} + \frac{(2 - 3\rho^2)\nu^2}{24}\right] T$$

**At-the-money ($K = F$):**
$$\boxed{\sigma^{\text{ATM}} = \frac{\alpha}{F^{1-\beta}} \left\{1 + \left[\frac{(1-\beta)^2 \alpha^2}{24 F^{2-2\beta}} + \frac{\rho \beta \nu \alpha}{4 F^{1-\beta}} + \frac{(2-3\rho^2)\nu^2}{24}\right] T\right\}}$$

### A4.5.3 Understanding the SABR Parameters

Each SABR parameter has intuitive meaning for the shape of the volatility smile:

| Parameter | What it does | Practical notes |
|-----------|--------------|-----------------|
| $\alpha$ | Overall level (pins ATM vol once $\beta$ is chosen) | Usually calibrated per expiry/tenor |
| $\beta$ | Backbone: how vol scales with the rate level | Often fixed to reduce instability; changing $\beta$ materially changes how ATM vol responds when rates move |
| $\rho$ | Skew direction and slope | Negative $\rho$ typically produces a downward-sloping smile (higher vol for low strikes) |
| $\nu$ | Curvature / wing strength (vol-of-vol) | Larger $\nu$ typically fattens wings (more curvature) |

**The skew decomposition:** Brigo & Mercurio show that the skew comes from two sources:
1. **Beta skew:** Proportional to $(1 - \beta)$; a pure CEV effect
2. **Vanna skew:** Proportional to $\rho\nu$; the correlation effect

> **Desk Reality: SABR Calibration Practice**
>
> A common desk approach to SABR calibration is:
> 1. **Fix $\beta$:** Often set $\beta = 0.5$ (square-root) or $\beta = 1$ (lognormal). The choice is usually made once and held constant across the surface.
> 2. **Calibrate $\alpha$ to ATM:** For each expiry-tenor pair, find $\alpha$ that matches ATM vol.
> 3. **Calibrate $\rho, \nu$ to smile:** Fit the remaining parameters to OTM quotes.
>
> Why fix $\beta$? Because $\beta$ and the other parameters are poorly identified—many ($\beta$, $\rho$, $\nu$) combinations fit the same smile. Fixing $\beta$ breaks this degeneracy.

### A4.5.4 SABR for Swaptions

For swaptions, replace the forward rate $F$ with the swap rate $S_{a,b}$. The dynamics under the swap measure $\mathbb{Q}^{a,b}$ are:
$$\begin{aligned}
dS_{a,b}(t) &= V(t) S_{a,b}(t)^\beta \, dZ^{a,b}(t) \\
dV(t) &= \nu V(t) \, dW^{a,b}(t)
\end{aligned}$$

The Hagan formula applies directly with $F \to S_{a,b}(0)$ and $T \to T_a$ (swaption expiry).

### A4.5.5 SABR Limitations

The SABR model, while practical, has known issues:

1. **Arbitrage at extreme strikes:** The Hagan approximation can produce negative implied densities for very OTM options
2. **Very long-dated maturities:** The approximation can degrade as $T$ grows large
3. **Negative rates:** Standard SABR assumes $F > 0$; see Section A4.6 for extensions
4. **No term structure of parameters:** Each expiry is calibrated independently; no dynamic consistency

---

## A4.6 Negative Rate Handling

Strict Black/log-normal models assume the underlying rate stays positive. When rates are near zero (and can plausibly go negative), desks typically switch to models and quoting conventions that remain well-defined around zero: **normal (Bachelier)** pricing, or **shifted/displaced** log-normal constructions, often combined with SABR-style smiles.

### A4.6.1 The Bachelier (Normal) Model

The simplest solution is the **Bachelier model** (also called Normal or Gaussian model), where the forward rate follows arithmetic Brownian motion:
$$dF(t) = \sigma_N \, dW(t)$$

Here $\sigma_N$ is the **normal volatility** or **basis-point volatility**, measured in absolute rate terms (e.g., 50 bp/year).

Andersen & Piterbarg provide the Normal option pricing formula. For a call option:
$$\boxed{c_N(t, F; T, K; \sigma_N) = (F - K)\Phi(d) + \sigma_N\sqrt{T-t} \, \phi(d)}$$

where:
$$d = \frac{F - K}{\sigma_N\sqrt{T-t}}$$

and $\Phi(\cdot)$, $\phi(\cdot)$ are the standard normal CDF and PDF.

**PV conversion:** This is the option price in forward units. Multiply by the appropriate numeraire/discount factor to get PV (e.g., caplet PV uses $\tau_j P(t,T_j)$; swaption PV uses the annuity $A_{a,b}(t)$).

**Key properties:**
- The forward rate can go negative
- Volatility is quoted in absolute terms (bp), not relative terms (%)
- The model is simple but doesn't produce realistic smiles

> **Desk Reality: Normal Vol Quoting**
>
> In some markets, swaptions are quoted in **normal (bp) vol** terms, especially when rates are low/near-zero. A trader saying “the 10y10y is at 60 bp vol” means $\sigma_N = 0.60\\%$ per year in the Bachelier model. Always confirm whether a quote is Black vol (%) or normal vol (bp).

### A4.6.2 Shifted Lognormal (Displaced Diffusion)

A popular middle ground is the **shifted lognormal** or **displaced diffusion** model:
$$dF(t) = \sigma (F(t) + s) \, dW(t)$$

where $s > 0$ is the **shift parameter**. The shifted forward $\tilde{F} = F + s$ follows geometric Brownian motion, so Black's formula applies to $\tilde{F}$:
$$c_{\text{shifted}}(F, K) = c_{\text{Black}}(F + s, K + s)$$

Andersen & Piterbarg give the explicit formula (Proposition 7.2.12):
$$\boxed{c_{\text{DLN}}(t, F; T, K; \sigma, s) = (F + s)\Phi(d_+) - (K + s)\Phi(d_-)}$$

where:
$$d_\pm = \frac{\ln\frac{F + s}{K + s} \pm \frac{1}{2}\sigma^2(T-t)}{\sigma\sqrt{T-t}}$$

**PV conversion:** As with Black, this is a forward-measure style expression. Multiply by the relevant discount factor/annuity for caplets/swaptions.

**The shift parameter:**
- Must be large enough that $F + s > 0$ for all scenarios
- Different desks can use different shifts; always confirm $s$ when comparing “implied vols” across systems

### A4.6.3 Shifted SABR

For smile modeling with negative rates, the market adopted **shifted SABR**:
$$\begin{aligned}
d(F(t) + s) &= V(t) (F(t) + s)^\beta \, dZ(t) \\
dV(t) &= \nu V(t) \, dW(t)
\end{aligned}$$

Apply the standard SABR formula with $F \to F + s$ and $K \to K + s$.

> **Desk Reality: The Shift Coordination Problem**
>
> Different desks can agree on a swaption price but disagree on the “implied vol” if they use different shifts. When you reconcile quotes, always bring the discussion back to **price** (or vega) and explicitly state the shift convention used.

### A4.6.4 Normal SABR

An alternative is **Normal SABR** (Bachelier SABR), which uses arithmetic rather than geometric dynamics:
$$\begin{aligned}
dF(t) &= V(t) \, dZ(t) \\
dV(t) &= \nu V(t) \, dW(t)
\end{aligned}$$

This is essentially SABR with $\beta = 0$. The Hagan approximation gives normal implied volatility directly.

### A4.6.5 Summary: Model Choice for Negative Rates

| Model | Handles Negative Rates? | Smile? | Common Usage |
|-------|------------------------|--------|--------------|
| Black (lognormal) | No | Via SABR | Log-normal quoting conventions; positive-rate setups |
| Bachelier (normal) | Yes | Limited (by itself) | Low/near-zero rate environments; spread-style payoffs |
| Shifted lognormal | Yes | Via shifted SABR | Keep a Black-style backbone while allowing modest negatives |
| Normal SABR | Yes | Yes | Smile modeling when rates can go negative (Bachelier backbone) |

---

## A4.7 Model Selection Guide for Practitioners

A perennial question from junior quants moving to the desk: "Which model should I use?" The answer depends on the product, the market, and the use case.

### A4.7.1 Product-Based Selection

| Product | Recommended Model | Why |
|---------|------------------|-----|
| **Caps/Floors** | LMM + SABR | Natural fit; Black's formula applies directly |
| **European Swaptions** | SABR (standalone) | Industry standard; fast calibration |
| **Bermudan Swaptions** | LMM with local vol or SV | Need dynamics for exercise boundary |
| **CMS Products** | LMM or SMM with convexity | Requires distribution under wrong measure |
| **Spread Options** | LMM with correlation | Need joint rate dynamics |
| **Callable Range Accruals** | Full LMM simulation | Path-dependent payoff |

### A4.7.2 The "Talking to the Traders" Perspective

Brigo & Mercurio include a fascinating appendix interviewing rates traders about model choice. Key insights:

- **Match complexity to the product:** vanilla books often rely on fast closed-form pricing (Black-style formulas under the right measure), while exotics require simulation.
- **Separate “quoting model” from “pricing dynamics”:** SABR-like parameterizations are often used to fit/interpolate a volatility surface, while a calibrated multi-rate dynamics model (e.g., LMM) is used to generate paths and price path-dependent payoffs.
- **Be explicit about conventions near zero:** normal vs shifted-lognormal quoting and parameter choices can materially affect “implied vols” even when prices agree. Always confirm what the desk/system uses.

### A4.7.3 Use Case Matrix

| Use Case | Model | Notes |
|----------|-------|-------|
| **Market-making vanilla** | SABR | Fast, standard quotes |
| **Exotic pricing** | Calibrated LMM | Full simulation required |
| **Risk sensitivities** | Same as pricing | Consistency is key |
| **XVA calculation** | LMM in multi-curve | Needs realistic dynamics |
| **Model validation** | Multiple models | Compare prices across models |

### A4.7.4 Red Flags: When Your Model Is Wrong

Watch for these signs that your model choice is inappropriate:

1. **Negative probabilities:** Implied density goes negative (SABR at extreme strikes)
2. **Arbitrage:** Calendar spread or butterfly arbitrage in the vol surface
3. **Unstable Greeks:** Delta or vega changes sign unexpectedly
4. **Poor hedging P&L:** Large unexplained P&L after hedging
5. **Calibration failure:** Cannot match liquid market quotes

> **Desk Reality: The Model Risk Hierarchy**
>
> Most desks operate a model hierarchy:
> - **Tier 1 (production):** The main pricing model, heavily validated
> - **Tier 2 (benchmark):** Alternative model for comparison
> - **Tier 3 (research):** New models under development
>
> When Tier 1 and Tier 2 disagree significantly, that's a red flag requiring investigation.

---

## A4.8 Greeks Computation

Once we have a pricing model, we need sensitivities for hedging. Market models present challenges because simulation is required for most exotics.

### A4.8.1 Analytical Greeks for Vanilla Products

For caplets and swaptions under Black's formula, Greeks are analytical.

**Caplet Greeks (Black model):**
$$\Delta_{\text{caplet}} = \frac{\partial}{\partial L_j} \text{Caplet} = \tau_j P(0, T_j) \Phi(d_+)$$

$$\text{Vega}_{\text{caplet}} = \tau_j P(0, T_j) L_j \phi(d_+) \sqrt{T_{j-1}}$$

**Swaption Greeks:**
$$\Delta_{\text{swaption}} = A_{a,b}(0) \Phi(d_+)$$

$$\text{Vega}_{\text{swaption}} = A_{a,b}(0) S_{a,b}(0) \phi(d_+) \sqrt{T_a}$$

### A4.8.2 Simulation-Based Greeks: The Problem

For exotics priced by simulation, the price is:
$$V_0 = \mathbb{E}[\text{Payoff}(L_1, \ldots, L_n)]$$

How do we compute $\partial V_0 / \partial L_j(0)$?

**Finite differences (bump-and-reprice):** Shift $L_j(0)$ by $\delta$, reprice, and compute:
$$\frac{\partial V_0}{\partial L_j(0)} \approx \frac{V_0(L_j(0) + \delta) - V_0(L_j(0) - \delta)}{2\delta}$$

Problems: Requires 2n simulations for n Greeks; noisy for small $\delta$.

### A4.8.3 Pathwise Differentiation

Andersen & Piterbarg (Section 3.3.2) present **pathwise differentiation** (also called IPA—Infinitesimal Perturbation Analysis):

$$\frac{\partial V_0}{\partial \theta} = \mathbb{E}\left[\frac{\partial}{\partial \theta} \text{Payoff}\right]$$

**Key requirement:** The payoff must be almost everywhere differentiable in the parameter.

**How it works:** Differentiate the SDE paths:
$$\frac{\partial L_j(t)}{\partial L_j(0)} = L_j(t) / L_j(0)$$

Then propagate through the payoff.

**Advantage:** One simulation gives all Greeks simultaneously.

**Limitation:** Fails for discontinuous payoffs (barriers, digitals).

### A4.8.4 Likelihood Ratio Method

For discontinuous payoffs, Andersen & Piterbarg (Section 3.3.3) present the **likelihood ratio method** (also called score function method):

$$\frac{\partial V_0}{\partial \theta} = \mathbb{E}\left[\text{Payoff} \cdot \frac{\partial}{\partial \theta} \ln p(\omega; \theta)\right]$$

where $p(\omega; \theta)$ is the probability density of the simulation path.

**Advantage:** Works for any payoff, including discontinuous.

**Disadvantage:** Higher variance than pathwise; requires density knowledge.

### A4.8.5 Practical Greek Computation

| Greek | Vanilla Method | Exotic Method |
|-------|---------------|---------------|
| Delta | Analytical | Pathwise preferred |
| Vega | Analytical | Pathwise or bump |
| Gamma | Analytical | Bump or adjoint |
| Cross-gamma | Analytical (limited) | Bump-and-reprice |

> **Desk Reality: AAD for Greeks**
>
> Modern desks increasingly use **Adjoint Algorithmic Differentiation (AAD)** to compute all Greeks in a single backward pass through the pricing code. This is particularly valuable for XVA calculations where thousands of Greeks are needed. If you're building a new rates library, consider AAD from the start.

---

## A4.9 Calibration Methodology

Calibration is the art and science of fitting model parameters to market data. For market models, this means matching cap/floor and swaption volatilities.

### A4.9.1 The Calibration Problem

**Given:**
- Market cap volatilities $\hat{v}_j^{\text{cap}}$ for maturities $T_1, \ldots, T_n$
- Market swaption volatilities $\hat{v}_{a,b}^{\text{swap}}$ for expiry-tenor pairs

**Find:**
- LMM volatilities $\sigma_j(t)$
- Correlations $\rho_{jk}$

Such that model prices match market prices.

### A4.9.2 Calibrating to Caps

Caps are the "easy" calibration target because each caplet depends on only one forward rate's volatility. The relationship is:
$$v_j^{\text{cap}} = \sqrt{\frac{1}{T_{j-1}} \int_0^{T_{j-1}} \sigma_j(t)^2 \, dt}$$

**Piecewise constant volatility:** If $\sigma_j(t) = \sigma_{j,k}$ for $t \in [T_{k-1}, T_k)$:
$$v_j^2 T_{j-1} = \sum_{k=1}^{j-1} \sigma_{j,k}^2 \tau_k$$

This can be inverted sequentially:
$$\sigma_{j,j-1}^2 = \frac{v_j^2 T_{j-1} - \sum_{k=1}^{j-2} \sigma_{j,k}^2 \tau_k}{\tau_{j-1}}$$

### A4.9.3 Calibrating to Swaptions: The Rebonato Approach

Swaptions involve correlations and multiple forward rates. Using the Rebonato approximation:
$$v_{a,b}^2 T_a \approx \sum_{j,k=a+1}^{b} w_j w_k \rho_{jk} \int_0^{T_a} \sigma_j(t) \sigma_k(t) \, dt$$

**Strategy 1: Fix correlations, fit volatilities**
- Assume a correlation structure (e.g., exponential decay)
- Solve for volatilities that match swaption vols

**Strategy 2: Joint calibration**
- Parameterize both volatilities and correlations
- Minimize sum of squared pricing errors

### A4.9.4 Regularization Techniques

Calibration is often ill-posed—many parameter combinations fit the data equally well. Regularization imposes additional structure.

**Tikhonov regularization:** Add a penalty term:
$$\min_\theta \sum_i (V_i^{\text{model}}(\theta) - V_i^{\text{market}})^2 + \lambda \|\theta\|^2$$

The regularization parameter $\lambda$ controls the smoothness vs. fit tradeoff.

**Eigenvalue zeroing:** For correlation matrices:
1. Compute eigendecomposition $\rho = Q \Lambda Q^T$
2. Zero small eigenvalues (those below some threshold)
3. Reconstruct: $\tilde{\rho} = Q \tilde{\Lambda} Q^T$

This produces a lower-rank correlation structure that is more stable.

### A4.9.5 The Freezing Approximation: Error Analysis

The freezing approximation is a practical shortcut, not a theorem. Its accuracy depends on how much the weights $w_j(t)$ move away from their time-0 values.

**When freezing works well:**
- Short-dated swaptions
- Low volatility environments
- Moderate rate levels

**When freezing degrades:**
- Long-dated swaptions
- High volatility
- Extreme rate levels
- Steeply sloped yield curves

> **Practitioner Note:** If freezing errors concern you, validate the approximation on your product set via Monte Carlo (or via a higher-fidelity numerical scheme) and treat Rebonato as a diagnostic, not as ground truth.

### A4.9.6 CMS Convexity Adjustment

Constant Maturity Swap (CMS) products pay the swap rate observed at one date but pay at a different date. This creates a convexity adjustment because the swap rate is observed under one measure but paid under another.

**The problem:** A CMS coupon pays $S_{a,b}(T_p)$ at time $T_p$, but the "natural" numeraire for $S_{a,b}$ is the annuity $A_{a,b}$, not $P(\cdot, T_p)$.

**The adjustment:**
$$\mathbb{E}^{T_p}[S_{a,b}(T)] = S_{a,b}(0) + \text{Convexity Adjustment}$$

Using a change of measure from $\mathbb{Q}^{a,b}$ to $\mathbb{Q}^{T_p}$, Brigo & Mercurio derive approximations for this adjustment involving:
- The correlation between swap rate and annuity
- The volatility of the swap rate
- The time to payment

For practical CMS pricing, desks often use replication arguments—expressing the CMS rate as an integral over swaptions—rather than explicit convexity formulas.

---

## A4.10 Simulation, Pricing, and SOFR Adaptation

### A4.10.1 Euler Discretization

For simulation, we discretize the LMM SDE. Under the spot-LIBOR measure $\mathbb{Q}^{\mathrm{spot}}$ with time steps $\Delta t$:
$$L_j(t + \Delta t) = L_j(t) \exp\left[\left(\mu_j(t) - \frac{1}{2}\sigma_j(t)^2\right)\Delta t + \sigma_j(t)\sqrt{\Delta t} \, Z_j\right]$$

where $Z_j$ are correlated standard normals: $\mathbb{E}[Z_j Z_k] = \rho_{jk}$.

**Generating correlated normals:** Use Cholesky decomposition:
$$Z = L \cdot \epsilon$$

where $\epsilon$ is a vector of independent standard normals and $LL^T = \rho$.

### A4.10.2 Predictor-Corrector for Drift

The drift $\mu_j(t)$ depends on $L_k(t)$, creating a chicken-and-egg problem. Solution: **predictor-corrector**:

1. **Predict:** Use $L_k(t)$ to compute $\mu_j(t)$; step to get $\tilde{L}_j(t+\Delta t)$
2. **Correct:** Average the drifts at $t$ and $t+\Delta t$ using predicted values
3. **Final step:** Use the averaged drift

### A4.10.3 Bermudan Swaption Pricing: Longstaff-Schwartz

Bermudan swaptions allow exercise at multiple dates. Pricing requires solving an optimal stopping problem.

**Longstaff-Schwartz (LSM) algorithm:**
1. Simulate forward rate paths
2. At each exercise date $T_k$, going backward:
   a. Compute exercise value: $\text{Swap Value}_{T_k}$
   b. Regress continuation value on basis functions of state variables
   c. Exercise if exercise value > continuation value
3. Average discounted payoffs across paths

**Basis functions:** Common choices include:
- Low-order polynomials of swap rate
- Swap rate and intrinsic value
- Principal components of forward rates

> **Desk Reality: LSM Noise and Bias**
>
> LSM is notoriously noisy for Bermudans. Desks typically use:
> - Large path counts
> - Antithetic variates
> - Control variates (European swaption as control)
> - Boundary smoothing techniques
>
> Despite these, LSM can still be biased high (due to using in-sample regression). Always validate against PDE or tree methods when available.

### A4.10.4 SOFR Adaptation: The Multi-Curve Framework

In an RFR (risk-free rate) world, market-model implementations need to respect **multi-curve reality**: discounting and projection can use different curves (see Chapters 18–22 for curve construction and multi-curve plumbing).

**Key conceptual difference vs LIBOR-style forwards:** many RFR floating coupons are **compounded in arrears** (built from daily overnight fixings) rather than being a single forward-looking fixing.

**Compounded-in-arrears coupon (illustrative form):**
$$R_{j}(T_{j-1},T_j)=\frac{1}{\tau_j}\left(\prod_{d \in [T_{j-1}, T_j)} (1 + \delta_d r_d) - 1\right)$$

where $r_d$ is the daily SOFR and $\delta_d$ is the daily accrual.

**Modeling approach (high level):** You have several consistent choices, depending on product and system:
1. **Market-model proxy:** model a simply-compounded forward-like rate on the tenor grid (LMM-style) and map it to the compounded coupon.
2. **Short-rate/HJM viewpoint:** model the short rate (or instantaneous forwards) and build the compounded coupon as an integral/product functional.
3. **Direct modeling:** treat the compounded rate itself as the primitive (less common, but conceptually clean).

**Two-curve framework:**
- **Discount curve:** OIS curve (based on overnight rates)
- **Projection curve:** SOFR curve (for floating payments)

**Practical reminder:** even when you simulate under a convenient measure (spot/terminal), the discounting curve used for PV should be consistent with the collateral/CSA assumptions of the trade.

---

## A4.11 Mathematical Derivations

### A4.11.1 Derivation of the LMM Drift

Under the spot-LIBOR measure $\mathbb{Q}^{\mathrm{spot}}$, we derive the drift of $L_j(t)$ using the change-of-numeraire / Girsanov toolkit.

**Setup:** Under $\mathbb{Q}^j$, $L_j$ is a martingale:
$$dL_j(t) = \sigma_j(t) L_j(t) \, dW_j^j(t)$$

**Key idea:** changing measures changes drifts by a covariance term involving the relative volatility of the numeraire ratio.

**Applying Girsanov (high level):** The drift adjustment for $L_j$ under $\mathbb{Q}^{\mathrm{spot}}$ can be written schematically as
$$\mu_j(t) = \frac{d\langle L_j, M \rangle_t}{L_j(t) \, dt}$$

where $M$ is the martingale part coming from the numeraire change (details in the references).

**Result (spot-LIBOR measure drift):** Define $\beta(t)$ as the index of the first forward rate still stochastic at time $t$. Then the spot-measure drift of $L_j$ is
$$\boxed{\mu_j(t) = \sum_{k=\beta(t)}^{j} \frac{\tau_k \rho_{jk} \sigma_j(t) \sigma_k(t) L_k(t)}{1 + \tau_k L_k(t)}}$$
which is the coupled-drift term used in LMM simulation.

### A4.11.2 Derivation of the Rebonato Formula

**Starting point:** The swap rate as a function of forwards:
$$S_{a,b}(t) = \sum_{j=a+1}^{b} w_j(t) L_j(t)$$

where $w_j(t) = \tau_j P(t, T_j) / A_{a,b}(t)$ and $\sum_j w_j(t) = 1$.

**Applying Itô's lemma:**
$$dS_{a,b}(t) = \sum_j \left(\frac{\partial S}{\partial L_j} dL_j + \frac{\partial S}{\partial w_j} dw_j\right) + \text{cross terms}$$

**Freezing the weights:** Set $w_j(t) \approx w_j(0)$:
$$dS_{a,b}(t) \approx \sum_j w_j(0) \, dL_j(t)$$

**Under swap measure:** $S_{a,b}$ is approximately lognormal with:
$$\begin{aligned}
\text{Var}[\ln S_{a,b}(T_a)] &= \int_0^{T_a} \frac{(\sum_j w_j \sigma_j L_j)^2}{S_{a,b}^2} \, dt \\
&\approx \sum_{j,k} w_j w_k \rho_{jk} \int_0^{T_a} \sigma_j(t) \sigma_k(t) \, dt
\end{aligned}$$

The swaption implied vol is:
$$v_{a,b}^2 T_a = \sum_{j,k=a+1}^{b} w_j(0) w_k(0) \rho_{jk} \int_0^{T_a} \sigma_j(t) \sigma_k(t) \, dt$$

### A4.11.3 SABR Asymptotic Expansion

The Hagan formula is derived using singular perturbation theory. The key steps:

1. **Rescale:** Let $\varepsilon = \nu$ be small; write expansion in powers of $\varepsilon$
2. **Leading order:** Solve the CEV problem (Black with $\beta$)
3. **First correction:** Account for stochastic volatility perturbation
4. **Match boundary conditions:** Ensure option prices are correct at $K = F$

The result is the formula given in Section A4.5.2. The expansion is accurate to $O(\varepsilon^2)$ and $O(T^2)$.

---

## A4.12 Worked Examples

### Example 1: Caplet Pricing with LMM

**Given:**
- Forward rate $L_3(0) = 5\%$ for the period $[T_2, T_3]$ with $\tau_3 = 0.25$ (quarterly)
- Discount factor $P(0, T_3) = 0.95$
- Caplet expiry $T_2 = 0.5$ years
- Strike $K = 5\%$
- Black volatility $v = 20\%$

**Solution:**

Integrated variance: $v^2 T_2 = 0.20^2 \times 0.5 = 0.02$, so $v\sqrt{T_2} = 0.1414$.

$$d_+ = \frac{\ln(0.05/0.05) + 0.5 \times 0.02}{0.1414} = \frac{0.01}{0.1414} = 0.0707$$
$$d_- = 0.0707 - 0.1414 = -0.0707$$

$$\Phi(0.0707) = 0.5282, \quad \Phi(-0.0707) = 0.4718$$

$$\text{Caplet} = 0.25 \times 0.95 \times [0.05 \times 0.5282 - 0.05 \times 0.4718]$$
$$= 0.2375 \times 0.05 \times 0.0564 = 0.000670 = 6.70 \text{ bp}$$

### Example 2: Swaption Pricing with SABR

**Given:**
- 1y-into-5y payer swaption
- Forward swap rate $S = 4\%$
- Strike $K = 4.5\%$ (50 bp OTM)
- SABR parameters: $\alpha = 0.25$, $\beta = 0.5$, $\rho = -0.3$, $\nu = 0.4$
- Annuity $A_{1,6}(0) = 4.2$
- Expiry $T = 1$ year

**Solution:**

Step 1: Compute SABR implied volatility at $K = 4.5\%$.

$(FK)^{(1-\beta)/2} = (0.04 \times 0.045)^{0.25} = 0.0424^{0.25} = 0.454$

$z = \frac{0.4}{0.25} \times 0.454 \times \ln(0.04/0.045) = 1.6 \times 0.454 \times (-0.1178) = -0.0856$

$x(z) = \ln\left\{\frac{\sqrt{1 + 0.6 \times 0.0856 + 0.0856^2} - 0.0856 + 0.3}{1.3}\right\}$
$= \ln\left\{\frac{1.025 - 0.0856 + 0.3}{1.3}\right\} = \ln(0.954) = -0.047$

$z/x(z) = -0.0856 / (-0.047) = 1.82$

Leading term: $\frac{\alpha}{(FK)^{(1-\beta)/2}} = \frac{0.25}{0.454} = 0.551$

Time correction (simplified): $\epsilon(T) \approx 0.02$

$\sigma^{\text{impl}} \approx 0.551 \times 1.82 \times 1.02 \approx 1.02 = 102\%$

Step 2: Price using Black's formula with this implied vol.

$v\sqrt{T} = 1.02 \times 1 = 1.02$

$d_+ = \frac{\ln(0.04/0.045) + 0.5 \times 1.02^2}{1.02} = \frac{-0.1178 + 0.520}{1.02} = 0.394$
$d_- = 0.394 - 1.02 = -0.626$

$\Phi(0.394) = 0.653, \quad \Phi(-0.626) = 0.266$

$\text{Swaption} = 4.2 \times [0.04 \times 0.653 - 0.045 \times 0.266]$
$= 4.2 \times [0.0261 - 0.0120] = 4.2 \times 0.0141 = 0.0593 = 5.93\%$

### Example 3: Normal Vol Conversion

**Given:**
- Swap rate $S = 3\%$
- Black implied vol $\sigma_{\text{Black}} = 25\%$
- Expiry $T = 2$ years

**Find:** Equivalent normal (Bachelier) volatility.

**Solution:**

For ATM options, the approximate conversion is:
$$\sigma_N \approx \sigma_{\text{Black}} \times S = 0.25 \times 0.03 = 0.0075 = 75 \text{ bp}$$

More precisely, using the ATM vega equivalence:
$$\sigma_N = S \cdot \sigma_{\text{Black}} \cdot \sqrt{\frac{2\pi}{e}} \cdot \frac{1}{\sqrt{2\pi}} = S \cdot \sigma_{\text{Black}}$$

So 25% Black vol at 3% swap rate ≈ 75 bp normal vol.

### Example 4: Shifted Lognormal Pricing

**Given:**
- Forward rate $F = -0.2\%$ (negative!)
- Strike $K = 0\%$
- Shift $s = 2\%$
- Shifted Black vol $\sigma = 30\%$
- Expiry $T = 1$ year
- Discount factor $P(0,T) = 1.002$
- Year fraction $\tau = 1$

**Solution:**

Shifted forward: $F + s = -0.2\% + 2\% = 1.8\%$
Shifted strike: $K + s = 0\% + 2\% = 2\%$

$d_+ = \frac{\ln(0.018/0.02) + 0.5 \times 0.30^2 \times 1}{0.30 \times 1} = \frac{-0.1054 + 0.045}{0.30} = -0.201$
$d_- = -0.201 - 0.30 = -0.501$

$\Phi(-0.201) = 0.420, \quad \Phi(-0.501) = 0.308$

Caplet (call on rate):
$= \tau \times P(0,T) \times [(F+s)\Phi(d_+) - (K+s)\Phi(d_-)]$
$= 1 \times 1.002 \times [0.018 \times 0.420 - 0.02 \times 0.308]$
$= 1.002 \times [0.00756 - 0.00616] = 1.002 \times 0.00140 = 0.140\%$

### Example 5: LMM Drift Calculation

**Given:**
- Three forward rates: $L_1 = 4\%$, $L_2 = 4.2\%$, $L_3 = 4.5\%$
- Year fractions: $\tau_1 = \tau_2 = \tau_3 = 0.5$
- Volatilities: $\sigma_1 = 18\%$, $\sigma_2 = 20\%$, $\sigma_3 = 22\%$
- Correlations: $\rho_{12} = 0.95$, $\rho_{13} = 0.85$, $\rho_{23} = 0.92$
- Simulating under spot measure starting at $T_0$

**Find:** Drift of $L_3(t)$ at time $t = 0$.

**Solution:**

$$\mu_3(0) = \sum_{k=1}^{3} \frac{\tau_k \rho_{3k} \sigma_3 \sigma_k L_k}{1 + \tau_k L_k}$$

$k=1$: $\frac{0.5 \times 0.85 \times 0.22 \times 0.18 \times 0.04}{1 + 0.5 \times 0.04} = \frac{0.000673}{1.02} = 0.000660$

$k=2$: $\frac{0.5 \times 0.92 \times 0.22 \times 0.20 \times 0.042}{1 + 0.5 \times 0.042} = \frac{0.000850}{1.021} = 0.000833$

$k=3$: $\frac{0.5 \times 1.0 \times 0.22 \times 0.22 \times 0.045}{1 + 0.5 \times 0.045} = \frac{0.001089}{1.0225} = 0.001065$

$\mu_3(0) = 0.000660 + 0.000833 + 0.001065 = 0.00256 = 0.256\%$

### Example 6: Rebonato Swaption Vol Approximation

**Given:**
- 2y-into-3y swaption (swap from $T_2$ to $T_5$)
- Forward rates: $L_3 = 4\%$, $L_4 = 4.2\%$, $L_5 = 4.4\%$
- Weights (frozen at $t=0$): $w_3 = 0.32$, $w_4 = 0.33$, $w_5 = 0.35$
- Volatilities: $\sigma_3 = 20\%$, $\sigma_4 = 19\%$, $\sigma_5 = 18\%$
- All correlations $\rho_{jk} = 0.9$
- Expiry $T_a = 2$ years

**Find:** Approximate swaption implied volatility.

**Solution:**

$$v_{2,5}^2 \times 2 = \sum_{j,k=3}^{5} w_j w_k \rho_{jk} \int_0^2 \sigma_j \sigma_k \, dt$$

With constant volatilities, $\int_0^2 \sigma_j \sigma_k \, dt = 2 \sigma_j \sigma_k$.

Diagonal terms ($j=k$, $\rho=1$):
- $w_3^2 \times 1 \times 2 \times 0.20^2 = 0.32^2 \times 0.08 = 0.00819$
- $w_4^2 \times 1 \times 2 \times 0.19^2 = 0.33^2 \times 0.0722 = 0.00787$
- $w_5^2 \times 1 \times 2 \times 0.18^2 = 0.35^2 \times 0.0648 = 0.00794$

Cross terms ($j \neq k$, $\rho=0.9$), factor of 2 for symmetry:
- $2 \times w_3 w_4 \times 0.9 \times 2 \times 0.20 \times 0.19 = 2 \times 0.32 \times 0.33 \times 0.9 \times 0.076 = 0.01446$
- $2 \times w_3 w_5 \times 0.9 \times 2 \times 0.20 \times 0.18 = 2 \times 0.32 \times 0.35 \times 0.9 \times 0.072 = 0.01452$
- $2 \times w_4 w_5 \times 0.9 \times 2 \times 0.19 \times 0.18 = 2 \times 0.33 \times 0.35 \times 0.9 \times 0.0684 = 0.01423$

Total: $0.00819 + 0.00787 + 0.00794 + 0.01446 + 0.01452 + 0.01423 = 0.06721$

$v_{2,5}^2 \times 2 = 0.06721$
$v_{2,5} = \sqrt{0.06721/2} = \sqrt{0.0336} = 18.3\%$

### Example 7: SABR Calibration

**Given:** ATM swaption volatility $\sigma^{\text{ATM}} = 20\%$, swap rate $S = 5\%$, expiry $T = 1$ year, $\beta = 0.5$ (fixed).

**Find:** SABR $\alpha$ parameter (ignoring higher-order terms).

**Solution:**

From the ATM formula:
$$\sigma^{\text{ATM}} \approx \frac{\alpha}{S^{1-\beta}} = \frac{\alpha}{0.05^{0.5}} = \frac{\alpha}{0.2236}$$

$$\alpha = 0.20 \times 0.2236 = 0.0447$$

### Example 8: Normal Option Price

**Given:**
- Forward swap rate $S = 2\%$
- Strike $K = 2.5\%$
- Normal volatility $\sigma_N = 80$ bp = 0.80%
- Expiry $T = 1$ year
- Annuity $A = 4.5$

**Find:** Payer swaption price.

**Solution:**

$d = \frac{S - K}{\sigma_N \sqrt{T}} = \frac{0.02 - 0.025}{0.008 \times 1} = \frac{-0.005}{0.008} = -0.625$

$\Phi(-0.625) = 0.266$
$\phi(-0.625) = 0.327$

$c_N = (S - K)\Phi(d) + \sigma_N \sqrt{T} \phi(d)$
$= (-0.005) \times 0.266 + 0.008 \times 0.327$
$= -0.00133 + 0.00262 = 0.00129$

Swaption = $A \times c_N = 4.5 \times 0.00129 = 0.58\%$

### Example 9: Pathwise Delta Calculation

**Given:** Caplet on $L_3$ with payoff $\tau_3 (L_3(T_2) - K)^+$ at $T_3$.

**Find:** Pathwise formula for $\partial \text{Caplet} / \partial L_3(0)$.

**Solution:**

If $L_3(T_2) > K$ (in-the-money):
$$\frac{\partial}{\partial L_3(0)} \tau_3 (L_3(T_2) - K) = \tau_3 \frac{\partial L_3(T_2)}{\partial L_3(0)}$$

Under lognormal dynamics:
$$L_3(T_2) = L_3(0) \exp(\text{drift terms} + \sigma \int_0^{T_2} dW)$$

$$\frac{\partial L_3(T_2)}{\partial L_3(0)} = \frac{L_3(T_2)}{L_3(0)}$$

So the pathwise delta (when ITM) is:
$$\frac{\partial \text{Caplet}}{\partial L_3(0)} = \tau_3 \frac{L_3(T_2)}{L_3(0)} \times \mathbf{1}_{L_3(T_2) > K}$$

Discount and average over paths to get the Monte Carlo estimate.

### Example 10: Two-Curve Pricing

**Given:**
- OIS discount factors: $P^{\text{OIS}}(0, T_1) = 0.99$, $P^{\text{OIS}}(0, T_2) = 0.97$
- SOFR forward rate: $L^{\text{SOFR}} = 3.5\%$
- Caplet strike $K = 3\%$, $\tau = 1$
- Black vol $v = 25\%$

**Find:** Caplet price.

**Solution:**

Under two-curve framework, discount with OIS but use SOFR for rate:

$d_+ = \frac{\ln(0.035/0.03) + 0.5 \times 0.25^2}{0.25} = \frac{0.154 + 0.03125}{0.25} = 0.74$
$d_- = 0.74 - 0.25 = 0.49$

$\Phi(0.74) = 0.770$, $\Phi(0.49) = 0.688$

$\text{Caplet} = \tau \times P^{\text{OIS}}(0, T_2) \times [L^{\text{SOFR}} \Phi(d_+) - K \Phi(d_-)]$
$= 1 \times 0.97 \times [0.035 \times 0.770 - 0.03 \times 0.688]$
$= 0.97 \times [0.02695 - 0.02064] = 0.97 \times 0.00631 = 0.612\%$

### Example 11: Correlation Impact on Swaption Vol

**Given:** Same setup as Example 6, but now vary correlation.

**Find:** Swaption vol with $\rho = 0.7$ (lower correlation).

**Solution:**

Recompute cross terms with $\rho = 0.7$:
- $2 \times 0.32 \times 0.33 \times 0.7 \times 0.076 = 0.01124$
- $2 \times 0.32 \times 0.35 \times 0.7 \times 0.072 = 0.01129$
- $2 \times 0.33 \times 0.35 \times 0.7 \times 0.0684 = 0.01107$

Total: $0.00819 + 0.00787 + 0.00794 + 0.01124 + 0.01129 + 0.01107 = 0.05760$

$v_{2,5} = \sqrt{0.05760/2} = \sqrt{0.0288} = 17.0\%$

Lower correlation reduces swaption volatility (from 18.3% to 17.0%).

### Example 12: CMS Convexity Intuition

**Setup:** CMS 10y coupon paying in 3 months. Swap rate currently $S = 4\%$.

**Convexity effect direction:**

When rates rise:
- $S$ increases
- Annuity $A$ (the "natural" numeraire) decreases
- Under payment measure, these effects don't cancel

The convexity adjustment is typically positive, meaning:
$$\mathbb{E}^{T_p}[S(T)] > S(0)$$

Rough magnitude: For 10y CMS with 20% vol, 3-month forward, adjustment is typically 1-5 bp.

### Example 13: Bermudan Boundary Approximation

**Given:** 5-year Bermudan payer swaption, exercisable quarterly into remaining 5y swap.

**Approximate exercise rule:** Exercise when:
$$S_{a,b}(T_k) > K + \text{time value premium}$$

For deep ITM (> 100 bp ITM), time value premium is small; exercise is optimal.
For near ATM, time value premium can be 10-30 bp depending on volatility.

This approximation is used as a starting point for LSM regression.

---

## A4.13 Summary

### Key Takeaways

1. **Market models (LMM, SMM) are designed for Black-formula compatibility:** The genius of market models is that they ensure vanilla options price exactly via Black's formula under the appropriate measure.

2. **The drift adjustment is the complexity:** Under any common measure, most rates acquire drifts that depend on other rates, creating coupled dynamics that require simulation.

3. **SABR is the smile standard:** For capturing volatility smiles in caplets and swaptions, SABR has become the industry standard due to its tractable approximation and intuitive parameters.

4. **Near-zero/negative rates require convention-aware modeling:** Normal (Bachelier) quoting, shifted/displaced lognormal models, or normal/shifted SABR are common toolkits when strict log-normal assumptions break down near zero.

5. **Calibration is an art:** Matching market prices requires regularization, careful parameter choices, and validation of approximations like freezing.

6. **Greeks in simulation require care:** Pathwise and likelihood ratio methods overcome the noise problem of finite differences.

7. **SOFR requires two-curve thinking:** The multi-curve framework separates discounting (OIS) from rate projection (SOFR).

### Model Hierarchy Summary

| Level | Model | Use Case |
|-------|-------|----------|
| **Quoting** | Black / SABR | Vanilla caps, swaptions |
| **Exotic Pricing** | Calibrated LMM | Bermudans, CMS, callables |
| **Risk** | Same as pricing | Consistency is paramount |
| **XVA** | LMM + multi-curve | Long-dated portfolio exposure |

---

## A4.14 Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Forward measure $\mathbb{Q}^j$** | Measure with numeraire $P(t, T_j)$; $L_j$ is martingale | Enables Black formula for caplets |
| **Swap measure $\mathbb{Q}^{a,b}$** | Measure with numeraire $A_{a,b}(t)$; $S_{a,b}$ is martingale | Enables Black formula for swaptions |
| **LMM drift (spot measure)** | $\mu_j = \sum_{k=\beta(t)}^{j} \frac{\tau_k \rho_{jk} \sigma_j \sigma_k L_k}{1 + \tau_k L_k}$ | Coupled dynamics require simulation |
| **Rebonato approximation** | Freezes weights in swap-rate expansion | Connects LMM to swaption vols |
| **SABR** | Stochastic volatility model for smile | Industry standard for vol surface |
| **$\beta$ (SABR)** | CEV exponent controlling backbone | Often fixed at 0.5 or 1 |
| **$\rho$ (SABR)** | Forward-vol correlation | Controls skew direction |
| **$\nu$ (SABR)** | Vol-of-vol | Controls smile curvature |
| **Normal volatility** | Volatility in bp terms (Bachelier) | Used for negative rates |
| **Shift parameter** | Displacement for negative rates | Enables lognormal with negative rates |
| **Freezing approximation** | Assume weights constant at $t=0$ | Key simplification for calibration |
| **Pathwise Greeks** | Differentiate through simulation paths | Efficient for smooth payoffs |
| **Likelihood ratio** | Use density score for Greeks | Works for discontinuous payoffs |
| **Two-curve framework** | Separate discount (OIS) and projection (SOFR) | Required for modern rates pricing |

---

## A4.15 Notation Reference

| Symbol | Definition |
|--------|------------|
| $L_j(t)$ | Simply-compounded forward rate for period $[T_{j-1}, T_j]$ |
| $S_{a,b}(t)$ | Swap rate for swap from $T_a$ to $T_b$ |
| $P(t, T)$ | Zero-coupon bond price at $t$ maturing at $T$ |
| $A_{a,b}(t)$ | Annuity (PV of fixed leg) |
| $\tau_j$ | Year fraction $T_j - T_{j-1}$ |
| $\sigma_j(t)$ | Instantaneous volatility of $L_j$ |
| $\rho_{jk}$ | Correlation between $L_j$ and $L_k$ |
| $\mathbb{Q}^j$ | $T_j$-forward measure |
| $\mathbb{Q}^{a,b}$ | Swap measure |
| $\mathbb{Q}^{\mathrm{spot}}$ | Spot-LIBOR measure (single simulation measure on the tenor grid) |
| $\mathbb{Q}^n$ | Terminal measure (numeraire $P(t,T_n)$) |
| $\beta(t)$ | Index of first forward rate still stochastic at time $t$ |
| $\alpha, \beta, \rho, \nu$ | SABR parameters |
| $\sigma_N$ | Normal (Bachelier) volatility |
| $s$ | Shift parameter for shifted lognormal |
| $v_{a,b}$ | Swaption implied volatility |
| $w_j$ | Weight of forward $L_j$ in swap rate |

---

## A4.16 Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does LMM stand for? | LIBOR Market Model (also called BGM: Brace-Gatarek-Musiela) |
| 2 | Why is the forward measure $\mathbb{Q}^j$ special for $L_j$? | Under $\mathbb{Q}^j$, $L_j$ is a martingale, so Black's formula applies directly |
| 3 | What is the LMM drift under the spot measure? | $\mu_j = \sum_{k=\beta(t)}^{j} \frac{\tau_k \rho_{jk} \sigma_j \sigma_k L_k}{1 + \tau_k L_k}$, where $\beta(t)$ is the first still-stochastic forward index |
| 4 | Why can't multiple swap rates all be lognormal? | Different swap rates are ratios of different bond portfolios; making one lognormal makes others non-lognormal |
| 5 | What is the Rebonato approximation? | Freeze the weights $w_j$ at time 0 to derive swaption vol from LMM parameters |
| 6 | What does SABR stand for? | Stochastic Alpha Beta Rho |
| 7 | In SABR, what does $\beta$ control? | The CEV exponent; determines how ATM vol moves with forward level (backbone) |
| 8 | In SABR, what does $\rho$ control? | The correlation between forward and volatility; determines skew direction |
| 9 | In SABR, what does $\nu$ control? | Vol-of-vol; determines smile curvature (wing behavior) |
| 10 | Why is $\beta$ often fixed in calibration? | Parameters are poorly identified; fixing $\beta$ breaks degeneracy |
| 11 | What is Normal (Bachelier) volatility? | Volatility in absolute (bp) terms, assuming arithmetic Brownian motion |
| 12 | How do you handle negative rates in Black's model? | Use shifted lognormal: $F \to F + s$, $K \to K + s$ |
| 13 | What is a “typical” shift? | There is no universal shift; it is a desk/system convention (often chosen to handle negative rates and fit quotes). Confirm $s$ from the quoting rulebook or risk system settings. |
| 14 | How accurate is the freezing approximation? | Often useful as a first approximation/diagnostic, but accuracy depends on expiry/tenor/vol regime; validate on your product set. |
| 15 | What is pathwise differentiation? | Differentiating through simulation paths to compute Greeks |
| 16 | When does pathwise differentiation fail? | For discontinuous payoffs (barriers, digitals) |
| 17 | What is the likelihood ratio method? | Using the density score function to compute Greeks; works for any payoff |
| 18 | What is Tikhonov regularization? | Adding $\lambda\|\theta\|^2$ penalty to calibration objective for stability |
| 19 | What is eigenvalue zeroing? | Setting small eigenvalues of correlation matrix to zero for lower-rank structure |
| 20 | In two-curve pricing, what is discounting curve? | OIS curve (overnight rate based) |
| 21 | In two-curve pricing, what is projection curve? | SOFR curve (for floating rate payments) |
| 22 | Why is CMS convexity adjustment positive? | Rising rates increase swap rate but decrease annuity; under payment measure, this creates positive bias |
| 23 | What algorithm is standard for Bermudan swaptions? | Longstaff-Schwartz (LSM) with backward induction and regression |
| 24 | What are common LSM basis functions? | Low-order polynomials of swap rate, swap rate + intrinsic value |
| 25 | What is the "backbone" in SABR? | How ATM implied vol changes as the forward rate moves |
| 26 | What is "vanna skew" in SABR? | Skew component from $\rho$ (forward-vol correlation), proportional to $\rho\nu$ |
| 27 | What does terminal measure mean? | Numeraire is the final bond $P(t, T_n)$; forward rates $j < n$ have negative drift |
| 28 | Why use terminal measure for Bermudans? | It can be convenient because the numeraire is a fixed final bond and the drift has a clean summation form; the best choice depends on implementation details. |
| 29 | What is the ATM SABR vol formula (leading term)? | $\sigma^{\text{ATM}} \approx \alpha / F^{1-\beta}$ |
| 30 | What is predictor-corrector in LMM simulation? | Two-step method: predict with current drift, correct using averaged drift |
| 31 | How does correlation affect swaption vol in LMM? | Higher correlation increases swaption vol (more coordinated rate moves) |
| 32 | What is the Hagan approximation? | Closed-form SABR implied vol formula using singular perturbation |
| 33 | When does Hagan approximation break down? | Very OTM strikes (can give negative density), very long maturities |
| 34 | What is "shifted SABR"? | SABR applied to $(F + s)$ instead of $F$ to handle negative rates |
| 35 | What is Normal SABR? | SABR with $\beta = 0$; uses absolute rather than relative volatility |
| 36 | What is AAD? | Adjoint Algorithmic Differentiation; computes all Greeks in one backward pass |
| 37 | What is wrong-way risk in model selection? | When model assumptions break precisely when they matter most (e.g., correlation in stress) |
| 38 | What is the co-terminal swaption problem? | Different swap rates can't all be lognormal under the same measure |
| 39 | What causes CMS convexity? | Swap rate and its natural numeraire (annuity) are correlated |
| 40 | How do you convert Black vol to Normal vol (ATM)? | $\sigma_N \approx \sigma_{\text{Black}} \times F$ |
| 41 | What is "beta skew" in SABR? | Skew component from $\beta < 1$ (CEV effect), proportional to $(1-\beta)$ |
| 42 | What is "volga" in SABR context? | Smile curvature from vol-of-vol, proportional to $\nu^2$ |
| 43 | Why calibrate to swaptions not just caps? | Swaptions contain information about rate correlations |
| 44 | What is the standard correlation structure in LMM? | Exponential decay: $\rho_{jk} = e^{-\beta|j-k|}$ |
| 45 | How do RFR coupons differ from LIBOR-style forwards? | Many are compounded in arrears from daily overnight fixings; your model must map its state variables to that coupon definition. |

---

## A4.17 Problem Set

### Foundational Problems

**Problem 1.** Derive the relationship between the forward rate $L_j(t)$ and zero-coupon bond prices. Show that $L_j(t)$ is a martingale under the $T_j$-forward measure $\mathbb{Q}^j$.

**Problem 2.** A 6-month caplet has strike $K = 5\%$, forward rate $L = 5.2\%$, volatility $\sigma = 22\%$, discount factor $P(0, T_j) = 0.975$, and $\tau = 0.5$. Price the caplet using Black's formula.

**Problem 3.** Explain why the LMM drift depends on rates $L_k$ for $k \leq j$ under the spot measure, but on rates $L_k$ for $k \geq j$ under the terminal measure.

**Problem 4.** Given SABR parameters $\alpha = 0.3$, $\beta = 0.5$, $\rho = -0.25$, $\nu = 0.5$, with forward $F = 4\%$ and expiry $T = 2$ years, compute the ATM implied volatility (leading term only).

**Problem 5.** A swaption is quoted at 75 bp normal vol. If the forward swap rate is $S = 3\%$ and expiry is 1 year, what is the approximate Black implied volatility?

### Intermediate Problems

**Problem 6.** Using the Rebonato formula, show that if all forward rate volatilities are equal ($\sigma_j = \sigma$ for all $j$) and all correlations are 1, the swaption implied volatility equals the common forward volatility.

**Problem 7.** Consider a shifted lognormal model with shift $s = 1.5\%$. If the current forward rate is $F = -0.5\%$, strike is $K = 0\%$, shifted Black vol is 35%, and $T = 0.5$ years:
   a) Compute the caplet price
   b) What is the equivalent normal volatility?

**Problem 8.** Derive the pathwise formula for the delta of a European payer swaption in an LMM simulation context. What additional information do you need beyond the payoff?

**Problem 9.** A CMS 10y leg pays quarterly. The current 10y swap rate is 4.5% with annualized volatility 18%. Explain qualitatively why the convexity adjustment is positive and estimate its order of magnitude for a 6-month forward CMS payment.

**Problem 10.** In SABR calibration, you observe that increasing $\nu$ and decreasing $|\rho|$ simultaneously produces nearly identical smiles. Explain this degeneracy and how practitioners resolve it.

### Advanced Problems

**Problem 11.** Prove that under the freezing approximation, the swaption implied variance is a weighted average of forward rate covariances:
$$v_{a,b}^2 T_a = \sum_{j,k} w_j w_k \text{Cov}^{a,b}[\ln L_j(T_a), \ln L_k(T_a)]$$

**Problem 12.** The Hagan SABR formula can produce negative implied probability densities for very OTM strikes. Describe:
   a) How you would detect this in practice
   b) Two alternative approaches that avoid this problem

**Problem 13.** Implement (pseudocode) a predictor-corrector scheme for simulating 3 forward rates under the terminal measure. Include:
   a) The drift formula for each rate
   b) The Cholesky decomposition step
   c) The predictor and corrector steps

**Problem 14.** Consider a 2-exercise Bermudan swaption (exercisable at $T_1$ and $T_2$ into a swap ending at $T_3$). Using LSM:
   a) Describe the regression setup at $T_2$
   b) Describe how the exercise decision at $T_1$ uses the continuation value from $T_2$
   c) What basis functions would you use?

**Problem 15.** In a two-curve framework, the floating leg of a swap pays SOFR-linked coupons discounted at OIS rates. Derive the formula for the par swap rate $S_{a,b}^{\text{2-curve}}$ in terms of OIS discount factors and SOFR forward rates.

**Problem 16.** A rates desk uses SABR with $\beta = 0.5$ for swaptions. After a 100 bp rally in rates, the trader notices ATM vol has increased. Explain how this is consistent with the SABR backbone. What would happen if $\beta = 1$?

**Problem 17.** Compare and contrast pathwise differentiation and the likelihood ratio method for computing caplet delta in Monte Carlo. Include:
   a) When each method is applicable
   b) Variance characteristics
   c) Computational requirements

**Problem 18.** A model validation team notices that their LMM produces swaption prices 3% higher than market for 10y-into-20y swaptions, but matches well for shorter tenors. List three possible causes and how you would investigate each.

**Problem 19.** Derive the drift adjustment for a forward rate under the spot measure starting from the Girsanov theorem. Specifically, show how $d\langle L_j, M \rangle / L_j$ produces the coupled drift formula.

**Problem 20.** Design a regularization strategy for calibrating an LMM to both caps and swaptions. Address:
   a) How to weight the fitting errors
   b) How to constrain the correlation matrix
   c) How to ensure calibration stability across daily recalibrations

**Problem 21.** The "smile arbitrage" constraint says calendar spreads (swaptions with different expiries on the same underlying swap) must be non-negative. Explain how this constrains the term structure of SABR parameters and how you would enforce it in calibration.

**Problem 22.** Consider a callable range accrual that pays enhanced coupons when SOFR stays within a band. Outline the pricing approach using:
   a) LMM simulation for path generation
   b) LSM for the call feature
   c) Two-curve discounting
   d) What Greeks would be most important to hedge?

---

## A4.18 Solutions (Selected)

### Solution to Problem 2

Given: $K = 5\%$, $L = 5.2\%$, $\sigma = 22\%$, $P(0, T_j) = 0.975$, $\tau = 0.5$, $T_{j-1} = 0.5$ (expiry).

$d_+ = \frac{\ln(0.052/0.05) + 0.5 \times 0.22^2 \times 0.5}{0.22\sqrt{0.5}} = \frac{0.0392 + 0.0121}{0.1556} = 0.330$

$d_- = 0.330 - 0.1556 = 0.174$

$\Phi(0.330) = 0.629$, $\Phi(0.174) = 0.569$

Caplet $= 0.5 \times 0.975 \times [0.052 \times 0.629 - 0.05 \times 0.569]$
$= 0.4875 \times [0.0327 - 0.0285] = 0.4875 \times 0.0042 = 0.205\%$

### Solution to Problem 4

$\sigma^{\text{ATM}} \approx \frac{\alpha}{F^{1-\beta}} = \frac{0.3}{0.04^{0.5}} = \frac{0.3}{0.2} = 1.5 = 150\%$

(This is the leading term; the full formula would give a slightly different value including the time correction.)

### Solution to Problem 5

ATM approximation: $\sigma_N \approx \sigma_{\text{Black}} \times S$

$75 \text{ bp} = \sigma_{\text{Black}} \times 3\%$

$\sigma_{\text{Black}} = 0.0075 / 0.03 = 0.25 = 25\%$

### Solution to Problem 6

If $\sigma_j = \sigma$ and $\rho_{jk} = 1$ for all $j, k$:

$v_{a,b}^2 T_a = \sum_{j,k} w_j w_k \cdot 1 \cdot \sigma^2 T_a = \sigma^2 T_a \left(\sum_j w_j\right)^2 = \sigma^2 T_a$

(since $\sum_j w_j = 1$)

Therefore $v_{a,b} = \sigma$.

---

## References

- Damiano Brigo & Fabio Mercurio, *Interest Rate Models: Theory and Practice* (LMM/SMM dynamics, measure changes, Rebonato approximation, SABR overview)
- Leif B. G. Andersen & Vladimir V. Piterbarg, *Interest Rate Modeling* (option pricing under different backbones; displaced diffusion; simulation/Greeks techniques)
- Patrick S. Hagan, Deep Kumar, Andrew Lesniewski, & Diana Woodward (2002), “Managing Smile Risk” (SABR model and asymptotic implied-vol approximations)
- Paul Glasserman, *Monte Carlo Methods in Financial Engineering* (Monte Carlo estimation, variance reduction, and simulation-based Greeks)


*Appendix A4 of Fixed Income: Practice and Theory*
