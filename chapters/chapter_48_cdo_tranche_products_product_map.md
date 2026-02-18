# Chapter 48: CDO / Tranche Products — What They Are (Product Map)

---

## Introduction

In the 2007–2008 credit crisis, CDO tranches became a blunt lesson: you can’t price or risk-manage what you don’t understand. Many investors treated senior tranches as “almost risk-free.” The crisis showed that when structure and default clustering (correlation) move against you, small modeling errors can turn into very large losses.

A collateralized debt obligation (CDO) starts from one pool of credit risk and slices it into multiple **tranches**. The tranches can have radically different behavior: a senior tranche can look bond-like most of the time, while the equity (first-loss) tranche can behave more like a leveraged option on “the economy.”

Mechanically, nothing mystical is happening: losses are allocated **sequentially**, not pro‑rata. The first-loss tranche absorbs losses first; only after that cushion is exhausted do higher tranches start to lose. This single “subordination” rule is what creates the wide range of risk/return profiles.

For a middle-office reader moving toward a trading-desk mindset, tranche mechanics matter because:

- **Notional is not risk**: a small tranche notional can carry a large fraction of the portfolio’s spread/correlation risk.
- **P&L is multi-factor**: spreads matter, but so does default clustering (correlation) and recovery assumptions.
- **Conventions become cash**: definitions of loss, premium accrual on default, settlement timing, and index rules drive real cashflows and breaks.
- **Standardization exists**: many tranches reference standardized index portfolios (e.g., CDX / iTraxx), which makes them useful for hedging and for model calibration (Chapters 49–51).

This chapter provides the **product map**—the mechanics of how tranches work, without diving into pricing models. We cover:

- **48.1 CDO Product Taxonomy**: Cash vs synthetic, funded vs unfunded, bespoke vs standard
- **48.2 Portfolio Loss as the State Variable**: The single number that drives everything
- **48.3 Tranche Attachment and Detachment**: How to read tranche "strikes"
- **48.4 The Tranche Loss Function**: The mathematical heart of waterfall mechanics
- **48.5 Premium and Protection Legs**: Who pays what and when
- **48.6 Standard Index Tranches**: Market-traded attachment points
- **48.7 Tranche Leverage**: Why equity can be highly leveraged
- **48.8 Conservation of Expected Loss**: The fundamental arbitrage constraint
- **48.9 Worked Examples**: From mechanics to dollars
- **48.10 CDOs as Correlation Products**: Why tranches need correlation models
- **48.11 The 2008 Crisis: What Went Wrong**: Historical lessons
- **48.12 Leveraged Super Senior**: An advanced structure

Pricing (Chapter 49) and correlation frameworks (Chapter 50) build directly on these foundations.

---

## Learning Objectives
- Translate tranche “strikes” (attachment/detachment) into cash loss allocation mechanics.
- Compute tranche loss and outstanding notional from a portfolio loss level $L(t)$.
- Explain premium vs protection legs and how a quote becomes cashflows.
- Preview PV and a simple “spread PV01” definition with explicit units and sign.
- Build intuition for why tranches are “correlation products” (loss clustering vs diversification).

**Prerequisites:** [Chapter 35 — Default, Recovery, Credit Events](chapters/chapter_35_default_recovery_credit_events.md), [Chapter 38 — CDS Contract Mechanics](chapters/chapter_38_cds_contract_mechanics.md), [Chapter 45 — CDS Indices — Contract Mechanics and Quotes](chapters/chapter_45_cds_indices_contract_mechanics_and_quotes.md), [Chapter 46 — Intrinsic Index Spread and Index Basis](chapters/chapter_46_intrinsic_index_spread_and_index_basis.md), [Chapter 47 — Hedging and Relative Value in CDS Indices](chapters/chapter_47_hedging_relative_value_cds_indices.md)  
**Follow-on:** [Chapter 49 — Tranche Core Concepts — Expected Tranche Loss and Present Value](chapters/chapter_49_tranche_core_concepts_etl_pv.md), [Chapter 50 — Correlation and Tranche Pricing Frameworks](chapters/chapter_50_correlation_tranche_pricing_frameworks.md), [Chapter 51 — Tranche Risk — Tranche PV01, Correlation Risk, and Jump-to-Default Clustering](chapters/chapter_51_tranche_risk.md)

### Quick Conventions (Read Once)
- Attachments/detachments $A,D$ and losses are stated as **fractions of original portfolio notional** $N_{\text{port}}$ unless explicitly stated otherwise.
- Portfolio loss $L(t)$ is the **cumulative** loss fraction (so $0 \le L(t) \le 1$, or $ \le L_{\max}$ if you assume fixed recoveries).
- A tranche is denoted $[A,D]$ with width $w=D-A$. Tranche face notional is $N_{\text{tr}}^{\text{face}} = w\,N_{\text{port}}$.
- “Tranche loss” can be expressed either (i) as a fraction of **portfolio** notional, or (ii) as a fraction of **tranche** notional. State which you mean.
- When we use basis points: $1\text{ bp} = 10^{-4}$.

---

## 48.1 CDO Product Taxonomy: From Cash Flow to Synthetic

### 48.1.1 What Is a CDO?

A useful working definition: a CDO is a security (or set of securities) whose payments depend on defaults and recoveries in an underlying portfolio of credit‑risky assets. The structure “transforms” one portfolio’s credit risk into multiple securities with different credit profiles by changing **who absorbs losses first**.

The core insight is **structural subordination**: rather than sharing losses proportionally, a CDO ranks investors in a "waterfall" so that junior tranches absorb losses first, protecting senior tranches until subordination is exhausted. This single mechanism transforms a portfolio of BBB credits into a menu of securities ranging from AAA to unrated.

> **Why "Equity" for the First-Loss Tranche?**
>
> In tranche language, "equity" means "most junior / first‑loss," by analogy with a corporate capital structure. It is **not** equity in the sense of common stock.

### 48.1.2 Cash Flow CDOs: The Traditional Structure

In a traditional cash flow CDO, investors buy **funded** notes issued by a special purpose vehicle (SPV). The SPV uses the sale proceeds to purchase a collateral portfolio (bonds, loans, etc.) and then distributes cashflows to tranche investors through a deal‑specific waterfall.

The SPV issues multiple tranches—typically labeled **equity** (first-loss), **mezzanine** (middle layers), and **senior** (most protected). Cash flows from the underlying portfolio are distributed according to a detailed **waterfall** specified in the deal documents:

> **Analogy: The Champagne Tower**
>
> Imagine a wedding champagne tower.
>
> *   **Losses = Champagne**: Pouring champagne represents defaults entering the system.
> *   **Equity Tranche (Top Glass)**: The first glass to fill. It absorbs the liquid (losses) immediately. It *protects* the glasses below it.
> *   **Mezzanine (Middle Tier)**: These glasses only get wet if the top glass overflows (Equity is wiped out).
> *   **Senior (Bottom Tier)**: This is the wide base. It stays dry unless the entire bottle is poured out (Catastrophic Systemic Collapse).
> *   **The Twist**: In finance, the *Top Glass* (Equity) gets paid the most to stand there and get wet. The *Bottom Tier* (Senior) pays for the privilege of staying dry.

```
                    ┌─────────────────────────────────┐
                    │    Collateral Portfolio         │
                    │  (Bonds, Loans, CDS positions)  │
                    └────────────────┬────────────────┘
                                     │ Interest + Principal
                                     ▼
                    ┌─────────────────────────────────┐
                    │    Special Purpose Vehicle      │
                    │         (SPV)                   │
                    └────────────────┬────────────────┘
                                     │ Waterfall Payments
              ┌──────────────────────┼──────────────────────┐
              ▼                      ▼                      ▼
    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
    │  Senior Tranche │    │ Mezzanine Tranche│    │  Equity Tranche │
    │   (AAA-rated)   │    │   (BBB-rated)   │    │   (Unrated)     │
    │  Paid first     │    │  Paid second    │    │  Paid last      │
    │  Loses last     │    │  Middle losses  │    │  First loss     │
    └─────────────────┘    └─────────────────┘    └─────────────────┘
```

In practice the waterfall can be complex. Deal documents can include triggers that divert cash away from junior tranches, accelerate senior amortization, or stop reinvestment after collateral deteriorates.

Cash flow CDO waterfalls typically include:
- **Interest waterfall**: Determines how coupon payments flow to tranches
- **Principal waterfall**: Determines how principal repayments flow
- **OC (Overcollateralization) tests**: Triggers that divert cash if collateral deteriorates
- **IC (Interest coverage) tests**: Triggers based on interest coverage ratios
- **Reinvestment period**: Manager can reinvest principal proceeds for specified period

### 48.1.3 Synthetic CDOs: Unfunded Derivative Structures

A **synthetic CDO** is a derivative whose payoff depends on defaults in a reference portfolio. A **single‑tranche synthetic CDO (STCDO)** is the common trading form: you trade one tranche $[A,D]$ versus a dealer; the rest of the capital structure is not issued as funded securities.

Key characteristics (vs cash CDOs):

1. **Bilateral derivative, not an SPV**: counterparty and collateral terms matter.
2. **Virtual reference portfolio**: a list of names with weights (often an index or index‑like basket), rather than a funded pool of bonds/loans.
3. **“Unfunded” in a specific sense**: there is no upfront principal purchase of collateral like a cash CDO, but tranche trades can involve upfront payments and typically require margin/collateral as MTM moves.
4. **Mechanically simple payoff**: premiums are paid on surviving tranche notional; protection payments are driven by changes in tranche loss.
5. **Often faster/customizable**: attachment/detachment, maturity, and portfolio can be tailored (subject to what the dealer can hedge).

**Desk reality (dealer side):** the dealer ends up with portfolio spread risk, default correlation risk, and recovery risk. Hedging often uses combinations of index/single‑name CDS and other tranches, with rebalancing as the market and the tranche state evolve.

### 48.1.4 Full Capital Structure vs Single-Tranche

In a **full capital structure** deal, all tranches from equity up to super‑senior are issued and sold to investors; the issuer/arranger aims to lay off the whole structure.

In **single‑tranche** trading, only one slice is traded with the end‑investor; the dealer hedges (or warehouse‑manages) the remaining exposures. The reference portfolio is “real” for payoff purposes even if the dealer doesn’t literally hold a name‑for‑name CDS portfolio at all times.

### 48.1.5 Cash CDO vs Synthetic CDO: Detailed Comparison

| Feature | Cash Flow CDO | Synthetic CDO (STCDO) |
|---------|---------------|----------------------|
| **Funding** | Funded—investor pays upfront | Usually unfunded (no principal funding); margin/collateral applies; upfront may apply depending on tranche |
| **Underlying** | Actual bonds/loans in SPV | Virtual reference portfolio (typically CDS-index-like) |
| **Counterparty risk** | SPV bankruptcy remoteness | Bilateral with dealer |
| **Waterfall complexity** | Complex, deal-specific triggers (OC/IC tests, reinvestment) | Simple payoff function |
| **Manager role** | Active management, reinvestment decisions | None (static reference portfolio) |
| **Customization** | Limited (compromise across investors) | Full (single investor) |
| **Execution speed** | Weeks–months | Days–weeks |
| **Liquidity** | Generally illiquid | Index tranches can be more liquid than bespoke; varies by market/vintage |
| **Default settlement** | Sell distressed bonds from portfolio | Cash settle via CDS auction process |
| **Modeling complexity** | High (manager behavior, reinvestment) | Simpler cashflow mechanics, but still needs defaults, recoveries, correlation, and mapping conventions |

For this book, we focus on **synthetic tranches** because their payoff mechanics are much cleaner: they can be written directly as functions of portfolio loss and tranche attachment/detachment.

---

## 48.2 Portfolio Loss $L(t)$: The State Variable

### 48.2.1 Definition

The portfolio loss $L(t)$ is the single state variable that drives tranche payoffs: once you know how much of the portfolio has been lost, the waterfall tells you exactly how much each tranche has lost.

A general (weighted) definition is:

$$L(T) = \sum_{i=1}^{N_c} w_i\,(1 - R_i)\,\mathbf{1}_{\tau_i \le T}, \qquad \sum_{i=1}^{N_c} w_i = 1$$

where:
- $N_c$ is the number of credits in the portfolio
- $w_i$ is the portfolio weight (fraction of notional) of credit $i$
- $R_i$ is the recovery rate for credit $i$
- $\tau_i$ is the default time of credit $i$
- $\mathbf{1}_{\tau_i \le T}$ equals 1 if credit $i$ has defaulted by time $T$, 0 otherwise

If exposures are equal ($w_i = 1/N_c$), this reduces to:

$$L(T) = \frac{1}{N_c} \sum_{i=1}^{N_c} (1 - R_i) \mathbf{1}_{\tau_i \le T}$$

Each default contributes $(1-R_i)/N_c$ to the portfolio loss.

### 48.2.2 Maximum Portfolio Loss and Recovery

If **every** name defaults by maturity, the maximum possible realized loss is:

$$\boxed{L_{\max} = \sum_{i=1}^{N_c} w_i\,(1 - R_i)}$$

If you assume a common recovery $R$ for intuition, this becomes $L_{\max} = 1 - R$. For example, with $R = 40\%$, the portfolio cannot lose more than $60\%$ of notional (in the “all default” extreme).

This matters for very senior tranches: if the detachment point $D$ is above $L_{\max}$, some of the quoted width is **unreachable** by credit losses. (We handle this explicitly in Section 48.5.4.)

### 48.2.3 Example: Loss from Defaults

Consider a portfolio of 100 credits, each with USD 10 million face value (total USD 1 billion). If 3 credits default with recovery rates of 40%, 35%, and 50%:

| Default | Face Value | Recovery | Loss per Name | Contribution to $L$ |
|---------|------------|----------|---------------|---------------------|
| 1 | USD 10mm | 40% | USD 6mm | 0.60% |
| 2 | USD 10mm | 35% | USD 6.5mm | 0.65% |
| 3 | USD 10mm | 50% | USD 5mm | 0.50% |
| **Total** | | | **USD 17.5mm** | **1.75%** |

The portfolio loss is $L = 1.75\%$ of notional, or USD 17.5 million.

### 48.2.4 Why $L(t)$ Is All That Matters

For **mechanics**, you don’t need to know *which* names defaulted — only the cumulative loss $L(t)$. Premium and protection payments can be written as functions of $L(t)$ and the tranche strikes $(A,D)$.

Desk note: in real risk management you still care about *which names* defaulted for hedging, basis, and P&L attribution. But the contractual waterfall itself is driven by portfolio loss.

---

## 48.3 Attachment and Detachment: Reading Tranche "Strikes"

### 48.3.1 Formal Definitions

The tranche is defined by two “strikes” on portfolio loss:

- **Attachment $A$ (lower strike / subordination):** tranche loss is zero while $L \le A$. As soon as portfolio loss exceeds $A$, this tranche starts absorbing losses.
- **Detachment $D$ (upper strike):** once $L \ge D$, the tranche is fully exhausted (100% loss of tranche face).

In our notation, $A = K_1$ (attachment) and $D = K_2$ (detachment), with tranche width $w = D - A$.

**Check (strikes → dollars):** if the portfolio notional is $N_{\text{port}}=USD 100\text{mm}$ and the tranche is $[A,D]=[3\%,7\%]$, then $w=4\%$ and the tranche face notional is $N_{\text{tr}}^{\text{face}} = w\,N_{\text{port}} = USD 4\text{mm}$. This “percent-of-portfolio” convention is a common source of booking mistakes when someone accidentally treats tranche notional as percent-of-tranche.

### 48.3.2 Intuition: Deductible and Limit

Think of attachment as an **insurance deductible**: the amount of portfolio loss that must occur before the tranche starts paying. Think of detachment as the **coverage limit**: once losses reach this level, the tranche is exhausted.

| If $L$ is... | Then tranche $[A, D]$... |
|--------------|-------------------------|
| Below $A$ | Has zero loss (protected by subordination) |
| Between $A$ and $D$ | Absorbs losses dollar-for-dollar |
| At or above $D$ | Is fully wiped out |

### 48.3.3 Tranche Labels by Position

Standard market terminology names tranches by their position in the capital structure:

| Label | Position | Risk/return intuition | Typical market participants (illustrative) |
|-------|----------|------------------------|-------------------------------------------|
| **Equity** | $[0, A_1]$ | First‑loss; most levered to portfolio loss and correlation | Hedge funds, prop/correlation desks |
| **Junior mezzanine** | $[A_1, A_2]$ | Takes losses after equity is gone; still quite “event‑driven” | Credit funds, hedge funds |
| **Senior mezzanine** | $[A_2, A_3]$ | More protected; sensitive to tail scenarios and correlation | Asset managers, hedge funds |
| **Senior** | $[A_3, A_4]$ | High subordination; tends to look “bond‑like” in benign markets | Insurance / ALM style investors, banks |
| **Super senior** | $[A_4, D]$ | Very remote credit losses, but can have large MTM under spread/correlation moves | Banks, structured credit investors |

Desk note: the exact “who owns what” changes by era and regulation. The important point is that **different tranches attract different risk budgets**.

---

## 48.4 The Tranche Loss Function: Mathematical Heart of the Waterfall

### 48.4.1 The Piecewise Linear Payoff

A standard contractual tranche loss function (expressed as a fraction of **tranche** notional) is:

$$\boxed{L(T, K_1, K_2) = \frac{\max(L(T) - K_1, 0) - \max(L(T) - K_2, 0)}{K_2 - K_1}}$$

This gives the fractional loss of the tranche—i.e., as a percentage of tranche notional, ranging from 0 to 1 (or 0% to 100%).

### 48.4.2 Portfolio-Scale Tranche Loss

In our notation, we also work with tranche loss expressed as a fraction of **portfolio** notional:

$$\boxed{TL(L) = \min\bigl(\max(L - A, 0), D - A\bigr)}$$

The relationship is:
$$L_{\text{tr}}(T; A, D) = \frac{TL(L(T))}{D - A}$$

**Check (toy numbers):** for a $[3\%,7\%]$ tranche and portfolio loss $L=5\%$,
$$
TL(L)=\min(\max(5-3,0),4)=2\%\ \text{of portfolio notional},
$$
so the tranche has lost $2/4=50\%$ of its face. If $N_{\text{port}}=USD 100\text{mm}$, that is a $USD 2\text{mm}$ loss on a $USD 4\text{mm}$ tranche.

### 48.4.3 Piecewise Derivation

The tranche loss function has three regimes:

**Regime 1: $L \le A$ (Below Attachment)**

Portfolio losses have not reached the tranche's deductible. Subordination fully protects the tranche:
$$TL(L) = 0$$

**Regime 2: $A \lt L \lt D$ (Within Tranche)**

The tranche absorbs losses dollar-for-dollar:
$$TL(L) = L - A$$

Every additional percentage point of portfolio loss flows directly to this tranche.

**Regime 3: $L \ge D$ (At or Above Detachment)**

The tranche has absorbed its full width and is wiped out:
$$TL(L) = D - A = w$$

No further losses affect this tranche—they "pass through" to the next tranche up.

### 48.4.4 Visual Representation

```
Tranche Loss TL(L) as fraction of portfolio notional

    TL
     ↑
 D-A │             ____________
     │            /
     │           /
     │          /
   0 │_________/
     └─────────┬────────┬────────→ L (Portfolio Loss)
              A        D

     ← Zero → ← Linear → ← Flat →
       loss     growth     (wiped)
```

This piecewise linear shape is worth memorizing; most tranche cashflow identities are simple consequences of it.

### 48.4.5 The Call Spread / Put Spread Analogy

The tranche loss function can be written in several mathematically equivalent forms that reveal its option-like nature:

**Call-spread form** (most common in practice):
$$\boxed{TL(L) = (L - A)^+ - (L - D)^+}$$

This is algebraically identical to $\min(\max(L-A, 0), D-A)$ but reveals the tranche as a **call spread** on portfolio loss—long a call struck at attachment, short a call struck at detachment.

> **Desk Reality: The Bull Call Spread**
>
> The Tranche Loss function is economically identical to a **bull call spread** on portfolio loss:
> - **Long call @ A:** You start losing when L exceeds attachment A
> - **Short call @ D:** Your loss is capped when L exceeds detachment D
>
> For the **Protection Seller** (receiving premium, paying losses):
> - This is a **short call spread** — you collect premium and hope losses stay low
> - Maximum loss = tranche width W = D - A
>
> This is why correlation traders speak the language of options. The mathematics of tranches is the mathematics of spreads.

**Put-spread form** (for surviving notional):

An equivalent view is that the **remaining tranche notional** is a put spread on portfolio loss:

$$f_\kappa(l) = (K_\kappa - l)^+ - (K_{\kappa-1} - l)^+$$

This is a **put-spread** representation: long a put at the upper strike and short a put at the lower strike. It’s a useful mental model when thinking about tranche “Greeks” (delta/gamma) with respect to portfolio loss.

**Alternative form** (using min functions):
$$TL(L) = \min(L, D) - \min(L, A)$$

This form reveals that any tranche loss equals the difference between two "equity" (0-K) tranche losses—a property exploited in the base correlation framework (Chapter 50).

---

## 48.5 Premium and Protection Legs

### 48.5.1 Premium Leg: Payments on Surviving Notional

The premium leg is the tranche analogue of a CDS premium leg: periodic payments from the protection buyer to the protection seller, calculated on **surviving (outstanding) tranche notional**. A common market convention is quarterly payments with an Act/360-style day count, but the contract governs.

The key mechanic to internalize is simple: **as the tranche takes losses, the premium base amortizes** and running premiums get smaller.

In dollar terms, for a tranche with face notional $N_{\text{tr}}^{\text{face}}$:

$$\boxed{PremPay_i = \alpha_i \cdot S(A, D) \cdot N_{\text{tr}}^{\text{out}}(t_i)}$$

where:
- $\alpha_i$ is the accrual year fraction
- $S(A, D)$ is the contractual spread (decimal per annum)
- $N_{\text{tr}}^{\text{out}}(t_i) = (w - TL(L(t_i))) \cdot N_{\text{port}}$ is outstanding notional

**Check (premium-base amortization):** in the toy above ($[3,7]$ tranche, $L=5\%$), outstanding tranche notional is $(w-TL)N_{\text{port}}=(4\%-2\%)N_{\text{port}}=2\%\,N_{\text{port}}$, i.e., **half** the original tranche face. Premium payments on that tranche are therefore about half their initial size (before discounting and accrual details).

### 48.5.2 Protection Leg: Loss Payments

The protection leg compensates the tranche protection buyer for realized losses. Each time portfolio loss increases, the tranche loss $TL(L(t))$ may increase; the protection seller pays the **incremental** tranche loss.

In portfolio‑fraction terms:

$$\Delta TL = TL(L(t^+)) - TL(L(t^-))$$

In dollars (on portfolio notional $N_{\text{port}}$):

$$\boxed{\text{ProtPay} = \Delta TL\cdot N_{\text{port}}}$$

A default only generates a protection payment for tranche $[A,D]$ if the post‑default portfolio loss lies in the tranche’s “loss window” (i.e., if the tranche is not fully below attachment or already exhausted).

### 48.5.3 Loss Settlement Timing

Premiums accrue through time, while defaults happen at specific dates. As with single‑name CDS, tranche conventions typically include:

- **Accrued premium on default:** if a default happens mid‑coupon period, premium is accrued up to the effective default/settlement date on the notional that was outstanding before the loss.
- **Discrete premium dates:** running premium is paid on scheduled dates on the surviving notional at that time.

Risk systems often approximate this (for intuition) by paying premium on end‑of‑period outstanding notional, but a production implementation needs the contract’s exact accrual/settlement rules.

### 48.5.4 Senior Tranche Adjustment for Maximum Loss

For very senior tranches, it’s common to quote detachment points that extend beyond the maximum possible realized portfolio loss implied by recovery assumptions. When $D$ exceeds $L_{\max}$, part of the tranche width is **unreachable** by credit losses.

Consider a portfolio of $N_c$ credits with assumed recovery rate $R$. The maximum possible portfolio loss is:
$$L_{\max} = (1 - R)$$

For a tranche with detachment $D \gt L_{\max}$, the tranche can never be fully wiped out by defaults alone. The effective tranche width for loss purposes is:
$$w_{\text{eff}} = \min(D, L_{\max}) - A$$

**Example:** If $R = 40\%$ (so $L_{\max} = 60\%$) and the super-senior tranche is $[30\%, 100\%]$:
- Nominal width: 70%
- Effective width: $60\% - 30\% = 30\%$
- The 40% above $L_{\max}$ is "unreachable"—this affects both pricing and risk calculations

This adjustment matters primarily for super-senior tranches with very high detachment points.

### 48.5.5 Quote → PV → Risk (Preview)

So far we have “mechanics”: how a portfolio loss process $L(t)$ maps into tranche loss and outstanding notional. To get from a **quote** to a **PV** you add discounting and expectations.

Fix a direction:
- **Buy protection on $[A,D]$:** you *pay* the premium leg and *receive* loss payments.
- **Sell protection on $[A,D]$:** you *receive* the premium leg and *pay* loss payments.

A simple (common) PV decomposition for the protection buyer is:
$$PV \approx PV(\text{protection leg}) - PV(\text{premium leg}) - \text{upfront (if any)}$$

At a desk level, you can think of:
- **Premium leg PV** (running spread $S$ paid on expected outstanding notional):
  $$PV_{\text{prem}} \approx S \sum_i \alpha_i\,P(0,t_i)\,\mathbb{E}[N_{\text{tr}}^{\text{out}}(t_i)]$$
- **Protection leg PV** (expected discounted incremental tranche losses):
  $$PV_{\text{prot}} \approx \sum_j P(0,u_j)\,\mathbb{E}[\Delta TL_j]\,N_{\text{port}}$$

The **par running spread** $S^*$ is the value that makes the trade have zero PV (ignoring any upfront), i.e.
$$\boxed{S^* \approx \frac{PV_{\text{prot}}}{\sum_i \alpha_i\,P(0,t_i)\,\mathbb{E}[N_{\text{tr}}^{\text{out}}(t_i)]}}$$

**Risk (be explicit about “what is bumped”)**
- **Spread PV01 (contract-spread bump):** bump the contractual running spread $S$ by $+1$ bp $=10^{-4}$, holding the expected outstanding-notional profile fixed.
  - **Bump object:** $S$ (the tranche running spread)
  - **Bump size:** $1\text{ bp}=10^{-4}$
  - **Units:** currency per 1 bp per stated tranche face
  - **Sign:** positive for a premium receiver (protection seller), negative for a premium payer (protection buyer)
  - Approximation: $PV01_S \approx 10^{-4}\times \sum_i \alpha_i P(0,t_i)\mathbb{E}[N_{\text{tr}}^{\text{out}}(t_i)]$.

> **Pitfall — “PV01” without a bump definition:** In tranche/risk reports, “PV01/DV01” might mean *contract spread* PV01, *index spread* PV01, or even *rates* DV01.
> **Why it matters:** You can’t compare risk numbers or size hedges if the bumped object differs.
> **Quick check:** Ask: “What was bumped (quotes vs curves vs model parameter), by how much, and was anything rebuilt/recalibrated after the bump?”

Correlation sensitivities and jump-to-default clustering risk are covered in Chapter 51; they come from how the **loss distribution** (not just its mean) shifts when spreads/correlation/recovery assumptions move.

---

## 48.6 Standard Index Tranches

### 48.6.1 Market-Traded Attachment Points

An interdealer market developed for **standardized** synthetic CDO tranches referencing CDS index portfolios. Standardization matters because it concentrates liquidity: many market participants trade the same attachment/detachment points on the same underlying index.

One commonly used grid (varies by index and vintage) is:

| Tranche | CDX IG NA | iTraxx Europe | CDX NA HY |
|---------|-----------|---------------|-----------|
| | $K_1$ — $K_2$ | $K_1$ — $K_2$ | $K_1$ — $K_2$ |
| **Equity** | 0% — 3% | 0% — 3% | 0% — 10% |
| **Junior Mezzanine** | 3% — 7% | 3% — 6% | 10% — 15% |
| **Senior Mezzanine** | 7% — 10% | 6% — 9% | 15% — 25% |
| **Senior** | 10% — 15% | 9% — 12% | 25% — 35% |
| **Super Senior** | 15% — 30% | 12% — 22% | 35% — 60% |

Some references also quote a residual “$K$–100%” tranche. If you treat recoveries as fixed so that $L_{\max} \lt 100\%$, detachment points above $L_{\max}$ create “unreachable width” and need the adjustment in Section 48.5.4.

### 48.6.2 Why These Specific Attachment Points?

The attachment points are chosen to reflect the expected loss characteristics of each index:

**CDX IG (0–3% equity):** For a 125-name equal‑weight index and a rough recovery assumption of $R \approx 40\%$, each default contributes about
$$\frac{1-R}{125} \approx \frac{0.60}{125} \approx 0.48\%$$
of portfolio loss. So the 0–3% tranche corresponds to roughly the “first ~6 defaults” under that simplified arithmetic. (Real life is messier: weights vary, recoveries vary, and defaults are not independent.)

**CDX HY (0-10% equity):** High-yield portfolios have much higher expected loss—defaults are more frequent and often clustered. The 0-10% equity tranche reflects this higher default probability, ensuring the equity absorbs a meaningful portion of expected loss before mezzanine tranches are impacted.

**iTraxx Europe:** Similar logic to CDX IG, though slight differences in attachment points (3-6% vs 3-7% for junior mezz) reflect different market conventions and portfolio characteristics.

### 48.6.3 Why Standard Tranches Matter

Standard tranches have the **same mechanics** as any single‑tranche synthetic CDO; the difference is **liquidity** and **observability**. Because standard tranches are quoted more regularly, their prices become inputs for:

Standard tranches serve as:
1. **Hedging instruments** for bespoke tranche books
2. **Calibration inputs** for correlation models (Chapter 50)
3. **Relative value benchmarks** for credit portfolio views

### 48.6.4 Bespoke vs Standard Tranches

**Standard tranches** are liquid, exchange-like products with:
- Fixed attachment points matching the tables above
- Reference portfolios = the underlying CDS indices
- Transparent quotes from dealer market
- Tight bid-offer spreads

**Bespoke tranches** are customized structures where the investor selects:
- Reference portfolio composition
- Attachment and detachment points
- Maturity
- Other structural features

In practice, bespoke tranches are often valued by mapping them to a correlation surface implied by liquid standard tranches (see Chapter 50 for frameworks and mapping methods).

### 48.6.5 Liquidity Hierarchy

Liquidity varies by index, vintage, and market regime. A common (illustrative) hierarchy is:

| Tranche | Relative Liquidity | Typical Market Participants |
|---------|-------------------|---------------------------|
| Equity | High | Hedge funds, prop desks |
| Junior Mezz | Moderate | Credit funds, hedge funds |
| Senior Mezz | Moderate | Insurance, asset managers |
| Senior | Lower | Insurance companies |
| Super Senior | Variable | Banks, monolines (pre-crisis) |

Equity is frequently among the more actively traded slices because it concentrates correlation exposure, but this can vary widely.

> **Deep Dive: The Equity Tranche ("Toxic Waste" or "Gold Mine"?)**
>
> - **The risk:** you are first‑loss. Any realized portfolio loss starts hitting you immediately (up to your detachment).
> - **The reward:** you receive the richest compensation in the capital structure (often a mix of upfront + running), because you’re providing the first layer of protection to everyone above you.
> - **The correlation view:** equity often “likes” higher correlation in the specific sense that a more bimodal loss distribution increases the probability of a near‑zero loss outcome — even though it also increases tail risk.

---

## 48.7 Tranche Leverage: Why Equity Can Be Highly Leveraged

### 48.7.1 Leverage Ratio Definition

$$\boxed{\text{Leverage} = \frac{\Delta_s}{F(K_1, K_2)}}$$

where $\Delta_s$ is the systemic delta (spread sensitivity of the tranche in terms of equivalent index notional) and $F(K_1, K_2)$ is the face value of the tranche.

More intuitively:

$$\boxed{\text{Leverage} = \frac{\text{Systemic spread PV01 of tranche}}{\text{Systemic spread PV01 of equivalent index notional}}}$$

### 48.7.2 Worked Example (DV01 Ratio)

Suppose a risk report shows the following **hypothetical** “systemic spread PV01” numbers (sometimes labeled “DV01” even though the bumped object is a credit spread, not an interest rate):

- 0–3% tranche systemic spread PV01: USD 550,000 per USD 10mm tranche face
- Index systemic spread PV01: USD 30,000 per USD 10mm index notional

Then leverage is:

$$\text{Leverage} \approx \frac{550{,}000}{30{,}000} \approx 18.3\times$$

Interpretation: **USD 10mm of equity tranche can behave (for small spread moves) like ~USD 180mm of index risk.**

### 48.7.3 Why Leverage Happens

Why is the equity tranche so leveraged? Consider:

- The equity tranche (0-3%) represents 3% of portfolio notional
- But it absorbs the *first* 3% of losses—which have the highest probability
- Small changes in expected portfolio loss (driven by spreads, correlation, and recovery) can translate into large percentage changes of the tranche’s *own* face notional

> **Desk Reality: What 18x Leverage Means**
>
> If you buy USD 10 million notional of a tranche whose leverage is ~18x, your spread sensitivity is roughly equivalent to holding ~USD 180 million notional of the index. A small index spread move can therefore create a large P&L swing on the tranche.
>
> This is why equity tranches are the domain of hedge funds and correlation desks, not traditional credit investors. The leverage amplifies both gains and losses dramatically.

### 48.7.4 Leverage Changes with Spread Level

Leverage is not constant. As spreads widen (or as losses accumulate), the “action” migrates up the capital structure:

When spreads widen significantly:
- Equity tranche becomes "more equity-like" (higher probability of wipeout)
- Mezzanine tranches become "equity-like" (loss distribution shifts through them)
- Senior tranches take on more risk

This dynamic leverage is one reason tranche risk management is complex.

---

## 48.8 Conservation of Expected Loss: The Arbitrage Constraint

### 48.8.1 The Fundamental Property

If you partition the capital structure into **contiguous** tranches that span the whole loss range, the total portfolio loss is just the sum of tranche losses. Taking expectations gives the **conservation of expected loss**:

$$\boxed{\sum_{m=1}^{M} \mathbb{E}[TL_m(L(T))] = \mathbb{E}[L(T)]}$$

where $TL_m(\cdot)$ is the *portfolio‑fraction* loss allocated to tranche $m$.

### 48.8.2 The Mathematical Derivation

Starting from the definition of tranche expected loss:

$$\text{Total EL} = \mathbb{E}\left[\sum_{m=1}^{M}(K_m - K_{m-1}) L(T, K_{m-1}, K_m)\right]$$

Using the alternative form of the tranche loss function:

$$= \mathbb{E}\left[\sum_{m=1}^{M}\left(\min[L(T), K_m] - \min[L(T), K_{m-1}]\right)\right]$$

This is a telescoping sum. Expanding:

$$= \mathbb{E}[\min(L(T), K_M) - \min(L(T), K_0)]$$

Since $K_0 = 0$ and $K_M = 1$ (or $L_{\max}$ if capped at maximum loss):

$$\boxed{\mathbb{E}\left[\sum_{m=1}^{M}(K_m - K_{m-1}) L(T, K_{m-1}, K_m)\right] = \mathbb{E}[L(T)]}$$

### 48.8.3 Why This Matters

Economically: if you buy protection on every tranche (covering the whole capital structure), you have bought protection on the **entire portfolio loss**, which is the same default‑loss exposure as buying protection on every underlying name (ignoring timing/premium differences).

> **Why This Matters for Pricing**
>
> If a pricing model violates conservation of expected loss, you could:
> 1. Buy protection on ALL tranches (covering 0-100%)
> 2. Sell protection on the index
> 3. Earn riskless profit if model over/under-prices total expected loss
>
> Models or quoting conventions that violate conservation of expected loss create internal inconsistencies. One reason “base correlation”-style constructions became popular is that they preserve this add‑up constraint by construction (Chapter 50).

### 48.8.4 Premium Legs Don't Perfectly Offset

Important caveat: even if the **protection legs** add up cleanly, the **premium legs** do not match perfectly in timing and amortization.

Mechanics intuition: when a default pushes losses into (say) the equity tranche, the tranche’s **outstanding notional** can drop sharply, which immediately shrinks future premium payments on that tranche. A “whole index CDS” premium stream does not amortize in the same way. So “add-up” is cleanest for protection legs; premium legs can differ because the premium base and accrual conventions differ.

---

## 48.9 Waterfall Mechanics: Loss Allocation Across Tranches

### 48.9.1 The Capital Structure Tower

Visualizing the capital structure as a tower helps understand loss flow:

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CAPITAL STRUCTURE                           │
│                                                                     │
│  SUPER SENIOR (15-30%)  │  Last to absorb losses. "Safe"...        │
│  ─────────────────────────────────────────────────────────────────  │
│  Width = 15%            │  unless catastrophe                       │
│  ═══════════════════════════════════════════════════════════════   │
│  SENIOR (10-15%)        │  Protected by substantial subordination   │
│  ─────────────────────────────────────────────────────────────────  │
│  Width = 5%             │                                           │
│  ═══════════════════════════════════════════════════════════════   │
│  MEZZANINE (7-10%)      │  Middle of capital structure              │
│  ─────────────────────────────────────────────────────────────────  │
│  Width = 3%             │                                           │
│  ═══════════════════════════════════════════════════════════════   │
│  JUNIOR MEZZ (3-7%)     │  Takes losses after equity exhausted      │
│  ─────────────────────────────────────────────────────────────────  │
│  Width = 4%             │                                           │
│  ═══════════════════════════════════════════════════════════════   │
│  EQUITY (0-3%)          │  FIRST LOSS. Takes the first losses       │
│  ─────────────────────────────────────────────────────────────────  │
│  Width = 3%             │  Often highly leveraged                   │
└─────────────────────────────────────────────────────────────────────┘

LOSSES FLOW UPWARD: Portfolio losses fill from bottom (Equity) up.
```

### 48.9.2 Loss Flow Through the Capital Structure

When portfolio losses occur, they flow through the capital structure in order:

```
Portfolio Loss $L(t)$
        │
        ▼
┌───────────────────────────────────────────────────────────┐
│                    Capital Structure                       │
│                                                           │
│   ┌─────────────────┐                                     │
│   │ [0%, 3%] Equity │ ◄── Losses hit here FIRST           │
│   │   Width = 3%    │     Until USD 3mm absorbed             │
│   └────────┬────────┘                                     │
│            │ Overflow when equity wiped                   │
│            ▼                                              │
│   ┌─────────────────┐                                     │
│   │ [3%, 7%] Jr Mezz│ ◄── SECOND layer                    │
│   │   Width = 4%    │     Until USD 4mm more absorbed        │
│   └────────┬────────┘                                     │
│            │ Overflow when mezz wiped                     │
│            ▼                                              │
│   ┌─────────────────┐                                     │
│   │ [7%, 10%] Mezz  │ ◄── THIRD layer                     │
│   │   Width = 3%    │                                     │
│   └────────┬────────┘                                     │
│            │                                              │
│            ▼                                              │
│   ┌─────────────────┐                                     │
│   │ [10%, 15%] Sr   │ ◄── FOURTH layer                    │
│   │   Width = 5%    │                                     │
│   └────────┬────────┘                                     │
│            │                                              │
│            ▼                                              │
│   ┌─────────────────┐                                     │
│   │ [15%, 30%] SSr  │ ◄── FIFTH layer                     │
│   │   Width = 15%   │                                     │
│   └─────────────────┘                                     │
└───────────────────────────────────────────────────────────┘

On a USD 100mm portfolio: Equity absorbs first USD 3mm, Jr Mezz next USD 4mm, etc.
```

---

## 48.10 CDOs as Correlation Products

### 48.10.1 Why Correlation Matters

Tranches are often called **correlation products** because their value depends on the tendency of defaults to **cluster** across names.

The right object is the **portfolio loss distribution**: the probability distribution of $L(T)$ at a horizon $T$. If you hold marginal default probabilities and recoveries fixed, correlation does not change the *expected* portfolio loss $\mathbb{E}[L(T)]$, but it can dramatically change the **shape** of the distribution — and tranches are nonlinear functions of $L(T)$.

### 48.10.2 Low Correlation vs High Correlation

Think in terms of how “spread out” the distribution of $L(T)$ is:

- **Low correlation (more idiosyncratic defaults):** by the law of large numbers, $L(T)$ tends to concentrate near its mean; extreme loss scenarios are relatively rare.
- **High correlation (more clustered defaults):** $L(T)$ becomes more **bimodal**: there is more probability of very low losses *and* more probability of very high losses.

### 48.10.3 Correlation Preferences by Tranche

This leads to a desk rule of thumb (with important caveats): **senior tranches tend to dislike correlation**, while **equity tranches can benefit from higher correlation**.

| Tranche | Correlation Preference | Reason |
|---------|------------------------|--------|
| **Equity (0-3%)** | **Wants HIGH correlation** | High correlation → more "all survive" or "all default" outcomes; fewer middle outcomes that wipe out just the equity |
| **Mezzanine** | Mixed / Non-monotonic | Effect depends on spread level and exact attachment points |
| **Super Senior** | **Wants LOW correlation** | Low correlation → losses stay small and distributed; high correlation → tail risk of catastrophic cluster defaults |

### 48.10.4 Extreme Correlation Limits

The intuition becomes clearest at the extremes:

**ρ = 0 (Independence):**
- Defaults are independent events
- By the law of large numbers, actual portfolio loss clusters tightly around expected loss
- Senior tranches are extremely safe (negligible probability of high losses)
- Equity tranche is exposed to “everyday” default risk: if expected loss is meaningfully positive, equity is the first place losses show up

**ρ = 100% (Perfect Correlation):**
- Credits default together or survive together
- Portfolio loss is (approximately) binary: either 0% (no defaults) or $L_{\max}$ (all default)
- Tranche loss is also a two‑point distribution, but the *severity* depends on $[A,D]$:
  - If $D \le L_{\max}$, the tranche is either untouched or fully wiped
  - If $A \ge L_{\max}$, the tranche is always untouched
  - If $A \lt L_{\max} \lt D$, the tranche takes a **partial** loss in the “all default” scenario

**Check (why equity can “like” higher correlation at the same mean loss):** take $L_{\max}=60\%$ and an equity tranche $[0,3\%]$. Suppose $\mathbb{E}[L]=2\%$.
- In a low-correlation picture where $L$ is tightly concentrated near its mean, equity is often **partially** hit, with tranche loss close to 2% of portfolio notional.
- In a perfect-correlation two-state picture, take $L=0$ with probability $1-p$ and $L=60\%$ with probability $p$. Matching $\mathbb{E}[L]=2\%$ gives $p=2/60\approx 3.33\%$. Equity loss is then $0$ (no-default state) or $3\%$ (all-default state), so $\mathbb{E}[TL_{0-3}]\approx p\times 3\% \approx 0.10\%$ of portfolio notional—much smaller than 2%.
This is purely a distribution-shape effect: higher correlation pushes probability mass toward “no loss” and “catastrophe,” and equity only cares about losses up to 3%.

This is the essence of why tranches are "correlation products": the same expected loss can produce vastly different outcomes depending on whether losses are dispersed or clustered.

---

## 48.11 The 2008 Crisis: What Went Wrong

### 48.11.1 The ABS CDO Structure

A crisis-era structure that matters for intuition is the **ABS CDO**: a CDO whose collateral is made up of tranches of other asset‑backed securities (ABS), rather than a direct pool of loans.

The important “product‑map” idea is **resecuritization**: you tranche the risk **twice**.

**Layer 1: Subprime Mortgages → ABS**
- Pools of subprime mortgages securitized into ABS
- ABS tranched into senior (AAA), mezzanine (BBB), equity

**Layer 2: ABS Tranches → ABS CDO**
- Mezzanine tranches of many ABSs pooled together
- This pool tranched again into senior (AAA), mezzanine, equity

This “mezzanine‑on‑mezzanine” construction can make an apparently senior tranche much more sensitive to underlying mortgage losses than a senior tranche of a plain ABS.

A simplified cliff‑effect illustration:

- A senior **ABS** tranche can still be protected even if losses on the underlying mortgage portfolios are substantial (because junior ABS tranches absorb losses first).
- A senior **ABS CDO** tranche built from **mezzanine ABS tranches** may only receive its full promised return if underlying mortgage‑portfolio losses stay below a much lower threshold (because it is “senior” only with respect to already-thin mezzanine collateral).

One stylized loss table for this structure is:

| Losses on underlying mortgage portfolios | Loss on senior ABS CDO tranche (illustrative) |
|---|---|
| 10% | 0% |
| 15% | ~33% |
| 20% | ~67% |
| 25% | 100% |

### 48.11.2 What Went Wrong

From a tranche desk perspective, the “what went wrong” story has several layers:

1. **Structure amplified modest collateral losses.** In a resecuritization (ABS CDO), senior risk can sit much closer to the underlying loss process than the rating label suggests. A “senior” ABS CDO tranche can be unharmed up to a threshold, but then lose value rapidly once that threshold is breached.

2. **Correlation / tail dependence was under‑appreciated.** If defaults (or loss severity) become more correlated in stress, probability mass moves into tail scenarios — precisely where senior tranches start to take losses.

3. **Incentives and underwriting quality mattered.** “Originate‑to‑distribute” can weaken incentives to maintain underwriting standards; when collateral quality declines, model‑based loss assumptions can become stale quickly.

4. **Liquidity and collateral calls can dominate realized defaults (at least initially).** Even if realized losses are small early on, spread and correlation repricing can cause large MTM swings, which can trigger margin calls and forced de‑risking.

5. **Model governance risk:** a “single calibrated model” is not a risk framework. Tranche desks stress the loss distribution, correlation regime shifts, recovery shocks, and mapping assumptions — not just expected loss.

### 48.11.3 The CDO-Squared Trap

A **CDO‑squared** pushes resecuritization further: the collateral is itself a portfolio of CDO tranches. This adds another layer of subordination and makes the structure extremely sensitive to whether losses are **dispersed** or **clustered** across sub‑portfolios.

The danger is extreme sensitivity to loss clustering:

Illustrative example: consider five sub‑portfolios, each with a tranche that has 4% subordination and 5% width. Assume equal weights and a fixed recovery so each default contributes the same portfolio‑loss increment. Now compare two ways 20 defaults can arrive:

**Scenario 1 (dispersed losses):** 4 defaults in each of 5 sub-portfolios
- Each sub-portfolio: 2.4% loss (below 4% subordination)
- Sub-tranche loss: 0%
- Super-tranche loss: 0%
- **Investor takes no loss**

**Scenario 2 (clustered losses):** All 20 defaults in one sub-portfolio
- That sub-portfolio: 12% loss (exceeds 4% subordination + 5% width)
- That sub-tranche: 100% wiped
- Super-tranche: 20% loss (that sub-tranche was 20% of super-portfolio)
- **Investor loses 20%**

Same number of defaults, vastly different outcomes. This is correlation risk at its most extreme.

---

## 48.12 Leveraged Super Senior: An Advanced Structure

### 48.12.1 Definition and Mechanics

A **leveraged super‑senior (LSS)** position gives an investor **leveraged exposure** to a super‑senior tranche. Instead of posting the full notional, the investor posts collateral/margin and takes a larger notional exposure; returns and losses are therefore magnified relative to posted capital.

Structure:
- Investor posts collateral/margin and is subject to MTM variation
- Notional exposure can be a multiple of posted collateral (deal‑specific leverage)
- Investor earns premium on the notional while the tranche survives
- Investor faces amplified MTM and (in extreme cases) credit loss

### 48.12.2 Why LSS Was Popular

LSS tends to look attractive when the market‑implied compensation for senior risk looks high relative to the investor’s view of tail default risk (i.e., “senior looks cheap”). Leverage can turn a small quoted spread into a meaningful return on posted collateral.

### 48.12.3 The Risk: Gap Risk

Even if the probability of **realized** credit loss is low, super‑senior MTM can be volatile because it is sensitive to spread levels and correlation assumptions. With leverage, an MTM move can exceed posted collateral — turning a “remote credit loss” trade into a **liquidity risk** trade.

To guard against dealer exposure in that scenario, LSS deal terms often include a **deleveraging trigger**. Common trigger linkages include:
1. The market super‑senior tranche spread, and/or
2. The number of defaults or total loss in the reference portfolio

Valuation note: to price a deleveraging trigger, you need the super‑senior tranche mark‑to‑market at the moment the deleveraging event is triggered.

> **Desk history note**
>
> A well‑known failure mode in 2008 was large collateral calls on super‑senior exposures as spreads and implied correlation repriced. The key lesson is that “remote credit loss” does not mean “small liquidity risk.”

---

## 48.13 Practical Notes

### 48.13.1 Practitioner Checklist

Before trading or booking a tranche, verify:

- [ ] **Reference portfolio definition**: Index series or bespoke list; weights/exposures
- [ ] **Loss definition**: How defaults and recoveries map into $L(t)$
- [ ] **Attachment/detachment**: As percent of portfolio notional
- [ ] **Tranche notional**: And whether quotes are "per USD 1 face" or in dollars
- [ ] **Premium terms**: Spread $S(A,D)$, upfront (if any), payment dates, day count
- [ ] **Settlement mechanics**: Cash vs physical; timing
- [ ] **Position direction**: Buying protection (pay premium, receive losses) or selling protection (receive premium, pay losses)?
- [ ] **Leverage**: If applicable, what is the effective notional exposure?

### 48.13.2 Common Pitfalls

1. **Confusing portfolio vs tranche notional**: The portfolio might be USD 1bn but your tranche notional is only USD 30mm (the width × portfolio notional)

2. **Mixing % and USD  units**: Always convert explicitly

3. **Misreading attachment/detachment**: They are percentages of the **reference portfolio** notional

4. **Forgetting premium is on outstanding notional**: After losses, premium base shrinks

5. **Assuming standard tranche grids are universal**: The 0-3, 3-7, 7-10, 10-15, 15-30 grid applies to CDX IG; iTraxx and HY indices have different attachment points

6. **Ignoring maximum portfolio loss**: With 40% recovery, losses can't exceed 60%; super-senior detachment above 60% has different risk profile

7. **Leverage confusion**: A small equity tranche position can carry many times its notional in index‑equivalent DV01/delta (depends on strikes and market state)

### 48.13.3 Verification Tests

**Bounds check:**
$$0 \le TL(L) \le w \text{ for all } L$$

**Monotonicity check:**
$$L_2 \ge L_1 \Rightarrow TL(L_2) \ge TL(L_1)$$

**Non-negative outstanding:**
$$w - TL(L) \ge 0$$

**Loss conservation** (for full capital structure):
$$\sum_m TL_m(L) = L \text{ when tranches are contiguous and span } [0, 1]$$

> **What Middle Office Sees:**
>
> If you're in risk or operations, you may see tranched positions on reports without fully understanding the exposure. Key things to know:
>
> 1. **Notional ≠ Risk:** tranche face is not comparable to index notional. Use DV01/delta (and correlation sensitivity) rather than notional.
>
> 2. **Mark-to-market volatility:** equity tranches can move multiple points in volatile markets. This can be “normal” for the product, not necessarily a booking error.
>
> 3. **Correlation is a risk factor:** Tranche P&L depends on both spreads AND correlation. If you see unexplained P&L, check if correlation moved.

---

## 48.14 Worked Examples

### Example 0: Par Running Spread and Spread PV01 (Toy, With Dates)

**Context**
- Price a 1-year single-tranche contract $[A,D]=[3\%,7\%]$ on a USD 100mm portfolio under a toy deterministic loss path.
- Why it matters: it connects **quote → cashflows → PV → a bump-defined risk number** without requiring a full correlation model.

**Timeline (illustrative; ignore holidays/business-day adjustments)**
- Trade/effective date: 2026-03-20
- Premium payment dates (quarterly): 2026-06-20, 2026-09-20, 2026-12-20, 2027-03-20
- Maturity: 2027-03-20

**Inputs**
- $N_{\text{port}}=USD 100\text{mm}$
- Tranche: $[3\%,7\%]$ so $w=4\%$ and $N_{\text{tr}}^{\text{face}}=w\,N_{\text{port}}=USD 4\text{mm}$
- Premium accrual: ACT/360-style quarterly approximation $\alpha=0.25$ each period
- Discounting: flat continuously compounded $r=5\%$ so:
  - $P(0,0.25)\approx 0.9876$, $P(0,0.5)\approx 0.9753$, $P(0,0.75)\approx 0.9632$, $P(0,1.0)\approx 0.9512$
- Assumed portfolio loss levels $L(t)$ at premium dates: 2.9%, 3.1%, 3.1%, 3.2%

**Outputs**
- Par running spread (no upfront): $S^*\approx 509\text{ bp}$
- Spread PV01 (bump $S$ by $+1$ bp, hold the assumed outstanding-notional path fixed):
  - protection seller: $+USD 378$ per 1 bp per USD 4mm tranche face
  - protection buyer: $-USD 378$ per 1 bp per USD 4mm tranche face

**Step-by-step**
1. Compute portfolio-scale tranche loss $TL(L(t_i))=\min(\max(L(t_i)-A,0),w)$.
2. Compute outstanding tranche notional $N_{\text{tr}}^{\text{out}}(t_i)=(w-TL(L(t_i)))N_{\text{port}}$.
3. Compute protection PV from incremental losses $\Delta TL\cdot N_{\text{port}}$.
4. Compute the “risky annuity” $ \mathcal{A}:=\sum_i \alpha\,P(0,t_i)\,N_{\text{tr}}^{\text{out}}(t_i)$.
5. Solve $S^*=PV_{\text{prot}}/\mathcal{A}$. Then $PV01_S \approx 10^{-4}\mathcal{A}$.

**Cashflows (protection buyer perspective; positive = receive)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-06-20 | $-0.25\,S^* \cdot USD 4.0\text{mm}\approx -USD 50{,}950$ | premium on full outstanding |
| 2026-09-20 | $+USD 100{,}000$ | tranche loss increases from 0 to 0.1% of portfolio |
| 2026-09-20 | $-0.25\,S^* \cdot USD 3.9\text{mm}\approx -USD 49{,}676$ | premium base reduced after loss |
| 2026-12-20 | $-0.25\,S^* \cdot USD 3.9\text{mm}\approx -USD 49{,}676$ | still outstanding USD 3.9mm |
| 2027-03-20 | $+USD 100{,}000$ | tranche loss increases from 0.1% to 0.2% of portfolio |
| 2027-03-20 | $-0.25\,S^* \cdot USD 3.8\text{mm}\approx -USD 48{,}403$ | premium on the remaining outstanding |

**P&L / Risk Interpretation**
- $S^*$ is “loss PV divided by risky annuity”: more expected loss $\Rightarrow$ higher par spread; faster amortization $\Rightarrow$ smaller annuity $\Rightarrow$ higher par spread.
- $PV01_S$ here is small because the trade is short-dated and the tranche notional is only USD 4mm. Scaling is linear in tranche face.
- “Hold fixed” matters: this $PV01_S$ is a **contract-spread bump** holding the (assumed) loss path fixed; if you bump index spreads and recalibrate a correlation surface, you will generally get a different number.

**Sanity Checks**
- Units check: $\alpha$ (years) × $S$ (per year) × notional (currency) → currency.
- Sign check: increasing $S$ increases PV for the premium receiver (protection seller), decreases PV for the premium payer (protection buyer).
- Limit check: if $\Delta TL=0$ everywhere (no losses), then $S^*=0$.

**Debug Checklist (When Your Result Looks Wrong)**
- Did you compute $TL$ as a **portfolio fraction** and then convert to dollars with $N_{\text{port}}$?
- Are you paying premium on **outstanding** notional (not original face)?
- Is the spread in **decimal per year** (e.g., 500 bp = 0.05) and is $1\text{ bp}=10^{-4}$?
- Are cashflow dates aligned with the assumed loss timing/settlement assumption?

### Common Setup

- **Portfolio notional:** $N_{\text{port}} = USD 100{,}000{,}000$ (USD 100mm)
- **Standard tranche ladder:** $[0, 3\%]$, $[3, 7\%]$, $[7, 10\%]$, $[10, 15\%]$, $[15, 30\%]$
- **Tranche loss formula:** $TL(L) = \min(\max(L - A, 0), D - A)$
- **Dollar loss:** $TL(L) = TL(L) \times N_{\text{port}}$

### Example 1: Dollar Attachment/Detachment

**Convert the standard tranche ladder to dollar terms:**

| Tranche | Attachment $A$ | Detachment $D$ | Width | Face Notional |
|---------|----------------|----------------|-------|---------------|
| $[0, 3\%]$ | USD 0mm | USD 3mm | USD 3mm | USD 3mm |
| $[3, 7\%]$ | USD 3mm | USD 7mm | USD 4mm | USD 4mm |
| $[7, 10\%]$ | USD 7mm | USD 10mm | USD 3mm | USD 3mm |
| $[10, 15\%]$ | USD 10mm | USD 15mm | USD 5mm | USD 5mm |
| $[15, 30\%]$ | USD 15mm | USD 30mm | USD 15mm | USD 15mm |

**Total face notional:** USD 3mm + USD 4mm + USD 3mm + USD 5mm + USD 15mm = USD 30mm (30% of portfolio)

### Example 2: Loss Within Equity Tranche

**Given:** $L = 1.5\%$, tranche $[0, 3\%]$

$$TL = \min(\max(0.015 - 0, 0), 0.03) = 0.015$$

- **Dollar tranche loss:** USD 1.5mm
- **Remaining notional:** $(0.03 - 0.015) \times USD 100\text{mm} = USD 1.5\text{mm}$
- **Tranche loss as % of tranche:** $0.015/0.03 = 50\%$

**Interpretation:** Half the equity tranche has been eroded. Premium payments will be based on the surviving USD 1.5mm.

### Example 3: Loss Below Mezzanine Attachment

**Given:** $L = 1.5\%$, tranche $[3, 7\%]$

$$TL = \min(\max(0.015 - 0.03, 0), 0.04) = 0$$

**Dollar tranche loss:** USD 0

**Explanation:** Portfolio loss has not reached the 3% attachment point. This tranche remains fully protected by the equity's subordination.

### Example 4: Loss Spanning Multiple Tranches

**Given:** $L = 5\%$ (USD 5mm portfolio loss)

| Tranche | Calculation | Dollar Loss | Status |
|---------|-------------|-------------|--------|
| $[0, 3\%]$ | $\min(0.05, 0.03) = 0.03$ | USD 3mm | **Wiped** |
| $[3, 7\%]$ | $\min(\max(0.05-0.03, 0), 0.04) = 0.02$ | USD 2mm | Partial |
| $[7, 10\%]$ | $\min(\max(0.05-0.07, 0), 0.03) = 0$ | USD 0 | Intact |

**Waterfall verification:** USD 3mm + USD 2mm = USD 5mm ✓

### Example 5: Large Loss Across Capital Structure

**Given:** $L = 12\%$ (USD 12mm portfolio loss)

| Tranche | Width | Tranche Loss | Dollar Loss | Status |
|---------|-------|--------------|-------------|--------|
| $[0, 3\%]$ | 3% | 3% | USD 3mm | **Wiped** |
| $[3, 7\%]$ | 4% | 4% | USD 4mm | **Wiped** |
| $[7, 10\%]$ | 3% | 3% | USD 3mm | **Wiped** |
| $[10, 15\%]$ | 5% | 2% | USD 2mm | Partial (remaining USD 3mm) |
| $[15, 30\%]$ | 15% | 0% | USD 0 | Intact |

**Total:** USD 3mm + USD 4mm + USD 3mm + USD 2mm = USD 12mm ✓

### Example 6: Incremental Loss Allocation

**Scenario:** Portfolio loss increases from $L_1 = 4\%$ to $L_2 = 6\%$ (incremental USD 2mm loss)

**At $L_1 = 4\%$:**
- Equity: TL = 3% → USD 3mm (wiped)
- Jr Mezz: TL = 1% → USD 1mm (remaining USD 3mm)

**At $L_2 = 6\%$:**
- Equity: TL = 3% → USD 3mm (unchanged, already wiped)
- Jr Mezz: TL = 3% → USD 3mm (remaining USD 1mm)

**Incremental allocation:**
- Equity: $\Delta TL = 0$ (cannot absorb more—wiped)
- Jr Mezz: $\Delta TL = 2\%$ → USD 2mm

All incremental losses flow to the mezzanine tranche.

### Example 7: Premium Payment Mechanics

**Setup:** Tranche $[3, 7\%]$ after portfolio loss $L = 5\%$ (from Example 4)

- **Remaining tranche notional:** USD 2mm
- **Contractual spread:** $S = 250$ bp = 0.025 per annum
- **Accrual period:** Quarterly, $\alpha = 0.25$

**Premium payment:**
$$PremPay = 0.25 \times 0.025 \times USD 2{,}000{,}000 = USD 12{,}500$$

**Before losses:** If notional had been full USD 4mm:
$$PremPay = 0.25 \times 0.025 \times USD 4{,}000{,}000 = USD 25{,}000$$

**Impact:** Premium halved due to loss erosion.

### Example 8: Default Ladder into a Mezzanine Tranche (Worked Illustration)

This is a simple “deterministic ladder” illustration: assume equal exposures and equal recovery so each default adds the same increment to portfolio loss.

**Setup:**
- Portfolio: 100 credits, each USD 10mm face (USD 1bn total)
- Tranche: $[3\%, 7\%]$, face USD 40mm, spread 250bp, 5-year maturity
- Recovery: 30% on all defaults
- Loss per default: $(1 - 0.30)/100 = 0.70\%$

**Defaults required to hit tranche:**
- First loss at default #5: Cumulative loss = $5 \times 0.70\% = 3.5\%$
- Tranche loss = $(3.5\% - 3.0\%)/4\% = 12.5\%$

**Tranche wiped at default #10:**
- Cumulative loss = $10 \times 0.70\% = 7.0\%$
- Tranche loss = $(7.0\% - 3.0\%)/4\% = 100\%$

This kind of ladder calculation is a useful sanity check before you do any real pricing (where default timing and loss severity are random).

### Example 9: Complete Capital Structure Loss Conservation

**Given:** $L = 25\%$, tranches span $[0, 30\%]$ with a residual $[30, 100\%]$

| Tranche | TL Calculation | Dollar Loss |
|---------|----------------|-------------|
| $[0, 3\%]$ | $\min(0.25, 0.03) = 0.03$ | USD 3mm |
| $[3, 7\%]$ | $\min(0.25-0.03, 0.04) = 0.04$ | USD 4mm |
| $[7, 10\%]$ | $\min(0.25-0.07, 0.03) = 0.03$ | USD 3mm |
| $[10, 15\%]$ | $\min(0.25-0.10, 0.05) = 0.05$ | USD 5mm |
| $[15, 30\%]$ | $\min(0.25-0.15, 0.15) = 0.10$ | USD 10mm |
| $[30, 100\%]$ | $\max(0.25-0.30, 0) = 0$ | USD 0mm |
| **Total** | | **USD 25mm** |

**Verification:** $USD 25\text{mm} = 25\% \times USD 100\text{mm}$ ✓

Loss conservation holds: sum of tranche losses equals portfolio loss.

### Example 10: Premium Calculation with Multiple Periods

**Scenario:** Track mezzanine $[3, 7\%]$ premium across four quarters with intervening defaults

| Quarter | Portfolio Loss $L$ | Tranche Loss | Outstanding | Premium (USD ) |
|---------|-------------------|--------------|-------------|--------------|
| Q1 start | 0% | 0% | USD 4.0mm | — |
| Q1 end | 2% | 0% | USD 4.0mm | $0.25 \times 0.02 \times 4.0\text{mm} = 20{,}000$ |
| Q2 end | 4% | 25% | USD 3.0mm | $0.25 \times 0.02 \times 3.0\text{mm} = 15{,}000$ |
| Q3 end | 5% | 50% | USD 2.0mm | $0.25 \times 0.02 \times 2.0\text{mm} = 10{,}000$ |
| Q4 end | 5% | 50% | USD 2.0mm | $0.25 \times 0.02 \times 2.0\text{mm} = 10{,}000$ |

(Using 200bp spread for illustration)

**Protection leg payments:** At Q2 default, protection seller pays $(4\%-3\%) \times USD 100\text{mm} = USD 1\text{mm}$. At Q3 default, pays $(5\%-4\%) \times USD 100\text{mm} = USD 1\text{mm}$.

### Example 11: Same Width, Different Attachment

**Compare:** $[0, 3\%]$ vs $[7, 10\%]$ (both 3% width) at $L = 5\%$

| Tranche | Calculation | Dollar Loss | Status |
|---------|-------------|-------------|--------|
| $[0, 3\%]$ | $\min(0.05, 0.03) = 0.03$ | USD 3mm | **Wiped** |
| $[7, 10\%]$ | $\max(0.05 - 0.07, 0) = 0$ | USD 0 | Intact |

**At $L = 8\%$:**

| Tranche | Calculation | Dollar Loss | Status |
|---------|-------------|-------------|--------|
| $[0, 3\%]$ | 0.03 | USD 3mm | Wiped (unchanged) |
| $[7, 10\%]$ | $\min(0.08 - 0.07, 0.03) = 0.01$ | USD 1mm | Partial |

**Key insight:** Higher attachment shifts loss exposure to more extreme scenarios.

---

## Summary

1. A **CDO** transforms portfolio credit risk into tranches with different risk-return profiles through structural subordination

2. **Cash CDOs** are funded structures using SPVs with complex waterfalls; **synthetic CDOs** are unfunded bilateral derivatives with simpler payoffs

3. **Portfolio loss $L(t)$** is the single state variable determining all tranche outcomes; with recoveries, maximum loss is $L_{\max}=\sum_i w_i(1-R_i)$ (and $L_{\max}=1-R$ under a common recovery)

4. **Attachment $A$** is the "deductible"—losses below this level don't affect the tranche

5. **Detachment $D$** is the "limit"—the tranche is fully wiped when $L \ge D$

6. **Tranche loss function:** $TL(L) = \min(\max(L - A, 0), D - A)$ — equivalent to a call spread on portfolio loss

7. **Premium leg** pays on surviving notional; **protection leg** pays incremental tranche losses

8. **Standard tranches** (0-3, 3-7, 7-10, 10-15, 15-30 for CDX IG) provide liquid hedging and calibration tools

9. **Leverage varies dramatically**: equity can be “double‑digit” leveraged (e.g., ~18× in the worked example), while very senior tranches can have much smaller spread sensitivity depending on state and definition

10. **Conservation of expected loss**: Sum of expected tranche losses = expected portfolio loss (a no-arbitrage constraint)

11. Tranches are **correlation products**: same expected portfolio loss gives different tranche values depending on default clustering

12. **Senior investors prefer low correlation** (diversification protects them); **equity investors prefer high correlation** (more scenarios with zero loss)

13. The **2008 crisis** demonstrated the dangers of underestimating correlation, particularly in ABS CDO structures

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| CDO | Structure that tranches portfolio credit risk | Creates securities with different risk profiles from single pool |
| STCDO | Single-tranche synthetic CDO traded as a bilateral derivative | Cleanest way to learn tranche mechanics before models |
| Portfolio loss $L(t)$ | Cumulative fractional loss on reference portfolio | Single state variable for all tranche payoffs |
| Attachment $A$ | Loss level below which tranche has zero loss | Determines subordination/protection |
| Detachment $D$ | Loss level at which tranche is fully wiped | Determines tranche capacity |
| Tranche width $w$ | $D - A$, the tranche's loss-absorbing capacity | Determines maximum tranche loss |
| Tranche loss function | $\min(\max(L-A, 0), D-A)$ | Maps portfolio loss to tranche loss |
| Premium leg | Running spread payments on outstanding tranche notional | Premium base amortizes as losses accrue |
| Protection leg | Payments equal to incremental tranche loss | Drives jump risk and default-loss cashflows |
| Par running spread $S^*$ | Spread that makes PV $\approx 0$ (given a model/assumptions) | Turns “expected loss” into a quote object |
| Risky annuity | $\sum_i \alpha_i P(0,t_i)\,\mathbb{E}[N_{\text{tr}}^{\text{out}}(t_i)]$ | Sets spread PV01 magnitude for a contract-spread bump |
| Spread PV01 | PV change for a +1bp bump to contractual spread $S$ (with a stated hold-fixed rule) | Only meaningful when the bumped object is explicit |
| Leverage ratio | Systemic spread PV01 of tranche ÷ systemic spread PV01 of equivalent index notional | Explains why small tranche face can carry huge spread risk |
| Conservation of expected loss | $\sum_m E[TL_m] = E[L]$ | No-arbitrage constraint on pricing |
| Standard tranches | Market-traded attachment points | Liquid hedging and correlation calibration |
| Correlation product | Value depends on default clustering | Why tranches require correlation models |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $N_{\text{port}}$ | Portfolio notional | currency |
| $L(t)$ | Cumulative portfolio loss fraction | fraction of $N_{\text{port}}$ |
| $A, D$ | Attachment / detachment | fractions of $N_{\text{port}}$ |
| $w=D-A$ | Tranche width | fraction of $N_{\text{port}}$ |
| $L(T,K_1,K_2)$ | Fractional tranche loss | fraction of tranche notional, $\in[0,1]$ |
| $TL(L)$ | Tranche loss (portfolio scale) | fraction of portfolio notional |
| $N_{\text{tr}}^{\text{face}}$ | Tranche face notional | $w\,N_{\text{port}}$ currency |
| $N_{\text{tr}}^{\text{out}}(t)$ | Outstanding tranche notional | $(w-TL(L(t)))N_{\text{port}}$ currency |
| $S(A,D)$ | Running tranche spread | decimal per year |
| $\alpha_i$ | Premium accrual factor | year fraction (contract-defined) |
| $P(0,t)$ | Discount factor | unitless |
| $PV01_S$ | Spread PV01 (contract-spread bump) | currency per 1 bp per stated face; bump $S\to S+1\text{bp}$ with stated hold-fixed rule |

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does $L(t)$ represent? | Fraction of portfolio notional lost due to defaults by time $t$ |
| 2 | Define attachment point $A$ | Loss level below which tranche loss is zero |
| 3 | Define detachment point $D$ | Loss level at/above which tranche is fully wiped |
| 4 | What is tranche width $w$? | $w = D - A$ |
| 5 | Write the tranche loss formula | $TL(L) = \min(\max(L - A, 0), D - A)$ |
| 6 | Who pays the premium leg? | Tranche protection buyer pays protection seller |
| 7 | On what base are tranche premiums paid? | Surviving/outstanding tranche notional |
| 8 | What triggers protection-leg payments? | Credit events that change tranche loss function |
| 9 | Why are tranches called correlation products? | Risk depends on tendency of credits to default together |
| 10 | What is "subordination" in tranche language? | Attachment point $A$—portfolio loss absorbed by junior tranches |
| 11 | What happens to equity tranche when $L \gt D$? | Wiped—loss equals full width |
| 12 | What is the standard equity tranche for CDX IG? | 0-3% |
| 13 | Do senior investors prefer high or low correlation? | Low correlation (diversification protects them) |
| 14 | Do equity investors prefer high or low correlation? | High correlation (more "all survive" scenarios) |
| 15 | What is a synthetic CDO? | Unfunded bilateral tranche contract referencing virtual portfolio |
| 16 | Key difference: cash CDO vs synthetic CDO? | Cash is funded with SPV; synthetic is unfunded bilateral |
| 17 | What is STCDO? | Single-Tranche Synthetic CDO |
| 18 | Why is STCDO waterfall "trivial"? | Simple payoff function vs complex cash flow waterfall |
| 19 | What is loss conservation? | Sum of tranche losses equals portfolio loss (for full capital structure) |
| 20 | How does a default affect premium payments? | Surviving notional decreases → premium payments decrease |
| 21 | Standard mezzanine tranche for CDX IG? | 3-7% or 7-10% |
| 22 | What is "virtual reference portfolio"? | List of reference entities—doesn't physically exist as CDS book |
| 23 | Why did ABS CDO AAA tranches fail in 2008? | Underlying mortgage defaults were highly correlated |
| 24 | Common premium accrual convention for index tranches? | Often quarterly payments with ACT/360-style day count (contract governs). |
| 25 | If $L = 2\%$, what is TL for $[3, 7\%]$ tranche? | Zero (below attachment) |
| 26 | If $L = 12\%$, is $[7, 10\%]$ wiped? | Yes (12% > 10% detachment) |
| 27 | Formula for outstanding tranche notional in dollars? | $(w - TL(L)) \times N_{\text{port}}$ |
| 28 | What makes tranches different from portfolio CDS? | Non-linear payoff: loss only within $[A, D]$ window |
| 29 | Who tends to hold super-senior risk? | Often banks/insurers; varies by era, regulation, and market regime. |
| 30 | What's needed to price tranches beyond this chapter? | Loss distribution model (Chapter 49-50) |
| 31 | What is the leverage ratio of a tranche? | Systemic spread PV01 of tranche ÷ systemic spread PV01 of equivalent index notional. Can be >> 1 for equity and much smaller for very senior tranches. |
| 32 | What is the conservation of expected loss property? | Sum of expected losses across all tranches = expected loss of underlying portfolio. A no-arbitrage constraint. |
| 33 | Why does the equity tranche want high correlation? | High correlation means more "all survive" or "all default" outcomes; fewer middle outcomes that wipe out exactly the equity. |
| 34 | What is a Leveraged Super Senior (LSS)? | Leveraged exposure to super-senior risk: investor posts collateral, takes larger notional, and may face triggers/unwind if MTM moves. |
| 35 | What distinguishes a cash CDO from a synthetic CDO? | Cash CDO holds physical bonds with active manager and complex waterfall. Synthetic CDO references CDS with no manager and simple loss-linked payments. |

---

## Mini Problem Set

### Questions

1. For $N_{\text{port}} = USD 500\text{mm}$, convert tranche $[7, 10]$ to dollar attachment/detachment and face notional.

2. With $L = 2.5\%$, compute TL and outstanding notional for equity $[0, 3]$.

3. With $L = 2.5\%$, compute TL for mezzanine $[3, 7]$. Explain why.

4. With $L = 8\%$, compute tranche losses for $[0, 3]$, $[3, 7]$, and $[7, 10]$.

5. Portfolio loss goes from 9% to 12%. Compute incremental losses for $[7, 10]$ and $[10, 15]$.

6. Tranche $[3, 7]$ has spread 400bp and quarterly accrual. Outstanding notional is USD 1.5mm. Compute next premium payment.

7. Prove algebraically that $TL(L) = \max(L-A, 0) - \max(L-D, 0)$ equals $\min(\max(L-A, 0), D-A)$.

8. In words, explain why equity tranche investors prefer high correlation.

9. Portfolio has 50 equal-weight names on USD 100mm. Four default with 40% recovery. Compute $L$.

10. For $[0, 3]$ and $[10, 15]$, sketch $TL(L)$ vs $L \in [0, 20\%]$.

11. A tranche has width 5% and has incurred 2% portfolio-scale loss. What fraction of tranche notional remains?

12. Explain what breaks in "loss conservation" if there's a gap between tranches (e.g., $[0, 3]$ and $[5, 7]$ with nothing at $[3, 5]$).

13. For a protection seller, what happens to future premium receipts after tranche losses occur?

14. Why might the standard equity tranche on CDX HY be 0-10% while CDX IG is 0-3%?

15. List three sources of model risk in tranche pricing.

16. **(Leverage Calculation)** A 0-3% equity tranche has systemic spread PV01 of USD 550,000 per USD 10mm tranche face. The underlying index has systemic spread PV01 of USD 30,000 per USD 10mm index notional. Calculate the leverage ratio and interpret.

17. **(Conservation of Expected Loss)** A portfolio has expected loss of 3% over 5 years. You are quoted:
    - 0-3% tranche: 25% expected loss (as fraction of tranche notional)
    - 3-7% tranche: 5% expected loss
    - 7-100% tranche: ? expected loss

    Calculate the expected loss of the 7-100% tranche required for no-arbitrage.

---

### Solution Sketches (Selected)

**1.** $[7, 10]$: $A = 0.07 \times 500 = USD 35\text{mm}$, $D = USD 50\text{mm}$, face = $3\% \times 500 = USD 15\text{mm}$

**2.** $[0, 3]$, $L = 2.5\%$: $TL = 2.5\%$, outstanding = $3\% - 2.5\% = 0.5\%$ of portfolio = $0.5\text{mm}$ on $100\text{mm}$

**3.** $[3, 7]$: $L = 2.5\% \lt A = 3\%$ → $TL = 0$. Subordination from equity protects mezzanine.

**4.** $L = 8\%$: $[0, 3]$ loss = 3% (wiped), $[3, 7]$ loss = 4% (wiped), $[7, 10]$ loss = 1% (remaining 2%)

**5.** $9\% \to 12\%$: At 9%, $[7, 10]$ has 2% loss; at 10% it's 3% (wiped). At 10%, $[10, 15]$ starts. From 10% to 12%, 2% goes to $[10, 15]$. So: $[7, 10]$ gets 1%, $[10, 15]$ gets 2%.

**6.** Premium = $\alpha \cdot S \cdot N_{\text{out}} = 0.25 \times 0.04 \times 1.5\text{mm} = USD 15{,}000$

**7.** Three cases: (i) $L \le A$: both formulas give 0. (ii) $A \lt L \lt D$: both give $L - A$. (iii) $L \ge D$: max-max gives $(L-A) - (L-D) = D-A$; min-max gives $D-A$.

**8.** High correlation means outcomes are bimodal: either most credits survive (equity untouched) or most default (equity wiped anyway). More probability of the "all survive" scenario benefits equity.

**9.** Each name has USD 100mm / 50 = USD 2mm notional. Loss per default = USD 2mm × 60% = USD 1.2mm. Four defaults = USD 4.8mm, so $L = 4.8\text{mm} / 100\text{mm} = 4.8\%$.

**10.** For $[0, 3]$: TL rises linearly from 0 to 3% as L goes 0→3%, then flat at 3%. For $[10, 15]$: TL = 0 for L ≤ 10%, rises linearly 10%→15%, then flat at 5%.

**16.** Leverage $= \frac{USD 550{,}000}{USD 30{,}000} \approx 18.33\times$. Under this “systemic spread PV01” definition, the tranche has ~18× the spread sensitivity of the same stated notional of the index.

**17.** By conservation: $3\% \times 25\% + 4\% \times 5\% + 93\% \times \text{EL}(7\text{-}100\%) = 3\%$

$0.75\% + 0.20\% + 93\% \times \text{EL} = 3\%$

$\text{EL}(7\text{-}100\%) = (3\% - 0.95\%) / 93\% = 2.05\% / 93\% \approx 0.022\%$ (very small, as expected for super-senior)

---

## References

- Dominic O’Kane, *Modelling Single-name and Multi-name Credit Derivatives* (CDO/tranche mechanics; tranche risk and leverage; structured credit extensions)
- John C. Hull, *Options, Futures, and Other Derivatives* (synthetic CDOs; standard portfolios and single-tranche trading)
- John C. Hull, *Risk Management and Financial Institutions* (ABS and ABS CDO illustration; crisis-era lessons)
- McNeil, Frey, Embrechts, *Quantitative Risk Management* (option-like representations of tranche payoffs)
- Pierre Privault, *Notes on Financial Risk and Analytics* (credit derivatives; tranche premium/protection leg PV representations)
- Robert Kosowski and Salih N. Neftci, *Principles of Financial Engineering* (leveraged super-senior notes; structured credit instruments)
