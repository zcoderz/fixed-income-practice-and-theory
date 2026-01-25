# Appendix A4: Market Models — LMM (BGM) and Swap Market Model; Calibration Logic

---

## Conventions & Notation

### (A) Verified Facts

The following statements are directly supported by the provided references:

**Tenor / accrual structure:** We work on a discrete tenor grid $\{T_0 < T_1 < \cdots < T_n\}$ with accrual fractions $\delta_i = \tau(T_i, T_{i+1})$ (day-count year-fraction). A "LIBOR-style" simply-compounded forward rate over $[T_i, T_{i+1}]$ is defined from bond prices via

$$F_i(t) \equiv F(t; T_i, T_{i+1}) = \frac{1}{\delta_i}\left(\frac{P(t, T_i)}{P(t, T_{i+1})} - 1\right), \quad \Longleftrightarrow \quad P(t, T_{i+1}) = \frac{P(t, T_i)}{1 + \delta_i F_i(t)}.$$

**Money-market numeraire and discount factor:** With short rate $r_t$, the money-market account is $B(t) = B_0 \exp\left(\int_0^t r_s \, ds\right)$, and the stochastic discount factor over $[t, T]$ can be written as

$$D(t, T) = \frac{B(t)}{B(T)} = \exp\left(-\int_t^T r_s \, ds\right).$$

**Forward measure pricing:** For a payoff $H_T$ at time $T$, pricing under the $T$-forward measure (numeraire $P(t, T)$) is

$$\Pi_t = P(t, T) \, \mathbb{E}^T\left[H_T \mid \mathcal{F}_t\right].$$

**Black(-style) building block:** The references define a Black "$\mathrm{Bl}$" function and use it to express Black cap and swaption prices. In particular, for "call/put sign" $\omega \in \{+1, -1\}$,

$$\mathrm{Bl}(K, F, v, \omega) = \omega\left(F \, \Phi(\omega d_1) - K \, \Phi(\omega d_2)\right), \quad d_1 = \frac{\ln(F/K) + \frac{1}{2}v^2}{v}, \quad d_2 = d_1 - v,$$

and cap pricing under Black is expressed as a sum of these $\mathrm{Bl}$ terms.

**Black swaption formula (annuity times Black call on swap rate):** A payer swaption price at $t = 0$ is given as

$$PS^{\mathrm{Black}}(0, T_\alpha, T, \tau, N, K, \sigma_{\alpha,\beta}) = N \left(\sum_{i=\alpha+1}^{\beta} \tau_i P(0, T_i)\right) \mathrm{Bl}\left(K, S_{\alpha,\beta}(0), \sigma_{\alpha,\beta}\sqrt{T_\alpha}, +1\right).$$

**LMM (LFM) "market model" dynamics under its natural forward measure:** Under the $T_k$-forward(-adjusted) measure $\mathbb{Q}^k$, the forward rate $F_k(t)$ is modeled (in the scalar version shown) as driftless lognormal:

$$dF_k(t) = \sigma_k(t) \, F_k(t) \, dZ_k^k(t).$$

**Coupled drift under other measures:** Under a different $T_i$-forward measure $\mathbb{Q}^i$ (with $k < i$), the drift of $F_k$ picks up a sum over later forwards:

$$dF_k(t) = -\sigma_k(t) F_k(t) \sum_{j=k+1}^{i} \frac{\tau_j F_j(t) \sigma_j(t) \rho_{k,j}}{1 + \tau_j F_j(t)} \, dt + \sigma_k(t) F_k(t) \, dZ_k^i(t),$$

and a risk-neutral drift expression (under the money-market measure) is also given with additional terms.

**Swap rate and swap annuity:** For $\alpha < \beta$, the forward swap rate and swap annuity are

$$S_{\alpha,\beta}(t) = \frac{P(t, T_\alpha) - P(t, T_\beta)}{\sum_{i=\alpha+1}^{\beta} \tau_i P(t, T_i)}, \qquad C_{\alpha,\beta}(t) = \sum_{i=\alpha+1}^{\beta} \tau_i P(t, T_i).$$

**Swap measure / annuity numeraire:** The annuity $A_{n,m}(t) = \sum_{i=n+1}^{m} \delta_i P(t, T_i)$ qualifies as a numeraire and the corresponding forward swap rate $S_{n,m}(t)$ is a martingale under the annuity measure.

**Swap market model (LSM) motivation and payoff representation:** A payer swaption payoff at $T_\alpha$ can be written as

$$\mathrm{Payoff}_{T_\alpha} = (S_{\alpha,\beta}(T_\alpha) - K)^+ \, C_{\alpha,\beta}(T_\alpha),$$

and the "lognormal forward-swap model" assumes lognormal swap-rate dynamics under the swap measure.

**Calibration practice themes:**
- Calibration is treated as an optimization problem; e.g., minimizing a sum of squared differences between model prices and market prices is explicitly suggested in the swaption-grid context.
- Practical calibration often trades off precision vs regularity/smoothness and can use regularization terms; "ringing" in calibrated volatilities is explicitly discussed.
- Joint caps+swaptions calibration and weighting between them is discussed as a practical question.
- Correlation structures include full-rank and reduced-rank parameterizations, with examples like exponential correlation and Rebonato-style parametric forms.

**Multi-curve (OIS discounting vs projection curve):** Multiple yield curves and OIS discounting are discussed in the derivatives text and in the interest-rate modeling text.

### (B) Reasoned Inference

The appendix focuses on market models in the sense emphasized by the sources: choose a tradable market rate (a forward rate or a swap rate) and specify its volatility so that, under an appropriate numeraire measure, the rate has a tractable (often lognormal) form. No-arbitrage then fixes (or constrains) drifts under other measures, creating the "coupled drift" feature seen in the LMM formulas.

"Black compatibility" is a design constraint: match the model's canonical measure/dynamics to how the market quotes caplet and swaption vols, so that calibration can be expressed as "invert Black" and/or minimize errors in Black-implied vol/price.

### (C) Speculation (Clearly Labeled)

I will use the term "LIBOR-style forward" to mean the simply-compounded forward rate on a discrete tenor, even though post-crisis markets may use different reference rates. This is terminology alignment rather than a claim about any specific benchmark.

I will present a concrete calibration workflow that resembles common desk practice (objective + regularization + stability checks). Where the references do not specify a particular algorithm (e.g., LM vs gradient methods), I will explicitly label that as "I'm not sure."

---

## 0. Setup

### Conventions Used in This Appendix

- Time $t \geq 0$, filtered probability space $(\Omega, \mathcal{F}, (\mathcal{F}_t), \mathbb{P})$.
- Rates are simply compounded on a discrete tenor grid $\{T_0 < T_1 < \cdots < T_n\}$.
- Accrual fractions: $\delta_i = \tau(T_i, T_{i+1})$ (year fraction for $[T_i, T_{i+1}]$).
- $P(t, T)$: time-$t$ price of a default-free zero-coupon bond maturing at $T$.
- Money-market account: $B(t) = B_0 \exp\left(\int_0^t r_s \, ds\right)$.

**Measures / numeraires:**
- Risk-neutral (money-market) measure $\mathbb{Q}$ with numeraire $B(t)$.
- $T$-forward measure $\mathbb{Q}^T$ with numeraire $P(t, T)$. Pricing: $\Pi_t = P(t, T) \mathbb{E}^T[H_T \mid \mathcal{F}_t]$.
- Discrete $T_i$-forward measure $\mathbb{Q}^i$ (shorthand for $\mathbb{Q}^{T_i}$).
- Swap (annuity) measure $\mathbb{Q}^{\alpha,\beta}$ with numeraire $C_{\alpha,\beta}(t) = \sum_{k=\alpha+1}^{\beta} \delta_k P(t, T_k)$.

### Notation Glossary

| Category | Symbol | Definition |
|----------|--------|------------|
| **Tenor** | $T_i$ | Tenor dates |
| | $\delta_i$ | Accrual fraction for $[T_i, T_{i+1}]$ |
| **Bonds and discounting** | $P(t, T)$ | Zero-coupon bond price |
| | $r_t$ | Short rate |
| | $B(t)$ | Money-market account |
| | $D(t, T)$ | Discount factor: $\exp\left(-\int_t^T r_s \, ds\right) = B(t)/B(T)$ |
| **Forward rates** | $F_i(t)$ | $F(t; T_i, T_{i+1}) = \frac{1}{\delta_i}\left(\frac{P(t, T_i)}{P(t, T_{i+1})} - 1\right)$ |
| **Swap rates** | $C_{\alpha,\beta}(t)$ | Swap annuity: $\sum_{k=\alpha+1}^{\beta} \delta_k P(t, T_k)$ |
| | $S_{\alpha,\beta}(t)$ | Forward swap rate: $\frac{P(t, T_\alpha) - P(t, T_\beta)}{C_{\alpha,\beta}(t)}$ |
| **Black building block** | $\Phi(\cdot)$ | Standard normal CDF |
| | $\mathrm{Bl}(K, F, v, \omega)$ | $\omega(F\Phi(\omega d_1) - K\Phi(\omega d_2))$ |
| | $d_1, d_2$ | $d_1 = \frac{\ln(F/K) + \frac{1}{2}v^2}{v}$, $d_2 = d_1 - v$ |
| **Volatilities** | $\sigma_i(t)$ | Instantaneous vol of forward rate $F_i$ |
| | $\sigma_{\alpha,\beta}$ | Swaption Black implied vol for swap rate $S_{\alpha,\beta}$ |
| **Correlation** | $\rho_{i,j}$ | Instantaneous correlation between Brownians for $F_i$ and $F_j$ |

---

## 1. Core Concepts (Definitions First)

### 1.1 Tenor Dates $\{T_0 < T_1 < \cdots < T_n\}$ and Accruals $\delta_i$

**Formal definition:** A discrete set of dates $\{T_i\}$ with associated accrual fractions $\delta_i = \tau(T_i, T_{i+1})$.

**Intuition:** These are the standard coupon/reset/payment times for money-market FRAs and swap legs.

**Trading/risk practice:** Most quoted caplets/swaptions reference such a grid (e.g., semiannual fixed legs, quarterly floating legs). The tenor determines which forwards/swaps you model and hedge.

### 1.2 Zero-Coupon Bonds $P(t, T)$

**Formal definition:** $P(t, T)$ is the time-$t$ price of \$1 paid at maturity $T$.

**Intuition:** Bonds are the primitives from which forwards and swaps are built.

**Trading/risk practice:** Curves are built to produce $P(0, T)$. A "market model" typically assumes the initial curve is given and focuses on the volatility of rates.

### 1.3 Forward Rate / "LIBOR-Style" Simply-Compounded Forward $F_i(t)$

**Formal definition:**

$$F_i(t) = \frac{1}{\delta_i}\left(\frac{P(t, T_i)}{P(t, T_{i+1})} - 1\right), \quad \text{so} \quad 1 + \delta_i F_i(t) = \frac{P(t, T_i)}{P(t, T_{i+1})}.$$

**Intuition:** $F_i(t)$ is the fair (no-arbitrage) simple rate for borrowing/lending over $[T_i, T_{i+1}]$, as implied by bonds.

**Trading/risk practice:**
- Caps/floors are portfolios of options on these forwards (caplets/floorlets).
- In LMM, these $F_i$ are the modeled state variables, so caplet pricing is designed to line up with Black quoting.

### 1.4 Numeraires and Measures (Money-Market, $T$-Forward, Swap Measure)

**Formal definitions:**
- **Money-market numeraire** $B(t) = B_0 e^{\int_0^t r_s \, ds}$.
- **$T$-bond numeraire** $P(t, T)$: induces the $T$-forward measure $\mathbb{Q}^T$ and pricing formula

$$\Pi_t = P(t, T) \, \mathbb{E}^T[H_T \mid \mathcal{F}_t].$$

- **Swap annuity numeraire** $C_{\alpha,\beta}(t) = \sum_{k=\alpha+1}^{\beta} \delta_k P(t, T_k)$ induces the swap/annuity measure $\mathbb{Q}^{\alpha,\beta}$, under which the swap rate is a martingale.

**Intuition:** A numeraire is "the unit of account" for measuring value. Under the measure associated to a chosen numeraire, any tradable price divided by the numeraire is a martingale.

**Trading/risk practice:** Picking the right measure is the heart of market models:
- Caplets are naturally priced under the relevant bond-forward measure.
- Swaptions are naturally priced under the relevant swap (annuity) measure.

### 1.5 Caplet, Cap, Swaption Payoffs; Black(-Style) Compatibility

#### Caplet

**Formal payoff (payer caplet):** set at $T_i$ and paid at $T_{i+1}$ (equivalently set at $T_{i-1}$ and paid at $T_i$ depending on indexing). In the references' caplet convention: payoff set at $T_{i-1}$ and paid at $T_i$, and the time-0 price is expressed under the $T_i$-forward measure as

$$\mathrm{Cpl}(0, T_{i-1}, T_i, \tau_i, K) = P(0, T_i) \tau_i \, \mathbb{E}^i\left[(F_i(T_{i-1}) - K)^+\right].$$

(cap price shown as sum; caplet decomposition uses this structure in the text)

#### Cap

**Formal definition:** a cap is a sum of caplets across payment dates (strip decomposition).

#### Swaption

**Formal payoff (payer swaption)** at expiry $T_\alpha$:

$$\mathrm{Payoff}_{T_\alpha} = (S_{\alpha,\beta}(T_\alpha) - K)^+ \, C_{\alpha,\beta}(T_\alpha).$$

#### Black Compatibility

**Formal meaning (within this appendix):** choose a model so that the payoff expectation reduces to the Black function $\mathrm{Bl}$ with lognormal-underlying assumptions under an appropriate measure.

**Why traders care:** caps and swaptions are commonly quoted in terms of Black implied volatilities, so calibration is naturally framed as "match Black vols/prices."

### 1.6 What is a "Market Model"?

**Formal definition (in spirit of the sources):** a model that directly specifies the stochastic dynamics of market-quoted rates (forward rates or swap rates) on a discrete tenor, using no-arbitrage via measure/numeraire consistency to imply drifts under other measures.

**Intuition:** Instead of modeling a short rate and deriving forwards/swaps, we model the forwards/swaps directly—because that's what the option market quotes.

**Trading/risk practice:**
- Enables direct calibration to caplet and/or swaption vol surfaces.
- Facilitates "vega bucket" risk reporting aligned with market quotes.

---

## 2. LMM (BGM) Essentials: Modeling Forward Rates

### 2.1 Modeling Target: Forward Rates $F_i(t)$ on a Discrete Tenor

We model the set $\{F_i(t)\}_{i=0}^{n-1}$ defined from bond prices.

### 2.2 Key No-Arbitrage Idea: Choose the Measure Where a Forward is Simplest

Under the $T_{i+1}$-forward measure $\mathbb{Q}^{i+1}$ (numeraire $P(t, T_{i+1})$), the forward $F_i(t)$ is a martingale. In the lognormal-forward model, it is postulated to be lognormal with driftless dynamics (in scalar form):

$$dF_i(t) = \sigma_i(t) F_i(t) \, dZ_i^{i+1}(t).$$

**Why this produces Black caplet pricing:** if $F_i$ is lognormal under its forward measure, then $\mathbb{E}^{i+1}[(F_i(T_i) - K)^+]$ is exactly a Black expectation (see Section 7 derivation).

### 2.3 SDE Form Under Its Natural Forward Measure (What is Lognormal)

**Scalar (one-factor) form** shown in the text:

$$dF_i(t) = \sigma_i(t) \, F_i(t) \, dZ_i^{i+1}(t) \quad \Rightarrow \quad \ln F_i(t) \text{ is normal under } \mathbb{Q}^{i+1}.$$

**What is not claimed:**
- This does not mean all $F_j$ are simultaneously lognormal under a single common measure.
- Under $\mathbb{Q}$ (money-market measure), each $F_i$ has a non-zero, state-dependent drift (coupled drift).

### 2.4 Drift Under Other Measures (Coupled Drift Feature)

#### 2.4.1 Drift Under Another Forward Measure $\mathbb{Q}^i$

The sources derive a general drift adjustment when changing from $\mathbb{Q}^k$ to $\mathbb{Q}^i$ (with $k < i$). The resulting dynamics (scalar notation) are:

$$\boxed{dF_k(t) = -\sigma_k(t) F_k(t) \sum_{j=k+1}^{i} \frac{\tau_j F_j(t) \sigma_j(t) \rho_{k,j}}{1 + \tau_j F_j(t)} \, dt + \sigma_k(t) F_k(t) \, dZ_k^i(t).}$$

**Interpretation:** under $\mathbb{Q}^i$, the drift of an earlier forward $F_k$ depends on the later forwards $\{F_{k+1}, \ldots, F_i\}$. This is the classic "triangular/coupled drift" structure of LMM.

#### 2.4.2 Drift Under the Risk-Neutral (Money-Market) Measure $\mathbb{Q}$

The risk-neutral drift expression in the source includes:
- a sum similar in spirit to the above coupling term, plus
- an additional term involving the instantaneous forward rate $f(t, T_{\beta(t)})$ over the current accrual interval.

I'm not sure how much of the risk-neutral drift expression you want to emphasize in this appendix, because it depends on the exact definition of $\beta(t)$ (the index locating $t$ inside the tenor) and on the discrete-money-market construction used in the source. To be certain, we would need to lock down:
- whether we are using the continuous money-market account $B(t)$ or the discretely compounded "spot LIBOR" bank account $B_d(t)$,
- the precise definition of $\beta(t)$ and the interpolation conventions used between tenor dates (the text discusses such details elsewhere).

---

## 3. Swap Market Model Essentials: Modeling Swap Rates

### 3.1 Modeling Target: Swap Rate $S_{\alpha,\beta}(t)$ for a Swap Spanning $[T_\alpha, T_\beta]$

**Definition (source-backed):**

$$S_{\alpha,\beta}(t) = \frac{P(t, T_\alpha) - P(t, T_\beta)}{C_{\alpha,\beta}(t)}, \qquad C_{\alpha,\beta}(t) = \sum_{k=\alpha+1}^{\beta} \tau_k P(t, T_k).$$

**Intuition:**
- $C_{\alpha,\beta}(t)$ is the PV of a unit fixed leg (PVBP / swap annuity).
- The numerator $P(t, T_\alpha) - P(t, T_\beta)$ is the PV of the floating leg of a forward-starting par swap.

### 3.2 Key Motivation: Swap Measure Lognormality $\Rightarrow$ Black Swaption Pricing

The swaption payoff at $T_\alpha$ can be written as

$$\mathrm{Payoff}_{T_\alpha} = (S_{\alpha,\beta}(T_\alpha) - K)^+ \, C_{\alpha,\beta}(T_\alpha).$$

Under the swap (annuity) measure associated with numeraire $C_{\alpha,\beta}$, the forward swap rate is a martingale (and the annuity is a valid numeraire).

The **lognormal forward-swap model** assumes $S_{\alpha,\beta}$ follows a lognormal diffusion under $\mathbb{Q}^{\alpha,\beta}$ (the text states this as a "lognormal dynamics" assumption).

Then the Black payer swaption formula applies:

$$\boxed{PS^{\mathrm{Black}}(0, \ldots) = N \left(\sum_{i=\alpha+1}^{\beta} \tau_i P(0, T_i)\right) \mathrm{Bl}\left(K, S_{\alpha,\beta}(0), \sigma_{\alpha,\beta}\sqrt{T_\alpha}, +1\right).}$$

### 3.3 Why is the Swap Annuity a Numeraire?

**Source-backed statement:** $A_{n,m}(t) = \sum_{i=n+1}^{m} \delta_i P(t, T_i)$ qualifies as a numeraire, and the corresponding forward swap rate is a martingale under the associated annuity measure.

**Interpretation (inference):** since $A_{n,m}(t)$ is a strictly positive tradable value process (portfolio of bonds with positive weights), it can serve as a numeraire.

### 3.4 Practical Comparison with LMM

#### Natural Calibration Fit

| Model | Natural Fit |
|-------|-------------|
| **LMM/LFM** | Caplets are directly Black(-lognormal) under each $T_{i+1}$-forward measure because $F_i$ is modeled as driftless lognormal under that measure |
| **Swap market model/LSM** | Swaptions are directly Black(-lognormal) under each swap measure because $S_{\alpha,\beta}$ is modeled as lognormal under that measure |

#### Consistency Across the Full Vol Surface

The sources emphasize an **incompatibility:** swap rates are not exactly lognormal under the LMM, and forward rates are not exactly lognormal under the LSM. However, the text also notes that the practical difference can be small in many cases (swap-rate distribution under LMM can be close to lognormal under the swap measure).

---

## 4. Connection to HJM and Drift Restrictions (Bridge Section)

### 4.1 HJM Drift Restriction (Continuous-Tenor Baseline)

In HJM, instantaneous forward rates $f(t, T)$ have dynamics under the $T$-forward measure:

$$df(t, T) = \sigma_f(t, T)^\top \, dW^T(t),$$

and under the risk-neutral measure:

$$df(t, T) = \sigma_f(t, T)^\top \left(\int_t^T \sigma_f(t, u) \, du\right) dt + \sigma_f(t, T)^\top dW(t).$$

**Interpretation:**
- Choose the volatility structure $\sigma_f(t, T)$, then no-arbitrage fixes the drift (risk-neutral drift is determined by volatility).
- Under the $T$-forward measure, the drift disappears (a martingale property).

### 4.2 How LMM Connects to HJM (Conceptual Map)

**Conceptual linkage (inference consistent with the above):**
- LMM replaces the continuum of maturities with a discrete tenor and replaces $f(t, T)$ with simply-compounded forwards $F_i(t)$.
- Instead of specifying $\sigma_f(t, T)$, LMM specifies $\sigma_i(t)$ for each $F_i$ (plus correlation across $i$).
- The drift under non-canonical measures becomes implied and coupled, analogous to HJM drift restriction. This is explicit in the LMM drift formulas under measure changes.

If you want a fully explicit "mapping" from a particular HJM volatility $\sigma_f(t, T)$ to an LMM $\sigma_i(t)$, I'm not sure the exact mapping is uniquely defined in the provided excerpts (it depends on how you discretize and how you translate instantaneous to simply-compounded forwards). To be certain, we would need the specific sections that define the discretization scheme and interpolation rule between tenor dates (the table of contents indicates related sections on drift interpolation/bridging).

### 4.3 Tradeoff Emphasized by Market Models

**Design choice:** Pick which tradable rate is "nice" (lognormal) under a convenient measure:
- **LMM:** $F_i$ is nice under $\mathbb{Q}^{i+1}$.
- **Swap market model:** $S_{\alpha,\beta}$ is nice under $\mathbb{Q}^{\alpha,\beta}$.

**No-arbitrage implication:** Once you specify vol/correlation, the drift under other measures is determined (and generally complicated).

---

## 5. Calibration Logic (The Centerpiece)

This section is intentionally practitioner-oriented: what you fit, why you fit it, and how the measure choice makes calibration feasible.

### A) What is Calibrated to What (Market Instruments)

#### A.1 Caps/Floors (Caplets) → Forward-Rate Vol

**Market quotes:** caps/floors are commonly quoted via Black implied volatilities (cap vol surface). Black cap price is represented as a sum over coupon periods using the $\mathrm{Bl}$ function.

**LMM calibration idea:**
- Because each $F_i$ is lognormal under its own forward measure, caplets are "directly Black."
- Practically, you can infer caplet volatilities from cap quotes (bootstrapping) and then fit $\sigma_i(t)$ (or integrated variances) so model caplet prices match those implied caplet vols.

#### A.2 Swaptions → Swap-Rate Vol (and Correlation/Structure)

**Market quotes:** swaptions are quoted by Black implied volatilities; a swaption price is "annuity × Black call on swap rate."

**Swap market model calibration idea:** Since swap rates are assumed lognormal under swap measures, swaptions can be fit "directly."

#### A.3 Reconciling Caps and Swaptions (Joint Calibration)

The references explicitly raise the question of calibrating to caps and swaptions simultaneously, noting that practitioners sometimes argue there is a persistent "basis" between the two markets and advocate calibrating to one set based on whether the target product is cap-like or swaption-like. The text then recommends considering both markets with a tunable weighting mechanism.

**Practical takeaway:** your calibration set is both a pricing and hedging choice.

### B) Parameterizations (Only What Sources Cover)

#### B.1 Volatility Parameterizations

The sources describe:

- **General piecewise constant (GPC)** instantaneous-volatility structures (used in Monte Carlo tests and calibration discussions).

- **Parametric instantaneous volatility structure** example ("Formulation 7"):

$$\sigma_i(t) = \Phi_i \left([a(T_{i-1} - t) + d] e^{-b(T_{i-1} - t)} + c\right),$$

with parameters $a, b, c, d$ and scaling factors $\Phi_i$.

**Interpretation (inference):**
- $\Phi_i$ typically ties the structure to caplet vol levels (the source notes $\Phi$'s being determined through caplet volatilities via a referenced equation).
- $(a, b, c, d)$ shape the term structure of instantaneous vol in maturity.

**Smile extensions:** The sources list several ways to extend beyond flat-smile lognormal market models, including displaced diffusion, CEV-like dynamics, local volatility, stochastic volatility, and SABR-style approaches.

#### B.2 Correlation Structure Across Tenor/Factors

The sources discuss:

- **Full-rank parameterizations,** e.g. constant correlation, and an exponential form $\rho_{ij} = \exp(-\beta |T_i - T_j|)$ with $\beta > 0$.

- **Reduced-rank formulations** (Rebonato's angles, eigenvalue zeroing) for factor reduction and simulation efficiency.

- A **Rebonato-like 3-parameter form:** $\rho_{i,j} = \rho_\infty + (1 - \rho_\infty) \exp(-\beta |T_i - T_j|^\gamma)$ is also listed.

**Interpretation (inference):** calibration to swaptions often informs correlation/factor structure, because swap rates are weighted combinations of many forwards.

### C) Calibration Workflow Steps (Concrete)

Below is a concrete workflow consistent with the sources' discussion. Where an exact algorithmic detail is not pinned down, I label it.

1. **Fix the initial curve** $\{P(0, T_i)\}$ and implied forwards $\{F_i(0)\}$
   - Treat curve construction as "given" (bootstrapped from deposits/futures/swaps, etc.) and compute forwards from bond ratios.

2. **Choose model family**
   - Caps-first: LMM/LFM (BGM).
   - Swaptions-first: swap market model / LSM.

3. **Choose parameterization for vol/correlation**
   - For LMM:
     - Choose $\sigma_i(t)$ either piecewise constant (GPC) or parametric (e.g., Formulation 7).
     - Choose a correlation structure $\rho_{ij}$ (full-rank exponential, or reduced-rank).
   - For swap market model:
     - Choose swap-rate vol parameterization across expiries/tenors; often directly implied from swaption vol surface (Black vols).

4. **Fit to market vol quotes** (objective function + constraints/regularization)
   - **Objective function example** (explicit in source): minimize sum of squared differences between model and market swaption prices (tables).
   - **Regularization / smoothness:** The sources emphasize adding regularity terms to avoid unstable "ringing" patterns in calibrated vol surfaces.
   - **Weighting caps vs swaptions:** The sources recommend considering both markets but allow unequal weights in the calibration objective.
   - **Algorithm choice:** The sources discuss row-by-row calibration and "full-blown" calibration, plus mention bootstrap/cascade ideas and their pitfalls. I'm not sure which numerical optimizer (LM, SQP, gradient-based) you want assumed; the sources focus more on problem structure (splitting, regularization, robustness) than on one canonical optimizer in the excerpted lines.

5. **Validate**
   - **Repricing checks:** reprice the caps and swaptions used in calibration and confirm errors are within tolerance.
   - **Stability checks:** bump a few market vols and see if calibrated $\sigma_i(t)$ moves smoothly (ringing control). This is directly motivated by the "ringing" discussion.
   - **Market sanity checks:** the sources warn about swaption matrix misalignment (stale quotes) which can cause unstable/impossible implied parameters.

### D) Why Black(-Style) Compatibility Matters

**In LMM, caplets are "built to be Black":**
- Under the $T_{i+1}$-forward measure, $F_i$ is modeled driftless lognormal.
- Hence caplet prices are Black expectations under that measure (Section 7 derivation), aligning directly with caplet implied vol quotes.

**In the swap market model, swaptions are "built to be Black":**
- Under the swap annuity measure, swap rate is modeled lognormal and swaption payoff is annuity times option on swap rate.

**Design message:** This is not a theorem about real-world dynamics; it is a quoting-compatibility and calibration-efficiency choice.

---

## 6. Simulation and Pricing Map (Implementation-Level but No Code Requirement)

### 6.1 How to Simulate LMM (Measure Choice and Drift Complexity)

The sources discuss simulation/pricing under a spot-LIBOR measure $\mathbb{Q}^d$ with a discrete bank account numeraire $B_d(t)$, and express swaption pricing as an expectation under $\mathbb{Q}^d$:

$$V_0 = B_d(0) \, \mathbb{E}^d\left[\frac{C_{\alpha,\beta}(T_\alpha) \, (S_{\alpha,\beta}(T_\alpha) - K)^+}{B_d(T_\alpha)}\right].$$

**Consequences:**
- A single common measure for joint simulation (e.g., $\mathbb{Q}^d$ or a terminal forward measure) avoids changing measure instrument-by-instrument, but introduces coupled drifts (need to recompute drift each step from current forward rates). The coupled drift phenomenon is explicit in the measure-change formulas.

**Discretization:**
- Discretize time on a grid; evolve all forwards $\{F_i\}$ forward using an Euler/log-Euler scheme. The text notes the need for discretized LMM dynamics and discusses Euler-type approximations and alternative schemes.

I'm not sure which measure your desk prefers for LMM simulation (terminal vs spot) because the excerpted lines explicitly show $\mathbb{Q}^d$ and discuss "$T_\gamma$-forward-adjusted" measures in the drift-change derivations, but choosing the default simulation measure is an implementation convention.

### 6.2 How to Price with the Models

**Caps/caplets:**
- Under LMM, caplets are Black(-lognormal) under their forward measures by construction; caps are sums of caplets.

**Swaptions:**
- Under the swap market model, swaptions are priced directly by Black with annuity numeraire.
- Under LMM, swaptions can be priced by:
  - Monte Carlo simulation (explicitly discussed), or
  - analytical approximations (the sources mention Rebonato/Hull-White formulas in the calibration discussion).

**More complex payoffs:**
- Use Monte Carlo under a common simulation measure; discount with the chosen numeraire; validate with plain-vanilla repricing.

### 6.3 Risk Outputs (Preview)

**Vega buckets:**
- Caplet vegas map to the caplet vol surface (or to the instantaneous vol parameters $\sigma_i(t)$).
- Swaption vegas map to swaption vol matrix.

**Correlation sensitivity:**
- In LMM, correlation parameterization influences swaption/exotic prices and hedge ratios; the sources discuss correlation parameterizations and calibration considerations.

---

## 7. Math and Derivations (Step-by-Step)

### 7.1 Forward Rate from Bond Prices on a Tenor Grid

Start from the no-arbitrage relationship between discount factors over $[T_i, T_{i+1}]$ and a simply-compounded forward rate $F_i(t)$:

$$1 + \delta_i F_i(t) = \frac{P(t, T_i)}{P(t, T_{i+1})}.$$

Solve for $F_i(t)$:

$$\boxed{F_i(t) = \frac{1}{\delta_i}\left(\frac{P(t, T_i)}{P(t, T_{i+1})} - 1\right).}$$

**Unit check:** $P(\cdot, \cdot)$ is dimensionless "price per \$1", so $P(t, T_i)/P(t, T_{i+1}) - 1$ is dimensionless; dividing by $\delta_i$ (years) gives rate units (1/year).

### 7.2 Under $T$-Forward Measure: Why Lognormal $F_i$ $\Rightarrow$ Black Caplet Formula

Consider a caplet whose payoff is fixed at $T_i$ and paid at $T_{i+1}$. Using a $T_{i+1}$-forward measure (numeraire $P(t, T_{i+1})$) is natural because the payoff is at $T_{i+1}$.

**Pricing under a bond numeraire:** for a payoff $H_{T_{i+1}}$,

$$\Pi_0 = P(0, T_{i+1}) \, \mathbb{E}^{T_{i+1}}[H_{T_{i+1}}].$$

(general forward-measure pricing)

For a payer caplet, $H_{T_{i+1}} = \delta_i (F_i(T_i) - K)^+$.

Hence

$$\mathrm{Cpl}_0 = P(0, T_{i+1}) \, \delta_i \, \mathbb{E}^{i+1}\left[(F_i(T_i) - K)^+\right].$$

In LMM, under $\mathbb{Q}^{i+1}$,

$$dF_i(t) = \sigma_i(t) F_i(t) dZ_i^{i+1}(t),$$

so $F_i(T_i)$ is lognormal under $\mathbb{Q}^{i+1}$.

Therefore the expectation is the Black call on a lognormal forward:

$$\mathbb{E}^{i+1}\left[(F_i(T_i) - K)^+\right] = \mathrm{Bl}\left(K, F_i(0), v_i, +1\right),$$

where $v_i$ is the total (integrated) volatility over $[0, T_i]$ (for constant $\sigma$, $v_i = \sigma\sqrt{T_i}$). The $\mathrm{Bl}$ function definition is given in the sources.

**Limiting case check:** If $v_i \to 0$, then the distribution collapses and the option value tends to discounted intrinsic: $P(0, T_{i+1}) \delta_i \max(F_i(0) - K, 0)$ (see Worked Example 10).

### 7.3 Swap Rate Definition and Swap Annuity

**Swap rate and annuity:**

$$\boxed{C_{\alpha,\beta}(t) = \sum_{k=\alpha+1}^{\beta} \delta_k P(t, T_k), \qquad S_{\alpha,\beta}(t) = \frac{P(t, T_\alpha) - P(t, T_\beta)}{C_{\alpha,\beta}(t)}.}$$

**Unit check:** $C_{\alpha,\beta}(t)$ has units of "years" (because it is a sum of $\delta_k \times$ dimensionless bond prices). Thus $K \cdot C$ is dimensionless PV, matching $P(t, T_\alpha) - P(t, T_\beta)$.

### 7.4 Under Swap Measure: Why Lognormal $S_{\alpha,\beta}$ $\Rightarrow$ Black Swaption Formula

**Swaption payoff at $T_\alpha$:**

$$\mathrm{Payoff}_{T_\alpha} = (S_{\alpha,\beta}(T_\alpha) - K)^+ \, C_{\alpha,\beta}(T_\alpha).$$

Use the annuity numeraire $C_{\alpha,\beta}(t)$. The sources state that the annuity qualifies as a numeraire and that the swap rate is a martingale under the associated annuity measure.

Therefore pricing becomes

$$\mathrm{Swpt}_0 = C_{\alpha,\beta}(0) \, \mathbb{E}^{\alpha,\beta}\left[(S_{\alpha,\beta}(T_\alpha) - K)^+\right].$$

If $S_{\alpha,\beta}$ is assumed lognormal under $\mathbb{Q}^{\alpha,\beta}$, then

$$\mathbb{E}^{\alpha,\beta}\left[(S_{\alpha,\beta}(T_\alpha) - K)^+\right] = \mathrm{Bl}\left(K, S_{\alpha,\beta}(0), \sigma_{\alpha,\beta}\sqrt{T_\alpha}, +1\right),$$

yielding the Black swaption formula shown in the sources:

$$\boxed{PS^{\mathrm{Black}} = N \left(\sum_{i=\alpha+1}^{\beta} \tau_i P(0, T_i)\right) \mathrm{Bl}\left(K, S_{\alpha,\beta}(0), \sigma_{\alpha,\beta}\sqrt{T_\alpha}, +1\right).}$$

### 7.5 Drift Coupling Under a Common Measure (As Far As Sources Allow)

Under a different forward measure $\mathbb{Q}^i$, $F_k$ picks up drift that depends on later forwards:

$$\mu_k^{(i)}(t) = -\sigma_k(t) F_k(t) \sum_{j=k+1}^{i} \frac{\tau_j F_j(t) \sigma_j(t) \rho_{k,j}}{1 + \tau_j F_j(t)}.$$

**Intuition:** The change-of-numeraire shifts Brownian motions by terms involving ratios of tradable assets; on a tenor grid, that produces sums over later accrual periods. The source shows this explicitly via the Brownian shift relation and the product form of bond ratios.

**Risk-neutral drift:** The source provides a risk-neutral (money-market) drift formula with additional terms.

I'm not sure whether to reproduce it in full detail here without also reproducing the full discrete bank-account construction and the definition of $\beta(t)$. What you would need to lock this down is: the exact bank-account choice ($B(t)$ vs $B_d(t)$) and interpolation rules between tenor dates.

---

## 8. Worked Examples (At Least 10 Numeric Examples; Fully Numeric)

**Numerical convention for normal CDF:** In these toy examples, $\Phi(\cdot)$ is evaluated using standard normal table values (rounded). Key values used:

| $x$ | $\Phi(x)$ |
|-----|-----------|
| 0.1 | 0.5398 |
| −0.1 | 0.4602 |
| 0.18 | ≈0.5714 |
| 0.54 | ≈0.7054 |
| 0.74 | ≈0.7704 |
| 0.80 | ≈0.7881 |
| 1.00 | 0.8413 |

---

### Example 1: Build a Small Tenor Grid and Compute Forward Rates from Discount Factors

**Tenor:** $T_0 = 0$, $T_1 = 0.5$, $T_2 = 1.0$, $T_3 = 1.5$.

**Accruals:** $\delta_0 = \delta_1 = \delta_2 = 0.5$.

**Given discount factors:**

| Maturity | $P(0, T)$ |
|----------|-----------|
| 0 | 1.0000 |
| 0.5 | 0.9900 |
| 1.0 | 0.9750 |
| 1.5 | 0.9550 |

**Forward rates:**

$$F_i(0) = \frac{1}{\delta_i}\left(\frac{P(0, T_i)}{P(0, T_{i+1})} - 1\right).$$

**$F_0(0)$ over $[0, 0.5]$:**
$$F_0(0) = \frac{1}{0.5}\left(\frac{1.0000}{0.9900} - 1\right) = 2 \times (1.010101 - 1) = 2 \times 0.010101 = 0.020202 \approx 2.0202\%.$$

**$F_1(0)$ over $[0.5, 1.0]$:**
$$F_1(0) = 2\left(\frac{0.9900}{0.9750} - 1\right) = 2 \times (1.0153846 - 1) = 2 \times 0.0153846 = 0.030769 \approx 3.0769\%.$$

**$F_2(0)$ over $[1.0, 1.5]$:**
$$F_2(0) = 2\left(\frac{0.9750}{0.9550} - 1\right) = 2 \times (1.0209424 - 1) = 2 \times 0.0209424 = 0.041885 \approx 4.1885\%.$$

---

### Example 2: Caplet Black Pricing (Toy)

We price a caplet on period $[T_1, T_2] = [0.5, 1.0]$, set at $T_1 = 0.5$, paid at $T_2 = 1.0$.

**Inputs:**
- Forward: $F = F_1(0) = 0.030769$.
- Strike: $K = F$ (ATM for algebraic simplicity).
- Accrual: $\delta = 0.5$.
- Pay date discount factor: $P(0, 1.0) = 0.975$.
- Expiry: $T = 0.5$.
- Choose $\sigma$ such that $v = \sigma\sqrt{T} = 0.2$. Since $\sqrt{0.5} = 0.7071$, take $\sigma = 0.2/0.7071 \approx 0.28284$.

**Black quantities (ATM implies $\ln(F/K) = 0$):**

$$d_1 = \frac{0 + \frac{1}{2}v^2}{v} = \frac{0.5 \times 0.04}{0.2} = 0.1, \qquad d_2 = d_1 - v = 0.1 - 0.2 = -0.1.$$

$\Phi(d_1) = 0.5398$, $\Phi(d_2) = 0.4602$.

**Black call on forward:**

$$\mathrm{Bl}(K, F, v, +1) = F\Phi(d_1) - K\Phi(d_2) = F(0.5398) - F(0.4602) = F(0.0796).$$

**Caplet PV:**

$$\mathrm{Cpl}_0 = P(0, 1.0) \cdot \delta \cdot \mathrm{Bl} = 0.975 \times 0.5 \times (0.030769 \times 0.0796).$$

**Compute:**
- $0.030769 \times 0.0796 \approx 0.00244923$.
- Multiply by $0.975 \times 0.5 = 0.4875$: $0.00244923 \times 0.4875 \approx 0.0011940$.

$$\boxed{\mathrm{Cpl}_0 \approx 0.001194 \text{ per unit notional.}}$$

(Black $\mathrm{Bl}$ definition is source-backed.)

---

### Example 3: Show How a Cap is a Sum of Caplets (Numeric Sum of 2 Caplets)

Consider a cap covering $[0.5, 1.5]$ with strike $K = 0.035$ and two caplets:
- **Caplet A** on $[0.5, 1.0]$: use $P(0, 1.0) = 0.975$, $\delta = 0.5$, $F_1(0) = 0.030769$, expiry $T = 0.5$, choose $v = 0.2 \Rightarrow \sigma = 0.28284$.
- **Caplet B** on $[1.0, 1.5]$: use $P(0, 1.5) = 0.955$, $\delta = 0.5$, $F_2(0) = 0.041885$, expiry $T = 1.0$, choose $v = 0.2 \Rightarrow \sigma = 0.2$.

Cap price is the sum of caplet prices (strip decomposition).

**Caplet A ($F = 0.030769 < K = 0.035$, $v = 0.2$):**

$$\ln(F/K) = \ln(0.030769/0.035) = \ln(0.87912) \approx -0.1288.$$

$$d_1 = \frac{-0.1288 + 0.02}{0.2} = \frac{-0.1088}{0.2} = -0.544.$$

$$d_2 = d_1 - 0.2 = -0.744.$$

Approx CDFs: $\Phi(-0.544) \approx 1 - \Phi(0.544) \approx 1 - 0.7054 = 0.2946$.
$\Phi(-0.744) \approx 1 - \Phi(0.744) \approx 1 - 0.7704 = 0.2296$.

Black call:
$$\mathrm{Bl} \approx 0.030769(0.2946) - 0.035(0.2296) = 0.00906 - 0.00804 \approx 0.00102.$$

PV:
$$\mathrm{CplA}_0 \approx 0.975 \times 0.5 \times 0.00102 = 0.4875 \times 0.00102 \approx 0.00050.$$

**Caplet B ($F = 0.041885 > K = 0.035$, $v = 0.2$):**

$$\ln(F/K) = \ln(1.1967) \approx 0.1795.$$

$$d_1 = (0.1795 + 0.02)/0.2 = 0.9975 \approx 1.00.$$

$$d_2 = 0.7975 \approx 0.80.$$

$\Phi(d_1) \approx \Phi(1.00) = 0.8413$, $\Phi(d_2) \approx \Phi(0.80) = 0.7881$.

$$\mathrm{Bl} \approx 0.041885(0.8413) - 0.035(0.7881) = 0.03524 - 0.02758 = 0.00766.$$

PV:
$$\mathrm{CplB}_0 \approx 0.955 \times 0.5 \times 0.00766 = 0.4775 \times 0.00766 \approx 0.00366.$$

**Cap price (sum):**

$$\boxed{\mathrm{Cap}_0 \approx 0.00050 + 0.00366 = 0.00416.}$$

---

### Example 4: Swaption Black Pricing (Toy)

Payer swaption expiring at $T_\alpha = 0.5$ into a swap from $0.5$ to $1.5$ with fixed payments at $T_2 = 1.0$, $T_3 = 1.5$.

**Discount factors:** $P(0, 0.5) = 0.99$, $P(0, 1.0) = 0.975$, $P(0, 1.5) = 0.955$.

**Annuity:**
$$A = C_{1,3}(0) = 0.5 \cdot P(0, 1.0) + 0.5 \cdot P(0, 1.5) = 0.5(0.975) + 0.5(0.955) = 0.4875 + 0.4775 = 0.9650.$$

**Swap rate:**
$$S = \frac{P(0, 0.5) - P(0, 1.5)}{A} = \frac{0.99 - 0.955}{0.965} = \frac{0.035}{0.965} \approx 0.036269.$$

**Strike:** $K = S$ (ATM).

**Choose** $v = \sigma\sqrt{T_\alpha} = 0.2 \Rightarrow \sigma = 0.28284$ since $\sqrt{0.5} = 0.7071$.

**Black terms (ATM):**

$$d_1 = 0.1, \quad d_2 = -0.1, \quad \Phi(d_1) - \Phi(d_2) = 0.5398 - 0.4602 = 0.0796.$$

$$\mathrm{Bl} = S \times 0.0796 \approx 0.036269 \times 0.0796 \approx 0.002887.$$

**Swaption PV:**

$$\mathrm{Swpt}_0 = A \times \mathrm{Bl} \approx 0.9650 \times 0.002887 \approx 0.002786.$$

This matches the "annuity times Black call on swap rate" structure in the source formula.

$$\boxed{\mathrm{Swpt}_0 \approx 0.002786 \text{ per unit notional.}}$$

---

### Example 5: Compute Swap Annuity and Verify Units (and a Par-Swap Identity)

Using the same swap as Example 4:

**Annuity** $A = 0.9650$ has units of "years" (sum of $\delta P$).

**Check par swap identity:**

$$K_{\mathrm{par}} = \frac{P(0, 0.5) - P(0, 1.5)}{A} \approx 0.036269.$$

**Verify that PV of fixed leg at par equals PV of float leg:**

$$K_{\mathrm{par}} \cdot A \approx 0.036269 \times 0.9650 \approx 0.0350 = P(0, 0.5) - P(0, 1.5) = 0.0350.$$

**Units:** $K$ is 1/year, $A$ is years, so $KA$ is dimensionless PV—consistent with bond-price differences.

(Definitions of $S_{\alpha,\beta}$ and $C_{\alpha,\beta}$ are source-backed.)

---

### Example 6: LMM vs Swap Market Model "Natural Fit" Demo (Toy)

Take the two vanilla options priced above:

**Caplet on $[0.5, 1.0]$** is directly Black under LMM because $F_1$ is lognormal under $\mathbb{Q}^2$ by construction.
- Using Example 2 inputs, we obtained $\mathrm{Cpl}_0 \approx 0.001194$.

**Swaption on $[0.5, 1.5]$** is directly Black under the swap market model because $S_{1,3}$ is modeled lognormal under $\mathbb{Q}^{1,3}$ and the payoff is annuity times $(S - K)^+$.
- Using Example 4 inputs, $\mathrm{Swpt}_0 \approx 0.002786$.

**Interpretation:** Each model gives you a "one-step" mapping from the quoted implied vol to the option price for the market instrument it is built around (caplets for LMM, swaptions for LSM). For the other instrument, you generally need either approximations or Monte Carlo (as emphasized in the calibration discussion).

---

### Example 7: Simple Calibration Step (Toy): Solve for One Caplet $\sigma$ by Bisection (Show Iterations)

**Goal:** find $\sigma$ so that the ATM caplet price equals the target price from Example 2:

$$\text{Target } \mathrm{Cpl}_0^{\mathrm{mkt}} = 0.001194.$$

**ATM formula (when $F = K$):**

$$\mathrm{Cpl}_0(\sigma) = P(0, 1.0) \cdot \delta \cdot F \cdot (\Phi(d_1) - \Phi(d_2)), \quad d_1 = \tfrac{1}{2}\sigma\sqrt{T}, \; d_2 = -\tfrac{1}{2}\sigma\sqrt{T}.$$

**Inputs:** $P(0, 1.0) = 0.975$, $\delta = 0.5$, $F = 0.030769$, $T = 0.5$.

#### Bracket

**Low vol $\sigma_L = 0.01$:** $v_L = \sigma_L\sqrt{0.5} = 0.01(0.7071) = 0.007071$.
Then $d_1 = 0.003536$, $d_2 = -0.003536$. For small $x$, $\Phi(x) - \Phi(-x)$ is tiny, so price is near 0: $\mathrm{Cpl}_0(\sigma_L) \approx 0.00004$ (very small).

**High vol $\sigma_H = 1.00$:** $v_H = 0.7071$, $d_1 = 0.3536$, $d_2 = -0.3536$.
$\Phi(0.35) \approx 0.6368 \Rightarrow \Phi(d_1) - \Phi(d_2) \approx 0.2736$.
Then $\mathrm{Cpl}_0(\sigma_H) \approx 0.975 \times 0.5 \times 0.030769 \times 0.2736 \approx 0.00411$.

So target $0.001194$ lies between.

#### Iteration 1: $\sigma_M = (0.01 + 1.00)/2 = 0.505$.

$v_M = 0.505(0.7071) = 0.3571$.
$d_1 = 0.1785$, $d_2 = -0.1785$.
$\Phi(0.18) \approx 0.5714 \Rightarrow \Phi(d_1) - \Phi(d_2) \approx 0.1428$.

Price:
$$\mathrm{Cpl}_0(\sigma_M) \approx 0.975 \times 0.5 \times 0.030769 \times 0.1428 \approx 0.00214.$$

This is above target, so set $\sigma_H \leftarrow 0.505$.

#### Iteration 2: $\sigma_M = (0.01 + 0.505)/2 = 0.2575$.

$v = 0.2575(0.7071) = 0.1821$.
$d_1 = 0.0910$, $d_2 = -0.0910$.
$\Phi(0.09) \approx 0.5359 \Rightarrow \Phi(d_1) - \Phi(d_2) \approx 0.0718$.

Price:
$$\mathrm{Cpl}_0 \approx 0.975 \times 0.5 \times 0.030769 \times 0.0718 \approx 0.00108.$$

This is below target, so set $\sigma_L \leftarrow 0.2575$.

At this point, $\sigma \in [0.2575, 0.505]$ and we already know the true toy value used in Example 2 was $\sigma \approx 0.28284$.

$$\boxed{\text{Final numeric output (after 2 iterations): bracket } \sigma \in [0.2575, 0.505]}$$

with midpoint $0.38125$ (not yet refined). With more iterations, it converges toward $\approx 0.283$.

---

### Example 8: Another Calibration Step (Toy): Solve for One Swaption $\sigma$ by Inversion Logic

**Target:** match the ATM swaption price from Example 4:

$$\mathrm{Swpt}_0^{\mathrm{mkt}} = 0.002786.$$

**ATM swaption Black:**

$$\mathrm{Swpt}_0(\sigma) = A \cdot S \cdot (\Phi(d_1) - \Phi(d_2)), \quad d_1 = \tfrac{1}{2}\sigma\sqrt{T_\alpha}, \; d_2 = -\tfrac{1}{2}\sigma\sqrt{T_\alpha}.$$

**Inputs:** $A = 0.9650$, $S = 0.036269$, $T_\alpha = 0.5$.

**Try $\sigma = 0.20$:** $v = 0.20(0.7071) = 0.1414$, $d_1 = 0.0707$.
Approx $\Phi(0.07) \approx 0.5279 \Rightarrow \Phi(d_1) - \Phi(d_2) \approx 0.0558$.
Price $\approx 0.965 \times 0.036269 \times 0.0558 \approx 0.00195$ (below target).

**Try $\sigma = 0.30$:** $v = 0.30(0.7071) = 0.2121$, $d_1 = 0.1061$.
$\Phi(0.11) \approx 0.5438 \Rightarrow \Phi(d_1) - \Phi(d_2) \approx 0.0876$.
Price $\approx 0.965 \times 0.036269 \times 0.0876 \approx 0.00306$ (above target).

So $\sigma \in [0.20, 0.30]$. The toy value used in Example 4 was $\sigma \approx 0.28284$ giving $v = 0.2$ and price $0.002786$.

$$\boxed{\text{Final numeric output (simple inversion bracket): } \sigma \in [0.20, 0.30], \text{ with calibrated } \sigma \text{ near } 0.283.}$$

---

### Example 9: Correlation Effect Toy (Illustrative Risk Intuition)

I'm not sure this exact "weighted-sum variance" proxy is explicitly presented in the market-model sections of the sources; I'm using it as a generic covariance identity to build intuition for why correlation matters for basket-like objects (swap rates are weighted combinations of forwards).

Let $X_1, X_2$ be random shocks (e.g., normalized log-forward increments) with:
- $\mathrm{Var}(X_1) = \sigma_1^2$, $\mathrm{Var}(X_2) = \sigma_2^2$, $\mathrm{Corr}(X_1, X_2) = \rho$.

Consider $Y = w_1 X_1 + w_2 X_2$. Then:

$$\mathrm{Var}(Y) = w_1^2 \sigma_1^2 + w_2^2 \sigma_2^2 + 2 w_1 w_2 \rho \sigma_1 \sigma_2.$$

**Choose numbers:**
- $w_1 = 0.6$, $w_2 = 0.4$
- $\sigma_1 = 1.0\% \Rightarrow 0.010$, $\sigma_2 = 1.5\% \Rightarrow 0.015$

**Compute:**
- $w_1^2 \sigma_1^2 = 0.36(0.0001) = 0.000036$
- $w_2^2 \sigma_2^2 = 0.16(0.000225) = 0.000036$
- Cross term $2 w_1 w_2 \sigma_1 \sigma_2 = 2(0.24)(0.00015) = 0.000072$

**If $\rho = 0$:** $\mathrm{Var}(Y) = 0.000036 + 0.000036 = 0.000072$.
Std $\approx \sqrt{0.000072} = 0.008485$ (0.8485%).

**If $\rho = 0.8$:** $\mathrm{Var}(Y) = 0.000072 + 0.8(0.000072) = 0.0001296$.
Std $\approx \sqrt{0.0001296} = 0.011387$ (1.1387%).

**Takeaway:** raising correlation boosts the variance (and therefore option value) of a basket-like rate.

---

### Example 10: Consistency Check: $\sigma = 0$ Collapses to Deterministic Rates and Options Go to Intrinsic/0

**Caplet B from Example 3:** $F = 0.041885$, $K = 0.035$, pay at $1.5$, $P(0, 1.5) = 0.955$, $\delta = 0.5$.

If $\sigma = 0$, then $F(T) \equiv F(0)$ deterministically under any equivalent measure, so

$$\mathrm{Cpl}_0(\sigma = 0) = P(0, 1.5) \cdot \delta \cdot \max(F - K, 0) = 0.955(0.5)(0.006885) = 0.4775(0.006885) \approx 0.003287.$$

Compare to $\sigma > 0$ price from Example 3 ($\approx 0.00366$): the option value is higher when volatility is positive.

**ATM swaption from Example 4:** $S = K$.

Intrinsic at expiry is zero, so with $\sigma = 0$,

$$\mathrm{Swpt}_0(\sigma = 0) = 0.$$

With $\sigma > 0$, we got $\approx 0.002786$.

**Sanity check:** option values are nonnegative and increasing in $\sigma$; and as $\sigma \to 0$ they approach intrinsic.

---

## 9. Practical Notes

### 9.1 Common Pitfalls

**Mixing measures (pricing under the wrong measure/numeraire):**
- Caplets: natural under $T_{i+1}$-forward measure when payoff at $T_{i+1}$.
- Swaptions: natural under swap annuity measure for that swap's annuity.

**Assuming lognormality in the wrong place:**
- In LMM, $F_i$ is lognormal under $\mathbb{Q}^{i+1}$, not under $\mathbb{Q}$.
- In swap market model, $S_{\alpha,\beta}$ is lognormal under $\mathbb{Q}^{\alpha,\beta}$, not necessarily under other measures.

**Discounting curve vs projection curve (multi-curve):**
- The sources discuss multiple yield curves and OIS discounting. If you price with OIS discounting but still model forwards as "LIBOR-style" projection rates, you must be explicit about which curve generates $P(t, T)$ and which generates $F_i(t)$.
- I'm not sure how deep you want multi-curve to go in this appendix (it quickly becomes model- and desk-convention dependent).

**Negative-rate environments:**
- Pure lognormal models keep rates strictly positive. The sources discuss extensions such as displaced diffusion / shifted lognormal / SABR-like approaches as smile/shape remedies.
- I'm not sure whether the book explicitly frames these as "negative-rate fixes" in the LMM chapter excerpt, but the displaced/shifted family is the natural direction if your desk requires it.

**Overfitting and instability ("ringing"):**
- The sources explicitly warn that calibration that is too local (row-by-row / bootstrap) can produce ringing and instability, and recommend regularization terms in the calibration norm.

**Bad market data / swaption matrix misalignment:**
- Swaption matrices can be stale in some entries; misalignment can create weird/unstable implied parameters (even imaginary vols).

### 9.2 Verification Tests

- **Reprice calibration instruments** (caps/caplets, swaptions) exactly.
- **Stability under small bumps:** bump a few quotes and re-calibrate; confirm parameter surface moves smoothly (ringing control).
- **Limiting cases:** $\sigma \to 0$: prices $\to$ intrinsic/0 (Example 10).
- **Arbitrage sanity:**
  - No negative option prices.
  - Option prices monotone increasing in vol.

---

## 10. Summary & Recall

### 10.1 Ten-Bullet Executive Summary

1. **Market models** target market-quoted rates (forwards or swaps) to align with Black(-style) quoting.

2. **Tenor grid** $\{T_i\}$ and accruals $\delta_i$ define the discrete structure for forwards and swaps.

3. **Forwards** $F_i(t)$ are defined from bonds by $F_i = \frac{1}{\delta_i}(P(t, T_i)/P(t, T_{i+1}) - 1)$.

4. Under $\mathbb{Q}^{i+1}$ (numeraire $P(t, T_{i+1})$), **LMM** models $F_i$ as driftless lognormal, enabling Black caplet pricing.

5. Under other measures, **LMM forward drifts become coupled** and depend on other forwards.

6. **Swap rate** $S_{\alpha,\beta} = (P(t, T_\alpha) - P(t, T_\beta))/C_{\alpha,\beta}(t)$ with annuity $C_{\alpha,\beta}(t) = \sum \delta P(t, T)$.

7. Under **swap annuity measure**, swap rate is a martingale; lognormal swap-rate assumption yields Black swaption pricing.

8. **LMM fits caplets naturally; swap market model fits swaptions naturally**; exact compatibility is not guaranteed but can be close in practice.

9. **Calibration** is an optimization balancing fit vs smoothness; regularization prevents "ringing."

10. **Simulation** uses a common measure (e.g., spot-LIBOR measure) and requires drift recomputation to maintain no-arbitrage.

### 10.2 Cheat Sheet

#### Key Objects

| Object | Definition |
|--------|------------|
| $P(t, T)$ | Bond price |
| $F_i(t)$ | Forward over $[T_i, T_{i+1}]$ |
| $C_{\alpha,\beta}(t)$ | Swap annuity |
| $S_{\alpha,\beta}(t)$ | Swap rate |
| $\mathrm{Bl}(K, F, v, \omega)$ | Black building block |

#### Measures / Numeraires

| Measure | Numeraire |
|---------|-----------|
| $\mathbb{Q}$ | $B(t)$ (money-market) |
| $\mathbb{Q}^i$ | $P(t, T_i)$ (bond forward measure) |
| $\mathbb{Q}^{\alpha,\beta}$ | $C_{\alpha,\beta}(t)$ (swap annuity measure) |

#### What is Lognormal Under What

| Model | Rate | Measure |
|-------|------|---------|
| LMM | $F_i$ lognormal | under $\mathbb{Q}^{i+1}$ (by model choice) |
| Swap market model | $S_{\alpha,\beta}$ lognormal | under $\mathbb{Q}^{\alpha,\beta}$ (by model choice) |

#### Calibration Workflow (High Level)

1. Build curve $\Rightarrow P(0, T) \Rightarrow F_i(0)$.
2. Choose model (LMM or swap market).
3. Choose vol structure + correlation.
4. Fit to caplet/cap and/or swaption prices/vols with regularization.
5. Validate and stress bump.

### 10.3 Twenty-Five Flashcards (Q/A)

1. **Q:** What is the simply-compounded forward rate $F_i(t)$ on $[T_i, T_{i+1}]$?
   **A:** $F_i(t) = \frac{1}{\delta_i}\left(\frac{P(t, T_i)}{P(t, T_{i+1})} - 1\right)$.

2. **Q:** What is the numeraire for the $T$-forward measure?
   **A:** The zero-coupon bond $P(t, T)$.

3. **Q:** Under which measure is $F_i$ driftless in LMM?
   **A:** Under $\mathbb{Q}^{i+1}$ (the $T_{i+1}$-forward measure).

4. **Q:** What is the LMM diffusion for $F_i$ under its forward measure (scalar form)?
   **A:** $dF_i(t) = \sigma_i(t) F_i(t) dZ_i^{i+1}(t)$.

5. **Q:** What is "coupled drift" in LMM?
   **A:** Under other measures, drift of $F_k$ depends on other forwards via a sum term.

6. **Q:** Define swap annuity $C_{\alpha,\beta}(t)$.
   **A:** $C_{\alpha,\beta}(t) = \sum_{k=\alpha+1}^{\beta} \delta_k P(t, T_k)$.

7. **Q:** Define swap rate $S_{\alpha,\beta}(t)$.
   **A:** $S_{\alpha,\beta}(t) = \frac{P(t, T_\alpha) - P(t, T_\beta)}{C_{\alpha,\beta}(t)}$.

8. **Q:** Why is the swap annuity a natural numeraire for swaptions?
   **A:** It qualifies as a numeraire and makes the swap rate a martingale under the annuity measure.

9. **Q:** What is the payer swaption payoff at expiry in the swap-rate formulation?
   **A:** $C_{\alpha,\beta}(T_\alpha)(S_{\alpha,\beta}(T_\alpha) - K)^+$.

10. **Q:** What is Black "$\mathrm{Bl}$" in these notes?
    **A:** $\mathrm{Bl}(K, F, v, \omega) = \omega(F\Phi(\omega d_1) - K\Phi(\omega d_2))$, $d_1 = \frac{\ln(F/K) + \frac{1}{2}v^2}{v}$, $d_2 = d_1 - v$.

11. **Q:** In the swap market model, what is assumed lognormal?
    **A:** The swap rate under the swap (annuity) measure.

12. **Q:** In LMM, are forwards lognormal under the risk-neutral measure?
    **A:** Not generally; under other measures their drift becomes state-dependent/coupled.

13. **Q:** What does "Black compatibility" mean in this appendix?
    **A:** Model is set up so caplets or swaptions reduce to Black pricing under the appropriate measure.

14. **Q:** Why do we add regularization in calibration?
    **A:** To prevent unstable "ringing" and improve smoothness/hedging behavior.

15. **Q:** What's the practical warning about swaption matrices?
    **A:** Some entries can be stale/misaligned, causing calibration instability.

16. **Q:** Give an example correlation parameterization.
    **A:** $\rho_{ij} = \exp(-\beta |T_i - T_j|)$.

17. **Q:** What is reduced-rank correlation used for?
    **A:** Factor reduction and computational efficiency in simulation/calibration.

18. **Q:** What market instruments most directly pin down LMM vol levels?
    **A:** Caps/caplets (caplet implied vol surface).

19. **Q:** What market instruments most directly pin down swap market model vols?
    **A:** Swaptions (swaption vol matrix).

20. **Q:** Under which measure is a bond price divided by itself a martingale?
    **A:** Under its own bond-numeraire forward measure.

21. **Q:** What is the HJM drift restriction conceptually?
    **A:** Once vol is chosen, drift is fixed by no-arbitrage.

22. **Q:** Why might one calibrate to both caps and swaptions?
    **A:** To avoid discarding information; many exotics embed both cap-like and swaption-like features.

23. **Q:** What is a typical calibration tradeoff?
    **A:** Fit accuracy vs smoothness/regularity and hedging stability.

24. **Q:** What is multi-curve?
    **A:** Discounting and projection can be on different curves (e.g., OIS discounting).

25. **Q:** Name a smile extension direction mentioned by the sources.
    **A:** Local volatility, stochastic volatility, displaced diffusion, SABR-like approaches.

---

## 11. Mini Problem Set (16 Questions)

*(Provide brief solution sketches for questions 1–8 only)*

1. **Given** $P(0, 0) = 1$, $P(0, 0.5) = 0.99$, $\delta_0 = 0.5$, compute $F_0(0)$.

   **Sketch:** Use $F_0 = \frac{1}{0.5}(1/0.99 - 1) \approx 0.020202$.

2. **Using Example 1 discount factors,** compute $F_1(0)$ and $F_2(0)$.

   **Sketch:** $F_1 = 2(0.99/0.975 - 1) \approx 0.030769$; $F_2 = 2(0.975/0.955 - 1) \approx 0.041885$.

3. **For a swap from $0.5$ to $1.5$** with semiannual payments, compute the annuity $A$.

   **Sketch:** $A = 0.5 \cdot P(0, 1.0) + 0.5 \cdot P(0, 1.5) = 0.965$.

4. **Using the annuity and discount factors,** compute the par swap rate $S$.

   **Sketch:** $S = (P(0, 0.5) - P(0, 1.5))/A = 0.035/0.965 \approx 0.036269$.

5. **For an ATM caplet** with $F = K = 3\%$, $P(0, T) = 0.97$, $\delta = 0.5$, $T = 1$, $\sigma = 0.2$, compute $d_1, d_2$.

   **Sketch:** $v = \sigma\sqrt{T} = 0.2$, so $d_1 = 0.1$, $d_2 = -0.1$.

6. **Explain in one sentence** why caplets are "Black-compatible" in LMM.

   **Sketch:** Under $T_{i+1}$-forward measure, the model makes $F_i$ driftless lognormal, so caplet payoff expectation is exactly Black.

7. **Explain in one sentence** why swaptions are "Black-compatible" in the swap market model.

   **Sketch:** Under the annuity measure, swap rate is modeled lognormal and payoff is annuity times $(S - K)^+$, giving Black swaption formula.

8. **What is meant by "coupled drift"** in LMM?

   **Sketch:** Under a non-canonical measure, drift of one forward depends on other forwards through a sum involving $\frac{\tau_j F_j}{1 + \tau_j F_j}$ terms.

9. Show that $C_{\alpha,\beta}(t)$ has units of years (PVBP interpretation).

10. In LMM, why is it computationally convenient to use reduced-rank correlation?

11. Describe one reason why joint caps+swaptions calibration can be challenging.

12. How can stale entries in a swaption matrix distort calibration?

13. If you increase correlation between forward rates, what qualitative effect do you expect on swaption prices (holding vols fixed)?

14. Describe a regularization term one might add to a calibration objective to reduce ringing. (I'm not sure if a specific form is specified in the excerpt; answer generically.)

15. Explain the conceptual analogy between HJM drift restriction and LMM drift coupling.

16. Give one reason why lognormal models may be problematic when rates can be negative.

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

- Forward rate definition, bond price relationships, measure definitions: Andersen & Piterbarg Vol 1, Brigo & Mercurio
- Black function $\mathrm{Bl}$ definition and cap/swaption formulas: Andersen & Piterbarg, Brigo & Mercurio
- LMM dynamics under forward measure, coupled drift formulas: Andersen & Piterbarg Vol 2, Brigo & Mercurio Ch 6-7
- Swap annuity as numeraire, swap rate martingale property: Andersen & Piterbarg, Brigo & Mercurio
- Calibration as optimization, regularization, ringing warnings: Andersen & Piterbarg Vol 2
- Correlation parameterizations (exponential, Rebonato-style): Andersen & Piterbarg Vol 2, Brigo & Mercurio

### (B) Reasoned Inference — Note Derivation Logic

- Connection between LMM and HJM (discrete vs continuous tenor): derived from comparing forward dynamics structure
- "Black compatibility" as design constraint: inferred from the alignment between model measures and market quoting conventions
- Calibration workflow structure: synthesized from calibration discussion themes in sources

### (C) Speculation — Flag Uncertainties

- Exact numerical optimizer choice for calibration (LM vs gradient methods): not pinned down in excerpts
- Full risk-neutral drift expression: depends on bank-account construction and $\beta(t)$ definition not fully reproduced
- Multi-curve depth appropriate for this appendix: desk-convention dependent
- Negative-rate handling as explicit "fix" in LMM context: natural direction but not explicitly framed this way in excerpts
