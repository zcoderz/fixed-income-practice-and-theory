# Chapter 36: Survival Probabilities and Hazard Rates

---

## Introduction

How long until a company defaults? This question lies at the heart of credit derivatives pricing—and the answer is not a single number, but a probability distribution.

When you buy protection on a CDS, you are betting that default will occur before your contract expires. When you sell protection, you are betting it won't. The premium you exchange depends entirely on how the market assesses the *timing* of potential default. Get this wrong, and your CDS is mispriced. Get it systematically wrong, and your credit portfolio's risk measures are fiction.

This chapter develops the mathematical framework for modeling default timing: **survival probabilities** and **hazard rates** (also called default intensities). These are the primitives of *reduced-form* credit modeling—an approach that treats default as a random event that arrives with some probability, rather than modeling the underlying firm dynamics that cause default.

O'Kane describes the reduced-form approach as modeling "default as the first arrival time $\tau$ of a Poisson process. The intensity or hazard rate for this process is given by $\lambda(t)$." Hull similarly defines the hazard rate such that "$\lambda(t) \Delta t$ is the probability of default between time $t$ and $t+\Delta t$ conditional on no earlier default." This framework has become the market standard for pricing single-name credit derivatives, as noted in the influential papers by Jarrow and Turnbull (1995) and Lando (1998).

Why has this framework become dominant in practice? O'Kane explains that the reduced-form approach satisfies several key requirements: it captures the risk of default as a single event at some unknown future time, it handles uncertain recovery, and it produces tractable pricing formulas. McNeil's *Quantitative Risk Management* similarly emphasizes that "credit risk is estimated by modeling default probabilities using stochastic failure rate processes" in the reduced-form paradigm.

We cover:

1. **Reduced-form vs structural models** — why markets use intensity-based approaches
2. **The survival function $Q(t)$** — the probability of no default by time $t$
3. **The hazard rate $\lambda(t)$** — instantaneous conditional default intensity
4. **The fundamental relationship** — how survival and hazard connect through an ODE
5. **Piecewise-constant hazards** — the practitioner's workhorse for curve construction
6. **The credit triangle** — linking spreads, hazard rates, and recovery
7. **Term structure shapes** — what curve patterns tell traders about credit quality
8. **Simulating default times** — Monte Carlo implementation
9. **Arbitrage constraints** — limits on how inverted curves can get

This framework is the foundation for CDS pricing (Chapters 38–41), survival curve bootstrapping (Chapter 42), and credit risk management (Chapter 43). If you understand this chapter, the mechanics of credit derivatives pricing become much clearer.

---

## 36.1 Reduced-Form vs. Structural Approaches

Before diving into the mathematics, it helps to understand where reduced-form models fit in the broader credit modeling landscape. This comparison motivates why the derivatives market has overwhelmingly adopted the hazard rate approach.

### 36.1.1 Structural Models: Merton's Insight and Its Limitations

**Structural models** (Merton, Black-Cox) model the firm's asset value process directly. O'Kane explains that "Merton's idea was to model default as the event which occurs when the value of the assets of a firm fall below the value of the firm's debt." In this framework, equity is a call option on firm assets, and debt is risk-free debt minus a put option.

This approach is elegant and economically intuitive—it connects default risk to observable firm fundamentals. However, O'Kane identifies several practical limitations:

- "Merton's model assumes that the bonds are zero coupon," and extending to coupon-paying bonds significantly increases complexity
- "There is limited transparency regarding the value of the assets of a company"—balance sheets are published only quarterly or semi-annually
- Most critically: "The credit spread for firms for which $A(t) > F$ always tends to zero as $T - t \to 0$. This is not consistent with the credit markets in which even corporates with very high credit ratings have a finite spread at very short maturities"

This last point is particularly damaging for derivatives pricing. Structural models predict that short-dated default probability goes to zero—but market spreads remain positive even for overnight credit exposure. O'Kane concludes that "structural models are not widely used in credit derivatives pricing" despite their theoretical appeal.

### 36.1.2 Reduced-Form Models: Default as a Surprise

**Reduced-form models** take a fundamentally different approach. As O'Kane explains, they are "based on the idea of modelling default as the first arrival time $\tau$ of a Poisson process." Rather than modeling *why* default occurs, we model the *probability* that it occurs.

A key property distinguishes this framework. O'Kane emphasizes: "An important property of the hazard rate approach is that default is always a surprise—since we are modelling default by modelling the probability of a default we do not know whether default will occur in the next period of length $dt$. We only know that it will default with probability $\lambda(t) dt$."

This "surprise" property is precisely what allows reduced-form models to fit positive short-term spreads. The hazard rate can be positive at time zero, implying non-zero short-dated default probability—something structural models cannot achieve without additional modifications.

Hull reinforces this interpretation: "The hazard rate multiplied by $dt$ is the probability of a credit defaulting in the time period from $t$ to $t+dt$ conditional on it having survived to time $t$."

### 36.1.3 Why Reduced-Form Dominates in Practice

O'Kane lists the requirements for a credit derivatives pricing model:

1. The model captures default as a single event at an unknown future time
2. The model captures uncertain recovery at the default time
3. The model captures the randomness of interest rates
4. The model allows for quick, stable, and accurate pricing
5. The model is as simple as possible while satisfying these requirements

Reduced-form models satisfy all five. They calibrate easily to CDS spreads, separate cleanly into discount factors and survival probabilities, and produce tractable pricing formulas. O'Kane notes that "the assumption of independence [between rates and default] is market standard" and that the reduced-form approach "follows the work of Duffie (1998), Hull and White (2000a) and O'Kane and Turnbull (2003)."

> **Desk Reality: Which Model Does Your System Use?**
>
> If you work in credit trading support, you've likely seen both models. Structural models (KMV, Moody's EDF) power credit risk analytics and rating systems—they estimate the *actual* probability of default. Reduced-form models power pricing systems—they ensure CDS quotes are internally consistent.
>
> A common confusion: someone sees Moody's EDF showing 0.5% one-year default probability, but the CDS implies 2%. Both are "right" in their own framework. The CDS spread reflects *risk-neutral* probability (including risk premium), while EDF estimates *real-world* probability. Section 36.7 explains this distinction in detail.

The rest of this chapter develops the reduced-form framework in detail.

---

## 36.2 The Survival Function $Q(t)$

### 36.2.1 Definition and Interpretation

The survival function is the probability that a credit has not defaulted by time $t$:

$$\boxed{Q(t) = \Pr(\tau > t)}$$

where $\tau \in (0, \infty)$ is the random default time. O'Kane uses the notation $Q(t, T)$ for survival from time $t$ to time $T$; when $t = 0$, this simplifies to $Q(0, T)$ or just $Q(T)$.

**Key properties of the survival function:**

1. $Q(0) = 1$ — the credit is alive today
2. $Q(t)$ is non-increasing in $t$ — survival probability cannot increase over time
3. $0 \leq Q(t) \leq 1$ — it's a probability
4. $\lim_{t \to \infty} Q(t) = 0$ — eventual default (for finite-lived entities)

The **cumulative default probability** is simply the complement:

$$F(t) = \Pr(\tau \leq t) = 1 - Q(t)$$

This is analogous to mortality tables in actuarial science. The Financial Risk Analytics text notes that "the reduced-form approach to credit risk relies on the concept of survival probability, defined as the probability $\Pr(\tau > t)$ that a random system with lifetime $\tau$ survives at least over $t$ years."

### 36.2.2 The Survival Curve as a Credit Discount Factor

There is a deep analogy between survival probabilities and risk-free discount factors. O'Kane makes this explicit in a comparative table that reveals the parallel structure:

| Interest Rates | Credit |
|----------------|--------|
| Discount curve $Z(t)$ | Survival curve $Q(t)$ |
| Short rate $r(s)$ | Hazard rate $\lambda(s)$ |
| No-arbitrage: $r(s) > 0$ | No-arbitrage: $\lambda(s) > 0$ |
| $Z(t) = \mathbb{E}[\exp(-\int_0^t r(s) ds)]$ | $Q(t) = \mathbb{E}[\exp(-\int_0^t \lambda(s) ds)]$ |

This parallel is more than aesthetic—it drives the entire approach to credit curve construction. Just as we bootstrap discount factors from interest rate instruments (see Chapter 17), we bootstrap survival probabilities from CDS spreads (see Chapter 42). Just as the short rate drives discount factor dynamics, the hazard rate drives survival probability dynamics.

In practical terms: just as discount factors "discount" future cash flows for time value, survival probabilities "discount" future cash flows for the possibility the credit won't survive to pay them. O'Kane confirms: "As a result we can write the price of the zero recovery zero coupon bond as the product of the risk-free zero coupon bond price and the survival probability":

$$\hat{Z}(0,T) = Z(0,T) \cdot Q(0,T)$$

This factorization—risky discount = riskless discount × survival probability—underlies virtually all credit derivatives pricing under the standard independence assumption.

> **On-Ramp: If You Know Discount Factors, You Know Survival Probabilities**
>
> You already understand from Chapter 2 that $Z(0,T)$ is "the value today of $1 to be received at time $T$." Think of $Q(0,T)$ the same way: it's "the probability today that the issuer will still be alive at time $T$ to make that payment."
>
> When pricing a risky bond, you need both: the time value of money ($Z$) AND the probability of getting paid ($Q$). The product $Z \cdot Q$ is the "credit-adjusted" discount factor.

### 36.2.3 Interval Default Probability

The probability of default occurring in a specific interval is:

$$\boxed{\Pr(t_1 < \tau \leq t_2) = Q(t_1) - Q(t_2)}$$

This follows directly from probability algebra: the probability of surviving to $t_1$ minus the probability of surviving to $t_2$ equals the probability of defaulting in between. This formula is essential for computing the protection leg value in CDS pricing, where we need to weight payments by the probability of default in each period.

---

## 36.3 The Hazard Rate (Default Intensity)

### 36.3.1 Definition: Conditional Default Probability per Unit Time

The hazard rate, also called the *default intensity*, is the instantaneous conditional probability of default given survival. O'Kane provides the mathematical definition:

$$\boxed{\lambda(t) = \lim_{\Delta t \to 0} \frac{1}{\Delta t} \Pr[\tau \leq t + \Delta t \mid \tau > t]}$$

Hull gives an equivalent definition: "The hazard rate $\lambda(t)$ at time $t$ is defined so that $\lambda(t) \Delta t$ is the probability of default between time $t$ and $t + \Delta t$ conditional on no earlier default."

**Units:** The hazard rate has units of $1/\text{year}$ (or inverse time). A hazard rate of $\lambda = 0.02$ means a 2% probability of default per year, given survival—not a 2% probability of default in total.

O'Kane emphasizes this subtle but important interpretation: "The hazard rate multiplied by $dt$ is the probability of a credit defaulting in the time period from $t$ to $t+dt$ conditional on it having survived to time $t$." The conditioning on survival is crucial—it distinguishes the hazard rate from unconditional default probability.

### 36.3.2 Economic Interpretation

If $\lambda(t) = 0.02$, what does this mean practically?

For a small time interval $\Delta t$ (say, one day = 1/365 year):

$$\Pr(\text{default in next day} \mid \text{alive today}) \approx 0.02 \times \frac{1}{365} \approx 0.0055\%$$

The hazard rate is the *per-unit-time* intensity. It tells you how "risky" each instant is, conditional on having survived to that instant.

> **Analogy: Radioactive Decay**
>
> A portfolio of bonds is like a block of **Radioactive Uranium**.
>
> *   **Decay (Default)**: Individual atoms (companies) decay (default) at random times.
> *   **Hazard Rate ($\lambda$)**: This is the **decay rate** (related to half-life). A high $\lambda$ means the block is highly radioactive and atoms are popping off quickly.
> *   **Survival Probability ($Q$)**: The weight of the uranium block remaining at time $t$.
> *   **Intensity**: If you put a Geiger counter next to the block, the "clicks" are the defaults. The rate of clicking is $\lambda$.
>
> **Half-Life Connection:** For constant hazard $\lambda$, the "half-life" (time for 50% to default) is $t_{1/2} = \ln(2)/\lambda \approx 0.693/\lambda$. A 2% hazard rate implies a half-life of about 35 years.

O'Kane provides a powerful Monte Carlo intuition: we can simulate default using Bernoulli trials where at each time step $dt$, default occurs with probability $\lambda(t) dt$. He notes: "It is clear from this algorithm that at time $t$ we cannot know until we do the test [a Bernoulli draw] whether a default will occur. Also, although the probability of defaulting in time $dt$ tends to zero as $dt \to 0$, the probability of defaulting per unit of time does not."

This "per unit time" interpretation—the hazard rate remains finite even as the time step shrinks—is what makes reduced-form models generate positive short-dated spreads.

### 36.3.3 Typical Hazard Rate Ranges by Credit Quality

Understanding typical hazard rate magnitudes helps build intuition. Based on Hull's Table 24.2 and market observations:

| Credit Quality | Rating Proxy | Typical Risk-Neutral Hazard Range |
|---------------|--------------|-----------------------------------|
| Prime | AAA/AA | 0.05% – 0.3% per year |
| High Investment Grade | A | 0.3% – 1.0% per year |
| Lower Investment Grade | BBB | 1.0% – 3.0% per year |
| High Yield | BB/B | 3.0% – 10% per year |
| Distressed | CCC or lower | 10% – 50%+ per year |

> **Desk Reality: Why Traders Care About Hazard Rate Ranges**
>
> When a credit desk quotes you a 5Y CDS at 150bp with 40% recovery, the implied hazard rate is approximately $0.015 / 0.60 = 2.5\%$ per year (using the credit triangle). That's solidly in BBB territory.
>
> If someone tries to sell you that name's CDS at 50bp, you'd immediately suspect a data error—50bp with 40% recovery implies only 0.83% hazard, which would be A-rated territory. Either the spread is wrong, the rating is wrong, or there's news you don't know about.

### 36.3.4 The Fundamental Relationship: Hazard Rate and Survival ODE

Starting from the hazard rate definition, we can derive a differential equation for the survival function. This derivation is fundamental to the entire framework.

For small $\Delta t$:

$$\Pr(t < \tau \leq t + \Delta t \mid \tau > t) \approx \lambda(t) \Delta t$$

Multiply both sides by $\Pr(\tau > t) = Q(t)$:

$$\Pr(t < \tau \leq t + \Delta t) \approx Q(t) \lambda(t) \Delta t$$

Now, the survival at $t + \Delta t$ equals survival at $t$ minus the probability of default in between:

$$Q(t + \Delta t) = Q(t) - Q(t) \lambda(t) \Delta t$$

Rearranging:

$$\frac{Q(t + \Delta t) - Q(t)}{\Delta t} \approx -\lambda(t) Q(t)$$

Taking the limit $\Delta t \to 0$:

$$\boxed{\frac{dQ(t)}{dt} = -\lambda(t) Q(t)}$$

This is the **survival ODE**. Hull derives the same equation (using $V(t)$ for survival): "It follows that $dV/dt = -\lambda(t)V(t)$."

**Unit check:** The left side $dQ/dt$ has units $1/\text{year}$. The right side $(-\lambda) \times Q$ has $(1/\text{year}) \times (1) = 1/\text{year}$. ✓

### 36.3.5 Solving the ODE: Survival in Terms of Cumulative Hazard

This is a separable first-order ODE. Separate variables and integrate:

$$\frac{dQ(t)}{Q(t)} = -\lambda(t) dt$$

$$\int_0^t \frac{dQ(s)}{Q(s)} = -\int_0^t \lambda(s) ds$$

$$\ln Q(t) - \ln Q(0) = -\int_0^t \lambda(s) ds$$

Using the boundary condition $Q(0) = 1$:

$$\boxed{Q(t) = \exp\left(-\int_0^t \lambda(s) ds\right) = e^{-H(t)}}$$

where the **cumulative hazard** (also called the integrated hazard) is:

$$\boxed{H(t) = \int_0^t \lambda(s) ds}$$

Equivalently, we can recover the hazard rate from the survival function:

$$\boxed{\lambda(t) = -\frac{d}{dt} \ln Q(t) = -\frac{Q'(t)}{Q(t)}}$$

O'Kane and Hull both use this exponential representation. O'Kane writes the general stochastic case:

$$Q(t) = \mathbb{E}\left[\exp\left(-\int_0^t \lambda(s) ds\right)\right]$$

The expectation appears when $\lambda(t)$ is itself stochastic (a Cox process). For deterministic hazard rates—the case most commonly used in practice—the expectation disappears and we have the clean exponential form.

### 36.3.6 Sanity Checks

Before trusting any survival formula, verify these properties:

- If $\lambda(t) \geq 0$, then $H(t)$ is non-decreasing and $Q(t)$ is non-increasing ✓
- If $\lambda(t) = 0$ for all $t$, then $Q(t) = 1$ (no default risk) ✓
- If $\lambda(t) = \lambda$ (constant), then $Q(t) = e^{-\lambda t}$ (exponential survival) ✓

---

## 36.4 The Default Density

When the distribution function $F(t) = \Pr(\tau \leq t)$ is absolutely continuous, the **default density** is:

$$f(t) = \frac{dF(t)}{dt} = -\frac{dQ(t)}{dt}$$

From the survival ODE $Q'(t) = -\lambda(t) Q(t)$, we get the key identity:

$$\boxed{f(t) = \lambda(t) Q(t)}$$

**Interpretation:** The probability of defaulting "at time $t$" (more precisely, in $[t, t+dt]$) equals the hazard rate at $t$ times the probability of surviving to $t$. You can only default at $t$ if you're still alive at $t$.

**Unit check:** $f(t)$ is a density (units $1/\text{year}$). The RHS is $\lambda(t) \times Q(t) = (1/\text{year}) \times (1) = 1/\text{year}$. ✓

This identity is essential for pricing protection legs in CDS contracts, where the protection payment occurs at the moment of default. O'Kane's pricing derivations weight payments by this density function—the probability of default arriving in each infinitesimal interval.

### 36.4.1 Verification: Density Integrates to Cumulative Default Probability

As a sanity check, integrate the density:

$$\int_0^T f(t) dt = \int_0^T \lambda(t) Q(t) dt = \int_0^T -Q'(t) dt = Q(0) - Q(T) = 1 - Q(T)$$

This confirms $\int_0^T f(t) dt = F(T) = 1 - Q(T)$. ✓

The density function is also crucial for computing expected default timing. The expected default time is $\mathbb{E}[\tau] = \int_0^\infty t f(t) dt$, which is well-defined when the integral converges.

---

## 36.5 Constant Hazard Rate: The Exponential Case

When the hazard rate is constant, $\lambda(t) = \lambda$, the survival function simplifies to:

$$Q(t) = e^{-\lambda t}$$

This is the **exponential survival** model—the continuous-time analog of geometric default probability. It has several useful properties that make it valuable for building intuition and for quick approximations.

### 36.5.1 Mean Default Time

O'Kane derives the expected default time for constant hazard. Using the density $f(t) = \lambda e^{-\lambda t}$:

$$\mathbb{E}[\tau] = \int_0^\infty t \cdot f(t) dt = \int_0^\infty t \cdot \lambda e^{-\lambda t} dt = \frac{1}{\lambda}$$

The variance is $\text{Var}(\tau) = 1/\lambda^2$, so the standard deviation equals the mean.

**Example:** A credit with hazard rate $\lambda = 0.02$ (2% per year) has expected default time:

$$\mathbb{E}[\tau] = \frac{1}{0.02} = 50 \text{ years}$$

O'Kane provides perspective: "A credit with a hazard rate of 1% therefore has a default time distribution with a mean of 100 years and a variance equal to 10000 years." He notes this "makes it clear that any simulation of default times for a credit with a very small $\lambda$ requires a large number of paths to ensure an acceptably low standard error."

### 36.5.2 The Memoryless Property

The exponential distribution is the only continuous distribution with the memoryless property: given that the credit has survived to time $s$, the remaining survival time has the same distribution as the original survival time.

$$\Pr(\tau > s + t \mid \tau > s) = \Pr(\tau > t)$$

This can be verified directly:
$$\Pr(\tau > s + t \mid \tau > s) = \frac{Q(s+t)}{Q(s)} = \frac{e^{-\lambda(s+t)}}{e^{-\lambda s}} = e^{-\lambda t} = Q(t)$$

The memoryless property is unrealistic for actual credits—past performance does provide information about future default risk. A company that has been struggling for years is more likely to default than one with a strong track record. However, this simplification makes the constant-hazard model analytically tractable and provides a useful baseline.

### 36.5.3 When Constant Hazard Is Appropriate

O'Kane provides clear guidance on when this simplification is valid: "This assumption only works if the spread curves are flat. However, even when they are not flat, an assumption of a constant hazard rate can serve as a good approximation. At the very least, assuming a constant hazard rate allows us to simplify the pricing equations and so helps us to better understand the dependence on the model parameters."

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

Define a time grid $0 = t_0 < t_1 < t_2 < \cdots < t_n$ and assume:

$$\lambda(t) = \lambda_i \quad \text{for } t \in [t_{i-1}, t_i)$$

The hazard rate is constant within each interval but can jump between intervals. This corresponds to a survival curve that is piecewise exponential—$Q(t)$ decays exponentially within each interval, with a possibly different decay rate in each.

> **Visual: The Staircase & The Slope**
>
> *   **Hazard Rate ($\lambda$)**: Imagine a **Staircase**. It stays flat at 1% for Year 1, then jumps up to 2% for Year 2, then 3% for Year 3.
> *   **Survival Curve ($Q$)**: This creates a **kinked slope**. The log-survival graph is a straight line sloping down. At Year 1, the slope gets steeper (more decay). At Year 2, it gets steeper again.
> *   **Bootstrapping**: We build this staircase one step at a time. We use the 1Y CDS spread to build the first step. Then we use the 2Y CDS spread (and the first step) to build the second step.

### 36.6.2 Forward Survival Formulas

The cumulative hazard over interval $i$ is simply the hazard rate times the interval length:

$$H(t_i) - H(t_{i-1}) = \lambda_i \cdot (t_i - t_{i-1}) = \lambda_i \cdot \Delta t_i$$

The survival probability updates multiplicatively:

$$\boxed{Q(t_i) = Q(t_{i-1}) \cdot e^{-\lambda_i \Delta t_i}}$$

Alternatively, cumulative hazard updates additively:

$$\boxed{H(t_i) = H(t_{i-1}) + \lambda_i \Delta t_i}$$

This recursive structure makes bootstrapping efficient: given market prices that imply $Q(t_i)$ at each maturity, we can solve for $\lambda_i$ interval by interval.

### 36.6.3 Inverting: Hazard from Survival

Given survival probabilities at grid points, we can recover the piecewise hazards by inverting the forward formula:

$$\boxed{\lambda_i = -\frac{1}{\Delta t_i} \ln\left(\frac{Q(t_i)}{Q(t_{i-1})}\right)}$$

This formula is central to survival curve bootstrapping from CDS spreads (Chapter 42). Given a term structure of CDS quotes, we price the shortest maturity CDS to find $Q(t_1)$, extract $\lambda_1$, then use this to price the next maturity and find $Q(t_2)$, and so on.

### 36.6.4 Interpolation Schemes

O'Kane discusses interpolation between grid points in detail. The choice of interpolation scheme affects both pricing accuracy and curve smoothness. Three common approaches:

| Scheme | Interpolation Of | Hazard Behavior | Properties |
|--------|-----------------|-----------------|------------|
| Linear in $Q$ | Survival probability | Piecewise non-constant | Can produce negative hazards |
| Linear in $\ln Q$ | Log survival | **Piecewise constant** hazard | Market standard; always positive hazards |
| Linear in $H$ | Cumulative hazard | Piecewise linear hazard | Smooth, but requires care |

O'Kane confirms: "Linear interpolation of the log of the survival probability is equivalent to assuming a piecewise constant forward default rate $h(t)$." This is the **market standard** interpolation scheme because it ensures:

- Survival probabilities remain between 0 and 1
- Hazard rates remain positive (no-arbitrage)
- The curve is continuous in $Q(t)$ but jumps in $\lambda(t)$ at knots

For $t^* \in (t_{n-1}, t_n)$:

$$Q(t^*) = Q(t_{n-1}) \cdot e^{-(t^* - t_{n-1}) \cdot \lambda_{n-1}}$$

> **Implementation Note: What Happens When Bootstrapping Fails**
>
> If your bootstrapping algorithm produces a negative hazard rate $\lambda_i < 0$, you have one of two problems:
> 1. **Data error**: Check your input spreads. Is the curve inverted beyond arbitrage limits?
> 2. **Arbitrage opportunity**: If the data is correct, the market is mispriced. See Section 36.10 on arbitrage bounds.
>
> Never force a negative hazard to zero—that masks the real problem. Either fix the inputs or investigate the arbitrage.

---

## 36.7 Risk-Neutral vs. Real-World Hazard Rates

A critical distinction pervades credit modeling: the difference between **risk-neutral** ($\mathbb{Q}$) and **real-world** ($\mathbb{P}$) default probabilities. Confusing these two measures is one of the most common errors in credit risk management.

### 36.7.1 Spread-Implied Hazards Are Risk-Neutral

When we calibrate survival curves to CDS spreads, the resulting hazard rates are risk-neutral—they are the hazard rates that correctly price the CDS under the risk-neutral measure used for derivatives valuation.

Hull is explicit: "The default probabilities or hazard rates implied from credit spreads are risk-neutral estimates. They can be used to calculate expected cash flows in a risk-neutral world when there is credit risk."

Risk-neutral hazard rates are derived from market prices, not from historical default frequencies. They incorporate:
- The market's best estimate of actual default probability
- A risk premium for bearing credit exposure
- A liquidity premium for holding credit-sensitive instruments

### 36.7.2 Historical Defaults Are Real-World

Default statistics from rating agencies (e.g., Moody's or S&P default studies) give real-world probabilities—the frequency of default observed historically in each rating category.

Hull presents striking evidence of the gap. His Table 24.2 shows "seven-year historical hazard rates" versus "hazard rates from bonds":

| Rating | Historical Hazard Rate | Hazard Rate from Bonds | Ratio | Difference |
|--------|------------------------|------------------------|-------|------------|
| Aaa | 0.04% | 0.67% | 16.8× | 0.63% |
| Aa | 0.06% | 0.78% | 13.0× | 0.72% |
| A | 0.13% | 1.28% | 9.8× | 1.15% |
| Baa | 0.47% | 2.20% | 4.7× | 1.73% |

Risk-neutral hazard rates exceed real-world rates by factors of 5 to 15 or more. Why?

Hull explains the wedge: "A large percentage difference between the two hazard rate estimates in Table 24.2 translates into a small expected excess return on the bond." The excess return compensates investors for:

- **Default risk premium** — investors are risk-averse and require compensation beyond expected losses
- **Systematic risk** — defaults tend to cluster in recessions when investors can least afford losses
- **Liquidity premium** — credit instruments are less liquid than risk-free alternatives
- **Uncertainty premium** — historical default rates are estimates, not certainties

O'Kane explicitly decomposes spreads: the observed spread equals the actuarial (real-world) expected loss plus these additional premia.

### 36.7.3 Why the Ratio Is Largest for High-Quality Credits

Notice that the ratio is 16.8× for Aaa but only 4.7× for Baa. This pattern is economically meaningful:

For high-quality credits (Aaa, Aa), the *actual* default probability is tiny. But the *spread* isn't zero—investors still demand compensation for:
- Liquidity (Treasuries are more liquid than even AAA corporates)
- Jump-to-default risk (small probability of catastrophic loss)
- Model uncertainty (what if our "AAA" estimate is wrong?)

The spread on a AAA bond is almost entirely risk premium and liquidity premium, with minimal expected loss component. Hence the huge ratio.

For lower-quality credits (Baa, BB), actual expected losses become meaningful. The spread reflects genuine loss expectations plus a smaller proportional risk premium. Hence the smaller ratio.

### 36.7.4 Practical Implications

This distinction has direct consequences for how you should use each measure:

**For pricing derivatives:** Use risk-neutral probabilities (from spreads). This ensures your prices are consistent with market-observable instruments and support no-arbitrage pricing.

**For risk management and capital:** Use real-world probabilities (from historical data or adjusted from spreads). VaR, economic capital, and stress testing require estimates of what actually happens, not risk-neutral prices.

Hull emphasizes: "Estimates of the probability of default from historical data should be used when scenario analysis to calculate potential future losses from defaults is being carried out. Estimates of the probability of default from credit spreads should be used when credit-dependent instruments are being valued."

Mixing them up leads to either systematic mispricing (using real-world for pricing) or understated risk measures (using risk-neutral for risk management).

> **Desk Reality: Your CDS Desk Uses Risk-Neutral; Your Credit VaR Uses Real-World**
>
> In a typical bank:
> - **CDS trading desk**: Marks positions using spread-implied hazards (risk-neutral). Their P&L is based on market moves.
> - **Credit VaR team**: Uses historical default data (real-world) adjusted for economic conditions. They measure "what could we lose?"
> - **CVA desk**: Uses risk-neutral for pricing adjustments, but may blend with real-world for wrong-way risk.
>
> A common error: applying CDS-implied PDs to a loan portfolio's economic capital calculation. This would *overstate* expected losses by 5-15×, leading to excessive capital requirements.
>
> The opposite error: using historical PDs to price a CDS. Your hedge would be systematically wrong, and you'd lose money on average.

### 36.7.5 Converting Between Measures

While there's no universal formula to convert between measures (the risk premium varies by name, rating, and market conditions), Hull provides a rough approach for understanding the wedge:

$$\text{Credit spread} \approx (1-R) \times \lambda_{\mathbb{Q}} \approx (1-R) \times \lambda_{\mathbb{P}} + \text{risk premium} + \text{liquidity premium}$$

The "risk premium" in credit is analogous to the equity risk premium—it compensates investors for bearing systematic risk. In credit, this is primarily the risk that defaults cluster in bad economic times (when the marginal utility of wealth is high).

---

## 36.8 The Credit Triangle

Under simplifying assumptions, there is a clean relationship between spread, hazard rate, and recovery. O'Kane derives what he calls the **credit triangle**—a fundamental identity that provides powerful intuition for credit derivatives pricing.

### 36.8.1 Derivation

Consider a stylized CDS-like contract with:
- Continuous premium payments at rate $S$
- Protection payment of $(1-R)$ at default
- Constant hazard rate $\lambda$
- Zero discount rate (for analytical simplicity)

O'Kane presents the premium leg value: "Between time $t$ and time $t+dt$ we have a payment of $S \cdot dt$ provided the credit has not defaulted. Discounting this payment and integrating over the lifetime of the contract" yields the premium leg present value.

The protection leg value is proportional to $\lambda(1-R)$—the hazard rate times the loss given default.

Setting these equal for a par-valued contract (where premium PV = protection PV):

$$\boxed{S = \lambda(1-R)}$$

Equivalently:

$$\boxed{\lambda = \frac{S}{1-R}}$$

O'Kane explains the interpretation: "We call this relationship the credit triangle because it is a function of three variables and knowledge of any two is sufficient to calculate the third. What it states is that the required continuously paid spread compensation for taking on a credit loss of $(1-R)$ equals the hazard rate times $(1-R)$."

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

$$\lambda = \frac{0.0050}{0.60} = 0.0083 = 0.83\%/\text{year}$$

5-year survival (assuming constant hazard):
$$Q(5) = e^{-0.0083 \times 5} = e^{-0.0417} = 0.959 = 95.9\%$$

5-year default probability: $1 - 0.959 = 4.1\%$

**Example B: High Yield (500bp spread)**

Given: CDS spread $S = 500$ bp = 0.05, Recovery $R = 40\%$

$$\lambda = \frac{0.05}{0.60} = 0.0833 = 8.33\%/\text{year}$$

5-year survival:
$$Q(5) = e^{-0.0833 \times 5} = e^{-0.417} = 0.659 = 65.9\%$$

5-year default probability: $1 - 0.659 = 34.1\%$

**Example C: Distressed Credit (2000bp spread)**

Given: CDS spread $S = 2000$ bp = 0.20, Recovery $R = 20\%$ (lower for distressed)

$$\lambda = \frac{0.20}{0.80} = 0.25 = 25\%/\text{year}$$

1-year survival:
$$Q(1) = e^{-0.25} = 0.779 = 77.9\%$$

1-year default probability: $1 - 0.779 = 22.1\%$

> **Desk Reality: The "Survival to Next Quarter" Test**
>
> For distressed credits, traders often focus on short-term survival. If $\lambda = 25\%$/year, quarterly survival is:
> $$Q(0.25) = e^{-0.25 \times 0.25} = e^{-0.0625} = 93.9\%$$
>
> That's a 6.1% probability of default *this quarter*. When you see spreads above 2000bp, the market is saying there's real risk of imminent default.

### 36.8.4 Sensitivity to Recovery Assumptions

Holding spread fixed at $S = 200$ bp, vary recovery:

| Recovery $R$ | Loss $(1-R)$ | Implied $\lambda$ | 5Y Default Prob |
|--------------|--------------|-------------------|-----------------|
| 20% | 0.80 | 2.50%/year | 11.8% |
| 40% | 0.60 | 3.33%/year | 15.4% |
| 60% | 0.40 | 5.00%/year | 22.1% |

**Interpretation:** Higher assumed recovery means smaller loss per default. To justify the same spread, implied hazard rate must be higher (more defaults of smaller severity). This is why recovery assumptions significantly affect survival curve calibration—a topic explored further in Chapter 42.

### 36.8.5 Caveats and When the Triangle Breaks

O'Kane is careful to emphasize the assumptions behind this approximation:

1. **Continuous premium payments** — actual CDS pay quarterly with accruals
2. **Constant hazard rate** — actual curves are not flat
3. **Zero interest rates** — actual discounting matters, especially for long-dated contracts
4. **Independence of rates and hazard** — actual correlation is small but exists

O'Kane notes that "this credit triangle is a very useful equation which is commonly used as a quick way to relate spreads, risk-neutral default probabilities and recovery rates." Hull similarly uses it for rough estimates, noting that "it is approximately true that $\bar{\lambda}(T)(1-R) = s(T)$."

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

---

## 36.9 Term Structure Shapes and Market Interpretation

Understanding what different credit curve shapes *mean* is critical desk knowledge. O'Kane provides explicit discussion with three canonical shapes that every credit practitioner should recognize.

### 36.9.1 Three Canonical Curve Shapes

O'Kane presents three representative CDS term structures in Table 7.2:

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

O'Kane describes: "We have taken three CDS curve shapes: (i) a flat curve with a single step which rises from 50 bp to 60 bp at the four-year maturity, (ii) an upward sloping curve going from 100 bp at the six-month term to 220 bp at 10 years, and (iii) a steeply inverted curve implying a distressed credit."

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

The instantaneous forward hazard rate $h(t)$ is defined analogously to forward interest rates:

$$h(t) = -\frac{d \ln Q(t)}{dt} = \lambda(t)$$

For the three curve shapes above, the forward hazard curves have distinct patterns:

- **Upward sloping spread curve** → upward sloping or stable forward hazard
- **Flat spread curve** → approximately flat forward hazard
- **Inverted spread curve** → *declining* forward hazard (high now, lower later)

O'Kane notes: "A sloped curve will cause the breakeven CDS market spread to the maturity of the CDS to change as we roll through time. If the CDS curve is upward sloping, then the fair market spread of the contract will fall as we move through time."

This has important implications for carry and rolldown in credit portfolios—the curve shape determines whether holding a CDS position generates positive or negative theta.

---

## 36.10 Arbitrage Constraints on Inverted Curves

### 36.10.1 Why Inverted Curves Have Limits

Not every inverted curve is valid. If a curve is "too inverted," it violates no-arbitrage constraints.

O'Kane explains: "The no-arbitrage constraint is that the forward default rate $h(t)$ has to be positive. In other words, we require that the survival probabilities have to be flat or monotonically decreasing with horizon time."

Mathematically:
$$\lambda(t) \geq 0 \quad \Leftrightarrow \quad Q(t) \text{ is non-increasing}$$

If bootstrapping a curve produces $Q(t_{i+1}) > Q(t_i)$ (survival probability *increases* over time), there's either an arbitrage or a data error.

### 36.10.2 The Arbitrage Argument

Consider a curve where the 6M spread is 800bp but the 1Y spread is only 100bp. This is extremely inverted.

**Can this be arbitrage-free?**

The trade to exploit extreme inversion:
1. **Buy protection** on 1Y CDS (you pay 100bp/year)
2. **Sell protection** on 6M CDS (you receive 800bp/year, pro-rated)

For the first 6 months:
- You receive ~400bp from the 6M protection sale
- You pay ~50bp on the 1Y protection purchase
- Net: +350bp carry

If no default occurs in 6 months:
- The 6M contract expires worthless (you keep the premium)
- You still hold 1Y protection, but now it's a 6M protection (rolling down the curve)

The trade generates riskless profit if the curve is inverted beyond certain bounds.

### 36.10.3 Quantitative Arbitrage Bounds

O'Kane derives approximate and exact arbitrage bounds. For a curve starting at 800bp at 6M:

$$S_{m} \gtrsim S_{m-1} \times \frac{T_{m-1}}{T_m}$$

This approximation says: the spread at maturity $T_m$ must be at least the spread at $T_{m-1}$ times the ratio of maturities.

**Example Arbitrage Limits (from O'Kane Table 7.3):**

| CDS Term | Approximate Lower Bound (bp) | Exact $\ln Q(t)$ Lower Bound (bp) |
|----------|------------------------------|-----------------------------------|
| 6M | 800 | 800 |
| 1Y | 400 | 419 |
| 2Y | 200 | 219 |
| 3Y | 133 | 151 |
| 4Y | 100 | 117 |
| 5Y | 80 | 96 |

O'Kane states: "Assuming bid-offer spreads of zero, any inverted spread curve that starts with a 6M spread of 800 bp which drops below this curve is arbitrageable."

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

where $H(t) = \int_0^t \lambda(s) ds$ is the cumulative hazard, then $\tau$ has the correct survival distribution.

**Verification:**
$$\Pr(\tau > t) = \Pr(H^{-1}(E) > t) = \Pr(E > H(t)) = e^{-H(t)} = Q(t) \quad \checkmark$$

McNeil states this formally in Lemma 9.12: "Let $E$ be a standard exponentially distributed rv... Define the random time $\tau$ by $\tau = \Gamma^{-1}(E)$ [where $\Gamma$ is the cumulative hazard]. Then $\tau$ is a doubly stochastic random time with hazard-rate process $(\gamma_t)$."

### 36.11.2 Algorithm: Simulating a Single Default Time

$$\boxed{\text{Algorithm: Threshold Simulation for Default Time}}$$

**Input:** Hazard rate function $\lambda(t)$ (or piecewise constant hazards $\lambda_1, \lambda_2, \ldots$)

**Output:** Simulated default time $\tau$

1. **Draw** $E \sim \text{Exponential}(1)$ (equivalently, $E = -\ln(U)$ where $U \sim \text{Uniform}(0,1)$)
2. **Compute** cumulative hazard function $H(t) = \int_0^t \lambda(s) ds$
3. **Solve** $H(\tau) = E$ for $\tau$

**For constant hazard $\lambda$:**
$$\tau = \frac{E}{\lambda}$$

**For piecewise constant hazards:**
Find the interval containing $\tau$ by searching:
- If $E \leq H(t_1) = \lambda_1 t_1$: $\tau = E/\lambda_1$
- If $H(t_1) < E \leq H(t_2)$: $\tau = t_1 + (E - H(t_1))/\lambda_2$
- Continue...

### 36.11.3 Worked Example: Constant Hazard

**Given:** $\lambda = 0.02$ (2%/year)
**Draw:** $E = 0.8$ (example exponential sample)

**Compute:**
$$\tau = \frac{E}{\lambda} = \frac{0.8}{0.02} = 40 \text{ years}$$

**Interpretation:** This simulated path shows the credit surviving 40 years before default.

### 36.11.4 Worked Example: Piecewise Hazard

**Given:** $\lambda_1 = 0.01$ for $[0,2)$, $\lambda_2 = 0.03$ for $[2, \infty)$
**Draw:** $E = 0.15$

**Step 1:** Compute $H(2) = \lambda_1 \times 2 = 0.02$

**Step 2:** Check if $E \leq H(2)$: Is $0.15 \leq 0.02$? No.

**Step 3:** Default occurs after $t_1 = 2$:
$$\tau = 2 + \frac{E - H(2)}{\lambda_2} = 2 + \frac{0.15 - 0.02}{0.03} = 2 + 4.33 = 6.33 \text{ years}$$

### 36.11.5 Connection to CVA and Portfolio Credit

This simulation method is the foundation for:

- **CVA Monte Carlo:** Simulate counterparty default time $\tau$, compute exposure at default, average over paths
- **Portfolio credit risk:** Simulate correlated default times for all names, compute portfolio loss distribution
- **Stress testing:** Condition on elevated hazard rates, re-run simulations

McNeil provides multivariate extensions (Algorithm 9.34) for simulating multiple correlated default times, which is essential for CDO pricing and portfolio credit risk.

---

## 36.12 Cox Processes and Stochastic Hazard Rates

### 36.12.1 When Hazard Rates Are Random

In basic reduced-form models, $\lambda(t)$ is a deterministic function calibrated to market data. But in reality, hazard rates change randomly over time as credit conditions evolve.

A **Cox process** (or doubly stochastic process) extends the framework to allow $\lambda(t)$ to be itself a stochastic process. McNeil provides the formal definition:

**Definition (McNeil 9.11):** A random time $\tau$ is called **doubly stochastic** with respect to background filtration $(\mathcal{F}_t)$ if $\tau$ admits the conditional hazard-rate process $(\gamma_t)$ and:
$$\Pr(\tau > t \mid \mathcal{F}_\infty) = \exp\left(-\int_0^t \gamma_s \, ds\right)$$

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

**Given:** Hazard rate $\lambda = 2\%$ per year = 0.02

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

If $Q(5) = e^{-\bar{\lambda} \cdot 5}$, then:

$$\bar{\lambda} = -\frac{\ln(0.90)}{5} = -\frac{-0.1054}{5} = 0.0211 = 2.11\%/\text{year}$$

---

### Example 36.3: Piecewise Hazard Construction

**Given:**
- $\lambda(t) = 1\%$ for $t \in [0, 2)$
- $\lambda(t) = 3\%$ for $t \in [2, 5)$

**Compute survival probabilities:**

For $t \leq 2$:
$$Q(t) = e^{-0.01t}$$

For $t > 2$:
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

$$\lambda_i = -\ln\left(\frac{Q(t_i)}{Q(t_{i-1})}\right) \quad (\text{for } \Delta t_i = 1)$$

| Period | Calculation | $\lambda_i$ |
|--------|-------------|-------------|
| Year 0–1 | $-\ln(0.99/1)$ | 1.01%/year |
| Year 1–2 | $-\ln(0.975/0.99)$ | 1.53%/year |
| Year 2–3 | $-\ln(0.955/0.975)$ | 2.07%/year |

The upward-sloping hazard curve implies increasing credit risk over time—a common pattern for investment-grade credits.

---

### Example 36.5: Interval Default Probability

Using Example 36.1's constant hazard ($\lambda = 2\%$):

**Probability of default between years 1 and 3:**

$$\Pr(1 < \tau \leq 3) = Q(1) - Q(3) = 0.9802 - 0.9418 = 0.0384 = 3.84\%$$

---

### Example 36.6: Credit Triangle Application

**Given:** 5-year CDS spread = 150 bp, Recovery = 40%

**Implied hazard rate:**

$$\lambda = \frac{0.0150}{1 - 0.40} = \frac{0.0150}{0.60} = 0.025 = 2.5\%/\text{year}$$

**5-year survival probability (assuming constant hazard):**

$$Q(5) = e^{-0.025 \times 5} = e^{-0.125} = 0.8825 = 88.25\%$$

**5-year default probability:**

$$\Pr(\tau \leq 5) = 1 - 0.8825 = 11.75\%$$

---

### Example 36.7: Expected Loss Calculation

**Given:** Notional $N = \$10\text{m}$, Recovery $R = 40\%$, 5-year horizon, constant hazard $\lambda = 2\%$

From Example 36.1: $Q(5) = 0.9048$

**Expected loss (undiscounted):**

$$EL = N \cdot (1-R) \cdot (1-Q(5)) = 10\text{m} \times 0.60 \times 0.0952 = \$571{,}200$$

If hazard doubles to $\lambda = 4\%$:

$Q(5) = e^{-0.20} = 0.8187$

$$EL = 10\text{m} \times 0.60 \times 0.1813 = \$1{,}087{,}800$$

Doubling the hazard rate nearly doubles the expected loss—a useful sensitivity to remember.

---

### Example 36.8: Sensitivity of Implied Hazard to Recovery

Holding spread fixed at $S = 120$ bp, vary recovery:

| Recovery $R$ | Loss $(1-R)$ | Implied $\lambda = S/(1-R)$ |
|--------------|--------------|----------------------------|
| 20% | 0.80 | 1.50%/year |
| 40% | 0.60 | 2.00%/year |
| 60% | 0.40 | 3.00%/year |

**Interpretation:** Higher assumed recovery means smaller loss per default. To justify the same spread, implied hazard rate must be higher (more defaults of smaller severity). This is why recovery assumptions significantly affect survival curve calibration—a topic explored further in Chapter 42.

---

### Example 36.9: Default Time Simulation

**Given:** $\lambda = 0.03$ (3%/year constant), Draw $E = 1.5$ from Exponential(1)

**Compute default time:**
$$\tau = \frac{E}{\lambda} = \frac{1.5}{0.03} = 50 \text{ years}$$

This path shows long survival; the credit doesn't default for 50 years.

---

### Example 36.10: Risk-Neutral vs Real-World Comparison

**Given (from Hull Table 24.2):** Baa credit

| Measure | Hazard Rate | 5-Year Survival | 5-Year PD |
|---------|-------------|-----------------|-----------|
| Historical (real-world) | 0.47%/year | $e^{-0.0235} = 97.7\%$ | 2.3% |
| Spread-implied (risk-neutral) | 2.20%/year | $e^{-0.11} = 89.6\%$ | 10.4% |

**Ratio:** 2.20 / 0.47 = 4.7×

The CDS market prices Baa credit as if it's 4.7× more likely to default than historical experience suggests. The difference is risk premium—compensation for bearing credit risk, especially during economic stress.

---

## 36.14 Practical Notes

### 36.14.1 Common Pitfalls

**Mixing measures:** Using spread-implied (risk-neutral) hazard rates in a VaR calculation, or using historical (real-world) default rates to price a CDS. Keep the measures consistent with their intended use.

**Year-fraction consistency:** Hazard rates are per-year quantities. Ensure $\Delta t$ uses the same day-count basis as your curve construction. CDS markets typically use Actual/360 for premium payments.

**Recovery convention:** Many formulas assume a fixed recovery fraction $R$ with loss $(1-R)$ paid at default. O'Kane notes there are variations: recovery of par, recovery of market value, etc. Know which convention your system uses and ensure consistency across pricing and risk.

**Accrued premium:** The credit triangle assumes continuous premium. Actual CDS pay quarterly with accrued premium on default. O'Kane shows the accrued-premium effect is quadratic in spread: higher spreads make the approximation less accurate.

### 36.14.2 Implementation Notes

**Numerical stability:** When $Q(t) \approx 1$, work with $H(t) = -\ln Q(t)$ to avoid catastrophic cancellation. When $Q(t)$ is very small (distressed credits), track $H(t)$ and exponentiate only when needed for final output.

**Curve "kinks":** Piecewise-constant hazards imply $Q(t)$ is continuous but $\lambda(t)$ jumps at knots. Sensitivities may show artifacts at these points. More sophisticated interpolation schemes (piecewise linear hazard, monotone splines) can smooth these effects.

**Arbitrage detection:** O'Kane emphasizes: "The no-arbitrage constraint is that the forward default rate $h(t)$ has to be positive." In other words, survival probabilities must be monotonically non-increasing. If your bootstrapped curve violates this—$Q(t_{i+1}) > Q(t_i)$—there's an arbitrage or data error in your inputs.

### 36.14.3 Verification Tests

Before trusting any survival curve, run these checks:

1. **Boundary check:** $Q(0) = 1$
2. **Monotonicity:** $Q(t)$ non-increasing at all grid points
3. **Range:** $0 \leq Q(t) \leq 1$ for all $t$
4. **Positivity:** All hazard rates $\lambda_i \geq 0$
5. **Probability mass:** $\sum_i [Q(t_{i-1}) - Q(t_i)] = 1 - Q(t_n)$

---

## Summary

1. **Reduced-form credit models** treat default as a random event with default time $\tau$; they dominate in credit derivatives pricing because they generate positive short-dated spreads and calibrate tractably
2. The **survival function** $Q(t) = \Pr(\tau > t)$ is the probability of surviving to time $t$—analogous to a credit "discount factor"
3. The **hazard rate** $\lambda(t)$ is the instantaneous conditional default probability rate; units are $1/\text{year}$
4. The fundamental ODE $Q'(t) = -\lambda(t)Q(t)$ links survival and hazard
5. The solution is $Q(t) = \exp(-\int_0^t \lambda(s) ds) = e^{-H(t)}$
6. **Default density** is $f(t) = \lambda(t)Q(t)$—used for weighting protection payments
7. **Interval default probability** is $\Pr(t_1 < \tau \leq t_2) = Q(t_1) - Q(t_2)$
8. Practitioners use **piecewise-constant hazards** for curve construction and bootstrapping
9. Spread-implied hazards are **risk-neutral** (typically 5-15× higher than real-world); historical rates are **real-world**
10. The **credit triangle** $S = \lambda(1-R)$ provides intuition but requires simplifying assumptions
11. **Curve shapes** signal credit quality: upward-sloping (safe now), inverted (distressed), flat (transitional)
12. **Arbitrage bounds** constrain how inverted curves can get
13. **Threshold simulation** ($\tau = H^{-1}(E)$ where $E \sim \text{Exp}(1)$) enables Monte Carlo for credit

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Survival function $Q(t)$ | $\Pr(\tau > t)$ | Core primitive for credit pricing; analogous to discount factor |
| Hazard rate $\lambda(t)$ | Conditional default intensity | Analogous to short rate for credit; drives survival dynamics |
| Cumulative hazard $H(t)$ | $\int_0^t \lambda(s) ds$ | $Q(t) = e^{-H(t)}$; useful for numerical stability |
| Default density $f(t)$ | $\lambda(t)Q(t)$ | Weights protection leg payments in CDS pricing |
| Credit triangle | $S = \lambda(1-R)$ | Quick spread ↔ hazard conversion; intuition builder |
| Risk-neutral hazard | Implied from spreads | Used for derivatives pricing |
| Real-world hazard | From historical defaults | Used for risk management and capital |
| Piecewise-constant hazard | $\lambda(t) = \lambda_i$ on $[t_{i-1}, t_i)$ | Industry standard for curve construction |
| Inverted curve | Short spreads > long spreads | Signals distressed credit; near-term survival uncertain |
| Threshold simulation | $\tau = H^{-1}(E)$, $E \sim \text{Exp}(1)$ | Monte Carlo method for default times |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $\tau$ | Default time random variable |
| $Q(t) = \Pr(\tau > t)$ | Survival probability to time $t$ |
| $F(t) = 1 - Q(t)$ | Cumulative default probability |
| $\lambda(t)$ or $h(t)$ | Hazard rate / default intensity (units: 1/year) |
| $H(t) = \int_0^t \lambda(s) ds$ | Cumulative hazard (dimensionless) |
| $f(t) = \lambda(t)Q(t)$ | Default density (units: 1/year) |
| $R$ | Recovery fraction; LGD $= 1 - R$ |
| $S$ | Credit spread (decimal per year) |
| $\mathbb{Q}$ | Risk-neutral measure |
| $\mathbb{P}$ | Real-world/physical measure |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does $\tau$ represent in reduced-form credit models? | The default time random variable |
| 2 | Define the survival function $Q(t)$ | $Q(t) = \Pr(\tau > t)$, probability of surviving to time $t$ |
| 3 | Define the cumulative default probability $F(t)$ | $F(t) = \Pr(\tau \leq t) = 1 - Q(t)$ |
| 4 | Define the hazard rate $\lambda(t)$ | Instantaneous conditional default probability rate given survival |
| 5 | What are the units of hazard rate? | 1/year (inverse time) |
| 6 | State the survival ODE | $Q'(t) = -\lambda(t)Q(t)$ |
| 7 | Express $Q(t)$ in terms of cumulative hazard | $Q(t) = e^{-H(t)} = \exp(-\int_0^t \lambda(s) ds)$ |
| 8 | Define cumulative hazard $H(t)$ | $H(t) = \int_0^t \lambda(s) ds$ |
| 9 | Express hazard rate in terms of survival function | $\lambda(t) = -d(\ln Q(t))/dt = -Q'(t)/Q(t)$ |
| 10 | What is the default density $f(t)$? | $f(t) = \lambda(t)Q(t) = -Q'(t)$ |
| 11 | Formula for interval default probability? | $\Pr(t_1 < \tau \leq t_2) = Q(t_1) - Q(t_2)$ |
| 12 | If hazard is constant $\lambda$, what is $Q(t)$? | $Q(t) = e^{-\lambda t}$ (exponential survival) |
| 13 | What is the expected default time for constant hazard $\lambda$? | $\mathbb{E}[\tau] = 1/\lambda$ |
| 14 | Piecewise hazard: update formula for $Q(t_i)$? | $Q(t_i) = Q(t_{i-1}) e^{-\lambda_i \Delta t_i}$ |
| 15 | Recover piecewise hazard from survivals? | $\lambda_i = -(1/\Delta t_i) \ln(Q(t_i)/Q(t_{i-1}))$ |
| 16 | What is the credit triangle relationship? | $S = \lambda(1-R)$, so $\lambda = S/(1-R)$ |
| 17 | What measure do spread-implied hazards live in? | Risk-neutral ($\mathbb{Q}$) |
| 18 | What measure do historical default rates live in? | Real-world ($\mathbb{P}$) |
| 19 | Why can risk-neutral PDs exceed real-world PDs? | Spreads include risk premia beyond expected loss |
| 20 | If $S = 120$ bp and $R = 40\%$, what is implied $\lambda$? | $\lambda = 0.012/0.6 = 0.02 = 2\%$/year |
| 21 | What does "default is a surprise" mean? | Can't predict default with certainty; only probability $\lambda dt$ over $dt$ |
| 22 | How does reduced-form differ from structural models? | Models hazard directly; doesn't require asset value below barrier |
| 23 | No-arbitrage constraint on hazard rates? | $\lambda(t) \geq 0$ for all $t$ |
| 24 | No-arbitrage constraint on survival curve? | $Q(t)$ must be non-increasing |
| 25 | What weights default-leg (protection) payments? | $f(t) dt = \lambda(t)Q(t) dt$ or $-dQ(t)$ |
| 26 | What weights premium-leg payments in CDS pricing? | $Q(t_i)$ (survival to payment date) |
| 27 | How to verify $\int_0^T f(t) dt$? | Should equal $1 - Q(T)$ |
| 28 | When is constant hazard assumption appropriate? | Flat spread curves; quick approximations |
| 29 | What is $H(t)$ useful for numerically? | Stability; work with $H = -\ln Q$ when $Q \approx 1$ |
| 30 | Why does recovery assumption matter for implied hazard? | $\lambda = S/(1-R)$; higher $R$ means higher implied $\lambda$ |
| 31 | What does an inverted credit curve signal? | Distressed credit; high near-term default risk, lower long-term if survives |
| 32 | What does an upward-sloping credit curve signal? | Investment grade; safe today, uncertain in long term |
| 33 | How to simulate a default time from survival curve? | Draw $E \sim \text{Exp}(1)$, set $\tau = H^{-1}(E)$ |
| 34 | What constrains how inverted a curve can be? | No-arbitrage: $\lambda(t) \geq 0$; implies bounds on spread decline rate |
| 35 | Spread 500bp, R=40% → hazard? | $\lambda = 0.05/0.60 = 8.33\%$/year |
| 36 | Hazard 2%, 5 years → survival probability? | $Q(5) = e^{-0.10} = 90.5\%$ |
| 37 | What hazard rate range is typical for BBB credits? | Approximately 1% – 3% per year (risk-neutral) |
| 38 | What is a Cox process / doubly stochastic default? | A model where the hazard rate $\lambda(t)$ is itself random |

---

## Mini Problem Set

*Solutions provided for questions 1–10.*

---

**1.** Given constant hazard $\lambda = 1.5\%$/year, compute $Q(2)$ and $\Pr(\tau \leq 2)$.

**Solution:** $Q(2) = e^{-0.015 \times 2} = e^{-0.03} = 0.9704$. Default probability $= 1 - 0.9704 = 2.96\%$.

---

**2.** If $Q(3) = 0.92$, compute the average constant hazard over $[0,3]$.

**Solution:** $\bar{\lambda} = -\ln(0.92)/3 = 0.0834/3 = 2.78\%$/year.

---

**3.** With $Q(1) = 0.97$, $Q(2) = 0.94$, compute piecewise hazards $\lambda_1$, $\lambda_2$.

**Solution:** $\lambda_1 = -\ln(0.97) = 3.05\%$/year. $\lambda_2 = -\ln(0.94/0.97) = -\ln(0.9691) = 3.14\%$/year.

---

**4.** For constant $\lambda = 2\%$, compute $\Pr(1 < \tau \leq 3)$.

**Solution:** $Q(1) = e^{-0.02} = 0.9802$, $Q(3) = e^{-0.06} = 0.9418$. Interval PD $= 0.9802 - 0.9418 = 3.84\%$.

---

**5.** Prove that $f(t) = \lambda(t)Q(t)$ implies $\int_0^T f(t) dt = 1 - Q(T)$.

**Solution:** Since $Q'(t) = -\lambda(t)Q(t) = -f(t)$, we have $\int_0^T f(t) dt = -\int_0^T Q'(t) dt = Q(0) - Q(T) = 1 - Q(T)$. ∎

---

**6.** Using credit triangle, compute $\lambda$ for $S = 80$ bp, $R = 35\%$.

**Solution:** $\lambda = 0.008/(1 - 0.35) = 0.008/0.65 = 1.23\%$/year.

---

**7.** For $N = \$5$m, $R = 40\%$, $Q(5) = 0.88$, compute undiscounted 5-year expected loss.

**Solution:** $EL = 5\text{m} \times 0.60 \times (1 - 0.88) = 5\text{m} \times 0.60 \times 0.12 = \$360{,}000$.

---

**8.** Build $Q(2)$ given $\lambda = 1\%$ on $[0,1)$ and $\lambda = 4\%$ on $[1,2)$.

**Solution:** $Q(1) = e^{-0.01} = 0.9900$. $Q(2) = Q(1) \cdot e^{-0.04} = 0.9900 \times 0.9608 = 0.9512$.

---

**9.** Given $E = 1.2$ from Exponential(1) and $\lambda = 0.04$, compute simulated default time.

**Solution:** $\tau = E/\lambda = 1.2/0.04 = 30$ years.

---

**10.** A Baa credit has historical hazard 0.47%/year but spread-implied hazard 2.20%/year. What is the ratio, and what does it represent?

**Solution:** Ratio = 2.20/0.47 = 4.7×. This represents the credit risk premium—the multiple by which the market prices credit risk above actuarial expected loss, compensating for systematic risk, liquidity, and uncertainty.

---

**11.** Explain why a short-maturity credit can trade at non-zero spread even when one-day default probability is tiny.

---

**12.** What happens to $\lambda(t)$ and $Q(t)$ at knot points in a piecewise-constant hazard curve?

---

**13.** Give two reasons a bond spread may differ from a CDS spread (the CDS-bond basis).

---

**14.** If you observe different survival curves from bonds vs CDS for the same name, list three diagnostics you would run.

---

**15.** Derive $\lambda(t) = -\frac{d}{dt}\ln Q(t)$ from $Q(t) = e^{-H(t)}$.

---

**16.** If $Q(10) = 0.60$, what is the implied 10-year average hazard? Interpret the result.

---

**17.** Given an inverted curve with 6M spread = 800bp, what is the approximate arbitrage lower bound for the 1Y spread?

---

**18.** A credit has piecewise hazards: 2% for [0,2), 5% for [2,5). Draw $E = 0.25$. Compute $\tau$.

---

**19.** Show why a curve with 6M=800bp and 1Y=100bp violates no-arbitrage using O'Kane's approximation.

---

**20.** Conceptually describe how risk-neutral hazard rates enter CVA-style expected exposure weighting.

---

## Source Map

### (A) Verified Facts — Source-Backed

| Fact | Source |
|------|--------|
| Reduced-form models based on Poisson process first arrival time | O'Kane Ch 3.5–3.6, citing Jarrow-Turnbull (1995), Lando (1998) |
| Hazard rate definition as instantaneous conditional default probability | O'Kane eq. 3.5, Hull Ch 24 |
| "Default is always a surprise" in reduced-form models | O'Kane Ch 3.6, explicit statement |
| Survival ODE $Q'(t) = -\lambda(t)Q(t)$ and exponential solution | O'Kane Ch 3.6.1, Hull Ch 24 |
| Structural models: "credit spread tends to zero as $T-t \to 0$" | O'Kane Ch 3.4, explicit statement |
| Credit triangle $S = \lambda(1-R)$ derivation | O'Kane Ch 3.10 |
| Risk-neutral vs real-world hazard distinction | Hull Ch 24.5, O'Kane Ch 3.11 |
| Hull Table 24.2: historical vs spread-implied hazard ratios | Hull Ch 24, Table 24.2 |
| Analogy between discount factors and survival probabilities (table) | O'Kane Ch 7, explicit table |
| Piecewise constant hazard interpolation | O'Kane Ch 7.4 |
| No-arbitrage requires $\lambda(t) \geq 0$ | O'Kane Ch 7.7 |
| Three canonical curve shapes (flat, upward, inverted) | O'Kane Ch 7.2, Table 7.2 |
| Arbitrage bounds on inverted curves | O'Kane Ch 7.7, Table 7.3 |
| Expected default time for constant hazard is $1/\lambda$ | O'Kane Ch 3.9.3 |
| Default density $f(t) = \lambda(t)Q(t)$ | O'Kane pricing derivations, standard probability theory |
| Independence assumption is "market standard" for CDS pricing | O'Kane Ch 6.3 |
| CDS pricing follows Duffie (1998), Hull-White (2000a), O'Kane-Turnbull (2003) | O'Kane Ch 6.4 |
| Doubly stochastic (Cox process) definition | McNeil Ch 9.2.3, Definition 9.11 |
| Threshold simulation algorithm | McNeil Ch 9, Lemma 9.12, Algorithm 9.14 |
| Multivariate threshold simulation | McNeil Ch 9, Algorithm 9.34 |

### (B) Claude-Extended Content

| Content | Basis |
|---------|-------|
| "Desk Reality" boxes on measure confusion, curve interpretation | Extended from O'Kane/Hull concepts with practical trading context |
| Typical hazard rate ranges by rating | Based on Hull Table 24.2 with interpolation for intermediate ratings |
| Decision tree for when to use constant hazard | Synthesized from O'Kane guidance with practical formatting |
| Back-of-envelope multipliers for credit triangle | Derived from credit triangle formula with standard recovery assumptions |
| Simulation worked examples | Constructed to illustrate McNeil algorithms with specific numbers |

### (C) Reasoned Inference — Derived from (A)

| Fact | Derivation |
|------|------------|
| Piecewise hazard inversion $\lambda_i = -(1/\Delta t)\ln(Q_i/Q_{i-1})$ | Direct integration of constant hazard over interval, inverted |
| Interval default probability $Q(t_1) - Q(t_2)$ | Probability algebra from survival definition |
| Cumulative hazard $H(t)$ properties | Integration of hazard rate definition |
| Unit checks for all formulas | Dimensional analysis applied to definitions |
| Memoryless property verification | Direct calculation from exponential survival formula |
| Half-life formula $t_{1/2} = \ln(2)/\lambda$ | Standard exponential decay derivation |

### (D) Flagged Uncertainties

| Item | Uncertainty |
|------|-------------|
| Recovery convention (par vs market-value vs face value) | Multiple conventions exist; not specified without contract/ISDA details |
| Exact spread component decomposition (risk premium, liquidity premium, etc.) | O'Kane mentions components qualitatively; exact magnitudes are market-specific and time-varying |
| Typical hazard ranges for specific ratings in current markets | I'm not sure about current market levels; values provided are illustrative based on historical patterns |
| Precise CDS-bond basis drivers | Multiple factors cited in sources; relative importance varies by name and market conditions |
