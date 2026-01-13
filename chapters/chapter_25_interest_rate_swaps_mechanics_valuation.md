# Chapter 25: Interest Rate Swaps — Mechanics, Valuation, and Curve Dependencies

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- A swap rate is the fixed rate that makes a new interest rate swap have value zero at initiation (par).
- An interest rate swap can be viewed as the difference between a fixed-rate bond and a floating-rate bond (a convenient valuation decomposition); notional is typically not exchanged in the swap itself.
- In standard swap cashflow conventions, the floating rate is observed at the beginning of a period and the coupon is paid at the end.
- In practice, fixed and floating legs often differ in payment frequency and day count; e.g., sources note typical IRS structures and that (in the U.S.) a common convention is fixed 30/360 and floating ACT/360.
- In a single-curve setup, the standard par swap rate admits the discount-factor formula (for aligned schedules) $S = \frac{P(0,T_\alpha) - P(0,T_\beta)}{A}$ with annuity $A = \sum \tau_i P(0,T_i)$.
- In a multi-curve setup, the swap is valued by discounting cashflows with a (universal) discount curve $P(\cdot)$, while forward rates for index $k$ are implied from an index-specific curve $P_k(\cdot)$; the provided text gives explicit formulas for swap value and forward rates.
- For collateralized deals, markets consider an overnight-index curve (e.g., Fed funds/OIS) as an appropriate discount curve, while a LIBOR curve is used as a forward (projection) curve.
- Swap spreads are commonly described as the difference between a par swap rate and a government yield of the same maturity; swap rates should (conceptually) exceed government yields due to bank credit embedded in swap floating rates, though multiple forces can affect the spread.

### (B) Reasoned Inference (Derived from A)

- Given leg PV formulas, the par fixed rate is "PV of expected floating coupons per unit notional divided by fixed-leg annuity," with the curve dependence made explicit (discount vs projection).
- In single-curve valuation (same curve for discounting and projecting), the floating leg PV can be shown to telescope to $1 - P(0,T)$ for a spot-start swap under standard assumptions (reset at $t=0$, no spread, no stubs), yielding the classic par swap rate formula.
- Curve-specific PV01 concepts follow by perturbing either $P_d$ (discount curve) or $P_k$ / implied forwards (projection curve) while holding the other fixed.

### (C) Speculation (Clearly Labeled; Minimal)

I'm not sure which desk-specific conventions you want for business-day adjustment, payment lags, stub treatment, and exact floating index fixing conventions (e.g., lookback, lockout, compounding-in-arrears for RFR swaps). The notes therefore treat these as configurable and focus on mechanics/valuation identities that remain valid once the schedule and accrual factors are specified.

---

## Conventions & Notation

### Setup

**Swap type:** Plain-vanilla fixed-for-floating interest rate swap.

**Payment timing:** Floating rate is set at the beginning of each accrual period and the coupon is paid at the end (standard "set in advance, pay in arrears" structure).

**Day count / frequency:** Market-dependent. For interest rate swaps, sources highlight that fixed and floating legs often use different day-count conventions (e.g., floating often ACT/360; fixed often 30/360 or ACT/365 depending on instrument/currency).

**Default toy convention for most numeric examples (for tractability):**
- Fixed leg: semiannual payments with $\tau^{\text{fix}} = 0.5$ each period (30/360-style simplification).
- Floating leg: quarterly payments with $\tau^{\text{flt}} = 0.25$ each period (ACT/360-style simplification).
- No stubs, no payment lags, no business-day adjustments (purely pedagogical).

**Curve inputs:** Treated as given. This chapter does not bootstrap curves (outside scope).

**Valuation time:** $t = 0$ unless stated otherwise. Time measured in years.

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $N$ | Notional principal (currency units, e.g., USD) |
| $c$ | Fixed swap rate (annualized decimal, e.g., 0.03 = 3%) |
| $\{T_j^{\text{fix}}\}_{j=1}^{n_f}$ | Fixed-leg payment dates |
| $\{T_i^{\text{flt}}\}_{i=0}^{n_p}$ | Floating-leg reset/payment dates (payments at $T_{i+1}^{\text{flt}}$) |
| $\tau_j^{\text{fix}}$ | Year fraction for fixed period: $\text{YearFrac}(T_{j-1}^{\text{fix}}, T_j^{\text{fix}})$ |
| $\tau_i^{\text{flt}}$ | Year fraction for floating period: $\text{YearFrac}(T_i^{\text{flt}}, T_{i+1}^{\text{flt}})$ |
| $P_d(0,T)$ | Discount factor from discount curve (dimensionless) |
| $P_k(0,T)$ | Discount-factor-like object for index/tenor $k$ used to infer forwards |
| $L_k(0; T_i, T_{i+1})$ | Forward money-market rate for tenor $k$ over $[T_i, T_{i+1}]$ |
| $\text{PV}_{\text{fixed}}$, $\text{PV}_{\text{float}}$ | PV of each leg (excluding notional exchange) |
| $V_{\text{payer}}$, $V_{\text{recv}}$ | Swap PV from payer / receiver viewpoint |
| $A^{\text{fix}}(0)$ | Fixed-leg annuity (a.k.a. PVBP per unit rate) |

### Key Formulas (Preview)

**Forward money-market rate (projection):**
$$L_k(0; T_i, T_{i+1}) = \frac{1}{\tau_i^k}\left(\frac{P_k(0, T_i)}{P_k(0, T_{i+1})} - 1\right)$$

**Fixed-leg annuity:**
$$A^{\text{fix}}(0) = \sum_{j=1}^{n_f} \tau_j^{\text{fix}} \, P_d(0, T_j^{\text{fix}})$$

**Payer vs receiver swap:**
- Payer (pay fixed / receive float): $V_{\text{payer}} = \text{PV}_{\text{float}} - \text{PV}_{\text{fixed}}$
- Receiver (receive fixed / pay float): $V_{\text{recv}} = -V_{\text{payer}} = \text{PV}_{\text{fixed}} - \text{PV}_{\text{float}}$

**PV01 preview metrics (curve-specific):**
- $\text{PV01}_{\text{disc}}$: PV change under a 1bp bump to discount curve (projection held fixed).
- $\text{PV01}_{\text{proj}}$: PV change under a 1bp bump to projection curve (discount held fixed).

---

## Core Concepts

### 1) Interest Rate Swap (Plain-Vanilla Fixed-for-Floating)

**Formal Definition:** A contract where two parties exchange interest payments on a notional $N$: one leg pays a fixed rate $c$, the other pays a floating reference rate (e.g., LIBOR tenor or another index) observed/reset periodically.

**Intuition:** You are swapping interest-rate exposure:
- Pay fixed / receive float: you benefit if rates rise (floating receipts increase) and lose if rates fall.
- Receive fixed / pay float: you benefit if rates fall and lose if rates rise.

**Trading / Risk / Portfolio Practice:**
- "Par swap" means the fixed rate is set so that PV is ~0 at inception.
- Swaps are building blocks for curve trades, liability management, and hedging interest-rate risk (e.g., swap a fixed-rate bond issuance into synthetic floating funding).

---

### 2) Fixed Leg

**Formal Definition:** Cashflows at fixed-leg payment dates $\{T_j^{\text{fix}}\}$:
$$\text{CF}_j^{\text{fix}} = N \, c \, \tau_j^{\text{fix}}$$
where $\tau_j^{\text{fix}}$ is the year fraction for the accrual period.

**Intuition:** A deterministic coupon stream (once $c$ and schedule are fixed).

**Trading / Risk / Portfolio Practice:**
- Fixed leg conventions vary by market (frequency/day count). For example, sources highlight common conventions like fixed 30/360 in U.S. markets (while other markets differ).

---

### 3) Floating Leg

**Formal Definition:** At each period $[T_i^{\text{flt}}, T_{i+1}^{\text{flt}}]$, the floating rate is set at the beginning and paid at the end:
$$\text{CF}_{i+1}^{\text{flt}} = N \, L_k(T_i^{\text{flt}}; T_i^{\text{flt}}, T_{i+1}^{\text{flt}}) \, \tau_i^{\text{flt}}$$
(Often this is a money-market/simple-compounded rate over the accrual period.)

**Intuition:** A floating-rate note-like stream: coupons reset so that (on reset dates) the instrument is near par in single-curve logic.

**Trading / Risk / Portfolio Practice:**
- Day count often differs from fixed leg (e.g., floating ACT/360 is common in LIBOR markets).

---

### 4) Payer vs Receiver and Sign Conventions

**Formal Definition:**
- **Payer swap** (pay fixed, receive float): $V_{\text{payer}} = \text{PV}_{\text{float}} - \text{PV}_{\text{fixed}}$
- **Receiver swap** (receive fixed, pay float): $V_{\text{recv}} = \text{PV}_{\text{fixed}} - \text{PV}_{\text{float}}$

This aligns with the fixed-vs-floating leg decomposition used in the provided sources.

**Intuition:** Your PV is positive if the leg you receive is "worth more" than what you pay.

**Trading / Risk / Portfolio Practice:** Sign mistakes are among the most common implementation errors (especially when swaps are netted and collateralized).

---

### 5) Discount Curve vs Projection Curve

**Formal Definition:**
- A **discount curve** produces discount factors $P_d(0,T)$ used to PV cashflows.
- A **projection curve** (or index curve) produces forward rates for an index $k$, commonly via an index-curve object $P_k(0,T)$ and the forward-rate formula:

$$\boxed{L_k(0; T_i, T_{i+1}) = \frac{1}{\tau_i^k}\left(\frac{P_k(0, T_i)}{P_k(0, T_{i+1})} - 1\right)}$$

**Intuition:**
- Discounting answers: "What is a dollar at $T$ worth today?"
- Projection answers: "What floating coupon is expected to be paid at $T$?"

**Trading / Risk / Portfolio Practice:**
- The provided sources explicitly discuss that, in collateralized markets, an overnight curve (e.g., Fed funds/OIS) is used for discounting, while a LIBOR curve is used for forward projection for LIBOR-linked cashflows.

---

### 6) Single-Curve vs Multi-Curve Worlds

#### Single-Curve (Legacy)

**Formal Definition:** One curve $P(0,T)$ is used for both discounting and for projecting forwards $L$ (i.e., forwards are implied from the same discount factors).

**Trading / Risk / Portfolio Practice:** The par swap rate admits a compact discount-factor formula $S = \frac{P(0,T_\alpha) - P(0,T_\beta)}{A}$.

#### Multi-Curve (Modern)

**Formal Definition:** Use a universal discount curve $P(0,T)$ plus multiple index-specific curves $P_k(0,T)$ for generating forward rates for each index $k$.

**Trading / Risk / Portfolio Practice:** The text motivates multiple curves because a single yield curve is not always consistent with market observations, especially under stress; separating discount and forward curves helps price linear instruments more consistently.

---

### 7) Par Swap Rate ("Swap Rate") and Fixed-Leg Annuity

**Formal Definition:** The par swap rate is the fixed rate that makes the swap PV equal to zero at inception.

**Intuition:** The fixed rate is the "fair" coupon so that PV(receive float) = PV(pay fixed).

**Trading / Risk / Portfolio Practice:** Traders quote swap rates by maturity (swap curve) and use annuities to convert between PV and rate risk (PVBP).

---

### 8) Swap Spread (Preview)

**Formal Definition:** Swap spread is often defined as a par swap rate minus a government (Treasury) yield of the same maturity.

**Intuition:** Swaps reference (historically) bank credit (e.g., LIBOR), so swap rates can exceed government yields; spreads also reflect market supply/demand and other forces.

**Trading / Risk / Portfolio Practice:** Swap spreads are used (carefully) as relative-value measures; they can move for reasons not purely "credit".

---

## Math and Derivations

### 2.1 Leg Cashflows (Deterministic Fixed, Stochastic Float)

**Fixed leg** (no principal exchange in plain-vanilla IRS):
$$\text{CF}_j^{\text{fix}} = N \, c \, \tau_j^{\text{fix}} \quad \text{paid at } T_j^{\text{fix}}$$

**Floating leg** (index $k$):
$$\text{CF}_{i+1}^{\text{flt}} = N \, \tau_i^{\text{flt}} \, L_k(T_i^{\text{flt}}; T_i^{\text{flt}}, T_{i+1}^{\text{flt}}) \quad \text{paid at } T_{i+1}^{\text{flt}}$$

**Unit check:**
- $c$ and $L_k$ are rates per year (dimensionless).
- $\tau$ is years (dimensionless year fraction).
- $c\tau$ and $L_k\tau$ are dimensionless.
- Multiply by $N$ ⇒ cashflow in currency units. ✓

---

### 2.2 Present Value with a Discount Curve

Discount each cashflow with discount factors $P_d(0,T)$:

**Fixed leg PV:**
$$\boxed{\text{PV}_{\text{fixed}}(0) = N \sum_{j=1}^{n_f} c \, \tau_j^{\text{fix}} \, P_d(0, T_j^{\text{fix}})}$$

This structure is explicit in the provided multi-curve swap valuation formula where fixed payments $c\tau$ are discounted with $P(t,\cdot)$.

**Floating leg PV (general, multi-curve):**
$$\boxed{\text{PV}_{\text{float}}(0) = N \sum_{i=0}^{n_p-1} \tau_i^{\text{flt}} \, \mathbb{E}[L_k(\cdot; T_i^{\text{flt}}, T_{i+1}^{\text{flt}})] \, P_d(0, T_{i+1}^{\text{flt}})}$$

In the provided reference, the swap value is written as a discounted expectation of floating payment minus fixed payment (with potentially different year fractions).

For vanilla swaps priced off a projection curve, we typically replace the expectation of the floating rate with the forward rate implied by the projection curve (see §2.6), under standard no-arbitrage assumptions.

---

### 2.3 Swap PV and Bond Replication Intuition

A common valuation decomposition is:

$$\text{Swap value} = (\text{value of receiving fixed leg}) - (\text{value of paying floating leg})$$

(receiver) or vice versa (payer).

Viewing a swap as "long fixed-rate bond, short floating-rate bond" is a convenient mental model; the notional is not exchanged in the swap, but the bond decomposition helps derive formulas.

---

### 2.4 Par Swap Rate (Single-Curve) and the Annuity Formula

In a single-curve world with aligned payment dates $\{T_i\}$ and year fractions $\tau_i$, the par swap rate (swap rate) over $[T_\alpha, T_\beta]$ is given by:

$$\boxed{S_{\alpha,\beta}(0) = \frac{P(0, T_\alpha) - P(0, T_\beta)}{\sum_{i=\alpha+1}^{\beta} \tau_i \, P(0, T_i)}}$$

This is the discount-factor/annuity formula stated in the reference text for swap rates and annuity factors.

**Interpretation:**
- **Numerator** $P(0,T_\alpha) - P(0,T_\beta)$: PV of a unit notional exchanged at $T_\alpha$ and returned at $T_\beta$ (in discount-factor terms).
- **Denominator** $A = \sum \tau_i P(0,T_i)$: fixed-leg annuity.

**Unit check:** Numerator and denominator are dimensionless, so $S$ is a rate (dimensionless per year). ✓

---

### 2.5 Floating-Leg PV Identity in Single-Curve (Telescoping Argument)

**Assumptions:**
- Spot-start swap with $T_0 = 0$
- Floating cashflow at $T_i$: $N \tau_i L_i$
- Forward rate implied from the same discount factors:
$$L_i = \frac{1}{\tau_i}\left(\frac{P(0, T_{i-1})}{P(0, T_i)} - 1\right)$$

**Derivation:**

$$\tau_i L_i P(0, T_i) = \left(\frac{P(0, T_{i-1})}{P(0, T_i)} - 1\right) P(0, T_i) = P(0, T_{i-1}) - P(0, T_i)$$

Summing from $i=1$ to $n$:

$$\sum_{i=1}^{n} \tau_i L_i P(0, T_i) = P(0, T_0) - P(0, T_n) = 1 - P(0, T_n)$$

**Result:**
$$\boxed{\frac{\text{PV}_{\text{float}}}{N} = 1 - P(0, T_n) \quad \text{(single-curve, spot-start)}}$$

This identity is consistent with the par-rate formula above and with the "floating-rate note is worth par on reset dates" intuition used in the swap valuation discussion.

---

### 2.6 Multi-Curve Valuation: Discounting vs Projection Separation

The provided multi-curve framework uses:
- A **universal discount curve** $P(0,T)$ for discounting, and
- An **index curve** $P_k(0,T)$ for generating forward rates for index $k$.

The reference gives the swap value (for a fixed-for-$k$-LIBOR swap, with same period grid for the shown formula) as:

$$V_{\text{swap}}^k(0) = \sum_{i=0}^{n-1} c \, \tau_i^k \, P(0, T_{i+1}) - \sum_{i=0}^{n-1} L_k(0; T_i, T_{i+1}) \, \tau_i^k \, P(0, T_{i+1})$$

with forward rates computed via:

$$L_k(0; T_i, T_{i+1}) = \frac{1}{\tau_i^k}\left(\frac{P_k(0, T_i)}{P_k(0, T_{i+1})} - 1\right)$$

**Practical generalization:** If fixed and float have different schedules/day-counts, you replace $\tau_i^k$ and dates accordingly (the reference notes such distinctions and writes a more general expectation form with separate year fractions).

**Par rate in multi-curve (general idea).** Set $V_{\text{swap}} = 0$ and solve for $c$:

$$\boxed{c_{\text{par}} = \frac{\sum_i \tau_i^{\text{flt}} \, L_k(0; T_i^{\text{flt}}, T_{i+1}^{\text{flt}}) \, P_d(0, T_{i+1}^{\text{flt}})}{\sum_j \tau_j^{\text{fix}} \, P_d(0, T_j^{\text{fix}})}}$$

**Key point:** $P_d$ (discount curve) enters both numerator and denominator through PV; $P_k$ (projection) enters through $L_k$.

---

### 2.7 Off-Market Swap PV as "(Coupon − Par) × Annuity"

Let $c_{\text{par}}$ be the par fixed rate under the same curves used for valuation. Then:

**Receiver PV:**
$$\boxed{V_{\text{recv}} = \text{PV}_{\text{fixed}} - \text{PV}_{\text{float}} = N(c - c_{\text{par}}) \, A^{\text{fix}}(0)}$$

where $A^{\text{fix}}(0) = \sum_j \tau_j^{\text{fix}} P_d(0, T_j^{\text{fix}})$.

**Payer PV** is the negative of this.

This is a direct algebraic consequence of the par condition and the linearity of PV.

---

### 2.8 Swap as a Strip of FRAs (Conceptual)

Sources note that swaps can be regarded as portfolios of forward contracts and (in a single-curve setting) are economically equivalent to a multi-period FRA structure, up to discounting details.

If fixed and floating payments occur on the same grid $\{T_i\}$, the payer swap's PV can be written as:

$$V_{\text{payer}} = N \sum_{i=1}^{n} \tau_i (F_i - c) \, P_d(0, T_i)$$

where $F_i$ are the forward rates used for the floating coupons (single-curve: implied by $P$; multi-curve: implied by $P_k$). Each term resembles the PV of a period-specific forward-rate agreement on $[T_{i-1}, T_i]$.

---

## Measurement & Risk (Preview for Chapter 25)

### 3.1 What Cashflows Depend On

**Fixed leg cashflows depend on:**
- Notional $N$
- Fixed rate $c$
- Accrual factors $\tau_j^{\text{fix}}$ (day count + schedule)

**Floating leg cashflows depend on:**
- Notional $N$
- Accrual factors $\tau_i^{\text{flt}}$
- Floating forward rates $L_k(\cdot)$ (projection curve/index curve)

### 3.2 What PV Depends On

Once cashflows are specified, PV depends on discount factors:
$$\text{PV} = \sum \text{CF}(T) \, P_d(0,T)$$

In multi-curve, forward rates come from $P_k$ but discounting uses $P_d$. This separation is explicit in the multi-curve swap valuation expression (discount factors $P(\cdot)$ and forward rates from $P_k(\cdot)$).

### 3.3 Which Curve Does What

| World | Projection | Discounting |
|-------|------------|-------------|
| **Single-curve (legacy)** | $P(0,T)$ implies $L$ | $P(0,T)$ discounts |
| **Multi-curve (modern)** | $P_k(0,T)$ implies $L_k$ | $P_d(0,T)$ discounts |

**Single-curve:** Par swap rate can be written purely in terms of $P(0,T)$ and an annuity.

**Multi-curve:** The text motivates multiple curves because a single yield curve is not always consistent with market observations, especially under stress; separating discount and forward curves helps price linear instruments more consistently.

**Collateralized market practice:** OIS/Fed funds curve as discount curve; LIBOR curve as forward curve.

### 3.4 Why Discounting vs Projection Separation Matters

If forward rates implied by the discount curve differ from the forward rates implied by the index/tenor curve, then:
- The floating leg PV cannot simplify to $1 - P_d(0,T)$
- The par fixed rate shifts (basis effect)
- Curve risk splits naturally into discount and projection components

The reference explicitly distinguishes a universal discount curve from index-specific curves and provides swap valuation in this framework.

### 3.5 Preview-Level Risk Decomposition: $\text{PV01}_{\text{disc}}$ vs $\text{PV01}_{\text{proj}}$

Define (for a given swap and a chosen bump methodology):

**Discount-curve PV01 (holding projection fixed):**
$$\text{PV01}_{\text{disc}} \equiv V(P_d + 1\text{bp}, P_k) - V(P_d, P_k)$$

*Interpretation:* Sensitivity to discounting (time value of money / collateral curve).

**Projection-curve PV01 (holding discount fixed):**
$$\text{PV01}_{\text{proj}} \equiv V(P_d, (P_k)_{+1\text{bp}}) - V(P_d, P_k)$$

*Interpretation:* Sensitivity to expected floating coupons (forward rates for the index).

These are natural "partial" risks in a multi-curve setup where discounting and forwarding curves are distinct.

**Note:** How you define "+1bp bump" (parallel bump to zero rates vs bump to discount factors vs bump to forwards) is a desk convention. I'm not sure which bump methodology you want; the worked example in §4 uses an explicit simple convention and labels it.

---

## Worked Examples

> **Note:** All examples are toy calculations for learning. They assume deterministic discount factors and forward rates implied as stated. Real swaps require calendar rules, stubs, business-day adjustments, payment lags, and index-specific fixing conventions. The sources explicitly warn that such complications exist.

---

### Example A: Swap Cashflow Schedule

**Goal:** Construct a simple fixed-for-floating schedule and compute accrual factors explicitly.

**Assumptions/conventions (illustrative):**
- Start date: 2026-01-01, maturity: 2027-01-01
- Floating leg: quarterly, day count ACT/360 (common for floating legs in LIBOR-style markets)
- Fixed leg: semiannual, day count 30/360 (common U.S. convention per source)
- Ignore business-day adjustments and payment lags (pedagogical)

**Floating schedule (quarterly):**

| Period | Dates | Days | $\tau$ (ACT/360) |
|--------|-------|------|------------------|
| 1 | 2026-01-01 → 2026-04-01 | 90 | $90/360 = 0.25$ |
| 2 | 2026-04-01 → 2026-07-01 | 91 | $91/360 \approx 0.25278$ |
| 3 | 2026-07-01 → 2026-10-01 | 92 | $92/360 \approx 0.25556$ |
| 4 | 2026-10-01 → 2027-01-01 | 92 | $92/360 \approx 0.25556$ |

**Fixed schedule (semiannual):**

| Period | Dates | $\tau$ (30/360) |
|--------|-------|-----------------|
| 1 | 2026-01-01 → 2026-07-01 | $180/360 = 0.5$ |
| 2 | 2026-07-01 → 2027-01-01 | $180/360 = 0.5$ |

**Sanity check:** Under ACT/360, a full calendar year is $365/360 \approx 1.0139$, matching the sum of quarterly $\tau$'s: $(90+91+92+92)/360 = 365/360$. ✓

---

### Example B: Par Swap Rate from Discount Factors (Single-Curve)

**Goal:** Given discount factors $P(0,T)$, compute the fixed-leg annuity and the par fixed rate $c_{\text{par}}$ for a spot-start 1Y swap.

**Assumptions:**
- Simplified year fractions: Fixed semiannual $\tau^{\text{fix}} = \{0.5, 0.5\}$; Floating quarterly $\tau^{\text{flt}} = \{0.25, 0.25, 0.25, 0.25\}$
- Single-curve: same curve for discounting and projecting forwards

**Given discount factors:**
- $P(0, 0.5) = 0.9850$
- $P(0, 1.0) = 0.9700$

**Step 1: Fixed-leg annuity**
$$A^{\text{fix}}(0) = 0.5 \times P(0, 0.5) + 0.5 \times P(0, 1.0) = 0.5(0.9850) + 0.5(0.9700) = 0.9775$$

**Step 2: Floating-leg PV (single-curve identity for spot-start)**

From the telescoping identity (§2.5):
$$\frac{\text{PV}_{\text{float}}}{N} = 1 - P(0, 1.0) = 1 - 0.9700 = 0.0300$$

**Step 3: Par fixed rate**

Set PV fixed = PV float:
$$c_{\text{par}} = \frac{1 - P(0, 1.0)}{A^{\text{fix}}(0)} = \frac{0.0300}{0.9775} = 0.030690537 \approx 3.0691\%$$

**Unit check:** $0.0300$ and $0.9775$ are dimensionless ⇒ $c_{\text{par}}$ is a rate. ✓

---

### Example C: Floating Leg PV Identity (Single-Curve)

**Goal:** Compute floating PV directly from forwards implied by discount factors and verify it equals $1 - P(0,1)$.

**Assumptions:** Same as Example B (single curve, quarterly float).

**Given discount factors (quarterly grid):**
- $P(0, 0.25) = 0.9925$
- $P(0, 0.50) = 0.9850$
- $P(0, 0.75) = 0.9775$
- $P(0, 1.00) = 0.9700$

**Step 1: Compute quarterly forwards implied by discount factors**

$$L_i = \frac{1}{0.25}\left(\frac{P(0, T_{i-1})}{P(0, T_i)} - 1\right)$$

| Period | Forward Rate |
|--------|--------------|
| $L_{0\to0.25}$ | $\frac{1}{0.25}\left(\frac{1}{0.9925} - 1\right) = 0.0302267$ |
| $L_{0.25\to0.50}$ | $\frac{1}{0.25}\left(\frac{0.9925}{0.9850} - 1\right) = 0.0304569$ |
| $L_{0.50\to0.75}$ | $\frac{1}{0.25}\left(\frac{0.9850}{0.9775} - 1\right) = 0.0306905$ |
| $L_{0.75\to1.00}$ | $\frac{1}{0.25}\left(\frac{0.9775}{0.9700} - 1\right) = 0.0309278$ |

**Step 2: PV the floating coupons**

Each coupon PV contribution: $\text{PV}_i/N = 0.25 \times L_i \times P(0, T_i)$

| $T$ | Calculation | Result |
|-----|-------------|--------|
| 0.25 | $0.25 \times 0.0302267 \times 0.9925$ | $0.0075$ |
| 0.50 | $0.25 \times 0.0304569 \times 0.9850$ | $0.0075$ |
| 0.75 | $0.25 \times 0.0306905 \times 0.9775$ | $0.0075$ |
| 1.00 | $0.25 \times 0.0309278 \times 0.9700$ | $0.0075$ |

Sum: $\text{PV}_{\text{float}}/N = 4 \times 0.0075 = 0.0300$

**Step 3: Compare to identity**
$$1 - P(0, 1.0) = 1 - 0.9700 = 0.0300$$

**Conclusion:** PV matches exactly (up to rounding), illustrating the single-curve telescoping result. ✓

---

### Example D: Value an Off-Market Swap

**Goal:** Value a swap where the fixed rate is not par.

**Assumptions:** Same curve and schedule as Examples B–C; notional $N = 100{,}000{,}000$.

- Par fixed rate from Example B: $c_{\text{par}} \approx 3.0691\%$
- Off-market fixed rate: $c = 3.20\% = 0.032$

**Step 1: PV of floating leg**

Using identity (single curve, spot-start):
$$\text{PV}_{\text{float}} = N(1 - P(0,1)) = 100{,}000{,}000 \times 0.0300 = 3{,}000{,}000$$

**Step 2: PV of fixed leg**

Fixed-leg annuity: $A^{\text{fix}}(0) = 0.9775$
$$\text{PV}_{\text{fixed}} = N \cdot c \cdot A^{\text{fix}}(0) = 100{,}000{,}000 \times 0.032 \times 0.9775 = 3{,}128{,}000$$

*Intermediate coupon-PV view:*
- Coupon at 0.5: $N c \tau = 100\text{m} \times 0.032 \times 0.5 = 1.6\text{m}$; PV $= 1.6\text{m} \times 0.9850 = 1.576\text{m}$
- Coupon at 1.0: PV $= 1.6\text{m} \times 0.9700 = 1.552\text{m}$
- Sum $= 3.128\text{m}$ ✓

**Step 3: Swap NPV**

- **Receiver** (receive fixed, pay float): $V_{\text{recv}} = 3.128\text{m} - 3.000\text{m} = +128{,}000$
- **Payer** (pay fixed, receive float): $V_{\text{payer}} = -128{,}000$

**Sign interpretation:** Fixed is "rich" vs par, so receiving fixed is valuable.

---

### Example E: Swap as a Strip of FRAs

**Goal:** Show that a swap can be decomposed into period-by-period forward-rate exposures (FRA-like).

**Assumptions (for exact grid alignment):**
- Quarterly payments for both legs at $T = \{0.25, 0.50, 0.75\}$ (toy)
- Same single curve as Example C
- Notional $N = 1$
- Fixed rate $c = 3.10\% = 0.031$

**Given:**
- Discount factors: $P(0, 0.25) = 0.9925$, $P(0, 0.50) = 0.9850$, $P(0, 0.75) = 0.9775$
- Forwards (from Example C): $L_1 = 0.0302267$, $L_2 = 0.0304569$, $L_3 = 0.0306905$

**Step 1: PV of floating and fixed legs**

*Floating PV (single curve):*
$$\text{PV}_{\text{float}}/N = 1 - P(0, 0.75) = 1 - 0.9775 = 0.0225$$

*Fixed PV:*
$$\text{PV}_{\text{fixed}}/N = c \sum_{i=1}^{3} (0.25 \times P(0, T_i)) = 0.031 \times 0.25(0.9925 + 0.9850 + 0.9775)$$

Annuity-like sum: $0.25(0.9925 + 0.9850 + 0.9775) = 0.73875$

Hence: $\text{PV}_{\text{fixed}}/N = 0.031 \times 0.73875 = 0.02290125$

*Payer PV:*
$$V_{\text{payer}} = 0.0225 - 0.02290125 = -0.00040125$$

**Step 2: FRA-like strip (net cashflow per period)**

For each period $i$, net cashflow at $T_i$:
$$\text{NetCF}_i/N = 0.25(L_i - c)$$

PV contribution: $\text{PV}_i/N = 0.25(L_i - c) \times P(0, T_i)$

| $i$ | Calculation | Result |
|-----|-------------|--------|
| 1 | $0.25(0.0302267 - 0.031) \times 0.9925$ | $-0.000191875$ |
| 2 | $0.25(0.0304569 - 0.031) \times 0.9850$ | $-0.000133750$ |
| 3 | $0.25(0.0306905 - 0.031) \times 0.9775$ | $-0.000075625$ |

Sum: $\sum \text{PV}_i/N = -0.00040125$

This matches the payer PV above. ✓

**Conceptual link to sources:** Swaps can be viewed as portfolios of forward contracts / FRA-like exposures.

---

### Example F: Single-Curve vs Multi-Curve Pricing Difference

**Goal:** Show how using a projection curve different from the discount curve changes the par fixed rate.

**Setup:** 1Y swap, quarterly float, semiannual fixed ($\tau^{\text{flt}} = 0.25$, $\tau^{\text{fix}} = 0.5$).

**Given curves (toy):**

| $T$ | OIS Discount $P_d$ | 3M Projection $P_{3M}$ |
|-----|-------------------|------------------------|
| 0.25 | 0.9940 | 0.9930 |
| 0.50 | 0.9880 | 0.9860 |
| 0.75 | 0.9820 | 0.9790 |
| 1.00 | 0.9760 | 0.9720 |

This matches the multi-curve framework in which forwards for index $k$ come from $P_k$ while discounting uses $P_d$.

#### (i) Single-curve par rate (use OIS curve for both)

*Fixed-leg annuity:*
$$A_d^{\text{fix}} = 0.5 \times P_d(0, 0.5) + 0.5 \times P_d(0, 1.0) = 0.5(0.9880) + 0.5(0.9760) = 0.9820$$

*Single-curve float PV identity:*
$$\text{PV}_{\text{float}}/N = 1 - P_d(0, 1.0) = 1 - 0.9760 = 0.0240$$

*Par fixed rate:*
$$c_{\text{par}}^{\text{single}} = \frac{0.0240}{0.9820} = 0.024440 \approx 2.4440\%$$

#### (ii) Multi-curve par rate (discount with OIS, project with 3M)

**Step 1: Compute 3M forwards from projection curve**

$$L_i^{3M} = \frac{1}{0.25}\left(\frac{P_{3M}(0, T_{i-1})}{P_{3M}(0, T_i)} - 1\right)$$

| Period | $L^{3M}$ |
|--------|----------|
| $0 \to 0.25$ | $0.0281974$ |
| $0.25 \to 0.50$ | $0.0283976$ |
| $0.50 \to 0.75$ | $0.0286006$ |
| $0.75 \to 1.00$ | $0.0288066$ |

**Step 2: PV floating coupons using OIS discount factors**

$$\text{PV}_{\text{float}}^{\text{multi}}/N = \sum_{i=1}^{4} 0.25 \times L_i^{3M} \times P_d(0, T_i)$$

| $T$ | Calculation | Result |
|-----|-------------|--------|
| 0.25 | $0.25 \times 0.0281974 \times 0.9940$ | $0.00700705$ |
| 0.50 | $0.25 \times 0.0283976 \times 0.9880$ | $0.00701420$ |
| 0.75 | $0.25 \times 0.0286006 \times 0.9820$ | $0.00702145$ |
| 1.00 | $0.25 \times 0.0288066 \times 0.9760$ | $0.00702881$ |

Sum: $\text{PV}_{\text{float}}^{\text{multi}}/N = 0.02807151$

**Step 3: Par fixed rate**
$$c_{\text{par}}^{\text{multi}} = \frac{0.02807151}{0.9820} = 0.028586 \approx 2.8586\%$$

#### Comparison (Basis Effect)

| Method | Par Rate |
|--------|----------|
| Single-curve | 2.4440% |
| Multi-curve | 2.8586% |
| **Difference** | **41.46 bp** |

**Interpretation:** The projection curve implies higher forwards than those implied by the OIS discount curve; in a multi-curve world you must pay a higher fixed rate to match the (higher) projected floating coupons when discounting on OIS.

---

### Example G: Discount vs Projection Bump Impact

**Goal:** For an off-market swap, compute PV change under:
1. +1bp parallel bump to discount curve (projection fixed)
2. +1bp parallel bump to projection curve (discount fixed)

**Swap setup (same as Example F):**
- Notional $N = 100{,}000{,}000$
- Fixed rate $c = 3.10\% = 0.031$ (off-market vs par 2.8586%)
- Discount curve: OIS $P_d$ from Example F
- Projection curve: 3M $P_{3M}$ from Example F

#### Base PV

*Fixed PV:*
$$\text{PV}_{\text{fixed}}/N = c \times A_d^{\text{fix}} = 0.031 \times 0.9820 = 0.030442$$

*Floating PV (from Example F):*
$$\text{PV}_{\text{float}}^{\text{multi}}/N = 0.02807151$$

*Payer PV:*
$$V_{\text{payer}}/N = 0.02807151 - 0.030442 = -0.00237049$$
$$V_{\text{payer}} \approx -237{,}049$$

#### (1) +1bp bump to discount curve (projection forwards held fixed)

**Bump convention (explicit):** Approximate a +1bp parallel bump to continuously-compounded zero rates by scaling discount factors as:
$$P_d^+(0,T) \approx P_d(0,T)(1 - 0.0001 \times T)$$
(first-order approximation, good for small bumps)

*Bumped discount factors:*

| $T$ | $P_d^+$ |
|-----|---------|
| 0.25 | 0.993975 |
| 0.50 | 0.987951 |
| 0.75 | 0.981926 |
| 1.00 | 0.975902 |

*Recompute PVs (forwards unchanged):*
- $\text{PV}_{\text{float}}^+/N = 0.02806975$
- $\text{PV}_{\text{fixed}}^+/N = 0.031 \times 0.5(0.987951 + 0.975902) = 0.03043972$

*New payer PV:*
$$V_{\text{payer}}^+/N = 0.02806975 - 0.03043972 = -0.00236997$$

**Discount bump PV change:**
$$\Delta V_{\text{disc}}/N = (-0.00236997) - (-0.00237049) = +5.23 \times 10^{-7}$$
$$\Delta V_{\text{disc}} \approx +52$$

#### (2) +1bp bump to projection curve (discount held fixed)

**Bump convention (explicit):** Add $+0.0001$ to each quarterly forward rate used to project coupons, leaving discount factors unchanged.

*PV change in floating leg per unit notional:*
$$\Delta(\text{PV}_{\text{float}})/N = \sum_{i=1}^{4} 0.25 \times 0.0001 \times P_d(0, T_i)$$
$$= 0.25 \times 0.0001 \times (0.9940 + 0.9880 + 0.9820 + 0.9760) = 0.0000985$$

Fixed leg unchanged ⇒ payer PV change:
$$\Delta V_{\text{proj}}/N = +0.0000985$$
$$\Delta V_{\text{proj}} \approx +9{,}850$$

#### Compare Magnitudes/Signs (Preview Interpretation)

| Bump | $\Delta V$ |
|------|------------|
| Discount +1bp | $\approx +52$ |
| Projection +1bp | $\approx +9{,}850$ |

- Bumping projection forwards raises floating receipts ⇒ payer PV increases (positive $\Delta V$).
- Discount bump affects both legs' PVs and may partially cancel, yielding a smaller net impact for short maturities and near-par structures.

This illustrates why multi-curve risk naturally splits into discount vs projection components.

---

### Example H: Day Count / Frequency Sensitivity

**Goal:** Show that quoting conventions (frequency/day count) affect annuity and thus the par swap rate.

**Use single-curve data from Example B:**
- $P(0, 0.5) = 0.9850$, $P(0, 1.0) = 0.9700$
- $\text{PV}_{\text{float}}/N = 1 - P(0,1) = 0.0300$

#### Case 1: Semiannual fixed (two coupons), $\tau^{\text{fix}} = \{0.5, 0.5\}$

$$A^{\text{fix}} = 0.5(0.9850) + 0.5(0.9700) = 0.9775$$
$$c_{\text{par}} = \frac{0.0300}{0.9775} = 0.030691 \approx 3.0691\%$$

#### Case 2: Annual fixed (one coupon), $\tau^{\text{fix}} = \{1.0\}$

$$A^{\text{fix}} = 1.0 \times 0.9700 = 0.9700$$
$$c_{\text{par}} = \frac{0.0300}{0.9700} = 0.030928 \approx 3.0928\%$$

**Difference:** $3.0928\% - 3.0691\% \approx 2.37$ bp

**Interpretation:** Changing the fixed-leg payment frequency changes the annuity (PVBP) and therefore changes the par coupon that equates PVs.

**Day-count note:** The sources emphasize that day-count conventions differ across legs and markets (e.g., fixed may be 30/360 or ACT/365; float often ACT/360).

---

### Example I: Swap Spread Intuition (Light Preview)

**Goal:** Compute a toy swap spread and interpret at a high level.

Take the multi-curve par swap rate from Example F for 1Y:
$$c_{\text{par}}^{\text{multi}} \approx 2.8586\%$$

Assume a toy 1Y Treasury/par government yield of:
$$y_{\text{gov}} = 2.65\%$$

**Swap spread (toy):**
$$\text{SwapSpread} = c_{\text{par}} - y_{\text{gov}} = 2.8586\% - 2.65\% = 0.2086\% \approx 20.86 \text{ bp}$$

**Interpretation (kept short):**
- The source notes that swap rates (historically tied to bank credit via floating indices) should exceed government yields, motivating positive swap spreads; spreads also reflect supply/demand and other market-specific influences.
- This example is purely illustrative; real swap spreads depend on the exact government benchmark, on/off-the-run effects, and the swap discounting/collateral framework.

---

### Example J: Collateral Discounting Intuition (Light)

**Goal:** Hold projected floating cashflows fixed but change the discount curve; show PV changes.

The sources state that markets consider OIS/Fed funds curves appropriate for discounting collateralized deals, with LIBOR used for forward projection.

**Use the off-market swap from Example G:**
- Same projected forwards from 3M curve
- Fixed $c = 3.10\%$
- $N = 100\text{m}$

**Projection forwards:** $L^{3M}$ (unchanged)

#### Case 1 (OIS discounting): Discount with $P_d$ from Example F

We already computed:
$$V_{\text{payer}}^{\text{OIS}}/N = -0.00237049 \Rightarrow V_{\text{payer}}^{\text{OIS}} \approx -237{,}049$$

#### Case 2 (non-OIS discounting): Discount with a "LIBOR-like" curve $P_L$

Here set equal to the projection curve discount factors:
- $P_L(0, 0.25) = 0.9930$, $P_L(0, 0.50) = 0.9860$
- $P_L(0, 0.75) = 0.9790$, $P_L(0, 1.00) = 0.9720$

**Step 1: PV float (discount with $P_L$, forwards unchanged)**
$$\text{PV}_{\text{float}}^{(L)}/N = 1 - P_L(0, 1.0) = 1 - 0.9720 = 0.0280$$

(Indeed equals the telescoping identity since forwards were implied from the same curve.)

**Step 2: PV fixed (discount with $P_L$)**
$$\text{PV}_{\text{fixed}}^{(L)}/N = 0.031 \times 0.5(0.9860 + 0.9720) = 0.030349$$

**Step 3: Payer PV**
$$V_{\text{payer}}^{(L)}/N = 0.0280 - 0.030349 = -0.002349$$
$$V_{\text{payer}}^{(L)} \approx -234{,}900$$

#### Comparison

$$V_{\text{payer}}^{(L)} - V_{\text{payer}}^{\text{OIS}} \approx (-234{,}900) - (-237{,}049) = +2{,}149$$

**Interpretation:** Changing only the discount curve (OIS vs non-OIS) changes PV even when projected floating cashflows are held fixed—this is the basic intuition behind collateral discounting effects.

---

## Practical Notes

### 5.1 Market Convention Checklist

| Convention | Status |
|------------|--------|
| **Fixed-leg frequency and day count** | Sources indicate typical differences by market/currency and highlight common conventions like fixed 30/360 (U.S.) vs other possibilities like ACT/365. |
| **Floating-leg accrual and index definition** | Floating often ACT/360 in LIBOR-style markets; rate set in advance, paid in arrears. |
| **Payment lags, stubs, business-day adjustments** | I'm not sure: the provided excerpts acknowledge complications (date adjustments, differences between year-fraction periods and forward-rate definitions) but do not give a full operational convention set for a particular currency/index. To be certain, we need: currency, index (e.g., USD 3M), documentation standard (ISDA, local conventions), and your desk's schedule generation rules. |

### 5.2 Common Pitfalls

1. **Confusing payer vs receiver PV sign:**
   - Use a consistent convention: payer = PV(float) − PV(fixed); receiver = opposite.

2. **Using a par swap rate formula with inconsistent curves:**
   - Single-curve formulas (telescoping, $1-P$) rely on using the same curve for projecting forwards and discounting.

3. **Mixing projection forwards with discounting-implied forwards in multi-curve:**
   - In multi-curve, forward rates come from $P_k$, discounting from $P$.

4. **Day count mismatch:**
   - Fixed and float legs can use different day counts; miscomputing $\tau$ directly changes PV and par rate.

### 5.3 Implementation Pitfalls

1. **Schedule generation and stub handling:**
   - I'm not sure: not fully specified in sources; must follow desk conventions and legal confirmation.

2. **Consistent accrual factors across legs:**
   - Ensure each cashflow uses the correct $\tau$ for its leg and convention.

3. **Consistent curve inputs:**
   - Ensure you discount every cashflow with the correct discount curve; project floating rates with the correct tenor/index curve.

### 5.4 Verification Tests

1. **Par swap PV test:**
   - If you compute $c_{\text{par}}$ from your chosen curves, then valuing the swap with $c = c_{\text{par}}$ should give PV $\approx 0$ (numerical tolerances aside).

2. **Floating-leg PV identity check (single-curve only):**
   - Verify $\sum \tau_i L_i P(0, T_i) \approx 1 - P(0, T_n)$ when forwards and discounting come from the same curve.

3. **Small-bump stability:**
   - PV changes should be approximately linear for small bumps; large nonlinearities suggest implementation errors.

---

## Summary & Recall

### 10-Bullet Executive Summary

1. A vanilla interest rate swap exchanges fixed coupons for floating coupons on a notional $N$, typically without exchanging principal.

2. The swap rate (par fixed rate) is the fixed rate that makes the swap PV zero at initiation.

3. Fixed-leg cashflows are deterministic given $c$ and $\tau^{\text{fix}}$; floating-leg cashflows depend on forward rates and $\tau^{\text{flt}}$.

4. PV depends on discount factors: $\text{PV} = \sum \text{CF}(T) P_d(0,T)$.

5. In single-curve valuation, par swap rates admit the discount-factor annuity formula $S = \frac{P(0,T_\alpha) - P(0,T_\beta)}{A}$.

6. In single-curve valuation for a spot-start swap, floating coupon PV telescopes to $1 - P(0,T)$ under standard assumptions.

7. In multi-curve valuation, discounting uses a universal curve $P$, while floating forwards come from index-specific curves $P_k$.

8. Separation of discounting and projection matters: it changes par rates and PV and produces basis effects.

9. A swap can be interpreted as a portfolio of forward-rate exposures (FRA-like), aiding intuition and risk decomposition.

10. Swap spreads (swap rate minus government yield) reflect multiple forces, including credit and market supply/demand, and should be interpreted with care.

---

### Cheat Sheet of Formulas

| Formula | Expression |
|---------|------------|
| **Fixed leg PV** | $\text{PV}_{\text{fixed}}(0) = N \sum_j c \, \tau_j^{\text{fix}} \, P_d(0, T_j^{\text{fix}})$ |
| **Floating leg PV (multi-curve)** | $\text{PV}_{\text{float}}(0) = N \sum_i \tau_i^{\text{flt}} \, L_k(0; T_i, T_{i+1}) \, P_d(0, T_{i+1})$ |
| **Forward rate from projection curve** | $L_k(0; T_i, T_{i+1}) = \frac{1}{\tau_i^k}\left(\frac{P_k(0, T_i)}{P_k(0, T_{i+1})} - 1\right)$ |
| **Swap PV** | Payer: $V_{\text{payer}} = \text{PV}_{\text{float}} - \text{PV}_{\text{fixed}}$; Receiver: $V_{\text{recv}} = -V_{\text{payer}}$ |
| **Par fixed rate (general)** | $c_{\text{par}} = \frac{\text{PV}_{\text{float}}/N}{A^{\text{fix}}(0)}$ |
| **Par swap rate (single-curve)** | $S_{\alpha,\beta}(0) = \frac{P(0, T_\alpha) - P(0, T_\beta)}{\sum_{i=\alpha+1}^{\beta} \tau_i P(0, T_i)}$ |
| **Single vs multi-curve** | Single: one curve projects and discounts. Multi: $P_d$ discounts; $P_k$ projects. |

---

### Flashcards (30 Q/A Pairs)

1. **Q:** What is the "swap rate"?
   **A:** The fixed rate that makes a new swap's PV equal to zero (par).

2. **Q:** In a payer swap, which leg do you pay?
   **A:** Pay fixed, receive floating.

3. **Q:** Write payer swap PV in terms of leg PVs.
   **A:** $V_{\text{payer}} = \text{PV}_{\text{float}} - \text{PV}_{\text{fixed}}$

4. **Q:** What is the fixed-leg "annuity"?
   **A:** $A^{\text{fix}}(0) = \sum_j \tau_j^{\text{fix}} P_d(0, T_j^{\text{fix}})$

5. **Q:** Units of $c$?
   **A:** Annualized rate (decimal per year).

6. **Q:** Units of $\tau$?
   **A:** Year fraction (dimensionless).

7. **Q:** Units of discount factor $P(0,T)$?
   **A:** Dimensionless.

8. **Q:** Formula for a forward money-market rate from curve discount factors?
   **A:** $L = \frac{1}{\tau}\left(\frac{P(0, T_i)}{P(0, T_{i+1})} - 1\right)$

9. **Q:** What does the discount curve do?
   **A:** Converts future cashflows to present value.

10. **Q:** What does the projection curve do?
    **A:** Generates forward rates for floating coupons.

11. **Q:** In single-curve, how is par swap rate expressed?
    **A:** $S = \frac{P(0, T_\alpha) - P(0, T_\beta)}{A}$

12. **Q:** In spot-start single-curve, PV(float coupons)/$N$ equals what?
    **A:** $1 - P(0,T)$ under standard assumptions.

13. **Q:** Why might the floating PV identity fail in multi-curve?
    **A:** Forwards come from $P_k$ while discounting uses $P_d$, so telescoping breaks.

14. **Q:** What is "OIS discounting"?
    **A:** Using an overnight-index curve (e.g., OIS/Fed funds) as the discount curve, common for collateralized deals.

15. **Q:** What is "index-discounting basis"?
    **A:** Differences between discounting curve and an index's forwarding curve (conceptual).

16. **Q:** What is the typical timing of floating coupons?
    **A:** Rate fixed at period start, paid at period end.

17. **Q:** Give one common floating day count in LIBOR markets.
    **A:** ACT/360.

18. **Q:** Give one common fixed day count mentioned for U.S. IRS.
    **A:** 30/360.

19. **Q:** How do day count conventions affect swaps?
    **A:** They change $\tau$, thus changing cashflows and PV.

20. **Q:** How does frequency affect par rates?
    **A:** Frequency changes the annuity, shifting $c_{\text{par}}$.

21. **Q:** What is the "bond replication" view of a swap?
    **A:** Long fixed-rate bond, short floating-rate bond (valuation convenience).

22. **Q:** What is $\text{PV01}_{\text{disc}}$?
    **A:** PV change under 1bp bump to discount curve, projection held fixed.

23. **Q:** What is $\text{PV01}_{\text{proj}}$?
    **A:** PV change under 1bp bump to projection curve, discount held fixed.

24. **Q:** When would $\text{PV01}_{\text{proj}}$ be large?
    **A:** When PV is sensitive to forwards (large floating-cashflow exposure).

25. **Q:** What does "par swap" mean operationally?
    **A:** Fixed rate set so PV ≈ 0 at inception (using the same curves for par computation and valuation).

26. **Q:** Off-market receiver PV formula?
    **A:** $V_{\text{recv}} = N(c - c_{\text{par}})A^{\text{fix}}(0)$ (derived).

27. **Q:** What is a swap spread (preview)?
    **A:** Swap rate minus government yield of same maturity (toy definition).

28. **Q:** Why can swap spreads move besides "credit"?
    **A:** Supply/demand and benchmark effects, among others.

29. **Q:** Why separate discount and projection curves?
    **A:** Single curve may not fit market quotes; separation supports consistent pricing and basis effects.

30. **Q:** What additional info is needed to lock conventions for a real swap?
    **A:** Currency, index/tenor, collateral/CSA, schedule rules (holidays, stubs, lags), and quoting conventions.

---

## Mini Problem Set

### Questions (Increasing Difficulty)

**Q1.** A 1Y spot-start single-curve swap has $P(0,1) = 0.965$. What is $\text{PV}_{\text{float}}/N$ (floating coupons only) under the standard single-curve assumptions?

**Q2.** Using Q1, suppose fixed leg is annual with $\tau = 1$ and discount factor $P(0,1) = 0.965$. What is the par fixed rate?

**Q3.** Same as Q2 but fixed leg is semiannual with $P(0, 0.5) = 0.982$, $P(0, 1) = 0.965$, $\tau = \{0.5, 0.5\}$. Compute $A$ and $c_{\text{par}}$.

**Q4.** For Q3, if the swap fixed rate is 20bp above par, compute the receiver PV per unit notional.

**Q5.** A payer swap has $N = 50\text{m}$, $A^{\text{fix}} = 4.2$ (for a longer maturity), and $c$ is 10bp above par. Approximate PV.

**Q6.** Given discount factors $P_d(0, T_i)$ and projection forwards $F_i$ for each floating period, write the general multi-curve par rate formula.

**Q7.** Explain why the floating PV identity $1 - P_d(0,T)$ generally fails in a multi-curve world.

**Q8.** Define $\text{PV01}_{\text{disc}}$ and $\text{PV01}_{\text{proj}}$ for a multi-curve swap.

**Q9.** For a payer swap, which direction does PV move if the projection curve shifts up (higher forwards), holding discounting fixed? (Qualitative.)

**Q10.** For a receiver swap, which direction does PV move if discount rates rise (discount factors fall), holding projected cashflows fixed? (Qualitative.)

**Q11.** Show algebraically that, holding $\text{PV}_{\text{float}}$ fixed, the receiver PV is linear in $c$ with slope $N A^{\text{fix}}$.

**Q12.** A swap has quarterly float and semiannual fixed. Describe how you would compute $c_{\text{par}}$ in a multi-curve setting (inputs and steps; no bootstrapping).

**Q13.** Suppose you mistakenly discount floating cashflows using the projection curve discount factors instead of the discount curve. Explain the conceptual error.

**Q14.** Construct a 2-period swap on dates $T_1, T_2$ with matching fixed/float dates and show its payer PV equals the sum of two FRA-like PVs.

**Q15.** Describe (qualitatively) how collateralization and CSA terms can affect swap discounting choices.

**Q16.** You observe a swap spread turning negative. List three possible non-credit explanations (qualitative, desk-style).

---

### Solution Sketches (Q1–Q8)

**Q1.** Use $1 - P(0,1) \Rightarrow 1 - 0.965 = 0.035$.

**Q2.** $A = P(0,1) = 0.965$. $c_{\text{par}} = (1-P)/A = 0.035/0.965 \approx 3.627\%$.

**Q3.** $A = 0.5(0.982) + 0.5(0.965) = 0.9735$. Then $c_{\text{par}} = (1 - 0.965)/0.9735 = 0.035/0.9735 \approx 3.595\%$.

**Q4.** $V_{\text{recv}}/N = (c - c_{\text{par}})A$ with $c - c_{\text{par}} = 0.002$. So $V_{\text{recv}}/N = 0.002 \times 0.9735 = 0.001947$.

**Q5.** $V_{\text{payer}} = -N(c - c_{\text{par}})A = -50\text{m} \times 0.001 \times 4.2 = -210{,}000$.

**Q6.** $c_{\text{par}} = \frac{\sum_i \tau_i^{\text{flt}} F_i P_d(0, T_{i+1})}{\sum_j \tau_j^{\text{fix}} P_d(0, T_j^{\text{fix}})}$

**Q7.** Identity relies on forward rates implied from the same discount factors used for discounting; in multi-curve, forwards come from $P_k$ while discounting uses $P_d$, so the telescoping cancellation does not hold.

**Q8.** $\text{PV01}_{\text{disc}} = V(P_d + 1\text{bp}, P_k) - V(P_d, P_k)$; $\text{PV01}_{\text{proj}} = V(P_d, (P_k)_{+1\text{bp}}) - V(P_d, P_k)$.

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

| Fact | Source |
|------|--------|
| Swap rate makes PV = 0 at initiation | Hull Ch 7, Andersen Vol 1 Ch 6 |
| Swap = long fixed bond, short floating bond (valuation decomposition) | Hull Ch 7 |
| Floating rate set in advance, paid in arrears | Hull Ch 7, Andersen Vol 1 |
| Fixed and floating legs often differ in frequency/day count | Hull Ch 7, Andersen Vol 1 Ch 6 |
| Single-curve par swap rate formula $S = (P_\alpha - P_\beta)/A$ | Andersen Vol 1, Tuckman Ch 18 |
| Multi-curve framework: universal discount curve + index curves for forwards | Andersen Vol 1 Ch 6 |
| OIS curve for discounting collateralized deals; LIBOR for projection | Andersen Vol 1 Ch 6, Hull Ch 9 |
| Swap spread = swap rate − government yield | Tuckman Ch 18 |

### (B) Reasoned Inference — Note Derivation Logic

| Inference | Derivation |
|-----------|------------|
| Par rate = PV(float)/Annuity | Algebraic consequence of setting $V = 0$ and solving for $c$ |
| Single-curve floating PV telescopes to $1 - P(0,T)$ | Term-by-term cancellation when forwards and discounting use same curve |
| Curve-specific PV01 decomposition | Follows from multi-curve valuation separating $P_d$ and $P_k$ |
| Off-market PV = $(c - c_{\text{par}}) \times N \times A$ | Linear algebra from leg PV formulas |

### (C) Speculation — Flag Uncertainties

| Topic | Uncertainty |
|-------|-------------|
| Desk-specific conventions (business-day adjustments, payment lags, stub treatment, RFR compounding) | I'm not sure — sources acknowledge complications but do not provide complete operational specifications for specific currencies/indices |
| Bump methodology for PV01 (parallel bump to zero rates vs DFs vs forwards) | I'm not sure which convention is desired; example uses explicit first-order approximation |
| Schedule generation details | I'm not sure — must follow desk conventions and legal confirmation |

---

*Chapter 25 complete.*
