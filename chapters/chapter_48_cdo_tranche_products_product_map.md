# Chapter 48: CDO / Tranche Products — What They Are (Product Map)

---

## Conventions & Notation

| Convention | Description |
|------------|-------------|
| **Attachment/detachment units** | All "%" attachment/detachment and losses are fractions of original portfolio notional $N_{\text{port}}$ unless explicitly stated otherwise |
| **Portfolio loss** | $L(t)$ is cumulative loss from inception to time $t$, expressed as a fraction of $N_{\text{port}}$ (so $0 \le L(t) \le 1$, or possibly $\le L_{\max}$ if recoveries imply a maximum loss less than 100%) |
| **Tranche notation** | Tranche is denoted $[A, D]$ with attachment $A$ and detachment $D$, $0 \le A < D \le 1$ |
| **Dollar conversion** | $x\%$ of $N_{\text{port}}$ means $x/100 \times N_{\text{port}}$ dollars |
| **Premium accrual** | Premium accrual factor $\alpha$ uses the deal's day count; where we need a number, we use a simple quarterly $\alpha = 0.25$ for illustration (not a documentation claim) |

---

## 0. Setup

### Conventions Used in This Chapter

- We model tranche cashflows at the "economic mechanics" level: premium leg paid on outstanding tranche notional, protection leg pays allocated portfolio losses to the tranche, consistent with standard synthetic tranche descriptions in the sources.
- We treat tranche attachment/detachment points as percentages of the original portfolio notional (the sources use $K_1, K_2$ as "percentage loss" strikes).
- When we show a tranche set like $0\text{–}3\%$, $3\text{–}7\%$, $7\text{–}10\%$, $10\text{–}15\%$, $15\text{–}30\%$, we label it as "illustrative/example set" (it appears repeatedly as an example tranche ladder in the sources).

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $N_{\text{port}}$ | Portfolio notional (USD) |
| $L(t)$ | Fractional portfolio loss by time $t$ |
| $A$, $D$ | Tranche attachment and detachment (fractions of $N_{\text{port}}$) |
| $w = D - A$ | Tranche width (fraction of $N_{\text{port}}$) |
| $\text{TL}(L)$ | Tranche loss amount expressed as a fraction of portfolio notional (our "portfolio-scale" tranche loss mapping) |
| $L_{\text{tr}}(t; A, D)$ | Fractional tranche loss as a fraction of tranche notional (ranges in $[0, 1]$); this aligns with the book's $L(t, K_1, K_2)$ |
| $N_{\text{tr}}^{\text{face}} = w \cdot N_{\text{port}}$ | Tranche face notional in dollars |
| $N_{\text{tr}}^{\text{out}}(t)$ | Outstanding tranche notional in dollars |
| $S(A, D)$ | Contractual tranche spread (per annum) used for premium leg |
| $\alpha_i = \Delta(t_{i-1}, t_i)$ | Accrual year fraction for the $i$-th premium period |

---

## 1. Core Concepts (Definitions First)

### 1.1 CDO as a Product Family (Cash vs Synthetic, Funded vs Unfunded)

**Formal Definition:**

A CDO (collateralized debt obligation) is a multi-name credit structure that "tranches" the credit risk of an underlying reference portfolio, allocating portfolio default losses to different tranches in a prescribed order.

- **Synthetic CDOs / tranche swaps:** one party agrees to make payments equal to portfolio losses that fall within a tranche's attachment/detachment range; the other party pays periodic premiums on the insured (remaining) tranche principal.
- **Cash CDOs:** tranches are funded by an initial investment used to finance/hold underlying bonds/loans; cash flows are distributed by a waterfall.

**Intuition:**

A CDO slices one pool of credit risk into layers: the bottom layer takes frequent small losses; upper layers take rare catastrophic losses.

**Trading/Risk/Portfolio Practice:**

Synthetic tranches are traded like insurance on a slice of portfolio losses; they are "correlation products" because their risk depends on whether defaults happen together.

**Funded vs Unfunded (high-level, source-backed as far as possible):**

- Cash CDO investors fund the purchase of underlying bonds/loans (funded).
- Synthetic tranche investors may not fund bond purchases; they agree on premium and loss-sharing, and collateral may be posted.

> **I'm not sure** about the exact collateral mechanics, triggers for collateral movements, and legal funding language for any specific deal. To be certain, you'd need the offering circular / indenture (cash CDO) or ISDA confirmation + credit derivatives definitions + index/tranche rulebook (synthetic/index tranches).

---

### 1.2 Portfolio Loss $L(t)$ as the State Variable (in % and $ Terms)

**Formal Definition:**

$L(t)$ is the fraction of the reference portfolio notional lost due to defaults by time $t$.

A standard portfolio-loss representation weights each name's loss by its exposure and loss-given-default; in the source, the portfolio loss is written as a sum over credits using fractional exposures $F_i$ and recoveries $R_i$, with typical practice $F_i = 1/N_c$ for equal exposures.

**Intuition:**

$L(t)$ is the single "health meter" of the portfolio. Tranches are functions of this meter.

**Trading/Risk/Portfolio Practice:**

Tranche valuation and risk management often reduce to understanding how the contract responds to changes in $L(t)$ (jumps at defaults, slow evolution otherwise).

---

### 1.3 Tranche Attachment $A$ and Detachment $D$ (in % and $ Terms)

**Formal Definition:**

- **Attachment $A$** (source notation $K_1$): the portfolio loss level below which the tranche loss is zero
- **Detachment $D$** ($K_2$): the portfolio loss level at or above which the tranche is fully wiped
- **Tranche width:** $w = D - A$

**Intuition:**

$A$ is the "deductible" (subordination). $D$ is the "limit" of the tranche's loss-bearing capacity.

**Trading/Risk/Portfolio Practice:**

Traders quote tranches by $[A, D]$ and a tranche spread $S(A, D)$ (and sometimes an upfront), with premium paid on surviving tranche notional.

---

### 1.4 Tranche Loss Function: How Portfolio Losses Map into Tranche Losses

**Formal Definition:**

The sources define a fractional tranche loss (as a fraction of tranche notional) as a piecewise-linear function of $L(T)$ between $K_1$ and $K_2$.

**Intuition:**

- Below $A$: no damage
- Between $A$ and $D$: damage accrues linearly
- Above $D$: tranche is wiped

**Trading/Risk/Portfolio Practice:**

This mapping is the "waterfall" for standard tranche CDOs (STCDO), but it's implemented as a payoff function rather than as a literal cashflow waterfall from underlying assets.

---

### 1.5 Subordination / Credit Enhancement: "Who Absorbs First Loss"

**Formal Definition:**

"Tranching" structures losses in a specific order: equity absorbs first losses, then mezzanine, then senior tranches.

**Intuition:**

Subordination is protection for senior tranches: junior layers must be exhausted before seniors lose principal.

**Trading/Risk/Portfolio Practice:**

Because losses are ordered, tranche risk is sensitive to default clustering (default correlation).

---

### 1.6 Premium Leg / Fee Leg Mechanics (High Level)

**Formal Definition:**

Premium leg cashflows are paid by the tranche protection buyer to the tranche protection seller, at contractual spread $S(K_1, K_2)$, on the surviving notional of the tranche, typically on a quarterly schedule; the source explicitly writes (per \$1 face value) the coupon at $t_i$ as:

$$\Delta(t_{i-1}, t_i) \cdot S(K_1, K_2) \cdot (1 - L(t_i, K_1, K_2))$$

**Intuition:**

As the tranche takes losses, less notional remains to pay premium on, so coupons decline.

**Trading/Risk/Portfolio Practice:**

Timing matters: the source notes premium calculations can depend on when defaults occur within the accrual period and draws an analogy between outstanding tranche notional and a CDS survival probability.

---

### 1.7 Protection Leg / Contingent Loss Settlement Mechanics (High Level, Only as Supported)

**Formal Definition:**

In STCDO terms, the protection leg consists of loss payments to cover default losses on the tranche; the size of the loss is the change in the tranche loss function $L(t, K_1, K_2)$, and it changes only when a credit event causes portfolio loss to move into the tranche's region.

**Intuition:**

Protection leg = "you get paid when losses that belong to your tranche happen."

**Trading/Risk/Portfolio Practice:**

The source states that underlying CDS losses are usually settled using the cash settlement process described for CDS.

> **I'm not sure** about the exact settlement timeline and documentation specifics for any given tranche product; to be certain you'd need the ISDA confirmation, plus (for index tranches) the index/tranche rulebook.

---

### 1.8 Standardized Tranches / STCDO: What "Standardized" Means (Only as Supported)

**Formal Definition (as supported):**

O'Kane frames STCDO as a tranche product whose "waterfall" is a payoff function mapping portfolio loss $L(T)$ into tranche loss via attachment/detachment strikes $K_1, K_2$.

The broader credit-derivatives product ecosystem includes CDS indices as building blocks for "products based on these indices," including tranched index trades.

The sources discuss "standard tranche strikes" used as key nodes in market quoting / implied-correlation frameworks, but they do not provide a complete contractual grid here.

**Intuition:**

"Standardized" (in the limited sense supported here) means: tranches are defined on well-known reference portfolios (often indices) and traded widely enough that market quoting conventions (e.g., standard strikes) emerge.

**Trading/Risk/Portfolio Practice:**

Single-tranche trading can occur without physically creating the underlying portfolio; an "imaginary reference portfolio" is used to define tranche cashflows.

> **I'm not sure** about the full details of what "standardized" means in documentation (e.g., exact roll dates, settlement conventions, coupon standards, auction mechanics for tranche losses). To be certain, you'd need the index/tranche rulebook (e.g., Markit index rules) and the relevant ISDA documentation.

---

## 2. Product Map: Instruments and Where Tranches Fit

### 2.1 Taxonomy (with Examples)

**Single-name CDS:**
Bilateral contract transferring credit risk of a single entity; protection buyer pays premium; protection seller compensates loss-from-par on credit event.

**Index CDS:**
One trade referencing a portfolio of many CDS names; an important building block for other products.

**Tranched index trades / index tranches (a.k.a. single-tranche portfolio products):**
Apply tranching to portfolio/index credit risk; losses allocated by attachment/detachment.

**Bespoke tranches / bespoke CDOs:**
Tranches on reference portfolios that are not the "standard" index constituents; still defined by the same $[A, D]$ machinery, but with bespoke portfolio composition.

(We avoid additional claims about bespoke documentation here; details would require deal docs.)

**A "chain" view (conceptual):**

```
Single-name CDS → CDS index (portfolio in one trade) → tranched index (index tranches/STCDO) → bespoke tranches / bespoke CDOs
```

---

### 2.2 Cash CDO vs Synthetic CDO: What the Underlying Reference Portfolio Is

**Cash CDO:**
Underlying = actual bonds/loans; tranches are funded by investors; waterfall allocates cash flows.

**Synthetic CDO / tranche swap:**
Underlying = reference portfolio of debt instruments or CDS; cash flows defined by portfolio losses and tranche range; periodic premium paid on remaining tranche principal.

---

### 2.3 Funded Notes vs Unfunded Swap-Style Protection (Only as Supported)

The sources clearly distinguish: cash CDOs require initial investment; synthetic structures can be arranged without "financing the underlying bonds," though collateral is often involved.

> **I'm not sure** about the exact funded/unfunded legal forms (notes vs swaps) for a specific product. To be certain: offering circular/indenture (notes) or ISDA confirmation (swap).

---

### 2.4 Equity / Mezzanine / Senior Tranches: Economic Meaning

**Equity tranche (lowest attachment):**
First-loss piece: absorbs initial portfolio defaults.

**Mezzanine tranches:**
Middle-loss pieces: take losses after equity is exhausted.

**Senior tranches:**
High-attachment pieces: only lose if portfolio losses are large enough to eat through subordination.

---

### 2.5 "Attachment/Detachment Ladder" Visualization (ASCII)

Below is an illustrative ladder using tranche points that appear as common examples in the sources (not asserted as a universal or current market grid).

```
Portfolio loss L(t)  ↑  (0% at bottom, 100% at top)

100% |----------------------------------------------|
     |              Super-senior (illustrative)      |
 30% |----------------------- D=30% -----------------|
     |                 Senior [15,30]                |
 15% |----------- D=15% -----------------------------|
     |               Senior/Mezz [10,15]             |
 10% |------ D=10% ----------------------------------|
     |                 Mezz [7,10]                   |
  7% |--- D=7% --------------------------------------|
     |                 Mezz [3,7]                    |
  3% |- D=3% ----------------------------------------|
     |                 Equity [0,3]                  |
  0% |----------------------------------------------|
         Losses hit bottom first; move upward as L increases
```

---

## 3. Waterfall Mechanics (The Centerpiece)

This section translates "portfolio loss" into "tranche loss + remaining notional" and then into who pays what.

### 3.1 Define Portfolio Loss $L$ (Cumulative) and Incremental Loss $dL$

**Cumulative loss:**

$L(t) \in [0, 1]$ is cumulative portfolio loss (fraction of original portfolio notional) by time $t$.

**Incremental loss:**

Over a small time interval, $dL(t) = L(t + dt) - L(t)$ is the incremental increase in cumulative portfolio loss. In practice, $L(t)$ jumps at credit events.

---

### 3.2 Define Tranche Loss for Tranche $[A, D]$

**Required tranche-loss mapping (portfolio-scale):**

$$\boxed{\text{TL}(L) = \min\bigl(\max(L - A, 0),\; D - A\bigr)}$$

where $\text{TL}(L)$ is the tranche loss as a fraction of portfolio notional.

**Connection to the source's normalized tranche-loss function:**

The source defines a fractional loss of the tranche (as a fraction of tranche notional) $L(T, K_1, K_2)$ as:

$$L(T, K_1, K_2) = \frac{\max(L(T) - K_1, 0) - \max(L(T) - K_2, 0)}{K_2 - K_1}$$

In our notation, $K_1 = A$, $K_2 = D$, and

$$L_{\text{tr}}(T; A, D) = \frac{\text{TL}(L(T))}{D - A}$$

So: portfolio-scale tranche loss $\text{TL}$ is the same object as the source's numerator, while $L_{\text{tr}}$ is that loss divided by tranche width.

---

### 3.3 Define Tranche Remaining Notional

**In % of portfolio notional:**

$$N_{\text{tr}}^{\%}(t) = (D - A) - \text{TL}(L(t)) \ge 0$$

**In dollars:**

$$\boxed{N_{\text{tr}}^{\text{out}}(t) = \bigl[(D - A) - \text{TL}(L(t))\bigr] \cdot N_{\text{port}}}$$

**Normalized surviving fraction of tranche notional:**

$$\text{SurvFrac}_{\text{tr}}(t) = \frac{N_{\text{tr}}^{\%}(t)}{D - A} = 1 - L_{\text{tr}}(t; A, D)$$

consistent with the source's "premium paid on the surviving notional" $1 - L(t_i, K_1, K_2)$.

---

### 3.4 Explain Who Pays/Receives What

**Protection seller vs buyer (tranche):**
Protection buyer pays the premium leg; protection seller pays losses in the tranche range, as they occur.

**Premium leg:**
Premium payments are on outstanding tranche notional (surviving notional).

**Protection leg:**
Protection leg equals changes in tranche loss function and is triggered by credit events that move portfolio loss into the tranche region.

---

### 3.5 Intuition: Equity Absorbs First; Senior Protected Until Subordination Is Gone

The source description of tranching is explicit: first losses hit equity, then mezzanine, then senior.

This ordering is exactly encoded by $\text{TL}(L)$: low-attachment tranches "turn on" first; high-attachment tranches remain untouched until $L$ crosses their attachment.

---

## 4. Math and Derivations (Step-by-Step)

### 4.1 Tranche Loss Mapping $\text{TL}(L)$ and Tranche Notional Remaining

**Assumptions:**

- $L \in [0, 1]$ is cumulative portfolio loss (fraction of original portfolio notional)
- Tranche is $[A, D]$ with width $w = D - A > 0$

**Derivation (piecewise):**

1. **If $L \le A$:** portfolio loss has not reached attachment → tranche loss is zero:
   $$\text{TL}(L) = 0$$

2. **If $A < L < D$:** tranche absorbs the amount of portfolio loss beyond attachment:
   $$\text{TL}(L) = L - A$$

3. **If $L \ge D$:** tranche absorbs its full width:
   $$\text{TL}(L) = D - A = w$$

**Compact form:**

$$\boxed{\text{TL}(L) = \min\bigl(\max(L - A, 0),\; D - A\bigr)}$$

**Equivalent "max–max" form (useful for algebra):**

$$\boxed{\text{TL}(L) = \max(L - A, 0) - \max(L - D, 0)}$$

This matches the numerator form in the source's normalized tranche loss formula (which divides by $D - A$).

**Outstanding notional (portfolio fraction and dollars):**

$$N_{\text{tr}}^{\%}(L) = w - \text{TL}(L) \quad \Rightarrow \quad N_{\text{tr}}^{\text{out}}(L) = \bigl(w - \text{TL}(L)\bigr) \cdot N_{\text{port}}$$

**Unit check:**

$L, A, D, w, \text{TL}(L)$ are unitless fractions. Multiplying by $N_{\text{port}}$ gives dollars.

---

### 4.2 Tranche "Payment Base" for Premium: Outstanding Tranche Notional

The source states that the coupon is paid on "surviving notional," and provides (per \$1 face value):

$$\Delta(t_{i-1}, t_i) \cdot S(A, D) \cdot (1 - L_{\text{tr}}(t_i; A, D))$$

To express this in dollars:

- **Tranche face notional:** $N_{\text{tr}}^{\text{face}} = w \cdot N_{\text{port}}$
- **Surviving fraction of tranche notional:** $1 - L_{\text{tr}}(t_i; A, D) = \dfrac{N_{\text{tr}}^{\%}(t_i)}{w}$

So premium payment at $t_i$ is:

$$\boxed{\text{PremPay}_i = \alpha_i \cdot S(A, D) \cdot N_{\text{tr}}^{\text{out}}(t_i)}$$

where $S$ is in decimal per year and $\alpha_i$ is the accrual year fraction.

**Unit check:**

$S$ is "per year", $\alpha_i$ is years, $N_{\text{tr}}^{\text{out}}$ is dollars → payment is dollars.

---

### 4.3 Sanity Checks (Required)

Let $\text{TL}(L) = \min(\max(L - A, 0), w)$, $w = D - A$.

1. If $L \le A$, then $L - A \le 0 \Rightarrow \max(L - A, 0) = 0 \Rightarrow \text{TL}(L) = 0$

2. If $L \ge D$, then $L - A \ge w \Rightarrow \max(L - A, 0) \ge w \Rightarrow \text{TL}(L) = w$ (fully wiped)

3. **Monotonicity:** $\max(L - A, 0)$ is non-decreasing in $L$; $\min(\cdot, w)$ preserves non-decreasingness → $\text{TL}(L)$ is non-decreasing

4. **Bounds:** $0 \le \text{TL}(L) \le w$ by construction, so remaining notional $w - \text{TL}(L) \ge 0$

---

## 5. Measurement & Risk (Only What Belongs in Chapter 48)

This is a product map, so we focus on what risks exist and why.

### 5.1 Key Risk Drivers (Conceptual)

**Spread / premium (mark-to-market) risk:**
Tranches have premium legs like CDS: value depends on discounting and expected future premium payments on surviving notional.

**Default / jump risk:**
Tranche loss changes only when credit events occur in the reference portfolio; protection leg is tied to changes in the tranche loss function.

**Correlation / dependence risk:**
Tranching products are "correlation products": tranche risk depends on the tendency of credits to default together. Empirically in the source's modeling discussions, tranche spreads change with assumed asset correlation (equity can move opposite to mezz/senior across correlation regimes).

**Recovery / loss-given-default risk:**
Portfolio loss depends on recoveries; in the source, portfolio loss is built from $(1 - R_i)$ terms.

**Liquidity and model risk (high-level):**
Tranche valuation relies on modeling the portfolio loss distribution and its dynamics; different modeling/quoting conventions (e.g., implied correlation surfaces) can introduce arbitrage issues and instability.

> **I'm not sure** how to quantify liquidity premia or model risk for your specific tranche without a stated model framework and market data; you'd need the desk model documentation, calibration methodology, and observed bid/offer quotes.

---

### 5.2 Preview: Expected Tranche Loss (ETL) as a Core Object (No Calibration Yet)

The source defines ETL $\psi(T, K)$ as the expected loss at time $T$ of an equity tranche of width $K$, with

$$\psi(T, K) = \mathbb{E}\bigl[\min(L(T), K)\bigr]$$

Later chapters will use ETL (and related objects) to price premium/protection legs, but here we only flag it as the key "loss distribution summary statistic."

---

## 6. Worked Examples (At Least 14 Numeric Examples)

### Common Setup for Examples 1–14

- **Portfolio notional:** $N_{\text{port}} = \$100{,}000{,}000$ (= \$100mm)
- **Example tranche ladder** (seen in the sources as common tranche examples):
  - $[0, 3\%]$, $[3, 7\%]$, $[7, 10\%]$, $[10, 15\%]$, $[15, 30\%]$
  - We also sometimes add a residual $[30, 100\%]$ tranche only to complete a 0–100% partition for "loss conservation" demonstrations (not a claim that it is quoted/standard)
- **Tranche loss (portfolio-scale):** $\text{TL}(L) = \min(\max(L - A, 0), D - A)$
- **Dollar loss:** $\$\text{TL}(L) = \text{TL}(L) \times N_{\text{port}}$
- **Remaining tranche notional (portfolio-scale):** $w - \text{TL}(L)$, dollars $(w - \text{TL}(L)) \cdot N_{\text{port}}$

---

### Example 1: Convert $A/D$ from % to \$ for a \$100mm Portfolio

**Compute dollar attachment/detachment:**

| Tranche | Attachment $A$ | Detachment $D$ |
|---------|----------------|----------------|
| $[0, 3\%]$ | \$0 | $0.03 \times 100\text{mm} = \$3\text{mm}$ |
| $[3, 7\%]$ | $0.03 \times 100\text{mm} = \$3\text{mm}$ | $0.07 \times 100\text{mm} = \$7\text{mm}$ |
| $[7, 10\%]$ | \$7mm | \$10mm |
| $[10, 15\%]$ | \$10mm | \$15mm |
| $[15, 30\%]$ | \$15mm | \$30mm |

**Unit check:** "% of portfolio notional" × "\$100mm" = dollars.

---

### Example 2: Given $L = 1.5\%$, Compute TL and Remaining Notional for Equity Tranche $[0, 3\%]$

**Inputs:**
- $L = 0.015$, $A = 0$, $D = 0.03$, $w = 0.03$

**Tranche loss (portfolio-scale):**

$$\text{TL} = \min(\max(0.015 - 0, 0), 0.03) = 0.015$$

**Dollar tranche loss:** $\$\text{TL} = 0.015 \times 100\text{mm} = \$1.5\text{mm}$

**Remaining notional (portfolio-scale):**

$$w - \text{TL} = 0.03 - 0.015 = 0.015$$

**Dollar remaining:** $0.015 \times 100\text{mm} = \$1.5\text{mm}$

---

### Example 3: Same $L$, Compute TL for Mezz Tranche $[3, 7\%]$ (Should Be 0) and Explain

**Inputs:**
- $L = 0.015$, $A = 0.03$, $D = 0.07$, $w = 0.04$

$$\text{TL} = \min(\max(0.015 - 0.03, 0), 0.04) = \min(0, 0.04) = 0$$

**Dollar tranche loss:** \$0

**Explanation:** Portfolio losses haven't reached the tranche's attachment point $A = 3\%$, so this tranche is still protected by subordination.

**Remaining notional:** $w = 0.04 \Rightarrow \$4\text{mm}$

---

### Example 4: Given $L = 5\%$, Compute TL for $[0, 3\%]$ and $[3, 7\%]$ and Show Waterfall

**Portfolio loss:** $L = 0.05 \Rightarrow \$5\text{mm}$

**Equity $[0, 3\%]$:**

$$\text{TL}_{0\text{-}3} = \min(\max(0.05 - 0, 0), 0.03) = 0.03 \Rightarrow \$3\text{mm}$$

Equity is fully wiped (remaining \$0).

**Mezz $[3, 7\%]$:**

$$\text{TL}_{3\text{-}7} = \min(\max(0.05 - 0.03, 0), 0.04) = \min(0.02, 0.04) = 0.02 \Rightarrow \$2\text{mm}$$

Remaining mezz notional: $0.04 - 0.02 = 0.02 \Rightarrow \$2\text{mm}$

**Waterfall check (loss conservation over these two tranches):**
- Total portfolio loss = \$5mm
- Loss allocated: \$3mm (equity) + \$2mm (mezz) = \$5mm ✓

---

### Example 5: Given $L = 12\%$, Compute TL for Multiple Tranches and Show Which Are Wiped

**Portfolio loss:** $L = 0.12 \Rightarrow \$12\text{mm}$

| Tranche | Width $w$ | Tranche Loss TL | Dollar Loss | Status |
|---------|-----------|-----------------|-------------|--------|
| $[0, 3\%]$ | 0.03 | $0.03 \Rightarrow \$3\text{mm}$ | \$3mm | Wiped |
| $[3, 7\%]$ | 0.04 | $\min(\max(0.12 - 0.03, 0), 0.04) = 0.04 \Rightarrow \$4\text{mm}$ | \$4mm | Wiped |
| $[7, 10\%]$ | 0.03 | $\min(\max(0.12 - 0.07, 0), 0.03) = 0.03 \Rightarrow \$3\text{mm}$ | \$3mm | Wiped |
| $[10, 15\%]$ | 0.05 | $\min(\max(0.12 - 0.10, 0), 0.05) = 0.02 \Rightarrow \$2\text{mm}$ | \$2mm | Remaining \$3mm |
| $[15, 30\%]$ | 0.15 | $\min(\max(0.12 - 0.15, 0), 0.15) = 0 \Rightarrow \$0$ | \$0 | Remaining \$15mm |

**Interpretation:** Loss has eaten through equity + two mezz tranches entirely, and is partway through $[10, 15]$.

---

### Example 6: Incremental Loss Allocation: $L$ Goes from $4\% \to 6\%$

Let $L_1 = 0.04$ and $L_2 = 0.06$. Incremental portfolio loss $\Delta L = 0.02 \Rightarrow \$2\text{mm}$.

**Compute tranche losses at each level:**

**At $L_1 = 4\%$:**
- Equity $[0, 3]$: $\text{TL}_{0\text{-}3} = 0.03 \Rightarrow \$3\text{mm}$ (wiped)
- Mezz $[3, 7]$: $\text{TL}_{3\text{-}7} = \min(0.04 - 0.03, 0.04) = 0.01 \Rightarrow \$1\text{mm}$
- Remaining mezz: $0.04 - 0.01 = 0.03 \Rightarrow \$3\text{mm}$

**At $L_2 = 6\%$:**
- Equity $[0, 3]$: still $\text{TL}_{0\text{-}3} = 0.03 \Rightarrow \$3\text{mm}$
- Mezz $[3, 7]$: $\text{TL}_{3\text{-}7} = \min(0.06 - 0.03, 0.04) = 0.03 \Rightarrow \$3\text{mm}$
- Remaining mezz: $0.04 - 0.03 = 0.01 \Rightarrow \$1\text{mm}$

**Incremental tranche loss from 4% to 6%:**
- Equity: $\Delta \text{TL}_{0\text{-}3} = 0$
- Mezz: $\Delta \text{TL}_{3\text{-}7} = 0.03 - 0.01 = 0.02 \Rightarrow \$2\text{mm}$

All incremental losses went to the mezzanine tranche, consistent with subordination ordering.

---

### Example 7: Premium Base Mechanics: Compute Next Premium Payment After Losses

Use tranche $[3, 7\%]$ after portfolio loss $L = 5\%$ (from Example 4).

- **Remaining tranche notional** = \$2mm
- **Contractual tranche spread** $S = 250$ bp = $0.025$ per year
- **Accrual** $\alpha = 0.25$ (quarter)

**Premium payment:**

$$\text{PremPay} = \alpha \cdot S \cdot N_{\text{tr}}^{\text{out}} = 0.25 \times 0.025 \times 2{,}000{,}000 = \$12{,}500$$

**Unit check:** $0.25$ years × (per year) × dollars = dollars.

(Conceptually consistent with "coupon paid on surviving notional" in the source.)

---

### Example 8: Demonstrate "Subordination": Senior Losses Start Only After Juniors Exhausted

Consider tranche $[10, 15\%]$ (width 5% = \$5mm).

- **If $L = 9\%$:** $L < A \Rightarrow \text{TL} = 0$. Senior untouched.
- **If $L = 12\%$:** $\text{TL} = \min(0.12 - 0.10, 0.05) = 0.02 \Rightarrow \$2\text{mm}$
  - Remaining senior: \$3mm

**Interpretation:** Senior tranche only begins losing after cumulative portfolio loss exceeds 10% (its attachment). This is the precise meaning of subordination $A$.

---

### Example 9: Same Width, Different Attachment: Compare Loss Exposure for the Same $L$

Compare two 3%-wide tranches: $[0, 3\%]$ and $[7, 10\%]$ (both width $w = 0.03$).

**Case A: $L = 5\%$ ($0.05$)**

| Tranche | Calculation | Dollar Loss |
|---------|-------------|-------------|
| $[0, 3]$ | $\text{TL} = \min(0.05, 0.03) = 0.03$ | \$3mm (wiped) |
| $[7, 10]$ | $\text{TL} = \min(\max(0.05 - 0.07, 0), 0.03) = 0$ | \$0 |

**Case B: $L = 8\%$ ($0.08$)**

| Tranche | Calculation | Dollar Loss |
|---------|-------------|-------------|
| $[0, 3]$ | still $\text{TL} = 0.03$ | \$3mm |
| $[7, 10]$ | $\text{TL} = \min(0.08 - 0.07, 0.03) = 0.01$ | \$1mm |

**Takeaway:** Higher attachment shifts loss exposure to more extreme portfolio-loss scenarios.

---

### Example 10: "Equity vs Senior" Payoff Intuition: Tabulate $\text{TL}(L)$ Across $L$

Compare:
- **Equity:** $[0, 3]$, width $0.03$
- **Senior:** $[10, 15]$, width $0.05$

All numbers below are portfolio-scale tranche loss $\text{TL}(L)$:

| Portfolio loss $L$ | $\text{TL}_{0\text{-}3}(L)$ | $\text{TL}_{10\text{-}15}(L)$ |
|--------------------|-----------------------------|-----------------------------|
| 0% | 0% | 0% |
| 2% | 2% | 0% |
| 5% | 3% | 0% |
| 10% | 3% | 0% |
| 20% | 3% | 5% |

**Convert last row to dollars (for intuition):**

At $L = 20\%$: equity loss = $3\% \times \$100\text{mm} = \$3\text{mm}$; senior loss = $5\% \times \$100\text{mm} = \$5\text{mm}$.

---

### Example 11: Recovery Affects Portfolio Loss (Toy): Defaults with Recovery $R$

Assume a simple equal-weight portfolio: 10 names, each \$10mm (total \$100mm).
Assume 2 names default with recovery $R = 40\%$.

**Loss per defaulted name:**

$$\$10\text{mm} \times (1 - R) = \$10\text{mm} \times 0.6 = \$6\text{mm}$$

**Total portfolio loss:** $2 \times \$6\text{mm} = \$12\text{mm} \Rightarrow L = 12\%$

**Now allocate tranche losses using Example 5 results (since $L = 12\%$):**

| Tranche | Dollar Loss |
|---------|-------------|
| $[0, 3]$ | \$3mm (wiped) |
| $[3, 7]$ | \$4mm (wiped) |
| $[7, 10]$ | \$3mm (wiped) |
| $[10, 15]$ | \$2mm (remaining \$3mm) |
| $[15, 30]$ | \$0mm |

**Source connection:** Portfolio loss depends on loss-given-default $(1 - R_i)$ terms; equal exposures $F_i = 1/N_c$ are typical in synthetic CDO portfolios.

---

### Example 12: Show Tranche Notional in \$ Terms After Losses (At Least 2 Tranches)

Using $L = 12\%$ again:

| Tranche | Width | Loss | Outstanding |
|---------|-------|------|-------------|
| $[10, 15]$ | \$5mm | \$2mm | \$5mm − \$2mm = **\$3mm** |
| $[15, 30]$ | \$15mm | \$0 | **\$15mm** |
| $[3, 7]$ | \$4mm | \$4mm | **\$0** (wiped) |

---

### Example 13: Attachment/Detachment Ladder Diagram with Example Numbers and Loss Absorption Order

Using the same ladder and $N_{\text{port}} = \$100\text{mm}$:

```
Top (most protected)
  [30,100]  width 70% = $70mm  (residual, illustrative)
  [15,30]   width 15% = $15mm  (senior)
  [10,15]   width  5% = $ 5mm  (senior/mezz)
  [ 7,10]   width  3% = $ 3mm  (mezz)
  [ 3, 7]   width  4% = $ 4mm  (mezz)
  [ 0, 3]   width  3% = $ 3mm  (equity, first loss)
Bottom (first loss)
```

**Loss absorption order (cumulative):**

1. First \$3mm of portfolio loss hits $[0, 3]$
2. Next \$4mm hits $[3, 7]$
3. Next \$3mm hits $[7, 10]$
4. Next \$5mm hits $[10, 15]$
5. Next \$15mm hits $[15, 30]$
6. Beyond that goes to $[30, 100]$ (if defined)

This ordering is exactly the "equity → mezzanine → senior" structure described in the tranching definition.

---

### Example 14: Mini "Deal Term Sheet" Toy (Non-Legal) + One Cashflow + One Loss Scenario

**Toy economic term sheet (minimum needed to define a tranche trade):**

| Term | Description |
|------|-------------|
| Reference portfolio | Specify the underlying (e.g., "CDS index X" or bespoke list) |
| Tranche | Attachment $A$, detachment $D$, width $w = D - A$ |
| Tranche notional | $N_{\text{tr}}^{\text{face}} = w \cdot N_{\text{port}}$ |
| Contractual tranche spread | $S(A, D)$ (and whether any upfront exists) |
| Maturity | $T$ and premium payment schedule $(t_i)$ |
| Loss settlement convention | Cash vs physical is defined for CDS; tranche-loss settlement follows the tranche loss evolution and is typically tied to CDS cash settlement in the source |

> **I'm not sure** about precise tranche settlement mechanics and timing without the ISDA confirmation / tranche rulebook.

**Compute one premium cashflow (no losses yet):**

Choose tranche $[3, 7]$: $w = 0.04 \Rightarrow N_{\text{tr}}^{\text{face}} = 0.04 \times \$100\text{mm} = \$4\text{mm}$

- Spread $S = 300$ bp = 0.03, accrual $\alpha = 0.25$
- If no losses yet, outstanding = \$4mm

$$\text{PremPay} = 0.25 \times 0.03 \times 4{,}000{,}000 = \$30{,}000$$

**Compute one loss scenario (portfolio loss reaches 5%):**

Suppose cumulative portfolio loss becomes $L = 5\%$ by a credit event sequence.

For tranche $[3, 7]$: $\text{TL} = \min(0.05 - 0.03, 0.04) = 0.02 \Rightarrow \$2\text{mm}$

**Outstanding tranche notional becomes** \$4mm − \$2mm = **\$2mm**

**Next premium payment** (same $S, \alpha$) is now:

$$0.25 \times 0.03 \times 2{,}000{,}000 = \$15{,}000$$

**Interpretation:** Protection buyer has received \$2mm of loss protection and hence pays future coupons on the reduced outstanding tranche notional, consistent with the source's "premium on surviving notional" principle.

---

## 7. Practical Notes

### 7.1 Practitioner Checklist (What You Must Know to Book/Understand a Tranche Product)

- [ ] Reference portfolio definition (index series or bespoke list; weights/exposures)
- [ ] Loss definition for the portfolio: how defaults and recoveries map into $L(t)$
- [ ] Attachment $A$ and detachment $D$ (percent of original portfolio notional)
- [ ] Tranche notional $w \cdot N_{\text{port}}$ and whether the trade notionals are "per \$1 face" or in dollars (the source often uses "per \$1 of face value")
- [ ] Premium terms: tranche spread $S(A, D)$, payment dates, accrual convention (source mentions quarterly and Act/360 as typical in one context)
- [ ] Loss settlement: how tranche losses are settled and when (source ties it to CDS cash settlement practice)
- [ ] Position direction: are you buying protection (pay premium, receive losses) or selling protection (receive premium, pay losses)?

---

### 7.2 Common Pitfalls

1. **Confusing portfolio notional $N_{\text{port}}$ with tranche notional $w \cdot N_{\text{port}}$**

2. **Mixing % and \$ units** (always convert explicitly)

3. **Misreading attachment/detachment:** they are percentages of the reference portfolio notional in the source's formulation

4. **Forgetting premium is on outstanding tranche notional** (surviving notional)

5. **Assuming "standard tranche grids" without sourcing:**
   - The sources refer to "standard tranche strikes" but do not provide a complete universal grid here
   - If you need exact market-standard strikes and documentation, consult the index/tranche rulebook (and ISDA confirmations)

---

### 7.3 Verification Tests (Quick Consistency Checks)

- **Bounds:** $0 \le \text{TL}(L) \le w$ for all $L$
- **Monotonicity:** if $L_2 \ge L_1$, then $\text{TL}(L_2) \ge \text{TL}(L_1)$
- **Outstanding notional non-negative:** $w - \text{TL}(L) \ge 0$

**Loss conservation across a full capital structure (when tranches span 0–100% with no gaps):**

If $[K_{m-1}, K_m]$ are contiguous and cover $[0, 1]$, the source shows that summing expected losses across tranches reproduces expected portfolio loss (a "conservation of expected loss" relationship).

In deterministic scenario terms, if tranches are contiguous and cover the whole interval with no gaps, then the sum of tranche losses (portfolio-scale) equals the portfolio loss, i.e., $\sum_m \text{TL}_m(L) = L$.

If gaps exist (e.g., you only trade some tranches), then $\sum \text{TL}_m(L) \le L$, with the missing interval absorbing the remainder.

---

## 8. Summary & Recall

### 8.1 10-Bullet Executive Summary

1. A **CDO** is a structure that tranches portfolio credit risk so losses are incurred in order: equity → mezzanine → senior.

2. **Synthetic tranches** are defined by payments on losses in a specified portfolio-loss range $[\alpha_L, \alpha_H]$ plus premiums on remaining insured principal.

3. **Portfolio loss $L(t)$** (fraction of portfolio notional lost by time $t$) is the key state variable.

4. Tranche **attachment $A$** and **detachment $D$** define when losses begin and when the tranche is wiped.

5. **Portfolio-scale tranche loss mapping:** $\text{TL}(L) = \min(\max(L - A, 0), D - A)$

6. **Outstanding tranche notional** equals tranche width minus tranche loss: $w - \text{TL}(L)$

7. **Premium leg:** protection buyer pays spread on surviving notional (per \$1 face value: $\alpha S(1 - L_{\text{tr}})$)

8. **Protection leg:** loss payments track changes in tranche loss function, triggered by credit events

9. Tranches are **"correlation products"** because default clustering changes how much of the loss distribution falls into each tranche

10. **ETL** $\psi(T, K) = \mathbb{E}[\min(L(T), K)]$ is a foundational object for later pricing chapters

---

### 8.2 Cheat Sheet: Key Definitions + Tranche Loss Formula + Unit Conversions

| Item | Formula/Definition |
|------|-------------------|
| Portfolio notional | $N_{\text{port}}$ dollars |
| Portfolio loss fraction | $L(t) \in [0, 1]$ |
| Tranche | $[A, D]$, width $w = D - A$ |
| **Tranche loss (portfolio-scale)** | $\text{TL}(L) = \min(\max(L - A, 0), w)$ |
| Tranche loss in dollars | $\$\text{TL}(L) = \text{TL}(L) \cdot N_{\text{port}}$ |
| **Outstanding tranche notional in dollars** | $N_{\text{tr}}^{\text{out}}(t) = (w - \text{TL}(L(t))) \cdot N_{\text{port}}$ |
| **Premium payment (simple)** | $\text{PremPay}_i = \alpha_i \cdot S(A, D) \cdot N_{\text{tr}}^{\text{out}}(t_i)$ |
| Normalized tranche loss (source-aligned) | $L_{\text{tr}}(T; A, D) = \dfrac{\max(L(T) - A, 0) - \max(L(T) - D, 0)}{D - A}$ |

---

### 8.3 Flashcards (30 Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What does $L(t)$ represent? | Fraction of portfolio notional lost due to defaults by time $t$ |
| 2 | Define attachment $A$ | Loss level below which tranche loss is zero |
| 3 | Define detachment $D$ | Loss level at/above which tranche is fully wiped |
| 4 | What is tranche width $w$? | $w = D - A$ |
| 5 | What is portfolio-scale tranche loss $\text{TL}(L)$? | $\min(\max(L - A, 0), D - A)$ |
| 6 | What is $N_{\text{tr}}^{\text{out}}(t)$? | Outstanding tranche notional after losses |
| 7 | Who pays the premium leg in a tranche protection trade? | Tranche protection buyer pays tranche protection seller |
| 8 | On what notional are tranche premiums paid? | Surviving/outstanding tranche notional |
| 9 | What triggers protection-leg payments? | Credit events that change the tranche loss function |
| 10 | Why are tranches called correlation products? | Risk depends on tendency of credits to default together |
| 11 | What is "subordination" in tranche language? | The attachment point $A$, the amount of portfolio loss that must occur before tranche is hit |
| 12 | What happens to equity tranche when losses exceed its detachment? | It is wiped (loss equals its full width) |
| 13 | How do you convert 3% attachment on \$100mm to dollars? | \$3mm |
| 14 | If $L = 1.5\%$ and tranche is $[0, 3]$, what is TL? | 1.5% of portfolio = \$1.5mm |
| 15 | If $L < A$, what is TL? | 0 (no tranche loss) |
| 16 | If $L \ge D$, what is TL? | $D - A$ (full wipe) |
| 17 | What is normalized tranche loss $L_{\text{tr}}(T; A, D)$? | Tranche loss as a fraction of tranche notional (0 to 1) |
| 18 | Relation between $\text{TL}(L)$ and $L_{\text{tr}}$? | $L_{\text{tr}} = \text{TL}/(D - A)$ |
| 19 | What is ETL $\psi(T, K)$? | $\mathbb{E}[\min(L(T), K)]$ for an equity tranche of width $K$ |
| 20 | Why do senior tranches have low default frequency but high tail exposure? | They attach high, so only large portfolio losses hit them |
| 21 | What is the economic meaning of "premium on outstanding notional"? | As losses erode protection, premium base shrinks |
| 22 | What does "single-tranche trading" mean (high level)? | Trading one tranche referencing an imaginary portfolio without building it |
| 23 | Difference between cash and synthetic CDO (high level)? | Cash holds bonds/loans funded by investors; synthetic references losses via CDS-style payoff |
| 24 | What's a practical pitfall in tranche calculations? | Mixing portfolio % units with tranche % units |
| 25 | How do incremental losses allocate across tranches? | The tranche whose attachment has been reached but not yet detached absorbs the increment |
| 26 | What is a "wiped-out" tranche? | One where cumulative tranche loss equals its full width $D - A$ |
| 27 | What is "loss conservation" across a full capital structure? | Sum of losses across contiguous tranches equals portfolio loss (when covering 0–100%) |
| 28 | What information is needed to book a tranche? | Portfolio, $A/D$, notional, spread, maturity, settlement conventions |
| 29 | What does "standard tranche strikes" refer to? | Market-quoted strike nodes used in implied-correlation frameworks |
| 30 | What must you consult for exact documentation conventions? | ISDA confirmation + index/tranche rulebook / offering docs (deal-specific) |

---

## 9. Mini Problem Set (18 Questions)

### Questions

1. For $N_{\text{port}} = \$100$mm, convert tranche $[7, 10]$ to dollar attachment/detachment and face notional.

2. With $L = 2\%$, compute TL and outstanding notional for $[0, 3]$.

3. With $L = 2\%$, compute TL for $[3, 7]$. Explain why.

4. With $L = 9\%$, compute tranche losses for $[0, 3]$, $[3, 7]$, and $[7, 10]$.

5. With $L = 9\% \to 11\%$, compute incremental tranche losses for $[7, 10]$ and $[10, 15]$.

6. For tranche $[3, 7]$ with $S = 400$ bp and quarterly accrual, compute premium payment when outstanding notional is \$1.5mm.

7. Show that $\text{TL}(L) = \max(L - A, 0) - \max(L - D, 0)$ equals $\min(\max(L - A, 0), D - A)$.

8. In words, explain why tranche values depend on default correlation.

9. Given 5 equal-weight defaults with recovery 40% in a \$100mm portfolio of 50 equal-weight names, compute $L$.

10. For tranches $[0, 3]$ and $[7, 10]$, plot (by hand) $\text{TL}(L)$ versus $L \in [0, 15\%]$.

11. If recoveries are random and negatively related to default rates, what qualitative effect might that have on senior-tranche risk? (No derivation.)

12. Explain (high level) why premium leg and protection leg have an analogy to CDS pricing objects (survival curves).

13. Define ETL and explain how it relates to tranche expected loss.

14. Suppose a tranche has width 4% and has incurred 1.2% portfolio-scale loss. What fraction of tranche notional remains?

15. Describe a "loss conservation" test for a full capital structure and explain what breaks if there are gaps.

16. Name two documentation items you would request before trading a bespoke tranche and why.

17. For a protection seller, what happens to future premium receipts after tranche losses occur?

18. List three sources of model risk in tranche pricing/risk management (high level).

---

### Brief Solution Sketches (1–9 Only)

**1.** $[7, 10]$: $A = 0.07 \times 100 = \$7$mm, $D = \$10$mm, face $w = 3\% \Rightarrow \$3$mm.

**2.** $[0, 3]$, $L = 2\%$: $\text{TL} = 2\% \Rightarrow \$2$mm; remaining $= 1\% \Rightarrow \$1$mm.

**3.** $[3, 7]$: $L < A \Rightarrow \text{TL} = 0$. Subordination protects.

**4.** $L = 9\%$: $[0, 3]$ loss $3\%$, $[3, 7]$ loss $4\%$, $[7, 10]$ loss $2\%$.

**5.** $9\% \to 11\%$: at 9% $[7, 10]$ has loss 2%; at 10% it's fully 3%; incremental 1% goes to $[7, 10]$. The remaining 1% (from 10% → 11%) goes to $[10, 15]$.

**6.** Premium = $\alpha S N_{\text{out}}$. With $S = 0.04$, $\alpha = 0.25$, $N_{\text{out}} = 1.5$mm → $0.25 \times 0.04 \times 1.5$mm = **\$15,000**.

**7.** Show piecewise equivalence: for $L \le A$ both are 0; for $A < L < D$ both are $L - A$; for $L \ge D$ both are $D - A$.

**8.** Defaults "together" shift probability mass to either very low loss or very high loss, changing which tranches are likely to be hit.

**9.** Each name weight $= 1/50$ of \$100mm = \$2mm. Loss per default = \$2mm × 0.6 = \$1.2mm. Five defaults → \$6mm → $L = 6\%$.

---

## Source Map

### (A) Verified Facts — Source Citations

| Fact | Source |
|------|--------|
| CDO definition and tranching mechanics | O'Kane Ch 11-12 |
| Tranche loss function $L(T, K_1, K_2)$ formula | O'Kane Ch 12 |
| Premium leg paid on surviving notional | O'Kane Ch 12 |
| Portfolio loss as sum over credits with exposures and recoveries | O'Kane Ch 11, 13 |
| ETL definition $\psi(T, K) = \mathbb{E}[\min(L(T), K)]$ | O'Kane Ch 12-13 |
| Cash CDO vs synthetic CDO distinction | O'Kane Ch 11 |
| Standard tranche strikes in implied correlation context | O'Kane Ch 14 |

### (B) Reasoned Inference — Derivation Logic

| Inference | Derivation |
|-----------|------------|
| Portfolio-scale tranche loss $\text{TL}(L) = \min(\max(L-A,0), D-A)$ | Algebraic equivalence to source's normalized formula times width |
| Dollar conversions | Direct multiplication by $N_{\text{port}}$ |
| Premium payment formula in dollars | Source's per-\$1-face formula scaled by face notional |
| Loss conservation across contiguous tranches | Sum of piecewise-linear functions covering $[0,1]$ |

### (C) Speculation — Flagged Uncertainties

| Topic | Uncertainty Note |
|-------|------------------|
| Exact collateral mechanics for synthetic tranches | Deal-specific; requires ISDA confirmation |
| Funded vs unfunded legal forms | Requires offering circular / indenture |
| Settlement timeline and documentation | Requires tranche rulebook / ISDA confirmation |
| "Standardized" tranche grid specifics | Not provided in sources; requires index rulebook |
| Liquidity premia and model risk quantification | Requires desk model documentation and market data |
