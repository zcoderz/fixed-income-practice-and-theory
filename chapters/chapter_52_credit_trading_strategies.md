# Chapter 52: Credit Trading Strategies (Risk + Instrument + Hedge)

---

## Introduction

A credit portfolio manager looks at her screen one morning and sees an unfamiliar P&L: her "fully hedged" bond-CDS basis position has just lost $2 million overnight. The CDS leg is up, the bond leg is flat, and she cannot explain the discrepancy. Her CS01 was matched—she ran the numbers twice—yet somehow the position moved against her. What went wrong?

The answer lies in a truth that separates credit traders from mere speculators: **a "hedged" position is not a risk-free position**. Every credit strategy carries residual exposures that survive any hedge you construct. The question is not whether residuals exist, but whether you *know* what they are and have tested them against scenarios that matter.

This chapter provides a rigorous framework for credit strategy construction. We adopt a simple but powerful discipline: every strategy must be expressed as a **Strategy Card** that decomposes the position into explicit exposures, specifies the hedges and their ratios, and enumerates the failure modes—the scenarios where the hedge breaks down and residual risks dominate. This is not optional documentation; it is the foundation of professional risk management.

The framework applies across the credit universe:
- **Basis strategies** (Section 52.4): bond-CDS packages, single-name vs index hedges, CDS curve steepeners/flatteners
- **Carry and rolldown** (Section 52.5): index holding-period P&L decomposition, series roll mechanics
- **Correlation and tranche RV** (Section 52.6): tranche PV01, correlation exposure, and tail/clustering scenarios
- **Capital structure arbitrage** (Section 52.7): senior vs sub CDS, equity-CDS relative value, LBO trades
- **Crisis behavior and execution** (Sections 52.9-52.10): how strategies behave under stress, position sizing, and unwind discipline

The concepts build on foundations from earlier chapters: CDS mechanics (Chapter 38), index structures (Chapters 45-47), tranche pricing (Chapters 48-50), and CDS risk measures (Chapter 43). While Chapter 44 developed analytical frameworks for CDS relative value, this chapter focuses on *implementation*—how to structure positions, size hedges, and stress-test against failure scenarios. Where those chapters focused on individual instruments, this chapter focuses on *combinations*—how exposures interact, offset, and fail to offset under stress.

O'Kane emphasizes throughout his treatment of credit derivatives that stress testing and scenario analysis are not optional add-ons but core components of risk management. The "perfect storm"—multiple adverse moves coinciding—is precisely when hedges are most likely to fail and when understanding residual risk becomes most valuable. Our Strategy Card framework operationalizes this insight.

### How to Use This Chapter (Especially if You're New)

This chapter is intentionally "desk-like": it mixes instruments, risk measures, and execution realities. If you are not yet fluent in fixed income jargon, use this roadmap rather than trying to read Chapter 52 straight through.

- **If CDS is new:** read Chapter 38 (CDS mechanics) and Chapters 39–40 (credit events + auctions) first.
- **If Greeks / risk reports are new:** read Chapter 43 (CS01/RPV01, VOD/JTD, recovery risk, theta) before Section 52.2.
- **If indices are new:** skim Chapters 45–47, then return to Sections 52.5 and Examples 5–10.
- **If tranches/correlation are new:** read Chapters 49–51 first. On a first pass, you can skip Section 52.6 and still get a lot of value from the basis/roll/capital-structure sections.

A good first-pass route through Chapter 52:

1. **Section 52.2** (risk-measure vocabulary) and **Section 52.3** (Strategy Card discipline)
2. **Examples 1–4** (basis packages, proxy hedges, curve trades)
3. **Examples 6–10** (index carry/rolldown/roll + a desk-style P&L explain)
4. Return to **Section 52.6** (tranches) once the earlier pieces feel natural

As you read, keep one goal in mind: when P&L surprises you, you want a short, checkable list of residual risks to investigate (funding, liquidity, recovery/final price, basis, correlation/tail).

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

**Desk note:** clean MTM will often show jumps around coupon dates; the cash coupon/accrual flows offset this in the “full” P&L. When a hedge looks “right” in risk but “wrong” in P&L, this clean-vs-full distinction is one of the first checks.

### CS01 Sign Convention (Explicit)

Definition used here:

$$CS01 = \frac{\partial PV}{\partial S} \times (1 \text{ bp})$$

where $S$ is the quoted/par spread being bumped (single-name CDS spread, index spread, tranche spread—explicitly stated each time).

- **Long protection CDS:** $PV$ increases when $S$ widens $\Rightarrow$ CS01 $\gt 0$.
- **Short protection CDS:** CS01 $\lt 0$.
- **Long cash bond:** $PV$ decreases when credit spread widens $\Rightarrow$ bond "credit CS01" $\lt 0$ (if you measure it as $\Delta PV$ for $+1$ bp spread).

### Jump-to-Default (JTD) Sign Convention

We measure JTD as the instantaneous PV jump at default (a scenario jump, not an infinitesimal derivative):

$$JTD \equiv PV_{\text{post-default}} - PV_{\text{pre-default}}$$

For long protection CDS, JTD is typically positive (you receive protection payment net of accrued premium). This is the value-on-default (VOD) framework in the sources.

---

## 52.1 Conventions and Setup

### Conventions Used in This Chapter

(Quoting regime assumptions; clean/dirty; bp vs decimals; per-$1mm scaling)

- Spreads are quoted in bp/year; valuation uses decimals with $1 \text{ bp} = 10^{-4}$.
- **Single-name CDS valuation identity (risk measure anchor):** the mark-to-market of a standard running-spread CDS can be written as

$$V(t,T) = (S(t,T) - S_0) \cdot RPV01(t,T)$$

(per unit notional), where $RPV01$ is the risky PV01 of the premium leg.

### Index Contracts (Mechanics Assumed)

- Index coupon $C$ is set near fair value (multiple of 5 bp), paid quarterly on an Actual/360 basis; an upfront payment $U_I$ is exchanged at settlement; index rolls every six months.
- **Default handling for index:** on a constituent credit event, the protection seller pays loss on the defaulted name and the index notional is reduced proportionally (simplified as $1/M$ per default in these notes, consistent with the cited mechanics).
- **Scaling:** when we report CS01/JTD/RecSens, we always state "per $1mm" or "per $10mm".

**Mental model (important):** CDS indices trade with a *fixed coupon* and an *upfront* (similar to a bond trading with a fixed coupon and a price). The market still quotes a spread (analogous to a bond’s yield). At trade time, the present value of the difference between paying the **quoted spread** and paying the **fixed coupon** is exchanged as an upfront so the trade is fair on day one:

- If the quoted spread is **above** the fixed coupon, a **protection buyer** (long protection) typically pays an upfront to the protection seller.
- If the quoted spread is **below** the fixed coupon, the protection buyer typically receives an upfront.

If that sign convention feels confusing, re-check the CDS MTM identity in Chapter 43 and the index quoting mechanics in Chapters 45–46.

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

Before we go strategy-by-strategy, align on the risk measures that show up in nearly every desk conversation. They answer different "what if" questions:

- **Small moves (linear):** CS01 (credit spreads), DV01 (rates)
- **Jump events:** JTD/VOD (default today), recovery/final price
- **Model/regime knobs:** index basis (quoted vs intrinsic), correlation (tranches)

If you already know these, skim this section and jump to Section 52.3.

#### Quick Desk Translation (What Each Number Really Means)

| Measure | Plain-English question it answers | Units | What it captures (and what it misses) | Deep dive |
|---------|-----------------------------------|-------|----------------------------------------|-----------|
| CS01 | “If the quoted spread moves +1 bp, how much does my PV move?” | USD/bp | Great for small spread moves; does **not** capture default jumps or nonlinearities | Chapter 43 |
| Rates DV01 | “If the risk-free curve moves +1 bp, how much does my PV move?” | USD/bp | Matters for cash bonds and swaps; does **not** hedge credit spread moves | Chapters 12 and 14 |
| JTD (VOD) | “If the name defaults right now, what is my instantaneous PV jump?” | USD | Captures the discontinuity at default; cannot be hedged by a small CS01 match | Chapters 39–40 and 43 |
| Rec01 | “If recovery/final price is 1% higher, what happens to PV (especially in default scenarios)?” | USD per 1% | Recovery assumptions dominate default P&L; often needs scenario sweeps | Chapter 43 |
| Index basis (quoted vs intrinsic) | “If the index moves but my constituents don’t (or vice versa), what breaks?” | bp or USD | Key for proxy hedges and index-vs-single-name P&L explains | Chapters 46–47 |
| Tranche PV01 / systemic DV01 / Corr01 | “If tranche spreads / all-name spreads / correlation move, how does tranche PV change?” | USD/bp or USD per 1% | Useful but model-dependent; must be stress-tested for clustering and convexity | Chapters 50–51 |

### 1.1 CS01 / Spread DV01 (and What "Spread" Is Being Bumped)

#### Formal Definition (Units and Sign)

For any instrument valued as $PV(S, \cdot)$, define

$$\boxed{CS01 \equiv \frac{\partial PV}{\partial S} \times 10^{-4} \quad \text{(USD per 1 bp)}}$$

What spread is bumped must be specified:

- **Single-name CDS:** bump the market par spread $S(t,T)$.
- **Index CDS:** bump the quoted index spread $S_I(t,T)$ or (alternatively) bump constituents and re-imply an "intrinsic" index spread (two different risk views; mixing them is a pitfall).
- **Tranche:** bump the tranche market spread (or tranche coupon/price quote, depending on convention).

#### Source-Backed Anchor (Single-Name CDS)

Using the CDS MTM identity

$$V(t,T) = (S(t,T) - S_0) \cdot RPV01(t,T),$$

a small spread move gives

$$dV \approx RPV01(t,T) \\, dS,$$

so per unit notional:

$$\boxed{CS01_{\text{CDS}} \approx RPV01(t,T) \times 10^{-4}}$$

#### Intuition

CS01 is the "linear spread risk": how much PV changes for small spread moves.

#### How It Appears in Practice

Risk systems compute $RPV01$ and CS01 by maturity; traders/risk managers size hedges to neutralize CS01 (or bucketed CS01).

**Common beginner mistakes (worth watching for in P&L explains):**

- Treating “CS01” as a universal object without specifying what was bumped (single-name par spread vs index quoted spread vs tranche quote).
- Matching *total* CS01 but leaving big *curve-shape* exposure (bucketed CS01 mismatch).
- Declaring a position “hedged” because CS01 is matched while ignoring JTD/recovery/funding/basis (the usual sources of overnight surprises).

If these distinctions feel unfamiliar, Chapter 43 is the prerequisite for the rest of this chapter.

---

### 1.2 Rates DV01 (Only as Needed for Cash Bond Legs)

#### Formal Definition

Let $r$ represent the relevant risk-free curve level (or yield). Define

$$\boxed{DV01_r \equiv \frac{\partial PV}{\partial r} \times 10^{-4} \quad \text{(USD per 1 bp)}}$$

#### Intuition

Rates DV01 isolates the interest-rate exposure of funded cash instruments (bonds) and some swap legs.

#### Practice

Duration/DV01 hedging is standard; it immunizes against small parallel shifts but not non-parallel curve changes (key-rate risk).

---

### 1.3 Jump-to-Default (JTD) Exposure

#### Formal Definition

For a position with pre-default PV $PV$ and post-default PV $PV'$:

$$\boxed{JTD \equiv PV' - PV \quad \text{(USD)}}$$

#### Source-Backed CDS Expression (Value-on-Default / VOD)

For a CDS, the value-on-default is expressed (per unit notional) as:

$$VOD = (1 - R - \text{Accrued Premium}) - (S(t,T) - S_0) \cdot RPV01(t,T)$$

(long protection sign).

**Mapping:** in these notes, JTD = VOD × Notional.

#### Intuition

CS01 is "small move"; JTD is the discontinuity when default happens.

**Concrete intuition:** If you are long protection on USD 5mm and recovery is 40%, the protection payment is about $(1-R)N = 0.60 \times 5{,}000{,}000 = 3{,}000{,}000$ USD (before accrued premium and any pre-default MTM). A CS01-matched hedge can look “tight” for day-to-day spread noise while still leaving a very large one-day default jump risk.

#### Practice

Hedging CS01 does not hedge JTD (especially single-name default vs index hedge).

---

### 1.4 Recovery / Final-Price Sensitivity

#### Formal Definition

Recovery sensitivity (per unit notional) can be measured as

$$Rec01 \equiv \frac{\partial PV}{\partial R} \times 0.01 \quad \text{(USD per 1\\% recovery)}$$

#### Source-Backed CDS Recovery Sensitivity

A recovery rate sensitivity can be expressed in terms of the CDS value and $(1 - R)$ (book's recovery DV01 expression).

#### Final Price

CDS protection leg can be cash-settled with payment based on final price of the reference obligation, determined by dealer poll/auction; payoff is based on face value minus recovery price.

If the auction/final-price process is unfamiliar, Chapter 40 is the practical reference.

#### Intuition

Recovery is a second key state variable in default scenarios; many "basis" and "hedge" surprises come from mismatched recovery assumptions.

---

### 1.5 (For Indices) Series/Roll Basis and "Intrinsic vs Quoted" Basis

If you only remember one thing about CDS indices: there are two consistent ways to talk about “the index level,” and mixing them creates P&L confusion.

- **Quoted (top-down) view:** treat the index like a single CDS and quote a flat “index spread” (a market convention).
- **Intrinsic (bottom-up) view:** price the index as the sum/average of its constituents’ CDS values.

The gap between these is the **index basis** (Chapters 46–47 cover this in depth). It matters because many desks hedge single-name or portfolio risk with an index. If the index basis moves, a hedge that was “CS01 matched” can still produce P&L breaks.

#### Intrinsic Value vs Market Value

O'Kane provides the precise intrinsic valuation framework. The intrinsic upfront value of an index (viewed as an equally weighted portfolio of constituent CDS) is:

$$\boxed{V_I(t) = \frac{1}{M} \sum_{m=1}^{M} (S_m(t,T) - C(T)) \cdot RPV01_m(t,T)}$$

The market upfront value using a flat index curve is:

$$\boxed{U_I(t) = (S_I(t,T) - C(T)) \cdot RPV01_I(t,T)}$$

O’Kane calls this the **intrinsic** view because it prices the index as the sum (average) of its constituent CDS values.

#### Index Basis

The **index basis** is the gap between what the market quotes for the index (via a flat index curve) and what you would infer if you valued the index as the sum of its constituents (the **intrinsic** view). In the literature, this basis can be positive or negative and can vary by maturity and index.

Key source-backed drivers to keep in mind:

1. **Contract differences (e.g., restructuring clause):** for example, the North American CDX index protection leg historically excluded restructuring (a “No‑Re” style trigger), while many US single‑name CDS contracts used Modified Restructuring (“Mod‑Re”). If the index and single‑name contracts do not have identical credit‑event terms, you should expect a mechanical basis component.
   If restructuring clauses are new, see Chapter 39.

2. **Liquidity and price discovery:** the index market is often more liquid than many single names. The literature notes that the index can embed a different liquidity premium and can “lead” single‑name spreads, especially in widening markets when investors hedge illiquid cash credit.

3. **Replication vs quoting conventions:** the intrinsic view requires all constituent curves; the quoted index view uses a flat index curve. Even if you model everything perfectly, you can still get a basis because the quoting object (index curve) is not the same object as the set of single‑name curves.

#### The Portfolio Swap Adjustment

To reconcile intrinsic and quoted index views, O’Kane describes a **portfolio swap adjustment**: adjust the individual issuer curves so that the portfolio‑implied index matches the market‑quoted index. The book emphasizes that the exact adjustment is somewhat arbitrary, but highlights practical desiderata like preserving each issuer’s term‑structure shape and relative ranking and avoiding arbitrage artifacts.

**How to think about this (beginner-friendly):**

1. Start with your best estimate of each constituent’s CDS curve.
2. Price the index bottom-up to get an intrinsic upfront (or intrinsic spread).
3. Compare that to the market-quoted index level (top-down).
4. Apply an adjustment rule to the constituent curves so the bottom-up price matches the top-down quote.

This is primarily a **model consistency** step (important for tranche pricing and some risk decompositions). From a trading/risk perspective, the takeaway is simpler: *index vs constituent hedges contain an extra moving part*, and you should monitor it explicitly (see Chapters 46–47).

Portfolio swap adjustment is implemented differently across systems (e.g., spread multipliers vs hazard‑rate scaling; global vs tenor‑by‑tenor; and varying constraints). Confirm your system’s method before using intrinsic/quoted decompositions or hedge weights operationally.

#### Series/Roll Basis

Indices roll every six months; on-the-run liquidity and maturity reset can create price/spread differences between old and new series.

#### Intuition

- "Quoted vs intrinsic" is a model/replication gap driven by restructuring clauses, liquidity, and market dynamics.
- "Series basis" is a liquidity + contract cohort gap.
- The portfolio swap adjustment is required for any tranche or structured product pricing built on the index.

---

### 1.6 (For Tranches) Tranche PV01, Correlation Sensitivity, and Tail/Default Clustering Scenario Risk

If tranches are new, pause here and read Chapters 49–51 first. The rest of this chapter assumes you are comfortable with:

- attachment/detachment and the tranche loss function (Chapter 49),
- what “correlation” means in tranche pricing (Chapter 50),
- systemic vs idiosyncratic deltas/gammas, Corr01, and VOD/JTD thinking for tranches (Chapter 51).

**Why this is hard (and why it matters):** in single-name CDS, “delta” usually means CS01. In tranches, there are multiple deltas depending on *what you bump* (tranche quote vs all-name spreads vs one name vs correlation). Many strategy mistakes start with mixing these objects.

#### Tranche PV01

The tranche risky PV01 is defined analogously to CDS premium-leg PV01, using expected discounted outstanding tranche notional; the source gives a tranche $RPV01$ expression and defines tranche PV01 as the sensitivity to 1 bp spread change.

#### Correlation Sensitivity

"Correlation 01" is defined as PV change for a 1% increase in correlation parameter, with sign depending on tranche (equity vs senior behave differently).

**Intuition (words, not math):** correlation changes *how losses arrive* (scattered defaults vs clustered waves). In the extreme of very high correlation, outcomes look more “all-or-nothing,” which tends to make senior riskier (more tail exposure) and equity less risky (less “death by a thousand cuts”). That is why equity and senior tranches can have opposite Corr01 signs.

#### Tail/Clustering Risk

Tranche risk is nonlinear in portfolio loss. Tranche loss fraction:

$$\boxed{L_{[A,B]} = \frac{1}{B - A} \left( \max(L - A, 0) - \max(L - B, 0) \right)}$$

**Interpretation:** tranche loss is 0 until portfolio loss reaches $A$; it ramps linearly between $A$ and $B$; and it is fully written down (loss fraction 1) once portfolio loss exceeds $B$.

Clustering scenarios can swamp small-spread hedges (PV01 hedges fail because payoff is driven by realized losses).

---

### 1.7 Carry and Rolldown Definitions (Cash Bonds; CDS Indices)

#### Cash Bond Carry/Rolldown (Brief)

For a bond portfolio, carry is the coupon accrual minus financing cost; rolldown is price change from moving "down" a sloped curve as time passes.

#### CDS / Index / Tranche Carry and Theta

O’Kane defines (for tranches, and the idea carries over to indices):

- **Carry:** the daily accrual of the contractual coupon/premium (even though it is paid quarterly). A key intuition is that premium accrued since the last payment date is still owed if a default occurs before the next payment date, so it is sensible to think in “daily accrual” terms.
- **Theta:** the daily change in the **full price** holding spreads/curves fixed and assuming no default over the day. For a long protection position, theta is the change in the protection leg minus the change in the premium leg.

Qualitative risk-report pattern from the source: for a **protection buyer**, carry is typically **negative** (you pay premium), and the absolute carry tends to be largest for the most junior tranche because it has the highest contractual spread. Theta for long protection is often negative because (i) there is one day less of protection remaining and (ii) the premium leg cashflows are one day closer (higher PV).

If “theta” still feels abstract, read the CDS theta discussion in Chapter 43; for tranche carry/theta intuition in a risk-report format, Chapter 51 is the best companion chapter.

#### Rolldown Component

Theta includes a **rolldown** component: if the spread curve is upward sloping, as time passes you effectively move to a shorter maturity point on the curve, which (all else equal) tends to reduce the implied par spread.

For a position with unchanged spreads and curve shape:
- **Carry** = coupon accrual component (deterministic)
- **Rolldown** = change in PV from aging along the curve (part of theta)
- **Theta** = total time decay = rolldown + other time effects (maturity shortening, etc.)

#### What We Do in This Chapter

We use carry + rolldown + spread move as a P&L explain framework for index positions, treating "rolldown" as the component of theta due to curve slope (derived from the theta definition).

---

### 1.8 Liquidity/Funding/Financing Risk (Conceptual, Only as Supported)

#### Bond–CDS Basis Drivers (Source-Backed)

The CDS–cash basis is defined as:

$$\text{CDS basis} = \text{CDS spread} - \text{Bond Libor spread}$$

with discussion that the bond spread choice is typically asset swap spread for fixed-rate bonds near par.
See Chapter 27 for asset swap spread mechanics and why "premium bonds" create basis traps.

**Drivers for persistence include:** funding differences (unfunded CDS vs funded bonds), delivery option, technical default, loss-on-default mismatch, premium accrued at default, and market liquidity/supply-demand factors.

#### Intuition

"Cheap" vs "rich" often reflects funding/liquidity optionality rather than pure expected default loss.

---

## 52.3 The Governing Principle: Strategy = Exposures + Hedges + Failure Modes

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

## 52.4 Strategy Family A: Basis Strategies

If you want the longer “cash vs swap vs CDS” foundations behind these trades, Chapter 27 is the companion chapter (asset swaps, swap spreads, CDS-bond basis, and the premium-bond trap). Chapter 43 is the prerequisite for the risk-measure language (CS01/JTD/recovery/theta).

### A1) Bond–CDS Basis Framework (Cash vs Synthetic)

The bond-CDS basis—the spread difference between a CDS and the equivalent cash bond—is one of the most studied phenomena in credit markets. O'Kane defines the CDS basis as:

$$\boxed{\text{CDS basis} = \text{CDS spread} - \text{Bond Libor spread}}$$

where the "bond Libor spread" is typically measured using the bond's asset swap spread when the bond is close to par (for fixed-rate bonds). A positive basis means CDS protection is more expensive than the bond spread suggests; a negative basis means protection is cheap relative to cash.

The basis can persist for extended periods because CDS and bonds are fundamentally different contracts. O'Kane divides the drivers into **fundamental factors** (contractual differences) and **market factors** (trading dynamics). Treat this list as a checklist when a "fully hedged" basis position surprises you.

#### 3.1.1 Fundamental Factors Driving the Basis

O'Kane identifies six fundamental factors that create structural differences between CDS and bond spreads:

1. **Funding**: CDS is economically unfunded (no bond principal to finance), while holding a bond requires financing/balance sheet. Investors with different funding costs (relative to Libor) naturally prefer one market over the other, and the basis reflects the marginal funding level.

2. **The delivery option (physical settlement)**: With physically settled CDS, the protection buyer can choose which deliverable obligation to deliver after a credit event. This cheapest-to-deliver option makes protection more valuable. O'Kane gives a concrete intuition: if one deliverable trades at 43 and another at 37, the buyer can capture the 6-point difference by delivering the cheaper bond.

3. **Technical default / credit-event definition**: CDS credit events can be broader than plain bond payment default (for example, restructuring depending on documentation). That extra trigger risk affects CDS pricing relative to cash.

4. **Loss-on-default mismatch**: CDS protection pays $(1-R)$ of notional (per 100), while the loss on a cash bond purchased at full price $P$ is $P-R$. If $P \neq 100$, notional matching by CS01 can leave a JTD mismatch; JTD matching is a separate sizing problem.

5. **Accrued premium/coupon around default**: In CDS, accrued premium is typically settled at default; in cash bonds, coupon cashflows stop and accrued can be lost. This small difference matters in tight "basis carry" calculations and in default P&L explain.

6. **CDS spreads cannot be negative**: Unlike asset swap spreads (which can go negative for very high-quality credits like Treasuries), CDS spreads must be positive to compensate for default risk and transaction costs.

#### 3.1.2 Market Factors Driving the Basis

O'Kane lists six market factors that create basis dynamics:

1. **Relative liquidity**: CDS liquidity clusters at standard tenors (3/5/7/10Y, often quoted on IMM dates) while bonds are liquid in specific issues. Liquidity and maturity mismatches can move basis.

2. **Synthetic CDO technical**: When dealers issue synthetic CDO risk, they can hedge by trading CDS on many constituents, creating mechanical spread pressure.

3. **New issuance/loans**: Corporate bond issuance and bank loan origination can lead to CDS hedging flows that move CDS spreads and cash spreads differently.

4. **Convertible issuance**: Convertible arbitrage and hedging can create protection-buying pressure in CDS.

5. **Demand for protection / shorting asymmetry**: It is operationally easier to get short credit via CDS (buy protection) than short a cash bond, so CDS can move "first" on negative news.

6. **Funding and liquidity regimes**: Even if CDS is economically unfunded, the cash-bond leg is exposed to financing and balance-sheet constraints; when that regime shifts, basis can gap.

O'Kane's practical point is that these interacting effects can make CDS and cash spreads diverge, and it is hard to assign a single dominant cause in real time.

#### Strategy Card: A1 — Bond–CDS Basis Package (Risk-First Framing)

> **In desk language:** A1 is “cash vs synthetic.” You try to hedge away the obvious risks (rates DV01, sometimes CS01), so the P&L that remains is usually about **basis + funding + event mechanics**, not about day-to-day spread noise.
>
> - If the package “mysteriously” loses money, first ask: did **funding/repo** move (Example 1B), did the **basis** move, or did a **default/auction/recovery** assumption change?
> - A CS01 match is not enough: the default payoff depends on bond price vs recovery ($P-R$) versus CDS payoff ($1-R$).

##### Objective (Conceptual; No Forecasts)

Isolate and measure the CDS–cash basis while understanding which factors are driving any observed divergence. In O’Kane’s framing, the basis can be actively traded as a relative‑value position once you are explicit about the residual risks.

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
| $CS01_{0\text{–}3}$ (USD/bp) | $\partial PV / \partial S \times 1\text{bp}$ | usually 0 | usually 0 | maturity placement |
| $CS01_{3\text{–}7}$ (USD/bp) | 5Y risk mostly here | $\lt 0$ (long bond) | $\gt 0$ (long prot) | can offset if sized |
| $CS01_{7\text{–}10}$ (USD/bp) | long-end bucket | 0 | 0 | — |
| Rates DV01 (USD/bp) | $\partial PV / \partial r \times 1\text{bp}$ | typically $\neq 0$ | ~0 | residual rates unless hedged |
| JTD (USD) | PV jump at default | $-(P - R) \times N$ | $+VOD \times N$ for long prot | can be matched, but not automatic |
| Rec01 (USD per 1%) | $\partial PV / \partial R \times 1\\%$ | depends on bond recovery view | CDS recovery sensitivity | mismatches matter |
| Basis sensitivity | PV change when CDS and bond spreads move differently | yes | yes | this is the residual you keep |

##### Hedge Set and Hedge Ratios (Show Math)

**Rates hedge (DV01 neutralization):**

$$N_{\text{rates hedge}} = -\frac{DV01_r^{\text{bond}}}{DV01_r^{\text{hedge instr}}}$$

**Credit hedge (CS01 match):**

$$N_{\text{CDS}} = -\frac{CS01_{\text{bond}}}{CS01_{\text{CDS per USD  notional}}}$$

**Event hedge (JTD match):** motivated by the loss-on-default mismatch discussion (bond loss $P - R$ vs CDS loss $1 - R$).

##### Scenario Test Suite

- Parallel spread move (bond and CDS widen together).
- Dispersion (CDS widens, bond unchanged; or vice versa).
- Default event: vary recovery/final price; include accrued premium at default.
- Rates move: bond yield curve shift with and without DV01 hedge.
- (Optional) Delivery option/restructuring clause differences (qualitative).

##### Failure Modes

The basis trade can fail in multiple ways, corresponding to the fundamental and market factors O'Kane identifies:

**Funding-Related Failures:**
- **Funding/repo shocks**: The bond leg is funded; CDS is (economically) unfunded. In the source’s framing, a CDS position does not require financing the cash bond, so a bond‑CDS package carries an embedded **funding asymmetry**. If your funding costs spike, the bond leg becomes expensive to carry while the CDS leg does not “feel” the same repo stress in the same way. That asymmetry can overwhelm a small positive carry.
- **Repo fails or specials**: If the specific bond becomes hard to finance (goes "special"), carrying the cash leg becomes punitive. This is a funding risk that CDS does not face.

**Contract-Related Failures:**
- **Delivery option / cheapest-to-deliver**: after certain credit events (especially restructuring), the protection buyer can choose which deliverable obligation to settle with. If you are hedging a specific bond, this creates a mismatch: the delivered asset may not be the one you expected, so the CDS leg can realize a different loss than the cash bond you are hedging.
- **Restructuring clause mismatch**: CDS spreads depend on the restructuring clause because it changes the trigger set and the deliverable set. Trading a Mod‑Re single‑name CDS against a No‑Re index (or vice versa) creates basis risk to restructuring‑related events.
- **Loss-on-default mismatch**: CDS pays $(1-R)$ on notional; a cash bond bought at a full price $P$ has loss $P-R$. If your bond is at 105 and defaults at 40% recovery, you lose 65 points but CDS only pays 60 per 100 notional. If your bond is at 95, you lose 55 but CDS pays 60—now you are over‑hedged in default.

**Market-Related Failures:**
- **Synthetic CDO technicals**: dealer hedging of synthetic CDO issuance can involve selling protection on many names, mechanically tightening CDS spreads and compressing the basis.
- **New issuance and hedging flows**: new bond issuance and loan hedging flows can push CDS and cash markets in different directions over short horizons.
- **Convertible issuance**: convertible hedging flows can drive protection buying and widen CDS spreads without an immediate, matching move in cash bonds.
- **Liquidity divergence**: CDS is often the easier instrument to go short credit quickly. In stress, that can make CDS spreads gap wider than bond spreads, producing large basis moves.

**Monitoring checks (thresholds are desk-specific):**
- CS01 drift after execution and after spread moves (hedge staleness).
- JTD mismatch driven by bond price ≠ par and recovery/final-price assumptions.
- Funding and liquidity regime shifts (repo terms changing, bid/offer widening).

##### Implementation Checklist + Verification Tests

- Reprice each leg under the same discounting assumptions where applicable (and document differences).
- Verify signs: long protection CS01 $\gt 0$; long bond credit CS01 $\lt 0$.
- Bump stability: CS01 should be stable for $\pm 1$ bp bumps; if not, you are in nonlinear regime.
- Event test: run default today with recovery sweep.

#### Negative Basis: Why It Persists

A "negative basis" means the CDS spread trades *inside* (below) the bond asset-swap spread: the synthetic market prices credit risk cheaper than the cash market. This seemingly arbitrageable condition can persist for extended periods because of structural demand dynamics.

**Desk intuition (source-aligned):** CDS is unfunded while a bond position must be financed. That creates a clientele effect: participants with cheaper funding can prefer cash bonds, while those with more expensive funding can prefer CDS. In addition, technical flows like synthetic CDO issuance can tighten single‑name CDS spreads via dealer hedging, which can contribute to a more negative basis.

#### Positive Basis: The Squeeze

A "positive basis" means CDS spreads trade *outside* (above) bond spreads. O'Kane's market factors explain why: demand for protection (shorting credit is easier via CDS), CDO dealer hedging, convertible arbitrage flows, and new issuance effects all tend to push CDS spreads wider relative to bonds.

The dangerous scenario is the **positive basis squeeze**: a trader who is short the basis (short CDS, long bond) faces losses when the basis widens further. In stress:

1. **Protection demand surges**: if negative news hits a credit, it is often operationally easier to express a bearish view via **buying CDS protection** than by shorting cash bonds. That flow can push CDS wider faster than cash bonds, widening the basis.
2. **Repo funding spikes**: The bond leg becomes expensive to carry as repo rates rise (Example 1B illustrates this).
3. **Forced unwinds**: Margin calls on the CDS leg force liquidation at the worst time, crystallizing losses.

**Stress-test mindset:** basis trades are funding- and liquidity-contingent. When funding terms or market liquidity shift, the P&L distribution changes qualitatively; a “CS01‑matched” package can still experience large adverse moves.

---

### A2) Single-Name vs Index (Proxy Hedge / Basis)

#### Strategy Card: A2 — Proxy Hedge: Single-Name CDS Hedged by an Index

> **In desk language:** A2 is “hedge the tape.” You keep the single-name view but try to remove broad-market drift with an index hedge.
>
> - What remains is usually **idiosyncratic/default jump risk** (a single default is huge for the name and small for the index), plus **index-vs-name basis** and **roll/series** effects.
> - If you want an intuition check, read Example 3 before you worry about any math: it shows how a clean CS01 match can still leave a big default scenario.

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
| $CS01_{3\text{–}7}$ (USD/bp) | dominant | dominant | can be matched |
| JTD (USD) | large (single default) | small per-name (diversified) | not hedged by CS01 match |
| Rec01 (USD/1%) | meaningful | meaningful | can mismatch |
| Basis sensitivity | single vs index composition | yes | residual "proxy basis" |
| Roll/series basis | none | yes | index-specific residual |

##### Hedge Set and Hedge Ratios (Show Math)

**CS01 match:**

$$N_{\text{index}} = -\frac{CS01_{\text{single}}}{CS01_{\text{index per USD  notional}}}$$

**If a beta $\beta$ is imposed:**

$$N_{\text{index}} = -\frac{\beta \cdot CS01_{\text{single}}}{CS01_{\text{index per USD  notional}}}$$

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

### A3) CDS Curve Steepener / Flattener

Credit curve trades express a view on the *shape* of a single name's CDS spread term structure rather than its level. They are a natural extension of the relative value framework from Chapter 44.

#### 3.3.1 Curve Shapes and What They Signal

The source discusses three representative CDS curve shapes: flat, upward‑sloping, and steeply inverted. The inverted curve case is especially relevant for stressed/high‑yield names:

- **Upward-sloping (typical IG):** Long-term default probability exceeds short-term. The hazard rate increases with tenor. This is normal for investment-grade credits where the risk of eventual deterioration exceeds near-term default risk.
- **Inverted (distressed HY):** The market prices near‑term default as the dominant risk. Conditional on surviving the near term, the longer‑dated hazard can be perceived as lower, which can pull longer‑dated spreads below the front end.

**Desk intuition:** an inverted curve is often interpreted as a “survival bet.” For example (toy numbers), a name trading 800bp in 1Y and 500bp in 5Y is saying the market is heavily focused on near‑term default risk. A curve steepener (long short‑dated protection, short longer‑dated protection) can monetize normalization *if the name survives*, but if the name defaults, both legs trigger and the net depends on notional sizing, premium accrual, and recovery assumptions.

#### 3.3.2 Forward CDS Curves

O'Kane shows how spot CDS curves map to forward curves (Table 9.1). For an upward-sloping curve, the forward curve is higher than the spot curve—forward-starting protection is more expensive, reflecting the market's expectation of higher future default risk. For an inverted curve, the forward curve is lower than spot, reflecting declining expected hazard rates.

#### 3.3.3 Curve Arbitrage Bounds

O'Kane derives an arbitrage lower bound for inverted curves. For a curve starting at 800 bp at 6 months, the approximate lower bound is:

$$S_m \gtrsim S_{m-1} \left(\frac{T_{m-1}}{T_m}\right)$$

O'Kane works out the exact bounds in his examples; the takeaway is that a severely inverted curve can violate no-arbitrage constraints implied by the CDS payoff structure and the non-negativity of survival probabilities. If you see “impossible” forward/default-implied behavior in a curve build, treat it as a red-flag for curve construction inputs (quotes, recovery, interpolation) before you treat it as tradable edge.

#### Strategy Card: A3 — CDS Curve Steepener/Flattener

> **In desk language:** A3 is “trade the curve shape, not the level.” You make money if the front end and back end move *differently* (steepen/flatten), and you try to remove pure “parallel spread” risk with CS01 matching.
>
> - The most common surprise is **default/JTD mismatch**: CS01-neutral sizing usually requires unequal notionals, so a default can create a large residual jump even when day-to-day spread P&L is near zero.
> - If you’re new to credit curves and hazard rates, Chapter 42 is the prerequisite; if you’re new to CDS risk measures, Chapter 43 is.

##### Objective (Conceptual; No Forecasts)

Express a view on the shape of a single name's credit curve: steepening (short end widens relative to long end) or flattening (long end widens relative to short end). Alternatively, express a "survival bet"—the steepener profits if the name survives and the curve normalizes.

##### Instruments and Position Construction

- **Steepener:** Buy protection at short tenor (e.g., 3Y), sell protection at long tenor (e.g., 5Y or 10Y).
- **Flattener:** The reverse.
- Size the trade CS01-neutral to isolate the curve shape view from parallel spread moves.

##### Exposure Decomposition (Units Explicit)

| Exposure (units) | Short leg | Long leg | Net |
|------------------|-----------|----------|-----|
| $CS01_{3Y}$ (USD/bp) | $\gt 0$ (long prot) | 0 | positive |
| $CS01_{5Y}$ (USD/bp) | 0 | $\lt 0$ (short prot) | negative |
| $CS01_{\text{net}}$ (USD/bp) | — | — | ~0 by construction |
| JTD (USD) | positive (receive prot pmt) | negative (pay prot pmt) | net depends on accrual |
| Rec01 (USD/1%) | meaningful | meaningful | partial offset |

##### Hedge Set and Hedge Ratios (Show Math)

**CS01-neutral sizing:**

$$\boxed{N_{\text{long}} = N_{\text{short}} \times \frac{RPV01_{\text{short}}}{RPV01_{\text{long}}}}$$

For a 3Y/5Y steepener with $RPV01_{3Y} = 2.8$ and $RPV01_{5Y} = 4.5$:

$$N_{5Y} = N_{3Y} \times \frac{2.8}{4.5} = 0.622 \times N_{3Y}$$

##### Scenario Test Suite

- **Parallel widening:** CS01-neutral construction should produce ~0 P&L.
- **Curve steepening:** 3Y widens more than 5Y → steepener profits.
- **Curve flattening:** 5Y widens more than 3Y → steepener loses.
- **Default event:** Both legs trigger; net JTD depends on notional sizing, accrual at default, and any RPV01 change.
- **Curve slope reversal:** Inverted curve normalizes → steepener profits (this is the "survival bet").

##### Failure Modes

- **Default jump:** If the name defaults, both legs trigger and the JTD residual (due to notional mismatch for CS01-neutrality) can dominate all spread P&L.
- **Curve parallel shift dominates:** Even with CS01-neutral sizing, convexity differences between tenors create residual P&L on large parallel moves.
- **Liquidity at short end:** Short-dated CDS can be less liquid; bid-offer may consume expected curve P&L.
- **Roll and coupon convention mismatch:** Different coupon conventions at different tenors create small but real basis effects.

##### Implementation Checklist + Verification Tests

- Verify CS01-neutral by bumping all spreads $\pm 1$ bp parallel; net P&L should be near zero.
- Bump curve slope: bump short end $+1$ bp, long end $-1$ bp (and vice versa); verify correct sign.
- Run default scenario with recovery sweep; verify JTD residual.
- Check bid-offer at both tenors; ensure expected curve P&L exceeds round-trip costs.

---

## 52.5 Strategy Family B: Carry, Rolldown, and Roll in CDS Indices

If CDS indices are new, Chapters 45–47 (index structure, intrinsic vs quoted, hedging) are the prerequisites. The goal here is not to “predict spreads,” but to build a clean desk-style decomposition of what moved and why.

### Definitions (Index-Specific, Risk-First)

#### Index Upfront Mechanics (Source-Backed)

Index trades with a fixed coupon $C$ and an upfront payment at settlement; coupon is paid quarterly on Actual/360; index rolls every six months.

**Beginner intuition:** the coupon is standardized (like a bond coupon), while the market quotes a “spread” that moves every day (like a yield). The upfront is the one-time cash amount that reconciles the fixed coupon with the current market spread so the trade is fair on day one. This is why you will often hear traders talk about an index “price” as well as an index “spread.”

#### Carry (Index)

In this chapter, "carry" means the deterministic premium accrual/coupon cashflow component over a horizon (plus any deterministic accrual conventions), consistent with the sources' carry definition as coupon accrual (for tranche) and general swap cashflow logic.

#### Rolldown (Index)

"Rolldown" means the component of P&L from aging along a sloped spread curve holding the curve fixed (a component of theta). This is directly aligned with the theta definition that includes roll down/up effects for upward/downward sloped spread curves.

---

### B1) Index Carry/Rolldown Holding-Period P&L (Risk Decomposition, Not a Recommendation)

#### Strategy Card: B1 — Index Holding-Period Decomposition ("Carry + Rolldown + Spread Move + Events")

> **In desk language:** B1 is the desk’s default “P&L explain” for an index position. Every day you want to be able to say: *how much was carry (accrual), how much was rolldown (aging along the curve), how much was the spread move (curve shift), and what was event-driven (defaults, roll, execution)?*
>
> If you’re new, start with Examples 6–10. They are less about “trading” and more about learning to read a credit P&L like a desk.

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

$$N_{\text{hedge}} = -\frac{CS01_{\text{target}}}{CS01_{\text{hedge}}}$$

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

The index roll is a critical event in CDS index markets: indices roll every six months, and investors who want to stay “on‑the‑run” typically sell the old series and buy the new one.

#### 3.4.1 What Drives Roll P&L

O'Kane identifies two primary sources of P&L impact during the roll:

1. **Composition changes**: the new series can differ because names are removed and replaced based on credit-quality and liquidity criteria. This can improve or worsen perceived credit quality; liquidity-driven replacements are not guaranteed to tighten spreads.

2. **Maturity extension**: the new series is about six months longer maturity than the previous series. With upward-sloping credit curves, longer maturity tends to imply a wider fair spread, all else equal.

These two effects can work in opposite directions. A credit-upgraded new index with a longer maturity may trade at the same spread as the old index if the composition improvement offsets the curve steepness.

#### 3.4.2 The Series Basis

The difference between the old (off-the-run) and new (on-the-run) series creates a **series basis** that can persist due to:
- **Liquidity premium**: On-the-run indices typically have tighter bid-offer spreads
- **Maturity mismatch**: Different remaining maturities mean different RPV01 values
- **Composition differences**: Credits may enter or exit the index

#### Strategy Card: B2 — Index Roll/Series Switch (Mechanics + Risk)

> **In desk language:** B2 is “stay on-the-run.” You are paying (or receiving) something to switch from an old series to the new, more liquid series.
>
> - Two sources of confusion: (i) **notional scaling** (CS01 changes because maturity changes) and (ii) **series/composition basis** (the contract you’re switching into is not identical).
> - If you want intuition, read Examples 8 and 8B before worrying about hedge ratios.

##### Objective (Conceptual; No Forecasts)

Understand P&L drivers when switching between series, focusing on the three key elements O'Kane identifies: composition changes, maturity reset, and liquidity/on-the-run premium.

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

$$N_{\text{new}} = N_{\text{old}} \cdot \frac{CS01_{\text{old}}}{CS01_{\text{new}}}$$

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

## 52.6 Strategy Family C: Correlation and Tranche Relative Value

> **If tranches are not yet intuitive:** read Chapters 49–51 first. On a first pass through Chapter 52, it is completely reasonable to skip to Section 52.7. The basis/roll/capital-structure strategies stand on their own.

Tranches are fundamentally different from single-name CDS and indices. Their value depends not just on spread levels but on the *distribution* of portfolio losses—and that distribution is shaped by correlation. O'Kane provides a comprehensive risk framework that reveals why tranche positions often behave counter-intuitively.

### 52.6.1 Tranche PV Decomposition and Why Correlation Matters

#### PV Decomposition (Conceptual)

A tranche behaves like a credit derivative on the portfolio loss distribution: PV depends on expected discounted protection leg (driven by losses) and premium leg (driven by outstanding tranche notional).

#### Systemic vs Idiosyncratic Risk

O’Kane distinguishes two types of spread moves for tranche risk:

1. **Systemic (portfolio‑wide) move:** bump *all* issuer curves together (parallel shift).
2. **Idiosyncratic (single‑name) move:** bump one issuer’s curve holding others fixed.

These lead to different deltas/gammas and, therefore, different hedge notions (systemic vs name‑specific hedges).

Rather than relying on any single numeric “risk report snapshot,” focus on the robust qualitative patterns (see Chapter 51 for definitions):

- **Leverage tends to be highest for equity and falls with seniority.** A small notional slice can embed large exposure to portfolio-wide spread moves.
- **Systemic gamma often flips sign across the stack.** Published examples show equity can have negative systemic gamma (unfavorable convexity in parallel spread moves), while more senior tranches can have positive systemic gamma.
- **Correlation sensitivity (Corr01) can flip sign.** In a one-factor setting, increasing correlation reallocates probability mass between “few scattered defaults” and “large clusters,” which can benefit one part of the capital structure and hurt another.

**Plain-English picture:** equity protection (0–3%) is like insuring the *first* slice of portfolio loss. It is sensitive to “lots of small/medium bad outcomes.” Senior protection (e.g., 15–30%) is closer to catastrophe insurance: it only starts paying in severe clustered-default outcomes. When correlation rises, the world becomes more “all-or-nothing,” which can reduce the frequency of small scattered losses while increasing the probability of big clustered loss scenarios. That is the core reason equity and senior tranches can react in opposite directions to correlation changes.

#### Correlation/Dependence Affects Tail

Higher correlation increases the probability of clustered defaults (tail events), which redistributes expected losses across attachment points. In the source’s example risk report, Corr01 is negative for the equity tranche (long protection loses as correlation rises) while a senior tranche can have Corr01 of the opposite sign.

This creates a natural tension: equity and senior tranches have *opposite* correlation exposures. A correlation trader can exploit this by combining tranches to isolate or neutralize correlation risk.

#### Gamma Trading

In the source’s discussion, traders can use a tranche’s systemic vs idiosyncratic gamma profile to express a view on “market‑wide” vs “name‑specific/dispersion” scenarios, while attempting to reduce first‑order spread exposure.

Published examples emphasize a useful sign intuition: **idiosyncratic gamma often has the opposite sign to systemic gamma** for a given tranche, reflecting the difference between “one-name deteriorates” vs “everything widens.” The source also highlights a common trade-off: positions that are “long convexity” (positive gamma) can come with an adverse carry/theta profile, so you must evaluate **carry vs convexity** together, not in isolation.

#### Base Correlation Framework

Base correlation is a widely discussed market convention for quoting “implied correlation” across strikes and for building a surface used in tranche pricing/risk.

Key structural identity: a $[K_1, K_2]$ tranche can be represented as a linear combination (difference) of two equity tranches $[0,K_2]$ and $[0,K_1]$ (see Chapter 50 for the construction and Chapter 51 for the risk implications).

**Interpolation warning:** linear interpolation of base correlation is not guaranteed to be arbitrage‑free for non‑standard strikes. The interpolation scheme is part of the model.

---

### Strategy Card: C1 — Tranche RV / Correlation Exposure with Explicit Hedge Map

> **In desk language:** C1 is “trade the loss distribution.” You are not just trading a spread level; you are trading *where losses land* in the capital structure.
>
> - A PV01 hedge can make the book look stable for small spread moves, but **clustering/tail scenarios** can dominate tranche P&L.
> - When you hedge a tranche with an index, make sure you know whether you are matching **tranche PV01** (tranche quote) or **systemic DV01** (all-name spread bump). Mixing them is a common beginner error.
> - Chapters 50–51 are the deep dive on what these risk measures mean in practice.

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
| Systemic DV01 | USD/bp | PV change under a 1bp parallel bump to all issuer curves (portfolio‑wide spread move) |
| Systemic delta | $ notional | CDS hedge notional implied by the source’s systemic‑hedge definition |
| Corr01 | USD per 1% corr | dependence sensitivity |
| Tail/clustering | scenario USD | nonlinear loss jump risk |
| Recovery sensitivity | USD/1% | recovery impacts losses and settlement |

**Beginner check (common confusion):**

- **Tranche PV01** here means sensitivity to the *tranche’s own quoted spread* (a market quote).
- **Systemic DV01** means sensitivity to a *parallel bump to all underlying issuer curves* (a portfolio-wide credit move).
- **Systemic delta** is often a *derived hedge notional* (e.g., “how much index CDS would hedge the systemic DV01”), so it is a number with **$ notional** units, not USD/bp.

If these objects feel interchangeable, pause and review Chapter 51 — this is where tranche “Greek” language diverges from single-name CDS.

##### Hedge Set and Hedge Ratios (Show Math)

**Systemic spread hedge (conceptual DV01 match):**

If you hedge portfolio‑wide spread risk with an index CDS, size the index notional to offset the tranche’s **systemic DV01** (not the tranche’s contractual spread PV01):

$$N_{\text{index}} \approx -\frac{SystemicDV01_{\text{tranche}}}{CS01_{\text{index per USD  notional}}}$$

**Adjacent tranche PV01 hedge:**

$$N_{\text{hedge tranche}} = -\frac{PV01_{\text{target tranche}}}{PV01_{\text{hedge tranche}}}$$

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
- Corr01: revalue with $\rho \pm 1\\%$; confirm stable sign/magnitude.
- Run discrete default scenarios: 1 default, 5 defaults, 10 defaults; compare hedge performance.

---

### 52.6.2 Tranche Example: Equity vs Senior Correlation Trade

This is a **toy** illustration of how you might combine two tranches to reduce (not eliminate) correlation exposure.

#### Setup (Toy Numbers)

- Reference portfolio: $N=125$ names (equal-weight), total notional USD 1,250mm.
- Equity tranche: 0–3% (long protection).
- Senior tranche: 15–30% (long protection).
- Current implied correlation parameter: $\rho_0$ (exact model details omitted).

Assume your risk system reports (per $USD 10$mm tranche notional):
- Equity Corr01 (long protection): $-USD 80{,}000$ per +1% correlation bump
- Senior Corr01 (long protection): $+USD 150{,}000$ per +1% correlation bump

These signs encode the intuition: higher correlation can make equity safer (hurting long‑protection equity) while making senior tail riskier (helping long‑protection senior).

#### Correlation-Neutral Sizing (Local)

To target Corr01 neutrality at the current point:

$$N_{\text{equity}} \approx N_{\text{senior}}\times \frac{|\text{Corr01}_{\text{senior}}|}{|\text{Corr01}_{\text{equity}}|} = N_{\text{senior}}\times \frac{150}{80} = 1.875\\,N_{\text{senior}}.$$

#### Residual Exposures (Why “Corr01-Neutral” ≠ “Risk-Neutral”)

Even if Corr01 is near zero locally, you still retain:
- **Systemic spread risk** (systemic DV01 / systemic delta): the two tranches generally do not offset the same way for parallel spread moves.
- **Gamma/convexity**: systemic and idiosyncratic gamma profiles differ across the stack; hedge ratios drift in large moves.
- **Jump / clustering risk**: realized defaults and clustered-default scenarios can dominate small-move hedges.
- **Model risk**: Corr01 depends on parameterization and calibration (compound vs base correlation, interpolation rules).

Practical takeaway: treat “Corr01‑neutral” as a *local* risk reduction, then validate the trade under stress scenarios (spread shocks, correlation shocks, clustered defaults, recovery/final price).

---

## 52.6.3 Strategy Selection: When to Use Which Approach

The four strategy families address different risk views and market opportunities:

### Basis Strategies (A1, A2, A3)

**Use when**: You have a view on the relationship between cash and synthetic markets, between single-name and index exposures, or on CDS curve shape.

| Strategy | Best For | Key Residual |
|----------|----------|--------------|
| Bond-CDS basis (A1) | Funding arbitrage, delivery option value, loss-on-default mismatch | Basis widening/tightening; funding shock |
| Single-name vs index (A2) | Isolating idiosyncratic risk with market hedge | Single-name default jump; composition mismatch |
| CDS curve steepener/flattener (A3) | View on term structure of default risk | JTD mismatch from unequal notionals; parallel moves |

**Decision criteria**:
- If you believe funding conditions favor CDS over bonds → basis package with long CDS
- If you want to isolate a single name's idiosyncratic risk → proxy hedge with index
- If you expect the name to outperform/underperform vs market → single vs index trade
- If you believe an inverted HY curve will normalize → curve flattener (but manage JTD)

### Carry and Rolldown Strategies (B1, B2)

**Use when**: You want to understand or capture the time-value components of an index position.

| Strategy | Best For | Key Residual |
|----------|----------|--------------|
| Index carry/rolldown (B1) | Systematic carry harvesting; P&L attribution | Spread moves dominate carry; default events |
| Series roll (B2) | Managing roll costs; on-the-run liquidity | Series basis shock; composition changes |

**Decision criteria**:
- If index curves are upward-sloping and you expect stability → rolldown capture
- If new series has favorable composition → roll early
- If basis trade vs constituents → watch portfolio swap adjustment effects

### Correlation/Tranche Strategies (C1)

**Use when**: You have a view on correlation, dispersion, or tail risk.

| Strategy | Best For | Key Residual |
|----------|----------|--------------|
| Long equity protection | Tail risk hedge; negative carry | Spread tightening; correlation decrease |
| Short senior protection | Carry collection; bullish credit | Correlation increase; default clustering |
| Equity-senior combo | Correlation view without full spread exposure | Systemic delta; gamma profile changes |

**Decision criteria**:
- If you expect increased dispersion (idiosyncratic moves) → long equity protection (positive I.gamma)
- If you expect systemic widening → short equity protection (positive S.gamma)
- If you want correlation exposure without full spread risk → combine equity and senior

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

Empirical studies summarized by O’Kane (sourced to Altman et al.) emphasize three desk‑relevant facts:

1. **Seniority matters:** senior claims tend to recover more on average than subordinated claims.
2. **Recoveries are noisy:** the dispersion of recoveries is wide; “one number” recoveries are dangerous in stress tests.
3. **Absolute priority is not guaranteed:** realized recoveries do not always obey a strict absolute‑priority ordering.

For intuition, the dataset summarized in the book shows materially higher mean recovery for senior secured loans (around the high‑60%s) than for senior unsecured bonds (around the mid‑30%s).

> **Desk Reality: Why Recovery Matters More Than You Think**
>
> Differences in expected recovery across the capital structure can translate into large spread differences in distressed regimes. If you assume the same recovery for senior and subordinated debt, you can mis-size hedges and understate jump and scenario P&L.

### 52.7.2 The Senior-Subordinated Spread Relationship

O'Kane derives the theoretical relationship between senior and subordinated CDS spreads. For a single hazard rate $\lambda$ shared across the capital structure:

$$S_{\text{sub}} = \lambda(1 - R_{\text{sub}}) \quad \text{and} \quad S_{\text{sen}} = \lambda(1 - R_{\text{sen}})$$

Dividing:

$$\boxed{\frac{S_{\text{sub}}}{S_{\text{sen}}} = \frac{1 - R_{\text{sub}}}{1 - R_{\text{sen}}}}$$

O’Kane emphasizes that this ratio is only a rough guide: market technicals and contract specifics can push observed spreads away from the simple relationship. The main use is as a consistency check: spreads across seniority should be broadly aligned with expected recovery differences.

**Worked Example:** If senior unsecured recovery is 40% and subordinated recovery is 20%:

$$\frac{S_{\text{sub}}}{S_{\text{sen}}} = \frac{1 - 0.20}{1 - 0.40} = \frac{0.80}{0.60} = 1.333$$

So subordinated CDS should trade at approximately 1.33× the senior spread. If senior CDS is 100 bp, sub CDS should be ~133 bp. Deviations from this ratio signal relative value opportunities—or reflect market technicals that the simple model doesn't capture.

### 52.7.3 The Merton Model: Equity-Credit Connection

O'Kane presents the Merton model (Section 3.4) as the foundational structural model linking equity and debt values. The model assumes a simplified capital structure:
- Face value $F$ of $T$-maturity zero coupon bonds with total value $D$
- Shares with total value $E$ paying no dividends
- Asset value $A(t) = D(t) + E(t)$

At maturity, the payoffs are:

$$E(T) = \max[A(T) - F, 0] \quad \text{(equity = call option on firm assets)}$$
$$D(T) = F - \max[F - A(T), 0] \quad \text{(debt = cash minus put option)}$$

In the Merton model, equity has the same payoff as a call option on firm assets with strike equal to the face value of debt due at $T$.

Using the Black-Scholes framework with asset volatility $\sigma_A$:

$$E(t) = A(t)\Phi(d_1) - Fe^{-r(T-t)}\Phi(d_2)$$

where $d_1, d_2$ follow the standard Black-Scholes definitions.

O'Kane derives the relationship between equity and asset volatility:

$$\sigma_E = \sigma_A \Phi(d_1) \frac{A(t)}{E(t)}$$

This relationship can be inverted to back out an implied asset volatility from observed equity volatility and leverage (one of the model's practical uses).

#### Limitations (O'Kane Section 3.4.1)

O'Kane lists key limitations that matter for desk intuition:
- The simplified capital structure is unrealistic.
- Default is only allowed at a single time $T$ in the basic setup.
- There is limited transparency regarding the value of a company's assets.
- Short-dated credit spreads can be understated when assets are well above debt (the model pushes spreads toward zero as $T-t \to 0$ in that region).

Despite these limitations, O’Kane connects the intuition to correlation models: in one‑factor copula models, defaults are often driven by a latent “asset” variable in the same broad spirit as structural models.

**Desk note (intuition, not a pricing formula):** the Merton linkage $\sigma_E = \sigma_A\\,\Phi(d_1)\\,A/E$ implies that as equity value $E$ falls (leverage rises), equity volatility can rise even if asset volatility is unchanged. In the same structural framing, higher leverage increases default risk and can push credit spreads wider. In practice, mapping equity vol to CDS levels is noisy and model‑dependent, but equity–credit divergences are a common starting point for capital‑structure RV discussions.

### Strategy Card: D1 — Senior vs Subordinated CDS

> **In desk language:** D1 is “same firm, different place in the capital structure.” You are effectively trading **relative recovery / relative protection value** (with market technicals on top).
>
> - The simple spread-ratio formula is a *consistency check*, not a guarantee. Real trades live and die on documentation, deliverables, liquidity, and the recovery distribution.
> - If you want background: Chapter 39 (documentation and restructuring) and Chapter 40 (auction/final price) are the operational prerequisites.

##### Objective (Conceptual; No Forecasts)

Exploit deviations of the senior-sub spread ratio from its theoretical value, or express a view on relative recovery rates across the capital structure.

##### Instruments and Position Construction

- **Long sub protection, short senior protection** (betting sub widens relative to senior → the ratio increases toward theoretical fair value)
- **Or the reverse** (betting ratio compresses)
- Size for CS01-neutrality or for a specific spread-ratio view.

##### Exposure Decomposition

| Exposure (units) | Senior leg | Sub leg | Net |
|------------------|-----------|---------|-----|
| $CS01$ (USD/bp) | $\lt 0$ (short prot) | $\gt 0$ (long prot) | can match or intentional residual |
| JTD (USD) | negative | positive | recovery-dependent residual |
| Rec01 (USD/1%) | senior recovery sens | sub recovery sens | **key residual**: different recovery drives P&L |
| Default event | pays $(1 - R_{\text{sen}})$ | receives $(1 - R_{\text{sub}})$ | net gain if $R_{\text{sub}} \lt R_{\text{sen}}$ (typical) |

##### Hedge Set and Hedge Ratios

**CS01-neutral sizing:**

$$N_{\text{sen}} = N_{\text{sub}} \times \frac{RPV01_{\text{sub}}}{RPV01_{\text{sen}}}$$

**Ratio-neutral sizing (maintaining spread ratio):**

$$\frac{N_{\text{sub}} \cdot S_{\text{sub}}}{N_{\text{sen}} \cdot S_{\text{sen}}} = \frac{1 - R_{\text{sub}}}{1 - R_{\text{sen}}}$$

##### Scenario Test Suite

- Parallel spread move (both senior and sub widen together).
- Ratio compression: sub tightens relative to senior.
- Ratio expansion: sub widens relative to senior.
- Default event: both trigger; compute net JTD using different recovery rates.
- Recovery surprise: actual recovery differs from assumed values.

##### Failure Modes

- **Recovery uncertainty dominates:** The trade's JTD depends critically on the difference $R_{\text{sen}} - R_{\text{sub}}$. If recovery turns out to be 20% for both (APR violation), the trade loses its structural advantage.
- **Liquidity asymmetry:** Sub CDS may be significantly less liquid than senior; execution costs can exceed the theoretical edge.
- **Market technicals:** supply/demand and positioning can push the ratio far from the simple recovery-implied relationship for extended periods.

##### Implementation Checklist

- Verify both CDS reference the same entity with aligned maturities.
- Use appropriate recovery assumptions for each seniority level.
- Check that the spread ratio is historically wide or tight relative to the theoretical level before trading.
- Run default scenario with recovery sweep ($R$ from 10% to 60%) on both legs.

---

### Strategy Card: D2 — Equity-CDS Relative Value (Merton-Inspired)

> **In desk language:** D2 is a cross-asset relative value trade: “is equity pricing a different credit story than CDS?” You are trying to isolate that disagreement.
>
> - The hard part is that the hedge is imperfect: equity brings delta/vega and discrete corporate actions; CDS brings default/settlement mechanics.
> - If you are new to this, read Section 52.7.3 for the structural intuition and then go straight to Example 19 (LBO toy) to see why sizing is nontrivial.

##### Objective (Conceptual; No Forecasts)

Exploit divergences between equity-implied credit risk and CDS-priced credit risk, using the Merton model's insight that both are functions of the same underlying asset value and volatility.

##### Instruments and Position Construction

- **CDS leg:** single-name CDS (long or short protection).
- **Equity leg:** equity options or equity shares used to hedge the equity-implied credit component.
- The trade exploits the link: when equity volatility implies a default probability different from what CDS spreads imply, one of them may be mispriced.

**Desk note (LBO/leverage events):** an LBO is a discrete capital‑structure shock. It can change leverage, covenant package, and expected recoveries, so both senior and subordinated CDS can reprice sharply. Some desks express an LBO view as an outright **credit‑widening** position (buying protection), while others express a **relative** senior vs sub view. The direction and sizing depend on the specific deal structure and which obligations are effectively being referenced by each CDS contract.

Senior vs sub recovery and settlement mechanics in LBO situations are documentation-dependent (guarantees, obligation characteristics, and the post‑deal capital structure). Before sizing a senior-vs-sub expression, confirm the relevant contract terms, deliverables/obligations, and the recovery/settlement assumptions used in your risk system.

##### Exposure Decomposition

| Exposure (units) | CDS leg | Equity leg | Net |
|------------------|---------|------------|-----|
| Credit spread sensitivity | CS01 | equity-implied CS01 | residual |
| Equity delta | ~0 | dominant | equity exposure |
| Equity vega | indirect (via Merton link) | dominant | vol exposure |
| JTD | CDS default payoff | equity goes to ~0 | significant residual |

##### Failure Modes

- **Model risk:** The Merton model is highly simplified. O'Kane lists limitations including unrealistic capital structure, single default time, and opaque asset values. Any trade based on Merton-implied fair value inherits these model risks.
- **Equity-credit decorrelation:** In practice, equity and CDS can diverge for reasons the model doesn't capture (e.g., equity buybacks, dividend changes, sector rotation).
- **Execution complexity:** Running a CDS-equity hedge requires managing two different asset classes with different settlement, margin, and liquidity characteristics.

---

## 52.8 Loan CDS vs Standard CDS

O'Kane discusses Loan CDS (LCDS) in Section 5.7 as a variant that references loans rather than bonds. A key intuition is recovery: secured loans have historically recovered more than senior unsecured bonds, so the same default intensity can imply a tighter spread for loan CDS.

From the recovery dataset summarized in the book (Altman et al.), mean recoveries are roughly 68.5% for senior secured loans and 34.89% for senior unsecured bonds. In a toy "equal hazard rate" world where spread scales like $(1-R)$:

$$\frac{S_{\text{CDS}}}{S_{\text{LCDS}}} = \frac{1 - R_{\text{bond}}}{1 - R_{\text{loan}}} = \frac{1 - 0.35}{1 - 0.685} = \frac{0.65}{0.315} = 2.06$$

**Interpretation:** higher expected recovery on loans implies lower loss-given-default and therefore a lower fair spread, all else equal. **Caveat:** LCDS contracts can embed additional features (deliverable definitions, and in some forms a cancellation/refinancing feature), so you should not treat the ratio as a universal rule.

O'Kane also highlights that LCDS trigger definitions can differ across products/regions and that cancellable LCDS introduces a cancellation feature. In O’Kane’s treatment, valuing cancellable LCDS introduces both a default survival curve $Q_D$ and a cancellation curve $Q_C$.

LCDS conventions and liquidity are contract- and time-dependent. Before trading or marking LCDS, confirm the relevant trigger definitions, any cancellation/refinancing feature, the quoting regime (spread vs coupon+upfront), and execution liquidity for the specific name/index you care about.

---

## 52.9 Crisis Behavior of Credit Strategies

The strategies described in this chapter are designed for normal market conditions. In systemic stress, the assumptions underlying hedging relationships break down. O'Kane's emphasis on "perfect storm" risk—multiple adverse moves coinciding—is especially relevant here.

### 52.9.1 Basis Strategies in Crisis

In O’Kane’s framing, basis trades are vulnerable in stress because multiple basis drivers can move in the same adverse direction:

1. **Funding asymmetry dominates:** the cash bond leg must be financed, while CDS is economically unfunded. When repo/funding terms tighten, the cash leg can become very expensive to carry.

2. **CDS becomes the “fast” short:** it is often operationally easier to go short credit via CDS than via cash bonds, so protection‑buying flows can push CDS wider faster than cash spreads.

3. **Technical flows can reverse:** dealer hedging of structured‑credit flows (e.g., synthetic CDO issuance and unwind) can mechanically tighten or widen CDS spreads relative to cash.

Stress‑test takeaway: a basis package that looks low‑risk under “small moves” can suffer large, discontinuous losses when funding and liquidity regimes shift.

### 52.9.2 Correlation and Tranche Trades in Crisis

Published tranche risk examples highlight that equity tranches can be highly leveraged and can carry negative systemic gamma. In stress regimes:

- **Correlation spikes:** Defaults cluster, losses concentrate in equity and mezzanine tranches.
- **Equity tranche losses accelerate:** The negative systemic gamma means large spread moves *amplify* equity tranche losses beyond the linear (delta) approximation.
- **Senior tranches initially benefit, then face tail risk:** Senior tranches have positive systemic gamma (they gain on large moves initially), but if defaults actually cluster and breach attachment points, losses can be sudden and severe.

Correlation hedges are inherently model‑dependent and tend to be least reliable precisely in the states where tail risk dominates. Treat correlation and clustering as scenario‑driven risks, not just a single Corr01 number.

### 52.9.3 Early Warning Indicators

Useful monitoring signals (exact thresholds are desk policy):

- CDS–bond basis and its rate of change.
- Index “quoted vs intrinsic” basis (and any portfolio‑swap adjustment changes).
- Implied correlation / tranche marks (and Corr01 P&L vs carry/theta).
- Funding terms for the cash leg (repo availability/terms) and margin requirements.
- Bid/offer and market depth (liquidity regime).

Thresholds and escalation playbooks are set by each desk (risk appetite, governance, and market regime). Use the signal list above as generic categories, then plug in your desk’s specific thresholds and escalation actions.

---

## 52.10 Execution and Position Management

Execution and position management are where “model hedges” meet market microstructure. The details are desk‑specific, but the general risk‑first discipline is consistent with the book’s stress‑testing emphasis.

### 52.10.1 Position Sizing Considerations

For any strategy in this chapter, position size should reflect:

1. **Risk budget allocation:** Express maximum loss tolerance in JTD and CS01 terms, not just notional.
2. **Carry-to-risk trade-off:** Carry only matters if you can survive the stress scenario. Size so that plausible stress losses and funding/liquidity shocks fit within the desk’s risk limits.
3. **Liquidity-adjusted sizing:** Expected edge should dominate round‑trip costs and expected slippage, especially for instruments that gap in stress.

### 52.10.2 Monitoring Triggers

Possible trigger categories (thresholds are desk‑specific):

- **Hedge drift:** CS01 / bucketed CS01 no longer near target.
- **Funding regime change:** cash-leg financing terms change materially versus the strategy’s carry.
- **Recovery / settlement assumptions:** recovery/final‑price inputs move materially; re-run JTD/VOD.
- **Correlation/tail regime:** tranche marks move in a way inconsistent with the hedge map; re-run clustered default scenarios.

Exact numeric trigger thresholds and escalation actions are set by desk playbooks; treat them as an input to this framework rather than a universal rule.

### 52.10.3 Unwind Playbook

When conditions warrant reducing or exiting a position:

1. **Unwind the most liquid leg first** — typically CDS over bonds, index over single-name.
2. **Don't unwind CS01-hedged positions one leg at a time** — this creates naked exposure. Unwind in matched pairs where possible.
3. **In illiquid markets, consider partial unwinds** — reducing by 50% captures some value while maintaining optionality.
4. **Document unwind rationale** — was it risk limit breach, P&L stop, or fundamental view change? This matters for post-trade analysis.

---

## 52.11 Mathematical Foundations

This section is here so you can sanity-check what a risk system is doing under the hood. If you do not like calculus, skip to Section 52.12 — the worked examples are the real learning tool. For a full CDS risk-measure derivation (CS01/RPV01, VOD/JTD, recovery risk, theta), Chapter 43 is the canonical reference in this book.

### 52.11.1 CS01 for a Standard CDS from the MTM Identity

**Source-backed starting point:**

$$V(t,T) = (S(t,T) - S_0) \cdot RPV01(t,T) \quad \text{(per unit notional)}.$$

**Differentiate w.r.t. market spread $S$:**

$$\frac{\partial V}{\partial S} = RPV01(t,T).$$

**Therefore,**

$$\boxed{CS01 = \frac{\partial PV}{\partial S} \times 10^{-4} = N \cdot RPV01(t,T) \cdot 10^{-4}}$$

**Unit check:**

$RPV01$ has units "years" (PV of 1 bp/yr premium). Multiply by $10^{-4}$ (per bp) and notional $N$ (USD) $\Rightarrow$ USD/bp.

---

### 52.11.2 CS01-Based Hedge Ratio Derivation

Let target position have $CS01_T$ and hedge instrument have $CS01_H$, both sign-inclusive (USD/bp).

We want:

$$CS01_{\text{net}} = CS01_T + N_H \cdot CS01_H^{(USD 1)} = 0.$$

Solve:

$$\boxed{N_H = -\frac{CS01_T}{CS01_H^{(USD 1)}}}$$

If CS01s are already in USD/bp for stated notionals:

$$N_H = -\frac{CS01_T}{CS01_H} \times N_H^{\text{current}}.$$

---

### 52.11.3 JTD (VOD) Mapping for CDS

**Source-backed VOD expression:**

$$VOD = (1 - R - \text{Accrued}) - (S - S_0) \cdot RPV01.$$

**Interpretation:**

If position is at par ($S = S_0$), then

$$VOD \approx 1 - R - \text{Accrued}.$$

**Dollar JTD:**

$$\boxed{JTD = N \cdot VOD}$$

**Unit check:**

$1 - R$ is unitless fraction of notional; accrued is fraction; multiply by $N$ gives USD.

---

### 52.11.4 P&L Explain Template (First-Order + Event Terms)

For a portfolio, approximate P&L over horizon as:

$$\boxed{\Delta PV \approx \sum_i (CS01_i \cdot \Delta S_i) + (Rec01 \cdot \Delta R) + JTD_{\text{events}} + BasisTerm + Residual}$$

- **CS01 term:** linear spread moves (bucketed by maturity or instrument).
- **Rec01 term:** recovery assumption change.
- **JTD events:** discrete defaults and settlement mechanics.
- **BasisTerm:** quoted–intrinsic or CDS–cash basis change.
- **Residual:** convexity, model error, liquidity/execution slippage.

---

### 52.11.5 Carry/Rolldown Decomposition

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

**Implementation note:** an exact closed‑form “carry” formula depends on the quoting regime (par spread vs fixed coupon + upfront), day count, clean vs full price, and your desk’s carry definition. To make this section fully operational, use the index rulebook conventions and your desk’s P&L attribution definition.

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

**Bond DV01 + CS01 vs CDS CS01 + JTD; choose CDS notional to hedge JTD; show residual under basis move.**

#### Setup

- Bond face: $N_B = USD 1{,}000{,}000$
- Bond full price: $P = 101$ (per 100) $\Rightarrow$ market value $= 1.01 \times 1{,}000{,}000 = USD 1{,}010{,}000$
- Recovery assumption: $R = 40\\% \Rightarrow$ recovery value $USD 400{,}000$
- Bond default loss (JTD for long bond):

$$JTD_{\text{bond}} = -(P - R) \times N_B = -(1.01 - 0.40) \times 1{,}000{,}000 = -USD 610{,}000.$$

(Loss-on-default mismatch is a key basis driver in the sources.)

#### CDS Leg: Choose Notional to Hedge JTD

Assume we buy protection at par: $S = S_0 = 150$ bp $\Rightarrow$ pre-default MTM $\approx 0$.

Accrued premium at default (assume halfway through quarter): $\Delta = 0.125$

$$\text{Accrued} = N_{\text{CDS}} \times (0.015) \times 0.125 = 0.001875 \\, N_{\text{CDS}}.$$

JTD for long protection at par (using VOD logic):

$$JTD_{\text{CDS}} \approx N_{\text{CDS}} \times (1 - R) - \text{Accrued} = 0.60 N_{\text{CDS}} - 0.001875 N_{\text{CDS}} = 0.598125 \\, N_{\text{CDS}}.$$

Solve $0.598125 N_{\text{CDS}} = USD 610{,}000$:

$$N_{\text{CDS}} = \frac{610{,}000}{0.598125} = USD 1{,}019{,}853.71 \approx USD 1.02\text{mm}.$$

#### Spread Sensitivities

Assume CDS $RPV01 = 4.5$. Then:

$$CS01_{\text{CDS}} = N_{\text{CDS}} \cdot RPV01 \cdot 10^{-4} = 1{,}019{,}853.71 \times 4.5 \times 10^{-4} = USD 458.93/\text{bp}.$$

Assume bond credit CS01 (toy): $CS01_{\text{bond}} = -USD 420/\text{bp}$.

#### Basis-Move Scenario

CDS spread widens $+20$ bp; bond spread widens $+5$ bp (basis widens).

**P&L (linear):**

- Bond: $\Delta PV_B = -420 \times 5 = -USD 2{,}100$.
- CDS: $\Delta PV_{\text{CDS}} = +458.93 \times 20 = +USD 9{,}178.68$.
- **Net:** $+7{,}078.68$.

#### Interpretation

JTD was hedged by construction, but basis risk remains: differing spread moves generate residual P&L.

---

### Example 1B: Basis Package Under Funding Stress

This example extends Example 1 to show how funding shocks can dominate a "hedged" basis position.

#### Continuing Setup

Same bond-CDS package from Example 1:
- Bond face: $1mm at clean price 101
- CDS notional: ~$1.02mm (JTD-matched)
- CS01 net: +$38.93/bp (unhedged basis exposure)

#### Funding Shock Scenario

Suppose the bond is financed via repo at Libor + 25bp initially. During a credit crisis:
- Repo rate on this name spikes to Libor + 150bp (125bp increase)
- Repo term: 3 months (0.25 years)

**Additional funding cost over 3 months:**

$$\text{Funding cost} = 1{,}010{,}000 \times 0.0125 \times 0.25 = USD 3{,}156.25$$

#### Combined Scenario: Basis Widens + Funding Shock

From Example 1: CDS spread widens +20bp, bond spread widens +5bp (basis widens 15bp).

**P&L components:**
- CS01 P&L: $+7{,}078.68$ (from Example 1)
- Funding shock: $-3{,}156.25$
- **Net P&L**: $+3{,}922.43$

The funding shock consumed 45% of the basis profit.

#### Extreme Case: Basis Flat but Funding Spikes

If CDS and bond spreads move together (basis unchanged, no spread P&L):
- Spread P&L: ~$0$
- Funding shock: $-3{,}156.25$
- **Net P&L**: $-3{,}156.25$ loss

**Lesson:** The bond leg's financing is a first-class risk. Even if your CS01/JTD look hedged, a change in repo/funding can dominate short-horizon P&L; CDS does not require financing the bond principal.

---

### Example 2: DV01-Neutral but Not Credit-Neutral

**Bond DV01 hedged with rates instrument, then credit spread widens; compute residual P&L.**

#### Setup

- Corporate bond notional: $N_C = USD 10\text{mm}$
- Rates DV01 (sign-inclusive for +1bp rates move): $DV01_{r,C} = -USD 7{,}500/\text{bp}$
- Treasury hedge instrument DV01 per USD 1mm (long): $-850$ USD/bp.
- If we short Treasuries, DV01 becomes $+850$ USD/bp per USD 1mm.

#### DV01 Hedge Ratio

Need $-7{,}500 + 850 \times N_T\text{ (USD 1mm units)} = 0$

$$N_T = \frac{7{,}500}{850} = 8.8235 \text{ mm}.$$

So: **short $8.8235mm Treasuries.**

#### Scenario

- Rates +10 bp (parallel)
- Credit spread on corporate widens +50 bp
- Treasury credit spread unchanged (assume risk-free)

#### P&L

**Rates move:**
- Corporate: $-7{,}500 \times 10 = -USD 75{,}000$
- Short Treasury: $+(850 \times 8.8235) \times 10 = +7{,}500 \times 10 = +USD 75{,}000$
- **Net rates P&L $\approx 0$.**

**Credit move:**
- Assume corporate credit CS01 (toy): $CS01_{\text{corp}} = -6{,}500$ USD/bp (for USD 10mm).
- $\Delta PV_{\text{credit}} = -6{,}500 \times 50 = -USD 325{,}000$

#### Conclusion

**DV01-neutral does not mean risk-neutral: credit spread risk dominates.**

---

### Example 3: Single-Name Hedged with Index by CS01 Matching

**Scenario: broad move vs idiosyncratic move; compute residual.**

#### Setup

- Single-name CDS: long protection, $N_s = USD 10\text{mm}$, $RPV01_s = 4.5$

$$CS01_s = 10{,}000{,}000 \times 4.5 \times 10^{-4} = USD 4{,}500/\text{bp}.$$

- Index CDS: use $RPV01_I = 4.2$ (toy). CS01 per USD 10mm long protection:

$$CS01_{I, USD 10mm} = 10{,}000{,}000 \times 4.2 \times 10^{-4} = USD 4{,}200/\text{bp}.$$

Hedge by short protection on index.

#### Hedge Notional

Solve $CS01_s + CS01_I^{\text{(short)}} = 0$:

$$4{,}500 - 4{,}200 \times \frac{N_I}{10\text{mm}} = 0 \Rightarrow N_I = 10\text{mm} \times \frac{4{,}500}{4{,}200} = USD 10.7143\text{mm}.$$

#### Scenario A: Broad +10 bp Move (Both)

- Single-name: $+4{,}500 \times 10 = +USD 45{,}000$
- Index short: $-4{,}200 \times 1.07143 \times 10 = -USD 45{,}000$
- **Net $\approx 0$.**

#### Scenario B: Idiosyncratic Single-Name +50 bp, Index +10 bp

- Single-name: $+4{,}500 \times 50 = +USD 225{,}000$
- Index short: $-45{,}000$
- **Net: $+USD 180{,}000$** (residual idiosyncratic spread risk).

#### Event Note (JTD)

If the single name defaults, single-name JTD is large; index hedge only absorbs the small loss on that one name inside the index (diversification). **CS01 matching does not hedge JTD.**

---

### Example 4: Index Hedged with Constituents (Bottom-Up) Using RPV01/CS01 Weights

**Important:** The sources show intrinsic index valuation as a sum/average of constituent CDS values and discuss portfolio swap adjustment. A full "bottom-up hedge" method depends on how you allocate index basis and curve adjustments; details are desk-convention dependent.

**Important:** there is no single canonical hedge-weight formula without specifying the portfolio swap adjustment rule (and the risk system’s definition of “intrinsic” vs “quoted” objects). The example below is a clearly labeled approximation and is best treated as a diagnostic (CS01 matching) rather than a production hedge recipe.

#### Approximation Used

- Small index with $M = 3$ names.
- Index notional $N_I = USD 30\text{mm} \Rightarrow USD 10\text{mm}$ per name.
- Assume all maturities aligned at 5Y.

#### Constituent RPV01s (Toy)

- Name A: 4.5
- Name B: 4.7
- Name C: 4.6

#### Index CS01 (Approx)

Index RPV01 $\approx$ average $= 4.6$

$$CS01_I = 30{,}000{,}000 \times 4.6 \times 10^{-4} = USD 13{,}800/\text{bp}.$$

#### Constituent Hedge (Short Protection on Names to Hedge Long Protection Index)

Short each name $10mm:

- A: $-10{,}000{,}000 \times 4.5 \times 10^{-4} = -USD 4{,}500/\text{bp}$
- B: $-USD 4{,}700/\text{bp}$
- C: $-USD 4{,}600/\text{bp}$
- **Total:** $-USD 13{,}800/\text{bp}$, which offsets index $+USD 13{,}800/\text{bp}$.

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

- $M = 3$, index notional $N_I = USD 10\text{mm}$, coupon $C = 100$ bp.
- Constituent 5Y spreads (bp): $S_1 = 120$, $S_2 = 90$, $S_3 = 110$.
- Constituent RPV01s: $4.5, 4.7, 4.6$.
- Quoted index spread: $S_I = 110$ bp.
- Index RPV01: $RPV01_I = 4.6$.

#### Compute Intrinsic Spread (RPV01-Weighted Average)

$$S_{\text{intr}} = \frac{\sum_m RPV01_m S_m}{\sum_m RPV01_m} = \frac{4.5(120) + 4.7(90) + 4.6(110)}{4.5 + 4.7 + 4.6} = 106.4493 \text{ bp}.$$

#### Index Basis (Spread)

$$\text{Basis} = S_I - S_{\text{intr}} = 110 - 106.4493 = 3.5507 \text{ bp}.$$

#### PV View (Upfront Difference)

**Market upfront value at coupon $C$:**

$$U_{\text{mkt}} = N_I \cdot RPV01_I \cdot (S_I - C) \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.6 \cdot (10) \cdot 10^{-4} = USD 46{,}000.$$

**Intrinsic upfront using constituents** (equal-weight notional $N_I/M = 3.3333\text{mm}$):

- Name 1: $(120 - 100) = 20$ bp
  - $PV_1 = 3.3333\text{mm} \cdot 4.5 \cdot 20 \cdot 10^{-4} = USD 30{,}000$
- Name 2: $(90 - 100) = -10$ bp
  - $PV_2 = 3.3333\text{mm} \cdot 4.7 \cdot (-10) \cdot 10^{-4} = -USD 15{,}666.67$
- Name 3: $(110 - 100) = 10$ bp
  - $PV_3 = 3.3333\text{mm} \cdot 4.6 \cdot 10 \cdot 10^{-4} = USD 15{,}333.33$
- **Sum** $V_{\text{intr}} = USD 29{,}666.67$

#### Basis PV

$$U_{\text{mkt}} - V_{\text{intr}} = 46{,}000 - 29{,}666.67 = USD 16{,}333.33,$$

which also equals:

$$N_I \cdot RPV01_I \cdot \text{Basis} \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.6 \cdot 3.5507 \cdot 10^{-4} = USD 16{,}333.33.$$

#### Basis P&L Scenario

Suppose basis decreases by 2 bp due to index tightening (constituents unchanged):

- Index $\Delta S_I = -2$ bp.
- Index CS01: $10{,}000{,}000 \times 4.6 \times 10^{-4} = USD 4{,}600/\text{bp}$.
- A position long index protection has P&L $\Delta PV = 4{,}600 \times (-2) = -USD 9{,}200$.
- The constituent legs (unchanged) contribute $\approx 0$.

**Interpretation:** basis moves can dominate short-horizon hedges.

---

### Example 6: Index Carry

**Compute one quarter's premium/coupon cashflow and upfront amortization effect (if applicable) and show carry under unchanged spreads.**

#### Setup (Long Protection on Index)

- Notional $N = USD 10\text{mm}$
- Coupon $C = 100$ bp
- Market spread at inception $S = 120$ bp
- $RPV01(t_0) = 4.6$, $RPV01(t_1) = 4.4$ after one quarter
- Quarter fraction $\Delta = 0.25$

#### Upfront at Inception (Paid by Protection Buyer Because $S \gt C$)

$$U_0 = N \cdot RPV01(t_0) \cdot (S - C) \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.6 \cdot 20 \cdot 10^{-4} = USD 92{,}000.$$

#### Coupon Paid over the Quarter (Carry Cashflow)

$$\text{Coupon CF} = -N \cdot C \cdot 10^{-4} \cdot \Delta = -10{,}000{,}000 \cdot 100 \cdot 10^{-4} \cdot 0.25 = -USD 25{,}000.$$

#### Unchanged Spreads: MTM "Amortization" Effect

At $t_1$, with spreads unchanged at $S = 120$ bp, the contract value at coupon $C$ is:

$$V(t_1) = N \cdot RPV01(t_1) \cdot (S - C) \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.4 \cdot 20 \cdot 10^{-4} = USD 88{,}000.$$

Since we paid $U_0 = 92{,}000$ upfront at inception, the clean value relative to inception is:

$$\Delta V_{\text{clean}} = 88{,}000 - 92{,}000 = -USD 4{,}000.$$

#### Carry under Unchanged Spreads (Quarter)

$$\text{Carry P\\&L} \approx \text{Coupon CF} + \Delta V_{\text{clean}} = -25{,}000 - 4{,}000 = -USD 29{,}000.$$

**Interpretation:** even with unchanged spreads, the upfront-related value decays as maturity shortens (RPV01 falls).

---

### Example 7: Index Rolldown

**Hold curve fixed, move forward one month, revalue (toy) and compute rolldown P&L.**

#### Setup (Continue Long Protection Example Style)

- Notional $N = USD 10\text{mm}$, coupon $C = 100$ bp
- Upfront paid at inception (from Example 6): $U_0 = 92{,}000$
- Assume spread curve is upward sloping:
  - 5Y par spread at $t_0$: 120 bp
  - After 1 month, remaining maturity is about 4.92Y with par spread 118 bp (curve held fixed, you "roll down")
- $RPV01(t_1) = 4.57$ (toy, slightly lower due to shorter maturity)
- Monthly coupon accrual (approx): $\Delta = 1/12$

#### MTM at $t_1$ Holding Curve Fixed

$$V(t_1) = N \cdot 4.57 \cdot (118 - 100) \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.57 \cdot 18 \cdot 10^{-4} = USD 82{,}260.$$

#### Clean Value Relative to Inception

$$\Delta V_{\text{clean}} = 82{,}260 - 92{,}000 = -USD 9{,}740.$$

#### Coupon Accrual over 1 Month

$$\text{Coupon accrual} = -N \cdot 100 \cdot 10^{-4} \cdot \frac{1}{12} = -10{,}000{,}000 \cdot 0.01 \cdot 0.083333 = -USD 8{,}333.33.$$

#### Rolldown + Carry over Month (No Curve Change Beyond Aging)

$$\text{P\\&L} \approx -8{,}333.33 - 9{,}740 = -USD 18{,}073.33.$$

---

### Example 8: Roll Trade Mechanics

**Old series vs new series upfront difference; compute cash exchanged and execution-cost effect (toy).**

O'Kane identifies two primary sources of P&L impact during the roll: (1) composition changes affecting perceived credit quality, and (2) maturity extension on an upward-sloping curve. This example illustrates both effects.

#### Setup

- Notional $N = USD 10\text{mm}$, coupon $C = 100$ bp.
- Old series: $S_{\text{old}} = 115$ bp, $RPV01_{\text{old}} = 4.1$ (remaining maturity ~4.5Y)
- New series: $S_{\text{new}} = 120$ bp, $RPV01_{\text{new}} = 4.6$ (full 5Y maturity)
- Position: long protection.

#### Understanding the Spread Difference

The new series trades 5 bp wider than the old series. Per O'Kane's framework, we can decompose this:

**Maturity extension effect**: The new index has 6 months longer maturity. With an upward-sloping credit curve (typical), this adds spread. Suppose the curve slope is ~10 bp per year at the 5Y point:
$$\Delta S_{\text{maturity}} \approx 10 \times 0.5 = +5 \text{ bp}$$

**Composition effect**: In this example, composition changes are neutral—the downgraded names removed are offset by wider-spread liquidity replacements:
$$\Delta S_{\text{composition}} \approx 0 \text{ bp}$$

**Total**: $\Delta S = 5 + 0 = 5$ bp (matches observed difference).

#### MTM Values (Pre-Upfront)

$$V_{\text{old}} = N \cdot 4.1 \cdot (115 - 100) \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.1 \cdot 15 \cdot 10^{-4} = USD 61{,}500.$$

$$V_{\text{new}} = N \cdot 4.6 \cdot (120 - 100) \cdot 10^{-4} = USD 92{,}000.$$

#### Cash to Switch (Close Old, Open New)

- Close old (sell your long protection): receive $61,500.
- Open new: pay $92,000.
- **Net cash outflow:**

$$92{,}000 - 61{,}500 = USD 30{,}500.$$

#### CS01-Adjusted Notional for Spread-Neutral Switch

If the goal is to maintain the same CS01 exposure after rolling:

$$CS01_{\text{old}} = 10{,}000{,}000 \times 4.1 \times 10^{-4} = USD 4{,}100/\text{bp}$$
$$CS01_{\text{new per USD 10mm}} = 10{,}000{,}000 \times 4.6 \times 10^{-4} = USD 4{,}600/\text{bp}$$

To maintain 4,100 USD/bp CS01 on the new series:

`N_new = 10,000,000 * (4,100 / 4,600) = USD 8,913,043`

This means **reducing notional by ~11%** when rolling to maintain spread-neutral exposure.

#### Execution Cost (Toy)

Suppose bid/offer equivalent to 0.5 bp spread on each leg.

- Approx PV cost per leg $\approx CS01 \times 0.5$.
- Use $CS01_{\text{new}} \approx 4{,}600$ USD/bp
- Cost per leg $\approx 4{,}600 \times 0.5 = USD 2{,}300$
- Two legs $\Rightarrow$ total $\approx USD 4{,}600$

#### Total "All-In" Cash Impact

$$30{,}500 + 4{,}600 = USD 35{,}100.$$

#### Interpretation

The roll cost ($30,500) is substantial relative to quarterly carry. A trader must weigh the liquidity benefits of being on-the-run against the roll cost. The CS01-adjusted notional calculation shows that maintaining risk-neutral exposure requires reducing size—rolling "flat notional" actually *increases* spread exposure by 12%.

---

### Example 8B: Roll with Favorable Composition

**What happens when composition improvement dominates maturity extension?**

#### Setup

Same as Example 8, but now suppose 3 distressed names (average spread 400 bp) are removed and replaced with 3 IG names (average spread 50 bp) in a 125-name index.

#### Composition Effect

Spread reduction from composition change:
$$\Delta S_{\text{composition}} = \frac{3}{125} \times (50 - 400) = -8.4 \text{ bp}$$

#### Net Spread Change

$$\Delta S = \Delta S_{\text{maturity}} + \Delta S_{\text{composition}} = 5 + (-8.4) = -3.4 \text{ bp}$$

The new series trades **tighter** than the old series despite the longer maturity.

#### Roll P&L Impact

- Old series: $S_{\text{old}} = 115$ bp, $V_{\text{old}} = USD 61{,}500$
- New series: $S_{\text{new}} = 111.6$ bp

$$V_{\text{new}} = 10{,}000{,}000 \cdot 4.6 \cdot (111.6 - 100) \cdot 10^{-4} = USD 53{,}360$$

**Net cash inflow** (favorable roll):
$$61{,}500 - 53{,}360 = USD 8{,}140 \text{ received}$$

#### Key Lesson

O’Kane emphasizes that roll composition cuts both ways: the new series may drop downgraded/less‑liquid names, but liquidity requirements can also lead to replacement by wider names. In practice, roll P&L decomposes into (i) maturity/curve effects and (ii) composition effects, and you need both to explain the move.

---

### Example 9: Default in Index

**Given final price FP, compute default settlement impact on index cashflows.**

#### Source-Backed Mechanics

Default payoff based on face value minus final price; settlement can be physical or cash with auction determining final price.

For index: a default causes a payout on that name and reduces future premium by a proportional notional reduction (illustrated as $1/M$ type reduction in the sources).

#### Toy Setup

- Total index notional: $N = USD 10\text{mm}$
- $M = 5 \Rightarrow$ notional per name $N_m = 2\text{mm}$
- Coupon $C = 100$ bp
- Final price $FP = 35\\% \Rightarrow R = 0.35$
- Accrual since last coupon: 50 days on Actual/360:

$$\Delta = \frac{50}{360} = 0.1388889.$$

#### Default Settlement (Long Protection)

**Protection payment:**

$$(1 - R) N_m = (1 - 0.35) \times 2{,}000{,}000 = 0.65 \times 2{,}000{,}000 = USD 1{,}300{,}000.$$

**Accrued premium owed by protection buyer on that name:**

$$N_m \cdot C \cdot 10^{-4} \cdot \Delta = 2{,}000{,}000 \cdot 100 \cdot 10^{-4} \cdot 0.1388889 = 2{,}000{,}000 \cdot 0.01 \cdot 0.1388889 = USD 2{,}777.78.$$

**Net immediate cash:**

$$1{,}300{,}000 - 2{,}777.78 = USD 1{,}297{,}222.22.$$

#### Future Coupon Impact

**Before default:** quarterly coupon on $10mm:

$$10{,}000{,}000 \cdot 0.01 \cdot 0.25 = USD 25{,}000.$$

**After default,** notional reduced by $1/M = 20\\% \Rightarrow$ remaining $= 8\text{mm}$:

$$8{,}000{,}000 \cdot 0.01 \cdot 0.25 = USD 20{,}000.$$

Coupon reduces by $5,000 per quarter from then on (toy).

---

### Example 10: Full P&L Explain for an Index Position over a Horizon

**Carry + rolldown + spread move + default event.**

#### Setup

- Long protection on index
- Notional $N = USD 10\text{mm}$, coupon $C = 100$ bp
- $RPV01 = 4.6 \Rightarrow CS01 = 10{,}000{,}000 \times 4.6 \times 10^{-4} = USD 4{,}600/\text{bp}$
- One-month horizon; use Example 7's carry+rolldown under fixed curve: $-USD 18{,}073.33$

#### Observed Changes (Toy)

**Index spread widens $+10$ bp** $\Rightarrow$ spread P&L:

$$\Delta PV_{\text{spread}} = 4{,}600 \times 10 = +USD 46{,}000.$$

**One constituent defaults** in a large index $M = 125$ (so per-name notional small):

- $N_m = 10{,}000{,}000 / 125 = USD 80{,}000$
- $FP = 35\\% \Rightarrow 1 - R = 0.65$
- Accrual: 50/360
- Default cash:

$$0.65 \cdot 80{,}000 - 80{,}000 \cdot 0.01 \cdot 0.1388889 = 52{,}000 - 111.11 = USD 51{,}888.89.$$

#### P&L Explain

$$\Delta PV \approx \underbrace{(-18{,}073.33)}_{\text{carry+rolldown}} + \underbrace{46{,}000}_{\text{spread move}} + \underbrace{51{,}888.89}_{\text{default event}} = USD 79{,}815.56.$$

#### Interpretation

Linear spread risk and discrete event risk both matter; carry/rolldown is smaller but non-negligible over time.

---

### Example 11: Tranche PV01

**Bump tranche spread ±1bp and compute PV01 by finite difference.**

#### Setup

- Tranche position notional $N_T = USD 10\text{mm}$
- Tranche $RPV01_T = 3.2$ (toy)
- At-par tranche: contractual spread = market spread, so PV $\approx 0$.

#### Revalue for ±1 bp

**If spread widens by +1 bp:**

$$PV(+1) = N_T \cdot 3.2 \cdot 1 \cdot 10^{-4} = 10{,}000{,}000 \cdot 3.2 \cdot 10^{-4} = USD 3{,}200.$$

**If spread tightens by −1 bp:**

$$PV(-1) = -USD 3{,}200.$$

#### Finite Difference PV01

$$PV01 = \frac{PV(+1) - PV(-1)}{2} = \frac{3{,}200 - (-3{,}200)}{2} = USD 3{,}200/\text{bp}.$$

---

### Example 12: Correlation Shock Toy

**PV(tranche) under low vs high dependence; compute PV change and interpret "correlation risk."**

#### Setup

- Same tranche notional $N_T = USD 10\text{mm}$
- Model-reported PVs (toy) for a given tranche:
  - $PV(\rho = 20\%) = -USD 400{,}000$
  - $PV(\rho = 30\%) = -USD 350{,}000$

#### Correlation P&L for +10% Correlation

$$\Delta PV = -350{,}000 - (-400{,}000) = +USD 50{,}000.$$

#### Correlation 01 (Per 1% Correlation)

$$\text{Corr01} \approx \frac{50{,}000}{10} = USD 5{,}000 \text{ per } 1\\%.$$

#### Interpretation

This tranche benefits (PV increases) when dependence increases. The source emphasizes that corr sensitivity varies by tranche and is measured by Corr01.

---

### Example 13: Tail/Clustering Scenario

**1 default vs 5 clustered defaults; compute tranche loss and PV impact; show why PV01 hedges fail.**

#### Setup

- Portfolio has $M = 100$ equal names; each default has recovery $R = 40\\% \Rightarrow$ loss per default on portfolio notional:

$$\ell = \frac{1 - R}{M} = \frac{0.60}{100} = 0.006 = 0.6\\%.$$

- Consider equity tranche $[A, B] = [0\\%, 3\\%] \Rightarrow B - A = 3\\%$.
- Tranche notional (toy) $N_T = USD 10\text{mm}$.

#### Scenario 1: 1 Default

Portfolio loss $L = 0.6\\%$.

Tranche loss fraction:

$$L_{[0,3]} = \frac{\min(L, 3\\%)}{3\\%} = \frac{0.6\\%}{3\\%} = 0.20.$$

Dollar loss on tranche (for protection seller):

$$0.20 \times 10{,}000{,}000 = USD 2{,}000{,}000.$$

#### Scenario 2: 5 Defaults (Cluster)

Portfolio loss $L = 5 \times 0.6\\% = 3.0\\%$.

Tranche loss fraction:

$$L_{[0,3]} = \frac{3.0\\%}{3\\%} = 1.00.$$

Dollar loss:

$$1.00 \times 10{,}000{,}000 = USD 10{,}000{,}000.$$

#### Why PV01 Hedges Fail

A PV01 hedge is calibrated to small spread moves (linear).

A clustered default scenario creates a jump in realized loss that is not proportional to a 1–5 bp spread bump.

---

### Example 14: Adjacent Tranche Hedge

**Hedge tranche spread PV01 with another tranche; validate under small spread move; show failure under clustering.**

#### Setup

- **Target tranche:** equity $[0, 3]$, notional $N_E = USD 10\text{mm}$, $PV01_E = USD 3{,}200/\text{bp}$ (from Example 11).
- **Hedge tranche:** mezz `[3, 7]`, assume $PV01_M = 1{,}800$ USD/bp per USD 10mm notional (toy).
- **Goal:** PV01-neutral hedge of spread moves.

#### Hedge Ratio

Need $PV01_E + PV01_M \times (N_M / 10\text{mm}) \times \text{sign} = 0$.

If we take opposite spread exposure using mezz, the notional ratio is:

$$\frac{N_M}{10\text{mm}} = \frac{PV01_E}{PV01_M} = \frac{3{,}200}{1{,}800} = 1.7778 \Rightarrow N_M = USD 17.778\text{mm}.$$

#### Check under Small +1 bp Spread Move

- Equity spread P&L: $+3{,}200$ (long protection) or $-3{,}200$ (short protection); choose opposite sign for hedge so it offsets.
- Mezz spread P&L magnitude: $1{,}800 \times 1.7778 = 3{,}200$.
- **Net $\approx 0$. ✅**

#### Failure under Clustering

Use the same portfolio assumptions as Example 13.

**Clustered defaults: 10 defaults** $\Rightarrow L = 10 \times 0.6\\% = 6\\%$.

**Equity tranche `[0, 3]` loss fraction** $= 1.00 \Rightarrow$ USD 10mm loss.

**Mezz tranche $[3, 7]$ loss fraction:**

$$L_{[3,7]} = \frac{\min(\max(6\\% - 3\\%, 0), 4\\%)}{4\\%} = \frac{3\\%}{4\\%} = 0.75.$$

Dollar loss on mezz notional $N_M = 17.778\text{mm}$ is $0.75 \times 17.778\text{mm} = USD 13.333\text{mm}$.

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

## 52.13 Practical Notes

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

### Example 16: Senior vs Subordinated CDS — Recovery-Driven Spread Relationship

**Setup:** Consider a BBB-rated issuer with both senior unsecured and subordinated CDS traded. Market recovery assumptions: $R_{\text{sen}} = 40\\%$, $R_{\text{sub}} = 20\\%$. Senior 5Y CDS trades at $S_{\text{sen}} = 120$ bp.

**Question:** What should the subordinated CDS spread be under the equal-default-probability assumption? If sub CDS actually trades at 170 bp, is the trade attractive?

**Solution:**

Using O'Kane's senior-sub relationship (derived from equal hazard rates, different LGDs):

$$\boxed{\frac{S_{\text{sub}}}{S_{\text{sen}}} = \frac{1 - R_{\text{sub}}}{1 - R_{\text{sen}}}}$$

$$S_{\text{sub}}^{\text{fair}} = 120 \times \frac{1 - 0.20}{1 - 0.40} = 120 \times \frac{0.80}{0.60} = 160 \text{ bp}$$

The theoretical ratio is $0.80/0.60 = 1.333$. Actual market sub spread is 170 bp, giving a ratio of $170/120 = 1.417$.

**Interpretation:** Sub CDS is *wider* than the model predicts by 10 bp. This could indicate:
- Sub recovery is lower than 20% (market pricing ~15% recovery), or
- Sub CDS has excess liquidity premium, or
- A genuine mispricing

**Trade construction (if you believe in the model):** Buy protection on senior, sell protection on sub (harvesting the excess sub spread). Size CS01-neutral:

$$N_{\text{sen}} = N_{\text{sub}} \times \frac{RPV01_{\text{sub}}}{RPV01_{\text{sen}}}$$

If $RPV01_{\text{sen}} = 4.1$ and $RPV01_{\text{sub}} = 3.8$ (lower survival probability compresses RPV01), then for USD 5mm sub notional:

$$N_{\text{sen}} = 5 \times \frac{3.8}{4.1} = 4.63\text{mm}$$

**Failure mode:** If actual sub recovery at default is 5% (not 20%), your "cheap sub protection" was actually fair and you lose on the senior leg. Recovery uncertainty is the dominant risk.

---

### Example 17: HY Curve Steepener — CS01-Neutral Sizing

**Setup:** A high-yield issuer trades with an inverted CDS curve: 3Y at 450 bp, 5Y at 380 bp. You believe the inversion is excessive (the issuer will survive the near-term liquidity crunch) and want to trade the curve flattening.

**Trade:** Buy 5Y protection (long) + Sell 3Y protection (short). This is a "bull flattener" — you profit if the curve flattens (3Y tightens more than 5Y, or 5Y widens less than 3Y).

**CS01-neutral sizing:** Per O'Kane's framework, match the CS01 so parallel moves cancel:

Given: $RPV01_{3Y} = 2.5$, $RPV01_{5Y} = 3.6$ (both per USD 1mm notional).

$$N_{3Y} = N_{5Y} \times \frac{RPV01_{5Y}}{RPV01_{3Y}} = 10\text{mm} \times \frac{3.6}{2.5} = 14.4\text{mm}$$

**P&L scenarios (per USD 10mm 5Y notional, USD 14.4mm 3Y notional):**

| Scenario | 3Y move | 5Y move | 3Y P&L | 5Y P&L | Net P&L |
|----------|---------|---------|--------|--------|---------|
| Parallel +50bp | +50 | +50 | $+50 \times 14.4 \times 2.5 \times 10^{-4} = +18{,}000$ USD | $-50 \times 10 \times 3.6 \times 10^{-4} = -18{,}000$ USD | **USD 0** |
| Curve flattens | -100 | -30 | $-100 \times 14.4 \times 2.5 \times 10^{-4} = -36{,}000$ USD | $+30 \times 10 \times 3.6 \times 10^{-4} = +10{,}800$ USD | **-USD 25,200** |
| Curve normalizes (3Y tightens) | -150 | +20 | $-54{,}000$ USD | $-7{,}200$ USD | **-USD 61,200** |
| Default | — | — | Lose on 3Y (short prot) | Gain on 5Y (long prot) | Net depends on timing |

Wait — check the signs. Short protection on 3Y means CS01 $\lt 0$; when 3Y spreads *tighten* (move negative), short protection *gains*:

| Scenario | 3Y move | 5Y move | 3Y P&L (short prot) | 5Y P&L (long prot) | Net P&L |
|----------|---------|---------|---------------------|---------------------|---------|
| Curve flattens (3Y tightens -100, 5Y flat) | -100 | 0 | $+36{,}000$ USD | USD 0 | **+USD 36,000** |
| Curve normalizes (3Y -150, 5Y -30) | -150 | -30 | $+54{,}000$ USD | $-10{,}800$ USD | **+USD 43,200** |
| Default | — | — | `~-(1-R) * 14.4mm` | `~+(1-R) * 10mm` | **-USD 2.6mm** net loss |

**Key risk:** JTD is *not* hedged. The 3Y short protection notional exceeds 5Y long protection notional, so default produces a net loss of approximately $(1-R)(14.4 - 10) = 0.65 \times 4.4 = USD 2.86\text{mm}$.

> **Desk Reality:** This is why HY curve trades are dangerous — the CS01-neutral sizing creates a JTD mismatch. Some desks cap the notional ratio or add a JTD overlay.

---

### Example 18: Negative Basis P&L Under Funding Stress

**Setup:** You hold a negative basis package: long a corporate bond at Z-spread of 180 bp, long protection via 5Y CDS at 150 bp. The basis is $-30$ bp (CDS tighter than bond spread). You fund the bond position at LIBOR + 50 bp.

**Carry calculation (annualized, per $10mm notional):**

- Bond coupon received (spread component): $+180$ bp $= +USD 180{,}000$/yr
- CDS premium paid: $-150$ bp $= -USD 150{,}000$/yr
- Funding cost (spread over LIBOR): $-50$ bp $= -USD 50{,}000$/yr
- **Net carry: $-20$ bp $= -USD 20{,}000$/yr**

The "free lunch" of $-30$ bp basis is consumed by $-50$ bp funding cost, leaving negative net carry.

**Funding stress scenario:** Suppose your funding cost widens to LIBOR + 120 bp (credit crunch):

- New net carry: $180 - 150 - 120 = -90$ bp $= -USD 90{,}000$/yr
- The position now *bleeds* $USD 90{,}000$/yr while waiting for convergence

**MTM impact:** If market expects elevated funding for 2 years, the MTM loss is approximately:

$$\Delta PV \approx -90 \text{ bp} \times RPV01_{\text{bond}} \times \text{Notional} \approx -90 \times 4.0 \times 10^{-4} \times 10{,}000{,}000 = -USD 360{,}000$$

**Desk note:** funding and balance-sheet costs are institution-specific. The same “basis” package can be positive carry for one participant and negative carry for another, and funding terms can change rapidly in stress—so always stress the financing leg, not just the spread leg.

---

### Example 19: Equity-CDS Relative Value — LBO Scenario

**Setup:** Company XYZ trades at equity market cap $8bn, with CDS at 90 bp (5Y). You observe that the equity is pricing minimal distress while CDS is relatively cheap. You believe an LBO is possible, which would load leverage onto the firm.

**Pre-LBO position:** Sell equity (short stock) + Buy CDS protection (long protection). This is the classic "short equity, long credit protection" Merton trade.

**LBO announcement:** Private equity acquires XYZ with 4× leverage. What happens?

| Instrument | Pre-LBO | Post-LBO | P&L direction |
|------------|---------|----------|---------------|
| Equity | $40/share | $55/share (premium) | **Loss** on short equity |
| Senior CDS | 90 bp | 350 bp (leverage spike) | **Gain** on long protection |
| Sub CDS | 200 bp | 800 bp | Even larger gain (if traded) |

**Sizing matters:** If you sized the trade so that:
- Short equity: $5mm notional
- Long CDS protection: $5mm notional, RPV01 = 4.3

Equity loss: $5\text{mm} \times 37.5\% = -USD 1.875\text{mm}$ (stock jumps from $40 to $55)

CDS gain: $(350 - 90) \times 4.3 \times 10^{-4} \times 5{,}000{,}000 = +USD 559{,}000$

**Net: $-USD 1.316\text{mm}$** — the CDS gain doesn't cover the equity loss because CDS leverage is lower than equity sensitivity.

> **Desk Reality:** LBO trades are notoriously difficult to size correctly. The equity move is discontinuous (tender premium), while CDS moves are large but capped. Professional desks often use options on equity (puts) rather than short stock to limit downside, and overweight the CDS leg. The key insight from the Merton model is that equity is effectively a call option—its sensitivity to leverage events can dwarf CDS sensitivity.

These post‑event levels are **toy numbers** for sizing intuition only. Actual moves depend on the deal structure, leverage, sector, and market regime.

---

### Example 20: Correlation Shock on Equity-Senior Combo

**Setup (toy):** you build a Corr01‑neutral package at $\rho=25\\%$ using two tranches with opposite Corr01 signs.

Assume (per $USD 10$mm tranche notional):
- Equity tranche (0–3%, **long protection**): Corr01 $= -USD 8{,}000$ per +1% corr
- Senior tranche (7–10% or 15–30%, **long protection**): Corr01 $= +USD 15{,}000$ per +1% corr

To Corr01‑neutralize locally, take $USD 10$mm senior long protection and $USD 18.75$mm equity long protection:

$$+15{,}000 \\; + \\; 1.875\times(-8{,}000) \approx 0.$$

**Scenario:** correlation jumps from 25% to 40% (a +15% shock).

Corr01 itself is not constant; suppose at $\rho=40\\%$ the sensitivities become:
- Equity Corr01 $= -USD 5{,}000$ per +1%
- Senior Corr01 $= +USD 18{,}000$ per +1%

Then the package is no longer Corr01‑neutral at the new level:

$$NetCorr01_{40\\%} \approx 18{,}000 + 1.875\times(-5{,}000) = 8{,}625 \text{ USD per 1\\%}.$$

A crude convexity estimate uses the average net Corr01 over the move:

$$\Delta PV_{\rho} \approx \frac{0 + 8{,}625}{2} \times 15 \approx +USD 64{,}700.$$

**Takeaway:** Corr01‑neutralization is a *local* hedge; large correlation moves introduce convexity/hedge‑ratio drift. Validate correlation packages with stressed moves, not just +/‑1% bumps.

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

**22.** You want to construct a CS01-neutral 3Y-5Y HY curve steepener. 5Y notional is USD 10mm with $RPV01_{5Y} = 3.4$. 3Y has $RPV01_{3Y} = 2.2$. (a) Compute 3Y notional. (b) Compute the JTD mismatch at recovery $R = 30\%$. (c) Should you add a JTD overlay? How?

> **Sketch:** (a) $N_{3Y} = 10 \times 3.4/2.2 = 15.45\text{mm}$. (b) JTD mismatch: $(1-R)(N_{3Y} - N_{5Y}) = 0.70 \times 5.45 = USD 3.82\text{mm}$ net loss on default. (c) Yes — buy additional 5Y protection for the JTD gap: $\Delta N_{5Y} = 5.45/(1) = 5.45\text{mm}$ (adjusting for the fact that both legs pay $(1-R)$ at default, so you need $N_{3Y} - N_{5Y} = 5.45\text{mm}$ of additional 5Y protection). This breaks CS01 neutrality — trade-off between curve and default risk.

---

**23.** A negative basis package has: bond spread 200 bp, CDS spread 160 bp (basis = $-40$ bp), funding cost = LIBOR + 60 bp. (a) Compute annual carry per USD 10mm. (b) At what funding cost does the trade break even? (c) If funding widens to LIBOR + 150 bp during a crisis and stays there for 6 months, what is the carry loss?

> **Sketch:** (a) Carry $= (200 - 160 - 60) \times 10^{-4} \times 10{,}000{,}000 = -USD 20{,}000$/yr (negative carry). (b) Break-even: $200 - 160 - x = 0 \Rightarrow x = 40$ bp. (c) At LIBOR + 150: carry $= (200-160-150) = -110$ bp $= -USD 110{,}000$/yr. Over 6 months: $-USD 55{,}000$.

---

**24.** An LBO increases a firm's leverage from 2× to 5× EBITDA. Pre-LBO: senior CDS = 80 bp, equity price = $45. Post-LBO: senior CDS = 280 bp, equity tender at $60. You are short $3mm equity and long $8mm CDS protection with $RPV01 = 4.0$. (a) Compute equity leg P&L. (b) Compute CDS leg P&L. (c) Was the trade profitable? (d) What sizing change would improve the outcome?

> **Sketch:** (a) Equity loss: $3\text{mm} \times (60-45)/45 = 3\text{mm} \times 33.3\% = -USD 1{,}000{,}000$. (b) CDS gain: $(280-80) \times 4.0 \times 10^{-4} \times 8{,}000{,}000 = +USD 640{,}000$. (c) Net: $-USD 360{,}000$ — unprofitable. (d) Increase CDS notional relative to equity — the CDS leg needs more weight because equity jumps are larger (percentage) than CDS spread moves. Alternatively, use equity puts to cap equity loss.

---

**25.** You hold a Corr01-neutral equity-senior tranche combo. (a) Why can this position still lose money on a large correlation shock? (b) What additional risk measure would you monitor? (c) Describe a scenario where the "neutral" position produces a large loss.

> **Sketch:** (a) Corr01 is a *local* (first-order) measure. For large moves, the correlation sensitivity itself changes — this is "correlation convexity" or second-order correlation risk. (b) Monitor the *change* in Corr01 as correlation moves (i.e., second-order sensitivity), and run full revaluation under ±10-20% correlation shocks. (c) A rapid decorrelation event (e.g., one sector defaults while others are fine) can cause equity tranche to move violently while senior barely responds — the local hedge ratio is stale and losses accumulate before rebalancing.

---

## References

- Dominic O'Kane, *Modelling Single-name and Multi-name Credit Derivatives* (bond-CDS basis drivers; CDS indices and index basis; roll mechanics; tranche risk measures/correlation; capital structure and recovery; LCDS)
- John C. Hull, *Risk Management and Financial Institutions* (fixed-coupon CDS/index quoting; CDS-bond basis; tranche correlation intuition)

*Chapter 52 of Fixed Income: Practice and Theory*
