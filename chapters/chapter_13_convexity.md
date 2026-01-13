# Chapter 13: Convexity — Second-Order P&L and When DV01 Breaks

---

## Fact Classification

### (A) Verified Facts (Source-Backed)
- Convexity definition $C = \frac{1}{P}\frac{d^2P}{dy^2}$ (Tuckman Ch 5-6, Hull Ch 4)
- Second-order Taylor expansion for price changes (Tuckman Ch 5)
- DV01-duration relationship $\text{DV01} = \frac{PD}{10,000}$ (Tuckman Ch 5)
- Barbell vs bullet convexity dominance formula (Tuckman Ch 5)
- Expected return decomposition including convexity term $\frac{1}{2}C\sigma^2 dt$ (Tuckman Ch 5)
- Callable bond negative convexity example with specific prices/DV01s (Tuckman callable example)
- Zero-coupon convexity formula $C = \frac{T(T+0.5)}{(1+y/2)^2}$ (Tuckman Ch 5)

### (B) Reasoned Inference (Derived from A)
- All worked example calculations (derived via algebra from source-backed formulas)
- Unit conversion formulas between decimal yield and bp notation
- Symmetry of convexity term for up/down moves (follows from $(\\Delta y)^2$ dependence)
- Interpretation that positive convexity is "long volatility" (derived from Jensen's inequality applied to convex price function)

### (C) Speculation (Clearly Labeled; Minimal)
- None in this chapter; all content is either source-backed or algebraically derived

---

## Conventions & Notation

| Symbol | Meaning | Notes |
|--------|---------|-------|
| $P$ | Price per 100 face (full/dirty) | When accrued interest is zero, clean = dirty |
| $y$ | Annual yield-to-maturity | Semiannual compounding (bond-equivalent convention) unless stated |
| $\Delta y$ | Yield change in decimal | e.g., 100 bp = 0.01 |
| $\Delta y_{\text{bp}}$ | Yield change in bp | $\Delta y_{\text{bp}} = 10{,}000 \cdot \Delta y$ |
| $D$ | Modified duration | $D = -\frac{1}{P}\frac{dP}{dy}$ |
| $C$ | (Normalized) convexity | $C = \frac{1}{P}\frac{d^2P}{dy^2}$ |
| $\text{DV01}$ | Price change per 1 bp yield move | Discrete: $(P_- - P_+)/2$; derivative: $-\frac{1}{10{,}000}\frac{dP}{dy}$ |
| $PC$ | Dollar convexity | $= \frac{d^2P}{dy^2}$ (units: price per yield$^2$) |
| $T$ | Time to maturity in years | With semiannual coupons: $N = 2T$ periods |
| $c$ | Annual coupon rate | Dollars per 100 face |
| $m$ | Compounding frequency | Default: $m=2$ (semiannual) |

**Defaults for this chapter:**
- Yield $y$ is annual, semiannual-compounded
- Coupons are semiannual; times are in exact half-years
- Prices are per 100 face; notional scaling is done at the end
- Sensitivities use the full price; examples set accrued interest = 0 so full = clean
- 1 bp $= 10^{-4}$ in yield decimal

---

## Core Concepts

### 1) Price-Rate Function $P(r)$

**Formal Definition:**
A one-factor model treats the instrument's price as $P(r)$, where $r$ is the single rate variable deemed relevant (e.g., a yield, a swap rate, a curve level).

**Intuition:**
You compress "the whole curve move" into one knob. DV01/duration/convexity are then derivatives of $P(r)$.

**Trading/Risk Practice:**
This is what you implicitly do when you say "this book is \$X DV01" without specifying key rates or curve buckets.

---

### 2) DV01 (First-Order P&L Sensitivity in bp Units)

**Formal Definition (Discrete):**
$$\boxed{\text{DV01} = \frac{P_- - P_+}{2}}$$
where $P_-$ and $P_+$ are prices after a 1 bp downward/upward shift in the relevant rate.

**Formal Definition (Derivative):**
$$\boxed{\text{DV01} \equiv -\frac{1}{10{,}000}\frac{dP}{dy}}$$
when the relevant rate is the yield $y$.

**Intuition:**
DV01 is your linear hedge ratio: how much money you lose/gain for a small 1 bp move.

**Trading/Risk Practice:**
- DV01-neutral $\neq$ "risk-neutral"; it only neutralizes the first derivative at the calibration point
- DV01 depends on which bump you define (yield bump vs curve bump; parallel vs key-rate)

---

### 3) Duration $D$ (Normalized Slope of Price-Yield Curve)

**Formal Definition:**
$$\boxed{D \equiv -\frac{1}{P}\frac{dP}{dy}}$$

**Intuition:**
Duration is the percentage price sensitivity to yield—your "delta" in rate space.

**Trading/Risk Practice:**
- "DV01" is usually what desks hedge (dollar sensitivity), while "duration" is DV01 scaled by price:
$$\text{DV01} = \frac{P \cdot D}{10{,}000}$$
- Duration hedges work well for small moves; they fail for large moves because curvature matters

---

### 4) Convexity $C$ (Normalized Curvature; Second-Order P&L)

**Formal Definition:**
$$\boxed{C \equiv \frac{1}{P}\frac{d^2P}{dy^2}}$$

**Intuition:**
Convexity is the curvature of the price-yield curve—your "gamma" in rate space.

**Trading/Risk Practice:**
- Convexity explains why DV01 hedges "drift" when yields move
- Long convexity is (locally) "long volatility" because curvature makes $\mathbb{E}[P(y)]$ exceed $P(\mathbb{E}[y])$ when rates are volatile

---

### 5) Positive vs Negative Convexity

**Option-Free Fixed-Coupon Bonds:**
- **Typical sign:** Positive convexity ($C > 0$), meaning the price-yield curve is convex upward
- **Practical meaning:** For the same DV01, you lose less when yields rise and gain more when yields fall, versus a purely linear exposure

**Option-Embedded Instruments:**
- Some securities (e.g., callable bonds, MBS) can have **negative convexity** at certain yield levels
- For a callable: price can become concave because the issuer's call option has convex value, so "bond minus option" can be concave $\Rightarrow$ negative second derivative

---

### 6) "When DV01 Breaks" (High-Level Taxonomy)

DV01 is a local, one-factor, first-order risk measure. It breaks (i.e., produces hedge errors) when:

| Failure Mode | Explanation | Reference |
|--------------|-------------|-----------|
| **Large moves** | Second-order term matters | §3.2–3.4 |
| **Non-parallel curve moves** | Curve not parallel; key rates move differently | §3.5 |
| **Convexity mismatch** | DV01-neutral portfolios have nonzero P&L for big moves | §3.4, Example D |
| **Optionality/regime dependence** | Duration/convexity change with rate level and volatility | §3.7, Example G |

---

## Math and Derivations

### 2.1 Taylor Expansion: Where Convexity Enters P&L

Let $P = P(y)$ and consider a yield move from $y$ to $y + \Delta y$. A second-order Taylor expansion gives:

$$\boxed{\frac{\Delta P}{P} \approx -D\,\Delta y + \frac{1}{2}C\,(\Delta y)^2}$$

where $D = -\frac{1}{P}\frac{dP}{dy}$ and $C = \frac{1}{P}\frac{d^2P}{dy^2}$.

**Unit/Sanity Checks:**
- $y$ is a rate in decimal (e.g., 5% = 0.05), so $\Delta y$ is dimensionless
- $D$ has units of "per yield unit" and is commonly interpreted in years (because it is PV-weighted time under standard bond pricing)
- $C$ has units of "per yield unit squared," often interpreted as years$^2$

**Key Symmetry:**
The convexity term uses $(\Delta y)^2$, so it is symmetric: it contributes the same sign to P&L for up and down moves when $C$ is constant over the move.

---

### 2.2 Duration-Only vs Duration+Convexity Approximation

**Duration-only (first-order):**
$$\frac{\Delta P}{P} \approx -D\,\Delta y \tag{duration-only}$$

**Duration + convexity (second-order):**
$$\frac{\Delta P}{P} \approx -D\,\Delta y + \frac{1}{2}C\,(\Delta y)^2 \tag{duration+convexity}$$

---

### 2.3 Translating the Taylor Expansion into DV01 Language

Start from the derivative DV01 definition:
$$\text{DV01} = -\frac{1}{10{,}000}\frac{dP}{dy}$$

Because $D = -\frac{1}{P}\frac{dP}{dy}$, we have:
$$\boxed{\text{DV01} = \frac{P \cdot D}{10{,}000}}$$

Now rewrite first-order price change:
$$\Delta P \approx \frac{dP}{dy}\Delta y = -10{,}000 \cdot \text{DV01} \cdot \Delta y = -\text{DV01} \cdot \Delta y_{\text{bp}}$$

For the second-order term:
$$\Delta P_{\text{conv}} \approx \frac{1}{2}\frac{d^2P}{dy^2}(\Delta y)^2 = \frac{1}{2}(PC)(\Delta y)^2 = \frac{1}{2}\left(\frac{PC}{10{,}000^2}\right)(\Delta y_{\text{bp}})^2$$

**Practical Risk-Reporting Units:**
- $PC$ ("dollar convexity" per 100) has units: price points per (yield$^2$)
- $\frac{PC}{10{,}000^2}$ has units: price points per bp$^2$

---

### 2.4 Numerical Estimation of Convexity (Finite Differences)

If you reprice at $y - \Delta y$, $y$, and $y + \Delta y$, you can estimate the second derivative (central difference):

$$\frac{d^2P}{dy^2} \approx \frac{P_+ - 2P_0 + P_-}{(\Delta y)^2}$$

and then:
$$C \approx \frac{1}{P}\frac{d^2P}{dy^2}$$

---

### 2.5 Yield-Based Convexity for Standard Coupon Bonds (Semiannual Compounding)

For a bond with annual coupon $c$ (dollars per 100), maturity $T$ (years), semiannual yield $y$, and price:

$$P = \sum_{t=1}^{2T}\frac{c/2}{(1+y/2)^t} + \frac{100}{(1+y/2)^{2T}}$$

yield-based convexity is:
$$C \equiv \frac{1}{P}\frac{d^2P}{dy^2}$$

and can be expressed (after differentiation) as:

$$\boxed{C = \frac{1}{P(1+y/2)^2}\left[\sum_{t=1}^{2T}\frac{t}{2}\frac{t+1}{2}\frac{c/2}{(1+y/2)^t} + T(T+0.5)\frac{100}{(1+y/2)^{2T}}\right]}$$

**Special Case: Zero-Coupon**

Setting $c = 0$ yields:
$$\boxed{C = \frac{T(T+0.5)}{(1+y/2)^2}}$$

---

### 2.6 Barbell vs Bullet and Convexity Dominance (Same Duration)

If two portfolios have the **same modified duration**, their return difference for a yield change $\Delta y$ is driven (to second order) by convexity:

$$\boxed{\bar{R}_B - \bar{R}_P = \frac{1}{2}(C_B - C_P)(\Delta y)^2}$$

This shows why, for any nonzero $|\Delta y|$, **higher convexity dominates** (second-order) if duration is matched.

---

### 2.7 Convexity as Exposure to Volatility (Expected Return Contribution)

Under a simple yield diffusion, Tuckman shows an expected return decomposition including a convexity term:

$$\mathbb{E}\left[\frac{dP}{P}\right] + \frac{c\,dt}{P} = y\,dt - D\,\mathbb{E}[dy] + \frac{1}{2}C\sigma^2 dt$$

So (holding other terms fixed) more yield variance increases expected return through the $\frac{1}{2}C\sigma^2 dt$ term.

---

## Measurement & Risk

### 3.1 Define Convexity Precisely (and State Yield Convention)

In this chapter's default convention (semiannual-compounded yield-to-maturity), convexity is:

$$C = \frac{1}{P}\frac{d^2P}{dy^2}$$

Alternative conventions exist (e.g., continuous compounding; different scaling such as "dollar convexity" or "per bp$^2$" reports). Hull defines convexity analogously as $C = \frac{1}{B}\frac{d^2B}{dy^2}$ and uses it in the second-order bond price approximation.

**Key Warning:** Convexity values are not invariant to yield definition (BEY vs continuous). Always report the convention.

---

### 3.2 Second-Order Taylor Approximation for Price Change

Let $\Delta y$ be a yield move in decimal.

**Duration-only:**
$$\frac{\Delta P}{P} \approx -D\,\Delta y$$

**Duration + convexity:**
$$\frac{\Delta P}{P} \approx -D\,\Delta y + \frac{1}{2}C\,(\Delta y)^2$$

---

### 3.3 Translate into DV01 Language with Explicit bp Conversions

Using $\Delta y_{\text{bp}} = 10{,}000\,\Delta y$ and $\text{DV01} = \frac{PD}{10{,}000}$:

**Duration-only (DV01 form):**
$$\Delta P \approx -\text{DV01} \cdot \Delta y_{\text{bp}}$$

**Duration + convexity (DV01 + convexity form):**
$$\Delta P \approx -\text{DV01} \cdot \Delta y_{\text{bp}} + \frac{1}{2}\left(\frac{PC}{10{,}000^2}\right)(\Delta y_{\text{bp}})^2$$

**Unit Check (per 100 face):**
- DV01: price points per bp
- $\frac{PC}{10{,}000^2}$: price points per bp$^2$
- Multiply by $(\Delta y_{\text{bp}})^2$ and by $1/2$ → price points

---

### 3.4 Convexity Sign and Its P&L Meaning

**Option-free fixed-coupon bonds:** typically $C > 0$. An equivalent intuition: DV01 tends to fall as yields rise (the slope becomes less negative).

**Option-embedded bonds (callables/MBS):** can have $C < 0$ at certain yield levels. For callables, the bond price curve can become concave because the call option's convexity "subtracts" from the bond.

---

### 3.5 Curve Moves: DV01 is Defined for a Specific Bump

Yield-based DV01 is a one-factor measure that assumes the bond is exposed only to the level of the curve and the curve shifts in parallel, which is unrealistic in practice.

**Preview (not a full key-rate chapter):**
- Non-parallel curve changes require multiple sensitivities (key-rate DV01s), because different maturities can move differently

---

### 3.6 Convexity Mismatch in DV01-Hedged Portfolios

Even if a portfolio is DV01-neutral at the current yield:

A large move produces a second-order P&L:
$$\Delta P_{\text{2nd}} \approx \frac{1}{2}(PC)_{\text{net}}(\Delta y)^2$$

where $(PC)_{\text{net}}$ is the portfolio's net dollar convexity.

This is the core reason DV01-only hedges fail for large moves (Example D).

---

### 3.7 Optionality / Regime Dependence (Effective Convexity Changes)

Callables show that DV01, duration, and convexity are functions of the level of rates (and, in practice, volatility). In Tuckman's callable example, the callable bond's DV01 and convexity vary materially across rate levels.

---

## Worked Examples

**Global Example Conventions (unless overridden):**
- Price is per 100 face, full/dirty; we set accrued interest = 0 (so clean = dirty)
- Yield $y$ is annual, semiannual-compounded
- Coupons are semiannual; times are in exact half-years
- "Exact repricing" means repricing using the bond pricing formula under the stated yield convention

---

### Example A — Zero-Coupon Convexity (Analytic vs Finite Difference; Maturity Dependence; Unit Checks)

**Instrument:** Zero-coupon, face 100, maturity $T$ years, yield $y$ (semiannual compounding).

**Price:**
$$P(y) = \frac{100}{(1+y/2)^{2T}}$$

#### A1. Analytic Convexity (Tuckman Special Case)

Zero-coupon convexity:
$$C = \frac{T(T+0.5)}{(1+y/2)^2}$$

Take $y = 5\% = 0.05$, so $(1+y/2) = 1.025$ and $(1+y/2)^2 = 1.050625$.

**Compute maturity dependence:**

| $T$ (yrs) | $C = \frac{T(T+0.5)}{1.025^2}$ |
|----------:|------------------------------:|
| 2 | 4.7591 |
| 5 | 26.1749 |
| 10 | 99.9405 |
| 30 | 870.9102 |

**Sanity check:** Convexity scales roughly like $T^2$, consistent with the $T(T+0.5)$ term.

#### A2. Finite-Difference Convexity Estimate

Use $T = 5$, $y = 5\%$, bump $\Delta y = 1\text{ bp} = 0.0001$.

Compute:
- $P_0 = P(0.05) = 78.11984017$
- $P_- = P(0.0499) = 78.15794943$
- $P_+ = P(0.0501) = 78.08184011$

Central second derivative:
$$\frac{d^2P}{dy^2} \approx \frac{P_+ - 2P_0 + P_-}{(\Delta y)^2}$$

and convexity $C \approx \frac{1}{P_0}\frac{d^2P}{dy^2}$.

**Numeric:**
- $d^2P/dy^2 \approx 2044.7787$
- $C_{\text{FD}} \approx 2044.7787 / 78.11984017 = 26.1749$

**Comparison:** $C_{\text{analytic}} = 26.1749$ vs $C_{\text{FD}} = 26.1749$ (matches to ~$10^{-6}$).

#### A3. Exact Repricing vs Approximations for a +100 bp Shock

Need duration for a zero. Tuckman gives:
$$D_{\text{Mod}}\big|_{c=0} = \frac{T}{1+y/2}, \qquad \text{DV01}\big|_{c=0} = \frac{P}{10{,}000} \cdot \frac{T}{1+y/2}$$

At $T = 5$, $y = 5\%$:
- $D = 5/1.025 = 4.87804878$
- $P_0 = 78.11984017$
- $\text{DV01} = P_0 D / 10{,}000 = 0.03810724$

**Shock:** $\Delta y = +0.01$ (100 bp) → $y_1 = 6\%$.

| Method | $\Delta P$ |
|--------|----------:|
| **Exact** | $P_1 = 74.40939149$ ⇒ $\Delta P_{\text{exact}} = -3.71044868$ |
| **Duration-only** | $\Delta P \approx -P_0 D \Delta y = -3.81072391$ |
| **Duration+convexity** | $\Delta P \approx P_0(-D\Delta y + \frac{1}{2}C\Delta y^2) = -3.70848498$ |

**Errors:**
- Duration-only error: $-0.10027523$ (overstates loss)
- Duration+convexity error: $+0.00196371$ (much tighter)

---

### Example B — Coupon Bond Convexity via Cashflows (PV Weights) vs Finite Differences

**Instrument:** 3-year bond, face 100, annual coupon $c = 6$ (paid semiannually → 3 per period), yield $y = 5\%$.

Number of periods: $N = 2T = 6$.
Discount factor per period: $1/(1+y/2) = 1/1.025$.

#### B1. Exact Repricing (Price)

$$P = \sum_{t=1}^{6}\frac{3}{(1.025)^t} + \frac{100}{(1.025)^6} = 102.75406268$$

#### B2. Convexity from Cashflows (Using Tuckman's Yield-Based Convexity Expression)

$$C = \frac{1}{P(1+y/2)^2}\left[\sum_{t=1}^{6}\frac{t}{2}\frac{t+1}{2}\frac{3}{(1.025)^t} + T(T+0.5)\frac{100}{(1.025)^6}\right]$$

**Compute PVs and weights (intermediate steps):**

| $t$ | $\tau_t = t/2$ | DF $= (1.025)^{-t}$ | PV(cpn) = $3 \cdot$ DF | factor $\tau_t(\tau_t+0.5)$ | weighted PV |
|----:|---------------:|--------------------:|-----------------------:|---------------------------:|------------:|
| 1 | 0.5 | 0.97560976 | 2.92682927 | 0.5 | 1.46341463 |
| 2 | 1.0 | 0.95181440 | 2.85544319 | 1.5 | 4.28316478 |
| 3 | 1.5 | 0.92859941 | 2.78579823 | 3.0 | 8.35739470 |
| 4 | 2.0 | 0.90595064 | 2.71785193 | 5.0 | 13.58925967 |
| 5 | 2.5 | 0.88385429 | 2.65156286 | 7.5 | 19.88672147 |
| 6 | 3.0 | 0.86229687 | 2.58689060 | 10.5 | 27.16235128 |

Principal PV: $100 \cdot 0.86229687 = 86.22968660$, factor $T(T+0.5) = 3 \cdot 3.5 = 10.5$, weighted PV $= 905.41170926$.

**Sum weighted PVs:**
- Coupons: $74.74230654$
- Principal: $905.41170926$
- Numerator: $980.15401579$

**Denominator:** $P(1.025)^2 = 102.75406268 \cdot 1.050625 = 107.95598710$.

Thus:
$$C_{\text{CF}} = \frac{980.15401579}{107.95598710} = 9.07920016$$

#### B3. Convexity by Finite Differences (Exact Repricing at Bumped Yields)

Reprice at $y \pm 1$ bp:
- $P_- = 102.78206851$ at 4.99%
- $P_0 = 102.75406268$ at 5.00%
- $P_+ = 102.72606618$ at 5.01%

Then:
$$C_{\text{FD}} = \frac{P_+ - 2P_0 + P_-}{P_0(\Delta y)^2} = \frac{102.72606618 - 2(102.75406268) + 102.78206851}{102.75406268(10^{-4})^2} = 9.07920034$$

**Comparison:** $C_{\text{CF}} \approx 9.07920016$ vs $C_{\text{FD}} \approx 9.07920034$ (tiny numerical difference).

#### B4. Exact Repricing vs Duration-Only vs Duration+Convexity (Quick Check, +50 bp)

Compute DV01 (±1bp):
$$\text{DV01} = \frac{P_- - P_+}{2} = \frac{102.78206851 - 102.72606618}{2} = 0.02800116$$

Then $D = \frac{10{,}000 \cdot \text{DV01}}{P_0} = 2.72506611$.

**Shock $\Delta y = +0.005$ (50 bp):**

| Method | Result |
|--------|-------:|
| Exact | $P(5.5\%) = 101.36559169$ ⇒ $\Delta P_{\text{exact}} = -1.38847099$ |
| Duration-only | $\Delta P \approx -P_0 D \Delta y = -1.40005807$ |
| Duration+convexity | $\Delta P \approx P_0(-D\Delta y + \frac{1}{2}C\Delta y^2) = -1.38839651$ |

---

### Example C — Duration-Only vs Duration+Convexity vs Exact for a +100 bp Shock (Error Scaling)

**Reuse Example B bond:** 3-year, 6% coupon, $y_0 = 5\%$, $P_0 = 102.75406268$.

From Example B:
- $D = 2.72506611$
- $C = 9.07920034$

**Shock:** $\Delta y = +0.01$ (100 bp) → $y_1 = 6\%$.

| Method | Result |
|--------|-------:|
| Exact | $P_1 = 100.00000000$ ⇒ $\Delta P_{\text{exact}} = -2.75406268$ |
| Duration-only | $\Delta P = -P_0 D \Delta y = -2.80011614$ |
| Duration+convexity | $\Delta P = P_0(-D\Delta y + \frac{1}{2}C\Delta y^2) = -2.75346990$ |

**Errors:**
- Duration-only error: $-0.04605346$
- Duration+convexity error: $+0.00059278$

**Scaling with Shock Size (Exact vs Approx):**

| Shock | $y_1$ | $\Delta P_{\text{exact}}$ | dur-only | dur+conv | dur-only error | dur+conv error |
|------:|------:|-------------------------:|---------:|---------:|---------------:|---------------:|
| +25 bp | 5.25% | -0.69712297 | -0.70002903 | -0.69711364 | -0.00290607 | +0.00000932 |
| +50 bp | 5.50% | -1.38847099 | -1.40005807 | -1.38839651 | -0.01158708 | +0.00007448 |
| +100 bp | 6.00% | -2.75406268 | -2.80011614 | -2.75346990 | -0.04605346 | +0.00059278 |

**Interpretation:** Duration-only error grows like $(\Delta y)^2$; adding convexity pushes leading error to higher order, consistent with the second-order Taylor structure.

---

### Example D — DV01-Hedged but Not Convexity-Hedged (Convexity Mismatch P&L)

**Goal:** Build a DV01-neutral portfolio at $y_0 = 5\%$, then show nonzero P&L for large moves due to convexity mismatch.

**Bond A:** 10-year par bond, coupon 5%, $P_{A,0} = 100$.
**Bond B:** 2-year par bond, coupon 5%, $P_{B,0} = 100$.
(Prices are par at 5% yield by construction.)

#### D1. Compute DV01s (±1 bp Repricing)

Using $\text{DV01} = (P_- - P_+)/2$:

**Bond A:**
- $P_- = 100.07798264$ (4.99%)
- $P_+ = 99.92209099$ (5.01%)
- $\text{DV01}_A = (100.07798264 - 99.92209099)/2 = 0.07794582$

**Bond B:**
- $P_- = 100.01881214$
- $P_+ = 99.98119239$
- $\text{DV01}_B = 0.01880987$

#### D2. Hedge Ratio for DV01 Neutrality

Long 1 unit of A, short $h$ units of B:
$$\text{DV01}_A - h\,\text{DV01}_B = 0 \Rightarrow h = \frac{\text{DV01}_A}{\text{DV01}_B} = 4.14387866$$

#### D3. Compute Convexities (±1 bp)

$$C = \frac{P_+ - 2P_0 + P_-}{P_0(\Delta y)^2} \quad (\Delta y = 10^{-4})$$

- $C_A = 73.62873833$
- $C_B = 4.53114117$

#### D4. Apply a Large Move: +100 bp

New yield $y_1 = 6\%$.

**Exact repricing:**
- $P_{A,1} = 92.56126257$ ⇒ $\Delta P_A = -7.43873743$
- $P_{B,1} = 98.14145080$ ⇒ $\Delta P_B = -1.85854920$

**Portfolio (long A, short $h$B):**
$$\Delta P_{\text{port, exact}} = \Delta P_A - h\,\Delta P_B = -7.43873743 - 4.14387866(-1.85854920) = +0.26286500$$

#### D5. Second-Order (Convexity) Explanation

Since the portfolio is DV01-neutral at $y_0$, the first-order term cancels locally; second-order predicts:
$$\Delta P_{\text{port}} \approx \frac{1}{2}(P_A C_A - h P_B C_B)(\Delta y)^2$$

With $P_A = P_B = 100$, $\Delta y = 0.01$:
$$\Delta P_{\text{port, 2nd}} \approx 0.27426119$$

Close to exact $0.26286500$. (Difference comes from higher-order terms and the fact that DV01 neutrality is local.)

#### D6. Symmetry Check: -100 bp

At $y_1 = 4\%$:
- $P_{A,1} = 108.17571667$ ⇒ $\Delta P_A = +8.17571667$
- $P_{B,1} = 101.90386435$ ⇒ $\Delta P_B = +1.90386435$

**Portfolio:**
$$\Delta P_{\text{port, exact}} = 8.17571667 - 4.14387866(1.90386435) = +0.28633399$$

Again positive, consistent with positive net convexity (second-order term depends on $(\Delta y)^2$).

---

### Example E — Barbell vs Bullet (Same Duration, Different Convexity)

**Bullet:** 10-year par bond (5% coupon, $P = 100$).
**Barbell:** Mix of 2-year and 30-year par bonds (both 5% coupon, $P = 100$).

#### E1. Compute Durations (via DV01)

- $D_2 = 1.88098713$
- $D_{10} = 7.79458241$
- $D_{30} = 15.45434366$

#### E2. Choose Barbell Weights to Match Duration

Let $w$ be weight in 30-year, $(1-w)$ in 2-year (weights by market value; prices are both 100 so weights = notional fractions). Solve:

$$wD_{30} + (1-w)D_2 = D_{10} \Rightarrow w = \frac{D_{10} - D_2}{D_{30} - D_2} = 0.43567671$$

So barbell = **43.5677% 30y + 56.4323% 2y**.

#### E3. Compute Convexities

Using ±1bp:
- $C_2 = 4.53114117$
- $C_{10} = 73.62873833$
- $C_{30} = 352.08524430$

**Barbell convexity (value-weighted):**
$$C_{\text{bar}} = wC_{30} + (1-w)C_2 = 155.95236844$$

which is much higher than bullet $C_{10} = 73.62873833$.

#### E4. Exact Repricing Comparison for ±100 bp

**At 4% yield:**
- Bullet: $P_{10}(4\%) = 108.17571667$
- Barbell: $P_{\text{bar}}(4\%) = 0.43567671 \cdot 117.38044334 + 0.56432329 \cdot 101.90386435 = 108.64664932$
- **Barbell advantage:** $+0.47093265$

**At 6% yield:**
- Bullet: $P_{10}(6\%) = 92.56126257$
- Barbell: $P_{\text{bar}}(6\%) = 0.43567671 \cdot 86.16221817 + 0.56432329 \cdot 98.14145080 = 92.92237817$
- **Barbell advantage:** $+0.36111560$

#### E5. Convexity Explanation (Second-Order Dominance)

Tuckman's barbell-bullet relation (duration-matched) is:
$$\bar{R}_B - \bar{R}_P = \frac{1}{2}(C_B - C_P)(\Delta y)^2$$

Using $P \approx 100$, $\Delta y = 0.01$:
$$\Delta P_{\text{bar}} - \Delta P_{\text{bullet}} \approx \frac{1}{2} \cdot 100 \cdot (155.9524 - 73.6287) \cdot 0.01^2 = 0.41161815$$

Close to the exact differences above.

**Desk interpretation:** A barbell often carries "extra convexity" relative to a bullet at similar duration, which matters in volatile/risky rate environments.

---

### Example F — Convexity "Value" via Jensen Effect (Expected P&L Under Volatility)

This example quantifies the statement: **positive convexity is exposure to volatility**.

**Use the 10-year par bond at $y_0 = 5\%$:** $P_0 = 100$, $C = 73.62873833$.

#### F1. Toy Two-State Rate Move (Exact Expected Price)

Assume with 50/50 probability:
- Yields go to 4% ($\Delta y = -0.01$) → $P(4\%) = 108.17571667$
- Yields go to 6% ($\Delta y = +0.01$) → $P(6\%) = 92.56126257$

**Exact expected price:**
$$\mathbb{E}[P] = \frac{108.17571667 + 92.56126257}{2} = 100.36848962$$

So $\mathbb{E}[\Delta P] = +0.36848962$.

#### F2. Second-Order Approximation (Convexity-Only Since $\mathbb{E}[\Delta y] = 0$)

From the Taylor expansion:
$$\mathbb{E}\left[\frac{\Delta P}{P}\right] \approx \frac{1}{2}C\,\mathbb{E}[(\Delta y)^2]$$

Here $\mathbb{E}[(\Delta y)^2] = (0.01)^2 = 0.0001$. Thus:
$$\mathbb{E}[\Delta P] \approx P_0 \cdot \frac{1}{2}C \cdot 0.0001 = 100 \cdot \frac{1}{2} \cdot 73.62873833 \cdot 0.0001 = 0.36814369$$

**Comparison:** Exact $0.36848962$ vs approx $0.36814369$ (very close).

#### F3. Link to Tuckman's Expected Return Decomposition

Tuckman's continuous-time expression includes the convexity term $\frac{1}{2}C\sigma^2 dt$ in expected return. Our toy example is the discrete analogue where $\sigma^2 dt$ is replaced by $\text{Var}(\Delta y)$.

---

### Example G — Negative Convexity Toy (Callable Preview Using Tuckman's Callable Example)

**Source-backed callable example:** 5% Treasury callable in one year at par ("5s of Feb 15, 2011 Callable Feb 15, 2002 at 100").

**Table excerpt (prices and risk measures as functions of rate level):**

| Rate | Callable Price | DV01 | Duration | Convexity |
|-----:|---------------:|-----:|---------:|----------:|
| 4% | 100.0251 | 0.0216 | 2.162983 | -146.618 |
| 5% | 96.9499 | 0.0411 | 4.238815 | -223.039 |
| 6% | 91.8734 | 0.0586 | 6.376924 | -119.563 |

#### G1. "Flattening" vs the Noncallable Bond

From the same table, the noncallable bond price at 4% is $108.1757$, while the callable is only $100.0251$.

This is the hallmark of **negative convexity**: when yields fall, price appreciation is capped by callability.

#### G2. Sign + Regime Dependence

- Callable convexity is **negative** at these rate levels (e.g., $-223.039$ at 5%), consistent with the concavity argument: the call option is convex, so "bond minus option" can be concave ⇒ negative second derivative
- DV01 **increases** as rates rise in the table (0.0216 → 0.0411 → 0.0586), consistent with negative convexity intuition (slope becomes more negative as yields rise)

#### G3. Exact Repricing vs Duration-Only vs Duration+Convexity (from the 5% Point)

Take the 5% row as the expansion point:
- $P_0 = 96.9499$
- $\text{DV01} = 0.0411$
- $C = -223.039$

**Move to 4% ($\Delta y = -0.01$, $\Delta y_{\text{bp}} = -100$):**

| Method | Result |
|--------|-------:|
| Exact | $P(4\%) = 100.0251$ ⇒ $\Delta P_{\text{exact}} = +3.0752$ |
| Duration-only | $\Delta P \approx -\text{DV01} \cdot \Delta y_{\text{bp}} = -0.0411(-100) = +4.11$ |
| Duration+convexity | $\Delta P \approx -\text{DV01}\,\Delta y_{\text{bp}} + \frac{1}{2}P_0 C(\Delta y)^2 = 4.11 + \frac{1}{2}(96.9499)(-223.039)(0.01)^2 = 3.0288$ |

**Move to 6% ($\Delta y = +0.01$):**

| Method | Result |
|--------|-------:|
| Exact | $P(6\%) = 91.8734$ ⇒ $\Delta P_{\text{exact}} = -5.0765$ |
| Duration-only | $-4.11$ |
| Duration+convexity | $-5.1912$ |

**Interpretation:** The negative convexity term $\frac{1}{2}C(\Delta y)^2$ is always negative, reducing gains in rallies and worsening losses in selloffs (relative to duration-only).

---

### Example H — Risk Reporting Units (per Yield Unit vs per bp$^2$) + Convexity P&L Term

**Use the 10-year par bond at $y_0 = 5\%$:** $P_0 = 100$, $D = 7.79458241$, $C = 73.62873833$.

#### H1. Convert Convexity to "per bp$^2$" Units

By definition, $C$ is per (decimal yield)$^2$. Since $\Delta y = \Delta y_{\text{bp}}/10{,}000$:
$$\frac{1}{2}C(\Delta y)^2 = \frac{1}{2}\left(\frac{C}{10{,}000^2}\right)(\Delta y_{\text{bp}})^2$$

So:
$$C_{\text{bp}^2} = \frac{C}{10{,}000^2} = 7.362873833 \times 10^{-7} \text{ per bp}^2$$

**Dollar convexity per bp$^2$ (per 100 face):**
$$\left(\frac{PC}{10{,}000^2}\right) = 7.362873833 \times 10^{-5} \text{ price points per bp}^2$$

#### H2. Compute Convexity P&L Term for a 75 bp Move

Let $\Delta y_{\text{bp}} = +75$ (so $\Delta y = 0.0075$).

**Convexity contribution (per 100 face):**
$$\Delta P_{\text{conv}} \approx \frac{1}{2}P_0 C(\Delta y)^2 = \frac{1}{2} \cdot 100 \cdot 73.62873833 \cdot (0.0075)^2 = 0.20708083$$

So on \$25mm face:
$$\text{P\&L}_{\text{conv}} \approx 0.20708083\text{ points} \times \frac{25{,}000{,}000}{100} = \$51{,}770.21$$

#### H3. Exact Repricing vs Duration-Only vs Duration+Convexity for the Same 75 bp Move

New yield $y_1 = 5.75\%$:

| Method | Result |
|--------|-------:|
| Exact repricing | $P_1 = 94.35592087$ ⇒ $\Delta P_{\text{exact}} = -5.64407913$ |
| Duration-only | $\Delta P = -P_0 D \Delta y = -5.84593680$ |
| Duration+convexity | $\Delta P = P_0(-D\Delta y + \frac{1}{2}C\Delta y^2) = -5.63885598$ |

**Errors:**
- Duration-only error: $-0.20185768$
- Duration+convexity error: $+0.00522315$

---

## Practical Notes

### Common Pitfalls

1. **Confusing raw second derivative with normalized convexity:**
   - Raw: $\frac{d^2P}{dy^2}$
   - Normalized: $C = \frac{1}{P}\frac{d^2P}{dy^2}$

2. **Mixing yield conventions (BEY vs continuous)** and getting inconsistent DV01/convexity numbers. Hull's bond convexity is defined similarly but can be presented under different yield compounding conventions.

3. **Using clean price in sensitivity calculations** when accrued interest is material; Tuckman notes duration definitions use the full/invoice price when accrued interest is not zero.

4. **Assuming convexity is always positive:** Callable bonds and MBS can have negative convexity at certain rate levels.

5. **Assuming DV01-neutral means risk-neutral:** Convexity mismatch causes P&L for large moves (Example D).

### Implementation Pitfalls

1. **Bump size choice for stable second derivatives:**
   - Too small → numerical noise (pricing rounding, curve construction noise)
   - Too large → higher-order terms contaminate the convexity estimate

2. **Numerical noise:** Convexity uses differences of differences; it is much noisier than DV01.

3. **Schedule issues (stubs, near coupon date):** Sensitivities can jump because accrued interest and cashflow timing changes matter (even for option-free bonds).

4. **Optionality:** Effective convexity depends on rate level (and, in practice, volatility); "one convexity number" can be misleading.

### Verification Tests

- For option-free fixed-coupon bonds, convexity should be **positive** under standard assumptions
- Convexity term is **symmetric** in $\Delta y$ because it depends on $(\Delta y)^2$
- Smaller bumps should converge toward stable estimates (if pricing is smooth)
- For small moves, duration+convexity approximation should be close to exact repricing; duration-only errors should grow roughly with $(\Delta y)^2$ (Example C)

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **DV01** is a first-order (linear) risk measure: price change per 1 bp yield move
2. **Duration** is the normalized slope: $D = -\frac{1}{P}\frac{dP}{dy}$
3. **Convexity** is the normalized curvature: $C = \frac{1}{P}\frac{d^2P}{dy^2}$
4. **Second-order P&L:** $\frac{\Delta P}{P} \approx -D\Delta y + \frac{1}{2}C(\Delta y)^2$
5. **DV01-only hedges fail** for large moves because they ignore the $(\Delta y)^2$ term
6. If two portfolios have the same duration, **higher convexity dominates:** return difference $\approx \frac{1}{2}(C_B - C_P)(\Delta y)^2$
7. **Option-free fixed-coupon bonds** typically have positive convexity; callable/MBS can have negative convexity at some rate levels
8. **Convexity is (locally) exposure to rate volatility:** expected return can include $\frac{1}{2}C\sigma^2 dt$
9. **Yield-based DV01 assumes parallel shifts**, which is unrealistic; curve shape risk requires richer measures (key rates)
10. **"DV01-neutral" is not "risk-neutral":** convexity mismatch and non-parallel moves produce residual P&L

### Cheat Sheet: Key Formulas + Unit Conversions

| Formula | Expression |
|---------|------------|
| DV01 (discrete, ±1bp) | $\text{DV01} = \frac{P_- - P_+}{2}$ |
| DV01 (derivative) | $\text{DV01} = -\frac{1}{10{,}000}\frac{dP}{dy}$ |
| Modified duration | $D = -\frac{1}{P}\frac{dP}{dy}$ |
| Convexity | $C = \frac{1}{P}\frac{d^2P}{dy^2}$ |
| Second-order price change | $\frac{\Delta P}{P} \approx -D\Delta y + \frac{1}{2}C(\Delta y)^2$ |
| DV01-duration relation | $\text{DV01} = \frac{PD}{10{,}000}$ |
| bp conversion | $\Delta y_{\text{bp}} = 10{,}000\,\Delta y$, $\Delta y = \frac{\Delta y_{\text{bp}}}{10{,}000}$ |
| DV01 form of duration-only | $\Delta P \approx -\text{DV01} \cdot \Delta y_{\text{bp}}$ |
| Convexity P&L in bp units | $\Delta P_{\text{conv}} \approx \frac{1}{2}\left(\frac{PC}{10{,}000^2}\right)(\Delta y_{\text{bp}})^2$ |

### Flashcards (25 Q/A)

**Q1:** What does DV01 measure?
**A:** Approximate price change per 1 bp change in the relevant rate; for yields, $\text{DV01} \approx -\frac{1}{10{,}000}\frac{dP}{dy}$.

**Q2:** What is the discrete DV01 definition used by Tuckman?
**A:** $\text{DV01} = (P_- - P_+)/2$ where $P_-$ and $P_+$ are prices after ±1 bp shifts.

**Q3:** Define modified duration.
**A:** $D = -\frac{1}{P}\frac{dP}{dy}$.

**Q4:** Define (normalized) convexity.
**A:** $C = \frac{1}{P}\frac{d^2P}{dy^2}$.

**Q5:** What is the second-order approximation for $\Delta P/P$?
**A:** $\Delta P/P \approx -D\Delta y + \frac{1}{2}C(\Delta y)^2$.

**Q6:** Why is the convexity term symmetric in yield moves?
**A:** It depends on $(\Delta y)^2$, so it has the same sign for $\pm\Delta y$.

**Q7:** How do you convert a yield move from bp to decimal?
**A:** $\Delta y = \Delta y_{\text{bp}}/10{,}000$.

**Q8:** What is the relationship between DV01 and duration?
**A:** $\text{DV01} = \frac{PD}{10{,}000}$.

**Q9:** What does positive convexity imply for large moves vs a linear approximation?
**A:** Losses for yield rises are smaller and gains for yield falls are larger than a pure linear (duration-only) approximation.

**Q10:** Which instruments can exhibit negative convexity?
**A:** Callable bonds and mortgage-backed securities at certain rate levels.

**Q11:** What does negative convexity do to rally performance?
**A:** It reduces price gains when yields fall (price-yield curve flattens).

**Q12:** Why does a callable bond's price curve become concave?
**A:** The call option is convex; subtracting it from the bond can make the callable bond price curve concave, yielding negative convexity.

**Q13:** What is "dollar convexity"?
**A:** $PC = d^2P/dy^2$, the second derivative in price units.

**Q14:** What is the convexity P&L term in bp notation?
**A:** $\Delta P_{\text{conv}} \approx \frac{1}{2}\left(\frac{PC}{10{,}000^2}\right)(\Delta y_{\text{bp}})^2$.

**Q15:** Why does DV01 hedging fail for big moves?
**A:** DV01 hedging only neutralizes the first derivative; second-order curvature creates residual P&L for large $|\Delta y|$.

**Q16:** What is the barbell vs bullet convexity result (duration-matched)?
**A:** $\bar{R}_B - \bar{R}_P = \frac{1}{2}(C_B - C_P)(\Delta y)^2$.

**Q17:** What does "DV01 is defined for a specific bump" mean?
**A:** DV01 depends on the assumed shock (yield bump, parallel shift, etc.); different shock definitions give different DV01s.

**Q18:** What unrealistic assumption underlies yield-based DV01?
**A:** The yield curve shifts in parallel and the bond is exposed only to the curve level.

**Q19:** How does convexity relate to volatility exposure in expected return?
**A:** Expected return can include $\frac{1}{2}C\sigma^2 dt$, so more yield volatility increases expected return for positive convexity.

**Q20:** How can you estimate convexity numerically?
**A:** Reprice at $y - \Delta y$, $y$, $y + \Delta y$ and compute a central second-derivative estimate; then normalize by price.

**Q21:** What does it mean that convexity is "long volatility"?
**A:** A convex price function implies $\mathbb{E}[P]$ rises with variance of yields, holding the mean fixed.

**Q22:** In practice, why might convexity comparisons be tricky?
**A:** Convexity depends on yield convention, curve bump definition, and can change with rate level (especially with options).

**Q23:** Why can a DV01-neutral portfolio still gain for both +100bp and -100bp moves?
**A:** If it has positive net convexity, the second-order term is positive for both directions.

**Q24:** What is the key practical message of convexity for hedging?
**A:** "DV01-neutral" hedges are local; convexity mismatch creates hedge error for large moves (and changing DV01).

**Q25:** What price should you use for duration/convexity: clean or dirty?
**A:** Use full/dirty (invoice) price when accrued interest is nonzero.

---

## Mini Problem Set (14 Questions)

### Problem 1
Compute DV01 for a 5-year zero-coupon bond (face 100) at $y = 4\%$ under semiannual compounding using the derivative formula.

**Sketch:** Use $P = 100/(1+y/2)^{2T}$, $D = T/(1+y/2)$, then $\text{DV01} = PD/10{,}000$.

---

### Problem 2
For the same bond in (1), compute convexity analytically and estimate it via ±1 bp finite differences.

**Sketch:** Use $C = T(T+0.5)/(1+y/2)^2$ and compare to central difference.

---

### Problem 3
A bond has $P = 101.25$, $\text{DV01} = 0.075$ per 100. Compute modified duration.

**Sketch:** $D = 10{,}000 \cdot \text{DV01}/P$.

---

### Problem 4
A bond has $P = 98.50$, $D = 6.2$, $C = 85$. Estimate $\Delta P$ for a +60 bp move using duration-only and duration+convexity.

**Sketch:** Convert 60 bp to $\Delta y = 0.006$; apply $\Delta P \approx P(-D\Delta y + \frac{1}{2}C\Delta y^2)$.

---

### Problem 5
Show that the convexity term is the same for +50 bp and -50 bp moves (holding $C$ constant).

**Sketch:** Convexity term depends on $(\Delta y)^2$.

---

### Problem 6
Two bonds have the same DV01 today but bond X has higher convexity. Which bond benefits more from rate volatility (qualitatively)?

**Sketch:** Higher $C$ implies higher expected return contribution $\frac{1}{2}C\sigma^2 dt$.

---

### Problem 7
Using the barbell-bullet identity, explain why a duration-matched higher convexity portfolio outperforms for sufficiently large $|\Delta y|$.

**Sketch:** $\bar{R}_B - \bar{R}_P = \frac{1}{2}(C_B - C_P)(\Delta y)^2$; sign depends on $C_B - C_P$.

---

### Problem 8
Construct a DV01-neutral portfolio using a 2-year and 10-year par bond at $y = 3\%$. Compute net convexity and estimate P&L for ±100 bp.

---

### Problem 9
For a 10-year par bond, compute how DV01 changes when yield increases from 2% to 5%. Interpret in terms of convexity.

---

### Problem 10
A callable bond has negative convexity at current yields. Explain (without full option modeling) why a DV01 hedge might become unstable after a rally.

---

### Problem 11
Define "dollar convexity per bp$^2$" and show how it appears in the convexity P&L term.

---

### Problem 12
Explain why yield-based DV01 can misstate risk when the curve twist is concentrated in the front end.

---

### Problem 13
Given a portfolio with DV01 = 0 and positive convexity, what is the sign of P&L for a large parallel shift (either direction)? Explain.

---

### Problem 14
Propose a practical bump size selection rule for estimating convexity and explain trade-offs (noise vs bias).

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

| Fact | Source |
|------|--------|
| Convexity definition $C = \frac{1}{P}\frac{d^2P}{dy^2}$ | Tuckman Ch 5-6; Hull Ch 4 |
| Second-order Taylor expansion for bond prices | Tuckman Ch 5 |
| DV01 discrete formula $(P_- - P_+)/2$ | Tuckman Ch 5-6 |
| DV01-duration relationship | Tuckman Ch 5 |
| Zero-coupon convexity formula $C = T(T+0.5)/(1+y/2)^2$ | Tuckman Ch 5 |
| Coupon bond convexity formula (PV-weighted) | Tuckman Ch 5 |
| Barbell vs bullet convexity dominance | Tuckman Ch 5 |
| Expected return convexity term $\frac{1}{2}C\sigma^2 dt$ | Tuckman Ch 5 |
| Callable bond negative convexity example (5s of Feb 15, 2011) | Tuckman callable example |

### (B) Reasoned Inference — Note Derivation Logic

| Inference | Derivation |
|-----------|------------|
| All worked example numerical calculations | Algebraic application of source-backed formulas |
| Unit conversion formulas (bp ↔ decimal) | Dimensional analysis from definitions |
| Symmetry of convexity term | Follows from $(\Delta y)^2$ dependence |
| "Long convexity = long volatility" interpretation | Jensen's inequality applied to convex price function |
| Duration-only error scaling as $(\Delta y)^2$ | Taylor expansion remainder analysis |

### (C) Speculation — Flag Uncertainties

None in this chapter. All content is either directly source-backed or algebraically derived from source-backed formulas.

---

*Chapter 13 — Fixed Income: Practice and Theory*
