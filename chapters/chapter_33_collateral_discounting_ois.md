# Chapter 33: Collateral Discounting and OIS

---

## Introduction

Collateral is not just a credit-risk mitigant; it changes the economics of a derivatives trade. When a trade is margined in cash and the collateral earns an overnight rate $c(t)$, the replication/funding logic pushes the discounting toward that same collateral rate.

This became unavoidable when overnight and term interbank rates diverged sharply during the 2007–2009 crisis (e.g., the spread between the USD overnight rate proxy and 3‑month LIBOR reached hundreds of basis points at its peak). In that environment, discounting a collateralized swap at a term bank rate while the collateral earns an overnight rate can create material PV and risk differences.

This chapter builds the “quote → cashflows → PV → risk” chain for collateralized OTC rates:
1. CSA terms → how collateral moves and when (VM/IM, thresholds, MTA, margin period of risk, rehypothecation/segregation).
2. Collateral remuneration rate $c(t)$ → discounting curve choice (often an OIS curve).
3. Multi-curve pricing → separate curves for discounting and for projecting the floating index.
4. What breaks → imperfect collateralization and why XVA enters.

## Learning Objectives
- Explain when “discount at the collateral rate” is justified and what assumptions it requires.
- Translate CSA terms into a cash/timeline view of exposure and funding.
- Define OIS discounting precisely (overnight index, curve object, and what is held fixed).
- Separate projection vs discounting curves and identify where single-curve identities fail.
- Compute and interpret discount-curve $DV01$ with explicit bump object, units, and sign.

Prerequisites: [Chapter 18 — OIS Discounting Curve](chapters/chapter_18_ois_discounting_curve.md), [Chapter 32 — Counterparty Exposure Basics](chapters/chapter_32_counterparty_exposure_basics.md)  
Prerequisites (optional refresher): [Chapter 19 — Projection Curves (Multi-Curve Framework)](chapters/chapter_19_projection_curves_libor_sofr_multi_curve.md), [Chapter 21 — Cross-Currency Curves](chapters/chapter_21_cross_currency_curves.md)  
Follow-on: [Chapter 34 — XVA Overview](chapters/chapter_34_xva_overview.md)

---

## 33.1 Collateral and CSA Mechanics

### 33.1.1 The Credit Support Annex (CSA)

A bilateral OTC derivatives relationship is typically documented under an ISDA Master Agreement with an annex called the **Credit Support Annex (CSA)**. The CSA specifies collateral mechanics, including (at a minimum):
- what collateral is exchanged (variation margin and sometimes initial margin)
- thresholds, independent amounts, and minimum transfer amounts (MTA)
- eligible collateral types and haircuts
- margin frequency and settlement timing
- interest paid on cash collateral (the **collateral remuneration rate**)
- rehypothecation/segregation and close-out / netting conventions

For valuation, the key point is: the CSA is part of the product. Two trades with identical contractual coupons can have different economics if their CSA terms differ, because the CSA changes both exposure and the effective funding of the hedged position.

### 33.1.2 Variation Margin (VM)

**Variation margin** is collateral exchanged as mark-to-market (MTM) changes. In a simple two-way CSA with zero threshold, if the MTM increases in value to party A by $X$ from one day to the next, party B posts collateral worth $X$ to party A (and vice versa if MTM moves the other way).

The purpose of VM is to keep current credit exposure close to zero by resetting collateral to match the MTM. In the ideal limit of continuous VM with no thresholds and no settlement frictions, current exposure is approximately zero.

### 33.1.3 Initial Margin (IM)

**Initial margin** is collateral posted at trade inception (or held throughout the trade's life) to protect against losses that might occur during the time it takes to close out a position after a counterparty default. While this chapter focuses on how collateral determines discounting—primarily a VM story—IM has become increasingly important under post-crisis regulations.

The distinction matters: VM aims to eliminate current exposure by matching collateral to MTM, while IM aims to cover **potential future exposure** over a liquidation period. A full treatment of IM requirements and their capital implications belongs in later chapters on regulatory capital; here we note only that IM posting has become more common due to regulation.

### 33.1.4 Thresholds and Minimum Transfer Amounts

Real-world CSAs rarely require collateral posting for every dollar of MTM movement. Two parameters create friction:

**Threshold ($H$):** The MTM level below which VM need not be posted. If the threshold is USD 10m, no collateral is required when $|V| \lt \mathrm{USD}\\,10\text{m}$, but if $V = \mathrm{USD}\\,11\text{m}$, the counterparty must post $\mathrm{USD}\\,1\text{m}$ in collateral.

**Minimum transfer amount (MTA):** The smallest collateral transfer that will actually occur. If MTA is $500{,}000$ and the required collateral movement is $300{,}000$, no transfer happens. This avoids the operational cost of nuisance transfers.

Both parameters leave **residual unsecured exposure**. A threshold of $H$ means that up to $H$ in MTM is never collateralized—this exposure is economically equivalent to unsecured credit risk.

### 33.1.5 The Cure Period (Margin Period of Risk)

Even with zero threshold and daily margining, exposure is not eliminated because margining is discrete and close-out takes time. A common modeling abstraction is to assume that the defaulting party stops posting collateral (and stops returning excess collateral) $c$ days before close-out. The parameter $c$ is called the cure period or margin period of risk (MPOR); toy calculations often use values like 10–20 days.

Under this abstraction, collateral at the default/close-out time reflects the portfolio value $c$ days earlier, not the portfolio value at default. Residual exposure is driven by the MTM move during the MPOR.

**Toy example (two-way, zero threshold; $c=20$ days):**
- MTM at default time: $V(\tau)=\mathrm{USD}\\,50\text{mm}$
- MTM 20 days earlier: $V(\tau-20d)=\mathrm{USD}\\,45\text{mm}$
- Collateral frozen near $\mathrm{USD}\\,45\text{mm}$ ⇒ net exposure $\approx \mathrm{USD}\\,5\text{mm}$

This cure-period effect explains why even "fully collateralized" trades carry credit exposure and therefore require CVA (see Chapter 34).

### 33.1.6 The Collateral Remuneration Rate

The collateral remuneration rate $c(t)$ is the contractual interest rate paid on posted cash collateral. For cash, it is commonly linked to an overnight benchmark for the collateral currency (sometimes with a spread, floor, or administrative conventions). In this chapter we treat $c(t)$ as the (possibly time-varying) collateral rate process specified by the CSA.

A useful mental model is:
- **CSA gives you $c(t)$:** the rate applied to the cash collateral balance.
- **Market gives you OIS quotes:** fixed vs realized overnight compounding, which lets you infer a discount-factor curve consistent with $c(t)$.

### 33.1.7 Rehypothecation

**Rehypothecation** is the practice of using collateral received from one counterparty to meet collateral demands from another—or more broadly, to fund hedging positions.

Rehypothecation matters because the clean “discount at $c(t)$” replication story typically assumes either:
- received cash collateral can be reused to fund the hedge, or
- you can otherwise fund/invest at the same rate $c(t)$ that applies to collateral.

If collateral is segregated and cannot be reused, the effective hedge funding can differ from the collateral remuneration economics; in that case, be explicit about what rate funds the hedge and whether an additional adjustment is required.

> **Pitfall — Rehypothecation vs segregation:** “Cash collateral earns $c(t)$” does not automatically mean “my hedge is funded at $c(t)$”. Always check reuse/segregation and the operational timing of margin.

---

## 33.2 The Economic Principle: Discounting Follows the Funding of the Hedge

### 33.2.1 The Core Insight: Collateral Rate = Discount Rate

For a **perfectly collateralized** derivative whose cash collateral accrues at rate $c(t)$, the clean no-arbitrage present value is obtained by discounting the contractual cashflows at $c(t)$.

Define the collateral discount factor

$$P^c(t,T) := \exp\left(-\int_t^T c(u)\\,du\right).$$

For a cashflow stream $CF(t_k)$ paid at dates $t_k$, the clean price is

$$V(t) = \mathbb{E}_t\left[\sum_k P^c(t,t_k)\\,CF(t_k)\right],$$

under the usual “clean” idealizations listed below.

**Check (units and direction):**
- $P^c(t,T)$ is a discount factor (unitless), $CF$ is currency $\Rightarrow V(t)$ is currency.
- For a deterministic positive cashflow, a higher discount factor means a higher PV. Equivalently, lowering $c(t)$ increases $P^c$ and increases PV.

**What assumptions are doing the work?** The statement above is exact only in an idealized setting. At minimum, you are assuming:
- cash collateral with remuneration specified by the CSA (the $c(t)$ process)
- perfect collateralization (collateral tracks MTM with no thresholds/MTA and no settlement lag)
- a close-out/settlement convention consistent with the collateralization scheme
- either rehypothecation is allowed or (economically equivalent) you can fund the hedge at the same rate $c(t)$

**Check (limiting case):** If $c(t)=r(t)$ (collateral rate equals the “risk-free” short rate used in a single-curve world), then $P^c(t,T)$ reduces to the standard money-market discount factor and you recover the familiar one-curve pricing identity.

### 33.2.2 The Replication Argument

The intuition is “fund the hedge the same way the trade is funded.”

- If $V(t)\gt 0$ (the trade is an asset to us), we receive collateral and (in the idealized case) can reuse it to fund the hedge, while paying the collateral remuneration rate on that balance.
- If $V(t)\lt 0$ (the trade is a liability to us), we post collateral and receive the collateral remuneration rate on the posted balance; the hedge financing is again tied to $c(t)$.

The point is not that the collateral rate is a metaphysical “risk-free rate,” but that it is the *relevant financing rate in the collateralized replication story*. Using a different discount rate would price the same cashflows with a funding assumption you are not actually facing under the CSA.

### 33.2.3 Numerical Arbitrage Example

To make the idea concrete, consider a simple toy example.

**Setup (hypothetical):**
- Receive \mathrm{USD}\\,1,000,000 exactly in 1 year.
- Collateral discount factor: $P^c(0,1)=0.9750$.
- A different (incorrect-for-this-CSA) discount factor: $P^{w}(0,1)=0.9700$.

Then:

$$PV^c = 1{,}000{,}000\times 0.9750 = \mathrm{USD}\\,975{,}000,\qquad
PV^w = 1{,}000{,}000\times 0.9700 = \mathrm{USD}\\,970{,}000.$$

The \mathrm{USD}\\,5,000 difference is purely a discounting choice: it is the PV impact of using the wrong funding/discounting assumption for a collateralized trade.

**Check:** For a positive cashflow, a higher discount factor must produce a higher PV.

### 33.2.4 The Formal Discounting Identity

Define the collateral account (the value of 1 unit of collateral cash rolled at the collateral rate):

$$\beta_c(t) := \exp\left(\int_0^t c(u)\\,du\right).$$

In continuous-time asset pricing, choosing a numeraire like $\beta_c$ induces a corresponding pricing measure under which “price divided by numeraire” is a martingale. This is a formal way of expressing the same idea: in the perfect-collateralization idealization, discounting at $c(t)$ is the right operation for clean pricing.

Different CSAs can imply different collateral numeraires (different currencies, different remuneration benchmarks, spreads/floors), which is why “OIS discounting” is only a shorthand until you specify the CSA.

### 33.2.5 Advanced Note: The Collateral Rate Is an *Effective* Discounting Rate

In practice, collateral is updated discretely (often daily), not continuously, and overnight benchmark rates are not “zero risk in every sense.” A helpful interpretation is to treat collateral discounting as access to a *special financing channel* created by the collateral agreement: you can lend/borrow (against collateral) at a rate that is close to the overnight index, even if that rate is not identical to an abstract, unobservable “true risk-free rate.”

This matters for language: use “OIS/overnight collateral curve used for discounting” rather than “the” risk-free curve, and be explicit about what is being approximated in your setup.

---

## 33.3 From CSA to OIS Discounting

### 33.3.1 What Is an OIS?

An **overnight indexed swap (OIS)** exchanges a fixed rate for the realized compounded return of an overnight benchmark over each accrual period.

For a single payment period $[T_1,T_2]$ with daily overnight rates $r_i$ and day fractions $\tau_i$, the overnight-compounded return is:

$$R_{\text{ON,comp}}(T_1,T_2) = \prod_{i=1}^{N} (1 + r_i \cdot \tau_i) - 1.$$

In practice, longer-dated OISs are typically structured as a sequence of subperiods (often quarterly), with net settlement at the end of each subperiod.

An **OIS curve** is the term structure implied by par OIS quotes. When the CSA remunerates cash collateral at the same overnight benchmark, that OIS curve is the natural proxy for the collateral discount curve.

### 33.3.2 Why OIS Discounting Became Standard Post-Crisis

When trades are collateralized in cash and collateral earns an overnight benchmark rate, discounting is tied to that same overnight collateral rate. Once term interbank rates and overnight rates diverge materially, discounting a collateralized trade at a term bank curve (while the collateral cash accrues at an overnight curve) produces a systematic PV difference.

This is one of the drivers of the modern “multi-curve” setup: use an overnight/OIS curve for discounting and a separate curve for projecting the floating index (Section 33.4).

### 33.3.3 Building an OIS Discount Curve

Bootstrapping an OIS curve follows the same logic as building any discount curve from par instruments: solve for discount factors so that each quoted OIS has zero PV at inception.

The technical details of curve construction belong in Chapter 18. Here we emphasize only that OIS quotes are market inputs that imply a discount-factor curve for the collateral rate benchmark.

### 33.3.4 Benchmark Names and CSA Specificity

“OIS” is shorthand: what matters is the **collateral remuneration benchmark in the CSA**. Two practical implications:

- Specify the overnight index and compounding convention (the object $c(t)$).
- Build/use a discount curve consistent with that index; do not assume every “OIS curve” is interchangeable.

For the rest of this chapter, we use $c(t)$ to denote the CSA collateral rate and treat “OIS discounting” as “discounting on the curve implied by instruments linked to $c(t)$.”

---

## 33.4 Multi-Curve Pricing: Discount vs. Projection

### 33.4.1 The Separation Principle

Discounting and projection are fundamentally different questions:

- **Discount curve:** what discount factors weight cashflows today, given the trade’s funding/collateralization economics?
- **Projection curve:** what forward rates determine the floating cashflows that will be paid/received?

In a collateralized rates trade, the discount curve is tied to the CSA collateral rate $c(t)$ (often proxied by an overnight/OIS curve). But the floating leg is defined by a *reference index* (historically a term interbank rate for many products; in other cases an overnight-compounded index). Once the reference index differs from the collateral rate, you need at least two curves:
1. a **discount curve** (collateral/OIS) for PV weights, and
2. a **fixing/projection curve** for the index cashflows.

This “two-curve minimum” shows up even in simple flow instruments (e.g., FRAs) once collateralization is widespread and term interbank rates are meaningfully different from overnight collateral rates.

### 33.4.2 Why the Curves Differ

Consider a vanilla interest rate swap where we receive a term floating index and pay fixed. Two curves are needed:

**The projection curve** forecasts what the floating index will be at each fixing date. This depends on the market’s forward expectations for that index (and, for some indices, may embed credit/liquidity effects and/or term premia).

**The discount curve** determines the present value weights applied to each cashflow. Under the clean collateralized idealization, that curve is built from (or consistent with) the collateral remuneration benchmark $c(t)$.

**Check (limiting case):** If the floating index is the same overnight-compounded index that defines $c(t)$ (i.e., an OIS-like structure), then discounting and projection collapse back toward a one-curve setup.

### 33.4.3 Numerical Illustration: Single-Curve vs. Multi-Curve

**Setup:**
- Notional: $100{,}000{,}000$
- Annual payments at $t=1,2$
- Pay fixed, receive floating

**Legacy single-curve (LIBOR discounting = LIBOR projection):**

Given discount factors:
- $P^L(0,1) = 0.9700$
- $P^L(0,2) = 0.9400$

Annuity: $A_L = 0.9700 + 0.9400 = 1.9100$

Floating-leg PV (single-curve identity): $PV_{\text{float}}^L = 1 - P^L(0,2) = 0.0600$

Par fixed rate: $K_L = 0.0600 / 1.9100 = 3.141\\%$

At $K = 3.141\\%$, the swap has zero PV in the single-curve world.

**Multi-curve (OIS discounting, separate projection):**

OIS discount factors:
- $P^{\text{OIS}}(0,1) = 0.9750$
- $P^{\text{OIS}}(0,2) = 0.9500$

OIS annuity: $A_{\text{OIS}} = 0.9750 + 0.9500 = 1.9250$

Projected floating rates (from a separate term rate curve):
- $L_1 = 3.20\\%$
- $L_2 = 3.50\\%$

Floating PV discounted on OIS:

$$PV_{\text{float}}^{\text{OIS}} = 0.0320 \times 0.9750 + 0.0350 \times 0.9500 = 0.0312 + 0.0333 = 0.0645$$

Fixed PV at $K = 3.141\\%$:

$$PV_{\text{fixed}}^{\text{OIS}} = 0.03141 \times 1.9250 = 0.0605$$

Swap PV (receive float, pay fixed):

$$PV^{\text{OIS}} = 0.0645 - 0.0605 = 0.0040$$

In dollars: $0.0040 \times \mathrm{USD}\\,100{,}000{,}000 = \mathrm{USD}\\,400{,}000$

**Key insight:** The same fixed rate that is "par" under legacy single-curve pricing is **not par** under multi-curve pricing. This difference—hundreds of thousands of dollars on a $\mathrm{USD}\\,100\text{mm}$ swap—is why the industry shifted to multi-curve frameworks.

> **Pitfall — “the same swap has two PVs”:** A swap can have different PVs under different discounting assumptions because you changed the funding/collateralization economics you are assuming. A single-curve setup implicitly uses the same curve for both projection and discounting; under collateralization, discounting should be consistent with the CSA collateral rate $c(t)$.
> **Quick check:** identify the CSA collateral remuneration benchmark and make sure the discount curve matches it (and that you haven’t accidentally changed the projection curve at the same time).

---

## 33.5 Multi-Currency CSAs and Cheapest-to-Deliver Collateral

### 33.5.1 CSA Optionality: Collateral Currency Is a Pricing Input

Some CSAs specify **more than one eligible collateral currency**. When that happens, “which curve discounts the trade” is no longer just “the OIS curve”—it depends on **which collateral currency is posted** (and sometimes on which currency is *cheapest to post* at the time).

This matters because the choice can be:
- **state-dependent** (rates, FX forwards/basis, and funding conditions change over time), and
- **exposure-dependent** (the party that is out-of-the-money posts collateral).

So multi-currency collateral can embed an option-like feature in the CSA (often called “cheapest-to-deliver collateral” in desk language). The detailed modeling of that optionality is out of scope here; the goal is to make the *discounting dependency* explicit and teachable.

### 33.5.2 Anchor: Discounting with Foreign-Currency Collateral (Perfect Collateralization)

In a two-currency setup:
- payoff is in **domestic** currency $x$: $X_T^x$
- cash collateral is remunerated in currency $y$ at rate $c^y(t)$
- $r^x(t)$ and $r^y(t)$ denote the (currency-specific) “risk-free” short rates used in the idealized pricing setup

Under perfect collateralization, a compact way to express the domestic-currency PV is:

$$
P_{X}^{x, C, y}(t)=\mathbb{E}_{t}^{x}\left[e^{-\int_{0}^{T}\left(r^{x}(s)+c^{y}(s)-r^{y}(s)\right) d s} X_{T}^{x}\right].
$$

**Corollary (same currency):** if the payoff and collateral currencies coincide $(x=y)$, the formula reduces to “discount at the collateral rate”:

$$
P_{X}^{x, C, y}(t)=\mathbb{E}_{t}^{x}\left[e^{-\int_{0}^{T} c^{x}(s) d s} X_{T}^{x}\right].
$$

### 33.5.3 Expand: Effective Discounting Rate and Sanity Checks

The anchor above is often summarized as an **effective domestic discount rate**

$$r_{\text{eff}}^{x|y}(t) := r^{x}(t) + \bigl(c^{y}(t)-r^{y}(t)\bigr).$$

Interpretation: start from domestic “risk-free” discounting in $x$, then **add the collateral spread in the collateral currency** $y$. This makes two important checks easy:

- **Check 1 (same currency):** $r_{\text{eff}}^{x|x}(t)=c^x(t)$.
- **Check 2 (collateral rate equals risk-free in $y$):** if $c^y(t)=r^y(t)$, then $r_{\text{eff}}^{x|y}(t)=r^x(t)$. In this idealized limit, the collateral currency choice becomes irrelevant for discounting.

These checks are not just “math niceties”: they tell you what must be true for slogans like “OIS everywhere” to be safe in your specific CSA.

**Check (toy numeric, hypothetical):** Suppose rates are flat and constant with $r^x=2.00\\%$, $r^y=1.00\\%$, and the collateral remuneration in $y$ is $c^y=1.30\\%$. Then

$$
r_{\text{eff}}^{x|y}=2.00\\%+(1.30\\%-1.00\\%)=2.30\\%.
$$

A domestic $x$-currency receive cashflow of $X_T^x=\mathrm{USD}\\,1{,}000{,}000$ at $T=1$ has PV $\approx e^{-0.023}\times 1{,}000{,}000=\mathrm{USD}\\,977{,}270$. If instead $c^y=r^y$, then $r_{\text{eff}}^{x|y}=2.00\\%$ and PV $\approx e^{-0.020}\times 1{,}000{,}000=\mathrm{USD}\\,980{,}199$. The $\sim \mathrm{USD}\\,2.9\text{k}$ difference comes purely from the collateral spread $c^y-r^y$.

### 33.5.4 Cheapest-to-Deliver (CTD) and Standardization (SCSA)

If a CSA allows multiple collateral currencies and the poster can switch, then “which $y$ applies” can become a **choice** rather than a fixed input. This makes valuation potentially non-linear/path-dependent because the optimal choice can change over time and across scenarios.

The **Standard CSA / SCSA** is intended to remove the optionality found in existing CSAs and to promote OIS discounting so that interest accrual on cash collateral aligns with discount rates. The SCSA amends collateral calculations so that derivatives exposures and their offsetting collateral positions are grouped by currency (silo) and evaluated independently.

> **Desk Reality — Multi-currency collateral breaks “one curve”:**
> **Common break:** one system prices with “USD OIS” while the CSA allows (or actually uses) EUR collateral remuneration, leading to PV and DV01 breaks versus a counterparty/clearing statement.
> **What to check:** eligible collateral currencies, the remuneration benchmark in each currency, and whether your system treats collateral currency as (i) fixed per netting set, (ii) scenario-dependent, or (iii) an explicit CTD option.

---

## 33.6 What Breaks When Collateralization Is Imperfect

### 33.6.1 Sources of Imperfection

The "clean" OIS discounting result assumes perfect collateralization: continuous VM, zero threshold, zero MTA, no settlement lag, no default. Real CSAs violate all of these:

**Threshold and MTA:** Leave residual unsecured exposure up to the threshold amount. If $H = \mathrm{USD}\\,5\text{m}$ and current MTM is $\mathrm{USD}\\,6.2\text{m}$, the counterparty posts $\mathrm{USD}\\,1.2\text{m}$ and $\mathrm{USD}\\,5\text{m}$ remains unsecured.

**Margining frequency (not continuous):** If VM is daily or weekly, MTM can move between calls, creating exposure during each margining interval.

**Margin settlement lag / MPOR:** Even with frequent calls, collateral can be “stale” because of settlement lags, disputes, and the time it takes to close out a defaulting counterparty. Residual exposure is then driven by MTM moves over the margin period of risk (MPOR).

**Counterparty default:** If the counterparty actually defaults, remaining exposure after collateral becomes a credit loss.

### 33.6.2 The "Clean + Adjustments" Framework

A practical way to organize valuation is to separate a **clean price** (the collateralized, funding-consistent value of promised cashflows) from **adjustments** that account for residual credit/funding effects when collateralization is imperfect.

$$\boxed{V_{\text{all-in}} \approx V_{\text{clean (OIS-discounted)}} - \text{CVA} + \text{DVA} + \text{(other adjustments)}}$$

where:
- **CVA (Credit Valuation Adjustment):** PV of expected loss from counterparty default
- **DVA (Debit Valuation Adjustment):** PV of expected loss to counterparty from our default
- **Other adjustments:** Effects of collateral imperfections, funding costs (FVA), capital costs (KVA)

This chapter focuses on the **clean** collateral-discounted logic and on how CSA terms move you away from the clean idealization. Chapter 34 develops the XVA objects and their modeling in more detail.

**Check (limiting case):** In the perfect-collateralization, no-default idealization, unsecured exposure is (approximately) zero and the adjustment layer vanishes, so $V_{\text{all-in}}\approx V_{\text{clean}}$. Non-zero CVA/DVA/FVA/KVA arise exactly when the “clean” assumptions fail (thresholds/MTAs, MPOR, default, funding frictions).

> **Desk Reality — “clean PV” vs “all-in PV”:**
> A common operating model is to compute a clean PV for promised cashflows using a collateral-consistent discount curve, and then compute valuation adjustments (CVA/FVA/etc.) in a separate layer.
> **Common break:** mixing discounting assumptions across the clean layer and the XVA layer (or double-counting collateral interest via both cashflows and discounting).
> **What to check:** which curve defines “clean discounting” for the netting set, and whether PAI/collateral interest is modeled as explicit cashflows or implicit via discounting.

### 33.6.3 Preview: The CVA Formula

Chapter 34 develops CVA in detail. Here we note only a common discrete-time approximation:

$$\boxed{\text{CVA} = \sum_{i=1}^{n} (1-R) \cdot q_i \cdot v_i}$$

where:
- $q_i$: risk-neutral probability of loss from counterparty default during interval $i$
- $v_i$: PV of expected net exposure (after collateral) at interval midpoint
- $R$: recovery rate

In realistic portfolios, the $v_i$ are portfolio-level objects (netting + collateral timing) and are often estimated via simulation; the point here is only the *shape* of the adjustment: expected discounted loss given default.

### 33.6.4 Hybrid Discounting for Partial Collateralization

With thresholds, MTAs, and margin timing frictions, the exact valuation can become **non-linear** (exposure-dependent) and you should be careful about statements like “just discount everything at OIS”.

At a minimum, you should recognize the mechanical fact:
- a positive threshold $H$ leaves **unsecured exposure up to $H$**, which is economically closer to an uncollateralized claim than to a perfectly collateralized one.

The practical implementation choice—explicit hybrid discounting versus “clean PV + adjustments”—is a modeling decision. This book’s approach is to keep the clean collateral-discounted PV as the anchor and treat the imperfections as inputs to Chapter 34’s adjustment framework.

### 33.6.5 Downgrade Triggers and Liquidity Risk

Many CSAs contain **downgrade triggers**—clauses that change collateral requirements if a counterparty's credit rating falls below a specified level.

**How downgrade triggers work:**

A typical clause might specify:
- If rated A or above: threshold = $\mathrm{USD}\\,50\text{m}$
- If rated BBB: threshold = $\mathrm{USD}\\,10\text{m}$
- If rated below investment grade: threshold = zero (full collateralization required)

When a downgrade occurs, the party must immediately post additional collateral equal to the threshold reduction. This can create severe **liquidity stress** precisely when the firm is already under pressure.

**Why this is dangerous:** a credit downgrade tends to coincide with worsening funding conditions and widening exposures, so downgrade triggers can create a feedback loop: *credit deterioration → more collateral required → less liquidity → more credit deterioration*.

**Historical example:** after several downgrades, AIG had posted more than \mathrm{USD}\\,40 billion in collateral as of November 2008.

**Implications for pricing:**

When valuing a portfolio with downgrade triggers:
1. **Scenario analysis:** Model collateral demands under rating downgrade scenarios
2. **Liquidity reserves:** Ensure funding is available for contingent collateral calls
3. **CVA sensitivity:** Recognize that exposure profiles change discontinuously at rating thresholds

---

## 33.7 Price Alignment Interest (PAI)

### 33.7.1 What Is PAI?

**Price Alignment Interest (PAI)** is interest on cash variation margin: when cash variation margin is transferred through the CCP from Party A to Party B, Party A earns interest and Party B pays interest on the funds transferred; this interest is known as price alignment interest.

In desk terms, PAI is the operational mechanism by which the collateral remuneration rate enters the trade's cashflows and aligns the economics of margining with the trade’s theoretical value.

For a cleared trade (or bilateral trade with daily settlement), PAI works as follows:
- Each day, the clearinghouse (or bilateral counterparty) calculates the change in MTM
- VM is exchanged to match this change
- **PAI is paid on the VM balance**, at the overnight rate specified by the margining/clearing setup

Without PAI, the party receiving VM would have "free money"—cash on which they pay nothing. PAI ensures this doesn't happen.

### 33.7.2 PAI Mechanics

**Daily PAI calculation:**

$$PAI_t = B_{t-1}^{\mathrm{VM}} \times r_{\text{overnight}} \times \frac{1}{360}$$

In this chapter’s examples, we assume an ACT/360 day count so that a 1-day accrual factor is $1/360$.

**Example:**
- VM balance at end of day $t-1$: $+\mathrm{USD}\\,10{,}000{,}000$ (we hold collateral)
- Overnight rate: 5.25%
- Day count: ACT/360

$$PAI_t = \mathrm{USD}\\,10{,}000{,}000 \times 0.0525 \times \frac{1}{360} = \mathrm{USD}\\,1{,}458.33$$

We **pay** this amount to the counterparty (who posted the VM).

**Check (sign and day-count):**
- If you record a “VM balance” with the sign convention $+$ = cash VM held by us, then PAI is a cash **outflow** when VM is positive (we pay interest to the poster) and a cash **inflow** when VM is negative (we receive interest on posted VM).
- The accrual factor should use the actual day count for the accrual window. For example, if VM is held over 3 calendar days under ACT/360, use $3/360$ rather than $1/360$.

### 33.7.3 PAI vs. Traditional Coupon Accrual

PAI fundamentally changes how swap economics work compared to legacy bilateral trades:

| Aspect | Traditional (No VM/PAI) | Modern (Daily VM + PAI) |
|--------|------------------------|------------------------|
| **Funding of MTM** | Party with positive MTM funds at own rate | VM provides funding at overnight rate |
| **Interest on collateral** | May differ from trade economics | PAI aligns collateral interest with discounting |
| **Effective discounting** | Depends on party's funding | Enforced as overnight rate |

**The key insight:** PAI operationalizes OIS discounting. When you receive PAI at the overnight rate on your VM, the economics are identical to discounting at that rate. This is not coincidence—it's the mechanism that makes the no-arbitrage argument work in practice.

### 33.7.4 PAI and P&L

For traders, PAI shows up as a daily P&L line item:

$$\text{Daily PnL} = \Delta\text{MTM} + \text{Coupon received} - \text{Coupon paid} + \text{PAI received} - \text{PAI paid}$$

A common mistake is ignoring PAI when computing carry. On a large swap book, daily PAI can be substantial:

**Example:** A desk with $\mathrm{USD}\\,500\text{mm}$ net positive VM at 5% overnight rate earns:

$$\text{Annual PAI} \approx \mathrm{USD}\\,500{,}000{,}000 \times 0.05 = \mathrm{USD}\\,25{,}000{,}000$$

This is real money that affects the economics of holding positions.

> **Desk Reality: PAI Reconciliation**
>
> One of the most common sources of P&L breaks on swap desks is PAI miscalculation or misreconciliation:
>
> - **Rate mismatch:** Your system uses yesterday's SOFR; the clearinghouse uses today's
> - **Day count errors:** Using 365 vs. 360
> - **Balance timing:** EOD vs. SOD VM balance
> - **Settlement lag:** PAI accrues on settled VM, not called VM
>
> If your daily P&L is breaking by small amounts that compound over time, check PAI first.

---

## 33.8 Risk Measurement for OIS-Discounted Portfolios

### 33.8.1 OIS Discount-Curve DV01 (What Is Being Bumped?)

For an OIS-discounted portfolio, a natural first risk scalar is the sensitivity of PV to the **OIS discount curve**.

**Definition (this book’s convention):**
- **Bump object:** the OIS *discount* curve (e.g., a parallel shift of continuously-compounded OIS zero rates `y^{OIS}(0,T)`)
- **Bump size:** 1 bp $=10^{-4}$
- **Units:** currency per 1 bp for the stated notional
- **Sign convention:** $DV01 := PV(\text{OIS rates down }1\text{bp})-PV(\text{base})$.

**What is held fixed?** In this definition, the projected cashflows are held fixed (same fixings and same projection curve); only the *discount factors* change.

For deterministic cashflows $CF(t_i)$, a first-order approximation under continuous compounding is:

$$DV01 \\;\approx\\; 10^{-4}\sum_i t_i \cdot P^{\text{OIS}}(0,t_i) \cdot CF(t_i).$$

**Sign check:** for a positive cashflow stream, OIS rates down $\Rightarrow$ discount factors up $\Rightarrow$ PV up, so $DV01\gt 0$.

**Check (toy magnitude):** For a single receive cashflow of $+\mathrm{USD}\\,1{,}000{,}000$ at $t=5$y with $P^{\text{OIS}}(0,5)=0.90$, the approximation gives

$$
DV01 \approx 10^{-4}\times 5 \times 0.90 \times 1{,}000{,}000 \approx \mathrm{USD}\\,450 \text{ per 1bp}.
$$

If you compute a DV01 that is 10× or 100× larger for this kind of toy stream, suspect a units (bp vs %) or time (years vs days) mismatch.

### 33.8.2 Discounting Basis Exposure

A useful risk metric is the **discounting basis** PV difference:

$$\Delta PV_{\text{basis}} = PV_{\text{OIS disc}} - PV_{\text{legacy disc}}$$

When the discount curves differ materially, this basis can be a meaningful P&L and risk dimension.

### 33.8.3 Collateral Terms as Risk Factors

The CSA parameters are themselves risk factors:
- **Threshold:** Higher threshold increases CVA (more unsecured exposure)
- **Margin frequency:** Less frequent margining increases expected exposure
- **Cure period:** Longer MPOR increases potential exposure at default
- **Collateral rate benchmark:** If the CSA rate changes (e.g., Fed Funds to SOFR), discounting changes

These effects must be captured in stress testing and risk limits.

---

## 33.9 Worked Examples

### Example 33.1: Clean vs. Collateralized PV (Single Cashflow)

**Task:** Price a deterministic cashflow under (i) a "risk-free" curve $r$ and (ii) a collateral rate $c$.

**Given:**
- Cashflow: $\text{CF}(1) = \mathrm{USD}\\,1{,}000{,}000$ at $t = 1$
- Discount factors: $P^r(0,1) = 0.9700$, $P^c(0,1) = 0.9750$

**PV under $r$:**

$$PV^r = \mathrm{USD}\\,1{,}000{,}000 \times 0.9700 = \mathrm{USD}\\,970{,}000$$

**PV under collateral rate $c$:**

$$PV^c = \mathrm{USD}\\,1{,}000{,}000 \times 0.9750 = \mathrm{USD}\\,975{,}000$$

**Difference:**

$$\Delta PV = \mathrm{USD}\\,975{,}000 - \mathrm{USD}\\,970{,}000 = \mathrm{USD}\\,5{,}000$$

**Sanity check:** Higher discount factor → higher PV for a positive cashflow. ✓

---

### Example 33.2: Collateralized IRS PV + OIS Discount-Curve DV01 (Worked Example)

**Context**
- What is being priced/measured? A 2Y fixed-for-float IRS priced under OIS discounting.
- Why it matters on the desk: the discount curve choice moves PV and creates a distinct “OIS discount DV01” risk, even when the floating index is not OIS.

**Timeline (Concrete Dates)**
- Trade date: 2026-02-17
- Effective date: 2026-02-19
- Payment dates: 2027-02-19 and 2028-02-19
- Day count / accrual: assume each year accrues $\Delta=1.0$ (toy simplification)

**Inputs**
- Notional: $N=\mathrm{USD}\\,100{,}000{,}000$
- Pay fixed at $K=3.141\\%$; receive floating with projected annual rates $L_1=3.20\\%$, $L_2=3.50\\%$
- OIS discount factors: $P^{\text{OIS}}(0,1)=0.9750$, $P^{\text{OIS}}(0,2)=0.9500$
- (For comparison only) a legacy discount curve: $P^{L}(0,1)=0.9700$, $P^{L}(0,2)=0.9400$

**Outputs (What You Produce)**
- Clean PV under OIS discounting (receive float, pay fixed)
- OIS discount-curve $DV01$: bump object = OIS discount curve, bump size = 1bp, cashflows held fixed

**Step-by-step**
1. **Translate rates to cashflows** (toy $\Delta=1$):
   - Fixed cashflow at $t_i$: $CF^{\text{fixed}}_i = N\cdot K\cdot \Delta$
   - Float cashflow at $t_i$: $CF^{\text{float}}_i = N\cdot L_i\cdot \Delta$
2. **Compute PV on the OIS curve**: $PV = \sum_i P^{\text{OIS}}(0,t_i)\\,\bigl(CF^{\text{float}}_i-CF^{\text{fixed}}_i\bigr)$.
3. **Compute discount-curve DV01** (rates down 1bp, forwards held fixed) using the first-order approximation from Section 33.8: $DV01 \approx 10^{-4}\sum_i t_i\\,P^{\text{OIS}}(0,t_i)\\,\bigl(CF^{\text{float}}_i-CF^{\text{fixed}}_i\bigr)$.

**Cashflows**
| Date | Net cashflow | Explanation |
|---|---:|---|
| 2027-02-19 | $N(L_1-K)\Delta = \mathrm{USD}\\,59{,}000$ | receive float $-$ pay fixed |
| 2028-02-19 | $N(L_2-K)\Delta = \mathrm{USD}\\,359{,}000$ | receive float $-$ pay fixed |

**PV**

$$
PV \approx 0.9750\times 59{,}000 + 0.9500\times 359{,}000
= 57{,}525 + 341{,}050 = \mathrm{USD}\\,398{,}575.
$$

**OIS discount-curve DV01**

$$
DV01 \approx 10^{-4}\left(1\times 0.9750\times 59{,}000 + 2\times 0.9500\times 359{,}000\right)
= 10^{-4}\left(57{,}525 + 682{,}100\right)=\mathrm{USD}\\,73.96 \approx \mathrm{USD}\\,74\text{ per bp}.
$$

**P&L / Risk Interpretation**
- If the OIS discount curve rallies by 1bp (rates down) and projected floating cashflows are unchanged, PV increases by about $\mathrm{USD}\\,74$.
- This is a **discount-curve** sensitivity. A “total rates DV01” that also moves the projection curve is a different bump design (see Chapter 22).

**Sanity Checks**
- Units check: PV is in dollars; DV01 is dollars per 1bp.
- Sign check: rates down $\Rightarrow$ discount factors up $\Rightarrow$ PV up for positive net cashflows, so $DV01\gt 0$.
- Limit check: if $L_i=K$ for all $i$, net cashflows are 0, so PV and discount-curve DV01 are 0.

**Debug Checklist (When Your Result Looks Wrong)**
- Did you bump the **discount** curve (OIS) or the **projection** curve (index forwards)?
- Are you holding projected cashflows fixed when computing discount-curve DV01?
- Did you mix up pay/receive direction (net cashflow sign)?
- Are discount factors consistent with the stated curve and compounding assumption?

---

### Example 33.3: Discount-Curve DV01 of a Fixed Leg (Toy)

**Setup:**
- Fixed-leg annuity, $\mathcal{N} = \mathrm{USD}\\,100{,}000{,}000$, $K = 3.50\\%$
- Annual payments at $t = 1, 2, 3, 4, 5$

**Base OIS discount factors:**
- $P_1 = 0.9750$, $P_2 = 0.9500$, $P_3 = 0.9250$, $P_4 = 0.9000$, $P_5 = 0.8750$

**Base annuity:**

$$A = 0.975 + 0.950 + 0.925 + 0.900 + 0.875 = 4.625$$

**Base PV:**

$$PV_{\text{fixed}} = \mathrm{USD}\\,100{,}000{,}000 \times 0.035 \times 4.625 = \mathrm{USD}\\,16{,}187{,}500$$

**Discount-curve DV01 (rates down 1bp, fixed cashflows):**

Here the cashflows are deterministic and only the discount curve is bumped. Using the first-order approximation from Section 33.8:

$$DV01 \approx 10^{-4}\sum_{i=1}^{5} t_i \cdot P_i \cdot (N K).$$

Compute the time-weighted discount-factor sum:

$$\sum_{i=1}^{5} t_i P_i = 1(0.975)+2(0.950)+3(0.925)+4(0.900)+5(0.875)=13.625.$$

And $N K = \mathrm{USD}\\,100{,}000{,}000 \times 0.035 = \mathrm{USD}\\,3{,}500{,}000$, so:

$$DV01 \approx 10^{-4}\times 13.625 \times \mathrm{USD}\\,3{,}500{,}000 = \mathrm{USD}\\,4{,}768.75 \text{ per bp}.$$

**Sign check:** OIS rates down $\Rightarrow$ PV up for a receive-fixed leg, so $DV01\gt 0$.

---

### Example 33.4: Collateral Frequency Effect (Daily vs. Weekly)

**MTM values over one week (in USD millions):**

| Day | MTM |
|-----|-----|
| 0 | 0.0 |
| 1 | 1.0 |
| 2 | 0.8 |
| 3 | 1.2 |
| 4 | 0.4 |
| 5 | 0.6 |

**Daily VM (collateral = MTM at end of each day):**

Exposure during day $d$ (before end-of-day call): $E_d = \max(V_d - C_{d-1}, 0)$

| Day | Exposure |
|-----|----------|
| 1 | $\max(1.0 - 0.0, 0) = 1.0$ |
| 2 | $\max(0.8 - 1.0, 0) = 0.0$ |
| 3 | $\max(1.2 - 0.8, 0) = 0.4$ |
| 4 | $\max(0.4 - 1.2, 0) = 0.0$ |
| 5 | $\max(0.6 - 0.4, 0) = 0.2$ |

**Max daily-margin exposure:** 1.0 (USD million)

**Weekly VM (collateral stays at $C_0 = 0$ all week):**

$$E_d^{\text{weekly}} = \max(V_d, 0) = V_d^+$$

**Max weekly-margin exposure:** 1.2 (USD million; Day 3)

**Conclusion:** Less frequent margining increases maximum residual exposure (1.0 mm → 1.2 mm).

---

### Example 33.5: Threshold and MTA Effect

**Given:**
- Current MTM: $V = \mathrm{USD}\\,5.0\text{m}$
- Threshold: $H = \mathrm{USD}\\,2.0\text{m}$
- MTA: $\mathrm{USD}\\,0.5\text{m}$

**Collateral required:**

$$\text{Call} = \max(V - H, 0) = \max(5.0 - 2.0, 0) = \mathrm{USD}\\,3.0\text{m}$$

Since $\mathrm{USD}\\,3.0\text{m} \gt \text{MTA}$, the transfer occurs.

**Residual exposure:**

$$E = \max(V - C, 0) = \max(5.0 - 3.0, 0) = \mathrm{USD}\\,2.0\text{m}$$

The threshold amount ($2.0\text{m}$) remains unsecured—this is credit exposure.

**Small-move case (MTA blocks transfer):**

If $V = \mathrm{USD}\\,2.3\text{m}$:
- Call $= \max(2.3 - 2.0, 0) = \mathrm{USD}\\,0.3\text{m} \lt \text{MTA}$
- No transfer occurs
- Exposure $\approx \mathrm{USD}\\,2.3\text{m}$

---

### Example 33.6: Margin Lag (Settlement Lag)

**Setup:**
- Daily VM with one-day settlement lag: collateral today = yesterday's MTM
- MTM path (in USD millions): Day 0: 0, Day 1: 5, Day 2: 2, Day 3: 6

**Collateral during day $d$:** $C_d = V_{d-1}$

**Exposure:** $E_d = \max(V_d - V_{d-1}, 0)$

| Day | Exposure |
|-----|----------|
| 1 | $\max(5 - 0, 0) = 5$ |
| 2 | $\max(2 - 5, 0) = 0$ |
| 3 | $\max(6 - 2, 0) = 4$ |

**Max exposure:** 5 (USD million)

This illustrates the settlement-lag / cure-period logic: collateral reflects stale MTM, leaving exposure equal to recent MTM increases.

---

### Example 33.7: CVA Calculation (Toy Discrete Sum)

**Given:**
- Recovery: $R = 40\\%$ → $(1-R) = 0.6$
- OIS discount factors: $P(0,1) = 0.97$, $P(0,2) = 0.94$, $P(0,3) = 0.90$
- Expected exposures after collateral: $EE(1) = \mathrm{USD}\\,2.0\text{m}$, $EE(2) = \mathrm{USD}\\,1.5\text{m}$, $EE(3) = \mathrm{USD}\\,1.0\text{m}$
- Default probabilities: $\Delta PD_1 = 1.0\\%$, $\Delta PD_2 = 1.2\\%$, $\Delta PD_3 = 1.5\\%$

**Year 1:**

$$0.6 \times 0.97 \times \mathrm{USD}\\,2{,}000{,}000 \times 0.01 = \mathrm{USD}\\,11{,}640$$

**Year 2:**

$$0.6 \times 0.94 \times \mathrm{USD}\\,1{,}500{,}000 \times 0.012 = \mathrm{USD}\\,10{,}152$$

**Year 3:**

$$0.6 \times 0.90 \times \mathrm{USD}\\,1{,}000{,}000 \times 0.015 = \mathrm{USD}\\,8{,}100$$

**Total CVA:**

$$\boxed{\text{CVA} = \mathrm{USD}\\,11{,}640 + \mathrm{USD}\\,10{,}152 + \mathrm{USD}\\,8{,}100 = \mathrm{USD}\\,29{,}892}$$

---

### Example 33.8: Collateral Rate Change

**Cashflows:** Receive $\mathrm{USD}\\,1{,}000{,}000$ at each of $t = 1, 2, 3$

**Under collateral rate $c_1$ (lower rate, higher DFs):**
- $P^{c_1}(0,1) = 0.975$, $P^{c_1}(0,2) = 0.950$, $P^{c_1}(0,3) = 0.925$

$$PV^{c_1} = \mathrm{USD}\\,1{,}000{,}000 \times 2.850 = \mathrm{USD}\\,2{,}850{,}000$$

**Under collateral rate $c_2$ (higher rate, lower DFs):**
- $P^{c_2}(0,1) = 0.970$, $P^{c_2}(0,2) = 0.940$, $P^{c_2}(0,3) = 0.910$

$$PV^{c_2} = \mathrm{USD}\\,1{,}000{,}000 \times 2.820 = \mathrm{USD}\\,2{,}820{,}000$$

**Impact:**

$$\Delta PV = \mathrm{USD}\\,2{,}820{,}000 - \mathrm{USD}\\,2{,}850{,}000 = -\mathrm{USD}\\,30{,}000$$

Higher collateral rate → lower PV for receiving cashflows.

---

### Example 33.9: Multi-Currency CSA (USD vs. EUR Collateral)

**Setup:**
- USD derivative pays $\mathrm{USD}\\,10{,}000{,}000$ in 1 year
- CSA allows USD or EUR collateral
- USD OIS: $P^{\text{USD}}(0,1) = 0.9700$ (~3.09% rate)
- EUR OIS: $P^{\text{EUR}}(0,1) = 0.9850$ (~1.52% rate)
- Spot EUR/USD: $S_0 = 1.10$
- 1-year forward: $F = 1.0833$

**Option A: USD Collateral**

$$PV^{\text{USD}} = \mathrm{USD}\\,10{,}000{,}000 \times 0.9700 = \mathrm{USD}\\,9{,}700{,}000$$

**Option B: EUR Collateral (with CIP holding)**

Effective USD discount factor when posting EUR:

$$P^{\text{EUR→USD}}(0,1) \approx P^{\text{EUR}}(0,1) \times \frac{F}{S_0} = 0.9850 \times \frac{1.0833}{1.10} = 0.9700$$

With CIP holding, the two are approximately equal.

**With cross-currency basis of -25bp (EUR cheaper):**

$$P^{\text{EUR→USD, adj}}(0,1) \approx 0.9700 \times e^{0.0025} = 0.9724$$

$$PV^{\text{EUR, adj}} = \mathrm{USD}\\,10{,}000{,}000 \times 0.9724 = \mathrm{USD}\\,9{,}724{,}000$$

**Advantage to posting EUR:** $\mathrm{USD}\\,24{,}000$

---

### Example 33.10: Self-Financing Replication Walkthrough

**Setup:**
- Bank A sells derivative paying $\mathrm{USD}\\,1{,}000{,}000$ to counterparty B in 1 year
- Perfect collateralization at $c = 2.50\\%$ (continuous)
- A hedges by buying a zero-coupon bond

**Step 1: A's hedge cost**

$$\text{Bond cost} = \mathrm{USD}\\,1{,}000{,}000 \times e^{-0.025} = \mathrm{USD}\\,975{,}309.91$$

**Step 2: Collateral flows**

At inception:
- Derivative value to A is $-V(0)$ (liability)
- A posts collateral $|V(0)|$ to B
- B pays premium $P$ to A

For A to be self-financing: $P - C(0) = \text{hedge cost}$

**Step 3: Collateral accrual**

If collateral rate = hedge funding rate = 2.50%, the funding is self-financing.

**Step 4: Terminal flows at $t = 1$**
- A receives $\mathrm{USD}\\,1{,}000{,}000$ from bond
- A pays $\mathrm{USD}\\,1{,}000{,}000$ to B
- A receives back collateral (including interest)
- Net to A: zero

**No-arbitrage price:**

$$\boxed{V(0) = \mathrm{USD}\\,1{,}000{,}000 \times e^{-0.025} = \mathrm{USD}\\,975{,}309.91}$$

---

### Example 33.11: Domestic PV with Foreign-Currency Collateral (Toy)

This example illustrates the `r^x + c^y - r^y` discounting identity from Section 33.5 in a simple constant-rate toy world.

**Setup (hypothetical, continuous compounding):**
- Payoff currency: USD ($x=\text{USD}$)
- Receive $X_T^{\text{USD}}=\mathrm{USD}\\,1{,}000{,}000$ at $T=5$ years
- Domestic rate: $r^{\text{USD}}=3.00\\%$
- Consider two collateral choices:
  1. USD collateral: $y=\text{USD}$, $c^{\text{USD}}=3.00\\%$
  2. EUR collateral: $y=\text{EUR}$, $r^{\text{EUR}}=1.00\\%$, $c^{\text{EUR}}=0.70\\%$

**USD collateral (same-currency corollary):**

$$
PV^{\text{USD}|\text{USD}} = 1{,}000{,}000 \times e^{-c^{\text{USD}}T}
= 1{,}000{,}000 \times e^{-0.03\times 5} = \mathrm{USD}\\,860{,}708.
$$

**EUR collateral (foreign-currency collateral):**
Effective USD discount rate:

$$
r_{\text{eff}}^{\text{USD}|\text{EUR}} = r^{\text{USD}} + c^{\text{EUR}} - r^{\text{EUR}}
= 3.00\\% + 0.70\\% - 1.00\\% = 2.70\\%.
$$

So:

$$PV^{\text{USD}|\text{EUR}} = 1{,}000{,}000 \times e^{-0.027\times 5} = \mathrm{USD}\\,873{,}715.$$

**PV difference:**

$$\Delta PV = \mathrm{USD}\\,873{,}715 - \mathrm{USD}\\,860{,}708 = \mathrm{USD}\\,13{,}007.$$

**Checks:**
- If $c^{\text{EUR}}=r^{\text{EUR}}$, then $r_{\text{eff}}^{\text{USD}|\text{EUR}}=r^{\text{USD}}$ and the PVs coincide.
- Discount factors stay $\lt 1$ because the effective discount rate is positive.

---

### Example 33.12: Downgrade Trigger Impact

**Setup:**
- Bilateral swap portfolio MTM: $+\mathrm{USD}\\,80\text{mm}$ to us
- Current CSA threshold: $\mathrm{USD}\\,50\text{mm}$ (counterparty rated A)
- Collateral currently held: $\mathrm{USD}\\,80\text{mm}-\mathrm{USD}\\,50\text{mm}=\mathrm{USD}\\,30\text{mm}$
- Downgrade trigger: if the counterparty falls to BBB, threshold becomes $\mathrm{USD}\\,10\text{mm}$

**Before downgrade:**
- Unsecured exposure: $\mathrm{USD}\\,50\text{mm}$ (the threshold)
- CVA based on this exposure level

**After downgrade to BBB:**
- New threshold: $\mathrm{USD}\\,10\text{mm}$
- Required collateral: $\mathrm{USD}\\,80\text{mm}-\mathrm{USD}\\,10\text{mm}=\mathrm{USD}\\,70\text{mm}$
- Collateral call: $\mathrm{USD}\\,70\text{mm}-\mathrm{USD}\\,30\text{mm}=\mathrm{USD}\\,40\text{mm}$ immediate demand

**Liquidity impact on counterparty:**
The counterparty must post an additional $\mathrm{USD}\\,40\text{mm}$ immediately. If they cannot:
- Potential margin dispute
- Possible default trigger
- Our exposure increases if they fail to post

**CVA impact:**
- Before: $\mathrm{USD}\\,50\text{mm}$ unsecured $\times$ (expected default loss)
- After (if posted): $\mathrm{USD}\\,10\text{mm}$ unsecured $\times$ (higher expected default loss due to lower rating)

The net CVA may increase even with lower threshold because default probability increased.

---

### Example 33.13: Daily PAI Calculation

**Setup:**
- Cleared USD swap
- VM balance (we hold): $+\mathrm{USD}\\,25{,}000{,}000$
- Assume overnight rate = 5.30% (illustrative)
- Day count: ACT/360

**Daily PAI we owe:**

$$\text{PAI} = \mathrm{USD}\\,25{,}000{,}000 \times 0.0530 \times \frac{1}{360} = \mathrm{USD}\\,3{,}680.56$$

**Monthly PAI (30 days):**

$$\text{PAI}_{\text{month}} \approx \mathrm{USD}\\,3{,}680.56 \times 30 = \mathrm{USD}\\,110{,}417$$

**Annual PAI:**

$$\text{PAI}_{\text{annual}} \approx \mathrm{USD}\\,25{,}000{,}000 \times 0.0530 = \mathrm{USD}\\,1{,}325{,}000$$

**P&L implication:** If we're holding $\mathrm{USD}\\,25\text{mm}$ positive MTM on a receiver swap, we earn PAI. But this PAI is offset by the fact that we're implicitly “short” rates on our discount curve exposure. The economics balance out—which is exactly the point of PAI.

---

## 33.10 Practical Notes

### 33.10.1 CSA Terms You Must Know Before Discounting

Before saying "OIS discounting," verify:

1. **Collateral currency** — USD/EUR/GBP/other
2. **Collateral remuneration rate** — which benchmark (Fed funds/SOFR/€STR/SONIA) and any spreads
3. **Margin frequency** — daily/weekly; intraday calls?
4. **Threshold** — amount and whether one-way or two-way
5. **MTA** — minimum transfer amount
6. **Cure period / MPOR** — including dispute time
7. **Eligible collateral** — cash vs. securities, haircuts
8. **Rehypothecation rights** — can received collateral fund hedges?
9. **Close-out convention** — mid-market vs. replacement cost
10. **Downgrade triggers** — how thresholds change with ratings

### 33.10.2 Common Pitfalls

**Mixing up discount and projection curves:** Discounting affects PV weights; projection affects floating cashflows. These are different economic questions.

**Saying "OIS discounting" without specifying details:** Which overnight rate? What if the CSA allows multiple currencies?

**Ignoring thresholds and margin lags:** These create residual unsecured exposure. A "USD 10m threshold CSA" leaves up to USD 10m unsecured—that's credit risk.

**Confusing CVA with "just a spread":** CVA is an expectation of discounted loss depending on exposure profiles, collateral terms, and default probabilities—not a simple spread.

**Ignoring PAI in carry calculations:** PAI is real cash flow that affects portfolio economics.

**Underestimating downgrade trigger risk:** Contingent collateral demands can create liquidity crises precisely when least affordable.

### 33.10.3 Verification Tests

**Repricing checks:**
- Rebuild curves → benchmark OIS instruments should reprice to near-zero
- Legacy vs. OIS PV differences should match expected basis PV

**Unit checks:**
- 1 bp = 0.0001 (decimal)
- PV units = currency units

**Sign checks:**
- Discount rates up → PV of positive cashflows down

**Limiting cases:**
- Perfect collateral + no default → clean price = OIS-discounted price
- Zero exposure → CVA = 0

---

## Summary

The shift from LIBOR discounting to OIS discounting reflects a fundamental economic truth: **the discount rate must match the funding rate of the replicating hedge**. For collateralized derivatives, that funding rate is the collateral remuneration rate—typically an overnight rate like Fed funds, SOFR, or €STR.

**Key points:**

1. **Collateral remuneration determines discounting.** If cash collateral earns overnight rates, discount at overnight rates. This is a no-arbitrage requirement, not a convention.

2. **The LIBOR-OIS spread blew out during the crisis.** A spread of 275 bp made ignoring the discounting basis impossible.

3. **Multi-curve pricing is now standard.** OIS for discounting, term rate curves for projection. The curves serve different economic functions.

4. **Imperfect collateral creates exposure.** Thresholds, MTAs, and margin lags leave residual unsecured exposure requiring CVA.

5. **The cure period (MPOR) matters.** Even with zero threshold, settlement lag means collateral at default reflects stale MTM.

6. **CSA terms are product-defining.** You cannot choose a discount curve without knowing collateral currency, remuneration rate, threshold, and rehypothecation rights.

7. **Multi-currency CSAs create CTD optionality.** The collateral poster chooses the cheapest currency, affecting the effective discount rate.

8. **Downgrade triggers add liquidity risk.** Collateral demands can spike precisely when a counterparty is least able to meet them.

9. **PAI operationalizes the funding argument.** Daily interest on VM ensures the economics match the theoretical discounting.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Credit Support Annex (CSA)** | Annex to ISDA Master Agreement specifying collateral terms | Determines which discount rate applies |
| **Variation Margin (VM)** | Collateral exchanged daily to reflect MTM changes | Reduces credit exposure; ties funding to collateral rate |
| **Collateral Remuneration Rate $c(t)$** | Rate paid on posted cash collateral | Directly determines the appropriate discount rate |
| **OIS (Overnight Indexed Swap)** | Fixed vs. geometric average of overnight rates | Market instrument for extracting risk-free term structure |
| **OIS Discounting** | Using OIS curve to discount collateralized derivatives | Standard post-crisis practice; reflects actual funding |
| **Projection Curve** | Curve for forecasting floating rate payments | Separated from discount curve in multi-curve framework |
| **Threshold** | MTM level below which no VM is required | Creates residual unsecured exposure |
| **Margin Period of Risk (MPOR)** | Time between last collateralization and close-out | Exposure can move even with zero threshold |
| **Rehypothecation** | Using received collateral to meet other collateral demands | Determines whether collateral actually funds hedges |
| **CTD Collateral** | Cheapest-to-deliver collateral in multi-currency CSAs | Creates optionality affecting discount rate |
| **Downgrade Trigger** | CSA clause changing terms when credit rating falls | Creates liquidity risk at worst times |
| **Price Alignment Interest (PAI)** | Daily interest paid on variation margin | Operationalizes OIS discounting in practice |
| **CVA** | PV of expected loss from counterparty default | Adjustment needed when collateral is imperfect |

---

## Notation

| Symbol | Definition |
|--------|------------|
| $V(t)$ | MTM value of derivative to us at time $t$ |
| $C(t)$ | Collateral held by us at time $t$ |
| $c(t)$ | Collateral remuneration rate |
| $P^{\text{OIS}}(0,t)$ | OIS discount factor to time $t$ |
| $r^{x}(t), r^{y}(t)$ | “Risk-free” short rates in currencies $x$ and $y$ (model inputs) |
| $c^{y}(t)$ | Collateral remuneration rate when collateral is in currency $y$ |
| $r_{\text{eff}}^{x|y}(t)$ | Effective discount rate in $x$ given collateral currency $y$: $r^{x}(t)+c^{y}(t)-r^{y}(t)$ |
| $H$ | CSA threshold |
| $R$ | Recovery rate |
| $q_i$ | Default probability for interval $i$ |
| $v_i$ | PV of expected exposure at interval $i$ |
| $\beta_c(t)$ | Collateral account numeraire |
| $DV01$ | $PV(\text{rates down }1\text{bp})-PV(\text{base})$ for the stated bump object |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a CSA? | Credit Support Annex to an ISDA Master Agreement; specifies collateral requirements for bilaterally cleared OTC trades |
| 2 | What is variation margin (VM)? | Collateral exchanged daily to reflect MTM changes between counterparties |
| 3 | What is the collateral remuneration rate? | The contractual rate paid on posted cash collateral |
| 4 | Why does the CSA matter for the discount curve? | Because the CSA specifies how cash collateral accrues (the $c(t)$ you should be discounting at under the clean idealization) |
| 5 | What is an OIS? | A swap exchanging a fixed rate for a compounded overnight rate over the period |
| 6 | What is “OIS discounting” in one sentence? | Using a discount curve consistent with the CSA’s overnight collateral remuneration benchmark |
| 7 | State the no-arbitrage argument for OIS discounting in one sentence | If your hedge is funded at the collateral rate, you must discount at that rate—otherwise competitors funding at that rate can arbitrage you |
| 8 | What is the discount curve used for? | Determining PV weights (time value of money for the replicating hedge) |
| 9 | What is the projection curve used for? | Forecasting floating cashflows (what rate will actually set) |
| 10 | Why do you need multi-curve pricing? | Payoffs reference one index (projection), but collateralization ties funding/discounting to another rate (discounting) |
| 11 | What is a CSA threshold? | MTM level below which VM is not required |
| 12 | What is MTA? | Minimum transfer amount; avoids small collateral transfers |
| 13 | What is MPOR? | The time between the last effective collateral update and close-out/settlement; drives gap risk |
| 14 | Why can exposure exist even with VM and zero threshold? | MTM can move during MPOR and before collateral is settled/updated |
| 15 | What is rehypothecation? | Using collateral posted by one counterparty to fund positions with another |
| 16 | What operational detail can break “discount at $c(t)$”? | If collateral is segregated (not reusable) or timing/settlement differs, the hedge may not be funded at $c(t)$ |
| 17 | Give the basic CVA approximation | $\text{CVA} \approx \sum(1-R)q_i v_i$ |
| 18 | What is wrong-way risk? | When exposure increases as counterparty credit deteriorates |
| 19 | Under perfect collateralization, how does foreign-currency collateral enter discounting? | Effective domestic discount rate: $r^{x}+c^{y}-r^{y}$ (and if $x=y$, discount at $c^x$) |
| 20 | What must you verify before choosing a discount curve? | Collateral currency, remuneration benchmark $c(t)$, threshold/MTA, timing (settlement/MPOR), and reuse/segregation |
| 21 | What is Price Alignment Interest (PAI)? | Daily interest payment on variation margin that aligns collateral economics with derivative discounting |
| 22 | How is daily PAI calculated? | $\text{PAI} = \text{VM Balance} \times r_{\text{overnight}} \times \text{day fraction}$ |
| 23 | What is a downgrade trigger in a CSA? | A clause that reduces thresholds (increases collateral requirements) if a party's credit rating falls |
| 24 | Why are downgrade triggers dangerous for liquidity? | They demand more collateral precisely when the firm is under financial stress and least able to fund it |
| 25 | What is CTD collateral optionality? | The right to post collateral in the cheapest available currency under a multi-currency CSA |

---

## Mini Problem Set

1. (Concept) Define CSA, VM, IM, threshold, and MTA.
2. (Concept) List three assumptions under which “discount at the collateral rate $c(t)$” is a clean no-arbitrage result.
3. (Compute) A cashflow of $\mathrm{USD}\\,2{,}000{,}000$ is paid in 2 years. If $P^{\text{OIS}}(0,2)=0.94$, compute PV.
4. (Compute) Using Example 33.2, compute (i) the clean PV and (ii) the OIS discount-curve $DV01$ (rates down 1bp, forwards held fixed).
5. (Compute) With $V=\mathrm{USD}\\,5.0\text{m}$, threshold $H=\mathrm{USD}\\,2.0\text{m}$, and MTA $=\mathrm{USD}\\,0.5\text{m}$, compute the collateral call and the residual unsecured exposure.
6. (Concept) Explain why “two curves” are required when the floating index differs from the collateral remuneration benchmark.
7. (Compute) Using the foreign-currency collateral toy in Example 33.11, compute $r_{\text{eff}}^{\text{USD}|\text{EUR}}$ and the PV of a $\mathrm{USD}\\,1{,}000{,}000$ cashflow at $T=5$. Compare to the USD-collateral PV.
8. (Compute) Calculate daily PAI on a VM balance of $\mathrm{USD}\\,50\text{mm}$ when the overnight rate is 4.80% (ACT/360).
9. (Desk) A risk report shows “OIS discount DV01” and “projection DV01” separately. What does each mean, and what “bump object” questions must you ask before using the numbers?
10. (Desk) A CSA allows USD or EUR collateral. List the minimum CSA inputs you need before you can choose a discount curve.
11. (Concept) Explain why downgrade triggers create liquidity wrong-way risk.
12. (Concept) Why can “OIS everywhere” be misleading as a slogan?

### Solution Sketches (Selected)
3. $PV=\mathrm{USD}\\,2{,}000{,}000\times 0.94=\mathrm{USD}\\,1{,}880{,}000$.
4. From Example 33.2: $PV\approx \mathrm{USD}\\,398{,}575$ and $DV01\approx \mathrm{USD}\\,74$ per bp (discount curve only; forwards held fixed).
7. $r_{\text{eff}}^{\text{USD}|\text{EUR}}=3.00\\%+0.70\\%-1.00\\%=2.70\\%$. So $PV^{\text{USD}|\text{EUR}}=1{,}000{,}000e^{-0.027\cdot 5}=\mathrm{USD}\\,873{,}715$. USD collateral PV is $1{,}000{,}000e^{-0.03\cdot 5}=\mathrm{USD}\\,860{,}708$.
8. $PAI=\mathrm{USD}\\,50{,}000{,}000\times 0.0480\times \frac{1}{360}=\mathrm{USD}\\,6{,}666.67$.
9. Discount DV01: sensitivity to the OIS *discount* curve holding projected cashflows fixed. Projection DV01: sensitivity to the curve used to project the floating index. Ask: “what was bumped (zero rates, par quotes, or curve nodes)? what was rebuilt? what was held fixed?”

---

## References

- Brigo, Morini, and Pallavicini, *Counterparty Credit Risk, Collateral and Funding*, “2.2 Definition of Exposures” (SCSA motivation) and “9.4.3 Consequences of Perfect Collateralization” (foreign-currency collateral discounting).
- Crépey, *Counterparty Risk and Funding*, “Outline” (clean price vs adjustments) and “2.1 Financial Setup” (two-curve minimum under collateralization).
- Crépey, *Counterparty Risk and Funding*, “11.2 Credit Value Adjustment and Collateralization under Rating Triggers” (rating-linked thresholds and liquidity stress example).
- Hull, *Risk Management and Financial Institutions*, “The OIS Rate” and “Central Clearing” (OIS definition and PAI).
- Hull, *Options, Futures, and Other Derivatives*, “Multiple Yield Curves” (discount vs payoff curves).
- Andersen & Piterbarg, *Interest Rate Modeling*, “6.4.2 Forward Rate Approach” (collateral rate motivation; LIBOR–overnight divergence).
- Neftci, *Principles of Financial Engineering*, example discussion of LIBOR curve for payoffs and OIS curve for discounting.

*Chapter 33 establishes why collateral terms determine discount rates. Chapter 34 develops the XVA framework (CVA, DVA, FVA) for trades with imperfect collateralization.*
