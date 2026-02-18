# Chapter 32: Counterparty Exposure Basics — Netting, Collateral, and Margin Timing

---

## Learning Objectives
- Translate netting-set MTM and collateral into exposure: $V(t), C(t) \rightarrow E(t)$.
- Explain how close-out netting changes trade-level exposure into *netting set* exposure.
- Connect CSA terms (VM/IM, thresholds, MTAs, haircuts, rehypothecation) to residual unsecured exposure.
- Explain why timing (settlement lags and MPOR/cure periods) creates “gap risk” even with daily VM.
- Compute and interpret $EE(t)$, $PFE_q(t)$, and $EPE$ from simulated (or toy) exposure distributions.

## Introduction

When Lehman Brothers filed for bankruptcy on September 15, 2008, it had over a million derivatives transactions outstanding with about 8,000 counterparties. Each of those counterparties faced an immediate question: *How much do they owe us, and how much of that will we ever recover?*

This question—the *exposure* question—is the foundation of counterparty credit risk. Before you can price CVA, stress-test a portfolio, or set a credit limit, you must first answer: what is the potential loss if the counterparty defaults tomorrow? And critically: how does that exposure evolve over time as markets move?

The challenge is that derivatives exposure is unlike loan exposure. A loan's exposure is simply its outstanding principal. But a derivative's exposure is uncertain—it depends on market movements, the structure of the trade, and the web of legal and operational arrangements that govern what happens when things go wrong. The exposure can be near zero today and substantial tomorrow depending on whether the portfolio is in-the-money or out-of-the-money.

Prerequisites: [Chapter 2](chapters/chapter_02_time_value_discount_factors_replication.md), [Chapter 14](chapters/chapter_14_key_rate_dv01_bucket_exposures.md)  
Follow-on: [Chapter 33](chapters/chapter_33_collateral_discounting_ois.md), [Chapter 34](chapters/chapter_34_xva_overview.md), [Chapter 36](chapters/chapter_36_survival_probabilities_hazard_rates.md)

This chapter builds the **exposure primitives**—the building blocks you need before tackling CVA, XVA, or any counterparty risk framework:

1. **Exposure as a function of MTM** — Why exposure equals $\max(V, 0)$ and how collateral modifies this
2. **Netting and close-out** — How ISDA master agreements transform counterparty risk and why "netting sets" matter
3. **Collateral mechanics** — VM, IM, thresholds, MTAs, haircuts, and how they reduce (but never eliminate) exposure
4. **Central clearing vs. bilateral clearing** — How CCPs change the exposure landscape
5. **The margin period of risk (MPOR)** — Why even "fully collateralized" trades carry residual exposure
6. **Exposure simulation** — How Monte Carlo generates PFE and EE profiles
7. **Incremental exposure** — How new trades affect portfolio CVA
8. **Wrong-way risk** — When exposure and default probability move together

None of this machinery is glamorous. But every CVA calculation, every credit limit decision, and every capital charge traces back to these exposure primitives. Get them wrong, and nothing downstream makes sense.

---

## Perspective and Signs

Before diving into the mechanics, lock down a sign convention. Counterparty exposure calculations are notorious for sign errors—getting the direction of collateral or exposure wrong can reverse your conclusion entirely.

### 32.0.1 Bank Perspective and Sign Rules

We adopt the **bank/dealer perspective** throughout. The bank has a portfolio of derivatives with a counterparty, and we measure the bank's credit risk to that counterparty.

| Item | Description |
|------------|-------------|
| **MTM sign** | We define $V(t)$ as the net mark-to-market of all trades in the netting set at time $t$, from the bank’s perspective. **$V(t) \gt 0$** means the bank is in-the-money (the counterparty owes the bank). |
| **Collateral sign** | We define $C(t)$ as collateral held by the bank at time $t$. $C(t)\gt 0$ means the counterparty has posted collateral; $C(t)\lt 0$ means the bank has posted collateral (amount $-C(t)$). |
| **Exposure** | Bank exposure to counterparty default at time $t$: $E(t)=\max(V(t)-C(t),0)$. |

We keep the full symbol list in `## Notation` near the end; the only symbols you need immediately are $V(t)$, $C(t)$, $E(t)$, and the CSA parameters “threshold” $H$, “minimum transfer amount” $m$, and “cure period / MPOR” $c$.

---

## 32.1 Exposure as a Function of MTM

### 32.1.1 The Basic Exposure Identity

Consider a dealer with a single derivatives transaction outstanding with a counterparty. The value of the transaction to the dealer is $V$. If the counterparty defaults, what does the dealer lose?

If $V \lt 0$, the dealer owes the counterparty—there is no credit loss (in fact, the dealer benefits from not having to pay). But if $V \gt 0$, the counterparty owes the dealer, and the dealer becomes an unsecured creditor for the amount $V$.

**Anchor (definition):** Ignoring recovery and collateral for the moment, the dealer’s exposure at time $t$ is the positive part of the net MTM:

$$\boxed{E(t) = \max(V(t), 0)}$$

> **Analogy: The Casino Chip**
>
> Counterparty risk is like holding a casino chip.
>
> *   **If you are winning (USD 1000 chip)**: You have exposure to the casino. You worry: "Does the cage have enough cash to pay me?"
> *   **If you are losing**: You owe the casino. You have *no* exposure to them (they have exposure to *you*).
> *   **Max(V, 0)**: You only care about the counterparty's health when you are winning. If you are losing, their bankruptcy doesn't hurt you (in fact, you might get out of paying!).

**This is the fundamental exposure identity** (without collateral). Exposure is the positive part of the portfolio's MTM.

**Unit check:** $V$ is currency $\Rightarrow E$ is currency.

**Sanity checks:**
- If $V(t) \leq 0$, the bank owes the counterparty, so the bank's credit exposure is zero.
- If $V(t) = 0$, the trade is at-the-money, and exposure is zero.
- Exposure is always non-negative.

### 32.1.2 Current Exposure vs. Potential Future Exposure

The exposure identity $E = \max(V, 0)$ can be evaluated at any point in time:

**Current exposure (CE)** is exposure computed from today’s MTM (and today’s collateral, if any). Informally: it is the replacement cost *right now* if the counterparty defaults immediately (before recovery and close-out costs).

**Potential future exposure (PFE)** recognizes that default (if it happens) occurs in the future, after markets have moved. PFE asks: *how large could exposure be at a future date, at a chosen confidence level?*

Operationally, PFE is usually computed by simulation: simulate many future paths for the netting set MTM (and collateral), compute $E(t)$ on each path, and take a high percentile of that distribution at each future time.

Two commonly used summary objects are:
- **$PFE_q(t)$:** the $q$-quantile of $E(t)$ at time $t$ (e.g., $q=95\\%$ or $97.5\\%$).
- **Maximum PFE / MPFE:** the maximum of $PFE_q(t)$ over a time window (often the trade life), used as a single “peak” number.

**Check (sanity):** if there is no uncertainty about future MTM (a deterministic $V(t)$), then $EE(t)=E(t)$ and $PFE_q(t)=E(t)$ for any $q$: there is no “tail” because there is no distribution.

---

## 32.2 Netting: From Trade-Level to Portfolio-Level Exposure

### 32.2.1 The Legal Basis for Netting

Suppose a bank has 50 different derivatives trades with a counterparty—some in-the-money to the bank, some out-of-the-money. If the counterparty defaults, what happens?

Without netting, the counterparty's bankruptcy trustee might adopt a "cherry-picking" strategy: claim full payment on trades where the bank owes money, while offering only a fractional recovery on trades where the counterparty owes the bank. The bank would have to pay its full obligations but receive only cents on the dollar.

**Anchor (definition):** *Close-out netting* is a contractual arrangement where, upon counterparty default, losses are evaluated at the **netted portfolio** level rather than deal-by-deal.

**Expand (mechanics):** With close-out netting, trades covered by the same master agreement are treated as a single net obligation for close-out. This makes “offsetting trades” (one in-the-money, one out-of-the-money) actually offset in the default settlement.

This legal protection is not automatic. It requires:
1. A master agreement (typically an ISDA Master Agreement) between the parties
2. A legal opinion confirming enforceability in the relevant jurisdictions
3. Proper documentation and identification of which trades are covered

**Check (toy default settlement):** Suppose two swaps have values $+1\text{mm}$ and $-1\text{mm}$ to the bank at the default time, and the counterparty recovery is $40\\%$.
- With netting: the net is $0$ so there is no loss.
- Without netting: only the $+1\text{mm}$ contributes to loss, so the loss is $(1-0.40)\times 1\text{mm}=0.6\text{mm}$.

### 32.2.2 The Mathematics of Netting

Let trades have individual values $V_1, V_2, \ldots, V_N$ at default time.

**Without netting** (trade-by-trade exposure):

$$E_{\text{no net}} = \sum_{i=1}^{N} \max(V_i, 0)$$

Each positive trade value contributes to exposure; negative trade values are ignored (you still owe them, but you're at risk on the positive ones).

**With netting** (net portfolio exposure):

The "single transaction" principle means the close-out is computed on the net value:

$$V_{\text{net}} = \sum_{i=1}^{N} V_i$$

$$\boxed{E_{\text{net}} = \max\left(\sum_{i=1}^{N} V_i, 0\right)}$$

Interpretation: without netting, exposure behaves like a **sum of call options** on each trade; with netting, it behaves like a **single call option** on the netted portfolio value.

### 32.2.3 The Netting Benefit

The key mathematical result is that netting can only reduce exposure:

$$\boxed{\max\left(\sum_{i=1}^{N} V_i, 0\right) \leq \sum_{i=1}^{N} \max(V_i, 0)}$$

This follows from the subadditivity of the $\max(\cdot, 0)$ function. The inequality is strict whenever some trades are positive and some are negative—exactly the situation where gains offset losses.

**Check (when netting does *not* help):** If all trades have the same sign at close-out (all $V_i\ge 0$ or all $V_i\le 0$), then the inequality becomes an equality. Netting only creates a benefit when positive and negative MTMs coexist so that gains can offset losses in the close-out amount.

**Perfect offset example:** Suppose trade 1 has $V_1 = +10$ and trade 2 has $V_2 = -10$ (perfectly offsetting). Without netting, exposure is $\max(10,0) + \max(-10,0) = 10$. With netting, exposure is $\max(0,0) = 0$. The netting benefit is complete.

**Imperfect offset example:** Suppose $V_1 = +10$ and $V_2 = -8$. Without netting, exposure is $10 + 0 = 10$. With netting, exposure is $\max(2,0) = 2$. Netting reduces exposure from 10 to 2.

### 32.2.4 Netting Sets in Practice

The term **netting set** refers to the collection of trades whose close-out amount is computed on a net basis and whose collateral is managed together under a single CSA.

Operationally, you cannot mix trades from different legal entities, or trades under different master agreements, into a single netting set. Netting is only available where it is **legally enforceable** for the relevant trade population.

---

## 32.3 Collateral and Margin

### 32.3.1 How Collateral Modifies Exposure

**Anchor (default close-out identity):** Using the sign convention that $C\gt 0$ means collateral posted by the counterparty to the bank and $C\lt 0$ means collateral posted by the bank, the **non-defaulting party’s claim** is $\max(V-C,0)$ and the **payment it must make** is $\max(C-V,0)$.

From the bank’s perspective, exposure to counterparty default at time $t$ is therefore:

$$\boxed{E(t) = \max(V(t) - C(t), 0)}$$

**Expand (mechanics):**
- If $V\gt C$, the bank keeps collateral $C$ and is an unsecured creditor for the residual $V-C$.
- If $V\lt C$, the bank keeps $V$ worth of collateral and returns the excess $C-V$.
- If $C\lt 0$, the bank has posted collateral. “Over-posted” collateral can become an exposure if the counterparty defaults while holding it.

**Check (numbers):**
- If $V=50$ and $C=45$, then $E=\max(50-45,0)=5$. (A residual unsecured claim.)
- If $V=50$ and $C=55$, then $E=0$ and the bank returns $55-50=5$ of collateral.
- If $V=-50$ and $C=-55$, then $E=\max(-50-(-55),0)=5$. (Excess collateral posted may not come back.)

**Check (interpret the “$V\lt 0$ but $E\gt 0$” case):** The last line is often the surprise. Exposure is about what you can lose if the counterparty defaults, not about whether your MTM is positive. If you have posted more collateral than you currently owe (because of timing, thresholds, haircuts, or disputes), that excess collateral is effectively a receivable from the counterparty and can be at risk while they hold it.

### 32.3.2 Variation Margin (VM)

**Variation margin** is collateral exchanged frequently to track the current MTM under the CSA.

In a **perfect, instantaneous, zero-threshold VM world**, current exposure would be driven to zero. If $V$ changes, collateral immediately adjusts so that $C = V$ when $V \gt 0$ (or $C = V$ for two-way VM). Then $E = \max(V - V, 0) = 0$.

In reality, this idealized state is never achieved because of:
1. **Thresholds** — collateral only required above a threshold
2. **Minimum transfer amounts (MTAs)** — small moves don't trigger transfers
3. **Timing lags** — collateral reflects yesterday's (or earlier) MTM, not today's
4. **Cure period / MPOR** — between default and close-out, collateral is stale

### 32.3.3 Initial Margin (IM) and Independent Amounts

**Initial margin** is a buffer posted at trade inception (or when exposure grows) to cover potential future exposure during the margin period of risk.

Conceptually, IM is intended to cover adverse MTM moves over the MPOR at a high confidence level. The specific calibration horizon and confidence level are **convention-sensitive** inputs that must match the CSA/regime you are modeling.

In bilateral OTC, IM is often called an **independent amount**. The purpose is the same: protect against exposure that emerges during the gap between the last collateral call and close-out.

> **Connection to discounting:** The presence of collateral affects not just exposure but also *how we discount* derivative cashflows. When collateral earns the overnight rate, OIS discounting becomes appropriate. This connection between collateral and discounting is developed fully in **Chapter 33**.

### 32.3.4 Haircuts and Eligible Collateral

Not all collateral is equal. When securities (rather than cash) are posted as collateral, their value is reduced by a **haircut** to account for potential price declines before liquidation.

**Anchor (definition):** A haircut is a percentage reduction applied to the market value of a security to determine its value for collateral purposes.

The effective collateral value is:

$$\boxed{C_{\text{effective}} = C_{\text{market}} \times (1 - \text{haircut})}$$

**Check (numbers):** A $10\\%$ haircut means a USD 100 security counts as USD 90 of collateral; a $30\\%$ haircut means it counts as USD 70.

> **Desk Reality:** When a CSA permits multiple collateral types, the posting party will optimize by posting the “cheapest-to-deliver” collateral after haircuts and eligibility rules.
> **Common break:** Modeling assumes “collateral = cash” or assumes a fixed collateral type; the effective $C(t)$ can change when the posting party switches what they deliver.
> **What to check:** Recompute $C_{\text{effective}}$ under each eligible collateral type and haircut; do not mix *market value* and *collateral value*.

### 32.3.5 Thresholds and MTAs

Thresholds and MTAs are operational features that intentionally leave a band of residual uncollateralized exposure.

A **threshold** means VM is required only above a certain amount. A simple stylized rule is:
$$
\text{VM call}(t)=\max(V(t)-H,0),
$$
where $H$ is the threshold (for the bank's exposure to the counterparty).

The threshold creates a **residual unsecured band**—even with VM in place, exposure up to the threshold level remains uncollateralized.

The **minimum transfer amount (MTA)** avoids operational "nuisance" transfers for small amounts. If the required transfer is less than the MTA, no transfer occurs.

Both features increase residual exposure but reduce operational friction.

### 32.3.6 Rehypothecation and Segregation

**Anchor (definition):** Rehypothecation means collateral received as a guarantee can be reused (as an investment or as further collateral).

**Expand (why it matters):** Rehypothecation can improve funding efficiency for the collateral receiver, but it adds **credit risk for the collateral poster**. If the receiver defaults while holding (and having reused) your collateral, recovering it can be harder. One mitigation is **segregation**: keeping collateral in a segregated third-party account, which can improve its seniority versus other unsecured claims.

**Check (failure mode):** If you post collateral when the bank is in-the-money, and the bank reinvests/rehypothecates it, then a fast market move that flips the MTM followed by the bank's default can leave you exposed to both (i) an MTM loss and (ii) impaired recovery of the collateral you posted.

### 32.3.7 Downgrade Triggers (Preview)

CSAs may include **downgrade triggers**—clauses stating that if one party's credit rating falls below a specified level, the other party can demand additional collateral or terminate all transactions.

Downgrade triggers are a double-edged sword:
- They provide some protection against counterparty deterioration
- But they do not protect against sudden "jump-to-default" events (e.g., AA to default)
- If a company has many triggers across its contracts, a single downgrade can generate large collateral calls—precisely when liquidity is most strained

> **Desk Reality:** Variation margin is an operational loop: both sides value the netting set, agree (or dispute), then collateral settles with a lag.
> **Common break:** The exposure model assumes instantaneous collateral updates, but real-world settlement/disputes mean $C(t)$ often reflects the *last agreed and settled* MTM, not today's.
> **What to check:** Reconcile (i) trade population, (ii) valuation curves/models, and (iii) settlement timing; when in doubt, model collateral using the last *agreed/settled* valuation date rather than the last valuation run.

---

## 32.4 Central Clearing vs. Bilateral Clearing

### 32.4.1 The Two Clearing Regimes

OTC derivatives can be cleared in two main regimes: bilateral clearing and central clearing.

**Bilateral clearing:** Each pair of market participants faces each other under a master agreement and CSA. Exposure is to the original counterparty (subject to bilateral netting and collateral terms).

**Central clearing:** A central counterparty (CCP) interposes itself and becomes the counterparty to both sides via offsetting trades. After clearing, each participant’s exposure is primarily to the CCP (not to the original trading counterparty).

### 32.4.2 How CCPs Operate

A CCP operates similarly to an exchange clearing house:

1. **Initial margin (IM):** Both sides post IM. A common design goal is that IM should cover market moves over a multi-day close-out horizon at high confidence; the exact horizon and confidence level are CCP- and product-specific.
2. **Variation margin (VM):** VM is exchanged frequently (often daily and sometimes intraday) to keep the CCP close to flat as market values move.
3. **Default fund:** Members contribute to a guaranty fund that mutualizes losses beyond a defaulting member's margin.

### 32.4.3 The CCP Default Waterfall

If a CCP member defaults, losses are absorbed in a specific order (a default **waterfall**):

1. The initial margin of the defaulting member
2. The default fund contribution of the defaulting member
3. The default fund contributions of other members
4. The equity of the CCP

This structure means that even highly creditworthy counterparties face some mutualized credit risk when clearing through a CCP—they could lose part of their default fund contribution if another member defaults.

### 32.4.4 Bilateral vs. CCP: A Comparison

| Feature | Bilateral Clearing | CCP Clearing |
|---------|-------------------|--------------|
| **Counterparty exposure** | To original counterparty | To CCP |
| **Netting** | Within bilateral netting set | Across all trades cleared at that CCP |
| **Margin terms** | CSA-defined VM/IM, thresholds, MTAs, eligible collateral | CCP-defined VM/IM methodology + default fund |
| **Default handling** | Close-out under the master agreement | CCP default management / close-out process |
| **Mutualization** | None | Default fund contributions |

### 32.4.5 Netting Efficiency and Fragmentation

Central clearing can increase netting efficiency because a market participant can net all trades cleared at a single CCP, even if originally transacted with many different dealers.

However, fragmentation matters. If a bank clears similar risk in multiple silos (different CCPs, or cleared vs. non-cleared), it cannot net across those silos.

Moreover, some trades cannot be centrally cleared (nonstandard transactions, some FX transactions). Splitting portfolios across cleared and bilateral books can reduce netting and change both exposure and margin requirements.

> **Desk Reality: Why Clearing Matters for Credit Limits**
>
> A trade executed bilaterally with Counterparty X consumes credit limit against X. The same trade cleared through a CCP consumes no credit limit against X—instead, it requires IM and default fund contributions to the CCP.
>
> For a trading desk, this changes deal economics:
> - Bilateral trade: Need credit approval, may hit limit, CVA charge against X
> - Cleared trade: No credit limit usage against X, but must fund IM
>
> Many desks now have "clearing mandates" requiring standard trades to be cleared whenever possible, precisely to preserve bilateral credit lines for nonstandard trades that *must* be bilateral.

---

## 32.5 The Margin Period of Risk (MPOR)

### 32.5.1 Why MPOR Matters

Even with a two-way, zero-threshold collateral agreement, exposure at default can be non-zero. The reason is timing: collateral available at close-out may reflect an older MTM.

**Expand (mechanics):** During MPOR:
1. The counterparty is in distress and stops responding to margin calls (and may stop returning excess collateral).
2. The bank invokes early termination and a close-out process starts.
3. Markets keep moving while collateral is effectively frozen at the last agreed/settled level.

### 32.5.2 Collateral at Default Is Stale

**Anchor (simulation convention):** In this calculation, it is usually assumed that the counterparty stops posting collateral and stops returning any excess collateral held $c$ days before a default. The parameter $c$—the number of days before default when the counterparty stops posting and returning collateral—is typically 10 or 20 days and is referred to as the *cure period* or *margin period of risk*. In order to know what collateral is held at the midpoint of an interval in the event of a default, it is necessary to calculate the value of transactions $c$ days earlier.

This stale-collateral effect means that even under "perfect collateralization," exposure can emerge from market moves during MPOR.

**Expand (timeline in words):**
- Up to some last “good” margin date, both sides exchange VM based on an agreed MTM, and collateral settles with an operational lag.
- Near default, margin calls may be disputed or unanswered; after default, no further collateral moves while close-out occurs.
- The net effect is that at close-out time $\tau$, the collateral you actually hold is closer to a *past* MTM than to $V(\tau)$. The MPOR parameter $c$ is a modeling knob that represents this margin-outage window.

**Check (magnitude):** In the stylized two-way, zero-threshold VM convention $C(\tau)\approx V(\tau-c)$, the gap exposure is the positive part of the MTM change over the outage window:
$$
E(\tau)=\max(V(\tau)-V(\tau-c),0).
$$
So if a portfolio MTM can move by “a few million” over $c$ days in stressed scenarios, a few million of gap exposure is not a bug—it is the mechanism.

> **Analogy: The Stale GPS**
>
> Collateral under MPOR is like a GPS that shows where you were 20 minutes ago, not where you are now. If you're driving fast on a twisting road, your "position" shown on the GPS (collateral) can be very different from your actual position (MTM). The faster the market moves, the larger this gap.

### 32.5.3 A Stale-Collateral Check (Toy)

Assume two-way, zero-threshold VM, and a cure period of $c=20$ days. On a given simulation path:
- Portfolio value at the default time is $V(\tau)=50$.
- Portfolio value 20 days earlier is $V(\tau-c)=45$.

Under the “stale collateral” convention, the calculation assumes the bank has collateral worth $45$ at the default time $\tau$. The exposure is:
$$
E(\tau)=\max(50-45,0)=5.
$$

If instead $V(\tau-c)=55$, the assumption implies collateral worth $55$ at $\tau$ and $E(\tau)=0$ (and excess collateral would be returned).

> **Pitfall — Fully collateralized ≠ zero exposure:** People mentally equate “daily VM” with “no counterparty exposure.”
> **Why it matters:** MPOR (settlement lags, disputes, and close-out time) means $C(\tau)$ typically reflects an *older* MTM than $V(\tau)$.
> **Quick check:** With $c=20$ days, if $V(\tau)=50$ and $V(\tau-c)=45$, then $E(\tau)=5$ even under zero-threshold VM.

---

## 32.6 Exposure Metrics: CE, EE, PFE, EPE, and EAD

This section standardizes the core exposure metrics used by risk, credit limits, and (downstream) CVA.

### 32.6.1 Current Exposure (CE)

**Current exposure** is exposure computed from today’s MTM and today’s collateral:
$$
CE := E(0)=\max(V(0)-C(0),0).
$$
Informally: it is the replacement cost *right now* if the counterparty defaulted immediately (before recovery and close-out costs).

### 32.6.2 Potential Future Exposure (PFE) and Maximum PFE (MPFE)

**Potential future exposure (PFE)** for a given date is the maximum exposure at that date, with a high degree of statistical confidence (i.e., a high percentile of the exposure distribution):
$$
\boxed{PFE_q(t) := \inf\\{x : P(E(t) \leq x) \geq q\\}}.
$$

**Maximum PFE / MPFE** is the maximum of $PFE_q(t)$ over a time window (often the trade life). It is used as a single “peak” number for credit limits.

**Important modeling point:** PFE is usually computed via simulation: for each future time, simulate the portfolio value (to obtain $V(t)$, then $E(t)$). There is no default simulation involved: only the portfolio is simulated, not the default of the counterparty. Default probabilities enter later when you convert exposure into expected loss (e.g., CVA).

**Check (order statistic):** With $N$ simulated exposures at a fixed time $t$, $PFE_q(t)$ is the $q$-quantile of the $N$ numbers. For example, if $N=10{,}000$ and $q=97.5\\%$, it is the 250th largest exposure.

### 32.6.3 Expected Exposure (EE)

**Expected exposure at time $t$** is the mean of the exposure distribution:
$$
\boxed{EE(t) := \mathbb{E}[E(t)] = \mathbb{E}[\max(V(t) - C(t), 0)]}.
$$

### 32.6.4 Expected Positive Exposure (EPE)

**EPE** is a time-aggregated measure of $EE(t)$. A common convention is a time-weighted average:
$$
EPE := \frac{1}{T}\int_0^T EE(t)\\,dt
\quad\text{or}\quad
EPE \approx \sum_i w_i\\,EE(t_i).
$$
Because definitions vary (time grid, weighting, collateral treatment), always confirm the exact EPE definition used by your system.

### 32.6.5 Exposure at Default (EAD) (Preview)

**Exposure at default** is exposure evaluated at the (random) default time $\tau$:
$$
EAD := E(\tau).
$$
This object appears in some capital and expected-loss calculations, but the detailed frameworks are outside this chapter’s scope.

### 32.6.6 Expected Negative Exposure (ENE) and DVA (Preview)

From the counterparty’s perspective, the relevant “mirror” quantity is the positive part of $(C-V)$. Under our sign convention, define:
$$
\boxed{ENE(t) := \mathbb{E}[\max(C(t)-V(t),0)] = \mathbb{E}[\max(-(V(t)-C(t)),0)]}.
$$

Downstream, **DVA** uses ENE in an analogous way to how CVA uses EE. A full treatment of DVA belongs in Chapter 34.

### 32.6.7 A Note on $P$ vs. $Q$

In one common convention, PFE is framed as a $P$-percentile (physical measure) for limit/stress purposes, while pricing adjustments (like CVA) rely on risk-neutral ($Q$) valuation. In practice, institutions vary in how they implement this split. The safe operational rule: always verify which measure and calibration your exposure engine is using before comparing numbers across systems.

---

## 32.7 Monte Carlo Simulation for Exposure

### 32.7.1 The Simulation Framework

Computing exposure metrics (EE, PFE, EPE) requires simulating how portfolio values evolve under many possible future scenarios. This section explains the Monte Carlo procedure.

**Step 1: Identify Risk Factors**

The portfolio value $V(t)$ depends on market variables: interest rates, FX rates, credit spreads, equity prices, volatilities. These are the **risk factors** to be simulated.

For an interest rate swap portfolio, typical risk factors include:
- Short rate or yield curve level
- Curve slope and curvature
- Volatilities (if options are present)

**Step 2: Generate Scenarios**

For each simulation path $\omega$ and each future time $t_i$:
1. Simulate market variable evolution: $X^{(\omega)}(t_i)$
2. These may use geometric Brownian motion, short-rate models, or more sophisticated processes

**Step 3: Compute Portfolio Values**

At each future time $t_i$ on each path $\omega$:
1. Price all trades in the netting set using the simulated market state
2. Sum to get $V^{(\omega)}(t_i)$

**Step 4: Apply Collateral Model**

If the portfolio is collateralized:
1. Compute collateral $C^{(\omega)}(t_i)$ based on the CSA rules
2. Account for MPOR: $C^{(\omega)}(t_i)$ reflects $V^{(\omega)}(t_i - c)$, not $V^{(\omega)}(t_i)$

**Step 5: Compute Exposure**

$$E^{(\omega)}(t_i) = \max(V^{(\omega)}(t_i) - C^{(\omega)}(t_i), 0)$$

**Step 6: Aggregate Across Paths**

$$EE(t_i) = \frac{1}{N} \sum_{\omega=1}^{N} E^{(\omega)}(t_i)$$

$$PFE_{0.975}(t_i) = \text{97.5th percentile of } \\{E^{(\omega)}(t_i)\\}_{\omega=1}^{N}$$

### 32.7.2 Time Grid Selection

The choice of time grid $\\{t_1, t_2, \ldots, t_n\\}$ affects accuracy and computation:

- **Finer at the front end:** Exposure changes rapidly in early periods; use daily or weekly steps for the first year
- **Coarser at the back end:** For long-dated portfolios, quarterly or semi-annual steps may suffice
- **Align with cashflow dates:** Include coupon and reset dates to capture discrete jumps

> **Desk Reality:** Exposure engines are computationally heavy for large netting sets, so teams often reuse stored scenarios and/or use approximations to get fast “what-if” answers.
> **Common break:** Comparing exposure numbers across systems that use different (i) time grids, (ii) collateral timing/MPOR assumptions, or (iii) valuation models.
> **What to check:** Netting-set definition, collateral rules (threshold/MTA/lag/MPOR), and whether your engine is doing full revaluation or a proxy.

### 32.7.3 Exposure Sensitivities (Preview)

Exposure metrics are functions of $V(t)$, which in turn depends on risk factors (rates, FX, spreads, vol). A practical way to understand “what drives” exposure is **bump-and-recompute**:

- **Bump object:** the market input you perturb (e.g., a parallel 1bp shift to the discount zero curve used for valuation).
- **Bump size:** $1\text{bp}=10^{-4}$.
- **Units:** currency per 1bp (state whether “per 100 notional” or “per USD 1 notional”).
- **Sign convention (book-wide):** $DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$. For a plain long fixed-income position, rates down $\Rightarrow PV$ up $\Rightarrow DV01\gt 0$.

To get an “exposure DV01,” you would bump the curve, revalue $V^{(\omega)}(t_i)$ on each path, recompute $E^{(\omega)}(t_i)$, then re-aggregate into bumped $EE(t_i)$ or $PFE_q(t_i)$. This is conceptually simple but computationally expensive, so many implementations use proxies.

**Expand (the $\max$ makes exposure “option-like”):** Because $E=\max(V-C,0)$ is a positive-part function, exposure is *one-sided*. On a path where $V^{(\omega)}(t_i)-C^{(\omega)}(t_i)$ is comfortably negative, $E^{(\omega)}(t_i)=0$ and a small risk-factor bump often leaves exposure unchanged (pathwise exposure sensitivity $\approx 0$). Near the boundary $V\approx C$, small bumps can switch exposure on/off, so finite-difference bump size and central-difference schemes matter.

**Check (two-path toy):** Suppose at a future date $t$ there are two equally likely states, and on both states the *portfolio PV* has $DV01=+USD 100\text{k}$ per 1bp (rates down $\Rightarrow V$ up by $0.10\text{mm}$).
- State A: $V-C=+USD 1.00\text{mm}\Rightarrow E=1.00\text{mm}$. After a 1bp rates-down bump, $E$ increases to $1.10\text{mm}$.
- State B: $V-C=-USD 1.00\text{mm}\Rightarrow E=0$. After the same bump, $V-C=-0.90\text{mm}\Rightarrow E$ stays 0.
So the expected-exposure DV01 is about $0.5\times USD 100\text{k}=USD 50\text{k}$ per 1bp, not USD 100k: the negative-exposure state contributes no exposure sensitivity.

### 32.7.4 The Exposure Cone

The distribution of exposure over time can be visualized as an **exposure cone**: a fan of percentile bands showing how exposure uncertainty grows (and eventually shrinks) over the portfolio's life.

For a typical interest rate swap:
- At $t = 0$: Exposure is zero (or small) since the swap is at-the-money
- At intermediate times: Exposure distribution widens as rates can move substantially
- Near maturity: Exposure shrinks because few payments remain

The 97.5% PFE line forms the "upper boundary" of the cone; the EE is the center of mass.

---

## 32.8 Exposure Profiles for Different Instruments

### 32.8.1 Interest Rate Swaps vs. Currency Swaps

Exposure profiles depend strongly on contract structure.

For many **plain interest rate swaps**, expected exposure tends to be **hump-shaped**: early on there are many remaining cashflows, so MTM can move meaningfully; near maturity there is little left to exchange, so exposure shrinks.

For **currency swaps / cross-currency trades** with a large notional exchange at maturity, expected exposure can be **ramp-shaped**: the uncertainty in the terminal exchange-rate-driven cashflow grows as maturity approaches.

> **Visual: The Exposure Shape**
>
> *   **Interest Rate Swap ("The Hump")**: Starts low (par), grows as rates move, then falls to zero as payments are made and time runs out. Peak risk is in the middle.
> *   **FX Swap / Cross-Currency ("The Ramp")**: Starts low, but grows steadily because the huge principal exchange at the end gets riskier and riskier as FX drifts further from the strike. Peak risk is at the very end.

### 32.8.2 Forward Contracts

For an uncollateralized forward to **buy** an asset at price $K$ at time $T$, the value at time $t$ can be written (in simple settings) as:
$$
V(t) = (F_t - K)\\,D(t,T),
$$
so the exposure is:
$$
E(t)=\max(V(t),0)=D(t,T)\\,\max(F_t-K,0).
$$
This makes the intuition clear: the forward’s positive exposure behaves like a **call option** on the forward price $F_t$.

### 32.8.3 Options: Bounded and Asymmetric Exposure

Options have distinct exposure profiles depending on whether you bought or sold them.

**Bought options (long):** With our sign convention, a long option’s MTM $V(t)$ is typically non-negative, so the exposure is essentially the option’s market value (net of any collateral).

**Sold options (short):** A short option is typically negative MTM for the writer, so the writer’s exposure $E(t)=\max(V(t)-C(t),0)$ is usually zero (but the **counterparty** has exposure to the writer).

---

## 32.9 Wrong-Way Risk (Preview)

### 32.9.1 Definition and Intuition

**Anchor (definition):** Wrong-way risk (WWR) is the risk that **exposure to a counterparty is adversely correlated with the counterparty’s credit quality**.

**Expand (intuition):** If the counterparty is most likely to default exactly in the scenarios where $V(t)$ is large and positive, then assuming independence between exposure and default can materially understate expected loss. It is the “double hit”: you are most owed when they are weakest.

**Check (two-state toy):** If “good” states have small exposure and “bad credit” states have large exposure, then $\mathbb{E}[E]$ can look modest while $E(\tau)$ (exposure conditional on default) is large. That gap is the practical signature of WWR.

### 32.9.2 Examples of Wrong-Way Risk

**Speculating counterparty:** If a counterparty is effectively “betting the farm” in one direction, adverse market moves both (i) increase your exposure and (ii) weaken their balance sheet.

**CDS seller / contagion:** If a counterparty sells credit protection, the position becomes very valuable to the buyer precisely in the scenarios where the seller may be stressed, creating WWR.

> **Classic Example: The Put Option on the Bank**
>
> If you buy a put option on Bank A's stock *from* Bank A...
> *   **The Payoff**: You make money if Bank A's stock crashes.
> *   **The Catch**: If Bank A's stock crashes, Bank A might be bankrupt.
> *   **Result**: You are "winning" exactly when the person paying you is "dying." Your expected recovery is terrible. This is extreme wrong-way risk.

### 32.9.3 Right-Way Risk

The opposite is **right-way risk**: exposure and default probability are negatively correlated.

Heuristically, a client using derivatives to hedge an underlying business exposure can create right-way risk: when the derivative moves against the client, the underlying exposure may be helping them, so their credit quality may be better in those states.

### 32.9.4 Modeling Wrong-Way Risk

At a high level, there are two approaches:
1. **Add-ons / stress-style mitigations:** Inflate exposure in “bad credit” states (or apply conservative multipliers) to reduce underestimation from an independence assumption.
2. **Joint modeling:** Specify dependence between market risk factors (driving $V(t)$) and counterparty credit (driving default probability), so exposure and credit quality co-move explicitly.

> **Scope note:** Quantitative modeling of wrong-way risk requires specifying the dependence between market variables and default probability. This is a separate (and difficult) modeling problem, treated in the XVA chapters. Here we only preview the concept.

---

## 32.10 Incremental Exposure and CVA

### 32.10.1 Why Incremental CVA Matters

When a trader considers adding a new trade to an existing portfolio with a counterparty, the key question is: *How does this trade affect the CVA we charge?*

The answer is **not** simply the standalone CVA of the new trade. Because of netting, the new trade interacts with existing trades:
- If the new trade has *opposite* exposure to the existing portfolio, it provides a netting benefit and *reduces* total exposure
- If the new trade has *similar* exposure to the existing portfolio, it adds to exposure

### 32.10.2 Incremental vs. Standalone

The incremental effect depends on how the new trade co-moves with the existing portfolio under the netting agreement:
- If it is positively correlated with existing exposure, incremental CVA tends to be positive.
- If it is negatively correlated (an “offsetting” trade), incremental CVA can be negative.

$$\boxed{\Delta CVA = CVA_{\text{portfolio + new}} - CVA_{\text{portfolio}}}$$

This **incremental CVA** can be negative—meaning the new trade *reduces* the credit charge because of netting benefits.

### 32.10.3 Efficient Computation

Recomputing a full CVA run for every proposed trade is often computationally expensive.

A common approach is to **reuse stored scenarios** from a base run. When a new trade is proposed:
1. Price the new trade on each stored path at each time step
2. Add to the portfolio values already stored
3. Recompute exposure on each path with the combined portfolio
4. Calculate the new CVA and compare

This "incremental CVA engine" allows traders to get real-time quotes on the credit charge for new trades.

### 32.10.4 Pricing Implications

> **Desk Reality: Pricing a New Trade**
>
> When a salesperson asks for a price on a new trade with an existing counterparty:
>
> 1. **Compute standalone price** (as if no credit risk)
> 2. **Query incremental CVA** from the CVA desk
> 3. **Adjust bid/offer** to include CVA
>
> If the new trade *reduces* portfolio exposure (e.g., an offsetting swap), the incremental CVA may be negative—the credit desk effectively "pays" the trading desk for the netting benefit. This can make otherwise marginal trades attractive.
>
> If the new trade *increases* exposure (e.g., same-direction risk), the CVA charge widens the bid-offer, making the trade less competitive.

---

## 32.11 CVA Preview: How Exposure Metrics Feed Into Valuation

This chapter focuses on exposure primitives. The full treatment of CVA, DVA, and other valuation adjustments belongs in **Chapter 34 (XVA Overview)**. The connection between collateral and discount curves is developed in **Chapter 33 (Collateral Discounting and OIS)**. But it is useful here to see how exposure connects to valuation.

A common discretized CVA structure is:

$$\boxed{CVA = \sum_{i=1}^{n}(1-R) q_i v_i}$$

where:
- $q_i$ = risk-neutral probability of default during interval $i$ (from credit spreads)
- $v_i$ = present value of expected net exposure at the midpoint of interval $i$
- $R$ = recovery rate

The exposure work in this chapter is about computing the $v_i$. The credit work (survival probabilities, hazard rates) is about computing the $q_i$. CVA combines them.

**Preview approximation:** If we approximate $v_i \approx D(0, t_i) \cdot EE(t_i)$ and $q_i \approx \Delta PD(t_i)$, then:

$$CVA \approx (1-R) \sum_i D(0, t_i) \cdot EE(t_i) \cdot \Delta PD(t_i)$$

This is a desk-friendly approximation of the generic $(1-R)\\,q_i\\,v_i$ structure.

**Check (units and limiting cases):**
- Units: $(1-R)$ is unitless, $q_i$ is unitless, and $v_i$ is currency $\Rightarrow$ CVA is currency.
- Limits: if $R=1$ (full recovery) or $q_i=0$ (no default risk), CVA is 0. If $EE(t)=0$ for all $t$, CVA is 0.
- Scaling: for a roughly linear portfolio, doubling notional roughly doubles $EE(t)$ and therefore doubles this CVA approximation.

---

## 32.12 Worked Examples

The following examples build intuition for how exposure, netting, and collateral interact. All use the bank perspective with $E = \max(V - C, 0)$.

### Example A: Single Trade Exposure Distribution

**Setup:** One derivative with future MTM at $t = 1$ year:
- $V = +5$ with probability $0.5$
- $V = -5$ with probability $0.5$

No collateral.

**Step 1: Compute exposure outcomes**
- If $V = +5$: $E = \max(5, 0) = 5$
- If $V = -5$: $E = \max(-5, 0) = 0$

Exposure is $\\{5, 0\\}$ with probabilities $\\{0.5, 0.5\\}$.

**Step 2: Expected exposure**

$$EE(1y) = 0.5 \times 5 + 0.5 \times 0 = 2.5$$

**Step 3: PFE at 95%**

CDF: $P(E \leq 0) = 0.5$, $P(E \leq 5) = 1.0$.

The 95% quantile is 5, so $PFE_{0.95}(1y) = 5$.

**Interpretation:** Expected exposure is 2.5, but the 95% worst case is 5—twice as high. PFE captures tail risk that EE averages away.

---

### Example B: Current Exposure vs. Future PFE

**Setup:** Today $t = 0$, MTM is $V(0) = +1$ (no collateral). At $t = 1y$, distribution is as in Example A.

**Current exposure (today):**
$$E(0) = \max(1, 0) = 1$$

**Future PFE (1 year):** From Example A, $PFE_{0.95}(1y) = 5$.

**Interpretation:** Current exposure is small (1), but the 95% "how bad can it get in a year?" measure is 5. This illustrates why risk teams monitor both current exposure and forward-looking tail measures like PFE.

---

### Example C: Netting Benefit

**Setup:** Two trades at $t = 1y$ with perfectly negative dependence:
- **Scenario S1** (prob 0.5): $V_1 = +10$, $V_2 = -8$
- **Scenario S2** (prob 0.5): $V_1 = -10$, $V_2 = +8$

**Without netting (trade-by-trade):**

$$E_{\text{no net}} = \max(V_1, 0) + \max(V_2, 0)$$

- S1: $10 + 0 = 10$
- S2: $0 + 8 = 8$

$EE_{\text{no net}} = 0.5 \times 10 + 0.5 \times 8 = 9$

$PFE_{0.95} = 10$

**With netting:**

Net MTM $V_{\text{net}} = V_1 + V_2$

- S1: $V_{\text{net}} = 2 \Rightarrow E_{\text{net}} = 2$
- S2: $V_{\text{net}} = -2 \Rightarrow E_{\text{net}} = 0$

$EE_{\text{net}} = 0.5 \times 2 + 0.5 \times 0 = 1$

$PFE_{0.95} = 2$

**Netting benefit:** EE drops from 9 to 1 (89% reduction). PFE drops from 10 to 2 (80% reduction). The offsetting positions almost fully hedge each other from a credit exposure perspective.

---

### Example D: Idealized Zero-Threshold VM

**Setup:** Same as Example A, but assume two-way zero-threshold VM with no lag—collateral always equals $\max(V, 0)$.

At $t = 1y$:
- If $V = +5$: counterparty posts $C = 5$, so $E = \max(5 - 5, 0) = 0$
- If $V = -5$: bank posts 5, so $C = -5$, and $E = \max(-5 - (-5), 0) = 0$

**Result:** Current exposure is eliminated in this idealized setting.

**What remains:** In reality, MPOR makes $C$ stale, and exposure can re-emerge even with "daily VM."

---

### Example E: Threshold and MTA

**Setup:** Same distribution as Example A, but with collateral terms:
- Threshold $H = 2$
- MTA $m = 1$
- Two-way VM, no lag

**Collateral rule:** Call amount $= \max(V - H, 0)$. If call amount $\lt m$, no transfer.

**Case 1: $V = +5$**
- Call amount $= \max(5 - 2, 0) = 3$
- $3 \geq 1 \Rightarrow C = 3$
- Exposure $E = \max(5 - 3, 0) = 2$

**Case 2: $V = -5$**
- Bank exposure is $\max(-5 - C, 0) = 0$ under symmetric posting

**Results:** $E \in \\{2, 0\\}$ with equal probability.

$EE = 0.5 \times 2 + 0.5 \times 0 = 1$

$PFE_{0.95} = 2$

**Interpretation:** The threshold creates a "residual unsecured band" of up to 2, even with VM in place.

---

### Example F: Margin Call Lag (Stale Collateral)

**Setup:** VM is called daily but settles with a 1-day lag. Collateral at day $t$ equals yesterday's MTM if positive.

**3-day MTM path:**
- Day 0: $V_0 = 0$
- Day 1: $V_1 = 4$
- Day 2: $V_2 = 6$
- Day 3: $V_3 = 1$

**Collateral:** $C_t = \max(V_{t-1}, 0)$

**Daily exposure:** $E_t = \max(V_t - C_t, 0)$

| Day | $V_t$ | $C_t$ | $E_t$ |
|-----|-------|-------|-------|
| 1 | 4 | 0 | 4 |
| 2 | 6 | 4 | 2 |
| 3 | 1 | 6 | 0 |

**Timing-gap risk:** The lag produces a peak exposure of 4 on Day 1 despite "daily VM." The faster the MTM moves, the larger the lag-induced exposure.

---

### Example G: MPOR and IM Sizing

**Toy calibration:** Suppose IM is set as the 99% quantile of the positive MTM move over a 10-day MPOR.

**Setup:** Daily MTM change $\Delta V \in \\{+1, -1\\}$ with probability 0.5 each. Over MPOR = 10 days:

$$\Delta V_{10} = \sum_{j=1}^{10} \Delta V_j$$

This is a symmetric random walk. Define $\Delta V_{10}^+ = \max(\Delta V_{10}, 0)$.

**CDF of $\Delta V_{10}^+$:**
- at 6: 98.93%
- at 8: 99.90%

**99% quantile of $\Delta V_{10}^+$ is 8.**

**IM proxy:** $IM \approx 8$ currency units covers the 99th percentile of 10-day exposure increase.

---

### Example H: Netting + Collateral Combined

**Setup:** 3 trades in one netting set. Two scenarios (each prob 0.5). Collateral terms: threshold $H = 1$, MTA $m = 0.5$, no lag.

**Trade MTMs:**

| Time | Scenario | $V_1$ | $V_2$ | $V_3$ | Net $V$ |
|------|----------|-------|-------|-------|---------|
| $t_1$ | S1 | 3 | -1 | 2 | 4 |
| $t_1$ | S2 | -2 | 1 | -1 | -2 |
| $t_2$ | S1 | 4 | -2 | 1 | 3 |
| $t_2$ | S2 | -3 | 2 | -2 | -3 |
| $t_3$ | S1 | 2 | -1 | -1 | 0 |
| $t_3$ | S2 | 1 | 1 | -1 | 1 |

**Collateral rule:** If $V \gt H$, counterparty posts $C = V - H$.

**Results:**

| Time | Scenario | $V$ | $C$ | $E$ |
|------|----------|-----|-----|-----|
| $t_1$ | S1 | 4 | 3 | 1 |
| $t_1$ | S2 | -2 | -1 | 0 |
| $t_2$ | S1 | 3 | 2 | 1 |
| $t_2$ | S2 | -3 | -2 | 0 |
| $t_3$ | S1 | 0 | 0 | 0 |
| $t_3$ | S2 | 1 | 0 | 1 |

At each time: $EE = 0.5$, $PFE_{0.95} = 1$

**Interpretation:** The threshold creates a floor on exposure at or below $H$, while netting reduces net MTM volatility by offsetting trades.

---

### Example I: Collateral Frequency Comparison

**Setup:** Same MTM path over 10 days. Compare daily VM (immediate) vs. weekly VM (update at days 0, 5, 10). Zero threshold, no IM.

**Daily MTM:** $V = [0, 1, 3, 2, 5, 4, 6, 3, 2, 1, 0]$

**Daily VM (immediate):** $C_t = V_t$ always, so $E_t = 0$ for all $t$.
- Peak exposure: 0
- Average exposure: 0

**Weekly VM:** Collateral updates only at days 0, 5, 10.

Days 1-4: $C = 0$, so $E = [1, 3, 2, 5]$

Day 5: $C = 4$, so $E_5 = 0$

Days 6-10: $C = 4$, so $E = [2, 0, 0, 0, 0]$

- Peak exposure: 5
- Average exposure: $(1+3+2+5+0+2)/10 = 1.3$

**Interpretation:** Less frequent VM increases both peak and average exposure—purely through timing.

---

### Example J: Cross-Netting Set Impact

**Setup:** Two trades with $V_A = +6$ (always) and $V_B = -5$ (always).

**Case 1: One netting set**

$V_{\text{net}} = 6 - 5 = 1 \Rightarrow E = 1$

**Case 2: Two separate netting sets**

$E_1 = \max(6, 0) = 6$

$E_2 = \max(-5, 0) = 0$

Total $E = 6$

**Fragmentation cost:** Exposure increases from 1 to 6 solely due to netting set boundaries. This illustrates why consolidating trades under a single master agreement is valuable for credit risk.

---

### Example K: Incremental Exposure (Netting Benefit)

**Setup:** Existing portfolio has $V_{\text{existing}} = +10$ with certainty. New trade has $V_{\text{new}} = -6$ with certainty (perfectly offsetting direction).

**Before new trade:**
$$E_{\text{before}} = \max(10, 0) = 10$$

**After new trade (with netting):**
$$V_{\text{combined}} = 10 - 6 = 4$$
$$E_{\text{after}} = \max(4, 0) = 4$$

**Incremental exposure:**
$$\Delta E = E_{\text{after}} - E_{\text{before}} = 4 - 10 = -6$$

**Interpretation:** The new trade *reduces* exposure by 6 due to netting. The incremental CVA would be negative—the credit desk "pays" for the benefit.

---

### Example L: CVA Preview Calculation

**Setup:** Discretized CVA with three intervals.
- Times: $t_1 = 1$, $t_2 = 2$, $t_3 = 3$ years
- Recovery $R = 40\\%$, so $(1-R) = 0.6$
- Flat discount rate $r = 2\\%$
- Expected exposures: $EE = [5, 4, 3]$ million
- Default probability increments: $\Delta PD = [1\\%, 1.5\\%, 2\\%]$

**Discount factors:**
- $D_1 = e^{-0.02} = 0.9802$
- $D_2 = e^{-0.04} = 0.9608$
- $D_3 = e^{-0.06} = 0.9418$

**Compute each term $D \cdot EE \cdot \Delta PD$:**
- Term 1: $0.9802 \times 5 \times 0.01 = 0.04901$
- Term 2: $0.9608 \times 4 \times 0.015 = 0.05765$
- Term 3: $0.9418 \times 3 \times 0.02 = 0.05651$

**Sum:** $0.04901 + 0.05765 + 0.05651 = 0.16317$

**CVA:** $0.6 \times 0.16317 = 0.0979$ million = **USD 97,900**

---

### Example M (Template): Stale Collateral Under a Cure Period (Dated Timeline)

**Context**
- We measure close-out exposure for a bilateral netting set under two-way, zero-threshold VM.
- The point is to see *gap exposure* created by MPOR/cure-period timing even when VM is “perfect” in principle.

**Timeline (Make Dates Concrete)**
- Trade date (portfolio already in place): 2026-02-02
- Last margin call date used for close-out collateral (lookback $c=20$ days): 2026-03-02
- Close-out date $\tau$: 2026-03-22

**Inputs**
- Sign convention: $V\gt 0$ means the counterparty owes the bank; $C\gt 0$ means collateral held by the bank.
- Two-way, zero-threshold VM with the modeling convention: collateral at $\tau$ is based on the MTM $c$ days earlier.
- Cure period / MPOR lookback: $c=20$ calendar days.
- Portfolio MTM at close-out: $V(\tau)=+USD 50\text{mm}$.
- Portfolio MTM $c$ days earlier: $V(\tau-c)=+USD 45\text{mm}$.

**Outputs (What You Produce)**
- Close-out exposure $E(\tau)$ in USD mm.
- Interpretation: collateral kept vs. residual unsecured claim.

**Step-by-step**
1. Collateral held at $\tau$ is based on the earlier MTM, so $C(\tau)=45\text{mm}$.
2. Exposure at close-out:
   $$
 E(\tau)=\max(V(\tau)-C(\tau),0)=\max(50-45,0)=\mathrm{USD}\\,5\text{mm}.
 $$

**Cashflows (table)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-03-02 | $+USD 45\text{mm}$ | Counterparty posts VM based on MTM $=45\text{mm}$ |
| 2026-03-22 | Close-out | Bank has net claim $50\text{mm}$; uses $45\text{mm}$ collateral; residual $5\text{mm}$ is unsecured |

**P&L / Risk Interpretation**
- The USD 5mm is **gap exposure** created purely by timing: the MTM moved from 45 to 50 during MPOR while collateral was frozen.
- If the counterparty defaults and recovery on the residual unsecured claim is $R$, the loss on this path is approximately $(1-R)\times USD 5\text{mm}$ (ignoring close-out costs and legal frictions).

**Sanity Checks**
- Units check: $V, C, E$ are currency; the result is in USD mm.
- Limiting case: if $c\to 0$, then $C(\tau)\to V(\tau)$ and $E(\tau)\to 0$.
- Sign check: if $V(\tau)\le 0$, then $E(\tau)=0$ regardless of $C(\tau)$.

**Debug Checklist (When Your Result Looks Wrong)**
- Did you use the *last agreed/settled* collateral date (not merely last valuation run)?
- Are you consistent on signs ($V\gt 0$ asset to bank; $C\gt 0$ collateral held by bank)?
- Did you apply the cure-period/MPOR lookback consistently (use $V(\tau-c)$ when modeling collateral at $\tau$)?

---

## 32.13 Practical Notes

### 32.13.1 Common Pitfalls

**Computing exposure trade-by-trade instead of at netting set level:** This violates netting logic and overstates exposure. Always aggregate within enforceable netting agreements.

**Forgetting timing lags and MPOR:** Collateral is stale by $c$ days. Exposure can appear even under perfect collateralization if MTM moves during the cure period.

**Confusing PFE with EE:** PFE (quantile) answers "how bad?"; EE (mean) answers "on average." Use the right measure for the right purpose—PFE for limits, EE for CVA.

**Sign convention errors:** Always verify that positive $V$ means in-the-money for the bank, and that collateral $C$ follows the correct convention. Sign errors reverse conclusions.

**Ignoring haircuts on securities collateral:** USD 100 of securities collateral is not USD 100 of collateral value. After a 10% haircut, it counts as USD 90 effective.

### 32.13.2 Implementation Considerations

**Time grid choices:** Exposure profiles depend on discretization. Finer grids near the front end (where exposure changes rapidly) may be needed.

**Collateral modeling:** Frequency, thresholds, MTAs, haircuts, and "stale collateral" assumptions can dominate exposure outcomes. Model these carefully.

**Simulation requirements:** Large institutions may have transactions with thousands of counterparties and many trades per counterparty, so exposure/CVA computation can be computationally intensive. Efficient simulation, sensible approximations, and scenario re-use matter.

**CCP vs. bilateral treatment:** Ensure your systems correctly identify which trades are centrally cleared (exposure to CCP) vs. bilateral (exposure to counterparty).

### 32.13.3 Sanity Checks

- **Netting cannot increase exposure:** If $E_{\text{net}} \gt E_{\text{no net}}$ in your model, something is wrong.
- **Tighter collateral reduces exposure:** Lower thresholds, lower MTAs, and more frequent VM should all reduce exposure metrics.
- **PFE ≥ EE** for typical distributions. Equality occurs only for degenerate cases (constant exposure).
- **IRS exposure is hump-shaped; FX swap exposure increases:** If your model shows opposite shapes, investigate.
- **Incremental exposure can be negative:** A trade that offsets existing exposure has negative incremental exposure and CVA.

---

## Summary

Counterparty exposure is the foundation of counterparty credit risk. Before you can price CVA or set credit limits, you must understand how exposure arises from derivatives positions and how it is modified by netting and collateral.

**Key takeaways:**

1. **Exposure identity (no collateral):** $E(t) = \max(V(t), 0)$ where $V$ is net MTM.

2. **Collateralized exposure:** $E = \max(V - C, 0)$ with careful sign conventions.

3. **Netting reduces exposure:** With netting, $E_{\text{net}} = \max(V_{\text{net}}, 0)$ instead of $\sum \max(V_i, 0)$.

4. **VM tracks current MTM** but thresholds and MTAs leave residual exposure.

5. **IM buffers MPOR risk:** Covers exposure change during the cure period.

6. **MPOR makes collateral stale:** Even "perfect" VM leaves exposure from market moves during the 10-20 day cure period.

7. **PFE and EE are different:** PFE is a quantile (tail risk); EE is a mean (for CVA).

8. **ENE feeds DVA:** Expected negative exposure is the counterparty's view, used for DVA.

9. **Exposure profiles differ by product:** Interest rate swaps are often hump-shaped; currency swaps can ramp up toward maturity; long options create one-sided exposure.

10. **Central clearing changes exposure:** Exposure is to the CCP, not the original counterparty; netting is across all trades at that CCP.

11. **Incremental CVA can be negative:** New trades that reduce portfolio exposure provide a netting benefit.

12. **Wrong-way risk is dangerous:** Exposure rising exactly when default probability rises amplifies losses.

13. **CVA uses exposure primitives:** $CVA = \sum (1-R) q_i v_i$ where $v_i$ comes from exposure modeling.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **MTM** $V(t)$ | Net mark-to-market value of netting set | The input to exposure calculation |
| **Exposure** $E(t)$ | $\max(V-C, 0)$—positive part of (MTM minus collateral) | What you lose at counterparty default |
| **CE** | $E(0)=\max(V(0)-C(0),0)$ | Replacement cost *now* |
| **EE** | $\mathbb{E}[E(t)]$ | Core input to expected-loss calculations |
| **PFE** | Quantile of $E(t)$—“how bad can it get?” | Used for credit limits and stress tests |
| **MPFE** | $\max_t PFE_q(t)$ over a horizon | Single “peak” limit number |
| **EPE** | Time-average of EE | Single-number exposure summary |
| **EAD** | $E(\tau)$ at default time $\tau$ | Appears in some capital/EL frameworks |
| **ENE** | $\mathbb{E}[\max(C(t)-V(t),0)]$ | Mirror quantity used in DVA-style adjustments |
| **Netting** | Legal aggregation under master agreement | Reduces exposure via offset |
| **VM** | Collateral tracking current MTM | Reduces current exposure |
| **IM / independent amount** | Buffer for MPOR gap moves | Mitigates exposure between last VM and close-out |
| **MPOR / cure period** | Last good collateral date → close-out | Makes collateral stale |
| **Threshold** | Level below which no VM required | Creates residual unsecured band |
| **MTA** | Minimum transfer amount | Avoids nuisance transfers |
| **Haircut** | Reduction in collateral value for securities | Adjusts effective collateral |
| **CCP** | Central counterparty | Changes exposure counterparty, increases netting |
| **Default fund** | CCP member contribution to mutualize losses | Shared loss after IM exhausted |
| **Rehypothecation** | Reuse of received collateral for other purposes | Creates recovery risk if holder defaults |
| **Downgrade trigger** | CSA clause demanding collateral on rating downgrade | Protection vs. cascade risk |
| **Wrong-way risk** | Exposure ↑ when default probability ↑ | Amplifies losses |
| **Incremental CVA** | CVA change from adding a new trade | Can be negative (netting benefit) |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $V(t)$ | Net MTM of the netting set at time $t$ | currency; $V\gt 0$ is an asset to the bank |
| $C(t)$ | Collateral held by the bank | currency; $C\gt 0$ held by bank, $C\lt 0$ posted by bank |
| $E(t)$ | Exposure | currency; $E(t)=\max(V(t)-C(t),0)$ |
| $CE$ | Current exposure | currency; $CE=E(0)$ |
| $EE(t)$ | Expected exposure | currency |
| $PFE_q(t)$ | Potential future exposure at confidence $q$ | currency |
| $MPFE_q$ | Maximum PFE over a horizon | currency |
| $EPE$ | Time-average of $EE(t)$ | currency |
| $EAD$ | Exposure at default | currency; $E(\tau)$ |
| $ENE(t)$ | Expected negative exposure | currency; $\mathbb{E}[\max(C(t)-V(t),0)]$ |
| $H$ | Threshold | currency |
| $m$ | Minimum transfer amount (MTA) | currency |
| $c$ | Cure period / MPOR lookback | days |
| $D(t,T)$ | Discount factor (preview) | unitless |
| $R$ | Recovery rate | unitless |
| $DV01$ | Rates sensitivity scalar (preview) | currency per 1bp; $PV(\text{rates down }1bp)-PV(\text{base})$ |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does $V(t)$ represent in counterparty exposure? | Net MTM of all trades in a netting set at time $t$. |
| 2 | What is the bank's exposure without collateral? | $E(t) = \max(V(t), 0)$ |
| 3 | What is the bank's exposure with collateral? | $E = \max(V - C, 0)$ |
| 4 | What is netting (contractually)? | A clause treating all covered trades as a single transaction on default. |
| 5 | How do you compute exposure without netting? | $\sum \max(V_i, 0)$ |
| 6 | How do you compute exposure with netting? | $\max(\sum V_i, 0)$ |
| 7 | What is current exposure (CE)? | $CE=E(0)=\max(V(0)-C(0),0)$. |
| 8 | Define EE(t). | $EE(t)=\mathbb{E}[E(t)]$. |
| 9 | Define ENE(t). | $ENE(t)=\mathbb{E}[\max(C(t)-V(t),0)]$. |
| 10 | Define PFE. | $PFE_q(t)$ is the $q$-quantile of $E(t)$. |
| 11 | Define MPFE. | $MPFE_q=\max_t PFE_q(t)$ over a horizon. |
| 12 | What is variation margin (VM)? | Collateral exchanged to track current MTM under the CSA. |
| 13 | What is initial margin (IM) for? | A buffer intended to cover MTM moves during MPOR. |
| 14 | What is MPOR / cure period? | The time window from last good collateral exchange to close-out; makes collateral stale. |
| 15 | Why does MPOR matter? | Because $C(\tau)$ may reflect $V(\tau-c)$, not $V(\tau)$, creating gap exposure. |
| 16 | What is a haircut? | A percentage reduction applied to a security’s market value for collateral purposes. |
| 17 | What do thresholds and MTAs do? | They leave an unsecured band of exposure to reduce operational frictions. |
| 18 | What is rehypothecation vs segregation? | Reuse of collateral vs holding it in a segregated third-party account. |
| 19 | What is wrong-way risk? | Exposure is largest when the counterparty’s credit quality is weakest. |
| 20 | What is the book’s DV01 convention (preview)? | Bump object must be stated; bump size $1$bp; $DV01=PV(\text{rates down }1bp)-PV(\text{base})$. |

---

## Mini Problem Set

1. (Compute) Single trade: $V \in \\{-2, +6\\}$ with probabilities $(0.7, 0.3)$. No collateral. Compute $EE$ and $PFE_{90}$.
2. (Compute) Two trades with values $V_1=+3$, $V_2=-1$. Compute exposure with and without netting.
3. (Compute) Threshold $H=5$: if net $V=7$, what VM is called and what residual exposure remains?
4. (Compute) Cure period $c$: if $V(\tau)=12$ but $V(\tau-c)=9$ under zero-threshold VM, what is $E(\tau)$?
5. (Compute) Securities collateral: the bank owes USD 95 and has posted securities collateral with market value USD 110 and haircut 10%. Compute the bank’s exposure (use the chapter’s sign convention).
6. (Concept) Explain why netting implies $\max(V_1+V_2,0) \le \max(V_1,0)+\max(V_2,0)$.
7. (Desk) Give two operational reasons realized exposure can exceed a “perfect daily VM” model, and one concrete thing you would check first.
8. (Concept) Explain why PFE computation does not require simulating default.
9. (Concept) Give one wrong-way-risk example and explain why an independence assumption can understate losses.
10. (Risk) State the DV01 convention used in this book (bump object, bump size, units, sign) and explain why mismatched bump definitions break hedge ratios.

### Solution Sketches (Selected)
1. $E\in\\{0,6\\}$. $EE=0.3\times 6=1.8$. $PFE_{90}=6$.
2. No netting: $E=3$. With netting: $E=\max(3-1,0)=2$.
4. Under the stale-collateral convention, $C(\tau)\approx 9$, so $E(\tau)=\max(12-9,0)=3$.
5. Effective collateral posted is $110\times(1-0.10)=99$. With $V=-95$ and $C=-99$, $E=\max(-95-(-99),0)=4$.
8. Exposure answers “*if* default happens, what is the loss?” You simulate the portfolio value to get $E(t)$; default probabilities enter later when converting exposure into expected loss (e.g., CVA).
10. Example answer: bump object = parallel 1bp shift to the valuation discount zero curve; bump size $1$bp $=10^{-4}$; units = currency per 1bp; sign $DV01=PV(\text{rates down }1bp)-PV(\text{base})$ (long rates risk $\Rightarrow DV01\gt 0$).

## References

- Hull, *Risk Management and Financial Institutions*, “How Defaults are Handled” and “Collateral and Cure Periods”
- Hull, *Options, Futures, and Other Derivatives*, “Cure period / margin period of risk” and Example 24.4
- Brigo, Morini, and Pallavicini, *Counterparty Credit Risk, Collateral and Funding*, “1.3 Exposure, CE, PFE, EPE, EE, EAD”; “1.19 Re-hypothecation”; “1.20 Netting”
- Crépey and Bielecki, *Counterparty Risk and Funding*, “Introduction”; “3.3 CSA Specifications (Cure Period; Rehypothecation Risk and Segregation)”
- Oosterlee and Grzelak, *Mathematical Modeling and Computation in Finance*, “Types of exposure”
