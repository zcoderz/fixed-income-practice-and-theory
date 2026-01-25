# Appendix A1: No-Arbitrage Pricing, Numeraires, and Measure Changes (Rates Toolkit) — Bridging to HJM Drift Restrictions

---

## Conventions & Notation

**Time:** continuous time $t \in [0, T^*]$ (calendar time measured in years).

**Information:** $(\mathcal{F}_t)_{t \geq 0}$ is the market filtration (what is known up to time $t$).

**Measures (probability laws):**

| Measure | Description |
|---------|-------------|
| $\mathbb{P}$ | "Physical" / reference measure (real-world probabilities) |
| $\mathbb{Q}^N$ | Numeraire-associated pricing measure induced by a chosen strictly-positive numeraire $N(t)$. Under $\mathbb{Q}^N$, any traded price divided by $N$ is a martingale (key theorem) |
| $\mathbb{Q}^B$ | The measure associated with the money-market account $B(t)$ (this is the common "risk-neutral" measure in rates when $B$ is the numeraire) |
| $\mathbb{Q}^T$ | The $T$-forward measure associated with the zero-coupon bond numeraire $P(t,T)$ |
| $\mathbb{Q}^C$ | The swap (annuity) measure associated with a swap annuity $C(t) = \sum_i \tau_i P(t, T_i)$ when supported (used for swaption-style payoffs) |

**Numeraires:**

- $B(t)$: money-market account (bank account), strictly positive, typically
$$B(t) = B(0) \exp\!\left(\int_0^t r(s)\, ds\right),$$
where $r(t)$ is the short rate.

- $P(t,T)$: default-free zero-coupon bond price (pays 1 at $T$).

**Discounting:**

Discount factor (money-market discount):
$$D(t,T) = \frac{B(t)}{B(T)} = \exp\!\left(-\int_t^T r(s)\, ds\right).$$

(This is consistent with bank-account discounting and risk-neutral valuation in the sources.)

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

### Conventions used in this appendix

- All prices are in a single currency and are nominal units (dimension: currency).
- A "traded asset" means an asset with an observable price process.
- Unless otherwise stated, we assume no intermediate cash flows (dividends) to keep notation light; the sources allow dividends, but many identities are clearest without them.
- Rates are in "per-year" units (dimension $1/\text{year}$).

### Notation glossary (symbols + definitions)

| Symbol | Meaning | Dimension |
|--------|---------|-----------|
| $V(t)$ | Time-$t$ price of a claim | currency |
| $H_T$ | Payoff at time $T$ | currency |
| $N(t)$ | Numeraire | currency, strictly positive |
| $B(t)$ | Money-market account numeraire | currency |
| $P(t,T)$ | Zero-coupon bond price | currency per unit payoff |
| $D(t,T) = B(t)/B(T)$ | Discount factor | dimensionless |
| $\pi(t)$ | State-price deflator / pricing kernel / SDF process | $1/\text{currency}$, used so that $V(t) = \frac{1}{\pi(t)} \mathbb{E}_t[\pi(T) H_T]$ |
| $\mathbb{Q}^N$ | Measure associated with numeraire $N$ | — |
| $f(t,T)$ | Instantaneous forward rate | $1/\text{year}$ |
| $r(t)$ | Short rate | $1/\text{year}$ |
| $\sigma(t,T)$ | HJM forward-rate volatility | units discussed in §5–§7 |

---

## 1. Core Concepts (Definitions First)

### 1.1 Arbitrage / No-Arbitrage

**Formal definition (as used in the sources):**

Duffie defines an arbitrage as a trading strategy $\theta$ whose associated dividend (cashflow) process $\delta^\theta$ is strictly positive (in the strict sense adopted there).

No-arbitrage is the absence of such strategies; in Duffie's development, no-arbitrage is tightly linked to existence of a state-price deflator (SDF).

**Intuition:**

Arbitrage = free lunch: a self-financing way to generate nonnegative cashflows with strictly positive payoff somewhere.

No-arbitrage means prices are internally consistent across all traded assets: you cannot lock in profit "by algebra."

**Trading / Risk / Portfolio Practice:**

Detecting arbitrage is often a consistency check: e.g., curve shapes + vol surfaces + model drifts must not imply predictable gains in discounted terms.

---

### 1.2 State-Price Density / Stochastic Discount Factor (SDF) / Pricing Kernel

**Formal definition:**

A state-price deflator $\pi(t)$ (Duffie also uses "state-price density" and relates to pricing kernel/marginal rate of substitution terminology) is a strictly positive adapted process that prices dividends via

$$S_t = \frac{1}{\pi_t}\, \mathbb{E}_t\!\left(\sum_{j=t}^{T} \pi_j\, \delta_j\right)$$

in discrete time (the key object is the same in continuous time).

The deflator characterization is equivalent to "deflated gains are martingales," and no-arbitrage holds iff a state-price deflator exists (Duffie's theorem).

Duffie explicitly notes naming variations: "state-price deflator," "state-price density," and links to "pricing kernel / marginal rates of substitution."

**Intuition:**

$\pi(t)$ is a random discount factor: it discounts payoffs more in "bad states" (high marginal utility) and less in "good states."

**Trading / Risk / Portfolio Practice:**

In rates/credit, the SDF viewpoint explains why "risk premia" exist under $\mathbb{P}$ and why pricing is naturally expressed under a measure where discounted prices are martingales.

---

### 1.3 Numeraire

**Formal definition:**

A numeraire is a strictly positive, non-dividend-paying asset chosen as a reference unit for pricing; it must be tradable and strictly positive.

**Intuition:**

Changing numeraire changes the unit of account (e.g., dollars vs "units of the bank account" vs "units of a bond maturing at $T$").

**Trading / Risk / Portfolio Practice:**

Pick the numeraire that makes the underlying of your payoff "simple" (often a martingale), reducing drift terms and simplifying valuation.

---

### 1.4 Equivalent Martingale Measure (Risk-Neutral Measure)

**Formal definition:**

Duffie defines a martingale measure $\mathbb{Q}$ (equivalent to $\mathbb{P}$) such that discounted asset values satisfy a martingale property and prices can be computed as discounted expectations under $\mathbb{Q}$.

In Brigo–Mercurio's numeraire language: under the measure $\mathbb{Q}^N$ associated with numeraire $N$, any traded price divided by $N$ is a martingale.

**"Discounted prices are martingales" means:**

Under $\mathbb{Q}^B$ (bank-account numeraire $B$), for a traded asset price $S(t)$,

$$\frac{S(t)}{B(t)} \text{ is a martingale under } \mathbb{Q}^B.$$

A key implication shown in the sources: if $S(t)$ has diffusion dynamics under $\mathbb{Q}^B$, its drift must be $r(t)$ (the short rate).

**Practice:**

"Risk-neutral probability" is not a belief about the real world; it's a computational device that depends on the numeraire.

---

### 1.5 Radon–Nikodym Derivative and Change of Measure

**Formal definition:**

For two equivalent measures $\mathbb{P}^* \sim \mathbb{P}$, the Radon–Nikodym derivative $\left.\frac{d\mathbb{P}^*}{d\mathbb{P}}\right|_{\mathcal{F}_t} = \rho_t$ is a (positive) $\mathcal{F}_t$-measurable density process linking probabilities/expectations.

Duffie uses the standard relationship: $d\mathbb{Q}/d\mathbb{P}$ defines how $\mathbb{Q}$-expectations are computed from $\mathbb{P}$-expectations.

**Intuition:**

Measure change = reweighting paths/scenarios; drifts change, diffusion coefficients remain the same in Ito form (Girsanov).

**Practice:**

Forward measures and swap measures are measure changes induced by selecting bond or annuity numeraires.

---

### 1.6 Forward Measure ($T$-Forward Measure)

**Formal definition:**

The $T$-forward measure $\mathbb{Q}^T$ is the measure induced by the numeraire $P(t,T)$ (a bond maturing at $T$); a central pricing identity is

$$\boxed{\text{Price at } t: \quad V(t) = P(t,T)\, \mathbb{E}_t^T\!(H_T)}$$

for payoffs paid at $T$.

**Intuition:**

If everything happens at time $T$, using $P(t,T)$ as the unit often removes discounting complexities.

**Practice:**

Caplets/FRA-style claims naturally align with forward measures for their payment dates (this is the conceptual basis of many "Black-style" formulas in market models).

---

### 1.7 Swap Measure (When Supported)

**Formal definition (supported case):**

Using the swap annuity $C(t)$ as numeraire defines a forward-swap measure $\mathbb{Q}^C$. Under $\mathbb{Q}^C$, the forward swap rate is a martingale by definition, and swaption-like payoffs can be written as annuity $\times$ expectation under $\mathbb{Q}^C$.

**Intuition:**

Choose the numeraire so the swap rate has zero drift, enabling lognormal approximations / Black formulas under that measure.

**Practice:**

Standard swaption desk modeling frequently leans on this measure choice for tractability (as indicated in the sources).

---

### 1.8 Zero-Coupon Bond Prices, Money-Market Account, Discounting

**Formal definitions:**

- $P(t,T)$: time-$t$ price of a bond paying 1 at $T$.
- $B(t)$: bank account (money-market) that accrues at the short rate $r(t)$.

**Intuition:**

$P(t,T)$ is a discount factor for maturity $T$ in a default-free world (possibly stochastic).

**Practice:**

Everything in rates reduces to decomposing cashflows into bond-like building blocks.

---

### 1.9 Instantaneous Forward Rate $f(t,T)$ and Short Rate $r(t)$

**Formal definitions:**

$$f(t,T) = -\partial_T \ln P(t,T).$$

$$P(t,T) = \exp\left(-\int_t^T f(t,u)\, du\right).$$

$$r(t) = f(t,t).$$

**Intuition:**

$f(t,T)$ is the "instantaneous marginal" rate for maturity $T$; $r(t)$ is the limiting short end.

**Practice:**

HJM models specify dynamics for $f(t,T)$ across maturities (the entire curve).

---

## 2. No-Arbitrage Pricing: From SDF to Risk-Neutral Valuation

### 2.1 Start with the SDF Pricing Equation (Source-Backed)

For a payoff $H_T$ at time $T$, Duffie's state-price deflator pricing relation implies the time-$t$ price can be written as

$$V(t) = \frac{1}{\pi(t)}\, \mathbb{E}_t^{\mathbb{P}}\!\left[\pi(T)\, H_T\right],$$

which is the single-payoff specialization of the discrete-time dividend pricing identity.

**Unit check:**

- $H_T$ has units currency.
- $\pi(T)$ has units $1/\text{currency}$ so $\pi(T) H_T$ is dimensionless.
- $\frac{1}{\pi(t)}$ returns currency. ✓

---

### 2.2 Choosing the Money-Market Account as Numeraire $\Rightarrow$ a Risk-Neutral Measure

Brigo–Mercurio's numeraire theorem says: if $N$ is a numeraire and $\mathbb{Q}^N$ the associated measure, then

$$V(t) = N(t)\, \mathbb{E}_t^{\mathbb{Q}^N}\!\left(\frac{H_T}{N(T)}\right).$$

Setting $N = B$ (the money-market account) yields

$$V(t) = B(t)\, \mathbb{E}_t^{\mathbb{Q}^B}\!\left(\frac{H_T}{B(T)}\right) = \mathbb{E}_t^{\mathbb{Q}^B}\!\left(D(t,T)\, H_T\right),$$

where $D(t,T) = B(t)/B(T)$ is the discount factor.

Duffie's continuous-time risk-neutral valuation formula is consistent with this bank-account discounting: prices can be written as expectations under a martingale measure with discounting at the short rate.

---

### 2.3 What Changes vs What Does Not Change When Switching Measures

**Payoffs do not change:** $H_T$ is the same contractual random variable under any measure.

**Drifts / probabilities change:** the law of the underlying processes changes; in diffusion settings, Girsanov implies drift changes while volatility stays the same in Ito form.

**Prices are invariant (if you numeraire-adjust correctly):**

$$V(t) = N(t)\, \mathbb{E}_t^{\mathbb{Q}^N}\!\left(\frac{H_T}{N(T)}\right) = U(t)\, \mathbb{E}_t^{\mathbb{Q}^U}\!\left(\frac{H_T}{U(T)}\right),$$

for any two numeraires $N, U$, provided the measures are linked by the correct Radon–Nikodym derivative (next section).

---

## 3. Change of Numeraire Theorem (The Workhorse)

### 3.1 Statement (Source-Backed)

Let $N(t)$ be a numeraire and $\mathbb{Q}^N$ its associated measure. Then for any traded asset with price $Z(t)$,

$$\frac{Z(t)}{N(t)} \text{ is a } \mathbb{Q}^N\text{-martingale.}$$

For a claim paying $Z(T)$ at time $T$,

$$\boxed{Z(t) = N(t)\, \mathbb{E}_t^{\mathbb{Q}^N}\!\left(\frac{Z(T)}{N(T)}\right).} \tag{CON pricing}$$

---

### 3.2 Derive the Radon–Nikodym Derivative Between Two Numeraire Measures (Step-by-Step)

Take two numeraires $N(t)$ and $U(t)$, with measures $\mathbb{Q}^N$ and $\mathbb{Q}^U$.

From the pricing identity under each numeraire:

$$Z(t) = N(t)\, \mathbb{E}_t^{\mathbb{Q}^N}\!\left(\frac{Z(T)}{N(T)}\right) = U(t)\, \mathbb{E}_t^{\mathbb{Q}^U}\!\left(\frac{Z(T)}{U(T)}\right).$$

Brigo–Mercurio gives the Radon–Nikodym derivative connecting the two measures:

$$\boxed{\left.\frac{d\mathbb{Q}^U}{d\mathbb{Q}^N}\right|_{\mathcal{F}_T} = \frac{U(T)\, N(0)}{U(0)\, N(T)}.} \tag{RN-CON}$$

This density ensures that for any integrable $\mathcal{F}_T$-measurable random variable $X$,

$$\mathbb{E}^{\mathbb{Q}^U}[X] = \mathbb{E}^{\mathbb{Q}^N}\!\left[X \cdot \left.\frac{d\mathbb{Q}^U}{d\mathbb{Q}^N}\right|_{\mathcal{F}_T}\right],$$

consistent with standard Radon–Nikodym logic.

---

### 3.3 General Martingale Statement (The "Numeraire-Martingale" Rule)

Under $\mathbb{Q}^N$, any traded $Z(t)$ satisfies:

$$\boxed{\mathbb{E}^{\mathbb{Q}^N}\!\left[\frac{Z(t)}{N(t)} \,\Big|\, \mathcal{F}_u\right] = \frac{Z(u)}{N(u)} \quad (u \leq t).} \tag{Mart}$$

---

### 3.4 Sanity Checks

**Price invariance across numeraires:** pricing $Z(t)$ under $N$ or $U$ must match; if not, you have mixed measures or used a wrong density.

**Deterministic rates limiting case:** if $B(t)$ and $P(t,T)$ are deterministic, many forward measures coincide (since the RN derivative becomes deterministic), and expectations under different measures agree (only discounting differs).

**Unit check:**

$\frac{d\mathbb{Q}^U}{d\mathbb{Q}^N}$ is dimensionless. In (RN-CON), $\frac{U(T)}{N(T)}$ is dimensionless (currency/currency), and $\frac{N(0)}{U(0)}$ is dimensionless. ✓

---

## 4. Bond Numeraires and Forward Measures (Rates Toolkit)

### 4.1 $T$-Forward Measure Induced by $P(t,T)$

Let $N(t) = P(t,T)$. Then $\mathbb{Q}^T$ is the $T$-forward measure and for any payoff $H_T$ paid at time $T$,

$$\boxed{V(t) = P(t,T)\, \mathbb{E}_t^T\!(H_T).} \tag{FwdMeasurePricing}$$

**Practitioner interpretation:**

If the payoff date matches the bond numeraire maturity, you "factor out" discounting immediately as $P(t,T)$. You then only need an expectation under $\mathbb{Q}^T$.

---

### 4.2 Forward Prices as Conditional Expectations Under $\mathbb{Q}^T$

For a traded asset $S(t)$, the $T$-maturity forward price (delivery at $T$, no intermediate cashflows) is naturally

$$F(t;T) = \frac{S(t)}{P(t,T)}.$$

By the numeraire-martingale rule, $\frac{S(t)}{P(t,T)}$ is a $\mathbb{Q}^T$-martingale, so

$$F(t;T) = \mathbb{E}_t^T\!(S(T)).$$

(Interpretation: "forward price equals expectation under the forward measure.")

---

### 4.3 Simple-Compounded Forward Rates are Martingales Under the Corresponding Forward Measure (Source-Backed)

Define the simple-compounded forward rate for the period $[S,T]$ with accrual $\tau(S,T)$:

$$\boxed{F(t;S,T) = \frac{1}{\tau(S,T)}\left(\frac{P(t,S)}{P(t,T)} - 1\right).} \tag{SimpleFwd}$$

Brigo–Mercurio states that $F(t;S,T)$ is a martingale under the $T$-forward measure $\mathbb{Q}^T$.

**Practice:**

This is the conceptual backbone of "Black caplet" style pricing: if the relevant forward rate is a martingale under a forward measure, modeling it as (approximately) lognormal yields closed-form option prices (as signposted in the sources).

---

### 4.4 Swap Measure (Supported Case: Swap Annuity Numeraire)

For swaption-style claims, choosing the swap annuity $C(t)$ as numeraire leads to a forward-swap measure $\mathbb{Q}^C$, and the swap rate is a martingale under $\mathbb{Q}^C$ "by definition."

---

## 5. Bridge to HJM: Why Drift Restrictions Appear

### 5.1 HJM Modeling Object: The Forward Rate Curve $f(t,T)$

HJM models specify the dynamics of the entire instantaneous forward curve $T \mapsto f(t,T)$, linked to bonds by:

$$P(t,T) = \exp\!\left(-\int_t^T f(t,u)\, du\right), \qquad f(t,T) = -\partial_T \ln P(t,T).$$

A standard HJM diffusion specification under a given measure (here: the bank-account measure $\mathbb{Q}$ / $\mathbb{Q}^B$) is:

$$\boxed{df(t,T) = \alpha(t,T)\, dt + \sum_{i=1}^{N} \sigma_i(t,T)\, dW_i(t).} \tag{HJM-f}$$

where $W$ is an $N$-dimensional Brownian motion under that measure.

---

### 5.2 No-Arbitrage Logic: Volatility "Free," Drift "Pinned Down"

Brigo–Mercurio emphasizes: in HJM, you can choose the volatility structure (within regularity constraints), but then no-arbitrage determines the drift.

**Mechanism (conceptual):**

1. Bond prices are tradable.
2. Under the bank-account numeraire $B(t)$, discounted bond prices $P(t,T)/B(t)$ must be martingales (no predictable gains).
3. The assumed forward-rate diffusion implies a bond-price diffusion (via $P(t,T) = \exp(-\int_t^T f(t,u)\, du)$).
4. Enforcing the martingale condition forces a relationship between $\alpha$ and $\sigma$.

---

### 5.3 The HJM Drift Restriction Under the Risk-Neutral / Bank-Account Measure (Explicit, Source-Backed)

Under the risk-neutral measure (bank-account numeraire), the HJM no-arbitrage condition is:

$$\boxed{\alpha(t,T) = \sum_{i=1}^{N} \sigma_i(t,T) \int_t^T \sigma_i(t,s)\, ds.} \tag{HJM-drift-$\mathbb{Q}$}$$

Equivalently, in vector notation:

$$\alpha(t,T) = \sigma(t,T)^\top \int_t^T \sigma(t,s)\, ds.$$

The resulting bond price dynamics under this measure are:

$$\boxed{\frac{dP(t,T)}{P(t,T)} = r(t)\, dt - \sum_{i=1}^{N}\left(\int_t^T \sigma_i(t,s)\, ds\right) dW_i(t).} \tag{BondDyn-$\mathbb{Q}$}$$

and the short rate is $r(t) = f(t,t)$.

---

### 5.4 Drift Under a Forward Measure (Supported by an Additional Provided Book)

A separate provided reference (Monte Carlo Methods in Financial Engineering) gives the drift of $f(t,T)$ under a forward measure $\mathbb{Q}^{T_F}$ associated with maturity $T_F$:

$$\boxed{df(t,T) = -\,\sigma(t,T)^\top\!\left(\int_T^{T_F} \sigma(t,u)\, du\right) dt + \sigma(t,T)\, dW^{T_F}(t).} \tag{HJM-drift-forward}$$

This formula explicitly shows the measure dependence of drift: under $\mathbb{Q}^B$ the drift involves $\int_t^T$, while under $\mathbb{Q}^{T_F}$ it involves $\int_T^{T_F}$.

---

### 5.5 Sanity Checks (Required)

**If $\sigma \equiv 0$:**

(HJM-drift-$\mathbb{Q}$) gives $\alpha \equiv 0$, so $df(t,T) = 0$ (deterministic evolution), consistent with a static curve and deterministic bond prices.

**Units check:**

- $f(t,T)$ has units $1/\text{year}$.
- In $df = \alpha\, dt + \sigma\, dW$:
  - $dt$ has units year, so $\alpha$ must have units $1/\text{year}^2$.
  - $dW$ has units $\sqrt{\text{year}}$, so $\sigma$ must have units $1/\text{year}^{3/2}$.
- In (HJM-drift-$\mathbb{Q}$), $\int_t^T \sigma\, ds$ has units $1/\text{year}^{1/2}$; multiplying by $\sigma$ gives $1/\text{year}^2$. ✓

---

## 6. Practical Market Interpretation

### 6.1 What "Drift is Determined by Volatility" Means (Non-Handwavy)

- In HJM, you specify $\sigma(t,T)$ to match the empirically observed term-structure of rate volatility and derivative prices.
- No-arbitrage then forces $\alpha(t,T)$ via (HJM-drift-$\mathbb{Q}$).
- This ensures the bond drift under $\mathbb{Q}^B$ is exactly $r(t)$, making $P(t,T)/B(t)$ a martingale (no predictable gains in discounted bond prices).

---

### 6.2 Calibration Implications

- You calibrate $\sigma$ to liquid instruments (caps/floors, swaptions, etc.).
- Drift is not a "free parameter" under pricing measures; it is computed from $\sigma$.
- If you independently "fit" drift and volatility, you can easily violate the martingale condition and introduce arbitrage.

---

### 6.3 Why Too-Flexible Curve Dynamics Can Create Arbitrage

- Any choice of $\alpha, \sigma$ that breaks the required relationship makes discounted bond prices drift away from martingales.
- Practically, this shows up as inconsistent prices across instruments: e.g., the model implies different prices for replicable cashflows depending on which replication you use.

---

### 6.4 Desk Checklist (Measure + Numeraire Selection, Source-Backed Where Possible)

| Use Case | Numeraire | Measure | Rationale |
|----------|-----------|---------|-----------|
| General pricing / simulation | $B(t)$ | $\mathbb{Q}^B$ | Discounting is always $B$-based and "risk-neutral" |
| Single cashflow at $T$ | $P(t,T)$ | $\mathbb{Q}^T$ | $V(t) = P(t,T)\, \mathbb{E}^T[H_T]$ |
| FRA/caplet-type objects | $P(t,T)$ | $\mathbb{Q}^T$ | Forward rates for the payment date are martingales (conceptual basis of Black-style formulas) |
| Swaption-type objects (supported) | $C(t)$ | $\mathbb{Q}^C$ | Swap rate is a martingale under $\mathbb{Q}^C$ |

---

## 7. Math and Derivations (Step-by-Step)

### 7.1 SDF Pricing Equation $\rightarrow$ Risk-Neutral Pricing Under the Money-Market Numeraire

**Start from Duffie's SDF pricing for a single payoff:**

$$V(t) = \frac{1}{\pi(t)}\, \mathbb{E}_t^{\mathbb{P}}\!\left[\pi(T)\, H_T\right].$$

**Under a numeraire $N$, Brigo–Mercurio gives the numeraire pricing identity:**

$$V(t) = N(t)\, \mathbb{E}_t^{\mathbb{Q}^N}\!\left(\frac{H_T}{N(T)}\right).$$

**Take $N = B$ (bank account).** Then

$$V(t) = B(t)\, \mathbb{E}_t^{\mathbb{Q}^B}\!\left(\frac{H_T}{B(T)}\right) = \mathbb{E}_t^{\mathbb{Q}^B}\left[D(t,T)\, H_T\right],$$

which matches the risk-neutral valuation form discounted at the short rate (Duffie's continuous-time statement).

**Limiting case check:** deterministic $r \Rightarrow D(t,T)$ deterministic; only the payoff randomness remains.

---

### 7.2 Change-of-Numeraire: Derive the Radon–Nikodym Derivative

From Brigo–Mercurio, for numeraires $N$ and $U$,

$$\left.\frac{d\mathbb{Q}^U}{d\mathbb{Q}^N}\right|_{\mathcal{F}_T} = \frac{U(T)\, N(0)}{U(0)\, N(T)}.$$

**Derivation sketch (aligned with the theorem's structure):**

1. Write $Z(t)$ in numeraire-$N$ form:
$$Z(t) = N(t)\, \mathbb{E}_t^{\mathbb{Q}^N}\!\left(\frac{Z(T)}{N(T)}\right).$$

2. Require equality with numeraire-$U$ form:
$$Z(t) = U(t)\, \mathbb{E}_t^{\mathbb{Q}^U}\!\left(\frac{Z(T)}{U(T)}\right).$$

3. The density (RN derivative) is the unique object that converts $\mathbb{Q}^N$-expectations into $\mathbb{Q}^U$-expectations for all $Z(T)$, yielding (RN-CON).

---

### 7.3 Bond Numeraire $\rightarrow$ $T$-Forward Measure and Forward-Price Formula

Let $N(t) = P(t,T)$. Then by definition of the induced measure:

$$V(t) = P(t,T)\, \mathbb{E}_t^T\!(H_T).$$

For a traded asset $S$, the $T$-forward price $F(t;T) = S(t)/P(t,T)$ is a $\mathbb{Q}^T$-martingale (since it is a traded price divided by the numeraire), giving:

$$F(t;T) = \mathbb{E}_t^T[S(T)].$$

---

### 7.4 HJM Drift Restriction (Most Explicit Form Supported by Sources)

**Assumptions (explicit):**

1. Forward rate diffusion under the bank-account/risk-neutral measure:
$$df(t,T) = \alpha(t,T)\, dt + \sum_{i=1}^{N} \sigma_i(t,T)\, dW_i(t).$$

2. Bond/forward relation:
$$P(t,T) = \exp\!\left(-\int_t^T f(t,u)\, du\right).$$

3. No-arbitrage under $B$: discounted bond prices $P(t,T)/B(t)$ must be martingales.

---

**Step 1:** Define $A(t,T) = \int_t^T f(t,u)\, du$.

Then $P(t,T) = e^{-A(t,T)}$.

---

**Step 2:** Differentiate $A(t,T)$ (Leibniz + Ito structure).

Formally,

$$dA(t,T) = -f(t,t)\, dt + \int_t^T df(t,u),$$

because the lower integration limit moves with speed 1, contributing $-f(t,t)\, dt$. Using $r(t) = f(t,t)$ and the HJM dynamics,

$$dA(t,T) = \left(-r(t) + \int_t^T \alpha(t,u)\, du\right) dt + \sum_{i=1}^{N}\left(\int_t^T \sigma_i(t,u)\, du\right) dW_i(t).$$

Define the integrated volatility vector:

$$\Sigma_i(t,T) := \int_t^T \sigma_i(t,u)\, du, \qquad \boldsymbol{\Sigma}(t,T) = (\Sigma_1, \ldots, \Sigma_N).$$

---

**Step 3:** Apply Ito to $P(t,T) = \exp(-A)$.

If $P = e^{-A}$, then

$$\frac{dP}{P} = -\,dA + \frac{1}{2}\,d\langle A \rangle.$$

Since the diffusion part of $A$ is $\sum_i \Sigma_i\, dW_i$, the quadratic variation contributes

$$d\langle A \rangle = \sum_{i=1}^{N} \Sigma_i(t,T)^2\, dt = \|\boldsymbol{\Sigma}(t,T)\|^2\, dt.$$

So,

$$\frac{dP(t,T)}{P(t,T)} = \left(r(t) - \int_t^T \alpha(t,u)\, du + \frac{1}{2}\|\boldsymbol{\Sigma}(t,T)\|^2\right) dt - \sum_{i=1}^{N} \Sigma_i(t,T)\, dW_i(t).$$

---

**Step 4:** Impose the risk-neutral bond drift condition.

Brigo–Mercurio gives the arbitrage-free bond dynamics under the risk-neutral measure:

$$\frac{dP(t,T)}{P(t,T)} = r(t)\, dt - \sum_{i=1}^{N}\left(\int_t^T \sigma_i(t,s)\, ds\right) dW_i(t).$$

Comparing the drift terms forces

$$\int_t^T \alpha(t,u)\, du = \frac{1}{2}\|\boldsymbol{\Sigma}(t,T)\|^2.$$

Differentiating w.r.t. $T$ yields the **HJM drift restriction**:

$$\boxed{\alpha(t,T) = \sum_{i=1}^{N} \sigma_i(t,T) \int_t^T \sigma_i(t,s)\, ds,}$$

matching the explicit source formula.

**Limiting case:** $\sigma \to 0 \Rightarrow \alpha \to 0$ and bond volatility vanishes. ✓

---

## 8. Worked Examples (At Least 10 Numeric Examples)

All examples use toy numbers for transparency. Intermediate steps are shown.

---

### Example 1: One-Period Discrete-Time SDF Pricing

Two states $\omega_1, \omega_2$ with $\mathbb{P}(\omega_i) = 0.5$. Payoff at $T = 1$:

- $H(\omega_1) = 120$
- $H(\omega_2) = 80$

Assume a one-period gross risk-free return $1.05$, so the risk-free price of 1 at $T$ is

$$P(0,1) = \frac{1}{1.05} = 0.9523809524.$$

Pick an SDF $m(\omega)$ such that $\mathbb{E}[m] = P(0,1)$ and $m > 0$:

- Let $m(\omega_1) = 0.90$.
- Solve for $m(\omega_2)$:

$$0.5 \cdot 0.90 + 0.5 \cdot m(\omega_2) = 0.9523809524 \quad\Rightarrow\quad m(\omega_2) = 1.0047619048.$$

**Price:**

$$V_0 = \mathbb{E}[m H] = 0.5(0.90 \cdot 120) + 0.5(1.0047619048 \cdot 80).$$

Compute:
- $0.90 \cdot 120 = 108$
- $1.0047619048 \cdot 80 = 80.380952384$

So

$$V_0 = 0.5(108 + 80.380952384) = 94.190476192 \approx 94.19.$$

---

### Example 2: Risk-Neutral Valuation Sanity Check (Equivalence)

Define risk-neutral probabilities via normalization:

$$q_i = \frac{p_i\, m(\omega_i)}{\mathbb{E}[m]}, \qquad \mathbb{E}[m] = 0.9523809524.$$

Compute:

$$q_1 = \frac{0.5 \cdot 0.90}{0.9523809524} = 0.4725$$

$$q_2 = \frac{0.5 \cdot 1.0047619048}{0.9523809524} = 0.5275$$

Risk-neutral expected payoff:

$$\mathbb{E}^{\mathbb{Q}}[H] = 0.4725 \cdot 120 + 0.5275 \cdot 80 = 56.7 + 42.2 = 98.9.$$

Discounted price:

$$V_0 = P(0,1)\, \mathbb{E}^{\mathbb{Q}}[H] = 0.9523809524 \cdot 98.9 = \frac{98.9}{1.05} = 94.19047619 \approx 94.19,$$

matching Example 1.

---

### Example 3: Change of Numeraire with a Bond — Compute a Forward Price via $T$-Forward Measure

Let delivery date be $T_2$ and underlying be a bond maturing at $T_3 > T_2$. With deterministic bond prices at $t = 0$:

- $P(0, T_2) = 0.95$
- $P(0, T_3) = 0.90$

Forward price at $0$ for delivery at $T_2$:

$$F(0; T_2) = \frac{P(0, T_3)}{P(0, T_2)} = \frac{0.90}{0.95} = 0.9473684211 \approx 0.94737.$$

---

### Example 4: Measure Invariance Check — Same Payoff Under Two Numeraires

One-period ($T = 1$), assume under $\mathbb{Q}^B$ two states with probabilities $0.5, 0.5$. Let the bank account be stochastic:

- $B_T(\omega_1) = 1.05 \Rightarrow D(\omega_1) = 1/1.05 = 0.9523809524$
- $B_T(\omega_2) = 1.10 \Rightarrow D(\omega_2) = 1/1.10 = 0.9090909091$

Bond price:

$$P(0,T) = \mathbb{E}^{\mathbb{Q}^B}[D] = 0.5(0.9523809524 + 0.9090909091) = 0.9307359308.$$

Same payoff as before:

- $H(\omega_1) = 120$
- $H(\omega_2) = 80$

**(i) Pricing under bank-account numeraire:**

$$V_0 = \mathbb{E}^{\mathbb{Q}^B}[D H] = 0.5(0.9523809524 \cdot 120) + 0.5(0.9090909091 \cdot 80).$$

Compute:
- $0.9523809524 \cdot 120 = 114.285714288$
- $0.9090909091 \cdot 80 = 72.727272728$

So

$$V_0 = 0.5(114.285714288 + 72.727272728) = 93.506493508 \approx 93.51.$$

**(ii) Pricing under bond numeraire $P(0,T)$ (forward measure):**

Forward-measure probabilities:

$$q_i^T = \frac{q_i^B\, D_i}{P(0,T)}.$$

Compute:

$$q_1^T = \frac{0.5 \cdot 0.9523809524}{0.9307359308} \approx 0.511628$$

$$q_2^T = 1 - q_1^T \approx 0.488372$$

Forward-measure expected payoff:

$$\mathbb{E}^T[H] \approx 0.511628 \cdot 120 + 0.488372 \cdot 80 = 61.39536 + 39.06976 = 100.46512.$$

Price:

$$V_0 = P(0,T)\, \mathbb{E}^T[H] \approx 0.9307359308 \cdot 100.46512 \approx 93.51,$$

matching (i), up to rounding.

---

### Example 5: Bond Forward-Measure Pricing of 1 at $T$

Using Example 4's bond price $P(0,T) = 0.9307359308$.

Payoff $H_T \equiv 1$. Then under the $T$-forward measure:

$$V_0 = P(0,T)\, \mathbb{E}^T[1] = P(0,T) = 0.9307359308.$$

---

### Example 6: Forward Rate from Bond Prices

Let:

- $P(0, T_1) = 0.97$
- $P(0, T_2) = 0.94$
- accrual $\tau = 0.5$ years

Simple-compounded forward rate:

$$F(0; T_1, T_2) = \frac{1}{\tau}\left(\frac{P(0, T_1)}{P(0, T_2)} - 1\right) = 2\left(\frac{0.97}{0.94} - 1\right).$$

Compute:
- $0.97 / 0.94 = 1.0319148936$
- $1.0319148936 - 1 = 0.0319148936$
- Multiply by 2: $0.0638297872$

So

$$F(0; T_1, T_2) = 0.0638297872 \approx 6.383\% \text{ (simple, per year)}.$$

---

### Example 7: Toy "Drift Changes Under Measure Change" (GBM; Source-Backed)

**Under $\mathbb{P}$:**

$$\frac{dS(t)}{S(t)} = \mu\, dt + \sigma\, dW(t).$$

**Under the risk-neutral measure $\mathbb{Q}$** induced by the money-market numeraire, the drift changes to $r$:

$$\frac{dS(t)}{S(t)} = r\, dt + \sigma\, dW^\beta(t),$$

with $\theta = (\mu - r)/\sigma$ controlling the drift shift.

Take:
- $S_0 = 100$
- $\mu = 0.08$
- $r = 0.03$
- $\sigma = 0.20$

Market price of risk:

$$\theta = \frac{\mu - r}{\sigma} = \frac{0.08 - 0.03}{0.20} = \frac{0.05}{0.20} = 0.25.$$

Expected value (ignoring volatility term's mean-zero property):

- Under $\mathbb{P}$: $\mathbb{E}^{\mathbb{P}}[S(1)] \approx 100 e^{0.08} = 100 \cdot 1.083287 = 108.33$.
- Under $\mathbb{Q}$: $\mathbb{E}^{\mathbb{Q}}[S(1)] \approx 100 e^{0.03} = 100 \cdot 1.030455 = 103.05$.

**Interpretation:** the same payoff is being priced; the measure change reweights paths so that discounted prices become martingales.

---

### Example 8: HJM Drift Restriction Numeric Check (Constant Volatility; Source-Backed Formula)

One-factor HJM under $\mathbb{Q}^B$. Suppose $\sigma(t,T) \equiv \sigma = 0.02$ (constant in $t, T$).

Then the drift restriction says:

$$\alpha(t,T) = \sigma \int_t^T \sigma\, ds = \sigma^2 (T - t).$$

Compute $\sigma^2 = 0.0004$.

For $t = 0$, $T = 3$:

$$\alpha(0, 3) = 0.0004 \cdot 3 = 0.0012.$$

For $t = 0$, $T = 1$:

$$\alpha(0, 1) = 0.0004 \cdot 1 = 0.0004.$$

---

### Example 9: Limiting Case — Set Volatility to Zero $\Rightarrow$ Deterministic Evolution

Set $\sigma \equiv 0$. Then (HJM-drift-$\mathbb{Q}$) gives $\alpha \equiv 0$, so $df(t,T) = 0$: $f(t,T) = f(0,T)$.

Assume a flat forward curve at 3%:

$f(0,u) = 0.03$ for all $u$.

Then

$$P(0,2) = \exp\left(-\int_0^2 0.03\, du\right) = e^{-0.06} = 0.9417645336.$$

At $t = 1$,

$$P(1,2) = \exp\left(-\int_1^2 0.03\, du\right) = e^{-0.03} = 0.9704455335.$$

Everything is deterministic: no random drift corrections needed.

---

### Example 10: Practitioner Check — Wrong Drift Violates Discounted-Bond Martingale Property (Toy Inconsistency)

One-factor, constant $\sigma = 0.02$, maturity $T = 2$, current $t = 0$. Integrated volatility:

$$\Sigma(0,2) = \int_0^2 \sigma\, du = 0.02 \cdot 2 = 0.04.$$

From the Ito calculation in §7.4, the drift of $\frac{P(t,T)}{B(t)}$ is zero only if the HJM drift restriction holds. If you (incorrectly) set $\alpha \equiv 0$ while keeping $\sigma \neq 0$, the drift term of the discounted bond becomes

$$\frac{1}{2} \Sigma(0,2)^2 = \frac{1}{2}(0.04)^2 = \frac{1}{2} \cdot 0.0016 = 0.0008 \text{ per year}.$$

Over $\Delta t = 1$ year, the discounted bond would have an expected growth factor (approx.)

$$\exp(0.0008 \cdot 1) \approx 1.0008003,$$

i.e., a predictable positive drift in discounted terms, contradicting the martingale requirement under $\mathbb{Q}^B$.

---

## 9. Practical Notes

### Common Pitfalls

**Confusing numeraire change with currency conversion:**

Numeraire change is within one currency; it changes the pricing measure used for conditional expectations.

**Forgetting drift depends on measure:**

Girsanov: drift changes under measure change; volatility stays (in Ito form).

**Mixing measures in a derivation:**

Example mistake: using $\mathbb{Q}^B$-drift with a $\mathbb{Q}^T$-expectation.

**Treating $\mathbb{Q}$-probabilities as real-world probabilities:**

$\mathbb{Q}$ is a pricing construct linked to a numeraire; it is not a forecast.

---

### Verification Tests

**Discounted-price martingale test:**

Under $\mathbb{Q}^B$, check that $S(t)/B(t)$ has zero drift in model dynamics.

**Invariance across numeraires:**

Price the same payoff under $B$ and under $P(\cdot,T)$; results must match (Example 4).

**Limiting-case checks:**

$\sigma \to 0 \Rightarrow \alpha \to 0$ in HJM, and curve evolution becomes deterministic.

---

## 10. Summary & Recall

### 10-Bullet Executive Summary

1. No-arbitrage is equivalent to the existence of a state-price deflator (SDF) in Duffie's framework.

2. The SDF prices payoffs via $V(t) = \frac{1}{\pi(t)} \mathbb{E}_t[\pi(T) H_T]$.

3. A numeraire is a strictly positive, non-dividend-paying traded asset used as unit of account.

4. Under the measure induced by numeraire $N$, any traded $Z(t)/N(t)$ is a martingale.

5. Pricing under numeraire $N$: $V(t) = N(t)\, \mathbb{E}_t^{\mathbb{Q}^N}[H_T/N(T)]$.

6. Measures induced by different numeraires are linked by $\frac{d\mathbb{Q}^U}{d\mathbb{Q}^N} = \frac{U(T) N(0)}{U(0) N(T)}$.

7. $T$-forward measure uses bond numeraire $P(t,T)$, giving $V(t) = P(t,T)\, \mathbb{E}_t^T[H_T]$.

8. In HJM, forward rates satisfy $df = \alpha\, dt + \sigma\, dW$ and bond prices satisfy $P(t,T) = \exp(-\int_t^T f(t,u)\, du)$.

9. Under $\mathbb{Q}^B$, no-arbitrage forces $\alpha(t,T) = \sum_i \sigma_i(t,T) \int_t^T \sigma_i(t,s)\, ds$.

10. Drift depends on measure; e.g. under a forward measure, the HJM drift changes (explicit formula available in one provided reference).

---

### Cheat Sheet of Core Identities

**SDF pricing (Duffie):**
$$V(t) = \frac{1}{\pi(t)}\, \mathbb{E}_t^{\mathbb{P}}[\pi(T) H_T].$$

**Numeraire pricing (Brigo–Mercurio):**
$$V(t) = N(t)\, \mathbb{E}_t^{\mathbb{Q}^N}\!\left(\frac{H_T}{N(T)}\right).$$

**Change of numeraire:**
$$\left.\frac{d\mathbb{Q}^U}{d\mathbb{Q}^N}\right|_{\mathcal{F}_T} = \frac{U(T) N(0)}{U(0) N(T)}.$$

**$T$-forward measure pricing:**
$$V(t) = P(t,T)\, \mathbb{E}_t^T[H_T].$$

**HJM drift restriction under $\mathbb{Q}^B$:**
$$\alpha(t,T) = \sum_{i=1}^{N} \sigma_i(t,T) \int_t^T \sigma_i(t,s)\, ds.$$

**Bond dynamics under $\mathbb{Q}^B$:**
$$\frac{dP(t,T)}{P(t,T)} = r(t)\, dt - \sum_{i=1}^{N}\left(\int_t^T \sigma_i(t,s)\, ds\right) dW_i(t).$$

---

### 25 Flashcards (Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What does no-arbitrage imply in Duffie's framework? | Existence of a state-price deflator (SDF). |
| 2 | What is a state-price deflator / SDF? | A strictly positive process $\pi(t)$ that prices payoffs by $V(t) = \frac{1}{\pi(t)} \mathbb{E}_t[\pi(T) H_T]$. |
| 3 | Name two synonyms for "state-price deflator" in the sources. | State-price density; (linked to) pricing kernel / marginal rates of substitution. |
| 4 | What is a numeraire? | A strictly positive, non-dividend-paying traded asset used as unit of account. |
| 5 | Under $\mathbb{Q}^N$, what is a martingale? | $Z(t)/N(t)$ for any traded asset price $Z(t)$. |
| 6 | Give the numeraire pricing formula. | $V(t) = N(t)\, \mathbb{E}_t^{\mathbb{Q}^N}[H_T/N(T)]$. |
| 7 | How do you change from $\mathbb{Q}^N$ to $\mathbb{Q}^U$? | Use $\frac{d\mathbb{Q}^U}{d\mathbb{Q}^N} = \frac{U(T) N(0)}{U(0) N(T)}$. |
| 8 | What is the $T$-forward measure's numeraire? | The bond $P(t,T)$. |
| 9 | Price of payoff $H_T$ under $T$-forward measure? | $V(t) = P(t,T)\, \mathbb{E}_t^T[H_T]$. |
| 10 | Define instantaneous forward rate. | $f(t,T) = -\partial_T \ln P(t,T)$. |
| 11 | Relationship between bond price and forward curve? | $P(t,T) = \exp(-\int_t^T f(t,u)\, du)$. |
| 12 | What is the short rate in HJM notation? | $r(t) = f(t,t)$. |
| 13 | HJM forward-rate SDE form? | $df(t,T) = \alpha(t,T)\, dt + \sum_i \sigma_i(t,T)\, dW_i$. |
| 14 | HJM drift restriction under $\mathbb{Q}^B$? | $\alpha(t,T) = \sum_i \sigma_i(t,T) \int_t^T \sigma_i(t,s)\, ds$. |
| 15 | Bond price diffusion term under HJM risk-neutral dynamics? | $-\sum_i (\int_t^T \sigma_i(t,s)\, ds)\, dW_i$. |
| 16 | What does Girsanov change? | Drift changes; diffusion coefficient stays the same (Ito form). |
| 17 | Under risk-neutral measure for GBM, what happens to $\mu$? | $\mu$ is replaced by $r$. |
| 18 | Define the market price of risk in GBM example. | $\theta = (\mu - r)/\sigma$. |
| 19 | What is a zero-coupon bond? | Pays 1 at maturity $T$; price is $P(t,T)$. |
| 20 | What is the "swap measure" numeraire (supported case)? | The swap annuity $C(t)$ (sum of discounted accrual factors). |
| 21 | Under $\mathbb{Q}^C$, what is a martingale by definition? | The forward swap rate. |
| 22 | Why is HJM attractive? | You can specify volatility and get drift from no-arbitrage, ensuring arbitrage-free curve dynamics. |
| 23 | What's the key "martingale object" under numeraire $N$? | $Z/N$ for traded $Z$. |
| 24 | If $\sigma = 0$ in HJM, what happens to $\alpha$? | $\alpha = 0$. |
| 25 | Under a forward measure $\mathbb{Q}^{T_F}$, how does HJM drift qualitatively differ? | Drift changes from an $\int_t^T$ structure to an $\int_T^{T_F}$ structure (explicit in one reference). |

---

## 11. Mini Problem Set (16 Questions)

1. In a one-period model with two states, compute a payoff price using an SDF $m$ and verify positivity implies no-arbitrage.

2. Normalize an SDF into a risk-neutral probability and show $V_0 = \mathbb{E}[mH] = P(0,1)\, \mathbb{E}^{\mathbb{Q}}[H]$.

3. Prove that under $\mathbb{Q}^N$, $Z(t)/N(t)$ must be a martingale for traded $Z$.

4. Derive the Radon–Nikodym derivative between $\mathbb{Q}^N$ and $\mathbb{Q}^U$ given two numeraires.

5. Using $P(t,T)$ as numeraire, derive $V(t) = P(t,T)\, \mathbb{E}^T[H_T]$.

6. Show the simple forward rate $F(t;S,T) = \frac{1}{\tau}\left(\frac{P(t,S)}{P(t,T)} - 1\right)$ is a martingale under $\mathbb{Q}^T$.

7. In the GBM setting, compute $\theta = (\mu - r)/\sigma$ and write the $\mathbb{Q}$-dynamics of $S$.

8. In one-factor HJM with constant $\sigma$, compute $\alpha(t,T)$ and interpret its sign.

9. (Conceptual) Explain why choosing $\alpha$ independently of $\sigma$ can produce arbitrage in HJM.

10. Under $\mathbb{Q}^B$, show how bond price volatility depends on the integrated forward vol $\int_t^T \sigma(t,u)\, du$.

11. Under a $T_F$-forward measure, explain how the HJM drift changes qualitatively (compare integrals).

12. Discuss how the swap annuity numeraire leads to a measure under which swap rates are martingales.

13. Explain why caplets are naturally associated with forward measures.

14. Explain why swaptions are naturally associated with swap measures.

15. Give a unit-consistent interpretation of $\sigma(t,T)$ and $\alpha(t,T)$ in HJM.

16. Propose a practical workflow to validate an HJM implementation using martingale tests.

---

### Brief Solution Sketches (1–8 Only)

1. Compute $V_0 = \mathbb{E}[mH]$. Check $m > 0$ in all states; positivity supports no-arbitrage in the SDF characterization.

2. Define $q_i \propto p_i m_i$. Show $V_0 = \mathbb{E}[mH] = \mathbb{E}[m] \cdot \mathbb{E}^{\mathbb{Q}}[H]$ and $\mathbb{E}[m] = P(0,1)$.

3. Use the numeraire pricing identity $Z(t) = N(t)\, \mathbb{E}^N[Z(T)/N(T) | \mathcal{F}_t]$ and tower property to obtain martingale property for $Z/N$.

4. Set the two pricing expressions equal for arbitrary $Z(T)$ and solve for the density that converts $\mathbb{Q}^N$-expectations into $\mathbb{Q}^U$-expectations (gives $\frac{U(T) N(0)}{U(0) N(T)}$).

5. Plug $N(t) = P(t,T)$ into the numeraire pricing identity.

6. Recognize $F(t;S,T)$ is (up to scaling) $\frac{P(t,S)}{P(t,T)}$, a traded ratio; thus a martingale under $\mathbb{Q}^T$.

7. Compute $\theta = (\mu - r)/\sigma$. Under $\mathbb{Q}$, drift becomes $r$: $dS/S = r\, dt + \sigma\, dW^\beta$.

8. With $\sigma$ constant, $\alpha(t,T) = \sigma^2 (T - t)$. Larger maturities imply larger drift under $\mathbb{Q}^B$.

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

- SDF definition, no-arbitrage equivalence, pricing identity: **Duffie** (Dynamic Asset Pricing Theory)
- Numeraire theorem, change-of-numeraire formula, forward measure pricing: **Brigo & Mercurio** (Interest Rate Models: Theory and Practice)
- HJM drift restriction, bond dynamics under risk-neutral measure: **Brigo & Mercurio**, **Andersen & Piterbarg** (Interest Rate Modeling, Vol. 1–2)
- HJM drift under forward measure: **Glasserman** (Monte Carlo Methods in Financial Engineering)

### (B) Reasoned Inference — Note Derivation Logic

- Step-by-step derivation of HJM drift restriction (§7.4) follows from applying Ito's lemma to $P = \exp(-A)$ and enforcing the martingale condition
- Unit checks throughout are derived from dimensional analysis of the SDE terms

### (C) Speculation — Flag Uncertainties

- None in this appendix; all content is source-backed or derived via standard algebra
