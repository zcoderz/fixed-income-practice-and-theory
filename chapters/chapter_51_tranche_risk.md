# Chapter 51: Tranche Risk — Tranche PV01, Correlation Risk, and Jump-to-Default Clustering

---

## Introduction

A trader hedges a mezzanine tranche position using an index CDS, matching spread DV01 to the penny. The next morning, a name in the portfolio defaults—and the P&L report shows a $3 million loss despite the "perfect" hedge. What went wrong?

The answer lies in a fundamental truth about tranche risk: **the sensitivities that matter most are not the ones that appear in a standard first-order risk report.** Single-name CDS and even index CDS respond to spread movements in roughly linear fashion. Tranches do not. The nonlinear mapping from portfolio losses to tranche losses—the "hockey stick" function that clips losses at attachment and caps them at detachment—creates risk characteristics that confound conventional hedging intuition.

O'Kane's treatment of synthetic tranche risk management (Ch 17) distinguishes two fundamentally different types of spread sensitivity: *systemic delta*, which measures exposure to parallel movements in all portfolio spreads, and *idiosyncratic delta*, which measures exposure to a single name widening in isolation. For a 0–3% equity tranche on a 125-name portfolio, these two numbers can differ by a factor of five. A trader who hedges only systemic risk leaves massive idiosyncratic exposure; one who hedges only idiosyncratic risk may be overhedged for market-wide moves. As O'Kane emphasizes, "the actual hedge will lie somewhere between the systemic and the idiosyncratic" depending on the trader's view of whether spread moves will be market-driven or name-specific.

This chapter develops the complete risk framework for synthetic tranches. Section 1 establishes the core definitions: tranche loss mapping, expected tranche loss, and the "tranche survival curve" that enables CDS-style valuation. Section 2 provides the practitioner's P&L decomposition—spread, correlation, jump-to-default, and recovery terms. Sections 3–5 develop each risk measure in detail: tranche PV01 and the systemic/idiosyncratic distinction (Section 3), correlation risk and the Corr01 measure (Section 4), and jump-to-default clustering with its tail dependence implications (Section 5). Section 6 presents the hedging map—what instruments address which risks, and crucially, why PV01 hedges fail under default clustering. Throughout, we draw on O'Kane's real-world risk report examples (Table 17.2) to ground the theory in desk practice.

---

## Conventions & Notation

- **Valuation perspective:** unless stated otherwise, PV and sensitivities are reported from the protection buyer (long protection) perspective.
- **Portfolio notional:** $F$ in USD.
- **Losses and tranche strikes:** expressed as fractions of portfolio notional. Example: $A = 3\% \Rightarrow A = 0.03$.
- **Discounting:** $Z(t)$ is the discount factor from time 0 to $t$ (unitless).
- **Spreads:** contractual running premium $s$ quoted in bp/year. Convert to decimal rate $s_{\text{dec}} = s / 10{,}000$.
- **Accrual fractions:** $\Delta(t_{i-1}, t_i)$ is the year fraction between payment dates (units: years).
- **Tranche survival curve:** $Q(t; A, D) \in [0, 1]$ is the expected surviving fraction of tranche notional (defined precisely below). This "tranche survival curve" is used to map tranche valuation to CDS-style analytics.
- **Correlation parameter:** when we use a single-parameter one-factor dependence knob, we write $\rho$ (unitless). In the one-factor Gaussian copula discussion in the tranche "correlation 01" definition, $\rho = \beta^2$.
- **Immediate default shock / VOD:** "value-on-default" (VOD) is an idiosyncratic jump measure defined by repricing after an immediate default (and adding/subtracting any immediate loss payment $G$).

---

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $F$ | Portfolio notional (USD) |
| $t \in [0, T]$ | Time; $T$ is tranche maturity |
| $L(t)$ | Portfolio cumulative loss fraction by time $t$ (unitless, in $[0, 1]$) |
| $[A, D]$ | Tranche attachment/detachment (fractions of portfolio); $W = D - A$ |
| $\text{TL}(L)$ | Tranche loss as a fraction of portfolio notional (unitless) |
| $\text{ON}(L)$ | Tranche outstanding as a fraction of portfolio notional (unitless) |
| $\text{ETL}(t)$ | Expected tranche loss fraction by $t$ (unitless) |
| $\text{EON}(t)$ | Expected outstanding fraction by $t$ (unitless); $\text{EON}(t) = W - \text{ETL}(t)$ |
| $Q(t; A, D)$ | Expected surviving fraction of tranche notional (unitless), i.e., "tranche survival curve"; $Q(t; A, D) = \text{EON}(t) / W$ |
| $Z(t)$ | Discount factor |
| $s$ | Contractual tranche spread (bp/year); $s_{\text{dec}} = s / 10{,}000$ |
| $\rho$ | Dependence (correlation) parameter; $\Delta\rho$ is a bump size |
| $\text{PV01}$ | PV sensitivity to a 1bp bump (defined precisely in Section 3) |
| $\text{Corr01}$ | PV change for a 1% absolute increase in correlation (as defined in the source) |
| $\text{JTD}$ | "Jump-to-default" for tranches, defined as PV change under a default-event scenario |
| $R$ | Recovery rate (unitless); $(1 - R)$ is loss-given-default (LGD) |

---

## 0. Setup

### Conventions Used in This Chapter

- All tranche strikes $[A, D]$ are expressed as fractions of portfolio notional $F$.
- Tranche width: $W = D - A$ (fraction of portfolio; unitless).
- Running premium $s$ is quoted in bp/year; $s_{\text{dec}} = s / 10{,}000$.
- Discounting uses $Z(t)$. (In practice: OIS/CSA, etc. I'm not sure which curve your desk uses; this depends on collateral, currency, and CSA.)
- Payment dates: $0 = t_0 < t_1 < \cdots < t_N = T$ with accruals $\Delta_i = \Delta(t_{i-1}, t_i)$.
- Perspective: PV is from protection buyer (long protection):

$$\text{PV} = \text{PV}_{\text{prot}} - \text{PV}_{\text{prem}} - \text{Upfront}.$$

(Source formulas in the references are often written for short protection; we flip signs consistently.)

---

## 1. Core Concepts (Definitions First)

### 1.1 Portfolio Loss and Tranche Loss Mapping

#### Formal Definition (Units Explicit)

Let $L(t) \in [0, 1]$ be the portfolio cumulative loss fraction by time $t$:

$$L(t) = \frac{\text{cumulative portfolio loss dollars by } t}{F}.$$

**Units:** unitless fraction of portfolio notional.

For tranche $[A, D]$ with width $W = D - A$, define **tranche loss** (fraction of portfolio) as:

$$\boxed{\text{TL}(L) = \min\{\max(L - A, 0),\; W\}}$$

**Units:** unitless fraction of portfolio notional.

Define **tranche outstanding** (fraction of portfolio) as:

$$\boxed{\text{ON}(L) = W - \text{TL}(L)}$$

**Units:** unitless fraction of portfolio notional.

#### Intuition

- $\text{TL}(L)$ is a "clipped" version of portfolio loss: nothing happens until attachment $A$, then tranche takes losses linearly until detachment $D$, then it is fully written down (loss $W$).
- $\text{ON}(L)$ is what remains of the tranche.

#### Trading/Risk Practice

Every risk measure for tranches ultimately flows through:

$$\text{portfolio loss distribution} \;\Rightarrow\; \text{tranche loss distribution} \;\Rightarrow\; \text{PV and Greeks}.$$

Nonlinearity ($\min/\max$) is why tranches exhibit strong tail and jump sensitivity.

---

### 1.2 Expected Tranche Loss and Expected Outstanding

#### Formal Definition (Units Explicit)

**Expected tranche loss:**

$$\text{ETL}(t) = \mathbb{E}[\text{TL}(L(t))].$$

**Units:** fraction of portfolio notional.

**Expected outstanding:**

$$\text{EON}(t) = \mathbb{E}[\text{ON}(L(t))] = W - \text{ETL}(t).$$

**Units:** fraction of portfolio notional.

**Tranche survival curve** (fraction of tranche notional outstanding):

$$\boxed{Q(t; A, D) = \frac{\text{EON}(t)}{W} \in [0, 1]}$$

This is the "expected tranche surviving percentage notional" used to value tranches via CDS-style formulas.

#### Intuition

- $\text{ETL}(t)$ is "how much of the tranche you expect to have lost by time $t$".
- $\text{EON}(t)$ is "expected remaining premium-paying notional".

#### Trading/Risk Practice

- $\text{ETL}(t)$ drives protection leg PV (expected loss payments).
- $\text{EON}(t)$ drives premium leg PV (expected running spread payments).

---

### 1.3 Premium Leg PV and Protection Leg PV (Chapter 49-Style, but Source-Backed Here)

#### Formal Definition

The source provides a CDS-style valuation representation for a tranche (written there for short protection) using $Q(t; A, D)$ and discounting $Z(t)$:

**Protection Leg PV** (per unit tranche notional in the source's convention):

$$\text{Protection Leg PV} = \int_0^T Z(s)\,(-dQ(s; A, D)).$$

**Tranche PV** (short protection, per unit tranche notional) is written as premium leg minus protection leg using a trapezoidal discretization:

$$V(A, D) = \frac{s_{\text{dec}}}{2} \sum_{i=1}^{N} \Delta_i Z(t_i)(Q(t_{i-1}) + Q(t_i)) \;-\; \int_0^T Z(s)\,(-dQ(s)).$$

The same source emphasizes an exact mapping between pricing a CDS and pricing a tranche once a tranche survival curve $Q$ is available: replace issuer survival by tranche survival and assume zero recovery in the CDS mapping.

#### Intuition

- Premium leg is "spread $\times$ risky annuity $\times$ expected outstanding".
- Protection leg is "discounted expected loss increments".

#### Trading/Risk Practice

Desk analytics frequently store/trade on objects like:
- $\text{ETL}(t)$ curves
- $Q(t)$ curves
- "tranche annuity / RPV01" objects (premium-leg PV per bp)

---

### 1.4 Tranche PV01 / Spread Risk (Quote Mechanics, $/bp)

#### Formal Definition (What Is Bumped, Units Explicit)

There are two distinct but commonly conflated "spread risks":

**1. Contractual tranche-spread PV01 (quote PV01)** — mechanical sensitivity to the contractual running premium $s$ on the tranche contract:

For a spread-quoted tranche, define:

$$\boxed{\text{PV01}_{\text{tranche spread}} = \frac{\partial \text{PV}}{\partial s} \times 1\text{bp}}$$

where $s$ is the contractual tranche spread quote (bp/year).

**Units:** USD per 1bp.

**2. Underlying portfolio spread risk (systemic/idiosyncratic spread deltas)** — PV sensitivity to issuer CDS curve moves. The primary source develops systemic vs idiosyncratic delta concepts for hedging tranche spread risk to issuer spreads (not the tranche contractual spread).

This chapter's "Tranche PV01" is item (1) unless explicitly stated otherwise.

#### Intuition

$\text{PV01}_{\text{tranche spread}}$ is basically the discounted expected outstanding notional: if the tranche is likely to be outstanding, PV01 is large; if likely to be quickly written down, PV01 is small.

#### How It Appears in Practice

Used to:
- compute/paraphrase par spread $s^*$ ("breakeven tranche spread"),
- convert PV moves into "spread moves",
- size hedges when hedging tranche spread quotes with other tranche contracts.

#### Quote Regime Caveat (Upfront + Running Premium)

Some tranche quotes are upfront + fixed coupon rather than pure running spread. For example, the reference notes that quotes are in bp except for a $0$–$3\%$ tranche where the quote equals an upfront percent of tranche principal plus 500 bp/year running premium.

I'm not sure what tranche quote regime you want assumed for each tranche in this chapter (index-vintage dependent). To be certain we'd need: the product (CDX/iTraxx), series, vintage, maturity, running coupon conventions, and premium accrual convention.

---

### 1.5 Correlation Risk (PV Sensitivity to Dependence)

#### Formal Definition

Correlation risk is PV sensitivity to a correlation/dependence parameter. In the one-factor Gaussian copula framing used in the source's risk report, "correlation 01" is:

$$\boxed{\text{Corr01} = V(\rho + 0.01) - V(\rho)}$$

i.e., the change in tranche value due to a 1% absolute increase in correlation (with $\rho = \beta^2$ in that notation).

A normalized "rho" is also defined as the change in value per unit correlation per unit tranche face value.

#### Intuition

- Correlation controls how often defaults arrive together (or, more broadly, how much probability mass is in the joint tail of portfolio loss).
- Changing correlation changes the shape of the portfolio loss distribution, not just its mean.

#### Trading/Risk Practice

Correlation is often an implied parameter extracted from tranche quotes, and it is not stable. The source explicitly warns that after hedging rate/spread risk, the remaining market risk is changes in correlation, and it should not be assumed fixed.

---

### 1.6 Jump-to-Default (JTD) / Default-Event Risk for Tranches

#### Formal Definition

Define tranche JTD under a specified default event scenario $\omega$ as:

$$\text{JTD}_\omega = \text{PV}_{\text{after}}(\omega) - \text{PV}_{\text{before}}.$$

The primary source uses **Value-on-Default (VOD)** to measure "the impact of an immediate default and the resulting change in value" for a tranche position.

In the case where the default generates an immediate tranche loss payment $G$, VOD is computed as:

$$\boxed{\text{VOD} = V'(t) - V(t) \pm G}$$

with "plus for long protection, minus for short protection."

#### Dependence on Recovery/Final Price Assumptions

The immediate loss jump depends on LGD $= (1 - R)$ and defaulted face value. The source's VOD steps explicitly include the defaulted face value $H_0$ and recovery $R_0$, and adjust attachment/detachment after default to reflect subordination consumption.

#### Trading/Risk Practice

JTD/VOD is crucial because PV01 hedges typically target small continuous moves, while defaults produce discrete jumps.

---

### 1.7 Jump-to-Default Clustering / Tail Risk

#### Formal Definition

"Clustering" means that defaults are not isolated; multiple defaults can occur in the same stress state or close in time.

In dependence language, a key notion is **tail dependence:** the tendency for variables to be jointly extreme.

QRM defines **upper tail dependence** as a limiting conditional exceedance probability:

$$\lambda_u = \lim_{q \to 1^-} P(X_2 > F_2^{\leftarrow}(q) \mid X_1 > F_1^{\leftarrow}(q)),$$

and similarly for lower tail dependence.

QRM shows the **Gaussian copula is asymptotically tail independent** ($\lambda = 0$ for $\rho < 1$).

#### Intuition

Senior tranches are exposed to rare joint-default states; clustering amplifies those states' probability.

#### Trading/Risk Practice

Clustering risk often appears as:
- large losses in "tail scenarios" despite benign average spread moves,
- poor hedge performance under jump events,
- strong sensitivity to model choice (Gaussian vs $t$-copula, mixture models, etc.).

#### Source-Backed Tranche-Specific Intuition

The tranche idiosyncratic-risk discussion notes that when an idiosyncratic beta is high, "default … will tend to be accompanied by lots of other defaults," which is precisely a clustering mechanism in a one-factor-plus-idiosyncratic setting.

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

I'm not sure how your desk defines recovery sensitivity (recovery of defaulted name? portfolio average recovery? auction final price conventions?). The source discusses recovery and its relationship with default rates in modeling loss distribution tails.

---

## 2. Risk Decomposition for Tranche Books (Organizing Backbone)

A practitioner decomposition is a bookkeeping identity for explaining daily P&L and sizing hedges:

$$\Delta\text{PV} \approx \underbrace{\Delta\text{PV}_{\text{Spread/PV01}}}_{\text{small spread-like moves}} + \underbrace{\Delta\text{PV}_{\text{Corr}}}_{\text{dependence shift}} + \underbrace{\Delta\text{PV}_{\text{JTD}}}_{\text{default jumps}} + \underbrace{\Delta\text{PV}_{\text{Rec/FinalPrice}}}_{\text{LGD/final price}} + \underbrace{\Delta\text{PV}_{\text{Residual}}}_{\text{model/basis/liquidity}}$$

Below is how to interpret each term economically and operationally.

---

### 2.1 Spread/PV01 Term (Small "Spread-Like" Moves)

#### Economic Meaning

Represents P&L from small changes in quoted tranche spread(s) and/or underlying credit spreads that shift the loss distribution slightly.

#### Operational Implementation

Two common measurements:
1. contractual tranche-spread PV01 (this chapter's Section 3),
2. underlying systemic/idiosyncratic spread risk (issuer curve bumps), treated in the source via systemic vs idiosyncratic delta and DV01-style measures.

#### Shock/Scenario

- "+5bp move in tranche spread" (quote move).
- "+1bp parallel move in issuer CDS curves" (systemic move) vs "single-name widening" (idiosyncratic move).

#### Hedges (Conceptual Mapping)

- **Quote PV01:** hedge with other tranche contracts (spread PV01 matching).
- **Underlying spread risk:** hedge with CDS index and/or baskets of single names; the source stresses that real markets exhibit a mixture of systemic and idiosyncratic moves, so hedging is not purely mechanical.

---

### 2.2 Correlation Term

#### Economic Meaning

Captures P&L due to changes in dependence (correlation, copula tail, skew), holding marginal default probabilities approximately fixed.

#### Operational Implementation

- **Correlation 01:** bump correlation and reprice.
- In a base-correlation framework, the tranche depends on $\rho(K_1)$ and $\rho(K_2)$; changes in the base correlation curve can create hedge slippage when spreads move (mapping effects).

#### Shock/Scenario

- "Index tranches reprice with index spreads fixed" (pure correlation move).
- Source describes an explicit procedure: hold index spreads fixed, bump standard tranche contractual spread by 1bp, rebuild base correlation curve, reprice to isolate correlation-only effects.

#### Hedges (Conceptual Mapping)

Hedging correlation requires other correlation products (other tranches). The source states the only hedging instruments are other tranches; exact offset would be selling the same tranche, which is often impractical.

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

The source notes empirical evidence of negative correlation between macro default rates and recovery rates and explains how this extends the upper tail of the loss distribution when defaults are high.

#### Auction Settlement Mechanics for Tranches

When a credit event occurs on a portfolio constituent, the settlement process determines the actual loss:

1. **Credit Event Declaration:** ISDA Determinations Committee confirms the credit event.

2. **Auction Process:** A CDS auction determines the final price for the defaulted reference obligation (see Chapter 40 for full auction mechanics).

3. **Final Price → Loss Calculation:**
   $$\text{Loss} = (1 - \text{Final Price}) \times \text{Defaulted Notional}$$

4. **Tranche Impact:** The portfolio loss $L$ increases, and tranche loss $\text{TL}(L)$ is recalculated.

5. **Settlement:** Protection seller pays $G = \Delta\text{TL} \times F$ to protection buyer if the tranche is hit.

#### Worked Example: Auction-Style Settlement

**Setup:**
- Portfolio: $F = \$500$mm across 100 names (\$5mm each)
- Tranche: 3–7% mezzanine
- Current portfolio loss: $L^- = 2.5\%$ (prior defaults)
- New credit event: Name XYZ defaults

**Auction outcome:** Final price = 35% (i.e., recovery = 35%)

**Step 1: Calculate loss increment**
$$\text{Loss}_{\text{XYZ}} = (1 - 0.35) \times \$5\text{mm} = \$3.25\text{mm}$$
$$\Delta L = \$3.25\text{mm} / \$500\text{mm} = 0.65\%$$

**Step 2: Update portfolio loss**
$$L^+ = 2.5\% + 0.65\% = 3.15\%$$

**Step 3: Calculate tranche loss before and after**
- Before: $\text{TL}(2.5\%) = \max(2.5\% - 3\%, 0) = 0$
- After: $\text{TL}(3.15\%) = \max(3.15\% - 3\%, 0) = 0.15\%$

**Step 4: Settlement payment**
$$G = 0.15\% \times \$500\text{mm} = \$0.75\text{mm}$$

Protection buyer receives $750,000.

**Key insight:** If the auction final price had been 50% (recovery = 50%), the loss increment would be $\Delta L = 0.5\%$, and $L^+ = 3.0\%$—exactly at attachment. The tranche loss would be zero, and no payment would occur despite the credit event.

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

$$\boxed{\text{PV01} \equiv \frac{\partial \text{PV}}{\partial s} \times 1\text{bp}}$$

- **Bump:** $s \mapsto s + 1$ bp (contractual tranche spread).
- **Units:** USD per 1bp.

Using the CDS-style premium leg representation (trapezoidal sum with tranche survival $Q$), PV is linear in $s$ in that representation, so PV01 is essentially the (negative) discounted expected outstanding "annuity" term.

---

### 3.2 Definition (Upfront + Coupon Quote Tranches)

If the tranche is quoted as upfront $U$ (percent of tranche principal) plus fixed running coupon $c$ (bp/year), then:

**Upfront sensitivity:**

$$\text{PV01}_{\text{upfront}} = \frac{\partial \text{PV}}{\partial U} \times 1\%,$$

**Units:** USD per 1% upfront.

**Coupon PV01** is still defined as $\frac{\partial \text{PV}}{\partial c} \times 1\text{bp}$.

The reference gives an example where the $0$–$3\%$ tranche is quoted as upfront percent plus 500 bp/year running, while other tranches are quoted in bp.

I'm not sure how to convert "1% upfront" to "bp-equivalent" for your specific tranche set without:
- the fixed coupon $c$,
- accrual convention on default,
- whether upfront is paid on tranche notional or remaining outstanding,
- settlement/standard docs assumptions.

---

### 3.3 Dependence of PV01 on ETL/EON and Discounting (Unit Checks)

For a long protection position, schematically:

$$\text{Premium leg} \propto s_{\text{dec}} \times \sum Z \Delta \times \text{expected outstanding}.$$

Therefore $|\text{PV01}|$ increases with expected outstanding and with discount factors (higher $Z$, longer maturity).

#### Unit Check

$\Delta$ in years, $Z$ unitless, expected outstanding in fraction of portfolio; multiply by $F$ gives USD notional base; multiply by 1bp $= 10^{-4}$ gives USD per bp. ✓

---

### 3.4 How PV01 Varies by Attachment (Equity vs Senior)

#### Source-Backed Qualitative Shape

The source's systemic risk report indicates tranche RPV01 is lowest for the riskiest equity tranche and rises for more senior tranches (with caveats for super-senior amortization).

#### O'Kane Table 17.2: Real-World RPV01 Data

From O'Kane's Chapter 17 risk report example for long protection positions on standard tranches. The reference portfolio has 125 credits, each \$10 million notional (total \$1,250 million), with an average spread of 60 bp and correlation ρ = β² = 25%:

| Tranche | Breakeven Spread (bp) | RPV01 | Systemic Delta (\$m) | Leverage Ratio | Corr01 (\$) |
|---------|----------------------|-------|---------------------|----------------|-------------|
| 0–3% (Equity) | 1,790.2 | 2.95 | 691 | 18.37 | −433,553 |
| 3–7% (Mezz) | 405.3 | 4.15 | 461 | 9.20 | +23,214 |
| 7–10% (Senior) | 148.4 | 4.39 | 170 | 4.52 | +78,274 |
| 10–15% (Senior) | 58.1 | 4.47 | 133 | 2.12 | +103,158 |
| 15–30% (Super-Sr) | 8.6 | 4.50 | 73 | 0.39 | +93,228 |

*Source: O'Kane Table 17.2, exact values*

**Key observations from Table 17.2:**
- **RPV01 increases with seniority:** 2.95 for equity → 4.50 for super-senior (1.5× increase). This reflects expected outstanding—equity tranches are expected to be written down early, reducing the premium leg duration.
- **Systemic delta decreases dramatically:** \$691m for equity → \$73m for super-senior (~9.5× decrease). The equity tranche captures roughly half the spread risk of the entire \$1,250m portfolio.
- **Leverage ratio falls from 18.37× (equity) to 0.39× (super-senior):** Equity tranches amplify portfolio spread moves; super-senior tranches de-lever them.
- **Corr01 sign flips:** Negative for equity (−\$433,553), positive for senior tranches. This confirms that equity is short correlation while senior is long correlation.

#### Intuition

- **Equity** is expected to be written down sooner → lower expected outstanding → lower PV01.
- **Senior** is expected to remain outstanding → higher PV01.

---

## 4. Correlation Risk (Why PV Changes When Dependence Changes)

### 4.1 Why Dependence Changes Tranche PV

Dependence changes the distribution of the number/timing of defaults and therefore the distribution of portfolio loss $L(t)$. The reference text (derivatives book) highlights:

> As default correlation increases, probability of "one or more defaults" can decline while probability of "many defaults" increases; junior protection becomes less valuable and more senior protection becomes more valuable.

That is the core mechanism:
- **Equity** cares about "at least some defaults" → correlation can reduce that likelihood (equity can become safer).
- **Senior** cares about "very many defaults" → correlation can increase that tail probability (senior becomes riskier).

---

### 4.2 Correlation Delta and Corr01 (Parameterization Clarity)

#### Primary Parameterization Used Here

We use a single-parameter $\rho$ (compound correlation / one-factor parameter) for clarity, consistent with the "correlation 01" definition in the source.

#### Definition

**Correlation 01 (Corr01):** PV change for a 1% absolute increase in correlation:

$$\boxed{\text{Corr01} = V(\rho + 0.01) - V(\rho)}$$

**Finite-difference correlation delta (symmetric):**

$$\text{CorrDelta} \approx \frac{V(\rho + \Delta\rho) - V(\rho - \Delta\rho)}{2\Delta\rho}$$

---

### 4.3 If Base Correlation Is Used (Only What Is Source-Supported)

The source explains the base correlation decomposition:

A $[K_1, K_2]$ tranche can be decomposed as a linear combination of two base (equity) tranches and is priced by assigning different correlations to $[0, K_1]$ and $[0, K_2]$.

**Expected loss relationship under base correlation:**

$$\mathbb{E}[L(T, K_1, K_2)] = \frac{\mathbb{E}_{\rho(K_2)}[\min(L(T), K_2)] - \mathbb{E}_{\rho(K_1)}[\min(L(T), K_1)]}{K_2 - K_1}.$$

The source explicitly notes this creates a contradiction (different base tranches assign different correlations to the same underlying portfolio) and that base correlation is not arbitrage-free, though it has practical advantages.

#### What Is Bumped in Base Correlation?

**Source-supported statement:** a $[K_1, K_2]$ tranche value is only sensitive to $\rho(K_1)$ and $\rho(K_2)$ within the base correlation framework.

I'm not sure which base correlation construction variant your desk uses for non-standard strikes and interpolation (the source warns linear interpolation can generate arbitrage-like behavior).

---

### 4.4 Tranche Dependence of Correlation Risk (Sign Intuition)

The source describes a typical pattern:
- Long protection equity tranches can be **short correlation** (their value falls if correlation increases).
- More senior long protection tranches tend to be **long correlation** (value rises as correlation increases).
- There exists a "correlation neutral point" where sensitivity crosses zero.

We will verify this tranche dependence numerically in Examples 8–9 (toy dependence settings).

---

## 5. Jump-to-Default Clustering and Default-Event Scenarios

### 5.1 Single-Name Default Shock vs Clustered Default Shock

**Single-name default shock:** one issuer defaults with recovery $R$, producing portfolio loss increment:

$$\Delta L = \frac{(1 - R) \cdot \text{defaulted face value}}{F}.$$

**Tranche loss increment:**

$$\Delta\text{TL} = \text{TL}(L^- + \Delta L) - \text{TL}(L^-).$$

**Clustered default shock:** $k$ names default "at once" (or in a tightly clustered window), producing:

$$\Delta L = \sum_{j=1}^{k} \frac{(1 - R_j) \cdot \text{face}_j}{F}.$$

This is a proxy for tail/clustering states.

---

### 5.2 Recovery/Final Price Mapping Defaults → Losses → Tranche Payouts

The source's VOD definition is explicit that:
- recovery $R_0$ and defaulted face $H_0$ determine the portfolio loss increment $H_0(1 - R_0)$,
- attachment/detachment are adjusted after default to reflect consumed subordination,
- if tranche is hit, an immediate loss payment $G$ is made and VOD includes $\pm G$.

In practice, "recovery" may be implemented via auction final price or a recovery assumption. I'm not sure which convention you want; we need the product documentation/desk convention.

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

## 6. Hedging Map (Conceptual, Risk-First; No Trade Tips)

### 6.1 Hedge Spread PV01 (Small-Move Risk)

**Goal:** reduce sensitivity to small spread-like changes.

#### The Systemic vs Idiosyncratic Distinction (O'Kane Ch 17)

O'Kane's risk management framework distinguishes two fundamentally different spread sensitivities:

**Systemic Delta:** The change in tranche value if all portfolio spreads widen by 1bp simultaneously. This measures exposure to market-wide spread movements.

**Idiosyncratic Delta:** The change in tranche value if a single name's spread widens by 1bp while others remain unchanged. This measures exposure to name-specific credit deterioration.

For a standard 125-name portfolio with 0–3% equity tranche, O'Kane's Table 17.2 shows:
- Systemic delta: $691 million (leverage ratio 18.37×)
- Per-name idiosyncratic delta: approximately $5.5 million

The ratio is roughly 5:1—hedging systemic risk alone would leave massive idiosyncratic exposure.

#### Practical Hedge Construction

**Systemic hedge with index CDS:**

$$n_{\text{index}} = -\frac{\text{Systemic Delta}_{\text{tranche}}}{\text{DV01}_{\text{index}}}$$

**Idiosyncratic hedge with single names:**

If hedging against name $i$ specifically:

$$n_i = -\frac{\text{Idiosyncratic Delta}_i}{\text{DV01}_{\text{CDS}_i}}$$

**The key insight from O'Kane:** "The actual hedge will lie somewhere between the systemic and the idiosyncratic." A trader expecting correlation-driven moves (systemic) uses the lower hedge ratio; one expecting dispersion (idiosyncratic) uses the higher ratio.

#### Worked Example: Equity Tranche Hedge Sizing

Consider a 0–3% equity tranche (long protection) with:
- Systemic delta = –$691 million (per the source's Table 17.2 example)
- Index DV01 = $50,000 per bp

**Systemic hedge:**

$$n_{\text{index}} = -\frac{-691{,}000{,}000}{50{,}000} = 13{,}820 \text{ index contracts}$$

But if spreads move idiosyncratically rather than in parallel, this hedge is too small by roughly 5×. The practical solution: hedge somewhere between systemic and idiosyncratic based on market view.

#### Failure Modes / Residual Risks

- **Basis:** tranche vs CDS index vs single names.
- **Liquidity:** hedges may be more/less liquid than tranche.
- **Model risk:** hedge ratios depend on model-implied deltas.
- **Jump risk:** PV01 hedges are not designed for defaults (Section 6.3).
- **Systemic/idiosyncratic mismatch:** hedge sized for one type of move underperforms for the other.

---

### 6.2 Hedge Correlation Risk (Hard by Construction)

#### Why It Is Hard (Source-Backed)

After spread and rate risks are hedged, remaining market risk is changes in correlation; correlation should not be assumed fixed.

The source states the only hedging instruments are other correlation products (other tranches); exact offset requires the same tranche, which is often unrealistic.

#### Conceptual Hedges Supported by the Source

- **Complete the capital structure:** combine equity, mezz, senior tranches on same portfolio to reduce net correlation exposure; however, premium legs do not perfectly offset, so a model is needed for precise hedges.
- **Skew hedging in base correlation:** the source provides a procedure to hold index spreads fixed, bump standard tranche spreads, rebuild base correlation curve, and compute hedge notionals in standard tranches that offset small skew-driven PV changes in a non-standard tranche.

#### Worked Example: Capital Structure Combination for Correlation Reduction

Consider a trader long protection on the 3–7% mezzanine tranche (typically long correlation). To reduce net correlation exposure:

**Step 1:** Measure Corr01 of the mezzanine position.
- Suppose Corr01(3–7%) = +$150,000 per 1% correlation increase.

**Step 2:** Find opposing correlation exposure.
- Equity tranches are typically short correlation.
- Suppose Corr01(0–3%) = –$80,000 per 1% correlation increase.

**Step 3:** Size the hedge.
$$n_{\text{equity}} = -\frac{\text{Corr01}_{3-7}}{\text{Corr01}_{0-3}} = -\frac{+150{,}000}{-80{,}000} = 1.875$$

Sell protection on 1.875× the notional of 0–3% equity to offset correlation exposure.

**Caveat:** This hedge is imperfect because:
1. Premium legs don't perfectly offset (different RPV01s).
2. The relationship between Corr01s is model-dependent.
3. Both tranches have spread risk that may or may not offset.

#### Skew Hedging Procedure (O'Kane Ch 17)

For a bespoke tranche with non-standard attachment points, the source outlines:

1. Hold index spreads fixed.
2. Bump each standard tranche contractual spread by 1bp.
3. Rebuild the base correlation curve from bumped standard tranche prices.
4. Reprice the bespoke tranche using the bumped base correlation curve.
5. The change in bespoke tranche value per standard tranche spread bump gives hedge ratios.

This procedure captures how the bespoke tranche is exposed to skew movements (changes in the base correlation curve shape).

#### Failure Modes / Residual Risks

- **Curve/mapping risk (base correlation "mapping" sensitivity):** bespoke tranche value depends on $\rho(K_1)$, $\rho(K_2)$ and their dependence on portfolio spread; hedges can slip when mapping changes.
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

**What it shows:** $\text{TL}(L)$ as a function of portfolio loss $L$ for a tranche $[A, D]$.

**Shape:**
- Flat at zero for $L < A$ (losses below attachment don't touch the tranche)
- Linear slope = 1 for $A \leq L \leq D$ (tranche absorbs losses dollar-for-dollar)
- Flat at $W$ for $L > D$ (tranche fully written down)

**Why it matters:** This piecewise-linear shape is why tranches have different risk characteristics than linear instruments. The derivative $\partial\text{TL}/\partial L$ jumps from 0 to 1 at attachment and from 1 to 0 at detachment.

### Correlation Sensitivity by Attachment Point

**What it shows:** Corr01 (or correlation delta) as a function of tranche attachment point.

**Typical shape:**
- **Equity (low attachment):** Negative Corr01 — value decreases when correlation increases
- **Mezzanine (middle):** Near-zero Corr01 — "correlation neutral point"
- **Senior (high attachment):** Positive Corr01 — value increases when correlation increases

**Why it matters:** This chart reveals where in the capital structure correlation risk concentrates. Senior tranche holders are effectively long options on correlation—they benefit when default clustering becomes more likely (because they're short protection, or equivalently, long credit).

### RPV01 Profile Across the Capital Structure

**What it shows:** Tranche RPV01 (risky PV01) as a function of attachment point.

**Typical shape (from O'Kane Table 17.2):**
- Equity: RPV01 ≈ 2.95 (low, due to expected early write-down)
- Senior: RPV01 ≈ 4.50 (high, expected to survive)

**Why it matters:** Premium-leg exposure varies dramatically across tranches. A senior tranche holder collects premium for longer, creating higher duration-like exposure.

### VOD Profile by Name

**What it shows:** Value-on-Default for each name in the portfolio.

**Typical shape:**
- Names trading at wider spreads: larger VOD (higher implied default probability means default is less "surprising")
- Names with higher face value: larger absolute VOD
- For equity tranches: VOD is positive (long protection benefits from defaults)
- For super-senior: VOD may be near zero (defaults don't reach the tranche)

**Why it matters:** Identifies which names contribute most to jump risk, enabling targeted hedging.

### Portfolio Loss Distribution: Low vs High Correlation

**What it shows:** PDF or CDF of portfolio loss $L(T)$ under different correlation assumptions.

**Typical shape:**
- Low correlation: bell-shaped, concentrated near expected loss
- High correlation: bimodal or fat-tailed, more probability mass at both zero and at extremes

**Why it matters:** Visualizes why correlation affects different tranches oppositely. High correlation increases "all or nothing" outcomes—good for equity (fewer moderate losses), bad for senior (more extreme losses).

---

## 7. Math and Derivations (Step-by-Step)

### 7.1 PV in Terms of ETL/EON on a Time Grid

Let $0 = t_0 < t_1 < \cdots < t_N = T$.

#### Step 1: Express Protection Payments via Expected Tranche Loss Increments

- Tranche loss fraction at time $t$: $\text{TL}(L(t))$.
- Expected tranche loss fraction: $\text{ETL}(t) = \mathbb{E}[\text{TL}(L(t))]$.
- Over interval $(t_{i-1}, t_i]$, the expected tranche loss increment is:

$$\Delta\text{ETL}_i = \text{ETL}(t_i) - \text{ETL}(t_{i-1}) \geq 0.$$

**Protection leg PV (long protection, USD):**

$$\boxed{\text{PV}_{\text{prot}} \approx F \sum_{i=1}^{N} Z(t_i) \, \Delta\text{ETL}_i}$$

**Unit check:** $F$ (USD) $\times$ $Z$ (unitless) $\times$ $\Delta\text{ETL}$ (unitless) = USD. ✓

This is consistent with the source's representation $\int_0^T Z(t)\,(-dQ(t))$ once we note $Q(t) = \text{EON}(t)/W$ and that loss increments correspond to $-dQ$ times tranche notional.

#### Step 2: Express Premium Payments via Expected Outstanding

**Expected outstanding fraction:**

$$\text{EON}(t) = W - \text{ETL}(t).$$

**Approximate expected outstanding over interval using trapezoid:**

$$\overline{\text{EON}}_i = \frac{\text{EON}(t_{i-1}) + \text{EON}(t_i)}{2}.$$

**Premium leg PV (long protection pays premium, USD):**

$$\boxed{\text{PV}_{\text{prem}} \approx F \cdot \frac{s}{10{,}000} \sum_{i=1}^{N} \Delta_i Z(t_i) \, \overline{\text{EON}}_i}$$

**Unit check:** $F$ (USD) $\times$ $s/10{,}000$ (1/year) $\times$ $\Delta$ (years) = USD, times $Z$ and $\overline{\text{EON}}$ (unitless). ✓

This matches the CDS-style trapezoidal premium leg in the source when written in $Q$ form.

#### PV (Long Protection)

$$\boxed{\text{PV} = \text{PV}_{\text{prot}} - \text{PV}_{\text{prem}} - \text{Upfront}}$$

---

### 7.2 PV01 Definition from Finite Differences (Robust)

Let $\text{PV}(s)$ be PV as a function of contractual tranche spread $s$ in bp/year.

**Central-difference PV01:**

$$\boxed{\text{PV01} \approx \frac{\text{PV}(s + 1) - \text{PV}(s - 1)}{2}}$$

**Units:** USD per bp.

**Sign sanity:** for long protection, increasing $s$ increases premium you pay, so PV should go down $\Rightarrow$ PV01 < 0.

---

### 7.3 Hedge Ratio via PV01 Matching (Transparent)

To hedge a target instrument "T" with hedge instrument "H" under small spread moves:

$$\Delta\text{PV}_{\text{total}} \approx \text{PV01}_T \Delta s + n_H \cdot \text{PV01}_H \Delta s \approx 0$$

so

$$\boxed{n_H = -\frac{\text{PV01}_T}{\text{PV01}_H}}$$

$n_H$ is the hedge notional scaling (or number of contracts) assuming PV01 scales linearly with notional.

**Unit check:** USD/bp divided by USD/bp is unitless. ✓

---

### 7.4 Correlation Delta via Finite Differences

Let $\text{PV}(\rho)$ be tranche PV as a function of correlation parameter $\rho$.

$$\text{CorrDelta} \approx \frac{\text{PV}(\rho + \Delta\rho) - \text{PV}(\rho - \Delta\rho)}{2\Delta\rho}$$

**Corr01 conversion:**

If Corr01 is defined as the PV change for 1% absolute correlation bump, then:

$$\boxed{\text{Corr01} \approx \text{CorrDelta} \times 0.01}$$

This aligns with the source definition of "correlation 01" as a 1% absolute bump in correlation.

---

## 8. Worked Examples (At Least 14 Numeric Examples; Fully Numeric)

### Common Toy Setup (Used Unless Otherwise Stated)

| Parameter | Value |
|-----------|-------|
| Portfolio notional | $F = \$100{,}000{,}000$ |
| Payment grid (quarterly, $T = 1$y) | $t_0 = 0$, $t_1 = 0.25$, $t_2 = 0.50$, $t_3 = 0.75$, $t_4 = 1.00$ |
| Accruals | $\Delta_i = 0.25$ |
| Discount factors (toy) | $Z(0.25) = 0.990$, $Z(0.50) = 0.985$, $Z(0.75) = 0.980$, $Z(1.00) = 0.975$ |

---

### Example 1: Compute $\text{TL}(L)$ and $\text{ON}(L)$ Table for $L = 0\%, 2\%, 5\%, 10\%, 20\%$

**Tranche:** $[A, D] = [3\%, 7\%] \Rightarrow W = 4\% = 0.04$.

Compute $\text{TL}(L) = \min(\max(L - A, 0), W)$ and $\text{ON}(L) = W - \text{TL}(L)$.

| Portfolio loss $L$ | $L - A$ | $\max(L - A, 0)$ | $\text{TL}(L)$ | $\text{ON}(L)$ |
|--------------------|---------|------------------|----------------|----------------|
| 0% | -3% | 0% | 0% | 4% |
| 2% | -1% | 0% | 0% | 4% |
| 5% | 2% | 2% | 2% | 2% |
| 10% | 7% | 7% | 4% (cap) | 0% |
| 20% | 17% | 17% | 4% (cap) | 0% |

**Convert to dollars** (multiply by $F = 100$mm):
- At $L = 5\%$: tranche loss = $2\% \times 100$mm = \$2.0mm; outstanding = \$2.0mm.
- At $L = 10\%$: tranche loss = $4\% \times 100$mm = \$4.0mm; outstanding = \$0.

---

### Example 2: Given a Discrete Loss Distribution for $L(T)$, Compute $\text{ETL}(T)$ and Expected Outstanding

Use $T = 1$y loss distribution:

| $L$ | Probability $p$ |
|-----|-----------------|
| 0% | 0.60 |
| 2% | 0.25 |
| 5% | 0.10 |
| 10% | 0.05 |

From Example 1, $\text{TL}(L)$ for $[3, 7]$ is: $0, 0, 2\%, 4\%$.

**Compute expected tranche loss:**

$$\text{ETL}(T) = 0.60(0) + 0.25(0) + 0.10(0.02) + 0.05(0.04) = 0.002 + 0.002 = 0.004.$$

So $\text{ETL}(T) = 0.4\%$ of portfolio.

**Expected outstanding:**

$$\text{EON}(T) = W - \text{ETL}(T) = 0.04 - 0.004 = 0.036.$$

So expected outstanding is $3.6\%$ of portfolio = \$3.6mm.

---

### Example 3: Compute $\text{PV}_{\text{prot}}$ from $\text{ETL}(t_i)$ Increments on the Time Grid

Assume (toy) linear time scaling of ETL:

$$\text{ETL}(t_i) = t_i \cdot \text{ETL}(1).$$

From Example 2, $\text{ETL}(1) = 0.004$.

Thus:
- $\text{ETL}(0) = 0$
- $\text{ETL}(0.25) = 0.001$
- $\text{ETL}(0.50) = 0.002$
- $\text{ETL}(0.75) = 0.003$
- $\text{ETL}(1.00) = 0.004$

Each increment: $\Delta\text{ETL}_i = 0.001$.

**Protection leg PV:**

$$\text{PV}_{\text{prot}} \approx F \sum_{i=1}^{4} Z(t_i) \Delta\text{ETL}_i = 100\text{mm} \times 0.001 \times (0.990 + 0.985 + 0.980 + 0.975).$$

Sum $Z$: $3.930$. So:

$$\boxed{\text{PV}_{\text{prot}} = 100\text{mm} \times 0.00393 = \$393{,}000}$$

---

### Example 4: Compute $\text{PV}_{\text{prem}}$ for Spread $s$ Using Expected Outstanding on the Same Grid

Take contractual tranche spread $s = 100$ bp/year ($s_{\text{dec}} = 0.01$).

Compute $\text{EON}(t_i) = W - \text{ETL}(t_i)$ with $W = 0.04$:

| $t_i$ | $\text{EON}(t_i)$ |
|-------|-------------------|
| 0 | 0.0400 |
| 0.25 | 0.0390 |
| 0.50 | 0.0380 |
| 0.75 | 0.0370 |
| 1.00 | 0.0360 |

**Trapezoidal averages:**

| $i$ | $\overline{\text{EON}}_i$ |
|-----|---------------------------|
| 1 | $(0.0400 + 0.0390)/2 = 0.0395$ |
| 2 | $(0.0390 + 0.0380)/2 = 0.0385$ |
| 3 | $(0.0380 + 0.0370)/2 = 0.0375$ |
| 4 | $(0.0370 + 0.0360)/2 = 0.0365$ |

**Premium leg PV:**

$$\text{PV}_{\text{prem}} = F \cdot \frac{s}{10{,}000} \sum_{i=1}^{4} \Delta_i Z(t_i) \overline{\text{EON}}_i.$$

Compute each term (using $F \cdot s/10{,}000 = 100\text{mm} \cdot 0.01 = 1\text{mm}$):

| $i$ | Calculation | Result |
|-----|-------------|--------|
| 1 | $1\text{mm} \cdot 0.25 \cdot 0.990 \cdot 0.0395$ | \$9,776.25 |
| 2 | $1\text{mm} \cdot 0.25 \cdot 0.985 \cdot 0.0385$ | \$9,475.63 |
| 3 | $1\text{mm} \cdot 0.25 \cdot 0.980 \cdot 0.0375$ | \$9,187.50 |
| 4 | $1\text{mm} \cdot 0.25 \cdot 0.975 \cdot 0.0365$ | \$8,898.38 |

**Total:**

$$\boxed{\text{PV}_{\text{prem}} = \$37{,}337.76}$$

---

### Example 5: Solve Par Spread $s^*$ That Sets PV $\approx 0$

For long protection:

$$\text{PV} = \text{PV}_{\text{prot}} - \text{PV}_{\text{prem}}.$$

Since $\text{PV}_{\text{prem}}$ is linear in $s$, define the "annuity per 1bp":

$$\text{PV01}_{\text{annuity}} = F \cdot \frac{1}{10{,}000} \sum_{i=1}^{4} \Delta_i Z(t_i) \overline{\text{EON}}_i.$$

From Example 4, $\text{PV}_{\text{prem}}(100\text{bp}) = \$37{,}337.76$, so:

$$\text{PV01}_{\text{annuity}} = \$37{,}337.76 / 100 = \$373.3776 \text{ per bp}.$$

**Par spread solves:**

$$0 = \text{PV}_{\text{prot}} - s^* \cdot \text{PV01}_{\text{annuity}} \quad\Rightarrow\quad s^* = \frac{\text{PV}_{\text{prot}}}{\text{PV01}_{\text{annuity}}}.$$

Using $\text{PV}_{\text{prot}} = \$393{,}000$:

$$\boxed{s^* = \frac{393{,}000}{373.3776} = 1052.6 \text{ bp}}$$

**Interpretation:**
- If contractual spread $s < s^*$, long protection has positive PV (cheap premium).
- If $s > s^*$, long protection has negative PV (expensive premium).

---

### Example 6: Tranche PV01: Bump $s$ by $\pm 1$bp and Compute PV01 via Central Difference

Use PV at $s = 100$bp (Examples 3–4):

$$\text{PV}(100) = 393{,}000 - 37{,}337.76 = 355{,}662.24.$$

Because premium leg PV scales linearly with $s$, we can compute:

- $\text{PV}(101) = \text{PV}(100) - 1 \times \text{PV01}_{\text{annuity}} = 355{,}662.24 - 373.3776 = 355{,}288.86$
- $\text{PV}(99) = \text{PV}(100) + 373.3776 = 356{,}035.62$

**Central-difference PV01:**

$$\text{PV01} \approx \frac{\text{PV}(101) - \text{PV}(99)}{2} = \frac{355{,}288.86 - 356{,}035.62}{2} = -373.38 \text{ \$/bp}.$$

**Sign check:** negative for long protection (higher contractual spread means paying more premium). ✓

---

### Example 7: Compare PV01 Across Two Tranches with Different Attachments (Equity vs Senior)

Use the same loss distribution as Example 2 and same time-grid assumptions.

#### (a) Equity Tranche $[0, 3]$: $W = 0.03$

Compute $\text{TL}(L) = \min(L, 0.03)$:

| $L$ | $\text{TL}(L)$ |
|-----|----------------|
| 0% | 0 |
| 2% | 2% |
| 5% | 3% (cap) |
| 10% | 3% (cap) |

**Expected loss:**

$$\text{ETL}(1) = 0.60(0) + 0.25(0.02) + 0.10(0.03) + 0.05(0.03) = 0.005 + 0.003 + 0.0015 = 0.0095.$$

So $\text{EON}(1) = 0.03 - 0.0095 = 0.0205$.

Assume linear ETL in time:
- $\text{ETL}(0.25) = 0.002375 \Rightarrow \text{EON} = 0.027625$
- $\text{ETL}(0.50) = 0.00475 \Rightarrow \text{EON} = 0.02525$
- $\text{ETL}(0.75) = 0.007125 \Rightarrow \text{EON} = 0.022875$

Compute $\sum \Delta Z \overline{\text{EON}}$:

| Term | Calculation | Result |
|------|-------------|--------|
| 1 | $0.25 \cdot 0.990 \cdot (0.03 + 0.027625)/2$ | 0.007131094 |
| 2 | $0.25 \cdot 0.985 \cdot (0.027625 + 0.02525)/2$ | 0.006510234 |
| 3 | $0.25 \cdot 0.980 \cdot (0.02525 + 0.022875)/2$ | 0.005895313 |
| 4 | $0.25 \cdot 0.975 \cdot (0.022875 + 0.0205)/2$ | 0.005286328 |

Sum $= 0.024822969$.

Thus PV01 annuity per bp:

$$\text{PV01}_{\text{equity}} = F \cdot \frac{1}{10{,}000} \cdot 0.024822969 = 100\text{mm}/10{,}000 \cdot 0.024822969 = \$248.23/\text{bp}.$$

#### (b) Senior Tranche $[15, 30]$: $W = 0.15$

Under loss scenarios up to 10%, tranche never attaches $\Rightarrow$ $\text{ETL}(t) = 0$ $\Rightarrow$ $\text{EON}(t) = 0.15$ constant.

Compute $\sum \Delta Z \overline{\text{EON}} = 0.15 \sum_{i=1}^{4} 0.25 Z(t_i)$:

$$= 0.15 \cdot 0.25(0.990 + 0.985 + 0.980 + 0.975) = 0.15 \cdot 0.9825 = 0.147375.$$

Thus:

$$\text{PV01}_{\text{senior}} = 10{,}000 \cdot 0.147375 = \$1{,}473.75/\text{bp}.$$

#### Conclusion (Numerical)

$$\boxed{|\text{PV01}_{\text{equity}}| < |\text{PV01}_{\text{mezz}}| < |\text{PV01}_{\text{senior}}|}$$

This matches the source's qualitative statement that tranche RPV01 is lowest for equity and rises for senior tranches (with caveats).

---

### Example 8: Correlation Shock (Toy): Low vs High Dependence, Compute ETL and PV Change for a Senior Tranche

We use tranche $[15, 30]$ (senior) to highlight tail sensitivity.

**Setup:**
- Maturity $T = 1$, same discount factors and grid.
- Contractual spread $s = 50$ bp.

#### Dependence Setting A (Low Tail / Low "Correlation")

Use loss distribution:
- $L = 0\%$ (0.60), $L = 2\%$ (0.25), $L = 5\%$ (0.10), $L = 10\%$ (0.05)

Since all losses $< 15\%$, tranche never attaches:
- $\text{ETL}_A(1) = 0$
- $\text{PV}_{\text{prot},A} = 0$
- $\text{PV01}_A = 1{,}473.75$ \$/bp (from Example 7(b))

**Premium PV:**

$$\text{PV}_{\text{prem},A} = s \cdot \text{PV01}_A = 50 \cdot 1{,}473.75 = \$73{,}687.50$$

So long-protection PV:

$$\text{PV}_A = 0 - 73{,}687.50 = -\$73{,}687.50.$$

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
- At $L = 25\%$: $\text{TL} = 25\% - 15\% = 10\% = 0.10$.
- Otherwise 0.

So:

$$\text{ETL}_B(1) = 0.05 \cdot 0.10 = 0.005.$$

Assume linear in time: $\Delta\text{ETL}_i = 0.005/4 = 0.00125$.

**Protection PV:**

$$\text{PV}_{\text{prot},B} = 100\text{mm} \cdot 0.00125 \cdot (0.990 + 0.985 + 0.980 + 0.975) = 100\text{mm} \cdot 0.0049125 = \$491{,}250.$$

**Premium PV01** (recompute with declining EON):

$W = 0.15$, $\text{EON}(1) = 0.15 - 0.005 = 0.145$.

Using the computed sum $\sum \Delta Z \overline{\text{EON}} = 0.1449265625$ (see derivation in analysis), PV01:

$$\text{PV01}_B = 10{,}000 \cdot 0.1449265625 = \$1{,}449.27/\text{bp}.$$

**Premium PV at $s = 50$bp:**

$$\text{PV}_{\text{prem},B} = 50 \cdot 1{,}449.27 = \$72{,}463.28.$$

Thus:

$$\text{PV}_B = 491{,}250 - 72{,}463.28 = \$418{,}786.72.$$

#### PV Change Due to Dependence Shift

$$\boxed{\Delta\text{PV} = \text{PV}_B - \text{PV}_A = 418{,}786.72 - (-73{,}687.50) = \$492{,}474.22}$$

**Interpretation:** A "pure dependence/tail" increase (with mean loss held fixed) massively impacts senior tranche PV.

---

### Example 9: Correlation Delta: Compute PV at $\rho \pm \Delta\rho$ and Report CorrDelta

We interpret settings A and B as $\rho - \Delta\rho$ and $\rho + \Delta\rho$ around a midpoint $\rho$.

Let:
- $\rho - \Delta\rho = 0.20 \Rightarrow \text{PV}(0.20) = \text{PV}_A = -73{,}687.50$
- $\rho + \Delta\rho = 0.40 \Rightarrow \text{PV}(0.40) = \text{PV}_B = 418{,}786.72$

Then midpoint $\rho = 0.30$, $\Delta\rho = 0.10$.

**Compute CorrDelta:**

$$\text{CorrDelta} \approx \frac{\text{PV}(0.40) - \text{PV}(0.20)}{2 \cdot 0.10} = \frac{418{,}786.72 - (-73{,}687.50)}{0.20} = \frac{492{,}474.22}{0.20} = \$2{,}462{,}371.10 \text{ per unit } \rho.$$

**Convert to Corr01 (1% absolute bump):**

$$\boxed{\text{Corr01} \approx 0.01 \times \text{CorrDelta} = \$24{,}623.71 \text{ per 1\%}}$$

This corresponds to the source's concept of "correlation 01" as the PV change for a 1% absolute increase in correlation.

---

### Example 10: Single-Name Default Scenario: Compute Portfolio Loss Increment, Tranche Loss Increment, PV Change Proxy

**Portfolio:** 100 equal names, each \$1mm. Total $F = \$100$mm.

**Default event:**
- One name defaults, recovery $R = 40\%$ $\Rightarrow$ LGD $= 60\%$.
- Loss dollars: $0.60 \times \$1$mm $= \$0.60$mm.

**Loss fraction increment:**

$$\Delta L = 0.60\text{mm} / 100\text{mm} = 0.006 = 0.6\%.$$

**Assume realized portfolio loss before default:** $L^- = 2.8\% = 0.028$. After default:

$$L^+ = 0.028 + 0.006 = 0.034 = 3.4\%.$$

**Tranche:** $[3, 7]$, $A = 0.03$, $W = 0.04$.

**Compute tranche loss before:**

$$\text{TL}(L^-) = \min(\max(0.028 - 0.03, 0), 0.04) = 0.$$

**After:**

$$\text{TL}(L^+) = \min(\max(0.034 - 0.03, 0), 0.04) = 0.004 = 0.4\%.$$

**So tranche loss increment:**

$$\Delta\text{TL} = 0.004 - 0 = 0.004.$$

**Dollar loss payment on tranche:**

$$\boxed{G = \Delta\text{TL} \cdot F = 0.004 \times 100\text{mm} = \$0.40\text{mm}}$$

#### PV Change Proxy

- Immediate protection payment to long protection is $+\$0.40$mm.
- Remaining outstanding tranche notional decreases from $W \cdot F = 0.04 \cdot 100$mm $= \$4.0$mm to $(W - \Delta\text{TL})F = 0.036 \cdot 100$mm $= \$3.6$mm.
- Premium leg PV will drop (because premium is paid on outstanding); protection leg PV also drops (less remaining protection).

A book-consistent way to capture the total jump is VOD, which includes repricing after the default and adding/subtracting $G$.

---

### Example 11: Clustered Default Scenario: $k$ Names Default Simultaneously; Compute Tranche Loss and PV Impact

Use same portfolio and tranche.

**Cluster event:**
- $k = 5$ defaults simultaneously, each with $R = 40\%$.
- Total loss dollars $= 5 \times 0.60$mm $= 3.0$mm.
- $\Delta L = 3.0/100 = 0.03 = 3\%$.

**Assume $L^- = 2.8\% = 0.028$. Then $L^+ = 0.058 = 5.8\%$.**

**Tranche loss:**

$$\text{TL}(L^-) = 0, \quad \text{TL}(L^+) = \min(\max(0.058 - 0.03, 0), 0.04) = 0.028 = 2.8\%.$$

**So loss payment:**

$$\boxed{G = 0.028 \times 100\text{mm} = \$2.8\text{mm}}$$

**Remaining tranche notional after the jump:**

$$(0.04 - 0.028) \times 100\text{mm} = 0.012 \times 100\text{mm} = \$1.2\text{mm}.$$

#### Comparison to Example 10

| Scenario | Loss Payment $G$ |
|----------|------------------|
| Single-name default | \$0.40mm |
| Cluster of 5 defaults | \$2.8mm |

This is the nonlinear "clustering amplification" of tranche jump exposure.

---

### Example 12: Recovery/Final Price Sensitivity: Vary Recovery and Compute Tranche Payout Differences

Single-name default with $L^- = 2.8\%$ and face value \$1mm.

**Compute $G$ for different recoveries:**

#### Case R = 20%

- LGD = 80%, loss dollars = \$0.80mm → $\Delta L = 0.008 = 0.8\%$
- $L^+ = 3.6\%$ $\Rightarrow$ $\text{TL}(L^+) = 3.6 - 3.0 = 0.6\%$
- $G = 0.006 \times 100$mm $= \$0.60$mm

#### Case R = 40%

From Example 10: $G = \$0.40$mm

#### Case R = 60%

- LGD = 40%, loss dollars = \$0.40mm → $\Delta L = 0.004 = 0.4\%$
- $L^+ = 3.2\%$ $\Rightarrow$ $\text{TL}(L^+) = 0.2\%$
- $G = 0.002 \times 100$mm $= \$0.20$mm

**Sensitivity (finite difference):**

$$\frac{\Delta G}{\Delta R} \approx \frac{0.20\text{mm} - 0.60\text{mm}}{0.60 - 0.20} = \frac{-0.40\text{mm}}{0.40} = -1.00\text{mm per 1.0 recovery}.$$

**So per 1% recovery:**

$$\boxed{\approx -0.01\text{mm} = -\$10{,}000 \text{ per 1\% recovery}}$$

---

### Example 13: Hedge PV01 with Another Tranche (PV01 Matching) and Validate Under Small Spread Moves

We hedge contractual tranche-spread PV01 (quote PV01) of $[3, 7]$ long protection using $[0, 3]$ short protection.

**From Examples:**
- $\text{PV01}_{3-7}^{\text{(long prot)}} = -\$373.38/\text{bp}$ (Example 6).
- $\text{PV01}_{0-3}^{\text{(long prot)}} = -\$248.23/\text{bp}$ (Example 7(a)).
- Therefore $\text{PV01}_{0-3}^{\text{(short prot)}} = +\$248.23/\text{bp}$.

Let hedge scale be $h$ (multiplier of notional exposure of the $0$–$3$ tranche relative to the base \$100mm portfolio in this toy setup). Solve:

$$-373.38 + h \cdot 248.23 = 0 \quad\Rightarrow\quad h = \frac{373.38}{248.23} = 1.504.$$

#### Validation Under Small Spread Move

Assume both tranche contractual spreads increase by $\Delta s = +5$bp.

**Target P&L:**

$$\Delta\text{PV}_{3-7} \approx -373.38 \times 5 = -\$1{,}866.9.$$

**Hedge P&L:**

$$\Delta\text{PV}_{\text{hedge}} \approx h \cdot (+248.23) \times 5 = 1.504 \times 1{,}241.15 = +\$1{,}867.0.$$

**Net:**

$$\boxed{\Delta\text{PV}_{\text{net}} \approx +0.1 \text{ (rounding)}}$$

So the position is PV01-neutral to small tranche spread quote moves in this toy setting.

---

### Example 14: Hedge Failure Under Clustering: Apply Clustered Default Scenario to the PV01-Hedged Portfolio

Use the PV01-neutral portfolio from Example 13:
- Long protection $[3, 7]$ on \$100mm portfolio.
- Short protection $[0, 3]$ scaled by $h = 1.504$.

**Now apply a clustered default shock from no prior losses:**
- $k = 8$ defaults, each \$1mm face, recovery 40%.
- Each loss = \$0.60mm → total loss = $8 \times 0.60 = \$4.8$mm.
- Portfolio loss fraction jump: $\Delta L = 4.8/100 = 4.8\%$.

**Compute tranche loss increments from $L^- = 0$ to $L^+ = 4.8\%$:**

#### Mezz Tranche [3, 7]

$$\text{TL}_{3-7}(4.8\%) = \min(4.8\% - 3.0\%,\; 4.0\%) = 1.8\%.$$

Dollar loss payment to long protection:

$$G_{3-7} = 1.8\% \times 100\text{mm} = \$1.8\text{mm}.$$

#### Equity Tranche [0, 3]

$$\text{TL}_{0-3}(4.8\%) = \min(4.8\%, 3.0\%) = 3.0\%.$$

Dollar loss payment for short protection position (a cost):

$$G_{0-3} = 3.0\% \times 100\text{mm} = \$3.0\text{mm}.$$

Scaled by $h = 1.504$, hedge pays:

$$h \cdot G_{0-3} = 1.504 \times 3.0 = \$4.512\text{mm}.$$

#### Net Jump P&L (Ignoring MTM Repricing of Remaining Legs for Simplicity)

$$\boxed{\Delta\text{PV}_{\text{jump}} \approx +1.8 - 4.512 = -\$2.712\text{mm}}$$

#### Interpretation

The portfolio was PV01-neutral for small tranche spread quote moves, but suffered a large loss under a clustered default event.

**This is the core message: first-order PV01 hedging does not immunize jump/clustering risk.**

---

## 9. Practical Notes

### 9.1 Risk Report Glossary (Desk-Ready Definitions)

| Measure | Bump | Definition | Interpretation |
|---------|------|------------|----------------|
| **Tranche PV01** | Contractual tranche spread $s$ by +1bp | $\text{PV}(s+1) - \text{PV}(s)$ (USD/bp) or central difference | Premium-leg annuity exposure |
| **Correlation delta / Corr01** | Correlation parameter $\rho$ by +1% absolute | $\text{Corr01} = V(\rho + 0.01) - V(\rho)$ | Parameterization: one-factor $\rho = \beta^2$ in the source's definition |
| **JTD / VOD** | Immediate default of a name (or set of names), with recovery $R$ | $\text{VOD} = V'(t) - V(t) \pm G$, with $+G$ for long protection if tranche is hit | Discrete jump exposure |

---

### 9.2 Common Pitfalls

1. **Mixing quote regimes** (spread vs upfront+coupon) in one PV01 report.
   - Source example: equity tranche can be quoted as upfront + 500bp while others are bp-only.

2. **Treating an implied "correlation number" as structural.** Correlation is not fixed; source explicitly warns it should not be assumed constant.

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
| ETL bounds | $0 \leq \text{ETL}(t) \leq W$, and $\text{ETL}(t)$ nondecreasing in $t$ |
| EON bounds | $0 \leq \text{EON}(t) \leq W$, nonincreasing in $t$ |
| Correlation toy sanity | Higher dependence should increase tail mass; senior tranche PV should rise in the tail-heavy setting (Examples 8–9) |
| Hedge validation | PV01-neutral portfolios show small $\Delta$PV under small spread shocks (Example 13), but not under default jumps (Example 14) |

---

## 10. Summary & Recall

### 10.1 Executive Summary (10 Bullets)

1. Tranche valuation is driven by the portfolio loss process $L(t)$ mapped through the nonlinear tranche loss function $\text{TL}(L)$.

2. Expected tranche loss $\text{ETL}(t)$ and expected outstanding $\text{EON}(t)$ are the core objects for protection and premium legs.

3. A tranche survival curve $Q(t) = \text{EON}(t)/W$ enables CDS-style valuation; the source notes an exact mapping once $Q$ is known.

4. Contractual tranche PV01 measures \$/bp sensitivity to the tranche's running premium quote; it is essentially the discounted expected outstanding "annuity."

5. PV01 varies by attachment: junior tranches have lower PV01 (they amortize faster), senior tranches higher PV01 (they survive longer), consistent with the source's risk report pattern.

6. Correlation/dependence changes the loss distribution tail and therefore tranche PV; equity and senior tranches often have opposite correlation exposure signs.

7. Corr01 is a practical risk measure: PV change for a 1% absolute bump in correlation.

8. JTD/VOD captures PV jumps under immediate defaults and depends on recovery and whether the tranche is hit; VOD includes $\pm G$.

9. Default clustering is tail risk: multiple defaults in stress states; tail dependence formalizes "joint extremes" and Gaussian copula tail independence highlights model risk in extremes.

10. Hedging is exposure-targeted: PV01 hedges address small moves, correlation hedges require other tranches, and jump/clustering risk demands scenario-based management.

---

### 10.2 Cheat Sheet (Definitions + Formulas)

**Tranche loss:**

$$\text{TL}(L) = \min(\max(L - A, 0), W), \quad W = D - A.$$

**Expected tranche loss / outstanding:**

$$\text{ETL}(t) = \mathbb{E}[\text{TL}(L(t))], \quad \text{EON}(t) = W - \text{ETL}(t).$$

**Tranche survival fraction:**

$$Q(t) = \text{EON}(t) / W.$$

**Protection PV (grid approximation):**

$$\text{PV}_{\text{prot}} \approx F \sum_i Z(t_i)(\text{ETL}(t_i) - \text{ETL}(t_{i-1})).$$

**Premium PV (grid approximation):**

$$\text{PV}_{\text{prem}} \approx F \cdot \frac{s}{10{,}000} \sum_i \Delta_i Z(t_i) \cdot \frac{\text{EON}(t_{i-1}) + \text{EON}(t_i)}{2}.$$

**PV01 (central difference):**

$$\text{PV01} \approx \frac{\text{PV}(s+1) - \text{PV}(s-1)}{2}.$$

**Hedge ratio:**

$$n_H = -\frac{\text{PV01}_T}{\text{PV01}_H}.$$

**CorrDelta & Corr01:**

$$\text{CorrDelta} \approx \frac{V(\rho + \Delta\rho) - V(\rho - \Delta\rho)}{2\Delta\rho}, \quad \text{Corr01} \approx 0.01 \cdot \text{CorrDelta}.$$

**VOD/JTD structure:**

$$\text{VOD} = V'(t) - V(t) \pm G.$$

---

### 10.3 Flashcards (35 Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What is portfolio loss $L(t)$? | Cumulative portfolio loss as a fraction of portfolio notional $F$. |
| 2 | Define tranche width. | $W = D - A$. |
| 3 | Define tranche loss $\text{TL}(L)$. | $\min(\max(L - A, 0), W)$. |
| 4 | Define tranche outstanding $\text{ON}(L)$. | $W - \text{TL}(L)$. |
| 5 | Define expected tranche loss $\text{ETL}(t)$. | $\mathbb{E}[\text{TL}(L(t))]$. |
| 6 | Define expected outstanding $\text{EON}(t)$. | $W - \text{ETL}(t)$. |
| 7 | What is tranche survival curve $Q(t)$? | $Q(t) = \text{EON}(t)/W$. |
| 8 | What leg does $\text{ETL}(t)$ drive? | Protection leg PV. |
| 9 | What leg does $\text{EON}(t)$ drive? | Premium leg PV. |
| 10 | What is tranche-spread PV01? | PV change per 1bp change in contractual tranche spread $s$. |
| 11 | Units of PV01? | USD per bp. |
| 12 | Why is PV01 usually smaller for equity tranches? | Expected outstanding is lower due to earlier write-down. |
| 13 | What is Corr01 (as defined in the source)? | PV change for a 1% absolute increase in correlation. |
| 14 | Why does correlation matter for senior tranches? | It changes tail probability of many defaults. |
| 15 | What is JTD for a tranche? | PV change under a specified default-event scenario. |
| 16 | What is VOD? | "Value-on-default," measuring PV impact of an immediate default. |
| 17 | How does VOD incorporate immediate loss payment $G$? | $\text{VOD} = V'(t) - V(t) \pm G$. |
| 18 | What is default clustering? | Multiple defaults occurring together/in stress states. |
| 19 | Define upper tail dependence coefficient $\lambda_u$. | Limiting conditional exceedance probability as quantile $\to 1$. |
| 20 | Gaussian copula tail dependence in QRM? | Asymptotically independent ($\lambda = 0$ if $\rho < 1$). |
| 21 | Why can PV01 hedges fail in defaults? | Defaults are discontinuous jumps, not small spread moves. |
| 22 | What is the "spread/PV01 term" in P&L decomposition? | P&L from small spread-like moves approximated by PV01-type sensitivities. |
| 23 | What is the "correlation term"? | P&L from dependence parameter shifts (Corr01/CorrDelta). |
| 24 | What is the "recovery term"? | P&L from recovery/LGD/final price changes. |
| 25 | What is residual/model risk? | P&L not explained by first-order terms; model/basis/liquidity effects. |
| 26 | In base correlation, which parameters drive a $[K_1, K_2]$ tranche? | $\rho(K_1)$ and $\rho(K_2)$. |
| 27 | What contradiction does base correlation introduce? | Different base tranches assign different correlations to same portfolio. |
| 28 | Why can base correlation interpolation create issues? | Linear interpolation can generate arbitrage-like tranchelet spreads. |
| 29 | What's the simplest PV01 hedge ratio formula? | $n_H = -\text{PV01}_T / \text{PV01}_H$. |
| 30 | What is a "pure correlation move" scenario? | Tranche price changes with index spreads held fixed. |
| 31 | What instruments hedge correlation risk per the source? | Other tranches (correlation products). |
| 32 | What is "complete the capital structure"? | Combine tranches across capital structure to reduce net correlation exposure. |
| 33 | Why is complete-capital-structure not perfectly correlation neutral? | Premium leg correlation exposures don't offset exactly. |
| 34 | What's a key proxy for clustering/tail risk? | Multi-default clustered default scenario ($k$-name default). |
| 35 | What should always accompany tranche risk numbers? | Clear definition of bumps, parameterization, and scenario assumptions. |
| 36 | What is systemic delta? | PV change if all portfolio spreads widen by 1bp simultaneously. |
| 37 | What is idiosyncratic delta? | PV change if a single name's spread widens by 1bp while others remain unchanged. |
| 38 | How do systemic and idiosyncratic deltas typically compare for equity tranches? | Systemic delta is often ~5× larger than per-name idiosyncratic delta (per O'Kane Table 17.2). |
| 39 | Where does "the actual hedge" lie per O'Kane? | "Somewhere between the systemic and the idiosyncratic" depending on expected spread move type. |
| 40 | What is RPV01 for a tranche? | Risky PV01 — the discounted expected outstanding that determines premium leg sensitivity. |
| 41 | Why is RPV01 lower for equity tranches? | Expected early write-down reduces expected outstanding and thus premium duration. |
| 42 | What does the "hockey stick" shape of TL(L) create? | Nonlinear risk characteristics — PV01 hedges fail for large moves and defaults. |
| 43 | How does auction final price affect tranche settlement? | Loss = (1 - Final Price) × Defaulted Notional; lower final price means larger tranche loss increment. |
| 44 | What is the leverage ratio for a tranche? | Systemic delta / tranche notional — measures effective exposure amplification. |

---

## 11. Mini Problem Set (18 Questions)

*Provide brief solution sketches for questions 1–9 only.*

---

**1.** Compute $\text{TL}(L)$ for tranche $[2\%, 5\%]$ at $L = \{0, 1, 3, 6\}\%$.

> **Sketch:** Use $\text{TL} = \min(\max(L - A, 0), W)$ with $A = 0.02$, $W = 0.03$. Evaluate each $L$.

---

**2.** Bounds check: show $0 \leq \text{ETL}(t) \leq W$.

> **Sketch:** Since $0 \leq \text{TL}(L) \leq W$ pointwise, expectation preserves bounds.

---

**3.** Premium annuity: Given $F = 50$mm, $W = 4\%$, $Q(t_i) = \{1, 0.95, 0.90\}$, $Z = \{0.99, 0.98\}$, $\Delta = \{0.5, 0.5\}$. Compute premium-leg PV for $s = 200$bp using trapezoid.

> **Sketch:** Convert $s$ to decimal; compute $\sum \Delta Z (Q_{i-1} + Q_i)/2$; multiply by $F \cdot W \cdot s_{\text{dec}}$.

---

**4.** Protection PV from ETL: If $\text{ETL}(t_0) = 0$, $\text{ETL}(t_1) = 0.2\%$, $\text{ETL}(t_2) = 0.5\%$, $F = 100$mm, $Z(t_1) = 0.99$, $Z(t_2) = 0.97$, compute $\text{PV}_{\text{prot}}$.

> **Sketch:** $\Delta\text{ETL}_1 = 0.2\%$, $\Delta\text{ETL}_2 = 0.3\%$; PV $= F[0.99(0.002) + 0.97(0.003)]$.

---

**5.** Par spread: Using Problems 3–4, compute par $s^*$.

> **Sketch:** $s^* = \text{PV}_{\text{prot}} / \text{PV01}_{\text{annuity}}$, with PV01 per bp from the annuity.

---

**6.** PV01 central difference: Given $\text{PV}(s-1) = 1.2$mm, $\text{PV}(s+1) = 1.15$mm, compute PV01.

> **Sketch:** $(1.15 - 1.2)/2 = -0.025$mm/bp = $-\$25{,}000$/bp.

---

**7.** Correlation delta: Given $\text{PV}(0.2) = -0.05$mm, $\text{PV}(0.4) = 0.35$mm, compute CorrDelta around 0.3 with $\Delta\rho = 0.1$.

> **Sketch:** $(0.35 - (-0.05))/0.2 = 2.0$mm per unit $\rho$.

---

**8.** Single default loss mapping: Portfolio \$200mm, single name \$5mm defaults at $R = 25\%$. Compute $\Delta L$.

> **Sketch:** LGD=75%; loss=\$3.75mm; $\Delta L = 3.75/200 = 1.875\%$.

---

**9.** Tranche loss increment: For tranche $[3, 7]$, realized loss goes from $2.5\%$ to $4.0\%$. Compute $\Delta\text{TL}$.

> **Sketch:** Before: 0; after: $4 - 3 = 1\%$ (cap at 4%); increment = 1%.

---

**10.** Define a scenario suite to approximate clustering and recovery coupling for a senior tranche; justify each scenario component.

---

**11.** Explain why Gaussian copula tail independence can understate clustered-default risk for senior tranches.

---

**12.** Provide a conceptual explanation of why equity tranches can be short correlation while senior tranches are long correlation.

---

**13.** In base correlation, explain why a $[K_1, K_2]$ tranche depends on $\rho(K_1)$ and $\rho(K_2)$, and what that implies for hedging.

---

**14.** Discuss how interpolation of base correlation can generate arbitrage-like tranchelet spreads and how that becomes model risk.

---

**15.** Construct a PV decomposition report template (headings and definitions only) for a tranche book.

---

**16.** Describe the operational steps to compute VOD/JTD for each name in a portfolio.

---

**17.** Explain how stress testing committees might assign probabilities to extreme scenarios and why this is difficult.

---

**18.** Describe how you would validate a hedge designed to be PV01-neutral, including what tests would reveal tail-risk failure.

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

**Core Valuation Framework:**
- Tranche loss mapping $\text{TL}(L)$, expected loss $\text{ETL}(t)$, tranche survival curve $Q(t)$ — O'Kane Ch 11–12
- CDS-style tranche valuation with trapezoidal premium leg — O'Kane Ch 12
- RPV01 (risky PV01) concept and profile across capital structure — O'Kane Ch 17

**Risk Measures:**
- Systemic delta vs idiosyncratic delta distinction — O'Kane Ch 17
- "The actual hedge will lie somewhere between the systemic and the idiosyncratic" — O'Kane Ch 17
- Correlation 01 definition (1% absolute bump) — O'Kane Ch 14, Ch 17 (risk report framework)
- VOD (Value-on-Default) definition with $\pm G$ — O'Kane Ch 14, Ch 17
- Table 17.2 risk report example data (equity RPV01=2.95, systemic delta=$691mm, leverage=18.37) — O'Kane Ch 17

**Dependence and Tail Risk:**
- Tail dependence coefficient $\lambda_u$ and Gaussian copula asymptotic independence ($\lambda = 0$ for $\rho < 1$) — QRM (McNeil et al)
- Base correlation decomposition and expected loss relationship — O'Kane Ch 13–14
- Empirical negative correlation between default rates and recovery rates — O'Kane Ch 3, QRM

**Hedging Framework:**
- Skew hedging procedure using base correlation — O'Kane Ch 17
- "Complete the capital structure" for correlation hedging — O'Kane Ch 17
- Base correlation interpolation artifacts and arbitrage issues — O'Kane Ch 13–14

### (B) Reasoned Inference — Note Derivation Logic

- PV01 is essentially discounted expected outstanding — follows from premium leg linearity in $s$
- Equity tranches have lower PV01 than senior — follows from expected outstanding patterns and confirmed by O'Kane Table 17.2
- PV01-neutral hedges fail under jump events — demonstrated via Examples 13–14
- Systemic/idiosyncratic hedge ratio difference of ~5× for equity — derived from O'Kane Table 17.2 data
- Correlation hedge using capital structure combination — derived from Corr01 offset principle

### (C) Speculation — Flag Uncertainties

- I'm not sure which discount curve your desk uses for tranche valuation (OIS/CSA depends on collateral, currency, CSA)
- I'm not sure what tranche quote regime (spread vs upfront+coupon) applies to each tranche without product/series specification
- I'm not sure which base correlation construction variant and interpolation method your desk uses
- I'm not sure how your desk defines recovery sensitivity (defaulted name vs portfolio average vs auction final price)
- I'm not sure about the exact timing of auction settlement relative to tranche premium accrual cutoffs without product documentation
