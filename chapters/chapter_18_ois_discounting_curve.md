# Chapter 18: OIS Discounting Curve — Building the "Risk-Free-ish" Curve

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- OIS exchanges a fixed rate for a compounded overnight rate over each accrual period
- The compounded overnight rate is defined via the product of daily accrual factors
- Longer-maturity OIS rates define par bonds and are used to bootstrap the zero curve
- OIS rates out to one year provide zero rates in a direct way
- The OIS discounting curve is linked to collateral remuneration practice—overnight rates are the closest proxy to "risk-free"
- Multiple yield curves are needed post-crisis: OIS for discounting, separate curves for projection
- Interpolation linear in $\ln P(T)$ corresponds to piecewise-flat forward rates

### (B) Reasoned Inference (Derived from A)

- The par-bond equation $1 = k \sum_i \tau_i P(0,T_i) + P(0,T_N)$ follows directly from discounted cashflow pricing
- The bootstrap formula $P(0,T_N) = \frac{1 - k_N \sum_{i=1}^{N-1} \tau_i P(0,T_i)}{1 + k_N \tau_N}$ is algebraic rearrangement of the par condition
- Zero rate conversions $z_c(T) = -\frac{1}{T}\ln P(0,T)$ and $z_a(T) = P(0,T)^{-1/T} - 1$ follow from definitions

### (C) Speculation (Clearly Labeled; Minimal)

- I'm not sure about the exact fails-charge formula—sources discuss fails conceptually but don't specify penalty mechanics
- I'm not sure about the fully rigorous statement "discount at the collateral remuneration rate"—the provided excerpts motivate the practice but don't contain the explicit theorem/derivation
- I'm not sure about specific OIS operational conventions (observation shift/lag, lockout, day count) for particular currencies/indices without the specific index definition

---

## Conventions & Notation

### Core Symbols

| Symbol | Definition |
|--------|------------|
| $t = 0$ | Valuation time |
| $T > 0$ | Maturity (in years) |
| $\tau(a,b)$ | Day-count year fraction from date $a$ to $b$ |
| $P(0,T)$ | Discount factor: time-0 price of a zero-coupon claim paying 1 at $T$ |
| $z_c(T)$ | Continuous-compounded zero rate: $P(0,T) = e^{-z_c(T) T}$ |
| $z_a(T)$ | Annual-compounded zero rate: $P(0,T) = (1 + z_a(T))^{-T}$ |
| $N$ | Notional |
| $k$ | Fixed rate on OIS (per annum, quote convention-dependent) |

### OIS Schedule Notation

| Symbol | Definition |
|--------|------------|
| $T_0, T_1, \ldots, T_N$ | Payment dates with $0 = T_0 < T_1 < \cdots < T_N = T$ |
| $\tau_i = \tau(T_{i-1}, T_i)$ | Year fraction for period $i$ |
| $r(T_n + t_j^n)$ | Overnight rate observed on day $j$ in period $[T_n, T_{n+1}]$ |
| $\Delta t_j^n$ | Day accrual fraction for day $j$ inside accrual period $n$ |
| $\bar{L}_n$ | Compounded overnight rate for accrual period $[T_n, T_{n+1}]$ |

### Defaults Used in Examples

- **Payment frequency:** Annual (unless stated otherwise)
- **Year fractions:** Exact fractions (e.g., 0.25, 0.5, 1.0) stated explicitly
- **Day count:** Toy conventions for learning; production requires currency/index-specific rules
- **Notional:** $N = 1$ unless stated otherwise

---

## Setup

This chapter is about building an **OIS discounting curve**—the mapping $T \mapsto P(0,T)$ implied by par OIS quotes.

**Key ideas to keep in mind throughout:**

1. We treat an OIS as a swap exchanging a fixed rate for a compounded overnight rate over each accrual period, with net cashflow at each payment date

2. We build a discounting curve $\{P(0,T)\}$ implied by par OIS quotes: each calibration instrument prices to zero at inception (or equivalently, the associated "par bond" prices to par)—this is the bootstrapping mindset

3. Currency/index specifics are intentionally generic. The sources stress that "risk-free" discounting is linked to overnight rates and market practice, but exact operational conventions can differ by market

---

## Core Concepts

### 1) Discount Factor Curve (Discounting Curve)

**Formal Definition:**

A discounting curve is the mapping $T \mapsto P(0,T)$, where $P(0,T)$ is the time-0 value of a claim paying 1 at $T$.

**Intuition:**

$P(0,T)$ tells you "how much is \$1 in the future worth today," under the market's discounting convention for the relevant collateral/credit setup.

**Trading / Risk / Portfolio Practice:**

- Pricing any instrument with deterministic cashflows (bonds, fixed legs, known fees) reduces to summing cashflows times discount factors
- Risk reports often bucket exposures to curve nodes (DV01/PV01 concepts appear later; previewed in Section 3)
- Curve construction and risk measurement are linked in the sources' curve-construction discussion

---

### 2) Zero Rate Curve

**Formal Definition:**

A zero rate is a rate representation of $P(0,T)$. For continuous compounding:

$$z_c(T) = -\frac{1}{T}\ln P(0,T)$$

For annual compounding:

$$z_a(T) = P(0,T)^{-1/T} - 1$$

**Intuition:**

Zero rates are a way to plot the curve in "rate space" rather than "price space."

**Trading / Risk / Portfolio Practice:**

- Traders quote and compare curves in rate terms (levels, slopes, humps)
- Quants implement discounting either directly with $P(0,T)$ or via $z(T)$ + an interpolation rule

---

### 3) Overnight Indexed Swap (OIS)

**Formal Definition (Mechanics-Level):**

An OIS exchanges:
- A fixed rate $k$, and
- A floating payment based on an overnight index over the accrual period, commonly a geometric average / compounded overnight rate (but sometimes arithmetic averaging is used)

**Intuition:**

- The floating leg mimics "rolling" overnight borrowing/lending across the period
- Because it is tied to overnight funding, it is often treated as the closest traded proxy to "risk-free-ish" funding in modern collateralized markets (within the assumptions of the product and CSA)

**Trading / Risk / Portfolio Practice:**

- OIS quotes across maturities are used to infer a discounting curve (the "OIS curve")
- In multi-curve setups, OIS is typically the discounting curve, while a separate curve projects the floating index (next chapter)

---

### 4) Par OIS Rate and "Par Instrument"

**Formal Definition:**

The par OIS rate $k_{\text{par}}$ is the fixed rate such that the swap has zero value at inception:

$$PV_{\text{fixed}}(k_{\text{par}}) - PV_{\text{float}} = 0$$

**Intuition:**

Par quotes are "calibration targets": the market tells you the rate that makes the instrument fair today.

**Trading / Risk / Portfolio Practice:**

Curve bootstrapping uses par instruments because:
- The pricing equation is a direct constraint on discount factors
- It is easy to check calibration quality by repricing to par

---

### 5) "Risk-Free-ish" (What It Does and Does Not Mean)

**Formal Definition (As Used Here):**

"Risk-free-ish" refers to the market-implied curve from OIS instruments, often used as the discounting curve in collateralized derivative pricing.

**Intuition:**

It is **not** a universal risk-free rate. It is:
- **Instrument-defined** (OIS mechanics)
- **Market-defined** (liquidity/quotes)
- **CSA/collateral-defined** (collateral remuneration assumptions)

**Trading / Risk / Portfolio Practice:**

Risk managers must remember:
- Different collateral terms → different discounting curve
- Cross-currency collateralization can induce bases and multiple curves (beyond this chapter)

---

## Math and Derivations

### 1) OIS Floating Leg: Compounded Overnight Rate

Consider an accrual period $[T_n, T_{n+1}]$ with year fraction $\tau_n = \tau(T_n, T_{n+1})$. The sources define the effective compounded overnight rate $\bar{L}_n$ via the product of daily accrual factors:

$$\boxed{\bar{L}_n = \frac{\prod_{j=1}^{\ell_n}\left(1 + r(T_n + t_j^n)\Delta t_j^n\right) - 1}{\tau_n}}$$

where $r(T_n + t_j^n)$ is the overnight rate for day $j$ and $\Delta t_j^n$ its day-count fraction.

**Unit check:**
- $r(\cdot)$ has units $1/\text{year}$
- $\Delta t_j^n$ has units $\text{year}$
- $r \cdot \Delta t$ is dimensionless, so the product is dimensionless
- $\bar{L}_n$ has units $1/\text{year}$ because we divide a dimensionless return by $\tau_n$ (years)

---

### 2) OIS Net Payment Per Period

For notional $N$ and fixed rate $k$, the OIS payoff over $[T_n, T_{n+1}]$ is the netted amount paid at $T_{n+1}$:

$$\boxed{\text{Payoff at } T_{n+1} = N \tau_n (\bar{L}_n - k)}$$

**Unit check:**
- $\tau_n(\bar{L}_n - k)$ is dimensionless
- Multiply by notional $N$ → currency units ✓

---

### 3) Par Condition and "Par Bond" Equivalence

A key practical simplification is to interpret a par swap as an exchange of a par floating-rate bond for a fixed-rate bond. The sources explain this "bond equivalence" idea for OIS.

**Par Fixed-Bond Condition (annualized fixed coupon $k$):**

For a bond with face value 1, coupon rate $k$, and coupon payments $k\tau_i$ at $T_i$, price is:

$$\text{Price} = \sum_{i=1}^{N} k\tau_i P(0,T_i) + 1 \cdot P(0,T_N)$$

At par, $\text{Price} = 1$, so:

$$\boxed{1 = k\sum_{i=1}^{N} \tau_i P(0,T_i) + P(0,T_N)}$$

**Define the fixed-leg annuity:**

$$\boxed{A(0; T_1, \ldots, T_N) \equiv \sum_{i=1}^{N} \tau_i P(0,T_i)}$$

This annuity factor appears explicitly in swap-rate expressions. Using the par bond equation, the par fixed rate can be written:

$$\boxed{k_{\text{par}} = \frac{1 - P(0,T_N)}{\sum_{i=1}^{N} \tau_i P(0,T_i)} = \frac{1 - P(0,T_N)}{A(0)}}$$

**Unit check:**
- Numerator $1 - P(0,T_N)$ is dimensionless
- Denominator $A(0)$ has units of years
- So $k_{\text{par}}$ has units $1/\text{year}$ ✓

**Sanity check:**
- If $P(0,T_N)$ is close to 1 (very short maturity, low rate), $k_{\text{par}} \approx 0$
- If $P(0,T_N)$ decreases, $k_{\text{par}}$ increases, all else equal ✓

---

### 4) Bootstrapping Discount Factors from Par OIS Rates

**Goal:** Infer $P(0,T)$ at a set of maturities from par OIS quotes.

**Bootstrap idea:** Solve sequentially for the newest unknown discount factor so that the calibration instrument prices to par (or PV to zero).

Assume you have a par OIS quote $k_N$ for maturity $T_N$ with payment dates $T_1, \ldots, T_N$. Using the par bond equation:

$$1 = k_N \sum_{i=1}^{N} \tau_i P(0,T_i) + P(0,T_N)$$

Suppose $P(0,T_i)$ for $i = 1, \ldots, N-1$ are already known. Then isolate the unknown $P(0,T_N)$:

$$1 = k_N \left(\sum_{i=1}^{N-1} \tau_i P(0,T_i) + \tau_N P(0,T_N)\right) + P(0,T_N)$$

$$= k_N \sum_{i=1}^{N-1} \tau_i P(0,T_i) + (1 + k_N \tau_N) P(0,T_N)$$

Solving:

$$\boxed{P(0,T_N) = \frac{1 - k_N \sum_{i=1}^{N-1} \tau_i P(0,T_i)}{1 + k_N \tau_N}}$$

**Unit check:**
- $k_N \tau_i$ is dimensionless
- Numerator and denominator are dimensionless → $P(0,T_N)$ dimensionless ✓

**Repricing check (must pass in implementation):**
After solving $P(0,T_N)$, recompute the par-bond price and verify it equals 1 (within tolerance).

---

### 5) Short Maturities: OIS Quotes "Directly" Define Zero Rates

The derivatives source states that OIS rates out to one year provide zero rates in a direct way; for longer maturities, they define par bonds and are used to bootstrap the zero curve.

**Practical interpretation:**
- For a single-payment maturity, a quoted OIS rate can be converted to a discount factor using its quote compounding convention, then mapped to a continuous zero rate via $z_c = -\ln P / T$
- For multi-payment maturities, use the par-bond equation and bootstrap

---

### 6) Converting Discount Factors to Zero Rates

Given $P(0,T)$:

**Continuous zero:**

$$\boxed{z_c(T) = -\frac{1}{T}\ln P(0,T)}$$

**Annual-compounded zero:**

$$\boxed{z_a(T) = P(0,T)^{-1/T} - 1}$$

Equivalently, since $P(0,T) = e^{-z_c(T) T}$:

$$1 + z_a(T) = e^{z_c(T)} \Rightarrow z_a(T) = e^{z_c(T)} - 1$$

---

### 7) Interpolation Between Curve Nodes

Market instruments exist only at discrete maturities, so you need an interpolation rule to create a continuous curve.

A common simple rule is **linear interpolation in $\ln P(T)$**, i.e., $\ln P(T)$ is linear between node dates. The interest-rate-modeling source shows this as:

$$P(T) = P(T_i) \, e^{-f(T_i)(T - T_i)}, \quad T \in [T_i, T_{i+1})$$

which corresponds to a **piecewise-flat forward rate** $f(T_i)$ on each interval.

**Practical consequence:**
- Discount factors are smooth in log-space
- Forwards are piecewise constant ("staircase" forward curve), which can cause kinks unless you smooth further

---

## Measurement & Risk (Chapter 18 Preview)

### Why OIS Is Used for Discounting in Modern Collateralized Pricing

The sources motivate that the overnight rate is viewed as the closest proxy to "risk-free," while interbank term rates like LIBOR embed credit/liquidity components and may not be appropriate for discounting collateralized trades.

They describe the market practice shift to multiple yield curves, where the curve used for discounting is the risk-free (OIS) zero curve obtained from the OIS market.

**Collateral/CSA logic (conceptual):**
The interest-rate-modeling source notes that interdealer trades are commonly collateralized and that the rate paid on collateral is generally the overnight rate.

**I'm not sure (formal theorem):**
The fully rigorous statement "discount at the collateral remuneration rate" is a known result in modern XVA literature, but the provided excerpts only motivate the practice and the link to collateral remuneration.

---

### What Curve Object Is Being Built

We build (at minimum) one of these equivalent representations:
- **Discount factors:** $T \mapsto P(0,T)$
- **Zero rates:** $T \mapsto z(T)$, commonly $z_c(T)$ or $z_a(T)$

---

### Risk Reporting Preview: OIS Curve PV01 (Concept)

PV01 is the change in PV for a 1 bp change in a rate/curve.

For a fixed-leg annuity, a classic sensitivity notion is that PV01 relates to the sum of discounted accrual factors. Tuckman notes PV01 equals the sum of discount factors used to discount the fixed cashflows (scaled by accrual conventions).

In practice for an OIS discount curve, "OIS PV01" might be computed via:
- **Bump-and-rebuild:** bump one OIS quote by 1 bp, rebuild the OIS curve, reprice the portfolio, and take the difference

More advanced Jacobian methods exist but are beyond Chapter 18.

---

## Worked Examples

> **Reminder:** These are toy examples for learning curve mechanics. They are not meant to encode any specific currency's operational conventions unless explicitly stated.

---

### Example A — OIS as Par Bond Equivalence (Annuity Equation)

**Goal:** Given discount factors, compute the fixed-leg annuity and solve for the par OIS fixed rate $k_{\text{par}}$. This shows the "par bond" structure:

$$1 = k_{\text{par}} A(0) + P(0,T_N)$$

**Conventions (toy):**
- Valuation: $t = 0$
- Notional: $N = 1$
- Payment frequency: annual
- Day count: assume $\tau_1 = \tau_2 = 1.0$
- Settlement lag/stubs: none

**Inputs:**
- Maturity $T_N = 2$ years, payment dates $T_1 = 1$, $T_2 = 2$
- Discount factors: $P(0,1) = 0.9800$, $P(0,2) = 0.9550$

**Step 1: Fixed-leg annuity**

$$A(0) = \sum_{i=1}^{2} \tau_i P(0,T_i) = 1 \cdot 0.9800 + 1 \cdot 0.9550 = 1.9350$$

**Step 2: Solve for par rate**

$$k_{\text{par}} = \frac{1 - P(0,T_N)}{A(0)} = \frac{1 - 0.9550}{1.9350}$$

Compute numerator: $1 - 0.9550 = 0.0450$

Divide: $k_{\text{par}} = 0.0450 / 1.9350 = 0.0232558$ per year

**Final output:**

$$\boxed{k_{\text{par}} \approx 2.3256\% \text{ per annum (toy annual-pay par rate)}}$$

**Unit check:** Numerator dimensionless; annuity in years → rate in $1/\text{year}$ ✓

---

### Example B — Bootstrap First OIS Node (Short Maturity)

**Goal:** Use a short-maturity OIS quote to infer the first discount factor $P(0,T)$.

**Conventions (toy):**
- Maturity: $T = 1/12$ year (≈ 1 month)
- Single payment at maturity
- Quote compounding: monthly
- Day count approximation: $T = 1/12$ exactly

**Input:**
- 1-month OIS rate: $R = 1.80\%$ per annum, monthly-compounded

**Step 1: Convert quoted rate to discount factor**

Monthly compounding implies the 1-month growth factor is $1 + R/12$. Therefore:

$$P(0,T) = \frac{1}{1 + R/12}$$

Plug in $R = 0.018$:

$$P\left(0, \frac{1}{12}\right) = \frac{1}{1 + 0.018/12} = \frac{1}{1 + 0.0015} = \frac{1}{1.0015}$$

Compute: $1/1.0015 \approx 0.9985022466$

**Step 2: (Optional) Convert to continuous zero rate**

$$z_c(T) = -\frac{1}{T}\ln P(0,T) = -12 \ln(0.9985022466)$$

Equivalently:

$$z_c(T) = 12 \ln(1 + R/12) = 12 \ln(1.0015) \approx 12(0.001498876) = 0.0179865$$

**Final outputs:**

$$\boxed{P(0, 1/12) \approx 0.998502}$$
$$\boxed{z_c(1/12) \approx 1.7987\% \text{ (continuous)}}$$

**Unit check:** $R$ is per year, $R/12$ is per month; $P$ dimensionless ✓

---

### Example C — Sequential Bootstrap with 5 Maturities

**Goal:** Given par OIS rates for maturities 1Y–5Y, bootstrap discount factors $P(0,1), \ldots, P(0,5)$ step-by-step and do repricing checks.

**Conventions (toy):**
- Payments: annual
- Year fractions: $\tau_i = 1$ for each year
- Notional: $N = 1$
- Interpretation: par OIS rate for maturity $N$ is treated as coupon $k_N$ on a par bond

**Inputs (toy par OIS rates):**

| Maturity | Par Rate |
|----------|----------|
| 1Y | $k_1 = 2.50\%$ |
| 2Y | $k_2 = 2.70\%$ |
| 3Y | $k_3 = 2.90\%$ |
| 4Y | $k_4 = 3.05\%$ |
| 5Y | $k_5 = 3.20\%$ |

**Step 1: Bootstrap $P(0,1)$**

For $N = 1$:

$$1 = k_1 P(0,1) + P(0,1) = (1 + k_1) P(0,1)$$

So:

$$P(0,1) = \frac{1}{1 + k_1} = \frac{1}{1.025} = 0.9756097561$$

**Repricing check (1Y par bond):**

$$k_1 P(0,1) + P(0,1) = 0.025(0.9756097561) + 0.9756097561 = 1.000000 \checkmark$$

**Step 2: Bootstrap $P(0,2)$**

$$P(0,2) = \frac{1 - k_2 P(0,1)}{1 + k_2}$$

Compute $k_2 P(0,1) = 0.027 \times 0.9756097561 = 0.0263414634$

$$P(0,2) = \frac{1 - 0.0263414634}{1.027} = \frac{0.9736585366}{1.027} = 0.9480608925$$

**Repricing check (2Y):**

$$k_2(P_1 + P_2) + P_2 = 0.027(0.9756097561 + 0.9480608925) + 0.9480608925 \approx 1.000000 \checkmark$$

**Step 3: Bootstrap $P(0,3)$**

$$P(0,3) = \frac{1 - k_3(P_1 + P_2)}{1 + k_3}$$

$P_1 + P_2 = 1.9236706486$

$k_3(P_1 + P_2) = 0.029 \times 1.9236706486 = 0.0557864488$

$$P(0,3) = \frac{1 - 0.0557864488}{1.029} = \frac{0.9442135512}{1.029} = 0.9176030624$$

**Step 4: Bootstrap $P(0,4)$**

$$P(0,4) = \frac{1 - k_4(P_1 + P_2 + P_3)}{1 + k_4}$$

$P_1 + P_2 + P_3 = 2.8412737109$

$k_4 \times \text{sum} = 0.0305 \times 2.8412737109 = 0.0866588482$

$$P(0,4) = \frac{0.9133411518}{1.0305} = 0.8863087354$$

**Step 5: Bootstrap $P(0,5)$**

$$P(0,5) = \frac{1 - k_5(P_1 + P_2 + P_3 + P_4)}{1 + k_5}$$

$P_1 + \cdots + P_4 = 3.7275824463$

$0.032 \times 3.7275824463 = 0.1192826383$

$$P(0,5) = \frac{0.8807173617}{1.032} = 0.8534082962$$

**Final bootstrapped discount factors:**

| $T$ (years) | 1 | 2 | 3 | 4 | 5 |
|-------------|---|---|---|---|---|
| $P(0,T)$ | 0.9756098 | 0.9480609 | 0.9176031 | 0.8863087 | 0.8534083 |

---

### Example D — Convert Bootstrapped DFs to Zero Rates

**Goal:** Using $P(0,T)$ from Example C, compute zero rates:
- $z_c(T) = -\ln(P(0,T))/T$
- $z_a(T) = e^{z_c(T)} - 1$

**Inputs:** Discount factors from Example C

**Calculations:**

**$T = 1$:**
- $z_c(1) = -\ln(0.9756098) = \ln(1.025) = 0.0246926 \Rightarrow 2.4693\%$
- $z_a(1) = e^{0.0246926} - 1 \approx 0.0250 \Rightarrow 2.5000\%$

**$T = 2$:**
- $-\ln P(0,2) \approx 0.05333647 \Rightarrow z_c(2) = 0.05333647/2 = 0.02666823 \Rightarrow 2.6668\%$
- $z_a(2) = e^{0.02666823} - 1 \approx 0.0270269 \Rightarrow 2.7027\%$

**$T = 3$:**
- $-\ln P(0,3) \approx 0.08599032 \Rightarrow z_c(3) = 0.02866344 \Rightarrow 2.8663\%$
- $z_a(3) = e^{0.02866344} - 1 \approx 0.0290778 \Rightarrow 2.9078\%$

**$T = 4$:**
- $-\ln P(0,4) \approx 0.12068953 \Rightarrow z_c(4) = 0.03017238 \Rightarrow 3.0172\%$
- $z_a(4) = e^{0.03017238} - 1 \approx 0.0306310 \Rightarrow 3.0631\%$

**$T = 5$:**
- $-\ln P(0,5) \approx 0.15851529 \Rightarrow z_c(5) = 0.03170306 \Rightarrow 3.1703\%$
- $z_a(5) = e^{0.03170306} - 1 \approx 0.032230 \Rightarrow 3.2230\%$

**Final outputs (rounded):**

| $T$ | $z_c(T)$ (%) | $z_a(T)$ (%) |
|-----|--------------|--------------|
| 1 | 2.4693 | 2.5000 |
| 2 | 2.6668 | 2.7027 |
| 3 | 2.8663 | 2.9078 |
| 4 | 3.0172 | 3.0631 |
| 5 | 3.1703 | 3.2230 |

**Interpretation:** Annual-compounded zeros $z_a$ are slightly higher than continuous zeros $z_c$ at the same $T$, as expected.

---

### Example E — Interpolation Sensitivity (Log DF vs Zero-Rate Linear)

**Goal:** Interpolate between OIS nodes and compare:
1. Linear in $\ln P$ (piecewise flat forwards)
2. Linear in continuous zero rate $z_c(T)$

**Conventions (toy):**
- Known nodes: $T_1 = 2$, $T_2 = 5$
- Want $T = 3$
- Use continuous-compounded representation

**Inputs:**
- $P(0,2) = 0.9480609$, so $\ln P(0,2) \approx -0.05333647$
- $P(0,5) = 0.8534083$, so $\ln P(0,5) \approx -0.15851529$
- $z_c(2) \approx 2.6668\%$, $z_c(5) \approx 3.1703\%$ (from Example D)

**Method 1: Linear in $\ln P$**

Let $\alpha = \frac{T - T_1}{T_2 - T_1} = \frac{3 - 2}{5 - 2} = \frac{1}{3}$

$$\ln P(0,3) = \ln P(0,2) + \alpha(\ln P(0,5) - \ln P(0,2))$$

Increment: $\ln P(0,5) - \ln P(0,2) = (-0.15851529) - (-0.05333647) = -0.10517883$

$$\ln P(0,3) = -0.05333647 + \frac{1}{3}(-0.10517883) = -0.05333647 - 0.03505961 = -0.08839608$$

Thus: $P_{\log}(0,3) = e^{-0.08839608} \approx 0.915397$

**Implied continuous forward rates:**

Over $[2,3]$:
$$f_{2,3} = \frac{\ln P(0,2) - \ln P(0,3)}{1} = \frac{-0.05333647 - (-0.08839608)}{1} = 0.03505961 \Rightarrow 3.5060\%$$

Over $[3,5]$:
$$f_{3,5} = \frac{-0.08839608 - (-0.15851529)}{2} = 0.03505961 \Rightarrow 3.5060\%$$

Forwards are **flat** within the whole $[2,5]$ segment (consistent with $\ln P$ linear → piecewise constant forward).

**Method 2: Linear in continuous zero rate $z_c(T)$**

$$z_c(3) = z_c(2) + \alpha(z_c(5) - z_c(2))$$

$$z_c(3) = 0.02666823 + \frac{1}{3}(0.03170306 - 0.02666823) = 0.02666823 + 0.00167828 = 0.02834651$$

$$P_z(0,3) = e^{-z_c(3) \cdot 3} = e^{-0.08503953} \approx 0.918475$$

**Implied forward rates:**

Over $[2,3]$: $f_{2,3} = \frac{-0.05333647 - (-0.08503953)}{1} = 0.03170306 \Rightarrow 3.1703\%$

Over $[3,5]$: $f_{3,5} = \frac{-0.08503953 - (-0.15851529)}{2} = 0.03673788 \Rightarrow 3.6738\%$

**Comparison:**
- Log-DF linear: flat forward between nodes → simpler, but staircase at node boundaries
- Zero-rate linear: non-flat forwards within segment; can create "bends"

**Final outputs:**

$$\boxed{P_{\log}(0,3) \approx 0.915397, \quad f_{2,3} = f_{3,5} \approx 3.5060\%}$$
$$\boxed{P_z(0,3) \approx 0.918475, \quad f_{2,3} \approx 3.1703\%, \; f_{3,5} \approx 3.6738\%}$$

---

### Example F — Quote Bump Locality (+1 bp) and Curve Rebuild

**Goal:** Bump a single OIS par rate by +1 bp, rebuild the curve, and show what discount factors move.

**Conventions (toy):**
- Same as Example C (annual pay, $\tau = 1$)
- Bump: $k_3$ from $2.90\%$ to $2.91\%$
- Keep $k_1, k_2, k_4, k_5$ unchanged

**Baseline (from Example C):**

| $T$ | 1 | 2 | 3 | 4 | 5 |
|-----|---|---|---|---|---|
| $k_T$ | 2.50% | 2.70% | 2.90% | 3.05% | 3.20% |
| $P(0,T)$ | 0.9756098 | 0.9480609 | 0.9176031 | 0.8863087 | 0.8534083 |

**Bumped curve:**

New $k_3' = 2.91\% = 0.0291$

**Step 1:** $P(0,1)$ and $P(0,2)$ unchanged (bootstrap uses only $k_1$ and $k_2$)

$$P'(0,1) = 0.9756098, \quad P'(0,2) = 0.9480609$$

**Step 2: Recompute $P'(0,3)$**

$$P'(0,3) = \frac{1 - k_3'(P_1 + P_2)}{1 + k_3'}$$

$k_3'(P_1 + P_2) = 0.0291 \times 1.9236706486 = 0.0559788159$

$$P'(0,3) = \frac{1 - 0.0559788159}{1.0291} = \frac{0.9440211841}{1.0291} = 0.9173270$$

**Step 3: Recompute $P'(0,4)$**

$$P'(0,4) = \frac{1 - k_4(P_1 + P_2 + P'(0,3))}{1 + k_4}$$

Sum to 3: $P_1 + P_2 + P'(0,3) = 2.8409976179$

$0.0305 \times 2.8409976179 = 0.08665042735$

$$P'(0,4) = \frac{0.91334957265}{1.0305} = 0.8863169$$

**Step 4: Recompute $P'(0,5)$**

Sum to 4: $P_1 + P_2 + P'(0,3) + P'(0,4) = 3.7273145249$

$0.032 \times 3.7273145249 = 0.11927406480$

$$P'(0,5) = \frac{0.88072593520}{1.032} = 0.8534166$$

**Before/after table:**

| $T$ | 1 | 2 | 3 | 4 | 5 |
|-----|---|---|---|---|---|
| $P(0,T)$ base | 0.9756098 | 0.9480609 | 0.9176031 | 0.8863087 | 0.8534083 |
| $P(0,T)$ bumped | 0.9756098 | 0.9480609 | 0.9173270 | 0.8863169 | 0.8534166 |
| $\Delta P$ | 0 | 0 | −0.0002761 | +0.0000082 | +0.0000083 |

**Key lesson (locality preview):**
- The bump did not affect earlier nodes (1Y, 2Y)
- From the bumped maturity onward, later discount factors can change in non-obvious ways because later par conditions must still hold

---

### Example G — Discounting Impact on PV (OIS Curve vs Non-OIS Curve)

**Goal:** Price a fixed cashflow stream using:
1. OIS discount curve (Example C)
2. A toy "LIBOR" discount curve (OIS zeros + 50 bp spread)

**Conventions (toy):**
- Cashflows annual at $T = 1, 2, 3, 4, 5$
- Notional: 100
- Fixed coupon: 3% annually → coupon = 3 each year
- Principal repaid at $T = 5$: 100
- Cashflows: 3 at years 1–4, and 103 at year 5
- "LIBOR" discount factors: $P_L(0,T) = P_{\text{OIS}}(0,T) \cdot e^{-0.005T}$

**Step 1: PV with OIS discounting**

$$PV_{\text{OIS}} = 3(P_1 + P_2 + P_3 + P_4) + 103 P_5$$

Sum $P_1 + \cdots + P_4 = 3.7275824463$

Coupon PV: $3 \times 3.7275824463 = 11.1827473390$

Final cashflow PV: $103 \times 0.8534082962 = 87.9010545086$

Total: $PV_{\text{OIS}} = 11.1827473390 + 87.9010545086 = 99.0838018476$

**Step 2: Build toy LIBOR discount factors**

Spread factors $e^{-0.005T}$:

| $T$ | $e^{-0.005T}$ | $P_L(0,T)$ |
|-----|---------------|------------|
| 1 | 0.9950125 | 0.9707439 |
| 2 | 0.9900498 | 0.9386275 |
| 3 | 0.9851119 | 0.9039417 |
| 4 | 0.9801987 | 0.8687586 |
| 5 | 0.9753099 | 0.8323376 |

**Step 3: PV with LIBOR discounting**

Sum first four: $0.9707439 + 0.9386275 + 0.9039417 + 0.8687586 = 3.6820718$

Coupon PV: $3 \times 3.6820718 = 11.0462154$

Final PV: $103 \times 0.8323376 = 85.7307697$

Total: $PV_L = 11.0462154 + 85.7307697 = 96.7769851$

**PV difference:**

$$\Delta PV = PV_{\text{OIS}} - PV_L = 99.0838018 - 96.7769851 = 2.3068167$$

**Final output:**

$$\boxed{PV_{\text{OIS}} \approx 99.0838, \quad PV_L \approx 96.7770, \quad \Delta PV \approx 2.3068}$$

**Interpretation:** Discounting at a lower-rate OIS curve produces a higher PV for fixed cashflows. This difference is part of what traders refer to as discounting-basis effects.

---

### Example H — Collateral Link Toy Illustration (Intuition Only)

**Goal:** Show numerically the intuition: if collateral remunerates at the overnight/OIS rate, discounting at OIS is consistent.

**Important note:** The sources motivate OIS discounting through collateral remuneration practice. I'm not sure the provided excerpts contain the full formal derivation; this example is intuition only.

**Conventions (toy):**
- Single payoff: receive 100 at $T = 1$
- Use OIS DF from Example C: $P(0,1) = 0.9756098$
- Assume collateral earns the overnight/OIS rate consistent with this DF

**Step 1: OIS-discounted PV**

$$PV_{\text{OIS}} = 100 \times P(0,1) = 100 \times 0.9756098 = 97.5609756$$

**Step 2: "Collateral grows at OIS" check**

From Example C, $P(0,1) = 1/(1 + 0.025)$ corresponds to a 1-year growth factor $1/P(0,1) = 1.025$.

If you hold 97.5609756 and earn 2.5% over a year:

$$97.5609756 \times 1.025 = 100.000000$$

**Final output:**

$$\boxed{PV_{\text{OIS}} = 97.5610 \Rightarrow \text{grows to } 100 \text{ at the OIS rate}}$$

**Interpretation:** If collateral posted/received is remunerated at the overnight index, the natural discounting rate for collateralized cashflows is tied to that overnight index.

---

### Example I — Sanity Constraints: Positivity, Monotonicity, and Negative Rates

**Goal:** Demonstrate:
1. $P(0,T) > 0$ always
2. Discount factors need not be decreasing if rates are negative

**Conventions (toy):**
- Assume constant continuous zero rate $z_c(T) \equiv -0.50\%$ (negative)
- Then $P(0,T) = e^{-z_c T} = e^{0.005T}$

**Compute discount factors:**

At $T = 0.5$:
$$P(0, 0.5) = e^{0.005 \times 0.5} = e^{0.0025} \approx 1.0025031$$

At $T = 1$:
$$P(0, 1) = e^{0.005} \approx 1.0050125$$

**Outputs:**

$$\boxed{P(0, 0.5) \approx 1.002503 > 0, \quad P(0, 1) \approx 1.005013 > 0}$$

**Sanity discussion:**

- **Positivity:** $e^{(\cdot)} > 0$ ensures $P(0,T)$ is always positive under the exponential discount representation
- **Monotonicity:**
  - If zero/forward rates are positive, $P(0,T)$ tends to decrease with $T$
  - If rates are negative over an interval, the exponent becomes positive and $P(0,T)$ can increase with $T$
  - So "discount factors must be decreasing" is not universally true; it depends on rate signs

---

## Practical Notes

### Common Convention Gotchas

**Overnight leg definition: compounded vs averaged**

The OIS floating leg is typically based on a compounded overnight rate (geometric accumulation), as in the product formula for $\bar{L}_n$.

The sources note that some OIS-like structures use an arithmetic average instead of compounding.

I'm not sure about the exact market rule set (observation shift/lag, lockout, etc.) for your currency/index based only on the provided excerpts; you would need the specific index definition (e.g., SONIA/€STR/SOFR conventions) and the CSA/product confirmation.

**Day count choices and accrual factors**

Day count conventions matter: the sources emphasize using the accrual fraction $a_k$ in formulas, e.g.:

$$1 + a_k F_k = \frac{P(0, t_k)}{P(0, t_{k+1})}$$

I'm not sure which day count is standard for your OIS index without specifying currency/index.

**Payment frequency**

One source description explains longer OIS being divided into subperiods (e.g., three-month) with exchanges at each subperiod end; this is common in some markets but not universal.

Always match the payment frequency to the market quote conventions for your index.

**Stubs and end-of-month rules**

I'm not sure: the excerpts provided do not specify OIS stub and EOM conventions. You need the market standard schedule rules (holiday calendar, business day adjustment, EOM rule) for your index.

---

### Implementation Pitfalls

**Date schedule generation**
- Ensure consistent business-day adjustments across fixed and floating legs
- Validate that accrual periods used for $\tau_i$ align with product definition

**Overnight compounding calculation**

Implement the daily compounding product carefully:

$$\prod_j (1 + r_j \Delta t_j)$$

For toy/quick calculations, you may approximate with constant rates, but production pricing must follow the exact published method for the index.

**Consistent use of accrual factors**
- Keep a consistent convention: $k$ must multiply the same $\tau_i$ used in discounting and schedule generation
- Errors in $\tau_i$ are a top source of curve mis-calibration

---

### Verification Tests

**Repricing test (calibration)**
- Every OIS instrument used to build the curve must reprice to par ($PV \approx 0$)

**Discount factor sanity**
- $P(0,T) > 0$ for all $T$
- Examine implied forward rates: avoid unrealistic spikes (especially after bumps)

**Bump stability**
- A 1 bp bump in a single quote should not cause large oscillations in neighboring forwards (unless the interpolation/bootstrapping design is unstable)

---

## Summary & Recall

### 10-Bullet Executive Summary

1. The **OIS discounting curve** is the curve of discount factors $P(0,T)$ implied by OIS market quotes

2. An **OIS** exchanges a fixed rate for a compounded overnight rate over each accrual period, with net payoff $N\tau(\bar{L} - k)$

3. **OIS discounting** became standard in modern practice because overnight rates are treated as the closest proxy to "risk-free-ish," and collateral remuneration is commonly tied to overnight benchmarks

4. **"Risk-free-ish" is not universal**: it is market- and collateral-convention dependent

5. **OIS par rates are par instruments**: they satisfy $PV_{\text{fixed}} = PV_{\text{float}}$

6. Longer-maturity OIS rates can be treated like **par bond coupons**, enabling bootstrap of discount factors from the par condition

7. **Bootstrapping** solves sequentially for $P(0,T_N)$ using:
   $$P(0,T_N) = \frac{1 - k_N \sum_{i=1}^{N-1} \tau_i P(0,T_i)}{1 + k_N \tau_N}$$

8. **Discount factors map to zero rates** via $z_c(T) = -\ln P(0,T)/T$; compounding choice changes numeric rates but not $P(0,T)$

9. **Interpolation is unavoidable**; linear in $\ln P$ implies piecewise-constant forwards

10. **PV01 risk** to the OIS curve can be introduced via bump-and-rebuild; detailed hedging is deferred to later chapters

---

### Cheat Sheet of Formulas

**OIS period compounded overnight rate:**
$$\bar{L}_n = \frac{\prod_{j=1}^{\ell_n}(1 + r(T_n + t_j^n)\Delta t_j^n) - 1}{\tau_n}$$

**Net payment at $T_{n+1}$:**
$$\text{Payoff} = N\tau_n(\bar{L}_n - k)$$

**Par-bond / par-swap condition:**
$$1 = k\sum_{i=1}^{N} \tau_i P(0,T_i) + P(0,T_N)$$

**Annuity factor:**
$$A(0) = \sum_{i=1}^{N} \tau_i P(0,T_i)$$

**Par rate:**
$$k_{\text{par}} = \frac{1 - P(0,T_N)}{A(0)}$$

**Bootstrap step:**
$$P(0,T_N) = \frac{1 - k_N \sum_{i=1}^{N-1} \tau_i P(0,T_i)}{1 + k_N \tau_N}$$

**DF to continuous zero:**
$$z_c(T) = -\frac{1}{T}\ln P(0,T)$$

**DF to annual zero:**
$$z_a(T) = P(0,T)^{-1/T} - 1 = e^{z_c(T)} - 1$$

**Forward rate (continuous, from zero rates):**
$$R_F = \frac{R_2 T_2 - R_1 T_1}{T_2 - T_1}$$

---

### Flashcards (30 Q/A)

1. **Q:** What is the object built by OIS discounting curve construction?
   **A:** Discount factors $P(0,T)$ (and equivalently zero rates $z(T)$) implied by OIS quotes.

2. **Q:** What does $P(0,T)$ represent?
   **A:** Time-0 value of a claim paying 1 at maturity $T$.

3. **Q:** Define the continuous zero rate $z_c(T)$.
   **A:** $z_c(T) = -\ln P(0,T)/T$.

4. **Q:** Define the annual-compounded zero rate $z_a(T)$.
   **A:** $P(0,T) = (1 + z_a(T))^{-T} \Rightarrow z_a(T) = P(0,T)^{-1/T} - 1$.

5. **Q:** What is an OIS?
   **A:** A swap exchanging a fixed rate for a floating rate based on overnight rates (usually compounded).

6. **Q:** How is the compounded overnight rate $\bar{L}_n$ constructed?
   **A:** Via $\prod_j(1 + r_j \Delta t_j)$ over the accrual period, normalized by $\tau_n$.

7. **Q:** What is the net payoff of an OIS period?
   **A:** $N\tau_n(\bar{L}_n - k)$ at $T_{n+1}$.

8. **Q:** What is a "par OIS rate"?
   **A:** The fixed rate $k$ that makes PV of the swap equal to zero at inception.

9. **Q:** Why are OIS rates useful for curve building?
   **A:** They act as par instruments that constrain discount factors directly.

10. **Q:** What is the "par bond" equation used for bootstrapping?
    **A:** $1 = k\sum_i \tau_i P(0,T_i) + P(0,T_N)$.

11. **Q:** What is the annuity factor in swap/bond terms?
    **A:** $A(0) = \sum_i \tau_i P(0,T_i)$.

12. **Q:** Express the par fixed rate in terms of $P(0,T_N)$ and $A(0)$.
    **A:** $k_{\text{par}} = (1 - P(0,T_N))/A(0)$.

13. **Q:** What is bootstrapping?
    **A:** Sequentially solving for unknown discount factors so each calibration instrument reprices to par.

14. **Q:** Give the one-step bootstrap formula for $P(0,T_N)$.
    **A:** $P(0,T_N) = \frac{1 - k_N \sum_{i=1}^{N-1} \tau_i P(0,T_i)}{1 + k_N \tau_N}$.

15. **Q:** What repricing test must always pass after curve construction?
    **A:** Every calibration OIS instrument prices to $PV \approx 0$ (or par).

16. **Q:** What does "risk-free-ish" mean?
    **A:** Market-implied discounting curve under overnight-index and collateral conventions; not universal.

17. **Q:** Why is OIS often used for discounting in modern practice?
    **A:** Overnight rates are treated as closest proxy to risk-free and collateral remuneration is tied to overnight benchmarks.

18. **Q:** What does Hull say about discounting curve in multiple yield curve world?
    **A:** Discounting uses the risk-free (OIS) zero curve from the OIS market.

19. **Q:** What is a common short-end simplification for OIS curve building?
    **A:** For maturities $\le 1$ year, OIS rates directly define zero rates (after quote-convention conversion).

20. **Q:** What's one reason curves require interpolation?
    **A:** Market quotes exist only at discrete maturities; you need values in between.

21. **Q:** What interpolation does the curve-construction source explicitly describe?
    **A:** Linear in $\ln P(T)$, implying piecewise flat forwards.

22. **Q:** Under $\ln P$ linear interpolation, what happens to forwards between nodes?
    **A:** They are piecewise constant (staircase forward curve).

23. **Q:** What is PV01 conceptually?
    **A:** Change in PV for a 1 bp change in a rate/curve input.

24. **Q:** How can you compute OIS PV01 in practice (basic method)?
    **A:** Bump an OIS quote by 1 bp, rebuild curve, reprice, take PV difference.

25. **Q:** What does Tuckman state about PV01 for fixed cashflows?
    **A:** PV01 equals the sum of discount factors used to discount fixed cashflows (scaled by conventions).

26. **Q:** Why do bumps sometimes affect later maturities in non-obvious ways?
    **A:** Because later bootstraps must still satisfy par conditions, forcing curve twists.

27. **Q:** Are discount factors always decreasing in maturity?
    **A:** No; they are positive, but can increase if rates are negative.

28. **Q:** What market fact supports considering negative rates?
    **A:** Negative interest rates became a feature in some markets from 2014 onward.

29. **Q:** What's a key implementation pitfall in OIS curves?
    **A:** Mis-handling day count/accrual factors and overnight compounding mechanics.

30. **Q:** What information is needed to make this chapter desk-accurate?
    **A:** Currency/index choice (SOFR/€STR/SONIA/etc), OIS compounding/averaging method, payment frequency/day count, CSA collateral terms.

---

## Mini Problem Set

*Instructions: Increasing difficulty. Solution sketches provided for questions 1–8 only.*

---

**1.** (Warm-up) Given $P(0,1) = 0.99$, compute $z_c(1)$.

**Sketch:** $z_c(1) = -\ln(0.99) \approx 0.0100503 = 1.0050\%$.

---

**2.** Given $z_c(2) = 3\%$, compute $P(0,2)$.

**Sketch:** $P(0,2) = e^{-0.03 \times 2} = e^{-0.06} \approx 0.9417645$.

---

**3.** A 2-year annual-pay par bond has coupon $k = 2\%$. If $P(0,1) = 0.985$, solve for $P(0,2)$.

**Sketch:** Par: $1 = 0.02(P_1 + P_2) + P_2 \Rightarrow 1 = 0.02 P_1 + (1.02) P_2$. Solve $P_2 = (1 - 0.02 P_1)/1.02$.

---

**4.** Using Example A's structure, show that $k_{\text{par}} = (1 - P_N)/A$.

**Sketch:** From $1 = kA + P_N \Rightarrow k = (1 - P_N)/A$.

---

**5.** Using Example C data, recompute the 4Y repricing check numerically and show it equals 1 (within rounding).

**Sketch:** Plug $k_4$ and $P_1, \ldots, P_4$ into $k_4 \sum_{i=1}^{4} P_i + P_4$.

---

**6.** With $P(0,2) = 0.95$ and $P(0,3) = 0.92$, compute the average continuous forward rate over $[2,3]$.

**Sketch:** $f_{2,3} = \ln(P(0,2)/P(0,3))/(1) = \ln(0.95/0.92)$.

---

**7.** Suppose an overnight leg uses arithmetic averaging instead of compounding. Qualitatively, what happens when daily rates are volatile?

**Sketch:** Compounding (geometric) and arithmetic averaging differ by a convexity-like effect; compounding can differ from arithmetic average when rates vary (direction depends on variability and level).

---

**8.** Define a basic bump-and-rebuild PV01 for a trade discounted on the OIS curve.

**Sketch:** PV01 $\approx PV(\text{curve with +1bp bump}) - PV(\text{base curve})$, where the bump is applied to a chosen OIS quote and curve is rebuilt.

---

**9.** (No solution sketch) Build a 3-node annual-pay OIS curve from par rates $k_1, k_2, k_3$ and derive closed-form expressions for $P(0,1), P(0,2), P(0,3)$.

---

**10.** (No solution sketch) Using log-DF interpolation, derive the implied constant forward rate between two nodes $(T_1, P_1)$ and $(T_2, P_2)$.

---

**11.** (No solution sketch) Show how a local bump in an intermediate par rate can induce a non-monotonic change in later discount factors under sequential bootstrapping.

---

**12.** (No solution sketch) Construct an example where discount factors increase with maturity and explain the rate environment required.

---

**13.** (No solution sketch) For a quarterly-pay OIS with fixed rate $k$, write the par equation in terms of quarterly discount factors and show how to solve for the last DF.

---

**14.** (No solution sketch) Describe three verification tests you would implement for an OIS curve builder and explain why each catches different bugs.

---

**15.** (No solution sketch) Explain why using the "wrong" discount curve can systematically bias PVs and hedges, referencing Example G logic.

---

**16.** (No solution sketch) Propose a method to smooth a bootstrapped OIS curve and list two trade-offs versus a purely bootstrapped curve.

---

## Source Map

### (A) Verified Facts — Specific Sources

| Fact | Source |
|------|--------|
| OIS exchanges fixed for compounded overnight | Andersen Vol 1 Ch 6, Hull Ch 4 |
| Compounded overnight rate defined as product of daily factors | Andersen Vol 1 Ch 6 |
| OIS rates ≤1Y define zero rates directly; longer define par bonds | Hull Ch 4 |
| OIS discounting linked to collateral remuneration | Andersen Vol 1 Ch 6 |
| Multiple yield curves needed post-crisis | Hull Ch 4, Andersen Vol 1 |
| Linear in ln P → piecewise flat forwards | Andersen Vol 1 Ch 4-6 |
| Par-bond / par-swap condition | Tuckman Ch 1-4, Hull Ch 4 |
| Bootstrap methodology | Hull Ch 4, Andersen Vol 1 Ch 4-6 |
| Annuity factor in swap rate expressions | Hull Ch 7, Andersen Vol 1 |

### (B) Reasoned Inference — Derivation Logic

| Inference | Derivation |
|-----------|------------|
| Par rate formula $k_{\text{par}} = (1 - P_N)/A$ | Algebraic rearrangement of par-bond equation |
| Bootstrap step formula | Isolation of $P(0,T_N)$ from par condition |
| Zero rate conversions | Direct application of exponential/log definitions |
| Interpolated forward rates | Definition of average forward from discount factors |

### (C) Speculation — Flagged Uncertainties

| Topic | Uncertainty |
|-------|-------------|
| Exact fails-charge formula | Sources discuss fails conceptually, not penalty mechanics |
| Formal theorem: "discount at collateral rate" | Excerpts motivate practice but don't provide full derivation |
| Currency/index-specific conventions (observation shift, lockout, day count) | Requires specific index definition (SOFR/SONIA/€STR etc.) |
| OIS stub and end-of-month rules | Not specified in provided excerpts |

---

*Last Updated: January 2026*
