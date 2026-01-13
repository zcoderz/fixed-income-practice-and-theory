# Chapter 28: Basis Trades in Rates — OIS–IBOR Basis, Swap-Spread RV, and Curve RV ("What are you long/short?")

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- OIS floating payoff is linked to the geometric average of overnight rates over the period (via compounding)
- OIS rates are used to build a zero curve from OIS quotes when OIS discounting is used
- Multi-curve practice separates discounting and projection curves and uses basis instruments to link them
- The LIBOR–OIS spread is described as a short-term credit spread in risk texts
- At inception, the floating leg of a standard fixed-for-floating swap is worth par (when reset at LIBOR flat), so the fixed leg must be worth par
- Basis swap par condition: PV of legs must match with quoted spread on one leg
- Swap spreads often use the on-the-run government bond by convention, but this can be misleading due to liquidity and special financing
- On-the-run Treasuries can trade "special" in repo; special spreads can be volatile and large
- Bond holding P&L decomposes as: price change + interest income − financing cost
- Butterflies are framed as buying the "cheap" maturity and selling surrounding maturities
- Par-point risk approach: bump a benchmark instrument quote, rebuild the curve, and reprice
- Tenor basis was small pre-2007 but widened significantly post-crisis due to credit/liquidity forces

### (B) Reasoned Inference (Derived from A)

- Discount PV01 and projection PV01 can be computed separately via finite-difference bumps holding the other curve fixed
- Basis PV01 ≈ ±N(∑τᵢPd(0,Tᵢ))10⁻⁴ depending on whether you pay or receive the spread
- Multi-curve par rate is the discounted average of projected forwards: $K_{\text{par}}^{\text{(multi)}} = \frac{\sum \tau_i P_d(0,T_i) L_{\text{IBOR}}}{\sum \tau_i P_d(0,T_i)}$
- DV01-neutral packages still have twist/curvature exposure
- A basis position can be replicated with two vanilla swaps at the same fixed rate

### (C) Speculation (Clearly Labeled; Minimal)

- I'm not sure how far to push "balance-sheet constraints" or regulatory drivers from these sources alone; the provided excerpts emphasize liquidity premiums and special financing, but do not fully develop dealer balance-sheet constraint models
- I'm not sure what your desk's convention is for quoting OIS–IBOR basis without: currency, indices (SOFR? ESTR? SONIA?), and market quoting rule
- I'm not sure whether the provided sources explicitly discuss "forward wiggles contaminating basis RV," but it is a standard implementation issue

---

## Conventions & Notation

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $t = 0$ | Valuation date |
| $T_1 < T_2 < \ldots$ | Future cashflow dates (years) |
| 1 bp | $10^{-4}$ |
| $P_d(0,T)$ | Discount factor to $T$ from the discount curve (default: OIS) |
| $L_k(0,T_i,T_{i+1})$ | Forward rate for index $k$ over $[T_i, T_{i+1}]$, used to project floating cashflows |
| $\tau_i$ | Accrual year fraction for $[T_i, T_{i+1}]$ |
| $N$ | Notional (dollars) |
| $K$ | Fixed rate on a vanilla fixed-for-floating swap |
| $e_{1,2}(T)$ | Quoted floating–floating basis spread for exchanging $L_1$ vs $L_2$ to maturity $T$, quoted on the $L_1$ leg (can be + or −) |
| $\text{SwapSpread}(T)$ | Swap rate minus government bond yield of "the same maturity" (benchmark choice matters) |
| DV01 | Dollar value of a 1 bp move; for a bond, $\text{DV01} = P \times D_{\text{mod}} / 10{,}000$ |
| $\text{PV01}_{\text{discount}}$ | PV sensitivity to a 1 bp bump to the discount curve holding projection curves fixed |
| $\text{PV01}_{\text{projection}}$ | PV sensitivity to a 1 bp bump to the projection curve(s) holding discount curve fixed |

### Defaults Used in Examples

- **Discounting curve**: OIS discount factors (because OIS rates are used for discounting in practice)
- **Accrual factors**: Toy convention where $\tau_i$ equals the period length (e.g., 0.5 for semiannual). We explicitly note this simplification; Tuckman similarly simplifies by ignoring fixed/float payment-convention differences in some discussions.
- **PV sign**: PV is the value to the position as described (positive = asset)

### Two Valuation Frameworks

| Framework | Description |
|-----------|-------------|
| **Legacy single-curve** | One curve is used to both project IBOR cashflows and discount them (historically common) |
| **Multi-curve** | One curve is used to discount (often OIS) and separate curve(s) are used to project IBOR forwards; basis instruments (e.g., basis swaps) tie curves together |

---

## Core Concepts

### 1) Basis / Relative-Value Package ("What are you long/short?")

**Formal Definition:**

A basis / relative-value (RV) trade is a portfolio $\Pi$ constructed so that:
- its PV is (approximately) insensitive to some "nuisance" risks (e.g., parallel rate level), and
- its PV is sensitive to one or more relative objects: a spread, a curve shape, a benchmark dislocation, or a funding/financing wedge.

**Intuition:**

You are not "betting on rates" (only); you are betting on a difference:
- OIS vs IBOR (discount/projection separation),
- swap rate vs government yield (swap spread),
- 2s–10s slope or 2–5–10 curvature (curve RV),
- cash-vs-derivative packages where repo/carry matters.

**Trading / Risk / Portfolio Practice:**

Risk management language is usually "what are you long/short?" (e.g., "long 5-year vs short wings" for a butterfly; "long spread" vs "short spread" for swap spreads). Tuckman explicitly frames butterflies as buying the "cheap" maturity and selling surrounding maturities.

---

### 2) OIS Discounting vs IBOR Projection (Single-Curve vs Multi-Curve)

**Formal Definition:**

- **Single-curve**: use one curve $P(0,T)$ to both discount and to imply forwards $L(0,T_i,T_{i+1})$.
- **Multi-curve**: use a discount curve $P_d(0,T)$ and separate projection curves for each index $k$ (e.g., $L_{3m}$, $L_{6m}$). These curves are linked by basis instruments (basis swaps, tenor basis swaps).

**Intuition:**

Post-crisis, unsecured interbank term rates (IBOR) embed credit/liquidity components relative to near-risk-free overnight rates; this shows up as an OIS–IBOR spread (often called LIBOR–OIS). Risk texts describe the LIBOR–OIS spread as a short-term credit spread, and note the OIS floating rate is a geometric average of overnight rates.

**Trading / Risk / Portfolio Practice:**

- If you price the same swap under single-curve vs multi-curve you get different PV and par rates (Example A).
- Multi-curve introduces two kinds of curve risk:
  - discount curve risk (OIS),
  - projection curve risk (IBOR forward curves),
  - plus basis-spread risk that links curves.

---

### 3) OIS–IBOR Basis and Tenor Basis (Basis Swaps as Linking Constraints)

**Formal Definition:**

A floating–floating basis swap exchanges floating payments linked to two indices $L_1$ and $L_2$, typically plus a quoted spread on one leg. If the swap is at par, PV of legs must match:

$$\sum_i L_2(0,t_i^2,t_{i+1}^2)\,\tau_i^2\,P(t_{i+1}^2) = \sum_i \bigl(L_1(0,t_i^1,t_{i+1}^1) + e_{1,2}(T)\bigr)\,\tau_i^1\,P(t_{i+1}^1),$$

with $e_{1,2}(T)$ quoted on the $L_1$ leg and possibly positive or negative.

**Intuition:**

The basis spread compensates for the fact that two "floating" indices are not economically identical (credit/liquidity/tenor effects). The multi-curve framework uses such basis swaps to build "spread" curves off a base curve.

**Trading / Risk / Portfolio Practice:**

Basis swap PV can be decomposed into:
- exposure to the discount curve $P_d$,
- exposure to each projection curve $L_k$,
- exposure to the quoted basis $e$.

Tenor basis (e.g., 3M vs 6M) is treated similarly; the reference notes that pre-2007 such basis was small, but widened significantly post-crisis due to credit/liquidity forces.

---

### 4) Swap Spreads (Swap Rate vs Government Yield) and Benchmark Dependence

**Formal Definition:**

Swap spread at maturity $T$:
$$\text{SwapSpread}(T) = S_{\text{swap}}(T) - y_{\text{gov}}(T),$$
where $y_{\text{gov}}(T)$ is a government bond yield used as benchmark. Tuckman emphasizes:
- by convention, swap spreads often use the on-the-run government bond,
- but this can be misleading because on-the-run yields are affected by liquidity and special financing; an alternative is a fitted/interpolated yield curve from off-the-run bonds, or adjustments for financing/liquidity (with judgement).

**Intuition:**

A swap rate reflects something like a "continually refreshed" short-term bank funding rate exposure (via rolling LIBOR/IBOR), while a Treasury yield reflects government borrowing plus its own liquidity/financing features.

**Trading / Risk / Portfolio Practice:**

A swap-spread RV trade is not a pure spread bet unless you explicitly neutralize:
- rates DV01,
- benchmark choice (OTR vs fitted),
- funding/repo of the cash bond leg (if any).

In other words: **swap spread RV = swap leg + Treasury leg + funding/financing**.

---

### 5) Curve RV (Steepeners/Flatteners/Butterflies) as Shape Bets

**Formal Definition:**

- A **steepener** is a package designed to profit from increases in slope (e.g., $y_{10} - y_2$ rising).
- A **flattener** profits from slope decreases.
- A **butterfly** isolates curvature: e.g., "long belly, short wings" (or reverse). Tuckman's discussion of butterfly spreads explicitly frames it as buying the "cheap" maturity (e.g., 5-year) and selling 4- and 6-year, or in 2-5-10 language "buy 5-year, sell 2- and 10-year."

**Intuition:**

Curve RV trades try to remove the first-order exposure to the level of rates and keep exposure to shape (slope/curvature).

**Trading / Risk / Portfolio Practice:**

- Construct weights using DV01 (risk-weighting) so that parallel shifts are hedged (approximately).
- Remaining P&L comes from twists/curvature moves and from residual convexity and curve-rebuild effects.

---

## Math and Derivations

### 2.1 OIS Payoff and Discounting Motivation (Rates-Only View)

#### OIS Compounded Floating Rate (Definition)

The reference defines an overnight indexed swap where fixed leg pays $K$ and floating leg pays a rate derived from compounding daily overnight rates:

Floating payment at maturity $T$ is $\displaystyle\frac{R_{01}(0,T) - 1}{\tau(0,T)}$ (scaled by accrual).

The compounded growth factor is:

$$\boxed{R_{01}(t,T) = \prod_{i=1}^{M}\bigl(1 + f_{i-1}\,\tau_i\bigr),}$$

where $f_i$ are daily overnight rates and $\tau_i$ day-count fractions.

**Unit check:** $R_{01}$ is dimensionless (a growth factor). $(R_{01} - 1)/\tau$ is an annualized rate (1/year).

**Connection to OIS discounting:** When OIS rates are used for discounting, you must build a zero curve from OIS rates (analogous to building a swap zero curve from swap rates).

---

### 2.2 Vanilla Swap PV as "Discounted Expected Net Coupons"

#### General Swap PV Structure

The reference gives a generic swap PV as a sum of discounted net cashflows:

$$V_{\text{genswap}}(t) = \sum_n \tau_n\,q(t,T_{n+1})\,P(t,T_{n+1})\,(L_n(t) - k_n),$$

with cashflow $\tau_n(L_n - k_n)$ paid at $T_{n+1}$.

**Interpretation for this chapter:** $P(t,T)$ is the discount factor (discount curve), and $L_n$ comes from the relevant projection curve.

#### Par Swap Rate Concept

Tuckman's key valuation fact: at inception, the floating leg of a standard fixed-for-floating swap is worth par (when reset at LIBOR flat), so the fixed leg must be worth par; therefore the fixed rate equals the par swap rate.

#### Derived Par-Rate Formula (Single-Curve, Annual-Pay Toy)

For a fixed-for-floating swap (no notional exchange) with payments at $T_1, \ldots, T_n$, accruals $\tau_i$, and discount factors $P(0,T_i)$:

**PV of fixed leg** (per unit notional):
$$PV_{\text{fixed}}(K) = K \sum_{i=1}^{n} \tau_i P(0,T_i) \equiv K \cdot A(0),$$
where $A(0)$ is the swap annuity.

In a single-curve world with internally consistent forwards, the floating leg PV equals $1 - P(0,T_n)$ (standard telescoping argument).

Setting PV fixed = PV float yields:

$$\boxed{K_{\text{par}} = \frac{1 - P(0,T_n)}{\sum_{i=1}^{n} \tau_i P(0,T_i)} = \frac{1 - P(0,T_n)}{A(0)}.}$$

**Sanity check:** If rates are near zero so $P(0,T) \approx 1$, then $K_{\text{par}} \approx 0$.

**Note:** The telescoping identity relies on the forward rates being implied by the same curve used for discounting (single-curve). In multi-curve, this identity generally does not hold because projection forwards and discount factors come from different curves.

---

### 2.3 Multi-Curve Decomposition: Discount Curve vs Projection Curve

#### Multi-Curve Setup

The reference explicitly describes constructing a discounting curve from OIS instruments and separate projection curves for IBOR tenors, with tenor basis handled by adding spreads/basis instruments.

#### PV Decomposition (Rates-Only, Linear Instruments)

Consider a pay-fixed swap with fixed rate $K$, payment dates $T_1, \ldots, T_n$, accruals $\tau_i$.

Under multi-curve, a clean "rates-only" PV (per notional) is:

$$PV_{\text{swap}}(0) = \sum_{i=1}^{n} \tau_i\,P_d(0,T_i)\bigl(L_{\text{IBOR}}(0,T_{i-1},T_i) - K\bigr).$$

Discount vs projection appears explicitly:
- $P_d(0,T_i)$ — discount curve,
- $L_{\text{IBOR}}$ — projection curve.

#### Derived Multi-Curve Par Rate

Set $PV_{\text{swap}}(0) = 0$ and solve:

$$\boxed{K_{\text{par}}^{\text{(multi)}} = \frac{\sum_{i=1}^{n} \tau_i P_d(0,T_i)\,L_{\text{IBOR}}(0,T_{i-1},T_i)}{\sum_{i=1}^{n} \tau_i P_d(0,T_i)}.}$$

This is the "discounted average" of projected forwards.

**Unit check:**
- Numerator: (year) × (dimensionless) × (1/year) summed ⇒ dimensionless.
- Denominator: (year) × (dimensionless) summed ⇒ years.
- Ratio: 1/year (a rate). ✅

---

### 2.4 Floating–Floating Basis Swaps: PV and Par Basis Spread

#### Par Condition from the Reference

At par, PV of the two legs match with basis spread $e_{1,2}(T)$ quoted on the $L_1$ leg.

#### Derived "Solve-for-Basis" Formula (Common Case: Same Schedule)

If both legs share the same payment dates and accruals (toy simplification):

$$\sum_i \tau_i P_d(0,T_i) L_i^2 = \sum_i \tau_i P_d(0,T_i)(L_i^1 + e).$$

Solve:

$$\boxed{e_{\text{par}} = \frac{\sum_i \tau_i P_d(0,T_i)(L_i^2 - L_i^1)}{\sum_i \tau_i P_d(0,T_i)}.}$$

#### Basis DV01 (Sensitivity to Quoted Basis)

For a position where $e$ is paid (i.e., PV decreases when $e$ increases):

$$\frac{\partial PV}{\partial e} = -N \sum_i \tau_i P_d(0,T_i), \quad\text{so}\quad \Delta PV \approx -N\Bigl(\sum_i \tau_i P_d(0,T_i)\Bigr)\Delta e.$$

If $\Delta e = +1\text{ bp} = 10^{-4}$, then "basis PV01" (in $) is $-N(\sum \tau_i P_d) \cdot 10^{-4}$.

**Convention warning:** The reference is explicit that $e_{1,2}(T)$ is quoted on the $L_1$ leg. But which market quotes 'OIS–IBOR basis' on which leg depends on the instrument/currency. I'm not sure what your desk's convention is without: currency, indices (SOFR? ESTR? SONIA?), and market quoting rule ("+spread on OIS leg" vs "+spread on IBOR leg"). For this chapter's examples we will always state which leg carries $e$.

---

### 2.5 Swap Spread RV: Why Benchmark and Funding Matter

#### Swap Spread Benchmark Dependence

Swap spreads are defined as swap rate minus a government bond yield, often using the on-the-run bond because no exact maturity match exists; but on-the-run yields embed liquidity and special financing, so swap spreads can be distorted. Alternatives include fitted/interpolated yields or adjusted OTR yields.

#### Funding/Carry Decomposition for Cash Bonds (Repo-Financed)

Tuckman decomposes bond holding P&L over $d$ days as:

$$\text{P\&L} = P(d) + AI(d) - \bigl(P(0) + AI(0)\bigr)(1 + rd/360) = \text{Price change} + \text{Interest income} - \text{Financing cost},$$

i.e., price change + carry.

This is the minimal repo-funding machinery we need for swap spread RV scenarios.

---

### 2.6 Risk Sensitivities and "What Gets Bumped?"

#### Par-Point / Bump-and-Rebuild Approach

The reference describes defining interest-rate risk by bumping a benchmark instrument quote, rebuilding the curve, and repricing ("par-point" approach).

**Key point for this chapter:** Your PV01 (and key-rate PV01) is not purely a property of the instrument; it also depends on:
- curve representation (zeros vs forwards vs par quotes),
- bump definition (zero bump vs par-quote bump),
- curve rebuild graph (which other curves are re-fit).

---

## Measurement & Risk (Only What Belongs in Chapter 28)

### 3.1 Central Organizing Framework: "What are you long/short?"

For any rates RV/basis package, write PV as:

$$PV = PV\Bigl(\underbrace{P_d}_{\text{discount curve}},\; \underbrace{\{L_k\}}_{\text{projection curves}},\; \underbrace{\{e_{k,\ell}\}}_{\text{quoted basis}},\; \underbrace{\text{benchmarks}}_{\text{gov curve choice}},\; \underbrace{\text{funding}}_{\text{repo/carry}}\Bigr).$$

Then the "long/short" decomposition is simply the sign of the partial derivatives / finite-difference bumps:

| Exposure | Sensitivity |
|----------|-------------|
| Discount curve exposure | $\partial PV / \partial P_d$ (or $\text{PV01}_{\text{discount}}$) |
| Projection curve exposure | $\partial PV / \partial L_k$ for each index curve $k$ (or $\text{PV01}_{\text{projection}}^{(k)}$) |
| Quoted basis exposure | $\partial PV / \partial e_{k,\ell}$ (basis PV01) |
| Benchmark choice exposure | Sensitivity of computed swap spread to which gov yield is used (OTR vs fitted) |
| Funding/repo exposure | Carry changes from repo rate changes (for cash bond legs) |

---

### 3.2 Discounting Basis Exposure (OIS vs Non-OIS Discounting)

#### PV Sensitivity to Discount Curve Choice

In multi-curve PV formulas, every cashflow is multiplied by $P_d(0,T)$. Even if the projection forwards are unchanged, changing $P_d$ changes PV.

**Discount PV01** (finite-difference definition used in this chapter):

$$\text{PV01}_{\text{discount}} \equiv PV(P_{d,+1\text{bp}}, \{L_k\}) - PV(P_d, \{L_k\}),$$

where only the discount curve is bumped.

**How it shows up in multi-curve pricing:** The same swap can have different PV under single-curve vs multi-curve because discounting differs (Example A).

**Unit checks:** PV01 is in dollars; often reported as $/bp per notional (e.g., per $1mm).

---

### 3.3 Projection Basis Exposure (IBOR Forwards and Tenor Basis)

#### PV Sensitivity to Projection Curve

Floating coupons depend on $L_k(0,T_i,T_{i+1})$. Bumping these changes projected cashflows directly.

**Projection PV01** (finite-difference definition):

$$\text{PV01}_{\text{projection}}^{(k)} \equiv PV(P_d, L_{k,+1\text{bp}}) - PV(P_d, L_k),$$

holding discount curve fixed.

#### Tenor Basis and Basis Swaps as Linking Constraints

Index curves can be constructed as spread curves tied down by floating–floating basis swaps; the par condition directly links the projection curves and quoted basis spreads under a given discount curve.

**Risk interpretation:** In a spread-based multi-curve construction, perturbing instruments of the base curve is interpreted as level risk, while perturbing basis instruments has a clearer "orthogonal" meaning as basis risk (conceptually).

---

### 3.4 Liquidity/Funding Components (Rates-RV Reality)

#### Repo Funding and Specialness (Cash Legs)

Treasury cash legs in swap spread trades are subject to repo financing. Tuckman decomposes bond P&L into price change + interest income − financing cost (carry).

On-the-run Treasuries can trade "special" in repo; special spreads can be volatile and large, reflecting collateral supply/demand.

#### Liquidity Premium and Benchmark Distortions

Recent/on-the-run issues can trade rich because of financing advantages and liquidity premiums.

#### Balance-Sheet / Technical Drivers

I'm not sure how far to push "balance-sheet constraints" or regulatory drivers from these sources alone; the provided excerpts emphasize liquidity premiums and special financing, but do not fully develop dealer balance-sheet constraint models. To be certain, we'd need an explicit section/source in your reference set that models balance-sheet costs as a spread driver.

---

### 3.5 Define and Contrast the RV Objects in This Chapter

| RV Object | Definition |
|-----------|------------|
| **OIS–IBOR basis** | Difference between "near-risk-free/overnight" curve (OIS) and unsecured term curve (IBOR), operationalized via basis swaps and multi-curve pricing |
| **Swap spread** | Swap rate minus government yield; depends on benchmark selection (OTR vs fitted/adjusted) |
| **Curve RV** | Shape changes (slope/curvature). Butterflies and steepeners are classic constructions; Tuckman discusses butterflies as "buy the cheap maturity, sell surrounding maturities." |

---

### 3.6 Risk and P&L Explain (Explicit and Reproducible)

#### PV01 to Discount Curve vs PV01 to Projection Curve

Report separately as $/bp per $1mm or per trade notional:
- $\text{PV01}_{\text{discount}}$ via bumping discount curve only,
- $\text{PV01}_{\text{projection}}$ via bumping projection curve(s) only.

#### Key-Rate / Bucket PV01 Preview (Brief)

Key-rate PV01 uses localized bumps (e.g., "2y key rate"). Implementation depends on curve construction and bump-and-rebuild choices ("par-point" methodology).

#### Basis Sensitivity

For basis swap quoted spread $e$, basis PV01 $\approx \pm N(\sum \tau_i P_d(0,T_i)) \cdot 10^{-4}$ depending on whether you pay or receive the spread.

#### Residuals to Track

- **Convexity:** second-order effects in larger moves (not captured by DV01/PV01).
- **Curve construction dependence:** "what gets bumped?" (Example L).
- **Funding effects:** repo/carry for cash legs can dominate short-horizon P&L even when DV01-hedged.

---

## Worked Examples

*All examples are toy numbers for pedagogy. Each example states conventions. Where we simplify day-count, we say so (following the spirit of Tuckman's simplification).*

---

### Example A — OIS vs IBOR Swap Valuation Difference (Same Swap, Two Frameworks)

**Goal:** Price the same 3Y pay-fixed swap under:
1. legacy single-curve (IBOR projects and discounts),
2. multi-curve (OIS discounts, IBOR projects),

and quantify PV and par-rate differences.

#### A.1 Conventions

- Notional $N = \$1{,}000{,}000$.
- Annual payments at $T_1 = 1$, $T_2 = 2$, $T_3 = 3$.
- Accruals $\tau_i = 1$ (toy).
- Swap direction: pay fixed $K$, receive floating IBOR.

**Legacy single-curve inputs (IBOR curve):** discount factors
$$P_L(0,1) = 0.9650,\quad P_L(0,2) = 0.9250,\quad P_L(0,3) = 0.8850.$$

**Multi-curve inputs:**
- Discount factors (OIS):
$$P_d(0,1) = 0.9700,\quad P_d(0,2) = 0.9400,\quad P_d(0,3) = 0.9100.$$

- Projection forwards (IBOR) taken from the legacy curve via simple no-arb identities:
$$\begin{aligned}
L(0,0,1) &= \frac{1}{P_L(0,1)} - 1 = \frac{1}{0.9650} - 1 = 0.036269,\\[4pt]
L(0,1,2) &= \frac{P_L(0,1)}{P_L(0,2)} - 1 = \frac{0.9650}{0.9250} - 1 = 0.043243,\\[4pt]
L(0,2,3) &= \frac{P_L(0,2)}{P_L(0,3)} - 1 = \frac{0.9250}{0.8850} - 1 = 0.045197.
\end{aligned}$$

(Toy: forwards remain unchanged between frameworks.)

#### A.2 Step 1 — Compute Legacy Single-Curve Par Rate $K_{\text{par}}^{\text{(single)}}$

Use the par swap-rate concept (floating leg at par implies fixed leg at par).

**Annuity:**
$$A_L = \sum_{i=1}^{3} \tau_i P_L(0,T_i) = 0.9650 + 0.9250 + 0.8850 = 2.7750.$$

**Float PV (single-curve):** $1 - P_L(0,3) = 1 - 0.8850 = 0.1150$.

**Par fixed rate:**
$$K_{\text{par}}^{\text{(single)}} = \frac{0.1150}{2.7750} = 0.04144 \approx 4.144\%.$$

Define the "same swap" as the contractual fixed rate:
$$K \equiv 4.144\%.$$

#### A.3 Step 2 — Price the Same Swap Under Legacy Single-Curve

Because $K$ is the single-curve par rate, $PV_{\text{single}} = 0$ (by construction).

#### A.4 Step 3 — Price Under Multi-Curve (OIS Discounting, IBOR Projection)

PV per unit notional (toy linear PV):
$$PV_{\text{multi}}(0) = \sum_{i=1}^{3} \tau_i P_d(0,T_i)\bigl(L(0,T_{i-1},T_i) - K\bigr).$$

Compute each term:
- $T_1$: $L - K = 0.036269 - 0.04144 = -0.005171$;
  $0.9700 \times (-0.005171) = -0.005016$.

- $T_2$: $L - K = 0.043243 - 0.04144 = +0.001803$;
  $0.9400 \times 0.001803 = 0.001694$.

- $T_3$: $L - K = 0.045197 - 0.04144 = +0.003757$;
  $0.9100 \times 0.003757 = 0.003418$.

**Sum:**
$$PV_{\text{multi}}/N \approx -0.005016 + 0.001694 + 0.003418 = 0.000095.$$

So:
$$PV_{\text{multi}} \approx 0.000095 \times 1{,}000{,}000 = \$95.$$

#### A.5 Step 4 — Compare Par Rates Under the Two Frameworks

**Multi-curve annuity:**
$$A_d = 0.9700 + 0.9400 + 0.9100 = 2.8200.$$

**Multi-curve numerator:**
$$\sum P_d L = 0.9700(0.036269) + 0.9400(0.043243) + 0.9100(0.045197) = 0.116959.$$

**Multi-curve par rate:**
$$K_{\text{par}}^{\text{(multi)}} = \frac{0.116959}{2.8200} = 0.041475 \approx 4.1475\%.$$

#### A.6 Interpretation ("Discount vs Projection Basis")

- The same contractual swap can have different PV when discounting switches from a legacy IBOR curve to OIS (multi-curve).
- The par rate shift $K_{\text{par}}^{\text{(multi)}} - K_{\text{par}}^{\text{(single)}} \approx 0.34$ bp is a clean way to quantify the discounting/projection separation in this toy setup.

---

### Example B — OIS–IBOR Basis Swap Par Condition and Par Basis Spread

**Goal:** Write the PV equation for an OIS–IBOR basis swap and solve for the par basis spread.

#### B.1 Conventions

- Notional $N = \$1{,}000{,}000$.
- Maturity $T = 2$ years.
- Semiannual payments: $T = \{0.5, 1.0, 1.5, 2.0\}$, $\tau = 0.5$.

**Discounting curve:** OIS discount factors (toy):
$$P_d(0,0.5) = 0.9850,\; P_d(0,1) = 0.9700,\; P_d(0,1.5) = 0.9550,\; P_d(0,2) = 0.9400.$$

**Projection curves:**
- IBOR-6M forwards (toy inputs):
$$L_{\text{IBOR}} = \{3.60\%, 3.80\%, 4.00\%, 4.10\%\} \text{ for the four periods}.$$

- OIS forwards (implied from discount factors; simple-comp, consistent with the curve):
$$L_{\text{OIS}}(0,T_{i-1},T_i) = \frac{1}{\tau}\Bigl(\frac{P_d(0,T_{i-1})}{P_d(0,T_i)} - 1\Bigr).$$

**Quoting convention (must be explicit):**

We map the reference's convention "spread quoted on the $L_1$ leg" by defining:
- $L_1 \equiv$ OIS leg,
- $L_2 \equiv$ IBOR leg,
- quoted spread $e$ is added to the OIS leg (so the OIS leg pays $L_{\text{OIS}} + e$).

I'm not sure this matches your market's quoting convention; if your market quotes on the IBOR leg, swap the roles of $L_1, L_2$ and the sign interpretation.

#### B.2 Step 1 — Compute OIS Forwards from Discount Factors

Using $\tau = 0.5$:

- $0 \to 0.5$:
$$L_{\text{OIS}} = \frac{1}{0.5}\Bigl(\frac{1}{0.9850} - 1\Bigr) = 2(1.015228 - 1) = 0.030456 = 3.0456\%.$$

- $0.5 \to 1.0$:
$$L_{\text{OIS}} = 2\Bigl(\frac{0.9850}{0.9700} - 1\Bigr) = 2(1.015464 - 1) = 0.030928 = 3.0928\%.$$

- $1.0 \to 1.5$:
$$L_{\text{OIS}} = 2\Bigl(\frac{0.9700}{0.9550} - 1\Bigr) = 0.031416 = 3.1416\%.$$

- $1.5 \to 2.0$:
$$L_{\text{OIS}} = 2\Bigl(\frac{0.9550}{0.9400} - 1\Bigr) = 0.031914 = 3.1914\%.$$

#### B.3 Step 2 — Write Par Condition and Solve for $e$

From the reference, par floating–floating basis swap means PV(IBOR leg) = PV(OIS+spread leg).

Using common schedule:
$$\sum_{i=1}^{4} \tau P_d(0,T_i)\,L_i^{\text{IBOR}} = \sum_{i=1}^{4} \tau P_d(0,T_i)\,(L_i^{\text{OIS}} + e).$$

Rearrange:
$$e = \frac{\sum_{i=1}^{4} \tau P_d(0,T_i)(L_i^{\text{IBOR}} - L_i^{\text{OIS}})}{\sum_{i=1}^{4} \tau P_d(0,T_i)}.$$

**Compute weights** $w_i = \tau P_d(0,T_i)$:
- $w_{0.5} = 0.5(0.9850) = 0.4925$
- $w_{1.0} = 0.5(0.9700) = 0.4850$
- $w_{1.5} = 0.5(0.9550) = 0.4775$
- $w_{2.0} = 0.5(0.9400) = 0.4700$

**Sum** $W = \sum w_i = 1.9250$.

**Compute differences** $\Delta_i = L_i^{\text{IBOR}} - L_i^{\text{OIS}}$:
- $0 \to 0.5$: $0.03600 - 0.030456 = 0.005544$
- $0.5 \to 1.0$: $0.03800 - 0.030928 = 0.007072$
- $1.0 \to 1.5$: $0.04000 - 0.031416 = 0.008584$
- $1.5 \to 2.0$: $0.04100 - 0.031914 = 0.009086$

**Compute numerator:**
$$\sum w_i \Delta_i = 0.4925(0.005544) + 0.4850(0.007072) + 0.4775(0.008584) + 0.4700(0.009086) \approx 0.014530.$$

**Therefore:**
$$e \approx \frac{0.014530}{1.9250} = 0.007548 \approx 0.7548\% = 75.5\text{ bp}.$$

#### B.4 Sanity Check

Because IBOR forwards are higher than OIS forwards in all periods, the spread $e$ added to the OIS leg must be positive to make the legs equal PV. ✅

---

### Example C — Discount PV01 vs Projection PV01 for a Swap (Finite-Difference)

**Goal:** Compute (finite-difference):
- $\text{PV01}_{\text{discount}}$: bump OIS discount curve $+1$ bp, hold projection fixed,
- $\text{PV01}_{\text{projection}}$: bump IBOR projection curve $+1$ bp, hold discount fixed.

#### C.1 Conventions

- Use a 3Y annual-pay swap, same dates as Example A.
- Notional $N = \$1{,}000{,}000$.
- Pay fixed $K = 4.00\%$, receive IBOR forwards $L = \{3.6269\%, 4.3243\%, 4.5197\%\}$.
- Discount curve = OIS discount factors $P_d(0,1) = 0.97$, $P_d(0,2) = 0.94$, $P_d(0,3) = 0.91$.
- Accruals $\tau = 1$.

**Bump rules (must be explicit):**
- **Discount-curve bump:** parallel zero-rate bump $+1$ bp approximated by
$$P_{d,+}(0,T) = P_d(0,T)\,e^{-0.0001\,T}.$$

- **Projection-curve bump:** add $+1$ bp to each forward:
$$L_i^{+} = L_i + 0.0001.$$

In both cases, hold the other curve fixed.

#### C.2 Step 1 — Base PV

PV per unit notional:
$$PV(0) = \sum_{i=1}^{3} P_d(0,T_i)\,(L_i - K).$$

Compute $L_i - K$:
- $T_1$: $0.036269 - 0.040000 = -0.003731$
- $T_2$: $0.043243 - 0.040000 = +0.003243$
- $T_3$: $0.045197 - 0.040000 = +0.005197$

Multiply by discount factors:
- $0.9700(-0.003731) = -0.003619$
- $0.9400(0.003243) = 0.003048$
- $0.9100(0.005197) = 0.004729$

**Sum:**
$$PV/N \approx -0.003619 + 0.003048 + 0.004729 = 0.004159.$$

So:
$$PV \approx 0.004159 \times 1{,}000{,}000 = \$4{,}159.$$

#### C.3 Step 2 — Discount PV01 (Bump Discount Only)

**Compute bumped discount factors:**
- $T = 1$: $0.97\,e^{-0.0001} \approx 0.97(0.9999) = 0.969903$
- $T = 2$: $0.94\,e^{-0.0002} \approx 0.939812$
- $T = 3$: $0.91\,e^{-0.0003} \approx 0.909727$

Recompute PV with same $L_i$ and $K$:
$$PV_{d,+}/N \approx 0.969903(-0.003731) + 0.939812(0.003243) + 0.909727(0.005197).$$

Numerically:
$$\approx -0.003619 + 0.003048 + 0.004728 = 0.004157.$$

So $PV_{d,+} \approx \$4{,}157$.

**Therefore:**
$$\boxed{\text{PV01}_{\text{discount}} = PV_{d,+} - PV \approx 4{,}157 - 4{,}159 = -\$2 \text{ per \$1mm}.}$$

#### C.4 Step 3 — Projection PV01 (Bump Projection Only)

Bump forwards by $+1$ bp: $L_i^{+} = L_i + 0.0001$.

PV change per unit notional:
$$\Delta PV / N = \sum_{i=1}^{3} P_d(0,T_i) \cdot 0.0001 = 0.0001(0.97 + 0.94 + 0.91) = 0.000282.$$

So:
$$\boxed{\text{PV01}_{\text{projection}} \approx 0.000282 \times 1{,}000{,}000 = \$282 \text{ per \$1mm}.}$$

#### C.5 Interpretation ("Where the Risk Lives")

In this decomposition, the swap's PV is far more sensitive to projection ($+\$282$/bp) than to discounting ($\approx -\$2$/bp).

**Intuition:** bumping forwards changes the size of projected coupons, while bumping discount factors changes primarily the present-valuing of already-nearly-offsetting legs.

**Caution:** If you instead bump market quotes and rebuild curves (par-point approach), the allocation between "discount PV01" and "projection PV01" can shift (Example L).

---

### Example D — Basis Spread Sensitivity ("Basis PV01") for Example B

**Goal:** PV change per $+1$ bp change in quoted basis spread $e$, per $\$1$mm notional.

#### D.1 Conventions

- Use Example B basis swap (2Y semiannual).
- Spread $e$ is on the OIS leg; position is receive IBOR, pay (OIS + e).
- Notional $N = \$1{,}000{,}000$.
- Discount factors and weights from Example B: $W = \sum \tau P_d = 1.9250$.

#### D.2 Calculation

PV of spread leg (per unit basis rate) is:
$$PV_{\text{spread}} = -N \sum_i \tau P_d(0,T_i)\,e = -N\,W\,e.$$

So per $+1$ bp ($\Delta e = 0.0001$):
$$\Delta PV \approx -N\,W\,0.0001 = -(1{,}000{,}000)(1.9250)(0.0001) = -\$192.50.$$

#### D.3 Output

$$\boxed{\text{Basis PV01 (for this position): } -\$192.50 \text{ per +1 bp of } e \text{ per \$1mm notional}.}$$

---

### Example E — Replicating a Basis Position with Two Swaps (Explicit PV Match)

**Goal:** Show a basis exposure can be represented as a portfolio of an IBOR swap and an OIS swap (toy, explicit) and match PV.

#### E.1 Conventions

- Use the same 2Y semiannual schedule as Example B.
- Discount curve $P_d$ same as Example B.
- Projection forwards:
  - IBOR forwards: $\{0.036, 0.038, 0.040, 0.041\}$.
  - OIS forwards: from Example B $\{0.030456, 0.030928, 0.031416, 0.031914\}$.
- Notional $N = \$1{,}000{,}000$.
- Accrual $\tau = 0.5$.
- Choose a common fixed rate for replication: $K_{\text{rep}} = 3.50\%$ (arbitrary).
- Define weights $w_i = \tau P_d(0,T_i)$ (from Example B):
$$w = \{0.4925, 0.4850, 0.4775, 0.4700\}.$$

#### E.2 Step 1 — PV of the "IBOR Pay-Fixed Swap" at $K_{\text{rep}}$

PV per unit notional:
$$PV_{\text{IBORswap}} = \sum_i w_i(L_i^{\text{IBOR}} - K_{\text{rep}}).$$

Differences $L_{\text{IBOR}} - K_{\text{rep}}$:
- $0.036 - 0.035 = 0.001$
- $0.038 - 0.035 = 0.003$
- $0.040 - 0.035 = 0.005$
- $0.041 - 0.035 = 0.006$

Weighted sum:
$$PV/N = 0.4925(0.001) + 0.4850(0.003) + 0.4775(0.005) + 0.4700(0.006) = 0.007155.$$

So $PV_{\text{IBORswap}} = \$7{,}155$.

#### E.3 Step 2 — PV of the "OIS Pay-Fixed Swap" at $K_{\text{rep}}$

$$PV_{\text{OISswap}} = \sum_i w_i(L_i^{\text{OIS}} - K_{\text{rep}}).$$

Differences $L_{\text{OIS}} - 0.035$:
- $0.030456 - 0.035 = -0.004544$
- $0.030928 - 0.035 = -0.004072$
- $0.031416 - 0.035 = -0.003584$
- $0.031914 - 0.035 = -0.003086$

Weighted sum:
$$PV/N \approx -0.4925(0.004544) - 0.4850(0.004072) - 0.4775(0.003584) - 0.4700(0.003086) = -0.007375.$$

So $PV_{\text{OISswap}} \approx -\$7{,}375$.

#### E.4 Step 3 — Replication Portfolio

Take:
- **Long** the IBOR pay-fixed swap (pay fixed, receive IBOR).
- **Short** the OIS pay-fixed swap (so you receive fixed, pay OIS).

Then fixed legs at $K_{\text{rep}}$ cancel, leaving net receive IBOR, pay OIS (a zero-spread basis swap).

**PV of portfolio:**
$$PV_{\text{rep}} = PV_{\text{IBORswap}} - PV_{\text{OISswap}} \approx 7{,}155 - (-7{,}375) = \$14{,}530.$$

#### E.5 Step 4 — Direct PV of the (No-Spread) Basis Swap

Directly:
$$PV_{\text{basis,no-spread}} = \sum_i w_i(L_i^{\text{IBOR}} - L_i^{\text{OIS}}).$$

But this numerator was computed in Example B as $\approx 0.014530$ per unit notional, so:
$$PV \approx 0.014530 \times 1{,}000{,}000 = \$14{,}530.$$

**Match:** $PV_{\text{rep}} = PV_{\text{basis,no-spread}}$ (to rounding). ✅

#### E.6 Adding the Quoted Spread $e$

If the basis swap pays $(OIS + e)$, its PV becomes:
$$PV_{\text{basis}} = PV_{\text{basis,no-spread}} - N\,W\,e.$$

At the par spread from Example B, this equals zero.

---

### Example F — Swap Spread Sensitivity to Benchmark Choice (OTR vs Fitted)

**Goal:** Compute swap spread using two benchmark yields and show the difference.

#### F.1 Conventions

- Maturity: 5Y.
- Swap rate: $S_{\text{swap}} = 4.25\%$.
- Government yield benchmarks:
  - On-the-run yield $y_{\text{OTR}} = 4.00\%$,
  - Fitted curve yield $y_{\text{fit}} = 3.92\%$.
- Swap spread defined as swap rate − government yield.

#### F.2 Calculations

**Using on-the-run:**
$$\text{SwapSpread}_{\text{OTR}} = 4.25\% - 4.00\% = 0.25\% = 25\text{ bp}.$$

**Using fitted curve:**
$$\text{SwapSpread}_{\text{fit}} = 4.25\% - 3.92\% = 0.33\% = 33\text{ bp}.$$

**Difference:**
$$33\text{ bp} - 25\text{ bp} = 8\text{ bp}.$$

#### F.3 Interpretation

Tuckman warns that using the on-the-run benchmark can be misleading because OTR yields are influenced by liquidity and special financing; a fitted/interpolated yield curve can mitigate this, though it introduces modeling choices.

Therefore, a swap-spread "signal" can move by several bp without economics changing, purely due to benchmark choice.

---

### Example G — Swap Spread RV Trade: DV01-Hedged Construction ("What are you long/short?")

**Goal:** Build a toy swap-spread trade and DV01-hedge it with a Treasury bond.

#### G.1 Conventions

- Maturity: 5Y.
- Swap leg: pay fixed on a 5Y swap (receive floating).
- Treasury leg: long a 5Y Treasury bond to DV01 hedge.
- Swap notional: $N_{\text{swap}} = \$100{,}000{,}000$.
- Swap fixed payment frequency: annual (toy), $\tau = 1$.

**Discount factors used for swap annuity (toy OIS DFs):**
$$P_d(0,1..5) = \{0.97, 0.94, 0.91, 0.88, 0.85\}.$$

**Treasury bond characteristics (toy):**
- Clean price $P = 100$ per $100 face.
- Modified duration $D_{\text{mod}} = 4.60$.
- DV01 formula: $\text{DV01} = P \cdot D_{\text{mod}} / 10{,}000$.

#### G.2 Step 1 — DV01 of the Swap (PV01 to Fixed Rate, Magnitude)

**Swap annuity:**
$$A = \sum_{i=1}^{5} \tau P_d(0,i) = 0.97 + 0.94 + 0.91 + 0.88 + 0.85 = 4.55.$$

PV change for a 1 bp change in fixed rate (magnitude):
$$\text{DV01}_{\text{swap}} \approx N_{\text{swap}} \cdot A \cdot 10^{-4} = 100{,}000{,}000 \times 4.55 \times 10^{-4} = \$45{,}500.$$

**Sign check:** for a pay-fixed swap, rates up tends to increase value, so the PV01 sign is $+$ for a +1bp rate shift (toy linear view).

#### G.3 Step 2 — DV01 of the Treasury Bond per $100 Face

$$\text{DV01}_{\text{bond, per \$100}} = \frac{P \cdot D_{\text{mod}}}{10{,}000} = \frac{100 \times 4.60}{10{,}000} = 0.046.$$

Thus DV01 per $1 face is $0.046/100 = 0.00046$.

#### G.4 Step 3 — Hedge Ratio

Let Treasury face amount be $F$ dollars. Then:
$$\text{DV01}_{\text{bond}} = 0.00046 \times F.$$

DV01 neutrality requires (using the standard hedge-ratio idea):
$$0.00046\,F \approx 45{,}500 \quad\Rightarrow\quad F = \frac{45{,}500}{0.00046} = 98{,}913{,}043 \approx \$98.9\text{ mm face}.$$

**Hedge ratio:**
$$h = \frac{F}{N_{\text{swap}}} \approx \frac{98.9}{100} = 0.989.$$

#### G.5 "What are you long/short?"

- **Short (swap spread leg):** pay fixed swap (profits if swap rates rise vs funding curve).
- **Long (gov leg):** Treasury bond (profits if gov yields fall).
- **DV01-neutralized:** parallel moves should cancel (approximately), leaving relative move exposure (swap vs Treasury) plus carry/funding.

---

### Example H — Swap Spread Trade P&L Scenarios (Rates, Spread, Funding)

**Goal:** For the Example G position, compute P&L under:
1. parallel rates shift,
2. swap spread change,
3. funding (repo) shock.

#### H.1 Conventions

- Use Example G hedge: pay-fixed swap $N_{\text{swap}} = \$100$mm, long Treasury face $F = \$98.913$mm.
- Use DV01 approximation:
  - Swap: $\$45{,}500$/bp (pay-fixed gains when rates rise).
  - Bond: $\$45{,}500$/bp (long bond loses when yields rise).
- Horizon: $d = 30$ days.
- Treasury bond coupon: $c = 4\%$ annual, paid semiannually (2% per 180 days).
- Repo financing rate: initial $r = 3.50\%$ (Actual/360-like approximation consistent with Tuckman's $rd/360$ term).
- Assume settle on coupon date so $AI(0) = 0$ for simplicity.

#### H.2 Scenario 1 — Parallel Shift: Swap Rate +10bp, Treasury Yield +10bp

**Swap P&L:**
$$\Delta PV_{\text{swap}} \approx +45{,}500 \times 10 = +\$455{,}000.$$

**Bond price P&L** (long bond loses when yields rise):
$$\Delta PV_{\text{bond}} \approx -45{,}500 \times 10 = -\$455{,}000.$$

**Net (ignoring carry):** $\approx 0$.

**Interpretation:** DV01 hedge worked for parallel moves.

#### H.3 Scenario 2 — Swap Spread Widening: Swap Rate +5bp, Treasury Yield Unchanged

**Swap P&L:**
$$+45{,}500 \times 5 = +\$227{,}500.$$

**Bond price P&L:** $\approx 0$.

**Add carry on the Treasury over 30 days** using Tuckman's decomposition:
$$\text{P\&L} = \text{Price change} + \text{Interest income} - \text{Financing cost}.$$

**Interest income over 30 days** (semiannual coupon accrual):
$$\text{Interest} = F \times 0.02 \times \frac{30}{180} = 98.913\text{ mm} \times 0.0033333 \approx \$329{,}710.$$

**Financing cost:**
$$\text{FinCost} = (F)(rd/360) = 98.913\text{ mm} \times 0.035 \times \frac{30}{360} = 98.913\text{ mm} \times 0.0029167 \approx \$288{,}496.$$

**Carry:**
$$\text{Carry} \approx 329{,}710 - 288{,}496 = +\$41{,}214.$$

**Total scenario-2 P&L (toy):**
$$\boxed{227{,}500 + 41{,}214 = \$268{,}714.}$$

#### H.4 Scenario 3 — Funding/Specialness Shock: Repo Rate +100bp (3.50% → 4.50%), No Price Move

**Swap P&L:** $0$.

**Bond carry under new repo** $r = 4.50\%$:
$$\text{FinCost} = 98.913\text{ mm} \times 0.045 \times \frac{30}{360} = 98.913\text{ mm} \times 0.00375 \approx \$370{,}924.$$

Interest income unchanged $\approx \$329{,}710$.

**Carry:**
$$\text{Carry} \approx 329{,}710 - 370{,}924 = -\$41{,}214.$$

Relative to the base carry $+\$41{,}214$, the funding shock changes P&L by:
$$\boxed{-41{,}214 - (+41{,}214) = -\$82{,}428.}$$

#### H.5 Interpretation

Even DV01-hedged, a swap-spread package can have meaningful P&L from repo/funding over realistic horizons (and repo specialness can be volatile).

This is why "swap spread" is not a pure derivatives spread: the cash leg drags in financing mechanics.

---

### Example I — Curve RV: DV01-Neutral Steepener (Receive 2Y, Pay 10Y) and Twist P&L

**Goal:** Build a swap-curve steepener and compute P&L under a twist.

#### I.1 Conventions

- Instruments: par swaps on 2Y and 10Y points.
- Use annual-pay toy annuities from OIS discount factors:
$$P_d(0,1..10) = \{0.97, 0.94, 0.91, 0.88, 0.85, 0.82, 0.79, 0.76, 0.73, 0.70\}.$$

- PV01 of a swap's fixed leg per $1mm notional (magnitude):
$$\text{PV01} \approx 1{,}000{,}000 \Bigl(\sum_{i=1}^{n} P_d(0,i)\Bigr) \cdot 10^{-4}.$$

**Trade direction (as required):**
- Receive fixed 2Y (long duration).
- Pay fixed 10Y (short duration).
- Notional scaling to DV01-neutralize.

#### I.2 Step 1 — Compute PV01s (per $1mm Notional)

**2Y annuity:**
$$A_2 = 0.97 + 0.94 = 1.91 \quad\Rightarrow\quad \text{PV01}_{2Y} = 1{,}000{,}000(1.91) \cdot 10^{-4} = \$191.$$

**10Y annuity:**
$$A_{10} = 0.97 + 0.94 + 0.91 + 0.88 + 0.85 + 0.82 + 0.79 + 0.76 + 0.73 + 0.70 = 8.35$$
$$\Rightarrow\quad \text{PV01}_{10Y} = 1{,}000{,}000(8.35) \cdot 10^{-4} = \$835.$$

#### I.3 Step 2 — Choose Notionals for DV01-Neutralization

Let 10Y notional be $N_{10} = \$1$mm, and 2Y notional be $N_2$. We want:
$$N_2(191) \approx N_{10}(835) \quad\Rightarrow\quad N_2 \approx \frac{835}{191} = 4.372\text{ mm}.$$

So:
- Receive 2Y fixed on $4.372mm notional.
- Pay 10Y fixed on $1.000mm notional.

**Parallel DV01 check (magnitudes):**
- Receive-2Y DV01 $\approx 4.372 \times 191 \approx \$835$.
- Pay-10Y DV01 $\approx \$835$.
- Net $\approx 0$. ✅

#### I.4 Step 3 — Twist Scenario P&L

**Twist scenario ("steepening"):**
- 2Y rate increases by $+5$ bp,
- 10Y rate increases by $+20$ bp.

**P&L approximations:**

Receive fixed loses when rates rise:
$$\Delta PV_{2Y} \approx -(835) \times 5 = -\$4{,}175.$$

Pay fixed gains when rates rise:
$$\Delta PV_{10Y} \approx +(835) \times 20 = +\$16{,}700.$$

**Net:**
$$\boxed{\Delta PV \approx +\$12{,}525.}$$

#### I.5 Interpretation

- **DV01-neutral:** limited exposure to parallel moves.
- **Main exposure:** slope factor (10Y–2Y) increasing.

---

### Example J — Curve RV: DV01-Neutral Butterfly (2Y/5Y/10Y) and Curvature Shock

**Goal:** Build a 2–5–10 butterfly and compute P&L under a curvature shock.

#### J.1 Conventions

Use same OIS discount factors as Example I.

**PV01 per $1mm:**
- 2Y: $191.
- 5Y: $A_5 = 0.97 + 0.94 + 0.91 + 0.88 + 0.85 = 4.55 \Rightarrow \text{PV01}_{5Y} = \$455$.
- 10Y: $835.

**Position structure:**
- Receive fixed 2Y and 10Y (wings),
- Pay fixed 5Y (belly),
- choose belly notional to DV01-neutralize.

#### J.2 Step 1 — Choose Weights

Let $N_2 = N_{10} = \$1$mm. Total wing DV01:
$$191 + 835 = 1026.$$

Choose belly notional $N_5$ such that:
$$N_5(455) \approx 1026 \quad\Rightarrow\quad N_5 = \frac{1026}{455} = 2.255\text{ mm}.$$

So the butterfly is:
- Receive 2Y fixed on $1mm.
- Pay 5Y fixed on $2.255mm.
- Receive 10Y fixed on $1mm.

**DV01 check:**
- Receive wings DV01 $\approx 1026$.
- Pay belly DV01 $\approx 2.255 \times 455 \approx 1026$.
- Net $\approx 0$. ✅

#### J.3 Step 2 — Curvature Shock P&L

**Curvature shock:** belly yield increases $+10$ bp, wings unchanged.

**Pay-fixed 5Y gains:**
$$\Delta PV_{5Y} \approx +(1026) \times 10 = \$10{,}260.$$

**Wings unchanged:** $\approx 0$.

**Net:**
$$\boxed{+\$10{,}260.}$$

#### J.4 Interpretation

DV01-neutral butterfly isolates relative movement of the belly vs wings, i.e., curvature.

---

### Example K — Carry/Rolldown vs Basis Move for a Basis Trade

**Goal:** Compute deterministic carry/rolldown over a horizon assuming curves unchanged, then apply a basis widening scenario.

#### K.1 Conventions

- Use Example B 2Y semiannual basis swap.
- Notional $N = \$1{,}000{,}000$.
- Position: receive IBOR, pay (OIS + e) where $e = 0.7548\%$ (par at $t = 0$).
- Horizon: first coupon date $t = 0.5$ years.
- **Assumption for carry/rolldown:**
  - realized fixings equal initial forwards (toy),
  - discount and projection curves unchanged.

#### K.2 Step 1 — First-Period Realized Net Coupon at $t = 0.5$

**Receive IBOR coupon:**
$$CF_{\text{IBOR}} = N\tau L_{0\to0.5}^{\text{IBOR}} = 1{,}000{,}000(0.5)(0.036) = \$18{,}000.$$

**Pay OIS + spread coupon:**
$$CF_{\text{OIS}+e} = N\tau(L_{0\to0.5}^{\text{OIS}} + e) = 1{,}000{,}000(0.5)(0.030456 + 0.007548).$$

Note $0.030456 + 0.007548 = 0.038004$, so:
$$CF_{\text{OIS}+e} = 1{,}000{,}000(0.5)(0.038004) = \$19{,}002.$$

**Net cashflow (receive − pay):**
$$CF_{0.5} = 18{,}000 - 19{,}002 = -\$1{,}002.$$

#### K.3 Step 2 — Mark-to-Market of Remaining Swap at $t = 0.5$ (Curves Unchanged)

Remaining payment dates: $1.0, 1.5, 2.0$.

**Discount factors from $t = 0.5$:**
$$P(0.5,T) = \frac{P(0,T)}{P(0,0.5)}.$$

So:
- $P(0.5,1) = 0.9700/0.9850 = 0.985279$
- $P(0.5,1.5) = 0.9550/0.9850 = 0.969543$
- $P(0.5,2) = 0.9400/0.9850 = 0.954314$

**Weights at $t = 0.5$:** $w_i' = \tau P(0.5,T_i)$:
- $w_1' = 0.5(0.985279) = 0.492640$
- $w_2' = 0.5(0.969543) = 0.484772$
- $w_3' = 0.5(0.954314) = 0.477157$

**Sum** $W' = 1.454568$.

**Compute remaining par spread** (same formula as Example B but on remaining periods):
$$e_{\text{rem}} = \frac{\sum w_i'(L_{\text{IBOR}} - L_{\text{OIS}})}{\sum w_i'}.$$

Using remaining diffs:
- $0.5 \to 1$: $0.007072$
- $1 \to 1.5$: $0.008584$
- $1.5 \to 2$: $0.009086$

**Numerator:**
$$0.492640(0.007072) + 0.484772(0.008584) + 0.477157(0.009086) \approx 0.011981.$$

Hence:
$$e_{\text{rem}} \approx \frac{0.011981}{1.454568} = 0.008237 = 0.8237\% = 82.4\text{ bp}.$$

**Value of remaining swap** to the position (receive IBOR, pay OIS+$e_{\text{contract}}$) is:
$$PV_{t=0.5} = N\,(e_{\text{rem}} - e_{\text{contract}})\,W' = 1{,}000{,}000(0.008237 - 0.007548)(1.454568).$$

Difference $= 0.000689$. Multiply:
$$PV_{t=0.5} \approx 1{,}000{,}000(0.000689)(1.454568) \approx \$1{,}002.$$

#### K.4 Step 3 — Deterministic Carry/Rolldown P&L (Curves Unchanged)

**Total P&L at $t = 0.5$** (realized coupon + MTM of remaining):
$$\text{P\&L} \approx CF_{0.5} + PV_{t=0.5} \approx (-1{,}002) + (+1{,}002) \approx 0.$$

**Interpretation:** at-par basis swap has ~zero deterministic carry/rolldown in this toy setup.

#### K.5 Step 4 — Basis Widening Scenario (+10bp on Remaining Basis)

At $t = 0.5$, suppose IBOR–OIS basis widens by +10 bp for remaining periods (projection change), discount curve unchanged.

**Approximate PV impact on remaining swap:**
$$\Delta PV_{t=0.5} \approx N\,W'\,(0.0010) = 1{,}000{,}000(1.454568)(0.0010) = +\$1{,}455.$$

**Total P&L becomes:**
$$\boxed{\text{P\&L} \approx 0 + 1{,}455 = +\$1{,}455.}$$

**Message:** realized P&L departs from carry/rolldown when the basis moves.

---

### Example L — "What Gets Bumped?": PV01 Under Two Bump Definitions

**Goal:** Recompute a PV01 under:
1. parallel zero-rate bump,
2. par-quote bump with curve rebuild,

and show the difference. This illustrates curve-model dependence (par-point methodology).

#### L.1 Conventions

**Single-curve (legacy-style) toy curve, annual payments.**

**Instruments used to bootstrap:**
- 1Y deposit discount factor $P(0,1) = 0.9700$.
- 2Y par swap rate $S_2 = 3.50\%$.
- 3Y par swap rate $S_3 = 3.80\%$.

**Swap to measure PV01 on:**
- 3Y pay-fixed swap with fixed rate $K = 4.00\%$,
- Notional $N = \$1{,}000{,}000$,
- Annual $\tau = 1$.

**Swap PV under single curve** (derived from standard par logic):
- $PV_{\text{float}} = 1 - P(0,3)$, $PV_{\text{fixed}} = K \sum_{i=1}^{3} P(0,i)$.

(Conceptually consistent with "floating leg at par implies par fixed rate.")

#### L.2 Step 1 — Bootstrap Base Discount Factors $P(0,2), P(0,3)$

**2Y par swap condition** (annual-pay, single curve):
$$S_2 = \frac{1 - P_2}{P_1 + P_2} \quad\Rightarrow\quad P_2 = \frac{1 - S_2 P_1}{1 + S_2}.$$

Plugging $P_1 = 0.9700$, $S_2 = 0.0350$:
$$P_2 = \frac{1 - 0.035(0.9700)}{1.035} = \frac{0.96605}{1.035} = 0.93338.$$

**3Y par swap condition:**
$$S_3 = \frac{1 - P_3}{P_1 + P_2 + P_3} \quad\Rightarrow\quad P_3 = \frac{1 - S_3(P_1 + P_2)}{1 + S_3}.$$

Compute $P_1 + P_2 = 0.9700 + 0.93338 = 1.90338$. Then:
$$P_3 = \frac{1 - 0.038(1.90338)}{1.038} = \frac{1 - 0.072328}{1.038} = \frac{0.927672}{1.038} = 0.89371.$$

#### L.3 Step 2 — Base PV of the 3Y Swap (Pay Fixed 4.00%)

**Float PV:**
$$PV_{\text{float}} = 1 - P_3 = 1 - 0.89371 = 0.10629.$$

**Fixed PV:**
$$PV_{\text{fixed}} = K(P_1 + P_2 + P_3) = 0.04(0.9700 + 0.93338 + 0.89371) = 0.04(2.79709) = 0.11188.$$

**Swap PV (receive float − pay fixed):**
$$PV/N = 0.10629 - 0.11188 = -0.00559 \quad\Rightarrow\quad PV \approx -\$5{,}594.$$

#### L.4 Method 1 — Parallel Zero-Rate Bump (+1bp), No Rebuild

**Bump rule:**
$$P^{+}(0,T) = P(0,T)\,e^{-0.0001\,T}.$$

So:
- $P_1^{+} = 0.9700\,e^{-0.0001} \approx 0.969903$
- $P_2^{+} = 0.93338\,e^{-0.0002} \approx 0.933193$
- $P_3^{+} = 0.89371\,e^{-0.0003} \approx 0.893442$

**Recompute PV:**

$PV_{\text{float}}^{+} = 1 - P_3^{+} = 0.106558$.

$\sum P^{+} = 0.969903 + 0.933193 + 0.893442 = 2.796538$.

$PV_{\text{fixed}}^{+} = 0.04(2.796538) = 0.111862$.

$PV^{+}/N = 0.106558 - 0.111862 = -0.005304 \Rightarrow PV^{+} \approx -\$5{,}304$.

**So PV01:**
$$\boxed{\text{PV01}_{\text{zero-bump}} = PV^{+} - PV = (-5{,}304) - (-5{,}594) = +\$290 \text{ per \$1mm}.}$$

#### L.5 Method 2 — Par-Quote Bump (+1bp) with Bootstrap Rebuild

This matches the "bump the market quote, rebuild, reprice" idea (par-point approach).

**Bump the market quotes:**

- Bump 1Y deposit rate by +1bp (toy).
  - Base simple 1Y rate $r_1 = \frac{1}{P_1} - 1 = \frac{1}{0.97} - 1 = 0.030928$.
  - Bumped: $r_1^{+} = 0.031028$.
  - New $P_1^{\text{reb}} = \frac{1}{1 + r_1^{+}} \approx \frac{1}{1.031028} \approx 0.9699$.

- Bump par swap quotes:
  - $S_2^{+} = 0.0351$,
  - $S_3^{+} = 0.0381$.

**Rebuild:**
$$P_2^{\text{reb}} = \frac{1 - S_2^{+} P_1^{\text{reb}}}{1 + S_2^{+}} \approx 0.93320,\quad P_3^{\text{reb}} = \frac{1 - S_3^{+}(P_1^{\text{reb}} + P_2^{\text{reb}})}{1 + S_3^{+}} \approx 0.89345.$$

**Reprice swap:**

$PV_{\text{float}}^{\text{reb}} = 1 - 0.89345 = 0.10655$.

$\sum P^{\text{reb}} \approx 0.9699 + 0.93320 + 0.89345 = 2.79655$.

$PV_{\text{fixed}}^{\text{reb}} = 0.04(2.79655) = 0.11186$.

$PV^{\text{reb}}/N \approx 0.10655 - 0.11186 = -0.00531 \Rightarrow PV^{\text{reb}} \approx -\$5{,}313$.

**So PV01:**
$$\boxed{\text{PV01}_{\text{par-bump+rebuild}} = PV^{\text{reb}} - PV = (-5{,}313) - (-5{,}594) = +\$281 \text{ per \$1mm}.}$$

#### L.6 Interpretation

The PV01 differs ($290 vs $281 per $1mm) purely because "1bp shift" was implemented differently.

This is why you must specify "what gets bumped?" and whether curves are rebuilt (par-point approach).

---

## Practical Notes

### 5.1 "What You're Long/Short" Checklist (Explicit)

When you write up (or review) any rates basis/RV trade, explicitly list:

| Exposure | Questions to Answer |
|----------|---------------------|
| **Discount curve exposure** | Are you long/short OIS discount factors $P_d$? Do you report $\text{PV01}_{\text{discount}}$ separately? |
| **Projection curve exposure** | Which forward curve(s) drive your coupons (IBOR 3M, IBOR 6M, OIS, etc.)? Do you report $\text{PV01}_{\text{projection}}^{(k)}$ per index? |
| **Quoted basis exposure** | Which basis quote(s) enter PV (e.g., $e_{1,2}(T)$)? What is your basis PV01 (per $1mm)? |
| **Benchmark choice exposure (swap spreads)** | Are you measuring swap spread vs on-the-run Treasury yields, or fitted/interpolated Treasury curve yields? Tuckman emphasizes this choice can distort the signal due to liquidity and special financing in on-the-run issues. |
| **Funding/repo exposure (cash legs)** | If you hold/short a cash bond, what repo rate is assumed? What is carry (interest income − financing cost) and how sensitive is it to repo moves? |
| **Convexity and curve-shape residuals** | Are you truly DV01-neutral (parallel) but exposed to twists/curvature? Are there convexity mismatches? |

---

### 5.2 Common Pitfalls

1. **Treating a basis trade as "purely a spread trade"** without recognizing embedded curve exposures:
   - discount curve vs projection curve risk (Examples A–C).

2. **Mixing discount/projection curves:**
   - projecting floating cashflows off the wrong curve is a conceptual error in multi-curve valuation (the whole point is separation).

3. **Confusing swap spread with other spread measures** (asset swap spread, Z-spread, spread-to-swaps):
   - Swap spread is explicitly swap rate minus government yield benchmark and is benchmark-dependent.

4. **Ignoring benchmark definition changes:**
   - moving from OTR to fitted curve can move your "swap spread" without economics changing (Example F).

5. **Forgetting funding assumptions in cash-vs-swap RV trades:**
   - repo/carry can materially change realized P&L (Example H).

---

### 5.3 Implementation Pitfalls (Model & System)

1. **Curve dependency graph:**
   - In multi-curve, discount curve and projection curves may share instruments indirectly via basis constructions. Be explicit about bump-and-rebuild scope.

2. **Bump-and-rebuild consistency:**
   - Par-point risk measures require bumping market quotes and rebuilding curves; risk numbers depend on the definition.

3. **Interpolation artifacts:**
   - I'm not sure whether the provided sources explicitly discuss "forward wiggles contaminating basis RV," but it is a standard implementation issue: poor interpolation can create unrealistic forward shapes, which can distort basis PV and hedges.

4. **Benchmark/CTD instability (if futures used):**
   - If you hedge Treasuries with futures proxies, hedge ratios can jump when the cheapest-to-deliver or benchmark changes. (This is standard practice knowledge; not explicitly sourced in the provided excerpts.)

---

### 5.4 Verification Tests (Desk-Reproducible)

| Test | Description |
|------|-------------|
| **Repricing tests** | Each instrument reprices to market quotes used in the curve set (swap, OIS, basis swap), within tolerance. |
| **Sign checks** | If you pay the basis spread $e$, PV should decrease when $e$ increases (Example D). If you are long a bond, PV should decrease when yields increase (DV01 sign). |
| **Scaling checks** | PV and PV01 scale linearly with notional; confirm per $1mm scaling. |
| **Scenario suite** | Parallel shift, twist, basis shock, funding shock (Examples H–K). |

---

## Summary & Recall

### 6.1 10-Bullet Executive Summary

1. A rates basis/RV trade is best understood as a portfolio of curve/spread/funding exposures—state what you are long/short.

2. Multi-curve pricing separates discounting (often OIS) from projection (IBOR forwards), linked by basis instruments.

3. OIS floating leg is tied to compounding/geometric averaging of overnight rates; OIS rates are used to build a discounting curve when OIS discounting is used.

4. Basis swaps enforce a par condition equating PVs of floating legs, with a quoted spread on a specified leg; the quoted basis can be $+$ or $-$.

5. Swap spreads are swap rate minus a government yield benchmark; benchmark choice (OTR vs fitted) matters materially.

6. Swap spread RV trades with cash Treasuries have repo funding/carry risk; bond P&L decomposes into price change + interest income − financing cost.

7. Curve RV trades (steepeners, butterflies) are shape bets; DV01-neutralization reduces level exposure but leaves slope/curvature exposure.

8. Risk decomposition: separate $\text{PV01}_{\text{discount}}$ and $\text{PV01}_{\text{projection}}$ to locate where risk lives (Example C).

9. Basis PV01 is PV sensitivity to the quoted basis $e$ and is essentially the discounted accrual "annuity" of the spread leg (Example D).

10. Risk numbers depend on what gets bumped and whether curves are rebuilt (par-point approach).

---

### 6.2 Cheat Sheet

#### Key PV Equations

**OIS compounded growth factor:**
$$\boxed{R_{01}(t,T) = \prod_i (1 + f_{i-1}\tau_i),\quad \text{floating payoff} = \frac{R_{01}(0,T) - 1}{\tau(0,T)}.}$$

**Generic swap PV (discounted net coupons):**
$$V = \sum_n \tau_n\,q\,P(t,T_{n+1})(L_n - k_n).$$

**Floating–floating basis par condition:**
$$\sum L_2 \tau P = \sum(L_1 + e_{1,2})\tau P.$$

**Bond DV01:**
$$\boxed{\text{DV01} = \frac{P\,D_{\text{mod}}}{10{,}000}.}$$

#### "What are you long/short?" Decomposition Map

$$PV(P_d, L_k, e, \text{benchmark}, \text{repo})$$

| Exposure | Sensitivity |
|----------|-------------|
| Discount curve exposure | $\text{PV01}_{\text{discount}}$ |
| Projection curve exposure | $\text{PV01}_{\text{projection}}^{(k)}$ |
| Basis exposure | $\partial PV / \partial e$ (basis PV01) |
| Benchmark dependence | $S_{\text{swap}} - y_{\text{gov}}$ depends on $y_{\text{gov}}$ definition |
| Funding/carry | price change + carry |

#### Risk Metrics and Units

| Metric | Units |
|--------|-------|
| PV | dollars |
| PV01 / DV01 | $/bp |
| Basis PV01 | $/bp per $1mm notional of basis spread $e$ |
| Key-rate PV01 | $/bp for a localized bump (definition depends on curve rebuild) |

---

### 6.3 Flashcards (35 Q/A)

1. **Q:** What is the core question behind "what are you long/short?"
   **A:** Identify which curves/spreads/funding inputs your PV increases with (long) or decreases with (short).

2. **Q:** In multi-curve pricing, what are the two main curve roles?
   **A:** Discounting curve $P_d$ and projection curves $L_k$.

3. **Q:** What does OIS floating leg reference?
   **A:** Compounded/geometric average of overnight rates.

4. **Q:** Why build a zero curve from OIS rates?
   **A:** If OIS rates are used for discounting, you need OIS discount factors for PV.

5. **Q:** Define swap spread.
   **A:** Swap rate minus government bond yield benchmark of that maturity.

6. **Q:** Why can on-the-run Treasuries distort swap spreads?
   **A:** Liquidity and special financing affect OTR yields.

7. **Q:** What is a floating–floating basis swap par condition?
   **A:** PV of legs equal; one leg includes quoted spread $e_{1,2}$.

8. **Q:** On which leg is $e_{1,2}(T)$ quoted in the reference?
   **A:** On the $L_1$ leg.

9. **Q:** Can $e_{1,2}(T)$ be negative?
   **A:** Yes; it can be positive or negative depending on desirability of indices.

10. **Q:** What is DV01 for a bond in terms of price and modified duration?
    **A:** $P D_{\text{mod}} / 10{,}000$.

11. **Q:** What is DV01 conceptually (Hull definition)?
    **A:** Dollar value of a 1 bp increase in all interest rates.

12. **Q:** What is the "par swap rate" concept?
    **A:** Fixed rate that makes swap PV zero; floating leg at par implies fixed leg at par.

13. **Q:** How do you DV01-hedge two instruments A and B?
    **A:** Choose sizes so $F_A \cdot \text{DV01}_A + F_B \cdot \text{DV01}_B \approx 0$.

14. **Q:** What is "discount PV01" in this chapter?
    **A:** PV change from bumping discount curve only.

15. **Q:** What is "projection PV01" in this chapter?
    **A:** PV change from bumping projection curve only.

16. **Q:** Why can projection PV01 dominate discount PV01 for near-par swaps?
    **A:** Forward bumps change coupon amounts directly; discount bumps rescale both legs and can partially cancel.

17. **Q:** What is basis PV01?
    **A:** PV change per 1bp change in quoted basis spread $e$.

18. **Q:** How does basis PV01 scale with notional?
    **A:** Linearly.

19. **Q:** What does "bump-and-rebuild" mean?
    **A:** Perturb market quotes, rebuild curve(s), reprice instruments.

20. **Q:** Why do risk numbers depend on "what gets bumped?"
    **A:** Because different parameterizations imply different DF changes for the same "1bp" concept.

21. **Q:** What is the repo P&L decomposition for a financed bond position?
    **A:** Price change + interest income − financing cost (carry).

22. **Q:** What is "specialness" in repo?
    **A:** Specific collateral financing advantage; can be volatile and large.

23. **Q:** Why does specialness matter for swap spread trades?
    **A:** It changes carry and can distort Treasury yields/relative value.

24. **Q:** What is a steepener?
    **A:** Trade that profits when long-term rates rise relative to short-term rates (slope increases).

25. **Q:** What is a flattener?
    **A:** Profits when slope decreases.

26. **Q:** What is a butterfly in curve RV?
    **A:** Long belly vs short wings (or reverse), isolating curvature.

27. **Q:** What does DV01-neutral mean for a curve RV package?
    **A:** Net PV01 to parallel shifts is approximately zero.

28. **Q:** What residual risk remains after DV01-neutralization?
    **A:** Twist/curvature exposure and convexity.

29. **Q:** What is benchmark-choice exposure?
    **A:** Dependence of measured "swap spread" on the chosen government yield proxy.

30. **Q:** What is the main linkage instrument between index curves in multi-curve?
    **A:** Floating–floating basis swaps tying curves via par conditions.

31. **Q:** Why did tenor basis widen after 2007–2009 in the reference's discussion?
    **A:** Credit and liquidity factors increased basis spreads.

32. **Q:** In a basis swap, why must you specify which leg carries the spread?
    **A:** Because PV and sign conventions depend on which index is "quoted leg."

33. **Q:** What is the simplest way to check PV scaling?
    **A:** Double notional; PV and PV01 should double.

34. **Q:** What is the simplest sign check for paying a spread $e$?
    **A:** If you pay $e$, PV decreases when $e$ increases.

35. **Q:** What is the key moral of this chapter?
    **A:** RV trades are never "just a spread"—they embed curve, benchmark, and funding exposures that must be decomposed.

---

## Mini Problem Set (18 Questions)

*Provide brief solution sketches for questions 1–9 only.*

---

**1. (Easy)** Define $\text{SwapSpread}(T)$. Why can the definition depend on the benchmark government yield?

**Sketch:** Swap spread = swap rate − gov yield; Tuckman notes OTR vs fitted yields can differ due to liquidity/special financing.

---

**2. (Easy)** Write the par condition for a floating–floating basis swap as given in the reference and state where the basis spread is applied.

**Sketch:** Use equation (6.49); spread $e_{1,2}(T)$ is quoted on $L_1$ leg and can be ±.

---

**3. (Easy)** If you are paying the quoted basis spread $e$, what is the sign of $\partial PV / \partial e$?

**Sketch:** Negative: PV decreases when what you pay increases.

---

**4. (Easy)** Compute the DV01 of a bond with price 102 and modified duration 7.5.

**Sketch:** DV01 per $100 = $102 \times 7.5 / 10{,}000 = 0.0765$.

---

**5. (Easy–Medium)** Using Tuckman's repo P&L decomposition, identify the three components of P&L for a financed bond position.

**Sketch:** Price change + interest income − financing cost = price change + carry.

---

**6. (Medium)** Given discount factors $P_d(0,T_i)$ and projection forwards $L_i$, derive the multi-curve par swap rate formula (discounted average of forwards).

**Sketch:** Set swap PV $\sum \tau P_d(L_i - K) = 0 \Rightarrow K = \frac{\sum \tau P_d L_i}{\sum \tau P_d}$. (Derived from generic swap PV form.)

---

**7. (Medium)** For a basis swap with spread on leg 1, derive a basis PV01 expression in terms of discounted accruals.

**Sketch:** PV includes $-Ne\sum\tau P_d$; 1bp bump gives $-N\sum\tau P_d \cdot 10^{-4}$.

---

**8. (Medium)** You have a swap PV01 of $60k/bp and a Treasury DV01 of $75k/bp. What Treasury notional ratio hedges DV01 if you pay fixed and go long Treasuries?

**Sketch:** Hedge ratio $h = 60/75 = 0.8$ (scale Treasury DV01 to match). Use DV01 hedge ratio concept.

---

**9. (Medium)** Explain why $\text{PV01}_{\text{discount}}$ and $\text{PV01}_{\text{projection}}$ depend on the bump definition and curve rebuild.

**Sketch:** Par-point approach bumps market quotes and rebuilds curves; "1bp" can map differently into discount factors and forwards.

---

**10. (Medium)** Create a DV01-neutral 2s–10s steepener using PV01(2Y)=$200/bp per $1mm and PV01(10Y)=$900/bp per $1mm. Provide weights.

---

**11. (Medium)** For a basis swap, show how to replicate "receive IBOR, pay OIS" with two vanilla swaps with the same fixed rate.

---

**12. (Medium–Hard)** In a swap spread trade, list at least four non-parallel risk drivers that can produce P&L even when DV01-neutral.

---

**13. (Hard)** Given a term structure of basis spreads $e(T)$, compute deterministic rolldown P&L over a horizon for an at-par basis swap (state assumptions).

---

**14. (Hard)** Compare swap-spread signals computed using OTR yields vs fitted yields and propose a rule for ensuring time-series consistency.

---

**15. (Hard)** Define "key-rate PV01" in a par-point framework and explain how it differs from a zero-rate bump framework.

---

**16. (Hard)** For a DV01-neutral butterfly, derive the weight on the belly that makes net DV01 zero.

---

**17. (Hard)** Discuss how repo specialness can affect (i) Treasury yields and (ii) realized carry on a swap-spread package.

---

**18. (Very Hard)** In a multi-curve world with tenor basis, explain qualitatively how bumping the base index curve instruments differs from bumping basis swap instruments in terms of risk interpretation.

---

## Source Map

### (A) Verified Facts — Specific Sources

| Fact | Source |
|------|--------|
| OIS floating leg = compounded/geometric average of overnight rates | Andersen & Piterbarg Vol 1, Hull Ch 4 |
| LIBOR–OIS spread as short-term credit spread | Risk texts per Andersen Vol 1 discussion |
| Floating leg at par at inception (LIBOR flat) | Tuckman Ch 18 |
| Basis swap par condition with spread on specified leg | Andersen Vol 1 Ch 6, equation (6.49) |
| Swap spread = swap rate − gov yield; OTR distortions | Tuckman Ch 18 |
| Bond P&L = price change + interest income − financing cost | Tuckman Ch 15 (equation 15.8) |
| Butterflies = buy cheap maturity, sell wings | Tuckman Ch 6-7 |
| Par-point / bump-and-rebuild approach | Andersen Vol 3 |
| Tenor basis widened post-crisis | Andersen Vol 1 Ch 6 |
| On-the-run specialness and liquidity premiums | Tuckman Ch 15-16 |
| DV01 = $P \times D_{\text{mod}} / 10{,}000$ | Tuckman Ch 5-6, Hull Ch 4 |

### (B) Reasoned Inference — Derivation Logic

| Inference | Derivation |
|-----------|------------|
| Multi-curve par rate = discounted average of forwards | Set PV = 0 in generic swap PV formula, solve for K |
| Basis PV01 = $\pm N \sum \tau P_d \cdot 10^{-4}$ | Differentiate PV w.r.t. quoted spread e |
| Discount PV01 vs projection PV01 decomposition | Finite-difference bumping each curve separately, holding the other fixed |
| DV01-neutral hedge ratios | Equate DV01 exposures of offsetting positions |
| Replication of basis swap via two vanilla swaps | Cancel fixed legs at same rate, leaving net floating exchange |

### (C) Speculation — Flagged Uncertainties

| Topic | Uncertainty |
|-------|-------------|
| Balance-sheet constraints as spread drivers | Sources emphasize liquidity/financing but do not fully develop dealer balance-sheet constraint models |
| Desk-specific basis quoting conventions | Currency, indices, and market rules vary; convention stated for examples may not match reader's market |
| Forward wiggle contamination of basis RV | Standard implementation issue but not explicitly sourced in provided excerpts |

---

*Last Updated: January 2026*
