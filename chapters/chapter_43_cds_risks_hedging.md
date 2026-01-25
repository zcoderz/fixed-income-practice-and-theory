# Chapter 43: Risks in CDS and Hedging These Risks (CS01, Jump-to-Default, Recovery Risk, and Cash–CDS Basis)

---

## Conventions & Notation

**Position sign:** Unless explicitly stated otherwise:
- **Protection buyer** (long protection) has positive value when credit worsens (spreads widen / default becomes more likely)
- **Protection seller** (short protection) has the opposite sign

**Rates & units:**
- 1 bp $= 10^{-4}$ in decimal spread
- Spreads $S$ are written in decimal per year unless explicitly shown in bp

**Valuation time:** $t$. Maturity $T$. Premium payment dates $t_1, \ldots, t_n$ with year fractions $\Delta_i$.

**Discount factor:** $Z(t,u)$ (risk-free discounting)

**Survival probability:** $Q(t,u) = P(\tau > u \mid \mathcal{F}_t)$ under the pricing measure

**Recovery:** $R \in [0,1]$, fraction of par recovered at default for the reference obligation in the model

**Notional:** $N$ dollars

**Contract spread:** $S_0$ (running coupon in the CDS contract)

**Current par spread:** $S(t,T)$, the market running spread that makes a new CDS have zero PV at time $t$

**Risky annuity / RPV01 object:** $\text{RPV01}(t,T)$, the premium-leg "annuity-like" PV object (defined precisely below)

**Value on default:** $\text{VOD}$, the value conditional on default occurring "now" (defined precisely below)

**CS01 / credit DV01:** first-order PV sensitivity to a spread bump; sign conventions vary by desk

---

## 0. Setup

### Conventions used in this chapter

We treat a CDS as: protection leg (pays $1-R$ on default) minus premium leg (pays running spread until default/maturity), consistent with standard reduced-form CDS valuation and O'Kane's presentation of MTM and the $\text{RPV01}$ object.

"Accrued premium at default" is included in the premium-leg PV via O'Kane's $\text{RPV01}$ approximation and in $\text{VOD}$ as a separate term.

**Important:** desk systems differ on whether CS01 is reported with the protection buyer sign or protection seller sign; O'Kane defines Credit DV01 with a sign choice so it is positive for short protection.

### Notation glossary (symbols + definitions)

| Symbol | Definition |
|--------|------------|
| $t$ | Valuation time |
| $T$ | CDS maturity |
| $t_i$ | Premium payment dates; $\Delta_i$: accrual year fraction between $t_{i-1}$ and $t_i$ |
| $N$ | Notional (USD) |
| $R$ | Recovery assumption used in pricing/risk |
| $Z(t,u)$ | Risk-free discount factor |
| $Q(t,u)$ | Survival probability to $u$ |
| $S_0$ | Contractual running spread on the CDS |
| $S(t,T)$ | Current par spread for maturity $T$ |
| $V(t)$ | PV (USD) of the CDS position at time $t$ |
| $\text{RPV01}(t,T)$ | Risky annuity-like PV object linking spread changes to PV changes |
| $\text{VOD}$ | Value on default (jump-to-default scenario metric) |

---

## Fact Classification

### (A) Verified Facts

- A CDS MTM can be written as
$$V(t) = \bigl(S(t,T) - S(0,T)\bigr) \cdot \text{RPV01}(t,T),$$
in O'Kane's notation for the value of an existing contract struck at $S(0,T)$.

- Premium leg PV is expressed using $\text{RPV01}$ and survival/discounting, and $\text{RPV01}$ can be represented as an integral and approximated as scheduled premium PV plus half-period accrued-at-default PV.

- O'Kane defines Credit DV01 as the change in value for a 1bp parallel increase in the CDS curve, with a sign convention chosen so it is positive for short protection (and also notes the derivative form).

- O'Kane defines Value on Default for a CDS as
$$\text{VOD} = 1 - R - \Delta_0 S_0,$$
reflecting protection payment and accrued premium.

- CDS cash settlement recovery price can be determined by dealer poll or auction; and auctions are used in practice to determine the market price of bonds after a credit event.

- O'Kane describes cash–CDS basis and lists contributing factors (liquidity, technicals, funding, deliverables/settlement, etc.).

### (B) Reasoned Inference

- The practical "risk playbook" for CDS can be organized around (i) first-order spread/curve shocks (CS01/buckets), (ii) discrete default jumps (JTD/VOD), (iii) recovery parameter uncertainty, and (iv) cross-instrument basis effects when hedging with bonds/indices.

- Any CS01 number is meaningless unless you specify what is bumped (quotes vs hazard nodes vs a constant shift) because the implied survival curve can change in different ways.

### (C) Speculation (clearly labeled)

Some desks compute CS01 using an ISDA-standardized curve construction and standard-coupon/upfront conventions; the attached sources here do not provide the full modern standard-coupon regime details. I'm not sure what exact standard your desk uses; to be certain we'd need: (i) the specific ISDA version/market standard, (ii) your curve build inputs and interpolation rules, and (iii) whether risk is "quote bump + rebuild" or "hazard bump" in your system.

---

## 1. Core Concepts (definitions first)

### 1.1 Spread risk (CS01 / credit DV01)

**Formal definition:**

Fix a CDS position with PV $V$. Define CS01 as the change in PV (USD) for a 1bp change in a specified credit spread input:

$$\text{CS01} \equiv \frac{V(\text{bumped}) - V(\text{base})}{1 \text{ bp}}.$$

O'Kane's Credit DV01 is defined for a 1bp parallel increase in the CDS curve, with a sign convention chosen so the number is positive for short protection; equivalently $\text{Credit DV01} = -\frac{\partial V}{\partial S} \times 1 \text{ bp}$.

**Intuition:**

Long protection is like being "long credit volatility / long bad news": if spreads widen (worse credit), the position tends to gain PV; CS01 measures that first-order slope.

**Trading / risk practice:**

- CS01 is the main day-to-day risk for a non-defaulting name.
- It is often reported by maturity bucket ("credit key rates") because curve twists matter.

---

### 1.2 Risky PV01 / $\text{RPV01}$ (premium-leg annuity-like object)

**Formal definition:**

O'Kane defines $\text{RPV01}(t,T)$ as the time-$t$ PV object linking spread differences to CDS MTM:

$$\boxed{V(t) = \bigl(S(t,T) - S(0,T)\bigr) \cdot \text{RPV01}(t,T)} \qquad \text{(O'Kane (6.1))}$$

He expresses $\text{RPV01}$ via survival and discounting integrals and approximations including accrued premium at default.

**Intuition:**

$\text{RPV01}$ is the "risky annuity": it is large when discounting is mild and survival is high (lots of expected premium payments).

**Trading / risk practice:**

- For small spread changes, $\Delta V \approx \text{RPV01} \times \Delta S$ (with sign depending on buyer/seller).
- Many desks call $N \times 10^{-4} \times \text{RPV01}$ (USD per bp) the "RPV01"; naming varies.

---

### 1.3 Jump-to-default (JTD) / Value on Default (VOD)

**Formal definition:**

O'Kane defines Value on Default for a CDS as:

$$\boxed{\text{VOD} = 1 - R - \Delta_0 S_0} \qquad \text{(O'Kane (8.9))}$$

i.e., protection payment $1-R$ minus accrued premium $\Delta_0 S_0$ (per unit notional; multiply by $N$ for USD).

A JTD measure can be reported as either:
- **(Convention 1)** $\text{JTD} = \text{VOD}$ (scenario value if default happens "now"), or
- **(Convention 2)** $\text{JTD} = \text{VOD} - V(t)$ (jump in PV relative to current MTM)

**Intuition:**

Spread moves are (mostly) continuous; default is a discontinuous event.

**Trading / risk practice:**

JTD matters most for short-protection books and for names near distress.

---

### 1.4 Recovery risk

**Formal definition:**

Sensitivity of PV to assumed recovery:

$$\frac{\partial V}{\partial R} \quad \text{or} \quad \Delta V \text{ under a recovery scenario shift}.$$

O'Kane highlights that recovery is uncertain and provides empirical recovery context; market practice must choose recovery assumptions for pricing and curve calibration.

**Intuition:**

Protection payoff is $1-R$; lower $R$ means larger default loss and typically larger protection value (for buyers).

**Trading / risk practice:**

Recovery risk is often managed by scenario analysis and by carefully aligning recovery assumptions across CDS and bond analytics.

---

### 1.5 Curve-shape / term structure of credit risk

**Formal definition:**

A set of CDS par spreads $S(t,T_i)$ across maturities implies a survival curve $Q(t,u)$ (assume you built it in Chapter 42).

"Bucketed credit risk" measures PV sensitivity to shocks at different maturities (analogy to key-rate duration for rates).

O'Kane describes computing hedge ratios by bumping each spread point on the CDS curve while holding other spreads fixed and repricing.

**Intuition:**

A 5y CDS is not only sensitive to "5y spread"; it is sensitive to the whole path of default risk up to 5y, so curve twists generate residual risk if you hedge with a single maturity.

**Trading / risk practice:**

Curve hedges often use multiple maturities (e.g., 3y + 7y hedging a 5y forward exposure).

---

### 1.6 Cash–CDS basis risk

**Formal definition:**

The cash–CDS basis is the divergence between the spread of the cash bond (under some bond spread measure) and the CDS spread on the same issuer.

O'Kane defines and discusses the basis and lists drivers: liquidity, technical factors, differences in credit risk exposure (e.g., non-par bond price changes), funding, and settlement/deliverability.

**Intuition:**

A bond is a funded position with coupons, repo/funding, and specific deliverables; a CDS is unfunded insurance with standardized credit-event terms. These are not identical claims.

**Trading / risk practice:**

Hedging a bond with CDS can remove some default loss exposure but often leaves basis and funding residuals.

---

## 2. Math and Derivations (step-by-step)

### 2.1 CDS PV decomposition and the $\text{RPV01}$ object

**Assumptions (for derivations in this chapter):**

- Reduced-form default time $\tau$ with survival $Q(t,u)$.
- Deterministic discounting $Z(t,u)$ (or independent of default).
- Running spread CDS with contractual spread $S_0$ and premium dates $t_i$.

**Premium leg PV:**

O'Kane writes the premium leg PV as spread times an annuity-like object:

$$\boxed{\text{PV}_{\text{prem}}(t) = S_0 \cdot \text{RPV01}(t,T)} \qquad \text{(O'Kane (6.2))}$$

He defines $\text{RPV01}$ via

$$\boxed{\text{RPV01}(t,T) = \int_t^T Z(t,s) \, Q(t,s) \, ds} \qquad \text{(O'Kane (6.3))}$$

and provides a practical payment-date approximation including accrued-at-default:

$$\boxed{\text{RPV01}(t,T) \approx \sum_{n=1}^{N} \Delta_n Z(t,t_n) Q(t,t_n) + \frac{1}{2} \sum_{n=1}^{N} \Delta_n Z(t,t_n) \bigl(Q(t,t_{n-1}) - Q(t,t_n)\bigr)} \qquad \text{(O'Kane (6.4))}$$

**Unit check:**

- $Z$: unitless discount factor
- $Q$: unitless survival
- $ds$ or $\Delta_n$: years

Thus $\text{RPV01}$ has "years" dimension; multiplying by spread (per year) gives unitless PV per unit notional.

**Protection leg PV (conceptual):**

O'Kane's integral form is:

$$\boxed{\text{PV}_{\text{prot}}(t) = (1-R) \int_t^T Z(t,s) \, \bigl(-dQ(t,s)\bigr)} \qquad \text{(O'Kane (6.5))}$$

**Value for a protection buyer:**

$$V_{\text{buy}}(t) = \text{PV}_{\text{prot}}(t) - \text{PV}_{\text{prem}}(t) = \text{PV}_{\text{prot}}(t) - S_0 \, \text{RPV01}(t,T).$$

---

### 2.2 Par spread and MTM representation

Define the par spread $S(t,T)$ as the spread that makes a new contract have zero PV at time $t$:

$$0 = \text{PV}_{\text{prot}}(t) - S(t,T) \, \text{RPV01}(t,T) \quad \Rightarrow \quad \text{PV}_{\text{prot}}(t) = S(t,T) \, \text{RPV01}(t,T).$$

Plugging into the value of an existing contract struck at $S(0,T)$ yields O'Kane's MTM formula:

$$\boxed{V(t) = \bigl(S(t,T) - S(0,T)\bigr) \cdot \text{RPV01}(t,T)} \qquad \text{(O'Kane (6.1))}$$

**Sanity checks:**

- If current par spread equals contractual spread: $S(t,T) = S(0,T) \Rightarrow V(t) = 0$.
- If spreads widen: $S(t,T) \uparrow \Rightarrow V(t) \uparrow$ for protection buyer (positive slope).

---

### 2.3 CS01 / Credit DV01 definitions and sign conventions

O'Kane defines Credit DV01 for a 1bp parallel increase in the CDS curve:

**Finite difference form (desk sign choice):**

$$\text{Credit DV01} = -\bigl(V(S + 1\text{bp}) - V(S)\bigr),$$

and equivalently $\text{Credit DV01} = -\frac{\partial V}{\partial S} \times 1\text{bp}$, with sign chosen so it is positive for short protection.

**How we will define CS01 in this chapter (default):**

**Default bump definition (market-quote risk):**
- Bump the market par CDS spread curve $S(t,T_i)$ by +1bp (either parallel or at a specific node)
- Rebuild/recalibrate the survival curve $Q(t,\cdot)$ to match bumped quotes (Chapter 42 machinery)
- Reprice the position and compute $\Delta V$

This matches the spirit of O'Kane's curve-bump hedging approach ("bumping each spread while holding others fixed").

**Alternative bump definition (model-risk): hazard-node bump:**
- Hold discount curve fixed
- Bump hazard rate(s) $\lambda_k$ in a bucket and recompute $Q$, then reprice

This produces a different "CS01-like" number (more like a hazard DV01 / credit key-rate).

---

### 2.4 Jump-to-default / VOD

O'Kane defines:

$$\boxed{\text{VOD} = 1 - R - \Delta_0 S_0} \qquad \text{(O'Kane (8.9))}$$

**Interpretation (per unit notional):**

- $1-R$: protection payment
- $\Delta_0 S_0$: accrued premium owed at default (buyer pays, seller receives)

**Define JTD (one practical choice):**

$$\text{JTD} = \text{VOD} - V(t),$$

the PV jump if default happens immediately.

---

### 2.5 Recovery sensitivity (holding survival fixed)

If $Q$ and $Z$ are held fixed, write:

$$V_{\text{buy}}(t) = (1-R) A(t) - S_0 \, \text{RPV01}(t,T), \qquad A(t) \equiv \int_t^T Z(t,s) \bigl(-dQ(t,s)\bigr).$$

Then:

$$\frac{\partial V_{\text{buy}}}{\partial R} = -A(t).$$

**Sanity check:**

Protection buyer loses value if assumed recovery $R$ increases (smaller payoff).

**Important caveat (market-quote calibration):**

If you hold market spreads fixed and recalibrate hazard when $R$ changes, then $Q$ is not fixed and recovery sensitivity changes. The sources here emphasize the link between spread, default intensity, and recovery assumptions (e.g., "credit triangle" intuition).

---

### 2.6 Curve bucket exposures and hedging logic

O'Kane discusses sensitivities to different points on the CDS curve and hedge ratio computation by bumping curve points.

A generic hedge condition for a portfolio of CDS instruments:

$$\sum_j \text{Notional}_j \cdot \text{CS01}_j \approx 0$$

(for the chosen bump definition).

**Analogy to rates DV01 hedging:**

Tuckman defines DV01 as the price change for a 1bp yield shift and gives hedge ratios by matching DV01s.

Credit "key-rate" hedging is conceptually analogous: match bucketed credit sensitivities, not just parallel CS01.

---

## 3. Measurement & Risk (only what belongs in Chapter 43)

### A) Risk taxonomy for CDS

#### 1) Spread risk (CS01 / spread DV01)

**Precise definition (must specify what is bumped):**

**Default in this chapter (market-quote bump + rebuild):**

$$\text{CS01}_{\text{buy}} \equiv V_{\text{buy}}(\text{recalibrated curve from } S + 1\text{bp}) - V_{\text{buy}}(\text{base}),$$

where "$S + 1\text{bp}$" means the chosen market quote(s) are bumped and the survival curve is rebuilt.

**Alternative (hazard-node bump):**

$$\text{CS01}(\lambda_k) \equiv V(\lambda_k + \Delta\lambda) - V(\lambda_k),$$

holding other inputs fixed.

**Alternative (constant shift):** add a constant to all par spreads (parallel curve shock); this is what O'Kane calls a parallel 1bp increase for Credit DV01.

**RPV01 link:**

Using O'Kane's MTM representation:

$$V(t) = \bigl(S(t,T) - S_0\bigr) \cdot \text{RPV01}(t,T),$$

so for small spread shocks under a fixed-$\text{RPV01}$ approximation,

$$\Delta V \approx \text{RPV01}(t,T) \cdot \Delta S.$$

**Unit check:** if $\Delta S = 1\text{bp} = 10^{-4}$, then $\Delta V \approx N \times \text{RPV01} \times 10^{-4}$ dollars.

---

#### 2) Jump-to-default (JTD)

**Definition:**

We define (buyer sign):

$$\text{JTD} = \text{VOD} - V(t),$$

with $\text{VOD} = 1 - R - \Delta_0 S_0$.

**Relation to payoff:**

- Protection payoff is $(1-R)N$
- Accrued premium is $N \Delta_0 S_0$

**Sign:**

- Protection buyer: positive JTD (receives protection payment)
- Protection seller: negative JTD

---

#### 3) Recovery risk

**Definition:**

Recovery sensitivity (one common measure):

$$\text{RecSens} = \frac{\partial V}{\partial R}, \qquad \text{Rec01} \equiv \frac{\partial V}{\partial R} \times 0.01$$

(PV change per +1% recovery).

**Why recovery is uncertain:**

Empirical recoveries vary by seniority and are uncertain ex ante; O'Kane emphasizes recovery variability and model reliance.

**Interaction with hazard inference:**

If you calibrate $Q$ from spreads, changing $R$ typically changes implied hazard/intensity (model-dependent).

---

#### 4) Curve-shape / term structure of credit risk

**Definition:**

Bucketed exposure to nodes $T_i$:

$$\text{CS01}(T_i) = V(\text{bump } S(t,T_i) \text{ by } 1\text{bp, rebuild}) - V(\text{base}).$$

Practical hedge ratios can be computed by bumping one point at a time and repricing (O'Kane's approach).

**Why single-maturity hedges leave residual risk:**

A non-parallel move changes the implied survival curve differently across maturities; matching only parallel CS01 does not immunize against twists.

---

#### 5) Basis risk (cash bond vs CDS)

**Definition:**

Residual P&L caused by divergence between cash-bond spread dynamics and CDS spread dynamics when you hedge one with the other.

**Source-consistent drivers:**

O'Kane lists drivers (liquidity, technicals, funding, differences in credit exposure like bonds not trading at par, settlement/deliverability effects).

Hull notes bond-yield spreads and CDS spreads can be close in some settings, but not identical in general (basis can exist).

---

### B) Hedging instruments and what risk they target

#### Hedging spread risk

**With another CDS maturity on the same name (curve hedge):**

- **Target:** reduce CS01 (parallel) or bucketed CS01 (curve shape)
- **Method:** choose hedge notional(s) so bucket exposures match (bump-and-reprice)
- **Failure mode:** residual curve twist risk if hedge is too sparse (single maturity)

**With a CDS index (proxy hedge):**

- **Target:** hedge systematic credit widening/tightening when the single name is illiquid
- O'Kane notes index basis behavior and that in widening markets investors use long protection index positions to hedge illiquid long credit exposures
- **Failure mode:** idiosyncratic move of the single name vs index; constituent mismatch; index basis

**With cash bonds:**

- **Target:** hedge default loss and (sometimes) spread risk
- **Failure mode:** cash–CDS basis (liquidity, funding, deliverables, non-par effects)

---

#### Hedging JTD

**Notional sizing intuition:**

- Bond JTD (approx): loss to recovery value
- CDS JTD: $(1-R)N$ (minus accrued premium), using O'Kane's VOD notion

**Limitations:**

- Recovery uncertainty (payout depends on realized recovery/auction price)
- Accrued premium at default (small but nonzero)

---

#### Hedging recovery risk

**Recovery swaps:**

I'm not sure. Recovery swaps are not clearly supported as a hedging instrument in the provided excerpts. To be certain we would need a source section explicitly defining recovery swap payoff and market conventions.

**What is supported in the sources:**

O'Kane discusses digital/default swaps (binary payoffs) which can reduce or change recovery dependence (a fixed payment on default versus $1-R$).

**Practical message:**

Recovery risk is often handled by scenario ranges (e.g., auction final price range) and by avoiding inconsistent recovery assumptions between CDS and bond analytics.

---

### C) "What's being bumped?" must be explicit

**Default bump definition used for CS01 in this chapter (market-quote risk):**

1. Choose which market par quote(s) $S(t,T_i)$ to bump
2. Bump by +1bp
3. Rebuild the survival curve (Chapter 42)
4. Reprice and compute $\Delta V$

This matches O'Kane's description of computing hedge ratios by bumping points along the CDS curve and repricing.

**Alternative bump definition 1: direct hazard-node bump (model risk):**

Bump hazard in a bucket; reprice; interpret as "hazard DV01 / credit key-rate."

**Alternative bump definition 2: constant spread shift (parallel):**

Bump the entire CDS curve by +1bp (O'Kane's Credit DV01 idea).

---

### D) Practical P&L explain around credit moves

For a small horizon move (no default), a practical decomposition is:

$$\Delta V \approx \underbrace{\text{CS01} \cdot \Delta s}_{\text{parallel spread move}} + \underbrace{\sum_i \text{CS01}(T_i) \cdot \Delta s_i}_{\text{curve-shape / buckets}} + \underbrace{\text{RecSens} \cdot \Delta R}_{\text{recovery}} + \underbrace{\mathbf{1}_{\{\text{default}\}} \cdot \text{JTD}}_{\text{event}} + \underbrace{\text{Residual}}_{\text{convexity/model/num}}.$$

**Residual includes:**

- Spread convexity / second order
- Differences between bump definitions
- Numerical noise from finite differences
- Basis P&L if hedged with bonds/indices

---

### E) Keep it focused

- We assume the survival curve construction is covered in Chapter 42.
- Indices are used only as hedging instruments, not as a full index/CDO chapter.

---

## 4. Worked Examples (at least 12 numeric examples)

### Shared conventions for Examples B–L (unless otherwise stated)

| Convention | Value |
|------------|-------|
| Valuation time | $t = 0$ |
| Notional | $N = 10{,}000{,}000$ |
| Recovery | $R = 40\%$ unless otherwise stated |
| Risk-free discounting | Flat $r = 3\%$ continuous, so $Z(0,t) = e^{-0.03t}$ |
| Premium schedule | Annual payments at $t = 1, 2, \ldots, T$ with $\Delta = 1$ |
| Accrued-at-default | Approximated using the $\frac{1}{2}\Delta$ term consistent with O'Kane's $\text{RPV01}$ approximation |
| CS01 bump definition | Par-spread bump logic; in computations we use the first-order $\Delta V \approx \text{RPV01} \cdot \Delta S$ approximation implied by O'Kane's MTM formula for small bumps |

**Survival curve (toy, piecewise hazard):**

$$\lambda(t) = \begin{cases} 1.5\% & 0 \leq t \leq 1 \\ 2.0\% & 1 < t \leq 3 \\ 2.5\% & 3 < t \leq 5 \end{cases}$$

---

### Example A (PV and sign conventions): PV of protection buyer vs seller

**Given (toy numbers, already PV'd):**

- Protection leg PV (buyer receives): $\text{PV}_{\text{prot}} = \$520{,}000$
- Premium leg PV (buyer pays): $\text{PV}_{\text{prem}} = \$500{,}000$

**Compute:**

Buyer value:
$$V_{\text{buy}} = \text{PV}_{\text{prot}} - \text{PV}_{\text{prem}} = 520{,}000 - 500{,}000 = \$20{,}000.$$

Seller value:
$$V_{\text{sell}} = -V_{\text{buy}} = -\$20{,}000.$$

**Check (sign symmetry):**

The two sides sum to zero (ignoring counterparty risk / funding).

---

### Example B (CS01 via finite difference): 5y CDS repriced at $s \pm 1$bp

#### Step 1: compute the 5y $\text{RPV01}$ and par spread under the toy curve

Using the shared conventions:

**Discount factors:**

| $i$ | $Z_i = e^{-0.03i}$ |
|-----|-------------------|
| 1 | 0.9704455 |
| 2 | 0.9417645 |
| 3 | 0.9139312 |
| 4 | 0.8869204 |
| 5 | 0.8607080 |

**Survival probabilities:**

| $i$ | $Q_i$ |
|-----|-------|
| 1 | $e^{-0.015} = 0.9851119$ |
| 2 | $Q_1 e^{-0.02} = 0.9656054$ |
| 3 | $Q_1 e^{-0.04} = 0.9464851$ |
| 4 | $Q_3 e^{-0.025} = 0.9231163$ |
| 5 | $Q_3 e^{-0.05} = 0.9003245$ |

**Default probabilities per year:**

| $i$ | $d_i = Q_{i-1} - Q_i$ |
|-----|----------------------|
| 1 | 0.0148881 |
| 2 | 0.0195065 |
| 3 | 0.0191203 |
| 4 | 0.0233688 |
| 5 | 0.0227918 |

**Compute $\sum Z_i d_i$:**

| $i$ | $Z_i d_i$ |
|-----|-----------|
| 1 | 0.0144481 |
| 2 | 0.0183706 |
| 3 | 0.0174746 |
| 4 | 0.0207263 |
| 5 | 0.0196171 |

**Sum** $= 0.0906366$

**Protection leg PV per unit notional:**
$$\text{PV}_{\text{prot}} = (1-R) \sum Z_i d_i = 0.6 \times 0.0906366 = 0.0543820$$

**Compute scheduled premium PV term $\sum Z_i Q_i$:**

| $i$ | $Z_i Q_i$ |
|-----|-----------|
| 1 | 0.9559975 |
| 2 | 0.9093729 |
| 3 | 0.8650223 |
| 4 | 0.8187308 |
| 5 | 0.7749165 |

**Sum** $= 4.3240400$

**Accrued-at-default approximation:**
$$\frac{1}{2} \sum Z_i d_i = 0.5 \times 0.0906366 = 0.0453183$$

Thus:
$$\text{RPV01}_{5y} \approx 4.3240400 + 0.0453183 = 4.3693583$$

**Par spread:**
$$S_{5y} = \frac{\text{PV}_{\text{prot}}}{\text{RPV01}} = \frac{0.0543820}{4.3693583} = 0.0124462 \Rightarrow 124.462 \text{ bp}$$

#### Step 2: set the contract at par

Contract spread $S_0 = 124.462$ bp $\Rightarrow V = 0$ at base.

#### Step 3: reprice at $s \pm 1$bp using O'Kane MTM approximation

Using $V \approx (S - S_0) \cdot \text{RPV01}$ for small bumps:

$$S_+ = S_0 + 1\text{bp} = 0.0124462 + 0.0001$$
$$V_+ \approx (0.0001) \times N \times \text{RPV01} = 0.0001 \times 10{,}000{,}000 \times 4.3693583 = \$4{,}369.36$$

$$S_- = S_0 - 1\text{bp}$$
$$V_- \approx -\$4{,}369.36$$

**Finite-difference CS01 (buyer sign):**
$$\text{CS01} \approx \frac{V_+ - V_-}{2} = \frac{4{,}369.36 - (-4{,}369.36)}{2} = \$4{,}369.36/\text{bp}$$

**Unit check:**
$N \times 1\text{bp} = 10{,}000{,}000 \times 10^{-4} = 1{,}000$.
Multiply by $\text{RPV01} \approx 4.37 \Rightarrow \$4{,}370$ per bp (reasonable magnitude).

---

### Example C (RPV01 link): $\Delta V \approx \text{RPV01} \cdot \Delta s$

Using Example B's $\text{RPV01}_{5y} = 4.3693583$.

Assume a 10bp widening: $\Delta s = +10\text{bp} = 0.0010$.

$$\Delta V \approx N \times \text{RPV01} \times \Delta s = 10{,}000{,}000 \times 4.3693583 \times 0.0010 = \$43{,}693.58$$

**Interpretation:**

Protection buyer gains about $43.7k for a 10bp widening in this simplified setup.

---

### Example D (JTD for protection buyer/seller): immediate default with $R = 40\%$

Use O'Kane's value-on-default formula: $\text{VOD} = 1 - R - \Delta_0 S_0$.

**Assume:**
- $R = 0.40 \Rightarrow 1 - R = 0.60$
- Contract spread $S_0 = 0.0124462$
- Default occurs $\Delta_0 = 0.25$ years after the last premium date (quarter of a year; toy assumption)

**Buyer cashflows at default:**
- Protection payment: $(1-R)N = 0.60 \times 10{,}000{,}000 = \$6{,}000{,}000$
- Accrued premium owed: $\Delta_0 S_0 N = 0.25 \times 0.0124462 \times 10{,}000{,}000 = \$31{,}115.53$

**VOD (buyer):**
$$\text{VOD}_{\text{buy}} \cdot N = 6{,}000{,}000 - 31{,}115.53 = \$5{,}968{,}884.47$$

**VOD (seller):**
$$\text{VOD}_{\text{sell}} \cdot N = -\$5{,}968{,}884.47$$

If current MTM is zero (par contract), then JTD = VOD under the "jump from today" convention.

---

### Example E (Recovery sensitivity): reprice under $R = 20\%, 40\%, 60\%$ (holding survival fixed)

**Convention for this example:**

Hold $Q$ (hazard curve) and $Z$ fixed; vary only $R$ in the payoff factor $(1-R)$.

Use the 5y par contract struck at $R = 40\%$, so $V = 0$ at $R = 40\%$.

**From Example B:**
- $\sum Z_i d_i = 0.0906366$
- $\text{RPV01} = 4.3693583$
- $S_0 = 0.0124462$

**Premium leg PV per unit notional:**
$$\text{PV}_{\text{prem}} = S_0 \cdot \text{RPV01} = 0.0124462 \times 4.3693583 = 0.0543820$$

**Protection leg PV per unit notional at recovery $R$:**
$$\text{PV}_{\text{prot}}(R) = (1-R) \times 0.0906366$$

**Compute PV per unit notional:**

**$R = 20\% \Rightarrow (1-R) = 0.8$:**
$$\text{PV}_{\text{prot}} = 0.8 \times 0.0906366 = 0.0725093$$
$$V = 0.0725093 - 0.0543820 = 0.0181273 \Rightarrow V(N) = 0.0181273 \times 10{,}000{,}000 = \$181{,}273.17$$

**$R = 40\% \Rightarrow V = 0$** (par by construction)

**$R = 60\% \Rightarrow (1-R) = 0.4$:**
$$\text{PV}_{\text{prot}} = 0.4 \times 0.0906366 = 0.0362546$$
$$V = 0.0362546 - 0.0543820 = -0.0181273 \Rightarrow V(N) = -\$181{,}273.17$$

**Finite-difference recovery sensitivity at $R = 40\%$:**

Using symmetric difference with $h = 0.20$:

$$\frac{\partial V}{\partial R} \approx \frac{V(0.60) - V(0.20)}{0.40} = \frac{-0.0181273 - 0.0181273}{0.40} = -0.0906366 \text{ (per unit notional)}$$

So Rec01 (per +1% recovery) for $N = 10$mm:

$$\text{Rec01} \approx (-0.0906366) \times 0.01 \times 10{,}000{,}000 = -\$9{,}063.66$$

**Interpretation:**

A +1% recovery shift reduces protection buyer PV by about $9.1k in this setup.

---

### Example F (Curve risk across maturities): PV of 1y/3y/5y CDS and hazard-bucket sensitivities

#### Step 1: compute par spreads (from the toy curve)

Using the same method as Example B (annual payments + accrued approximation):

**1y:**
- $\text{RPV01}_{1y} = 0.9632215$
- $S_{1y} = 0.0089998 = 89.998$ bp

**3y:**
- $\text{RPV01}_{3y} = 2.7555393$
- $S_{3y} = 0.0109510 = 109.510$ bp

**5y:**
- $\text{RPV01}_{5y} = 4.3693583$
- $S_{5y} = 0.0124462 = 124.462$ bp

Each par CDS has PV $\approx 0$ at base.

#### Step 2: define hazard-bucket bump (model-risk bucket)

- Bump one hazard segment by $\Delta\lambda = +0.001$ (10bp intensity), hold others fixed
- Recompute PV of the CDS holding contractual spread fixed at base par for that maturity
- Report $\Delta V$ in dollars

**Results (protection buyer PV change for $N = 10$mm):**

| Instrument | Hazard bucket bumped | $\Delta V$ for $\Delta\lambda = +0.001$ | $\Delta V$ per 1bp hazard ($/0.0001$) |
|------------|---------------------|----------------------------------------|---------------------------------------|
| 1y par CDS | $\lambda_{0-1}$ | +$5,776.12 | +$577.61 |
| 1y par CDS | $\lambda_{1-3}$ | $0.00 | $0.00 |
| 1y par CDS | $\lambda_{3-5}$ | $0.00 | $0.00 |
| 3y par CDS | $\lambda_{0-1}$ | +$5,766.65 | +$576.67 |
| 3y par CDS | $\lambda_{1-3}$ | +$10,723.77 | +$1,072.38 |
| 3y par CDS | $\lambda_{3-5}$ | $0.00 | $0.00 |
| 5y par CDS | $\lambda_{0-1}$ | +$5,759.40 | +$575.94 |
| 5y par CDS | $\lambda_{1-3}$ | +$10,667.76 | +$1,066.78 |
| 5y par CDS | $\lambda_{3-5}$ | +$9,631.51 | +$963.15 |

**Interpretation:**

- Short-maturity CDS only loads on early hazard
- A 5y CDS has exposure across the curve; hedging with a single short maturity will leave long-end hazard/curve risk

---

### Example G (Hedge spread risk with another maturity): hedge 5y with 3y and show residual curve risk

**Goal:** hedge the parallel CS01 of a 5y protection-buyer position using a 3y CDS on the same name.

**From earlier:**
- $\text{RPV01}_{5y} = 4.3693583$
- $\text{RPV01}_{3y} = 2.7555393$

**CS01 in USD per bp for notional $N$ (buyer sign):**
$$\text{CS01} \approx N \times 10^{-4} \times \text{RPV01}$$

For $N = 10$mm:
- $\text{CS01}_{5y} = 10{,}000{,}000 \times 10^{-4} \times 4.3693583 = \$4{,}369.36/\text{bp}$
- $\text{CS01}_{3y} = 10{,}000{,}000 \times 10^{-4} \times 2.7555393 = \$2{,}755.54/\text{bp}$

**Hedge ratio:**

Let:
- Long 5y protection notional $N_5 = 10$mm
- Take short 3y protection notional $N_3$ to offset CS01

**Condition (parallel CS01 neutral):**
$$\text{CS01}_{5y}(N_5) - \text{CS01}_{3y}(N_3) = 0 \Rightarrow N_3 = N_5 \cdot \frac{\text{RPV01}_{5y}}{\text{RPV01}_{3y}} = 10\text{mm} \times \frac{4.3693583}{2.7555393} = 15.856 \text{ mm}$$

**Residual under a non-parallel move (curve twist):**

**Scenario:**
- 5y spread widens by +5bp
- 3y spread tightens by −5bp

**Approx P&L:**

5y long protection:
$$\Delta V_5 \approx 10\text{mm} \times 4.3693583 \times 0.0005 = +\$21{,}846.79$$

3y short protection with $15.856$mm:
- Long 3y would change by $15.856\text{mm} \times 2.7555393 \times (-0.0005) = -\$21{,}846.79$
- Therefore short 3y changes by $+\$21{,}846.79$

**Net:**
$$\Delta V_{\text{portfolio}} \approx 21{,}846.79 + 21{,}846.79 = +\$43{,}693.58$$

**Message:**

You hedged the parallel component, but a twist can create large residual P&L.

---

### Example H (Hedge a bond with CDS): JTD hedge sizing + residual basis/spread risk

**Bond position:**
- Face $= \$10$mm
- Clean price $= 102.00$ (per $100 par)
- Assume recovery $R = 40 \Rightarrow$ recovery value $\approx 40.00$ per 100 (toy)

**Immediate-default loss proxy:**
$$\text{Loss} = (102 - 40)\% \times 10\text{mm} = 0.62 \times 10{,}000{,}000 = \$6{,}200{,}000$$

**CDS hedge notional to offset JTD:**

CDS default payoff $\approx (1-R)N_{\text{CDS}} = 0.6 N_{\text{CDS}}$.

Set $0.6 N_{\text{CDS}} \approx 6.2$mm:
$$N_{\text{CDS}} = \frac{6.2}{0.6} = 10.333333 \text{ mm}$$

**Residual spread/basis risk scenario (no default):**

**Assume:**
- Bond spread widens by 30bp (treated as yield increase $\Delta y = 0.003$, toy)
- Bond modified duration $D = 4.5$ (toy)
- CDS 5y spread widens by 50bp ($\Delta s = 0.005$)

**Bond price change (Tuckman-style duration approximation):**
$$\Delta P \approx -D \times P \times \Delta y = -4.5 \times 102 \times 0.003 = -1.377$$

**Dollar P&L:**
$$\Delta V_{\text{bond}} \approx (-1.377/100) \times 10{,}000{,}000 = -\$137{,}700$$

**CDS P&L (using $\text{RPV01}_{5y} = 4.3693583$):**
$$\Delta V_{\text{CDS}} \approx N_{\text{CDS}} \times \text{RPV01} \times \Delta s = 10.333333\text{mm} \times 4.3693583 \times 0.005 = \$225{,}750.18$$

**Net:**
$$\Delta V_{\text{net}} \approx 225{,}750.18 - 137{,}700 = +\$88{,}050.18$$

**Interpretation:**

The hedge was sized for default loss, not for mark-to-market spread moves; and bond spreads didn't move one-for-one with CDS spreads (basis). O'Kane lists multiple basis drivers.

---

### Example I (Cash–CDS basis scenario): bond + CDS hedge leaves basis P&L

Use the same positions as Example H.

**Scenario:**
- Bond spread widens by 80bp ($\Delta y = 0.008$)
- CDS spread widens by 40bp ($\Delta s = 0.004$)

**Bond P&L:**
$$\Delta P \approx -4.5 \times 102 \times 0.008 = -3.672 \Rightarrow \Delta V_{\text{bond}} \approx (-3.672/100) \times 10{,}000{,}000 = -\$367{,}200$$

**CDS P&L:**
$$\Delta V_{\text{CDS}} \approx 10.333333\text{mm} \times 4.3693583 \times 0.004 = \$180{,}600.14$$

**Net:**
$$\Delta V_{\text{net}} \approx -367{,}200 + 180{,}600.14 = -\$186{,}599.86$$

**Attribution:**

The residual is primarily a basis move (bond spread moved more than CDS spread), consistent with O'Kane's basis discussion and drivers.

---

### Example J (Index proxy hedge): systematic hedge works, idiosyncratic move breaks it

**Setup (toy but risk-consistent):**

- Single-name 5y CDS: long protection $N_{\text{SN}} = 10$mm with $\text{CS01}_{\text{SN}} = \$4{,}369/\text{bp}$ (from Example B)
- Index 5y CDS: assume $\text{CS01}_{\text{IDX}}(10\text{mm}) = \$3{,}800/\text{bp}$ (toy index annuity)
- Assume spread beta (toy): $\Delta s_{\text{SN}} \approx \beta \, \Delta s_{\text{IDX}}$ with $\beta = 0.8$. (This beta assumption is a practical modeling choice; not a sourced formula.)

**Choose index notional:**

Let index hedge be short protection (so it loses when index spreads widen, offsetting SN gains).

We want:
$$\text{CS01}_{\text{SN}} \cdot \beta \approx \text{CS01}_{\text{IDX}}(N_{\text{IDX}})$$

Since CS01 scales with notional:
$$\text{CS01}_{\text{IDX}}(N_{\text{IDX}}) = 3{,}800 \times \frac{N_{\text{IDX}}}{10\text{mm}}$$

Solve:
$$3{,}800 \times \frac{N_{\text{IDX}}}{10} = 4{,}369 \times 0.8 \Rightarrow N_{\text{IDX}} = 10 \times \frac{3{,}495.2}{3{,}800} = 9.198 \text{ mm}$$

**Scenario 1: broad move (hedge works)**

- Index widens +20bp
- Single name widens $\beta \times 20 = 16$bp

**Single-name P&L:**
$$\Delta V_{\text{SN}} = 4{,}369 \times 16 = +\$69{,}904$$

**Index hedge CS01:**
$$\text{CS01}_{\text{IDX}}(9.198) = 3{,}800 \times 0.9198 = 3{,}495.24$$

**Index short-protection P&L:**
$$\Delta V_{\text{IDX}} = -3{,}495.24 \times 20 = -\$69{,}904.8$$

**Net** $\approx 0$.

**Scenario 2: idiosyncratic (hedge fails)**

- Index widens +10bp
- Single name widens +40bp

**Single-name P&L:**
$$4{,}369 \times 40 = +\$174{,}760$$

**Index hedge P&L:**
$$-3{,}495.24 \times 10 = -\$34{,}952.4$$

**Residual:**
$$+\$139{,}807.6$$

**Practice note:**

O'Kane notes indices are used to hedge illiquid long credit positions in widening markets; proxy hedges leave residual idiosyncratic/basis risk.

---

### Example K (Event/settlement risk preview): auction final price range and payout uncertainty

**Sources:**
- Hull: auctions determine the market price of bonds after a credit event and enable cash settlement
- O'Kane: cash settlement recovery price may be determined by dealer poll or auction process

**Assume:**
- Notional $N = 10$mm
- Auction final price $FP \in [0.30, 0.45]$ (fraction of par)

**Protection payout to buyer (ignoring accrued premium for the moment):**
$$\text{Payout} = (1 - FP) \cdot N$$

**Range:**
- If $FP = 0.45$: payout $= 0.55 \times 10\text{mm} = \$5.5$mm
- If $FP = 0.30$: payout $= 0.70 \times 10\text{mm} = \$7.0$mm

So protection seller faces P&L uncertainty:
$$\text{Seller payout} \in [-\$7.0\text{mm}, -\$5.5\text{mm}]$$

**Include accrued premium (toy):**

If $\Delta_0 = 0.25$ and $S_0 = 124.462$bp, accrued premium is $\$31{,}116$ (from Example D), which slightly shifts the net payout range.

---

### Example L (Full P&L explain): 3 scenarios and decomposition table

**Position:**
- 5y par CDS, protection buyer, $N = 10$mm
- Base MTM $V_0 = 0$
- $\text{CS01} \approx \$4{,}369.36/\text{bp}$ (Example B)
- Recovery sensitivity per +1% recovery: $\text{Rec01} \approx -\$9{,}063.66$ (Example E)
- Hazard-bucket PV changes for +0.001 hazard bump (Example F):
  - $\lambda_{0-1}$: +$5,759.40
  - $\lambda_{1-3}$: +$10,667.76
  - $\lambda_{3-5}$: +$9,631.51

We now run three scenarios and decompose.

**Scenario 1: spread widening only (parallel)**

$\Delta s = +20$bp.

**Spread contribution:**
$$\Delta V_{\text{spread}} \approx 4{,}369.36 \times 20 = +\$87{,}387.20$$

- Curve-shape: 0
- Recovery: 0
- Event: 0
- Residual: 0 (by construction)

**Scenario 2: curve steepening (short tighter, long wider) modeled as hazard twist**

To keep this practical without full curve rebuild:
- $\Delta\lambda_{0-1} = -0.0005$ (−5bp hazard)
- $\Delta\lambda_{1-3} = 0$
- $\Delta\lambda_{3-5} = +0.0010$ (+10bp hazard)

Using linear scaling from Example F:
- $\lambda_{0-1}$: +0.001 gives +$5,759.40 $\Rightarrow$ −0.0005 gives $-\$2{,}879.70$
- $\lambda_{3-5}$: +0.001 gives +$9,631.51

**Curve-shape contribution:**
$$\Delta V_{\text{curve}} \approx -2{,}879.70 + 9{,}631.51 = +\$6{,}751.81$$

- Spread (parallel) contribution: 0 (by scenario design)
- Recovery: 0
- Event: 0

**Scenario 3: recovery shock**

$\Delta R = -10\%$ (from 40% to 30%), survival held fixed (as in Example E).

**Recovery contribution:**
$$\Delta V_{\text{rec}} \approx \text{RecSens} \cdot \Delta R$$

Using $\text{Rec01} = -\$9{,}063.66$ per +1%:

$-10\%$ is $-10$ percentage points, so:
$$\Delta V_{\text{rec}} \approx (-\$9{,}063.66) \times (-10) = +\$90{,}636.60$$

**Reconciliation table:**

| Scenario | Inputs | Spread contrib | Curve contrib | Recovery contrib | Event/JTD contrib | Total approx | Residual |
|----------|--------|----------------|---------------|------------------|-------------------|--------------|----------|
| 1 | +20bp parallel | +87,387.20 | 0.00 | 0.00 | 0.00 | +87,387.20 | 0.00 |
| 2 | $\Delta\lambda_{0-1}=-0.0005$, $\Delta\lambda_{3-5}=+0.001$ | 0.00 | +6,751.81 | 0.00 | 0.00 | +6,751.81 | 0.00 |
| 3 | $\Delta R = -10\%$ | 0.00 | 0.00 | +90,636.60 | 0.00 | +90,636.60 | 0.00 |

**Interpretation:**

- Scenario 1 is dominated by CS01
- Scenario 2 shows "credit key-rate" exposure (curve shape) can create P&L even with no parallel move
- Scenario 3 shows recovery assumptions can move PV materially

---

## 5. Practical Notes

### 5.1 Risk report glossary (desk-neutral)

**CS01 / Credit DV01:**

Must specify:
- Bump size (usually 1bp)
- Bump target (par CDS quote curve, hazard nodes, constant shift)
- Whether survival curve is rebuilt

O'Kane's Credit DV01: 1bp parallel curve bump with sign chosen positive for short protection.

**JTD / VOD:**

- VOD per O'Kane: $1 - R - \Delta_0 S_0$
- JTD may be reported as VOD or VOD minus current MTM (system-dependent)

**Recovery sensitivity:**

PV change per +1% recovery (Rec01) or $\partial V / \partial R$.

**Bucketed / curve risk:**

Sensitivity to bumps at particular maturities (quote-buckets) or hazard-buckets.

O'Kane: bump each spread point holding others fixed for hedge computations.

**Basis:**

Cash–CDS basis: residual P&L due to divergence between bond spread and CDS spread; driven by liquidity/technicals/funding/settlement differences.

---

### 5.2 Common pitfalls

- Mixing CS01 numbers computed by different bump definitions across systems (quote-bump vs hazard-bump)
- Treating CDS spread as "pure default intensity" without acknowledging recovery assumptions and modeling choices
- Forgetting accrued premium at default in event P&L ($\Delta_0 S_0$ term)
- Assuming bond–CDS hedge removes all credit risk: basis remains (liquidity/funding/settlement/non-par effects)
- Using inconsistent recovery assumptions between CDS and bond analytics

---

### 5.3 Implementation pitfalls

- Using different discount curves for pricing vs risk (breaks reconciliation)
- Calendar/schedule errors in premium timing and accrual
- Numerical noise in small-bump CS01 (too small bump, coarse curve rebuild, unstable solver)

---

### 5.4 Verification tests

**Sign check (under this chapter's convention):**

Spread up $\Rightarrow$ protection buyer PV up (CS01 $> 0$).

**Scaling check:**

Doubling notional doubles CS01 and JTD.

**Hedge check:**

A CS01-neutral portfolio has small $\Delta V$ under a small parallel spread shock under the same bump method.

---

## 6. Summary & Recall

### 6.1 Executive summary (10 bullets)

1. CDS risk is dominated by spread risk (CS01) and default jump risk (JTD/VOD).

2. O'Kane expresses CDS MTM via $V(t) = (S(t,T) - S_0) \cdot \text{RPV01}(t,T)$, making $\text{RPV01}$ central to spread risk.

3. $\text{RPV01}$ is the risky premium-leg annuity, including accrued premium at default in practical approximations.

4. CS01 is not unique: it depends on what you bump (quotes vs hazards vs constant shift).

5. O'Kane's Credit DV01 is a 1bp parallel curve bump with a sign convention often positive for short protection.

6. JTD/VOD captures the discrete default event: $\text{VOD} = 1 - R - \Delta_0 S_0$.

7. Recovery risk can be significant and is inseparable from calibration assumptions.

8. Curve-shape risk requires bucketed measures; hedging with one maturity leaves residual twists.

9. Bond–CDS hedges leave cash–CDS basis due to liquidity, funding, and settlement/deliverability differences.

10. For event risk, auction/cash-settlement mechanisms imply payout uncertainty tied to final price / recovery.

---

### 6.2 Cheat sheet (definitions + bump methods + hedge formulas)

**MTM (existing contract):**
$$V(t) = \bigl(S(t,T) - S_0\bigr) \cdot \text{RPV01}(t,T)$$

**Premium leg:**
$$\text{PV}_{\text{prem}}(t) = S_0 \cdot \text{RPV01}(t,T)$$

**RPV01 approximation:**
$$\text{RPV01}(t,T) \approx \sum \Delta Z Q + \frac{1}{2} \sum \Delta Z (Q_{i-1} - Q_i)$$

**VOD / JTD building block:**
$$\text{VOD} = 1 - R - \Delta_0 S_0$$

**CS01 (chapter default):** bump par quote(s) by +1bp, rebuild survival curve, reprice.

**Hedge ratio (two instruments, parallel CS01):**
$$N_2 = N_1 \cdot \frac{\text{CS01}_1}{\text{CS01}_2} \quad (\text{opposite position sign})$$

**Scenario checklist:**
- Spread move: parallel vs twist?
- Curve rebuild rule?
- Recovery assumption and whether recalibrated?
- Default event: auction final price range?
- Basis: bond vs CDS vs index?

---

### 6.3 Flashcards (35 Q/A)

1. **Q:** What is $\text{RPV01}(t,T)$?
   **A:** The premium-leg annuity-like PV object linking spread to PV; used in $V(t) = (S(t,T) - S_0) \cdot \text{RPV01}(t,T)$.

2. **Q:** Write O'Kane's MTM formula for an existing CDS.
   **A:** $V(t) = (S(t,T) - S(0,T)) \cdot \text{RPV01}(t,T)$.

3. **Q:** What is CS01 conceptually?
   **A:** PV change for a 1bp change in a specified credit spread input.

4. **Q:** Why must "what is bumped" be stated for CS01?
   **A:** Different bumps (quote rebuild vs hazard bump) change the survival curve differently.

5. **Q:** What is O'Kane's Credit DV01?
   **A:** PV change for a 1bp parallel increase in the CDS curve, with sign convention often positive for short protection.

6. **Q:** What is VOD for a CDS?
   **A:** $1 - R - \Delta_0 S_0$ per unit notional.

7. **Q:** How does accrued premium affect default payoff?
   **A:** It reduces buyer's net default receipt by $\Delta_0 S_0 N$.

8. **Q:** What is JTD in a P&L explain?
   **A:** The PV jump associated with an immediate default scenario (often VOD minus current MTM).

9. **Q:** If spreads widen, what happens to a protection buyer's PV (typical sign)?
   **A:** It increases (positive CS01 under buyer sign convention).

10. **Q:** What is recovery sensitivity?
    **A:** $\partial V / \partial R$ or PV change under a recovery scenario shift.

11. **Q:** Why can recovery sensitivity depend on calibration?
    **A:** Changing recovery can change implied hazard if spreads are held fixed.

12. **Q:** What is curve-shape credit risk?
    **A:** Sensitivity to non-parallel movements in the term structure of CDS spreads/hazard.

13. **Q:** How does O'Kane suggest computing curve hedge ratios?
    **A:** By bumping each spread on the CDS curve holding others fixed and repricing.

14. **Q:** What is cash–CDS basis?
    **A:** Divergence between bond spread and CDS spread on same issuer.

15. **Q:** Name two drivers of cash–CDS basis from O'Kane.
    **A:** Liquidity and funding (also technicals, deliverables/settlement, etc.).

16. **Q:** Why can a bond+CDS hedge leave residual P&L even without default?
    **A:** Bond and CDS spreads can move differently (basis moves).

17. **Q:** What determines cash settlement recovery price per O'Kane?
    **A:** Dealer poll or auction process.

18. **Q:** What does Hull say about auctions in CDS settlement?
    **A:** Auction determines market price of bonds after a credit event to enable cash settlement.

19. **Q:** What is a proxy hedge using an index?
    **A:** Using index CDS to hedge systematic credit risk of a single-name exposure.

20. **Q:** Why can index hedges fail?
    **A:** Idiosyncratic single-name moves and index basis/constituent mismatch.

21. **Q:** What is the key difference between hedging spread risk and hedging JTD?
    **A:** Spread hedges address continuous MTM changes; JTD hedges address discrete default payout.

22. **Q:** How do you size a CDS hedge to bond JTD loss?
    **A:** Choose CDS notional so $(1-R)N_{\text{CDS}} \approx$ bond loss-to-recovery.

23. **Q:** What is a major pitfall in CS01 reporting?
    **A:** Mixing CS01 computed under different bump/rebuild conventions.

24. **Q:** What is the residual in a P&L explain?
    **A:** Convexity, model differences, numerical noise, and basis effects.

25. **Q:** What is a "bucketed CS01"?
    **A:** PV sensitivity to a 1bp shock at a specific maturity node of the CDS curve.

26. **Q:** What is a hazard-bucket sensitivity?
    **A:** PV sensitivity to bumping hazard rate in a time bucket (model risk view).

27. **Q:** Why is $\text{RPV01}$ smaller for shorter maturities?
    **A:** Fewer expected premium payments (less time, lower survival-weighted PV).

28. **Q:** What happens to $\text{RPV01}$ when credit worsens sharply?
    **A:** It can decrease because survival probability drops (fewer expected premiums).

29. **Q:** What does "clean vs full MTM" refer to in CDS quoting?
    **A:** Whether accrued premium is included separately (affects settlement/P&L).

30. **Q:** What is the simplest recovery-risk management tool if no explicit hedge exists?
    **A:** Scenario analysis over plausible recovery/auction price ranges.

31. **Q:** Why does a bond trading above par increase JTD hedging notional?
    **A:** Default loss exceeds $(1-R) \times$ face.

32. **Q:** What is the most important question before using a CS01 number?
    **A:** "What exactly was bumped and was the curve rebuilt?"

33. **Q:** What risk does a single-maturity CDS hedge not remove?
    **A:** Curve twist risk (term-structure mismatch).

34. **Q:** What risk does a bond+CDS hedge not remove?
    **A:** Cash–CDS basis risk.

35. **Q:** In O'Kane's framework, what links value change to spread change?
    **A:** $\text{RPV01}$ via the MTM formula.

---

## 7. Mini Problem Set (18 questions)

1. A CDS protection buyer has $\text{PV}_{\text{prot}} = \$410k$ and $\text{PV}_{\text{prem}} = \$450k$. Compute PV for buyer and seller.

   *Sketch:* Buyer $V = 410 - 450 = -\$40k$. Seller $+\$40k$.

2. A 5y CDS has $\text{RPV01} = 4.2$ (years). Notional is $20mm. Approximate CS01 (buyer sign) in $ per bp.

   *Sketch:* CS01 $\approx N \times 10^{-4} \times \text{RPV01} = 20{,}000{,}000 \times 10^{-4} \times 4.2 = 2{,}000 \times 4.2 = \$8{,}400/\text{bp}$.

3. Using O'Kane's VOD formula, compute VOD for a buyer with $R = 35\%$, $S_0 = 150$bp, $\Delta_0 = 0.25$, $N = \$5$mm.

   *Sketch:* $1 - R = 0.65$. Accrued $= 0.25 \times 0.015 \times 5{,}000{,}000 = \$18{,}750$. Payout $= 0.65 \times 5{,}000{,}000 = \$3{,}250{,}000$. VOD $= 3{,}250{,}000 - 18{,}750 = \$3{,}231{,}250$.

4. A par 3y CDS has $\text{RPV01} = 2.7$. The par spread widens by 12bp. Approximate $\Delta V$ for $N = \$10$mm protection buyer.

   *Sketch:* $\Delta V \approx 10{,}000{,}000 \times 2.7 \times 0.0012 = 10{,}000{,}000 \times 0.00324 = \$32{,}400$.

5. Explain (in one sentence) why two CS01 numbers can disagree even for the same trade.

   *Sketch:* They can use different bump targets (quotes vs hazard nodes) and different curve rebuild rules.

6. A 5y CDS has CS01 = $5,000/bp (buyer sign). You want to hedge with a 5y index CDS with CS01 = $4,000/bp per $10mm notional. What index notional (short protection) makes you CS01-neutral?

   *Sketch:* Need index CS01 $= 5{,}000$. Notional $= 10\text{mm} \times 5{,}000/4{,}000 = 12.5$mm (short protection).

7. A bond is $10mm face at price 95. Recovery assumption 40. Compute the immediate-default loss proxy.

   *Sketch:* Loss per 100 $= 95 - 40 = 55$. Dollar loss $= 0.55 \times 10\text{mm} = \$5.5$mm.

8. Size CDS notional to hedge the bond in (7) for default loss, using $R = 40\%$.

   *Sketch:* CDS payout $0.6N \approx 5.5\text{mm} \Rightarrow N \approx 9.1667$mm.

9. If Rec01 for a position is $-\$12{,}000$, what is the approximate PV change for a $-5\%$ recovery shock?

   *Sketch:* $-5\%$ is $-5$ percentage points; $\Delta V \approx (-12{,}000) \times (-5) = +\$60{,}000$.

10. Show how a curve steepening can create P&L even if the 5y par spread is unchanged (describe a bucket move).

11. Explain how accrued premium at default affects a protection seller's event P&L.

12. Describe two reasons cash–CDS basis can widen in stressed markets (source-consistent).

13. Given bucketed CS01s at 2y and 5y, set up the linear system to hedge a 3y exposure using 2y and 5y hedges.

14. Why might an index hedge reduce liquidity risk but increase basis risk?

15. Construct a scenario where a bond hedged with CDS makes money even though both spreads widen (basis compression).

16. Describe a verification test to ensure your CS01 calculation is stable numerically.

17. Discuss why recovery risk is difficult to hedge with vanilla instruments if no explicit recovery hedge is available.

18. For a distressed name, explain why linear CS01 may fail to explain daily P&L.

---

## Source Map

### (A) Verified Facts — cite specific sources

- CDS MTM formula, RPV01 definitions and approximations: O'Kane Ch 6
- VOD formula: O'Kane Ch 8 (equation 8.9)
- Credit DV01 definition and sign convention: O'Kane Ch 8
- Cash settlement via auction: O'Kane Ch 5, Hull Ch 25
- Cash–CDS basis and drivers: O'Kane Ch 4, Ch 8
- Hedge ratio computation via curve bumping: O'Kane Ch 8

### (B) Reasoned Inference — note derivation logic

- Risk taxonomy organization follows from the structure of CDS payoffs (premium vs protection legs) and the reduced-form pricing model
- CS01 ambiguity follows from the observation that different bump methods produce different survival curve perturbations
- P&L decomposition formula follows standard first-order Taylor expansion logic applied to the CDS PV function

### (C) Speculation — flag uncertainties

- ISDA standard-coupon regime details and specific desk conventions for CS01 computation are not fully specified in the provided excerpts
- Recovery swap mechanics are not clearly supported in the attached sources
