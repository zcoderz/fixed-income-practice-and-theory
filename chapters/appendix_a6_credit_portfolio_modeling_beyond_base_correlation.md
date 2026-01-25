# Appendix A6: Credit Portfolio Modeling Beyond Base Correlation (Advanced)

---

## 0. Setup

### Conventions Used in This Appendix

- Time is continuous. Horizon times are denoted by $t$ or $T$ (years).
- Portfolio losses are expressed as fractions of portfolio notional (dimensionless).
- Recoveries $R_i \in [0,1]$ are taken as deterministic in the core definitions; stochastic recovery is mentioned only when explicitly sourced (e.g., discussion of recovery–default interaction in skew modeling).
- "Pricing" means risk-neutral valuation conditional on the model-implied loss distribution; "risk" means distributional tail metrics (VaR/ES/tail probabilities) that may be physical- or risk-neutral depending on application. (The sources emphasize that calibration/data issues are central in credit.)

---

### Notation Glossary (Symbols + Definitions)

| Symbol | Definition |
|--------|------------|
| $N$ | Number of names in the portfolio |
| $i \in \{1, \ldots, N\}$ | Name index |
| $\tau_i$ | Default time of name $i$ |
| $R_i$ | Recovery rate of name $i$; $\text{LGD}_i := 1 - R_i$ |
| $w_i \geq 0$ | Portfolio weight / exposure fraction of name $i$. Often $\sum_i w_i = 1$ (so portfolio loss is in $[0,1]$) |
| $\mathbf{1}_{\{\tau_i \leq t\}}$ | Default indicator by time $t$ |

**Portfolio loss:**

$$L(t) := \sum_{i=1}^{N} w_i \, \text{LGD}_i \, \mathbf{1}_{\{\tau_i \leq t\}} \in [0,1]$$

**Tranche parameters:**
- Attachment/detachment: $0 \leq K_1 < K_2 \leq 1$
- Tranche width: $W := K_2 - K_1$

**Tranche loss fraction** (normalized to tranche notional):

$$\text{TL}_{[K_1, K_2]}(L) := \frac{\min(L, K_2) - \min(L, K_1)}{K_2 - K_1} \in [0,1]$$

**Expected tranche loss (ETL)** at horizon $T$:

$$\text{ETL}_{[K_1, K_2]}(T) := \mathbb{E}\!\left[\text{TL}_{[K_1, K_2]}(L(T))\right]$$

(O'Kane defines ETL and uses it to express no-arbitrage constraints in strike.)

| Symbol | Definition |
|--------|------------|
| Copula $C$ | A distribution function on $[0,1]^d$ used to couple margins into a joint distribution via Sklar's theorem |

---

### Dependence vs Correlation vs Tail Dependence

| Concept | Definition |
|---------|------------|
| **Correlation** | Linear Pearson correlation (depends on marginal scaling/shape) |
| **Dependence** | Full joint distribution structure (captured by a copula for continuous margins) |
| **Tail dependence coefficients** $\lambda_u, \lambda_\ell$ | Limiting conditional probabilities of joint extreme events |

**Factor model:** Common systematic factor(s) + idiosyncratic components, often yielding conditional independence. (Example factor-copula specification: $U_i = a_i F + \sqrt{1 - a_i^2} Z_i$.)

**Bottom-up model:** Specifies name-level default dynamics (e.g., intensities or latent variables), then aggregates to portfolio/tranche.

**Top-down model:** Specifies portfolio loss/default-count dynamics directly (e.g., Markov chain on number of defaults), then prices tranches from that loss process.

---

### Static vs Dynamic

| Type | Description |
|------|-------------|
| **Static** | One-period/horizon joint model for $(\tau_i \leq T)$ or $L(T)$ |
| **Dynamic** | Time-evolving process $t \mapsto L(t)$ or $t \mapsto (\lambda_i(t))$ designed for time-consistent multi-maturity valuation. (O'Kane highlights time-dimension arbitrage concerns when forcing time-varying copula parameters.) |

---

### Calibration Targets (As Sourced)

- **Tranche prices/spreads and index swap prices:** central calibration targets in structured credit modeling.
- **Default dependence metrics** in portfolio credit risk (e.g., default correlation and tail heaviness) matter for loss-tail behavior.

---

## 1. Core Concepts (Definitions First)

### 1.1 Portfolio Defaults, Loss, and Tranche Loss Mapping

**Formal Definition**

- **Defaults:** $\tau_i$ for $i = 1, \ldots, N$.

- **Portfolio loss fraction:**

$$L(t) = \sum_{i=1}^{N} w_i (1 - R_i) \mathbf{1}_{\{\tau_i \leq t\}}$$

If $\sum_i w_i = 1$ and $R_i \in [0,1]$, then $0 \leq L(t) \leq 1$.

- **Tranche loss mapping (normalized):**

$$\text{TL}_{[K_1, K_2]}(L) = \frac{\min(L, K_2) - \min(L, K_1)}{K_2 - K_1} \in [0,1]$$

**Intuition**

- $L(t)$ is "how much of the reference pool is gone" by time $t$.
- A tranche only absorbs losses between $K_1$ and $K_2$; everything below $K_1$ is protected, everything above $K_2$ is already fully wiped.

**Trading/Risk Practice**

ETL curves (across $T$ and across $[K_1, K_2]$) summarize what a model implies about expected tranche impairment and drive PV/spread calculations.

O'Kane formulates no-arbitrage conditions directly in terms of ETL and the portfolio loss distribution.

---

### 1.2 Dependence vs Correlation vs Tail Dependence

**Formal Definition**

- **Dependence:** property of the joint law of $(\tau_1, \ldots, \tau_N)$ or of default indicators $(\mathbf{1}_{\{\tau_i \leq T\}})$.
- **Correlation:** scalar summary (e.g., Pearson correlation) of two random variables; not sufficient to characterize dependence.
- **Tail dependence (bivariate):** for rvs $X_1, X_2$,

$$\lambda_u = \lim_{q \to 1^-} \Pr\!\left(X_2 > F_2^{\leftarrow}(q) \mid X_1 > F_1^{\leftarrow}(q)\right)$$

$$\lambda_\ell = \lim_{q \to 0^+} \Pr\!\left(X_2 \leq F_2^{\leftarrow}(q) \mid X_1 \leq F_1^{\leftarrow}(q)\right)$$

when limits exist.

**Intuition**

Correlation can be identical across models while tail co-movement differs drastically (especially important for senior tranches and portfolio VaR/ES).

**Trading/Risk Practice**

Tail dependence is a direct lens on "clustered bad outcomes" (many defaults together) and helps explain why models with similar "correlation" can imply different tranche risks.

---

### 1.3 Copulas (Linking Marginals to Joint)

**Formal Definition (Sklar)**

For a joint distribution $F$ with margins $F_1, \ldots, F_d$, there exists a copula $C$ s.t.

$$F(x_1, \ldots, x_d) = C(F_1(x_1), \ldots, F_d(x_d))$$

and if margins are continuous, $C$ is unique.

**Intuition**

Copula = "dependence glue," margins = "single-name behavior." Change the copula, keep the same single-name default curves.

**Trading/Risk Practice**

Portfolio models often take calibrated single-name survival curves as marginals and choose a copula/factor structure to produce a tranche-consistent loss distribution.

---

### 1.4 Factor Models (Systematic + Idiosyncratic)

**Formal Definition**

A (one-factor) factor copula structure can be written as

$$U_i = a_i F + \sqrt{1 - a_i^2} \, Z_i$$

with $F$ and $Z_i$ standard and mutually independent, producing correlated $U_i$.

Conditional on $F$, the $U_i$ are independent: this is the key computational simplification.

**Intuition**

"Economy" factor drives joint stress; idiosyncratic noise drives name-specific outcomes.

**Trading/Risk Practice**

This conditional independence is exploited for efficient computation of $L(T)$ and ETL, and it aligns with economic narratives (macro stress → many defaults).

---

### 1.5 Bottom-Up vs Top-Down Portfolio Credit Models

**Bottom-Up (Formal)**

Specify $\{\tau_i\}$ (via latent variables or intensities), then compute $L(t)$ and tranche quantities.

**Top-Down (Formal)**

Specify $L(t)$ or a default-count process directly. O'Kane's Markov chain model calibrates a generator matrix to reprice tranche and index quotes.

**Intuition**

- Bottom-up: "names first."
- Top-down: "portfolio first."

**Trading/Risk Practice**

Bottom-up is natural for bespoke portfolios and name-level hedges; top-down can be natural for index tranche calibration when the market quotes are on portfolio objects.

---

### 1.6 Static vs Dynamic Models (Horizon vs Time-Evolving)

**Static Models**

A single horizon $T$: generate $L(T)$ distribution and derive ETLs for all tranches at $T$.

**Dynamic Models**

Provide a consistent evolution $t \mapsto L(t)$ or $(\lambda_i(t))$, enabling multi-maturity products and time-consistent hedging.

O'Kane stresses that making copula parameters time-dependent can break no-arbitrage in the time dimension for copula models.

In contrast, the intensity gamma model is presented as a "full dynamic multi-issuer default model" and "arbitrage free in both the strike dimension and the time dimension."

---

### 1.7 Calibration Targets (As Sourced)

**Tranche Quotes / Index Instruments**

O'Kane calibrates copula skew models by minimizing tranche PV errors relative to on-market tranches (PV ≈ 0) using an objective function over model parameters.

Top-down Markov chain calibration: determine generator $A(T)$ to reprice market tranches and the underlying index swap.

**Historical / Risk-Side Dependence Signals (As Sourced, But Not "Market Standard" Claims)**

- Default dependence affects the upper tail of loss distributions in large portfolios; dependence typically shifts mass and lengthens the right tail.
- Tail dependence coefficients are explicit extremal-dependence measures tied to copulas.

---

## 2. Why Base Correlation Is Not Enough (Motivation)

### What Base Correlation Provides (As a Quoting / Interpolation Language)

A base correlation curve $\rho(K)$ can be used to fit quoted index tranche prices exactly (after choosing an interpolation scheme in ETL space), producing an implied loss distribution.

### Source-Backed Limitations / Failure Modes

**Not Arbitrage-Free as a Full Pricing Model**

O'Kane explicitly notes base correlation is "not an arbitrage-free model for the correlation skew" and using it as a pricing/risk model requires ad hoc extensions to minimize arbitrage risk.

**Interpolation Can Create Arbitrage and Even Negative Tranchelet Spreads**

Direct linear interpolation of base correlation is "not guaranteed" to produce arbitrage-free prices; O'Kane shows tranchelet spreads can become non-monotone and even negative under some interpolations.

**No Sound Theoretical Basis for Certain Bespoke Mapping Choices**

For bespoke mapping methodologies, O'Kane states there is "no sound theoretical basis for discriminating" among approaches.

**Static Nature and Maturity Consistency**

O'Kane highlights a major issue: fitting a term structure of tranche prices is problematic for copula models; forcing time-dependent copula parameters can break time-dimension no-arbitrage, motivating dynamic models (Chs. 23–24).

**"Base-Correlation Skew" as a Symptom**

QRM notes that for exchangeable Gauss copula calibration, the implied asset correlation $\rho$ needed to match quotes varies with tranche attachment (base-correlation skew).

> **Caution:** Many additional critiques circulate in practice (e.g., hedging instability, maturity inconsistency). If a specific critique is not explicitly supported by the provided sources, I'm not sure and I won't present it as source-established.

---

## 3. Framework Map (The Main "Taxonomy")

Below is a structured map of model families beyond base correlation. Each bucket lists: state variables, dependence mechanism, strengths/limitations, usage, and calibration approach (high-level, source-grounded).

---

### A) Static Copula Models (One-Period / Horizon-Based)

#### A1. Gaussian Copula (Baseline Recap)

**State Variables:** Latent $Z$ (possibly multi-factor), idiosyncratic noises $\varepsilon_i$; or equivalently $(U_i)$ uniforms coupled by Gaussian copula.

**Dependence Mechanism:** Correlation in latent Gaussian variables; conditional independence given factor(s).

**Strengths:**
- Computationally convenient (conditional independence).

**Limitations (Source-Backed Symptom):**
- Implied correlation varies by tranche attachment when calibrating to observed tranche prices (base-correlation skew).

**Use:** Baseline for tranche loss distributions and as a reference point for skew models.

**Calibration (High-Level):** Fit to tranche quotes by solving for correlation parameter(s) (or curve $\rho(K)$ in base correlation).

---

#### A2. Student-$t$ Copula (Elliptical Copula with Tail Dependence)

**State Variables:** $t$-distributed latent variables (or Gaussian scaled by chi-square as in Student-$t$ construction).

**Dependence Mechanism:** Shared scale mixture introduces joint tail events; produces tail dependence.

**What It Captures:** Tail dependence (joint extremes). O'Kane gives the (symmetric) tail dependence coefficient for Student-$t$ copula:

$$\lambda_L = \lambda_U = 2 \, t_{\nu+1}\!\left(-\sqrt{\frac{\nu+1}{1+\rho}} \cdot \sqrt{1-\rho}\right)$$

**Strengths/Limitations:**
- *Strength:* explicit tail dependence vs Gauss.
- *Limitation:* still a static horizon model; multi-maturity consistency is not automatic (time-dimension concerns are emphasized generally for copula parameter time-dependence).

**Use:** Stressing senior-tranche risk and loss tails without changing marginals.

**Calibration:** Calibrate $(\rho, \nu)$ (and possibly factor loadings) to tranche prices/spreads.

---

#### A3. Archimedean Copulas (e.g., Clayton/Gumbel)

**State Variables:** Generator function $\psi$ defining

$$C(u_1, \ldots, u_n) = \psi^{-1}\!\left(\psi(u_1) + \cdots + \psi(u_n)\right)$$

with conditions on $\psi$.

**Dependence Mechanism:** Non-elliptical dependence; often yields asymmetric tail dependence (lower vs upper tail depending on family).

QRM illustrates qualitative tail behavior differences across copulas:
- **Gumbel** → upper tail dependence
- **Clayton** → lower tail dependence
- **Gauss** → no tail dependence

**Strengths/Limitations:**
- *Strength:* flexible asymmetric tail dependence (conceptually).
- *Limitation:* exchangeability / high-dimensional construction choices matter; calibration can be delicate.

**Use:** Exploratory dependence structures, especially where asymmetric clustering is relevant.

**Calibration:** Fit copula parameter(s) to dependence measures or directly to tranche prices (if embedded in a tranche-pricing engine).

---

#### A4. Marshall–Olkin (Common-Shock) Copula

**State Variables:** $M$ independent Poisson shock processes with intensities $\lambda_m$; shock–name incidence matrix $I_{im} \in \{0,1\}$.

**Dependence Mechanism:** Common shocks: one jump can trigger defaults of multiple names (subset-specific).

**What It Captures:** Simultaneous default / jump-to-default clustering, with a rich parameterization.

**Strengths/Limitations:**
- *Strength:* interpretable common-shock clustering.
- *Limitation:* parameter explosion; calibration identification can be hard.

**Use:** Modeling clustered/joint defaults beyond Gaussian latent correlation.

**Calibration:** Calibrate shock intensities and incidence structure to tranche prices and/or joint-default statistics (data permitting).

---

#### A5. Copula "Skew Models" to Refit Tranche Skew (Static but Tranche-Consistent)

**State Variables:** Depend on specific model; unifying output is a single portfolio loss distribution that reprices all tranches (no strike-arbitrage).

**Mechanism:** Modify dependence structure so implied loss distribution shifts probability mass in tranche-relevant regions.

**Source-Backed Rationale:** Copula models "produce one loss distribution for the portfolio from which any tranche can be priced," overcoming strike-dimension issues of base correlation.

**Calibration:** O'Kane's copula skew calibration uses an objective function minimizing absolute PV errors across standard tranches.

> **Note:** Many named "copula skew models" exist. I only treat the generic family-level idea (single loss distribution → all tranches) and the specific copula families explicitly supported above.

---

### B) Factor Models for Default Indicators (Latent-Variable and Mixture Views)

#### B1. Gaussian Threshold / Vasicek-Style One-Factor Model (Large Portfolio Lens)

**State Variables:** Systematic factor $F \sim N(0,1)$, idiosyncratic noises.

**Dependence Mechanism:** Conditional on $F$, defaults are independent; $F$ drives common stress.

**What It Captures:** Dependence in default counts and hence heavy right tails vs independence.

**Risk Use (Source-Backed Formula):** Hull gives Vasicek's worst-case default rate:

$$\boxed{\text{WCDR}(T, X) = N\!\left(\frac{N^{-1}(\text{PD}) + \sqrt{\rho} \, N^{-1}(X)}{\sqrt{1-\rho}}\right)}$$

where $N$ is standard normal CDF.

For large portfolios, an approximate high-percentile loss sums contributions across loans.

**Strengths/Limitations:**
- *Strength:* closed-form/high-speed tail quantiles for large portfolios; interpretable $\rho$.
- *Limitation:* Hull notes it "incorporates very little tail correlation" and suggests alternative copulas to remedy this.

**Calibration:**
- *Risk-side:* estimate PD and $\rho$ from data.
- *Pricing-side:* calibrate dependence parameters to tranche quotes (if used for tranche pricing).

---

#### B2. Threshold Models and Bernoulli Mixture Representations (Including LT-Archimedean)

**State Variables:** Mixing variable $\Psi$ (systematic), conditional Bernoulli defaults given $\Psi$.

**Dependence Mechanism:** Conditional independence given $\Psi$; dependence induced by randomness of $\Psi$.

QRM explicitly links LT-Archimedean copulas to threshold/mixture models and shows independence and comonotonic limits for the Clayton parameter ($\theta \to 0$ → independence; $\theta \to \infty$ → comonotonicity).

**Use:** Portfolio risk (VaR/ES), stress testing, exploring tail sensitivity to mixing distribution choice.

**Calibration:** Risk-side: fit moments / default correlation; QRM discusses model risk under fixed default probability and default correlation but varying mixing distribution.

---

#### B3. CreditRisk+ (Poisson–Gamma Mixture; Count Clustering)

**State Variables:** Default count modeled as Poisson with uncertain intensity; intensity (expected defaults) follows a gamma distribution.

**Dependence Mechanism:** Mixing Poisson with gamma yields negative binomial distribution for default counts (fatter tails than Poisson).

Hull: assume uncertain default rate; letting $qn$ have gamma distribution leads to negative binomial distribution.

**Strengths/Limitations:**
- *Strength:* analytic approximations; insurance-style tractability.
- *Limitation:* dependence captured through intensity uncertainty rather than explicit name-to-name structure; may be limited for tranche pricing without extensions.

**Use:** Credit VaR / economic capital style applications; default-count tails.

**Calibration:** Fit mean and variance (or distribution) of default counts/default rates (risk-side).

---

#### B4. Large Homogeneous Portfolio (LHP) / Asymptotics

**What Is Sourced:** QRM gives an asymptotic approximation for high quantiles in one-factor Gaussian threshold models.

**How It's Used:** Replace Monte Carlo for large $m$ with analytic approximation; good for portfolio VaR/ES and sensitivity studies.

**If You Want Deeper LHP Tranche Formulas:** I'm not sure from the provided excerpts alone what exact tranche-ETL closed forms are stated in the books; to be certain, I would need the specific section(s) where tranche ETL is derived under LHP.

---

### C) Dynamic Intensity (Reduced-Form) Portfolio Models

#### C1. Business-Time / Time-Change Intensity Models (Intensity Gamma Model)

**State Variables:** Information arrival / business time $I(t)$ modeled as sum of gamma processes plus drift; issuer hazard per unit business time $c_i(t)$.

**Dependence Mechanism:** Conditional on $I(\cdot)$, issuers are independent; dependence arises because all names "see" the same $I(t)$ (common jumps in compensators).

**Core Object / Survival:** Conditional survival:

$$Q_i(0, T \mid \{I(s)\}) = \exp\!\left(-\int_0^T c_i(s) \, dI(s)\right)$$

**What It Captures:**
- Default clustering via jumps in $I(t)$.
- Independence limit exists by removing jump variability (gamma jumps vanish and business time becomes calendar time).

**Strengths/Limitations:**
- *Strength:* dynamic, strike- and time-arbitrage-free (as stated by O'Kane).
- *Limitation:* bespoke extensions require additional assumptions (e.g., correlation across regions).

**Use:** Pricing non-standard strikes and maturities while maintaining time consistency; risk sensitivities by bump-and-recalibrate.

**Calibration:** Search over parameters $(\gamma, \lambda)$ (and drift) to fit tranche prices; O'Kane discusses optimization issues due to Monte Carlo noise.

---

#### C2. Correlated Stochastic Intensities with Common Jumps (Affine Jump Diffusion, AJD)

**State Variables:** Intensity drivers $X_c(t)$ (common) and $X_i(t)$ (idiosyncratic), each an affine jump diffusion; issuer intensity:

$$\lambda_i(t) = \alpha_i X_c(t) + X_i(t)$$

**Dependence Mechanism:** Common jump component $X_c(t)$ introduces joint intensity spikes → clustered defaults.

**Computation Hook (Source-Backed):** Conditioning on integrated common factor $Z(T) = \int_t^T X_c(s) \, ds$, defaults become conditionally independent with conditional default probabilities $p_i(T \mid Z(T))$ (given by a model-specific exponential-affine form).

**Strengths/Limitations:**
- *Strength:* dynamic reduced-form with explicit intensity co-movement and common jumps.
- *Limitation:* calibration fit may be imperfect; time-dependent parameters do not violate the model's no-arbitrage properties "as it would for a copula model" (per O'Kane's discussion).

**Use:** Pricing and risk for multi-maturity tranche products; scenario analysis via factor/intensity shocks.

**Calibration:** Fit a small set of AJD parameters to tranche quotes (O'Kane cites an example calibration study).

---

#### C3. Default Contagion and Interacting Intensities

**What Is Sourced:** QRM dedicates a section to default contagion, including "models with interacting intensities, where default contagion and counterparty risk are modelled explicitly."

QRM also shows that martingale default intensities can be characterized using derivatives of the joint survival function / survival copula, but notes this gives "little economic intuition" and is hard to use to impose a desired contagion pattern.

**How to Interpret (Disciplined):** I'm not sure what exact parametric interacting-intensity functional forms are presented in the provided excerpts; to write explicit SDE/jump forms, I would need the precise equations from QRM Section 9.8.3.

**Use:**
- *Risk:* stress "default causes more default" scenarios; counterparty-risk interactions.
- *Pricing:* potentially for products sensitive to clustering through time (but details depend on the exact model).

> **About "Hawkes processes":** I'm not sure whether Hawkes/self-exciting point processes are explicitly used for credit contagion in the provided credit sources. To confirm, I would need a source excerpt where Hawkes (or an explicitly self-exciting intensity) is defined and applied to defaults.

---

### D) Other Dependence Mechanisms (Only If Sourced)

#### D1. Top-Down Markov Chain Loss/Default-Count Model

**State Variables:** Default-count state $n \in \{0, 1, \ldots, N_c\}$ (or discretized loss states). Generator matrix $A(T)$ governs transitions.

**Dependence Mechanism:** Dependence is embedded directly in the portfolio transition rates (not in name-level correlations).

**Calibration (Source-Backed):** Calibration aims to find $A(T)$ that reprices market tranches and index swap; an arbitrage-free loss distribution can be represented with transition rates (cited to Schönbucher).

**Practical Approach:** Assume time-homogeneous $A(t) = A$ (for 5Y calibration) and a bi-diagonal "single-transition" generator; then calibrate $N_c$ transition rates $(a_0, \ldots, a_{N_c-1})$.

**Important Failure Mode:** If a transition rate is set to zero at a critical state, it can cap the loss process and violate plausible loss support.

**Strengths/Limitations:**
- *Strength:* "portfolio-first" model can be calibrated to tranche quotes by construction.
- *Limitation:* name-level hedging / bespoke mapping is non-trivial (top-down lacks explicit issuer identities unless augmented).

---

#### D2. Hybrid Structural–Reduced-Form Sketches

**What Is Sourced:** O'Kane describes hybrid models inspired by structural "firm value" dynamics but used as reduced-form models calibrated to market prices (no capital-structure interpretation).

**Dependence Mechanism:** Correlate firm-value-like state variables to induce default dependence; use rich dynamics to fit skew.

> **I'm Not Sure:** Without the full derivations for a particular hybrid model in the provided excerpts, I cannot write a specific pricing formula beyond this schematic description.

---

#### D3. Credit Migration Markov Chains (Ratings Transitions)

**What Is Sourced (Risk-Side):** Hull discusses rating transition matrices and notes that independence across periods is an approximation; "ratings momentum" can violate strict independence.

**Use:** Risk models (CreditMetrics-style) that incorporate downgrades and default as multi-state Markov processes (high-level mention in Hull's summary).

> **I'm Not Sure:** I do not have, from the provided excerpts alone, a full multi-name migration dependence mechanism (e.g., correlated migrations) beyond the one-name transition-matrix idea.

---

## 4. Math Sketches (Step-by-Step, with Unit Checks)

### 4.1 From Joint Default Model to $L(T)$: The Universal Pipeline

1. **Specify marginal default behavior** for each name at horizon $T$:

$$p_i(T) = \Pr(\tau_i \leq T) = 1 - Q_i(0, T)$$

2. **Specify dependence:**
   - Copula $C$ for $(U_1, \ldots, U_N)$ where $U_i := p_i(\tau_i)$ are uniforms under continuous margins (Sklar).
   - Or factor/intensity dynamics that imply a joint law for $(\tau_i)$.

3. **Map defaults to portfolio loss** $L(T)$:

$$L(T) = \sum_{i=1}^{N} w_i (1 - R_i) \mathbf{1}_{\{\tau_i \leq T\}}$$

4. **Compute tranche quantities** from $L(T)$ via $\text{TL}_{[K_1, K_2]}(L(T))$.

**Unit Checks:**
- $w_i$: dimensionless (fraction of notional).
- $(1 - R_i)$: dimensionless.
- Indicator: dimensionless.
- Hence $L(T)$ dimensionless; consistent with $K_1, K_2$ and tranche widths.

**Sanity Bounds:**
- If $\sum_i w_i \leq 1$, $0 \leq L(T) \leq \sum_i w_i(1 - R_i) \leq 1$.
- Tranche loss mapping ensures $0 \leq \text{TL}_{[K_1, K_2]}(L) \leq 1$ for any $L \in [0,1]$.

---

### 4.2 Static Copula Family: Core Objects and Limiting Cases

**Copula Object**

Given margins $F_i$ and copula $C$, joint DF is

$$F(x_1, \ldots, x_N) = C(F_1(x_1), \ldots, F_N(x_N))$$

**Limiting Cases (Dependence → 0 or → Maximal)**

- **Independence copula:**

$$C_{\text{ID}}(u_1, \ldots, u_N) = \prod_{i=1}^{N} u_i$$

- **Perfect positive dependence (Fréchet–Hoeffding upper bound):**

$$C_M(u_1, \ldots, u_N) = \min(u_1, \ldots, u_N)$$

**Portfolio Loss Distribution at Horizon**

Under any copula model for defaults at $T$, the distribution of $L(T)$ is obtained by:
1. Simulating $(U_1, \ldots, U_N) \sim C$,
2. Setting $\mathbf{1}_{\{\tau_i \leq T\}} = \mathbf{1}_{\{U_i \leq p_i(T)\}}$,
3. Aggregating into $L(T)$.

**Tranche ETL from Loss Distribution**

For any tranche $[K_1, K_2]$:

$$\text{ETL}_{[K_1, K_2]}(T) = \mathbb{E}\!\left[\frac{\min(L(T), K_2) - \min(L(T), K_1)}{K_2 - K_1}\right]$$

**Independence Limit:** Joint default probabilities factorize; extreme-loss probabilities decay rapidly with $N$ (binomial-like).

**Perfect Dependence Limit:** Defaults are comonotonic; large jumps in $L(T)$ become likely; senior-tranche ETL rises sharply.

---

### 4.3 Tail Dependence: Gaussian vs $t$ Copula (Rigorous Sketch)

**General Definition:** Tail dependence coefficients are limits of conditional extreme-event probabilities.

**Gaussian Copula:** QRM shows Gaussian copula tail dependence coefficient is $0$ when $\rho < 1$: asymptotic tail independence.

**$t$ Copula:** QRM gives (bivariate) tail dependence coefficient:

$$\lambda = 2 \, t_{\nu+1}\!\left(-\sqrt{\frac{(\nu+1)(1-\rho)}{1+\rho}}\right)$$

and it is strictly positive for $\rho > -1$ (asymptotic dependence).

O'Kane states the same symmetric tail dependence for Student-$t$ copula.

**Unit Check:** $\lambda$ is a probability in $[0,1]$.

---

### 4.4 Factor/Threshold Models: From Factor to Loss Distribution

**One-Factor Gaussian Threshold (Vasicek-Style)**

Conditional default probability given factor $F$ has a probit form in Hull's derivation; the key output is the high-percentile default rate formula WCDR.

In large homogeneous portfolios, the loss distribution concentrates conditional on $F$, giving analytic quantiles (WCDR).

**Mixture / LT-Archimedean Threshold**

QRM: conditional on mixing variable $\Psi$, defaults are independent Bernoulli with probabilities $p_i(\psi)$, and varying copula parameter (e.g., Clayton $\theta$) moves from independence to comonotonicity.

**Sanity / Limiting Cases:**
- If factor loading → 0: conditional PD becomes unconditional; defaults tend to independence.
- If factor loading → 1: defaults become highly dependent; tranche losses approach "single systematic driver" behavior.

---

### 4.5 Dynamic Bottom-Up Intensities: Conditional Independence + Integration

#### (i) Intensity Gamma (Business-Time) Model

**Conditional Survival** (given information arrival path):

$$Q_i(0, T \mid \{I(s)\}) = \exp\!\left(-\int_0^T c_i(s) \, dI(s)\right)$$

- Conditional on $I(\cdot)$, names are independent; hence conditional loss distribution can be built like a (possibly inhomogeneous) independent-default model.
- Unconditional distribution is obtained by integrating over the law of $I(\cdot)$ (sketch).
- Independence limit: remove jump variability so business time ≈ calendar time.

**Unit Checks:**
- $c_i(t)$: hazard per unit business time (dimension $1/(\text{business-time})$).
- $I(t)$: business time (same dimension as time if scaled).
- $\int c_i \, dI$ is dimensionless ⇒ exponent is dimensionless ⇒ survival probability is valid.

#### (ii) AJD Correlated Intensities

**Intensities:**

$$\lambda_i(t) = \alpha_i X_c(t) + X_i(t)$$

with common jump driver $X_c$.

**Conditional Independence Device:**
1. Condition on $Z(T) = \int_t^T X_c(s) \, ds$, obtain conditional default probabilities $p_i(T \mid Z(T))$.
2. Then:
   - Build conditional portfolio loss distribution,
   - Integrate over distribution of $Z(T)$.

**Unit Checks:**
- $\lambda_i(t)$: intensity (units $1/\text{year}$).
- $Z(T) = \int X_c ds$: dimensionless if $X_c$ has units $1/\text{year}$.

---

### 4.6 Dynamic Top-Down Markov Chain: From Generator to Loss Distribution

**Core Object**

A Markov chain on default count (or discretized loss) with generator matrix $A(T)$.

Calibration determines $A(T)$ so the induced distribution of $L(T)$ reprices index and tranches.

**Single-Transition (Bi-Diagonal) Sketch**

- States $n = 0, 1, \ldots, N_c$.
- Transition $n \to n+1$ occurs at rate $a_n$.
- Then $P(n, T)$ is obtained from Kolmogorov forward equations (schematic; explicit equations are in O'Kane's chapter, but not fully reproduced in the provided excerpt).

**Sanity Checks:**
- Rates $a_n \geq 0$ (units $1/\text{year}$).
- If a critical $a_n = 0$, loss support can be artificially capped (O'Kane's caution).

---

## 5. Calibration and Validation Map (Practitioner-Facing)

### End-to-End Conceptual Workflow (Source-Grounded)

#### Inputs

- Single-name survival curves $Q_i(0, T)$ (from CDS curves) and recovery assumptions $R_i$.
- Index and tranche quotes (pricing calibration instruments).
- Risk-side: default dependence evidence / constraints; data limitations are a major obstacle in credit modeling.

#### Choose Model Family and Parameterization

- Static copula (Gaussian, $t$, Archimedean, Marshall–Olkin, mixture).
- Bottom-up dynamic intensities (IG, AJD common jumps).
- Top-down Markov chain generator model.

#### Calibration Objective (Pricing Context)

- For static copula skew models, O'Kane uses an objective function minimizing average absolute PV errors across standard tranches (on-market PV ≈ 0).
- For top-down Markov chain, calibration = determine generator $A(T)$ to reprice market tranches and index swap; direct vs indirect approaches are discussed.
- For dynamic intensity models, calibrate parameters to fit tranche skew and/or term structure; for IG, O'Kane describes parameter search under Monte Carlo noise and discusses bootstrap over maturity buckets.

#### Validation

**Repricing Checks (Fit Instruments):** Tranches used in calibration should be repriced within tolerance/bid-offer. (O'Kane explicitly lists model fit within bid-offer as a requirement.)

**No-Arbitrage Checks:**
- *Strike dimension:* ETL monotonicity/convexity constraints (O'Kane motivates no-arbitrage constraints in ETL space).
- *Interpolation sanity:* avoid tranchelet spread violations/negative spreads due to interpolation.

**Stability Checks:**
- Sensitivity stability under small quote bumps (O'Kane stresses parameter stability and usability).

**Out-of-Sample / Scenario Checks (Risk Context):**
- Stress the systematic factor(s), dependence parameters, or intensity jump components and observe tail metrics (see Section 6).

#### Model Risk Controls

- **Non-uniqueness:** multiple dependence structures can fit the same tranche quotes (QRM: dependence affects tails strongly; O'Kane: multiple bespoke mapping approaches have no clear theoretical discriminator).
- **Data uncertainty:** credit data scarcity limits reliable statistical calibration.
- **Robustness:** check limiting cases, bounds, and scenario plausibility (rates nonnegative; ETLs within $[0,1]$; loss support not artificially capped).

---

## 6. Measurement & Risk (Advanced but Disciplined)

### 6.1 What Risks These Models Target

**Tail Risk / Clustering Risk**

Default dependence "drastically" impacts the right tail of a credit loss distribution; dependence lengthens the upper tail.

**Dispersion Risk (Distribution-Shape Risk)**

Two models can match the same expected loss (or a single tranche) but imply different tail probabilities.

**Contagion Risk**

QRM explicitly discusses default contagion and interacting intensities as an explicit modeling route.

---

### 6.2 Correlation/Covariance vs Tail Dependence (Discipline)

- Correlation summarizes linear association but does not determine tail co-movement.
- Tail dependence coefficients $\lambda_u, \lambda_\ell$ are copula-driven extremal dependence summaries.
- Gaussian vs $t$ illustrates the point sharply: Gauss has $\lambda = 0$ for $\rho < 1$; $t$ has $\lambda > 0$ for $\rho > -1$.

---

### 6.3 Stress Testing Dependence (How to "Shock")

**Static Copula Shocks**
- Shock copula parameters (e.g., $\rho$, $\nu$ for $t$ copula; Archimedean $\theta$).
- Compare tail metrics: $\Pr(L(T) > x)$, $\text{ETL}_{[K_1, K_2]}(T)$.

**Factor Model Shocks**
- Shock systematic factor realization $F$ (scenario analysis) or increase factor loading(s) $a_i$ to raise correlation.

**Intensity Shocks (Dynamic)**
- *IG:* shock jump intensity/size parameters of business time $I(t)$; observe survival and ETL.
- *AJD:* shock common jump parameters of $X_c(t)$ to amplify clustering.

**Contagion Shocks**

I'm not sure of the specific parametric form in QRM's interacting intensity models from the excerpt; in practice, a stress can be represented as a post-default jump in other names' intensities (schematic).

---

### 6.4 Sensitivity Outputs (Pricing/Risk)

**Tranche PV Sensitivity to Dependence Parameters**

If closed-form Greeks are not provided, define sensitivity by finite difference:

$$\frac{\partial \text{ETL}}{\partial \theta} \approx \frac{\text{ETL}(\theta + \Delta) - \text{ETL}(\theta - \Delta)}{2\Delta}$$

**For Hedging Correlations in Practice (Source-Backed Warning)**

O'Kane emphasizes that for bespoke tranches, correlation sensitivity should be measured relative to the index base correlation curve (hedging in standard tranches).

---

## 7. Worked Examples (At Least 12 Numeric Examples; Fully Numeric, Toy)

### Common Toy Assumptions Unless Stated

- Equal weights unless stated; loss = default count × LGD × weight.
- If $R$ is constant, $\text{LGD} = 1 - R$.

---

### Example 1: Two-Name Toy — Same Marginal PD, Independent vs Perfectly Dependent → Distribution of Default Count

Let each name have one-year default probability $p = 0.10$.

**Independent Defaults:**

| $K$ | Probability |
|-----|-------------|
| 0 | $(1-p)^2 = 0.9^2 = 0.81$ |
| 1 | $2p(1-p) = 2(0.1)(0.9) = 0.18$ |
| 2 | $p^2 = 0.01$ |

**Perfect Positive Dependence (Comonotonic):**

Either both default or both survive.

| $K$ | Probability |
|-----|-------------|
| 0 | $1 - p = 0.90$ |
| 1 | $0$ |
| 2 | $p = 0.10$ |

**Takeaway:** Same marginal PD, but extreme outcome $K = 2$ jumps from 1% to 10%.

---

### Example 2: $N$-Name Toy — Increased Dependence Changes Probability of Extreme Losses

Let $N = 10$, each has $p = 0.05$.

**Independent Defaults:** $K \sim \text{Binomial}(10, 0.05)$. Compute $\Pr(K \geq 5)$.

Compute terms:

$$P(K = 5) = \binom{10}{5} 0.05^5 \cdot 0.95^5$$

- $\binom{10}{5} = 252$
- $0.05^5 = 3.125 \times 10^{-7}$
- $0.95^5 = 0.7737809375$
- Product $= 252 \times (3.125 \times 10^{-7} \cdot 0.7737809375) \approx 6.0935 \times 10^{-5}$

$$P(K = 6) = \binom{10}{6} 0.05^6 \cdot 0.95^4 \approx 2.6726 \times 10^{-6}$$

$$P(K = 7) = \binom{10}{7} 0.05^7 \cdot 0.95^3 \approx 8.0379 \times 10^{-8}$$

$$P(K = 8) = \binom{10}{8} 0.05^8 \cdot 0.95^2 \approx 1.5864 \times 10^{-9}$$

$$P(K = 9) \approx 1.8555 \times 10^{-11}$$

$$P(K = 10) \approx 9.7656 \times 10^{-14}$$

**Sum:**

$$\Pr(K \geq 5) \approx 6.36898 \times 10^{-5} = 0.00637\%$$

**Perfect Dependence (Comonotonic, Identical PDs):**

- $\Pr(K = 10) = p = 0.05$
- $\Pr(K = 0) = 0.95$
- Hence $\Pr(K \geq 5) = 0.05 = 5\%$

**Takeaway:** A dependence shift can move extreme-loss probability by ~three orders of magnitude.

---

### Example 3: Tranche ETL Under Two Dependence Settings (Toy Loss Distributions)

**Tranches:** equity $[0, 3\%]$ and senior mezz $[10\%, 15\%]$.

Define two toy loss distributions for $L$ at maturity $T$ (dimensionless):

#### Case A (Lighter Tail)

| $L$ | Probability |
|-----|-------------|
| 0 | 0.70 |
| 0.02 | 0.20 |
| 0.06 | 0.08 |
| 0.12 | 0.015 |
| 0.25 | 0.005 |

**(a) ETL for $[0, 3\%]$:**

$$\text{TL}_{[0, 0.03]}(L) = \frac{\min(L, 0.03)}{0.03}$$

| $L$ | $\text{TL}$ |
|-----|-------------|
| 0 | 0 |
| 0.02 | $0.02/0.03 = 0.6667$ |
| $\geq 0.03$ | 1 |

$$\text{ETL} = 0.70 \cdot 0 + 0.20 \cdot 0.6667 + (0.08 + 0.015 + 0.005) \cdot 1 = 0.1333 + 0.1000 = \mathbf{0.2333}$$

**(b) ETL for $[10\%, 15\%]$:**

$$\text{TL}_{[0.10, 0.15]}(L) = \frac{\min(\max(L - 0.10, 0), 0.05)}{0.05}$$

Only $L = 0.12$ and $0.25$ contribute:

| $L$ | $\text{TL}$ |
|-----|-------------|
| 0.12 | $(0.02)/0.05 = 0.4$ |
| 0.25 | 1 |

$$\text{ETL} = 0.015 \cdot 0.4 + 0.005 \cdot 1 = 0.006 + 0.005 = \mathbf{0.011}$$

#### Case B (Heavier Tail / More Clustering)

| $L$ | Probability |
|-----|-------------|
| 0 | 0.68 |
| 0.02 | 0.17 |
| 0.06 | 0.08 |
| 0.12 | 0.04 |
| 0.25 | 0.03 |

**Equity ETL:**

$$0.17 \cdot 0.6667 + (0.08 + 0.04 + 0.03) \cdot 1 = 0.1133 + 0.1500 = \mathbf{0.2633}$$

**Senior Mezz ETL:**

$$0.04 \cdot 0.4 + 0.03 \cdot 1 = 0.016 + 0.03 = \mathbf{0.046}$$

**Takeaway:** Tail-weight shifts can mildly move equity ETL but strongly reprice senior mezz ETL.

---

### Example 4: Gaussian vs "Heavier Tail" Dependence (Student-$t$ Copula) Illustration (Sourced)

Use the tail dependence coefficient $\lambda$ as an extremal dependence summary.

- **Gaussian copula:** $\lambda = 0$ for $\rho < 1$ (asymptotic tail independence).
- **$t$ copula:** QRM provides $\lambda = 2 \, t_{\nu+1}\!\left(-\sqrt{\frac{(\nu+1)(1-\rho)}{1+\rho}}\right)$.

**Table values:** for $\nu = 4$ and $\rho = 0.5$, $\lambda \approx 0.25$.

**Toy Interpretation Using a 1% Tail Threshold**

Let "extreme" mean $U > 0.99$ on uniform scale (a high quantile event).

- **Gaussian copula:** $\Pr(U_2 > 0.99 \mid U_1 > 0.99) \to 0$ as tail threshold $\to 1$.
- **$t$ copula ($\nu = 4$, $\rho = 0.5$):** asymptotically,

$$\Pr(U_2 > q \mid U_1 > q) \approx \lambda \approx 0.25 \quad \text{for very high } q$$

So conditional co-extremes are far more likely under $t$ than Gauss even at similar "correlation."

---

### Example 5: Multi-Factor Toy — Adding a Second Factor Creates Sector Clustering (Sourced Factor-Model Idea)

Hull gives a one-factor structure $U_i = a_i F + \sqrt{1 - a_i^2} Z_i$.

Extend to two independent factors: global $G$ and sector $S$.

Let sector A names have:

$$U = \underbrace{0.3 G}_{\text{global}} + \underbrace{0.4 S_A}_{\text{sector}} + \underbrace{\sqrt{1 - 0.3^2 - 0.4^2} \, \varepsilon}_{\text{idiosyncratic}}$$

where $\sqrt{1 - 0.09 - 0.16} = \sqrt{0.75} = 0.8660$.

Sector B names have:

$$U = 0.3 G + 0.4 S_B + 0.8660 \, \varepsilon$$

**Compute Correlations** (since all components are standard normal and independent):

- **Same sector:**

$$\text{Corr}(U_i, U_j) = 0.3^2 + 0.4^2 = 0.09 + 0.16 = \mathbf{0.25}$$

- **Different sectors:**

$$\text{Corr}(U_i, U_j) = 0.3^2 = \mathbf{0.09}$$

**Takeaway:** Adding a sector factor increases within-sector dependence without forcing the same increase cross-sector.

---

### Example 6: Dynamic Intensity Toy (IG-Style) — Survival Changes Under a Common-Factor Intensity Shift

Use IG conditional survival form (business time $I(t)$):

$$Q(0, T \mid I) = \exp\!\left(-\int_0^T c(s) \, dI(s)\right)$$

**Toy Choices:**
- Constant hazard-per-business-time $c(s) = 0.02$.
- Baseline: $I(t) = t$ (business time equals calendar time).

Then:
- $\int_0^5 0.02 \, dI = 0.02 \cdot 5 = 0.10$
- $Q(0, 5) = e^{-0.10} = 0.9048$

**Now introduce a common jump in business time at year 2:**

$I(5) = 5 + \Delta I$ with $\Delta I = 3$ (information shock).

Then:
- $\int_0^5 0.02 \, dI = 0.02 \cdot 8 = 0.16$
- $Q(0, 5) = e^{-0.16} = 0.8521$

**Takeaway:** A shared information-time jump lowers survival for all names simultaneously → clustered defaults.

---

### Example 7: Contagion/Clustering Toy (ONLY Schematic; Explicit Parametric Form Not in Excerpt)

QRM discusses default contagion and mentions models with interacting intensities. But the exact formula is not in the excerpt, so I'm not sure of the canonical parametric form used there. Below is a purely illustrative numeric toy consistent with "default increases others' intensity."

**Toy Setup (4 Names A, B, C, D):**

- Baseline: A defaults with prob 0.02. If A defaults, each of B, C, D defaults with prob 0.06; if A survives, B, C, D default with prob 0.02. (Conditional independence assumed.)

**Compute Probability of ≥ 2 Total Defaults:**

**(i) A defaults and at least one among B, C, D defaults:**

- $\Pr(A) = 0.02$
- $\Pr(\geq 1 \text{ of 3 with } p = 0.06) = 1 - 0.94^3$
- $0.94^3 = 0.830584$, so $= 0.169416$
- Contribution $= 0.02 \cdot 0.169416 = 0.00338832$

**(ii) A survives and at least two among B, C, D default:**

- $\Pr(A^c) = 0.98$

For 3 names with $p = 0.02$:
- $P(0) = 0.98^3 = 0.941192$
- $P(1) = 3(0.02)(0.98^2) = 3(0.02)(0.9604) = 0.057624$
- $P(\geq 2) = 1 - 0.941192 - 0.057624 = 0.001184$
- Contribution $= 0.98 \cdot 0.001184 = 0.00116032$

**Total:**

$$\Pr(\geq 2) = 0.00338832 + 0.00116032 = 0.00454864 = \mathbf{0.4549\%}$$

**Baseline (No Contagion; All 4 Independent with $p = 0.02$):**

- $P(0) = 0.98^4 = 0.92236816$
- $P(1) = 4(0.02)(0.98^3) = 4(0.02)(0.941192) = 0.07529536$
- $P(\geq 2) = 1 - 0.92236816 - 0.07529536 = 0.00233648 = \mathbf{0.2336\%}$

**Takeaway:** Even a simple post-default PD step-up roughly doubles $\Pr(\geq 2)$.

---

### Example 8: Calibration Toy — Solving for Two Dependence Parameters (Conceptual, with Numeric Steps)

O'Kane calibrates tranche models by minimizing PV errors with a multidimensional optimizer. A real tranche PV is nonlinear in parameters, so I'm not sure of a closed-form two-parameter solve in the sourced models. Below is a toy linear "pricing response" used only to illustrate the mechanics.

Suppose a model outputs two tranche ETLs:

$$\text{ETL}_{0-3}(\theta_1, \theta_2) = 0.20 + 0.10 \theta_1 - 0.05 \theta_2$$

$$\text{ETL}_{10-15}(\theta_1, \theta_2) = 0.01 + 0.03 \theta_1 + 0.04 \theta_2$$

Given target ETLs:

$$\text{ETL}_{0-3} = 0.25, \quad \text{ETL}_{10-15} = 0.02$$

**Solve:**

From first equation:

$$0.20 + 0.10 \theta_1 - 0.05 \theta_2 = 0.25 \Rightarrow 0.10 \theta_1 - 0.05 \theta_2 = 0.05 \Rightarrow \theta_1 = 0.5 + 0.5 \theta_2$$

Plug into second:

$$0.01 + 0.03(0.5 + 0.5 \theta_2) + 0.04 \theta_2 = 0.02$$

$$\Rightarrow 0.01 + 0.015 + 0.015 \theta_2 + 0.04 \theta_2 = 0.02$$

$$0.025 + 0.055 \theta_2 = 0.02 \Rightarrow 0.055 \theta_2 = -0.005 \Rightarrow \theta_2 = -0.090909$$

Then:

$$\theta_1 = 0.5 + 0.5(-0.090909) = 0.454545$$

**Takeaway:** Real calibrations do this with a pricing engine; the toy just shows how multi-target calibration pins parameters (and can yield unintuitive values, here negative $\theta_2$, prompting constraint checks).

---

### Example 9: Model-Risk Comparison — Two Models Fit One Tranche but Imply Different Tails

We construct two toy loss distributions that match the same equity ETL for $[0, 3\%]$ but differ in tail probability $\Pr(L > 10\%)$.

Equity ETL uses:

$$\text{ETL}_{0-3} = \mathbb{E}\!\left[\frac{\min(L, 0.03)}{0.03}\right]$$

#### Model A

| $L$ | Probability |
|-----|-------------|
| 0 | 0.675 |
| 0.02 | 0.225 |
| 0.12 | 0.10 |

**Compute Equity ETL:**

| $L$ | $\text{TL}$ |
|-----|-------------|
| 0 | 0 |
| 0.02 | 0.6667 |
| 0.12 | 1 |

$$\text{ETL}_{0-3} = 0.225(0.6667) + 0.10(1) = 0.1500 + 0.10 = \mathbf{0.25}$$

**Tail Metric:**

$$\Pr(L > 0.10) = \Pr(L = 0.12) = \mathbf{0.10}$$

#### Model B

| $L$ | Probability |
|-----|-------------|
| 0 | 0.70 |
| 0.02 | 0.15 |
| 0.25 | 0.15 |

**Equity ETL:**

$$\text{ETL}_{0-3} = 0.15(0.6667) + 0.15(1) = 0.10 + 0.15 = \mathbf{0.25}$$

**Tail Metric:**

$$\Pr(L > 0.10) = \Pr(L = 0.25) = \mathbf{0.15}$$

**Takeaway:** Same fitted equity ETL, different tail probability—this is "dispersion/model-risk" in the tail.

---

### Example 10: Stress Test — Shock Dependence and Compute $\Delta$ETL for a Senior Tranche

Use Model A as "base" and Model B as "stressed dependence" (more tail mass).

**Senior Tranche:** $[10\%, 15\%]$.

#### Model A ETL

Only $L = 0.12$ contributes.

Tranche loss at $L = 0.12$:

$$\text{TL}_{10-15}(0.12) = \frac{0.12 - 0.10}{0.05} = 0.4$$

ETL:

$$\text{ETL}_{10-15} = 0.10(0.4) = \mathbf{0.04}$$

#### Model B ETL

Only $L = 0.25$ contributes fully:

$$\text{TL}_{10-15}(0.25) = 1$$

ETL:

$$\text{ETL}_{10-15} = 0.15(1) = \mathbf{0.15}$$

**Stress Impact:**

$$\Delta \text{ETL}_{10-15} = 0.15 - 0.04 = \mathbf{0.11}$$

**Interpretation:** Senior risk is far more sensitive to tail-mass changes than equity.

---

### Example 11: Recovery Interaction — Hold Dependence Fixed, Vary Recovery; Compute Tail Loss Changes

**Toy default-count distribution** for $N = 10$ equal weights $w = 0.1$.

**Default count $K$ distribution:**

| $K$ | Probability |
|-----|-------------|
| 0 | 0.80 |
| 1 | 0.15 |
| 2 | 0.04 |
| 5 | 0.01 |

Portfolio loss $L = (K/N) \cdot (1 - R)$.

#### Case 1: $R = 40\% \Rightarrow \text{LGD} = 0.6$

| $K$ | $L$ |
|-----|-----|
| 1 | $0.1 \cdot 0.6 = 0.06$ |
| 2 | $0.2 \cdot 0.6 = 0.12$ |
| 5 | $0.5 \cdot 0.6 = 0.30$ |

**Expected Loss:**

$$\mathbb{E}[L] = 0.15(0.06) + 0.04(0.12) + 0.01(0.30) = 0.009 + 0.0048 + 0.003 = \mathbf{0.0168}$$

**Tail Probability:**

$$\Pr(L > 10\%) = \Pr(K \geq 2) = 0.04 + 0.01 = \mathbf{0.05}$$

**Conditional Tail Mean (ES-Like at Threshold 10%):**

$$\mathbb{E}[L \mid L > 0.10] = \frac{0.04(0.12) + 0.01(0.30)}{0.05} = \frac{0.0048 + 0.003}{0.05} = \mathbf{0.156}$$

#### Case 2: $R = 20\% \Rightarrow \text{LGD} = 0.8$

| $K$ | $L$ |
|-----|-----|
| 1 | 0.08 |
| 2 | 0.16 |
| 5 | 0.40 |

**Expected Loss:**

$$\mathbb{E}[L] = 0.15(0.08) + 0.04(0.16) + 0.01(0.40) = 0.012 + 0.0064 + 0.004 = \mathbf{0.0224}$$

**Tail Probability** $\Pr(L > 10\%)$ is still $0.05$ (since $K \geq 2$ still needed), but tail severity increases:

$$\mathbb{E}[L \mid L > 0.10] = \frac{0.04(0.16) + 0.01(0.40)}{0.05} = \frac{0.0064 + 0.004}{0.05} = \mathbf{0.208}$$

**Takeaway:** Lower recovery increases expected loss and tail severity even if tail probability is unchanged.

---

### Example 12: Sanity-Check Example — Detect Impossible ETL / Loss Bounds Violations

**Tranche:** $[10\%, 15\%]$, so tranche loss fraction must satisfy $0 \leq \text{TL} \leq 1$, hence $0 \leq \text{ETL} \leq 1$.

Suppose someone reports (incorrectly) $\text{ETL}_{10-15} = 1.20$.

This is **impossible** because tranche loss fraction is bounded by 1.

**How to Detect:**

Check tranche-width bound:
- Unnormalized expected tranche loss (portfolio-notional units) is $W \cdot \text{ETL} \leq W$.
- Here $W = 0.05$. If $\text{ETL} = 1.20$, unnormalized ETL would be $0.06 > 0.05$, violating the maximum possible tranche loss.

**Related "Arbitrage Smell" from Sources:**

O'Kane shows that poor interpolation can generate negative tranchelet spreads and other arbitrage indicators—another form of sanity failure.

---

## 8. Practical Notes

### Common Pitfalls

**Confusing Correlation with Dependence / Tail Dependence**

Tail dependence is a separate extremal dependence concept; Gaussian copula has $\lambda = 0$ for $\rho < 1$, while $t$ copula has $\lambda > 0$.

**Mixing Horizon-Based Copulas with Time-Dependent Intensities Without Consistency**

O'Kane warns that making copula parameters time dependent can break time-dimension no-arbitrage for copula models.

**Treating Implied Dependence Parameters as "True"**

Model parameters are often calibrated to prices; QRM stresses data limitations and the role of modeling assumptions.

**Calibration Non-Uniqueness**

Multiple dependence structures can fit the same tranche quotes; model risk remains in the tail (see Example 9).

**Ignoring No-Arbitrage Constraints in Strike Dimension**

O'Kane emphasizes no-arbitrage conditions in terms of ETL.

**Interpolation Artifacts (Spikes, Negative Spreads)**

Direct base-correlation interpolation can violate monotonicity and generate negative tranchelet spreads.

---

### Verification Tests

**Bounds:**
- $0 \leq L(T) \leq 1$
- $0 \leq \text{TL} \leq 1$
- $0 \leq \text{ETL} \leq 1$

**Monotonicity:**

For equity base tranche ETL $\psi(T, K) = \mathbb{E}[\min(L(T), K)]$, check monotonicity in $K$ and shape constraints consistent with a valid loss distribution (O'Kane uses $\psi$ and its second derivative to infer implied density).

**Repricing Checks:**

Calibration instruments (index, standard tranches) should reprice within tolerance/bid-offer.

**Stability Under Small Perturbations:**

Recalibrate after small quote bumps and verify parameter and PV stability (O'Kane's "parameter stability is vital").

**Limiting-Case Checks:**

Independence and comonotonic limits for copulas/mixtures should recover expected behavior.

---

## 9. Summary & Recall

### 10-Bullet Executive Summary

1. Portfolio credit modeling is about building a joint law for default times $(\tau_i)$ to obtain $L(T)$ and tranche ETLs.

2. Sklar's theorem separates modeling into marginals + copula (dependence glue).

3. Correlation is not dependence; tail dependence measures extremal co-movement.

4. Gaussian copula is asymptotically tail independent ($\lambda = 0$ if $\rho < 1$); $t$ copula can have $\lambda > 0$.

5. Base correlation is widely used but is not an arbitrage-free skew model; interpolation can generate arbitrage/negative tranchelet spreads.

6. Copula skew models aim to produce a single coherent loss distribution that prices all strikes (strike-arbitrage-free).

7. Static copulas face maturity consistency issues; time-varying copula parameters can break time no-arbitrage, motivating dynamic models.

8. Dynamic bottom-up intensity models (IG, AJD) create dependence via common factors/jumps while keeping conditional independence for tractability.

9. Top-down Markov chain models calibrate portfolio transition rates (generator matrices) to tranche/index quotes, embedding dependence at portfolio level.

10. Model risk is fundamentally tail risk: different models can fit the same tranche but imply different extreme-loss probabilities (Example 9).

---

### Cheat Sheet: Model Family Map + Key Definitions

**Loss + Tranche Mapping:**

$$L(T) = \sum w_i (1 - R_i) \mathbf{1}_{\{\tau_i \leq T\}}, \quad \text{TL}_{[K_1, K_2]}(L) = \frac{\min(L, K_2) - \min(L, K_1)}{K_2 - K_1}$$

**Copula (Sklar):**

$$F = C(F_1, \ldots, F_d)$$

**Tail Dependence:**

$$\lambda_u = \lim_{q \to 1^-} \Pr\!\left(X_2 > F_2^{\leftarrow}(q) \mid X_1 > F_1^{\leftarrow}(q)\right)$$

**Static Dependence Families:**
- Gaussian copula (tail independent)
- $t$ copula (tail dependent)
- Archimedean (asymmetric tails)
- Marshall–Olkin (common shocks)

**Dynamic Families:**
- IG (business time with gamma-process jumps)
- AJD (common jump intensity factor)
- Top-down Markov chain (generator calibration)

---

### 25 Flashcards (Q/A)

1. **Q:** Define portfolio loss $L(t)$.
   **A:** $L(t) = \sum_{i=1}^{N} w_i (1 - R_i) \mathbf{1}_{\{\tau_i \leq t\}}$.

2. **Q:** What is tranche loss fraction $\text{TL}_{[K_1, K_2]}(L)$?
   **A:** $\frac{\min(L, K_2) - \min(L, K_1)}{K_2 - K_1}$.

3. **Q:** State Sklar's theorem (informally).
   **A:** Any joint DF equals a copula applied to its margins; copula is unique if margins are continuous.

4. **Q:** What does a copula model "buy you" for tranche pricing?
   **A:** One coherent portfolio loss distribution from which all tranche prices follow.

5. **Q:** Define upper tail dependence coefficient.
   **A:** $\lambda_u = \lim_{q \to 1^-} \Pr(X_2 > F_2^{\leftarrow}(q) \mid X_1 > F_1^{\leftarrow}(q))$.

6. **Q:** Gaussian copula tail dependence for $\rho < 1$?
   **A:** $\lambda = 0$ (asymptotic tail independence).

7. **Q:** Why can $t$ copula have more joint extremes than Gauss at similar correlation?
   **A:** It has positive tail dependence ($\lambda > 0$).

8. **Q:** What is "base-correlation skew" (model symptom)?
   **A:** Implied $\rho$ varies with tranche attachment when fitting Gauss copula to tranche quotes.

9. **Q:** Is base correlation arbitrage-free as a skew model?
   **A:** No; using it requires ad hoc extensions to minimize arbitrage.

10. **Q:** Give an example of interpolation pathology in base correlation.
    **A:** Non-monotone tranchelet spreads or negative spreads.

11. **Q:** Define bottom-up vs top-down.
    **A:** Bottom-up models names then aggregates; top-down models portfolio loss/count directly.

12. **Q:** What is a factor copula model (one-factor form)?
    **A:** $U_i = a_i F + \sqrt{1 - a_i^2} Z_i$.

13. **Q:** What's the main computational benefit of factor models?
    **A:** Conditional independence given factors.

14. **Q:** Write Hull's Vasicek WCDR formula.
    **A:** $\text{WCDR}(T, X) = N\!\left(\frac{N^{-1}(\text{PD}) + \sqrt{\rho} N^{-1}(X)}{\sqrt{1-\rho}}\right)$.

15. **Q:** What does Hull say is a disadvantage of Vasicek's model?
    **A:** It incorporates very little tail correlation.

16. **Q:** What is CreditRisk+'s key modeling trick?
    **A:** Poisson defaults with gamma uncertainty → negative binomial default counts.

17. **Q:** IG model conditional survival formula?
    **A:** $Q_i(0, T \mid I) = \exp\!\left(-\int_0^T c_i(s) \, dI(s)\right)$.

18. **Q:** What creates dependence in IG?
    **A:** Common jumps in business time $I(t)$ seen by all names.

19. **Q:** What does O'Kane claim about IG arbitrage properties?
    **A:** Arbitrage free in strike and time dimensions.

20. **Q:** AJD multi-issuer intensity form?
    **A:** $\lambda_i(t) = \alpha_i X_c(t) + X_i(t)$.

21. **Q:** How does AJD regain tractability?
    **A:** Condition on $Z(T) = \int X_c ds$ to get conditional independence.

22. **Q:** What is a top-down Markov chain model calibrated to?
    **A:** Generator matrix $A(T)$ to reprice tranches and index swap.

23. **Q:** What is a Marshall–Olkin copula mechanism?
    **A:** Independent Poisson shocks affecting subsets of names.

24. **Q:** What is default contagion (QRM lens)?
    **A:** Default dependence where information/structure makes defaults influence intensities; includes interacting intensities.

25. **Q:** Key model-risk lesson for tranche modeling?
    **A:** Fitting quotes does not uniquely determine tail behavior; tail metrics can differ across models.

---

## 10. Mini Problem Set (16 Questions)

Provide brief solution sketches for questions 1–8 only.

1. Compute $\Pr(K = 0, 1, 2)$ for two names with PD $p = 0.08$ under independence.

2. Same as (1) under perfect dependence.

3. For $N = 20$, $p = 0.02$, compute $\Pr(K \geq 3)$ under independence (binomial tail).

4. Using the tranche mapping, compute $\text{TL}_{[0.03, 0.07]}(L)$ for $L = 0.01, 0.05, 0.10$.

5. Given a discrete loss distribution, compute $\text{ETL}_{[0, 0.03]}$ and $\text{ETL}_{[0.10, 0.15]}$.

6. Using Hull's Vasicek WCDR, compute $\text{WCDR}(1, 0.999)$ for PD $= 0.02$ and $\rho = 0.1$.

7. Show that Gaussian copula has $\lambda = 0$ when $\rho < 1$ (conceptual explanation using "asymptotic independence").

8. In an IG model toy, compute the survival change when a business-time jump adds $\Delta I = 2$ by $T = 5$ with constant $c = 0.03$.

9. Explain why two different copulas can have the same linear correlation but different tail dependence.

10. Design a stress test for a top-down Markov chain model: which transition rates would you shock to stress senior tranches?

11. Discuss why setting some transition rate $a_n = 0$ can be problematic in top-down models.

12. Explain how conditional independence enables faster computation in factor and intensity models.

13. Describe a workflow for calibrating a dynamic intensity model to both 5Y and 10Y tranche quotes.

14. Discuss model risk from recovery assumptions and their interaction with default clustering.

15. Explain the tradeoffs between bottom-up and top-down approaches for bespoke tranches.

16. Propose validation tests for ensuring strike-dimension no-arbitrage in an interpolated ETL curve.

---

### Solution Sketches (1–8)

**1. Independence, $p = 0.08$:**

- $P_0 = (0.92)^2 = 0.8464$
- $P_1 = 2(0.08)(0.92) = 0.1472$
- $P_2 = 0.08^2 = 0.0064$

**2. Perfect dependence:**

- $P_0 = 1 - p = 0.92$
- $P_2 = p = 0.08$
- $P_1 = 0$

**3. Binomial tail:** $K \sim \text{Bin}(20, 0.02)$. Compute $1 - P_0 - P_1 - P_2$.

- $P_0 = 0.98^{20}$
- $P_1 = 20(0.02) \cdot 0.98^{19}$
- $P_2 = \binom{20}{2}(0.02)^2 \cdot 0.98^{18}$

Then subtract from 1.

**4. Tranche loss $[3\%, 7\%]$, width $0.04$:**

- $L = 0.01 < 0.03 \Rightarrow 0$
- $L = 0.05 \Rightarrow (0.05 - 0.03)/0.04 = 0.5$
- $L = 0.10 \Rightarrow 1$

**5.** Apply $\text{TL} = \frac{\min(L, K_2) - \min(L, K_1)}{K_2 - K_1}$ state-by-state; multiply by probabilities and sum.

**6. Use Hull formula:**

$$\text{WCDR} = N\!\left(\frac{N^{-1}(0.02) + \sqrt{0.1} \cdot N^{-1}(0.999)}{\sqrt{0.9}}\right)$$

Compute with normal quantiles and CDF (calculator/spreadsheet).

**7. Sketch:** QRM shows conditional distribution under Gauss copula yields $\lambda = 0$ as tail threshold $\to \infty$ for $\rho < 1$.

**8. IG survival:** $Q = \exp\!\left(-\int_0^T c \, dI\right) = \exp(-c \cdot I(T))$ if $c$ constant.

- Baseline $I(T) = 5 \Rightarrow Q_0 = e^{-0.03 \cdot 5} = e^{-0.15} = 0.8607$
- Jump adds $2 \Rightarrow I(T) = 7 \Rightarrow Q_1 = e^{-0.21} = 0.8106$
- Change: $Q_1 - Q_0 \approx -0.0501$

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

| Content | Source |
|---------|--------|
| Portfolio loss / tranche loss definitions | O'Kane Ch. 11–12 |
| ETL and no-arbitrage constraints | O'Kane Ch. 12–13 |
| Sklar's theorem | QRM Ch. 5 |
| Tail dependence definitions and formulas | QRM Ch. 5; O'Kane Ch. 13 |
| Gaussian copula tail independence | QRM Ch. 5 |
| Student-$t$ copula tail dependence formula | QRM Ch. 5; O'Kane Ch. 13 |
| Archimedean copula generator form | QRM Ch. 5 |
| Base correlation limitations | O'Kane Ch. 10 |
| Factor copula structure | Hull Ch. 24; O'Kane Ch. 13 |
| Vasicek WCDR formula | Hull Ch. 24 |
| CreditRisk+ Poisson-gamma mixture | Hull Ch. 24 |
| Intensity Gamma model | O'Kane Ch. 23 |
| AJD intensity model | O'Kane Ch. 24 |
| Top-down Markov chain calibration | O'Kane Ch. 22 |
| Default contagion / interacting intensities | QRM Ch. 9 |
| Marshall–Olkin copula | O'Kane Ch. 13 |

### (B) Reasoned Inference — Note Derivation Logic

| Content | Derivation |
|---------|------------|
| Worked examples (1–12) | Direct application of sourced formulas to toy parameters |
| Unit checks | Dimensional analysis of sourced formulas |
| Limiting case behaviors | Logical consequences of sourced definitions |

### (C) Speculation — Flag Uncertainties

| Content | Uncertainty |
|---------|-------------|
| Hawkes processes for credit contagion | Not explicitly confirmed in provided excerpts |
| Exact parametric forms for interacting intensities | Would need QRM Section 9.8.3 for specifics |
| LHP tranche-ETL closed forms | Not fully reproduced in excerpts |
| Hybrid structural–reduced-form pricing formulas | Schematic only without full derivations |

---

*Last Updated: January 2026*
