# Chapter 36: Survival Probabilities and Hazard Rates

## Learning Objectives
- Define survival probability $Q(t)$, hazard rate $h(t)$, cumulative hazard $H(t)$, and default density $f(t)$ with correct units.
- Convert $h \leftrightarrow Q$ and compute interval default probabilities.
- Build a piecewise-constant hazard curve from survival points and run basic arbitrage sanity checks.
- Use the credit triangle $S \approx h(1-R)$ as a back-of-envelope approximation (and know its assumptions).
- Define and interpret credit risk bumps (hazard bumps and CS01) with explicit bump object, bump size, units, and sign.

## Introduction

“How long until default?” is a question about a **random time** $\tau$. In reduced-form (intensity) models we summarize the distribution of $\tau$ with:

- a **survival curve** $Q(t)=\Pr(\tau\gt t)$, and
- a **hazard rate** $h(t)$, which controls how fast $Q(t)$ decays.

On a desk, these objects sit in the middle of a workflow you will use repeatedly:

**quote** (bond/CDS spread, upfront) → **default-timing object** ($Q$ or $h$) → **PV equation** (expected discounted cashflows) → **risk** (CS01 / hazard bumps).

Prerequisites: [Chapter 35: Default, recovery, and credit events](chapters/chapter_35_default_recovery_credit_events.md); [Chapter 02: Time value and discount factors](chapters/chapter_02_time_value_discount_factors_replication.md).  
Follow-on: [Chapter 37: Cash credit, risky bonds, spreads, CS01](chapters/chapter_37_cash_credit_risky_bonds_spreads_cs01.md); [Chapter 38: CDS contract mechanics](chapters/chapter_38_cds_contract_mechanics.md); [Chapter 42: Bootstrapping a CDS survival curve](chapters/chapter_42_bootstrapping_cds_survival_curve.md); [Chapter 43: CDS risks and hedging](chapters/chapter_43_cds_risks_hedging.md).

**Conventions in this chapter**
- Time $t$ is a year-fraction. Keep units consistent: if $h$ is “per year”, then $t$ is in years.
- We write the hazard rate as $h(t)$. Many texts use $\lambda(t)$; it is the same object.

---

## 36.1 Reduced-Form vs. Structural Approaches

Structural and reduced-form models answer different questions:

- **Structural models** tie default to an underlying state variable (e.g., firm asset value crossing a boundary).
- **Reduced-form models** specify default timing directly via $Q(t)$ or $h(t)$, and are built to fit traded credit spreads.

For most CDS-style pricing and hedging, you will spend more time with reduced-form objects because they plug directly into valuation formulas and curve construction.

### 36.1.1 Structural Models (Conceptual)

Structural models are economically appealing because they connect default to leverage and asset volatility. In their simplest forms, however, they can be awkward for desk calibration: very short-dated spreads can come out too close to zero, and the underlying state (firm asset value / default boundary) is not directly observable at high frequency.

### 36.1.2 Reduced-Form Models (Intensity)

Reduced-form models treat default as a jump at a random time $\tau$. Over a small interval $dt$, conditional on survival to time $t$,

$$\Pr(t \lt  \tau \le t+dt \mid \tau\gt t) \approx h(t)\,dt.$$

This “surprise default” property is exactly what makes intensity models flexible enough to fit positive short-maturity credit spreads.

> **Desk Reality:** Many pricing systems ingest a spread curve and output a survival curve $Q(t)$ (risk-neutral), then reuse $Q$ for pricing and hedging.  
> **Common break:** Using a spread-implied (risk-neutral) curve as if it were a historical default-frequency estimate.  
> **What to check:** Ask “pricing or scenario?” and verify which measure the curve is in (Section 36.7).

---

## 36.2 The Survival Function $Q(t)$

### 36.2.1 Definition and Interpretation

The survival function is the probability that a credit has not defaulted by time $t$:

$$\boxed{Q(t)=\Pr(\tau\gt t)}.$$

The corresponding cumulative default probability is:

$$F(t)=\Pr(\tau \le t)=1-Q(t).$$

**Basic sanity checks:**
1. $Q(0)=1$.
2. $Q(t)$ is non-increasing in $t$.
3. $0 \le Q(t) \le 1$.

### 36.2.2 Survival as a “Credit Discount Factor”

Let $P(0,T)$ be the risk-free discount factor (PV of 1 paid at $T$). In a baseline intensity model with **deterministic discounting and hazard**, a payoff of 1 received at $T$ only if the name survives to $T$ (zero recovery) has:

$$\text{PV} = P(0,T)\,Q(0,T).$$

This is the cleanest place to see what $Q$ does: it “shrinks” risky cashflows for the chance that the issuer is not alive to pay them. Later credit pricing chapters add coupons, recovery assumptions, discretization, and curve bootstrapping.

**Check (small-hazard approximation):** If cumulative hazard $H(T)=\int_0^T h(s)\,ds$ is small, then $Q(T)=e^{-H(T)}\approx 1-H(T)$. So, to first order, “credit discounting” reduces PV by about $P(0,T)\times H(T)$ for a \$1 survival-contingent payoff. This is a useful magnitude check for very high-quality names where $H(T)\ll 1$.

> **On-Ramp:** If you already think of $P(0,T)$ as “today’s value of \$1 at $T$,” think of $Q(0,T)$ as “today’s probability the name is still alive at $T$.”

### 36.2.3 Interval Default Probability

The probability of default occurring in a specific interval is:

$$\boxed{\Pr(t_1 \lt  \tau \leq t_2) = Q(t_1) - Q(t_2)}$$

This identity is the workhorse for turning a survival curve into period-by-period default probabilities.

**Check (telescoping on a grid):** On a grid $0=t_0\lt t_1\lt \dots\lt t_n$, the interval default probabilities add up:

$$
\sum_{i=1}^n \Pr(t_{i-1}\lt \tau\le t_i)=\sum_{i=1}^n \bigl(Q(t_{i-1})-Q(t_i)\bigr)=1-Q(t_n).
$$

If your interval probabilities do not sum to the cumulative default probability $1-Q(t_n)$, something is inconsistent.

---

## 36.3 The Hazard Rate (Default Intensity)

### 36.3.1 Definition: Conditional Default Probability per Unit Time

The hazard rate, also called the *default intensity*, is defined by the conditional default probability over a short interval:

$$\boxed{h(t) = \lim_{\Delta t \to 0} \frac{1}{\Delta t} \Pr[\tau \leq t + \Delta t \mid \tau \gt  t]}$$

Equivalently, for small $\Delta t$,

$$\Pr(t\lt \tau\le t+\Delta t\mid \tau\gt t)\approx h(t)\,\Delta t.$$

**Units:** The hazard rate has units of $1/\text{year}$ (or inverse time). A hazard rate of $h = 0.02$ means a 2% probability of default per year, given survival—not a 2% probability of default in total.

### 36.3.2 Economic Interpretation

If $h(t) = 0.02$, what does this mean practically?

For a small time interval $\Delta t$ (say, one day = 1/365 year):

$$\Pr(\text{default in next day} \mid \text{alive today}) \approx 0.02 \times \frac{1}{365} \approx 0.0055\%$$

The hazard rate is the *per-unit-time* intensity. It tells you how "risky" each instant is, conditional on having survived to that instant.

> **Analogy: Radioactive Decay**
>
> A portfolio of bonds is like a block of **Radioactive Uranium**.
>
> *   **Decay (Default)**: Individual atoms (companies) decay (default) at random times.
> *   **Hazard Rate ($h$)**: This is the **decay rate** (related to half-life). A high $h$ means the block is highly radioactive and atoms are popping off quickly.
> *   **Survival Probability ($Q$)**: The weight of the uranium block remaining at time $t$.
> *   **Intensity**: If you put a Geiger counter next to the block, the "clicks" are the defaults. The rate of clicking is $h$.
>
> **Half-Life Connection:** For constant hazard $h$, the "half-life" (time for 50% to default) is $t_{1/2} = \ln(2)/h \approx 0.693/h$. A 2% hazard rate implies a half-life of about 35 years.

One Monte Carlo intuition: discretize time into small steps $dt$. Conditional on survival to $t$, default occurs in $[t,t+dt]$ with probability $h(t)\,dt$. As $dt\to 0$, the per-step default probability goes to zero, but the **per-year** intensity $h(t)$ stays finite.

This "per unit time" interpretation—the hazard rate remains finite even as the time step shrinks—is what makes reduced-form models generate positive short-dated spreads.

### 36.3.3 Order-of-Magnitude Hazard Levels (Credit Triangle Intuition)

To build intuition for magnitudes, practitioners often use the **credit triangle** approximation:

$$S \approx h(1-R) \quad \Rightarrow \quad h \approx \frac{S}{1-R}$$

For an illustrative recovery assumption of $R = 40\%$ (so $1-R = 60\%$), the mapping from CDS spread to (risk-neutral) hazard is:

| 5Y CDS Spread (bp) | Implied Hazard $h$ (%/year) |
|-------------------|-----------------------------------|
| 50 | $\approx 0.83$ |
| 150 | $\approx 2.50$ |
| 500 | $\approx 8.33$ |
| 1000 | $\approx 16.67$ |

> **Desk Reality: Quick Sanity Checks**
>
> The triangle is a back-of-envelope tool, not a rating model. Still, it is useful for spotting obvious issues:
> - Wrong ticker / tier / currency / documentation.
> - Recovery assumption mismatch (40% vs name-specific or distressed recoveries).
> - A spread that moved but your hazard curve did not (or vice versa) due to stale inputs.
>
> Also remember: the hazard inferred this way is a *risk-neutral* intensity used for pricing, and it can differ materially from historical default-frequency intuition.

### 36.3.4 The Fundamental Relationship: Hazard Rate and Survival ODE

Starting from the hazard rate definition, we can derive a differential equation for the survival function. This derivation is fundamental to the entire framework.

For small $\Delta t$:

$$\Pr(t \lt  \tau \leq t + \Delta t \mid \tau \gt  t) \approx h(t) \Delta t$$

Multiply both sides by $\Pr(\tau \gt  t) = Q(t)$:

$$\Pr(t \lt  \tau \leq t + \Delta t) \approx Q(t) h(t) \Delta t$$

Now, the survival at $t + \Delta t$ equals survival at $t$ minus the probability of default in between:

$$Q(t + \Delta t) = Q(t) - Q(t) h(t) \Delta t$$

Rearranging:

$$\frac{Q(t + \Delta t) - Q(t)}{\Delta t} \approx -h(t) Q(t)$$

Taking the limit $\Delta t \to 0$:

$$\boxed{\frac{dQ(t)}{dt} = -h(t) Q(t)}$$

This is the **survival ODE**.

**Unit check:** The left side $dQ/dt$ has units $1/\text{year}$. The right side $(-h) \times Q$ has $(1/\text{year}) \times (1) = 1/\text{year}$. ✓

### 36.3.5 Solving the ODE: Survival in Terms of Cumulative Hazard

This is a separable first-order ODE. Separate variables and integrate:

$$\frac{dQ(t)}{Q(t)} = -h(t) dt$$

$$\int_0^t \frac{dQ(s)}{Q(s)} = -\int_0^t h(s) ds$$

$$\ln Q(t) - \ln Q(0) = -\int_0^t h(s) ds$$

Using the boundary condition $Q(0) = 1$:

$$\boxed{Q(t) = \exp\left(-\int_0^t h(s) ds\right) = e^{-H(t)}}$$

where the **cumulative hazard** (also called the integrated hazard) is:

$$\boxed{H(t) = \int_0^t h(s) ds}$$

Many sources denote the hazard rate by $\lambda(t)$. With $\lambda(t)\equiv h(t)$, the deterministic-hazard formula is often written as $\Pr(\tau\gt T)=\exp(-\int_0^T \lambda(t)\,dt)$.

Equivalently, we can recover the hazard rate from the survival function:

$$\boxed{h(t) = -\frac{d}{dt} \ln Q(t) = -\frac{Q'(t)}{Q(t)}}$$

If $h(t)$ is stochastic (a Cox process), survival involves an expectation:

$$Q(t) = \mathbb{E}\left[\exp\left(-\int_0^t h(s) ds\right)\right]$$

If $h(t)$ is deterministic, the expectation disappears and $Q(t)=\exp(-\int_0^t h(s)\,ds)$.

### 36.3.6 Sanity Checks

Before trusting any survival formula, verify these properties:

- If $h(t) \geq 0$, then $H(t)$ is non-decreasing and $Q(t)$ is non-increasing ✓
- If $h(t) = 0$ for all $t$, then $Q(t) = 1$ (no default risk) ✓
- If $h(t) = h$ (constant), then $Q(t) = e^{-h t}$ (exponential survival) ✓

---

## 36.4 The Default Density

When the distribution function $F(t) = \Pr(\tau \leq t)$ is absolutely continuous, the **default density** is:

$$f(t) = \frac{dF(t)}{dt} = -\frac{dQ(t)}{dt}$$

From the survival ODE $Q'(t) = -h(t) Q(t)$, we get the key identity:

$$\boxed{f(t) = h(t) Q(t)}$$

Equivalently, the default probability in a short interval is:
$$\Pr(T\lt \tau \le T+dT)=f(T)\,dT=h(T)\exp\left(-\int_0^T h(t)\,dt\right)dT.$$

**Interpretation:** The probability of defaulting "at time $t$" (more precisely, in $[t, t+dt]$) equals the hazard rate at $t$ times the probability of surviving to $t$. You can only default at $t$ if you're still alive at $t$.

**Unit check:** $f(t)$ is a density (units $1/\text{year}$). The RHS is $h(t) \times Q(t) = (1/\text{year}) \times (1) = 1/\text{year}$. ✓

This identity is essential for pricing protection legs in CDS contracts, where the protection payment occurs at the moment of default.

### 36.4.1 Verification: Density Integrates to Cumulative Default Probability

As a sanity check, integrate the density:

$$\int_0^T f(t) dt = \int_0^T h(t) Q(t) dt = \int_0^T -Q'(t) dt = Q(0) - Q(T) = 1 - Q(T)$$

This confirms $\int_0^T f(t) dt = F(T) = 1 - Q(T)$. ✓

The density function is also crucial for computing expected default timing. The expected default time is $\mathbb{E}[\tau] = \int_0^\infty t f(t) dt$, which is well-defined when the integral converges.

---

## 36.5 Constant Hazard Rate: The Exponential Case

When the hazard rate is constant, $h(t) = h$, the survival function simplifies to:

$$Q(t) = e^{-h t}$$

This is the **exponential survival** model—the continuous-time analog of geometric default probability. It has several useful properties that make it valuable for building intuition and for quick approximations.

### 36.5.1 Mean Default Time

O'Kane derives the expected default time for constant hazard. Using the density $f(t) = h e^{-h t}$:

$$\mathbb{E}[\tau] = \int_0^\infty t \cdot f(t) dt = \int_0^\infty t \cdot h e^{-h t} dt = \frac{1}{h}$$

The variance is $\text{Var}(\tau) = 1/h^2$, so the standard deviation equals the mean.

**Example:** A credit with hazard rate $h = 0.02$ (2% per year) has expected default time:

$$\mathbb{E}[\tau] = \frac{1}{0.02} = 50 \text{ years}$$

Perspective: when $h$ is small, default times are extremely dispersed (mean $1/h$, standard deviation $1/h$). In Monte Carlo, that means you may need many paths to estimate rare defaults with acceptable standard error.

### 36.5.2 The Memoryless Property

The exponential distribution is the only continuous distribution with the memoryless property: given that the credit has survived to time $s$, the remaining survival time has the same distribution as the original survival time.

$$\Pr(\tau \gt  s + t \mid \tau \gt  s) = \Pr(\tau \gt  t)$$

This can be verified directly:
$$\Pr(\tau \gt  s + t \mid \tau \gt  s) = \frac{Q(s+t)}{Q(s)} = \frac{e^{-h(s+t)}}{e^{-h s}} = e^{-h t} = Q(t)$$

The memoryless property is unrealistic for actual credits—past performance does provide information about future default risk. A company that has been struggling for years is more likely to default than one with a strong track record. However, this simplification makes the constant-hazard model analytically tractable and provides a useful baseline.

### 36.5.3 When Constant Hazard Is Appropriate

Constant hazard is most defensible when the spread curve is roughly flat. Even when it is not, the constant-hazard model can be a useful approximation for intuition and quick sensitivity checks.

**Decision Tree: When to Use Constant Hazard**

| Situation | Use Constant Hazard? |
|-----------|---------------------|
| CDS curve is flat (or nearly flat) | Yes |
| Quick back-of-envelope estimate needed | Yes |
| Understanding parameter sensitivities | Yes |
| Precise pricing for risk management | No — use piecewise |
| Valuing forward-starting products | No — term structure matters |
| Significant curve slope or inversion | No — piecewise required |

The constant hazard case is valuable for:
- Building intuition about credit derivatives pricing
- Quick back-of-envelope calculations
- Understanding model parameter sensitivities
- Analytical solutions that serve as benchmarks

For actual pricing with market calibration, practitioners use piecewise-constant or more general hazard structures.

---

## 36.6 Piecewise-Constant Hazard Rates

In practice, CDS curves are not flat, and we need a richer model. The **piecewise-constant hazard** model is the workhorse of credit curve construction, balancing flexibility with tractability.

### 36.6.1 Setup

Define a time grid $0 = t_0 \lt  t_1 \lt  t_2 \lt  \cdots \lt  t_n$ and assume:

$$h(t) = h_i \quad \text{for } t \in [t_{i-1}, t_i)$$

The hazard rate is constant within each interval but can jump between intervals. This corresponds to a survival curve that is piecewise exponential—$Q(t)$ decays exponentially within each interval, with a possibly different decay rate in each.

> **Visual: The Staircase & The Slope**
>
> *   **Hazard Rate ($h$)**: Imagine a **Staircase**. It stays flat at 1% for Year 1, then jumps up to 2% for Year 2, then 3% for Year 3.
> *   **Survival Curve ($Q$)**: This creates a **kinked slope**. The log-survival graph is a straight line sloping down. At Year 1, the slope gets steeper (more decay). At Year 2, it gets steeper again.
> *   **Bootstrapping**: We build this staircase one step at a time. We use the 1Y CDS spread to build the first step. Then we use the 2Y CDS spread (and the first step) to build the second step.

### 36.6.2 Forward Survival Formulas

The cumulative hazard over interval $i$ is simply the hazard rate times the interval length:

$$H(t_i) - H(t_{i-1}) = h_i \cdot (t_i - t_{i-1}) = h_i \cdot \Delta t_i$$

The survival probability updates multiplicatively:

$$\boxed{Q(t_i) = Q(t_{i-1}) \cdot e^{-h_i \Delta t_i}}$$

Alternatively, cumulative hazard updates additively:

$$\boxed{H(t_i) = H(t_{i-1}) + h_i \Delta t_i}$$

This recursive structure makes bootstrapping efficient: given market prices that imply $Q(t_i)$ at each maturity, we can solve for $h_i$ interval by interval.

### 36.6.3 Inverting: Hazard from Survival

Given survival probabilities at grid points, we can recover the piecewise hazards by inverting the forward formula:

$$\boxed{h_i = -\frac{1}{\Delta t_i} \ln\left(\frac{Q(t_i)}{Q(t_{i-1})}\right)}$$

**Check (toy numbers):** Suppose an annual grid with $Q(0)=1$, $Q(1)=0.98$, and $Q(2)=0.94$.
- $h_1=-\ln(0.98/1)/1\approx 2.02\%$/year.
- $h_2=-\ln(0.94/0.98)/1\approx 4.17\%$/year.
Plugging back, $Q(2)=Q(1)e^{-h_2}\approx 0.98\times e^{-0.0417}\approx 0.94$ as required.

This formula is central to survival curve bootstrapping from CDS spreads (Chapter 42). Given a term structure of CDS quotes, we price the shortest maturity CDS to find $Q(t_1)$, extract $h_1$, then use this to price the next maturity and find $Q(t_2)$, and so on.

### 36.6.4 Interpolation Schemes

O'Kane discusses interpolation between grid points in detail. The choice of interpolation scheme affects both pricing accuracy and curve smoothness. Three common approaches:

| Scheme | Interpolation Of | Hazard Behavior | Properties |
|--------|-----------------|-----------------|------------|
| Linear in $Q$ | Survival probability | Piecewise non-constant | Can produce negative hazards |
| Linear in $\ln Q$ | Log survival | **Piecewise constant** hazard | ensures $Q\in(0,1]$ and $h\ge 0$ if endpoints are consistent |
| Linear in $H$ | Cumulative hazard | Piecewise linear hazard | Smooth, but requires care |

Linear interpolation of $\ln Q(t)$ between knot points is equivalent to assuming a piecewise-constant forward default rate (hazard) between those knot points. Concretely, for $t_{n-1}\lt t^*\lt t_n$:

- Survival probabilities remain between 0 and 1
- Hazard rates remain positive (no-arbitrage)
- The curve is continuous in $Q(t)$ but jumps in $h(t)$ at knots

$$h_{n-1}=\frac{1}{t_n-t_{n-1}}\ln\left(\frac{Q(t_{n-1})}{Q(t_n)}\right), \qquad Q(t^*) = Q(t_{n-1}) \cdot e^{-(t^* - t_{n-1}) \cdot h_{n-1}}.$$

> **Implementation Note: What Happens When Bootstrapping Fails**
>
> If your bootstrapping algorithm produces a negative hazard rate $h_i \lt  0$, you have one of two problems:
> 1. **Data error**: Check your input spreads. Is the curve inverted beyond arbitrage limits?
> 2. **Arbitrage opportunity**: If the data is correct, the market is mispriced. See Section 36.10 on arbitrage bounds.
>
> Never force a negative hazard to zero—that masks the real problem. Either fix the inputs or investigate the arbitrage.

---

## 36.7 Risk-Neutral vs. Real-World Hazard Rates

A critical distinction pervades credit modeling: the difference between **risk-neutral** ($\mathbb{Q}$) and **real-world** ($\mathbb{P}$) default probabilities. Confusing these two measures is one of the most common errors in credit risk management.

### 36.7.1 Spread-Implied Hazards Are Risk-Neutral

Default probabilities or hazard rates implied from credit spreads are **risk-neutral estimates**. When we calibrate survival curves to CDS spreads, the resulting hazard rates are the ones that correctly price those instruments under risk-neutral valuation (expected cashflows discounted at a risk-free rate).

Risk-neutral hazard rates are derived from market prices, not from historical default frequencies. They incorporate:
- The market's best estimate of actual default probability
- A risk premium for bearing credit exposure
- A liquidity premium for holding credit-sensitive instruments

### 36.7.2 Historical Defaults Are Real-World

Default statistics from rating agencies (e.g., Moody's or S&P default studies) give real-world probabilities—the frequency of default observed historically in each rating category.

Hull presents striking evidence of the gap by comparing seven-year historical hazard rates to hazard rates implied by bond yields:

| Rating | Historical Hazard Rate | Hazard Rate from Bonds | Ratio | Difference |
|--------|------------------------|------------------------|-------|------------|
| Aaa | 0.04% | 0.67% | 16.8× | 0.63% |
| Aa | 0.06% | 0.78% | 13.0× | 0.72% |
| A | 0.13% | 1.28% | 9.8× | 1.15% |
| Baa | 0.47% | 2.38% | 5.1× | 1.91% |

In this sample, the risk-neutral-implied hazard can be many times larger than the historical hazard (e.g., ~5× for Baa and ~17× for Aaa).

The economic story is that spreads include compensation beyond actuarial expected loss, such as:

- **Default risk premium** — investors are risk-averse and require compensation beyond expected losses
- **Systematic risk** — defaults tend to cluster in recessions when investors can least afford losses
- **Liquidity premium** — credit instruments are less liquid than risk-free alternatives
- **Uncertainty premium** — default probabilities are estimated and can jump in stress

### 36.7.3 Why the Ratio Is Largest for High-Quality Credits

Notice that the ratio is 16.8× for Aaa but only 5.1× for Baa. This pattern is economically meaningful:

For high-quality credits (Aaa, Aa), the *actual* default probability is tiny. But the *spread* isn't zero—investors still demand compensation for:
- Liquidity (Treasuries are more liquid than even AAA corporates)
- Jump-to-default risk (small probability of catastrophic loss)
- Model uncertainty (what if our AAA estimate is wrong?)

The spread on a AAA bond is almost entirely risk premium and liquidity premium, with minimal expected loss component. Hence the huge ratio.

For lower-quality credits (Baa, BB), actual expected losses become meaningful. The spread reflects genuine loss expectations plus a smaller proportional risk premium. Hence the smaller ratio.

### 36.7.4 Practical Implications

This distinction has direct consequences for how you should use each measure:

**For pricing derivatives:** Use risk-neutral probabilities (from spreads). This ensures your prices are consistent with market-observable instruments and support no-arbitrage pricing.

**For risk management and capital:** Use real-world probabilities (from historical data or adjusted from spreads). VaR, economic capital, and stress testing require estimates of what actually happens, not risk-neutral prices.

Rule of thumb:
- Use **spread-implied** hazards/probabilities for **valuation and hedging** (be consistent with traded prices).
- Use **historical / real-world** estimates for **scenario analysis and loss forecasting**.

Mixing them up leads to either systematic mispricing (using real-world for pricing) or understated risk measures (using risk-neutral for risk management).

> **Desk Reality: Your CDS Desk Uses Risk-Neutral; Your Credit VaR Uses Real-World**
>
> In a typical bank:
> - **CDS trading desk**: Marks positions using spread-implied hazards (risk-neutral). Their P&L is based on market moves.
> - **Credit VaR team**: Uses historical default data (real-world) adjusted for economic conditions. They measure "what could we lose?"
> - **CVA desk**: Uses risk-neutral for pricing adjustments, but may blend with real-world for wrong-way risk.
>
> A common error: applying CDS-implied PDs to a loan portfolio's economic capital calculation. This can overstate expected losses by large multiples (e.g., 5–17× in Hull’s sample table).
>
> The opposite error: using historical PDs to price a CDS. Your hedge would be systematically wrong, and you'd lose money on average.

### 36.7.5 Converting Between Measures

While there's no universal formula to convert between measures (the risk premium varies by name, rating, and market conditions), Hull provides a rough approach for understanding the wedge:

$$\text{Credit spread} \approx (1-R) \times h_{\mathbb{Q}} \approx (1-R) \times h_{\mathbb{P}} + \text{risk premium} + \text{liquidity premium}$$

The "risk premium" in credit is analogous to the equity risk premium—it compensates investors for bearing systematic risk. In credit, this is primarily the risk that defaults cluster in bad economic times (when the marginal utility of wealth is high).

---

## 36.8 The Credit Triangle

Under strong simplifying assumptions, a par CDS running spread is approximately proportional to hazard $\times$ loss-given-default. This back-of-envelope mapping is often called the **credit triangle**.

### 36.8.1 Derivation (Continuous-Premium Toy CDS)

Assume a stylized CDS-like contract with:
- Continuous premium payments at rate $S$ until default or maturity $T$
- Protection payment of $(1-R)$ at default time $\tau$ if $\tau \le T$
- Constant hazard rate $h$
- Zero interest rates (so $P(0,t)=1$)

Premium-leg PV:
$$PV_{\text{prem}}=\int_0^T S\,Q(t)\,dt.$$

Protection-leg PV (using $f(t)=hQ(t)$):
$$PV_{\text{prot}}=\int_0^T (1-R)\,f(t)\,dt=\int_0^T (1-R)\,h\,Q(t)\,dt.$$

At par, $PV_{\text{prem}}=PV_{\text{prot}}$, so:

$$\boxed{S = h(1-R)}$$

Equivalently:

$$\boxed{h = \frac{S}{1-R}}$$

Interpretation: holding $R$ fixed, wider spreads map to higher implied hazard; holding $S$ fixed, higher assumed recovery maps to higher implied hazard.

> **Pitfall — Credit triangle treated as an identity:** Using $S=h(1-R)$ as if it were exact for real CDS.
> **Why it matters:** You will mis-map spreads to hazard, mis-size CS01, and get the wrong “sanity check” in steep/discounted/discrete settings.
> **Quick check:** Price one CDS maturity with the full discretized premium/protection legs; compare the implied $h$ to $S/(1-R)$ and note the gap.

### 36.8.2 Back-of-Envelope Rules

The credit triangle enables rapid mental math. Memorize these patterns:

**Rule of Thumb: 40% Recovery**

With standard 40% recovery ($1-R = 0.60$):

| Spread | Implied Hazard | Mental Math |
|--------|----------------|-------------|
| 60 bp | 1.0%/year | Spread × 1.67 |
| 120 bp | 2.0%/year | Spread × 1.67 |
| 300 bp | 5.0%/year | Spread × 1.67 |
| 600 bp | 10%/year | Spread × 1.67 |

The multiplier is $1/(1-R) = 1/0.6 \approx 1.67$.

**Varying Recovery:**

| Recovery | LGD | Multiplier | 300bp Spread → Hazard |
|----------|-----|------------|----------------------|
| 20% | 0.80 | 1.25× | 3.75%/year |
| 40% | 0.60 | 1.67× | 5.0%/year |
| 60% | 0.40 | 2.50× | 7.5%/year |

Higher assumed recovery means larger implied hazard rate (more defaults needed to justify the same spread).

### 36.8.3 Worked Examples Across Credit Quality

**Example A: Investment Grade (50bp spread)**

Given: CDS spread $S = 50$ bp = 0.0050, Recovery $R = 40\%$

$$h = \frac{0.0050}{0.60} = 0.0083 = 0.83\%/\text{year}$$

5-year survival (assuming constant hazard):
$$Q(5) = e^{-0.0083 \times 5} = e^{-0.0417} = 0.959 = 95.9\%$$

5-year default probability: $1 - 0.959 = 4.1\%$

**Example B: High Yield (500bp spread)**

Given: CDS spread $S = 500$ bp = 0.05, Recovery $R = 40\%$

$$h = \frac{0.05}{0.60} = 0.0833 = 8.33\%/\text{year}$$

5-year survival:
$$Q(5) = e^{-0.0833 \times 5} = e^{-0.417} = 0.659 = 65.9\%$$

5-year default probability: $1 - 0.659 = 34.1\%$

**Example C: Distressed Credit (2000bp spread)**

Given: CDS spread $S = 2000$ bp = 0.20, Recovery $R = 20\%$ (lower for distressed)

$$h = \frac{0.20}{0.80} = 0.25 = 25\%/\text{year}$$

1-year survival:
$$Q(1) = e^{-0.25} = 0.779 = 77.9\%$$

1-year default probability: $1 - 0.779 = 22.1\%$

> **Desk Reality: The "Survival to Next Quarter" Test**
>
> For distressed credits, traders often focus on short-term survival. If $h = 25\%$/year, quarterly survival is:
> $$Q(0.25) = e^{-0.25 \times 0.25} = e^{-0.0625} = 93.9\%$$
>
> That's a 6.1% probability of default *this quarter*. When you see spreads above 2000bp, the market is saying there's real risk of imminent default.

### 36.8.4 Sensitivity to Recovery Assumptions

Holding spread fixed at $S = 200$ bp, vary recovery:

| Recovery $R$ | Loss $(1-R)$ | Implied $h$ | 5Y Default Prob |
|--------------|--------------|-------------------|-----------------|
| 20% | 0.80 | 2.50%/year | 11.8% |
| 40% | 0.60 | 3.33%/year | 15.4% |
| 60% | 0.40 | 5.00%/year | 22.1% |

**Interpretation:** Higher assumed recovery means smaller loss per default. To justify the same spread, implied hazard rate must be higher (more defaults of smaller severity). This is why recovery assumptions significantly affect survival curve calibration—a topic explored further in Chapter 42.

### 36.8.5 Caveats and When the Triangle Breaks

The triangle relies on assumptions that are only approximately true for real CDS:

1. **Continuous premium payments** — actual CDS pay quarterly with accruals
2. **Constant hazard rate** — actual curves are not flat
3. **Zero interest rates** — actual discounting matters, especially for long-dated contracts
4. **Rates/credit interactions** — systems often assume independence; correlation can matter

**When the triangle approximation is poor:**

- **High spreads**: Accrued premium effect is quadratic in spread; error grows
- **Long-dated CDS**: Discounting matters more; ignoring it creates error
- **Steep curve slopes**: Constant hazard assumption fails
- **Non-standard recovery**: If recovery is uncertain or contract-specific

The credit triangle is valuable for:
- Building intuition quickly
- Sanity-checking model outputs
- Understanding the spread-hazard-recovery relationship
- Back-of-envelope calculations

But for precise pricing, use the full CDS pricing equations developed in Chapter 41.

### 36.8.6 Risk Objects (Bump Conventions)

When someone says “credit risk went up,” ask: *what object moved and what was held fixed?* This chapter uses explicit bump designs so you can sanity-check numbers across systems.

**(1) Hazard bump (curve risk)** (definition used in this book)
- **Bump object:** the hazard curve $h(t)$.
- **Bump size:** $1$ bp/year $=10^{-4}$ added to $h(t)$ (parallel shift), unless a bucketed bump is stated.
- **Units:** currency per 1 bp/year for the stated notional.
- **Sign (common-sense check):** higher $h$ means more default risk $\Rightarrow$ long risky cashflows usually lose PV.

**Check (magnitude of a 1bp/year hazard bump):** A parallel bump $\Delta h=10^{-4}$ changes deterministic survival multiplicatively:

$$
Q_{\text{bumped}}(t)=Q(t)e^{-\Delta h t}.
$$

At $t=5$y, this factor is $e^{-0.0005}\approx 0.9995$, i.e., about a 0.05% relative drop in survival. The survival change can look small, but the PV impact can still be meaningful on large notionals because it multiplies large cashflows.

**(2) CS01 (spread risk)** (definition used in this book)
- **Bump object:** the quoted spread $S$ (decimal per year).
- **Bump size:** $+1$ bp $=10^{-4}$ per year, with a *curve rebuild* rule (bootstrapped $Q/h$ recomputed consistently).
- **Definition:** $\mathrm{CS01}:=PV(S+1\text{bp})-PV(S)$.
- **Units:** currency per 1 bp for the stated notional.
- **Sign check (typical):** long protection tends to have $\mathrm{CS01}\gt 0$; long cash-bond credit tends to have $\mathrm{CS01}\lt 0$ (verify your system’s sign convention).

---

## 36.9 Term Structure Shapes and Market Interpretation

Credit curve shapes encode the market’s view of **near-term vs long-term** default risk. They also drive carry/rolldown: as time passes, the par spread of a CDS position can move even if the curve is unchanged.

### 36.9.1 Three Canonical Curve Shapes

A representative teaching example uses three CDS term structures:

| CDS Term | Flat with Step (bp) | Upward Sloping (bp) | Steeply Inverted (bp) |
|----------|--------------------|--------------------|----------------------|
| 6M | 50 | 100 | 800 |
| 1Y | 50 | 115 | 600 |
| 2Y | 50 | 135 | 450 |
| 3Y | 50 | 160 | 380 |
| 4Y | 60 | 180 | 350 |
| 5Y | 60 | 195 | 350 |
| 7Y | 60 | 220 | 350 |
| 10Y | 60 | 220 | 350 |

### 36.9.2 What Each Shape Tells You

**Shape 1: Upward Sloping (Normal)**

- **Typical of:** Investment-grade corporates, sovereigns, financial institutions
- **Market interpretation:** "Safe today, uncertain in 5-10 years"
- **Explanation:** Near-term survival is highly probable, but longer horizons introduce uncertainty about business model, management changes, competitive dynamics
- **Example:** A well-capitalized bank might trade at 50bp for 1Y but 150bp for 10Y—the market sees no near-term stress but acknowledges long-term uncertainties

**Shape 2: Flat**

- **Typical of:** Transitional credits, names with stable but elevated risk
- **Market interpretation:** "Risk is roughly constant over time"
- **Explanation:** Neither improving nor deteriorating outlook; hazard rate approximately constant
- **Example:** A BB-rated company with stable cash flows but permanent leverage might trade flat at 300bp across tenors

**Shape 3: Inverted**

- **Typical of:** Distressed credits, names facing imminent stress
- **Market interpretation:** "Will they survive the next quarter? If yes, long-term might be okay"
- **Explanation:** High near-term default risk (debt maturity, liquidity crisis, legal event); if they survive, the business may stabilize
- **Example:** A company facing a debt maturity wall trades at 800bp for 6M but only 350bp for 5Y—the market prices significant near-term risk but a viable business if restructuring succeeds

> **Desk Reality: Reading Curve Shapes in Practice**
>
> When you see an inverted credit curve, ask these questions:
> 1. **What's the catalyst?** (Debt maturity? Covenant breach? Litigation?)
> 2. **What's the cliff?** (When does the near-term risk resolve?)
> 3. **What's the recovery play?** (If they survive, what's the upside?)
>
> Inverted curves often signal event-driven trading opportunities. Distressed debt funds specifically look for inverted curves where they believe the market overestimates near-term risk.
>
> When you see a *very* steep upward-sloping curve (e.g., 30bp at 1Y but 300bp at 10Y), ask:
> 1. **Why is short-term so safe?** (Explicit guarantee? Strong cash position?)
> 2. **What's the long-term uncertainty?** (Business model obsolescence? Regulatory risk?)

### 36.9.3 Forward Hazard Rates and Term Structure

The (instantaneous) forward hazard rate is:

$$h(t) = -\frac{d \ln Q(t)}{dt}.$$

For the three curve shapes above, the forward hazard curves have distinct patterns:

- **Upward sloping spread curve** → upward sloping or stable forward hazard
- **Flat spread curve** → approximately flat forward hazard
- **Inverted spread curve** → *declining* forward hazard (high now, lower later)

Implication: on an upward-sloping curve, a fixed-maturity CDS you own will typically “roll down” toward lower spreads as time passes (all else equal), affecting carry/theta.

---

## 36.10 Arbitrage Constraints on Inverted Curves

### 36.10.1 Why Inverted Curves Have Limits

Not every inverted curve is valid. If a curve is "too inverted," it violates no-arbitrage constraints.
No-arbitrage requires a non-negative forward default rate (hazard), i.e., survival must be non-increasing with maturity.

Mathematically:
$$h(t) \geq 0 \quad \Leftrightarrow \quad Q(t) \text{ is non-increasing}$$

If bootstrapping a curve produces $Q(t_{i+1}) \gt  Q(t_i)$ (survival probability *increases* over time), there's either an arbitrage or a data error.

### 36.10.2 The Arbitrage Argument

Consider a curve where the 6M spread is 800bp but the 1Y spread is only 100bp. This is extremely inverted.

**Can this be arbitrage-free?**

The intuition: buy longer-dated protection and sell shorter-dated protection to earn the extreme spread difference while being close to default-neutral over the short horizon.
1. **Buy protection** on 1Y CDS (you pay 100bp/year)
2. **Sell protection** on 6M CDS (you receive 800bp/year, pro-rated)

For the first 6 months:
- You receive ~400bp from the 6M protection sale
- You pay ~50bp on the 1Y protection purchase
- Net: +350bp carry

If no default occurs in 6 months:
- The 6M contract expires worthless (you keep the premium)
- You still hold 1Y protection, but now it's a 6M protection (rolling down the curve)

If the curve is inverted beyond certain bounds (and under idealized assumptions), the trade can create an arbitrage-like profit opportunity.

### 36.10.3 Quantitative Arbitrage Bounds

O'Kane derives approximate and exact arbitrage bounds. For a curve starting at 800bp at 6M:

$$S_{m} \gtrsim S_{m-1} \times \frac{T_{m-1}}{T_m}$$

This approximation says: the spread at maturity $T_m$ must be at least the spread at $T_{m-1}$ times the ratio of maturities.

**Example approximate lower bounds (starting from 6M = 800bp):**

| CDS Term | Approximate Lower Bound (bp) |
|----------|------------------------------|
| 6M | 800 |
| 1Y | 400 |
| 2Y | 200 |
| 3Y | 133 |
| 4Y | 100 |
| 5Y | 80 |

Interpretation: under the approximation and zero bid/offer, an inverted curve that drops *below* these bounds is arbitrageable.

> **Desk Reality: What to Do When You See Arbitrage**
>
> If you observe a curve that violates these bounds:
>
> 1. **Check data quality first.** Stale quotes, wrong tenors, or bid/offer confusion are common.
> 2. **Check liquidity.** The "arbitrage" may exist on screens but not be executable.
> 3. **Check the name.** Is there a restructuring event, name change, or reference obligation issue?
> 4. **If real, trade it.** But size appropriately—distressed name liquidity is thin.
>
> Extreme inversions that violate bounds are rare in liquid markets but can persist for hours in fast-moving distressed situations.

---

## 36.11 Simulating Default Times

### 36.11.1 The Threshold Simulation Method

For Monte Carlo simulation of credit portfolios, CVA calculations, and stress testing, we need to simulate random default times. McNeil provides a clean algorithm based on the **threshold method**.

**Key Insight:** If $E$ is a standard exponential random variable (rate 1), and we define:

$$\tau = H^{-1}(E)$$

where $H(t) = \int_0^t h(s) ds$ is the cumulative hazard, then $\tau$ has the correct survival distribution.

Some texts write the cumulative hazard as $\Gamma(t)$ and the hazard-rate process as $(\gamma_t)$; the threshold method is the same idea: draw a unit exponential threshold and invert the cumulative hazard.

**Verification:**
$$\Pr(\tau \gt  t) = \Pr(H^{-1}(E) \gt  t) = \Pr(E \gt  H(t)) = e^{-H(t)} = Q(t) \quad \checkmark$$

### 36.11.2 Algorithm: Simulating a Single Default Time

$$\boxed{\text{Algorithm: Threshold Simulation for Default Time}}$$

**Input:** Hazard rate function $h(t)$ (or piecewise constant hazards $h_1, h_2, \ldots$)

**Output:** Simulated default time $\tau$

1. **Draw** `E ~ Exponential(1)` (equivalently, `E = -ln(U)` where `U ~ Uniform(0,1)`)
2. **Compute** cumulative hazard function $H(t) = \int_0^t h(s) ds$
3. **Solve** $H(\tau) = E$ for $\tau$

**For constant hazard $h$:**
$$\tau = \frac{E}{h}$$

**For piecewise constant hazards:**
Find the interval containing $\tau$ by searching:
- If $E \leq H(t_1) = h_1 t_1$: $\tau = E/h_1$
- If $H(t_1) \lt  E \leq H(t_2)$: $\tau = t_1 + (E - H(t_1))/h_2$
- Continue...

### 36.11.3 Worked Example: Constant Hazard

**Given:** $h = 0.02$ (2%/year)
**Draw:** $E = 0.8$ (example exponential sample)

**Compute:**
$$\tau = \frac{E}{h} = \frac{0.8}{0.02} = 40 \text{ years}$$

**Interpretation:** This simulated path shows the credit surviving 40 years before default.

### 36.11.4 Worked Example: Piecewise Hazard

**Given:** $h_1 = 0.01$ for $[0,2)$, $h_2 = 0.03$ for $[2, \infty)$
**Draw:** $E = 0.15$

**Step 1:** Compute $H(2) = h_1 \times 2 = 0.02$

**Step 2:** Check if $E \leq H(2)$: Is $0.15 \leq 0.02$? No.

**Step 3:** Default occurs after $t_1 = 2$:
$$\tau = 2 + \frac{E - H(2)}{h_2} = 2 + \frac{0.15 - 0.02}{0.03} = 2 + 4.33 = 6.33 \text{ years}$$

### 36.11.5 Connection to CVA and Portfolio Credit

This simulation method is the foundation for:

- **CVA Monte Carlo:** Simulate counterparty default time $\tau$, compute exposure at default, average over paths
- **Portfolio credit risk:** Simulate correlated default times for all names, compute portfolio loss distribution
- **Stress testing:** Condition on elevated hazard rates, re-run simulations

McNeil provides multivariate extensions (Algorithm 9.34) for simulating multiple correlated default times, which is essential for CDO pricing and portfolio credit risk.

---

## 36.12 Cox Processes and Stochastic Hazard Rates

### 36.12.1 When Hazard Rates Are Random

In basic reduced-form models, $h(t)$ is a deterministic function calibrated to market data. But in reality, hazard rates change randomly over time as credit conditions evolve.

A **Cox process** (or doubly stochastic process) extends the framework to allow $h(t)$ to be itself a stochastic process. McNeil provides the formal definition:

**Definition (McNeil 9.11):** A random time `tau` is called **doubly stochastic** with respect to background filtration `F_t` if `tau` admits the conditional hazard-rate process `gamma_t` and:

$$
\Pr(\tau \gt t \mid \mathcal{F}_{\infty}) = \exp\left(-\int_0^t \gamma_s\,ds\right)
$$

### 36.12.2 Why Stochastic Hazard Matters

**Correlation through common factors:**

If multiple credits share a common factor driving their hazards, their defaults become correlated. For example:
- Recession hits → all corporate hazard rates increase → defaults cluster

This is how reduced-form models capture default correlation for portfolio credit products (CDOs, index tranches).

**Time-varying credit quality:**

A company's hazard rate might jump when:
- Earnings disappoint
- Leverage increases
- Credit rating is downgraded

Modeling these dynamics requires stochastic hazard rates.

### 36.12.3 Practical Implications

For **single-name CDS pricing**, deterministic hazard rates (calibrated daily to market spreads) are sufficient—we mark to market, not mark to model.

For **portfolio credit risk**, **CVA with wrong-way risk**, and **options on credit spreads**, stochastic hazard matters. McNeil notes that the threshold simulation method extends naturally: simulate both the factor process and the hazard path, then apply the same $\tau = H^{-1}(E)$ formula.

> **Practitioner Note:** Most single-name CDS desks use deterministic hazard curves. Stochastic hazard enters when you need:
> - Default correlation (portfolio products)
> - Wrong-way risk (CVA where exposure and default are correlated)
> - Credit spread options (volatility of hazard matters)
>
> If you're just pricing vanilla CDS, deterministic hazard is the market standard.

---

## 36.13 Worked Examples

### Example 36.1: Constant Hazard → Survival Curve

**Given:** Hazard rate $h = 2\%$ per year = 0.02

**Compute survival probabilities:**

$$Q(t) = e^{-0.02t}$$

| $t$ (years) | $Q(t) = e^{-0.02t}$ |
|-------------|---------------------|
| 1 | $e^{-0.02} = 0.9802$ |
| 3 | $e^{-0.06} = 0.9418$ |
| 5 | $e^{-0.10} = 0.9048$ |

**Default probability by year 5:**

$$\Pr(\tau \leq 5) = 1 - Q(5) = 1 - 0.9048 = 9.52\%$$

---

### Example 36.2: Survival Curve → Average Hazard Rate

**Given:** $Q(5) = 0.90$

**Find the average constant hazard rate over $[0,5]$:**

If $Q(5) = e^{-\bar{h} \cdot 5}$, then:

$$\bar{h} = -\frac{\ln(0.90)}{5} = -\frac{-0.1054}{5} = 0.0211 = 2.11\%/\text{year}$$

---

### Example 36.3: Piecewise Hazard Construction

**Given:**
- $h(t) = 1\%$ for $t \in [0, 2)$
- $h(t) = 3\%$ for $t \in [2, 5)$

**Compute survival probabilities:**

For $t \leq 2$:
$$Q(t) = e^{-0.01t}$$

For $t \gt  2$:
$$Q(t) = Q(2) \cdot e^{-0.03(t-2)}$$

| $t$ | Calculation | $Q(t)$ |
|-----|-------------|--------|
| 1 | $e^{-0.01}$ | 0.9900 |
| 2 | $e^{-0.02}$ | 0.9802 |
| 3 | $0.9802 \cdot e^{-0.03}$ | 0.9512 |
| 5 | $0.9802 \cdot e^{-0.09}$ | 0.8958 |

The upward-sloping hazard structure (1% to 3%) causes faster decay in $Q(t)$ after year 2.

---

### Example 36.4: Recovering Hazard Rates from Survival Curve

**Given:** Annual survival probabilities $Q(0) = 1$, $Q(1) = 0.99$, $Q(2) = 0.975$, $Q(3) = 0.955$

**Compute piecewise hazard rates:**

$$h_i = -\ln\left(\frac{Q(t_i)}{Q(t_{i-1})}\right) \quad (\text{for } \Delta t_i = 1)$$

| Period | Calculation | $h_i$ |
|--------|-------------|-------------|
| Year 0–1 | $-\ln(0.99/1)$ | 1.01%/year |
| Year 1–2 | $-\ln(0.975/0.99)$ | 1.53%/year |
| Year 2–3 | $-\ln(0.955/0.975)$ | 2.07%/year |

The upward-sloping hazard curve implies increasing credit risk over time—a common pattern for investment-grade credits.

---

### Example 36.5: Interval Default Probability

Using Example 36.1's constant hazard ($h = 2\%$):

**Probability of default between years 1 and 3:**

$$\Pr(1 \lt  \tau \leq 3) = Q(1) - Q(3) = 0.9802 - 0.9418 = 0.0384 = 3.84\%$$

---

### Example 36.6: Credit Triangle Application

**Given:** 5-year CDS spread = 150 bp, Recovery = 40%

**Implied hazard rate:**

$$h = \frac{0.0150}{1 - 0.40} = \frac{0.0150}{0.60} = 0.025 = 2.5\%/\text{year}$$

**5-year survival probability (assuming constant hazard):**

$$Q(5) = e^{-0.025 \times 5} = e^{-0.125} = 0.8825 = 88.25\%$$

**5-year default probability:**

$$\Pr(\tau \leq 5) = 1 - 0.8825 = 11.75\%$$

---

### Example 36.7: Expected Loss Calculation

**Given:** Notional $N = \mathrm{USD}\\,10\text{m}$, Recovery $R = 40\%$, 5-year horizon, constant hazard $h = 2\%$

From Example 36.1: $Q(5) = 0.9048$

**Expected loss (undiscounted):**

$$EL = N \cdot (1-R) \cdot (1-Q(5)) = 10\text{m} \times 0.60 \times 0.0952 = \mathrm{USD}\\,571{,}200$$

If hazard doubles to $h = 4\%$:

$Q(5) = e^{-0.20} = 0.8187$

$$EL = 10\text{m} \times 0.60 \times 0.1813 = \mathrm{USD}\\,1{,}087{,}800$$

Doubling the hazard rate nearly doubles the expected loss—a useful sensitivity to remember.

---

### Example 36.8: Sensitivity of Implied Hazard to Recovery

Holding spread fixed at $S = 120$ bp, vary recovery:

| Recovery $R$ | Loss $(1-R)$ | Implied $h = S/(1-R)$ |
|--------------|--------------|----------------------------|
| 20% | 0.80 | 1.50%/year |
| 40% | 0.60 | 2.00%/year |
| 60% | 0.40 | 3.00%/year |

**Interpretation:** Higher assumed recovery means smaller loss per default. To justify the same spread, implied hazard rate must be higher (more defaults of smaller severity). This is why recovery assumptions significantly affect survival curve calibration—a topic explored further in Chapter 42.

---

### Example 36.9: Default Time Simulation

**Given:** $h = 0.03$ (3%/year constant), Draw $E = 1.5$ from Exponential(1)

**Compute default time:**
$$\tau = \frac{E}{h} = \frac{1.5}{0.03} = 50 \text{ years}$$

This path shows long survival; the credit doesn't default for 50 years.

---

### Example 36.10: Risk-Neutral vs Real-World Comparison

**Given (from Hull Table 24.2):** Baa credit (seven-year average hazard rates)

| Measure | Hazard Rate | Ratio (market / historical) |
|---------|-------------|-----------------------------|
| Historical (real-world) | 0.47%/year |  |
| Implied from bond yields (risk-neutral estimate) | 2.38%/year | $2.38/0.47 \approx 5.1\times$ |

**Sanity check (constant-hazard translation to 7Y PD):**
- Historical: $1-e^{-0.0047\cdot 7}\approx 3.24\%$
- Market-implied: $1-e^{-0.0238\cdot 7}\approx 15.35\%$

Interpretation: market-implied hazards used for valuation can be materially larger than historical default-frequency estimates used for scenario and loss forecasting.

---

### Example 36.11 (Worked Example): Quote → $Q/h$ → PV → hazard bump and CS01

**Example Title**: Pricing a risky zero-coupon payoff from a CDS quote (toy model)

**Context**
- You see a 5Y CDS running spread $S$ and want a quick, transparent mapping to survival, PV, and spread risk.
- This example is a *toy* (credit triangle + flat curves) that you can reproduce in a spreadsheet before you move on to full CDS bootstrapping.

**Timeline (Make Dates Concrete)**
- Trade date: 2026-02-15
- Settlement date: 2026-02-17 (assumed)
- Maturity date: 2031-02-15
- Year-fraction to maturity: $T=5.0$ (given explicitly)

**Inputs**
- Notional (par): $N=\mathrm{USD}\\,10{,}000{,}000$
- CDS quote (running): $S=150$ bp $=0.0150$ per year
- Recovery assumption: $R=40\%$ (so $LGD=1-R=60\%$)
- Risk-free discounting (toy): flat continuously-compounded $r=3\%$ per year
- Hazard model (toy): constant hazard $h$, inferred from the credit triangle

**Outputs (What You Produce)**
- Survival $Q(0,T)$
- PV of a defaultable payoff with fractional recovery of par
- Hazard-bump sensitivity (PV change per $+1$ bp/year in $h$)
- CS01 (PV change per $+1$ bp in $S$ with a curve rebuild rule approximated by the triangle)

**Step-by-step**
1. **Translate quote → hazard (triangle):**
   $$h=\frac{S}{1-R}=\frac{0.0150}{0.60}=0.0250.$$
2. **Compute survival to maturity:**
   $$Q(0,T)=e^{-hT}=e^{-0.0250\cdot 5}\approx 0.8825.$$
3. **Specify the payoff you are valuing (fractional recovery of par):**
   - If $\tau\gt T$: receive $N$ at $T$.
   - If $\tau\le T$: receive $RN$ at $\tau$.
4. **Write PV using the default density $f(t)=hQ(t)$:**
   $$PV = N\Big(P(0,T)Q(0,T) + R\int_0^T P(0,t)\,f(t)\,dt\Big).$$
   With flat $r$ and constant $h$, this simplifies to:
   $$PV = N\left(e^{-(r+h)T} + R\frac{h}{r+h}\left(1-e^{-(r+h)T}\right)\right).$$
   Plugging $r=0.03$, $h=0.025$, $T=5$, $R=0.40$, $N=10{,}000{,}000$:
   $$PV \approx \mathrm{USD}\\,8{,}032{,}863.$$
5. **Hazard bump (parallel $+1$ bp/year in $h$):**
   $$\Delta h = 1\text{ bp/year} = 10^{-4}.$$
   Compute $PV(h+\Delta h)-PV(h)\approx -\mathrm{USD}\\,2{,}153$ (per USD 10mm notional).
6. **CS01 (spread $+1$ bp, curve rebuild via triangle):**
   $$h(S)=\frac{S}{1-R}, \qquad \Delta S = 1\text{ bp} = 10^{-4}.$$
   So $\Delta h = \Delta S/(1-R)\approx 1.67$ bp/year, and:
   $$\mathrm{CS01} \approx PV(S+\Delta S)-PV(S) \approx -\mathrm{USD}\\,3{,}588 \;\;(\text{per } \mathrm{USD}\\,10\text{mm}).$$

**Cashflows (table)**
| Date | Cashflow | Explanation |
|---|---|---|
| 2031-02-15 | $+N$ | paid if $\tau\gt T$ |
| $\tau\in(0,T]$ | $+RN$ | paid at default time if $\tau\le T$ |

**P&L / Risk Interpretation**
- The hazard-bump result is negative: higher default intensity makes a long risky payoff worth less.
- The CS01 is also negative for this “long credit” position: wider spreads map to higher hazard $\Rightarrow$ lower PV.
- What is held fixed matters: here we held $r$ and $R$ fixed, and used the triangle as the curve-rebuild rule.

**Sanity Checks**
- Units: $h$ is per year; $T$ is years; $hT$ is unitless in $e^{-hT}$.
- Limits: if $h=0$, then $PV=N e^{-rT}$ (risk-free); if $R=0$, then $PV=N e^{-(r+h)T}$.
- Sign: $PV(h+\Delta h)-PV(h)\lt 0$ for a long risky payoff.

**Debug Checklist (When Your Result Looks Wrong)**
- Did you convert bp to decimals correctly (1 bp = $10^{-4}$)?
- Are you holding recovery $R$ fixed (and using the same recovery convention)?
- Is your spread-to-hazard mapping a triangle approximation or a bootstrapped curve?

See `## References` for sources on the hazard model, the credit triangle, and the risk-neutral vs real-world distinction.

---

## 36.14 Practical Notes

### 36.14.1 Common Pitfalls

**Mixing measures:** Using spread-implied (risk-neutral) hazard rates in a VaR calculation, or using historical (real-world) default rates to price a CDS. Keep the measures consistent with their intended use.

**Year-fraction consistency:** Hazard rates are per-year quantities. Ensure $\Delta t$ uses the same year-fraction convention as the cashflow schedule you are pricing. For CDS premium accrual conventions, follow the contract details (see Chapters 38–41).

**Recovery convention:** Many formulas assume a fixed recovery fraction $R$ with loss $(1-R)$ paid at default. O'Kane notes there are variations: recovery of par, recovery of market value, etc. Know which convention your system uses and ensure consistency across pricing and risk.

**Accrued premium:** The credit triangle assumes continuous premium. Real CDS pay discretely (often quarterly) and include premium accrual to the default date; this matters more when spreads are large.

### 36.14.2 Implementation Notes

**Numerical stability:** When $Q(t) \approx 1$, work with $H(t) = -\ln Q(t)$ to avoid catastrophic cancellation. When $Q(t)$ is very small (distressed credits), track $H(t)$ and exponentiate only when needed for final output.

**Curve "kinks":** Piecewise-constant hazards imply $Q(t)$ is continuous but $h(t)$ jumps at knots. Sensitivities may show artifacts at these points. More sophisticated interpolation schemes (piecewise linear hazard, monotone splines) can smooth these effects.

**Arbitrage detection:** No-arbitrage requires $h(t)\ge 0$, equivalently a non-increasing survival curve. If your bootstrapped curve has $Q(t_{i+1})\gt Q(t_i)$, your inputs/interpolation are inconsistent with the model (or you have a data issue).

### 36.14.3 Verification Tests

Before trusting any survival curve, run these checks:

1. **Boundary check:** $Q(0) = 1$
2. **Monotonicity:** $Q(t)$ non-increasing at all grid points
3. **Range:** $0 \leq Q(t) \leq 1$ for all $t$
4. **Positivity:** All hazard rates $h_i \geq 0$
5. **Probability mass:** $\sum_i [Q(t_{i-1}) - Q(t_i)] = 1 - Q(t_n)$

---

## Summary

1. **Reduced-form credit models** treat default as a random event with default time $\tau$; they dominate in credit derivatives pricing because they generate positive short-dated spreads and calibrate tractably
2. The **survival function** $Q(t) = \Pr(\tau \gt  t)$ is the probability of surviving to time $t$—analogous to a credit "discount factor"
3. The **hazard rate** $h(t)$ is the instantaneous conditional default probability rate; units are $1/\text{year}$
4. The fundamental ODE $Q'(t) = -h(t)Q(t)$ links survival and hazard
5. The solution is $Q(t) = \exp(-\int_0^t h(s) ds) = e^{-H(t)}$
6. **Default density** is $f(t) = h(t)Q(t)$—used for weighting protection payments
7. **Interval default probability** is $\Pr(t_1 \lt  \tau \leq t_2) = Q(t_1) - Q(t_2)$
8. Practitioners use **piecewise-constant hazards** for curve construction and bootstrapping
9. Spread-implied hazards are **risk-neutral** (can be many× higher than real-world); historical rates are **real-world**
10. The **credit triangle** $S = h(1-R)$ provides intuition but requires simplifying assumptions
11. **Curve shapes** signal credit quality: upward-sloping (safe now), inverted (distressed), flat (transitional)
12. **Arbitrage bounds** constrain how inverted curves can get
13. **Threshold simulation** (`\tau = H^{-1}(E)` where `E ~ Exp(1)`) enables Monte Carlo for credit

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Survival function $Q(t)$ | $\Pr(\tau \gt  t)$ | Core primitive for credit pricing; analogous to discount factor |
| Hazard rate $h(t)$ | Conditional default intensity | Analogous to short rate for credit; drives survival dynamics |
| Cumulative hazard $H(t)$ | $\int_0^t h(s) ds$ | $Q(t) = e^{-H(t)}$; useful for numerical stability |
| Default density $f(t)$ | $h(t)Q(t)$ | Weights protection leg payments in CDS pricing |
| Credit triangle | $S = h(1-R)$ | Quick spread ↔ hazard conversion; intuition builder |
| Risk-neutral hazard | Implied from spreads | Used for derivatives pricing |
| Real-world hazard | From historical defaults | Used for risk management and capital |
| Piecewise-constant hazard | $h(t) = h_i$ on $[t_{i-1}, t_i)$ | Industry standard for curve construction |
| Inverted curve | Short spreads > long spreads | Signals distressed credit; near-term survival uncertain |
| Threshold simulation | `\tau = H^{-1}(E)`, `E ~ Exp(1)` | Monte Carlo method for default times |

---

## Notation

| Symbol | Definition |
|--------|------------|
| $\tau$ | Default time random variable |
| $Q(t) = \Pr(\tau \gt  t)$ | Survival probability to time $t$ |
| $F(t) = 1 - Q(t)$ | Cumulative default probability |
| $h(t)$ or $\lambda(t)$ | Hazard rate / default intensity (units: 1/year) |
| $H(t) = \int_0^t h(s) ds$ | Cumulative hazard (dimensionless) |
| $f(t) = h(t)Q(t)$ | Default density (units: 1/year) |
| $R$ | Recovery fraction; LGD $= 1 - R$ |
| $S$ | Credit spread (decimal per year) |
| $\mathbb{Q}$ | Risk-neutral measure |
| $\mathbb{P}$ | Real-world/physical measure |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does $\tau$ represent in reduced-form credit models? | The default time random variable |
| 2 | Define the survival function $Q(t)$ | $Q(t) = \Pr(\tau \gt  t)$, probability of surviving to time $t$ |
| 3 | Define the cumulative default probability $F(t)$ | $F(t) = \Pr(\tau \leq t) = 1 - Q(t)$ |
| 4 | Define the hazard rate $h(t)$ | Instantaneous conditional default probability rate given survival |
| 5 | What are the units of hazard rate? | 1/year (inverse time) |
| 6 | State the survival ODE | $Q'(t) = -h(t)Q(t)$ |
| 7 | Express $Q(t)$ in terms of cumulative hazard | $Q(t) = e^{-H(t)} = \exp(-\int_0^t h(s) ds)$ |
| 8 | Define cumulative hazard $H(t)$ | $H(t) = \int_0^t h(s) ds$ |
| 9 | Express hazard rate in terms of survival function | $h(t) = -d(\ln Q(t))/dt = -Q'(t)/Q(t)$ |
| 10 | What is the default density $f(t)$? | $f(t) = h(t)Q(t) = -Q'(t)$ |
| 11 | Formula for interval default probability? | $\Pr(t_1 \lt  \tau \leq t_2) = Q(t_1) - Q(t_2)$ |
| 12 | If hazard is constant $h$, what is $Q(t)$? | $Q(t) = e^{-h t}$ (exponential survival) |
| 13 | What is the expected default time for constant hazard $h$? | $\mathbb{E}[\tau] = 1/h$ |
| 14 | Piecewise hazard: update formula for $Q(t_i)$? | $Q(t_i) = Q(t_{i-1}) e^{-h_i \Delta t_i}$ |
| 15 | Recover piecewise hazard from survivals? | $h_i = -(1/\Delta t_i) \ln(Q(t_i)/Q(t_{i-1}))$ |
| 16 | What is the credit triangle relationship? | $S = h(1-R)$, so $h = S/(1-R)$ |
| 17 | What measure do spread-implied hazards live in? | Risk-neutral ($\mathbb{Q}$) |
| 18 | What measure do historical default rates live in? | Real-world ($\mathbb{P}$) |
| 19 | Why can risk-neutral PDs exceed real-world PDs? | Spreads include risk premia beyond expected loss |
| 20 | If $S = 120$ bp and $R = 40\%$, what is implied $h$? | $h = 0.012/0.6 = 0.02 = 2\%$/year |
| 21 | What does "default is a surprise" mean? | Can't predict default with certainty; only probability $h dt$ over $dt$ |
| 22 | How does reduced-form differ from structural models? | Models hazard directly; doesn't require asset value below barrier |
| 23 | No-arbitrage constraint on hazard rates? | $h(t) \geq 0$ for all $t$ |
| 24 | No-arbitrage constraint on survival curve? | $Q(t)$ must be non-increasing |
| 25 | What weights default-leg (protection) payments? | $f(t) dt = h(t)Q(t) dt$ or $-dQ(t)$ |
| 26 | What weights premium-leg payments in CDS pricing? | $Q(t_i)$ (survival to payment date) |
| 27 | How to verify $\int_0^T f(t) dt$? | Should equal $1 - Q(T)$ |
| 28 | When is constant hazard assumption appropriate? | Flat spread curves; quick approximations |
| 29 | What is $H(t)$ useful for numerically? | Stability; work with $H = -\ln Q$ when $Q \approx 1$ |
| 30 | Why does recovery assumption matter for implied hazard? | $h = S/(1-R)$; higher $R$ means higher implied $h$ |
| 31 | What does an inverted credit curve signal? | Distressed credit; high near-term default risk, lower long-term if survives |
| 32 | What does an upward-sloping credit curve signal? | Investment grade; safe today, uncertain in long term |
| 33 | How to simulate a default time from survival curve? | Draw `E ~ Exp(1)`, set `\tau = H^{-1}(E)` |
| 34 | What constrains how inverted a curve can be? | No-arbitrage: $h(t) \geq 0$; implies bounds on spread decline rate |
| 35 | Spread 500bp, R=40% → hazard? | $h = 0.05/0.60 = 8.33\%$/year |
| 36 | Hazard 2%, 5 years → survival probability? | $Q(5) = e^{-0.10} = 90.5\%$ |
| 37 | What is a hazard bump sensitivity (what is being bumped)? | Reprice after adding $+1$ bp/year ($10^{-4}$) to the hazard curve $h(t)$; report PV change in currency per 1 bp/year for the stated notional (sign depends on position) |
| 38 | Define CS01 (bump object, size, units, sign). | $\mathrm{CS01}:=PV(S+1\text{bp})-PV(S)$ where $S$ is the quoted spread (decimal/yr) and 1 bp $=10^{-4}$; units are currency per 1 bp; long protection often has CS01 $\gt 0$, long cash credit often has CS01 $\lt 0$ |
| 39 | What is a Cox process / doubly stochastic default? | A model where the hazard rate $h(t)$ is itself random (defaults are conditionally Poisson given the hazard path) |

---

## Mini Problem Set

1. (Compute) Given constant hazard $h = 1.5\%$/year, compute $Q(2)$ and $\Pr(\tau \le 2)$.
2. (Compute) If $Q(3)=0.92$, compute the average constant hazard over $[0,3]$.
3. (Compute) With $Q(1)=0.97$, $Q(2)=0.94$, compute piecewise hazards $h_1$ (year 0–1) and $h_2$ (year 1–2).
4. (Compute) For constant $h=2\%$, compute $\Pr(1\lt \tau\le 3)$.
5. (Concept) Prove that $f(t)=h(t)Q(t)$ implies $\int_0^T f(t)\,dt = 1-Q(T)$.
6. (Compute) Credit triangle: $S=80$ bp and $R=35\%$. Compute $h$.
7. (Compute) $N=\mathrm{USD}\\,5\text{mm}$, $R=40\%$, $Q(5)=0.88$. Compute undiscounted 5Y expected loss.
8. (Compute) Piecewise hazard: $h=1\%$ on $[0,1)$ and $h=4\%$ on $[1,2)$. Compute $Q(2)$.
9. (Compute) Threshold simulation: $E=1.2\sim\mathrm{Exp}(1)$ and constant $h=0.04$. Compute $\tau$.
10. (Concept) When should you use risk-neutral vs real-world default probabilities?
11. (Concept) List three assumptions behind the credit triangle and one situation where it can be misleading.
12. (Desk) A report shows $\mathrm{CS01}=-\mathrm{USD}\\,50{,}000$ per 1bp for a bond position. What does the sign mean, and what methodology questions do you ask?

### Solution Sketches (Selected)
1. $Q(2)=e^{-0.015\cdot 2}=e^{-0.03}\approx 0.9704$. PD $=1-Q(2)\approx 2.96\%$.
2. $\bar{h}=-\ln(0.92)/3 \approx 0.0278$, i.e., $\bar{h}\approx 2.78\%$/year.
6. $h\approx S/(1-R)=0.008/0.65\approx 0.0123$, i.e., $1.23\%$/year.
10. Use risk-neutral (spread-implied) for valuation/hedging; use real-world/historical for scenario analysis and loss forecasting.
12. Negative CS01 means PV falls when spreads widen. Ask: what spread is bumped, what is the curve rebuild rule, what recovery is held fixed, and what notional/price scaling is used (per 1bp, per \$1mm, per 100 par).

## References

- O’Kane, *Modelling Single-name and Multi-name Credit Derivatives*, “The Hazard Rate Model”; “Calculating the Survival Probability”; “The Credit Triangle”; “Detecting Arbitrage in the Curve”
- Hull, *Options, Futures, and Other Derivatives*, “Real-World vs. Risk-Neutral Probabilities” (Table 24.2)
- McNeil, Frey, Embrechts, *Quantitative Risk Management*, hazard rates and cumulative hazard (Chapter 9); “Algorithm 9.14 (univariate threshold simulation)”
- Glasserman, *Monte Carlo Methods in Financial Engineering*, “Default Times and Valuation” (intensity identity and valuation with $r+\lambda$)
- Brigo, Morini, Pallavicini, *Counterparty Credit Risk, Collateral and Funding*, “Intensity Models” (cumulative intensity transform and inversion)
