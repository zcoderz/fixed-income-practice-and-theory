# Chapter 52: Credit Trading Strategies (Risk + Instrument + Hedge)

---

## Conventions & Notation

All numbers are educational toy examples (no real market data; no trade recommendations).

### Currency / Scaling

- PV, P&L, sensitivities are reported in USD.
- Unless stated otherwise, sensitivities are scaled per $1mm notional for single-name CDS and cash bonds, and per $10mm index notional in the index examples (explicit in each example).

### Basis Points vs Decimals

- $1 \text{ bp} = 10^{-4}$ per annum.
- A spread move written as $\Delta S = +10 \text{ bp}$ means $\Delta S = +0.0010$ in decimal.

### CDS / Index Coupon and Day Count

- $C$ denotes the contractual running coupon (bp/yr).
- For index examples, coupons are assumed paid quarterly using an Actual/360 year fraction $\Delta$, consistent with the cited index mechanics.

### Clean vs Full (Dirty) MTM for CDS

$$\text{Clean MTM} = \text{Full MTM} - \text{Accrued}$$

Accrued premium sign convention (between coupon dates): positive for short protection, negative for long protection (because short receives, long pays).

### CS01 Sign Convention (Explicit)

Definition used here:

$$\text{CS01} = \frac{\partial PV}{\partial S} \times (1 \text{ bp})$$

where $S$ is the quoted/par spread being bumped (single-name CDS spread, index spread, tranche spread—explicitly stated each time).

- **Long protection CDS:** $PV$ increases when $S$ widens $\Rightarrow$ CS01 $> 0$.
- **Short protection CDS:** CS01 $< 0$.
- **Long cash bond:** $PV$ decreases when credit spread widens $\Rightarrow$ bond "credit CS01" $< 0$ (if you measure it as $\Delta PV$ for $+1$ bp spread).

### Jump-to-Default (JTD) Sign Convention

We measure JTD as the instantaneous PV jump at default (a scenario jump, not an infinitesimal derivative):

$$\text{JTD} \equiv PV_{\text{post-default}} - PV_{\text{pre-default}}$$

For long protection CDS, JTD is typically positive (you receive protection payment net of accrued premium). This is the value-on-default (VOD) framework in the sources.

---

## 0. Setup

### Conventions Used in This Chapter

(Quoting regime assumptions; clean/dirty; bp vs decimals; per-$1mm scaling)

- Spreads are quoted in bp/year; valuation uses decimals with $1 \text{ bp} = 10^{-4}$.
- **Single-name CDS valuation identity (risk measure anchor):** the mark-to-market of a standard running-spread CDS can be written as

$$V(t,T) = (S(t,T) - S_0) \cdot \text{RPV01}(t,T)$$

(per unit notional), where $\text{RPV01}$ is the risky PV01 of the premium leg.

### Index Contracts (Mechanics Assumed)

- Index coupon $C$ is set near fair value (multiple of 5 bp), paid quarterly on an Actual/360 basis; an upfront payment $U_I$ is exchanged at settlement; index rolls every six months.
- **Default handling for index:** on a constituent credit event, the protection seller pays loss on the defaulted name and the index notional is reduced proportionally (simplified as $1/M$ per default in these notes, consistent with the cited mechanics).
- **Scaling:** when we report CS01/JTD/RecSens, we always state "per $1mm" or "per $10mm".

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $t$ | Valuation time (years) |
| $T$ | Contract maturity time (years) |
| $S(t,T)$ | Market/par CDS spread for maturity $T$ observed at time $t$ (bp/yr unless stated) |
| $S_0$ or $C$ | Contractual running spread/coupon of the position (bp/yr) |
| $\Delta$ | Accrual year fraction for a coupon period (e.g., Actual/360) |
| $Z(t,u)$ | Discount factor from $t$ to $u$ |
| $Q(t,u)$ | Survival probability from $t$ to $u$ |
| $R$ | Recovery rate (decimal, e.g., 0.40) |
| $\text{RPV01}(t,T)$ | Risky PV01 (PV of 1 bp/yr premium leg per unit notional) |
| $\text{CS01}$ | Spread DV01 (PV change for +1 bp change in quoted spread) |
| $\text{DV01}_r$ | Rates DV01 (PV change for +1 bp parallel move in chosen risk-free curve) |
| $\text{VOD}$ | Value-on-default (PV jump at default), used as JTD measure |
| $U_I$ | Index upfront payment (currency) |
| $M$ | Number of names in an index |
| $A, B$ | Tranche attachment/detachment points (as portfolio loss fractions) |
| $L$ | Portfolio loss fraction over horizon (0–1) |
| $L_{[A,B]}$ | Tranche loss fraction for tranche $[A,B]$ |

---

## 1. Core Concepts (Definitions First)

### 1.1 CS01 / Spread DV01 (and What "Spread" Is Being Bumped)

#### Formal Definition (Units and Sign)

For any instrument valued as $PV(S, \cdot)$, define

$$\boxed{\text{CS01} \equiv \frac{\partial PV}{\partial S} \times 10^{-4} \quad \text{(USD per 1 bp)}}$$

What spread is bumped must be specified:

- **Single-name CDS:** bump the market par spread $S(t,T)$.
- **Index CDS:** bump the quoted index spread $S_I(t,T)$ or (alternatively) bump constituents and re-imply an "intrinsic" index spread (two different risk views; mixing them is a pitfall).
- **Tranche:** bump the tranche market spread (or tranche coupon/price quote, depending on convention).

#### Source-Backed Anchor (Single-Name CDS)

Using the CDS MTM identity

$$V(t,T) = (S(t,T) - S_0) \cdot \text{RPV01}(t,T),$$

a small spread move gives

$$dV \approx \text{RPV01}(t,T) \, dS,$$

so per unit notional:

$$\boxed{\text{CS01}_{\text{CDS}} \approx \text{RPV01}(t,T) \times 10^{-4}}$$

#### Intuition

CS01 is the "linear spread risk": how much PV changes for small spread moves.

#### How It Appears in Practice

Risk systems compute $\text{RPV01}$ and CS01 by maturity; traders/risk managers size hedges to neutralize CS01 (or bucketed CS01).

---

### 1.2 Rates DV01 (Only as Needed for Cash Bond Legs)

#### Formal Definition

Let $r$ represent the relevant risk-free curve level (or yield). Define

$$\boxed{\text{DV01}_r \equiv \frac{\partial PV}{\partial r} \times 10^{-4} \quad \text{(USD per 1 bp)}}$$

#### Intuition

Rates DV01 isolates the interest-rate exposure of funded cash instruments (bonds) and some swap legs.

#### Practice

Duration/DV01 hedging is standard; it immunizes against small parallel shifts but not non-parallel curve changes (key-rate risk).

---

### 1.3 Jump-to-Default (JTD) Exposure

#### Formal Definition

For a position with pre-default PV $PV$ and post-default PV $PV'$:

$$\boxed{\text{JTD} \equiv PV' - PV \quad \text{(USD)}}$$

#### Source-Backed CDS Expression (Value-on-Default / VOD)

For a CDS, the value-on-default is expressed (per unit notional) as:

$$\text{VOD} = (1 - R - \text{Accrued Premium}) - (S(t,T) - S_0) \cdot \text{RPV01}(t,T)$$

(long protection sign).

**Mapping:** in these notes, JTD = VOD × Notional.

#### Intuition

CS01 is "small move"; JTD is the discontinuity when default happens.

#### Practice

Hedging CS01 does not hedge JTD (especially single-name default vs index hedge).

---

### 1.4 Recovery / Final-Price Sensitivity

#### Formal Definition

Recovery sensitivity (per unit notional) can be measured as

$$\text{Rec01} \equiv \frac{\partial PV}{\partial R} \times 0.01 \quad \text{(USD per 1\% recovery)}$$

#### Source-Backed CDS Recovery Sensitivity

A recovery rate sensitivity can be expressed in terms of the CDS value and $(1 - R)$ (book's recovery DV01 expression).

#### Final Price

CDS protection leg can be cash-settled with payment based on final price of the reference obligation, determined by dealer poll/auction; payoff is based on face value minus recovery price.

#### Intuition

Recovery is a second key state variable in default scenarios; many "basis" and "hedge" surprises come from mismatched recovery assumptions.

---

### 1.5 (For Indices) Series/Roll Basis and "Intrinsic vs Quoted" Basis

#### Intrinsic Value vs Market Value

The intrinsic upfront value of an index (viewed as an equally weighted portfolio of constituent CDS) is:

$$V_I(t) = \frac{1}{M} \sum_{m=1}^{M} (S_m(t,T) - C(T)) \cdot \text{RPV01}_m(t,T)$$

The market upfront value using a flat index curve is:

$$U_I(t) = (S_I(t,T) - C(T)) \cdot \text{RPV01}_I(t,T)$$

#### Index Basis

The index basis is framed in the sources as the gap between quoted index spread and a CDS-implied intrinsic index spread (example table).

#### Series/Roll Basis

Indices roll every six months; on-the-run liquidity and maturity reset can create price/spread differences between old and new series (liquidity/technical effects are explicitly noted).

#### Intuition

- "Quoted vs intrinsic" is a model/replication gap.
- "Series basis" is a liquidity + contract cohort gap.

---

### 1.6 (For Tranches) Tranche PV01, Correlation Sensitivity, and Tail/Default Clustering Scenario Risk

#### Tranche PV01

The tranche risky PV01 is defined analogously to CDS premium-leg PV01, using expected discounted outstanding tranche notional; the source gives a tranche $\text{RPV01}$ expression and defines tranche PV01 as the sensitivity to 1 bp spread change.

#### Correlation Sensitivity

"Correlation 01" is defined as PV change for a 1% increase in correlation parameter, with sign depending on tranche (equity vs senior behave differently).

#### Tail/Clustering Risk

Tranche risk is nonlinear in portfolio loss. Tranche loss fraction:

$$\boxed{L_{[A,B]} = \frac{1}{B - A} \left( \max(L - A, 0) - \max(L - B, 0) \right)}$$

Clustering scenarios can swamp small-spread hedges (PV01 hedges fail because payoff is driven by realized losses).

---

### 1.7 Carry and Rolldown Definitions (Cash Bonds; CDS Indices)

#### Cash Bond Carry/Rolldown (Brief)

For a bond portfolio, carry is the coupon accrual minus financing cost; rolldown is price change from moving "down" a sloped curve as time passes (source provides carry expression components and return framing).

#### CDS / Index Carry and Theta

The sources define carry (for tranches) as daily coupon accrued and theta as the daily change in full price; theta includes the effect of rolling down/up an upward/downward sloped spread curve.

CDS theta is similarly defined as the day-to-day change in value holding key inputs fixed.

#### What We Do in This Chapter

We use carry + rolldown + spread move as a P&L explain framework for index positions, treating "rolldown" as the component of theta due to curve slope (derived from the theta definition).

---

### 1.8 Liquidity/Funding/Financing Risk (Conceptual, Only as Supported)

#### Bond–CDS Basis Drivers (Source-Backed)

The CDS–cash basis is defined as:

$$\text{CDS basis} = \text{CDS spread} - \text{Bond Libor spread}$$

with discussion that the bond spread choice is typically asset swap spread for fixed-rate bonds near par.

**Drivers for persistence include:** funding differences (unfunded CDS vs funded bonds), delivery option, technical default, loss-on-default mismatch, premium accrued at default, and market liquidity/supply-demand factors.

#### Intuition

"Cheap" vs "rich" often reflects funding/liquidity optionality rather than pure expected default loss.

---

## 2. The Chapter's Governing Rule: "Strategy = Exposures + Hedges + Failure Modes"

**Principle:** A "credit strategy" is not a slogan. It is a risk blueprint:

$$\text{Strategy} = \underbrace{\text{exposure decomposition}}_{\text{what you truly own}} + \underbrace{\text{hedge set + ratios}}_{\text{what you try to neutralize}} + \underbrace{\text{scenario suite + failure modes}}_{\text{what can still break}}$$

This is consistent with the risk management emphasis on stress/scenario thinking and "perfect storm" risk (multiple adverse moves at once).

### Strategy Card Template (Repeat for Each Strategy)

1. **Objective** / what mispricing or risk premium the strategy is targeting (conceptual, no forecasts)
2. **Instruments and position construction** (legs, notionals, maturities)
3. **Exposure decomposition** (a table with units), including at least:
   - CS01 by maturity bucket (or clearly stated alternative)
   - JTD
   - recovery sensitivity
   - basis sensitivity (when relevant)
   - (if relevant) rates DV01 / funding carry
4. **Hedge set and hedge ratios** (show math)
5. **Scenario test suite:**
   - parallel spread move
   - dispersion / idiosyncratic move
   - default event (final price/recovery shock)
   - for indices: roll/series basis change
   - for tranches: correlation/tail/clustering shock
6. **Failure modes** (what breaks and why)
7. **Implementation checklist + verification tests** (repricing checks, unit checks, bump stability)

---

## 3. Strategy Family A: Basis Strategies

### A1) Bond–CDS Basis Framework (Cash vs Synthetic)

#### Strategy Card: A1 — Bond–CDS Basis Package (Risk-First Framing)

##### Objective (Conceptual; No Forecasts)

Isolate and measure the CDS–cash basis:

$$\text{basis} = \text{CDS spread} - \text{bond Libor spread},$$

acknowledging it can persist due to contractual and market frictions (funding, delivery option, technical default, loss-on-default differences, premium accrued at default, liquidity).

##### Instruments and Position Construction

A generic basis package uses:

- **Cash bond** (funded instrument) or asset swap representation of the bond credit spread.
- **Single-name CDS** on the same reference entity (unfunded).
- Often an **interest-rate hedge** (swap/UST) to reduce rates DV01 of the bond leg.

Position direction is not prescribed; the educational focus is what risks remain after hedging.

##### Exposure Decomposition (Illustrative Buckets; Units Explicit)

Buckets used in this chapter: 0–3Y, 3–7Y, 7–10Y (coarse).

| Exposure (units) | Definition | Bond leg | CDS leg | Net intuition |
|------------------|------------|----------|---------|---------------|
| $\text{CS01}_{0\text{–}3}$ (USD/bp) | $\partial PV / \partial S \times 1\text{bp}$ | usually 0 | usually 0 | maturity placement |
| $\text{CS01}_{3\text{–}7}$ (USD/bp) | 5Y risk mostly here | $< 0$ (long bond) | $> 0$ (long prot) | can offset if sized |
| $\text{CS01}_{7\text{–}10}$ (USD/bp) | long-end bucket | 0 | 0 | — |
| Rates DV01 (USD/bp) | $\partial PV / \partial r \times 1\text{bp}$ | typically $\neq 0$ | ~0 | residual rates unless hedged |
| JTD (USD) | PV jump at default | $-(P - R) \times N$ | $+\text{VOD} \times N$ for long prot | can be matched, but not automatic |
| Rec01 (USD per 1%) | $\partial PV / \partial R \times 1\%$ | depends on bond recovery view | CDS recovery sensitivity | mismatches matter |
| Basis sensitivity | PV change when CDS and bond spreads move differently | yes | yes | this is the residual you keep |

##### Hedge Set and Hedge Ratios (Show Math)

**Rates hedge (DV01 neutralization):**

$$N_{\text{rates hedge}} = -\frac{\text{DV01}_r^{\text{bond}}}{\text{DV01}_r^{\text{hedge instr}}}$$

**Credit hedge (CS01 match):**

$$N_{\text{CDS}} = -\frac{\text{CS01}_{\text{bond}}}{\text{CS01}_{\text{CDS per \$ notional}}}$$

**Event hedge (JTD match):** motivated by the loss-on-default mismatch discussion (bond loss $P - R$ vs CDS loss $1 - R$).

##### Scenario Test Suite

- Parallel spread move (bond and CDS widen together).
- Dispersion (CDS widens, bond unchanged; or vice versa).
- Default event: vary recovery/final price; include accrued premium at default.
- Rates move: bond yield curve shift with and without DV01 hedge.
- (Optional) Delivery option/restructuring clause differences (qualitative).

##### Failure Modes

- **Funding/repo shocks:** the bond leg is funded; CDS is unfunded.
- **Delivery option / restructuring clause changes:** CDS embeds deliverability optionality.
- **Loss-on-default mismatch** when bond price $P \neq 100$.
- **Accrued-at-default** vs bond accrued coupon loss differences.
- **Liquidity/transaction cost** dominates small basis edges.

##### Implementation Checklist + Verification Tests

- Reprice each leg under the same discounting assumptions where applicable (and document differences).
- Verify signs: long protection CS01 $> 0$; long bond credit CS01 $< 0$.
- Bump stability: CS01 should be stable for $\pm 1$ bp bumps; if not, you are in nonlinear regime.
- Event test: run default today with recovery sweep.

---

### A2) Single-Name vs Index (Proxy Hedge / Basis)

#### Strategy Card: A2 — Proxy Hedge: Single-Name CDS Hedged by an Index

##### Objective (Conceptual; No Forecasts)

Reduce broad-market credit spread exposure of a single-name position using an index, leaving mainly idiosyncratic risk (and default jump risk).

##### Instruments and Position Construction

- **One leg:** single-name CDS (maturity $T$).
- **Hedge leg:** CDS index (similar maturity $T$ or nearest liquid point).
- If "beta" is used to scale the index hedge, treat beta as an input (estimation is not standardized in these sources).

##### Exposure Decomposition (Units Explicit; Bucketed CS01)

We bucket both single-name and index CS01 into 0–3Y / 3–7Y / 7–10Y (coarse).

| Exposure (units) | Single-name leg | Index hedge leg | Net risk comment |
|------------------|-----------------|-----------------|------------------|
| $\text{CS01}_{3\text{–}7}$ (USD/bp) | dominant | dominant | can be matched |
| JTD (USD) | large (single default) | small per-name (diversified) | not hedged by CS01 match |
| Rec01 (USD/1%) | meaningful | meaningful | can mismatch |
| Basis sensitivity | single vs index composition | yes | residual "proxy basis" |
| Roll/series basis | none | yes | index-specific residual |

##### Hedge Set and Hedge Ratios (Show Math)

**CS01 match:**

$$N_{\text{index}} = -\frac{\text{CS01}_{\text{single}}}{\text{CS01}_{\text{index per \$ notional}}}$$

**If a beta $\beta$ is imposed:**

$$N_{\text{index}} = -\frac{\beta \cdot \text{CS01}_{\text{single}}}{\text{CS01}_{\text{index per \$ notional}}}$$

(Beta treatment is a desk convention input in these notes.)

##### Scenario Test Suite

- Parallel market spread move (both single and index move similarly).
- Idiosyncratic widening/tightening of the single name with small index move.
- Single-name default (final price/recovery shock).
- Index roll/series basis change (index leg reprices).

##### Failure Modes

- **Composition mismatch:** index may not contain the name; weights differ; sector tilts.
- **Index basis:** quoted vs intrinsic; hedging off-the-run with on-the-run creates mismatch in constituents.
- **Default jump dominates.**

##### Implementation Checklist + Verification Tests

- Confirm maturity alignment and coupon conventions.
- Recompute CS01 after trade; ensure hedged bucket(s) are near zero.
- Run event stress: single-name default today; verify residual.

---

## 4. Strategy Family B: Carry, Rolldown, and Roll in CDS Indices

### Definitions (Index-Specific, Risk-First)

#### Index Upfront Mechanics (Source-Backed)

Index trades with a fixed coupon $C$ and an upfront payment at settlement; coupon is paid quarterly on Actual/360; index rolls every six months.

#### Carry (Index)

In this chapter, "carry" means the deterministic premium accrual/coupon cashflow component over a horizon (plus any deterministic accrual conventions), consistent with the sources' carry definition as coupon accrual (for tranche) and general swap cashflow logic.

#### Rolldown (Index)

"Rolldown" means the component of P&L from aging along a sloped spread curve holding the curve fixed (a component of theta). This is directly aligned with the theta definition that includes roll down/up effects for upward/downward sloped spread curves.

---

### B1) Index Carry/Rolldown Holding-Period P&L (Risk Decomposition, Not a Recommendation)

#### Strategy Card: B1 — Index Holding-Period Decomposition ("Carry + Rolldown + Spread Move + Events")

##### Objective (Conceptual; No Forecasts)

Build a disciplined P&L explain for an index position:

- cashflows (carry),
- curve aging (rolldown/theta),
- spread moves,
- default events and index mechanics.

##### Instruments and Position Construction

- Single index CDS position (long or short protection).
- Optional hedges: another index series (to manage roll exposure) or constituents (to manage default/event risk—imperfect).

##### Exposure Decomposition (Units Explicit)

| Exposure | Units | Notes |
|----------|-------|-------|
| CS01 buckets | USD/bp | linear spread risk |
| JTD (per index default) | USD | depends on $M$ and recovery/final price |
| Rec01 | USD/1% | recovery dependence |
| Roll/series basis | USD per bp of basis | difference old vs new series quotes/liquidity |
| Carry | USD/day or USD/horizon | coupon accrual (sign depends on long/short protection) |
| Rolldown | USD/horizon | theta due to curve slope |

##### Hedge Set and Hedge Ratios

If hedging spread risk with another index:

$$N_{\text{hedge}} = -\frac{\text{CS01}_{\text{target}}}{\text{CS01}_{\text{hedge}}}$$

If hedging with constituents, see strategy A2/Example 4 and note basis complications.

##### Scenario Test Suite

- Parallel spread move.
- Dispersion: constituents move differently from index ("intrinsic vs quoted" basis move).
- Default event inside index (final price shock; notional reduction).
- Roll/series basis change (on-the-run vs off-the-run repricing).
- Liquidity/execution-cost stress (widen bid/offer assumption).

##### Failure Modes

- Default cluster events invalidate linear approximations.
- Index basis and portfolio swap adjustment issues if hedging vs constituents.
- Execution costs dominate short-horizon carry/rolldown.

##### Implementation Checklist + Verification Tests

- Separate clean vs accrued P&L (coupon date jumps should net out with cash account).
- Verify CS01 with symmetric $\pm 1$ bp bumps.
- Event check: apply one-name default; verify notional reduction logic.

---

### B2) Roll / Series Switch Mechanics (On-the-Run vs Off-the-Run)

#### Strategy Card: B2 — Index Roll/Series Switch (Mechanics + Risk)

##### Objective (Conceptual; No Forecasts)

Understand P&L drivers when switching between series:

- maturity reset,
- coupon differences,
- liquidity/on-the-run premium.

##### Instruments and Position Construction

- Two index series (old vs new) with possibly different quoted spreads and remaining maturity.
- Switch involves: closing old + opening new (cash exchange equals MTM difference plus execution costs).

##### Exposure Decomposition

| Exposure | Units | Typical effect |
|----------|-------|----------------|
| Series basis | USD/bp | key driver |
| CS01 mismatch | USD/bp | residual spread risk if maturities differ |
| Default/event | USD | both series exposed, but composition differs |
| Liquidity | qualitative | on-the-run often tighter bid/offer |

##### Hedge Set and Hedge Ratios

If goal is spread-neutral switch:

$$N_{\text{new}} = N_{\text{old}} \cdot \frac{\text{CS01}_{\text{old}}}{\text{CS01}_{\text{new}}}$$

(sign consistent with long/short protection).

##### Scenario Test Suite

- Roll basis widens/tightens.
- Old series constituents differ from new (idiosyncratic composition change).
- Default just before/after roll (mechanical cashflow and notional effects).

##### Failure Modes

- Composition mismatch and basis shocks dominate.
- Liquidity dries up in off-the-run; switching costs spike.

##### Implementation Checklist + Verification Tests

- Confirm coupon and upfront conventions for each series; do not assume.
- Ensure consistent curve used to compute RPV01 in both series.

---

## 5. Strategy Family C: Correlation / Tranche RV Framing (Risk-First)

### Tranche PV Decomposition and Why Correlation Matters

#### PV Decomposition (Conceptual)

A tranche behaves like a credit derivative on the portfolio loss distribution: PV depends on expected discounted protection leg (driven by losses) and premium leg (driven by outstanding tranche notional).

#### Correlation/Dependence Affects Tail

Higher dependence increases probability of clustered defaults (tail), which redistributes expected losses across attachment points (equity vs mezz vs senior).

#### Base Correlation Language

The sources discuss base correlation as a widely used approach and warn about arbitrage issues (implied correlation skew, mapping).

---

### Strategy Card: C1 — Tranche RV / Correlation Exposure with Explicit Hedge Map

##### Objective (Conceptual; No Forecasts)

Compare tranches by which risk dominates:

- spread level (PV01),
- dependence/correlation (corr01),
- tail/default clustering scenarios,
- recovery uncertainty.

##### Instruments and Position Construction

- One tranche $[A, B]$ (e.g., equity 0–3%, mezz 3–7%, etc.).
- Potential hedges (conceptual map, only as supported):
  - index CDS to hedge broad spread level,
  - systemic delta hedging approach in the source (hedge tranche spread sensitivity to systemic spread moves).
  - adjacent tranches as spread hedges (PV01-neutralization).

##### Exposure Decomposition

| Exposure | Units | Meaning |
|----------|-------|---------|
| Tranche PV01 | USD/bp | linear tranche spread risk |
| Systemic delta | USD/bp | sensitivity to "systemic" spread move |
| Corr01 | USD per 1% corr | dependence sensitivity |
| Tail/clustering | scenario USD | nonlinear loss jump risk |
| Recovery sensitivity | USD/1% | recovery impacts losses and settlement |

##### Hedge Set and Hedge Ratios (Show Math)

**PV01 hedge with index:**

$$N_{\text{index}} = -\frac{\text{PV01}_{\text{tranche}}}{\text{CS01}_{\text{index}}}$$

**Adjacent tranche PV01 hedge:**

$$N_{\text{hedge tranche}} = -\frac{\text{PV01}_{\text{target tranche}}}{\text{PV01}_{\text{hedge tranche}}}$$

##### Scenario Test Suite

- Parallel spread move (small).
- Dispersion (single-name basket shocks).
- Default event / jump (one default).
- Correlation shock (dependence up/down).
- Tail/clustering shock (multiple defaults clustered; tranche loss nonlinear).

##### Failure Modes

- PV01 hedges fail under default clustering (loss-driven jump).
- Model risk: base correlation interpolation and mapping choices.
- Liquidity and unwind risk in tranches.

##### Implementation Checklist + Verification Tests

- Bump PV01 with $\pm 1$ bp; check symmetry.
- Corr01: revalue with $\rho \pm 1\%$; confirm stable sign/magnitude.
- Run discrete default scenarios: 1 default, 5 defaults, 10 defaults; compare hedge performance.

---

## 6. Math and Derivations (Step-by-Step)

### 6.1 CS01 for a Standard CDS from the MTM Identity

**Source-backed starting point:**

$$V(t,T) = (S(t,T) - S_0) \cdot \text{RPV01}(t,T) \quad \text{(per unit notional)}.$$

**Differentiate w.r.t. market spread $S$:**

$$\frac{\partial V}{\partial S} = \text{RPV01}(t,T).$$

**Therefore,**

$$\boxed{\text{CS01} = \frac{\partial PV}{\partial S} \times 10^{-4} = N \cdot \text{RPV01}(t,T) \cdot 10^{-4}}$$

**Unit check:**

$\text{RPV01}$ has units "years" (PV of 1 bp/yr premium). Multiply by $10^{-4}$ (per bp) and notional $N$ (USD) $\Rightarrow$ USD/bp.

---

### 6.2 CS01-Based Hedge Ratio Derivation

Let target position have $\text{CS01}_T$ and hedge instrument have $\text{CS01}_H$, both sign-inclusive (USD/bp).

We want:

$$\text{CS01}_{\text{net}} = \text{CS01}_T + N_H \cdot \text{CS01}_H^{(\$1)} = 0.$$

Solve:

$$\boxed{N_H = -\frac{\text{CS01}_T}{\text{CS01}_H^{(\$1)}}}$$

If CS01s are already in USD/bp for stated notionals:

$$N_H = -\frac{\text{CS01}_T}{\text{CS01}_H} \times N_H^{\text{current}}.$$

---

### 6.3 JTD (VOD) Mapping for CDS

**Source-backed VOD expression:**

$$\text{VOD} = (1 - R - \text{Accrued}) - (S - S_0) \cdot \text{RPV01}.$$

**Interpretation:**

If position is at par ($S = S_0$), then

$$\text{VOD} \approx 1 - R - \text{Accrued}.$$

**Dollar JTD:**

$$\boxed{\text{JTD} = N \cdot \text{VOD}}$$

**Unit check:**

$1 - R$ is unitless fraction of notional; accrued is fraction; multiply by $N$ gives USD.

---

### 6.4 P&L Explain Template (First-Order + Event Terms)

For a portfolio, approximate P&L over horizon as:

$$\boxed{\Delta PV \approx \sum_i (\text{CS01}_i \cdot \Delta S_i) + (\text{Rec01} \cdot \Delta R) + \text{JTD}_{\text{events}} + \text{BasisTerm} + \text{Residual}}$$

- **CS01 term:** linear spread moves (bucketed by maturity or instrument).
- **Rec01 term:** recovery assumption change.
- **JTD events:** discrete defaults and settlement mechanics.
- **BasisTerm:** quoted–intrinsic or CDS–cash basis change.
- **Residual:** convexity, model error, liquidity/execution slippage.

---

### 6.5 Carry/Rolldown Decomposition (Transparent, with Assumptions)

We want to separate:

- Accrual / coupon / premium flows (carry cashflows),
- Curve aging (rolldown) holding the spread curve fixed,
- Spread move component.

**A practical decomposition:**

Let $PV(t; S_t)$ be PV at time $t$ using today's spread curve $S_t(\cdot)$.

Define a "rolled" PV at $t + \Delta t$ holding the curve fixed but shortening maturity:

$$PV_{\text{roll}}(t + \Delta t; S_t).$$

Then:

$$\Delta PV = \underbrace{\text{Cashflows over } [t, t + \Delta t]}_{\text{carry}} + \underbrace{(PV_{\text{roll}}(t + \Delta t; S_t) - PV(t; S_t))}_{\text{rolldown/theta}} + \underbrace{(PV(t + \Delta t; S_{t + \Delta t}) - PV_{\text{roll}}(t + \Delta t; S_t))}_{\text{spread move}} + \text{events}.$$

**If you need an exact closed-form carry formula:** I'm not sure, because it depends on the contract quoting regime (par spread vs fixed coupon + upfront), day count, and whether you use clean vs full price. What is needed: the desk's carry definition and the index rulebook conventions.

---

## 7. Worked Examples (At Least 15 Numeric Examples; Fully Numeric)

**Reminder:** all examples use toy inputs; magnitudes are chosen to look plausible but are not market data.

---

### Example 1: Bond–CDS Basis Toy

**Bond DV01 + CS01 vs CDS CS01 + JTD; choose CDS notional to hedge JTD; show residual under basis move.**

#### Setup

- Bond face: $N_B = \$1{,}000{,}000$
- Bond full price: $P = 101$ (per 100) $\Rightarrow$ market value $= 1.01 \times 1{,}000{,}000 = \$1{,}010{,}000$
- Recovery assumption: $R = 40\% \Rightarrow$ recovery value $\$400{,}000$
- Bond default loss (JTD for long bond):

$$\text{JTD}_{\text{bond}} = -(P - R) \times N_B = -(1.01 - 0.40) \times 1{,}000{,}000 = -\$610{,}000.$$

(Loss-on-default mismatch is a key basis driver in the sources.)

#### CDS Leg: Choose Notional to Hedge JTD

Assume we buy protection at par: $S = S_0 = 150$ bp $\Rightarrow$ pre-default MTM $\approx 0$.

Accrued premium at default (assume halfway through quarter): $\Delta = 0.125$

$$\text{Accrued} = N_{\text{CDS}} \times (0.015) \times 0.125 = 0.001875 \, N_{\text{CDS}}.$$

JTD for long protection at par (using VOD logic):

$$\text{JTD}_{\text{CDS}} \approx N_{\text{CDS}} \times (1 - R) - \text{Accrued} = 0.60 N_{\text{CDS}} - 0.001875 N_{\text{CDS}} = 0.598125 \, N_{\text{CDS}}.$$

Solve $0.598125 N_{\text{CDS}} = \$610{,}000$:

$$N_{\text{CDS}} = \frac{610{,}000}{0.598125} = \$1{,}019{,}853.71 \approx \$1.02\text{mm}.$$

#### Spread Sensitivities

Assume CDS $\text{RPV01} = 4.5$. Then:

$$\text{CS01}_{\text{CDS}} = N_{\text{CDS}} \cdot \text{RPV01} \cdot 10^{-4} = 1{,}019{,}853.71 \times 4.5 \times 10^{-4} = \$458.93/\text{bp}.$$

Assume bond credit CS01 (toy): $\text{CS01}_{\text{bond}} = -\$420/\text{bp}$.

#### Basis-Move Scenario

CDS spread widens $+20$ bp; bond spread widens $+5$ bp (basis widens).

**P&L (linear):**

- Bond: $\Delta PV_B = -420 \times 5 = -\$2{,}100$.
- CDS: $\Delta PV_{\text{CDS}} = +458.93 \times 20 = +\$9{,}178.68$.
- **Net:** $+7{,}078.68$.

#### Interpretation

JTD was hedged by construction, but basis risk remains: differing spread moves generate residual P&L.

---

### Example 2: DV01-Neutral but Not Credit-Neutral

**Bond DV01 hedged with rates instrument, then credit spread widens; compute residual P&L.**

#### Setup

- Corporate bond notional: $N_C = \$10\text{mm}$
- Rates DV01 (sign-inclusive for +1bp rates move): $\text{DV01}_{r,C} = -\$7{,}500/\text{bp}$
- Treasury hedge instrument DV01 per $1mm (long): $-\$850/\text{bp}$
- If we short Treasuries, DV01 becomes $+\$850/\text{bp}$ per $1mm.

#### DV01 Hedge Ratio

Need $-7{,}500 + 850 \times N_T\text{ (\$1mm units)} = 0$

$$N_T = \frac{7{,}500}{850} = 8.8235 \text{ mm}.$$

So: **short $8.8235mm Treasuries.**

#### Scenario

- Rates +10 bp (parallel)
- Credit spread on corporate widens +50 bp
- Treasury credit spread unchanged (assume risk-free)

#### P&L

**Rates move:**
- Corporate: $-7{,}500 \times 10 = -\$75{,}000$
- Short Treasury: $+(850 \times 8.8235) \times 10 = +7{,}500 \times 10 = +\$75{,}000$
- **Net rates P&L $\approx 0$.**

**Credit move:**
- Assume corporate credit CS01 (toy): $\text{CS01}_{\text{corp}} = -\$6{,}500/\text{bp}$ (for $10mm).
- $\Delta PV_{\text{credit}} = -6{,}500 \times 50 = -\$325{,}000$

#### Conclusion

**DV01-neutral does not mean risk-neutral: credit spread risk dominates.**

---

### Example 3: Single-Name Hedged with Index by CS01 Matching

**Scenario: broad move vs idiosyncratic move; compute residual.**

#### Setup

- Single-name CDS: long protection, $N_s = \$10\text{mm}$, $\text{RPV01}_s = 4.5$

$$\text{CS01}_s = 10{,}000{,}000 \times 4.5 \times 10^{-4} = \$4{,}500/\text{bp}.$$

- Index CDS: use $\text{RPV01}_I = 4.2$ (toy). CS01 per $10mm long protection:

$$\text{CS01}_{I, \$10mm} = 10{,}000{,}000 \times 4.2 \times 10^{-4} = \$4{,}200/\text{bp}.$$

Hedge by short protection on index.

#### Hedge Notional

Solve $\text{CS01}_s + \text{CS01}_I^{\text{(short)}} = 0$:

$$4{,}500 - 4{,}200 \times \frac{N_I}{10\text{mm}} = 0 \Rightarrow N_I = 10\text{mm} \times \frac{4{,}500}{4{,}200} = \$10.7143\text{mm}.$$

#### Scenario A: Broad +10 bp Move (Both)

- Single-name: $+4{,}500 \times 10 = +\$45{,}000$
- Index short: $-4{,}200 \times 1.07143 \times 10 = -\$45{,}000$
- **Net $\approx 0$.**

#### Scenario B: Idiosyncratic Single-Name +50 bp, Index +10 bp

- Single-name: $+4{,}500 \times 50 = +\$225{,}000$
- Index short: $-45{,}000$
- **Net: $+\$180{,}000$** (residual idiosyncratic spread risk).

#### Event Note (JTD)

If the single name defaults, single-name JTD is large; index hedge only absorbs the small loss on that one name inside the index (diversification). **CS01 matching does not hedge JTD.**

---

### Example 4: Index Hedged with Constituents (Bottom-Up) Using RPV01/CS01 Weights

**Important:** The sources show intrinsic index valuation as a sum/average of constituent CDS values and discuss portfolio swap adjustment. A full "bottom-up hedge" method depends on how you allocate index basis and curve adjustments; details are desk-convention dependent.

**So:** I'm not sure there is a single canonical hedge-weight formula without specifying the portfolio swap adjustment rule. Below is a clearly labeled approximation.

#### Approximation Used

- Small index with $M = 3$ names.
- Index notional $N_I = \$30\text{mm} \Rightarrow \$10\text{mm}$ per name.
- Assume all maturities aligned at 5Y.

#### Constituent RPV01s (Toy)

- Name A: 4.5
- Name B: 4.7
- Name C: 4.6

#### Index CS01 (Approx)

Index RPV01 $\approx$ average $= 4.6$

$$\text{CS01}_I = 30{,}000{,}000 \times 4.6 \times 10^{-4} = \$13{,}800/\text{bp}.$$

#### Constituent Hedge (Short Protection on Names to Hedge Long Protection Index)

Short each name $10mm:

- A: $-10{,}000{,}000 \times 4.5 \times 10^{-4} = -\$4{,}500/\text{bp}$
- B: $-\$4{,}700/\text{bp}$
- C: $-\$4{,}600/\text{bp}$
- **Total:** $-\$13{,}800/\text{bp}$, which offsets index $+\$13{,}800/\text{bp}$.

#### Parallel +10 bp Scenario

- Index: $+13{,}800 \times 10 = +138{,}000$
- Hedge: $-138{,}000$
- **Net $\approx 0$.**

#### Dispersion Scenario

If one name widens much more than index (or index basis shifts), hedge breaks—this is exactly why index basis matters.

---

### Example 5: Index Intrinsic vs Quoted Basis

**Compute intrinsic from constituents, basis = quoted − intrinsic; show a basis P&L scenario.**

#### Setup

- $M = 3$, index notional $N_I = \$10\text{mm}$, coupon $C = 100$ bp.
- Constituent 5Y spreads (bp): $S_1 = 120$, $S_2 = 90$, $S_3 = 110$.
- Constituent RPV01s: $4.5, 4.7, 4.6$.
- Quoted index spread: $S_I = 110$ bp.
- Index RPV01: $\text{RPV01}_I = 4.6$.

#### Compute Intrinsic Spread (RPV01-Weighted Average)

$$S_{\text{intr}} = \frac{\sum_m \text{RPV01}_m S_m}{\sum_m \text{RPV01}_m} = \frac{4.5(120) + 4.7(90) + 4.6(110)}{4.5 + 4.7 + 4.6} = 106.4493 \text{ bp}.$$

#### Index Basis (Spread)

$$\text{Basis} = S_I - S_{\text{intr}} = 110 - 106.4493 = 3.5507 \text{ bp}.$$

#### PV View (Upfront Difference)

**Market upfront value at coupon $C$:**

$$U_{\text{mkt}} = N_I \cdot \text{RPV01}_I \cdot (S_I - C) \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.6 \cdot (10) \cdot 10^{-4} = \$46{,}000.$$

**Intrinsic upfront using constituents** (equal-weight notional $N_I/M = 3.3333\text{mm}$):

- Name 1: $(120 - 100) = 20$ bp
  - $PV_1 = 3.3333\text{mm} \cdot 4.5 \cdot 20 \cdot 10^{-4} = \$30{,}000$
- Name 2: $(90 - 100) = -10$ bp
  - $PV_2 = 3.3333\text{mm} \cdot 4.7 \cdot (-10) \cdot 10^{-4} = -\$15{,}666.67$
- Name 3: $(110 - 100) = 10$ bp
  - $PV_3 = 3.3333\text{mm} \cdot 4.6 \cdot 10 \cdot 10^{-4} = \$15{,}333.33$
- **Sum** $V_{\text{intr}} = \$29{,}666.67$

#### Basis PV

$$U_{\text{mkt}} - V_{\text{intr}} = 46{,}000 - 29{,}666.67 = \$16{,}333.33,$$

which also equals:

$$N_I \cdot \text{RPV01}_I \cdot \text{Basis} \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.6 \cdot 3.5507 \cdot 10^{-4} = \$16{,}333.33.$$

#### Basis P&L Scenario

Suppose basis decreases by 2 bp due to index tightening (constituents unchanged):

- Index $\Delta S_I = -2$ bp.
- Index CS01: $10{,}000{,}000 \times 4.6 \times 10^{-4} = \$4{,}600/\text{bp}$.
- A position long index protection has P&L $\Delta PV = 4{,}600 \times (-2) = -\$9{,}200$.
- The constituent legs (unchanged) contribute $\approx 0$.

**Interpretation:** basis moves can dominate short-horizon hedges.

---

### Example 6: Index Carry

**Compute one quarter's premium/coupon cashflow and upfront amortization effect (if applicable) and show carry under unchanged spreads.**

#### Setup (Long Protection on Index)

- Notional $N = \$10\text{mm}$
- Coupon $C = 100$ bp
- Market spread at inception $S = 120$ bp
- $\text{RPV01}(t_0) = 4.6$, $\text{RPV01}(t_1) = 4.4$ after one quarter
- Quarter fraction $\Delta = 0.25$

#### Upfront at Inception (Paid by Protection Buyer Because $S > C$)

$$U_0 = N \cdot \text{RPV01}(t_0) \cdot (S - C) \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.6 \cdot 20 \cdot 10^{-4} = \$92{,}000.$$

#### Coupon Paid over the Quarter (Carry Cashflow)

$$\text{Coupon CF} = -N \cdot C \cdot 10^{-4} \cdot \Delta = -10{,}000{,}000 \cdot 100 \cdot 10^{-4} \cdot 0.25 = -\$25{,}000.$$

#### Unchanged Spreads: MTM "Amortization" Effect

At $t_1$, with spreads unchanged at $S = 120$ bp, the contract value at coupon $C$ is:

$$V(t_1) = N \cdot \text{RPV01}(t_1) \cdot (S - C) \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.4 \cdot 20 \cdot 10^{-4} = \$88{,}000.$$

Since we paid $U_0 = 92{,}000$ upfront at inception, the clean value relative to inception is:

$$\Delta V_{\text{clean}} = 88{,}000 - 92{,}000 = -\$4{,}000.$$

#### Carry under Unchanged Spreads (Quarter)

$$\text{Carry P\&L} \approx \text{Coupon CF} + \Delta V_{\text{clean}} = -25{,}000 - 4{,}000 = -\$29{,}000.$$

**Interpretation:** even with unchanged spreads, the upfront-related value decays as maturity shortens (RPV01 falls).

---

### Example 7: Index Rolldown

**Hold curve fixed, move forward one month, revalue (toy) and compute rolldown P&L.**

#### Setup (Continue Long Protection Example Style)

- Notional $N = \$10\text{mm}$, coupon $C = 100$ bp
- Upfront paid at inception (from Example 6): $U_0 = 92{,}000$
- Assume spread curve is upward sloping:
  - 5Y par spread at $t_0$: 120 bp
  - After 1 month, remaining maturity $\approx 4.92$Y has par spread 118 bp (curve held fixed, you "roll down")
- $\text{RPV01}(t_1) = 4.57$ (toy, slightly lower due to shorter maturity)
- Monthly coupon accrual (approx): $\Delta = 1/12$

#### MTM at $t_1$ Holding Curve Fixed

$$V(t_1) = N \cdot 4.57 \cdot (118 - 100) \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.57 \cdot 18 \cdot 10^{-4} = \$82{,}260.$$

#### Clean Value Relative to Inception

$$\Delta V_{\text{clean}} = 82{,}260 - 92{,}000 = -\$9{,}740.$$

#### Coupon Accrual over 1 Month

$$\text{Coupon accrual} = -N \cdot 100 \cdot 10^{-4} \cdot \frac{1}{12} = -10{,}000{,}000 \cdot 0.01 \cdot 0.083333 = -\$8{,}333.33.$$

#### Rolldown + Carry over Month (No Curve Change Beyond Aging)

$$\text{P\&L} \approx -8{,}333.33 - 9{,}740 = -\$18{,}073.33.$$

---

### Example 8: Roll Trade Mechanics

**Old series vs new series upfront difference; compute cash exchanged and execution-cost effect (toy).**

#### Setup

- Notional $N = \$10\text{mm}$, coupon $C = 100$ bp.
- Old series: $S_{\text{old}} = 115$ bp, $\text{RPV01}_{\text{old}} = 4.1$
- New series: $S_{\text{new}} = 120$ bp, $\text{RPV01}_{\text{new}} = 4.6$
- Position: long protection.

#### MTM Values (Pre-Upfront)

$$V_{\text{old}} = N \cdot 4.1 \cdot (115 - 100) \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.1 \cdot 15 \cdot 10^{-4} = \$61{,}500.$$

$$V_{\text{new}} = N \cdot 4.6 \cdot (120 - 100) \cdot 10^{-4} = \$92{,}000.$$

#### Cash to Switch (Close Old, Open New)

- Close old (sell your long protection): receive $61,500.
- Open new: pay $92,000.
- **Net cash outflow:**

$$92{,}000 - 61{,}500 = \$30{,}500.$$

#### Execution Cost (Toy)

Suppose bid/offer equivalent to 0.5 bp spread on each leg.

- Approx PV cost per leg $\approx \text{CS01} \times 0.5$.
- Use $\text{CS01}_{\text{new}} \approx 10{,}000{,}000 \times 4.6 \times 10^{-4} = 4{,}600$ USD/bp
- Cost per leg $\approx 4{,}600 \times 0.5 = \$2{,}300$
- Two legs $\Rightarrow$ total $\approx \$4{,}600$

#### Total "All-In" Cash Impact

$$30{,}500 + 4{,}600 = \$35{,}100.$$

---

### Example 9: Default in Index

**Given final price FP, compute default settlement impact on index cashflows.**

#### Source-Backed Mechanics

Default payoff based on face value minus final price; settlement can be physical or cash with auction determining final price.

For index: a default causes a payout on that name and reduces future premium by a proportional notional reduction (illustrated as $1/M$ type reduction in the sources).

#### Toy Setup

- Total index notional: $N = \$10\text{mm}$
- $M = 5 \Rightarrow$ notional per name $N_m = 2\text{mm}$
- Coupon $C = 100$ bp
- Final price $FP = 35\% \Rightarrow R = 0.35$
- Accrual since last coupon: 50 days on Actual/360:

$$\Delta = \frac{50}{360} = 0.1388889.$$

#### Default Settlement (Long Protection)

**Protection payment:**

$$(1 - R) N_m = (1 - 0.35) \times 2{,}000{,}000 = 0.65 \times 2{,}000{,}000 = \$1{,}300{,}000.$$

**Accrued premium owed by protection buyer on that name:**

$$N_m \cdot C \cdot 10^{-4} \cdot \Delta = 2{,}000{,}000 \cdot 100 \cdot 10^{-4} \cdot 0.1388889 = 2{,}000{,}000 \cdot 0.01 \cdot 0.1388889 = \$2{,}777.78.$$

**Net immediate cash:**

$$1{,}300{,}000 - 2{,}777.78 = \$1{,}297{,}222.22.$$

#### Future Coupon Impact

**Before default:** quarterly coupon on $10mm:

$$10{,}000{,}000 \cdot 0.01 \cdot 0.25 = \$25{,}000.$$

**After default,** notional reduced by $1/M = 20\% \Rightarrow$ remaining $= 8\text{mm}$:

$$8{,}000{,}000 \cdot 0.01 \cdot 0.25 = \$20{,}000.$$

Coupon reduces by $5,000 per quarter from then on (toy).

---

### Example 10: Full P&L Explain for an Index Position over a Horizon

**Carry + rolldown + spread move + default event.**

#### Setup

- Long protection on index
- Notional $N = \$10\text{mm}$, coupon $C = 100$ bp
- $\text{RPV01} = 4.6 \Rightarrow \text{CS01} = 10{,}000{,}000 \times 4.6 \times 10^{-4} = \$4{,}600/\text{bp}$
- One-month horizon; use Example 7's carry+rolldown under fixed curve: $-\$18{,}073.33$

#### Observed Changes (Toy)

**Index spread widens $+10$ bp** $\Rightarrow$ spread P&L:

$$\Delta PV_{\text{spread}} = 4{,}600 \times 10 = +\$46{,}000.$$

**One constituent defaults** in a large index $M = 125$ (so per-name notional small):

- $N_m = 10{,}000{,}000 / 125 = \$80{,}000$
- $FP = 35\% \Rightarrow 1 - R = 0.65$
- Accrual: 50/360
- Default cash:

$$0.65 \cdot 80{,}000 - 80{,}000 \cdot 0.01 \cdot 0.1388889 = 52{,}000 - 111.11 = \$51{,}888.89.$$

#### P&L Explain

$$\Delta PV \approx \underbrace{(-18{,}073.33)}_{\text{carry+rolldown}} + \underbrace{46{,}000}_{\text{spread move}} + \underbrace{51{,}888.89}_{\text{default event}} = \$79{,}815.56.$$

#### Interpretation

Linear spread risk and discrete event risk both matter; carry/rolldown is smaller but non-negligible over time.

---

### Example 11: Tranche PV01

**Bump tranche spread ±1bp and compute PV01 by finite difference.**

#### Setup

- Tranche position notional $N_T = \$10\text{mm}$
- Tranche $\text{RPV01}_T = 3.2$ (toy)
- At-par tranche: contractual spread = market spread, so PV $\approx 0$.

#### Revalue for ±1 bp

**If spread widens by +1 bp:**

$$PV(+1) = N_T \cdot 3.2 \cdot 1 \cdot 10^{-4} = 10{,}000{,}000 \cdot 3.2 \cdot 10^{-4} = \$3{,}200.$$

**If spread tightens by −1 bp:**

$$PV(-1) = -\$3{,}200.$$

#### Finite Difference PV01

$$\text{PV01} = \frac{PV(+1) - PV(-1)}{2} = \frac{3{,}200 - (-3{,}200)}{2} = \$3{,}200/\text{bp}.$$

---

### Example 12: Correlation Shock Toy

**PV(tranche) under low vs high dependence; compute PV change and interpret "correlation risk."**

#### Setup

- Same tranche notional $N_T = \$10\text{mm}$
- Model-reported PVs (toy) for a given tranche:
  - $PV(\rho = 20\%) = -\$400{,}000$
  - $PV(\rho = 30\%) = -\$350{,}000$

#### Correlation P&L for +10% Correlation

$$\Delta PV = -350{,}000 - (-400{,}000) = +\$50{,}000.$$

#### Correlation 01 (Per 1% Correlation)

$$\text{Corr01} \approx \frac{50{,}000}{10} = \$5{,}000 \text{ per } 1\%.$$

#### Interpretation

This tranche benefits (PV increases) when dependence increases. The source emphasizes that corr sensitivity varies by tranche and is measured by Corr01.

---

### Example 13: Tail/Clustering Scenario

**1 default vs 5 clustered defaults; compute tranche loss and PV impact; show why PV01 hedges fail.**

#### Setup

- Portfolio has $M = 100$ equal names; each default has recovery $R = 40\% \Rightarrow$ loss per default on portfolio notional:

$$\ell = \frac{1 - R}{M} = \frac{0.60}{100} = 0.006 = 0.6\%.$$

- Consider equity tranche $[A, B] = [0\%, 3\%] \Rightarrow B - A = 3\%$.
- Tranche notional (toy) $N_T = \$10\text{mm}$.

#### Scenario 1: 1 Default

Portfolio loss $L = 0.6\%$.

Tranche loss fraction:

$$L_{[0,3]} = \frac{\min(L, 3\%)}{3\%} = \frac{0.6\%}{3\%} = 0.20.$$

Dollar loss on tranche (for protection seller):

$$0.20 \times 10{,}000{,}000 = \$2{,}000{,}000.$$

#### Scenario 2: 5 Defaults (Cluster)

Portfolio loss $L = 5 \times 0.6\% = 3.0\%$.

Tranche loss fraction:

$$L_{[0,3]} = \frac{3.0\%}{3\%} = 1.00.$$

Dollar loss:

$$1.00 \times 10{,}000{,}000 = \$10{,}000{,}000.$$

#### Why PV01 Hedges Fail

A PV01 hedge is calibrated to small spread moves (linear).

A clustered default scenario creates a jump in realized loss that is not proportional to a 1–5 bp spread bump.

---

### Example 14: Adjacent Tranche Hedge

**Hedge tranche spread PV01 with another tranche; validate under small spread move; show failure under clustering.**

#### Setup

- **Target tranche:** equity $[0, 3]$, notional $N_E = \$10\text{mm}$, $\text{PV01}_E = \$3{,}200/\text{bp}$ (from Example 11).
- **Hedge tranche:** mezz $[3, 7]$, assume $\text{PV01}_M = \$1{,}800/\text{bp}$ per $10mm notional (toy).
- **Goal:** PV01-neutral hedge of spread moves.

#### Hedge Ratio

Need $\text{PV01}_E + \text{PV01}_M \times (N_M / 10\text{mm}) \times \text{sign} = 0$.

If we take opposite spread exposure using mezz, the notional ratio is:

$$\frac{N_M}{10\text{mm}} = \frac{\text{PV01}_E}{\text{PV01}_M} = \frac{3{,}200}{1{,}800} = 1.7778 \Rightarrow N_M = \$17.778\text{mm}.$$

#### Check under Small +1 bp Spread Move

- Equity spread P&L: $+3{,}200$ (long protection) or $-3{,}200$ (short protection); choose opposite sign for hedge so it offsets.
- Mezz spread P&L magnitude: $1{,}800 \times 1.7778 = 3{,}200$.
- **Net $\approx 0$. ✅**

#### Failure under Clustering

Use the same portfolio assumptions as Example 13.

**Clustered defaults: 10 defaults** $\Rightarrow L = 10 \times 0.6\% = 6\%$.

**Equity tranche $[0, 3]$ loss fraction** $= 1.00 \Rightarrow$ $10mm loss.

**Mezz tranche $[3, 7]$ loss fraction:**

$$L_{[3,7]} = \frac{\min(\max(6\% - 3\%, 0), 4\%)}{4\%} = \frac{3\%}{4\%} = 0.75.$$

Dollar loss on mezz notional $N_M = 17.778\text{mm}$ is $0.75 \times 17.778\text{mm} = \$13.333\text{mm}$.

#### Interpretation

The PV01-neutral hedge can become over-hedged or under-hedged in default clusters because losses are nonlinear and tranche-dependent.

---

### Example 15: Strategy Comparison Table

**For 3 strategies (bond–CDS basis, single-name–index, tranche RV), present exposures side-by-side and map hedges.**

#### Toy Exposure Snapshot (Illustrative)

| Strategy | Primary instrument(s) | CS01 (USD/bp) | JTD (USD) | Rec01 (USD/1%) | Basis sensitivity | Typical hedge map |
|----------|----------------------|---------------|-----------|----------------|-------------------|-------------------|
| Bond–CDS basis (A1) | bond + CDS | bond: $-420$; CDS: $+459$ | bond: $-610k$; CDS: $+610k$ | meaningful | CDS–cash basis | DV01 hedge (rates), CS01 or JTD hedge (CDS notional) |
| Single-name vs index (A2) | single CDS + index CDS | single: $+4{,}500$; index short: $-4{,}500$ | single default dominates | meaningful | proxy/index basis + roll | CS01 match; event stress for residual JTD |
| Tranche RV (C1) | tranche + hedge(s) | tranche PV01: $3{,}200$ | jump via losses | recovery + corr | dependence/tail | PV01 hedge (index/adjacent), Corr01 monitoring, cluster stress |

---

## 8. Practical Notes

### No Trade Tips Disclaimer (Educational Only)

This chapter is an educational risk framework. It does not provide recommendations, forecasts, or "what to trade now."

### Common Pitfalls

- **Mixing CS01 definitions:**
  - quote bump vs hazard-rate bump; index spread bump vs constituent bump.
  - Mixing quoting regimes (running spread vs fixed coupon + upfront) inconsistently.
- **Ignoring recovery/final-price risk** in event scenarios (default payoff depends on recovery/final price).
- **Assuming roll calendars/coupons** without sourcing (index conventions are product- and rulebook-specific).
- **Confusing hedging** (risk reduction) **with RV exposure** (basis risk intentionally retained).

### Implementation Pitfalls

- **Unit mistakes:** bp vs decimals; per-100 price vs per-$ notional; inconsistent scaling.
- **Inconsistent curves and recovery assumptions** between legs.
- **Execution costs** dominate small basis edges.

### Verification Tests

- **Scaling with notional:** CS01 and PV should scale linearly for small bumps.
- **Repricing checks** for all legs under base scenario.
- **Scenario suite passes:**
  - parallel / dispersion / event / roll / tail.

---

## 9. Summary & Recall

### 10-Bullet Executive Summary

1. A credit "strategy" must be written as **exposures + hedges + failure modes** (not a slogan).
2. **CS01 is linear spread risk**; specify exactly what spread is bumped.
3. **JTD/VOD captures default discontinuity**; CS01 hedges do not neutralize default jumps.
4. **Recovery/final price** is a key state variable; mismatches drive hedge surprises.
5. **Bond–CDS basis persists** due to funding, delivery option, technical default, loss-on-default mismatch, and liquidity.
6. **Indices have upfront + coupon mechanics**; defaults reduce notional and future coupon payments.
7. **Index quoted vs intrinsic basis** matters for hedging vs constituents.
8. **Carry/rolldown** requires separating cashflows, curve aging, and spread moves.
9. **Tranche risk** is dominated by tail/clustering and correlation sensitivity (Corr01 varies by tranche).
10. **Always run a scenario suite:** parallel, dispersion, default, roll basis, and tail shocks.

### Cheat Sheet

**Strategy card:** Objective → Instruments → Exposures table → Hedge ratios → Scenario suite → Failure modes → Checks.

**Key formulas:**

- CDS MTM: $V = (S - S_0) \cdot \text{RPV01}$.
- CS01 hedge: $N_H = -\text{CS01}_T / \text{CS01}_H$.
- VOD/JTD (CDS): $\text{VOD} = (1 - R - \text{Accrued}) - (S - S_0) \cdot \text{RPV01}$.
- Tranche loss: $L_{[A,B]} = \frac{1}{B - A} \left( \max(L - A, 0) - \max(L - B, 0) \right)$.

---

### 30 Flashcards (Q/A)

1. **Q:** What does CS01 measure? **A:** PV change for a +1 bp change in the specified quoted spread.

2. **Q:** For long protection CDS, is CS01 positive or negative? **A:** Positive (PV rises when spreads widen).

3. **Q:** What is RPV01? **A:** PV of 1 bp/yr of premium leg per unit notional (risky PV01).

4. **Q:** Give the CDS MTM identity used for CS01. **A:** $V = (S - S_0) \cdot \text{RPV01}$.

5. **Q:** Define JTD in these notes. **A:** $PV_{\text{post-default}} - PV_{\text{pre-default}}$.

6. **Q:** What is VOD? **A:** Value-on-default; the PV jump at default for CDS (used as JTD measure).

7. **Q:** Why can bond–CDS basis persist? **A:** Funding, delivery option, technical default, loss-on-default mismatch, accrued-at-default differences, liquidity.

8. **Q:** What is "loss on default" mismatch? **A:** CDS loss is $1 - R$; bond loss is $P - R$ if bond price $P \neq 100$.

9. **Q:** What is index "intrinsic value"? **A:** Index value as sum/average of constituent CDS values at index coupon.

10. **Q:** What is index "quoted vs intrinsic" basis? **A:** Difference between quoted index spread and intrinsic implied spread.

11. **Q:** What is portfolio swap adjustment? **A:** Adjusting constituent curves to reconcile intrinsic with quoted index (rule is somewhat arbitrary).

12. **Q:** What happens to index notional after a constituent default (simplified)? **A:** Reduced proportionally (e.g., by $1/M$ per default).

13. **Q:** What is carry for credit indices in this chapter? **A:** Coupon/premium accrual cashflow component.

14. **Q:** What is rolldown? **A:** PV change from aging along a sloped spread curve with curve held fixed.

15. **Q:** What is theta in CDS risk terms? **A:** Change in value from one day passing holding inputs fixed.

16. **Q:** Why do PV01 hedges fail in default clusters? **A:** Losses are nonlinear; PV01 is linear small-move risk.

17. **Q:** Define tranche PV01. **A:** PV change for 1 bp change in tranche spread (often equals tranche RPV01 × notional × 1e-4).

18. **Q:** Define Corr01. **A:** PV change for a 1% increase in the correlation/dependence parameter.

19. **Q:** What is the tranche loss function $L_{[A,B]}$ used for? **A:** Maps portfolio loss to tranche loss fraction.

20. **Q:** What does "dispersion" mean in these notes? **A:** Idiosyncratic moves across names vs index/systemic move.

21. **Q:** Why can index hedging a single name leave large residual risk? **A:** Single-name default jump dwarfs diversified index default impact.

22. **Q:** What is clean MTM for CDS? **A:** Full MTM minus accrued premium.

23. **Q:** Sign of accrued premium for long protection CDS? **A:** Negative (you owe accrued when unwinding).

24. **Q:** What is the key unit check for CS01? **A:** USD/bp.

25. **Q:** What scenario suite is mandatory for strategy validation? **A:** Parallel, dispersion, default, roll basis, tail/correlation.

26. **Q:** What is series/roll basis? **A:** Price/spread difference between index series due to roll, maturity reset, liquidity, composition.

27. **Q:** Why does recovery matter beyond spreads? **A:** Default settlement payoff depends on recovery/final price.

28. **Q:** What is a key risk management lesson about "perfect storm" events? **A:** Multiple adverse moves can coincide; stress testing is essential.

29. **Q:** What is the simplest CS01 hedge ratio? **A:** $N_H = -\text{CS01}_T / \text{CS01}_H$.

30. **Q:** What is the core discipline of this chapter? **A:** Define exposures, hedge explicitly, then enumerate failure modes.

---

## 10. Mini Problem Set (20 Questions)

**Questions 1–10 include brief solution sketches.**

---

**1.** A 5Y CDS has $\text{RPV01} = 4.2$. Compute CS01 per $1mm notional for long protection.

> **Sketch:** $\text{CS01} = 1{,}000{,}000 \times 4.2 \times 10^{-4} = \$420/\text{bp}$.

---

**2.** A long bond has rates DV01 $-\$6{,}000/\text{bp}$. A hedging Treasury has DV01 $-\$800/\text{bp}$ per $1mm (long). What Treasury notional (short) DV01-hedges the bond?

> **Sketch:** Short notional $N$ gives $+800N$. Solve $-6000 + 800N = 0 \Rightarrow N = 7.5\text{mm}$.

---

**3.** Using VOD at par $\approx 1 - R - \text{Accrued}$, compute JTD for long protection with notional $5mm, $R = 40\%$, accrued premium $6,000.

> **Sketch:** $0.60 \times 5{,}000{,}000 - 6{,}000 = 2{,}994{,}000$.

---

**4.** A single-name has CS01 $+3{,}800/\text{bp}$. An index has CS01 $+4{,}200/\text{bp}$ per $10mm. What index notional (short protection) CS01-hedges the single name?

> **Sketch:** Need $-4{,}200(N/10) + 3{,}800 = 0 \Rightarrow N = 9.0476\text{mm}$.

---

**5.** Compute tranche loss fraction for $[A, B] = [3\%, 7\%]$ when portfolio loss $L = 5\%$.

> **Sketch:** Loss in tranche $= \min(\max(5 - 3, 0), 4) / 4 = 2/4 = 0.5$.

---

**6.** A tranche PV at $\rho = 15\%$ is $-200k$ and at $\rho = 20\%$ is $-230k$. Compute Corr01.

> **Sketch:** $\Delta PV = -30k$ for +5% corr $\Rightarrow$ Corr01 $\approx -6k$ per 1%.

---

**7.** An index default on one name pays $(1 - R) N_m$. If $N = 25\text{mm}$, $M = 125$, $R = 35\%$, compute payout ignoring accrued.

> **Sketch:** $N_m = 25/125 = 0.2\text{mm}$; payout $= 0.65 \times 200k = 130k$.

---

**8.** You have a bond–CDS package with bond CS01 $-500/\text{bp}$ and CDS CS01 $+450/\text{bp}$ per $1mm. What CDS notional makes CS01 net zero?

> **Sketch:** Need $+450N - 500 = 0 \Rightarrow N = 1.1111\text{mm}$.

---

**9.** For an index long protection position, coupon $C = 100$ bp, notional $10mm, month fraction 1/12. Compute coupon accrual cashflow sign and magnitude.

> **Sketch:** Long protection pays premium: $-10\text{mm} \cdot 0.01 \cdot 1/12 = -\$8{,}333.33$.

---

**10.** A PV01-neutral adjacent tranche hedge uses notional ratio $N_H = \text{PV01}_T / \text{PV01}_H$. If PV01s are 2,400 and 1,600 (both per $10mm), compute hedge notional for a $20mm target.

> **Sketch:** Ratio $= 2400/1600 = 1.5$. For $20mm target (=2×$10mm), hedge notional $= 2 \times 1.5 \times 10\text{mm} = 30\text{mm}$.

---

**11.** Explain why "portfolio swap adjustment" is needed when hedging index vs constituents if index basis is nonzero.

---

**12.** Describe a scenario where DV01-neutral hedging worsens P&L due to curve twist.

---

**13.** For a bond priced at 105 with recovery 40, compare loss-on-default to CDS loss-on-default and discuss basis implication.

---

**14.** Construct a scenario suite for a single-name vs index proxy hedge and identify the worst-case scenario.

---

**15.** Give a reason why accrued premium conventions matter when unwinding CDS positions.

---

**16.** Design a P&L explain for a tranche over a month including spread move and one default event.

---

**17.** Show how CS01 bucket hedging can fail if the spread curve steepens rather than shifting in parallel.

---

**18.** Discuss how liquidity can create persistent basis and how you would reflect it in scenario analysis.

---

**19.** For an equity tranche, compare 1 default vs 5 defaults and compute the difference in tranche loss fraction.

---

**20.** Explain why correlation risk is not hedgeable by a single CS01 hedge and requires scenario testing.

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

- CDS MTM identity, RPV01, CS01 definitions: O'Kane Ch 6–8
- Index upfront mechanics, roll mechanics, intrinsic vs quoted basis: O'Kane Ch 9–10
- Tranche loss function, correlation sensitivity, base correlation: O'Kane Ch 11–14
- Bond–CDS basis drivers (funding, delivery option, etc.): O'Kane Ch 4–5
- VOD/JTD framework: O'Kane Ch 8
- Carry/theta definitions for tranches: O'Kane Ch 12

### (B) Reasoned Inference — Note Derivation Logic

- CS01 hedge ratio derivation: algebraic from CS01 definition
- P&L explain template: summation of first-order sensitivities plus events
- Tranche loss fraction computation: direct application of tranche loss formula

### (C) Speculation — Flag Uncertainties

- Exact closed-form carry formula for indices: depends on desk conventions and rulebook; flagged as "I'm not sure"
- Portfolio swap adjustment details: desk-convention dependent; flagged as approximation
