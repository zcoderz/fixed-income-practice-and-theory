# Chapter 46: Intrinsic Index Spread and Index Basis

---

## Conventions & Notation

| Convention | Description |
|------------|-------------|
| **Instrument focus** | CDS portfolio indices (e.g., CDX / iTraxx) treated as an equal-notional portfolio of single-name CDS unless explicitly stated otherwise |
| **Premium accrual & day count** | Premiums are typically paid quarterly and (in the reference) use Actual/360 for accrual |
| **Timing** | Trades are entered on trade date $t$ and are described as cash-settled shortly after trade date (e.g., "three days later" in the reference) |
| **Discounting** | $Z(t,u)$ is the (default-free) discount factor from $t$ to $u$. (The reference models the index PV using discounted expectations $Z(t,\cdot)$.) |
| **Credit modeling** | Each name $m$ has default time $\tau_m$, recovery $R_m$, and survival probability $Q_m(t,u) = P(\tau_m > u \mid \mathcal{F}_t)$ |
| **RPV01 definition (this chapter)** | $\text{RPV01}_m(t,T)$ is the risky PV01 / risky annuity for name $m$ from $t$ to $T$: the PV (per unit notional) of paying 1 unit of annual spread continuously (or its discretized equivalent) until default/maturity. In the reference, RPV01 is represented by $\int_t^T Z(t,u) Q_m(t,u) \, du$ under simplifying assumptions |
| **Protection buyer vs seller sign** | "Long protection" = protection buyer; benefits from spread widening. "Short protection" = protection seller; benefits from spread tightening |
| **Index basis sign convention (explicit)** | $b(t,T) \equiv S_{\text{quoted}}(t,T) - S_{\text{intrinsic}}(t,T)$ (in bp). So positive basis means the index is quoted wider than its bottom-up intrinsic level |

---

## 0. Setup

### Conventions Used in This Chapter

- The index has $M$ constituents, each with equal notional weight $1/M$ (this is the standard setup used in the primary reference's mechanics and valuation formulas)
- Index constituents remain fixed over the life of an issued index, except that defaulted names are removed without replacement. New index issues (new "rolls") can have different constituents
- Coupon $C(T)$ is the contractual index coupon set near inception and tends to be chosen as a multiple of 5 bp (per the reference)

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $t$ | Valuation time (trade date / pricing time) |
| $T$ | Index maturity (or "term") |
| $M$ | Number of index constituents |
| $m \in \{1, \ldots, M\}$ | Constituent index |
| $\tau_m$ | Default (credit event) time of name $m$ |
| $R_m$ | Recovery rate of name $m$ (fraction of par) |
| $Z(t,u)$ | Discount factor from $t$ to $u$ |
| $Q_m(t,u)$ | Survival probability to $u$ |
| $C(T)$ | Contractual index coupon (bp/yr) |
| $S_m(t,T)$ | Par CDS spread for name $m$ to maturity $T$ (bp/yr) |
| $\text{RPV01}_m(t,T)$ | Risky PV01 (risky annuity) for name $m$ to $T$ |
| $S_{\text{intrinsic}}(t,T)$ | Bottom-up intrinsic (implied) index spread level |
| $S_{\text{quoted}}(t,T)$ | Market quoted index spread level (index "level") |
| $b(t,T)$ | Index basis $= S_{\text{quoted}} - S_{\text{intrinsic}}$ (bp) |
| $U(t,T)$ | Upfront PV amount (per unit notional unless scaled) |

---

## 1. Core Concepts (Definitions First)

### 1.1 CDS Portfolio Index

**Formal Definition:**

A CDS portfolio index is a standardized CDS referencing a fixed portfolio of $M$ entities where premium and default-contingent payments are defined at the portfolio level, commonly with equal per-name notional $1/M$. The index can be traded in an "unfunded" CDS-like format, and (conceptually) can also be viewed as packaged with a floating-rate note to create a "bond format" quote.

**Intuition:**

It's a liquid way to take (or hedge) broad credit risk. You get diversification across names, standardized documentation, and centralized quoting/clearing practices (varies by market and time—verify for a specific trade).

**Trading/Risk Practice:**

Market participants frequently use indices to express macro credit views and to hedge portfolios of credit exposures because the index market can be deep and liquid relative to some single names.

---

### 1.2 Quoted Index Spread (Index "Level")

**Formal Definition:**

The quoted index spread is the market-quoted flat spread at which the index would be priced "as if it were a simple CDS contract" (investment-grade convention in the reference), paired with the index coupon $C$ and an upfront amount that makes PV consistent.

**Intuition:**

A single number summarizing "where the index trades" on a spread basis.

**Trading/Risk Practice:**

Traders often speak in terms of "index at 60 bp" even though the contract has a fixed coupon and potentially an upfront amount; the quoted spread is a convenient level for risk reporting and comparisons.

---

### 1.3 Intrinsic (Bottom-Up) Index Spread

**Formal Definition:**

The intrinsic index spread is the spread level implied by valuing the index from its constituents' CDS curves, i.e., by equating (i) the index PV implied by summing constituent PVs to (ii) the PV of the index treated as a single flat-spread instrument. The reference derives an equation linking these quantities and gives a close approximation as a risky-PV01-weighted average of constituent spreads.

**Intuition:**

"What the index spread should be if it were exactly the portfolio of its names, priced bottom-up."

**Trading/Risk Practice:**

Used for:
- Fairness checks (is the index rich/cheap to constituents?)
- Basis trades (index vs single-name portfolio)
- Building consistent pricing inputs for index options / tranche models (often requiring some adjustment so intrinsic aligns to market)

---

### 1.4 Index Basis

**Formal Definition:**

The index basis is the difference between the market quoted index spread and the intrinsic index spread:

$$b(t,T) = S_{\text{quoted}}(t,T) - S_{\text{intrinsic}}(t,T)$$

The reference documents that quoted and intrinsic spreads often differ and provides examples of positive and negative basis across indices/maturities.

**Intuition:**

A "wedge" capturing effects not explained by a frictionless replication of index = portfolio of CDS.

**Trading/Risk Practice:**

Basis affects hedging (index hedge for a single-name book is imperfect), valuation (how you calibrate), and risk attribution (spread vs basis P&L).

---

## 2. Intrinsic Index Spread: Definition and Computation

### 2.1 Quoted Index Spread vs Intrinsic Spread

**Quoted index spread $S_{\text{quoted}}(t,T)$:**

A market quote (often described as a flat spread level) used to price the index "as if it were a simple CDS," with an upfront adjustment when the contractual coupon differs from the market level.

**Intrinsic index spread $S_{\text{intrinsic}}(t,T)$:**

A model-implied spread computed bottom-up from constituent CDS curves/spreads via the intrinsic PV equality (exact) or an RPV01-weighted approximation (close).

---

### 2.2 Economic Meaning

**Intrinsic = bottom-up fair level:**

Intrinsic is the index level you would infer if you:
1. Value each constituent CDS using its own curve to produce PV and RPV01, then
2. Aggregate the PV across names, and
3. Solve for the single "flat" index spread that matches that PV

**Index basis = quoted − intrinsic:**

With sign convention $b = S_{\text{quoted}} - S_{\text{intrinsic}}$:
- $b > 0$: index quoted wider than bottom-up intrinsic
- $b < 0$: index quoted tighter than bottom-up intrinsic

---

### 2.3 Be Explicit About the Quoting Regime

The primary reference describes multiple quoting conventions for indices:

**Investment grade indices: spread quote ("flat spread as if simple CDS"):**

The market quotes an index spread level; when that spread differs from the contractual coupon, an upfront payment is implied.

**Other lower credit quality indices: "bond price" quote:**

Some indices are quoted using a bond price convention where the upfront value is computed by subtracting 100; the reference notes this avoids disagreement about the index PV01 needed to convert a spread quote into an upfront.

**Standard-coupon + upfront framework (supported in the sources):**

- At inception, the index coupon is set close to fair value (often a multiple of 5 bp), and the buyer makes an upfront payment equal to contract value (close to zero at issuance)
- For later trading, PV consistency uses an upfront amount plus the contractual coupon stream

If you want a pure "running-spread-only" regime (no fixed coupon, no upfront), I'm not sure how your target market's conventions should be implemented for indices based only on the provided sources; you'd need the relevant index rulebook / dealer convention note / ISDA definitions update to specify the trading convention. (The primary reference's intrinsic-vs-quoted discussion is explicitly framed around a contractual coupon and upfront equivalence.)

---

## 3. Math and Derivations (Step-by-Step)

### 3.1 Index PV from Constituents (Protection Leg and Premium Leg)

**Assumptions (as in the reference derivation):**

- Equal per-name notional $1/M$
- Recovery $R_m$ is treated as constant for valuation
- To reach an integral form, default and interest rates are treated as independent in the simplification step

#### 3.1.1 Protection Leg PV

For a long-protection position on the index, the discounted payoff from default of name $m$ before maturity $T$ is $(1 - R_m)/M$ at time $\tau_m$. The reference writes the protection leg PV as:

$$\text{PV}_{\text{prot}}(t,T) = \frac{1}{M} \sum_{m=1}^{M} \mathbb{E}_t \left[ Z(t,\tau_m) (1 - R_m) \mathbf{1}_{\{\tau_m < T\}} \right] \tag{PL}$$

Using the simplifying independence step in the reference, this becomes:

$$\text{PV}_{\text{prot}}(t,T) = \frac{1}{M} \sum_{m=1}^{M} (1 - R_m) \int_t^T Z(t,u) \, dQ_m(t,u) \tag{PL-int}$$

**Unit check:**

| Component | Unit |
|-----------|------|
| $Z(t,u)$ | Unitless discount factor |
| $dQ$ | Unitless probability increment |
| $(1 - R)$ | Unitless loss fraction |
| $\text{PV}_{\text{prot}}$ | Unitless PV fraction of notional (multiply by notional $N$ to get currency PV) |

#### 3.1.2 Premium Leg PV and Risky PV01 (RPV01)

Under the continuous approximation shown in the reference, the premium leg PV for contractual coupon $C$ (in decimal per year) is:

$$\text{PV}_{\text{prem}}(t,T) = \frac{C}{M} \sum_{m=1}^{M} \int_t^T Z(t,u) \, Q_m(t,u) \, du \tag{PR}$$

Define the constituent risky PV01 (risky annuity) as:

$$\boxed{\text{RPV01}_m(t,T) \equiv \int_t^T Z(t,u) \, Q_m(t,u) \, du}$$

This is exactly the object appearing in the premium leg integral.

**Unit check:**

- $du$ has units of years; hence RPV01 has units of years (PV-weighted expected life)
- If $C$ is in decimal per year, then $C \cdot \text{RPV01}$ is unitless PV fraction of notional

#### 3.1.3 Par Spread for Constituent CDS

The reference defines the constituent par spread $S_m(t,T)$ as:

$$S_m(t,T) = \frac{\text{PV}_{\text{prot},m}(t,T)}{\text{RPV01}_m(t,T)} \tag{par}$$

This implies $\text{PV}_{\text{prot},m}(t,T) = S_m(t,T) \cdot \text{RPV01}_m(t,T)$.

---

### 3.2 Intrinsic Index PV at Contractual Coupon

Using the par-spread relation, the reference shows the intrinsic value (upfront-equivalent PV) of the index at coupon $C(T)$ as:

$$\boxed{V_{\text{intrinsic}}(t,T) = \frac{1}{M} \sum_{m=1}^{M} \left( S_m(t,T) - C(T) \right) \text{RPV01}_m(t,T)} \tag{V}$$

**Sanity check:**

- If $S_m = C$ for all names, the intrinsic PV is 0
- If spreads widen ($S_m$ increases) holding coupon fixed, long protection value increases: $V_{\text{intrinsic}} \uparrow$

---

### 3.3 From Intrinsic PV to an Intrinsic Index "Spread Level"

#### 3.3.1 Index Quoted Spread and Upfront Mapping (Standard Coupon + Upfront)

The reference states that market convention can be represented by a flat index curve $S_I(t,T)$ and the "upfront value" is computed as:

$$U_{\text{index}}(t,T) = \left( S_I(t,T) - C(T) \right) \text{RPV01}_I(t,T) \tag{U}$$

where $\text{RPV01}_I$ is the risky PV01 computed from the index-implied curve.

#### 3.3.2 Exact Intrinsic Spread Definition (PV-Consistent)

The intrinsic index spread is the $S_I(t,T)$ that equates the constituent-implied intrinsic PV to the index-implied upfront:

$$\boxed{\frac{1}{M} \sum_{m=1}^{M} \left( S_m(t,T) - C(T) \right) \text{RPV01}_m(t,T) = \left( S_{\text{intrinsic}}(t,T) - C(T) \right) \text{RPV01}_I(t,T; S_{\text{intrinsic}})} \tag{Exact}$$

This is the reference's Equation 10.8 relationship (expressed in our notation).

**Practical implication:**

Because $\text{RPV01}_I$ depends on the index spread curve, the "exact" intrinsic spread generally requires solving a (mildly) nonlinear equation.

#### 3.3.3 Operational Approximation: RPV01-Weighted Average Spread

The reference provides a close approximation:

$$\boxed{S_{\text{intrinsic}}(t,T) \approx \frac{\sum_{m=1}^{M} S_m(t,T) \cdot \text{RPV01}_m(t,T)}{\sum_{m=1}^{M} \text{RPV01}_m(t,T)}} \tag{Approx}$$

**Derivation (step-by-step):**

1. Start from (Exact)

2. If $\text{RPV01}_I(t,T) \approx \frac{1}{M} \sum_m \text{RPV01}_m(t,T)$ and is weakly dependent on $S$, then (Exact) implies approximately:

$$\frac{1}{M} \sum_m (S_m - C) \text{RPV01}_m \approx (S_{\text{intr}} - C) \cdot \frac{1}{M} \sum_m \text{RPV01}_m$$

3. Multiply both sides by $M$ and expand:

$$\sum_m S_m \cdot \text{RPV01}_m - C \sum_m \text{RPV01}_m \approx S_{\text{intr}} \sum_m \text{RPV01}_m - C \sum_m \text{RPV01}_m$$

4. The coupon term cancels, giving:

$$S_{\text{intr}} \approx \frac{\sum_m S_m \cdot \text{RPV01}_m}{\sum_m \text{RPV01}_m}$$

**Sanity checks:**

- If all $\text{RPV01}_m$ equal, $S_{\text{intr}}$ is the simple average spread
- If one name has a shorter risky duration (smaller RPV01), its spread gets less weight—consistent with the intuition that very wide names are expected to pay spread for less time

---

## 4. Index Basis: What It Is and Why It Exists

### 4.1 Definition and Measurement

**Definition (restate):**

$$b(t,T) = S_{\text{quoted}}(t,T) - S_{\text{intrinsic}}(t,T)$$

**Source-backed fact:** The reference explicitly notes that quoted index spreads often differ from intrinsic index spreads, and provides numerical examples across indices and maturities (including positive and negative basis).

---

### 4.2 Taxonomy of Basis Drivers (Source-Backed)

#### (i) Documentation / Restructuring Clause Differences

The reference states that for the North American CDX index, protection is triggered by bankruptcy or failure to pay and restructuring is not included ("No-Re"), while the market standard for US single-name CDS uses Mod-Re, and No-Re spreads are typically about 5% lower, creating an immediate basis.

**Important conflict / uncertainty note:**

A secondary reference notes an "other complication" that for CDX NA IG "the definition of default applicable to the index includes restructuring whereas the definition for CDS contracts on the underlying companies may not."

These statements conflict. I'm not sure which documentation convention applies to your target market/series/date based solely on the provided books. To be certain, you would need the current index rulebook and the relevant ISDA Definitions / confirmation terms for:
- The index contract, and
- The single-name CDS used for constituents,

including the precise credit events and restructuring terms.

#### (ii) Liquidity / Supply–Demand / Technicals

The reference lists liquidity-related reasons for basis:

- The size and liquidity of the index market can embed a lower liquidity risk premium than less liquid single-name CDS
- The index is often the preferred instrument to express a broad credit view; it may "lead" the CDS market, especially in widening markets when investors use long protection index positions to hedge illiquid long-credit exposures

**Reasoned inference (built on these facts):**

If constituents are illiquid and have wide bid/ask spreads, an intrinsic computed from mid quotes can differ materially from an executable intrinsic (using bids/asks), mechanically producing an apparent basis. (The reference emphasizes liquidity premia and hedging flows; bid/ask dispersion is a direct microstructural channel.)

#### (iii) Composition and Roll Effects

Indices "roll every six months" and rolling involves selling the old and buying the new, creating P&L effects from:
- Changes in composition (replacement of names due to downgrade, liquidity decline, etc.), and
- Maturity extension by roughly six months, which can widen spreads if credit curves slope upward

These effects can change intrinsic vs quoted comparisons across "series" or between on-the-run and off-the-run indices (even if each is internally priced correctly).

#### (iv) Defaults and Removal of Constituents

- Defaulted names are removed without replacement
- After each default, the notional of the contract is reduced by $1/M$ and subsequent coupon payments shrink accordingly; accrued coupon at default is handled as part of premium leg mechanics

**Reasoned inference:**

As defaults occur, both the effective portfolio and the remaining premium base change, which can affect intrinsic computations vs quoted levels (especially when comparing indices with different default histories).

#### (v) Settlement Mechanics / Deliverable Effects (Limited Support)

The reference notes that cash settlement (auction) protocols were introduced to handle situations where derivative notional exceeds deliverable obligations, with a fallback to cash settlement determined by auction.

A secondary reference describes that in cash settlement an auction determines the post-credit-event value (e.g., "$35 per $100"), producing a payoff of face minus that value (e.g., $65 per $100).

I'm not sure how each index contract maps auction outcomes into index settlement cashflows for your exact trade without the specific index documentation (rulebook/confirmations).

---

## 5. Measurement & Risk (Only What Belongs in Chapter 46)

### 5.1 Define Index Basis Exposure

Let $b = S_{\text{quoted}} - S_{\text{intrinsic}}$. A first-order PV sensitivity (per unit notional) can be expressed via the index risky PV01:

$$\frac{\partial \text{PV}}{\partial b} \approx \text{RPV01}_I(t,T) \times 10^{-4} \quad \text{(PV fraction per 1 bp of basis)}$$

Multiply by notional $N$ for currency DV01.

**Sign depends on position:**

- Long protection index: PV increases when $S_{\text{quoted}}$ widens (basis widens if intrinsic held fixed)
- Short protection index: opposite

This is consistent with the reference's use of $(S - C) \cdot \text{RPV01}$ as the upfront-equivalent value mapping.

---

### 5.2 Risk Decomposition (Conceptual but Disciplined)

Consider a book combining an index position and constituent CDS hedges.

- **Broad/systematic spread move:** common movement across the credit market affecting many names together
- **Idiosyncratic constituent moves:** single-name deviations around the common move (earnings, downgrade risk, sector events)
- **Basis move:** movement of the index relative to the intrinsic aggregation of constituents (liquidity, doc differences, technical flow)

Mathematically, in a first-order (small-move) approximation:

$$\Delta \text{PV} \approx \underbrace{N_I \cdot \text{CS01}_I \cdot \Delta S_{\text{quoted}}}_{\text{index spread risk}} - \underbrace{\sum_m N_m \cdot \text{CS01}_m \cdot \Delta S_m}_{\text{constituent hedge risk}}$$

Basis risk emerges when $\Delta S_{\text{quoted}} \neq$ the constituent-implied move (e.g., the weighted average).

---

### 5.3 Hedging Mindset (Framework, Not Tips)

#### 5.3.1 Hedging an Index with Constituents (Bottom-Up Hedge)

**Goal:** Neutralize first-order spread exposure by offsetting index CS01 with constituent CS01s.

Start from the intrinsic approximation:

$$S_{\text{intrinsic}} \approx \frac{\sum_m S_m \cdot \text{RPV01}_m}{\sum_m \text{RPV01}_m}$$

implying that the effective spread exposure of the intrinsic index is naturally RPV01-weighted.

Construct hedges so that the portfolio of constituents matches the index's spread PV01/CS01.

**Residual risks (failure modes):**

- Basis changes (quoted index moves without corresponding constituent move)
- Documentation mismatch (e.g., restructuring clause)
- Liquidity mismatch (index is liquid; some names are not)
- Default mechanics: after a default, contract notional and premium base change by $1/M$

#### 5.3.2 Hedging Constituents with an Index (Proxy Hedge)

**Goal:** Use the liquid index to hedge a less liquid credit book.

- Works best when the book is "index-like" and basis is stable
- Residual risks include name-specific moves and mismatch between off-the-run/on-the-run constituents

#### 5.3.3 Validation Tests for Hedge Design

- Recompute CS01 under small bumps to spreads (index and constituents)
- Check default scenario impacts ("VOD risk"): the reference emphasizes careful management of VOD when managing books of index swaps

---

### 5.4 Portfolio Swap Adjustment (Model-Consistency Tool)

When you need a model where "intrinsic = quoted" (e.g., for index option or tranche pricing), the reference describes adjusting constituent curves so that adjusted intrinsic equals market quoted spread, noting the adjustment is somewhat arbitrary but should be stable/fast/reasonable.

---

## 6. Worked Examples (At Least 12 Numeric Examples)

### Global Numeric Conventions (Used in All Examples)

- Spreads in bp unless stated
- Convert bp to decimal per year by $s = \text{bp} \times 10^{-4}$
- Notional in \$mm unless stated
- Currency PV approximation for a small spread move:

$$\Delta \text{PV} \approx (\pm) \, \Delta S(\text{bp}) \times 10^{-4} \times \text{RPV01} \times N$$

(Sign $+$ for long protection, $-$ for short protection.)

---

### Example 1: Intrinsic Spread from Constituents (Simple Weighted Case)

**Goal:** 5 names, equal weights, given spreads and RPV01s → compute intrinsic.

**Inputs (to maturity $T$):**

- $M = 5$
- Constituent spreads $S_m$ (bp): $[50, 60, 70, 80, 90]$
- Constituent $\text{RPV01}_m$: $[4.4, 4.2, 4.0, 3.8, 3.6]$

**Step 1:** Compute weighted numerator $\sum S_m \cdot \text{RPV01}_m$:

| Name | $S_m$ | $\text{RPV01}_m$ | $S_m \times \text{RPV01}_m$ |
|------|-------|------------------|----------------------------|
| 1 | 50 | 4.4 | 220 |
| 2 | 60 | 4.2 | 252 |
| 3 | 70 | 4.0 | 280 |
| 4 | 80 | 3.8 | 304 |
| 5 | 90 | 3.6 | 324 |

Sum: $220 + 252 + 280 + 304 + 324 = 1380$

**Step 2:** Compute denominator $\sum \text{RPV01}_m = 4.4 + 4.2 + 4.0 + 3.8 + 3.6 = 20.0$

**Step 3:** Intrinsic spread (RPV01-weighted average):

$$S_{\text{intrinsic}} \approx 1380 / 20.0 = 69.0 \text{ bp}$$

**Output:** $S_{\text{intrinsic}} = 69.0$ bp.

---

### Example 2: Intrinsic Using RPV01 Weighting (Non-Equal)

**Goal:** Show how different risky durations change intrinsic.

**Inputs:**

- $S_m$ (bp): $[50, 60, 70, 80, 90]$
- $\text{RPV01}_m$: $[5.0, 4.8, 4.0, 3.0, 2.0]$

**Compute numerator:**

| Name | $S_m \times \text{RPV01}_m$ |
|------|----------------------------|
| 1 | $50 \cdot 5.0 = 250.0$ |
| 2 | $60 \cdot 4.8 = 288.0$ |
| 3 | $70 \cdot 4.0 = 280.0$ |
| 4 | $80 \cdot 3.0 = 240.0$ |
| 5 | $90 \cdot 2.0 = 180.0$ |

Sum numerator $= 1238.0$

Denominator $= 5.0 + 4.8 + 4.0 + 3.0 + 2.0 = 18.8$

**Intrinsic:**

$$S_{\text{intrinsic}} \approx 1238 / 18.8 = 65.8511 \text{ bp} \approx 65.85 \text{ bp}$$

**Interpretation:** High RPV01 names (longer expected premium stream) receive more weight; here they are the tighter-spread names, so intrinsic is pulled below the simple average (70 bp).

---

### Example 3: Index Basis Calculation

**Goal:** Given quoted index spread and computed intrinsic → compute basis and interpret sign.

**Assume:**

- $S_{\text{quoted}} = 72.0$ bp
- From Example 1: $S_{\text{intrinsic}} = 69.0$ bp

**Basis:**

$$b = 72.0 - 69.0 = 3.0 \text{ bp}$$

**Interpretation (with our sign):**

$b > 0$ means the index is quoted wider than bottom-up intrinsic.

---

### Example 4: Upfront vs Spread (Mechanics-Only, Standard Coupon + Upfront)

**Supported by sources:** Upfront mapping uses $(S - C) \cdot \text{RPV01}$.

**Inputs:**

- Notional $N = \$100$mm
- Contractual coupon $C = 60$ bp
- Quoted index spread $S_{\text{quoted}} = 72$ bp
- Index risky PV01 $\text{RPV01}_I = 4.5$

**Step 1:** Spread difference in bp:

$$S_{\text{quoted}} - C = 72 - 60 = 12 \text{ bp}$$

**Step 2:** Upfront as PV fraction of notional:

$$U = (12 \text{ bp}) \times 10^{-4} \times 4.5 = 12 \times 4.5 / 10{,}000 = 54 / 10{,}000 = 0.0054$$

So $U = 0.54\%$ of notional.

**Step 3:** Convert to dollars:

$$U_{\$} = 0.0054 \times 100{,}000{,}000 = \$540{,}000$$

**Output:** Upfront $= 0.54\%$ or $\$540$k on \$100mm.

**Consistency note:** Intrinsic must be computed under the same coupon $C$ and the same upfront convention; mixing regimes creates spurious basis.

---

### Example 5: Bid/Ask Dispersion Effect (Liquidity Driver)

**Goal:** Show how using mid vs bid/ask constituent quotes changes intrinsic.

**Inputs:**

- $\text{RPV01}_m = [4.4, 4.2, 4.0, 3.8, 3.6]$, sum $= 20$
- Bid/ask spreads (bp):

| Name | Bid | Ask |
|------|-----|-----|
| 1 | 49 | 51 |
| 2 | 58 | 62 |
| 3 | 68 | 72 |
| 4 | 78 | 82 |
| 5 | 88 | 92 |

**Compute intrinsic using bids:**

Numerator $= 49 \cdot 4.4 + 58 \cdot 4.2 + 68 \cdot 4.0 + 78 \cdot 3.8 + 88 \cdot 3.6$

$= 215.6 + 243.6 + 272.0 + 296.4 + 316.8 = 1344.4$

Intrinsic(bid) $= 1344.4 / 20 = 67.22$ bp

**Compute intrinsic using asks:**

Numerator $= 51 \cdot 4.4 + 62 \cdot 4.2 + 72 \cdot 4.0 + 82 \cdot 3.8 + 92 \cdot 3.6$

$= 224.4 + 260.4 + 288.0 + 311.6 + 331.2 = 1415.6$

Intrinsic(ask) $= 1415.6 / 20 = 70.78$ bp

**Output:** Bid/ask intrinsic range $[67.22, 70.78]$ bp, width $= 3.56$ bp.

**Interpretation:** Even before "true" basis, microstructure can move the computed intrinsic by several bp.

---

### Example 6: Doc Clause Driver (Restructuring) — Toy Numeric

**Source-backed driver:** No-Re vs Mod-Re can create an immediate basis; No-Re spreads are typically ~5% lower than Mod-Re in the reference.

**Assume:**

- Intrinsic computed from Mod-Re single-name spreads (Example 1): $S_{\text{intrinsic,ModRe}} = 69.0$ bp
- If the index contract effectively corresponds to No-Re, and No-Re is ~5% lower:

$$S_{\text{NoRe}} \approx 0.95 \times 69.0 = 65.55 \text{ bp}$$

- Assume market quoted index spread (No-Re) $S_{\text{quoted}} = 65.55$ bp

**Basis (quoted − intrinsic(Mod-Re)):**

$$b = 65.55 - 69.0 = -3.45 \text{ bp}$$

**Output:** $b = -3.45$ bp (index quoted tighter than the Mod-Re-implied intrinsic).

**Important:** Because sources conflict on whether restructuring is included for some indices (see Section 4), you must verify the exact credit event terms for your trade.

---

### Example 7: Defaulted Constituent Handling (Toy with Auction Final Price)

**Index mechanics source:** After default, contract notional reduces by $1/M$, premium amount reduces; accrued coupon is paid at default.

**Auction price mapping source (single-name CDS):** Auction determines bond value; payoff equals face minus that value (e.g., 35 → payoff 65).

**Inputs:**

- Index notional $N = \$100$mm
- $M = 5$ names (equal weights)
- One name defaults; auction final price $FP = 35$ per \$100 face (so recovery $R = 0.35$)
- Contractual coupon $C = 60$ bp (0.0060)
- Default occurs 2 months after last coupon date (accrual fraction $\Delta = 2/12$)

**Step 1:** Protection payment on the defaulted name (to protection buyer):

$$\text{Payoff} = N \times \frac{1}{M} \times (1 - R) = 100{,}000{,}000 \times \frac{1}{5} \times 0.65 = \$13{,}000{,}000$$

**Step 2:** Reduction in outstanding notional for future premiums:

$$N_{\text{after}} = N \times \frac{M-1}{M} = 100{,}000{,}000 \times \frac{4}{5} = 80{,}000{,}000$$

**Step 3:** Accrued premium due at default (paid by protection buyer to seller, per CDS convention):

$$\text{Accrued} \approx C_{\text{decimal}} \times N \times \frac{1}{M} \times \Delta = 0.0060 \times 100{,}000{,}000 \times \frac{1}{5} \times \frac{2}{12}$$

Compute:

- $0.0060 \times 100{,}000{,}000 = 600{,}000$ per year
- Per name $= 600{,}000 / 5 = 120{,}000$ per year
- Accrued over $2/12$: $120{,}000 \times 2/12 = 20{,}000$

**Output:**

- Default payoff: \$13.0mm (to protection buyer, from seller)
- Premium base after default: \$80.0mm
- Accrued premium on the defaulted slice (toy): \$20k

I'm not sure whether your specific index uses exactly this cashflow mapping without checking the index documentation and settlement protocol (especially the exact treatment of auction outcomes and whether settlement is physical or cash in your contract).

---

### Example 8: Composition/Roll Effect (Toy)

**Source-backed concept:** Indices roll every six months; P&L impact from composition change and maturity extension; constituents can change due to liquidity, downgrade, etc.

**Old series (same as Example 1):**

- Spreads $[50, 60, 70, 80, 90]$, RPV01 sum $= 20$
- Intrinsic $= 69.0$ bp

**New series (toy):** Replace one name, overall a bit wider:

- Spreads $[55, 65, 75, 85, 110]$
- Same RPV01 for simplicity

**Compute numerator:**

| Name | $S_m \times \text{RPV01}_m$ |
|------|----------------------------|
| 1 | $55 \cdot 4.4 = 242.0$ |
| 2 | $65 \cdot 4.2 = 273.0$ |
| 3 | $75 \cdot 4.0 = 300.0$ |
| 4 | $85 \cdot 3.8 = 323.0$ |
| 5 | $110 \cdot 3.6 = 396.0$ |

Sum $= 1534.0$

Intrinsic(new) $= 1534 / 20 = 76.7$ bp

**Difference:**

$$76.7 - 69.0 = 7.7 \text{ bp}$$

**Output:** New series intrinsic is 7.7 bp wider (toy), illustrating how roll/composition can alter intrinsic and therefore measured basis across series.

---

### Example 9: Hedging Index with Constituents (Beta Hedge via CS01/RPV01)

**Goal:** Choose hedge notionals so first-order spread risk is neutral.

**Assume:**

- Index notional $N_I = \$100$mm
- Index $\text{RPV01}_I = 4.1$ (toy)
- Constituents $M = 5$ with $\text{RPV01}_m = [4.4, 4.2, 4.0, 3.8, 3.6]$

**Step 1:** Index CS01 (PV change per 1 bp) for long protection:

$$\text{CS01}_I = N_I \times 10^{-4} \times \text{RPV01}_I = 100{,}000{,}000 \times 10^{-4} \times 4.1 = \$41{,}000/\text{bp}$$

**Step 2:** If we short protection on each name with equal notional $N_m$, the total CS01 magnitude is:

$$\sum_m (N_m \times 10^{-4} \times \text{RPV01}_m) = N_m \times 10^{-4} \times \sum_m \text{RPV01}_m = N_m \times 10^{-4} \times 20$$

Set equal to 41,000:

$$N_m \times 10^{-4} \times 20 = 41{,}000 \Rightarrow N_m = \frac{41{,}000}{20 \times 10^{-4}} = 20{,}500{,}000$$

So $N_m = \$20.5$mm per name.

**Output:** Hedge a \$100mm long-protection index position with \$20.5mm short protection per name (5 names) to match CS01 to first order.

---

### Example 10: Residual Basis Risk Under an Idiosyncratic Move

**Goal:** Scenario: one constituent widens a lot, index moves less → compute hedge P&L and residual.

**Positions:**

- Long protection index: $N_I = \$100$mm, $\text{RPV01}_I = 4.1$
- Short protection each constituent: $N_m = \$20.5$mm (from Example 9)

**Scenario:**

- Only Name 5 widens by $+100$ bp; others unchanged
- Index quoted spread widens by $+30$ bp (toy; reflects partial transmission + basis effects)

**Step 1:** Index P&L (long protection):

$$\Delta \text{PV}_I \approx +30 \times 41{,}000 = +\$1{,}230{,}000$$

**Step 2:** Hedge P&L on Name 5 (short protection; widening hurts):

First compute Name 5 CS01 magnitude:

$$\text{CS01}_5 = N_5 \times 10^{-4} \times \text{RPV01}_5 = 20.5\text{mm} \times 10^{-4} \times 3.6 = \$7{,}380/\text{bp}$$

Then:

$$\Delta \text{PV}_5 \approx -100 \times 7{,}380 = -\$738{,}000$$

Other names: 0.

**Net P&L:**

$$\Delta \text{PV}_{\text{net}} \approx 1{,}230{,}000 - 738{,}000 = +\$492{,}000$$

**Optional diagnostic: basis change (toy)**

- Start: $S_{\text{quoted}} = 72$, $S_{\text{intrinsic}} = 69$ ⇒ $b = 3$
- Intrinsic after Name 5 widens by 100 bp (using Example 1 weights):
  - Intrinsic increases by weight $3.6/20 = 0.18$ times 100 bp = 18 bp
  - $S_{\text{intrinsic,new}} = 87$ bp
- Quoted new: $72 + 30 = 102$ bp ⇒ new basis $= 102 - 87 = 15$ bp

**Interpretation:** Even if initial CS01 is hedged, idiosyncratic moves can create basis drift and residual P&L.

---

### Example 11: Basis Trade P&L Decomposition (Risk-First)

**Trade (toy):** "Long index protection vs short constituent protection" in a CS01-neutral way (Example 9).

- Long index: CS01 $= +41$k/bp
- Short constituents: total CS01 $= -41$k/bp

**Scenario suite:**

**(i) Parallel widening everywhere, no basis change**

- $\Delta S_{\text{quoted}} = +20$ bp
- Each name $\Delta S_m = +20$ bp

Index P&L: $+20 \times 41\text{k} = +\$820$k

Hedge P&L: $-20 \times 41\text{k} = -\$820$k

Net: \$0

→ This is the systematic component hedged away.

**(ii) Pure basis move: index tightens vs constituents**

- Constituents unchanged: $\Delta S_m = 0$
- Index quoted tightens by 10 bp: $\Delta S_{\text{quoted}} = -10$

Index P&L: $-10 \times 41\text{k} = -\$410$k

Hedge P&L: $0$

Net: $-\$410$k

→ This is basis P&L.

**(iii) Liquidity widening in single names only**

- Constituents widen by 15 bp: $\Delta S_m = +15$
- Index quoted unchanged: $\Delta S_{\text{quoted}} = 0$

Index P&L: $0$

Hedge P&L: $-15 \times 41\text{k} = -\$615$k

Net: $-\$615$k

→ This is a basis/technical effect (constituents move, index does not).

**Takeaway:** With a CS01-neutral construction, parallel moves disappear; remaining P&L comes from basis and relative moves.

---

### Example 12: Sanity Checks and Failure Example (Inconsistent Inputs)

**Goal:** Create a quote set implying an implausible intrinsic due to inconsistent units; show detection.

Start with Example 1 "correct" inputs, intrinsic $= 69$ bp.

**Failure:** Suppose Name 3 spread is mistakenly entered as 0.07 (bp) when the intended value was 70 bp (unit/decimal error).

**Inputs:**

- Spreads $[50, 60, 0.07, 80, 90]$ bp
- $\text{RPV01} = [4.4, 4.2, 4.0, 3.8, 3.6]$, sum $= 20$

**Compute numerator:**

| Name | $S_m \times \text{RPV01}_m$ |
|------|----------------------------|
| 1 | $50 \cdot 4.4 = 220$ |
| 2 | $60 \cdot 4.2 = 252$ |
| 3 | $0.07 \cdot 4.0 = 0.28$ |
| 4 | $80 \cdot 3.8 = 304$ |
| 5 | $90 \cdot 3.6 = 324$ |

Sum $= 220 + 252 + 0.28 + 304 + 324 = 1100.28$

**Intrinsic:**

$$S_{\text{intrinsic}} = 1100.28 / 20 = 55.014 \text{ bp} \approx 55.01 \text{ bp}$$

**Detection logic (robust checks):**

- Check each spread input is in a plausible range and consistent units (bp vs decimal)
- Check that the intrinsic is stable under small bumps; a single name's tiny spread should not dominate unless its RPV01 weight is enormous

I'm not sure what standardized "quote cleaning" rules your desk uses without a dealer convention note, but the minimum defensible checks are: unit sanity, missing data, stale quotes, and consistent day count/discounting inputs.

---

## 7. Practical Notes

### 7.1 Production Checklist (What You Need in Practice)

- **Index identity:** series, maturity, whether on-the-run vs off-the-run
- **Constituent list** and whether any names have defaulted (and hence are removed)
- **Constituent CDS curves/spreads** $S_m(t,T)$ and consistent recovery assumptions $R_m$
- **Discount curve** $Z(t,u)$ and premium accrual convention (e.g., quarterly, Act/360)
- **Contractual index coupon** $C(T)$ (multiple of 5 bp in the reference)
- **Quoting convention:**
  - Spread quote vs bond-price quote (and the spread-to-upfront conversion)
  - Documentation terms (credit events, restructuring clause, settlement type). I'm not sure without index rulebook + ISDA definitions for your trade; both are required.

---

### 7.2 Common Pitfalls

- Mixing quoting regimes (spread vs upfront vs bond price) inconsistently
- Using mismatched discounting across constituents vs index
- Using inconsistent recoveries across names (or mixing fixed vs stochastic recovery assumptions)
- Ignoring doc clause differences (e.g., restructuring inclusion/exclusion) that directly shift spreads
- Ignoring liquidity/bid–ask when interpreting basis; intrinsic computed from mid may not be executable
- Comparing off-the-run hedges to on-the-run indices with different constituents (mismatch risk)

---

### 7.3 Verification Tests

- **Weights sum correctly:** equal weights $1/M$, or (if using stated weights) verify they sum to 1
- **Scaling with notional:** PV and CS01 scale linearly with notional
- **Intrinsic recomputation** stable under small bumps to single-name spreads
- **Basis sign convention** consistent everywhere: $b = S_{\text{quoted}} - S_{\text{intrinsic}}$
- **Repricing consistency** if using PV equations: Confirm that plugging $S_{\text{intrinsic}}$ into the index PV mapping reproduces the constituent-implied upfront (within approximation error)

---

## 8. Summary & Recall

### 8.1 10-Bullet Executive Summary

1. A CDS index can be priced bottom-up by valuing constituent CDS and aggregating their PVs
2. The intrinsic index PV at coupon $C$ equals the average across names of $(S_m - C) \cdot \text{RPV01}_m$
3. The market quoted index "level" is represented as a flat spread $S_{\text{quoted}}$ for pricing as if it were a simple CDS
4. Under standard coupon + upfront, upfront value is mapped by $(S - C) \cdot \text{RPV01}$
5. The intrinsic index spread is found by matching intrinsic PV to the index PV mapping (solve Equation 10.8)
6. A close approximation is the RPV01-weighted average of constituent spreads
7. Index basis is $S_{\text{quoted}} - S_{\text{intrinsic}}$ and can be positive or negative in practice
8. Basis drivers include documentation differences (e.g., restructuring clauses), liquidity premia, and index technical/hedging flows
9. Index roll (every six months) changes composition and maturity, affecting intrinsic and comparisons across series
10. For model calibration (options/tranches), practitioners may adjust constituent curves to force intrinsic = quoted ("portfolio swap adjustment")

---

### 8.2 Cheat Sheet

**Intrinsic PV at coupon $C$:**

$$V_{\text{intrinsic}} = \frac{1}{M} \sum_m (S_m - C) \cdot \text{RPV01}_m$$

**Index upfront mapping (flat curve):**

$$U_{\text{index}} = (S - C) \cdot \text{RPV01}_I$$

**Exact intrinsic spread:** solve $V_{\text{intrinsic}} = U_{\text{index}}$

**Approx intrinsic spread:**

$$S_{\text{intrinsic}} \approx \frac{\sum_m S_m \cdot \text{RPV01}_m}{\sum_m \text{RPV01}_m}$$

**Basis:**

$$b = S_{\text{quoted}} - S_{\text{intrinsic}}$$

**First-order PV impact of 1 bp spread move (per notional $N$):**

$$\Delta \text{PV} \approx \pm (1 \text{ bp}) \times 10^{-4} \times \text{RPV01} \times N$$

**Driver taxonomy (source-backed):** doc/restructuring; liquidity premia; technical/flow; roll/composition; defaults

---

### 8.3 Flashcards (30)

**Q:** What is the intrinsic index spread?
**A:** The spread level implied by pricing the index bottom-up from constituent CDS curves and matching PV to a flat-spread index representation.

**Q:** What is the quoted index spread?
**A:** The market-quoted flat spread level used to price the index as if it were a simple CDS (in the reference's IG quoting description).

**Q:** Define index basis with a sign convention.
**A:** $b = S_{\text{quoted}} - S_{\text{intrinsic}}$ (bp).

**Q:** Why isn't intrinsic simply the average of constituent spreads?
**A:** Because high-spread names are expected to pay premiums for less time; RPV01 weighting lowers their effective weight.

**Q:** What is RPV01 conceptually?
**A:** The risky PV of paying 1 unit of annual spread until default/maturity; a risky annuity.

**Q:** State the approximation for intrinsic spread.
**A:** $S_{\text{intrinsic}} \approx \frac{\sum S_m \cdot \text{RPV01}_m}{\sum \text{RPV01}_m}$

**Q:** What creates an "immediate basis" per the primary reference?
**A:** Restructuring clause differences (No-Re vs Mod-Re) with No-Re spreads ~5% lower.

**Q:** How can liquidity create basis?
**A:** Index can embed lower liquidity risk premium and can lead market due to hedging flows.

**Q:** What happens to index notional after a default?
**A:** It is reduced by $1/M$, reducing future premium payments.

**Q:** What is "portfolio swap adjustment"?
**A:** Adjusting constituent spreads/curves so intrinsic equals quoted index spread; used for stable model calibration.

**Q:** Why does the index roll matter for basis comparisons?
**A:** Composition can change and maturity extends by ~6 months, altering spreads.

**Q:** In the reference, how is the index coupon chosen at inception?
**A:** Close to fair value and often a multiple of 5 bp.

**Q:** What is meant by "bond price" quote for an index?
**A:** A quoting convention where upfront is bond price minus 100; avoids PV01 disputes.

**Q:** How do you compute a first-order CS01 from RPV01?
**A:** $\text{CS01} \approx N \times 10^{-4} \times \text{RPV01}$ dollars per bp.

**Q:** What is VOD risk in index context (high level)?
**A:** Default-related jump risk; reference stresses special care managing VOD in index books.

**Q:** What's the PV-consistent equation for intrinsic spread?
**A:** Match intrinsic PV from constituents to index upfront PV under flat index curve.

**Q:** If all constituent spreads are equal, what is intrinsic spread?
**A:** It equals that common spread (by the weighted-average formula).

**Q:** If a name has a low RPV01, does it get more or less weight in intrinsic?
**A:** Less weight.

**Q:** If $b > 0$, is the index quoted wider or tighter than intrinsic?
**A:** Wider.

**Q:** Give one flow-based reason for basis per the reference.
**A:** In widening markets, investors use long protection index positions to hedge illiquid long-credit positions.

**Q:** What must be consistent when computing intrinsic and quoted levels?
**A:** Coupon $C$, discounting, recoveries, and the upfront/spread quoting regime.

**Q:** What happens to premium accrual at default?
**A:** Accrued premium is paid at default (premium-leg convention).

**Q:** What is the "intrinsic PV" of the index used for?
**A:** To infer intrinsic spread and measure basis vs market quote.

**Q:** Why can off-the-run vs on-the-run hedging be imperfect?
**A:** Different constituent sets across series.

**Q:** What is the payoff in a cash-settled single-name CDS after auction price $FP$?
**A:** $(1 - FP/100) \times$ notional (e.g., 35 → 65%).

**Q:** Does the primary reference mention auction protocols for cash settlement?
**A:** Yes, as a fallback when deliverables are scarce.

**Q:** What's the key numerical object that links spreads to PV?
**A:** RPV01.

**Q:** What's a quick "intrinsic" calculation if you only have $S_m$ and $\text{RPV01}_m$?
**A:** Compute the RPV01-weighted average of spreads.

**Q:** When might a "bond price" convention be used for indices?
**A:** For lower credit quality indices in the reference (e.g., EM/HY), to avoid PV01 disputes.

**Q:** What document do you need to be certain about restructuring clauses for your trade?
**A:** The index rulebook and applicable ISDA definitions/confirmations (not fully specified in the provided books).

---

## 9. Mini Problem Set (18 Questions)

*Questions 1–9 include brief solution sketches. Questions 10–18: no solutions provided.*

---

**1.** Compute intrinsic (RPV01-weighted) spread for 4 names with spreads $[40, 60, 80, 100]$ and RPV01 $[4.5, 4.0, 3.5, 3.0]$.

*Sketch:* Compute numerator $\sum S \cdot \text{RPV01}$ and divide by $\sum \text{RPV01}$.

---

**2.** Compute basis given $S_{\text{quoted}} = 90$ bp and $S_{\text{intrinsic}} = 86$ bp. Interpret sign.

*Sketch:* $b = 90 - 86 = +4$ bp; index quoted wider than intrinsic.

---

**3.** Upfront calculation: $C = 50$ bp, $S_{\text{quoted}} = 80$ bp, $\text{RPV01} = 4.2$, $N = \$200$mm. Compute upfront in \$.

*Sketch:* $U = (30 \times 10^{-4} \times 4.2) \times 200$mm $= (0.0126) \times 200$mm $= \$2.52$mm.

---

**4.** Bid/ask intrinsic band: two-name index; Name A bid/ask 90/110, Name B bid/ask 10/12, both RPV01=4.0. Compute intrinsic at bid-bid and ask-ask.

*Sketch:* Average spreads since equal RPV01; compute $(90 + 10)/2 = 50$ and $(110 + 12)/2 = 61$.

---

**5.** Weight intuition: explain qualitatively why a 1000 bp name should carry less weight in an index level than a 10 bp name.

*Sketch:* High spread implies higher default intensity; expected premium-paying life shorter, so lower RPV01 weight.

---

**6.** Hedge notionals: given index CS01 = \$60k/bp and three constituents have CS01 magnitudes $[20\text{k}, 15\text{k}, 10\text{k}]$ per bp per \$1mm notional, find constituent notionals that match the index CS01 using proportional scaling to those CS01s.

*Sketch:* Allocate notionals so sum equals 60k; e.g., scale vector $[20, 15, 10]$ to sum 60.

---

**7.** Default impact: index has $M = 125$, notional \$100mm, recovery 40%. Compute loss fraction of notional from one default (index-level loss on that slice).

*Sketch:* $(1 - R)/M = 0.6/125 = 0.0048 = 0.48\%$ of notional (matches reference-style arithmetic).

---

**8.** Roll effect: describe two reasons rolling from old to new index can cause P&L even if "credit market unchanged."

*Sketch:* Composition changes; maturity extends ~6 months (curve slope).

---

**9.** Basis driver: give one documentation-driven and one liquidity-driven basis reason from the primary reference.

*Sketch:* Doc: restructuring clause differences; liquidity: index embeds lower liquidity risk premium / leads market.

---

**10.** Show how (Exact) reduces to (Approx) under the approximation $\text{RPV01}_I \approx \frac{1}{M} \sum_m \text{RPV01}_m$.

---

**11.** If basis is persistently negative for an index, list three potential explanations consistent with the taxonomy in Section 4.

---

**12.** Explain why hedging an off-the-run index with the on-the-run index can leave residual risk.

---

**13.** Propose a stress test to detect sensitivity to a single name becoming distressed (spread jumps to 1500 bp).

---

**14.** How would you adjust intrinsic calculations if you had non-equal index weights $w_m$? (State a formula.)

---

**15.** In a cash settlement, how does auction final price affect payoff? Provide a numerical example.

---

**16.** Define "basis CS01" for a basis trade long index/short constituents.

---

**17.** Discuss how defaulted-name removal affects future premium payments and why that matters for PV.

---

**18.** Describe why model calibration for index options/tranches may require enforcing intrinsic = quoted.

---

## Source Map

### (A) Verified Facts — Source-Backed

| Fact | Source |
|------|--------|
| Index PV as sum of constituent protection/premium legs | O'Kane Ch 10 (Equation derivations) |
| RPV01 definition and integral form | O'Kane Ch 6-7, 10 |
| Intrinsic spread approximation as RPV01-weighted average | O'Kane Ch 10 (Eq 10.8 and discussion) |
| Basis = quoted − intrinsic with sign | O'Kane Ch 10 |
| Restructuring clause differences (No-Re vs Mod-Re) as basis driver | O'Kane Ch 10 |
| Liquidity/hedging flow as basis driver | O'Kane Ch 10 |
| Index roll every six months; composition changes | O'Kane Ch 9-10 |
| Defaulted names removed without replacement; notional reduces by $1/M$ | O'Kane Ch 9-10 |
| Coupon chosen as multiple of 5 bp | O'Kane Ch 9-10 |
| Portfolio swap adjustment for calibration | O'Kane Ch 10 |
| Auction cash settlement | O'Kane Ch 5; Hull Ch 25 |

### (B) Reasoned Inference — Derived from (A)

| Inference | Logic |
|-----------|-------|
| Bid/ask dispersion creates mechanical apparent basis | If intrinsic uses mid but execution uses bid/ask, difference emerges |
| CS01 formula from RPV01 and notional | Direct multiplication per definition |
| Basis P&L decomposition in hedge scenarios | First-order Taylor expansion of PV functions |

### (C) Speculation — Flagged Uncertainties

| Topic | Note |
|-------|------|
| Exact restructuring clause for specific index series | Sources conflict; I'm not sure without current index rulebook |
| Settlement cashflow mapping for specific index trades | Requires index documentation not fully specified in provided books |
| Desk-specific quote cleaning rules | Convention note required |
