# Chapter 51: Tranche Risk — Tranche PV01, Correlation Risk, and Jump-to-Default Clustering

---

## Introduction

A trader hedges a mezzanine tranche position using an index CDS, matching spread DV01 to the penny. The next morning, a name in the portfolio defaults—and the P&L report shows a USD 3 million loss despite the "perfect" hedge. What went wrong?

The answer lies in a fundamental truth about tranche risk: **the sensitivities that matter most are not the ones that appear in a standard first-order risk report.** Single-name CDS and even index CDS respond to spread movements in roughly linear fashion. Tranches do not. The nonlinear mapping from portfolio losses to tranche losses—the "hockey stick" function that clips losses at attachment and caps them at detachment—creates risk characteristics that confound conventional hedging intuition.

Practitioner tranche risk reports distinguish two fundamentally different types of spread sensitivity: *systemic delta*, which measures exposure to parallel movements in all portfolio spreads, and *idiosyncratic delta*, which measures exposure to a single name widening in isolation. For equity tranches, these two risk measures can be very different in magnitude. A trader who hedges only systemic risk can be left with large idiosyncratic exposure; one who hedges only idiosyncratic risk can be poorly hedged for market-wide moves.

Practical takeaway: the “right” hedge ratio often lies somewhere between the pure systemic and pure idiosyncratic extremes, depending on whether you expect the next move to be market‑driven (systemic) or name‑specific (dispersion).

This chapter develops a risk framework for synthetic tranches. Section 1 establishes the core definitions: tranche loss mapping, expected tranche loss, and a tranche survival curve that enables CDS-style valuation. Section 2 provides a desk-friendly P&L decomposition—spread, correlation, jump-to-default, and recovery terms. Sections 3–5 develop each risk measure in detail: tranche PV01 and the systemic/idiosyncratic distinction (Section 3), correlation risk and Corr01 (Section 4), and jump-to-default clustering with its tail dependence implications (Section 5). Section 6 presents a hedging map—what instruments address which risks, and why PV01 hedges can fail under default clustering.

Prerequisites: [Chapter 38 — CDS Contract Mechanics](chapters/chapter_38_cds_contract_mechanics.md), [Chapter 39 — CDS Credit Events and Settlement](chapters/chapter_39_cds_credit_events_settlement.md), [Chapter 40 — CDS Auction Process](chapters/chapter_40_cds_auction_process.md), [Chapter 49 — Tranche Core Concepts](chapters/chapter_49_tranche_core_concepts_etl_pv.md), [Chapter 50 — Correlation and Tranche Pricing Frameworks](chapters/chapter_50_correlation_tranche_pricing_frameworks.md)  
Follow-on: [Chapter 52 — Credit Trading Strategies](chapters/chapter_52_credit_trading_strategies.md)

---

## Learning Objectives
- Translate a tranche quote regime (running spread vs upfront+fixed coupon) into cashflows and PV objects.
- Compute and interpret tranche PV01, Corr01, and VOD/JTD with explicit bump object, units, and sign.
- Explain (mechanistically) why “PV01-matched” hedges can still lose money under defaults and default clustering.
- Use unit/sign checks and limiting cases to debug tranche PV and risk numbers.

---

## Setup and Definitions

- **Valuation perspective:** unless stated otherwise, PV and sensitivities are reported from the protection buyer (long protection) perspective.
- **Portfolio notional:** $F$ in USD.
- **Losses and tranche strikes:** expressed as fractions of portfolio notional. Example: $A = 3\% \Rightarrow A = 0.03$.
- **Discounting:** $Z(t)$ is the discount factor from time 0 to $t$ (unitless).
- **Spreads:** contractual running premium $s$ quoted in bp/year. Convert to decimal rate $s_{\mathrm{dec}} = s / 10{,}000$.
- **Accrual fractions:** $\Delta(t_{i-1}, t_i)$ is the year fraction between payment dates (units: years).
- **Tranche survival curve:** $Q(t; A, D) \in [0, 1]$ is the expected surviving fraction of tranche notional (defined precisely below). This "tranche survival curve" is used to map tranche valuation to CDS-style analytics.
- **Correlation parameter:** when we use a single-parameter one-factor dependence knob, we write $\rho$ (unitless). In the one-factor Gaussian copula discussion in the tranche "correlation 01" definition, $\rho = \beta^2$.
- **Immediate default shock / VOD:** "value-on-default" (VOD) is an idiosyncratic jump measure defined by repricing after an immediate default (and adding/subtracting any immediate loss payment $G$).

PV is reported from protection buyer (long protection):

$$\mathrm{PV} = \mathrm{PV}_{\mathrm{prot}} - \mathrm{PV}_{\mathrm{prem}} - \mathrm{Upfront}.$$

Many reference formulas are written for short protection; be explicit about sign flips when comparing formulas.

---

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $F$ | Portfolio notional (USD) |
| $t \in [0, T]$ | Time; $T$ is tranche maturity |
| $L(t)$ | Portfolio cumulative loss fraction by time $t$ (unitless, in $[0, 1]$) |
| $[A, D]$ | Tranche attachment/detachment (fractions of portfolio); $W = D - A$ |
| $\mathrm{TL}(L)$ | Tranche loss as a fraction of portfolio notional (unitless) |
| $\mathrm{ON}(L)$ | Tranche outstanding as a fraction of portfolio notional (unitless) |
| $\mathrm{ETL}(t)$ | Expected tranche loss fraction by $t$ (unitless) |
| $\mathrm{EON}(t)$ | Expected outstanding fraction by $t$ (unitless); $\mathrm{EON}(t) = W - \mathrm{ETL}(t)$ |
| $Q(t; A, D)$ | Expected surviving fraction of tranche notional (unitless), i.e., "tranche survival curve"; $Q(t; A, D) = \mathrm{EON}(t) / W$ |
| $Z(t)$ | Discount factor |
| $s$ | Contractual tranche spread (bp/year); $s_{\mathrm{dec}} = s / 10{,}000$ |
| $\rho$ | Dependence (correlation) parameter; $\Delta\rho$ is a bump size |
| $\mathrm{PV01}$ | PV sensitivity to a 1bp bump (defined precisely in Section 3) |
| $\mathrm{Corr01}$ | PV change for a 1% absolute increase in correlation (defined precisely in Section 4) |
| $\mathrm{JTD}$ | "Jump-to-default" for tranches, defined as PV change under a default-event scenario |
| $R$ | Recovery rate (unitless); $(1 - R)$ is loss-given-default (LGD) |

---

## 1. Core Concepts (Definitions First)

### 1.1 Portfolio Loss and Tranche Loss Mapping

#### Formal Definition (Units Explicit)

Let $L(t) \in [0, 1]$ be the portfolio cumulative loss fraction by time $t$:

$$L(t) = \frac{\mathrm{cumulative portfolio loss dollars by } t}{F}.$$

**Units:** unitless fraction of portfolio notional.

For tranche $[A, D]$ with width $W = D - A$, define **tranche loss** (fraction of portfolio) as:

$$\boxed{\mathrm{TL}(L) = \min\{\max(L - A, 0),\; W\}}$$

**Units:** unitless fraction of portfolio notional.

Define **tranche outstanding** (fraction of portfolio) as:

$$\boxed{\mathrm{ON}(L) = W - \mathrm{TL}(L)}$$

**Units:** unitless fraction of portfolio notional.

#### Intuition

- $\mathrm{TL}(L)$ is a "clipped" version of portfolio loss: nothing happens until attachment $A$, then tranche takes losses linearly until detachment $D$, then it is fully written down (loss $W$).
- $\mathrm{ON}(L)$ is what remains of the tranche.

#### Trading/Risk Practice

Every risk measure for tranches ultimately flows through:

$$\mathrm{portfolio loss distribution} \;\Rightarrow\; \mathrm{tranche loss distribution} \;\Rightarrow\; \mathrm{PV and Greeks}.$$

Nonlinearity ($\min/\max$) is why tranches exhibit strong tail and jump sensitivity.

---

### 1.2 Expected Tranche Loss and Expected Outstanding

#### Formal Definition (Units Explicit)

**Expected tranche loss:**

$$\mathrm{ETL}(t) = \mathbb{E}[\mathrm{TL}(L(t))].$$

**Units:** fraction of portfolio notional.

**Expected outstanding:**

$$\mathrm{EON}(t) = \mathbb{E}[\mathrm{ON}(L(t))] = W - \mathrm{ETL}(t).$$

**Units:** fraction of portfolio notional.

**Tranche survival curve** (fraction of tranche notional outstanding):

$$\boxed{Q(t; A, D) = \frac{\mathrm{EON}(t)}{W} \in [0, 1]}$$

This is the "expected tranche surviving percentage notional" used to value tranches via CDS-style formulas.

#### Intuition

- $\mathrm{ETL}(t)$ is "how much of the tranche you expect to have lost by time $t$".
- $\mathrm{EON}(t)$ is "expected remaining premium-paying notional".

#### Trading/Risk Practice

- $\mathrm{ETL}(t)$ drives protection leg PV (expected loss payments).
- $\mathrm{EON}(t)$ drives premium leg PV (expected running spread payments).

---

### 1.3 Premium Leg PV and Protection Leg PV (CDS-Style Representation)

#### Formal Definition

A useful CDS-style valuation representation for a tranche uses the tranche survival curve $Q(t;A,D)$ and discount factors $Z(t)$.

**Protection Leg PV** (per unit tranche notional):

$$\mathrm{Protection Leg PV} = \int_0^T Z(s)\,(-dQ(s; A, D)).$$

**Tranche PV** (short protection, per unit tranche notional) is written as premium leg minus protection leg using a trapezoidal discretization:

$$V(A, D) = \frac{s_{\mathrm{dec}}}{2} \sum_{i=1}^{N} \Delta_i Z(t_i)(Q(t_{i-1}) + Q(t_i)) \;-\; \int_0^T Z(s)\,(-dQ(s)).$$

Once $Q$ is available, tranche valuation follows the same structure as CDS valuation: replace issuer survival by tranche survival (and, in the CDS mapping, treat the tranche as a zero-recovery exposure).

#### Intuition

- Premium leg is "spread $\times$ risky annuity $\times$ expected outstanding".
- Protection leg is "discounted expected loss increments".

#### Trading/Risk Practice

Desk analytics frequently store/trade on objects like:
- $\mathrm{ETL}(t)$ curves
- $Q(t)$ curves
- "tranche annuity / RPV01" objects (premium-leg PV per bp)

---

### 1.4 Tranche PV01 / Spread Risk (Quote Mechanics, USD /bp)

#### Formal Definition (What Is Bumped, Units Explicit)

There are two distinct but commonly conflated "spread risks":

**1. Contractual tranche-spread PV01 (quote PV01)** — mechanical sensitivity to the contractual running premium $s$ on the tranche contract:

For a spread-quoted tranche, define:

$$\boxed{\mathrm{PV01}_{\mathrm{tranche spread}} = \frac{\partial \mathrm{PV}}{\partial s} \times 1\mathrm{bp}}$$

where $s$ is the contractual tranche spread quote (bp/year).

**Units:** USD per 1bp.

**2. Underlying portfolio spread risk (systemic/idiosyncratic spread DV01 + deltas)** — DV01-style PV changes (and corresponding hedge notionals) under issuer CDS curve moves, e.g., a **systemic** bump (all names) versus an **idiosyncratic** bump (one name, others held fixed).

This chapter's "Tranche PV01" is item (1) unless explicitly stated otherwise.

> **Pitfall — “PV01” bump-object mismatch:** tranche PV01 can mean (a) sensitivity to the contractual tranche spread $s$, or (b) sensitivity to the underlying issuer spread curves (systemic/idiosyncratic DV01/deltas).  
> **Why it matters:** a hedge that matches (a) will generally *not* hedge (b), and neither one hedges discrete default jumps (VOD/JTD).  
> **Quick check:** before trusting a “PV01-matched” hedge, write down (i) what is bumped, (ii) what is rebuilt/recalibrated, and (iii) the units/sign (USD per 1bp).

#### Intuition

`PV01_tranche_spread` is basically the discounted expected outstanding notional: if the tranche is likely to be outstanding, PV01 is large; if likely to be quickly written down, PV01 is small.

**Check (sign + scale):** for a protection buyer (who pays the running spread), $\partial PV/\partial s \lt 0$ so `PV01_tranche_spread` is typically **negative**; for a protection seller it is typically positive. As a rough order-of-magnitude check, if tranche face is `N_tr_face = W*F = USD 40mm`, average surviving fraction is about `Q_avg = 75%`, and premium runs for about `T = 5` years, then `|PV01_tranche_spread| per bp ≈ 1e-4 * Q_avg * N_tr_face * T ≈ USD 15,000`, before discounting and accrual details. If your PV01 is `USD 150,000/bp`, you are probably off by a factor of 10 in bp/percent or notional scaling.

#### How It Appears in Practice

Used to:
- compute/paraphrase par spread $s^{\star}$ ("breakeven tranche spread"),
- convert PV moves into "spread moves",
- size hedges when hedging tranche spread quotes with other tranche contracts.

#### Quote Regime Caveat (Upfront + Running Premium)

Some tranche quotes are upfront + fixed coupon rather than pure running spread. One common illustrative convention is: quotes are in bp except for a $0$–$3\%$ tranche where the quote is an upfront percent of tranche principal plus 500 bp/year running premium.

Quote conventions vary by index, series, vintage, and clearing/settlement conventions. In this chapter we use generic notation (running spread, fixed coupon + upfront) and highlight the conversion formulas; confirm the exact quoting conventions for the specific product you trade.

---

### 1.5 Correlation Risk (PV Sensitivity to Dependence)

#### Formal Definition

Correlation risk is PV sensitivity to a correlation/dependence parameter. A practical desk scalar is:

$$\boxed{\mathrm{Corr01} = V(\rho + 0.01) - V(\rho)}$$

i.e., the change in tranche value due to a 1% absolute increase in correlation (with $\rho = \beta^2$ in that notation).

A normalized "rho" is also defined as the change in value per unit correlation per unit tranche face value.

**Check (methodology + nonlinearity):** because tranche PV can be nonlinear in $\rho$ (and base-correlation bumps can propagate through interpolation), it is often worth computing a symmetric finite difference as a diagnostic: `Corr01_sym ≈ (V(rho+0.01)-V(rho-0.01))/2`, holding the same calibration/interpolation rules fixed. Large one-sided vs symmetric differences are a red flag that a local Corr01 is not stable and scenario shocks are more informative.

#### Intuition

- Correlation controls how often defaults arrive together (or, more broadly, how much probability mass is in the joint tail of portfolio loss).
- Changing correlation changes the shape of the portfolio loss distribution, not just its mean.

#### Trading/Risk Practice

Correlation is often treated as an implied parameter extracted from tranche quotes. Even if marginal spreads are unchanged, a dependence move reshapes the portfolio loss distribution and can move tranche PV materially.

---

### 1.6 Jump-to-Default (JTD) / Default-Event Risk for Tranches

#### Formal Definition

Define tranche JTD under a specified default event scenario $\omega$ as:

$$\mathrm{JTD}_\omega = \mathrm{PV}_{\mathrm{after}}(\omega) - \mathrm{PV}_{\mathrm{before}}.$$

A standard idiosyncratic tranche risk measure is **value-on-default (VOD)**. This measures the impact of an immediate default and the resulting change in value of a tranche position.

Consider an immediate-default scenario for one portfolio constituent. Let:
- $H_0$ be the defaulted name’s notional as a **fraction** of portfolio notional $F$ (so the default loss fraction is `H0*(1-R0)`),
- $R_0$ be the recovery (or auction final price proxy),
- $L_1$ be the portfolio cumulative loss fraction just before the default.

Then the portfolio loss jumps by:

$$\Delta L = H_0(1-R_0), \qquad L^+ = L_1 + \Delta L.$$

**Check (why “PV01-matched” can still blow up):** take an equal-weight 125-name portfolio with $H_0=1/125$ and assume $R_0=40\%$, so $\Delta L \approx 0.60/125 \approx 0.48\%$. For a $[3\%,7\%]$ tranche sitting just below attachment at $L_1=2.8\%$, the post-default loss is $L^+=3.28\%$, so tranche loss jumps by about $0.28\%$ of portfolio notional (7% of tranche face because $0.28/4=7\%$). On $F=USD 1\mathrm{bn}$, that is a $USD 2.8\mathrm{mm}$ loss—an event P&L that has little to do with small-spread PV01 hedges.

The immediate tranche loss increment (fraction of portfolio) is:

$$\Delta \mathrm{TL} = \mathrm{TL}(L^+) - \mathrm{TL}(L_1), \qquad 0 \le \Delta \mathrm{TL} \le W.$$

In strike notation `K1-K2` (think `K1 = A` and `K2 = D`), the same subordination-consumption shift is often written:

$$K_{1} \rightarrow K_{1}-H_{0}\left(1-R_{0}\right), \qquad K_{2} \rightarrow K_{2}-H_{0}\left(1-R_{0}\right).$$

In the common sub-case where the tranche is unhit just before the event and becomes hit after the event, the loss increment simplifies to:

$$\Delta \mathrm{TL}=\max \left[L_{1}+H_{0}\left(1-R_{0}\right)-K_{1}, 0\right] \quad \mathrm{(assume } \Delta \mathrm{TL} \lt K_{2}-K_{1} \mathrm{)}.$$

So the **cash settlement** to a long-protection holder is:

$$\boxed{G = F \cdot \Delta \mathrm{TL} \;\;\;(\mathrm{USD})}$$

A common modeling recipe for the **post-event MTM** is:
1. Remove the defaulted name from the remaining portfolio (updating portfolio notional/factor consistently).
2. Treat the realized loss as consuming subordination: shift strikes down by the loss increment, $A \rightarrow A - \Delta L$ and $D \rightarrow D - \Delta L$ (equivalently: `K1 -> K1 - H0*(1-R0)` and `K2 -> K2 - H0*(1-R0)`).
3. Reprice the surviving tranche on the reduced outstanding to obtain $\mathrm{PV}_{\mathrm{after}}$.

In this chapter, we treat VOD/JTD-style measures as **total default-event value change**: post-event MTM change plus any immediate cash loss payment. Some systems report the cash payment and the MTM change separately; confirm your reporting convention.

#### Trading/Risk Practice

JTD/VOD is crucial because PV01 hedges typically target small continuous moves, while defaults produce discrete jumps.

---

### 1.7 Jump-to-Default Clustering / Tail Risk

#### Formal Definition

"Clustering" means that defaults are not isolated; multiple defaults can occur in the same stress state or close in time.

In dependence language, a key notion is **tail dependence:** whether variables can remain jointly extreme deep in the tail.

One standard definition is the **upper tail dependence coefficient**, which asks: as the threshold moves deeper into the tail, does the conditional probability of an extreme in $X_2$ given an extreme in $X_1$ converge to a nonzero limit?

Formally, for $X_1,X_2$ with marginal CDFs $F_1,F_2$,

$$\lambda_{\mathrm{u}}\left(X_{1}, X_{2}\right)=\lim_{q \rightarrow 1^{-}} P\!\left(X_{2}\gt F_{2}^{\leftarrow}(q) \mid X_{1}\gt F_{1}^{\leftarrow}(q)\right).$$

If $\lambda_{\mathrm{u}}\in(0,1]$, the pair has upper tail dependence; if $\lambda_{\mathrm{u}}=0$, they are asymptotically independent in the upper tail.

Analogously, the **lower** tail dependence coefficient is:

$$\lambda_{\ell}\left(X_{1}, X_{2}\right)=\lim_{q \rightarrow 0^{+}} P\!\left(X_{2}\le F_{2}^{\leftarrow}(q) \mid X_{1}\le F_{1}^{\leftarrow}(q)\right).$$

If the limit exists, it lies in $[0,1]$.

Gaussian copulas do not exhibit tail dependence except in the extreme case of perfect positive correlation; by contrast, the $t$-copula exhibits positive tail dependence for finite degrees of freedom. Section 5.4 gives a closed-form expression (and limiting-case checks) that make this contrast precise.

#### Intuition

Senior tranches are exposed to rare joint-default states; clustering amplifies those states' probability.

#### Trading/Risk Practice

Clustering risk often appears as:
- large losses in "tail scenarios" despite benign average spread moves,
- poor hedge performance under jump events,
- strong sensitivity to model choice (Gaussian vs $t$-copula, mixture models, etc.).

#### Tranche-Specific Mechanism (Why Clustering Shows Up in One-Factor Models)

In one-factor credit models, defaults cluster mechanically because the common factor shifts many names’ latent variables together. A name with a high loading to the common factor is most likely to default in the same “systemic stress” states in which many other names are also close to (or beyond) their default thresholds.

---

### 1.8 Risk Report View of a Tranche Position (Minimum Set)

A desk "risk report" view (conceptual) should include:

| Measure | Description |
|---------|-------------|
| **PV** | USD |
| **Tranche-spread PV01** | USD per 1bp in contractual tranche spread |
| **Correlation delta** | e.g., Corr01: USD per 1% correlation bump |
| **JTD / VOD** | USD jump under specified default scenario |
| **Recovery sensitivity** | USD per 1% recovery change |

Recovery sensitivity is convention-dependent: it may refer to recovery on the defaulted name(s), a portfolio recovery assumption, or “final price” from an auction settlement mechanism. Always confirm which definition your risk system uses.

---

## 2. Risk Decomposition for Tranche Books (Organizing Backbone)

A practitioner decomposition is a bookkeeping identity for explaining daily P&L and sizing hedges:

$$\Delta\mathrm{PV} \approx \underbrace{\Delta\mathrm{PV}_{\mathrm{Spread/PV01}}}_{\mathrm{small spread-like moves}} + \underbrace{\Delta\mathrm{PV}_{\mathrm{Corr}}}_{\mathrm{dependence shift}} + \underbrace{\Delta\mathrm{PV}_{\mathrm{JTD}}}_{\mathrm{default jumps}} + \underbrace{\Delta\mathrm{PV}_{\mathrm{Rec/FinalPrice}}}_{\mathrm{LGD/final price}} + \underbrace{\Delta\mathrm{PV}_{\mathrm{Residual}}}_{\mathrm{model/basis/liquidity}}$$

Below is how to interpret each term economically and operationally.

---

### 2.1 Spread/PV01 Term (Small "Spread-Like" Moves)

#### Economic Meaning

Represents P&L from small changes in quoted tranche spread(s) and/or underlying credit spreads that shift the loss distribution slightly.

#### Operational Implementation

Two common measurements:
1. contractual tranche-spread PV01 (this chapter's Section 3),
2. underlying systemic/idiosyncratic spread risk (issuer curve bumps), measured by repricing under systemic vs idiosyncratic bumps (DV01/delta-style measures).

#### Shock/Scenario

- "+5bp move in tranche spread" (quote move).
- "+1bp parallel move in issuer CDS curves" (systemic move) vs "single-name widening" (idiosyncratic move).

#### Hedges (Conceptual Mapping)

- **Quote PV01:** hedge with other tranche contracts (spread PV01 matching).
- **Underlying spread risk:** hedge with CDS index and/or baskets of single names; in practice, spread moves mix systemic and idiosyncratic components, so hedging is not purely mechanical.

---

### 2.2 Correlation Term

#### Economic Meaning

Captures P&L due to changes in dependence (correlation, copula tail, skew), holding marginal default probabilities approximately fixed.

#### Operational Implementation

- **Correlation 01:** bump correlation and reprice.
- In a base-correlation framework, the tranche depends on `rho(K1)` and `rho(K2)`; changes in the base correlation curve can create hedge slippage when spreads move (mapping effects).

#### Shock/Scenario

- "Index tranches reprice with index spreads fixed" (pure correlation move).
- A common base-correlation risk procedure is: hold index spreads fixed, bump standard tranche contractual spreads, rebuild the base correlation curve, then reprice to isolate correlation-only effects.

#### Hedges (Conceptual Mapping)

Hedging correlation requires other correlation products (other tranches). The only exact offset is selling the same tranche, which is often impractical.

---

### 2.3 Default-Event / JTD Term

#### Economic Meaning

Captures discrete jumps from realized defaults (or clustered defaults).

#### Operational Implementation

VOD/JTD computed by repricing after an immediate default; include immediate loss payment $G$ when the default crosses the attachment and hits the tranche.

#### Shock/Scenario

- "One-name default at recovery $R$"
- "Cluster of $k$ defaults" (tail proxy)

#### Hedges (Conceptual Mapping)

Use scenario-based hedging mentality: hedges that work for small spread moves may not work for jump events.

---

### 2.4 Recovery/Final-Price Term

#### Economic Meaning

Captures P&L from changes in loss-given-default assumptions (recovery rate, auction final price), and from systematic co-movement between default rates and recoveries.

#### Operational Implementation

Reprice with different recovery assumptions.

If your scenario framework assumes recoveries are lower in high-default states, that coupling extends the upper tail of the portfolio loss distribution and increases senior-tranche tail risk.

#### Auction Settlement Mechanics for Tranches

When a credit event occurs on a portfolio constituent, the settlement process determines the actual loss:

1. **Credit Event Declaration:** ISDA Determinations Committee confirms the credit event.

2. **Auction Process:** A CDS auction determines the final price for the defaulted reference obligation (see Chapter 40 for full auction mechanics).

3. **Final Price → Loss Calculation:**
   $$\mathrm{Loss} = (1 - \mathrm{Final Price}) \times \mathrm{Defaulted Notional}$$

4. **Tranche Impact:** The portfolio loss $L$ increases, and tranche loss $\mathrm{TL}(L)$ is recalculated.

5. **Settlement:** Protection seller pays $G = \Delta\mathrm{TL} \times F$ to protection buyer if the tranche is hit.

**Example Title**: Auction settlement → tranche loss payment $G$ (toy timeline)

**Context**
- You are long protection on a 3–7% tranche. A name defaults and settles via an auction-style final price.
- Goal: translate “Final Price” → portfolio loss jump → tranche loss jump → cash settlement amount.

**Timeline (Toy Dates; Real Dates Follow the Auction Process)**
- Trade date (valuation date): 2026-02-15
- Credit event date (assumed): 2026-03-01
- Auction final price published (assumed): 2026-03-10
- Cash settlement date (assumed): 2026-03-12

**Inputs**
- Portfolio notional: $F = USD 500$mm across 100 equal names (USD 5mm each)
- Tranche: $[A,D]=[3\%,7\%]$ (width $W=4\%$)
- Portfolio loss before the new default: $L^- = 2.5\%$
- Defaulted name notional: USD 5mm
- Auction final price: 35% (treat as recovery for loss calculation)

**Outputs (What You Produce)**
- Portfolio loss jump: $\Delta L$
- Tranche loss jump: $\Delta \mathrm{TL}$
- Cash settlement amount: $G = \Delta\mathrm{TL}\times F$ (USD)

**Step-by-step**
1. **Loss dollars on the defaulted name**
   $$\mathrm{Loss}_{\mathrm{name}} = (1-\mathrm{Final Price})\times USD 5\mathrm{mm} = (1-0.35)\times USD 5\mathrm{mm}=USD 3.25\mathrm{mm}$$
2. **Convert to portfolio loss fraction**
   $$\Delta L=USD 3.25\mathrm{mm}/USD 500\mathrm{mm}=0.0065=0.65\%$$
3. **Update portfolio loss**
   $$L^+=L^-+\Delta L=2.5\%+0.65\%=3.15\%$$
4. **Tranche loss before/after**
   - Before: $\mathrm{TL}(2.5\%)=\min(\max(2.5\%-3\%,0),4\%)=0$
   - After: $\mathrm{TL}(3.15\%)=\min(\max(3.15\%-3\%,0),4\%)=0.15\%$
   - Increment: $\Delta\mathrm{TL}=0.15\%$
5. **Cash settlement**
   $$G=\Delta\mathrm{TL}\times F=0.15\%\times USD 500\mathrm{mm}=USD 0.75\mathrm{mm}$$

**Cashflows (table)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-03-12 | +USD 0.75mm | Protection buyer receives $G$ because the default pushed $L$ above attachment |

**P&L / Risk Interpretation**
- $G$ is the realized “jump” cashflow that a PV01 hedge is *not* designed to offset.
- Separately (and additionally), the surviving position reprices because subordination has been consumed.

**Sanity Checks**
- Unit check: $\Delta\mathrm{TL}$ is a fraction; $\Delta\mathrm{TL}\times F$ is USD.
- Limit check: if Final Price were 50%, then $\Delta L=0.5\%$ and $L^+=3.0\%$ (exactly at attachment), so $G=0$.

**Debug Checklist (When Your Settlement Amount Looks Wrong)**
- Did you apply the loss to the correct name notional (not the whole index notional)?
- Did you convert points/percent correctly (0.15% = 0.0015)?
- Did you apply $\mathrm{TL}(L)=\min(\max(L-A,0),W)$ with $A,D,W$ as fractions?

#### Shock/Scenario

- "Recovery down 10 points in stress"
- "Cluster default with low recoveries" (tail + recovery coupling)
- "Auction final price uncertainty" (pre-auction vs post-auction exposure)

---

### 2.5 Residual/Model Risk Term

#### Economic Meaning

Everything not captured above: model specification error, calibration instability, interpolation arbitrage, liquidity/basis.

#### Operational Implementation

Model comparison, backtesting, stress tests.

The base correlation discussion highlights that interpolation choices can generate arbitrage-like features for non-standard tranches and that linear interpolation of base correlation is not guaranteed arbitrage-free.

#### Shock/Scenario

"Skew shift," "mapping change," "liquidity gap," "bid/offer widen."

#### Hedges

Often not hedgeable mechanically; managed via limits, diversification, and stress testing (scenario weighting approaches appear in standard risk management practice).

---

## 3. Tranche PV01 / Spread Risk (First-Order Risk)

### 3.1 Definition (Spread-Quoted Tranches)

Assume the tranche is quoted purely as a running spread $s$ (bp/year). Define:

$$\boxed{\mathrm{PV01} \equiv \frac{\partial \mathrm{PV}}{\partial s} \times 1\mathrm{bp}}$$

- **Bump:** $s \mapsto s + 1$ bp (contractual tranche spread).
- **Units:** USD per 1bp.

Using the CDS-style premium leg representation (trapezoidal sum with tranche survival $Q$), PV is linear in $s$ in that representation, so PV01 is essentially the (negative) discounted expected outstanding "annuity" term.

---

### 3.2 Definition (Upfront + Coupon Quote Tranches)

If the tranche is quoted as upfront $U$ (percent of tranche principal) plus fixed running coupon $c$ (bp/year), then:

**Upfront sensitivity:**

$$\mathrm{PV01}_{\mathrm{upfront}} = \frac{\partial \mathrm{PV}}{\partial U} \times 1\%,$$

**Units:** USD per 1% upfront.

**Coupon PV01** is still defined as $\frac{\partial \mathrm{PV}}{\partial c} \times 1\mathrm{bp}$.

The reference gives an example where the $0$–$3\%$ tranche is quoted as upfront percent plus 500 bp/year running, while other tranches are quoted in bp.

**Upfront ↔ spread conversion (CDS-style):** In the fixed-coupon quote convention, the upfront (as a fraction of tranche notional) converts to par spread via:

$$\boxed{\mathrm{Upfront} = (s^{\star} - c_{\mathrm{fixed}})\times \mathrm{RPV01}}$$

So:

$$\boxed{s^{\star} = c_{\mathrm{fixed}} + \frac{\mathrm{Upfront}}{\mathrm{RPV01}}}$$

As a quick rule of thumb, “1% upfront” corresponds to roughly $\frac{100}{\mathrm{RPV01}}$ bp of spread difference from the fixed coupon (with RPV01 in years). Always confirm the product’s conventions (fixed coupon, accrual-on-default, and whether quotes are given as price vs points upfront).

---

### 3.3 Dependence of PV01 on ETL/EON and Discounting (Unit Checks)

For a long protection position, schematically:

$$\mathrm{Premium leg} \propto s_{\mathrm{dec}} \times \sum Z \Delta \times \mathrm{expected outstanding}.$$

Therefore $|\mathrm{PV01}|$ increases with expected outstanding and with discount factors (higher $Z$, longer maturity).

#### Unit Check

Delta in years, $Z$ unitless, expected outstanding in fraction of portfolio; multiply by $F$ gives USD notional base; multiply by 1bp $= 10^{-4}$ gives USD per bp. ✓

---

### 3.4 How PV01 Varies by Attachment (Equity vs Senior)

#### Qualitative Shape (Attachment Dependence)

A common qualitative pattern is: tranche RPV01 is lowest for the riskiest equity tranche and rises for more senior tranches (with caveats for super-senior amortization).

#### Practical Risk Report Patterns (RPV01, Delta, Leverage)

Rather than relying on any single numeric “risk report snapshot,” focus on the robust qualitative patterns:

- **RPV01 tends to increase with seniority.** Equity is expected to amortize earlier (losses eat notional), reducing premium‑leg duration.
- **Systemic delta tends to decrease with seniority.** Equity can carry a very large share of portfolio spread sensitivity; super‑senior often carries much less (until the market is distressed).
- **Leverage falls as you go up the capital structure.** Small notional slices can embed large spread exposure.
- **Correlation sensitivity can flip sign.** Equity is often “long correlation” when short protection, while senior tranches are often the opposite.

#### Intuition

- **Equity** is expected to be written down sooner → lower expected outstanding → lower PV01.
- **Senior** is expected to remain outstanding → higher PV01.

---

## 3.5 Systemic and Idiosyncratic Gamma (Second-Order Sensitivity)

Beyond first-order delta measures, tranches exhibit significant **gamma**—the second derivative of value with respect to spread moves. O'Kane Ch 17 distinguishes two types of gamma that parallel the systemic/idiosyncratic delta distinction.
Beyond first-order delta measures, tranches exhibit significant **gamma**—the second derivative of value with respect to spread moves. Two useful notions parallel the systemic/idiosyncratic delta distinction.

### 3.5.1 Systemic Gamma Definition

**Systemic gamma** measures the curvature of tranche value with respect to parallel portfolio spread moves:

$$\boxed{\Gamma_s = \frac{\partial^2 V}{\partial S^2} \times (1\mathrm{bp})^2}$$

where $S$ represents a parallel shift in all portfolio spreads.

**Finite-difference calculation:**

$$\Gamma_s = V(S + 1\mathrm{bp}) - 2V(S) + V(S - 1\mathrm{bp})$$

**Units:** USD (the P&L impact of the gamma effect per 1bp squared move).

### 3.5.2 Idiosyncratic Gamma Definition

**Idiosyncratic gamma** measures the curvature with respect to a single name's spread:

$$\boxed{\Gamma_i = \frac{\partial^2 V}{\partial S_i^2} \times (1\mathrm{bp})^2}$$

For a portfolio of $N$ names, the **aggregate idiosyncratic gamma** is the sum:

$$\Gamma_{\mathrm{idio, total}} = \sum_{i=1}^{N} \Gamma_i$$

### 3.5.3 Gamma Sign Patterns Across the Capital Structure

Published tranche risk report examples show a useful qualitative pattern:

- **Equity:** systemic gamma tends to be **negative**, while aggregate idiosyncratic gamma can be **positive**.
- **Senior:** systemic gamma tends to be **positive**, while idiosyncratic gamma can be **negative**.

The exact magnitudes depend on the calibration and the market state; the sign pattern is the key hedging intuition.

### 3.5.4 The Gamma Sign Pattern

The gamma signs exhibit a striking pattern that has profound hedging implications:

**Equity tranches have negative systemic gamma and positive idiosyncratic gamma:**
- **Negative systemic gamma** means convexity works *against* equity holders for large parallel spread moves. A 10bp spread widening costs more than 10× a 1bp move—the position "bleeds" under volatility.
- **Positive idiosyncratic gamma** reflects that individual name spread moves have diminishing marginal impact as the equity tranche approaches its attachment point.

**Senior tranches have positive systemic gamma and negative idiosyncratic gamma:**
- **Positive systemic gamma** means senior tranches benefit from convexity in large market-wide moves—they are "long volatility" in the systemic dimension.
- **Negative idiosyncratic gamma** reflects increasing marginal sensitivity as the tranche approaches attachment from individual name deterioration.

> **Desk Note: Negative Gamma and Re‑Hedging Costs**
>
> **Desk Reality:** when systemic gamma is negative, a tranche can look “delta-hedged” for small moves but still bleed P&L in volatile, market-wide spread moves because the hedge needs frequent rebalancing.  
> **Common break:** a close-of-business delta hedge looks fine on a 1bp shock, but a 10–20bp move (and the re-hedging needed along the way) produces slippage and transaction-cost P&L that a linear risk report won’t explain.  
> **What to check:** keep a simple “bump ladder” (±1bp, ±5bp, ±10bp) for the tranche+hedge and track second differences (gamma) alongside delta.

### 3.5.5 Gamma's Impact on Delta Hedging

Why does gamma matter for hedging? Consider a delta-neutral position hedged at current spreads. If spreads move significantly:

1. **The delta itself changes** (because gamma measures $\partial\Delta/\partial S$)
2. **The hedge becomes stale** and must be rebalanced
3. **Rebalancing costs real money** (transaction costs, bid-offer spreads)
4. **With negative gamma, rebalancing is always costly** (you buy high, sell low)

One standard finite-difference definition of systemic gamma uses a 1bp second difference:

$$\Gamma_s^{(1\mathrm{bp})} = V(S+1\mathrm{bp}) - 2V(S) + V(S-1\mathrm{bp}).$$

If you treat $\Gamma_s^{(1\mathrm{bp})}$ as the “gamma‑01” (units: USD per $(1\mathrm{bp})^2$), then a rough second‑order approximation for a $\Delta S$ move is:

$$\Delta V_{\Gamma} \approx \frac{1}{2}\,\Gamma_s^{(1\mathrm{bp})}\left(\frac{\Delta S}{1\mathrm{bp}}\right)^2.$$

For an equity tranche with $\Gamma_s^{(1\mathrm{bp})} \approx -4{,}000$ (USD), a 10bp parallel spread move gives:

$$\Delta V_{\Gamma} \approx \frac{1}{2} \times \Gamma_s^{(1\mathrm{bp})}\left(\frac{10\mathrm{bp}}{1\mathrm{bp}}\right)^2 = \frac{1}{2} \times (-4{,}000) \times 100 = -USD 200{,}000$$

This is *in addition to* any delta slippage from hedge rebalancing.

### 3.5.6 Hedging Decision Framework: Systemic vs Idiosyncratic

The hedging decision depends critically on the move you are trying to hedge:

| Market View | Hedge Approach | Rationale |
|-------------|----------------|-----------|
| Spreads move together (correlation-driven) | Use systemic delta | Parallel moves dominate |
| Spreads disperse (name-specific) | Use idiosyncratic delta | Name moves are independent |
| Mixed/uncertain | Blend of both | Hedge between the extremes |

**Quantitative hedging framework:**

Let $\alpha$ be the trader's view on the fraction of spread variance that is systemic:
- $\alpha = 1$: All moves are parallel → use systemic delta
- $\alpha = 0$: All moves are idiosyncratic → use idiosyncratic delta

**Blended hedge ratio:**

$$\Delta_{\mathrm{hedge}} = \alpha \cdot \Delta_{\mathrm{systemic}} + (1-\alpha) \cdot \Delta_{\mathrm{idio, total}}$$

This $\alpha$‑blend is a simple heuristic for “between the extremes.” Key point: spread moves are a mix of systemic and idiosyncratic components, and the hedge that works best depends on which component is likely to dominate.

---

## 4. Correlation Risk (Why PV Changes When Dependence Changes)

### 4.1 Why Dependence Changes Tranche PV

Dependence changes the distribution of the number/timing of defaults and therefore the distribution of portfolio loss $L(t)$.

In words: higher default clustering can reduce the probability of “a few scattered defaults” while increasing the probability of “a big cluster.” That tends to make **junior** protection less valuable and **senior** protection more valuable (all else equal, and for a fixed marginal default level).

That is the core mechanism:
- **Equity** cares about "at least some defaults" → correlation can reduce that likelihood (equity can become safer).
- **Senior** cares about "very many defaults" → correlation can increase that tail probability (senior becomes riskier).

---

### 4.2 Correlation Delta and Corr01 (Parameterization Clarity)

#### Primary Parameterization Used Here

We use a single-parameter $\rho$ (compound correlation / one-factor parameter) for clarity.

#### Definition

**Correlation 01 (Corr01):** PV change for a 1% absolute increase in correlation:

$$\boxed{\mathrm{Corr01} = V(\rho + 0.01) - V(\rho)}$$

**Finite-difference correlation delta (symmetric):**

$$\mathrm{CorrDelta} \approx \frac{V(\rho + \Delta\rho) - V(\rho - \Delta\rho)}{2\Delta\rho}$$

---

### 4.3 If Base Correlation Is Used (Minimal Desk Summary)

A `[K1, K2]` tranche can be decomposed as a linear combination of two base (equity) tranches and is priced by assigning different correlations to `[0, K1]` and `[0, K2]`.

**Expected loss relationship under base correlation:**

$$\mathbb{E}[L(T, K_1, K_2)] = \frac{\mathbb{E}_{\rho(K_2)}[\min(L(T), K_2)] - \mathbb{E}_{\rho(K_1)}[\min(L(T), K_1)]}{K_2 - K_1}.$$

This creates a known inconsistency (different base tranches assign different correlations to the same underlying portfolio) and base correlation is not arbitrage-free, though it has practical advantages.

#### What Is Bumped in Base Correlation?

A `[K1, K2]` tranche value is only sensitive to `rho(K1)` and `rho(K2)` within the base correlation framework.

Desk implementations differ for non‑standard strikes and interpolation (e.g., interpolating in correlation vs ETL space, enforcing shape constraints). Naive linear interpolation can generate arbitrage‑like behavior; confirm what your system does.

---

### 4.4 Tranche Dependence of Correlation Risk (Sign Intuition)

A typical pattern in one-factor copula-style models is:
- Long protection equity tranches can be **short correlation** (their value falls if correlation increases).
- More senior long protection tranches tend to be **long correlation** (value rises as correlation increases).
- There exists a "correlation neutral point" where sensitivity crosses zero.

We will verify this tranche dependence numerically in Examples 8–9 (toy dependence settings).

---

## 5. Jump-to-Default Clustering and Default-Event Scenarios

### 5.1 Single-Name Default Shock vs Clustered Default Shock

**Single-name default shock:** one issuer defaults with recovery $R$, producing portfolio loss increment:

$$\Delta L = \frac{(1 - R) \cdot \mathrm{defaulted face value}}{F}.$$

**Tranche loss increment:**

$$\Delta\mathrm{TL} = \mathrm{TL}(L^- + \Delta L) - \mathrm{TL}(L^-).$$

**Clustered default shock:** $k$ names default "at once" (or in a tightly clustered window), producing:

$$\Delta L = \sum_{j=1}^{k} \frac{(1 - R_j) \cdot \mathrm{face}_j}{F}.$$

This is a proxy for tail/clustering states.

---

### 5.2 Recovery/Final Price Mapping Defaults → Losses → Tranche Payouts

In a default-event scenario:
- recovery $R_0$ and defaulted face $H_0$ determine the portfolio loss increment $H_0(1 - R_0)$,
- attachment/detachment may be adjusted after default to reflect consumed subordination (depending on modeling/reporting choices),
- if the tranche is hit, an immediate loss payment $G$ is made and tranche outstanding is reduced.

In practice, “recovery” may be implemented via auction final price (for CDS-style settlement) or via an assumed recovery input in the model. Confirm which convention your desk and risk system use.

---

### 5.3 Scenario Suite (and What Proxies Clustering/Tail Risk)

A minimal scenario suite for tranche books:

| # | Scenario | Purpose |
|---|----------|---------|
| 1 | Single default, average recovery | Idiosyncratic JTD |
| 2 | Single default, low recovery | Recovery stress |
| 3 | $k$-name clustered default, average recovery | Default clustering proxy |
| 4 | $k$-name clustered default, low recovery | Tail + recovery coupling |
| 5 | Pure correlation move with spreads fixed | Dependence shock |
| 6 | Systemic spread widening + correlation move | Joint stress |

Scenarios (3)–(4) are the clearest proxies for clustering/tail risk.

---

### 5.4 Tail Dependence: t-Copula vs Gaussian Copula

A critical model risk for senior tranches concerns **tail dependence**—whether joint extremes persist far into the tail. A convenient scalar is a tail dependence coefficient (Section 1.7).

#### Gaussian Copula: No Tail Dependence (Except at Perfect Correlation)

For any bivariate Gaussian copula with $\rho \neq 1$, the lower tail dependence coefficient satisfies:

$$\boxed{\lambda_{\ell}=0}$$

In words: Gaussian copulas do not have tail dependence except in the extreme case of perfect positive correlation.

#### t-Copula: Closed-Form Tail Dependence Coefficient

For a bivariate $t$-copula with tail index $\nu$ and correlation $\rho$, the lower tail dependence coefficient is:

$$\boxed{\lambda_{\ell}=2 F_{t, \nu+1}\left(-\sqrt{\frac{(\nu+1)(1-\rho)}{1+\rho}}\right)}$$

where $F_{t, \nu+1}$ is the CDF of the $t$-distribution with tail index $\nu+1$.

**Check (limiting case):** since $F_{t, \nu+1}(-\infty)=0$, we have $\lambda_{\ell} \rightarrow 0$ as $\nu \rightarrow \infty$, consistent with the $t$-copula converging to the Gaussian copula as $\nu \rightarrow \infty$.

#### Implications for Senior Tranche Risk

**Why this matters for senior tranches:**

1. **Senior tranches are exposed to tail states:** The 15–30% super-senior tranche only suffers losses when portfolio losses exceed 15%—an extreme event requiring multiple defaults.

2. **Gaussian copula can understate clustering risk:** With $\lambda_{\ell}=0$, the Gaussian copula has no lower-tail dependence (except at perfect correlation), whereas tail-dependent copulas allocate more probability mass to joint-extreme states. For senior strikes, that can translate into materially different PVs versus tail-dependent alternatives.

3. **Model risk is largest for senior tranches:** small differences in how a model treats clustered defaults can translate into large PV differences for senior strikes, because senior protection only “turns on” in the far tail.

#### Practical Sanity Checks (Using Only the Tail-Dependence Coefficient)

- **Gaussian benchmark:** for a bivariate Gaussian copula with $\rho \neq 1$, $\lambda_{\ell}=0$ (no lower-tail dependence).
- **t-copula to Gaussian limit:** as $\nu \to \infty$, $\lambda_{\ell}\to 0$ (the $t$-copula converges to the Gaussian copula).
- **Perfect negative dependence limit:** as $\rho \to -1$, $\lambda_{\ell}\to 0$ (lower-tail dependence measures *positive* co-movement in the tail).

---

## 6. Hedging Map (Conceptual, Risk-First; No Trade Tips)

### 6.1 Hedge Spread PV01 (Small-Move Risk)

**Goal:** reduce sensitivity to small spread-like changes.

#### The Systemic vs Idiosyncratic Distinction

Distinguish **PV changes** from **hedge notionals**:

1. **Systemic DV01 (PV change):** bump *all* issuer curves up by 1bp (a parallel “portfolio‑wide” spread move) and reprice. The change in tranche PV is the systemic DV01.

2. **Systemic delta (hedge notional):** convert the tranche’s systemic DV01 into an amount of CDS protection to sell on the reference portfolio. One implementation is to value the reference portfolio under the same 1bp bump (an on‑market index starts near PV $\approx 0$) and scale so the CDS hedge offsets the tranche PV change.

3. **Idiosyncratic DV01\_i (PV change):** bump only issuer $i$ by 1bp and reprice; the tranche PV change is the idiosyncratic DV01 for name $i$.

4. **Idiosyncratic delta\_i (hedge notional):** convert idiosyncratic DV01\_i into CDS notional on name $i$ using that name’s CDS RPV01 to maturity (PV change divided by CDS RPV01).

**How this shows up on a desk:** in a homogeneous portfolio, a systemic hedge tends to be “equal‑weight” across names (roughly $1/N$ per name). Idiosyncratic hedges can be name‑dependent when spreads, recoveries, or factor loadings differ.

**Practical takeaway:** a systemic hedge works well only when moves are close to parallel; in practice, spreads move as a mixture of systemic and idiosyncratic components, so the “right” hedge lies between the systemic and idiosyncratic extremes and often requires judgement (and/or an explicit factor model).

> **Desk note (market impact):** hedging a tranche by selling single‑name protection can mechanically tighten those CDS spreads (a “synthetic bid”), which can also affect observed CDS‑cash basis. This is one reason a “perfect hedge” in a model can look imperfect in real P&L.

#### Failure Modes / Residual Risks

- **Basis:** tranche vs CDS index vs single names.
- **Liquidity:** hedges may be more/less liquid than tranche.
- **Model risk:** hedge ratios depend on model-implied deltas.
- **Jump risk:** PV01 hedges are not designed for defaults (Section 6.3).
- **Systemic/idiosyncratic mismatch:** hedge sized for one type of move underperforms for the other.

---

### 6.2 Hedge Correlation Risk (Hard by Construction)

#### Why It Is Hard

After spread and rate risks are hedged, remaining market risk is changes in correlation; correlation should not be assumed fixed.

In this simplified view, the natural hedging instruments are other correlation products (other tranches); exact offset requires the same tranche, which is often unrealistic.

#### Conceptual Hedges

- **Complete the capital structure:** combine equity, mezz, senior tranches on same portfolio to reduce net correlation exposure; however, premium legs do not perfectly offset, so a model is needed for precise hedges.
- **Skew hedging in base correlation:** hold index spreads fixed, bump standard tranche spreads, rebuild base correlation curve, and compute hedge notionals in standard tranches that offset small skew-driven PV changes in a non-standard tranche.

#### Worked Example: Capital Structure Combination for Correlation Reduction

Consider a trader long protection on the 3–7% mezzanine tranche (typically long correlation). To reduce net correlation exposure:

**Step 1:** Measure Corr01 of the mezzanine position.
- Suppose Corr01(3–7%) = +USD 150,000 per 1% correlation increase.

**Step 2:** Find opposing correlation exposure.
- Equity tranches are typically short correlation.
- Suppose Corr01(0–3%) = –USD 80,000 per 1% correlation increase.

**Step 3:** Size the hedge.
Use the hedge ratio `n_equity = -Corr01(3-7)/Corr01(0-3) = 1.875`.

Sell protection on 1.875× the notional of 0–3% equity to offset correlation exposure.

**Caveat:** This hedge is imperfect because:
1. Premium legs don't perfectly offset (different RPV01s).
2. The relationship between Corr01s is model-dependent.
3. Both tranches have spread risk that may or may not offset.

#### Skew Hedging Procedure (Base Correlation)

For a bespoke tranche with non-standard attachment points, one procedure is:

1. Hold index spreads fixed.
2. Bump each standard tranche contractual spread by 1bp.
3. Rebuild the base correlation curve from bumped standard tranche prices.
4. Reprice the bespoke tranche using the bumped base correlation curve.
5. The change in bespoke tranche value per standard tranche spread bump gives hedge ratios.

This procedure captures how the bespoke tranche is exposed to skew movements (changes in the base correlation curve shape).

#### Failure Modes / Residual Risks

- **Curve/mapping risk (base correlation "mapping" sensitivity):** bespoke tranche value depends on `rho(K1)`, `rho(K2)` and their dependence on portfolio spread; hedges can slip when mapping changes.
- **Interpolation/arbitrage artifacts** (linear interpolation can create arbitrage-like issues).
- **Model dependence:** correlation hedges rely on the pricing model being correct; if the model is wrong, hedges fail.

---

### 6.3 Hedge Default-Event / Tail Risk

#### Why Hedges May Fail

Default events produce discontinuous jumps (VOD/JTD). PV01 hedges address small continuous moves, not jumps.

#### Mindset

Use scenario-based hedging/limits: evaluate residual P&L under clustered default scenarios and recovery stress.

#### Residual Risks

- Recovery uncertainty (auction final price).
- Clustering/tail dependence model risk (Gaussian vs $t$-copula differences in tail dependence).

---

## 6.4 Key Visualizations for Tranche Risk

Understanding tranche risk requires visualizing the nonlinear relationships that distinguish tranches from linear instruments. The following charts are essential for risk management:

### The "Hockey Stick": Tranche Loss vs Portfolio Loss

**What it shows:** $\mathrm{TL}(L)$ as a function of portfolio loss $L$ for a tranche $[A, D]$.

**Shape:**
- Flat at zero for $L \lt A$ (losses below attachment don't touch the tranche)
- Linear slope = 1 for $A \leq L \leq D$ (tranche absorbs losses dollar-for-dollar)
- Flat at $W$ for $L \gt D$ (tranche fully written down)

**Why it matters:** This piecewise-linear shape is why tranches have different risk characteristics than linear instruments. The derivative $\partial\mathrm{TL}/\partial L$ jumps from 0 to 1 at attachment and from 1 to 0 at detachment.

### Correlation Sensitivity by Attachment Point

**What it shows:** Corr01 (or correlation delta) as a function of tranche attachment point.

**Typical shape:**
- **Equity (low attachment):** Negative Corr01 — value decreases when correlation increases
- **Mezzanine (middle):** Near-zero Corr01 — "correlation neutral point"
- **Senior (high attachment):** Positive Corr01 — value increases when correlation increases

**Why it matters:** Corr01 sign is position- and seniority-dependent. From the long-protection PV perspective used in this chapter, senior tranches are typically long correlation (PV rises as correlation increases), while equity tranches are typically short correlation; protection sellers have the opposite sign.

### RPV01 Profile Across the Capital Structure

**What it shows:** Tranche RPV01 (risky PV01) as a function of attachment point.

**Typical shape (qualitative):**
- Equity: RPV01 is often **lower** (expected notional amortizes earlier)
- Senior: RPV01 is often **higher** (expected to survive longer)

**Why it matters:** Premium-leg exposure varies dramatically across tranches. A senior tranche holder collects premium for longer, creating higher duration-like exposure.

### VOD Profile by Name

**What it shows:** Value-on-Default for each name in the portfolio.

**Typical shape:**
- Names trading at wider spreads: larger VOD (higher implied default probability means default is less "surprising")
- Names with higher face value: larger absolute VOD
- For long protection, VOD is driven by (i) any immediate loss payment if the default hits the tranche and (ii) the MTM reprice after subordination is consumed; sign/magnitude vary by tranche seniority and reporting convention (cash vs MTM).
- For very senior tranches, single-name defaults often produce little or no immediate cash payment, but MTM can still move because subordination is reduced.

**Why it matters:** Identifies which names contribute most to jump risk, enabling targeted hedging.

### Portfolio Loss Distribution: Low vs High Correlation

**What it shows:** PDF or CDF of portfolio loss $L(T)$ under different correlation assumptions.

**Typical shape:**
- Low correlation: bell-shaped, concentrated near expected loss
- High correlation: bimodal or fat-tailed, more probability mass at both zero and at extremes

**Why it matters:** High correlation increases "all or nothing" outcomes (more mass near zero loss and more mass in the extreme tail). From the long-protection PV perspective used in this chapter, this typically makes equity protection cheaper and senior protection more expensive, consistent with the Corr01 sign pattern above.

---

## 7. Math and Derivations (Step-by-Step)

### 7.1 PV in Terms of ETL/EON on a Time Grid

Let $0 = t_0 \lt t_1 \lt \cdots \lt t_N = T$.

#### Step 1: Express Protection Payments via Expected Tranche Loss Increments

- Tranche loss fraction at time $t$: $\mathrm{TL}(L(t))$.
- Expected tranche loss fraction: $\mathrm{ETL}(t) = \mathbb{E}[\mathrm{TL}(L(t))]$.
- Over interval $(t_{i-1}, t_i]$, the expected tranche loss increment is:

$$\Delta\mathrm{ETL}_i = \mathrm{ETL}(t_i) - \mathrm{ETL}(t_{i-1}) \geq 0.$$

**Protection leg PV (long protection, USD):**

$$\boxed{\mathrm{PV}_{\mathrm{prot}} \approx F \sum_{i=1}^{N} Z(t_i) \, \Delta\mathrm{ETL}_i}$$

**Unit check:** $F$ (USD) $\times$ $Z$ (unitless) $\times$ $\Delta\mathrm{ETL}$ (unitless) = USD. ✓

This is consistent with the representation $\int_0^T Z(t)\,(-dQ(t))$ once we note $Q(t) = \mathrm{EON}(t)/W$ and that loss increments correspond to $-dQ$ times tranche notional.

#### Step 2: Express Premium Payments via Expected Outstanding

**Expected outstanding fraction:**

$$\mathrm{EON}(t) = W - \mathrm{ETL}(t).$$

**Approximate expected outstanding over interval using trapezoid:**

$$\overline{\mathrm{EON}}_i = \frac{\mathrm{EON}(t_{i-1}) + \mathrm{EON}(t_i)}{2}.$$

**Premium leg PV (long protection pays premium, USD):**

$$\boxed{\mathrm{PV}_{\mathrm{prem}} \approx F \cdot \frac{s}{10{,}000} \sum_{i=1}^{N} \Delta_i Z(t_i) \, \overline{\mathrm{EON}}_i}$$

**Unit check:** $F$ (USD) $\times$ $s/10{,}000$ (1/year) $\times$ Delta (years) = USD, times $Z$ and $\overline{\mathrm{EON}}$ (unitless). ✓

This matches the CDS-style trapezoidal premium leg when written in $Q$ form.

#### PV (Long Protection)

$$\boxed{\mathrm{PV} = \mathrm{PV}_{\mathrm{prot}} - \mathrm{PV}_{\mathrm{prem}} - \mathrm{Upfront}}$$

---

### 7.2 PV01 Definition from Finite Differences (Robust)

Let $\mathrm{PV}(s)$ be PV as a function of contractual tranche spread $s$ in bp/year.

**Central-difference PV01:**

$$\boxed{\mathrm{PV01} \approx \frac{\mathrm{PV}(s + 1) - \mathrm{PV}(s - 1)}{2}}$$

**Units:** USD per bp.

**Sign sanity:** for long protection, increasing $s$ increases premium you pay, so PV should go down $\Rightarrow$ PV01 < 0.

---

### 7.3 Hedge Ratio via PV01 Matching (Transparent)

To hedge a target instrument "T" with hedge instrument "H" under small spread moves:

$$\Delta\mathrm{PV}_{\mathrm{total}} \approx \mathrm{PV01}_T \Delta s + n_H \cdot \mathrm{PV01}_H \Delta s \approx 0$$

so

$$\boxed{n_H = -\frac{\mathrm{PV01}_T}{\mathrm{PV01}_H}}$$

$n_H$ is the hedge notional scaling (or number of contracts) assuming PV01 scales linearly with notional.

**Unit check:** USD/bp divided by USD/bp is unitless. ✓

---

### 7.4 Correlation Delta via Finite Differences

Let $\mathrm{PV}(\rho)$ be tranche PV as a function of correlation parameter $\rho$.

$$\mathrm{CorrDelta} \approx \frac{\mathrm{PV}(\rho + \Delta\rho) - \mathrm{PV}(\rho - \Delta\rho)}{2\Delta\rho}$$

**Corr01 conversion:**

If Corr01 is defined as the PV change for 1% absolute correlation bump, then:

$$\boxed{\mathrm{Corr01} \approx \mathrm{CorrDelta} \times 0.01}$$

This matches the definition of Corr01 as a 1% absolute bump in correlation used throughout this chapter.

---

## 8. Worked Examples

### Common Toy Setup (Used Unless Otherwise Stated)

| Parameter | Value |
|-----------|-------|
| Portfolio notional | $F = USD 100{,}000{,}000$ |
| Payment grid (quarterly, $T = 1$y) | $t_0 = 0$, $t_1 = 0.25$, $t_2 = 0.50$, $t_3 = 0.75$, $t_4 = 1.00$ |
| Accruals | $\Delta_i = 0.25$ |
| Discount factors (toy) | $Z(0.25) = 0.990$, $Z(0.50) = 0.985$, $Z(0.75) = 0.980$, $Z(1.00) = 0.975$ |

---

### Example 1: Compute TL(L) and ON(L) Table for L = 0%, 2%, 5%, 10%, 20%

**Tranche:** $[A, D] = [3\%, 7\%] \Rightarrow W = 4\% = 0.04$.

Compute $\mathrm{TL}(L) = \min(\max(L - A, 0), W)$ and $\mathrm{ON}(L) = W - \mathrm{TL}(L)$.

| Portfolio loss $L$ | $L - A$ | $\max(L - A, 0)$ | $\mathrm{TL}(L)$ | $\mathrm{ON}(L)$ |
|--------------------|---------|------------------|----------------|----------------|
| 0% | -3% | 0% | 0% | 4% |
| 2% | -1% | 0% | 0% | 4% |
| 5% | 2% | 2% | 2% | 2% |
| 10% | 7% | 7% | 4% (cap) | 0% |
| 20% | 17% | 17% | 4% (cap) | 0% |

**Convert to dollars** (multiply by `F = USD 100mm`):
- At `L = 5%`: tranche loss = `2% * 100mm = USD 2.0mm`; outstanding = USD 2.0mm.
- At `L = 10%`: tranche loss = `4% * 100mm = USD 4.0mm`; outstanding = USD 0.

---

### Example 2: Given a Discrete Loss Distribution for L(T), Compute ETL(T) and Expected Outstanding

Use $T = 1$y loss distribution:

| $L$ | Probability $p$ |
|-----|-----------------|
| 0% | 0.60 |
| 2% | 0.25 |
| 5% | 0.10 |
| 10% | 0.05 |

From Example 1, $\mathrm{TL}(L)$ for $[3, 7]$ is: $0, 0, 2\%, 4\%$.

**Compute expected tranche loss:**

$$\mathrm{ETL}(T) = 0.60(0) + 0.25(0) + 0.10(0.02) + 0.05(0.04) = 0.002 + 0.002 = 0.004.$$

So $\mathrm{ETL}(T) = 0.4\%$ of portfolio.

**Expected outstanding:**

$$\mathrm{EON}(T) = W - \mathrm{ETL}(T) = 0.04 - 0.004 = 0.036.$$

So expected outstanding is $3.6\%$ of portfolio = USD 3.6mm.

---

### Example 3: Compute PV_prot from ETL(t_i) Increments on the Time Grid

Assume (toy) linear time scaling of ETL:

$$\mathrm{ETL}(t_i) = t_i \cdot \mathrm{ETL}(1).$$

From Example 2, $\mathrm{ETL}(1) = 0.004$.

Thus:
- $\mathrm{ETL}(0) = 0$
- $\mathrm{ETL}(0.25) = 0.001$
- $\mathrm{ETL}(0.50) = 0.002$
- $\mathrm{ETL}(0.75) = 0.003$
- $\mathrm{ETL}(1.00) = 0.004$

Each increment: $\Delta\mathrm{ETL}_i = 0.001$.

**Protection leg PV:**

$$\mathrm{PV}_{\mathrm{prot}} \approx F \sum_{i=1}^{4} Z(t_i) \Delta\mathrm{ETL}_i = 100\mathrm{mm} \times 0.001 \times (0.990 + 0.985 + 0.980 + 0.975).$$

Sum $Z$: $3.930$. So:

$$\boxed{\mathrm{PV}_{\mathrm{prot}} = 100\mathrm{mm} \times 0.00393 = USD 393{,}000}$$

---

### Example 4: Compute PV_prem for Spread s Using Expected Outstanding on the Same Grid

Take contractual tranche spread $s = 100$ bp/year ($s_{\mathrm{dec}} = 0.01$).

Compute $\mathrm{EON}(t_i) = W - \mathrm{ETL}(t_i)$ with $W = 0.04$:

| $t_i$ | $\mathrm{EON}(t_i)$ |
|-------|-------------------|
| 0 | 0.0400 |
| 0.25 | 0.0390 |
| 0.50 | 0.0380 |
| 0.75 | 0.0370 |
| 1.00 | 0.0360 |

**Trapezoidal averages:**

| $i$ | $\overline{\mathrm{EON}}_i$ |
|-----|---------------------------|
| 1 | $(0.0400 + 0.0390)/2 = 0.0395$ |
| 2 | $(0.0390 + 0.0380)/2 = 0.0385$ |
| 3 | $(0.0380 + 0.0370)/2 = 0.0375$ |
| 4 | $(0.0370 + 0.0360)/2 = 0.0365$ |

**Premium leg PV:**

$$\mathrm{PV}_{\mathrm{prem}} = F \cdot \frac{s}{10{,}000} \sum_{i=1}^{4} \Delta_i Z(t_i) \overline{\mathrm{EON}}_i.$$

Compute each term (using $F \cdot s/10{,}000 = 100\mathrm{mm} \cdot 0.01 = 1\mathrm{mm}$):

| $i$ | Calculation | Result |
|-----|-------------|--------|
| 1 | $1\mathrm{mm} \cdot 0.25 \cdot 0.990 \cdot 0.0395$ | USD 9,776.25 |
| 2 | $1\mathrm{mm} \cdot 0.25 \cdot 0.985 \cdot 0.0385$ | USD 9,475.63 |
| 3 | $1\mathrm{mm} \cdot 0.25 \cdot 0.980 \cdot 0.0375$ | USD 9,187.50 |
| 4 | $1\mathrm{mm} \cdot 0.25 \cdot 0.975 \cdot 0.0365$ | USD 8,898.38 |

**Total:**

$$\boxed{\mathrm{PV}_{\mathrm{prem}} = USD 37{,}337.76}$$

---

### Example 5: Solve Par Spread s_star That Sets PV about 0

For long protection:

$$\mathrm{PV} = \mathrm{PV}_{\mathrm{prot}} - \mathrm{PV}_{\mathrm{prem}}.$$

Since $\mathrm{PV}_{\mathrm{prem}}$ is linear in $s$, define the "annuity per 1bp":

$$\mathrm{PV01}_{\mathrm{annuity}} = F \cdot \frac{1}{10{,}000} \sum_{i=1}^{4} \Delta_i Z(t_i) \overline{\mathrm{EON}}_i.$$

From Example 4, $\mathrm{PV}_{\mathrm{prem}}(100\mathrm{bp}) = USD 37{,}337.76$, so:

$$\mathrm{PV01}_{\mathrm{annuity}} = USD 37{,}337.76 / 100 = USD 373.3776 \mathrm{ per bp}.$$

**Par spread solves:**

$$0 = \mathrm{PV}_{\mathrm{prot}} - s^{\star} \cdot \mathrm{PV01}_{\mathrm{annuity}} \quad\Rightarrow\quad s^{\star} = \frac{\mathrm{PV}_{\mathrm{prot}}}{\mathrm{PV01}_{\mathrm{annuity}}}.$$

Using $\mathrm{PV}_{\mathrm{prot}} = USD 393{,}000$:

$$\boxed{s^{\star} = \frac{393{,}000}{373.3776} = 1052.6 \mathrm{ bp}}$$

**Interpretation:**
- If contractual spread $s \lt s^{\star}$, long protection has positive PV (cheap premium).
- If $s \gt s^{\star}$, long protection has negative PV (expensive premium).

---

### Example 6: Tranche PV01: Bump s by +/-1bp and Compute PV01 via Central Difference

Use PV at $s = 100$bp (Examples 3–4):

$$\mathrm{PV}(100) = 393{,}000 - 37{,}337.76 = 355{,}662.24.$$

Because premium leg PV scales linearly with $s$, we can compute:

- $\mathrm{PV}(101) = \mathrm{PV}(100) - 1 \times \mathrm{PV01}_{\mathrm{annuity}} = 355{,}662.24 - 373.3776 = 355{,}288.86$
- $\mathrm{PV}(99) = \mathrm{PV}(100) + 373.3776 = 356{,}035.62$

**Central-difference PV01:**

$$\mathrm{PV01} \approx \frac{\mathrm{PV}(101) - \mathrm{PV}(99)}{2} = \frac{355{,}288.86 - 356{,}035.62}{2} = -373.38 \mathrm{ USD /bp}.$$

**Sign check:** negative for long protection (higher contractual spread means paying more premium). ✓

---

### Example 7: Compare PV01 Across Two Tranches with Different Attachments (Equity vs Senior)

Use the same loss distribution as Example 2 and same time-grid assumptions.

#### (a) Equity Tranche $[0, 3]$: $W = 0.03$

Compute $\mathrm{TL}(L) = \min(L, 0.03)$:

| $L$ | $\mathrm{TL}(L)$ |
|-----|----------------|
| 0% | 0 |
| 2% | 2% |
| 5% | 3% (cap) |
| 10% | 3% (cap) |

**Expected loss:**

$$\mathrm{ETL}(1) = 0.60(0) + 0.25(0.02) + 0.10(0.03) + 0.05(0.03) = 0.005 + 0.003 + 0.0015 = 0.0095.$$

So $\mathrm{EON}(1) = 0.03 - 0.0095 = 0.0205$.

Assume linear ETL in time:
- $\mathrm{ETL}(0.25) = 0.002375 \Rightarrow \mathrm{EON} = 0.027625$
- $\mathrm{ETL}(0.50) = 0.00475 \Rightarrow \mathrm{EON} = 0.02525$
- $\mathrm{ETL}(0.75) = 0.007125 \Rightarrow \mathrm{EON} = 0.022875$

Compute $\sum \Delta Z \overline{\mathrm{EON}}$:

| Term | Calculation | Result |
|------|-------------|--------|
| 1 | $0.25 \cdot 0.990 \cdot (0.03 + 0.027625)/2$ | 0.007131094 |
| 2 | $0.25 \cdot 0.985 \cdot (0.027625 + 0.02525)/2$ | 0.006510234 |
| 3 | $0.25 \cdot 0.980 \cdot (0.02525 + 0.022875)/2$ | 0.005895313 |
| 4 | $0.25 \cdot 0.975 \cdot (0.022875 + 0.0205)/2$ | 0.005286328 |

Sum $= 0.024822969$.

Thus PV01 annuity per bp:

$$\mathrm{PV01}_{\mathrm{equity}} = F \cdot \frac{1}{10{,}000} \cdot 0.024822969 = 100\mathrm{mm}/10{,}000 \cdot 0.024822969 = USD 248.23/\mathrm{bp}.$$

#### (b) Senior Tranche $[15, 30]$: $W = 0.15$

Under loss scenarios up to 10%, tranche never attaches $\Rightarrow$ $\mathrm{ETL}(t) = 0$ $\Rightarrow$ $\mathrm{EON}(t) = 0.15$ constant.

Compute $\sum \Delta Z \overline{\mathrm{EON}} = 0.15 \sum_{i=1}^{4} 0.25 Z(t_i)$:

$$= 0.15 \cdot 0.25(0.990 + 0.985 + 0.980 + 0.975) = 0.15 \cdot 0.9825 = 0.147375.$$

Thus:

$$\mathrm{PV01}_{\mathrm{senior}} = 10{,}000 \cdot 0.147375 = USD 1{,}473.75/\mathrm{bp}.$$

#### Conclusion (Numerical)

$$\boxed{|\mathrm{PV01}_{\mathrm{equity}}| \lt |\mathrm{PV01}_{\mathrm{mezz}}| \lt |\mathrm{PV01}_{\mathrm{senior}}|}$$

This matches the qualitative pattern that tranche RPV01 is lowest for equity and rises for more senior tranches (with caveats).

---

### Example 8: Correlation Shock (Toy): Low vs High Dependence, Compute ETL and PV Change for a Senior Tranche

We use tranche $[15, 30]$ (senior) to highlight tail sensitivity.

**Setup:**
- Maturity $T = 1$, same discount factors and grid.
- Contractual spread $s = 50$ bp.

#### Dependence Setting A (Low Tail / Low "Correlation")

Use loss distribution:
- $L = 0\%$ (0.60), $L = 2\%$ (0.25), $L = 5\%$ (0.10), $L = 10\%$ (0.05)

Since all losses are below 15%, tranche never attaches:
- $\mathrm{ETL}_A(1) = 0$
- $\mathrm{PV}_{\mathrm{prot},A} = 0$
- $\mathrm{PV01}_A = 1{,}473.75$ USD /bp (from Example 7(b))

**Premium PV:**

$$\mathrm{PV}_{\mathrm{prem},A} = s \cdot \mathrm{PV01}_A = 50 \cdot 1{,}473.75 = USD 73{,}687.50$$

So long-protection PV:

$$\mathrm{PV}_A = 0 - 73{,}687.50 = -USD 73{,}687.50.$$

#### Dependence Setting B (Higher Tail / Higher "Correlation")

Keep the same mean portfolio loss as setting A but add a tail state beyond 15%:

| $L$ | Probability |
|-----|-------------|
| 0% | 0.83 |
| 5% | 0.12 |
| 25% | 0.05 |

**Mean check:**

$$0 \cdot 0.83 + 0.05 \cdot 0.12 + 0.25 \cdot 0.05 = 0.006 + 0.0125 = 0.0185,$$

same as setting A's mean (1.85%).

**Compute tranche loss for $[15, 30]$:**
- At $L = 25\%$: $\mathrm{TL} = 25\% - 15\% = 10\% = 0.10$.
- Otherwise 0.

So:

$$\mathrm{ETL}_B(1) = 0.05 \cdot 0.10 = 0.005.$$

Assume linear in time: $\Delta\mathrm{ETL}_i = 0.005/4 = 0.00125$.

**Protection PV:**

$$\mathrm{PV}_{\mathrm{prot},B} = 100\mathrm{mm} \cdot 0.00125 \cdot (0.990 + 0.985 + 0.980 + 0.975) = 100\mathrm{mm} \cdot 0.0049125 = USD 491{,}250.$$

**Premium PV01** (recompute with declining EON):

$W = 0.15$, $\mathrm{EON}(1) = 0.15 - 0.005 = 0.145$.

Using the computed sum $\sum \Delta Z \overline{\mathrm{EON}} = 0.1449265625$ (see derivation in analysis), PV01:

$$\mathrm{PV01}_B = 10{,}000 \cdot 0.1449265625 = USD 1{,}449.27/\mathrm{bp}.$$

**Premium PV at $s = 50$bp:**

$$\mathrm{PV}_{\mathrm{prem},B} = 50 \cdot 1{,}449.27 = USD 72{,}463.28.$$

Thus:

$$\mathrm{PV}_B = 491{,}250 - 72{,}463.28 = USD 418{,}786.72.$$

#### PV Change Due to Dependence Shift

$$\boxed{\Delta\mathrm{PV} = \mathrm{PV}_B - \mathrm{PV}_A = 418{,}786.72 - (-73{,}687.50) = USD 492{,}474.22}$$

**Interpretation:** A "pure dependence/tail" increase (with mean loss held fixed) massively impacts senior tranche PV.

---

### Example 9: Correlation Delta: Compute PV at rho +/- Delta rho and Report CorrDelta

We interpret settings A and B as $\rho - \Delta\rho$ and $\rho + \Delta\rho$ around a midpoint $\rho$.

Let:
- $\rho - \Delta\rho = 0.20 \Rightarrow \mathrm{PV}(0.20) = \mathrm{PV}_A = -73{,}687.50$
- $\rho + \Delta\rho = 0.40 \Rightarrow \mathrm{PV}(0.40) = \mathrm{PV}_B = 418{,}786.72$

Then midpoint $\rho = 0.30$, $\Delta\rho = 0.10$.

**Compute CorrDelta:**

$$\mathrm{CorrDelta} \approx \frac{\mathrm{PV}(0.40) - \mathrm{PV}(0.20)}{2 \cdot 0.10} = \frac{418{,}786.72 - (-73{,}687.50)}{0.20} = \frac{492{,}474.22}{0.20} = USD 2{,}462{,}371.10 \mathrm{ per unit } \rho.$$

**Convert to Corr01 (1% absolute bump):**

Corr01 is approximately USD 24,623.71 per 1% absolute correlation bump.

This corresponds to Corr01 as the PV change for a 1% absolute increase in correlation.

---

### Example 10: Single-Name Default Scenario: Compute Portfolio Loss Increment, Tranche Loss Increment, PV Change Proxy

**Portfolio:** 100 equal names, each USD 1mm. Total $F = USD 100$mm.

**Default event:**
- One name defaults, recovery $R = 40\%$ $\Rightarrow$ LGD $= 60\%$.
- Loss dollars: `0.60 * USD 1mm = USD 0.60mm`.

**Loss fraction increment:**

$$\Delta L = 0.60\mathrm{mm} / 100\mathrm{mm} = 0.006 = 0.6\%.$$

**Assume realized portfolio loss before default:** $L^- = 2.8\% = 0.028$. After default:

$$L^+ = 0.028 + 0.006 = 0.034 = 3.4\%.$$

**Tranche:** $[3, 7]$, $A = 0.03$, $W = 0.04$.

**Compute tranche loss before:**

$$\mathrm{TL}(L^-) = \min(\max(0.028 - 0.03, 0), 0.04) = 0.$$

**After:**

$$\mathrm{TL}(L^+) = \min(\max(0.034 - 0.03, 0), 0.04) = 0.004 = 0.4\%.$$

**So tranche loss increment:**

$$\Delta\mathrm{TL} = 0.004 - 0 = 0.004.$$

**Dollar loss payment on tranche:**

$$\boxed{G = \Delta\mathrm{TL} \cdot F = 0.004 \times 100\mathrm{mm} = USD 0.40\mathrm{mm}}$$

#### PV Change Proxy

- Immediate protection payment to long protection is $+USD 0.40$mm.
- Remaining outstanding tranche notional decreases from `W*F = 0.04*100mm = USD 4.0mm` to `(W-DeltaTL)*F = 0.036*100mm = USD 3.6mm`.
- Premium leg PV will drop (because premium is paid on outstanding); protection leg PV also drops (less remaining protection).

A book-consistent way to capture the total jump is VOD, which includes repricing after the default and adding/subtracting $G$.

---

### Example 11: Clustered Default Scenario: k Names Default Simultaneously; Compute Tranche Loss and PV Impact

Use same portfolio and tranche.

**Cluster event:**
- $k = 5$ defaults simultaneously, each with $R = 40\%$.
- Total loss dollars = `5 * 0.60mm = 3.0mm`.
- $\Delta L = 3.0/100 = 0.03 = 3\%$.

**Assume $L^- = 2.8\% = 0.028$. Then $L^+ = 0.058 = 5.8\%$.**

**Tranche loss:**

$$\mathrm{TL}(L^-) = 0, \quad \mathrm{TL}(L^+) = \min(\max(0.058 - 0.03, 0), 0.04) = 0.028 = 2.8\%.$$

**So loss payment:**

$$\boxed{G = 0.028 \times 100\mathrm{mm} = USD 2.8\mathrm{mm}}$$

**Remaining tranche notional after the jump:**

$$(0.04 - 0.028) \times 100\mathrm{mm} = 0.012 \times 100\mathrm{mm} = USD 1.2\mathrm{mm}.$$

#### Comparison to Example 10

| Scenario | Loss Payment $G$ |
|----------|------------------|
| Single-name default | USD 0.40mm |
| Cluster of 5 defaults | USD 2.8mm |

This is the nonlinear "clustering amplification" of tranche jump exposure.

---

### Example 12: Recovery/Final Price Sensitivity: Vary Recovery and Compute Tranche Payout Differences

Single-name default with $L^- = 2.8\%$ and face value USD 1mm.

**Compute $G$ for different recoveries:**

#### Case R = 20%

- LGD = 80%, loss dollars = USD 0.80mm → $\Delta L = 0.008 = 0.8\%$
- $L^+ = 3.6\%$ $\Rightarrow$ $\mathrm{TL}(L^+) = 3.6 - 3.0 = 0.6\%$
- `G = 0.006 * 100mm = USD 0.60mm`

#### Case R = 40%

From Example 10: $G = USD 0.40$mm

#### Case R = 60%

- LGD = 40%, loss dollars = USD 0.40mm → $\Delta L = 0.004 = 0.4\%$
- $L^+ = 3.2\%$ $\Rightarrow$ $\mathrm{TL}(L^+) = 0.2\%$
- `G = 0.002 * 100mm = USD 0.20mm`

**Sensitivity (finite difference):**

$$\frac{\Delta G}{\Delta R} \approx \frac{0.20\mathrm{mm} - 0.60\mathrm{mm}}{0.60 - 0.20} = \frac{-0.40\mathrm{mm}}{0.40} = -1.00\mathrm{mm per 1.0 recovery}.$$

**So per 1% recovery:**

Approximately \(-0.01\) mm, i.e., USD 10,000 lower PV per +1% recovery.

---

### Example 13: Hedge PV01 with Another Tranche (PV01 Matching) and Validate Under Small Spread Moves

We hedge contractual tranche-spread PV01 (quote PV01) of $[3, 7]$ long protection using $[0, 3]$ short protection.

**From Examples:**
- $\mathrm{PV01}_{3-7}^{\mathrm{(long prot)}} = -USD 373.38/\mathrm{bp}$ (Example 6).
- $\mathrm{PV01}_{0-3}^{\mathrm{(long prot)}} = -USD 248.23/\mathrm{bp}$ (Example 7(a)).
- Therefore $\mathrm{PV01}_{0-3}^{\mathrm{(short prot)}} = +USD 248.23/\mathrm{bp}$.

Let hedge scale be $h$ (multiplier of notional exposure of the $0$–$3$ tranche relative to the base USD 100mm portfolio in this toy setup). Solve:

$$-373.38 + h \cdot 248.23 = 0 \quad\Rightarrow\quad h = \frac{373.38}{248.23} = 1.504.$$

#### Validation Under Small Spread Move

Assume both tranche contractual spreads increase by `Delta s = +5bp`.

**Target P&L:**

$$\Delta\mathrm{PV}_{3-7} \approx -373.38 \times 5 = -USD 1{,}866.9.$$

**Hedge P&L:**

$$\Delta\mathrm{PV}_{\mathrm{hedge}} \approx h \cdot (+248.23) \times 5 = 1.504 \times 1{,}241.15 = +USD 1{,}867.0.$$

**Net:**

$$\boxed{\Delta\mathrm{PV}_{\mathrm{net}} \approx +0.1 \mathrm{ (rounding)}}$$

So the position is PV01-neutral to small tranche spread quote moves in this toy setting.

---

### Example 14: Hedge Failure Under Clustering: Apply Clustered Default Scenario to the PV01-Hedged Portfolio

Use the PV01-neutral portfolio from Example 13:
- Long protection $[3, 7]$ on USD 100mm portfolio.
- Short protection $[0, 3]$ scaled by $h = 1.504$.

**Now apply a clustered default shock from no prior losses:**
- $k = 8$ defaults, each USD 1mm face, recovery 40%.
- Each loss = USD 0.60mm → total loss = `8 * 0.60 = USD 4.8mm`.
- Portfolio loss fraction jump: $\Delta L = 4.8/100 = 4.8\%$.

**Compute tranche loss increments from $L^- = 0$ to $L^+ = 4.8\%$:**

#### Mezz Tranche [3, 7]

$$\mathrm{TL}_{3-7}(4.8\%) = \min(4.8\% - 3.0\%,\; 4.0\%) = 1.8\%.$$

Dollar loss payment to long protection:

$$G_{3-7} = 1.8\% \times 100\mathrm{mm} = USD 1.8\mathrm{mm}.$$

#### Equity Tranche [0, 3]

$$\mathrm{TL}_{0-3}(4.8\%) = \min(4.8\%, 3.0\%) = 3.0\%.$$

Dollar loss payment for short protection position (a cost):

$$G_{0-3} = 3.0\% \times 100\mathrm{mm} = USD 3.0\mathrm{mm}.$$

Scaled by $h = 1.504$, hedge pays:

$$h \cdot G_{0-3} = 1.504 \times 3.0 = USD 4.512\mathrm{mm}.$$

#### Net Jump P&L (Ignoring MTM Repricing of Remaining Legs for Simplicity)

$$\boxed{\Delta\mathrm{PV}_{\mathrm{jump}} \approx +1.8 - 4.512 = -USD 2.712\mathrm{mm}}$$

#### Interpretation

The portfolio was PV01-neutral for small tranche spread quote moves, but suffered a large loss under a clustered default event.

**This is the core message: first-order PV01 hedging does not immunize jump/clustering risk.**

---

## 9. Practical Notes

### 9.1 Risk Report Glossary (Desk-Ready Definitions)

| Measure | Bump | Definition | Interpretation |
|---------|------|------------|----------------|
| **Tranche PV01** | Contractual tranche spread $s$ by +1bp | $\mathrm{PV}(s+1) - \mathrm{PV}(s)$ (USD/bp) or central difference | Premium-leg annuity exposure |
| **Correlation delta / Corr01** | Correlation parameter $\rho$ by +1% absolute | $\mathrm{Corr01} = V(\rho + 0.01) - V(\rho)$ | In a one-factor Gaussian copula parameterization, $\rho = \beta^2$ |
| **JTD / VOD** | Immediate default of a name (or set of names), with recovery $R$ | Total PV change under the default-event scenario (MTM reprice plus any immediate loss payment $F\cdot G$ if tranche is hit) | Discrete jump exposure |

---

### 9.2 Common Pitfalls

1. **Mixing quote regimes** (running spread vs upfront + fixed coupon) in one PV01 report.
   - Example: an equity tranche may be quoted as upfront + fixed coupon while other tranches are quoted as running spread only.

2. **Treating an implied "correlation number" as structural.** Correlation is a model/market parameter that can move; do not assume it is constant.

3. **Ignoring recovery/final price uncertainty** in jump scenarios (VOD depends on $R_0$).

4. **Relying on PV01 hedges for jump/clustering risk** (Example 14 shows failure).

5. **Model risk from loss distribution assumptions:**
   - Gaussian copula tail independence vs $t$-copula tail dependence matters for extremes.
   - Base correlation interpolation artifacts for non-standard strikes; linear interpolation can induce arbitrage-like behavior.

---

### 9.3 Verification Tests (Quick Sanity Suite)

| Test | Expected Result |
|------|-----------------|
| PV01 scaling | Double notional $F$ $\Rightarrow$ PV01 doubles |
| ETL bounds | $0 \leq \mathrm{ETL}(t) \leq W$, and $\mathrm{ETL}(t)$ nondecreasing in $t$ |
| EON bounds | $0 \leq \mathrm{EON}(t) \leq W$, nonincreasing in $t$ |
| Correlation toy sanity | Higher dependence should increase tail mass; senior tranche PV should rise in the tail-heavy setting (Examples 8–9) |
| Hedge validation | PV01-neutral portfolios show small DeltaPV under small spread shocks (Example 13), but not under default jumps (Example 14) |

---

## Summary

1. Tranche PV and risk are driven by the portfolio loss process $L(t)$ passed through the nonlinear tranche loss function $\mathrm{TL}(L)$ (attachment/detachment clipping).
2. The key valuation objects are $\mathrm{ETL}(t)$ (expected tranche loss) and $\mathrm{EON}(t)$ (expected outstanding): protection PV is driven by changes in $\mathrm{ETL}$, while premium PV is driven by discounted expected outstanding.
3. The tranche survival curve $Q(t)=\mathrm{EON}(t)/W$ turns the premium leg into a CDS-style “risky annuity” object, so many CDS plumbing checks (units, scaling, par-spread logic) apply.
4. **Contractual tranche PV01** bumps the contractual running premium $s$ by 1bp (1bp = $10^{-4}$ in decimal) and reprices; units are USD/bp and the sign is typically negative for long protection.
5. **Underlying spread risk** (systemic/idiosyncratic deltas and DV01-style measures) instead bumps issuer CDS curves and rebuilds the loss distribution; do not mix this with contractual tranche PV01 in one “DV01” number.
6. Correlation (dependence) affects the *shape* of the loss distribution tail; equity and senior tranches can have opposite Corr01 signs, and the sign depends on tranche seniority and position direction.
7. Default events are discontinuous: PV01-matched hedges can still lose money when defaults arrive or cluster, so jump/scenario risk belongs alongside small-move sensitivities.
8. A tranche risk report is only interpretable if it states the **bump object**, **bump size**, **units**, and **sign convention** for PV01/Corr01/VOD (and the quote regime: “running spread” vs “upfront + fixed coupon”).

## Key Concepts

| Concept | Definition | Why It Matters |
|---|---|---|
| Portfolio loss $L(t)$ | Cumulative portfolio loss fraction by $t$ (unitless) | The state variable that drives tranche payoffs |
| Tranche $[A,D]$, width $W$ | Attachment/detachment strikes; $W=D-A$ | Defines what slice of portfolio loss the tranche absorbs |
| Tranche loss $\mathrm{TL}(L)$ | $\min\{\max(L-A,0),W\}$ | Nonlinearity creates convexity, tail sensitivity, and jump risk |
| Tranche outstanding $\mathrm{ON}(L)$ | $W-\mathrm{TL}(L)$ | Notional on which running premium is paid |
| $\mathrm{ETL}(t)$ and $\mathrm{EON}(t)$ | $\mathrm{ETL}(t)=\mathbb{E}[\mathrm{TL}(L(t))]$, $\mathrm{EON}(t)=W-\mathrm{ETL}(t)$ | Protection PV depends on changes in $\mathrm{ETL}$; premium PV depends on $\mathrm{EON}$ |
| Tranche survival $Q(t)$ | $Q(t)=\mathrm{EON}(t)/W$ | Converts premium PV into a CDS-style risky annuity object |
| Quote regime | Running spread $s$, or upfront + fixed coupon (product-specific) | Misreading the quote regime is a common cause of PV01/settlement errors |
| Contractual tranche PV01 | $\mathrm{PV}(s+1\mathrm{bp})-\mathrm{PV}(s)$ (USD/bp) | Sensitivity to the tranche’s contractual running premium quote |
| Systemic vs idiosyncratic spread risk | Reprice after bumping all issuer curves vs one issuer curve | Explains why an index hedge can miss name-specific dispersion risk |
| Corr01 | $V(\rho+0.01)-V(\rho)$ (USD per 1% absolute) | Measures dependence/tail exposure, not “spread” exposure |
| VOD / JTD | PV change under a default-event scenario, including any immediate loss payment $G$ | Captures discontinuous jump risk that PV01 cannot hedge |
| Tail dependence | Probability of joint extremes persisting deep into the tail | Senior tranches live in the far tail, so tail dependence is first-order model risk |

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $F$ | Portfolio notional | USD |
| $[A,D]$, $W$ | Attachment/detachment, width $W=D-A$ | Fractions of $F$ (unitless) |
| $L(t)$ | Portfolio cumulative loss fraction | unitless |
| $\mathrm{TL}(L)$, $\mathrm{ON}(L)$ | Tranche loss / outstanding (as fraction of $F$) | unitless |
| $\mathrm{ETL}(t)$, $\mathrm{EON}(t)$ | Expected tranche loss / expected outstanding (as fraction of $F$) | unitless |
| $Q(t)$ | Tranche survival fraction, $Q(t)=\mathrm{EON}(t)/W$ | unitless |
| $Z(t)$ | Discount factor | unitless |
| $s$ | Contractual running premium quote | bp/year; $s_{\mathrm{dec}}=s/10{,}000$ |
| $\mathrm{PV01}_s$ | Contractual tranche PV01 | USD per 1bp bump to $s$; signed (typically negative for long protection) |
| $\rho$ | Dependence parameter | unitless; Corr01 uses a $+0.01$ absolute bump |
| $\mathrm{Corr01}$ | Correlation 01 | USD per $+1\%$ absolute bump to $\rho$ |
| $R$ | Recovery rate | unitless; LGD $=1-R$ |
| $G$ | Immediate loss payment in a default-event scenario | USD; received by long protection if the tranche is hit |

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is portfolio loss $L(t)$? | Cumulative portfolio loss as a fraction of portfolio notional $F$. |
| 2 | Define tranche width. | $W = D - A$. |
| 3 | Define tranche loss $\mathrm{TL}(L)$. | $\min(\max(L - A, 0), W)$. |
| 4 | Define tranche outstanding $\mathrm{ON}(L)$. | $W - \mathrm{TL}(L)$. |
| 5 | Define expected tranche loss $\mathrm{ETL}(t)$. | $\mathbb{E}[\mathrm{TL}(L(t))]$. |
| 6 | Define expected outstanding $\mathrm{EON}(t)$. | $W - \mathrm{ETL}(t)$. |
| 7 | What is tranche survival curve $Q(t)$? | $Q(t) = \mathrm{EON}(t)/W$. |
| 8 | What leg does $\mathrm{ETL}(t)$ drive? | Protection leg PV. |
| 9 | What leg does $\mathrm{EON}(t)$ drive? | Premium leg PV. |
| 10 | What is tranche-spread PV01? | PV change per 1bp change in contractual tranche spread $s$. |
| 11 | Units of PV01? | USD per bp. |
| 12 | Why is PV01 usually smaller for equity tranches? | Expected outstanding is lower due to earlier write-down. |
| 13 | What is Corr01? | PV change for a 1% absolute increase in correlation. |
| 14 | Why does correlation matter for senior tranches? | It changes tail probability of many defaults. |
| 15 | What is JTD for a tranche? | PV change under a specified default-event scenario. |
| 16 | What is VOD? | "Value-on-default," measuring PV impact of an immediate default. |
| 17 | How does VOD incorporate immediate loss payment $G$? | Under a loss-payment default scenario, include the cash payment (dollars: $F\cdot G$) and the post-default MTM of the reduced tranche; some systems report cash and MTM separately. |
| 18 | What is default clustering? | Multiple defaults occurring together/in stress states. |
| 19 | Define upper tail dependence coefficient $\lambda_u$. | Limiting conditional exceedance probability as quantile $\to 1$. |
| 20 | Gaussian copula tail dependence? | No tail dependence except at perfect correlation; e.g., lower tail dependence coefficient $\lambda_{\ell}=0$ for $\rho\neq 1$. |
| 21 | Why can PV01 hedges fail in defaults? | Defaults are discontinuous jumps, not small spread moves. |
| 22 | What is the "spread/PV01 term" in P&L decomposition? | P&L from small spread-like moves approximated by PV01-type sensitivities. |
| 23 | What is the "correlation term"? | P&L from dependence parameter shifts (Corr01/CorrDelta). |
| 24 | What is the "recovery term"? | P&L from recovery/LGD/final price changes. |
| 25 | What is residual/model risk? | P&L not explained by first-order terms; model/basis/liquidity effects. |
| 26 | In base correlation, which parameters drive a `[K1, K2]` tranche? | `rho(K1)` and `rho(K2)`. |
| 27 | What contradiction does base correlation introduce? | Different base tranches assign different correlations to same portfolio. |
| 28 | Why can base correlation interpolation create issues? | Linear interpolation can generate arbitrage-like tranchelet spreads. |
| 29 | What's the simplest PV01 hedge ratio formula? | $n_H = -\mathrm{PV01}_T / \mathrm{PV01}_H$. |
| 30 | What is a "pure correlation move" scenario? | Tranche price changes with index spreads held fixed. |
| 31 | What instruments hedge tranche correlation risk in this framework? | Other tranches (correlation products). |
| 32 | What is "complete the capital structure"? | Combine tranches across capital structure to reduce net correlation exposure. |
| 33 | Why is complete-capital-structure not perfectly correlation neutral? | Premium leg correlation exposures don't offset exactly. |
| 34 | What's a key proxy for clustering/tail risk? | Multi-default clustered default scenario ($k$-name default). |
| 35 | What should always accompany tranche risk numbers? | Clear definition of bumps, parameterization, and scenario assumptions. |
| 36 | What is systemic delta (issuer-curve bump)? | CDS hedge notional that offsets tranche PV change under a 1bp parallel bump to all issuer curves. |
| 37 | What is idiosyncratic delta (single-name bump)? | CDS hedge notional in a specific name $i$ that offsets the tranche PV change under a 1bp bump to that name. |
| 38 | How do systemic and idiosyncratic deltas compare for equity tranches? | They can differ materially; in practice the “right” hedge often lies between the pure systemic and pure idiosyncratic extremes and depends on regime/portfolio heterogeneity. |
| 39 | Where should the hedge ratio lie in practice? | Often between the pure systemic and pure idiosyncratic extremes, depending on whether you expect market-wide moves or dispersion. |
| 40 | What is RPV01 for a tranche? | Risky PV01 — the discounted expected outstanding that determines premium leg sensitivity. |
| 41 | Why is RPV01 lower for equity tranches? | Expected early write-down reduces expected outstanding and thus premium duration. |
| 42 | What does the "hockey stick" shape of TL(L) create? | Nonlinear risk characteristics — PV01 hedges fail for large moves and defaults. |
| 43 | How does auction final price affect tranche settlement? | Loss = (1 - Final Price) × Defaulted Notional; lower final price means larger tranche loss increment. |
| 44 | What is the leverage ratio for a tranche? | Systemic delta / tranche notional — measures effective exposure amplification. |
| 45 | What is systemic gamma? | Second derivative of tranche value with respect to parallel portfolio spread moves: $\Gamma_s = \frac{\partial^2 V}{\partial S^2}(1\mathrm{bp})^2$. |
| 46 | What is idiosyncratic gamma? | Second derivative of tranche value with respect to a single name's spread: $\Gamma_i = \frac{\partial^2 V}{\partial S_i^2}(1\mathrm{bp})^2$. |
| 47 | What sign is systemic gamma for equity tranches? | Negative—convexity works against equity holders for large parallel spread moves. |
| 48 | What sign is systemic gamma for senior tranches? | Positive—senior tranches benefit from convexity in large market-wide moves (long volatility). |
| 49 | What is the Gaussian copula tail dependence coefficient (lower tail)? | Zero: $\lambda_{\ell}=0$ for $\rho\neq 1$. |
| 50 | What is the t-copula tail dependence formula (lower tail)? | $\lambda_{\ell}=2 F_{t, \nu+1}\left(-\sqrt{\frac{(\nu+1)(1-\rho)}{1+\rho}}\right)$ where $F_{t,\nu+1}$ is a $t$ CDF. |
| 51 | Why can Gaussian copula understate senior tranche risk? | Zero tail dependence means joint extreme events can be understated relative to tail-dependent dependence structures. |
| 52 | What does negative systemic gamma imply operationally? | Delta hedges become stale quickly in volatile parallel moves; frequent re-hedging can create a “bleed” via execution costs and convexity. |
| 53 | What is the hedging decision parameter $\alpha$ in the blended hedge framework? | Trader's view on fraction of spread variance that is systemic; $\alpha = 1$ means all parallel, $\alpha = 0$ means all idiosyncratic. |
| 54 | How do you compute systemic gamma via finite differences? | $\Gamma_s = V(S + 1\mathrm{bp}) - 2V(S) + V(S - 1\mathrm{bp})$. |
| 55 | What is the approximate gamma P&L for a 10bp spread move with systemic gamma = −USD 4,000? | $\frac{1}{2} \times (-4{,}000) \times 100 = -USD 200{,}000$ loss. |

---

## Mini Problem Set

1. (Compute) Tranche $[2\%,5\%]$: compute $\mathrm{TL}(L)$ for $L=\{0,1,3,6\}\%$.
2. (Compute) Premium PV (toy grid): $F=50$mm, $[A,D]=[3\%,7\%]$ so $W=4\%$, $s=200$bp, $Q(t_0,t_1,t_2)=\{1,0.95,0.90\}$, $Z(t_1,t_2)=\{0.99,0.98\}$, $\Delta(t_0,t_1,t_2)=\{0.5,0.5\}$. Compute $\mathrm{PV}_{\mathrm{prem}}$ using trapezoid on expected outstanding.
3. (Compute) Protection PV (toy grid): using $F=50$mm, $\mathrm{ETL}(t_0,t_1,t_2)=\{0,0.002,0.005\}$ and $Z(t_1,t_2)=\{0.99,0.97\}$, compute $\mathrm{PV}_{\mathrm{prot}}$.
4. (Compute) Par spread (toy): using Problems 2–3 and ignoring upfront, solve for `s_star` such that `PV_prot = PV_prem`.
5. (Compute) PV01 (central difference): `PV(s-1bp)=1.20mm` and `PV(s+1bp)=1.15mm`. Compute contractual tranche PV01.
6. (Compute) CorrDelta and Corr01: `PV(0.2)=-0.05mm` and `PV(0.4)=0.35mm`. Approximate CorrDelta around `0.3` with `Delta rho = 0.1`, and compute Corr01.
7. (Compute) Default-event loss payment: $F=500$mm, tranche $[3,7]$, realized loss jumps from $L^-=2.5\%$ to $L^+=3.15\%$. Compute the immediate loss payment $G$ to a long-protection holder (assume the tranche is hit).
8. (Concept) In one paragraph: why is contractual tranche PV01 not the same as “systemic DV01” to issuer spreads? Name the bumped object in each case.
9. (Desk) A book is PV01-neutral to contractual spread bumps, but a default causes a large loss. List two diagnostics/tests that would have caught the jump/clustering exposure.
10. (Concept) Define “upper tail dependence” in words and explain why Gaussian-copula tail independence is a model risk for senior tranches.

### Solution Sketches (Selected)

1. $A=2\%$, $W=3\%$. $\mathrm{TL}(0\%)=0\%$, $\mathrm{TL}(1\%)=0\%$, $\mathrm{TL}(3\%)=1\%$, $\mathrm{TL}(6\%)=3\%$ (capped at $W$).
2. `F*W*s_dec = 50mm*0.04*0.02 = USD 40,000` per year. Trapezoid sum is `sum_i Delta_i Z_i (Q_{i-1}+Q_i)/2 = 0.935875`. So `PV_prem ≈ 40,000*0.935875 = USD 37,435`.
3. `DeltaETL1=0.002`, `DeltaETL2=0.003`. `PV_prot = 50mm*[0.99(0.002)+0.97(0.003)] = USD 244,500`.
4. Using the annuity sum from (2): `PV_prem = F*W*s_dec*0.935875`. Solve `s_dec=244,500/(50mm*0.04*0.935875)=0.13065`, so `s_star ≈ 1306.5bp`.
5. `PV01 ≈ (1.15-1.20)/2 mm/bp = -0.025 mm/bp = -USD 25,000/bp`.
6. `CorrDelta ≈ (0.35-(-0.05))/0.2 = 2.0mm` per unit rho. `Corr01 ≈ 0.01*2.0mm = 0.02mm = USD 20,000` per +1% absolute.
7. Before: `TL(2.5%)=0`. After: `TL(3.15%)=min(3.15%-3%,4%)=0.15%`. So `G=0.15%*500mm=USD 0.75mm`.
8. Contractual PV01 bumps the tranche’s running premium $s$ holding the tranche loss/discount inputs fixed. “Systemic DV01” bumps issuer CDS curves (all names), rebuilds hazard rates and the tranche loss distribution, and reprices the tranche.
9. Example diagnostics: (i) compute VOD/JTD name-by-name and a $k$-name clustered-default scenario; (ii) run a default-event scenario on a “PV01-hedged” book to quantify the jump component not explained by PV01.

## References

- Dominic O’Kane, *Modelling Single-name and Multi-name Credit Derivatives*, “The Tranche Survival Curve”, “Tranche RPV01”, “Systemic Delta, DV01 and Leverage Ratio”, “Correlation 01”, “Value-on-Default (VOD)”, “Base Correlation”
- John C. Hull, *Options, Futures, and Other Derivatives*, “CDOs” (quote regimes: running spread vs upfront + fixed coupon)
- John C. Hull, *Risk Management and Financial Institutions*, “CDOs” (quote regimes; correlation language in the CDO worksheet context)
- Alexander J. McNeil, Rüdiger Frey, Paul Embrechts, *Quantitative Risk Management: Concepts, Techniques and Tools*, “Copulas and Dependence” (tail dependence; Gaussian copula asymptotic tail independence)
- David Ruppert, David S. Matteson, *Statistics and Data Analysis for Financial Engineering with R examples*, “Rank Correlation” (Gaussian vs $t$-copula tail dependence coefficients; limiting cases)
- Gianluca Fusai, Andrea Roncoroni, *Implementing Models in Quantitative Finance*, “Copulas” (tail dependence definitions and intuition)
