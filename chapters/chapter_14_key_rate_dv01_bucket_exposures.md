# Chapter 14: Key-Rate DV01 and Bucket Exposures (and Why Parallel DV01 = 0 ≠ No Risk)

---

## Fact Classification

### (A) Verified Facts (Source-Backed)
- DV01 definition: $\mathrm{DV01} \equiv -\frac{\Delta P}{10{,}000 \, \Delta y}$ (Tuckman Ch 5-6)
- DV01 can be defined for any measure of rates, not only yield-to-maturity (Tuckman Ch 5-6)
- Key-rate shifts: applying the technique requires choices of (i) number of keys, (ii) rate type (spot or par yields), (iii) key terms, and (iv) rule mapping key rates to all other rates (Tuckman Ch 7)
- Tuckman's triangular key-rate shifts: piecewise-linear between keys, 1 bp at own maturity declining to 0 at adjacent keys, constant beyond endpoints (Tuckman Ch 7, Figure 7.1)
- Sum of Tuckman's key-rate shifts equals a parallel shift (Tuckman Ch 7)
- Key-rate exposures "decompose a sensitivity measure like DV01 or duration into component sensitivities" (Tuckman Ch 7)
- Bucket exposures: PV changes from bumping individual forward rates/buckets by 1 bp (Tuckman Ch 7)
- Eurodollar futures: 1 bp move is worth $25 per contract, consistent with $1mm for 3 months (Hull Ch 6)
- GAP management: dividing the zero curve into buckets and investigating the effect of changing only the zero rates in one bucket (Hull Ch 4)
- A portfolio with total bucket exposure = 0 "can hardly be said to have no interest-rate risk"; it profits from flattening and loses from steepening (Tuckman Ch 7)
- With par-yield key rates and par bonds as hedges, "each hedging security has an exposure to one and only one key rate," and the key-rate exposure of the hedging security equals its DV01 (Tuckman Ch 7)
- Risk numbers depend on curve construction and interpolation; a 1 bp par-point shift can create different forward-curve perturbations depending on method (Tuckman, Andersen)
- Par-yield bumps can imply "bizarre" forward-curve changes; switching the object you bump (par vs forward) changes locality (Tuckman Ch 7)

### (B) Reasoned Inference (Derived from A)
- Under additive key-shift designs, $\sum_k \mathrm{KRDV01}_k \approx \mathrm{DV01}_{\parallel}$ (follows from linearity and the sum-to-parallel property)
- Scenario P&L approximation: $\Delta P \approx -\sum_k \mathrm{KRDV01}_k \, \Delta y_{k,\mathrm{bp}}$ (first-order Taylor expansion)
- Multi-instrument hedge construction as a linear system: $\min_n \|k + Hn\|$ (linear algebra from DV01 additivity)
- Key-rate hedges fail for large moves due to convexity mismatch (second-order effects not captured by first-order DV01)

### (C) Speculation (Clearly Labeled; Minimal)
- I'm not sure which exact bucket-definition convention your desk uses (e.g., bump spot rates, zero rates, instantaneous forwards; stepwise in time vs maturity; overlapping vs disjoint). The provided sources motivate bucketed analysis but do not fully standardize a single "piecewise-constant zero bump" implementation.
- I'm not sure about the exact fails-charge formula — the sources discuss fails conceptually but don't specify penalty mechanics.

---

## Conventions & Notation

### Defaults Used in This Chapter

| Convention | Default | Notes |
|------------|---------|-------|
| Rates and bp | 1 bp = 0.01% = 0.0001 in decimal | |
| DV01 sign (Tuckman) | Positive when price rises as rates decline | "Change in value for a 1 bp decline in rates" |
| For +1 bp increase | First-order value change $\approx -\mathrm{DV01}$ | |
| Curve object | Term structure $y(t)$ (par yields, spot/zero, or forward) | Must specify bump design |
| Payment frequency | Annual (didactic simplification) | Market conventions may differ |
| Base curve | Flat curve at 4% in many examples | Flatness not required; keeps arithmetic transparent |
| Price units | Per $100 face unless noted | Portfolio notionals explicitly stated |

### Notation Glossary

| Symbol | Meaning | Units |
|--------|---------|-------|
| $P$ | Present value (price) of an instrument | Dollars per 100 face |
| $y$ | Rate level used in DV01 definition | Decimal |
| $\Delta y$ | Rate change in decimal | Decimal |
| $\Delta y_{\mathrm{bp}}$ | Rate change in basis points; $\Delta y_{\mathrm{bp}} = 10{,}000 \, \Delta y$ | bp |
| $\mathrm{DV01}$ | Dollar value of a 1 bp decline in the chosen rate measure | Dollars per bp |
| $\{T_k\}$ | Key maturities | Years |
| $w_k(t)$ | Bump weight at maturity $t$ for a 1 bp move in key rate $T_k$ | Dimensionless |
| $\mathrm{KR01}_k$ / $\mathrm{KRDV01}_k$ | Price sensitivity to $k$-th key-rate shift | Dollars per bp |
| $B_j$ | Bucket exposure: PV change per 1 bp move in forward-rate bucket $j$ | Dollars per bp |
| $DF(t)$ | Discount factor to time $t$ | Dimensionless |
| $CF_t$ | Cash flow at time $t$ | Dollars |
| $A$ | Annuity factor $\sum_t DF(t)$ | Dimensionless |
| $K$ | Fixed rate (e.g., swap rate) | Decimal |
| $N$ | Notional | Dollars |

---

## Core Concepts

### 1) DV01 (General Definition)

**Formal Definition:**

Let $\Delta P$ be the change in value when a chosen rate measure changes by $\Delta y$ (decimal). Tuckman defines:

$$\boxed{\mathrm{DV01} \equiv -\frac{\Delta P}{10{,}000 \times \Delta y}}$$

**Intuition:**

DV01 rescales a price change into "dollars per bp." The negative sign makes DV01 positive for typical fixed-coupon exposure (rates down → price up).

**Trading / Risk / Portfolio Practice:**

- DV01 is a first-order hedge metric: it tells you how much you gain/lose for a small parallel move in whatever "rate" you chose
- Tuckman warns that yield-based DV01 only hedges as intended if the yield of what you bought moves in line with the yield of what you sold (the "parallel shift" requirement)
- "DV01" in the market often implicitly means yield-based DV01; Tuckman notes alternative DV01 labels exist depending on which rate measure is bumped (forward vs spot/zero)

---

### 2) Key Rates, Key-Rate Shifts, and Key-Rate DV01 (KRDV01 / KR01)

**Formal Definition (Key Rates):**

Choose key maturities $\{T_1, \ldots, T_K\}$ and a rule that determines the full curve given the key rates. Tuckman: applying the technique requires choices of (i) number of keys, (ii) rate type (spot or par yields), (iii) key terms, and (iv) rule mapping key rates to all other rates.

**Formal Definition (Triangular Key-Rate Shifts, Tuckman's Design):**

With key par yields at (for example) 2y, 5y, 10y, 30y, Tuckman assumes each key-rate move produces a piecewise-linear (triangular) impact on nearby maturities:
- 1 bp at its own maturity, declining linearly to 0 at adjacent key maturities
- Remaining 1 bp beyond the endpoints (left of first key, right of last key)

He also notes the exact shape is not essential; the arbitrariness of the shift shape is a theoretical weakness.

**Formal Definition (KRDV01 / KR01):**

Let $P_0$ be the base PV and $P^{(k)}$ the PV after applying only the $k$-th key-rate shift of +1 bp (per the chosen bump design), with all else held fixed. Define:

$$\boxed{\mathrm{KR01}_k \equiv P_0 - P^{(k)}}$$

This matches the sign convention in Tuckman's Table 7.1 where "after key-rate shift" PV drops and KR01 is reported positive as the PV loss for +1 bp.

**Intuition:**

KRDV01 tells you where along the curve the DV01 "lives." It decomposes a one-number DV01 into a vector of exposures by maturity region.

**Trading / Risk / Portfolio Practice:**

- Used to hedge curve shape risk (steepeners/flatteners/twists) by matching multiple bucket/key sensitivities rather than only total DV01
- Tuckman: key-rate exposures "decompose a sensitivity measure like DV01 or duration into component sensitivities"
- With Tuckman's shift construction, the sum of key-rate shifts is a parallel shift, so the sum of KR01s closely matches DV01 under that parallel-shift assumption

---

### 3) Bucket Exposures (Forward-Rate Bucket DV01s)

**Formal Definition:**

Pick a discrete set of forward-rate buckets (e.g., the 6-month forward starting 2.5 years forward). Bump only that forward by +1 bp, keep all other forwards unchanged, rebuild the implied spot/discount curve, reprice, and record PV change. Tuckman illustrates: a +1 bp increase in a specific forward "lowers the value" of a $100mm swap by about $4,200.

**Intuition:**

Bucket exposures answer: "If one particular forward fixing moved 1 bp (holding other forwards fixed), what happens to my PV?"

**Trading / Risk / Portfolio Practice:**

- Tuckman motivates bucket exposures because Eurodollar futures can hedge forward-rate risk directly, especially at short maturities
- Hull describes "GAP management" as dividing the zero curve into buckets and investigating the effect of changing only the zero rates in one bucket

---

### 4) Why "Parallel DV01 = 0" Does Not Mean "No Risk"

**Formal Statement:**

A portfolio can be constructed so that its PV change under a parallel shift of the chosen curve representation is ~0. This does not imply the PV change under non-parallel (twist/steepener/flattening) scenarios is ~0.

**Intuition:**

Parallel DV01 is one projection of risk onto one direction in a high-dimensional space of curve moves.

**Trading / Risk / Portfolio Practice (Tuckman's Explicit Warning):**

In bucket-exposure terms, Tuckman gives an example portfolio that has total bucket exposure = 0 (so no risk to parallel shifts) but still has meaningful positive exposures in some forward buckets and negative exposures in others—so it profits from flattening and loses from steepening.

---

## Math and Derivations

### 1) DV01 as a Finite Difference and as a Derivative

Tuckman defines DV01 via finite differences:

$$\boxed{\mathrm{DV01} \equiv -\frac{\Delta P}{10{,}000 \, \Delta y}}$$

In the differentiable limit, he writes:

$$\boxed{\mathrm{DV01} = -\frac{1}{10{,}000}\frac{dP(y)}{dy}}$$

**Unit Check:**
- $P$ in dollars per $100 face (or total $), $y$ in decimal
- $dP/dy$ has units "dollars per 1.00 (100%) rate." Multiplying by $1/10{,}000$ converts to "dollars per bp"

**Sanity Check:**
- For a plain fixed-coupon bond, $dP/dy < 0$ so DV01 $> 0$ (rates down → PV up)

---

### 2) Key-Rate Shifts (Tuckman) and the Decomposition Property

Tuckman's key-rate shift construction (illustrated for 2y/5y/10y/30y par yields):
- Each key affects par yields between adjacent keys, peaking at 1 bp at its own maturity and linearly declining to 0 at neighboring keys; outside endpoints it stays at 1 bp
- Importantly, "the sum of the key rate shifts equals a parallel shift in the par yield curve"

Let $w_k(t)$ be the bump weight function for key rate $k$. Under this construction:

$$\boxed{\sum_{k=1}^{K} w_k(t) = 1 \quad \text{(for all } t \text{ on the curve)}}$$

If $P$ is (approximately) linear in small bumps, then the PV change under a parallel shift is the sum of PV changes under the individual key-rate shifts:

$$\boxed{\Delta P_{\text{parallel}} \approx \sum_{k=1}^{K} \Delta P_k \quad \Rightarrow \quad \mathrm{DV01}_{\text{parallel}} \approx \sum_{k=1}^{K} \mathrm{KR01}_k}$$

This is the conceptual reason behind Tuckman's statement that the sums of key-rate 01s "closely match" DV01 under the parallel-shift assumption.

**Assumptions:**
- Small shocks (first-order linearization)
- A consistent "rate measure" and bump design across DV01 and KR01 calculations

---

### 3) Bucket Exposures and Why "Sum of Buckets ≈ DV01"

Tuckman defines bucket exposures by bumping one forward and holding other forwards fixed, then repricing. He notes: for a flat swap-rate curve, the sum of forward bucket exposures equals the DV01 of the fixed side exactly; otherwise it is typically close.

**Interpretation:**

Bucket exposures are a different basis (forward buckets vs key par-yield maturities). They should approximately reconcile to DV01 when the underlying set of bucket moves spans a parallel move in the chosen curve representation.

---

### 4) Hedge Construction as a Linear System

Let $k \in \mathbb{R}^K$ be the KRDV01 vector of the position (in $ per bp). Let $H \in \mathbb{R}^{K \times M}$ collect the KRDV01 vectors of $M$ hedge instruments (columns), expressed per unit notional of each hedge.

We choose hedge notionals $n \in \mathbb{R}^M$ to minimize residual exposure:

$$\boxed{\min_n \|k + Hn\|^2}$$

If $K = M$ and $H$ is invertible, the exact match is $n = -H^{-1}k$.

If $K > M$ (more buckets than hedges), we typically use least squares.

**Desk Interpretation:**

This is exactly how multi-bucket hedges are built in risk systems: exposures are treated as locally linear and notionals are solved to "zero" key-rate/bucket risk.

---

## Measurement & Risk

### 1) Precise Definition: KRDV01

**Definition (Bump-and-Reprice):**

Pick:
- Key maturities $\{T_k\}$
- A curve object to bump (par yield curve, spot/zero curve, or forward curve)
- A bump design $w_k(t)$ that specifies the full-curve perturbation implied by a +1 bp move at key $T_k$

Then:

$$\boxed{\mathrm{KRDV01}_k \equiv \mathrm{KR01}_k \equiv P_0 - P^{(k)}}$$

where $P^{(k)}$ is the PV under the curve perturbed by +1 bp according to $w_k$. This matches Tuckman's "key rate 01s" (key-rate equivalent of DV01).

---

### 2) Bucket Exposure vs Parallel DV01

- **Parallel DV01:** PV change for a simultaneous 1 bp move in the chosen "rate" at all maturities (one direction in curve space)
- **Bucket exposure / KRDV01:** PV change for a localized move in a region of the curve

**Why Parallel DV01 Can Be ~0 While Twist Risk Is Large:**

A portfolio can be set so that positive and negative bucket exposures sum to 0. Tuckman's example: a portfolio with total bucket exposure = 0 "can hardly be said" to have no interest-rate risk; it profits from flattening and loses from steepening.

---

### 3) Practical Bump Designs (and What the Sources Do/Do Not Specify)

#### (i) Triangular "Localized" Key-Rate Shifts (Tuckman Default)

**Source-backed design:** Piecewise-linear impacts between adjacent key maturities; constant beyond endpoints; sum of shifts equals a parallel shift.

- **Pro:** Smooth-ish, local, and additive (good decomposition property)
- **Con:** Bump shape is somewhat arbitrary (Tuckman calls it a theoretical weakness)

#### (ii) Piecewise-Constant Zero-Rate Bumps by Bucket

I'm not sure which exact bucket-definition convention your desk uses (e.g., bump spot rates, zero rates, instantaneous forwards; stepwise in time vs maturity; overlapping vs disjoint). The provided sources motivate bucketed analysis (e.g., Hull's "GAP management" buckets) but do not fully standardize a single "piecewise-constant zero bump" implementation.

**Two common alternatives (clearly labeled):**
- **Disjoint buckets:** Bump only rates for maturities $t \in (T_{j-1}, T_j]$ by +1 bp
- **Overlapping buckets:** Bump a window around each key (e.g., $[T_k - \Delta, T_k + \Delta]$) by +1 bp with overlaps

#### (iii) Quote-Bump / Rebuild-Curve (Preview)

**What the sources emphasize:** Risk numbers depend on curve construction and interpolation; a 1 bp par-point shift can create different forward-curve perturbations depending on method, and "tension" can dampen noise in risk reports.

**Preview definition:** Bump a market quote (e.g., a 2y swap rate), rebuild the full curve using your curve-building algorithm, then reprice and take PV differences.

**Desk relevance:** If your risk report uses quote-bump sensitivities but your hedge ratios assume a different bump (e.g., direct zero-rate bump), hedges can systematically miss.

---

### 4) KRDV01 and Scenario Analysis (Steepeners/Flatteners)

Once you have a KRDV01 vector $k$, a small scenario defined as key-rate moves $\Delta y_{\mathrm{bp}}$ gives a first-order PV estimate:

$$\boxed{\Delta P \approx -\sum_{k=1}^{K} \mathrm{KRDV01}_k \, \Delta y_{k,\mathrm{bp}}}$$

(Negative because KRDV01 is "PV up for rates down," per Tuckman's DV01 sign convention.)

---

### 5) Hedging with Multiple Instruments

Use multiple hedges to match multiple buckets/keys (linear system in §2.4).

Tuckman highlights a particularly convenient setup: with par-yield key rates and par bonds as hedges, "each hedging security has an exposure to one and only one key rate," and the key-rate exposure of the hedging security equals its DV01.

---

### 6) When Key-Rate Hedges Fail (What to Watch)

- **Convexity mismatch:** First-order (DV01/KRDV01) hedges do not neutralize second-order effects; large moves produce residual P&L (Example G)
- **Spread/basis risk:** Hedging corporates with swaps/futures introduces spread and liquidity basis exposure (curve moves may differ across Treasury/swap/corporate curves)

(We keep this a short warning here; a full spread/basis chapter is separate.)

- **Multi-curve effects (preview only):** If valuation uses different curves for discounting vs projection, "rate risk" splits into discount-curve risk and forward/projection-curve risk; a single KRDV01 vector may not capture both

---

## Worked Examples

### Common Conventions for Examples A–I (Unless Overridden)

- Prices per $100 face unless stated
- Base curve: flat annual-compounded spot curve at $y = 4\%$ (didactic)
- DV01 sign follows Tuckman: DV01 is PV change for a 1 bp decline in the bumped rate measure
- We compute sensitivities by bump-and-reprice logic, but use the derivative approximation consistent with Tuckman's derivative DV01 definition

---

### Example A — Parallel DV01 vs KRDV01 Decomposition (Single Fixed-Rate Bond)

**Instrument:** 10-year bond, annual coupons, 5% coupon, $100 face.

**Cash flows:**
- $CF_t = 5$ for $t = 1, \ldots, 9$
- $CF_{10} = 105$

**Step A1: Discount Factors Under Flat 4%**

$$DF(t) = (1.04)^{-t}$$

Compute:
- $DF(1) = 0.961538$
- $DF(2) = 0.924556$
- $DF(3) = 0.889996$
- ...
- $DF(10) = 0.676325$

**Step A2: Local DV01 Contribution Per Cash Flow Date**

Using Tuckman's derivative-style DV01 idea applied to a rate $y(t)$:

$$\mathrm{DV01}_t \approx -\frac{1}{10{,}000}\frac{\partial P}{\partial y(t)}$$

For $DF(t) = (1+y)^{-t}$, $\frac{\partial DF}{\partial y} = -\frac{t \, DF(t)}{1+y}$. Hence:

$$\mathrm{DV01}_t = \frac{CF_t \cdot t \cdot DF(t)}{(1+y)} \cdot \frac{1}{10{,}000}$$

Here $1+y = 1.04$.

**Compute $\mathrm{DV01}_t$ (per $100 face):**

| $t$ | $\mathrm{DV01}_t$ |
|-----|-------------------|
| 1 | 0.000462278 |
| 2 | 0.000889996 |
| 3 | 0.001283646 |
| 4 | 0.001645706 |
| 5 | 0.0019780125 |
| 6 | 0.002282322 |
| 7 | 0.0025602955 |
| 8 | 0.002813512 |
| 9 | 0.0030434625 |
| 10 | 0.06828276 |

**Step A3: Parallel DV01 (Spot/Zero Curve +1 bp Everywhere)**

$$\mathrm{DV01}_{\parallel} = \sum_{t=1}^{10} \mathrm{DV01}_t = 0.085241992 \text{ (price dollars per bp, per \$100 face)}$$

**Unit Check:**
- $0.08524$ dollars per bp = 8.524 cents per bp per $100 face
- Per $1mm face: $0.08524 \times 10{,}000 = \$852.42$ per bp

**Step A4: Define Key Rates and Tuckman-Style Triangular Shifts**

Choose key maturities: $\{2, 3, 5, 7, 10\}$ years.

Define weights $w_k(t)$ piecewise-linear between adjacent keys, with endpoint behavior as in Tuckman's Figure 7.1 description.

**Weights at Integer Cashflow Times:**

| $t$ | $w_2(t)$ | $w_3(t)$ | $w_5(t)$ | $w_7(t)$ | $w_{10}(t)$ |
|-----|----------|----------|----------|----------|-------------|
| 1 | 1 | 0 | 0 | 0 | 0 |
| 2 | 1 | 0 | 0 | 0 | 0 |
| 3 | 0 | 1 | 0 | 0 | 0 |
| 4 | 0 | 0.5 | 0.5 | 0 | 0 |
| 5 | 0 | 0 | 1 | 0 | 0 |
| 6 | 0 | 0 | 0.5 | 0.5 | 0 |
| 7 | 0 | 0 | 0 | 1 | 0 |
| 8 | 0 | 0 | 0 | 2/3 | 1/3 |
| 9 | 0 | 0 | 0 | 1/3 | 2/3 |
| 10 | 0 | 0 | 0 | 0 | 1 |

**Step A5: Compute KRDV01 by Weighting Cashflow DV01s**

$$\mathrm{KRDV01}_k = \sum_t \mathrm{DV01}_t \, w_k(t)$$

Compute:
- $\mathrm{KRDV01}_2 = \mathrm{DV01}_1 + \mathrm{DV01}_2 = 0.001352274$
- $\mathrm{KRDV01}_3 = \mathrm{DV01}_3 + 0.5 \, \mathrm{DV01}_4 = 0.002106499$
- $\mathrm{KRDV01}_5 = 0.5 \, \mathrm{DV01}_4 + \mathrm{DV01}_5 + 0.5 \, \mathrm{DV01}_6 = 0.0039420265$
- $\mathrm{KRDV01}_7 = 0.5 \, \mathrm{DV01}_6 + \mathrm{DV01}_7 + \frac{2}{3}\mathrm{DV01}_8 + \frac{1}{3}\mathrm{DV01}_9 = 0.0065916187$
- $\mathrm{KRDV01}_{10} = \frac{1}{3}\mathrm{DV01}_8 + \frac{2}{3}\mathrm{DV01}_9 + \mathrm{DV01}_{10} = 0.0712495723$

**Step A6: Check Decomposition**

$$\sum_k \mathrm{KRDV01}_k = 0.0852419905 \approx 0.085241992 = \mathrm{DV01}_{\parallel}$$

This matches the decomposition logic emphasized by Tuckman when the sum of key-rate shifts forms a parallel shift.

**Residual Discussion:**

Here the residual is rounding-level. In real systems, residuals appear from:
- Different interpolation choices
- Nonlinear repricing (finite differences)
- Curve rebuild artifacts

---

### Example B — "DV01-Neutral but Risky" Portfolio (Twist P&L)

**Goal:** Construct a 2-instrument portfolio with net parallel DV01 ≈ 0, but large P&L under a twist.

**Instrument 1 (long):** The 10y 5% coupon bond from Example A.
- $\mathrm{DV01}_{\parallel} = 0.085241992$ per $100 face
- KRDV01 vector (keys 2/3/5/7/10): $[0.0013523, 0.0021065, 0.0039420, 0.0065916, 0.0712496]$

**Instrument 2 (short):** 2-year zero-coupon bond, $100 face at $t = 2$.
- $P = 100(1.04)^{-2} = 92.4556$
- $\mathrm{DV01}_{\parallel} = 100 \cdot 2 \cdot \frac{DF(2)}{1.04} \cdot 10^{-4} = 0.01779992$
- KRDV01 vector: $[0.0177999, 0, 0, 0, 0]$

**Step B1: Choose Notionals to Make Parallel DV01 ≈ 0**

Let portfolio be:
- Long 1 unit of Bond 1 (per $100 face)
- Short $n$ units of the 2y zero

Solve:

$$0.085241992 - n \cdot 0.01779992 \approx 0 \quad \Rightarrow \quad n \approx \frac{0.085241992}{0.01779992} \approx 4.789$$

**Step B2: Confirm Net Parallel DV01**

$$\mathrm{DV01}_{\text{port}} \approx 0.085241992 - 4.789 \cdot 0.01779992 \approx 0$$

**Step B3: Define a Twist Scenario**

- +10 bp at 2y
- −10 bp at 10y
- Linearly interpolated in between (so short end up, long end down)

Key-rate moves (bp) at $\{2, 3, 5, 7, 10\}$:
- $\Delta y_2 = +10$
- $\Delta y_3 = +7.5$
- $\Delta y_5 = +2.5$
- $\Delta y_7 = -2.5$
- $\Delta y_{10} = -10$

**Step B4: Compute Portfolio KRDV01 Vector**

Portfolio KRDV01 (Bond1 minus $4.789 \times$ Zero2y):
- 2y: $0.0013523 - 4.789(0.0177999) \approx -0.0838917$
- 3y: $0.0021065$
- 5y: $0.0039420$
- 7y: $0.0065916$
- 10y: $0.0712496$

Check sum:

$$-0.0838917 + 0.0021065 + 0.0039420 + 0.0065916 + 0.0712496 \approx 0$$

So parallel DV01 ≈ 0 but the shape exposures are large and offsetting.

**Step B5: Twist P&L Using Key-Rate Approximation**

$$\Delta P \approx -\sum_k \mathrm{KRDV01}_k \, \Delta y_{k,\mathrm{bp}}$$

Compute:
- Contribution from 2y: $-(-0.0838917) \cdot 10 = +0.838917$
- 3y: $-(0.0021065) \cdot 7.5 = -0.015799$
- 5y: $-(0.0039420) \cdot 2.5 = -0.009855$
- 7y: $-(0.0065916) \cdot (-2.5) = +0.016479$
- 10y: $-(0.0712496) \cdot (-10) = +0.712496$

Sum:

$$\Delta P \approx 1.54224 \text{ (per \$100-face unit of Bond1)}$$

**Interpretation:**

The portfolio is "DV01-neutral" (parallel) but has large twist exposure. This is exactly the conceptual warning Tuckman makes in bucket form: "no risk to parallel shifts" does not mean "no interest rate risk."

---

### Example C — Key-Rate DV01 for a Swap (Shape vs Bond)

We compute KRDV01 for a par 5-year receive-fixed swap and compare the exposure shape to a bond.

**Assumptions:**
- Annual payments for simplicity
- Single-curve (discount = projection) to stay within scope
- Notional $N = \$100{,}000{,}000$
- Flat curve $y = 4\%$, so $DF(t) = (1.04)^{-t}$

**Swap PV Formula (Receive Fixed):**

Use the standard single-curve identity:

$$PV = N\left(K \sum_{t=1}^{T} DF(t) - (1 - DF(T))\right)$$

where $K$ is the fixed rate set to make $PV = 0$ initially:

$$K = \frac{1 - DF(T)}{\sum_{t=1}^{T} DF(t)}$$

**Step C1: Compute the Annuity**

For $T = 5$:

$$A = \sum_{t=1}^{5} DF(t) = 0.961538 + 0.924556 + 0.889996 + 0.855764 + 0.822853 = 4.454707$$

$DF(5) = 0.822853 \Rightarrow 1 - DF(5) = 0.177147$

So par swap rate:

$$K = \frac{0.177147}{4.454707} = 0.03977 \; (\approx 3.977\%)$$

**Step C2: Parallel DV01 of the Swap (Bump-and-Reprice First Order)**

Differentiate PV with respect to a parallel shift $y$ (holding $K$ fixed once set):

$$\frac{dPV}{dy} = N\left(K \sum_{t=1}^{5} \frac{dDF(t)}{dy} + \frac{dDF(5)}{dy}\right)$$

With $\frac{dDF(t)}{dy} = -\frac{t \, DF(t)}{1+y}$:

$$\mathrm{DV01}_{\parallel} = -\frac{1}{10{,}000}\frac{dPV}{dy} = N \cdot \frac{K\sum_{t=1}^{5} t \, DF(t) + 5 \, DF(5)}{(1+y)} \cdot 10^{-4}$$

Compute $\sum t \, DF(t)$:
- $1 \cdot DF(1) = 0.961538$
- $2 \cdot DF(2) = 1.849112$
- $3 \cdot DF(3) = 2.669988$
- $4 \cdot DF(4) = 3.423056$
- $5 \cdot DF(5) = 4.114265$

Sum $= 13.017959$.

Now:
- $K \sum t \, DF(t) = 0.03977 \times 13.017959 \approx 0.5177$
- $5 \, DF(5) = 4.114265$
- Total $\approx 4.631965$

So:

$$\mathrm{DV01}_{\parallel} \approx 100{,}000{,}000 \cdot \frac{4.631965}{1.04} \cdot 10^{-4} \approx \$44{,}520 \text{ per bp}$$

**Step C3: KRDV01 by Maturity (Simple "Payment-Date" Buckets)**

Because PV depends on discount factors at each payment date, we can allocate DV01 by the maturity where $DF(t)$ is bumped (a "bucket" view consistent with curve-bump logic).

For $t = 1, \ldots, 4$, PV depends on $DF(t)$ only through the fixed leg:

$$\mathrm{DV01}_t = N \cdot \frac{K \cdot t \, DF(t)}{1+y} \cdot 10^{-4}$$

For $t = 5$, PV depends on $DF(5)$ through the fixed coupon and through the floating leg term $+DF(5)$:

$$\mathrm{DV01}_5 = N \cdot \frac{(K+1) \cdot 5 \, DF(5)}{1+y} \cdot 10^{-4}$$

Compute a common factor for $t \leq 4$:

$$F = N \cdot \frac{K}{1+y} \cdot 10^{-4} = 100{,}000{,}000 \cdot \frac{0.03977}{1.04} \cdot 10^{-4} \approx 382.4$$

Then $\mathrm{DV01}_t = F \cdot t \, DF(t)$:
- $t = 1$: $382.4 \cdot 0.961538 \approx \$368$
- $t = 2$: $382.4 \cdot 1.849112 \approx \$708$
- $t = 3$: $382.4 \cdot 2.669988 \approx \$1{,}021$
- $t = 4$: $382.4 \cdot 3.423056 \approx \$1{,}309$

For $t = 5$:

$$F_5 = N \cdot \frac{K+1}{1+y} \cdot 10^{-4} = 100{,}000{,}000 \cdot \frac{1.03977}{1.04} \cdot 10^{-4} \approx 9{,}997.8$$

So:

$$\mathrm{DV01}_5 \approx 9{,}997.8 \cdot 4.114265 \approx \$41{,}134$$

**Interpretation:**

The swap's DV01 is heavily concentrated near maturity (here at 5y) because the floating leg behaves like a par instrument with principal sensitivity represented via $DF(T)$. Compared with the bond in Example A (whose sensitivity is spread across coupon dates and the maturity), the swap has a distinctly different key/bucket profile—important when hedging curve shape risk.

---

### Example D — Bucket Hedge via Linear System (Par-Yield Key Rates; Diagonal Hedge)

This example follows Tuckman's "par-yield key rate" logic where par bonds at key maturities are natural hedges.

**Source-Backed Setup:**

Tuckman computes key-rate 01s for a 30-year nonprepayable mortgage (Table 7.1) and shows key-rate shifts that sum to a parallel shift. He also notes: "each hedging security has an exposure to one and only one key rate," and if the hedging security is a par bond, its key-rate exposure equals its DV01.

**Target Exposures (from Tuckman Table 7.1):**

Key-rate 01s (in $) for the mortgage (par curve flat at 5%):
- 2y: $0.98$
- 5y: $3.77$
- 10y: $42.37$
- 30y: $67.26$

**Hedge Instruments:**

Use par bonds at 2y, 5y, 10y, 30y (same par-yield curve). Tuckman provides the DV01 per $100 par bond at 5% (semiannual) used in the simplified hedging equations:
- 2y: 0.01881
- 5y: 0.04375
- 10y: 0.08308
- 30y: 0.15444

**Step D1: Write the Linear System**

Let $F_2, F_5, F_{10}, F_{30}$ be face amounts (in $) of the hedging par bonds to sell (short) to hedge the mortgage exposures.

The hedge condition (match each key-rate 01) is:

$$\frac{0.01881}{100}F_2 = 0.98129, \quad \frac{0.04375}{100}F_5 = 3.77314$$
$$\frac{0.08308}{100}F_{10} = 42.36832, \quad \frac{0.15444}{100}F_{30} = 67.25637$$

which is exactly the simplified diagonal form Tuckman presents when the hedging bonds are assumed to sell at par.

**Step D2: Solve (Diagonal)**

$$F_2 = \frac{0.98129 \cdot 100}{0.01881} \approx 5{,}217$$
$$F_5 = \frac{3.77314 \cdot 100}{0.04375} \approx 8{,}624$$
$$F_{10} = \frac{42.36832 \cdot 100}{0.08308} \approx 50{,}995$$
$$F_{30} = \frac{67.25637 \cdot 100}{0.15444} \approx 43{,}550$$

**Matrix View:**

$$\underbrace{\begin{bmatrix} 0.01881/100 & 0 & 0 & 0 \\ 0 & 0.04375/100 & 0 & 0 \\ 0 & 0 & 0.08308/100 & 0 \\ 0 & 0 & 0 & 0.15444/100 \end{bmatrix}}_{H} \underbrace{\begin{bmatrix} F_2 \\ F_5 \\ F_{10} \\ F_{30} \end{bmatrix}}_{n} = \underbrace{\begin{bmatrix} 0.98129 \\ 3.77314 \\ 42.36832 \\ 67.25637 \end{bmatrix}}_{k}$$

**Desk Interpretation:**

This is a "perfect" key-rate hedge in the chosen key-rate basis. It will still not hedge:
- Non-modeled curve shapes (between keys)
- Large moves (convexity)
- Basis/spread effects if hedges trade on different curves

---

### Example E — Hedging Short-End Buckets with STIR/Eurodollar Futures (Quarterly Buckets to 2y)

We illustrate forward-rate bucket hedging with a strip of Eurodollar futures, consistent with:
- Tuckman: Eurodollar futures hedge forward-rate risk and bucket exposures can be computed to forward rates
- Hull: Eurodollar futures are designed so that a 1 bp move is worth $25 per contract, consistent with $1mm for 3 months

**Position:** Pay 3M floating rate on $100mm for each quarter over the next 2 years (8 quarters). We want bucket DV01s by quarter and hedge each with Eurodollar futures.

**Step E1: Define Buckets**

Quarter $i$ has payment at $t_i = 0.25i$ years. The interest paid is approximately:

$$\text{Interest}_i \approx N \cdot \alpha \cdot L_i, \quad \alpha = 0.25$$

A +1 bp increase in the forward rate $L_i$ increases the payment by:

$$\Delta \text{Interest}_i = N \cdot \alpha \cdot 0.0001$$

Discounting (simple preview): multiply by $DF(t_i)$.

So the bucket DV01 (PV loss per +1 bp) is approximately:

$$B_i \approx -\frac{N \alpha \, DF(t_i)}{10{,}000}$$

Magnitude:

$$|B_i| = \frac{100{,}000{,}000 \cdot 0.25}{10{,}000} \, DF(t_i) = 2{,}500 \cdot DF(t_i)$$

**Step E2: Compute Discount Factors and Bucket DV01s**

Use flat $y = 4\%$ and annual comp $DF(t) = (1.04)^{-t}$. Approximate for quarter steps:

| Quarter $i$ | $t_i$ (yrs) | $DF(t_i)$ (approx) | Bucket DV01 $|B_i|$ ($/bp) |
|-------------|-------------|--------------------|-----------------------------|
| 1 | 0.25 | 0.990 | 2,475 |
| 2 | 0.50 | 0.981 | 2,453 |
| 3 | 0.75 | 0.971 | 2,428 |
| 4 | 1.00 | 0.962 | 2,404 |
| 5 | 1.25 | 0.952 | 2,380 |
| 6 | 1.50 | 0.943 | 2,358 |
| 7 | 1.75 | 0.934 | 2,334 |
| 8 | 2.00 | 0.925 | 2,311 |

(Sign: since we pay floating, the exposure is negative; the table reports magnitudes.)

**Step E3: Map Bucket DV01 to Eurodollar Futures Contracts**

Hull: 1 bp move in Eurodollar futures quote corresponds to $25 per contract.

So approximate contracts needed per bucket:

$$N_i \approx \frac{|B_i|}{25} = 100 \cdot DF(t_i)$$

| Quarter $i$ | $|B_i|$ ($/bp) | Contracts $N_i \approx |B_i|/25$ |
|-------------|----------------|-----------------------------------|
| 1 | 2,475 | 99 |
| 2 | 2,453 | 98 |
| 3 | 2,428 | 97 |
| 4 | 2,404 | 96 |
| 5 | 2,380 | 95 |
| 6 | 2,358 | 94 |
| 7 | 2,334 | 93 |
| 8 | 2,311 | 92 |

**Hedge Direction:**

Paying floating loses when rates rise, so you want a hedge that gains when rates rise.

Hull: Eurodollar futures quote is $100 - \text{rate}$, so a rise in rates lowers the futures quote; a trader short futures gains when the quote falls.

So this bucket hedge corresponds to shorting the appropriate Eurodollar futures in each relevant contract month.

---

### Example F — Mapping DV01 to "bp Value" Per Contract (Single Futures DV01)

We compute the "DV01 per contract" and scale to a hedge.

**Eurodollar Futures DV01 Per Contract:**

Hull's derivation: a 1 bp change in the futures quote corresponds to a 1 bp change in the underlying rate and a $25 change per contract, consistent with $1mm for 3 months:

$$1{,}000{,}000 \times 0.0001 \times 0.25 = 25$$

So:
- **PV01/DV01 per contract:** $25 per bp (ignoring convexity adjustment and discounting)

**Example Hedge Calculation:**

Suppose you need to hedge a short-end bucket exposure of $12,500 per bp.

Then required contracts:

$$N \approx \frac{12{,}500}{25} = 500 \text{ contracts}$$

**Sign Check:**

If your position loses $12,500 per bp when rates rise, you need a hedge that gains $12,500 per bp when rates rise → short Eurodollar futures (rate up → futures quote down → short gains).

---

### Example G — Key-Rate Hedging Error from Convexity (±100 bp Move)

We build a portfolio that is DV01- and key-rate-hedged (for a coarse key set) and show residual P&L for large moves due to convexity mismatch.

**Instruments (per $100 face):**
- **Bond L (long):** 10y, 5% coupon, annual pay
- **Zero Z2 (short):** 2y zero
- **Zero Z10 (short):** 10y zero

**Base curve:** flat $y = 4\%$ annual.

**Step G1: Prices at y = 4%**

From Example A:
- $P_L(4\%) = 108.2220$

Zeros:
- $P_{Z2}(4\%) = 100(1.04)^{-2} = 92.4556$
- $P_{Z10}(4\%) = 100(1.04)^{-10} = 67.6325$

**Step G2: Define a 2-Key Key-Rate System $\{2, 10\}$**

We hedge only 2y and 10y key-rate exposures (coarse but illustrative).

Compute KRDV01 of Bond L under 2-key triangular weights (details in analysis; summary results):
- $\mathrm{KRDV01}_{L,2} \approx 0.0081311$
- $\mathrm{KRDV01}_{L,10} \approx 0.0771109$

(and the sum is the bond's total DV01 $0.085242$).

Key-rate DV01s of hedges:
- Z2: $\mathrm{KRDV01}_{Z2,2} = 0.0177999$, $\mathrm{KRDV01}_{Z2,10} = 0$
- Z10: $\mathrm{KRDV01}_{Z10,10} = 0.0650312$, $\mathrm{KRDV01}_{Z10,2} = 0$

**Step G3: Solve Hedge Notionals for KRDV01-Neutrality**

Let portfolio be:

$$\text{Portfolio} = 1 \cdot L - a \cdot Z2 - b \cdot Z10$$

Solve:

$$0.0081311 - a(0.0177999) = 0 \quad \Rightarrow \quad a = 0.4568$$
$$0.0771109 - b(0.0650312) = 0 \quad \Rightarrow \quad b = 1.186$$

So the DV01- and key-rate-hedged portfolio (in this 2-key basis) is:
- Long 1× Bond L
- Short 0.4568× Z2
- Short 1.186× Z10

**Step G4: Reprice Under Large Parallel Shifts (±100 bp)**

Compute prices at $y = 3\%$ and $y = 5\%$ (exact repricing):

Bond L:
- $P_L(3\%) = 117.0604$
- $P_L(5\%) \approx 100.0000$

Zeros:
- $P_{Z2}(3\%) = 100(1.03)^{-2} = 94.2596$
- $P_{Z2}(5\%) = 100(1.05)^{-2} = 90.7029$
- $P_{Z10}(3\%) = 100(1.03)^{-10} = 74.4094$
- $P_{Z10}(5\%) = 100(1.05)^{-10} = 61.3913$

**Step G5: Compute Portfolio Value Changes**

Base (4%):

$$V(4\%) = 108.2220 - 0.4568(92.4556) - 1.186(67.6325) \approx -14.2239$$

At 3%:

$$V(3\%) = 117.0604 - 0.4568(94.2596) - 1.186(74.4094) \approx -14.2470$$

So:

$$\Delta V_{-100\text{bp}} = V(3\%) - V(4\%) \approx -0.0231 \text{ per \$100 face}$$

At 5%:

$$V(5\%) = 100.0000 - 0.4568(90.7029) - 1.186(61.3913) \approx -14.2431$$

So:

$$\Delta V_{+100\text{bp}} = V(5\%) - V(4\%) \approx -0.0192 \text{ per \$100 face}$$

**Interpretation:**

First-order key-rate/DV01 hedging predicts near-zero change for small moves, but for ±100 bp the residual is non-zero due to convexity mismatch.

Scaling: if the "1 unit" Bond L position represents $100mm face, then $0.02$ price points is about $20,000 residual P&L.

---

### Example H — Curve Construction Dependency Preview (Bump Design Changes the Numbers)

We compute KRDV01-like allocations under two bump designs and show they differ.

**Instrument:** Bond L from Example A.

**Design 1 (Tuckman Triangular Key-Rate Shifts on Key Maturities):**

Keys: $\{2, 3, 5, 7, 10\}$

KRDV01 (per $100 face):
- 2y: 0.001352
- 3y: 0.002106
- 5y: 0.003942
- 7y: 0.006592
- 10y: 0.071250

Sum: 0.085242 (matches parallel DV01)

This aligns with Tuckman's approach where shifts are triangular and additive.

**Design 2 (Disjoint Piecewise-Constant "Time Buckets"):**

I'm not sure if your desk defines buckets in this exact way (sources do not pin down a single standard), but it is a common risk-reporting alternative.

Define disjoint buckets (years):
- B1: (0, 2]
- B2: (2, 4]
- B3: (4, 6]
- B4: (6, 8]
- B5: (8, 10]

Bump spot rates in one bucket by +1 bp (others unchanged). Under the linearized cashflow DV01 contributions from Example A, bucket DV01 is just the sum of $\mathrm{DV01}_t$ for cashflows in the bucket:

- B1: $t = 1, 2$: $0.0004623 + 0.0008900 = 0.0013523$
- B2: $t = 3, 4$: $0.0012836 + 0.0016457 = 0.0029293$
- B3: $t = 5, 6$: $0.0019780 + 0.0022823 = 0.0042603$
- B4: $t = 7, 8$: $0.0025603 + 0.0028135 = 0.0053738$
- B5: $t = 9, 10$: $0.0030435 + 0.0682828 = 0.0713262$

**Compare:**

Triangular key rates put more weight into "key-centered" regions (e.g., some of $t = 4$ is split between 3y and 5y keys). Disjoint buckets allocate each maturity entirely to one bucket.

Both can sum to total DV01 if the bucket moves span a parallel move, but the distribution differs—and hedge ratios will differ.

**Quote-Bump / Rebuild (Why Curve Method Matters):**

Interest-rate-modeling sources show that how you build and perturb curves changes the induced forward-curve move and hence risk numbers; adding spline tension can dampen perturbation noise.

**Desk Takeaway:** If your risk report uses quote-bump sensitivities, the resulting KRDV01/bucket numbers can differ materially from direct zero/forward bumps, affecting hedge construction and attribution.

---

### Example I — Risk Report Presentation (One Instrument)

Use Bond L from Example A, scaled to a $100mm face position.

**Step I1: Scale DV01**

Per $100 face:
- $\mathrm{DV01}_{\parallel} = 0.085242$

Scale factor: $\$100\text{mm} / \$100 = 1{,}000{,}000$.

So total DV01:

$$\mathrm{DV01}_{\text{total}} = 0.085242 \times 1{,}000{,}000 = \$85{,}242 \text{ per bp}$$

**Step I2: Scale KRDV01 by Key**

Multiply each per-$100 KRDV01 by 1,000,000:

| Key | KRDV01 (per $100) | KRDV01 ($100mm face) |
|-----|-------------------|----------------------|
| 2y | 0.001352 | 1,352 |
| 3y | 0.002106 | 2,106 |
| 5y | 0.003942 | 3,942 |
| 7y | 0.006592 | 6,592 |
| 10y | 0.071250 | 71,250 |
| **Total** | 0.085242 | 85,242 |

**Top 3 Buckets:**
- 10y: $71,250
- 7y: $6,592
- 5y: $3,942

**How This Guides Hedge Choice:**

The report says the risk "lives" mostly in the 10y sector → start hedging with a 10y instrument (Treasury, swap, or futures equivalent), then fine-tune residual 5–7y exposure with additional hedges.

---

## Practical Notes

### 1) Common Pitfalls

- **"DV01-neutral" ⇒ "no rate risk."** False: you can have offsetting bucket exposures and still be exposed to curve twists; Tuckman explicitly demonstrates this in bucket exposures
- **Inconsistent bump definitions between reports and hedge construction.** If one system uses quote-bump/rebuild and another uses direct zero-rate bumps, your hedge ratios won't line up
- **Confusing yield-bump DV01 with curve-bump DV01.** Tuckman notes "DV01" is often used to mean yield-based DV01; but DV01 can be defined for spot/zero or forward bumps (DVDZ vs DVDF terminology)
- **Key-rate set choice:**
  - Too few keys: miss shape risk between points
  - Too many keys: unstable/noisy hedges (especially if curve interpolation is not robust)
- **Hedging across instruments with different spread/basis characteristics:** Treasury vs swap vs futures can embed different risk (swap spread, funding, delivery options for bond futures, etc.)

### 2) Implementation Pitfalls

- **Bump-and-reprice numerical noise:** Tuckman notes you shouldn't choose bumps "too small" if pricing is noisy; otherwise you amplify errors
- **Interpolation artifacts:** Par-point bumps can create odd forward-curve shapes; Tuckman notes par-yield bumps can imply "bizarre" forward-curve changes, and switching the object you bump (par vs forward) changes locality
- **Coupon-date discontinuities / settlement assumptions:** Bucket/key risk jumps around coupon dates if you don't standardize cashflow calendars

### 3) Verification Tests

- **Additivity check:** Under an additive key-shift design (Tuckman triangular shifts), $\sum \mathrm{KRDV01}_k \approx \mathrm{DV01}_{\parallel}$
- **Sign checks:** Long fixed-coupon exposure should have DV01 > 0; paying fixed should have DV01 < 0 under Tuckman's sign convention
- **Stability check:** Smaller bump sizes should converge if pricing is stable
- **Scenario check:** Twist/steepener P&L should be approximated by $-\sum \mathrm{KRDV01}_k \Delta y_k$ for small moves

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **DV01** is "dollars per bp" defined by Tuckman as PV change for a 1 bp decline in the chosen rate measure
2. **DV01 is only a one-direction (parallel) view** of curve risk; it is insufficient for curve-shape moves
3. **Key-rate DV01 (KR01/KRDV01)** is computed by bumping one key rate under a specified shift design and repricing
4. **Tuckman's key-rate shifts** are triangular, piecewise-linear between keys, constant beyond endpoints, and additive (sum to a parallel shift)
5. Under additive shifts and small moves, $\sum \mathrm{KRDV01} \approx \text{parallel DV01}$
6. **Bucket exposures** often use forward-rate bumps (one forward bucket at a time); sums can approximate DV01
7. **"Parallel DV01 = 0" does not mean "no risk":** offsetting bucket exposures can leave large twist exposure
8. **Multi-instrument hedges** are solved as linear systems on KRDV01/bucket vectors
9. **Eurodollar (and similar STIR) futures** are natural for short-end bucket hedging; $25 per bp per contract is the key mapping
10. **Key-rate hedges can fail** due to convexity mismatch, basis/spread risk, and (in modern frameworks) multi-curve effects

---

### Cheat Sheet: Definitions + Bump Designs + Hedge Construction Steps

#### Definitions

$$\mathrm{DV01} \equiv -\frac{\Delta P}{10{,}000 \, \Delta y}$$

$$\mathrm{KR01}_k \equiv P_0 - P^{(k)}$$

where $P^{(k)}$ is PV under a +1 bp key-rate shift.

**Scenario P&L (small moves):**

$$\Delta P \approx -\sum_k \mathrm{KRDV01}_k \, \Delta y_{k,\mathrm{bp}}$$

#### Bump Designs

- **Tuckman triangular key shifts (par yields):** Piecewise-linear, additive
- **Forward bucket bump:** Bump one forward rate bucket and rederive spot/DFs
- **Quote-bump/rebuild:** Bump market quote and rebuild curve; results depend on curve construction

#### Hedge Construction

1. Compute $k$: KRDV01/bucket vector for your position
2. Compute $H$: exposure vectors for hedge instruments
3. Solve $\min \|k + Hn\|$
4. Validate by scenario shocks (steepeners/flatteners) and by bump-size stability tests

---

### Flashcards (30 Q/A Pairs)

**Q1:** What is DV01 in Tuckman's sign convention?
**A1:** $\mathrm{DV01} = -\frac{\Delta P}{10{,}000 \, \Delta y}$, positive when rates down increase PV.

**Q2:** Why does DV01 depend on the "rate measure" you bump?
**A2:** Because DV01 is defined for any chosen measure (yield, spot/zero, forward); the PV response differs.

**Q3:** What is a key rate?
**A3:** A selected maturity rate used to parameterize term-structure moves.

**Q4:** What is a key-rate shift (Tuckman)?
**A4:** A prescribed shape of curve perturbation due to a 1 bp move in one key rate.

**Q5:** What shape does Tuckman use for key-rate shifts?
**A5:** Piecewise-linear (triangular) between keys; constant beyond endpoints.

**Q6:** What property makes Tuckman's key shifts good for decomposition?
**A6:** Their sum equals a parallel shift.

**Q7:** Define KR01/KRDV01 precisely.
**A7:** $P_0 - P^{(k)}$ where $P^{(k)}$ is PV after applying only key shift $k$ of +1 bp.

**Q8:** Why can $\sum \mathrm{KRDV01}$ match DV01?
**A8:** Because the sum of key-rate shifts is a parallel shift (under that assumption).

**Q9:** What are bucket exposures in Tuckman's Chapter 7?
**A9:** PV changes from bumping individual forward rates/buckets by 1 bp.

**Q10:** What instrument does Tuckman highlight for hedging forward-rate buckets?
**A10:** Eurodollar futures.

**Q11:** What does "parallel DV01 = 0" guarantee?
**A11:** Only near-zero PV change for parallel shifts (in the chosen bump definition), not for twists.

**Q12:** What does Tuckman say about a parallel-hedged portfolio with bucket exposures?
**A12:** It can still have large interest-rate risk; can profit from flattening and lose from steepening.

**Q13:** Why is key-rate shift shape called a theoretical weakness?
**A13:** The shape is arbitrary; one could argue for smoother curves.

**Q14:** What does Hull call dividing the curve into segments for risk analysis?
**A14:** GAP management; segments are "buckets."

**Q15:** What is the value of 1 bp move in Eurodollar futures per contract?
**A15:** $25 per contract.

**Q16:** What is the Eurodollar futures quote convention?
**A16:** Quote $= 100 -$ futures rate.

**Q17:** If you are short Eurodollar futures, when do you profit?
**A17:** When rates rise (quote falls).

**Q18:** What is the main goal of KRDV01 reporting?
**A18:** Identify where along the curve the risk resides ("where the DV01 lives").

**Q19:** How do you use KRDV01 for a steepener scenario?
**A19:** Multiply bucket/key exposures by the bp moves and sum with the appropriate sign.

**Q20:** Why can key-rate hedges fail for large moves?
**A20:** Convexity mismatch (second-order effects).

**Q21:** What is one risk when hedging corporates with swaps?
**A21:** Spread/basis risk (curves can move differently).

**Q22:** What does "quote-bump/rebuild" mean?
**A22:** Bump market quotes and rebuild the curve before repricing.

**Q23:** Why do quote-bump risks depend on curve construction?
**A23:** Different interpolation/bootstrapping methods induce different forward-curve perturbations.

**Q24:** In Tuckman's par-yield key-rate + par-bond hedge setup, why is hedging easy?
**A24:** Each hedge security maps to a single key rate; exposure equals its DV01.

**Q25:** What is the "scenario check" for KRDV01?
**A25:** Twist P&L should be approximated by the dot product of KRDV01 and the twist.

**Q26:** What is the "stability check" in bump-and-reprice?
**A26:** Smaller bump sizes should converge (if the model is stable).

**Q27:** Why might par-yield bumps create odd forward-curve moves?
**A27:** Kinks in yields can translate into jumps/kinks in forwards.

**Q28:** What does Tuckman call the key-rate equivalent of duration?
**A28:** Key-rate duration.

**Q29:** What's the key difference between DV01 and duration?
**A29:** DV01 is an absolute price change; duration is a percentage change.

**Q30:** What's the core warning of this chapter?
**A30:** Parallel DV01 can be near zero even when curve-shape risk is large.

---

## Mini Problem Set (16 Questions)

*Provide brief solution sketches for questions 1–8 only.*

**1.** Compute DV01 (Tuckman sign convention) for a 5-year zero-coupon bond at a flat 4% annual curve.

**Sketch:** Use $P = 100(1.04)^{-5}$, and $\mathrm{DV01} = \frac{CF \cdot t \cdot DF}{1+y} \cdot 10^{-4}$.

---

**2.** For the 10y 5% bond in Example A, verify that the maturity cashflow contributes the majority of DV01.

**Sketch:** Compare $\mathrm{DV01}_{10}$ to $\sum_{t \leq 9} \mathrm{DV01}_t$.

---

**3.** Using Example A's KRDV01 vector, estimate P&L for a +5 bp parallel shift (all keys +5).

**Sketch:** Parallel shift $\Rightarrow \Delta P \approx -5 \sum \mathrm{KRDV01}_k = -5 \, \mathrm{DV01}$.

---

**4.** Recreate Example B's "DV01-neutral but risky" result under the opposite twist: −10 bp at 2y and +10 bp at 10y.

**Sketch:** Flip the signs of $\Delta y_k$ in the dot product.

---

**5.** For the 5y par swap in Example C, explain why the longest maturity bucket dominates DV01.

**Sketch:** Floating leg PV depends on $DF(T)$; sensitivity to $DF(T)$ is large (like principal).

---

**6.** Solve a 3-bucket hedge problem: you have exposures $[10k, 5k, 2k]$ $/bp and hedge instruments with exposure matrix

$$\begin{bmatrix} 8 & 1 & 0 \\ 0 & 4 & 1 \\ 0 & 0 & 2 \end{bmatrix}$$

(in k$/bp). Find hedge notionals.

**Sketch:** Back-substitute (upper triangular).

---

**7.** A forward-rate bucket exposure is $-\$2{,}500$/bp. How many Eurodollar futures contracts hedge it approximately?

**Sketch:** $2{,}500 / 25 = 100$ contracts; sign implies short futures for payer.

---

**8.** Give two reasons quote-bump/rebuild risks may differ from direct zero-rate bumps.

**Sketch:** Curve construction and interpolation differences induce different forward curve moves; model noise and smoothing/tension choices.

---

**9.** (No sketch) Show how adding more key rates can reduce residual twist exposure but increase hedge instability.

**10.** (No sketch) Explain how basis risk appears when hedging corporate bond DV01 with Treasury futures.

**11.** (No sketch) Describe how multi-curve (discount vs projection) breaks a single KRDV01 view into multiple curve risks.

**12.** (No sketch) Construct a portfolio with zero KRDV01 at 2y and 10y but nonzero KRDV01 at 5y, and interpret the scenario risk.

**13.** (No sketch) Compare bucketing by spot rates vs by forwards: when is one more natural?

**14.** (No sketch) Explain why bucket exposures can be used to hedge with Eurodollar futures per Tuckman.

**15.** (No sketch) Discuss how convexity adjustment in Eurodollar futures affects hedge precision (conceptual only).

**16.** (No sketch) Propose a verification workflow for a production KRDV01 engine.

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

| Fact | Source |
|------|--------|
| DV01 definition and sign convention | Tuckman Ch 5-6 |
| Key-rate shift construction (triangular, additive) | Tuckman Ch 7, Figure 7.1 |
| Sum of key-rate shifts = parallel shift | Tuckman Ch 7 |
| Key-rate exposures decompose DV01/duration | Tuckman Ch 7 |
| Bucket exposures via forward-rate bumps | Tuckman Ch 7 |
| "Parallel-hedged portfolio can still have large rate risk" | Tuckman Ch 7 (bucket exposure example) |
| Par-bond hedges have one-to-one key-rate mapping | Tuckman Ch 7 |
| Eurodollar futures: $25 per bp per contract | Hull Ch 6 |
| GAP management / bucket analysis | Hull Ch 4 |
| Curve construction affects risk numbers | Tuckman Ch 7, Andersen Vol 1 |
| Par-yield bumps can create "bizarre" forward curves | Tuckman Ch 7 |

### (B) Reasoned Inference — Note Derivation Logic

| Inference | Derivation |
|-----------|------------|
| $\sum \mathrm{KRDV01}_k \approx \mathrm{DV01}_{\parallel}$ | Follows from linearity + sum-to-parallel property of Tuckman shifts |
| Scenario P&L formula $\Delta P \approx -\sum \mathrm{KRDV01}_k \Delta y_k$ | First-order Taylor expansion |
| Hedge construction as linear system | Linear algebra from DV01 additivity |
| Convexity mismatch causes residual P&L for large moves | Second-order effects not captured by first-order DV01 |

### (C) Speculation — Flag Uncertainties

| Uncertainty | Note |
|-------------|------|
| Exact bucket-definition conventions | I'm not sure which exact convention your desk uses; sources motivate bucketed analysis but don't standardize a single implementation |
| Disjoint vs overlapping bucket designs | Common alternatives listed but not pinned down in sources |
| Specific fails-charge formulas | Sources discuss conceptually but don't specify mechanics |
