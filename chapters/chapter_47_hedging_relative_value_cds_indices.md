# Chapter 47: Hedging and Relative Value in CDS Indices — Index vs Single-Name Hedge Logic, Default Events, and Evolving Index Risk

---

## Conventions & Notation

This chapter is risk-first: we describe exposures, hedges, residuals, and validation (not "trade tips").

No index rulebook specifics (standard coupons, exact roll calendars, operational settlement mechanics) are asserted unless supported by the provided sources. Where needed, we say "I'm not sure."

All spreads are in bp unless stated; conversion: $1 \text{ bp} = 10^{-4}$ (decimal rate per year).

---

## 0. Setup

### Conventions Used in This Chapter

- We work with single-name CDS and CDS portfolio indices as in O'Kane.
- Index positions are described in "CDS language": buying protection = long protection, selling protection = short protection.
- Index valuation language distinguishes:
  - **Market upfront** $U_I(t)$ implied by a flat index curve $S_I(t,T)$ and the contractual coupon $C(T)$.
  - **Intrinsic value** $V_I(t)$ built from constituent CDS curves.
- Equal-weighting $1/M$ is used when following the O'Kane index valuation/mechanics setup (e.g., losses and notional changes per default scale by $1/M$).

**I'm not sure** whether your target index uses equal weights, notional weights, or another weighting scheme. To be certain, we'd need the index rulebook (e.g., Markit/ICE index methodology) or your desk's index analytics convention.

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $t$ | Valuation time ("today") |
| $T$ | Maturity date of the CDS / index swap |
| $M$ | Number of index constituents (alive at inception; defaults remove names without replacement) |
| $m \in \{1, \ldots, M\}$ | Constituent name index |
| $N$ | Trade notional (USD); $N_I$ for index notional, $N_m$ for constituent notional |
| $S_m(t,T)$ | Market spread for name $m$ to $T$ |
| $S_I(t,T)$ | Quoted index spread (market "index curve" level) to $T$ |
| $C(T)$ | Contractual index coupon (fixed for the series; typically chosen to be a multiple of 5 bp) |
| $U_I(t)$ | Market upfront payment for the index at time $t$ |
| $V_I(t)$ | Intrinsic value of the index (constituent-implied) |
| $\text{RPV01}(t,T)$ | Risky PV01 (PV of premium leg per unit spread; includes accrued-at-default in the CDS setup) |
| CS01 / Credit DV01 | Spread sensitivity defined as in O'Kane (finite difference; sign convention chosen so short-protection CS01 is positive) |
| VOD | Value-on-default (jump-to-default style risk) for a CDS; explicit decomposition into MTM cancellation, protection payment, and accrued premium |
| $FP$ | Auction final price / recovery proxy in cash settlement (per 100); payout uses face minus recovery price |

---

## 1. Core Concepts (Definitions First)

### 1.1 CDS Index as "One Trade, Many Names"

**Formal Definition:**

A CDS portfolio index is a contract on a fixed set of reference entities for the life of the series; if a constituent defaults it is removed without replacement.

The index pays a fixed contractual coupon $C(T)$ on the outstanding notional, and is traded with an upfront $U_I(t)$ that can be positive or negative.

**Intuition:**

You get "macro credit beta" in one line item, with liquidity often better than many single names.

**Trading/Risk/Portfolio Practice:**

Indices are used for macro hedges, benchmarking, and as underlyings for index options (portfolio swaptions).

---

### 1.2 Intrinsic Value vs Quoted Index Level ("Index Basis")

**Formal Definition:**

- **Intrinsic value** $V_I(t)$: value implied by constituent CDS curves (sum/average of constituents in the model).
- **Quoted index level**: market uses a simplified flat index curve $S_I(t,T)$ to quote/price the index "as though it were a single name."
- **Index basis (spread form)**: $\text{Basis} = S_I - S_I^{\text{intrinsic}}$. O'Kane documents that $S_I$ often differs from intrinsic and lists drivers.

**Intuition:**

Basis is the gap between "what the index trades at" and "what the constituents imply," reflecting contractual differences, liquidity, and flow/lead-lag effects.

**Trading/Risk/Portfolio Practice:**

Hedging an index with constituents forces you to decide where the basis lives in your model; ignoring it makes replication hedges drift.

---

### 1.3 Portfolio Swap Adjustment (PSA)

**Formal Definition:**

A modeling adjustment that modifies constituent curves so that adjusted intrinsic matches quoted index levels; the adjustment is described as "somewhat arbitrary," chosen for stability/speed/reasonableness.

Example method: apply a spread multiplier $\alpha(T)$ so $S_m^*(t,T) = \alpha(T) S_m(t,T)$, and solve for $\alpha(T_n)$ so adjusted intrinsic equals market index upfront.

**Intuition:**

PSA is a bookkeeping layer: "I want my constituent-based model to reproduce the index quotes I actually hedge with."

**Trading/Risk/Portfolio Practice:**

Critical when you hedge index products (options/tranches) with constituents: otherwise you embed a model basis you didn't choose.

---

### 1.4 Spread Risk Measures: CS01 / Credit DV01 and RPV01

**Formal Definition:**

O'Kane's Credit DV01 is the change in value for a 1 bp parallel increase in the CDS curve, defined by a finite difference with a sign convention so short protection DV01 is positive.

RPV01 is the premium-leg PV factor, including accrued-at-default (in the CDS setup).

**Intuition:**

CS01 answers: "How many dollars do I gain/lose if spreads move 1 bp (parallel)?"

**Trading/Risk/Portfolio Practice:**

Hedge sizing often starts with CS01 matching; then scenario tests reveal residuals (dispersion, basis, event).

---

### 1.5 Default-Event Risk: VOD (Jump-to-Default Style Risk)

**Formal Definition:**

For a CDS, VOD is the change in value on an immediate default, decomposed into:
1. MTM goes to zero,
2. Protection payment $(1-R)$ (sign depends on protection buyer/seller),
3. Accrued premium at default.

**Intuition:**

Even if you hedge spread risk, a sudden default can produce a jump P&L.

**Trading/Risk/Portfolio Practice:**

O'Kane flags that index books must manage VOD; in index books, a default should be offset by another index swap of the same face value (in that discussion).

---

### 1.6 Roll / Series Risk and On-the-Run vs Off-the-Run

**Formal Definition:**

O'Kane describes indices as rolling semiannually and notes that index expiries beyond six months are rare because liquidity drops after the next roll, raising hedging costs.

Hedging off-the-run with on-the-run can mismatch constituents (some credits in one series but not the other).

**Intuition:**

"Same index family" does not mean "same risk": membership, maturity, and liquidity can differ.

**Trading/Risk/Portfolio Practice:**

Roll/series basis becomes a distinct risk bucket in hedging reports.

---

## 2. The Hedge Problem in Indices (Organizing Framework)

This section is the backbone: what exposure do you have, what do you hedge with, how do you size, and how do you validate?

### 2.1 Identify the Exposure You Actually Have

Decompose your position into orthogonal risk channels:

**Systematic (macro) credit spread risk**

The "index level" $S_I(t,T)$ behaves like a macro factor; indices are used to express/hedge market-wide views.

**Idiosyncratic (name-specific) spread risk**

Even in an equal-weight index, names move differently → dispersion risk.

**Index–constituent basis**

Quoted $S_I$ differs from intrinsic; sources of basis include restructuring clause differences (e.g., No-Re vs Mod-Re), liquidity premia, and index leading the single-name market in stress.

**Default-event / jump risk**

A constituent default changes cashflows and notional; default settlement is via physical or cash, with auction processes for cash settlement.

On a CDS, VOD quantifies the "surprise jump" component.

**Roll / series / liquidity risk**

Semiannual roll and post-roll liquidity drop create hedge slippage and transaction cost risk.

---

### 2.2 Choose Hedge Instruments (What Can Actually Offset What?)

A practitioner's menu:

**Index hedge (same family, same maturity)**

Best for systematic spread risk (macro beta).

Be cautious for series mismatches: off-the-run vs on-the-run membership differences can matter.

**Single-name hedges (constituents or proxies)**

Best for idiosyncratic risk (especially concentration in a few names).

Can also be used to build intrinsic replication (bottom-up).

**Nearby maturities / curve hedges**

Indices trade multiple maturities (3Y/5Y/7Y/10Y) and the most liquid are 5Y and 10Y.

A 5Y hedge may not hedge a 10Y exposure well if the term structure moves non-parallel.

**Related indices (cross-market hedges)**

Useful when your target index is illiquid; introduces cross-index basis risk (not detailed in sources → treat as desk-convention territory).

---

### 2.3 Compute Hedge Ratios (Risk Matching)

Core sizing approaches:

**CS01 / Credit DV01 matching**

Primary "first-pass" hedge sizing.

Works best for parallel spread moves (by construction).

**Beta-weighted (proxy) hedges**

Useful when hedging a single name or sector book with an index proxy.

**I'm not sure** which beta estimation method your desk uses (historical regression, implied beta, risk model). The sources here do not specify a beta estimation protocol; to be certain we'd need a desk convention or a risk model document.

---

### 2.4 Validate with Scenario Sets (Don't Trust a Single Number)

A minimal scenario suite:

1. **Parallel widening/tightening** (index and names move together)
2. **Dispersion** (names move differently; same average)
3. **Single-name shock** (one name gaps; index moves modestly)
4. **Default event** (one name defaults; choose recovery/final price scenarios)
5. **Roll/series shift** (on-the-run/off-the-run basis + membership mismatch)
6. **Liquidity/execution cost stress** (wider bid/ask in names than index)

---

### 2.5 Must-Distinguish: Top-Down vs Bottom-Up vs Basis "Hedge"

**Top-down hedge (proxy hedge)**

Hedge a single name or portfolio using an index.

You remove macro component but keep idiosyncratic and basis.

**Bottom-up hedge (replication / intrinsic hedge)**

Hedge an index using constituents.

You aim to replicate intrinsic; you keep index basis and execution dispersion.

**Basis trade vs hedge**

If you "hedge" an index with constituents, the residual is often exactly the index basis + liquidity/roll effects. O'Kane emphasizes basis and motivates PSA because intrinsic vs quoted can differ materially.

---

## 3. Exposures and Risk Measures (Must Be Explicit and Unit-Checked)

### 3.1 Index Spread Risk: CS01 (Credit DV01)

**Definition (bump-and-reprice, primary)**

Let $V(S_I)$ be the PV of the position as a function of the quoted index spread $S_I$.

Define:

$$\boxed{\text{CS01}_I \equiv \text{Credit DV01} = -\bigl(V(S_I + 1 \text{ bp}) - V(S_I)\bigr)}$$

This is O'Kane's Credit DV01 definition (with sign convention).

**What is bumped?**

A 1 bp parallel increase in the index spread curve level (often treated as flat for quoting).

**Units**

$V$ is in dollars ⇒ CS01 is \$/bp.

**Sign convention**

With the above definition, short protection positions generally have positive CS01 (value falls when spreads rise, so CS01 is positive by construction).

**RPV01 language**

O'Kane explicitly uses RPV01 and computes it for indices.

For at-the-money situations, DV01 is closely linked to RPV01 in his discussion (for CDS).

---

### 3.2 Constituent Dispersion Risk

**Why equal-weight indices still have dispersion**

Even if weights are $1/M$, constituents have different spread levels, curve shapes, liquidity, and idiosyncratic news → $\Delta S_m$ are not equal.

**How it appears**

A top-down hedge (single name vs index) leaves residual:

$$\text{Residual P\&L} \approx \sum_m (\text{net single-name CS01 exposure}) \times \Delta S_m$$

which can be large even if index level is well-hedged.

---

### 3.3 Default-Event Risk (Jump-to-Default / VOD-Style)

**Single-name VOD (source-backed)**

O'Kane decomposes VOD into:
1. MTM cancels to zero,
2. Protection payment $(1-R)$ (sign depends on buyer/seller),
3. Accrued premium $\Delta_0 S_0$.

**Index default-event mapping**

O'Kane's index valuation treats each name as contributing $(1 - R_m)/M$ losses to the index buyer.

Contractually, defaults reduce future premium payments by $1/M$ increments.

**Units**

Event payout sensitivity to $FP$ is \$/point (point = 1 on price per 100).

---

### 3.4 Recovery / Final Price Risk

**Cash settlement mapping (source-backed)**

In cash settlement, protection seller pays face minus recovery price.

**Sensitivity**

For notional $N$ and final price $FP$ (per 100):

$$\boxed{\text{Payout} = N \left(1 - \frac{FP}{100}\right), \quad \frac{\partial \text{Payout}}{\partial FP} = -\frac{N}{100}}$$

Units: \$/price point.

---

### 3.5 Roll / Series Risk

**What changes**

New series has different maturity (O'Kane notes the new on-the-run has a longer maturity by about six months, tending to widen if curves are upward sloping).

Liquidity drops after roll, increasing hedging costs (notably for index options).

**Residual risk**

Hedging off-the-run with on-the-run can leave membership mismatch risk.

---

### 3.6 Index Basis Risk (Quoted vs Intrinsic)

**Definition**

$$\text{Basis} = S_I - S_I^{\text{intrinsic}}$$

where intrinsic is derived from constituent spreads/curves.

**What is bumped?**

A "basis bump" changes $S_I$ holding constituents fixed, or changes constituent spreads holding $S_I$ fixed—these are different shocks.

**Source-backed drivers**

Restructuring clause differences (No-Re vs Mod-Re), liquidity, and index leading market flows.

---

## 4. Hedging Logic: Index vs Single Name

### 4.1 Single-Name Hedged with Index (Top-Down Proxy Hedge)

**Goal:** offset single-name spread risk using index spread risk.

**Beta / proxy framework**

Define a spread beta $\beta_{m,I}$ such that

$$\Delta S_m \approx \beta_{m,I} \, \Delta S_I + \varepsilon_m$$

where $\varepsilon_m$ is idiosyncratic.

**I'm not sure** how your desk estimates $\beta_{m,I}$ because the sources here do not specify an estimation method; to be certain we'd need a desk risk model (or a statistical estimation convention).

**CS01 matching hedge ratio**

Let $\text{CS01}_m$ be the single-name CS01 (in \$/bp) for your position, and $\text{CS01}_I$ be the index CS01 per unit index notional.

Choose index hedge notional $N_I$ such that:

$$\text{CS01}_m + \text{CS01}_I(N_I) \approx 0$$

If $\text{CS01}_I$ is linear in notional (first-order), then:

$$\boxed{N_I \approx -\frac{\text{CS01}_m}{\text{CS01}_I(1)}}$$

(Derivation in Section 7.)

**Residuals you keep**

- **Idiosyncratic:** $\varepsilon_m$ shocks.
- **Name–index basis:** the single name can move vs the index (including liquidity differences).
- **Contract differences** (e.g., restructuring clause differences are explicitly noted as a basis source in indices vs underlying CDS).
- **Default-event mismatch:** index VOD style risk differs because index is a basket with notional reduction mechanics.

---

### 4.2 Index Hedged with Constituents (Bottom-Up Intrinsic Hedge)

**Goal:** offset index spread risk using constituent positions.

**Start from intrinsic representation**

O'Kane writes intrinsic value of an index (short protection) as a sum over constituents:

$$V_I(t) = \frac{1}{M} \sum_{m=1}^{M} \bigl(C(T) - S_m(t,T)\bigr) \, \text{RPV01}_m(t,T)$$

**Hedge ratio logic**

A bottom-up hedge chooses constituent notionals $N_m$ so that the combined CS01 matches the index CS01.

If you hedge only a subset of names, you knowingly keep residual dispersion + basis.

**Residuals you keep**

- **Bid/ask dispersion:** many single names have wider costs than the index.
- **Missing constituents:** incomplete replication.
- **Defaults / removals:** membership shrinks (no replacement after default).
- **Index basis:** quoted index may not equal intrinsic (hence PSA motivation).

---

### 4.3 Hedging Curve Risk Inside Indices (Term-Structure Mismatch)

Indices trade multiple maturities (3Y/5Y/7Y/10Y).

A hedge in 5Y index does not fully hedge 10Y exposure if:
- the credit curve twists (non-parallel), or
- roll/series maturity differs (new series longer by ~6 months).

**Practical implementation:**

Compute bucketed CS01s (e.g., by maturity point) via bump-and-reprice.

---

## 5. Default Events and How Index Risk Evolves (Must Include)

### 5.1 Mechanics + Risk Consequences (Source-Backed Where Possible)

**Cashflows and notional after a constituent default**

Index premium leg is reduced after a credit event by a factor $1/M$ per default.

O'Kane's mechanics narrative says after each default "the size of the coupon is reduced as the face value is reduced by $1/M$."

Defaults are handled with settlement protocols; O'Kane notes ISDA introduced a 2005 protocol allowing fallback to cash settlement via auction when physical deliverables are scarce.

**How risk evolves**

After defaults, the index is effectively a smaller basket:
- Outstanding notional decreases (premium leg dollars per bp decrease).
- Composition changes (remaining names may be riskier/less diversified).

Therefore the index CS01 per trade changes after defaults even if "per unit notional" CS01 were unchanged.

---

### 5.2 Before/After Risk Reporting (What to Show)

A minimal index risk report should include:

**CS01 (spread DV01)**

**VOD / event measure**

For constituents, VOD is explicit for CDS.

For index, you can aggregate constituent-like event exposures scaled by $1/M$ (see Example 11); exact operational netting is index-rulebook dependent.

**Recovery/final-price sensitivity**

Use cash settlement mapping: face minus recovery price.

---

### 5.3 Front-End Protection and Default Settlement Netting

**I'm not sure.** The provided sources in this prompt do not fully specify "front-end protection" mechanics or netting conventions for CDS indices across defaults and settlements in the exact rulebook sense.

To be certain, we would need: the relevant index rulebook and the applicable ISDA definitions / auction protocol (vintage) for the trades you are analyzing.

---

## 6. Relative Value Frameworks in Indices (Risk-First, Not Trade Tips)

### 6.1 Index Basis: Quoted vs Intrinsic (Framework)

**Definition**

Basis $B = S_I - S_I^{\text{intrinsic}}$, where intrinsic is computed from constituents.

**Intrinsic computation**

O'Kane emphasizes the index spread is not a simple average, but is close to an RPV01-weighted average of constituent spreads (as an approximation).

**What you're implicitly long/short**

A "basis position" (index vs intrinsic replication) typically leaves you exposed to:
- liquidity (index vs names),
- contractual differences (e.g., restructuring clause differences),
- dispersion (constituent heterogeneity),
- roll/series effects.

---

### 6.2 Series Basis / Roll Basis

**Key idea**

New on-the-run is longer maturity by ~6 months; upward sloping curves tend to widen it, all else equal.

Liquidity drops after roll, raising hedging costs.

**Risk-first lens**

Measure:
- spread difference $\Delta S$ between series,
- CS01 mismatch,
- membership mismatch risk (credits not shared).

---

### 6.3 Index vs Single-Name "Dispersion" Framing

If index looks rich/cheap vs constituents, your exposure is often:
- systematic spread (macro),
- dispersion (idiosyncratic residual),
- liquidity and execution (index tighter than names),
- default-event mapping (basket vs single name).

---

## 7. Math and Derivations (Step-by-Step)

**Conventions:** CS01 is O'Kane Credit DV01: $\text{CS01} = -\Delta V$ for a +1 bp spread bump.

Therefore first-order P&L from a spread move $\Delta S$ (in bp) is:

$$\boxed{\Delta V \approx -\text{CS01} \cdot \Delta S}$$

**Unit check:** CS01 is \$/bp, $\Delta S$ is bp ⇒ $\Delta V$ is \$.

---

### 7.1 CS01-Based Hedge Ratio (Two Instruments)

Suppose you hold instrument A and hedge with instrument B.

Let:
- $\text{CS01}_A$ = CS01 of your current position (includes its notional).
- $\text{CS01}_B(1)$ = CS01 per \$1 of notional in hedge instrument B.

Choose hedge notional $N_B$.

**Total CS01:**

$$\text{CS01}_{\text{tot}} = \text{CS01}_A + N_B \, \text{CS01}_B(1)$$

Set $\text{CS01}_{\text{tot}} = 0$:

$$\boxed{N_B^* = -\frac{\text{CS01}_A}{\text{CS01}_B(1)}}$$

**Sign sanity check:**

If A is long protection (typically negative CS01) and B is short protection (positive CS01 per notional), then $N_B^* > 0$.

---

### 7.2 Simple Two-Instrument Hedge Solve for Two Risk Factors

If you want to hedge two risk factors (e.g., 5Y and 10Y spread moves), you need two hedge instruments.

Let risk factors be $\Delta S^{(5)}, \Delta S^{(10)}$.

Let instrument sensitivities:

$$\begin{pmatrix} \text{CS01}_A^{(5)} \\ \text{CS01}_A^{(10)} \end{pmatrix}, \quad \begin{pmatrix} \text{CS01}_B^{(5)} \\ \text{CS01}_B^{(10)} \end{pmatrix}, \quad \begin{pmatrix} \text{CS01}_C^{(5)} \\ \text{CS01}_C^{(10)} \end{pmatrix}$$

Choose hedge notionals $x_B, x_C$ so:

$$\begin{pmatrix} \text{CS01}_A^{(5)} \\ \text{CS01}_A^{(10)} \end{pmatrix} + x_B \begin{pmatrix} \text{CS01}_B^{(5)} \\ \text{CS01}_B^{(10)} \end{pmatrix} + x_C \begin{pmatrix} \text{CS01}_C^{(5)} \\ \text{CS01}_C^{(10)} \end{pmatrix} = \begin{pmatrix} 0 \\ 0 \end{pmatrix}$$

Solve the 2×2 linear system.

**Unit check:** each CS01 component is \$/bp; $x_B, x_C$ are notionals.

---

### 7.3 Least-Squares Extension (Many Constituents)

If hedging an index with many names, you may solve:

$$\min_{\{x_m\}} \left\| \text{CS01}_{\text{index}} + \sum_m x_m \, \text{CS01}_m \right\|^2$$

possibly with constraints (e.g., only top $K$ names, non-negativity).

**I'm not sure** what exact optimization constraints your desk uses; we'd need your risk policy / execution constraints.

---

### 7.4 P&L Decomposition Template (Index + Constituents + Events + Basis)

A risk-first decomposition:

$$\Delta V \approx -\text{CS01}_I \, \Delta S_I - \sum_m \text{CS01}_m \, \Delta S_m + \sum_{\text{defaults}} \Delta V_{\text{event}} + \Delta V_{\text{basis}} + \text{higher-order/curve terms}$$

- $\Delta V_{\text{event}}$ can be organized using VOD-style components (MTM cancellation, protection payment, accrued premium).
- $\Delta V_{\text{basis}}$ captures moves in $S_I - S_I^{\text{intrinsic}}$.

---

## 8. Worked Examples (At Least 14 Numeric Examples, Fully Numeric)

**Note:** The numerical PVs below are toy "pricing-engine outputs." They are used to demonstrate hedge math and risk decomposition. Where index operational details matter, assumptions are stated and labeled.

---

### Example 1: Index CS01 via Finite Differences

Assume we have an index short-protection position (so CS01 should be positive under O'Kane's convention).

**Given PVs at spread $S$ and $S \pm 1$ bp:**

- $V(S-1) = \$54,300$
- $V(S) = \$50,000$
- $V(S+1) = \$45,800$

**Central-difference slope:**

$$\left.\frac{\partial V}{\partial S}\right|_{\text{bp}} \approx \frac{V(S+1) - V(S-1)}{2} = \frac{45{,}800 - 54{,}300}{2} = \frac{-8{,}500}{2} = -4{,}250 \; \$/\text{bp}$$

**CS01 (Credit DV01):**

$$\text{CS01}_I \approx -\left.\frac{\partial V}{\partial S}\right|_{\text{bp}} = 4{,}250 \; \$/\text{bp}$$

**Unit check:** dollars per bp ✓

---

### Example 2: Single-Name CS01 via Finite Differences

**Single-name short-protection PVs:**

- $V(S-1) = \$12,500$
- $V(S) = \$10,000$
- $V(S+1) = \$7,600$

**Central slope:**

$$\frac{7{,}600 - 12{,}500}{2} = \frac{-4{,}900}{2} = -2{,}450 \; \$/\text{bp}$$

$$\text{CS01}_{\text{SN}} = 2{,}450 \; \$/\text{bp}$$

---

### Example 3: Proxy Hedge Ratio — Hedge a Single-Name CDS Using an Index (CS01 Matching)

**Assume:**

- Example 2 CS01 was for \$10mm single-name notional.
- So per \$1mm: $2{,}450 / 10 = 245$ \$/bp per \$1mm.
- Example 1 CS01 was for \$10mm index notional.
- Per \$1mm: $4{,}250 / 10 = 425$ \$/bp per \$1mm.

We hold long protection on the single name with notional $N_{\text{SN}} = \$20\text{mm}$.

Long protection CS01 is negative of short protection CS01 (same bump definition).

$$\text{CS01}_{\text{SN,long}} = -245 \times 20 = -4{,}900 \; \$/\text{bp}$$

We hedge by selling protection on the index (short protection), CS01 per \$1mm is $+425$.

**Solve:**

$$N_I = -\frac{\text{CS01}_{\text{SN,long}}}{425} = -\frac{-4{,}900}{425} = 11.5294 \text{ mm}$$

**Answer:** Sell protection on \$11.53mm index notional.

---

### Example 4: Scenario Test — Broad Spread Move and Hedge Performance

Use the hedge from Example 3.

**Scenario:** all spreads widen by +10 bp, and the index also widens +10 bp.

**Single-name P&L (first-order):**

$$\Delta V_{\text{SN}} \approx -\text{CS01}_{\text{SN,long}} \cdot \Delta S = -(-4{,}900) \times 10 = +49{,}000$$

**Index hedge P&L:**

Hedge CS01: $425 \times 11.5294 = 4{,}900$ \$/bp.

$$\Delta V_I \approx -(+4{,}900) \times 10 = -49{,}000$$

**Net:** $0$.

**Interpretation:** CS01 matching neutralizes parallel moves by construction.

---

### Example 5: Idiosyncratic Move Residual

Keep the same hedge.

**Scenario:**
- Single name widens +50 bp
- Index widens +10 bp

**Single name:**

$$\Delta V_{\text{SN}} = -(-4{,}900) \times 50 = +245{,}000$$

**Index hedge:**

$$\Delta V_I = -(+4{,}900) \times 10 = -49{,}000$$

**Net:**

$$+245{,}000 - 49{,}000 = +196{,}000$$

**Residual meaning:** you are still exposed to name-specific risk (dispersion/idiosyncratic).

---

### Example 6: Dispersion Shock — Constituents Disperse, Same Average

Toy portfolio of 5 single names, each \$10mm notional, short protection (so CS01 positive). CS01 per bp per name:

| Name | CS01 (\$/bp) |
|------|--------------|
| 1 | 300 |
| 2 | 250 |
| 3 | 200 |
| 4 | 350 |
| 5 | 300 |

**Dispersion scenario (bp changes):**

$$\Delta S = [+30, -10, +20, -5, -35]$$

Simple average: $(30 - 10 + 20 - 5 - 35)/5 = 0$ bp → "index level unchanged" (toy).

**Single-name book P&L:**

$$\Delta V = -\sum_i \text{CS01}_i \cdot \Delta S_i$$

**Compute term-by-term:**

| Name | Calculation | Result |
|------|-------------|--------|
| 1 | $-300 \times 30$ | $-9{,}000$ |
| 2 | $-250 \times (-10)$ | $+2{,}500$ |
| 3 | $-200 \times 20$ | $-4{,}000$ |
| 4 | $-350 \times (-5)$ | $+1{,}750$ |
| 5 | $-300 \times (-35)$ | $+10{,}500$ |

**Sum:**

$$-9{,}000 + 2{,}500 - 4{,}000 + 1{,}750 + 10{,}500 = +1{,}750$$

**Result:** even with "average move = 0," you get nonzero P&L because CS01 weights differ → dispersion risk.

---

### Example 7: Bottom-Up Intrinsic Hedge — Hedge an Index Using a Subset of Constituents (Toy 5 Names)

Assume index short-protection notional $N_I = \$50\text{mm}$. Index CS01 per \$1mm is 450 \$/bp.

**Index CS01:**

$$\text{CS01}_I = 450 \times 50 = 22{,}500 \; \$/\text{bp}$$

Five constituents, CS01 per \$1mm (short protection):

$$[500, 400, 480, 520, 450] \; \$/\text{bp per \$1mm}$$

**Step 1: Full replication hedge (buy protection on names)**

Buying protection flips sign, so each name contributes $-\text{CS01}$.

Try equal notionals first: \$10mm each.

Total constituent CS01 (long protection):

$$-(500 + 400 + 480 + 520 + 450) \times 10 = -(2{,}350) \times 10 = -23{,}500 \; \$/\text{bp}$$

Residual CS01:

$$22{,}500 + (-23{,}500) = -1{,}000 \; \$/\text{bp}$$

**Step 2: Scale to match CS01 exactly**

Scale factor:

$$k = \frac{22{,}500}{23{,}500} = 0.9574468$$

So hedge notionals become $10 \times k = 9.574\text{mm}$ each.

**Step 3: Missing constituent**

Suppose name 5 is not tradable today. Hedge only names 1–4 with the same scaled notionals 9.574mm each.

Constituent CS01 with 4 names:

Sum per \$1mm: $500 + 400 + 480 + 520 = 1{,}900$.

Hedge CS01:

$$-(1{,}900) \times 9.574 = -18{,}190.6 \; \$/\text{bp}$$

Residual CS01:

$$22{,}500 - 18{,}190.6 = 4{,}309.4 \; \$/\text{bp}$$

**Interpretation:** incomplete replication leaves large residual systematic exposure.

---

### Example 8: Index Basis Computation (Quoted − Intrinsic)

We approximate intrinsic index spread using an RPV01-weighted average (O'Kane notes intrinsic spread is close to an RPV01-weighted average, not a simple average).

**I'm not sure** about your desk's exact intrinsic-spread algorithm (e.g., solving Equation 10.8/10.9 in full). To be certain, we'd need the desk's index pricer spec or the full methodology section you use.

**Toy constituents:**

| Name | Spread $S_i$ (bp) | RPV01 weight $w_i$ |
|------|-------------------|-------------------|
| 1 | 60 | 4.5 |
| 2 | 80 | 4.4 |
| 3 | 100 | 4.3 |
| 4 | 120 | 4.1 |
| 5 | 200 | 3.0 |

**Compute weighted average:**

Numerator $\sum S_i w_i$:

| Name | Calculation | Result |
|------|-------------|--------|
| 1 | $60 \cdot 4.5$ | 270 |
| 2 | $80 \cdot 4.4$ | 352 |
| 3 | $100 \cdot 4.3$ | 430 |
| 4 | $120 \cdot 4.1$ | 492 |
| 5 | $200 \cdot 3.0$ | 600 |

Total $= 2{,}144$

Denominator $\sum w_i = 4.5 + 4.4 + 4.3 + 4.1 + 3.0 = 20.3$

**Intrinsic spread:**

$$S_{\text{intrinsic}} = \frac{2{,}144}{20.3} = 105.67 \text{ bp}$$

Quoted index spread $S_I = 110$ bp.

**Basis:**

$$B = 110 - 105.67 = 4.33 \text{ bp}$$

**Sign convention used here:** positive basis means quoted wider than intrinsic.

---

### Example 9: Basis Position P&L Decomposition (Risk-First)

Construct a "basis-neutral" position:
- Long protection on index $N_I = 50\text{mm}$.
- Short protection on intrinsic replication sized so CS01 cancels.

**Assume:**
- Index CS01 per \$1mm (short prot) = 450 \$/bp ⇒ long protection CS01 per \$1mm = $-450$.
- So index CS01 for position: $\text{CS01}_I = -450 \times 50 = -22{,}500$ \$/bp.
- Constituents sized so their short-protection CS01 sums to $+22{,}500$ \$/bp.

Total CS01 $\approx 0$.

**Now scenarios:**

**(i) Parallel widening +10 bp (index and intrinsic both widen)**

Net CS01 is zero ⇒

$$\Delta V \approx -\text{CS01}_{\text{tot}} \cdot 10 \approx 0$$

**(ii) Basis tightening by 5 bp (index tightens 5 bp, constituents unchanged)**

$\Delta S_I = -5$, $\Delta S_{\text{const}} = 0$

$$\Delta V \approx -\text{CS01}_I \cdot \Delta S_I = -(-22{,}500) \times (-5) = 22{,}500 \times (-5) = -112{,}500$$

**Loss:** long index protection loses when index tightens.

**(iii) Dispersion widening (index unchanged, constituents disperse)**

Assume constituent spread changes (bp): $[+20, -15, +10, -5, -10]$.

Assume their CS01s (short protection, in \$/bp) sum to 22,500 but differ by name:

$$[5{,}000, \; 4{,}000, \; 5{,}500, \; 4{,}500, \; 3{,}500] \; \$/\text{bp} \quad (\text{sum } 22{,}500)$$

Constituent P&L (short protection):

$$\Delta V_{\text{const}} = -\sum \text{CS01}_i \cdot \Delta S_i$$

**Compute:**

| Name | Calculation | Result |
|------|-------------|--------|
| 1 | $-5{,}000 \times 20$ | $-100{,}000$ |
| 2 | $-4{,}000 \times (-15)$ | $+60{,}000$ |
| 3 | $-5{,}500 \times 10$ | $-55{,}000$ |
| 4 | $-4{,}500 \times (-5)$ | $+22{,}500$ |
| 5 | $-3{,}500 \times (-10)$ | $+35{,}000$ |

**Sum:**

$$-100{,}000 + 60{,}000 - 55{,}000 + 22{,}500 + 35{,}000 = -37{,}500$$

Index leg unchanged ($\Delta S_I = 0$) ⇒ $\Delta V_I \approx 0$.

**Net:** $-37{,}500$.

**Interpretation:** basis-neutral to parallel moves, but not to dispersion.

---

### Example 10: Default Event in a Constituent — Cash Settlement Contribution to Index Cashflow (Toy)

**Sources:**
- Cash settlement pays face minus recovery price.
- Index defaults reduce premium leg payments by $1/M$ increments.
- CDS VOD includes accrued premium term.

**Toy index:**
- $M = 5$, index notional $N_I = 50\text{mm}$ ⇒ per-name notional $N_m = 50/5 = 10\text{mm}$.
- Contractual coupon $C = 100$ bp $= 0.01$.
- Default occurs with auction final price $FP = 40$ (per 100 ⇒ 40% recovery proxy).
- Accrual fraction since last coupon $\Delta_0 = 0.20$ years.

**(a) Protection payment (to protection buyer)**

$$\text{Payout} = N_m \left(1 - \frac{FP}{100}\right) = 10{,}000{,}000 \times (1 - 0.40) = 10{,}000{,}000 \times 0.60 = 6{,}000{,}000$$

**(b) Accrued premium at default (paid by protection buyer)**

$$\text{Accrued} = C \cdot \Delta_0 \cdot N_m = 0.01 \times 0.20 \times 10{,}000{,}000 = 20{,}000$$

**(c) Net default cashflow to protection buyer (simplified)**

$$6{,}000{,}000 - 20{,}000 = 5{,}980{,}000$$

**(d) Post-default notional**

Contractual premium leg payments reduce by $1/M$ after default (toy: notional becomes \$40mm).

**I'm not sure** about the exact operational netting/timing of index default cashflows vs coupon schedules for your exact index vintage; we'd need the index rulebook + ISDA auction settlement details.

---

### Example 11: Before/After Risk Report — CS01, VOD/JTD Proxy, Recovery Sensitivity

Continue Example 10, assume we are long protection on the index.

**Before default**

Assume index CS01 per \$1mm (long protection) $= -450$ \$/bp.

Notional \$50mm:

$$\text{CS01}_{\text{pre}} = -450 \times 50 = -22{,}500 \; \$/\text{bp}$$

**Event measure (per default of one name, toy VOD proxy)**

From Example 10 net default cashflow per default $= +5.98\text{mm}$ to protection buyer.

**Recovery/final price sensitivity**

$$\frac{\partial \text{Payout}}{\partial FP} = -\frac{N_m}{100} = -\frac{10{,}000{,}000}{100} = -100{,}000 \; \$/\text{point}$$

**After default**

Notional reduces from 50mm to 40mm (premium leg reduced by $1/M$ increments).

If CS01 per \$1mm unchanged (simplifying assumption):

$$\text{CS01}_{\text{post}} = -450 \times 40 = -18{,}000 \; \$/\text{bp}$$

**Interpretation:**
- Spread risk per bp (trade-level) shrinks in magnitude as notional shrinks.
- Event risk remains large (one default produces a multi-million payout), and depends strongly on $FP$.

---

### Example 12: Recovery/Final Price Sensitivity Sweep (FP = 25, 40, 55)

Per-name notional $N_m = 10\text{mm}$.

**Payout to protection buyer:**

$$\text{Payout}(FP) = 10{,}000{,}000 \left(1 - \frac{FP}{100}\right)$$

| $FP$ | Calculation | Payout |
|------|-------------|--------|
| 25 | $1 - 0.25 = 0.75$ | \$7.5mm |
| 40 | $1 - 0.40 = 0.60$ | \$6.0mm |
| 55 | $1 - 0.55 = 0.45$ | \$4.5mm |

**Range** $= 7.5 - 4.5 = 3.0\text{mm}$ per defaulted name.

**Interpretation:** default-event P&L is highly sensitive to auction/final price.

---

### Example 13: Series Roll Mechanics (Toy) — Old vs New Series Upfront and Hedge Choice

**Sources:**
- New index longer maturity by ~6 months; upward sloping curves tend to make it wider.
- Liquidity drops after roll and can increase hedging costs (esp. options).

**I'm not sure** about your exact roll date and operational calendar; we'd need the index rulebook / market convention.

**Toy:**
- Notional $N = 100\text{mm}$
- Contractual coupon $C = 100$ bp $= 0.01$
- RPV01 (per unit spread) $\approx 4.5$ years (toy)
- Old series quoted spread $S_{\text{old}} = 95$ bp $= 0.0095$
- New series quoted spread $S_{\text{new}} = 100$ bp $= 0.0100$

**Upfront formula** for a flat curve quote (conceptual form appears in O'Kane for index): $U = (S - C) \cdot \text{RPV01}$.

**Compute upfronts:**

**Old:**

$$U_{\text{old}} = (0.0095 - 0.0100) \times 4.5 \times 100{,}000{,}000 = (-0.0005) \times 4.5 \times 100{,}000{,}000$$

First multiply: $(-0.0005) \times 4.5 = -0.00225$.

Then:

$$U_{\text{old}} = -0.00225 \times 100{,}000{,}000 = -225{,}000$$

**New:**

$$U_{\text{new}} = (0.0100 - 0.0100) \times 4.5 \times 100{,}000{,}000 = 0$$

**Cash difference (new − old):**

$$0 - (-225{,}000) = +225{,}000$$

**Interpretation (risk-first):**

Hedging old-series risk with new-series instruments embeds a series basis of ~5 bp in this toy, plus liquidity/membership considerations.

---

### Example 14: Execution Cost Impact — Bid/Ask Dominates Basis P&L (Toy)

**Toy:**
- You run a basis-neutral position like Example 9 with net CS01 ≈ 0.
- Index leg notional \$50mm; index CS01 magnitude 22,500 \$/bp.
- Assume bid/ask half-spreads:
  - Index: 0.25 bp
  - Constituents (aggregate): effective 1.50 bp (wider and fragmented)

**Execution cost approximation:**

$$\text{Cost} \approx \text{Half-spread} \times |\text{CS01}|$$

(Spread × CS01 converts to dollars; unit check: bp × \$/bp = \$).

**Index cost:**

$$0.25 \times 22{,}500 = 5{,}625$$

**Constituent cost:**

$$1.50 \times 22{,}500 = 33{,}750$$

**Total round-trip-ish execution burden (toy):**

$$5{,}625 + 33{,}750 = 39{,}375$$

**Interpretation:** even modest basis moves can be overwhelmed by execution costs when hedging with many single names—consistent with the liquidity-basis intuition in O'Kane's index basis discussion.

---

## 9. Practical Notes

### 9.1 Risk Report Glossary

**Index CS01 (Credit DV01)**
- Definition: $\text{CS01} = -(V(S+1) - V(S))$.
- Bump method: 1 bp parallel bump to the quoted spread curve level (flat index curve assumption for quoting is common in the source discussion).

**Single-name CS01**
- Same definition; computed on that name's spread curve.

**Basis**
- $B = S_I - S_I^{\text{intrinsic}}$; drivers include contractual differences, liquidity, and lead-lag.

**Event/default settlement mapping**
- Cash settlement payout = face − recovery price (final price).
- CDS VOD decomposition includes accrued premium.
- Index premium leg reduces by $1/M$ increments after defaults.

---

### 9.2 Common Pitfalls

1. **Mixing quoting regimes** (coupon+upfront vs par spread) inconsistently:
   - Index quotes use $C(T)$ and upfront $U_I(t)$ with a quoted $S_I(t,T)$.

2. **Using inconsistent bump definitions** across index and constituents

3. **Thinking an index hedge removes idiosyncratic risk** (it does not; see dispersion examples)

4. **Ignoring roll/series liquidity changes** (post-roll liquidity drop raises costs)

5. **Assuming index default-handling mechanics without sourcing**
   - Always verify with the index rulebook and ISDA definitions for your trade.

---

### 9.3 Implementation Pitfalls

**Constituent weights and membership changes**
- O'Kane: constituents fixed for series life, removed on default; new series can change membership.

**Missing data / stale quotes for constituents**

**Discount curve/recovery assumption mismatches** across index vs single-name analytics

**PSA implementation details:**
- Spread multiplier and bootstrap issues including maturity-date alignment/interpolation are explicitly flagged.

---

### 9.4 Verification Tests

- **Parallel shock test:** CS01-hedged book should be near-flat for parallel $\Delta S$
- **Dispersion/idiosyncratic scenario:** residual P&L should match explained net single-name exposures
- **Scaling with notional:** CS01 and event measures scale linearly (first-order)
- **Payout bounds:** for $FP \in [0, 100]$, payout in $[0, N]$; sensitivity $-N/100$ \$/point

---

## 10. Summary & Recall

### 10.1 10-Bullet Executive Summary

1. CDS indices bundle many names into one liquid macro instrument; series constituents are fixed except defaults remove names without replacement.

2. Market quotes indices using a simplified "index curve" (often flat in the source discussion) and contractual coupon $C(T)$, leading to upfront $U_I(t)$.

3. Intrinsic valuation builds the index PV from constituent CDS curves.

4. Quoted vs intrinsic spreads differ: the index basis reflects contractual differences, liquidity, and flow/lead-lag.

5. Hedging starts with CS01/Credit DV01 matching, defined as in O'Kane.

6. Top-down hedges (single name → index) remove macro risk but keep idiosyncratic, basis, and event residuals.

7. Bottom-up hedges (index → constituents) aim to replicate intrinsic but keep basis and execution dispersion; PSA exists to align intrinsic with index quotes.

8. Default events change index cashflows and notional: premium leg reduces by $1/M$ increments; event payout depends on recovery/final price.

9. Roll/series effects matter: liquidity drops after roll; hedging OTR with RTR can create membership mismatch.

10. Always validate hedges with scenario sets: parallel, dispersion, single-name gap, default event, roll shift, and execution cost stress.

---

### 10.2 Cheat Sheet: Hedge Ratios, Basis Decomposition, Scenario Validation Suite

**CS01 definition:**

$$\text{CS01} = -(V(S+1) - V(S))$$

**First-order P&L:**

$$\Delta V \approx -\text{CS01} \cdot \Delta S$$

**CS01 hedge notional:**

$$N_{\text{hedge}} = -\frac{\text{CS01}_{\text{position}}}{\text{CS01}_{\text{hedge}}(1)}$$

**Basis:**

$$B = S_I - S_I^{\text{intrinsic}}$$

Intrinsic spread is close to RPV01-weighted average (approx).

**Default cash settlement payoff:**

$$\text{Payout} = N \left(1 - \frac{FP}{100}\right)$$

**Validation suite:**

| Scenario | Description |
|----------|-------------|
| 1 | Parallel ±10bp |
| 2 | Dispersion (same average) |
| 3 | Single-name gap (+50bp vs index +10bp) |
| 4 | Default event (FP sweep) |
| 5 | Roll/series basis move |
| 6 | Bid/ask widening stress |

---

### 10.3 35 Flashcards (Q/A)

1. **Q:** What is the index basis?
   **A:** $S_I - S_I^{\text{intrinsic}}$; drivers include contractual differences, liquidity, lead/lag.

2. **Q:** What does "intrinsic value" of an index mean?
   **A:** PV implied by constituent CDS curves (sum/average representation).

3. **Q:** What is the contractual index coupon $C(T)$?
   **A:** Fixed series coupon used for premium payments; set near fair value and typically a multiple of 5bp.

4. **Q:** Define Credit DV01/CS01.
   **A:** $-[V(S + 1\text{bp}) - V(S)]$ for a 1bp parallel spread increase.

5. **Q:** Why is the sign defined with a negative in DV01?
   **A:** So short-protection DV01 is positive.

6. **Q:** What is RPV01 conceptually?
   **A:** PV factor for premium leg per unit spread, including accrued-at-default in CDS formulation.

7. **Q:** What is VOD for a CDS?
   **A:** Change in value on immediate default; includes MTM cancellation, protection payment, accrued premium.

8. **Q:** What happens to index premium payments after a constituent default (in O'Kane)?
   **A:** Contractual spread payments reduce by $1/M$.

9. **Q:** Are index constituents replaced after default?
   **A:** No; defaulted names are removed without replacement (series life).

10. **Q:** Why can quoted index spread differ from intrinsic?
    **A:** No-Re vs Mod-Re, liquidity differences, index leads in stress.

11. **Q:** What is portfolio swap adjustment (PSA)?
    **A:** Adjust constituent curves so intrinsic matches quoted index; method is somewhat arbitrary but aims to be stable/fast.

12. **Q:** Give PSA spread multiplier form.
    **A:** $S_m^*(t,T) = \alpha(T) S_m(t,T)$.

13. **Q:** Why might PSA require a root solve?
    **A:** $\alpha(T)$ appears inside RPV01; no simple linear solution.

14. **Q:** What is top-down hedging?
    **A:** Hedging single-name/portfolio risk using an index proxy.

15. **Q:** What residual remains in a top-down hedge?
    **A:** Idiosyncratic spread risk, name–index basis, event mismatch.

16. **Q:** What is bottom-up hedging?
    **A:** Hedging an index using constituents (replication).

17. **Q:** What residual remains in bottom-up hedging?
    **A:** Index basis + execution dispersion + missing constituents + defaults/membership evolution.

18. **Q:** What is series risk?
    **A:** Risk from hedging across index series with different maturity/membership/liquidity.

19. **Q:** Why are index option expiries beyond six months rare (per O'Kane)?
    **A:** Semiannual roll reduces liquidity after roll, raising hedging costs.

20. **Q:** How is cash settlement payout determined?
    **A:** Face minus recovery price (auction/final price).

21. **Q:** What's the sensitivity of payout to final price $FP$?
    **A:** $-N/100$ dollars per price point.

22. **Q:** What does "index curve is flat" mean in quoting?
    **A:** Market often solves for a single level $S_I(t,T)$ used as a flat curve for quoting.

23. **Q:** What is "intrinsic premium leg PV" for an index in O'Kane?
    **A:** $\frac{C(T)}{M} \sum_m \text{RPV01}_m(t,T)$.

24. **Q:** Why doesn't an index hedge remove idiosyncratic moves?
    **A:** Constituents can move differently (dispersion); index captures average macro move only.

25. **Q:** What scenario best reveals dispersion risk?
    **A:** High-dispersion moves with unchanged average spread.

26. **Q:** What scenario best reveals event risk?
    **A:** Immediate default with FP sweep and accrued premium.

27. **Q:** What scenario best reveals roll risk?
    **A:** Old-series vs new-series spread move + liquidity/bid-ask widening.

28. **Q:** What is the key hedging warning about on-the-run vs off-the-run?
    **A:** Membership mismatch can leave unhedged credits.

29. **Q:** What does it mean that the new index has longer maturity by six months?
    **A:** Series renewal changes tenor; upward curves can widen new series.

30. **Q:** When is CS01 matching most reliable?
    **A:** For small parallel spread moves (first-order).

31. **Q:** When does CS01 matching fail?
    **A:** Non-parallel curve moves, dispersion, defaults, basis shifts, liquidity shocks.

32. **Q:** How do you incorporate basis into modeling for bottom-up hedges?
    **A:** Use PSA to align adjusted intrinsic with quoted index.

33. **Q:** What is the "index buyer" default loss scaling in O'Kane?
    **A:** Default of name $m$ causes immediate loss $(1 - R_m)/M$ to index buyer.

34. **Q:** What cashflow is often overlooked at default?
    **A:** Coupon/premium accrued at default.

35. **Q:** What document do you need to pin down index default mechanics precisely?
    **A:** Index rulebook + ISDA definitions/auction protocol for your trade vintage (not fully specified here).

---

## 11. Mini Problem Set (18 Questions)

*Provide brief solution sketches for questions 1–9 only.*

---

**1.** Compute CS01 from PVs: $V(S-1) = 101{,}200$, $V(S) = 100{,}000$, $V(S+1) = 98{,}700$.

**Sketch:** Central slope $= (98{,}700 - 101{,}200)/2 = -1{,}250$ \$/bp; CS01 $= +1{,}250$ \$/bp.

---

**2.** A long-protection position has CS01 $-6{,}000$ \$/bp. What is the first-order P&L for a +8bp widening?

**Sketch:** $\Delta V \approx -(-6{,}000) \times 8 = +48{,}000$.

---

**3.** Index CS01 per \$1mm is 420 \$/bp. You need +12,600 \$/bp of hedge CS01. What index notional do you trade (short protection)?

**Sketch:** $N = 12{,}600 / 420 = 30\text{mm}$.

---

**4.** You hedge a single name with an index by CS01 matching. Single name widens +40bp, index widens +15bp. Compute residual P&L given CS01s: $\text{CS01}_{\text{SN}} = -5{,}000$ \$/bp, $\text{CS01}_I = +5{,}000$ \$/bp.

**Sketch:** $\Delta V = -(-5{,}000) \cdot 40 - (+5{,}000) \cdot 15 = 200{,}000 - 75{,}000 = 125{,}000$.

---

**5.** Two-factor hedge: you have exposures $(\text{CS01}^{(5)}, \text{CS01}^{(10)}) = (10{,}000, 6{,}000)$. Hedge instruments B and C have vectors (per \$1mm): $B = (400, 100)$, $C = (100, 300)$. Solve hedge notionals $x_B, x_C$ (mm) to neutralize both.

**Sketch:** Solve linear system:

$$10{,}000 + 400 x_B + 100 x_C = 0$$
$$6{,}000 + 100 x_B + 300 x_C = 0$$

Multiply second by 4: $24{,}000 + 400 x_B + 1{,}200 x_C = 0$. Subtract first: $14{,}000 + 1{,}100 x_C = 0 \Rightarrow x_C = -12.727$. Plug back: $10{,}000 + 400 x_B - 1{,}272.7 = 0 \Rightarrow 400 x_B = -8{,}727.3 \Rightarrow x_B = -21.818$.

---

**6.** Compute intrinsic spread (RPV01-weighted) from 3 names: spreads (bp) $= [80, 120, 200]$, weights $= [4.0, 3.5, 2.5]$.

**Sketch:** Numerator $= 80 \cdot 4 + 120 \cdot 3.5 + 200 \cdot 2.5 = 320 + 420 + 500 = 1{,}240$. Denominator $= 10$. Intrinsic $= 124$ bp.

---

**7.** For a cash-settled default with notional $N = 5\text{mm}$ and final price $FP = 35$, compute payout.

**Sketch:** $5{,}000{,}000 (1 - 0.35) = 3{,}250{,}000$.

---

**8.** Compute accrued premium at default for coupon $C = 500$bp, $\Delta_0 = 0.10$ years, notional $N = 5\text{mm}$.

**Sketch:** $0.05 \times 0.10 \times 5{,}000{,}000 = 25{,}000$.

---

**9.** An index has $M = 100$ and notional 200mm. After 3 defaults, what is remaining notional under the equal-weight $1/M$ reduction assumption?

**Sketch:** Per default reduction = $200/100 = 2\text{mm}$ ⇒ remaining $= 200 - 3 \cdot 2 = 194\text{mm}$. (Assumption aligns with $1/M$ reduction described in O'Kane mechanics.)

---

**10.** Explain why hedging an off-the-run index with an on-the-run index can leave residual risk even if CS01 is matched. *(No solution sketch.)*

---

**11.** Describe how PSA changes a bottom-up hedge and why it is described as somewhat arbitrary. *(No solution sketch.)*

---

**12.** Design a 5-scenario hedge validation suite for an index basis-neutral position. *(No solution sketch.)*

---

**13.** Provide a qualitative argument for why index options beyond six months are less common (per O'Kane). *(No solution sketch.)*

---

**14.** Show how a dispersion scenario can produce P&L even if the average spread move is zero. *(No solution sketch.)*

---

**15.** How does recovery/final price risk differ between a single-name default and an index default event aggregation? *(No solution sketch.)*

---

**16.** Propose a reporting format that separates systematic, idiosyncratic, basis, event, and roll risks. *(No solution sketch.)*

---

**17.** Discuss why consistent bump definitions across index and constituents matter in basis measurement. *(No solution sketch.)*

---

**18.** Explain what additional documents are needed to precisely model index default settlement mechanics for a real trade. *(No solution sketch.)*

---

## Source Map

### (A) Verified Facts — Source-Backed

- Index mechanics, intrinsic valuation, basis drivers, PSA methodology: O'Kane, *Modeling Single-Name and Multi-Name Credit Derivatives*, Chapters 9-10
- CS01 / Credit DV01 definition and sign convention: O'Kane, Chapter 8
- VOD decomposition and jump-to-default risk: O'Kane, Chapter 8
- Index roll mechanics and liquidity considerations: O'Kane, Chapter 10
- Cash settlement and auction protocol: O'Kane, Chapter 5; ISDA 2005 protocol

### (B) Reasoned Inference — Derived from (A)

- Hedge ratio algebra (CS01 matching, two-factor solve, least-squares extension)
- P&L decomposition template
- Execution cost impact calculation
- Scenario test residual analysis

### (C) Speculation — Flagged as Uncertain

- Beta estimation methods for proxy hedges (desk-specific)
- Exact index weighting schemes (rulebook-dependent)
- Precise operational netting/timing of default cashflows (ISDA version-dependent)
- Front-end protection mechanics (not fully specified in sources)
