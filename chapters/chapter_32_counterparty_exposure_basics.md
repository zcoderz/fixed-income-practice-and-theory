# Chapter 32: Counterparty Exposure Basics — Netting, Collateral, and Margin Timing

---

## Conventions & Notation (used in this chapter only)

| Convention | Description |
|------------|-------------|
| **Perspective** | "Bank/dealer" perspective unless stated otherwise. Positive values are assets to the bank (counterparty owes the bank). |
| **MTM sign convention** | $V(t)$ = net mark-to-market value of the whole netting set at time $t$, measured in currency units (e.g., USD). $V(t) > 0$ means the bank is in-the-money. |
| **Collateral sign convention (Hull-style)** | $C$ = collateral posted by the counterparty to the bank at the time of default; if $C < 0$, then $-C$ is collateral posted by the bank to the counterparty at default. |
| **Exposure (at default time)** | $E = \max(V - C, 0)$ in all situations (bank's exposure to counterparty). |
| **Cure period / MPOR** | Margin period of risk: A delay (typically 10 or 20 days) between cessation of collateral posting and close-out; collateral at default reflects portfolio value $c$ days earlier. |
| **Threshold $H$** | Variation margin (VM) is required only above a threshold; example logic: if value is 9 and threshold is 10, no collateral; if value is 11, collateral of 1. |
| **Minimum transfer amount (MTA) $m$** | Small collateral movements can be ignored to avoid operational "nuisance" transfers. |
| **Probability/expectation** | We view future portfolio values as random variables on a probability space; portfolio value at time $s$ is $V(s)$. |

---

## 0. Setup

### Conventions used in this chapter

- Bank/dealer perspective; $V(t)$ is net MTM of the netting set.
- Collateral sign convention follows Hull (see above).
- When I introduce desk-style terms (EE, PFE, EPE), I will flag where the books are explicit vs where I'm building a consistent "plumbing" layer.

### Notation glossary (symbols + definitions)

| Symbol | Definition |
|--------|------------|
| $t$ | time (years unless explicitly "days") |
| $V(t)$ | net MTM of all trades in a netting set at time $t$ (currency) |
| $C(t)$ | collateral held by the bank at time $t$ (currency; negative if bank posted) |
| $E(t)$ | bank's exposure to the counterparty at time $t$ (currency) |
| $H$ | threshold for VM (currency) |
| $m$ | minimum transfer amount (currency) |
| $c$ | cure period / MPOR length (days) |
| $q$ | quantile level (e.g., $q = 95\%$) |
| $D(0,t)$ | discount factor from 0 to $t$ (dimensionless) |
| $R$ | recovery rate (dimensionless, in $[0,1]$) |

---

## Fact Classification

### (A) Verified facts (directly supported by sources)

- Netting in ISDA master agreements means that upon default, all covered transactions are treated as a single transaction; this can materially reduce credit risk.
- Without netting, exposure at default today for $N$ derivatives with values $V_i$ is $\sum_{i=1}^{N} \max(V_i, 0)$.
- With netting (and no collateral), dealer exposure is $\max(V, 0)$ where $V$ is the net MTM of all outstanding transactions.
- With collateral, exposure at default is $E = \max(V - C, 0)$ using the collateral sign convention above.
- Cure period / MPOR is typically 10 or 20 days; collateral at default reflects portfolio value $c$ days earlier, and exposure simulations must therefore evaluate the portfolio at both $t_i^*$ and $t_i^* - c$.
- "Peak exposure" is defined as a high percentile of simulated exposures at each future time (e.g., 97.5%); the book flags that peak exposure is conceptually a real-world "how bad can it get?" scenario analysis, even though risk-neutral simulation is often used in practice.
- Bilateral collateral arrangements: VM can be required (posting collateral equal to positive MTM to the other side); IM may also be required and is often called an "independent amount" in bilateral OTC. Thresholds and MTAs are common terms.
- Post-crisis rules (as described in the text) require IM+VM for many bilaterally cleared transactions between financial institutions; IM is described as covering a 10-day stressed period at 99% confidence and is typically segregated/held by a custodian.
- Credit exposures should be assessed using current replacement value and potential future replacement costs, and aggregated to reflect enforceable netting agreements.

### (B) Reasoned inference (derived from (A), consistent plumbing)

- If netting makes covered trades a single transaction, then loss at default is based on the net portfolio value: $E_{\text{net}}(t) = \max(V_{\text{net}}(t), 0)$. This is the natural extension of $\max(V, 0)$ when $V$ is net MTM.
- Desk metrics like EE and PFE are best viewed as expectation/quantiles of the exposure distribution, in the same "risk measure" spirit as VaR/quantiles used elsewhere in risk management.

### (C) Speculation (clearly labeled; minimal)

- I'm not sure which precise operational timing conventions (call cutoffs, settlement cycles, dispute resolution mechanics) your desk uses; the books emphasize a cure period/MPOR but do not fully standardize the intraday/settlement-lag microstructure. To be certain, we'd need your CSA/operating model (call time, settlement lag, dispute process, and close-out timing).

---

## 1. Core Concepts (definitions first)

### 1.1 Mark-to-market (MTM) and portfolio value

**Formal definition:** $V(t)$ is the (net) market value of outstanding transactions at time $t$; positive means the bank is owed value. In counterparty exposure discussions, dealers focus on $\max(V, 0)$ when no collateral is posted and $V$ is net MTM.

**Intuition:** MTM is "what it would cost to replace the portfolio today at mid" (ignoring close-out adjustments).

**In practice:** MTM is the input to exposure, margin calls (VM), and "credit limits."

### 1.2 Exposure (current replacement cost)

**Formal definition:** With no collateral, dealer exposure at time $t$ is $\max(V(t), 0)$ where $V(t)$ is net MTM.

With collateral, exposure at default time is:

$$E(t) = \max(V(t) - C(t), 0)$$

where $C(t)$ is collateral held by the bank at default (negative if bank posted).

**Intuition:** Exposure is "how much I lose if you default right now," after using collateral I already hold.

**In practice:** Risk policies stress measuring exposure based on current replacement value and potential future replacement costs.

### 1.3 Potential future exposure (PFE) / "peak exposure"

**Formal definition (source-backed concept):** "Peak exposure" is a high percentile of simulated exposures at each future time (e.g., 97.5%); the maximum peak exposure is the maximum across future times.

**Intuition:** PFE answers "how bad can exposure get?" at a chosen confidence level.

**In practice:** The text flags an important conceptual distinction: CVA is a valuation exercise (risk-neutral simulation and discounting), whereas peak exposure is a scenario-analysis question that, in principle, calls for real-world dynamics.

### 1.4 Netting and netting sets (contractual aggregation)

**Formal definition:** Netting is a clause (typically in an ISDA master agreement) that treats all covered transactions as a single transaction upon default.

**Intuition:** Gains on some trades can offset losses on others when calculating the close-out amount.

**In practice:** Exposure aggregation should reflect enforceable netting agreements. This motivates the operational "netting set": the set of trades whose close-out is computed on a net basis under a master agreement / CSA.

### 1.5 Collateral: VM, IM (independent amount), threshold, MTA

#### Variation margin (VM)

**Formal definition:** VM is collateral that tracks the current positive value of transactions to the other side (bank posts when it is out-of-the-money; counterparty posts when it is in-the-money).

**Intuition:** VM reduces current exposure by covering present MTM.

#### Initial margin (IM) / independent amount

**Formal definition (as described):** For bilaterally cleared transactions subject to the post-crisis rules described, IM must cover market moves over a 10-day stressed period with 99% confidence, and is typically segregated/held by a custodian. In bilateral OTC, the text notes IM is often called an independent amount.

**Intuition:** IM is a buffer for future exposure during MPOR (the gap between last effective margin and close-out).

#### Threshold and MTA

**Formal definition:** A threshold means VM is required only above a threshold amount; MTA avoids small collateral moves.

**Intuition:** Threshold/MTA increase residual exposure but reduce operational friction.

### 1.6 Cure period / margin period of risk (MPOR)

**Formal definition:** Cure period (aka margin period of risk) is the time between the counterparty stopping collateral posting and close-out; typically 10 or 20 days; collateral at default reflects the portfolio value $c$ days earlier.

**Intuition:** Even with "daily VM," you can still lose money if MTM moves during the MPOR.

**In practice:** Simulation must value the portfolio at both $t_i^*$ and $t_i^* - c$ to compute collateral available at $t_i^*$.

### 1.7 Wrong-way risk (preview only)

**Formal definition (conceptual):** Wrong-way risk is when exposure and default probability are positively related; e.g., speculative positions can worsen the counterparty's financial position when they move against it.

**Intuition:** "The counterparty is weakest exactly when I'm most exposed."

**In practice:** We only preview it here; modeling dependence is a separate (later) chapter.

---

## 2. Math and Derivations (step-by-step)

### 2.1 Exposure as a function of MTM (no collateral)

**Assumption:** close-out exposure is computed at mid MTM (ignoring bid-offer close-out adjustments; the text notes exposures are typically computed without those adjustments).

If a single netting set has net MTM $V(t)$, then the bank loses money on counterparty default only when $V(t) > 0$.

Therefore, exposure (replacement cost) is:

$$\boxed{E(t) = \max(V(t), 0)}$$

This is explicitly how dealer net exposure is stated in the text when no collateral is posted and $V$ is the net MTM.

**Unit check:** $V$ is currency $\Rightarrow E$ is currency.

**Sanity checks:**
- If $V(t) \leq 0$, bank owes (or is flat), so exposure is $0$.
- If $V(t) = 0$, exposure is $0$.

### 2.2 Netting: from trade-level exposure to portfolio exposure

The master agreement netting clause treats all covered trades as one transaction in default.

Let trades have individual values $V_1, \ldots, V_N$ at default time.

**Without netting (trade-by-trade pick-and-choose):**

$$E_{\text{no net}} = \sum_{i=1}^{N} \max(V_i, 0)$$

as stated.

**With netting (single close-out amount):**

The "single transaction" principle implies the close-out is computed on net value:

$$V_{\text{net}} = \sum_{i=1}^{N} V_i$$

so the exposure becomes:

$$\boxed{E_{\text{net}} = \max\left(\sum_{i=1}^{N} V_i, 0\right)}$$

This is the natural extension of the text's statement that exposure is $\max(V, 0)$ with $V$ equal to the net MTM of all transactions.

**Key inequality (netting benefit):**

$$\boxed{\max\left(\sum_{i=1}^{N} V_i, 0\right) \leq \sum_{i=1}^{N} \max(V_i, 0)}$$

because $\max(\cdot, 0)$ is subadditive for two terms and extends by induction.

**Unit check:** both sides are currency.

**Sanity check (perfect offset):** if $V_2 = -V_1$ always, net exposure is $0$ while trade-by-trade exposure is $|V_1|$.

### 2.3 Collateralized exposure: $E = \max(V - C, 0)$

The text defines collateral $C$ at default and gives the universal exposure identity:

$$\boxed{E = \max(V - C, 0)} \tag{2.1}$$

**Interpretation of signs (important):**
- If $V > 0$ and counterparty posted collateral $C > 0$, then exposure shrinks to $\max(V - C, 0)$.
- If bank posted collateral ($C < 0$), the posted amount may be lost on counterparty default; if it exceeds what the bank owed ($-V$), that "excess posted" is an exposure.

**Unit check:** $V$, $C$, $E$ all currency.

### 2.4 Cure period / MPOR: collateral is stale

The cure period $c$ is typically 10 or 20 days; collateral at default reflects portfolio value $c$ days earlier.

Let default occur around time $t$. A simplified representation of the book's modeling idea is:
- CSA determines collateral from the portfolio value at $t - c$.
- Exposure at $t$ uses $V(t)$ and the stale collateral $C(t)$ via (2.1).

The example in the text illustrates precisely how a zero-threshold two-way CSA plus a 20-day cure period can leave residual exposure when $V(t) \neq V(t - 20d)$.

### 2.5 EE, PFE, and time aggregation (desk metrics)

The books explicitly discuss:
- exposure as $\max(V, 0)$ or $\max(V - C, 0)$,
- "peak exposure" as a high percentile of simulated exposures.

**Reasoned inference (standardized notation for this chapter):**

**Expected exposure (EE):**

$$\boxed{EE(t) := \mathbb{E}[E(t)]}$$

**Potential future exposure (PFE):**

$$\boxed{PFE_q(t) := \inf\{x : P(E(t) \leq x) \geq q\}}$$

i.e., the $q$-quantile of $E(t)$. This is the same logic as "peak exposure" (a high percentile).

**EPE / ENE (time-aggregated exposure):**

I'm not sure the books give a single universal definition of EPE/ENE in this excerpted material. The consistent "plumbing" convention (commonly used on desks) is:

$$EPE := \frac{1}{T} \int_0^T EE(t) \, dt \quad \text{or} \quad EPE \approx \sum_i w_i \, EE(t_i)$$

and similarly:

$$ENE := \frac{1}{T} \int_0^T \mathbb{E}[\max(-V(t), 0)] \, dt$$

or with collateral using $\max(C(t) - V(t), 0)$ depending on sign conventions.

To be certain, we'd need your desk's definition (time-weighting, whether collateral is included, and whether "effective maturity"/regulatory style averaging is used).

---

## 3. Measurement & Risk (only what belongs in Chapter 32)

### A) Exposure primitives

#### A.1 Mark-to-market (MTM) / portfolio value $V(t)$

$V(t)$ is net MTM of all transactions in the netting set. When there are many transactions, the bilateral agreement means they are netted and exposure is based on the net value.

#### A.2 Exposure $E(t) = \max(V(t), 0)$ and collateralized exposure

- **No collateral:** $E(t) = \max(V(t), 0)$
- **With collateral:** $E(t) = \max(V(t) - C(t), 0)$
- **Negative exposure / ENE idea:** if $V(t) < 0$, then the counterparty has exposure to the bank; this is central for DVA (mirror of CVA).

#### A.3 "Current exposure" vs "potential future exposure"

- **Current exposure:** exposure computed from today's MTM and today's collateral (replacement cost "right now").
- **Potential future exposure:** the distribution of exposure at future times; risk policy language emphasizes both current replacement value and potential future replacement costs.

### B) Summary exposure metrics (definitions + interpretation)

#### B.1 Expected exposure $EE(t)$

**Definition (chapter convention):** $EE(t) = \mathbb{E}[E(t)]$

**Interpretation:** "average exposure" at time $t$. The books describe Monte Carlo systems that estimate expected net exposure at future times conditional on default for CVA work.

#### B.2 Potential future exposure $PFE_q(t)$

**Definition (chapter convention):** $PFE_q(t)$ is the $q$-quantile of $E(t)$.

**Interpretation:** aligns with "peak exposure," defined as a high percentile of simulated exposures.

#### B.3 EPE / ENE

I'm not sure the books fix a single EPE/ENE definition here.

**Practical convention used in this chapter:** time-average of $EE(t)$ (for EPE) or time-average of expected negative exposure (for ENE), with explicit time-grid weights when discretized.

#### B.4 Margin period of risk (MPOR) and why it matters

- MPOR (cure period) is typically 10 or 20 days; during this period collateral is stale and exposure can emerge even under daily margining.
- The exposure simulation must compute values at $t_i^* - c$ to determine collateral at $t_i^*$.

### C) Netting sets and close-out

#### C.1 What "netting set" means (conceptually)

- **Contractually:** netting arises from a master agreement clause: upon default, all covered transactions are treated as a single transaction.
- **Practically:** a "netting set" is the collection of trades whose close-out amount is computed on a net basis and whose collateral is managed together (CSA-aligned).

#### C.2 How close-out netting changes exposure

- **Without netting:** add up positive parts of individual trades.
- **With netting:** compute exposure on net MTM $V$ via $\max(V, 0)$.

#### C.3 Wrong-way risk preview (concept only)

If the counterparty's financial condition worsens when trades move against it, exposure and default probability can be positively related (wrong-way risk).

### D) Collateral mechanics

#### D.1 VM: how it tracks MTM and reduces current exposure

VM can be set so that when transactions are positive to one side, the other posts collateral equal to that positive value.

Under a perfect, instantaneous zero-threshold VM world, current exposure can be driven close to zero; residual risk comes from MPOR/timing and contractual frictions.

#### D.2 IM: why it exists (future exposure during MPOR)

For bilaterally cleared transactions subject to the rules described, IM is designed to protect over a 10-day stressed period with 99% confidence and is typically segregated.

In bilateral OTC, IM is often called an "independent amount."

#### D.3 Threshold, MTA, independent amount, haircuts

- Threshold and MTA are explicitly discussed as common CSA features.
- Independent amount terminology is explicitly mentioned.
- **Haircuts:** I'm not sure the provided chapters define CSA haircuts mechanically (they do appear in other contexts such as collateral value adjustments in regulatory settings, but not as a detailed CSA "effective collateral" rule here). I'll use haircuts only as an illustrative toy in Example J and label it.

#### D.4 Collateral frequency, margin call timing, settlement lags (timing risk)

- The books emphasize MPOR/cure period and the idea that collateral at default reflects values from $c$ days earlier.
- I'm not sure the sources standardize "VM call time" vs "settlement lag" as separate operational parameters; we model them explicitly in examples as timing frictions consistent with the MPOR idea.

#### D.5 Rehypothecation (mention only if source-supported)

Rehypothecation is defined as using collateral posted by one counterparty to satisfy collateral requirements of another counterparty.

### E) Keep it focused

- We do not build a full CVA pricing chapter here; we use CVA only as motivation.
- We do not build a regulatory capital chapter; any regulatory-style statements are limited to what the text explicitly states about margin requirements.

---

## 4. Worked Examples (at least 12 numeric examples)

**General reminder:** All examples use the bank perspective, with $E = \max(V - C, 0)$ as the exposure identity.

---

### Example A (Single trade exposure)

**Setup:** One derivative; future MTM at $t = 1y$ is:
- $V = +5$ with probability $0.5$
- $V = -5$ with probability $0.5$

No collateral. Exposure at $t$: $E = \max(V, 0)$.

**Step 1: Compute exposure outcomes**
- If $V = +5$, $E = \max(5, 0) = 5$
- If $V = -5$, $E = \max(-5, 0) = 0$

So $E \in \{5, 0\}$ with probabilities $0.5, 0.5$.

**Step 2: Expected exposure**

$$EE(1y) = \mathbb{E}[E] = 0.5 \cdot 5 + 0.5 \cdot 0 = 2.5$$

Units: currency.

**Step 3: PFE at 95%**

CDF: $P(E \leq 0) = 0.5$, $P(E \leq 5) = 1$.

Therefore $PFE_{95\%}(1y) = 5$.

---

### Example B (Current exposure vs PFE)

**Setup:** Today $t = 0$, MTM is $V(0) = +1$ (no collateral). At $t = 1y$, distribution is as in Example A.

**Current exposure (today):**

$$E(0) = \max(V(0), 0) = \max(1, 0) = 1$$

**Future PFE (1y):** from Example A, $PFE_{95\%}(1y) = 5$.

**Interpretation:** current exposure is small ($1$), but the 95% "how bad can it get in a year?" measure is larger ($5$), aligning with the idea of assessing both current replacement value and potential future replacement costs.

---

### Example C (Netting benefit)

**Setup:** Two trades at $t = 1y$ with perfectly negative dependence:
- **Scenario S1** (prob 0.5): $V_1 = +10$, $V_2 = -8$
- **Scenario S2** (prob 0.5): $V_1 = -10$, $V_2 = +8$

#### (1) Exposure without netting (sum of trade exposures)

$$E_{\text{no net}} = \max(V_1, 0) + \max(V_2, 0)$$

- **S1:** $E_{\text{no net}} = \max(10, 0) + \max(-8, 0) = 10 + 0 = 10$
- **S2:** $E_{\text{no net}} = \max(-10, 0) + \max(8, 0) = 0 + 8 = 8$

So $E_{\text{no net}} \in \{10, 8\}$.

**EE:**

$$EE_{\text{no net}} = 0.5 \cdot 10 + 0.5 \cdot 8 = 9$$

**PFE95:** CDF hits 0.5 at 8 and 1.0 at 10 $\Rightarrow PFE_{95} = 10$.

#### (2) Exposure with netting (exposure of net MTM)

Net MTM $V_{\text{net}} = V_1 + V_2$.

- **S1:** $V_{\text{net}} = 10 - 8 = 2 \Rightarrow E_{\text{net}} = \max(2, 0) = 2$
- **S2:** $V_{\text{net}} = -10 + 8 = -2 \Rightarrow E_{\text{net}} = 0$

This matches the "exposure is $\max(V, 0)$ with $V$ equal to the net MTM" logic.

**EE:**

$$EE_{\text{net}} = 0.5 \cdot 2 + 0.5 \cdot 0 = 1$$

**PFE95:** $PFE_{95} = 2$.

**Netting benefit (numeric):** EE drops from $9$ to $1$; PFE95 drops from $10$ to $2$.

---

### Example D (Collateralized current exposure with VM, idealized zero threshold/no lag)

**Setup:** Same as Example A at $t = 1y$. Assume two-way zero-threshold VM with no lag: collateral always equals $\max(V, 0)$ posted by the out-of-the-money side (concept matches the "zero threshold" collateral description).

So at $t = 1y$:
- If $V = +5$: counterparty posts $C = 5$
- If $V = -5$: bank posts $5 \Rightarrow C = -5$

Use $E = \max(V - C, 0)$.

- If $V = +5$: $E = \max(5 - 5, 0) = 0$
- If $V = -5$: $E = \max(-5 - (-5), 0) = 0$

So current exposure is eliminated in this idealized setting.

**What remains (conceptual):** In reality, MPOR/cure period makes $C$ stale and can reintroduce exposure even with "daily VM."

---

### Example E (Threshold + MTA)

**Setup:** Same distribution as Example A, but collateral terms:
- Threshold $H = 2$
- MTA $m = 1$
- Two-way VM, no lag.

**Book example logic:** collateral required only above threshold. MTA avoids small transfers.

**Rule (chapter convention, consistent with book example):**
- If $V > 0$, call amount $= \max(V - H, 0)$. If call amount $< m$, no transfer.
- If $V < 0$, bank posts similarly; this doesn't affect positive exposure in this example (but matters for posted-collateral exposure).

**Case 1: $V = +5$**
- Call amount $= \max(5 - 2, 0) = 3$
- $3 \geq 1 \Rightarrow C = 3$
- Exposure $E = \max(5 - 3, 0) = 2$

**Case 2: $V = -5$**
- From bank exposure viewpoint: $E = \max(-5 - C, 0)$ will be $0$ under symmetric posting (the counterparty is exposed to the bank instead).

**Results for bank exposure distribution:**
- $E = 2$ w.p. 0.5
- $E = 0$ w.p. 0.5

So $EE = 1$, $PFE_{95} = 2$.

**Interpretation:** threshold creates a "residual unsecured band" (here up to $2$) even when VM exists.

---

### Example F (Margin call timing / settlement lag)

**Setup (illustrative timing model):** VM is called daily but settles with a 1-day lag. I'm not sure the sources standardize this "settlement lag" separately from MPOR; we model it as an operational friction consistent with the broader idea that collateral can be stale.

Assume zero threshold and that collateral held at day $t$ equals yesterday's MTM if positive (for bank exposure).

**3-day MTM path (bank perspective):**
- Day 0: $V_0 = 0$
- Day 1: $V_1 = 4$
- Day 2: $V_2 = 6$
- Day 3: $V_3 = 1$

Let collateral at day $t$ be $C_t = \max(V_{t-1}, 0)$ (lagged by 1 day). Compute daily exposure:

$$E_t = \max(V_t - C_t, 0)$$

- **Day 1:** $C_1 = \max(V_0, 0) = 0 \Rightarrow E_1 = \max(4 - 0, 0) = 4$
- **Day 2:** $C_2 = \max(V_1, 0) = 4 \Rightarrow E_2 = \max(6 - 4, 0) = 2$
- **Day 3:** $C_3 = \max(V_2, 0) = 6 \Rightarrow E_3 = \max(1 - 6, 0) = 0$

**Timing-gap risk quantified:** the lag produces a peak exposure of $4$ on Day 1 despite "daily VM."

---

### Example G (MPOR and IM intuition)

**Source motivation:** IM is described as protecting over a 10-day stressed period with 99% confidence in the bilateral margin rules described. MPOR/cure period is typically 10 or 20 days.

**Illustrative calculation (clearly labeled):**

I'm not sure the sources provide a universal IM formula for all desks; the text gives the target (10-day, 99% stressed coverage). We compute a toy quantile-based buffer consistent with that idea.

Assume daily MTM change $\Delta V \in \{+1, -1\}$ with probability 0.5 each (currency units). Over MPOR $= 10$ days:

$$\Delta V_{10} = \sum_{j=1}^{10} \Delta V_j$$

This is a symmetric random walk with outcomes $-10, -8, \ldots, 10$. Using binomial coefficients (out of $2^{10} = 1024$):

$$P(\Delta V_{10} = 2k - 10) = \binom{10}{k} / 1024$$

We care about exposure increase (positive change), so define $\Delta V_{10}^+ = \max(\Delta V_{10}, 0)$. Then:

| $\Delta V_{10}$ | $\Delta V_{10}^+$ | Prob |
|-----------------|-------------------|------|
| $\leq 0$ | 0 | $638/1024 = 0.6230$ |
| 2 | 2 | $210/1024 = 0.2051$ |
| 4 | 4 | $120/1024 = 0.1172$ |
| 6 | 6 | $45/1024 = 0.04395$ |
| 8 | 8 | $10/1024 = 0.00977$ |
| 10 | 10 | $1/1024 = 0.00098$ |

**CDF of $\Delta V_{10}^+$:**
- at 6: $0.6230 + 0.2051 + 0.1172 + 0.04395 = 0.9893$
- at 8: add $0.00977 \Rightarrow 0.9990$

So the 99% quantile of $\Delta V_{10}^+$ is 8.

**IM proxy (toy):** $IM \approx 8$ currency units.

**Interpretation:** IM covers "how much exposure could jump over MPOR" at a high confidence level, aligning with the stated 10-day/99% intent.

---

### Example H (Netting + collateral together)

**Setup:** 3 trades in one netting set, two scenarios (each prob 0.5). We evaluate at three future dates $t_1, t_2, t_3$ (ignore discounting).

**Collateral terms:** two-way VM with threshold $H = 1$, MTA $m = 0.5$, no lag.

**Trade MTMs:**

| Time | Scenario | $V_1$ | $V_2$ | $V_3$ | Net $V$ |
|------|----------|-------|-------|-------|---------|
| $t_1$ | S1 | 3 | -1 | 2 | 4 |
| $t_1$ | S2 | -2 | 1 | -1 | -2 |
| $t_2$ | S1 | 4 | -2 | 1 | 3 |
| $t_2$ | S2 | -3 | 2 | -2 | -3 |
| $t_3$ | S1 | 2 | -1 | -1 | 0 |
| $t_3$ | S2 | 1 | 1 | -1 | 1 |

**Collateral rule (chapter convention consistent with threshold example):**
- If $V > H$, counterparty posts $C = V - H$ (unless $< m$).
- If $V < -H$, bank posts $-C = (-V) - H$.

**Compute $C$ and exposure $E = \max(V - C, 0)$.**

**At $t_1$:**
- **S1:** $V = 4 \Rightarrow C = 4 - 1 = 3 \Rightarrow E = \max(4 - 3, 0) = 1$
- **S2:** $V = -2 \Rightarrow C = -((2 - 1)) = -1 \Rightarrow E = \max(-2 - (-1), 0) = 0$

**At $t_2$:**
- **S1:** $V = 3 \Rightarrow C = 2 \Rightarrow E = 1$
- **S2:** $V = -3 \Rightarrow C = -(3 - 1) = -2 \Rightarrow E = 0$

**At $t_3$:**
- **S1:** $V = 0 \Rightarrow C = 0 \Rightarrow E = 0$
- **S2:** $V = 1 \Rightarrow C = \max(1 - 1, 0) = 0 \Rightarrow E = 1$

**Compute EE and PFE95 at each time:**
- $t_1$: $E \in \{1, 0\} \Rightarrow EE = 0.5$, $PFE_{95} = 1$
- $t_2$: $E \in \{1, 0\} \Rightarrow EE = 0.5$, $PFE_{95} = 1$
- $t_3$: $E \in \{0, 1\} \Rightarrow EE = 0.5$, $PFE_{95} = 1$

**Interpretation:** threshold leaves a "floor" of exposure at or below $H$, while netting reduces $V$ volatility by offsetting trades.

---

### Example I (Effect of collateral frequency)

**Setup:** Same netting-set MTM path (deterministic) over 10 days; compare:
- Daily VM (immediate settlement)
- Weekly VM (every 5 days)

**Path (bank MTM):**

Day 0 to 10: $V = [0, 1, 3, 2, 5, 4, 6, 3, 2, 1, 0]$

Assume zero threshold, no IM.

#### Daily VM (immediate)

If collateral is updated to $C_t = V_t$ each day, then:

$$E_t = \max(V_t - C_t, 0) = 0 \quad \text{for all } t$$

(Concept matches the idea that VM can track positive MTM.)

- Average exposure over days 1–10: $0$
- Peak exposure: $0$

#### Weekly VM (update at day 0, day 5, day 10)

**Days 1–4:** collateral remains $C = V_0 = 0$
- $E = [1, 3, 2, 5]$

**Day 5:** update collateral to $C = V_5 = 4 \Rightarrow E_5 = \max(4 - 4, 0) = 0$

**Days 6–10:** collateral remains $C = 4$
- Day 6: $E_6 = \max(6 - 4, 0) = 2$
- Days 7–10: $V \leq 4 \Rightarrow E = 0$

**Weekly exposures days 1–10:** $[1, 3, 2, 5, 0, 2, 0, 0, 0, 0]$

- **Peak exposure:** $\max = 5$
- **Average exposure:** $(1 + 3 + 2 + 5 + 0 + 2)/10 = 13/10 = 1.3$

**Interpretation:** less frequent VM increases both peak and average exposure—purely through timing.

---

### Example J (Haircut / collateral quality toy)

**Important label:** I'm not sure the provided sources define a CSA haircut rule in this section set; this is an illustrative "effective collateral" calculation.

**Setup:** At a future time, $V = 10$. Counterparty posts non-cash collateral with nominal value $C = 9$. Haircut $h = 20\%$. Effective collateral:

$$C_{\text{eff}} = (1 - h)C = 0.8 \times 9 = 7.2$$

**Exposure (toy):**

$$E = \max(V - C_{\text{eff}}, 0) = \max(10 - 7.2, 0) = 2.8$$

**Compare to no-haircut:**

$$E_{\text{no haircut}} = \max(10 - 9, 0) = 1$$

**Takeaway:** haircuts reduce effective collateral and increase residual exposure.

---

### Example K (Cross-product netting sets)

**Setup:** Same two trades, two organizational structures:
- Trade A has $V_A = +6$ (always)
- Trade B has $V_B = -5$ (always)

#### Case 1: One netting set

Net $V = 6 - 5 = 1 \Rightarrow E = \max(1, 0) = 1$.

#### Case 2: Two separate netting sets

Exposure is computed per netting set and summed (because netting does not cross sets):
- **Set 1 (Trade A):** $E_1 = \max(6, 0) = 6$
- **Set 2 (Trade B):** $E_2 = \max(-5, 0) = 0$
- **Total** $E = 6$.

**Netting set fragmentation cost:** exposure increases from $1$ to $6$ solely due to contractual netting boundaries, consistent with "aggregate exposures in a way that reflects enforceable netting agreements."

---

### Example L (From exposures to a CVA "preview")

**Source anchor:** The text gives a discretized CVA representation:

$$CVA = \sum_{i=1}^{n} (1 - R) \, q_i \, v_i$$

with $q_i$ the default probability over interval $i$ and $v_i$ the present value of expected net exposure (after collateral) at the midpoint conditional on default.

**Preview approximation (derived, motivational):** If we approximate:
- $q_i \approx \Delta PD(t_i)$ (default probability increment over the interval), and
- $v_i \approx D(0, t_i) \, EE(t_i)$ (PV of expected exposure at $t_i$),

then a desk-friendly approximation is:

$$\boxed{CVA \approx (1 - R) \sum_i D(0, t_i) \, EE(t_i) \, \Delta PD(t_i)}$$

This is a direct reinterpretation of the book's $(1 - R) q_i v_i$ structure.

**Numeric toy:**

- **Times:** $t_1 = 1$, $t_2 = 2$, $t_3 = 3$ years
- **Recovery** $R = 40\% \Rightarrow (1 - R) = 0.6$
- **Discount rate** $r = 2\%$ (flat): $D(0, t) = e^{-0.02t}$
  - $D_1 = e^{-0.02} = 0.9802$
  - $D_2 = e^{-0.04} = 0.9608$
  - $D_3 = e^{-0.06} = 0.9418$
- **Expected exposures** (in \$ millions): $EE = [5, 4, 3]$
- **Default probability increments:** $\Delta PD = [1\%, 1.5\%, 2\%] = [0.01, 0.015, 0.02]$

**Compute each term $D \cdot EE \cdot \Delta PD$ (in \$m):**

- **Term 1:** $0.9802 \times 5 \times 0.01 = 0.9802 \times 0.05 = 0.04901$
- **Term 2:** $0.9608 \times 4 \times 0.015 = 0.9608 \times 0.06 = 0.05765$
- **Term 3:** $0.9418 \times 3 \times 0.02 = 0.9418 \times 0.06 = 0.05651$

**Sum:** $0.04901 + 0.05765 + 0.05651 = 0.16317$

**Multiply by $0.6$:**

$$CVA \approx 0.6 \times 0.16317 = 0.09790 \text{ (\$m)} = \$97,900$$

**Unit check:** $D$ dimensionless; $EE$ in \$m; $\Delta PD$ dimensionless $\Rightarrow$ term in \$m.

**Scope note:** This is only a motivational bridge; full CVA modeling needs careful default timing, dependence assumptions, discounting conventions, and collateral modeling (beyond Chapter 32).

---

## 5. Practical Notes

### 5.1 Vocabulary mapping used on desks

| Term | Definition |
|------|------------|
| **MTM / $V(t)$** | net portfolio value |
| **Exposure / $E(t)$** | $\max(V, 0)$ (uncollateralized) or $\max(V - C, 0)$ (collateralized) |
| **EE** | expected exposure $= \mathbb{E}[E(t)]$ (chapter convention) |
| **PFE** | quantile of exposure (maps to "peak exposure" percentile) |
| **EPE / ENE** | time-averaged exposure measures (definition varies; verify desk convention) |
| **MPOR** | margin period of risk = cure period |
| **VM** | collateral that tracks current MTM |
| **IM** | buffer for MPOR; bilateral IM described as 10-day stressed 99% and segregated; called independent amount in bilateral OTC |
| **Threshold** | unsecured band before VM kicks in |
| **MTA** | minimum movement required before transferring collateral |
| **Netting set** | trades covered by a netting clause (single transaction at default) |

### 5.2 Common pitfalls

- Computing exposure trade-by-trade instead of at the netting set level (violates netting logic).
- Forgetting timing lags and MPOR: collateral is stale by $c$ days; exposure can appear even under collateralization.
- Confusing PFE (quantile) with EE (mean): quantile answers "how bad," mean answers "on average."
- Mixing risk-free vs "collateral discounting" without stating assumptions (and without aligning to the valuation objective vs scenario objective).

### 5.3 Implementation pitfalls

- **Sign conventions:** define clearly whether positive $V$ is bank asset; define collateral sign (Hull convention).
- **Time grid choices:** exposure profiles depend on discretization and MPOR modeling.
- **Collateral modeling:** frequency, thresholds, MTAs, and "stale collateral" assumptions can dominate exposure outcomes.

### 5.4 Verification tests (quick sanity checks)

- Netting cannot increase exposure in the perfect-negative-correlation toy (Example C); check $E_{\text{net}} \leq E_{\text{no net}}$.
- Tighter collateral terms reduce current exposure: lower threshold, lower MTA, more frequent VM.
- PFE ≥ EE for typical distributions; equality occurs in degenerate cases (constant exposure).

---

## 6. Summary & Recall

### 6.1 10-Bullet Executive Summary

1. Exposure is built from MTM: without collateral, $E(t) = \max(V(t), 0)$ for net MTM $V(t)$.
2. With collateral, exposure at default is $E = \max(V - C, 0)$ with a careful collateral sign convention.
3. Netting (ISDA master agreement) treats covered trades as a single transaction at default and can substantially reduce exposure.
4. Without netting, exposure sums positive parts across trades: $\sum \max(V_i, 0)$.
5. With netting, exposure is computed on net MTM: $\max(V_{\text{net}}, 0)$.
6. VM reduces current exposure by tracking positive MTM; thresholds and MTAs leave residual exposure.
7. IM (independent amount) is a buffer against exposure changes during MPOR; the text describes 10-day stressed 99% for many bilateral cases and segregation of IM.
8. MPOR/cure period (typically 10–20 days) makes collateral stale; exposure can appear even with VM.
9. "Peak exposure" is a high percentile of simulated exposure; conceptually it is a real-world scenario question.
10. Wrong-way risk is the dangerous case where exposure and default probability move together; treat as a separate modeling layer.

### 6.2 Cheat Sheet (definitions + key formulas + "what reduces exposure?" checklist)

#### Core formulas

| Formula | Description |
|---------|-------------|
| $E(t) = \max(V(t), 0)$ | Uncollateralized exposure |
| $E(t) = \max(V(t) - C(t), 0)$ | Collateralized exposure |
| $\sum_{i=1}^{N} \max(V_i, 0)$ | No netting exposure (trade sum) |
| High percentile of simulated $E(t)$ | Peak exposure / PFE idea |

#### What reduces exposure? (plumbing checklist)

- More netting coverage (fewer netting sets, broader enforceable netting).
- More frequent VM; lower thresholds; lower MTAs.
- Shorter MPOR / cure period (operationally faster close-out) — where contract/operations allow.
- Adequate IM sized to MPOR stress (as described).

### 6.3 Flashcards (35 Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What does $V(t)$ represent here? | Net MTM of all trades in a netting set at time $t$. |
| 2 | Bank exposure without collateral? | $E(t) = \max(V(t), 0)$. |
| 3 | Bank exposure with collateral? | $E = \max(V - C, 0)$. |
| 4 | What is netting (contractually)? | A clause treating all covered trades as a single transaction on default. |
| 5 | Exposure without netting for values $V_i$? | $\sum \max(V_i, 0)$. |
| 6 | Exposure with netting, no collateral? | $\max(V_{\text{net}}, 0)$. |
| 7 | What is VM? | Collateral posted to track positive MTM to the other side. |
| 8 | What is IM in bilateral OTC also called? | Independent amount. |
| 9 | What is the cure period / MPOR? | Delay (typically 10–20 days) between collateral cessation and close-out. |
| 10 | Why does MPOR matter? | Collateral at default reflects value $c$ days earlier, creating exposure if MTM moves. |
| 11 | What is "peak exposure"? | High percentile of simulated exposures at each future time. |
| 12 | Real-world vs risk-neutral issue for peak exposure? | Peak exposure is conceptually scenario analysis (real-world), though risk-neutral is often used. |
| 13 | Define EE(t) (chapter convention). | $EE(t) = \mathbb{E}[E(t)]$. |
| 14 | Define $PFE_q(t)$ (chapter convention). | $q$-quantile of $E(t)$. |
| 15 | What is threshold? | VM required only above threshold; e.g., value 11 vs threshold 10 implies 1 collateral. |
| 16 | What is MTA? | Minimum transfer amount to avoid small collateral moves. |
| 17 | Two-sided vs one-sided collateral? | Two-sided both post; one-sided typically only less creditworthy posts. |
| 18 | IM target described for bilateral rules? | 10-day stressed period, 99% confidence. |
| 19 | How is IM typically held (as described)? | Segregated / held by a third-party custodian. |
| 20 | What is wrong-way risk? | Exposure rises when counterparty becomes more likely to default. |
| 21 | What does "aggregate exposures reflecting netting" mean? | Compute exposures on netted portfolios under enforceable agreements. |
| 22 | What is "current exposure" in words? | Replacement cost if counterparty defaulted now. |
| 23 | What is "potential future exposure" in words? | Future distribution of exposure; "how big it could get." |
| 24 | When can exposure be positive even if you posted collateral? | If posted collateral exceeds what you owed and isn't returned on default. |
| 25 | What is rehypothecation? | Reuse of posted collateral to meet other collateral needs. |
| 26 | Why does netting break "transaction-by-transaction" CVA? | Exposure depends on the net portfolio, not per trade. |
| 27 | In Example A, what is EE? | 2.5. |
| 28 | In Example A, what is PFE95? | 5. |
| 29 | In Example C, what is netted EE? | 1. |
| 30 | In Example C, what is unnetted EE? | 9. |
| 31 | What is the core operational driver of exposure under VM? | Timing: how quickly collateral updates vs MTM moves. |
| 32 | What is the modeling implication of MPOR for simulations? | Need values at $t_i^* - c$ and $t_i^*$. |
| 33 | What do thresholds do to exposure? | Create residual unsecured exposure bands. |
| 34 | What do MTAs do to exposure? | Leave small moves uncollateralized. |
| 35 | CVA "preview" structure? | Sum of $(1 - R) q_i v_i$ terms. |

---

## 7. Mini Problem Set (18 questions)

**Instructions:** Increasing difficulty. Brief solution sketches for Q1–Q9 only.

---

**Q1 (Easy)** Single trade: $V \in \{-2, +6\}$ with probs $(0.7, 0.3)$. Compute $E$, $EE$, $PFE_{90}$.

> **Sketch:** $E = \{0, 6\}$. $EE = 0.3 \cdot 6 = 1.8$. $PFE_{90} = 6$ because CDF at 0 is 0.7 < 0.9.

---

**Q2 (Easy)** If today $V(0) = -4$ and no collateral, what is current exposure?

> **Sketch:** $E(0) = \max(-4, 0) = 0$.

---

**Q3 (Easy)** Two trades with values $V_1 = +3$, $V_2 = -1$. Compute exposure with and without netting.

> **Sketch:** No net: $3 + 0 = 3$. Net: $\max(2, 0) = 2$.

---

**Q4 (Easy)** Threshold $H = 5$: if net $V = 7$, what VM is called (chapter convention) and what residual exposure remains?

> **Sketch:** Call $= 7 - 5 = 2$. Exposure $= 7 - 2 = 5$.

---

**Q5 (Easy)** Cure period $c$: if $V(t) = 12$ but $V(t - c) = 9$ under zero-threshold VM, what is exposure at $t$?

> **Sketch:** Collateral $C = 9$. $E = \max(12 - 9, 0) = 3$. (Matches Example-style logic.)

---

**Q6 (Medium)** Build a two-scenario example where netting reduces EE but increases ENE (counterparty exposure to the bank). Explain.

> **Sketch:** Use scenarios where netting reduces positive tail but increases negative tail; compute both sides' positive parts.

---

**Q7 (Medium)** Show numerically that $\max(V_1 + V_2, 0) \leq \max(V_1, 0) + \max(V_2, 0)$ in two different states.

> **Sketch:** Pick $(V_1, V_2) = (5, -4)$ and $(-5, 4)$.

---

**Q8 (Medium)** Weekly vs daily VM on a 5-day rising MTM path. Compute max exposure under each.

> **Sketch:** Daily immediate gives ~0; weekly leaves exposure equal to MTM until call.

---

**Q9 (Medium)** Using the "peak exposure" definition, explain how to compute the 97.5% peak exposure from 10,000 simulated exposures.

> **Sketch:** Sort exposures; pick the 250th highest (as stated).

---

**Q10 (Medium)** Extend Example E by adding a 1-day lag; compute day-by-day exposures for a 4-day path.

---

**Q11 (Medium)** Construct a 3-trade example where splitting into two netting sets increases PFE but leaves EE unchanged. Can it happen?

---

**Q12 (Hard)** Create a toy wrong-way-risk example where exposure is higher exactly in the "bad credit" state; compare unconditional EE vs conditional EE.

---

**Q13 (Hard)** Suppose IM is sized to a 99% 10-day move but your MPOR operationally becomes 20 days. Qualitatively, what happens to residual exposure?

---

**Q14 (Hard)** Propose a simple time-averaging scheme for EPE on a non-uniform time grid and discuss bias sources.

---

**Q15 (Hard)** In the Hull-style collateral sign convention, give an example where $C < 0$ increases exposure.

---

**Q16 (Hard)** Describe how disputes in margin calls could be represented as a stochastic delay; what impact on PFE would you expect?

---

**Q17 (Hard)** Using $CVA = \sum (1 - R) q_i v_i$, map each term to a "risk report" quantity you'd need operationally.

---

**Q18 (Hard)** Explain why "peak exposure" and CVA naturally use different probability measures (real-world vs risk-neutral).

---

## 8. Source Map

### Directly source-backed (high confidence)

- Netting definition ("single transaction" on default) and its risk-reducing effect.
- Exposure without netting: $\sum \max(V_i, 0)$.
- Exposure with netting/no collateral: $\max(V, 0)$ where $V$ is net MTM.
- Collateralized exposure identity $E = \max(V - C, 0)$ and collateral sign convention.
- Cure period/MPOR definition and typical 10–20 day magnitude; collateral reflects value $c$ days earlier; simulation must consider $t_i^* - c$.
- "Peak exposure" as a high percentile of simulated exposures and the risk-neutral vs real-world conceptual issue.
- Bilateral VM/IM/threshold/MTA descriptions; IM called independent amount; IM coverage target (10-day, 99%, stressed) and segregation.
- Risk policy framing: assess current replacement value and potential future replacement costs; aggregate exposures reflecting netting.
- Rehypothecation definition.
- CVA discretization structure $\sum (1 - R) q_i v_i$ and definitions of $q_i, v_i$.

### Derived / constructed for this chapter (consistent with sources, but not always explicitly named)

- EE as $\mathbb{E}[E(t)]$, PFE as quantile of $E(t)$, using the same idea as "peak exposure" percentiles.
- The explicit inequality showing netting cannot increase exposure (mathematical consequence of "single transaction" netting).
- Example L "CVA preview" mapping from $(1 - R) q_i v_i$ to $(1 - R) D \cdot EE \cdot \Delta PD$ as a discretization/interpretation.

### Uncertain / depends on legal/operational conventions

- **EPE/ENE exact definitions and weighting:** I'm not sure the provided excerpts standardize them; desks vary (time grids, discounting, collateral inclusion).
- **Settlement-lag modeling vs MPOR:** I'm not sure the sources separate intraday call/settlement mechanics; examples treat lag explicitly as an illustrative extension of "stale collateral" logic.
- **Haircuts as CSA mechanics:** I'm not sure they are defined here; Example J is illustrative only.
