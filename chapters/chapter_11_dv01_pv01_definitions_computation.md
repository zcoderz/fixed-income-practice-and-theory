# Chapter 11: DV01/PV01 — Definitions, Computation, and "What's Being Bumped?"

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- DV01 is the dollar change in value for a basis point change in interest rates (Tuckman Ch 5-6)
- Tuckman's DV01 definition: $\text{DV01} \equiv -\frac{\Delta P}{10000\,\Delta y}$, making DV01 positive for securities whose prices rise when yields fall (Tuckman)
- Hull glossary describes DV01 as the dollar value of a 1bp increase in all rates — opposite sign convention (Hull Ch 4 glossary)
- Yield to maturity $y$ is the IRR that discounts all promised cashflows to the full price (Tuckman Ch 3)
- The price–yield function for a coupon bond: $P(y) = \sum_i \frac{\text{CF}_i}{(1+y)^{t_i}}$ (Tuckman Ch 3-4)
- Discount factors $P(0,T)$ are the prices of default-free zero-coupon bonds (Andersen & Piterbarg Vol 1)
- The swap annuity factor: $A_{k,m}(t) = \sum_{n=k}^{k+m-1} P(t,T_{n+1})\,\tau_n$ (Andersen & Piterbarg Vol 1)
- The forward swap rate: $S_{k,m}(t) = \frac{P(t,T_k) - P(t,T_{k+m})}{A_{k,m}(t)}$ (Andersen & Piterbarg Vol 1)
- Key rate shifts are perturbations where one "key rate" is shifted while the curve is adjusted in between key rates (Tuckman Ch 6)
- Par-point bumping and rebuilding can introduce interpolation artifacts (Andersen & Piterbarg Vol 1 Ch 6)
- After the crisis, markets often require multiple curves (one for discounting, one for forwarding/projection) due to basis spreads (Andersen & Piterbarg Vol 1 Ch 6)
- Using different curves for pricing and risk may result in poor P&L prediction (Andersen & Piterbarg Vol 3)

### (B) Reasoned Inference (Derived from A)

- Yield DV01 and curve DV01 can differ because yield is an "average" of spot rates; a parallel spot-rate shift does not map identically to a yield shift
- For long fixed-cashflow exposures, $dP/dy < 0$, so DV01 under Tuckman's sign is positive
- PVBP (PV change from bumping fixed rate $K$ by 1bp) equals $N \cdot A \times 10^{-4}$ where $A$ is the annuity factor
- Portfolio PV01 is additive when and only when the bump definition is consistent across all instruments
- The Jacobian $\frac{d\theta}{dq}$ bridges sensitivities in parameter space $(\partial V/\partial\theta)$ to sensitivities in quote space $(dV/dq)$

### (C) Speculation (Clearly Labeled; Minimal)

- I'm not sure which naming convention ("PV01" vs "DV01" vs "PVBP") your desk uses without a desk standard. To be certain, I would need: (i) whether "PV01" refers to a curve bump or a fixed-rate bump, and (ii) which curve(s) are bumped (single-curve vs multi-curve).

---

## Conventions & Notation

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $P$ | Bond full (dirty) price per 100 face (unless stated) |
| $P_{\text{flat}}$ | Bond flat/clean price; $P_{\text{flat}} = P_{\text{full}} - \text{AI}$ |
| $\text{AI}$ | Accrued interest |
| $y$ | Yield to maturity (IRR) |
| $\Delta y$ | Yield change in decimal units (e.g., 1bp $= 0.0001$) |
| $\text{DV01}$ | $-\Delta P/(10000\,\Delta y)$, PV change for 1bp fall in the specified rate measure |
| $P(0,T)$ | Discount factor / zero-coupon bond price at time 0 for maturity $T$ |
| $z(T)$ | Spot/zero rate used to construct $P(0,T)$ (compounding basis must be stated) |
| $\tau_i$ | Accrual year fraction for period $i$ |
| $K$ | Swap fixed rate |
| $A$ | Swap annuity factor: $A = \sum_i \tau_i P(0,T_i)$ |
| $S$ | Par/forward swap rate: $S = \frac{P(0,T_0) - P(0,T_m)}{A}$ |
| $\text{PV01}$ | Naming varies (see §3); in this chapter: "DV01 of PV" under a stated curve bump |
| 1 bp | $10^{-4}$ in decimal rate units |

### Key Identities

$$\boxed{\text{DV01} \equiv -\frac{\Delta P}{10000\,\Delta y}}$$

$$\boxed{A_{k,m}(t) = \sum_{n=k}^{k+m-1} P(t,T_{n+1})\,\tau_n}$$

$$\boxed{S_{k,m}(t) = \frac{P(t,T_k) - P(t,T_{k+m})}{A_{k,m}(t)}}$$

### Defaults Used in Examples

| Convention | Default |
|------------|---------|
| Settlement | At $t=0$ on a coupon date $\Rightarrow$ AI = 0, so clean = dirty |
| Compounding | Annual |
| Coupon frequency | Annual, $\tau_i = 1$ |
| Bond prices | Per 100 face |
| Swap PVs | In dollars for stated notional |
| DV01 sign | Tuckman: PV change for 1bp **fall** in the rate measure |
| Curve bump default | Parallel +1bp bump to zero/spot rates, holding contractual cashflows fixed |

### Sign Convention Warning

- **Tuckman sign (default here):** DV01 is PV change for a 1bp **decline** in rates — typically positive for long fixed-cashflow exposures
- **Hull sign (alternative):** DV01 described as dollar value of a 1bp **increase** in rates — sign flipped
- **Always ask:** "Is DV01 defined for +1bp up, or −1bp down?"

---

## Setup

This chapter answers one of the most important practical questions in rates trading and risk: **"How much money do I make or lose when rates move by 1bp?"**

The answer depends critically on **what you bump**:
- A single yield (YTM)?
- A parallel shift to zero rates?
- Par swap quotes with curve rebuild?
- A localized key-rate?

We carefully distinguish:
- **Yield DV01:** bump a single yield $y$ (bond's YTM)
- **Curve DV01 / PV01:** bump a curve object (zeros/discount factors/par quotes) and reprice

---

## Core Concepts

### 1) DV01 (Dollar Value of 1bp)

**Formal Definition:**

Tuckman defines DV01 as the dollar change in value for a basis point change in interest rates. For a yield bump:

$$\text{DV01} \equiv -\frac{\Delta P}{10000\,\Delta y}$$

The minus sign makes DV01 positive for securities whose prices rise when yields fall.

**Intuition:**

DV01 answers: *"If rates move by 1bp (in the way I've defined), how many dollars do I gain/lose?"*

It is a local (small-shock) linear sensitivity — the first-order term in a Taylor expansion of PV with respect to the chosen rate measure.

**Trading/Risk/Portfolio Practice:**

- **Hedging:** To hedge security $A$ with security $B$, Tuckman gives the hedge ratio:
$$\mathcal{H} \equiv \frac{\text{DV01}_A}{\text{DV01}_B}$$
i.e., use $\mathcal{H}$ units of $B$ per unit of $A$ to match first-order rate exposure.

- **Scaling:** DV01 per 100 face converts to DV01 in dollars for notional $F$:
$$\text{DV01}_\$ = \text{DV01}_{/100} \times \frac{F}{100}$$

---

### 2) Yield-Based DV01 (YTM DV01)

**Formal Definition:**

Yield-to-maturity $y$ is defined as the IRR that discounts all promised cashflows to the full price.

The price–yield function for a coupon bond (Tuckman's notation):

$$P(y) = \sum_i \frac{\text{CF}_i}{(1+y)^{t_i}}$$

where $t_i$ are cashflow times in years.

**Yield DV01** is DV01 where the "rate measure" is this single $y$.

**Intuition:**

Yield DV01 treats the bond as if it were discounted at a single rate $y$. This is a useful approximation and common quoting/risk language for bonds.

**Trading/Risk/Portfolio Practice:**

Many market participants speak of DV01 as yield-based DV01. Tuckman explicitly notes that while DV01's definition allows "any interest rate measure," market usage commonly means yield-based DV01.

**Limitation:** Yield-based DV01 implicitly corresponds to a specific assumption about how the term structure moves. Tuckman notes it effectively assumes a particular pattern (e.g., parallel shifts) and can mis-measure risk when the curve changes shape.

---

### 3) Curve-Based DV01 / PV01 (Discount-Factor DV01)

**Formal Definition:**

A curve-based sensitivity starts from a PV written directly in discount factors:

$$V = \sum_i \text{CF}_i \, P(0,t_i)$$

i.e., "discount each cashflow with the appropriate zero-coupon bond price."

A **curve DV01/PV01** then specifies:
1. Which curve object is shocked (zeros, discount factors, forwards, or quotes)
2. How it is shocked (parallel/localized, bump-and-rebuild, etc.)

and defines the PV change per bp under that shock.

**Intuition:**

Curve DV01 is the "desk-realistic" version: it measures PV sensitivity to the risk factors the desk actually bumps (e.g., discount curve nodes, par swap quotes).

**Trading/Risk/Portfolio Practice:**

In curve frameworks, DV01/PV01 is computed by "bump-and-reprice" on the curve object. Andersen & Piterbarg discuss several ways to perturb curves, including perturbing par points (market quotes) and rebuilding, or perturbing forwards to avoid interpolation artifacts.

---

### 4) PVBP / "Swap Annuity" (A Naming Collision with PV01)

**Formal Definition (Source-Backed):**

Andersen & Piterbarg define the swap annuity factor:

$$A_{k,m}(t) = \sum_{n=k}^{k+m-1} P(t,T_{n+1})\,\tau_n$$

and use it to define the swap rate $S_{k,m}(t)$.

In market language, this annuity factor (times notional, and sometimes times 1bp) is often referred to as **PVBP** (present value of a basis point) for the fixed leg — i.e., the PV change from changing the fixed rate $K$ by 1bp, holding the curve fixed.

**Intuition:**

PVBP answers: *"If I increase the fixed coupon on the swap by 1bp, how much does PV change?"* This is **different** from "PV change for a curve shift," even though both are called "PV01" on some desks.

**Trading/Risk/Portfolio Practice:**

Pay attention to whether someone's "PV01" means:
- Curve PV01 (bump curve), or
- PVBP/annuity PV01 (bump the swap fixed rate $K$)

Andersen & Piterbarg explicitly mention "PVBP matching" as a practical method for mapping risks (swaptions replication), reinforcing PVBP as a standard object.

---

### 5) Key Rate 01 (Localized DV01)

**Formal Definition:**

Tuckman defines a **key rate shift** as a perturbation of a yield curve where one "key rate" is shifted while the curve is adjusted in between key rates.

A **key rate 01** is the change in security value from a 1bp shift in one key rate, holding the rest of the key rates fixed (with the curve adjusted between them per the key-rate-shift definition).

**Intuition:**

Key rate 01s decompose "rate risk" into maturity buckets (2y, 5y, 10y, …) rather than assuming a parallel shift. This is the "what's being bumped" answer at a more granular level.

**Trading/Risk/Portfolio Practice:**

- **Bucketed risk reporting:** Risk systems output DV01 by tenor bucket
- **Curve-hedging:** Choose hedges that match bucket exposures rather than just total DV01

---

## Math and Derivations

### 2.1 DV01 as a Scaled Derivative

Tuckman's discrete definition (for yield $y$):

$$\boxed{\text{DV01} = -\frac{\Delta P}{10000\,\Delta y}} \tag{1}$$

If $P$ is differentiable in the bumped variable (and $\Delta y$ is "small"), then as $\Delta y \to 0$:

$$\boxed{\text{DV01} \approx -\frac{1}{10000}\frac{dP}{dy}} \tag{2}$$

**Unit Check:**
- $P$ is in price points per 100 (or dollars if already scaled)
- $y$ is in decimal units
- $dP/dy$ has units "price per 1.00 of yield"
- Dividing by 10000 converts "per 1.00" to "per 1bp"

**Sanity Check:**

For a long bond, $dP/dy < 0$ (higher yields $\Rightarrow$ lower price), so DV01 in (2) is typically positive under the Tuckman sign convention.

---

### 2.2 Derivative of the Bond Price–Yield Function

From Tuckman's price–yield function:

$$P(y) = \sum_i \frac{\text{CF}_i}{(1+y)^{t_i}} \tag{3}$$

Differentiate one term $\text{CF}_i (1+y)^{-t_i}$:

$$\frac{d}{dy}\left[\text{CF}_i (1+y)^{-t_i}\right] = -t_i \, \text{CF}_i \, (1+y)^{-(t_i+1)}$$

So:

$$\boxed{\frac{dP}{dy} = -\sum_i \frac{t_i \, \text{CF}_i}{(1+y)^{t_i+1}}} \tag{4}$$

Plug into DV01 approximation (2):

$$\boxed{\text{DV01} \approx \frac{1}{10000}\sum_i \frac{t_i \, \text{CF}_i}{(1+y)^{t_i+1}}} \tag{5}$$

**Sanity Check:**

Each term is positive; longer-dated cashflows contribute more (via $t_i$).

---

### 2.3 Curve DV01 When PV is Expressed in Discount Factors

Let PV be:

$$V(\{P(0,t)\}) = \sum_i \text{CF}_i \, P(0,t_i) \tag{6}$$

Discount factors $P(0,t)$ are foundational in interest-rate modeling: they are the prices of default-free zero-coupon bonds.

A curve bump must specify a mapping:

$$P(0,t) \mapsto P_{\text{bumped}}(0,t)$$

For example, if the curve is parameterized by spot rates $z(t)$ with a stated compounding convention, then a "parallel +1bp zero-rate bump" means:

$$z(t) \mapsto z(t) + 10^{-4} \quad \forall t$$

and then recompute $P(0,t)$ from $z(t)$.

A bump-and-reprice DV01/PV01 (Tuckman sign) can be computed as:

$$\boxed{\text{PV01}_{\text{curve}} \approx \frac{V_\downarrow - V_\uparrow}{2} \quad \text{when the shock is } \pm 1\text{ bp}} \tag{7}$$

(Here $V_\uparrow$ is PV under +1bp, $V_\downarrow$ under −1bp.)

**Unit Check:**
- $V$ in dollars $\Rightarrow$ PV01 in dollars per bp
- If $V$ is a price per 100 $\Rightarrow$ PV01 in price points per 100 per bp

---

### 2.4 Swap PV Decomposition and Its Connection to PVBP

Andersen & Piterbarg define the annuity factor and swap rate:

$$A = \sum_i \tau_i P(0,T_i), \quad S = \frac{P(0,T_0) - P(0,T_m)}{A} \tag{8}$$

A common single-curve representation for the PV of a spot-starting fixed-for-float swap:

$$\boxed{V_{\text{swap}} = N(K-S) \, A} \tag{9}$$

where:
- $N$ is notional
- $K$ is the contractual fixed rate
- $S$ is the par swap rate from the curve
- $A$ is the annuity factor

**Key Interpretation:**

Holding the curve fixed, $\frac{\partial V_{\text{swap}}}{\partial K} = NA$.

Thus "PVBP" (PV per 1bp change in $K$) is often $NA \times 10^{-4}$.

---

### 2.5 "DV01 Depends on Parameterization" and the Jacobian Idea

Andersen & Piterbarg emphasize that curve perturbations can be defined in different parameterizations (par points vs forwards), and that interpolation artifacts can cause inconsistencies.

Conceptually:
- Let market quotes be $q$ (e.g., par swap rates)
- The curve-building procedure produces internal parameters $\theta$ (e.g., zero rates at nodes)
- PV is $V(\theta)$, but we often want risk to quotes $q$

Then, by the chain rule:

$$\boxed{\frac{dV}{dq} = \frac{\partial V}{\partial \theta} \, \frac{d\theta}{dq}} \tag{10}$$

This is the core of "mapping sensitivities":
- Bump in quote space (par-quote bump + rebuild) gives $dV/dq$ directly
- Bump in parameter space (zero-rate bump) gives $\partial V/\partial \theta$
- The Jacobian bridges them

Andersen & Piterbarg explicitly discuss a Jacobian method for yield curve risk (and distinguish it from "rebuild").

---

## "What's Being Bumped?" Taxonomy

### (i) Yield Bump (Single YTM)

- **Definition:** Bump the bond's yield to maturity $y \mapsto y \pm 1\text{bp}$ and reprice via $P(y)$
- **Use case:** Bond risk reporting when traders hedge/think in "yield space"
- **Caveat:** Yield is an average of spot rates; a curve move does not necessarily map to a pure $y$ move

### (ii) Zero-Rate Bump (Curve Bump in Zero Space)

- **Definition:** Bump spot/zero rates $z(t)$ by +1bp for all maturities (parallel) or locally (key-rate style), recompute discount factors $P(0,t)$, then reprice
- **Use case:** Curve-hedging and risk systems that store curves as zero nodes
- **Caveat:** Depends on compounding convention and curve interpolation

### (iii) Discount-Factor Bump

- **Definition:** Bump $P(0,t)$ directly (e.g., via a mapping designed to approximate a rate shift)
- **Use case:** Numerical stability or when the curve is stored as discount factors
- **Caveat:** "1bp DF bump" is ambiguous unless you specify the mapping from DF changes to rate changes

### (iv) Par-Rate / Instrument-Quote Bump with Curve Rebuild

- **Definition:** Bump market quotes (e.g., par swap rates) by +1bp and rebuild the curve, then reprice
- **Source support:** Andersen & Piterbarg describe this "par-point approach" and warn that interpolation can introduce artifacts
- **Use case:** Risk reporting consistent with how curves are bootstrapped from quotes ("quote PV01")

### (v) Key-Rate / Localized Bump (Preview-Level)

- **Definition:** Bump one key rate and adjust the curve between adjacent key rates to remain continuous; compute "key rate 01"
- **Use case:** Bucketed hedging; understanding curve-shape exposures

---

## Worked Examples

**Reminder:** In all examples we state (i) what is being bumped, (ii) bump size, (iii) compounding, (iv) clean vs dirty, and (v) price units.

---

### Example A: Bond DV01 via Yield Bump

**Goal:** Compute DV01 by bumping the bond's yield to maturity $y$ by $\pm 1$bp and using a central difference.

**Instrument:**
- 5-year annual coupon bond
- Face $= 100$
- Coupon rate $= 5\%$ $\Rightarrow$ coupon $C = 5$ per year
- Maturity $T = 5$ years
- Yield to maturity $y = 4\% = 0.04$ (annual compounding)

**Conventions:**
- Price is full/dirty, but assume settlement on coupon date so AI = 0 $\Rightarrow$ clean = dirty
- Price quoted per 100 face
- Bump: $y \mapsto y \pm 0.0001$

**Pricing (Tuckman price–yield function):**

$$P(y) = \sum_{t=1}^{4} \frac{5}{(1+y)^t} + \frac{105}{(1+y)^5} \tag{A.1}$$

**Step 1: Base price**

$$P(4.00\%) = 104.4518223310$$

**Step 2: Reprice at bumped yields**

$$P(4.01\%) = 104.4060662637$$

$$P(3.99\%) = 104.4976039645$$

**Step 3: Central-difference DV01 (Tuckman sign)**

For a symmetric $\pm 1$bp bump, $10000\,\Delta y = 1$ and the central difference simplifies to:

$$\text{DV01}_{/100} \approx \frac{P(y-1\text{bp}) - P(y+1\text{bp})}{2} \tag{A.2}$$

So:

$$\text{DV01}_{/100} \approx \frac{104.4976039645 - 104.4060662637}{2} = 0.0457688504$$

**Interpretation:**

Under Tuckman sign, DV01 is positive, meaning a 1bp fall in yield increases price by about $0.04577$ points per 100 face.

**Convert to dollars per $1mm face:**

$$\text{DV01}_\$ = 0.0457688504 \times \frac{1{,}000{,}000}{100} = 0.0457688504 \times 10{,}000 = \$457.6885$$

---

### Example B: Bond DV01 via Zero-Curve Bump

**Goal:** Price the same bond using a non-flat zero curve, then compute a curve DV01 by bumping the zero rates +1bp (parallel) and repricing.

**Instrument:** Same bond as Example A: 5y, 5% annual coupon, face 100.

**Conventions:**
- Full price, AI = 0
- Zero curve expressed as annual-compounded spot rates $z(t)$ at integer maturities
- Discount factor: $P(0,t) = \frac{1}{(1+z(t))^t}$ (toy compounding choice)
- Bump: parallel $z(t) \mapsto z(t) \pm 1$bp for all maturities $t = 1, \ldots, 5$

**Step 1: Choose a non-flat zero curve**

| Maturity | Spot Rate |
|----------|-----------|
| 1y | 2.00% |
| 2y | 2.50% |
| 3y | 5.00% |
| 4y | 5.50% |
| 5y | 3.9679% (calibrated to match Example A base price) |

**Step 2: Compute discount factors**

$$\begin{aligned}
DF_1 &= \frac{1}{1.02} = 0.980392 \\
DF_2 &= \frac{1}{1.025^2} = 0.951814 \\
DF_3 &= \frac{1}{1.05^3} = 0.863838 \\
DF_4 &= \frac{1}{1.055^4} = 0.807217 \\
DF_5 &= \frac{1}{1.039679336^5} = 0.823195
\end{aligned}$$

**Step 3: Price from discount factors**

$$P_{\text{curve}} = \sum_{t=1}^{4} 5 \cdot DF_t + 105 \cdot DF_5$$

Numerically:
- $5(DF_1 + DF_2 + DF_3 + DF_4) = 5(3.603261) = 18.016305$
- $105 \cdot DF_5 = 86.435517$
- $P_{\text{curve}} = 104.451822$ (matches Example A by construction)

**Step 4: Parallel +1bp zero-rate bump and reprice**

- Under +1bp bump: $P_\uparrow = 104.4060930975$
- Under −1bp bump: $P_\downarrow = 104.4975768239$

**Step 5: Curve DV01**

$$\text{DV01}_{\text{curve},/100} \approx \frac{P_\downarrow - P_\uparrow}{2} = \frac{104.4975768239 - 104.4060930975}{2} = 0.0457418632$$

---

### Example C: Compare A vs B — Yield-DV01 $\neq$ Curve-DV01

**Numbers:**
- Yield DV01 (Example A): $0.0457688504$ per 100
- Curve DV01 (Example B): $0.0457418632$ per 100

**Difference:**

$$0.0457688504 - 0.0457418632 \approx 0.00002699 \text{ per 100}$$

**Why they differ (conceptual):**

- Yield DV01 bumps one scalar $y$ in the price–yield function $P(y)$
- Curve DV01 bumps multiple curve points $z(t)$ and reprices by discounting each cashflow
- In a non-flat curve, "+1bp in yield" is not the same economic shock as "+1bp to every spot rate"
- Yield is an "average" of spot rates; the mapping from a curve shift to a yield shift is not identity

**Practical implication:** If your hedges are computed in yield space but your desk P&L is driven by curve reshaping, you can see residual P&L even when DV01s "match." This aligns with Tuckman's warning that yield-based DV01 can mis-measure risk when the curve does not move in the assumed way.

---

### Example D: Swap PV01 — Bump the Discount Curve +1bp

**Goal:** Value a plain-vanilla fixed-for-float swap and compute PV01 under a parallel +1bp bump to the discount curve.

**Instrument:**
- Spot-starting swap, annual payments at $T_1 = 1$, $T_2 = 2$, $T_3 = 3$ years
- Accruals: $\tau_i = 1$
- Notional $N = \$1{,}000{,}000$
- Receive fixed rate $K = 4.5\% = 0.045$
- Pay floating (single-curve assumption)

**Conventions:**
- Discount curve: flat annual-compounded $r = 4\%$
- "PV01 (curve)" in this example: PV change for 1bp fall in the discount curve level

**Step 1: Discount factors**

$$P(0,t) = \frac{1}{(1+r)^t}, \quad r = 0.04$$

| Maturity | Discount Factor |
|----------|-----------------|
| 1y | $P_1 = 1/1.04 = 0.9615384615$ |
| 2y | $P_2 = 1/1.04^2 = 0.9245562130$ |
| 3y | $P_3 = 1/1.04^3 = 0.889000$ |

**Step 2: Annuity factor and par swap rate**

$$A = \sum_{i=1}^{3} \tau_i P_i = P_1 + P_2 + P_3 = 2.7750910332$$

$$S = \frac{1 - P_3}{A} = \frac{0.110999}{2.775091} \approx 0.04$$

**Step 3: Swap PV**

Using $V = N(K-S)A$:

$$V_0 = 1{,}000{,}000 \times (0.045 - 0.04) \times 2.7750910332 = \$13{,}875.4552$$

**Step 4: Bump discount curve +1bp**

- New curve level: $r_\uparrow = 4.01\%$
- Recompute $P_i^\uparrow$, $A^\uparrow = 2.7745644209$, and $S^\uparrow = 4.01\%$
- New PV:

$$V_\uparrow = 1{,}000{,}000 \times (0.045 - 0.0401) \times 2.7745644209 = \$13{,}595.3657$$

**PV01 (Tuckman sign):**

For a +1bp rise, PV falls by $V_\uparrow - V_0 = -\$280.0895$.

Under Tuckman sign (PV change for 1bp fall):

$$\text{PV01} \approx V_0 - V_\uparrow = \$280.0895 \text{ per bp}$$

**Central-difference cross-check:**

- $r_\downarrow = 3.99\%$ gives $V_\downarrow = \$14{,}155.6508$
- Central PV01:

$$\text{PV01}_{\text{cd}} \approx \frac{V_\downarrow - V_\uparrow}{2} = \$280.1426$$

**Sanity check:** Receiving fixed is "long fixed cashflows," so higher rates generally reduce PV; PV01 should be positive under the "1bp fall" sign convention.

---

### Example E: Projection vs Discount Bump (Multi-Curve PV01 Split)

**Goal:** In a multi-curve setup, bump discount curve vs projection curve separately and compare PV01s.

**Source-backed motivation:** Multiple curves are used due to basis spreads; separate discounting and forwarding curves are often required (Andersen & Piterbarg Vol 1).

**Instrument (toy 2-year swap):**
- Annual payments at $T_1 = 1$, $T_2 = 2$
- Notional $N = \$1{,}000{,}000$
- Receive fixed $K = 5.00\% = 0.05$
- Pay floating coupons based on projection forwards $L_1, L_2$

**Curves:**

| Curve | Parameterization |
|-------|------------------|
| Discount (OIS-like) | Flat $r_d = 2\%$ annual comp |
| Projection (Libor-like) | 1y spot = 3.00%, 2y spot = 3.50% |

Discount factors:
- $P_d(0,1) = 1/1.02 = 0.980392$
- $P_d(0,2) = 1/1.02^2 = 0.961169$

Projection forwards:
- $L_1 = P_f(0,0)/P_f(0,1) - 1 = 3.00\%$
- $L_2 = P_f(0,1)/P_f(0,2) - 1 \approx 4.0024\%$

**PV formula (multi-curve coupon PV):**

$$V = N\left(K\sum_{i=1}^{2} \tau_i P_d(0,T_i)\right) - N\left(\sum_{i=1}^{2} \tau_i L_i \, P_d(0,T_i)\right)$$

**Step 1: Base PV**

- $\sum P_d = 1.941561$
- Fixed leg PV: $NK\sum P_d = 1{,}000{,}000 \times 0.05 \times 1.941561 = 97{,}078.0469$
- Float leg PV: $N(L_1 P_1^d + L_2 P_2^d) = 67{,}881.8507$
- Swap PV: $V_0 = 29{,}196.1962$

**Step 2: Discount curve PV01** (bump $r_d$, hold projection curve fixed)

Bump $r_d \mapsto r_d \pm 1$bp, recompute $P_d$, but keep $L_1, L_2$ unchanged.

- $V_d^\uparrow = 29{,}192.3995$
- $V_d^\downarrow = 29{,}200.0043$

Central PV01 (discount):

$$\text{PV01}_d \approx \frac{V_d^\downarrow - V_d^\uparrow}{2} = \$3.8024$$

**Step 3: Projection curve PV01** (bump projection spots, hold discount curve fixed)

Bump projection spot rates by $\pm 1$bp, recompute $P_f$ and forwards $L_1, L_2$.

- $V_f^\uparrow = 29{,}002.0478$
- $V_f^\downarrow = 29{,}390.3554$

Central PV01 (projection):

$$\text{PV01}_f \approx \frac{V_f^\downarrow - V_f^\uparrow}{2} = \$194.1538$$

**Interpretation:**

- **Discount PV01 is small** because it measures sensitivity to discounting when net cashflows are already largely set
- **Projection PV01 is much larger** because a 1bp change in forwards directly changes floating coupons by roughly $N \times 1\text{bp} \times \sum P_d$

**Important caveat:** This is a preview-level toy multi-curve computation. Production systems must define precisely which curve instruments are bumped and how curve calibration is re-run.

---

### Example F: Discount-Factor Bump vs Zero-Rate Bump — Nonlinearity Matters

**Goal:** Implement the "same ~1bp" shock two ways and show PV01 differences due to nonlinearity.

**Instrument:** Single cashflow: $100 at $T = 10$ years (a zero-coupon payoff)

**Conventions:**
- Annual compounding
- Base rate $r = 5\%$

**Step 1: Base discount factor and PV**

$$DF = \frac{1}{(1+r)^{10}} = \frac{1}{1.05^{10}} = 0.6139132535$$

$$V_0 = 100 \times DF = 61.39132535$$

**Method 1 (Exact zero-rate bump):**

Bump rate $r \mapsto r + 1\text{bp} = 0.0501$:

$$DF_\uparrow = \frac{1}{(1.0501)^{10}} = 0.6133288804, \quad V_\uparrow = 61.33288804$$

PV01 (Tuckman sign for a +1bp rise):

$$\text{PV01}_{\text{exact}} = V_0 - V_\uparrow = 0.05843731 \text{ per 100 notional}$$

**Method 2 (Approximate discount-factor bump via linearization):**

Linearize $DF(r)$ around $r$ using:

$$DF(r + \Delta r) \approx DF(r) + \frac{dDF}{dr}\Delta r, \quad DF(r) = (1+r)^{-10}$$

Then:

$$\frac{dDF}{dr} = -10(1+r)^{-11} = -\frac{10}{1+r}DF$$

So for $\Delta r = 0.0001$:

$$DF_{\text{lin}}^\uparrow \approx DF\left(1 - \frac{10\Delta r}{1+r}\right) = 0.6139132535\left(1 - \frac{10 \times 0.0001}{1.05}\right) = 0.6133285743$$

$$V_{\text{lin}}^\uparrow = 100 \times DF_{\text{lin}}^\uparrow = 61.33285743$$

$$\text{PV01}_{\text{lin}} = V_0 - V_{\text{lin}}^\uparrow = 0.05846793$$

**Difference (nonlinearity / convexity):**

$$\text{PV01}_{\text{lin}} - \text{PV01}_{\text{exact}} = 0.00003061 \text{ per 100}$$

This is tiny for 1bp, but for very large notionals it can matter (and for larger bumps it matters more).

**Which is preferable?**

For risk systems, "bump-and-reprice with the exact curve mapping" is usually preferred because it avoids approximation error. Linear DF approximations are useful for intuition but miss convexity terms by construction.

---

### Example G: Par-Instrument Bump / Rebuild Curve — "Quote PV01"

**Goal:** Bump a small set of par quotes, rebuild a toy curve, and compute PV01 for an instrument priced off that curve.

**Source-backed context:** Andersen & Piterbarg describe a par-point approach: perturb par points and rebuild a curve (and note possible interpolation artifacts). This is the conceptual basis for "quote PV01."

**Toy market quotes (par swap rates as curve nodes):**

| Tenor | Par Rate |
|-------|----------|
| 2y | 3.00% |
| 5y | 3.50% |
| 10y | 4.00% |

**Toy curve-building rule (explicitly a simplification):**

Assumption (toy): treat these par swap rates as if they were zero rates at those maturities, and linearly interpolate zero rates between nodes $(0, 0)$, $(2, 3\%)$, $(5, 3.5\%)$, $(10, 4\%)$.

**Instrument:** 7-year annual coupon bond, face 100, coupon rate 5% (coupon 5)

**Step 1: Build base interpolated zero rates**

| Year | Zero Rate |
|------|-----------|
| 1 | 1.50% |
| 2 | 3.00% |
| 3 | 3.1667% |
| 4 | 3.3333% |
| 5 | 3.50% |
| 6 | 3.60% |
| 7 | 3.70% |

**Step 2: Compute discount factors and base price**

| Year | DF |
|------|----|
| 1 | $1/1.015 = 0.985222$ |
| 2 | $1/1.03^2 = 0.942596$ |
| 3 | $1/1.0316667^3 = 0.910714$ |
| 4 | $1/1.0333333^4 = 0.877078$ |
| 5 | $1/1.035^5 = 0.841973$ |
| 6 | $1/1.036^6 = 0.808801$ |
| 7 | $1/1.037^7 = 0.775441$ |

Price:

$$P_0 = 5\sum_{t=1}^{6} DF_t + 105 \cdot DF_7 \approx 108.253225$$

**Step 3: Bump par quotes +1bp and rebuild**

New quotes: $S_{2y} = 3.01\%$, $S_{5y} = 3.51\%$, $S_{10y} = 4.01\%$

Rebuild zero curve with the same interpolation rule, recompute $DF_t$, and reprice:

- $P_{\text{quote}}^\uparrow \approx 108.189755$
- $P_{\text{quote}}^\downarrow \approx 108.316795$ (for −1bp bump)

**Quote PV01 (central):**

$$\text{PV01}_{\text{quote},/100} \approx \frac{P_{\text{quote}}^\downarrow - P_{\text{quote}}^\uparrow}{2} = 0.063520$$

**Interpretation:** This PV01 is sensitivity to input quotes (par rates) under a rebuild procedure. It can differ from a "parallel zero-rate bump PV01" because the rebuild induces a different shape change — exactly the "what's being bumped" message.

---

### Example H: Localized Bump Preview — A Key-Rate-Style Bump

**Goal:** Apply a localized/key-rate-style bump around the 5-year point and show the PV01 concentrates in that bucket.

**Source-backed concept:** Tuckman defines key rate shifts and key rate 01 as PV changes under 1bp shifts of individual key rates with curve adjustment in between.

**Baseline curve:** Same toy curve node set as Example G: nodes at 2y (3.00%), 5y (3.50%), 10y (4.00%)

**Localized bump definition (toy key-rate bump at 5y):**
- Increase the 5y node by +1bp: $z(5) = 3.50\% \to 3.51\%$
- Keep 2y and 10y nodes unchanged
- Reinterpolate linearly on $[2,5]$ and $[5,10]$

This produces a triangular-shaped bump: 0 at 2y, +1bp at 5y, 0 at 10y.

**Instrument:** Same 7y 5% coupon bond from Example G.

**Step 1: Rebuilt curve under +1bp 5y key-rate bump**

Derived interpolated zeros: $z(1), z(2)$ unchanged; $z(3) \approx 3.17\%$, $z(4) \approx 3.34\%$, $z(5) = 3.51\%$, $z(6) \approx 3.608\%$, $z(7) \approx 3.706\%$

Reprice: $P_{5y}^\uparrow \approx 108.214775$

**Step 2: Rebuilt curve under −1bp 5y key-rate bump**

$z(5) = 3.49\%$, 2y and 10y fixed, reinterpolate.

Reprice: $P_{5y}^\downarrow \approx 108.291670$

**Key-rate PV01 (central):**

$$\text{KRD01}_{5y,/100} \approx \frac{P_{5y}^\downarrow - P_{5y}^\uparrow}{2} = 0.0384475$$

**Interpretation:**

Compare to Example G quote PV01 (parallel bump to multiple par quotes): $0.06352$.

The localized bump isolates sensitivity concentrated around the 5y bucket, as key rate risk intends.

---

### Example I: Portfolio Additivity — Consistent Bump Definition is Everything

**Goal:** Show PV01 additivity for a small portfolio under a consistent bump definition.

**Bump definition (consistent across all positions):**
Parallel $\pm 1$bp bump to a flat discount curve $r = 4\%$ (annual comp).

**Portfolio:**

| Position | Instrument | Face/Notional | DV01 per 100 | Position DV01 |
|----------|------------|---------------|--------------|---------------|
| Bond 1 (long) | 5y 5% coupon bond | $+\$1{,}000{,}000$ | $0.0457688504$ | $+\$457.6885$ |
| Bond 2 (short) | 2y 2% coupon bond | $-\$1{,}000{,}000$ | $0.0183204373$ | $-\$183.2044$ |
| Swap (receive fixed) | 3y swap, $K = 4.5\%$ | $\$1{,}000{,}000$ | — | $+\$280.1426$ |

**Step 1: Convert bond DV01s to dollars**

For face $F = \$1{,}000{,}000$:

$$\text{DV01}_\$ = \text{DV01}_{/100} \times \frac{F}{100} = \text{DV01}_{/100} \times 10{,}000$$

**Step 2: Sum individual PV01s**

$$\text{PV01}_{\text{sum}} = 457.6885 - 183.2044 + 280.1426 = \$554.6267$$

**Step 3: Portfolio bump-and-reprice check**

Compute portfolio PV at $r$, $r \pm 1$bp and apply central difference.

- Base PV: $V_0 = 1{,}044{,}518.2233 - 962{,}278.1065 + 13{,}875.4552 = 96{,}115.5720$
- PV under +1bp: $V_\uparrow = 1{,}044{,}060.6626 - 962{,}094.9285 + 13{,}595.3657 = 95{,}561.0998$
- PV under −1bp: $V_\downarrow = 1{,}044{,}976.0396 - 962{,}461.3372 + 14{,}155.6508 = 96{,}670.3533$

Portfolio PV01:

$$\text{PV01}_{\text{portfolio}} = \frac{V_\downarrow - V_\uparrow}{2} = \$554.6267$$

This matches the sum (up to rounding).

**Key message:** Additivity holds when and only when the bump definition is consistent across instruments.

---

## Practical Notes

### Common Ambiguity Traps

**DV01 sign conventions:**
- Tuckman: change for 1bp decline, via the minus sign in the definition
- Hull glossary: describes DV01 for 1bp increase
- Desk practice may report absolute values — always document the sign

**"PV01" meaning (naming collisions):**
- Discount-curve PV01 (curve bump)
- Swap PVBP / annuity PV01 (fixed-rate bump)
- "Risky PV01" can mean something else in credit contexts (not developed here)

**Yield bump vs curve bump:**
- Yield DV01 assumes a specific mapping between curve changes and yields; Tuckman notes yield-based DV01 can be inaccurate when the curve changes shape
- Curve PV01 depends on curve parameterization (zeros vs quotes vs forwards)

**Clean vs dirty price in risk:**
- Tuckman distinguishes flat vs full price and AI
- Risk systems often compute PV01 on full price because it is the PV of future cashflows; but reporting may quote clean prices

**Compounding/day-count mismatches:**
- A "1bp bump" is meaningless unless the compounding basis and day-count conventions are fixed

### Implementation Pitfalls

**Bump-and-rebuild stability:**
- Par-point bumping + rebuild can introduce artifacts due to interpolation; Andersen & Piterbarg discuss this issue and motivate forward-rate-based perturbations to avoid oscillations

**Discontinuities around coupon dates / settlement:**
- Accrued interest and cashflow timing changes can create apparent "jumps" in DV01 if you don't hold settlement conventions fixed

**Bump size too large/small:**
- Too large: captures convexity and cross terms (not pure DV01)
- Too small: numerical noise; Tuckman notes smaller bumps can create errors due to imperfect pricing

### Verification Tests

1. **Repricing check:** Base PV matches the system's base PV
2. **Sign check:** Higher rates should generally lower PV for long fixed cashflows (receive fixed, long bond)
3. **Scaling with notional:** PV01 should scale linearly with face/notional
4. **Small bump consistency:** Compute DV01 with 1bp, 0.5bp, 0.1bp; results should converge (within numerical noise)
5. **Consistency across curve definitions:** If you change the bumped object (yield vs zero vs quotes), DV01 should change — that is the point — but you must know which one your desk reports

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **DV01 is a PV change per 1bp move** — but only after you specify what rate measure is bumped
2. **Tuckman's DV01 definition** uses a minus sign so DV01 is PV change for a 1bp fall in the chosen rate
3. **Hull's DV01 wording** references a 1bp increase; sign conventions differ
4. **Yield DV01** bumps a single YTM in the price–yield function
5. **Curve PV01** bumps a curve object (zeros/DFs/quotes) and reprices by discounting cashflows
6. **Yield DV01 and curve PV01 can differ** because yield is an "average" of spot rates; curve moves do not map 1–1 to yield moves
7. **Par-quote bump-and-rebuild PV01** measures sensitivity to market quotes, not necessarily to zero nodes
8. **Key rate 01s** isolate localized curve exposures and avoid the parallel-shift assumption
9. **Multi-curve risk** splits into discount-curve PV01 and projection-curve PV01; bumping one vs the other can produce very different sensitivities
10. **Inconsistent curve/risk frameworks** can break P&L explain; ensure the bump and curve used for risk align with pricing

### Cheat Sheet: Definitions + Bump Taxonomy + "When to Use Which"

**DV01 (Tuckman sign):**

$$\text{DV01} = -\frac{\Delta V}{10000\,\Delta r} \quad \Rightarrow \quad \text{DV01} \approx V(r - 1\text{bp}) - V(r) \text{ for } \Delta r = 1\text{bp}$$

**Bump Taxonomy:**

| Bump Type | What is Shocked | When to Use |
|-----------|-----------------|-------------|
| Yield bump (YTM) | $y \pm 1$bp $\to P(y)$ | Bond trader quoting in yield (but beware curve-shape moves) |
| Zero-rate bump | $z(t) \pm 1$bp $\to P(0,t) \to V$ | Curve hedging / swap books |
| DF bump | $P(0,t)$ shocked directly | Must define mapping |
| Par-quote bump | Market quotes $\pm 1$bp $\to$ curve rebuild $\to V$ | Risk to market quotes |
| Key-rate bump | Bump one key rate $\to$ key rate 01 | Hedging curve shape |

---

### Flashcards (30 Q/A Pairs)

1. **Q:** What is 1bp in decimal units? **A:** $10^{-4}$
2. **Q:** Tuckman DV01 sign: up or down move? **A:** DV01 is PV change for a 1bp fall in the rate measure
3. **Q:** Hull DV01 wording refers to what direction? **A:** 1bp increase in rates
4. **Q:** What must be specified to make DV01 meaningful? **A:** What rate/curve object is bumped and how
5. **Q:** Define yield to maturity in one phrase. **A:** The IRR that discounts cashflows to the full price
6. **Q:** Write the price–yield function for a coupon bond. **A:** $P(y) = \sum_i \text{CF}_i/(1+y)^{t_i}$
7. **Q:** Yield DV01 bumps what object? **A:** The scalar yield $y$
8. **Q:** Curve PV01 bumps what object? **A:** A curve (zero rates, DFs, par quotes, etc.)
9. **Q:** What is $P(0,T)$ in curve modeling? **A:** Price of a zero-coupon bond paying 1 at $T$
10. **Q:** What is a key rate 01? **A:** PV change from a 1bp shift in one key rate
11. **Q:** Why can yield DV01 mis-measure risk? **A:** It assumes a particular curve movement pattern; curve reshaping breaks it
12. **Q:** What is "quote PV01"? **A:** Sensitivity to market quotes under bump-and-rebuild
13. **Q:** What is PVBP (conceptually)? **A:** PV change from bumping fixed coupon rate by 1bp (holding curve fixed)
14. **Q:** Swap annuity factor formula? **A:** $A = \sum \tau_i P(0,T_i)$
15. **Q:** Swap rate formula? **A:** $S = (P(0,T_0) - P(0,T_m))/A$
16. **Q:** Why does PV01 depend on parameterization? **A:** Different bumps correspond to different shocks in risk-factor space
17. **Q:** What is a par-point approach? **A:** Bump par points and rebuild curve
18. **Q:** Why can par-point bumping cause artifacts? **A:** Interpolation can oscillate under perturbations
19. **Q:** What does "hold fixed" usually mean in risk bumps? **A:** Contractual cashflow schedule, settlement conventions
20. **Q:** Clean vs dirty price relationship? **A:** Full (dirty) = flat (clean) + accrued interest
21. **Q:** How do you convert DV01 per 100 to DV01 in dollars? **A:** Multiply by face/100
22. **Q:** What is the key sanity sign check for a receiver swap? **A:** Higher rates → lower PV (typically)
23. **Q:** What is the Jacobian idea in curve risk? **A:** $dV/dq = (\partial V/\partial\theta)(d\theta/dq)$
24. **Q:** What can break P&L explain in curve risk? **A:** Using different curves for pricing and risk
25. **Q:** What is a "parallel bump"? **A:** Same bp shift applied to all maturities
26. **Q:** What is a "localized bump"? **A:** Shift concentrated in one maturity bucket
27. **Q:** Multi-curve splits PV01s into what? **A:** Discount PV01 and projection PV01
28. **Q:** Why can projection PV01 be large? **A:** Forwards directly scale coupon cashflows
29. **Q:** Why can tiny bump sizes be problematic? **A:** Numerical noise/pricing errors
30. **Q:** What is the main question to ask when someone reports "PV01"? **A:** "What exactly was bumped?"

---

## Mini Problem Set (16 Questions)

**Solution sketches provided for questions 1–8.**

---

**1.** Define DV01 using Tuckman's convention and state its sign interpretation for a long bond.

*Sketch:* DV01 $= -\Delta P/(10000\,\Delta y)$; positive for long bond because $\Delta P > 0$ when $\Delta y < 0$.

---

**2.** A bond is priced at 101.20 per 100 and has DV01 $= 0.042$ per 100. What is DV01 in dollars for $25mm face?

*Sketch:* Multiply by $F/100$: $0.042 \times 25{,}000{,}000/100 = 0.042 \times 250{,}000 = \$10{,}500$.

---

**3.** Explain why yield DV01 can differ from curve DV01 even for the same bond.

*Sketch:* Yield DV01 bumps scalar $y$; curve DV01 bumps term structure $z(t)$; yield is an average of spot rates; mapping not identity when curve non-flat.

---

**4.** For a 3-year annual coupon bond (face 100, coupon 6%), write $P(y)$ and compute $dP/dy$ symbolically.

*Sketch:* Use $P(y) = 6/(1+y) + 6/(1+y)^2 + 106/(1+y)^3$; derivative via $-t \cdot \text{CF}/(1+y)^{t+1}$ (see §2.2).

---

**5.** A swap annuity factor is $A = 4.20$ (per 1 notional). What is PVBP (PV change for 1bp change in fixed rate) for $N = \$50$mm?

*Sketch:* PVBP $\approx NA \times 10^{-4} = 50{,}000{,}000 \times 4.20 \times 10^{-4} = \$21{,}000$. (Note: naming PVBP vs PV01 must be stated.)

---

**6.** Construct a central-difference PV01 formula for a curve bump of $\pm 1$bp.

*Sketch:* PV01 $\approx (V_\downarrow - V_\uparrow)/2$.

---

**7.** Describe a key rate shift and key rate 01 in words.

*Sketch:* Shift one key rate by 1bp, adjust curve between adjacent key rates to remain continuous; key rate 01 is PV change under that shift.

---

**8.** Why might using a very small bump (e.g., 0.01bp) yield unstable DV01 estimates?

*Sketch:* Numerical noise/model pricing errors can dominate; Tuckman notes smaller bumps can introduce errors due to imperfect price models.

---

**9.** In a multi-curve framework, define discount vs projection PV01 and give a practical reason they differ.

---

**10.** Show that for a par swap in a single-curve world, PV is zero but PV01 is non-zero.

---

**11.** For a bond portfolio, explain why reporting DV01 as a single number implicitly assumes a parallel curve shift.

---

**12.** Design a localized bump around 5y using a triangular bump that is 0 at 2y and 10y.

---

**13.** Give a real-world scenario where yield DV01 hedging fails even if DV01s match.

---

**14.** Explain why quote PV01 depends on curve construction and interpolation.

---

**15.** Describe (conceptually) how the Jacobian method maps quote sensitivities to curve node sensitivities.

---

**16.** List three verification tests you would run on a PV01 implementation.

---

## Source Map

### (A) Verified Facts — Specific Citations

| Fact | Source |
|------|--------|
| DV01 = dollar change in value for 1bp change | Tuckman Ch 5-6 |
| DV01 definition with minus sign | Tuckman Ch 5 |
| Hull DV01 = dollar value of 1bp increase | Hull Ch 4 glossary |
| YTM is IRR discounting cashflows to full price | Tuckman Ch 3 |
| Price–yield function $P(y) = \sum \text{CF}/(1+y)^t$ | Tuckman Ch 3-4 |
| Discount factors = prices of zero-coupon bonds | Andersen & Piterbarg Vol 1 |
| Annuity factor $A = \sum \tau P(0,T)$ | Andersen & Piterbarg Vol 1 |
| Swap rate $S = (P_0 - P_m)/A$ | Andersen & Piterbarg Vol 1 |
| Key rate shifts / key rate 01 | Tuckman Ch 6 |
| Par-point approach with rebuild | Andersen & Piterbarg Vol 1 Ch 6 |
| Interpolation artifacts in par-point bumping | Andersen & Piterbarg Vol 1 Ch 6 |
| Multi-curve: discounting vs forwarding curves | Andersen & Piterbarg Vol 1 Ch 6 |
| Mismatch in pricing/risk curves breaks P&L explain | Andersen & Piterbarg Vol 3 |
| PVBP matching for swaptions | Andersen & Piterbarg (context) |

### (B) Reasoned Inference — Derivation Logic

| Inference | Logic Chain |
|-----------|-------------|
| Yield DV01 $\neq$ curve DV01 for non-flat curves | Yield is an average of spot rates; parallel spot bump $\neq$ scalar yield bump |
| DV01 positive for long bonds under Tuckman sign | $dP/dy < 0$ for coupon bonds; minus sign in definition makes DV01 positive |
| PVBP = $NA \times 10^{-4}$ | From $\partial V_{\text{swap}}/\partial K = NA$ and 1bp = $10^{-4}$ |
| Portfolio PV01 is additive under consistent bump | Linearity of bump-and-reprice when same curve perturbation applies to all positions |
| Jacobian bridges quote and parameter sensitivities | Chain rule: $dV/dq = (\partial V/\partial\theta)(d\theta/dq)$ |

### (C) Speculation — Flagged Uncertainties

| Item | Uncertainty |
|------|-------------|
| Desk naming conventions for "PV01" | I'm not sure which convention your desk uses without a desk standard |

---

*Last Updated: January 2026*
