# Appendix A2: Short-Rate Models (Vasicek / CIR / Hull–White)

## What They Fit and Why

---

## Conventions & Notation

| Symbol | Definition |
|--------|------------|
| $t \geq 0$ | Time in years |
| $T > t$ | Maturity |
| $r_t$ | Short rate at time $t$ (instantaneous continuously compounded rate) |
| $B(t)$ | Money-market account numeraire, $B(t) = \exp\left(\int_0^t r_u \, du\right)$ |
| $Q$ | Risk-neutral (bank-account) measure |
| $W_t$ | Brownian motion under $Q$ |
| $P(t,T)$ | Zero-coupon bond price at $t$ for maturity $T$ |
| $D(t,T)$ | Stochastic discount factor, $D(t,T) = \exp\left(-\int_t^T r_u \, du\right)$ |
| $P^M(0,t)$ | Initial ("market") discount curve observed at time 0 |
| $f^M(0,t)$ | Market instantaneous forward rate, $f^M(0,t) = -\frac{\partial \ln P^M(0,t)}{\partial t}$ |

**Vasicek parameters:** $a > 0$ (mean-reversion speed, 1/year), $b$ (mean level, 1/year), $\sigma > 0$ (vol scale)

**CIR parameters:** $a > 0$, $b \geq 0$, $\sigma > 0$

**Hull–White (extended Vasicek) parameters:** $a > 0$, $\sigma > 0$, deterministic function $\theta(t)$

**Note:** Hull's notation in one-factor Hull–White uses $F(0,t)$ for the instantaneous forward at time 0.

### Key Relationships

**Money-market account** (standard numeraire for risk-neutral pricing):
$$B(t) = \exp\left(\int_0^t r_u \, du\right)$$

**Risk-neutral pricing** for payoff $H$ at $T$:
$$\pi_t = \mathbb{E}^Q\left[D(t,T) H \mid \mathcal{F}_t\right]$$

**Zero-coupon bond price:**
$$P(t,T) = \mathbb{E}^Q\left[D(t,T) \mid \mathcal{F}_t\right]$$

---

## 0. Setup

### Conventions used in this appendix

- One-factor short-rate diffusion models under a pricing (bank-account) measure $Q$
- Continuous-time, continuous compounding
- Curve-fitting language refers to matching the observed initial discount curve $P^M(0,T)$ (or equivalently $f^M(0,t)$)

### Notation glossary (symbols + definitions)

| Symbol | Definition | Units |
|--------|------------|-------|
| $r_t$ | Short rate at time $t$ | 1/year |
| $B(t)$ | Money-market account numeraire | dimensionless |
| $D(t,T)$ | Stochastic discount factor from $t$ to $T$ | dimensionless |
| $P(t,T)$ | ZCB price at $t$ for maturity $T$ | dimensionless |
| $Q$ | Bank-account (risk-neutral) measure | — |
| $W_t$ | Brownian motion under $Q$ | — |

---

## 1. Core Concepts (Definitions First)

### 1.1 Short Rate $r_t$ and Money-Market Account $B(t)$

**Formal Definition**

The short rate $r_t$ is the instantaneous continuously compounded riskless rate.

The money-market account (bank account numeraire) is:
$$B(t) = \exp\left(\int_0^t r_u \, du\right)$$
which is the accumulation of investing at the short rate continuously.

**Intuition**

Over a small interval $[t, t + \Delta t]$, the bank account grows approximately by $r_t \Delta t$ in relative terms.

**Trading / Risk / Portfolio Practice**

- $r_t$ is not directly traded; it is a model state variable chosen so that the model implies ZCB prices and other rates
- $B(t)$ is the natural numeraire for discounting under $Q$ in short-rate models

### 1.2 Zero-Coupon Bond Price $P(t,T)$

**Formal Definition**

$P(t,T)$ is the time-$t$ price of a claim paying 1 at time $T$.

**Risk-Neutral Pricing Assumption (state clearly)**

Under an equivalent martingale measure associated with the bank-account numeraire, a payoff $H$ at time $T$ has time-$t$ price:
$$\pi_t = \mathbb{E}^Q\left[D(t,T) H \mid \mathcal{F}_t\right], \quad D(t,T) = \exp\left(-\int_t^T r_u \, du\right)$$

Taking $H = 1$ gives:
$$\boxed{P(t,T) = \mathbb{E}^Q\left[D(t,T) \mid \mathcal{F}_t\right]}$$

**Intuition**

A ZCB price is an average (under $Q$) of stochastic discounting driven by the path of the short rate.

**Trading / Risk / Portfolio Practice**

Even if you never trade ZCBs directly, they are the "atoms" of the curve: many rate products can be decomposed into claims on discount factors (e.g., caplets as portfolios of ZCB options in standard short-rate frameworks).

### 1.3 Risk-Neutral Measure and Numeraire Choice

**Formal Definition**

A pricing measure $Q$ is tied to a numeraire (here $B(t)$). Under this measure, discounted asset prices (by the numeraire) are martingales, enabling expectation-based pricing.

**Intuition**

You can price by "expected discounted payoff" once you specify:
1. What you discount with (numeraire), and
2. Under which measure you take expectations

**Trading / Risk / Portfolio Practice**

A recurring pitfall is mixing a real-world short-rate dynamics (for forecasting / scenario analysis) with a risk-neutral dynamics (for derivative pricing). Hull explicitly distinguishes real-world and risk-neutral processes for Vasicek/CIR via a market price of interest rate risk.

### 1.4 Affine Term Structure (What "Affine" Means)

**Formal Definition**

A term structure is affine (in the short rate) if the spot rate $R(t,T)$ is an affine function of $r_t$:
$$R(t,T) = \alpha(t,T) \, r_t + \beta(t,T)$$

which is equivalent to an exponential-affine ZCB price:
$$\boxed{P(t,T) = A(t,T) \, e^{-B(t,T) \, r_t}}$$

**Intuition**

"Affine" means: once you know $r_t$, the whole curve $T \mapsto P(t,T)$ moves in a structured way, with $\log P$ linear in $r_t$.

**Trading / Risk / Portfolio Practice**

Affine structure is prized because it yields:
- Closed-form ZCB prices
- Often tractable option pricing (or at least efficient numerical methods)
- Simple risk sensitivities such as $\partial P / \partial r = -BP$ in these models

### 1.5 "Exogenous Term Structure" and Fitting the Initial Curve

**Formal Definition**

"Exogenous term structure" means the initial curve $P^M(0,T)$ (or $f^M(0,t)$) is treated as an input from the market. The model is then constructed (or extended) so that model-implied bond prices match $P^M(0,T)$ at time 0.

**Key Fact About Basic (Time-Homogeneous) Short-Rate Models**

Time-homogeneous models (e.g., Vasicek/CIR with constant parameters) imply a specific functional family for $P(0,T)$. In practice this may fail to match the observed $P^M(0,T)$ even with parameter tuning, and the small number of parameters can prevent satisfactory calibration.

**Intuition**

In a basic one-factor model, the entire initial curve is "endogenous" — determined by a handful of parameters and the current state. Markets, however, provide a full curve across maturities, which is too rich to be matched exactly by a low-parameter model unless you add flexibility.

**Trading / Risk / Portfolio Practice**

Hull emphasizes that Vasicek/CIR are useful for simulating interest-rate behavior, but for derivatives valuation it is usually necessary to use a model that provides an exact fit to the current term structure.

The Hull–White extension introduces a deterministic time-dependent drift function precisely to achieve that curve fit (while keeping much of the tractability).

---

## 2. Vasicek Model (Gaussian OU Short Rate)

### 2.1 Model SDE

Under a pricing (risk-neutral) specification, the Vasicek short rate follows an Ornstein–Uhlenbeck (OU) process:

$$\boxed{dr_t = a(b - r_t) \, dt + \sigma \, dW_t}$$

This is the same structure as $dr_t = k(\theta - r_t) \, dt + \sigma \, dW_t$ used in other references, with $a \leftrightarrow k$ and $b \leftrightarrow \theta$.

**Real-World vs Risk-Neutral Note (Hull)**

Hull writes a real-world Vasicek with market price of risk $\lambda$, and shows the risk-neutral process has the same form with adjusted mean level $b^*$ (notation in the source):
$$dr = a(b - r) \, dt + \sigma \, dz \quad \Rightarrow \quad dr = a(b^* - r) \, dt + \sigma \, dz$$
where $b^* = b + \lambda\sigma/a$.

### 2.2 Exact Solution for $r_t$ and Its Distribution

**Exact Solution (Integrating Factor Method)**

$$\boxed{r_t = r_0 e^{-at} + b(1 - e^{-at}) + \sigma \int_0^t e^{-a(t-u)} \, dW_u}$$

**Conditional / Marginal Distribution**

The process is Gaussian with:

$$\boxed{\mathbb{E}[r_t] = r_0 e^{-at} + b(1 - e^{-at})}$$

$$\boxed{\text{Var}(r_t) = \frac{\sigma^2}{2a}(1 - e^{-2at})}$$

**Negative Rate Implication**

Because the distribution is normal, negative rates can occur with positive probability. This is explicitly noted as a consequence of the normal distribution of the short rate.

### 2.3 Bond Pricing: Affine Form and $A(t,T)$, $B(t,T)$

**Affine (Exponential-Affine) Bond Price**

$$P(t,T) = A(t,T) \exp(-B(t,T) \, r_t)$$

**Closed-Form $B(t,T)$**

$$\boxed{B(t,T) = \frac{1 - e^{-a(T-t)}}{a}}$$

**Closed-Form $A(t,T)$ (Hull's Expression)**

$$\boxed{A(t,T) = \exp\left(\frac{(B(t,T) - T + t)(a^2 b - \sigma^2/2)}{a^2} - \frac{\sigma^2 B(t,T)^2}{4a}\right)}$$

### 2.4 Practical Interpretation: What It Fits Well and What It Fails At

**Fits Well (Why Used)**

- **Tractability:** Gaussian OU dynamics yield closed-form expressions for ZCB prices (and, in many treatments, tractable option pricing). The model is listed as having analytical bond and bond option prices in the comparative summary table of short-rate models.
- **Mean reversion:** parameter $a$ controls the speed at which $r_t$ is pulled toward $b$.

**Fails / Limitations (As Supported)**

- **Negative rates:** normality allows $r_t < 0$
- **Initial curve fit:** time-homogeneous short-rate models generally do not match the observed initial term structure, and the limited number of parameters can prevent satisfactory calibration to market curves

Hull specifically notes the Vasicek/CIR models do not provide an exact fit to the current term structure, which motivates using models (like Hull–White) that do.

### 2.5 Unit Checks and Limiting Cases

**Units**
- $r$ is $1/\text{year}$
- $B(t,T)$ has units of years, so $B(t,T) \, r_t$ is dimensionless
- $A(t,T)$ is dimensionless, and $P(t,T) \in (0, \infty)$ is dimensionless

**Limiting Cases**

| Limit | Result |
|-------|--------|
| $T \to t$ | $B(t,T) \to 0$, $A(t,T) \to 1$, so $P(t,t) = 1$ (boundary condition) |
| $\sigma \to 0$ | Stochastic integral disappears; $r_t$ becomes deterministic (pure mean reversion) |
| $a \to 0$ | $B(t,T) \to (T-t)$ by the expansion $1 - e^{-a\Delta} \sim a\Delta$ |

---

## 3. CIR Model (Square-Root Diffusion)

### 3.1 Model SDE

The CIR short rate follows:

$$\boxed{dr_t = a(b - r_t) \, dt + \sigma\sqrt{r_t} \, dW_t}$$

Hull notes a common specification for the market price of interest rate risk in CIR, $\lambda = \kappa\sqrt{r}$, and derives that the risk-neutral process remains CIR-form with adjusted parameters.

### 3.2 Positivity Discussion

**Non-Negativity**

Hull states that (unlike Vasicek) CIR does not allow negative short rates.

**Strict Positivity Condition (Feller-Type)**

Hull: if $2ab \geq \sigma^2$, then $r(t)$ is never zero; otherwise it can touch zero.

Interest Rate Modeling explicitly links the classical strict positivity requirement for CIR to $2\kappa\vartheta \geq \sigma^2$ (same structure with different notation).

### 3.3 Bond Pricing: Affine Form and $A(t,T)$, $B(t,T)$

**Affine Bond Price**

$$P(t,T) = A(t,T) \, e^{-B(t,T) \, r(t)}$$

**Closed-Form $B(t,T)$ and $A(t,T)$ (Hull)**

Define:
$$\boxed{\gamma = \sqrt{a^2 + 2\sigma^2}}$$

Then:

$$\boxed{B(t,T) = \frac{2(e^{\gamma(T-t)} - 1)}{(\gamma + a)(e^{\gamma(T-t)} - 1) + 2\gamma}}$$

$$\boxed{A(t,T) = \left[\frac{2\gamma \exp\left(\frac{(a+\gamma)(T-t)}{2}\right)}{(\gamma + a)(e^{\gamma(T-t)} - 1) + 2\gamma}\right]^{2ab/\sigma^2}}$$

**Useful Sensitivity Identity**

For both Vasicek and CIR:
$$\boxed{\frac{\partial P(t,T)}{\partial r(t)} = -B(t,T) \, P(t,T)}$$

### 3.4 Practical Interpretation

**Level-Dependent Volatility**

Hull: the standard deviation of the change in the short rate in a small time interval is proportional to $\sqrt{r}$, so volatility increases with the level of rates.

**Benefits vs Vasicek**

Positivity / non-negativity is a key advantage compared with Gaussian models.

**Limitations (As Supported)**

As with Vasicek, the basic time-homogeneous CIR structure generally will not match the observed initial curve exactly; exact initial-curve fit motivates deterministic-shift extensions (CIR++ / shifted CIR), which are treated as extensions in the term-structure modeling literature.

### 3.5 Unit/Limiting Checks

**Units**

Same affine check: $B(t,T)$ has units of time; $Br$ is dimensionless; $A$ dimensionless.

**Limiting Checks**

| Limit | Result |
|-------|--------|
| $T \to t$ | $B(t,T) \to 0$ and $P(t,t) = 1$ (consistent with ZCB payoff) |
| $\sigma \to 0$ | Diffusion vanishes; model becomes deterministic mean reversion |

---

## 4. Hull–White (Extended Vasicek / Time-Dependent Drift) and "Fitting the Initial Curve"

### 4.1 Model Form and Relationship to Vasicek

**Hull–White One-Factor (Extended Vasicek) Model**

$$\boxed{dr(t) = [\theta(t) - a \, r(t)] \, dt + \sigma \, dz}$$

where $\theta(t)$ is a deterministic function.

**Relationship to Vasicek**

Hull notes this is Vasicek with a time-dependent mean-reversion level:
$$dr(t) = a \, [b(t) - r(t)] \, dt + \sigma \, dz, \quad b(t) = \frac{\theta(t)}{a}$$

### 4.2 How $\theta(t)$ Is Chosen to Fit Today's Curve $P^M(0,T)$

This is the exogenous term structure idea: the initial curve is taken from the market and built into the model.

**Step 1 (Input): Market Curve $P^M(0,t)$ and Its Forward Rates**

Assume the market discount curve $t \mapsto P^M(0,t)$ is given and sufficiently smooth, and define the market instantaneous forward rates:
$$f^M(0,t) = -\frac{\partial \ln P^M(0,t)}{\partial t}$$

**Step 2 (Choose $a, \sigma$; Then Set $\theta(t)$)**

Hull provides the specific function:

$$\boxed{\theta(t) = \frac{\partial F(0,t)}{\partial t} + a \, F(0,t) + \frac{\sigma^2}{2a}(1 - e^{-2at})}$$

where $F(0,t)$ is the instantaneous forward rate at time 0 for maturity $t$.

**Step 3 (Interpretation: What Is Exogenous vs Endogenous)**

| Type | Description |
|------|-------------|
| **Exogenous** | The entire initial curve $P^M(0,T)$ (or $F(0,t)$) is an input |
| **Endogenous** | Future randomness of $r_t$ (and therefore future $P(t,T)$) is generated by the diffusion with parameters $(a, \sigma)$ and deterministic $\theta(t)$ |

This construction is explicitly motivated by the observed "poor fitting of the initial term structure" by time-homogeneous models and the need to impose the initial curve.

### 4.3 Bond Pricing and Affine Form Under Hull–White

**Affine Bond Price**

$$P(t,T) = A(t,T) \exp(-B(t,T) \, r(t))$$

**$B(t,T)$**

$$\boxed{B(t,T) = \frac{1 - e^{-a(T-t)}}{a}}$$

**$A(t,T)$ Explicitly Uses the Initial Curve**

Hull gives:

$$\boxed{\ln A(t,T) = \ln\frac{P(0,T)}{P(0,t)} + B(t,T) \, F(0,t) - \frac{\sigma^2}{4a^3}(e^{-aT} - e^{-at})^2(e^{2at} - 1)}$$

**Key "Fit" Message**

Because $A(t,T)$ is built from $P(0,T)/P(0,t)$, the model is designed to be consistent with the initial curve used in the construction.

### 4.4 Practical Interpretation: What It Fits and Why; What It Doesn't

**What It Fits (And Why)**

- **Exact fit to initial curve:** the time-dependent drift is chosen precisely to incorporate the observed initial discount curve into the model. This addresses the limitation of time-homogeneous models that do not match the observed term structure.
- **Tractability:** in one-factor Gaussian exogenous term structure models (including Hull–White), ZCB bonds and (in many setups) options on ZCBs can be priced explicitly, which then supports pricing of caps/floors as portfolios of ZCB options and swaptions via Jamshidian decomposition.

**What It Doesn't (As Supported)**

- **Negative rates:** Hull–White is Gaussian; short rates are normally distributed and can become negative.
- **One-factor limitation:** a one-factor short-rate model implies perfect correlation between rates of different maturities, which is cited as a limitation for concrete pricing problems; these models are noted as still used for risk management.
- **Fitting volatility term structure by time-dependent $a(t), \sigma(t)$:** making more parameters time-dependent can fit more market features but is described as potentially dangerous due to unstable parameter estimation and the need for ad-hoc functional forms; the chapter motivation is to use only a deterministic shift/drift to fit the yield curve.

**Multi-Curve Reality**

I'm not sure. The provided short-rate model chapters here are largely formulated in a single-curve setting (one discount curve $P(0,T)$ and one short rate). To be certain about "multi-curve" (e.g., OIS discounting vs IBOR projection) adaptations, we would need a source section explicitly treating multi-curve short-rate modeling and calibration conventions.

---

## 5. "What They Fit and Why" — Model Comparison Table

| Feature | Vasicek | CIR | Hull–White (1F Extended Vasicek) |
|---------|---------|-----|----------------------------------|
| **State variable** | $r_t$ | $r_t$ | $r_t$ |
| **SDE under pricing measure** | $dr = a(b-r)dt + \sigma dW$ | $dr = a(b-r)dt + \sigma\sqrt{r}\,dW$ | $dr = [\theta(t) - ar]dt + \sigma dW$ |
| **Positivity** | No (Gaussian; can be negative) | Non-negative; strict positivity if $2ab \geq \sigma^2$ | No (Gaussian; can be negative) |
| **Volatility behavior** | Constant $\sigma$ | $\propto \sqrt{r}$ (level-dependent) | Constant $\sigma$ (Gaussian) |
| **Fits $P^M(0,T)$ exactly (as-is)** | Generally no (time-homogeneous limitation) | Generally no (same issue) | Yes, by choosing $\theta(t)$ from initial curve and using $A(t,T)$ built from $P(0,T)$ |
| **Analytical tractability** | ZCB closed form; often analytical bond options | ZCB closed form; analytical in summaries | ZCB affine with explicit $A, B$; used as exogenous term structure Gaussian model |
| **Typical "why used" (supported)** | Simple mean reversion; tractable; simulation and intuition; but not exact curve fit | Positivity + level-dependent vol; tractable; but not exact curve fit | Exact initial curve fit + tractability; used (at least) for risk management and for building analyzable pricing frameworks |

---

## 6. Calibration and Implementation Map (Practitioner-Facing)

### 6.1 Concrete Workflow (One-Desk, One-Currency Short-Rate Framework)

**Inputs**

- Market discount curve $P^M(0,T)$ (or equivalently $f^M(0,t)$)
- Optionally: market prices for caps/floors/swaptions (these can be related to ZCB options and priced in these frameworks)

**Choose Model**

Vasicek, CIR, or Hull–White(1F).

Decision driver: do you require an exact initial curve fit for pricing and risk explainability (Hull emphasizes this need for derivatives valuation).

**Fit Initial Curve**

- **Basic Vasicek/CIR:** no exact curve-fit mechanism; the implied curve is endogenous and may not match $P^M(0,T)$.
- **Practical workaround in the literature:** deterministic-shift extensions $r_t = x_t + \varphi(t)$ (Vasicek++ / CIR++), which can fit the initial curve while preserving a base-model structure.
- **Hull–White:** compute $\theta(t)$ from the initial forward curve and chosen $(a, \sigma)$:
$$\theta(t) = \frac{\partial F(0,t)}{\partial t} + aF(0,t) + \frac{\sigma^2}{2a}(1 - e^{-2at})$$

**Calibrate Remaining Parameters**

Goal: select $(a, \sigma)$ to match liquid option prices/vols (conceptually: minimize pricing errors).

The sources explicitly note one can attempt to fit not only the yield curve but also term structures of volatility by allowing time-dependent parameters, but warn this can be dangerous and unstable; they motivate focusing on deterministic curve-fitting while keeping other parameters simpler.

I'm not sure about a single canonical step-by-step calibration algorithm (objective function choice, weighting, exact instrument set) because the provided excerpts here do not lay out one unified recipe. To be certain, we'd need the specific calibration section(s) for the chosen model variant (e.g., Hull–White calibration to cap/floor vols vs swaption vols, and the market quoting conventions assumed).

**Produce Outputs**

*Pricing outputs:*
- $P(t,T)$, model-implied zero rates/forwards
- (When available) ZCB option prices, caps/floors, swaptions (via decompositions)

*Risk outputs:*
- State sensitivity: $\partial P / \partial r = -BP$ for ZCBs in affine models
- Scenario generation: simulate $r_t$ paths under a chosen measure (real-world for risk scenarios vs $Q$ for pricing)

---

## 7. Math and Derivations (Step-by-Step)

### 7.1 Risk-Neutral Valuation in Short-Rate Models

For payoff $H$ at $T$, price at $t$ is:

$$\pi_t = \mathbb{E}^Q\left[\exp\left(-\int_t^T r_u \, du\right) H \;\Big|\; \mathcal{F}_t\right]$$

which implies for a ZCB ($H = 1$):

$$P(t,T) = \mathbb{E}^Q\left[\exp\left(-\int_t^T r_u \, du\right) \Big| \; \mathcal{F}_t\right]$$

**Sanity Check:** $P(t,t) = 1$ because the integral is zero and payoff is immediate.

### 7.2 OU Solution (Vasicek) — Step-by-Step

Start from:
$$dr_t = a(b - r_t) \, dt + \sigma \, dW_t$$

**Step 1.** Rewrite:
$$dr_t + a r_t \, dt = ab \, dt + \sigma \, dW_t$$

**Step 2.** Multiply by integrating factor $e^{at}$:
$$e^{at} dr_t + a e^{at} r_t \, dt = ab \, e^{at} \, dt + \sigma e^{at} \, dW_t$$

**Step 3.** Recognize left-hand side:
$$d(e^{at} r_t) = ab \, e^{at} \, dt + \sigma e^{at} \, dW_t$$

**Step 4.** Integrate from 0 to $t$:
$$e^{at} r_t - r_0 = ab \int_0^t e^{au} \, du + \sigma \int_0^t e^{au} \, dW_u$$

**Step 5.** Compute the deterministic integral:
$$\int_0^t e^{au} \, du = \frac{e^{at} - 1}{a}$$

**Step 6.** Solve for $r_t$:
$$\boxed{r_t = r_0 e^{-at} + b(1 - e^{-at}) + \sigma \int_0^t e^{-a(t-u)} \, dW_u}$$

matching the sourced closed form.

**Mean and Variance**

The stochastic integral has mean 0, so:
$$\mathbb{E}[r_t] = r_0 e^{-at} + b(1 - e^{-at})$$

By Itô isometry:
$$\text{Var}(r_t) = \sigma^2 \int_0^t e^{-2a(t-u)} \, du = \sigma^2 \frac{1 - e^{-2at}}{2a}$$

consistent with the stated formula.

**Unit Check:** $\text{Var}(r_t)$ has units $(1/\text{year})^2$.

### 7.3 Affine Bond Pricing Derivation: Vasicek (Hull's PDE + Exponential-Affine Guess)

**Step 0 (PDE Under the Model)**

Hull states that, under Vasicek, $P(t,T)$ satisfies:
$$\frac{\partial P}{\partial t} + a(b-r)\frac{\partial P}{\partial r} + \frac{\sigma^2}{2}\frac{\partial^2 P}{\partial r^2} - rP = 0$$

with boundary $P(T,T) = 1$.

**Step 1 (Guess Exponential-Affine Form)**

Assume:
$$P(t,T) = A(t,T) e^{-B(t,T)r}$$

**Step 2 (Differentiate)**

- $\partial P / \partial r = -B \, P$
- $\partial^2 P / \partial r^2 = B^2 P$
- $\partial P / \partial t = A_t e^{-Br} - AB_t r e^{-Br} = P(A_t/A - B_t r)$

**Step 3 (Substitute Into PDE, Divide by $P$)**

$$\left(\frac{A_t}{A} - B_t r\right) + a(b-r)(-B) + \frac{\sigma^2}{2}(B^2) - r = 0$$

Collect terms in $r$ and constants:

- **Coefficient of $r$:** $-B_t + aB - 1 = 0$
- **Constant term:** $\frac{A_t}{A} - abB + \frac{\sigma^2}{2}B^2 = 0$

Hull presents the resulting ODEs (equivalently):
$$B_t - aB + 1 = 0, \quad A_t - abAB + \frac{\sigma^2}{2}AB^2 = 0$$

**Step 4 (Solve for $B(t,T)$)**

Solve $B_t - aB + 1 = 0$ backward with boundary $B(T,T) = 0$, giving:
$$\boxed{B(t,T) = \frac{1 - e^{-a(T-t)}}{a}}$$

**Step 5 (Solve for $A(t,T)$)**

With $B$ known, solve the $A$ ODE; Hull provides the closed form:
$$\boxed{A(t,T) = \exp\left(\frac{(B(t,T) - T + t)(a^2 b - \sigma^2/2)}{a^2} - \frac{\sigma^2 B(t,T)^2}{4a}\right)}$$

**Sanity Check:** $T = t \Rightarrow B = 0 \Rightarrow A = 1 \Rightarrow P(t,t) = 1$.

### 7.4 Affine Bond Pricing Derivation: CIR (Hull's PDE + Exponential-Affine Guess)

**Step 0 (PDE)**

Hull's PDE for CIR ZCB pricing:
$$\frac{\partial P}{\partial t} + a(b-r)\frac{\partial P}{\partial r} + \frac{\sigma^2 r}{2}\frac{\partial^2 P}{\partial r^2} - rP = 0$$

**Step 1 (Guess)**
$$P(t,T) = A(t,T) e^{-B(t,T)r}$$

**Step 2 (Substitute)**

Using $\partial P / \partial r = -BP$, $\partial^2 P / \partial r^2 = B^2 P$, substitute into PDE and divide by $P$.

Hull states the resulting ODE system:
$$B_t - aB - \frac{1}{2}\sigma^2 B^2 + 1 = 0, \quad A_t - abAB = 0$$

with boundary $P(T,T) = 1$.

**Step 3 (Closed Forms)**

Hull provides:
$$\gamma = \sqrt{a^2 + 2\sigma^2}$$

$B(t,T)$ and $A(t,T)$ as in Section 3.3.

### 7.5 Hull–White Curve-Fit Derivation Map: How $\theta(t)$ Links to $P^M(0,T)$

**Core Point (Supported)**

Hull gives an explicit $\theta(t)$ in terms of the initial instantaneous forward curve $F(0,t)$ and its derivative:

$$\boxed{\theta(t) = \frac{\partial F(0,t)}{\partial t} + a \, F(0,t) + \frac{\sigma^2}{2a}(1 - e^{-2at})}$$

**How This Encodes "Fit the Initial Curve" (Mechanics You Can Verify)**

1. From the market curve $P^M(0,t)$, compute $F(0,t)$ via:
$$F(0,t) = -\frac{\partial \ln P^M(0,t)}{\partial t}$$
(Same definition appears as $f^M(0,t)$ in the short-rate chapter setup.)

2. Having fixed $(a, \sigma)$, compute $\theta(t)$ from the sourced formula.

3. Use Hull–White bond price:
$$P(t,T) = A(t,T)\exp(-B(t,T)r(t)), \quad B(t,T) = \frac{1 - e^{-a(T-t)}}{a}$$
and:
$$\ln A(t,T) = \ln\frac{P(0,T)}{P(0,t)} + B(t,T)F(0,t) - \frac{\sigma^2}{4a^3}(e^{-aT} - e^{-at})^2(e^{2at} - 1)$$

**Repricing Check at $t = 0$ (Algebraic Verification)**

Plug $t = 0$ into $\ln A$. The last term vanishes because $e^{2a \cdot 0} - 1 = 0$.

Then $A(0,T) = P(0,T)\exp(B(0,T)F(0,0))$.

Hence:
$$P(0,T) = A(0,T)e^{-B(0,T)r(0)} = P(0,T)\exp(B(0,T)(F(0,0) - r(0)))$$

Setting $r(0) = F(0,0)$ gives exact equality.

This is the clean way to see how the initial curve is "inserted" via $A(t,T)$.

**Connection to Deterministic Shifts (General Exogenous Term Structure Idea)**

The short-rate chapter also presents a general deterministic shift construction $r_t = x_t + \varphi(t)$ leading to bond prices that factor into a deterministic shift term times a base-model bond price, which is another way to view "fitting $P^M(0,T)$ by construction."

---

## 8. Worked Examples (At Least 10 Numeric Examples)

### Conventions for Examples

- Rates are in decimals (e.g., 5% $\to$ 0.05)
- Times are in years
- Exponentials are rounded to 4 d.p. as needed

---

### Example 1: Vasicek — Compute $\mathbb{E}[r_T]$ and $\text{Var}(r_T)$

**Given:**
- $a = 0.5$, $b = 0.05$, $\sigma = 0.02$, $r_0 = 0.03$, $T = 2$

**From Vasicek mean/variance:**
$$\mathbb{E}[r_T] = r_0 e^{-aT} + b(1 - e^{-aT}), \quad \text{Var}(r_T) = \frac{\sigma^2}{2a}(1 - e^{-2aT})$$

**Step-by-Step:**

1. $e^{-aT} = e^{-1} = 0.3679$

2. **Mean:**
$$\mathbb{E}[r_2] = 0.03(0.3679) + 0.05(1 - 0.3679) = 0.0110 + 0.05(0.6321) = 0.0110 + 0.0316 = 0.0426$$

3. **Variance:** $\sigma^2 = 0.0004$, $2a = 1$, $e^{-2aT} = e^{-2} = 0.1353$
$$\text{Var}(r_2) = \frac{0.0004}{1}(1 - 0.1353) = 0.0004(0.8647) = 0.0003459$$

4. **Standard deviation:** $\sqrt{0.0003459} = 0.0186$

**Result:** $\mathbb{E}[r_2] \approx 0.0426$, $\text{Var}(r_2) \approx 3.459 \times 10^{-4}$, $\text{SD}(r_2) \approx 0.0186$

---

### Example 2: Vasicek — Compute $B(0,T)$, $A(0,T)$, and $P(0,T)$

**Given:**
- $a = 0.3$, $b = 0.05$, $\sigma = 0.01$, $r_0 = 0.04$, $t = 0$, $T = 2$

**Formulas:**
$$B(0,T) = \frac{1 - e^{-aT}}{a}, \quad P(0,T) = A(0,T) e^{-B(0,T)r_0}$$
$$A(0,T) = \exp\left(\frac{(B(0,T) - T)(a^2 b - \sigma^2/2)}{a^2} - \frac{\sigma^2 B(0,T)^2}{4a}\right)$$

**Step-by-Step:**

1. $e^{-aT} = e^{-0.6} = 0.5488$

2. $B = \frac{1 - 0.5488}{0.3} = \frac{0.4512}{0.3} = 1.5039$

3. Compute $a^2 b - \sigma^2/2$:
   - $a^2 = 0.09$
   - $a^2 b = 0.09 \times 0.05 = 0.0045$
   - $\sigma^2/2 = 0.0001/2 = 0.00005$
   - Difference $= 0.00445$

4. Divide by $a^2$: $\frac{0.00445}{0.09} = 0.049444$

5. First exponent term: $(B - T) \times 0.049444 = (1.5039 - 2) \times 0.049444 = (-0.4961) \times 0.049444 = -0.02455$

6. Second exponent term: $\frac{\sigma^2 B^2}{4a} = \frac{0.0001 \times (1.5039)^2}{1.2}$
   - $B^2 = 2.2617$
   - Numerator $= 0.00022617$
   - Divide by 1.2 gives $0.0001885$

7. Total exponent: $\ln A = -0.02455 - 0.0001885 = -0.02474$
   - $A = e^{-0.02474} = 0.9756$

8. Bond price: $P(0,2) = 0.9756 \times \exp(-1.5039 \times 0.04)$
   - $-1.5039 \times 0.04 = -0.06016$
   - $\exp(-0.06016) = 0.9416$
   - $P = 0.9756 \times 0.9416 = 0.9189$

**Result:** $B(0,2) = 1.5039$, $A(0,2) \approx 0.9756$, $P(0,2) \approx 0.9189$

---

### Example 3: CIR — Check Positivity Intuition and Strict-Positivity Condition

**Given:**
- $a = 0.5$, $b = 0.04$, $\sigma = 0.10$, $r_0 = 0.03$

**Hull's Condition:** If $2ab \geq \sigma^2$, $r(t)$ is never zero; otherwise it can touch zero.

**Compute:**
- $2ab = 2(0.5)(0.04) = 0.04$
- $\sigma^2 = 0.01$
- Since $0.04 \geq 0.01$, the condition holds.

**Result:** The model suggests $r(t)$ stays strictly positive (never hits zero) under this parameter set.

---

### Example 4: CIR — Compute a Sample $P(0,T)$ Using $A, B$

**Use parameters designed for simple arithmetic:**
- $a = 0.3$, $b = 0.05$, $\sigma = 0.2828427$ (so $\sigma^2 = 0.08$)
- $r_0 = 0.04$, $T = 2$, $t = 0$

**Formulas:**
$$\gamma = \sqrt{a^2 + 2\sigma^2}$$

**Step-by-Step:**

1. $\gamma = \sqrt{0.09 + 0.16} = \sqrt{0.25} = 0.5$

2. Compute $e^{\gamma T} = e^1 = 2.7183$, so $e^{\gamma T} - 1 = 1.7183$

3. **Denominator:**
$$(\gamma + a)(e^{\gamma T} - 1) + 2\gamma = (0.5 + 0.3)(1.7183) + 1.0 = 0.8(1.7183) + 1.0 = 1.3746 + 1.0 = 2.3746$$

4. **$B(0,2)$:**
$$B = \frac{2(1.7183)}{2.3746} = \frac{3.4366}{2.3746} = 1.4470$$

5. **Exponent for $A$:**
$$\frac{2ab}{\sigma^2} = \frac{2(0.3)(0.05)}{0.08} = \frac{0.03}{0.08} = 0.375$$

6. **$A$ base ratio:**
$$\frac{2\gamma e^{(a+\gamma)T/2}}{(\gamma + a)(e^{\gamma T} - 1) + 2\gamma} = \frac{1.0 \cdot e^{(0.8) \cdot 1}}{2.3746} = \frac{e^{0.8}}{2.3746} = \frac{2.2255}{2.3746} = 0.9361$$

7. Raise to power 0.375:
   - $\ln A = 0.375 \ln(0.9361)$
   - $\ln(0.9361) \approx -0.0661$
   - $\ln A \approx 0.375(-0.0661) = -0.0248$
   - $A \approx e^{-0.0248} = 0.9755$

8. **Bond price:**
$$P(0,2) = A e^{-B r_0} = 0.9755 \cdot e^{-1.4470 \cdot 0.04}$$
   - Exponent $-1.4470 \cdot 0.04 = -0.05788$
   - $e^{-0.05788} = 0.9438$
   - $P = 0.9755 \times 0.9438 = 0.9200$

**Result:** $B(0,2) \approx 1.4470$, $A(0,2) \approx 0.9755$, $P(0,2) \approx 0.9200$

---

### Example 5: Hull–White — Compute $\theta(t)$ From a Toy Initial Curve

**Choose an initial discount curve defined by:**
$$P(0,t) = \exp(-0.02t - 0.0025t^2)$$

**Then:**
$$F(0,t) = -\frac{\partial \ln P(0,t)}{\partial t} = 0.02 + 0.005t, \quad \frac{\partial F(0,t)}{\partial t} = 0.005$$

This forward-rate definition matches the market-forward definition structure.

**Pick Hull–White parameters:**
- $a = 0.1$, $\sigma = 0.01$ ($\sigma^2 = 10^{-4}$)

**Hull's $\theta(t)$:**
$$\theta(t) = F_t(0,t) + aF(0,t) + \frac{\sigma^2}{2a}(1 - e^{-2at})$$

Compute $\frac{\sigma^2}{2a} = \frac{0.0001}{0.2} = 0.0005$.

**Now compute $\theta(t)$ at $t = 1, 2, 3, 4$:**

| $t$ | $F$ | $aF$ | $1 - e^{-2at}$ | Last term | $\theta(t)$ |
|-----|-----|------|----------------|-----------|-------------|
| 1 | 0.025 | 0.0025 | $1 - 0.8187 = 0.1813$ | 0.0000907 | $0.005 + 0.0025 + 0.0000907 = 0.0075907$ |
| 2 | 0.03 | 0.0030 | $1 - 0.6703 = 0.3297$ | 0.0001649 | $0.005 + 0.0030 + 0.0001649 = 0.0081649$ |
| 3 | 0.035 | 0.0035 | $1 - 0.5488 = 0.4512$ | 0.0002256 | $0.005 + 0.0035 + 0.0002256 = 0.0087256$ |
| 4 | 0.04 | 0.0040 | $1 - 0.4493 = 0.5507$ | 0.0002753 | $0.005 + 0.0040 + 0.0002753 = 0.0092753$ |

**Result:** $\theta(1) \approx 0.00759$, $\theta(2) \approx 0.00816$, $\theta(3) \approx 0.00873$, $\theta(4) \approx 0.00928$

---

### Example 6: Hull–White — Repricing Check Using the Fitted Curve

Use the same curve and parameters as Example 5.

**Compute a few market discount factors:**

| Maturity | $P(0,T)$ |
|----------|----------|
| 1 | $e^{-0.0225} = 0.9777$ |
| 2 | $e^{-0.05} = 0.9512$ |
| 3 | $e^{-0.0825} = 0.9208$ |

**From Hull's bond price:**
$$P(0,T) = A(0,T) e^{-B(0,T)r(0)}, \quad \ln A(0,T) = \ln P(0,T) + B(0,T)F(0,0)$$

because the last term in $\ln A(t,T)$ vanishes at $t = 0$.

Take $r(0) = F(0,0) = 0.02$ (since $F(0,t) = 0.02 + 0.005t$).

Then algebraically:
$$P(0,T) = P(0,T)\exp(B(0,T)(F(0,0) - r(0))) = P(0,T)$$

So repricing holds exactly in this idealized setup.

**Numeric Spot-Check (Showing Cancellation Explicitly)**

Pick $T = 2$:

1. $B(0,2) = \frac{1 - e^{-0.2}}{0.1} = \frac{1 - 0.8187}{0.1} = 1.8127$

2. $A(0,2) = P(0,2)\exp(B(0,2)F(0,0)) = 0.9512 \times e^{1.8127(0.02)}$
   - $e^{1.8127(0.02)} = e^{0.036254} = 1.0369$
   - $A(0,2) = 0.9512 \times 1.0369 = 0.9863$

3. Now multiply by $e^{-Br(0)} = e^{-1.8127(0.02)} = e^{-0.036254} = 0.9644$

4. $P(0,2) = 0.9863 \times 0.9644 = 0.9512$ (matches input)

**Result:** The fitted construction reproduces the input discount factors (within rounding).

---

### Example 7: Compare $P(0,T)$ Sensitivity to $r_0$: Vasicek vs CIR

Use Example 2 (Vasicek) and Example 4 (CIR), both with $r_0 = 0.04$, $T = 2$.

**Sensitivity Identity:**
$$\frac{\partial P}{\partial r_0} = -B(0,T) \, P(0,T)$$

**Vasicek:** $B = 1.5039$, $P = 0.9189$
$$\frac{\partial P}{\partial r_0} = -(1.5039)(0.9189) = -1.3818$$

A +1 bp shift ($\Delta r_0 = 0.0001$) gives $\Delta P \approx -0.0001382$.

**CIR:** $B = 1.4470$, $P = 0.9200$
$$\frac{\partial P}{\partial r_0} = -(1.4470)(0.9200) = -1.3312$$

A +1 bp shift gives $\Delta P \approx -0.0001331$.

**Result:** Under these toy parameters, CIR produces slightly smaller ZCB sensitivity to $r_0$ than Vasicek (because $B$ and $P$ differ).

---

### Example 8: Negative Rates Illustration Under Vasicek/HW — Compute $\mathbb{P}(r_T < 0)$

Because Vasicek is Gaussian with known mean/variance, $r_T \sim N(m, s^2)$.

**Pick parameters so that $m = 0.01$ and $s = 0.01$ at $T = 2$:**

Choose $a = 0.5$, $T = 2$, and set $r_0 = b = 0.01$ so:
$$m = r_0 e^{-aT} + b(1 - e^{-aT}) = 0.01$$

For the variance:
$$s^2 = \frac{\sigma^2}{2a}(1 - e^{-2aT}) = \sigma^2(1 - e^{-2})$$

since $2a = 1$. With $1 - e^{-2} = 0.8647$, choose $\sigma$ so that $s = 0.01$:
$$0.0001 = 0.8647 \, \sigma^2 \quad \Rightarrow \quad \sigma^2 = 0.0001156 \quad \Rightarrow \quad \sigma = 0.01075$$

**Then:**
$$\mathbb{P}(r_T < 0) = \mathbb{P}\left(\frac{r_T - m}{s} < \frac{0 - m}{s}\right) = \Phi\left(-\frac{m}{s}\right) = \Phi(-1) = 0.1587$$

**Result:** $\mathbb{P}(r_2 < 0) \approx 15.87\%$ for this toy Gaussian setup.

**Hull–White Note:** Hull–White is also Gaussian (normal short-rate distribution), so the same type of calculation applies once the mean is determined by $\theta(\cdot)$.

---

### Example 9: Mean-Reversion Speed Effect — Vary $a$ and Show Impact on $B$ and Bond Sensitivity

Use Vasicek ZCB sensitivity $\partial P / \partial r = -BP$.

**Hold:** $b = 0.05$, $\sigma = 0.01$, $r_0 = 0.04$, $T = 5$, $t = 0$. Compare:

#### Case A: $a = 0.1$

1. $B = \frac{1 - e^{-0.5}}{0.1} = \frac{1 - 0.6065}{0.1} = 3.9347$

2. Compute $A$ exponent:
   - $a^2 = 0.01$, $a^2 b = 0.0005$, $\sigma^2/2 = 0.00005$, difference $= 0.00045$
   - $(a^2 b - \sigma^2/2)/a^2 = 0.00045/0.01 = 0.045$
   - $B - T = 3.9347 - 5 = -1.0653$, so first term $= -1.0653 \times 0.045 = -0.04794$
   - Second term $\sigma^2 B^2/(4a) = 0.0001(15.480)/(0.4) = 0.00387$
   - $\ln A = -0.04794 - 0.00387 = -0.05181 \Rightarrow A = 0.9495$

3. $P = A e^{-Br_0} = 0.9495 \times e^{-3.9347(0.04)}$
   - Exponent $-0.15739$, $e^{-0.15739} = 0.8544$
   - $P = 0.9495 \times 0.8544 = 0.8110$

4. **Sensitivity:** $\frac{\partial P}{\partial r_0} = -BP = -(3.9347)(0.8110) = -3.190$

#### Case B: $a = 0.5$

1. $B = \frac{1 - e^{-2.5}}{0.5} = \frac{1 - 0.0821}{0.5} = 1.8358$

2. Compute $A$ exponent:
   - $a^2 = 0.25$, $a^2 b = 0.0125$, $\sigma^2/2 = 0.00005$, difference $= 0.01245$
   - $(a^2 b - \sigma^2/2)/a^2 = 0.01245/0.25 = 0.0498$
   - $B - T = 1.8358 - 5 = -3.1642$, first term $= -3.1642 \times 0.0498 = -0.1575$
   - Second term $\sigma^2 B^2/(4a) = 0.0001(3.368)/(2.0) = 0.000168$
   - $\ln A = -0.1575 - 0.000168 = -0.1577 \Rightarrow A = 0.8541$

3. $P = 0.8541 \times e^{-1.8358(0.04)}$
   - Exponent $-0.07343$, $e^{-0.07343} = 0.9292$
   - $P = 0.8541 \times 0.9292 = 0.7932$

4. **Sensitivity:** $\frac{\partial P}{\partial r_0} = -(1.8358)(0.7932) = -1.456$

**Result:** Higher mean reversion $a$ reduces $B$ and reduces the bond's sensitivity to the short rate.

---

### Example 10: Sanity Check — $\sigma \to 0$ Makes the Evolution Deterministic

Use Vasicek with $a = 0.3$, $b = 0.05$, $r_0 = 0.04$, $T = 2$.

**From the exact solution:**
$$r_T = r_0 e^{-aT} + b(1 - e^{-aT}) + \sigma \int_0^T e^{-a(T-u)} \, dW_u$$

If $\sigma = 0$, the stochastic integral is zero and $r_T$ is deterministic.

**Compute deterministic $r_2$:**

1. $e^{-aT} = e^{-0.6} = 0.5488$

2. $r_2 = 0.04(0.5488) + 0.05(1 - 0.5488) = 0.02195 + 0.05(0.4512) = 0.02195 + 0.02256 = 0.04451$

**Now compare with $\sigma = 0.01$:**

Variance (from formula) is:
$$\text{Var}(r_2) = \frac{0.0001}{2(0.3)}(1 - e^{-1.2}) = \frac{0.0001}{0.6}(1 - 0.3010) = 0.0001667(0.6990) = 0.0001165$$

So SD $\approx 0.01079$.

**Result:**
- $\sigma = 0 \Rightarrow r_2 = 0.04451$ exactly
- $\sigma = 0.01 \Rightarrow r_2$ is random with SD about $1.08\%$

---

## 9. Practical Notes

### 9.1 Common Pitfalls

**Mixing Measures Without Stating It**

Hull explicitly distinguishes real-world and risk-neutral short-rate processes using market price of risk adjustments. Treating an estimated real-world drift as if it were risk-neutral (or vice versa) can break pricing consistency.

**Confusing "Fit Initial Curve" with "Fit Vol Surface/Term Structure"**

- Deterministic drift/shift ($\theta(t)$ or $\varphi(t)$) fits the yield curve at time 0.
- Allowing time-dependent parameters can be used to also fit volatility term structures, but is described as potentially dangerous due to instability and ad-hoc parameterizations.

**Using Vasicek/HW Without Acknowledging Negative-Rate Implications**

Gaussian models imply normal distributions and can generate negative rates, which may be unacceptable depending on product and horizon.

**Over-Interpreting One-Factor Dynamics**

One-factor models imply perfect correlation between rates of different maturities; this limitation is explicitly highlighted for concrete pricing problems.

### 9.2 Verification Tests (Minimum Viable Model QA)

**Hull–White Curve Repricing**

Ensure that model $P(0,T)$ reproduces the input $P^M(0,T)$ to numerical tolerance (Example 6 shows the algebraic identity in the analytic setup).

**Bond Price Monotonicity in $r_t$**

Verify $\partial P / \partial r = -BP < 0$. This is a direct identity in affine Vasicek/CIR.

**Parameter Perturbation Stability**

Small changes in $a, \sigma$ should not lead to discontinuous shifts in key outputs (unless your calibration objective is ill-conditioned).

If you allow time-dependent parameters, re-check stability carefully (the sources warn about instability/overfitting concerns).

---

## 10. Summary & Recall

### 10.1 Executive Summary (10 Bullets)

1. Short-rate models specify an SDE for the instantaneous short rate $r_t$, and price ZCBs via $P(t,T) = \mathbb{E}^Q[\exp(-\int_t^T r_u \, du) \mid \mathcal{F}_t]$.

2. The bank account numeraire is $B(t) = \exp(\int_0^t r_u \, du)$.

3. An affine term structure means $P(t,T) = A(t,T) e^{-B(t,T)r_t}$ (equivalently spot rates affine in $r_t$).

4. Vasicek is Gaussian OU: $dr = a(b-r)dt + \sigma dW$, with explicit mean/variance and normal distribution.

5. Vasicek allows negative rates because it is Gaussian.

6. CIR: $dr = a(b-r)dt + \sigma\sqrt{r} \, dW$; volatility rises with rate level and rates are non-negative.

7. CIR strict positivity is linked to $2ab \geq \sigma^2$ (Feller-type).

8. Basic time-homogeneous Vasicek/CIR generally do not match the observed initial curve; limited parameters can prevent satisfactory curve calibration.

9. Hull–White extends Vasicek with time-dependent drift $\theta(t)$ so the model can fit the initial discount curve exactly while keeping tractable affine bond prices.

10. One-factor Gaussian exogenous term structure models are analytically convenient but limited: negative rates and perfect correlation across maturities; still used for risk management.

### 10.2 Cheat Sheet (SDEs + Key Pricing Identities + HW Curve-Fit Step)

**Risk-Neutral Pricing**
$$P(t,T) = \mathbb{E}^Q\left[\exp\left(-\int_t^T r_u \, du\right) \Big| \mathcal{F}_t\right]$$

**Vasicek**
$$dr = a(b-r)dt + \sigma dW, \quad P(t,T) = A(t,T)e^{-B(t,T)r_t}, \quad B(t,T) = \frac{1 - e^{-a(T-t)}}{a}$$
$$A(t,T) = \exp\left(\frac{(B - T + t)(a^2 b - \sigma^2/2)}{a^2} - \frac{\sigma^2 B^2}{4a}\right)$$

**CIR**
$$dr = a(b-r)dt + \sigma\sqrt{r} \, dW, \quad P(t,T) = A(t,T)e^{-B(t,T)r_t}, \quad \gamma = \sqrt{a^2 + 2\sigma^2}$$
$$B(t,T) = \frac{2(e^{\gamma(T-t)} - 1)}{(\gamma + a)(e^{\gamma(T-t)} - 1) + 2\gamma}, \quad A(t,T) = \left[\frac{2\gamma e^{(a+\gamma)(T-t)/2}}{(\gamma + a)(e^{\gamma(T-t)} - 1) + 2\gamma}\right]^{2ab/\sigma^2}$$

**Sensitivity (Vasicek and CIR)**
$$\frac{\partial P(t,T)}{\partial r(t)} = -B(t,T) P(t,T)$$

**Hull–White 1F**
$$dr = [\theta(t) - ar]dt + \sigma dW, \quad P(t,T) = A(t,T)e^{-B(t,T)r(t)}, \quad B(t,T) = \frac{1 - e^{-a(T-t)}}{a}$$
$$\ln A(t,T) = \ln\frac{P(0,T)}{P(0,t)} + B(t,T)F(0,t) - \frac{\sigma^2}{4a^3}(e^{-aT} - e^{-at})^2(e^{2at} - 1)$$

**Curve-Fit Step**
$$\theta(t) = F_t(0,t) + aF(0,t) + \frac{\sigma^2}{2a}(1 - e^{-2at})$$

### 10.3 Flashcards (25 Prompts with Answers)

| # | Prompt | Answer |
|---|--------|--------|
| 1 | Define the money-market account $B(t)$. | $B(t) = \exp(\int_0^t r_u \, du)$ |
| 2 | Define the ZCB price $P(t,T)$ in short-rate modeling. | Price at $t$ of payoff 1 at $T$; $P(t,T) = \mathbb{E}^Q[\exp(-\int_t^T r_u \, du) \mid \mathcal{F}_t]$ |
| 3 | What does "affine term structure" mean (in $r_t$)? | $P(t,T) = A(t,T) e^{-B(t,T)r_t}$ (equivalently spot rate affine in $r_t$) |
| 4 | Vasicek SDE. | $dr = a(b-r)dt + \sigma dW$ |
| 5 | Vasicek mean of $r_t$. | $\mathbb{E}[r_t] = r_0 e^{-at} + b(1 - e^{-at})$ |
| 6 | Vasicek variance of $r_t$. | $\text{Var}(r_t) = \frac{\sigma^2}{2a}(1 - e^{-2at})$ |
| 7 | Why can Vasicek generate negative rates? | $r_t$ is normally distributed (Gaussian) |
| 8 | Vasicek bond pricing form. | $P(t,T) = A(t,T) e^{-B(t,T)r_t}$ |
| 9 | Vasicek $B(t,T)$. | $B(t,T) = (1 - e^{-a(T-t)})/a$ |
| 10 | CIR SDE. | $dr = a(b-r)dt + \sigma\sqrt{r} \, dW$ |
| 11 | CIR positivity idea. | CIR does not allow negative rates; diffusion term vanishes at 0 |
| 12 | CIR strict-positivity condition in Hull. | If $2ab \geq \sigma^2$, $r(t)$ is never zero |
| 13 | CIR "level-dependent volatility" meaning. | Short-rate volatility scales like $\sqrt{r}$ |
| 14 | CIR bond pricing form. | $P(t,T) = A(t,T) e^{-B(t,T)r(t)}$ with $\gamma = \sqrt{a^2 + 2\sigma^2}$ |
| 15 | Common ZCB sensitivity identity in Vasicek/CIR. | $\partial P / \partial r = -BP$ |
| 16 | What is the market instantaneous forward rate $f^M(0,t)$? | $f^M(0,t) = -\partial_t \ln P^M(0,t)$ |
| 17 | Hull–White 1F SDE. | $dr = [\theta(t) - ar]dt + \sigma dW$ |
| 18 | Hull–White as time-dependent Vasicek. | $dr = a(b(t) - r)dt + \sigma dW$ with $b(t) = \theta(t)/a$ |
| 19 | Hull–White curve-fit formula for $\theta(t)$. | $\theta(t) = F_t(0,t) + aF(0,t) + \frac{\sigma^2}{2a}(1 - e^{-2at})$ |
| 20 | Hull–White bond pricing form. | $P(t,T) = A(t,T) e^{-B(t,T)r(t)}$ |
| 21 | Hull–White $B(t,T)$. | $B(t,T) = (1 - e^{-a(T-t)})/a$ |
| 22 | Why Hull–White can fit the initial curve. | $A(t,T)$ explicitly uses $P(0,T)/P(0,t)$ and $\theta(t)$ is built from the initial forward curve |
| 23 | Why time-homogeneous models struggle to fit $P^M(0,T)$. | They generate a restricted functional family for $P(0,T)$ with few parameters; may not match observed term structure |
| 24 | What is Jamshidian's decomposition used for in this context? | To price European swaptions explicitly by decomposing into bond options (in the short-rate framework) |
| 25 | What is a key limitation of one-factor short-rate models for pricing realism? | One-factor implies perfect correlation across maturities; Gaussian versions also allow negative rates; used more in risk management |

---

## 11. Mini Problem Set (16 Questions)

*Brief solution sketches for questions 1–8 only.*

---

**1. Derive the OU solution for Vasicek using an integrating factor.**

*Sketch:* Multiply by $e^{at}$, integrate, solve for $r_t$. Match the closed form in the notes.

---

**2. Using the Vasicek solution, compute $\mathbb{E}[r_t]$ and $\text{Var}(r_t)$.**

*Sketch:* Take expectation (stochastic integral mean 0) and apply Itô isometry for variance to obtain $\sigma^2(1 - e^{-2at})/(2a)$.

---

**3. Starting from Hull's Vasicek PDE, show that the exponential-affine ansatz implies ODEs for $A$ and $B$.**

*Sketch:* Substitute $P = Ae^{-Br}$ into the PDE, compute derivatives, collect $r$ and constant terms to recover Hull's ODEs.

---

**4. Solve the ODE $B_t - aB + 1 = 0$ with $B(T,T) = 0$ and recover $B(t,T)$.**

*Sketch:* Solve linear ODE; $B(t,T) = (1 - e^{-a(T-t)})/a$.

---

**5. In CIR, explain why volatility is "level dependent" and write the diffusion coefficient.**

*Sketch:* Diffusion term is $\sigma\sqrt{r}$; hence standard deviation in small time $\propto \sqrt{r}$.

---

**6. Using Hull's CIR positivity statement, determine whether parameters $a = 0.2$, $b = 0.03$, $\sigma = 0.15$ satisfy the strict-positivity condition.**

*Sketch:* Compute $2ab = 0.012$, $\sigma^2 = 0.0225$; condition fails, so it may touch zero.

---

**7. Show $\partial P / \partial r = -BP$ given $P = Ae^{-Br}$.**

*Sketch:* Differentiate: $\partial_r P = -BAe^{-Br} = -BP$.

---

**8. In Hull–White, explain how $P(0,T)$ is recovered from the bond pricing formula when $r(0) = F(0,0)$.**

*Sketch:* At $t = 0$, the last term in $\ln A$ is zero; $A(0,T) = P(0,T)\exp(BF(0,0))$; multiply by $\exp(-Br(0))$ to recover $P(0,T)$.

---

**9. Using the deterministic shift idea $r_t = x_t + \varphi(t)$, describe how bond prices factor into a deterministic term and a base-model term.**

---

**10. Compare the shape flexibility of Vasicek/CIR term structures as functions of $r(t)$ and parameters.**

---

**11. For Vasicek, compute the probability $r_T < 0$ given $r_T \sim N(m, s^2)$.**

---

**12. For CIR, write $\gamma$ and interpret its role in $A(t,T)$ and $B(t,T)$.**

---

**13. Explain why one-factor models imply perfect correlation between rates of different maturities.**

---

**14. Discuss why fitting time-dependent $a(t)$, $\sigma(t)$ might improve calibration but be "dangerous" in practice.**

---

**15. Outline how caps can be priced as portfolios of ZCB options in the short-rate framework.**

---

**16. Outline how European swaptions can be priced via Jamshidian decomposition in a one-factor short-rate model.**

---

## Source Map

### (A) Verified Facts — Source-Backed

- Vasicek/CIR/Hull–White SDEs, bond pricing formulas, and $A(t,T)$, $B(t,T)$ expressions: **Hull, *Options, Futures, and Other Derivatives***
- Feller positivity condition for CIR ($2ab \geq \sigma^2$): **Hull; Andersen & Piterbarg, *Interest Rate Modeling***
- Hull–White curve-fit formula for $\theta(t)$: **Hull**
- Risk-neutral pricing framework and bank-account numeraire: **Hull; Brigo & Mercurio, *Interest Rate Models: Theory and Practice***
- Limitations of time-homogeneous models for initial curve fit: **Hull**
- Deterministic shift extensions (Vasicek++/CIR++): **Andersen & Piterbarg; Brigo & Mercurio**

### (B) Reasoned Inference — Derived from (A)

- Limiting-case behavior ($\sigma \to 0$, $a \to 0$, $T \to t$): derived from closed-form expressions
- Unit checks on formulas: dimensional analysis applied to sourced formulas
- Worked examples: numeric calculations using sourced formulas

### (C) Speculation — Flagged Uncertainties

- Multi-curve adaptations: "I'm not sure" — sources here are single-curve focused
- Canonical calibration algorithm details: "I'm not sure" — excerpts do not provide unified recipe
