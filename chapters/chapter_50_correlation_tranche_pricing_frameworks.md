# Chapter 50: Correlation and Tranche Pricing Frameworks (Conceptual Map)

**Why Tranches Are Correlation-Sensitive; Gaussian Copula & Base Correlation as Market Language**

---

## Conventions & Notation

| Convention | Description |
|------------|-------------|
| **Time** | $t \ge 0$ in **years**; trade inception at $t=0$, maturity at $T$ |
| **Loss units** | Losses are expressed as **fractions of portfolio notional** unless stated otherwise |
| **Tranche strikes** | $A,D$ (attachment, detachment) are **fractions of portfolio notional** (e.g., $3\% \equiv 0.03$); tranche width $w=D-A$ |
| **Tranche loss fraction** | Always bounded in $[0,1]$ (loss as a fraction of tranche notional) |
| **Discount factor** | $Z(t)$ is the present value of \$1 at time $t$; risky cashflows are discounted with $Z(\cdot)$ |
| **Quoting conventions** | When quoting examples mention coupon frequency/day count, they are **as described in the reference text** (e.g., quarterly with Act/360 in one historical tranche quote table). Verify current market conventions separately |

---

## Notation Glossary

| Symbol | Definition |
|--------|------------|
| $i=1,\dots,N$ | Names in the reference portfolio |
| $\tau_i$ | Default time of name $i$ |
| $Y_{t,i}=\mathbf{1}_{\{\tau_i\le t\}}$ | Default indicator process (1 if defaulted by $t$, else 0) |
| $R_i$ | Recovery rate of name $i$; $\mathrm{LGD}_i=1-R_i$ |
| $L(t)$ | **Fractional portfolio loss** at time $t$ (fraction of portfolio notional lost to defaults by $t$) |
| $L_{\max}$ | Maximum possible portfolio loss under the recovery assumptions (e.g., with fixed $R=40\%$, $L_{\max}=60\%$) |
| $A,D$ | Tranche attachment/detachment (fractions of portfolio notional); $w=D-A$ |
| $L(t;A,D)$ (or $TL(t)$) | **Tranche loss fraction** at time $t$, in $[0,1]$ |
| $Q(t;A,D)=\mathbb{E}[1-L(t;A,D)]$ | Tranche survival curve / expected surviving tranche notional fraction |
| $\psi(T,K)=\mathbb{E}[\min(L(T),K)]$ | ETL of the **equity/base tranche** $[0,K]$ |
| $S(A,D)$ | Contractual tranche spread (annualized), paid on outstanding tranche notional |
| $\rho$ | (Copula) correlation parameter in one-factor Gaussian copula / latent-variable model |
| $\Phi(\cdot)$, $\Phi^{-1}(\cdot)$ | Standard normal CDF and inverse CDF |

---

## 0. Setup

### Conventions Used in This Chapter

- We treat **single-name marginal default curves** (survival/hazard) and **recoveries** as given inputs (from CDS calibration in prior chapters)
- We focus on how a **dependence model** turns these marginals into a **joint loss distribution** for the portfolio, which then drives tranche ETL and PV
- We speak in terms of **expected tranche loss** (ETL) and the **tranche survival curve** $Q(t,A,D)$ because tranche pricing can be mapped to CDS-style PV once $Q$ is known

---

## 1. Core Concepts (Definitions First)

### 1.1 Portfolio Loss $L(t)$ and Tranche Loss $L(t;A,D)$

**Formal Definition (Portfolio):**

The book defines **fractional portfolio loss** $L(T)$ as the fraction of reference-portfolio notional lost due to defaults by horizon $T$. With recoveries, $L(T)\in[0,L_{\max}]$.

**Formal Definition (Tranche):**

A key identity connects portfolio loss and tranche loss using "min" operators:

$$
(D-A)\,L(T;A,D)=\min(L(T),D)-\min(L(T),A)
$$

This is exactly how the text aggregates expected losses across contiguous tranches. Hence the tranche loss fraction can be written as:

$$
L(T;A,D)=\frac{\min(L(T),D)-\min(L(T),A)}{D-A}\in[0,1]
$$

**Intuition:**

- $L(T)$ is "how much the whole portfolio lost"
- $L(T;A,D)$ is "what fraction of the tranche notional got eaten after the portfolio burned through subordination $A$, capped at $D$"

**How It Appears in Practice:**

Traders/risk managers think in terms of:
- **Probability of zero tranche loss**: $\Pr(L(T)\le A)$
- **Probability of full tranche wipeout**: $\Pr(L(T)\ge D)$, which (per the text) increases with correlation in Gaussian-factor settings

---

### 1.2 Dependence vs Correlation

**Dependence (Formal):**

"Dependence" is about the **full joint behavior** of defaults (or default times) across names: joint probabilities like $\Pr(\tau_1\le t_1,\dots,\tau_N\le t_N)$.

**Correlation as a Summary Statistic:**

The Pearson linear correlation between random variables $X$ and $Y$ is:

$$
\rho_P=\frac{\mathbb{E}[XY]-\mathbb{E}[X]\mathbb{E}[Y]}{\sqrt{(\mathbb{E}[X^2]-\mathbb{E}[X]^2)(\mathbb{E}[Y^2]-\mathbb{E}[Y]^2)}}
$$

The text emphasizes why linear correlation can be misleading in credit: default indicators are Bernoulli, and the feasible range of their linear correlation depends on marginals.

**Copula as the Object Linking Marginals to Joint Behavior:**

A copula is a multivariate distribution function with uniform marginals that "links uni-variate marginals to a full multi-variate distribution." Formally, for uniform $\hat u_i$:

$$
\Pr(\hat u_1\le u_1,\dots,\hat u_N\le u_N)=C(u_1,\dots,u_N)
$$

**Sklar's Theorem:** Any joint CDF $H$ can be written using a copula and marginals $F_i$:

$$
H(x_1,\dots,x_N)=C(F_1(x_1),\dots,F_N(x_N))
$$

with uniqueness if marginals are continuous.

**Intuition:**

- Marginals tell you "how likely each name defaults by time $t$"
- The copula tells you "how likely they default together (cluster) vs separately"

**How It Appears in Practice:**

Desk language often compresses the copula choice into a parameter labeled "correlation," but conceptually it stands in for the dependence model. The book explicitly frames copulas as suited to correlation products because marginals come from CDS calibration, leaving flexibility for the joint.

---

### 1.3 Default Indicators and Default-Time Representation

**Formal Definition:**

Default time $\tau_i>0$, default indicator $Y_{t,i}=\mathbf{1}_{\{\tau_i\le t\}}$.

**Intuition:**

Modeling $\tau_i$ (time-to-default) is natural for copulas: we specify each $\Pr(\tau_i\le t)$ and then bind them together with a copula.

**How It Appears in Practice:**

Single-name calibration gives survival curves $Q_i(t)=\Pr(\tau_i>t)$. In latent-variable Gaussian settings, default thresholds are chosen to match these marginals.

---

## 2. Why Tranche Valuation Is Correlation-Sensitive (The Key Intuition)

### 2.1 Tranche Nonlinearity: $L(T;A,D)$ is a Nonlinear Function of $L(T)$

From the tranche mapping:

$$
L(T;A,D)=\frac{\min(L(T),D)-\min(L(T),A)}{D-A}
$$

the tranche loss is **piecewise linear** in $L(T)$, with kinks at $A$ and $D$.

**Why This Matters:**

- Expected tranche loss $\mathbb{E}[L(T;A,D)]$ depends on the **distribution** of $L(T)$, not just its mean
- Changing dependence (often summarized by a "correlation" parameter) reshapes the distribution of $L(T)$ and therefore reshapes ETL differently for different $[A,D]$

---

### 2.2 Qualitative "Shape" Explanation: Higher Correlation Pushes Probability Mass to Extremes

In one-factor Gaussian-style models, conditional on the market factor, defaults become independent; but unconditionally, the portfolio exhibits **systemic clustering** (good states: few defaults; bad states: many defaults).

The text explicitly highlights extreme-factor limits:
- $Z\to -\infty$: portfolio loss tends to $(1-R)$ (all default)
- $Z\to +\infty$: portfolio loss tends to $0$ (no defaults)

**Interpretation:** Higher correlation makes "good vs bad world" states more influential → more probability at **very low** and **very high** losses.

---

### 2.3 Who Benefits / Who Is Hurt? (Must Be Tied to Tranche Position)

To avoid sign confusion, fix a position:

Consider a **short protection** tranche position (i.e., you receive premium and pay tranche losses). This is the "protection seller" framing used in the text's implied correlation discussion.

**Equity (Low Attachment):**

The book states a short protection equity tranche holder is **long correlation** in the sense that increasing correlation can increase tranche PV for that holder (more scenarios with no defaults).

*Intuition:* Equity mostly cares about whether there are *any* losses. More correlation increases the chance of "almost no defaults," which keeps equity outstanding.

**Senior / Super-Senior (High Attachment):**

Senior tranches are dominated by tail events. The probability of full loss $\Pr(L(T)\ge D)$ increases with correlation in the LHP tranche-loss discussion.

*Intuition:* Correlation increases the chance of **many defaults together**, which is exactly what can reach high attachments.

**Tie to ETL(t) and PV Legs (Chapter 49 Treated as Known):**

Tranche PV depends on the **expected path of tranche notional outstanding**, equivalently $Q(t;A,D)=\mathbb{E}[1-L(t;A,D)]$.

- Premium leg PV is increasing in expected outstanding notional
- Protection leg PV is increasing in expected increments of tranche loss

Therefore anything (like correlation/dependence) that changes ETL over time changes both legs.

---

## 3. From Marginal Default Risk to Joint Loss Distribution: Modeling Map

This section is a **conceptual workflow** (not code).

### Step 1: Specify Single-Name Marginal Default Behavior (Given Input)

Inputs per name $i$:
- Survival curve $Q_i(t)=\Pr(\tau_i>t)$ from CDS calibration
- Recovery $R_i$ (assumption or market-implied)

In Gaussian latent-variable language, thresholds are calibrated via:

$$
C_i(T)=\Phi^{-1}(1-Q_i(T))
$$

i.e., default by $T$ when the latent variable falls below $C_i(T)$.

---

### Step 2: Specify Dependence Across Names

**Independence Baseline:**

Independence copula:

$$
C(u_1,\dots,u_N)=\prod_{i=1}^N u_i
$$

**One-Factor Copula / Latent-Variable Models:**

Gaussian copula / one-factor Gaussian latent variable model is the baseline "market language" (see next section). The text notes Li (2000) as early application and stresses equivalence of one-factor Gaussian copula and one-factor Gaussian latent variable.

**t-Copula / Other Copulas:**

Sources do cover that alternative copulas can generate tail dependence:
- QRM contrasts Gaussian copula (no tail dependence) with t-copula (tail dependence)
- Hull notes Student's t-copula can better describe joint extreme behavior than Gaussian in some contexts

We keep this as a **brief mention** here; Gaussian remains the baseline for implied correlation language.

---

### Step 3: Generate/Approximate Portfolio Loss Distribution $L(t)$

Under one-factor Gaussian latent-variable ideas, conditional on the factor, names are independent and (for homogeneous assumptions) conditional loss can be approximated as binomial, with conditional expected loss $\mathbb{E}[L(T)\mid Z]=(1-R)p(T\mid Z)$.

---

### Step 4: Map $L(t)\rightarrow L(t;A,D)\rightarrow \mathrm{ETL}(t)\rightarrow$ PV and Tranche Spread

Use the tranche loss mapping:

$$
L(t;A,D)=\frac{\min(L(t),D)-\min(L(t),A)}{D-A}
$$

Compute ETL:

$$
\mathrm{ETL}_{A,D}(t)=\mathbb{E}[L(t;A,D)]
$$

Convert ETL to tranche survival curve $Q(t;A,D)=\mathbb{E}[1-L(t;A,D)]$.

Then premium/protection legs follow CDS-like formulas using $Q(\cdot;A,D)$ (with the mapping noted in the text).

---

## 4. Gaussian Copula Framework (High-Level but Mathematically Explicit)

### 4.1 One-Factor Gaussian Latent Variable Construction

A standard one-factor representation (aligned with the sources) is:

$$
Z_i=\sqrt{\rho}\,X+\sqrt{1-\rho}\,\varepsilon_i,
\quad X\sim N(0,1),\ \varepsilon_i\sim N(0,1)\ \text{i.i.d., independent of }X
$$

This factor copula form is shown in Hull's treatment (with $U_i$ notation).

**Interpretation:**
- $X$: common/systematic "state of the world" factor
- $\varepsilon_i$: idiosyncratic shock for name $i$
- $\rho$: pairwise correlation between $Z_i$ and $Z_j$ in this symmetric one-factor setting

---

### 4.2 Default Event via Threshold Determined by Marginal Default Probability

Let the marginal (risk-neutral) default probability by horizon $T$ be:

$$
\mathrm{PD}_i(T)=\Pr(\tau_i\le T)=1-Q_i(T)
$$

Choose threshold:

$$
a_i(T)=\Phi^{-1}(\mathrm{PD}_i(T))
$$

Then define default by $T$ as the event:

$$
\tau_i\le T \iff Z_i \le a_i(T)
$$

This matches the book's statement that default occurs if the latent variable falls below a threshold calibrated to survival probabilities, expressed there as $C_i(T)=\Phi^{-1}(1-Q_i(T))$.

---

### 4.3 Conditional Default Probability Given the Factor

Condition on $X=x$. Then $Z_i\mid X=x\sim N(\sqrt{\rho}x,\,1-\rho)$, so:

$$
\Pr(\tau_i\le T\mid X=x)
=\Pr(Z_i\le a_i(T)\mid X=x)
=\Phi\!\left(\frac{a_i(T)-\sqrt{\rho}\,x}{\sqrt{1-\rho}}\right)
$$

This is the standard conditional probability form (the book writes an equivalent form with $\beta_i$ as factor loading).

---

### 4.4 Sanity Checks and Limiting Cases

**$\rho=0$:**

$$
Z_i=\varepsilon_i,\quad \Pr(\tau_i\le T\mid X=x)=\Phi(a_i(T))=\mathrm{PD}_i(T)
$$

and defaults are (conditionally and unconditionally) independent.

**$\rho\to 1$:**

$$
Z_i\approx X \quad (\text{comonotonic limit})
$$

so names default together in extreme clustering (all driven by the single factor).

**Unit Check:**
- $Z_i,X,\varepsilon_i,a_i$ are all dimensionless standard-normal variables
- $\rho\in[0,1]$ dimensionless

---

### 4.5 Multi-Factor Variants

The provided snippets explicitly develop one-factor factor models for copulas. A full multi-factor Gaussian copula discussion is not in the snippets we used.

**I'm not sure** how deeply the attached credit-derivatives text develops multi-factor tranche calibration in the same implied-correlation market language (vs one-factor). To be certain, we would need: the book's explicit multi-factor specification (loadings), and how implied "correlation" is quoted/aggregated in that setting.

---

## 5. Market Correlation Language: Compound Correlation vs Base Correlation

This section is a **core deliverable**: how the market turns tranche prices into "correlations."

### 5.1 Compound Correlation (A Single $\rho$ for One Tranche)

**Definition:**

Compound correlation is the **flat correlation** $\rho$ implied by the price of a given tranche using the Gaussian copula model: solve for $\rho$ such that the tranche PV at initiation is zero.

In the text:

$$
PV(A,D,\rho)=0
$$

is solved by a one-dimensional root search.

**Practical Meaning:**
- It is tranche-specific: $\rho=\rho(A,D)$
- It is a convenient *diagnostic* ("what correlation does this tranche price correspond to in a one-factor Gaussian model?")

**Known Issue (From the Text):**

For mezzanine tranches, PV as a function of $\rho$ can be non-monotonic, leading to multiple solutions for compound correlation.

---

### 5.2 Base Correlation (Correlation by Strike / Capital-Structure "Skew")

**Definition:**

Base correlation assigns a correlation $\rho(0,K)\equiv \rho(K)$ to each **equity/base tranche** $[0,K]$.

Any tranche $[A,D]$ is decomposed into a linear combination of two base tranches, and **two base correlations** are used:

$$
\mathbb{E}[L(T;A,D)]
=\frac{\mathbb{E}_{\rho(D)}[\min(L(T),D)]-\mathbb{E}_{\rho(A)}[\min(L(T),A)]}{D-A}
$$

**Intuition:**

The market observes that a single flat $\rho$ cannot match all tranches at once ("correlation smile/skew"). Base correlation is a way to create a **term structure across attachment points**: $\rho(K)$ varies with $K$. The text notes the base correlation curve is generally termed a "skew."

**Why It Exists:**

The text discusses that compound correlation fails key properties in a smile world (e.g., conservation of expected loss across capital structure). Base correlation, while still not fully arbitrage-free, has practical advantages and became preferred.

**Pros (Source-Backed):**

- **Bootstrap feasibility:** standard tranches are contiguous, enabling a bootstrap up the capital structure from the equity tranche
- **Uniqueness/monotonicity for base-tranche steps:** in the bootstrap, because the changing part is the PV of a base tranche, the PV can be monotone and yield a unique solution (when it exists)

**Cons / Pathologies (Source-Backed at High Level):**

Interpolation can generate arbitrage-like behavior in tranchelets:
- Tranchelet spreads not monotonically decreasing with subordination
- Even negative tranchelet spreads in some strike ranges in an example
- Implied loss density can show spikes and even negative regions depending on interpolation
- Extrapolation below the lowest quoted strike (often 3%) is ambiguous and can materially affect thin equity tranchelet pricing

---

## 6. Calibration Workflow (Conceptual + Transparent Numerics)

### 6.1 Inputs

- **Portfolio definition:** constituents, weights/notionals, maturity, credit-event definitions (standard index vs bespoke)
- **Single-name marginals:** survival curves $Q_i(t)$ or hazards; calibration instruments and conventions
- **Recoveries:** fixed $R_i$ or recovery model; $L_{\max}$ implied
- **Dependence model choice:** Gaussian copula one-factor (baseline), parameterization of $\rho$ or $\beta$
- **Tranche contract:** $[A,D]$, coupon schedule, accrual convention, upfront vs running terms
- **Discount curve** $Z(t)$ and settlement assumptions for default timing
- **Market tranche quotes** (spread and/or upfront)
  - In one historical table, equity tranche quotes included an upfront plus a running spread (paid quarterly Act/360), while more senior tranches had running spreads without upfront
  - **Important:** These are examples; **verify current quoting conventions** for the specific index/vintage

---

### 6.2 Objective

Choose correlation parameter(s) so that model PV matches market PV:

$$
PV_{\text{model}}(\text{corr params})=PV_{\text{market}}
$$

In standard "par" quoting, this often means setting $PV=0$ at initiation when contractual spread/upfront is set to market.

---

### 6.3 Inversion: "Implied Correlation"

**Compound Correlation Inversion:**

For each tranche $[A,D]$, solve $PV(A,D,\rho)=0$ for a single $\rho$. Beware non-monotonicity / multiple solutions for some tranches.

**Base Correlation Bootstrap Inversion:**

Use contiguity of standard tranches:
1. Solve $\rho(K_1)$ from equity tranche $[0,K_1]$
2. Then solve $\rho(K_2)$ from tranche $[K_1,K_2]$ holding $\rho(K_1)$ fixed, etc.

Conceptually: each step is a one-dimensional root search in the next node $\rho(K_m)$.

---

### 6.4 Quote-Type Caveat

The book provides concrete examples of upfront + running for equity and running-only for other tranches in a specific historical dataset.

**I'm not sure** what the exact quote package is for *your* index/vintage (e.g., standard coupon + upfront vs spread-only, front-end protection conventions, etc.). To be certain we'd need: the exact index tranche quoting convention, contractual coupon conventions, and settlement rules used by your market.

---

## 7. Measurement & Risk (Only What Belongs in Chapter 50)

### 7.1 Correlation Risk (Framework-Level)

Correlation risk is the sensitivity of tranche PV to changes in the correlation parameter(s) used in the dependence model.

The text gives an example of "correlation 01" (PV change for a 1% change in correlation) with opposite signs across the capital structure:
- Correlation 01 of the $0\!-\!3\%$ tranche is negative (loss for a 1% increase in correlation)
- Correlation 01 of the $15\!-\!30\%$ tranche is positive

**Interpretation:**

Sign depends on:
- Tranche attachment/detachment
- Whether you are **long or short protection**
- And whether you are using compound or base correlation as your "bump" parameterization

---

### 7.2 "Who Is Long Correlation?" (Qualitative, Position-Aware)

From the perspective of a **short protection** position:
- Equity/base tranches can be **long correlation** (PV rises with correlation)
- Seniority can flip the sign (as the correlation 01 example indicates across equity vs senior)

For a **long protection** position, signs reverse.

---

### 7.3 Model Risk (Dependence, Recovery, Marginal Curves)

**Copula Choice:**

Gaussian copula has no tail dependence; t-copula can create tail dependence. This matters because senior tranches are tail-driven.

**Recovery Assumptions:**

Changing $R$ changes $L_{\max}$ and the mapping from default counts to losses; implied correlations can shift materially (see Worked Example 10).

**Marginal Curves:**

If single-name curves are inconsistent with tranche quotes, "implied correlation" may become unstable or nonsensical (e.g., no solution).

---

### 7.4 Hedging Preview Only (No Recommendations)

Tranche exposures can be decomposed into:
- Systemic spread risk (portfolio-wide spread moves)
- Idiosyncratic/default-event risk
- Correlation-parameter risk (smile/skew risk)

(The text discusses systemic measures and correlation 01 as part of tranche risk measurement.)

---

## 8. Math and Derivations (Step-by-Step)

### 8.1 Tranche Loss Mapping Reminder

Start from the identity (for $0\le A<D\le 1$):

$$
(D-A)L(T;A,D)=\min(L(T),D)-\min(L(T),A)
$$

Divide by $D-A$ to obtain:

$$
L(T;A,D)=\frac{\min(L(T),D)-\min(L(T),A)}{D-A}
$$

**Unit check:** numerator and denominator are both fractions of portfolio notional; ratio is dimensionless $\in[0,1]$.

---

### 8.2 ETL and Its Role in PV (Brief)

Define tranche survival curve:

$$
Q(t;A,D)=\mathbb{E}[1-L(t;A,D)]
$$

**Premium leg PV** (approximation using average outstanding notional over accrual period):

$$
PV_{\text{prem}}
=\frac{S(A,D)}{2}\sum_{i=1}^{N_T}\Delta(t_{i-1},t_i)\,Z(t_i)\big(Q(t_{i-1};A,D)+Q(t_i;A,D)\big)
$$

**Protection leg PV:**

$$
PV_{\text{prot}}=\int_0^T Z(s)\,(-dQ(s;A,D))
$$

These mirror CDS pricing once tranche $Q$ is known.

---

### 8.3 Gaussian Copula One-Factor Construction and Thresholds

**Latent variable:**

$$
Z_i=\sqrt{\rho}\,X+\sqrt{1-\rho}\,\varepsilon_i
$$

**Default by $T$ if $Z_i\le a_i(T)$**, where $a_i(T)=\Phi^{-1}(\mathrm{PD}_i(T))$ and $\mathrm{PD}_i(T)=1-Q_i(T)$.

**Conditional default probability:**

$$
p_i(T\mid X=x)=\Phi\!\left(\frac{a_i(T)-\sqrt{\rho}\,x}{\sqrt{1-\rho}}\right)
$$

---

### 8.4 "Implied Correlation Solve" Problem Statement

Given market quote(s) defining $PV_{\text{market}}$ for tranche $[A,D]$, define:

$$
f(\rho)=PV_{\text{model}}(A,D;\rho)-PV_{\text{market}}
$$

The implied (compound) correlation solves:

$$
\boxed{\ \rho^\star\ \text{s.t.}\ f(\rho^\star)=0\ }
$$

This is exactly the one-dimensional root search framing for compound correlation.

For base correlation, one solves sequentially for $\rho(K_m)$ via a bootstrap using contiguous tranches.

---

## 9. Worked Examples (12 Numeric Examples, Fully Numeric)

> **Reminder:** Examples below are **toy** calculations meant to make the correlation/tranche-loss mapping concrete. When a pricing shortcut deviates from the book's full cashflow formula, it is explicitly labeled.

---

### Example 1: Two-State Dependence Toy (2 Names, Same Marginal PD): Independence vs Perfect Correlation

Let two names $A,B$ each have 1y default probability $p=10\%=0.10$.

**Case 1: Independent Defaults**
- $\Pr(0\ \text{defaults})=(1-p)^2=0.9^2=0.81$
- $\Pr(1\ \text{default})=2p(1-p)=2(0.1)(0.9)=0.18$
- $\Pr(2\ \text{defaults})=p^2=0.1^2=0.01$

**Case 2: Perfectly Positively Dependent (Comonotonic) Defaults**

To preserve marginal $\Pr(\text{default})=0.10$, enforce "default together":
- $\Pr(2\ \text{defaults})=0.10$
- $\Pr(0\ \text{defaults})=0.90$
- $\Pr(1\ \text{default})=0$

**Key Takeaway:** Dependence changes the probability of *clusters* (2 defaults) from $1\%$ to $10\%$ while keeping each name's PD fixed.

---

### Example 2: Tranche Sensitivity Intuition via ETL Under Low vs High Dependence

We create two **portfolio loss** distributions (loss $L$ as % of portfolio notional). Compute ETL for equity $[0,3\%]$ and senior $[10,15\%]$.

**Define tranche loss fraction:**

$$
L_{\text{tr}}(L;A,D)=\frac{\min(L,D)-\min(L,A)}{D-A}
$$

#### Case "Low Dependence" (More Mass in the Middle)

| Outcome | $L$ | Probability |
|---------|-----|-------------|
| Low loss | 0% | 0.05 |
| Moderate loss | 8% | 0.90 |
| High loss | 40% | 0.05 |

#### Case "High Dependence" (More Mass at Extremes, Same Mean)

| Outcome | $L$ | Probability |
|---------|-----|-------------|
| Low loss | 0% | 0.30 |
| Moderate loss | 8% | 0.5875 |
| High loss | 40% | 0.1125 |

**Check Means:**
- Low: $0\cdot0.05+0.08\cdot0.90+0.40\cdot0.05=0.072+0.020=0.092$ (9.2%)
- High: $0\cdot0.30+0.08\cdot0.5875+0.40\cdot0.1125=0.0470+0.0450=0.092$ (9.2%)

#### ETL for Equity Tranche $[0,3\%]$, Width $w=0.03$

For $A=0$, $L_{\text{tr}}=\min(L,0.03)/0.03$.

- If $L=0\%$: $L_{\text{tr}}=0$
- If $L=8\%$: $\min(0.08,0.03)/0.03=0.03/0.03=1$
- If $L=40\%$: also $1$

Results:
- Low dep: $\mathrm{ETL}=0.05\cdot0+0.95\cdot1=0.95$
- High dep: $\mathrm{ETL}=0.30\cdot0+0.70\cdot1=0.70$

**Equity ETL decreases** when probability mass moves to "no loss" extreme.

#### ETL for Senior Tranche $[10,15\%]$, Width $w=0.05$

Compute $L_{\text{tr}}=\frac{\min(L,0.15)-\min(L,0.10)}{0.05}$.

- If $L=0\%$ or $8\%$ (<10%): $\min(L,0.15)=\min(L,0.10)=L\Rightarrow L_{\text{tr}}=0$
- If $L=40\%$ (≥15%): $\min(L,0.15)=0.15,\ \min(L,0.10)=0.10\Rightarrow L_{\text{tr}}=(0.05)/0.05=1$

Results:
- Low dep: $\mathrm{ETL}=0.05\cdot 1=0.05$
- High dep: $\mathrm{ETL}=0.1125\cdot 1=0.1125$

**Senior ETL increases** because "tail loss" probability increased.

---

### Example 3: Limiting-Case Toy — Probability of "Many Defaults" Rises with Stronger Systemic Dependence

Let $N=5$ names, each with marginal PD $p=2\%$ over 1y.

**Case A (Low Dependence / Effectively Independent):** $K\sim \mathrm{Binomial}(5,0.02)$

- $P(K=0)=0.98^5$
  - $0.98^2=0.9604$
  - $0.98^4=0.9604^2=0.92236816$
  - $0.98^5=0.92236816\cdot0.98=0.90392080$
- $P(K=1)=\binom{5}{1}(0.02)(0.98^4)=5\cdot0.02\cdot0.92236816=0.09223682$
- Therefore $P(K\ge2)=1-0.90392080-0.09223682=0.00384238$ (≈0.38%)

**Case B (High Systemic Dependence via 2-State Mixture; Toy Proxy for Higher $\rho$):**

- Good state (prob 90%): default probability $p_g=0.2\%=0.002$
- Bad state (prob 10%): default probability $p_b=18.2\%=0.182$
- Check marginal PD: $0.9(0.002)+0.1(0.182)=0.0018+0.0182=0.0200=2\%$

Compute $P(K\ge2)$ in each state and mix:

**Good state** $p=0.002$:
- $P_0=0.998^5\approx 0.9900$
- $P_1=5(0.002)(0.998^4)\approx 0.00992$
- $P(K\ge2)\approx 0.00008$

**Bad state** $p=0.182$, $q=0.818$:
- $P_0=q^5=0.818^5\approx 0.366$
- $P_1=5pq^4\approx 0.408$
- $P(K\ge2)=1-0.366-0.408=0.226$

**Mix:**

$$
P(K\ge2)\approx 0.9(0.00008)+0.1(0.226)=0.000072+0.0226=0.0227
$$

**Result:** $P(K\ge2)$ jumps from ≈0.38% (independent) to ≈2.27% (high systemic clustering), even with the same marginal PD.

---

### Example 4: One-Factor Gaussian Copula Threshold Mapping — $\mathrm{PD}=2\%$ $\Rightarrow a=\Phi^{-1}(0.02)$

Compute:

$$
a=\Phi^{-1}(0.02)\approx -2.0537
$$

(Meaning: the 2% left-tail quantile of a standard normal.)

**Sanity check:** $\Phi(-2.05)\approx 0.0202$ so the magnitude is consistent.

---

### Example 5: Conditional Default Probability Under One-Factor Gaussian Copula

**Parameters:**
- Marginal 1y PD $=2\%\Rightarrow a\approx -2.0537$
- $\rho=0.20\Rightarrow \sqrt{\rho}=0.4472,\ \sqrt{1-\rho}=0.8944$

**Formula:**

$$
p(x)=\Pr(\text{default}\mid X=x)=\Phi\!\left(\frac{a-\sqrt{\rho}\,x}{\sqrt{1-\rho}}\right)
$$

**Case 1: Bad Factor Realization $x=-1$**
- Numerator: $a-\sqrt{\rho}x=a-0.4472(-1)=a+0.4472=-2.0537+0.4472=-1.6065$
- Divide: $-1.6065/0.8944=-1.796$
- $p(-1)=\Phi(-1.796)\approx 0.036$ (3.6%)

**Case 2: Good Factor Realization $x=+1$**
- Numerator: $a-0.4472(1)=-2.0537-0.4472=-2.5009$
- Divide: $-2.5009/0.8944=-2.796$
- $p(1)=\Phi(-2.796)\approx 0.0026$ (0.26%)

**Interpretation:** Bad systemic states multiply default probability relative to marginal.

---

### Example 6: Approximate Portfolio Default-Count Distribution via Conditional Mixture (Toy, Small $N$)

Let $N=5$, marginal PD $=2\%$ ($a=-2.0537$), $\rho=0.20$.

Use a **3-point factor approximation** (toy quadrature): $x\in\{-1.5,0,+1.5\}$ with weights $(0.25,0.50,0.25)$.

**Constants:** $\sqrt{\rho}=0.4472,\ \sqrt{1-\rho}=0.8944$

**Compute conditional PDs:**

| $x$ | $(a-\sqrt{\rho}x)/\sqrt{1-\rho}$ | $p(x)=\Phi(\cdot)$ |
|-----|----------------------------------|-------------------|
| $-1.5$ | $(-2.0537+0.6708)/0.8944=-1.5459$ | $0.0612$ |
| $0$ | $-2.0537/0.8944=-2.296$ | $0.0108$ |
| $+1.5$ | $(-2.0537-0.6708)/0.8944=-3.0456$ | $0.00116$ |

Now conditional on each $x$, default count $K\mid x \sim \mathrm{Binomial}(5,p(x))$.

**At $x=-1.5$, $p=0.0612,\ q=0.9388$:**
- $P_0=q^5\approx 0.7285$
- $P_1=5pq^4\approx 0.2377$
- $P_2=10p^2q^3\approx 0.0310$
- $P_3\approx 0.00202$
- $P_4\approx 0.0000658$
- $P_5\approx 0.00000086$

**At $x=0$, $p=0.0108,\ q=0.9892$:**
- $P_0\approx 0.9473$
- $P_1\approx 0.0517$
- $P_2\approx 0.00113$
- $P_3\approx 0.0000123$
- Higher orders negligible

**At $x=1.5$, $p=0.00116,\ q=0.99884$:**
- $P_0\approx 0.99420$
- $P_1\approx 0.00577$
- $P_2\approx 0.0000134$
- Higher orders negligible

**Now mix:**

$$
P(K=k)\approx 0.25P_k(-1.5)+0.5P_k(0)+0.25P_k(1.5)
$$

| $K$ | Mixture $P(K=k)$ |
|-----|-----------------|
| 0 | $0.25(0.7285)+0.5(0.9473)+0.25(0.9942)=0.9044$ |
| 1 | $0.25(0.2377)+0.5(0.0517)+0.25(0.00577)=0.0867$ |
| 2 | $0.25(0.0310)+0.5(0.00113)+0.25(0.0000134)=0.00831$ |
| 3 | $0.25(0.00202)+0.5(0.0000123)+0.25(0)=0.000510$ |

**Interpretation:** Unconditional distribution has a fatter right tail than a simple binomial with $p=2\%$, reflecting systemic mixing.

---

### Example 7: Implied (Compound) Correlation Solve (Toy Bisection) — Using a Simplified PV Proxy

We demonstrate a root-search concept consistent with "solve $PV(\rho)=0$" for implied correlation.

**Important Simplifications (Explicit):**
- We use a **single-payment-at-maturity** PV proxy rather than the book's full coupon schedule integrals. **I'm not sure** this proxy matches any specific index tranche quoting convention; it's purely illustrative.
- We use an **LHP-style** approximation: conditional portfolio loss is deterministic given factor $X=x$. (The text discusses large homogeneous portfolio limits where conditional loss variance shrinks with $N$.)

**Setup:**
- Horizon $T=5$ years
- Homogeneous marginal default probability to $T$: $p=10\%\Rightarrow a=\Phi^{-1}(0.10)\approx -1.2816$
- Recovery $R=40\%\Rightarrow \mathrm{LGD}=0.60$
- Equity tranche $[0,K]$ with $K=3\%=0.03$
- Market running spread (toy) $S_{\text{mkt}}=2000$ bp $=0.20$ per year, paid once at $T$ on expected outstanding tranche notional

**PV Proxy (Short Protection):**
- Protection leg PV proxy: $PV_{\text{prot}}=Z(T)\cdot \mathrm{ETL}(\rho)$
- Premium leg PV proxy: $PV_{\text{prem}}=Z(T)\cdot S_{\text{mkt}}T\cdot (1-\mathrm{ETL}(\rho))$
- Set $PV_{\text{prem}}-PV_{\text{prot}}=0$:

$$
S_{\text{mkt}}T(1-\mathrm{ETL})-\mathrm{ETL}=0
\Rightarrow \mathrm{ETL}_{\text{target}}=\frac{S_{\text{mkt}}T}{1+S_{\text{mkt}}T}
$$

Here $S_{\text{mkt}}T=0.20\cdot5=1\Rightarrow \mathrm{ETL}_{\text{target}}=1/2=0.50$.

**Compute $\mathrm{ETL}(\rho)$ Approximately:**
- Use factor points $x\in\{-1.5,0,+1.5\}$ with weights $(0.25,0.50,0.25)$
- Conditional default probability: $p(x)=\Phi\!\left(\frac{a-\sqrt{\rho}\,x}{\sqrt{1-\rho}}\right)$
- Conditional portfolio loss (LHP proxy): $L(x)=\mathrm{LGD}\cdot p(x)=0.60p(x)$
- Equity tranche loss fraction: $\mathrm{TL}(x)=\min(L(x),0.03)/0.03$
- Approximate ETL: $\mathrm{ETL}(\rho)\approx \sum w_j \mathrm{TL}(x_j)$

**Step 1: Evaluate at $\rho=0.50$**

$\sqrt{\rho}=0.7071,\ \sqrt{1-\rho}=0.7071$

| $x$ | Ratio | $p$ | $L=0.60p$ | $\mathrm{TL}$ |
|-----|-------|-----|-----------|---------------|
| $-1.5$ | $(a+1.0607)/0.7071=-0.3125$ | $0.377$ | $0.226$ | $1$ |
| $0$ | $a/0.7071=-1.812$ | $0.0350$ | $0.0210$ | $0.700$ |
| $+1.5$ | $(a-1.0607)/0.7071=-3.312$ | $0.00046$ | $0.000276$ | $0.0092$ |

ETL: $\mathrm{ETL}(0.50)\approx 0.25(1)+0.5(0.700)+0.25(0.0092)=0.6023$

Define $f(\rho)=\mathrm{ETL}(\rho)-0.50$:
- $f(0.50)=0.6023-0.50=+0.1023$ (too much loss; need higher $\rho$ to reduce equity ETL)

**Step 2: Evaluate at $\rho=0.80$**

$\sqrt{\rho}=0.8944,\ \sqrt{1-\rho}=0.4472$

| $x$ | Ratio | $p$ | $L$ | $\mathrm{TL}$ |
|-----|-------|-----|-----|---------------|
| $-1.5$ | $(a+1.3416)/0.4472=0.134$ | $0.553$ | $0.332$ | $1$ |
| $0$ | $a/0.4472=-2.866$ | $0.00208$ | $0.00125$ | $0.0416$ |
| $+1.5$ | $\approx -5.867$ | $\approx 0$ | $\approx 0$ | $\approx 0$ |

ETL: $\mathrm{ETL}(0.80)\approx 0.25(1)+0.5(0.0416)+0.25(0)=0.2708$

So $f(0.80)=0.2708-0.50=-0.2292$ (too little loss; need lower $\rho$).

**Bisection Interval:** Root lies between $\rho=0.50$ and $\rho=0.80$.

**Step 3: Midpoint $\rho=0.65$**

$\sqrt{\rho}=0.8062,\ \sqrt{1-\rho}=0.5916$

| $x$ | Ratio | $p$ | $L$ | $\mathrm{TL}$ |
|-----|-------|-----|-----|---------------|
| $-1.5$ | $(a+1.2093)/0.5916=-0.122$ | $0.451$ | $0.271$ | $1$ |
| $0$ | $-2.166$ | $0.0152$ | $0.00912$ | $0.304$ |
| $+1.5$ | $-4.211$ | $0.0000124$ | $\approx 0$ | $\approx 0.00025$ |

ETL: $\mathrm{ETL}(0.65)\approx 0.25(1)+0.5(0.304)+0.25(0.00025)=0.4021$

So $f(0.65)=0.4021-0.50=-0.0979$. Root between $0.50$ and $0.65$.

**Step 4: Midpoint $\rho=0.575$**

$\sqrt{\rho}=0.7583,\ \sqrt{1-\rho}=0.6519$

| $x$ | Ratio | $p$ | $L$ | $\mathrm{TL}$ |
|-----|-------|-----|-----|---------------|
| $-1.5$ | $-0.221$ | $0.4125$ | $0.2475$ | $1$ |
| $0$ | $-1.966$ | $0.0247$ | $0.0148$ | $0.494$ |
| $+1.5$ | $-3.711$ | $0.000102$ | $0.000061$ | $0.00204$ |

ETL: $\mathrm{ETL}(0.575)\approx 0.25(1)+0.5(0.494)+0.25(0.00204)=0.4975$

So $f(0.575)=0.4975-0.50=-0.0025$ (very close).

**Toy Implied Correlation (Compound):**

$$
\boxed{\rho^\star \approx 0.58}
$$

under these approximations.

---

### Example 8: Base Correlation Idea via "Difference of Equities" (Source-Backed Identity)

Suppose we have (toy) base correlations:
- $\rho(3\%)=20\%$
- $\rho(7\%)=30\%$

The base-correlation method prices tranche $[3\%,7\%]$ via two equity/base tranche ETLs:

$$
\mathbb{E}[L(T;3\%,7\%)]
=\frac{\mathbb{E}_{\rho(7\%)}[\min(L(T),7\%)]-\mathbb{E}_{\rho(3\%)}[\min(L(T),3\%)]}{7\%-3\%}
$$

**Numerical Illustration (Toy ETLs):**

Assume (just to illustrate arithmetic):
- $\mathbb{E}_{\rho(3\%)}[\min(L,3\%)]=2.4\%$
- $\mathbb{E}_{\rho(7\%)}[\min(L,7\%)]=4.9\%$

Then:
- Numerator $=4.9\%-2.4\%=2.5\%$
- Denominator $=4\%$
- $\mathbb{E}[L(T;3,7)]=2.5/4=0.625$

So expected **tranche loss fraction** is $62.5\%$ of tranche notional.

**Unit Check:** Numerator and denominator are both in "% of portfolio notional," ratio is dimensionless.

---

### Example 9: Correlation Smile / Skew Illustration (Toy)

A toy implied base correlation "skew" by detachment $K$:

| Detachment $K$ | Base Correlation $\rho(K)$ |
|----------------|---------------------------|
| 3% | 20% |
| 7% | 30% |
| 10% | 40% |
| 15% | 55% |
| 30% | 70% |

**Interpretation:**

Increasing $\rho(K)$ with $K$ suggests the market needs "more dependence strength" (in a Gaussian one-factor sense) to match the pricing of more senior risk. The text notes base correlation is typically not flat and is termed a "skew."

---

### Example 10: Recovery Sensitivity Interaction (Toy) — How Implied Correlation Shifts When Recovery Changes

We reuse Example 7 setup but change recovery.

**Recall Example 7:**
- Target equity ETL = 0.50 (from toy PV proxy)
- With $R=40\%$ (LGD=0.60), we found $\rho^\star \approx 0.575$

**Now set higher recovery** $R=50\%\Rightarrow \mathrm{LGD}=0.50$, keep everything else fixed (same marginal PD curve).

Compute ETL at $\rho=0.575$ using the same factor points and conditional PDs from Example 7:

At $\rho=0.575$, we had $p(-1.5)=0.4125,\ p(0)=0.0247,\ p(1.5)=0.000102$.

Portfolio loss $L(x)=0.50p(x)$:

| $x$ | $L$ | $\mathrm{TL}$ |
|-----|-----|---------------|
| $-1.5$ | $0.2063$ | $1$ |
| $0$ | $0.01235$ | $0.01235/0.03=0.412$ |
| $+1.5$ | $0.000051$ | $0.0017$ |

ETL: $\mathrm{ETL}(0.575)\approx 0.25(1)+0.5(0.412)+0.25(0.0017)=0.4564$

This is **below** the target 0.50, so to increase ETL we must move $\rho$ downward (because for equity, higher $\rho$ tends to reduce ETL by creating more "no loss" mass).

**Try $\rho=0.54$ (Quick Check):**

$\sqrt{\rho}=0.7348,\ \sqrt{1-\rho}=0.6782$, $a=-1.2816$

| $x$ | Ratio | $p$ | $L$ | $\mathrm{TL}$ |
|-----|-------|-----|-----|---------------|
| $-1.5$ | $-0.2646$ | $0.3957$ | $0.1979$ | $1$ |
| $0$ | $-1.889$ | $0.0294$ | $0.0147$ | $0.490$ |
| $+1.5$ | $-3.515$ | $0.000219$ | $0.0001095$ | $0.00365$ |

ETL: $\mathrm{ETL}(0.54)\approx 0.25(1)+0.5(0.490)+0.25(0.00365)=0.4959\approx 0.50$

**Toy Conclusion:**

$$
R\uparrow\ (LGD\downarrow)\ \Rightarrow\ \rho^\star \downarrow\ \text{(for this equity tranche under this proxy)}
$$

---

### Example 11: Nonlinearity by Tranche — Same Dependence Change Impacts Senior Much More Than Mezz (Toy Numbers)

Use two portfolio loss distributions representing "low corr" vs "high corr" (not derived from a full copula here—purely illustrative distribution shift).

**Low Dependence Distribution:**

| $L$ | Probability |
|-----|-------------|
| 0% | 0.10 |
| 12% | 0.89 |
| 40% | 0.01 |

**High Dependence Distribution:**

| $L$ | Probability |
|-----|-------------|
| 0% | 0.30 |
| 12% | 0.50 |
| 40% | 0.20 |

Consider two tranches:
1. Mezzanine $[10,15]$, width $0.05$
2. Super-senior $[15,30]$, width $0.15$

#### Mezzanine $[10,15]$

Compute $L_{\text{tr}}=(\min(L,0.15)-\min(L,0.10))/0.05$.

| $L$ | $L_{\text{tr}}$ |
|-----|-----------------|
| $12\%$ | $(0.12-0.10)/0.05=0.4$ |
| $40\%$ | $(0.15-0.10)/0.05=1$ |
| $0\%$ | $0$ |

ETL:
- Low dep: $0.89(0.4)+0.01(1)=0.356+0.01=0.366$
- High dep: $0.50(0.4)+0.20(1)=0.20+0.20=0.40$

**Change:** $\Delta \mathrm{ETL}=+0.034$

#### Super-Senior $[15,30]$

Compute $L_{\text{tr}}=(\min(L,0.30)-\min(L,0.15))/0.15$.

| $L$ | $L_{\text{tr}}$ |
|-----|-----------------|
| $12\%$ | $0$ (below 15%) |
| $40\%$ | $(0.30-0.15)/0.15=1$ |
| $0\%$ | $0$ |

ETL:
- Low dep: $0.01(1)=0.01$
- High dep: $0.20(1)=0.20$

**Change:** $\Delta \mathrm{ETL}=+0.19$

**Key Takeaway:** Tail-probability shifts barely move mezz ETL but can massively move senior ETL, consistent with "senior is tail-driven."

---

### Example 12: Sanity Checks — ETL Bounds and Detecting Inconsistent Tranche-Loss Mapping

#### A. ETL Bounds Check

For any tranche, by construction $0\le L(T;A,D)\le 1$, so:

$$
0\le \mathrm{ETL}_{A,D}(T)\le 1
$$

Check Example 11:
- Mezz ETL: 0.366 and 0.40 (both in $[0,1]$) ✓
- Super-senior ETL: 0.01 and 0.20 (both in $[0,1]$) ✓

#### B. Expected Loss Amount Bounds

Expected loss amount (as fraction of portfolio notional) for tranche $[A,D]$ is:

$$
\mathbb{E}[\min(L,D)-\min(L,A)] \in [0, D-A]
$$

Because $\min(L,D)-\min(L,A)$ is always between 0 and $D-A$.

Example 11, super-senior $[15,30]$: $D-A=0.15$.
- In high dep distribution, loss amount is $0.15$ only when $L=40\%$ (prob 0.20):
  $\mathbb{E}[\text{loss amount}]=0.20\cdot 0.15=0.03$, which is in $[0,0.15]$ ✓

#### C. Detecting a Wrong Mapping (Common Pitfall)

Suppose someone mistakenly uses:

$$
\widetilde{L}_{\text{tr}}(L;A,D)=\frac{L-A}{D-A}\quad\text{(without capping at 0 and 1)}
$$

Take $A=0.15,\ D=0.30,\ L=0.40$:
- $\widetilde{L}_{\text{tr}}=(0.40-0.15)/0.15=0.25/0.15=1.666$ (impossible: >1)

**How you detect it:** Any ETL computation yielding values outside $[0,1]$ is automatically inconsistent with tranche-loss definitions.

---

## 10. Practical Notes

### 10.1 Checklist of Inputs Needed (Copula Tranche Pricing / Calibration)

| Category | Inputs |
|----------|--------|
| **Portfolio definition** | Constituents, weights/notionals, maturity, credit-event definitions (standard index vs bespoke) |
| **Single-name marginals** | Survival curves $Q_i(t)$ or hazards; calibration instruments and conventions |
| **Recoveries** | Fixed $R_i$ or recovery model; $L_{\max}$ implied |
| **Dependence model choice** | Gaussian copula one-factor (baseline), parameterization of $\rho$ or $\beta$ |
| **Tranche contract** | $[A,D]$, coupon schedule, accrual convention, upfront vs running terms |
| **Discount curve** | $Z(t)$ and settlement assumptions for default timing |

---

### 10.2 Common Pitfalls

| Pitfall | Explanation |
|---------|-------------|
| **Confusing correlation with dependence** | Linear correlation is a summary statistic; copula encodes the dependence |
| **Treating "$\rho$" as universal** | Market-implied $\rho$ varies by tranche (smile/skew), motivating base correlation |
| **Mixing inconsistent inputs** | Single-name curves inconsistent with tranche quotes can produce unstable implied correlations or no solution |
| **Ignoring recovery and discounting** | Recovery changes $L_{\max}$ and loss mapping when inferring implied correlation |
| **Treating base correlation as structural truth** | Base correlation is a practical quoting framework; interpolation/extrapolation can create implied-density pathologies |

---

### 10.3 Verification Tests (Practical "Does This Implementation Make Sense?")

| Test | Description |
|------|-------------|
| **Repricing check** | Each calibration tranche should reprice to its market PV/quote |
| **ETL bounds** | $0\le \mathrm{ETL}\le 1$; loss amount in $[0,D-A]$ |
| **Monotonicity checks (strike dimension)** | Tranchelet spreads should generally decrease with more subordination; violations can indicate interpolation-arbitrage issues |
| **Limiting-case checks** | $\rho=0$ should approach independence behavior; $\rho\to 1$ should approach extreme clustering |
| **Stability** | Small quote bumps should not create absurd implied-correlation jumps (qualitative; often affected by interpolation choice) |

---

## 11. Summary & Recall

### 11.1 Executive Summary (10 Bullets)

1. Portfolio loss $L(t)$ is a fraction of portfolio notional lost to defaults; tranche loss is a nonlinear mapping of $L(t)$ with kinks at $A$ and $D$
2. Because tranche loss is nonlinear in $L$, tranche PV depends on the **full distribution** of portfolio loss, not just expected loss
3. "Dependence" is the joint default behavior; "correlation" is often just a compressed parameterization of a dependence model
4. Copulas separate marginals (from CDS curves) from joint structure; Sklar's theorem formalizes this separation
5. Gaussian one-factor copula uses $Z_i=\sqrt{\rho}X+\sqrt{1-\rho}\varepsilon_i$ and default thresholds from marginal PDs
6. Higher correlation tends to shift probability to extremes (few defaults vs many defaults), which affects ETL differently by tranche attachment
7. Tranche pricing can be expressed via tranche survival curve $Q(t;A,D)=\mathbb{E}[1-L(t;A,D)]$ and mapped to CDS-style PV formulas
8. Compound correlation is a flat $\rho$ implied by one tranche (solve $PV(\rho)=0$); it can have multiple solutions for mezz tranches
9. Base correlation assigns $\rho(K)$ to equity/base tranches $[0,K]$ and prices $[A,D]$ as a difference of two base tranches using $\rho(A)$ and $\rho(D)$
10. Base correlation is useful but can generate interpolation/extrapolation pathologies (non-monotone tranchelet spreads, negative implied density regions)

---

### 11.2 Cheat Sheet (Key Formulas)

**Tranche Loss Fraction:**

$$
L(T;A,D)=\frac{\min(L(T),D)-\min(L(T),A)}{D-A}
$$

**Equity/Base Tranche ETL:**

$$
\psi(T,K)=\mathbb{E}[\min(L(T),K)]
$$

**Gaussian One-Factor Latent Variable:**

$$
Z_i=\sqrt{\rho}\,X+\sqrt{1-\rho}\,\varepsilon_i
$$

**Threshold from Marginal PD:**

$$
a_i(T)=\Phi^{-1}(\mathrm{PD}_i(T)),\quad \tau_i\le T \iff Z_i\le a_i(T)
$$

**Conditional PD:**

$$
p_i(T\mid X=x)=\Phi\!\left(\frac{a_i(T)-\sqrt{\rho}\,x}{\sqrt{1-\rho}}\right)
$$

**Compound Correlation Implied Solve:**

$$
\rho^\star:\ PV(A,D,\rho^\star)=0
$$

**Base Correlation Tranche ETL Identity:**

$$
\mathbb{E}[L(T;A,D)]
=\frac{\mathbb{E}_{\rho(D)}[\min(L(T),D)]-\mathbb{E}_{\rho(A)}[\min(L(T),A)]}{D-A}
$$

---

### 11.3 Flashcards (30 Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What is $L(T)$? | Fractional portfolio loss (fraction of portfolio notional lost by $T$) |
| 2 | What is $L_{\max}$? | Maximum possible portfolio loss implied by recovery assumptions (e.g., $1-R$ if fixed recovery) |
| 3 | Define tranche loss $L(T;A,D)$ | $\frac{\min(L(T),D)-\min(L(T),A)}{D-A}$ |
| 4 | Why are tranches correlation-sensitive? | Because tranche loss is nonlinear in portfolio loss, so dependence reshapes loss distribution and ETL |
| 5 | What is Pearson correlation? | $\rho_P=\frac{\mathrm{Cov}(X,Y)}{\sigma_X\sigma_Y}$ |
| 6 | Why can Pearson correlation be misleading for defaults? | Defaults are Bernoulli; correlation bounds depend on marginals |
| 7 | What is a copula? | A multivariate CDF with uniform marginals linking marginals to joint distribution |
| 8 | State Sklar's theorem (informally) | Any joint CDF can be expressed as a copula applied to its marginals; unique if marginals continuous |
| 9 | What is a default indicator process? | $Y_{t,i}=1$ if name $i$ defaulted by $t$, else 0 |
| 10 | Write the one-factor Gaussian copula latent variable | $Z_i=\sqrt{\rho}X+\sqrt{1-\rho}\varepsilon_i$ |
| 11 | How do marginals determine thresholds? | $a_i(T)=\Phi^{-1}(\mathrm{PD}_i(T))$ with $\mathrm{PD}_i(T)=1-Q_i(T)$ |
| 12 | Conditional PD formula in Gaussian one-factor? | $\Phi\!\big(\frac{a_i(T)-\sqrt{\rho}x}{\sqrt{1-\rho}}\big)$ |
| 13 | Limiting case $\rho=0$? | Defaults independent; conditional PD equals marginal PD |
| 14 | Limiting case $\rho\to 1$? | Comonotonic extreme clustering (single factor drives all) |
| 15 | Define tranche survival curve $Q(t;A,D)$ | $\mathbb{E}[1-L(t;A,D)]$ |
| 16 | Protection leg PV in terms of $Q$? | $\int_0^T Z(s)(-dQ(s;A,D))$ |
| 17 | Premium leg PV uses what notion? | Expected outstanding tranche notional via $Q$ |
| 18 | What is compound correlation? | Flat $\rho$ that reprices a specific tranche in Gaussian copula |
| 19 | How is compound correlation found? | Solve $PV(A,D,\rho)=0$ via 1D root search |
| 20 | Why can compound correlation be problematic? | PV may be non-monotone → multiple solutions; also fails conservation of expected loss in smile world |
| 21 | What is base correlation? | Correlation $\rho(K)$ assigned to base tranche $[0,K]$ |
| 22 | How to price $[A,D]$ under base correlation? | Use $\rho(A)$ and $\rho(D)$ via difference-of-equities ETL identity |
| 23 | Why does base correlation exist? | Need strike-dependent dependence to fit tranche "skew/smile" |
| 24 | What is a key base-correlation challenge? | Interpolation can imply arbitrage-like tranchelet spreads or negative densities |
| 25 | What is tail dependence? | Tendency of variables to be jointly extreme; differs by copula choice |
| 26 | Which copula has no tail dependence in the sources? | Gaussian copula |
| 27 | Which copula can have tail dependence? | Student's t-copula (sources mention) |
| 28 | What is "correlation 01"? | PV change for a 1% correlation change (example shows sign differs by tranche) |
| 29 | Why verify quoting conventions? | Upfront vs running, coupons/day counts vary; examples are historical |
| 30 | What is the conceptual pipeline for tranche pricing? | Marginals → dependence model → portfolio loss distribution → tranche loss → ETL/$Q$ → PV/spread |

---

## 12. Mini Problem Set (18 Questions)

*(Brief solution sketches provided for questions 1–9 only.)*

**1)** Show that $L(T;A,D)\in[0,1]$ given the min-formula.

> **Sketch:** Since $\min(L,D)-\min(L,A)\in[0,D-A]$, divide by $D-A>0$.

**2)** Compute $L(T;A,D)$ for $A=3\%$, $D=7\%$ when $L(T)=2\%,5\%,9\%$.

> **Sketch:** Use $(\min(L,D)-\min(L,A))/(D-A)$; evaluate three cases (below $A$, between, above $D$).

**3)** For two names with PD $p$, compute distribution of default count under independence.

> **Sketch:** Binomial with $N=2$: $(1-p)^2, 2p(1-p), p^2$.

**4)** Under perfect positive dependence with marginal PD $p$, compute distribution of default count for two names.

> **Sketch:** Only 0 or 2 defaults: $P(2)=p$, $P(0)=1-p$.

**5)** Given PD $=1\%$, compute threshold $a=\Phi^{-1}(0.01)$ approximately.

> **Sketch:** Use known quantile $\Phi^{-1}(0.01)\approx -2.326$.

**6)** Derive the conditional PD formula $p(x)=\Phi((a-\sqrt{\rho}x)/\sqrt{1-\rho})$.

> **Sketch:** Condition on $X=x$, $Z_i\mid x\sim N(\sqrt{\rho}x,1-\rho)$; standardize.

**7)** Explain qualitatively why increasing $\rho$ can decrease equity ETL but increase senior ETL.

> **Sketch:** Higher $\rho$ moves mass to extremes: more $L\approx 0$ helps equity; more far-tail $L$ hurts seniors.

**8)** Define compound correlation and explain one reason it can fail as a smile-consistent model.

> **Sketch:** Flat $\rho$ per tranche from $PV(\rho)=0$; fails conservation of expected loss across structure.

**9)** Write the base-correlation "difference of equities" formula for $\mathbb{E}[L(T;A,D)]$.

> **Sketch:** $\frac{\mathbb{E}_{\rho(D)}[\min(L,D)]-\mathbb{E}_{\rho(A)}[\min(L,A)]}{D-A}$.

**10)** Explain how interpolation of base correlation can create negative tranchelet spreads.

**11)** Describe a verification test that checks "conservation of expected loss" across tranches.

**12)** For a Gaussian one-factor model, explain what happens to conditional PD as $x\to-\infty$ and $x\to+\infty$.

**13)** Describe how recovery assumptions affect $L_{\max}$ and why this impacts implied correlation.

**14)** Why might senior tranches be more sensitive to tail dependence than equity tranches?

**15)** In a base correlation bootstrap, why does contiguity of standard tranches matter?

**16)** Describe conceptually how you would compute "correlation 01" for a tranche under base correlation.

**17)** Explain why "correlation" is not a universal dependence descriptor across all marginals/copulas.

**18)** Give two reasons why Gaussian copula might be chosen in practice even if it lacks tail dependence.

---

## 13. Source Map

### (A) Verified Facts — Directly Supported by the Provided Sources

- Definition of copula and its role linking marginals to a joint distribution; Sklar's theorem representation and uniqueness (continuous marginals)
- Independence copula and dependence bounds discussion (independence formula shown)
- Pearson correlation definition and limitations for non-Gaussian marginals / default indicators
- Default indicator process $Y_{t,i}$ and default time $\tau_i$
- Fractional portfolio loss $L(T)$, max loss $L_{\max}$, and ETL $\psi(T,K)=\mathbb{E}[\min(L(T),K)]$
- Tranche loss identity via "min" operators and conservation of expected loss across capital structure
- Tranche pricing in terms of tranche survival curve $Q(t;A,D)$ and CDS-analogy; premium/protection leg PV formulas in terms of $Q$
- Gaussian factor copula form $U_i=\sqrt{\rho}F+\sqrt{1-\rho}Z_i$ (Hull) consistent with one-factor Gaussian latent variable
- Threshold calibration to marginals and conditional default probability form (credit-derivatives text)
- Definition of compound correlation as solving $PV(A,D,\rho)=0$ and possibility of multiple solutions
- Base correlation decomposition and tranche expected loss formula using two base correlations $\rho(A),\rho(D)$
- Base correlation bootstrap logic (contiguous tranches) and monotonic/unique-solution rationale in the text's bootstrap framing
- Pathologies from naive base-correlation interpolation (non-monotone tranchelet spreads, negative spreads, implied density spikes/negative regions) and extrapolation ambiguity below 3%
- Tail dependence contrast: Gaussian copula has no tail dependence; t-copula can have tail dependence (QRM; Hull mentions)

### (B) Reasoned Inference — Derived/Implicated by (A)

- The "mass to extremes" intuition is derived from one-factor conditioning and the tranche loss nonlinearity, using the conditional-independence structure and extreme factor limits
- The qualitative mapping "equity vs senior" sensitivity is derived by applying the tranche loss function to shifts in $\Pr(L\le A)$ and $\Pr(L\ge D)$, consistent with the tranche loss distribution discussion
- Worked examples compute numeric ETLs and toy distributions using the source-backed tranche loss mapping; they illustrate how ETL can move differently by tranche

### (C) Speculation (Clearly Labeled)

- **I'm not sure** the toy PV proxy in Example 7 reflects any specific tranche quoting convention; it is a didactic simplification (single payment at maturity) to demonstrate the root-search concept
- **I'm not sure** whether your desk's base correlation implementation uses the exact same interpolation/extrapolation procedure as the one described in the book's later sections; implementation details can vary (ETL-space interpolation, spline choices, etc.). To be certain, we would need your desk's specified interpolation and arbitrage-check framework

---

> *"Generated from the provided reference books; verify tranche market quoting and calibration conventions for your specific desk."*

---

## Open Questions for Discussion

1. Should the notes treat **base correlation** as the primary market quoting language, or present **compound correlation first** and base correlation as an extension?

2. Should later chapters include an alternative to Gaussian copula (e.g., **t-copula / factor intensity model**) if the sources support it, or stay Gaussian-copula-centric?
