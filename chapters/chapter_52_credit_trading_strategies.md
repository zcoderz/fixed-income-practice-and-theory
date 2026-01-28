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

## 52.1 Conventions and Setup

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

## 52.2 Core Risk Measures

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

O'Kane provides the precise intrinsic valuation framework. The intrinsic upfront value of an index (viewed as an equally weighted portfolio of constituent CDS) is:

$$\boxed{V_I(t) = \frac{1}{M} \sum_{m=1}^{M} (S_m(t,T) - C(T)) \cdot \text{RPV01}_m(t,T)}$$

The market upfront value using a flat index curve is:

$$\boxed{U_I(t) = (S_I(t,T) - C(T)) \cdot \text{RPV01}_I(t,T)}$$

O'Kane explains: "We call this the intrinsic value since we have calculated the value of the index swap as the sum of the values of its constituent parts."

#### Index Basis

The index basis is the gap between quoted index spread and CDS-implied intrinsic spread. O'Kane provides empirical examples from Table 10.6:

| Index | Term | Quoted (bp) | Intrinsic (bp) | Basis (bp) |
|-------|------|-------------|----------------|------------|
| CDX NA IG S7 | 5Y | 34 | 33.1 | +0.9 |
| CDX NA IG S7 | 10Y | 55 | 55.3 | -0.3 |
| CDX NA HY S7 | 5Y | 276 | 275 | +1.0 |
| CDX NA HY S7 | 10Y | 315 | 324 | -9.0 |
| iTraxx Europe S6 | 5Y | 24 | 26.9 | -2.9 |
| iTraxx Europe S6 | 10Y | 42 | 45.8 | -3.8 |

O'Kane lists specific drivers of index basis:

1. **Restructuring clause**: "In the case of the North American CDX index, the payment of protection on the index protection leg is only triggered when the credit event is a bankruptcy or failure to pay... However, the market standard for CDS in the US is based on the use of the Mod-Re restructuring clause. Since No-Re spreads are typically about 5% lower than Mod-Re spreads, we have an immediate basis."

2. **Liquidity premium**: "The considerable size and liquidity of the CDS index market means that the CDS index spread embeds a lower liquidity risk premium than the less liquid CDS spreads."

3. **Index leads the market**: "The CDS index may be considered to lead the CDS market. This is especially true in a widening market where investors use long protection positions in the index to hedge illiquid long credit positions."

#### The Portfolio Swap Adjustment

O'Kane describes the portfolio swap adjustment as the mechanism to reconcile intrinsic and quoted values: "One way to ensure that the index swap spread equals the intrinsic swap spread is to see if there is an adjustment that can be made to the individual issuer curves which can enforce [equality]."

The adjustment is "somewhat arbitrary" but typically proportional: "we would rather increase all spreads by 1% than add say 5 bp to all spreads which would have a large effect on the credit risk of a name trading at 10 bp and a much smaller effect on another trading at 200 bp."

**I'm not sure** about exact portfolio swap adjustment algorithms beyond the proportional approach O'Kane describes—implementations vary by desk and system.

#### Series/Roll Basis

Indices roll every six months; on-the-run liquidity and maturity reset can create price/spread differences between old and new series.

#### Intuition

- "Quoted vs intrinsic" is a model/replication gap driven by restructuring clauses, liquidity, and market dynamics.
- "Series basis" is a liquidity + contract cohort gap.
- The portfolio swap adjustment is required for any tranche or structured product pricing built on the index.

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

For a bond portfolio, carry is the coupon accrual minus financing cost; rolldown is price change from moving "down" a sloped curve as time passes.

#### CDS / Index / Tranche Carry and Theta

O'Kane provides precise definitions for tranches that apply equally to indices:

> "**Carry**: This is the daily coupon which is accrued. Although coupons are only paid on the quarterly payment dates, we effectively accrue a risk-free coupon each day. This is because the coupon which was accrued since yesterday will be sure to be paid whether or not default occurs since it will be part of the premium accrued at default."

> "**Theta**: This is the daily change in the full price of the tranche. For a long protection position it is the change in the value of the protection leg minus the change in the value of the premium leg."

O'Kane explains why theta is typically negative for long protection positions: "As we move one day forward, the value of the protection leg falls as we have one day less of protection. Over the same period, the value of the premium leg increases as the future cash flows move one day closer to today and so their time value increases."

From O'Kane's Table 17.2, representative daily values for a $1,250mm reference portfolio:

| Tranche | Daily Carry ($) | Daily Theta ($) |
|---------|-----------------|-----------------|
| 0-3% | -18,229 | -16,632 |
| 3-7% | -5,556 | -7,553 |
| 7-10% | -1,563 | -2,583 |
| 10-15% | -1,042 | -1,971 |
| 15-30% | -417 | -1,068 |

O'Kane notes: "The magnitude of the daily carry is largest for the 0-3% tranche which is paying the highest spread. It is negative as a protection buyer has to pay the premium."

#### Rolldown Component

Theta includes the rolldown effect. O'Kane explains: "If the CDS spread curves are upward sloped, as time passes we roll down the curve with the effect of reducing the average portfolio spread."

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

### A1) Bond–CDS Basis Framework (Cash vs Synthetic)

The bond-CDS basis—the spread difference between a CDS and the equivalent cash bond—is one of the most studied phenomena in credit markets. O'Kane defines the CDS basis as:

$$\boxed{\text{CDS basis} = \text{CDS spread} - \text{Bond Libor spread}}$$

where "for a fixed rate bond, the natural choice is the asset swap spread of a bond trading close to par." A positive basis means CDS protection is more expensive than the bond spread suggests; a negative basis means protection is cheap relative to cash.

The basis can persist for extended periods because CDS and bonds are fundamentally different contracts. O'Kane divides the drivers into **fundamental factors** (contractual differences) and **market factors** (trading dynamics). Understanding both is essential for any basis strategy.

#### 3.1.1 Fundamental Factors Driving the Basis

O'Kane identifies six fundamental factors that create structural differences between CDS and bond spreads:

1. **Funding**: "Default swaps are unfunded transactions and bonds are funded. For the same spread, CDS are favoured by investors who have funding costs above Libor while bonds are favoured by those who fund below." This creates a natural clientele effect—investors sort themselves into CDS or bonds based on their funding costs, and the basis reflects the marginal participant's funding level.

2. **The delivery option**: "In a CDS contract, the protection buyer is able to choose which physical asset out of a basket of obligations to deliver following a credit event." O'Kane provides a concrete example: if the protection buyer holds an asset trading at 43% and another deliverable trades at 37%, they can sell the first, buy the second, and deliver it for par, capturing a $6 profit from the option. This optionality makes long protection more valuable, widening the CDS spread and the basis.

3. **Technical default**: "The standard credit events may be viewed as being broader than those which constitute default on a bond." CDS can trigger on restructuring or other events that don't technically default the bond, so protection sellers demand compensation for this additional risk.

4. **Loss on default**: "The protection payment on a CDS following a credit event is a fraction $(1-R)$ of the face value of the contract. The default risk on a bond (or asset swap) purchased at a full price of $P$ is to a loss of $P-R$." This is the **loss-on-default mismatch**: if you buy a bond at 105 and it defaults with 40% recovery, you lose $65; but CDS protection pays only $60 (per 100 notional). This asymmetry affects how much CDS protection to buy.

5. **Premium accrued at default**: "Following a credit event, a CDS pays the protection seller the premium which has accrued since the previous payment date. However, when a bond defaults, the owner's claim is on the face value, and so any accrued coupons are lost." This difference slightly lowers the CDS spread relative to bonds.

6. **CDS spreads cannot be negative**: Unlike asset swap spreads (which can go negative for very high-quality credits like Treasuries), CDS spreads must be positive to compensate for default risk and transaction costs.

#### 3.1.2 Market Factors Driving the Basis

O'Kane lists six market factors that create basis dynamics:

1. **Relative liquidity**: "The CDS market has its liquidity points at fixed term points with most liquidity concentrated at the three-, five-, seven- and 10-year maturity 'IMM' dates," while bonds have liquidity at their specific issue maturities. Maturity mismatches create basis effects.

2. **Synthetic CDO technical short**: "When dealers issue synthetic CDOs, they then hedge their spread risk by selling CDS credit protection on each of the 100 or more credits in the reference portfolio. CDO issuance is therefore usually accompanied by a tightening (reduction) in CDS spreads."

3. **New issuance/loans**: "When bonds are newly issued by a corporate, CDS spreads often widen... Banks who have assumed large credit exposures to corporates by granting them loans, hedge this risk in the CDS market."

4. **Convertible issuance**: "The issuance of convertible bonds often results in a demand to buy protection on the issuer" as convertible arbitrage funds isolate equity volatility risk.

5. **Demand for protection**: "It is much easier to go short a credit by buying protection in the CDS market than by shorting a cash bond. As a result, if there is negative news on a credit, it will tend to result in a flurry of protection buying."

6. **Funding risk**: "Since the CDS is unfunded, it effectively locks in a funding rate of Libor flat until maturity or a credit event... There is therefore no funding risk."

O'Kane concludes: "All of these fundamental and market factors are reasons why the CDS and cash market spreads should diverge. It is difficult to assign degrees of importance to the individual factors as it is hard to tease them apart empirically."

#### Strategy Card: A1 — Bond–CDS Basis Package (Risk-First Framing)

##### Objective (Conceptual; No Forecasts)

Isolate and measure the CDS–cash basis while understanding which factors are driving any observed divergence. As O'Kane notes, basis trading has become "a new relative value opportunity" for market participants who understand the risk decomposition.

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

The basis trade can fail in multiple ways, corresponding to the fundamental and market factors O'Kane identifies:

**Funding-Related Failures:**
- **Funding/repo shocks**: The bond leg is funded; CDS is unfunded. O'Kane notes: "Since the CDS is unfunded, it effectively locks in a funding rate of Libor flat until maturity or a credit event." If your funding costs spike (e.g., during a credit crisis), the bond leg becomes increasingly expensive to carry while the CDS leg is unaffected. This asymmetry can overwhelm a small positive carry.
- **Repo fails or specials**: If the specific bond becomes hard to finance (goes "special"), carrying the cash leg becomes punitive. This is a funding risk that CDS does not face.

**Contract-Related Failures:**
- **Delivery option exercise**: O'Kane's example shows a $6 profit from delivery option when the cheapest deliverable trades 6 points below the hedged asset. If you're short protection and a restructuring occurs, the protection buyer will deliver the cheapest asset—not the one you expected.
- **Restructuring clause mismatch**: O'Kane notes that "the choice of restructuring clause should affect the CDS spread for that contract." Trading a Mod-Re CDS against a No-Re index (or vice versa) creates basis risk to restructuring events.
- **Loss-on-default mismatch**: O'Kane emphasizes: "The protection payment on a CDS following a credit event is a fraction $(1-R)$ of the face value... The default risk on a bond purchased at a full price of $P$ is to a loss of $P-R$." If your bond is at 105 and defaults at 40% recovery, you lose 65 points but CDS only pays 60. If your bond is at 95, you lose 55 but CDS pays 60—now you're over-hedged.

**Market-Related Failures:**
- **CDO issuance waves**: O'Kane notes that "CDO issuance is therefore usually accompanied by a tightening (reduction) in CDS spreads" as dealers hedge by selling protection. A sudden wave of CDO issuance can compress the basis against a long-basis position.
- **Convertible issuance**: "The issuance of convertible bonds often results in a demand to buy protection on the issuer" as hedge funds isolate equity volatility. This widens the CDS leg without affecting bonds.
- **Liquidity divergence**: In stress, "it is much easier to go short a credit by buying protection in the CDS market than by shorting a cash bond." CDS spreads can gap wider than bond spreads, creating violent basis moves.

**Quantitative Failure Indicators:**
- CS01 mismatch exceeds 10% after hedge
- JTD mismatch exceeds 5% due to bond price ≠ 100
- Repo rate volatility > 50bp annualized
- Bid-offer > expected carry over holding period

##### Implementation Checklist + Verification Tests

- Reprice each leg under the same discounting assumptions where applicable (and document differences).
- Verify signs: long protection CS01 $> 0$; long bond credit CS01 $< 0$.
- Bump stability: CS01 should be stable for $\pm 1$ bp bumps; if not, you are in nonlinear regime.
- Event test: run default today with recovery sweep.

#### Negative Basis: Why It Persists

A "negative basis" means the CDS spread trades *inside* (below) the bond asset-swap spread: the synthetic market prices credit risk cheaper than the cash market. This seemingly arbitrageable condition can persist for extended periods because of structural demand dynamics.

> **Desk Reality: The CLO Bid and Bank Funding Arbitrage**
>
> O'Kane identifies the key mechanism: "Default swaps are unfunded transactions and bonds are funded. For the same spread, CDS are favoured by investors who have funding costs above Libor while bonds are favoured by those who fund below."
>
> This creates a natural clientele separation. Banks that fund below Libor have a structural advantage in owning cash bonds—they earn the full bond spread while paying below-Libor funding, capturing excess carry. Meanwhile, investors who fund above Libor prefer the unfunded CDS, which locks in Libor-flat funding.
>
> **The CLO demand channel** reinforces this. Collateralized Loan Obligations need floating-rate spread exposure and are natural buyers of bonds and loans. Their demand pushes bond spreads tighter relative to CDS, creating systematic negative basis. When CLO issuance is strong, negative basis tends to compress further.
>
> **Practitioner Note (Claude-extended):** In practice, the negative basis trade—buying the bond and buying CDS protection—is a carry trade that earns the spread differential minus funding costs. Banks with low funding costs can earn positive carry on this package even when the CDS spread is below the bond spread, because their all-in funding cost is below Libor. This creates a structural floor for negative basis: it persists until bank balance sheet capacity is exhausted or funding conditions change.

#### Positive Basis: The Squeeze

A "positive basis" means CDS spreads trade *outside* (above) bond spreads. O'Kane's market factors explain why: demand for protection (shorting credit is easier via CDS), CDO dealer hedging, convertible arbitrage flows, and new issuance effects all tend to push CDS spreads wider relative to bonds.

The dangerous scenario is the **positive basis squeeze**: a trader who is short the basis (short CDS, long bond) faces losses when the basis widens further. In stress:

1. **Protection demand surges**: O'Kane notes that "if there is negative news on a credit, it will tend to result in a flurry of protection buying in the CDS market"—widening CDS but not necessarily bonds.
2. **Repo funding spikes**: The bond leg becomes expensive to carry as repo rates rise (Example 1B illustrates this).
3. **Forced unwinds**: Margin calls on the CDS leg force liquidation at the worst time, crystallizing losses.

> **Practitioner Note (Claude-extended):** During the 2008 financial crisis and the COVID-19 shock in March 2020, CDS-bond basis dislocated dramatically. Negative basis positions that appeared profitable under normal conditions faced funding stress as repo markets seized and mark-to-market losses triggered margin calls. The key lesson: basis trades are funding-contingent strategies. A trader must model the worst-case funding scenario, not just the base case. I'm not sure about exact basis magnitudes during these episodes without additional data sources; the qualitative pattern is well-established but specific numbers require market data verification.

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

### A3) CDS Curve Steepener / Flattener

Credit curve trades express a view on the *shape* of a single name's CDS spread term structure rather than its level. They are a natural extension of the relative value framework from Chapter 44.

#### 3.3.1 Curve Shapes and What They Signal

O'Kane provides three representative curve shapes (Table 7.2): (i) a flat curve, (ii) an upward-sloping curve, and (iii) a steeply inverted curve. The inverted curve is particularly important for high-yield names:

- **Upward-sloping (typical IG):** Long-term default probability exceeds short-term. The hazard rate increases with tenor. This is normal for investment-grade credits where the risk of eventual deterioration exceeds near-term default risk.
- **Inverted (distressed HY):** Short-term default probability dominates long-term. O'Kane notes that for a name with very high short-term spreads, "it would not be possible to redeem the bond if it matured immediately." If the firm survives the near term, it is likely to survive longer—hence spreads decline at longer tenors.

> **Desk Reality: Why HY Curves Invert**
>
> **Practitioner Note (Claude-extended):** When a credit is in distress, the market prices near-term default as the dominant risk. A name trading at 800 bp in 1Y but 500 bp in 5Y is telling you: "If this company survives the next year, it's probably OK." This creates a natural curve steepener trade—buy short-dated protection (expensive but high expected payoff) and sell long-dated protection (cheaper, and you bet on survival). The key risk: if the name defaults, both legs trigger and the net depends on recovery and premium accrual differences.

#### 3.3.2 Forward CDS Curves

O'Kane shows how spot CDS curves map to forward curves (Table 9.1). For an upward-sloping curve, the forward curve is higher than the spot curve—forward-starting protection is more expensive, reflecting the market's expectation of higher future default risk. For an inverted curve, the forward curve is lower than spot, reflecting declining expected hazard rates.

#### 3.3.3 Curve Arbitrage Bounds

O'Kane derives an arbitrage lower bound for inverted curves. For a curve starting at 800 bp at 6 months, the approximate lower bound is:

$$S_m \gtrsim S_{m-1} \left(\frac{T_{m-1}}{T_m}\right)$$

Table 7.3 in O'Kane gives the exact bounds: a 1Y spread cannot fall below ~419 bp if 6M trades at 800 bp (assuming 40% recovery). Any inverted curve breaching these bounds is arbitrageable through calendar-spread strategies.

#### Strategy Card: A3 — CDS Curve Steepener/Flattener

##### Objective (Conceptual; No Forecasts)

Express a view on the shape of a single name's credit curve: steepening (short end widens relative to long end) or flattening (long end widens relative to short end). Alternatively, express a "survival bet"—the steepener profits if the name survives and the curve normalizes.

##### Instruments and Position Construction

- **Steepener:** Buy protection at short tenor (e.g., 3Y), sell protection at long tenor (e.g., 5Y or 10Y).
- **Flattener:** The reverse.
- Size the trade CS01-neutral to isolate the curve shape view from parallel spread moves.

##### Exposure Decomposition (Units Explicit)

| Exposure (units) | Short leg | Long leg | Net |
|------------------|-----------|----------|-----|
| $\text{CS01}_{3Y}$ (USD/bp) | $> 0$ (long prot) | 0 | positive |
| $\text{CS01}_{5Y}$ (USD/bp) | 0 | $< 0$ (short prot) | negative |
| $\text{CS01}_{\text{net}}$ (USD/bp) | — | — | ~0 by construction |
| JTD (USD) | positive (receive prot pmt) | negative (pay prot pmt) | net depends on accrual |
| Rec01 (USD/1%) | meaningful | meaningful | partial offset |

##### Hedge Set and Hedge Ratios (Show Math)

**CS01-neutral sizing:**

$$\boxed{N_{\text{long}} = N_{\text{short}} \times \frac{\text{RPV01}_{\text{short}}}{\text{RPV01}_{\text{long}}}}$$

For a 3Y/5Y steepener with $\text{RPV01}_{3Y} = 2.8$ and $\text{RPV01}_{5Y} = 4.5$:

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

The index roll is a critical event in CDS index markets. O'Kane explains: "The indices roll every six months and investors who hold the existing on-the-run index will generally try to roll into the new on-the-run index by selling the old index contract and buying the new one."

#### 3.4.1 What Drives Roll P&L

O'Kane identifies two primary sources of P&L impact during the roll:

1. **Composition changes**: "Any changes in the composition of the index which change its perceived credit quality. For example, the new index will not contain any names which have been downgraded below investment grade or whose liquidity has declined." However, this does not guarantee the new index trades tighter: "liquidity requirements may result in some names being replaced with wider spread names."

2. **Maturity extension**: "The new index has a longer maturity, by six months, than the previous index. As credit curves tend to be upward sloping, this would tend to cause the new index spread to be wider, all other things being the same."

These two effects can work in opposite directions. A credit-upgraded new index with a longer maturity may trade at the same spread as the old index if the composition improvement offsets the curve steepness.

#### 3.4.2 The Series Basis

The difference between the old (off-the-run) and new (on-the-run) series creates a **series basis** that can persist due to:
- **Liquidity premium**: On-the-run indices typically have tighter bid-offer spreads
- **Maturity mismatch**: Different remaining maturities mean different RPV01 values
- **Composition differences**: Credits may enter or exit the index

#### Strategy Card: B2 — Index Roll/Series Switch (Mechanics + Risk)

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

## 52.6 Strategy Family C: Correlation and Tranche Relative Value

Tranches are fundamentally different from single-name CDS and indices. Their value depends not just on spread levels but on the *distribution* of portfolio losses—and that distribution is shaped by correlation. O'Kane provides a comprehensive risk framework that reveals why tranche positions often behave counter-intuitively.

### 52.6.1 Tranche PV Decomposition and Why Correlation Matters

#### PV Decomposition (Conceptual)

A tranche behaves like a credit derivative on the portfolio loss distribution: PV depends on expected discounted protection leg (driven by losses) and premium leg (driven by outstanding tranche notional).

#### Systemic vs Idiosyncratic Risk

O'Kane distinguishes two types of spread risk for tranches:

1. **Systemic risk**: "the risk to a parallel movement of all of the CDS spreads in the portfolio" (measured by systemic delta and systemic gamma)
2. **Idiosyncratic risk**: "the risk to the spread movements of one credit" holding other credits fixed

These risks behave very differently across the capital structure. Table 17.2 from O'Kane shows representative values for a 125-name portfolio with 60 bp average spread:

| Tranche | Systemic Delta ($mm) | Leverage | Systemic Gamma ($) | Corr01 ($) |
|---------|---------------------|----------|-------------------|------------|
| 0-3% | 691 | 18.37 | -4,013 | -433,553 |
| 3-7% | 461 | 9.20 | 47 | 23,214 |
| 7-10% | 170 | 4.52 | 667 | 78,274 |
| 10-15% | 133 | 2.12 | 901 | 103,158 |
| 15-30% | 73 | 0.39 | 805 | 93,228 |

The table reveals key structural features:
- **Equity tranches are highly leveraged** (18× for 0-3%) with negative systemic gamma
- **Senior tranches are deleveraged** (0.39× for 15-30%) with positive systemic gamma
- **Correlation sensitivity reverses**: equity tranches lose on correlation increases; senior tranches gain

#### Correlation/Dependence Affects Tail

Higher correlation increases the probability of clustered defaults (tail events), which redistributes expected losses across attachment points. O'Kane explains: "The correlation 01 of the 0-3% tranche is negative, resulting in a loss of over $400,000 for a 1% increase in correlation. The correlation 01 of the 15-30% tranche is positive."

This creates a natural tension: equity and senior tranches have *opposite* correlation exposures. A correlation trader can exploit this by combining tranches to isolate or neutralize correlation risk.

#### Gamma Trading

O'Kane describes how market participants "use the gamma risk profile of tranches as a way to express a view on systemic versus idiosyncratic credit scenarios, while remaining more or less credit spread neutral."

The key insight from Table 17.5 in O'Kane:

| Tranche | Long Protection | Short Protection |
|---------|-----------------|------------------|
| Equity | I.Gamma > 0, S.Gamma < 0 | I.Gamma < 0, S.Gamma > 0 |
| Senior | I.Gamma < 0, S.Gamma > 0 | I.Gamma > 0, S.Gamma < 0 |

This means:
- Long protection on equity **benefits** from idiosyncratic spread moves but **loses** on systemic moves
- Long protection on senior tranches has the opposite profile

O'Kane notes: "Once we have put on an idiosyncratic delta hedge to every credit in the portfolio, we typically find that there is a net carry which may be positive or negative... We typically find that a positive gamma position is associated with a negative carry."

#### Base Correlation Framework

O'Kane describes base correlation as "the standard convention for quoting implied correlation" that "has since become the standard convention for quoting implied correlation and is also widely used for pricing and risk managing synthetic CDOs."

The key insight: "any tranche can be represented as a linear combination of equity tranches." A 3-7% tranche can be replicated as long $7 face of 0-7% base tranche and short $3 face of 0-3% base tranche.

**Warning on base correlation interpolation**: O'Kane shows that "a direct linear interpolation of the base correlation curve is not guaranteed to produce prices for non-standard tranches which are arbitrage free." The interpolation scheme matters significantly for pricing non-standard strikes.

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

### 52.6.2 Tranche Example: Equity vs Senior Correlation Trade

To illustrate the opposing correlation exposures, consider a stylized equity-senior structure:

#### Setup

- **Reference portfolio**: 125 names, $10mm each, total $1,250mm
- **Equity tranche**: 0-3% ($37.5mm), long protection
- **Senior tranche**: 15-30% ($187.5mm), short protection
- **Average portfolio spread**: 60 bp
- **Correlation**: 25%

From O'Kane's Table 17.2:
- Equity Corr01 = -$433,553 per 1% correlation (long protection)
- Senior Corr01 = +$93,228 per 1% correlation (short protection)

But note we are *short* protection on senior, so our Corr01 is *negative* $93,228.

#### Correlation-Neutral Sizing

To neutralize correlation exposure:

$$N_{\text{senior}} = \frac{|\text{Corr01}_{\text{equity}}|}{|\text{Corr01}_{\text{senior}}|} = \frac{433,553}{93,228} = 4.65$$

So we need 4.65× the notional on senior short protection relative to equity long protection to be correlation-neutral.

#### Residual Exposures

Even with correlation neutralized:
- **Systemic delta remains**: equity has $691mm delta, senior has $73mm × 4.65 = $340mm in opposite direction; net delta ~$350mm
- **Gamma profile differs**: equity has negative systemic gamma, senior has positive; the combination has mixed gamma depending on spread level
- **Default clustering**: actual defaults will hit equity first; the hedge only works for small correlation moves, not actual clustered defaults

#### Interpretation

O'Kane's framework reveals that correlation-neutral is not risk-neutral. The trade still has substantial spread delta, gamma profile that changes with spread levels, and nonlinear exposure to actual default clusters.

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
│   └─ Dispersion view → Gamma trade per O'Kane Table 17.5
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

Capital structure arbitrage exploits mispricings between different levels of a firm's debt (senior vs subordinated) or between debt and equity. These strategies draw on the structural relationship between seniority, recovery, and spread—and on the Merton model's insight that equity and debt are both claims on the same underlying asset value.

### 52.7.1 Recovery Rates by Seniority

O'Kane provides empirical recovery data (Table 3.2, sourced from Altman et al. 2003b) that anchors capital structure relationships:

| Seniority | Debt Type | Median Recovery (%) | Mean Recovery (%) | Std Dev (%) |
|-----------|-----------|--------------------|--------------------|-------------|
| Senior secured | Loans | 73.00 | 68.50 | 24.4 |
| Senior unsecured | Loans | 50.50 | 55.00 | 28.4 |
| Senior secured | Bonds | 54.49 | 52.84 | 23.1 |
| Senior unsecured | Bonds | 42.27 | 34.89 | 26.6 |
| Senior subordinated | Bonds | 32.35 | 30.17 | 25.0 |
| Subordinated | Bonds | 31.96 | 29.03 | 22.5 |

O'Kane notes: "The most important driver of the recovery rate is the position of the debt in the capital structure of the firm, i.e. senior debt recovers more on average than subordinated debt." He also observes that "the standard deviation of the recovery rate distributions is rather broad" and that the absolute priority rule (APR) "is not always obeyed."

> **Desk Reality: Why Recovery Matters More Than You Think**
>
> The ~20 percentage point gap between senior unsecured (35%) and subordinated (29%) mean recovery may seem small, but it drives large spread differences. When a name widens to distressed levels, the market reprices recovery aggressively—and senior vs sub recovery assumptions diverge further. A trader who assumes identical recovery across the capital structure will misprice hedges and underestimate P&L breaks.

### 52.7.2 The Senior-Subordinated Spread Relationship

O'Kane derives the theoretical relationship between senior and subordinated CDS spreads. For a single hazard rate $\lambda$ shared across the capital structure:

$$S_{\text{sub}} = \lambda(1 - R_{\text{sub}}) \quad \text{and} \quad S_{\text{sen}} = \lambda(1 - R_{\text{sen}})$$

Dividing:

$$\boxed{\frac{S_{\text{sub}}}{S_{\text{sen}}} = \frac{1 - R_{\text{sub}}}{1 - R_{\text{sen}}}}$$

O'Kane cautions: "In practice, this relationship is only a rough guide. Market technicals will often mean that it is not obeyed exactly. However, it does remind us that we should take care to price different levels of the capital structure consistent with their expected recovery rates."

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

O'Kane explains: "The equity payoff at time $T$ has the same payoff as a call option on the asset value of the firm with a strike price at the face value of the outstanding debt."

Using the Black-Scholes framework with asset volatility $\sigma_A$:

$$E(t) = A(t)\Phi(d_1) - Fe^{-r(T-t)}\Phi(d_2)$$

where $d_1, d_2$ follow the standard Black-Scholes definitions.

O'Kane derives the relationship between equity and asset volatility:

$$\sigma_E = \sigma_A \Phi(d_1) \frac{A(t)}{E(t)}$$

This equation "can then be used to calibrate the asset volatility parameter in the Merton model using the volatility of equity prices."

#### Limitations (O'Kane Section 3.4.1)

O'Kane lists key limitations:
- "The highly simplified capital structure is unrealistic"
- "The model only allows default at a single time $T$"
- "There is limited transparency regarding the value of the assets of a company"
- "The credit spread for firms for which $A(t) > F$ always tends to zero as $T - t \to 0$"

Despite these limitations, O'Kane notes: "The Merton model has been the inspiration behind one of the main correlation pricing models"—the Gaussian copula model uses a latent variable $A_i$ "in the spirit of Merton's structural model."

> **Desk Reality: How Equity Vol and CDS Spreads Connect**
>
> **Practitioner Note (Claude-extended):** While the Merton model has limited direct pricing application, it provides powerful trading intuition. When equity volatility spikes without a corresponding CDS move (or vice versa), traders look for convergence opportunities. The relationship $\sigma_E \propto \sigma_A \cdot A/E$ means that as a firm's equity value drops (leverage increases), equity vol should rise and CDS spreads should widen. A persistent divergence between implied equity vol and CDS spreads signals a potential relative value trade.
>
> Commercial implementations like Moody's KMV operationalize this by computing "distance to default" from equity prices and balance sheet data. Desk traders often watch CDS-equity implied correlations as a signal for capital structure trades.

### Strategy Card: D1 — Senior vs Subordinated CDS

##### Objective (Conceptual; No Forecasts)

Exploit deviations of the senior-sub spread ratio from its theoretical value, or express a view on relative recovery rates across the capital structure.

##### Instruments and Position Construction

- **Long sub protection, short senior protection** (betting sub widens relative to senior → the ratio increases toward theoretical fair value)
- **Or the reverse** (betting ratio compresses)
- Size for CS01-neutrality or for a specific spread-ratio view.

##### Exposure Decomposition

| Exposure (units) | Senior leg | Sub leg | Net |
|------------------|-----------|---------|-----|
| $\text{CS01}$ (USD/bp) | < 0 (short prot) | > 0 (long prot) | can match or intentional residual |
| JTD (USD) | negative | positive | recovery-dependent residual |
| Rec01 (USD/1%) | senior recovery sens | sub recovery sens | **key residual**: different recovery drives P&L |
| Default event | pays $(1 - R_{\text{sen}})$ | receives $(1 - R_{\text{sub}})$ | net gain if $R_{\text{sub}} < R_{\text{sen}}$ (typical) |

##### Hedge Set and Hedge Ratios

**CS01-neutral sizing:**

$$N_{\text{sen}} = N_{\text{sub}} \times \frac{\text{RPV01}_{\text{sub}}}{\text{RPV01}_{\text{sen}}}$$

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
- **Market technicals:** O'Kane notes the ratio is "only a rough guide." Supply-demand for sub protection (e.g., from structured product hedging) can push the ratio far from theoretical for extended periods.

##### Implementation Checklist

- Verify both CDS reference the same entity with aligned maturities.
- Use appropriate recovery assumptions for each seniority level.
- Check that the spread ratio is historically wide or tight relative to the theoretical level before trading.
- Run default scenario with recovery sweep ($R$ from 10% to 60%) on both legs.

---

### Strategy Card: D2 — Equity-CDS Relative Value (Merton-Inspired)

##### Objective (Conceptual; No Forecasts)

Exploit divergences between equity-implied credit risk and CDS-priced credit risk, using the Merton model's insight that both are functions of the same underlying asset value and volatility.

##### Instruments and Position Construction

- **CDS leg:** single-name CDS (long or short protection).
- **Equity leg:** equity options or equity shares used to hedge the equity-implied credit component.
- The trade exploits the link: when equity volatility implies a default probability different from what CDS spreads imply, one of them may be mispriced.

> **Desk Reality: LBO Protection Trades**
>
> **Practitioner Note (Claude-extended):** One classic capital structure trade involves anticipating leveraged buyouts (LBOs). When a private equity firm acquires a company using debt, the capital structure changes dramatically:
>
> - **Senior debt:** Often *worsens* slightly (more leverage in the structure)
> - **Subordinated debt:** Can *improve* if the sub moves from the bottom of a deep capital structure to a relatively more senior position in the restructured entity, or if the LBO price validates firm value
>
> Traders who anticipate LBOs may buy sub CDS protection (betting on sub tightening relative to senior) before the announcement. The key risk: if the LBO doesn't happen, sub CDS may widen. I'm not sure about the precise mechanics of sub recovery treatment in all LBO scenarios—the outcome depends on the specific deal structure and documentation.

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

O'Kane discusses Loan CDS (LCDS) in Section 5.7 as a variant that references loans rather than bonds. The key difference is recovery:

O'Kane states: "In general, loan CDS trade at a tighter (lower) credit default spread than standard CDS on the same name. This effect can be ascribed to the generally superior recovery rates of secured loans as opposed to the recovery rates of the bond and loan deliverable obligations of traditional CDS."

From Table 3.2, senior secured loans have a mean recovery of 68.5% vs 34.9% for senior unsecured bonds. Using the senior-sub spread relationship:

$$\frac{S_{\text{CDS}}}{S_{\text{LCDS}}} = \frac{1 - R_{\text{bond}}}{1 - R_{\text{loan}}} = \frac{1 - 0.35}{1 - 0.685} = \frac{0.65}{0.315} = 2.06$$

So standard CDS should trade at roughly 2× the LCDS spread for the same name, reflecting the lower loss-given-default on secured loans.

O'Kane also notes an important structural difference: "The European LCDS can only be triggered by the default of the reference obligation, which is a loan. The US LCDS can be triggered by all borrowings of the reference entity." Additionally, LCDS incorporates a **cancellation feature**—if the loan is refinanced, the LCDS cancels with payment of accrued coupon only.

**Index products:** The LCDX.NA index references 100 first-lien leveraged loans. The LevX index covers the European LCDS market with senior and subordinated sub-indices.

> **Practitioner Note (Claude-extended):** The LCDS-CDS basis is a tradeable relative value. When the ratio diverges from the recovery-implied level, it may signal a shift in loan recovery expectations or LCDS-specific supply-demand dynamics. The cancellation feature of LCDS makes its valuation more complex—O'Kane derives a two-curve model with both default probability $Q_D$ and cancellation probability $Q_C$.

---

## 52.9 Crisis Behavior of Credit Strategies

The strategies described in this chapter are designed for normal market conditions. In systemic stress, the assumptions underlying hedging relationships break down. O'Kane's emphasis on "perfect storm" risk—multiple adverse moves coinciding—is especially relevant here.

### 52.9.1 Basis Strategies in Crisis

O'Kane's framework reveals why basis trades are particularly vulnerable to systemic stress. Several of his market factors amplify simultaneously:

1. **Funding risk crystallizes:** O'Kane notes that "the CDS is unfunded, it effectively locks in a funding rate of Libor flat." During a crisis, the bond leg faces repo funding spikes while CDS funding is unaffected. The funding asymmetry—a theoretical basis driver in normal times—becomes the dominant P&L factor.

2. **Protection demand surges:** "It is much easier to go short a credit by buying protection in the CDS market than by shorting a cash bond." CDS spreads gap wider as market participants rush to hedge, while bond markets may freeze entirely.

3. **CDO unwind flows:** Normally, "CDO issuance is accompanied by a tightening in CDS spreads" (negative basis compression). When CDO structures unwind in stress, the reverse occurs—protection buying widens CDS spreads relative to bonds.

> **Practitioner Note (Claude-extended):** The 2008 crisis demonstrated that "hedged" basis positions could lose multiples of their expected annual carry in a single week. The mechanism: repo funding on the bond leg spiked (or became unavailable), CDS spreads gapped wider than bond spreads, and margin calls forced liquidation of both legs at the worst possible time. The lesson is not that basis trades are bad, but that they are *funding-contingent*—their risk profile changes qualitatively when funding conditions shift.
>
> I'm not sure about exact basis spread magnitudes during specific crisis episodes without market data sources. The qualitative pattern—funding-driven basis blowouts during systemic stress—is well-documented in practitioner literature.

### 52.9.2 Correlation and Tranche Trades in Crisis

O'Kane's Table 17.2 shows that equity tranches have ~18× leverage with negative systemic gamma. In a crisis:

- **Correlation spikes:** Defaults cluster, losses concentrate in equity and mezzanine tranches.
- **Equity tranche losses accelerate:** The negative systemic gamma means large spread moves *amplify* equity tranche losses beyond the linear (delta) approximation.
- **Senior tranches initially benefit, then face tail risk:** Senior tranches have positive systemic gamma (they gain on large moves initially), but if defaults actually cluster and breach attachment points, losses can be sudden and severe.

O'Kane notes that "even this approach is not practical" for exact correlation hedging—dealers use approximations that break down precisely when accuracy matters most.

### 52.9.3 Early Warning Indicators

> **Practitioner Note (Claude-extended):** While O'Kane's framework doesn't prescribe specific monitoring metrics, the risk decomposition suggests watching:
>
> | Indicator | What It Signals | Threshold for Concern |
> |-----------|----------------|-----------------------|
> | CDS-bond basis widening | Funding stress beginning | Basis exceeds 2× normal range |
> | Index basis (quoted vs intrinsic) widening | Index liquidity premium shifting | Basis exceeds historical 95th percentile |
> | Implied correlation rising | Default clustering fear | Correlation 01 P&L exceeds 5× daily carry |
> | Repo rate on specific names spiking | Funding for cash bonds under stress | Repo rate > Libor + 100bp for IG names |
> | Bid-offer widening in CDS | Liquidity deteriorating | Bid-offer > 2× normal |
>
> These are practitioner-level guidelines, not derived from the textbook sources. Specific threshold levels will depend on market regime and desk risk tolerance.

---

## 52.10 Execution and Position Management

> **Practitioner Note (Claude-extended):** The following execution guidance extends the mathematical hedge ratio framework in this chapter with desk-level practice. It is sourced from general credit trading knowledge, not from the textbook sources.

### 52.10.1 Position Sizing Considerations

For any strategy in this chapter, position size should reflect:

1. **Risk budget allocation:** Express maximum loss tolerance in JTD and CS01 terms, not just notional.
2. **Carry-to-risk ratio:** The expected carry should compensate for the worst-case scenario P&L. A rule of thumb: expected annual carry should be at least 2-3× the worst historical weekly P&L for the strategy.
3. **Liquidity-adjusted sizing:** Scale position size inversely with bid-offer cost as a fraction of expected P&L. If round-trip execution cost exceeds 20% of expected 3-month P&L, the position may be too large for the available liquidity.

### 52.10.2 Monitoring Triggers

| Trigger | Action |
|---------|--------|
| CS01 hedge drift > 15% of target | Rebalance hedge |
| Basis P&L exceeds 50% of 6-month carry (adverse) | Review position; prepare unwind plan |
| Recovery assumption changes by > 5% | Recalculate JTD and hedge ratios |
| Funding cost exceeds carry on basis trade | Escalate; consider reducing or unwinding |
| Correlation shock > 3% in one day | Stress-test tranche positions immediately |

### 52.10.3 Unwind Playbook

When conditions warrant reducing or exiting a position:

1. **Unwind the most liquid leg first** — typically CDS over bonds, index over single-name.
2. **Don't unwind CS01-hedged positions one leg at a time** — this creates naked exposure. Unwind in matched pairs where possible.
3. **In illiquid markets, consider partial unwinds** — reducing by 50% captures some value while maintaining optionality.
4. **Document unwind rationale** — was it risk limit breach, P&L stop, or fundamental view change? This matters for post-trade analysis.

---

## 52.11 Mathematical Foundations

### 52.11.1 CS01 for a Standard CDS from the MTM Identity

**Source-backed starting point:**

$$V(t,T) = (S(t,T) - S_0) \cdot \text{RPV01}(t,T) \quad \text{(per unit notional)}.$$

**Differentiate w.r.t. market spread $S$:**

$$\frac{\partial V}{\partial S} = \text{RPV01}(t,T).$$

**Therefore,**

$$\boxed{\text{CS01} = \frac{\partial PV}{\partial S} \times 10^{-4} = N \cdot \text{RPV01}(t,T) \cdot 10^{-4}}$$

**Unit check:**

$\text{RPV01}$ has units "years" (PV of 1 bp/yr premium). Multiply by $10^{-4}$ (per bp) and notional $N$ (USD) $\Rightarrow$ USD/bp.

---

### 52.11.2 CS01-Based Hedge Ratio Derivation

Let target position have $\text{CS01}_T$ and hedge instrument have $\text{CS01}_H$, both sign-inclusive (USD/bp).

We want:

$$\text{CS01}_{\text{net}} = \text{CS01}_T + N_H \cdot \text{CS01}_H^{(\$1)} = 0.$$

Solve:

$$\boxed{N_H = -\frac{\text{CS01}_T}{\text{CS01}_H^{(\$1)}}}$$

If CS01s are already in USD/bp for stated notionals:

$$N_H = -\frac{\text{CS01}_T}{\text{CS01}_H} \times N_H^{\text{current}}.$$

---

### 52.11.3 JTD (VOD) Mapping for CDS

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

### 52.11.4 P&L Explain Template (First-Order + Event Terms)

For a portfolio, approximate P&L over horizon as:

$$\boxed{\Delta PV \approx \sum_i (\text{CS01}_i \cdot \Delta S_i) + (\text{Rec01} \cdot \Delta R) + \text{JTD}_{\text{events}} + \text{BasisTerm} + \text{Residual}}$$

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

**If you need an exact closed-form carry formula:** I'm not sure, because it depends on the contract quoting regime (par spread vs fixed coupon + upfront), day count, and whether you use clean vs full price. What is needed: the desk's carry definition and the index rulebook conventions.

---

## 52.12 Worked Examples

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

$$\text{Funding cost} = 1{,}010{,}000 \times 0.0125 \times 0.25 = \$3{,}156.25$$

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

**Lesson**: O'Kane's point about "funding risk" is concrete—the unfunded CDS "effectively locks in a funding rate of Libor flat" while the bond leg is exposed to repo market dynamics.

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

O'Kane identifies two primary sources of P&L impact during the roll: (1) composition changes affecting perceived credit quality, and (2) maturity extension on an upward-sloping curve. This example illustrates both effects.

#### Setup

- Notional $N = \$10\text{mm}$, coupon $C = 100$ bp.
- Old series: $S_{\text{old}} = 115$ bp, $\text{RPV01}_{\text{old}} = 4.1$ (remaining maturity ~4.5Y)
- New series: $S_{\text{new}} = 120$ bp, $\text{RPV01}_{\text{new}} = 4.6$ (full 5Y maturity)
- Position: long protection.

#### Understanding the Spread Difference

The new series trades 5 bp wider than the old series. Per O'Kane's framework, we can decompose this:

**Maturity extension effect**: The new index has 6 months longer maturity. With an upward-sloping credit curve (typical), this adds spread. Suppose the curve slope is ~10 bp per year at the 5Y point:
$$\Delta S_{\text{maturity}} \approx 10 \times 0.5 = +5 \text{ bp}$$

**Composition effect**: In this example, composition changes are neutral—the downgraded names removed are offset by wider-spread liquidity replacements:
$$\Delta S_{\text{composition}} \approx 0 \text{ bp}$$

**Total**: $\Delta S = 5 + 0 = 5$ bp (matches observed difference).

#### MTM Values (Pre-Upfront)

$$V_{\text{old}} = N \cdot 4.1 \cdot (115 - 100) \cdot 10^{-4} = 10{,}000{,}000 \cdot 4.1 \cdot 15 \cdot 10^{-4} = \$61{,}500.$$

$$V_{\text{new}} = N \cdot 4.6 \cdot (120 - 100) \cdot 10^{-4} = \$92{,}000.$$

#### Cash to Switch (Close Old, Open New)

- Close old (sell your long protection): receive $61,500.
- Open new: pay $92,000.
- **Net cash outflow:**

$$92{,}000 - 61{,}500 = \$30{,}500.$$

#### CS01-Adjusted Notional for Spread-Neutral Switch

If the goal is to maintain the same CS01 exposure after rolling:

$$\text{CS01}_{\text{old}} = 10{,}000{,}000 \times 4.1 \times 10^{-4} = \$4{,}100/\text{bp}$$
$$\text{CS01}_{\text{new per \$10mm}} = 10{,}000{,}000 \times 4.6 \times 10^{-4} = \$4{,}600/\text{bp}$$

To maintain $4,100/bp CS01 on the new series:
$$N_{\text{new}} = 10{,}000{,}000 \times \frac{4{,}100}{4{,}600} = \$8{,}913{,}043$$

This means **reducing notional by ~11%** when rolling to maintain spread-neutral exposure.

#### Execution Cost (Toy)

Suppose bid/offer equivalent to 0.5 bp spread on each leg.

- Approx PV cost per leg $\approx \text{CS01} \times 0.5$.
- Use $\text{CS01}_{\text{new}} \approx 4{,}600$ USD/bp
- Cost per leg $\approx 4{,}600 \times 0.5 = \$2{,}300$
- Two legs $\Rightarrow$ total $\approx \$4{,}600$

#### Total "All-In" Cash Impact

$$30{,}500 + 4{,}600 = \$35{,}100.$$

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

- Old series: $S_{\text{old}} = 115$ bp, $V_{\text{old}} = \$61{,}500$
- New series: $S_{\text{new}} = 111.6$ bp

$$V_{\text{new}} = 10{,}000{,}000 \cdot 4.6 \cdot (111.6 - 100) \cdot 10^{-4} = \$53{,}360$$

**Net cash inflow** (favorable roll):
$$61{,}500 - 53{,}360 = \$8{,}140 \text{ received}$$

#### Key Lesson

O'Kane notes that "this does not mean that the new index will be issued at a tighter spread than the current market spread of the old index since liquidity requirements may result in some names being replaced with wider spread names." The composition effect can work either way, and traders must analyze both components to understand roll P&L.

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
| **CS01 (Spread DV01)** | PV change per +1 bp change in quoted spread: $\text{CS01} = \frac{\partial PV}{\partial S} \times 10^{-4}$ | The primary linear risk measure for credit positions; must specify *which* spread is bumped |
| **JTD / VOD** | Jump-to-default = $PV_{\text{post}} - PV_{\text{pre}}$; VOD = $(1-R-\text{Accrued}) - (S-S_0) \cdot \text{RPV01}$ | Captures discontinuous default risk that CS01 hedges do not neutralize |
| **Bond–CDS Basis** | CDS spread − Bond Libor spread (typically asset swap spread) | Twelve drivers (6 fundamental + 6 market) per O'Kane; a persistent tradeable phenomenon |
| **Loss-on-Default Mismatch** | Bond loss = $P - R$; CDS loss = $1 - R$; difference when bond price ≠ 100 | A basis driver that causes hedge ratios to differ from par assumptions |
| **Intrinsic vs Quoted Basis** | Gap between index's quoted spread and RPV01-weighted average of constituent spreads | Driven by restructuring clause, liquidity, and "index leads the market" effects |
| **Portfolio Swap Adjustment** | Adjusting constituent curves so intrinsic index value = quoted value | Required for tranche pricing; O'Kane notes it is "somewhat arbitrary" |
| **Index Roll P&L** | P&L from switching series due to (1) composition changes and (2) maturity extension | The two effects can offset or compound; crucial for roll timing decisions |
| **Systemic Delta** | Tranche sensitivity to parallel portfolio spread move | Equity tranches are highly leveraged (~18×); senior tranches are deleveraged (~0.4×) |
| **Systemic Gamma** | Second-order tranche sensitivity to parallel spread moves | Negative for equity (large moves hurt), positive for senior (large moves help) |
| **Corr01** | PV change per +1% correlation increase | *Opposite* sign for equity vs senior: negative for equity, positive for senior (long protection) |
| **Tranche Loss Fraction** | $L_{[A,B]} = \frac{1}{B-A}(\max(L-A,0) - \max(L-B,0))$ | Translates portfolio loss to tranche-specific loss; highly nonlinear for equity |
| **Gamma vs Carry Trade-off** | Positive gamma ↔ negative carry; negative gamma ↔ positive carry | O'Kane: "a positive gamma position is associated with a negative carry" |
| **Strategy Card Discipline** | Exposures + Hedges + Failure Modes | Every strategy must enumerate residual risks and scenarios where hedges break |
| **Idiosyncratic Delta** | Tranche sensitivity to single-name spread move | Differs from systemic delta; reveals name-specific risk concentration |
| **Delivery Option** | CDS protection buyer's right to choose cheapest deliverable bond | Widens CDS spreads relative to bonds; source of basis |
| **Funding Risk** | Exposure to repo/financing rate changes on funded instruments | CDS is unfunded; bonds face funding risk that can dominate carry |
| **Negative Basis** | CDS spread < bond asset swap spread; synthetic protection cheaper than cash | Exploitable if funding cost is low enough; destroyed by funding stress |
| **Positive Basis** | CDS spread > bond spread; protection demand exceeds cash supply | Tends to emerge during distress; squeeze dynamics |
| **Senior-Sub Spread Ratio** | $S_{\text{sub}}/S_{\text{sen}} = (1-R_{\text{sub}})/(1-R_{\text{sen}})$ under equal hazard rates | Foundation for capital structure arbitrage; recovery uncertainty is key risk |
| **Merton Model (Credit)** | Equity = call on firm assets; debt = cash − put on assets | Links equity and credit markets; basis for equity-CDS RV trades |
| **LCDS** | Loan CDS referencing senior secured loans with higher recovery | Trades at ~50% of standard CDS spread due to recovery difference |
| **CDS Curve Inversion** | Short-dated spreads exceed long-dated (common in HY distressed) | Signals near-term default risk; curve trades carry JTD mismatch |
| **Corr01 Convexity** | Corr01 itself changes with correlation level (second-order) | "Neutral" positions can have large global P&L; local ≠ global |

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
11. **Capital structure arbitrage** exploits the senior-sub spread relationship ($S_{\text{sub}}/S_{\text{sen}} = (1-R_{\text{sub}})/(1-R_{\text{sen}})$), but recovery uncertainty is the dominant risk.
12. **Tranche risk is multi-dimensional**: systemic delta, systemic gamma, idiosyncratic delta, idiosyncratic gamma, Corr01, carry, and theta all matter.
13. **Equity and senior tranches have opposite correlation exposures**: Corr01 is negative for equity, positive for senior (for long protection).
14. **Positive gamma positions typically have negative carry** (O'Kane)—there is a cost to owning convexity.
15. **Crisis behavior differs by strategy**: funding stress destroys basis trades; correlation spikes destroy tranche RV; curve inversion signals imminent default.
16. **Always run a scenario suite:** parallel, dispersion, default event, roll/series basis, correlation/tail shocks, and funding stress.

### Cheat Sheet

**Strategy card:** Objective → Instruments → Exposures table → Hedge ratios → Scenario suite → Failure modes → Checks.

**Key formulas:**

- CDS MTM: $V = (S - S_0) \cdot \text{RPV01}$.
- CS01 hedge: $N_H = -\text{CS01}_T / \text{CS01}_H$.
- VOD/JTD (CDS): $\text{VOD} = (1 - R - \text{Accrued}) - (S - S_0) \cdot \text{RPV01}$.
- Tranche loss: $L_{[A,B]} = \frac{1}{B - A} \left( \max(L - A, 0) - \max(L - B, 0) \right)$.
- Senior-sub spread ratio: $S_{\text{sub}}/S_{\text{sen}} = (1 - R_{\text{sub}})/(1 - R_{\text{sen}})$.
- Curve trade CS01-neutral sizing: $N_{\text{short}} = N_{\text{long}} \times \text{RPV01}_{\text{long}} / \text{RPV01}_{\text{short}}$.

---

### Example 16: Senior vs Subordinated CDS — Recovery-Driven Spread Relationship

**Setup:** Consider a BBB-rated issuer with both senior unsecured and subordinated CDS traded. Market recovery assumptions: $R_{\text{sen}} = 40\%$, $R_{\text{sub}} = 20\%$. Senior 5Y CDS trades at $S_{\text{sen}} = 120$ bp.

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

$$N_{\text{sen}} = N_{\text{sub}} \times \frac{\text{RPV01}_{\text{sub}}}{\text{RPV01}_{\text{sen}}}$$

If $\text{RPV01}_{\text{sen}} = 4.1$ and $\text{RPV01}_{\text{sub}} = 3.8$ (lower survival probability compresses RPV01), then for $5mm sub notional:

$$N_{\text{sen}} = 5 \times \frac{3.8}{4.1} = 4.63\text{mm}$$

**Failure mode:** If actual sub recovery at default is 5% (not 20%), your "cheap sub protection" was actually fair and you lose on the senior leg. Recovery uncertainty is the dominant risk.

---

### Example 17: HY Curve Steepener — CS01-Neutral Sizing

**Setup:** A high-yield issuer trades with an inverted CDS curve: 3Y at 450 bp, 5Y at 380 bp. You believe the inversion is excessive (the issuer will survive the near-term liquidity crunch) and want to trade the curve flattening.

**Trade:** Buy 5Y protection (long) + Sell 3Y protection (short). This is a "bull flattener" — you profit if the curve flattens (3Y tightens more than 5Y, or 5Y widens less than 3Y).

**CS01-neutral sizing:** Per O'Kane's framework, match the CS01 so parallel moves cancel:

Given: $\text{RPV01}_{3Y} = 2.5$, $\text{RPV01}_{5Y} = 3.6$ (both per $1mm notional).

$$N_{3Y} = N_{5Y} \times \frac{\text{RPV01}_{5Y}}{\text{RPV01}_{3Y}} = 10\text{mm} \times \frac{3.6}{2.5} = 14.4\text{mm}$$

**P&L scenarios (per $10mm 5Y notional, $14.4mm 3Y notional):**

| Scenario | 3Y move | 5Y move | 3Y P&L | 5Y P&L | Net P&L |
|----------|---------|---------|--------|--------|---------|
| Parallel +50bp | +50 | +50 | $+50 \times 14.4 \times 2.5 \times 10^{-4} = +\$18{,}000$ | $-50 \times 10 \times 3.6 \times 10^{-4} = -\$18{,}000$ | **$0** |
| Curve flattens | -100 | -30 | $-100 \times 14.4 \times 2.5 \times 10^{-4} = -\$36{,}000$ | $+30 \times 10 \times 3.6 \times 10^{-4} = +\$10{,}800$ | **-$25,200** |
| Curve normalizes (3Y tightens) | -150 | +20 | $-\$54{,}000$ | $-\$7{,}200$ | **-$61,200** |
| Default | — | — | Lose on 3Y (short prot) | Gain on 5Y (long prot) | Net depends on timing |

Wait — check the signs. Short protection on 3Y means CS01 < 0; when 3Y spreads *tighten* (move negative), short protection *gains*:

| Scenario | 3Y move | 5Y move | 3Y P&L (short prot) | 5Y P&L (long prot) | Net P&L |
|----------|---------|---------|---------------------|---------------------|---------|
| Curve flattens (3Y tightens -100, 5Y flat) | -100 | 0 | $+\$36{,}000$ | $\$0$ | **+$36,000** |
| Curve normalizes (3Y -150, 5Y -30) | -150 | -30 | $+\$54{,}000$ | $-\$10{,}800$ | **+$43,200** |
| Default | — | — | ~$-(1-R) \times 14.4\text{mm}$ | ~$+(1-R) \times 10\text{mm}$ | **-$2.6mm** net loss |

**Key risk:** JTD is *not* hedged. The 3Y short protection notional exceeds 5Y long protection notional, so default produces a net loss of approximately $(1-R)(14.4 - 10) = 0.65 \times 4.4 = \$2.86\text{mm}$.

> **Desk Reality:** This is why HY curve trades are dangerous — the CS01-neutral sizing creates a JTD mismatch. Some desks cap the notional ratio or add a JTD overlay.

---

### Example 18: Negative Basis P&L Under Funding Stress

**Setup:** You hold a negative basis package: long a corporate bond at Z-spread of 180 bp, long protection via 5Y CDS at 150 bp. The basis is $-30$ bp (CDS tighter than bond spread). You fund the bond position at LIBOR + 50 bp.

**Carry calculation (annualized, per $10mm notional):**

- Bond coupon received (spread component): $+180$ bp $= +\$180{,}000$/yr
- CDS premium paid: $-150$ bp $= -\$150{,}000$/yr
- Funding cost (spread over LIBOR): $-50$ bp $= -\$50{,}000$/yr
- **Net carry: $-20$ bp $= -\$20{,}000$/yr**

The "free lunch" of $-30$ bp basis is consumed by $-50$ bp funding cost, leaving negative net carry.

**Funding stress scenario:** Suppose your funding cost widens to LIBOR + 120 bp (credit crunch):

- New net carry: $180 - 150 - 120 = -90$ bp $= -\$90{,}000$/yr
- The position now *bleeds* $\$90{,}000$/yr while waiting for convergence

**MTM impact:** If market expects elevated funding for 2 years, the MTM loss is approximately:

$$\Delta PV \approx -90 \text{ bp} \times \text{RPV01}_{\text{bond}} \times \text{Notional} \approx -90 \times 4.0 \times 10^{-4} \times 10{,}000{,}000 = -\$360{,}000$$

> **Practitioner Note (Claude-extended):** This is why bank-affiliated desks with cheap balance sheet funding (LIBOR + 20bp) can run negative basis books profitably while hedge funds at LIBOR + 80bp cannot. The "same" trade has different economics for different institutions. The 2008 crisis demonstrated this violently: funding costs spiked from ~30bp to 200+bp, annihilating negative basis P&L for leveraged investors even though the basis eventually converged.

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

Equity loss: $5\text{mm} \times 37.5\% = -\$1.875\text{mm}$ (stock jumps from $40 to $55)

CDS gain: $(350 - 90) \times 4.3 \times 10^{-4} \times 5{,}000{,}000 = +\$559{,}000$

**Net: $-\$1.316\text{mm}$** — the CDS gain doesn't cover the equity loss because CDS leverage is lower than equity sensitivity.

> **Desk Reality:** LBO trades are notoriously difficult to size correctly. The equity move is discontinuous (tender premium), while CDS moves are large but capped. Professional desks often use options on equity (puts) rather than short stock to limit downside, and overweight the CDS leg. The key insight from the Merton model is that equity is effectively a call option—its sensitivity to leverage events can dwarf CDS sensitivity.

I'm not sure about exact post-LBO CDS spread levels — these are illustrative. Actual levels depend on leverage ratios, sector, and market conditions.

---

### Example 20: Correlation Shock on Equity-Senior Combo

**Setup:** You hold a correlation trade: short $10mm equity tranche (0-3%) protection + long $30mm senior tranche (7-10%) protection, sized to be Corr01-neutral at current correlation of 25%.

From O'Kane Table 17.2 (illustrative sensitivities per $10mm):
- Equity tranche (short prot): Corr01 = $+\$6{,}000$/1% (positive because short protection *gains* when correlation rises)
- Senior tranche (long prot): Corr01 = $+\$2{,}000$/1% (positive because long protection gains when correlation rises)

Net Corr01: $-6{,}000 + 3 \times 2{,}000 = \$0$ (by construction)

**Scenario: Correlation jumps from 25% to 40% (15% shock, e.g., during a crisis):**

Linear estimate: Net P&L $= 0 \times 15 = \$0$

But Corr01 is *not* constant — it changes with correlation level. At $\rho = 40\%$:
- Equity Corr01 shrinks to $+\$3{,}500$/1% (the tranche is now deeper in the money)
- Senior Corr01 increases to $+\$3{,}200$/1%

The *convexity* of the correlation exposure means your hedge ratio drifted during the move. Approximate P&L using average Corr01 over the path:

- Equity leg: $-\frac{6{,}000 + 3{,}500}{2} \times 15 = -\$71{,}250$
- Senior leg: $+3 \times \frac{2{,}000 + 3{,}200}{2} \times 15 = +\$117{,}000$
- **Net: $+\$45{,}750$**

The "Corr01-neutral" position actually *makes* money on a large correlation shock because the senior leg's convexity dominates.

> **Practitioner Note (Claude-extended):** This is why tranche traders distinguish between "local" hedge ratios (valid for small moves) and "global" risk profiles (how P&L behaves over large moves). A position that is locally neutral can have significant global exposure. O'Kane's gamma framework (Table 17.5) captures this: the sign and magnitude of gamma determine whether large moves help or hurt.

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

31. **Q:** What are O'Kane's six fundamental basis factors? **A:** Funding, delivery option, technical default, loss-on-default mismatch, premium accrued at default, CDS spreads non-negative.

32. **Q:** What are O'Kane's six market basis factors? **A:** Relative liquidity, CDO technical short, new issuance/loans, convertible issuance, demand for protection, funding risk.

33. **Q:** Why does CDO issuance affect CDS spreads? **A:** Dealers hedge by selling protection on portfolio credits, which tightens CDS spreads and reduces the basis.

34. **Q:** What does O'Kane mean by "the index leads the market"? **A:** The CDS index is more liquid and is the preferred instrument for expressing market-wide credit views, especially for hedging.

35. **Q:** What is systemic gamma for an equity tranche? **A:** Large and negative (for long protection)—large parallel spread moves decrease PV.

36. **Q:** What is systemic gamma for a senior tranche? **A:** Positive (for long protection)—large parallel spread moves increase PV.

37. **Q:** What carry profile is typical for a positive gamma position? **A:** Negative carry—O'Kane notes "a positive gamma position is associated with a negative carry."

38. **Q:** How many times leverage does a 0-3% equity tranche typically have? **A:** About 18× according to O'Kane's Table 17.2.

39. **Q:** What is the portfolio swap adjustment? **A:** Adjusting individual CDS curves so that intrinsic index value equals quoted market value.

40. **Q:** Why is base correlation interpolation risky? **A:** O'Kane shows linear interpolation is not guaranteed to be arbitrage-free for non-standard tranches.

41. **Q:** What are O'Kane's two primary sources of P&L during index roll? **A:** (1) Composition changes affecting perceived credit quality, (2) Maturity extension on an upward-sloping curve.

42. **Q:** Why might a new index series trade tighter than the old series? **A:** Composition improvement (removing distressed names) can dominate the maturity extension effect.

43. **Q:** How do you calculate CS01-adjusted notional for spread-neutral roll? **A:** $N_{\text{new}} = N_{\text{old}} \times \frac{\text{CS01}_{\text{old}}}{\text{CS01}_{\text{new}}}$.

44. **Q:** What's the risk of rolling "flat notional" between index series? **A:** You increase spread exposure because RPV01 increases with longer maturity (new series has higher CS01 per dollar).

45. **Q:** What is a "negative basis" in bond-CDS terms? **A:** When CDS spread < bond asset swap spread, i.e., synthetic protection is cheaper than cash credit risk.

46. **Q:** Why does CLO demand compress negative basis? **A:** CLO vehicles buy bonds (pushing spreads tighter) and buy CDS protection (pushing CDS spreads wider), narrowing the gap.

47. **Q:** What is the "positive basis squeeze"? **A:** When CDS spreads widen above bond spreads—often during distress when protection demand surges and bond liquidity dries up.

48. **Q:** What is the senior-sub CDS spread ratio under equal hazard rates? **A:** $S_{\text{sub}} / S_{\text{sen}} = (1 - R_{\text{sub}}) / (1 - R_{\text{sen}})$ — driven purely by recovery difference.

49. **Q:** What does the Merton model say about equity and debt? **A:** Equity is a call option on firm assets; debt is equivalent to risk-free cash minus a put on firm assets.

50. **Q:** Why are HY CDS curves often inverted? **A:** Near-term default probability is high; if the issuer survives, forward default rates decline. Survival selection effect.

51. **Q:** What is LCDS? **A:** Loan CDS—credit protection on senior secured loans, which trade tighter than standard CDS due to higher recovery (~65-70% vs ~35-40%).

52. **Q:** What is the theoretical LCDS-to-CDS spread ratio? **A:** $S_{\text{LCDS}} / S_{\text{CDS}} = (1 - R_{\text{loan}}) / (1 - R_{\text{bond}})$ — typically around 0.5× for investment-grade names.

53. **Q:** What happens to negative basis trades in a funding crisis? **A:** Funding costs spike, eroding or reversing carry. Leveraged holders face margin calls and forced unwinds, driving basis wider.

54. **Q:** What happens to base correlation during a crisis? **A:** It typically spikes (e.g., from 20-30% to 50-70%), driven by systemic contagion expectations.

55. **Q:** Name three early warning indicators for credit strategy stress. **A:** (1) Funding market stress (repo/CP spreads), (2) Bid-ask widening in CDS, (3) CDS-cash basis divergence.

56. **Q:** Why does CS01-neutral sizing create a JTD mismatch in curve trades? **A:** Matching spread risk requires unequal notionals (short-dated leg is larger), so default produces a net loss on the excess notional.

57. **Q:** What is the "local vs global" hedge ratio distinction in tranches? **A:** Local ratios (Corr01, PV01) are valid for small moves; global P&L profiles can diverge for large moves due to nonlinearity/convexity.

58. **Q:** In an LBO scenario, why does equity move more than CDS? **A:** Equity is a call option on assets (high delta to leverage events); CDS spread increases are bounded by the LGD cap.

---

## 52.15 Mini Problem Set

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

**21.** A senior CDS trades at 100 bp with $R_{\text{sen}} = 40\%$. A subordinated CDS trades at 175 bp with $R_{\text{sub}} = 20\%$. (a) Compute the theoretical sub spread using the O'Kane senior-sub relationship. (b) Is sub CDS rich or cheap relative to the model? (c) What recovery assumption would rationalize the market sub spread?

> **Sketch:** (a) $S_{\text{sub}}^{\text{fair}} = 100 \times (1-0.20)/(1-0.40) = 133.3$ bp. (b) Market sub at 175 is wide of model by ~42 bp — sub protection looks cheap (or market prices lower sub recovery). (c) Solve $175 = 100 \times (1-R_{\text{sub}})/(0.60)$ → $R_{\text{sub}} = 1 - 1.05 = -5\%$, which is impossible, so the gap must reflect liquidity premium or correlation with default intensity, not pure recovery.

---

**22.** You want to construct a CS01-neutral 3Y-5Y HY curve steepener. 5Y notional is $10mm with $\text{RPV01}_{5Y} = 3.4$. 3Y has $\text{RPV01}_{3Y} = 2.2$. (a) Compute 3Y notional. (b) Compute the JTD mismatch at recovery $R = 30\%$. (c) Should you add a JTD overlay? How?

> **Sketch:** (a) $N_{3Y} = 10 \times 3.4/2.2 = 15.45\text{mm}$. (b) JTD mismatch: $(1-R)(N_{3Y} - N_{5Y}) = 0.70 \times 5.45 = \$3.82\text{mm}$ net loss on default. (c) Yes — buy additional 5Y protection for the JTD gap: $\Delta N_{5Y} = 5.45/(1) = 5.45\text{mm}$ (adjusting for the fact that both legs pay $(1-R)$ at default, so you need $N_{3Y} - N_{5Y} = 5.45\text{mm}$ of additional 5Y protection). This breaks CS01 neutrality — trade-off between curve and default risk.

---

**23.** A negative basis package has: bond spread 200 bp, CDS spread 160 bp (basis = $-40$ bp), funding cost = LIBOR + 60 bp. (a) Compute annual carry per $10mm. (b) At what funding cost does the trade break even? (c) If funding widens to LIBOR + 150 bp during a crisis and stays there for 6 months, what is the carry loss?

> **Sketch:** (a) Carry $= (200 - 160 - 60) \times 10^{-4} \times 10{,}000{,}000 = -\$20{,}000$/yr (negative carry). (b) Break-even: $200 - 160 - x = 0 \Rightarrow x = 40$ bp. (c) At LIBOR + 150: carry $= (200-160-150) = -110$ bp $= -\$110{,}000$/yr. Over 6 months: $-\$55{,}000$.

---

**24.** An LBO increases a firm's leverage from 2× to 5× EBITDA. Pre-LBO: senior CDS = 80 bp, equity price = $45. Post-LBO: senior CDS = 280 bp, equity tender at $60. You are short $3mm equity and long $8mm CDS protection with $\text{RPV01} = 4.0$. (a) Compute equity leg P&L. (b) Compute CDS leg P&L. (c) Was the trade profitable? (d) What sizing change would improve the outcome?

> **Sketch:** (a) Equity loss: $3\text{mm} \times (60-45)/45 = 3\text{mm} \times 33.3\% = -\$1{,}000{,}000$. (b) CDS gain: $(280-80) \times 4.0 \times 10^{-4} \times 8{,}000{,}000 = +\$640{,}000$. (c) Net: $-\$360{,}000$ — unprofitable. (d) Increase CDS notional relative to equity — the CDS leg needs more weight because equity jumps are larger (percentage) than CDS spread moves. Alternatively, use equity puts to cap equity loss.

---

**25.** You hold a Corr01-neutral equity-senior tranche combo. (a) Why can this position still lose money on a large correlation shock? (b) What additional risk measure would you monitor? (c) Describe a scenario where the "neutral" position produces a large loss.

> **Sketch:** (a) Corr01 is a *local* (first-order) measure. For large moves, the correlation sensitivity itself changes — this is "correlation convexity" or second-order correlation risk. (b) Monitor the *change* in Corr01 as correlation moves (i.e., second-order sensitivity), and run full revaluation under ±10-20% correlation shocks. (c) A rapid decorrelation event (e.g., one sector defaults while others are fine) can cause equity tranche to move violently while senior barely responds — the local hedge ratio is stale and losses accumulate before rebalancing.

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

| Fact | Source |
|------|--------|
| CDS MTM identity, RPV01, CS01 definitions | O'Kane Ch 6–8 |
| CDS basis definition: CDS spread - Bond Libor spread | O'Kane Ch 5, Section 5.6 |
| Six fundamental basis factors (funding, delivery option, technical default, loss-on-default, premium accrued, CDS non-negative) | O'Kane Ch 5, Section 5.6.1 |
| Six market basis factors (relative liquidity, CDO technical short, new issuance, convertible issuance, demand for protection, funding risk) | O'Kane Ch 5, Section 5.6.2 |
| Delivery option example ($6 profit from cheapest-to-deliver) | O'Kane Ch 5, Section 5.4.3 |
| Index upfront mechanics, roll mechanics | O'Kane Ch 10, Section 10.2 |
| Intrinsic vs quoted index basis | O'Kane Ch 10, Sections 10.3, 10.5 |
| Index basis examples (Table 10.6: CDX IG, HY, iTraxx) | O'Kane Ch 10, Section 10.5.2 |
| Index basis drivers (restructuring clause, liquidity, index leads market) | O'Kane Ch 10, Section 10.5.2 |
| Portfolio swap adjustment mechanics | O'Kane Ch 10, Section 10.6 |
| Roll P&L drivers (composition changes, maturity extension) | O'Kane Ch 10, Section 10.2.2 |
| Tranche loss function, correlation sensitivity, base correlation | O'Kane Ch 11–14, 17, 20 |
| Systemic delta, systemic gamma, idiosyncratic gamma (Table 17.2) | O'Kane Ch 17, Section 17.2 |
| Carry and theta definitions for tranches (daily values) | O'Kane Ch 17, Section 17.2.6 |
| Gamma trading framework (Table 17.5) | O'Kane Ch 17, Section 17.5.3 |
| Correlation hedging via completing capital structure | O'Kane Ch 17, Section 17.5.4 |
| Base correlation framework and interpolation | O'Kane Ch 20 |
| VOD/JTD framework | O'Kane Ch 8 |
| Recovery rates by seniority (Table 3.2): senior unsecured 36.4%, subordinated 31.4%, senior secured bonds 54.5%, senior secured loans 68.5% | O'Kane Ch 3, Table 3.2 |
| Senior-sub spread ratio: $S_{\text{sub}}/S_{\text{sen}} = (1-R_{\text{sub}})/(1-R_{\text{sen}})$ | O'Kane Ch 3, Section 3.3 |
| Merton model: equity as call option on firm assets, debt as cash minus put | O'Kane Ch 3, Section 3.4 |
| Three CDS curve shapes (steep upward, moderate upward, steeply inverted) | O'Kane Ch 7, Table 7.2 |
| Arbitrage bounds on CDS spreads from inverted curves | O'Kane Ch 7, Table 7.3 |
| Forward CDS curves implied from CDS term structure | O'Kane Ch 9, Table 9.1 |
| LCDS mechanics: references senior secured loans, higher recovery | O'Kane Ch 5, Section 5.7 |
| LCDS spread ratio to standard CDS via recovery differential | O'Kane Ch 6, Section 6.9 |
| Negative basis: six fundamental factors including funding | O'Kane Ch 5, Section 5.6.1 |
| Twelve basis drivers (6 fundamental + 6 market) | O'Kane Ch 5, Sections 5.6.1-5.6.2 |

### (B) Reasoned Inference — Note Derivation Logic

- CS01 hedge ratio derivation: algebraic from CS01 definition and O'Kane's MTM identity
- P&L explain template: summation of first-order sensitivities plus events (consistent with O'Kane's risk framework)
- Tranche loss fraction computation: direct application of tranche loss formula from O'Kane Ch 12
- Strategy decision tree: synthesized from O'Kane's discussion of when each risk measure dominates
- Equity-senior correlation trade sizing: direct application of Corr01 values from Table 17.2
- Funding shock P&L example: application of O'Kane's funding risk discussion to concrete scenario
- Roll P&L decomposition (Example 8): direct application of O'Kane's two roll drivers (composition + maturity extension)
- CS01-adjusted notional for roll: algebraic derivation from CS01 matching to maintain spread-neutral exposure
- Example 8B composition effect: calculated using O'Kane's framework applied to hypothetical 3-name replacement scenario
- Senior-sub worked example (Example 16): direct application of O'Kane's spread ratio formula with hypothetical recovery values
- Curve steepener sizing (Example 17): algebraic from RPV01 matching; JTD mismatch derived from notional imbalance
- Negative basis carry decomposition (Example 18): application of O'Kane's funding cost discussion to concrete P&L
- LBO scenario (Example 19): application of Merton model logic from O'Kane Section 3.4 to a hypothetical event
- Correlation convexity (Example 20): extension of O'Kane's gamma framework (Table 17.5) to show local vs global hedge behavior

### (B2) Claude-Extended Content

| Content | Context |
|---------|---------|
| CLO demand as driver of negative basis compression | Extended from general structured credit knowledge; O'Kane discusses CDO issuance effects but not CLO-specific mechanics |
| Bank funding arbitrage in negative basis (LIBOR + 20bp vs + 80bp) | Extended from O'Kane's funding discussion; specific spread levels are illustrative |
| 2008 crisis impact on negative basis trades | General market knowledge; O'Kane discusses funding stress conceptually but not specific crisis data |
| LBO trade construction and equity-CDS sizing challenges | Extended from Merton model discussion; specific LBO mechanics from general practitioner knowledge |
| Early warning indicators table (funding, bid-ask, basis divergence) | Synthesized from O'Kane's risk framework; specific indicators from practitioner experience |
| Position sizing and monitoring triggers | General risk management practice; not specifically from O'Kane |
| Unwind playbook and liquidity hierarchy | General desk practice; O'Kane discusses liquidity conceptually |

### (C) Speculation — Flag Uncertainties

- Exact closed-form carry formula for indices: depends on desk conventions and rulebook; flagged as "I'm not sure"
- Portfolio swap adjustment implementation details: O'Kane describes as "somewhat arbitrary" and notes "we choose to adjust the individual issuer curves rather than the index swap since the index swap is substantially more liquid"; exact algorithms vary by desk
- Historical basis behavior during 2008: O'Kane mentions funding and liquidity effects but does not provide specific numerical examples from that period; any such examples would require additional sources
- Correlation hedging effectiveness in practice: O'Kane notes "even this approach is not practical" for exact offsetting and describes approximations dealers use
- Post-LBO CDS spread levels in Example 19: illustrative only; actual levels depend on leverage ratios, sector, and market conditions
- Exact LCDS spread levels and trading conventions: O'Kane discusses LCDS but market conventions have evolved since publication; I'm not sure about current LCDS market liquidity
- Crisis correlation spike magnitudes (20-30% to 50-70%): general market knowledge, not sourced from O'Kane with specific numbers
- Specific recovery rate thresholds triggering capital structure trades: depends on issuer, sector, and market conditions
