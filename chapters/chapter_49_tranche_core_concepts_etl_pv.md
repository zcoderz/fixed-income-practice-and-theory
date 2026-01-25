# Chapter 49: Tranche Core Concepts — Loss Distribution, Expected Tranche Loss (ETL), and PV

---

## Conventions & Notation

| Convention | Description |
|------------|-------------|
| **Time** | Valuation at $t=0$; maturity $T$; coupon dates $0=t_0 < t_1 < \cdots < t_N = T$ |
| **Measure** | Expectations $E[\cdot]$ are risk-neutral unless stated otherwise |
| **Discounting** | $Z(t) = D(0,t)$ is the discount factor to $t$ (the sources describe tranche pricing using discount factors and "Libor discount factors") |
| **Portfolio loss state variable** | $L(t) \in [0, L_{\max}]$ is fraction of portfolio notional lost by $t$ |
| **Tranche strikes** | Attachment $A$ and detachment $D$ (also written $K_1, K_2$ in sources); width $W = D - A$ |
| **Units** | "Fractions" are in portfolio-notional units unless explicitly normalized; $ values use a portfolio notional $N_{\text{port}}$ |

---

## 0. Setup

### Conventions Used in This Chapter

- $L(t)$ is the single state variable: portfolio cumulative loss fraction by time $t$, bounded by a (possibly) sub-100% maximum $L_{\max}$ if recoveries cap losses.
- Tranche is characterized by $[A, D]$ with width $W = D - A$.
- We treat the loss distribution (or equivalently the ETL curve) as an input; pricing uses discounting and risk-neutral expectations.

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $N_{\text{port}}$ | Portfolio notional (e.g., \$100mm in examples) |
| $L(t)$ | Portfolio cumulative loss fraction of $N_{\text{port}}$ by time $t$ |
| $A, D$ | Tranche attachment/detachment (fractions of $N_{\text{port}}$) |
| $W = D - A$ | Tranche width (fraction of $N_{\text{port}}$) |
| $\text{TL}(L)$ | Tranche loss amount (fraction of $N_{\text{port}}$) induced by portfolio loss level $L$ |
| $\text{ON}(L)$ | Tranche outstanding notional (fraction of $N_{\text{port}}$) remaining at portfolio loss $L$ |
| $L_{\text{tr}}(t)$ | Fractional loss on tranche notional (normalized to $[0,1]$) |
| $\text{ETL}(t) = E[\text{TL}(L(t))]$ | Expected tranche loss amount (fraction of $N_{\text{port}}$) |
| $\text{EON}(t) = E[\text{ON}(L(t))] = W - \text{ETL}(t)$ | Expected outstanding (fraction of $N_{\text{port}}$) |
| $Q(t; A, D)$ | Tranche "survival" (expected surviving fraction of tranche notional), $Q(t) = E[1 - L_{\text{tr}}(t)]$ |
| $Z(t) = D(0,t)$ | Discount factor to $t$ |
| $s$ | Tranche running spread/coupon (per year, e.g., 500 bp = 0.05) |
| $\alpha_i = \Delta(t_{i-1}, t_i)$ | Accrual year-fraction for $(t_{i-1}, t_i)$ (sources: often quarterly, Act/360; we keep $\alpha_i$ generic) |

---

## 1. Core Concepts (Definitions First)

### 1.1 Portfolio Loss $L(t)$ as the State Variable

**Formal Definition:**

$L(t)$ is the fraction of the portfolio notional that has been lost to defaults by time $t$, with $0 \le L(t) \le L_{\max}$.

**Intuition:**

Think of $L(t)$ as "how far through the capital structure the portfolio has burned."

**Trading/Risk Practice:**

Tranche payoffs are functions of $L(t)$; models aim to produce the distribution of $L(t)$ under pricing (risk-neutral) assumptions.

---

### 1.2 Attachment $A$, Detachment $D$, Width $W = D - A$

**Formal Definition:**

A tranche absorbs portfolio losses from $A$ up to $D$; its width is $W = D - A$.

**Intuition:**

$A$ is subordination (how much portfolio loss must occur before this tranche is hit); $D$ is where it is fully exhausted.

**Trading/Risk Practice:**

Market tranches are quoted by $[A, D]$ strikes; premium is paid on outstanding tranche notional and falls as losses accrue.

---

### 1.3 Tranche Loss Function $\text{TL}(L) = \min(\max(L - A, 0), W)$

**Formal Definition (portfolio-notional units):**

$$\text{TL}(L) = \min(\max(L - A, 0), W)$$

**Equivalent Forms (useful identities):**

$$\text{TL}(L) = \max(L - A, 0) - \max(L - D, 0) = \min(L, D) - \min(L, A)$$

The sources define the normalized tranche loss as a piecewise-linear function of $L(T)$ using a "max–max" form.

**Intuition:**

Below $A$, no loss; between $A$ and $D$, you take dollar-for-dollar loss; above $D$, you're fully wiped (loss $W$).

**Trading/Risk Practice:**

This nonlinearity is why tranche risk differs across the capital structure—same portfolio loss can mean 0% loss for senior but 100% for equity.

---

### 1.4 Tranche Outstanding Notional $\text{ON}(L) = W - \text{TL}(L)$

**Formal Definition (portfolio-notional units):**

$$\text{ON}(L) = W - \text{TL}(L) \in [0, W]$$

**Intuition:**

It's the remaining "principal" of the tranche (as a fraction of the portfolio notional).

**Trading/Risk Practice:**

Premium is paid on outstanding notional; as losses happen, premium payments decline.

---

### 1.5 Normalized Tranche Loss $L_{\text{tr}}(t)$ and Tranche "Survival" $Q(t)$

**Formal Definition (fraction of tranche notional):**

$$L_{\text{tr}}(t) = \frac{\text{TL}(L(t))}{W} \in [0, 1]$$

In the sources, the fractional tranche loss is defined (at horizon $T$) as

$$L(T, K_1, K_2) = \frac{\max(L(T) - K_1, 0) - \max(L(T) - K_2, 0)}{K_2 - K_1}, \quad (K_1 = A,\ K_2 = D)$$

i.e., exactly $\text{TL}(L(T))/W$.

**Tranche Survival / Expected Surviving Notional:**

$$Q(t; A, D) = E[1 - L_{\text{tr}}(t)]$$

and the sources explicitly define $Q(t, K_1, K_2) = E[1 - L(t, K_1, K_2)]$.

Equivalently,

$$Q(t; A, D) = \frac{E[\text{ON}(L(t))]}{W} = \frac{\text{EON}(t)}{W}$$

---

### 1.6 Loss Distribution of $L(t)$

**Formal Definition:**

The (risk-neutral) distribution is described by a CDF $F_{L(t)}(\ell) = P(L(t) \le \ell)$ (and density $f$ when it exists). In risk measurement language, determining a distribution function $F_X(x) = P(X \le x)$ (or its functionals) is the core object.

**Intuition:**

This is "where the portfolio might end up" by time $t$.

**Trading/Risk Practice:**

A tranche pricing model is largely a machine that outputs (or implies) the loss distribution of $L(t)$ or, more directly, the tranche survival curve $Q(t; A, D)$.

---

### 1.7 Expected Tranche Loss $\text{ETL}(t) = E[\text{TL}(L(t))]$

**Formal Definition (portfolio-notional units):**

$$\text{ETL}(t) = E[\text{TL}(L(t))] = E[\min(L(t), D) - \min(L(t), A)]$$

For an equity/base tranche $[0, K]$, the sources define the ETL curve

$$\psi(T, K) = E[\min(L(T), K)]$$

where $L(T)$ is fractional portfolio loss.

**Intuition:**

Average amount of the tranche that is expected to be used up by time $t$.

**Trading/Risk Practice:**

Traders track the ETL curve over time because it drives both legs of tranche PV and the "expected outstanding" used for premium payments.

---

### 1.8 Expected Outstanding $\text{EON}(t) = W - \text{ETL}(t)$

**Formal Definition:**

$$\text{EON}(t) = E[\text{ON}(L(t))] = W - \text{ETL}(t)$$

**Intuition:**

Expected remaining principal of the tranche (in portfolio-notional units).

**Trading/Risk Practice:**

Premium leg PV is essentially "spread $\times$ discounted expected outstanding $\times$ accrual."

---

### 1.9 How ETL Drives PV of Premium and Protection Legs

**Conceptual Link:**

- **Protection leg** pays realized tranche losses → PV is an expected discounted loss, i.e. discounted increments of expected tranche loss.
- **Premium leg** pays spread on surviving tranche notional → PV is spread times expected outstanding (with accrual and discounting).

---

## 2. Loss Distribution → Tranche Loss Distribution

A distribution for portfolio loss $L(t)$ induces a distribution for tranche loss $\text{TL}(L(t))$ because the tranche loss is a deterministic nonlinear mapping of the state variable:

$$L(t) \mapsto \text{TL}(L(t)) = \min(\max(L(t) - A, 0), W)$$

**Key Idea:** Portfolio loss is the state variable; tranche loss is a nonlinear function of it.

### Shape Differences Across Capital Structure

- **Equity ($A \approx 0$):** Losses start immediately; often little/no probability mass at "0 tranche loss" once the portfolio has any chance of loss.

- **Mezzanine ($A > 0$):** There is typically positive probability of 0 tranche loss (portfolio loss never reaches $A$). The sources describe tranche loss distributions having a probability mass at 0 equal to $\Pr(L(T) \le K_1)$, and at 100% tranche loss equal to $\Pr(L(T) \ge K_2)$.

- **Senior ($A$ high):** Most scenarios do not reach attachment; but tail scenarios can wipe it out.

### Why Equity/Mezz/Senior Behave Differently

The mapping $\text{TL}(L)$ has a "kink" at $A$ and caps at $W$; as you move $A$ upward, the tranche becomes more "tail dependent" (depends on rare high-loss scenarios).

---

## 3. Expected Tranche Loss (ETL): Definitions and Interpretation

### 3.1 ETL at a Horizon vs ETL Curve Over Time

**Horizon ETL:**

$$\text{ETL}(T) = E[\text{TL}(L(T))]$$

**ETL Curve:** $t \mapsto \text{ETL}(t)$ for $t \in [0, T]$.

In practice, pricing requires ETL (or equivalently $Q(t)$) at the coupon grid dates.

---

### 3.2 Incremental Expected Tranche Loss

For a time grid $0 = t_0 < \cdots < t_N = T$,

$$\Delta\text{ETL}_i := \text{ETL}(t_i) - \text{ETL}(t_{i-1}) = E\!\left[\text{TL}(L(t_i)) - \text{TL}(L(t_{i-1}))\right]$$

**Interpretation:** Expected loss "arriving" in interval $(t_{i-1}, t_i]$.

---

### 3.3 Expected Outstanding vs Expected Loss

- $\text{ETL}(t)$ tells you expected tranche principal used up (portfolio-notional units).
- $\text{EON}(t) = W - \text{ETL}(t)$ tells you expected tranche principal still alive (portfolio-notional units).

The sources encode this through tranche "survival" $Q(t; A, D) = E[1 - L_{\text{tr}}(t)]$, i.e., expected surviving fraction of tranche notional.

---

## 4. PV of Tranche Legs: Premium Leg and Protection Leg (Core Deliverable)

We write everything first in portfolio-notional units (fractions), then convert to \$ by multiplying by $N_{\text{port}}$. This makes unit checks easy.

Let tranche $[A, D]$ have width $W = D - A$. Define:

- **Realized tranche loss fraction (portfolio units):** $\text{TL}(t) := \text{TL}(L(t)) \in [0, W]$
- **Expected tranche loss:** $\text{ETL}(t) = E[\text{TL}(t)]$
- **Expected outstanding:** $\text{EON}(t) = W - \text{ETL}(t)$
- **Normalized loss on tranche notional:** $L_{\text{tr}}(t) = \text{TL}(t)/W \in [0, 1]$
- **Tranche survival:** $Q(t) = E[1 - L_{\text{tr}}(t)] = \text{EON}(t)/W$

---

### A) Premium Leg PV (Fee Leg)

**Concept:** Pay running spread $s$ on outstanding tranche notional.

The sources state that premium coupons are paid on surviving tranche notional (typically quarterly; Actual/360), and per \$1 face value at $t_i$ the coupon is $\alpha_i \, s \, (1 - L_{\text{tr}}(t_i))$.

**Per \$1 of tranche face value** (i.e., per \$1 of $N_{\text{tr}} = N_{\text{port}} W$):

If we (crudely) pay on end-of-period notional, PV would resemble

$$PV_{\text{prem}}(\$1\ \text{face}) \approx s \sum_{i=1}^{N} \alpha_i \, Z(t_i) \, E[1 - L_{\text{tr}}(t_i)]$$

The sources describe a more accurate approximation: pay on average tranche notional over the accrual period, giving

$$\boxed{PV_{\text{prem}}(\$1\ \text{face}) = s \sum_{i=1}^{N} \alpha_i \, Z(t_i) \, E\!\left[1 - \frac{L_{\text{tr}}(t_{i-1}) + L_{\text{tr}}(t_i)}{2}\right]}$$

Equivalently (using $Q(t) = E[1 - L_{\text{tr}}(t)]$),

$$PV_{\text{prem}}(\$1\ \text{face}) = \frac{s}{2} \sum_{i=1}^{N} \alpha_i \, Z(t_i) \, \left(Q(t_{i-1}) + Q(t_i)\right)$$

**In portfolio-notional-\$ units:**

Tranche face value is $N_{\text{tr}} = N_{\text{port}} W$. Multiply the "per \$1 face" PV by $N_{\text{tr}}$, or write directly:

$$\boxed{PV_{\text{prem}}(\$) = s \sum_{i=1}^{N} \alpha_i \, Z(t_i) \, \underbrace{E\!\left[\frac{\text{ON}(t_{i-1}) + \text{ON}(t_i)}{2}\right]}_{\text{expected average outstanding (portfolio fraction)}} \, N_{\text{port}}}$$

since $\text{ON}(t) = W(1 - L_{\text{tr}}(t))$.

**Premium Accrual on Default / Intra-Period Losses:**

The sources emphasize that losses inside an accrual period affect the next coupon via pro-rated notional exposure, and suggest a practical approximation: assume losses occur mid-period so the tranche notional is the average of start/end notional; and they note a direct analogy to premium accrued at default in a CDS.

If you need the exact accrual/settlement convention (e.g., specific index tranche docs, day count stub rules, front-end protection, pay-as-you-go conventions), I'm not sure—we would need the trade's documentation and desk conventions.

**Unit Check:**

| Quantity | Units |
|----------|-------|
| $s$ | 1/year (e.g., 500 bp/year = 0.05/year) |
| $\alpha_i$ | years |
| $Z(t_i)$ | unitless |
| $E[\text{ON}]$ | fraction of portfolio notional |
| Multiply by $N_{\text{port}}$ | \$ |

So PV is in \$.

---

### B) Protection Leg PV (Default/Loss Leg)

**Concept:** Protection leg pays realized tranche losses when they occur.

The sources write (in normalized tranche-loss terms) that the protection leg PV is

$$PV_{\text{prot}} = \int_0^T Z(s) \, E[\,dL_{\text{tr}}(s)\,]$$

and equivalently in terms of tranche survival $Q$: $PV_{\text{prot}} = \int_0^T Z(s)(-dQ(s))$.

**Discrete-Time Approximation (portfolio-notional units):**

Using tranche loss amount $\text{TL}(t)$ (portfolio fraction),

$$\boxed{PV_{\text{prot}}(\$) \approx N_{\text{port}} \sum_{i=1}^{N} Z(t_i) \, E[\text{TL}(t_i) - \text{TL}(t_{i-1})] = N_{\text{port}} \sum_{i=1}^{N} Z(t_i) \left(\text{ETL}(t_i) - \text{ETL}(t_{i-1})\right)}$$

**Why This Works:** Protection pays increments of realized tranche loss; expectation and discounting yield "expected discounted loss."

**Unit Check:**

| Quantity | Units |
|----------|-------|
| $\text{ETL}$ | fraction of portfolio notional (unitless) |
| $Z$ | unitless |
| Multiply by $N_{\text{port}}$ | \$ |

So PV is in \$.

---

### C) Fair Tranche Spread / Par Premium

Define the premium PV01 ("risky PV01") as PV of paying 1 unit of spread on the premium leg.

The sources define a tranche risky PV01 in terms of $Q(t)$.

With the average-notional approximation,

$$PV01_{\text{prem}}(\$) = N_{\text{port}} \sum_{i=1}^{N} \alpha_i \, Z(t_i) \, E\!\left[\frac{\text{ON}(t_{i-1}) + \text{ON}(t_i)}{2}\right]$$

The **par spread** $s^*$ (ignoring any upfront $U$) solves $PV_{\text{prot}}(\$) = s^* \, PV01_{\text{prem}}(\$)$, so

$$\boxed{s^* = \frac{PV_{\text{prot}}(\$)}{PV01_{\text{prem}}(\$)} = \frac{\sum_i Z(t_i)\left(\text{ETL}(t_i) - \text{ETL}(t_{i-1})\right)}{\sum_i \alpha_i \, Z(t_i) \, E\!\left[\frac{\text{ON}(t_{i-1}) + \text{ON}(t_i)}{2}\right]}}$$

**Units:**

- Numerator: unitless (fraction) because $Z$ is unitless, ETL increment unitless
- Denominator: years $\times$ unitless $\times$ unitless = years

$\Rightarrow$ $s^*$ is 1/year. Convert to bp/year by multiplying by $10^4$.

**Upfronts:**

Some tranches (notably equity) can involve an upfront plus running spread; the sources' index tranche quote example includes upfront for equity and running spread paid quarterly.

This chapter focuses on ETL-driven PV of the two legs; incorporating upfront is straightforward algebra once leg PVs are defined.

---

### D) What Is Not in This Chapter

- We do not calibrate dependence/correlation models (e.g., Gaussian copula, compound/base correlation, etc.).
- We treat the loss distribution (or ETL curve) as given and show how it drives pricing.
- We do not specify product-specific documentation choices beyond what is explicitly stated in the sources (e.g., exact settlement mechanics for all tranche types). If needed, I'm not sure without the term sheet and desk conventions.

---

## 5. Where the Loss Distribution Comes From (High-Level Only)

A tranche pricing model needs the distribution of $L(t)$ (or equivalently expectations of $\min(L(t), K)$ for various strikes).

**Common Modeling Routes (preview only):**

1. **Independent defaults:** Simplest baseline; tends to understate tail risk.

2. **Correlated defaults via factor/latent-variable models:** Condition on a common factor so names are conditionally independent, compute conditional loss distribution, then integrate over factor density. The sources describe computing unconditional loss density $f(L(T))$ by integrating conditional density $f(L(T)|Z)$ over a market factor $Z$.

3. **Copula / dependence models:** Map single-name default probabilities and a dependence structure into a portfolio loss distribution.

Full modeling, calibration, and "correlation skew" are deferred to the next chapters.

---

## 6. Measurement & Risk (Only What Belongs in Chapter 49)

### Risk Drivers in ETL Language

- **Equity tranches:** Sensitive to "body/mean" of loss distribution; a modest increase in expected loss can wipe out large fraction of a thin equity width.

- **Senior/super-senior:** Sensitive to the tail; probability of reaching attachment dominates.

### Tail Risk Preview via Tranche Loss Distribution

The sources emphasize that tranche loss distributions often have mass at 0 loss ($\Pr(L(T) \le A)$) and at full loss ($\Pr(L(T) \ge D)$), highlighting the importance of tail probabilities for senior tranches.

### "Correlation Risk" (Preview Only)

Dependence primarily reshapes the tail of $L(t)$, and therefore can strongly affect senior tranche ETL even if marginal default probabilities are unchanged (full treatment later).

### Risk Report Preview (ETL-Based)

- **ETL curve:** $t \mapsto \text{ETL}(t)$
- **Expected outstanding curve:** $t \mapsto \text{EON}(t)$
- **PV decomposition:** $PV_{\text{prot}}$ vs $PV_{\text{prem}}$ (and any upfront if applicable)

---

## 7. Math and Derivations (Step-by-Step)

### 7.1 Derive $\text{TL}(L)$ and $\text{ON}(L)$

Start from the tranche's piecewise-linear normalized loss (sources' definition at horizon $T$):

$$L_{\text{tr}}(L) = \frac{\max(L - A, 0) - \max(L - D, 0)}{W}$$

Multiply both sides by $W$ to express the loss in portfolio-notional units:

$$\text{TL}(L) = W \, L_{\text{tr}}(L) = \max(L - A, 0) - \max(L - D, 0)$$

Now show equivalence to the "clipped linear" form:

- If $L \le A$: both max terms are 0 $\Rightarrow$ $\text{TL}(L) = 0 = \min(\max(L - A, 0), W)$.
- If $A < L < D$: $\max(L - A, 0) = L - A$, $\max(L - D, 0) = 0$ $\Rightarrow$ $\text{TL}(L) = L - A$.
- If $L \ge D$: $\max(L - A, 0) = L - A$, $\max(L - D, 0) = L - D$ $\Rightarrow$ $\text{TL}(L) = D - A = W$.

Thus

$$\boxed{\text{TL}(L) = \min(\max(L - A, 0), W)}$$

Outstanding notional (portfolio fraction) is then

$$\boxed{\text{ON}(L) = W - \text{TL}(L)}$$

**Sanity Checks:**

- **Bounds:** $0 \le \text{TL}(L) \le W$, $0 \le \text{ON}(L) \le W$.
- **Monotonicity:** $\text{TL}(L)$ nondecreasing in $L$; $\text{ON}(L)$ nonincreasing in $L$.

---

### 7.2 Derive $\text{ETL}(t) = E[\text{TL}(L(t))]$

By definition,

$$\boxed{\text{ETL}(t) = E[\text{TL}(L(t))]}$$

Write it as an integral under the loss distribution $F_{L(t)}$:

If $L(t)$ has density $f_{L(t)}$,

$$\text{ETL}(t) = \int_0^{L_{\max}} \text{TL}(\ell) \, f_{L(t)}(\ell) \, d\ell$$

If $L(t)$ is discrete (scenario distribution), $\text{ETL}(t) = \sum_j p_j \, \text{TL}(\ell_j)$.

For equity/base tranche $[0, K]$, the sources define $\psi(T, K) = E[\min(L(T), K)]$, consistent with $\text{ETL}(T)$ when $A = 0, D = K$.

---

### 7.3 Derive $PV_{\text{prot}}$ in Terms of ETL Increments

Protection PV in discrete time:

$$PV_{\text{prot}}(\$) \approx N_{\text{port}} \sum_{i=1}^{N} Z(t_i) \, E[\text{TL}(t_i) - \text{TL}(t_{i-1})] = N_{\text{port}} \sum_{i=1}^{N} Z(t_i) \, \left(\text{ETL}(t_i) - \text{ETL}(t_{i-1})\right)$$

This matches the sources' "protection leg PV is the discounted expected loss increments," expressed either via $E[dL_{\text{tr}}]$ or via $-dQ$.

**Unit Check:**

Each term: $Z$ unitless $\times$ ETL increment unitless $\times$ $N_{\text{port}}$ dollars $\Rightarrow$ dollars.

---

### 7.4 Derive $PV_{\text{prem}}$ in Terms of Expected Outstanding

The sources' premium leg (per \$1 face) is paid on surviving tranche notional.

Using the mid-period loss approximation (average notional over the period), they obtain:

$$PV_{\text{prem}}(\$1\ \text{face}) = s \sum_{i=1}^{N} \alpha_i Z(t_i) \, E\!\left[1 - \frac{L_{\text{tr}}(t_{i-1}) + L_{\text{tr}}(t_i)}{2}\right]$$

Convert to portfolio-notional dollars by multiplying by tranche face $N_{\text{tr}} = N_{\text{port}} W$, or equivalently:

$$PV_{\text{prem}}(\$) = s \sum_{i=1}^{N} \alpha_i Z(t_i) \, E\!\left[\frac{\text{ON}(t_{i-1}) + \text{ON}(t_i)}{2}\right] \, N_{\text{port}}$$

**Unit Check:**

$\alpha$: years; $s$: 1/year; outstanding fraction: unitless; $Z$: unitless $\Rightarrow$ dollars after multiplying by $N_{\text{port}}$.

---

### 7.5 Par Spread and Limiting Cases

**Par Spread:**

$$\boxed{s^* = \frac{PV_{\text{prot}}(\$)}{PV01_{\text{prem}}(\$)}}$$

**Limiting Case 1: $\text{ETL}(t) \equiv 0$**

Then $PV_{\text{prot}} = 0$, $\text{EON}(t) = W$ (full outstanding), so

$$s^* = 0$$

*Sanity check:* If tranche never expected to lose, fair premium is zero (ignoring any risk premium beyond model).

**Limiting Case 2: Tranche Wiped Immediately**

Suppose $\text{ETL}(t)$ jumps from $0$ to $W$ "at $t = 0$" (all loss realized instantly).

- Protection PV is approximately $N_{\text{port}} W$ (discount factor near 1).
- Premium leg collapses because outstanding becomes ~0 almost immediately (premium PV01 near 0).

In this limit, a running-spread-only par $s^*$ becomes ill-defined (ratio "finite/near-zero"); economically, the value is captured by an upfront loss payment (and documentation dictates whether any accrued premium is owed). I'm not sure about the exact settlement/coupon accrual for this edge case without the tranche documentation.

---

## 8. Worked Examples (At Least 12 Numeric Examples, Fully Numeric)

### Global Example Settings (Unless Stated Otherwise)

- **Portfolio notional:** $N_{\text{port}} = \$100{,}000{,}000$ (\$100mm).
- **Tranche strikes** are fractions of portfolio notional; convert to \$ by multiplying by $N_{\text{port}}$.
- When we compute a par spread in a one-period toy, we use:
  - Maturity $T = 5$ years, discount factor $Z(T) = 0.82$ (toy),
  - Premium paid once at $T$ with accrual $\alpha = 5$,
  - Premium outstanding approximated by average $\frac{W + \text{EON}(T)}{2}$ (mid-period approximation analog).

---

### Example 1: TL(L) Mapping Table (Single Tranche)

Choose tranche $[A, D] = [3\%, 7\%]$.

So $A = 0.03$, $D = 0.07$, $W = 0.04$ (4% of portfolio, i.e. \$4mm).

Compute $\text{TL}(L) = \min(\max(L - A, 0), W)$ and $\text{ON}(L) = W - \text{TL}(L)$:

| Portfolio Loss $L$ | $\max(L - A, 0)$ | $\text{TL}(L)$ | $\text{ON}(L)$ | TL \$ | ON \$ |
|:------------------:|:----------------:|:--------------:|:--------------:|:-----:|:-----:|
| 0% (0.00) | 0 | 0.00 | 0.04 | \$0.0mm | \$4.0mm |
| 1% (0.01) | 0 | 0.00 | 0.04 | \$0.0mm | \$4.0mm |
| 3% (0.03) | 0 | 0.00 | 0.04 | \$0.0mm | \$4.0mm |
| 5% (0.05) | 0.02 | 0.02 | 0.02 | \$2.0mm | \$2.0mm |
| 10% (0.10) | 0.07 | 0.04 (cap) | 0.00 | \$4.0mm | \$0.0mm |
| 20% (0.20) | 0.17 | 0.04 (cap) | 0.00 | \$4.0mm | \$0.0mm |

**Intermediate Check (5% row):** $L - A = 0.05 - 0.03 = 0.02 < W = 0.04$, so TL = 0.02.

---

### Example 2: Compare Equity vs Mezz vs Senior at the Same $L$

Use tranches (consistent with common index strikes):

- **Equity:** $[0\%, 3\%]$, $W = 0.03$
- **Mezz:** $[3\%, 7\%]$, $W = 0.04$
- **Senior:** $[7\%, 10\%]$, $W = 0.03$

These strike levels appear as standard index tranche strikes in the sources (e.g., CDX 0–3, 3–7, 7–10, etc.).

**Case A: Portfolio Loss $L = 5\%$**

| Tranche | TL | ON | Status |
|---------|----|----|--------|
| Equity $[0, 3]$ | $\min(0.05, 0.03) = 0.03$ | 0 | Wiped |
| Mezz $[3, 7]$ | 0.02 | 0.02 | Partial loss |
| Senior $[7, 10]$ | 0 (since $L < A$) | 0.03 | Untouched |

**Case B: Portfolio Loss $L = 12\%$**

| Tranche | TL | ON | Status |
|---------|----|----|--------|
| Equity $[0, 3]$ | 0.03 | 0 | Wiped |
| Mezz $[3, 7]$ | 0.04 | 0 | Wiped |
| Senior $[7, 10]$ | $\min(0.12 - 0.07, 0.03) = 0.03$ | 0 | Wiped |

---

### Example 3: Discrete Loss Distribution → Compute $\text{ETL}(T)$ for a Tranche

Let $T$ be a horizon date with portfolio loss outcomes:

$$L(T) \in \{0\%, 2\%, 6\%, 15\%\}, \quad \Pr = \{0.5,\, 0.2,\, 0.2,\, 0.1\}$$

Tranche: $[3\%, 7\%]$, $W = 4\%$.

Compute $\text{TL}(L)$ scenario-by-scenario:

| $L$ | $\text{TL}(L)$ | Probability |
|:---:|:--------------:|:-----------:|
| 0% | 0 | 0.5 |
| 2% (< A) | 0 | 0.2 |
| 6% | $L - A = 3\% = 0.03$ | 0.2 |
| 15% ($\ge D$) | $W = 4\% = 0.04$ | 0.1 |

Now expectation:

$$\text{ETL}(T) = 0.5 \cdot 0 + 0.2 \cdot 0 + 0.2 \cdot 0.03 + 0.1 \cdot 0.04 = 0.006 + 0.004 = 0.010$$

So $\text{ETL}(T) = 1.0\%$ of portfolio notional = **\$1.0mm**.

*Bound check:* $0 \le 0.010 \le W = 0.04$ ✔

---

### Example 4: From $\text{ETL}(T)$ to Expected Outstanding $\text{EON}(T)$

Same tranche $[3, 7]$, $W = 0.04$, and $\text{ETL}(T) = 0.010$.

$$\text{EON}(T) = W - \text{ETL}(T) = 0.04 - 0.010 = 0.030$$

So expected outstanding is 3.0% of portfolio = **\$3.0mm**.

*Interpretation:* On average, 1% has been eaten; 3% of the original 4% tranche remains.

---

### Example 5: Time Curve of ETL — Compute Incremental ETL Over Intervals (Toy)

Tranche $[3, 7]$, $W = 0.04$. Suppose an ETL curve (portfolio units):

| Time $t$ | $\text{ETL}(t)$ |
|:--------:|:---------------:|
| 0 | 0.000 |
| 1 | 0.002 (0.2%) |
| 3 | 0.008 (0.8%) |
| 5 | 0.012 (1.2%) |

**Incremental ETL:**

| Interval | $\Delta\text{ETL}$ |
|:--------:|:------------------:|
| $(0, 1]$ | $0.002 - 0.000 = 0.002$ |
| $(1, 3]$ | $0.008 - 0.002 = 0.006$ |
| $(3, 5]$ | $0.012 - 0.008 = 0.004$ |

**Expected Outstanding:**

| Time $t$ | $\text{EON}(t)$ |
|:--------:|:---------------:|
| 0 | 0.040 |
| 1 | 0.038 |
| 3 | 0.032 |
| 5 | 0.028 |

---

### Example 6: Protection Leg PV from ETL Increments (Explicit Sum)

Use Example 5 ETL curve and toy discount factors:

- $Z(1) = 0.97$, $Z(3) = 0.90$, $Z(5) = 0.82$

Compute protection PV (portfolio fraction first):

$$PV_{\text{prot}}(\text{fraction}) = 0.97 \cdot 0.002 + 0.90 \cdot 0.006 + 0.82 \cdot 0.004$$
$$= 0.00194 + 0.00540 + 0.00328 = 0.01062$$

Convert to \$:

$$PV_{\text{prot}}(\$) = 0.01062 \cdot \$100\text{mm} = \boxed{\$1.062\text{mm}}$$

---

### Example 7: Premium Leg PV from Expected Outstanding (Explicit Sum; Average-Notional Approximation)

Use:
- Spread $s = 500$ bp/year $= 0.05$
- ETL/EON from Example 5
- Same discount factors
- Accruals: $\alpha_1 = 1$ (0–1y), $\alpha_2 = 2$ (1–3y), $\alpha_3 = 2$ (3–5y)
- Premium paid on average outstanding over each interval (mid-period approximation)

**Average Expected Outstanding Per Interval:**

| Interval | Average EON |
|:--------:|:-----------:|
| $[0, 1]$ | $(0.040 + 0.038)/2 = 0.039$ |
| $[1, 3]$ | $(0.038 + 0.032)/2 = 0.035$ |
| $[3, 5]$ | $(0.032 + 0.028)/2 = 0.030$ |

**Compute PV01** (premium PV per unit spread) in portfolio fraction-years:

$$PV01_{\text{prem}}(\text{fraction-years}) = 0.97 \cdot 1 \cdot 0.039 + 0.90 \cdot 2 \cdot 0.035 + 0.82 \cdot 2 \cdot 0.030$$
$$= 0.03783 + 0.06300 + 0.04920 = 0.15003$$

Convert PV01 to \$ per 100% spread:

$$PV01_{\text{prem}}(\$,\, s=1) = 0.15003 \cdot \$100\text{mm} = \$15.003\text{mm}$$

Now premium PV at $s = 0.05$:

$$PV_{\text{prem}}(\$) = s \cdot PV01(\$,\, s=1) = 0.05 \cdot 15.003 = \boxed{\$0.75015\text{mm}}$$

---

### Example 8: Solve Par Tranche Spread $s^*$ (bp/year)

From Examples 6–7:

- $PV_{\text{prot}} = \$1.062\text{mm}$
- $PV01_{\text{prem}}(s=1) = \$15.003\text{mm}$

$$s^* = \frac{1.062}{15.003} = 0.07078 \text{ per year}$$

In bp/year:

$$s^* = 0.07078 \times 10^4 = \boxed{707.8 \text{ bp/year}}$$

*Check:* $s^*$ is higher than 500 bp because in this toy ETL curve the protection PV is relatively large versus the premium PV01.

---

### Example 9: Sensitivity to Attachment — Compare $[0, 3]$ vs $[3, 7]$ (One-Period Toy)

Use the discrete horizon distribution from Example 3 at $T = 5$ years, discount $Z(T) = 0.82$, accrual $\alpha = 5$, premium on average outstanding $\frac{W + \text{EON}(T)}{2}$.

**Tranche A: $[0, 3]$, $W = 0.03$**

Scenario TLs:

| $L$ | TL |
|:---:|:--:|
| 0% | 0 |
| 2% | 0.02 |
| 6% | 0.03 (cap) |
| 15% | 0.03 (cap) |

$$\text{ETL} = 0.5 \cdot 0 + 0.2 \cdot 0.02 + 0.2 \cdot 0.03 + 0.1 \cdot 0.03 = 0.004 + 0.006 + 0.003 = 0.013$$

$$\text{EON} = 0.03 - 0.013 = 0.017$$

$$PV_{\text{prot}} = 0.82 \cdot 0.013 \cdot 100 = \$1.066\text{mm}$$

$$PV01 = 0.82 \cdot 5 \cdot \frac{0.03 + 0.017}{2} \cdot 100 = 0.82 \cdot 5 \cdot 0.0235 \cdot 100 = 0.82 \cdot 11.75 = \$9.635\text{mm}$$

$$s^* = \frac{1.066}{9.635} = 0.1106 \Rightarrow \boxed{1106 \text{ bp/year}}$$

**Tranche B: $[3, 7]$, $W = 0.04$** (from Example 3)

- $\text{ETL} = 0.010$, $\text{EON} = 0.030$

$$PV_{\text{prot}} = 0.82 \cdot 0.010 \cdot 100 = \$0.820\text{mm}$$

$$PV01 = 0.82 \cdot 5 \cdot \frac{0.04 + 0.03}{2} \cdot 100 = 0.82 \cdot 5 \cdot 0.035 \cdot 100 = 0.82 \cdot 17.5 = \$14.35\text{mm}$$

$$s^* = \frac{0.820}{14.35} = 0.0571 \Rightarrow \boxed{571 \text{ bp/year}}$$

**Interpretation:** Lower attachment $[0, 3]$ is riskier (higher ETL relative to width), so higher par spread.

---

### Example 10: Tail Shift Scenario — Senior Tranche Reacts More to Tail-Heavy Losses (One-Period Toy)

**Baseline Distribution (Example 3):**

$L = \{0, 2, 6, 15\}\%$, $p = \{0.5, 0.2, 0.2, 0.1\}$

**Tail-Heavy Distribution:**

Same losses, $p = \{0.4, 0.2, 0.1, 0.3\}$

Use $T = 5$, $Z = 0.82$, $\alpha = 5$, average-outstanding approximation.

**Senior Tranche $[10, 15]$, $W = 0.05$**

Scenario TL:
- $0, 2, 6\%$ are below 10% $\Rightarrow$ TL = 0
- $15\%$ $\Rightarrow$ TL = 0.05 (fully wiped)

**Baseline ETL:**

$$0.1 \cdot 0.05 = 0.005$$

Baseline EON $= 0.045$; avg outstanding $= (0.05 + 0.045)/2 = 0.0475$.

$$PV_{\text{prot}} = 0.82 \cdot 0.005 \cdot 100 = \$0.410\text{mm}$$

$$PV01 = 0.82 \cdot 5 \cdot 0.0475 \cdot 100 = 0.82 \cdot 23.75 = \$19.475\text{mm}$$

$$s^* = 0.410 / 19.475 = 0.02105 \Rightarrow \boxed{210.5 \text{ bp}}$$

**Tail-Heavy ETL:**

$$0.3 \cdot 0.05 = 0.015$$

EON $= 0.035$; avg $= (0.05 + 0.035)/2 = 0.0425$.

$$PV_{\text{prot}} = 0.82 \cdot 0.015 \cdot 100 = \$1.230\text{mm}$$

$$PV01 = 0.82 \cdot 5 \cdot 0.0425 \cdot 100 = 0.82 \cdot 21.25 = \$17.425\text{mm}$$

$$s^* = 1.230 / 17.425 = 0.0706 \Rightarrow \boxed{706 \text{ bp}}$$

**Equity Tranche $[0, 3]$, $W = 0.03$**

Baseline $s^* \approx 1106$ bp (Example 9).

**Tail-Heavy ETL:**

TLs: $0 \to 0$, $2\% \to 0.02$, $6\% \to 0.03$, $15\% \to 0.03$

$$\text{ETL} = 0.4 \cdot 0 + 0.2 \cdot 0.02 + 0.1 \cdot 0.03 + 0.3 \cdot 0.03 = 0.004 + 0.003 + 0.009 = 0.016$$

EON $= 0.014$; avg $= (0.03 + 0.014)/2 = 0.022$.

$$PV_{\text{prot}} = 0.82 \cdot 0.016 \cdot 100 = \$1.312\text{mm}$$

$$PV01 = 0.82 \cdot 5 \cdot 0.022 \cdot 100 = 0.82 \cdot 11 = \$9.02\text{mm}$$

$$s^* = 1.312 / 9.02 = 0.1455 \Rightarrow \boxed{1455 \text{ bp}}$$

**Takeaway:** Tail-heavy probability mass increases senior spread dramatically (210 → 706 bp) because senior ETL depends almost entirely on extreme-loss scenarios.

---

### Example 11: Recovery Sensitivity Toy — Higher Recovery → Lower Portfolio Loss → Lower ETL and Par Spread

Baseline (Example 3) implicitly corresponds to some LGD. Now suppose recoveries rise so that loss severities scale down by factor $50/60 = 0.8333$ (toy scaling).

- Baseline loss grid: $\{0, 2, 6, 15\}\%$
- Scaled loss grid: $\{0, 1.667, 5, 12.5\}\%$
- Keep probabilities $\{0.5, 0.2, 0.2, 0.1\}$

**Tranche $[3, 7]$, $W = 4\%$.**

Compute TL under scaled losses:

| Scaled $L$ | TL |
|:----------:|:--:|
| 0% | 0 |
| 1.667% (< 3%) | 0 |
| 5% | $5 - 3 = 2\% \Rightarrow 0.02$ |
| 12.5% | capped at $W = 0.04$ |

$$\text{ETL} = 0.2 \cdot 0.02 + 0.1 \cdot 0.04 = 0.004 + 0.004 = 0.008$$

EON $= 0.04 - 0.008 = 0.032$; avg $= (0.04 + 0.032)/2 = 0.036$.

One-period PVs ($T = 5$, $Z = 0.82$, $\alpha = 5$):

$$PV_{\text{prot}} = 0.82 \cdot 0.008 \cdot 100 = \$0.656\text{mm}$$

$$PV01 = 0.82 \cdot 5 \cdot 0.036 \cdot 100 = 0.82 \cdot 18 = \$14.76\text{mm}$$

$$s^* = \frac{0.656}{14.76} = 0.04444 \Rightarrow \boxed{444 \text{ bp}}$$

Compare to baseline $[3, 7]$ spread from Example 9 ($\approx 571$ bp): higher recovery lowers ETL and spread.

---

### Example 12: Consistency Checks — Bounds and PV Signs Under a Clear Convention

Adopt **protection-buyer convention:**

- Protection leg PV is a benefit (positive)
- Premium leg PV is a cost (negative)
- Net PV (ignoring upfront) is:

$$PV_{\text{net}} = PV_{\text{prot}} - PV_{\text{prem}}$$

Use Example 6–8 with par spread $s^* = 707.8$ bp:

- By construction, $PV_{\text{prot}} = s^* \cdot PV01$, so $PV_{\text{net}} \approx 0$ at initiation.

**Bound Checks (from Example 5):**

- $W = 0.04$, $\text{ETL}(5) = 0.012$ satisfies $0 \le \text{ETL} \le W$.
- $\text{EON}(5) = 0.028$ satisfies $0 \le \text{EON} \le W$.
- In any scenario, $\text{TL}(L) \in [0, W]$ and $\text{ON}(L) \in [0, W]$ (Example 1 table demonstrates this numerically).

**Sign Sanity:**

If $\text{ETL}$ increases (more expected loss), $PV_{\text{prot}}$ increases (more expected payouts), while premium PV01 typically decreases (less outstanding to pay on), both pushing fair $s^*$ upward—consistent with the sources' intuition that premium payments decline as tranche losses rise.

---

## 9. Practical Notes

### Production Checklist

1. Define the portfolio (names, notionals, recovery/LGD assumptions, maturity).
2. Define tranche $[A, D]$ and width $W = D - A$.
3. Specify payment dates $\{t_i\}$ and accruals $\{\alpha_i\}$ (sources: often quarterly, Act/360).
4. Choose discount curve $Z(t)$.
5. Provide loss distribution input:
   - Full distribution of $L(t_i)$, or
   - ETL curve $\text{ETL}(t_i)$, or
   - Tranche survival curve $Q(t_i)$ (expected surviving tranche notional fraction).
6. Compute:
   - $PV_{\text{prot}} = \sum Z(t_i)(\text{ETL}_i - \text{ETL}_{i-1}) N_{\text{port}}$
   - $PV01_{\text{prem}} = \sum \alpha_i Z(t_i) E[\text{avg ON}] N_{\text{port}}$
   - $s^* = PV_{\text{prot}} / PV01_{\text{prem}}$

### Common Pitfalls

- Mixing portfolio % units and \$ units (always track whether you're in fractions or dollars).
- Confusing tranche width $W$ (a fraction of the portfolio) with the portfolio notional $N_{\text{port}}$.
- Forgetting premium is on outstanding tranche notional (not initial notional).
- Using inconsistent discounting or accrual conventions.
- Treating ETL as linear in correlation/dependence—typically it is not (preview only; full treatment later).

### Verification Tests

- $\text{TL}(L)$ is monotone nondecreasing and bounded in $[0, W]$.
- $\text{ETL}(t)$ is nondecreasing in $t$ and bounded by $W$.
- Repricing check: using $s^*$ gives $PV_{\text{prot}} \approx PV_{\text{prem}}$ (net PV near 0) at initiation.

---

## 10. Summary & Recall

### 10-Bullet Executive Summary

1. **Portfolio cumulative loss** $L(t)$ (fraction of portfolio notional) is the state variable for tranche payoffs.

2. **Tranche** $[A, D]$ has width $W = D - A$; it absorbs losses only between attachment and detachment.

3. **Tranche loss amount** (portfolio fraction) is $\text{TL}(L) = \min(\max(L - A, 0), W)$.

4. **Outstanding tranche notional** (portfolio fraction) is $\text{ON}(L) = W - \text{TL}(L)$.

5. **Normalized tranche loss fraction** is $L_{\text{tr}}(t) = \text{TL}(L(t))/W$, matching the sources' definition.

6. A **model delivers the distribution** of $L(t)$ (or equivalently ETL/Q curves) required for pricing.

7. **Expected tranche loss** is $\text{ETL}(t) = E[\text{TL}(L(t))]$; **expected outstanding** is $\text{EON}(t) = W - \text{ETL}(t)$.

8. **Protection leg PV** is an expected discounted loss: $\sum Z(t_i) \Delta\text{ETL}_i$.

9. **Premium leg PV** is spread times discounted expected outstanding, with mid-period average notional approximation in the sources.

10. **Par spread** is $s^* = PV_{\text{prot}} / PV01_{\text{prem}}$; senior tranches are tail-driven while equity is body/mean-driven.

---

### Cheat Sheet: Key Formulas (with Units)

**Portfolio loss:**
$$L(t) \in [0, L_{\max}] \quad \text{(unitless fraction)}$$

**Width:**
$$W = D - A \quad \text{(unitless fraction)}$$

**Tranche loss amount:**
$$\boxed{\text{TL}(L) = \min(\max(L - A, 0), W)} \quad \text{(fraction of portfolio)}$$

**Outstanding:**
$$\boxed{\text{ON}(L) = W - \text{TL}(L)} \quad \text{(fraction of portfolio)}$$

**Normalized tranche loss:**
$$L_{\text{tr}}(t) = \frac{\text{TL}(L(t))}{W} \in [0, 1]$$

(Matches sources' $L(T, K_1, K_2)$.)

**ETL:**
$$\boxed{\text{ETL}(t) = E[\text{TL}(L(t))]} \quad \text{(fraction of portfolio)}$$

**Expected outstanding:**
$$\boxed{\text{EON}(t) = W - \text{ETL}(t)} \quad \text{(fraction of portfolio)}$$

**Protection PV (discrete):**
$$\boxed{PV_{\text{prot}}(\$) = N_{\text{port}} \sum_i Z(t_i)\left(\text{ETL}(t_i) - \text{ETL}(t_{i-1})\right)}$$

**Premium PV (average-notional approximation):**
$$\boxed{PV_{\text{prem}}(\$) = s \, N_{\text{port}} \sum_i \alpha_i Z(t_i) E\!\left[\frac{\text{ON}(t_{i-1}) + \text{ON}(t_i)}{2}\right]}$$

**Par spread:**
$$\boxed{s^* = \frac{PV_{\text{prot}}(\$)}{PV01_{\text{prem}}(\$)}} \quad \text{(1/year; convert to bp/year)}$$

---

### 30 Flashcards (Q/A)

1. **Q:** What is the tranche pricing state variable in this chapter?
   **A:** Portfolio loss $L(t)$ as a fraction of portfolio notional.

2. **Q:** Define tranche width.
   **A:** $W = D - A$.

3. **Q:** Define tranche loss amount $\text{TL}(L)$.
   **A:** $\min(\max(L - A, 0), W)$.

4. **Q:** What are the units of $\text{TL}(L)$?
   **A:** Fraction of portfolio notional.

5. **Q:** Define tranche outstanding $\text{ON}(L)$.
   **A:** $W - \text{TL}(L)$.

6. **Q:** What is normalized tranche loss $L_{\text{tr}}(t)$?
   **A:** $\text{TL}(L(t))/W \in [0, 1]$.

7. **Q:** How do the sources define fractional tranche loss at $T$?
   **A:** As a max–max linear payoff over $[K_1, K_2]$.

8. **Q:** Define ETL.
   **A:** $\text{ETL}(t) = E[\text{TL}(L(t))]$.

9. **Q:** Define expected outstanding.
   **A:** $\text{EON}(t) = W - \text{ETL}(t)$.

10. **Q:** What is tranche survival $Q(t)$?
    **A:** $Q(t) = E[1 - L_{\text{tr}}(t)]$.

11. **Q:** Relationship between $Q(t)$ and $\text{EON}(t)$?
    **A:** $Q(t) = \text{EON}(t)/W$.

12. **Q:** What does the premium leg pay on?
    **A:** Surviving/outstanding tranche notional.

13. **Q:** Why do premium payments decline after losses?
    **A:** Outstanding notional shrinks.

14. **Q:** What does the protection leg pay?
    **A:** Realized tranche losses (increments of tranche loss).

15. **Q:** Discrete protection PV in ETL terms?
    **A:** $\sum Z(t_i)(\text{ETL}_i - \text{ETL}_{i-1}) N_{\text{port}}$.

16. **Q:** Premium PV01 meaning?
    **A:** PV of paying 1 unit of spread on expected outstanding.

17. **Q:** Par spread formula?
    **A:** $s^* = PV_{\text{prot}} / PV01_{\text{prem}}$.

18. **Q:** If ETL is identically zero, what is $s^*$?
    **A:** $0$.

19. **Q:** Why are senior tranches tail-sensitive?
    **A:** They pay only when losses exceed high attachment.

20. **Q:** Why is mezzanine often "kinked" risk?
    **A:** It depends strongly on whether loss distribution reaches its attachment.

21. **Q:** What mass points can appear in tranche loss distribution?
    **A:** Mass at 0 loss and at 100% loss.

22. **Q:** How is unconditional loss density built in a one-factor model?
    **A:** Integrate conditional loss density over factor density.

23. **Q:** Why keep ETL in portfolio units?
    **A:** It makes aggregation and \$ conversion simple (multiply by $N_{\text{port}}$).

24. **Q:** Convert ETL to \$ loss.
    **A:** $N_{\text{port}} \cdot \text{ETL}$.

25. **Q:** Convert normalized tranche loss to \$ loss.
    **A:** $N_{\text{tr}} \cdot L_{\text{tr}}$ where $N_{\text{tr}} = N_{\text{port}} W$.

26. **Q:** What input objects are sufficient to price a tranche (in this chapter)?
    **A:** Discount curve + ETL/Q curve on coupon dates.

27. **Q:** What is not done here?
    **A:** Correlation/base correlation calibration.

28. **Q:** Why is premium-on-average-notional used?
    **A:** To approximate intra-period loss timing.

29. **Q:** What additional info is needed for exact accrual/settlement?
    **A:** Trade documentation (index vs bespoke, settlement, day count, accrual conventions).

30. **Q:** What is the main conceptual summary?
    **A:** Tranche is a nonlinear transform of portfolio loss; ETL drives both legs' PV.

---

## 11. Mini Problem Set (18 Questions)

*(Brief solution sketches for questions 1–9 only.)*

---

**1. Compute TL/ON:**

For tranche $[A, D] = [2\%, 5\%]$, compute $\text{TL}(L)$ and $\text{ON}(L)$ at $L = \{0\%, 2\%, 4\%, 5\%, 8\%\}$.

*Sketch:* Use $\text{TL} = \min(\max(L - A, 0), W)$ with $W = 3\%$; then $\text{ON} = W - \text{TL}$.

---

**2. Normalized vs unnormalized loss:**

Show that the sources' $L(T, K_1, K_2)$ equals $\text{TL}(L(T))/W$.

*Sketch:* Multiply the sources' definition by $W = K_2 - K_1$ to get the unnormalized $\max - \max$ form.

---

**3. ETL from scenarios:**

Given $L(T) \in \{1\%, 4\%, 9\%\}$ with probs $\{0.3, 0.5, 0.2\}$, compute $\text{ETL}(T)$ for tranche $[3\%, 7\%]$.

*Sketch:* Compute TL per scenario: $1\% \to 0$, $4\% \to 1\%$, $9\% \to 4\%$; weight by probabilities.

---

**4. EON and Q:**

For a tranche with width $W$, show $Q(T) = \text{EON}(T)/W$.

*Sketch:* $Q = E[1 - L_{\text{tr}}] = E[1 - \text{TL}/W] = E[\text{ON}/W] = \text{EON}/W$.

---

**5. Protection PV from ETL increments:**

Given ETL(1) = 0.2%, ETL(2) = 0.5%, ETL(3) = 0.7% and discount factors 0.98, 0.95, 0.92, compute protection PV for $N_{\text{port}} = \$200$mm.

*Sketch:* Compute increments $(0.2, 0.3, 0.2)\%$; PV fraction $= \sum Z \cdot \Delta\text{ETL}$; multiply by \$200mm.

---

**6. Premium PV01:**

With accruals $\alpha_i = 1$ yearly, discount factors as in (5), and expected outstanding $\text{EON}(0) = 4\%$, $\text{EON}(1) = 3.8\%$, $\text{EON}(2) = 3.6\%$, $\text{EON}(3) = 3.5\%$, compute PV01 using average outstanding.

*Sketch:* For each year use avg outstanding $(\text{EON}_{i-1} + \text{EON}_i)/2$, sum $Z \cdot \alpha \cdot \text{avg}$, multiply by $N_{\text{port}}$.

---

**7. Par spread:**

Using your answers to (5) and (6), compute $s^*$ in bp/year.

*Sketch:* $s^* = PV_{\text{prot}} / PV01$; multiply by $10^4$.

---

**8. Tail sensitivity:**

For a senior tranche with $A = 10\%$, explain why increasing $\Pr(L(T) \ge D)$ can dominate spread even if $E[L(T)]$ changes little.

*Sketch:* Senior TL is nearly 0 unless $L$ reaches attachment; tail probability controls expected loss.

---

**9. Bounds check:**

Prove $0 \le \text{ETL}(t) \le W$.

*Sketch:* Since $0 \le \text{TL}(L(t)) \le W$ almost surely, expectation preserves bounds.

---

**10.** Show how the tranche loss distribution can have an atom at 0 and at 100% loss, and express these masses via $F_{L(T)}$.

**11.** Derive the identity $\text{TL}(L) = \min(L, D) - \min(L, A)$.

**12.** Suppose a tranche has $D > L_{\max}$. Explain what breaks in the "mapping to a CDS with zero recovery" and what extra modeling is needed.

**13.** If $\text{ETL}(t)$ is nondecreasing, what does this imply about expected outstanding and premium PV01?

**14.** Describe how conditional independence in a one-factor model simplifies building $f(L(T)|Z)$.

**15.** Consider two tranches with same width but different attachments. Under what condition must the more subordinated tranche have lower ETL?

**16.** Construct a two-period example where $\text{ETL}(T)$ is the same but PV differs due to timing of losses.

**17.** Explain how different recovery assumptions change $L_{\max}$ and why this matters for super-senior tranches.

**18.** Explain why premium leg and protection leg of tranches across the whole capital structure do not perfectly hedge CDS premium legs even if protection legs aggregate.

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

- Tranche loss function definition (max–max form): O'Kane Ch 11–12
- Tranche survival $Q(t)$ definition: O'Kane Ch 12–13
- ETL curve $\psi(T, K) = E[\min(L(T), K)]$: O'Kane Ch 12
- Premium leg on surviving notional, protection leg as discounted expected loss: O'Kane Ch 12
- Mid-period average notional approximation for premium accrual: O'Kane Ch 12
- Tranche loss distribution mass at 0 and 100%: O'Kane Ch 12–13
- Factor model for building $f(L(T)|Z)$: O'Kane Ch 13–14

### (B) Reasoned Inference — Note Derivation Logic

- Unit conversions between portfolio-fraction and dollar amounts: algebraic consistency with source definitions
- Limiting cases (ETL = 0, tranche wiped immediately): derived from PV formulas
- Tail sensitivity of senior vs body sensitivity of equity: inferred from tranche loss function shape and ETL definition

### (C) Speculation — Flag Uncertainties

- Exact accrual/settlement conventions for specific index tranches or bespoke tranches: I'm not sure without trade documentation
- Pay-as-you-go vs standard settlement nuances: I'm not sure beyond what sources state
- Edge case behavior when tranche wiped at $t = 0$: settlement mechanics unclear without documentation
