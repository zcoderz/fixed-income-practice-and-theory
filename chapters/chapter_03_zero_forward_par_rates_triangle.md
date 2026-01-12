# Chapter 3: Zero Rates, Forward Rates, Par Rates — The Triangle

---

## Conventions & Notation

### Time and Discount Factors

| Symbol | Definition |
|--------|------------|
| $T$ | Time measured in **years** from today ($t=0$) |
| $P(0,T)$ | **Discount factor**: present value at $t=0$ of \$1 paid at $T$ (also written $d(T)$) |

### Zero (Spot) Rates

| Convention | Formula |
|------------|---------|
| **Continuously compounded** | $z_c(T)$ satisfies $P(0,T) = e^{-z_c(T) \cdot T}$ |
| **Simple-interest** over $[0,T]$ | $z_s(T)$ satisfies $P(0,T) = \dfrac{1}{1 + z_s(T) \cdot T}$ |
| **$m$-times-per-year compounding** | $z_m(T)$ satisfies $P(0,T) = \left(1 + \dfrac{z_m(T)}{m}\right)^{-mT}$ |

### Forward Rates for Period $[T_1, T_2]$ where $T_2 > T_1$

| Convention | Formula |
|------------|---------|
| **Simple** (money-market style) | $F_s(0;T_1,T_2)$ satisfies $1 + F_s(0;T_1,T_2) \cdot \tau = \dfrac{P(0,T_1)}{P(0,T_2)}$, where $\tau = T_2 - T_1$ |
| **Continuously compounded** | $f_c(0;T_1,T_2) = \dfrac{1}{\tau} \ln\left(\dfrac{P(0,T_1)}{P(0,T_2)}\right)$ |
| **Instantaneous forward** | $f(0,T) = -\dfrac{\partial}{\partial T} \ln P(0,T)$ |

### Par Rates

- **Par coupon** for a bond (or **par swap fixed rate**) is the fixed rate that makes PV of fixed-leg cashflows equal par
- Expressed using the **annuity**: $A(0) = \sum_i \tau_i P(0,T_i)$

---

## Setup

This chapter formalizes the tight mapping among four objects:

1. Discount factors $P(0,T)$
2. Zero rates $z(T)$ (must specify compounding basis)
3. Forward rates $f(0;T_1,T_2)$
4. Par rates (bond par coupons and/or par swap rates)

**The key idea:** Once you pick conventions (day count, compounding basis, payment dates), **any one of these curves determines the others**.

---

## Conventions Used in This Chapter

1. **Primary object:** the discount curve $\{P(0,T)\}_{T \geq 0}$, with $P(0,0) = 1$

2. **Positivity:** $P(0,T) > 0$ for all $T$ (a basic no-arbitrage requirement; otherwise you can create infinite-money trades by discounting "through zero")

3. **Compounding bases used here:**
   - **Continuous compounding** for clean calculus identities and "instantaneous forwards." Discounting at rate $R$ for $n$ years multiplies by $e^{-Rn}$
   - **Simple interest** for money-market-style rates over an accrual period; a discount factor for a $d$-day period is $1/(1 + r \cdot d/360)$

4. **Accrual fractions:** for a set of payment dates $0 < T_1 < \dots < T_n$, let $\tau_i$ be the year fraction for $[T_{i-1}, T_i]$. For swaps/money markets, the floating payment is typically $L \times R \times n/360$ under actual/360

---

## Notation Glossary

| Symbol | Definition |
|--------|------------|
| $t$ | Valuation time (here mostly $t=0$) |
| $T$ | Maturity (years from $t=0$) |
| $P(0,T)$ | Discount factor to $T$ |
| $d(T)$ | Same as $P(0,T)$ (Tuckman's notation) |
| $z_c(T)$ | Continuously compounded zero rate to $T$ |
| $z_s(T)$ | Simple-interest zero rate over $[0,T]$ |
| $z_m(T)$ | Zero rate compounded $m$ times per year |
| $F_s(0;T_1,T_2)$ | Simple forward rate for $[T_1,T_2]$ |
| $f_c(0;T_1,T_2)$ | Continuously compounded forward rate for $[T_1,T_2]$ |
| $f(0,T)$ | Instantaneous forward rate at maturity $T$ (continuous curve) |
| $c_{\text{par}}$ | Par coupon rate (bond) per year |
| $S_{\alpha,\beta}(t)$ | Par (forward) swap rate for swap from $T_\alpha$ to $T_\beta$ |
| $A(0)$ | Fixed-leg annuity PV: $A(0) = \sum_{i=1}^n \tau_i P(0,T_i)$ |

---

## Core Concepts

### 1) Discount Factors $P(0,T)$

**Formal Definition:**

$P(0,T)$ is the time-0 price of a default-free zero-coupon bond paying 1 unit of currency at time $T$. Equivalently, $P(0,T)$ is the discount applied to a deterministic cashflow at $T$.

**Intuition:**

Discount factors are the "primitive" of the term structure: they tell you *how much a future dollar is worth today*. Everything else—zero rates, forwards, par coupons—are just different coordinate systems for the same information.

**Trading / Risk / Portfolio Practice:**

- **Pricing:** PV of any fixed cashflow stream is $\sum \text{CF}_i \cdot P(0,T_i)$
- **Curve quoting:** desks may quote a curve as par swap rates, but internally convert to discount factors for valuation

---

### 2) Zero (Spot) Rates $z(T)$

**Formal Definition (Must Specify Compounding):**

**Continuous:**

Define $z_c(T)$ by:

$$P(0,T) = e^{-z_c(T) \cdot T}$$

The spot rate is defined via the exponential discounting relationship.

**Semiannual:**

If $\hat{s}(t)$ is the semiannually compounded spot rate, then:

$$d(t) = \frac{1}{(1 + \hat{s}(t)/2)^{2t}}$$

**Simple interest (money-market-style over $d$ days):**

A discount factor over $d$ days can be written $1/(1 + r \cdot d/360)$.

**Intuition:**

A zero rate is "the one rate for the whole horizon" that would produce the same discount factor under a chosen compounding convention.

**Trading / Risk / Portfolio Practice:**

- Zero rates are often plotted as the "zero curve." Forward rates are extracted from it and are central to pricing FRAs, floaters, and many derivatives
- **Convention risk:** quoting a "5y zero rate" is meaningless unless you know compounding and day count; compounding frequency changes the *units* of the rate, like miles vs kilometers

---

### 3) Forward Rates $f(0;T_1,T_2)$

**Formal Definition:**

A forward rate is the rate for a *future* accrual period implied by today's term structure.

**Discrete/Simple Forward for Period $[T_1,T_2]$:**

Define $F_s(0;T_1,T_2)$ by the no-arbitrage identity:

$$1 + F_s(0;T_1,T_2) \cdot \tau = \frac{P(0,T_1)}{P(0,T_2)}$$

where $\tau = T_2 - T_1$.

**Continuously Compounded Forward:**

$$f_c(0;T_1,T_2) = \frac{1}{\tau} \ln\left(\frac{P(0,T_1)}{P(0,T_2)}\right)$$

**Instantaneous Forward (Continuous-Time Limit):**

$$f(0,T) = -\frac{d'(T)}{d(T)} = -\frac{\partial}{\partial T} \ln d(T)$$

The forward rate is minus the derivative of log discount factor.

**Intuition:**

A forward rate answers: *"What rate for future borrowing/lending over $[T_1,T_2]$ makes you indifferent today between investing to $T_2$ directly vs investing to $T_1$ and rolling forward?"*

**Trading / Risk / Portfolio Practice:**

- Floating coupons (and FRAs/caps) are fundamentally about *forwards over accrual periods*
- Forward curves are often more "wiggly" than discount curves because forwards depend on **differences/derivatives** of discount factors

---

### 4) Par Rates (Par Coupon on a Bond; Par Swap Rate)

**Formal Definition:**

**Par yield / par coupon:**

The par yield on a bond of a given maturity is the coupon rate that makes the bond sell at its par value.

**Par (forward) swap rate:**

The fixed rate that makes the swap have zero value at time $t$. A standard identity is:

$$S_{\alpha,\beta}(t) = \frac{P(t,T_\alpha) - P(t,T_\beta)}{\sum_{i=\alpha+1}^{\beta} \tau_i \, P(t,T_i)}$$

**Intuition:**

Par rates are *annuity-weighted* summaries of the discount curve. The denominator $\sum \tau_i P(\cdot)$ is the PV of "\$1 per year paid on schedule" (an annuity). This annuity is the natural scale for fixed-leg PV01.

**Trading / Risk / Portfolio Practice:**

- Par swap rates are quoted markets; translating them to discount factors is required for pricing most swaps consistently
- Par coupons are used to build "par curves" and for relative value (par bonds, swap par rates, etc.)

---

## Math and Derivations

### 1) Discount Factors ↔ Zero Rates (Multiple Compounding Bases)

#### (a) Continuous Compounding

Start from the definition $P(0,T) = e^{-z_c(T)T}$.

Solve for $z_c(T)$:

$$\boxed{z_c(T) = -\frac{1}{T} \ln P(0,T)}$$

**Unit Check:** $\ln P$ is dimensionless; dividing by $T$ (years) gives "per year."

**Sanity Checks:**

- If $P(0,T) = 1$, then $z_c(T) = 0$
- If $P(0,T) < 1$, then $\ln P(0,T) < 0 \Rightarrow z_c(T) > 0$
- If $z_c(T) < 0$, then $P(0,T) = e^{-z_c(T)T} > 1$: negative rates imply discount factors above 1

#### (b) $m$-Times Per Year Compounding (Including Semiannual)

The semiannual case:

$$P(0,T) = d(T) = \frac{1}{(1 + \hat{s}(T)/2)^{2T}}$$

Generalize to compounding $m$ times per year:

$$P(0,T) = \left(1 + \frac{z_m(T)}{m}\right)^{-mT}$$

Solving:

$$\boxed{z_m(T) = m\left(P(0,T)^{-1/(mT)} - 1\right)}$$

**Unit Check:** $z_m(T)$ is "per year" since $m$ is "per year" and the bracket is dimensionless.

**Limiting Check:** As $m \to \infty$, $z_m(T) \to z_c(T)$ (continuous compounding is the limit of more frequent compounding).

#### (c) Simple Interest (Money-Market Style)

A discount factor for $d$ days: $1/(1 + r \cdot d/360)$.

Let $\tau = d/360$ (a year fraction). Then:

$$P(0,\tau) = \frac{1}{1 + r \cdot \tau}$$

For a maturity $T$ measured in years (simple interest over the whole horizon):

$$P(0,T) = \frac{1}{1 + z_s(T) \cdot T}$$

Solving:

$$\boxed{z_s(T) = \frac{1}{T}\left(\frac{1}{P(0,T)} - 1\right)}$$

**Sanity Check:** For small $T$, the linear-in-$T$ structure matches money-market accrual conventions.

---

### 2) Discount Factors ↔ Forward Rates

#### (a) Discrete Forward Over $[T_1,T_2]$ Under Simple Interest

Consider investing \$1 today to $T_2$. Its future value at $T_2$ is $1/P(0,T_2)$.

Alternatively, invest to $T_1$ and then reinvest from $T_1$ to $T_2$ at a forward rate $F_s$ for $\tau = T_2 - T_1$:

$$\frac{1}{P(0,T_1)} \times \left(1 + F_s(0;T_1,T_2) \cdot \tau\right) = \frac{1}{P(0,T_2)}$$

Rearrange:

$$1 + F_s(0;T_1,T_2) \cdot \tau = \frac{P(0,T_1)}{P(0,T_2)}$$

So:

$$\boxed{F_s(0;T_1,T_2) = \frac{1}{\tau}\left(\frac{P(0,T_1)}{P(0,T_2)} - 1\right)}$$

**Unit Check:** RHS is dimensionless divided by years → per year.

**Limiting Case:** As $\tau \to 0$, discrete forwards approximate instantaneous forward rates.

#### (b) Continuous Forward and Instantaneous Forward

The instantaneous forward:

$$f(0,T) = -\frac{d'(T)}{d(T)} = -\frac{\partial}{\partial T} \ln d(T)$$

Integrate from $0$ to $T$:

$$\ln d(T) - \ln d(0) = -\int_0^T f(0,u) \, du$$

Since $d(0) = P(0,0) = 1 \Rightarrow \ln d(0) = 0$:

$$\boxed{P(0,T) = d(T) = \exp\left(-\int_0^T f(0,u) \, du\right)}$$

For an average continuously compounded forward over $[T_1,T_2]$:

$$\int_{T_1}^{T_2} f(0,u) \, du = \ln\frac{P(0,T_1)}{P(0,T_2)}$$

Therefore:

$$\boxed{f_c(0;T_1,T_2) = \frac{1}{\tau} \ln\left(\frac{P(0,T_1)}{P(0,T_2)}\right)}$$

**Sanity Checks:**

- **Flat curve:** if $P(0,T) = e^{-rT}$, then $f(0,T) = r$ constant, and $f_c(0;T_1,T_2) = r$
- **Sign:** if $P(0,T_2) > P(0,T_1)$, then $\ln(P(0,T_1)/P(0,T_2)) < 0$ → forward is negative

---

### 3) Spot–Forward Relationship and Curve Slope

Under continuous compounding:

$$\boxed{f(0,T) = z_c(T) + T \cdot \frac{d}{dT} z_c(T)}$$

**Interpretation:**

- If the zero curve is **upward sloping** ($z_c'(T) \geq 0$), then forwards tend to lie **above** zeros at that maturity
- If the zero curve is **downward sloping** ($z_c'(T) \leq 0$), forwards tend to lie **below** zeros

**Limiting Case Check:**

- If $z_c(T)$ is constant (flat), then $z_c'(T) = 0$ and $f(0,T) = z_c$

---

### 4) Discount Factors ↔ Par Rates

#### (a) Par Swap Rate in Terms of Discount Factors

A standard identity for the forward swap rate:

$$\boxed{S_{\alpha,\beta}(t) = \frac{P(t,T_\alpha) - P(t,T_\beta)}{\sum_{i=\alpha+1}^{\beta} \tau_i \, P(t,T_i)}}$$

At $t=0$, for a spot-starting swap with $T_\alpha = 0$ and $P(0,0) = 1$:

$$S_{0,n}(0) = \frac{1 - P(0,T_n)}{\sum_{i=1}^{n} \tau_i \, P(0,T_i)}$$

**Annuity Interpretation:**

Denominator $A(0) = \sum \tau_i P(0,T_i)$ is PV of receiving 1 unit of fixed-rate "coupon" per year on the schedule; it is the natural scale for fixed-leg PV01.

#### (b) Bond Par Coupon as a Special Case

Consider a par bond with face 1 that pays coupon rate $c_{\text{par}}$ per year on dates $T_1, \ldots, T_n$, with accruals $\tau_i$.

Cashflows:
- Coupon at $T_i$: $c_{\text{par}} \cdot \tau_i$
- Principal at $T_n$: $1$

**Par condition (Price = 1):**

$$1 = \sum_{i=1}^n c_{\text{par}} \cdot \tau_i \cdot P(0,T_i) + 1 \cdot P(0,T_n)$$

Solve:

$$\boxed{c_{\text{par}} = \frac{1 - P(0,T_n)}{\sum_{i=1}^n \tau_i \, P(0,T_i)}}$$

This matches the spot-start swap par rate formula (bond par coupon is "swap fixed rate" when the floating leg is par by construction).

**Unit Check:** Numerator is dimensionless; denominator is in years; result is per year.

---

### 5) The "Triangle" Mapping (Conceptual Identity Map)

Once conventions are fixed:

**From discount factors $P(0,T)$ you get:**

- **Zeros $z(\cdot)$** by inverting the chosen compounding formula (e.g., $z_c(T) = -(1/T)\ln P(0,T)$)
- **Forwards $f(\cdot)$** by ratios/differences (discrete) or derivatives (instantaneous)
- **Par rates** by annuity formulas

**Conversely:**

- From a continuous forward curve $f(0,T)$, recover discount factors by integration:
  $$P(0,T) = \exp\left(-\int_0^T f(0,u) \, du\right)$$

- From a zero curve $z_c(T)$, recover discount factors directly:
  $$P(0,T) = e^{-z_c(T)T}$$

- From par swap rates across maturities, recover discount factors by **bootstrapping** (method deferred to later chapters)

---

### Unit Checks and Sanity Checks (Summary)

| Check | Description |
|-------|-------------|
| **Boundary** | $P(0,0) = 1$, so $z_c(0)$ is well-behaved as a limit |
| **Positivity** | $P(0,T) > 0$ ⇒ $z_c(T)$ finite, and $\ln P$ defined |
| **Monotonicity (conditional)** | If $P(0,T)$ is decreasing in $T$, then average discounting is positive. If $P(0,T)$ increases over some range, the implied forward is negative |
| **Forward "wiggles" warning** | Forwards depend on *differences/derivatives* of discount factors; avoid jumps in the slope of the spot curve |

---

## Measurement & Risk

### How Curve "Shape" Affects Forwards and Par Rates

- **Upward sloping zero curve → forwards above zeros**

  Under continuous compounding, the identity $f(0,T) = z_c(T) + T z_c'(T)$ implies $f(0,T) \geq z_c(T)$ iff $z_c'(T) \geq 0$

- **Downward sloping zero curve → forwards below zeros** by the same logic

- **Par rates are annuity-weighted averages** of the discounting over the coupon schedule; they are "smoother" summaries than forwards because they aggregate over many maturities

### Small-Change Intuition (Conceptual; Not Full Duration)

Changing the curve at some maturity $T^*$ changes:

- The PV of cashflows at/after $T^*$ (directly through $P(0,T^*)$ and indirectly through curve consistency)
- Forward rates around $T^*$ (because forwards use ratios/differences)
- Par rates (because they depend on a sum of discounted accruals)

**Heuristic:** Forwards are **local**, par rates are **global** (over the annuity window).

---

## Worked Examples

### Example A: Convert Discount Factors into Zero Rates

**Given discount factors:**

| $T$ (years) | $P(0,T)$ |
|---:|---:|
| 1 | 0.9700 |
| 2 | 0.9400 |
| 3 | 0.9000 |

#### (i) Simple-Interest Zero Rate $z_s(T)$

Use $z_s(T) = \dfrac{1}{T}\left(\dfrac{1}{P(0,T)} - 1\right)$

**$T = 1$:**
$$z_s(1) = \left(\frac{1}{0.9700} - 1\right) = 1.0309278 - 1 = 0.0309278 = 3.0928\%$$

**$T = 2$:**
$$z_s(2) = \frac{1}{2}\left(\frac{1}{0.9400} - 1\right) = \frac{1}{2}(1.0638298 - 1) = \frac{0.0638298}{2} = 3.1915\%$$

**$T = 3$:**
$$z_s(3) = \frac{1}{3}\left(\frac{1}{0.9000} - 1\right) = \frac{1}{3}(1.1111111 - 1) = 3.7037\%$$

#### (ii) Continuously Compounded Zero Rate $z_c(T)$

Use $z_c(T) = -\dfrac{1}{T}\ln P(0,T)$

**$T = 1$:**
$$z_c(1) = -\ln(0.9700) = 0.0304592 = 3.0459\%$$

**$T = 2$:**
$$z_c(2) = -\frac{1}{2}\ln(0.9400) = -\frac{1}{2}(-0.0618754) = 3.0938\%$$

**$T = 3$:**
$$z_c(3) = -\frac{1}{3}\ln(0.9000) = -\frac{1}{3}(-0.1053605) = 3.5120\%$$

**Observation:** Simple vs continuous differs because the "unit" of the rate differs; compounding frequency is a unit choice and conversion formulas apply.

---

### Example B: Compute a Forward Rate from Discount Factors (Negative-Rate Case)

Take a 6-month period $[T_1,T_2] = [0.5, 1.0]$, $\tau = 0.5$. Suppose:

$$P(0,0.5) = 1.0005, \qquad P(0,1.0) = 1.0020$$

These discount factors are **above 1**, consistent with **negative** average rates.

#### (i) Simple Forward Rate $F_s(0;0.5,1.0)$

Use $1 + F_s \tau = \dfrac{P(0,0.5)}{P(0,1.0)}$:

$$1 + F_s \cdot 0.5 = \frac{1.0005}{1.0020} = 0.9985030$$

So:
$$F_s = \frac{0.9985030 - 1}{0.5} = \frac{-0.0014970}{0.5} = -0.2994\% \text{ per year}$$

#### (ii) Continuously Compounded Forward Rate $f_c(0;0.5,1.0)$

$$f_c = \frac{1}{0.5}\ln\left(\frac{1.0005}{1.0020}\right) = 2\ln(0.9985030) = 2(-0.0014981) = -0.2996\% \text{ per year}$$

#### Economic Interpretation

- A **negative forward rate** over $[0.5, 1.0]$ means that *locking in funding/investing for that future period implies paying (or receiving less than principal)* over the period
- Mechanically: $P(0,1.0) > P(0,0.5)$ implies the discount curve rises with maturity over that interval; by the log/ratio formulas, this forces a negative forward

---

### Example C: Compute a Par Coupon Rate from Discount Factors

Consider a 3-year bond with annual coupons, face value 100, coupon rate $c$ per year, paying at $T_1 = 1, T_2 = 2, T_3 = 3$.

**Discount factors:**
$$P(0,1) = 0.9600, \quad P(0,2) = 0.9150, \quad P(0,3) = 0.8700$$

**Par condition (price = 100):**

$$100 = 100c \cdot P(0,1) + 100c \cdot P(0,2) + 100c \cdot P(0,3) + 100 \cdot P(0,3)$$

Divide by 100:

$$1 = c\left[P(0,1) + P(0,2) + P(0,3)\right] + P(0,3)$$

Solve:

$$c = \frac{1 - P(0,3)}{P(0,1) + P(0,2) + P(0,3)} = \frac{1 - 0.8700}{0.9600 + 0.9150 + 0.8700}$$

Compute:
- Numerator: $1 - 0.8700 = 0.1300$
- Denominator: $0.9600 + 0.9150 + 0.8700 = 2.7450$

$$\boxed{c = \frac{0.1300}{2.7450} = 0.04736 = 4.736\% \text{ per year}}$$

**Annuity View:**

The denominator is the annuity PV $A(0) = \sum_{i=1}^3 1 \cdot P(0,i)$. This matches the par swap-rate structure $S = \dfrac{P(0,0) - P(0,T_3)}{\sum \tau_i P(0,T_i)}$ with $P(0,0) = 1$.

---

### Example D: Two Interpolation Choices Create Different Forward Curves

Assume we only know discount factors at 1y and 2y:

$$P(0,1) = 0.9800, \qquad P(0,2) = 0.9400$$

We want an intermediate discount factor at $T = 1.5$ to compute forwards over $[1, 1.5]$ and $[1.5, 2]$.

#### Method 1: Linear Interpolation of Discount Factors (DF-Linear)

$$P_{\text{DF-lin}}(0,1.5) = 0.5 \cdot 0.9800 + 0.5 \cdot 0.9400 = 0.9600$$

Compute continuously compounded forwards:

$$f_c(0;1,1.5) = \frac{1}{0.5}\ln\left(\frac{0.9800}{0.9600}\right) = 2\ln(1.0208333) = 2(0.0206193) = 4.1239\%$$

$$f_c(0;1.5,2) = \frac{1}{0.5}\ln\left(\frac{0.9600}{0.9400}\right) = 2\ln(1.0212766) = 2(0.0210526) = 4.2105\%$$

**DF-linear produces:** relatively smooth forwards: **4.124%** then **4.211%**

#### Method 2: Linear Interpolation of Continuously Compounded Zero Rates (Zero-Linear)

First compute endpoint continuous zeros:

$$z_c(1) = -\ln(0.9800) = 0.0202027 = 2.0203\%$$

$$z_c(2) = -\frac{1}{2}\ln(0.9400) = 0.0309377 = 3.0938\%$$

Interpolate $z_c(1.5)$ linearly:

$$z_c(1.5) = z_c(1) + \frac{1.5 - 1}{2 - 1}\left[z_c(2) - z_c(1)\right] = 0.0202027 + 0.5(0.0107350) = 0.0255702$$

Convert back to a discount factor:

$$P_{\text{Z-lin}}(0,1.5) = \exp(-z_c(1.5) \cdot 1.5) = \exp(-0.0383553) = 0.96236$$

Now compute forwards:

$$f_c(0;1,1.5) = \frac{1}{0.5}\ln\left(\frac{0.9800}{0.96236}\right) = 2\ln(1.01833) = 3.633\%$$

$$f_c(0;1.5,2) = \frac{1}{0.5}\ln\left(\frac{0.96236}{0.9400}\right) = 2\ln(1.02379) = 4.702\%$$

**Zero-linear produces:** forwards **3.633%** then **4.702%**, a much bigger "wiggle"

#### Why This Matters

- Forward rates enter directly into pricing/hedging of floating legs and forward-starting products
- If your interpolation creates artificial "local" kinks in the spot curve, the forward curve can swing sharply because forwards depend on derivatives
- It is desirable to avoid jumps in slope when fitting spot curves; splines are designed to ensure continuity of slope and curvature

---

## Practical Notes

### Quoting Conventions That Commonly Cause Mistakes

| Issue | Description |
|-------|-------------|
| **Compounding frequency** | Part of the quote. Continuous uses $Ae^{Rn}$ and discounting uses $e^{-Rn}$ |
| **Money-market simple accrual** | Many short-rate instruments use simple interest with a day count (often actual/360). Floating payments are $L \times R \times n/360$ |
| **Annualization traps** | A "3-month rate" quoted with actual/360 is not directly comparable to an annual bond yield with actual/actual or 30/360 |

### Implementation Pitfalls

| Pitfall | Description |
|---------|-------------|
| **Rounding** | Forwards are ratios/differences of discount factors; small rounding in $P(0,T)$ can materially move short forwards |
| **Negative rates** | Allow $P(0,T) > 1$ and/or increasing segments in $P(0,T)$; formulas still work as long as $P(0,T) > 0$ |
| **Stub periods** | If accrual fraction $\tau_i$ differs from "regular" (e.g., 0.27y instead of 0.25y), it must be used consistently in forward-rate definitions, annuity, and par rate formulas |

### Verification Tests and Sanity Checks

1. **Boundary:** ensure $P(0,0) = 1$
2. **Positivity:** check $P(0,T) > 0$ for all nodes; otherwise logs and continuous rates break
3. **Monotonicity (conditional):**
   - If your market regime assumes nonnegative rates, expect $P(0,T)$ nonincreasing in $T$
   - If negative rates are possible, monotonicity can fail; still require positivity
4. **Forward consistency:** for a tenor grid $T_0 < T_1 < \dots < T_n$, verify the multiplicative chain:
   $$\frac{1}{P(0,T_n)} = \frac{1}{P(0,T_0)} \prod_{i=1}^n \left(1 + F_s(0;T_{i-1},T_i) \cdot \tau_i\right)$$
5. **No "free lunch" kinks:** if a fitted curve creates extreme oscillations in forwards, investigate interpolation/fitting choices

---

## Summary & Recall

### 10-Bullet Executive Summary

1. The discount curve $P(0,T)$ is the most primitive object; it prices any deterministic cashflow stream by PV summation
2. A zero rate $z(T)$ is an alternative representation of $P(0,T)$, but it is meaningless without compounding/day-count conventions
3. Under continuous compounding, $P(0,T) = e^{-z_c(T)T}$ and $z_c(T) = -(1/T)\ln P(0,T)$
4. Under semiannual compounding, $d(T) = 1/(1 + \hat{s}(T)/2)^{2T}$
5. Discrete (simple) forward rates satisfy $1 + F\tau = P(0,T_1)/P(0,T_2)$
6. Instantaneous forward rates are the negative derivative of log discount factors: $f(0,T) = -\partial_T \ln P(0,T)$
7. The slope of the zero curve controls forwards: $f(0,T) = z_c(T) + Tz_c'(T)$ and "forward above spot iff spot increasing"
8. Par rates are annuity-weighted: $c_{\text{par}} = (1 - P(0,T_n))/\sum \tau_i P(0,T_i)$; swap par rates share the same structure
9. Negative rates correspond to $P(0,T) > 1$ and can produce negative forwards; formulas remain valid if $P(0,T) > 0$
10. Interpolation/fitting choices can create forward-curve "wiggles" even if discount factors look smooth; slope continuity matters

---

### Cheat Sheet of Core Identities

#### Discount Factors and Zeros

| Convention | Discount Factor | Zero Rate |
|------------|-----------------|-----------|
| Continuous | $P(0,T) = e^{-z_c(T)T}$ | $z_c(T) = -(1/T)\ln P(0,T)$ |
| $m$-comp | $P(0,T) = \left(1 + \dfrac{z_m(T)}{m}\right)^{-mT}$ | $z_m(T) = m\left(P(0,T)^{-1/(mT)} - 1\right)$ |
| Simple | $P(0,T) = \dfrac{1}{1 + z_s(T)T}$ | $z_s(T) = \dfrac{1}{T}\left(\dfrac{1}{P(0,T)} - 1\right)$ |

#### Forwards

| Type | Formula |
|------|---------|
| Simple forward | $1 + F_s(0;T_1,T_2)\tau = \dfrac{P(0,T_1)}{P(0,T_2)}$, so $F_s = \dfrac{1}{\tau}\left(\dfrac{P(0,T_1)}{P(0,T_2)} - 1\right)$ |
| Continuous forward (average) | $f_c(0;T_1,T_2) = \dfrac{1}{\tau}\ln\left(\dfrac{P(0,T_1)}{P(0,T_2)}\right)$ |
| Instantaneous | $f(0,T) = -\partial_T \ln P(0,T)$ |

#### Spot–Forward Slope Link (Continuous)

$$f(0,T) = z_c(T) + T z_c'(T)$$

#### Par Rates (Annuity Form)

| Type | Formula |
|------|---------|
| Swap | $S_{\alpha,\beta}(t) = \dfrac{P(t,T_\alpha) - P(t,T_\beta)}{\sum_{i=\alpha+1}^{\beta} \tau_i P(t,T_i)}$ |
| Bond par coupon (spot start) | $c_{\text{par}} = \dfrac{1 - P(0,T_n)}{\sum_{i=1}^n \tau_i P(0,T_i)}$ |

---

## Flashcards (Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What is $P(0,T)$? | The discount factor: PV today of 1 paid at $T$ |
| 2 | Under continuous compounding, how do you get $z_c(T)$ from $P(0,T)$? | $z_c(T) = -(1/T)\ln P(0,T)$ |
| 3 | What does $P(0,T) > 1$ imply about rates (continuous convention)? | Negative average continuously compounded rate to $T$ |
| 4 | Define a simple forward rate $F_s(0;T_1,T_2)$ from discount factors | $1 + F_s\tau = P(0,T_1)/P(0,T_2)$ |
| 5 | Define a continuously compounded forward $f_c(0;T_1,T_2)$ | $f_c = (1/\tau)\ln(P(0,T_1)/P(0,T_2))$ |
| 6 | What is the instantaneous forward rate $f(0,T)$? | $-\partial_T \ln P(0,T)$ |
| 7 | Why can forwards be "wigglier" than discount factors? | Forwards use ratios/differences/derivatives of $P$, amplifying local fitting noise |
| 8 | What is Hull's interpretation of compounding frequency? | It defines the *units* of the rate; different frequencies are like different measurement units |
| 9 | In Tuckman's semiannual convention, how is $d(T)$ linked to the spot rate? | $d(T) = 1/(1 + \hat{s}(T)/2)^{2T}$ |
| 10 | State the continuous spot–forward relationship linking curve slope | $f(0,T) = z_c(T) + Tz_c'(T)$ |
| 11 | If $z_c'(T) > 0$, how do forwards compare to zeros at $T$? | Forwards exceed zeros at $T$ (continuous case) |
| 12 | What is a par yield (bond)? | The coupon rate making the bond sell at par |
| 13 | What is the "annuity" $A(0)$ used in par rate formulas? | $A(0) = \sum \tau_i P(0,T_i)$ |
| 14 | Write the par swap rate in terms of discount factors | $S = (P(0,T_\alpha) - P(0,T_\beta))/\sum \tau_i P(0,T_i)$ |
| 15 | Why is a par rate a weighted average object? | It divides a PV difference by an annuity PV (sum of discounted accruals) |
| 16 | What happens to forward rates if the fitted zero curve has slope discontinuities (kinks)? | Forwards can jump/spike because they depend on slope/derivatives |
| 17 | What is a basic boundary condition for discount factors? | $P(0,0) = 1$ |
| 18 | Can discount factors be negative? | No—positivity is required for log-based rates and to avoid arbitrage |
| 19 | How do you convert a continuously compounded rate $R_c$ to an $m$-compounded rate $R_m$? | $R_m = m(e^{R_c/m} - 1)$ |
| 20 | How do you discount a cashflow under continuous compounding? | Multiply by $e^{-Rn}$ |

---

## Mini Problem Set

### Questions

1. Given $P(0,1) = 0.975$, compute $z_c(1)$ and $z_s(1)$.

2. Given $P(0,1) = 0.99$ and $P(0,1.5) = 0.975$, compute $f_c(0;1,1.5)$.

3. Given $P(0,0.5) = 0.995$ and $P(0,1) = 0.985$, compute the simple forward rate over $[0.5, 1]$.

4. Given $P(0,1) = 0.97$, $P(0,2) = 0.94$, $P(0,3) = 0.90$, compute the 3y par coupon on an annual-pay bond with face 100.

5. Suppose $P(0,0.5) = 1.001$ and $P(0,1) = 1.000$. Compute the simple and continuous forward rates over $[0.5, 1]$ and interpret the sign.

6. Show algebraically that if you know all period simple forwards $F_s(0;T_{i-1},T_i)$, you can reconstruct $P(0,T_n)$ recursively.

7. Prove (continuous case) that if $P(0,T) = e^{-rT}$, then the instantaneous forward rate is constant and equals $r$.

8. Using $f(0,T) = z_c(T) + Tz_c'(T)$, explain what happens to forwards at maturities where the zero curve has a kink (a jump in slope).

9. Consider two interpolation methods for $z_c(T)$ between 2y and 3y. Describe qualitatively why they might produce different short forwards even if they match endpoint discount factors.

10. If $P(0,T)$ is decreasing and smooth, can forwards still be negative? Under what condition?

11. For a swap with payment dates $T_1, \ldots, T_n$, explain why the denominator $\sum \tau_i P(0,T_i)$ behaves like a "PV01 scale factor."

12. Explain why converting par swap rates to discount factors requires a bootstrapping-style procedure (outline conceptually; no full algorithm).

### Brief Solution Sketches (1–5 Only)

1. $z_c(1) = -\ln(0.975) = 0.02532 = 2.532\%$. $z_s(1) = 1/0.975 - 1 = 0.02564 = 2.564\%$.

2. $f_c = (1/0.5)\ln(0.99/0.975) = 2\ln(1.0153846) = 2(0.015266) = 0.03053 = 3.053\%$.

3. $1 + F_s \cdot 0.5 = 0.995/0.985 = 1.010152 \Rightarrow F_s = (1.010152 - 1)/0.5 = 0.020304 = 2.030\%$.

4. $c = (1 - P(0,3))/[P(0,1) + P(0,2) + P(0,3)] = (0.10)/(0.97 + 0.94 + 0.90) = 0.10/2.81 = 0.03559 = 3.559\%$. Coupon = 3.559% of face.

5. Simple: $1 + F_s \cdot 0.5 = 1.001/1.000 = 1.001 \Rightarrow F_s = 0.001/0.5 = 0.002 = 0.20\%$. Continuous: $f_c = (1/0.5)\ln(1.001) = 2(0.0009995) = 0.001999 \approx 0.20\%$. Sign positive because $P(0,0.5) > P(0,1)$.

---

## Source Map

### (A) Verified Facts

- Continuous discounting relationship defining spot rate via $d(t) = e^{-\hat{r}(t)t}$
- Tuckman semiannual spot-rate / discount-factor identity $d(t) = 1/(1 + \hat{s}(t)/2)^{2t}$
- Tuckman forward-rate/discount-function relationship in discrete and continuous limits, including $r(t) = -d'(t)/d(t)$
- Tuckman continuous spot–forward slope identity $r(t) = \hat{r}(t) + t\hat{r}'(t)$ and the "forward vs spot" monotonicity statement
- Hull's continuous compounding growth/discounting form $Ae^{Rn}$ and $e^{-Rn}$, plus conversion equations between compounding bases
- Hull's definition of par yield as the coupon rate making a bond sell at par
- Swap par (forward) rate formula in terms of discount factors and annuity
- Money-market/simple-interest discount factor form $1/(1 + r \cdot d/360)$ and floating payment proportional to $n/360$
- Tuckman's emphasis on avoiding slope jumps in fitted spot curves (motivation for spline smoothness)

### (B) Reasoned Inference (Derived from A)

- Conversions $z_c(T) = -(1/T)\ln P(0,T)$, $z_s(T) = (1/T)(1/P - 1)$, $z_m(T) = m(P^{-1/(mT)} - 1)$
- Period forward formulas from discount-factor ratios (simple and continuous) and the integral representation $P(0,T) = \exp(-\int_0^T f(0,u) \, du)$
- Par bond coupon formula $c_{\text{par}} = (1 - P(0,T_n))/\sum \tau_i P(0,T_i)$ by pricing cashflows at par
- Interpretation of annuity $A(0) = \sum \tau_i P(0,T_i)$ as PV scale factor for par rates / PV01 per unit rate
- Example D's demonstration that different interpolation choices yield materially different forwards due to derivative sensitivity

### (C) Speculation (Clearly Labeled)

- Specific desk conventions for which curve is "primary" (OIS discounting vs single-curve), and which day count/compounding is default for quoted zeros and par rates. This depends on market, currency, collateralization, and product set.

---

*Generated from the provided reference books; verify market conventions for your specific desk.*
