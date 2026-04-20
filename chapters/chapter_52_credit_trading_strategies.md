# Chapter 52: Credit Trading Strategies (Risk + Instrument + Hedge)

---

## Introduction

A credit portfolio manager looks at her screen one morning and sees an unfamiliar P&L: her "fully hedged" bond-CDS basis position has just lost USD 2 million overnight. The CDS leg is up, the bond leg is flat, and she cannot explain the discrepancy. Her CS01 was matched—she ran the numbers twice—yet the position moved against her. What went wrong?

The answer is a truth that separates credit traders from speculators: **a "hedged" position is not a risk-free position**. Every credit strategy carries residual exposures that survive any hedge you construct. The professional question is not whether residuals exist, but whether you *know* what they are and have tested them against scenarios that matter.

### The Strategy Card Discipline

This chapter teaches one core discipline: write every strategy as a **Strategy Card** with three parts.

1. **Exposure decomposition** — what you actually own (CS01 by bucket, JTD, recovery sensitivity, basis sensitivity, rates DV01).
2. **Hedge set + ratios** — what you neutralize, with the hedge math shown.
3. **Failure modes + scenario suite** — where the hedge breaks (parallel moves, dispersion, default events, roll/series basis, correlation/tail, funding stress).

If you cannot fill in all three parts, you do not have a strategy — you have a slogan.

### What This Chapter Covers

The framework applies across five strategy families:

| Section | Family | Key trades |
|---------|--------|------------|
| 52.4 | Basis | Bond-CDS packages; single-name vs index; CDS curve steepeners/flatteners |
| 52.5 | Carry / rolldown | Index holding-period P&L decomposition; series roll mechanics |
| 52.6 | Correlation / tranche RV | Tranche PV01; correlation exposure; tail/clustering scenarios |
| 52.7 | Capital structure | Senior vs sub CDS; equity-CDS RV; LBO trades |
| 52.9–52.10 | Crisis / execution | Stress behavior; position sizing; unwind discipline |

This chapter focuses on *implementation*: how to structure positions, size hedges, and stress-test against failure scenarios. Earlier chapters cover the underlying instruments — CDS mechanics (Chapter 38), index structures (Chapters 45–47), tranche pricing (Chapters 48–50), and CDS risk measures (Chapter 43). Chapter 44 developed analytical frameworks for CDS relative value; this chapter is about how *combinations* of those instruments interact, offset, and fail to offset under stress.

The recurring theme is the **"perfect storm"** — multiple adverse moves coinciding. That is precisely when hedges are most likely to fail and when understanding residual risk pays off. The Strategy Card framework is built to make those failure modes visible *before* they show up in your P&L.

### How to Use This Chapter

This chapter mixes instruments, risk measures, and execution realities. If you are new to credit, use the roadmap below rather than reading straight through.

**Prerequisites by topic:**

| If this is new | Read first |
|----------------|------------|
| CDS basics | Chapter 38 (mechanics); Chapters 39–40 (credit events, auctions) |
| Risk measures (CS01, RPV01, JTD, theta) | Chapter 43 |
| CDS indices | Chapters 45–47 |
| Tranches and correlation | Chapters 49–51 |

**Recommended first-pass route:**

1. **Section 52.2** — risk-measure vocabulary
2. **Section 52.3** — the Strategy Card discipline
3. **Examples 1–4** — basis packages, proxy hedges, curve trades
4. **Examples 6–10** — index carry, rolldown, roll, and a desk-style P&L explain
5. **Section 52.6** — tranches (return here once the earlier pieces feel natural)

The single goal to keep in mind: when P&L surprises you, you want a short, checkable list of residual risks to investigate — funding, liquidity, recovery/final price, basis, correlation/tail. The Strategy Card gives you that list before the surprise happens.

---

## Conventions & Notation

All numbers are educational toy examples (no real market data; no trade recommendations).

### Units and Scaling

| Item | Convention |
|------|------------|
| Currency | USD throughout |
| Single-name CDS / cash bond sensitivities | Per USD 1mm notional (unless stated) |
| Index sensitivities | Per USD 10mm index notional (unless stated) |
| Basis points | $1 \text{ bp} = 10^{-4}$ per annum |
| Spread move example | $\Delta S = +10 \text{ bp}$ means $+0.0010$ in decimal |

### Coupon and Day Count

- $C$ denotes the contractual running coupon (bp/yr).
- Index coupons are paid quarterly using an Actual/360 year fraction $\Delta$, consistent with the cited index mechanics.

### Clean vs Full (Dirty) MTM for CDS

$$\text{Clean MTM} = \text{Full MTM} - \text{Accrued}$$

**Accrued premium sign convention** (between coupon dates):
- Short protection: accrued is **positive** (short receives premium).
- Long protection: accrued is **negative** (long pays premium).

**Desk note:** clean MTM jumps around coupon dates; the cash coupon/accrual flows offset this in the "full" P&L. When a hedge looks right in risk but wrong in P&L, the clean-vs-full distinction is one of the first checks.

### CS01 Sign Convention

$$CS01 = \frac{\partial PV}{\partial S} \times (1 \text{ bp})$$

where $S$ is the quoted/par spread being bumped — single-name CDS spread, index spread, or tranche spread (always stated explicitly).

| Position | Sign of CS01 | Reason |
|----------|--------------|--------|
| Long protection CDS | CS01 $\gt 0$ | PV rises when $S$ widens |
| Short protection CDS | CS01 $\lt 0$ | mirror of above |
| Long cash bond | CS01 $\lt 0$ | PV falls when credit spread widens |

### JTD Sign Convention

JTD is the instantaneous PV jump at default — a scenario jump, not an infinitesimal derivative:

$$JTD \equiv PV_{\text{post-default}} - PV_{\text{pre-default}}$$

For long protection CDS, JTD is typically positive (you receive protection payment net of accrued premium). This is the **value-on-default (VOD)** framework used in the sources.

---

## 52.1 Conventions and Setup

### Single-Name CDS Valuation Identity (The Anchor)

Throughout this chapter, the mark-to-market of a standard running-spread CDS is written (per unit notional) as:

$$V(t,T) = (S(t,T) - S_0) \cdot RPV01(t,T)$$

where $RPV01$ is the risky PV01 of the premium leg. This identity is the anchor for nearly every risk measure that follows — CS01, hedge ratios, and JTD all derive from it.

### Index Contract Mechanics

- Index coupon $C$ is set near fair value (multiple of 5 bp), paid quarterly Actual/360.
- An upfront payment $U_I$ is exchanged at settlement.
- Index rolls every six months.
- **Default handling:** on a constituent credit event, the protection seller pays loss on the defaulted name and the index notional is reduced proportionally (simplified as $1/M$ per default in these notes).

### Mental Model: Index = Fixed Coupon + Upfront

CDS indices trade with a *fixed coupon* and an *upfront* — much like a bond trading with a fixed coupon and a price. The market still quotes a spread (analogous to a bond's yield). At trade time, the present value of the difference between the **quoted spread** and the **fixed coupon** is exchanged as upfront so the trade is fair on day one:

| Quoted spread vs coupon | Long-protection buyer's upfront |
|--------------------------|---------------------------------|
| Quoted spread $\gt$ coupon | Pays upfront to protection seller |
| Quoted spread $\lt$ coupon | Receives upfront from protection seller |

If this sign convention feels confusing, re-check the CDS MTM identity in Chapter 43 and the index quoting mechanics in Chapters 45–46.

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
| $RPV01(t,T)$ | Risky PV01 (PV of 1 bp/yr premium leg per unit notional) |
| $CS01$ | Spread DV01 (PV change for +1 bp change in quoted spread) |
| $DV01_r$ | Rates DV01 (PV change for +1 bp parallel move in chosen risk-free curve) |
| $VOD$ | Value-on-default (PV jump at default), used as JTD measure |
| $U_I$ | Index upfront payment (currency) |
| $M$ | Number of names in an index |
| $A, B$ | Tranche attachment/detachment points (as portfolio loss fractions) |
| $L$ | Portfolio loss fraction over horizon (0–1) |
| $L_{[A,B]}$ | Tranche loss fraction for tranche $[A,B]$ |

---

## 52.2 Core Risk Measures

Every strategy in this chapter reuses the same vocabulary. Before going strategy-by-strategy, align on the risk measures that show up in nearly every desk conversation. They answer three different categories of "what if":

| Category | Measures | Question answered |
|----------|----------|-------------------|
| Small linear moves | CS01 (credit), DV01 (rates) | "What if the spread/rate moves +1 bp?" |
| Jump events | JTD/VOD, Rec01 | "What if the name defaults today?" |
| Model / regime | Index basis, Corr01 | "What if the *quoting framework* shifts (basis, correlation)?" |

If these are already familiar, skim this section and jump to 52.3.

#### Quick Desk Translation

| Measure | Plain-English question | Units | What it captures (and misses) | Deep dive |
|---------|------------------------|-------|-------------------------------|-----------|
| CS01 | "Quoted spread +1 bp — how much PV?" | USD/bp | Small spread moves; **not** default jumps or nonlinearities | Chapter 43 |
| Rates DV01 | "Risk-free curve +1 bp — how much PV?" | USD/bp | Rate exposure on bonds/swaps; **not** credit spread moves | Chapters 12, 14 |
| JTD (VOD) | "Name defaults right now — instantaneous PV jump?" | USD | Default discontinuity; cannot be hedged by CS01 match | Chapters 39–40, 43 |
| Rec01 | "Recovery / final price +1% — PV impact?" | USD per 1% | Recovery dominates default P&L; needs scenario sweeps | Chapter 43 |
| Index basis | "Index moves but constituents don't (or vice versa)?" | bp or USD | Proxy hedges and index-vs-single-name P&L | Chapters 46–47 |
| Tranche PV01 / Systemic DV01 / Corr01 | "Tranche / all-name / correlation move — tranche PV?" | USD/bp or USD/1% | Model-dependent; stress-test for clustering and convexity | Chapters 50–51 |

### 1.1 CS01 / Spread DV01

CS01 is the linear spread-risk measure: how much PV changes for a 1 bp move in the *quoted* spread.

#### Definition

For any instrument valued as $PV(S, \cdot)$:

$$\boxed{CS01 \equiv \frac{\partial PV}{\partial S} \times 10^{-4} \quad \text{(USD per 1 bp)}}$$

**Always specify which spread is bumped** — they are not interchangeable:

| Instrument | Spread that is bumped |
|------------|------------------------|
| Single-name CDS | Market par spread $S(t,T)$ |
| Index CDS | Quoted index spread $S_I(t,T)$ — or, alternatively, the constituents (a different risk view) |
| Tranche | Tranche market spread (or tranche coupon/price quote, per convention) |

Mixing the index *quoted* CS01 with an *intrinsic* CS01 built from constituents is a common pitfall.

#### Anchor for Single-Name CDS

Differentiating the CDS MTM identity $V(t,T) = (S(t,T) - S_0) \cdot RPV01(t,T)$ with respect to $S$:

$$dV \approx RPV01(t,T) \\, dS$$

So, per unit notional:

$$\boxed{CS01_{\text{CDS}} \approx RPV01(t,T) \times 10^{-4}}$$

#### Common Mistakes (Watch For These in P&L Explains)

1. **No spread specified** — calling something "CS01" without saying *which* spread was bumped (par vs index vs tranche).
2. **Total CS01 matched, buckets mismatched** — net-zero overall, but big curve-shape exposure across maturity buckets.
3. **"CS01-hedged = risk-hedged" fallacy** — ignoring JTD, recovery, funding, and basis. These are the usual sources of overnight P&L surprises.

If these distinctions feel unfamiliar, Chapter 43 is the prerequisite for the rest of this chapter.

---

### 1.2 Rates DV01

Rates DV01 isolates interest-rate exposure on funded instruments (cash bonds, some swap legs). It only enters this chapter when a strategy has a bond leg.

$$\boxed{DV01_r \equiv \frac{\partial PV}{\partial r} \times 10^{-4} \quad \text{(USD per 1 bp)}}$$

where $r$ is the relevant risk-free curve level (or yield). Standard duration/DV01 hedging immunizes against small parallel shifts; non-parallel curve changes leave key-rate residuals.

---

### 1.3 Jump-to-Default (JTD) Exposure

JTD is the discontinuous PV jump if the name defaults today. Where CS01 captures small spread moves, JTD captures the cliff.

#### Definition

$$\boxed{JTD \equiv PV_{\text{post-default}} - PV_{\text{pre-default}} \quad \text{(USD)}}$$

#### CDS Expression (Value-on-Default)

For a CDS, the value-on-default (per unit notional) is:

$$VOD = (1 - R - \text{Accrued Premium}) - (S(t,T) - S_0) \cdot RPV01(t,T)$$

(long protection sign). In these notes, $JTD = VOD \times \text{Notional}$.

#### Why It Matters

A CS01-matched hedge can look "tight" for daily spread noise while leaving an enormous one-day default jump.

**Concrete numbers:** Long protection on USD 5mm, recovery 40%. The protection payment at default is roughly
$$(1-R) \cdot N = 0.60 \times 5{,}000{,}000 = 3{,}000{,}000 \text{ USD}$$
(before accrued premium and any pre-default MTM). **CS01 matching does not hedge JTD** — this matters most when single-name default is hedged with an index.

---

### 1.4 Recovery / Final-Price Sensitivity

Recovery is the second key state variable in default scenarios. Many "basis" and "hedge" surprises trace back to mismatched recovery assumptions.

#### Definition

$$Rec01 \equiv \frac{\partial PV}{\partial R} \times 0.01 \quad \text{(USD per 1\\% recovery)}$$

The CDS recovery sensitivity is naturally expressed in terms of the CDS value and $(1 - R)$ (the book's recovery-DV01 formulation).

#### Final Price (Auction Settlement)

The CDS protection leg can be cash-settled at a final price determined by dealer poll/auction. The payoff is **face value minus recovery price**. If the auction process is unfamiliar, Chapter 40 is the practical reference.

---

### 1.5 Index Basis (Quoted vs Intrinsic) and Series/Roll Basis

If you remember one thing about CDS indices: there are **two consistent ways** to talk about "the index level," and mixing them creates P&L confusion.

| View | What it means |
|------|---------------|
| **Quoted (top-down)** | Treat the index like a single CDS and quote a flat "index spread" (market convention) |
| **Intrinsic (bottom-up)** | Price the index as the sum/average of constituents' CDS values |

The gap is the **index basis** (Chapters 46–47 cover this in depth). It matters because many desks hedge single-name or portfolio risk with an index. If the index basis moves, a "CS01-matched" hedge can still produce P&L breaks.

#### Intrinsic Value vs Market Value

O'Kane provides the precise intrinsic valuation framework. The intrinsic upfront value of an index (viewed as an equally weighted portfolio of constituent CDS) is:

$$\boxed{V_I(t) = \frac{1}{M} \sum_{m=1}^{M} (S_m(t,T) - C(T)) \cdot RPV01_m(t,T)}$$

The market upfront value using a flat index curve is:

$$\boxed{U_I(t) = (S_I(t,T) - C(T)) \cdot RPV01_I(t,T)}$$

O’Kane calls this the **intrinsic** view because it prices the index as the sum (average) of its constituent CDS values.

#### Index Basis Drivers

The **index basis** is the gap between the market-quoted index level and the bottom-up intrinsic value. It can be positive or negative and varies by maturity and index. Three structural drivers (per the sources):

1. **Contract differences (restructuring clause).** For example, the North American CDX index protection leg historically excluded restructuring ("No-Re"), while many US single-name CDS contracts used Modified Restructuring ("Mod-Re"). Different credit-event terms create a mechanical basis component. (See Chapter 39 for restructuring clauses.)

2. **Liquidity and price discovery.** The index market is often more liquid than many single names. The index can embed a different liquidity premium and can "lead" single-name spreads — especially in widening markets when investors hedge illiquid cash credit via the index.

3. **Replication vs quoting conventions.** The intrinsic view requires all constituent curves; the quoted view uses a flat index curve. Even with perfect modeling, the *quoting object* (single index curve) is not the same as the *set* of single-name curves, so a basis remains.

#### The Portfolio Swap Adjustment

To reconcile intrinsic and quoted views, O'Kane describes a **portfolio swap adjustment**: tweak the individual issuer curves so the portfolio-implied index matches the market-quoted index. The exact adjustment is somewhat arbitrary; practical desiderata are preserving each issuer's term-structure shape and relative ranking, and avoiding arbitrage artifacts.

**Mental model (four steps):**

1. Start with your best estimate of each constituent's CDS curve.
2. Price the index bottom-up to get an intrinsic upfront/spread.
3. Compare to the market-quoted index level.
4. Apply an adjustment rule to the constituent curves so the bottom-up price matches the top-down quote.

This is primarily a **model-consistency** step (important for tranche pricing and some risk decompositions). The trading/risk takeaway is simpler: **index vs constituent hedges contain an extra moving part — monitor it explicitly** (see Chapters 46–47).

> **Caveat:** portfolio swap adjustment is implemented differently across systems (spread multipliers vs hazard-rate scaling; global vs tenor-by-tenor; varying constraints). Confirm your system's method before using intrinsic/quoted decompositions or hedge weights operationally.

#### Series/Roll Basis

Indices roll every six months. On-the-run liquidity and maturity reset can create price/spread differences between old and new series.

#### Quick Summary

| Basis type | What it is | Driver |
|------------|-----------|--------|
| Quoted vs intrinsic | Model/replication gap | Restructuring clauses, liquidity, quoting conventions |
| Series basis | Liquidity + contract-cohort gap | On-the-run premium, maturity reset, composition |
| Portfolio swap adjustment | Required for tranche/structured pricing built on the index | Reconciles bottom-up with top-down |

---

### 1.6 Tranche Risk Measures (PV01, Correlation, Tail)

> **Prerequisite:** if tranches are new, pause and read Chapters 49–51. The rest of this chapter assumes comfort with attachment/detachment and the tranche loss function (Ch. 49), correlation in tranche pricing (Ch. 50), and systemic vs idiosyncratic deltas/gammas (Ch. 51).

**Why tranches are harder than single-name CDS.** In single-name CDS, "delta" almost always means CS01. In tranches, there are *multiple* deltas depending on what you bump:
- the tranche quote,
- all-name spreads (systemic),
- one name (idiosyncratic),
- correlation.

Many strategy mistakes start with mixing these objects.

#### Tranche PV01

The tranche risky PV01 is defined analogously to CDS premium-leg PV01, using expected discounted outstanding tranche notional. **Tranche PV01** is then the sensitivity to a 1 bp change in the tranche spread.

#### Correlation Sensitivity (Corr01)

$$\text{Corr01} \equiv \Delta PV \text{ for a +1\\% increase in correlation parameter}$$

The sign depends on the tranche — equity and senior tranches behave differently.

**Intuition (no math):** correlation changes *how losses arrive*: scattered defaults vs clustered waves. In the limit of very high correlation, outcomes look more "all-or-nothing" (from the protection seller's perspective):
- **Senior tranches** become *riskier* — more tail/clustering exposure → expected loss on senior rises.
- **Equity tranches** become *less risky* — fewer scattered "death by a thousand cuts" outcomes → expected loss on equity falls.

For a long-protection holder the signs flip: long senior protection **gains** as correlation rises (Corr01 > 0); long equity protection **loses** (Corr01 < 0). That is why equity and senior tranches typically have opposite Corr01 signs.

#### Tail / Clustering Risk

Tranche payoff is nonlinear in portfolio loss. The tranche loss fraction is:

$$\boxed{L_{[A,B]} = \frac{1}{B - A} \left( \max(L - A, 0) - \max(L - B, 0) \right)}$$

Reading this piece by piece:
- $L \le A$ — tranche loss is **0** (no losses yet reach the tranche).
- $A \lt L \lt B$ — tranche loss **ramps linearly** from 0 to 1.
- $L \ge B$ — tranche is **fully written down** (loss fraction = 1).

Clustering scenarios swamp small-spread hedges: PV01 hedges fail because the payoff is driven by *realized losses*, not by spread bumps.

---

### 1.7 Carry, Rolldown, and Theta

#### Cash Bond (Brief)

| Component | Definition |
|-----------|------------|
| Carry | Coupon accrual − financing cost |
| Rolldown | Price change from aging "down" a sloped curve |

#### CDS / Index / Tranche (O'Kane definitions)

| Component | Definition |
|-----------|------------|
| Carry | Daily accrual of the contractual coupon/premium (even though paid quarterly) |
| Theta | Daily change in **full price** holding spreads/curves fixed and assuming no default |

**Long-protection intuition.** Theta = $\Delta$(protection leg) − $\Delta$(premium leg). Both effects typically push it negative: one day less of protection remains, and premium-leg cashflows are one day closer (higher PV).

**Risk-report pattern (per source).** For a protection buyer, carry is typically **negative** (you pay premium). Absolute carry is largest for the most junior tranche (highest contractual spread).

If theta still feels abstract, see Chapter 43 (CDS theta); for tranche carry/theta in risk-report form, Chapter 51 is the companion chapter.

#### Rolldown vs Theta

For a position with unchanged spreads and curve shape:

| Component | Meaning |
|-----------|---------|
| Carry | Coupon accrual (deterministic) |
| Rolldown | PV change from aging along a sloped curve (a piece of theta) |
| Theta | Total time decay = rolldown + other time effects (maturity shortening, etc.) |

If the spread curve is upward sloping, aging moves you to a shorter-maturity point on the curve, which (all else equal) reduces the implied par spread.

#### What We Do in This Chapter

We use **carry + rolldown + spread move + events** as the P&L-explain framework for index positions, treating rolldown as the component of theta due to curve slope.

---

### 1.8 Liquidity / Funding / Financing Risk

The CDS–cash basis is defined as:

$$\text{CDS basis} = \text{CDS spread} - \text{Bond Libor spread}$$

The "bond Libor spread" is typically the asset swap spread for fixed-rate bonds near par. See Chapter 27 for asset-swap-spread mechanics and why premium bonds create basis traps.

**Why the basis persists:**
- Unfunded CDS vs funded bonds (financing asymmetry)
- Delivery option in physical settlement
- Technical default differences
- Loss-on-default mismatch ($1-R$ vs $P-R$)
- Premium accrued at default differs across CDS and cash
- Market liquidity and supply/demand technicals

**Big picture:** "cheap" vs "rich" basis often reflects funding/liquidity optionality, not pure expected default loss.

---

## 52.3 The Governing Principle: Strategy = Exposures + Hedges + Failure Modes

A credit strategy is a **risk blueprint**, not a slogan:

$$\text{Strategy} = \underbrace{\text{exposure decomposition}}_{\text{what you truly own}} + \underbrace{\text{hedge set + ratios}}_{\text{what you try to neutralize}} + \underbrace{\text{scenario suite + failure modes}}_{\text{what can still break}}$$

This aligns with the risk-management emphasis on stress/scenario thinking and "perfect storm" risk (multiple adverse moves at once).

### Strategy Card Template

Every Strategy Card in this chapter follows the same seven-section template:

| # | Section | What goes here |
|---|---------|----------------|
| 1 | **Objective** | Mispricing or risk premium targeted (conceptual; no forecasts) |
| 2 | **Instruments and position construction** | Legs, notionals, maturities |
| 3 | **Exposure decomposition** | Table with units: CS01 by bucket, JTD, recovery sensitivity, basis sensitivity, rates DV01 / funding carry (where relevant) |
| 4 | **Hedge set and hedge ratios** | Hedge instruments and the math for sizing |
| 5 | **Scenario test suite** | Parallel move; dispersion; default event; roll/series basis (indices); correlation/tail/clustering (tranches) |
| 6 | **Failure modes** | What breaks and why |
| 7 | **Implementation checklist** | Repricing checks, unit checks, bump stability |

The discipline is *the same* across all strategies — only the contents change.

---

## 52.4 Strategy Family A: Basis Strategies

If you want the longer “cash vs swap vs CDS” foundations behind these trades, Chapter 27 is the companion chapter (asset swaps, swap spreads, CDS-bond basis, and the premium-bond trap). Chapter 43 is the prerequisite for the risk-measure language (CS01/JTD/recovery/theta).

### A1) Bond–CDS Basis Framework (Cash vs Synthetic)

The bond–CDS basis is the spread difference between a CDS and the equivalent cash bond. It is one of the most-studied phenomena in credit markets:

$$\boxed{\text{CDS basis} = \text{CDS spread} - \text{Bond Libor spread}}$$

The "bond Libor spread" is typically the asset swap spread for fixed-rate bonds near par. The sign of the basis tells you which market is pricing protection more expensively:

| Sign | Meaning |
|------|---------|
| Positive basis | CDS protection more expensive than the bond spread suggests |
| Negative basis | Protection cheap relative to cash bond |

The basis persists because CDS and bonds are fundamentally different contracts. O'Kane organizes the drivers into **fundamental factors** (contractual differences) and **market factors** (trading dynamics). Treat both lists as a checklist when a "fully hedged" basis position surprises you.

#### 3.1.1 Fundamental (Contract-Driven) Factors

| # | Factor | What it does |
|---|--------|--------------|
| 1 | **Funding asymmetry** | CDS is economically unfunded; bonds require financing. Marginal funding level shows up in the basis. |
| 2 | **Delivery option** (physical settlement) | Protection buyer can deliver the cheapest deliverable bond after a credit event — captures price differences across deliverables (e.g., delivering a 37 bond instead of a 43 bond captures 6 points). |
| 3 | **Technical default / credit-event definition** | CDS triggers can be broader than bond payment default (e.g., restructuring per documentation). Extra trigger risk affects CDS pricing. |
| 4 | **Loss-on-default mismatch** | CDS pays $(1-R)$; cash-bond loss is $(P-R)$. If $P \neq 100$, CS01 matching leaves a JTD residual. |
| 5 | **Accrued premium at default** | CDS settles accrued at default; bond coupons stop and accrued can be lost. Small but matters in tight basis carry. |
| 6 | **CDS spreads non-negative** | Asset-swap spreads can go negative (high-quality Treasuries); CDS cannot. Asymmetric floor. |

#### 3.1.2 Market (Flow-Driven) Factors

| # | Factor | What it does |
|---|--------|--------------|
| 1 | **Relative liquidity** | CDS clusters at standard tenors (3/5/7/10Y, IMM dates); bonds are liquid in specific issues. Mismatches move basis. |
| 2 | **Synthetic CDO technical** | Dealer hedging of synthetic CDO issuance moves CDS spreads via constituent flows. |
| 3 | **New issuance / loans** | Corporate bond and bank loan origination drive CDS hedging flows that decouple from cash. |
| 4 | **Convertible arbitrage** | Hedging convertibles creates protection-buying pressure in CDS. |
| 5 | **Shorting asymmetry** | Easier to short credit via CDS (buy protection) than via cash bonds. CDS can move first on negative news. |
| 6 | **Funding / liquidity regimes** | Even though CDS is unfunded, bond-leg financing and balance-sheet costs can shift; basis can gap. |

These effects interact, so it is hard to pin a single dominant cause in real time.

#### Strategy Card: A1 — Bond–CDS Basis Package (Risk-First Framing)

> **In desk language:** A1 is “cash vs synthetic.” You try to hedge away the obvious risks (rates DV01, sometimes CS01), so the P&L that remains is usually about **basis + funding + event mechanics**, not about day-to-day spread noise.
>
> - If the package “mysteriously” loses money, first ask: did **funding/repo** move (Example 1B), did the **basis** move, or did a **default/auction/recovery** assumption change?
> - A CS01 match is not enough: the default payoff depends on bond price vs recovery ($P-R$) versus CDS payoff ($1-R$).

##### 1. Objective

Isolate and measure the CDS–cash basis as a relative-value position, with explicit accounting for the residual risks that a CS01 match leaves behind.

##### 2. Instruments and Position Construction

| Leg | Role |
|-----|------|
| Cash bond (or asset-swap representation) | Funded credit exposure |
| Single-name CDS on same reference entity | Unfunded credit exposure (synthetic) |
| Rates hedge (swap or UST) | Removes bond's rates DV01 |

Direction is not prescribed; the educational focus is *what risks remain* after hedging.

##### 3. Exposure Decomposition

Buckets: 0–3Y / 3–7Y / 7–10Y (coarse).

| Exposure | Bond leg | CDS leg | Net comment |
|----------|----------|---------|-------------|
| $CS01_{3\text{–}7}$ (USD/bp) | $\lt 0$ (long bond) | $\gt 0$ (long prot) | Can offset if sized |
| Other CS01 buckets | usually 0 | usually 0 | Maturity placement |
| Rates DV01 (USD/bp) | $\neq 0$ | $\approx 0$ | Residual unless hedged |
| JTD (USD) | $-(P - R) \cdot N$ | $+VOD \cdot N$ (long prot) | Match is *not* automatic |
| Rec01 (USD/1%) | depends on bond view | CDS recovery sens | Mismatches matter |
| Basis sensitivity | yes | yes | **This is the residual you keep** |

##### 4. Hedge Set and Hedge Ratios

**Rates hedge (DV01 neutralization):**

$$\boxed{N_{\text{rates hedge}} = -\frac{DV01_r^{\text{bond}}}{DV01_r^{\text{hedge instr}}}}$$

**Credit hedge (CS01 match):**

$$\boxed{N_{\text{CDS}} = -\frac{CS01_{\text{bond}}}{CS01_{\text{CDS per USD notional}}}}$$

**Event hedge (JTD match):** size to match $-(P-R) \cdot N_B$ on the bond leg with $+(1-R) \cdot N_{\text{CDS}}$ on the CDS leg (loss-on-default mismatch — see Example 1).

##### 5. Scenario Test Suite

| # | Scenario | What it tests |
|---|----------|---------------|
| 1 | Parallel spread move (bond + CDS widen together) | CS01 match quality |
| 2 | Dispersion (CDS widens, bond flat — or vice versa) | Pure basis exposure |
| 3 | Default event with recovery/final-price sweep | JTD match and Rec01 |
| 4 | Rates move | Rates DV01 hedge effectiveness |
| 5 | Delivery option / restructuring clause variation | Qualitative — see failure modes |

##### 6. Failure Modes

**Funding-related:**
- **Funding/repo shock** — bond leg is funded, CDS is not. A repo spike raises bond carry without affecting the CDS leg, breaking carry assumptions (Example 1B).
- **Repo specials** — if the bond goes "special," carry becomes punitive. CDS has no equivalent.

**Contract-related:**
- **Delivery option / CTD** — after certain credit events, the protection buyer chooses the cheapest deliverable. The delivered asset may not match the bond you hedged.
- **Restructuring clause mismatch** — CDS spreads depend on the restructuring clause; mixing Mod-Re single names with No-Re indices creates basis to restructuring events.
- **Loss-on-default mismatch** — CDS pays $1-R$; bond loss is $P-R$. *Examples:* bond at 105 defaults at 40% recovery → bond loses 65 points, CDS pays 60 → under-hedged. Bond at 95 → loses 55, CDS pays 60 → over-hedged.

**Market-related:**
- **Synthetic CDO technicals** — dealer hedging of CDO issuance can tighten single-name CDS, compressing basis.
- **New issuance and hedging flows** — bond and loan issuance push CDS and cash differently.
- **Convertible flows** — drive protection buying that widens CDS without matching cash move.
- **Liquidity divergence** — CDS is the easier short. In stress, CDS spreads gap wider than bonds.

**Monitoring checks** (thresholds are desk-specific):

| Signal | What it indicates |
|--------|-------------------|
| CS01 drift post-trade or after spread moves | Hedge staleness |
| JTD mismatch | Bond price ≠ par or recovery assumption changed |
| Funding / liquidity regime shift | Repo terms tightening, bid/offer widening |

##### 7. Implementation Checklist

- [ ] Reprice each leg under the same discounting assumptions; document any differences.
- [ ] Verify signs: long protection CS01 $\gt 0$; long bond credit CS01 $\lt 0$.
- [ ] Bump stability: CS01 stable for $\pm 1$ bp bumps (otherwise you are in a nonlinear regime).
- [ ] Event test: run "default today" with a recovery sweep.

#### Negative Basis: Why It Persists

A **negative basis** means CDS spread trades *inside* (below) the bond asset-swap spread — synthetic protection prices cheaper than cash credit risk. The condition looks arbitrageable but persists due to **funding-driven clientele effects**:

| Participant type | Preference |
|------------------|------------|
| Cheap-funding (e.g., bank balance sheets) | Often prefer **cash bonds** |
| Expensive-funding (e.g., hedge funds) | Often prefer **CDS** |

Technical flows compound the effect: synthetic CDO issuance leads dealers to hedge by selling protection across many names, mechanically tightening single-name CDS and pushing the basis more negative.

#### Positive Basis: The Squeeze

A **positive basis** means CDS trades *outside* (above) bond spreads. The market factors that produce this:

- Demand for protection (shorting credit is easier via CDS).
- CDO dealer hedging in the opposite direction.
- Convertible-arb hedging flows.
- New-issuance effects that widen CDS faster than bonds.

The dangerous scenario is the **positive basis squeeze** — a trader short the basis (short CDS, long bond) sees the basis widen *further*. The stress mechanics:

1. **Protection demand surges.** Negative news → expressing the bearish view via CDS is operationally easier than shorting cash bonds. CDS widens faster than cash, widening the basis.
2. **Repo funding spikes.** The bond leg becomes expensive to carry (Example 1B).
3. **Forced unwinds.** Margin calls on the CDS leg force liquidation at the worst time, crystallizing losses.

**Stress-test mindset.** Basis trades are funding- and liquidity-contingent. When funding terms or market liquidity shift, the P&L distribution changes *qualitatively*. A CS01-matched package can still take large adverse moves.

---

### A2) Single-Name vs Index (Proxy Hedge / Basis)

#### Strategy Card: A2 — Proxy Hedge: Single-Name CDS Hedged by an Index

> **In desk language:** A2 is “hedge the tape.” You keep the single-name view but try to remove broad-market drift with an index hedge.
>
> - What remains is usually **idiosyncratic/default jump risk** (a single default is huge for the name and small for the index), plus **index-vs-name basis** and **roll/series** effects.
> - If you want an intuition check, read Example 3 before you worry about any math: it shows how a clean CS01 match can still leave a big default scenario.

##### 1. Objective

Reduce broad-market credit-spread exposure of a single-name position using an index hedge, leaving mainly idiosyncratic risk (and default jump risk).

##### 2. Instruments and Position Construction

| Leg | Role |
|-----|------|
| Single-name CDS (maturity $T$) | Position to be hedged |
| Index CDS (similar maturity $T$, or nearest liquid point) | Hedge instrument |
| Optional beta $\beta$ | Scaling factor (treated as an input; estimation not standardized) |

##### 3. Exposure Decomposition

CS01 bucketed into 0–3Y / 3–7Y / 7–10Y (coarse).

| Exposure | Single-name leg | Index hedge leg | Net comment |
|----------|-----------------|-----------------|-------------|
| $CS01_{3\text{–}7}$ (USD/bp) | dominant | dominant | Can be matched |
| JTD (USD) | large (single default) | small per-name (diversified) | **Not hedged** by CS01 match |
| Rec01 (USD/1%) | meaningful | meaningful | Can mismatch |
| Basis sensitivity | single vs index composition | yes | Residual "proxy basis" |
| Roll/series basis | none | yes | Index-specific residual |

##### 4. Hedge Set and Hedge Ratios

**CS01 match:**

$$\boxed{N_{\text{index}} = -\frac{CS01_{\text{single}}}{CS01_{\text{index per USD notional}}}}$$

**With beta scaling $\beta$:**

$$N_{\text{index}} = -\frac{\beta \cdot CS01_{\text{single}}}{CS01_{\text{index per USD notional}}}$$

##### 5. Scenario Test Suite

| # | Scenario | What it tests |
|---|----------|---------------|
| 1 | Parallel market move (single + index together) | CS01 match quality |
| 2 | Idiosyncratic move (single only) | Residual idiosyncratic exposure (the *intended* exposure) |
| 3 | Single-name default with recovery sweep | JTD residual — the central failure mode |
| 4 | Index roll / series basis change | Index-specific residual |

##### 6. Failure Modes

| Failure | Why it happens |
|---------|----------------|
| Composition mismatch | Index may not contain the name; weights differ; sector tilts |
| Index basis (quoted vs intrinsic) | Hedging on-the-run vs off-the-run creates constituent mismatch |
| **Default jump dominates** | Single-name default vs index hedge — the diversified index absorbs only the per-name slice |

##### 7. Implementation Checklist

- [ ] Confirm maturity alignment and coupon conventions.
- [ ] Recompute CS01 after the trade; hedged bucket(s) should be near zero.
- [ ] Run event stress: single-name default today; quantify the residual.

---

### A3) CDS Curve Steepener / Flattener

Credit curve trades express a view on the *shape* of a single name's CDS spread term structure rather than its level. They extend the relative-value framework from Chapter 44.

#### 3.3.1 Curve Shapes and What They Signal

| Shape | Typical context | Hazard-rate implication |
|-------|-----------------|--------------------------|
| **Upward-sloping** | Investment grade | Hazard rate increases with tenor — long-term default risk exceeds near-term |
| **Flat** | Stable, mixed signal | Roughly constant hazard rate |
| **Inverted** | Distressed high-yield | Market prices near-term default as dominant; longer-dated hazard perceived as lower *conditional on survival* |

**Desk intuition for inversion.** An inverted curve is a "survival bet" by the market. *Toy example:* a name trading 800 bp at 1Y and 500 bp at 5Y is saying the market is overwhelmingly focused on near-term default risk. A **curve flattener** (short the short-dated protection, long the longer-dated protection — the structure used in Example 17) monetizes normalization *if the name survives*: as the short-end spread tightens, the short-protection leg gains, dominating the smaller long-protection leg. If the name *defaults* instead, both legs trigger; CS01-neutral sizing requires a *larger* short-protection notional on the short tenor, so the flattener typically realizes a meaningful net loss on default (the larger short leg pays out more than the smaller long leg receives).

#### 3.3.2 Forward CDS Curves

Spot CDS curves map to forward curves (O'Kane, Table 9.1):

| Spot shape | Forward curve vs spot |
|-----------|------------------------|
| Upward-sloping | Forward $\gt$ spot — forward-starting protection more expensive |
| Inverted | Forward $\lt$ spot — declining expected hazard rates |

#### 3.3.3 Curve Arbitrage Bounds

O'Kane derives an arbitrage lower bound for inverted curves. Approximately:

$$S_m \gtrsim S_{m-1} \left(\frac{T_{m-1}}{T_m}\right)$$

A severely inverted curve can violate no-arbitrage constraints implied by the CDS payoff structure and non-negative survival probabilities. **Red-flag rule:** if you see "impossible" forward or default-implied behavior in a curve build, suspect the curve-construction inputs (quotes, recovery, interpolation) *before* treating the violation as tradable edge.

#### Strategy Card: A3 — CDS Curve Steepener/Flattener

> **In desk language:** A3 is “trade the curve shape, not the level.” You make money if the front end and back end move *differently* (steepen/flatten), and you try to remove pure “parallel spread” risk with CS01 matching.
>
> - The most common surprise is **default/JTD mismatch**: CS01-neutral sizing usually requires unequal notionals, so a default can create a large residual jump even when day-to-day spread P&L is near zero.
> - If you’re new to credit curves and hazard rates, Chapter 42 is the prerequisite; if you’re new to CDS risk measures, Chapter 43 is.

##### 1. Objective

Express a view on the *shape* of a single name's credit curve:
- **Steepening** — short end widens relative to long end (slope $S_{\text{short}} - S_{\text{long}}$ increases — e.g., an upward-sloping curve flattens toward the short end, or an inverted curve becomes more inverted).
- **Flattening** — long end widens relative to short end (slope $S_{\text{short}} - S_{\text{long}}$ decreases — e.g., an upward curve steepens, or an inverted curve normalizes).
- **Survival bet** — for a name with an inverted curve, "the name survives" implies the short end will **tighten** relative to the long end, i.e., the inverted curve **normalizes**. Per the definitions above, normalization is **flattening**, so the **flattener** profits (matches Example 17). The steepener is the opposite trade and *loses* on this scenario.

##### 2. Instruments and Position Construction

| Trade | Short tenor (e.g., 3Y) | Long tenor (e.g., 5Y) |
|-------|------------------------|------------------------|
| **Steepener** | Buy protection | Sell protection |
| **Flattener** | Sell protection | Buy protection |

Size CS01-neutral to isolate the curve-shape view from parallel spread moves.

##### 3. Exposure Decomposition

| Exposure | Short leg | Long leg | Net |
|----------|-----------|----------|-----|
| $CS01_{3Y}$ (USD/bp) | $\gt 0$ (long prot) | 0 | positive |
| $CS01_{5Y}$ (USD/bp) | 0 | $\lt 0$ (short prot) | negative |
| $CS01_{\text{net}}$ | — | — | $\approx 0$ by construction |
| JTD (USD) | positive (receive payment) | negative (pay) | Net depends on accrual + sizing |
| Rec01 (USD/1%) | meaningful | meaningful | Partial offset |

##### 4. Hedge Set and Hedge Ratios

CS01-neutral sizing:

$$\boxed{N_{\text{long}} = N_{\text{short}} \times \frac{RPV01_{\text{short}}}{RPV01_{\text{long}}}}$$

*Worked example* (3Y/5Y steepener, $RPV01_{3Y} = 2.8$, $RPV01_{5Y} = 4.5$):

$$N_{5Y} = N_{3Y} \times \frac{2.8}{4.5} = 0.622 \times N_{3Y}$$

The notionals are **unequal** — this is what creates the JTD residual.

##### 5. Scenario Test Suite

| # | Scenario | Expected behavior |
|---|----------|-------------------|
| 1 | Parallel widening | $\approx 0$ P&L (validates CS01 neutrality) |
| 2 | Curve steepening (3Y widens more than 5Y) | Steepener profits; flattener loses |
| 3 | Curve flattening (5Y widens more than 3Y) | Steepener loses; flattener profits |
| 4 | Default event | Both legs trigger; net JTD depends on sizing, accrual, RPV01 |
| 5 | Inverted curve normalizes (3Y tightens more than 5Y) | **Flattener profits** (the "survival bet"); steepener loses |

##### 6. Failure Modes

| Failure | Why it happens |
|---------|----------------|
| **Default JTD residual** | CS01 neutrality requires *unequal* notionals → default produces a net jump |
| Convexity on large parallel moves | RPV01 differs across tenors; CS01 neutrality is only first-order |
| Short-end liquidity | Short-dated CDS less liquid → bid/offer can consume curve P&L |
| Coupon convention mismatch | Different conventions across tenors create small basis effects |

##### 7. Implementation Checklist

- [ ] Verify CS01-neutral by parallel $\pm 1$ bp bump (net P&L $\approx 0$).
- [ ] Bump curve slope: short $+1$ bp, long $-1$ bp (and reverse); verify correct sign.
- [ ] Default scenario with recovery sweep; quantify JTD residual.
- [ ] Confirm bid/offer at both tenors; expected curve P&L should exceed round-trip costs.

---

## 52.5 Strategy Family B: Carry, Rolldown, and Roll in CDS Indices

If CDS indices are new, Chapters 45–47 (index structure, intrinsic vs quoted, hedging) are the prerequisites. The goal here is not to “predict spreads,” but to build a clean desk-style decomposition of what moved and why.

### Definitions (Index-Specific, Risk-First)

#### Index Upfront Mechanics

Indices trade with a **fixed coupon $C$** and an **upfront** at settlement. Coupon is paid quarterly Actual/360. Indices roll every six months.

**Beginner intuition.** The coupon is standardized (like a bond coupon). The market quotes a "spread" that moves daily (like a yield). The upfront is the one-time cash amount that reconciles the fixed coupon with today's quoted spread so the trade is fair on day one. That is why traders speak of both an index "price" and an index "spread."

#### Carry and Rolldown (Index)

| Term | Definition (this chapter) |
|------|---------------------------|
| **Carry** | Deterministic premium accrual / coupon cashflow over the horizon (plus deterministic accrual conventions) |
| **Rolldown** | The piece of P&L from aging along a sloped spread curve, holding the curve fixed (a component of theta) |

Rolldown follows directly from the theta definition: an upward-sloped spread curve produces a "roll-down" effect, while a downward-sloped curve produces a "roll-up."

---

### B1) Index Carry/Rolldown Holding-Period P&L (Risk Decomposition, Not a Recommendation)

#### Strategy Card: B1 — Index Holding-Period Decomposition ("Carry + Rolldown + Spread Move + Events")

> **In desk language:** B1 is the desk’s default “P&L explain” for an index position. Every day you want to be able to say: *how much was carry (accrual), how much was rolldown (aging along the curve), how much was the spread move (curve shift), and what was event-driven (defaults, roll, execution)?*
>
> If you’re new, start with Examples 6–10. They are less about “trading” and more about learning to read a credit P&L like a desk.

##### 1. Objective

Build a disciplined P&L explain for an index position, splitting it into:

- **Carry** (deterministic cashflows),
- **Rolldown / theta** (curve aging),
- **Spread moves** (the random piece),
- **Events** (defaults and index mechanics).

##### 2. Instruments and Position Construction

| Component | Role |
|-----------|------|
| Index CDS (long or short protection) | Core position |
| Optional: another series | Manage roll exposure |
| Optional: constituents | Manage default/event risk (imperfect — see A2) |

##### 3. Exposure Decomposition

| Exposure | Units | Notes |
|----------|-------|-------|
| CS01 buckets | USD/bp | Linear spread risk |
| JTD (per index default) | USD | Depends on $M$ and recovery / final price |
| Rec01 | USD/1% | Recovery dependence |
| Roll / series basis | USD per bp of basis | Old vs new series quotes / liquidity |
| Carry | USD/day or USD/horizon | Coupon accrual (sign depends on long/short prot) |
| Rolldown | USD/horizon | Theta due to curve slope |

##### 4. Hedge Set and Hedge Ratios

Spread hedge with another index:

$$\boxed{N_{\text{hedge}} = -\frac{CS01_{\text{target}}}{CS01_{\text{hedge}}}}$$

For constituent hedges, see Strategy A2 and Example 4 — note the basis complications.

##### 5. Scenario Test Suite

| # | Scenario | What it tests |
|---|----------|---------------|
| 1 | Parallel spread move | CS01 sensitivity |
| 2 | Dispersion (constituents diverge from index) | Quoted-vs-intrinsic basis |
| 3 | Default event inside index | JTD + notional reduction logic |
| 4 | Roll / series basis change | On-the-run vs off-the-run repricing |
| 5 | Liquidity / execution-cost stress | Bid/offer widening assumption |

##### 6. Failure Modes

| Failure | Why it happens |
|---------|----------------|
| Default cluster | Invalidates linear approximations |
| Index basis / portfolio swap adjustment | Hedging vs constituents introduces extra moving parts |
| Execution-cost drag | Short-horizon carry/rolldown can be eaten by costs |

##### 7. Implementation Checklist

- [ ] Separate clean vs accrued P&L (coupon-date jumps should net with cash account).
- [ ] Verify CS01 with symmetric $\pm 1$ bp bumps.
- [ ] Event check: one-name default — verify notional reduction logic.

---

### B2) Roll / Series Switch Mechanics (On-the-Run vs Off-the-Run)

The index roll is a critical event: indices roll every six months, and investors who want to stay on-the-run typically sell the old series and buy the new one.

#### 3.4.1 What Drives Roll P&L

Two primary sources of roll P&L:

| # | Effect | Mechanism |
|---|--------|-----------|
| 1 | **Composition change** | Names removed/replaced based on credit-quality and liquidity criteria. Can improve or worsen perceived credit quality. Liquidity-driven replacements are not guaranteed to tighten spreads. |
| 2 | **Maturity extension** | New series is ~6 months longer than the previous one. On an upward-sloping curve, longer maturity → wider fair spread, all else equal. |

The two effects can offset. A credit-upgraded new index with longer maturity may trade at the *same* spread as the old index if composition improvement offsets the curve steepness. (See Examples 8 and 8B.)

#### 3.4.2 The Series Basis

The price/spread gap between old (off-the-run) and new (on-the-run) series. It persists due to:

| Driver | Effect |
|--------|--------|
| **Liquidity premium** | On-the-run typically has tighter bid/offer |
| **Maturity mismatch** | Different remaining maturities → different RPV01 |
| **Composition differences** | Credits may enter or exit the index |

#### Strategy Card: B2 — Index Roll/Series Switch (Mechanics + Risk)

> **In desk language:** B2 is “stay on-the-run.” You are paying (or receiving) something to switch from an old series to the new, more liquid series.
>
> - Two sources of confusion: (i) **notional scaling** (CS01 changes because maturity changes) and (ii) **series/composition basis** (the contract you’re switching into is not identical).
> - If you want intuition, read Examples 8 and 8B before worrying about hedge ratios.

##### 1. Objective

Understand the P&L drivers of switching between series — composition change, maturity reset, and on-the-run liquidity premium.

##### 2. Instruments and Position Construction

| Component | Description |
|-----------|-------------|
| Old series | About to go off-the-run; lower liquidity |
| New series | On-the-run; tighter bid/offer; longer maturity by ~6 months |
| Switch transaction | Close old + open new → cash exchange = MTM difference + execution costs |

##### 3. Exposure Decomposition

| Exposure | Units | Typical effect |
|----------|-------|----------------|
| Series basis | USD/bp | **Key driver** |
| CS01 mismatch | USD/bp | Residual if maturities differ |
| Default / event | USD | Both series exposed; composition differs |
| Liquidity | qualitative | On-the-run usually tighter bid/offer |

##### 4. Hedge Set and Hedge Ratios

For a spread-neutral switch:

$$\boxed{N_{\text{new}} = N_{\text{old}} \cdot \frac{CS01_{\text{old}}}{CS01_{\text{new}}}}$$

(sign consistent with long/short protection). Because the new series has higher CS01 per dollar (longer maturity), this typically requires a *smaller* notional — see Example 8.

##### 5. Scenario Test Suite

| # | Scenario | What it tests |
|---|----------|---------------|
| 1 | Roll basis widens / tightens | Series basis exposure |
| 2 | Composition change (idiosyncratic) | Replacement names differ |
| 3 | Default just before / after roll | Mechanical cashflow and notional effects |

##### 6. Failure Modes

- Composition mismatch and basis shocks dominate.
- Liquidity dries up in off-the-run; switching costs spike.

##### 7. Implementation Checklist

- [ ] Confirm coupon and upfront conventions for each series — do not assume.
- [ ] Ensure consistent curve used to compute RPV01 in both series.

---

## 52.6 Strategy Family C: Correlation and Tranche Relative Value

> **If tranches are not yet intuitive:** read Chapters 49–51 first. On a first pass through Chapter 52, it is completely reasonable to skip to Section 52.7. The basis/roll/capital-structure strategies stand on their own.

Tranches are fundamentally different from single-name CDS and indices. Their value depends not just on spread levels but on the *distribution* of portfolio losses—and that distribution is shaped by correlation. O'Kane provides a comprehensive risk framework that reveals why tranche positions often behave counter-intuitively.

### 52.6.1 Tranche PV Decomposition and Why Correlation Matters

#### Conceptual PV Decomposition

A tranche behaves like a credit derivative on the **portfolio loss distribution**:

| Leg | Driven by |
|-----|-----------|
| Protection | Expected discounted losses inside the tranche |
| Premium | Outstanding tranche notional over time |

#### Systemic vs Idiosyncratic Risk

There are *two* kinds of spread move for tranches — and they have *different* deltas, gammas, and hedge notions:

| Type | What you bump | Hedge concept |
|------|---------------|---------------|
| **Systemic** (portfolio-wide) | All issuer curves together (parallel) | Index-based hedge |
| **Idiosyncratic** (single name) | One issuer's curve, others fixed | Name-specific hedge |

#### Robust Qualitative Patterns

Rather than memorizing any single risk-report snapshot, focus on three patterns (see Chapter 51 for definitions):

1. **Leverage is highest at equity and falls with seniority.** A small notional slice in equity embeds large exposure to portfolio-wide spread moves.
2. **Systemic gamma flips sign across the stack.** Equity can have *negative* systemic gamma (unfavorable convexity); senior can have *positive* systemic gamma.
3. **Corr01 flips sign.** Increasing correlation reallocates probability mass between "scattered defaults" and "clusters" — benefits one part of the stack and hurts another.

#### Plain-English Picture

| Tranche | What it insures | Reacts most to |
|---------|-----------------|----------------|
| **Equity (0–3%)** | The *first* slice of portfolio loss | Lots of small/medium bad outcomes ("death by a thousand cuts") |
| **Senior (e.g., 15–30%)** | Catastrophe — only pays under clustered-default outcomes | Tail / clustering scenarios |

When correlation rises, the world becomes more "all-or-nothing":
- Fewer scattered losses → equity-protection sellers benefit (long-protection equity gets *worse*).
- More clustered loss scenarios → senior-protection sellers suffer (long-protection senior gets *better*).

That is the core reason equity and senior tranches react in *opposite* directions to correlation changes.

#### Correlation / Dependence Affects the Tail

Higher correlation increases the probability of **clustered defaults** (tail events), redistributing expected losses across attachment points:

| Tranche (long protection) | Sign of Corr01 |
|---------------------------|-----------------|
| Equity | Negative (loses as correlation rises) |
| Senior | Positive (gains as correlation rises) |

This natural tension — equity and senior have *opposite* Corr01 — is the building block of correlation trading. A trader can combine tranches to isolate or neutralize correlation exposure (see 52.6.2 and Examples 12 and 20).

#### Gamma Trading

A tranche's **systemic vs idiosyncratic gamma profile** can be used to express a view on "market-wide" vs "name-specific / dispersion" scenarios, while reducing first-order spread exposure.

**Sign intuition:** for a given tranche, **idiosyncratic gamma often has the opposite sign to systemic gamma** — reflecting the difference between "one name deteriorates" vs "everything widens."

**Trade-off to evaluate explicitly:** "long convexity" (positive gamma) positions typically come with **adverse carry/theta**. Always evaluate **carry vs convexity** together — never in isolation.

#### Base Correlation Framework

Base correlation is a market convention for quoting "implied correlation" across strikes, used to build a surface for tranche pricing and risk.

**Key structural identity:** a $[K_1, K_2]$ tranche = $[0, K_2]$ equity tranche minus $[0, K_1]$ equity tranche (linear combination). See Chapter 50 for the construction and Chapter 51 for the risk implications.

> **Interpolation warning.** Linear interpolation of base correlation is **not guaranteed arbitrage-free** for non-standard strikes. The interpolation scheme is part of the model — not a neutral choice.

---

### Strategy Card: C1 — Tranche RV / Correlation Exposure with Explicit Hedge Map

> **In desk language:** C1 is “trade the loss distribution.” You are not just trading a spread level; you are trading *where losses land* in the capital structure.
>
> - A PV01 hedge can make the book look stable for small spread moves, but **clustering/tail scenarios** can dominate tranche P&L.
> - When you hedge a tranche with an index, make sure you know whether you are matching **tranche PV01** (tranche quote) or **systemic DV01** (all-name spread bump). Mixing them is a common beginner error.
> - Chapters 50–51 are the deep dive on what these risk measures mean in practice.

##### 1. Objective

Compare tranches by which risk dominates the position:

- Spread level (PV01),
- Dependence / correlation (Corr01),
- Tail / default-clustering scenarios,
- Recovery uncertainty.

##### 2. Instruments and Position Construction

| Component | Role |
|-----------|------|
| One tranche $[A, B]$ (e.g., equity 0–3%, mezz 3–7%) | Core position |
| Index CDS | Hedge broad spread level |
| Systemic-delta hedge (sized for systemic DV01) | Hedge tranche sensitivity to systemic spread moves |
| Adjacent tranche | PV01-neutral spread hedge |

##### 3. Exposure Decomposition

| Exposure | Units | Meaning |
|----------|-------|---------|
| Tranche PV01 | USD/bp | Linear sensitivity to *tranche's own quoted spread* |
| Systemic DV01 | USD/bp | PV change under a 1 bp parallel bump to *all* issuer curves |
| Systemic delta | USD notional | Index CDS notional implied by the systemic-hedge definition |
| Corr01 | USD per 1% corr | Dependence sensitivity |
| Tail / clustering | Scenario USD | Nonlinear loss jump risk |
| Recovery sensitivity | USD/1% | Recovery affects losses and settlement |

> **Beginner check (common confusion):**
> - **Tranche PV01** = sensitivity to the *tranche's own quoted spread* (a market quote).
> - **Systemic DV01** = sensitivity to a *parallel bump to all underlying issuer curves* (a portfolio-wide credit move).
> - **Systemic delta** = a *derived hedge notional* ("how much index CDS hedges the systemic DV01"). Units: USD notional, not USD/bp.
>
> If these objects feel interchangeable, review Chapter 51 — this is where tranche Greek language diverges from single-name CDS.

##### 4. Hedge Set and Hedge Ratios

**Systemic spread hedge** (size index to offset tranche's *systemic* DV01, not its quoted PV01):

$$\boxed{N_{\text{index}} \approx -\frac{SystemicDV01_{\text{tranche}}}{CS01_{\text{index per USD notional}}}}$$

**Adjacent tranche PV01 hedge:**

$$\boxed{N_{\text{hedge tranche}} = -\frac{PV01_{\text{target tranche}}}{PV01_{\text{hedge tranche}}}}$$

##### 5. Scenario Test Suite

| # | Scenario | What it tests |
|---|----------|---------------|
| 1 | Parallel spread move (small) | PV01 hedge linear |
| 2 | Dispersion (single-name basket shocks) | Idiosyncratic exposure |
| 3 | Default event / jump (1 default) | JTD residual |
| 4 | Correlation shock ($\pm$10% or larger) | Corr01 + convexity |
| 5 | Tail / clustering (5–10 clustered defaults) | Nonlinear tranche loss |

##### 6. Failure Modes

| Failure | Why it happens |
|---------|----------------|
| PV01 hedge fails under clustering | Linear hedge vs jump payoff (Examples 13, 14) |
| Model risk | Base-correlation interpolation and mapping choices |
| Liquidity / unwind risk | Tranche markets can be thin |

##### 7. Implementation Checklist

- [ ] Bump PV01 with $\pm 1$ bp; check symmetry.
- [ ] Corr01: revalue with $\rho \pm 1\\%$; confirm stable sign/magnitude (then run a *large* shock — see Example 20).
- [ ] Run discrete default scenarios: 1, 5, 10 defaults; compare hedge performance.

---

### 52.6.2 Tranche Example: Equity vs Senior Correlation Trade

A **toy** illustration of combining two tranches to *reduce* (not eliminate) correlation exposure.

#### Setup (Toy Numbers)

| Item | Value |
|------|-------|
| Reference portfolio | 125 equal-weight names, total notional USD 1,250mm |
| Equity tranche | 0–3%, long protection |
| Senior tranche | 15–30%, long protection |
| Current implied correlation | $\rho_0$ (model details omitted) |

Risk-system Corr01 (per USD 10mm tranche notional, long protection):

| Tranche | Corr01 per $+1\\%$ |
|---------|--------------------|
| Equity (0–3%) | $-USD 80{,}000$ |
| Senior (15–30%) | $+USD 150{,}000$ |

These signs encode the intuition: higher correlation makes equity safer (hurts long-protection equity) and senior tail riskier (helps long-protection senior).

#### Correlation-Neutral Sizing (Local)

For local Corr01 neutrality:

$$N_{\text{equity}} \approx N_{\text{senior}}\times \frac{|\text{Corr01}_{\text{senior}}|}{|\text{Corr01}_{\text{equity}}|} = N_{\text{senior}}\times \frac{150}{80} = 1.875\\,N_{\text{senior}}$$

#### Residual Exposures — Why "Corr01-Neutral" ≠ "Risk-Neutral"

Even if Corr01 is near zero locally, the position still carries:

| Residual | Why it remains |
|----------|----------------|
| Systemic spread risk (DV01 / systemic delta) | Two tranches don't offset for parallel spread moves |
| Gamma / convexity | Systemic and idiosyncratic gamma profiles differ across the stack; hedge ratios drift in large moves |
| Jump / clustering | Realized defaults and clustered scenarios dominate small-move hedges |
| Model risk | Corr01 depends on parameterization and calibration (compound vs base correlation, interpolation) |

**Practical takeaway:** treat Corr01-neutrality as a **local** risk reduction. Validate the trade under stress scenarios — spread shocks, correlation shocks, clustered defaults, recovery/final price (see Example 20).

---

## 52.6.3 Strategy Selection: When to Use Which Approach

The strategy families address different risk views. The tables below match each view to a strategy and flag the dominant residual.

### Basis Strategies (A1, A2, A3)

**Use when:** you have a view on cash vs synthetic, single-name vs index, or CDS curve shape.

| Strategy | Best for | Key residual |
|----------|----------|--------------|
| **A1** Bond-CDS basis | Funding arbitrage; delivery option value; loss-on-default mismatch | Basis widening/tightening; funding shock |
| **A2** Single-name vs index | Isolating idiosyncratic risk with market hedge | Single-name JTD; composition mismatch |
| **A3** CDS curve steepener/flattener | Term-structure view | JTD mismatch from unequal notionals; parallel moves |

**Decision criteria:**

| If you believe... | Use |
|-------------------|-----|
| Funding conditions favor CDS over bonds | Basis package with long CDS (A1) |
| Single name has idiosyncratic risk to isolate | Proxy hedge with index (A2) |
| Name will outperform / underperform market | Single vs index (A2) |
| Inverted HY curve will normalize | Curve flattener (A3) — manage JTD |

### Carry and Rolldown Strategies (B1, B2)

**Use when:** you want to capture or explain time-value components of an index position.

| Strategy | Best for | Key residual |
|----------|----------|--------------|
| **B1** Index carry/rolldown | Systematic carry; P&L attribution | Spread moves; default events |
| **B2** Series roll | Manage roll cost; on-the-run liquidity | Series basis shock; composition changes |

**Decision criteria:**

| If... | Use |
|-------|-----|
| Index curves upward-sloping, you expect stability | Rolldown capture (B1) |
| New series has favorable composition | Roll early (B2) |
| Basis trade vs constituents | Watch portfolio-swap-adjustment effects |

### Correlation / Tranche Strategies (C1)

**Use when:** you have a view on correlation, dispersion, or tail risk.

| Position | Best for | Key residual |
|----------|----------|--------------|
| Long equity protection | Tail-risk hedge | Spread tightening; correlation decrease; negative carry |
| Short senior protection | Carry collection; bullish credit | Correlation increase; default clustering |
| Equity-senior combo | Correlation view without full spread exposure | Systemic delta; gamma profile drift |

**Decision criteria:**

| If you expect... | Position |
|------------------|----------|
| Increased dispersion (idiosyncratic moves) | Long equity protection (positive idiosyncratic gamma) |
| Systemic widening | Short equity protection (positive systemic gamma) |
| Correlation view without full spread risk | Combine equity and senior |

### The Master Decision Tree

```
START: What is your primary view?

├─ View on FUNDING / CASH-SYNTHETIC relationship
│   └─ Use Strategy A1 (Bond-CDS Basis)
│
├─ View on SINGLE NAME vs MARKET
│   └─ Use Strategy A2 (Proxy Hedge)
│
├─ View on CDS TERM STRUCTURE / CURVE SHAPE
│   └─ Use Strategy A3 (Curve Steepener/Flattener)
│       └─ Caution: JTD mismatch from unequal notionals
│
├─ View on TIME VALUE / CARRY
│   ├─ Holding period P&L → Strategy B1
│   └─ Roll timing → Strategy B2
│
├─ View on CORRELATION / TAIL
│   ├─ Correlation increase → Short equity / Long senior
│   ├─ Correlation decrease → Long equity / Short senior
│   └─ Dispersion view → Gamma-driven tranche trade (see Chapter 51)
│
├─ View on CAPITAL STRUCTURE / SENIORITY
│   ├─ Senior vs sub mispricing → Strategy D1 (Senior-Sub CDS)
│   └─ Equity-credit divergence → Strategy D2 (Equity-CDS RV)
│       └─ LBO / leverage event → Overweight CDS leg
│
└─ View on DEFAULTS
    ├─ Single-name default → CDS hedge (JTD-matched)
    └─ Clustered defaults → Long equity tranche protection
```

---

## 52.7 Strategy Family D: Capital Structure Arbitrage

This section is where “trading strategies” touch legal/documentation reality. If terms like restructuring clauses, deliverables, and auction final price are fuzzy, Chapters 39–40 are worth reading before you size anything.

Capital structure arbitrage exploits mispricings between different levels of a firm's debt (senior vs subordinated) or between debt and equity. These strategies draw on the structural relationship between seniority, recovery, and spread—and on the Merton model's insight that equity and debt are both claims on the same underlying asset value.

### 52.7.1 Recovery Rates by Seniority

Empirical studies summarized by O'Kane (sourced to Altman et al.) emphasize three desk-relevant facts:

| # | Fact | Why it matters |
|---|------|----------------|
| 1 | **Seniority matters** | Senior claims recover more on average than subordinated claims |
| 2 | **Recoveries are noisy** | Dispersion of recoveries is wide — "one number" recoveries are dangerous in stress tests |
| 3 | **Absolute priority is not guaranteed** | Realized recoveries do not always obey strict absolute-priority ordering |

**Indicative mean recoveries** (per the dataset summarized in the book):

| Claim type | Mean recovery |
|------------|---------------|
| Senior secured loans | ~high 60%s |
| Senior unsecured bonds | ~mid 30%s |

> **Desk Reality.** Differences in expected recovery across the capital structure translate into large spread differences in distressed regimes. Assuming the same recovery for senior and sub debt → mis-sized hedges and understated jump/scenario P&L.

### 52.7.2 The Senior-Subordinated Spread Relationship

For a single hazard rate $\lambda$ shared across the capital structure (different LGDs):

$$S_{\text{sub}} = \lambda(1 - R_{\text{sub}}) \quad \text{and} \quad S_{\text{sen}} = \lambda(1 - R_{\text{sen}})$$

Dividing eliminates $\lambda$:

$$\boxed{\frac{S_{\text{sub}}}{S_{\text{sen}}} = \frac{1 - R_{\text{sub}}}{1 - R_{\text{sen}}}}$$

This ratio is a **consistency check**, not a guarantee. Market technicals and contract specifics push observed spreads away from the simple relationship. The main use is sanity-checking: spreads across seniority should be broadly aligned with expected recovery differences.

**Worked example.** Senior unsecured recovery 40%, sub recovery 20%:

$$\frac{S_{\text{sub}}}{S_{\text{sen}}} = \frac{1 - 0.20}{1 - 0.40} = \frac{0.80}{0.60} \approx 1.333$$

If senior CDS is 100 bp, sub CDS should be ~133 bp. Material deviations signal RV opportunities — or hidden market technicals.

### 52.7.3 The Merton Model: Equity-Credit Connection

The Merton model is the foundational structural model linking equity and debt. Setup:

| Item | Definition |
|------|------------|
| Face value $F$ | $T$-maturity zero-coupon bonds with total value $D$ |
| Equity $E$ | Shares paying no dividends |
| Asset value | $A(t) = D(t) + E(t)$ |

#### Maturity Payoffs

$$E(T) = \max[A(T) - F, 0] \quad \text{(equity = call option on firm assets)}$$

$$D(T) = F - \max[F - A(T), 0] \quad \text{(debt = cash minus put on assets)}$$

Equity has the same payoff as a **call option** on firm assets with strike equal to the face value of debt due at $T$.

#### Black-Scholes Form

With asset volatility $\sigma_A$:

$$E(t) = A(t)\Phi(d_1) - Fe^{-r(T-t)}\Phi(d_2)$$

(Standard Black-Scholes $d_1, d_2$.)

#### Equity vs Asset Volatility

$$\boxed{\sigma_E = \sigma_A \\, \Phi(d_1) \\, \frac{A(t)}{E(t)}}$$

This identity can be **inverted** to back out an implied asset volatility from observed equity volatility and leverage — one of the model's main practical uses.

#### Limitations

| Limitation | Why it matters |
|------------|----------------|
| Capital structure simplified | Unrealistic for real firms |
| Default at single time $T$ | Real defaults can occur any time |
| Asset values opaque | Hard to estimate $A(t)$ in practice |
| Short-dated spreads understated | Model pushes spreads → 0 as $T - t \to 0$ when assets $\gg$ debt |

Despite these limits, the Merton intuition connects to correlation modeling: in one-factor copula models, defaults are driven by a latent "asset" variable in the same spirit.

> **Desk note (intuition, not a pricing formula).** The Merton linkage $\sigma_E = \sigma_A \\, \Phi(d_1) \\, A/E$ implies that as $E$ falls (leverage rises), equity vol can rise even with $\sigma_A$ unchanged. In the same framing, higher leverage → wider credit spreads. In practice, mapping equity vol to CDS levels is noisy and model-dependent — but equity-credit divergences are a common starting point for capital-structure RV.

### Strategy Card: D1 — Senior vs Subordinated CDS

> **In desk language:** D1 is “same firm, different place in the capital structure.” You are effectively trading **relative recovery / relative protection value** (with market technicals on top).
>
> - The simple spread-ratio formula is a *consistency check*, not a guarantee. Real trades live and die on documentation, deliverables, liquidity, and the recovery distribution.
> - If you want background: Chapter 39 (documentation and restructuring) and Chapter 40 (auction/final price) are the operational prerequisites.

##### 1. Objective

Exploit deviations of the senior-sub spread ratio from its theoretical value, or express a view on relative recovery rates across the capital structure.

##### 2. Instruments and Position Construction

| Direction | Position | Bet |
|-----------|----------|-----|
| **Sub widens vs senior** | Long sub protection + short senior protection | Ratio rises toward theoretical fair value |
| **Sub tightens vs senior** | Short sub protection + long senior protection | Ratio compresses |

Size for CS01-neutrality *or* for a specific spread-ratio view.

##### 3. Exposure Decomposition

| Exposure | Senior leg | Sub leg | Net |
|----------|-----------|---------|-----|
| $CS01$ (USD/bp) | $\lt 0$ (short prot) | $\gt 0$ (long prot) | Can match or carry intentional residual |
| JTD (USD) | negative | positive | Recovery-dependent residual |
| Rec01 (USD/1%) | senior recovery | sub recovery | **Key residual** — different recovery drives P&L |
| Default payoff | pays $(1 - R_{\text{sen}})$ | receives $(1 - R_{\text{sub}})$ | Net gain if $R_{\text{sub}} \lt R_{\text{sen}}$ (typical) |

##### 4. Hedge Set and Hedge Ratios

**CS01-neutral sizing:**

$$\boxed{N_{\text{sen}} = N_{\text{sub}} \times \frac{RPV01_{\text{sub}}}{RPV01_{\text{sen}}}}$$

**Ratio-neutral sizing** (maintain the theoretical spread ratio):

$$\boxed{\frac{N_{\text{sub}} \cdot S_{\text{sub}}}{N_{\text{sen}} \cdot S_{\text{sen}}} = \frac{1 - R_{\text{sub}}}{1 - R_{\text{sen}}}}$$

##### 5. Scenario Test Suite

| # | Scenario | What it tests |
|---|----------|---------------|
| 1 | Parallel spread move (both widen together) | CS01 match |
| 2 | Ratio compression (sub tightens vs senior) | Direction of view |
| 3 | Ratio expansion (sub widens vs senior) | Direction of view |
| 4 | Default event with different recovery rates | JTD residual |
| 5 | Recovery surprise (realized $\neq$ assumed) | Rec01 dominance |

##### 6. Failure Modes

| Failure | Why it happens |
|---------|----------------|
| **Recovery uncertainty dominates** | JTD depends on $R_{\text{sen}} - R_{\text{sub}}$. If APR is violated and recoveries equalize at 20%, the structural edge disappears. |
| Liquidity asymmetry | Sub CDS often much less liquid than senior; execution costs can exceed theoretical edge |
| Market technicals | Supply/demand can hold the ratio away from the recovery-implied level for long periods |

##### 7. Implementation Checklist

- [ ] Verify both CDS reference the same entity with aligned maturities.
- [ ] Use appropriate recovery assumptions for each seniority level.
- [ ] Check the spread ratio against history — is it wide or tight relative to the theoretical level?
- [ ] Default scenario with recovery sweep ($R$ from 10% to 60%) on both legs.

---

### Strategy Card: D2 — Equity-CDS Relative Value (Merton-Inspired)

> **In desk language:** D2 is a cross-asset relative value trade: “is equity pricing a different credit story than CDS?” You are trying to isolate that disagreement.
>
> - The hard part is that the hedge is imperfect: equity brings delta/vega and discrete corporate actions; CDS brings default/settlement mechanics.
> - If you are new to this, read Section 52.7.3 for the structural intuition and then go straight to Example 19 (LBO toy) to see why sizing is nontrivial.

##### 1. Objective

Exploit divergences between **equity-implied** credit risk and **CDS-priced** credit risk, using the Merton-model insight that both are functions of the same underlying asset value and volatility.

##### 2. Instruments and Position Construction

| Leg | Role |
|-----|------|
| **CDS leg** | Single-name CDS (long or short protection) |
| **Equity leg** | Equity options or shares used to hedge the equity-implied credit component |

The trade exploits a logical inconsistency: when equity volatility implies one default probability and CDS spreads imply another, one market may be mispriced.

> **Desk note — LBO / leverage events.** An LBO is a discrete capital-structure shock. It can change leverage, covenant package, and expected recoveries — both senior and subordinated CDS can reprice sharply. Common expressions:
> - **Outright credit-widening** (buy protection).
> - **Relative senior vs sub** view.
>
> Direction and sizing depend on deal structure and which obligations each CDS contract references. Senior vs sub recovery in LBOs is documentation-dependent (guarantees, obligation characteristics, post-deal capital structure). Confirm contract terms, deliverables, and recovery assumptions in your risk system before sizing.

##### 3. Exposure Decomposition

| Exposure | CDS leg | Equity leg | Net |
|----------|---------|------------|-----|
| Credit spread sensitivity | CS01 | equity-implied CS01 | Residual |
| Equity delta | $\approx 0$ | dominant | Equity exposure |
| Equity vega | indirect (Merton link) | dominant | Vol exposure |
| JTD | CDS default payoff | equity goes to ~0 | **Significant residual** |

##### 4. Failure Modes

| Failure | Why it happens |
|---------|----------------|
| **Model risk** | Merton is highly simplified — unrealistic capital structure, single default time, opaque asset values. Any Merton-implied fair value inherits these limitations. |
| Equity-credit decorrelation | Equity and CDS can diverge for reasons the model doesn't capture (buybacks, dividend changes, sector rotation) |
| Execution complexity | Running a CDS-equity hedge spans two asset classes with different settlement, margin, and liquidity profiles |

---

## 52.8 Loan CDS vs Standard CDS

**Loan CDS (LCDS)** references loans rather than bonds. The key economic difference is **recovery**: secured loans have historically recovered more than senior unsecured bonds, so the same default intensity implies a *tighter* fair spread for LCDS.

#### Recovery-Driven Spread Ratio

Indicative mean recoveries (Altman et al. dataset):

| Claim | Recovery $R$ |
|-------|--------------|
| Senior secured loan | ~68.5% |
| Senior unsecured bond | ~34.89% |

Under an equal-hazard-rate assumption (spread scales like $1-R$):

$$\frac{S_{\text{CDS}}}{S_{\text{LCDS}}} = \frac{1 - R_{\text{bond}}}{1 - R_{\text{loan}}} = \frac{0.65}{0.315} \approx 2.06$$

**Interpretation:** higher expected recovery on loans → lower LGD → lower fair spread, all else equal.

#### Caveats and Contract Features

| Feature | Implication |
|---------|-------------|
| Deliverable definitions | Loan-specific deliverability rules differ from bond CDS |
| Cancellation / refinancing (in some LCDS forms) | The contract can be cancelled if the underlying loan is refinanced |
| Trigger definitions | Vary across products and regions |

For **cancellable LCDS**, valuation requires both:
- a default survival curve $Q_D$, and
- a cancellation curve $Q_C$.

> **Operational note.** LCDS conventions and liquidity are contract- and time-dependent. Before trading or marking LCDS, confirm:
> - Trigger definitions
> - Any cancellation/refinancing feature
> - Quoting regime (spread vs coupon + upfront)
> - Execution liquidity for the specific name/index

---

## 52.9 Crisis Behavior of Credit Strategies

Strategies in this chapter are designed for normal market conditions. In systemic stress, the assumptions underlying hedging relationships break down. The "perfect storm" — multiple adverse moves coinciding — is especially relevant here.

### 52.9.1 Basis Strategies in Crisis

Basis trades are vulnerable because multiple drivers can move adversely *together*:

| # | Stress mechanism | Effect |
|---|-------------------|--------|
| 1 | **Funding asymmetry dominates** | Bond leg must be financed; CDS is unfunded. Tightening repo makes the cash leg expensive to carry. |
| 2 | **CDS becomes the "fast" short** | Easier to short credit via CDS than via cash bonds → protection-buying flows push CDS wider faster than cash spreads. |
| 3 | **Technical flows can reverse** | Dealer hedging of structured-credit flows (e.g., synthetic CDO issuance and unwind) can mechanically tighten or widen CDS spreads vs cash. |

**Stress-test takeaway:** a basis package that looks low-risk under small moves can suffer large, *discontinuous* losses when funding and liquidity regimes shift.

### 52.9.2 Correlation and Tranche Trades in Crisis

Published tranche-risk examples highlight that equity tranches are highly leveraged and often carry **negative systemic gamma**. In stress:

| Trigger | Tranche behavior |
|---------|------------------|
| Correlation spike | Defaults cluster → losses concentrate in equity and mezzanine |
| Large parallel widening | Negative systemic gamma *amplifies* equity tranche losses beyond the linear delta |
| Realized clustered defaults | Senior tranches gain initially (positive systemic gamma) but suffer **sudden, severe losses** if attachment points are breached |

Correlation hedges are inherently model-dependent and least reliable precisely when tail risk dominates. **Treat correlation and clustering as scenario-driven risks**, not a single Corr01 number.

### 52.9.3 Early Warning Indicators

Useful monitoring signals (thresholds are desk-specific):

| Signal | What to watch |
|--------|---------------|
| CDS–bond basis | Level + rate of change |
| Index quoted vs intrinsic basis | Including any portfolio-swap-adjustment changes |
| Implied correlation / tranche marks | Plus Corr01 P&L vs carry/theta |
| Funding terms (cash leg) | Repo availability/terms; margin requirements |
| Bid/offer + market depth | Liquidity regime |

Thresholds and escalation playbooks are desk policy (risk appetite, governance, market regime). Use the signal list as generic *categories* — plug in desk-specific thresholds and escalation actions.

---

## 52.10 Execution and Position Management

Execution and position management are where model hedges meet market microstructure. Details are desk-specific; the general risk-first discipline is consistent with the book's stress-testing emphasis.

### 52.10.1 Position Sizing Considerations

For any strategy in this chapter, position size should reflect three checks:

| # | Check | Question |
|---|-------|----------|
| 1 | **Risk-budget allocation** | Is max loss tolerance expressed in JTD and CS01 — not just notional? |
| 2 | **Carry-to-risk trade-off** | Will plausible stress losses + funding/liquidity shocks fit within desk risk limits? |
| 3 | **Liquidity-adjusted sizing** | Does expected edge dominate round-trip costs + expected slippage (especially for instruments that gap in stress)? |

### 52.10.2 Monitoring Triggers

Possible trigger categories (thresholds are desk-specific):

| Trigger | What it indicates | Action |
|---------|-------------------|--------|
| **Hedge drift** | CS01 / bucketed CS01 no longer near target | Re-hedge or accept the residual |
| **Funding regime change** | Cash-leg financing materially different from strategy's carry assumption | Re-cost the position |
| **Recovery / settlement assumption change** | Recovery / final-price inputs moved materially | Re-run JTD/VOD scenarios |
| **Correlation / tail regime change** | Tranche marks inconsistent with hedge map | Re-run clustered-default scenarios |

Exact numeric thresholds and escalation actions are set by desk playbooks — this framework provides the *categories*, not the numbers.

### 52.10.3 Unwind Playbook

When conditions warrant reducing or exiting a position:

1. **Unwind the most liquid leg first** — typically CDS over bonds; index over single-name.
2. **Do not unwind CS01-hedged positions one leg at a time** — that creates naked exposure. Unwind in **matched pairs** where possible.
3. **In illiquid markets, consider partial unwinds** — reducing by 50% captures some value while maintaining optionality.
4. **Document the unwind rationale** — risk-limit breach, P&L stop, or fundamental view change? This matters for post-trade analysis.

---

## 52.11 Mathematical Foundations

This section is here so you can sanity-check what a risk system is doing under the hood. If you do not like calculus, skip to Section 52.12 — the worked examples are the real learning tool. For full CDS risk-measure derivations (CS01/RPV01, VOD/JTD, recovery risk, theta), Chapter 43 is the canonical reference.

### 52.11.1 CS01 from the CDS MTM Identity

**Starting point** (per unit notional):

$$V(t,T) = (S(t,T) - S_0) \cdot RPV01(t,T)$$

**Differentiate w.r.t. market spread $S$:**

$$\frac{\partial V}{\partial S} = RPV01(t,T)$$

**Result:**

$$\boxed{CS01 = \frac{\partial PV}{\partial S} \times 10^{-4} = N \cdot RPV01(t,T) \cdot 10^{-4}}$$

**Unit check:** $RPV01$ has units "years" (PV of 1 bp/yr premium). Multiply by $10^{-4}$ (per bp) and notional $N$ (USD) $\Rightarrow$ USD/bp. ✓

---

### 52.11.2 CS01-Based Hedge Ratio Derivation

Let the target position have $CS01_T$ and the hedge instrument have $CS01_H$, both sign-inclusive (USD/bp).

**Setup the neutrality condition:**

$$CS01_{\text{net}} = CS01_T + N_H \cdot CS01_H^{(USD 1)} = 0$$

**Solve for hedge notional:**

$$\boxed{N_H = -\frac{CS01_T}{CS01_H^{(USD 1)}}}$$

**Variant** — if both CS01s are already quoted in USD/bp at *stated* notionals:

$$N_H = -\frac{CS01_T}{CS01_H} \times N_H^{\text{current}}$$

---

### 52.11.3 JTD (VOD) Mapping for CDS

**Source-backed VOD expression** (per unit notional):

$$VOD = (1 - R - \text{Accrued}) - (S - S_0) \cdot RPV01$$

**At par** ($S = S_0$):

$$VOD \approx 1 - R - \text{Accrued}$$

**Dollar JTD:**

$$\boxed{JTD = N \cdot VOD}$$

**Unit check:** $1 - R$ and accrued are unitless fractions of notional; multiplying by $N$ (USD) gives USD. ✓

---

### 52.11.4 P&L Explain Template (First-Order + Event Terms)

For a portfolio, approximate horizon P&L as:

$$\boxed{\Delta PV \approx \sum_i (CS01_i \cdot \Delta S_i) + (Rec01 \cdot \Delta R) + JTD_{\text{events}} + BasisTerm + Residual}$$

| Term | Meaning |
|------|---------|
| $\sum_i CS01_i \cdot \Delta S_i$ | Linear spread moves (bucketed by maturity or instrument) |
| $Rec01 \cdot \Delta R$ | Recovery-assumption change |
| $JTD_{\text{events}}$ | Discrete defaults and settlement mechanics |
| $BasisTerm$ | Quoted–intrinsic or CDS–cash basis change |
| $Residual$ | Convexity, model error, liquidity/execution slippage |

---

### 52.11.5 Carry/Rolldown Decomposition

The goal is to separate three sources of P&L over $[t, t + \Delta t]$:

1. **Carry** — accrual / coupon / premium cashflows.
2. **Rolldown** — curve aging holding the spread curve fixed.
3. **Spread move** — what the curve actually did.

**Construction.** Let $PV(t; S_t)$ be PV at time $t$ using today's spread curve $S_t(\cdot)$. Define a *rolled* PV at $t + \Delta t$ holding the curve fixed but shortening maturity:

$$PV_{\text{roll}}(t + \Delta t; S_t)$$

**Decomposition:**

$$\Delta PV = \underbrace{\text{Cashflows over } [t, t + \Delta t]}_{\text{carry}} + \underbrace{(PV_{\text{roll}}(t + \Delta t; S_t) - PV(t; S_t))}_{\text{rolldown / theta}} + \underbrace{(PV(t + \Delta t; S_{t + \Delta t}) - PV_{\text{roll}}(t + \Delta t; S_t))}_{\text{spread move}} + \text{events}$$

> **Implementation note.** An exact closed-form carry formula depends on quoting regime (par spread vs fixed coupon + upfront), day count, clean vs full price, and the desk's carry definition. Use the relevant index rulebook conventions and your desk's P&L attribution definition.

---

## 52.12 Worked Examples

**Reminder:** all examples use toy inputs; magnitudes are chosen to look plausible but are not market data.

**How to read these (recommended):**

1. Track **units** aggressively (USD/bp vs USD vs USD per 1% recovery).
2. Do a quick **sign check** (long protection CS01 should be positive; long bonds lose on spread widening).
3. Separate **small-move hedging** (CS01/DV01) from **event risk** (JTD/recovery/final price) and from **regime risk** (basis, correlation).
4. If a step feels unfamiliar, jump back to the prerequisite chapter: CDS risk measures (Chapter 43), indices and basis (Chapters 45–47), or tranches (Chapters 49–51).

---

### Example 1: Bond–CDS Basis Toy

**Goal:** size the CDS leg to hedge JTD on a long-bond position, then show that basis risk still produces P&L when the legs move differently.

#### Setup

| Item | Value |
|------|-------|
| Bond face $N_B$ | USD 1,000,000 |
| Bond full price $P$ | 101 (per 100) → market value USD 1,010,000 |
| Recovery $R$ | 40% → recovery value USD 400,000 |

**Bond JTD** (loss to a long bond on default):

$$JTD_{\text{bond}} = -(P - R) \times N_B = -(1.01 - 0.40) \times 1{,}000{,}000 = -USD 610{,}000$$

This loss-on-default mismatch ($P-R$ for the bond vs $1-R$ for CDS) is a key basis driver in the sources.

#### Step 1 — Size CDS Notional to Match Bond JTD

Buy protection at par: $S = S_0 = 150$ bp, so pre-default MTM $\approx 0$.

Accrued at default (assume halfway through the quarter, $\Delta = 0.125$):

$$\text{Accrued} = N_{\text{CDS}} \times 0.015 \times 0.125 = 0.001875 \\, N_{\text{CDS}}$$

CDS JTD at par (VOD):

$$JTD_{\text{CDS}} \approx N_{\text{CDS}} \times (1 - R) - \text{Accrued} = 0.598125 \\, N_{\text{CDS}}$$

Solve $0.598125 \\, N_{\text{CDS}} = USD 610{,}000$:

$$\boxed{N_{\text{CDS}} = \frac{610{,}000}{0.598125} \approx USD 1{,}019{,}854 \approx USD 1.02\text{mm}}$$

#### Step 2 — Compute Spread Sensitivities

Assume $RPV01_{\text{CDS}} = 4.5$ and (toy) bond credit $CS01_{\text{bond}} = -USD 420/\text{bp}$.

$$CS01_{\text{CDS}} = 1{,}019{,}854 \times 4.5 \times 10^{-4} = USD 458.93/\text{bp}$$

#### Step 3 — Stress: Basis Widens

CDS spread $+20$ bp; bond spread $+5$ bp (basis widens by 15 bp).

| Leg | Move | P&L |
|-----|------|-----|
| Bond | $+5$ bp | $-420 \times 5 = -USD 2{,}100$ |
| CDS (long protection) | $+20$ bp | $+458.93 \times 20 = +USD 9{,}179$ |
| **Net** | | **$+USD 7{,}079$** |

#### Interpretation

JTD is hedged *by construction*, but the package still has basis risk: when CDS and bond spreads move by different amounts, residual P&L is generated. **CS01 was not matched** in this example — only JTD was — so the residual is primarily basis exposure.

---

### Example 1B: Basis Package Under Funding Stress

**Goal:** show that funding shocks can dominate a "hedged" basis package, even when spread legs are well-matched.

#### Continuing Setup (from Example 1)

| Item | Value |
|------|-------|
| Bond face | USD 1mm at clean price 101 |
| CDS notional | ~USD 1.02mm (JTD-matched) |
| CS01 net | +USD 38.93/bp (unhedged basis exposure) |

#### Funding Shock

The bond is financed via repo at Libor + 25 bp. During a crisis, repo on this name spikes to Libor + 150 bp (a 125 bp increase). Term: 3 months ($0.25$y).

$$\text{Additional funding cost} = 1{,}010{,}000 \times 0.0125 \times 0.25 = USD 3{,}156.25$$

#### Two Scenarios

**Scenario 1 — Basis widens (Example 1 stress) plus funding shock:**

| Component | P&L |
|-----------|-----|
| CS01 P&L (from Example 1) | $+USD 7{,}079$ |
| Funding shock | $-USD 3{,}156$ |
| **Net** | **$+USD 3{,}923$** |

The funding shock consumed about 45% of the basis profit.

**Scenario 2 — Basis flat, funding spikes** (small parallel spread move; CS01 net of $+USD 38.93$/bp produces only minor spread P&L compared to the funding shock):

| Component | P&L |
|-----------|-----|
| Spread P&L (small parallel move) | $\approx 0$ |
| Funding shock | $-USD 3{,}156$ |
| **Net** | **$-USD 3{,}156$ loss** |

#### Lesson

The bond leg's financing is a **first-class risk**, not a footnote. CDS does not require financing the bond principal, so even a well-hedged CS01/JTD package leaves a one-sided exposure to repo/funding regime shifts.

---

### Example 2: DV01-Neutral but Not Credit-Neutral

**Goal:** show that hedging a corporate bond's rates DV01 with Treasuries leaves the credit-spread risk wide open.

#### Setup

| Item | Value |
|------|-------|
| Corporate bond notional $N_C$ | USD 10mm |
| Bond rates DV01 ($+1$ bp rates move) | $-USD 7{,}500$/bp |
| Treasury DV01 per USD 1mm (long) | $-USD 850$/bp |
| Treasury DV01 per USD 1mm (short) | $+USD 850$/bp |

#### Step 1 — Compute the DV01 Hedge

Need $-7{,}500 + 850 \times N_T = 0$ (with $N_T$ in USD-1mm units):

$$N_T = \frac{7{,}500}{850} \approx 8.8235\text{mm}$$

**Hedge: short USD 8.8235mm Treasuries.**

#### Step 2 — Stress: Rates and Credit Both Move

| Move | Magnitude |
|------|-----------|
| Parallel rates | $+10$ bp |
| Corporate credit spread | $+50$ bp |
| Treasury credit spread | unchanged |

#### Step 3 — Compute P&L

**Rates leg:**

| Position | P&L |
|----------|-----|
| Corporate bond | $-7{,}500 \times 10 = -USD 75{,}000$ |
| Short Treasury | $+(850 \times 8.8235) \times 10 = +USD 75{,}000$ |
| **Net rates** | **$\approx 0$** |

**Credit leg:** with corporate CS01 (toy) $= -6{,}500$ USD/bp on USD 10mm:

$$\Delta PV_{\text{credit}} = -6{,}500 \times 50 = -USD 325{,}000$$

#### Conclusion

**DV01-neutral does not mean risk-neutral.** The credit-spread leg dominates: $-USD 325{,}000$ loss despite a perfect rates hedge. Hedging rates is necessary but not sufficient for a credit position.

---

### Example 3: Single-Name Hedged with Index by CS01 Matching

**Goal:** show what a CS01-matched proxy hedge does and does not protect against.

#### Setup

| Leg | Notional | RPV01 | CS01 |
|-----|----------|-------|------|
| Single-name (long protection) | $N_s = USD 10$mm | 4.5 | $+USD 4{,}500$/bp |
| Index (per USD 10mm, long protection) | base unit USD 10mm | 4.2 | $+USD 4{,}200$/bp |

Hedge: **short protection on the index.**

#### Step 1 — Compute Hedge Notional

Solve $CS01_s + CS01_I^{(\text{short})} = 0$:

$$4{,}500 - 4{,}200 \times \frac{N_I}{10\text{mm}} = 0 \\;\Rightarrow\\; \boxed{N_I = 10\text{mm} \times \frac{4{,}500}{4{,}200} \approx USD 10.7143\text{mm}}$$

#### Step 2 — Stress: Broad vs Idiosyncratic Moves

**Scenario A — Broad market move (+10 bp on both):**

| Leg | P&L |
|-----|-----|
| Single-name long | $+4{,}500 \times 10 = +USD 45{,}000$ |
| Index short | $-4{,}200 \times 1.07143 \times 10 = -USD 45{,}000$ |
| **Net** | **$\approx 0$** ✅ |

**Scenario B — Idiosyncratic widening (single-name +50 bp, index +10 bp):**

| Leg | P&L |
|-----|-----|
| Single-name long | $+4{,}500 \times 50 = +USD 225{,}000$ |
| Index short | $-USD 45{,}000$ |
| **Net** | **$+USD 180{,}000$** (residual idiosyncratic risk) |

#### JTD Warning

If the single name **defaults**, the single-name JTD is large (the full $1-R$ payout). The index hedge only absorbs the small loss on that one name inside the diversified index. **CS01 matching does not hedge JTD** — this is the central limitation of proxy hedging.

---

### Example 4: Index Hedged with Constituents (Bottom-Up) Using RPV01/CS01 Weights

**Goal:** show how a CS01-matched bottom-up hedge of an index using its constituents works for parallel moves but breaks under dispersion.

> **Caveat:** there is no single canonical bottom-up hedge formula without specifying the portfolio swap adjustment rule and the risk system's definition of "intrinsic" vs "quoted" objects. Treat this example as a CS01-matching *diagnostic*, not a production hedge recipe.

#### Setup (Toy Index, $M = 3$ Names)

| Item | Value |
|------|-------|
| Index notional $N_I$ | USD 30mm → USD 10mm per name |
| Maturity | 5Y (all aligned) |
| Index RPV01 (≈ average) | 4.6 |

| Constituent | RPV01 |
|-------------|-------|
| Name A | 4.5 |
| Name B | 4.7 |
| Name C | 4.6 |

#### Step 1 — Compute Index CS01

$$CS01_I = 30{,}000{,}000 \times 4.6 \times 10^{-4} = USD 13{,}800/\text{bp}$$

#### Step 2 — Build Constituent Hedge (Short Protection per Name, USD 10mm Each)

| Name | Calc | CS01 contribution |
|------|------|-------------------|
| A | $-10{,}000{,}000 \times 4.5 \times 10^{-4}$ | $-USD 4{,}500$/bp |
| B | $-10{,}000{,}000 \times 4.7 \times 10^{-4}$ | $-USD 4{,}700$/bp |
| C | $-10{,}000{,}000 \times 4.6 \times 10^{-4}$ | $-USD 4{,}600$/bp |
| **Total** | | $-USD 13{,}800$/bp ✅ offsets index |

#### Step 3 — Stress

**Parallel +10 bp move (all names + index together):**

| Leg | P&L |
|-----|-----|
| Index long | $+13{,}800 \times 10 = +USD 138{,}000$ |
| Constituent shorts | $-USD 138{,}000$ |
| **Net** | **$\approx 0$** ✅ |

**Dispersion:** if one constituent widens much more than the index — or the quoted-vs-intrinsic basis shifts — the hedge breaks. This is exactly *why* index basis matters operationally.

---

### Example 5: Index Intrinsic vs Quoted Basis

**Goal:** compute the intrinsic spread from constituents, derive the index basis, and show how a basis-only move generates P&L.

#### Setup

| Item | Value |
|------|-------|
| Number of names $M$ | 3 |
| Index notional $N_I$ | USD 10mm |
| Coupon $C$ | 100 bp |
| Quoted index spread $S_I$ | 110 bp |
| Index RPV01 | 4.6 |

| Constituent | 5Y spread (bp) | RPV01 |
|-------------|----------------|-------|
| Name 1 | 120 | 4.5 |
| Name 2 | 90 | 4.7 |
| Name 3 | 110 | 4.6 |

#### Step 1 — Intrinsic Spread (RPV01-Weighted Average)

$$S_{\text{intr}} = \frac{\sum_m RPV01_m \\, S_m}{\sum_m RPV01_m} = \frac{4.5(120) + 4.7(90) + 4.6(110)}{4.5 + 4.7 + 4.6} \approx 106.4493 \text{ bp}$$

#### Step 2 — Index Basis (Spread)

$$\text{Basis} = S_I - S_{\text{intr}} = 110 - 106.4493 \approx 3.5507 \text{ bp}$$

#### Step 3 — PV View (Upfront Difference)

**Market upfront** at coupon $C$:

$$U_{\text{mkt}} = N_I \cdot RPV01_I \cdot (S_I - C) \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.6 \cdot 10 \cdot 10^{-4} = USD 46{,}000$$

**Intrinsic upfront** (equal-weight notional $N_I / M = USD 3.3333$mm per name):

| Name | $S_m - C$ | PV calc | PV |
|------|-----------|---------|-----|
| 1 | $+20$ bp | $3.3333\text{mm} \cdot 4.5 \cdot 20 \cdot 10^{-4}$ | $+USD 30{,}000$ |
| 2 | $-10$ bp | $3.3333\text{mm} \cdot 4.7 \cdot (-10) \cdot 10^{-4}$ | $-USD 15{,}667$ |
| 3 | $+10$ bp | $3.3333\text{mm} \cdot 4.6 \cdot 10 \cdot 10^{-4}$ | $+USD 15{,}333$ |
| **Sum $V_{\text{intr}}$** | | | $USD 29{,}667$ |

**Basis PV:**

$$U_{\text{mkt}} - V_{\text{intr}} = 46{,}000 - 29{,}667 = USD 16{,}333$$

which equals (cross-check):

$$N_I \cdot RPV01_I \cdot \text{Basis} \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.6 \cdot 3.5507 \cdot 10^{-4} = USD 16{,}333 \\;\checkmark$$

#### Step 4 — Basis-Only P&L Scenario

Suppose the basis decreases by 2 bp because the *quoted* index tightens (constituents unchanged):

| Leg | P&L |
|-----|-----|
| Long index protection ($CS01_I = USD 4{,}600$/bp) | $4{,}600 \times (-2) = -USD 9{,}200$ |
| Constituent legs (unchanged) | $\approx 0$ |
| **Net** | $-USD 9{,}200$ |

**Interpretation:** basis moves alone (no change in fundamental constituent spreads) can dominate short-horizon hedges. This is why basis monitoring is non-optional for index-vs-constituent strategies.

---

### Example 6: Index Carry

**Goal:** show that even with unchanged spreads, a long-protection index position has negative carry — it bleeds the coupon plus the upfront's RPV01-driven decay.

#### Setup (Long Protection on Index)

| Item | Value |
|------|-------|
| Notional $N$ | USD 10mm |
| Coupon $C$ | 100 bp |
| Market spread at inception $S$ | 120 bp |
| RPV01 at $t_0$ | 4.6 |
| RPV01 at $t_1$ (one quarter later) | 4.4 |
| Quarter fraction $\Delta$ | 0.25 |

Since $S \gt C$, the protection buyer pays upfront at inception.

#### Step 1 — Upfront Paid at Inception

$$U_0 = N \cdot RPV01(t_0) \cdot (S - C) \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.6 \cdot 20 \cdot 10^{-4} = USD 92{,}000$$

#### Step 2 — Coupon Cashflow over the Quarter

$$\text{Coupon CF} = -N \cdot C \cdot 10^{-4} \cdot \Delta = -10{,}000{,}000 \cdot 100 \cdot 10^{-4} \cdot 0.25 = -USD 25{,}000$$

(Negative because the long-protection buyer *pays* the coupon.)

#### Step 3 — MTM at $t_1$ (Spreads Unchanged)

$$V(t_1) = N \cdot RPV01(t_1) \cdot (S - C) \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.4 \cdot 20 \cdot 10^{-4} = USD 88{,}000$$

Clean value relative to inception (we paid $USD 92{,}000$ upfront):

$$\Delta V_{\text{clean}} = 88{,}000 - 92{,}000 = -USD 4{,}000$$

#### Step 4 — Quarter Carry P&L (Unchanged Spreads)

| Component | P&L |
|-----------|-----|
| Coupon CF | $-USD 25{,}000$ |
| Upfront amortization ($\Delta V_{\text{clean}}$) | $-USD 4{,}000$ |
| **Total carry P&L** | **$-USD 29{,}000$** |

#### Interpretation

Even with spreads frozen, long protection bleeds:
- The **coupon** drains cash quarterly.
- The **upfront amortizes** as RPV01 falls with shortening maturity.

This is the "negative carry" that any long-protection holder pays for default insurance.

---

### Example 7: Index Rolldown

**Goal:** with the curve held fixed, age the position one month and compute the carry + rolldown P&L. This is the deterministic time-decay piece of P&L.

#### Setup (Continues Example 6 — Long Protection)

| Item | Value |
|------|-------|
| Notional $N$ | USD 10mm |
| Coupon $C$ | 100 bp |
| Upfront paid at inception (Example 6) | $U_0 = USD 92{,}000$ |
| 5Y par spread at $t_0$ | 120 bp |
| Spread curve | Upward sloping |
| Remaining maturity at $t_1$ (1 month later) | ~4.92Y |
| Par spread at $t_1$ (curve fixed → "roll down") | 118 bp |
| $RPV01(t_1)$ | 4.57 (slightly lower as maturity shortens) |
| Monthly fraction $\Delta$ | $1/12$ |

#### Step 1 — MTM at $t_1$ (Curve Held Fixed)

$$V(t_1) = N \cdot 4.57 \cdot (118 - 100) \cdot 10^{-4} = USD 82{,}260$$

#### Step 2 — Clean Value Change vs Inception

$$\Delta V_{\text{clean}} = 82{,}260 - 92{,}000 = -USD 9{,}740$$

#### Step 3 — Coupon Accrual over the Month

$$\text{Coupon accrual} = -N \cdot 100 \cdot 10^{-4} \cdot \frac{1}{12} = -USD 8{,}333$$

#### Step 4 — Total Carry + Rolldown P&L

| Component | P&L |
|-----------|-----|
| Coupon accrual | $-USD 8{,}333$ |
| MTM change (rolldown + RPV01 decay) | $-USD 9{,}740$ |
| **Total** | **$-USD 18{,}073$** |

#### Interpretation

Even with no spread changes, the long-protection holder bled USD 18k in one month — purely from time passing on a fixed curve. This is exactly the carry+rolldown figure carried into Example 10.

---

### Example 8: Roll Trade Mechanics

**Goal:** decompose the cost of switching from off-the-run to on-the-run series, and show why "flat notional" rolls inadvertently increase spread exposure.

Per O'Kane, two effects drive roll P&L: **(1) composition changes** (perceived credit quality), and **(2) maturity extension** on an upward-sloping curve.

#### Setup

| | Old series | New series |
|---|------------|------------|
| Notional $N$ | USD 10mm | USD 10mm |
| Coupon $C$ | 100 bp | 100 bp |
| Quoted spread $S$ | 115 bp | 120 bp |
| RPV01 | 4.1 (~4.5Y maturity) | 4.6 (full 5Y) |

Position: **long protection**.

#### Step 1 — Decompose the 5 bp Spread Difference

| Effect | Calculation | Contribution |
|--------|-------------|--------------|
| Maturity extension | Curve slope $\approx 10$ bp/yr × 0.5 yr | $+5$ bp |
| Composition | Downgraded names removed offset by wider-spread liquidity replacements | $\approx 0$ bp |
| **Total** | | **$+5$ bp** ✅ matches observed |

#### Step 2 — MTM Values (Pre-Upfront)

$$V_{\text{old}} = N \cdot 4.1 \cdot (115 - 100) \cdot 10^{-4} = USD 61{,}500$$
$$V_{\text{new}} = N \cdot 4.6 \cdot (120 - 100) \cdot 10^{-4} = USD 92{,}000$$

#### Step 3 — Cash to Switch (Close Old, Open New)

| Action | Cash flow |
|--------|-----------|
| Close old position | $+USD 61{,}500$ (received) |
| Open new position | $-USD 92{,}000$ (paid) |
| **Net cash outflow** | **$USD 30{,}500$** |

#### Step 4 — CS01-Adjusted Notional for Spread-Neutral Switch

| | Value |
|---|-------|
| $CS01_{\text{old}}$ on USD 10mm | $USD 4{,}100$/bp |
| $CS01_{\text{new}}$ per USD 10mm | $USD 4{,}600$/bp |

To hold CS01 constant at $USD 4{,}100$/bp on the new series:

$$N_{\text{new}} = USD 10\text{mm} \times \frac{4{,}100}{4{,}600} \approx USD 8.913\text{mm}$$

**Implication: reduce notional by ~11%** when rolling to keep spread-neutral exposure. Rolling "flat notional" *increases* spread exposure because the longer-maturity new series has higher CS01 per dollar.

#### Step 5 — Execution Cost (Toy)

Bid/offer equivalent to 0.5 bp on each leg:

| | Calc | Cost |
|---|------|------|
| Per-leg cost | $\approx CS01 \times 0.5$ bp $\approx 4{,}600 \times 0.5$ | $USD 2{,}300$ |
| Two legs | $2 \times 2{,}300$ | $USD 4{,}600$ |

#### All-In Roll Cost

$$30{,}500 + 4{,}600 = USD 35{,}100$$

This is substantial relative to quarterly carry, so the liquidity benefits of being on-the-run must justify the roll cost.

---

### Example 8B: Roll with Favorable Composition

**Goal:** show what happens when a composition improvement dominates the maturity-extension effect — the new series can trade tighter than the old.

#### Setup

Same as Example 8, with one change: 3 distressed names (avg 400 bp) are removed and replaced with 3 IG names (avg 50 bp) in a 125-name index.

#### Step 1 — Composition Effect

$$\Delta S_{\text{composition}} = \frac{3}{125} \times (50 - 400) = -8.4 \text{ bp}$$

#### Step 2 — Net Spread Change

| Effect | Contribution |
|--------|--------------|
| Maturity extension (Example 8) | $+5$ bp |
| Composition | $-8.4$ bp |
| **Net** | **$-3.4$ bp** |

The new series trades **tighter** despite the longer maturity.

#### Step 3 — Roll P&L

| Series | Spread | Calc | MTM |
|--------|--------|------|-----|
| Old | 115 bp | (from Example 8) | $USD 61{,}500$ |
| New | 111.6 bp | $10\text{mm} \cdot 4.6 \cdot (111.6 - 100) \cdot 10^{-4}$ | $USD 53{,}360$ |

**Net cash inflow** (favorable roll for the long-protection holder):

$$61{,}500 - 53{,}360 = USD 8{,}140 \text{ received}$$

#### Key Lesson

Roll composition cuts both ways. The new series may drop downgraded names *or* may pull in wider replacements driven by liquidity criteria. To explain any roll P&L, decompose it into:
1. Maturity / curve effects (deterministic from curve slope).
2. Composition effects (depends on which names enter and exit).

---

### Example 9: Default in Index

**Goal:** trace what happens to a long-protection index position when one constituent defaults — both the immediate cash settlement and the ongoing coupon adjustment.

**Mechanics recap (per source):** the default payout is based on face value minus the auction-determined final price. The defaulted name is removed from the index and the future premium is reduced by the proportional notional ($1/M$ in these notes).

#### Setup

| Item | Value |
|------|-------|
| Total index notional $N$ | USD 10mm |
| Number of names $M$ | 5 → per-name notional $N_m = USD 2$mm |
| Coupon $C$ | 100 bp |
| Final price $FP$ | 35% → $R = 0.35$ |
| Days since last coupon (Actual/360) | 50 → $\Delta = 50/360 \approx 0.1389$ |

#### Step 1 — Immediate Default Cash (Long Protection)

| Item | Calc | Value |
|------|------|-------|
| Protection payment | $(1-R) \cdot N_m = 0.65 \times 2{,}000{,}000$ | $+USD 1{,}300{,}000$ |
| Accrued premium owed | $N_m \cdot C \cdot 10^{-4} \cdot \Delta = 2{,}000{,}000 \cdot 0.01 \cdot 0.1389$ | $-USD 2{,}778$ |
| **Net immediate cash** | | **$+USD 1{,}297{,}222$** |

#### Step 2 — Future Coupon Impact

| | Notional | Quarterly coupon |
|---|----------|------------------|
| Before default | $USD 10$mm | $10\text{mm} \cdot 0.01 \cdot 0.25 = USD 25{,}000$ |
| After default (notional $-1/M = -20\\%$) | $USD 8$mm | $8\text{mm} \cdot 0.01 \cdot 0.25 = USD 20{,}000$ |
| **Reduction** | | $-USD 5{,}000$ per quarter |

The protection buyer pays USD 5,000 less coupon each quarter going forward — a permanent cashflow change tied to the smaller surviving notional.

---

### Example 10: Full P&L Explain for an Index Position over a Horizon

**Goal:** decompose one-month P&L into carry + rolldown + spread move + default event, the canonical desk explain.

#### Setup

| Item | Value |
|------|-------|
| Position | Long protection on index |
| Notional $N$ | USD 10mm |
| Coupon $C$ | 100 bp |
| RPV01 | 4.6 → CS01 = $USD 4{,}600$/bp |
| Horizon | One month |
| Carry + rolldown (from Example 7) | $-USD 18{,}073$ |

#### Step 1 — Spread Move (+10 bp)

$$\Delta PV_{\text{spread}} = 4{,}600 \times 10 = +USD 46{,}000$$

#### Step 2 — One Constituent Defaults (large index, $M = 125$)

| Item | Value |
|------|-------|
| Per-name notional $N_m$ | $10{,}000{,}000 / 125 = USD 80{,}000$ |
| Final price $FP$ | 35% → $1-R = 0.65$ |
| Accrual fraction | $50/360 \approx 0.1389$ |

Default cash:

$$0.65 \times 80{,}000 - 80{,}000 \times 0.01 \times 0.1389 = 52{,}000 - 111.11 \approx USD 51{,}889$$

#### P&L Explain

| Component | P&L |
|-----------|-----|
| Carry + rolldown | $-USD 18{,}073$ |
| Spread move (+10 bp) | $+USD 46{,}000$ |
| Default event | $+USD 51{,}889$ |
| **Total** | **$+USD 79{,}816$** |

#### Interpretation

All three components matter:
- **Linear spread risk** drove the largest piece (after the default).
- **The discrete default event** was the biggest single contribution.
- **Carry/rolldown** was smaller but non-negligible — and *deterministic*, so it is the easiest piece to forecast and budget against.

---

### Example 11: Tranche PV01

**Goal:** compute tranche PV01 by symmetric finite difference (the standard production method).

#### Setup

| Item | Value |
|------|-------|
| Tranche notional $N_T$ | USD 10mm |
| Tranche $RPV01_T$ | 3.2 (toy) |
| At-par tranche | Contractual spread = market spread → PV $\approx 0$ |

#### Step 1 — Revalue at ±1 bp

| Bump | Calc | $PV$ |
|------|------|------|
| $+1$ bp | $N_T \cdot 3.2 \cdot 1 \cdot 10^{-4}$ | $+USD 3{,}200$ |
| $-1$ bp | $-N_T \cdot 3.2 \cdot 1 \cdot 10^{-4}$ | $-USD 3{,}200$ |

#### Step 2 — Symmetric Finite Difference

$$PV01 = \frac{PV(+1) - PV(-1)}{2} = \frac{3{,}200 - (-3{,}200)}{2} = USD 3{,}200/\text{bp}$$

Symmetric bumps are preferred in practice because they remove the linear bias of one-sided differences.

---

### Example 12: Correlation Shock Toy

**Goal:** revalue a tranche at two correlation levels and back out Corr01 — the standard way correlation risk is reported.

#### Setup

| Item | Value |
|------|-------|
| Tranche notional $N_T$ | USD 10mm |
| $PV(\rho = 20\\%)$ | $-USD 400{,}000$ |
| $PV(\rho = 30\\%)$ | $-USD 350{,}000$ |

#### Step 1 — P&L from a +10% Correlation Shock

$$\Delta PV = PV(\rho = 30\\%) - PV(\rho = 20\\%) = -350{,}000 - (-400{,}000) = +USD 50{,}000$$

#### Step 2 — Corr01 (per 1% correlation)

$$\text{Corr01} \approx \frac{\Delta PV}{\Delta \rho \\, (\\%)} = \frac{50{,}000}{10} = USD 5{,}000 \text{ per } 1\\%$$

#### Interpretation

This tranche **benefits** as correlation rises (positive Corr01). The sign reveals where on the capital structure this tranche sits — a typical pattern is:
- Long-protection equity → negative Corr01 (loses when correlation rises).
- Long-protection senior → positive Corr01 (gains when correlation rises).

A positive Corr01 here is consistent with a senior-style tranche.

> **Caveat:** Corr01 is *local*. It assumes Corr01 itself is constant over the bumped range — a poor assumption for large correlation moves. See Example 20 for a stressed scenario.

---

### Example 13: Tail / Clustering Scenario

**Goal:** show that PV01 (linear) hedges fail when defaults cluster, because tranche loss is nonlinear in realized portfolio loss.

#### Setup

| Item | Value |
|------|-------|
| Portfolio names $M$ | 100 (equal weight) |
| Recovery $R$ | 40% |
| Loss per default (% of portfolio) | $\ell = (1-R)/M = 0.6\\%$ |
| Equity tranche $[A, B]$ | $[0\\%, 3\\%]$ → width $B-A = 3\\%$ |
| Tranche notional $N_T$ | USD 10mm |

#### Scenario Comparison

| | 1 default | 5 defaults (cluster) |
|---|-----------|----------------------|
| Portfolio loss $L$ | $0.6\\%$ | $5 \times 0.6\\% = 3.0\\%$ |
| Tranche loss fraction $L_{[0,3]}$ | $0.6 / 3 = 0.20$ | $3.0 / 3 = 1.00$ (fully written down) |
| Dollar loss (protection seller) | $0.20 \times 10\text{mm} = USD 2\text{mm}$ | $1.00 \times 10\text{mm} = USD 10\text{mm}$ |

5× the defaults → 5× the tranche loss in this range, but the equity tranche is **completely consumed** at 5 defaults. Beyond that, additional losses skip equity entirely and hit the next tranche.

#### Why PV01 Hedges Fail

A PV01 hedge is calibrated to **small linear spread moves**. A clustered default scenario creates a *jump* in realized loss that has no linear relationship to a 1–5 bp spread bump. The hedge ratio that worked yesterday gives no protection against tomorrow's cluster.

---

### Example 14: Adjacent Tranche Hedge

**Goal:** PV01-neutral hedge of an equity tranche with a mezz tranche works for small spread moves but fails badly under default clustering.

#### Setup

| Tranche | Range | Notional | PV01 (per USD 10mm) |
|---------|-------|----------|---------------------|
| **Target** — equity | $[0, 3]$ | $N_E = USD 10$mm | $USD 3{,}200$/bp (from Ex. 11) |
| **Hedge** — mezz | $[3, 7]$ | $N_M$ (TBD) | $USD 1{,}800$/bp |

#### Step 1 — PV01-Neutral Sizing

Take opposite spread exposure on mezz so the legs offset:

$$\frac{N_M}{10\text{mm}} = \frac{PV01_E}{PV01_M} = \frac{3{,}200}{1{,}800} \approx 1.7778 \\;\Rightarrow\\; \boxed{N_M = USD 17.778\text{mm}}$$

#### Step 2 — Check Under +1 bp Spread Move

| Leg | P&L magnitude |
|-----|---------------|
| Equity | $USD 3{,}200$ |
| Mezz (opposite sign) | $1{,}800 \times 1.7778 = USD 3{,}200$ |
| **Net** | **$\approx 0$** ✅ |

#### Step 3 — Failure Under Clustering (10 defaults)

Use Example 13's portfolio assumptions ($M=100$, $R=40\\%$, loss per default $= 0.6\\%$).

10 defaults → portfolio loss $L = 6\\%$.

| Tranche | Loss fraction | Dollar loss |
|---------|---------------|-------------|
| Equity $[0, 3]$ | $1.00$ (fully consumed) | $1.00 \times USD 10$mm $= USD 10$mm |
| Mezz $[3, 7]$ | $\frac{\min(\max(6-3, 0), 4)}{4} = \frac{3}{4} = 0.75$ | $0.75 \times USD 17.778$mm $= USD 13.333$mm |

#### Interpretation

A PV01-neutral hedge can become **over-hedged** or **under-hedged** in default clusters because losses are nonlinear and tranche-dependent. The hedge that costs nothing on small spread moves can produce massive net losses under clustering.

---

### Example 15: Strategy Comparison Table

**Goal:** put three of the chapter's core strategies side by side so the *shape* of their exposures is easy to compare.

#### Toy Exposure Snapshot (Illustrative)

| Strategy | Instruments | CS01 (USD/bp) | JTD (USD) | Rec01 | Dominant basis risk | Typical hedge map |
|----------|-------------|---------------|-----------|-------|---------------------|-------------------|
| **Bond–CDS basis (A1)** | bond + CDS | bond $-420$; CDS $+459$ | bond $-610$k; CDS $+610$k | meaningful | CDS–cash basis | DV01 hedge (rates) + CS01 or JTD hedge (CDS notional) |
| **Single-name vs index (A2)** | single CDS + index CDS | single $+4{,}500$; index short $-4{,}500$ | single default dominates | meaningful | proxy/index basis + roll | CS01 match; event stress for residual JTD |
| **Tranche RV (C1)** | tranche + hedge(s) | tranche PV01 $3{,}200$ | jump via losses | recovery + correlation | dependence / tail | PV01 hedge (index or adjacent tranche); Corr01 monitoring; cluster stress |

**Reading the table:** notice that CS01 numbers do not tell the whole story — basis sensitivity and JTD/cluster behavior differ qualitatively across strategies. The Strategy Card discipline forces you to map each of these dimensions explicitly.

---

## 52.13 Practical Notes

### Educational Only — No Trade Tips

This chapter is an educational risk framework. It does **not** provide recommendations, forecasts, or "what to trade now."

### Conceptual Pitfalls

| Pitfall | Specifics |
|---------|-----------|
| **Mixing CS01 definitions** | Quote bump vs hazard-rate bump; index-spread bump vs constituent bump |
| **Mixing quoting regimes** | Running spread vs fixed coupon + upfront — inconsistent treatment |
| **Ignoring recovery / final-price risk** | Default payoff depends on recovery / final price (Examples 1, 9, 16) |
| **Assuming roll calendars / coupons** | Index conventions are product- and rulebook-specific — confirm, don't assume |
| **Confusing hedging with RV exposure** | Hedging = risk reduction; RV = basis risk *intentionally* retained |

### Implementation Pitfalls

| Pitfall | What goes wrong |
|---------|-----------------|
| **Unit mistakes** | bp vs decimals; per-100 price vs per-USD notional; inconsistent scaling |
| **Inconsistent curves / recovery** | Different assumptions across legs lead to phantom P&L |
| **Execution costs dominate** | Especially on small basis edges |

### Verification Tests

| Test | What to check |
|------|---------------|
| **Linear scaling with notional** | CS01 and PV should scale linearly for small bumps |
| **Repricing** | Each leg should reprice consistently under the base scenario |
| **Scenario suite passes** | Parallel / dispersion / event / roll / tail scenarios all behave as expected |

---

### Example 16: Senior vs Subordinated CDS — Recovery-Driven Spread Relationship

**Goal:** apply the O'Kane senior-sub spread relationship to spot relative-value mispricings driven by recovery assumptions.

#### Setup

| Item | Value |
|------|-------|
| Senior 5Y CDS spread $S_{\text{sen}}$ | 120 bp |
| Senior recovery $R_{\text{sen}}$ | 40% |
| Sub recovery $R_{\text{sub}}$ | 20% |
| Market sub spread (observed) | 170 bp |

#### Step 1 — Theoretical Sub Spread

Under equal hazard rates:

$$\boxed{\frac{S_{\text{sub}}}{S_{\text{sen}}} = \frac{1 - R_{\text{sub}}}{1 - R_{\text{sen}}}}$$

$$S_{\text{sub}}^{\text{fair}} = 120 \times \frac{0.80}{0.60} = 160 \text{ bp}$$

#### Step 2 — Compare to Market

| | Ratio |
|---|-------|
| Theoretical | $0.80 / 0.60 = 1.333$ |
| Observed | $170 / 120 = 1.417$ |

Sub CDS is **10 bp wider than the model predicts**. Possible explanations:
- Market is pricing lower sub recovery than 20% (perhaps ~15%).
- Sub CDS has an extra liquidity premium.
- Genuine mispricing.

#### Step 3 — Trade Construction (If You Trust the Model)

Buy senior protection + sell sub protection — harvesting the excess sub spread. Size CS01-neutral:

$$N_{\text{sen}} = N_{\text{sub}} \times \frac{RPV01_{\text{sub}}}{RPV01_{\text{sen}}}$$

With $RPV01_{\text{sen}} = 4.1$ and $RPV01_{\text{sub}} = 3.8$ (lower survival probability compresses RPV01), for $N_{\text{sub}} = USD 5$mm:

$$N_{\text{sen}} = 5 \times \frac{3.8}{4.1} \approx USD 4.63\text{mm}$$

#### Failure Mode

If realized sub recovery at default is 5% (not 20%), then the "cheap sub protection" was actually fair-priced and you lose on the senior leg. **Recovery uncertainty dominates** — this is the single biggest risk in capital-structure RV.

---

### Example 17: HY Curve Steepener — CS01-Neutral Sizing

**Goal:** size a CS01-neutral 3Y/5Y curve trade on an inverted HY name, then show the JTD mismatch baked into CS01 neutrality.

#### Setup

| Item | Value |
|------|-------|
| 3Y CDS spread | 450 bp |
| 5Y CDS spread | 380 bp |
| Curve shape | Inverted (you believe inversion is excessive) |
| $RPV01_{3Y}$ | 2.5 (per USD 1mm) |
| $RPV01_{5Y}$ | 3.6 (per USD 1mm) |

**Trade — "bull flattener":**
- **Long 5Y protection** ($N_{5Y} = USD 10$mm).
- **Short 3Y protection** ($N_{3Y}$ to be sized).

You profit if the curve flattens (3Y tightens more than 5Y, or 5Y widens more than 3Y).

#### Step 1 — CS01-Neutral Sizing

Match CS01 so a parallel move cancels:

$$N_{3Y} = N_{5Y} \times \frac{RPV01_{5Y}}{RPV01_{3Y}} = 10\text{mm} \times \frac{3.6}{2.5} = USD 14.4\text{mm}$$

**Sign reminder.** Long protection has CS01 $\gt 0$; short protection has CS01 $\lt 0$. So when the 3Y leg (short protection) sees spreads *tighten*, it *gains*.

#### Step 2 — Spread Move Scenarios

Per-leg CS01 (in USD/bp): $CS01 = N \cdot RPV01 \cdot 10^{-4}$ (signed by long/short prot).

- 3Y short prot: $-(14{,}400{,}000 \times 2.5 \times 10^{-4}) = -USD 3{,}600/\text{bp}$
- 5Y long prot: $+(10{,}000{,}000 \times 3.6 \times 10^{-4}) = +USD 3{,}600/\text{bp}$

| Scenario | 3Y move | 5Y move | 3Y P&L (short prot) | 5Y P&L (long prot) | Net |
|----------|---------|---------|---------------------|---------------------|-----|
| Parallel $+50$ bp | $+50$ | $+50$ | $-180{,}000$ | $+180{,}000$ | **0** ✅ CS01 neutral |
| Curve flattens (3Y $-100$, 5Y flat) | $-100$ | $0$ | $+360{,}000$ | $0$ | **$+USD 360{,}000$** |
| Curve normalizes (3Y $-150$, 5Y $-30$) | $-150$ | $-30$ | $+540{,}000$ | $-108{,}000$ | **$+USD 432{,}000$** |

(Each P&L = CS01 × spread move.)

#### Step 3 — Default Scenario (Critical Failure Mode)

If the name *defaults* (recovery $R = 0.35$; all figures in USD mm):

| Leg | Payoff direction | Approx magnitude |
|-----|------------------|------------------|
| Short 3Y protection (14.4mm notional) | **Pays** $(1-R)\,N_{3Y}$ | $-9.36$ |
| Long 5Y protection (10mm notional) | **Receives** $(1-R)\,N_{5Y}$ | $+6.5$ |
| **Net JTD** | $-(1-R)(N_{3Y} - N_{5Y}) = -0.65 \times 4.4$ | $-2.86$ |

#### Lesson

CS01 neutrality requires *unequal* notionals. The larger short-protection leg means default produces a substantial net loss. **CS01-neutral sizing creates a JTD mismatch — by construction.**

> **Desk Reality:** HY curve trades are dangerous for exactly this reason. Some desks cap the notional ratio; others add a JTD overlay (extra long protection) at the cost of breaking CS01 neutrality. There is no free lunch.

---

### Example 18: Negative Basis P&L Under Funding Stress

**Goal:** show that a "negative basis" package can have *negative* carry once funding cost is included, and that a funding shock can wipe out the position.

#### Setup

| Item | Value |
|------|-------|
| Position | Long corporate bond + long 5Y CDS protection |
| Bond Z-spread | 180 bp |
| CDS spread | 150 bp |
| Bond–CDS basis | $-30$ bp (CDS tighter than bond) |
| Funding cost | LIBOR + 50 bp |
| Notional | USD 10mm |

#### Step 1 — Initial Annual Carry

| Component | bp | USD/yr |
|-----------|----|--------|
| Bond coupon (spread component) | $+180$ | $+USD 180{,}000$ |
| CDS premium paid | $-150$ | $-USD 150{,}000$ |
| Funding cost over LIBOR | $-50$ | $-USD 50{,}000$ |
| **Net carry** | **$-20$** | **$-USD 20{,}000$** |

The "free lunch" of $-30$ bp basis is *more than consumed* by $-50$ bp funding cost — net carry is negative.

#### Step 2 — Funding Stress (LIBOR + 120 bp)

| Component | bp | USD/yr |
|-----------|----|--------|
| Bond coupon | $+180$ | $+USD 180{,}000$ |
| CDS premium paid | $-150$ | $-USD 150{,}000$ |
| Funding cost (stressed) | $-120$ | $-USD 120{,}000$ |
| **Net carry (stressed)** | **$-90$** | **$-USD 90{,}000$/yr** |

The position now bleeds USD 90k per year while waiting for basis convergence.

#### Step 3 — MTM Impact (Elevated Funding Persists Across the Bond's Remaining Life)

If the market prices the elevated funding as persistent across the bond's remaining ~5Y maturity, MTM moves by approximately the present value of the carry deterioration over that horizon ($RPV01_{\text{bond}} \approx 4.0$):

$$\Delta PV \approx -90 \text{ bp} \times RPV01_{\text{bond}} \times N \approx -90 \times 4.0 \times 10^{-4} \times 10{,}000{,}000 = -USD 360{,}000$$

(For a shorter assumed funding-stress horizon $\tau$, replace $RPV01_{\text{bond}}$ with the matching annuity factor — e.g., a 2-year assumption uses ~2.0 instead of 4.0.)

#### Desk Note

Funding and balance-sheet costs are *institution-specific*. The same "basis" package can be:
- **Positive carry** for a bank with cheap repo,
- **Negative carry** for a hedge fund paying a wide spread.

Always stress the **financing leg**, not just the spread leg. A basis position is a funding position whether you frame it that way or not.

---

### Example 19: Equity-CDS Relative Value — LBO Scenario

**Goal:** show why naive equal-notional sizing of an "equity short / CDS long" Merton trade *underweights the CDS leg* — and loses money if an LBO actually happens.

#### Setup

| Item | Value |
|------|-------|
| Equity market cap | USD 8bn |
| 5Y CDS spread | 90 bp |
| View | Equity prices minimal distress; CDS relatively cheap; LBO possible |

**Pre-LBO position:** short equity + long CDS protection (the classic Merton trade — both legs profit if leverage rises).

**Trade size (naive, equal notional):**
- Short equity: USD 5mm
- Long CDS protection: USD 5mm, RPV01 = 4.3

#### Step 1 — LBO Announcement (4× Leverage)

| Instrument | Pre-LBO | Post-LBO | P&L direction |
|------------|---------|----------|---------------|
| Equity | USD 40/share | USD 55/share (tender premium) | **Loss** on short equity |
| Senior CDS | 90 bp | 350 bp (leverage spike) | **Gain** on long protection |
| Sub CDS | 200 bp | 800 bp | Even larger gain (not in this position) |

#### Step 2 — Compute P&L

| Leg | Calc | P&L |
|-----|------|-----|
| Short equity | $-5\text{mm} \times (55-40)/40 = -5\text{mm} \times 37.5\\%$ | $-USD 1{,}875{,}000$ |
| Long CDS (5mm × RPV01 × $\Delta S$) | $(350 - 90) \times 4.3 \times 10^{-4} \times 5{,}000{,}000$ | $+USD 559{,}000$ |
| **Net** | | **$-USD 1{,}316{,}000$** |

The trade *loses* money even though both legs moved "the right way" — the equity tender premium is much larger (in dollar terms) than the CDS spread move.

#### Lesson

Equity-CDS sizing is hard:
- Equity moves are **discontinuous** (tender premium, gap risk).
- CDS spread moves are large but **bounded** (capped by LGD).

Professional desks typically:
1. Use **equity puts** rather than short stock to limit downside.
2. **Overweight the CDS leg** because equity is more leverage-sensitive (Merton: equity = call on assets).

The Merton intuition — equity has higher delta to leverage events than CDS — implies the dollar weights should *not* be equal.

> **Caveat:** post-event levels here are **toy numbers** for sizing intuition only. Actual moves depend on deal structure, sector, leverage, and market regime.

---

### Example 20: Correlation Shock on Equity-Senior Combo

**Goal:** show that a Corr01-neutral combo at one correlation level is *not* Corr01-neutral after a large correlation shock — there is "correlation convexity."

#### Setup

Build a combo at $\rho = 25\\%$ using two long-protection tranches with opposite Corr01 signs (per USD 10mm notional):

| Tranche | Corr01 at $\rho = 25\\%$ |
|---------|---------------------------|
| Equity (0–3%), long protection | $-USD 8{,}000$ per $+1\\%$ corr |
| Senior (7–10% or 15–30%), long protection | $+USD 15{,}000$ per $+1\\%$ corr |

#### Step 1 — Corr01-Neutral Sizing (at $\rho = 25\\%$)

Take USD 10mm senior plus USD 18.75mm equity (ratio $15{,}000 / 8{,}000 = 1.875$):

$$+15{,}000 + 1.875 \times (-8{,}000) \approx 0$$

#### Step 2 — Correlation Shock: $\rho$ Jumps to 40% ($+15\\%$)

Corr01 itself drifts with $\rho$. Suppose at $\rho = 40\\%$:

| Tranche | Corr01 at $\rho = 40\\%$ |
|---------|---------------------------|
| Equity | $-USD 5{,}000$ per $+1\\%$ |
| Senior | $+USD 18{,}000$ per $+1\\%$ |

#### Step 3 — Net Corr01 at New Level

$$\text{NetCorr01}_{40\\%} \approx 18{,}000 + 1.875 \times (-5{,}000) = 8{,}625 \text{ USD per } 1\\%$$

The package is **no longer Corr01-neutral** at $\rho = 40\\%$.

#### Step 4 — Convexity Estimate of Total P&L

Crude trapezoidal estimate (average net Corr01 over the move):

$$\Delta PV_{\rho} \approx \frac{0 + 8{,}625}{2} \times 15 \approx +USD 64{,}700$$

#### Lesson

Corr01 neutrality is a **local first-order hedge**. Large correlation moves introduce:
- **Convexity** (the Greek itself drifts).
- **Hedge-ratio drift** (yesterday's neutral ratio is today's stale ratio).

Validate any correlation package with **stressed** correlation moves, not just $\pm 1\\%$ bumps.

---

### Example Index

| Example | Topic | Key Concept |
|---------|-------|-------------|
| 1 | Bond-CDS Basis Toy | JTD matching, basis P&L |
| 1B | Basis Package Under Funding Stress | Funding shock impact |
| 2 | DV01-Neutral but Not Credit-Neutral | Rates vs credit distinction |
| 3 | Single-Name Hedged with Index | CS01 matching, idiosyncratic residual |
| 4 | Index Hedged with Constituents | Bottom-up hedge, basis complications |
| 5 | Index Intrinsic vs Quoted Basis | Basis computation, P&L scenario |
| 6 | Index Carry | Coupon cashflow, upfront decay |
| 7 | Index Rolldown | Aging along curve |
| 8 | Roll Trade Mechanics | Composition + maturity effects |
| 8B | Roll with Favorable Composition | When composition dominates |
| 9 | Default in Index | Settlement mechanics, notional reduction |
| 10 | Full P&L Explain | Carry + rolldown + spread + event |
| 11 | Tranche PV01 | Finite difference computation |
| 12 | Correlation Shock Toy | Corr01 calculation |
| 13 | Tail/Clustering Scenario | Why PV01 hedges fail |
| 14 | Adjacent Tranche Hedge | PV01 match, clustering failure |
| 15 | Strategy Comparison Table | Side-by-side exposure map |
| 16 | Senior vs Sub CDS | Recovery-driven spread relationship |
| 17 | HY Curve Steepener | CS01-neutral sizing, JTD mismatch |
| 18 | Negative Basis Funding Stress | Carry decomposition, funding risk |
| 19 | Equity-CDS Relative Value (LBO) | Merton model application, sizing |
| 20 | Correlation Shock on Combo | Corr01 convexity, local vs global hedging |

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **CS01 (Spread DV01)** | PV change per +1 bp change in quoted spread: $CS01 = \frac{\partial PV}{\partial S} \times 10^{-4}$ | The primary linear risk measure for credit positions; must specify *which* spread is bumped |
| **JTD / VOD** | Jump-to-default = $PV_{\text{post}} - PV_{\text{pre}}$; VOD = $(1-R-\text{Accrued}) - (S-S_0) \cdot RPV01$ | Captures discontinuous default risk that CS01 hedges do not neutralize |
| **Bond–CDS Basis** | CDS spread − Bond Libor spread (typically asset swap spread) | Twelve drivers (6 fundamental + 6 market) per O'Kane; a persistent tradeable phenomenon |
| **Loss-on-Default Mismatch** | Bond loss = $P - R$; CDS loss = $1 - R$; difference when bond price ≠ 100 | A basis driver that causes hedge ratios to differ from par assumptions |
| **Intrinsic vs Quoted Basis** | Gap between index's quoted spread and RPV01-weighted average of constituent spreads | Driven by restructuring clause, liquidity, and "index leads the market" effects |
| **Portfolio Swap Adjustment** | Adjusting constituent curves so intrinsic index value = quoted value | Often required for tranche/correlation analytics; the specific adjustment rule is a modeling choice |
| **Index Roll P&L** | P&L from switching series due to (1) composition changes and (2) maturity extension | The two effects can offset or compound; crucial for roll timing decisions |
| **Systemic DV01 / Delta** | PV change under a parallel portfolio spread move, and the hedge notional implied by the source definition | Leverage tends to be highest for equity and lower for senior; hedge ratios are regime/model dependent |
| **Systemic Gamma** | Second-order tranche sensitivity to parallel spread moves | Negative for equity (large moves hurt), positive for senior (large moves help) |
| **Corr01** | PV change per +1% correlation increase | *Opposite* sign for equity vs senior: negative for equity, positive for senior (long protection) |
| **Tranche Loss Fraction** | $L_{[A,B]} = \frac{1}{B-A}(\max(L-A,0) - \max(L-B,0))$ | Translates portfolio loss to tranche-specific loss; highly nonlinear for equity |
| **Gamma vs Carry Trade-off** | Convexity exposures can come with adverse carry/theta | Always evaluate carry and convexity together under scenarios; avoid “carry-only” thinking |
| **Strategy Card Discipline** | Exposures + Hedges + Failure Modes | Every strategy must enumerate residual risks and scenarios where hedges break |
| **Idiosyncratic Delta** | Tranche sensitivity to single-name spread move | Differs from systemic delta; reveals name-specific risk concentration |
| **Delivery Option** | CDS protection buyer's right to choose cheapest deliverable bond | Widens CDS spreads relative to bonds; source of basis |
| **Funding Risk** | Exposure to repo/financing rate changes on funded instruments | CDS is unfunded; bonds face funding risk that can dominate carry |
| **Negative Basis** | CDS spread < bond asset swap spread; synthetic protection cheaper than cash | Exploitable if funding cost is low enough; destroyed by funding stress |
| **Positive Basis** | CDS spread > bond spread; protection demand exceeds cash supply | Tends to emerge during distress; squeeze dynamics |
| **Senior-Sub Spread Ratio** | `S_sub/S_sen = (1-R_sub)/(1-R_sen)` under equal hazard rates | Foundation for capital structure arbitrage; recovery uncertainty is key risk |
| **Merton Model (Credit)** | Equity = call on firm assets; debt = cash − put on assets | Links equity and credit markets; basis for equity-CDS RV trades |
| **LCDS** | Loan CDS referencing loans and embedding a cancellation feature | Often trades tighter than vanilla CDS due to higher expected recovery; conventions and liquidity matter |
| **CDS Curve Inversion** | Short-dated spreads exceed long-dated (common in HY distressed) | Signals near-term default risk; curve trades carry JTD mismatch |
| **Corr01 Convexity** | Corr01 itself changes with correlation level (second-order) | Corr01-neutral is local; large moves require full revaluation and scenario testing |

---

## 52.14 Summary and Key Takeaways

### 16-Bullet Executive Summary

1. A credit "strategy" must be written as **exposures + hedges + failure modes** (not a slogan).
2. **CS01 is linear spread risk**; specify exactly what spread is bumped.
3. **JTD/VOD captures default discontinuity**; CS01 hedges do not neutralize default jumps.
4. **Recovery/final price** is a key state variable; mismatches drive hedge surprises.
5. **Bond–CDS basis has six fundamental drivers** (O'Kane): funding, delivery option, technical default, loss-on-default mismatch, premium accrued at default, and CDS non-negativity.
6. **Bond–CDS basis has six market drivers** (O'Kane): relative liquidity, CDO issuance effects, new bond/loan issuance, convertible arbitrage flows, demand for protection, and funding risk.
7. **Negative basis persistence** depends on funding costs; the "same" trade has different economics for bank desks vs hedge funds.
8. **Indices have intrinsic vs quoted basis**; the portfolio swap adjustment reconciles them but is "somewhat arbitrary" (O'Kane).
9. **Index rolls create P&L** from composition changes and maturity extension—these can offset or compound.
10. **CDS curve strategies** require CS01-neutral sizing but carry inherent JTD mismatch from unequal notionals.
11. **Capital structure arbitrage** exploits the senior-sub spread relationship (`S_sub/S_sen = (1-R_sub)/(1-R_sen)`), but recovery uncertainty is the dominant risk.
12. **Tranche risk is multi-dimensional**: systemic delta, systemic gamma, idiosyncratic delta, idiosyncratic gamma, Corr01, carry, and theta all matter.
13. **Equity and senior tranches have opposite correlation exposures**: Corr01 is negative for equity, positive for senior (for long protection).
14. **Positive gamma positions typically have negative carry** (O'Kane)—there is a cost to owning convexity.
15. **Crisis behavior differs by strategy**: funding stress destroys basis trades; correlation spikes destroy tranche RV; curve inversion signals imminent default.
16. **Always run a scenario suite:** parallel, dispersion, default event, roll/series basis, correlation/tail shocks, and funding stress.

### Cheat Sheet

**Strategy card:** Objective → Instruments → Exposures table → Hedge ratios → Scenario suite → Failure modes → Checks.

**Key formulas:**

- CDS MTM: $V = (S - S_0) \cdot RPV01$.
- CS01 hedge: $N_H = -CS01_T / CS01_H$.
- VOD/JTD (CDS): $VOD = (1 - R - \text{Accrued}) - (S - S_0) \cdot RPV01$.
- Tranche loss: $L_{[A,B]} = \frac{1}{B - A} \left( \max(L - A, 0) - \max(L - B, 0) \right)$.
- Senior-sub spread ratio: `S_sub/S_sen = (1 - R_sub)/(1 - R_sen)`.
- Curve trade CS01-neutral sizing: $N_{\text{short}} = N_{\text{long}} \times RPV01_{\text{long}} / RPV01_{\text{short}}$.

---

### 58 Flashcards (Q/A)

1. **Q:** What does CS01 measure? **A:** PV change for a +1 bp change in the specified quoted spread.

2. **Q:** For long protection CDS, is CS01 positive or negative? **A:** Positive (PV rises when spreads widen).

3. **Q:** What is RPV01? **A:** PV of 1 bp/yr of premium leg per unit notional (risky PV01).

4. **Q:** Give the CDS MTM identity used for CS01. **A:** $V = (S - S_0) \cdot RPV01$.

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

25. **Q:** What scenario suite should you run to validate a "hedged" strategy? **A:** Parallel/systemic moves, dispersion, default/JTD, recovery/final price, roll/basis, funding/liquidity, and (for tranches) tail/correlation.

26. **Q:** What is series/roll basis? **A:** Price/spread difference between index series due to roll, maturity reset, liquidity, composition.

27. **Q:** Why does recovery matter beyond spreads? **A:** Default settlement payoff depends on recovery/final price.

28. **Q:** What is a key risk management lesson about "perfect storm" events? **A:** Multiple adverse moves can coincide; stress testing is essential.

29. **Q:** What is the simplest CS01 hedge ratio? **A:** $N_H = -CS01_T / CS01_H$.

30. **Q:** What is the core discipline of this chapter? **A:** Define exposures, hedge explicitly, then enumerate failure modes.

31. **Q:** What are O'Kane's six fundamental basis factors? **A:** Funding, delivery option, technical default, loss-on-default mismatch, premium accrued at default, CDS spreads non-negative.

32. **Q:** What are O'Kane's six market basis factors? **A:** Relative liquidity, CDO technical short, new issuance/loans, convertible issuance, demand for protection, funding risk.

33. **Q:** How can structured-credit issuance affect CDS spreads? **A:** Through technical flows and hedging that change net demand/supply for buying vs selling protection; the direction is deal- and market-dependent.

34. **Q:** What does O'Kane mean by "the index leads the market"? **A:** The CDS index is more liquid and is the preferred instrument for expressing market-wide credit views, especially for hedging.

35. **Q:** What is systemic gamma for an equity tranche? **A:** Large and negative (for long protection)—large parallel spread moves decrease PV.

36. **Q:** What is systemic gamma for a senior tranche? **A:** Positive (for long protection)—large parallel spread moves increase PV.

37. **Q:** What carry profile is typical for a positive gamma position? **A:** Often negative carry: you tend to “bleed” in calm markets and earn when volatility/large moves arrive.

38. **Q:** Why are equity tranches described as “leveraged”? **A:** Because attachment/detachment concentrates portfolio loss into a thin slice, making PV extremely sensitive to small changes in expected loss and dispersion.

39. **Q:** What is the portfolio swap adjustment? **A:** Adjusting individual CDS curves so that intrinsic index value equals quoted market value.

40. **Q:** Why is base correlation interpolation risky? **A:** O'Kane shows linear interpolation is not guaranteed to be arbitrage-free for non-standard tranches.

41. **Q:** What are O'Kane's two primary sources of P&L during index roll? **A:** (1) Composition changes affecting perceived credit quality, (2) Maturity extension on an upward-sloping curve.

42. **Q:** Why might a new index series trade tighter than the old series? **A:** Composition improvement (removing distressed names) can dominate the maturity extension effect.

43. **Q:** How do you calculate CS01-adjusted notional for spread-neutral roll? **A:** $N_{\text{new}} = N_{\text{old}} \times \frac{CS01_{\text{old}}}{CS01_{\text{new}}}$.

44. **Q:** What's the risk of rolling "flat notional" between index series? **A:** You increase spread exposure because RPV01 increases with longer maturity (new series has higher CS01 per dollar).

45. **Q:** What is a "negative basis" in bond-CDS terms? **A:** When CDS spread < bond asset swap spread, i.e., synthetic protection is cheaper than cash credit risk.

46. **Q:** How can technical flows move bond–CDS basis? **A:** If one leg is easier/harder to hold or trade (funding, balance-sheet, liquidity), cash-bond spreads and CDS spreads can decouple, compressing or widening basis.

47. **Q:** What is the "positive basis squeeze"? **A:** When CDS spreads widen above bond spreads—often during distress when protection demand surges and bond liquidity dries up.

48. **Q:** What is the senior-sub CDS spread ratio under equal hazard rates? **A:** `S_sub / S_sen = (1 - R_sub) / (1 - R_sen)` — driven purely by recovery difference.

49. **Q:** What does the Merton model say about equity and debt? **A:** Equity is a call option on firm assets; debt is equivalent to risk-free cash minus a put on firm assets.

50. **Q:** Why are HY CDS curves often inverted? **A:** Near-term default probability is high; if the issuer survives, forward default rates decline. Survival selection effect.

51. **Q:** What is LCDS? **A:** Loan CDS—credit protection on senior secured loans; higher expected recovery than senior unsecured bonds often implies lower spreads under similar default-intensity assumptions.

52. **Q:** What is the theoretical LCDS-to-CDS spread ratio under equal hazard rates? **A:** $S_{\text{LCDS}} / S_{\text{CDS}} \approx (1 - R_{\text{loan}}) / (1 - R_{\text{bond}})$; the result is recovery-driven and sensitive to contract conventions.

53. **Q:** What happens to negative basis trades in a funding crisis? **A:** Funding costs spike, eroding or reversing carry. Leveraged holders face margin calls and forced unwinds, driving basis wider.

54. **Q:** What can happen to base correlation in a stress scenario? **A:** It can move sharply and change shape across strikes; treat correlation as a state variable and stress-test beyond small Corr01 bumps.

55. **Q:** Name three early warning indicators for credit strategy stress. **A:** (1) Funding/balance-sheet stress, (2) Liquidity deterioration (bid–ask widening), (3) Breakdown of historically stable relationships (basis/roll/correlation).

56. **Q:** Why does CS01-neutral sizing create a JTD mismatch in curve trades? **A:** Matching spread risk requires unequal notionals (short-dated leg is larger), so default produces a net loss on the excess notional.

57. **Q:** What is the "local vs global" hedge ratio distinction in tranches? **A:** Local ratios (Corr01, PV01) are valid for small moves; global P&L profiles can diverge for large moves due to nonlinearity/convexity.

58. **Q:** In an LBO scenario, why does equity move more than CDS? **A:** Equity is a call option on assets (high delta to leverage events); CDS spread increases are bounded by the LGD cap.

---

## 52.15 Mini Problem Set

**Questions 1–10 include brief solution sketches.**

---

**1.** A 5Y CDS has $RPV01 = 4.2$. Compute CS01 per USD 1mm notional for long protection.

> **Sketch:** $CS01 = 1{,}000{,}000 \times 4.2 \times 10^{-4} = USD 420/\text{bp}$.

---

**2.** A long bond has rates DV01 $-6{,}000$ USD/bp. A hedging Treasury has DV01 $-800$ USD/bp per USD 1mm (long). What Treasury notional (short) DV01-hedges the bond?

> **Sketch:** Short notional $N$ gives $+800N$. Solve $-6000 + 800N = 0 \Rightarrow N = 7.5\text{mm}$.

---

**3.** Using VOD at par $\approx 1 - R - \text{Accrued}$, compute JTD for long protection with notional USD 5mm, $R = 40\%$, accrued premium USD 6,000.

> **Sketch:** $0.60 \times 5{,}000{,}000 - 6{,}000 = 2{,}994{,}000$.

---

**4.** A single-name has CS01 $+3{,}800$ USD/bp. An index has CS01 $+4{,}200$ USD/bp per USD 10mm. What index notional (short protection) CS01-hedges the single name?

> **Sketch:** Need $-4{,}200(N/10) + 3{,}800 = 0 \Rightarrow N = 9.0476\text{mm}$.

---

**5.** Compute tranche loss fraction for $[A, B] = [3\\%, 7\\%]$ when portfolio loss $L = 5\\%$.

> **Sketch:** Loss in tranche $= \min(\max(5 - 3, 0), 4) / 4 = 2/4 = 0.5$.

---

**6.** A tranche PV at $\rho = 15\\%$ is $-200k$ and at $\rho = 20\\%$ is $-230k$. Compute Corr01.

> **Sketch:** $\Delta PV = -30k$ for +5% corr $\Rightarrow$ Corr01 $\approx -6k$ per 1%.

---

**7.** An index default on one name pays $(1 - R) N_m$. If $N = 25\text{mm}$, $M = 125$, $R = 35\\%$, compute payout ignoring accrued.

> **Sketch:** $N_m = 25/125 = 0.2\text{mm}$; payout $= 0.65 \times 200k = 130k$.

---

**8.** You have a bond–CDS package with bond CS01 $-500$ USD/bp and CDS CS01 $+450$ USD/bp per USD 1mm. What CDS notional makes CS01 net zero?

> **Sketch:** Need $+450N - 500 = 0 \Rightarrow N = 1.1111\text{mm}$.

---

**9.** For an index long protection position, coupon $C = 100$ bp, notional USD 10mm, month fraction 1/12. Compute coupon accrual cashflow sign and magnitude.

> **Sketch:** Long protection pays premium: $-10\text{mm} \cdot 0.01 \cdot 1/12 = -USD 8{,}333.33$.

---

**10.** A PV01-neutral adjacent tranche hedge uses notional ratio $N_H = PV01_T / PV01_H$. If PV01s are 2,400 and 1,600 (both per USD 10mm), compute hedge notional for a USD 20mm target.

> **Sketch:** Ratio $= 2400/1600 = 1.5$. For USD 20mm target (=2×USD 10mm), hedge notional $= 2 \times 1.5 \times 10\text{mm} = 30\text{mm}$.

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

**21.** A senior CDS trades at 100 bp with $R_{\text{sen}} = 40\\%$. A subordinated CDS trades at 175 bp with $R_{\text{sub}} = 20\\%$. (a) Compute the theoretical sub spread using the O'Kane senior-sub relationship. (b) Is sub CDS rich or cheap relative to the model? (c) What recovery assumption would rationalize the market sub spread?

> **Sketch:** (a) $S_{\text{sub}}^{\text{fair}} = 100 \times (1-0.20)/(1-0.40) = 133.3$ bp. (b) Market sub at 175 is wide of model by ~42 bp — sub protection looks cheap (or market prices lower sub recovery). (c) Solve $175 = 100 \times (1-R_{\text{sub}})/(0.60)$ → $R_{\text{sub}} = 1 - 1.05 = -5\\%$, which is impossible, so the gap must reflect liquidity premium or correlation with default intensity, not pure recovery.

---

**22.** You want to construct a CS01-neutral 3Y-5Y HY curve **flattener** (long 5Y protection, short 3Y protection — the "survival bet" structure from Example 17). 5Y notional is USD 10mm with $RPV01_{5Y} = 3.4$. 3Y has $RPV01_{3Y} = 2.2$. (a) Compute 3Y notional. (b) Compute the JTD mismatch at recovery $R = 30\%$. (c) Should you add a JTD overlay? How?

> **Sketch:** (a) $N_{3Y} = N_{5Y} \times RPV01_{5Y}/RPV01_{3Y} = 10 \times 3.4/2.2 = 15.45\text{mm}$. (b) For a flattener (short 3Y prot, long 5Y prot), default produces a net JTD of $-(1-R)(N_{3Y} - N_{5Y}) = -0.70 \times 5.45 = -USD 3.82\text{mm}$ (net loss). (c) Yes — buy additional 5Y protection of $\Delta N_{5Y} = 5.45\text{mm}$ so the long-protection notional matches the short-protection notional ($N_{3Y} = N_{5Y} = 15.45\text{mm}$ in total long, neutralizing JTD). This breaks CS01 neutrality — trade-off between curve and default risk.

---

**23.** A negative basis package has: bond spread 200 bp, CDS spread 160 bp (basis = $-40$ bp), funding cost = LIBOR + 60 bp. (a) Compute annual carry per USD 10mm. (b) At what funding cost does the trade break even? (c) If funding widens to LIBOR + 150 bp during a crisis and stays there for 6 months, what is the carry loss?

> **Sketch:** (a) Carry $= (200 - 160 - 60) \times 10^{-4} \times 10{,}000{,}000 = -USD 20{,}000$/yr (negative carry). (b) Break-even: $200 - 160 - x = 0 \Rightarrow x = 40$ bp. (c) At LIBOR + 150: carry $= (200-160-150) = -110$ bp $= -USD 110{,}000$/yr. Over 6 months: $-USD 55{,}000$.

---

**24.** An LBO increases a firm's leverage from 2× to 5× EBITDA. Pre-LBO: senior CDS = 80 bp, equity price = USD 45. Post-LBO: senior CDS = 280 bp, equity tender at USD 60. You are short USD 3mm equity and long USD 8mm CDS protection with $RPV01 = 4.0$. (a) Compute equity leg P&L. (b) Compute CDS leg P&L. (c) Was the trade profitable? (d) What sizing change would improve the outcome?

> **Sketch:** (a) Equity loss: $3\text{mm} \times (60-45)/45 = 3\text{mm} \times 33.3\% = -USD 1{,}000{,}000$. (b) CDS gain: $(280-80) \times 4.0 \times 10^{-4} \times 8{,}000{,}000 = +USD 640{,}000$. (c) Net: $-USD 360{,}000$ — unprofitable. (d) Increase CDS notional relative to equity — the CDS leg needs more weight because equity jumps are larger (percentage) than CDS spread moves. Alternatively, use equity puts to cap equity loss.

---

**25.** You hold a Corr01-neutral equity-senior tranche combo. (a) Why can this position still lose money on a large correlation shock? (b) What additional risk measure would you monitor? (c) Describe a scenario where the "neutral" position produces a large loss.

> **Sketch:** (a) Corr01 is a *local* (first-order) measure. For large moves, the correlation sensitivity itself changes — this is "correlation convexity" or second-order correlation risk. (b) Monitor the *change* in Corr01 as correlation moves (i.e., second-order sensitivity), and run full revaluation under ±10-20% correlation shocks. (c) A rapid decorrelation event (e.g., one sector defaults while others are fine) can cause equity tranche to move violently while senior barely responds — the local hedge ratio is stale and losses accumulate before rebalancing.

---

## References

- Dominic O'Kane, *Modelling Single-name and Multi-name Credit Derivatives* (bond-CDS basis drivers; CDS indices and index basis; roll mechanics; tranche risk measures/correlation; capital structure and recovery; LCDS)
- John C. Hull, *Risk Management and Financial Institutions* (fixed-coupon CDS/index quoting; CDS-bond basis; tranche correlation intuition)

*Chapter 52 of Fixed Income: Practice and Theory*
