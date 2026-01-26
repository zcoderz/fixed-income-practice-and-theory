# Appendix A1: No-Arbitrage Pricing, Numeraires, and Measure Changes

---

## Introduction

Every model you will ever build in rates—whether for a vanilla swap, an exotic callable, or a multi-currency structured note—must satisfy one unbreakable rule: *it cannot permit arbitrage*. This constraint is not merely philosophical; it is the mathematical backbone that determines how drift terms, discount factors, and probability measures must interlock. Violate it, and your model will produce internally inconsistent prices: a five-year bond priced differently depending on which replication strategy you use, or a cap and a floor that together imply a different forward rate than the one you bootstrapped.

**Why this matters on a desk:** A quant building a rates model doesn't get to choose the drift of forward rates independently of their volatility. A risk manager validating a pricing library must check that discounted prices are martingales under the right measure. A trader using Black's formula for swaptions relies implicitly on the swap rate being a martingale under the swap measure—without understanding why, they cannot diagnose when the formula breaks. The no-arbitrage framework is not abstract theory; it is the operating system of every serious rates desk.

This appendix covers the essential toolkit:

1. **Core definitions** (Sections 1–2): State-price deflators (SDFs), numeraires, equivalent martingale measures, and the fundamental pricing equation. We connect the SDF to Arrow-Debreu securities and risk premia.

2. **Change-of-numeraire machinery** (Section 3): The workhorse theorem that lets you move between measures—with explicit Radon-Nikodym derivations. This is how caplet formulas use forward measures and swaption formulas use swap measures.

3. **Bond numeraires and forward measures** (Section 4): The $T$-forward measure, why forward rates are martingales under it, and the swap measure.

4. **Bridge to HJM** (Section 5): Why specifying volatility "pins down" drift, with full intuition for the integral structure $\alpha(t,T) = \sigma(t,T) \int_t^T \sigma(t,s)\, ds$.

5. **Practical desk applications** (Section 6): A checklist for measure selection, common practitioner mistakes, and diagnostic tests.

6. **Derivations and worked examples** (Sections 7–8): Step-by-step mathematics and 13 numeric examples including caplet and swaption pricing.

**Prerequisites:** Familiarity with stochastic calculus (Ito's lemma, Brownian motion), basic bond mathematics (Chapter 2–3), and swap mechanics (Chapter 25). We reference Duffie's *Dynamic Asset Pricing Theory*, Brigo-Mercurio's *Interest Rate Models*, and Glasserman's *Monte Carlo Methods* throughout.

---

## Conventions & Notation

**Time:** Continuous time $t \in [0, T^*]$ (calendar time measured in years).

**Information:** $(\mathcal{F}_t)_{t \geq 0}$ is the market filtration (what is known up to time $t$).

**Measures (probability laws):**

| Measure | Description |
|---------|-------------|
| $\mathbb{P}$ | "Physical" / reference measure (real-world probabilities) |
| $\mathbb{Q}^N$ | Numeraire-associated pricing measure induced by a chosen strictly-positive numeraire $N(t)$. Under $\mathbb{Q}^N$, any traded price divided by $N$ is a martingale |
| $\mathbb{Q}^B$ | The measure associated with the money-market account $B(t)$ (the common "risk-neutral" measure) |
| $\mathbb{Q}^T$ | The $T$-forward measure associated with the zero-coupon bond numeraire $P(t,T)$ |
| $\mathbb{Q}^A$ | The swap (annuity) measure associated with the swap annuity $A(t) = \sum_i \tau_i P(t, T_i)$ |

**Numeraires:**

- $B(t)$: money-market account (bank account), strictly positive:
$$B(t) = B(0) \exp\!\left(\int_0^t r(s)\, ds\right),$$
where $r(t)$ is the short rate.

- $P(t,T)$: default-free zero-coupon bond price (pays 1 at $T$).

**Discounting:**

$$D(t,T) = \frac{B(t)}{B(T)} = \exp\!\left(-\int_t^T r(s)\, ds\right).$$

**Rates:**

- Instantaneous forward rate:
$$f(t,T) \equiv -\frac{\partial}{\partial T} \ln P(t,T).$$

- Bond–forward-rate relation:
$$P(t,T) = \exp\!\left(-\int_t^T f(t,u)\, du\right).$$

- Short rate:
$$r(t) = f(t,t).$$

**Brownian motions:** $W^{\mathbb{Q}}(t)$ denotes a Brownian motion under the specified measure $\mathbb{Q}$. Under a change of measure, drifts change but diffusion terms remain (Girsanov).

---

## 0. Setup

### 0.1 Conventions Used in This Appendix

- All prices are in a single currency and are nominal units (dimension: currency).
- A "traded asset" means an asset with an observable price process.
- Unless otherwise stated, we assume no intermediate cash flows (dividends) to keep notation light.
- Rates are in "per-year" units (dimension $1/\text{year}$).

### 0.2 Notation Glossary

| Symbol | Meaning | Dimension |
|--------|---------|-----------|
| $V(t)$ | Time-$t$ price of a claim | currency |
| $H_T$ | Payoff at time $T$ | currency |
| $N(t)$ | Numeraire | currency, strictly positive |
| $B(t)$ | Money-market account numeraire | currency |
| $P(t,T)$ | Zero-coupon bond price | currency per unit payoff |
| $D(t,T) = B(t)/B(T)$ | Discount factor | dimensionless |
| $\pi(t)$ | State-price deflator / SDF | $1/\text{currency}$ |
| $\mathbb{Q}^N$ | Measure associated with numeraire $N$ | — |
| $f(t,T)$ | Instantaneous forward rate | $1/\text{year}$ |
| $r(t)$ | Short rate | $1/\text{year}$ |
| $\sigma(t,T)$ | HJM forward-rate volatility | discussed in §5 |

---

## 1. Core Concepts

### 1.1 Arbitrage and No-Arbitrage

**Formal definition:** In a finite-state setting, Duffie defines an *arbitrage* as a portfolio $\theta$ with $q \cdot \theta \leq 0$ and $D^\top \theta > 0$, or $q \cdot \theta < 0$ and $D^\top \theta \geq 0$—where $q$ is the vector of security prices and $D$ is the payoff matrix. In Duffie's words, "an arbitrage is a portfolio offering 'something for nothing.'" In continuous time, an arbitrage is a self-financing trading strategy that generates strictly positive wealth from zero initial investment with no possibility of loss.

*No-arbitrage* is the absence of such strategies. In Duffie's development, no-arbitrage is equivalent to the existence of a *state-price vector* (or deflator).

**The Fundamental Theorem of Asset Pricing (finite-state version):**

$$\boxed{\text{No arbitrage} \iff \text{Existence of a state-price vector } \psi \gg 0 \text{ with } q = D\psi}$$

Duffie proves this using the Separating Hyperplane Theorem: if no arbitrage exists, the cone of non-negative payoffs and the linear subspace of portfolio payoffs intersect only at zero, which implies a separating hyperplane exists. This hyperplane corresponds to the state-price vector. Geometrically, the state prices define a "pricing plane" orthogonal to all arbitrage-free portfolio directions.

**Intuition:** Arbitrage is the proverbial "free lunch"—a way to lock in profit "by algebra" through clever combinations of traded instruments. The fundamental theorem says that if no such free lunch exists, there must be a consistent way to assign positive values to each state of the world (state prices). These state prices enforce internal consistency: any two replicating strategies for the same payoff must have the same cost.

**Practice:** Detecting potential arbitrage is a consistency check. For example:
- If your curve construction implies that a 5-year bond can be replicated two ways with different prices, you have broken no-arbitrage.
- If your model's forward-rate drift doesn't satisfy the HJM restriction, discounted bond prices will have predictable gains—pure arbitrage in the limit.
- If a butterfly spread on the yield curve can be constructed at negative cost with non-negative payoff, your curve is arbitrage-ridden.

---

### 1.2 State-Price Deflator (SDF) / Pricing Kernel

**Formal definition:** A *state-price deflator* $\pi(t)$ is a strictly positive adapted process that prices payoffs via:

$$\boxed{V(t) = \frac{1}{\pi(t)}\, \mathbb{E}_t^{\mathbb{P}}\!\left[\pi(T)\, H_T\right]}$$

This is Duffie's fundamental pricing equation. He explicitly notes naming variations: "state-price deflator," "state-price density," and links to "pricing kernel" and "marginal rate of substitution" from consumption-based asset pricing.

**Connection to Arrow-Debreu securities:** Cochrane's *Asset Pricing* emphasizes that the SDF can be understood through contingent claims. An Arrow-Debreu security pays \$1 in exactly one state $\omega$ and zero otherwise. If $\psi(\omega)$ is the price today of this security, then for any payoff $H$:

$$V(0) = \sum_{\omega} \psi(\omega) H(\omega) = \mathbb{E}^{\mathbb{P}}[m \cdot H]$$

where $m(\omega) = \psi(\omega)/\pi(\omega)$ is the pricing kernel. Cochrane writes: "The state-price density $m$ gives the value of a dollar in each state of nature, adjusted for probability." In continuous time, this becomes $\pi(t)$.

**Connection to marginal utility and risk premia:** In equilibrium models, Duffie shows that the SDF equals the representative agent's intertemporal marginal rate of substitution:

$$\pi(t) = e^{-\rho t} \frac{u'(c_t)}{u'(c_0)}$$

where $\rho$ is the subjective discount rate and $u'(c)$ is marginal utility of consumption. This explains why the SDF discounts more heavily in "bad states" (low consumption, high marginal utility) and less in "good states."

**Risk premia from SDF covariance:** From the SDF pricing equation, one can derive that the expected excess return on any asset depends on its covariance with the SDF:

$$\mathbb{E}[R_i - R_f] = -\frac{\text{Cov}(R_i, m)}{\mathbb{E}[m]}$$

Assets that pay off well when $m$ is low (good times) must offer higher expected returns—the essence of risk premia.

**CAPM as a special case:** When the SDF is linear in the market return ($m = a - b R_M$), the SDF framework reduces to CAPM. The beta of an asset measures its covariance with $m$, rescaled.

**Practice:** In rates/credit, the SDF viewpoint explains why "risk premia" exist under $\mathbb{P}$ (real-world measure) and why pricing is naturally expressed under a measure $\mathbb{Q}$ where discounted prices are martingales. The SDF reweights states—under $\mathbb{Q}$, we've already incorporated the risk adjustment into probabilities.

---

### 1.3 Numeraire

**Formal definition:** Following Geman et al. (1995), Brigo-Mercurio define: "A numeraire is any positive non-dividend-paying asset." More generally, a numeraire $Z$ can be identified with a self-financing trading strategy $\phi$ such that $Z_t = V_t(\phi)$ for all $t$—the numeraire's value equals the portfolio value.

**The reference-asset interpretation:** Brigo-Mercurio emphasize: "Intuitively, a numeraire is a reference asset that is chosen so as to normalize all other asset prices with respect to it." Choosing numeraire $Z$ means working with *relative prices* $S^k/Z$ rather than absolute prices $S^k$. This is not merely notational—it induces a genuine change in probability measure.

**Self-financing invariance:** A key technical result (Geman et al., 1995) is that self-financing strategies remain self-financing after a numeraire change. The self-financing condition $dV_t(\phi) = \sum_k \phi_t^k\, dS_t^k$ transforms consistently under the change of unit.

**Common numeraire choices:**

| Numeraire | Notation | Associated Measure | Key Property |
|-----------|----------|-------------------|--------------|
| Money-market account | $B(t)$ | $\mathbb{Q}^B$ (risk-neutral) | Discounted prices are martingales |
| Zero-coupon bond | $P(t,T)$ | $\mathbb{Q}^T$ (T-forward) | Forward prices for delivery at $T$ are martingales |
| Swap annuity | $A(t) = \sum_i \tau_i P(t,T_i)$ | $\mathbb{Q}^A$ (swap) | Forward swap rates are martingales |

**Why numeraire choice matters:** Brigo-Mercurio's "Fact One" states: "The price of any asset divided by a reference positive non-dividend-paying asset (called numeraire) is a martingale (no drift) under the measure associated with that numeraire." This is the core insight: choosing your numeraire strategically can make the key underlying rate or price a martingale, enabling tractable pricing formulas.

**Practice:** Pick the numeraire that makes your payoff's underlying "simple" (i.e., a martingale). For a caplet paying $(L_T - K)^+$ at $T$, using $P(t,T)$ as numeraire makes $L_T$ a martingale—hence Black's formula. For a swaption, using the annuity makes the swap rate a martingale—hence the swaption Black formula.

---

### 1.4 Equivalent Martingale Measure (Risk-Neutral Measure)

**Formal definition:** An *equivalent martingale measure* $\mathbb{Q}$ is a probability measure equivalent to $\mathbb{P}$ under which discounted asset prices are martingales.

Brigo-Mercurio state the key theorem: under the measure $\mathbb{Q}^N$ associated with numeraire $N$, any traded price divided by $N$ is a martingale:

$$\frac{Z(t)}{N(t)} \text{ is a } \mathbb{Q}^N\text{-martingale.}$$

**"Discounted prices are martingales" means:** Under $\mathbb{Q}^B$ (bank-account numeraire), for a traded asset price $S(t)$:

$$\frac{S(t)}{B(t)} \text{ is a martingale under } \mathbb{Q}^B.$$

A key implication: if $S(t)$ has diffusion dynamics under $\mathbb{Q}^B$, its drift must be $r(t)$ (the short rate).

**Practice:** "Risk-neutral probability" is not a belief about the real world; it's a computational device. The measure $\mathbb{Q}$ depends on which numeraire you choose.

---

### 1.5 Radon-Nikodym Derivative and Change of Measure

**Formal definition:** For two equivalent measures $\mathbb{Q}^* \sim \mathbb{Q}$, the Radon-Nikodym derivative:

$$\left.\frac{d\mathbb{Q}^*}{d\mathbb{Q}}\right|_{\mathcal{F}_t} = \rho_t$$

is a positive $\mathcal{F}_t$-measurable density process linking expectations:

$$\mathbb{E}^{\mathbb{Q}^*}[X] = \mathbb{E}^{\mathbb{Q}}[\rho_T \cdot X]$$

**Intuition:** Measure change = reweighting paths/scenarios. In diffusion settings, Girsanov's theorem says drifts change while volatility coefficients remain the same in Ito form.

**Practice:** Forward measures and swap measures are measure changes induced by selecting bond or annuity numeraires. The Radon-Nikodym derivative tells you exactly how to convert expectations.

---

### 1.6 Forward Measure ($T$-Forward Measure)

**Formal definition:** The $T$-forward measure $\mathbb{Q}^T$ is the measure induced by the numeraire $P(t,T)$. The central pricing identity is:

$$\boxed{V(t) = P(t,T)\, \mathbb{E}_t^T\!(H_T)}$$

for payoffs paid at $T$.

**Intuition:** If everything happens at time $T$, using $P(t,T)$ as the unit often removes discounting complexities—the discount factor is pulled outside the expectation.

**Practice:** Caplets and FRA-style claims naturally align with forward measures for their payment dates. This is the conceptual basis of Black's formula in market models.

---

### 1.7 Swap Measure (Annuity Measure)

**Formal definition:** The swap annuity:

$$A(t) = \sum_{i=1}^{n} \tau_i P(t, T_i)$$

where $\tau_i$ are accrual fractions and $T_i$ are payment dates, can serve as a numeraire. The induced measure $\mathbb{Q}^A$ is the *swap measure*.

Under $\mathbb{Q}^A$, the forward swap rate $S(t)$ is a martingale by construction. Swaption payoffs become:

$$V(t) = A(t)\, \mathbb{E}_t^A\!\left[(S(T) - K)^+\right]$$

**Practice:** Standard swaption desk modeling uses this measure for tractability—if $S(t)$ is a martingale, you can model it as lognormal (or shifted lognormal) and get closed-form Black formulas.

---

### 1.8 Zero-Coupon Bond Prices and Money-Market Account

**Formal definitions:**

- $P(t,T)$: time-$t$ price of a bond paying 1 at $T$.
- $B(t)$: bank account that accrues at the short rate $r(t)$, i.e., $B(t) = B(0)\exp(\int_0^t r(s)\, ds)$.

**Why these are the building blocks:** Every fixed-income instrument can be decomposed into portfolios of zero-coupon bonds. A coupon bond is a sum of zeros. A swap is a difference of bonds divided by an annuity. Understanding bond prices means understanding the entire rates market.

**The two fundamental numeraires:** $B(t)$ and $P(t,T)$ represent the two most common numeraire choices:

- Under $B(t)$ (money-market numeraire): $P(t,T)/B(t)$ is a martingale under $\mathbb{Q}^B$. This is the "risk-neutral" world where all assets earn the short rate on average.
- Under $P(t,T)$ (bond numeraire): $S(t)/P(t,T)$ is a martingale under $\mathbb{Q}^T$ for any traded asset $S$. This is the "forward" world where forward prices are martingales.

**Connection to discounting:** The stochastic discount factor $D(t,T) = B(t)/B(T) = \exp(-\int_t^T r(s)\, ds)$ satisfies $P(t,T) = \mathbb{E}_t^{\mathbb{Q}^B}[D(t,T)]$—the bond price is the expected discount factor under the risk-neutral measure.

---

### 1.9 Instantaneous Forward Rate and Short Rate

**Formal definitions:**

$$f(t,T) = -\partial_T \ln P(t,T), \qquad P(t,T) = \exp\left(-\int_t^T f(t,u)\, du\right), \qquad r(t) = f(t,t).$$

**Intuition:** The forward rate $f(t,T)$ represents the "marginal cost of extending maturity by $dT$" at time $t$. If you think of $-\ln P(t,T)$ as the total "log-cost" of borrowing from $t$ to $T$, then $f(t,T)$ is the instantaneous rate at which this cost accumulates as you extend $T$.

**The short rate as a limit:** The short rate $r(t) = f(t,t)$ is the instantaneous borrowing rate—the forward rate for immediate, infinitesimally short lending. In practice, this corresponds to overnight rates (approximated by Fed Funds or SOFR in USD markets).

**Why HJM uses forward rates:** The forward-rate curve $T \mapsto f(t,T)$ completely determines the bond price curve $T \mapsto P(t,T)$ through integration. HJM's insight was to model the entire forward curve's evolution, rather than just the short rate. This automatically keeps bond prices consistent across maturities, but requires the drift restriction (Section 5) for no-arbitrage.

**Practice:** When a trader says "the curve steepened," they mean forward rates at longer maturities rose relative to shorter maturities—the slope of $T \mapsto f(t,T)$ increased.

---

## 2. No-Arbitrage Pricing: From SDF to Risk-Neutral Valuation

### 2.1 The SDF Pricing Equation

For a payoff $H_T$ at time $T$, Duffie's state-price deflator pricing relation gives:

$$V(t) = \frac{1}{\pi(t)}\, \mathbb{E}_t^{\mathbb{P}}\!\left[\pi(T)\, H_T\right]$$

**Unit check:**
- $H_T$ has units currency.
- $\pi(T)$ has units $1/\text{currency}$ so $\pi(T) H_T$ is dimensionless.
- $\frac{1}{\pi(t)}$ returns currency. ✓

**Arrow-Debreu interpretation (Cochrane):** Think of $\pi(T)/\pi(t)$ as the "price per unit probability per dollar payoff" in each state. States where $\pi$ is high (bad times, high marginal utility) are expensive—payoffs there are valued more.

---

### 2.2 Choosing the Money-Market Account as Numeraire

Brigo-Mercurio's numeraire theorem: if $N$ is a numeraire and $\mathbb{Q}^N$ the associated measure, then:

$$V(t) = N(t)\, \mathbb{E}_t^{\mathbb{Q}^N}\!\left(\frac{H_T}{N(T)}\right)$$

Setting $N = B$ (money-market account):

$$V(t) = B(t)\, \mathbb{E}_t^{\mathbb{Q}^B}\!\left(\frac{H_T}{B(T)}\right) = \mathbb{E}_t^{\mathbb{Q}^B}\!\left[D(t,T)\, H_T\right]$$

where $D(t,T) = B(t)/B(T)$ is the stochastic discount factor.

This is the classic risk-neutral valuation formula: prices are discounted expectations under $\mathbb{Q}^B$.

---

### 2.3 What Changes vs What Stays Fixed Under Measure Change

**Payoffs do not change:** $H_T$ is the same contractual random variable under any measure.

**Drifts/probabilities change:** The law of underlying processes changes. In diffusion settings, Girsanov implies drift changes while volatility stays the same in Ito form.

**Prices are invariant (if you numeraire-adjust correctly):**

$$V(t) = N(t)\, \mathbb{E}_t^{\mathbb{Q}^N}\!\left(\frac{H_T}{N(T)}\right) = U(t)\, \mathbb{E}_t^{\mathbb{Q}^U}\!\left(\frac{H_T}{U(T)}\right)$$

for any two numeraires $N, U$, provided the measures are linked by the correct Radon-Nikodym derivative.

---

## 3. Change of Numeraire Theorem (The Workhorse)

### 3.1 Statement

Let $N(t)$ be a numeraire and $\mathbb{Q}^N$ its associated measure. Then for any traded asset with price $Z(t)$:

$$\frac{Z(t)}{N(t)} \text{ is a } \mathbb{Q}^N\text{-martingale.}$$

For a claim paying $Z(T)$ at time $T$:

$$\boxed{Z(t) = N(t)\, \mathbb{E}_t^{\mathbb{Q}^N}\!\left(\frac{Z(T)}{N(T)}\right)} \tag{CON}$$

> **Brigo-Mercurio's "Three Facts" for Numeraire Changes**
>
> Brigo-Mercurio distill the change-of-numeraire toolkit into three operational facts:
>
> **Fact One (Martingale property):** The price of any asset divided by a numeraire is a martingale under the measure associated with that numeraire.
>
> **Fact Two (Price invariance):** The time-$t$ risk-neutral price is invariant by change of numeraire: if $B$ and $S$ are any two numeraires,
> $$\mathbb{E}_t^{\mathbb{Q}^B}\!\left[\frac{B(t)}{B(T)} \cdot \text{Payoff}(T)\right] = \mathbb{E}_t^{\mathbb{Q}^S}\!\left[\frac{S(t)}{S(T)} \cdot \text{Payoff}(T)\right]$$
>
> **Fact Three (Drift adjustment):** When moving from numeraire $N_1$ to $N_2$, the drift of any asset changes by:
> $$\text{drift}^{N_2} = \text{drift}^{N_1} - \sigma_{\text{asset}}^\top \left(\frac{\sigma_{N_1}}{N_1} - \frac{\sigma_{N_2}}{N_2}\right)$$
> where $\sigma$ denotes volatility vectors. The diffusion coefficient is unchanged (diffusion invariance).
>
> These three facts, together with Girsanov's theorem, form the complete operational toolkit for switching between measures in interest rate modeling.

---

### 3.2 Step-by-Step Radon-Nikodym Derivation

Brigo-Mercurio provide the explicit derivation. Here we spell out each step.

**Step 1: Start with two numeraires.** Let $N(t)$ and $U(t)$ be two valid numeraires with associated measures $\mathbb{Q}^N$ and $\mathbb{Q}^U$.

**Step 2: Write the pricing identity under each.** For any traded asset $Z$:

$$Z(t) = N(t)\, \mathbb{E}_t^{\mathbb{Q}^N}\!\left(\frac{Z(T)}{N(T)}\right) = U(t)\, \mathbb{E}_t^{\mathbb{Q}^U}\!\left(\frac{Z(T)}{U(T)}\right)$$

**Step 3: Define what we seek.** We want the density $\rho_T$ such that for any $\mathcal{F}_T$-measurable $X$:

$$\mathbb{E}^{\mathbb{Q}^U}[X] = \mathbb{E}^{\mathbb{Q}^N}[\rho_T \cdot X]$$

**Step 4: Apply the pricing identity to a specific claim.** Consider the claim that pays $Z(T) = U(T)$ at time $T$. Under numeraire $N$:

$$U(t) = N(t)\, \mathbb{E}_t^{\mathbb{Q}^N}\!\left(\frac{U(T)}{N(T)}\right)$$

At $t=0$:

$$U(0) = N(0)\, \mathbb{E}^{\mathbb{Q}^N}\!\left(\frac{U(T)}{N(T)}\right)$$

So:

$$\mathbb{E}^{\mathbb{Q}^N}\!\left(\frac{U(T)}{N(T)}\right) = \frac{U(0)}{N(0)}$$

**Step 5: Identify the Radon-Nikodym derivative.** For the change-of-measure identity to hold for all payoffs, we need:

$$\boxed{\left.\frac{d\mathbb{Q}^U}{d\mathbb{Q}^N}\right|_{\mathcal{F}_T} = \frac{U(T)\, N(0)}{U(0)\, N(T)}} \tag{RN-CON}$$

**Step 6: Verify consistency.** Check that $\mathbb{E}^{\mathbb{Q}^N}[\frac{d\mathbb{Q}^U}{d\mathbb{Q}^N}] = 1$:

$$\mathbb{E}^{\mathbb{Q}^N}\!\left[\frac{U(T) N(0)}{U(0) N(T)}\right] = \frac{N(0)}{U(0)} \cdot \mathbb{E}^{\mathbb{Q}^N}\!\left[\frac{U(T)}{N(T)}\right] = \frac{N(0)}{U(0)} \cdot \frac{U(0)}{N(0)} = 1 \quad \checkmark$$

---

### 3.3 The General Martingale Rule

Under $\mathbb{Q}^N$, any traded $Z(t)$ satisfies:

$$\boxed{\mathbb{E}^{\mathbb{Q}^N}\!\left[\frac{Z(t)}{N(t)} \,\Big|\, \mathcal{F}_u\right] = \frac{Z(u)}{N(u)} \quad (u \leq t)} \tag{Mart}$$

---

### 3.4 Girsanov's Theorem: What Changes

When switching from $\mathbb{Q}^N$ to $\mathbb{Q}^U$, Girsanov tells us:

- **Volatility unchanged:** The diffusion coefficient $\sigma$ in $dX = \mu\, dt + \sigma\, dW$ stays the same.
- **Drift changes:** The new drift under $\mathbb{Q}^U$ differs from the old drift under $\mathbb{Q}^N$ by a term involving the volatility of the Radon-Nikodym derivative.

Andersen-Piterbarg call this the **diffusion invariance principle**: "While a change of probability measure affects the drift $\mu$ of an Ito process, it does not change the diffusion coefficient $\sigma$." This principle is fundamental to interest rate modeling—it means that when you switch between forward measures or from risk-neutral to physical measure, your volatility calibration remains valid. Only the drift adjustment changes.

Specifically, if $W^N$ is a Brownian motion under $\mathbb{Q}^N$, then under $\mathbb{Q}^U$:

$$\boxed{dW^U = dW^N - \gamma(t)\, dt}$$

where $\gamma(t)$ is the "market price of risk" associated with the numeraire change. For an Ito process $dX = \mu^N dt + \sigma\, dW^N$ under $\mathbb{Q}^N$, the same process under $\mathbb{Q}^U$ becomes:

$$dX = (\mu^N + \sigma \gamma)\, dt + \sigma\, dW^U$$

The drift changes by $\sigma \gamma$; the diffusion coefficient $\sigma$ is invariant.

---

### 3.5 Sanity Checks

**Price invariance:** Pricing $Z(t)$ under $N$ or $U$ must match; if not, you have mixed measures or used a wrong density.

**Deterministic rates:** If $B(t)$ and $P(t,T)$ are deterministic, the RN derivative becomes deterministic, and expectations under different measures agree (only discounting differs).

**Unit check:** $\frac{d\mathbb{Q}^U}{d\mathbb{Q}^N}$ is dimensionless: $\frac{U(T)}{N(T)}$ and $\frac{N(0)}{U(0)}$ are both dimensionless. ✓

---

## 4. Bond Numeraires and Forward Measures

### 4.1 $T$-Forward Measure Induced by $P(t,T)$

Let $N(t) = P(t,T)$. Then $\mathbb{Q}^T$ is the $T$-forward measure and:

$$\boxed{V(t) = P(t,T)\, \mathbb{E}_t^T\!(H_T)} \tag{FwdPricing}$$

**Practitioner interpretation:** If the payoff date matches the bond numeraire maturity, you "factor out" discounting as $P(t,T)$. You then only need an expectation under $\mathbb{Q}^T$—no stochastic discounting inside the expectation.

---

### 4.2 Forward Prices as Martingales

For a traded asset $S(t)$, the $T$-maturity forward price is:

$$F(t;T) = \frac{S(t)}{P(t,T)}$$

By the numeraire-martingale rule, $\frac{S(t)}{P(t,T)}$ is a $\mathbb{Q}^T$-martingale, so:

$$F(t;T) = \mathbb{E}_t^T\!(S(T))$$

The forward price equals the expected spot under the forward measure.

---

### 4.3 Simple-Compounded Forward Rates are Martingales

Define the simple-compounded forward rate for the period $[S,T]$ with accrual $\tau(S,T)$:

$$\boxed{F(t;S,T) = \frac{1}{\tau(S,T)}\left(\frac{P(t,S)}{P(t,T)} - 1\right)} \tag{SimpleFwd}$$

Brigo-Mercurio state that $F(t;S,T)$ is a martingale under the $T$-forward measure $\mathbb{Q}^T$.

**Practice:** This is the backbone of caplet pricing. If the forward rate is a martingale under a forward measure, modeling it as lognormal yields closed-form Black formulas.

---

### 4.4 Swap Measure (Annuity Numeraire)

For swaption-style claims, choosing the swap annuity $A(t) = \sum_i \tau_i P(t, T_i)$ as numeraire leads to the swap measure $\mathbb{Q}^A$. Under this measure, the forward swap rate:

$$S(t) = \frac{P(t,T_0) - P(t,T_n)}{A(t)}$$

is a martingale by construction.

**Radon-Nikodym from bank-account measure:**

$$\frac{d\mathbb{Q}^A}{d\mathbb{Q}^B}\bigg|_{\mathcal{F}_T} = \frac{A(T)}{A(0)\, B(T)}$$

---

## 5. Bridge to HJM: Why Drift Restrictions Appear

### 5.1 The HJM Modeling Object

HJM models specify the dynamics of the entire instantaneous forward curve $T \mapsto f(t,T)$. The fundamental link is:

$$P(t,T) = \exp\!\left(-\int_t^T f(t,u)\, du\right), \qquad f(t,T) = -\partial_T \ln P(t,T)$$

A standard HJM diffusion under the risk-neutral measure $\mathbb{Q}^B$ is:

$$\boxed{df(t,T) = \alpha(t,T)\, dt + \sum_{i=1}^{d} \sigma_i(t,T)\, dW_i(t)} \tag{HJM-f}$$

where $W$ is a $d$-dimensional Brownian motion under $\mathbb{Q}^B$.

---

### 5.2 Why Volatility is "Free" but Drift is "Pinned Down"

Brigo-Mercurio emphasize: in HJM, you can choose the volatility structure $\sigma(t,T)$ (within regularity constraints), but then no-arbitrage determines the drift $\alpha(t,T)$.

**The mechanism:**

1. Bond prices $P(t,T)$ are tradable.
2. Under the bank-account numeraire $B(t)$, discounted bond prices $P(t,T)/B(t)$ must be martingales.
3. The assumed forward-rate diffusion implies a bond-price diffusion (via $P = e^{-\int f}$).
4. Enforcing the martingale condition forces a relationship between $\alpha$ and $\sigma$.

---

### 5.3 The HJM Drift Restriction

Under the risk-neutral measure $\mathbb{Q}^B$, no-arbitrage requires:

$$\boxed{\alpha(t,T) = \sum_{i=1}^{d} \sigma_i(t,T) \int_t^T \sigma_i(t,s)\, ds} \tag{HJM-drift}$$

In vector notation: $\alpha(t,T) = \sigma(t,T)^\top \int_t^T \sigma(t,s)\, ds$.

---

### 5.4 Why the Integral Appears: Deep Intuition

The integral $\int_t^T \sigma(t,s)\, ds$ appears because of how forward rates aggregate into bond prices. Here is the intuition:

**Step 1: Bond price volatility comes from integrated forward-rate volatility.**

Since $P(t,T) = \exp(-\int_t^T f(t,u)\, du)$, a shock to forward rates at all maturities between $t$ and $T$ affects the bond price. The bond price volatility is:

$$\sigma^P(t,T) = -\int_t^T \sigma(t,s)\, ds$$

(the negative arises from $P = e^{-A}$).

**Step 2: Bonds must have drift $r(t)$ under $\mathbb{Q}^B$.**

Discounted bonds are martingales, so bond prices must drift at rate $r(t)$:

$$\frac{dP}{P} = r(t)\, dt + (\text{diffusion})$$

**Step 3: Ito's lemma on $P = e^{-A}$ generates an extra term.**

When you apply Ito to $e^{-A}$, you get a $\frac{1}{2}(\text{diffusion})^2$ drift correction. This is:

$$\frac{1}{2}\left(\int_t^T \sigma(t,s)\, ds\right)^2$$

**Step 4: The forward-rate drift must cancel this correction.**

For the bond drift to equal $r(t)$, the forward-rate drift must compensate exactly:

$$\int_t^T \alpha(t,u)\, du = \frac{1}{2}\left(\int_t^T \sigma(t,u)\, du\right)^2$$

Differentiating with respect to $T$ gives the HJM drift restriction.

**Takeaway:** The integral structure is not magic—it's the mathematical consequence of bonds being exponentials of integrated forward rates, combined with Ito's correction term.

---

### 5.5 Drift Under Forward Measures

Glasserman gives the drift of $f(t,T)$ under the forward measure $\mathbb{Q}^{T_F}$ associated with maturity $T_F$:

$$\boxed{df(t,T) = -\,\sigma(t,T)^\top\!\left(\int_T^{T_F} \sigma(t,u)\, du\right) dt + \sigma(t,T)\, dW^{T_F}(t)} \tag{HJM-fwd}$$

The drift changes from $\int_t^T$ under $\mathbb{Q}^B$ to $-\int_T^{T_F}$ under $\mathbb{Q}^{T_F}$.

**Special case:** Under $\mathbb{Q}^T$ (i.e., $T_F = T$), the drift of $f(t,T)$ is zero—the forward rate for the numeraire maturity is a martingale (consistent with $F(t;T-\epsilon,T)$ being a martingale under $\mathbb{Q}^T$).

---

### 5.6 Sanity Checks

**If $\sigma \equiv 0$:** (HJM-drift) gives $\alpha \equiv 0$, so $df = 0$ (deterministic evolution). Bond prices become deterministic. ✓

**Units check:**
- $f(t,T)$ has units $1/\text{year}$.
- In $df = \alpha\, dt + \sigma\, dW$: $dt$ has units year, so $\alpha$ has units $1/\text{year}^2$.
- $dW$ has units $\sqrt{\text{year}}$, so $\sigma$ has units $1/\text{year}^{3/2}$.
- In (HJM-drift), $\int_t^T \sigma\, ds$ has units $1/\text{year}^{1/2}$; multiplying by $\sigma$ gives $1/\text{year}^2$. ✓

---

## 6. Practical Desk Applications

### 6.1 Measure Selection Checklist

| Payoff Type | Numeraire | Measure | Why This Works |
|-------------|-----------|---------|----------------|
| Generic discounted pricing | $B(t)$ | $\mathbb{Q}^B$ | Universal; discounting is $B$-based |
| Single cashflow at $T$ | $P(t,T)$ | $\mathbb{Q}^T$ | $V = P(t,T) \mathbb{E}^T[H_T]$—no stochastic discounting |
| Caplet on $L(T_1,T_2)$ paying at $T_2$ | $P(t,T_2)$ | $\mathbb{Q}^{T_2}$ | Forward rate $L(t;T_1,T_2)$ is martingale |
| Swaption on swap rate $S$ | $A(t)$ | $\mathbb{Q}^A$ | Swap rate $S(t)$ is martingale |
| Cross-currency (DOM-denominated) | $B^{\text{DOM}}(t)$ | $\mathbb{Q}^{\text{DOM}}$ | FX-adjusted foreign rates appear |

### 6.2 Common Practitioner Mistakes

**Mistake 1: Confusing numeraire change with currency conversion.**

Numeraire change is within one currency; it changes the pricing measure. Currency conversion involves FX rates and different economies.

**Mistake 2: Forgetting drift depends on measure.**

Girsanov: drift changes under measure change; volatility stays (in Ito form). The forward rate has different drift under $\mathbb{Q}^B$ vs $\mathbb{Q}^T$.

**Mistake 3: Mixing measures in a calculation.**

Example error: using the $\mathbb{Q}^B$ drift for a forward rate while taking a $\mathbb{Q}^T$ expectation. Always verify that the dynamics match the measure under which you're computing expectations.

**Mistake 4: Treating $\mathbb{Q}$ as real-world probabilities.**

$\mathbb{Q}$ is a pricing construct linked to a numeraire; it is not a forecast. Expected returns under $\mathbb{Q}$ are $r$, not the physical expected return.

**Mistake 5: Independently fitting drift and volatility in HJM.**

If you "calibrate" $\alpha$ and $\sigma$ independently to different data, you break the no-arbitrage restriction. Volatility determines drift.

**Mistake 6: Ignoring the Jacobian in multi-curve setups.**

When projection and discounting curves differ, the sensitivity of a rate to curve bumps involves a Jacobian. Using single-curve intuition can give wrong hedge ratios.

### 6.3 Diagnostic Tests for Implementation

**Discounted-price martingale test:** Under $\mathbb{Q}^B$, simulate paths and verify $S(t)/B(t)$ has zero drift (sample mean should not trend).

**Cross-numeraire consistency:** Price the same payoff under two numeraires; results must match (see Example 4).

**HJM drift test:** With your volatility function, compute $\alpha$ from the restriction. Simulate forward rates and verify discounted bonds are martingales.

**Put-call parity:** For European options, verify $C - P = S - K \cdot P(0,T)$ holds numerically.

---

## 7. Derivations

### 7.1 SDF Pricing → Risk-Neutral Pricing

**Start from Duffie's SDF pricing:**

$$V(t) = \frac{1}{\pi(t)}\, \mathbb{E}_t^{\mathbb{P}}\!\left[\pi(T)\, H_T\right]$$

**Under numeraire $N$, Brigo-Mercurio give:**

$$V(t) = N(t)\, \mathbb{E}_t^{\mathbb{Q}^N}\!\left(\frac{H_T}{N(T)}\right)$$

**Take $N = B$.** Then:

$$V(t) = B(t)\, \mathbb{E}_t^{\mathbb{Q}^B}\!\left(\frac{H_T}{B(T)}\right) = \mathbb{E}_t^{\mathbb{Q}^B}\left[D(t,T)\, H_T\right]$$

This is the risk-neutral valuation formula.

---

### 7.2 HJM Drift Restriction (Full Derivation)

**Assumptions:**
1. Forward rate diffusion: $df(t,T) = \alpha(t,T)\, dt + \sum_i \sigma_i(t,T)\, dW_i(t)$.
2. Bond-forward relation: $P(t,T) = \exp(-\int_t^T f(t,u)\, du)$.
3. No-arbitrage: $P(t,T)/B(t)$ is a $\mathbb{Q}^B$-martingale.

**Step 1:** Define $A(t,T) = \int_t^T f(t,u)\, du$, so $P(t,T) = e^{-A(t,T)}$.

**Step 2:** Differentiate $A$ using Leibniz and the forward-rate SDE:

$$dA(t,T) = -f(t,t)\, dt + \int_t^T df(t,u)$$

Since $r(t) = f(t,t)$ and using the SDE:

$$dA(t,T) = \left(-r(t) + \int_t^T \alpha(t,u)\, du\right) dt + \sum_i \Sigma_i(t,T)\, dW_i(t)$$

where $\Sigma_i(t,T) := \int_t^T \sigma_i(t,u)\, du$.

**Step 3:** Apply Ito to $P = e^{-A}$:

$$\frac{dP}{P} = -dA + \frac{1}{2}d\langle A \rangle$$

The quadratic variation is $d\langle A \rangle = \sum_i \Sigma_i^2\, dt = \|\boldsymbol{\Sigma}\|^2\, dt$.

So:

$$\frac{dP}{P} = \left(r(t) - \int_t^T \alpha(t,u)\, du + \frac{1}{2}\|\boldsymbol{\Sigma}\|^2\right) dt - \sum_i \Sigma_i\, dW_i$$

**Step 4:** Impose the risk-neutral drift condition.

For $P/B$ to be a martingale, $P$ must have drift $r(t)$. Therefore:

$$r(t) - \int_t^T \alpha(t,u)\, du + \frac{1}{2}\|\boldsymbol{\Sigma}\|^2 = r(t)$$

This gives:

$$\int_t^T \alpha(t,u)\, du = \frac{1}{2}\|\boldsymbol{\Sigma}(t,T)\|^2 = \frac{1}{2}\sum_i \left(\int_t^T \sigma_i(t,s)\, ds\right)^2$$

**Step 5:** Differentiate with respect to $T$:

$$\alpha(t,T) = \sum_i \sigma_i(t,T) \int_t^T \sigma_i(t,s)\, ds$$

This is the HJM drift restriction. ∎

---

## 8. Worked Examples

### Example 1: One-Period SDF Pricing

Two states $\omega_1, \omega_2$ with $\mathbb{P}(\omega_i) = 0.5$. Payoff at $T = 1$:
- $H(\omega_1) = 120$
- $H(\omega_2) = 80$

Risk-free rate 5%, so $P(0,1) = 1/1.05 = 0.952381$.

Pick SDF with $m(\omega_1) = 0.90$. Then:
$$0.5 \cdot 0.90 + 0.5 \cdot m(\omega_2) = 0.952381 \implies m(\omega_2) = 1.004762$$

**Price:**
$$V_0 = 0.5(0.90 \cdot 120) + 0.5(1.004762 \cdot 80) = 0.5(108 + 80.38) = 94.19$$

---

### Example 2: Risk-Neutral Verification

Risk-neutral probabilities:
$$q_i = \frac{p_i \cdot m(\omega_i)}{\mathbb{E}[m]}$$

$$q_1 = \frac{0.5 \cdot 0.90}{0.952381} = 0.4725, \quad q_2 = 0.5275$$

Risk-neutral expected payoff:
$$\mathbb{E}^{\mathbb{Q}}[H] = 0.4725 \cdot 120 + 0.5275 \cdot 80 = 98.9$$

Discounted price:
$$V_0 = 0.952381 \cdot 98.9 = 94.19 \quad \checkmark$$

---

### Example 3: Forward Price via Bond Numeraire

Bond prices: $P(0, T_2) = 0.95$, $P(0, T_3) = 0.90$.

Forward price of $T_3$-bond for delivery at $T_2$:
$$F(0; T_2) = \frac{P(0, T_3)}{P(0, T_2)} = \frac{0.90}{0.95} = 0.9474$$

---

### Example 4: Cross-Numeraire Consistency

One period, stochastic short rate:
- $B_T(\omega_1) = 1.05 \Rightarrow D(\omega_1) = 0.9524$
- $B_T(\omega_2) = 1.10 \Rightarrow D(\omega_2) = 0.9091$

Bond price: $P(0,T) = 0.5(0.9524 + 0.9091) = 0.9307$.

Payoff: $H(\omega_1) = 120$, $H(\omega_2) = 80$.

**(i) Under bank-account numeraire:**
$$V_0 = 0.5(0.9524 \cdot 120 + 0.9091 \cdot 80) = 93.51$$

**(ii) Under bond numeraire:**

Forward-measure probabilities:
$$q_1^T = \frac{0.5 \cdot 0.9524}{0.9307} = 0.5116, \quad q_2^T = 0.4884$$

$$\mathbb{E}^T[H] = 0.5116 \cdot 120 + 0.4884 \cdot 80 = 100.47$$

$$V_0 = 0.9307 \cdot 100.47 = 93.51 \quad \checkmark$$

---

### Example 5: Forward Rate from Bond Prices

$P(0, T_1) = 0.97$, $P(0, T_2) = 0.94$, $\tau = 0.5$.

$$F(0; T_1, T_2) = \frac{1}{0.5}\left(\frac{0.97}{0.94} - 1\right) = 2 \cdot 0.0319 = 6.38\%$$

---

### Example 6: GBM Measure Change

Under $\mathbb{P}$: $dS/S = \mu\, dt + \sigma\, dW$ with $\mu = 8\%$, $r = 3\%$, $\sigma = 20\%$.

Market price of risk: $\theta = (0.08 - 0.03)/0.20 = 0.25$.

Under $\mathbb{Q}$: $dS/S = r\, dt + \sigma\, dW^{\mathbb{Q}}$.

Expected values at $T = 1$:
- $\mathbb{E}^{\mathbb{P}}[S(1)] = 100 e^{0.08} = 108.33$
- $\mathbb{E}^{\mathbb{Q}}[S(1)] = 100 e^{0.03} = 103.05$

---

### Example 7: HJM Drift (Constant Volatility)

One-factor HJM with $\sigma(t,T) \equiv \sigma = 0.02$.

$$\alpha(t,T) = \sigma \int_t^T \sigma\, ds = \sigma^2 (T - t) = 0.0004 \cdot (T - t)$$

For $T = 3$, $t = 0$: $\alpha(0,3) = 0.0012$.

---

### Example 8: Volatility Zero → Deterministic Evolution

Set $\sigma \equiv 0$. Then $\alpha \equiv 0$, so $f(t,T) = f(0,T)$ (constant).

With flat forwards at 3%:
$$P(0,2) = e^{-0.06} = 0.9418, \quad P(1,2) = e^{-0.03} = 0.9704$$

---

### Example 9: Wrong Drift Creates Arbitrage

With $\sigma = 0.02$, $T = 2$: $\Sigma(0,2) = 0.04$.

If we incorrectly set $\alpha \equiv 0$, the discounted bond has drift:
$$\frac{1}{2}(0.04)^2 = 0.0008 \text{ per year}$$

Over 1 year, expected growth factor $\approx 1.0008$—a predictable gain, violating the martingale property.

---

### Example 10: Martingale Check

Verify forward rate is martingale under $\mathbb{Q}^T$:

$$F(t;S,T) = \frac{P(t,S) - P(t,T)}{\tau \cdot P(t,T)}$$

The numerator/denominator ratio has $P(t,T)$ as numeraire, hence martingale under $\mathbb{Q}^T$.

---

### Example 11: Caplet Pricing Under Forward Measure

**Setup:** Caplet on 3M LIBOR $L(T_1, T_2)$ with strike $K = 5\%$, $\tau = 0.25$, paying at $T_2$.

**Market data:** $P(0, T_1) = 0.98$, $P(0, T_2) = 0.97$, forward rate volatility $\sigma_L = 20\%$, $T_1 = 1$ year.

**Forward rate:**
$$L(0; T_1, T_2) = \frac{1}{0.25}\left(\frac{0.98}{0.97} - 1\right) = 4 \cdot 0.01031 = 4.124\%$$

**Pricing under $\mathbb{Q}^{T_2}$:**

Since $L(t; T_1, T_2)$ is a martingale under $\mathbb{Q}^{T_2}$, we model it as lognormal:
$$L(T_1) = L(0) \exp\left(-\frac{1}{2}\sigma_L^2 T_1 + \sigma_L W_{T_1}\right)$$

**Black's formula:**
$$\text{Caplet} = \tau \cdot P(0, T_2) \cdot [L(0) N(d_1) - K N(d_2)]$$

where:
$$d_1 = \frac{\ln(L(0)/K) + \frac{1}{2}\sigma_L^2 T_1}{\sigma_L \sqrt{T_1}}, \quad d_2 = d_1 - \sigma_L \sqrt{T_1}$$

**Numerical calculation:**
$$d_1 = \frac{\ln(0.04124/0.05) + 0.5 \cdot 0.04 \cdot 1}{0.20 \cdot 1} = \frac{-0.1928 + 0.02}{0.20} = -0.864$$
$$d_2 = -0.864 - 0.20 = -1.064$$

$$N(d_1) \approx 0.194, \quad N(d_2) \approx 0.144$$

$$\text{Caplet} = 0.25 \cdot 0.97 \cdot [0.04124 \cdot 0.194 - 0.05 \cdot 0.144]$$
$$= 0.2425 \cdot [0.00800 - 0.00720] = 0.2425 \cdot 0.00080 = 0.000194$$

**Per \$1 notional:** 0.0194% or about 1.94 bps.

---

### Example 12: Swaption Pricing Under Swap Measure

**Setup:** 1Y into 2Y payer swaption (right to pay fixed), strike $K = 4\%$, semiannual payments.

**Market data:** Swap rate $S(0) = 3.8\%$, swap rate volatility $\sigma_S = 15\%$, annuity $A(0) = 1.85$.

**Pricing under $\mathbb{Q}^A$:**

Under the swap measure, $S(t)$ is a martingale. Model as lognormal:
$$S(T) = S(0) \exp\left(-\frac{1}{2}\sigma_S^2 T + \sigma_S W_T\right)$$

**Black's formula for payer swaption:**
$$\text{Swaption} = A(0) \cdot [S(0) N(d_1) - K N(d_2)]$$

where $T = 1$ (option expiry):
$$d_1 = \frac{\ln(0.038/0.04) + 0.5 \cdot 0.0225 \cdot 1}{0.15 \cdot 1} = \frac{-0.0513 + 0.01125}{0.15} = -0.267$$
$$d_2 = -0.267 - 0.15 = -0.417$$

$$N(d_1) \approx 0.395, \quad N(d_2) \approx 0.338$$

$$\text{Swaption} = 1.85 \cdot [0.038 \cdot 0.395 - 0.04 \cdot 0.338]$$
$$= 1.85 \cdot [0.01501 - 0.01352] = 1.85 \cdot 0.00149 = 0.00276$$

**Per \$1 notional:** 27.6 bps.

---

### Example 13: Cross-Currency Sketch

**Setup:** USD investor wants to price a EUR-denominated cashflow of €1 at $T = 1$.

**Key insight:** Under the USD risk-neutral measure $\mathbb{Q}^{\text{USD}}$, EUR assets must be converted via FX.

Let $X(t)$ = USD per EUR. Under $\mathbb{Q}^{\text{USD}}$, the drift of $X$ is:
$$\frac{dX}{X} = (r^{\text{USD}} - r^{\text{EUR}})\, dt + \sigma_X\, dW^{\text{USD}}$$

**USD price of €1 at $T$:**
$$V^{\text{USD}}(0) = \mathbb{E}^{\mathbb{Q}^{\text{USD}}}\left[\frac{X(T)}{B^{\text{USD}}(T)}\right] = X(0) \cdot P^{\text{EUR}}(0,T)$$

by covered interest parity (the FX forward adjusts for rate differentials).

---

## 9. Summary

### Key Takeaways

1. **No-arbitrage ⟺ SDF existence:** Duffie's framework ties internally consistent prices to a strictly positive state-price deflator.

2. **SDF interpretation:** The SDF discounts payoffs more in bad states (high marginal utility). Risk premia arise from SDF covariance.

3. **Numeraire pricing:** Under measure $\mathbb{Q}^N$, price is $V(t) = N(t)\, \mathbb{E}^N[H_T/N(T)]$.

4. **Change of numeraire:** The Radon-Nikodym derivative is $\frac{U(T) N(0)}{U(0) N(T)}$.

5. **Forward measures:** The $T$-forward measure uses $P(t,T)$ as numeraire; forward rates for payment at $T$ are martingales.

6. **Swap measure:** Using the annuity as numeraire makes the swap rate a martingale—enabling Black swaption formulas.

7. **HJM drift restriction:** $\alpha(t,T) = \sigma(t,T) \int_t^T \sigma(t,s)\, ds$ under $\mathbb{Q}^B$.

8. **Why the integral:** Bond prices are exponentials of integrated forwards; Ito's $\frac{1}{2}(\text{vol})^2$ correction must be canceled by the forward-rate drift.

9. **Girsanov:** Measure change alters drift, not volatility.

10. **Practice:** Choose the numeraire that makes your payoff's key rate a martingale—then Black-style formulas apply.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| State-price deflator | Positive process $\pi$ pricing via $V = \frac{1}{\pi_t}\mathbb{E}[\pi_T H_T]$ | Foundation of no-arbitrage pricing |
| Numeraire | Positive traded asset used as unit of account | Different choices simplify different payoffs |
| Equivalent martingale measure | Measure under which $Z/N$ is martingale | Enables risk-neutral pricing |
| $T$-forward measure | Measure from bond numeraire $P(t,T)$ | Makes forward rates martingales |
| Swap measure | Measure from annuity numeraire | Makes swap rates martingales |
| HJM drift restriction | $\alpha = \sigma \int \sigma$ | Ensures arbitrage-free curve dynamics |
| Girsanov's theorem | Measure change affects drift only | How to switch between measures |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does no-arbitrage imply in Duffie's framework? | Existence of a state-price deflator (SDF). |
| 2 | What is a state-price deflator? | Positive process $\pi(t)$ pricing via $V = \frac{1}{\pi_t}\mathbb{E}[\pi_T H_T]$. |
| 3 | How does the SDF relate to marginal utility? | SDF equals intertemporal marginal rate of substitution in equilibrium. |
| 4 | What is an Arrow-Debreu security? | Pays \$1 in one state, \$0 otherwise; prices reveal state-price density. |
| 5 | What is a numeraire? | Strictly positive traded asset used as pricing unit. |
| 6 | Under $\mathbb{Q}^N$, what is a martingale? | $Z(t)/N(t)$ for any traded $Z$. |
| 7 | Give the numeraire pricing formula. | $V(t) = N(t)\, \mathbb{E}^N[H_T/N(T)]$. |
| 8 | Radon-Nikodym derivative for numeraire change? | $\frac{U(T) N(0)}{U(0) N(T)}$. |
| 9 | What is the $T$-forward measure's numeraire? | The bond $P(t,T)$. |
| 10 | Price under $T$-forward measure? | $V(t) = P(t,T)\, \mathbb{E}^T[H_T]$. |
| 11 | Define instantaneous forward rate. | $f(t,T) = -\partial_T \ln P(t,T)$. |
| 12 | Bond-forward relation? | $P(t,T) = \exp(-\int_t^T f(t,u)\, du)$. |
| 13 | HJM forward-rate SDE? | $df = \alpha\, dt + \sigma\, dW$. |
| 14 | HJM drift restriction under $\mathbb{Q}^B$? | $\alpha(t,T) = \sigma(t,T) \int_t^T \sigma(t,s)\, ds$. |
| 15 | Why does the integral $\int_t^T \sigma\, ds$ appear? | Bond vol is integrated forward-rate vol; Ito correction requires compensating drift. |
| 16 | What does Girsanov change? | Drift changes; volatility unchanged. |
| 17 | Swap measure numeraire? | Swap annuity $A(t) = \sum \tau_i P(t,T_i)$. |
| 18 | Under swap measure, what is a martingale? | The forward swap rate. |
| 19 | Why use forward measure for caplets? | Forward LIBOR is martingale → Black formula applies. |
| 20 | Why use swap measure for swaptions? | Swap rate is martingale → Black formula applies. |
| 21 | Market price of risk in GBM? | $\theta = (\mu - r)/\sigma$. |
| 22 | If $\sigma = 0$ in HJM, what is $\alpha$? | Zero (deterministic evolution). |
| 23 | How to test HJM implementation? | Verify discounted bonds have zero drift in simulation. |
| 24 | Mistake: independently fitting $\alpha$ and $\sigma$? | Violates no-arbitrage; drift determined by volatility. |
| 25 | Cross-numeraire check? | Same payoff, different numeraires → same price. |

---

## Mini Problem Set

1. In a one-period model, construct an SDF and verify it prices a payoff correctly.

2. Show that normalizing the SDF gives risk-neutral probabilities with $V_0 = P(0,1) \mathbb{E}^{\mathbb{Q}}[H]$.

3. Prove that $Z(t)/N(t)$ is a martingale under $\mathbb{Q}^N$.

4. Derive the Radon-Nikodym derivative between two numeraire measures.

5. Using $P(t,T)$ as numeraire, derive the forward-measure pricing formula.

6. Show simple forward rate $F(t;S,T)$ is a $\mathbb{Q}^T$-martingale.

7. In GBM, compute the market price of risk and write $\mathbb{Q}$-dynamics.

8. For constant-volatility HJM, compute $\alpha(t,T)$.

9. Explain why setting $\alpha$ independently of $\sigma$ creates arbitrage.

10. Derive bond price volatility from integrated forward-rate volatility.

11. Under the $T_F$-forward measure, how does HJM drift change qualitatively?

12. Price a caplet using forward-measure Black formula (with given data).

13. Price a swaption using swap-measure Black formula (with given data).

14. Explain why caps and swaptions naturally use different measures.

15. Give units for $\sigma(t,T)$ and $\alpha(t,T)$ in HJM.

16. Design a simulation test to validate HJM implementation.

---

### Solution Sketches (1–8)

1. Pick $m(\omega) > 0$ with $\mathbb{E}[m] = P(0,1)$. Compute $V_0 = \mathbb{E}[m H]$.

2. Define $q_i = p_i m_i / \mathbb{E}[m]$. Show $V_0 = \mathbb{E}[m] \cdot \mathbb{E}^{\mathbb{Q}}[H] = P(0,1) \mathbb{E}^{\mathbb{Q}}[H]$.

3. Use numeraire pricing identity and tower property: $\mathbb{E}^N[\frac{Z_t}{N_t}|\mathcal{F}_s] = \frac{Z_s}{N_s}$.

4. Equate pricing under $N$ and $U$; solve for density converting $\mathbb{E}^N$ to $\mathbb{E}^U$.

5. Substitute $N(t) = P(t,T)$ into numeraire formula.

6. $F = (P_S - P_T)/(\tau P_T)$ is ratio of traded prices; martingale under $\mathbb{Q}^T$.

7. $\theta = (\mu - r)/\sigma$. Under $\mathbb{Q}$: $dS/S = r\, dt + \sigma\, dW^{\mathbb{Q}}$.

8. With $\sigma$ constant: $\alpha(t,T) = \sigma^2(T-t)$.

---

## Source Map

### (A) Verified Facts

| Fact | Source |
|------|--------|
| Formal arbitrage definition (finite-state) | Duffie, *Dynamic Asset Pricing Theory* Ch. 1 |
| Fundamental Theorem (Separating Hyperplane proof) | Duffie, *Dynamic Asset Pricing Theory* Ch. 1 |
| SDF definition, no-arbitrage equivalence | Duffie, *Dynamic Asset Pricing Theory* Ch. 1-2 |
| SDF as marginal rate of substitution | Duffie Ch. 1–2; Cochrane Ch. 1 |
| Arrow-Debreu interpretation | Cochrane, *Asset Pricing* Ch. 1–4 |
| Numeraire definition (Geman et al. 1995) | Brigo-Mercurio, *Interest Rate Models* Ch. 2, Def. 2.2.1 |
| Self-financing invariance under numeraire change | Brigo-Mercurio Ch. 2 (Geman et al. 1995) |
| Numeraire theorem, change-of-numeraire formula | Brigo-Mercurio, *Interest Rate Models* Ch. 2 |
| "Three Facts" for numeraire changes | Brigo-Mercurio Ch. 2 |
| Forward measure pricing | Brigo-Mercurio Ch. 2 |
| Diffusion invariance principle | Andersen-Piterbarg, *Interest Rate Modeling* Vol. 1, §1.5 |
| HJM drift restriction | Brigo-Mercurio Ch. 5; Andersen-Piterbarg Vol. 1 |
| HJM drift under forward measure | Glasserman, *Monte Carlo Methods* Ch. 3 |
| Black caplet/swaption formulas | Brigo-Mercurio Ch. 6 |

### (B) Reasoned Inference

- Step-by-step HJM derivation (§7.2) follows from applying Ito to $P = e^{-A}$ and enforcing martingale condition
- Unit checks throughout from dimensional analysis
- "Why the integral appears" intuition synthesized from multiple sources
- Connection between bond/money-market numeraires and risk-neutral vs forward measures (§1.8) derived from standard martingale theory

### (C) Flagged Uncertainties

- None; all content is source-backed or derived via standard algebra

---

*Appendix A1 — Last Updated: January 2026*
