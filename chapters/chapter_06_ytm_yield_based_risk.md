# Chapter 6: Yield-to-Maturity (YTM) and Yield-Based Risk

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- YTM is the single rate that, when used to discount a bond's promised cash flows, reproduces the bond's market price (Tuckman Ch 3)
- Yield is not automatically a good measure of relative value or realized return-to-maturity; higher yield does not necessarily mean "better value" (Tuckman)
- Bond market convention: cash paid = quoted (clean/flat) price + accrued interest (Tuckman)
- Synonyms: full/dirty price vs quoted/flat (clean) price (Tuckman)
- Yields are typically quoted with semiannual compounding ("bond-style" convention) (Tuckman)
- For day-based discounting under semiannual yield: discount by $(1 + y/2)^{d/182.5}$ (Tuckman)
- YTM is solved by trial-and-error or numerical methods (Tuckman)
- DV01 general definition: $\text{DV01} = -\frac{\Delta P}{10{,}000 \cdot \Delta r}$ (Tuckman)
- Modified duration: $D_{\text{mod}} = -\frac{1}{P}\frac{dP}{dy}$ (Tuckman)
- Yield convexity: $C_y = \frac{1}{P}\frac{d^2P}{dy^2}$ (Tuckman)
- Coupon-bond price as portfolio of zeros: $P = \sum \text{CF}(t_i) \cdot d(t_i)$ (Tuckman)
- Parallel-yield-shift assumptions are "not particularly good," can be internally inconsistent, and apply only to fixed cash flows (Tuckman)
- There is generally no static hedge of one bond by two others that works under all yield curve moves (Tuckman)
- If a bond's yield remains unchanged over a short period, realized total return equals its yield (Tuckman)
- Holding to maturity will not necessarily earn the initial yield (reinvestment and changing yields matter) (Tuckman)

### (B) Reasoned Inference (Derived from A)

- YTM compresses the whole term structure into one "average discount rate" that reproduces the price—it is a quoting device
- Clean price strips out the "mechanical" coupon accrual since the last coupon date, so quoted prices do not jump simply because time passes
- Yield calculations should be based on the dirty price, because discounting promised cash flows should equal the cash paid
- Since $C_y > 0$ for plain vanilla fixed-rate bonds, losses from rate rises are partially cushioned (convexity benefit)
- A yield-DV01 hedge implicitly assumes bond yields change in lockstep; when the curve twists, the hedge P&L becomes exposed to the unhedged shape component
- For near-par bonds, yield tends to be near coupon rate (under the same compounding convention)
- For a zero-coupon bond under matched conventions, YTM equals the corresponding spot rate

### (C) Speculation (Clearly Labeled; Minimal)

- "BEY" (bond-equivalent yield) as a market label is not explicitly named in the cited passages; I'm not sure which desks/markets label the semiannual-quoted bond yield as "BEY" without additional context (market, instrument type, and quoting standard)

---

## Conventions & Notation

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $F$ | Face value (par), default 100 |
| $c$ | Annual coupon rate (in currency per 100 notional per year); coupon per half-year is $c/2$ |
| $T_1, \ldots, T_N$ | Scheduled coupon payment dates; maturity is $T_N$ |
| $P_{\text{clean}}$ | Quoted (flat/clean) price per 100 |
| $\text{AI}$ | Accrued interest per 100 |
| $P_{\text{dirty}}$ | Full (dirty) price per 100: $P_{\text{dirty}} = P_{\text{clean}} + \text{AI}$ |
| $y$ | Yield-to-maturity (annualized), quoted with semiannual compounding |
| $r = 1 + y/2$ | Per-half-year gross discount factor base under YTM convention |
| $d$ | Number of days from settlement to a cash-flow date |
| $a_i = d_i / 182.5$ | Exponent for day-based discounting under semiannual convention |
| $P(0,t)$ or $d(t)$ | Discount factor to maturity $t$ from a curve |
| $\text{DV01}_y$ | Yield DV01: price change per 1 bp move in YTM |
| $D_{\text{mod}}$ | Modified duration in yield space |
| $C_y$ | Yield convexity |

### Key Identity

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + \text{AI}}$$

### Accrual Fraction (Actual/Actual-style)

$$\text{accrual fraction} = \frac{\text{actual days since last coupon}}{\text{actual days in coupon period}}$$

(Market inclusion/exclusion rules vary; verify for your desk.)

### Defaults Used in Examples

| Convention | Default |
|------------|---------|
| Compounding | Semiannual (bond-style) |
| Day-exponent | $d / 182.5$ for cash flows due in $d$ days |
| Face value | $F = 100$ (prices per 100 notional) |
| Basis point | 1 bp = 0.01% = 0.0001 in decimal |

### Curve Preview Convention

When using a curve of discount factors, value is $\sum \text{CF}_i \cdot P(0,t_i)$. For a parallel bump in continuously compounded spot rates $\hat{r}(t)$:
- $d(t) = \exp(-\hat{r}(t) \cdot t)$
- Bumped: $d_{\Delta}(t) = d(t) \exp(-\Delta t)$ for bump $\Delta$

---

## Setup

**Yield-to-maturity is a single-rate summary statistic:** it is the one rate that, when used to discount a bond's promised cash flows, reproduces the bond's market price. This is convenient for quoting and quick comparisons, but it can also be misleading: Tuckman emphasizes that yield is not automatically a good measure of relative value or realized return-to-maturity, and that higher yield does not necessarily mean "better value."

**This chapter has two aims:**

1. Define YTM precisely (including clean/dirty pricing and compounding/day-count conventions)
2. Build yield-based risk measures (DV01, modified duration, convexity) and show why they can diverge from curve-based risk when the term structure moves in non-parallel ways

---

## Core Concepts

### 1) Yield-to-Maturity (YTM) as an Internal Rate of Return (IRR)

**Formal Definition:**

YTM is the single rate such that discounting the bond's promised cash flows at that rate reproduces the bond's market price. For a standard $T$-year bond with semiannual coupons, Tuckman writes the price–yield relationship as:

$$\boxed{P(T) = \sum_{t=1}^{2T} \frac{c/2}{(1+y/2)^t} + \frac{F}{(1+y/2)^{2T}}}$$

**Intuition:**

YTM compresses the whole term structure into one "average discount rate" that reproduces the price. It is therefore a quoting device (price $\leftrightarrow$ yield is easy to invert).

**Trading/Risk Practice:**

- Bonds are frequently quoted by yield rather than price because it is easy to move between them
- Desk shorthand: "Bond yields $y$" often means "the $y$ that reprices this bond under the street compounding rule"

---

### 2) Clean (Flat) vs Dirty (Full) Price and Accrued Interest

**Formal Definition:**

Bond market convention: cash paid = quoted price + accrued interest:

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + \text{AI}}$$

Tuckman notes the synonyms: full/dirty vs quoted/flat (clean).

**Intuition:**

Clean price strips out the "mechanical" coupon accrual since the last coupon date, so quoted prices do not jump simply because time passes.

**Trading/Risk Practice:**

- Trading screens typically show clean prices for coupon bonds; settlement cash uses dirty price
- Yield calculations should be based on the dirty price, because discounting promised cash flows should equal the cash paid (full price). Tuckman's equation $P + \text{AI} =$ PV of future cash flows makes this explicit

---

### 3) What YTM Summarizes vs What It Hides

**Formal Definition (what it summarizes):**

YTM summarizes the level of discounting that matches price, replacing a term structure of discount rates by one rate.

**What it hides (and why that matters):**

- **Term structure:** Spot rates differ by maturity, and each cash flow should be discounted at a rate appropriate for that date. A single YTM cannot tell you whether the bond is "cheap at the front" and "rich at the back"
- **Realized return assumptions:** Tuckman explicitly warns that a bond bought at a yield and held to maturity will not necessarily earn that initial yield
- **A "safe" short-horizon interpretation:** If a bond's yield remains unchanged over a short period, realized total return equals its yield (the book highlights this as an appealing interpretation)

**Trading/Risk Practice:**

- Relative-value screens often rank by yield; Tuckman cautions this can be misleading
- Risk hedging based on yield assumes a particular mapping from "the world" (a curve move) to "the bond's yield move," which may fail

---

### 4) Yield-Based DV01, Modified Duration, and Convexity

**Formal Definitions:**

**General DV01 definition** (for a pricing factor $r$):

$$\boxed{\text{DV01} = -\frac{\Delta P}{10{,}000 \cdot \Delta r}}$$

In yield space, take $r \equiv y$ (a single yield factor).

**Modified duration (yield duration):**

$$\boxed{D_{\text{mod}}(y) = -\frac{1}{P(y)}\frac{dP}{dy}}$$

**Yield convexity:**

$$\boxed{C_y = \frac{1}{P}\frac{d^2P}{dy^2}}$$

**Intuition:**

- **DV01 / duration:** First-order slope of the price–yield curve
- **Convexity:** Curvature; it explains why price increases from yield decreases are larger than price decreases from equal-sized yield increases

**Trading/Risk Practice:**

DV01 matching is a common "quick hedge" in yield space, but it assumes yield shifts behave in a simple way (often parallel shifts).

---

### 5) Curve-Based Risk (Preview): "Curve DV01" vs "Yield DV01"

**Formal Definition (Curve PV):**

Because there is a term structure of spot rates, coupon-bond price is naturally written as a portfolio of zeros:

$$P = \sum \text{CF}(t_i) \cdot d(t_i)$$

with discount factors $d(t)$.

**Curve DV01 (one precise version for this chapter):**

Fix a bump rule, e.g., a parallel 1 bp bump to continuously compounded spot rates $\hat{r}(t)$. Since $d(t) = \exp(-\hat{r}(t) \cdot t)$, a bump $\Delta = 0.0001$ gives:

$$d_{\Delta}(t) = d(t) \exp(-\Delta t)$$

Define:

$$\text{DV01}_{\text{curve}} \equiv P_{\text{base}} - P_{\Delta}$$

(per 100 notional) for a +1bp bump.

**When they differ materially:**

- Yield DV01 treats the bond as if its price depends on one number $y$. Curve DV01 treats the price as depending on the entire curve $t \mapsto d(t)$
- Non-parallel curve moves (twists) break yield-hedge logic: Tuckman stresses that parallel-yield-shift assumptions are "not particularly good," can be internally inconsistent, and apply only to fixed cash flows. The multi-factor reality is previewed in the next chapter: there is generally no static hedge of one bond by two others that works under all yield curve moves

---

## Math and Derivations

### 1) YTM as the Solution to a Nonlinear Pricing Equation

For a standard semiannual coupon bond:

$$P(y) = \sum_{k=1}^{N-1} \frac{c/2}{(1+y/2)^k} + \frac{c/2 + F}{(1+y/2)^N}$$

which is the same structure as Tuckman's equation (3.2).

**Assumption:** Fixed promised cash flows; no embedded option, no default, no path-dependent cash flows.

**Solving for $y$:** Given $P$, solve $f(y) = P(y) - P_{\text{dirty}} = 0$. Tuckman notes this is done by trial-and-error or numerical methods.

**Unit/sanity checks:**
- $y$ is in annual units; $1 + y/2$ is a per-half-year gross rate
- If $y$ increases, $P(y)$ decreases (monotonicity check)

---

### 2) Yield Discounting with Day-Count Exponents (Semiannual-Quoted Yield)

When a cash flow is due in $d$ days, the semiannual yield convention discounts by $(1 + y/2)^{d/182.5}$. For irregular spacing, apply this exponent cash-flow-by-cash-flow:

$$\boxed{P(y) = \sum_i \frac{\text{CF}_i}{(1 + y/2)^{d_i/182.5}}}$$

**Assumption:** We are applying the book's semiannual-compounding day-exponent convention cash-flow-by-cash-flow.

---

### 3) Yield DV01 and Modified Duration

Start from the definition of modified duration:

$$D_{\text{mod}}(y) = -\frac{1}{P}\frac{dP}{dy}$$

For discrete cash flows $\text{CF}_i$ discounted as $P(y) = \sum_i \text{CF}_i \cdot r^{-a_i}$ with $r = 1 + y/2$ and exponent $a_i$ (e.g., $a_i = k$ for regular periods or $a_i = d_i/182.5$ for day-based discounting):

$$\frac{d}{dy} r^{-a_i} = -\frac{a_i}{2} r^{-a_i - 1}$$

So:

$$\frac{dP}{dy} = \sum_i \text{CF}_i \left(-\frac{a_i}{2}\right) r^{-a_i - 1}$$

**DV01 in yield space:**

From the general DV01 definition (divide $\frac{dP}{dy}$ by $-10{,}000$):

$$\boxed{\text{DV01}_y \approx -\frac{1}{10{,}000}\frac{dP}{dy}}$$

Equivalently, using $D_{\text{mod}}$:

$$\boxed{\text{DV01}_y \approx \frac{P \cdot D_{\text{mod}}}{10{,}000}}$$

which matches the book's DV01–duration link.

**Unit checks:**
- $dP/dy$ has units "price per 1.00 of yield" (e.g., per 100% yield)
- Divide by 10,000 to get "price per 1 bp"
- $D_{\text{mod}}$ has units of years (because it's a weighted-average time scale)

---

### 4) Yield Convexity and Second-Order Price Approximation

Tuckman defines yield-based convexity as:

$$C_y = \frac{1}{P}\frac{d^2P}{dy^2}$$

For a standard semiannual bond, the second derivative yields a closed-form weighted sum (equations (6.34)–(6.35)).

**A second-order Taylor approximation in yield space** is the usual "duration + convexity" heuristic:

$$\boxed{\Delta P \approx -P \cdot D_{\text{mod}} \cdot \Delta y + \frac{1}{2} P \cdot C_y \cdot (\Delta y)^2}$$

**Sanity checks:**
- For small $\Delta y$, the convexity term is tiny vs duration
- Since $C_y > 0$ for plain vanilla fixed-rate bonds, losses from rate rises are partially cushioned (convexity benefit)

---

## Measurement & Risk

### 1) Yield-Duration / Modified Duration as Yield Sensitivity

**Definition:** $D_{\text{mod}} = -\frac{1}{P}\frac{dP}{dy}$

**Interpretation:** Approximate percentage price change per unit change in yield:

$$\frac{\Delta P}{P} \approx -D_{\text{mod}} \cdot \Delta y$$

**Unit check:** If $D_{\text{mod}} = 4$ and $\Delta y = 10$ bp $= 0.001$, then $\Delta P / P \approx -0.4\%$.

---

### 2) Yield DV01 (Price Change per 1bp in YTM) and How It Is Computed

Two common computations (both consistent with the DV01 concept):

**Finite-difference bump (robust):**

$$\text{DV01}_y \approx \frac{P(y - \delta) - P(y + \delta)}{2}, \quad \delta = 0.0001$$

**Duration-based approximation:**

$$\text{DV01}_y \approx \frac{P \cdot D_{\text{mod}}}{10{,}000}$$

---

### 3) Convexity in Yield Space (Yield Convexity)

**Definition:** $C_y = \frac{1}{P}\frac{d^2P}{dy^2}$

**Operational estimate:** With a larger bump such as $\Delta y = 50$ bp:

$$C_y \approx \frac{P(y - \Delta) + P(y + \Delta) - 2P(y)}{P(y) \cdot \Delta^2}$$

---

### 4) Yield-Based DV01 vs Curve-Based DV01: Precise Definitions and When They Differ

**Yield DV01 (single-factor):**
Sensitivity of the bond price to a bump in its own YTM $y$, holding cash flows fixed.

**Curve DV01 (multi-curve object, preview):**
Sensitivity of PV $\sum \text{CF}_i \cdot P(0,t_i)$ to a bump in the discount curve, i.e., the discount factors or the spot rates that generate them. Under continuously compounded spot rates, $d(t) = \exp(-\hat{r}(t) \cdot t)$ makes it easy to define a parallel bump.

**Why they differ materially:**
- Yield DV01 assumes the world moves through a single scalar $y$. Tuckman emphasizes that the "measure of sensitivity based on parallel yield shifts" is "not particularly good," may be internally inconsistent, and is limited to fixed cash flows
- Curve DV01 accounts for the fact that different cash flows depend on different points on the curve

**Hedge failure intuition:** As soon as curve shape changes matter, there is no universal static hedge using only yield DV01 (multi-factor risk).

---

### 5) P&L Intuition: Why Yield Hedges Can Fail When Curve Shape Moves

A yield-DV01 hedge typically says: "match DV01 of bond A with bond B."

This implicitly assumes bond A's yield and bond B's yield change in lockstep for the relevant market move. When the curve twists, the two yields can respond differently, and the hedge P&L becomes exposed to the unhedged shape component (a preview of why multi-factor curve risk matters).

---

## Worked Examples

All prices are per 100 notional. Basis points: 1 bp = 0.01% = 0.0001 in decimal.

---

### Example A: Compute YTM — Clean → Accrued → Dirty → Solve YTM as IRR

**Bond:**
- U.S.-Treasury-style semiannual coupon schedule: coupons on Feb 15 and Aug 15
- Coupon rate: $c = 6\%$ $\Rightarrow$ coupon cash flow $= c/2 = 3.00$
- Maturity: Feb 15, 2031
- Settlement: Jan 12, 2026
- Quoted clean price: $P_{\text{clean}} = 101.25$

**Step 1 — Accrued interest:**

Use Actual/Actual-style fraction:
- Last coupon date: Aug 15, 2025
- Next coupon date: Feb 15, 2026
- Days in coupon period: 184 (Aug 15 → Feb 15)
- Days accrued: 150 (Aug 15 → Jan 12)

Accrual fraction:

$$\alpha = \frac{150}{184} = 0.8152173913$$

Coupon per period is 3.00, so:

$$\text{AI} = 3.00 \times \alpha = 3 \cdot \frac{150}{184} = 2.4456521739$$

**Step 2 — Dirty price:**

Full/dirty price equals clean plus accrued interest:

$$P_{\text{dirty}} = 101.25 + 2.4456521739 = 103.6956521739$$

**Step 3 — YTM equation (day-exponent discounting):**

Under the semiannual yield convention, discount a cash flow due in $d$ days by $(1 + y/2)^{d/182.5}$. Let $r = 1 + y/2$.

Compute days from settlement to each coupon date:

| Payment date | Cash flow | Days $d_i$ | Exponent $a_i = d_i/182.5$ |
|--------------|-----------|------------|---------------------------|
| Feb 15, 2026 | 3 | 34 | 0.186301 |
| Aug 15, 2026 | 3 | 215 | 1.178082 |
| Feb 15, 2027 | 3 | 399 | 2.186301 |
| Aug 15, 2027 | 3 | 580 | 3.178082 |
| Feb 15, 2028 | 3 | 764 | 4.186301 |
| Aug 15, 2028 | 3 | 946 | 5.183562 |
| Feb 15, 2029 | 3 | 1130 | 6.191781 |
| Aug 15, 2029 | 3 | 1311 | 7.183562 |
| Feb 15, 2030 | 3 | 1495 | 8.191781 |
| Aug 15, 2030 | 3 | 1676 | 9.183562 |
| Feb 15, 2031 | 103 | 1860 | 10.191781 |

Solve for $y$ in:

$$\sum_{i=1}^{10} \frac{3}{r^{a_i}} + \frac{103}{r^{a_{11}}} = 103.6956521739$$

**Step 4 — Solution (by numerical root finding):**

A yield of $y = 5.709\%$ (semiannual compounding) gives $r = 1 + 0.05709/2 = 1.028545$ and PV $\approx 103.6944$, matching the dirty price to ~0.0013.

**Answer:**

$$\boxed{\text{YTM} \approx 5.709\% \text{ (annualized, semiannual compounding)}}$$

with the stated day-exponent convention.

---

### Example B: Root Finding — One Newton–Raphson Iteration + Sanity Checks

We solve $f(y) = P(y) - P_{\text{dirty}} = 0$, where $P(y) = \sum_i \text{CF}_i \cdot r^{-a_i}$, $r = 1 + y/2$.

**Inputs:**
- From Example A: $P_{\text{dirty}} = 103.6956521739$
- Start guess: $y_0 = 6.00\%$ $\Rightarrow$ $r_0 = 1.03$
- At $y_0 = 6\%$, computed PV: $P(y_0) = 102.4231821769$

So:

$$f(y_0) = 102.4231821769 - 103.6956521739 = -1.2724699970$$

**Derivative:**

$$\frac{dP}{dy} = \sum_i \text{CF}_i \left(-\frac{a_i}{2}\right) r^{-a_i - 1}$$

Equivalently, using $r^{-a_i-1} = r^{-a_i}/r$:

$$\frac{dP}{dy} = -\frac{1}{2r} \sum_i a_i \cdot \text{CF}_i \cdot r^{-a_i}$$

At $y_0 = 6\%$, the computed derivative is:

$$f'(y_0) = \frac{dP}{dy} \approx -433.5549515$$

(Units: "price per 1.00 of yield"; so per 1 bp it's about $-0.04336$.)

**Newton step:**

$$y_1 = y_0 - \frac{f(y_0)}{f'(y_0)} = 0.06 - \frac{-1.2724699970}{-433.5549515} = 0.0570650318$$

So $y_1 \approx 5.7065\%$.

**Sanity checks:**
- Since $f(y_0) < 0$ (PV too low), we need a lower yield. The step indeed moves from 6.00% down to 5.7065%
- Reprice check at $y_1$: PV $\approx 103.7054$, error $\approx +0.0097$, far smaller than $-1.27$ at the initial guess

**One-iteration result:**

$$y_0 = 6.00\% \rightarrow y_1 \approx 5.7065\% \text{ (much closer to the true 5.709\%)}$$

---

### Example C: Price–Yield Curve — Price at YTM ± 50bp and Convexity Link

Use the Example A bond and its solved yield $y^* = 5.709\%$ with dirty price $P^* = 103.695652$.

**Compute prices at:**
- $y^- = 5.209\% = y^* - 50$ bp
- $y^+ = 6.209\% = y^* + 50$ bp

Using the same day-exponent discounting:
- $P(y^-) \approx 105.924029$
- $P(y^*) \approx 103.695652$
- $P(y^+) \approx 101.521940$

**Nonlinearity (convexity) observation:**
- Up-move loss: $P(y^+) - P(y^*) \approx -2.173712$
- Down-move gain: $P(y^-) - P(y^*) \approx +2.228376$

The magnitudes differ, consistent with positive convexity for plain-vanilla bonds.

**Estimate yield convexity from these points:**

With $\Delta = 0.005$ (50 bp):

$$C_y \approx \frac{P(y^-) + P(y^+) - 2P(y^*)}{P(y^*) \cdot \Delta^2} = \frac{105.924029 + 101.521940 - 2 \cdot 103.695652}{103.695652 \cdot 0.005^2} \approx 21.08634$$

---

### Example D: Yield DV01 — Finite Difference ±1bp vs Modified-Duration Approximation

Continue with Example A bond at $y^* = 5.709\%$, $P^* = 103.695652$.

**Step 1 — Finite-difference DV01:**

Compute prices at $y^* \pm 1$ bp:
- $P(y^* - 1 \text{ bp}) \approx 103.738427$
- $P(y^* + 1 \text{ bp}) \approx 103.650390$

Central-difference DV01:

$$\text{DV01}_y \approx \frac{P(y^* - 1 \text{ bp}) - P(y^* + 1 \text{ bp})}{2} = \frac{103.738427 - 103.650390}{2} \approx 0.0440186$$

So:

$$\boxed{\text{DV01}_y \approx 0.04402 \text{ per 100 notional per 1 bp in YTM}}$$

**Step 2 — Modified-duration approximation:**

Infer modified duration via the DV01–duration link:

$$D_{\text{mod}} \approx \frac{\text{DV01}_y \cdot 10{,}000}{P^*} = \frac{0.0440186 \cdot 10{,}000}{103.695652} \approx 4.24498$$

Then the duration approximation for a +1bp move is:

$$\Delta P \approx -P^* \cdot D_{\text{mod}} \cdot \Delta y = -103.695652 \cdot 4.24498 \cdot 0.0001 \approx -0.0440186$$

**Compare to actual one-sided bump:**
- Actual +1bp change: $P(y^* + 1 \text{ bp}) - P^* \approx -0.0452620$
- Duration approx: $-0.0440186$
- Difference is convexity (small but visible even at 1 bp)

---

### Example E: Yield DV01 vs Curve DV01 — Compute Both from a Discount Factor Curve

We intentionally keep this as a preview: curve risk is inherently multi-dimensional.

**Bond:**
- Maturity: 2 years, semiannual coupons
- Coupon rate: $4\%$ $\Rightarrow$ 2 per half-year
- Cash flows: 2, 2, 2, 102 at $t = 0.5, 1.0, 1.5, 2.0$

**Given discount-factor curve input $P(0,t)$:**

| $t$ (years) | $P(0,t)$ |
|-------------|----------|
| 0.5 | 0.9850 |
| 1.0 | 0.9700 |
| 1.5 | 0.9530 |
| 2.0 | 0.9350 |

This PV-by-discount-factor approach is consistent with writing coupon-bond prices as portfolios of zeros.

**(i) Bond PV from the curve:**

$$P_{\text{curve}} = 2(0.9850) + 2(0.9700) + 2(0.9530) + 102(0.9350)$$
$$= 1.9700 + 1.9400 + 1.9060 + 95.3700 = 101.1860$$

So $\boxed{P_{\text{curve}} = 101.1860}$.

**(ii) Curve DV01 by bumping the curve (parallel bump to continuous spot rates):**

Use $d(t) = \exp(-\hat{r}(t) \cdot t)$ and define a parallel bump $\Delta = 0.0001$:

$$d_{\Delta}(t) = d(t) \exp(-\Delta t)$$

Multipliers $\exp(-\Delta t)$ for $\Delta = 0.0001$:

| $t$ | $\exp(-0.0001 \cdot t)$ |
|-----|-------------------------|
| 0.5 | 0.9999500013 |
| 1.0 | 0.9999000050 |
| 1.5 | 0.9998500112 |
| 2.0 | 0.9998000200 |

Compute bumped PV:

$$P_{\Delta} = 2(0.9850)(0.9999500013) + 2(0.9700)(0.9999000050)$$
$$+ 2(0.9530)(0.9998500112) + 102(0.9350)(0.9998000200)$$
$$= 101.1663495409$$

So curve DV01 (for +1bp bump in $\hat{r}(t)$) is:

$$\text{DV01}_{\text{curve}} = P_{\text{curve}} - P_{\Delta} = 101.1860 - 101.1663495409 = 0.0196504591$$

$$\boxed{\text{DV01}_{\text{curve}} \approx 0.01965 \text{ per 100 notional}}$$

**(iii) Yield DV01 for the same bond (via the YTM that matches price):**

Solve $y$ from the standard semiannual YTM equation:

$$101.186 \approx \frac{2}{(1+y/2)^1} + \frac{2}{(1+y/2)^2} + \frac{2}{(1+y/2)^3} + \frac{102}{(1+y/2)^4}$$

Numerically, $r = 1 + y/2 \approx 1.01691$ $\Rightarrow$ $y \approx 3.382\%$.

Now bump yield by +1bp:
- $P(y) \approx 101.1854643$
- $P(y + 1 \text{ bp}) \approx 101.1661399$

Yield DV01:

$$\text{DV01}_y \approx P(y) - P(y + 1 \text{ bp}) = 101.1854643 - 101.1661399 = 0.0193244162$$

**Comparison:**
- $\text{DV01}_y \approx 0.01932$
- $\text{DV01}_{\text{curve}} \approx 0.01965$

**Why they differ (conceptually):**
- Curve DV01 bumps each maturity point in a way consistent with $d(t) = \exp(-\hat{r}(t) \cdot t)$, which weights longer maturities more by $t$
- Yield DV01 uses a single rate $y$ as a summary statistic; it cannot replicate all curve shifts exactly

---

### Example F: What YTM Hides — Same YTM, Different DV01/Convexity

Construct two 5-year semiannual coupon bonds priced at the same yield $y = 5\%$ (so both have YTM = 5% by construction).

Let $r = 1 + y/2 = 1.025$, $N = 10$ half-year periods, $r^{-10} = 0.7811984017$, and the annuity factor:

$$A = \sum_{k=1}^{10} r^{-k} = \frac{1 - r^{-10}}{r - 1} = 8.752063931$$

**Bond A (high coupon):**
- Coupon rate $8\%$ $\Rightarrow$ $C = 4$ per half-year
- Price:

$$P_A = 4A + 100 \cdot r^{-10} = 4(8.752063931) + 100(0.7811984017) = 113.1280958965$$

**Bond B (low coupon):**
- Coupon rate $2\%$ $\Rightarrow$ $C = 1$ per half-year
- Price:

$$P_B = 1 \cdot A + 100 \cdot r^{-10} = 86.8719041035$$

Both bonds have the same YTM $y = 5\%$ because we priced them at that yield (YTM is the single rate that reproduces price).

**Compare yield-based risk (modified duration, DV01, convexity):**

Using $D_{\text{mod}} = -\frac{1}{P}\frac{dP}{dy}$ and $\text{DV01} \approx P \cdot D_{\text{mod}}/10{,}000$:

| Bond | $D_{\text{mod}}$ | $\text{DV01}$ | $C_y$ |
|------|------------------|---------------|-------|
| A (8% coupon) | $\approx 4.1680$ | $\approx 0.04715$ | $\approx 21.1363$ |
| B (2% coupon) | $\approx 4.6469$ | $\approx 0.04037$ | $\approx 24.5345$ |

**Takeaway:**

Even with identical YTM, the timing of cash flows differs (coupon structure), so duration/convexity differ. YTM alone "hides" this risk structure.

---

## Practical Notes

### Yield Quoting Conventions (and Where Conventions Differ)

- Tuckman distinguishes **money market quoting** (simple interest with a day-count, e.g., Actual/360) from **bond quoting** (semiannual compounding)
- Continuously compounded spot rates $\hat{r}(t)$ relate to discount factors via $d(t) = \exp(-\hat{r}(t) \cdot t)$

**Bond-equivalent yield (BEY) vs effective annual yield vs continuous (desk note):**
- The book text here uses semiannual compounding for bond-style yields
- "BEY" as a market label is not explicitly named in the cited passages; **I'm not sure** which desks/markets in your workflow label the semiannual-quoted bond yield as "BEY" without additional context (market, instrument type, and quoting standard)

### Common Pitfalls

1. **Mixing clean and dirty price when solving YTM:**
   Settlement cash is full price, and $P_{\text{dirty}} = P_{\text{clean}} + \text{AI}$; using the clean price in the PV equation misstates YTM

2. **Day-count mismatch in coupon accrual vs yield compounding:**
   Discounting exponents can be day-based under bond yield conventions $(1 + y/2)^{d/182.5}$; mixing conventions changes computed yields and risk

3. **Using yield DV01 as if it were curve DV01:**
   Yield-shift sensitivity is explicitly built on parallel yield shifts and has known weaknesses

4. **Interpreting YTM as an expected return:**
   Tuckman warns that holding to maturity will not necessarily earn the initial yield (reinvestment and changing yields matter)

5. **Optionality / credit / liquidity:**
   Yield-based measures assume fixed promised cash flows; embedded options or credit/liquidity effects change both "cash flows" and discounting (outside this chapter's scope; treat as a warning flag)

### Verification Tests (Quick Diagnostics)

- **Monotonicity:** Bond price decreases as yield increases (for fixed cash flows)
- **Bounds near par:** For near-par bonds, yield tends to be near coupon rate (under the same compounding convention)
- **Zero-coupon limit:** For a zero-coupon bond under matched conventions, the YTM equals the corresponding spot rate convention for that maturity (since there is only one cash flow). (Derived inference from discounting definitions.)
- **Reprice check:** Plug computed YTM back into the PV formula and recover $P_{\text{dirty}}$ (Example A demonstrates this)

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **YTM is the single rate** that reproduces a bond's market price when used to discount its cash flows
2. **Clean/flat price plus accrued interest equals full/dirty price**; YTM should be solved off the dirty price
3. **Bonds are commonly quoted** in semiannually compounded yield terms
4. **Day-count conventions** can enter yield discounting via exponents like $d/182.5$
5. **YTM is a convenient summary**, but it can be misleading for relative value and realized return
6. **A defensible short-horizon interpretation:** If yield is unchanged over a short period, realized return equals yield
7. **Yield DV01** is the price change per 1 bp in YTM; operationally computed via finite differences or via duration
8. **Modified duration** is the yield sensitivity $D_{\text{mod}} = -\frac{1}{P}\frac{dP}{dy}$
9. **Convexity** in yield space is $C_y = \frac{1}{P}\frac{d^2P}{dy^2}$ and explains price–yield nonlinearity
10. **Yield-based hedges can fail** when curve shape moves matter: parallel-yield-shift assumptions are weak and multi-factor curve moves require richer risk measures

---

### Cheat Sheet (Key Formulas + Meaning + Units)

| Formula | Meaning | Units |
|---------|---------|-------|
| $P_{\text{dirty}} = P_{\text{clean}} + \text{AI}$ | Dirty price from clean | Price per 100 notional |
| $P = \sum_{k=1}^{N-1} \frac{c/2}{(1+y/2)^k} + \frac{c/2 + F}{(1+y/2)^N}$ | Standard YTM pricing (semiannual) | Price per 100 notional |
| $\text{PV}(F \text{ due in } d \text{ days}) = \frac{F}{(1+y/2)^{d/182.5}}$ | Day-exponent discounting under semiannual quote | Price per 100 notional |
| $D_{\text{mod}} = -\frac{1}{P}\frac{dP}{dy}$ | Modified duration (yield duration) | Years |
| $\text{DV01}_y \approx \frac{P(y-1\text{bp}) - P(y+1\text{bp})}{2}$ | Yield DV01 (finite difference) | Price per 100 per 1 bp |
| $\text{DV01}_y \approx \frac{P \cdot D_{\text{mod}}}{10{,}000}$ | Yield DV01 (duration-based) | Price per 100 per 1 bp |
| $C_y = \frac{1}{P}\frac{d^2P}{dy^2}$ | Yield convexity | $1/(\text{yield})^2$ |
| $P = \sum_i \text{CF}_i \cdot P(0,t_i)$ | Curve PV (discount factors) | Price per 100 |
| $d(t) = \exp(-\hat{r}(t) \cdot t)$ | Continuous-spot to discount factor | Dimensionless |

---

### 25 Flashcards (Q/A)

**Q1:** What is YTM?
**A:** The single rate that discounts promised cash flows to the bond's market price.

**Q2:** What price should be used to solve for YTM: clean or dirty?
**A:** Dirty/full price, because settlement cash is quoted price plus accrued interest.

**Q3:** Define dirty price in Tuckman's terminology.
**A:** Full price = quoted (flat/clean) price + accrued interest.

**Q4:** Why can YTM be misleading for relative value?
**A:** Higher yield doesn't necessarily mean better value; yield is a summary statistic and can mislead.

**Q5:** What does "term structure of spot rates" imply about discounting coupon bonds?
**A:** Each payment must be discounted at a rate appropriate for its payment date.

**Q6:** What is modified duration in yield space?
**A:** $D_{\text{mod}} = -\frac{1}{P}\frac{dP}{dy}$

**Q7:** What is DV01 (general definition)?
**A:** $\text{DV01} = -\frac{\Delta P}{10{,}000 \cdot \Delta r}$

**Q8:** What is yield DV01?
**A:** DV01 computed with $r = y$: price change per 1 bp move in YTM.

**Q9:** How do you compute yield DV01 via finite differences?
**A:** $\text{DV01}_y \approx \frac{P(y - 1\text{ bp}) - P(y + 1\text{ bp})}{2}$

**Q10:** How is DV01 related to modified duration?
**A:** $\text{DV01} \approx \frac{P \cdot D_{\text{mod}}}{10{,}000}$

**Q11:** Define yield convexity.
**A:** $C_y = \frac{1}{P}\frac{d^2P}{dy^2}$

**Q12:** What does positive convexity imply for equal up/down yield shifts?
**A:** Price gains from yield decreases exceed price losses from equal yield increases.

**Q13:** When is "return equals yield" a valid interpretation?
**A:** If yield remains unchanged over a short period.

**Q14:** What is the typical compounding convention for bond yields in these notes?
**A:** Semiannual compounding.

**Q15:** How does Tuckman discount a cash flow due in $d$ days under semiannual yield quoting?
**A:** By $(1 + y/2)^{d/182.5}$

**Q16:** What is "curve PV"?
**A:** $P = \sum_i \text{CF}_i \cdot P(0,t_i)$

**Q17:** What is the continuous-spot representation of discount factors?
**A:** $d(t) = \exp(-\hat{r}(t) \cdot t)$

**Q18:** Define curve DV01 (one version).
**A:** $P_{\text{base}} - P_{\Delta}$ for a +1bp parallel bump in $\hat{r}(t)$.

**Q19:** Why can yield DV01 differ from curve DV01?
**A:** Yield DV01 is single-factor; curve DV01 depends on how the whole curve is bumped and how each cash flow maps to curve points.

**Q20:** What assumption underlies yield-based sensitivity measures?
**A:** Parallel yield shifts; Tuckman notes this assumption is not particularly good and can be inconsistent.

**Q21:** What does "no position in two securities hedges a third under all possible yield curve moves" mean?
**A:** Curve risk is multi-factor; yield-only hedges can fail under twists.

**Q22:** What's a "reprice check"?
**A:** Plug the solved YTM back into the PV formula and recover the dirty price.

**Q23:** For near-par bonds, what is a rough expectation for YTM?
**A:** Near the coupon rate under matched conventions.

**Q24:** Why does coupon level affect duration even if YTM is the same?
**A:** Higher coupons pull PV earlier, reducing yield duration (cash-flow timing effect).

**Q25:** What is the biggest "hidden variable" behind YTM?
**A:** The term structure: different spot rates for different maturities.

---

## Mini Problem Set

Provide brief solution sketches for questions 1–7 only.

---

**1. (Accrued interest)** A 5% coupon semiannual bond has 90 days accrued in a 180-day coupon period. What is accrued interest per 100?

**Sketch:** Coupon per period $= 2.5$. Accrual fraction $= 90/180 = 0.5$. $\text{AI} = 2.5 \times 0.5 = 1.25$.

---

**2. (Clean vs dirty)** If $P_{\text{clean}} = 99.60$ and $\text{AI} = 1.10$, compute $P_{\text{dirty}}$.

**Sketch:** $P_{\text{dirty}} = P_{\text{clean}} + \text{AI} = 100.70$.

---

**3. (YTM monotonicity)** Explain why $P(y)$ decreases in $y$ for a fixed cash-flow bond.

**Sketch:** Each term is $\text{CF}/(1+y/2)^{\text{positive exponent}}$; increasing $y$ increases the denominator.

---

**4. (Reprice check)** You solved $y$ for a bond and got $y = 4.2\%$. What should you do next to verify the computation?

**Sketch:** Plug $y$ into the PV formula and confirm the PV matches the dirty price to tolerance.

---

**5. (DV01 from bumps)** A bond price is 102.00 at $y$. At $y + 1$bp price is 101.95. Approximate DV01.

**Sketch:** $\text{DV01} \approx P(y) - P(y + 1 \text{ bp}) = 0.05$.

---

**6. (Duration approximation)** A bond has $P = 105$ and modified duration $D_{\text{mod}} = 6$. Approximate price change for +25bp in yield.

**Sketch:** $\Delta y = 0.0025$. $\Delta P \approx -P \cdot D_{\text{mod}} \cdot \Delta y = -105 \cdot 6 \cdot 0.0025 = -1.575$.

---

**7. (Convexity correction)** Same bond as Q6 with convexity $C_y = 50$. Include convexity for $\Delta y = 0.0025$.

**Sketch:** Add $\frac{1}{2} P \cdot C_y \cdot \Delta y^2 = 0.5 \cdot 105 \cdot 50 \cdot (0.0025)^2 \approx 0.0164$. Total $\approx -1.575 + 0.016 = -1.559$.

---

**8. (Solve YTM)** A 2-year 4% semiannual bond has dirty price 100.50. Solve for $y$ (semiannual).

---

**9. (Duration ordering)** For equal maturity and yield, which has higher modified duration: a higher coupon bond or a lower coupon bond? Explain.

---

**10. (Curve vs yield DV01)** Give two reasons curve DV01 can differ from yield DV01 even for the same bond.

---

**11. (Parallel bump definition)** Define precisely what "parallel 1bp bump" means for continuously compounded spot rates.

---

**12. (Term structure)** Explain why a single YTM cannot identify whether a bond is "rich/cheap" at specific maturities.

---

**13. (Hedge failure)** Describe a scenario where a yield-DV01 matched hedge loses money because the curve twists.

---

**14. (Day-count sensitivity)** How could changing the day-count convention in YTM discounting affect a reported yield?

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

| Content | Source |
|---------|--------|
| YTM definition as IRR | Tuckman Ch 3, equation (3.2) |
| Clean/dirty price relationship | Tuckman Ch 1, 3 |
| Full price = quoted + AI | Tuckman |
| Semiannual compounding convention | Tuckman |
| Day-exponent discounting $(1+y/2)^{d/182.5}$ | Tuckman |
| YTM solved by numerical methods | Tuckman |
| General DV01 definition | Tuckman Ch 5-6 |
| Modified duration definition | Tuckman Ch 5 |
| Yield convexity definition | Tuckman Ch 6, equations (6.34)–(6.35) |
| Parallel yield shift assumptions weak | Tuckman |
| No static hedge under all curve moves | Tuckman |
| Short-horizon return = yield interpretation | Tuckman |
| Bond PV as portfolio of zeros | Tuckman Ch 1-2 |

### (B) Reasoned Inference — Note Derivation Logic

| Content | Derivation Logic |
|---------|------------------|
| YTM as "average discount rate" | Follows from IRR definition—single rate matching price |
| Clean price avoids mechanical accrual drift | Follows from $P_{\text{dirty}} = P_{\text{clean}} + \text{AI}$ |
| Convexity benefit for plain vanilla bonds | Follows from $C_y > 0$ and Taylor expansion |
| Yield-DV01 hedge failure under twists | Follows from single-factor vs multi-factor logic |
| Near-par yield ≈ coupon rate | Follows from pricing equation structure |
| Zero-coupon YTM = spot rate | Follows from single cash flow discounting |

### (C) Speculation — Flag Uncertainties

| Content | Uncertainty |
|---------|-------------|
| "BEY" terminology in practice | Not explicitly named in cited sources; desk/market usage varies |
