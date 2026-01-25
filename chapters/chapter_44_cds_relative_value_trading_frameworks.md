# Chapter 44: CDS Relative Value and Trading Frameworks (Risk-First, Not "Tips")

---

## Conventions & Notation

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $t$ | Time (today) |
| $T$ | Maturity in years |
| $N$ | Notional in USD |
| $S$ | CDS quoted spread in bp/year (convert to decimal/year via $S_{\text{dec}} = S_{\text{bp}} / 10{,}000$) |
| $R \in [0,1]$ | Recovery rate (e.g., $40\% \Rightarrow R = 0.40$) |
| $\text{LGD}$ | Loss-given-default $= 1 - R$ |
| $Z(t,u)$ | Risk-free discount factor (dimensionless) |
| $Q(t,u)$ | Survival probability $= P(\tau > u \mid \mathcal{F}_t)$ (dimensionless) |
| $h(u)$ | Hazard / forward default rate (per year) with $Q(t,u) = \exp\bigl(-\int_t^u h(s)\,ds\bigr)$ |
| $\text{RPV01}(t,T)$ | Risky PV01 / risky annuity in years (present value of 1 unit spread paid until default/maturity). Discrete approximation via trapezoid in survival between coupon dates is used in the reference text for forward-starting CDS annuities. |
| $V$ | Mark-to-market value (USD) uses signed convention: value to the holder of the position |
| $\text{CS01}_i$ | Signed CS01 (bucketed): USD per 1bp for bucket/node $i$ |
| $\text{VOD}$ | Value-on-Default (jump-to-default proxy from the reference): value at default minus current value |

**Additional Notes:**
- "Carry" and "rolldown" are treated as P&L decompositions under explicit "curves unchanged" assumptions (not forecasts).

---

## 0. Setup

### Conventions Used in This Chapter

- Single-name CDS described via a premium leg (spread payments) and a protection leg (default contingent payment), with settlement determined by physical or cash settlement conventions and deliverables; premium generally paid periodically and a final accrued premium may be due at default under standard market conventions discussed in the sources.
- Credit risk modeled via survival curve $Q(t,u)$ and hazard rate $h(u)$, with the standard reduced-form relation $Q = \exp(-\int h)$.
- "Risk-first" means we define exposures first (CS01 buckets, JTD/VOD, recovery sensitivity, liquidity/roll/technical risks), then design hedges and tests—not "trade tips."

### Notation Glossary (Symbols + Definitions)

| Symbol | Definition |
|--------|------------|
| $\tau$ | Default time |
| $Q(t,u)$ | Survival probability to $u$ conditional on information at $t$ |
| $h(u)$ | Hazard/forward default rate used in $Q(t,u) = \exp\bigl(-\int_t^u h(s)\,ds\bigr)$ |
| $Z(t,u)$ | Discount factor from $t$ to $u$ |
| $S(t,T)$ | Par CDS spread for maturity $T$ observed/derived at time $t$ |
| $c$ | Contractual CDS coupon/spread in the contract |
| $\text{RPV01}(t,T)$ | Risky PV01 (years) |
| $V$ | MTM value of a CDS position |
| $\text{CS01}$ | Spread DV01 (USD per 1bp) |
| $\text{VOD}$ | Value-on-default (USD) (jump-to-default proxy) |
| "Basis" | Difference between CDS spread and bond spread (definition varies by chosen bond spread measure) |

---

## 1. Core Concepts (Definitions First)

### 1.1 Credit Default Swap (CDS)

**Formal Definition:**

A CDS is an agreement where the protection buyer pays periodic premiums and receives a default-contingent payoff if a credit event occurs; a standard description is: the buyer has the right to "sell" eligible bonds for face value upon a credit event, and the seller agrees to "buy" at face (physical settlement framing), or equivalently cash settles against a recovery price determined via market mechanisms (e.g., auction).

**Intuition:**

A CDS converts default risk into an insurance-like premium stream + a jump payoff. The premium leg resembles a risky annuity; the protection leg resembles a default digital scaled by $(1 - R)$.

**How It Appears in Trading/Risk/Portfolio Practice:**

A CDS position carries (i) spread risk (mark-to-market from changes in the CDS curve), (ii) jump-to-default risk (large discrete move on default), (iii) recovery risk (payout depends on recovery), (iv) contract/settlement risk (deliverables, auction mechanics, accrual). These are explicitly discussed in the sources via DV01 and value-on-default style measures.

---

### 1.2 Credit Curve (Single-Name CDS Term Structure)

**Formal Definition:**

The credit curve is the term structure of credit risk for an issuer, represented by any equivalent object:
- par CDS spreads $S(t,T)$ across maturities $T$,
- survival probabilities $Q(t,T)$,
- hazard rates $h(T)$ or piecewise-constant forward hazards.

A common representation uses $Q(t,u) = \exp\bigl(-\int_t^u h(s)\,ds\bigr)$.

**Intuition:**

Short maturities reflect near-term default risk; longer maturities incorporate longer-horizon default risk plus term premia and liquidity/technical effects. The curve's shape matters: "parallel shifts" are not the only plausible moves.

**How It Appears in Practice:**

"Curve trades" are relative value positions between maturities (e.g., 5y vs 1y) and must be risk-managed via bucketed CS01 (credit key-rate style), not only total CS01.

---

### 1.3 Par Spread, Risky PV01 (RPV01), and CDS Value

**Formal Definition:**

In reduced-form pricing, premium leg PV and protection leg PV can be written (continuous-time form shown in the reference) as:

$$\text{PV}_{\text{prem}} = S \int_0^T Z(0,t)\,Q(0,t)\,dt$$

$$\text{PV}_{\text{prot}} = (1-R) \int_0^T Z(0,t)\,\lambda(t)\,Q(0,t)\,dt$$

with $\lambda$ the default intensity/hazard term used in the reference. A "credit triangle" approximation gives $S \approx h(1-R)$ (useful but approximate).

A forward-starting CDS spread is written in the reference as:

$$S(t, t_F, T) = (1-R) \frac{\int_{t_F}^T Z(t,t')\,(-dQ(t,t'))}{\text{RPV01}(t, t_F, T)}$$

The reference also gives a practical mark-to-market identity:

$$\boxed{V(0,T) \approx \bigl(S(t,T) - S(0,T)\bigr) \cdot \text{RPV01}(t,T)}$$

i.e., value is approximately "spread difference $\times$ risky PV01".

**Intuition:**

$\text{RPV01}$ is the present value of 1 unit of spread payments, survival-weighted. If credit worsens (spreads widen), the protection leg becomes more valuable and the premium leg less costly for a protection buyer, leading to positive spread sensitivity.

**How It Appears in Practice:**

Most first-order spread risk reports for CDS are driven by $\text{RPV01}$ scaled by notional and quoted spread bumps. But whether you bump market quotes with curve rebuild or hazard nodes directly changes the computed CS01 and bucket decomposition (model risk).

---

### 1.4 CS01 / Credit DV01 (Spread DV01)

**Formal Definition:**

The reference defines the credit DV01 as the change in value under a +1bp parallel increase in the CDS curve, with a sign convention chosen so that the credit DV01 of a short-protection position is positive:

$$\text{Credit DV01} = -\bigl(V(S + 1\text{bp}) - V(S)\bigr)$$

$$\text{Credit DV01} \approx -\frac{\partial V}{\partial S} \cdot 1\text{bp}$$

**Intuition:**

"DV01-style" reporting often wants positive numbers for positions that lose value when spreads widen (e.g., short protection), hence the leading minus sign.

**How It Appears in Practice:**

Risk systems often provide both:
- a **signed CS01**: $\Delta V$ for +1bp widening (useful for scenario P&L), and
- a **DV01-style magnitude** (positive by convention for specific position directions).

Mixing these conventions is a common failure mode (see §8).

---

### 1.5 Jump-to-Default / Value-on-Default (VOD)

**Formal Definition:**

The reference defines the "value at default" of a CDS (for the protection buyer) as $N(1-R)$, adjusted for accrued premium owed at default, and defines the value on default (VOD) as value-at-default minus current value:

$$\boxed{\text{VOD} = N(1-R) - (\text{accrued premium}) - V}$$

**Intuition:**

Even if a curve trade is "CS01-neutral," default produces a discontinuous payoff that can dominate.

**How It Appears in Practice:**

JTD/VOD is a key stress metric. Hedging spread risk alone does not hedge default-event P&L.

---

### 1.6 Interest-Rate DV01 and Theta (Time Decay Under Fixed Curves)

**Formal Definition:**

Interest rate DV01 is defined analogously (bump discount curve by 1bp, holding credit curve fixed), again with a sign convention in the reference:

$$\text{IR DV01} = -\bigl(V(H + 1\text{bp}) - V(H)\bigr)$$

Theta for CDS is defined as the change in value due to passage of time with curves unchanged:

$$\Theta = -\bigl(V(T - \Delta t) - V(T)\bigr)$$

**Intuition:**

Theta captures "time decay / rolldown" under fixed curves. It is not realized P&L if curves move or if default occurs.

**How It Appears in Practice:**

Theta is used in carry/roll decompositions and in explaining daily P&L, especially for curve positions.

---

### 1.7 Recovery Sensitivity ("Recovery DV01")

**Formal Definition:**

The reference discusses sensitivity to an absolute change of 1% in recovery rate ("recovery DV01").

**Intuition:**

Recovery affects the size of default payoff and (in calibration) can affect inferred hazard rates and therefore $\text{RPV01}$.

**How It Appears in Practice:**

Recovery risk is often "invisible" in spread-only hedges but material in distressed names and event scenarios (auction final price, deliverable price).

---

### 1.8 Basis (Bond–CDS) and Index Basis

**Formal Definition:**

The reference defines the CDS basis as the difference between the quoted CDS spread and a bond spread measure:

$$\text{basis} = S_{\text{CDS}} - S_{\text{Bond}}$$

The reference defines index basis as the difference between index spread and intrinsic spread, and defines a "portfolio swap adjustment" as the difference between the (scaled) index spread and intrinsic spread (index minus intrinsic).

**Intuition:**

"Basis" is not a pure "mispricing" statistic; it reflects funding, liquidity, contract/settlement, deliverability, and technical demand/supply factors.

**How It Appears in Practice:**

Basis trades require explicit controls for non-credit risks (funding/liquidity), instrument-definition risk, and event/settlement risk.

---

## 2. Risk-First Trading Framework (The Organizing Backbone)

This section is an educational workflow: **object → metric → exposures → hedges → failure modes → verification tests**.

### 2.1 Identify the Object Being Traded

Common "objects" (choose one explicitly):

| Object | Description |
|--------|-------------|
| Single-name CDS curve point | e.g., 5y CDS |
| Single-name CDS curve spread | e.g., 5y–1y steepener |
| Index CDS vs intrinsic | index basis / PSA |
| Bond–CDS basis | bond + CDS package |
| Capital structure slice | senior vs subordinated CDS where relative value is linked (conceptually) to recoveries and hazard assumptions; the reference gives a rough relationship and cautions it is "rough in practice" |

---

### 2.2 Define the "Relative Value" Metric

Pick the metric that matches the object:

| Metric | Description |
|--------|-------------|
| **Level** | $S(t,T)$ vs peer/sector/curve model level (educationally: "is spread high/low vs a curve fit?") |
| **Slope / curvature** | $S(t,T_2) - S(t,T_1)$ (steepener/flattener) |
| **Basis** | bond–CDS basis $S_{\text{CDS}} - S_{\text{Bond}}$; index basis = index spread − intrinsic spread |
| **Implied recovery / implied default loss** | implied from auction final price / recovery price at settlement; auctions are referenced in the sources as the mechanism for cash settlement in standard practice descriptions |

**Risk-first rule:** a metric is not "tradeable" unless you can map it into exposures and design hedges/tests.

---

### 2.3 Translate the "Trade Idea" into Explicit Risk Exposures

#### (i) CS01 (Spread DV01) by Maturity Bucket / Curve Node

Define a vector of spread sensitivities:

$$\text{CS01} = \bigl(\text{CS01}_{1y},\; \text{CS01}_{3y},\; \text{CS01}_{5y},\; \ldots\bigr)$$

computed either by:
- **market-quote bumps + curve rebuild**: bump par spreads, recalibrate $Q$ / $h$, reprice, or
- **direct hazard-node / key-rate hazard bumps**: bump hazard buckets, reprice.

**Why bucket exposures matter (rates analogy):** in interest rates, "key rate exposures decompose DV01 into component sensitivities," and more key rates improves hedge quality. Credit curve hedging is structurally analogous (replace yield key rates with credit curve nodes/buckets).

#### (ii) Jump-to-Default (JTD) / VOD

Use VOD/JTD as the default-event stress:

$$\text{VOD} \approx N(1-R) - \text{accrued premium} - V$$

#### (iii) Recovery Sensitivity

Measure PV change under $\Delta R$ (e.g., 1% absolute recovery bump), as discussed as "recovery DV01" in the reference.

#### (iv) Liquidity / Roll / Technical Risks (Conceptual but Source-Consistent)

- **Liquidity & technical factors** are explicitly cited in basis discussions (e.g., liquidity preference for on-the-run maturities like 5y, demand for protection, technical supply/demand).
- **Settlement / deliverability risk:** deliverables and settlement method (physical vs cash/auction) affect realized payoff and can introduce option-like value (delivery option).

---

### 2.4 Specify Hedge Instruments and What Each Hedge Targets

A hedge is defined by what exposure it neutralizes:

| Hedge Target | Method |
|--------------|--------|
| **Hedge CS01 level (parallel shift)** | Use another CDS maturity on the same name, or an index hedge (for broad beta) if that aligns with the intended risk (but basis to index can remain) |
| **Hedge curve shape (bucketed CS01)** | Use multiple maturities to match multiple bucket exposures—analogous to key rate hedging in rates |
| **Hedge JTD** | Requires offsetting default payoff exposure (e.g., offsetting notional in protection buying/selling). Two-leg CS01-neutral curve trades can still be heavily net long/short default (see Example G) |
| **Hedge recovery sensitivity** | Often only partially hedgeable with standard CDS instruments; scenario management is central |

**Recovery swaps:** I'm not sure. The provided sources here discuss recovery in CDS payoff/settlement and recovery sensitivity, but do not provide contract mechanics for recovery swaps. To be certain, we would need (i) recovery swap contract definitions (ISDA product definitions), (ii) market quotation conventions, and (iii) settlement terms.

---

### 2.5 Failure Modes + Verification Tests (Mandatory)

For any RV framework, define in advance:

#### Failure Modes

- **Curve-shape risk** left behind by "parallel CS01 neutral" hedges.
- **Default-event risk:** JTD dominates.
- **Recovery shock / auction surprise:** payout differs from assumed recovery.
- **Liquidity/roll:** hedge instrument liquidity differs (e.g., 5y on-the-run vs off-the-run maturity) and impacts ability to rebalance.
- **Model risk:** quote bump vs hazard bump; discount curve assumptions; accrual at default conventions.

#### Verification Tests

| Test | Description |
|------|-------------|
| Parallel spread shock | $\Delta S(T) = +\delta$ for all $T$ |
| Twist | front-end $+\delta_1$, back-end $+\delta_2$ |
| Recovery shock | $R \to R \pm 10\%$ absolute |
| Default event | apply VOD |
| Repricing check | compare linear CS01 approximation vs full repricing under bumps |

---

## 3. Credit Curve Trades (Steepeners/Flatteners) — Must Include

### 3.1 Define the "Credit Curve"

**Formal Definition:**

A credit curve is the term structure $T \mapsto S(t,T)$ and its equivalent survival/hazard curve $T \mapsto Q(t,T)$ or $h(T)$, with:

$$Q(t,T) = \exp\biggl(-\int_t^T h(s)\,ds\biggr)$$

**Practice:**

Credit curve construction requires interpolation choices; the reference discusses linear interpolation of $-\log Q$ implying piecewise-constant forward hazards.

---

### 3.2 Define Steepener vs Flattener Explicitly in Instruments

Let $T_1 < T_2$, and define:

| Trade | Definition | Instrument Example |
|-------|------------|--------------------|
| **Steepener (in spreads)** | Position that profits if $S(t,T_2) - S(t,T_1)$ increases | Long (buy protection) $T_2$-maturity CDS, short (sell protection) $T_1$-maturity CDS |
| **Flattener (in spreads)** | The opposite exposure | Short $T_2$, long $T_1$ |

**Important:** The "object traded" is a pair of CDS contracts; the risk is a vector of bucket CS01 plus JTD, not a scalar.

---

### 3.3 Designing Curve Trades: Parallel CS01-Neutral vs Target-Slope Exposure

#### (a) Parallel CS01-Neutral Design

Choose notionals $N_1$ (short maturity) and $N_2$ (long maturity) such that:

$$\text{CS01}_{\text{net}} = \text{CS01}(T_2; N_2) + \text{CS01}(T_1; N_1) = 0$$

with signs reflecting buy vs sell protection.

This reduces sensitivity to parallel curve shifts but **does not eliminate twist risk**.

#### (b) Bucketed Exposures ("Credit Key-Rate Style")

Define buckets (e.g., 0–2y, 2–5y, 5–10y). Compute bucket CS01s by bumping the curve only in that bucket (or proxy by splitting $\text{RPV01}$ contributions by time interval, in a simplified setting).

**Rates analogy:** key rate exposures decompose DV01 and enable hedging of curve shapes beyond parallel shifts.

#### (c) "Target Slope Exposure"

After making parallel CS01 near-neutral, the remaining bucket profile is the intended slope exposure (e.g., short front bucket, long belly bucket). You must report it explicitly.

---

### 3.4 Carry and Roll-Down Intuition for CDS Curve Trades (Cautious Decomposition)

**Carry (definition):**
Cash premium accrual/payments from the premium leg over a holding horizon.

**Rolldown (definition):**
Change in mark-to-market from the remaining maturity shortening while the credit curve $S(t,\cdot)$ is assumed unchanged; related to theta defined in the reference via $V(T - \Delta t) - V(T)$.

**Caution:**
Carry/rolldown are not guaranteed positive and are not forecasts. They are conditional decompositions under "curves unchanged" assumptions.

---

### 3.5 Scenario Analysis for Curve Trades

For a two-leg trade (long $T_2$, short $T_1$):

| Scenario | Description |
|----------|-------------|
| **Parallel move** | $\Delta S(T_1) = \Delta S(T_2) = \delta$ |
| **Twist** | $\Delta S(T_1) = \delta_1$, $\Delta S(T_2) = \delta_2$ |
| **Recovery shock** | $R \to R + \Delta R$ |
| **Default event** | apply VOD/JTD as discontinuous payoff shock |

---

## 4. Capital Structure / Recovery Trades (Conceptual) — Must Include

### 4.1 Define "Capital Structure" in This Context

**Formal Definition:**

Capital structure here refers to different claims on the same issuer: senior vs subordinated obligations, and the associated CDS referencing those obligations; also bond vs CDS exposures (cash vs synthetic).

**Practice:**

Different CDS contracts can reference different obligation tiers and have different recoveries and liquidity.

---

### 4.2 Senior vs Subordinated CDS Spreads (Conceptual; Avoid Strict Arb Unless Sourced)

**Source-Backed Conceptual Relationship:**

The reference gives a rough relation: for two CDS contracts on the same name (senior and subordinated) assumed to share default probability, the spread ratio is linked to recoveries:

$$\frac{S_{\text{sub}}}{S_{\text{senior}}} \approx \frac{1 - R_{\text{sub}}}{1 - R_{\text{senior}}}$$

and explicitly notes this is "very rough in practice."

**Risk-First Interpretation:**

This relationship is best treated as a diagnostic about whether differences are plausibly coming from assumed recoveries, liquidity, deliverability, or modeling assumptions—not as a strict arbitrage identity.

---

### 4.3 Bond–CDS Basis: Definition and Drivers (Source-Consistent)

**Definition:**

$$\text{CDS basis} = S_{\text{CDS}} - S_{\text{Bond}}$$

where "bond spread" depends on the spread measure chosen (Z-spread, asset swap spread, OAS, etc.).

**Drivers (qualitative, source-consistent):**

The reference emphasizes multiple factors (fundamental and market/technical), including liquidity differences and maturity points like the 5y as a common liquidity point.

**Risk-First Implications:**

A basis position is exposed to:
- credit (default and spread),
- rates (bond DV01 / curve),
- funding and liquidity (bond financing, repo, balance-sheet constraints),
- settlement/contract differences (deliverability option).

---

### 4.4 Recovery as an Implied Quantity

**Mechanisms:**

Cash settlement recovery can be tied to a "recovery price" determined via auction processes for CDS cash settlement (as described in general terms in the sources).

**Risk Framing:**

Recovery can be treated as implied from auction final price, distressed bond prices, or via pricing assumptions; the key is consistency across analytics.

---

### 4.5 Recovery Trade Framing

**Recovery Sensitivity in CDS PV and in JTD:**
- JTD/VOD depends directly on $(1-R)$.
- PV recovery sensitivity ("recovery DV01") is the PV change for a 1% absolute recovery change.

**Instruments:**

Recovery swaps: I'm not sure (not in provided sources). To be certain, we need recovery swap payoff definition, quotation convention, settlement, and documentation.

---

## 5. Math and Derivations (Step-by-Step)

### 5.1 CDS Valuation Skeleton (Premium Leg vs Protection Leg)

**Assumptions (for clean derivations):**
- Reduced-form survival curve $Q(t,u)$ and discounting $Z(t,u)$.
- Fixed recovery $R$ for pricing.
- Ignore counterparty risk (CVA/DVA) in this chapter.

From the reference (continuous-time form):

$$\text{PV}_{\text{prem}} = S \int_0^T Z(0,t)\,Q(0,t)\,dt$$

$$\text{PV}_{\text{prot}} = (1-R) \int_0^T Z(0,t)\,\lambda(t)\,Q(0,t)\,dt$$

A par spread is determined by equating legs (modulo accrual conventions):

$$S_{\text{par}} \approx (1-R) \frac{\int_0^T Z(0,t)\,\lambda(t)\,Q(0,t)\,dt}{\int_0^T Z(0,t)\,Q(0,t)\,dt}$$

**Unit Check:**
- $S$: 1/year (or bp/year).
- $\int ZQ\,dt$: years.
- So $S \times \int ZQ\,dt$ is dimensionless; multiplying by $N$ gives USD PV.

---

### 5.2 Risky PV01 (RPV01)

For forward-starting CDS, the reference gives a discrete approximation:

$$\boxed{\text{RPV01}(t, t_F, T) = \frac{1}{2} \sum_{n=1}^{N} \Delta(t_{n-1}, t_n)\,Z(t, t_n)\bigl(Q(t, t_{n-1}) + Q(t, t_n)\bigr)}$$

which includes the effect of accrued coupon at default through the trapezoidal survival weighting across payment intervals.

For spot-starting CDS (today to $T$), set $t_F = t$.

**Interpretation:**

$\text{RPV01}$ is "risky annuity": survival-weighted PV of premium payments.

---

### 5.3 Mark-to-Market Value vs Spread Difference

The reference provides:

$$\boxed{V(0,T) \approx \bigl(S(t,T) - S(0,T)\bigr) \cdot \text{RPV01}(t,T)}$$

**Interpretation:**

If you entered at $S(0,T)$ and today's par is $S(t,T)$, the PV is approximately "spread change $\times$ risky PV01" (times notional).

**Unit Check:**
- $\Delta S$ in decimal/year; $\text{RPV01}$ in years; product dimensionless; multiply by $N$ gives USD.

---

### 5.4 CS01 (Credit DV01) Definitions and Computation

The reference defines:

$$\text{Credit DV01} = -\bigl(V(S + 1\text{bp}) - V(S)\bigr)$$

$$\text{Credit DV01} \approx -\frac{\partial V}{\partial S} \cdot 1\text{bp}$$

**Practical Computation Options:**
- **Market-quote bump + survival-curve rebuild:** bump quoted par spreads, rebuild $Q/h$, reprice.
- **Hazard-bucket bumps:** bump hazard in chosen maturity buckets.

**Unit Check:**
- $1\text{bp} = 10^{-4}$ in decimal/year.
- CS01 in USD per bp.

---

### 5.5 Hedge Ratios to Neutralize Exposures

#### (a) Parallel CS01 Neutral (Two Instruments)

For instruments A and B with signed CS01s $\text{CS01}_A$, $\text{CS01}_B$, choose hedge notional ratio:

$$\boxed{w_B = -w_A \frac{\text{CS01}_A}{\text{CS01}_B}}$$

#### (b) Bucketed Neutralization (Credit Key-Rate Style)

Let $i = 1, \ldots, k$ buckets. For $m$ instruments with weights $w_j$:

$$\sum_{j=1}^{m} w_j\,\text{CS01}_{j,i} = 0, \quad i = 1, \ldots, k$$

This is the same linear algebra structure as key-rate hedging in rates: more buckets/key rates means better protection against curve shape changes.

---

### 5.6 P&L Decomposition (First-Order + Jump + Residual)

A risk-first approximation over a short horizon:

$$\boxed{\Delta V \approx \sum_i \text{CS01}_i\,\Delta S_i + \text{RecSens} \cdot \Delta R + \text{JTD scenario} + \text{Residual}}$$

| Component | Description |
|-----------|-------------|
| $\sum_i \text{CS01}_i\,\Delta S_i$ | Spread P&L (bucketed) |
| $\text{RecSens} \cdot \Delta R$ | Recovery sensitivity contribution (model-dependent) |
| JTD scenario | Default event shock via VOD/JTD |
| **Residual sources** | second-order terms (convexity), curve rebuild nonlinearities, discounting changes, basis / liquidity effects, accrual/settlement conventions |

---

## 6. Measurement & Risk (Only What Belongs in Chapter 44)

### 6.1 "Risk Report" View (Template)

For any CDS RV framework, a minimum report includes:

| Risk Category | Metric |
|---------------|--------|
| **Spread risk** | Total CS01 (USD/bp); Bucket CS01s (USD/bp by maturity bucket) |
| **Default risk** | JTD/VOD (USD) under immediate default scenario |
| **Recovery risk** | PV change per +1% recovery (USD per 1% absolute $R$) |
| **Rates risk** | IR DV01 (USD/bp) |
| **Carry / theta** | Theta (USD/day) under curves unchanged |
| **Liquidity / roll / technical** | Qualitative flags: on-the-run maturity liquidity, deliverability/settlement, rebalancing feasibility |

---

### 6.2 Model/Convention Dependence (Must Be Explicit)

| Aspect | Dependence |
|--------|------------|
| **What is bumped?** | par spread quotes vs hazard buckets; each yields different "CS01 decomposition" |
| **Discount curve** | Interest-rate DV01 depends on how discount factors are shocked |
| **Recovery assumptions** | Recovery affects protection payoff and calibration; recovery DV01 must state whether spreads are held fixed and curve recalibrated |
| **Accrual-at-default conventions** | VOD includes accrued premium at default in the reference discussion |

---

### 6.3 Validation Checklist (Scenario Suite + Repricing Tests)

| Test | Description |
|------|-------------|
| **Parallel spread shock** | +10bp, +50bp (all nodes) |
| **Twist** | front-end +10bp, back-end +50bp; and reverse |
| **Recovery shock** | $R \pm 10\%$ absolute (e.g., 40% → 30% / 50%) |
| **Default-event scenario** | apply VOD/JTD |
| **Repricing checks** | Compare linear CS01 approximation vs full repricing under the same shocks |
| **Numerical stability** | bump size sensitivity (0.5bp vs 1bp), curve interpolation robustness |

---

## 7. Worked Examples (At Least 12 Numeric Examples)

### Global Toy Conventions for Examples A–L (Unless Stated Otherwise)

- **Notional** $N$ in USD.
- **Recovery** $R = 40\%$ unless stated otherwise.
- **Discounting:** flat $r = 0\%$ so $Z = 1$ (toy simplification).
- **Survival curve representation** uses:
  - $Q(t,T) = \exp\bigl(-\int_t^T h(s)\,ds\bigr)$ (source-backed)
  - linear interpolation of $-\log Q$ $\Rightarrow$ piecewise-constant forward hazard (source-backed)
  - "credit triangle" approximation $S \approx h(1-R)$ as a quick mapping (source-backed as an approximation)
- **Mark-to-market identity:**
  $$V \approx N\,(S_{\text{mkt}} - c)\,\text{RPV01}$$
  consistent with the reference's "spread difference $\times$ RPV01" relation.
- $1\text{bp} = 10^{-4}$ in decimal.

---

### Example A: Define a CDS Curve

**Given (toy par spreads):**
- $S(1y) = 80\text{bp}$, $S(3y) = 120\text{bp}$, $S(5y) = 160\text{bp}$
- $R = 40\% \Rightarrow 1 - R = 0.60$

**Step 1: Convert spreads to decimals**

$$0.0080,\; 0.0120,\; 0.0160 \text{ per year}$$

**Step 2: "Flat hazard" mapping via credit triangle** $h(T) \approx S(T)/(1-R)$

$$h_1 = 0.0080 / 0.60 = 0.013333$$
$$h_3 = 0.0120 / 0.60 = 0.020000$$
$$h_5 = 0.0160 / 0.60 = 0.026667$$

**Step 3: Convert to survival at tenor points** using $Q(T) = e^{-h(T) \cdot T}$

$$Q(1) = e^{-0.013333 \cdot 1} = 0.9868$$
$$Q(3) = e^{-0.020000 \cdot 3} = e^{-0.06} = 0.9418$$
$$Q(5) = e^{-0.026667 \cdot 5} = e^{-0.13333} = 0.8752$$

**Step 4: Piecewise-constant forward hazards** from linear interpolation of $-\log Q$

Let $H(T) = -\log Q(T)$.
- $H(1) = 0.013333$
- $H(3) = 0.060000$
- $H(5) = 0.133333$

Forward hazards:

$$h_{0 \to 1} = H(1)/1 = 0.013333$$

$$h_{1 \to 3} = \frac{H(3) - H(1)}{3 - 1} = \frac{0.060000 - 0.013333}{2} = 0.023333$$

$$h_{3 \to 5} = \frac{H(5) - H(3)}{5 - 3} = \frac{0.133333 - 0.060000}{2} = 0.036667$$

**Output (simple survival/hazard representation):**
- Tenor survival: $Q(1) = 0.9868$, $Q(3) = 0.9418$, $Q(5) = 0.8752$
- Forward hazards: $h_{0-1} = 1.33\%$, $h_{1-3} = 2.33\%$, $h_{3-5} = 3.67\%$

(We treat this as an input representation, consistent with the chapter instruction that the curve can be considered bootstrapped in prior material.)

---

### Example B: Compute CS01 by Maturity

Compute CS01 for: (i) 1y CDS, (ii) 5y CDS

- Notional $N = 10{,}000{,}000$
- Recovery $R = 40\%$
- Bump method: ±1bp bump to the relevant spread quote; for this toy we hold $\text{RPV01}$ fixed under the bump (first-order).

**Step 1: Compute toy** $\text{RPV01}(T) = \int_0^T Q(t)\,dt$ **(continuous-time toy)**

For 1y with hazard $h_{0-1} = 0.013333$:

$$\text{RPV01}(1) = \frac{1 - e^{-0.013333 \cdot 1}}{0.013333} = 0.9934$$

For 5y using piecewise hazards:

$$\text{RPV01}(5) = \int_0^1 e^{-0.013333t}\,dt + \int_1^3 Q(1)\,e^{-0.023333(t-1)}\,dt + \int_3^5 Q(3)\,e^{-0.036667(t-3)}\,dt$$

Numeric components:
- $0\text{–}1$: $0.9934$
- $1\text{–}3$: $1.9281$
- $3\text{–}5$: $1.8156$

**Total** $\text{RPV01}(5) = 4.7371$

**Step 2: CS01 via ±1bp repricing (first-order)**

Using $V \approx N(S_{\text{mkt}} - c)\,\text{RPV01}$.

For a par CDS at inception, $c = S_{\text{mkt}} \Rightarrow V = 0$. Under a +1bp move, $\Delta S = +0.0001$:

$$V_+ \approx N \cdot 0.0001 \cdot \text{RPV01}$$

Similarly $V_- \approx -N \cdot 0.0001 \cdot \text{RPV01}$.

Thus the signed CS01 is:

$$\text{CS01} = \frac{V_+ - V_-}{2} = N \cdot 0.0001 \cdot \text{RPV01}$$

**1y:**
$$\text{CS01}_{1y} = 10{,}000{,}000 \cdot 0.0001 \cdot 0.9934 = 1000 \cdot 0.9934 = \$993.4/\text{bp}$$

**5y:**
$$\text{CS01}_{5y} = 10{,}000{,}000 \cdot 0.0001 \cdot 4.7371 = 1000 \cdot 4.7371 = \$4{,}737.1/\text{bp}$$

**Unit check:** $N \times 0.0001$ has units USD; multiplying by $\text{RPV01}$ (years) is consistent with the "spread-per-year" unit embedded in $0.0001$.

---

### Example C: Build a Credit Curve Steepener; Parallel CS01-Neutral

**Trade definition:**
- **Long 5y CDS:** buy protection (signed CS01 $+$).
- **Short 1y CDS:** sell protection (signed CS01 $-$).

Choose notionals so net parallel CS01 is zero.

**Given:**
- $\text{CS01}_{5y}$ per 10mm buy protection: $+4{,}737.1/\text{bp}$ (from Example B)
- $\text{CS01}_{1y}$ per 10mm buy protection: $+993.4/\text{bp}$

Let $N_5 = 10\text{mm}$ (buy 5y). Let $N_1$ be the 1y notional (sell 1y).

**Step 1: Write net CS01**
- 5y leg: $+4{,}737.1/\text{bp}$
- 1y sell leg: $-(993.4/\text{bp}) \cdot (N_1 / 10\text{mm})$

Set net = 0:

$$4{,}737.1 - 993.4 \cdot \frac{N_1}{10} = 0 \quad \Rightarrow \quad \frac{N_1}{10} = \frac{4{,}737.1}{993.4} = 4.768$$

**Step 2: Solve**

$$N_1 = 10 \cdot 4.768 = 47.68\text{mm}$$

**Output:**
- Buy protection 5y: $N_5 = 10.00\text{mm}$
- Sell protection 1y: $N_1 = 47.68\text{mm}$
- Net parallel CS01 $\approx 0$

---

### Example D: Steepener P&L Scenarios Using CS01 Buckets

Use Example C trade. For simplicity, treat the trade as having two nodes: 1y and 5y.

**Signed CS01s:**
- 5y buy: $+4{,}737.1/\text{bp}$
- 1y sell 47.68mm: $-4{,}737.1/\text{bp}$ (by construction)

**Approximate P&L:**

$$\Delta V \approx \text{CS01}_{5y}\,\Delta S_{5y} + \text{CS01}_{1y}\,\Delta S_{1y}$$

#### Scenario 1: Parallel widening +20bp all tenors

$\Delta S_{1y} = +20$, $\Delta S_{5y} = +20$

$$\Delta V \approx 4{,}737.1 \cdot 20 + (-4{,}737.1) \cdot 20 = 0$$

**Output:** $\Delta V \approx \$0$

#### Scenario 2: Twist (front +5bp, back +25bp)

$\Delta S_{1y} = +5$, $\Delta S_{5y} = +25$

$$\Delta V \approx 4{,}737.1 \cdot 25 + (-4{,}737.1) \cdot 5 = 4{,}737.1 \cdot 20 = \$94{,}742$$

**Output:** $\Delta V \approx +\$94{,}742$

#### Scenario 3: Flattening (front +25bp, back +5bp)

$\Delta S_{1y} = +25$, $\Delta S_{5y} = +5$

$$\Delta V \approx 4{,}737.1 \cdot 5 + (-4{,}737.1) \cdot 25 = -4{,}737.1 \cdot 20 = -\$94{,}742$$

**Output:** $\Delta V \approx -\$94{,}742$

**Repricing reconciliation (toy):** this equals direct repricing under $V \approx N\,\Delta S\,\text{RPV01}$, which is the same first-order approximation used here.

---

### Example E: Credit Key-Rate Style Buckets

**Define buckets:**
- **Bucket A:** 0–2y
- **Bucket B:** 2–5y
- **Bucket C:** 5–10y

Compute bucket exposures for a 5y CDS with $N = 10\text{mm}$ buy protection.

**Step 1: Compute $\text{RPV01}$ contributions**

From Example A hazards:

**0–1 integral:**
$$\int_0^1 Q(t)\,dt = 0.9934$$

**1–2 integral** (hazard $h_{1-3} = 0.023333$, start survival $Q(1) = 0.9868$):
$$\int_1^2 Q(t)\,dt = Q(1) \cdot \frac{1 - e^{-0.023333 \cdot 1}}{0.023333} \approx 0.9751$$

So:
- **Bucket A (0–2):** $0.9934 + 0.9751 = 1.9685$
- **Total** $\text{RPV01}(5) = 4.7371$ $\Rightarrow$ **Bucket B (2–5):** $4.7371 - 1.9685 = 2.7686$
- **Bucket C:** 0 (5y maturity)

**Step 2: Convert to bucket CS01s**

$$\text{CS01}_{\text{bucket}} = N \cdot 0.0001 \cdot \text{RPV01}_{\text{bucket}}$$

For $N = 10\text{mm}$, multiplier $N \cdot 0.0001 = 1000$.

- **Bucket A:** $1000 \cdot 1.9685 = \$1{,}968.5/\text{bp}$
- **Bucket B:** $1000 \cdot 2.7686 = \$2{,}768.6/\text{bp}$
- **Bucket C:** $0$

**Step 3: Why a 1y hedge doesn't neutralize all bucket risk**

A 1y CDS only loads Bucket A (0–1 subset of 0–2). If you hedge total CS01 with 1y (Example C), you end up with:
- Bucket A net: $+1{,}968.5 - 4{,}737.1 = -2{,}768.6$
- Bucket B net: $+2{,}768.6$

So the "parallel CS01 neutral" trade is actually a **bucket twist exposure**: short front bucket, long belly bucket.

This mirrors the rates insight that DV01 hedging can leave curve exposure; key rate/bucket analysis reveals what remains.

---

### Example F: Carry/Rolldown Decomposition — Cautious

Use Example C steepener. Horizon: $\Delta t = 1/12$ year (≈1 month). Assume:
- hazard curve and discount curve unchanged,
- no default during horizon,
- ignore accrual-at-default and discounting (toy),
- "rolldown" approximated by interpolating the quoted spread curve.

**Step 1: Carry (premium accrual over the month)**

**5y leg (buy protection):** pay $S_{5y} = 160\text{bp} = 0.016/\text{year}$ on 10mm:
- annual premium = $0.016 \cdot 10{,}000{,}000 = 160{,}000$
- 1-month carry ≈ $-160{,}000 / 12 = -13{,}333.33$

**1y leg (sell protection):** receive $S_{1y} = 80\text{bp} = 0.008/\text{year}$ on 47.68mm:
- annual premium = $0.008 \cdot 47{,}680{,}000 = 381{,}440$
- 1-month carry ≈ $+381{,}440 / 12 = +31{,}786.67$

**Net carry:** $31{,}786.67 - 13{,}333.33 = +18{,}453.34$

**Step 2: Rolldown (curve unchanged; maturity shortens)**

5y becomes 4.9167y. Approximate spread at 4.9167y by linear interpolation between 3y (120bp) and 5y (160bp):
- slope = $(160 - 120) / (5 - 3) = 20$ bp per year
- $S(4.9167) \approx 120 + 20 \cdot (4.9167 - 3) = 120 + 38.333 = 158.333\text{bp}$
- spread change from rolldown: $\Delta S_{\text{roll}} = 158.333 - 160 = -1.667\text{bp}$

Rolldown MTM impact (using CS01):
$$\Delta V_{\text{roll},5y} \approx \text{CS01}_{5y} \cdot (-1.667) = 4{,}737.1 \cdot (-1.667) = -\$7{,}911 \text{ (approx)}$$

Assume short-end curve is flat near 1y $\Rightarrow$ $\Delta V_{\text{roll},1y} \approx 0$ (toy).

**Step 3: Total (carry + rolldown)**

$$\Delta V_{\text{carry+roll}} \approx 18{,}453 - 7{,}911 = +\$10{,}542$$

**Caution labels:**
- This is **not** a realized-return forecast. It is a conditional decomposition using theta/rolldown logic (theta defined in the reference).
- Default and curve moves dominate in stressed scenarios (see Example G).

---

### Example G: JTD Sizing for Curve Trades

Use Example C. Assume PV $V \approx 0$ at inception (par trades).

Reference VOD for protection buyer is approximately $N(1-R)$ (minus accrued premium), and VOD = value-at-default minus current value.

Ignore accrued premium (toy).

**5y buy protection, $N_5 = 10\text{mm}$:**
$$\text{JTD}_5 \approx +N_5(1-R) = 10{,}000{,}000 \cdot 0.60 = +\$6{,}000{,}000$$

**1y sell protection, $N_1 = 47.68\text{mm}$:**
$$\text{JTD}_1 \approx -N_1(1-R) = -47{,}680{,}000 \cdot 0.60 = -\$28{,}608{,}000$$

**Net JTD:**
$$\text{JTD}_{\text{net}} \approx 6{,}000{,}000 - 28{,}608{,}000 = -\$22{,}608{,}000$$

**Interpretation:**

The curve steepener is CS01-neutral but has a **large net short-default exposure** (negative JTD). This is the central "risk-first" warning: curve trades can be dominated by default-event risk.

---

### Example H: Bond–CDS Basis Hedge Toy

**Goal:** show a JTD-matched hedge can still have basis/spread residuals.

**Bond position (toy):**
- Face $F = 10\text{mm}$
- Full price $P = 1.02$ (102% of par)
- Recovery $R = 0.40$

Reference states: default risk on a bond purchased at full price $P$ is a loss of $P - R$.

**Step 1: Bond JTD**

Loss fraction = $P - R = 1.02 - 0.40 = 0.62$

Bond JTD $\approx -0.62 \cdot 10{,}000{,}000 = -\$6{,}200{,}000$

**Step 2: CDS notional to match JTD**

A CDS protection buyer receives $(1-R)N$ at default (ignoring accrued premium).

Set $(1-R)N_{\text{CDS}} = 6.2\text{mm}$:
$$0.60 \cdot N_{\text{CDS}} = 6.2 \quad \Rightarrow \quad N_{\text{CDS}} = 10.3333\text{mm}$$

**Step 3: Spread sensitivity (bond vs CDS)**

Assume bond modified duration $D = 4.5$ (toy) and bond PV $= 10.2\text{mm}$.

Bond DV01:
$$\text{DV01}_{\text{bond}} = D \cdot PV \cdot 0.0001 = 4.5 \cdot 10.2\text{mm} \cdot 0.0001 = 4.5 \cdot 1020 = \$4{,}590/\text{bp}$$

CDS CS01 (use $\text{RPV01}(5) = 4.7371$ from Example B):
$$\text{CS01}_{\text{CDS}} = N_{\text{CDS}} \cdot 0.0001 \cdot \text{RPV01} = 10.3333\text{mm} \cdot 0.0001 \cdot 4.7371 \approx 1033.33 \cdot 4.7371 \approx \$4{,}895/\text{bp}$$

#### Scenario 1: Basis move (CDS widens more than bond)

- CDS $\Delta S_{\text{CDS}} = +50\text{bp}$
- Bond credit spread widens $\Delta s_{\text{bond}} = +30\text{bp}$

Bond P&L (spread-only):
$$\Delta V_{\text{bond}} \approx -4{,}590 \cdot 30 = -\$137{,}700$$

CDS P&L (spread-only):
$$\Delta V_{\text{CDS}} \approx +4{,}895 \cdot 50 = +\$244{,}750$$

**Net:**
$$\Delta V_{\text{net}} \approx +\$107{,}050$$

**Interpretation:** JTD hedge $\neq$ basis hedge.

#### Scenario 2: Funding/liquidity shock (conceptual)

Suppose bond price drops 1 point (1%) due to funding/liquidity, CDS unchanged.
- Bond P&L $\approx -0.01 \cdot 10\text{mm} = -\$100{,}000$
- CDS P&L $\approx 0$
- Net $-\$100{,}000$

**Label:** This is a conceptual non-credit driver consistent with the reference discussion that basis reflects multiple factors beyond "pure credit."

---

### Example I: Senior vs Subordinated CDS Conceptual Trade

**Given:**
- 5y senior CDS: $S_{\text{sen}} = 120\text{bp}$, assume $R_{\text{sen}} = 40\%$
- 5y sub CDS: $S_{\text{sub}} = 250\text{bp}$, assume $R_{\text{sub}} = 20\%$
- Notional for sub leg: $N_{\text{sub}} = 10\text{mm}$ (buy protection)
- Hedge senior notional to be parallel-CS01 neutral

**Step 1: Compute toy hazards via credit triangle** $h = S/(1-R)$

- Senior: $h_{\text{sen}} = 0.012 / 0.60 = 0.02$
- Sub: $h_{\text{sub}} = 0.025 / 0.80 = 0.03125$

**Step 2: Compute toy** $\text{RPV01}(5) = (1 - e^{-hT})/h$ **(flat hazard toy)**

**Senior:**
- $e^{-0.1} = 0.904837$
- $\text{RPV01}_{\text{sen}} = (1 - 0.904837) / 0.02 = 4.75815$

**Sub:**
- $e^{-0.15625} \approx 0.85535$
- $\text{RPV01}_{\text{sub}} = (1 - 0.85535) / 0.03125 = 4.62893$

**Step 3: CS01 per 10mm (signed, buy protection)**

- Senior: $1000 \cdot 4.75815 = 4{,}758.15/\text{bp}$
- Sub: $1000 \cdot 4.62893 = 4{,}628.93/\text{bp}$

**Step 4: Choose senior notional $N_{\text{sen}}$ to neutralize parallel CS01**

Trade: buy sub, sell senior. Net CS01 = 0:

$$4{,}628.93 \cdot \frac{N_{\text{sub}}}{10} - 4{,}758.15 \cdot \frac{N_{\text{sen}}}{10} = 0 \quad \Rightarrow \quad N_{\text{sen}} = 10 \cdot \frac{4.62893}{4.75815} = 9.73\text{mm}$$

**Step 5: Remaining exposures**

**Net JTD** (ignore accrued premium; use $(1-R)N$):
- Sub buy JTD $= +10 \cdot 0.80 = +8.00\text{mm}$
- Senior sell JTD $= -9.73 \cdot 0.60 = -5.84\text{mm}$
- Net $= +2.16\text{mm}$ (still material)

**Interpretation:**

Even with CS01 neutrality, you retain default-event and recovery exposures, and the "spread ratio ↔ recovery ratio" relation is only rough in practice per the reference.

---

### Example J: Recovery Sensitivity Comparison

Compute PV at $R = 20\%, 40\%, 60\%$ and compare two positions.

#### Position 1 (off-market 5y CDS)

- Notional $N = 10\text{mm}$
- Contract coupon $c = 100\text{bp} = 0.010$
- Current market spread $S_{\text{mkt}} = 160\text{bp} = 0.016$
- Maturity $T = 5$

Toy pricing: $V \approx N(S_{\text{mkt}} - c)\,\text{RPV01}(R)$ with $\text{RPV01}(R)$ implied by hazard $h = S_{\text{mkt}}/(1-R)$ (flat hazard toy).

Compute $\Delta S = 0.016 - 0.010 = 0.006$.

**For each $R$:**

**$R = 20\%$:** $1 - R = 0.8$, $h = 0.016/0.8 = 0.02$
- $\text{RPV01} = (1 - e^{-0.02 \cdot 5}) / 0.02 = 4.75815$
- $V = 10\text{mm} \cdot 0.006 \cdot 4.75815 = 10\text{mm} \cdot 0.0285489 = \$285{,}489$

**$R = 40\%$:** $h = 0.016/0.6 = 0.0266667$
- $e^{-0.133333} \approx 0.87517$
- $\text{RPV01} = (1 - 0.87517) / 0.0266667 = 4.6810$
- $V = 10\text{mm} \cdot 0.006 \cdot 4.6810 = \$280{,}860$

**$R = 60\%$:** $h = 0.016/0.4 = 0.04$
- $\text{RPV01} = (1 - e^{-0.2}) / 0.04 = 4.53173$
- $V = 10\text{mm} \cdot 0.006 \cdot 4.53173 = \$271{,}902$

**Approx recovery sensitivity (coarse slope over 40% range):**

$$\frac{V(60\%) - V(20\%)}{0.60 - 0.20} = \frac{271{,}902 - 285{,}489}{0.40} = -33{,}968 \text{ USD per 1.00 in } R \Rightarrow -\$340 \text{ per 1\% } R$$

#### Position 2 (off-market 1y CDS)

- $N = 10\text{mm}$
- $c = 50\text{bp} = 0.005$
- $S_{\text{mkt}} = 80\text{bp} = 0.008$
- $T = 1$
- $\Delta S = 0.003$

**$R = 20\%$:** $h = 0.008/0.8 = 0.01$, $\text{RPV01} = (1 - e^{-0.01})/0.01 = 0.99502$
- $V = 10\text{mm} \cdot 0.003 \cdot 0.99502 = \$29{,}851$

**$R = 40\%$:** $h = 0.008/0.6 = 0.013333$, $\text{RPV01} = 0.99336$
- $V = 10\text{mm} \cdot 0.003 \cdot 0.99336 = \$29{,}801$

**$R = 60\%$:** $h = 0.008/0.4 = 0.02$, $\text{RPV01} = 0.99010$
- $V = 10\text{mm} \cdot 0.003 \cdot 0.99010 = \$29{,}703$

**Sensitivity:**

$$\frac{29{,}703 - 29{,}851}{0.40} = -370 \text{ USD per 1.00 in } R \Rightarrow -\$3.7 \text{ per 1\% } R$$

**Conclusion:**

The longer-dated off-market CDS shows materially larger recovery sensitivity in this toy setup, consistent with the idea that recovery impacts default leg sizing and calibration (and thus PV). The reference identifies recovery DV01 as a key sensitivity metric.

---

### Example K: Auction Final Price Shock

Assume CDS cash settlement uses an auction-determined "final price" (as described conceptually in the sources).

**Given:**
- CDS notional $N = 10\text{mm}$
- Expected final price = 40 (i.e., 40% of par)
- Realized final price = 30
- Difference = 10 points

**Step 1: CDS payout difference**

Cash-settled payoff per notional is approximately $1 - \text{final price}$.

- Expected payout = $10\text{mm} \cdot (1 - 0.40) = 6\text{mm}$
- Realized payout = $10\text{mm} \cdot (1 - 0.30) = 7\text{mm}$
- Difference $= +1\text{mm}$ to the protection buyer.

**Step 2: Effect on a hedged bond+CDS package (toy)**

Bond face $F = 10\text{mm}$.
- Expected recovery value = $0.40 \cdot 10 = 4\text{mm}$
- Realized recovery value = $0.30 \cdot 10 = 3\text{mm}$
- Unexpected bond loss = $-1\text{mm}$

If you held long bond + long CDS on the same face:
- bond unexpected $-1\text{mm}$
- CDS unexpected $+1\text{mm}$
- Net ≈ 0.

**Interpretation:**

This is event/recovery risk: the hedge effectiveness depends on settlement mechanics and sizing, and auction outcomes can shift realized P&L materially.

---

### Example L: Full Risk Report and Hedge Plan

**Framework:** 5y vs 1y steepener (Example C), plus optional improvement.

#### L1) Base Position

- Buy 5y protection: $N_5 = 10.00\text{mm}$
- Sell 1y protection: $N_1 = 47.68\text{mm}$
- Recovery $R = 40\%$

#### L2) Exposures Table (Toy)

Use buckets from Example E.

| Exposure | 0–2y bucket | 2–5y bucket | 5–10y bucket | Total |
|----------|-------------|-------------|--------------|-------|
| 5y buy (10mm) CS01 (USD/bp) | +1,968.5 | +2,768.6 | 0.0 | +4,737.1 |
| 1y sell (47.68mm) CS01 (USD/bp) | −4,737.1 | 0.0 | 0.0 | −4,737.1 |
| **Net CS01 (USD/bp)** | **−2,768.6** | **+2,768.6** | **0.0** | **0.0** |

**JTD** (ignore accrued premium; VOD logic):
- 5y buy: $+10 \cdot 0.60 = +6.000\text{mm}$
- 1y sell: $-47.68 \cdot 0.60 = -28.608\text{mm}$
- **Net JTD** $= -22.608\text{mm}$ (large)

**Recovery sensitivity (JTD-only proxy):**

$$\text{JTD} = (1-R)(N_5 - N_1) \quad \Rightarrow \quad \frac{\partial \text{JTD}}{\partial R} = -(N_5 - N_1) = -(10 - 47.68) = +37.68\text{mm}$$

So +1% absolute recovery change increases JTD by $0.01 \cdot 37.68 = 0.3768\text{mm}$ (i.e., +$376,800) in this simplified proxy.

#### L3) Hedge Instruments and Hedge Ratios (What They Target)

**Target:** reduce net JTD while keeping curve-shape exposure

Add a third CDS maturity to create degrees of freedom (credit "key-rate" style), similar in spirit to using more key rates in rates hedging.

**Illustration (toy 3-leg adjustment):** add a 3y leg to make net JTD = 0 and net parallel CS01 = 0.

From Example A:
- $\text{RPV01}(1) = 0.9934$
- $\text{RPV01}(3) = 2.9215$
- $\text{RPV01}(5) = 4.7371$

Let signed notionals (buy protection positive):
- $N_5 = +10\text{mm}$ (given)
- choose $N_1$, $N_3$ so:
  1. JTD neutral $\Rightarrow N_1 + N_3 + N_5 = 0$
  2. CS01 neutral $\Rightarrow N_1\,\text{RPV01}(1) + N_3\,\text{RPV01}(3) + N_5\,\text{RPV01}(5) = 0$

**Solve:**

From (1): $N_1 = -10 - N_3$

Plug into (2):
$$(-10 - N_3) \cdot 0.9934 + N_3 \cdot 2.9215 + 10 \cdot 4.7371 = 0$$
$$-9.934 - 0.9934 N_3 + 2.9215 N_3 + 47.371 = 0 \quad \Rightarrow \quad 1.9281 N_3 + 37.437 = 0 \quad \Rightarrow \quad N_3 = -19.41\text{mm}$$

Then:
$$N_1 = -10 - (-19.41) = +9.41\text{mm}$$

**Resulting 3-leg package (toy):**
- Buy 5y 10.00mm
- Sell 3y 19.41mm
- Buy 1y 9.41mm

Net notional $= 0$ $\Rightarrow$ JTD approx 0; net parallel CS01 approx 0; remaining bucket exposures represent a curve-shape bet.

#### L4) Scenario Validation Table (Base 2-Leg Trade)

| Scenario | $\Delta S_{1y}$ | $\Delta S_{5y}$ | Approx P&L |
|----------|-----------------|-----------------|------------|
| Parallel +20bp | +20 | +20 | $0 |
| Twist (front +5, back +25) | +5 | +25 | +$94,742 |
| Flatten (front +25, back +5) | +25 | +5 | −$94,742 |
| Default event (immediate) | — | — | ≈ −$22.6mm (JTD) |
| Recovery shock +10% | — | — | JTD improves by ≈ +$3.768mm (toy JTD proxy) |

#### L5) Failure Modes Narrative (Base 2-Leg Trade)

1. **Default dominates:** CS01 neutrality does not control JTD; a default-event overwhelms spread P&L (Example G).
2. **Bucket mismatch:** total CS01 neutral still leaves a large twist exposure across 0–2 vs 2–5 buckets (Example E).
3. **Recovery/auction risk:** payoff depends on auction final price and accrued premium conventions.
4. **Liquidity/roll risk:** rebalancing is sensitive to liquidity points (e.g., 5y maturity liquidity) and technical demand/supply.

---

## 8. Practical Notes

### 8.1 "No-Trade-Tips" Rule

Everything in this chapter is framed as:
1. define exposures,
2. choose hedges by targeted exposures,
3. enumerate failure modes,
4. run verification tests.

**No forecasting, signals, or recommendations.**

---

### 8.2 Common Pitfalls

| Pitfall | Description |
|---------|-------------|
| **Mixing CS01 definitions** | quote bump + curve rebuild vs hazard-bucket bump (can give different bucket decompositions) |
| **"CS01-neutral" $\neq$ "curve-risk neutral"** | ignoring curve-shape risk (bucket CS01) |
| **Ignoring JTD/VOD** | curve trades can carry huge default-event exposure (Example G) |
| **Ignoring recovery sensitivity** | auction/final price uncertainty changes realized payout; recovery DV01 matters |
| **Treating bond–CDS basis as pure credit mispricing** | liquidity/technical/funding/settlement features can dominate (source emphasizes multiple drivers) |
| **Inconsistent recovery assumptions across analytics** | bond valuation vs CDS valuation vs hedge sizing |

---

### 8.3 Implementation Pitfalls

- Survival curve bootstrapping sensitivity to recovery and discount inputs.
- Calendar/accrual inconsistencies (quarterly payments, accrual at default).
- Numerical noise in small bumps (0.5bp vs 1bp) and interpolation artifacts.
- Liquidity points and contract rolls: hedges may not remain stable across roll dates (technical risk).

---

### 8.4 Verification Tests (Minimum Suite)

1. Parallel shock test (all maturities).
2. Twist/flattening tests (front vs back).
3. Recovery shock test ($\pm 10\%$ absolute).
4. Default-event scenario (apply VOD/JTD).
5. Repricing checks: linear vs full repricing under same bumps.

---

## 9. Summary & Recall

### 9.1 Ten-Bullet Executive Summary

1. A CDS RV "trade idea" is only meaningful after mapping it to exposures (CS01 buckets, JTD, recovery sensitivity, IR DV01, liquidity/roll).

2. The credit curve can be represented by spreads $S(t,T)$, survival $Q(t,T)$, or hazards $h(T)$, linked by $Q = \exp(-\int h)$.

3. CDS PV can be approximated by $(S_{\text{today}} - S_{\text{entry}}) \times \text{RPV01}$ (times notional).

4. Credit DV01/CS01 is a first-order spread sensitivity; sign conventions differ and must be explicit.

5. "CS01-neutral" curve trades still carry shape risk (bucket CS01) and can carry huge JTD.

6. VOD/JTD captures default-event risk; it is discontinuous and can dominate P&L.

7. Recovery risk matters via payoff $(1-R)$ and via calibration; recovery DV01 must state conventions.

8. Bond–CDS basis is $S_{\text{CDS}} - S_{\text{Bond}}$ and reflects many drivers beyond pure credit.

9. Key-rate style thinking applies: bucket exposures decompose total sensitivity and improve hedging against curve moves.

10. A proper framework includes failure modes and verification tests (parallel/twist/recovery/default/repricing).

---

### 9.2 Cheat Sheet

#### Core Definitions

| Formula | Description |
|---------|-------------|
| $Q(t,T) = \exp\bigl(-\int_t^T h(s)\,ds\bigr)$ | Survival probability |
| $\text{RPV01} = \frac{1}{2}\sum \Delta\,Z(Q_{n-1} + Q_n)$ | Risky annuity (discrete approximation in reference) |
| $V \approx (S_{\text{today}} - S_{\text{entry}})\,\text{RPV01}$ | MTM value (× notional) |
| $\text{Credit DV01} = -(V(S + 1\text{bp}) - V(S))$ | Credit spread sensitivity |
| $\text{IR DV01} = -(V(H + 1\text{bp}) - V(H))$ | Interest rate sensitivity |
| $\Theta = -(V(T - \Delta t) - V(T))$ | Time decay / theta |
| $\text{VOD} \approx N(1-R) - \text{accrued premium} - V$ | Value on default |

#### Hedge Ratio

Parallel CS01 neutral (two legs):

$$\boxed{w_B = -w_A \frac{\text{CS01}_A}{\text{CS01}_B}}$$

#### Exposure Checklist

- [ ] CS01 by bucket/node
- [ ] Total CS01
- [ ] JTD/VOD
- [ ] Recovery DV01
- [ ] IR DV01
- [ ] Theta/carry
- [ ] Liquidity/roll/settlement flags

#### Scenario Suite

- [ ] Parallel +10/+50bp
- [ ] Twist/flatten
- [ ] Recovery ±10%
- [ ] Default event (VOD)
- [ ] Full repricing vs linear check

---

### 9.3 Flashcards (35 Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the premium leg of a CDS? | The stream of periodic spread payments made by the protection buyer until maturity or default. |
| 2 | What is the protection leg of a CDS? | The default-contingent payoff (physical delivery at par or cash settlement against recovery price/auction). |
| 3 | Define survival probability $Q(t,T)$. | $P(\tau > T \mid \mathcal{F}_t)$. |
| 4 | How is $Q(t,T)$ linked to hazard $h$? | $Q(t,T) = \exp\bigl(-\int_t^T h(s)\,ds\bigr)$. |
| 5 | What is $\text{RPV01}$? | Risky PV01 (risky annuity): survival-weighted PV of 1 unit spread payments. |
| 6 | Give the reference's MTM approximation for CDS value. | $V \approx (S_{\text{today}} - S_{\text{entry}})\,\text{RPV01}$ (× notional). |
| 7 | Define credit DV01 in the reference. | $-(V(S + 1\text{bp}) - V(S))$. |
| 8 | Why does the reference include a minus sign in credit DV01? | To make DV01 positive for short-protection positions (sign convention). |
| 9 | What is IR DV01 for a CDS? | $-(V(H + 1\text{bp}) - V(H))$ (discount curve bump). |
| 10 | Define theta for CDS in the reference. | $\Theta = -(V(T - \Delta t) - V(T))$. |
| 11 | What is VOD/JTD? | Value at default minus current value; proxy for jump-to-default risk. |
| 12 | Why can a CS01-neutral curve trade still be risky? | It can have large JTD and bucket CS01 (curve-shape) exposures. |
| 13 | What does "bucketed CS01" mean? | Spread sensitivity decomposed by maturity buckets/nodes (credit key-rate style). |
| 14 | What is the bond–CDS basis? | $S_{\text{CDS}} - S_{\text{Bond}}$ (bond spread measure dependent). |
| 15 | Name one non-credit driver of basis emphasized by sources. | Liquidity/technical demand-supply effects. |
| 16 | Define index basis in the reference. | Index spread minus intrinsic spread. |
| 17 | Define portfolio swap adjustment in the reference. | (Scaled) index spread minus intrinsic spread. |
| 18 | What is a CDS curve steepener (instrument definition)? | Buy longer-maturity CDS and sell shorter-maturity CDS. |
| 19 | What is a CDS curve flattener? | Sell longer maturity and buy shorter maturity (opposite slope exposure). |
| 20 | What does "parallel CS01 neutral" mean? | Net CS01 to a parallel spread shift is ~0. |
| 21 | Why is key-rate logic relevant to credit? | It shows how total sensitivity decomposes and why more buckets improve hedging. |
| 22 | What is the "credit triangle" approximation? | $S \approx h(1-R)$ (approximate). |
| 23 | What is a primary verification test for a curve trade? | Twist/flatten shocks to front vs back spreads. |
| 24 | What happens to CDS payoff if recovery is lower? | Protection payout $(1-R)N$ increases (all else equal). |
| 25 | Why is auction final price important? | It can determine cash settlement recovery/price and thus realized payout. |
| 26 | What is the key risk-first question before any RV position? | "What exposures am I taking (CS01 buckets, JTD, recovery, liquidity)?" |
| 27 | What is a common failure mode in basis trades? | Confusing funding/liquidity effects with pure credit mispricing. |
| 28 | How do you hedge curve-shape risk beyond total CS01? | Match bucketed CS01 exposures using multiple maturities. |
| 29 | What exposure does JTD capture that CS01 does not? | Discrete default jump payoff. |
| 30 | What does "model dependence" mean for CS01? | Results change depending on quote-bump vs hazard-bump and curve rebuild conventions. |
| 31 | What is "rolldown" in CDS terms? | MTM change from moving to shorter maturity on an unchanged curve (theta concept). |
| 32 | What is "carry" in CDS terms? | Premium accrual/receipt over the holding horizon. |
| 33 | What is an example of a technical liquidity point in CDS? | 5y is often cited as a common liquidity point. |
| 34 | Why can senior vs sub spreads differ? | Different recoveries, liquidity, contract terms; the reference links spread ratio to recovery ratio but notes it is rough in practice. |
| 35 | What's the core deliverable from a risk-first framework? | A hedgeable exposure map + scenario test plan + identified failure modes. |

---

## 10. Mini Problem Set (18 Questions)

**1.** Given $R = 40\%$ and $S = 150\text{bp}$, compute credit triangle hazard $h \approx S/(1-R)$.

> **Solution sketch:** Convert $S = 0.015$; $1-R = 0.6$; $h = 0.015/0.6 = 0.025$ (2.5%/yr).

**2.** For flat hazard $h = 2\%$ and $T = 5$, compute toy $\text{RPV01} = (1 - e^{-hT})/h$.

> **Solution sketch:** $hT = 0.10$; $e^{-0.10} = 0.9048$; $\text{RPV01} = (1 - 0.9048)/0.02 = 4.76$.

**3.** For $N = 10\text{mm}$ and $\text{RPV01} = 4.5$, compute CS01 $= N \cdot 0.0001 \cdot \text{RPV01}$.

> **Solution sketch:** $N \cdot 0.0001 = 1000$; CS01 $= 1000 \cdot 4.5 = \$4{,}500/\text{bp}$.

**4.** A 5y CDS buy position has CS01 $+4{,}600/\text{bp}$. What notional of 1y sell with CS01 $+900/\text{bp}$ per 10mm (buy) makes net CS01 zero if 5y is 10mm?

> **Solution sketch:** Need 1y sell CS01 magnitude $= 4{,}600$. Per 10mm buy is 900, so scale $= 4600/900 = 5.111 \Rightarrow N_{1y} = 51.11\text{mm}$ sell.

**5.** Compute net JTD for a two-leg trade: buy 5y $N_5 = 10\text{mm}$, sell 1y $N_1 = 40\text{mm}$, $R = 40\%$.

> **Solution sketch:** Net notional $= 10 - 40 = -30\text{mm}$; JTD $(1-R) \cdot (-30) = -18\text{mm}$.

**6.** Explain (one paragraph) why CS01-neutral does not imply JTD-neutral.

> **Solution sketch:** CS01 scales with RPV01 (maturity-dependent), while JTD scales mainly with notional and $(1-R)$ (weakly maturity dependent), so matching CS01 can require large notional imbalances.

**7.** Define basis risk in a bond–CDS hedge and list two drivers.

> **Solution sketch:** Basis risk is residual P&L when bond spread and CDS spread move differently; drivers include liquidity/funding differences and contract/settlement differences.

**8.** Describe a minimum scenario suite for a CDS curve trade.

> **Solution sketch:** Parallel spread shock, twist/flatten, recovery shock, default event (VOD), repricing vs linear check.

**9.** A CDS protection buyer expects auction final price 40 but realizes 30 on $N = 10\text{mm}$. Compute payoff difference.

> **Solution sketch:** Difference $= (0.40 - 0.30) \cdot N = 0.10 \cdot 10\text{mm} = 1\text{mm}$ to the protection buyer.

**10.** Show how to set up linear equations to match bucket CS01 for three buckets using three CDS maturities.

**11.** Explain why liquidity differences between 5y and 4y CDS can create roll/technical risk in a hedge.

**12.** Using the reference's senior/sub relation, discuss why it is "very rough in practice" and what data you'd need to refine it.

**13.** Define index basis and portfolio swap adjustment and explain the difference.

**14.** Explain how accrual-at-default affects VOD measurement.

**15.** Provide a method to distinguish "curve move" vs "basis move" in daily P&L.

**16.** Discuss model risk between quote-bump CS01 and hazard-bump CS01 in a single-name CDS.

**17.** Design a test to detect numerical noise in CS01 computation.

**18.** Describe how you would validate recovery DV01 in an analytics system (repricing and bump consistency).

*(Solution sketches are provided for 1–9 only, as required.)*

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

- CDS mechanics (premium leg, protection leg, settlement): O'Kane Ch 5–6
- Survival probability and hazard rate framework: O'Kane Ch 3, 6–7
- RPV01 discrete approximation with trapezoidal survival weighting: O'Kane Ch 6
- Credit DV01 / CS01 sign convention: O'Kane Ch 8
- VOD / value-on-default definition: O'Kane Ch 8
- IR DV01 and Theta definitions: O'Kane Ch 8
- Recovery DV01 sensitivity: O'Kane Ch 8
- Bond–CDS basis definition and drivers: O'Kane Ch 8
- Index basis and portfolio swap adjustment: O'Kane Ch 10
- Senior vs subordinated spread ratio relation: O'Kane Ch 8 (noted as "very rough in practice")
- Credit triangle approximation $S \approx h(1-R)$: O'Kane Ch 6
- Auction settlement mechanisms: O'Kane Ch 5

### (B) Reasoned Inference — Note Derivation Logic

- Bucket CS01 decomposition via RPV01 time intervals: derived from RPV01 integral structure
- Hedge ratio formula $w_B = -w_A \cdot \text{CS01}_A / \text{CS01}_B$: direct from CS01 neutrality condition
- P&L decomposition structure: first-order Taylor expansion of valuation formulas
- JTD-neutrality via net notional: derived from VOD formula structure

### (C) Speculation — Flag Uncertainties

- Recovery swaps: I'm not sure. The provided sources discuss recovery in CDS payoff/settlement and recovery sensitivity, but do not provide contract mechanics for recovery swaps. To be certain, we would need (i) recovery swap contract definitions (ISDA product definitions), (ii) market quotation conventions, and (iii) settlement terms.

---

*Last Updated: January 2026*
