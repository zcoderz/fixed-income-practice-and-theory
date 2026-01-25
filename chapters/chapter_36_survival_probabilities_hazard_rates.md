# Chapter 36: Survival Probabilities and Hazard Rates (Default Intensities)

---

## Conventions & Notation

### Time and Default

| Symbol | Definition |
|--------|------------|
| $t \geq 0$ | Time in years unless stated otherwise |
| $\Delta t$ | Year-fraction |
| $\tau \in (0, \infty)$ | Default time — the (random) time of default (a one-time event) |

### Measures

| Symbol | Definition |
|--------|------------|
| $\mathbb{Q}$ | Risk-neutral (pricing) measure when explicitly discussing pricing. O'Kane frames reduced-form credit modeling as a risk-neutral approach for pricing |
| $\mathbb{P}$ | Real-world/physical measure when discussing historical default rates |

If not explicitly specified, results are stated generically and then specialized to $\mathbb{Q}$ when discussing spreads.

### Survival and Default

| Symbol | Definition |
|--------|------------|
| $Q(t) := \Pr(\tau > t)$ | Survival function — probability of surviving to time $t$. O'Kane also uses $Q(0,T)$ for survival from 0 to $T$ |
| $F(t) := \Pr(\tau \leq t) = 1 - Q(t)$ | Default CDF — cumulative default probability by time $t$ |

### Hazard and Density

| Symbol | Definition |
|--------|------------|
| $h(t)$ or $\lambda(t)$ | Hazard rate / intensity, with units 1/year |
| $H(t) := \int_0^t h(u)\,du$ | Cumulative hazard (dimensionless) |
| $f(t)$ | Default density, with units 1/year |

### Recovery and Spread

| Symbol | Definition |
|--------|------------|
| $R \in [0,1]$ | Recovery fraction; loss given default $\text{LGD} = 1 - R$ |
| $S$ or $s$ | Continuously paid spread in decimal per year (e.g., 120 bp $= 0.012$) when using the "credit triangle" approximation |
| $Z(0,t)$ | Risk-free discount factor (O'Kane notation) |

---

## Setup

### Conventions Used in This Chapter

We treat $\tau$ as a single default time and model its distribution via a survival curve and hazard rate (reduced-form / intensity approach).

When connecting to market spreads, we default to the risk-neutral lens (since spreads come from pricing), but we explicitly flag $\mathbb{P}$ vs $\mathbb{Q}$ where it matters.

### Notation Glossary (Symbols + Definitions)

| Symbol | Definition |
|--------|------------|
| $\tau$ | Default time random variable |
| $Q(t) = \Pr(\tau > t)$ | Survival probability to time $t$ |
| $F(t) = 1 - Q(t) = \Pr(\tau \leq t)$ | Cumulative default probability by $t$ |
| $h(t)$ or $\lambda(t)$ | Hazard rate / default intensity at time $t$, defined as an instantaneous conditional default probability per unit time |
| $H(t) = \int_0^t h(u)\,du$ | Cumulative hazard |
| $f(t)$ | Default time density; for absolutely continuous $F$, $f(t) = F'(t)$ |
| $R$ | Recovery fraction; $1 - R$ is loss fraction used in protection payments |
| $Z(0,t)$ | Risk-free discount factor |

---

## Fact Classification

### (A) Verified Facts (Directly Supported by Sources)

- Hazard/intensity is defined as an instantaneous conditional default probability rate
- Survival can be expressed in intensity form $Q(0,T) = \mathbb{E}\left[\exp\left(-\int_0^T \lambda(t)\,dt\right)\right]$ in reduced-form pricing frameworks
- Under (deterministic) hazard rates, survival is exponential in cumulative hazard; in Hull's notation $V(t) = \exp\left(-\int_0^t \lambda(u)\,du\right)$
- O'Kane's "credit triangle" (under stated simplifying assumptions) gives $S = \lambda(1-R)$

### (B) Reasoned Inference (Derived Consequences from (A))

- Piecewise-constant hazard conversion formulas; interval PDs; density $f(t) = h(t)Q(t)$; expected loss formulas

### (C) Speculation (Clearly Labeled; Minimal)

- None needed beyond clearly labeled "toy approximations" where assumptions are stronger than desk reality

---

## Core Concepts (Definitions First)

### 1.1 Default Time $\tau$

**Formal Definition:**

$\tau$ is a random time representing the (first) occurrence of default; reduced-form modeling treats it as a one-time event with $\tau \in (0, \infty)$.

**Intuition:**

Instead of modeling firm value and a default barrier (structural), we model the arrival time of default directly (an intensity-driven "surprise" event).

**Trading/Risk Practice:**

- Default-time modeling is the backbone of CDS/bond analytics: PVs depend on whether the name survives to payment dates or defaults before them
- "Default is a surprise" is a key feature of hazard models (useful for fitting non-zero short-end spreads)

---

### 1.2 Survival Function $Q(t)$

**Formal Definition:**

$Q(t) = \Pr(\tau > t)$. In O'Kane's reduced-form pricing setup, survival probabilities appear as $Q(0,T)$ and can be represented via intensity as an exponential-of-integral expression (or expectation of such).

**Intuition:**

$Q(t)$ is the "alive" probability to $t$; it is decreasing, starts at $Q(0) = 1$, and acts like a credit discount factor.

**Trading/Risk Practice:**

- A survival curve (term structure of $Q(0,t)$) is what your pricer wants as input for credit legs (premium/protection) and expected loss
- Risk managers translate curve moves into default probability moves (especially for scenario analysis / stress)

---

### 1.3 Hazard Rate / Default Intensity $h(t)$ or $\lambda(t)$

**Formal Definition:**

O'Kane defines $\lambda(t)$ via the instantaneous conditional default probability rate:

$$\boxed{\lambda(t) = \lim_{\Delta t \to 0} \frac{1}{\Delta t} \Pr(\tau \leq t + \Delta t \mid \tau > t)}$$

QRM gives the same concept as the hazard rate $\gamma(t)$ of a random time $\tau$.

**Intuition:**

"Conditional default probability per year, given survival so far." If $h(t) = 2\%$, then over a short interval $\Delta t$, conditional PD $\approx 0.02\,\Delta t$.

**Trading/Risk Practice:**

- Desk shorthand often treats intensity as "instantaneous credit spread" (with many caveats). Brigo–Mercurio emphasize the analogy: intensity enters discounting like an additive spread to $r$
- Calibrations often solve for a piecewise-constant hazard curve from CDS quotes (next chapter)

---

### 1.4 Default Density $f(t)$ and Cumulative Hazard $H(t)$

**Formal Definition:**

QRM: cumulative hazard $\Gamma(t) = -\ln(\bar{F}(t))$ and hazard rate $\gamma(t) = f(t)/\bar{F}(t)$, where $\bar{F}(t) = \Pr(\tau > t)$.

Translate to our notation: $\bar{F}(t) = Q(t)$, $\Gamma(t) = H(t)$, $\gamma(t) = h(t)$.

**Intuition:**

$H(t)$ accumulates default "pressure" over time; $Q(t) = e^{-H(t)}$ is exponential decay in that pressure.

**Trading/Risk Practice:**

Densities matter whenever cashflow timing depends on default time (recovery payments, protection legs). O'Kane notes that pricing payments at default requires the full default-time density, not just $\Pr(\tau \leq T)$.

---

## Math and Derivations (Step-by-Step)

### 2.1 From Hazard Definition to the Survival ODE

Start from the hazard definition (O'Kane/QRM):

$$h(t) = \lim_{\Delta t \to 0} \frac{\Pr(t < \tau \leq t + \Delta t \mid \tau > t)}{\Delta t}$$

For small $\Delta t$:

$$\Pr(t < \tau \leq t + \Delta t \mid \tau > t) \approx h(t)\,\Delta t$$

Multiply both sides by $\Pr(\tau > t) = Q(t)$:

$$\Pr(t < \tau \leq t + \Delta t) \approx Q(t)\,h(t)\,\Delta t$$

Now connect to the survival decrement:

$$Q(t + \Delta t) = \Pr(\tau > t + \Delta t) = \Pr(\tau > t) - \Pr(t < \tau \leq t + \Delta t) \approx Q(t) - Q(t)h(t)\,\Delta t$$

Rearrange:

$$\frac{Q(t + \Delta t) - Q(t)}{\Delta t} \approx -h(t)Q(t)$$

Take the limit $\Delta t \to 0$:

$$\boxed{\frac{dQ(t)}{dt} = -h(t)Q(t)}$$

This matches the standard derivation shown in Hull's text (he uses $V(t)$ for survival):

$$\frac{dV(t)}{dt} = -\lambda(t)V(t)$$

**Unit Check:**

- LHS $dQ/dt$ has units $1/\text{year}$ (since $Q$ is dimensionless)
- RHS $(-h) \times Q$ has $(1/\text{year}) \times (1)$ ✓

---

### 2.2 Solve the ODE: Survival in Terms of Cumulative Hazard

Solve:

$$\frac{dQ(t)}{Q(t)} = -h(t)\,dt$$

Integrate from $0$ to $t$, using $Q(0) = 1$:

$$\ln Q(t) - \ln Q(0) = -\int_0^t h(u)\,du$$

$$\boxed{Q(t) = \exp\left(-\int_0^t h(u)\,du\right) = e^{-H(t)}}$$

This exponential-of-integral form for survival appears directly in the sources (Hull: $V(t) = e^{-\int_0^t \lambda}$). QRM expresses the same idea via cumulative hazard $\Gamma(t) = \int_0^t \gamma(s)\,ds$ and $\bar{F}(t) = e^{-\Gamma(t)}$.

Equivalently:

$$\boxed{h(t) = -\frac{d}{dt}\ln Q(t)}$$

(Immediate from differentiating $\ln Q(t) = -H(t)$.)

**Sanity Checks:**

- If $h(t) \geq 0$, then $H(t)$ is increasing and $Q(t)$ is non-increasing in $t$
- If $h(t) = 0$, then $Q(t) \equiv 1$ (no default risk)
- If $h(t) = \lambda$ constant, then $Q(t) = e^{-\lambda t}$ (exponential survival), consistent with QRM's exponential example

---

### 2.3 Default Density $f(t)$ and Identity $f(t) = h(t)Q(t)$

Assuming $F(t) = \Pr(\tau \leq t)$ is absolutely continuous, $f(t) = F'(t)$. QRM defines hazard:

$$h(t) = \frac{f(t)}{1 - F(t)} = \frac{f(t)}{Q(t)}$$

$$\boxed{f(t) = h(t)Q(t)}$$

A closely related identity appears in O'Kane's pricing manipulations: he uses

$$dQ(0,t) = -\lambda(t)Q(0,t)\,dt$$

which is the differential form of $Q'(t) = -\lambda(t)Q(t)$.

**Unit Check:** $f(t)$ is a density over time $\Rightarrow 1/\text{year}$. RHS $h(t)Q(t)$ is $(1/\text{year}) \times 1$ ✓

---

### 2.4 Interval Default Probability

Using $Q(t) = \Pr(\tau > t)$:

$$\boxed{\Pr(t_1 < \tau \leq t_2) = \Pr(\tau > t_1) - \Pr(\tau > t_2) = Q(t_1) - Q(t_2)}$$

(Direct probability algebra; no extra modeling assumption.)

---

### 2.5 Risk-Neutral Survival Representation in Reduced-Form Pricing

O'Kane shows that under simplifying assumptions (e.g., independence between short rate and hazard), survival enters pricing as:

$$Q(0,T) = \mathbb{E}\left[\exp\left(-\int_0^T \lambda(t)\,dt\right)\right]$$

Brigo–Mercurio similarly relate $\mathbb{Q}\{\tau > t\}$ to an exponential of integrated intensity, emphasizing the discount-factor analogy.

**Important Measure Note:**

O'Kane explicitly frames reduced-form models as a risk-neutral modeling approach for pricing. So when we infer hazard from market spreads, it is typically a $\mathbb{Q}$-hazard, not a historical ($\mathbb{P}$) hazard. Hull (OFOD) states that hazard rates implied from credit spreads are risk-neutral estimates and contrasts them with historical (real-world) default probabilities.

---

## Measurement & Risk (Only What Belongs in Chapter 36)

### A) Default Time and Survival

**Default time $\tau$:** one-time event default time.

**Survival function:**

$$\boxed{Q(t) = \Pr(\tau > t)}$$

In reduced-form pricing contexts, survival commonly takes the exponential-integral form (or expectation of it).

**Default probability over an interval:**

$$\boxed{\Pr(t_1 < \tau \leq t_2) = Q(t_1) - Q(t_2)}$$

**Measure clarity:**

- If $Q(t)$ is built from market spreads, it is generally risk-neutral (pricing) $Q^{\mathbb{Q}}(t)$
- If $Q(t)$ is built from historical default data, it is real-world $Q^{\mathbb{P}}(t)$
- If you do not specify which, you risk mixing apples (pricing) and oranges (risk forecasting)

---

### B) Hazard Rate / Intensity

**Definition (instantaneous conditional default rate):**

$$\boxed{h(t) = \lim_{\Delta t \to 0} \frac{\Pr(t < \tau \leq t + \Delta t \mid \tau > t)}{\Delta t}}$$

**Relationship to survival:**

$$\boxed{h(t) = -\frac{d}{dt}\ln Q(t)}$$

$$\boxed{Q(t) = \exp\left(-\int_0^t h(u)\,du\right)}$$

(Consistent with Hull's survival ODE and solution.)

**Default density:**

$$\boxed{f(t) = h(t)Q(t)}$$

**Cumulative hazard:**

$$\boxed{H(t) = \int_0^t h(u)\,du, \qquad Q(t) = e^{-H(t)}}$$

---

### C) Discrete-Time / Piecewise-Constant Hazards (Practitioner Workhorse)

Assume a time grid $0 = t_0 < t_1 < \cdots < t_n$ and set hazard piecewise-constant:

$$h(t) = h_i \quad \text{for } t \in [t_{i-1}, t_i), \qquad \Delta t_i = t_i - t_{i-1}$$

Then:

$$\boxed{H(t_i) = H(t_{i-1}) + h_i \Delta t_i}$$

$$\boxed{Q(t_i) = Q(t_{i-1})\,e^{-h_i \Delta t_i}}$$

**Invert:**

$$\boxed{h_i = -\frac{1}{\Delta t_i}\ln\left(\frac{Q(t_i)}{Q(t_{i-1})}\right)}$$

**Unit Checks:**

- $h_i$ is 1/year
- $\Delta t_i$ is years
- $h_i \Delta t_i$ is dimensionless, so the exponential is valid ✓

---

### D) Intuition Bridge: Spread ↔ Default Intensity (Conceptual)

A widely used (but assumption-heavy) bridge is the **credit triangle**:

$$\boxed{S = \lambda(1 - R) \quad \Rightarrow \quad \lambda = \frac{S}{1-R}}$$

O'Kane derives this by comparing:

- A continuously paid premium stream $S\,dt$ received only if the name survives, and
- A protection payment of $(1-R)$ at default

under the simplifying assumptions:

1. Continuous premium payments
2. Protection payment proportional to $(1-R)$
3. (For the final simplification) constant hazard rate $\lambda$

**Interpretation:** If you earn spread $S$ continuously, you are "funding" expected losses of size $(1-R)$ that arrive at intensity $\lambda$.

**Caveats (Desk Reality):**

- Observed "spreads" often embed risk premia and liquidity/funding components; O'Kane explicitly decomposes spreads into actuarial spread plus several premia (default risk, volatility risk, liquidity)
- Risk-neutral intensities inferred from spreads tend to exceed historical ones (risk premia)

---

### E) Keep It Focused (Preview Only)

We do not derive full CDS par spread here.

**Preview only:** In CDS-style pricing, survival $Q(0,t)$ weights premium leg cashflows, and $-dQ(0,t)$ (or $\lambda(t)Q(0,t)\,dt$) weights protection leg/default cashflows.

---

## Worked Examples (At Least 10 Numeric Examples)

**Reminder on units:** Hazards in "per year," spreads in decimal per year, and time in years.

---

### Example A: Constant Hazard → Survival

**Given:** $h = 2\%$ per year $= 0.02$

For constant hazard:

$$Q(t) = e^{-ht} = e^{-0.02t}$$

**Compute:**

| $t$ | $Q(t) = e^{-0.02t}$ |
|-----|---------------------|
| 1 | $e^{-0.02} \approx 0.9801987$ |
| 3 | $e^{-0.06} \approx 0.9417645$ |
| 5 | $e^{-0.10} \approx 0.9048374$ |

**Probability of default by 5y:**

$$\Pr(\tau \leq 5) = 1 - Q(5) \approx 1 - 0.9048374 = 0.0951626$$

**Outputs:**

- $Q(1) \approx 0.980199$
- $Q(3) \approx 0.941765$
- $Q(5) \approx 0.904837$
- $\Pr(\tau \leq 5) \approx 9.516\%$

---

### Example B: Survival → Hazard

**Given:** $Q(1) = 0.98$, $Q(5) = 0.90$, and $Q(0) = 1$

**(i) Implied average constant hazard over $[0,5]$**

Assume constant hazard $\bar{h}_{0,5}$ such that:

$$Q(5) = e^{-\bar{h}_{0,5} \cdot 5} \quad \Rightarrow \quad \bar{h}_{0,5} = -\frac{1}{5}\ln(0.90)$$

Compute $\ln(0.90) \approx -0.1053605$:

$$\bar{h}_{0,5} \approx \frac{0.1053605}{5} = 0.0210721 \quad (= 2.10721\%/\text{yr})$$

**(ii) Implied average hazard over $[1,5]$**

On $[1,5]$ (length 4 years), assume constant $\bar{h}_{1,5}$:

$$\frac{Q(5)}{Q(1)} = e^{-\bar{h}_{1,5} \cdot 4} \quad \Rightarrow \quad \bar{h}_{1,5} = -\frac{1}{4}\ln\left(\frac{0.90}{0.98}\right)$$

Compute $\ln(0.90/0.98) = \ln(0.90) - \ln(0.98) \approx (-0.1053605) - (-0.0202027) = -0.0851578$:

$$\bar{h}_{1,5} \approx \frac{0.0851578}{4} = 0.0212895 \quad (= 2.12895\%/\text{yr})$$

**Outputs:**

- $\bar{h}_{0,5} \approx 2.107\%$ per year
- $\bar{h}_{1,5} \approx 2.129\%$ per year

---

### Example C: Interval Default Probability

Use Example A's constant hazard $h = 0.02$, so $Q(t) = e^{-0.02t}$.

**Compute:**

- $Q(2) = e^{-0.04} \approx 0.9607894$
- $Q(4) = e^{-0.08} \approx 0.9231163$

**Interval default probability:**

$$\Pr(2 < \tau \leq 4) = Q(2) - Q(4) \approx 0.9607894 - 0.9231163 = 0.0376731$$

**Interpretation:** About 3.77% chance of default occurring between years 2 and 4, given the model.

---

### Example D: Piecewise Hazard Construction

**Given:**

- $h(t) = 1\%$ for years $0$–$2$
- $h(t) = 3\%$ for years $2$–$5$

**Compute:**

For $t \leq 2$: $Q(t) = e^{-0.01t}$

For $t > 2$: $Q(t) = Q(2)\,e^{-0.03(t-2)}$

| $t$ | Calculation | $Q(t)$ |
|-----|-------------|--------|
| 1 | $e^{-0.01}$ | $\approx 0.9900498$ |
| 2 | $e^{-0.02}$ | $\approx 0.9801987$ |
| 3 | $Q(2) \cdot e^{-0.03} \approx 0.9801987 \times 0.9704455$ | $\approx 0.9512287$ |
| 5 | $Q(2) \cdot e^{-0.09} \approx 0.9801987 \times 0.9139312$ | $\approx 0.895834$ |

**Outputs:**

- $Q(1) \approx 0.990050$
- $Q(2) \approx 0.980199$
- $Q(3) \approx 0.951229$
- $Q(5) \approx 0.895834$

---

### Example E: Recover Hazard from Node Survivals

**Given:** Yearly nodes $Q(0) = 1$, $Q(1) = 0.99$, $Q(2) = 0.975$, $Q(3) = 0.955$

Assume piecewise-constant hazard on each year $[0,1), [1,2), [2,3)$ with $\Delta t = 1$.

**Use:**

$$h_i = -\ln\left(\frac{Q(i)}{Q(i-1)}\right)$$

**Year 0–1:**

$$h_1 = -\ln(0.99) \approx 0.0100503 \quad (1.0050\%/\text{yr})$$

**Year 1–2:**

$$h_2 = -\ln\left(\frac{0.975}{0.99}\right) = -\ln(0.9848485) \approx 0.0152675 \quad (1.5267\%/\text{yr})$$

**Year 2–3:**

$$h_3 = -\ln\left(\frac{0.955}{0.975}\right) = -\ln(0.9794872) \approx 0.0207257 \quad (2.0726\%/\text{yr})$$

**Outputs:**

- $h_1 \approx 1.005\%$, $h_2 \approx 1.527\%$, $h_3 \approx 2.073\%$ per year

---

### Example F: Default Density and "Integral $\approx 1 - Q(T)$" Check

#### F1) Constant Hazard

Let $h = 0.02$, $Q(t) = e^{-0.02t}$, $f(t) = hQ(t)$.

**Compute selected points:**

- $f(1) = 0.02 \times 0.9801987 \approx 0.0196040$
- $f(4) = 0.02 \times 0.9231163 \approx 0.0184623$

**Verify $\int_0^5 f(t)\,dt \approx 1 - Q(5)$ using a coarse trapezoid rule on $t = 0,1,2,3,4,5$:**

| $t$ | $f(t) = 0.02 \cdot Q(t)$ |
|-----|--------------------------|
| 0 | $0.02 \cdot 1 = 0.0200000$ |
| 1 | $\approx 0.0196040$ |
| 2 | $0.02 \cdot 0.9607894 \approx 0.0192158$ |
| 3 | $0.02 \cdot 0.9417645 \approx 0.0188353$ |
| 4 | $\approx 0.0184623$ |
| 5 | $0.02 \cdot 0.9048374 \approx 0.0180967$ |

**Trapezoid:**

$$\int_0^5 f(t)\,dt \approx \frac{1}{2}\left(f(0) + 2(f(1) + f(2) + f(3) + f(4)) + f(5)\right)$$

Compute inner sum:

$$f(1) + f(2) + f(3) + f(4) \approx 0.0196040 + 0.0192158 + 0.0188353 + 0.0184623 = 0.0761174$$

So:

$$\approx \frac{1}{2}(0.0200000 + 2(0.0761174) + 0.0180967) = \frac{1}{2}(0.1903315) = 0.0951658$$

**Exact survival at 5:**

$$1 - Q(5) = 1 - 0.9048374 = 0.0951626$$

**Close match ✓**

#### F2) Piecewise Hazard (from Example D)

Use midpoint Riemann sum with $\Delta t = 1$ over $[0,5]$. Midpoints: $0.5, 1.5, 2.5, 3.5, 4.5$.

Compute $f(t) = h(t)Q(t)$ at midpoints:

| $t$ | $h$ | $Q(t)$ | $f(t)$ |
|-----|-----|--------|--------|
| 0.5 | 0.01 | $e^{-0.005} \approx 0.9950125$ | $\approx 0.0099501$ |
| 1.5 | 0.01 | $e^{-0.015} \approx 0.9851119$ | $\approx 0.0098511$ |
| 2.5 | 0.03 | $Q(2)e^{-0.015} \approx 0.9656047$ | $\approx 0.0289681$ |
| 3.5 | 0.03 | $\approx 0.937067$ | $\approx 0.0281120$ |
| 4.5 | 0.03 | $\approx 0.909373$ | $\approx 0.0272812$ |

**Riemann sum:**

$$\int_0^5 f(t)\,dt \approx \sum f(\text{mid})\Delta t \approx 0.0099501 + 0.0098511 + 0.0289681 + 0.0281120 + 0.0272812 = 0.1041625$$

From Example D, $Q(5) \approx 0.895834$, so $1 - Q(5) \approx 0.104166$. **Close match ✓**

---

### Example G: Expected Loss Without Discounting

Assume recovery of par-style loss fraction $1-R$ (same loss fraction used in O'Kane's protection-leg setup).

**Given:** Notional $N = 10{,}000{,}000$, recovery $R = 40\% \Rightarrow (1-R) = 0.6$, horizon $T = 5$

**Scenario 1:** Constant hazard $h = 2\%$. From Example A, $1 - Q(5) \approx 0.0951626$.

$$EL(5) = N(1-R)(1-Q(5)) = 10{,}000{,}000 \times 0.6 \times 0.0951626 = 10{,}000{,}000 \times 0.0570976 \approx 570{,}976$$

**Scenario 2:** Constant hazard $h = 4\%$. Then $Q(5) = e^{-0.20} \approx 0.8187308$, so $1 - Q(5) \approx 0.1812692$.

$$EL(5) \approx 10{,}000{,}000 \times 0.6 \times 0.1812692 = 10{,}000{,}000 \times 0.1087615 \approx 1{,}087{,}615$$

**Outputs:**

- $EL_{2\%} \approx \$0.571\text{m}$
- $EL_{4\%} \approx \$1.088\text{m}$

---

### Example H: Expected Discounted Loss Preview

We compute a toy PV of expected loss using a coarse annual grid:

$$PV(EL) \approx \sum_{i=1}^{5} D(0,t_i)\,N(1-R)\,[Q(t_{i-1}) - Q(t_i)]$$

where $D(0,t) = e^{-rt}$ with $r = 3\%$.

**Source note:** This "loss PV as discounted default increments" is consistent with O'Kane's protection-leg form $(1-R)\int_0^T Z(0,t)(-dQ(0,t))$. We keep it as a preview because we are not building full CDS machinery here.

**Given:** $N = 10\text{m}$, $R = 40\% \Rightarrow N(1-R) = 6\text{m}$, hazard $h = 2\% \Rightarrow Q(t) = e^{-0.02t}$

**Compute annual survivals and default increments:**

| $t$ | $Q(t)$ | $\Delta PD = Q(t-1) - Q(t)$ |
|-----|--------|----------------------------|
| 0 | 1 | — |
| 1 | 0.9801987 | 0.0198013 |
| 2 | 0.9607894 | 0.0194092 |
| 3 | 0.9417645 | 0.0190249 |
| 4 | 0.9231163 | 0.0186482 |
| 5 | 0.9048374 | 0.0182789 |

**Discount factors:**

| $t$ | $D(0,t) = e^{-0.03t}$ |
|-----|----------------------|
| 1 | $\approx 0.9704455$ |
| 2 | $\approx 0.9417645$ |
| 3 | $\approx 0.9139312$ |
| 4 | $\approx 0.8869204$ |
| 5 | $\approx 0.8607080$ |

**Compute PV terms (each term $= D \times 6\text{m} \times \Delta PD$):**

| $t$ | PV Term |
|-----|---------|
| 1 | $0.9704455 \times 6\text{m} \times 0.0198013 \approx 115{,}297$ |
| 2 | $0.9417645 \times 6\text{m} \times 0.0194092 \approx 109{,}674$ |
| 3 | $0.9139312 \times 6\text{m} \times 0.0190249 \approx 104{,}325$ |
| 4 | $0.8869204 \times 6\text{m} \times 0.0186482 \approx 99{,}237$ |
| 5 | $0.8607080 \times 6\text{m} \times 0.0182789 \approx 94{,}397$ |

**Sum:**

$$PV(EL) \approx 115{,}297 + 109{,}674 + 104{,}325 + 99{,}237 + 94{,}397 \approx 522{,}930$$

**Output:** $PV(EL) \approx \$0.523\text{m}$ (lower than undiscounted $EL \approx \$0.571\text{m}$)

---

### Example I: Spread ↔ Hazard Toy Approximation

Use O'Kane's credit triangle $S = \lambda(1-R)$.

**Given:** Spread $s = 120$ bp $= 0.0120$ per year (continuous), recovery $R = 40\% \Rightarrow 1-R = 0.6$

**Implied hazard:**

$$\lambda = \frac{s}{1-R} = \frac{0.0120}{0.6} = 0.0200$$

**Output:** $\lambda \approx 2.00\%$ per year

**Assumptions (must be stated):** Continuous premium, constant hazard, and a protection payoff proportional to $(1-R)$.

---

### Example J: Sensitivity to Recovery

Hold $s = 120$ bp $= 0.012$ fixed, vary recovery $R$. Using $\lambda = s/(1-R)$:

| $R$ | $1-R$ | $\lambda = 0.012/(1-R)$ |
|-----|-------|-------------------------|
| 20% | 0.8 | $0.015 = 1.50\%/\text{yr}$ |
| 40% | 0.6 | $0.020 = 2.00\%/\text{yr}$ |
| 60% | 0.4 | $0.030 = 3.00\%/\text{yr}$ |

**Interpretation:** Higher assumed recovery means each default is "less costly," so to justify the same spread, the implied intensity must be higher (more defaults of smaller severity).

---

## Practical Notes

### 5.1 Common Ambiguity Traps

**Real-world vs risk-neutral hazard:**

- Spreads imply risk-neutral hazards (pricing inputs)
- Historical data implies real-world default rates
- Mixing them can cause major confusion in risk reports and valuation explain

**Recovery conventions:**

- Many simplified formulas use a fixed recovery fraction $R$ and a loss fraction $(1-R)$ paid at default (as in O'Kane's protection-leg modeling and credit triangle)
- I'm not sure whether your desk defaults to "recovery of par" vs "market-value recovery" without seeing the specific contract/ISDA settlement assumptions and the exact source section you want to adopt

**Observed spreads include more than expected loss:**

- O'Kane decomposes market spreads into actuarial spread plus default/volatility/liquidity premia
- Therefore, inferring hazard from a bond or CDS spread is model-dependent (liquidity/funding, deliverable option, technical defaults, etc.)

---

### 5.2 Implementation Pitfalls

**Year-fraction consistency:**

- Hazards are per year: always ensure $\Delta t$ uses the same day-count basis as your curve build

**Piecewise hazards and curve "kinks":**

- Piecewise-constant $h_i$ implies $Q(t)$ is continuous but $h(t)$ jumps at knots. That's fine mathematically, but sensitivities may show knot artifacts

**Numerical stability:**

- When $Q(t) \approx 1$, use stable transforms (work with $\ln Q(t)$ or $H(t)$ to avoid catastrophic cancellation)
- When $Q(t)$ is very small, be cautious about underflow in exponentials; track $H(t)$ and exponentiate only when needed

---

### 5.3 Verification Tests (Quick Sanity Suite)

**Survival curve checks:**

- $Q(0) = 1$; $Q(t)$ non-increasing; $0 \leq Q(t) \leq 1$

**Hazard checks:**

- Under standard intensity models, $h(t) \geq 0$. I'm not sure about allowing negative hazards as a modeling convenience unless you specify a source/model that permits it

**Probability mass:**

$$\sum_{i=1}^{n} \Pr(t_{i-1} < \tau \leq t_i) = 1 - Q(t_n)$$

**Unit checks:**

- $h$ in 1/year; $H$ dimensionless; spreads in decimal/year; bp converted correctly (100 bp = 0.01)

---

## Summary & Recall

### 6.1 Ten-Bullet Executive Summary

1. Reduced-form credit models treat default as a random time $\tau \in (0, \infty)$
2. The survival function is $Q(t) = \Pr(\tau > t)$; the default CDF is $F(t) = 1 - Q(t)$
3. Hazard/intensity $h(t)$ is the instantaneous conditional default probability rate
4. Hazard implies the survival ODE $Q'(t) = -h(t)Q(t)$ and solution $Q(t) = \exp(-\int_0^t h)$
5. Cumulative hazard $H(t) = \int_0^t h$ is dimensionless and $Q(t) = e^{-H(t)}$
6. Default density is $f(t) = h(t)Q(t)$
7. Interval PD satisfies $\Pr(t_1 < \tau \leq t_2) = Q(t_1) - Q(t_2)$
8. Practitioners often use piecewise-constant hazards and node survivals to build curves
9. Spreads typically imply risk-neutral hazard rates, not historical ones
10. Under simplifying assumptions, O'Kane's credit triangle connects spread, hazard, and recovery: $S = \lambda(1-R)$

---

### 6.2 Cheat Sheet: Key Identities

**Survival / default:**

$$Q(t) = \Pr(\tau > t), \qquad F(t) = 1 - Q(t)$$

**Interval PD:**

$$\Pr(t_1 < \tau \leq t_2) = Q(t_1) - Q(t_2)$$

**Hazard definition:**

$$h(t) = \lim_{\Delta t \to 0} \frac{\Pr(t < \tau \leq t + \Delta t \mid \tau > t)}{\Delta t}$$

**Hazard ↔ survival:**

$$Q'(t) = -h(t)Q(t), \qquad h(t) = -\frac{d}{dt}\ln Q(t), \qquad Q(t) = e^{-\int_0^t h}$$

**Density:**

$$f(t) = h(t)Q(t)$$

**Cumulative hazard:**

$$H(t) = \int_0^t h(u)\,du, \qquad Q(t) = e^{-H(t)}$$

**Piecewise constant hazard:**

$$Q(t_i) = Q(t_{i-1})e^{-h_i \Delta t_i}, \qquad h_i = -\frac{1}{\Delta t_i}\ln\left(\frac{Q(t_i)}{Q(t_{i-1})}\right)$$

**Credit triangle (assumption-heavy but source-backed in O'Kane):**

$$S = \lambda(1-R), \qquad \lambda = \frac{S}{1-R}$$

---

### 6.3 Flashcards (30)

| # | Question | Answer |
|---|----------|--------|
| 1 | What does $\tau$ represent? | The (one-time) default time random variable |
| 2 | Define $Q(t)$ | $Q(t) = \Pr(\tau > t)$, the survival probability |
| 3 | Define $F(t)$ | $F(t) = \Pr(\tau \leq t) = 1 - Q(t)$ |
| 4 | Define hazard rate $h(t)$ | Instantaneous conditional default probability rate given survival |
| 5 | Units of $h(t)$? | 1/year |
| 6 | Relationship between $h(t)$ and $Q(t)$? | $Q'(t) = -h(t)Q(t)$ |
| 7 | Express $Q(t)$ in terms of $h(t)$ | $Q(t) = \exp(-\int_0^t h)$ |
| 8 | Define cumulative hazard $H(t)$ | $H(t) = \int_0^t h(u)\,du$ |
| 9 | Express $Q(t)$ using $H(t)$ | $Q(t) = e^{-H(t)}$ |
| 10 | Define default density $f(t)$ | $f(t) = \frac{d}{dt}\Pr(\tau \leq t)$ (when it exists) |
| 11 | Key identity for density? | $f(t) = h(t)Q(t)$ |
| 12 | Interval PD formula using survival? | $\Pr(t_1 < \tau \leq t_2) = Q(t_1) - Q(t_2)$ |
| 13 | If $h$ is constant, what is $Q(t)$? | $Q(t) = e^{-ht}$ |
| 14 | If $Q(5) = 0.9$, what is average hazard over $[0,5]$? | $-\ln(0.9)/5$ |
| 15 | What does "default is a surprise" mean in hazard models? | You can't predict default with certainty over an infinitesimal horizon; only assign probability $\lambda(t)\,dt$ |
| 16 | What measure do spread-implied hazards typically live in? | Risk-neutral ($\mathbb{Q}$) |
| 17 | What measure do historical default rates live in? | Real-world ($\mathbb{P}$) |
| 18 | Why can $\mathbb{Q}$-PDs exceed $\mathbb{P}$-PDs? | Risk premia (and other spread components) |
| 19 | What is the "credit triangle"? | $S = \lambda(1-R)$ under simplifying assumptions |
| 20 | Convert 120 bp to decimal per year | $0.012$ |
| 21 | If $S = 0.012$ and $R = 0.4$, implied $\lambda$ (triangle)? | $0.012/0.6 = 0.02$ |
| 22 | Piecewise hazard update formula | $Q(t_i) = Q(t_{i-1})e^{-h_i \Delta t_i}$ |
| 23 | Piecewise hazard from survivals | $h_i = -(1/\Delta t_i)\ln(Q_i/Q_{i-1})$ |
| 24 | What is $H(t)$ used for numerically? | Stability; work with $H = -\ln Q$ when $Q$ near 1 or tiny |
| 25 | How to check a survival curve quickly? | $Q(0) = 1$, monotone non-increasing, stays in $[0,1]$ |
| 26 | What does $1-R$ represent? | Loss fraction (LGD) in simple recovery modeling |
| 27 | Expected loss by $T$ (no discount) in simple model? | $N(1-R)(1-Q(T))$ |
| 28 | What object weights default-leg PVs in reduced-form pricing? | $-dQ$ or $\lambda Q\,dt$ |
| 29 | Why do we need density $f(t)$ for some payoffs? | Default time determines cashflow timing (e.g., recovery at default) |
| 30 | What's the main limitation of triangle intuition? | Real spreads embed non-default components and CDS cashflows are discrete with accruals, etc. |

---

## Mini Problem Set (16 Questions)

*Provide brief solution sketches for questions 1–8 only.*

---

**1.** Given constant hazard $h = 1.5\%$, compute $Q(2)$ and $\Pr(\tau \leq 2)$.

**Sketch:** $Q(2) = e^{-0.015 \cdot 2}$. Then $\Pr(\tau \leq 2) = 1 - Q(2)$.

---

**2.** If $Q(3) = 0.92$, compute the average constant hazard over $[0,3]$.

**Sketch:** $\bar{h} = -(1/3)\ln(0.92)$.

---

**3.** With $Q(1) = 0.97$, $Q(2) = 0.94$, compute piecewise hazards $h_1, h_2$ on yearly intervals.

**Sketch:** $h_1 = -\ln(0.97)$, $h_2 = -\ln(0.94/0.97)$.

---

**4.** For constant $h = 2\%$, compute $\Pr(1 < \tau \leq 3)$.

**Sketch:** $Q(1) = e^{-0.02}$, $Q(3) = e^{-0.06}$. Interval PD $= Q(1) - Q(3)$.

---

**5.** Show that $f(t) = h(t)Q(t)$ implies $\int_0^T f(t)\,dt = 1 - Q(T)$.

**Sketch:** Since $Q'(t) = -h(t)Q(t) = -f(t)$, integrate: $Q(T) - Q(0) = -\int_0^T f$. Use $Q(0) = 1$.

---

**6.** Using credit triangle $S = \lambda(1-R)$, compute $\lambda$ for $S = 80$ bp, $R = 35\%$.

**Sketch:** Convert $S = 0.008$. Then $\lambda = 0.008/(1-0.35) = 0.008/0.65$.

---

**7.** For notional $N = 5\text{m}$, $R = 40\%$, $Q(5) = 0.88$, compute undiscounted $EL(5)$.

**Sketch:** $EL = N(1-R)(1-Q(5)) = 5\text{m} \cdot 0.6 \cdot 0.12$.

---

**8.** Build $Q(2)$ given $h = 1\%$ on $[0,1)$ and $h = 4\%$ on $[1,2)$.

**Sketch:** $Q(1) = e^{-0.01}$. Then $Q(2) = Q(1)e^{-0.04}$.

---

**9.** Explain (conceptually) why a short-maturity credit can trade at non-zero spread even when default over the next day is unlikely.

---

**10.** In a piecewise-constant hazard curve, what happens to $h(t)$ and $Q(t)$ at knot points?

---

**11.** Give two reasons a bond spread may not equal a CDS spread (basis).

---

**12.** Suppose you observe two different survival curves for the same name from bonds vs CDS. List three diagnostics you would run.

---

**13.** Derive the relationship $h(t) = -\frac{d}{dt}\ln Q(t)$ from $Q(t) = e^{-H(t)}$.

---

**14.** If $Q(10) = 0.6$, what is the implied 10y average hazard? How would you interpret it?

---

**15.** Describe how risk-neutral hazard rates are used in CVA-style expected exposure weighting (conceptually).

---

**16.** Explain why recovery assumptions materially change implied hazards from a given spread.

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

| Concept | Source |
|---------|--------|
| Hazard rate definition | O'Kane Ch 3, 6–7; QRM hazard rate definition; Hull OFOD |
| Survival ODE and exponential solution | Hull OFOD; O'Kane reduced-form framework |
| Risk-neutral vs historical distinction | Hull OFOD; O'Kane explicit discussion |
| Credit triangle $S = \lambda(1-R)$ | O'Kane (under stated simplifying assumptions) |
| Default density $f(t) = h(t)Q(t)$ | QRM; O'Kane pricing derivations |

### (B) Reasoned Inference — Note Derivation Logic

| Concept | Derivation |
|---------|------------|
| Piecewise hazard formulas | Direct integration of constant hazard over intervals |
| Interval default probability | Probability algebra from survival definition |
| Expected loss formulas | Standard PV calculation using survival-weighted losses |

### (C) Speculation — Flag Uncertainties

| Item | Uncertainty |
|------|-------------|
| Recovery convention (par vs market-value) | Not specified without contract/ISDA details |
| Negative hazard allowance | Not verified in sources; standard models assume $h(t) \geq 0$ |
