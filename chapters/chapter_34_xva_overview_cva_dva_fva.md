# Chapter 34: XVA Overview — CVA, DVA, and FVA (Conceptual + Implementation Map)

---

## Conventions & Notation

Time $t$ is measured in years from valuation time $0$. A discrete grid uses endpoints $0 = t_0 < t_1 < \cdots < t_N = T$ with midpoints $t_i^* = (t_{i-1} + t_i)/2$.

| Symbol | Definition |
|--------|------------|
| $f_{\text{nd}}$ | No-default value of the (netting-set) derivatives portfolio (value assuming neither side defaults) |
| $V(t)$ | Market value (MTM) of the netted outstanding portfolio at time $t$ (mid-market unless otherwise stated) |
| $\tau$ | Default time |
| $Q(t)$ | Survival probability $Q(t) = P(\tau > t)$ |
| $\lambda_t$ | Hazard/intensity defined via conditional short-horizon default probability |
| $R$ | Recovery rate $R \in [0,1]$ (dimensionless) |
| LGD | Loss-given-default $= 1 - R$ |
| $D(0,t)$ | Discount factor from the discounting curve (dimensionless) |

**Hazard definition:**

$$P(\tau \in (t, t+h] \mid \tau > t, \mathcal{F}_t) = \lambda_t h + o(h)$$

**Exposure at default** is based on early-termination settlement logic: after an event of default, trades are early-terminated and a settlement amount is computed (often reflecting replacement costs, e.g., half bid–ask).

### Important Convention Note (CVA Sum)

One source presents $\text{CVA} = \sum_i q_i v_i$ with $v_i$ defined as the PV of expected loss at default midpoint, while another presents $\text{CVA} = \sum_i (1-R) \, q_i \, v_i$ with $v_i$ defined as PV of expected exposure (loss separated via LGD).

**In this chapter, when we write discrete-sum formulas in terms of exposure, we will use the LGD-outside convention (Hull-style):**

$$\text{CVA} = \sum_i (1-R) \, q_i \, v_i$$

---

## Setup

### Conventions Used in This Chapter

- **Bilateral OTC context:** We focus on bilaterally cleared derivatives where netting applies under an ISDA master agreement (netting set / counterparty portfolio).

- **Early termination at default:** At an event of default, trades are terminated shortly after and a settlement amount is computed; settlement reflects portfolio value and (in the discussion source) replacement costs approximated by half bid–ask spreads.

- **Risk-neutral valuation for XVA pricing:** For valuation objectives, default probabilities and market-factor simulations are taken in the risk-neutral world (with explicit caveats for "scenario" exposure measures).

- **Collateral modeled via CSA terms:** Exposure under collateral at default is modeled as $E = \max(V - C, 0)$, where $C$ is collateral held at default (negative $C$ means the dealer posted collateral).

- **Cure period / margin period of risk (MPOR):** Collateral at default typically reflects portfolio value some days earlier (10–20 days typical) due to assumed lag between collateral failure and closeout.

- **FVA treatment:** The sources emphasize that FVA is widely used in practice but has weaker theoretical consensus than CVA/DVA; we treat FVA as an overview and clearly separate assumptions.

### Notation Glossary (Symbols + Definitions)

| Symbol | Definition | Units |
|--------|------------|-------|
| $t_i$ | Endpoint of interval $i$ | Years |
| $t_i^* = (t_{i-1} + t_i)/2$ | Midpoint of interval $i$ | Years |
| $\tau$ | Default time (counterparty or own, depending on context) | Years |
| $Q(t)$ | Survival probability $P(\tau > t)$ | Dimensionless |
| $\lambda_t$ | Hazard/intensity process | 1/year |
| $q_i$ | Probability of default during interval $(t_{i-1}, t_i]$ | Dimensionless |
| $R$ | Recovery rate; LGD $= 1 - R$ | Dimensionless |
| $D(0,t)$ | Discount factor | Dimensionless |
| $f_{\text{nd}}$ | No-default portfolio value | Currency |
| $V(t)$ | (Netting-set) portfolio MTM at time $t$ | Currency |
| $C(t)$ | Collateral held at time $t$ (positive if held by dealer, negative if posted by dealer) | Currency |
| $E(t)$ | Positive exposure at time $t$ | Currency |
| CVA | Credit valuation adjustment (cost from counterparty default) | Currency |
| DVA | Debit valuation adjustment (benefit from own default; "mirror image" of CVA) | Currency |
| FVA | Funding valuation adjustment; sometimes decomposed into FCA and FBA | Currency |
| FCA | Funding cost adjustment | Currency |
| FBA | Funding benefit adjustment | Currency |

---

## Core Concepts (Definitions First)

**Meta-idea:** XVAs are "price adjustments" layered on top of a no-default valuation, reflecting credit, funding, and related frictions. They are widely used; the sources explicitly note that some have stronger theoretical foundations than others.

### 1.1 XVA (Family)

**Formal Definition:**

"XVAs" are a collection of valuation adjustments such as CVA, DVA, FVA (and others like MVA/KVA), applied to a no-default derivative value.

**Intuition:**

No-default models assume perfect performance. Real OTC markets have default risk, collateral frictions, and funding costs; XVA attempts to reflect these in price/valuation.

**Practice:**

Banks quote/track XVA at the counterparty level (netting set), manage XVA P&L, and hedge components (especially CVA spread risk).

> **Analogy: The Price Tag Layers**
>
> XVA is like peeling an onion or adding layers to a price tag.
>
> 1.  **Risk-Free Meaning**: The "theoretical" price if everyone was honest and immortal. (e.g., \$100).
> 2.  **CVA Layer (Insurance)**: "But you might not pay me." (Subtract \$2 for insurance cost $\to$ \$98).
> 3.  **FVA Layer (Funding)**: "But I have to borrow cash to hedge this." (Subtract \$1 for interest cost $\to$ \$97).
> 4.  **DVA Layer (My Risk)**: "But *I* might not pay *you*." (Add \$0.50 benefit $\to$ \$97.50).
>
> **The XVA Waterfall**:
> $$ \text{Fair Value} = \text{RiskFree Price} - \text{CVA} - \text{FVA} + \text{DVA} - \text{KVA} - \text{MVA} $$

---

### 1.2 Netting Set and Early Termination (Why Portfolio Matters)

**Formal Definition:**

Under netting, exposure at default "today" is $\max\left(\sum_{i=1}^{N} V_i, 0\right)$ instead of $\sum_{i=1}^{N} \max(V_i, 0)$.

At default, all outstanding derivatives are treated as a single derivative for closeout/settlement under typical master agreements.

**Intuition:**

Defaulting party cannot "cherry-pick" which trades to honor; netting reduces credit losses by offsetting positive and negative MTMs.

**Practice:**

CVA desks compute one CVA per counterparty (netting set), not per trade; incremental pricing requires efficient methods rather than full recomputation.

---

### 1.3 Exposure (With and Without Collateral)

**Formal Definition:**

- **Without collateral:** If portfolio value to dealer is $V$, exposure to counterparty (in loss-from-counterparty-default sense) is $E = \max(V, 0)$ (positive only).

- **With collateral $C$** (collateral posted by counterparty, negative if posted by dealer), exposure is:

$$E = \max(V - C, 0)$$

**Intuition:**

- If the counterparty defaults when $V > 0$, the dealer is an unsecured creditor for the settlement amount (net of collateral), hence potential loss.

- If $V < 0$ and the counterparty defaults, the dealer owes money and typically pays in full (so no loss from counterparty default in that direction).

**Practice:**

Exposure engines simulate $V(t)$ across scenarios, apply CSA collateral rules to infer $C(t)$, then compute $E(t)$ pathwise and average/quantile statistics.

---

### 1.4 CVA

**Formal Definition (Cost):**

CVA is the dealer/bank's estimate of the present value of expected future loss due to counterparty default on positive exposure.

**Discrete-interval framework:** Divide $[0,T]$ into intervals; for each interval compute (i) probability of early termination from default and (ii) PV of expected loss if default occurs at midpoint.

**Intuition:**

CVA is "expected loss priced today": exposure is stochastic, default is stochastic, and you discount losses back to time 0.

**Practice:**

- Requires simulating underlying market variables to estimate expected net exposure conditional on default; accounts for netting and collateral; simplest assumes independence of default and exposure.

- CVA is treated as a complex derivative with risk to credit spreads and to underlying market risk factors; spread risk is often directly measured/hedged.

---

### 1.5 DVA

**Formal Definition (Benefit):**

DVA is the mirror image of CVA: the PV of expected cost to the counterparty from the dealer's possible default; therefore a gain/benefit to the dealer (zero-sum logic).

**Book value under accounting (per source):**

$$f_{\text{nd}} - \text{CVA} + \text{DVA}$$

**Intuition:**

If the dealer defaults, it may avoid paying liabilities on derivatives; that "non-payment option" increases the dealer's value.

**Practice and Caveats:**

- DVA can increase when the dealer's own credit spread widens, generating accounting profits; regulators and practitioners find this economically controversial.

- Implementation mirrors CVA but uses own default probabilities and negative exposure (counterparty's exposure to dealer).

---

### 1.6 FVA (and FCA/FBA If Used)

**Formal Definition (Conceptual):**

FVA reflects the present value impact of funding costs/benefits created by derivatives positions (e.g., funding uncollateralized positive MTM or funding margin).

The sources define:

- **FCA:** PV of expected future funding cost
- **FBA:** PV of expected future funding benefit
- **FVA** reduces derivative value by $\text{FCA} - \text{FBA}$

**Intuition:**

If a derivative is uncollateralized and has positive value to the bank, the bank must finance that receivable; if it has negative value, it may provide funding benefit (depending on whether you recognize/allow that benefit).

**Practice and Controversy:**

- The text flags that economists accept CVA/DVA but have reservations about FVA and similar adjustments (and sometimes DVA).

- FVA depends strongly on internal funding policy (what rate you assume, whether funding benefit is recognized, netting across desks, etc.). The same source notes that FVA can arise whenever a bank enters into an uncollateralized derivative; positive MTM → cost, negative MTM → benefit.

> **Deep Dive: The FVA Debate**
>
> *   **The Theorists (Hull & White)**: "FVA is not real. You shouldn't charge for funding costs any more than you charge for the electricity to run your computers. It's double counting if you already discount at OIS."
> *   **The Practitioners (The Market)**: "I have to borrow cash to hedge this trade. If I don't charge for it, I lose money. I don't care about theory; I care about P&L."
> *   **The Verdict**: The Market won. FVA is standard practice.

### 1.7 KVA (Capital Valuation Adjustment)

**Formal Definition:**
KVA reflects the cost of holding regulatory capital (Basel III / FRTB) against the trade throughout its life.

**Intuition:**
"Rent on Capital." The bank's shareholders demand a return (e.g., 10% ROE) on the capital tied up by the trade. KVA is the PV of this required return. It is often the *largest* adjustment for long-dated trades.

---

## Math and Derivations (Step-by-Step)

### 2.1 Discrete-Interval CVA/DVA Framework (Source-Backed Skeleton)

A widely used discrete framework is:

1. Partition $[0,T]$ into $N$ intervals.

2. For each interval $i$:
   - Compute $q_i$ = (risk-neutral) probability of default in interval $(t_{i-1}, t_i]$
   - Compute $v_i$ = PV of expected exposure or expected loss if default occurs at midpoint $t_i^*$

**Two equivalent presentations in the provided sources are:**

**Loss-embedded convention:**

$$\text{CVA} = \sum_{i=1}^{N} q_i v_i$$

where $v_i$ is PV of expected loss at default midpoint.

**LGD-separated convention:**

$$\boxed{\text{CVA} = \sum_{i=1}^{n} (1-R) \, q_i \, v_i}$$

where $v_i$ is PV of expected exposure at default midpoint and LGD multiplies outside.

**Similarly, DVA can be expressed as:**

$$\text{DVA} = \sum_{i=1}^{N} q_i^* v_i^*$$

with $q_i^*$ and $v_i^*$ based on own default and the counterparty's exposure to the dealer (dealer's negative exposure).

**Unit check:**
- $q_i$ is a probability (unitless)
- $v_i$ is PV of exposure/loss (currency units, e.g., USD)
- CVA is currency units ✓

---

### 2.2 From Portfolio Value to Exposure (With Collateral + Cure Period)

At default time, with collateral $C$ available:

$$\boxed{E = \max(V - C, 0)}$$

where $V$ is the market value of netted transactions to the dealer at default, and $C$ is collateral posted by the counterparty (negative if posted by the dealer).

**Cure period / MPOR modeling (discrete setup):**

If midpoint of interval $i$ is $t_i^*$ and cure period is $c$, simulations must produce $V(t_i^* - c)$ as well as $V(t_i^*)$, because collateral at $t_i^*$ is a CSA function of $V(t_i^* - c)$.

**Unit check:**
- $V$ and $C$ are currency units
- $V - C$ is currency
- $\max(\cdot, 0)$ yields currency ✓

---

### 2.3 "Expected Exposure" Interpretation of $v_i$ (Implementation-Friendly Mapping)

The source states that $v_i$ is commonly computed via Monte Carlo in a risk-neutral world, by averaging exposure at each midpoint and discounting to time 0:

1. Simulate underlying market variables risk-neutrally
2. Compute exposure at each midpoint on each trial
3. Average exposures; set $v_i$ equal to PV of that average exposure

So a convenient operational mapping under the LGD-separated convention is:

$$\boxed{v_i \approx D(0, t_i^*) \cdot \text{EE}(t_i^*)}$$

where $\text{EE}(t) = \mathbb{E}[E(t)]$ under the chosen (risk-neutral) measure for valuation.

**Status vs sources:**

- The books explicitly describe "average exposure across simulation trials" and setting $v_i$ to PV of average exposure.

- The symbol $\text{EE}(t)$ is industry-standard terminology; the sources use "average exposure" rather than the abbreviation "EE". (We use EE as a label for that quantity.)

---

### 2.4 Default Modeling Primitives: $q_i$, Hazard, Survival

#### (i) Intensity/Hazard Definition

The hazard/intensity $\lambda_t$ is defined by:

$$\boxed{P(\tau \in (t, t+h] \mid \tau > t, \mathcal{F}_t) = \lambda_t h + o(h)}$$

**Interpretation:** Conditional default probability over a short horizon $\approx \lambda_t \, h$.

**Unit check:** $\lambda_t$ must be 1/time (e.g., 1/year) so that $\lambda_t h$ is dimensionless probability ✓

#### (ii) Piecewise-Constant Hazard → Survival and Interval Default Probability

If $\lambda$ is constant over an interval of length $\Delta t$, a standard survival form is:

$$\boxed{Q(t + \Delta t) = Q(t) \, e^{-\lambda \Delta t}}$$

This exponential survival appears explicitly in the CDS valuation exposition where survival is $e^{-\lambda t}$ under constant hazard.

Then the default probability in $(t_{i-1}, t_i]$ is:

$$\boxed{q_i = Q(t_{i-1}) - Q(t_i)}$$

This "difference of survivals" is shown in the same CDS setup as "probability of default during year $t$" equals survival to beginning minus survival to end.

#### (iii) Credit Spread → Hazard (Approximation in Source)

A source provides an approximation relating hazard to credit spreads and recovery:

$$\boxed{\bar{\lambda}(T) = \frac{s(T)}{1-R}}$$

where $s(T)$ is a credit spread for maturity $T$.

This underlies a convenient mapping for incremental default probabilities in the CVA sum (noting that real-world implementations often bootstrap full CDS curves).

#### (iv) Credit Spreads at Multiple Maturities → Discrete $q_i$

One source gives:

$$\boxed{q_i = \exp\left(-\frac{s_{i-1} t_{i-1}}{1-R}\right) - \exp\left(-\frac{s_i t_i}{1-R}\right)}$$

and then

$$\text{CVA} = \sum_{i=1}^{n} (1-R) \, q_i \, v_i$$

---

### 2.5 Putting It Together: "Discrete Sum" CVA/DVA in Exposure Form

Under the LGD-separated convention, with midpoint exposures:

$$\boxed{\text{CVA} \approx \sum_{i=1}^{N} (1-R_{\text{cp}}) \, D(0, t_i^*) \, \text{EE}(t_i^*) \, q_i^{\text{cp}}}$$

where:
- $R_{\text{cp}}$ is counterparty recovery rate (recovery to the bank as unsecured creditor)
- $q_i^{\text{cp}}$ is risk-neutral probability counterparty defaults in interval $i$

**Similarly:**

$$\boxed{\text{DVA} \approx \sum_{i=1}^{N} (1-R_{\text{own}}) \, D(0, t_i^*) \, \text{ENE}(t_i^*) \, q_i^{\text{own}}}$$

where $\text{ENE}(t)$ is expected negative exposure (counterparty's exposure to dealer).

**Sign convention (value):**

$$\boxed{\text{Value with default} = f_{\text{nd}} - \text{CVA} + \text{DVA}}$$

**Sanity checks:**
- If $\text{EE}(t) = 0$ for all $t$, then $\text{CVA} = 0$ ✓
- If hazard/PD increases (higher $q_i$), CVA increases ✓
- If recovery $R$ increases (LGD decreases), CVA decreases ✓

---

### 2.6 Discounting: Which Curve?

The sources give the following practical guidance:

- "Risk-free rates are best determined from the OIS curve."

- The Fed funds rate is described as a proxy for risk-free in USD and, for fully collateralized trades, "interest rates are assumed to correspond to the collateral remuneration rate."

- For CVA valuation, one source explicitly states: simulate market variables in the risk-neutral world and discount at the risk-free rate.

**Implementation consequence:**

A consistent "pricing XVA" setup often uses an OIS-based discount curve (as "risk-free proxy"), especially when collateral remunerates overnight/OIS-like rates.

If your desk discounts on a different curve (e.g., legacy LIBOR discounting), you must reconcile that with collateral remuneration and with FVA assumptions to avoid double counting (see Section 3D).

---

## Measurement & Risk (Only What Belongs in Chapter 34)

### A) What Each XVA "Prices"

| XVA | What It Prices | Key Inputs |
|-----|----------------|------------|
| **CVA** | PV of expected future loss to the bank from counterparty default | Positive exposure, counterparty PD, counterparty LGD |
| **DVA** | PV of expected future loss to counterparty from bank default; a *gain* to the bank | Negative exposure, own PD, own LGD |
| **FVA** | Funding costs/benefits from derivative-driven financing | Funding requirement, funding spread, policy on FBA |

**CVA (counterparty default on positive exposure):** PV of expected future loss to the bank from counterparty default. Requires probabilities of early termination/default and expected loss conditional on default at midpoints.

**DVA (own default on negative exposure):** PV of expected future loss to counterparty from bank default; therefore a gain to the bank. The book value is $f_{\text{nd}} - \text{CVA} + \text{DVA}$.

> **Caveat:** DVA can increase as the bank's own credit spread increases, generating accounting profit; this is controversial and regulators have acted to limit its effect in regulatory capital definitions.

**FVA (funding costs/benefits from derivative-driven financing):** Adjustment reflecting funding costs/benefits created by uncollateralized derivatives and/or margin. Defined conceptually with FCA/FBA and FVA reduces value by $\text{FCA} - \text{FBA}$.

> **Caveat:** The source explicitly states economists are comfortable with CVA/DVA but have reservations about FVA (and similar).

---

### B) The Building Blocks (Re-Defined Briefly)

#### Portfolio Value and Exposures

| Quantity | Formula | Description |
|----------|---------|-------------|
| Portfolio value | $V(t)$ | MTM of netted portfolio at time $t$ |
| Exposure without collateral | $E(t) = \max(V(t), 0)$ | Bank has loss only when portfolio is positive to bank at counterparty default |
| Exposure with collateral | $E(t) = \max(V(t) - C(t), 0)$ | Collateral reduces exposure |
| Negative exposure (for DVA) | $\text{NE}(t) = \max(-V(t), 0)$ | Counterparty's exposure to dealer (dealer owes money) |

#### Exposure Statistics (EE, PFE, EPE/ENE)

**EE($t$) / "average exposure at time $t$":** $\text{EE}(t) = \mathbb{E}[E(t)]$. The sources describe computing exposure per Monte Carlo trial at each midpoint and averaging to obtain an average exposure used in $v_i$.

**PFE / peak exposure:** The sources explicitly define "peak exposure at a midpoint" as a high percentile of the simulated exposure distribution (e.g., 97.5%).

**EPE/ENE:** I'm not sure whether the provided sources use the Basel-style terms "EPE/ENE" explicitly (they emphasize "average exposure" and "peak exposure").

> If you want these metrics in an implementation: a common industry definition is time-averaged expected exposure (EPE) and time-averaged expected negative exposure (ENE), but you should confirm the exact definition used by your organization/regulator (time-weighting, survival conditioning, etc.).

#### Default Modeling Primitives

- **Survival $Q(t)$, hazard $\lambda_t$:** Hazard defined via short-horizon conditional default probability.

- **Interval default probability:** $q_i = Q(t_{i-1}) - Q(t_i)$ (difference of survivals) is consistent with the survival-based default probability logic used in the CDS exposition.

- **Recovery $R$, LGD $= 1-R$:** LGD multiplies exposure in expected loss formulas (explicit in $\text{CVA} = \sum(1-R) \, q_i \, v_i$).

#### Discounting

- For valuation, simulate risk-neutral and discount at risk-free.
- "Risk-free" proxy is often OIS; the text states risk-free rates are best determined from OIS.
- For fully collateralized trades, rates correspond to collateral remuneration (Fed funds as proxy in USD context).

---

### C) Implementation Map (Key Deliverable)

**Goal:** Implement end-to-end desk/risk workflow to produce CVA/DVA/FVA numbers and explain daily P&L.

#### Step 1: Define Legal/Portfolio Scope

- Identify netting sets (ISDA master agreement scope) and portfolio membership
- Capture early termination logic and settlement conventions (including replacement bid–ask adjustments if used)
- CSA terms: thresholds, minimum transfer amount, independent amount / initial margin, eligible collateral, haircuts, rehypothecation status (if relevant)

#### Step 2: Define Collateral Mechanics Precisely

- Margin frequency (daily vs less frequent), collateral currencies, interest on cash collateral (often close to overnight rates; CCP margin interest often near Fed funds for USD)
- Cure period / MPOR $c$ (often 10–20 days)
- Exposure at default uses $E = \max(V - C, 0)$

#### Step 3: Generate Future Market Scenarios

- Choose model(s) for relevant market risk factors (rates/FX/equities/commodities/vols) and simulate under risk-neutral dynamics for valuation CVA/DVA (per source guidance)
- Choose time grid $\{t_i^*\}$ and ensure it captures product features (exercise dates, resets, payment dates)

#### Step 4: Revalue the Portfolio in Each Scenario

- For each path and each midpoint $t_i^*$, compute portfolio MTM $V(t_i^*)$
- For collateral modeling with cure period, also compute $V(t_i^* - c)$ to infer collateral at $t_i^*$

#### Step 5: Compute Exposures and Exposure Statistics

- **Pathwise net exposure:** $E(t_i^*) = \max(V(t_i^*) - C(t_i^*), 0)$
- **Expected exposure $\text{EE}(t_i^*)$:** Average across Monte Carlo trials (source describes average exposure → $v_i$)
- **Peak exposure/PFE:** High percentile at each midpoint (e.g., 97.5%)

#### Step 6: Compute Default Probabilities

- Build counterparty risk-neutral default curve (from credit spreads / CDS / bond-implied). Source uses credit spreads and recovery to infer default probabilities; it gives a spread-to-hazard approximation $\bar{\lambda} = s/(1-R)$
- Convert to interval probabilities $q_i$ (e.g., via survival differences $Q(t_{i-1}) - Q(t_i)$ or via provided spread-based formula)

#### Step 7: Assemble CVA and DVA

- **CVA:** Combine EE, LGD, discounting, and $q_i$ via discrete sum consistent with your convention
- **DVA:** Same mechanics with own default probabilities and negative exposure. DVA can be calculated "at the same time as CVA" in the same framework

#### Step 8: Add Funding Assumptions to Compute FVA (Overview-Level)

- Identify what creates funding requirement: uncollateralized positive MTM, posted collateral/margin, etc.
- Choose funding spread and whether funding benefit is recognized (FBA). Source defines FCA/FBA and FVA reduces value by FCA–FBA
- **Note:** The sources emphasize theoretical debate; treat implementation as policy-driven (Section 3D)

#### Step 9: Produce Sensitivities and Explain P&L

- **CVA spread sensitivity:** Use relationship between $q_i$ and credit spreads (e.g., $q_i$ changes with $s_i$)
- **Exposure Greeks:** Sensitivities of $v_i$ to market variables (rates/FX/etc.). The source notes CVA is a complex derivative and Greeks can be computed (adjoint differentiation mentioned as a technique)
- **Incremental XVA:** Store simulation paths and valuations to compute incremental effects without full rerun (both RMFI and OFOD describe storing paths/valuations)

---

### D) Key Modeling Choices and Failure Modes (Explicit)

#### Unilateral vs Bilateral

- Bilateral value uses $f_{\text{nd}} - \text{CVA} + \text{DVA}$
- Some desks report "unilateral CVA" (ignore own default) for risk management or pricing policy. Choice is a desk/accounting convention

#### Independence vs Wrong-Way Risk (WWR)

- **Simplest assumption:** Exposure and default probability are independent
- **Wrong-way risk:** Default probability high when exposure high; right-way risk opposite. Defined explicitly
- **Failure mode:** Ignoring WWR can materially understate CVA for counterparties whose credit worsens exactly when exposures increase

#### Close-Out and Recovery Conventions

- The sources describe early termination settlement amount adjusted for replacement costs (half bid–ask)
- I'm not sure the provided sources fully specify "risk-free close-out" vs "replacement close-out" beyond this bid–ask replacement-cost adjustment; to be precise you need the ISDA/CSA close-out amount definition and legal interpretation for your jurisdiction and contract set

#### Collateral Modeling Details

- Cure period / MPOR is critical; collateral at default may reflect values 10–20 days earlier, not current MTM
- Threshold and minimum transfer amounts create residual uncollateralized exposure; independent amount/initial margin adds protection but has funding cost implications
- **Failure mode:** Exposure engine uses simplified CSA logic inconsistent with legal terms → systematically biased CVA

#### Funding Policy Assumptions (For FVA)

- FVA depends on whether you recognize funding benefit as well as cost (FBA vs FCA)
- **Failure mode:** Inconsistent funding curve/spreads across desks; double counting with discounting

#### Double-Counting Pitfalls

- If you discount using a "funding" curve rather than a risk-free/OIS proxy and then also add FVA, you may double count funding effects. The sources flag that economists dispute whether funding should alter valuation and treat FVA as weaker-theory
- **Practical control:** Document discounting + FVA policy together and run "policy-consistency" reconciliation tests

---

## Worked Examples (At Least 8 Numeric Examples)

*All examples use toy numbers for transparency. Currency units are USD. Time is in years. Probabilities are dimensionless. Hazards (where used) are per year.*

---

### Example A — CVA from EE and PD Increments (Discrete Sum)

**Given:**

- Intervals: $[0,1], [1,2], [2,3]$. Midpoints $t_1^* = 0.5$, $t_2^* = 1.5$, $t_3^* = 2.5$

- Expected positive exposure (average exposure) at midpoints:
  - $\text{EE}(0.5) = 10{,}000{,}000$
  - $\text{EE}(1.5) = 8{,}000{,}000$
  - $\text{EE}(2.5) = 6{,}000{,}000$

- Discount factors:
  - $D(0, 0.5) = 0.99$, $D(0, 1.5) = 0.97$, $D(0, 2.5) = 0.95$

- Recovery $R = 40\% \Rightarrow \text{LGD} = 1 - R = 60\% = 0.6$

- Risk-neutral default probability increments (by interval):
  - $\Delta\text{PD}_1 = 1.0\% = 0.010$
  - $\Delta\text{PD}_2 = 1.2\% = 0.012$
  - $\Delta\text{PD}_3 = 1.5\% = 0.015$

**Compute (discrete sum):**

$$\text{CVA} = \sum_{i=1}^{3} \text{LGD} \cdot D(0, t_i^*) \cdot \text{EE}(t_i^*) \cdot \Delta\text{PD}_i$$

**Interval 1:**
- $\text{EE} \cdot \Delta\text{PD} = 10{,}000{,}000 \times 0.010 = 100{,}000$
- Discount: $100{,}000 \times 0.99 = 99{,}000$
- LGD: $99{,}000 \times 0.6 = 59{,}400$

**Interval 2:**
- $8{,}000{,}000 \times 0.012 = 96{,}000$
- $96{,}000 \times 0.97 = 93{,}120$
- $93{,}120 \times 0.6 = 55{,}872$

**Interval 3:**
- $6{,}000{,}000 \times 0.015 = 90{,}000$
- $90{,}000 \times 0.95 = 85{,}500$
- $85{,}500 \times 0.6 = 51{,}300$

**Total:**

$$\boxed{\text{CVA} = 59{,}400 + 55{,}872 + 51{,}300 = 166{,}572}$$

**Unit check:** $D$ unitless, LGD unitless, $\Delta\text{PD}$ unitless, EE in USD → CVA in USD ✓

---

### Example B — From Hazard to PD Increments (Piecewise Constant $\lambda$) and Reuse Example A

**Given:**

- Same intervals and midpoints as Example A
- Piecewise-constant hazard (per year):
  - $\lambda_1 = 0.02$, $\lambda_2 = 0.03$, $\lambda_3 = 0.04$

- Survival recursion (constant hazard over interval):

$$Q(t_i) = Q(t_{i-1}) \, e^{-\lambda_i \Delta t}, \quad \Delta t = 1$$

(Exponential survival used in the source CDS exposition.)

**Step 1: Survival probabilities**

- $Q(0) = 1$
- $Q(1) = e^{-0.02} \approx 0.9802$
- $Q(2) = Q(1) e^{-0.03} \approx 0.9802 \times 0.9704 = 0.9512$
- $Q(3) = Q(2) e^{-0.04} \approx 0.9512 \times 0.9608 = 0.9139$

**Step 2: Interval default probabilities**

Using $q_i = Q(t_{i-1}) - Q(t_i)$ (difference of survivals; consistent with "default during year" logic):

- $\Delta\text{PD}_1 = 1 - 0.9802 = 0.0198$
- $\Delta\text{PD}_2 = 0.9802 - 0.9512 = 0.0290$
- $\Delta\text{PD}_3 = 0.9512 - 0.9139 = 0.0373$

**Check:** Cumulative default by 3y $= 1 - Q(3) \approx 0.0861$ (8.61%) ✓

**Step 3: Reuse Example A exposures/discounting/LGD**

$$\text{CVA} = \sum_{i=1}^{3} 0.6 \cdot D(0, t_i^*) \cdot \text{EE}(t_i^*) \cdot \Delta\text{PD}_i$$

**Interval 1:**
- $10{,}000{,}000 \times 0.0198 = 198{,}000$
- $\times 0.99 = 196{,}020$
- $\times 0.6 = 117{,}612$

**Interval 2:**
- $8{,}000{,}000 \times 0.0290 = 232{,}000$
- $\times 0.97 = 225{,}040$
- $\times 0.6 = 135{,}024$

**Interval 3:**
- $6{,}000{,}000 \times 0.0373 = 223{,}800$
- $\times 0.95 = 212{,}610$
- $\times 0.6 = 127{,}566$

**Total:**

$$\boxed{\text{CVA} = 117{,}612 + 135{,}024 + 127{,}566 = 380{,}202}$$

---

### Example C — Effect of Collateral / VM on EE and Hence CVA

We use the same default probabilities and discount factors as Example B. We compare:

**Uncollateralized expected exposure profile:**
$$\text{EE} = [10, 8, 6] \text{ million at } [0.5, 1.5, 2.5]$$

**Collateralized expected exposure profile (toy):**
$$\text{EE}_c = [2, 1.5, 1] \text{ million}$$

This reflects the source idea that collateral reduces exposure via $E = \max(V - C, 0)$ and cure-period effects can leave residual exposure.

**Compute CVA under collateralized profile:**

$$\text{CVA}_c = \sum_{i=1}^{3} 0.6 \cdot D(0, t_i^*) \cdot \text{EE}_c(t_i^*) \cdot \Delta\text{PD}_i$$

**Interval 1:**
- $2{,}000{,}000 \times 0.0198 = 39{,}600$
- $\times 0.99 = 39{,}204$
- $\times 0.6 = 23{,}522.4$

**Interval 2:**
- $1{,}500{,}000 \times 0.0290 = 43{,}500$
- $\times 0.97 = 42{,}195$
- $\times 0.6 = 25{,}317$

**Interval 3:**
- $1{,}000{,}000 \times 0.0373 = 37{,}300$
- $\times 0.95 = 35{,}435$
- $\times 0.6 = 21{,}261$

**Total:**

$$\boxed{\text{CVA}_c \approx 23{,}522.4 + 25{,}317 + 21{,}261 = 70{,}100.4}$$

**Compare:**
- Uncollateralized (Example B): $380{,}202$
- Collateralized: $\approx 70{,}100$

**Interpretation:** Collateral reduces EE dramatically → CVA falls ~81.6% in this toy setup.

**Sanity check:** Better collateralization → lower CVA ✓

---

### Example D — DVA from Negative Exposure (Own Default)

We compute a toy DVA using the same discrete-sum logic but with:

- Expected negative exposure $\text{ENE}(t) = \mathbb{E}[\max(-V(t), 0)]$
- Own-default probabilities $\Delta\text{PD}_i^{\text{own}}$
- LGD based on recovery to counterparty on the dealer

This matches the source statement that DVA can be calculated like CVA using equation (20.1) if $q_i$ and $v_i$ are based on dealer default and counterparty exposure to dealer.

**Given:**

- $\text{ENE} = [5, 7, 9]$ million at $[0.5, 1.5, 2.5]$
- Own-default increments: $\Delta\text{PD}^{\text{own}} = [0.012, 0.015, 0.018]$
- $R_{\text{own}} = 40\% \Rightarrow \text{LGD}_{\text{own}} = 0.6$
- Discount factors as before: $[0.99, 0.97, 0.95]$

**Compute:**

$$\text{DVA} = \sum_{i=1}^{3} 0.6 \cdot D(0, t_i^*) \cdot \text{ENE}(t_i^*) \cdot \Delta\text{PD}_i^{\text{own}}$$

**Interval 1:**
- $5{,}000{,}000 \times 0.012 = 60{,}000$
- $\times 0.99 = 59{,}400$
- $\times 0.6 = 35{,}640$

**Interval 2:**
- $7{,}000{,}000 \times 0.015 = 105{,}000$
- $\times 0.97 = 101{,}850$
- $\times 0.6 = 61{,}110$

**Interval 3:**
- $9{,}000{,}000 \times 0.018 = 162{,}000$
- $\times 0.95 = 153{,}900$
- $\times 0.6 = 92{,}340$

**Total:**

$$\boxed{\text{DVA} = 35{,}640 + 61{,}110 + 92{,}340 = 189{,}090}$$

**Sign explanation:** In the value formula $f_{\text{nd}} - \text{CVA} + \text{DVA}$, DVA increases the bank's valuation (benefit).

---

### Example E — Bilateral "CVA − DVA" Illustration (Two Naming Conventions)

**Using results:**
- CVA from Example B: $380{,}202$
- DVA from Example D: $189{,}090$
- Assume no-default value $f_{\text{nd}} = 1{,}000{,}000$

**Convention 1 (value form in source):**

$$f = f_{\text{nd}} - \text{CVA} + \text{DVA}$$

**Compute:**
- Net adjustment to value: $-380{,}202 + 189{,}090 = -191{,}112$
- Value with default:

$$\boxed{f = 1{,}000{,}000 - 380{,}202 + 189{,}090 = 808{,}888}$$

**Convention 2 (reporting "bilateral CVA" as a deduction):**

Some practitioners define a single reported number:

$$\text{BCVA} := \text{CVA} - \text{DVA} = 191{,}112$$

so that $f = f_{\text{nd}} - \text{BCVA}$.

**Be explicit in reporting:** Either is fine as long as you state which quantity you report.

---

### Example F — FVA from Expected Funding Requirement (Illustrative Mapping)

The sources define FCA/FBA and how FVA is formed conceptually, but they do not (in the excerpts we used) provide a single canonical discrete-sum formula in terms of a "funding requirement profile."

> **I'm not sure** what exact formula your desk should use without:
> - Your funding policy (what is "funding requirement"?)
> - Whether you recognize FBA
> - What discounting curve is used for FVA
> - And whether funding is computed at trade, netting-set, or bank level

That said, an overview-level mapping many teams use is:

$$\text{FCA} \approx \sum_{i=1}^{N} D(0, t_i^*) \cdot s_f \cdot F(t_i^*) \cdot \Delta t$$

where:
- $s_f$ is a funding spread over the discounting rate (units: 1/year)
- $F(t)$ is an expected funding requirement (USD)
- $\Delta t$ is year fraction

**Toy inputs:**
- $\Delta t = 1$ year each interval
- Funding requirement $F = [10, 8, 6]$ million (toy: equal to uncollateralized EE)
- Funding spread $s_f = 1\% = 0.01$ per year
- Discount factors $D = [0.99, 0.97, 0.95]$

**Compute:**
- Interval 1: $0.99 \times 0.01 \times 10{,}000{,}000 \times 1 = 99{,}000$
- Interval 2: $0.97 \times 0.01 \times 8{,}000{,}000 \times 1 = 77{,}600$
- Interval 3: $0.95 \times 0.01 \times 6{,}000{,}000 \times 1 = 57{,}000$

$$\boxed{\text{FCA} \approx 99{,}000 + 77{,}600 + 57{,}000 = 233{,}600}$$

**Interpretation:** If you recognize only a funding cost on positive funding requirement, this is an FCA-style number. If you also recognize funding benefit (FBA), you would compute an analogous term for negative funding requirement and net them as $\text{FVA} = \text{FCA} - \text{FBA}$ (consistent with the source's definition).

---

### Example G — Sensitivity "CVA01" to a Small Spread/Hazard Bump (Toy)

We compute a simple bump impact:

- Base CVA from Example B: $\text{CVA}_0 = 380{,}202$

- Use the spread–hazard approximation $\lambda \approx s/(1-R)$

- Assume we bump the counterparty credit spread by +1 bp $= 0.0001$ (in decimal per year). With $R = 0.40$, the implied hazard bump is:

$$\Delta\lambda = \frac{0.0001}{1-R} = \frac{0.0001}{0.6} = 0.0001667 \text{ per year}$$

**Toy recomputation approach:** Bump each yearly hazard:

$$\lambda_1' = 0.0201667, \quad \lambda_2' = 0.0301667, \quad \lambda_3' = 0.0401667$$

Using the same exponential survival recursion as Example B yields slightly larger $\Delta\text{PD}_i$, and recomputing CVA with the same EE and discounting gives (toy arithmetic):

$$\text{CVA}_1 \approx 382{,}225$$

Therefore:

$$\boxed{\Delta\text{CVA} = \text{CVA}_1 - \text{CVA}_0 \approx 2{,}023}$$

**Report:**

CVA01 $\approx \$2{,}023$ per 1 bp spread increase (in this toy setup).

**Units:** "USD per bp" is a sensitivity unit; it translates spread moves into PV moves.

---

### Example H — Wrong-Way Risk Toy: Exposure High When PD Is High (Conceptual Illustration)

The sources define wrong-way risk qualitatively (PD positively correlated with exposure). They do not fully formalize a two-state numerical model here, so treat this as a conceptual illustration.

**Setup (single interval, no discounting for simplicity):**

- LGD $= 0.6$
- Two states (each probability 50%):
  - **State A ("bad"):** exposure $E_A = 20$ million and default probability in interval $p_A = 5\%$
  - **State B ("good"):** exposure $E_B = 2$ million and default probability $p_B = 1\%$

**Independence-style approximation (using averages):**

- Average exposure: $\bar{E} = 0.5 \times 20 + 0.5 \times 2 = 11$ million
- Average PD: $\bar{p} = 0.5 \times 0.05 + 0.5 \times 0.01 = 0.03$
- CVA approximation:

$$\text{CVA}_{\text{ind}} \approx 0.6 \times 11{,}000{,}000 \times 0.03 = 198{,}000$$

**Wrong-way risk (expected product):**

Expected exposure–PD product:

$$\mathbb{E}[E \cdot p] = 0.5(20{,}000{,}000 \cdot 0.05) + 0.5(2{,}000{,}000 \cdot 0.01)$$
$$= 0.5(1{,}000{,}000) + 0.5(20{,}000) = 510{,}000$$

CVA:

$$\text{CVA}_{\text{WWR}} = 0.6 \times 510{,}000 = 306{,}000$$

**Increase due to WWR:** $306{,}000 - 198{,}000 = 108{,}000$ (+54.5%)

**Takeaway:** Correlation can materially increase CVA; this matches the qualitative warning in the sources.

---

## Practical Notes

### 5.1 Data Inputs Checklist

#### A) Legal / Portfolio Structure

- Counterparty and netting sets; portfolio membership; trade population and lifecycle events
- ISDA master agreement: netting scope, early termination triggers ("event of default"), settlement approach

#### B) CSA / Collateral Terms

- Collateral currencies; eligible collateral types; haircuts; rehypothecation
- Variation margin rules: frequency, threshold, minimum transfer amount, independent amount / initial margin. (CSA specifies thresholds, independent amounts, MTAs, haircuts.)
- Cure period / MPOR (often 10–20 days)

#### C) Market Data and Models (Exposure Engine)

- Curves for discounting and projection (single-curve vs multi-curve approach belongs elsewhere; here just ensure consistency)
- For risk-free proxy: OIS curve guidance; Fed funds proxy and collateral remuneration notes
- Vol surfaces / calibration inputs if simulation used

#### D) Credit Data

- Counterparty credit spreads/curves (CDS or bond-implied)
- Recovery assumptions $R$ (and consistency with credit curve construction)
- Own credit curve for DVA

#### E) Funding Inputs (For FVA)

- Funding spread(s) used for FCA and (if applicable) investing/excess-cash spreads for FBA
- Funding policy boundary: netting set vs desk vs bank-level aggregation

---

### 5.2 Implementation Pitfalls (Common Ways XVA Breaks)

#### Inconsistent Netting/Collateral Modeling

- Exposure must be computed on the net portfolio under enforceable netting; otherwise you overstate CVA
- CSA cure period requires valuing at $t_i^* - c$ as well as $t_i^*$; missing this can understate exposure spikes during MPOR

#### Wrong Sign Conventions

- Remember value is $f_{\text{nd}} - \text{CVA} + \text{DVA}$
- Mixing "reported BCVA" vs "value adjustment" can cause confusion in P&L explain

#### Time Discretization Error

- Coarse grids can miss exposure peaks (especially around optionality/exercise/payment dates), biasing EE/PFE and thus CVA
- **Practical fix:** Align grid with cashflow dates, resets, Bermudan exercise dates, etc.

#### Mixing Discounting Frameworks Without a Policy

- The sources discuss risk-free discounting (OIS proxy) and collateral remuneration. Combining inconsistent curves with FVA can double count funding

#### WWR Ignored When It Matters

- Wrong-way risk is explicitly defined; ignoring it can understate CVA when exposures rise in stress that also increases default probabilities

#### Incremental XVA Computed Naively

- Full recomputation is often infeasible; the source describes storing simulation paths/portfolio values and revaluing only the new trade on stored paths to estimate incremental CVA using equation (20.1)

---

### 5.3 Validation and Sanity Checks

#### Core Monotonicity Checks

CVA should increase when:
- EE increases
- Default probabilities/hazards increase
- Recovery decreases (LGD increases)

(All follow directly from the discrete-sum structure.)

DVA should increase when ENE increases and own hazard increases; and DVA can rise as dealer spreads widen (controversial but source-backed).

#### Collateral Sanity Check

Under idealized daily two-way VM with zero threshold and no cure-period lag, exposure should be near zero → CVA near zero (subject to modeling simplifications). The cure-period example shows residual exposure arises precisely from lag and collateral mismatch.

#### Peak Exposure (PFE) Sanity

Peak exposure is a high percentile of trial exposures; should exceed average exposure at each time; maximum peak exposure should occur near periods of high optionality/volatility.

#### P&L Explain (Daily XVA)

CVA P&L decomposes into:
- Credit-spread moves ($q_i$ moves) and
- Exposure/market-factor moves ($v_i$ moves)

DVA P&L similarly depends on own spread and ENE.

---

## Summary & Recall

### 6.1 Ten-Bullet Executive Summary

1. **CVA** is the PV of expected loss from counterparty default on positive exposure; computed over time intervals with default probabilities and expected exposure/loss at midpoints.

2. **DVA** is the mirror image: PV of expected loss to counterparty from own default; a benefit to the dealer and enters as $f_{\text{nd}} - \text{CVA} + \text{DVA}$.

3. **Netting** means CVA/DVA are portfolio (netting-set) quantities, not trade-by-trade, because exposure is $\max(\sum V_i, 0)$.

4. **Collateral** reduces exposure via $E = \max(V - C, 0)$; cure period/MPOR means collateral at default reflects earlier values, leaving residual risk.

5. **Default probabilities** for valuation should be risk-neutral, not real-world; peak exposure scenario analysis conceptually uses real-world scenarios.

6. **Hazard/intensity** $\lambda_t$ is defined by short-horizon conditional default probability; units are 1/year.

7. A practical approximation links hazard to credit spread: $\lambda \approx s/(1-R)$ (assumptions matter).

8. **Wrong-way risk** is positive dependence between exposure and default probability; ignoring it can understate CVA.

9. **FVA** reflects funding costs/benefits; sources define FCA/FBA and emphasize weaker theoretical basis and policy dependence.

10. **Practical implementation** relies on Monte Carlo exposure engines, stored scenario paths, and efficient incremental XVA computations.

---

### 6.2 Cheat Sheet (Key Definitions + Discrete-Sum Formulas with Units)

#### Netting Exposure Today

| Without Netting | With Netting |
|-----------------|--------------|
| $\sum_{i=1}^{N} \max(V_i, 0)$ (USD) | $\max\left(\sum_{i=1}^{N} V_i, 0\right)$ (USD) |

#### Collateralized Exposure at Default

$$\boxed{E = \max(V - C, 0) \quad [\text{USD}]}$$

#### Peak Exposure (PFE)

"Peak exposure at midpoint" = high percentile of simulated exposures (e.g., 97.5%)

#### CVA (LGD-Separated Discrete Sum; Midpoint Exposure Form)

$$\boxed{\text{CVA} \approx \sum_i (1-R_{\text{cp}}) \, D(0, t_i^*) \, \text{EE}(t_i^*) \, \Delta\text{PD}_i^{\text{cp}}}$$

**Units:** $(1-R)$ unitless, $D$ unitless, EE USD, $\Delta\text{PD}$ unitless → USD

#### DVA (Analog)

$$\boxed{\text{DVA} \approx \sum_i (1-R_{\text{own}}) \, D(0, t_i^*) \, \text{ENE}(t_i^*) \, \Delta\text{PD}_i^{\text{own}}}$$

#### Value with Default Risk

$$\boxed{f = f_{\text{nd}} - \text{CVA} + \text{DVA}}$$

#### Hazard Definition

$$\boxed{P(\tau \in (t, t+h] \mid \tau > t, \mathcal{F}_t) = \lambda_t h + o(h), \quad \lambda_t \text{ [1/year]}}$$

---

### 6.3 Flashcards (35)

| # | Question | Answer |
|---|----------|--------|
| 1 | What is $f_{\text{nd}}$? | No-default value of the derivatives portfolio, assuming neither side defaults |
| 2 | In the source framework, how does default risk change portfolio value? | $f_{\text{nd}} - \text{CVA} + \text{DVA}$ |
| 3 | When does the bank incur loss from counterparty default (uncollateralized)? | When the portfolio value is positive to the bank at early termination; then it is an unsecured creditor and may not be paid in full |
| 4 | What is netting, conceptually? | All trades between two parties are treated as a single trade at default/for collateral, preventing cherry-picking |
| 5 | Exposure with netting today equals what? | $\max(\sum_i V_i, 0)$ |
| 6 | Exposure without netting today equals what? | $\sum_i \max(V_i, 0)$ |
| 7 | What is the exposure formula with collateral $C$? | $E = \max(V - C, 0)$ |
| 8 | What is the cure period (MPOR) in CVA modeling? | Assumed time between collateral failure/event of default and closeout; collateral reflects earlier portfolio value (often 10–20 days) |
| 9 | How is peak exposure defined in the sources? | A high percentile (e.g., 97.5%) of simulated exposures at each midpoint |
| 10 | What is wrong-way risk? | Positive dependence between default probability and exposure (PD high when exposure high) |
| 11 | What is right-way risk? | Negative dependence between default probability and exposure |
| 12 | Why are risk-neutral default probabilities used for CVA valuation? | Because the calculation is valuing future cash flows using risk-neutral valuation |
| 13 | Why might peak exposure ideally use real-world simulation? | It is a scenario/worst-case question; theory suggests real-world dynamics for scenario generation |
| 14 | What does $q_i$ represent in the CVA sum? | Probability of default in interval $i$ |
| 15 | What does $v_i$ represent (one source's definition)? | PV of expected loss if default occurs at midpoint of interval $i$ |
| 16 | What does $v_i$ represent (alternative convention)? | PV of expected exposure at midpoint, with LGD applied separately |
| 17 | State the hazard/intensity definition | $P(\tau \in (t, t+h] \mid \tau > t, \mathcal{F}_t) = \lambda_t h + o(h)$ |
| 18 | What are the units of hazard $\lambda$? | 1/time (e.g., per year) |
| 19 | Under constant hazard $\lambda$, what is survival $Q(t)$ in the CDS exposition? | $Q(t) = e^{-\lambda t}$ |
| 20 | How do you compute interval default probability from survival? | $q_i = Q(t_{i-1}) - Q(t_i)$ |
| 21 | What is the spread–hazard approximation given? | $\lambda \approx s/(1-R)$ |
| 22 | Why is DVA controversial economically? | Dealer spread widening increases DVA and can increase reported profits, which is counterintuitive; regulators have been uncomfortable |
| 23 | What does FVA attempt to capture? | Funding costs/benefits created by derivatives (e.g., uncollateralized positive MTM or margin funding) |
| 24 | Define FCA and FBA | FCA = PV of expected funding cost; FBA = PV of expected funding benefit |
| 25 | How is FVA related to FCA and FBA in the source? | FVA reduces value by the excess of FCA over FBA (i.e., FCA − FBA) |
| 26 | What curve is suggested for determining risk-free rates? | OIS curve |
| 27 | What rate is cited as a proxy risk-free in USD context? | Overnight federal funds rate (Fed funds) |
| 28 | Under full collateralization, what should discounting correspond to? | Collateral remuneration rate (per the interest-rate modeling note) |
| 29 | Why is CVA computationally intensive? | Requires simulating market variables and valuing large portfolios across many scenarios/timepoints |
| 30 | How can incremental CVA be computed efficiently? | Store scenario paths and portfolio values; revalue only the new trade on stored paths to estimate incremental exposure changes and CVA change |
| 31 | What are the two major risks in CVA per the source? | Counterparty credit spread risk and underlying market-variable risk affecting exposures |
| 32 | What does "event of default" trigger in ISDA? | The non-defaulting party has right to terminate all transactions after a short period. (Events include bankruptcy, failure to pay, failure to post collateral.) |
| 33 | Why does netting affect collateral requirements? | Collateral is computed on net portfolio value, not individual trades, reducing posted amounts |
| 34 | What is "peak exposure" used for vs CVA? | Peak exposure is a scenario/worst-case percentile measure; CVA is a pricing/valuation expected loss measure |
| 35 | What is the minimum you must specify to compute CVA? | Exposure profile (from simulation + collateral/netting), default probabilities (risk-neutral), recovery/LGD, and discounting curve. (All are implicit in the CVA setup described.) |

---

## Mini Problem Set (18 Questions)

*Provide brief solution sketches for questions 1–9 only*

---

### Basic Questions

**1.** A netting set has two trades with values $+12$ and $-9$ million. Compute exposure today with and without netting.

> **Sketch:** Without netting: $12 + 0 = 12$. With netting: $\max(12 - 9, 0) = 3$. (Matches netting formulas.)

**2.** With collateral $C = 8$ million held and portfolio value $V = 10$ million at default, compute exposure $E$.

> **Sketch:** $E = \max(10 - 8, 0) = 2$ million using $E = \max(V - C, 0)$.

**3.** Using Example A numbers but with recovery $R = 60\%$, recompute CVA.

> **Sketch:** Replace LGD 0.6 with 0.4; CVA scales linearly by $0.4/0.6$. New CVA $= 166{,}572 \times (2/3) = 111{,}048$.

**4.** Given hazard $\lambda = 0.05$ constant and $Q(0) = 1$, compute $Q(2)$ under exponential survival.

> **Sketch:** $Q(2) = e^{-0.05 \times 2} = e^{-0.10} \approx 0.9048$. (Survival form appears in CDS exposition.)

**5.** If $Q(1) = 0.97$ and $Q(2) = 0.94$, compute default probability in year 2.

> **Sketch:** $q_2 = Q(1) - Q(2) = 0.03$. (Difference of survivals.)

---

### Intermediate Questions

**6.** In a 3-interval CVA sum, exposures are $[2, 1, 0.5]$ million, discount factors $[0.99, 0.97, 0.95]$, PD increments $[0.02, 0.03, 0.04]$, recovery $R = 0.4$. Compute CVA.

> **Sketch:** Compute each term $0.6 \times D \times EE \times PD$ and add:
> - 1: $0.6 \cdot 0.99 \cdot 2{,}000{,}000 \cdot 0.02 = 23{,}760$
> - 2: $0.6 \cdot 0.97 \cdot 1{,}000{,}000 \cdot 0.03 = 17{,}460$
> - 3: $0.6 \cdot 0.95 \cdot 500{,}000 \cdot 0.04 = 11{,}400$
>
> Total $= 52{,}620$.

**7.** Explain why DVA is a benefit to the dealer in the bilateral value formula.

> **Sketch:** DVA is "counterparty's CVA" and derivatives are zero-sum; dealer gains from possibility it avoids payments when it defaults. Book value adds DVA: $f_{\text{nd}} - \text{CVA} + \text{DVA}$.

**8.** What modeling inputs are needed to incorporate cure period into exposure simulation?

> **Sketch:** Need values $V(t_i^*)$ and $V(t_i^* - c)$ to compute collateral at $t_i^*$ per CSA; then compute $E = \max(V - C, 0)$.

**9.** Why should default probabilities be risk-neutral in CVA valuation?

> **Sketch:** Because CVA values future cash flows using risk-neutral valuation; the text explicitly states $q_i$ should be risk-neutral default probabilities in this context.

---

### Additional Questions (No Solutions Provided)

**10.** Construct a toy example showing that netting can reduce CVA even if a new trade has positive standalone EE.

**11.** Given a term structure of credit spreads $s_i$ and recovery $R$, compute $q_i$ using $q_i = \exp(-s_{i-1} t_{i-1}/(1-R)) - \exp(-s_i t_i/(1-R))$.

**12.** Explain how wrong-way risk changes the expected-loss calculation conceptually.

---

### Advanced Questions

**13.** Describe how you would compute "exposure Greeks" (sensitivity of EE to market factors) and why it is computationally hard.

**14.** Provide a policy-consistency argument for why discounting curve choice and FVA must be decided together.

**15.** Discuss how initial margin requirements (IM) could reduce CVA but increase funding costs, and how that might appear in FVA/MVA.

**16.** Design a validation test suite for an XVA engine (list at least 8 tests).

**17.** Explain, using the bilateral value equation, why a bank's reported derivative value can rise when its own credit worsens, and why this is controversial.

**18.** Sketch an approach to incorporate wrong-way risk via a hazard rate that depends on market variables (conceptually), and list what data you'd need.

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

- CVA/DVA discrete-sum framework: Hull Ch 9, Ch 24
- Exposure definitions and netting: Hull Ch 9
- Hazard/survival/intensity definitions: Hull Ch 24 (CDS exposition)
- Spread-to-hazard approximation: Hull Ch 24
- FVA/FCA/FBA definitions: Hull Ch 9
- OIS as risk-free proxy: Andersen & Piterbarg Vol 1, Hull Ch 4
- Collateral remuneration and discounting: Andersen & Piterbarg Vol 1 Ch 6
- Wrong-way risk definition: Hull Ch 9

### (B) Reasoned Inference — Note Derivation Logic

- Monotonicity of CVA with respect to EE, PD, and LGD follows directly from the discrete-sum structure
- The implementation map synthesizes guidance across sources into a workflow
- Unit checks follow from dimensional analysis of the formulas

### (C) Speculation — Flag Uncertainties

- I'm not sure whether the provided sources use the Basel-style terms "EPE/ENE" explicitly
- I'm not sure the provided sources fully specify "risk-free close-out" vs "replacement close-out" beyond bid–ask replacement-cost adjustment
- I'm not sure what exact FVA formula your desk should use without knowing your funding policy
