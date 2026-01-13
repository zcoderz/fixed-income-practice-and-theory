# Chapter 27: Swap Spreads, Asset Swaps, and Swap-Curve Relative Value

---

## Conventions & Notation

### Rates & Units

| Symbol | Definition |
|--------|------------|
| Rates | In decimals (e.g., 5% = 0.05) |
| 1 bp | $= 0.0001$ |
| $f$ | Coupon frequency (payments per year) |
| $c/f$ | Coupon per period |
| $\Delta(t_{n-1}, t_n)$ | Year fraction (accrual factor) for period $n$; often $\Delta = 0.5$ for semiannual (pedagogical simplification; real swaps/bonds may differ by market) |

### Discount Factors and Zero Rates

| Symbol | Definition |
|--------|------------|
| $Z(0,t)$ | Discount factor to time $t$ |
| $r(0,t_n)$ | Discretely compounded zero rate to $t_n$ (when used) |
| Relation | $Z(0,t_n) = (1 + r(0,t_n)/f)^{-n}$ for $t_n = n/f$ |

### Swap Spread

| Symbol | Definition |
|--------|------------|
| $S_T$ | Par swap rate for maturity $T$ |
| $y_T^{gov}$ | Government bond yield used as benchmark for maturity $T$ (often on-the-run) |
| $\text{SS}_T$ | Swap spread at maturity $T$: $\text{SS}_T = S_T - y_T^{gov}$ |

### Bond Prices

| Symbol | Definition |
|--------|------------|
| $P_c$ | Clean bond price |
| $P$ | Full/dirty bond price |
| $\text{AI}$ | Accrued interest |
| Relation | $P = P_c + \text{AI}$ |

### Asset Swap Spreads

| Symbol | Definition |
|--------|------------|
| $A$ | Par asset swap spread (decimal per year) on the floating leg |
| $A^*$ | Market asset swap spread; $A^* = A/P$ |
| $\theta$ | ZVS (zero-volatility spread) over the Libor discount rate that reprices a bond |

### Other Notation

| Symbol | Definition |
|--------|------------|
| $c$ | Annual coupon rate (decimal) |
| $t_n$ | Payment date $n$ |
| $N$ | Final payment index |
| $T = t_N$ | Maturity |
| $P_{\text{Libor}}(0,T)$ | PV of bond's fixed cashflows using Libor/swap discount factors |
| $\text{PV01}(0,T)$ | Discounted accrual sum: $\sum_{n=1}^{N} Z(0,t_n) \Delta(t_{n-1}, t_n)$ |
| $\text{DV01}$ | Dollar value of 1 bp move: $\text{DV01} \equiv -\Delta P / (10{,}000 \cdot \Delta y)$ |

---

## Setup

### Conventions Used in This Chapter

- **Swap spreads** are defined as (par swap rate $-$ government yield) at the same quoted maturity, with the quoted benchmark typically the on-the-run government bond.
- We treat the swap curve and government curve as given inputs (no full bootstrapping in this chapter).
- **Asset swap discussion** uses two linked conventions:
  - **Par asset swap:** exchange of a bond for par within the structure; spread $A$ is solved so that valuation conditions hold.
  - **Market asset swap:** uses the bond's full price as swap notional; spread $A^*$ relates to $A$ via $A^* = A/P$.
- We explicitly flag where market practice varies (benchmark choice, discounting regime, par/par vs market/market).

---

## Core Concepts

### 1) Swap Spread

**Formal Definition:**

For a given maturity $T$,

$$\text{SS}_T \equiv S_T - y_T^{gov}$$

i.e., the difference between the swap rate and the government bond yield of that maturity.

**Benchmark Dependence (Built into the Definition):**

Tuckman notes a key practical issue: a "$T$-year swap" matures in exactly $T$ years, but often there is no government bond with exactly $T$ years to maturity; by convention, the "$T$-year swap spread" is quoted versus the $T$-year on-the-run government bond.

He also warns that on-the-run yields embed liquidity premiums and special financing, so swap spreads quoted off on-the-run benchmarks can be misleading in "the small."

**Intuition:**

- Swap rates reflect rolling short-term bank credit (e.g., LIBOR-based) and therefore should exceed government bond rates, so swap spreads "should certainly exceed government bond rates."
- But the government yield you subtract may be distorted by security-specific effects (on-the-run richness, repo specialness), which contaminates the interpretation.

**How It Appears in Trading/Risk/Portfolio Practice:**

- A widely watched indicator of relative pricing between swaps and government bonds.
- Used in relative value (RV) trades that are long one curve and short the other, but practitioners must manage benchmark choice and basis risk (swap vs Treasury).

---

### 2) Asset Swap and Asset Swap Spread (ASW)

**Formal Definition (as a Spread Measure):**

Tuckman (spread measure framing): "The asset swap spread of a bond is the spread such that discounting the bond's cash flows by swap rates plus that spread gives the bond price."

This is a spread-to-swap-curve concept: a constant spread applied (in some convention) so the swap-anchored valuation matches market price.

O'Kane (contract + PV framing): In a par asset swap, the spread $A$ is chosen so that the swap-plus-bond structure satisfies a par/valuation condition; it leads to

$$A(0) = \frac{P_{\text{Libor}}(0,T) - P}{\text{PV01}(0,T)}$$

with $P$ the bond full price, $P_{\text{Libor}}$ the "Libor-curve" PV of the bond's fixed cashflows, and $\text{PV01}$ the discounted accrual sum.

**Intuition:**

- ASW is an attempt to express "how much extra spread over swaps" a bond offers, i.e., a bond's value/credit relative to the swap curve, not relative to Treasuries.
- Tuckman emphasizes that asset swap spreads mostly measure a bond's credit risk relative to the credit risk embedded in the swap curve, but can also include security-specific effects (special financing, supply/demand).
- Hull (via *Risk Management and Financial Institutions*) describes asset swaps as giving "direct estimates of the excess of bond yields over LIBOR/swap rates," and explains the standard upfront adjustment if the bond price is not par.

**How It Appears in Trading/Risk/Portfolio Practice:**

- Buy bond vs swap packages to isolate a bond's "spread to swaps," often financing the bond in repo. Tuckman's schematic example: buy an agency bond, repo it, and enter a swap paying the bond coupon and receiving LIBOR + ASW; coupons cancel fixed swap payments, leaving a net floating stream linked to (ASW + LIBOR $-$ repo).
- **Important risk point:** if the bond defaults, bond coupons stop but swap fixed payments remain due (bond default does not cancel swap obligations).

---

### 3) Swap-Curve Relative Value (RV): Anchoring Bonds to Swaps

**Formal Definition (Trader's Lens):**

"Swap-curve RV" is not a single formula; it is the practice of comparing bond value to the swap curve (swap rates / swap discount curve) rather than to Treasuries, often via ASW or other swap-anchored spreads.

**Intuition:**

- Tuckman motivates using non-security-specific curves (e.g., TED for short maturities, ASW for longer maturities) to avoid distortions from individual Treasuries.
- O'Kane similarly argues that swap spreads are not "pure credit spreads" because government yields are affected by government market supply/demand, while Libor-based benchmarks remove many of those effects.

**How It Appears in Trading/Risk/Portfolio Practice:**

Trades such as "bond vs swap," "spread-of-spreads," and "rich/cheap vs swaps," where the goal is to:
1. Neutralize most rate exposure, and
2. Take a view on relative spread (ASW) or basis (Treasury vs swap).

Tuckman's on-the-run/off-the-run "spread of spreads" case study exemplifies how desks use swaps and repo to express RV between specific Treasuries and the swap curve.

---

### 4) Why Swap Spreads Are Not "Pure Credit Spreads"

**Formal Statement (Source-Consistent Drivers):**

- Swap rates reflect bank credit embedded in LIBOR-type funding ("three-month rolling bank credit"), but the government benchmark yield can be heavily affected by liquidity premiums and special financing in on-the-run bonds.
- Hence quoted swap spreads mix: bank credit + swap market supply/demand + Treasury market security-specific effects.

**Intuition:**

A widening swap spread can come from: swap rates up (bank credit/liquidity) or Treasury yields down (on-the-run specialness/flight-to-quality), even if bank credit is unchanged.

**Practice:**

Desks sometimes compute swap spreads vs a fitted off-the-run government curve or "financing/liquidity-adjusted" on-the-run yields, but these require subjective judgment and are not widely standardized.

---

## Math and Derivations

### 1) Swap Spread: Definition and Benchmark Variants

**Definition (Quoted Convention):**

$$\text{SS}_T = S_T - y_T^{gov}$$

Tuckman: conventionally $y_T^{gov} = y_T^{\text{OTR}}$ for maturity $T$.

**Benchmark Variants (Must Be Stated in Any Calculation):**

| Variant | Description |
|---------|-------------|
| On-the-run benchmark | $y_T^{gov} = y_T^{\text{OTR}}$ (widely quoted) |
| Fitted/off-the-run curve benchmark | $y_T^{gov}$ comes from a fitted off-the-run government curve to reduce on-the-run liquidity/specials distortion |
| Financing/liquidity adjusted on-the-run | Subtract an estimate of specialness/liquidity from OTR yield |

**Sanity/Unit Checks:**

- If $S_T$ and $y_T^{gov}$ are in decimals, $\text{SS}_T$ is in decimals. Convert to bp by $\times 10{,}000$.
- Typical sign: $S_T > y_T^{gov} \Rightarrow \text{SS}_T > 0$ (but can be negative in some market episodes; this chapter does not assume sign).

---

### 2) Asset Swap Mechanics: Contract View and PV Equations (Par Asset Swap)

We present the O'Kane-style derivation because it yields a clean PV equation for $A$.

**Step 1: Clean vs Dirty (Full) Bond Price**

Bond trades with a quoted clean price $P_c$ and accrued interest AI; the settlement/full price is

$$P = P_c + \text{AI}$$

**Step 2: Fixed Leg PV of the Bond Cashflows on the Libor/Swap Discount Curve**

Define

$$P_{\text{Libor}}(0,T) \equiv \frac{c}{f} \sum_{n=1}^{N} Z(0,t_n) + Z(0,t_N)$$

i.e., the PV of the bond's fixed coupons plus principal using the Libor/swap discount factors.

**Step 3: PV01 (The Discounted Accrual "Annuity")**

Define

$$\text{PV01}(0,T) \equiv \sum_{n=1}^{N} Z(0,t_n) \, \Delta(t_{n-1}, t_n)$$

**Unit check:** $Z$ is dimensionless; $\Delta$ is in years $\Rightarrow$ PV01 has units of "discounted years."

If $A$ is in decimals per year, $A \cdot \text{PV01}$ is dimensionless PV (price-per-par).

If $A$ is quoted in bp, use $(A_{\text{bp}}/10{,}000) \cdot \text{PV01}$.

**Step 4: Par Asset Swap Value and the Equation That Pins Down $A$**

O'Kane gives the swap value (par asset swap) as

$$V(0) = 1 + A(0) \sum_{n=1}^{N} Z(0,t_n) \Delta(t_{n-1}, t_n) - \left( \frac{c}{f} \sum_{n=1}^{N} Z(0,t_n) + Z(0,t_N) \right)$$

i.e.,

$$V(0) = 1 + A(0) \, \text{PV01}(0,T) - P_{\text{Libor}}(0,T)$$

The par asset swap is structured so that

$$V(0) + P = 1$$

which yields the asset swap spread

$$\boxed{A(0) = \frac{P_{\text{Libor}}(0,T) - P}{\text{PV01}(0,T)}}$$

**Interpretation:**

- If $P < P_{\text{Libor}}$ (bond is "cheap" vs the Libor/swap curve), then $A > 0$.
- If $P > P_{\text{Libor}}$ (bond is "rich" vs the curve), then $A < 0$. Negative ASW spreads can occur for very high quality issuers relative to AA-bank funding (also noted in the credit basis discussion).

---

### 3) Market Asset Swap: Notional = Full Price and the $A^*$ Convention

O'Kane also describes a "market asset swap" whose notional equals the bond full price $P$, and shows the relationship:

$$\boxed{A^*(0) = \frac{A(0)}{P} = \frac{A(0)}{P_c + \text{AI}}}$$

**Why This Matters (Intuition):**

In the par asset swap, exchanging a bond worth $P$ for par $1$ creates a counterparty exposure proportional to $(1 - P)$ (premium/discount). Market asset swap uses notional $P$ and includes a final payment adjustment $(P - 1)$, reducing that exposure.

---

### 4) Z-Spread / ZVS to the Libor (Swap) Curve and Its Relation to ASW

O'Kane defines the ZVS (zero-volatility spread) $\theta$ as the fixed spread adjustment to the Libor discount rate that reprices the bond.

**Discrete-Compounding Definition:**

$$P = \frac{c}{f} \sum_{n=1}^{N} \frac{1}{(1 + (r(0,t_n) + \theta)/f)^n} + \frac{1}{(1 + (r(0,t_N) + \theta)/f)^N}$$

with $Z(0,t_n) = (1 + r(0,t_n)/f)^{-n}$.

**Continuous-Compounding Alternative (Also Used in Practice):**

$$P = \frac{c}{f} \sum_{n=1}^{N} Z(0,t_n) e^{-\theta t_n} + Z(0,t_N) e^{-\theta T}$$

**Conceptual Mapping (Why ASW $\neq$ ZVS in General):**

- **ZVS** is a discounting spread: add $\theta$ to the discount rate for all cashflows.
- **ASW** $A$ is a swap-structure spread: a spread on a floating leg in a package designed to align the bond with a par/floating benchmark.

They can be numerically close for simple fixed-rate bonds under single-curve assumptions, but differ when:
1. Curve is not flat (Tuckman notes equality to "yield minus swap rate" only in special flat cases),
2. Conventions differ (clean/dirty inputs, par/par vs market/market),
3. Multi-curve discounting/projection is used (OIS discounting vs Libor forwards),
4. Funding/financing enters the realized P&L of the trade (repo specialness, etc.).

---

## Measurement & Risk

### 1) Swap Spread: Precise Definition + Benchmark Choice

**Definition (Must State Benchmark):**

$$\text{SS}_T = S_T - y_T^{gov}$$

with quoted convention $y_T^{gov} = y_T^{\text{OTR}}$ for maturity $T$.

**Benchmark Choices and Why They Matter:**

| Benchmark | Characteristic |
|-----------|----------------|
| On-the-run | Easy/standard, but contaminated by liquidity/specials |
| Fitted/off-the-run curve | Aims to remove on-the-run idiosyncrasies; less standardized and more judgmental |

**Why Swap Spreads Are Not "Pure Credit Spreads":**

- Swap spreads embed banking credit (LIBOR-based rolling bank credit) and Treasury scarcity/special financing/liquidity effects.
- Historical example: Tuckman attributes early-1990s narrowing to bank sector recovery and late-1990s widening to perceived scarcity of U.S. Treasuries.
- Therefore swap spreads are a joint outcome of credit + supply/demand + microstructure.

---

### 2) Asset Swap and ASW Spread: Contract, Clean/Dirty, and Conventions

**Contract Description (Bond + Swap Overlay):**

Practical "bond vs swap" package: buy a bond and enter a swap paying the bond coupon and receiving floating LIBOR + ASW, often financing bond via repo so coupons offset fixed swap payments.

**Default linkage:** bond default stops bond coupons, but swap fixed payments remain due.

**Clean vs Dirty Price and Accrued:**

Always specify whether input price is clean $P_c$ or full $P$; full price is $P = P_c + \text{AI}$.

**Two Common ASW Definitions/Conventions (Must Be Stated):**

| Convention | Definition |
|------------|------------|
| Par asset swap spread $A$ (O'Kane) | $A = (P_{\text{Libor}} - P) / \text{PV01}$ |
| Market asset swap spread $A^*$ (O'Kane) | $A^* = A/P$, with notional equal to bond full price |

**Related market practice note:** Hull's description matches the common "upfront adjustment" intuition: if a bond trades below (above) par, one side pays an upfront amount so the swap still exchanges coupons vs LIBOR+spread thereafter.

---

### 3) Swap-Curve RV: Bond vs Swaps and Mapping Across Spread Measures

**Bond vs Swap Curve Anchoring:**

Tuckman: market participants use asset swap spreads for longer-term bonds to measure value relative to curves "not contaminated by individual security effects."

**Swap Spread vs ASW vs ZVS (Conceptual Map):**

| Measure | What It Compares |
|---------|------------------|
| Swap spread | Swaps vs government yields (benchmark-sensitive) |
| ASW | Bond vs swap curve (as spread on floating leg in a package) |
| ZVS | Constant spread $\theta$ added to Libor discount rate to reprice bond |

They differ because they compare to different benchmarks (gov vs swap), use different mechanics (bond+swap package vs discounting spread), and can use different curves (single-curve vs multi-curve).

---

### 4) Risk & P&L Explain (Rates vs Spread vs Basis/Funding)

**Rates DV01/PV01 vs "Spread Exposure" in ASW Packages:**

- DV01 is the price change per 1 bp move in the chosen rate factor.
- Hedging sizes can be set using DV01 ratios (general hedge sizing formula).
- Tuckman emphasizes that an asset swap's P&L comes from:
  - Rate moves because bond DV01 differs from the fixed leg DV01 when bond yield differs from swap rate, and
  - ASW spread changes.

**Basis Risk:**

Even if swaps hedge rates "better" than Treasuries for some credit exposures, substantial basis risk remains (swaps reflect banking credit and cannot hedge issuer-specific credit spread changes).

Treasury vs swap basis is central to swap spread RV.

**Financing/Repo Effects (Brief, Desk-Relevant):**

- In Tuckman's asset swap example, the investor's net floating receipts are linked to "LIBOR + ASW $-$ repo," and he interprets ASW as bond-vs-swap-curve credit while "LIBOR $-$ repo" reflects the swap-curve credit/funding component.
- For repo-financed bonds, realized return depends on repo rate and accrued interest conventions; Tuckman's repo P&L expression explicitly accounts for clean price and accrued interest.

---

## Worked Examples

> **Reminder:** Every example states its benchmark/conventions. Numbers are illustrative.

### Example A: Swap Spread Computation; Benchmark Sensitivity

**Given:**

- 5Y par swap rate: $S_5 = 3.20\%$
- 5Y government yield benchmarks:
  - (i) on-the-run: $y_5^{\text{OTR}} = 2.95\%$
  - (ii) fitted/off-the-run curve: $y_5^{\text{fit}} = 3.05\%$

**Compute:**

**On-the-run swap spread (quoted convention):**

$$\text{SS}_5^{\text{OTR}} = 3.20\% - 2.95\% = 0.25\% = 25 \text{ bp}$$

**Fitted-curve swap spread (desk-adjusted alternative):**

$$\text{SS}_5^{\text{fit}} = 3.20\% - 3.05\% = 0.15\% = 15 \text{ bp}$$

**Interpretation:**

The "same" swap spread differs by 10 bp purely from benchmark choice, illustrating Tuckman's warning about on-the-run distortions.

---

### Example B: Swap Spread Curve: Two Maturities, Slope Interpretation

**Given:**

- 5Y: $S_5 = 3.20\%$, $y_5^{\text{OTR}} = 2.95\%$
- 10Y: $S_{10} = 3.80\%$, $y_{10}^{\text{OTR}} = 3.60\%$
- Benchmark: on-the-run yields (quoted)

**Compute swap spreads:**

$$\text{SS}_5 = 3.20\% - 2.95\% = 25 \text{ bp}$$
$$\text{SS}_{10} = 3.80\% - 3.60\% = 20 \text{ bp}$$

**Swap spread curve slope (10Y $-$ 5Y):**

$$\Delta \text{SS} = 20 \text{ bp} - 25 \text{ bp} = -5 \text{ bp}$$

**Interpretation:**

"Swap spread curve" is downward sloping from 5Y to 10Y in this snapshot (tightening with maturity).

Drivers need not be "credit only" because Treasury-side benchmark can move for liquidity/specials reasons.

---

### Example C: Bond Spread to Swaps vs to Gov: Two Curve Anchors

**Goal:** Show that "spread to gov" and "spread to swaps" differ because the base curves differ.

**Given:**

- A 5Y bond has yield-to-maturity $y_{\text{bond}} = 3.70\%$
- 5Y on-the-run government yield $y_5^{\text{OTR}} = 3.00\%$
- 5Y par swap rate $S_5 = 3.20\%$
- We treat these as given quotes (no curve build)

**Compute:**

**Yield spread to gov (OTR)** (a "G-spread-like" notion; name varies by desk):

$$\text{Spread}_{\text{gov}} = y_{\text{bond}} - y_5^{\text{OTR}} = 3.70\% - 3.00\% = 70 \text{ bp}$$

**Yield spread to swaps (proxy):**

$$\text{Spread}_{\text{swap}} = y_{\text{bond}} - S_5 = 3.70\% - 3.20\% = 50 \text{ bp}$$

**Interpretation:**

The same bond looks "70 over gov" but "50 over swaps" because swap rates incorporate banking-credit/funding while gov yields can include security-specific richness.

If you need a term-structure-consistent measure vs swaps, you typically move from this proxy to ASW or ZVS (Examples E–F).

> **I'm not sure** whether your desk's terminology would call these "G-spread" vs "I-spread" without seeing the specific definition used in your reference set (some desks use "I-spread" for interpolated gov curve spread). To be certain, I'd need the exact convention (government curve source + interpolation rule + whether it's yield spread or Z-spread style).

---

### Example D: ASW Setup Cashflows: Bond + Swap Overlay, 3 Periods

**Convention (Tuckman-style "bond + swap" picture):** buy bond, finance in repo, enter a swap paying the bond coupon and receiving LIBOR + ASW; bond coupons cancel swap fixed payments.

We use a simplified semiannual schedule and ignore fixed/float convention differences (as Tuckman does).

**Given (per $100 notional):**

- Bond coupon: $c = 5\%$ annual, paid semiannually $\Rightarrow$ bond coupon cashflow each period = $c/2 = 2.50$
- ASW spread: $A = +0.20\%$ (20 bp) on the floating leg (illustrative)
- LIBOR resets (semiannual, simple): $L_{0.5} = 3.00\%$, $L_{1.0} = 3.20\%$, $L_{1.5} = 3.10\%$
- Repo rate (semiannual simple): $r_{\text{repo}} = 2.70\%$
- Accrual per period: $\Delta = 0.5$

**Cashflows over first 3 semiannual periods:**

| Date | Bond coupon received | Swap fixed paid | Swap float received | Repo interest paid | Net cashflow |
|------|---------------------|-----------------|--------------------|--------------------|--------------|
| 0.5y | $+2.50$ | $-2.50$ | $+100 \cdot (3.00\% + 0.20\%) \cdot 0.5 = +1.60$ | $-100 \cdot 2.70\% \cdot 0.5 = -1.35$ | $+0.25$ |
| 1.0y | $+2.50$ | $-2.50$ | $+100 \cdot (3.20\% + 0.20\%) \cdot 0.5 = +1.70$ | $-1.35$ | $+0.35$ |
| 1.5y | $+2.50$ | $-2.50$ | $+100 \cdot (3.10\% + 0.20\%) \cdot 0.5 = +1.65$ | $-1.35$ | $+0.30$ |

**Interpretation:**

- The coupon-vs-fixed leg cancels (by design), leaving a net that resembles "(LIBOR + ASW $-$ repo)" cashflow logic highlighted by Tuckman.
- If the bond defaults, coupons stop but swap fixed remains due (big tail risk for the package).

---

### Example E: Solve for Asset Swap Spread $A$ Using the PV Equation; Repricing Check

We now use the O'Kane par asset swap formula.

**Conventions:**

- Use full price $P = P_c + \text{AI}$
- Coupon frequency $f = 2$; $\Delta = 0.5$ each period
- Discount factors $Z(0,t_n)$ are given (single-curve Libor/swap discounting)

**Given:**

- Bond: maturity $T = 3$y, semiannual coupons, annual coupon rate $c = 5\%$ $\Rightarrow$ coupon per period $c/f = 0.025$ (per par 1)
- Clean price $P_c = 1.0470$, accrued interest $\text{AI} = 0.0030$ $\Rightarrow$ full price:

$$P = 1.0470 + 0.0030 = 1.0500$$

- Payment dates: $t_n = \{0.5, 1.0, 1.5, 2.0, 2.5, 3.0\}$ with discount factors:

$$Z = \{0.985, 0.970, 0.954, 0.938, 0.922, 0.905\}$$

**Step 1: Compute $P_{\text{Libor}}(0,T)$**

Sum of discount factors:

$$\sum Z = 0.985 + 0.970 + 0.954 + 0.938 + 0.922 + 0.905 = 5.674$$

Then

$$P_{\text{Libor}} = \frac{c}{f} \sum Z + Z(0,T) = 0.025 \cdot 5.674 + 0.905 = 0.14185 + 0.905 = 1.04685$$

**Step 2: Compute $\text{PV01}(0,T)$**

$$\text{PV01} = \sum Z \cdot \Delta = 5.674 \cdot 0.5 = 2.837$$

**Step 3: Solve for $A$**

$$A = \frac{P_{\text{Libor}} - P}{\text{PV01}} = \frac{1.04685 - 1.0500}{2.837} = \frac{-0.00315}{2.837} = -0.001110$$

Convert to bp:

$$A_{\text{bp}} = -0.001110 \times 10{,}000 \approx -11.10 \text{ bp}$$

**Repricing Check (Show the PV Identity):**

O'Kane's swap value formula: $V(0) = 1 + A \cdot \text{PV01} - P_{\text{Libor}}$.

Compute $A \cdot \text{PV01} = -0.001110 \cdot 2.837 = -0.00315$. Then

$$V(0) = 1 - 0.00315 - 1.04685 = -0.05000$$

Check the par-asset-swap condition $V(0) + P = 1$:

$$V(0) + P = -0.05000 + 1.05000 = 1.00000 \; \checkmark$$

**Interpretation:**

- Bond is rich vs Libor/swap curve here, so $A < 0$.
- **Sign check:** from $A = (P_{\text{Libor}} - P)/\text{PV01}$, if $P$ falls (bond cheapens), $A$ rises (ASW widens).

---

### Example F: ASW vs ZVS Comparison on the Same Bond

We compute ZVS $\theta$ and compare to ASW $A$.

**Given (Use Example E Inputs):**

- Full price $P = 1.0500$, coupon $c = 5\%$, $f = 2$, same $Z(0,t_n)$

**ZVS Definition (Continuous Version):**

$$P = \frac{c}{f} \sum_{n=1}^{N} Z(0,t_n) e^{-\theta t_n} + Z(0,t_N) e^{-\theta T}$$

**Step 1: Evaluate PV at $\theta = 0$**

That is exactly $P_{\text{Libor}} = 1.04685$ (Example E). Since $P = 1.0500 > P_{\text{Libor}}$, we need $\theta < 0$ (a negative spread increases PV).

**Step 2: Trial $\theta = -0.00106$ ($\approx -10.6$ bp)**

Compute $e^{-\theta t} = e^{0.00106 t}$:

| $t$ | $e^{0.00106 t}$ |
|-----|-----------------|
| 0.5 | $\approx 1.000530$ |
| 1.0 | $\approx 1.001061$ |
| 1.5 | $\approx 1.001592$ |
| 2.0 | $\approx 1.002122$ |
| 2.5 | $\approx 1.002653$ |
| 3.0 | $\approx 1.003186$ |

Adjust discounted factors $Z \cdot e^{-\theta t}$ (rounded):

| $Z(0,t_n)$ | Adjusted |
|------------|----------|
| 0.985 | 0.985522 |
| 0.970 | 0.971029 |
| 0.954 | 0.955518 |
| 0.938 | 0.939989 |
| 0.922 | 0.924446 |
| 0.905 | 0.907884 |

Sum adjusted $Z$: $5.684388$

Compute PV:

- Coupon PV: $(c/f) \sum = 0.025 \cdot 5.684388 = 0.142110$
- Principal PV: $0.907884$
- Total: $0.142110 + 0.907884 = 1.049994 \approx 1.0500$

So $\boxed{\theta \approx -10.6 \text{ bp}}$.

**Compare:**

| Measure | Value |
|---------|-------|
| ASW from Example E | $A \approx -11.1$ bp |
| ZVS here | $\theta \approx -10.6$ bp |

**Why They Can Differ (Structural Reasons):**

- ZVS is a discount-rate shift applied to all cashflows; ASW is solved from a par-asset-swap PV identity involving a floating-leg spread and PV01.
- Under simple single-curve, fixed-rate, no-optionality settings, they can be close; differences expand with curve shape, conventions, and multi-curve discounting.

---

### Example G: Impact of OIS Discounting on ASW — Sensitivity Experiment

**What is source-backed?** Multi-curve practice often builds separate curves for discounting (e.g., OIS) and projection (e.g., LIBOR).

> **I'm not sure** we can rigorously recompute ASW under OIS discounting without specifying (i) the projection curve for LIBOR, (ii) collateral/CSA terms, and (iii) the exact market instrument set used to build OIS vs LIBOR curves. The books confirm multi-curve separation, but the full desk convention set is not pinned down here.

So below is a **clearly labeled sensitivity experiment:**

**Assumption (Toy):**

- Keep everything from Example E the same except replace discount factors $Z$ by a slightly higher "OIS-like" discount curve (lower discount rates).
- Keep the ASW formula $A = (P_{\text{disc}} - P)/\text{PV01}_{\text{disc}}$ as a sensitivity measure (this matches the algebraic structure of Example E but abstracts from separate projection).

**Given (Toy OIS Discount Factors):**

$$Z_{\text{OIS}} = \{0.986, 0.972, 0.956, 0.940, 0.925, 0.909\}, \quad P = 1.0500, \quad c = 5\%, \quad f = 2$$

**Compute:**

Sum $Z_{\text{OIS}}$:

$$\sum Z_{\text{OIS}} = 0.986 + 0.972 + 0.956 + 0.940 + 0.925 + 0.909 = 5.688$$

$$\text{PV01}_{\text{OIS}} = 0.5 \cdot 5.688 = 2.844$$

$$P_{\text{disc}}^{\text{OIS}} = 0.025 \cdot 5.688 + 0.909 = 0.1422 + 0.909 = 1.0512$$

Solve:

$$A_{\text{OIS}} \approx \frac{1.0512 - 1.0500}{2.844} = \frac{0.0012}{2.844} = 0.000422 \approx 4.2 \text{ bp}$$

**Result:**

| Discounting | ASW |
|-------------|-----|
| Example E (Libor/swap discounting) | $A \approx -11.1$ bp |
| Toy OIS discounting sensitivity | $A_{\text{OIS}} \approx +4.2$ bp |

**Interpretation:**

Changing the discount curve alone can materially change ASW, consistent with the importance of the discounting choice in modern multi-curve frameworks.

---

### Example H: Risk Decomposition of an ASW Position: Rates DV01 and Spread Sensitivity

We illustrate two exposures:
1. **Rates DV01** (per 1 bp parallel shift in chosen rate), and
2. **ASW spread sensitivity** (a "CS01-like" to $A$).

We use a simplified, internally consistent flat-curve setup to make the arithmetic transparent and to mirror Tuckman's DV01-based hedging logic.

#### H.1 Bond DV01 (Yield-Based, Using Tuckman Pricing + DV01 Definition)

**Given:**

- Bond: 3Y, coupon $c = 5\%$, semiannual payments ($c/2 = 2.50$), face $F = 100$
- Bond yield $y_b = 5.20\%$ (semiannual comp)

**Price at $y_b = 5.20\%$:**

Let $q = 1 + y_b/2 = 1.026$. Discount factors $d_t = q^{-t}$ for $t = 1, \ldots, 6$:

$$d = \{0.974659, 0.949969, 0.925686, 0.902010, 0.879042, 0.857573\}$$

Sum $d = 5.488939$.

Price:

$$P(y_b) = 2.5 \cdot 5.488939 + 100 \cdot 0.857573 = 13.72235 + 85.7573 = 99.4796$$

**Price at $y_b + 1$ bp $= 5.21\%$:**

Using $q' = 1.02605$, discount factors (computed by power recursion):

$$d' = \{0.974610, 0.949876, 0.925554, 0.901839, 0.878835, 0.857332\}$$

sum $d' = 5.488046$.

$$P(y_b + 1 \text{ bp}) = 2.5 \cdot 5.488046 + 100 \cdot 0.857332 = 13.72012 + 85.7332 = 99.4533$$

**DV01 (per 100 face):**

With $\Delta y = +0.0001$, DV01 definition gives $\text{DV01} = -\Delta P / (10{,}000 \cdot \Delta y) = -\Delta P$.

$$\Delta P = 99.4533 - 99.4796 = -0.0263 \Rightarrow \text{DV01}_{\text{bond}} = 0.0263$$

Per \$100mm face: $\$100{,}000{,}000 \times 0.0263 / 100 = \$26{,}300$.

#### H.2 Swap Leg DV01 and DV01 Mismatch (Tuckman-Style Idea)

Tuckman notes that in asset swaps, bond DV01 can differ from the swap fixed leg DV01 when bond yield differs from swap rate, creating P&L from rate moves.

**Assume:**

- Flat swap curve yield $y_s = 4.80\%$ (semiannual comp)
- A swap is structured so that paying 5% fixed vs receiving floating + $A = 20$ bp is "fair" when par swap rate is 4.80% (same logic as Tuckman's 6.10% vs 6.25% and +15 bp example)
- For DV01 magnitude, approximate the pay-fixed swap's DV01 by the fixed-leg DV01 (sign depends on direction of position)

**Fixed leg value at $y_s = 4.80\%$:**

$q = 1 + y_s/2 = 1.024$. Discount factors:

$$d = \{0.976563, 0.953674, 0.931346, 0.909537, 0.888220, 0.867402\}$$

sum $d = 5.526741$.

Fixed leg PV (coupon 5%, semiannual):

$$P_{\text{fixed}} = 2.5 \cdot 5.526741 + 100 \cdot 0.867402 = 13.81685 + 86.7402 = 100.5570$$

**Fixed leg value at $y_s + 1$ bp $= 4.81\%$:**

Computed similarly (power recursion) gives

$$P_{\text{fixed}}' \approx 100.5220$$

So $\Delta P_{\text{fixed}} = -0.0350 \Rightarrow \text{DV01}_{\text{fixed}} \approx 0.0350$.

**DV01 Mismatch in a Naive "\$100 Bond vs \$100 Swap" Package:**

- Bond DV01: $0.0263$
- Fixed leg DV01 magnitude: $0.0350$

If you hedge the bond by paying fixed in swaps on the same \$100 notional, you will typically be over-hedged in DV01 here. Hedge ratio using Tuckman's DV01 sizing logic:

$$\text{Swap notional fraction} = \frac{0.0263}{0.0350} = 0.751$$

So a \$100mm bond would be DV01-matched with about \$75.1mm swap notional in this toy setup.

#### H.3 Spread Sensitivity (ASW "CS01-Like")

From $V(0) = 1 + A \cdot \text{PV01} - P_{\text{Libor}}$, the sensitivity to $A$ is $\partial V / \partial A = \text{PV01}$.

Using the flat swap curve $y_s = 4.80\%$, we computed $\text{PV01} = \sum d \cdot \Delta = 0.5 \cdot 5.526741 = 2.763371$.

**PV change for +1 bp in $A$ on \$100 notional:**

$$\Delta V \approx 100 \cdot (0.0001) \cdot 2.763371 = 0.02763 \text{ price points}$$

For \$100mm notional: $0.02763 / 100 \times 100{,}000{,}000 = \$27{,}634$.

---

### Example I: Swap Spread Move Scenario; Toy P&L Explain

> **Label:** TOY scenario (illustrative; not a sourced historical scenario).

**Given:**

- 10Y swap rate stays fixed: $S_{10} = 3.80\%$
- 10Y Treasury yield falls: $y_{10}^{gov}$ from 3.60% to 3.40%
- Swap spread widens: from 20 bp to 40 bp

**Compute swap spread change:**

$$\Delta \text{SS}_{10} = (3.80 - 3.40) - (3.80 - 3.60) = 0.40\% - 0.20\% = +20 \text{ bp}$$

**P&L Intuition:**

- A "swap spread tightener/widener" trade typically DV01-matches a swap leg and a Treasury leg; P&L then comes mainly from basis (swap vs Treasury) moves rather than parallel rates.
- Here the move is entirely Treasury-driven (Treasury yields down), consistent with Tuckman's caution that on-the-run Treasuries can move for liquidity/specials, contaminating swap spread interpretation.

**Simple DV01-Matched P&L Sketch:**

Suppose a DV01-matched position is constructed so that:
- Receiving fixed in swaps has DV01 magnitude $\$X$ per bp
- Short Treasury has DV01 magnitude $\$X$ per bp (matched)

If swap rates don't move but Treasury yields fall 20 bp:
- Treasury leg loses about $20 \times \$X$
- Swap leg $\sim$ flat (in this toy)
- Net loses about $20 \times \$X$

A trader expecting swap spread widening from Treasury specialness would instead hold the opposite structure (long Treasury / pay swap) depending on the desk's implementation and hedge target.

> **I'm not sure** which exact "canonical" swap-spread-trade sign convention you want for these notes (receive-fixed/short-Tsy vs pay-fixed/long-Tsy) without specifying the market and the implementation (cash Treasury vs futures vs OTR vs fitted curve). The P&L direction depends on that convention.

---

### Example J: Relative Value Trade Framing: Long Bond "Cheap on ASW" / Short Swap; Scenarios

We construct a simplified ASW RV trade and explain P&L under three scenario types.

**Trade Idea:**

Bond looks cheap on ASW (ASW spread high). Buy bond and hedge rates with swaps (or do an asset swap package) expecting ASW to tighten.

**Setup (per \$100mm bond face):**

- Bond coupon 5%, maturity 3Y; current bond DV01 (from Example H): $\$26{,}300$ per bp
- Swap fixed-leg DV01 (toy, Example H): $\$35{,}000$ per bp per \$100mm swap notional
- DV01-matched swap notional fraction: $0.751$
  $\Rightarrow$ Use \$75.1mm notional pay-fixed swap to hedge bond DV01

**Entry Metrics:**

- Assume current ASW spread $A_0 = +150$ bp (quote, desk-defined)
- ASW spread sensitivity (CS01-like) per \$100mm (use Example H's \$27,634 per bp per \$100mm for the swap notional in that example; scale to your actual PV01). Here we simplify:
  - Suppose PV01 of the ASW package $\approx \$27{,}000$ per bp per \$100mm (order-of-magnitude)

**Scenario 1: Parallel Rates Shift +10 bp (No ASW Move)**

- Bond P&L $\approx -10 \times \$26{,}300 = -\$263{,}000$
- Pay-fixed swap hedge P&L $\approx +10 \times \$26{,}300 = +\$263{,}000$ (DV01 matched)
- Net $\approx 0$ (by construction), ignoring convexity/curve-shape effects

**Scenario 2: ASW Tightening by 20 bp (Rates Unchanged)**

- Using PV01-to-spread: P&L $\approx +20 \times \$27{,}000 = +\$540{,}000$ (tightening benefits the "long cheap bond vs swaps" view)
- Hedge swap DV01 doesn't matter if rates unchanged

**Scenario 3: Bond-Specific Spread Widens by 30 bp (Swap Curve Unchanged)**

- Bond cheapens relative to swaps; ASW widens
- P&L $\approx -30 \times \$27{,}000 = -\$810{,}000$

**Practical Interpretation:**

Tuckman's point: even though the strategy is often framed as "collect the spread," interim P&L from rate moves (DV01 mismatch if not well hedged) and from ASW changes can force position reductions.

---

## Practical Notes

### 1) Convention Checklist (Flag Anything Not Sourced)

**Swap Spread:**

- **Government benchmark:** on-the-run vs interpolated/fitted off-the-run curve. Quoted convention is on-the-run; fitted/adjusted alternatives exist but are subjective.
- **Swap rate:** confirm it is a par swap rate and which swap curve is used.

**ASW Convention:**

- **Par/par asset swap** (spread $A$) vs **market/market** (spread $A^*$), and whether notional equals par or full price.
- **Bond price input:** clean $P_c$ vs full $P = P_c + \text{AI}$.
- **Settlement and upfront payments** when bond price $\neq$ par (common market description).

**Discounting Choice:**

- Legacy single-curve "LIBOR/swap discounting" vs modern OIS discounting with separate projection curves.
- If OIS discounting applies, you must specify CSA/collateral terms and the projection curve definition (or state "I'm not sure").

---

### 2) Common Pitfalls

| Pitfall | Description |
|---------|-------------|
| Mixing swap spread with ASW | Swap spread is (swap $-$ gov); ASW is (bond vs swap curve) — not identical |
| ZVS vs ASW confusion | Different constructions (discounting spread vs swap-package spread) |
| Clean/dirty mismatch | Using clean price where full price is required (or vice versa) |
| Ignoring default terms | Bond default does not extinguish swap fixed payments; swap default loss is limited to positive replacement value (no principal exchange in standard swaps) |
| Forgetting basis and funding | Treasury specialness/liquidity can distort swap spreads; repo levels affect realized carry |
| Compounding/quoting mismatches | Semiannual vs money-market simple conventions across curves and instruments |

---

### 3) Implementation Pitfalls

| Pitfall | Description |
|---------|-------------|
| Curve source inconsistency | Treasury curve vendor A vs swap curve vendor B produces unstable spreads |
| Interpolation artifacts | Fitted curve choices can "move the benchmark" in swap spreads and RV metrics |
| Risk methodology | Bump-and-rebuild vs direct zero bumps can produce materially different DV01/key-rate exposures (Tuckman's bucket-exposure example illustrates rebuild-style logic) |

---

### 4) Verification Tests

**Repricing Checks:**

- Par asset swap: verify $V(0) + P \approx 1$

**Sign Checks:**

- In O'Kane par ASW: $A = (P_{\text{Libor}} - P)/\text{PV01}$. If bond price $P$ falls (bond cheapens), $A$ should rise (widen).

**Unit Checks:**

- DV01 conversion: DV01 is per-100 price points; convert to dollars by $\text{Notional} \times \text{DV01} / 100$

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **Swap spread** is $S_T - y_T^{gov}$; quoted convention often uses on-the-run government yields.
2. Swap spreads are **not pure credit measures** because on-the-run yields embed liquidity/special financing effects.
3. **Benchmark choice** (OTR vs fitted/off-the-run curve) can change swap spreads materially.
4. **Asset swap spread** anchors bonds to the swap curve, helping avoid idiosyncratic Treasury distortions for longer maturities.
5. Tuckman defines ASW as the spread added to swap rates so discounting bond cashflows matches bond price.
6. O'Kane's **par ASW formula:** $A = (P_{\text{Libor}} - P)/\text{PV01}$ with $P = P_c + \text{AI}$.
7. **Market asset swap** uses notional $P$ and $A^* = A/P$; conventions vary across desks/vendors.
8. **ZVS** ($\theta$) is a discounting spread over Libor (or gov) curve that reprices the bond; it differs conceptually from ASW.
9. ASW trades carry **rates risk** (DV01 mismatch) and **spread risk** (ASW changes); funding/repo affects realized return.
10. **Multi-curve** (OIS discounting + separate projection curves) can change ASW levels; you must state discounting/projection conventions.

---

### Cheat Sheet (Definitions + Key PV Equations + "When to Use Which")

| Formula | Expression | Use For |
|---------|------------|---------|
| **Swap spread (quoted)** | $\text{SS}_T = S_T - y_T^{\text{OTR gov}}$ | Broad "swaps vs gov" RV, but beware OTR distortions |
| **Full price** | $P = P_c + \text{AI}$ | Always reconcile clean/dirty |
| **Par ASW (O'Kane)** | $A = \dfrac{P_{\text{Libor}} - P}{\text{PV01}}$ | Bond-vs-swap valuation |
| **$P_{\text{Libor}}$** | $\dfrac{c}{f} \sum Z + Z(T)$ | PV of bond cashflows on swap curve |
| **PV01** | $\sum Z \cdot \Delta$ | Discounted accrual annuity |
| **Market ASW (O'Kane)** | $A^* = \dfrac{A}{P}$ | Variant with notional = full price |
| **ZVS (continuous form)** | $P = \dfrac{c}{f} \sum Z(0,t_n) e^{-\theta t_n} + Z(0,T) e^{-\theta T}$ | Term-structure-consistent discounting spread |
| **DV01 (general)** | $\text{DV01} = -\dfrac{\Delta P}{10{,}000 \cdot \Delta y}$ | Rate risk sizing, hedge ratios, P&L explain |

**"When to Use Which" (Quick Guide):**

| Goal | Measure |
|------|---------|
| Want "swaps vs gov" basis? | Swap spread (but state benchmark) |
| Want bond valuation vs swaps (desk RV, credit-vs-bank-credit lens)? | ASW |
| Want a clean discounting spread over a chosen curve (Libor/OIS/gov)? | ZVS/OAS |

---

### Flashcards (30 Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | Define the swap spread at maturity $T$. | $\text{SS}_T = S_T - y_T^{gov}$, usually vs on-the-run gov yield |
| 2 | Why is on-the-run benchmarking problematic? | On-the-run yields include liquidity premiums and special financing effects |
| 3 | Give two desk alternatives to on-the-run swap spreads. | Use fitted/off-the-run gov curve yields or adjust OTR yields for financing/liquidity |
| 4 | Tuckman's basic interpretation of swap rates vs gov rates? | Swap rates reflect rolling bank credit and should exceed gov rates |
| 5 | Tuckman's definition of asset swap spread (spread measure view)? | Spread such that discounting bond cashflows by swap rates plus spread gives bond price |
| 6 | What bond price should you use in ASW PV formulas: clean or dirty? | Full/dirty price $P = P_c + \text{AI}$ (must state) |
| 7 | What is $\text{PV01}(0,T)$ in O'Kane's ASW formula? | $\sum Z(0,t_n) \Delta(t_{n-1}, t_n)$ |
| 8 | Write the par ASW spread formula $A$. | $A = (P_{\text{Libor}} - P)/\text{PV01}$ |
| 9 | Define $P_{\text{Libor}}(0,T)$ in O'Kane's setup. | $(c/f) \sum Z + Z(T)$ |
| 10 | Market asset swap spread $A^*$ relates to $A$ how? | $A^* = A/P$ |
| 11 | Why introduce market asset swaps? | To avoid extra counterparty risk when exchanging a premium/discount bond for par; notional is full price |
| 12 | What happens to swap obligations if the bond defaults in an asset swap trade? | Bond coupons cease but fixed swap payments remain due |
| 13 | What limits losses in a standard interest rate swap under counterparty default? | No principal is exchanged; loss is limited to the swap's positive value |
| 14 | Define ZVS $\theta$. | Fixed spread adjustment to Libor discount rate that reprices the bond |
| 15 | Give the continuous-compounding ZVS pricing equation. | $P = (c/f) \sum Z e^{-\theta t} + Z(T) e^{-\theta T}$ |
| 16 | Why does O'Kane prefer "ZVS" terminology vs "OAS" for plain bonds? | To avoid confusion: OAS originated in callable bonds; for plain bonds it's just a spread |
| 17 | What is DV01 (general definition)? | $-\Delta P / (10{,}000 \cdot \Delta y)$ |
| 18 | How do you size a DV01 hedge between two instruments (sign aside)? | Use notional ratio proportional to DV01s (e.g., $F_B = -F_A \cdot \text{DV01}_A / \text{DV01}_B$) |
| 19 | What are two major P&L drivers of an asset swap trade per Tuckman? | Rate moves (DV01 mismatch) and ASW spread changes |
| 20 | Why do desks use asset swap spreads for longer maturities? | To measure value relative to curves less contaminated by individual security effects |
| 21 | What does a positive ASW spread generally indicate in O'Kane's formula? | Bond is cheap vs the Libor/swap discount curve (holding other conventions fixed) |
| 22 | Can ASW spreads be negative? | Yes (e.g., if issuer perceived better than AA banks); negative spreads are discussed as possible |
| 23 | Why might swap spreads rise even if bank credit improves? | Treasury scarcity/specialness can lower Treasury yields, widening swap spreads |
| 24 | What's the key difference between swap spread and ASW? | Swap spread is swaps vs gov; ASW is bond vs swap curve |
| 25 | In Hull's description, what happens if the bond trades at 95 in an asset swap? | One side makes a \$5 upfront payment per \$100 so the coupon exchange continues at LIBOR+spread |
| 26 | Why can swap-hedging corporate debt still leave basis risk? | Swap rates reflect banking credit, not general or issuer-specific credit spreads |
| 27 | What does $P = P_c + \text{AI}$ protect you from operationally? | Clean/dirty mismatches that misstate package PV and spread |
| 28 | What does Tuckman say about differences in fixed vs floating payment conventions in some examples? | He simplifies by ignoring them in the illustrative asset swap example |
| 29 | What's a practical reason to compute swap spreads vs fitted gov yields? | To reduce distortion from on-the-run liquidity/special financing |
| 30 | Why does discounting choice (OIS vs legacy) matter for ASW? | Multi-curve frameworks use different discount curves; changing discounting changes PV and hence implied spreads |

---

## Mini Problem Set (16 Questions)

> Questions 1–8 include brief solution sketches. Questions 9–16 do not.

**1.** Swap spread: $S_7 = 3.55\%$, $y_7^{\text{OTR}} = 3.40\%$. Compute swap spread in bp.

*Sketch:* $(3.55 - 3.40)\% = 0.15\% = 15$ bp.

**2.** Benchmark sensitivity: Using Q1, fitted gov yield is $3.46\%$. Recompute swap spread and the difference vs OTR.

*Sketch:* $3.55 - 3.46 = 0.09\% = 9$ bp; difference is $-6$ bp.

**3.** ASW inputs: Clean price $P_c = 101.20$, AI $= 0.30$ (per 100). What full price $P$ do you use?

*Sketch:* $P = 101.50 \Rightarrow 1.0150$ per par 1.

**4.** Compute PV01: Semiannual schedule with $Z = \{0.99, 0.975, 0.96, 0.945\}$, $\Delta = 0.5$. Compute PV01.

*Sketch:* $\sum Z = 3.87$; PV01 $= 0.5 \cdot 3.87 = 1.935$.

**5.** Compute $P_{\text{Libor}}$: Coupon $c = 4\%$, $f = 2$, same $Z$ as Q4, maturity at last date with $Z(T) = 0.945$.

*Sketch:* $P_{\text{Libor}} = 0.02 \cdot 3.87 + 0.945 = 0.0774 + 0.945 = 1.0224$.

**6.** Solve ASW $A$: Use Q3–Q5 with $P = 1.0150$. Compute $A$ in bp.

*Sketch:* $A = (1.0224 - 1.0150)/1.935 = 0.0074/1.935 = 0.00382 \approx 38.2$ bp.

**7.** Market ASW $A^*$: Use Q6 and $P = 1.0150$.

*Sketch:* $A^* = A/P \approx 0.00382/1.0150 = 0.00376 \approx 37.6$ bp.

**8.** ZVS sign intuition: If $P > P_{\text{Libor}}$, what is the sign of $\theta$ (continuous-compounding ZVS)?

*Sketch:* Need higher PV $\Rightarrow \theta < 0$ (negative spread increases PV).

**9.** Compare swap spread vs ASW: give one scenario where swap spread widens but ASW for a given corporate bond tightens.

**10.** Describe how repo specialness in the benchmark Treasury can distort a quoted swap spread.

**11.** Using Tuckman's caution, propose a desk procedure to compute "adjusted swap spreads" and list its weaknesses.

**12.** Give two reasons ASW and ZVS can diverge for the same bond even without embedded options.

**13.** Explain why an asset swap is not a perfect "credit-only" trade even if DV01-matched.

**14.** Suppose OIS discounting is adopted. List the minimum additional conventions/data you need to compute ASW rigorously.

**15.** A bond's ASW spread widens 25 bp and its yield curve is unchanged. Explain the likely direction of the bond price and the P&L of a long-ASW position.

**16.** Construct a DV01-matched bond vs swap hedge using DV01s of 0.030 and 0.045 (per 100). What swap notional per \$100mm bond?

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| Swap spread definition $\text{SS}_T = S_T - y_T^{gov}$ with on-the-run benchmark convention | Tuckman |
| Swap rates reflect rolling bank credit; should exceed gov rates | Tuckman |
| On-the-run yields embed liquidity premiums and special financing | Tuckman |
| ASW defined as spread such that discounting bond cashflows by swap rates + spread gives bond price | Tuckman |
| Par asset swap formula $A = (P_{\text{Libor}} - P)/\text{PV01}$ | O'Kane |
| Market asset swap $A^* = A/P$ with notional = full price | O'Kane |
| ZVS as fixed spread adjustment to Libor discount rate | O'Kane |
| Bond default stops coupons but swap payments remain due | Tuckman |
| Multi-curve separation (OIS discounting vs Libor projection) | Andersen & Piterbarg, Hull |
| Asset swaps give "direct estimates of excess of bond yields over LIBOR/swap rates" | Hull |
| Swap spread is not a pure credit spread (joint outcome of credit + supply/demand + microstructure) | Tuckman, O'Kane |
| DV01 definition $\text{DV01} = -\Delta P / (10{,}000 \cdot \Delta y)$ | Tuckman |

### (B) Reasoned Inference (Derived from A)

| Inference | Derivation Logic |
|-----------|------------------|
| Benchmark choice (OTR vs fitted) can change swap spreads by multiple bp | Direct application of definition with different $y_T^{gov}$ inputs |
| ASW and ZVS can differ due to curve shape, conventions, multi-curve | Structural difference in definitions (discounting spread vs swap-package spread) |
| ASW spread sensitivity $\partial V / \partial A = \text{PV01}$ | Differentiation of O'Kane's swap value formula |
| DV01 mismatch in bond-vs-swap package when bond yield $\neq$ swap rate | Follows from Tuckman's discussion of rates P&L in asset swaps |

### (C) Speculation (Clearly Labeled)

| Item | Uncertainty Flag |
|------|------------------|
| Exact "canonical" swap-spread-trade sign convention (receive-fixed/short-Tsy vs pay-fixed/long-Tsy) | **I'm not sure** — depends on market, implementation, and desk convention |
| Desk terminology for "G-spread" vs "I-spread" | **I'm not sure** — some desks use "I-spread" for interpolated gov curve spread; would need exact convention |
| Full multi-curve ASW computation under OIS discounting | **I'm not sure** — requires specifying projection curve, CSA/collateral terms, and instrument set for curve build |

---
