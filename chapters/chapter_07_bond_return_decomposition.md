# Chapter 7: Bond Return Decomposition — Carry, Rolldown, Curve Moves, and Spread Changes

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- Clean vs dirty / accrued interest: full (dirty) price equals clean price plus accrued interest; accrued interest is proportional to the fraction of the coupon period elapsed (Tuckman Ch 1-4)
- Repo-funded P&L identity and carry definition: for a position financed in repo, P&L can be written as price change (clean) plus carry, where carry includes coupon/interest income and financing cost; repo interest uses a 360-day basis in that expression (Tuckman Ch 3, 8)
- DV01: defined as $\text{DV01} = -\frac{1}{10000}\frac{\partial P}{\partial y}$, i.e., the dollar (price-point) change for a 1bp change in yield under the yield-based pricing function $P(y)$ (Tuckman Ch 5-6)
- Duration/convexity 2nd-order approximation: percentage price change can be approximated by $\Delta P/P \approx -D\,\Delta y + \frac{1}{2}C(\Delta y)^2$ (Tuckman Ch 5)
- OAS: constant spread added (in a lattice setting) so model price matches market price; DVOAS is the dollar value of a 0.01 (i.e., 1bp) change in OAS (Tuckman Ch 4, 17-18)
- Z-spread / ZVS: defined as a spread added to the benchmark discount rates so discounted cashflows equal the bond price; sources note compounding-convention variants (discrete vs continuous) (Tuckman Ch 4)

### (B) Reasoned Inference (Derived from A)

- We define rolldown operationally as a counterfactual reprice at the horizon date using the initial curve/spread, with cashflow times shortened
- We build an exact telescoping decomposition of the clean price change using intermediate "what-if" prices (unchanged curve/spread, curve-changed-only, spread-changed)
- We interpret the difference between exact repricing and DV01-based approximations as convexity and cross-term residuals, consistent with the 2nd-order expansions in the sources

### (C) Speculation (Clearly Labeled; Minimal)

- **Terminology warning:** Many desks use "rolldown" to mean "aging along the curve" and sometimes reserve "pull-to-par" for the time-decay of premium/discount. The sources discuss the mechanics (e.g., pull-to-par) but do not appear to impose a single industry-wide definition of "rolldown." I therefore adopt an explicit operational definition in this chapter and flag alternatives in Practical Notes. (If your desk has a strict definition, adopt it consistently.)

---

## Conventions & Notation

### Price Units

Prices are in points per 100 of face value (e.g., 99.25 = 99.25 per 100 notional).

### Clean vs Dirty

| Term | Definition |
|------|------------|
| Clean (quoted) price | Excludes accrued interest |
| Dirty (full/invoice) price | Includes accrued interest: $P_{\text{dirty}} = P_{\text{clean}} + AI$ |

### Accrued Interest (Generic Semiannual Coupon Bond)

$$AI = \frac{c}{2} \cdot \frac{t - t_0}{t_1 - t_0}$$

where $t_0$ is the last coupon date and $t_1$ is the next coupon date.

### Coupons

- Annual coupon rate $c$ (decimal), paid semiannually as $C = \frac{c}{2} \cdot 100$

### Day Count

- Worked examples use 30/360 for coupon accrual and repo interest (explicitly stated when used)
- Repo interest uses a 360-day basis in the P&L identity used here
- **Market note:** Many government bonds use Actual/Actual, while many corporates use 30/360 (convention-dependent)

### Discounting/Compounding (Curve-Based Examples)

Semiannual compounding with discount factor:

$$df(t) = \left(1 + \frac{r(t)}{2}\right)^{-2t}$$

where $t$ is in years and $2t$ is an integer for half-year cashflows (0.5y, 1.0y, ...).

### Curve + Spread Setup

| Symbol | Definition |
|--------|------------|
| $z(t)$ | Benchmark zero curve |
| $s$ | Spread measure (Z-spread / zero-volatility spread), additive to benchmark zero rates |
| $0$ | Start date |
| $h$ | Horizon date (often expressed in days $d$ or years) |

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $100$ | Face value / par notional for pricing examples |
| $c$ | Annual coupon rate (decimal, e.g., 0.05 for 5%) |
| $C_n$ | Coupon or other payment received during the horizon (cash amount per 100) |
| $P_{\text{clean}}(t)$ | Clean price at time $t$ (per 100 face) |
| $P_{\text{dirty}}(t)$ | Dirty/full price at time $t$ (per 100 face), $= P_{\text{clean}}(t) + AI(t)$ |
| $AI(t)$ | Accrued interest at time $t$ |
| $h$ | Horizon length in years |
| $d$ | Horizon length in days (as used in the repo carry formula) |
| $r$ | Repo rate (decimal) |
| $z(t)$ | Benchmark zero rate for maturity $t$ |
| $s$ | Spread measure (here: Z-spread / ZVS) |
| $df(t)$ | Discount factor to maturity $t$ |
| $\text{DV01}$ | Dollar value of a 1bp change in yield, $-\frac{1}{10000}\frac{\partial P}{\partial y}$ |
| $D$ | (Modified) duration |
| $Cvx$ | Convexity |
| $\text{DVOAS}$ | Dollar value of a 1bp change in OAS (spread DV01 for OAS) |

---

## Setup

### What This Chapter Is For

A practitioner's job is often: "Explain the bond's P&L over a horizon." This chapter gives a disciplined way to attribute realized horizon P&L into:

1. **Carry** (coupon accrual and coupon cashflows, optionally net of funding)
2. **Rolldown** (price change from aging on an unchanged curve/spread surface)
3. **Curve moves** (benchmark curve shocks)
4. **Spread changes** (credit/liquidity spread shocks)
5. **Residual terms** (convexity + curve–spread interaction + approximation error)

### Conventions Used in This Chapter

**Pricing convention:**
- Prices are quoted as clean unless explicitly labeled dirty
- Dirty price is used when computing cash-on-cash horizon return

**Coupon and accrued interest:**
- Semiannual coupons $C = \frac{c}{2} \cdot 100$
- Accrued interest uses the proportional formula $AI = \frac{c}{2} \cdot \frac{t - t_0}{t_1 - t_0}$ (with the relevant day-count embedded in the time fractions)

**Funding (repo):**
When repo funding is used, the chapter's "carry" follows the identity:

$$\text{P\&L} = \underbrace{[P(d) - P(0)]}_{\text{clean price change}} + \underbrace{\left(\sum C_n + AI(d) - AI(0) - (P(0) + AI(0)) \, r\frac{d}{360}\right)}_{\text{carry}}$$

(notation per the source)

**Benchmark curve and spread:**
Worked curve/spread decompositions use:
- Benchmark: a stylized Treasury (or "risk-free") zero curve $z(t)$
- Spread: a Z-spread $s$ added to benchmark discount rates (a deterministic-cashflow spread definition)

**Reinvestment:**
Unless stated, coupon cashflows are treated as cash received (not reinvested). If reinvestment is assumed, it will be stated explicitly.

---

## Core Concepts

### 1) Clean Price, Dirty Price, and Accrued Interest

**Formal Definition:**

Dirty price (full/invoice):

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + AI}$$

Accrued interest between coupon dates:

$$\boxed{AI = \frac{c}{2} \cdot \frac{t - t_0}{t_1 - t_0}}$$

**Intuition:**

- Clean price isolates the "bond price" from the mechanical accrual of the coupon
- Dirty price is what you actually pay/receive in settlement: it reflects the fact that the seller has earned some coupon interest since the last coupon date

**Trading/Risk Practice:**

- Traders quote and hedge clean prices; P&L explain often uses clean price change + carry (carry includes accrued interest changes)
- Many return calculations must be done on dirty price to avoid errors around coupon dates (see Example A)

---

### 2) Holding-Period P&L and Horizon Return

**Formal Definition:**

For a long bond position held from $0$ to horizon $h$, ignoring funding:

$$\text{P\&L}_{\text{unfunded}} = (P_{\text{dirty}}(h) - P_{\text{dirty}}(0)) + \sum_{n=1}^{N} C_n$$

A standard horizon return (unlevered, cash-on-cash) is:

$$R_{0 \to h} = \frac{\text{P\&L}}{P_{\text{dirty}}(0)}$$

**Intuition:**

You earn money from:
- Price change (mark-to-market)
- Income (coupon cashflows and coupon accrual)

And you may lose money to:
- Funding cost (if financed)

**Trading/Risk Practice:**

A widely used identity for repo-financed positions writes:

$$\boxed{\text{P\&L} = \underbrace{[P(h) - P(0)]}_{\text{clean price change}} + \underbrace{\left(\sum C_n + AI(h) - AI(0) - (P(0) + AI(0)) \, r\frac{d}{360}\right)}_{\text{carry}}}$$

This form is extremely useful for P&L explain, because it separates the mechanical carry from the mark-to-market.

---

### 3) Carry

**Formal Definition:**

In the repo-financed P&L identity above, carry is the bracketed term:

$$\boxed{\text{Carry} = \sum C_n + AI(h) - AI(0) - (P(0) + AI(0)) \, r\frac{d}{360}}$$

with the source explicitly noting repo interest uses a 30/360 (360-day) basis in that setup.

**Intuition:**

Carry is the "mechanical" part of P&L:
- You accrue coupon interest (and sometimes receive the coupon)
- You pay financing if you borrow cash to hold the bond

Carry is not a forecast of realized P&L; it is a component.

**Trading/Risk Practice:**

Desk meanings vary:
- **Income-only carry:** $\sum C_n + AI(h) - AI(0)$
- **Net carry:** income minus funding cost (as in the repo P&L identity)
- **Hedged carry:** net carry after the carry of hedges (e.g., paying fixed on a swap, futures roll, etc.). (Conceptually standard; exact hedge-carry conventions are desk-specific.)

---

### 4) Rolldown and Pull-to-Par

**Formal Definition (operational, for this chapter):**

Define the counterfactual horizon clean price under unchanged curve and unchanged spread:

$$P_h^{\text{uc}} := P_{\text{clean}}(h; \, z_0(\cdot), \, s_0)$$

where cashflow times are shortened by the passage of time, but the term structures $z_0(\cdot)$ and $s_0$ are held fixed.

Then define rolldown as:

$$\boxed{\text{Rolldown} := P_h^{\text{uc}} - P_0^{\text{clean}}}$$

where $P_0^{\text{clean}} = P_{\text{clean}}(0; \, z_0(\cdot), \, s_0)$.

**Intuition:**

- Even if "nothing happens" to the curve, the bond is one day closer to maturity tomorrow than today
- If the curve is upward sloping, the bond typically "ages" into a lower-yield point on the curve and price can rise (the "roll" benefit)
- Separately, if the bond is at a premium or discount, it tends to move toward par as maturity approaches ("pull to par"). The sources explicitly discuss this pull-to-par mechanism: as time passes and cashflows are received, a bond priced away from par tends to move toward par at maturity

**Trading/Risk Practice:**

- "Carry + rolldown" is often used as a desk's expected return under unchanged curve/spread assumption
- Importantly, realized P&L can be dominated by curve and spread shocks, so carry+rolldown is not guaranteed

---

### 5) Benchmark Curve Move Effect

**Formal Definition:**

Let $z_0(\cdot)$ be the benchmark curve at time 0 and $z_1(\cdot)$ at time $h$. Holding spread fixed at $s_0$, define:

$$\boxed{\text{Curve effect} := P_{\text{clean}}(h; \, z_1(\cdot), \, s_0) - P_{\text{clean}}(h; \, z_0(\cdot), \, s_0)}$$

**Intuition:**

- If benchmark yields rise, discount factors fall, so the PV (price) falls (for long positions)
- For small shocks, the effect is approximately linear in the yield change; for larger shocks, convexity matters

**Trading/Risk Practice:**

Approximations:
- **DV01:** $\text{DV01} = -\frac{1}{10000}\frac{\partial P}{\partial y}$
- **Duration/convexity:** $\frac{\Delta P}{P} \approx -D \, \Delta y + \frac{1}{2} Cvx (\Delta y)^2$

---

### 6) Spread Change Effect

**Formal Definition:**

Pick a spread definition $s$ (Z-spread, OAS, asset swap spread, etc.). Then, holding benchmark curve fixed at $z_1(\cdot)$:

$$\boxed{\text{Spread effect} := P_{\text{clean}}(h; \, z_1(\cdot), \, s_1) - P_{\text{clean}}(h; \, z_1(\cdot), \, s_0)}$$

**Intuition:**

- Wider spreads increase discounting of risky cashflows → lower price
- Narrower spreads do the opposite

**How It Appears in Practice (and Why Definitions Matter):**

- **Z-spread/ZVS:** spread added to benchmark discount rates to match price; sources note discrete vs continuous compounding definitions
- **OAS:** model-dependent spread in an option-pricing lattice; DVOAS measures price sensitivity to OAS in bp
- **Asset swap spread (ASW):** spread over the swap curve that makes the fixed bond cashflows equivalent to floating (LIBOR + spread) plus principal; equivalently, the bond's yield over the swap curve after adjusting for the swap fixed rate

**Key Point:**

The decomposition depends on:
- Which curve is treated as "benchmark" (Treasury vs swaps vs OIS discount curve)
- Which spread definition is used (Z, OAS, ASW, G-spread, etc.)
- Compounding/day-count conventions

---

### 7) Residual, Convexity, and Curve–Spread Interaction

**Formal Definition:**

When you compute curve and spread effects by exact repricing, the clean-price decomposition can be made exact by construction (telescoping differences).

When you compute curve and spread effects by first-order approximations (DV01 × bp), the remainder is the residual:

$$\text{Residual} = \Delta P_{\text{exact}} - \Delta P_{\text{DV01}}$$

(and similarly for separate curve and spread approximations)

**Intuition:**

- The price–yield relationship is curved (convex), so linear approximations miss second-order effects
- If curve and spread move together, cross terms appear in a second-order expansion. A source provides a 2nd-order Taylor expansion in both Treasury yield changes and spread changes including a cross term $dr \, ds$

**Trading/Risk Practice:**

Residual is a diagnostic:
- It should shrink for smaller shocks
- It grows with longer duration, higher convexity, and larger rate/spread moves

---

## Math and Derivations

### 1) From Dirty-Price P&L to "Clean Price Change + Carry"

**Step 1: Start with a basic (unfunded) P&L identity**

$$\text{P\&L}_{\text{unfunded}} = (P_{\text{dirty}}(h) - P_{\text{dirty}}(0)) + \sum_{n=1}^{N} C_n$$

**Step 2: Substitute $P_{\text{dirty}} = P_{\text{clean}} + AI$**

$$\text{P\&L}_{\text{unfunded}} = (P_{\text{clean}}(h) + AI(h) - P_{\text{clean}}(0) - AI(0)) + \sum_{n=1}^{N} C_n$$

$$= \underbrace{(P_{\text{clean}}(h) - P_{\text{clean}}(0))}_{\text{clean price change}} + \underbrace{\left(\sum_{n=1}^{N} C_n + AI(h) - AI(0)\right)}_{\text{interest income}}$$

**Step 3: Add repo financing**

If the initial purchase is financed at repo rate $r$ for $d$ days, a standard approximation for financing cost is:

$$\text{Financing cost} = (P_{\text{clean}}(0) + AI(0)) \, r\frac{d}{360}$$

and the repo-based decomposition becomes:

$$\boxed{\text{P\&L} = (P_{\text{clean}}(h) - P_{\text{clean}}(0)) + \left(\sum C_n + AI(h) - AI(0) - (P_{\text{clean}}(0) + AI(0)) \, r\frac{d}{360}\right)}$$

which matches the source's equation structure and its definition of carry (interest income plus financing cost term).

**Unit Checks:**
- $P$ is in price points per 100
- $r\frac{d}{360}$ is dimensionless (rate × year fraction)
- Financing cost is in price points per 100, consistent with coupons and price changes ✓

---

### 2) Exact (Telescoping) Decomposition of Clean Price Change into Rolldown, Curve, and Spread Effects

Let:
- $z_0(\cdot)$, $s_0$: benchmark curve and spread at time 0
- $z_1(\cdot)$, $s_1$: benchmark curve and spread at horizon $h$
- $P_0 := P_{\text{clean}}(0; \, z_0, s_0)$

Define three horizon-date counterfactual clean prices:

$$\begin{aligned}
P_h^{\text{uc}} &:= P_{\text{clean}}(h; \, z_0, s_0) \quad \text{(unchanged curve/spread)} \\
P_h^{\text{curve}} &:= P_{\text{clean}}(h; \, z_1, s_0) \quad \text{(curve moved only)} \\
P_h^{\text{actual}} &:= P_{\text{clean}}(h; \, z_1, s_1) \quad \text{(actual)}
\end{aligned}$$

Define:

$$\begin{aligned}
\text{Rolldown} &:= P_h^{\text{uc}} - P_0 \\
\text{Curve effect} &:= P_h^{\text{curve}} - P_h^{\text{uc}} \\
\text{Spread effect} &:= P_h^{\text{actual}} - P_h^{\text{curve}}
\end{aligned}$$

Then by construction:

$$\boxed{\Delta P_{\text{clean}} = P_h^{\text{actual}} - P_0 = \text{Rolldown} + \text{Curve effect} + \text{Spread effect}}$$

This is an identity (no approximation) once the intermediate prices are defined.

---

### 3) First-Order and Second-Order Approximations (DV01/Duration/Convexity)

**DV01 Definition (yield-based):**

A source defines:

$$\boxed{\text{DV01} = -\frac{1}{10000}\frac{\partial P}{\partial y}}$$

where $y$ is yield (in decimals, e.g., 0.05).

If $\Delta y$ is expressed in bp, $\Delta y_{\text{bp}} = 10000 \Delta y$, then:

$$\Delta P \approx \frac{\partial P}{\partial y} \Delta y = -10000 \, \text{DV01} \cdot \frac{\Delta y_{\text{bp}}}{10000} = -\text{DV01} \cdot \Delta y_{\text{bp}}$$

**Duration/Convexity Approximation:**

A source gives the 2nd-order approximation:

$$\boxed{\frac{\Delta P}{P} \approx -D \, \Delta y + \frac{1}{2} Cvx (\Delta y)^2}$$

(with $D$ duration and $Cvx$ convexity in that setup)

**Spread DV01 / DVOAS:**

For a spread measure like OAS, a source defines DVOAS as the dollar value of a 0.01 change in OAS (i.e., price change per 1bp change in spread).

Operationally, this plays the role of a spread DV01:

$$\Delta P_{\text{spread}} \approx -\text{DVOAS} \cdot \Delta s_{\text{bp}}$$

**Curve–Spread Cross Term (when both move):**

A source presents a 2nd-order expansion for corporate bond price changes in terms of changes in Treasury yields $dr$ and spread changes $ds$, including a cross term $dr \, ds$.

This provides the rationale for why "curve effect + spread effect" linear approximations can miss interaction terms when both move.

---

## Measurement & Risk (Chapter 7 Scope)

### 1) Carry Over a Horizon

**Unfunded carry (income-only):**

$$\text{Carry}_{\text{unfunded}} = \sum_{n=1}^{N} C_n + AI(h) - AI(0)$$

**Repo-funded carry (net carry):**

Using the repo P&L decomposition:

$$\boxed{\text{Carry}_{\text{repo}} = \sum_{n=1}^{N} C_n + AI(h) - AI(0) - (P_{\text{clean}}(0) + AI(0)) \, r\frac{d}{360}}$$

with repo interest calculated on a 360-day basis in that formulation.

**Hedged carry (conceptual):**

If a hedge is held (swap, futures, etc.), hedged carry = bond carry + hedge carry. Hedge-carry conventions are instrument- and desk-specific; define them explicitly if used.

---

### 2) Rolldown (Aging Down the Curve)

**Definition (this chapter):**

$$\text{Rolldown} = P_{\text{clean}}(h; \, z_0, s_0) - P_{\text{clean}}(0; \, z_0, s_0)$$

**How to compute:**
1. Keep the initial curve and spread surfaces fixed
2. Move the valuation date forward to the horizon and shorten the time-to-cashflows
3. Reprice the bond (clean) at the horizon date using the unchanged curve/spread

---

### 3) Curve Shift Effect

**Definition:**

$$\text{Curve effect} = P_{\text{clean}}(h; \, z_1, s_0) - P_{\text{clean}}(h; \, z_0, s_0)$$

**Approximation:**

For a small parallel shift $\Delta z_{\text{bp}}$:

$$\Delta P_{\text{curve}} \approx -\text{DV01}_{\text{curve}} \cdot \Delta z_{\text{bp}}$$

Convexity matters when $|\Delta z|$ or duration is large; 2nd-order terms improve accuracy.

---

### 4) Spread Change Effect

**Definition:**

$$\text{Spread effect} = P_{\text{clean}}(h; \, z_1, s_1) - P_{\text{clean}}(h; \, z_1, s_0)$$

**Approximation:**

$$\Delta P_{\text{spread}} \approx -\text{DVOAS} \cdot \Delta s_{\text{bp}}$$

(where "DVOAS" can be interpreted as a spread DV01 for the chosen spread measure)

---

### 5) Why "Carry + Rolldown" Is Not Guaranteed P&L

- Carry + rolldown is a conditional expectation under "unchanged curve/spread" assumptions
- Realized P&L includes unexpected curve moves and unexpected spread moves, which can dominate
- A source illustrates that realized returns depend on realized rate paths (forward rates realized vs not), emphasizing that returns are contingent on how rates evolve, not just current yields

---

## Worked Examples

### Example A — Horizon Return Basics (Dirty vs Clean; Coupon Date vs None)

**Bond:**
- Face: 100
- Coupon: 6.00% annual, semiannual coupons $C = 3.00$
- Coupon dates: Apr 30 and Oct 30
- Day count for accrual: 30/360
- Clean/dirty relationship: $P_{\text{dirty}} = P_{\text{clean}} + AI$

**Start (Jan 30, 2026):**
- Given clean price $P_{\text{clean}}(0) = 101.80$
- Last coupon date: Oct 30, 2025; next coupon: Apr 30, 2026
- Days since last coupon: 90, days in period: 180
- Accrued interest:

$$AI(0) = 3.00 \cdot \frac{90}{180} = 1.50$$

- Dirty price:

$$P_{\text{dirty}}(0) = 101.80 + 1.50 = 103.30$$

#### Case A1: Horizon with No Coupon Date (Mar 30, 2026)

- Given horizon clean price $P_{\text{clean}}(h) = 101.50$
- Accrued interest at horizon: Oct 30 → Mar 30 = 150 days (30/360)

$$AI(h) = 3.00 \cdot \frac{150}{180} = 2.50$$

- Dirty price at horizon:

$$P_{\text{dirty}}(h) = 101.50 + 2.50 = 104.00$$

- Coupons received: none

**Total P&L:**

$$\text{P\&L} = P_{\text{dirty}}(h) - P_{\text{dirty}}(0) = 104.00 - 103.30 = 0.70$$

**Holding-period return (on dirty price):**

$$R = \frac{0.70}{103.30} = 0.00678 \approx 0.678\%$$

**Decompose into price vs income:**
- Clean price change: $\Delta P_{\text{clean}} = 101.50 - 101.80 = -0.30$
- Interest income (accrual): $AI(h) - AI(0) = 2.50 - 1.50 = +1.00$
- Check: $-0.30 + 1.00 = 0.70$ ✓

#### Case A2: Horizon Crossing One Coupon Date (May 30, 2026; Coupon on Apr 30)

- Given horizon clean price $P_{\text{clean}}(h) = 101.20$
- Coupon received on Apr 30: $C = 3.00$
- Accrued interest at May 30 (Apr 30 → May 30 = 30 days):

$$AI(h) = 3.00 \cdot \frac{30}{180} = 0.50$$

- Dirty price at horizon:

$$P_{\text{dirty}}(h) = 101.20 + 0.50 = 101.70$$

**Total P&L:**

$$\text{P\&L} = P_{\text{dirty}}(h) + C - P_{\text{dirty}}(0) = 101.70 + 3.00 - 103.30 = 1.40$$

**Holding-period return:**

$$R = \frac{1.40}{103.30} = 0.0136 \approx 1.36\%$$

**Decompose:**
- Clean price change: $101.20 - 101.80 = -0.60$
- Income: $C + AI(h) - AI(0) = 3.00 + 0.50 - 1.50 = 2.00$
- Check: $-0.60 + 2.00 = 1.40$ ✓

---

### Example B — Carry-Only Scenario (Unfunded vs Repo-Funded Carry)

Use the same bond and start point as Example A:
- $P_{\text{clean}}(0) = 101.80$, $AI(0) = 1.50$, so $P_{\text{dirty}}(0) = 103.30$

**Repo carry identity (with 360-day basis for repo interest):**
- Assume repo rate $r = 4.50\% = 0.045$

#### Horizon B1: Jan 30 → Mar 30 (60 days; no coupon)

From Example A1: $AI(h) = 2.50$

**Unfunded carry (income-only):**

$$\text{Carry}_{\text{unfunded}} = AI(h) - AI(0) = 2.50 - 1.50 = 1.00$$

**Repo financing cost:**

$$\text{Fin} = (P_{\text{clean}}(0) + AI(0)) \, r\frac{d}{360} = 103.30 \cdot 0.045 \cdot \frac{60}{360}$$

Since $\frac{60}{360} = \frac{1}{6}$:

$$\text{Fin} = 103.30 \cdot 0.045 \cdot \frac{1}{6} = 103.30 \cdot 0.0075 = 0.77475$$

**Repo-funded carry:**

$$\text{Carry}_{\text{repo}} = 1.00 - 0.77475 = 0.22525$$

**Carry returns (on initial dirty price):**
- Unfunded: $1.00 / 103.30 = 0.968\%$
- Repo-funded: $0.22525 / 103.30 = 0.218\%$

#### Horizon B2: Jan 30 → May 30 (120 days; one coupon)

From Example A2: coupon $C = 3.00$, $AI(h) = 0.50$

**Income:**

$$\sum C_n + AI(h) - AI(0) = 3.00 + 0.50 - 1.50 = 2.00$$

**Financing:**

$$103.30 \cdot 0.045 \cdot \frac{120}{360} = 103.30 \cdot 0.045 \cdot \frac{1}{3} = 103.30 \cdot 0.015 = 1.5495$$

**Repo-funded carry:**

$$2.00 - 1.5495 = 0.4505$$

**Carry returns:**
- Unfunded: $2.00 / 103.30 = 1.936\%$
- Repo-funded: $0.4505 / 103.30 = 0.436\%$

---

### Example C — Rolldown-Only Scenario (Curve Held Fixed; Aging Reduces Time-to-Cashflows)

Here we compute rolldown = clean price change under unchanged curve/spread, excluding carry.

**Bond (used for Examples C–F):**
- Face: 100
- Coupon: 5.00% annual, semiannual coupons $C = 2.50$
- Start date $t_0$: Jul 30, 2026 (coupon date → $AI(0) = 0$)
- Maturity: Jul 30, 2028 (2.0 years from $t_0$)
- Cashflows (years from $t_0$): $t = \{0.5, 1.0, 1.5, 2.0\}$ with coupons 2.5 each period and final 102.5

**Benchmark zero curve $z(t)$ (semiannual comp; stylized):**

| Maturity | $z(t)$ |
|----------|--------|
| 0.5 | 4.00% |
| 1.0 | 4.20% |
| 1.5 | 4.35% |
| 2.0 | 4.50% |

**Spread measure:** Z-spread $s_0 = 1.00\%$ (100bp), additive to benchmark rates

**Total discount rates:** $r(t) = z(t) + s_0$

| Maturity | $r(t)$ |
|----------|--------|
| 0.5 | 5.00% |
| 1.0 | 5.20% |
| 1.5 | 5.35% |
| 2.0 | 5.50% |

**Discount factors $df(t) = (1 + r(t)/2)^{-2t}$:**

| Maturity | $df(t)$ |
|----------|---------|
| 0.5 | 0.9756097561 |
| 1.0 | 0.9499599117 |
| 1.5 | 0.9238593648 |
| 2.0 | 0.8971657337 |

**Start clean price $P_0^{\text{clean}}$ (exact repricing):**

$$P_0^{\text{clean}} = 2.5 \, df(0.5) + 2.5 \, df(1.0) + 2.5 \, df(1.5) + 102.5 \, df(2.0)$$
$$= 2.43902439 + 2.37489978 + 2.30964841 + 91.95948770$$
$$= 99.08306029$$

**Horizon:** $h = 0.5$ years (Jan 30, 2027), coupon date ⇒ clean pricing, $AI(h) = 0$.

Under unchanged curve/spread, remaining cashflows from horizon are at 0.5, 1.0, 1.5 years:
- 2.5 at 0.5, 2.5 at 1.0, 102.5 at 1.5

**Horizon clean price under unchanged curve/spread:**

$$P_h^{\text{uc}} = 2.5 \, df(0.5) + 2.5 \, df(1.0) + 102.5 \, df(1.5)$$
$$= 2.43902439 + 2.37489978 + 94.69558489$$
$$= 99.50950906$$

**Rolldown (clean price):**

$$\boxed{\text{Rolldown} = P_h^{\text{uc}} - P_0^{\text{clean}} = 99.50950906 - 99.08306029 = 0.42644877}$$

**Rolldown return on initial price:**

$$0.42644877 / 99.08306029 = 0.4304\%$$

---

### Example D — Curve Move Decomposition (Parallel +10bp Benchmark Zero Shock)

Use the start-date bond and curve from Example C.

**Shock:**
- Benchmark curve shifts up +10bp at all maturities: $z'(t) = z(t) + 0.10\%$
- Spread stays $s_0 = 1.00\%$
- Total discount rates become: $r'(0.5) = 5.10\%$, $r'(1.0) = 5.30\%$, $r'(1.5) = 5.45\%$, $r'(2.0) = 5.60\%$

**Exact repricing:**

Shocked discount factors $df'(t) = (1 + r'(t)/2)^{-2t}$:

| Maturity | $df'(t)$ |
|----------|----------|
| 0.5 | 0.9751340809 |
| 1.0 | 0.9490347012 |
| 1.5 | 0.9225109932 |
| 2.0 | 0.8954215481 |

Shocked price:

$$P' = 2.5 \, df'(0.5) + 2.5 \, df'(1.0) + 2.5 \, df'(1.5) + 102.5 \, df'(2.0) = 98.89740812$$

**Exact curve-driven price change:**

$$\Delta P_{\text{exact}} = P' - P_0 = 98.89740812 - 99.08306029 = -0.18565217$$

**DV01-based approximation (first order):**

DV01 concept (yield DV01) is defined as $-\frac{1}{10000}\frac{\partial P}{\partial y}$.

Here we compute a parallel curve DV01 by bumping the benchmark curve by ±1bp and using a central difference:

- Price with +1bp parallel shift: $P_{+1\text{bp}} = 99.06447499$
- Price with −1bp parallel shift: $P_{-1\text{bp}} = 99.10165005$

Central-difference DV01 (per 1bp):

$$\text{DV01} \approx \frac{P_{-1\text{bp}} - P_{+1\text{bp}}}{2} = \frac{99.10165005 - 99.06447499}{2} = 0.01858753$$

**First-order price change for +10bp:**

$$\Delta P_{\text{DV01}} \approx -\text{DV01} \cdot 10 = -0.18587527$$

**Convexity error (DV01 approx − exact):**

$$-0.18587527 - (-0.18565217) = -0.00022310$$

DV01 (linear) slightly overstates the price drop because convexity is positive.

**Convexity-adjusted (second order; optional check):**

Using the second difference from the ±1bp prices, the convexity correction for $\Delta y = 10\text{bp} = 0.0010$ is $\approx +0.00022332$, so the 2nd-order approximation matches the exact repricing extremely closely in this small-shock example.

---

### Example E — Spread Change Decomposition (Z-Spread +25bp; Benchmark Curve Fixed)

Use Example C at start date $t_0$.

**Spread definition:**
We use a Z-spread/ZVS-style definition: spread $s$ added to benchmark discount rates, with compounding convention specified.

**Shock:**
- Benchmark curve $z(t)$ unchanged
- Spread widens: $s_1 = s_0 + 25\text{bp} = 1.25\%$
- New total rates: $r(0.5) = 5.25\%$, $r(1.0) = 5.45\%$, $r(1.5) = 5.60\%$, $r(2.0) = 5.75\%$

**Exact repricing:**

Discount factors:

| Maturity | $df(t)$ |
|----------|---------|
| 0.5 | 0.9744214373 |
| 1.0 | 0.9476494178 |
| 1.5 | 0.9204933514 |
| 2.0 | 0.8928132094 |

Price with widened spread:

$$P_{\text{wide}} = 98.61976448$$

**Exact spread-driven price change:**

$$\Delta P_{\text{exact}} = 98.61976448 - 99.08306029 = -0.46329581$$

**Spread DV01 / DVOAS-style approximation:**

A source defines DVOAS as the dollar value of a 1bp change in OAS.

We compute an analogous spread DV01 by bumping $s$ by ±1bp:
- $P(s_0 + 1\text{bp}) = 99.06447499$
- $P(s_0 - 1\text{bp}) = 99.10165005$

Spread DV01:

$$\text{SDV01} \approx \frac{99.10165005 - 99.06447499}{2} = 0.01858753$$

**First-order approximation for +25bp:**

$$\Delta P_{\text{SDV01}} \approx -0.01858753 \cdot 25 = -0.46468817$$

**Error (linear − exact):**

$$-0.46468817 - (-0.46329581) = -0.00139236$$

Convexity is more visible for 25bp than for 10bp.

**Convexity-adjusted check (optional):**

Using the same second-difference logic, the convexity correction for $\Delta s = 25\text{bp} = 0.0025$ is about $+0.00139573$, bringing the approximation nearly on top of the exact repricing.

---

### Example F — Full Decomposition Over a Horizon (Carry + Rolldown + Curve + Spread + Residual)

Use the bond and initial curve/spread from Example C.

**Horizon:**
- $t_0 =$ Jul 30, 2026 (coupon date ⇒ $AI(0) = 0$)
- $t_1 =$ Jan 30, 2027 (coupon date ⇒ $AI(1) = 0$)
- Horizon length: 180 days on 30/360 basis (0.5 year)

**Start price:**
- $P_0^{\text{clean}} = 99.08306029$
- $P_0^{\text{dirty}} = 99.08306029$ (since $AI(0) = 0$)

**Assumed realized market changes over horizon:**
- Benchmark curve shifts up +8bp parallel
- Spread widens +15bp (from 1.00% to 1.15%)

#### Step 1: Carry (income and financing)

- Coupon received at $t_1$: $C = 2.50$

**Unfunded carry (income-only):**

$$\text{Carry}_{\text{unfunded}} = 2.50 + AI(1) - AI(0) = 2.50$$

**Repo-funded carry (if financed at repo $r = 4.50\%$, 30/360):**

$$\text{Fin} = P_0^{\text{dirty}} \cdot r\frac{d}{360} = 99.08306029 \cdot 0.045 \cdot \frac{180}{360} = 99.08306029 \cdot 0.0225 = 2.22936886$$

$$\text{Carry}_{\text{repo}} = 2.50 - 2.22936886 = 0.27063114$$

(This matches the repo carry structure in the cited P&L decomposition.)

#### Step 2: Rolldown (unchanged curve/spread)

From Example C:

$$P_1^{\text{uc}} = 99.50950906$$

$$\text{Rolldown} = P_1^{\text{uc}} - P_0 = 0.42644877$$

#### Step 3: Curve effect (curve moved, spread held at $s_0$)

Compute horizon price with curve shifted +8bp and spread still 1.00%:

$$P_1^{\text{curve}} = 99.39611893$$

$$\text{Curve effect} = P_1^{\text{curve}} - P_1^{\text{uc}} = 99.39611893 - 99.50950906 = -0.11339013$$

#### Step 4: Spread effect (spread moved, curve held at new level)

Now widen spread by +15bp (to 1.15%) keeping the +8bp curve shift:

$$P_1^{\text{actual}} = 99.18398395$$

$$\text{Spread effect} = P_1^{\text{actual}} - P_1^{\text{curve}} = 99.18398395 - 99.39611893 = -0.21213498$$

#### Step 5: Check the clean-price decomposition (exact)

Sum of clean-price components:

$$0.42644877 - 0.11339013 - 0.21213498 = 0.10092367$$

Actual clean price change:

$$P_1^{\text{actual}} - P_0 = 99.18398395 - 99.08306029 = 0.10092367$$

**Matches exactly (telescoping identity).** ✓

#### Step 6: Total P&L and return

**Unfunded P&L:**

$$\text{P\&L}_{\text{unfunded}} = (P_1^{\text{actual}} - P_0) + 2.50 = 0.10092367 + 2.50 = 2.60092367$$

Return on dirty price:

$$R_{\text{unfunded}} = \frac{2.60092367}{99.08306029} = 2.6250\% \text{ over 6 months}$$

**Repo-funded P&L (on full price; ignoring haircut/leverage):**

$$\text{P\&L}_{\text{repo}} = (P_1^{\text{actual}} - P_0) + 0.27063114 = 0.37155481$$

Return on full price:

$$R_{\text{repo}} = \frac{0.37155481}{99.08306029} = 0.3750\% \text{ over 6 months}$$

(Return on equity depends on haircut/leverage; not specified here.)

#### Step 7: First-order approximation vs exact (residual)

Compute horizon DV01 (for the aged bond) by ±1bp bump around the unchanged horizon curve/spread:

- $P_{1,-1\text{bp}} = 99.52369515$
- $P_{1,+1\text{bp}} = 99.49532571$

$$\text{DV01}_h \approx \frac{99.52369515 - 99.49532571}{2} = 0.01418472$$

**Approximate effects:**

Curve effect (+8bp):

$$\Delta P_{\text{curve}}^{(1)} \approx -0.01418472 \cdot 8 = -0.11347776$$

Spread effect (+15bp):

$$\Delta P_{\text{spread}}^{(1)} \approx -0.01418472 \cdot 15 = -0.21277079$$

**Approximate total P&L (unfunded):**

$$\widehat{\text{P\&L}} = 2.50 + 0.42644877 - 0.11347776 - 0.21277079 = 2.60020022$$

**Residual:**

$$\text{Residual} = \text{P\&L}_{\text{exact}} - \widehat{\text{P\&L}} = 2.60092367 - 2.60020022 = 0.00072345$$

**Interpretation:**
This is the convexity/cross-term remainder from linearizing curve and spread effects, consistent with 2nd-order expansions that include squared and cross terms in rate/spread changes.

---

## Practical Notes

### Common Desk Definitions and Ambiguity Traps

#### 1) Different Meanings of "Carry"

**"Carry = income" vs "carry = income − funding":**
- The repo-funded identity explicitly defines carry to include the financing term (negative for long positions financed in repo)
- "Hedged carry" depends on what hedge is used and how its own carry/roll is measured

#### 2) Spread Measure Ambiguity

**Yield spread (bond yield − benchmark yield) vs Z-spread/ZVS vs OAS vs ASW:**

| Spread | Definition |
|--------|------------|
| **Z-spread/ZVS** | Discount-rate spread needed to match price; compounding conventions can differ (discrete vs continuous) |
| **OAS** | Model-dependent spread in an option-pricing lattice; DVOAS measures price sensitivity to OAS in bp |
| **ASW** | Spread over the swap curve implied by matching the bond to floating + spread; definition depends on swap curve usage and is not the same as Treasury spread |

**Implication for decomposition:**
If you change the spread definition, "spread effect" changes, and parts of what you used to call "curve effect" may migrate into "spread effect," and vice versa.

#### 3) Clean vs Dirty Price Mistakes

Returns computed on clean prices without adjusting for accrued interest and coupons can be badly wrong around coupon dates (Example A).

**Always ensure:**
- Price change is measured on clean price
- Income includes $AI(h) - AI(0)$ plus coupons received
- Or equivalently, work directly with dirty prices plus coupons

### Implementation Pitfalls

| Issue | Notes |
|-------|-------|
| **Schedule generation** | Coupon dates, ex-coupon conventions, and settlement calendars materially affect AI and which cashflows are included |
| **Horizon-date accrued handling** | If horizon date is between coupon dates, you must compute $AI(h)$ and use dirty price consistently |
| **Coupon reinvestment** | State whether coupons are reinvested and at what rate. Otherwise, "total return" is ambiguous |
| **Modern multi-curve context** | In collateralized markets, discounting may use an OIS discount curve, while floating projections use a LIBOR (or other) curve. If your decomposition uses a benchmark curve that is not your discount curve, explicitly state the mapping |

### Verification Tests (Quick Sanity Checks)

| Test | Expected Result |
|------|-----------------|
| **If curve and spread are unchanged** | Total return ≈ carry + rolldown (within rounding) |
| **Sign checks** | Higher yields/spreads → lower prices (long bond loses) |
| **Approximation consistency** | Residual shrinks as shocks become small (linearization becomes accurate) |

---

## Summary & Recall

### 10-Bullet Executive Summary

1. Use dirty prices (or clean + AI consistently) to compute horizon returns; coupon timing matters
2. Dirty price = clean price + accrued interest; accrued interest is proportional to elapsed coupon-period time
3. A practical P&L explain identity: P&L = clean price change + carry (carry = income − funding in a repo setup)
4. Carry captures coupon accrual/cashflows and (optionally) financing cost; it is mechanical, not a forecast
5. Define rolldown operationally as the clean price change from revaluing the aged bond on the unchanged curve/spread surface
6. Decompose clean price change exactly via counterfactual prices: rolldown + curve effect + spread effect
7. Curve moves are approximated by DV01 × bp; convexity improves the approximation when shocks are larger
8. Spread moves use a spread sensitivity such as DVOAS (or Z-spread DV01 analog)
9. Carry + rolldown is not guaranteed P&L; curve/spread shocks can dominate
10. Decomposition depends on benchmark curve choice, spread definition, and day-count/compounding conventions

### Cheat Sheet: Key Definitions + Key Decompositions + "When It Fails"

**Key Definitions:**

- $P_{\text{dirty}} = P_{\text{clean}} + AI$
- Repo P&L identity:

$$\text{P\&L} = [P(h) - P(0)] + \left(\sum C_n + AI(h) - AI(0) - (P(0) + AI(0)) \, r\frac{d}{360}\right)$$

- DV01:

$$\text{DV01} = -\frac{1}{10000}\frac{\partial P}{\partial y}$$

**Exact Clean-Price Decomposition:**

$$\Delta P_{\text{clean}} = \underbrace{(P_h^{\text{uc}} - P_0)}_{\text{rolldown}} + \underbrace{(P_h^{\text{curve}} - P_h^{\text{uc}})}_{\text{curve}} + \underbrace{(P_h^{\text{actual}} - P_h^{\text{curve}})}_{\text{spread}}$$

**First-Order Approximations:**
- Curve: $\Delta P \approx -\text{DV01} \cdot \Delta z_{\text{bp}}$
- Spread: $\Delta P \approx -\text{DVOAS} \cdot \Delta s_{\text{bp}}$

**When It Fails / Residual Grows:**
- Large shocks (duration/convexity nonlinearity): need convexity term
- Joint curve + spread moves: cross terms $dr \, ds$ matter in 2nd-order expansions
- Changing spread definitions or benchmark curve: attribution changes materially

---

## Flashcards (25 Q/A Pairs)

**Q1:** What is the relationship between dirty and clean price?
**A1:** $P_{\text{dirty}} = P_{\text{clean}} + AI$

**Q2:** What does accrued interest represent?
**A2:** Coupon interest earned since the last coupon date (pro-rata through the period).

**Q3:** In a repo-funded position, what is "carry" (in the cited decomposition)?
**A3:** $\sum C_n + AI(h) - AI(0) - (P(0) + AI(0))r(d/360)$

**Q4:** Why separate clean price change from carry?
**A4:** Clean price change captures MTM; carry captures coupon accrual and funding—useful for P&L explain.

**Q5:** Define horizon return (unfunded).
**A5:** $R = \text{P\&L} / P_{\text{dirty}}(0)$ where $\text{P\&L} = (P_{\text{dirty}}(h) - P_{\text{dirty}}(0)) + \sum C_n$

**Q6:** What is rolldown (as defined in this chapter)?
**A6:** $P_{\text{clean}}(h; z_0, s_0) - P_{\text{clean}}(0; z_0, s_0)$ (aging with curve/spread fixed)

**Q7:** What is pull-to-par?
**A7:** The tendency of a bond price to approach par at maturity as time passes and cashflows are received.

**Q8:** What is DV01?
**A8:** $\text{DV01} = -\frac{1}{10000}\frac{\partial P}{\partial y}$

**Q9:** How do you use DV01 to approximate a price change for $\Delta y_{\text{bp}}$?
**A9:** $\Delta P \approx -\text{DV01} \cdot \Delta y_{\text{bp}}$

**Q10:** What does convexity correct in DV01-based approximations?
**A10:** Nonlinearity of the price–yield relationship; linear DV01 misses 2nd-order effects.

**Q11:** State the duration/convexity approximation for $\Delta P/P$.
**A11:** $\Delta P/P \approx -D\Delta y + \frac{1}{2}Cvx(\Delta y)^2$

**Q12:** What is OAS?
**A12:** Constant spread (in a tree-based model) that makes model price equal market price.

**Q13:** What is DVOAS?
**A13:** Dollar value of a 1bp change in OAS.

**Q14:** What is Z-spread/ZVS?
**A14:** Spread added to benchmark discount rates so discounted cashflows equal the bond's price; definition depends on compounding convention.

**Q15:** Why does spread attribution depend on the spread measure?
**A15:** Different spread measures embed different curve assumptions and option adjustments, changing what is labeled "spread-driven."

**Q16:** What is the "curve effect" in horizon clean-price explain?
**A16:** Price change at horizon from changing benchmark curve $z_0 \to z_1$ holding spread fixed.

**Q17:** What is the "spread effect" in horizon clean-price explain?
**A17:** Price change at horizon from changing spread $s_0 \to s_1$ holding benchmark curve fixed.

**Q18:** Why can carry+rolldown be positive while realized P&L is negative?
**A18:** Adverse curve and/or spread moves can overwhelm carry and rolldown.

**Q19:** Give a quick sign check for a long bond when yields rise.
**A19:** Price falls ⇒ P&L negative (absent large carry offsets).

**Q20:** What happens to accrued interest on a coupon date (typical convention)?
**A20:** It resets (AI goes back to ~0) after the coupon is paid; price quotes remain clean.

**Q21:** What is an "invoice price"?
**A21:** Another term for dirty price (clean + accrued).

**Q22:** Why does decomposition depend on benchmark curve choice (Treasury vs swap vs OIS)?
**A22:** Because the "curve move" component is defined relative to that benchmark; changing the benchmark changes the measured curve and spread components.

**Q23:** What is an asset swap spread (conceptually)?
**A23:** The spread over the swap curve implied by matching bond cashflows to floating + spread; equivalently yield over swap curve after adjusting for swap fixed rate.

**Q24:** In a multi-curve framework, what curve is commonly used for discounting collateralized trades?
**A24:** An OIS discount curve (with separate projection curves).

**Q25:** What should happen to the residual as shocks become tiny?
**A25:** It should approach zero as linear approximations become accurate.

---

## Mini Problem Set (14 Questions)

**Problem 1:** A bond has clean price 99.20 and accrued interest 1.10. What is dirty price?

**Problem 2:** A semiannual 6% bond has 120 days elapsed in a 180-day coupon period (30/360). Compute accrued interest per 100.

**Problem 3:** A bond is bought at clean 101.80 with AI 1.50 and sold at clean 101.50 with AI 2.50. No coupon is received. Compute P&L and horizon return on dirty price.

**Problem 4:** Using the repo carry formula, compute repo-funded carry over 60 days when dirty price is 103.30, repo rate is 4.5%, and interest income is 1.00.

**Problem 5:** For a bond with DV01 = 0.0186, estimate the price change for a +12bp yield move.

**Problem 6:** A bond's exact price change for +25bp is −0.4633. DV01 approximation gives −0.4647. What does the difference represent?

**Problem 7:** Define an exact telescoping decomposition of clean price change into rolldown, curve effect, and spread effect using counterfactual horizon prices.

**Problem 8:** In Example C, compute the rolldown return (rolldown / initial price) to 4 decimals.

**Problem 9:** Explain two reasons why "carry + rolldown" can be a poor forecast of realized P&L.

**Problem 10:** Give two different market definitions of "spread" used in credit relative value and explain how the decomposition changes across them.

**Problem 11:** Describe how you would compute spread DV01 for a Z-spreaded bond (no options) using a finite-difference bump.

**Problem 12:** If both benchmark curve and spread move, what second-order interaction term can appear in a Taylor approximation?

**Problem 13:** In a multi-curve setting, why can a Treasury-benchmark decomposition be inconsistent with an OIS-discounting valuation?

**Problem 14:** Propose a robust set of verification tests for a P&L explain system implementing carry/rolldown/curve/spread decomposition.

### Brief Solution Sketches (1–7 only)

**1.** $P_{\text{dirty}} = 99.20 + 1.10 = 100.30$

**2.** Semiannual coupon $= 3.00$. AI $= 3.00 \cdot (120/180) = 2.00$

**3.** Dirty start $= 103.30$. Dirty end $= 104.00$. P&L $= 0.70$. Return $= 0.70/103.30 \approx 0.678\%$

**4.** Financing cost $= 103.30 \cdot 0.045 \cdot (60/360) = 0.77475$. Repo-funded carry $= 1.00 - 0.77475 = 0.22525$

**5.** $\Delta P \approx -0.0186 \cdot 12 = -0.2232$ price points

**6.** The difference is primarily convexity (nonlinearity) and any mismatch between the sensitivity point and the actual move; it is the "residual."

**7.** Define $P_0 = P_{\text{clean}}(0; z_0, s_0)$, $P_h^{\text{uc}} = P_{\text{clean}}(h; z_0, s_0)$, $P_h^{\text{curve}} = P_{\text{clean}}(h; z_1, s_0)$, $P_h^{\text{actual}} = P_{\text{clean}}(h; z_1, s_1)$. Then rolldown $= P_h^{\text{uc}} - P_0$, curve effect $= P_h^{\text{curve}} - P_h^{\text{uc}}$, spread effect $= P_h^{\text{actual}} - P_h^{\text{curve}}$, and they sum to $P_h^{\text{actual}} - P_0$.

---

## Source Map

### (A) Verified Facts

| Content | Source |
|---------|--------|
| Dirty = clean + AI; accrued interest proportional formula | Tuckman Ch 1-4 |
| Repo P&L identity and carry definition; 360-day basis | Tuckman Ch 3, 8 |
| DV01 definition: $-\frac{1}{10000}\frac{\partial P}{\partial y}$ | Tuckman Ch 5-6 |
| Duration/convexity 2nd-order approximation | Tuckman Ch 5 |
| OAS definition; DVOAS as dollar value of 1bp OAS change | Tuckman Ch 4, 17-18 |
| Z-spread/ZVS definition; compounding variants noted | Tuckman Ch 4 |
| Pull-to-par mechanism | Tuckman Ch 3 |
| 2nd-order expansion including $dr \, ds$ cross term | Tuckman (corporate bond pricing) |

### (B) Reasoned Inference

| Inference | Derivation Logic |
|-----------|------------------|
| Rolldown = counterfactual reprice at horizon with unchanged curve/spread | Operational definition using source mechanics |
| Exact telescoping decomposition via intermediate prices | Direct algebraic construction |
| Residual = convexity + cross terms | Follows from 2nd-order Taylor expansion |

### (C) Speculation / Uncertainty

| Topic | Notes |
|-------|-------|
| "Rolldown" terminology | Sources discuss mechanics (pull-to-par, aging) but do not impose a single industry-wide definition. Adopted explicit operational definition; desks may differ. |
| Hedged carry conventions | Instrument- and desk-specific; not fully specified in sources |

---

*Cross-references:*
- Clean/dirty price and accrued interest: see Chapter 4, Chapter 5
- DV01 and duration: see Chapter 11, Chapter 12
- Repo mechanics and financing: see Chapter 9
- Spread measures (Z-spread, OAS, ASW): see Chapter 8
