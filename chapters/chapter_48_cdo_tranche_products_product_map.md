# Chapter 48: CDO / Tranche Products — What They Are (Product Map)

---

## Introduction

In September 2008, when Lehman Brothers collapsed and AIG required an $85 billion government bailout, one instrument was at the center of the chaos: the collateralized debt obligation. AAA-rated CDO tranches—supposedly as safe as U.S. Treasuries—had suffered devastating losses, exposing how profoundly the market had misunderstood these products. The lesson was expensive: you cannot price or risk-manage what you do not understand.

A collateralized debt obligation takes a single pool of credit risk and transforms it into six different products—products so different in their risk-return profiles that an insurance company might eagerly buy one while a hedge fund bets aggressively on another. This alchemy of credit risk redistribution lies at the heart of structured credit markets.

The mechanism is deceptively simple: instead of sharing losses proportionally across all investors, a CDO allocates losses sequentially. The first layer of investors absorbs all losses until wiped out; only then does the next layer begin to suffer. This "subordination" structure creates securities ranging from equity-like first-loss pieces earning spreads of 1000+ basis points to AAA-rated senior tranches that only lose money in catastrophic scenarios.

Understanding tranche mechanics is essential for several reasons. First, tranches are "correlation products"—their value depends critically on whether defaults cluster or remain dispersed, making them sensitive to risks invisible in single-name credit analysis (see Chapter 50). Second, the 2007-2008 financial crisis demonstrated what happens when tranche risks are misunderstood: AAA-rated tranches on subprime mortgages suffered severe losses because investors underestimated how correlated housing defaults would be. Third, tranches remain actively traded on standardized indices (CDX, iTraxx), providing efficient exposure to portfolio credit risk.

This chapter provides the **product map**—the mechanics of how tranches work, without diving into pricing models. We cover:

- **48.1 CDO Product Taxonomy**: Cash vs synthetic, funded vs unfunded, bespoke vs standard
- **48.2 Portfolio Loss as the State Variable**: The single number that drives everything
- **48.3 Tranche Attachment and Detachment**: How to read tranche "strikes"
- **48.4 The Tranche Loss Function**: The mathematical heart of waterfall mechanics
- **48.5 Premium and Protection Legs**: Who pays what and when
- **48.6 Standard Index Tranches**: Market-traded attachment points
- **48.7 Tranche Leverage**: Why equity tranches are 18x leveraged
- **48.8 Conservation of Expected Loss**: The fundamental arbitrage constraint
- **48.9 Worked Examples**: From mechanics to dollars
- **48.10 CDOs as Correlation Products**: Why tranches need correlation models
- **48.11 The 2008 Crisis: What Went Wrong**: Historical lessons
- **48.12 Leveraged Super Senior**: An advanced structure

Pricing (Chapter 49) and correlation frameworks (Chapter 50) build directly on these foundations.

---

## Conventions & Notation

| Convention | Description |
|------------|-------------|
| **Attachment/detachment units** | All "%" attachment/detachment and losses are fractions of original portfolio notional $N_{\text{port}}$ unless explicitly stated otherwise |
| **Portfolio loss** | $L(t)$ is cumulative loss from inception to time $t$, expressed as a fraction of $N_{\text{port}}$ (so $0 \le L(t) \le 1$) |
| **Tranche notation** | Tranche is denoted $[A, D]$ with attachment $A$ and detachment $D$, $0 \le A < D \le 1$ |
| **Dollar conversion** | $x\%$ of $N_{\text{port}}$ means $x/100 \times N_{\text{port}}$ dollars |
| **Premium accrual** | Premium accrual factor $\alpha$ uses the deal's day count; where we need a number, we use quarterly $\alpha = 0.25$ for illustration |

| Symbol | Definition |
|--------|------------|
| $N_{\text{port}}$ | Portfolio notional (USD) |
| $L(t)$ | Fractional portfolio loss by time $t$ |
| $A$, $D$ | Tranche attachment and detachment (fractions of $N_{\text{port}}$) |
| $w = D - A$ | Tranche width (fraction of $N_{\text{port}}$) |
| $\text{TL}(L)$ | Tranche loss expressed as fraction of portfolio notional |
| $L_{\text{tr}}(t; A, D)$ | Fractional tranche loss as fraction of tranche notional (ranges in $[0, 1]$) |
| $N_{\text{tr}}^{\text{face}} = w \cdot N_{\text{port}}$ | Tranche face notional in dollars |
| $N_{\text{tr}}^{\text{out}}(t)$ | Outstanding tranche notional in dollars |
| $S(A, D)$ | Contractual tranche spread (per annum) |
| $\alpha_i$ | Accrual year fraction for the $i$-th premium period |
| $R$ | Recovery rate |
| $L_{\max}$ | Maximum possible portfolio loss = $1 - R$ |

---

## 48.1 CDO Product Taxonomy: From Cash Flow to Synthetic

### 48.1.1 What Is a CDO?

O'Kane defines collateralized debt obligations as "securities whose payments are linked to the incidence of default on an underlying portfolio of credit risky assets. This is done in a way which transforms the credit risk of the underlying portfolio into a set of securities with different credit profiles."

The core insight is **structural subordination**: rather than sharing losses proportionally, a CDO ranks investors in a "waterfall" so that junior tranches absorb losses first, protecting senior tranches until subordination is exhausted. This single mechanism transforms a portfolio of BBB credits into a menu of securities ranging from AAA to unrated.

> **Why "Equity" for the First-Loss Tranche?**
>
> O'Kane clarifies the terminology: "By analogy to the capital structure of a company, the term 'equity' simply means the first loss or most subordinate part of the CDO structure. It should not be confused with equity in the sense of stocks." Just as corporate equity holders are paid last (after bondholders), CDO equity holders absorb losses first.

### 48.1.2 Cash Flow CDOs: The Traditional Structure

In a traditional cash flow CDO, investors purchase securities in a **funded** form—they pay cash upfront, typically at par. O'Kane explains the structure:

> "The sale proceeds are used to purchase the collateral portfolio of credit risky assets, typically bonds and loans, which is then sold into a structure known as a special purpose vehicle or SPV. The SPV then issues the CDO securities."

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

O'Kane notes that "the waterfall, described in the priority of payments section of the issuing documents of the CDO is often quite complex. For example, it may contain triggers which cause the more senior tranches to amortise early if there have been a large number of defaults on the underlying portfolio."

Cash flow CDO waterfalls typically include:
- **Interest waterfall**: Determines how coupon payments flow to tranches
- **Principal waterfall**: Determines how principal repayments flow
- **OC (Overcollateralization) tests**: Triggers that divert cash if collateral deteriorates
- **IC (Interest coverage) tests**: Triggers based on interest coverage ratios
- **Reinvestment period**: Manager can reinvest principal proceeds for specified period

### 48.1.3 Synthetic CDOs: Unfunded Derivative Structures

A **synthetic CDO** or **single-tranche synthetic CDO (STCDO)** differs fundamentally from the cash flow structure. O'Kane highlights the key differences:

1. **No SPV**: "There is a bilateral contract between a dealer and an investor."
2. **Synthetic reference portfolio**: "The reference portfolio is linked to 50-150 reference entities with each reference entity equating to a position in a CDS on that reference entity."
3. **Unfunded structure**: "The contract typically costs nothing to enter into"—no upfront principal payment.
4. **Single-tranche**: "Only one CDO security, or tranche, is issued. There is no need to issue the other tranches."
5. **Faster execution**: "The STCDO can typically be issued within a few days of the initial enquiry."
6. **Simplified waterfall**: "The STCDO waterfall is trivial when compared to the waterfall of a typical cash flow CDO."
7. **Full customization**: The investor can select portfolio composition, attachment/detachment, maturity, and rating.

**Why synthetic became dominant for traded tranches:** The unfunded nature eliminates funding risk—the investor is not exposed to changes in their own borrowing costs over the trade's life. O'Kane notes that "default swaps are unfunded transactions" which makes them "favored by investors who have funding costs above Libor while bonds are favoured by those who fund below."

The dealer perspective is also critical: "The dealer retains the credit risk of the STCDO... Unlike a CDS, the risk held by the dealer is a CDO tranche and so includes an exposure to all of the credits in the reference portfolio and their default correlation. These risks must be hedged dynamically."

### 48.1.4 Full Capital Structure vs Single-Tranche

In a **full capital structure** deal, all tranches from equity to super-senior are issued and sold to different investors. O'Kane explains: "We call these types of CDO full capital structure deals since the entire capital structure—meaning all of the CDO tranche securities—are sold, leaving the issuer with no risk."

In single-tranche trading, only one tranche is created, with the dealer dynamically hedging the remaining risk. O'Kane notes a subtle but important point: "The reference portfolio does not actually exist as a distinct portfolio of CDS on the dealer's book. It is simply a list of reference entities whose defaults affect the payments of the STCDO via the waterfall. It can be thought of as a virtual reference portfolio."

### 48.1.5 Cash CDO vs Synthetic CDO: Detailed Comparison

| Feature | Cash Flow CDO | Synthetic CDO (STCDO) |
|---------|---------------|----------------------|
| **Funding** | Funded—investor pays upfront | Unfunded—no initial payment |
| **Underlying** | Actual bonds/loans in SPV | Virtual reference portfolio of CDS |
| **Counterparty risk** | SPV bankruptcy remoteness | Bilateral with dealer |
| **Waterfall complexity** | Complex, deal-specific triggers (OC/IC tests, reinvestment) | Simple payoff function |
| **Manager role** | Active management, reinvestment decisions | None (static reference portfolio) |
| **Customization** | Limited (compromise across investors) | Full (single investor) |
| **Execution speed** | Months | Days |
| **Liquidity** | Generally illiquid | Index tranches are liquid |
| **Default settlement** | Sell distressed bonds from portfolio | Cash settle via CDS auction process |
| **Modeling complexity** | High (manager behavior, reinvestment) | Lower (just model default correlation) |

O'Kane emphasizes the key simplification: "The STCDO waterfall is trivial when compared to the waterfall of a typical cash flow CDO." This is why nearly all tranche modeling focuses on synthetic structures.

---

## 48.2 Portfolio Loss $L(t)$: The State Variable

### 48.2.1 Definition

The portfolio loss $L(t)$ is the single state variable that determines all tranche payoffs. O'Kane defines it as:

$$L(T) = \frac{1}{N_c} \sum_{i=1}^{N_c} (1 - R_i) \mathbf{1}_{\tau_i \le T}$$

where:
- $N_c$ is the number of credits in the portfolio
- $R_i$ is the recovery rate for credit $i$
- $\tau_i$ is the default time of credit $i$
- $\mathbf{1}_{\tau_i \le T}$ equals 1 if credit $i$ has defaulted by time $T$, 0 otherwise

This formula assumes equal face value exposures (each credit is $1/N_c$ of the portfolio). Each default contributes $(1-R_i)/N_c$ to the portfolio loss.

### 48.2.2 Maximum Portfolio Loss and Recovery

An important bound on portfolio loss is the **maximum possible loss**:

$$\boxed{L_{\max} = 1 - \bar{R}}$$

where $\bar{R}$ is the average recovery rate. If all credits default with 40% recovery, the portfolio can lose at most 60% of notional.

O'Kane notes this has implications for senior tranches: "the senior tranche upper strike... reduced from 100% to 100% - 40% = 60%." A super-senior tranche with detachment above $L_{\max}$ can never be fully wiped out by defaults—though mark-to-market volatility can still be substantial.

### 48.2.3 Example: Loss from Defaults

Consider a portfolio of 100 credits, each with $10 million face value (total $1 billion). If 3 credits default with recovery rates of 40%, 35%, and 50%:

| Default | Face Value | Recovery | Loss per Name | Contribution to $L$ |
|---------|------------|----------|---------------|---------------------|
| 1 | $10mm | 40% | $6mm | 0.60% |
| 2 | $10mm | 35% | $6.5mm | 0.65% |
| 3 | $10mm | 50% | $5mm | 0.50% |
| **Total** | | | **$17.5mm** | **1.75%** |

The portfolio loss is $L = 1.75\%$ of notional, or $17.5 million.

### 48.2.4 Why $L(t)$ Is All That Matters

For synthetic tranche pricing, O'Kane emphasizes: "The mechanics of the STCDO make it much easier to model than the cash flow CDO. This is because the premium and protection payments at time $t$ are purely a function of the tranche loss $L(t, K_1, K_2)$ which in turn is simply a function of the portfolio loss $L(t)$."

This simplification is profound: we don't need to track which credits defaulted, only the cumulative percentage loss. All tranche mechanics reduce to functions of a single scalar $L(t)$.

---

## 48.3 Attachment and Detachment: Reading Tranche "Strikes"

### 48.3.1 Formal Definitions

O'Kane provides precise definitions:

> "$K_1$: This is the **attachment point**, also known as the subordination or lower strike of the tranche. This is the percentage loss on the reference portfolio below which the tranche loss is zero. As soon as $L(T) > K_1$, the tranche loss is non-zero."

> "$K_2$: This is the **detachment point**, also known as the upper strike. The quantity $K_2 - K_1$ is the tranche width. If $L(T) \ge K_2$, the tranche loss is 100%."

In our notation, $A = K_1$ (attachment) and $D = K_2$ (detachment), with tranche width $w = D - A$.

### 48.3.2 Intuition: Deductible and Limit

Think of attachment as an **insurance deductible**: the amount of portfolio loss that must occur before the tranche starts paying. Think of detachment as the **coverage limit**: once losses reach this level, the tranche is exhausted.

| If $L$ is... | Then tranche $[A, D]$... |
|--------------|-------------------------|
| Below $A$ | Has zero loss (protected by subordination) |
| Between $A$ and $D$ | Absorbs losses dollar-for-dollar |
| At or above $D$ | Is fully wiped out |

### 48.3.3 Tranche Labels by Position

Standard market terminology names tranches by their position in the capital structure:

| Label | Position | Typical Spread | Investor Type |
|-------|----------|----------------|---------------|
| **Equity** | $[0, A_1]$ | Highest (500-2000+ bp) | Hedge funds, CDO managers |
| **Junior Mezzanine** | $[A_1, A_2]$ | High | Credit funds |
| **Senior Mezzanine** | $[A_2, A_3]$ | Moderate | Asset managers |
| **Senior** | $[A_3, A_4]$ | Low | Insurance companies |
| **Super Senior** | $[A_4, D]$ | Very low (10-30 bp) | Banks, mono-line insurers |

O'Kane notes that "insurance companies mainly use credit derivatives as a form of investment which sits on the asset side of their business. They are principally sellers of credit protection and tend to prefer highly rated credits such as the senior tranches of CDOs."

---

## 48.4 The Tranche Loss Function: Mathematical Heart of the Waterfall

### 48.4.1 The Piecewise Linear Payoff

O'Kane provides the formal tranche loss function. For portfolio loss $L(T)$ and tranche with attachment $K_1$ and detachment $K_2$:

$$\boxed{L(T, K_1, K_2) = \frac{\max(L(T) - K_1, 0) - \max(L(T) - K_2, 0)}{K_2 - K_1}}$$

This gives the fractional loss of the tranche—i.e., as a percentage of tranche notional, ranging from 0 to 1 (or 0% to 100%).

### 48.4.2 Portfolio-Scale Tranche Loss

In our notation, we also work with tranche loss expressed as a fraction of **portfolio** notional:

$$\boxed{\text{TL}(L) = \min\bigl(\max(L - A, 0), D - A\bigr)}$$

The relationship is:
$$L_{\text{tr}}(T; A, D) = \frac{\text{TL}(L(T))}{D - A}$$

### 48.4.3 Piecewise Derivation

The tranche loss function has three regimes:

**Regime 1: $L \le A$ (Below Attachment)**

Portfolio losses have not reached the tranche's deductible. Subordination fully protects the tranche:
$$\text{TL}(L) = 0$$

**Regime 2: $A < L < D$ (Within Tranche)**

The tranche absorbs losses dollar-for-dollar:
$$\text{TL}(L) = L - A$$

Every additional percentage point of portfolio loss flows directly to this tranche.

**Regime 3: $L \ge D$ (At or Above Detachment)**

The tranche has absorbed its full width and is wiped out:
$$\text{TL}(L) = D - A = w$$

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

O'Kane's Figure 12.5 shows exactly this piecewise linear relationship: "Tranche loss $L(T, K_1, K_2)$ as a function of the percentage portfolio loss $L(T)$."

### 48.4.5 The Call Spread / Put Spread Analogy

The tranche loss function can be written in several mathematically equivalent forms that reveal its option-like nature:

**Call-spread form** (most common in practice):
$$\boxed{\text{TL}(L) = (L - A)^+ - (L - D)^+}$$

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

McNeil (QRM) notes that tranche notional can be written as:

$$f_\kappa(l) = (K_\kappa - l)^+ - (K_{\kappa-1} - l)^+$$

"Such positions are sometimes called a **put spread**"—a long put at the upper strike minus a short put at the lower strike. This options-based formulation is useful for understanding tranche risk as analogous to option Greeks.

**Alternative form** (using min functions):
$$\text{TL}(L) = \min(L, D) - \min(L, A)$$

This form reveals that any tranche loss equals the difference between two "equity" (0-K) tranche losses—a property exploited in the base correlation framework (Chapter 50).

---

## 48.5 Premium and Protection Legs

### 48.5.1 Premium Leg: Payments on Surviving Notional

The premium leg consists of periodic payments from the tranche protection buyer to the seller. O'Kane writes:

> "The coupons on the premium leg are paid on the surviving notional of the tranche according to a schedule which is typically quarterly and uses an Actual 360 day count convention. Per $1 of face value, the size of a single premium payment at time $t_i$ is given by:
> $$\Delta(t_{i-1}, t_i) \cdot S(K_1, K_2) \cdot (1 - L(t_i, K_1, K_2))$$"

The term $(1 - L(t_i, K_1, K_2))$ is the surviving fraction of tranche notional. **As losses erode the tranche, premium payments decline.**

In dollar terms, for a tranche with face notional $N_{\text{tr}}^{\text{face}}$:

$$\boxed{\text{PremPay}_i = \alpha_i \cdot S(A, D) \cdot N_{\text{tr}}^{\text{out}}(t_i)}$$

where:
- $\alpha_i$ is the accrual year fraction
- $S(A, D)$ is the contractual spread (decimal per annum)
- $N_{\text{tr}}^{\text{out}}(t_i) = (w - \text{TL}(L(t_i))) \cdot N_{\text{port}}$ is outstanding notional

### 48.5.2 Protection Leg: Loss Payments

The protection leg compensates the tranche protection buyer for losses. O'Kane explains:

> "The protection leg consists of the loss payments made by the investor to the dealer to cover default losses on the tranche. The size of the loss is simply the change in the value of the tranche loss function $L(t, K_1, K_2)$, which will only change if there is a credit event on the reference portfolio such that after the default $K_1 < L(T) \le K_2$."

Protection payments occur only when defaults push $L(t)$ into the tranche's region $[A, D]$. Each time $L$ increases within this region, the protection seller pays the increment in tranche loss.

### 48.5.3 Loss Settlement Timing

O'Kane notes an important timing consideration:

> "Premium payments also take into account the timing of the default of credits in the reference portfolio. As a result, a default loss which occurs immediately before a premium payment has no effect on the premium paid. However, if the default and loss occurred at the start of the accrual period, the entire coupon is based on the reduced surviving notional."

There is a "direct analogy between the outstanding tranche notional and the survival probability in a standard CDS which means that we can incorporate the tranche amortisation in the same way as we handle the payment of premium accrued at default on a standard CDS."

### 48.5.4 Senior Tranche Adjustment for Maximum Loss

O'Kane (Ch 12.5.4) highlights an important practical consideration for senior tranches. When the detachment point exceeds the maximum possible portfolio loss, the tranche effectively has "super-senior" protection that can never be touched.

Consider a portfolio of $N_c$ credits with assumed recovery rate $R$. The maximum possible portfolio loss is:
$$L_{\max} = (1 - R)$$

For a tranche with detachment $D > L_{\max}$, the tranche can never be fully wiped out. The effective tranche width for loss purposes is:
$$w_{\text{eff}} = \min(D, L_{\max}) - A$$

O'Kane notes: "If the detachment point is above the maximum loss level, there will be an adjustment for the part of the width which cannot be reached by losses on the reference portfolio."

**Example:** If $R = 40\%$ (so $L_{\max} = 60\%$) and the super-senior tranche is $[30\%, 100\%]$:
- Nominal width: 70%
- Effective width: $60\% - 30\% = 30\%$
- The 40% above $L_{\max}$ is "unreachable"—this affects both pricing and risk calculations

This adjustment matters primarily for super-senior tranches with very high detachment points.

---

## 48.6 Standard Index Tranches

### 48.6.1 Market-Traded Attachment Points

O'Kane describes the emergence of standardized tranches:

> "In mid-2003 an interdealer market appeared for a standardised set of synthetic CDO tranches. These took as their underlying reference portfolio the CDS index portfolios described in Chapter 10. The market for these standard index tranches has grown significantly since then."

Standard tranches trade on the major investment-grade and high-yield indices with the following attachment and detachment points:

| Tranche | CDX IG NA | iTraxx Europe | CDX NA HY |
|---------|-----------|---------------|-----------|
| | $K_1$ — $K_2$ | $K_1$ — $K_2$ | $K_1$ — $K_2$ |
| **Equity** | 0% — 3% | 0% — 3% | 0% — 10% |
| **Junior Mezzanine** | 3% — 7% | 3% — 6% | 10% — 15% |
| **Senior Mezzanine** | 7% — 10% | 6% — 9% | 15% — 25% |
| **Senior** | 10% — 15% | 9% — 12% | 25% — 35% |
| **Super Senior** | 15% — 30% | 12% — 22% | 35% — 100% |

*Source: O'Kane, Table 12.4*

### 48.6.2 Why These Specific Attachment Points?

The attachment points are chosen to reflect the expected loss characteristics of each index:

**CDX IG (0-3% equity):** With 125 investment-grade credits and ~40% expected recovery, each default contributes approximately 0.6% to portfolio loss. The 0-3% equity tranche captures roughly the first 4-5 defaults. This is calibrated so that the equity tranche has significant expected loss while mezzanine tranches remain investment-grade.

**CDX HY (0-10% equity):** High-yield portfolios have much higher expected loss—defaults are more frequent and often clustered. The 0-10% equity tranche reflects this higher default probability, ensuring the equity absorbs a meaningful portion of expected loss before mezzanine tranches are impacted.

**iTraxx Europe:** Similar logic to CDX IG, though slight differences in attachment points (3-6% vs 3-7% for junior mezz) reflect different market conventions and portfolio characteristics.

### 48.6.3 Why Standard Tranches Matter

O'Kane emphasizes the significance:

> "The standard tranches have exactly the same mechanics as the STCDOs already discussed. They differ from bespoke STCDOs in that they are much more liquid and so present much tighter bid-offer spreads. Since they are quoted in the dealer market, their pricing is transparent. As we will see in later chapters, these prices allow us to extract the implied correlations the market is assigning to the reference portfolios."

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

O'Kane notes the pricing relationship: bespoke tranches are typically priced by mapping to the standard base correlation curve derived from liquid standard tranche prices. The mapping methods (TLP, ATM correlation scaling) are covered in Chapter 50.

### 48.6.5 Liquidity Hierarchy

In practice, liquidity varies across the capital structure:

| Tranche | Relative Liquidity | Typical Market Participants |
|---------|-------------------|---------------------------|
| Equity | High | Hedge funds, prop desks |
| Junior Mezz | Moderate | Credit funds, hedge funds |
| Senior Mezz | Moderate | Insurance, asset managers |
| Senior | Lower | Insurance companies |
| Super Senior | Variable | Banks, monolines (pre-crisis) |

The equity tranche is often the most liquid because it offers the most "pure" correlation exposure and is used heavily for correlation trading strategies.

> **Deep Dive: The Equity Tranche ("Toxic Waste" or "Gold Mine"?)**
>
> *   **The Risk**: You are the First Loss. If *anyone* defaults, you pay. You are effectively short a "Digital Put" on the economy.
> *   **The Reward**: You get paid upfront (often 30-50 points) + 500bp running. You effectively buy the spread at a massive discount.
> *   **The Correlation View**: You *love* Correlation. Why?
>     *   **Low Correlation**: Defaults happen randomly. Someone will surely default. You die.
>     *   **High Correlation**: It's binary. Either Everyone Survives (You get rich) OR Everyone Dies (You lose, but you were going to lose anyway).
>     *   Equity investors pray for "Feast or Famine." Systemic risk is their friend because it increases the chance of the "Feast" scenario.

---

## 48.7 Tranche Leverage: Why Equity is 18x Leveraged

### 48.7.1 Leverage Ratio Definition

O'Kane defines the tranche leverage ratio in Chapter 17:

$$\boxed{\text{Leverage} = \frac{\Delta_s}{F(K_1, K_2)}}$$

where $\Delta_s$ is the systemic delta (spread sensitivity of the tranche in terms of equivalent index notional) and $F(K_1, K_2)$ is the face value of the tranche.

More intuitively:

$$\boxed{\text{Leverage} = \frac{\text{Systemic DV01 of Tranche}}{\text{Systemic DV01 of Equivalent Notional of Index}}}$$

### 48.7.2 O'Kane's Leverage Table

O'Kane provides leverage ratios for standard tranches (Table 17.3, at portfolio spread 60bp and correlation 25%):

| Tranche | Leverage Ratio | Interpretation |
|---------|----------------|----------------|
| 0-3% | **18.37** | Equity is 18x leveraged on the index |
| 3-7% | 9.20 | Junior mezz is 9x leveraged |
| 7-10% | 4.52 | Senior mezz is 4.5x leveraged |
| 10-15% | 2.12 | Senior is 2x leveraged |
| 15-30% | **0.39** | Super senior is **deleveraged** (0.4x) |

O'Kane observes: "The equity tranche is highly leveraged with a leverage ratio of 18.37, while the senior tranche is deleveraged with a leverage ratio of 0.39."

### 48.7.3 Understanding Leverage Intuitively

Why is the equity tranche so leveraged? Consider:

- The equity tranche (0-3%) represents 3% of portfolio notional
- But it absorbs the *first* 3% of losses—which have the highest probability
- The systemic delta of the 0-3% tranche is $691 million per $1,250 million reference portfolio—meaning it contains roughly *half* the total spread risk despite being only 2.4% of notional

Conversely, the super-senior tranche (15-30%):
- Represents 15% of notional
- But only absorbs losses in catastrophic scenarios
- Has systemic delta of only $73 million—far less than its pro-rata share

> **Desk Reality: What 18x Leverage Means**
>
> If you buy $10 million notional of the 0-3% equity tranche, your spread sensitivity is equivalent to holding approximately $183.7 million of the index. A 10bp move in index spreads produces ~18x the P&L you'd see on an equivalent index position.
>
> This is why equity tranches are the domain of hedge funds and correlation desks, not traditional credit investors. The leverage amplifies both gains and losses dramatically.

### 48.7.4 Leverage Changes with Spread Level

O'Kane notes that leverage is not constant:

> "As the portfolio spread increases, the risk of the equity tranche increases but the equity delta starts to fall. At the same time, the deltas of the mezzanine and senior tranches start to increase."

When spreads widen significantly:
- Equity tranche becomes "more equity-like" (higher probability of wipeout)
- Mezzanine tranches become "equity-like" (loss distribution shifts through them)
- Senior tranches take on more risk

This dynamic leverage is one reason tranche risk management is complex.

---

## 48.8 Conservation of Expected Loss: The Arbitrage Constraint

### 48.8.1 The Fundamental Property

O'Kane proves a crucial no-arbitrage constraint (Chapter 12.7.1):

> "Suppose we buy protection on $m = 1, \ldots, M$ contiguous tranches with strikes $[K_{m-1}, K_m]$ where $K_0 = 0$ and $K_M = 1$. The value of the sum of the expected losses to some horizon time $T$ is given by:
> $$\text{Total expected loss} = \mathbb{E}[L(T)]$$
> which is the expected loss of the reference portfolio."

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

O'Kane explains the economic implication:

> "As a result, we conclude that the present value of the protection legs of the STCDOs, which is the discounted expected losses, summed across the capital structure is identical to owning the protection legs of all of the CDS in the reference portfolio. This makes sense since a trade which involved buying all the STCDO protection legs and selling all the CDS protection legs would be perfectly hedged as any default on the reference portfolio would involve an equal and opposite payment from the tranches to the CDS."

> **Why This Matters for Pricing**
>
> If a pricing model violates conservation of expected loss, you could:
> 1. Buy protection on ALL tranches (covering 0-100%)
> 2. Sell protection on the index
> 3. Earn riskless profit if model over/under-prices total expected loss
>
> O'Kane notes this is a key criticism of using different correlations for different tranches (compound correlation): "Failure to conserve the expected loss" creates arbitrage opportunities. The base correlation framework (Chapter 50) preserves this constraint by construction.

### 48.8.4 Premium Legs Don't Perfectly Offset

O'Kane provides an important caveat:

> "However, buying protection with tranches across the capital structure and selling protection on all of the CDS is not an exact hedge since the timing and sizes of the tranche and CDS premium flows do not match exactly."

He gives a concrete example: if a credit defaults, the equity tranche notional falls significantly (reducing its premium base by a large amount), but the CDS premium loss is just that single name's spread. The protection legs offset perfectly; the premium legs do not.

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
│  EQUITY (0-3%)          │  FIRST LOSS. Takes first ~4-5 defaults    │
│  ─────────────────────────────────────────────────────────────────  │
│  Width = 3%             │  Leveraged 18x+                           │
└─────────────────────────────────────────────────────────────────────┘

LOSSES FLOW UPWARD: Portfolio losses fill from bottom (Equity) up.
```

### 48.9.2 Loss Flow Through the Capital Structure

When portfolio losses occur, they flow through the capital structure in order:

```
Portfolio Loss $L(t)
        │
        ▼
┌───────────────────────────────────────────────────────────┐
│                    Capital Structure                       │
│                                                           │
│   ┌─────────────────┐                                     │
│   │ [0%, 3%] Equity │ ◄── Losses hit here FIRST           │
│   │   Width = 3%    │     Until $3mm absorbed             │
│   └────────┬────────┘                                     │
│            │ Overflow when equity wiped                   │
│            ▼                                              │
│   ┌─────────────────┐                                     │
│   │ [3%, 7%] Jr Mezz│ ◄── SECOND layer                    │
│   │   Width = 4%    │     Until $4mm more absorbed        │
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

On $100mm portfolio: Equity absorbs first $3mm, Jr Mezz next $4mm, etc.
```

---

## 48.10 CDOs as Correlation Products

### 48.10.1 Why Correlation Matters

O'Kane explains the fundamental insight:

> "CDOs are credit correlation products. The best way to see this is to introduce the portfolio loss distribution. The portfolio loss distribution tells us the probability of a certain loss on the portfolio at some future time horizon."

The **expected portfolio loss** is independent of correlation (it depends only on individual default probabilities and recoveries). But the **shape of the loss distribution**—and hence tranche values—depends critically on how defaults cluster.

### 48.10.2 Low Correlation vs High Correlation

O'Kane describes the effects:

> "When the correlation is zero, the credits are independent and do not tend to survive or default together. As a result the losses are distributed close to the expected loss in a range between 0% and just over 10% loss. This suggests that the risk of senior tranches with an attachment point above 10% is very low."

> "In the high correlation case credits become more likely to default and survive together. We therefore see that there is a reasonable probability of the losses exceeding 10%. This suggests that there is an increased probability of the senior tranche incurring a loss."

> "We also see that the probability of very few defaults is high when the correlation is high. There is therefore an increased probability of the equity tranche incurring no loss."

### 48.10.3 Correlation Preferences by Tranche

O'Kane concludes with the key insight:

> "We can therefore conclude that **senior investors prefer low correlation** and **equity investors prefer high correlation** portfolios."

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
- Equity tranche almost certainly takes some loss (expected loss ≈ 5% for typical IG portfolio)

**ρ = 100% (Perfect Correlation):**
- Credits default together or survive together
- Portfolio loss is binary: either 0% or ~60% (with 40% recovery)
- **All tranches have the same risk profile!** (They all face the same binary outcome)
- Equity tranche has substantial probability of zero loss
- Senior tranche faces meaningful probability of total wipeout

This is the essence of why tranches are "correlation products": the same expected loss can produce vastly different outcomes depending on whether losses are dispersed or clustered.

---

## 48.11 The 2008 Crisis: What Went Wrong

### 48.11.1 The ABS CDO Structure

Hull (Risk Management and Financial Institutions, Chapter 6) explains the structure that became toxic:

> "Portfolios of loans are packaged into tranches which are then sold to investors... The originate-to-distribute model got out of control during the 2000 to 2006 period. Banks relaxed their mortgage lending standards and the credit quality of the instruments being originated declined sharply."

The particularly problematic structure was the **ABS CDO**—a CDO backed by tranches of other asset-backed securities (often subprime mortgage-backed securities). Hull illustrates:

**Layer 1: Subprime Mortgages → ABS**
- Pools of subprime mortgages securitized into ABS
- ABS tranched into senior (AAA), mezzanine (BBB), equity

**Layer 2: ABS Tranches → ABS CDO**
- Mezzanine tranches of many ABSs pooled together
- This pool tranched again into senior (AAA), mezzanine, equity

Hull provides a stark warning: "The AAA-rated tranche of the ABS CDO in Figure 6.4 is much more risky [than it appears]. It will get paid the promised return if losses on the underlying portfolios are 10% or less."

### 48.11.2 What Went Wrong

> **Historical Note: CDOs and the 2008 Crisis**
>
> The credit crisis of 2007-2008 was deeply connected to CDO products, particularly "ABS CDOs"—CDOs backed by tranches of other securitizations (CDO-squared structures).
>
> **What went wrong:**
>
> 1. **Correlation underestimation:** Models assumed low default correlation in subprime mortgages. Housing was viewed as a local market—defaults in Nevada shouldn't affect Florida. When housing prices fell *nationally* (not just locally), correlations spiked and "AAA" tranches suffered massive losses.
>
> 2. **The thin margin for error:** Hull notes that ABS CDO AAA tranches only survived if underlying losses stayed below ~10%. This narrow margin proved fatal. At 15% underlying losses, the AAA tranche of an ABS CDO was completely wiped out.
>
> 3. **Rating agency failures:** Senior tranches of ABS CDOs were rated AAA despite being backed by *mezzanine* tranches of subprime MBS—a classic "garbage in, garbage out" problem. The rating agencies' correlation assumptions were too low.
>
> 4. **Liquidity evaporation:** When marks became uncertain, the entire market for complex CDO products seized up. Bid-ask spreads exploded from basis points to points.
>
> 5. **Model risk:** The Gaussian copula became "the formula that killed Wall Street" (a dramatic oversimplification, but it captured public sentiment). More precisely, the problem was using models calibrated to benign periods in a stress scenario.

### 48.11.3 The CDO-Squared Trap

O'Kane describes the CDO-squared structure:

> "The CDO squared is an extension of the standard CDO tranche concept in which the reference portfolio is itself a portfolio of single tranche CDOs. The effect of this double layer of tranching is to further leverage the spread premium embedded in the reference portfolios."

The danger is extreme sensitivity to loss clustering:

O'Kane's example: Consider a CDO-squared with five sub-tranches, each with 4% subordination and 5% width. If 20 credits default:

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

O'Kane describes the leveraged super-senior (LSS) tranche:

> "As its name suggests, the leveraged super-senior (LSS) tranche is a product in which the investor assumes a leveraged exposure to the super-senior part of the capital structure."

Structure:
- Investor posts collateral (say, 10% of notional exposure)
- Takes 10x leveraged exposure to super-senior tranche
- Earns leveraged premium (if super-senior pays 10bp, investor earns 100bp on their collateral)
- Faces leveraged losses if super-senior takes losses

### 48.12.2 Why LSS Was Popular

O'Kane explains the appeal:

> "This is a trade which is usually popular when the senior tranches of an index appear 'cheap', i.e. given where the index swap is trading, the combined risk implied by the spreads of the equity, mezzanine and senior tranches appear low and so imply a higher spread to the senior tranche than usual. Given that the actuarial risk of senior tranches is low, this situation becomes very interesting to investors who exploit the leverage to be paid a relatively large coupon for taking exposure to a very low risk and highly rated part of the capital structure."

### 48.12.3 The Risk: Gap Risk

> "Although the default risk of the leveraged super-senior tranche is low, it can exhibit mark-to-market volatility. For this reason, the structure requires the investor to post an upfront amount of collateral. Given the leverage of the trade, it is possible for the mark-to-market value of the trade to fall by more than the value of the collateral."

To protect dealers, LSS structures typically include **triggers** that force deleveraging if:
1. The market super-senior spread widens beyond a threshold, or
2. Losses on the reference portfolio exceed a level

> **Practitioner Note: AIG and Leveraged Super Senior**
>
> AIG Financial Products was a major writer of super-senior protection, often in leveraged form. When credit spreads widened dramatically in 2008, the mark-to-market losses on these positions far exceeded the collateral posted. This created the "collateral calls" that ultimately required federal intervention. The lesson: super-senior risk is real, and leverage magnifies it catastrophically.

---

## 48.13 Practical Notes

### 48.13.1 Practitioner Checklist

Before trading or booking a tranche, verify:

- [ ] **Reference portfolio definition**: Index series or bespoke list; weights/exposures
- [ ] **Loss definition**: How defaults and recoveries map into $L(t)$
- [ ] **Attachment/detachment**: As percent of portfolio notional
- [ ] **Tranche notional**: And whether quotes are "per $1 face" or in dollars
- [ ] **Premium terms**: Spread $S(A,D)$, upfront (if any), payment dates, day count
- [ ] **Settlement mechanics**: Cash vs physical; timing
- [ ] **Position direction**: Buying protection (pay premium, receive losses) or selling protection (receive premium, pay losses)?
- [ ] **Leverage**: If applicable, what is the effective notional exposure?

### 48.13.2 Common Pitfalls

1. **Confusing portfolio vs tranche notional**: The portfolio might be $1bn but your tranche notional is only $30mm (the width × portfolio notional)

2. **Mixing % and $ units**: Always convert explicitly

3. **Misreading attachment/detachment**: They are percentages of the **reference portfolio** notional

4. **Forgetting premium is on outstanding notional**: After losses, premium base shrinks

5. **Assuming standard tranche grids are universal**: The 0-3, 3-7, 7-10, 10-15, 15-30 grid applies to CDX IG; iTraxx and HY indices have different attachment points

6. **Ignoring maximum portfolio loss**: With 40% recovery, losses can't exceed 60%; super-senior detachment above 60% has different risk profile

7. **Leverage confusion**: A $10mm equity tranche position has ~$180mm of index-equivalent spread risk

### 48.13.3 Verification Tests

**Bounds check:**
$$0 \le \text{TL}(L) \le w \text{ for all } L$$

**Monotonicity check:**
$$L_2 \ge L_1 \Rightarrow \text{TL}(L_2) \ge \text{TL}(L_1)$$

**Non-negative outstanding:**
$$w - \text{TL}(L) \ge 0$$

**Loss conservation** (for full capital structure):
$$\sum_m \text{TL}_m(L) = L \text{ when tranches are contiguous and span } [0, 1]$$

> **What Middle Office Sees:**
>
> If you're in risk or operations, you may see tranched positions on reports without fully understanding the exposure. Key things to know:
>
> 1. **Notional ≠ Risk:** A $100mm equity tranche has ~18x the spread risk of $100mm of the index. Don't compare notionals directly.
>
> 2. **Mark-to-market volatility:** Equity tranches can move 10-20 points in a day during spread moves. This is normal, not a break.
>
> 3. **Correlation is a risk factor:** Tranche P&L depends on both spreads AND correlation. If you see unexplained P&L, check if correlation moved.

---

## 48.14 Worked Examples

### Common Setup

- **Portfolio notional:** $N_{\text{port}} = \$100{,}000{,}000$ ($100mm)
- **Standard tranche ladder:** $[0, 3\%]$, $[3, 7\%]$, $[7, 10\%]$, $[10, 15\%]$, $[15, 30\%]$
- **Tranche loss formula:** $\text{TL}(L) = \min(\max(L - A, 0), D - A)$
- **Dollar loss:** $\$\text{TL}(L) = \text{TL}(L) \times N_{\text{port}}$

### Example 1: Dollar Attachment/Detachment

**Convert the standard tranche ladder to dollar terms:**

| Tranche | Attachment $A$ | Detachment $D$ | Width | Face Notional |
|---------|----------------|----------------|-------|---------------|
| $[0, 3\%]$ | $0 | $3mm | $3mm | $3mm |
| $[3, 7\%]$ | $3mm | $7mm | $4mm | $4mm |
| $[7, 10\%]$ | $7mm | $10mm | $3mm | $3mm |
| $[10, 15\%]$ | $10mm | $15mm | $5mm | $5mm |
| $[15, 30\%]$ | $15mm | $30mm | $15mm | $15mm |

**Total face notional:** $3mm + $4mm + $3mm + $5mm + $15mm = $30mm (30% of portfolio)

### Example 2: Loss Within Equity Tranche

**Given:** $L = 1.5\%$, tranche $[0, 3\%]$

$$\text{TL} = \min(\max(0.015 - 0, 0), 0.03) = 0.015$$

- **Dollar tranche loss:** $1.5mm
- **Remaining notional:** $(0.03 - 0.015) \times \$100\text{mm} = \$1.5\text{mm}$
- **Tranche loss as % of tranche:** $0.015/0.03 = 50\%$

**Interpretation:** Half the equity tranche has been eroded. Premium payments will be based on the surviving $1.5mm.

### Example 3: Loss Below Mezzanine Attachment

**Given:** $L = 1.5\%$, tranche $[3, 7\%]$

$$\text{TL} = \min(\max(0.015 - 0.03, 0), 0.04) = 0$$

**Dollar tranche loss:** $0

**Explanation:** Portfolio loss has not reached the 3% attachment point. This tranche remains fully protected by the equity's subordination.

### Example 4: Loss Spanning Multiple Tranches

**Given:** $L = 5\%$ ($5mm portfolio loss)

| Tranche | Calculation | Dollar Loss | Status |
|---------|-------------|-------------|--------|
| $[0, 3\%]$ | $\min(0.05, 0.03) = 0.03$ | $3mm | **Wiped** |
| $[3, 7\%]$ | $\min(\max(0.05-0.03, 0), 0.04) = 0.02$ | $2mm | Partial |
| $[7, 10\%]$ | $\min(\max(0.05-0.07, 0), 0.03) = 0$ | $0 | Intact |

**Waterfall verification:** $3mm + $2mm = $5mm ✓

### Example 5: Large Loss Across Capital Structure

**Given:** $L = 12\%$ ($12mm portfolio loss)

| Tranche | Width | Tranche Loss | Dollar Loss | Status |
|---------|-------|--------------|-------------|--------|
| $[0, 3\%]$ | 3% | 3% | $3mm | **Wiped** |
| $[3, 7\%]$ | 4% | 4% | $4mm | **Wiped** |
| $[7, 10\%]$ | 3% | 3% | $3mm | **Wiped** |
| $[10, 15\%]$ | 5% | 2% | $2mm | Partial (remaining $3mm) |
| $[15, 30\%]$ | 15% | 0% | $0 | Intact |

**Total:** $3mm + $4mm + $3mm + $2mm = $12mm ✓

### Example 6: Incremental Loss Allocation

**Scenario:** Portfolio loss increases from $L_1 = 4\%$ to $L_2 = 6\%$ (incremental $2mm loss)

**At $L_1 = 4\%$:**
- Equity: TL = 3% → $3mm (wiped)
- Jr Mezz: TL = 1% → $1mm (remaining $3mm)

**At $L_2 = 6\%$:**
- Equity: TL = 3% → $3mm (unchanged, already wiped)
- Jr Mezz: TL = 3% → $3mm (remaining $1mm)

**Incremental allocation:**
- Equity: $\Delta\text{TL} = 0$ (cannot absorb more—wiped)
- Jr Mezz: $\Delta\text{TL} = 2\%$ → $2mm

All incremental losses flow to the mezzanine tranche.

### Example 7: Premium Payment Mechanics

**Setup:** Tranche $[3, 7\%]$ after portfolio loss $L = 5\%$ (from Example 4)

- **Remaining tranche notional:** $2mm
- **Contractual spread:** $S = 250$ bp = 0.025 per annum
- **Accrual period:** Quarterly, $\alpha = 0.25$

**Premium payment:**
$$\text{PremPay} = 0.25 \times 0.025 \times \$2{,}000{,}000 = \$12{,}500$$

**Before losses:** If notional had been full $4mm:
$$\text{PremPay} = 0.25 \times 0.025 \times \$4{,}000{,}000 = \$25{,}000$$

**Impact:** Premium halved due to loss erosion.

### Example 8: O'Kane's Detailed STCDO Example

O'Kane provides a complete worked example which we replicate here.

**Setup:**
- Portfolio: 100 credits, each $10mm face ($1bn total)
- Tranche: $[3\%, 7\%]$, face $40mm, spread 250bp, 5-year maturity
- Recovery: 30% on all defaults
- Loss per default: $(1 - 0.30)/100 = 0.70\%$

**Defaults required to hit tranche:**
- First loss at default #5: Cumulative loss = $5 \times 0.70\% = 3.5\%$
- Tranche loss = $(3.5\% - 3.0\%)/4\% = 12.5\%$

**Tranche wiped at default #10:**
- Cumulative loss = $10 \times 0.70\% = 7.0\%$
- Tranche loss = $(7.0\% - 3.0\%)/4\% = 100\%$

This matches O'Kane's Table 12.3 showing the full payment schedule.

### Example 9: Complete Capital Structure Loss Conservation

**Given:** $L = 25\%$, tranches span $[0, 30\%]$ with a residual $[30, 100\%]$

| Tranche | TL Calculation | Dollar Loss |
|---------|----------------|-------------|
| $[0, 3\%]$ | $\min(0.25, 0.03) = 0.03$ | $3mm |
| $[3, 7\%]$ | $\min(0.25-0.03, 0.04) = 0.04$ | $4mm |
| $[7, 10\%]$ | $\min(0.25-0.07, 0.03) = 0.03$ | $3mm |
| $[10, 15\%]$ | $\min(0.25-0.10, 0.05) = 0.05$ | $5mm |
| $[15, 30\%]$ | $\min(0.25-0.15, 0.15) = 0.10$ | $10mm |
| $[30, 100\%]$ | $\max(0.25-0.30, 0) = 0$ | $0mm |
| **Total** | | **$25mm** |

**Verification:** $\$25\text{mm} = 25\% \times \$100\text{mm}$ ✓

Loss conservation holds: sum of tranche losses equals portfolio loss.

### Example 10: Premium Calculation with Multiple Periods

**Scenario:** Track mezzanine $[3, 7\%]$ premium across four quarters with intervening defaults

| Quarter | Portfolio Loss $L$ | Tranche Loss | Outstanding | Premium ($) |
|---------|-------------------|--------------|-------------|--------------|
| Q1 start | 0% | 0% | $4.0mm | — |
| Q1 end | 2% | 0% | $4.0mm | $0.25 \times 0.02 \times 4.0\text{mm} = 20{,}000$ |
| Q2 end | 4% | 25% | $3.0mm | $0.25 \times 0.02 \times 3.0\text{mm} = 15{,}000$ |
| Q3 end | 5% | 50% | $2.0mm | $0.25 \times 0.02 \times 2.0\text{mm} = 10{,}000$ |
| Q4 end | 5% | 50% | $2.0mm | $0.25 \times 0.02 \times 2.0\text{mm} = 10{,}000$ |

(Using 200bp spread for illustration)

**Protection leg payments:** At Q2 default, protection seller pays $(4\%-3\%) \times \$100\text{mm} = \$1\text{mm}$. At Q3 default, pays $(5\%-4\%) \times \$100\text{mm} = \$1\text{mm}$.

### Example 11: Same Width, Different Attachment

**Compare:** $[0, 3\%]$ vs $[7, 10\%]$ (both 3% width) at $L = 5\%$

| Tranche | Calculation | Dollar Loss | Status |
|---------|-------------|-------------|--------|
| $[0, 3\%]$ | $\min(0.05, 0.03) = 0.03$ | $3mm | **Wiped** |
| $[7, 10\%]$ | $\max(0.05 - 0.07, 0) = 0$ | $0 | Intact |

**At $L = 8\%$:**

| Tranche | Calculation | Dollar Loss | Status |
|---------|-------------|-------------|--------|
| $[0, 3\%]$ | 0.03 | $3mm | Wiped (unchanged) |
| $[7, 10\%]$ | $\min(0.08 - 0.07, 0.03) = 0.01$ | $1mm | Partial |

**Key insight:** Higher attachment shifts loss exposure to more extreme scenarios.

---

## Summary

1. A **CDO** transforms portfolio credit risk into tranches with different risk-return profiles through structural subordination

2. **Cash CDOs** are funded structures using SPVs with complex waterfalls; **synthetic CDOs** are unfunded bilateral derivatives with simpler payoffs

3. **Portfolio loss $L(t)$** is the single state variable determining all tranche outcomes; maximum loss is bounded by $L_{\max} = 1 - R$

4. **Attachment $A$** is the "deductible"—losses below this level don't affect the tranche

5. **Detachment $D$** is the "limit"—the tranche is fully wiped when $L \ge D$

6. **Tranche loss function:** $\text{TL}(L) = \min(\max(L - A, 0), D - A)$ — equivalent to a call spread on portfolio loss

7. **Premium leg** pays on surviving notional; **protection leg** pays incremental tranche losses

8. **Standard tranches** (0-3, 3-7, 7-10, 10-15, 15-30 for CDX IG) provide liquid hedging and calibration tools

9. **Leverage varies dramatically**: Equity ~18x, Super-senior ~0.4x

10. **Conservation of expected loss**: Sum of expected tranche losses = expected portfolio loss (a no-arbitrage constraint)

11. Tranches are **correlation products**: same expected portfolio loss gives different tranche values depending on default clustering

12. **Senior investors prefer low correlation** (diversification protects them); **equity investors prefer high correlation** (more scenarios with zero loss)

13. The **2008 crisis** demonstrated the dangers of underestimating correlation, particularly in ABS CDO structures

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| CDO | Structure that tranches portfolio credit risk | Creates securities with different risk profiles from single pool |
| Portfolio loss $L(t)$ | Cumulative fractional loss on reference portfolio | Single state variable for all tranche payoffs |
| Attachment $A$ | Loss level below which tranche has zero loss | Determines subordination/protection |
| Detachment $D$ | Loss level at which tranche is fully wiped | Determines tranche capacity |
| Tranche width $w$ | $D - A$, the tranche's loss-absorbing capacity | Determines maximum tranche loss |
| Tranche loss function | $\min(\max(L-A, 0), D-A)$ | Maps portfolio loss to tranche loss |
| Leverage ratio | Systemic DV01 of tranche ÷ DV01 of equivalent index | Equity ~18x, super-senior ~0.4x |
| Conservation of expected loss | $\sum_m E[\text{TL}_m] = E[L]$ | No-arbitrage constraint on pricing |
| Standard tranches | Market-traded attachment points | Liquid hedging and correlation calibration |
| Correlation product | Value depends on default clustering | Why tranches require correlation models |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does $L(t)$ represent? | Fraction of portfolio notional lost due to defaults by time $t$ |
| 2 | Define attachment point $A$ | Loss level below which tranche loss is zero |
| 3 | Define detachment point $D$ | Loss level at/above which tranche is fully wiped |
| 4 | What is tranche width $w$? | $w = D - A$ |
| 5 | Write the tranche loss formula | $\text{TL}(L) = \min(\max(L - A, 0), D - A)$ |
| 6 | Who pays the premium leg? | Tranche protection buyer pays protection seller |
| 7 | On what base are tranche premiums paid? | Surviving/outstanding tranche notional |
| 8 | What triggers protection-leg payments? | Credit events that change tranche loss function |
| 9 | Why are tranches called correlation products? | Risk depends on tendency of credits to default together |
| 10 | What is "subordination" in tranche language? | Attachment point $A$—portfolio loss absorbed by junior tranches |
| 11 | What happens to equity tranche when $L > D$? | Wiped—loss equals full width |
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
| 24 | What day count does tranche premium typically use? | Actual/360 |
| 25 | If $L = 2\%$, what is TL for $[3, 7\%]$ tranche? | Zero (below attachment) |
| 26 | If $L = 12\%$, is $[7, 10\%]$ wiped? | Yes (12% > 10% detachment) |
| 27 | Formula for outstanding tranche notional in dollars? | $(w - \text{TL}(L)) \times N_{\text{port}}$ |
| 28 | What makes tranches different from portfolio CDS? | Non-linear payoff: loss only within $[A, D]$ window |
| 29 | Who typically invests in super-senior tranches? | Banks, insurance companies, mono-line insurers |
| 30 | What's needed to price tranches beyond this chapter? | Loss distribution model (Chapter 49-50) |
| 31 | What is the leverage ratio of a tranche? | Systemic DV01 of tranche ÷ Systemic DV01 of equivalent notional of index. Equity ~18x, super senior ~0.4x. |
| 32 | What is the conservation of expected loss property? | Sum of expected losses across all tranches = expected loss of underlying portfolio. A no-arbitrage constraint. |
| 33 | Why does the equity tranche want high correlation? | High correlation means more "all survive" or "all default" outcomes; fewer middle outcomes that wipe out exactly the equity. |
| 34 | What is a Leveraged Super Senior (LSS)? | Super senior tranche with leveraged notional exposure. Investor posts margin and takes 10x+ notional. High carry, catastrophic tail risk. |
| 35 | What distinguishes a cash CDO from a synthetic CDO? | Cash CDO holds physical bonds with active manager and complex waterfall. Synthetic CDO references CDS with no manager and simple loss-linked payments. |

---

## Mini Problem Set

### Questions

1. For $N_{\text{port}} = \$500\text{mm}$, convert tranche $[7, 10]$ to dollar attachment/detachment and face notional.

2. With $L = 2.5\%$, compute TL and outstanding notional for equity $[0, 3]$.

3. With $L = 2.5\%$, compute TL for mezzanine $[3, 7]$. Explain why.

4. With $L = 8\%$, compute tranche losses for $[0, 3]$, $[3, 7]$, and $[7, 10]$.

5. Portfolio loss goes from 9% to 12%. Compute incremental losses for $[7, 10]$ and $[10, 15]$.

6. Tranche $[3, 7]$ has spread 400bp and quarterly accrual. Outstanding notional is $1.5mm. Compute next premium payment.

7. Prove algebraically that $\text{TL}(L) = \max(L-A, 0) - \max(L-D, 0)$ equals $\min(\max(L-A, 0), D-A)$.

8. In words, explain why equity tranche investors prefer high correlation.

9. Portfolio has 50 equal-weight names on $100mm. Four default with 40% recovery. Compute $L$.

10. For $[0, 3]$ and $[10, 15]$, sketch $\text{TL}(L)$ vs $L \in [0, 20\%]$.

11. A tranche has width 5% and has incurred 2% portfolio-scale loss. What fraction of tranche notional remains?

12. Explain what breaks in "loss conservation" if there's a gap between tranches (e.g., $[0, 3]$ and $[5, 7]$ with nothing at $[3, 5]$).

13. For a protection seller, what happens to future premium receipts after tranche losses occur?

14. Why might the standard equity tranche on CDX HY be 0-10% while CDX IG is 0-3%?

15. List three sources of model risk in tranche pricing.

16. **(Leverage Calculation)** A 0-3% equity tranche has systemic DV01 of $550,000 per $10mm notional. The underlying index has systemic DV01 of $30,000 per $10mm notional. Calculate the leverage ratio and interpret.

17. **(Conservation of Expected Loss)** A portfolio has expected loss of 3% over 5 years. You are quoted:
    - 0-3% tranche: 25% expected loss (as fraction of tranche notional)
    - 3-7% tranche: 5% expected loss
    - 7-100% tranche: ? expected loss

    Calculate the expected loss of the 7-100% tranche required for no-arbitrage.

---

### Solution Sketches

**1.** $[7, 10]$: $A = 0.07 \times 500 = \$35\text{mm}$, $D = \$50\text{mm}$, face = $3\% \times 500 = \$15\text{mm}$

**2.** $[0, 3]$, $L = 2.5\%$: $\text{TL} = 2.5\%$, outstanding = $3\% - 2.5\% = 0.5\%$ of portfolio = $0.5\text{mm}$ on $100\text{mm}$

**3.** $[3, 7]$: $L = 2.5\% < A = 3\%$ → $\text{TL} = 0$. Subordination from equity protects mezzanine.

**4.** $L = 8\%$: $[0, 3]$ loss = 3% (wiped), $[3, 7]$ loss = 4% (wiped), $[7, 10]$ loss = 1% (remaining 2%)

**5.** $9\% \to 12\%$: At 9%, $[7, 10]$ has 2% loss; at 10% it's 3% (wiped). At 10%, $[10, 15]$ starts. From 10% to 12%, 2% goes to $[10, 15]$. So: $[7, 10]$ gets 1%, $[10, 15]$ gets 2%.

**6.** Premium = $\alpha \cdot S \cdot N_{\text{out}} = 0.25 \times 0.04 \times 1.5\text{mm} = \$15{,}000$

**7.** Three cases: (i) $L \le A$: both formulas give 0. (ii) $A < L < D$: both give $L - A$. (iii) $L \ge D$: max-max gives $(L-A) - (L-D) = D-A$; min-max gives $D-A$.

**8.** High correlation means outcomes are bimodal: either most credits survive (equity untouched) or most default (equity wiped anyway). More probability of the "all survive" scenario benefits equity.

**9.** Each name = $2mm. Loss per default = $\$2\text{mm} \times 0.60 = \$1.2\text{mm}$. Four defaults = $4.8mm. $L = 4.8\%$.

**10.** For $[0, 3]$: TL rises linearly from 0 to 3% as L goes 0→3%, then flat at 3%. For $[10, 15]$: TL = 0 for L ≤ 10%, rises linearly 10%→15%, then flat at 5%.

**16.** Leverage = $550,000 / $30,000 = 18.33x. The equity tranche has 18.33 times the spread sensitivity of an equivalent notional of the index. A 1bp index move produces ~18x P&L.

**17.** By conservation: $3\% \times 25\% + 4\% \times 5\% + 93\% \times \text{EL}(7\text{-}100\%) = 3\%$

$0.75\% + 0.20\% + 93\% \times \text{EL} = 3\%$

$\text{EL}(7\text{-}100\%) = (3\% - 0.95\%) / 93\% = 2.05\% / 93\% \approx 0.022\%$ (very small, as expected for super-senior)

---

## Source Map

### (A) Book-Verified Facts

| Fact | Source |
|------|--------|
| CDO definition as securities linked to portfolio defaults | O'Kane Ch 12 |
| Cash flow CDO structure with SPV and waterfall | O'Kane Ch 12.4 |
| STCDO vs traditional CDO differences (7 points) | O'Kane Ch 12.5 |
| "Equity" terminology from corporate capital structure analogy | O'Kane Ch 12, footnote |
| Tranche loss function formula $L(T, K_1, K_2)$ | O'Kane Ch 12.5.1, Eq 12.1 |
| Premium leg paid on surviving notional | O'Kane Ch 12.5.2 |
| Protection leg as change in tranche loss function | O'Kane Ch 12.5.3 |
| Standard index tranche attachment points table | O'Kane Ch 12.8, Table 12.4 |
| Conservation of expected loss derivation | O'Kane Ch 12.7.1, Eq 12.3 |
| Portfolio loss formula with recovery | O'Kane Ch 12.6 |
| Correlation effect on loss distribution shape | O'Kane Ch 12.6 |
| Senior prefers low correlation, equity prefers high | O'Kane Ch 12.6 |
| Leverage ratio definition and formula | O'Kane Ch 17.2.2 |
| Leverage ratio table (equity 18.37x, super-senior 0.39x) | O'Kane Table 17.3 |
| Leveraged super-senior structure and mechanics | O'Kane Ch 22.8 |
| CDO-squared structure and risk | O'Kane Ch 22.4 |
| Virtual reference portfolio concept | O'Kane Ch 12.5.3 |
| Insurance companies prefer senior tranches | O'Kane Ch 1 |
| Tranche notional as put spread | McNeil QRM Ch 9 |
| Originate-to-distribute model failures | Hull RM Ch 2, Ch 6 |
| ABS CDO structure and 2008 crisis | Hull RM Ch 6 |
| ABS CDO loss amplification example (10% threshold) | Hull RM Ch 6 |
| Senior tranche adjustment for $D > L_{\max}$ | O'Kane Ch 12.5.4 |
| Unfunded nature favors above-Libor funders | O'Kane Ch 1 |
| Call-spread formulation of tranche loss | Standard result; equivalent to O'Kane Eq 12.1 |

### (B) Claude-Extended Content

| Content | Basis |
|---------|-------|
| "What Middle Office Sees" box | Extends O'Kane's discussion with operational perspective for target audience |
| Flood insurance analogy | Pedagogical extension of the subordination concept |
| CDX IG vs HY attachment point rationale | Inferred from O'Kane's discussion of expected loss and standard tranches |
| AIG and leveraged super-senior connection | Historical extension from O'Kane Ch 22.8 |

### (C) Reasoned Inference

| Inference | Derivation |
|-----------|------------|
| Portfolio-scale TL formula from normalized form | Multiply O'Kane's $L(T, K_1, K_2)$ by width $(K_2 - K_1)$ |
| Call-spread equivalence $(L-A)^+ - (L-D)^+$ | Algebraic manipulation of $\min(\max(L-A,0), D-A)$ |
| Dollar conversions | Direct multiplication by $N_{\text{port}}$ |
| Premium payment formula in dollars | Scale O'Kane's per-$1 formula by face notional |
| Loss conservation for contiguous tranches | Telescoping sum from O'Kane's Eq 12.3 |
| Effective width $w_{\text{eff}}$ formula | From O'Kane 12.5.4 discussion of unreachable detachment |
| Liquidity hierarchy across tranches | From market participant preferences in O'Kane Ch 1 |

### (D) Flagged Uncertainties

| Topic | Uncertainty Note |
|-------|------------------|
| Exact collateral mechanics for synthetic tranches | Deal-specific; requires ISDA confirmation |
| Current standard tranche grids | May have evolved post-crisis; verify with current index rules |
| Settlement timeline specifics | Requires tranche rulebook / ISDA confirmation |
| Quantification of model risk | Requires desk methodology and market data |
| Post-crisis market structure changes | Sources predate some regulatory reforms (Dodd-Frank, Basel III) |

---

## Cross-References

- **Chapters 35-37** (Credit Fundamentals): Default, recovery, survival probabilities that feed into portfolio loss
- **Chapters 38-44** (CDS Mechanics): CDS mechanics—standard tranches reference these indices
- **Chapters 45-46** (CDS Indices): Index mechanics, intrinsic spread—standard tranches are slices of these indices
- **Chapter 47** (Index Hedging): Single-name vs index hedging strategies applicable to tranche books
- **Chapter 49** (ETL and PV): Expected tranche loss and tranche pricing mechanics
- **Chapter 50** (Correlation): Gaussian copula, base correlation, why tranches need correlation models
- **Chapter 51** (Tranche Risk): Tranche sensitivities, delta, gamma, correlation risk

---

*Chapter 48 of Fixed Income: Practice and Theory*
