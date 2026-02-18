# Chapter 34: XVA Overview — CVA, DVA, FVA, and the New Derivatives Valuation Landscape

---

## Introduction

Prerequisites: [Chapter 32 — Counterparty Exposure Basics](chapters/chapter_32_counterparty_exposure_basics.md), [Chapter 33 — Collateral Discounting and OIS](chapters/chapter_33_collateral_discounting_ois.md), [Chapter 36 — Survival Probabilities and Hazard Rates](chapters/chapter_36_survival_probabilities_hazard_rates.md)

Follow-on: [Chapter 35 — Default, Recovery, and Credit Events](chapters/chapter_35_default_recovery_credit_events.md), [Chapter 37 — Cash Credit — Risky Bonds, Credit Spreads, and CS01](chapters/chapter_37_cash_credit_risky_bonds_spreads_cs01.md), [Chapter 38 — CDS Contract Mechanics](chapters/chapter_38_cds_contract_mechanics.md), [Chapter 42 — Bootstrapping a CDS Survival Curve](chapters/chapter_42_bootstrapping_cds_survival_curve.md)

A dealer’s “clean” price for an OTC derivative is the output of a no-default pricing model under a stated discounting convention (often collateral/OIS for fully collateralized trades). In practice, the value used for client pricing, limits, and P&L reserves often includes a stack of adjustments for counterparty credit risk, funding and margin costs, and capital usage. These are valuation adjustments, collectively called **XVAs**.

This chapter is an overview of the XVA stack. The goal is not to settle every theoretical debate or regulatory detail, but to make you fluent in:
- the objects that drive each adjustment (exposure profile, default curve, and desk policy inputs),
- the core present-value equations, and
- the sanity checks that catch the most common implementation breaks.

> **Desk Reality:** Many desks treat XVAs like “manufacturing costs” that must be recovered in the all-in client price.
> **Common break:** Two banks agree on a clean price but quote different all-in prices because they use different netting sets/CSAs, funding assumptions, initial-margin treatment, or capital policies.
> **What to check:** Ask “Is this clean or all-in?”, “What netting set and CSA applies?”, and “Which XVA components are included?”

## Learning Objectives
- Translate a clean PV into an explicit all-in PV using a consistent XVA stack and sign convention.
- Compute a toy CVA from an exposure profile and a default curve, with unit/sign checks.
- Explain why collateral does not eliminate CVA (margin period of risk / cure period).
- Define wrong-way vs right-way risk and the alpha-multiplier shortcut.
- Define CVA01 and state precisely what is being bumped (and what is held fixed).
- Describe what an XVA/CVA desk does (pricing, hedging, and transfer pricing).

---

## 34.1 The Clean Valuation Baseline

Before diving into adjustments, we must establish what we are adjusting *from*. This section defines the no-default value that serves as the baseline for all XVA calculations.

### 34.1.1 No-Default Value

The **no-default value** (also called the **clean value**) of a derivatives portfolio is the value assuming that both sides perform (no default) and ignoring funding/capital frictions. Denote it by $f_{\text{nd}}$. It is the output of standard pricing models (e.g., discounted expected cashflows for swaps, option models for options).

For fully collateralized trades, the discounting rate is typically linked to the collateral remuneration rate (often an overnight/OIS-type rate; see Chapter 33). For uncollateralized trades, funding and credit effects enter and the “clean” value is no longer the full story.

### 34.1.2 The Adjustment Stack

Once we account for real-world frictions, the all-in value becomes:

$$\boxed{V_{\text{all-in}} = f_{\text{nd}} - \text{CVA} + \text{DVA} - \text{FVA} - \text{MVA} - \text{KVA} + \cdots}$$

Each term has economic meaning and a definite sign convention:

| Adjustment | Sign in Formula | Economic Interpretation |
|------------|-----------------|------------------------|
| **CVA** | Subtracted (reduces value) | Cost of counterparty default |
| **DVA** | Added (increases value) | "Benefit" from own potential default |
| **FVA** | Subtracted if net cost | Funding costs net of benefits |
| **MVA** | Subtracted (reduces value) | Cost of funding initial margin |
| **KVA** | Subtracted (reduces value) | Cost of regulatory capital |

In many treatments, **CVA/DVA** are viewed as direct credit-risk adjustments, while **FVA/MVA/KVA** are more sensitive to funding/capital assumptions and therefore more policy-dependent and more debated.

### 34.1.3 The Three Perspectives on Value

Understanding XVA requires distinguishing three different notions of "value":

| Perspective | What It Includes | Who Uses It |
|-------------|------------------|-------------|
| **Mid-Market (No-Default)** | $f_{\text{nd}}$ only | Quant models, academic pricing |
| **Credit-Adjusted Value** | $f_{\text{nd}} - \text{CVA} + \text{DVA}$ | Fair-value style notions that incorporate counterparty + own credit (details vary) |
| **Desk Economics** | $f_{\text{nd}} - \text{CVA} + \text{DVA} - \text{FVA} - \text{MVA} - \text{KVA}$ | P&L, trader compensation, deal approval |

> **Desk Reality: Why Sales and Trading Fight About Value**
>
> Sales: “The clean NPV is positive. Let’s quote close to mid.”
>
> Trader: “After XVA charges, the all-in economics can be much smaller. If you quote clean mid, I may be trading below hurdle.”
>
> This tension is structural: Sales optimizes flow and client relationships; Trading is measured on risk-adjusted returns after internal charges/reserves. The practical resolution is transparency: agree on an XVA methodology and show the client a clean price alongside an all-in price.

---

### 34.1.4 TVA (Advanced): Why the Stack Is Not Unique

We will call **total valuation adjustment (TVA)** the aggregate value of all the adjustments which are required in order to account for bilateral counterparty risk under funding constraints.

Operationally, this corresponds to a common division of labor:
- A product trading desk produces a **clean** price-and-hedge for promised cashflows: a no-default valuation and hedge under the discounting conventions of this book (see Chapter 33 for collateral/OIS discounting), ignoring counterparty credit and institution-specific funding/capital frictions.
- A centralized XVA/TVA function produces a **price-and-hedge adjustment** (and hedges/transfer-prices the resulting risks), often with funding handled in coordination with treasury/ALM.

Proper inclusion of funding costs leads to an implicit (or recursive if you want, as in a fixed-point equation) pricing problem.

When we try to include funding costs consistently with credit and collateral, we can obtain a highly non-linear and recursive pricing equation.

Because one entity cannot know in detail the funding policy of another entity, including funding cannot be bilateral in a valuation procedure.

In plain terms: funding-inclusive “all-in” prices are perspective-dependent, which is one reason the XVA stack is not unique.

**Key technical point (beyond the basics):** once funding, collateral, and default interact, the adjustments can become **non-separable**. A funding cost depends on future collateral and exposure, but collateral and exposure depend on the value process of the trade, which itself depends on funding assumptions. In that case, “clean price + (CVA + DVA + FVA + …)” is best thought of as a *modeling decomposition* of a fixed-point (recursive) valuation problem—not a law of nature.

**Check (limiting cases):**
- If counterparty and own default are ruled out by assumption, CVA and DVA should collapse to 0.
- If funding spreads are set to 0 and collateral is remunerated at the discounting rate, funding-driven adjustments should collapse toward 0, and “all-in” should revert toward “clean.”

## 34.2 CVA: Credit Value Adjustment

### 34.2.1 Economic Intuition

The credit value adjustment (CVA) is the present value of the expected loss to the bank from counterparty default on a derivatives **netting set** (i.e., the enforceable portfolio you close out together).

A simple loss-at-default picture is:
- $V(\tau)$: portfolio value to the bank at default time $\tau$ (positive = you are owed money),
- $C(\tau)$: collateral held at $\tau$,
- exposure $E(\tau) = \max(V(\tau)-C(\tau),0)$,
- loss $\approx (1-R)\\,E(\tau)$, where $R$ is the recovery rate (so $1-R$ is LGD).

CVA prices that loss: it is the discounted expectation of loss-at-default across default times and market scenarios (under the pricing measure).

> **Connection to Chapter 32:** The expected exposure $v_i$ in the CVA formula is exactly the quantity we developed in Chapter 32. The CVA calculation takes the exposure framework from Chapter 32 and prices it using the credit spreads and survival probabilities we develop in Chapter 36 and the CDS chapters (Chapters 38–42).

### 34.2.2 The CVA Formula

Suppose the life of the longest derivative is $T$ years, divided into $n$ intervals. Define:

- $q_i$: risk-neutral probability of default during interval $i$
- $v_i$: present value of the expected **net exposure** (after collateral) at the midpoint of interval $i$, conditional on default
- $R$: recovery rate

Then:

$$\boxed{\text{CVA} = \sum_{i=1}^{n} (1-R) \\, q_i \\, v_i}$$

This formula is simple to write but usually hard to implement, because the $v_i$ require simulating the portfolio forward under market dynamics and collateral rules.

**Intuition:** The formula says: for each possible default interval, multiply:
- Probability of defaulting in that interval ($q_i$)
- Loss given default ($1-R$)
- Expected exposure at that time ($v_i$)

Then sum across all intervals.

**Unit Check:**

| Quantity | Units | Interpretation |
|----------|-------|----------------|
| $(1-R)$ | dimensionless | Loss given default fraction |
| $q_i$ | dimensionless | Probability |
| $v_i$ | currency | PV of expected exposure |
| CVA | currency | Expected loss |

The product $(1-R) q_i v_i$ gives expected loss from defaults in interval $i$; summing over all intervals gives total CVA.

### 34.2.3 Computing Default Probabilities

The $q_i$ are **risk-neutral** default probabilities, typically implied from market credit instruments (e.g., CDS). In reduced-form (intensity) models one works with a hazard rate $h(t)$ and the survival probability.

$$
Q(t)=\exp\Big(-\int_0^t h(u)\\,du\Big).
$$

In a very common *approximation*, if $s_i$ is a credit spread for maturity $t_i$ and recovery is treated as constant $R$, the average hazard rate over $[0,t_i]$ is:

$$
\bar{h}_i \approx \frac{s_i}{1-R}.
$$

Then the approximation is `Q(t_i) \approx exp(-\bar{h}_i t_i)`, and a discrete interval default probability is:

$$
q_i = Q(t_{i-1})-Q(t_i).
$$

**Check (units and quick magnitude):** A spread $s$ and hazard $h$ both have units of “per year.” For example, if $s=200\text{bp}=0.02$ and $R=40\\%$, then $\bar{h}\approx 0.02/0.6\approx 0.0333$ per year. Over 5 years this implies $Q(5)\approx e^{-0.0333\times 5}\approx 0.846$, i.e., a cumulative default probability of about $15.4\\%$. If your curve mapping produces a higher survival probability when spreads go up, something is inverted.

This connects directly to the survival-probability framework in Chapter 36 and the CDS curve bootstrapping mechanics in Chapter 42.

**Why risk-neutral probabilities?** CVA is a *price* (an expectation of discounted cashflows), so it uses risk-neutral default probabilities rather than real-world “physical” default probabilities.

> **Connection to CDS Market:** CVA uses the same survival probabilities as CDS pricing. If you can translate CDS quotes into a survival curve (Chapters 38–42), you have the building blocks for the default-probability inputs in CVA. A bank's CVA desk often calibrates counterparty default probabilities directly from observable CDS spreads using the techniques in Chapter 42.

### 34.2.4 Computing Expected Exposure

The $v_i$ are typically obtained by Monte Carlo simulation of the market factors that drive $V(t)$, combined with a model of collateral and netting. The exposure at time $t$, after netting and collateral, is often represented as:

$$E(t) = \max(V(t) - C(t), 0)$$

where $V(t)$ is the portfolio value and $C(t)$ is collateral held. This connects to the exposure concepts developed in Chapter 32.

**The simulation process:**
1. Generate thousands of paths for all market variables (rates, FX, commodities, etc.)
2. At each time point $t_i^{\ast}$ (midpoint of interval $i$), compute portfolio value $V(t_i^{\ast})$
3. Apply collateral/netting rules to determine $C(t_i^{\ast})$
4. Compute exposure $E(t_i^{\ast}) = \max(V(t_i^{\ast}) - C(t_i^{\ast}), 0)$
5. Average exposures across paths to get expected exposure
6. Discount to get $v_i$

Because dealers face many counterparties and large portfolios, computing $v_i$ at scale can be computationally intensive.

### 34.2.5 Cure Period and Collateral Timing

A critical practical detail is that collateral at default is usually based on a mark-to-market from several days *earlier*. A common modeling convention is: the counterparty stops posting collateral (and stops returning excess collateral) $c$ days before default, where $c$ is often taken to be 10–20 days. The parameter $c$ is the **cure period**, also called the **margin period of risk (MPOR)**.

This creates a fundamental insight: **even perfectly collateralized trades carry residual exposure** equal to the potential MTM change during the cure period (also called the margin period of risk or MPOR).

**Example 34.1 — Cure Period Effect (Toy Illustration)**

Assume a two-way, zero-threshold margining setup and a cure period of 20 days. Consider what happens at default time $\tau$:

*Scenario A:* Portfolio value to bank at $\tau$ = 50. Value 20 days earlier = 45.
- Collateral at default = 45 (reflects old MTM)
- Exposure = $\max(50 - 45, 0) = 5$

*Scenario B:* Portfolio value to bank at $\tau$ = 50. Value 20 days earlier = 55.
- Collateral at default = 55 (more than current value)
- Exposure = $\max(50 - 55, 0) = 0$

*Scenario C:* Portfolio value to bank at $\tau$ = -50. Value 20 days earlier = -55.
- Bank has posted collateral of 55 with counterparty
- If counterparty defaults, only 50 needed but 55 posted
- Exposure = $\max(-50 - (-55), 0) = 5$ (excess collateral not returned)

The cure period creates exposure even under “zero-threshold” collateralization: collateral is based on a stale MTM, so the gap is the MTM move during the MPOR.

**Example 34.2 — Collateral Effect on CVA (Quantitative Illustration)**

To see how dramatically collateral reduces CVA, consider a 3-year portfolio with the following inputs:

| Interval | Midpoint | $\text{EE}$ (uncollat.) | $\text{EE}$ (collat.) | $D(0, t_i^{\ast})$ | $\Delta PD_i$ |
|----------|----------|------------------------|----------------------|----------------|---------------------|
| 1 | 0.5 | \mathrm{USD}\\,10M | \mathrm{USD}\\,2M | 0.99 | 0.0198 |
| 2 | 1.5 | \mathrm{USD}\\,8M | \mathrm{USD}\\,1.5M | 0.97 | 0.0290 |
| 3 | 2.5 | \mathrm{USD}\\,6M | \mathrm{USD}\\,1M | 0.95 | 0.0373 |

Recovery $R=40\\%$ (LGD $=0.6$). Default probabilities are derived from a simple piecewise-constant hazard-rate setup (toy numbers).

**Uncollateralized CVA:**
$$\text{CVA} = 0.6 \times (0.99 \times 10M \times 0.0198 + 0.97 \times 8M \times 0.0290 + 0.95 \times 6M \times 0.0373)$$
$$= 0.6 \times (196{,}020 + 225{,}040 + 212{,}610) = 0.6 \times 633{,}670 = \mathrm{USD}\\,380{,}202$$

**Collateralized CVA:**
$$CVA_c = 0.6 \times (0.99 \times 2M \times 0.0198 + 0.97 \times 1.5M \times 0.0290 + 0.95 \times 1M \times 0.0373)$$
$$= 0.6 \times (39{,}204 + 42{,}195 + 35{,}435) = 0.6 \times 116{,}834 = \mathrm{USD}\\,70{,}100$$

**Result:** Collateral reduces CVA by **81.6%** (USD 380K to USD 70K). The residual \mathrm{USD}\\,70K reflects cure-period exposure — even "perfect" collateralization leaves a gap.

**Sanity Check:** Better collateralization → lower CVA ✓. The residual amount reflects the margin period of risk, not a modeling error.

### 34.2.6 CVA After Accounting for Defaults

Taking CVA into account, the value of the derivatives portfolio to the bank becomes:

$$f_{\text{nd}} - \text{CVA}$$

This is a **unilateral adjustment**—it reflects only the counterparty's default risk, not the bank's own default risk. As we will see, the full picture requires DVA as well.

---

## 34.3 Worked Example: Toy CVA for a Swap Netting Set

This example follows the “exposure grid + default curve” pipeline: **exposure profile** → **default probabilities** → **PV of expected loss**.

**Example Title:** Toy CVA for a 5Y uncollateralized swap (netting set)

**Context**
- A bank has an uncollateralized vanilla interest rate swap with a corporate counterparty.
- We want an order-of-magnitude CVA using a simple exposure grid and a flat credit spread.

**Timeline (Make Dates Concrete)**
- Valuation date: 2026-02-15 (hypothetical)
- Trade date: 2026-02-15
- Effective date (settlement): 2026-02-17
- Maturity date: 2031-02-17
- Exposure evaluation grid: midpoints of annual intervals $t_i^{\ast} \in \\{0.5,1.5,2.5,3.5,4.5\\}$ years, i.e. approximately:
  - 2026-08-15, 2027-08-15, 2028-08-15, 2029-08-15, 2030-08-15

**Inputs**
- Recovery $R=40\\%$ (LGD $=0.6$)
- Counterparty credit spread (toy, flat): $s=200\text{ bp}=0.02$ per year
- Expected exposure at each midpoint (after netting/collateral), in \mathrm{USD}\\, millions:

| $t_i^{\ast}$ (years) | $EE_i$ (\mathrm{USD}\\,mm) |
|---:|---:|
| 0.5 | 1.5 |
| 1.5 | 2.5 |
| 2.5 | 2.8 |
| 3.5 | 2.2 |
| 4.5 | 1.5 |

- Discount factors (toy OIS curve):
  - $P(0,0.5)=0.98$, $P(0,1.5)=0.94$, $P(0,2.5)=0.90$, $P(0,3.5)=0.87$, $P(0,4.5)=0.84$

**Outputs (What You Produce)**
- CVA (USD)

**Step-by-step**
1. Convert spread to a flat hazard rate (approx.): $\bar{h} \approx \frac{s}{1-R}=\frac{0.02}{0.60}=0.03333$ per year.
2. Compute annual survival probabilities $Q(t)=e^{-\bar{h}t}$ at $t=1,2,3,4,5$:
   - $Q(1)=0.9672$, $Q(2)=0.9355$, $Q(3)=0.9048$, $Q(4)=0.8752$, $Q(5)=0.8465$
3. Convert to annual default probabilities $q_i = Q(i-1)-Q(i)$:
   - $q_1=0.0328$, $q_2=0.0317$, $q_3=0.0307$, $q_4=0.0296$, $q_5=0.0287$
4. Discount expected exposures $v_i = EE_i \cdot P(0,t_i^{\ast})$ (in \mathrm{USD}\\,mm):
   - $v=[1.47,\\,2.35,\\,2.52,\\,1.91,\\,1.26]$
5. Assemble CVA: $\text{CVA} = (1-R)\sum_{i=1}^5 q_i v_i =0.60\times 0.2929\text{ mm} \approx \mathrm{USD}\\,175{,}700.$

**Cashflows (expected loss contributions)**
Think of each interval as contributing a PV “expected loss cashflow” $(1-R) q_i v_i$.

| Approx date | PV expected loss | Explanation |
|---|---:|---|
| 2026-08-15 | \mathrm{USD}\\,28.9k | $0.6\times q_1\times v_1$ |
| 2027-08-15 | \mathrm{USD}\\,44.7k | $0.6\times q_2\times v_2$ |
| 2028-08-15 | \mathrm{USD}\\,46.4k | $0.6\times q_3\times v_3$ |
| 2029-08-15 | \mathrm{USD}\\,33.9k | $0.6\times q_4\times v_4$ |
| 2030-08-15 | \mathrm{USD}\\,21.7k | $0.6\times q_5\times v_5$ |

**P&L / Risk Interpretation**
- CVA is a cost/reserve: in a “clean minus CVA” convention, larger CVA reduces the value to the bank.
- CVA moves for two big reasons:
  1. **Credit**: spreads widen/narrow (changes $q_i$).
  2. **Exposure**: rates/FX/etc move (changes the future $EE_i$ profile).

**Sanity Checks**
- Units check: $q_i$ and LGD are unitless; $v_i$ is currency → CVA is currency.
- Sign check: higher spreads (larger $q_i$) or higher exposure (larger $v_i$) should increase CVA.
- Limit check: if $R\to 1$ (LGD→0) then CVA→0.

**Debug Checklist (When Your Result Looks Wrong)**
- Did you discount $EE_i$ twice (or not at all)? Be explicit about whether $EE_i$ is a forward exposure or a PV exposure.
- Are your $q_i$ consistent with your spread-to-hazard convention and recovery assumption?
- Are exposures computed at the **netting-set** level, after collateral and netting?
- Are you mixing year-end $q_i$ with midpoint $v_i$ without being clear about the approximation?

---

## 34.4 DVA: Debit Value Adjustment

### 34.4.1 The Controversial "Gain from Default"

The debit (or debt) value adjustment (DVA) is the mirror image of CVA: it accounts for the possibility that **the bank itself** might default. From the counterparty’s perspective, this is a cost (it is *their* CVA). From the bank’s perspective, it is recorded as a benefit (under the sign convention in this chapter).

The formula is symmetric to CVA:

$$\boxed{\text{DVA} = \sum_{i=1}^{n} (1-R^*) \\, q_i^* \\, v_i^*}$$

where:
- $q_i^*$: (risk-neutral) probability of default by the **bank** during interval $i$
- $v_i^*$: PV of the counterparty’s expected exposure to the bank at the midpoint of interval $i$
- $R^*$: recovery rate in the event the bank defaults

**Expand (link to negative exposure):** From the bank’s perspective, CVA is driven by scenarios where we are owed money ($V-C\gt 0$). DVA is driven by the mirror scenarios where we owe the counterparty ($V-C\lt 0$): it is closely related to the **expected negative exposure** (ENE) object from Chapter 32, just viewed through the counterparty’s eyes.

**Check (limiting cases):**
- If a portfolio is always an asset to the bank (netted and collateralized $V(t)-C(t)\ge 0$ on all relevant scenarios), then the counterparty has no claim on us and DVA should be near 0.
- If a portfolio is always a liability to the bank ($V(t)-C(t)\le 0$), then CVA should be near 0 while DVA can be material.

### 34.4.2 Why DVA Exists: The Zero-Sum Argument

The logic for DVA rests on a basic symmetry: ignoring default, derivatives are (approximately) zero-sum. If the counterparty is worse off because the bank might fail to pay when it owes money, then the bank (or its creditors) must be better off in expectation.

Without DVA, two parties can easily “talk past each other” in pricing. For example:

Consider two parties X and Y negotiating a swap with no-default fair rate of 2.2%.

- X calculates: "Accounting for Y's default risk, I should pay only 2.1%"
- Y calculates: "Accounting for X's default risk, I should receive 2.35%"

If each side considers only the *other's* default risk, there is no overlap—no deal gets done. DVA helps both sides value expected cashflows similarly, enabling trade.

### 34.4.3 Bilateral CVA

Taking both CVA and DVA into account, the value to the bank is:

$$\boxed{V = f_{\text{nd}} - \text{CVA} + \text{DVA}}$$

This is the **bilateral CVA** framework:
- CVA reduces value (cost of counterparty default)
- DVA increases value (benefit from potential own default)

Many fair-value frameworks incorporate both counterparty credit and own credit. On desks, DVA is often treated as part of a consistent bilateral view (even when some downstream reporting views differ).

### 34.4.4 Closeout Conventions: Risk-Free vs. Replacement (Advanced)

Closeout is what happens when the first of the two parties in the deal defaults. The closeout procedure fixes the residual value of the contract to us and establishes how much of that is going to be paid to us. If, however, it is negative then we will have to pay the whole amount to the counterparty.

Here “us” simply means the surviving / non-defaulting party: closeout is a convention, but you still need to keep track of which side you are valuing from.

A key modeling choice is what valuation is used to compute that residual value at the default time. At the default time of the counterparty, do we value the remaining contract by taking into account our own residual credit risk (replacement closeout) or just by using a default risk-free valuation (risk-free closeout)?

- **Replacement closeout:** value the remaining contract taking into account the surviving party’s own residual credit risk (often described as including a unilateral DVA for the surviving party). The motivation is continuity: if you included DVA before default, you do not stop including it at default.
- **Risk-free closeout:** value the remaining contract using a default-risk-free valuation for the surviving party (ignore the surviving party’s residual credit risk). The motivation is liquidation: you are closing the position now, so you might not want to account for future default of the surviving party.

This is not just legal plumbing: it changes the payoff at default, so it changes today’s value and the meaning of “DVA” in your system.

**Check:** if your pre-default valuation includes DVA but your closeout convention is risk-free, the value process can jump at the default time. Be explicit and keep the convention consistent with how you define bilateral valuation.

### 34.4.5 The Accounting Paradox and Phantom Equity

DVA can create a counterintuitive accounting effect: when a bank’s credit spreads widen (its default probability rises), DVA increases. Under a $V=f_{\text{nd}}-\text{CVA}+\text{DVA}$ convention, the derivatives portfolio can therefore look *more valuable* to the bank.

> **Desk Reality: DVA as "Phantom Equity"**
>
> DVA gains can be “phantom” in the sense that they arise from *your own* worsening credit (a higher chance you don’t pay your liabilities in full).
>
> Regulatory capital definitions typically **do not give credit** for retained-earnings changes driven by a bank’s own credit risk. So an accounting gain from DVA does not automatically translate into more loss-absorbing regulatory capital.

**Example 34.4 — DVA Gain from Spread Widening (Toy)**

A bank has \mathrm{USD}\\,500 million notional of swaps where it is out-of-the-money to counterparties (negative MTM to bank). The bank's 5-year CDS spread widens from 80bp to 180bp. Recovery assumption is 40%.

**Before:** 5-year survival probability = $e^{-0.008 \times 5 / 0.6} = 0.9355$
**After:** 5-year survival probability = $e^{-0.018 \times 5 / 0.6} = 0.8607$

Change in default probability = $0.9355 - 0.8607 = 0.0748$ (7.48%)

**Approximate DVA change:**
$$\Delta \text{DVA} \approx (1-R) \times \Delta(\text{default prob}) \times \text{Average negative exposure}$$

If average negative exposure (counterparty's claim) is \mathrm{USD}\\,25 million:
$$\Delta \text{DVA} \approx 0.6 \times 0.0748 \times 25 = \mathrm{USD}\\,1.12 \text{ million profit}$$

**Interpretation:** The bank reports \mathrm{USD}\\,1.12 million profit because its liabilities became less valuable. But this profit:
- Cannot be distributed as dividends
- Is not counted in regulatory capital
- Would only "realize" if the bank actually defaults

---

## 34.5 Interest Rate Swaps vs. Currency Swaps: Exposure Profiles

### 34.5.1 The Shape of Exposure Over Time

Consider a dealer with matched pairs of offsetting swaps—one with each of two counterparties. The **expected exposure profiles** differ strikingly between interest rate swaps and currency swaps:

**Interest Rate Swaps:** Expected exposure starts at zero, increases to a peak around the middle of the swap's life, then decreases back toward zero as maturity approaches. The "hump-shaped" profile arises because:
- At inception, the swap is at-market (zero value)
- As time passes, rate movements create potential value divergence
- Near maturity, very little remains to be exchanged, so exposure diminishes

**Currency Swaps:** Expected exposure can increase with time because principals are exchanged at maturity. The PV of that final principal exchange depends on the future FX rate, and FX uncertainty generally grows with horizon. By contrast, a plain interest rate swap does not exchange notional, so there is “less left to exchange” near maturity.

### 34.5.2 Implications for CVA

If two trades face the same counterparty, the $q_i$ are (by definition) counterparty-specific. Differences in CVA across products are therefore largely driven by the exposure profile $v_i$.

This explains why:
- currency swaps can have larger CVA than interest rate swaps of comparable maturity/notional (because of larger exposure),
- principal-exchange features (and FX volatility) tend to matter a lot for exposure,
- MPOR/cure-period modeling matters whenever the underlying risk factor can move significantly during the margin period.

---

## 34.6 FVA: Funding Value Adjustment

### 34.6.1 The Funding Problem

Consider a dealer who enters an interest rate swap with a corporate end user and hedges with another bank through a CCP:

```
End User ←→ Bank A ←→ Bank B (via CCP)
    (bilateral)     (cleared)
```

The swap with the end user is uncollateralized. The hedge with Bank B is cleared and therefore requires margining. When the hedge moves against Bank A, Bank A posts variation margin (and may post initial margin) to the CCP, but receives no corresponding collateral from the end user. This creates a **funding need**.

The **funding valuation adjustment (FVA)** captures the cost of funding derivative positions when there is asymmetric collateralization.

### 34.6.2 FCA and FBA

It is common to decompose funding effects into two components:
- **FCA** (Funding Cost Adjustment): present value of expected future funding costs
- **FBA** (Funding Benefit Adjustment): present value of expected future funding benefits

$$\boxed{\text{FVA} = \text{FCA} - \text{FBA}}$$

When an uncollateralized derivative has positive value, there is a funding cost (the bank has an asset it cannot finance via collateral). When it has negative value, there is a funding benefit (the bank receives an implicit loan from the counterparty).

### 34.6.3 The Theory vs. Practice Debate

FVA is one of the places where “what is the right fair value?” and “what does the bank charge internally?” can diverge.

- **A common desk implementation** uses an **average** funding spread (how Treasury funds the bank) to value the cost of funding margin and uncollateralized exposure.
- **A common finance-theory objection** is a **marginal** argument: the required return should reflect the risk of the incremental project, and low-risk collateral/margin should not be charged at the bank’s average funding cost.

You will see both viewpoints in practice. For this book, treat FVA as **policy-dependent**: it exists operationally as a charge, but the “correct” theoretical definition depends on the funding assumptions you choose to bake into the valuation.

**Beyond the basics: recursion and overlap.** Once you decide how collateral is treated for funding purposes (e.g., whether received collateral can be re-used / rehypothecated), funding effects can become *recursive*: the funding requirement depends on the adjusted value of the portfolio, but the adjusted value depends on the funding policy. In that setting, a clean “add-on” FVA can be a decomposition of a fixed-point problem rather than an independent term you can compute in isolation.

Moreover, there is some overlap (or double counting) between **DVA and FVA**. If your funding policy already embeds an own-credit benefit (or if closeout/funding terms depend on own default), and you also add a separate DVA term, you can end up double counting.

> **Pitfall — DVA/FVA overlap (double counting):** mixing “funding benefit” ideas with a separate DVA term without a consistent definition of what each term includes.
> **Why it matters:** you can get a plausible-looking all-in number but the wrong sensitivities and the wrong P&L attribution (credit vs funding).
> **Quick check:** set funding spreads to 0 and confirm your funding-driven adjustment collapses toward 0; then check that your sign convention reproduces $V_{\text{all-in}}$ from the stated stack.

### 34.6.4 The FVA Asymmetry in Practice

> **Desk Reality: The FVA Screaming Match**
>
> Sales: "Client wants to trade at mid-market."
>
> Trader: "Treasury charges me SOFR + 50bp to fund this! Who pays for that?"
>
> **Policy asymmetry:** Some organizations recognize FCA more readily than FBA, or apply different spreads to the two legs. The result is that “FVA” is not a single market convention; it is part of a bank’s transfer-pricing and funding policy.
>
> Example (hypothetical): if Treasury funding is SOFR + 50bp and a trade requires \mathrm{USD}\\,10M of funding for 5 years, that’s on the order of \mathrm{USD}\\,250k of carry cost from funding spread alone (\mathrm{USD}\\,10M × 0.50% × 5), before discounting and netting effects.
>
> When the trade goes the other way (the bank has a net liability and therefore an “implicit funding benefit”), whether that benefit is passed through to the client is policy- and negotiation-dependent. In practice, institutions often apply asymmetric internal transfer-pricing rules, so “FVA” is not automatically symmetric.
>
> **Why this matters for pricing:** Clients dealing with multiple banks can see different all-in prices because each bank can use different netting sets/CSAs, funding spreads, and FVA methodologies—even if their clean models agree.

### 34.6.5 Worked Example: FVA from Expected Funding Requirement

> **Implementation note (policy-dependent):** The sources define FCA/FBA conceptually but do not give a single canonical discrete-sum formula for FVA in terms of a desk-ready “funding requirement profile.” The right implementation depends on your funding policy (what constitutes “funding requirement,” whether FBA is recognized, what curve is used for discounting, and whether funding is computed at trade, netting-set, or bank level). The following mapping is **illustrative**:

$$\text{FCA} \approx \sum_{i=1}^{N} D(0, t_i^{\ast}) \cdot s_f \cdot F(t_i^{\ast}) \cdot \Delta t$$

where $s_f$ is the funding spread over the discounting rate (units: 1/year), $F(t)$ is the expected funding requirement (currency), and $\Delta t$ is the year fraction.

**Check (units and limits):**
- Units: $D$ is unitless, $s_f$ is 1/year, $F$ is currency, and $\Delta t$ is years $\Rightarrow$ each term is currency, as it should be for FCA.
- Limits: if $s_f=0$ or $F(t)\equiv 0$, then FCA collapses to 0. If you halve the time step (e.g., from annual to semiannual) while keeping the same funding profile, the sum should be broadly stable once you scale $\Delta t$ consistently.

**Example 34.5 — FCA Calculation (Toy)**

| Interval | Midpoint | Funding Requirement $F$ | $D(0, t_i^{\ast})$ |
|----------|----------|------------------------|----------------|
| 1 | 0.5 | \mathrm{USD}\\,10M | 0.99 |
| 2 | 1.5 | \mathrm{USD}\\,8M | 0.97 |
| 3 | 2.5 | \mathrm{USD}\\,6M | 0.95 |

Funding spread $s_f = 1\\%$ per year, $\Delta t = 1$ year per interval.

$$\text{FCA} = 0.99 \times 0.01 \times 10M \times 1 + 0.97 \times 0.01 \times 8M \times 1 + 0.95 \times 0.01 \times 6M \times 1$$
$$= 99{,}000 + 77{,}600 + 57{,}000 = \boxed{\mathrm{USD}\\,233{,}600}$$

If the bank also recognizes funding benefit (FBA) for negative funding requirement, it computes an analogous term and nets: $\text{FVA} = \text{FCA} - \text{FBA}$.

### 34.6.6 MVA: Margin Value Adjustment

The **margin valuation adjustment (MVA)** captures the cost of funding **initial margin (IM)**. IM arises for cleared trades and (in some setups) for bilateral portfolios. The exact IM methodology (and interest on IM) is product-, venue-, and policy-dependent.

Unlike FVA, **incremental MVA** for CCP-cleared transactions is usually **portfolio-dependent**, because IM is calculated on the cleared portfolio: a new trade can increase, decrease, or barely change total IM depending on offsets.

**Check (toy magnitude):** If incremental IM is $\mathrm{USD}\\,10\text{M}$, the incremental funding spread is $50\text{bp}=0.50\%$ per year, and the effective horizon is 2 years, then a back-of-the-envelope IM funding cost is $\mathrm{USD}\\,10\text{M}\times 0.50\%\times 2=\mathrm{USD}\\,100\text{k}$ (before discounting). If your MVA is orders of magnitude larger or smaller, first check units (bp vs %) and what “IM balance” you are funding (posted vs net).

---

## 34.7 KVA: Capital Value Adjustment

### 34.7.1 The Capital Cost Problem

Regulatory capital requirements tie up equity. If shareholders require 15% returns and a trade requires \mathrm{USD}\\,10 million of additional capital, should the trade earn 15% on that capital?

**KVA** (capital valuation adjustment) is a charge to reflect incremental capital costs.

Two competing framings show up in practice:

- **Practitioner framing:** apply a hurdle rate (target return on equity) to incremental capital and charge KVA so the trade clears the hurdle.
- **Finance-theory objection:** the relevant cost is **marginal** and should reflect the incremental risk of the project; using a bank-wide average can overcharge low-risk, capital-efficient trades.

### 34.7.2 KVA and Capital Framework Plumbing

> **Desk Reality: The Balance Sheet is Not Free**
>
> For banks subject to regulatory capital frameworks, derivatives generally consume regulatory capital. A common capital-to-price plumbing is:
>
> 1. **Exposure at Default (EAD):** A supervisory measure of counterparty exposure based on replacement cost (today's MTM after collateral) plus an add-on meant to capture potential future exposure (PFE). The exact formula depends on the regulatory approach and your jurisdiction/bank implementation.
>
> 2. **Risk-Weighted Assets (RWA):** Apply a counterparty risk weight to EAD. How the risk weight is set depends on the regulatory method and jurisdiction.
>
> 3. **Required Capital:** Multiply RWA by the capital ratio required by regulation + buffers (your bank typically uses an internal “all-in” ratio).
>
> 4. **KVA:** Convert required capital into a pricing charge by applying a cost of capital over the horizon (in practice often via a discounted sum/integral across future capital profiles).
>
> **Toy example (illustrative numbers):** A 5-year USD 100M interest rate swap with a BBB corporate:
> - Assume EAD (per your capital model) = USD 5M
> - Assume risk weight = 100%
> - Assume all-in capital ratio = 12% → required capital = USD 600K
> - If cost of capital = 15%, annual capital cost ≈ USD 90K
> - Over 5 years (discounted) → on the order of a few hundred thousand dollars
>
> This is why desks care about capital consumption: even if mid-market NPV is near zero, the “all-in” economics after KVA can change whether the trade clears the hurdle.

### 34.7.3 KVA Formula

The general KVA formula integrates capital requirements over the life of the trade:

$$\boxed{\text{KVA} = \int_0^T h_c \times K(t) \times d(t) \\, dt}$$

where:
- $h_c$ = hurdle rate (cost of capital minus risk-free rate)
- $K(t)$ = expected regulatory capital at time $t$
- $d(t)$ = discount factor to time $t$

In discrete form:
$$\text{KVA} = \sum_{i=1}^{n} h_c \times K_i \times \Delta t_i \times d_i$$

**Check (units and toy magnitude):**
- Units: $h_c$ is 1/year, $K$ is currency, $d$ is unitless, and $\Delta t$ is years $\Rightarrow$ KVA is currency.
- Toy magnitude: if $K(t)\approx \mathrm{USD}\\,1\text{M}$ is roughly flat over 5 years and $h_c\approx 8\%$, then ignoring discounting $\text{KVA}\approx 0.08\times 1\text{M}\times 5\approx \mathrm{USD}\\,400\text{k}$.

### 34.7.4 The Complete XVA Stack

Putting it all together:

$$V_{\text{all-in}} = f_{\text{nd}} - \text{CVA} + \text{DVA} - (\text{FCA} - \text{FBA}) - \text{MVA} - \text{KVA}$$

All XVAs are "computationally time-consuming to calculate. Monte Carlo simulations are necessary to determine expected credit exposures, expected funding costs, and expected capital requirements at future times."

---

## 34.8 The Deal Ticket: Putting It All Together

**Example 34.6 — Comprehensive XVA Decomposition**

A derivatives desk is quoting a 7-year \mathrm{USD}\\,50 million notional receiver swap (bank receives fixed, pays floating) to a BBB-rated corporate client. The swap will be uncollateralized.

**Given:**
- Mid-market NPV: +\mathrm{USD}\\,1,200,000 (to bank)
- Counterparty CDS spread: 150bp (5-year), recovery = 40%
- Bank funding cost: SOFR + 45bp
- Expected peak exposure: \mathrm{USD}\\,4.5M (occurs around year 4)
- Incremental capital requirement (per the bank’s capital model): \mathrm{USD}\\,2.2M
- Bank's cost of capital: 12% (target ROE)
- Risk-free rate (for discounting KVA): 4%

**Step 1: CVA Calculation**

Using the simplified CVA formula with average expected exposure:

5-year survival probability: $e^{-0.015 \times 5/0.6} = 0.8825$

Approximate 7-year cumulative default probability ≈ 15%

Average expected exposure (discounted) ≈ \mathrm{USD}\\,2.5M

$$\text{CVA} \approx (1-0.40) \times 0.15 \times \mathrm{USD}\\,2.5M = \mathrm{USD}\\,225,000$$

**Step 2: DVA Calculation**

Assume bank CDS spread: 60bp, bank's negative exposure (when swap is in-the-money to counterparty) averages \mathrm{USD}\\,1.5M.

Bank's 7-year default probability ≈ 6%

$$\text{DVA} \approx (1-0.40) \times 0.06 \times \mathrm{USD}\\,1.5M = \mathrm{USD}\\,54,000$$

**Step 3: FVA Calculation**

The swap has positive value, so the bank has a funding cost. Average positive MTM over the life ≈ \mathrm{USD}\\,800K.

Funding spread = 45bp

$$\text{FVA} \approx 0.0045 \times \mathrm{USD}\\,800K \times 5 \text{ (effective duration)} = \mathrm{USD}\\,18,000$$

**Step 4: MVA Calculation**

Initial margin estimate (per the bank’s IM model) ≈ \mathrm{USD}\\,1.8M

Funding cost of IM = 45bp

$$\text{MVA} \approx 0.0045 \times \mathrm{USD}\\,1.8M \times 6 \text{ (years)} = \mathrm{USD}\\,48,600$$

**Step 5: KVA Calculation**

Capital requirement: \mathrm{USD}\\,2.2M

Cost of capital above risk-free: 12% - 4% = 8%

$$\text{KVA} \approx 0.08 \times \mathrm{USD}\\,2.2M \times 5 \text{ (effective duration)} = \mathrm{USD}\\,880,000$$

**Step 6: Deal Ticket Summary**

| Component | Amount | Sign |
|-----------|--------|------|
| Mid-Market NPV | \mathrm{USD}\\,1,200,000 | + |
| CVA | \mathrm{USD}\\,(225,000) | − |
| DVA | \mathrm{USD}\\,54,000 | + |
| FVA | \mathrm{USD}\\,(18,000) | − |
| MVA | \mathrm{USD}\\,(48,600) | − |
| KVA | \mathrm{USD}\\,(880,000) | − |
| **Net Desk Economics** | **\mathrm{USD}\\,82,400** | |

**Interpretation:** The "profitable" \mathrm{USD}\\,1.2M trade actually nets only $82K for the desk after XVA charges. The KVA alone consumes $880K—over 70% of the gross NPV.

> **Desk Reality: The Deal/No-Deal Decision**
>
> At \mathrm{USD}\\,82K net, should the desk do this trade?
>
> **Arguments for:**
> - Client relationship value
> - Cross-sell opportunity
> - Flow that improves hedging efficiency elsewhere
>
> **Arguments against:**
> - ROE on \mathrm{USD}\\,2.2M capital = $82K / $2.2M / 7 years = 0.5% per year
> - Bank's hurdle rate is 12%—this trade destroys value
> - Capital better deployed elsewhere
>
> **The XVA desk's role:** Provide these numbers so the decision is informed. Don't let sales book a trade as "$1.2M profit" when desk economics say $82K.

---

## 34.9 Wrong-Way Risk

### 34.9.1 Definition

**Wrong-way risk (WWR)** is the situation where a counterparty is more likely to default in the same states of the world where your exposure to them is high. **Right-way risk (RWR)** is the opposite: default is more likely when your exposure is low.

In practice, the “independence” simplification behind many toy CVA calculations is treating default probabilities and exposures as independent inputs. WWR is the reminder that the economically correct object is a *joint* expectation: the default event and the exposure-at-default can be statistically dependent.

### 34.9.2 Common WWR Patterns (Intuition)

WWR tends to show up when the same risk factor drives both:
1) your exposure profile, and  
2) the counterparty’s credit quality.

Examples:
- A commodity producer counterparty vs a commodity derivative: commodity prices move your exposure *and* the producer’s default likelihood.
- A highly leveraged counterparty taking one-way bets: the same market move both increases your MTM against them and weakens their balance sheet.

> **Desk Reality:** WWR is handled either by explicitly modeling dependence between market factors and counterparty credit, or by applying conservative add-ons to independence-based CVA.
> **Common break:** treating WWR as “just a bigger PD” without checking what scenarios actually drive exposure.
> **What to check:** identify the dominant exposure driver and ask whether the counterparty’s spreads tend to widen in those same scenarios.

### 34.9.3 Worked Example: Wrong-Way Risk Impact on CVA

**Example 34.7 — Independence vs. Wrong-Way Risk (Two-State Model)**

Consider a single-interval setup with LGD = 0.6 and two equally likely states:

| State | Probability | Exposure $E$ | Default Prob $p$ |
|-------|------------|--------------|------------------|
| A ("bad") | 50% | \mathrm{USD}\\,20M | 5% |
| B ("good") | 50% | \mathrm{USD}\\,2M | 1% |

**Independence-based CVA (using averages):**
- Average exposure: $\bar{E} = 0.5 \times 20 + 0.5 \times 2 = \mathrm{USD}\\,11M$
- Average PD: $\bar{p} = 0.5 \times 0.05 + 0.5 \times 0.01 = 3\\%$

$$\text{CVA}_{\text{ind}} = 0.6 \times 11M \times 0.03 = \mathrm{USD}\\,198{,}000$$

**Wrong-way risk CVA (expected product, acknowledging correlation):**

$$\mathbb{E}[E \cdot p] = 0.5 \times (20M \times 0.05) + 0.5 \times (2M \times 0.01) = 0.5 \times 1{,}000{,}000 + 0.5 \times 20{,}000 = 510{,}000$$

$$\text{CVA}_{\text{WWR}} = 0.6 \times 510{,}000 = \boxed{\mathrm{USD}\\,306{,}000}$$

**Increase due to wrong-way risk:** $306{,}000 - 198{,}000 = \mathrm{USD}\\,108{,}000$ (+54.5%)

**Takeaway:** The positive correlation between exposure and default probability materially increases CVA. In this stylized example, using “average exposure × average PD” understates the true expected loss by over 50%.

---

## 34.10 The XVA Desk: Organization and Transfer Pricing

### 34.10.1 How Banks Structure XVA Management

Banks organize XVA management in one of two ways:

**Centralized XVA Desk:**
- A dedicated desk calculates and owns all XVA P&L
- Trading desks are "charged" XVA at trade inception
- XVA desk hedges the aggregate CVA/DVA exposure
- Common at larger dealer-style institutions (exact org design varies)

**Embedded in Business Lines:**
- Each trading desk owns its own XVA
- Desks hedge their own CVA exposure
- Coordination is informal
- More common when the book is smaller or less centralized

> **Desk Reality: Who Owns the P&L?**
>
> **At trade inception:**
> - Trader shows +\mathrm{USD}\\,1.2M NPV (mid-market)
> - XVA desk charges \mathrm{USD}\\,400K (CVA + FVA + KVA, net of DVA)
> - Trader's day-one P&L: \mathrm{USD}\\,800K
> - XVA desk books a \mathrm{USD}\\,400K reserve
>
> **Over the trade's life:**
> - Trader's P&L: changes in mid-market value
> - XVA desk P&L: changes in XVA values (spread moves, exposure changes)
>
> **At maturity/close-out:**
> - XVA reserve releases (if no default occurred)
> - XVA desk shows profit equal to original charge minus hedging costs
>
> **The conflict:** Traders want low XVA charges; XVA desk wants conservative reserves. Both report to the same P&L, so there's tension about "fair" charging.

### 34.10.2 XVA Reserve vs. XVA Charge

A common source of confusion:

| Concept | Timing | Nature |
|---------|--------|--------|
| **XVA Charge** | At trade inception | Upfront cost to the trading desk |
| **XVA Reserve** | Over trade life | Liability on the balance sheet, released as risk declines |

The charge is what hits the trader's P&L immediately. The reserve is what the bank carries until the exposure resolves (trade matures, counterparty defaults, or trade is closed out).

### 34.10.3 Regulatory vs. Accounting vs. Economic XVA

Different stakeholders can use different “views” of value and adjustments:

| View | Purpose | What It Typically Emphasizes |
|---|---|---|
| **Financial reporting view** | Period-end valuation and disclosures | Credit adjustments (often CVA/DVA) under the applicable framework |
| **Capital / regulatory view** | Supervisory capital and limits | Exposure measures and capital charges that may not align 1:1 with “desk XVA” |
| **Economic / desk view** | Deal approval and transfer pricing | A bank-specific XVA stack (CVA, DVA, FVA, MVA, KVA) consistent with internal policies |

Banks must reconcile these versions. DVA is particularly tricky: it can show up as an accounting valuation effect, but it does not automatically translate into usable loss-absorbing capital.

---

## 34.11 Collateralized vs. Uncollateralized Economics

### 34.11.1 The Collateral Trade-Off

> **Desk Reality: The Collateral Trade-Off**
>
> In an idealized model of frequent two-way variation margin with tight operational features (small thresholds, frequent calls, small MPOR), collateralization can reduce exposure dramatically, pushing CVA/DVA down.
>
> But the economics do not disappear: initial margin and liquidity/funding policies can introduce MVA/FVA-style costs, and operational/legal features (MPOR, closeout, rehypothecation/segregation) determine the residual risk.
>
> **What to check:** when someone says “CVA is basically gone,” ask what assumptions are being made about MPOR, margin frequency, thresholds/MTA, and whether initial margin is in scope.

### 34.11.2 Cleared vs. Bilateral Comparison

| Aspect | Bilateral Uncollateralized | Bilateral with CSA | CCP Cleared |
|--------|---------------------------|-------------------|-------------|
| CVA | Higher | Lower (MPOR / gap-risk residual) | Lower (not zero; depends on margining + liquidation assumptions) |
| FVA | Can be material | Often smaller than uncollateralized | Depends (VM reduces some funding need; IM can create another) |
| MVA | Typically none | Depends (if IM or independent amounts exist) | Often material (IM funding cost) |
| Margin calls | None | Periodic (often daily) | Daily (by venue) |
| Close-out risk | Higher | Lower | Lower (but not eliminated) |
| Documentation | Bilateral master agreement + terms | Bilateral master agreement + CSA | CCP rules + clearing agreements |

### 34.11.3 CSA Features That Move XVA (Beyond “Collateral Yes/No”)

Treat “collateralization” as a bundle of operational/legal features, not a single boolean. The following CSA/margining features are usually first-order drivers of residual exposure and funding effects:

Rehypothecation is one of the features that can make the economics less intuitive. If rehypothecation is permitted, the collateral receiver can use the same collateral to satisfy a demand for collateral from another party.

| Feature | Why it matters for XVA | Quick check |
|---|---|---|
| Margin frequency + MPOR | Even with VM, exposure at default reflects stale MTM and liquidation lag (gap risk). | If you set MPOR to 0 and assume frictionless VM, CVA should fall sharply. |
| Thresholds / MTA / independent amounts | These create “dead zones” where exposure is not fully covered by collateral. | If thresholds/MTA are nonzero, small MTM moves should not trigger collateral, so exposure remains. |
| Eligible collateral + haircuts | Non-cash collateral is typically haircut, reducing effective protection; collateral value can move during MPOR. | Stress collateral value during MPOR and confirm exposure increases in the direction you expect. |
| Segregation vs rehypothecation | If collateral can be re-used, it can change funding economics and default-time claims; some modeling choices treat it as a funding resource, which can introduce recursion. | If you allow rehypothecation in the model, verify funding requirement depends on $V$ and that the algorithm solves consistently (often backward/recursive). |
| Closeout convention | Determines the payoff at default and the meaning of “DVA” (risk-free vs replacement closeout). | Check that your closeout convention is consistent with your bilateral valuation and does not create a valuation jump at default. |

---

## 34.12 CVA Risk Management

### 34.12.1 CVA Is a Risk Position, Not Just a Number

Once you compute CVA, you have created a **risk position** whose value moves with:
1. **Counterparty credit** (spreads/hazards move $\Rightarrow q_i$ moves), and
2. **Market factors** (rates/FX/commodities move $\Rightarrow$ the exposure profile $v_i$ moves).

A useful mental model is: CVA behaves like a path-dependent “option-like” claim on exposure-at-default. It inherits model risk from both the exposure engine and the credit curve inputs.

### 34.12.2 CVA01 (Credit Spread Risk): Define the Bump Precisely

Define **CVA01** as a bump-and-reprice sensitivity to a parallel **+1bp** shift in the counterparty’s credit spread term structure $s(t)$ (or the corresponding hazard curve $h(t)$, if that is your internal object):

$$
\mathrm{CVA01} := \mathrm{CVA}(s+1\text{ bp})-\mathrm{CVA}(s).
$$

To make this unambiguous, always state:
- **Bump object:** the counterparty spread curve $s(t)$ (or hazard curve $h(t)$) used to produce $q_i$.
- **Bump size:** $1\text{ bp}=10^{-4}$ in rate units.
- **Units:** currency per 1bp (usually reported per netting set or per counterparty).
- **Sign convention:** with CVA defined as a non-negative cost, $\mathrm{CVA01}\ge 0$ in typical cases.

**What is held fixed?** A common *credit-only* approximation is to hold the exposure PV profile $v_i$ fixed and recompute only the default probabilities $q_i$. A more complete approach would also allow the exposure profile to change (market-factor moves, collateral dynamics, and any feedback from funding/collateral policies).

> **Pitfall — What is being bumped?:** “CVA01” is meaningless unless you specify whether you bump a hazard curve, a fitted spread curve, or market quotes (and whether you rebootstrap after the bump).
> **Why it matters:** different bump designs can give different numbers (and different hedges).
> **Quick check:** re-run CVA01 with an alternative bump design (hazard vs spread) and confirm differences are explainable by your conventions.

**Example 34.8 — CVA01 via bump-and-reprice (toy)**

Using the spread-to-hazard approximation $h\approx s/(1-R)$, a +1bp spread bump corresponds to a hazard bump of $\Delta h\approx 10^{-4}/(1-R)$. For $R=40\\%$, $\Delta h\approx 1.667\times 10^{-4}$ per year. Recompute $q_i$ under the bumped hazard and re-run the discrete CVA sum holding $v_i$ fixed. The difference $\mathrm{CVA}(s+1\text{bp})-\mathrm{CVA}(s)$ is the CVA01 in USD/bp.

### 34.12.3 Hedging and Governance (High-Level)

At a high level, desks manage CVA risk via:
- **Credit hedges:** instruments linked to counterparty spread risk (e.g., single-name CDS when available).
- **Market hedges:** hedges that reduce the exposure profile (rates/FX/commodity deltas/vegas that drive $v_i$).
- **Structural mitigants:** netting, collateral terms, and closeout conventions that reduce exposure-at-default in the first place.

## 34.14 Practical Implementation Notes

### 34.14.1 What Goes Into an XVA System

A production XVA system implements the following end-to-end workflow:

**Step 1: Define Legal/Portfolio Scope**
- Identify legal netting sets (master agreement scope) and portfolio membership
- Capture early termination logic and settlement conventions
- CSA terms: thresholds, minimum transfer amount, independent amount / initial margin, eligible collateral, haircuts, rehypothecation status

**Step 2: Define Collateral Mechanics**
- Margin frequency (daily vs less frequent), collateral currencies, interest on cash collateral
- Cure period / MPOR (often 10–20 days)
- Exposure at default uses $E = \max(V - C, 0)$

**Step 3: Generate Future Market Scenarios**
- Choose models for relevant risk factors (rates, FX, equities, commodities, volatilities)
- Simulate under risk-neutral dynamics for CVA valuation
- Choose time grid $\\{t_i^{\ast}\\}$ aligned with product features (exercise dates, resets, payment dates)

**Step 4: Revalue Portfolio in Each Scenario**
- For each path and each midpoint $t_i^{\ast}$, compute portfolio MTM $V(t_i^{\ast})$
- For cure-period modeling, also compute $V(t_i^{\ast} - c)$ to infer collateral at $t_i^{\ast}$

**Step 5: Compute Exposures**
- Pathwise net exposure: $E(t_i^{\ast}) = \max(V(t_i^{\ast}) - C(t_i^{\ast}), 0)$
- Expected exposure $\text{EE}(t_i^{\ast})$: average across Monte Carlo trials
- Peak exposure / PFE: high percentile at each midpoint (e.g., 97.5%)

**Step 6: Compute Default Probabilities**
- Build counterparty risk-neutral default curve from credit spreads / CDS / bond-implied
- Use spread-to-hazard approximation $\bar{\lambda} = s/(1-R)$ or bootstrap full CDS curve (Chapter 42)
- Convert to interval probabilities $q_i = Q(t_{i-1}) - Q(t_i)$

**Step 7: Assemble CVA and DVA**
- CVA: combine EE, LGD, discounting, and $q_i$ via discrete sum
- DVA: same mechanics with own default probabilities and negative exposure

**Step 8: Compute FVA (Policy-Dependent)**
- Identify funding requirements: uncollateralized positive MTM, posted margin
- Choose funding spread and whether FBA is recognized
- Note: FVA implementation is policy-driven (see Section 34.6)

**Step 9: Produce Sensitivities and P&L Explain**
- CVA spread sensitivity (CVA01): relationship between $q_i$ and credit spreads
- Exposure Greeks: sensitivities of $v_i$ to market variables
- Incremental XVA: stored simulation paths for new-trade pricing (see Section 34.14.2)

### 34.14.2 Incremental XVA for New Trades

When a new trade is proposed, the desk needs to know its incremental CVA impact. Recalculating the full Monte Carlo is impractical.

One common approach is to **store** the market-factor paths (and the resulting portfolio values) from a prior full run. To assess a candidate new trade, revalue that new trade on the stored paths and compute its incremental effect on portfolio exposure and XVA.

The stored-path approach allows rapid "incremental XVA" pricing for trade approval by computing:
1. New trade's value on each stored path at each time
2. Incremental impact on portfolio value and exposure
3. Incremental effect on CVA via $\sum (1-R) q_i \Delta v_i$

### 34.14.3 Machine Learning Applications

Given the computational intensity, some institutions use **surrogate models** (e.g., regression models or neural networks) to approximate XVA as a function of portfolio features. The idea is to train on many Monte Carlo-labeled examples and then evaluate the surrogate quickly for real-time quoting.

The approach:
1. Generate many random portfolio configurations
2. Compute XVAs via full Monte Carlo
3. Train neural network to replicate results
4. Use network for real-time incremental XVA

This allows rapid responses to trade inquiries while maintaining accuracy calibrated to full Monte Carlo.

### 34.14.4 Validation and Sanity Checks

**Core Monotonicity Checks:**
CVA should increase when expected exposure increases, default probabilities/hazards increase, or recovery decreases (LGD increases). All follow directly from the discrete-sum structure. DVA should increase when expected negative exposure increases and own hazard increases.

**Collateral Sanity Check:**
Under idealized daily two-way VM with zero threshold and no cure-period lag, exposure should be near zero → CVA near zero. Residual exposure arises precisely from the cure period and collateral mismatch (see Example 34.2).

**Peak Exposure (PFE) Sanity:**
PFE (high percentile) should exceed average exposure at each time; maximum PFE should occur near periods of high optionality or volatility.

**P&L Explain (Daily XVA Decomposition):**
CVA P&L decomposes into:
- Credit-spread moves ($q_i$ moves)
- Exposure/market-factor moves ($v_i$ moves)

DVA P&L similarly depends on own spread and expected negative exposure. Any unexplained residual should be investigated — common causes include time discretization error (coarse grids missing exposure peaks), inconsistent CSA modeling, or mixed discounting frameworks.

**Implementation Pitfalls:**

| Pitfall | Consequence | Fix |
|---------|-------------|-----|
| Inconsistent netting/collateral modeling | Overstated or understated CVA | Ensure exposure computed on enforceable netting sets |
| Missing cure-period modeling | Understated exposure spikes | Value at $t_i^{\ast} - c$ to infer collateral at $t_i^{\ast}$ |
| Wrong sign conventions | P&L explain errors | Remember: $f_{\text{nd}} - \text{CVA} + \text{DVA}$ |
| Coarse time grids | Missed exposure peaks | Align grid with cashflow dates, resets, exercise dates |
| Mixing discounting with FVA | Double-counted funding | Document discounting + FVA policy together |
| Ignoring wrong-way risk | Understated CVA | Model dependence explicitly or apply conservative add-ons |

---

## Summary

1. **XVAs adjust a clean PV**: start from a no-default (clean) value $f_{\text{nd}}$ and apply a signed stack to get an all-in value.
2. **CVA is expected loss pricing**: $\text{CVA}$ is the PV of expected loss at counterparty default, driven by exposure $v_i$, default probabilities $q_i$, and LGD $(1-R)$.
3. **DVA is symmetric but delicate**: it depends on your own default inputs and on closeout conventions; it can look like a gain when your credit worsens.
4. **Collateral reduces but does not erase exposure**: MPOR/gap risk and CSA features determine the residual exposure-at-default.
5. **FVA/MVA/KVA are policy-heavy**: funding, margin, and capital costs depend on internal assumptions; funding-inclusive valuation can become recursive.
6. **Define risk measures precisely**: for CVA01, state bump object, bump size, units, sign, and what is held fixed vs recomputed.
7. **Wrong-way risk is dependence risk**: when exposure and counterparty credit co-move, independence-based CVA can be materially understated.
8. **XVA is also governance**: desk organization and transfer pricing determine how XVA is charged, hedged, and explained in P&L.

## Key Concepts

| Concept | Definition | Why It Matters |
|---|---|---|
| Clean value $f_{\text{nd}}$ | No-default PV under a stated discounting convention | Baseline for all adjustments |
| All-in value $V_{\text{all-in}}$ | Clean PV plus a signed stack of adjustments | What drives pricing, reserves, and deal approval |
| Netting set | Enforceable portfolio that closes out together | CVA is a netting-set quantity, not trade-by-trade |
| CVA | PV of expected loss from counterparty default | Core counterparty credit cost |
| DVA | PV effect of own default on liabilities | Creates “phantom” valuation gains and governance issues |
| TVA | Aggregate adjustment when credit + funding constraints are included | Highlights that the stack can be non-unique / non-separable |
| MPOR / cure period | Lag between last effective margin exchange and closeout | Source of residual exposure under collateralization |
| Closeout convention | Risk-free vs replacement style closeout at default | Changes default-time payoff and DVA interpretation |
| FVA (FCA−FBA) | PV of funding costs minus funding benefits (policy-dependent) | Major source of bank-to-bank price dispersion |
| MVA | PV cost of funding initial margin (policy-dependent) | Prominent for cleared or IM-heavy portfolios |
| KVA | PV cost of capital tied up over time (policy-dependent) | Drives “hurdle rate” economics and transfer pricing |
| WWR / RWR | Dependence between exposure and default likelihood | Can materially change CVA vs independence assumption |
| CVA01 | $\text{CVA}(s+1\text{bp})-\text{CVA}(s)$ under stated bump rules | Credit spread risk scalar used for hedging/governance |
| Rehypothecation | Re-use of received collateral (when allowed) | Changes funding economics and default-time claims assumptions |

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $f_{\text{nd}}$ | clean (no-default) value | currency; positive = asset |
| $V_{\text{all-in}}$ | value including XVAs | currency; uses this chapter’s sign table |
| $P(0,t)$ | discount factor | unitless |
| $Q(t)$ | survival probability to $t$ | unitless; $Q(0)=1$ |
| $h(t)$ | hazard rate (intensity) | 1/year; $Q(t)=\exp(-\int_0^t h)$ |
| $R$, $LGD$ | recovery, loss-given-default | unitless; $LGD=1-R$ |
| $E(t)$ | exposure | currency; $E(t)=\max(V(t)-C(t),0)$ |
| $C(t)$ | collateral held by the bank at $t$ | currency; positive reduces exposure in $E(t)$ |
| $q_i$ | default probability in interval $i$ | unitless |
| $v_i$ | PV of expected exposure in interval $i$ | currency |
| $s(t)$ | credit spread term structure (generic) | 1/year (or bp); $1\text{bp}=10^{-4}$ |
| $\mathrm{CVA01}$ | credit spread sensitivity | currency per 1bp; bump rules must be stated |
| $F(t)$, $s_f$ | funding requirement and funding spread | currency; 1/year |
| $K(t)$, $h_c$ | capital profile and hurdle rate | currency; 1/year |

## Flashcards

| # | Question | Answer |
|---:|---|---|
| 1 | What is the difference between “clean” and “all-in” value? | Clean ignores default/funding/capital frictions; all-in adds a signed XVA stack consistent with a policy and conventions. |
| 2 | Under the chapter sign convention, why is CVA subtracted? | CVA is treated as a non-negative expected loss cost to the bank, so it reduces value. |
| 3 | Write the discrete unilateral CVA formula used in this chapter. | $\text{CVA}=\sum_i (1-R)\\,q_i\\,v_i$. |
| 4 | What object is $v_i$? | A discounted expected exposure amount (net of collateral) for the netting set in interval $i$. |
| 5 | Why does collateral not make CVA identically zero? | Because exposure at default reflects MPOR/gap risk and CSA features (timing, thresholds, disputes, closeout). |
| 6 | What is MPOR (cure period) in words? | The time between the last effective collateral exchange and the closeout/settlement of the portfolio after default. |
| 7 | What is the economic meaning of DVA? | It reflects that some liabilities may not be paid in full if you default; symmetric to the counterparty’s CVA view. |
| 8 | Risk-free vs replacement closeout: what is the key difference? | Whether the surviving party’s own residual credit is included in the default-time valuation of remaining cashflows. |
| 9 | What is TVA? | An aggregate “total valuation adjustment” umbrella; highlights that decomposing into CVA/DVA/FVA can be a modeling choice, and funding can introduce recursion. |
| 10 | What is wrong-way risk (WWR)? | Default is more likely in the scenarios where exposure is high (dependence between exposure and credit). |
| 11 | Define CVA01 precisely. | $\mathrm{CVA01}:=\mathrm{CVA}(s+1\text{bp})-\mathrm{CVA}(s)$ where $s(t)$ is the counterparty spread curve and $1\text{bp}=10^{-4}$. State what is held fixed. |
| 12 | What must you specify for any “01” risk metric? | Bump object, bump size, units, sign convention, and whether you rebuild curves after the bump. |
| 13 | What is the common “credit-only” approximation for CVA01? | Hold the exposure profile $v_i$ fixed and recompute only the default probabilities $q_i$ under the bumped credit curve. |
| 14 | What is FVA (high-level)? | A policy-dependent adjustment for funding costs/benefits when collateralization is asymmetric. |
| 15 | Why can FVA become recursive? | Funding requirements can depend on the adjusted value process, but that value process depends on funding policy assumptions. |
| 16 | What is the DVA/FVA overlap pitfall? | Counting an own-credit benefit in both DVA and the funding term, producing wrong P&L attribution and sensitivities. |
| 17 | What is rehypothecation? | If rehypothecation is permitted, the collateral receiver can re-use the same collateral (e.g., to meet collateral calls from another party). |
| 18 | What is the desk reality behind XVA disagreements across banks? | Different netting sets/CSAs, margin assumptions, funding spreads, IM policies, and capital policies lead to different all-in numbers. |
| 19 | What is the “stored paths” idea for incremental XVA? | Reuse previously simulated market-factor paths and revalue only the proposed new trade to estimate incremental impact quickly. |
| 20 | What is a surrogate XVA model? | A fast approximation (e.g., regression/NN) trained on Monte Carlo outputs to speed quoting and what-if analysis. |

## Mini Problem Set

1. (Concept) Define clean value $f_{\text{nd}}$ vs all-in value $V_{\text{all-in}}$. Under this chapter’s sign convention, why is CVA subtracted and DVA added?
2. (Compute) A netting set has PV expected exposures of \mathrm{USD}\\,3mm at year 1 and \mathrm{USD}\\,2mm at year 2. Default probability in each year is 2% and recovery is 40%. Compute CVA (assume the exposures are already discounted PVs).
3. (Concept) Explain why MPOR (cure period) creates residual exposure even with frequent variation margin.
4. (Compute) A counterparty has a flat 5-year credit spread of 150bp. Using $h\approx s/(1-R)$ with $R=40\\%$, compute $Q(5)$.
5. (Compute) With $R=40\\%$, what hazard bump corresponds to a +1bp spread bump under $h\approx s/(1-R)$?
6. (Concept) Define CVA01 and list bump object, bump size, units, sign, and what is held fixed vs recomputed in a “credit-only” approximation.
7. (Desk) Two banks agree on clean PV but disagree on all-in price. List 4–6 questions you would ask to reconcile the difference.
8. (Concept) In one paragraph, explain why adding funding effects can make the valuation recursive (non-separable).
9. (Concept) Compare risk-free vs replacement closeout in one paragraph. Why can changing the convention change the meaning of “DVA”?
10. (Concept/Desk) Give an example of wrong-way risk and one conservative response a desk might take if it cannot model the dependence explicitly.

### Solution Sketches (Selected)
2. $\text{CVA}=0.6\times(0.02\\times3{,}000{,}000+0.02\\times2{,}000{,}000)=0.6\\times100{,}000=\boxed{\mathrm{USD}\\,60{,}000}$.
3. Even with frequent VM, collateral at default reflects a stale MTM and a liquidation lag. The MTM can move during MPOR, creating a gap between $V(\tau)$ and $C(\tau)$.
4. $h\approx 0.015/0.6=0.025$. So $Q(5)=e^{-0.025\\times5}=e^{-0.125}=\\boxed{0.8825}$ (default prob $\approx 11.75\\%$).
5. $\Delta h\\approx 10^{-4}/0.6\\approx\\boxed{1.667\\times10^{-4}}$ per year.

---

## References
- Hull, *Risk Management and Financial Institutions*, “CVA, DVA, wrong-way risk” (chapter on counterparty risk)
- Hull, *Options, Futures, and Other Derivatives*, “Credit and funding adjustments / CVA-DVA-FVA overview”
- Neftci, *Principles of Financial Engineering*, “CVA desk / pricing with CVA, DVA, FVA”
- Crépey, *Counterparty Risk and Funding*, “Outline” and “Closeout conventions (risk-free vs replacement)”
- Brigo, Morini, Pallavicini, *Counterparty Credit Risk, Collateral and Funding*, “Collateral, close-out and rehypothecation” and “Funding with collateral”
