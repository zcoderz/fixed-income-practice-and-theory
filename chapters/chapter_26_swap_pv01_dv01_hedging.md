# Chapter 26: Swap PV01 / DV01 and Hedging with Swaps

---

## Fact Classification

### (A) Verified Facts (Source-Backed)
- Swap PV sign convention: $PV_{\text{rec}} = V_{\text{fixed}} - V_{\text{float}}$, $PV_{\text{pay}} = -PV_{\text{rec}}$ (Tuckman)
- "PV01" in swap markets normally means change in swap value for a 1bp change in the fixed rate $K$ (Tuckman)
- Bond DV01 definition: $DV01 = \frac{1}{10,000}\left(-\frac{dP}{dy}\right)$ (Tuckman)
- At initiation, both legs of a new swap at "LIBOR flat" are worth par (Tuckman)
- Standard float-for-fixed swaps valued by projecting cashflows from forward rates and discounting at risk-free (OIS) rate (Hull)
- Multi-curve framework: separate discount and forward curves as post-crisis standard (Hull, Andersen & Piterbarg)
- Forward rate from discount factors: $L(t;t_i,t_{i+1}) = \frac{1}{\tau_i}\left(\frac{P^{(L)}(t,t_i)}{P^{(L)}(t,t_{i+1})} - 1\right)$ (Andersen & Piterbarg)
- DV01 hedge ratio: $F_B = -F_A \frac{DV01_A}{DV01_B}$ (Tuckman)
- Combined swap DV01 (fixed minus floating) not often used because it mixes long-term fixed-rate risk with short-term floating-rate risk (Tuckman)
- Floating-leg DV01 depends on time to next reset and can be very small relative to fixed leg (Tuckman)

### (B) Reasoned Inference (Derived from A)
- Par swap rate formula $K^* = \frac{\sum F_i \alpha_i P_d(0,T_i)}{A}$ derived from setting $PV_{\text{rec}} = 0$
- Single-curve simplification $K^* = \frac{1 - P(0,T_n)}{A}$ derived from forward-from-DF formula
- PV01-to-$K$ equals $\pm NA \times 0.0001$ derived from differentiating PV w.r.t. $K$
- Discount PV01 vs projection PV01 can differ because they affect different parts of the valuation

### (C) Speculation (Clearly Labeled; Minimal)
- Specific desk curve-bump methodologies (bump zero rates vs par rates vs instrument quotes) vary; sources describe multiple approaches but don't specify which is "correct"
- Exact projection-curve bump rules (bump zeros and recompute forwards vs bump forwards directly) are implementation choices

---

## Conventions & Notation

### Perspective / Sign for Swap PV

Unless stated otherwise, swap PV is from the **fixed-rate receiver's** perspective:

$$PV_{\text{rec}} = V_{\text{fixed}} - V_{\text{float}}, \quad PV_{\text{pay}} = -PV_{\text{rec}} = V_{\text{float}} - V_{\text{fixed}}$$

A positive $PV_{\text{rec}}$ is favorable to the fixed receiver and unfavorable to the fixed payer.

### PV01 Naming Collision (Important)

| Term | Tuckman Swap-Market Usage | Modern Risk System Usage |
|------|---------------------------|--------------------------|
| **PV01** | Change in swap value for 1bp change in fixed rate $K$ (curves fixed) | Often means curve bump sensitivity (DV01-like) |

This chapter distinguishes them explicitly as **PV01-to-$K$** vs **curve PV01**.

### Multi-Curve Language

| Curve | Symbol | Purpose |
|-------|--------|---------|
| **Discount curve** | $P_d(0,T)$ | Discount factors for PV of all cashflows |
| **Projection (index) curve** | $F_i$ | Forward rates for floating leg from the floating reference index |

This matches the "forward rates realized + discount at (OIS) risk-free" valuation approach described in Hull.

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $N$ | Swap notional (currency units) |
| $0 = T_0 < T_1 < \cdots < T_n$ | Payment dates |
| $\alpha_i$ | Accrual year fraction for $[T_{i-1}, T_i]$ |
| $K$ | Fixed swap rate (per annum). Fixed payment at $T_i$: $N K \alpha_i$ |
| $F_i$ | Forward rate for floating reference over $[T_{i-1}, T_i]$. Floating payment at $T_i$: $N F_i \alpha_i$ |
| $P_d(0,T_i)$ | Discount factor from discount curve |
| $V_{\text{fixed}}$, $V_{\text{float}}$ | PV of fixed and floating legs |
| $PV_{\text{rec}}$, $PV_{\text{pay}}$ | Swap PV to fixed receiver / fixed payer |
| $A$ | Swap annuity (fixed-leg PVBP kernel): $A \equiv \sum_{i=1}^{n} \alpha_i P_d(0,T_i)$ |
| $PV01_K$ | Fixed-rate PV01: $\frac{\partial PV}{\partial K} \times 1\text{bp}$ (curves fixed) |
| $DV01$ | Bond-style dollar value of 1bp: $\frac{1}{10,000}\left(-\frac{dP}{dy}\right)$ |

### Rates and Units

- Rates are decimals (e.g., 3% = 0.03)
- $1 \text{ bp} = 0.0001$
- PV is in currency (e.g., USD)
- PV01/DV01 are in currency per bp

### Curve Bump Convention (Used in Worked Examples)

When we say "+1bp parallel bump to zero rates" we mean: bump continuously compounded zero rates by $\Delta z = +0.0001$ so

$$P_d^{\text{bump}}(0,T) = P_d(0,T) e^{-\Delta z \cdot T}$$

This is a chosen numerical convention for the examples; real desks may bump par rates, zero nodes, or instrument quotes.

---

## Core Concepts

### 1) Plain-Vanilla Interest Rate Swap: Legs and "Payer/Receiver"

**Formal Definition:**
An interest rate swap exchanges cashflows where one side pays fixed interest $K$ on a notional and the other pays floating interest on the same notional on scheduled dates.

**Intuition:**
Think of the swap as "turning fixed into floating" (or vice versa) without refinancing a bond: you overlay a swap on the funding or asset to transform cashflow exposure. Hull illustrates this "transform an asset/liability" interpretation.

**Trading/Risk Practice:**
Desks talk in **fixed payer** ("pay fixed, receive float") vs **fixed receiver** ("receive fixed, pay float"), because PV and risk signs flip.

---

### 2) Swap PV and the Par Swap Rate

**Formal Definition:**
The swap has value equal to the difference between the fixed leg and floating leg values. From Tuckman: $PV_{\text{rec}} = V_{\text{fixed}} - V_{\text{float}}$ and the sign determines who would pay to terminate.

**Par Swap Rate $K^*$:**
A newly initiated ("at market") swap typically has PV $\approx 0$. Tuckman frames this by noting both legs are worth par at initiation, so the fixed rate must equal the par swap rate.

**Intuition:**
$K^*$ is the fixed coupon that makes a fixed-rate bond's PV equal the floating-rate bond's PV (both $\approx$ par) when using the same valuation framework.

**Trading/Risk Practice:**
Par swap rates are quoted; traders mark swaps by comparing contracted $K$ to current $K^*$ (plus any credit/CSA/basis adjustments in real systems).

---

### 3) PV01, DV01, PVBP: What Do These Names Mean?

**Formal Definitions:**

| Measure | Definition |
|---------|------------|
| **Swap PV01** (Tuckman convention) | PV change for a 1bp change in the swap fixed rate $K$, holding discounting/projection assumptions fixed |
| **Bond DV01** (Tuckman) | $DV01 = \frac{1}{10,000}\left(-\frac{dP}{dy}\right)$ where $y$ is yield; DV01 is a positive magnitude for a long bond |

**Intuition:**
- Swap PV01-to-$K$ is basically the PV of a 1bp fixed coupon stream on the swap's payment schedule (the "annuity")
- Curve DV01 measures exposure to rates/curve shifts; it's what hedgers usually mean by "rate risk"

**Trading/Risk Practice:**
Risk reports may show PV01 but mean either:
- "swap PV01 to fixed rate" (annuity PV01), or
- "curve PV01" (bump the curve)

You must confirm which is used before hedging.

---

### 4) Why Multi-Curve Matters for Swaps

**Formal Definition (Conceptual):**
In a multi-curve world:
- **Discounting** uses a (nearly) risk-free curve (often OIS)
- **Projection** uses a curve associated with the floating reference rate (e.g., 3M term rate)

Hull describes valuing standard float-for-fixed swaps by projecting cashflows from forward rates and discounting at the risk-free (OIS) rate. Andersen & Piterbarg explains the historical move from single-curve to multi-curve frameworks with separate discount and forward curves.

**Intuition:**
A swap's PV depends on:
1. **Discounting** (how you PV any cashflow), and
2. **Projection** (what floating cashflows are expected to be)

If these curves differ, the risk "splits" into discount-curve PV01 and projection-curve PV01.

**Trading/Risk Practice:**
Curve choice is tied to CSA/collateral and market plumbing. Even without doing "full XVA," the PV01 can differ materially depending on whether discounting is OIS vs Libor, etc.

---

### 5) Hedging with Swaps: What DV01 Matching Does (and Does Not) Guarantee

**Formal Definition (DV01 Hedge Ratio):**
Tuckman's DV01 hedging logic: to hedge instrument A with instrument B using DV01, choose position sizes $F_A, F_B$ such that first-order price changes offset:

$$\boxed{F_B = -F_A \frac{DV01_A}{DV01_B}}$$

**Intuition:**
DV01 matching neutralizes small parallel rate moves (first-order).

**Trading/Risk Practice:**
DV01 hedges fail when:
- The curve twists (key-rate mismatch)
- Convexities differ (nonlinear risk)
- Instruments reference different curves/spreads (basis risk)

---

## Math and Derivations

### 1) Swap PV from Fixed Receiver vs Fixed Payer

From Tuckman's sign convention:

$$PV_{\text{rec}} = V_{\text{fixed}} - V_{\text{float}}, \quad PV_{\text{pay}} = V_{\text{float}} - V_{\text{fixed}} = -PV_{\text{rec}}$$

If $V_{\text{fixed}} - V_{\text{float}} > 0$, PV is positive to the fixed receiver and negative to the fixed payer.

**Assumption (valuation method):** "Assume forward rates are realized, then discount cashflows." Hull states this applies to standard float-for-fixed swaps, with forward rates from the floating reference yield curve and discounting at the risk-free (OIS) rate.

---

### 2) PV of Fixed and Floating Legs (Generic Cashflow Form)

Let fixed payments occur at $T_i$, $i = 1, \ldots, n$, with accrual $\alpha_i$.

**Fixed Leg PV:**

$$V_{\text{fixed}} = N \sum_{i=1}^{n} K \alpha_i P_d(0, T_i)$$

**Floating Leg PV** (projected by forwards, discounted by discount curve):

$$V_{\text{float}} = N \sum_{i=1}^{n} F_i \alpha_i P_d(0, T_i)$$

This is the direct "discounted projected cashflows" valuation consistent with Hull's description.

**Swap PV to Fixed Receiver:**

$$\boxed{PV_{\text{rec}} = N \sum_{i=1}^{n} (K - F_i) \alpha_i P_d(0, T_i)}$$

**Swap PV to Fixed Payer:**

$$\boxed{PV_{\text{pay}} = N \sum_{i=1}^{n} (F_i - K) \alpha_i P_d(0, T_i)}$$

**Unit Check:**
- $K, F_i$: rate per year (dimensionless)
- $\alpha_i$: year fraction (dimensionless)
- $P_d$: dimensionless
- Each term $(K - F_i)\alpha_i P_d$ is dimensionless; multiplied by $N$ gives currency units → PV ✓

---

### 3) Par Swap Rate $K^*$

A par swap has PV $\approx 0$ at initiation. Tuckman's narrative: at initiation both legs are worth par, so the fixed rate must equal the par swap rate.

Set $PV_{\text{rec}} = 0$:

$$0 = N \sum_{i=1}^{n} (K^* - F_i) \alpha_i P_d(0, T_i)$$

Solving:

$$K^* = \frac{\sum_{i=1}^{n} F_i \alpha_i P_d(0, T_i)}{\sum_{i=1}^{n} \alpha_i P_d(0, T_i)}$$

Define the **swap annuity**:

$$\boxed{A \equiv \sum_{i=1}^{n} \alpha_i P_d(0, T_i)}$$

Then:

$$\boxed{K^* = \frac{1}{A} \sum_{i=1}^{n} F_i \alpha_i P_d(0, T_i)}$$

---

### 4) Single-Curve Special Case: Forward Rates Implied by Discount Factors

Andersen & Piterbarg gives a forward-rate-from-discount-factors relationship:

$$L(t; t_i, t_{i+1}) = \frac{1}{\tau_i} \left( \frac{P^{(L)}(t, t_i)}{P^{(L)}(t, t_{i+1})} - 1 \right)$$

In the pre-crisis single-curve setting, one often assumed the same curve for discounting and projection. With $P^{(L)} = P_d = P$, the forward rates satisfy:

$$F_i = \frac{1}{\alpha_i} \left( \frac{P(0, T_{i-1})}{P(0, T_i)} - 1 \right)$$

Then the numerator simplifies (telescoping sum):

$$\sum_{i=1}^{n} F_i \alpha_i P(0, T_i) = \sum_{i=1}^{n} (P(0, T_{i-1}) - P(0, T_i)) = P(0, T_0) - P(0, T_n) = 1 - P(0, T_n)$$

So in the **single-curve case**:

$$\boxed{K^* = \frac{1 - P(0, T_n)}{A}}$$

**Sanity Checks:**
- If maturity $T_n \to 0$, $P(0, T_n) \to 1$, so $K^* \to 0$ ✓
- If rates increase so $P(0, T_n)$ decreases, $1 - P(0, T_n)$ increases, so $K^*$ tends to increase ✓

---

### 5) "Swap Annuity PV01" / PVBP of Fixed Leg (PV01-to-Fixed-Rate $K$)

Tuckman states that "PV01" in swap markets is normally the change in swap value for a 1bp change in its fixed rate, and it is tied to discount factors for fixed cashflows.

From the generic PV expression (fixed receiver):

$$PV_{\text{rec}} = N \sum_{i=1}^{n} (K - F_i) \alpha_i P_d(0, T_i)$$

Differentiate w.r.t. $K$ holding curves and forwards fixed:

$$\frac{\partial PV_{\text{rec}}}{\partial K} = N \sum_{i=1}^{n} \alpha_i P_d(0, T_i) = NA$$

Therefore the PV change for a +1bp change in fixed rate $K$ is:

$$\boxed{PV01_{K,\text{rec}} = (NA) \times 0.0001}$$

For the fixed payer, $PV_{\text{pay}} = -PV_{\text{rec}}$, so:

$$\boxed{PV01_{K,\text{pay}} = -(NA) \times 0.0001}$$

**Unit Check:**
- $A$ behaves like "PV of 1 unit-annuity" → dimensionless (often interpreted as "years PV-weighted")
- $NA$ is currency
- Multiplying by 0.0001 (bp) gives currency per bp ✓

---

### 6) DV01 as Curve Sensitivity

Tuckman notes you can compute a swap DV01 (fixed receiver) as DV01 of fixed leg minus DV01 of floating leg, but this combined measure is not used often because it mixes long-term fixed-rate risk with short-term floating-rate risk that are managed separately. He also notes floating-leg DV01 depends on time to next reset and can be very small relative to the fixed leg.

**Bond DV01 Definition (Tuckman):**

$$\boxed{DV01 = \frac{1}{10,000} \left( -\frac{dP}{dy} \right)}$$

**Useful Identity:**

$$\boxed{DV01 = \frac{P \cdot D_{\text{mod}}}{10,000}}$$

where $D_{\text{mod}}$ is modified duration and $P$ is price (per 100).

---

## Measurement & Risk

### A) Definitions (Be Explicit About What Is Being Bumped)

#### A1) "Swap Annuity PV01" / PVBP of the Fixed Leg

**Definition:**

$$PV01_K \equiv \frac{\partial PV}{\partial K} \times 1\text{bp}, \quad \text{with discount and projection curves held fixed}$$

Tuckman notes "PV01" in swap markets is normally this object: change in swap value for a 1bp change in its fixed rate.

**Formula:**

Let $A = \sum \alpha_i P_d(0, T_i)$. Then:
- Fixed receiver: $PV01_{K,\text{rec}} = +NA \times 0.0001$
- Fixed payer: $PV01_{K,\text{pay}} = -NA \times 0.0001$

**Intuition:**
Change the fixed coupon by 1bp: you change each fixed payment by $N \times 1\text{bp} \times \alpha_i$, and PV those changes with the discount curve. That sum is the "annuity."

**Desk Practice:**
This measure is central for:
- Swaption annuity scaling (swap PV is often approximated as annuity × (swap rate difference))
- Quoting PVBP for swap books
- Understanding exposure to a re-strike / renegotiation of fixed rate

---

#### A2) Curve PV01/DV01

**Definition (Generic):**

Pick a curve $C$ (discount or projection) and a bump rule $B$ (e.g., +1bp parallel to zero rates). Define:

$$PV01_C \equiv PV(C \text{ bumped by } 1\text{bp via } B) - PV(C)$$

This is a scenario/bump definition, not a unique mathematical object until $(C, B)$ are specified.

**Discount-Curve PV01 vs Projection-Curve PV01:**
- **Discount PV01:** bump $P_d(0,T)$ (or the discounting zero rates) and hold projection forwards fixed
- **Projection PV01:** bump the projection curve (i.e., the forwards $F_i$ implied by that curve) and hold discounting fixed

**Why "Bump Which Curve?" Changes Risk Numbers (Multi-Curve):**

Hull's description explicitly separates projection ("forward rates from the yield curve for the floating reference rate") from discounting ("discounted at the risk-free (OIS) rate"). Andersen & Piterbarg explains the industry move to multiple yield curves, with one used for discounting and one per index for forecasting.

**I'm not sure (market convention alert):**
Different desks define "curve PV01" differently:
- Bumping zero rates vs bumping par swap rates vs bumping instrument quotes and rebuilding
- "Bump and rebuild" vs analytic Jacobian approaches

To be certain, we'd need your desk's risk-definition document: which curve, which nodes, which rebuild rule, and whether bumps are in yield space or DF space.

---

#### A3) Payer vs Receiver Swap PV Sign Conventions

**Definition (Tuckman):**
- Fixed receiver PV: $PV_{\text{rec}} = V_{\text{fixed}} - V_{\text{float}}$
- Fixed payer PV: $PV_{\text{pay}} = V_{\text{float}} - V_{\text{fixed}}$

If $V_{\text{fixed}} - V_{\text{float}} > 0$: PV positive to fixed receiver; if the reverse, PV positive to fixed payer.

**Risk Sign Implication:**
- Fixed receiver resembles long fixed-rate bond (positive duration)
- Fixed payer resembles short fixed-rate bond (negative duration)

---

#### A4) How Swap PV Depends on Discount vs Projection Curves

| Curve | Effect |
|-------|--------|
| **Discount curve** | Enters through $P_d(0, T_i)$ multiplying all cashflows (fixed and floating) |
| **Projection curve** | Enters through $F_i$ (forward floating rates) determining expected floating cashflows |

Multi-curve context: discounting at OIS vs projecting a term rate curve is explicitly described in Hull; Andersen & Piterbarg describes separate discount and forward curves as the post-crisis standard.

---

### B) Relationships to Bonds

#### B1) Swap = Fixed-Rate Bond − Floating-Rate Bond (Conceptual Replication)

Hull shows for OIS that if you add notional exchanges at maturity, the swap cashflows are equivalent to exchanging a floating-rate bond for a fixed-rate bond; the floating-rate bond is worth par, so the fixed-rate bond must also be worth par when the swap starts at zero value.

Tuckman similarly notes at initiation both legs of a new swap at "LIBOR flat" are worth par (100), implying the fixed rate equals the par swap rate.

**Implication:** The fixed leg behaves like a coupon bond's PV ("annuity"), and the floating leg behaves like a par floater (low duration except for the next reset interval).

#### B2) Why Swap PV01 and Bond DV01 Differ (Even at Same Maturity)

**Key Drivers:**

1. **Cashflow timing and frequency:** Swaps often pay fixed semiannually and float quarterly (market standard), whereas many bonds pay semiannually; the PV weights differ

2. **Floating leg duration is short:** Tuckman emphasizes floating-leg DV01 depends on time to next payment and can be tiny compared to the fixed leg

3. **Curve choice:** In multi-curve, fixed leg is discounted with the discount curve; floating cashflows depend on the projection curve. So the DV01 depends on which curve you bump

4. **Bond-specific features:** Bonds have embedded options (callable/putable), credit spread components, and special repo effects—swaps generally don't (though they have CSA, credit, and basis effects)

**Practical Hedging Interpretation:** Tuckman notes that in "asset swap" style contexts, you should not hedge all bond risk with swaps because you would leave floating-rate risk unhedged; you often hedge floating exposure with short-term futures (e.g., Eurodollars) instead.

---

### C) Hedging with Swaps

#### C1) DV01/PV01 Hedge Ratios

**Generic DV01 Hedge (Tuckman):**

$$F_B = -F_A \frac{DV01_A}{DV01_B}$$

to offset first-order price changes under small yield/curve moves.

**Hedging a Bond's DV01 Using a Swap:**

1. Compute bond DV01 (per bp) under the same bump definition used for swap DV01
2. Compute swap curve DV01 (per bp) per unit notional
3. Choose swap notional $N_{\text{swap}}$ such that:
   $$\Delta PV_{\text{bond}} + \Delta PV_{\text{swap}} \approx 0$$

**Direction:**
Long bond loses when rates rise ⇒ hedge with a position that gains when rates rise, often a fixed payer swap (negative duration).

#### C2) Hedging a Swap Book Using Bonds/Futures (Conceptual)

Fixed leg is long-duration; floating leg is short-duration.

In practice (Tuckman), because fixed-rate and floating-rate risks are managed separately, you might hedge:
- Fixed-leg risk with swaps/bonds
- Floating-leg risk with short-dated instruments (futures, FRAs, short swap buckets)

#### C3) Why Swap Hedges Fail (Preview)

| Failure Mode | Description |
|--------------|-------------|
| **Curve twist / key-rate mismatch** | DV01-neutral under parallel shifts does not mean neutral under twists. Tuckman discusses "bucket exposure" of swaps to forward rates |
| **Convexity mismatch** | DV01 hedges are first-order; the second-order term involves convexity |
| **Basis risk** | Treasury vs swap curve (swap spread changes), discount vs projection basis (multi-curve), instrument-quote vs node risk |
| **Curve-construction dependence** | If the curve is built from market instruments, bumping one quote and rebuilding redistributes risk across the curve |

---

## Worked Examples

### Global Numeric Conventions for Examples A–G

| Parameter | Value |
|-----------|-------|
| Notional $N$ | $100,000,000 (USD 100mm) |
| Payment dates | Annual: $T_1 = 1$, $T_2 = 2$, $T_3 = 3$ years |
| Day count | 30/360 with $\alpha_i = 1.00$ each year (simplification) |
| Discount factors | $P_d(0,1) = 0.98$, $P_d(0,2) = 0.955$, $P_d(0,3) = 0.93$ |
| Projection forwards | $F_1 = 3.00\%$, $F_2 = 3.40\%$, $F_3 = 3.80\%$ |
| Swap PV sign | Report PV to fixed payer unless stated |
| Bump rule | +1bp parallel to continuously compounded zero rates: $P_d^b(0,T) = P_d(0,T)e^{-0.0001T}$ |

---

### Example A: Fixed-Leg Annuity / PVBP

**Task:** Given $P_d(0, T_i)$ and $\alpha_i$, compute $A = \sum_{i=1}^{3} \alpha_i P_d(0, T_i)$ and interpret $A$ as fixed-leg PV01 per 1 unit of fixed rate, then convert to per bp.

**Step 1: Compute annuity**

$$A = 1 \cdot 0.98 + 1 \cdot 0.955 + 1 \cdot 0.93 = 2.865$$

**Step 2: Interpret**

PV change for a change $\Delta K$ in fixed rate (holding curves fixed) is:

$$\Delta PV_{\text{rec}} \approx NA \cdot \Delta K, \quad \Delta PV_{\text{pay}} \approx -NA \cdot \Delta K$$

**Step 3: Convert to per bp**

$$PV01_{K,\text{rec}} = NA \times 0.0001 = 100,000,000 \times 2.865 \times 0.0001 = \boxed{28,650}$$

So the fixed receiver gains +\$28,650 for a +1bp increase in $K$; fixed payer loses −\$28,650.

---

### Example B: Par Swap Rate and PV Check

**Task:** Compute par swap rate $K^*$ using provided projection forwards (multi-curve) and verify PV $\approx 0$ at $K^*$.

**Formula:**

$$K^* = \frac{\sum_{i=1}^{3} F_i \alpha_i P_d(0, T_i)}{\sum_{i=1}^{3} \alpha_i P_d(0, T_i)} = \frac{\sum F_i P_i}{A}$$

**Step 1: Numerator**

| $i$ | $F_i$ | $P_i$ | $F_i \times P_i$ |
|-----|-------|-------|------------------|
| 1 | 0.03 | 0.98 | 0.0294 |
| 2 | 0.034 | 0.955 | 0.03247 |
| 3 | 0.038 | 0.93 | 0.03534 |
| **Sum** | | | **0.09721** |

**Step 2: Par rate**

$$K^* = \frac{0.09721}{2.865} \approx 0.03393 = \boxed{3.393\%}$$

**Step 3: PV check at $K^*$**

For fixed payer: $PV_{\text{pay}} = N(\sum F_i P_i - K^* A)$

Compute $K^* A = 0.03393 \times 2.865 \approx 0.09720945$

$$PV_{\text{pay}} \approx 100,000,000(0.09721 - 0.09720945) \approx 100,000,000(0.00000055) \approx 55$$

PV is essentially zero (rounding error), consistent with "par swap rate makes initiation require no cash exchange." ✓

---

### Example C: Off-Market Swap NPV

**Task:** Price a payer swap at $K = 4.00\% \neq K^*$. Compute:

$$PV_{\text{pay}} = PV(\text{float}) - PV(\text{fixed}) = N(\sum F_i P_i - KA)$$

**Step 1: Fixed PV factor**

$$KA = 0.04 \times 2.865 = 0.1146$$

**Step 2: Float PV factor (projected)**

$$\sum F_i P_i = 0.09721 \quad \text{(from Example B)}$$

**Step 3: Swap PV**

$$PV_{\text{pay}} = 100,000,000(0.09721 - 0.1146) = 100,000,000(-0.01739) = \boxed{-1,739,000}$$

**Interpretation:**
- Fixed payer has negative PV (pays fixed too high vs market)
- Fixed receiver PV is $+1,739,000$ (opposite sign per Tuckman's convention)

---

### Example D: PV01 w.r.t Fixed Rate $K$

**Task:** For the off-market swap in Example C, compute PV change for +1bp change in fixed rate $K$ holding curves fixed; confirm equals $-A \times 1\text{bp} \times N$ for payer.

**Step 1: Analytic PV01-to-$K$**

$$PV01_{K,\text{pay}} = -NA \times 0.0001 = -28,650$$

**Step 2: Confirm by repricing**

New fixed rate: $K' = 4.01\% = 0.0401$

Compute new fixed PV factor: $K' A = 0.0401 \times 2.865 = 0.1148865$

New PV:
$$PV'_{\text{pay}} = 100,000,000(0.09721 - 0.1148865) = -1,767,650$$

Difference:
$$PV'_{\text{pay}} - PV_{\text{pay}} = (-1,767,650) - (-1,739,000) = \boxed{-28,650}$$

This matches the annuity PV01 exactly. ✓

---

### Example E: Discount-Curve PV01

**Task:** Apply a +1bp parallel bump to the discount curve zero rates, hold projection forwards fixed, reprice, compute:

$$PV01_{\text{disc}} = PV^{\text{bump disc}} - PV^{\text{base}}$$

**Bump definition:** $P_d^b(0,T) = P_d(0,T)e^{-0.0001T}$

**Step 1: Bumped discount factors**

Using $e^{-0.0001T} \approx 1 - 0.0001T$:

| $T$ | $P_d(0,T)$ | $e^{-0.0001T}$ | $P_d^b(0,T)$ |
|-----|------------|----------------|--------------|
| 1 | 0.98 | 0.9999 | 0.979902 |
| 2 | 0.955 | 0.9998 | 0.954809 |
| 3 | 0.93 | 0.9997 | 0.929721 |

**Step 2: Recompute annuity**

$$A^b = 0.979902 + 0.954809 + 0.929721 = 2.864432$$

**Step 3: Recompute projected-float PV factor (forwards held fixed)**

| $i$ | $F_i$ | $P_i^b$ | $F_i \times P_i^b$ |
|-----|-------|---------|-------------------|
| 1 | 0.03 | 0.979902 | 0.02939706 |
| 2 | 0.034 | 0.954809 | 0.032463506 |
| 3 | 0.038 | 0.929721 | 0.035329398 |
| **Sum** | | | **0.097189964** |

**Step 4: Reprice payer swap at $K = 4\%$**

$$PV_{\text{pay}}^b = N(\sum F_i P_i^b - K A^b)$$

Compute $K A^b = 0.04 \times 2.864432 = 0.11457728$

$$PV_{\text{pay}}^b = 100,000,000(0.097189964 - 0.11457728) = -1,738,731.6$$

**Step 5: Discount-curve PV01**

$$PV01_{\text{disc}} = PV_{\text{pay}}^b - PV_{\text{pay}} = (-1,738,731.6) - (-1,739,000) = \boxed{+268.4}$$

**Interpretation:** With forwards fixed, the discount bump mainly scales PV of both legs; the net effect can be small relative to projection PV01.

---

### Example F: Projection-Curve PV01

**Task:** Apply +1bp parallel bump to the projection curve while holding discount curve fixed; compute $PV01_{\text{proj}}$.

**Chosen bump definition:** Add $+0.0001$ to each forward $F_i$: $F_i^b = F_i + 0.0001$

**I'm not sure** this matches your desk's exact projection-curve bump rule (some bump zero rates of the index curve and recompute forwards). To be certain, we need the desk's curve-bump methodology.

**Step 1: Recompute $\sum F_i P_i$**

$$\sum (F_i + 0.0001)P_i = \sum F_i P_i + 0.0001 \sum P_i = 0.09721 + 0.0001(2.865) = 0.09721 + 0.0002865 = 0.0974965$$

**Step 2: Reprice payer swap**

Fixed part unchanged: $KA = 0.1146$

$$PV_{\text{pay}}^b = 100,000,000(0.0974965 - 0.1146) = -1,710,350$$

**Step 3: Projection PV01**

$$PV01_{\text{proj}} = PV_{\text{pay}}^b - PV_{\text{pay}} = (-1,710,350) - (-1,739,000) = \boxed{+28,650}$$

**Interpretation:** Projection PV01 matches $NA \times 1\text{bp}$ because PV is linear in forwards with weights $\alpha_i P_i$.

---

### Example G: Single-Curve vs Multi-Curve PV01 Difference

**Task:** Recompute Example E/F under a single-curve setup where projection=discount, and show how PV01 attribution changes.

**Single-curve assumption:** Forwards are implied by discount factors:

$$F_i = \frac{1}{\alpha_i}\left(\frac{P(0, T_{i-1})}{P(0, T_i)} - 1\right)$$

We keep the same contract: payer swap with $K = 4.00\%$, $N = 100$mm, annual payments.

**Step 1: Compute single-curve implied forwards (annual accrual $\alpha = 1$)**

| $i$ | Formula | $F_i$ |
|-----|---------|-------|
| 1 | $\frac{1}{0.98} - 1$ | 2.04082% |
| 2 | $\frac{0.98}{0.955} - 1$ | 2.61780% |
| 3 | $\frac{0.955}{0.93} - 1$ | 2.68817% |

**Useful identity (single-curve):**

$$\sum F_i P_i = 1 - P_3 = 1 - 0.93 = 0.07$$

**Base PV (single curve):**

$$PV_{\text{pay}}^{\text{single}} = N((1 - P_3) - KA) = 100,000,000(0.07 - 0.1146) = -4,460,000$$

**Step 2: "Total PV01" under +1bp bump to the single curve**

Bumped DFs from Example E: $P_1^b = 0.979902$, $P_2^b = 0.954809$, $P_3^b = 0.929721$

Then $(1 - P_3^b) = 0.070279$, and $A^b = 2.864432$

Reprice:
$$PV_{\text{pay}}^{\text{single},b} = 100,000,000(0.070279 - 0.04(2.864432)) = 100,000,000(-0.04429828) = -4,429,828$$

**Total PV01:**
$$PV01_{\text{total}}^{\text{single}} = (-4,429,828) - (-4,460,000) = \boxed{+30,172}$$

**Step 3: Artificial attribution**

| Type | PV01 |
|------|------|
| Discount-only (DFs bumped, forwards fixed) | ≈ +825.6 |
| Projection-only (forwards +1bp, DFs fixed) | +28,650 |

**Interpretation:**
- In multi-curve, discount-only and projection-only PV01 are conceptually meaningful because curves are distinct
- In single-curve, discounting and forwards are mechanically linked; bumping the curve moves both together. Attempting to split risk into "discount" and "projection" becomes a modeling artifact

---

### Examples H–J: Bond Mapping and Hedging Scenarios

#### Global Numeric Conventions for Examples H–I

| Parameter | Value |
|-----------|-------|
| Structure | 5-year annual-pay with dates $T_1, \ldots, T_5 = 1, \ldots, 5$ |
| Discount factors | $P(0,1) = 0.98$, $P(0,2) = 0.955$, $P(0,3) = 0.93$, $P(0,4) = 0.90$, $P(0,5) = 0.87$ |
| Accrual | $\alpha_i = 1$ |
| Curve bump | +1bp to continuously compounded zero rates: $P^b(0,T) = P(0,T)e^{-0.0001T}$ |

---

### Example H: Swap DV01 Mapping to Bond DV01

**Task:** Provide a bond, compute its DV01, compute swap PV01 per notional, solve for swap notional that DV01-hedges the bond. Show scaling to \$10mm.

We will hedge a \$10mm face 5-year 3% annual coupon bond using a 5-year payer swap.

#### H1) Bond Pricing and DV01

**Bond cashflows per \$100 face:**
- Years 1–4: coupon = 3
- Year 5: coupon + principal = 103

**Base price:**

$$P_0 = 3(P_1 + P_2 + P_3 + P_4) + 103 P_5$$

Sum $P_1 + \cdots + P_4 = 0.98 + 0.955 + 0.93 + 0.90 = 3.765$

Coupons PV $= 3 \times 3.765 = 11.295$
Final PV $= 103 \times 0.87 = 89.61$

$$P_0 = 100.905$$

**Bumped discount factors (+1bp):**

| $T$ | $P(0,T)$ | $P^b(0,T)$ |
|-----|----------|------------|
| 1 | 0.98 | 0.979902 |
| 2 | 0.955 | 0.954809 |
| 3 | 0.93 | 0.929721 |
| 4 | 0.90 | 0.899640 |
| 5 | 0.87 | 0.869565 |

Sum $P_1^b + \cdots + P_4^b = 3.764072$
Coupons PV bumped $= 3 \times 3.764072 = 11.292216$
Final PV bumped $= 103 \times 0.869565 = 89.565206$

$$P^b \approx 100.857422$$

**Bond DV01 (per \$100 face):**

$$DV01_{\text{bond}} \approx 100.905 - 100.857422 = 0.047578 \text{ per \$100}$$

**Dollar DV01 for \$10mm face:**

$$DV01_{\text{bond},\$} = \frac{10,000,000}{100} \times 0.047578 = 100,000 \times 0.047578 = \boxed{4,757.8 \text{ \$/bp}}$$

#### H2) Swap PV01 (Annuity PV01) per Notional

**Compute 5y swap annuity:**

$$A_{5y} = \sum_{i=1}^{5} P(0,i) = 0.98 + 0.955 + 0.93 + 0.90 + 0.87 = 4.635$$

**Swap PV01-to-fixed-rate (annuity PV01):**

$$PV01_K = NA_{5y} \times 0.0001$$

Per \$1mm notional:
$$PV01_{K,\$1\text{mm}} = 1,000,000 \times 4.635 \times 0.0001 = 463.5 \text{ \$/bp}$$

Per \$100mm notional: $46,350$ \$/bp

#### H3) Choose Swap Notional to DV01-Hedge the Bond

We want bond loss for +1bp to be offset by swap gain.

- Long bond: +1bp rates → bond price down by $4,757.8$
- A payer swap tends to gain when rates rise (short fixed-rate exposure)

**Approximate hedge notional using annuity PV01:**

$$N_{\text{swap}} \approx \frac{DV01_{\text{bond},\$}}{463.5} \text{ (in \$1mm units)}$$

$$\frac{4,757.8}{463.5} \approx 10.27 \text{ mm}$$

**Result:** Hedge \$10mm bond with approximately **\$10.3mm payer swap**.

---

### Example I: Hedging Failure from Curve Twist

**Task:** Construct a bond hedged with a maturity-matched swap so parallel DV01 is ~0. Then apply a twist scenario (short end +10bp, long end −10bp). Show residual P&L and explain why.

To make the twist effect clear, hedge a 5-year zero-coupon bond with a 5-year payer swap.

#### I1) Instruments and Base Values

- **Bond:** \$10mm face, 5-year zero, price per \$100 = $100 P_5 = 87.0$
- **Swap:** 5-year annual-pay payer swap, fixed rate set at par:

$$K^* = \frac{1 - P_5}{A_{5y}} = \frac{1 - 0.87}{4.635} = \frac{0.13}{4.635} \approx 2.80475\%$$

#### I2) Parallel DV01 Matching

**Bond DV01 under +1bp parallel bump:**

Bumped $P_5^b = 0.869565109$
Price per \$100 bumped: $100 P_5^b = 86.9565109$

$$DV01_{\text{bond}} = 87.0 - 86.9565109 = 0.0434891 \text{ per \$100}$$

Dollar DV01 for \$10mm face:
$$DV01_{\text{bond},\$} = 100,000 \times 0.0434891 = 4,348.91 \text{ \$/bp}$$

**Swap PV change under +1bp parallel bump:**

Under single curve, payer swap PV per \$1 notional after bump:
$$PV_{\text{pay}}^b = (1 - P_5^b) - K^* A^b = 0.130434891 - 0.129961937 = 0.000472954$$

Per \$100mm: ≈ 47,295 \$/bp

**Choose swap notional to DV01-neutralize:**

$$N_{\text{swap}} \approx \frac{4,348.91}{0.000472954} \approx 9,200,000$$

Hedge with **\$9.20mm payer swap**. Parallel DV01 is ~0 by construction.

#### I3) Apply Twist Scenario

**Twist definition:**
- Short end (1y and 2y): +10bp
- Middle (3y): 0bp
- Long end (4y and 5y): −10bp

**Twisted DFs:** $P^{tw}(0,T) = P(0,T)e^{-\Delta z(T) \cdot T}$

| $T$ | $\Delta z$ | $P^{tw}(0,T)$ |
|-----|------------|---------------|
| 1 | +0.001 | 0.9790205 |
| 2 | +0.001 | 0.95309 |
| 3 | 0 | 0.93 |
| 4 | −0.001 | 0.9036072 |
| 5 | −0.001 | 0.874361 |

**Bond P&L under twist:**

$$P_{\text{bond}}^{tw} = 100 P_5^{tw} = 87.4361$$

Change per \$100: $\Delta P_{\text{bond}} = 0.4361$

Dollar P&L for \$10mm face:
$$\Delta PV_{\text{bond}} = 100,000 \times 0.4361 = +43,610$$

**Swap P&L under twist:**

$$A^{tw} = 0.9790205 + 0.95309 + 0.93 + 0.9036072 + 0.874361 = 4.6400787$$
$$1 - P_5^{tw} = 0.125639$$
$$K^* A^{tw} = 0.0280475 \times 4.6400787 = 0.130142607$$

Swap PV per \$1 notional:
$$PV_{\text{pay}}^{tw} = 0.125639 - 0.130142607 = -0.004503607$$

Dollar PV for $N_{\text{swap}} = 9.20$mm:
$$\Delta PV_{\text{swap}} \approx -0.004503607 \times 9,200,000 \approx -41,433$$

**Residual P&L (hedged book):**

$$\Delta PV_{\text{total}} \approx +43,610 + (-41,433) = \boxed{+2,177}$$

**Explanation:**
- DV01 matching neutralized parallel moves, not twists
- The zero bond's exposure is concentrated at the 5-year point, while the swap's fixed leg has meaningful PV weight at earlier maturities via its annuity cashflows
- This is a "key-rate mismatch" effect, aligned with Tuckman's discussion of bucket exposure

---

### Example J: Basis Risk: Treasury vs Swap (Toy)

**Task:** Show a toy scenario where Treasury yields move differently from swap rates (swap spread move), leaving residual P&L.

**Setup (toy):**
- Treasury bond position: \$10mm face, assume DV01 (to Treasury curve) = 4,758 \$/bp
- Hedge with payer swap sized so swap DV01 (to swap curve) matches 4,758 \$/bp

**Hedge size (simplified):**
If 5y swap DV01 is ≈ 463.5 \$/bp per \$1mm (Example H annuity PV01 proxy), hedge notional ≈ 10.27mm so swap sensitivity ≈ 4,758 \$/bp.

**Scenario: Swap spread widens**

Treasury yields rise +10bp (bond loses):
$$\Delta PV_{\text{bond}} \approx -4,758 \times 10 = -47,580$$

Swap rates rise only +6bp (payer swap gains less than expected):
$$\Delta PV_{\text{swap}} \approx +4,758 \times 6 = +28,548$$

**Residual:**
$$\Delta PV_{\text{total}} \approx -47,580 + 28,548 = \boxed{-19,032}$$

**Interpretation:**
- The hedge assumed Treasury and swap curves move together (stable swap spread)
- In reality, different instruments embed different liquidity/credit/hedging frictions; the difference is "basis risk"

**I'm not sure (source scope):**
The provided sources in this chapter's required list do not give a full empirical decomposition of swap spread moves; the calculation above is a toy illustration of basis risk mechanics.

---

## Practical Notes

### 5.1 Naming Collisions and Reporting Conventions

| Term | Possible Meanings |
|------|-------------------|
| **PV01 / PVBP / annuity** | Tuckman: PV change for 1bp change in fixed rate $K$. Many risk systems: "curve PV01" (DV01-like), possibly split by curve nodes |
| **DV01 vs duration** | DV01 is dollar sensitivity; duration is percentage sensitivity. Tuckman links DV01 to modified duration via $DV01 = PD_{\text{mod}}/10,000$ |
| **"Rollover" risk measure choices** | Swap DV01 can be computed as fixed leg DV01 minus floating leg DV01, but Tuckman notes this combined measure is often avoided because the risks are managed separately |

### 5.2 Common Pitfalls

1. **Mixing PV01-to-fixed-rate with PV01-to-curve:**
   - $PV01_K$ answers "what if contract fixed rate changes?"
   - Curve PV01 answers "what if market curve moves?"

2. **Wrong sign for payer vs receiver:**
   - Fixed payer PV is the negative of fixed receiver PV

3. **Scaling errors:**
   - per \$100 vs per \$1mm vs per \$100mm
   - $1\text{bp} = 0.0001$

4. **Inconsistent bump definitions between risk report and hedge build:**
   - "Parallel bump to zero rates" vs "bump par swap rates" vs "bump instrument quotes"

5. **Forgetting multi-curve split:**
   - Discount PV01 and projection PV01 can differ substantially (Examples E/F)

### 5.3 Implementation Pitfalls

| Pitfall | Impact |
|---------|--------|
| **Schedule generation** | Stubs, business day adjustments, day-count conventions (Actual/360 vs 30/360) |
| **Accrual factors** | PV01 scales with $\alpha_i$; a 28/90 stub is not "0.25" if you approximate |
| **Bump-and-rebuild vs direct zero bumps** | Rebuilding ensures bumped curve remains arbitrage-free but changes multiple nodes at once |
| **Numerical stability** | Bump size: 1bp is standard, but too small can be noisy; too large introduces convexity |

### 5.4 Verification Tests (Desk-Relevant)

| Test | What to Check |
|------|---------------|
| **Par swap test** | PV $\approx 0$ when using the same curves to compute par rate and to discount/project (Example B logic) |
| **PV01-to-fixed-rate test** | $PV01_K = \pm NA \times 1\text{bp}$ (Example D) |
| **Notional linearity** | PV and PV01 scale linearly with notional $N$ in plain-vanilla setting |
| **Discount vs projection sanity** | Discount PV01 should generally be smaller when forwards held fixed; projection PV01 should scale with annuity |

---

## Summary & Recall

### 10-Bullet Executive Summary

1. Swap PV sign depends on whether you are fixed receiver or fixed payer; $PV_{\text{pay}} = -PV_{\text{rec}}$

2. Par swap rate $K^*$ is the fixed rate that makes PV $\approx 0$ at initiation

3. In Tuckman's swap-market convention, PV01 usually means PV change for 1bp change in fixed rate $K$

4. The fixed-leg annuity $A = \sum \alpha_i P_d(0, T_i)$ drives PV01-to-$K$: $PV01_K = \pm NA \times 1\text{bp}$

5. Curve PV01/DV01 requires specifying: which curve and how it's bumped/rebuilt

6. Multi-curve valuation separates projection (forwards) from discounting (risk-free/OIS), so PV01 can split into discount PV01 and projection PV01

7. Swap resembles fixed-rate bond minus floating-rate bond; floating leg is near par with short duration, so most duration sits in the fixed leg

8. DV01 hedging uses $F_B = -F_A(DV01_A / DV01_B)$ to match first-order exposure

9. DV01-neutral ≠ riskless: twists (key-rate mismatch), convexity mismatch, and basis risk create residual P&L

10. Always reconcile the desk's definition of PV01/DV01 and bump method before using PV01 numbers for hedge ratios

---

### Cheat Sheet of Formulas

| Formula | Expression |
|---------|------------|
| **Swap PV (fixed receiver)** | $PV_{\text{rec}} = N \sum_{i=1}^{n} (K - F_i) \alpha_i P_d(0, T_i)$ |
| **Swap PV (fixed payer)** | $PV_{\text{pay}} = -PV_{\text{rec}} = N \sum_{i=1}^{n} (F_i - K) \alpha_i P_d(0, T_i)$ |
| **Annuity / fixed-leg PVBP kernel** | $A = \sum_{i=1}^{n} \alpha_i P_d(0, T_i)$ |
| **Par swap rate** | $K^* = \frac{\sum_{i=1}^{n} F_i \alpha_i P_d(0, T_i)}{A}$ |
| **Single-curve simplification** | $F_i = \frac{1}{\alpha_i}\left(\frac{P(0, T_{i-1})}{P(0, T_i)} - 1\right), \quad K^* = \frac{1 - P(0, T_n)}{A}$ |
| **PV01-to-fixed-rate $K$** | $PV01_{K,\text{rec}} = +NA \times 0.0001, \quad PV01_{K,\text{pay}} = -NA \times 0.0001$ |
| **Bond DV01 (Tuckman)** | $DV01 = \frac{1}{10,000}\left(-\frac{dP}{dy}\right) = \frac{P D_{\text{mod}}}{10,000}$ |
| **DV01 hedge ratio (Tuckman)** | $F_B = -F_A \frac{DV01_A}{DV01_B}$ |

---

### Flashcards (30)

| Q | A |
|---|---|
| What is $PV_{\text{rec}}$ for a swap? | $PV_{\text{rec}} = V_{\text{fixed}} - V_{\text{float}}$ |
| What is $PV_{\text{pay}}$ for a swap? | $PV_{\text{pay}} = V_{\text{float}} - V_{\text{fixed}} = -PV_{\text{rec}}$ |
| When is a swap PV positive to the fixed receiver? | When $V_{\text{fixed}} - V_{\text{float}} > 0$ |
| What is the par swap rate? | The fixed rate $K^*$ that makes swap PV $\approx 0$ at initiation |
| In Tuckman's swap discussion, what does PV01 "normally mean"? | Change in swap value for a 1bp change in the fixed rate $K$ |
| Define the swap annuity $A$ | $A = \sum \alpha_i P_d(0, T_i)$ |
| What is $PV01_K$ for a fixed receiver? | $+NA \times 1\text{bp}$ |
| What is $PV01_K$ for a fixed payer? | $-NA \times 1\text{bp}$ |
| How does Hull describe valuing a standard float-for-fixed swap? | Project cashflows from forward rates for the floating reference and discount at the risk-free (OIS) rate |
| What is "multi-curve" in one sentence? | Separate curves for discounting and for forecasting floating rates |
| Why can discount PV01 be smaller than projection PV01? | Discounting scales PV of both legs, while projection bumps change expected floating cashflows directly |
| Give the bond DV01 definition in Tuckman | $DV01 = \frac{1}{10,000}\left(-\frac{dP}{dy}\right)$ |
| DV01 sign: what happens to a long bond when yields rise 1bp? | PV decreases by approximately DV01 (DV01 is reported as positive magnitude) |
| How is DV01 related to modified duration? | $DV01 = \frac{P D_{\text{mod}}}{10,000}$ |
| DV01 hedge ratio formula? | $F_B = -F_A(DV01_A / DV01_B)$ |
| Why might swap DV01 be managed by leg? | Fixed and floating risks are different horizons; combined measure mixes them |
| What drives floating-leg DV01? | Mainly the time to next reset/payment; often small |
| Replication idea: swap equals what two bonds? | A fixed-rate bond and a floating-rate bond (conceptual) |
| In Hull's OIS discussion, why is the fixed-rate bond worth par at inception? | Because the swap is zero and the floating-rate bond is worth par |
| What is "basis risk" in a Treasury vs swap hedge? | Treasury yields and swap rates can move differently (swap spread changes) |
| What is the key-rate mismatch problem? | DV01-neutral under parallel shifts but not neutral under twists |
| What is convexity mismatch? | Second-order (nonlinear) price responses differ, leaving residual |
| What is the most common scaling error in PV01 reporting? | Confusing per \$100 with per \$1mm and forgetting the $0.0001$ bp factor |
| What must be specified to define "curve PV01"? | Which curve, what bump, and rebuild rule |
| In multi-curve, what does discount curve affect? | PV of all cashflows via discount factors |
| In multi-curve, what does projection curve affect? | Floating cashflows via forwards $F_i$ |
| How do you sanity-check a swap PV engine? | Verify a par swap has PV $\approx 0$ using consistent curves |
| How do you sanity-check $PV01_K$? | Reprice with $K \pm 1$bp holding curves fixed; PV change should match $\pm NA \times 1$bp |
| Why shouldn't you hedge "all bond risk with swaps" in an asset-swap context? | You may leave floating-rate risk unhedged; use short-rate futures for that component |
| What is the desk question you must ask before using PV01? | "Is PV01 defined vs the fixed rate $K$ or vs a curve bump (and which curve)?" |

---

## Mini Problem Set (16 Questions)

1. Compute annuity $A$ and $PV01_K$ for a 2-year semiannual fixed leg with given discount factors and accruals.

2. Given discount factors and forwards, compute par swap rate $K^*$ and verify PV $\approx 0$.

3. For a payer swap off-market by +50bp, compute PV and $PV01_K$.

4. Using Tuckman's DV01 definition, compute DV01 of a 5-year zero-coupon bond given a yield and price.

5. Compute curve PV01 of a swap under a +1bp parallel bump to zero rates (single-curve) and compare to annuity PV01.

6. In a multi-curve setting, explain (qualitatively) why discount PV01 and projection PV01 can have different signs for an off-market swap.

7. Given a bond DV01 and a swap DV01 per \$1mm, compute the swap notional needed for a DV01-neutral hedge.

8. Show with numbers that a DV01-neutral hedge can have residual P&L under a twist.

9. Describe how stub periods can materially change annuity PV01.

10. Describe the difference between bumping zero rates and bumping par swap rates in curve PV01 calculations.

11. A swap's floating leg is reset quarterly. Explain why its DV01 is related to the time to the next reset (qualitative).

12. Explain why a swap hedge against a Treasury bond can have residual due to swap spread changes.

13. For a fixed receiver swap, predict the sign of PV change when all rates increase (qualitative).

14. Sketch how you would compute key-rate DV01s for a swap's fixed leg (no full derivation required).

15. Explain why convexity mismatch can matter for large rate moves even when DV01 is neutral.

16. Explain how collateral/CSA could affect discounting and thus curve PV01 attribution (briefly).

---

### Brief Solution Sketches (Questions 1–8)

1. $A = \sum \alpha_i P_i$. $PV01_K = \pm NA \times 0.0001$ with sign depending on payer/receiver.

2. $K^* = (\sum F_i \alpha_i P_i) / (\sum \alpha_i P_i)$. Plug back into PV formula; PV should be $\approx 0$.

3. $PV_{\text{pay}} = N(\sum F_i \alpha_i P_i - KA)$. $PV01_{K,\text{pay}} = -NA \times 0.0001$.

4. Use $DV01 = (1/10,000)(-dP/dy)$ or $DV01 = P D_{\text{mod}} / 10,000$.

5. Bump curve, rebuild discount factors, recompute forwards if single-curve, reprice swap; PV01 is PV difference.

6. Discount PV01 affects discounting of both legs; projection PV01 changes expected floating payments—sign depends on whether swap is payer/receiver and on moneyness (K vs forwards).

7. Hedge ratio: $N_{\text{swap}} \approx DV01_{\text{bond}} / (DV01_{\text{swap per notional}})$, with opposite direction.

8. DV01-neutral under parallel: choose $N_{\text{swap}}$ to match DV01. Apply twist by bumping short and long nodes differently; compute each instrument's PV change and take the difference (residual).

---

## Source Map

### (A) Verified Facts — Specific Sources

| Fact | Source |
|------|--------|
| Swap PV = $V_{\text{fixed}} - V_{\text{float}}$ (receiver perspective) | Tuckman |
| "PV01" in swap markets = change for 1bp change in fixed rate $K$ | Tuckman |
| At initiation both legs worth par | Tuckman |
| DV01 = $(1/10,000)(-dP/dy)$ | Tuckman |
| DV01 hedge ratio formula | Tuckman |
| Combined swap DV01 not often used; floating-leg DV01 depends on time to next reset | Tuckman |
| Project forwards, discount at OIS | Hull |
| Forward rate from discount factors formula | Andersen & Piterbarg |
| Multi-curve: separate discount and forward curves | Hull, Andersen & Piterbarg |
| Pre-crisis single-curve vs post-crisis multi-curve | Andersen & Piterbarg |

### (B) Reasoned Inference — Derivation Logic

| Inference | Derivation Chain |
|-----------|------------------|
| Par swap rate formula | Set $PV_{\text{rec}} = 0$, solve for $K$ |
| Single-curve $K^* = (1 - P_n)/A$ | Forward-from-DF formula + telescoping sum |
| $PV01_K = \pm NA \times 0.0001$ | Differentiate PV w.r.t. $K$ |
| Discount vs projection PV01 attribution | Multi-curve structure + sensitivity definitions |

### (C) Speculation — Flagged Uncertainties

| Item | Uncertainty |
|------|-------------|
| Exact desk curve-bump methodologies | Varies by desk; sources describe multiple approaches |
| Projection-curve bump rule (bump zeros vs forwards directly) | Implementation choice not specified in sources |
| Full empirical decomposition of swap spread moves | Not covered in required source list for this chapter |

---

*Last Updated: January 2026*
