# Chapter 8: Spreads 101 — G-spread, I-spread, Z-spread, OAS, and "what spread are we talking about?"

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- Dirty/full price equals clean/flat price plus accrued interest (Tuckman Ch 1-4)
- Yield spread does not take term structure into account and depends on bond coupon (O'Kane Ch 4-5)
- A single yield is not a complete description when interest rates change over time (Tuckman)
- Z-spread (ZVS) is a constant spread $\theta$ added so that PV of discounted cash flows equals bond price (O'Kane Ch 4-5)
- OAS is the spread (in bp) that must be added to all rates in a tree so the bond is priced correctly (Tuckman)
- For callable bonds: ZVS = OAS + Option Cost (O'Kane)
- Asset swap spread is the spread such that discounting bond cash flows by swap rates plus that spread gives bond price (Tuckman)
- Credit spread can be decomposed into actuarial spread + default risk premium + liquidity risk premium (O'Kane)
- On-the-run government yields can be affected by liquidity/specialness (Tuckman)
- Asset swap spreads reflect security-specific effects like special financing or supply/demand imbalances (Tuckman)
- DV01 = $-\frac{\partial P}{\partial y} \times 10^{-4}$ (O'Kane)

### (B) Reasoned Inference (Derived from A)

- G-spread and I-spread inherit the same benchmark-selection sensitivities as swap spreads (derived from Tuckman's discussion of swap spread benchmark issues)
- Concerns about on-the-run yield contamination apply equally to G-spreads computed versus on-the-run yields (derived from Tuckman's swap spread discussion)
- "CS01" is not a single universal number—it must be tied to a specific spread definition and curve/model (derived from multiple spread definitions having different price sensitivities)

### (C) Speculation (Clearly Labeled; Minimal)

- None in this chapter. All content traces to source material.

---

## Conventions & Notation

### Notation Glossary

| Symbol | Meaning |
|--------|---------|
| $100$ | Par/notional used for bond price quoting |
| $P_{\text{clean}}$ | Clean price (quoted price excluding accrued interest) |
| $P_{\text{dirty}}$ | Dirty/full price (cash price) including accrued interest |
| $AI$ | Accrued interest |
| $c$ | Annual coupon rate (decimal); e.g., 6% $\Rightarrow c = 0.06$ |
| $f$ | Coupon frequency (payments per year), e.g., $f = 2$ for semiannual |
| $T$ | Maturity in years |
| $t_k$ | Time in years to the $k$-th cash flow |
| $CF_k$ | Cash flow amount at time $t_k$ (per 100 notional) |
| $y$ | Bond yield to maturity (per annum), with stated compounding convention |
| $y_{\text{gov}}(T)$ | Government benchmark yield at maturity $T$ (or interpolated to $T$) |
| $y_{\text{swap}}(T)$ | Swap/par swap yield at maturity $T$ (or interpolated) |
| $s_G$ | G-spread (yield spread vs government benchmark) |
| $s_I$ | I-spread (yield spread vs swap benchmark) |
| $P(0,t)$ or $Z(0,t)$ | Discount factor from 0 to $t$ on the chosen benchmark curve |
| $s_Z$ | Z-spread (constant spread applied to benchmark discounting so PV matches $P_{\text{dirty}}$) |
| $s_{\text{OAS}}$ | Option-adjusted spread (constant spread applied to model rates so PV matches $P_{\text{dirty}}$) |
| $PV01$ | "Annuity": $\sum_m Z(0,t_m) \Delta(t_{m-1}, t_m)$ (used in asset swap formulas) |
| $D_s$ | Spread duration (price sensitivity to spread changes) |
| $CS01$ | Credit spread 01 — price change for a 1 bp change in spread |
| $\Delta$ | Accrual fraction within coupon period, $\Delta \in [0,1]$ |

### Defaults Used in Examples

- **Prices**: Per 100 notional
- **Price convention**: Market quotes are clean; valuation equations match dirty prices
- **Accrued interest**: Linear accrual fraction $\Delta$ within coupon period
- **YTM compounding**: Semiannual (bond-equivalent yield, $f_y = 2$) unless stated otherwise
- **Spreads**: Quoted in basis points (bp): 1 bp = 0.0001 in decimal rate units
- **Day count**: Stated explicitly in each example (varies by market/instrument)

### Spread Definitions Used

| Spread | Definition |
|--------|------------|
| **G-spread** | Yield spread to government/sovereign benchmark yield at same maturity |
| **I-spread** | Yield spread to swap/par swap curve yield at same maturity |
| **Z-spread** | Constant spread added to benchmark discounting so PV matches $P_{\text{dirty}}$ |
| **OAS** | Constant spread added to all model rates in a tree/model so model price matches $P_{\text{dirty}}$ |

---

## Setup

Spreads are a practical language for "everything in a bond price that is not the risk-free term structure." They are used to:

1. **Compare bonds** with different coupons/maturities by mapping their prices into a single number (a spread)
2. **Attribute price differences** to "rates" vs "spread" moves
3. **Risk-manage** credit and liquidity exposures (e.g., CS01 / spread DV01)

**But there is a trap**: the word "spread" can refer to multiple different objects. Even the seemingly simple "yield spread" depends on compounding and benchmark choice, and it is not term-structure consistent. In O'Kane's discussion of bond spread measures, the yield spread is noted as not taking term structure into account and as depending on bond coupon, motivating more robust measures like Z-spreads.

Also: yields themselves are summaries that can be misleading when rates vary through time; Tuckman emphasizes that a single yield is not a complete description when interest rates change over time.

This chapter builds a taxonomy so that when someone says "the spread widened 10 bp," you can immediately ask:

1. Which benchmark curve? (Treasury? fitted Treasury zero curve? OIS? swap?)
2. Which spread type? (G-spread? I-spread? Z-spread? OAS? asset swap?)
3. Is it clean or dirty price?
4. Which compounding/day-count?
5. Option-adjusted or not?

---

## Core Concepts

### 1) Clean vs Dirty Price

**Formal Definition:**

Dirty/full price equals clean/flat price plus accrued interest:

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + AI}$$

Tuckman states this directly and notes the naming conventions (dirty/full vs flat/clean).

A common accrued interest approximation is linear in accrual fraction $\Delta$:

$$AI = \Delta \cdot \frac{c}{f} \cdot 100$$

**Intuition:**

A bond coupon accrues over time; the seller has "earned" part of the next coupon. Clean price strips out this mechanical accrual so that quoted prices are less jumpy at coupon dates.

When you solve for a yield/spread that reprices a bond, you must be consistent: match the same price convention (usually dirty/full for PV equations).

**Trading / Risk Practice:**

- Spread engines (Z-spread, OAS) typically solve for a spread that matches the dirty price (cash price), because discounting future cash flows should match the total value exchanged at settlement
- Yield spreads in market commentary can be ambiguous about clean vs dirty; always confirm

---

### 2) Yield to Maturity (YTM) and Yield Spread

**Formal Definition:**

A yield $y$ is the (internal) rate such that discounting the bond's promised cash flows at that yield gives the bond's full price. In O'Kane's notation, the price of a bond with coupon rate $c$ and frequency $f$ can be written as:

$$P = \frac{c}{f} \sum_{n=1}^{N} \frac{1}{(1 + y/f)^n} + \frac{1}{(1 + y/f)^N}$$

where $N = fT$ (number of coupon periods).

The yield spread (generic) is often represented as:

$$y = y_{\text{ref}} + s$$

where $y_{\text{ref}}$ is a benchmark yield (e.g., Treasury yield at the same maturity) and $s$ is the yield spread.

**Intuition:**

YTM compresses an entire term structure into one number. It is convenient but can hide important details (coupon effects, curve slope).

**Trading / Risk Practice:**

- A yield spread is fast to quote and compare across issues
- But it is not term-structure consistent: O'Kane notes yield spread does not take into account the term structure and depends on coupon, so different bonds of the same issuer can show different yield spreads even if they are "equally credit risky"

---

### 3) G-spread (Spread to Government Curve)

**Formal Definition:**

G-spread at maturity $T$ is:

$$\boxed{s_G(T) = y_{\text{bond}} - y_{\text{gov}}(T)}$$

where $y_{\text{gov}}(T)$ is a government (sovereign) yield at the same maturity $T$.

If there is no benchmark at exactly $T$, $y_{\text{gov}}(T)$ is obtained by interpolation between surrounding government yields, or by using a fitted government curve.

**Intuition:**

"How much extra yield am I getting above governments?"

**Trading / Risk Practice:**

Must specify whether "government" means:
- On-the-run Treasury yield (liquidity-rich but potentially distorted), or
- Fitted zero/yield curve (smoother, arguably better for valuation)

Tuckman notes that swap spreads are often computed versus on-the-run government yields, but on-the-run yields can be affected by security-specific liquidity (specialness), and comparing to fitted yields can be better to avoid this contamination.

*Reasoned extension*: the same concern applies to G-spreads computed versus on-the-run yields.

---

### 4) I-spread (Spread to Swap Curve)

**Formal Definition:**

I-spread is commonly used to mean:

$$\boxed{s_I(T) = y_{\text{bond}} - y_{\text{swap}}(T)}$$

where $y_{\text{swap}}(T)$ is a swap curve yield (often par swap rate) at maturity $T$, possibly interpolated.

**Intuition:**

"How much extra yield above swaps?"

If swaps are used as the discounting benchmark (or as the "risk-free-ish" curve in some markets), I-spread is a natural quote.

**Trading / Risk Practice:**

You must clarify which swap curve is meant:
- OIS discounting curve
- LIBOR/single-curve swap curve (historical)
- "ISDA-style" curve construction (details matter)

Tuckman defines the related concept of swap spread as the difference between swap rates and government bond yields, and discusses benchmark choice issues in practice.

*Reasoned extension*: I-spread inherits the same "curve definition" sensitivities.

---

### 5) Z-spread (Zero-Volatility Spread)

**Formal Definition:**

The Z-spread (also called "zero-volatility spread", ZVS in some texts) is a constant spread such that discounting each cash flow by the benchmark curve plus the spread matches the bond price. O'Kane defines ZVS $\theta$ so that the PV of cash flows discounted with $(r(0,t_n) + \theta)$ equals the bond price.

A commonly used continuous-compounding variant is:

$$\boxed{P_{\text{dirty}} = \sum_k CF_k \cdot P_{\text{bench}}(0, t_k) \cdot e^{-s_Z t_k}}$$

which is consistent with the "constant spread added to discounting" idea (and is explicitly noted as an alternative definition used by some authors).

**Intuition:**

You are not comparing a single yield to a single benchmark yield. You are asking: "What constant spread must I apply to the entire benchmark discounting so the PV matches price?"

**Trading / Risk Practice:**

- Used for straight (non-callable) bonds because it is more term-structure consistent than yield spreads
- Still benchmark-dependent: Z-spread depends on the chosen discount curve and interpolation of discount factors

---

### 6) Asset Swap Spread

**Formal Definition:**

Tuckman defines asset swap spread as the spread such that discounting the bond's cash flows by swap rates plus that spread gives the bond price.

O'Kane provides a commonly used "par asset swap" formula:

$$\boxed{A(0) = \frac{P_{\text{Libor}} - P}{PV01(0,T)}}$$

with:

$$P_{\text{Libor}} = \frac{c}{f} \sum_{n=1}^{N} Z(0, \hat{t}_n) + Z(0,T)$$

and:

$$PV01(0,T) = \sum_{m=1}^{M} Z(0, t_m) \cdot \Delta(t_{m-1}, t_m)$$

interpretable as the present value "annuity" of the floating leg cash flows used to scale the spread leg.

**Intuition:**

Asset swaps convert a fixed-rate bond into a floating-rate exposure plus a spread. That "asset swap spread" is then interpreted as a spread-to-swaps measure.

**Trading / Risk Practice:**

O'Kane highlights that there are different asset swap market conventions, including "market asset swap" where the floating leg notional equals the market value of the bond, reducing counterparty exposure versus a par-notional structure.

If someone quotes "ASW," confirm whether it is par or market asset swap.

---

### 7) OAS (Option-Adjusted Spread)

**Formal Definition:**

Tuckman: OAS is the spread (in bp) that must be added to all rates in the tree so that the bond is priced correctly (i.e., matches market price).

O'Kane: OAS can be defined as the spread to the zero curve implied by a binomial tree; for callable (option-embedded) bonds, the relation is stated as:

$$\boxed{ZVS = OAS + \text{Option Cost}}$$

i.e., ZVS includes both the option effect and the "option-adjusted" spread component.

**Intuition:**

Z-spread treats cash flows as fixed. Callable/MBS-like instruments have state-dependent cash flows. OAS backs out a spread after accounting for option value under a rate model.

**Trading / Risk Practice:**

- OAS is model-dependent: different trees/vol assumptions can change OAS even if market price is the same
- OAS is commonly reported alongside "OAS duration" (DVOAS), the sensitivity to a 1 bp change in OAS

---

### 8) "What spread are we talking about?" — A Taxonomy of Spread Components

**Formal (Source-Backed) Components:**

O'Kane provides a conceptual decomposition:

$$\boxed{\text{Credit spread} = \text{Actuarial spread} + \text{Default risk premium} + \text{Liquidity risk premium}}$$

where the actuarial spread is tied to expected loss, and the other terms are risk premia (including liquidity).

**Intuition:**

A quoted "spread" is not purely default probability. It can embed:
- Expected credit loss (actuarial component)
- Risk premia for uncertainty about default and recovery
- Liquidity premium (difficulty/uncertainty of trading)
- And potentially other non-credit effects (funding specialness, technical supply/demand)

**Trading / Risk Practice:**

Tuckman emphasizes that asset swap spreads are computed using a swap curve "not contaminated by security-specific effects" and that asset swap spreads do reflect security-specific effects like special financing or supply/demand imbalances.

That is: even "spread-to-swaps" can reflect liquidity/technical effects, not just credit.

---

## Math and Derivations

### 1) Clean/Dirty Conversion and Accrued Interest

From the clean/dirty convention:

$$P_{\text{dirty}} = P_{\text{clean}} + AI$$

Tuckman notes this directly and ties it to how bonds are quoted.

A common accrued interest approximation is:

$$AI = \Delta \cdot \frac{c}{f} \cdot 100$$

with $\Delta$ the fraction of the coupon period elapsed.

**Unit check:**
- $c/f$ is coupon per period in decimal per year divided by periods per year $\Rightarrow$ decimal per period
- Multiply by 100 notional gives cash dollars per 100

---

### 2) Yield-to-Maturity Pricing Equation

For a bond with maturity $T$ years, coupon rate $c$, and semiannual yield $y$ (bond-equivalent yield), Tuckman gives a semiannual-compounded pricing formula. One form (with coupons as dollars per year per 100 notional) is:

$$\boxed{P = \frac{c}{y}\left(1 - \frac{1}{(1 + y/2)^{2T}}\right) + \frac{100}{(1 + y/2)^{2T}}}$$

(Here $c$ is annual coupon in dollars per 100; e.g., a 6% coupon bond has $c = 6$.)

**Solving for $y$:** For a given price $P$, $y$ generally requires a numerical root solve.

**Sanity checks:**
- For a zero-coupon bond ($c = 0$), the formula reduces to $P = 100/(1 + y/2)^{2T}$, so $y$ is directly solvable
- The function is monotone in $y$

---

### 3) Yield Spread, G-spread, and I-spread

A generic yield spread is:

$$s = y_{\text{bond}} - y_{\text{ref}}(T)$$

consistent with the "$y = y_T + s$" representation used in the sources.

- **G-spread:** $y_{\text{ref}}(T) = y_{\text{gov}}(T)$
- **I-spread:** $y_{\text{ref}}(T) = y_{\text{swap}}(T)$

**Interpolation (simple maturity-linear):**

If you have benchmark yields at maturities $T_1 < T < T_2$, a simple linear interpolation is:

$$y_{\text{ref}}(T) = y_{\text{ref}}(T_1) + \frac{T - T_1}{T_2 - T_1}\left(y_{\text{ref}}(T_2) - y_{\text{ref}}(T_1)\right)$$

**Important ambiguity:** Government yields used in practice may be on-the-run and impacted by liquidity; Tuckman highlights this issue when discussing swap spreads vs on-the-run Treasuries and suggests fitted yields can be a better benchmark.

---

### 4) Z-spread Equation, Root-Finding, and Repricing Check

**Core PV identity (continuous-spread-on-discount-factors form):**

Given benchmark discount factors $P_{\text{bench}}(0, t_k)$ and bond cash flows $CF_k$, define the Z-spread $s_Z$ by:

$$P_{\text{dirty}} = \sum_k CF_k \cdot P_{\text{bench}}(0, t_k) \cdot e^{-s_Z t_k}$$

This matches the "constant spread to discounting so PV matches price" idea and corresponds to the continuous-compounding variant noted in the sources.

**Root-finding:**

Define $f(s) = \sum_k CF_k \cdot P(0, t_k) \cdot e^{-s t_k} - P_{\text{dirty}}$. Solve $f(s) = 0$.

- $f(s)$ is decreasing in $s$ (higher spread $\rightarrow$ heavier discounting $\rightarrow$ lower PV)
- Use bisection or Newton. Bisection is robust.

**Repricing check:**

After solving $s_Z$, plug back into the PV equation and confirm the PV matches $P_{\text{dirty}}$ within tolerance.

**Why Z-spread differs from yield spreads:**

Yield spreads compare one yield to one yield. Z-spread revalues each cash flow with the entire curve. O'Kane explicitly notes yield spread ignores term structure and depends on coupon, motivating more curve-consistent measures.

---

### 5) Asset Swap Spread via PV Equation

Tuckman's asset swap spread definition is "the spread such that discounting the bond's cash flows by swap rates plus that spread gives the bond price," and notes that only under a flat swap curve does that spread equal the difference between bond yield and swap rate.

O'Kane provides a practical par asset swap formula:

$$A(0) = \frac{P_{\text{Libor}} - P}{PV01(0,T)}$$

**Unit check:**
- Numerator: price difference (dimensionless, "per 1 notional")
- Denominator: annuity-like term with units of years
- Result: annualized spread (rate, per year)

---

### 6) OAS in a Short-Rate Tree

Tuckman's definition: OAS is the constant spread added to all rates in the tree so that the bond prices correctly.

In a discrete one-period discounting step with short rate $r$ and OAS $x$, the discount factor is often written as:

$$DF = \frac{1}{1 + r + x}$$

(Convention varies; we state it explicitly in the example.)

For a callable bond, backward induction applies:

At each call date, the investor's node value is:

$$V = \min(\text{Call payoff}, \text{Continuation value})$$

because the issuer exercises to minimize its liability.

---

## Measurement & Risk

### 1) Spread Duration and CS01

**Definition (generic):**

Let $P(s)$ be the bond price as a function of a chosen spread measure $s$ (e.g., Z-spread or OAS). Define spread duration:

$$\boxed{D_s := -\frac{1}{P}\frac{\partial P}{\partial s}}$$

Define CS01 (credit spread 01) as the price change for a 1 bp change in spread:

$$\boxed{CS01 \approx \frac{\partial P}{\partial s} \cdot 0.0001 = -P \cdot D_s \cdot 0.0001}$$

**Intuition:**

If spread widens by 1 bp, price should go down (so $CS01$ is typically negative in "price points per 100").

**Z-spread case (closed form):**

If:
$$P(s) = \sum_k CF_k \cdot P_{\text{bench}}(0, t_k) \cdot e^{-s t_k}$$

then:
$$\frac{\partial P}{\partial s} = -\sum_k t_k \cdot \left(CF_k \cdot P_{\text{bench}}(0, t_k) \cdot e^{-s t_k}\right)$$

so:
$$D_s = \frac{\sum_k t_k \cdot PV_k(s)}{\sum_k PV_k(s)}$$

i.e., a PV-weighted average time under the spreaded discounting.

**Sanity checks:**
- $D_s > 0$
- Longer maturity (all else equal) $\rightarrow$ larger $D_s$

---

### 2) Relation to DV01 (High Level)

DV01 is the price change for a 1 bp change in yield. O'Kane defines DV01 as:

$$DV01 = -\frac{\partial P}{\partial y} \times 10^{-4}$$

i.e., negative derivative times 1 bp.

**High-level link:**

- If your spread definition is literally $s = y_{\text{bond}} - y_{\text{ref}}$ with $y_{\text{ref}}$ held fixed, then a 1 bp change in $s$ is a 1 bp change in $y_{\text{bond}}$, so spread DV01 is close to yield DV01
- For Z-spread or OAS, "spread" is applied to discounting (curve/model) rather than to a single yield, so the sensitivity is conceptually similar but not identical, especially on non-flat curves and for bonds with embedded options

---

### 3) Why Different Spread Definitions Imply Different "Spread Risk" Measures

A portfolio's "spread risk" depends on what you mean by spread:

| Spread | Risk Sensitivities |
|--------|-------------------|
| **G-spread risk** | Sensitive to (i) bond yield changes, (ii) government benchmark yield changes, (iii) benchmark selection (on-the-run vs fitted) |
| **I-spread risk** | Depends on which swap curve (OIS vs LIBOR), curve construction, interpolation |
| **Z-spread risk** | Depends on benchmark discount factors across maturities |
| **OAS risk** | Depends on rate model (tree dynamics, volatility), because option exercise and cash flows are model-dependent |

**Practical takeaway:** "CS01" is not a single universal number; it must be tied to a specific spread definition and curve/model.

---

## Worked Examples

All examples are per 100 notional. Spreads are in bp unless stated otherwise.

### Example A: G-spread — Clean to Dirty, Solve YTM, Interpolate Treasury Yield

**Given:**
- Corporate bond: maturity $T = 5$ years, coupon rate $c = 6\%$, semiannual coupons ($f = 2$)
- Cash flows:
  - $CF_k = 3$ at $t_k = 0.5, 1.0, \ldots, 4.5$ (9 coupons)
  - $CF_{10} = 103$ at $t_{10} = 5.0$ (final coupon + principal)
- Clean price quoted: $P_{\text{clean}} = 98.50$
- Settlement: on a coupon date $\Rightarrow \Delta = 0 \Rightarrow AI = 0$
- Treasury benchmark yields (annual, BEY):
  - $y_{\text{gov}}(4y) = 4.20\%$
  - $y_{\text{gov}}(6y) = 4.60\%$

**Step 1 — Dirty price:**

$$P_{\text{dirty}} = P_{\text{clean}} + AI = 98.50 + 0 = 98.50$$

**Step 2 — Solve yield to maturity $y$ (semiannual compounding):**

Using Tuckman's semiannual coupon-bond price-by-yield formula (with $c = 6$ dollars per year per 100):

$$P(y) = \frac{6}{y}\left(1 - \frac{1}{(1 + y/2)^{10}}\right) + \frac{100}{(1 + y/2)^{10}}$$

We solve $P(y) = 98.50$. Two trial yields:
- At $y_0 = 6.30\%$: $P(y_0) = 98.74168$
- At $y_1 = 6.40\%$: $P(y_1) = 98.27690$

One iteration (linear interpolation):
$$y \approx 6.30\% + \frac{0.24168}{0.46478} \cdot 0.10\% \approx 6.352\%$$

Refine check (plug in $y = 6.355\%$): $P(6.355\%) \approx 98.4995 \approx 98.50$

**Result:** $y_{\text{bond}} \approx 6.355\%$

**Step 3 — Interpolate Treasury yield at 5y:**

$$y_{\text{gov}}(5) = 4.20\% + \frac{5-4}{6-4}(4.60\% - 4.20\%) = 4.40\%$$

**Step 4 — Compute G-spread:**

$$s_G = y_{\text{bond}} - y_{\text{gov}}(5) = 6.355\% - 4.40\% = 1.955\% = \boxed{195.5 \text{ bp}}$$

---

### Example B: I-spread — Same Bond with Swap Curve Benchmark

**Given:**
- Same corporate bond and $y_{\text{bond}} = 6.355\%$ from Example A
- Swap curve (par swap rate yields):
  - $y_{\text{swap}}(4y) = 4.40\%$
  - $y_{\text{swap}}(6y) = 4.80\%$

**Step 1 — Interpolate swap yield at 5y:**

$$y_{\text{swap}}(5) = 4.40\% + \frac{5-4}{6-4}(4.80\% - 4.40\%) = 4.60\%$$

**Step 2 — Compute I-spread:**

$$s_I = y_{\text{bond}} - y_{\text{swap}}(5) = 6.355\% - 4.60\% = 1.755\% = \boxed{175.5 \text{ bp}}$$

**Compare:**
- Example A: $s_G \approx 195.5$ bp
- Example B: $s_I \approx 175.5$ bp

**Interpretation:** Different benchmark curves $\Rightarrow$ different spread levels. This is not a "math error"; it is the definition.

---

### Example C: Z-spread — Solve from Discount Factors

**Conventions:**
- Annual coupons ($f = 1$), valuation on coupon date so $AI = 0$ and clean = dirty
- Z-spread applied in continuous form:
$$P_{\text{dirty}} = \sum_k CF_k \cdot P(0, t_k) \cdot e^{-s_Z t_k}$$

**Given:**
- Bond: 3-year, annual coupon $c = 5\%$ on 100
- Cash flows: $CF_1 = 5$ at $t = 1$, $CF_2 = 5$ at $t = 2$, $CF_3 = 105$ at $t = 3$
- Dirty price: $P_{\text{dirty}} = 98.00$
- Benchmark discount factors:
  - $P(0,1) = 0.97$
  - $P(0,2) = 0.93$
  - $P(0,3) = 0.88$

**Define PV function:**

$$PV(s) = 5 \cdot 0.97 \cdot e^{-s \cdot 1} + 5 \cdot 0.93 \cdot e^{-s \cdot 2} + 105 \cdot 0.88 \cdot e^{-s \cdot 3}$$

Solve $PV(s) = 98.00$.

**Step 1 — Bracket the root:**
- $s = 1.00\%$: $PV(0.0100) = 99.0288$ (too high)
- $s = 2.00\%$: $PV(0.0200) = 96.2407$ (too low)

So $s_Z \in (1\%, 2\%)$.

**Step 2 — Bisection iterations:**
- Midpoint $s = 1.50\%$: $PV(0.0150) = 97.6245$ (too low) $\Rightarrow$ new bracket: $(1.00\%, 1.50\%)$
- Midpoint $s = 1.25\%$: $PV(0.0125) = 98.3241$ (too high) $\Rightarrow$ new bracket: $(1.25\%, 1.50\%)$
- Try $s = 1.365\%$: $PV(0.01365) = 98.0017$ (very close)

**Repricing check:** Using $s_Z = 1.365\%$, the repriced PV is $98.0017$, matching $98.00$ within 0.002.

**Final output:** $s_Z \approx 1.365\% \approx \boxed{136.5 \text{ bp}}$

**Sanity check:** Since $PV(0) = 101.9 > 98$, we need a positive spread to reduce PV. Good.

---

### Example D: Why G-spread $\neq$ Z-spread — Steep Curve Example

This example shows why a single-yield spread (G-spread) can differ from a term-structure spread (Z-spread) on a steep curve—exactly the failure mode highlighted in the sources.

**Conventions:**
- Annual coupon bond, valuation on coupon date ($AI = 0$)
- Government curve represented by spot (zero) rates at 1y/2y/3y, converted to discount factors
- G-spread uses the 3-year par yield implied by those discount factors

**Given:**
- Corporate bond: 3-year, coupon $c = 10\%$ annual
- Cash flows: $CF_1 = 10$, $CF_2 = 10$, $CF_3 = 110$
- Market dirty price: $P_{\text{dirty}} = 95.00$
- Benchmark (government) spot rates (annual comp):
  - 1y spot rate $r_1 = 2\%$ $\Rightarrow$ $P(0,1) = 1/1.02 = 0.980392$
  - 2y spot rate $r_2 = 4\%$ $\Rightarrow$ $P(0,2) = 1/1.04^2 = 0.924556$
  - 3y spot rate $r_3 = 6\%$ $\Rightarrow$ $P(0,3) = 1/1.06^3 = 0.839619$

**Step 1 — Corporate YTM $y_{\text{bond}}$ from price:**

Solve:
$$95 = \frac{10}{1+y} + \frac{10}{(1+y)^2} + \frac{110}{(1+y)^3}$$

Trial yields:
- $y = 12.05\%$ $\Rightarrow$ price $\approx 95.0805$ (too high)
- $y = 12.10\%$ $\Rightarrow$ price $\approx 94.9648$ (too low)

Interpolate to hit 95: $y_{\text{bond}} \approx 12.085\%$

**Step 2 — Government 3-year "benchmark yield" $y_{\text{gov}}(3)$ (par yield from discount factors):**

For an annual-pay par bond, the par coupon rate is:

$$y_{\text{par}} = \frac{1 - P(0,3)}{P(0,1) + P(0,2) + P(0,3)}$$

Compute:
- Numerator: $1 - 0.839619 = 0.160381$
- Denominator: $0.980392 + 0.924556 + 0.839619 = 2.744567$

So: $y_{\text{gov}}(3) \approx 0.160381 / 2.744567 = 5.8436\%$

**Step 3 — Compute G-spread:**

$$s_G = y_{\text{bond}} - y_{\text{gov}}(3) = 12.085\% - 5.8436\% = 6.2414\% \approx \boxed{624.1 \text{ bp}}$$

**Step 4 — Compute Z-spread $s_Z$ from discount factors:**

Solve:
$$95 = 10 \cdot 0.980392 \cdot e^{-s \cdot 1} + 10 \cdot 0.924556 \cdot e^{-s \cdot 2} + 110 \cdot 0.839619 \cdot e^{-s \cdot 3}$$

Root-finding (bisection bracket):
- $s = 5.8\%$: PV $\approx 95.0926$ (too high)
- $s = 5.9\%$: PV $\approx 94.8345$ (too low)

Interpolate:
$$s_Z \approx 5.8\% + \frac{95.0926 - 95.0000}{95.0926 - 94.8345} \cdot 0.1\% \approx 5.836\%$$

So: $s_Z \approx 5.836\% \approx \boxed{583.6 \text{ bp}}$

**Step 5 — Explain the difference:**

- G-spread is based on one yield (corporate YTM) minus one benchmark yield (3-year par yield)
- Z-spread is based on matching PV against the full discount curve, so each cash flow is discounted by its own maturity discount factor

On a steep curve and a high-coupon bond, these can differ by tens of bp (here ~40 bp). This aligns with the source warning that yield spreads ignore term structure and depend on coupon.

---

### Example E: Asset Swap Spread (Par Asset Swap)

**Conventions:**
- Annual payments ($f = 1$), on coupon date ($AI = 0$)
- "Swap curve" discount factors given as $Z(0,t)$
- Par asset swap spread via: $A(0) = \frac{P_{\text{Libor}} - P}{PV01(0,T)}$

**Given:**
- Bond: 5-year annual coupon $c = 6\%$ on 100
- Dirty price: $P = 98.50$
- "Swap/LIBOR-quality" discount factors:
  - $Z(0,1) = 0.97$, $Z(0,2) = 0.94$, $Z(0,3) = 0.90$, $Z(0,4) = 0.85$, $Z(0,5) = 0.80$
- Year fractions $\Delta = 1$ each year

**Step 1 — Compute $PV01(0,T)$ (annuity):**

$$PV01(0,5) = \sum_{m=1}^{5} Z(0,m) \cdot \Delta = 0.97 + 0.94 + 0.90 + 0.85 + 0.80 = 4.46$$

**Step 2 — Compute $P_{\text{Libor}}$:**

- Coupon PV: $6 \cdot \sum_{m=1}^{5} Z(0,m) = 6 \cdot 4.46 = 26.76$
- Principal PV: $100 \cdot Z(0,5) = 100 \cdot 0.80 = 80.00$

So: $P_{\text{Libor}} = 26.76 + 80.00 = 106.76$

**Step 3 — Compute asset swap spread $A(0)$:**

Work per 1 notional to match formula units:
- $P_{\text{Libor}}/100 = 1.0676$
- $P/100 = 0.9850$
- Difference $= 0.0826$

Then:
$$A(0) = \frac{0.0826}{4.46} = 0.01852 = 1.852\% \approx \boxed{185.2 \text{ bp}}$$

**Market convention warning:** O'Kane notes "market asset swap" uses floating-leg notional equal to bond's market value (reducing counterparty exposure), so "ASW" can mean different things on different desks.

---

### Example F: OAS — Callable Bond in a 2-Step Rate Tree

**Conventions (toy model):**
- Annual time steps
- Discounting uses $\frac{1}{1 + r + x}$, where $r$ is node short rate, $x$ is OAS (decimal)
- Risk-neutral probability: $p = 0.5$ up, $0.5$ down
- Call decision at $t = 1$: investor value is $\min(\text{call payoff}, \text{continuation})$

**Bond:**
- 2-year bond, annual coupon 5% on 100
- Callable at $t = 1$ at call price 100 (on coupon date)
- Cash flows if not called: 5 at $t = 1$, 105 at $t = 2$
- If called at $t = 1$: investor receives 105 (coupon 5 + call principal 100)

**Short-rate tree:**
- From $t = 0$ to $t = 1$: $r_0 = 4\%$
- From $t = 1$ to $t = 2$:
  - Up node: $r_u = 6\%$
  - Down node: $r_d = 3\%$

**Market dirty price:** $P_{\text{mkt}} = 99.50$

**Step 1 — Price with OAS $x = 0$:**

At $t = 1$, compute node values:

*Up node ($r = 6\%$):*
- Continuation: $5 + \frac{105}{1 + 0.06} = 5 + 99.0566 = 104.0566$
- Call payoff: $105$
- Issuer chooses min $\Rightarrow$ $V_1(u) = 104.0566$ (not called)

*Down node ($r = 3\%$):*
- Continuation: $5 + \frac{105}{1 + 0.03} = 5 + 101.9417 = 106.9417$
- Call payoff: $105$
- Min $\Rightarrow$ $V_1(d) = 105$ (called)

Expected value at $t = 1$:
$$E[V_1] = 0.5 \cdot 104.0566 + 0.5 \cdot 105 = 104.5283$$

Discount to $t = 0$ at $r_0 = 4\%$:
$$P(0) = \frac{104.5283}{1.04} = 100.5080$$

With $x = 0$, model price is 100.51, above market 99.50 $\Rightarrow$ need positive OAS.

**Step 2 — Price with $x = 1\%$ (100 bp) to bracket:**

At $t = 1$:
- Up node: $5 + \frac{105}{1.07} = 5 + 98.1308 = 103.1308$ $\Rightarrow$ not called
- Down node: $5 + \frac{105}{1.04} = 5 + 100.9615 = 105.9615$ $\Rightarrow$ called $\Rightarrow$ 105

Expected: $E[V_1] = 0.5 \cdot 103.1308 + 0.5 \cdot 105 = 104.0654$

Discount at $1 + r_0 + x = 1.05$:
$$P(1\%) = \frac{104.0654}{1.05} = 99.1090$$

Model price is 99.11, below 99.50 $\Rightarrow$ OAS is between 0% and 1%.

**Step 3 — Solve for OAS (try $x = 0.72\%$):**

*Up node:*
$$V_1(u) = 5 + \frac{105}{1.0672} = 5 + 98.3901 = 103.3901$$

*Down node continuation:*
$$5 + \frac{105}{1.0372} = 5 + 101.2348 = 106.2348 > 105 \Rightarrow \text{called} \Rightarrow V_1(d) = 105$$

Expected:
$$E[V_1] = 0.5 \cdot 103.3901 + 0.5 \cdot 105 = 104.1951$$

Discount at $1 + 0.04 + 0.0072 = 1.0472$:
$$P(0.72\%) = \frac{104.1951}{1.0472} = 99.4979 \approx 99.50$$

**Final output:** $OAS \approx \boxed{72 \text{ bp}}$

**Key lesson (model dependence):** If you change the tree volatility (how far apart 6% and 3% are), the option value changes and so does the OAS. This is inherent in the OAS definition.

---

### Example G: Component Taxonomy — Liquidity/Funding Effects

**Label:** Conceptual attribution (not a structural identity)

**Given:**
- Two bonds A and B have identical promised cash flows (same issuer, same coupon, same maturity)
- Market quotes:
  - Bond A (more liquid): Z-spread $s_Z^A = 150$ bp
  - Bond B (less liquid): Z-spread $s_Z^B = 200$ bp
  - Difference $\Delta s = 50$ bp $= 0.0050$
- Assume both trade near price $P \approx 100$, spread duration $D_s \approx 4.2$

**Step 1 — Translate spread difference into a price difference:**

Using first-order approximation:
$$\Delta P \approx -P \cdot D_s \cdot \Delta s = -100 \cdot 4.2 \cdot 0.0050 = -2.10$$

So the illiquid bond (wider spread by 50 bp) is cheaper by about 2.1 price points per 100.

**Step 2 — Interpret as non-credit components:**

O'Kane's decomposition explicitly includes a liquidity risk premium as part of "credit spread" in market quotes. So it is consistent with the idea that Bond B's extra ~50 bp reflects liquidity premium rather than higher expected default loss.

**Optional extra (funding/technical):**

Tuckman remarks that asset swap spreads can reflect special financing or supply/demand imbalances (security-specific effects). So part of the 50 bp could be viewed as "funding specialness/technical" rather than fundamental credit.

---

## Practical Notes

### Common Ambiguity Traps

| Issue | Details |
|-------|---------|
| **Benchmark curve choice** | On-the-run government yields can be distorted by liquidity/specialness; fitted curves can reduce contamination. "Swap curve" can mean OIS vs LIBOR vs legacy single-curve constructions. |
| **Clean vs dirty price** | Z-spread/OAS should be solved to match the dirty price; mixing conventions will bias spreads. Recall: dirty/full price = clean/flat price + accrued interest. |
| **Compounding/annualization mismatch** | BEY (semiannual), annual effective, and continuous spreads differ. If you subtract yields with different conventions, the spread is meaningless. Tuckman notes changing the compounding convention changes the numerical yield. |
| **Par yields vs zero rates vs discount factors** | Par yields (coupon rates that price a par bond) are not the same as spot/zero rates. Z-spread is defined through discounting each cash flow; using par yields "as if they were zeros" is a common mistake. |
| **"Spread" on callable/option bonds** | Z-spread on a callable bond includes option value; OAS attempts to remove it (model-dependent). |

### Implementation Pitfalls

| Issue | Details |
|-------|---------|
| **Interpolation choices** | Interpolating yields vs discount factors vs forward rates produces different implied forwards, hence different PVs and different Z-spreads. |
| **Stubs and irregular coupon periods** | Accrued interest and year-fractions must match market conventions; otherwise Z-spreads/OAS can be off materially. |
| **Negative rates** | Discount factors remain positive, but rate/spread formulas can behave oddly if implemented with log/continuous conversions; test carefully. |
| **Asset swap convention mismatch** | Par vs market asset swap structures differ; confirm the convention before comparing quoted ASW levels. |

### Verification Tests

| Test | Purpose |
|------|---------|
| **Spread sign check** | Wider spread $\Rightarrow$ lower price (holding benchmark curve fixed) |
| **Repricing check** | Always plug computed Z-spread/OAS back in and confirm you reproduce $P_{\text{dirty}}$ |
| **Limiting cases** | Zero-coupon bond: yield spread and Z-spread should be much closer (less coupon effect). Flat curve: yield spread and Z-spread should be closer. Near-par bond: small changes translate approximately linearly. |

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **Clean vs dirty:** PV equations should match the dirty/full price, where dirty = clean + accrued interest
2. **Yield is a summary;** it can be misleading when rates vary over time
3. **Yield spreads** (like G-spread, I-spread) are fast but ignore term structure and depend on coupon
4. **G-spread** = bond YTM $-$ government benchmark yield at same maturity (interpolated or fitted)
5. **I-spread** = bond YTM $-$ swap curve yield at same maturity (must specify which swap curve)
6. **Z-spread** solves a PV match using the full benchmark discount curve and a constant spread
7. **OAS** solves a PV match using an interest-rate model/tree, adding a spread to all rates; it is model-dependent
8. **For callable bonds,** Z-spread includes option value; OAS attempts to remove it (conceptually ZVS = OAS + option cost)
9. **Asset swap spread** is a spread-to-swaps concept; conventions differ (par vs market asset swap)
10. **"Spread" reflects multiple components** (expected loss + risk premia + liquidity, etc.), not only default probability

---

### Cheat Sheet: Definitions + When to Use + Dependencies

| Spread | Definition | Use | Depends On |
|--------|------------|-----|------------|
| **G-spread** | $s_G = y_{\text{bond}} - y_{\text{gov}}(T)$ | Quick relative value vs government curve | Benchmark choice (on-the-run vs fitted), interpolation, yield convention |
| **I-spread** | $s_I = y_{\text{bond}} - y_{\text{swap}}(T)$ | Relative value vs swaps (common in credit markets) | Which swap curve (OIS vs LIBOR), curve construction, yield convention |
| **Z-spread** | Constant spread $s_Z$ so PV with curve+spread matches $P_{\text{dirty}}$ | Non-callable bonds; term-structure consistent quoting | Discount curve (Treasury, OIS, swaps), interpolation method, clean/dirty |
| **OAS** | Spread added to all rates in a model/tree so bond prices correctly | Callable bonds, MBS; isolate "option-adjusted" compensation | Rate model, volatility assumption, call/prepay model, curve |
| **ASW** | Spread-to-swaps from asset swap package; $A = (P_{\text{Libor}} - P)/PV01$ | Spread-to-swaps quote in credit relative value | Par vs market convention, swap discount curve, bond price, accrual details |

---

### Flashcards (25 Q/A)

**Q1:** What is the dirty price of a bond?
**A1:** Dirty (full) price = clean (flat) price + accrued interest.

**Q2:** Why do Z-spreads usually match dirty price, not clean price?
**A2:** PV discounting should reproduce the total cash price exchanged; clean price excludes accrued interest mechanically.

**Q3:** Define yield spread in the simplest form.
**A3:** $s = y_{\text{bond}} - y_{\text{ref}}(T)$ (often written $y = y_T + s$).

**Q4:** Define G-spread.
**A4:** Bond YTM minus a government benchmark yield at the same maturity (interpolated/fitted).

**Q5:** Define I-spread.
**A5:** Bond YTM minus a swap curve yield at the same maturity (must define the swap curve).

**Q6:** What is the key limitation of yield spreads?
**A6:** They ignore term structure and depend on coupon, so they can mislead across bonds.

**Q7:** What is Z-spread (ZVS)?
**A7:** Constant spread added to benchmark discounting so PV equals the bond price.

**Q8:** What is the continuous-compounding Z-spread PV equation?
**A8:** $P = \sum_k CF_k \cdot P(0, t_k) \cdot e^{-s_Z t_k}$

**Q9:** Why can G-spread differ from Z-spread?
**A9:** G-spread is a one-yield comparison; Z-spread uses the full curve for each cashflow (term-structure effect).

**Q10:** Define OAS in one sentence.
**A10:** OAS is the spread added to all rates in a tree/model so the bond prices correctly.

**Q11:** How is ZVS related to OAS for callable bonds (conceptually)?
**A11:** ZVS = OAS + option cost (ZVS includes option value).

**Q12:** What is "spread duration"?
**A12:** $D_s = -\frac{1}{P}\frac{\partial P}{\partial s}$ for a chosen spread measure.

**Q13:** What is CS01?
**A13:** Approximate price change for a 1 bp change in spread: $CS01 \approx \frac{\partial P}{\partial s} \cdot 10^{-4}$.

**Q14:** Why isn't there a single universal CS01?
**A14:** Because "spread" can mean G-, I-, Z-, or OAS spread; each implies a different mapping from spread to PV.

**Q15:** What is asset swap spread (conceptually, Tuckman)?
**A15:** Spread such that discounting bond cash flows by swap rates plus spread gives the bond price.

**Q16:** What's the par asset swap spread formula (O'Kane)?
**A16:** $A = (P_{\text{Libor}} - P)/PV01$

**Q17:** What does $PV01$ represent in the par asset swap formula?
**A17:** An annuity-like present value term $\sum Z \cdot \Delta$.

**Q18:** What's a key convention ambiguity for asset swaps?
**A18:** Par vs market asset swap (floating notional differs).

**Q19:** Name a day-count convention used for US Treasuries.
**A19:** Actual/Actual.

**Q20:** Name a day-count convention often used for corporates.
**A20:** 30/360.

**Q21:** If spread widens, what should happen to price (all else equal)?
**A21:** Price decreases.

**Q22:** What's the repricing test for Z-spread?
**A22:** Plug $s_Z$ back into PV and ensure it reproduces $P_{\text{dirty}}$.

**Q23:** Why is OAS model-dependent?
**A23:** The option's value depends on the interest-rate dynamics/volatility used in the model.

**Q24:** List three components that market "credit spread" may include (O'Kane).
**A24:** Actuarial spread (expected loss), default risk premium, liquidity risk premium.

**Q25:** What question should you ask when someone says "spread widened"?
**A25:** "Which spread definition and which benchmark curve are you using?"

---

## Mini Problem Set

### Questions

**1.** A bond has clean price 101.20, accrued interest 0.80. What is the dirty price?

**2.** A semiannual coupon bond has coupon rate 6% and trades between coupons with accrual fraction $\Delta = 0.40$. What is accrued interest per 100?

**3.** A 2-year annual coupon bond with cashflows 5, 105 has dirty price 99.00. Solve the YTM (annual compounding) approximately.

**4.** A bond YTM is 7.10%. The interpolated government yield at maturity is 4.85%. Compute G-spread in bp.

**5.** Given discount factors $P(0,1) = 0.97$, $P(0,2) = 0.93$, $P(0,3) = 0.88$ and bond cashflows 5, 5, 105, dirty price 98.00, solve for Z-spread approximately (continuous form).

**6.** Explain in one paragraph why yield spread can differ from Z-spread on a steep curve.

**7.** A bond has price 100 and spread duration 4.5. Estimate the price change for a 10 bp spread widening.

**8.** You are given two curves: fitted Treasury zero curve and on-the-run Treasury yields. Which one is more appropriate for Z-spread computation and why?

**9.** A trader quotes "I-spread 120 bp." List three follow-up clarifications you need.

**10.** A Z-spread engine accidentally matches clean price instead of dirty price. What is the likely direction of bias in the computed Z-spread (qualitatively)?

**11.** For a callable bond, would you expect Z-spread to be larger or smaller than OAS? Explain.

**12.** How can different interpolation methods (linear-in-yield vs linear-in-log-discount) change Z-spreads?

**13.** In negative rate environments, what can go wrong if you apply a spread by adding to rates vs multiplying discount factors?

**14.** Provide a conceptual decomposition for why two bonds with the same issuer might trade at different spreads.

---

### Solution Sketches (1–7)

**1.** $P_{\text{dirty}} = P_{\text{clean}} + AI = 101.20 + 0.80 = 102.00$

**2.** Coupon per period $= \frac{c}{f} \cdot 100 = \frac{0.06}{2} \cdot 100 = 3.00$. Accrued interest $AI = \Delta \cdot 3.00 = 0.40 \cdot 3.00 = 1.20$.

**3.** Solve $99 = \frac{5}{1+y} + \frac{105}{(1+y)^2}$.
- Trial $y = 6\%$: PV $= 5/1.06 + 105/1.06^2 \approx 4.717 + 93.46 = 98.18$ (too low) $\Rightarrow$ yield too high
- Trial $y = 5\%$: PV $= 4.762 + 95.24 = 100.00$ (too high) $\Rightarrow$ yield too low
- So $y$ is between 5% and 6%, closer to ~5.6% (refine if desired)

**4.** $s_G = 7.10\% - 4.85\% = 2.25\% = 225$ bp

**5.** Use Example C logic: bracket $s$ by evaluating $PV(s) = 5 \cdot 0.97 \cdot e^{-s} + 5 \cdot 0.93 \cdot e^{-2s} + 105 \cdot 0.88 \cdot e^{-3s}$. Try $s = 1\%$ and $s = 2\%$; bisection until PV $\approx 98$. (Answer: approximately 136.5 bp per Example C.)

**6.** Yield spread compares one yield to one yield and ignores that different cashflows should be discounted at different maturities; Z-spread revalues each cashflow using the full curve, so on a steep curve and/or high coupon bond the two measures can diverge materially.

**7.** $\Delta P \approx -P \cdot D_s \cdot \Delta s = -100 \cdot 4.5 \cdot 0.0010 = -0.45$ price points (10 bp $= 0.0010$).

---

## Source Map

### (A) Verified Facts — Specific Sources

| Fact | Source |
|------|--------|
| Full price = flat price + accrued interest | Tuckman Ch 1-4 |
| Yield spread ignores term structure, depends on coupon | O'Kane Ch 4-5 |
| Single yield is not complete description when rates vary | Tuckman |
| Z-spread (ZVS) definition as constant spread to discounting | O'Kane Ch 4-5 |
| OAS definition (spread added to tree rates) | Tuckman |
| ZVS = OAS + Option Cost relation | O'Kane |
| Asset swap spread definition | Tuckman, O'Kane |
| Credit spread decomposition (actuarial + risk premia + liquidity) | O'Kane |
| On-the-run yields affected by liquidity/specialness | Tuckman |
| Asset swap spreads reflect security-specific effects | Tuckman |
| DV01 formula | O'Kane |
| Par asset swap formula | O'Kane |
| Semiannual bond pricing formula | Tuckman |

### (B) Reasoned Inference — Derivation Logic

| Inference | Derivation |
|-----------|------------|
| G-spread inherits benchmark-selection sensitivities | Extended from Tuckman's swap spread benchmark discussion |
| CS01 is not universal | Follows from multiple spread definitions having different price sensitivities |
| Spread duration formula for Z-spread | Direct calculus on the continuous-form Z-spread PV equation |

### (C) Speculation — Flagged Uncertainties

None. All content in this chapter traces to user-provided material backed by Tuckman and O'Kane sources.

---

*Chapter 8 — Fixed Income: Practice and Theory*
